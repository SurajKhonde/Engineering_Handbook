# Comment Feature (Cache + Queue) — Interview Notes (Chatty Backend)

Use this as your **interview explanation script**. It’s written from an engineer POV: *what happens, why we did it, what Redis stores, and what workers run.*

---

## 0) One-liner (say this first)

> “For comments we do **fast writes** to Redis for instant UI + **async durability** via Bull workers to MongoDB. Reads use **cache-aside**: Redis first, DB fallback.”

---

## 1) Comment API routes (entry points)

Routes file:
- `src/features/comments/routes/commentRoutes.ts`

Endpoints:
- `POST /api/v1/post/comment` → create comment  
- `GET /api/v1/post/comments/:postId` → list comments for a post  
- `GET /api/v1/post/commentsnames/:postId` → usernames + count  
- `GET /api/v1/post/single/comment/:postId/:commentId` → fetch one comment

Auth:
- all routes use `authMiddleware.checkAuthentication`

---

## 2) End-to-end flow: “User adds a comment”

### Step A — HTTP hits the controller
Controller:
- `src/features/comments/controllers/add-comment.ts`

What it does:
1. **Validate input** with `@joiValidation(addCommentSchema)`
2. Build `commentData` (shape = `ICommentDocument`):
   - `_id` (new ObjectId)
   - `postId`
   - `username`, `avatarColor` (from `req.currentUser`)
   - `profilePicture`, `comment`
   - `createdAt`

3. **Write to Redis immediately**:
   - `commentCache.savePostCommentToCache(postId, JSON.stringify(commentData))`

4. **Enqueue DB write** (async):
   - `commentQueue.addCommentJob('addCommentToDB', databaseCommentData)`

5. Respond instantly:
   - HTTP 200 `{ message: 'Comment created successfully' }`

✅ So your write path is **Write-behind**:
- Redis first (fast)
- Mongo later (durable)

---

## 3) Redis: what keys + data types we store (Comment feature)

### Key 1 — Comments list per post
**Key:** `comments:<postId>`  
**Type:** Redis **LIST**  
**Command used:** `LPUSH`, `LRANGE`, `LLEN`

What we store inside the list:
- Each list item is a **string** (JSON string) of the comment object  
  (`JSON.stringify(commentData)`)

So the data is stored like:
```json
"{"_id":"...","postId":"...","username":"...","comment":"...","createdAt":"..."}"
```

Why LIST?
- We want **append/insert** and **range reads** for “show all comments”.
- `LRANGE comments:<postId> 0 -1` returns all comment strings.

---

### Key 2 — Post hash (comment count)
**Key:** `posts:<postId>`  
**Type:** Redis **HASH**  
**Commands:** `HMGET`, `HSET`

`CommentCache.savePostCommentToCache()` also increments:
- `posts:<postId> -> commentsCount`

Important detail:
- Redis hash values are stored as **strings**
- You read count via `HMGET` then parse to number then `HSET` back as string.

---

## 4) How CommentCache works (your exact methods)

File:
- `src/shared/services/redis/comment.cache.ts`

### (2) “Which key / how they save / is it string?”
✅ Yes: they save as **string**.

#### `savePostCommentToCache(postId, value)`
- Ensures redis client is connected
- `LPUSH comments:<postId> value`
- reads `commentsCount` from `posts:<postId>`
- increments and writes back to `posts:<postId>`

**Value format:** JSON string of `ICommentDocument`.

---

### (3) `getCommentsNamesFromCache(postId)` — what is it?
Use case:
- UI wants “who commented” (and count) quickly without DB.

Implementation:
- `LLEN comments:<postId>` → total count
- `LRANGE comments:<postId> 0 -1` → get all comment strings
- parse each comment, push `comment.username` into an array
- return:
```ts
[{ count: <number>, names: <string[]> }]
```

⚠️ Note (honest improvement):
- This cache method returns **all usernames** (duplicates possible).
- DB version (`commentService.getPostCommentNames`) uses Mongo aggregation `$addToSet` which returns **unique names**.
- If interviewer asks: say you’d change Redis version to store a **SET** of usernames or de-dupe in code.

---

### (4) `getSingleCommentFromCache(postId, commentId)` — why and is it fast?
Why this exists:
- For endpoints like “open one comment details”, “edit comment”, “delete comment”, etc. you might want **one comment** without reading from Mongo.

How it finds:
1. `LRANGE comments:<postId> 0 -1` (fetch all)
2. parse into objects
3. `lodash.find(list, item => item._id === commentId)`

Is it fast?
- For small/medium comment lists: yes, it avoids Mongo call.
- Big-O: **O(N)** because it scans the list.
- Interview improvement:
  - store each comment also in a hash:
    - `comment:<commentId>` (HASH/JSON string)
  - store commentIds in the list and fetch by id  
  - then single-comment lookup becomes **O(1)**

So: current approach is OK for demo scale; can be optimized.

---

## 5) Why do we use lodash in this project?
In *this comment cache* file, lodash is used for:
- `find(list, predicate)`

Reason:
- consistent utility style across the codebase
- clean, readable operations on arrays/objects

Other lodash uses in your repo (examples):
- `omit` (remove fields from objects safely)
- `remove` / `filter` / `findIndex`
- `map`
- `indexOf`
- `random`, `floor` (seeding)

Interview line:
> “We used lodash for reliable, readable collection/object utilities. In modern Node you can replace many with native methods, but lodash keeps consistency.”

---

## 6) What “data we are saving in interface” means (types)

Interfaces file:
- `src/features/comments/interfaces/comment.interface.ts`

### `ICommentDocument`
This describes the **comment object shape** (used in cache + DB):
- `_id`
- `username`
- `avatarColor`
- `postId`
- `profilePicture`
- `comment`
- `createdAt`

That is exactly what gets stored as JSON string in:
- `comments:<postId>` list

### `ICommentJob`
This is the **Bull job payload** (extra fields needed for DB + notifications):
- `postId`
- `userTo` (post owner)
- `userFrom` (commenter)
- `username` (commenter username)
- `comment` (the comment object)

---

## 7) The Bull queue + worker (durable write + notifications)

### Queue
File:
- `src/shared/services/queues/comment.queue.ts`

Queue name:
- `comments`

Job processor:
- `this.processJob('addCommentToDB', 5, commentWorker.addCommentToDB)`

So concurrency is **5**.

### Worker
File:
- `src/shared/workers/comment.worker.ts`

Worker method:
- `commentWorker.addCommentToDB`

It calls:
- `commentService.addCommentToDB(data)`

### DB service (what happens in Mongo)
File:
- `src/shared/services/db/comment.service.ts`

`addCommentToDB()` does:
1. `CommentsModel.create(comment)` → inserts the comment
2. `PostModel.findOneAndUpdate({ _id: postId }, { $inc: { commentsCount: 1 } }, { new: true })`
3. Gets post owner data from Redis:
   - `userCache.getUserFromCache(userTo)` (so it doesn’t hit DB for user)

Then (important engineering):
4. If post owner enabled comment notifications AND commenter != owner:
   - creates notification in Mongo via `NotificationModel.insertNotification(...)`
   - emits socket event:
     - `socketIONotificationObject.emit('insert notification', notifications, { userTo })`
   - sends email via queue:
     - `emailQueue.addEmailJob('commentsEmail', {...})`

✅ This is a full production-like flow: DB write + notification + email, all async.

---

## 8) Read path: how we serve comments fast (cache-aside)

Controller:
- `src/features/comments/controllers/get-comments.ts`

### GET `/post/comments/:postId`
- tries `commentCache.getCommentsFromCache(postId)`
- if cache has items → return them
- else → query Mongo:
  - `commentService.getPostComments({ postId }, { createdAt: -1 })`

### GET `/post/commentsnames/:postId`
- tries `commentCache.getCommentsNamesFromCache(postId)`
- else → Mongo aggregation (`$addToSet` unique + count)

### GET `/post/single/comment/:postId/:commentId`
- tries `commentCache.getSingleCommentFromCache(postId, commentId)`
- else → Mongo by `_id`

---

## 9) Redis data types summary (comment feature)

| Key pattern | Data type | Stores | Used by |
|---|---|---|---|
| `comments:<postId>` | LIST | JSON strings of `ICommentDocument` | comment list + usernames + single comment |
| `posts:<postId>` | HASH | `commentsCount` string (and other post fields) | keep counts consistent with cache |

---

## 10) Interview “deep questions” about this feature

1. Why LIST for comments? When would you use a HASH instead?
2. Your single comment lookup is O(N). How would you make it O(1)?
3. Cache vs DB mismatch: usernames list duplicates in cache. How to fix?
4. What happens if Bull job runs twice? How do you avoid duplicates (idempotency)?
5. What happens if Redis is down during comment create?
6. What if Mongo insert fails but Redis already has the comment?
7. How do you rebuild Redis cache if Redis restarts?
8. Why do you update `commentsCount` in both Redis and Mongo?
9. How do you ensure notification is only sent to post owner?
10. How do you test this feature end-to-end (cache + worker)?

---

## 11) 30-second “talk track” (practice)

> “Creating a comment goes through auth + Joi validation. We immediately push the comment into Redis list `comments:<postId>` as a JSON string and update `posts:<postId>.commentsCount`. This gives instant read-your-write behavior for the UI. Then we enqueue a Bull job in queue `comments` (`addCommentToDB`) processed by `commentWorker.addCommentToDB` to write to Mongo and increment post count. The worker also sends notification and email asynchronously if enabled. Reads use cache-aside: Redis first, Mongo fallback.”

