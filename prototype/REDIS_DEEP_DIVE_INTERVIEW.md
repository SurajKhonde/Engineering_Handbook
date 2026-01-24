# Redis Deep Dive — What This Project Stores, Why, and How (Interview Notes)

This is a **code-driven** explanation of the Redis layer inside your project folder:

`src/shared/services/redis/`

Covered in detail:
- `base.cache.ts`
- `redis.connection.ts`
- `user.cache.ts`
- `post.cache.ts` (main focus)
- `comment.cache.ts`
- `follower.cache.ts`
- `reaction.cache.ts`

**Not covered today (as you asked):**
- `message.cache.ts` (chat/message caching)

---

## 1) What Redis is used for in this project (mental model)

Your backend is designed around:
- **FAST path**: update UI quickly (Socket + Redis)
- **SLOW path**: persist reliably (Bull workers → MongoDB)

Redis acts as:
1) **Read cache** for feeds and lists (posts, comments, followers, reactions)
2) **Write-behind buffer**: you write to Redis immediately, then Bull workers write to MongoDB later
3) **Counters & denormalized fields**: postsCount, followersCount, followingCount, commentsCount, reactions

MongoDB remains the **source of truth** (durable store). Redis is a **speed layer**.

---

## 2) Redis key schema (what keys exist + data types)

This is the “database schema” of Redis in your app:

### Users
- `user` → **ZSET**: sorted index of user keys (for listing users)
- `users:<userId>` → **HASH**: user profile fields (stored as strings)
- `followers:<userId>` → **LIST**: list of follower userIds
- `following:<userId>` → **LIST**: list of followee userIds

### Posts / Feed
- `post` → **ZSET**: sorted index of postIds (feed index)
- `posts:<postId>` → **HASH**: post fields (stored as strings)
- `comments:<postId>` → **LIST**: list of comments for that post (each item is JSON string)
- `reactions:<postId>` → **LIST**: list of reactions for that post (each item is JSON string)

> Important: Redis stores **strings**, so objects/arrays are stored using `JSON.stringify(...)` and parsed back with `Helpers.parseJson(...)`.

---

## 3) Why every cache checks `if (!this.client.isOpen) await this.client.connect()`

You saw this pattern everywhere:

```ts
if (!this.client.isOpen) {
  await this.client.connect();
}
```

### Why?
- `createClient(...)` **creates a client object**, but **does not guarantee it’s connected**.
- If a command runs while the client isn’t connected, node-redis can throw.
- In your code, you often instantiate caches like `new PostCache()` inside controllers, so the connection might not yet be open for that instance.

### Design note (honest engineering)
Right now **each cache class creates its own Redis client** (because `BaseCache` constructor calls `createClient`).
So:
- `redisConnection.connect()` connects only the **redisConnection client**
- `PostCache`, `UserCache`, `CommentCache`, etc. still need to connect their **own** clients

**Production improvement**
Use a single shared Redis client (singleton) and reuse across caches to avoid many connections.

---

# FILE-BY-FILE

---

## 4) `base.cache.ts` — the base Redis client for every cache

```ts
export abstract class BaseCache {
  client: RedisClient;

  constructor(cacheName: string) {
    this.client = createClient({ url: config.REDIS_HOST });
    this.log = config.createLogger(cacheName);
    this.cacheError();
  }

  private cacheError(): void {
    this.client.on('error', (error: unknown) => {
      this.log.error(error);
    });
  }
}
```

### What it does
- Creates a redis client pointed to `REDIS_HOST`
- Attaches an `error` event handler for logging

### What it does **not** do
- It **does not call `connect()`**
That’s why every function checks `isOpen`.

---

## 5) `redis.connection.ts` — “startup connection test”

```ts
async connect(): Promise<void> {
  await this.client.connect();
  log.info(`Redis connection: ${await this.client.ping()}`);
}
```

### Why it exists
- At server startup, you want to ensure Redis credentials/URL are correct
- `PING` gives a clean “Redis is alive” log

### But why do caches still connect?
Because each cache has its own client instance (see section 3).

---

# 6) `comment.cache.ts` — Comment caching (LIST + HASH)

File: `src/shared/services/redis/comment.cache.ts`

### Keys used
| Key | Type | Stores |
|---|---|---|
| `comments:<postId>` | LIST | each comment as JSON string |
| `posts:<postId>` | HASH | `commentsCount` field |

---

## 6.1 `savePostCommentToCache(postId, value)`

Relevant code:

```ts
await this.client.LPUSH(`comments:${postId}`, value);

const commentsCount: string[] = await this.client.HMGET(`posts:${postId}`, 'commentsCount');
let count: number = Helpers.parseJson(commentsCount[0]) as number;
count += 1;
await this.client.HSET(`posts:${postId}`, 'commentsCount', `${count}`);
```

### What `LPUSH` means here
- `LPUSH key value` inserts at the **head** of a LIST
- You are maintaining a **separate comment list per post** because the key includes postId:
  - `comments:POST_123` is different from `comments:POST_999`

So yes: every post has its own list.

### What `HMGET` is (and why it returns an array)
- `HMGET <hashKey> <field1> <field2> ...`
- It returns an **array** of values (one per field).
Even if you ask for 1 field, Redis still returns an array length 1:

```ts
const commentsCount = await HMGET("posts:POST_123", "commentsCount");
// commentsCount = ["0"]  (string)
```

### What `HSET` is
- `HSET <hashKey> <field> <value>`
- It sets/overwrites one field inside a Redis HASH.

### Why update `posts:<postId>.commentsCount` in Redis?
Because feed reads come from Redis `posts:<postId>` hash.
If you don’t update the count there, UI counters would be stale until MongoDB catches up.

---

## 6.2 `getCommentsFromCache(postId)`

```ts
const reply: string[] = await this.client.LRANGE(`comments:${postId}`, 0, -1);
for (const item of reply) list.push(Helpers.parseJson(item));
```

### What it does
- `LRANGE key 0 -1` fetches the whole list
- each list entry is JSON string → parsed into `ICommentDocument`

### Data type + storage
- LIST values are **strings**, so you stored JSON strings using `JSON.stringify(...)`.

---

## 6.3 `getCommentsNamesFromCache(postId)` — what is it?

```ts
const commentsCount: number = await this.client.LLEN(`comments:${postId}`);
const comments: string[] = await this.client.LRANGE(`comments:${postId}`, 0, -1);
...
list.push(comment.username);
return [{ count: commentsCount, names: list }];
```

### What it returns
- `count`: number of comments (fast, via `LLEN`)
- `names`: list of usernames pulled from each comment

### Engineering note
- This returns usernames **with duplicates** (if same user comments multiple times).
Mongo aggregation could return unique names; Redis version is “quick but not deduped”.
If interviewer asks: “I’d store usernames in a Redis SET or dedupe in code.”

---

## 6.4 `getSingleCommentFromCache(postId, commentId)` — why single? how found?

```ts
const comments = await LRANGE(`comments:${postId}`, 0, -1);
const list = comments.map(JSON.parse);
const result = find(list, item => item._id === commentId);
return [result];
```

### What it tries to do
Fetch one comment (e.g. open comment details, verify before update/delete, etc.) without hitting Mongo.

### What Redis structure is used
- Still the LIST `comments:<postId>`

### Is it fast?
- It avoids Mongo (good)
- But it scans the whole list (**O(N)**) because it loads all comments then finds one.

**Production improvement**
Store comments also by id:
- `comment:<commentId>` as HASH/JSON
- and store only commentIds in `comments:<postId>`
Then single comment lookup becomes **O(1)**.

---

# 7) `follower.cache.ts` — Followers & Following caching (LIST + HASH)

File: `src/shared/services/redis/follower.cache.ts`

### Keys used
| Key | Type | Stores |
|---|---|---|
| `followers:<userId>` | LIST | userIds of followers |
| `following:<userId>` | LIST | userIds the user follows |
| `users:<userId>` | HASH | counters + blocked arrays |

---

## 7.1 `saveFollowerToCache(key, value)`
```ts
await this.client.LPUSH(key, value);
```

### What it stores (based on controller usage)
Controller calls:
```ts
saveFollowerToCache(`following:${currentUserId}`, `${followeeId}`);
saveFollowerToCache(`followers:${followeeId}`, `${currentUserId}`);
```

So:
- `following:<me>` list contains **IDs of people I follow**
- `followers:<them>` list contains **IDs of their followers**

---

## 7.2 `removeFollowerFromCache(key, value)` and `LREM`

```ts
await this.client.LREM(key, 1, value);
```

### What is `LREM`?
`LREM <key> <count> <value>`

- It removes occurrences of `<value>` from a LIST
- `<count>` controls how many and which direction:

| count | meaning |
|---:|---|
| `> 0` | remove first `count` matches scanning from head (left) |
| `< 0` | remove first `abs(count)` matches scanning from tail (right) |
| `0` | remove **all** matches |

So `LREM(key, 1, value)` means:
✅ “remove the **first** occurrence of this userId from the list”

---

## 7.3 `updateFollowersCountInCache(userId, prop, value)`

```ts
await this.client.HINCRBY(`users:${userId}`, prop, value);
```

### What is `HINCRBY`?
- It increments a numeric field inside a HASH atomically.
Example:
- `HINCRBY users:123 followersCount 1`

This is safer than:
- HMGET → parse → +1 → HSET
Because HINCRBY is **atomic** and handles concurrent updates better.

---

## 7.4 `getFollowersFromCache(key)` — is it “particular user followers”?

```ts
const response: string[] = await this.client.LRANGE(key, 0, -1);
for (const item of response) {
  const user = await userCache.getUserFromCache(item);
  list.push({ ...followerData });
}
```

### What it returns
- It returns a list of follower “cards” (IFollowerData)
- It reads the follower ids from Redis LIST (`followers:<userId>` or `following:<userId>`)
- Then it fetches each follower profile from `users:<id>` HASH using `userCache.getUserFromCache(...)`

### So yes:
- If key = `followers:USER_A` → “people who follow USER_A”
- If key = `following:USER_A` → “people USER_A follows”

---

## 7.5 `updateBlockedUserPropInCache(key, prop, value, type)`

```ts
const response = await HGET(`users:${key}`, prop);
let blocked = JSON.parse(response); // string[] stored as JSON string
if (type === 'block') blocked = [...blocked, value];
else remove(blocked, id => id === value);

multi.HSET(`users:${key}`, prop, JSON.stringify(blocked));
await multi.exec();
```

### What it stores
- In `users:<key>` HASH there are fields like:
  - `blocked`
  - `blockedBy`
These fields are arrays stored as JSON strings.

### Why use `multi()` here?
So the update is executed as a single Redis transaction step (and also reduces round trips).

---

# 8) `reaction.cache.ts` — Reactions (likes) caching (LIST + HASH)

File: `src/shared/services/redis/reaction.cache.ts`

### Keys used
| Key | Type | Stores |
|---|---|---|
| `reactions:<postId>` | LIST | each reaction as JSON string |
| `posts:<postId>` | HASH | `reactions` field (aggregated counts map) |

---

## 8.1 `savePostReactionToCache(key, reaction, postReactions, type, previousReaction)`

- If user had a previous reaction → remove old one from LIST
- If new reaction exists → push new reaction to LIST and update `posts:<id>.reactions`

This keeps:
- reaction **history** (LIST items)
- reaction **summary** (HASH field on the post)

---

## 8.2 `removePostReactionFromCache(key, username, postReactions)`
```ts
const response = await LRANGE(`reactions:${key}`, 0, -1);
const prev = getPreviousReaction(response, username);
multi.LREM(`reactions:${key}`, 1, JSON.stringify(prev));
await multi.exec();

await HSET(`posts:${key}`, 'reactions', JSON.stringify(postReactions));
```

### Why `LREM(..., 1, ...)`?
Remove only one entry (the user's previous reaction object).

---

## 8.3 `getReactionsFromCache(postId)`
- `LLEN` for count
- `LRANGE` and parse JSON list

---

# 9) `post.cache.ts` — POST caching (ZSET + HASH) **(main focus)**

File: `src/shared/services/redis/post.cache.ts`

### Keys used
| Key | Type | Stores |
|---|---|---|
| `post` | ZSET | postId index for feed |
| `posts:<postId>` | HASH | post fields |
| `users:<userId>` | HASH | `postsCount` counter |
| `comments:<postId>` | LIST | comment list |
| `reactions:<postId>` | LIST | reaction list |

---

## 9.1 `savePostToCache(data)` — what it saves and why

### Step 1: Create the HASH payload
`dataToSave` maps post fields to strings:

- `_id`, `userId`, `username`, `email`
- `post`, `bgColor`, `privacy`, `gifUrl`
- `commentsCount` as string
- `reactions` as JSON string
- `imgId/imgVersion`, `videoId/videoVersion`
- `createdAt` as string

**Why strings?** Redis HASH stores field values as strings. For arrays/objects, you stringify.

---

### Step 2: Index the post in a ZSET
```ts
await this.client.ZADD('post', { score: parseInt(uId, 10), value: `${key}` });
```

This means:
- key `post` is a sorted index of post ids
- score = `uId` (the numeric user id)

**Why use score = uId in this codebase?**
Because later you do:
```ts
ZRANGE('post', uId, uId, { BY: 'SCORE' })
```
to fetch all posts by that user quickly.

✅ That’s a clever trick for “get user posts by uId”.

**Tradeoff / interview improvement**
It’s not perfect for a global feed because uId is not time.
Better design:
- Global feed ZSET uses `createdAt` timestamp as score
- User posts are stored in a separate `userPosts:<userId>` ZSET

---

### Step 3: Read postsCount with `HMGET`
```ts
const postCount: string[] = await HMGET(`users:${currentUserId}`, 'postsCount');
```

Again returns an array:
- `postCount[0]` is the value for `postsCount`

---

## 9.2 What is `multi()` here?

You asked:
```ts
const multi = this.client.multi();
multi.HSET(...)
multi.exec()
```

### Meaning of `multi()`
In node-redis:
- `client.multi()` starts a **transaction command queue** (MULTI/EXEC)
- You add many Redis commands to it
- `exec()` sends them together

Benefits:
1) **Fewer round trips** (performance)
2) **Atomic execution** (all-or-nothing inside Redis)

In your `savePostToCache`, you do:
- many `HSET` fields into the post hash
- update `users:<id>.postsCount`

So they put those into multi.

### What these lines do
```ts
multi.HSET(`users:${currentUserId}`, 'postsCount', count);
multi.exec();
```

Meaning:
- Update the user hash field `postsCount` to the new count
- Execute the queued transaction

**Note (small bug to be aware of)**
In `savePostToCache`, it calls `multi.exec()` without `await`.
So the function may return before Redis finishes.
Safer:
```ts
await multi.exec();
```

---

## 9.3 `getPostsFromCache(key, start, end)` — how paginated feed works

```ts
const reply: string[] = await ZRANGE(key, start, end, { REV: true });
for (const value of reply) multi.HGETALL(`posts:${value}`);
const replies = await multi.exec();
```

### What’s happening
1) **Get list of post ids** from the ZSET:
   - `ZRANGE post start end REV`
   - This is “pagination” over an ordered index

2) **Batch fetch all posts** using `HGETALL`:
   - one hash read per post id
   - executed via `multi.exec()` to reduce round-trips

3) Parse types:
   - `commentsCount` string → number
   - `reactions` string → object
   - `createdAt` string → Date

### Why this is a good design
This is the classic Redis pattern:
- **Index in ZSET** (cheap to paginate)
- **Objects in HASH** (cheap to fetch by id)

---

## 9.4 `getPostsWithImagesFromCache` / `getPostsWithVideosFromCache`

These do the same thing as `getPostsFromCache` but filter in code:
- images: `imgId && imgVersion` OR `gifUrl`
- videos: `videoId && videoVersion`

---

## 9.5 `getUserPostsFromCache(key, uId)` — per-user posts

```ts
ZRANGE(key, uId, uId, { REV: true, BY: 'SCORE' })
```

This means:
- “return all values in ZSET whose score is exactly uId”

That’s why uId is used as score.

---

## 9.6 Delete post cache (cleanup)

`deletePostFromCache(postId, currentUserId)` does:

```ts
multi.ZREM('post', postId);
multi.DEL(`posts:${postId}`);
multi.DEL(`comments:${postId}`);
multi.DEL(`reactions:${postId}`);
multi.HSET(`users:${currentUserId}`, 'postsCount', newCount);
await multi.exec();
```

### Why delete comments/reactions keys too?
Because comments and reactions are stored separately by postId.
If you delete post but keep these lists, you leave garbage keys.

---

## 9.7 Update post cache

`updatePostInCache(postId, updatedPost)`:
- `HSET` updated fields into `posts:<id>`
- then does `HGETALL` to return the updated post to API

This is used so UI gets updated post data immediately.

---

# 10) `user.cache.ts` — user objects and listing

### Keys used
| Key | Type | Stores |
|---|---|---|
| `user` | ZSET | user key index (for listing users) |
| `users:<userId>` | HASH | user object fields |

### Important storage details
Some fields are JSON:
- `blocked`, `blockedBy`, `notifications`, `social`

So on read:
- `HGETALL` returns strings
- code parses fields back into arrays/objects and numbers

### Why ZSET `user` exists
To paginate users quickly:
- `ZRANGE user start end { REV: true }`
Then batch fetch `HGETALL users:<id>` using multi.

---

# 11) Where Bull workers fit in (why Redis doesn’t replace Mongo)

Redis here is a cache + fast layer. MongoDB is durable.

Bull queue config:
- attempts: **3**
- backoff: **fixed 5 seconds**

Queues and workers related to these caches:

| Feature | Queue | Worker methods | DB service |
|---|---|---|---|
| Posts | `posts` | `postWorker.savePostToDB`, `deletePostFromDB`, `updatePostInDB` | `postService.*` |
| Comments | `comments` | `commentWorker.addCommentToDB` | `commentService.addCommentToDB` |
| Followers | `followers` | `followerWorker.addFollowerToDB`, `removeFollowerFromDB` | `followerService.*` |
| Reactions | `reactions` | `reactionWorker.addReactionToDB`, `removeReactionFromDB` | `reactionService.*` |

---

# 12) Quick answers to your exact questions

### Q: `LPUSH` means list for every post different or same?
Different. The key includes postId:
- `comments:<postId>` → each post has its own LIST.

### Q: HMGET returns array?
Yes. Even for one field, it returns an array with one item.

### Q: What is HMGET / HSET?
- `HMGET`: read one or more fields from a HASH
- `HSET`: set a field value in a HASH

### Q: `LREM(key, 1, value)` — what is `1`?
Remove the **first** occurrence of value scanning from head.

### Q: `getFollowersFromCache` returns what?
It returns the followers (or following) for whichever key you pass:
- `followers:<userId>` or `following:<userId>`

### Q: Why do we still need redisConnection if BaseCache exists?
BaseCache creates clients but does not connect.
redisConnection is a startup “ping and log”.
But because every cache makes its own client, each cache still needs to connect too.

---

# 13) “PostCache is fantastic” — what you should say in interview

> “We model feed caching using a Redis sorted-set as an index and hashes as the object store. Pagination is a ZRANGE over the index, and we batch-fetch post objects via multi/exec to reduce RTT. This is a scalable pattern for high read traffic. For per-user posts we also use the ZSET score to filter.”

Then add the upgrade:
> “For production, I would use createdAt as score for chronological ordering and keep a separate per-user index.”

---

## Chat cache note
`message.cache.ts` contains chat-specific keys and logic (chat list, message lists, etc.). We skipped it today as requested.

