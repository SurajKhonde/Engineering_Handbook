# Minimum Jumps to Reach End - Intuition

## Problem

Given:

```js
arr = [2, 3, 1, 1, 4]
```

Meaning:

```text
Index 0 -> can jump max 2 steps
Index 1 -> can jump max 3 steps
Index 2 -> can jump max 1 step
Index 3 -> can jump max 1 step
Index 4 -> can jump max 4 steps
```

Goal:

```text
Reach the last index using the minimum number of jumps.
```

---

# Main Idea

We start from index 0.

Instead of immediately deciding where to jump, we first check:

```text
How far can I go with my current jump?
```

We keep track of:

```js
currentEnd
```

Current jump range.

```js
farthest
```

Best future reach found so far.

```js
jumps
```

Number of jumps taken.

---

# Why `i + arr[i]`?

Suppose:

```js
index = 1
arr[1] = 3
```

Then:

```js
1 + 3 = 4
```

Meaning:

```text
If I stand at index 1,
I can reach up to index 4.
```

So:

```js
i + arr[i]
```

means:

```text
Maximum reachable index from position i.
```

---

# Dry Run

Input:

```js
arr = [2,3,1,1,4]
```

Initialize:

```js
jumps = 0
currentEnd = 0
farthest = 0
```

---

## i = 0

```js
farthest = Math.max(0, 0 + 2)
         = 2
```

Meaning:

```text
Best reachable position found so far is index 2.
```

Now:

```js
i === currentEnd
```

```js
0 === 0
```

True.

Meaning:

```text
Current range is finished.
Need to take a jump.
```

Update:

```js
jumps++
currentEnd = farthest
```

Result:

```js
jumps = 1
currentEnd = 2
```

Meaning:

```text
With 1 jump,
I can reach anywhere up to index 2.
```

---

## i = 1

```js
1 + arr[1]
= 1 + 3
= 4
```

Update:

```js
farthest = 4
```

Meaning:

```text
If I take another jump,
I can potentially reach index 4.
```

---

## i = 2

```js
2 + arr[2]
= 3
```

Not better than 4.

So:

```js
farthest = 4
```

Now:

```js
i === currentEnd
```

```js
2 === 2
```

True.

Meaning:

```text
Finished checking all positions reachable with 1 jump.
```

Take another jump:

```js
jumps++
currentEnd = farthest
```

Result:

```js
jumps = 2
currentEnd = 4
```

Since:

```js
currentEnd >= n - 1
```

We reached the end.

Answer:

```js
2
```

---

# Why Not Jump Immediately?

Wrong thinking:

```text
arr[0] = 2

So jump directly to index 2.
```

Problem:

```text
Index 1 may provide a much better future reach.
```

Instead:

```text
Check every position inside the current range.
Choose the one that gives the maximum future reach.
```

---

# What Happens If There Is a 0?

Example:

```js
arr = [2,3,1,0,4]
```

At:

```js
index = 3
arr[3] = 0
```

Meaning:

```text
Cannot move from index 3.
```

But that's okay if another path exists.

Example:

```js
index 1
```

can reach:

```js
1 + 3 = 4
```

So we can jump over the 0.

---

# When Do We Get Stuck?

Example:

```js
arr = [3,2,1,0,4]
```

Possible reaches:

```text
Index 1 -> 1 + 2 = 3
Index 2 -> 2 + 1 = 3
Index 3 -> 3 + 0 = 3
```

Maximum reachable index:

```text
3
```

But last index is:

```text
4
```

Impossible to reach.

Eventually:

```js
i >= farthest
```

becomes true.

Meaning:

```text
No new position is reachable.
We are stuck.
```

Return:

```js
-1
```

---

# Code

```js
function minJumpToReachEnd(arr) {
    let n = arr.length;

    if (n === 1) return 0;
    if (arr[0] === 0) return -1;

    let jumps = 0;
    let currentEnd = 0;
    let farthest = 0;

    for (let i = 0; i < n - 1; i++) {

        farthest = Math.max(farthest, i + arr[i]);

        if (i === currentEnd) {
            jumps++;
            currentEnd = farthest;

            if (currentEnd >= n - 1) {
                return jumps;
            }
        }

        if (i >= farthest) {
            return -1;
        }
    }

    return -1;
}
```

---

# Variables Summary

| Variable | Meaning |
|----------|----------|
| `jumps` | Number of jumps taken |
| `currentEnd` | Current jump range end |
| `farthest` | Furthest index reachable from current range |
| `i + arr[i]` | Furthest position reachable from index `i` |

---

# Time Complexity

```text
O(n)
```

Each element is visited once.

---

# Space Complexity

```text
O(1)
```

Only a few variables are used.

---

# Mental Model

Think of:

```js
currentEnd
```

as:

```text
Current fuel range.
```

and:

```js
farthest
```

as:

```text
Best fuel station discovered so far.
```

While driving within your current fuel range:

```text
Keep searching for the station
that allows the longest future travel.
```

When fuel runs out:

```text
Refuel at the best station found.
```

That is exactly how the greedy solution works.

### Dutch Alogorithm

```js
function dutchAlgorith(arr) {
    let low = 0;
    let mid = 0;
    let high = arr.length - 1;

    while (mid <= high) {
        if (arr[mid] === 0) {
            [arr[low], arr[mid]] = [arr[mid], arr[low]];
            low++;
            mid++;
        }
        else if (arr[mid] === 1) {
            mid++;
        }
        else {
            [arr[mid], arr[high]] = [arr[high], arr[mid]];
            high--;
        }
    }

    return arr;
}
```
## Set and Map work 
# JavaScript Set and Map - Complete CRUD Operations

# 1. Set

A `Set` stores only unique values.

```js
const mySet = new Set();
```

Example:

```js
const mySet = new Set([1, 2, 3]);

console.log(mySet);
```

Output:

```js
Set(3) {1, 2, 3}
```

---

# Set CRUD Operations

## C - Create

### Empty Set

```js
const mySet = new Set();
```

### Create With Values

```js
const mySet = new Set([1, 2, 3]);
```

---

## R - Read

### Check Value Exists

```js
mySet.has(2);
```

Output:

```js
true
```

---

### Get Size

```js
mySet.size;
```

Output:

```js
3
```

---

### Iterate

```js
for (let value of mySet) {
    console.log(value);
}
```

Output:

```js
1
2
3
```

---

### Convert To Array

```js
const arr = [...mySet];
```

or

```js
const arr = Array.from(mySet);
```

---

## U - Update

Set doesn't have an update method.

Remove and add again.

```js
mySet.delete(2);
mySet.add(20);
```

---

## D - Delete

### Delete Single Value

```js
mySet.delete(2);
```

Output:

```js
true
```

Set becomes:

```js
Set(2) {1, 3}
```

---

### Clear Entire Set

```js
mySet.clear();
```

Output:

```js
Set(0) {}
```

---

# Important Set Methods

## Add

```js
mySet.add(10);
```

---

## Duplicate Not Added

```js
mySet.add(10);
mySet.add(10);
mySet.add(10);
```

Output:

```js
Set(1) {10}
```

---

## Check Exists

```js
mySet.has(10);
```

---

## Remove

```js
mySet.delete(10);
```

---

## Clear

```js
mySet.clear();
```

---

# Set Use Cases

## Remove Duplicates

```js
const arr = [1,1,2,2,3,3];

const unique = [...new Set(arr)];

console.log(unique);
```

Output:

```js
[1,2,3]
```

---

## Fast Lookup

```js
const visited = new Set();

visited.add("A");

if (visited.has("A")) {
    console.log("Already visited");
}
```

---

# 2. Map

Map stores:

```text
Key -> Value
```

---

Example

```js
const map = new Map();
```

---

# Map CRUD Operations

## C - Create

### Empty Map

```js
const map = new Map();
```

---

### Create With Data

```js
const map = new Map([
    ["name", "Suraj"],
    ["age", 25]
]);
```

---

## R - Read

### Get Value

```js
map.get("name");
```

Output:

```js
"Suraj"
```

---

### Check Key Exists

```js
map.has("name");
```

Output:

```js
true
```

---

### Size

```js
map.size;
```

---

### Iterate

```js
for (let [key, value] of map) {
    console.log(key, value);
}
```

Output:

```js
name Suraj
age 25
```

---

## U - Update

Use set()

```js
map.set("age", 26);
```

Output:

```js
Map {
  "name" => "Suraj",
  "age" => 26
}
```

---

## D - Delete

### Delete One Key

```js
map.delete("age");
```

---

### Clear Map

```js
map.clear();
```

---

# Important Map Methods

## Insert

```js
map.set("name", "Suraj");
```

---

## Read

```js
map.get("name");
```

---

## Exists

```js
map.has("name");
```

---

## Remove

```js
map.delete("name");
```

---

## Clear

```js
map.clear();
```

---

# Frequency Counter Pattern

Most common interview usage.

```js
function countFrequency(arr) {
    const map = new Map();

    for (let num of arr) {
        map.set(num, (map.get(num) || 0) + 1);
    }

    return map;
}

console.log(countFrequency([1,1,2,3,2,1]));
```

Output:

```js
Map {
  1 => 3,
  2 => 2,
  3 => 1
}
```

---

# Object vs Map

## Object

```js
const obj = {};

obj.name = "Suraj";
```

---

## Map

```js
const map = new Map();

map.set("name", "Suraj");
```

---

### Advantages of Map

```text
1. Any type can be a key
2. Better iteration support
3. Cleaner API
4. Size property available
```

---

# Interview Cheat Sheet

## Set

```js
const set = new Set();

set.add(value);
set.has(value);
set.delete(value);
set.clear();
set.size;
```

---

## Map

```js
const map = new Map();

map.set(key, value);
map.get(key);
map.has(key);
map.delete(key);
map.clear();
map.size;
```

---

# When To Use Set

```text
1. Remove duplicates
2. Fast lookup
3. Visited nodes in Graph
4. Unique values only
```

---

# When To Use Map

```text
1. Frequency counting
2. Key-value storage
3. Caching
4. Prefix Sum problems
5. Majority Element
6. Two Sum
7. Grouping data
```

# Time Complexity

| Operation | Set | Map |
|------------|------|------|
| Add/Set | O(1) | O(1) |
| Delete | O(1) | O(1) |
| Has | O(1) | O(1) |
| Get | - | O(1) |
| Size | O(1) | O(1) |
```js
function majorityElement(arr) {
    let n = arr.length;
    let calMap = new Map();

    for (let i = 0; i < n; i++) {
        if (!calMap.has(arr[i])) {
            calMap.set(arr[i], 1);
        } else {
            calMap.set(arr[i], calMap.get(arr[i]) + 1);
        }
    }

    for (let [key, value] of calMap) {
        if (value > n / 2) {
            return key;
        }
    }

    return -1;
}

console.log(majorityElement([1, 1, 2, 1, 3, 5, 1])); // 1
```