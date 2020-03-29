---
title: GraphQL Basics 101
date: "2020-03-29T22:12:03.284Z"
tags: ["GraphQL", "NodeJS", "Backend"]
---

# What's Graph

![](https://i.loli.net/2020/03/29/AJmr51n6NZOl2Wu.png)

**the relationship of datas is graph**

# What's GraphQL

```graphql
query {
  # each field one line
  hello
  course
  # get an user with id & name field
  user {
    id
    name
  }
  # get all the users(array) with name & email
  users {
    name
    email
  }
}
```

# Why GraphQL

## Blog case in RESTful API

![](https://i.loli.net/2020/03/29/KB4HCYfPXNxpzqi.png)

## Blog case in GraphQL

![](https://i.loli.net/2020/03/29/9UCe5Zy3cJXn2vr.png)

## Difference

- GraphQL is fast with less requests
- GraphQL is more flexible because what should be fetched is decided by client
- GraphQL is easy to mantain with strong typing and itself is documentation

# How to Serve a GraphQL

## Schema (TypeDefs)

Define the shape of current query

```js
const typeDefs = `
  type Query {
    hello: String!
    course: String!
  }
`
```

## Resolver (Data Loader)

load data according to the schema

```js
const resolvers = {
  Query: {
    hello: () => "some string",
    course: () => "another string",
  },
}
```

## Combine Schema and Resolver as a running server

```js
import {GraphQLServer} from 'graphql-yoga
// schema and resolver definitions above
// ...
const server = new GraphQLServer({
  typeDefs,
  resolvers,
})
server(() => console.log('graphQL Server Running'))
```

## Types

### Scala Types

`String` `Boolean` `Int` `Float` `ID`(Unique Ids)

```js
const typeDefs = `
  type Query {
    id: ID!
    name: String!
    age: Int!
    employed: Boolean!
    gpa: Float
  }
`
```

### Complex Types

#### Schema

```js
const typeDefs = `
  type Query {
    me: User!
    
  }
  type User {
    id: ID!
    name: String!
    email: String!
    age: Int
  }
  type Post {
    id: ID!
    title: String!
    body: String!
    published: Boolean!
  }
`
```

#### Resolvers

```js
const resolvers = {
  Query: {
    me: () => ({
      id: "123",
      name: "West",
      email: "west@mail.com",
      // you don't have to return the optional field
    }),
  },
}
```

#### Query

```graphql
query {
  me {
    id
    name
    email
    # you can query an optional field
    age
  }
}
```

#### Result

```json
{
  "data": {
    "me": {
      "id": "123",
      "name": "weapon",
      "email": "weapon@mail.com",
      // optional field default to be null
      "age": null
    }
  }
}
```

### Operation Argument

```js
const typeDefs = `
  type Query {
    // name is optional provided, returns must be a string
    greet(name: String): String!
  }
`
const resolvers = {
  Query: {
    greet: (parent, args, ctx, info) => `Hello ${args.name || "There"}`,
  },
}
```

#### Query

```graphql
query {
  # you can pass name arg or not like just greet
  greet(name: 'West')
}
```

#### Result

```json
{
  "data": {
    "greeting": "Hello Mike!"
  }
}
```

### Array

#### From Server to Client

```js
const typeDefs = `
  type Query {
    grades: [Int!]!
  }
`;
const resolvers = {
  Query: {
    grades: return [100, 10]
  }
}
```

##### Query

```graphql
query {
  grades
}
```

##### Result

```json
{
  "data": {
    "grades": [100, 10]
  }
}
```

#### From Client to Server

```js
const typeDefs = `
  type Query {
    add(num: Init!): Int!
  }
`
const resolvers = {
  Query: {
    add: (parent, args) => args.num.reduce((prev, curr) => prev + curr, 0),
  },
}
```

##### Query

```graphql
query {
  data(num: [1, 2, 3])
}
```

##### Response

```json
{
  "data": {
    "add": 6
  }
}
```

### Implement the Graph

#### Raw Data

```js
// Mock Data
const users = [
  {
    id: "123",
    name: "West",
    email: "west@mail.com",
    posts: ["123"],
  },
  {
    id: "124",
    name: "Nancy",
    email: "nancy@mail.com",
    posts: ["124"],
    age: 28,
  },
]

const posts = [
  {
    id: "123",
    title: "First Post",
    body: "First Post Body",
    published: true,
    author: "123",
  },
  {
    id: "124",
    title: "Second Post",
    body: "Second Post Body",
    published: false,
    author: "124",
  },
]

const comments = [
  {
    id: "123",
    content: "first comment",
    post: "123",
    user: "123",
  },
  {
    id: "124",
    content: "second comment",
    post: "124",
    user: "124",
  },
]
```

#### Schema & Resolvers

```js
const typeDefs = `
    type Query {
        users(keyword: String): [User!]!
        posts(keyword: String): [Post!]!
        comments: [Comment!]!
        me: User!
    }

    type User {
        id: ID!
        name: String!
        email: String!
        age: Int
        posts: [Post!]!
        comments: [Comment!]!
    }

    type Post {
        id: ID!
        title: String!
        body: String!
        published: Boolean!
        author: User!
        comments: [Comment!]!
    }

    type Comment {
        id: ID!
        content: String!
        post: Post!,
        user: User!,
    }
`

const resolvers = {
  Query: {
    users: (parent, args) =>
      users.filter(({ name }) => name.includes(args.keyword || "")),
    posts: (parent, { keyword }) =>
      posts.filter(({ title }) => title.includes(keyword || "")),
    comments: () => comments,
    me: () => ({
      id: "123",
      name: "weapon",
      email: "weapon@mail.com",
    }),
  },
  Post: {
    author: (parent, args, ctx, info) =>
      users.find(user => user.id === parent.author),
    comments: parent => comments.filter(comment => comment.post === parent.id),
  },
  User: {
    posts: (parent, args, ctx, info) =>
      posts.filter(post => post.author === parent.id),
    comments: parent => comments.filter(comment => comment.user === parent.id),
  },
  Comment: {
    post: parent => posts.find(post => post.id === parent.post),
    user: parent => users.find(user => user.id === parent.user),
  },
}
```

#### Query

```graphql
# Write your query or mutation here
query {
  comments {
    id
    content
    post {
      id
      title
      comments {
        id
        content
      }
    }
    user {
      id
      name
      comments {
        id
        content
      }
    }
  }
}
```

#### Result

```json
{
  "data": {
    "comments": [
      {
        "id": "123",
        "content": "first comment",
        "post": {
          "id": "123",
          "title": "First Post",
          "comments": [
            {
              "id": "123",
              "content": "first comment"
            }
          ]
        },
        "user": {
          "id": "123",
          "name": "West",
          "comments": [
            {
              "id": "123",
              "content": "first comment"
            }
          ]
        }
      },
      {
        "id": "124",
        "content": "second comment",
        "post": {
          "id": "124",
          "title": "Second Post",
          "comments": [
            {
              "id": "124",
              "content": "second comment"
            }
          ]
        },
        "user": {
          "id": "124",
          "name": "Nancy",
          "comments": [
            {
              "id": "124",
              "content": "second comment"
            }
          ]
        }
      }
    ]
  }
}
```
