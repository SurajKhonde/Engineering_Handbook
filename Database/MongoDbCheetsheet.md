
# 📌 MongoDB CRUD Cheat Sheet

# MongoDB Cheat Sheet

## 1. CRUD Operations

| Method         | Description                        | Parameters / Takes             | Example                                                        | Returns                                                                 |
|----------------|------------------------------------|--------------------------------|----------------------------------------------------------------|-------------------------------------------------------------------------|
| `insertOne()`  | Insert a single document           | Object                         | `db.users.insertOne({name:"Suraj", age:25})`                   | `{ acknowledged: true, insertedId: ObjectId("...") }`                   |
| `insertMany()` | Insert multiple documents          | Array of objects               | `db.users.insertMany([{name:"A"},{name:"B"}])`                 | `{ acknowledged: true, insertedIds: { "0": ObjectId("..."), "1": ObjectId("...") } }` |
| `findOne()`    | Find a single document             | Query object                   | `db.users.findOne({name:"Suraj"})`                             | One document object or `null`                                           |
| `find()`       | Find multiple documents            | Query object, optional projection | `db.users.find({age:{$gt:20}})`                             | Cursor (iterable list of docs). Example: `[ { name:"Suraj", age:25 }, { name:"A", age:30 } ]` |
| `updateOne()`  | Update first document matching filter | Filter object + update object | `db.users.updateOne({name:"Suraj"}, {$set:{age:26}})`          | `{ acknowledged: true, matchedCount: 1, modifiedCount: 1 }`             |
| `updateMany()` | Update all matching documents      | Filter object + update object  | `db.users.updateMany({age:{$lt:20}}, {$inc:{age:1}})`          | `{ acknowledged: true, matchedCount: 3, modifiedCount: 3 }`             |
| `deleteOne()`  | Delete first matching document     | Filter object                  | `db.users.deleteOne({name:"A"})`                               | `{ acknowledged: true, deletedCount: 1 }`                               |
| `deleteMany()` | Delete all matching documents      | Filter object                  | `db.users.deleteMany({age:{$lt:18}})`                          | `{ acknowledged: true, deletedCount: 5 }`                               |

---
## 2. Array Update Operators

| Operator | Description | Example |
|----------|------------|--------|
| `$push` | Add element to array | `{$push:{hobbies:"coding"}}` |
| `$pop` | Remove first (-1) or last (1) element | `{$pop:{hobbies:1}}` |
| `$addToSet` | Add element only if not exists | `{$addToSet:{hobbies:"coding"}}` |
| `$pull` | Remove elements matching condition | `{$pull:{hobbies:"gaming"}}` |
| `$each` | Add multiple elements with `$push` | `{$push:{hobbies:{$each:["a","b"]}}}` |
| `$slice` | Limit array elements when pushing | `{$push:{hobbies:{$each:["a","b"], $slice:1}}}` |

---

## 3. Query / Filter Operators

| Operator | Description | Example |
|----------|------------|--------|
| `$eq` | Equal | `{age:{$eq:25}}` |
| `$ne` | Not equal | `{age:{$ne:25}}` |
| `$gt` | Greater than | `{age:{$gt:20}}` |
| `$gte` | Greater than or equal | `{age:{$gte:20}}` |
| `$lt` | Less than | `{age:{$lt:30}}` |
| `$lte` | Less than or equal | `{age:{$lte:30}}` |
| `$in` | Value in array | `{age:{$in:[20,25,30]}}` |
| `$nin` | Not in array | `{age:{$nin:[20,25]}}` |
| `$exists` | Field exists | `{address:{$exists:true}}` |
| `$regex` | Match string pattern | `{name:{$regex:/^S/}}` |

---

## 4. Aggregation Pipeline Operators

| Operator | Description | Example |
|----------|------------|--------|
| `$match` | Filter documents | `{ $match: {age:{$gt:20}} }` |
| `$project` | Include/exclude fields | `{ $project: {name:1, age:1} }` |
| `$group` | Group documents and aggregate | `{ $group: {_id:"$city", total:{$sum:1}} }` |
| `$sum` | Sum values | `{ $group:{_id:null, total:{$sum:"$age"}} }` |
| `$avg` | Average | `{ $group:{_id:null, avgAge:{$avg:"$age"}} }` |
| `$min` | Minimum | `{ $group:{_id:null, minAge:{$min:"$age"}} }` |
| `$max` | Maximum | `{ $group:{_id:null, maxAge:{$max:"$age"}} }` |
| `$count` | Count documents | `{ $count:"totalUsers" }` |
| `$sort` | Sort documents | `{ $sort:{age:-1} }` |
| `$limit` | Limit number of documents | `{ $limit:5 }` |
| `$skip` | Skip N documents | `{ $skip:10 }` |
| `$unwind` | Deconstruct array into multiple docs | `{ $unwind:"$hobbies" }` |
| `$lookup` | Left join with another collection | `{ $lookup:{from:"orders", localField:"_id", foreignField:"userId", as:"orders"} }` |
| `$addFields` | Add new fields | `{ $addFields:{fullName:{$concat:["$first"," ","$last"]}} }` |
| `$replaceRoot` | Replace document root | `{ $replaceRoot:{newRoot:"$address"} }` |

---

## 5. Miscellaneous / Other Methods

| Method / Operator | Description | Example |
|------------------|------------|--------|
| `distinct()` | Unique values of a field | `db.users.distinct("city")` |
| `$size` | Size of array | `{friends:{$size:3}}` |
| `count()` | Count matching docs | `db.users.count({age:{$gt:20}})` |
| `sort()` | Sort in find | `db.users.find().sort({age:-1})` |
| `limit()` | Limit docs in find | `db.users.find().limit(5)` |
| `skip()` | Skip docs in find | `db.users.find().skip(10)` |

---









 #### 🔹 Create
 **Insert one document**
```js
db.collection.insertOne({ name: "Suraj", age: 25 })
```

 **Insert multiple documents**
```js
db.collection.insertMany([{ name: "Raj" }, { name: "Aman" }])
```
#### 🔹 Read
 **Find all documents**
```js
db.collection.find()
```
 **Find one document**
```js
db.collection.findOne({ name: "Suraj" })
```
 **Query with condition**
```js
db.collection.find({ age: { $gt: 20 } })
```

 **Projection (select specific fields)**
```js
db.collection.find({}, { name: 1, age: 1, _id: 0 })
```

 **Sort**
```js
db.collection.find().sort({ age: -1 })  // -1 = desc, 1 = asc

```
**Limit**
```js
db.collection.find().limit(5)

```
#### 🔹 Update
**Update one document**
```js
db.collection.updateOne(
  { name: "Suraj" },
  { $set: { age: 26 } }
)
```
**Update multiple documents**
```js
db.collection.updateMany(
  { country: "India" },
  { $set: { isActive: true } }
)
```

**Increment a field**
```js
db.collection.updateOne(
  { name: "Suraj" },
  { $inc: { loginCount: 1 } }
)
```

**Push into array**
```js
db.collection.updateOne(
  { name: "Suraj" },
  { $push: { hobbies: "Music" } }
)
```
 **Pull from array** 
```js
db.collection.updateOne(
  { name: "Suraj" },
  { $pull: { hobbies: "Music" } }
)
```

#### 🔹 Delete
**Delete one document**
```js
db.collection.deleteOne({ name: "Suraj" })
```

**Delete many documents**
```js
db.collection.deleteMany({ country: "India" });
```
**Drop entire collection**
```js
db.collection.drop()
```

#### 🔹 1. $sort
 - Used to order documents.

Syntax:
```js
{ $sort: { fieldName: 1 } }   // ascending
{ $sort: { fieldName: -1 } }  // descending
```

#### 🔹 2. $limit
Used to restrict number of documents in result.
Example:
```js

db.users.aggregate([
  { $sort: { age: -1 } },
  { $limit: 5 } // top 5 oldest
])
```

### 🔹 3. $sum
Used inside $group to add values or count docs.

- Two main patterns:

```js
{ $sum: "$field" } // add up values of field
{ $sum: 1 }        // just count docs
db.orders.aggregate([
  { $group: { _id: "$status", total: { $sum: 1 } } }
])
```
### 🔹 4. $count

- Shortcut stage to count documents in pipeline.
```js
db.users.aggregate([
  { $match: { country: "India" } },
  { $count: "totalUsers" }
])
```
⚠️ Difference:
- $sum:1 → used inside $group
- $count → standalone stage at the end.

### 🔹 5. $size
- Used to get the length of an array.

✅ Works in $project (or $addFields), not in $match.
Example:
```js
db.users.aggregate([
  { $project: { name: 1, favCount: { $size: "$favoriteSongs" } } }
])
```

⚠️ If array is missing or null, $size will throw error → we often use $ifNull.

#### 🔹 6. $unwind
- If one user has 3 songs → becomes 3 rows.
- Flattens array → one document per element.
Example:
```js
db.users.aggregate([
  { $unwind: "$favoriteSongs" }
])
```
### 🔹 7. $exists
- Used in filters (find or $match), not in aggregation projection.
- Checks if field exists.
Example:
```js
db.users.find({ email: { $exists: true } })
db.users.find({ email: { $exists: false } }) // users without email
```

