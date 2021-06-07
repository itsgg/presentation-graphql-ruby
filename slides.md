---
theme: default
class: "text-center"
download: true
highlighter: shiki
info: |
  GraphQL Ruby 101 (Ruby Edition).
  https://itsgg.com
---

<!-- markdownlint-disable no-duplicate-header -->
<!-- markdownlint-disable no-inline-html -->

# GraphQL 101 <br /> (Ruby Edition)

---

## Introduction

- **Graph**: data structure **QL**: query language.
- Query language for data API.
- Declarative, flexible, and efficient.
- Optimize data communication between client and server.
- JSON is commonly used for data communication.
- Allows API introspection.

<!--
Don't confuse with Graph Databases.
Avoids under-fetching and over-fetching.
-->

---

<center>
  <img src="/images/overview.png" GraphQL Overview />
</center>

---

## GraphQL Way

### Typed Schema

- All fields have types, either primitive or custom.

### Declarative Language

- The client language is declarative instead of imperative.

### Single endpoint and client language

- These allows easy client access, without under-fetching or over-fetching.

### Simple versioning

- Versioning is avoided all together.
- To maintain compatbility add new fields and types without removing the old ones.

---

## Cons

### Security

- Overly complex queries can be used for resource-exhaustion attacks.
- Implement cost analysis on the query in advance.
- Enforce limits on the amount of data that can be consumed.
- Implement timeout to kill long running requests.
- Implement other general API security restrictions like whitelist, rate limiting etc.

### Caching and optimizing

- REST APIs can easily be cached because of dictionrary like nature, using URL as the cache key.
- In graphql we can use the query text as key to cache its response.
- We can normalize a query response into a flat collection of records with a global unique ID.

### Learning curve

- Steep learning curve.
- Have to learn the syntax of a new declarative language.
- Have to understand concepts like schemas, resolvers etc.

---

## Sample Application

A simple TODO application (Ruby/React)

- Signup
- Login
- Create a todo.
- Update a todo.
- Delete a todo.
- View realtime updates.

---

## Schema

Also called as

- Schema Definition Language (SDL)
- Interface Definition Language (IDL)

```graphql
type Todo(id: Int!) {
  title: String!
  complete: Boolean!
}
```

> Exclamation(!) after the types means that they cannot be empty

<!--
The Todo model can be looked by with an integer id.
-->

---

## Queries

Represent **READ** operations.

### Request

```graphql
query {
  todo(id: 1) {
    id
    title
    complete
  }
}
```

### Response

```json
{
  "data": {
    "todo": {
      "id": 1,
      "title": "Buy milk",
      "complete": false
    }
  }
}
```

---

## Mutations

Represent **WRITE**-then-**READ** operations.

### Request

```graphql
mutation {
  createTodo(title: "Buy Dogecoin!", complete: true) {
    todo {
      id
      title
      complete
    }
  }
}
```

### Response

```json
{
  "data": {
    "todo": {
      "id": 2,
      "title": "Buy Dogecoin!",
      "complete": true
    }
  }
}
```

---

## Subscriptions

Continous **READ** operations

### Request

```graphql
subscription todosChanged {
  todo {
    id
    title
    complete
  }
}
```

### Response

```json
{
  "data": {
    "todos": [
      { "id": 1, "title": "Buy milk", "complete": false },
      { "id": 2, "title": "Buy Dogecoin!", "complete": true }
    ]
  }
}
```

---

<style style="text/css">

h2 {
  margin-bottom: 0.5em !important;
}

h3 {
  margin-top: 0.5em !important;
}

</style>
