---
title: "MongoDB Performance Optimization Part 2"
date: 2021-10-17 08:15:00 +0900
permalink: ":categories/backend/:title"
---

> Reference [MongoDB University](https://university.mongodb.com/mercury/M201/2021_October_12/chapter/Chapter_2_MongoDB_Indexes/lesson)

This document keeps all the performance enhancement tips related to QueenBee backend service.

## MongoDB Performance Optimization Part 2

- Compound Indexes
- Multikey Indexes
- Partial Indexes
- Text Indexes

### Compound Indexes

Simply speaking, compound indexes are indexes that hold more than one fields.

```js
{ last_name: 1, first_name: 1 }
```

For compound indexes, the order of index matters. For example, based on the above compound index. You can effectively search for `last_name` and `first_name` together.

But if you search for `first_name` only, you still have to scan the entire collection. Whereas, if you search by the `last_name` you can use the above index. This is because there is `Index Prefixes` in the compoun index.

Compound Index:

For the compound index of:

```js
{ last_name: 1, first_name: 1, type: 1 }
```

Index prefixes are as follows:

```js
1. { last_name: 1 }
2. { last_name: 1, first_name: 1 }
```

This means that when you are searching with `last_name` only, `last_name` + `first_name`, and `last_name` + `first_name` + `type` mongodb will effectively use the compound index. But it won't use compound index if you are using, lets say, `first_name` or `first_name` + `type`.

### Sorting with compound indexes

#### Example 1. sorting with a complete compound index

1. create compound indexes

   ```js
   db.people.createIndex({
     job: 1,
     employer: 1,
     last_name: 1,
     first_name: 1,
   });
   ```

1. Set `exp` var

   ```js
   var exp = db.people.explain("executionStats");
   ```

1. Run a query

   ```js
   exp.find({}).sort({
     job: 1,
     employer: 1,
     last_name: 1,
     first_name: 1,
   });
   ```

1. Result: `IXSCAN` has been performed. Which is what we want.

```js
...
 winningPlan: {
      stage: 'FETCH',
      inputStage: {
        stage: 'IXSCAN',
        keyPattern: { job: 1, employer: 1, last_name: 1, first_name: 1 }
        ...
      }
 }
```

#### Example 2. sorting with a part of compound index - in order

1. Run a query

   ```js
   exp.find({}).sort({
     job: 1,
     employer: 1,
   });
   ```

1. Result: `IXSCAN` has been performed as well. This was possible because the query used `index prefixes`

   ```js
   winningPlan: {
       stage: 'FETCH',
       inputStage: {
         stage: 'IXSCAN',
         keyPattern: { job: 1, employer: 1, last_name: 1, first_name: 1 }
       }
   }
   ```

#### Example 3. sorting with a part of compound index - out of order

1. Run a query

   ```js
   exp.find({}).sort({
     employer: 1,
     job: 1,
   });
   ```

1. Result: `In-memory SORT` and `COLLSCAN` have been performed. Meaning no index was used to achieve the goal. This is because in order for query to use index prefixes, it must be used in order.

   ```js
    winningPlan: {
          stage: 'SORT',
          sortPattern: { employer: 1, job: 1 },
          memLimit: 33554432,
          type: 'simple',
          inputStage`: {
            stage: 'COLLSCAN',
            direction: 'forward'
          }
      }
   ```

#### Example 4. sorting with a query predicate outside of index prefixes

1. Run a query

   ```js
   exp
     .find({
       email: "jenniferfreeman@hotmail.com",
     })
     .sort({
       job: 1,
     });
   ```

2. Result: `IXSCAN` was used for sorting but we still had to look through all 50K document for filtering `email: jenniferfreeman@hotmail.com`. This was because we didn't have email index.

   ```js
   winningPlan: {
         stage: 'FETCH',
         filter: { email: { '$eq': 'jenniferfreeman@hotmail.com' } },
         inputStage: {
           stage: 'IXSCAN',
           keyPattern: { job: 1, employer: 1, last_name: 1, first_name: 1 },
           indexName: 'job_1_employer_1_last_name_1_first_name_1',
         }
   },
   executionStats: {
     executionSuccess: true,
     nReturned: 1,
     executionTimeMillis: 110,
     totalKeysExamined: 50474,
     totalDocsExamined: 50474,
   }
   ```

#### Example 5. sorting with a query predicate within index prefixes

1. Run a query

   ```js
   exp
     .find({
       job: "Graphic designer",
       employer: "Wilson Ltd",
     })
     .sort({
       last_name: 1,
     });
   ```

1. Result: `nReturned` = 2, `totalKeysExamined` = 2 and `totalDocsExamined` = 2, a best ratio you can think of. This was possible because `query predicate` and `sort` order is in a correct order where the compound index is `{job, employer, last_name, first_name}`.

   ```js
    winningPlan: {
      stage: 'FETCH',
      inputStage: {
        stage: 'IXSCAN',
        keyPattern: { job: 1, employer: 1, last_name: 1, first_name: 1 },
        indexName: 'job_1_employer_1_last_name_1_first_name_1'
      }
    },
    executionStats: {
      executionSuccess: true,
      nReturned: 2,
      executionTimeMillis: 0,
      totalKeysExamined: 2,
      totalDocsExamined: 2,
    }
   ```

### Backward sorting with compound index

#### Example 1. Backward sorting with compound index

1. Create index

   ```js
   db.coll.createIndex({ a: 1, b: -1, c: 1 });
   ```

2. Backward sorting

   ```js
   db.coll.find().sort({ a: -1, b: 1, c: -1 });
   ```

3. Possible combinations:

   - forward sorting:

     ```js
     db.coll.find().sort({ a: 1 });
     db.coll.find().sort({ a: 1, b: -1 });
     ```

   - backward sorting:

     ```js
     db.coll.find().sort({ a: -1 });
     db.coll.find().sort({ a: -1, b: 1 });
     ```

4. This won't work. Backward or forward index pattern is invalid.

   ```js
   db.coll.find().sort({ a: 1, b: 1, c: 1 });
   ```

### Multikey Indexes

`Multikey index` is an index of a field that holds an array of data.

Example Data:

```js
{
  _id: ObjectId("57d7a121fa937f710a7d486d"),
  productName: "MongoDB Long Sleeve T-Shirt",
  categories: ["T-Shirts", "Clothing", "Apparel"],
  stock: [
    { size: "S", color: "red", quantity: 25 },
    { size: "S", color: "blue", quantity: 10 },
    { size: "M", color: "blue", quantity: 50 },
  ]
}
```

#### Example 1: Creating a multikey index for `categories` field that holds an array of data

```js
db.products.createIndex({ categories: 1 });
```

#### Example 2: Creating a multikey index for nested documents of `stock`

```js
db.products.createIndex({ "stock.quantity": 1 });
```

#### Example 3: Only one field whose values are in an array is allowed for each document

`productName`: a single value field
`stock.quantity`: a multi-value array
`categories`: a multi-value array

```js
possible: db.products.createIndex({ productName: 1, "stock.quantity": 1 });

impossible: db.products.createIndex({ categories: 1, "stock.quantity": 1 });
```

> NOTE: `Multikey Indexes` don't support covered queries.

### Partial Indexes

Example Data:

```js
{
  _id: ObjectId("57d7a121fa937f710a7d486d"),
  name: "Han Dynasty",
  cuisine: "Sichuan",
  stars: 4.4,
  address: {
    street: "90 3rd Ave",
    city: "New York",
    state: "NY",
    zipcode: "10003"
  }
}
```

#### Example 1. Basic partial index

Scenario: About 99%of queries will focus on getting restaurants that are rated more than 3.5 starts. So instead of writing a compound index like this:

```js
db.restaurants.createIndex({
  "address.city": 1,
  cuisine: 1,
  starts: 1,
});
```

we can have partial index where we create index on `address.city` and `cuisine` only if `stars` above 3.5:

```js
db.restaurants.createIndex(
  { "address.city": 1, cuisine: 1 },
  { partialFilterExpression: { stars: { $gte: 3.5 } } }
);
```

#### Example 2. Sparse index

Another type of a partial index where we create index only if the field exists.

The above functionality can be simplified into:

Simplified:

```js
db.restaurants.createIndex({ stars: 1 }, { sparse: true });
```

More Expressive:

```js
db.restaurants.createIndex(
  { stars: 1 },
  { partialFilterExpression: { stars: { $exists: true } } }
);
```

Using more expressive partial index we can have a sparse index based on different conditions:

```js
db.restaurants.createIndex(
  { stars: 1 },
  { partialFilterExpression: { cuisine: { $exists: true } } }
);
```

#### Example 3. Weird Case

1. add new document in a collection

   ```js
   db.restaurants.insert({
     name: "Han Dynasty",
     cuisine: "Sichuan",
     stars: 4.4,
     address: {
       street: "90 3rd Ave",
       city: "New York",
       state: "NY",
       zipcode: "10003",
     },
   });
   ```

1. create explainable object and run a query

   ```js
   > var exp = db.restaurants.explain()
   > exp.find({'address.city': 'New York', cuisine: 'Sichuan'})
   ```

1. result: `COLLSCAN` which makes sense because we don't have any indexes yet.

   ```js
   winningPlan: {
     stage: "COLLSCAN";
   }
   ```

1. create a partial index

   ```js
   db.restaurants.createIndex(
     { "address.city": 1, cuisine: 1 },
     { partialFilterExpression: { stars: { $gte: 3.5 } } }
   );
   ```

1. query again

   ```js
   exp.find({ "address.city": "New York", cuisine: "Sichuan" });
   ```

1. result: WTF `COLLSCAN`!!!!???? This is because in order to use a partial index, the query must be guaranteed to match the subset of the documents specified by the filter expression.

   ```js
   winningPlan: {
         stage: 'COLLSCAN',
         filter: {
           '$and': [
             { 'address.city': { '$eq': 'New York' } },
             { cuisine: { '$eq': 'Sichuan' } }
           ]
         },
         direction: 'forward'
       }
   ```

1. in order for us to trigger an index scan utilizing `partial indexes` we need to include the stars predicate.

   ```js
   exp.find({
     "address.city": "New York",
     cuisine: "Sichuan",
     stars: { $gt: 4.0 },
   });
   ```

### Text Indexes

It allows us to search by the text easily.

Example Data:

```js
{
  _id: ObjectId("57d7a121fa937f710a7d486d"),
  productName: "MongoDB Long Sleeve T-Shirt",
  category: "Clothing"
}
```

Let's say we want to search by the `productName.` There are several options.

#### Option1. Use good old `find()`

```js
db.products.find({ productName: "MongoDB Long Sleeve T-Shirt" });
```

But a user must put correct words word-by-word which is unlikely and very inefficient

#### Option2. Use mongodb's built-in regex

```js
db.products.find({ productName: /T-Shirt/ });
```

You can use mongodb's built-in regular expression functionality but this might have a performance issue.

### Option3. Use `Text Indexes`

```js
db.products.createIndex({ productName: "text" });
```

This is likely a best solution to query a document that has a specific `text` match to its value. We can use something like this:

```js
db.products.find({ $text: { $search: "t-shirt" } });
```

Under the hood this works very similar to `Multikey Indexes.` The server will process, let's say, for our text index of the string "MongoDB Long Sleeve T-Shirt" it will create the following indexes:

- mongodb
- long
- sleeve
- t
- shirt

#### Downsides of using `Text Indexes`

- More keys to examine
- Increased index size
- Increased time to build index
- Decreased write performance

### Strategies to reduce number of index keys to be examined when using `Text Indexes`

1. Create compound index with `Text Indexes.` This way you can significantly reduce the number of examined `Text Indexes` keys to be confined to a certain `category`

   ```js
   db.products.createIndex({ category: 1, productName: "text" });
   ```

   usage:

   ```js
   db.products.find({
     category: "Clothing",
     $text: { $search: "t-shirt" },
   });
   ```

### Example 1. Text Index Basic Query

1. Insert two sample data to a new collection.

   ```js
   db.textExample.insertOne({ statement: "MongoDB is the best" });
   db.textExample.insertOne({ statement: "MongoDB is the worst" });
   ```

1. Create `text index` of `statement`

   ```js
   db.textExample.createIndex({ statement: "text" });
   ```

1. Find documents based on `text`

   ```js
   db.textExample.find({ $text: { $search: "MongoDB best" } });
   ```

   Result:

   ```js
   [
     {
       _id: ObjectId("6170097e416903bc3f5f6553"),
       statement: "MongoDB is the best",
     },
     {
       _id: ObjectId("61700989416903bc3f5f6554"),
       statement: "MongoDB is the worst",
     },
   ];
   ```

   Interesting thing to note here is that when we search for `MongoDB Best` we get both `MongoDB is the best` and `MongoDB is the worst`. This is because each text key is examined separately and `OR` operation is included in the operation. So we are searching any document that has either `MongoDB` or `best`.

### Example 2. Text Index Enhanced Query

So to address the issue we mentioned above. We can use `textScore` to get some sort of a quality point for each query.

1. project `textScore` at the end of a query

   ```js
   db.textExample.find(
     { $text: { $search: "MongoDB best" } },
     { score: { $meta: "textScore" } }
   );
   ```

2. Result: a new field `score` is added to each queried document which indicates the quality of the match in a decimal number.

   ```js
   [
     {
       _id: ObjectId("6170097e416903bc3f5f6553"),
       statement: "MongoDB is the best",
       score: 1.5,
     },
     {
       _id: ObjectId("61700989416903bc3f5f6554"),
       statement: "MongoDB is the worst",
       score: 0.75,
     },
   ];
   ```

3. We can further use `sort` to order it by the quality of the search.

   ```js
   db.textExample
     .find(
       { $text: { $search: "MongoDB best" } },
       { score: { $meta: "textScore" } }
     )
     .sort({ score: { $meta: "textScore" } });
   ```
