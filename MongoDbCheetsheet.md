
# 📌 MongoDB CRUD Cheat Sheet
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