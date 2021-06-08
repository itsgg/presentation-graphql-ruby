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

### Source Code

- [https://github.com/itsgg/todo-app](https://github.com/itsgg/todo-app)

### API Documentation

- [https://documenter.getpostman.com/view/13745138/TzY7cD19](https://documenter.getpostman.com/view/13745138/TzY7cD19)

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

## Requests

### Document

- A request document contains queries, mutations, subscriptions and fragments

### Variables

- An object that represents values used in the document.

### Meta-information

- Which operation to execute if the document contains more than one operations.

```graphql
query Todo {
  todo(id: $userId) {
    title
    ...userInfo
  }
}

fragment userInfo on User {
  email
}
```

---

## Mutation

```ruby
module Mutations
  class Register < Types::BaseMutation
    type Types::UserType
    null false

    argument :email, String, required: true
    argument :password, String, required: true

    def resolve(email:, password:)
      User.create! email: email, password: password
    end
  end
end
```

```ruby
module Types
  class MutationType < Types::BaseObject
    field :register, mutation: Mutations::Register
  end
end
```

---

## Query Resolver

```ruby
module Resolvers
  class TodoResolver < Types::BaseResolver
    type [Types::TodoType], null: false

    def resolve
      user = context[:current_user]
      authorize! user, to: :manage_todos?
      user.todos
    end
  end
end
```

```ruby
module Types
  class QueryType < Types::BaseObject
    field :todos, resolver: Resolvers::TodoResolver
  end
end
```

---

## Subscriptions

```ruby
module Subscriptions
  class TodoCreated < GraphQL::Schema::Subscription
    payload_type Types::TodoType
    null true

    def subscribe
      nil
    end

    def update
      object
    end
  end
end
```

```ruby
module Types
  class SubscriptionType < Types::BaseObject
    field :todo_created, subscription: Subscriptions::TodoCreated
  end
end
```

---

## Configuring Subscriptions

```ruby
class TodoAppSchema < GraphQL::Schema
  use GraphQL::Subscriptions::ActionCableSubscriptions

  mutation(Types::MutationType)
  query(Types::QueryType)
  subscription(Types::SubscriptionType)
end
```

```ruby
class GraphqlChannel < ApplicationCable::Channel
  def subscribed
    @subscription_ids = []
  end

  def execute(data)
    # Snip....
    transmit({ result: result.subscription? ? { data: nil } : result.to_h, more: result.subscription? })
  end

  def unsubscribed
    @subscription_ids.each do |sid|
      TodoAppSchema.subscriptions.delete_subscription(sid)
    end
  end
end

```

---

## Triggering Subscriptions

```ruby
class Todo < ApplicationRecord
  after_commit :notify_new_todo

  private

  def notify_new_todo
    TodoAppSchema.subscriptions.trigger('todoCreated', {}, self)
  end
end
```

### Only to few clients

```ruby
subscription_scope :current_user_id
TodoAppSchema.subscriptions.trigger('todoCreated', {}, self, scope: self.user_id)
```

---

## Testing Query Resolvers

```ruby
require 'rails_helper'

module Resolvers
  RSpec.describe TodoResolver, type: :request do
    # snip...

    describe '.resolve' do
      it 'success' do
        post_to_graphql(query: gql, token: user.token, variables: {})
        json = JSON.parse(response.body)
        data = json['data']
        expect(data['todos'].count).to eq(todos.count)
      end

      it 'failure' do
        post_to_graphql(query: gql, token: 'INVALID', variables: {})
        json = JSON.parse(response.body)
        expect(json['errors']).to be_present
      end
    end
  end
end

```

---

## Testing Mutations

```ruby
require 'rails_helper'

module Mutations
  RSpec.describe CreateTodo, type: :request do
    # snip...
    describe '.resolve' do
      it 'success' do
        expect do
          post_to_graphql(query: gql, token: user.token, variables: { title: 'Change the world' })
        end.to change(Todo, :count)
      end

      it 'failure' do
        expect do
          post_to_graphql(query: gql, token: 'INVALID', variables: { title: 'Give up' })
        end.not_to change(Todo, :count)
      end
    end
  end
end
```

<style style="text/css">

h2 {
  margin-bottom: 0.5em !important;
}

h3 {
  margin-top: 0.5em !important;
}

</style>
