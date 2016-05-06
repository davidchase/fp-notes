# fp-notes
Experiments and Learnings in Functional Programming 


## Short cut fusion
> Optimization techinque that allows for removal of intermediate data structures. <sup>[1]</sup>

##### Simple Example:
```js
const list = [1,2,3,4];

// calls map 3x creating intermediate lists :(
const f = compose(map(triple), map(double), map(inc));

f(list); // [12, 18, 24, 30]

// fancy ƒ calls map once passing in a function composed
// of triple, double, inc... fusion FTW :)
const ƒ = map(compose(triple, double, inc));

ƒ(list); // [12, 18, 24, 30]
```
##### More Envolved Example:

```js
const list = range(1, 10);

// same as before calls take, then filter and finally map
// not as efficient as we can be :(
const f = compose(map(inc), filter(odd), take(3));

// ideally we would like build a composition of the above functions
// to pass to a single "looping" mechanism

// what if we redefined those functions in terms of... let say reduce
const mapβ = curry((fn, combineFn) => (acc, input) => combineFn(acc, fn(input)));
const filterβ = curry((pred, combineFn) => (acc, input) => pred(input) ? combineFn(acc, input) : acc);
const takeβ = curry((n, combineFn) => (acc, input) => acc.length !== n ? combineFn(acc, input) : acc);

// each of these functions redefined in terms of reduce have some new parts
// - combineFn is also the accumulation function IOW it is the way the values get accumulated ie: concat for lists and add for numbers, etc
// - (acc, input) is sometimes referred to as a "reducer" or a reducing function becauses its the 1st parameter to reduce :)

// visulize with some moar code:

// 1st we increment all of the values in the list by 10
// the accumulation fn or combineFn here is `concat` so we are going from list -> list

// mapβ(add(10)) is `fn`
// concat is `combineFn`
// [] is `acc` or initial value
// range(1,5) is the `input`
reduce(mapβ(add(10), concat), [], range(1, 5)); // [11, 12, 13, 14]

// 1st we increment all of the values in the list by 1
// and then combine all of the values together with `add` so going from list -> num
reduce(mapβ(inc), add), [], range(1, 5)); // 14

// back to the above example, we can do this
// 1st map (inc) then filter (odd) then take (3) finally concat to accumulate the results
// mapβ, filterβ, takeβ and concat are all reducing functions (reducers) getting passed to each fns `combineFn` parameter
reduce(mapβ(inc, filterβ(odd, takeβ(3, concat))), [], range(1, 10)); // [3, 5, 7]

```

#### References 
- https://wiki.haskell.org/Short_cut_fusion
- http://www.slideshare.net/drboolean/millionways
[1]: https://wiki.haskell.org/Short_cut_fusion
