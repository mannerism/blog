---
title: "MongoDB Performance Optimization Part 1"
date: 2021-08-21 08:15:00 +0900
permalink: ":categories/backend/:title"
---

> Reference [MongoDB University](https://university.mongodb.com/mercury/M201/2021_October_12/chapter/Chapter_2_MongoDB_Indexes/lesson)

This document keeps all the performance enhancement tips related to QueenBee backend service.

## Basic MongoDB performance examination steps

### Installation

1. Check in a terminal if `mongosh` is installed in your local machine.

   ```zsh
   ls "$(which mongo | sed 's/mongo//')" | grep mongo
   ```

   expected:

   ```zsh
   mongo
   mongod
   mongodump
   mongoexport
   mongofiles
   mongoimport
   mongoreplay
   mongorestore
   mongos
   mongostat
   mongotop
   ```

1. If #1 is not fulfilled, install following [this guide](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-os-x/)
1. Once `mongosh` is installed, connect your db by clicking connect in mongodb atlas and copy and paste the string

   ```zsh
   mongosh "mongodb+srv://<projectName>.af13yu.mongodb.net/<dbName>" --username <username>
   ```

### Examination

1. once you are connected to db using `mongosh` set `explain` variable

   ```javascript
   exp = db.users.explain("executionStats");
   ```

1. Run sample queries using `explain` to see some detail information about the query

   ```javascript
   exp.find({ _id: ObjectId("616529c7fcddd") });
   ```

1. Examine the result. Key items to examine are:
   - `queryPlanner.winningPlan`
   - `executionStats.totalKeysExamined`
   - `executionStats.totalDocsExamined`
   - `executionStats.executionStages`

### Examination Example

#### When Properly Used Index

Note for `totalDocsExamined` and `nReturned`.

```js
{
  queryPlanner: {
    plannerVersion: 1,
    namespace: 'QUEENBEE.users',
    indexFilterSet: false,
    parsedQuery: { _id: { '$eq': ObjectId("616529c7fb24c81d8dfdcddd") } },
    winningPlan: { stage: 'IDHACK' },
    rejectedPlans: []
  },
  executionStats: {
    executionSuccess: true,
    nReturned: 1,
    executionTimeMillis: 0,
    totalKeysExamined: 1,
    totalDocsExamined: 1,
    executionStages: {
      stage: 'IDHACK',
      nReturned: 1,
      executionTimeMillisEstimate: 0,
      works: 2,
      advanced: 1,
      needTime: 0,
      needYield: 0,
      saveState: 0,
      restoreState: 0,
      isEOF: 1,
      keysExamined: 1,
      docsExamined: 1
    }
  },
  serverInfo: {
    host: 'queenbee-shard-00-01.aa91y.mongodb.net',
    port: 27017,
    version: '4.4.9',
    gitVersion: 'b4048e19814bfebac717cf5a880076aa69aba481'
  },
  ok: 1,
  '$clusterTime': {
    clusterTime: Timestamp({ t: 1634276962, i: 100 }),
    signature: {
      hash: Binary(Buffer.from("88d8880fdecbd0c1752d2b78760c8869aa371459", "hex"), 0),
      keyId: Long("6975140387509764097")
    }
  },
  operationTime: Timestamp({ t: 1634276962, i: 100 })
}
```

#### When index is not used

Note for `totalDocsExamined` and `nReturned`.

```js
{
  queryPlanner: {
    plannerVersion: 1,
    namespace: 'myFirstDatabase.people',
    indexFilterSet: false,
    parsedQuery: { last_name: { '$eq': 'Acevedo' } },
    winningPlan: {
      stage: 'COLLSCAN',
      filter: { last_name: { '$eq': 'Acevedo' } },
      direction: 'forward'
    },
    rejectedPlans: []
  },
  executionStats: {
    executionSuccess: true,
    nReturned: 10,
    executionTimeMillis: 26,
    totalKeysExamined: 0,
    totalDocsExamined: 50474,
    executionStages: {
      stage: 'COLLSCAN',
      filter: { last_name: { '$eq': 'Acevedo' } },
      nReturned: 10,
      executionTimeMillisEstimate: 12,
      works: 50476,
      advanced: 10,
      needTime: 50465,
      needYield: 0,
      saveState: 50,
      restoreState: 50,
      isEOF: 1,
      direction: 'forward',
      docsExamined: 50474
    }
  },
  serverInfo: {
    host: 'nodeexpressproject-shard-00-02.wsxzr.mongodb.net',
    port: 27017,
    version: '4.4.9',
    gitVersion: 'b4048e19814bfebac717cf5a880076aa69aba481'
  },
  ok: 1,
  '$clusterTime': {
    clusterTime: Timestamp({ t: 1634277299, i: 12 }),
    signature: {
      hash: Binary(Buffer.from("c81291e5c758c6b4b9708d04555418b4475fa280", "hex"), 0),
      keyId: Long("6959643930757431297")
    }
  },
  operationTime: Timestamp({ t: 1634277299, i: 12 })
}
```

### Creation of Index

1. Pick a field that you want to add index on and create one

   ```js
   db.users.createIndex({ nickName: 1 });
   ```

   This will ensure that when you search by nickName you don’t have to run CollectionScan of entire documents. This will result in O(1)

1. You can also create index of sub-documents

   ```js
   db.users.createIndex( { profile.height :1 } )
   ```

   This will ensure the faster search when you search by `profile.height`

> NEVER ADD INDEX ON A FIELD THAT POINTS TO A SUB-DOCUMENT!! You should just add index directly to the sub-document field.

### Explain()

There are three types of explain()

1. `queryPlanner`: see detail query info before executing
2. `executionStats`: see detail query info when actually executing a query
3. `allPlansExecution`: see most verbose query info when actually executing a query

```js
exp = db.people.explain("queryPlanner");
expRun = db.people.explain("executionStats");
expRunVerbose = db.people.explain("allPlansExecution");
```

### Performance Improvement Example 1.

---

#### Raw find() without any index

Query:

```js
expRun.find({ last_name: "Johnson", "address.state": "New York" });
```

Explain() result:

- `totalDocsExamined`: 50474
- `nReturned`: 7
- `examined-docs-to-returned-ratio`(D2R): 7200:1

`note`: We want D2R to be closer to each other.

```js
{
  queryPlanner: {

...

    winningPlan: {
      stage: 'COLLSCAN',
      filter: {
        '$and': [
          { 'address.state': { '$eq': 'New York' } },
          { last_name: { '$eq': 'Johnson' } }
        ]
      },
      direction: 'forward'
    },
    rejectedPlans: []
  },
  executionStats: {
    executionSuccess: true,
    nReturned: 7,
    executionTimeMillis: 28,
    totalKeysExamined: 0,
    totalDocsExamined: 50474,
    executionStages: {
      stage: 'COLLSCAN',
      filter: {
        '$and': [
          { 'address.state': { '$eq': 'New York' } },
          { last_name: { '$eq': 'Johnson' } }
        ]
      },

...
}
```

#### Raw find() with only `last_name` index

1. add index of `last_name`

   ```js
   db.people.createIndex({ last_name: 1 });
   ```

1. run a query

   ```js
   expRun.find({ last_name: "Johnson", "address.state": "New York" });
   ```

Explain() result:

- `totalDocsExamined`: 794
- `nReturned`: 7
- `examined-docs-to-returned-ratio`(D2R): 110:1

`note`: A bit more enhanced query performance

```js
{
  queryPlanner: {
...
    winningPlan: {
      stage: 'FETCH',
      filter: { 'address.state': { '$eq': 'New York' } },
      inputStage: {
        stage: 'IXSCAN',
...
  },
  executionStats: {
    executionSuccess: true,
    nReturned: 7,
    executionTimeMillis: 2,
    totalKeysExamined: 794,
    totalDocsExamined: 794,
...
}
```

#### Raw find() with `last_name` and `address.state` index

1. add index of `last_name` and `address.state`

   ```js
   db.people.createIndex({ "address.state": 1, last_name: 1 });
   ```

1. run a query

   ```js
   expRun.find({ last_name: "Johnson", "address.state": "New York" });
   ```

Explain() result:

- `totalDocsExamined`: 7
- `nReturned`: 7
- `examined-docs-to-returned-ratio`(D2R): 1:1

`note`: Best performance

```js
{
  queryPlanner: {
...
    winningPlan: {
      stage: 'FETCH',
      inputStage: {
        stage: 'IXSCAN',
        keyPattern: { 'address.state': 1, last_name: 1 },
        indexName: 'address.state_1_last_name_1',
 ...
      }
    },
    rejectedPlans: [
      {
        stage: 'FETCH',
        filter: { 'address.state': { '$eq': 'New York' } },
        inputStage: {
          stage: 'IXSCAN',
          keyPattern: { last_name: 1 },
          indexName: 'last_name_1',
  ...
    ]
  },
  executionStats: {
    executionSuccess: true,
    nReturned: 7,
    executionTimeMillis: 0,
    totalKeysExamined: 7,
    totalDocsExamined: 7,
 ...
 ...
}
```

### Sort.explain()

Run `sort()` after `find()` and attach `explain()` at the end.

```zsh
> var res = db.people
      .find( { "last_name":"Johnson", "address.state":"New York" } )
      .sort( { "birthday":1 } )
      .explain( "executionStats" )

// Here we can just look at `executionStages`
> res.executionStats.executionStages
```

Result:

- note: execution order is IXSCAN > FETCH > SORT
- When we are executing SORT in memory usage of 2950 occurred when memLimit is 33MB. This means that when we have a sort operation that exceeds 33MB, then the server will cancel that query. So we can predict whether the data will exceed 33MB limit by calculating the total based on how many documents we are returning and how big is each document.

```js
{
  stage: 'SORT',
  ...
  memLimit: 33554432,
  totalDataSizeSorted: 2950,
  inputStage: {
    stage: 'FETCH',
 ...
    inputStage: {
      stage: 'IXSCAN',
      nReturned: 7,
 ...
    }
  }
}
```

### Sorting with Indexes

#### Example 1. Getting all documents after sorting on an indexed field of `ssn`

1. create `explainable object` and run the `query` followed by `sort`

   ```js
   var exp = db.people.explain(`executionStats`);
   exp
     .find({}, { _id: 0, last_name: 1, first_name: 1, ssn: 1 })
     .sort({ ssn: 1 });
   ```

1. result: we scanned about 50k documents with `IXSCAN(index scan)` because we retrieved all 50k documents.

   ```js
   winningPlan: {
       inputStage: {
         stage: 'FETCH',
         inputStage: {
           stage: 'IXSCAN',
           keyPattern: { ssn: 1 },
           ...
         }
       }
   },
   executionStats: {
     executionSuccess: true,
     nReturned: 50474,
     executionTimeMillis: 94,
     totalKeysExamined: 50474,
     totalDocsExamined: 50474,
     executionStages: { ... },
   }
   ```

#### Example 2. Getting all documents after sorting on an unindexed field of `last_name`

1. Run the `query` on `Unindexed` field `last_name` followed by `sort`

   ```js
   exp
     .find({}, { _id: 0, last_name: 1, first_name: 1, ssn: 1 })
     .sort({ first_name: 1 });
   ```

1. result: This time, we did not examine any index keys. Therefore, we performed `COLLSCAN` and `in-memory-sort` which is a quite expensive operation.

   ```js
   {
     queryPlanner: {
       winningPlan: {
         stage: 'SORT',
         sortPattern: { last_name: 1 },
         memLimit: 33554432,
         type: 'simple',
         inputStage: {
           stage: 'PROJECTION_SIMPLE',
           transformBy: { _id: 0, last_name: 1, first_name: 1, ssn: 1 },
           inputStage: { stage: 'COLLSCAN', direction: 'forward' }
         }
       },
       rejectedPlans: []
     },
     executionStats: {
         executionSuccess: true,
         nReturned: 50474,
         executionTimeMillis: 95,
         totalKeysExamined: 0,
         totalDocsExamined: 50474,
     }
   }
   ```

#### Example 3. Getting all documents after sorting after creating new index on `last_name`

1. create new index on `last_name`

   ```js
   db.people.createIndex({ last_name: 1 });
   ```

1. run a query

```js
exp
  .find({}, { _id: 0, last_name: 1, first_name: 1, ssn: 1 })
  .sort({ last_name: 1 });
```

1. result: successfully ran `IXSCAN` without `in-memory-sort`

```js
{
  queryPlanner: {
    plannerVersion: 1,
      stage: 'PROJECTION_SIMPLE',
      transformBy: { _id: 0, last_name: 1, first_name: 1, ssn: 1 },
      inputStage: {
        stage: 'FETCH',
        inputStage: {
          stage: 'IXSCAN',
          keyPattern: { last_name: 1 },
        }
      }
    },
    rejectedPlans: []
  },
  executionStats: {
    executionSuccess: true,
    nReturned: 50474,
    executionTimeMillis: 95,
    totalKeysExamined: 50474,
    totalDocsExamined: 50474,
    ...
  }
}
```

#### Example 4. Getting matching documents after sorting with an indexed field

1. run a query: We are scanning matching documents where `ssn` starts with `555` and `sort` by `ssn` in a decending order.

   ```js
   exp
     .find({ ssn: /^555/ }, { _id: 0, last_name: 1, first_name: 1, ssn: 1 })
     .sort({ ssn: -1 });
   ```

1. result: we did both `match` and `sort` using index keys

   ```js
   {
     winningPlan: {
       stage: 'PROJECTION_SIMPLE',
       transformBy: { _id: 0, last_name: 1, first_name: 1, ssn: 1 },
       inputStage: {
         stage: 'FETCH',
         inputStage: {
           stage: 'IXSCAN',
           keyPattern: { ssn: 1 }
         }
       }
     },
     executionStats: {
       executionSuccess: true,
       nReturned: 49,
       executionTimeMillis: 0,
       totalKeysExamined: 51,
       totalDocsExamined: 49,
       executionStages: {...}
     }
   }
   ```
