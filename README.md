# What is GraphQL
GraphQL is a query language for APIs and a runtime for fulfilling those queries with your existing data. GraphQL provides a complete and understandable description of the data in your API, gives clients the power to ask for exactly what they need and nothing more, makes it easier to evolve APIs over time, and enables powerful developer tools.

Some advantages with GraphQL :
- Ask for what you need, get exactly that
- Get many resources in a single request
- Describe what’s possible with a type system
- Move faster with powerful developer tools
- Evolve your API without versions
- Bring your own data and code

# Defining Schema
### GraphQL schema basics
Your GraphQL server uses a schema to describe the shape of your available data. This schema defines a hierarchy of types with fields that are populated from your back-end data stores. The schema also specifies exactly which queries and mutations are available for clients to execute.

The GraphQL specification defines a human-readable schema definition language (or SDL) that you use to define your schema and store it as a string.

Here's a short example schema that defines two object types: Book and Author:
```
type Book {
  title: String
  author: Author
}

type Author {
  name: String
  books: [Book]
}
```
A schema defines a collection of types and the relationships between those types. In the example schema above, a Book can have an associated author, and an Author can have a list of books.

Because these relationships are defined in a unified schema, client developers can see exactly what data is available and then request a specific subset of that data with a single optimized query.

Every type definition in a GraphQL schema belongs to one of the following categories:
- Scalar
- Object (This includes the three special root operation types: `Query`, `Mutation`, and `Subscription`.)
- Input
- Enum
- Union
- Interface

### GraphQL Resolver
Resolver is a collection of functions that generate response for a GraphQL query. In simple terms, a resolver acts as a GraphQL query handler. Every resolver function in a GraphQL schema accepts four positional arguments as given below
```
fieldName:(root, args, context, info) => { result }
```
Given below are the positional arguments and their description :
- root - The object that contains the result returned from the resolver on the parent field.
- args - An object with the arguments passed into the field in the query.
- context - This is an object shared by all resolvers in a particular query.
- info - It contains information about the execution state of the query, including the field name, path to the field from the root.

Resolvers in GraphQL can return different types of values as given below :
- null or undefined - this indicates the object could not be found
- array - this is only valid if the schema indicates that the result of a field should be a list
- promise - resolvers often do asynchronous actions like fetching from a database or backend API, so they can return promises
- scalar or object - a resolver can also return other values

# HTTP Request
Your GraphQL HTTP server should handle the HTTP GET and POST methods.
### GET request
When receiving an HTTP GET request, the GraphQL query should be specified in the "query" query string. For example, if we wanted to execute the following GraphQL query:
```
{
  me {
    name
  }
}
```
This request could be sent via an HTTP GET like so:
> http://myapi/graphql?query={me{name}}

### POST request
A standard GraphQL POST request should use the application/json content type, and include a JSON-encoded body of the following form:
```
{
  "query": "...",
  "operationName": "...",
  "variables": { "myVariable": "someValue", ... }
}
```
`operationName` and `variables` are optional fields. operationName is only required if multiple operations are present in the query. In addition to the above, we recommend supporting two additional cases:
- If the "query" query string parameter is present (as in the GET example above), it should be parsed and handled in the same way as the HTTP GET case.
- If the "application/graphql" Content-Type header is present, treat the HTTP POST body contents as the GraphQL query string.

### Response
Regardless of the method by which the query and variables were sent, the response should be returned in the body of the request in JSON format. As mentioned in the spec, a query might result in some data and some errors, and those should be returned in a JSON object of the form:
```
{
  "data": { ... },
  "errors": [ ... ]
}
```
### GraphQL Client
GraphQL client is code that makes a POST request to a GraphQL Server. In the body of the request we send a GraphQL query or mutation as well as some variables and we expect to get some JSON back. So really, the simplest GraphQL Client is curl.

# Mutations
Most discussions of GraphQL focus on data fetching, but any complete data platform needs a way to modify server-side data as well.

In REST, any request might end up causing some side-effects on the server, but by convention it's suggested that one doesn't use GET requests to modify data. GraphQL is similar - technically any query could be implemented to cause a data write. However, it's useful to establish a convention that any operations that cause writes should be sent explicitly via a mutation.

Just like in queries, if the mutation field returns an object type, you can ask for nested fields. This can be useful for fetching the new state of an object after an update. Let's look at a simple example mutation:

<table>
    <thead>
      <tr></tr>
      <tr></tr>
    </thead>
    <tbody>
        <tr>
            <td>
              mutation CreateReviewForEpisode($ep: Episode!, $review: ReviewInput!) {
                createReview(episode: $ep, review: $review) {
                  stars
                  commentary
                }
              }
          </td>
            <td rowspan=2>{
  "data": {
    "createReview": {
      "stars": 5,
      "commentary": "This is a great movie!"
    }
  }
}</td>
        </tr>
        <tr>
            <td>{
  "ep": "JEDI",
  "review": {
    "stars": 5,
    "commentary": "This is a great movie!"
  }
}</td>
        </tr>
    </tbody>
</table>

### Inline Fragments
Like many other type systems, GraphQL schemas include the ability to define interfaces and union types. If you are querying a field that returns an interface or a union type, you will need to use inline fragments to access data on the underlying concrete type. It's easiest to see with an example:

<table>
  <thead>
    <tr></tr>
    <tr></tr>
  </thead>
  <tbody>
    <tr>
      <td>query HeroForEpisode($ep: Episode!) {
  hero(episode: $ep) {
    name
    ... on Droid {
      primaryFunction
    }
    ... on Human {
      height
    }
  }
}</td>
      <td rowspan=2>{
  "data": {
    "hero": {
      "name": "R2-D2",
      "primaryFunction": "Astromech"
    }
  }
}</td>
    </tr>
    <tr>
      <td>{
  "ep": "JEDI"
}</td>
    </tr>
  </tbody>
</table>

### Meta fields
Given that there are some situations where you don't know what type you'll get back from the GraphQL service, you need some way to determine how to handle that data on the client. GraphQL allows you to request `__typename`, a meta field, at any point in a query to get the name of the object type at that point.

<table>
  <thead>
    <tr></tr>
    <tr></tr>
  </thead>
  <tbody>
    <tr>
      <td>{
  search(text: "an") {
    __typename
    ... on Human {
      name
    }
    ... on Droid {
      name
    }
    ... on Starship {
      name
    }
  }
}</td>
      <td>{
  "data": {
    "search": [
      {
        "__typename": "Human",
        "name": "Han Solo"
      },
      {
        "__typename": "Human",
        "name": "Leia Organa"
      },
      {
        "__typename": "Starship",
        "name": "TIE Advanced x1"
      }
    ]
  }
}</td>
    </tr>
  </tbody>
</table>

In the above query, search returns a union type that can be one of three options. It would be impossible to tell apart the different types from the client without the `__typename` field.

# Authentication
Authorization is a type of business logic that describes whether a given user/session/context has permission to perform an action or see a piece of data. For example:

> Only authors can see their drafts

Defining authorization logic inside the resolver is fine when learning GraphQL or prototyping. However, for a production codebase, delegate authorization logic to the business logic layer. Here’s an example:

```
//Authorization logic lives inside postRepository
var postRepository = require('postRepository');

var postType = new GraphQLObjectType({
  name: ‘Post’,
  fields: {
    body: {
      type: GraphQLString,
      resolve: (post, args, context, { rootValue }) => {
        return postRepository.getBody(context.user, post);
      }
    }
  }
});
```
In the example above, we see that the business logic layer requires the caller to provide a user object. If you are using GraphQL.js, the User object should be populated on the context argument or rootValue in the fourth argument of the resolver.

We recommend passing a fully-hydrated User object instead of an opaque token or API key to your business logic layer. This way, we can handle the distinct concerns of authentication and authorization in different stages of the request processing pipeline.

# Apollo Server
Apollo Server is an open-source, spec-compliant GraphQL server that's compatible with any GraphQL client, including Apollo Client. It's the best way to build a production-ready, self-documenting GraphQL API that can use data from any source.

![Apollo Server](https://www.apollographql.com/docs/apollo-server/ee7fbac9c0ca5b1dd6aef886bb695e63/index-diagram.svg)

You can use Apollo Server as:
- A stand-alone GraphQL server, including in a serverless environment
- An add-on to your application's existing Node.js middleware (such as Express or Fastify)
- A gateway for a federated graph

Apollo Server provides:
- Straightforward setup, so your client developers can start fetching data quickly
- Incremental adoption, allowing you to add features as they're needed
- Universal compatibility with any data source, any build tool, and any GraphQL client
- Production readiness, enabling you to ship features faster

