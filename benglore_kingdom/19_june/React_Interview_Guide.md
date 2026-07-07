# React Interview Preparation Guide
## From Basic to Senior Level

---

## SECTION 1: REACT FUNDAMENTALS

### BASIC LEVEL

#### 1. What is Virtual DOM?

**Simple Explanation:**
- React doesn't directly update the browser's DOM (which is slow)
- Instead, React creates a "copy" of the DOM in memory = Virtual DOM
- Virtual DOM is faster because it's just JavaScript objects
- When state changes, React updates Virtual DOM first, then updates real DOM

**Analogy to Remember:**
```
Real DOM = Your actual house (slow to change)
Virtual DOM = Your blueprint (fast to change)

React updates blueprint → compares → changes only what's different in house
```

**Key Point:** Virtual DOM = JavaScript object representation of real DOM

---

#### 2. How does React update the UI?

**4 Steps React Uses:**

1. **State/Props Change** → Something triggers an update
2. **Virtual DOM Update** → React creates new Virtual DOM
3. **Reconciliation** → React compares old vs new Virtual DOM
4. **Commit Phase** → React updates real DOM with only changed parts

**Remember:** React = State Change → Virtual DOM → Compare → Update Real DOM

---

#### 3. What is Reconciliation?

**Simple Definition:**
Reconciliation = Process of comparing old Virtual DOM with new Virtual DOM

**What it does:**
- Finds what changed
- Figures out minimal changes needed
- Passes changes to real DOM

**Analogy:**
```
Old Document: "Hello World"
New Document: "Hello React World"

Reconciliation: 
- Detects: "React " was added
- Only updates that part (not entire string)
```

---

#### 4. What is Diffing?

**Diffing = The actual comparison algorithm**

React uses 2 rules:
1. **Type-based:** If component type changes (div → p), rebuild entire tree
2. **Key-based:** Elements with same key are same element

**Example:**
```jsx
// Old
<div>
  <Child key="1" name="Ali" />
  <Child key="2" name="Sara" />
</div>

// New
<div>
  <Child key="2" name="Sara" />
  <Child key="1" name="Ali" />
</div>

// Diffing result: Just reorders, doesn't recreate (because keys help identify)
```

---

#### 5. Why are Keys Important?

**Problem without keys:**
```jsx
// Without keys - React gets confused
{items.map((item) => (
  <li>{item.name}</li>  // ❌ No key = confusion
))}

// List reordered → React rebuilds everything
// Form inputs lose focus (bad user experience)
```

**Solution with keys:**
```jsx
// With keys - React knows which is which
{items.map((item) => (
  <li key={item.id}>{item.name}</li>  // ✅ Key helps identify
))}

// List reordered → React just moves elements (keeps state)
```

**Why?** Keys tell React: "This is the same element, just moved"

**⚠️ Never use:**
- Index as key (causes bugs when list changes)
- Random numbers as key

**Use:** Unique ID from data (item.id)

---

### MID LEVEL

#### 1. What causes a component to re-render?

**5 Reasons Component Re-renders:**

1. **State changes** (useState updates)
   ```jsx
   const [count, setCount] = useState(0);
   setCount(count + 1); // ← Triggers re-render
   ```

2. **Props change** (parent passes new props)
   ```jsx
   // Parent
   <Child name={newName} /> // ← Child re-renders
   ```

3. **Parent re-renders** (child always re-renders when parent does)
   ```jsx
   // Parent updates → ALL children re-render
   ```

4. **Context value changes**
   ```jsx
   <ThemeContext.Provider value={newTheme}>
     {/* ← All consumers re-render */}
   </ThemeContext.Provider>
   ```

5. **This (class) changes**
   ```jsx
   this.setState({...}) // ← Re-renders class component
   ```

---

#### 2. Does parent re-render always re-render child?

**Answer:** YES, by default

**Example:**
```jsx
function Parent() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>
        Parent: {count}
      </button>
      <Child /> {/* ← Re-renders even if props didn't change */}
    </div>
  );
}
```

**Problem:** Unnecessary re-renders = Performance waste

**Solution:** Use React.memo (prevent unnecessary re-renders)
```jsx
const Child = React.memo(() => {
  return <div>Child</div>;
});
```

**Important:** React.memo only stops re-render if props are same

---

#### 3. How does React compare old and new trees?

**React's Comparison Rules:**

1. **Element Type Check**
   ```jsx
   // Different types → Rebuild
   <div>Hello</div>  vs  <span>Hello</span> // Rebuilds entire tree
   
   // Same type → Compare props/children
   <div>Hello</div>  vs  <div>Hello React</div> // Just updates text
   ```

2. **Key Check** (for lists)
   ```jsx
   // With same key → Same element (just reorder/update)
   // Different key → Different element (rebuild)
   ```

3. **Props/State Check**
   ```jsx
   // Old props: {name: "Ali", age: 25}
   // New props: {name: "Ali", age: 26}
   // Result: Only age property updated
   ```

---

#### 4. What is the Commit Phase?

**Timeline of Render:**

```
1. RENDER PHASE (safe, can pause)
   ↓
   React creates Virtual DOM
   React calculates changes
   (Can be interrupted)
   
2. COMMIT PHASE (unsafe, happens fast)
   ↓
   React applies changes to real DOM
   useEffect runs
   Browser paints screen
   (Cannot interrupt - must finish)
```

**Commit Phase Details:**
- Updates real DOM
- Updates refs
- Runs `useEffect` cleanup + effect
- Updates class component lifecycle methods
- Performs browser repaints

**Remember:** Commit = Point of no return (changes become visible)

---

### SENIOR LEVEL

#### 1. React app is slow. How do you investigate?

**STEP-BY-STEP INVESTIGATION METHOD:**

**Step 1: Identify Problem Area**
```
Open Chrome DevTools → Performance tab → Record
Do the slow action
Stop recording
Look for long yellow/red bars (slow tasks)
```

**Step 2: Check Bundle Size**
```bash
# Build your app
npm run build

# Check bundle size
npm install -g source-map-explorer
source-map-explorer 'build/js/*'
```
- Is bundle too large? (> 250KB = problem)
- Are unnecessary libraries included?

**Step 3: Check Network**
```
DevTools → Network tab
- API calls slow?
- Check response time
- Are images optimized?
```

**Step 4: Check Re-renders**
```
Use React DevTools Profiler
- Which components render most?
- Are re-renders necessary?
- Parent causing child unnecessary re-renders?
```

**Step 5: Memory Check**
```
DevTools → Memory tab
- Memory growing constantly? (Memory leak)
- useEffect not cleaning up?
- Event listeners not removed?
```

**Common Slow Reasons:**
- Large bundle size
- Too many API calls
- Unnecessary re-renders
- Memory leaks
- Unoptimized images
- Slow database queries

---

#### 2. How do you identify unnecessary re-renders?

**Method 1: React DevTools Profiler**

```
1. Open React DevTools
2. Go to "Profiler" tab
3. Click record
4. Do the action
5. See which components render
6. Look for "Why did this render?" section
```

**Method 2: Use why-did-you-render Library**
```jsx
import whyDidYouRender from '@welldone-software/why-did-you-render';

whyDidYouRender(React, {
  trackAllPureComponents: true,
});
```

**Method 3: Add console.log (Quick check)**
```jsx
function Child({ name }) {
  console.log('Child rendered with:', name);
  return <div>{name}</div>;
}

// If logs appear without name changing = unnecessary re-render
```

**Method 4: Check Dependency Arrays**
```jsx
// ❌ Wrong - useEffect runs every render
useEffect(() => {
  fetchData();
}); // No dependency array

// ✅ Right - runs only once
useEffect(() => {
  fetchData();
}, []); // Empty array = once on mount
```

---

#### 3. What tools do you use? (React DevTools Profiler)

**React DevTools Profiler Features:**

1. **See Render Time**
   - Which components took longest to render
   - In milliseconds

2. **Rank Components**
   - "Ranked chart" shows slowest first
   - Easy to spot problem components

3. **Flame Graph**
   - Shows tree structure
   - Which parent → child renders
   - Identifies long chains

4. **Component Props**
   - See what props component received
   - Helps identify unnecessary changes

5. **Why Did This Render?**
   - Shows if props changed
   - Shows if state changed
   - Shows if parent re-rendered

**How to Use:**
```
1. Open React DevTools (Chrome extension)
2. Click "Profiler" tab
3. Red circle = record
4. Interact with app
5. Red circle again = stop
6. Analyze results
```

**Metrics to Check:**
- Render duration (should be < 16ms for 60 FPS)
- Component count
- Re-render reasons

---

## SECTION 2: HOOKS

### BASIC LEVEL

#### 1. Difference between useState and useRef?

| Feature | useState | useRef |
|---------|----------|---------|
| Purpose | Store component state | Access DOM/store value |
| Re-render? | YES (updates trigger re-render) | NO (updates don't trigger re-render) |
| Returns | [value, setValue] | {current: value} |
| When to use | Display data, form inputs | Focus input, store timer ID |

**useState Example:**
```jsx
const [count, setCount] = useState(0);

// When count changes → component re-renders
return (
  <div>
    <p>{count}</p> {/* Updates on screen */}
    <button onClick={() => setCount(count + 1)}>+</button>
  </div>
);
```

**useRef Example:**
```jsx
const inputRef = useRef(null);

const focusInput = () => {
  inputRef.current.focus();
};

return (
  <>
    <input ref={inputRef} /> {/* No re-render when accessed */}
    <button onClick={focusInput}>Focus</button>
  </>
);
```

**Key Difference:**
- **useState** = "I need to show this on screen"
- **useRef** = "I need to access this, not show it"

---

#### 2. When does useEffect run?

**useEffect Timing:**

```jsx
// 1. No dependency array = Every render
useEffect(() => {
  console.log('Runs after EVERY render');
});

// 2. Empty array = Only once (mount)
useEffect(() => {
  console.log('Runs ONCE when component mounts');
}, []);

// 3. With dependencies = When dependency changes
useEffect(() => {
  console.log('Runs when count or name changes');
}, [count, name]);
```

**Timeline:**
```
1. Component renders
2. Browser paints screen
3. useEffect runs (AFTER paint)
4. Next render starts
5. useEffect cleanup runs (BEFORE new effect)
```

**Example:**
```jsx
useEffect(() => {
  console.log('Effect ran');
  
  return () => {
    console.log('Cleanup ran (before next effect or unmount)');
  };
}, []);

// Output:
// "Effect ran" (mount)
// ... (component used)
// "Cleanup ran" (unmount)
```

---

#### 3. Why dependency array?

**Without Dependency Array = Problem:**
```jsx
function Component() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    const timer = setInterval(() => {
      setCount(count + 1); // ❌ Always uses old count
    }, 1000);
  }); // No dependency array = runs every render
  
  // Problem: Every render creates new interval
  // Result: Memory leak + wrong counting
}
```

**With Dependency Array = Solution:**
```jsx
function Component() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    const timer = setInterval(() => {
      setCount(c => c + 1); // ✅ Uses latest count
    }, 1000);
    
    return () => clearInterval(timer); // Cleanup
  }, []); // Runs only once
  
  // Result: Single interval, correct counting, no leak
}
```

**Why Dependency Array?**
- Tells React: "Re-run effect only when these change"
- Without it = Effect runs every time (wasting resources)
- With it = Effect runs only when needed

**Rule:** If variable changes, add to dependency array

---

### MID LEVEL

#### 1. useMemo vs useCallback?

| Feature | useMemo | useCallback |
|---------|---------|-------------|
| What it memoizes | Computed value | Function |
| Returns | Memoized value | Memoized function |
| Use case | Expensive calculations | Pass function to child |
| Cost | Runs calculation, stores result | Stores function reference |

**useMemo Example:**
```jsx
function Parent() {
  const [count, setCount] = useState(0);
  const [dark, setDark] = useState(false);
  
  // Expensive calculation
  const expensiveValue = useMemo(() => {
    console.log('Calculating...');
    return count * 2; // Only recalculates if count changes
  }, [count]); // Dependency: count
  
  return (
    <div>
      <p>Value: {expensiveValue}</p>
      <button onClick={() => setDark(!dark)}>Toggle Dark</button>
      {/* Dark toggle doesn't cause expensive calculation */}
    </div>
  );
}
```

**useCallback Example:**
```jsx
function Parent() {
  const [count, setCount] = useState(0);
  
  // Memoized function
  const handleClick = useCallback(() => {
    console.log('Clicked');
    setCount(count + 1);
  }, [count]); // Dependency: count
  
  return (
    <Child onClickFunction={handleClick} />
  );
}

const Child = React.memo(({ onClickFunction }) => {
  console.log('Child rendered');
  return <button onClick={onClickFunction}>Click</button>;
});

// Without useCallback: Child re-renders every time parent renders
// With useCallback: Child only re-renders when handleClick changes
```

**Quick Rule:**
- useMemo = Memoize **value**
- useCallback = Memoize **function**

---

#### 2. When should you NOT use useMemo?

**❌ DON'T use useMemo when:**

1. **Value is primitive or simple**
   ```jsx
   // ❌ Wrong
   const doubled = useMemo(() => count * 2, [count]);
   
   // ✅ Right (simple calculation)
   const doubled = count * 2;
   ```

2. **Calculating is faster than memoizing**
   ```jsx
   // ❌ useMemo overhead > calculation cost
   const result = useMemo(() => array.sort(), [array]);
   
   // ✅ Just calculate
   const result = array.sort();
   ```

3. **Dependency array would be everything anyway**
   ```jsx
   // ❌ No benefit
   const value = useMemo(() => {
     return a + b + c + d + e;
   }, [a, b, c, d, e]); // Changes frequently
   
   // ✅ Just calculate
   const value = a + b + c + d + e;
   ```

4. **Component doesn't have performance issue**
   ```jsx
   // Don't optimize before problem exists
   // Test first, optimize only if slow
   ```

**Golden Rule:** Only use memoization if you've proven it's slow

---

#### 3. Why infinite loops happen in useEffect?

**Infinite Loop Patterns:**

**Pattern 1: Missing Dependency Array**
```jsx
// ❌ Infinite loop
useEffect(() => {
  setCount(count + 1); // Triggers re-render → effect runs again
}); // No dependency array = runs every render
```

**Pattern 2: State in Dependency (Self-referencing)**
```jsx
// ❌ Infinite loop
const [count, setCount] = useState(0);

useEffect(() => {
  setCount(count + 1); // Sets count
}, [count]); // Depends on count → effect runs → count changes → loop
```

**Pattern 3: Object as Dependency**
```jsx
// ❌ Infinite loop
const [data, setData] = useState({});

useEffect(() => {
  const newData = { ...data, updated: true };
  setData(newData); // Creates new object
}, [data]); // data changed → effect runs → object created again
```

**Fix 1: Empty Dependency Array**
```jsx
// ✅ Correct
useEffect(() => {
  setCount(count + 1);
}, []); // Only runs once on mount
```

**Fix 2: Use Functional Update**
```jsx
// ✅ Correct
useEffect(() => {
  setCount(prev => prev + 1);
}, []); // Doesn't depend on count
```

**Fix 3: Move Object Outside**
```jsx
// ✅ Correct
const initialData = { updated: false };

useEffect(() => {
  setData(initialData);
}, []); // initialData never changes
```

---

### SENIOR LEVEL

#### 1. Explain stale closure problem

**What is Stale Closure?**

When a function captures a variable from an outer scope, but the variable's value has changed since the function was created.

**Example:**
```jsx
function Component() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    const timer = setInterval(() => {
      console.log(count); // ← Always logs 0 (stale)
      setCount(count + 1); // ← Always sets count to 1
    }, 1000);
    
    return () => clearInterval(timer);
  }, []); // ← Missing count in dependencies
  
  // Closure captured count = 0
  // But count actually changes
  // Effect sees old value = stale closure
}
```

**Result:** Timer always adds 1, doesn't increment properly

**Solution 1: Add to Dependency Array**
```jsx
useEffect(() => {
  const timer = setInterval(() => {
    console.log(count); // ← Always logs current count
    setCount(count + 1);
  }, 1000);
  
  return () => clearInterval(timer);
}, [count]); // ← Add to dependencies
```

**Problem:** Effect runs every second (too often)

**Solution 2: Use Functional Update**
```jsx
useEffect(() => {
  const timer = setInterval(() => {
    setCount(prev => prev + 1); // ← Uses current value
  }, 1000);
  
  return () => clearInterval(timer);
}, []); // ← Still empty, but no stale closure
```

**Why Solution 2 Works:**
- Functional update `(prev) => prev + 1` always uses latest state
- No dependency on count needed
- Effect runs only once

**Key Concept:** Stale closure = Function sees old value of variable

---

#### 2. How do you cancel API calls in useEffect?

**Problem: Memory Leak**
```jsx
useEffect(() => {
  fetch('/api/data')
    .then(res => res.json())
    .then(data => setData(data)); // ← Problem: component may unmount
    // But this still tries to setState
    // React warning: "Memory leak"
}, []);
```

**Solution 1: AbortController (Modern)**
```jsx
useEffect(() => {
  const controller = new AbortController();
  
  fetch('/api/data', {
    signal: controller.signal // ← Pass signal
  })
    .then(res => res.json())
    .then(data => setData(data))
    .catch(error => {
      if (error.name !== 'AbortError') {
        console.error(error);
      }
    });
  
  return () => {
    controller.abort(); // ← Cancel request on cleanup
  };
}, []);
```

**Solution 2: Cancel Flag**
```jsx
useEffect(() => {
  let isMounted = true; // Flag
  
  fetch('/api/data')
    .then(res => res.json())
    .then(data => {
      if (isMounted) { // ← Check flag
        setData(data);
      }
    });
  
  return () => {
    isMounted = false; // ← Cleanup
  };
}, []);
```

**Solution 3: Axios with cancelToken (Deprecated)**
```jsx
useEffect(() => {
  const source = axios.CancelToken.source();
  
  axios.get('/api/data', {
    cancelToken: source.token
  }).then(res => setData(res.data));
  
  return () => {
    source.cancel('Component unmounted');
  };
}, []);
```

**Best Practice:** Use AbortController (native, no library needed)

---

#### 3. How do you avoid race conditions?

**What is Race Condition?**

When multiple async calls run at the same time, and responses arrive in different order than requests.

**Example Problem:**
```jsx
function SearchUsers({ query }) {
  const [results, setResults] = useState(null);
  
  useEffect(() => {
    // Request 1: Search "Ali"
    fetch(`/api/search?q=${query}`)
      .then(res => res.json())
      .then(data => setResults(data)); // Sets "Ali" results
  }, [query]);
  
  // User types fast: "Ali" → "Alice" → "Alison"
  // Request 1 (Ali) - slow response
  // Request 2 (Alice) - fast response
  // Request 3 (Alison) - fast response
  
  // Responses arrive: 2 → 3 → 1
  // Final result shows "Ali" instead of "Alison" = WRONG
}
```

**Solution 1: AbortController**
```jsx
useEffect(() => {
  const controller = new AbortController();
  
  fetch(`/api/search?q=${query}`, {
    signal: controller.signal
  })
    .then(res => res.json())
    .then(data => setResults(data));
  
  return () => {
    controller.abort(); // Cancel previous request
  };
}, [query]); // New query = cancel old search
```

**Solution 2: Ignore Late Responses**
```jsx
useEffect(() => {
  let latestQuery = query;
  
  fetch(`/api/search?q=${query}`)
    .then(res => res.json())
    .then(data => {
      if (latestQuery === query) { // Only use if still current
        setResults(data);
      }
    });
  
  return () => {
    latestQuery = null; // Mark as ignored
  };
}, [query]);
```

**Solution 3: Track Request Number**
```jsx
useEffect(() => {
  let requestNumber = 1;
  const currentRequest = requestNumber;
  
  fetch(`/api/search?q=${query}`)
    .then(res => res.json())
    .then(data => {
      if (currentRequest === requestNumber) {
        setResults(data);
      }
    });
}, [query]);
```

**Best Practice:** Use AbortController (cleanest, most modern)

---

## SECTION 3: STATE MANAGEMENT

### BASIC LEVEL

#### 1. Why Redux?

**Problem Redux Solves:**

```jsx
// Without Redux - Prop Drilling Problem
function App() {
  const [user, setUser] = useState({ name: 'Ali' });
  
  return <Parent user={user} setUser={setUser} />;
}

function Parent({ user, setUser }) {
  return <Child user={user} setUser={setUser} />;
}

function Child({ user, setUser }) {
  return <GrandChild user={user} setUser={setUser} />;
}

// Props passed through every level = Prop Drilling
// Hard to maintain, confusing
```

**Redux Solution:**

```jsx
// Global state store (accessible from anywhere)
const store = {
  user: { name: 'Ali' }
};

function GrandChild() {
  const user = useSelector(state => state.user); // Access directly
  const dispatch = useDispatch();
  
  return (
    <div>
      {user.name}
      <button onClick={() => dispatch(updateUser({ name: 'Sara' }))}>
        Update
      </button>
    </div>
  );
}
```

**When to Use Redux:**
- Complex app with many state updates
- Deeply nested components need same data
- Multiple unrelated components share state
- State management is complex

**When NOT to use Redux:**
- Small apps (use useState)
- Simple state (use Context)
- Single parent-child communication

---

#### 2. Redux vs Context API?

| Feature | Redux | Context API |
|---------|-------|------------|
| Learning curve | Steep | Gentle |
| Setup | Verbose (actions, reducers, store) | Simple (useContext, useState) |
| Performance | Optimized (selective updates) | Can cause unnecessary re-renders |
| DevTools | Excellent (Redux DevTools) | Basic |
| Middleware | Yes (for async) | No |
| Good for | Large apps | Small to medium apps |
| State | Immutable | Any |

**Redux - Good For:**
```jsx
// Large app with complex state
{
  user: { name, email, role },
  posts: [{ id, title, body }],
  ui: { theme, sidebar, notifications },
  api: { loading, error }
}
```

**Context - Good For:**
```jsx
// Small app or simple state
const ThemeContext = createContext();
const UserContext = createContext();
```

**Rule of Thumb:**
- Start with Context
- Move to Redux when Context becomes hard to manage

---

### MID LEVEL

#### 1. What problem does RTK solve?

**RTK = Redux Toolkit**

**Problem with Plain Redux:**
```jsx
// PLAIN REDUX (Lots of boilerplate)

// 1. Actions
const SET_USER = 'SET_USER';
const setUser = (user) => ({ type: SET_USER, payload: user });

// 2. Reducer
function userReducer(state = {}, action) {
  switch(action.type) {
    case SET_USER:
      return { ...state, ...action.payload };
    default:
      return state;
  }
}

// 3. Store
const store = createStore(userReducer);

// Way too much code for simple state update
```

**RTK Solution:**
```jsx
// REDUX TOOLKIT (Clean and simple)

const userSlice = createSlice({
  name: 'user',
  initialState: {},
  reducers: {
    setUser: (state, action) => {
      state.name = action.payload.name; // Immer handles immutability
    }
  }
});

export const { setUser } = userSlice.actions;
export default userSlice.reducer;

// Much less code, same result
```

**RTK Benefits:**
1. **Less boilerplate** - No need for separate action creators
2. **Immer built-in** - Can write mutable code, Immer makes it immutable
3. **DevTools included** - Redux DevTools integrated
4. **Best practices by default** - Follows Redux best practices automatically

---

#### 2. What is createSlice?

**createSlice = Combines actions + reducer + initial state**

```jsx
const todoSlice = createSlice({
  name: 'todos',
  initialState: {
    items: [],
    loading: false
  },
  reducers: {
    // Synchronous actions
    addTodo: (state, action) => {
      state.items.push(action.payload);
    },
    removeTodo: (state, action) => {
      state.items = state.items.filter(
        todo => todo.id !== action.payload
      );
    },
    setLoading: (state, action) => {
      state.loading = action.payload;
    }
  },
  extraReducers: (builder) => {
    // Async actions (handled by thunks)
    builder
      .addCase(fetchTodos.pending, (state) => {
        state.loading = true;
      })
      .addCase(fetchTodos.fulfilled, (state, action) => {
        state.items = action.payload;
        state.loading = false;
      });
  }
});

export const { addTodo, removeTodo, setLoading } = todoSlice.actions;
export default todoSlice.reducer;
```

**What it gives you:**
- Action creators (addTodo, removeTodo, setLoading)
- Reducer function (automatically handles all actions)
- Initial state

---

#### 3. What is middleware?

**Middleware = Function that runs between action and reducer**

```
Dispatch Action → Middleware → Reducer → Store
```

**Common middleware:**
- Logging
- API calls (async actions)
- Error handling
- State persistence

**Redux Thunk (Most common):**
```jsx
// Middleware that lets you dispatch functions

// Without Thunk (doesn't work)
dispatch(fetchUser); // ❌ Can't dispatch function

// With Thunk (works)
dispatch(fetchUser); // ✅ Returns function
// Thunk receives dispatch, getState
```

**Example:**
```jsx
// Create async action using createAsyncThunk
const fetchUser = createAsyncThunk(
  'user/fetchUser',
  async (userId) => {
    const response = await fetch(`/api/users/${userId}`);
    return response.json();
  }
);

// Use in component
function UserComponent() {
  const dispatch = useDispatch();
  
  useEffect(() => {
    dispatch(fetchUser(1)); // Dispatch async action
  }, []);
  
  return <div>{/* ... */}</div>;
}
```

---

### SENIOR LEVEL

#### 1. When should Context be preferred over Redux?

**Use Context when:**

1. **Passing data through many components**
   ```jsx
   // Instead of Redux, use Context
   const UserContext = createContext();
   
   <UserContext.Provider value={user}>
     <App />
   </UserContext.Provider>
   ```

2. **Simple, non-changing state**
   ```jsx
   // Theme, language, user authentication
   // These don't change often
   const ThemeContext = createContext();
   ```

3. **Avoiding Redux setup overhead**
   ```jsx
   // For small projects, Context is simpler
   // No need for reducers, actions, thunks
   ```

4. **State is loosely connected**
   ```jsx
   // Each feature is independent
   // Not many dependencies between them
   ```

**Use Redux when:**
- State is complex and interconnected
- Many developers working on same codebase
- Need advanced DevTools
- Require middleware (logging, API calls)
- State changes frequently

---

#### 2. How do you structure Redux in large apps?

**Feature-Sliced Architecture:**

```
src/
├── features/
│   ├── user/
│   │   ├── userSlice.js
│   │   ├── userApi.js (API calls)
│   │   ├── userSelectors.js (Select from state)
│   │   ├── components/
│   │   └── hooks/
│   │
│   ├── posts/
│   │   ├── postsSlice.js
│   │   ├── postsApi.js
│   │   ├── postsSelectors.js
│   │   ├── components/
│   │   └── hooks/
│   │
│   └── comments/
│       ├── commentsSlice.js
│       └── ...
│
├── store.js (Configure store)
├── App.js
└── index.js
```

**store.js Example:**
```jsx
import { configureStore } from '@reduxjs/toolkit';
import userReducer from './features/user/userSlice';
import postsReducer from './features/posts/postsSlice';

export const store = configureStore({
  reducer: {
    user: userReducer,
    posts: postsReducer
  }
});
```

**userSelectors.js (Reusable selectors):**
```jsx
export const selectUser = (state) => state.user;
export const selectUserName = (state) => state.user.name;
export const selectUserRole = (state) => state.user.role;
```

**Benefits:**
- Each feature is self-contained
- Easy to add/remove features
- Clear separation of concerns
- Easy to test

---

#### 3. How do you avoid over-fetching?

**Problem: Fetching data not needed**

```jsx
// ❌ Over-fetching
function UserProfile({ userId }) {
  useEffect(() => {
    // Fetches all user data even if you only need name
    dispatch(fetchUser(userId));
  }, [userId]);
  
  const name = useSelector(state => state.user.name);
  return <h1>{name}</h1>;
}
```

**Solution 1: Send specific query**
```jsx
// ✅ Fetch only needed fields
const fetchUser = createAsyncThunk(
  'user/fetchUser',
  async ({ userId, fields }) => {
    const response = await fetch(
      `/api/users/${userId}?fields=${fields.join(',')}`
    );
    return response.json();
  }
);

// Use it
dispatch(fetchUser({ userId: 1, fields: ['name', 'email'] }));
```

**Solution 2: Pagination**
```jsx
// ✅ Load data in chunks
const fetchPosts = createAsyncThunk(
  'posts/fetchPosts',
  async ({ page, limit }) => {
    const response = await fetch(
      `/api/posts?page=${page}&limit=${limit}`
    );
    return response.json();
  }
);
```

**Solution 3: Lazy Loading**
```jsx
// ✅ Load data when needed
function UserDetails({ userId }) {
  const details = useSelector(state => state.user.details);
  const dispatch = useDispatch();
  
  const loadDetails = () => {
    dispatch(fetchUserDetails(userId)); // Load only when requested
  };
  
  return (
    <>
      <button onClick={loadDetails}>Show Details</button>
      {details && <Details {...details} />}
    </>
  );
}
```

---

## SECTION 4: REACT PERFORMANCE

### MID LEVEL

#### 1. What is React.memo?

**React.memo = Prevents re-render if props don't change**

```jsx
// Without React.memo
function Child({ name }) {
  console.log('Child rendered');
  return <div>{name}</div>;
}

// Parent re-renders → Child ALWAYS re-renders
function Parent() {
  const [count, setCount] = useState(0);
  
  return (
    <>
      <button onClick={() => setCount(count + 1)}>
        Count: {count}
      </button>
      <Child name="Ali" /> {/* Re-renders even though name didn't change */}
    </>
  );
}
```

**With React.memo:**
```jsx
// Wrapped with React.memo
const Child = React.memo(({ name }) => {
  console.log('Child rendered');
  return <div>{name}</div>;
});

// Parent re-renders → Child checks props
// If name is same → Child doesn't re-render
// If name changed → Child re-renders
```

**Advanced: Custom Comparison**
```jsx
const Child = React.memo(
  ({ name, onUpdate }) => {
    return <div>{name}</div>;
  },
  (prevProps, nextProps) => {
    // Return true if props are same (DON'T re-render)
    // Return false if props are different (DO re-render)
    return prevProps.name === nextProps.name;
    // Note: onUpdate function always different = Child re-renders
  }
);
```

**When to use React.memo:**
- Child receives props that don't often change
- Child is expensive to render (complex calculations)
- Parent re-renders frequently

---

#### 2. What is lazy loading?

**Lazy Loading = Load component only when needed**

**Without lazy loading:**
```jsx
// All components loaded at once (bigger bundle)
import Home from './pages/Home';
import About from './pages/About';
import Dashboard from './pages/Dashboard';

function App() {
  const [page, setPage] = useState('home');
  
  return (
    <>
      {page === 'home' && <Home />}
      {page === 'about' && <About />}
      {page === 'dashboard' && <Dashboard />}
    </>
  );
}
```

**With lazy loading:**
```jsx
import { lazy, Suspense } from 'react';

const Home = lazy(() => import('./pages/Home'));
const About = lazy(() => import('./pages/About'));
const Dashboard = lazy(() => import('./pages/Dashboard'));

function App() {
  const [page, setPage] = useState('home');
  
  return (
    <Suspense fallback={<Loading />}>
      {page === 'home' && <Home />}
      {page === 'about' && <About />}
      {page === 'dashboard' && <Dashboard />}
    </Suspense>
  );
}

// Benefits:
// - Smaller initial bundle
// - Pages load when first accessed
// - Faster initial page load
```

**For Routes (Next.js):**
```jsx
import dynamic from 'next/dynamic';

const Dashboard = dynamic(
  () => import('./pages/Dashboard'),
  { loading: () => <Loading /> }
);

export default Dashboard;
```

---

#### 3. What is code splitting?

**Code Splitting = Break bundle into smaller files**

**Without code splitting:**
```
bundle.js (500KB)
- Everything in one file
- First load: 500KB download
- User waits
```

**With code splitting:**
```
main.js (150KB) - loaded first
admin.js (100KB) - loaded only if admin
profile.js (80KB) - loaded only if accessing profile
ui-components.js (70KB) - shared, lazy loaded
```

**How to do code splitting:**

1. **Route-based (most common)**
   ```jsx
   const Home = lazy(() => import('./pages/Home'));
   const Admin = lazy(() => import('./pages/Admin'));
   
   function App() {
     return (
       <Routes>
         <Route path="/" element={<Home />} />
         <Route path="/admin" element={<Admin />} />
       </Routes>
     );
   }
   ```

2. **Component-based**
   ```jsx
   const Modal = lazy(() => import('./components/Modal'));
   
   function App() {
     const [showModal, setShowModal] = useState(false);
     
     return (
       <>
         <button onClick={() => setShowModal(true)}>
           Open Modal
         </button>
         {showModal && (
           <Suspense fallback={null}>
             <Modal />
           </Suspense>
         )}
       </>
     );
   }
   ```

3. **Library splitting**
   ```jsx
   // Dynamic import of heavy library
   const PDFViewer = lazy(() => import('pdf-lib'));
   ```

**Tools:**
- webpack (automatic)
- Vite (automatic)
- Next.js (automatic on routes)

---

### SENIOR LEVEL

#### 1. Table has 10,000 rows. How optimize?

**Problem:** Rendering 10,000 DOM nodes = Slow

**Solution: Virtualization**

Virtualization = Render only visible rows

```jsx
import { FixedSizeList } from 'react-window';

function LargeTable({ data }) {
  return (
    <FixedSizeList
      height={600}
      itemCount={data.length}
      itemSize={35}
    >
      {({ index, style }) => (
        <div style={style}>
          <Row data={data[index]} />
        </div>
      )}
    </FixedSizeList>
  );
}

// If 10,000 rows visible:
// - Without virtualization: Renders 10,000 rows = slow
// - With virtualization: Renders only 20 visible rows = fast
```

**Other Optimizations:**

1. **Pagination**
   ```jsx
   // Show 50 rows per page
   const [page, setPage] = useState(1);
   const itemsPerPage = 50;
   const visibleData = data.slice(
     (page - 1) * itemsPerPage,
     page * itemsPerPage
   );
   ```

2. **Memoization**
   ```jsx
   const Row = React.memo(({ data }) => {
     return <div>{data.name}</div>;
   });
   ```

3. **useCallback for handlers**
   ```jsx
   const handleRowClick = useCallback((id) => {
     // Handle click
   }, []);
   ```

4. **Lazy loading**
   ```jsx
   // Load data as user scrolls
   const [data, setData] = useState([]);
   
   const onScroll = () => {
     if (userAtBottom()) {
       fetchMoreData();
     }
   };
   ```

**Best Approach for 10,000 rows:**
1. Use virtualization (react-window, react-virtual)
2. Add pagination
3. Memoize row components
4. Add lazy loading

---

#### 2. Why first page load is slow?

**First Page Load = Initial HTML + JS + CSS downloaded + Parsed + Rendered**

**Common Causes:**

1. **Large Bundle Size**
   ```bash
   npm run build
   ls -lh build/js/
   
   # If main.js > 250KB = problem
   # Solutions:
   # - Code splitting
   # - Remove unused libraries
   # - Tree shaking
   ```

2. **Blocking Scripts**
   ```html
   <!-- ❌ Blocks page rendering -->
   <script src="analytics.js"></script>
   <script src="ads.js"></script>
   
   <!-- ✅ Load after page -->
   <script async src="analytics.js"></script>
   <script defer src="ads.js"></script>
   ```

3. **Large Images**
   ```jsx
   // ❌ 5MB image
   <img src="background.jpg" />
   
   // ✅ Optimized image
   <img 
     src="background.webp"
     alt="Background"
     loading="lazy"
   />
   ```

4. **API Calls on Load**
   ```jsx
   // ❌ Wait for API before showing page
   useEffect(() => {
     fetch('/api/user').then(setUser);
   }, []);
   
   // ✅ Show page, load data after
   // Fetch in background, show partial UI
   ```

5. **No Caching**
   ```
   - Don't cache JS/CSS = Download every time
   - Cache headers missing = Browser downloads again
   ```

**Solutions:**
```
1. Check bundle size (webpack-bundle-analyzer)
2. Code split routes
3. Lazy load images
4. Compress images (Tinypng, ImageOptim)
5. Use CDN
6. Enable gzip compression
7. Minify CSS/JS
8. Remove unused dependencies
```

---

#### 3. How reduce bundle size?

**Step 1: Analyze Bundle**
```bash
npm install --save-dev webpack-bundle-analyzer

# Add to build script, then build
npm run build

# See what's taking up space
```

**Step 2: Remove Unused Libraries**
```bash
# Find dependencies you don't use
npm list

# Uninstall unused
npm uninstall library-name
```

**Step 3: Import Specific Functions**
```jsx
// ❌ Imports entire library (full size)
import _ from 'lodash';

// ✅ Imports only needed function (smaller)
import map from 'lodash/map';
```

**Step 4: Lazy Load Heavy Libraries**
```jsx
// ❌ Loaded on page init
import moment from 'moment';

// ✅ Loaded when needed
const moment = await import('moment');
```

**Step 5: Use Lighter Alternatives**
```jsx
// ❌ moment.js = 67KB
import moment from 'moment';

// ✅ date-fns = 13KB
import { format } from 'date-fns';

// ❌ axios = 13KB
// ✅ fetch = 0KB (built-in)
```

**Step 6: Enable Compression**
```
Server configuration (gzip compression):
- .js files compressed automatically
- Saves ~70% size on transfer
```

**Step 7: Production Build**
```bash
npm run build

# Creates optimized, minified bundle
# Removes console logs, comments
# Tree shakes unused code
```

**Typical Results:**
- Before optimization: 400KB
- After optimization: 120KB
- Fast initial load: 2-3 seconds instead of 10 seconds

---

## SECTION 5: FRONTEND ARCHITECTURE

#### 1. How do you structure a large React project?

**Feature-Sliced Architecture (Recommended):**

```
src/
├── features/
│   ├── user/
│   │   ├── components/
│   │   │   ├── UserCard.jsx
│   │   │   ├── UserProfile.jsx
│   │   │   └── UserForm.jsx
│   │   ├── hooks/
│   │   │   └── useUser.js
│   │   ├── services/
│   │   │   └── userApi.js
│   │   ├── store/
│   │   │   └── userSlice.js
│   │   └── types/
│   │       └── user.types.ts
│   │
│   ├── posts/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── services/
│   │   ├── store/
│   │   └── types/
│   │
│   └── auth/
│       ├── components/
│       ├── hooks/
│       ├── services/
│       └── ...
│
├── shared/
│   ├── components/
│   │   ├── Button.jsx
│   │   ├── Modal.jsx
│   │   └── Card.jsx
│   ├── hooks/
│   │   ├── useFetch.js
│   │   └── useDebounce.js
│   ├── utils/
│   │   └── formatDate.js
│   └── styles/
│       └── theme.css
│
├── config/
│   ├── api.config.js
│   └── constants.js
│
├── App.jsx
└── index.js
```

**Advantages:**
- Each feature is self-contained
- Easy to add/remove features
- Clear separation of concerns
- Scales to large teams

---

#### 2. How do you separate business logic from UI?

**Bad: Logic mixed with UI**
```jsx
// ❌ Bad
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(false);
  
  useEffect(() => {
    setLoading(true);
    // Business logic mixed with UI
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => {
        setUser({
          ...data,
          fullName: `${data.firstName} ${data.lastName}`,
          age: new Date().getFullYear() - data.birthYear,
          isAdult: new Date().getFullYear() - data.birthYear >= 18
        });
        setLoading(false);
      });
  }, [userId]);
  
  if (loading) return <div>Loading...</div>;
  return <div>{user?.fullName}</div>;
}
```

**Good: Separate business logic**
```jsx
// ✅ Good

// 1. Custom hook (business logic)
function useUser(userId) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    setLoading(true);
    userApi.fetchUser(userId)
      .then(data => setUser(processUserData(data)))
      .catch(err => setError(err))
      .finally(() => setLoading(false));
  }, [userId]);
  
  return { user, loading, error };
}

// 2. Helper functions (pure logic)
function processUserData(rawUser) {
  return {
    ...rawUser,
    fullName: `${rawUser.firstName} ${rawUser.lastName}`,
    age: calculateAge(rawUser.birthYear),
    isAdult: calculateAge(rawUser.birthYear) >= 18
  };
}

function calculateAge(birthYear) {
  return new Date().getFullYear() - birthYear;
}

// 3. Component (only UI)
function UserProfile({ userId }) {
  const { user, loading, error } = useUser(userId);
  
  if (loading) return <Loading />;
  if (error) return <Error message={error} />;
  
  return <div>{user.fullName}</div>;
}
```

**Benefits:**
- Logic testable independently
- Easy to reuse logic
- Component focuses only on UI
- Clear responsibilities

---

#### 3. Why custom hooks?

**Custom hooks = Share logic between components**

**Problem without custom hooks:**
```jsx
// Component 1: WindowSize tracker
function Dashboard() {
  const [windowSize, setWindowSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight
  });
  
  useEffect(() => {
    const handleResize = () => {
      setWindowSize({
        width: window.innerWidth,
        height: window.innerHeight
      });
    };
    
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);
  
  return <div>Width: {windowSize.width}</div>;
}

// Component 2: Same logic repeated
function Sidebar() {
  const [windowSize, setWindowSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight
  });
  
  useEffect(() => {
    const handleResize = () => {
      setWindowSize({
        width: window.innerWidth,
        height: window.innerHeight
      });
    };
    
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);
  
  return <div>Width: {windowSize.width}</div>;
}

// Code duplication = Maintenance nightmare
```

**Solution with custom hook:**
```jsx
// Custom hook (reusable logic)
function useWindowSize() {
  const [windowSize, setWindowSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight
  });
  
  useEffect(() => {
    const handleResize = () => {
      setWindowSize({
        width: window.innerWidth,
        height: window.innerHeight
      });
    };
    
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);
  
  return windowSize;
}

// Use in any component
function Dashboard() {
  const windowSize = useWindowSize();
  return <div>Width: {windowSize.width}</div>;
}

function Sidebar() {
  const windowSize = useWindowSize();
  return <div>Width: {windowSize.width}</div>;
}

// No code duplication, cleaner, testable
```

**Other Custom Hooks Examples:**
```jsx
// useLocalStorage
function useLocalStorage(key, initialValue) {
  const [storedValue, setStoredValue] = useState(() => {
    const item = window.localStorage.getItem(key);
    return item ? JSON.parse(item) : initialValue;
  });
  
  const setValue = (value) => {
    setStoredValue(value);
    window.localStorage.setItem(key, JSON.stringify(value));
  };
  
  return [storedValue, setValue];
}

// useFetch
function useFetch(url) {
  const [data, setData] = useState(null);
  const [error, setError] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    fetch(url)
      .then(res => res.json())
      .then(data => setData(data))
      .catch(err => setError(err))
      .finally(() => setLoading(false));
  }, [url]);
  
  return { data, error, loading };
}
```

---

#### 4. Shared component vs feature component?

**Shared Components = Used in multiple features**

```
shared/components/Button.jsx
- Used everywhere
- Generic, no feature-specific logic
- Reusable, configurable
```

```jsx
// ✅ Shared Button
function Button({ children, onClick, variant = 'primary' }) {
  return (
    <button
      onClick={onClick}
      className={`btn btn-${variant}`}
    >
      {children}
    </button>
  );
}

// Used in user feature, post feature, etc.
```

**Feature Components = Specific to one feature**

```
features/user/components/UserProfile.jsx
- Only used in user feature
- Contains feature-specific logic
- Knows about user data/structure
```

```jsx
// ❌ User Feature Component (not shared)
function UserProfile({ userId }) {
  const { user, updateUser } = useUser(userId);
  
  return (
    <div className="user-profile">
      <h2>{user.name}</h2>
      <Button onClick={() => updateUser()}>
        Update Profile
      </Button>
    </div>
  );
}
```

**Rule of Thumb:**
- **Shared:** Used in 2+ features = Shared
- **Feature:** Only in 1 feature = Feature component

---

#### 5. How do you avoid prop drilling?

**Prop Drilling = Passing props through many levels**

```jsx
// ❌ Prop Drilling
function App() {
  const [user, setUser] = useState({ name: 'Ali' });
  
  return <Level1 user={user} />;
}

function Level1({ user }) {
  return <Level2 user={user} />;
}

function Level2({ user }) {
  return <Level3 user={user} />;
}

function Level3({ user }) {
  return <Level4 user={user} />;
}

function Level4({ user }) {
  return <div>{user.name}</div>;
}

// User prop passed through Level1, Level2, Level3 unnecessarily
```

**Solution 1: Context API**
```jsx
// ✅ Context (simplest)
const UserContext = createContext();

function App() {
  const [user, setUser] = useState({ name: 'Ali' });
  
  return (
    <UserContext.Provider value={user}>
      <Level1 />
    </UserContext.Provider>
  );
}

function Level1() {
  return <Level2 />;
}

function Level2() {
  return <Level3 />;
}

function Level3() {
  return <Level4 />;
}

function Level4() {
  const user = useContext(UserContext);
  return <div>{user.name}</div>;
}

// No prop drilling, user accessed directly where needed
```

**Solution 2: Redux**
```jsx
// ✅ Redux (for complex state)
function Level4() {
  const user = useSelector(state => state.user);
  return <div>{user.name}</div>;
}
```

**Solution 3: Composition**
```jsx
// ✅ Component composition
function App() {
  const [user, setUser] = useState({ name: 'Ali' });
  
  return (
    <Level1>
      <Level2>
        <Level3>
          <Level4 user={user} /> {/* Only 1 level deep */}
        </Level3>
      </Level2>
    </Level1>
  );
}
```

---

## SECTION 6: NEXT.JS

### BASIC LEVEL

#### 1. What is Next.js?

**Next.js = React framework for production**

**What it provides:**
- **Server-side rendering (SSR)** - Render on server
- **Static generation (SSG)** - Pre-render at build time
- **API routes** - Backend endpoints in same project
- **File-based routing** - Routes from folder structure
- **Automatic code splitting** - Each route = separate bundle
- **Image optimization** - Automatic image optimization
- **Built-in CSS support** - CSS modules, Tailwind, etc.

**Key Difference:**

```
React App:
- Browser downloads empty HTML
- Browser downloads JS
- JS renders component
- Page appears (slow)

Next.js App:
- Server renders component to HTML
- Browser downloads HTML with content
- Browser downloads JS (for interactivity)
- Page appears immediately (fast)
```

---

#### 2. Why Next.js over React?

**Reasons to use Next.js:**

1. **SEO** - Content in HTML = Better for search engines
   ```
   React: HTML is empty, content added by JS
   Next.js: HTML has content, search engines see it
   ```

2. **Performance** - SSR = Faster page load
   ```
   React: User waits for JS to load + render
   Next.js: Page appears immediately
   ```

3. **Built-in routing** - No need for react-router
   ```
   pages/
   ├── index.js (/)
   ├── about.js (/about)
   └── products/
       └── [id].js (/products/:id)
   ```

4. **API routes** - Backend in same project
   ```
   pages/api/
   ├── users.js (/api/users)
   └── posts.js (/api/posts)
   ```

5. **Better deployment** - Vercel (creator's platform)
   ```
   Deploy with one command
   Automatic scaling
   ```

**Use React when:**
- Building single-page app (no SSR needed)
- Only client-side rendering needed
- Don't need SEO

**Use Next.js when:**
- Need SEO (e-commerce, blogs, marketing)
- Want better performance
- Want server-side rendering
- Want API routes
- Building production app

---

### MID LEVEL

#### 1. SSR vs CSR?

| Feature | SSR | CSR |
|---------|-----|-----|
| Render location | Server | Browser |
| HTML content | Full | Empty |
| First page load | Fast | Slow |
| SEO | Good | Bad |
| Server load | High | Low |
| Good for | Public content | Admin panels |

**SSR - Server-Side Rendering:**
```
1. User requests page
2. Server renders React component
3. Server sends HTML (with content)
4. Browser displays HTML
5. Browser downloads JS (hydration)
6. Page is interactive
```

```jsx
// Next.js SSR
export async function getServerProps() {
  const data = await fetch('API');
  return { props: { data } };
}

export default function Page({ data }) {
  return <div>{data.name}</div>;
}

// Every request = server renders = fresh data
```

**CSR - Client-Side Rendering:**
```
1. User requests page
2. Server sends empty HTML
3. Browser downloads JS
4. JS runs in browser
5. JS renders component
6. Page appears
```

```jsx
// React CSR
export default function Page() {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    fetch('API').then(res => setData(res.data));
  }, []);
  
  return <div>{data?.name}</div>;
}

// Browser renders, slower first load
```

---

#### 2. SSG vs SSR?

| Feature | SSG | SSR |
|---------|-----|-----|
| When rendered | Build time | Every request |
| Speed | Fast | Medium |
| Fresh data | No | Yes |
| Good for | Static content | Dynamic content |
| Build time | Slow | Fast |

**SSG - Static Site Generation:**
```
Build time: npm run build
  ↓
Next.js renders all pages to HTML
  ↓
HTML files created
  ↓
Serve HTML files (super fast)

Good for: Blog posts, documentation, static pages
```

```jsx
export async function getStaticProps() {
  const posts = await fetch('API');
  return {
    props: { posts },
    revalidate: 3600 // Regenerate every hour
  };
}

export default function Blog({ posts }) {
  return <div>{posts.map(p => <div>{p.title}</div>)}</div>;
}
```

**SSR - Server-Side Rendering:**
```
User requests page
  ↓
Server renders component (every time)
  ↓
Server sends HTML
  ↓
Browser displays

Good for: Dynamic content, user-specific data
```

```jsx
export async function getServerProps() {
  const data = await fetch('API', {
    headers: { 'Authorization': 'Bearer token' }
  });
  return { props: { data } };
}

export default function Dashboard({ data }) {
  return <div>{data.userName}</div>;
}
```

---

#### 3. Hydration?

**Hydration = Attaching JS interactivity to server-rendered HTML**

```
Server renders:
<div>Hello</div>

Browser downloads HTML:
<div>Hello</div>

Browser downloads JS

Hydration: JS "attaches" to HTML
<div>Hello</div> + onClick handlers + state

Page becomes interactive
```

**Hydration Mismatch - Common Problem:**

```jsx
// ❌ Causes hydration mismatch
export default function Page() {
  const [mounted, setMounted] = useState(false);
  
  useEffect(() => {
    setMounted(true);
  }, []);
  
  return (
    <div>
      {mounted ? <p>Client rendered</p> : <p>Server rendered</p>}
    </div>
  );
}

// Server renders: "Server rendered"
// Browser renders: "Client rendered"
// Mismatch = Error
```

**Fix:**
```jsx
// ✅ Correct
export default function Page() {
  const [mounted, setMounted] = useState(false);
  
  useEffect(() => {
    setMounted(true);
  }, []);
  
  if (!mounted) {
    return null; // Don't render on server
  }
  
  return <div>Client rendered</div>;
}

// Or use dynamic import
import dynamic from 'next/dynamic';

const ClientComponent = dynamic(
  () => import('./ClientComponent'),
  { ssr: false }
);
```

---

### SENIOR LEVEL

#### 1. When would you choose CSR over SSR?

**Choose CSR when:**

1. **Admin panels / Dashboards**
   - Only authenticated users see
   - No SEO needed
   - Frequently updated data
   ```jsx
   // Admin panel (CSR is fine)
   export default function AdminDashboard() {
     const [analytics, setAnalytics] = useState(null);
     
     useEffect(() => {
       fetchAnalytics().then(setAnalytics);
     }, []);
     
     return <div>{/* dashboard UI */}</div>;
   }
   ```

2. **User-specific content**
   - Each user sees different data
   - SSR would be slow (render per user)
   ```jsx
   // User profile (CSR better)
   export default function Profile() {
     const { user } = useAuth();
     return <div>{user.name}</div>;
   }
   ```

3. **Real-time updates**
   - WebSocket, live data
   - Server-render not needed
   ```jsx
   // Chat app (CSR needed)
   useEffect(() => {
     socket.on('message', (msg) => {
       setMessages(prev => [...prev, msg]);
     });
   }, []);
   ```

4. **High performance needed**
   - Don't want server load
   - CDN caching less effective
   ```jsx
   // Complex dashboard
   // Too expensive to render on server
   // Better: SSG + CSR updates
   ```

**Hybrid Approach (Best):**
```jsx
// Combine SSG + CSR
export async function getStaticProps() {
  // Generate static HTML at build
  const initialData = await fetch('API');
  return { props: { initialData }, revalidate: 3600 };
}

export default function Page({ initialData }) {
  const [data, setData] = useState(initialData);
  
  useEffect(() => {
    // Update with fresh data on client
    fetch('API').then(setData);
  }, []);
  
  return <div>{data}</div>;
}

// Benefits:
// - Static HTML loads fast
// - Fresh data on client
// - Best of both worlds
```

---

#### 2. SEO benefits of SSR?

**SEO = Search Engine Optimization**

Search engines crawl HTML to index pages

**React CSR Problem:**
```
Search engine requests page
  ↓
Server sends: <html><body><div id="root"></div></body></html>
  ↓
Search engine can't see content (JS not executed)
  ↓
Page not indexed properly
  ↓
Bad SEO
```

**Next.js SSR Solution:**
```
Search engine requests page
  ↓
Server renders and sends:
<html>
  <body>
    <h1>Product Name</h1>
    <p>Product description</p>
    <div id="root">...</div>
  </body>
</html>
  ↓
Search engine sees content
  ↓
Page indexed properly
  ↓
Good SEO
```

**SSR SEO Benefits:**

1. **Content is visible in HTML**
   ```jsx
   <title>Product - E-commerce Site</title>
   <meta name="description" content="...">
   <h1>Product Name</h1>
   ```

2. **Social media preview works**
   ```
   Share link on Twitter/Facebook
   Preview shows: title, description, image
   (needs HTML content)
   ```

3. **Rich snippets**
   ```
   Search engine extracts structured data
   Shows in search results
   (needs HTML)
   ```

**Implementation:**
```jsx
import Head from 'next/head';

export async function getServerProps() {
  const product = await fetchProduct();
  
  return {
    props: { product }
  };
}

export default function Product({ product }) {
  return (
    <>
      <Head>
        <title>{product.name} - Store</title>
        <meta name="description" content={product.description} />
        <meta property="og:image" content={product.image} />
      </Head>
      
      <h1>{product.name}</h1>
      <p>{product.description}</p>
    </>
  );
}
```

---

#### 3. What causes hydration mismatch?

**Hydration Mismatch = Server HTML ≠ Client HTML**

**Cause 1: Conditional Rendering Based on Client State**
```jsx
// ❌ Mismatch
export default function Component() {
  const [isMobile, setIsMobile] = useState(false);
  
  useEffect(() => {
    setIsMobile(window.innerWidth < 768);
  }, []);
  
  return (
    <div>
      {isMobile ? <MobileNav /> : <DesktopNav />}
    </div>
  );
}

// Server renders: <DesktopNav /> (no window)
// Client renders: <MobileNav /> (detected window.innerWidth)
// Mismatch error
```

**Fix:**
```jsx
// ✅ Correct
export default function Component() {
  const [isMobile, setIsMobile] = useState(false);
  const [mounted, setMounted] = useState(false);
  
  useEffect(() => {
    setIsMobile(window.innerWidth < 768);
    setMounted(true);
  }, []);
  
  if (!mounted) return null; // Skip hydration
  
  return (
    <div>
      {isMobile ? <MobileNav /> : <DesktopNav />}
    </div>
  );
}
```

**Cause 2: Random/Dynamic Values**
```jsx
// ❌ Mismatch
export default function Component() {
  return <div>{Math.random()}</div>;
}

// Server: <div>0.123</div>
// Client: <div>0.456</div>
// Mismatch
```

**Fix:**
```jsx
// ✅ Correct
export default function Component() {
  const [random, setRandom] = useState(null);
  
  useEffect(() => {
    setRandom(Math.random());
  }, []);
  
  return <div>{random}</div>;
}
```

**Cause 3: Date/Time Issues**
```jsx
// ❌ Mismatch
export default function Component() {
  return <div>{new Date().toLocaleString()}</div>;
}

// Server: renders at 2:00 PM
// Client: renders at 2:05 PM
// Different times = Mismatch
```

**Fix:**
```jsx
// ✅ Correct
export default function Component() {
  const [date, setDate] = useState(null);
  
  useEffect(() => {
    setDate(new Date().toLocaleString());
  }, []);
  
  return <div>{date}</div>;
}
```

**Common Rule:**
- If value depends on browser APIs (window, document, Date)
- Don't render on server
- Only render on client after mount

---

## SECTION 7: API LAYER

### MID LEVEL

#### 1. Axios vs Fetch?

| Feature | Fetch | Axios |
|---------|-------|-------|
| Built-in | Yes | No |
| Bundle size | 0KB | 13KB |
| Syntax | Promise | Promise |
| Request timeout | Needs AbortController | Built-in timeout |
| Interceptors | No | Yes |
| Default params | No | Yes |
| Learning | Harder | Easier |

**Fetch API:**
```jsx
// GET request
const response = await fetch('/api/users');
const data = await response.json();

// POST request
await fetch('/api/users', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ name: 'Ali' })
});

// Error handling
try {
  const response = await fetch('/api/users');
  if (!response.ok) throw new Error('API error');
  const data = await response.json();
} catch (error) {
  console.error(error);
}
```

**Axios:**
```jsx
// GET request
const { data } = await axios.get('/api/users');

// POST request
await axios.post('/api/users', { name: 'Ali' });

// Error handling
try {
  const { data } = await axios.get('/api/users');
} catch (error) {
  console.error(error.response.data);
}

// Interceptors (useful for auth)
axios.interceptors.request.use((config) => {
  config.headers.Authorization = `Bearer ${token}`;
  return config;
});
```

**Recommendation:**
- **Small project:** Use Fetch (no dependencies)
- **Complex project:** Use Axios (features, cleaner syntax)
- **Modern approach:** Fetch + wrapper library

---

#### 2. Where should API calls live?

**Option 1: In Components (❌ Bad)**
```jsx
// ❌ Don't do this
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    // API call directly in component
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => setUser(data));
  }, [userId]);
  
  return <div>{user?.name}</div>;
}

// Problems:
// - Hard to reuse
// - Hard to test
// - Components mixed with logic
```

**Option 2: In Custom Hook (✅ Good)**
```jsx
// ✅ Better
function useUser(userId) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    setLoading(true);
    userApi.fetchUser(userId)
      .then(data => setUser(data))
      .catch(err => setError(err))
      .finally(() => setLoading(false));
  }, [userId]);
  
  return { user, loading, error };
}

function UserProfile({ userId }) {
  const { user, loading, error } = useUser(userId);
  return <div>{user?.name}</div>;
}

// Benefits:
// - Reusable
// - Component clean
// - Logic separated
```

**Option 3: Service Layer (✅ Best)**
```jsx
// services/userApi.js
const userApi = {
  async fetchUser(userId) {
    const response = await fetch(`/api/users/${userId}`);
    return response.json();
  },
  
  async updateUser(userId, data) {
    const response = await fetch(`/api/users/${userId}`, {
      method: 'PUT',
      body: JSON.stringify(data)
    });
    return response.json();
  }
};

// hooks/useUser.js
function useUser(userId) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(false);
  
  useEffect(() => {
    setLoading(true);
    userApi.fetchUser(userId)
      .then(setUser)
      .finally(() => setLoading(false));
  }, [userId]);
  
  return { user, loading };
}

// Component clean and reusable
function UserProfile({ userId }) {
  const { user, loading } = useUser(userId);
  return <div>{user?.name}</div>;
}
```

---

#### 3. Service Layer Pattern?

**Service Layer = Centralized API calls**

```
services/
├── userApi.js (All user API calls)
├── postApi.js (All post API calls)
└── authApi.js (All auth API calls)
```

**Example Structure:**
```jsx
// services/userApi.js
export const userApi = {
  // GET all users
  async getUsers(page = 1) {
    const response = await fetch(`/api/users?page=${page}`);
    if (!response.ok) throw new Error('Failed to fetch');
    return response.json();
  },
  
  // GET single user
  async getUser(id) {
    const response = await fetch(`/api/users/${id}`);
    if (!response.ok) throw new Error('Failed to fetch');
    return response.json();
  },
  
  // CREATE user
  async createUser(userData) {
    const response = await fetch('/api/users', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(userData)
    });
    if (!response.ok) throw new Error('Failed to create');
    return response.json();
  },
  
  // UPDATE user
  async updateUser(id, userData) {
    const response = await fetch(`/api/users/${id}`, {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(userData)
    });
    if (!response.ok) throw new Error('Failed to update');
    return response.json();
  },
  
  // DELETE user
  async deleteUser(id) {
    const response = await fetch(`/api/users/${id}`, {
      method: 'DELETE'
    });
    if (!response.ok) throw new Error('Failed to delete');
    return response.json();
  }
};
```

**Using Service Layer:**
```jsx
// hooks/useUser.js
function useUser(userId) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    const fetchUser = async () => {
      try {
        setLoading(true);
        const data = await userApi.getUser(userId);
        setUser(data);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };
    
    fetchUser();
  }, [userId]);
  
  const updateUser = async (newData) => {
    try {
      const updated = await userApi.updateUser(userId, newData);
      setUser(updated);
    } catch (err) {
      setError(err.message);
    }
  };
  
  return { user, loading, error, updateUser };
}

// Component
function UserProfile({ userId }) {
  const { user, loading, error, updateUser } = useUser(userId);
  
  return (
    <div>
      {loading && <p>Loading...</p>}
      {error && <p>Error: {error}</p>}
      {user && (
        <>
          <h2>{user.name}</h2>
          <button onClick={() => updateUser({ name: 'New Name' })}>
            Update
          </button>
        </>
      )}
    </div>
  );
}
```

**Benefits:**
- All API calls in one place
- Easy to change endpoints
- Consistent error handling
- Easy to mock for tests
- Easy to add middleware (auth tokens, etc.)

---

#### 4. Error handling strategy?

**Strategy: Layer-based error handling**

**Layer 1: Service Layer**
```jsx
// services/userApi.js
export const userApi = {
  async getUser(id) {
    try {
      const response = await fetch(`/api/users/${id}`);
      
      if (!response.ok) {
        throw new ApiError(
          response.status,
          `Failed to fetch user: ${response.statusText}`
        );
      }
      
      return response.json();
    } catch (error) {
      throw new ApiError(500, error.message);
    }
  }
};

class ApiError extends Error {
  constructor(status, message) {
    super(message);
    this.status = status;
    this.name = 'ApiError';
  }
}
```

**Layer 2: Hook Layer**
```jsx
function useUser(userId) {
  const [user, setUser] = useState(null);
  const [error, setError] = useState(null);
  const [loading, setLoading] = useState(false);
  
  useEffect(() => {
    const fetchUser = async () => {
      try {
        setLoading(true);
        setError(null);
        const data = await userApi.getUser(userId);
        setUser(data);
      } catch (err) {
        if (err.status === 404) {
          setError('User not found');
        } else if (err.status === 401) {
          setError('Unauthorized - please login');
        } else {
          setError(err.message || 'Something went wrong');
        }
      } finally {
        setLoading(false);
      }
    };
    
    fetchUser();
  }, [userId]);
  
  return { user, error, loading };
}
```

**Layer 3: Component Layer**
```jsx
function UserProfile({ userId }) {
  const { user, error, loading } = useUser(userId);
  
  if (loading) return <Spinner />;
  
  if (error) return <ErrorMessage message={error} />;
  
  if (!user) return <p>No user found</p>;
  
  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}
```

---

#### 5. Retry strategy?

**Retry = Automatically retry failed requests**

**Simple Retry:**
```jsx
async function fetchWithRetry(url, options = {}, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      const response = await fetch(url, options);
      if (response.ok) return response.json();
      
      // Don't retry client errors (4xx)
      if (response.status >= 400 && response.status < 500) {
        throw new Error(`Client error: ${response.status}`);
      }
    } catch (error) {
      if (i === maxRetries - 1) throw error; // Last attempt
      
      // Wait before retrying (exponential backoff)
      const delay = Math.pow(2, i) * 1000; // 1s, 2s, 4s
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}

// Use it
try {
  const user = await fetchWithRetry('/api/user/1');
} catch (error) {
  console.error('Failed after retries:', error);
}
```

**With Exponential Backoff:**
```jsx
async function fetchWithExponentialBackoff(url, maxRetries = 3) {
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      const response = await fetch(url);
      if (response.ok) return response.json();
      
      // Don't retry on client error
      if (response.status < 500) throw new Error('Client error');
    } catch (error) {
      if (attempt === maxRetries - 1) throw error;
      
      // Exponential backoff: 1s, 2s, 4s, 8s
      const delayMs = Math.pow(2, attempt) * 1000;
      console.log(`Retrying in ${delayMs}ms...`);
      
      await new Promise(resolve => setTimeout(resolve, delayMs));
    }
  }
}
```

**Using in Hook:**
```jsx
function useUserWithRetry(userId) {
  const [user, setUser] = useState(null);
  const [error, setError] = useState(null);
  const [loading, setLoading] = useState(false);
  
  useEffect(() => {
    const fetchUser = async () => {
      try {
        setLoading(true);
        const data = await fetchWithExponentialBackoff(
          `/api/users/${userId}`,
          3 // max retries
        );
        setUser(data);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };
    
    fetchUser();
  }, [userId]);
  
  return { user, error, loading };
}
```

**Best Practices:**
- Retry only server errors (5xx), not client errors (4xx)
- Use exponential backoff (wait longer each time)
- Set maximum retry attempts
- Log retry attempts
- Don't retry forever (eventual failure is better)

---

## SECTION 8: SECURITY

#### 1. Why HttpOnly cookies instead of localStorage?

**localStorage = Accessible from JavaScript**
```jsx
// ❌ Vulnerable
localStorage.setItem('token', authToken);

// Any script can access it
const token = localStorage.getItem('token');

// XSS attack steals token
// Attacker injects: <script>alert(localStorage.getItem('token'))</script>
```

**HttpOnly Cookies = Not accessible from JavaScript**
```
// ✅ Secure
Server sets: Set-Cookie: token=xyz; HttpOnly; Secure

// Browser stores in cookie
// JavaScript CANNOT access it
localStorage.getItem('token') // Always null

// Browser automatically sends with requests
// XSS can't steal it
```

**Comparison:**

| Feature | localStorage | HttpOnly Cookie |
|---------|--------------|-----------------|
| Accessible from JS | Yes (🔴 Vulnerable) | No (🟢 Secure) |
| XSS vulnerable | Yes | No |
| CSRF vulnerable | No | Yes (but fixable) |
| Storage limit | 5-10MB | Unlimited |
| Persistence | Until deleted | Until expires |

**How to use HttpOnly Cookies:**

```jsx
// Backend sets cookie
res.cookie('token', authToken, {
  httpOnly: true,    // Not accessible from JS
  secure: true,      // Only sent over HTTPS
  sameSite: 'strict' // CSRF protection
});

// Frontend doesn't handle token
// Browser automatically sends with requests
const response = await fetch('/api/protected');
// Cookie automatically included in request

// Server verifies cookie
const token = req.cookies.token;
```

---

#### 2. XSS?

**XSS = Cross-Site Scripting**

Attack where attacker injects malicious JavaScript

**Example Attack:**
```jsx
// User input not sanitized
function Comment({ comment }) {
  return <div>{comment.text}</div>;
}

// Attacker posts: <script>alert('hacked')</script>
// In comment.text

// React renders: <script>alert('hacked')</script>
// Script runs = XSS attack
```

**Prevention:**

1. **Use React (escapes by default)**
   ```jsx
   // ✅ Safe - React escapes HTML
   function Comment({ comment }) {
     return <div>{comment.text}</div>;
   }
   
   // Even if comment.text = "<script>alert('xss')</script>"
   // React renders it as text, not script
   ```

2. **Never use dangerouslySetInnerHTML**
   ```jsx
   // ❌ Dangerous
   <div dangerouslySetInnerHTML={{ __html: userInput }} />
   
   // ✅ If you must, sanitize first
   import DOMPurify from 'dompurify';
   <div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(userInput) }} />
   ```

3. **Content Security Policy (CSP)**
   ```
   Header: Content-Security-Policy: script-src 'self'
   
   Only scripts from same origin run
   Injected scripts blocked
   ```

---

#### 3. CSRF?

**CSRF = Cross-Site Request Forgery**

Attacker tricks user into making unwanted request

**Example Attack:**
```
1. User logs into bank.com
2. User visits attacker.com (without logging out of bank)
3. attacker.com has hidden form:
   <form action="bank.com/transfer" method="POST">
     <input name="amount" value="1000">
     <input name="to" value="attacker-account">
   </form>
4. Form auto-submits
5. Bank transfers money (because user is logged in)
```

**Prevention:**

1. **CSRF Tokens**
   ```jsx
   // Server generates token
   const csrfToken = generateToken();
   
   // Send to client
   <form>
     <input type="hidden" name="csrf" value={csrfToken} />
     <button>Submit</button>
   </form>
   
   // Client sends with request
   await fetch('/api/transfer', {
     method: 'POST',
     body: JSON.stringify({ amount: 1000, csrf: csrfToken })
   });
   
   // Server verifies token
   if (req.body.csrf !== sessionToken) {
     reject('CSRF');
   }
   ```

2. **SameSite Cookies**
   ```
   Set-Cookie: token=xyz; SameSite=Strict
   
   Cookie only sent from same site
   Attacker site can't access cookie
   ```

3. **Check Origin Header**
   ```
   Request header: Origin: bank.com
   
   Server checks: Is origin our domain?
   If not: Reject request
   ```

---

#### 4. How do you secure JWT auth in React?

**Insecure Approach:**
```jsx
// ❌ Bad - JWT in localStorage
// Can be stolen by XSS

localStorage.setItem('token', jwtToken);

// Every request
const response = await fetch('/api/protected', {
  headers: { 'Authorization': `Bearer ${localStorage.getItem('token')}` }
});
```

**Secure Approach:**
```jsx
// ✅ Good - JWT in HttpOnly Cookie

// Server sends JWT as HttpOnly cookie
res.cookie('token', jwtToken, {
  httpOnly: true,
  secure: true,      // HTTPS only
  sameSite: 'strict', // CSRF protection
  maxAge: 3600000    // 1 hour
});

// Client doesn't store token
// Browser automatically sends cookie with requests
const response = await fetch('/api/protected');
// Cookie automatically included

// Server verifies JWT
const token = req.cookies.token;
const decoded = jwt.verify(token, secret);
```

**Refresh Token Pattern:**
```jsx
// Short-lived access token (15 minutes)
// Long-lived refresh token (7 days)

// Login
const loginResponse = await fetch('/api/login', {
  method: 'POST',
  body: JSON.stringify({ email, password })
});

// Server sets:
// - accessToken (15 min, in memory)
// - refreshToken (7 days, HttpOnly cookie)

// Use API
const response = await fetch('/api/protected');

// If access token expired:
// Client automatically gets new one using refresh token
const newAccessToken = await fetch('/api/refresh');

// Server verifies refresh token (in cookie)
// Returns new access token
```

**Complete Secure Setup:**
```jsx
// services/auth.js
export const authApi = {
  async login(email, password) {
    const response = await fetch('/api/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      credentials: 'include', // Send cookies
      body: JSON.stringify({ email, password })
    });
    
    if (!response.ok) throw new Error('Login failed');
    
    // Server sets HttpOnly cookies
    // Client doesn't need to handle token
    return response.json();
  },
  
  async getProtectedData() {
    const response = await fetch('/api/protected', {
      credentials: 'include' // Send cookies with request
    });
    
    if (response.status === 401) {
      // Token expired, try refresh
      const refreshResponse = await fetch('/api/refresh', {
        method: 'POST',
        credentials: 'include'
      });
      
      if (refreshResponse.ok) {
        // Retry original request
        return this.getProtectedData();
      }
      
      // Refresh failed, redirect to login
      window.location.href = '/login';
      return null;
    }
    
    return response.json();
  },
  
  async logout() {
    await fetch('/api/logout', {
      method: 'POST',
      credentials: 'include'
    });
    
    // Server clears cookies
  }
};
```

**Key Security Points:**
- ✅ JWT in HttpOnly cookie (not accessible from JS)
- ✅ Secure flag (only HTTPS)
- ✅ SameSite flag (CSRF protection)
- ✅ Refresh token pattern (short-lived access tokens)
- ✅ Credentials: 'include' (send cookies with requests)
- ❌ Never localStorage for JWT
- ❌ Never expose token in URL
- ❌ Never dangerouslySetInnerHTML

---

## SECTION 9: PRODUCTION SCENARIOS

### Scenario 1: "Application feels slow"

**Investigation Checklist (STAR Method - START):**

**S - SYMPTOMS (Identify the problem)**
- Ask: "What feels slow? Entire app or just certain actions?"
- Load time? Interaction? Scrolling?
- All users or some users?

**T - TOOLS (Gather data)**

1. **Chrome DevTools - Performance Tab**
```
1. Open DevTools (F12)
2. Go to Performance tab
3. Click record
4. Do the slow action
5. Stop recording
6. Look for red/yellow bars = slow parts
```

2. **Chrome DevTools - Network Tab**
```
Check:
- API latency (slow backend?)
- Large assets (images > 500KB?)
- Waterfall view (bottlenecks?)
```

3. **React DevTools - Profiler**
```
1. Install React DevTools
2. Go to Profiler tab
3. Record
4. Do the action
5. See which components render most
6. Check "Why did this render?" for unnecessary re-renders
```

4. **Lighthouse**
```
Chrome DevTools → Lighthouse
- Performance score
- SEO score
- Accessibility issues
```

5. **Check Bundle Size**
```bash
npm install -g source-map-explorer
npm run build
source-map-explorer 'build/js/*'

# See what's taking space
```

**A - ANALYSIS (Diagnose)**

Look for:
- **Large bundle** (> 250KB) → Code split, lazy load
- **Slow API** (> 2 seconds) → Backend issue or network
- **Unnecessary re-renders** → Use React.memo, useMemo
- **Memory leak** → Check useEffect cleanup
- **Unoptimized images** → Compress, use WebP
- **Long JavaScript** → Bundle too large

**R - RECOMMEND (Fix)**

Based on findings:
1. "Bundle is 500KB → Let's code split routes"
2. "API takes 5 seconds → Ask backend team"
3. "20 re-renders for one action → Add React.memo"
4. "Memory growing → Add cleanup to useEffect"

**T - TEST (Verify)**

```bash
# Measure before
npm run build
# Check bundle size

# Make changes
# Code split, optimize images, etc.

# Measure after
npm run build
# Compare bundle sizes

# Use Lighthouse to get performance score
```

---

### Scenario 2: "API call triggered 10 times"

**Causes (Most likely first):**

**Cause 1: useEffect with no dependency array**
```jsx
// ❌ Runs on every render
useEffect(() => {
  fetchData(); // Called every render
});

// Fix: Add empty dependency array
useEffect(() => {
  fetchData(); // Called once on mount
}, []);
```

**Cause 2: Dependency array includes state that changes**
```jsx
// ❌ Called every time user changes
useEffect(() => {
  fetchUser(userId);
}, [userId]); // userId changes frequently

// Better: Only fetch when actually needed
useEffect(() => {
  if (userId) {
    fetchUser(userId);
  }
}, [userId]);
```

**Cause 3: StrictMode in development**
```jsx
// In development, React calls effects twice to detect issues
<React.StrictMode>
  <App />
</React.StrictMode>

// Normal behavior
// Not a problem in production
```

**Cause 4: Component re-renders, effect re-runs**
```jsx
// ❌ Parent re-renders → Child re-renders → useEffect re-runs
function Parent() {
  const [count, setCount] = useState(0);
  
  return (
    <>
      <button onClick={() => setCount(count + 1)}>
        {count}
      </button>
      <Child /> {/* Re-renders even though props didn't change */}
    </>
  );
}

function Child() {
  useEffect(() => {
    fetchData(); // Runs every time Child re-renders
  }, []); // Even with empty array
}
```

**Fix 1: Use AbortController to cancel**
```jsx
useEffect(() => {
  const controller = new AbortController();
  
  fetch('/api/data', { signal: controller.signal })
    .then(res => res.json())
    .then(data => setData(data));
  
  return () => {
    controller.abort(); // Cancel if effect runs again
  };
}, []);
```

**Fix 2: Check dependency array**
```jsx
// Look for dependencies that cause re-runs
useEffect(() => {
  fetchData();
}, [userId, filters]); // Are these changing too often?

// Consider memoizing dependencies
const memoizedFilters = useMemo(() => filters, [filters.page, filters.sort]);

useEffect(() => {
  fetchData();
}, [userId, memoizedFilters]);
```

---

### Scenario 3: "Search input causing lag"

**Problem: Every keystroke triggers action**

```jsx
// ❌ Lag - API called on every keystroke
function SearchUsers() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  
  useEffect(() => {
    // Called for every character typed
    fetch(`/api/search?q=${query}`)
      .then(res => res.json())
      .then(data => setResults(data));
  }, [query]); // Runs when query changes (every keystroke)
  
  return (
    <>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
      />
      {results.map(r => <div>{r}</div>)}
    </>
  );
}
```

**Solution 1: Debounce**
```jsx
// ✅ Debounce - wait 500ms before API call
function useDebounce(value, delay = 500) {
  const [debouncedValue, setDebouncedValue] = useState(value);
  
  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);
    
    return () => clearTimeout(handler);
  }, [value, delay]);
  
  return debouncedValue;
}

function SearchUsers() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  const debouncedQuery = useDebounce(query, 500);
  
  useEffect(() => {
    // Only called if user stops typing for 500ms
    if (debouncedQuery) {
      fetch(`/api/search?q=${debouncedQuery}`)
        .then(res => res.json())
        .then(data => setResults(data));
    }
  }, [debouncedQuery]); // Depends on debounced value
  
  return (
    <>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Search..."
      />
      {results.map(r => <div>{r}</div>)}
    </>
  );
}
```

**Solution 2: Pagination**
```jsx
function SearchUsers() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  const [page, setPage] = useState(1);
  
  useEffect(() => {
    const debouncedFetch = setTimeout(() => {
      fetch(`/api/search?q=${query}&page=${page}&limit=10`)
        .then(res => res.json())
        .then(data => setResults(data));
    }, 500);
    
    return () => clearTimeout(debouncedFetch);
  }, [query, page]);
  
  return (
    <>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
      />
      {/* Show 10 results per page */}
      {results.map(r => <div>{r}</div>)}
    </>
  );
}
```

**Solution 3: useMemo for expensive filtering**
```jsx
function SearchUsers() {
  const [query, setQuery] = useState('');
  const [allResults, setAllResults] = useState([]);
  
  const filteredResults = useMemo(() => {
    // Only recalculate if query changes
    return allResults.filter(item =>
      item.name.toLowerCase().includes(query.toLowerCase())
    );
  }, [query, allResults]);
  
  useEffect(() => {
    // Fetch all data once
    fetch('/api/users')
      .then(res => res.json())
      .then(data => setAllResults(data));
  }, []);
  
  return (
    <>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
      />
      {filteredResults.map(r => <div>{r}</div>)}
    </>
  );
}
```

**Best Practice:** Combine debounce + pagination
- Debounce API calls
- Paginate results
- Show loading indicator
- Cancel previous requests

---

## BONUS: Highest Probability Questions (80/20)

These 15 topics are most likely asked:

```
1. Virtual DOM - How React renders
2. Reconciliation - Comparing old vs new
3. useEffect - Side effects and cleanup
4. useMemo & useCallback - Performance
5. React.memo - Prevent re-renders
6. Redux vs Context - State management
7. Lazy Loading - Load components on demand
8. Code Splitting - Smaller bundles
9. SSR vs CSR - Server vs Client rendering
10. Next.js - React production framework
11. Folder Structure - Organize large projects
12. Frontend Architecture - Design patterns
13. API Layer Design - Services pattern
14. Custom Hooks - Reusable logic
15. Production Debugging - Finding and fixing issues
```

**Priority Study Order:**
1. Virtual DOM, Reconciliation (React fundamentals)
2. useEffect, useState, Hooks (Essential)
3. Performance (React.memo, useMemo, lazy loading)
4. State Management (Redux/Context)
5. Next.js (If on resume)
6. Architecture & Best Practices

---

## Tips for Interview Success

**What Interviewers Want to Hear:**

1. **Show Problem-Solving**
   - Not just "What is Virtual DOM?"
   - But "Virtual DOM is JavaScript representation that React uses to..."
   - Show you understand WHY, not just WHAT

2. **Use Real Examples**
   - "In my project, we had X problem, so we used Y solution"
   - References to actual code you've written

3. **Discuss Trade-offs**
   - "Redux is overkill for small apps, but good for complex state"
   - Show nuanced thinking

4. **Ask Clarifying Questions**
   - "Do you want me to consider performance?"
   - "Are we building for mobile or desktop?"
   - Shows you think about context

5. **Admit When You Don't Know**
   - "I haven't used that library, but I'd approach it like..."
   - Better than guessing wrong

---

**Good Luck! 🚀**

You've got this! Remember to stay calm, explain your thinking clearly, and remember that interviews are conversations, not tests.
