# GraphQL vs tRPC vs REST

## GraphQL

As opposed to REST's HTTP methods, GraphQL uses queries, mutations, and subscriptions for sourcing and manipulating data. **Queries** request data from the server while **mutations** send data to and modify data gated by the server. **Subscriptions** get live updates when data changes, usually through [websockets](https://blog.logrocket.com/websocket-tutorial-real-time-node-react/).

GraphQL is not designed to talk with your database. It is designed for the client to query the server for information. Tying your API to the database can create a range of problems when it comes to scaling your app. When you connect your frontend directly to the database with GraphQL, they need to have similar architectures so they can communicate effectively. In many cases, you'll be limited to the changes you can make on the frontend because of the interface that you'll need to complement them on the backend.

### Representing data with schemas

A **schema** defines an object, the fields that the object contains, and the type of data the fields can hold. Supported types include Int, Float, String, Boolean, ID.

```graphql
type User {
  name: String!
  age: Number!
}
```

This means that for any query operating on the `User` object, the only fields that can appear are `name` and `age`. For both of these fields, the only kinds of data that can be written to or read from are strings and numbers, respectively. The exclamation marks means that the fields are non-nullable.

### Defining queries, mutations, and subscriptions with schemas

Request a list of users in the database, specifically their names and ages:

```graphql
query {
  users {
    name
    age
  }
}
```

Create new user or update details of existing user:

```graphql
mutation {
  newUser($name: name!, $age: age!) {
    name
    age
  }
}
```

With a subscription schema, you can subscribe to an object and have your server update your client in real-time whenever changes are made to that object.

```graphql
subscription User {
  newUser {
    name
    age
  }
}
```

### GraphQL Pros

- Significant performance benefits (resource efficient)
  - Focuses on giving client exactly the data it needs. Does this by wrapping requests around queries so they describe specific requirements for what information needs to be returned. REST, on the other hand, returns all the data from an endpoint.
  - More flexible because result structure can be customized, which saves memory and CPU consumption that goes into filtering data.
  - Usually processed faster than REST requests because they generally request less data.
- The type of data needs to be typed in properties, which can greatly reduce the frequency of bugs on the frontend.
  - Types make sure client can only request data the server is capable of delivering. Also means more helpful and easy-to-understand errors.
  - Can add new fields to the data model on the server side without the need to create a new versioned API.

### GraphQL Cons

- Serious [caching problems](https://travislsmith.com/caching-problems-with-graphql-at-the-edge/)(specifically HTTP caching)
- Requesting too many nested fields at once can lead to circular queries and crash your server. Implementing rate-limiting in REST is more seamless.
- Does not support file upload out of the box

## tRPC

- TypeScript Remote Procedure Call
- define a set of procedures that can be called by a client and enables the server to respond with data
- is the most simple and lightweight library for remotely calling backend functions on the client side
- aims to provide developers with TS inference to make communication between backend and frontend more productive
- will require TS on both backend and frontend
- tRPC is not a schema but a protocol (or method) for exposing backend functions to the frontend
- simplifies your API by making the backend and frontend work more closely together to ultimately result in a more lightweight and better-performing app
- does not use schemas or code generation to build APIs
- frontend uses procedures to remotely call backend data
  - procedures are composable
  - two types of procedures: queries and mutations
- when working with tRPC, you'll find it much easier to use a monorepo, as tRPC's type definitions are generated from your TS code itself
  - monorepos are highly beneficial for things like version control and git history

## REST

REST (representational state transfer) APIs use HTTP methods like `GET`, `POST`, `PUT`, and `DELETE` to access and manipulate data with URIs (uniform resource identifiers).

- `GET` methods retrieve data from a server
- `POST` methods transfer data from the client to the server
- `PUT` methods modify data in the server
- `DELETE` methods erase data in the server

For example, a `GET` request might perform on a URI that looks like `/api/users`. A REST operation typically returns data from the server to the client and can come in formats like JSON, CSV, XML, or RSS.

### REST Pros

- Caching is easy because it takes advantage of HTTP by eliminating need for client and server to constantly interact with each other, improving performance and scalability
- Maturity in tech world
- Easier to monitor and rate-limit, as each resource is usually located behind a unique endpoint

### REST Cons

- Large payloads
- Different requests must be sent to different endpoints, resulting in multiple round trips. With GraphQL, we can batch different resource requests and send them to the same endpoint at `/graphql`.

## Use GraphQL if:

- want to separate backend and frontend (or work from two repos)
- GraphQL creates a standard for passing information between backend and frontend devs working on same project
- you're scaling your backend and need a clear set of rules for your API
- the application has increasingly more complex and intricate requirements
- TypeScript isn't an option for you; your backend or frontend devs prefer different technologies

## Use tRPC if;

- want to bring backend and frontend close together
- small to medium sized team of devs
- since it's a method rather than a schema, things are less clearly defined
- you don't have complex needs and don't forsee the need for using other (non-JS) languages in your app; your architecture and requirements are relatively straightforward
- app needs to be lightweight and simple; lot quicker to implement so dev times will be drastically reduced
- you're building a Next.js project
