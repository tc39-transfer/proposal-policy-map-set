# Policy Maps and Sets for JavaScript
ECMAScript Stage-0 Proposal. 2022.

Champions: Hemanth HM; J. S. Choi.

## Rationale
> [There are only two hard things in computer science: cache invalidation and naming things.][two hard things]

[two hard things]: https://www.martinfowler.com/bliki/TwoHardThings.html

Developers often use mapping data structures as caches, and they often wish to
limit the memory consumption of said caches. [Cache-replacement
policies][policies] are common, useful, repetitive, and annoying to
reimplement.

For example, the proposed [memo function][] (for [function memoization][])
would use Map-like objects to store recent function calls’ arguments and
results, giving the developer the power to decide on what cache-replacement
policy and limit they wish:

```js
const cache = new LIFOMap(256);
const fnMemo = expensiveFn.memo(cache);
console.log([ fnMemo(0) ]);
// Only the first fnMemo(0) calls expensiveFn; the second fnMemo(0) looks up
// the result of the first call in the cache.
```

However, without built-in Map-like classes with cache policies, it would be
difficult, annoying, and unergonomic to assign a policy to the memo function
from a userland library or hand-written data structure.

[policies]: https://en.wikipedia.org/wiki/Cache_replacement_policies
[memo function]: https://github.com/js-choi/proposal-function-memo
[function memoization]: https://en.wikipedia.org/wiki/Memoization

## Description
The proposal would add several built-in classes to the global object. Each of
these have either a mutable Map-like interface or a mutable Set-like interface (although neither are actual subclasses of Map or Set; see [issue #1][]).

For the Map-like classes, these include:

`m.size`: The number of entries in `m`.

`m.has(key)`: Returns a boolean of whether `m` has an entry with the given
`key`.

`m.get(key)`: Returns the value of the entry in `m` with the given `key`, if
any such entry exists; otherwise returns `undefined`.

`m.set(key, value)`: Adds an entry to `m` with the given `key` mapped to the
given `value`. Returns `m` itself.

`m.delete(key)`: Deletes the entry in `m` with the given `key`, if any. Returns
a boolean of whether `m` did have an entry with the `key` before it was
deleted.

`m.clear()`: Removes all entries from `m`. Returns `undefined`.

`m[Symbol.iterator]()`: Uncertain if we should implement. See [issue #3][].

`m.entries()`: Uncertain if we should implement. See [issue #3][].

`m.keys()`: Uncertain if we should implement. See [issue #3][].

`m.values()`: Uncertain if we should implement. See [issue #3][].

`m.forEach()`: Uncertain if we should implement. See [issue #3][].

***

For the Set-like classes, these include:

`s.size`: The number of values in `s`.

`s.has(value)`: Returns a boolean of whether `s` has the given `value`.

`s.add(value)`: Adds the given `value` to `s`. Returns `s` itself.

`s.delete(key)`: Deletes the given `value` from `s`, if `s` has `value`.
Returns a boolean of whether `s` did have `value` before `value` was deleted.

`s.clear()`: Removes all values from `s`. Returns `undefined`.

`m[Symbol.iterator]()`: Uncertain if we should implement. See [issue #3][].

`s.values()`: Uncertain if we should implement. See [issue #3][].

`s.forEach()`: Uncertain if we should implement. See [issue #3][].

***

Unlike Map and Set, none of these classes have a Symbol.species instance
property.

### FIFOMap and FIFOSet

```js
new FIFOMap(maxNumOfEntries, entries = [])
new FIFOSet(maxNumOfValues, values = [])
```

The constructors throw TypeErrors if they are given non-integer max numbers of
entries/values, or if the initial entries/values are not iterable.

Their instances evict entries/values in the order they were added, as if they
were [FIFO queues][].

[FIFO queues]: https://en.wikipedia.org/wiki/FIFO_(computing_and_electronics)

### LIFOMap and LIFOSet

```js
new LIFOMap(maxNumOfEntries, entries = [])
new LIFOSet(maxNumOfValues, values = [])
```

These constructors throw TypeErrors if they are given non-integer max numbers
of entries/values, or if the initial entries/values are not iterable.

Their instances evict entries/values in the order they were added, as if they
were [LIFO stacks][].

[LIFO stacks]: https://en.wikipedia.org/wiki/Stack_(abstract_data_type)

### LRUMap and LRUSet

```js
new LIFOMap(maxNumOfEntries, entries = [])
new LIFOSet(maxNumOfValues, values = [])
```

These constructors throw TypeErrors if they are given non-integer max numbers
of entries/values, or if the initial entries/values are not iterable.

Their instances first evict the entries/values that were least recently
accessed with `.get` (for LRUMap) or `.has` (for LRUSet).

### LFUMap and LFUSet

```js
new LIFOMap(maxNumOfEntries, entries = [])
new LIFOSet(maxNumOfValues, values = [])
```

These constructors throw TypeErrors if they are given non-integer max numbers
of entries/values, or if the initial entries/values are not iterable.

Their instances first evict the entries/values that were least frequently
accessed with `.get` (for LRUMap) or `.has` (for LRUSet).

## Real-world examples
[Real-world examples wanted][issue #4].

## Precedents and web compatibility

* [collections.js][]
* [Python functools.lru_cache][]

[collections.js]: https://www.collectionsjs.com/lru-map
[Python functools.lru_cache]: https://docs.python.org/3/library/functools.html#functools.lru_cache

[issue #1]: https://github.com/js-choi/proposal-policy-map-set/issues/1
[issue #3]: https://github.com/js-choi/proposal-policy-map-set/issues/3
[issue #4]: https://github.com/js-choi/proposal-policy-map-set/issues/4
