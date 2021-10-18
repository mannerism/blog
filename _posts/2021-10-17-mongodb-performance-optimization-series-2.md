---
title: "MongoDB Performance Optimization Part 2"
date: 2021-08-21 08:15:00 +0900
permalink: ":categories/backend/:title"
---

> Reference [MongoDB University](https://university.mongodb.com/mercury/M201/2021_October_12/chapter/Chapter_2_MongoDB_Indexes/lesson)

This document keeps all the performance enhancement tips related to QueenBee backend service.

## MongoDB Performance Optimization Part 2

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
