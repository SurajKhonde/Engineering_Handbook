# Next.js Interview Questions — 50 Questions (App Router Focus)

> Study order: go topic by topic. Don't jump around. Each section builds on the previous one.

---

## Section 1: Routing Fundamentals (Q1–10)

# Next.js Interview Questions — Section 1: Routing Fundamentals (Q1–10)

> Study order: go topic by topic. Each section builds on the previous one.

---

## Q1. How does file-based routing work in Next.js App Router? What folder structure creates `/dashboard/settings`?

**Core idea:**
- Each folder = a route segment
- `page.tsx` = actual route entry
- Nested folders = nested routes

```
app/
  dashboard/
    settings/
      page.tsx    ← creates /dashboard/settings
```

| File | URL |
|------|-----|
| `app/page.tsx` | `/` |
| `app/dashboard/page.tsx` | `/dashboard` |
| `app/dashboard/settings/page.tsx` | `/dashboard/settings` |

**Shared layout pattern:**

```
app/
  dashboard/
    layout.tsx    ← wraps all dashboard routes
    settings/
      page.tsx
```

> 🧠 **Mental model:** URL path = folder nesting. `page.tsx` is the endpoint. `layout.tsx` is the shared wrapper.

---

## Q2. What is the difference between `page.tsx`, `layout.tsx`, `loading.tsx`, and `error.tsx`?

| File | Purpose | Required | Must be client? |
|------|---------|----------|-----------------|
| `page.tsx` | Main route UI | ✅ Yes | No |
| `layout.tsx` | Shared wrapper, persists across nav | No | No |
| `loading.tsx` | Suspense-based loading UI | No | No |
| `error.tsx` | Error boundary fallback | No | ✅ Yes |

**Render order for `/dashboard/settings`:**

```
layout.tsx wraps → loading.tsx shows (if needed) → page.tsx loads → if error → error.tsx takes over
```

**`error.tsx` pattern:**

```tsx
"use client";

export default function Error({ error, reset }) {
  return (
    <div>
      <p>Something went wrong</p>
      <button onClick={reset}>Try again</button>
    </div>
  );
}
```

> 🧠 `error.tsx` must be a Client Component. `reset()` retries rendering the segment.

---

## Q3. What is a dynamic route? How do you create a route like `/users/123`?

A dynamic route has a variable segment — the folder name uses square brackets.

```
app/
  users/
    [id]/
      page.tsx    ← matches /users/123, /users/abc, etc.
```

```tsx
export default function Page({ params }) {
  return <h1>User ID: {params.id}</h1>;
}
```

**Nested dynamic params:**

```
app/users/[id]/posts/[postId]/page.tsx

URL:    /users/123/posts/456
params: { id: "123", postId: "456" }
```

> 🧠 Brackets in folder name = dynamic segment. Params are always strings.

---

## Q4. What is the difference between `[id]`, `[...slug]`, and `[[...slug]]`?

| Syntax | Matches | Required? | Params shape |
|--------|---------|-----------|--------------|
| `[id]` | `/users/123` | Yes (1 segment) | `"123"` |
| `[...slug]` | `/docs/a/b/c` | Yes (≥1 segment) | `["a","b","c"]` |
| `[[...slug]]` | `/docs` or `/docs/a/b` | No (optional) | `undefined` or array |

**Important gotcha:**

`[...slug]` does **not** match the base path — `/docs` returns 404. Use `[[...slug]]` if you also need to match the parent route.

> 🧠 One bracket = single. Three dots = many. Double bracket = optional.

---

## Q5. What is a Route Group? What does wrapping a folder in `(parentheses)` do? Does it appear in the URL?

A route group organises routes or shares layouts **without** adding a URL segment.

```
app/
  (auth)/
    login/
      page.tsx    ← URL: /login
    register/
      page.tsx    ← URL: /register
```

`(auth)` is completely ignored by the router. It only exists for folder organisation and layout scoping.

**Why use it?**
- Group related routes
- Share a layout without changing the URL
- Have multiple distinct layouts at the same level

> 🧠 **( ) = organise, not route.**

---

## Q6. You want `/dashboard` and `/marketing` to share the same layout but NOT affect the URL. How?

```
app/
  (main)/
    layout.tsx      ← shared layout
    dashboard/
      page.tsx      ← URL: /dashboard
    marketing/
      page.tsx      ← URL: /marketing
```

Both routes use the same `layout.tsx`, but `(main)` never appears in the URL.

> 🧠 Route group + `layout.tsx` = shared UI, clean URLs.

---

## Q7. What is a Parallel Route? Give a real example of when you would use `@slot` folders.

Parallel routes let you render multiple independent UI sections simultaneously in the same layout — each with its own loading, error, and navigation state.

```
app/
  dashboard/
    layout.tsx
    page.tsx
    @notifications/
      page.tsx
    @activity/
      page.tsx
```

```tsx
export default function DashboardLayout({
  children,
  notifications,
  activity,
}) {
  return (
    <div>
      {children}
      {notifications}
      {activity}
    </div>
  );
}
```

**Real use cases:**
- Dashboard panels (analytics, activity feed)
- Chat app (chat list + message thread)
- Admin panel (sidebar + preview pane)

> 🧠 URL stays the same. Each slot loads independently and can have its own `loading.tsx` and `error.tsx`.

---

## Q8. What is an Intercepting Route? What does `(.)`, `(..)`, `(..)(..)`, `(...)` mean?

An intercepting route renders a route in the current UI (e.g. a modal) instead of navigating away. A hard refresh shows the full standalone page.

**Real example — photo modal:**

```
app/
  feed/
    @modal/
      (.)photo/[id]/page.tsx   ← intercepted inside feed
  photo/
    [id]/page.tsx              ← full photo page (on hard refresh)
```

**Intercept level syntax:**

| Syntax | Meaning | Like file path |
|--------|---------|----------------|
| `(.)` | Same folder level | `./` |
| `(..)` | One level up | `../` |
| `(..)(..)` | Two levels up | `../../` |
| `(...)` | From app root | `/` |

> 🧠 Always combine intercepting routes with a `@modal` parallel slot.

---

## Q9. What is the difference between `useRouter`, `usePathname`, and `useSearchParams`?

| Hook | Purpose | Think of it as |
|------|---------|----------------|
| `useRouter` | Navigate / change route | move |
| `usePathname` | Read current URL path | where am I? |
| `useSearchParams` | Read query params | what filters are set? |

All three are **client-only hooks** from `next/navigation` (not `next/router`). They require `"use client"`.

**Combined usage:**

```tsx
"use client";
import { useRouter, usePathname, useSearchParams } from "next/navigation";

const router = useRouter();
const pathname = usePathname();   // "/dashboard/settings"
const params = useSearchParams(); // ?page=2 → params.get("page") === "2"

router.push("/dashboard");
router.replace("/login");
router.back();
```

> 🧠 `useSearchParams` is read-only. To update query params, build a new URL string and use `router.push()`.

---

## Q10. How does `redirect()` differ from `useRouter().push()`?

| Feature | `redirect()` | `router.push()` |
|---------|-------------|-----------------|
| Runs on | Server | Client |
| Timing | Before render | After render |
| Stops execution | ✅ Yes | No |
| UI flicker | No | Possible |
| Best for | Auth, protection | User interaction |

**`redirect()` — protect a route on the server:**

```tsx
import { redirect } from "next/navigation";

export default async function Page() {
  const user = await getUser();
  if (!user) redirect("/login"); // ← happens before any UI renders
  return <div>Dashboard</div>;
}
```

**`router.push()` — navigate on button click:**

```tsx
"use client";
import { useRouter } from "next/navigation";

const router = useRouter();

<button onClick={() => router.push("/dashboard")}>Go</button>
```

> 🧠 Server decision → `redirect()`. User action → `router.push()`.

---


---

## Section 2: Server vs Client Components (Q11–18)

# Next.js Interview Questions — Section 2: Server vs Client Components (Q11–18)

> 🧠 **Big picture before you start:**
> Your Next.js app has TWO environments — the **server** (your computer/cloud) and the **client** (the user's browser).
> Server Components run on the server. Client Components run in the browser.
> This section is ALL about knowing which is which — and why it matters.

---

## Q11. What is a Server Component? What is a Client Component? What is the default in App Router?

### 🖥️ Server Component — runs on the SERVER

Think of it like a chef in the kitchen.
The chef (server) prepares the food (HTML) and sends it to the table (browser).
The user never sees the kitchen — they just get the final plate.

**What it can do:**
- Fetch data directly (database, API)
- Use secret keys / environment variables safely
- Import heavy libraries without sending them to the browser
- Read files from the server

**What it CANNOT do:**
- Use `useState`, `useEffect`, `useRef` (React hooks)
- Handle click events, form inputs, browser stuff
- Access `window`, `document`, `localStorage`

```tsx
// This is a Server Component (default in App Router)
// No "use client" at the top = Server Component

export default async function UserProfile() {
  const user = await fetch("https://api.example.com/user/1");
  const data = await user.json();

  return <h1>Hello, {data.name}</h1>;
}
```

---

### 🌐 Client Component — runs in the BROWSER

Think of it like the waiter at the table.
The waiter (client) handles live interaction — taking new orders, responding to the customer.

**What it can do:**
- Use `useState`, `useEffect`, all React hooks
- Handle `onClick`, `onChange`, keyboard events
- Access `window`, `localStorage`, browser APIs
- Show/hide things based on user interaction

**What it CANNOT do:**
- Directly fetch from database (not safe — secrets exposed)
- Use `async/await` at component level

```tsx
"use client"; // ← THIS line makes it a Client Component

import { useState } from "react";

export default function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  );
}
```

---

### ✅ Default in App Router?

**Server Component is the default.**

Any file in `app/` folder WITHOUT `"use client"` at the top = Server Component automatically.

| Type | How to create | Default? |
|------|--------------|----------|
| Server Component | Just create the file normally | ✅ Yes |
| Client Component | Add `"use client"` at very top | No |

> 🧠 **Memory trick:** No label = Server. Add `"use client"` = Client.
> Server = chef in kitchen. Client = waiter at the table.

---

## Q12. You wrote `useState` in a component and got an error. Why? How do you fix it?

### Why the error happens

You wrote `useState` inside a **Server Component**.
Server Components run on the server — there is NO browser, NO interactivity, NO React hooks allowed.

`useState` needs the browser to work (it stores values that change on screen).
The server just generates HTML once and sends it — it doesn't "remember" state.

```tsx
// ❌ BROKEN — Server Component trying to use useState
export default function Counter() {
  const [count, setCount] = useState(0); // 💥 ERROR!

  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

**The error you see:**
```
Error: useState is not allowed in Server Components.
You may need to add "use client" to this file.
```

---

### How to fix it

Add `"use client"` at the very top of the file — before any imports.

```tsx
"use client"; // ← Add this line FIRST

import { useState } from "react";

export default function Counter() {
  const [count, setCount] = useState(0); // ✅ Works now

  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

### The rule is simple:

| Hook / Feature | Needs `"use client"`? |
|---------------|----------------------|
| `useState` | ✅ Yes |
| `useEffect` | ✅ Yes |
| `useRef` | ✅ Yes |
| `onClick`, `onChange` | ✅ Yes |
| `async/await` fetch | ❌ No (server is fine) |
| No hooks, no events | ❌ No |

> 🧠 **Memory trick:** If you're using a React hook or browser event → you need `"use client"`. Simple as that.

---

## Q13. Can a Server Component import a Client Component? Can a Client Component import a Server Component?

This is one of the most confusing parts of Next.js. Let's break it down clearly.

---

### ✅ YES — Server Component CAN import a Client Component

This is the normal, correct pattern. Do this all the time.

```tsx
// app/page.tsx — Server Component (no "use client")

import AddToCartButton from "./AddToCartButton"; // Client Component

export default async function ProductPage() {
  const product = await fetchProduct(); // ✅ server can fetch data

  return (
    <div>
      <h1>{product.name}</h1>
      <AddToCartButton id={product.id} /> {/* ✅ Client Component inside */}
    </div>
  );
}
```

```tsx
// AddToCartButton.tsx — Client Component
"use client";

export default function AddToCartButton({ id }) {
  return <button onClick={() => addToCart(id)}>Add to Cart</button>;
}
```

**Why this works:** The server renders the outer shell, then hands off just the interactive button part to the client. Best of both worlds.

---

### ❌ NO — Client Component CANNOT import a Server Component directly

```tsx
"use client";

import ServerComponent from "./ServerComponent"; // ❌ This won't work as expected!

export default function ClientComponent() {
  return <ServerComponent />; // ❌ It becomes a Client Component too
}
```

**Why?** Once you're inside a Client Component, everything gets sent to the browser. A Server Component inside it loses its "server powers" — it can't fetch from DB, use secrets, etc.

---

### ✅ Workaround — pass Server Component as `children` prop

You CAN pass a Server Component INTO a Client Component as `children`. This pattern is very common.

```tsx
// layout.tsx or page.tsx — Server Component
import ClientWrapper from "./ClientWrapper";
import ServerData from "./ServerData";

export default function Page() {
  return (
    <ClientWrapper>
      <ServerData /> {/* ✅ passed as children, stays server */}
    </ClientWrapper>
  );
}
```

```tsx
// ClientWrapper.tsx — Client Component
"use client";

export default function ClientWrapper({ children }) {
  const [open, setOpen] = useState(false);

  return (
    <div>
      <button onClick={() => setOpen(!open)}>Toggle</button>
      {open && children} {/* ✅ children is still a Server Component */}
    </div>
  );
}
```

> 🧠 **Memory trick:**
> Server → Client ✅ (always fine, very common)
> Client → Server ❌ (not directly — use `children` pattern instead)

---

## Q14. What is the `"use client"` directive? Where exactly do you place it and why?

### What it is

`"use client"` is a special instruction you write at the top of a file.
It tells Next.js: **"This component and everything it imports should run in the browser."**

It's called a **directive** — a one-line declaration, not code.

---

### Where to place it

**At the very top of the file — before EVERYTHING, including imports.**

```tsx
"use client"; // ← Line 1. Before imports. Before anything.

import { useState } from "react";
import SomeOtherThing from "./SomeOtherThing";

export default function MyComponent() {
  // ...
}
```

**Wrong placements:**

```tsx
// ❌ Wrong — after an import
import { useState } from "react";
"use client"; // Too late!
```

```tsx
// ❌ Wrong — inside the function
export default function MyComponent() {
  "use client"; // Doesn't work here
}
```

---

### Why it matters — the "boundary" concept

`"use client"` creates a **client boundary**.

Everything in this file + everything this file imports → becomes client-side.

```
Server Component
  └── imports ClientComponent ("use client") ← BOUNDARY starts here
        └── imports AnotherComponent ← also becomes client (even without "use client")
```

You don't need to add `"use client"` to every file — just the first file that needs browser features. Everything below it in the import tree is automatically treated as client.

---

### When do you need it?

Add `"use client"` when your component uses any of these:

| Feature | Needs `"use client"`? |
|---------|----------------------|
| `useState`, `useEffect`, `useRef` | ✅ Yes |
| `onClick`, `onChange`, `onSubmit` | ✅ Yes |
| `window`, `document`, `localStorage` | ✅ Yes |
| Third-party UI libraries (most of them) | ✅ Yes |
| Just displaying data, no interaction | ❌ No |

> 🧠 **Memory trick:** `"use client"` = "hey Next.js, send this to the browser". Place it FIRST, like a label on a package before you ship it.

---

## Q15. You have a page that fetches data AND has a button with `onClick`. How do you structure this WITHOUT making the entire page a Client Component?

### The problem

If you add `"use client"` to the whole page just for one button:
- The entire page (including data fetching) runs on the client
- You lose server-side benefits (performance, security, SEO)
- The page becomes heavier

---

### The solution — split into two components

Keep the page as a **Server Component** for data fetching.
Extract ONLY the interactive button into a **Client Component**.

```
app/
  products/
    page.tsx          ← Server Component (fetches data)
    AddToCartButton.tsx ← Client Component (handles click)
```

**Server Component (page.tsx) — fetches data:**

```tsx
// app/products/page.tsx
// No "use client" = Server Component ✅

import AddToCartButton from "./AddToCartButton";

export default async function ProductsPage() {
  // ✅ Can fetch data on server — fast, secure
  const products = await fetch("https://api.example.com/products");
  const data = await products.json();

  return (
    <div>
      {data.map((product) => (
        <div key={product.id}>
          <h2>{product.name}</h2>
          <p>{product.price}</p>
          {/* ✅ Drop in the Client Component just for the button */}
          <AddToCartButton id={product.id} />
        </div>
      ))}
    </div>
  );
}
```

**Client Component (AddToCartButton.tsx) — handles interaction:**

```tsx
// app/products/AddToCartButton.tsx
"use client"; // ← Only this small file is a Client Component

export default function AddToCartButton({ id }) {
  return (
    <button onClick={() => alert(`Added product ${id} to cart!`)}>
      Add to Cart
    </button>
  );
}
```

### Why this is the right pattern

| | Bad (whole page is client) | Good (split) |
|--|---------------------------|--------------|
| Data fetching | Runs in browser (slow, exposed) | Runs on server (fast, secure) |
| Bundle size | Entire page JS sent to browser | Only button JS sent |
| SEO | Worse | Better |
| Interactivity | Works | Works |

> 🧠 **Rule:** Push `"use client"` as far DOWN the component tree as possible. Make only the interactive leaf components client-side.

---

## Q16. What are the rules about using `async/await` in components? Where is it allowed and where is it not?

### Simple answer first

| Component type | `async/await` allowed? |
|---------------|----------------------|
| Server Component | ✅ Yes — fully supported |
| Client Component | ❌ No — not at component level |

---

### ✅ Server Component — async/await is fully supported

Server Components can be `async` functions. This is one of their superpowers.

```tsx
// ✅ Server Component — async works perfectly
export default async function UserPage() {
  const res = await fetch("https://api.example.com/user/1");
  const user = await res.json();

  return <h1>Hello, {user.name}</h1>;
}
```

You can also `await` database calls, file reads, anything:

```tsx
export default async function Dashboard() {
  const [user, orders] = await Promise.all([
    getUser(),
    getOrders(),
  ]);

  return (
    <div>
      <h1>{user.name}</h1>
      <p>Orders: {orders.length}</p>
    </div>
  );
}
```

---

### ❌ Client Component — async NOT allowed at component level

```tsx
"use client";

// ❌ WRONG — async Client Component
export default async function ClientPage() {
  const data = await fetchSomething(); // 💥 Not allowed
  return <div>{data}</div>;
}
```

**Why?** Client Components re-render constantly based on state. Making them async would break React's rendering model.

---

### ✅ How to fetch data in a Client Component

Use `useEffect` with a regular async function inside it:

```tsx
"use client";

import { useState, useEffect } from "react";

export default function ClientPage() {
  const [data, setData] = useState(null);

  useEffect(() => {
    // ✅ Async function INSIDE useEffect — this is the correct pattern
    async function fetchData() {
      const res = await fetch("https://api.example.com/data");
      const json = await res.json();
      setData(json);
    }

    fetchData();
  }, []); // empty array = runs once when component loads

  if (!data) return <p>Loading...</p>;
  return <div>{data.name}</div>;
}
```

> 🧠 **Memory trick:**
> Server Component = chef who cooks before serving → `async` is fine
> Client Component = waiter reacting live → no `async` at top level, use `useEffect` instead

---

## Q17. What is a Server Action? How do you define one and call it from a form?

### What is a Server Action?

A Server Action is a **function that runs on the server** but can be called directly from the client (like from a form submit).

Before Server Actions existed, you needed to:
1. Create a separate API route
2. Fetch that API from the client
3. Handle errors on both sides

Now you can just write a function with `"use server"` and call it directly. Much simpler.

---

### How to define a Server Action

```tsx
// app/actions.ts
"use server"; // ← This makes every function in this file a Server Action

export async function createUser(formData: FormData) {
  const name = formData.get("name");
  const email = formData.get("email");

  // ✅ This runs on the SERVER — safe to use DB, secrets, etc.
  await db.users.create({ name, email });

  console.log("User created:", name); // This console.log appears in your TERMINAL, not browser
}
```

---

### How to call it from a form

```tsx
// app/signup/page.tsx — Server Component
import { createUser } from "../actions";

export default function SignupPage() {
  return (
    <form action={createUser}> {/* ← pass the Server Action directly */}
      <input name="name" placeholder="Your name" />
      <input name="email" placeholder="Your email" />
      <button type="submit">Sign Up</button>
    </form>
  );
}
```

When the user clicks "Sign Up":
1. Form data is sent to the server
2. `createUser` runs on the server
3. Database is updated
4. No separate API route needed

---

### You can also define it inline inside a Server Component

```tsx
// app/signup/page.tsx
export default function SignupPage() {
  async function createUser(formData: FormData) {
    "use server"; // ← can also go inside a function

    const name = formData.get("name");
    await db.users.create({ name });
  }

  return (
    <form action={createUser}>
      <input name="name" />
      <button type="submit">Sign Up</button>
    </form>
  );
}
```

---

### Server Action vs API Route

| | Server Action | API Route (`/api/...`) |
|--|--------------|----------------------|
| Where to define | In component file or `actions.ts` | Separate `route.ts` file |
| How to call | Pass to `action={}` on form | `fetch("/api/signup")` |
| Boilerplate | Very little | More setup |
| Best for | Forms, mutations | External apps, webhooks |

> 🧠 **Memory trick:** Server Action = a server function you can hand directly to a form. No API route needed. `"use server"` = runs on server. `"use client"` = runs in browser.

---

## Q18. You have a shopping cart. The cart count in the navbar must update instantly when a user adds an item. Is this Server Component or Client Component? Why?

### Answer: Client Component ✅

### Why?

"Update instantly" = the UI changes in the browser WITHOUT a full page reload.

That means you need:
- `useState` to store the cart count
- `onClick` or some event to update it
- The change to appear immediately on screen

All of these are browser features → **Client Component**.

---

### Let's build it step by step

**Step 1 — Cart state (Client Component):**

```tsx
// components/CartProvider.tsx
"use client";

import { useState, createContext, useContext } from "react";

const CartContext = createContext(null);

export function CartProvider({ children }) {
  const [cartCount, setCartCount] = useState(0);

  function addItem() {
    setCartCount((prev) => prev + 1);
  }

  return (
    <CartContext.Provider value={{ cartCount, addItem }}>
      {children}
    </CartContext.Provider>
  );
}

export function useCart() {
  return useContext(CartContext);
}
```

**Step 2 — Navbar showing cart count (Client Component):**

```tsx
// components/Navbar.tsx
"use client";

import { useCart } from "./CartProvider";

export default function Navbar() {
  const { cartCount } = useCart();

  return (
    <nav>
      <span>My Store</span>
      <span>🛒 {cartCount}</span> {/* ✅ Updates instantly */}
    </nav>
  );
}
```

**Step 3 — Add to Cart button (Client Component):**

```tsx
// components/AddToCartButton.tsx
"use client";

import { useCart } from "./CartProvider";

export default function AddToCartButton({ productId }) {
  const { addItem } = useCart();

  return (
    <button onClick={addItem}>
      Add to Cart
    </button>
  );
}
```

**Step 4 — Product page (Server Component — fetches data):**

```tsx
// app/products/page.tsx — Server Component
import AddToCartButton from "@/components/AddToCartButton";

export default async function ProductsPage() {
  const products = await fetchProducts(); // ✅ server fetch

  return (
    <div>
      {products.map((p) => (
        <div key={p.id}>
          <h2>{p.name}</h2>
          <AddToCartButton productId={p.id} /> {/* Client Component */}
        </div>
      ))}
    </div>
  );
}
```

---

### Why NOT a Server Component for the cart?

If the cart were a Server Component:
- Clicking "Add to Cart" would need a full page reload
- The count wouldn't update instantly — it would feel slow and broken
- You can't use `useState` to track the count

| Requirement | Server Component | Client Component |
|-------------|-----------------|-----------------|
| Updates instantly (no reload) | ❌ No | ✅ Yes |
| Uses `useState` for count | ❌ No | ✅ Yes |
| Responds to `onClick` | ❌ No | ✅ Yes |
| Fetches product data | ✅ Yes | Possible but slower |

> 🧠 **Rule to remember:**
> If the UI must react to user actions instantly → **Client Component**.
> If you just need to display data from a database → **Server Component**.
> Most real apps use BOTH — server for data, client for interaction.

---

## 🗺️ Section 2 — Big Picture Summary

```
Your Next.js App
│
├── Server Components (default)
│     ✅ fetch data, use DB, use secrets
│     ❌ no hooks, no events, no browser APIs
│     → like the kitchen (user never sees it)
│
└── Client Components ("use client")
      ✅ useState, useEffect, onClick
      ✅ instant UI updates, browser APIs
      ❌ no direct DB access
      → like the waiter (handles live interaction)
```

| Question | Answer |
|----------|--------|
| Default component type? | Server Component |
| Got a `useState` error? | Add `"use client"` at top |
| Server import Client? | ✅ Yes — very common |
| Client import Server? | ❌ No — use `children` instead |
| `"use client"` placement? | Very first line of file |
| Async in Server Component? | ✅ Yes |
| Async in Client Component? | ❌ Use `useEffect` instead |
| Server Action? | Server function called from a form |
| Live cart count? | Client Component (needs `useState`) |

> 🧠 **Golden rule:** Keep things on the server by default. Only move to the client when you need interactivity.

---

## Section 3: Data Fetching (Q19–26)

# Next.js Interview Questions — Section 3: Data Fetching & Caching (Q19–26)

> 🧠 **Big picture before you start:**
> This section is all about HOW and WHEN Next.js fetches and stores data.
> The core question is always: **"Should this data be fetched fresh every time, or can it be cached?"**
> Getting this right = fast apps. Getting it wrong = slow apps or stale data.

---

## Q19. How do you fetch data in a Server Component? Show the pattern.

### The simple answer

In a Server Component, you just use `async/await` directly in the component — no `useEffect`, no loading state, no extra library needed.

It feels like writing a normal async function.

---

### Basic pattern

```tsx
// app/users/page.tsx
// No "use client" = Server Component ✅

export default async function UsersPage() {
  // Step 1: fetch the data
  const res = await fetch("https://api.example.com/users");

  // Step 2: parse JSON
  const users = await res.json();

  // Step 3: render it
  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

That's it. No `useState`, no `useEffect`, no loading spinner needed at this level.

---

### Fetching multiple things at once

If you need data from two places, use `Promise.all` so they run **in parallel** (at the same time) instead of one after another.

```tsx
export default async function DashboardPage() {
  // ✅ Both requests run at the same time — faster
  const [usersRes, ordersRes] = await Promise.all([
    fetch("https://api.example.com/users"),
    fetch("https://api.example.com/orders"),
  ]);

  const users = await usersRes.json();
  const orders = await ordersRes.json();

  return (
    <div>
      <h2>Users: {users.length}</h2>
      <h2>Orders: {orders.length}</h2>
    </div>
  );
}
```

**Why `Promise.all`?**

```
// ❌ Sequential (slow) — waits 1s + 2s = 3 seconds total
const users = await fetchUsers();   // 1 second
const orders = await fetchOrders(); // 2 seconds

// ✅ Parallel (fast) — waits max(1s, 2s) = 2 seconds total
const [users, orders] = await Promise.all([fetchUsers(), fetchOrders()]);
```

---

### Fetching from a database directly

Server Components can also query a database directly — no API needed.

```tsx
import { db } from "@/lib/database";

export default async function ProductsPage() {
  // ✅ Safe — this code never reaches the browser
  const products = await db.query("SELECT * FROM products");

  return (
    <div>
      {products.map((p) => <p key={p.id}>{p.name}</p>)}
    </div>
  );
}
```

---

### Handling errors

```tsx
export default async function UsersPage() {
  const res = await fetch("https://api.example.com/users");

  // Always check if the request worked
  if (!res.ok) {
    throw new Error("Failed to fetch users");
    // This will trigger your nearest error.tsx
  }

  const users = await res.json();
  return <ul>{users.map((u) => <li key={u.id}>{u.name}</li>)}</ul>;
}
```

> 🧠 **Memory trick:**
> Server Component data fetching = `async function` + `await fetch` at the top.
> No hooks. No loading state. Just `async/await` like a normal JS function.

---

## Q20. What is the difference between `cache: 'force-cache'`, `cache: 'no-store'`, and `next: { revalidate: 60 }`?

### First — what is caching?

Caching means **saving the result of a request so you don't have to fetch it again**.

Imagine a librarian (the server). When someone asks for a book:
- **`force-cache`** → "I already got this book, here's my copy. I won't go back to the shelf."
- **`no-store`** → "I'll go get a fresh copy from the shelf every single time."
- **`revalidate: 60`** → "I'll use my copy for 60 seconds, then go get a fresh one."

---

### `cache: 'force-cache'` — cache forever (static)

```tsx
const res = await fetch("https://api.example.com/products", {
  cache: "force-cache", // ← save the result, never fetch again
});
```

- Fetches ONCE, saves the result
- Every user gets the same cached response
- Never fetches again (until you redeploy)
- **Use for:** data that almost never changes (country list, product catalogue, blog posts)

> This is actually the **default** in Next.js if you don't specify anything.

---

### `cache: 'no-store'` — never cache (dynamic)

```tsx
const res = await fetch("https://api.example.com/cart", {
  cache: "no-store", // ← always fetch fresh data
});
```

- Fetches fresh data **every single request**
- Each user gets their own fresh response
- **Use for:** user-specific data, live prices, stock levels, shopping carts

---

### `next: { revalidate: 60 }` — cache, then refresh (ISR)

```tsx
const res = await fetch("https://api.example.com/news", {
  next: { revalidate: 60 }, // ← refresh every 60 seconds
});
```

- Caches the result for 60 seconds
- After 60 seconds, the next request gets fresh data (and that gets cached for another 60s)
- **Use for:** data that changes occasionally — news, scores, product prices

---

### Side-by-side comparison

| Option | Fetches how often? | Speed | Use case |
|--------|-------------------|-------|----------|
| `force-cache` | Once (until redeploy) | ⚡ Fastest | Static content (blog, docs) |
| `no-store` | Every request | 🐢 Slowest | Personal/live data (cart, user profile) |
| `revalidate: 60` | Every 60 seconds | ⚡ Fast + Fresh | News, prices, leaderboards |

---

### Real world example — which to use where

```tsx
// Blog post — never changes often
const post = await fetch(`/api/posts/${id}`, {
  cache: "force-cache" // ✅ Cache forever
});

// User's cart — personal and changes often
const cart = await fetch(`/api/cart`, {
  cache: "no-store" // ✅ Always fresh
});

// Football scores — changes every few minutes
const scores = await fetch(`/api/scores`, {
  next: { revalidate: 30 } // ✅ Fresh every 30 seconds
});
```

> 🧠 **Memory trick:**
> `force-cache` = frozen in time ❄️
> `no-store` = fresh every time 🔄
> `revalidate` = refresh timer ⏱️

---

## Q21. What is ISR (Incremental Static Regeneration)? How does `revalidate` work in App Router?

### What is ISR?

ISR stands for **Incremental Static Regeneration**.

Big words, simple idea:
> "Build a static (pre-made) page, but automatically rebuild it after a set amount of time."

It combines the best of two worlds:
- **Static pages** = fast (like a printed flyer — ready instantly)
- **Dynamic pages** = fresh (like a live news ticker — always current)

ISR gives you: **fast pages that also stay up to date** ✅

---

### The problem ISR solves

Imagine you have a blog with 1000 posts.

| Approach | Problem |
|----------|---------|
| Build all pages at deploy time | Takes forever. 1000 posts × 2 seconds = 33 minutes build time |
| Fetch fresh data on every request | Slow for every user. Database hit every time |
| ISR | Build once, auto-rebuild after X seconds. Fast AND fresh |

---

### How `revalidate` works in App Router

**Option 1 — revalidate on individual `fetch` calls:**

```tsx
const res = await fetch("https://api.example.com/posts", {
  next: { revalidate: 60 }, // rebuild this data every 60 seconds
});
```

**Option 2 — revalidate the entire route segment:**

```tsx
// app/blog/page.tsx

export const revalidate = 60; // ← this whole page rebuilds every 60 seconds

export default async function BlogPage() {
  const posts = await fetchPosts();
  return <div>{/* render posts */}</div>;
}
```

---

### How it actually works step by step

```
Timeline with revalidate: 60

Second 0:    User visits /blog → page is BUILT and CACHED
Second 1-59: All users get the CACHED page (super fast ⚡)
Second 60:   Page is "stale" — marked for rebuild
Second 61:   Next user visits → gets OLD cached page immediately
             → Next.js rebuilds the page IN THE BACKGROUND
Second 62:   New cached page is ready — next user gets the fresh version
```

This is called **"stale while revalidate"** — you always get a fast response, and the rebuild happens in the background.

---

### `revalidate: 0` vs `revalidate: false`

```tsx
export const revalidate = 0;     // = same as no-store (always fresh, no cache)
export const revalidate = false; // = cache forever (same as force-cache)
```

---

### On-demand revalidation (manual trigger)

You can also manually clear the cache when your data changes — no waiting for the timer.

```tsx
// app/api/revalidate/route.ts
import { revalidatePath } from "next/cache";

export async function POST(request) {
  revalidatePath("/blog"); // ← clears cache for /blog immediately
  return Response.json({ revalidated: true });
}
```

You'd call this from a CMS webhook — so when you publish a new blog post, the page rebuilds immediately.

> 🧠 **Memory trick:**
> ISR = static page with a refresh timer.
> `revalidate: 60` = "this page expires in 60 seconds, then rebuild it."
> Like milk with an expiry date — still good until it expires, then you get fresh.

---

## Q22. What is `generateStaticParams`? When and why do you use it?

### What is it?

`generateStaticParams` tells Next.js: **"Here are all the dynamic route values — pre-build pages for all of them."**

Without it, dynamic pages like `/blog/[slug]` are built on-demand (when someone visits them).
With it, they're ALL built at deploy time — so they're instant for every user.

---

### The problem it solves

You have a blog with dynamic routes:

```
app/
  blog/
    [slug]/
      page.tsx
```

This means URLs like `/blog/hello-world`, `/blog/my-post`, etc.

Without `generateStaticParams`:
- First visitor to `/blog/hello-world` has to WAIT while Next.js builds that page
- Slow first load

With `generateStaticParams`:
- All blog pages are pre-built at deploy time
- Every visitor gets an instant pre-made page ⚡

---

### Basic pattern

```tsx
// app/blog/[slug]/page.tsx

// Step 1: Tell Next.js all the possible slugs
export async function generateStaticParams() {
  const posts = await fetch("https://api.example.com/posts").then(r => r.json());

  // Return an array of objects — one per page you want to pre-build
  return posts.map((post) => ({
    slug: post.slug, // matches the [slug] folder name
  }));
}

// Step 2: Use the param to fetch the specific page data
export default async function BlogPost({ params }) {
  const post = await fetch(`https://api.example.com/posts/${params.slug}`).then(r => r.json());

  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </article>
  );
}
```

**What Next.js does with this:**
```
generateStaticParams returns: [
  { slug: "hello-world" },
  { slug: "my-second-post" },
  { slug: "nextjs-tips" },
]

Next.js pre-builds:
  /blog/hello-world      ← ready at deploy
  /blog/my-second-post   ← ready at deploy
  /blog/nextjs-tips      ← ready at deploy
```

---

### Multiple dynamic params

```tsx
// app/[lang]/blog/[slug]/page.tsx

export async function generateStaticParams() {
  return [
    { lang: "en", slug: "hello-world" },
    { lang: "fr", slug: "bonjour-monde" },
    { lang: "en", slug: "second-post" },
  ];
}
```

Pre-builds: `/en/blog/hello-world`, `/fr/blog/bonjour-monde`, `/en/blog/second-post`

---

### When to use it

| Situation | Use `generateStaticParams`? |
|-----------|---------------------------|
| Blog posts | ✅ Yes |
| Product pages | ✅ Yes |
| User profile pages (`/users/[id]`) | ⚠️ Only if user count is small |
| Millions of pages | ❌ Too slow to pre-build all |
| Pages with real-time data | ❌ Use dynamic rendering instead |

> 🧠 **Memory trick:**
> `generateStaticParams` = "give me the guest list so I can prepare a seat for everyone before the party starts."
> No guest list = set up seats as people arrive (slower).

---

## Q23. What is the difference between Static Rendering, Dynamic Rendering, and Streaming?

### These are the three ways Next.js can render a page.

Think of a restaurant:
- **Static** = pre-made meal, ready in seconds 🥡
- **Dynamic** = cooked fresh for each order, takes time 🍳
- **Streaming** = courses arrive one at a time as they're ready 🍽️🍽️🍽️

---

### 1. Static Rendering — built at deploy time

```tsx
// app/about/page.tsx

export default function AboutPage() {
  return <h1>About Us</h1>; // No fetching = always static
}
```

Or with cached data:

```tsx
export default async function BlogPage() {
  const posts = await fetch("/api/posts", { cache: "force-cache" });
  // ↑ cached = Next.js pre-builds this page
  return <div>{/* posts */}</div>;
}
```

- Page is built ONCE when you deploy
- HTML is saved and served to everyone instantly
- **Fastest possible** — no server work per request
- **Best for:** marketing pages, blogs, docs, anything that doesn't change per user

---

### 2. Dynamic Rendering — built per request

```tsx
export default async function UserDashboard() {
  const user = await fetch("/api/me", {
    cache: "no-store" // ← this triggers dynamic rendering
  });
  return <div>Hello, {user.name}</div>;
}
```

Automatically becomes dynamic when you use:
- `cache: "no-store"` in fetch
- `cookies()` or `headers()` functions
- `searchParams` from URL

- Page is built fresh for EVERY request
- Server does work each time a user visits
- **Best for:** user dashboards, personalised pages, real-time data

---

### 3. Streaming — send pieces as they're ready

```tsx
import { Suspense } from "react";
import SlowComponent from "./SlowComponent";
import FastComponent from "./FastComponent";

export default function Page() {
  return (
    <div>
      <FastComponent /> {/* ← sent to browser immediately */}

      <Suspense fallback={<p>Loading slow part...</p>}>
        <SlowComponent /> {/* ← sent when it's ready, without blocking the fast part */}
      </Suspense>
    </div>
  );
}
```

- Page is sent to browser in pieces (chunks)
- Fast parts appear immediately
- Slow parts show a loading state, then appear when ready
- **Best for:** dashboards with multiple data sources, any page with some slow and some fast content

---

### Side-by-side comparison

| | Static | Dynamic | Streaming |
|--|--------|---------|-----------|
| When is it built? | At deploy | Per request | Per request, in pieces |
| Speed for user | ⚡ Instant | 🐢 Wait for all data | ⚡ Fast parts first |
| Personalised? | ❌ No | ✅ Yes | ✅ Yes |
| Best for | Blogs, marketing | Dashboards, user data | Mixed fast/slow pages |

> 🧠 **Memory trick:**
> Static = frozen meal 🥡
> Dynamic = cooked fresh 🍳
> Streaming = courses arrive one by one 🍽️

---

## Q24. What does `<Suspense>` do in Next.js? How does it work with Server Components?

### What is Suspense?

`<Suspense>` is a React component that lets you show a **fallback** (loading message/spinner) while waiting for something to load.

Without Suspense: user stares at a blank screen until EVERYTHING loads.
With Suspense: user sees parts of the page immediately, slow parts show a spinner then pop in.

---

### Basic usage

```tsx
import { Suspense } from "react";

export default function Page() {
  return (
    <div>
      <h1>Dashboard</h1>  {/* ← shows immediately */}

      <Suspense fallback={<p>Loading user info...</p>}>
        <UserProfile />   {/* ← shows spinner until UserProfile finishes loading */}
      </Suspense>
    </div>
  );
}
```

The `fallback` is what the user sees WHILE waiting.
Once `<UserProfile />` finishes, the fallback disappears and the real content appears.

---

### How it works with Server Components

This is the magic part. When you have a Server Component that fetches slow data, wrapping it in `<Suspense>` lets the REST of the page load without waiting.

```tsx
// SlowDataComponent.tsx — Server Component that takes 3 seconds
export default async function SlowDataComponent() {
  const data = await fetch("https://slow-api.example.com/data");
  // This takes 3 seconds
  const json = await data.json();
  return <div>{json.value}</div>;
}
```

```tsx
// page.tsx
import { Suspense } from "react";
import SlowDataComponent from "./SlowDataComponent";

export default function Page() {
  return (
    <div>
      {/* This renders immediately — doesn't wait for SlowDataComponent */}
      <h1>Welcome!</h1>
      <p>This text appears instantly.</p>

      {/* This shows spinner for 3 seconds, then shows the real data */}
      <Suspense fallback={<div>Loading data...</div>}>
        <SlowDataComponent />
      </Suspense>
    </div>
  );
}
```

**What the user experiences:**
```
0 seconds:  "Welcome!" and "This text appears instantly." are visible
            "Loading data..." spinner is shown
3 seconds:  Spinner disappears, real data appears
```

Without Suspense, the whole page would be blank for 3 seconds.

---

### Multiple Suspense boundaries

You can have as many `<Suspense>` wrappers as you want — each one is independent.

```tsx
export default function Dashboard() {
  return (
    <div>
      <h1>Dashboard</h1>

      <Suspense fallback={<p>Loading profile...</p>}>
        <UserProfile />      {/* loads in 1 second */}
      </Suspense>

      <Suspense fallback={<p>Loading orders...</p>}>
        <RecentOrders />     {/* loads in 2 seconds */}
      </Suspense>

      <Suspense fallback={<p>Loading stats...</p>}>
        <StatsChart />       {/* loads in 3 seconds */}
      </Suspense>
    </div>
  );
}
```

All three load at the same time. Each one shows its spinner independently, then pops in when ready.

> 🧠 **Memory trick:**
> `<Suspense>` = a placeholder frame on the wall.
> You hang the frame immediately, it shows "loading" until the painting arrives.
> The rest of the room is ready — you don't wait to furnish the whole room for one painting.

---

## Q25. You have a dashboard with 5 data sources. Each takes different time to load. How do you prevent them from blocking each other?

### The problem

```tsx
// ❌ BAD — each fetch waits for the previous one to finish
export default async function Dashboard() {
  const user = await fetchUser();      // 1 second
  const orders = await fetchOrders();  // 2 seconds
  const stats = await fetchStats();    // 3 seconds
  // Total: 1 + 2 + 3 = 6 seconds before ANYTHING shows
}
```

The user waits 6 seconds staring at nothing.

---

### The solution — Suspense + parallel fetching

Split each data source into its own Server Component, wrap each in `<Suspense>`.

**Step 1 — Create a separate component for each data source:**

```tsx
// components/UserProfile.tsx
export default async function UserProfile() {
  const user = await fetchUser(); // 1 second
  return <div>Hello, {user.name}</div>;
}

// components/RecentOrders.tsx
export default async function RecentOrders() {
  const orders = await fetchOrders(); // 2 seconds
  return <ul>{orders.map(o => <li key={o.id}>{o.name}</li>)}</ul>;
}

// components/StatsChart.tsx
export default async function StatsChart() {
  const stats = await fetchStats(); // 3 seconds
  return <div>Revenue: {stats.revenue}</div>;
}

// components/Notifications.tsx
export default async function Notifications() {
  const notifs = await fetchNotifications(); // 1.5 seconds
  return <div>{notifs.length} new notifications</div>;
}

// components/ActivityFeed.tsx
export default async function ActivityFeed() {
  const activity = await fetchActivity(); // 2.5 seconds
  return <div>{/* activity list */}</div>;
}
```

**Step 2 — Wrap each in its own `<Suspense>` on the dashboard:**

```tsx
// app/dashboard/page.tsx
import { Suspense } from "react";
import UserProfile from "@/components/UserProfile";
import RecentOrders from "@/components/RecentOrders";
import StatsChart from "@/components/StatsChart";
import Notifications from "@/components/Notifications";
import ActivityFeed from "@/components/ActivityFeed";

export default function Dashboard() {
  return (
    <div>
      <h1>Dashboard</h1>

      {/* Each section loads independently */}
      <Suspense fallback={<Skeleton />}>
        <UserProfile />       {/* ready at 1s */}
      </Suspense>

      <Suspense fallback={<Skeleton />}>
        <Notifications />     {/* ready at 1.5s */}
      </Suspense>

      <Suspense fallback={<Skeleton />}>
        <RecentOrders />      {/* ready at 2s */}
      </Suspense>

      <Suspense fallback={<Skeleton />}>
        <ActivityFeed />      {/* ready at 2.5s */}
      </Suspense>

      <Suspense fallback={<Skeleton />}>
        <StatsChart />        {/* ready at 3s */}
      </Suspense>
    </div>
  );
}

// Simple skeleton loader
function Skeleton() {
  return (
    <div style={{
      height: "100px",
      background: "#f0f0f0",
      borderRadius: "8px",
      marginBottom: "16px"
    }} />
  );
}
```

**What the user experiences:**
```
0.0s: Dashboard heading appears, 5 skeleton loaders visible
1.0s: UserProfile pops in (skeleton disappears)
1.5s: Notifications pops in
2.0s: RecentOrders pops in
2.5s: ActivityFeed pops in
3.0s: StatsChart pops in
```

**Total wait for SOMETHING useful: 0 seconds** (heading + skeletons show instantly)
**Total wait for everything: 3 seconds** (the slowest one — same as before, but the page feels much faster)

---

### Also — kick off fetches in parallel with `Promise.all`

If two pieces of data are needed by the SAME component, use `Promise.all`:

```tsx
export default async function UserSection() {
  // ✅ Both start at the same time
  const [user, preferences] = await Promise.all([
    fetchUser(),
    fetchUserPreferences(),
  ]);

  return <div>{user.name} — Theme: {preferences.theme}</div>;
}
```

---

### The full mental model

```
Dashboard page (renders instantly — no awaits at page level)
├── <Suspense> → <UserProfile>     — fetches independently, shows when ready
├── <Suspense> → <Notifications>   — fetches independently, shows when ready
├── <Suspense> → <RecentOrders>    — fetches independently, shows when ready
├── <Suspense> → <ActivityFeed>    — fetches independently, shows when ready
└── <Suspense> → <StatsChart>      — fetches independently, shows when ready
```

> 🧠 **Memory trick:**
> Each `<Suspense>` = a separate oven.
> All 5 ovens start cooking at the same time.
> Each dish comes out when it's ready — you don't wait for the slowest dish to serve the first one.

---

## Q26. What is `unstable_cache`? How is it different from `fetch` caching?

### First — what problem does it solve?

`fetch` caching only works for HTTP requests.
But what if you're querying a **database directly**? There's no `fetch` involved.

```tsx
// This DB query has NO caching by default
const users = await db.query("SELECT * FROM users");
```

That's where `unstable_cache` comes in — it lets you cache **any async function**, not just `fetch`.

---

### How to use it

```tsx
import { unstable_cache } from "next/cache";

// Wrap your database function with unstable_cache
const getCachedUsers = unstable_cache(
  async () => {
    // This could be a DB query, external SDK, anything
    const users = await db.query("SELECT * FROM users");
    return users;
  },
  ["users-list"],     // ← cache key (must be unique)
  {
    revalidate: 60,   // ← refresh every 60 seconds
    tags: ["users"],  // ← tag for manual invalidation
  }
);

// Use it in your component
export default async function UsersPage() {
  const users = await getCachedUsers(); // ✅ result is cached
  return <ul>{users.map(u => <li key={u.id}>{u.name}</li>)}</ul>;
}
```

---

### `fetch` caching vs `unstable_cache`

| | `fetch` caching | `unstable_cache` |
|--|----------------|-----------------|
| Works with | HTTP `fetch` calls only | ANY async function |
| Database queries | ❌ No | ✅ Yes |
| External SDKs | ❌ No | ✅ Yes |
| Cache key | URL + options | You define it manually |
| Revalidation | `next: { revalidate }` | `{ revalidate }` option |
| Tag-based clearing | `next: { tags }` | `{ tags }` option |

---

### Tag-based manual invalidation

You can clear the cache on demand using tags — useful when data changes.

```tsx
// Define the cached function with a tag
const getCachedProduct = unstable_cache(
  async (id) => db.products.findOne(id),
  ["product"],
  { tags: ["products"] }
);

// Clear the cache manually when a product is updated
import { revalidateTag } from "next/cache";

export async function updateProduct(id, data) {
  "use server";
  await db.products.update(id, data);
  revalidateTag("products"); // ← clears all caches tagged "products"
}
```

---

### Why is it called `unstable_`?

The `unstable_` prefix means the API **might change** in future Next.js versions.
It's usable in production — "unstable" just means the API signature isn't finalised yet.
Think of it as "beta but works fine."

---

### Real world example — when to use it

```tsx
// ❌ No caching — database hit on EVERY page load
const posts = await db.posts.findAll();

// ✅ With unstable_cache — database hit ONCE, then cached for 5 minutes
const getCachedPosts = unstable_cache(
  () => db.posts.findAll(),
  ["all-posts"],
  { revalidate: 300 } // 5 minutes
);
const posts = await getCachedPosts();
```

> 🧠 **Memory trick:**
> `fetch` cache = automatic caching for web requests
> `unstable_cache` = manual caching wrapper for ANYTHING else (DB, SDK, files)
> If there's no `fetch`, reach for `unstable_cache`.

---

## 🗺️ Section 3 — Big Picture Summary

```
Ways to get data in Next.js
│
├── fetch() in Server Component       ← most common
│     ├── cache: "force-cache"        ← static, never refetch
│     ├── cache: "no-store"           ← fresh every request
│     └── next: { revalidate: 60 }   ← refresh every 60 seconds (ISR)
│
├── Direct DB query in Server Component
│     └── wrap with unstable_cache()  ← to cache non-fetch data
│
└── useEffect + fetch in Client Component
      └── for browser-only data (user interactions, live updates)
```

| Concept | One-line summary |
|---------|-----------------|
| `force-cache` | Cache once, use forever ❄️ |
| `no-store` | Fresh every time 🔄 |
| `revalidate: 60` | Expire and rebuild every 60s ⏱️ |
| ISR | Static page with a refresh timer |
| `generateStaticParams` | Pre-build all dynamic routes at deploy |
| Static rendering | Page built at deploy time ⚡ |
| Dynamic rendering | Page built fresh per request 🍳 |
| Streaming | Send page in chunks as ready 📦 |
| `<Suspense>` | Show fallback while slow part loads |
| `unstable_cache` | Cache any async function, not just fetch |

> 🧠 **Golden rule:**
> Default to `force-cache` (static) for speed.
> Opt into `no-store` (dynamic) only when data must be fresh per user.
> Use `revalidate` when data changes occasionally but not constantly.
> Use `<Suspense>` to prevent slow parts from blocking fast parts.


---

## Section 4: Protected Routes & Middleware (Q27–34)

# Next.js Interview Questions — Section 4: Middleware & Authentication (Q27–34)

> 🧠 **Big picture before you start:**
> Middleware is like a **security guard at the entrance** of your building.
> Every request (visitor) passes through the guard BEFORE reaching any room (page).
> The guard can check IDs, turn people away, redirect them, or let them through.
> This section is all about that guard — how to set it up, what it can do, and common patterns.

---

## Q27. What is `middleware.ts`? Where do you place it? When does it run?

### What is it?

`middleware.ts` is a special file that runs **before every request** reaches your pages.

It sits in the middle — between the user's request and your app's response.
That's why it's called **middleware** — it's in the middle.

You can use it to:
- Check if a user is logged in
- Redirect to a different page
- Rewrite the URL
- Add/modify headers
- Block certain requests

---

### Where to place it

**At the root of your project** — same level as `app/`, NOT inside `app/`.

```
my-nextjs-app/
  app/              ← your pages live here
  public/           ← static files
  middleware.ts     ← RIGHT HERE, at root level ✅
  next.config.js
  package.json
```

```
❌ Wrong — inside app/
app/
  middleware.ts     ← won't work here

✅ Correct — at root
middleware.ts       ← works here
```

---

### When does it run?

Before **every single request** — before the page renders, before any data is fetched.

```
User requests /dashboard
       ↓
middleware.ts runs first   ← checks token, verifies auth
       ↓
If allowed → page.tsx renders
If blocked → redirect to /login
```

The order is:
```
Request → middleware → (matched route) → page/layout/api
```

---

### Basic structure

```ts
// middleware.ts (at project root)
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export function middleware(request: NextRequest) {
  // request = everything about the incoming request
  // NextResponse = what you send back

  console.log("Someone visited:", request.nextUrl.pathname);

  // Just let the request through (do nothing)
  return NextResponse.next();
}
```

`NextResponse.next()` = "okay, let them through to the page"

---

### The `config` export — control which routes it runs on

Without a config, middleware runs on EVERY request (including images, CSS, fonts).
This is wasteful. Use `matcher` to only run on specific paths.

```ts
// middleware.ts
export function middleware(request: NextRequest) {
  // your logic
}

// Tell Next.js which paths to run middleware on
export const config = {
  matcher: ["/dashboard/:path*", "/admin/:path*"],
};
```

We'll cover `matcher` in detail in Q29.

> 🧠 **Memory trick:**
> `middleware.ts` = the bouncer at the club entrance.
> Placed at the ROOT of the project.
> Runs BEFORE every request reaches the page.
> Returns `next()` (let through), `redirect()` (send elsewhere), or `rewrite()` (secretly change destination).

---

## Q28. How do you protect a route so only logged-in users can access `/dashboard`?

### The pattern

Check if the user has a valid auth token/cookie.
If yes → let them through.
If no → redirect to `/login`.

---

### Step by step

**Step 1 — When user logs in, save a token in a cookie:**

```ts
// app/api/login/route.ts
import { NextResponse } from "next/server";

export async function POST(request: Request) {
  const { email, password } = await request.json();

  // Verify credentials (simplified)
  const user = await verifyUser(email, password);

  if (!user) {
    return NextResponse.json({ error: "Invalid credentials" }, { status: 401 });
  }

  const response = NextResponse.json({ success: true });

  // Save token in a cookie
  response.cookies.set("auth-token", user.token, {
    httpOnly: true,    // ← browser JS cannot read this (more secure)
    secure: true,      // ← only sent over HTTPS
    maxAge: 60 * 60 * 24 * 7, // ← expires in 7 days (in seconds)
    path: "/",
  });

  return response;
}
```

**Step 2 — Middleware checks for that cookie on every request:**

```ts
// middleware.ts
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export function middleware(request: NextRequest) {
  // Read the auth token cookie
  const token = request.cookies.get("auth-token")?.value;

  // Check if user is trying to access a protected route
  const isProtectedRoute = request.nextUrl.pathname.startsWith("/dashboard");

  if (isProtectedRoute && !token) {
    // No token → redirect to login
    return NextResponse.redirect(new URL("/login", request.url));
  }

  // Token exists OR not a protected route → let through
  return NextResponse.next();
}

export const config = {
  matcher: ["/dashboard/:path*"],
};
```

---

### What happens step by step

```
Scenario 1 — Not logged in, visits /dashboard:
  Request → middleware checks cookie → no cookie found
  → redirect("/login")
  → user never sees the dashboard ✅

Scenario 2 — Logged in, visits /dashboard:
  Request → middleware checks cookie → cookie found ✅
  → NextResponse.next()
  → dashboard page renders ✅

Scenario 3 — Visits /login (public route):
  matcher doesn't include /login
  → middleware doesn't run at all
  → login page shows ✅
```

---

### Also redirect with the original URL (better UX)

After login, redirect the user BACK to where they were trying to go.

```ts
if (isProtectedRoute && !token) {
  const loginUrl = new URL("/login", request.url);

  // Save where they were trying to go
  loginUrl.searchParams.set("callbackUrl", request.nextUrl.pathname);

  return NextResponse.redirect(loginUrl);
  // Redirects to: /login?callbackUrl=/dashboard
}
```

Then after login, read `callbackUrl` and redirect there.

> 🧠 **Memory trick:**
> Check the cookie → no cookie = turn away → send to `/login`.
> It's like a wristband check at an event. No wristband = back to the ticket desk.

---

## Q29. What is the `matcher` config? Write a matcher that protects `/dashboard` and `/admin` but leaves `/login` and `/` public.

### What is `matcher`?

`matcher` tells middleware **which URL paths to run on**.

Without it → middleware runs on EVERY request (images, CSS, fonts, everything — wasteful).
With it → middleware ONLY runs on the paths you specify.

---

### Basic matcher syntax

```ts
export const config = {
  matcher: ["/dashboard"],            // exact path only
  matcher: ["/dashboard/:path*"],     // /dashboard AND everything under it
  matcher: ["/dashboard/:path*", "/admin/:path*"], // multiple paths
};
```

The `:path*` part means "match any path after this".

```
/dashboard/:path* matches:
  /dashboard
  /dashboard/settings
  /dashboard/settings/profile
  /dashboard/anything/deep/nested
```

---

### The answer — protect `/dashboard` and `/admin`, leave `/login` and `/` public

```ts
// middleware.ts
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export function middleware(request: NextRequest) {
  const token = request.cookies.get("auth-token")?.value;

  if (!token) {
    return NextResponse.redirect(new URL("/login", request.url));
  }

  return NextResponse.next();
}

export const config = {
  matcher: [
    "/dashboard/:path*",  // protects /dashboard and all sub-routes
    "/admin/:path*",      // protects /admin and all sub-routes
    // /login is NOT listed here → middleware never runs on it ✅
    // / (homepage) is NOT listed here → middleware never runs on it ✅
  ],
};
```

Since `/login` and `/` are not in the matcher, middleware never runs on those paths at all. They're naturally public.

---

### Advanced matcher — using regex to exclude static files

```ts
export const config = {
  matcher: [
    /*
     * Match everything EXCEPT:
     * - _next/static (static files)
     * - _next/image (image optimization)
     * - favicon.ico
     * - public folder files
     */
    "/((?!_next/static|_next/image|favicon.ico|.*\\.png$).*)",
  ],
};
```

This is the "run on everything except assets" pattern. We'll cover this more in Q34.

---

### Matcher quick reference

| Pattern | Matches |
|---------|---------|
| `"/dashboard"` | Only `/dashboard` exactly |
| `"/dashboard/:path*"` | `/dashboard` and all sub-paths |
| `["/dashboard/:path*", "/admin/:path*"]` | Both dashboard and admin trees |
| `"/((?!login).*)"` | Everything except `/login` (regex) |

> 🧠 **Memory trick:**
> `matcher` = the guest list.
> If a path is on the list → middleware runs.
> If it's not on the list → middleware skips it completely.

---

## Q30. What is the difference between protecting routes in middleware vs inside the page component itself?

### Two ways to protect a route

Both work, but they're very different in HOW and WHEN they run.

---

### Option 1 — Protect in middleware (recommended)

```ts
// middleware.ts
export function middleware(request: NextRequest) {
  const token = request.cookies.get("auth-token")?.value;

  if (!token) {
    return NextResponse.redirect(new URL("/login", request.url));
  }

  return NextResponse.next();
}
```

- Runs at the **edge** — before the page even starts loading
- User is redirected BEFORE any page code runs
- No flash of protected content
- Works for ALL types of pages (Server, Client, API routes)
- One place to manage all protection logic

---

### Option 2 — Protect inside the page component

```tsx
// app/dashboard/page.tsx
import { redirect } from "next/navigation";
import { getUser } from "@/lib/auth";

export default async function DashboardPage() {
  const user = await getUser(); // check auth inside the page

  if (!user) {
    redirect("/login"); // redirect from inside the page
  }

  return <div>Dashboard for {user.name}</div>;
}
```

- Runs after the page starts rendering
- Page code runs before the redirect
- Could briefly show page content (flash) before redirect
- You have to add this check to EVERY protected page
- But you have full access to page context, params, etc.

---

### Side-by-side comparison

| | Middleware | Page component |
|--|-----------|---------------|
| When it runs | Before page loads | During page render |
| Flash of content | ❌ None | ⚠️ Possible briefly |
| Where to add | One file for all routes | Every protected page |
| Access to page data | ❌ Limited (no params) | ✅ Full access |
| Performance | ⚡ Faster (edge) | Slightly slower |
| Best for | Route-level protection | Fine-grained, data-aware checks |

---

### When to use each

**Use middleware when:**
- Simple auth check (logged in or not)
- Protecting entire sections (`/dashboard/*`, `/admin/*`)
- You want one central place for protection

**Use page-level check when:**
- You need to check user ROLE (admin vs regular user)
- The check depends on route params (`/posts/[id]` — does user own this post?)
- You need database data to decide access

---

### Best practice — use BOTH together

```ts
// middleware.ts — first line of defence
// Just checks if token exists
export function middleware(request: NextRequest) {
  const token = request.cookies.get("auth-token")?.value;
  if (!token) return NextResponse.redirect(new URL("/login", request.url));
  return NextResponse.next();
}
```

```tsx
// app/admin/page.tsx — second line of defence
// Checks if user has ADMIN role
export default async function AdminPage() {
  const user = await getUser();
  if (user.role !== "admin") redirect("/dashboard"); // not admin → back to dashboard
  return <div>Admin Panel</div>;
}
```

> 🧠 **Memory trick:**
> Middleware = front door security (basic ID check, fast)
> Page protection = inner room check (detailed access control)
> Real buildings have both — a front door guard AND a keycard for specific rooms.

---

## Q31. How do you read a cookie inside middleware? How do you set a cookie in middleware?

### Reading a cookie

```ts
// middleware.ts
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export function middleware(request: NextRequest) {
  // Method 1 — get a specific cookie by name
  const token = request.cookies.get("auth-token");
  console.log(token);
  // → { name: "auth-token", value: "abc123..." }

  // Method 2 — get just the value
  const tokenValue = request.cookies.get("auth-token")?.value;
  console.log(tokenValue);
  // → "abc123..."

  // Method 3 — get all cookies
  const allCookies = request.cookies.getAll();
  // → [{ name: "auth-token", value: "..." }, { name: "theme", value: "dark" }]

  // Method 4 — check if cookie exists
  const hasToken = request.cookies.has("auth-token");
  // → true or false

  return NextResponse.next();
}
```

---

### Setting a cookie

You set cookies on the **response**, not the request.

```ts
export function middleware(request: NextRequest) {
  const response = NextResponse.next();

  // Set a cookie on the response
  response.cookies.set("theme", "dark", {
    httpOnly: false,    // browser JS can read this (okay for theme)
    secure: true,       // HTTPS only
    maxAge: 60 * 60 * 24 * 365, // 1 year in seconds
    path: "/",          // available on all routes
    sameSite: "lax",    // CSRF protection
  });

  return response;
}
```

---

### Setting a cookie AND redirecting

```ts
export function middleware(request: NextRequest) {
  // Example: set a "visited" cookie then redirect

  const redirectResponse = NextResponse.redirect(new URL("/welcome", request.url));

  // Set cookie on the redirect response
  redirectResponse.cookies.set("first-visit", "true", {
    maxAge: 60 * 60 * 24, // 1 day
  });

  return redirectResponse;
}
```

---

### Deleting a cookie

```ts
export function middleware(request: NextRequest) {
  const response = NextResponse.next();

  // Delete a cookie (set it with maxAge: 0)
  response.cookies.delete("auth-token");

  return response;
}
```

---

### Common cookie options explained

| Option | What it does | Example |
|--------|-------------|---------|
| `httpOnly: true` | Browser JS can't read it (safer for tokens) | Auth tokens |
| `secure: true` | Only sent over HTTPS | Production apps |
| `maxAge: 3600` | Expires after 3600 seconds (1 hour) | Session length |
| `path: "/"` | Available on all routes | Most cookies |
| `sameSite: "lax"` | CSRF protection | Most cookies |

> 🧠 **Memory trick:**
> Read cookies from `request.cookies.get("name")`
> Set cookies on `response.cookies.set("name", "value")`
> Request = what came IN. Response = what goes OUT.

---

## Q32. You are using JWT for auth. The token is in an httpOnly cookie. How does middleware verify the user is logged in?

### First — what is JWT?

JWT = **JSON Web Token**.

It's a string that looks like this:
```
eyJhbGciOiJIUzI1NiJ9.eyJ1c2VySWQiOiIxMjMifQ.abc123xyz
```

It has 3 parts separated by dots:
```
HEADER.PAYLOAD.SIGNATURE

Header:    algorithm used to sign
Payload:   the actual data (userId, role, expiry)
Signature: proof it hasn't been tampered with
```

When you decode the payload, you get:
```json
{
  "userId": "123",
  "role": "admin",
  "exp": 1735000000
}
```

---

### Why httpOnly cookie?

An `httpOnly` cookie **cannot be read by browser JavaScript**.
This protects against XSS attacks — even if an attacker injects JS into your page, they can't steal the token.

The browser automatically sends it with every request, so middleware CAN read it.

---

### The verification pattern

```ts
// middleware.ts
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";
import { jwtVerify } from "jose"; // lightweight JWT library that works in middleware

const SECRET = new TextEncoder().encode(process.env.JWT_SECRET);
// ↑ your secret key from .env — used to verify the signature

export async function middleware(request: NextRequest) {
  // Step 1: read the token from the cookie
  const token = request.cookies.get("auth-token")?.value;

  // Step 2: if no token, redirect to login immediately
  if (!token) {
    return NextResponse.redirect(new URL("/login", request.url));
  }

  // Step 3: verify the token is real and not expired
  try {
    const { payload } = await jwtVerify(token, SECRET);
    // ↑ this throws an error if:
    //   - token was tampered with
    //   - token is expired
    //   - wrong signature

    // Step 4: token is valid — let the request through
    // Optionally pass user info to the page via headers
    const response = NextResponse.next();
    response.headers.set("x-user-id", payload.userId as string);
    return response;

  } catch (error) {
    // Step 5: token is invalid or expired — clear it and redirect
    const response = NextResponse.redirect(new URL("/login", request.url));
    response.cookies.delete("auth-token"); // clean up the bad cookie
    return response;
  }
}

export const config = {
  matcher: ["/dashboard/:path*", "/admin/:path*"],
};
```

---

### Step by step flow

```
User visits /dashboard
       ↓
middleware runs
       ↓
reads "auth-token" cookie
       ↓
token missing? → redirect /login
       ↓
jwtVerify(token, SECRET)
       ↓
verification fails? (expired / tampered) → delete cookie + redirect /login
       ↓
verification passes → NextResponse.next() → page renders ✅
```

---

### Important — why use `jose` instead of `jsonwebtoken`?

`jsonwebtoken` (the popular library) uses Node.js APIs that are NOT available in Next.js middleware.
Middleware runs on the **Edge Runtime** — a lightweight environment without full Node.js.
`jose` is written for edge environments, so it works in middleware.

```
# Install it
npm install jose
```

---

### The `.env` file

```env
# .env.local
JWT_SECRET=your-super-secret-key-min-32-characters-long
```

```ts
// Access in middleware
const SECRET = new TextEncoder().encode(process.env.JWT_SECRET);
```

> 🧠 **Memory trick:**
> 1. Get cookie → 2. No cookie? Redirect → 3. Verify with `jwtVerify` → 4. Error? Redirect → 5. Valid? Let through.
> Like a stamp on your hand at a club — check for stamp → verify it's real → let in or turn away.

---

## Q33. What is the difference between `NextResponse.redirect()` and `NextResponse.rewrite()` in middleware?

### One-line difference

- `redirect()` → browser KNOWS it's being sent elsewhere (URL changes in browser bar)
- `rewrite()` → browser DOESN'T KNOW (URL stays the same, different content served)

---

### `NextResponse.redirect()` — visible redirect

```ts
return NextResponse.redirect(new URL("/login", request.url));
```

**What happens:**
1. Browser sends request to `/dashboard`
2. Middleware returns a redirect response (HTTP 307)
3. Browser sees the redirect
4. **Browser URL changes to `/login`**
5. Browser makes a new request to `/login`

```
Browser bar: /dashboard → /login   (URL CHANGES ✅)
```

**Use for:**
- Sending unauthenticated users to login
- Sending users to a different page intentionally
- Any time you WANT the user to know they were redirected

---

### `NextResponse.rewrite()` — invisible rewrite

```ts
return NextResponse.rewrite(new URL("/home", request.url));
```

**What happens:**
1. Browser sends request to `/`
2. Middleware rewrites it to `/home` internally
3. Next.js serves the content of `/home`
4. **Browser URL still shows `/`** — user doesn't know

```
Browser bar: / stays as /   (URL DOES NOT CHANGE)
But content from: /home
```

**Use for:**
- A/B testing (show different page to different users, same URL)
- Showing maintenance page without changing URL
- Internationalisation (show `/en/home` content at `/home` based on locale)
- Hiding internal URL structure

---

### Real world examples

```ts
export function middleware(request: NextRequest) {
  const token = request.cookies.get("auth-token")?.value;
  const locale = request.cookies.get("locale")?.value || "en";
  const isMaintenance = process.env.MAINTENANCE_MODE === "true";

  // REDIRECT — user needs to know they're going to login
  if (!token && request.nextUrl.pathname.startsWith("/dashboard")) {
    return NextResponse.redirect(new URL("/login", request.url));
  }

  // REWRITE — serve different locale content, URL stays the same
  // User visits /about → serves /en/about (URL stays /about)
  if (!request.nextUrl.pathname.startsWith(`/${locale}`)) {
    return NextResponse.rewrite(
      new URL(`/${locale}${request.nextUrl.pathname}`, request.url)
    );
  }

  // REWRITE — maintenance mode (URL stays the same)
  if (isMaintenance) {
    return NextResponse.rewrite(new URL("/maintenance", request.url));
  }

  return NextResponse.next();
}
```

---

### Side-by-side comparison

| | `redirect()` | `rewrite()` |
|--|-------------|-------------|
| Browser URL changes? | ✅ Yes | ❌ No |
| HTTP status | 307 (temporary redirect) | 200 (normal) |
| User can see destination? | ✅ Yes | ❌ No |
| Use for | Auth redirect, page moves | A/B test, i18n, maintenance |
| Browser makes 2nd request? | ✅ Yes | ❌ No (1 request, different content) |

> 🧠 **Memory trick:**
> `redirect()` = "You need to go to a different room. Here's the new door." (URL changes)
> `rewrite()` = "I'll serve you what you need, but the room number stays the same." (URL stays)

---

## Q34. Your middleware runs on every request including `_next/static` files. How do you fix this?

### Why this is a problem

When middleware runs on EVERY request, it also runs on:
- `/_next/static/chunks/main.js` (JavaScript bundles)
- `/_next/image?url=...` (optimised images)
- `/favicon.ico`
- Any file in your `/public` folder

These requests don't need auth checks. Running middleware on them:
- Wastes server time
- Slows down page loads
- Can break static asset loading

---

### The fix — use a `matcher` that excludes static files

**Option 1 — Simple: only run on specific paths you care about**

```ts
// middleware.ts
export const config = {
  matcher: [
    "/dashboard/:path*",
    "/admin/:path*",
    "/profile/:path*",
  ],
};
```

Only runs on these paths. Everything else (static files, homepage, login) is untouched.

---

**Option 2 — Run on everything EXCEPT static files (more flexible)**

```ts
export const config = {
  matcher: [
    /*
     * Match all request paths EXCEPT:
     * - _next/static  (Next.js static files — JS, CSS)
     * - _next/image   (image optimisation endpoint)
     * - favicon.ico   (browser tab icon)
     * - Files with extensions (.png, .jpg, .svg, etc.)
     */
    "/((?!_next/static|_next/image|favicon.ico|.*\\.(?:svg|png|jpg|jpeg|gif|webp)$).*)",
  ],
};
```

This single regex pattern says: "match everything EXCEPT these patterns."

Breaking down the regex:
```
/(                     ← start matching from /
  (?!                  ← but NOT if it starts with:
    _next/static|      ←   Next.js static assets
    _next/image|       ←   image optimization
    favicon.ico|       ←   favicon
    .*\\.              ←   any file with an extension
    (?:svg|png|jpg|jpeg|gif|webp)$  ← these image extensions
  )
.*)/                   ← match the rest of the path
```

---

**Option 3 — Check inside middleware with an early return**

```ts
export function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl;

  // Skip static files manually
  if (
    pathname.startsWith("/_next/static") ||
    pathname.startsWith("/_next/image") ||
    pathname.includes("/favicon.ico") ||
    pathname.match(/\.(png|jpg|jpeg|gif|svg|webp|ico|css|js)$/)
  ) {
    return NextResponse.next(); // skip — do nothing
  }

  // Your actual middleware logic below
  const token = request.cookies.get("auth-token")?.value;
  if (!token && pathname.startsWith("/dashboard")) {
    return NextResponse.redirect(new URL("/login", request.url));
  }

  return NextResponse.next();
}
```

---

### Which option to use?

| Situation | Best option |
|-----------|------------|
| Only protecting a few routes | Option 1 — explicit matcher list |
| Running middleware on all pages but skipping assets | Option 2 — regex matcher |
| Complex logic where you need to decide inside the function | Option 3 — early return inside middleware |

**Option 1 is the simplest and recommended for most cases.** Only add complexity when you need it.

---

### Full production-ready middleware template

```ts
// middleware.ts
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";
import { jwtVerify } from "jose";

const SECRET = new TextEncoder().encode(process.env.JWT_SECRET);

export async function middleware(request: NextRequest) {
  const token = request.cookies.get("auth-token")?.value;

  if (!token) {
    return NextResponse.redirect(new URL("/login", request.url));
  }

  try {
    await jwtVerify(token, SECRET);
    return NextResponse.next();
  } catch {
    const response = NextResponse.redirect(new URL("/login", request.url));
    response.cookies.delete("auth-token");
    return response;
  }
}

// ✅ Only runs on dashboard and admin — static files are untouched
export const config = {
  matcher: [
    "/dashboard/:path*",
    "/admin/:path*",
  ],
};
```

> 🧠 **Memory trick:**
> Middleware on static files = asking every delivery truck to show ID at the front gate.
> Use `matcher` to only check the cars going to the office — let delivery trucks through the side gate.

---

## 🗺️ Section 4 — Big Picture Summary

```
Incoming request
      ↓
middleware.ts (runs first, at the edge)
      ↓
matcher config — does this path match?
  No  → skip middleware, go to page
  Yes → run middleware function
        ↓
        Read cookie (request.cookies.get)
        Verify JWT (jwtVerify)
          ↓
          Valid token → NextResponse.next() → page renders
          No token   → NextResponse.redirect("/login")
          Bad token  → delete cookie + redirect("/login")
```

| Concept | One-line summary |
|---------|-----------------|
| `middleware.ts` location | Root of project, NOT inside `app/` |
| When it runs | Before every matched request |
| `matcher` | Limits which paths trigger middleware |
| Middleware vs page protection | Middleware = fast, one place. Page = detailed, per-page |
| Read cookie | `request.cookies.get("name")?.value` |
| Set cookie | `response.cookies.set("name", "value", options)` |
| JWT in middleware | Use `jose` library — `jsonwebtoken` won't work (Node.js only) |
| `redirect()` | URL changes in browser bar |
| `rewrite()` | URL stays the same, different content served |
| Static files fix | Use `matcher` to only target your app routes |

> 🧠 **Golden rule:**
> Middleware is your first and fastest line of defence.
> Keep it simple — just check token exists + is valid.
> Do fine-grained checks (roles, ownership) inside the page component.

---

## Section 5: Layouts & Special Files (Q35–40)

# Next.js Interview Questions — Section 5: Special Files & Metadata (Q35–40)

> 🧠 **Big picture before you start:**
> App Router has a set of "special files" — each filename has a specific job.
> You've already seen `page.tsx`, `layout.tsx`, `loading.tsx`, `error.tsx`.
> This section goes deeper — WHY they work the way they do, the edge cases,
> and the files you haven't covered yet (`template.tsx`, `not-found.tsx`, `opengraph-image.tsx`).

---

## Q35. What is a Root Layout? What must it always contain? Can you have multiple layouts?

### What is a Root Layout?

The **Root Layout** is the single required layout at the very top of your app.
It lives at `app/layout.tsx` and wraps EVERY page in your entire application.

Think of it as the outer shell of your house — every room is inside it.
No matter which page the user visits, the root layout always renders around it.

---

### What must it always contain?

The Root Layout **must** include `<html>` and `<body>` tags.

This is required because Next.js does not add them automatically — you control the full HTML document structure.

```tsx
// app/layout.tsx — the Root Layout
export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">        {/* ← REQUIRED */}
      <body>                {/* ← REQUIRED */}
        {children}          {/* ← all pages render here */}
      </body>
    </html>
  );
}
```

**You will get an error if `<html>` or `<body>` are missing from the root layout.**

---

### What else goes in the Root Layout?

Since it wraps everything, it's the right place for things that appear on EVERY page:

```tsx
// app/layout.tsx
import "./globals.css";     // ← global styles
import Navbar from "@/components/Navbar";
import Footer from "@/components/Footer";

export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body>
        <Navbar />          {/* ← on every page */}
        <main>{children}</main>
        <Footer />          {/* ← on every page */}
      </body>
    </html>
  );
}
```

---

### Can you have multiple layouts?

**Yes — absolutely.** You can have nested layouts at any level.

```
app/
  layout.tsx          ← Root Layout (wraps everything)
  page.tsx            ← homepage
  dashboard/
    layout.tsx        ← Dashboard Layout (wraps only dashboard pages)
    page.tsx
    settings/
      page.tsx
  blog/
    layout.tsx        ← Blog Layout (wraps only blog pages)
    page.tsx
    [slug]/
      page.tsx
```

**How they nest:**

When a user visits `/dashboard/settings`:

```
Root Layout (app/layout.tsx)
  └── Dashboard Layout (app/dashboard/layout.tsx)
        └── Settings Page (app/dashboard/settings/page.tsx)
```

The user gets ALL layouts stacked together.

---

### Each layout only wraps its own section

```tsx
// app/dashboard/layout.tsx
// This layout ONLY applies to /dashboard and its children
export default function DashboardLayout({ children }) {
  return (
    <div style={{ display: "flex" }}>
      <aside>Sidebar</aside>    {/* only on dashboard pages */}
      <main>{children}</main>
    </div>
  );
}
```

```tsx
// app/blog/layout.tsx
// This layout ONLY applies to /blog and its children
export default function BlogLayout({ children }) {
  return (
    <div style={{ maxWidth: "700px", margin: "0 auto" }}>
      {children}   {/* clean reading layout for blog only */}
    </div>
  );
}
```

---

### Layouts persist across navigation

A key feature of layouts: **they do NOT re-render when navigating between child pages.**

```
User navigates: /dashboard → /dashboard/settings

Dashboard Layout stays mounted — it does NOT re-render
Only the page content (children) changes
```

This means:
- State inside a layout is preserved during navigation
- Expensive layout components don't re-run on every page change
- Animations or transitions in a layout are not interrupted

---

### Rules recap

| Rule | Detail |
|------|--------|
| `app/layout.tsx` | Required — must exist |
| Must contain | `<html>` and `<body>` tags |
| Only in root layout | `<html>` and `<body>` — nested layouts don't need them |
| Multiple layouts | ✅ Yes — one per folder level |
| Re-renders on navigation? | ❌ No — layouts persist |
| Nested layouts | All stack from outermost to innermost |

> 🧠 **Memory trick:**
> Root Layout = the outer walls of your house. Every room is inside it. Must have a roof (`<html>`) and a floor (`<body>`).
> Nested layouts = rooms with their own interior design — only applies inside that room.

---

## Q36. What is the difference between a layout and a template? When would you use `template.tsx` instead of `layout.tsx`?

### The key difference in one sentence

- `layout.tsx` → **persists** across navigation (does NOT re-render)
- `template.tsx` → **re-mounts** on every navigation (ALWAYS re-renders fresh)

---

### Layout — persists (does not re-render)

```tsx
// app/dashboard/layout.tsx
export default function DashboardLayout({ children }) {
  console.log("Dashboard Layout rendered"); // logs ONCE then never again
  return (
    <div>
      <Sidebar />
      {children}
    </div>
  );
}
```

When user navigates from `/dashboard` to `/dashboard/settings`:
- `DashboardLayout` stays mounted
- `console.log` does NOT run again
- Sidebar keeps its state (scroll position, open/closed, etc.)
- Only `{children}` (the page content) changes

---

### Template — re-mounts on every navigation

```tsx
// app/dashboard/template.tsx
export default function DashboardTemplate({ children }) {
  console.log("Template rendered"); // logs EVERY time user navigates
  return (
    <div>
      {children}
    </div>
  );
}
```

When user navigates from `/dashboard` to `/dashboard/settings`:
- `DashboardTemplate` is completely **unmounted and re-mounted**
- `console.log` runs AGAIN
- All state inside the template is reset
- `useEffect` runs again (as if it's a fresh mount)

---

### When would you actually use `template.tsx`?

Use template when you WANT things to reset or re-run on every page change:

**1. Page enter animations — every page should animate in freshly:**

```tsx
// app/template.tsx
"use client";

import { useEffect, useState } from "react";

export default function Template({ children }) {
  const [visible, setVisible] = useState(false);

  useEffect(() => {
    // Runs every time you navigate to a new page
    setVisible(true);
  }, []);

  return (
    <div style={{
      opacity: visible ? 1 : 0,
      transition: "opacity 0.3s ease"
    }}>
      {children}
    </div>
  );
}
```

If this were a layout, the animation would only run ONCE (first load). As a template, it runs on every navigation.

**2. `useEffect` that must run on every page visit:**

```tsx
// app/template.tsx
"use client";

import { useEffect } from "react";
import { trackPageView } from "@/lib/analytics";

export default function Template({ children }) {
  useEffect(() => {
    // Logs every page view — runs on every navigation ✅
    trackPageView(window.location.pathname);
  }, []);

  return <>{children}</>;
}
```

With a layout, this `useEffect` would only run once.

**3. When you want state to reset between pages:**

If a layout has `useState`, that state persists when navigating between child pages.
If you want the state to reset on navigation, use a template instead.

---

### Side-by-side comparison

| | `layout.tsx` | `template.tsx` |
|--|-------------|----------------|
| Re-renders on navigation? | ❌ No — persists | ✅ Yes — fresh every time |
| State preserved? | ✅ Yes | ❌ Resets on each navigation |
| `useEffect` re-runs? | ❌ Only on first mount | ✅ On every navigation |
| Good for | Sidebars, navbars, persistent UI | Page animations, analytics, resetting state |
| Performance | Better (no re-render) | Slightly more work per navigation |
| How common? | Very common | Rare — specific use cases |

---

### Can you use both?

Yes — you can have BOTH a `layout.tsx` AND a `template.tsx` in the same folder.
The layout wraps the template.

```
layout.tsx (outer, persists)
  └── template.tsx (inner, re-mounts)
        └── page.tsx
```

> 🧠 **Memory trick:**
> Layout = **permanent wallpaper** in a room. It's there when you enter, stays when you move to the next room.
> Template = **fresh coat of paint** every time you enter. Always applied new on each visit.

---

## Q37. How does `loading.tsx` work? What React primitive does Next.js use under the hood?

### What it does

`loading.tsx` is a special file that automatically shows a loading UI while the page content is being fetched.

You don't have to write any logic — just create the file and it shows up automatically while the page loads.

---

### How to create one

```tsx
// app/dashboard/loading.tsx
export default function Loading() {
  return (
    <div>
      <p>Loading dashboard...</p>
    </div>
  );
}
```

```tsx
// app/dashboard/page.tsx
export default async function DashboardPage() {
  const data = await fetchSlowData(); // takes 2 seconds
  return <div>{data.value}</div>;
}
```

**What happens:**
- User visits `/dashboard`
- `loading.tsx` content shows IMMEDIATELY
- `page.tsx` fetches data in the background (2 seconds)
- When data is ready, `loading.tsx` disappears and page content shows

---

### The React primitive under the hood — `<Suspense>`

Next.js automatically wraps your `page.tsx` in a `<Suspense>` boundary.
`loading.tsx` becomes the `fallback` prop of that `<Suspense>`.

You never write this yourself — Next.js does it for you:

```tsx
// What Next.js does automatically (you don't write this):
<Suspense fallback={<Loading />}>   {/* ← your loading.tsx */}
  <DashboardPage />                 {/* ← your page.tsx */}
</Suspense>
```

So `loading.tsx` = the `fallback` of an auto-generated `<Suspense>`.

---

### `loading.tsx` applies to the whole route segment

```
app/
  dashboard/
    loading.tsx    ← shows while ANY page under /dashboard loads
    page.tsx
    settings/
      page.tsx     ← also covered by dashboard's loading.tsx
      loading.tsx  ← OR you can add a specific one for /settings
```

If both exist, the more specific one (settings/loading.tsx) takes priority.

---

### What `loading.tsx` does NOT do

It does NOT help with parts of a page that load at different speeds.

```tsx
// ❌ loading.tsx won't help here — the whole page waits for BOTH fetches
export default async function Page() {
  const fast = await fetchFast(); // 0.5s
  const slow = await fetchSlow(); // 3s — BLOCKS everything
  return <div>{fast} {slow}</div>;
}
```

For this, you need manual `<Suspense>` wrapping (covered in Q24/Q25).

---

### Instant loading states

A great pattern is to show a **skeleton** (greyed out placeholder shapes) while loading:

```tsx
// app/dashboard/loading.tsx
export default function Loading() {
  return (
    <div>
      {/* Skeleton for a user card */}
      <div style={{ height: 40, background: "#f0f0f0", borderRadius: 8, marginBottom: 12 }} />
      <div style={{ height: 20, background: "#f0f0f0", borderRadius: 8, width: "60%" }} />
      <div style={{ height: 20, background: "#f0f0f0", borderRadius: 8, marginTop: 8, width: "40%" }} />
    </div>
  );
}
```

> 🧠 **Memory trick:**
> `loading.tsx` = automatic `<Suspense fallback={...}>` wrapper.
> Just create the file with your loading UI — Next.js does the `<Suspense>` wiring for you.
> Under the hood: `Suspense` is the primitive. `loading.tsx` is the shortcut.

---

## Q38. How does `error.tsx` work? Why must it be a Client Component? What props does it receive?

### What it does

`error.tsx` is an automatic error boundary for a route segment.

If ANY component within that segment throws an error (during render, data fetching, anything), `error.tsx` catches it and shows a friendly error UI instead of crashing the whole page.

---

### The React primitive under the hood — Error Boundary

Next.js wraps your page in a React Error Boundary automatically.
`error.tsx` becomes that error boundary's fallback UI.

```tsx
// What Next.js does automatically (you don't write this):
<ErrorBoundary fallback={<ErrorPage error={error} reset={reset} />}>
  <DashboardPage />
</ErrorBoundary>
```

---

### Basic `error.tsx`

```tsx
// app/dashboard/error.tsx
"use client"; // ← REQUIRED — must be Client Component

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  return (
    <div>
      <h2>Something went wrong!</h2>
      <p>{error.message}</p>
      <button onClick={reset}>Try again</button>
    </div>
  );
}
```

---

### Why MUST it be a Client Component?

Two reasons:

**Reason 1 — Error Boundaries are a React CLIENT feature**

React Error Boundaries only exist in the browser. They rely on React's component lifecycle (`componentDidCatch`, `getDerivedStateFromError`) which are client-side concepts.

Server Components render once and send HTML. They don't have lifecycle methods.

**Reason 2 — The `reset` prop needs interactivity**

`reset()` is a function that retries rendering the component.
To call a function on a button click, you need `onClick` — which only works in a Client Component.

```tsx
// ❌ Can't do this in a Server Component
<button onClick={reset}>Try again</button>

// ✅ Works only in a Client Component ("use client")
<button onClick={reset}>Try again</button>
```

---

### Props it receives

`error.tsx` receives exactly two props:

**`error` — the Error object**

```ts
error: Error & { digest?: string }
```

| Property | What it is |
|----------|-----------|
| `error.message` | Human-readable error message |
| `error.name` | Error type (e.g. "TypeError") |
| `error.digest` | Server-side error hash (for matching server logs) — only in production |

```tsx
export default function Error({ error, reset }) {
  console.log(error.message);  // "Failed to fetch user data"
  console.log(error.digest);   // "abc123" (matches your server logs)
  // ...
}
```

**`reset` — a function to retry**

```tsx
<button onClick={reset}>Try again</button>
```

When called:
1. React unmounts the error boundary
2. Re-renders the segment that failed
3. If it works this time, the error UI disappears
4. If it fails again, `error.tsx` shows again

---

### `error.tsx` does NOT catch errors in the layout

This is an important gotcha.

```
app/
  dashboard/
    layout.tsx    ← error.tsx does NOT catch errors here
    error.tsx     ← catches errors in page.tsx and components
    page.tsx      ← errors here are caught ✅
```

`error.tsx` only catches errors in its sibling `page.tsx` and any components that page renders.
It does NOT catch errors in the layout at the same level.

**To catch layout errors, put `error.tsx` one level up:**

```
app/
  error.tsx           ← catches errors in dashboard/layout.tsx
  dashboard/
    layout.tsx
    error.tsx         ← catches errors in dashboard/page.tsx
    page.tsx
```

---

### `global-error.tsx` — catch errors in the root layout

The root layout (`app/layout.tsx`) is special — a regular `error.tsx` can't catch its errors.
For that, you need `app/global-error.tsx`:

```tsx
// app/global-error.tsx
"use client";

export default function GlobalError({ error, reset }) {
  return (
    <html>
      <body>
        <h2>Something went very wrong!</h2>
        <button onClick={reset}>Try again</button>
      </body>
    </html>
  );
}
```

Note: `global-error.tsx` must include `<html>` and `<body>` because it replaces the root layout.

> 🧠 **Memory trick:**
> `error.tsx` = automatic Error Boundary wrapper.
> Must be `"use client"` because Error Boundaries are a client/browser concept.
> Gets two props: `error` (what went wrong) and `reset` (retry button function).
> Does NOT catch layout errors — go one level up for that.

---

## Q39. What is `not-found.tsx`? How do you trigger it programmatically?

### What it does

`not-found.tsx` is the UI shown when a page or resource is not found (404).

Next.js has a built-in default 404 page, but `not-found.tsx` lets you create a custom one — either for the whole app or for a specific section.

---

### Creating a custom 404 page

**App-wide 404 (shown for any unmatched route):**

```tsx
// app/not-found.tsx
export default function NotFound() {
  return (
    <div>
      <h1>404 — Page Not Found</h1>
      <p>The page you're looking for doesn't exist.</p>
      <a href="/">Go home</a>
    </div>
  );
}
```

**Section-specific 404 (shown only for unmatched routes under `/blog`):**

```tsx
// app/blog/not-found.tsx
export default function BlogNotFound() {
  return (
    <div>
      <h1>Blog post not found</h1>
      <p>This article doesn't exist or has been removed.</p>
      <a href="/blog">Back to all posts</a>
    </div>
  );
}
```

---

### How to trigger it programmatically

Use `notFound()` from `next/navigation` inside your page to trigger the not-found UI.

```tsx
// app/blog/[slug]/page.tsx
import { notFound } from "next/navigation";

export default async function BlogPost({ params }) {
  const post = await fetchPost(params.slug);

  // If the post doesn't exist in the database, show 404
  if (!post) {
    notFound(); // ← triggers not-found.tsx immediately
  }

  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </article>
  );
}
```

**What happens:**
1. User visits `/blog/this-post-does-not-exist`
2. `fetchPost()` returns `null`
3. `notFound()` is called
4. Next.js stops rendering and shows `not-found.tsx` instead
5. HTTP status is set to 404

---

### `notFound()` vs `redirect()`

```tsx
// Use notFound() when the resource genuinely doesn't exist
if (!post) notFound();          // → 404, shows not-found.tsx

// Use redirect() when you want to send them somewhere else
if (!user) redirect("/login");  // → 307, browser goes to /login
```

---

### `not-found.tsx` is triggered automatically too

It also triggers automatically when a URL doesn't match any route.

```
User visits: /this-does-not-exist
Next.js: no route matches → show app/not-found.tsx automatically
```

---

### The closest `not-found.tsx` wins

Next.js uses the `not-found.tsx` nearest to where `notFound()` was called.

```
app/
  not-found.tsx          ← fallback for the whole app
  blog/
    not-found.tsx        ← used for /blog routes
    [slug]/
      page.tsx           ← calls notFound() here → uses blog/not-found.tsx
```

> 🧠 **Memory trick:**
> `not-found.tsx` = your custom 404 page.
> `notFound()` = the programmatic way to say "this thing doesn't exist, show the 404."
> Like a librarian saying "we don't have that book" — they don't crash, they just tell you it's not there.

---

## Q40. What is `opengraph-image.tsx`? How does Next.js handle metadata and SEO in App Router?

### First — what is metadata and SEO?

When you share a link on Twitter, WhatsApp, Slack — the platform shows a preview card:
- A title
- A description
- An image

That preview comes from **meta tags** in the HTML `<head>`.
SEO (Search Engine Optimisation) also depends on these tags — they tell Google what your page is about.

Without them your link preview looks blank. With them it looks professional.

---

### How to add metadata in App Router

**Option 1 — Static metadata (simple, for pages that don't change)**

Export a `metadata` object from `page.tsx` or `layout.tsx`:

```tsx
// app/about/page.tsx
import type { Metadata } from "next";

export const metadata: Metadata = {
  title: "About Us — My Company",
  description: "Learn about our mission and team.",
  keywords: ["company", "about", "team"],
  openGraph: {
    title: "About Us — My Company",
    description: "Learn about our mission and team.",
    url: "https://mysite.com/about",
    images: [
      {
        url: "https://mysite.com/about-og.png",
        width: 1200,
        height: 630,
        alt: "About Us page image",
      },
    ],
  },
  twitter: {
    card: "summary_large_image",
    title: "About Us",
    description: "Learn about our mission.",
    images: ["https://mysite.com/about-og.png"],
  },
};

export default function AboutPage() {
  return <div>About Us content</div>;
}
```

Next.js puts all this in the `<head>` automatically.

---

**Option 2 — Dynamic metadata (for pages with variable data)**

For pages like `/blog/[slug]` where the title depends on the post:

```tsx
// app/blog/[slug]/page.tsx
import type { Metadata } from "next";

// generateMetadata runs on the server, has access to params
export async function generateMetadata({
  params,
}: {
  params: { slug: string };
}): Promise<Metadata> {
  const post = await fetchPost(params.slug);

  if (!post) {
    return { title: "Post Not Found" };
  }

  return {
    title: `${post.title} — My Blog`,
    description: post.excerpt,
    openGraph: {
      title: post.title,
      description: post.excerpt,
      images: [{ url: post.coverImage }],
    },
  };
}

export default async function BlogPost({ params }) {
  const post = await fetchPost(params.slug);
  return <article><h1>{post.title}</h1></article>;
}
```

---

### Metadata merging — layouts + pages combine

If a layout defines metadata and a page also defines metadata, they merge together (page wins on conflicts).

```tsx
// app/layout.tsx — default metadata for the whole site
export const metadata: Metadata = {
  title: {
    default: "My Site",         // used if no page title
    template: "%s — My Site",   // page title fills %s
  },
  description: "Welcome to my site",
};
```

```tsx
// app/blog/page.tsx — this page's metadata
export const metadata: Metadata = {
  title: "Blog",   // becomes "Blog — My Site" (thanks to the template)
};
```

Result in `<head>`: `<title>Blog — My Site</title>`

---

### `opengraph-image.tsx` — auto-generated OG images

Instead of making a separate image file for every page, you can generate OG images with code.

`opengraph-image.tsx` is a special file that generates an image using React-like syntax.
Next.js renders it as an actual PNG/JPG for the OG image tag.

```tsx
// app/blog/[slug]/opengraph-image.tsx
import { ImageResponse } from "next/og";

export const size = { width: 1200, height: 630 };
export const contentType = "image/png";

export default async function OGImage({
  params,
}: {
  params: { slug: string };
}) {
  const post = await fetchPost(params.slug);

  return new ImageResponse(
    (
      // This JSX becomes the actual image
      <div
        style={{
          width: "100%",
          height: "100%",
          display: "flex",
          flexDirection: "column",
          justifyContent: "center",
          padding: "60px",
          backgroundColor: "#0f172a",
          color: "white",
        }}
      >
        <h1 style={{ fontSize: 64, fontWeight: 700 }}>{post.title}</h1>
        <p style={{ fontSize: 32, color: "#94a3b8" }}>{post.excerpt}</p>
      </div>
    ),
    { ...size }
  );
}
```

Next.js automatically:
- Generates this image at request time
- Sets the URL in the OG meta tag
- Caches and serves it

---

### Other special image files for SEO

| File | What it generates |
|------|------------------|
| `opengraph-image.tsx` | Open Graph image (Facebook, LinkedIn, WhatsApp previews) |
| `twitter-image.tsx` | Twitter-specific card image |
| `icon.tsx` | App icon (favicon) |
| `apple-icon.tsx` | Apple touch icon (iOS home screen) |

You can also just drop static files:

```
app/
  favicon.ico           ← browser tab icon (auto-detected)
  opengraph-image.png   ← static OG image (no code needed)
  twitter-image.png     ← static Twitter image
```

---

### Full SEO checklist for a page

```tsx
export const metadata: Metadata = {
  // 1. Basic
  title: "Page Title — Site Name",
  description: "150-160 character description of the page content.",

  // 2. Open Graph (Facebook, LinkedIn, WhatsApp)
  openGraph: {
    type: "website",
    url: "https://mysite.com/page",
    title: "Page Title",
    description: "Description for social sharing",
    images: [{ url: "https://mysite.com/og.png", width: 1200, height: 630 }],
    siteName: "My Site",
  },

  // 3. Twitter
  twitter: {
    card: "summary_large_image",
    title: "Page Title",
    description: "Description for Twitter",
    images: ["https://mysite.com/og.png"],
  },

  // 4. Robots (tell Google what to do)
  robots: {
    index: true,    // allow indexing
    follow: true,   // follow links on this page
  },

  // 5. Canonical URL (avoid duplicate content issues)
  alternates: {
    canonical: "https://mysite.com/page",
  },
};
```

---

### Where does metadata go in the HTML?

Next.js converts the metadata object into real HTML `<head>` tags:

```html
<!-- Generated by Next.js in <head> -->
<title>Blog — My Site</title>
<meta name="description" content="My blog posts" />
<meta property="og:title" content="Blog — My Site" />
<meta property="og:description" content="My blog posts" />
<meta property="og:image" content="https://mysite.com/blog-og.png" />
<meta name="twitter:card" content="summary_large_image" />
```

You never write these manually — just fill in the `metadata` object.

> 🧠 **Memory trick:**
> `metadata` export = fills in the `<head>` tags automatically.
> `generateMetadata` = dynamic version for pages with variable data.
> `opengraph-image.tsx` = code that generates the social media preview image.
> Think of metadata as the label on a package — it tells platforms (Google, Twitter) what's inside before they open it.

---

## 🗺️ Section 5 — Big Picture Summary

```
Special files in App Router
│
├── layout.tsx        → persistent wrapper (stays mounted across navigation)
├── template.tsx      → re-mounting wrapper (fresh on every navigation)
├── page.tsx          → the actual page content
├── loading.tsx       → Suspense fallback (auto-shown while page loads)
├── error.tsx         → Error Boundary fallback (auto-shown on errors)
├── not-found.tsx     → 404 page (shown when notFound() called or no route matches)
└── opengraph-image.tsx → auto-generated social media preview image
```

| Concept | One-line summary |
|---------|-----------------|
| Root Layout | Required at `app/layout.tsx`. Must have `<html>` and `<body>` |
| Multiple layouts | ✅ Yes — nest them per folder. They all stack |
| Layout vs template | Layout persists. Template re-mounts on every navigation |
| `loading.tsx` | Shortcut for `<Suspense fallback={...}>`. Wrapped automatically |
| `error.tsx` | Error Boundary. Must be `"use client"`. Gets `error` + `reset` props |
| `error.tsx` scope | Does NOT catch errors in its sibling layout |
| `global-error.tsx` | Catches errors in the root layout |
| `not-found.tsx` | Custom 404. Call `notFound()` to trigger it in code |
| `metadata` export | Sets `<head>` SEO tags. Use `generateMetadata` for dynamic data |
| `opengraph-image.tsx` | Code-generated social preview image |

> 🧠 **Golden rule:**
> Each special file handles one specific concern — don't try to put loading logic in error.tsx or error logic in layout.tsx.
> Next.js designed these files so each has exactly one job. Learn the job of each file and you'll always know where to put things.

---

## Section 6: Performance & Advanced (Q41–50)
# Next.js Interview Questions — Section 6: Components, APIs & Deployment (Q41–50)

> 🧠 **Big picture before you start:**
> This is the final section. It covers the practical, real-world stuff —
> built-in components that handle performance for you, how to build API endpoints,
> environment variables, deployment options, and a capstone question that ties
> everything together with a real product scenario.
> By the end of this section you have covered the full interview prep set.

---

## Q41. What is the Next.js Image component? What problems does it solve over a plain `<img>` tag?

### The problem with plain `<img>`

```html
<!-- Plain HTML img tag -->
<img src="/hero.png" alt="Hero" />
```

Problems:
- Loads the full original file size — even if it's a 4MB photo shown at 300px wide
- No lazy loading by default — loads ALL images even ones below the fold
- No format conversion — serves PNG even when WebP/AVIF would be 5x smaller
- Can cause **layout shift** — page jumps when the image loads (hurts Core Web Vitals)
- You manage everything manually

---

### The Next.js `<Image>` component

```tsx
import Image from "next/image";

export default function Page() {
  return (
    <Image
      src="/hero.png"
      alt="Hero image"
      width={800}
      height={400}
    />
  );
}
```

Under the hood, Next.js does all of this automatically:

**1. Resizes the image to what's actually needed**

If you display an image at 300px wide, Next.js serves a 300px image — not the original 4MB file.

**2. Converts to modern formats**

Automatically serves WebP or AVIF (much smaller file sizes) if the browser supports them.
Your original file stays PNG — Next.js converts on the fly.

```
Original: hero.png — 4MB
Served as: hero.webp — 280KB  (93% smaller)
```

**3. Lazy loads by default**

Images below the visible area are NOT loaded until the user scrolls near them.
This makes the initial page load much faster.

**4. Prevents layout shift**

Because you provide `width` and `height`, the browser reserves space before the image loads.
The page doesn't jump around. (This improves your Core Web Vitals / SEO score.)

**5. Priority loading for above-the-fold images**

```tsx
<Image
  src="/hero.png"
  alt="Hero"
  width={1200}
  height={600}
  priority  // ← load this FIRST, don't lazy load (use for hero/banner images)
/>
```

---

### Fill mode — for responsive images

When you don't know the exact size (background images, gallery thumbnails):

```tsx
<div style={{ position: "relative", height: "400px" }}>
  <Image
    src="/background.jpg"
    alt="Background"
    fill                        // ← fills the parent container
    style={{ objectFit: "cover" }}
  />
</div>
```

---

### Remote images — allow external sources

For images from external URLs (e.g. user profile photos from S3, Cloudinary):

```js
// next.config.js
module.exports = {
  images: {
    remotePatterns: [
      {
        protocol: "https",
        hostname: "res.cloudinary.com",
      },
      {
        protocol: "https",
        hostname: "**.amazonaws.com",  // ← all S3 subdomains
      },
    ],
  },
};
```

```tsx
<Image
  src="https://res.cloudinary.com/demo/image/upload/sample.jpg"
  alt="Profile photo"
  width={100}
  height={100}
/>
```

---

### `<img>` vs `<Image>` comparison

| Feature | `<img>` | `<Image>` |
|---------|---------|-----------|
| Auto resize | ❌ No | ✅ Yes |
| Modern formats (WebP/AVIF) | ❌ No | ✅ Auto |
| Lazy loading | ❌ Manual | ✅ Default |
| Layout shift prevention | ❌ No | ✅ Yes |
| Priority loading | ❌ Manual | ✅ `priority` prop |
| Setup needed | None | `width`/`height` or `fill` |

> 🧠 **Memory trick:**
> `<img>` = you drive manually, manage everything yourself.
> `<Image>` = self-driving car — handles speed, fuel efficiency, safety automatically.
> Always use `<Image>` in Next.js unless you have a specific reason not to.

---

## Q42. What is `next/font`? Why is it better than a `<link>` tag?

### The problem with a Google Fonts `<link>` tag

```html
<!-- Old way — Google Fonts link in <head> -->
<link href="https://fonts.googleapis.com/css2?family=Inter&display=swap" rel="stylesheet" />
```

Problems:
- Browser makes an extra network request to Google's servers
- Fonts can cause **Flash of Unstyled Text (FOUT)** — text appears in fallback font then jumps to the loaded font
- External request = privacy concerns (Google sees who visits your site)
- Slower because fonts load AFTER the page starts rendering

---

### `next/font` — fonts downloaded at build time

With `next/font`, fonts are downloaded when you BUILD the app (not when users visit).
They're hosted on YOUR server — no external request at runtime.

```tsx
// app/layout.tsx
import { Inter } from "next/font/google";

// Configure the font
const inter = Inter({
  subsets: ["latin"],       // only download the Latin character set
  weight: ["400", "700"],   // only download these weights
  display: "swap",          // show fallback font first, swap when loaded
});

export default function RootLayout({ children }) {
  return (
    <html lang="en" className={inter.className}>
      <body>{children}</body>
    </html>
  );
}
```

---

### What `next/font` does under the hood

```
At build time:
  → Downloads Inter font files from Google
  → Stores them in your app's output
  → Generates CSS with @font-face pointing to your own files

At runtime (user visits):
  → Fonts are served from YOUR server — no external request
  → Zero layout shift (Next.js calculates the exact fallback size)
```

---

### Automatic layout shift prevention

`next/font` generates a **size-adjusted fallback font** — the fallback font is scaled so it takes up almost exactly the same space as the real font.

Result: when the real font loads, the text size barely changes. No layout jump.

```tsx
const inter = Inter({
  subsets: ["latin"],
  adjustFontFallback: true, // ← default: true. Generates perfect-fit fallback
});
```

---

### Using local fonts

```tsx
import localFont from "next/font/local";

const myFont = localFont({
  src: "./fonts/MyCustomFont.woff2",
  variable: "--font-custom",   // CSS variable name
});
```

---

### Using as a CSS variable (for Tailwind)

```tsx
const inter = Inter({
  subsets: ["latin"],
  variable: "--font-inter",   // creates a CSS custom property
});

// In layout
<html className={inter.variable}>

// In Tailwind config
fontFamily: {
  sans: ["var(--font-inter)"],
}
```

---

### `<link>` vs `next/font` comparison

| Feature | `<link>` Google Fonts | `next/font` |
|---------|----------------------|-------------|
| Font hosted on | Google's CDN | Your server |
| Extra network request | ✅ Yes (slow) | ❌ No |
| Privacy (Google tracking) | ❌ Google sees users | ✅ No external request |
| Layout shift | ⚠️ Common | ✅ Prevented |
| Performance | Slower | Faster |
| Setup | Copy/paste link | Import from `next/font/google` |

> 🧠 **Memory trick:**
> `<link>` = ordering a pizza from a restaurant across town (extra trip, delay).
> `next/font` = you already have the pizza in your fridge at build time (instant, no external trip).

---

## Q43. What is the Next.js Link component? How is it different from `<a>`? What is prefetching?

### The problem with plain `<a>` tags

```html
<a href="/dashboard">Dashboard</a>
```

A plain `<a>` tag does a **full page reload** — the browser downloads the entire page (HTML, CSS, JS) from scratch. It's like closing the app and opening it again.

---

### The `<Link>` component — client-side navigation

```tsx
import Link from "next/link";

export default function Navbar() {
  return (
    <nav>
      <Link href="/dashboard">Dashboard</Link>
      <Link href="/blog">Blog</Link>
    </nav>
  );
}
```

`<Link>` does **client-side navigation** — only the page content changes.
The browser doesn't reload. The layout stays mounted. It feels instant.

---

### What `<Link>` does differently

**1. Client-side navigation (SPA-like)**
No full page reload. Only the changed parts update.

**2. Prefetching — loads the page before you click**

This is the big one.

When a `<Link>` component enters the **viewport** (becomes visible on screen), Next.js automatically downloads that page's code in the background — BEFORE you click.

```tsx
// This link, when visible on screen, silently prefetches /dashboard
<Link href="/dashboard">Go to Dashboard</Link>
```

So when you click, the page appears **instantly** — the data was already downloaded.

**3. Scroll restoration**

Navigating back restores your scroll position automatically.

**4. Prefetch control**

```tsx
// Disable prefetching (useful for rarely-visited pages)
<Link href="/admin" prefetch={false}>
  Admin Panel
</Link>
```

---

### Programmatic navigation vs Link

`<Link>` = for clickable navigation in JSX (buttons, menu items, etc.)
`useRouter().push()` = for navigation triggered by code (after form submit, after a delay, etc.)

```tsx
// Use Link for clickable UI elements
<Link href="/dashboard">Go to dashboard</Link>

// Use router.push() for code-triggered navigation
const router = useRouter();
router.push("/dashboard"); // called after login succeeds
```

---

### `<a>` vs `<Link>` comparison

| Feature | `<a>` tag | `<Link>` |
|---------|-----------|---------|
| Navigation type | Full page reload | Client-side (SPA) |
| Prefetching | ❌ No | ✅ Yes (automatic) |
| Speed | Slower | Faster |
| Layout re-renders? | ✅ Yes (full reload) | ❌ No (only content changes) |
| External links | ✅ Use `<a>` | `<Link>` works but `<a>` is conventional |

> 🧠 **Memory trick:**
> `<a>` = restart the whole car to go somewhere.
> `<Link>` = change destination while the car is already running.
> Prefetching = the GPS already downloaded the route before you typed it in.

---

## Q44. What is the difference between `next build` output modes: `static`, `server`, and `standalone`?

### These are three output types when you run `next build`.

They determine what gets generated and HOW your app can be deployed.

---

### `server` — default output (Node.js server required)

```js
// next.config.js — default, no output setting needed
module.exports = {};
```

- Generates a `.next/` folder
- Requires a **Node.js server** to run
- Supports ALL Next.js features (SSR, ISR, Server Actions, middleware, etc.)
- Deploy with: `next start`, Docker + Node.js, Vercel, Railway, Render

```
next build
next start    ← starts the Node.js server
```

**Use when:** You need all Next.js features and have a server environment.

---

### `static` — fully static export (no server)

```js
// next.config.js
module.exports = {
  output: "export",
};
```

- Generates a `out/` folder with plain HTML/CSS/JS files
- NO server required — can be hosted anywhere (S3, GitHub Pages, Netlify, CDN)
- Loses server-side features: no SSR, no API routes, no Server Actions, no Image optimization (built-in), no ISR

```
next build
# generates: out/
#   index.html
#   dashboard.html
#   blog/post-1.html
#   ...
```

**Use when:** Your app is fully static — blog, documentation, marketing site with no dynamic data per request.

---

### `standalone` — self-contained bundle (for Docker/containers)

```js
// next.config.js
module.exports = {
  output: "standalone",
};
```

- Generates a `.next/standalone/` folder with EVERYTHING needed to run
- Includes Node.js server code + only the dependencies actually used
- No need for `node_modules/` in production — much smaller Docker image
- Still requires Node.js but package is lean

```
next build
# generates: .next/standalone/

# Run it (no npm install needed):
node .next/standalone/server.js
```

**Docker example:**

```dockerfile
FROM node:18-alpine

COPY .next/standalone ./
COPY .next/static ./.next/static
COPY public ./public

EXPOSE 3000
CMD ["node", "server.js"]
```

**Use when:** Deploying to Docker / Kubernetes / your own EC2 / any container environment.

---

### Side-by-side comparison

| | `server` (default) | `static` (export) | `standalone` |
|--|-------------------|--------------------|-------------|
| Output folder | `.next/` | `out/` | `.next/standalone/` |
| Needs Node.js? | ✅ Yes | ❌ No | ✅ Yes |
| SSR / Server Actions | ✅ Yes | ❌ No | ✅ Yes |
| ISR | ✅ Yes | ❌ No | ✅ Yes |
| Image optimization | ✅ Yes | ❌ No | ✅ Yes |
| Docker-friendly | ⚠️ Large image | N/A | ✅ Optimized |
| Host on CDN/S3 | ❌ No | ✅ Yes | ❌ No |
| Best for | Vercel, Railway | Blogs, docs, marketing | Docker, EC2, Kubernetes |

> 🧠 **Memory trick:**
> `server` = full restaurant with a chef (server required)
> `static` = pre-packed lunchbox (no chef, serve anywhere)
> `standalone` = food truck with its own kitchen (portable, self-contained)

---

## Q45. What is a Route Handler (`route.ts`)? How is it different from `pages/api`?

### What is a Route Handler?

A Route Handler is how you create API endpoints in the App Router.

You create a file called `route.ts` inside the `app/` folder.
Each HTTP method (`GET`, `POST`, `PUT`, `DELETE`, etc.) is an exported function.

---

### Basic Route Handler

```ts
// app/api/users/route.ts
import { NextResponse } from "next/server";

// Handles GET /api/users
export async function GET() {
  const users = await db.users.findAll();
  return NextResponse.json(users);
}

// Handles POST /api/users
export async function POST(request: Request) {
  const body = await request.json();
  const user = await db.users.create(body);
  return NextResponse.json(user, { status: 201 });
}
```

Each exported function = one HTTP method. Clean and explicit.

---

### How it's different from `pages/api`

**Old way — `pages/api/users.ts`:**

```ts
// pages/api/users.ts — Pages Router style
import type { NextApiRequest, NextApiResponse } from "next";

export default function handler(req: NextApiRequest, res: NextApiResponse) {
  if (req.method === "GET") {
    res.json({ users: [] });
  } else if (req.method === "POST") {
    res.status(201).json({ created: true });
  }
  // Have to manually check req.method for everything
}
```

**New way — `app/api/users/route.ts`:**

```ts
// app/api/users/route.ts — App Router style
export async function GET() {
  return Response.json({ users: [] });
}

export async function POST(request: Request) {
  return Response.json({ created: true }, { status: 201 });
}
// One export per method — no if/else chains
```

---

### Key differences

| | `pages/api` | `app/route.ts` |
|--|------------|----------------|
| One function handles all methods? | ✅ Yes (check `req.method`) | ❌ No — one export per method |
| Uses Web standard `Request`/`Response`? | ❌ No (custom types) | ✅ Yes |
| Runs on Edge runtime? | ❌ No | ✅ Yes (optional) |
| Colocation with pages? | ❌ Separate folder | ✅ Can live next to your pages |
| Caching support? | ❌ No | ✅ Yes (`GET` is cacheable) |

---

### Route Handler with URL params

```ts
// app/api/users/[id]/route.ts
export async function GET(
  request: Request,
  { params }: { params: { id: string } }
) {
  const user = await db.users.findById(params.id);

  if (!user) {
    return Response.json({ error: "Not found" }, { status: 404 });
  }

  return Response.json(user);
}

export async function DELETE(
  request: Request,
  { params }: { params: { id: string } }
) {
  await db.users.delete(params.id);
  return new Response(null, { status: 204 }); // no content
}
```

---

### `route.ts` CANNOT coexist with `page.tsx` in the same folder

```
app/
  dashboard/
    page.tsx    ← ✅ fine
    route.ts    ← ❌ conflict — can't have both in same folder

  api/
    users/
      route.ts  ← ✅ fine (no page.tsx here)
```

> 🧠 **Memory trick:**
> `pages/api` = one bouncer checks all IDs at one door.
> `route.ts` = separate doors for GET, POST, DELETE — each clearly labelled.

---

## Q46. You want to create a `GET /api/users` endpoint in App Router. What file do you create and what does it look like?

### File location

```
app/
  api/
    users/
      route.ts    ← this is the file
```

URL: `/api/users`

The `api/` folder is just a convention — it's not required. You could put it anywhere in `app/`.
But `/api/...` is the standard naming everyone expects.

---

### The complete file

```ts
// app/api/users/route.ts
import { NextResponse } from "next/server";

export async function GET(request: Request) {
  try {
    // Option A — fetch from your database
    const users = await db.users.findAll();

    // Return JSON response
    return NextResponse.json(users, { status: 200 });

  } catch (error) {
    // Always handle errors
    return NextResponse.json(
      { error: "Failed to fetch users" },
      { status: 500 }
    );
  }
}
```

---

### Reading query parameters

```ts
// GET /api/users?page=2&limit=10

export async function GET(request: Request) {
  const { searchParams } = new URL(request.url);

  const page = Number(searchParams.get("page") ?? "1");
  const limit = Number(searchParams.get("limit") ?? "10");

  const users = await db.users.findMany({
    skip: (page - 1) * limit,
    take: limit,
  });

  return NextResponse.json({ users, page, limit });
}
```

---

### Adding headers (e.g. CORS)

```ts
export async function GET(request: Request) {
  const users = await db.users.findAll();

  return NextResponse.json(users, {
    status: 200,
    headers: {
      "Access-Control-Allow-Origin": "*",  // CORS
      "Cache-Control": "public, s-maxage=60", // cache for 60s
    },
  });
}
```

---

### Protecting the endpoint

```ts
export async function GET(request: Request) {
  // Check for auth token in header
  const authHeader = request.headers.get("authorization");

  if (!authHeader || !authHeader.startsWith("Bearer ")) {
    return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
  }

  const token = authHeader.split(" ")[1];
  const user = await verifyToken(token);

  if (!user) {
    return NextResponse.json({ error: "Invalid token" }, { status: 401 });
  }

  const users = await db.users.findAll();
  return NextResponse.json(users);
}
```

> 🧠 **Memory trick:**
> File: `app/api/[resource]/route.ts`
> Export: one function per HTTP method (`GET`, `POST`, `PUT`, `DELETE`)
> Return: `NextResponse.json(data, { status: 200 })`

---

## Q47. What is `revalidatePath` and `revalidateTag`? When would you call them?

### The problem they solve

You have a blog. Posts are cached (ISR or `force-cache`).
An editor publishes a new post — but the site still shows the old cached version.
How do you clear just that one page's cache without redeploying?

Answer: `revalidatePath` or `revalidateTag`.

These let you **manually clear the cache on demand** — usually called from a Server Action or an API route.

---

### `revalidatePath` — clear cache for a specific URL path

```ts
import { revalidatePath } from "next/cache";

// Clear cache for a specific page
revalidatePath("/blog");                  // clears /blog
revalidatePath("/blog/my-post-slug");     // clears one specific post
revalidatePath("/dashboard", "layout");   // clears layout + all pages under /dashboard
```

**Real example — called from a Server Action:**

```tsx
// app/actions.ts
"use server";

import { revalidatePath } from "next/cache";

export async function publishPost(postId: string) {
  // Save the post to DB
  await db.posts.publish(postId);

  // Clear the cache for the blog listing and the specific post
  revalidatePath("/blog");
  revalidatePath(`/blog/${postId}`);
  // Next visitor to /blog gets fresh data ✅
}
```

---

### `revalidateTag` — clear cache for a named group of fetches

Instead of clearing specific paths, you tag your fetches and clear all fetches with that tag at once.

**Step 1 — Tag your fetch calls:**

```tsx
// In your page or component
const posts = await fetch("https://api.example.com/posts", {
  next: { tags: ["posts"] },  // ← tag this fetch as "posts"
});

const user = await fetch("https://api.example.com/user/1", {
  next: { tags: ["user-1", "users"] },  // ← multiple tags
});
```

**Step 2 — Revalidate all fetches with that tag:**

```ts
import { revalidateTag } from "next/cache";

// In a Server Action or API route
revalidateTag("posts");    // clears EVERY fetch tagged "posts" across ALL pages
revalidateTag("user-1");   // clears only user 1's data
```

---

### `revalidatePath` vs `revalidateTag`

| | `revalidatePath` | `revalidateTag` |
|--|-----------------|----------------|
| Targets | A specific URL path | All fetches with a specific tag |
| Precision | URL-level | Data-level |
| Use when | You know which page to clear | Data is shared across multiple pages |
| Example | Clear `/blog/my-post` | Clear "posts" tag — affects every page that fetches posts |

---

### When to call them — real world patterns

**CMS webhook — when content is updated:**

```ts
// app/api/revalidate/route.ts
// Your CMS calls this URL when content changes

export async function POST(request: Request) {
  const body = await request.json();
  const secret = request.headers.get("x-revalidate-secret");

  if (secret !== process.env.REVALIDATE_SECRET) {
    return Response.json({ error: "Invalid secret" }, { status: 401 });
  }

  // Clear cache for the updated content
  revalidateTag("posts");
  revalidatePath("/blog");

  return Response.json({ revalidated: true });
}
```

**After a Server Action:**

```ts
"use server";

export async function deleteUser(userId: string) {
  await db.users.delete(userId);
  revalidatePath("/admin/users");    // refresh the admin list
  revalidateTag(`user-${userId}`);   // clear that user's cached data
}
```

> 🧠 **Memory trick:**
> `revalidatePath` = "clear the cache for this specific PAGE URL"
> `revalidateTag` = "clear the cache for anything tagged with this LABEL"
> Both are how you tell Next.js "the data changed, go get fresh data next time."

---

## Q48. How does Next.js handle environment variables? What is the difference between `NEXT_PUBLIC_` and non-prefixed variables?

### Environment variable files

```
.env                  ← loaded in all environments
.env.local            ← loaded in all environments, gitignored (for secrets)
.env.development      ← only when running next dev
.env.production       ← only when running next build / next start
```

Most of the time you only need `.env.local` for secrets and `.env` for shared config.

---

### Two types of variables

**Type 1 — Server-only (no prefix):**

```env
# .env.local
DATABASE_URL=postgresql://user:password@localhost:5432/mydb
JWT_SECRET=super-secret-key-never-share-this
STRIPE_SECRET_KEY=sk_live_abc123
```

- Available ONLY on the server
- Never sent to the browser
- Safe for secrets, API keys, database credentials

```tsx
// ✅ Works in Server Components, API routes, Server Actions
const db = new Database(process.env.DATABASE_URL);

// ❌ UNDEFINED in Client Components — value is never sent to browser
console.log(process.env.DATABASE_URL); // → undefined in browser
```

---

**Type 2 — Public (with `NEXT_PUBLIC_` prefix):**

```env
# .env.local
NEXT_PUBLIC_API_URL=https://api.mysite.com
NEXT_PUBLIC_STRIPE_PUBLIC_KEY=pk_live_abc123
NEXT_PUBLIC_APP_NAME=My App
```

- Available on BOTH server AND client (browser)
- Baked into the JavaScript bundle at build time
- Only use for non-sensitive values

```tsx
// ✅ Works EVERYWHERE — server and client
console.log(process.env.NEXT_PUBLIC_API_URL); // → "https://api.mysite.com"
```

---

### How `NEXT_PUBLIC_` variables work

At build time, Next.js replaces `process.env.NEXT_PUBLIC_*` with the actual string value.

```tsx
// What you write:
const url = process.env.NEXT_PUBLIC_API_URL;

// What Next.js bundles (in the browser JavaScript):
const url = "https://api.mysite.com";
```

The variable is literally inlined. This is why:
1. They're available in the browser (no `process.env` lookup needed)
2. They can't be changed at runtime — only at build time
3. They're not secret — anyone can see them in the browser bundle

---

### Summary table

| Variable type | Example | Available on server? | Available in browser? | Safe for secrets? |
|--------------|---------|---------------------|----------------------|------------------|
| No prefix | `DATABASE_URL` | ✅ Yes | ❌ No | ✅ Yes |
| `NEXT_PUBLIC_` | `NEXT_PUBLIC_API_URL` | ✅ Yes | ✅ Yes | ❌ No |

---

### Common mistake to avoid

```env
# ❌ DANGEROUS — NEVER do this
NEXT_PUBLIC_DATABASE_URL=postgresql://...   # Anyone can see your DB connection!
NEXT_PUBLIC_JWT_SECRET=my-secret            # Anyone can forge tokens!
NEXT_PUBLIC_STRIPE_SECRET=sk_live_...       # Anyone can charge customers!
```

**Rule:** If it's a secret → no prefix. If it's okay to be public → `NEXT_PUBLIC_`.

---

### Runtime environment variables (for Docker/production)

`NEXT_PUBLIC_` variables are baked in at BUILD time.
But server-only variables can be changed at RUNTIME (when you start the server):

```dockerfile
# Docker — set at runtime, not build time
ENV DATABASE_URL=postgresql://prod-server/mydb
CMD ["node", "server.js"]
```

This is why you don't bake secret values into the image — you inject them at runtime.

> 🧠 **Memory trick:**
> No prefix = kitchen-only ingredient (never leaves the kitchen/server)
> `NEXT_PUBLIC_` = ingredient that goes on the menu (visible to everyone)
> Never put your secret recipe on the menu.

---

## Q49. What is the difference between deploying on Vercel vs a plain Node.js server (EC2)? What changes in `next.config.js`?

### Deploying on Vercel — zero config

Vercel is made by the same team as Next.js — everything works out of the box.

```
# No next.config.js changes needed
next build   ← run by Vercel automatically
```

What Vercel handles for you:
- Edge network (CDN in 100+ locations worldwide)
- Automatic scaling (handles traffic spikes)
- ISR support (revalidation works automatically)
- Image optimization (runs as a serverless function)
- Middleware runs on the Edge (fast, near the user)
- Preview deployments for every PR
- Automatic HTTPS

You push to GitHub → Vercel builds and deploys automatically.

**Cost:** Free tier is generous. Paid plans for more builds/bandwidth.

---

### Deploying on your own EC2 / Node.js server

You manage the server yourself.

```js
// next.config.js — use standalone output for efficient Docker packaging
module.exports = {
  output: "standalone",
};
```

```bash
# On your server:
npm run build          # build the app
node .next/standalone/server.js  # start the server

# Or with PM2 (process manager — keeps it running):
pm2 start .next/standalone/server.js --name "nextjs-app"
```

What YOU handle:
- Server setup and maintenance
- Scaling (add more EC2 instances manually or with Auto Scaling)
- HTTPS / SSL certificate (use Nginx + Certbot)
- Zero-downtime deploys (blue/green, or rolling with a load balancer)
- Image optimization still works (runs in Node.js)
- ISR still works (runs in Node.js process)

---

### EC2 with Nginx (typical setup)

```nginx
# /etc/nginx/sites-available/myapp
server {
  listen 80;
  server_name mysite.com;

  location / {
    proxy_pass http://localhost:3000;  # forward to Next.js
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
  }
}
```

---

### What changes in `next.config.js` for self-hosting

```js
// next.config.js for EC2/self-hosted
module.exports = {
  output: "standalone",         // ← REQUIRED for Docker/EC2

  // Image optimization needs a domain config
  images: {
    remotePatterns: [
      { hostname: "res.cloudinary.com" },
    ],
  },

  // If behind a proxy/load balancer
  // (so Next.js knows the real IP and protocol)
  // Not needed on Vercel — handled automatically
};
```

If using a **custom image loader** (when you can't use Next.js built-in image optimization):

```js
module.exports = {
  images: {
    loader: "cloudinary",       // use Cloudinary for images
    path: "https://res.cloudinary.com/demo/",
  },
};
```

---

### Vercel vs EC2 comparison

| | Vercel | EC2 / Self-hosted |
|--|--------|-----------------|
| Setup time | Minutes | Hours (DevOps needed) |
| `next.config.js` changes | None | `output: "standalone"` |
| Scaling | Automatic | Manual / Auto Scaling |
| Cost at scale | More expensive | Cheaper at high volume |
| ISR / Image optimization | ✅ Automatic | ✅ Works (Node.js) |
| Maintenance | ❌ None | ✅ You manage everything |
| Best for | Startups, fast deploys | Cost control, compliance |

> 🧠 **Memory trick:**
> Vercel = fully serviced hotel (everything handled, premium price)
> EC2 = renting an apartment (cheaper long-term, you handle maintenance)
> `output: "standalone"` = the moving box that makes your app portable

---

## Q50. You have a MehFil-style dashboard that polls every 3 seconds for campaign status. How would you replace polling with a better approach using Next.js features?

### Understanding the problem first

**Polling** = client sends a request every 3 seconds asking "anything changed?"

```tsx
// Current polling approach (bad pattern)
"use client";

useEffect(() => {
  const interval = setInterval(async () => {
    const res = await fetch("/api/campaign-status");
    const data = await res.json();
    setStatus(data.status);
  }, 3000); // asks every 3 seconds

  return () => clearInterval(interval);
}, []);
```

Problems with this:
- 20 users = 20 × 20 requests/min = 400 requests/min for NOTHING (most return "no change")
- Wasted server resources
- 3-second delay even when status changes at second 1
- Increases server costs

---

### Better approach 1 — Server-Sent Events (SSE) with a Route Handler

SSE = the server PUSHES updates to the client when they happen.
The client opens ONE connection and waits — server sends data when ready.

```ts
// app/api/campaign-status/route.ts
export async function GET(request: Request) {
  const { searchParams } = new URL(request.url);
  const campaignId = searchParams.get("id");

  // Create a readable stream that stays open
  const stream = new ReadableStream({
    async start(controller) {
      const encoder = new TextEncoder();

      // Send an update every time status changes
      const checkStatus = async () => {
        const status = await db.campaigns.getStatus(campaignId);

        // SSE format: "data: {json}\n\n"
        const message = `data: ${JSON.stringify({ status })}\n\n`;
        controller.enqueue(encoder.encode(message));
      };

      // Check immediately
      await checkStatus();

      // Then check on DB changes (or use a real-time DB like Supabase)
      const interval = setInterval(checkStatus, 1000);

      // Clean up when client disconnects
      request.signal.addEventListener("abort", () => {
        clearInterval(interval);
        controller.close();
      });
    },
  });

  return new Response(stream, {
    headers: {
      "Content-Type": "text/event-stream",  // ← SSE content type
      "Cache-Control": "no-cache",
      "Connection": "keep-alive",
    },
  });
}
```

```tsx
// app/dashboard/CampaignStatus.tsx — Client Component
"use client";

import { useEffect, useState } from "react";

export default function CampaignStatus({ campaignId }: { campaignId: string }) {
  const [status, setStatus] = useState("loading");

  useEffect(() => {
    // ONE connection — server pushes updates as they happen
    const eventSource = new EventSource(
      `/api/campaign-status?id=${campaignId}`
    );

    eventSource.onmessage = (event) => {
      const data = JSON.parse(event.data);
      setStatus(data.status);         // update instantly when server sends
    };

    eventSource.onerror = () => {
      eventSource.close();
      setStatus("error");
    };

    // Clean up connection when component unmounts
    return () => eventSource.close();
  }, [campaignId]);

  return (
    <div>
      Campaign Status: <strong>{status}</strong>
    </div>
  );
}
```

---

### Better approach 2 — `revalidatePath` triggered by the actual status change

If the status change is triggered by a Server Action (e.g. user clicks "Launch Campaign"), just revalidate the path immediately:

```ts
// app/actions.ts
"use server";

import { revalidatePath } from "next/cache";

export async function launchCampaign(campaignId: string) {
  await db.campaigns.launch(campaignId);

  // Immediately invalidate the dashboard cache
  revalidatePath("/dashboard");
  // Next visitor (or the user who triggered it) gets fresh data instantly
}
```

```tsx
// app/dashboard/page.tsx — Server Component
export default async function Dashboard() {
  const campaigns = await db.campaigns.findAll();
  // This is always fresh after revalidatePath is called
  return <CampaignList campaigns={campaigns} />;
}
```

---

### Better approach 3 — use a realtime database (Supabase)

If you're using Supabase (Postgres + realtime), you get push updates for free:

```tsx
"use client";

import { useEffect, useState } from "react";
import { createClient } from "@supabase/supabase-js";

const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
);

export default function CampaignStatus({ campaignId }: { campaignId: string }) {
  const [status, setStatus] = useState("");

  useEffect(() => {
    // Subscribe to changes on this specific campaign row
    const channel = supabase
      .channel("campaign-status")
      .on(
        "postgres_changes",
        {
          event: "UPDATE",
          schema: "public",
          table: "campaigns",
          filter: `id=eq.${campaignId}`,
        },
        (payload) => {
          setStatus(payload.new.status); // instant push when DB row changes
        }
      )
      .subscribe();

    return () => supabase.removeChannel(channel);
  }, [campaignId]);

  return <div>Status: {status}</div>;
}
```

---

### Comparison of all approaches

| Approach | How it works | Requests | Delay | Complexity |
|----------|-------------|----------|-------|-----------|
| Polling every 3s | Client asks repeatedly | Many (wasteful) | Up to 3s | Low |
| SSE (Route Handler) | Server pushes when ready | One connection | Instant | Medium |
| `revalidatePath` | Cache cleared on action | Zero extra | Instant | Low |
| Supabase realtime | DB pushes changes | One WS connection | Instant | Low (library handles it) |

---

### Which to use when

```
Status changes because of a SERVER ACTION (user clicks button)?
  → Use revalidatePath — simplest, no extra code

Status changes because of an EXTERNAL PROCESS (background job, webhook)?
  → Use SSE Route Handler

You're using SUPABASE as your database?
  → Use Supabase realtime — easiest, built-in

Status change affects many users simultaneously (live scores, stock prices)?
  → Use SSE or WebSockets
```

> 🧠 **Memory trick:**
> Polling = sending a letter every 3 seconds asking "have you replied yet?"
> SSE = giving them your phone number — they CALL YOU when they have an answer.
> `revalidatePath` = you already know when it changes (you changed it) — just refresh the page.

---

## 🗺️ Section 6 — Big Picture Summary

```
Built-in Optimizations
├── <Image>     → auto resize, WebP, lazy load, no layout shift
├── next/font   → fonts hosted locally, no external request, no layout shift
└── <Link>      → client-side navigation + automatic prefetching

API Endpoints (App Router)
└── app/api/[resource]/route.ts
    ├── export async function GET()
    ├── export async function POST()
    └── export async function DELETE()

Build Outputs
├── server (default)  → Node.js server, all features
├── static (export)   → plain HTML, no server needed
└── standalone        → self-contained bundle for Docker/EC2

Cache Management
├── revalidatePath("/blog")        → clear cache for a URL
└── revalidateTag("posts")         → clear cache for all tagged fetches

Environment Variables
├── DATABASE_URL         → server only (never sent to browser)
└── NEXT_PUBLIC_API_URL  → browser + server (baked into bundle)
```

| Concept | One-line summary |
|---------|-----------------|
| `<Image>` | Auto-optimizes images: resize, WebP, lazy load |
| `next/font` | Hosts fonts locally at build time, zero layout shift |
| `<Link>` | Client-side nav + auto prefetching on viewport |
| `output: "standalone"` | For Docker/EC2 — self-contained bundle |
| `output: "export"` | For CDN/S3 — static HTML, no server |
| Route Handler | `route.ts` — one export per HTTP method |
| `revalidatePath` | Clear cache for a specific page URL |
| `revalidateTag` | Clear cache for all fetches with a tag |
| `NEXT_PUBLIC_` | Available in browser — not for secrets |
| No prefix env var | Server only — safe for secrets |
| Polling → SSE | Push updates instead of repeated requests |

> 🧠 **Golden rule for the whole interview prep:**
> Next.js App Router makes you think in two worlds: **server** (fast, secure, cached) and **client** (interactive, real-time).
> Push as much as possible to the server. Only go to the client when you need interactivity.
> Use built-in components (`<Image>`, `<Link>`, `next/font`) — they solve hard problems for free.

---


## Quick Cheat Sheet — Most Asked Concepts

| Topic | What to memorize |
|---|---|
| Default component type | Server Component (no "use client" = server) |
| useState/useEffect/onClick | Only in Client Components |
| Data fetching | `async` Server Components, `fetch` with cache options |
| Protected routes | `middleware.ts` at project root |
| Dynamic route | `[id]` folder → `params.id` in page props |
| Catch-all route | `[...slug]` → `params.slug` is an array |
| Route group | `(name)` folder → no URL effect, shared layout |
| Loading UI | `loading.tsx` → wraps page in `<Suspense>` automatically |
| Error UI | `error.tsx` → must be `"use client"`, receives `error` and `reset` props |
| Redirect in server | `import { redirect } from 'next/navigation'` |
| Redirect in client | `useRouter().push('/path')` |

---

