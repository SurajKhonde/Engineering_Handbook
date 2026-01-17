# JavaScript String Methods — Cheat Sheet (with examples)

A complete, practical reference for **JavaScript `String`** built‑ins:
- **Static** `String.*`
- **Instance** methods on `String.prototype` (`"text".method()`)

> Key idea: **Strings are immutable** in JS — methods never change the original string. They return a **new string** (or another value).

---

## Fast table (quick scan)

### Static methods (`String.*`)

| Method | What it does | Returns |
|---|---|---|
| `String.fromCharCode(...codes)` | build string from UTF‑16 *code units* | string |
| `String.fromCodePoint(...points)` | build string from Unicode *code points* | string |
| `String.raw(template, ...subs)` | “raw” template literal helper (keeps backslashes) | string |

---

### Instance methods (`str.*`)

#### Access / inspect
| Method | What it does | Returns |
|---|---|---|
| `at(i)` | char at index (supports negative index; code unit) | string |
| `charAt(i)` | char at index (old style) | string |
| `charCodeAt(i)` | UTF‑16 code unit number | number |
| `codePointAt(i)` | Unicode code point number (handles surrogate pairs) | number \| `undefined` |
| `concat(...s)` | join strings | string |
| `endsWith(s, endPos?)` | ends with substring? | boolean |
| `startsWith(s, pos?)` | starts with substring? | boolean |
| `includes(s, pos?)` | contains substring? | boolean |
| `indexOf(s, from?)` | first index of substring | number |
| `lastIndexOf(s, from?)` | last index of substring | number |
| `length` *(property)* | number of UTF‑16 code units | number |

#### Extract / split
| Method | What it does | Returns |
|---|---|---|
| `slice(start?, end?)` | extract part (supports negatives) | string |
| `substring(start, end?)` | extract part (no negatives; swaps args if needed) | string |
| `split(sep?, limit?)` | split to array | string[] |

> `substr(start, length)` exists in some environments but is **deprecated** — avoid using it.

#### Transform (case / normalize / padding / trim)
| Method | What it does | Returns |
|---|---|---|
| `toLowerCase()` / `toUpperCase()` | basic case transform | string |
| `toLocaleLowerCase(loc?)` / `toLocaleUpperCase(loc?)` | locale-aware case | string |
| `normalize(form?)` | Unicode normalization (`"NFC"`, `"NFD"`, `"NFKC"`, `"NFKD"`) | string |
| `padStart(len, pad?)` / `padEnd(len, pad?)` | pads string to length | string |
| `repeat(count)` | repeats string | string |
| `trim()` | trims both sides whitespace | string |
| `trimStart()` / `trimEnd()` | trims one side (`trimLeft/trimRight` are aliases) | string |

#### Replace / match (Regex)
| Method | What it does | Returns |
|---|---|---|
| `replace(pattern, repl)` | replace first match (or regex matches) | string |
| `replaceAll(pattern, repl)` | replace **all** matches | string |
| `match(regex)` | regex match result | array \| `null` |
| `matchAll(regex)` | iterator of **all** matches | iterator |
| `search(regex)` | index of first regex match | number |

#### Compare / convert
| Method | What it does | Returns |
|---|---|---|
| `localeCompare(other, locales?, options?)` | compare strings (sorting) | number |
| `toString()` | string representation | string |
| `valueOf()` | primitive value of string object | string |
| `toLocaleString()` | locale string (same as `toString` for primitives) | string |

#### Unicode well‑formed strings
| Method | What it does | Returns |
|---|---|---|
| `isWellFormed()` | checks if string has **no lone surrogates** | boolean |
| `toWellFormed()` | replaces lone surrogates with `\uFFFD` | string |

#### Legacy / deprecated HTML wrapper methods (avoid)
These create HTML markup strings and are **deprecated**:
`anchor`, `big`, `blink`, `bold`, `fixed`, `fontcolor`, `fontsize`, `italics`, `link`, `small`, `strike`, `sub`, `sup`

---

## Core examples (daily use)

```js
const s = "  Hello World  ";

s.trim();                 // "Hello World"
s.toLowerCase();          // "  hello world  "
s.includes("World");      // true
s.startsWith("  He");     // true
s.endsWith("  ");         // true

"abc".at(-1);             // "c"
"abc".slice(0, 2);        // "ab"
"abc".substring(0, 2);    // "ab"

"a,b,c".split(",");       // ["a","b","c"]
"ha".repeat(3);           // "hahaha"

"5".padStart(3, "0");     // "005"
"5".padEnd(3, ".");       // "5.."
```

---

## Replace & regex examples

### `replace()` vs `replaceAll()`
```js
const t = "foo foo foo";

t.replace("foo", "bar");      // "bar foo foo"
t.replaceAll("foo", "bar");   // "bar bar bar"
```

### Replace using a function
```js
const t = "price: 100";

t.replace(/\d+/g, (m) => String(Number(m) + 1)); // "price: 101"
```

### `match()` vs `matchAll()`
```js
const t = "a1 b2 c3";

t.match(/\d/g);     // ["1","2","3"]  (because /g)
t.match(/\d/);      // ["1"] (first match, with extra groups info)

for (const m of t.matchAll(/(\w)(\d)/g)) {
  console.log(m[0], m[1], m[2]); // "a1 a 1" ...
}
```

---

## Slice vs substring (common interview gotcha)

```js
const x = "abcdef";

// slice supports negative indexes:
x.slice(-2);        // "ef"

// substring does NOT support negatives (treats as 0):
x.substring(-2);    // "abcdef"

// substring swaps arguments if start > end:
x.substring(4, 2);  // "cd"
```

---

## Unicode basics (important in real apps)

### Code units vs code points (emoji issue)
```js
const emoji = "😀";     // U+1F600

emoji.length;           // 2 (surrogate pair, 2 UTF-16 code units)
emoji.charAt(0);        // may show half/garbage in some contexts
emoji.codePointAt(0);   // 0x1F600
String.fromCodePoint(0x1F600); // "😀"
```

### Well‑formed strings (`isWellFormed` / `toWellFormed`)
A “lone surrogate” is an invalid UTF‑16 half (common from bad data).

```js
const bad = "Hi \uD800";     // lone high surrogate

bad.isWellFormed();          // false
bad.toWellFormed();          // "Hi �"  (U+FFFD replacement)
```

---

## Iterating a string

```js
const s = "hi";

for (const ch of s) {
  console.log(ch); // "h", "i"
}

// Convert to array of characters:
[...s];            // ["h","i"]
```

> Note: `for...of` iterates **Unicode code points** (handles most emojis better than index access).

---

## Practical mini-recipes

### 1) Get file extension safely
```js
function ext(name) {
  const i = name.lastIndexOf(".");
  return i === -1 ? "" : name.slice(i + 1).toLowerCase();
}
```

### 2) Capitalize words
```js
function titleCase(str) {
  return str
    .trim()
    .split(/\s+/)
    .map(w => w.charAt(0).toUpperCase() + w.slice(1).toLowerCase())
    .join(" ");
}
```

### 3) Remove all spaces
```js
const noSpaces = s => s.replaceAll(" ", "");
```

### 4) Check palindrome (simple)
```js
function isPalindrome(s) {
  const cleaned = s.toLowerCase().replace(/[^a-z0-9]/g, "");
  return cleaned === [...cleaned].toReversed().join("");
}
```

---

## Quick “which one should I use?”

- **Contains?** → `includes()` (or `indexOf() !== -1` legacy)
- **Starts/ends?** → `startsWith()`, `endsWith()`
- **Extract** → `slice()` (preferred), `substring()` (legacy behavior)
- **Replace** → `replaceAll()` for global replace, `replace()` for first match
- **Regex matches** → `matchAll()` when you need *all matches + groups*
- **Trim** → `trim()` / `trimStart()` / `trimEnd()`
- **Unicode correctness** → `codePointAt()` / `fromCodePoint()` / `isWellFormed()` / `toWellFormed()`

---

## Full method list (checklist)

### `String.*` (static)
- `String.fromCharCode`
- `String.fromCodePoint`
- `String.raw`

### `String.prototype.*` (instance)
- Access/inspect: `at`, `charAt`, `charCodeAt`, `codePointAt`, `concat`
- Search/compare: `includes`, `indexOf`, `lastIndexOf`, `startsWith`, `endsWith`, `localeCompare`
- Extract/split: `slice`, `substring`, `split`
- Case/format: `toLowerCase`, `toUpperCase`, `toLocaleLowerCase`, `toLocaleUpperCase`
- Trim/pad/repeat: `trim`, `trimStart` (`trimLeft`), `trimEnd` (`trimRight`), `padStart`, `padEnd`, `repeat`
- Regex: `match`, `matchAll`, `replace`, `replaceAll`, `search`
- Unicode: `normalize`, `isWellFormed`, `toWellFormed`
- Convert: `toString`, `valueOf`, `toLocaleString`
- Legacy HTML wrapper (deprecated): `anchor`, `big`, `blink`, `bold`, `fixed`, `fontcolor`, `fontsize`, `italics`, `link`, `small`, `strike`, `sub`, `sup`

---

## Small note about “String objects”
Normally you use primitives: `"text"`.
If you do `new String("text")`, you get an object wrapper (rarely needed).

```js
typeof "x";         // "string"
typeof new String("x"); // "object"
```

---

Happy learning ✅
