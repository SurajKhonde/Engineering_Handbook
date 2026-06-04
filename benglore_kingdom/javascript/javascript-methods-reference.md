# JavaScript Methods Reference

> JavaScript strings are **primitive and immutable**: all string methods produce a *new* string without altering the original.

---

## String

### Properties
| Method | Description | Example |
|--------|-------------|---------|
| `length` | Number of UTF-16 code units | `"hello".length` → `5` |

### Character Access
| Method | Description | Example |
|--------|-------------|---------|
| `charAt(i)` | Character at index `i` | `"abc".charAt(1)` → `"b"` |
| `charCodeAt(i)` | UTF-16 code at index `i` | `"A".charCodeAt(0)` → `65` |
| `codePointAt(i)` | Unicode code point at `i` | `"😀".codePointAt(0)` → `128512` |
| `at(i)` | Char at `i` (supports negatives) | `"abc".at(-1)` → `"c"` |
| `str[i]` | Bracket access | `"abc"[0]` → `"a"` |

### Building & Combining
| Method | Description | Example |
|--------|-------------|---------|
| `concat(...s)` | Joins strings | `"a".concat("b","c")` → `"abc"` |
| `repeat(n)` | Repeats string `n` times | `"ab".repeat(2)` → `"abab"` |
| `padStart(len, pad)` | Pads from start | `"5".padStart(3,"0")` → `"005"` |
| `padEnd(len, pad)` | Pads from end | `"5".padEnd(3,"0")` → `"500"` |

### Extracting
| Method | Description | Example |
|--------|-------------|---------|
| `slice(start, end)` | Extract section (supports negatives) | `"abcde".slice(1,3)` → `"bc"` |
| `substring(start, end)` | Extract section (no negatives) | `"abcde".substring(1,3)` → `"bc"` |
| `substr(start, len)` | Extract by length *(deprecated)* | `"abcde".substr(1,3)` → `"bcd"` |
| `split(sep)` | Splits into array | `"a,b,c".split(",")` → `["a","b","c"]` |

### Transforming
| Method | Description | Example |
|--------|-------------|---------|
| `toUpperCase()` | Uppercase | `"abc".toUpperCase()` → `"ABC"` |
| `toLowerCase()` | Lowercase | `"ABC".toLowerCase()` → `"abc"` |
| `trim()` | Removes whitespace both ends | `"  hi  ".trim()` → `"hi"` |
| `trimStart()` | Removes leading whitespace | `"  hi".trimStart()` → `"hi"` |
| `trimEnd()` | Removes trailing whitespace | `"hi  ".trimEnd()` → `"hi"` |
| `replace(a, b)` | Replaces first match | `"aa".replace("a","b")` → `"ba"` |
| `replaceAll(a, b)` | Replaces all matches | `"aa".replaceAll("a","b")` → `"bb"` |
| `isWellFormed()` | Checks valid UTF-16 | `"😀".isWellFormed()` → `true` |
| `toWellFormed()` | Returns well-formed copy | replaces lone surrogates with `�` |

### Search Methods
| Method | Description | Example |
|--------|-------------|---------|
| `indexOf(s)` | First index of `s` (or `-1`) | `"abca".indexOf("a")` → `0` |
| `lastIndexOf(s)` | Last index of `s` (or `-1`) | `"abca".lastIndexOf("a")` → `3` |
| `search(regex)` | Index of regex match | `"abc".search(/b/)` → `1` |
| `match(regex)` | Match result / `null` | `"a1b2".match(/\d/g)` → `["1","2"]` |
| `matchAll(regex)` | Iterator of all matches | `[..."a1".matchAll(/\d/g)]` |
| `includes(s)` | Boolean contains | `"abc".includes("b")` → `true` |
| `startsWith(s)` | Boolean prefix | `"abc".startsWith("a")` → `true` |
| `endsWith(s)` | Boolean suffix | `"abc".endsWith("c")` → `true` |

---

## Number

### Basic Methods
*Usable on any number.*
| Method | Description | Example |
|--------|-------------|---------|
| `toString(radix)` | Number to string | `(255).toString(16)` → `"ff"` |
| `toExponential(d)` | Exponential notation | `(12345).toExponential(2)` → `"1.23e+4"` |
| `toFixed(d)` | Fixed decimal places | `(3.14159).toFixed(2)` → `"3.14"` |
| `toPrecision(d)` | Specified precision | `(3.14159).toPrecision(3)` → `"3.14"` |
| `valueOf()` | Primitive number value | `(5).valueOf()` → `5` |

### Static Methods
*Usable only on `Number`.*
| Method | Description | Example |
|--------|-------------|---------|
| `Number.isFinite(x)` | Is finite number | `Number.isFinite(10)` → `true` |
| `Number.isInteger(x)` | Is integer | `Number.isInteger(10.5)` → `false` |
| `Number.isNaN(x)` | Is NaN | `Number.isNaN(NaN)` → `true` |
| `Number.isSafeInteger(x)` | Within safe int range | `Number.isSafeInteger(2**53)` → `false` |
| `Number.parseInt(s)` | String to integer | `Number.parseInt("42px")` → `42` |
| `Number.parseFloat(s)` | String to float | `Number.parseFloat("3.14m")` → `3.14` |

### Useful Constants
| Constant | Value |
|----------|-------|
| `Number.MAX_SAFE_INTEGER` | `9007199254740991` |
| `Number.MIN_SAFE_INTEGER` | `-9007199254740991` |
| `Number.MAX_VALUE` | `~1.79e308` |
| `Number.MIN_VALUE` | `~5e-324` |
| `Number.EPSILON` | `~2.22e-16` |
| `Number.POSITIVE_INFINITY` | `Infinity` |
| `Number.NEGATIVE_INFINITY` | `-Infinity` |
| `Number.NaN` | `NaN` |

---

## Object

### Static Methods
| Method | Description | Example |
|--------|-------------|---------|
| `Object.keys(o)` | Array of own keys | `Object.keys({a:1})` → `["a"]` |
| `Object.values(o)` | Array of own values | `Object.values({a:1})` → `[1]` |
| `Object.entries(o)` | Array of `[key, value]` pairs | `Object.entries({a:1})` → `[["a",1]]` |
| `Object.fromEntries(arr)` | Pairs back to object | `Object.fromEntries([["a",1]])` → `{a:1}` |
| `Object.assign(t, ...s)` | Copies props into target | `Object.assign({},{a:1})` → `{a:1}` |
| `Object.freeze(o)` | Makes immutable | prevents add/change/delete |
| `Object.isFrozen(o)` | Is frozen | `Object.isFrozen(frozen)` → `true` |
| `Object.seal(o)` | Prevents add/delete (allows edit) | — |
| `Object.isSealed(o)` | Is sealed | — |
| `Object.create(proto)` | New object with prototype | `Object.create(null)` |
| `Object.getPrototypeOf(o)` | Returns prototype | — |
| `Object.setPrototypeOf(o, p)` | Sets prototype | — |
| `Object.defineProperty(o, k, d)` | Defines property with descriptor | — |
| `Object.defineProperties(o, ds)` | Defines multiple properties | — |
| `Object.getOwnPropertyNames(o)` | All own keys incl. non-enumerable | — |
| `Object.getOwnPropertyDescriptor(o, k)` | Property descriptor | — |
| `Object.getOwnPropertyDescriptors(o)` | All descriptors | — |
| `Object.hasOwn(o, k)` | Own property check *(modern)* | `Object.hasOwn({a:1},"a")` → `true` |
| `Object.is(a, b)` | Strict equality (handles `NaN`, `-0`) | `Object.is(NaN,NaN)` → `true` |
| `Object.groupBy(arr, fn)` | Groups items by callback | `{even:[...], odd:[...]}` |

### Instance Methods
| Method | Description | Example |
|--------|-------------|---------|
| `hasOwnProperty(k)` | Own property check | `({a:1}).hasOwnProperty("a")` → `true` |
| `isPrototypeOf(o)` | In prototype chain | — |
| `propertyIsEnumerable(k)` | Is enumerable | — |
| `toString()` | String representation | `"[object Object]"` |
| `valueOf()` | Primitive value | — |

---

## Array

### Creating & Checking
| Method | Description | Example |
|--------|-------------|---------|
| `Array.isArray(x)` | Is an array | `Array.isArray([])` → `true` |
| `Array.from(iter, fn?)` | Iterable/array-like to array | `Array.from("ab")` → `["a","b"]` |
| `Array.of(...items)` | Array from arguments | `Array.of(1,2,3)` → `[1,2,3]` |

### Adding / Removing (mutating)
| Method | Description | Example |
|--------|-------------|---------|
| `push(...x)` | Adds to end, returns new length | `[1].push(2)` → `2` |
| `pop()` | Removes & returns last | `[1,2].pop()` → `2` |
| `shift()` | Removes & returns first | `[1,2].shift()` → `1` |
| `unshift(...x)` | Adds to start, returns length | `[2].unshift(1)` → `2` |
| `splice(start, count, ...items)` | Add/remove in place | `[1,2,3].splice(1,1)` → `[2]` |
| `fill(val, start?, end?)` | Fills with value | `[1,2,3].fill(0)` → `[0,0,0]` |
| `copyWithin(t, s, e?)` | Copies part within array | `[1,2,3,4].copyWithin(0,2)` → `[3,4,3,4]` |

### Sorting / Reversing (mutating)
| Method | Description | Example |
|--------|-------------|---------|
| `sort(fn?)` | Sorts in place | `[3,1,2].sort()` → `[1,2,3]` |
| `reverse()` | Reverses in place | `[1,2,3].reverse()` → `[3,2,1]` |

### Immutable Variants (return new array)
| Method | Description | Example |
|--------|-------------|---------|
| `toSorted(fn?)` | Sorted copy | `[3,1].toSorted()` → `[1,3]` |
| `toReversed()` | Reversed copy | `[1,2].toReversed()` → `[2,1]` |
| `toSpliced(s, c, ...x)` | Spliced copy | — |
| `with(i, val)` | Copy with index replaced | `[1,2,3].with(1,9)` → `[1,9,3]` |

### Combining / Extracting (non-mutating)
| Method | Description | Example |
|--------|-------------|---------|
| `concat(...arrs)` | Merges arrays | `[1].concat([2,3])` → `[1,2,3]` |
| `slice(start?, end?)` | Shallow copy of section | `[1,2,3].slice(1)` → `[2,3]` |
| `join(sep)` | Joins into string | `[1,2,3].join("-")` → `"1-2-3"` |
| `flat(depth?)` | Flattens nested arrays | `[1,[2,[3]]].flat()` → `[1,2,[3]]` |
| `flatMap(fn)` | Map then flatten one level | `[1,2].flatMap(x=>[x,x])` → `[1,1,2,2]` |

### Searching
| Method | Description | Example |
|--------|-------------|---------|
| `indexOf(x)` | First index (or `-1`) | `[1,2,1].indexOf(1)` → `0` |
| `lastIndexOf(x)` | Last index (or `-1`) | `[1,2,1].lastIndexOf(1)` → `2` |
| `includes(x)` | Boolean contains | `[1,2].includes(2)` → `true` |
| `find(fn)` | First matching element | `[1,2,3].find(x=>x>1)` → `2` |
| `findIndex(fn)` | Index of first match | `[1,2,3].findIndex(x=>x>1)` → `1` |
| `findLast(fn)` | Last matching element | `[1,2,3].findLast(x=>x<3)` → `2` |
| `findLastIndex(fn)` | Index of last match | `[1,2,3].findLastIndex(x=>x<3)` → `1` |
| `at(i)` | Element at index (negatives ok) | `[1,2,3].at(-1)` → `3` |

### Iterating / Transforming (non-mutating)
| Method | Description | Example |
|--------|-------------|---------|
| `forEach(fn)` | Runs fn per element | `[1,2].forEach(x=>console.log(x))` |
| `map(fn)` | New array of results | `[1,2].map(x=>x*2)` → `[2,4]` |
| `filter(fn)` | Elements passing test | `[1,2,3].filter(x=>x>1)` → `[2,3]` |
| `reduce(fn, init?)` | Reduce left→right | `[1,2,3].reduce((a,b)=>a+b)` → `6` |
| `reduceRight(fn, init?)` | Reduce right→left | `[1,2,3].reduceRight((a,b)=>a+b)` → `6` |
| `some(fn)` | True if any match | `[1,2].some(x=>x>1)` → `true` |
| `every(fn)` | True if all match | `[1,2].every(x=>x>0)` → `true` |
| `keys()` | Iterator of indexes | `[..."ab".keys?]` → `0,1` |
| `values()` | Iterator of values | — |
| `entries()` | Iterator of `[i, value]` | — |
| `Array.prototype.toString()` | Comma-joined string | `[1,2].toString()` → `"1,2"` |

---

## Function Methods

| Method | Description | Example |
|--------|-------------|---------|
| `call(thisArg, ...args)` | Calls function with chosen `this` + list of args | `fn.call(obj, 1, 2)` |
| `apply(thisArg, argsArray)` | Calls function with chosen `this` + array of args | `fn.apply(obj, [1, 2])` |
| `bind(thisArg, ...args)` | Returns a new function with bound `this`, to run later | `const g = fn.bind(obj)` |

### IIFE — Immediately Invoked Function Expression
A self-starting function that runs as soon as it is defined.

```js
(function () {
  console.log("runs immediately");
})();

// Arrow version
(() => {
  console.log("runs immediately");
})();
```

**call() vs apply() vs bind()**

```js
const person = { name: "Suraj" };

function greet(greeting, punct) {
  return greeting + ", " + this.name + punct;
}

greet.call(person, "Hi", "!");      // "Hi, Suraj!"   → args as list
greet.apply(person, ["Hi", "!"]);   // "Hi, Suraj!"   → args as array
const later = greet.bind(person);   // returns function
later("Hi", "!");                   // "Hi, Suraj!"   → runs later
```
