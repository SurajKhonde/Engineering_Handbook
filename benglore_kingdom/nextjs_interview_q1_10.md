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

*Next.js App Router · Section 1 complete · Q1–10*
