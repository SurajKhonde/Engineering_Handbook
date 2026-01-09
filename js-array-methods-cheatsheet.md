# JavaScript Array Methods — Cheat Sheet (with examples)

This is a **quick reference** for JavaScript **Array built‑in methods** (both **static** `Array.*` and instance methods on `Array.prototype`).

> Notes:
> - **Mutates?** = changes the original array.
> - “Copying methods” (like `toSorted`) return a new array and keep original unchanged (ES2023+). citeturn0search1turn0search10turn0search7  
> - `Object.groupBy()` and `Map.groupBy()` are the standardized “grouping” APIs (some browsers previously exposed `Array.prototype.group`). citeturn0search16turn0search3

---

## Fast table (scan + pick)

### Static methods (`Array.*`)

| Method | What it does | Mutates? | Returns |
|---|---|---:|---|
| `Array.isArray(x)` | checks if value is an array | — | boolean |
| `Array.from(iterable, mapFn?)` | makes an array from iterable/array‑like | — | new array |
| `Array.of(...items)` | makes an array from arguments | — | new array |

### Instance methods (`arr.*`)

| Method | Category | What it does | Mutates? | Returns |
|---|---|---|---:|---|
| `at(i)` | access | element at index (supports negative index) | ❌ | value/`undefined` |
| `concat(...arrays)` | copy/merge | merges arrays | ❌ | new array |
| `copyWithin(t, s, e?)` | edit | copies part within same array | ✅ | same array |
| `entries()` | iterate | iterator of `[index, value]` | ❌ | iterator |
| `every(fn)` | test | all items pass? | ❌ | boolean |
| `fill(v, s?, e?)` | edit | fills range with value | ✅ | same array |
| `filter(fn)` | transform | keeps items that pass test | ❌ | new array |
| `find(fn)` | search | first item that matches | ❌ | value/`undefined` |
| `findIndex(fn)` | search | index of first match | ❌ | number |
| `findLast(fn)` | search | last item that matches | ❌ | value/`undefined` citeturn0search11 |
| `findLastIndex(fn)` | search | index of last match | ❌ | number citeturn0search2 |
| `flat(depth=1)` | transform | flattens nested arrays | ❌ | new array |
| `flatMap(fn)` | transform | `map` then `flat(1)` | ❌ | new array |
| `forEach(fn)` | iterate | runs fn for each item | ❌ | `undefined` |
| `includes(x, from?)` | search | contains value? (`NaN` works) | ❌ | boolean |
| `indexOf(x, from?)` | search | first index of value | ❌ | number |
| `join(sep=',')` | convert | joins to string | ❌ | string |
| `keys()` | iterate | iterator of indexes | ❌ | iterator |
| `lastIndexOf(x, from?)` | search | last index of value | ❌ | number |
| `map(fn)` | transform | transforms every item | ❌ | new array |
| `pop()` | add/remove | remove last | ✅ | removed item |
| `push(...items)` | add/remove | add to end | ✅ | new length |
| `reduce(fn, init?)` | reduce | folds to single value | ❌ | value |
| `reduceRight(fn, init?)` | reduce | reduce from right | ❌ | value |
| `reverse()` | reorder | reverses in place | ✅ | same array |
| `shift()` | add/remove | remove first | ✅ | removed item |
| `slice(s?, e?)` | copy | extracts portion | ❌ | new array |
| `some(fn)` | test | any item pass? | ❌ | boolean |
| `sort(compare?)` | reorder | sorts in place (lexicographic by default) | ✅ | same array |
| `splice(i, del?, ...items)` | edit | add/remove in place | ✅ | removed items array |
| `toLocaleString()` | convert | localized string | ❌ | string |
| `toString()` | convert | comma‑separated string | ❌ | string |
| `unshift(...items)` | add/remove | add to start | ✅ | new length |
| `values()` | iterate | iterator of values | ❌ | iterator |
| `with(i, v)` | copy/edit | copy with one index replaced | ❌ | new array citeturn0search5turn0search7 |
| `toReversed()` | copy/reorder | copy version of `reverse()` | ❌ | new array citeturn0search5turn0search7 |
| `toSorted(compare?)` | copy/reorder | copy version of `sort()` | ❌ | new array citeturn0search1turn0search7 |
| `toSpliced(i, del?, ...items)` | copy/edit | copy version of `splice()` | ❌ | new array citeturn0search10turn0search7 |

---

## Core examples (based on your snippet)

```js
let fruits = ['apple', 'banana', 'mango'];

// push/pop
fruits.push('orange');       // ✅ mutates, returns new length
fruits.pop();                // ✅ mutates, returns removed item

// unshift/shift
fruits.unshift('grape');     // ✅ mutates, returns new length
fruits.shift();              // ✅ mutates, returns removed item

// indexOf
fruits.indexOf('banana');    // 1 (or -1 if not found)

// splice (mutates)
fruits.splice(1, 1, 'kiwi'); // remove 1 at index 1 and insert 'kiwi'

// slice (non-mutating)
const citrus = fruits.slice(1, 3);

// forEach (returns undefined)
fruits.forEach((item, index) => console.log(index + ': ' + item));

// map (new array)
const upperFruits = fruits.map(item => item.toUpperCase());

// filter (new array)
// NOTE: 'kiwi'.length is 4, so length > 4 will keep only apple & mango
const longFruits = fruits.filter(item => item.length > 4);
```

---

## More examples by category

### 1) Access & copy
```js
const a = [10, 20, 30];

a.at(0);    // 10
a.at(-1);   // 30

a.slice(1);       // [20, 30]
a.concat([40]);   // [10, 20, 30, 40]
a.with(1, 99);    // [10, 99, 30]  (original a unchanged)
```

### 2) Add / remove (mutating)
```js
const a = ['x', 'y'];

a.push('z');   // a: ['x','y','z']
a.pop();       // returns 'z', a: ['x','y']

a.unshift('w'); // a: ['w','x','y']
a.shift();      // returns 'w', a: ['x','y']
```

### 3) Edit in place
```js
const a = [1, 2, 3, 4];

a.splice(1, 2, 9, 9); // remove [2,3], insert [9,9] => a: [1,9,9,4]
a.fill(0, 1, 3);      // a: [1,0,0,4]
a.copyWithin(0, 2);   // copy from index 2..end to start => a: [0,4,0,4] (after above)
```

### 4) Transform
```js
const a = [1, 2, 3];

a.map(x => x * 2);          // [2,4,6]
a.filter(x => x % 2 === 1); // [1,3]
a.flatMap(x => [x, x]);     // [1,1,2,2,3,3]

[1, [2, [3]]].flat(2);      // [1,2,3]
```

### 5) Search
```js
const a = [5, 6, 7, 6];

a.includes(6);         // true
a.indexOf(6);          // 1
a.lastIndexOf(6);      // 3

a.find(x => x > 5);           // 6
a.findIndex(x => x > 5);      // 1

a.findLast(x => x === 6);     // 6
a.findLastIndex(x => x === 6);// 3
```

### 6) Tests
```js
const a = [2, 4, 6];

a.every(x => x % 2 === 0); // true
a.some(x => x > 5);        // true
```

### 7) Reduce (fold)
```js
const a = [1, 2, 3, 4];

a.reduce((sum, x) => sum + x, 0); // 10
a.reduceRight((s, x) => s + x, ""); // "4321"
```

### 8) Reorder
```js
const a = [3, 1, 2];

a.sort();         // ✅ mutates: [1,2,3] (string sort can be surprising)
a.sort((x,y)=>x-y); // numeric sort

a.toSorted((x,y)=>x-y); // ✅ non-mutating copy sort

a.reverse();      // ✅ mutates
a.toReversed();   // ✅ non-mutating copy reverse
```

### 9) Convert
```js
const a = [1, 2, 3];

a.join("-");        // "1-2-3"
a.toString();       // "1,2,3"
a.toLocaleString(); // locale formatted string
```

### 10) Iterators
```js
const a = ["a", "b"];

[...a.keys()];    // [0, 1]
[...a.values()];  // ["a", "b"]
[...a.entries()]; // [[0,"a"], [1,"b"]]
```

---

## Grouping (modern standard)

Use **`Object.groupBy()`** when you want an object result, or **`Map.groupBy()`** when you want a `Map`. citeturn0search16turn0search3

```js
const people = [
  { name: "Suraj", city: "Hyderabad" },
  { name: "Asha",  city: "Hyderabad" },
  { name: "Vijay", city: "Pune" },
];

const byCityObj = Object.groupBy(people, p => p.city);
// { Hyderabad: [...], Pune: [...] }

const byCityMap = Map.groupBy(people, p => p.city);
// Map("Hyderabad" => [...], "Pune" => [...])
```

---

## Quick “which one should I use?”

- **Need a new array** (don’t mutate): `map`, `filter`, `slice`, `concat`, `flat`, `flatMap`, `toSorted`, `toReversed`, `toSpliced`, `with`
- **Need to mutate** (performance / intentional): `push`, `pop`, `shift`, `unshift`, `splice`, `sort`, `reverse`, `fill`, `copyWithin`
- **Need a single value**: `reduce`
- **Need to search**: `find` / `findIndex` (by condition), `includes` / `indexOf` (by value)

---

## Tiny gotchas to remember

- `sort()` sorts **as strings** by default; use `compareFn` for numbers.
- `forEach()` returns **`undefined`** (use `map()` if you need a new array).
- `splice()` mutates; `toSpliced()` is the non-mutating copy version. citeturn0search10turn0search7

---
