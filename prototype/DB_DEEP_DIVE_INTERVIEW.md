# MongoDB / Mongoose Deep Dive — Data Model + Indexing (Interview Notes)

This explains **how your app stores data in MongoDB**, what queries you run, which **indexes you already have**, and which **indexes you’d add for production**.

Project root (relevant folders):
- Schemas (collections): `src/features/*/models/*.schema.ts`
- DB services (query patterns): `src/shared/services/db/*.service.ts`
- DB connection: `src/setupDatabase.ts`

---

## 1) Big picture (what MongoDB is doing here)

MongoDB is your **source of truth**:
- Durable storage for posts, comments, reactions, followers, users, notifications, messages.
- Redis is the “fast layer”; Bull workers ensure Mongo is updated reliably.

**Read flow (typical):**
- Redis → if miss → MongoDB query

**Write flow (typical):**
- Redis update + socket event → Bull job → MongoDB write

---

## 2) How “indexes” are created in this project

### 2.1 Default indexes you always get
MongoDB automatically creates an index on:
- `_id` for every collection.

### 2.2 Indexes defined by your Mongoose schemas
In your schemas, you define indexes using:
- Field option: `index: true`

Example:
```ts
postId: { type: mongoose.Schema.Types.ObjectId, ref: 'Post', index: true }
```

### 2.3 Where Mongoose connects
Your connection is:
- `mongoose.connect(config.DATABASE_URL)` in `src/setupDatabase.ts`

You didn’t pass `autoIndex: false`, so in development Mongoose usually builds indexes automatically at startup.

**Production note (interview-ready):**
Many teams disable autoIndex in production and manage indexes via migrations (predictable, safe rollouts).

---

# 3) Collections (Schemas) + Current Indexes

## 3.1 Auth (collection: `Auth`)
Schema: `src/features/auth/models/auth.schema.ts`

Fields (important):
- `username`, `email`, `password`, `uId`
- `passwordResetToken`, `passwordResetExpires` (number)

✅ **Indexes currently in schema:** none (only default `_id`)

### Query patterns (DB service: `src/shared/services/db/auth.service.ts`)
- Find by **username**
- Find by **email**
- Find by **passwordResetToken + expires**

### What index you would add (strong interview answer)
You’re querying username/email a lot, so add:
- `username` **unique**
- `email` **unique**
- `passwordResetToken` (optional)
- `uId` (optional if queried often)

Example:
- `db.Auth.createIndex({ username: 1 }, { unique: true })`
- `db.Auth.createIndex({ email: 1 }, { unique: true })`

**Why unique matters:** without unique indexes, duplicates are possible under race conditions (two signup requests at the same time).

---

## 3.2 User (collection: `User`)
Schema: `src/features/user/models/user.schema.ts`

✅ **Indexes currently in schema:**
- `authId` has `index: true`

Why this index exists:
- You frequently fetch user profile by `authId`.

### Query patterns (DB service: `src/shared/services/db/user.service.ts`)
- `$match` by `_id`
- `$match` by `authId`
- list users excluding a given user (`_id != userId`)
- search users via `AuthModel.aggregate` with regex on username

### Index improvements
- If you do “search by username” at scale, you want:
  - `Auth.username` index (and possibly a text index)
- For listing users sorted by createdAt, add `createdAt` (if you store it).

---

## 3.3 Post (collection: `Post`)
Schema: `src/features/post/models/post.schema.ts`

✅ **Indexes currently in schema:**
- `userId` has `index: true`

### Query patterns (DB service: `src/shared/services/db/post.service.ts`)
- Create post (`PostModel.create`)
- Get posts with filters like:
  - has image/gif
  - has video
- Sort + paginate (`$sort`, `$skip`, `$limit`)
- Delete by `_id`
- Update by `_id`
- Count documents

### Index improvements (this is the #1 important one)
Your feed queries typically need **time order**:
- Add index on `createdAt`
- Add compound index for user posts:
  - `{ userId: 1, createdAt: -1 }`

**Why:** you often want “user’s posts latest first” and “global feed latest first”.

---

## 3.4 Comment (collection: `Comment`)
Schema: `src/features/comments/models/comment.schema.ts`

✅ **Indexes currently in schema:**
- `postId` has `index: true`

### Query patterns (DB service: `src/shared/services/db/comment.service.ts`)
- Aggregate:
  - `$match: { postId }`
  - `$sort: { createdAt: -1 }`

### Index improvement (very common interview question)
Add compound index:
- `{ postId: 1, createdAt: -1 }`

**Why:** match by postId + sort by time becomes very fast when a post has many comments.

---

## 3.5 Reaction (collection: `Reaction`)
Schema: `src/features/reactions/models/reaction.schema.ts`

✅ **Indexes currently in schema:**
- `postId` has `index: true`

### Query patterns (DB service: `src/shared/services/db/reaction.service.ts`)
- Upsert/replace by:
  - `{ postId, username, type/previousReaction }`
- Delete by `{ postId, username, type }`
- Aggregate match by postId + sort

### Index improvements
Common choices:
- `{ postId: 1, createdAt: -1 }` (for “show reactions of a post”)
- `{ postId: 1, username: 1 }` (for “user’s reaction on this post”)

If your design is “one reaction per user per post”, enforce:
- unique compound index: `{ postId: 1, username: 1 }` unique

---

## 3.6 Follower (collection: `Follower`)
Schema: `src/features/followers/models/follower.schema.ts`

✅ **Indexes currently in schema:**
- `followerId` has `index: true`
- `followeeId` has `index: true`

### Query patterns (DB service: `src/shared/services/db/follower.service.ts`)
- Create follower relation
- Delete follower relation
- Aggregate match by:
  - followerId (who I follow)
  - followeeId (who follows me)

### Index improvements (correctness + speed)
To avoid duplicate follow rows:
- unique compound index: `{ followerId: 1, followeeId: 1 }` unique

For fast “followers list by time”:
- `{ followeeId: 1, createdAt: -1 }`

---

## 3.7 Notification (collection: `Notification`)
Schema: `src/features/notifications/models/notification.schema.ts`

✅ **Indexes currently in schema:**
- `userTo` has `index: true`

### Query patterns (DB service: `src/shared/services/db/notification.service.ts`)
- Get notifications for a user (userTo), often sorted by createdAt

### Index improvements
Very common:
- `{ userTo: 1, createdAt: -1 }`
- `{ userTo: 1, read: 1, createdAt: -1 }` (if you filter unread)

---

## 3.8 Image (collection: `Image`)
Schema: `src/features/images/models/image.schema.ts`

✅ **Indexes currently in schema:**
- `userId` has `index: true`
- `createdAt` has `index: true`

### Query patterns (DB service: `src/shared/services/db/image.service.ts`)
- Find one by bgImageId
- Aggregate match by userId

### Index improvements
- If bgImageId is frequently searched, index it:
  - `{ bgImageId: 1 }` (possibly unique)

---

## 3.9 Conversation + Message (collections: `Conversation`, `Message`)
Schemas:
- `src/features/chat/models/conversation.schema.ts`
- `src/features/chat/models/chat.schema.ts`

✅ **Indexes currently in schema:** none

### Query patterns (DB service: `src/shared/services/db/chat.service.ts`)
Typical:
- find conversation between 2 users
- fetch messages by conversationId sorted by createdAt

### Index improvements (big performance win)
- Conversation:
  - `{ senderId: 1, receiverId: 1 }` (and handle reverse direction)
  - often enforce uniqueness for the pair
- Message:
  - `{ conversationId: 1, createdAt: -1 }`
  - `{ receiverId: 1, isRead: 1 }` (if you query unread messages)

---

# 4) “How to explain indexing decisions” (interview script)

## 4.1 Rule of thumb
> “I add indexes for the fields used in the most common match/filter conditions, and I include the sort field in compound indexes to avoid in-memory sorting.”

## 4.2 Your comment example (perfect interview answer)
> “We query comments by postId and sort by createdAt, so the correct index is a compound `{ postId: 1, createdAt: -1 }`.”

## 4.3 Your feed answer
> “A chronological feed needs a createdAt index, and profile pages need a `{ userId, createdAt }` compound index.”

---

# 5) Practical: how to check indexes in Mongo

Inside Mongo shell:
```js
use chatty
db.Post.getIndexes()
db.Comment.getIndexes()
db.Follower.getIndexes()
```

Create an index (example):
```js
db.Comment.createIndex({ postId: 1, createdAt: -1 })
```

---

# 6) Where “correctness” is enforced right now (important!)

Some constraints are not enforced by indexes in the schema:
- Auth.username / Auth.email are not unique in schema → app logic prevents duplicates but DB does not enforce.

Interview-ready line:
> “In production I’d enforce uniqueness at the database level using unique indexes because app-level checks alone can race.”

---

# 7) 12 interview questions they can ask about your DB design

1. What’s the source of truth: Redis or MongoDB? Why?
2. Which queries are the hottest paths for your app?
3. Why index postId in comments?
4. What compound index improves comments list pagination?
5. How do you prevent duplicate follow relationships?
6. What index is best for getting latest posts fast?
7. What’s the downside of too many indexes?
8. Why can unique constraints be required even if you validate in code?
9. What happens to writes when you add too many indexes?
10. Why is skip/limit expensive for deep pages?
11. How would you do cursor pagination in MongoDB?
12. How do you handle migrations/index creation safely in production?

---

## If you want next
I can add:
- an ER-style diagram (mermaid)
- a “Top 5 indexes to add right now” checklist
- and a “query → index mapping” table (per service function)
