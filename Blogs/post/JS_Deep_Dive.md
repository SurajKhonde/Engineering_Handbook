# JavaScript Deep Dive - Closures, Counters & Caching

## Part 1: Closures & Counter Patterns

### Pattern 1: Function-Based Counter (Closure)

**Concept:** Function returns object with methods that access outer function's variables.

```javascript
function userInfo() {
  let count = 0; // Closure variable - inaccessible directly
  
  return {
    incrementCount() {
      count++;
    },
    
    decrementCount() {
      if (count > 0) count--; // Prevent negative
    },
    
    getCount() {
      return count;
    }
  };
}

const user = userInfo();
user.incrementCount();
user.incrementCount();
console.log(user.getCount()); // 2

user.decrementCount();
console.log(user.getCount()); // 1
```

**Why This Works:**
- `count` is private (can't access `user.count` directly)
- Closure preserves `count` in memory
- Each call to `userInfo()` creates new independent closure

**Interview Q:** "How is `count` protected?"
**Answer:** "Through closure. The `count` variable lives in userInfo's scope. Only the returned object's methods can access it. This creates data privacy."

---

### Pattern 2: Object-Based Counter (this binding)

```javascript
let counter = {
  count: 0,
  
  increment() {
    this.count++;
  },
  
  decrement() {
    if (this.count > 0) this.count--;
  },
  
  getCount() {
    return this.count;
  }
};

counter.increment();
counter.increment();
console.log(counter.getCount()); // 2
```

**Key Difference from Pattern 1:**
- Properties are PUBLIC (accessible as `counter.count`)
- Uses `this` binding
- Simpler, but less encapsulation

**Use When:** You don't need privacy

---

### Pattern 3: Class-Based Counter (Private Fields)

```javascript
class CounterFunction {
  #count = 0; // Private field (can't access directly)
  
  incrementCounter() {
    this.#count++;
  }
  
  decrementCounter() {
    if (this.#count > 0) this.#count--; // FIXED: was "= 0"
  }
  
  getCount() {
    return this.#count;
  }
}

let newCounter = new CounterFunction();
newCounter.incrementCounter();
newCounter.incrementCounter();
newCounter.incrementCounter();
console.log(newCounter.getCount()); // 3
```

**What's Different:**
- `#count` is a private field (ES2022 feature)
- Can't access `newCounter.#count` outside the class
- Modern way to create private properties

**Bug in Original Code:**
```javascript
// WRONG:
decrementCounter() {
  if(this.#count > 0) this.#count = 0; // Sets to 0, doesn't decrement!
}

// CORRECT:
decrementCounter() {
  if(this.#count > 0) this.#count--;
}
```

**Interview Q:** "What's the difference between Pattern 1 and Pattern 3?"
**Answer:** "Pattern 1 uses closures (function scopes). Pattern 3 uses private fields (#). Both protect data privacy. Pattern 1 is older but works everywhere. Pattern 3 is modern and syntactically cleaner."

---

## Part 2: One-Time Execution

### Problem
Function should run only once, even if called multiple times.

### Solution

```javascript
function once(fn) {
  let called = false;
  let result;
  
  return function (...args) {
    if (!called) {
      called = true;
      result = fn.apply(this, args);
    }
    return result;
  };
}

// Usage
const expensiveInit = once(() => {
  console.log("Initializing...");
  return "initialized";
});

console.log(expensiveInit()); // "Initializing..." → "initialized"
console.log(expensiveInit()); // (nothing logged) → "initialized"
console.log(expensiveInit()); // (nothing logged) → "initialized"
```

**How It Works:**
1. `called` tracks if function ran
2. First call: Sets `called = true`, saves `result`, runs function
3. Subsequent calls: Skip execution, return cached `result`

**Real-World Use:**
- Analytics initialization (send only once)
- Feature flag checks
- Resource initialization

**Interview Q:** "Why use `apply(this, args)`?"
**Answer:** "To preserve the original function's context (this) and pass all arguments correctly."

---

## Part 3: Caching

### Basic Cache Pattern

```javascript
function cache() {
  let storage = {}; // Private cache
  
  return {
    addCache(key, value) {
      storage[key] = value;
    },
    
    getCache(key) {
      return storage.hasOwnProperty(key) 
        ? storage[key] 
        : "key not in cache";
    },
    
    removeCache(key) {
      if (key in storage) {
        delete storage[key];
        return true;
      }
      return false;
    }
  };
}

let userCache = cache();
userCache.addCache("name", "raj");
console.log(userCache.getCache("name")); // "raj"

userCache.removeCache("name");
console.log(userCache.getCache("name")); // "key not in cache"
```

**Key Concepts:**
- `storage` is private (closure)
- `delete` removes key from object
- `hasOwnProperty()` checks existence safely

**Interview Q:** "Why use `delete` instead of setting to null?"
**Answer:** "delete removes the property completely. Setting to null leaves the property, just with null value. delete ensures the key doesn't exist anymore."

---

## Part 4: LRU Cache (Most Important)

### What is LRU?
**LRU = Least Recently Used**

Cache evicts oldest accessed item when full.

```javascript
function LRUCache(limit = 3) {
  let cache = new Map();
  
  function get(key) {
    if (!cache.has(key)) return "Not found";
    
    // Move to most recently used (delete + re-add)
    let value = cache.get(key);
    cache.delete(key);
    cache.set(key, value);
    
    return value;
  }
  
  function put(key, value) {
    // If key exists, remove it first
    if (cache.has(key)) {
      cache.delete(key);
    }
    
    // Add/update key
    cache.set(key, value);
    
    // Evict LRU if over limit
    if (cache.size > limit) {
      // First key in Map = LRU (oldest)
      let lruKey = cache.keys().next().value;
      cache.delete(lruKey);
    }
  }
  
  function show() {
    return [...cache.entries()]; // Return all entries
  }
  
  return { get, put, show };
}

// Usage
const lru = LRUCache(3);

lru.put("a", 1);
lru.put("b", 2);
lru.put("c", 3);
console.log(lru.show()); // [["a", 1], ["b", 2], ["c", 3]]

lru.put("d", 4); // Evicts "a" (oldest)
console.log(lru.show()); // [["b", 2], ["c", 3], ["d", 4]]

lru.get("b"); // "b" becomes most recent
lru.put("e", 5); // Evicts "c" (now oldest)
console.log(lru.show()); // [["d", 4], ["b", 2], ["e", 5]]
```

### Why Map > Object?
- **Map** maintains insertion order
- **Object** order is unreliable
- **Map** has `.delete()`, `.has()` methods

### How It Works

**Step 1: Access Item**
```
Before: [a, b, c] (a is oldest)
Get b
After: [a, c, b] (b moved to end = most recent)
```

**Step 2: Add When Full**
```
Before: [a, c, b] (limit = 3)
Add d
Delete a (first = LRU)
After: [c, b, d]
```

### Real-World Example

```javascript
const pageCache = LRUCache(5); // Keep 5 pages in memory

// User visits pages
pageCache.put(1, "Home page data");
pageCache.put(2, "About page data");
pageCache.put(3, "Contact page data");

// User navigates to page 1 again (moves to recent)
let data = pageCache.get(1);

// New pages added
pageCache.put(4, "Products page");
pageCache.put(5, "Services page");
pageCache.put(6, "Blog page"); // Evicts page 2 (least recent)
```

### Interview Questions

**Q: Why do we delete then re-add in `get()`?**
A: "Map maintains insertion order. Deleting and re-adding moves the key to the end, marking it as most recently used."

**Q: How is LRU different from FIFO cache?**
A: "FIFO evicts oldest added. LRU evicts oldest accessed. LRU is smarter - frequently used items stay."

**Q: What's time complexity?**
A: "O(1) for all operations (get, put, delete) because Map is hash-based."

---

## Quick Comparison Table

| Pattern | Encapsulation | Privacy | Modern | Use Case |
|---------|---|---|---|---|
| Function Closure | ✓ | ✓ | ✓ | Node.js, works everywhere |
| Object | ✗ | ✗ | - | Simple data containers |
| Class Private | ✓ | ✓ | ✓ | React, modern JS |

---

## Interview Checklist

✓ Closures and data privacy
✓ Counter patterns (all 3 types)
✓ One-time execution pattern
✓ Basic caching
✓ LRU cache (most important!)
✓ Why Map > Object
✓ Time complexity O(1)
✓ Eviction strategy

---

## Code You Should Know

```javascript
// Closure pattern
function makeCounter() {
  let count = 0;
  return {
    inc: () => count++,
    get: () => count
  };
}

// One-time execution
function once(fn) {
  let called = false, result;
  return (...args) => !called 
    ? (called = true, result = fn(...args)) 
    : result;
}

// LRU Cache
function LRUCache(limit) {
  let map = new Map();
  return {
    get: (key) => {
      if (!map.has(key)) return null;
      let val = map.get(key);
      map.delete(key);
      map.set(key, val);
      return val;
    },
    put: (key, val) => {
      map.has(key) && map.delete(key);
      map.set(key, val);
      map.size > limit && map.delete(map.keys().next().value);
    }
  };
}
```

---

## Key Insights

1. **Closure = Privacy** - Variables in closure are protected
2. **Map > Object** - Map maintains order, has better methods
3. **LRU is Essential** - Appears in interviews, caching, browser engines
4. **Delete > Null** - delete removes property completely
5. **One-time = State** - Use closure variable to track execution

**Remember:** "Less noise, more action." Master these patterns and you're 80% ready for JavaScript interviews.
