# Common Ideas
 
**You have a user:**
```js
{
  name: "Suraj",
  favSinger: ObjectId("A1")
}

```

**You do a $lookup into the singers collection:**
```js
{
  _id: "A1",
  name: "Arijit Singh"
}
```
**After $lookup, MongoDB always puts the result inside an array:**

```js
{
  name: "Suraj",
  favSinger: ObjectId("A1"),
  singerDetails: [
    { _id: "A1", name: "Arijit Singh" }
  ]
}
```

**Problem:**

- singerDetails is inside an array, even though we know it has only one item.
- If we want to group, sort, or project fields like singerDetails.name, we’d need to write:

`singerDetails[0].name`

That’s messy.

`Solution → $unwind`
`$unwind: "$singerDetails"` removes the array wrapper.

Now the document becomes:

``` js
{
  name: "Suraj",
  favSinger: ObjectId("A1"),
  singerDetails: { _id: "A1", name: "Arijit Singh" }
}
```

Much simpler ✅

- Rule of Thumb (super simple version):
- If you need only one singer and don’t mind the array, skip $unwind.
- If you want to treat each singer separately (group, sort, count, display nicely), use $unwind.