# Time & Space Complexity (Big‑O) — Interview Notes (JS/DSA)

This is a practical cheat‑sheet to **calculate time and space complexity** in interviews.

---

## 0) What Big‑O means (in one line)
**Big‑O describes how runtime/extra memory grows when input size `n` grows**, ignoring constants.

---

## 1) Time Complexity — Core Rules

### Rule 1: Ignore constants
- `O(2n)` → `O(n)`
- `O(n/2)` → `O(n)`
- `O(100)` → `O(1)`

### Rule 2: Sequential blocks add (but we later drop smaller terms)
```js
for (let i = 0; i < n; i++) {}   // O(n)
for (let i = 0; i < n; i++) {}   // O(n)
```
Total: `O(n + n) = O(2n) = O(n)`

### Rule 3: Nested loops multiply
```js
for (let i = 0; i < n; i++) {
  for (let j = 0; j < n; j++) {}
}
```
Total: `O(n * n) = O(n^2)`

### Rule 4: If inner loop length depends on outer, count total work
```js
for (let i = 0; i < n; i++) {
  for (let j = i; j < n; j++) {}
}
```
Operations: `n + (n-1) + ... + 1` → `O(n^2)`

### Rule 5: “Halving” loops are `log n`
```js
while (n > 1) n = Math.floor(n / 2);
```
Time: `O(log n)` because you can divide by 2 only ~`log2(n)` times until reaching 1.

### Rule 6: Drop smaller terms (keep the dominant term)
- `O(n^2 + n)` → `O(n^2)`
- `O(n log n + n)` → `O(n log n)`
- `O(n + log n)` → `O(n)`

---

## 2) What is `log n` (simple intuition)

### Meaning
`log2(n)` = **how many times you can divide `n` by 2 until it becomes 1**.

Example: `n = 16`
- 16 → 8 (1)
- 8 → 4 (2)
- 4 → 2 (3)
- 2 → 1 (4)

So `log2(16) = 4` → `O(log n)` steps.

### Why base doesn’t matter
In Big‑O we ignore constants, so `log2(n)` and `log10(n)` are the same growth rate: `O(log n)`.

### `log n` is smaller than `n`
Some quick values:

| n | log2(n) |
|---:|---:|
| 8 | 3 |
| 16 | 4 |
| 32 | 5 |
| 1,000 | ~10 |
| 1,000,000 | ~20 |
| 1,000,000,000 | ~30 |

So for huge `n`, `log n` stays small.

---

## 3) Space Complexity — Core Rules

### What “space” means
**Extra memory** used by your solution.  
Typically we **do not count input array/string space** (unless asked).

### Rule 1: A few variables is `O(1)`
```js
let a = 1, b = 2; // O(1) extra space
```

### Rule 2: New array/object/map/set of size `n` is `O(n)`
```js
const temp = new Array(n); // O(n)
const map = new Map();     // up to O(n) entries
```

### Rule 3: Recursion uses stack space
- Depth `n` → `O(n)` space
- Depth `log n` → `O(log n)` space (balanced tree recursion)

Example:
```js
function f(n){
  if(n <= 0) return;
  f(n - 1); // recursion depth n
}
```
Space: `O(n)` (call stack)

### Rule 4: BFS / DFS space is about “visited structure”
- Graph BFS: queue + visited → `O(V)`
- Tree DFS recursion: stack height → `O(H)` (H=height)

---

## 4) Common Pattern Cheat‑Sheet (Time + Space)

### Single loop
```js
for (let i = 0; i < n; i++) {}
```
- Time: `O(n)`
- Space: `O(1)`

### Two pointers / sliding window
Usually:
- Time: `O(n)` (each pointer moves at most `n` times)
- Space: `O(1)` or `O(k)` / `O(n)` if using a map for frequency

### Hash map counting
```js
for (const x of arr) map.set(x, (map.get(x) || 0) + 1);
```
- Time: `O(n)` average
- Space: `O(n)` (map can store up to n keys)

### Sorting
- Time: typically `O(n log n)`
- Space: depends on implementation; in interviews, focus on time:
  - say: **Time `O(n log n)`**

### Binary search
- Time: `O(log n)`
- Space: `O(1)` iterative, `O(log n)` recursive stack

### Nested loops
- `O(n^2)` for two full loops
- `O(n^3)` for three full loops

### Graph traversal
- Time: `O(V + E)`
- Space: `O(V)` (visited + queue/stack)

---

## 5) How to answer in interviews (script)

When asked complexity, say:

1. “There is a loop over `n` elements → `O(n)`.”
2. “Inside, operations are constant time (or another loop) → …”
3. “So total time is …”
4. “Extra space is variables `O(1)` plus map/array/recursion stack up to …”

Example (cache/map):
> “One pass through `n` elements, each map operation is O(1) average, so time O(n).  
> The map can store up to n entries, so space O(n).”

---

## 6) Quick practice (with answers)

### Q1
```js
for (let i = 0; i < n; i++) {
  for (let j = i; j < n; j++) {}
}
```
- Time: `O(n^2)`
- Space: `O(1)`

### Q2
```js
function f(n){
  if(n<=1) return;
  f(n-1);
}
```
- Time: `O(n)`
- Space: `O(n)` (recursion depth)

### Q3 (sliding window pattern)
```js
let left = 0;
for (let right = 0; right < n; right++) {
  while (left < right && sum > k) left++;
}
```
- Time: `O(n)` (left moves at most n times total)
- Space: `O(1)`

---

## 7) Growth order (fast → slow)
`O(1) < O(log n) < O(n) < O(n log n) < O(n^2) < O(2^n) < O(n!)`

---

### Mini memory tricks
- **`log n` = number of halvings**
- **`n` = number of items**
- **Two pointers often = `O(n)`** (because each pointer moves forward only)

