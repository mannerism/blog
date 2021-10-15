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

```json
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
