# GraphQL API Fundamentals

This exercise is part of the [React GraphQL Academy](http://reactgraphql.academy) learning material. The goal of the exercise is to learn how to implement [GraphQL Cursor Connections Specification](https://relay.dev/graphql/connections.htm).

## Learning objectives

- Understand the advantages of using the GraphQL Cursor Connections Specification
- Learn how to implement in JavaScrip a GraphQL API compliant with the specification
- Learn how to extend the implementation of the GraphQL Cursor Connections Specification and add things such as filters or total count.

## To get started

- `git clone https://github.com/reactgraphqlacademy/graphql-api-training.git`
- `cd graphql-api-training`
- `git checkout relay-v2`
- `yarn install` or `npm install`
- `yarn start` or `npm start`

## Exercise part 1 - Context

### 🥑 Before we start

- ⚠️ Don't forget to checkout the `relay-v2` branch, install the dependencies, and let me walk you through the code meanwhile.
- This exercise builds on top of [the fundamentals branch](https://github.com/reactgraphqlacademy/graphql-api-training/tree/fundamentals-v2)
- We use nodemon in the `start` script, so every time you save, the server will restart automatically.

### Tasks

- [ ] 1. Given the `services` object imported at the top of the file `src/server.js`, include the `services` in the context of the ApolloServer. You'll know it works because the following query will work on Playground, otherwise, you'll get an error such as `"message": "Cannot read property 'findTrainings' of undefined"`

```graphql
query {
  trainings {
    id
  }
}
```

- [ ] 2. In `src/schema.js`, replace all the `fetch` functions in all the Query resolvers with the corresponding function in the service. You'll know it works because all the queries will return ids like this `5e93558ae06e3d37d8f3705f` instead of `dis:421`

```graphql
query {
  discounts {
    id
  }
}
```

#### 🏋️‍♀️ Bonus exercise part 1

Congratulations, you've completed part 1! You've learned how to use the context for passing things that any resolver might need. In our exercise, we used context to pass all the functions used to "resolve" data.

In this bonus exercise you are going to use the context to pass the authentication scope to the resolvers. You have an example [here](https://www.apollographql.com/docs/apollo-server/data/resolvers/#the-context-argument). You can use the helper function `authScope` defined at the top of `src/server.js`.

## Exercise part 2 - GraphQL Cursor Connections Specification

### 🥑 Before we start

There are two reasons for which you might want to use the [GraphQL Cursor Connections Specification](https://relay.dev/graphql/connections.htm):
1- Provide an option for GraphQL clients to consistently handle [pagination best practices](https://graphql.org/learn/pagination/) with support for related metadata via a GraphQL server.
2- Standardize the way those patterns are exposed. This is especially important for public APIs.

### Tasks

- [ ] 3. Create a DiscountConnection type. 🕵️‍♀️ Hint, it's very similar to the TrainingConnection.
- [ ] 4. Create a DiscountEdge type. 🕵️‍♀️ Hint, it's very similar to the TrainingEdge.
- [ ] 5. Do you need to create another PageInfo type?
- [ ] 6. Replace the Query type field `discounts: [Discount!]` with your connection. You know it works because the following query will work:

```graphql
query {
  discounts {
    edges {
      cursor
      node {
        id
        code
      }
    }
    pageInfo {
      hasNextPage
      hasPreviousPage
      startCursor
      endCursor
    }
  }
}
```

🤔 Some thoughts about this exercise:

Some implementations of this spec suffix the field with "Connection", example discountsConnection. However, from the official [GraphQL documentation](https://graphql.org/learn/pagination/#plurals):

> The simplest way to expose a connection between objects is with a field that returns a plural type.

GraphQL is a strongly typed query language, which means it probably doesn't make much sense to embed the `type` on the field name.

#### 🏋️‍♀️ Bonus exercise part 2

A) Add a field to the Training type called "discounts" that returns a "connection" of discounts.

B) Would it make sense to do the same from Discount to Training?

## Exercise part 3 - Extending the GraphQL Cursor Connections Specification

### 🥑 Before we start

We can extend the "Connections" to accomodate particular use cases. One example is the `totalCount` field we added to the `TrainingConnection` type.

To complete this exercise you'll use [GraphQL input types](https://graphql.org/learn/schema/#input-types). Input types are not only used for mutations but also to pass complex objects to our queries.

The goal of this exercise it to add filtering and sorting to our `discounts` connection. Notice the service.findDiscounts is ready to handle our schema modifications. You'll only need to edit this file `src/schema.js`

### Tasks

- [ ] 7. We want to be able to filter discounts by training ID. Modify `DiscountFilter` to include the field `trainingId`. You need to add the type to each field of the input type, [example](https://graphql.org/learn/schema/#input-types).
- [ ] 8. Add a field `code` to the `DiscountOrderField` type.
- [ ] 9. Edit the type `DiscountOrder` and add the right types to each field.
- [ ] 10. Update the field `discounts` in the `Query` type to include all the connection [arguments](https://relay.dev/graphql/connections.htm#sec-Arguments). 🕵️‍♀️ Hint, it's **very similar** to the `trainings` field in the `Query` type. You'll now it works when the following query works:

```graphql
query {
  discounts(
    filter: { trainingId: "5e934e68e06e3d37d8f21b5b" }
    orderBy: { field: code, direction: DESC }
  ) {
    totalCount
    edges {
      node {
        id
        code
      }
    }
  }
}
```

🤔 Some thoughts about this exercise:

Some APIs expose a field called `filter` (for instance, [Gatsby](https://www.gatsbyjs.org/docs/graphql-reference/#filter) or GitHub) and others call it `where` (for instance, [Hasura](https://hasura.io/docs/1.0/graphql/manual/queries/query-filters.html)). Either case, it'll be up to you to do define the capabilities (e.g. [OR/ AND operators](https://github.com/graphql/graphql-js/issues/585#issuecomment-262402544)) of that field given your specific use cases.

#### 🏋️‍♀️ Bonus exercise part 3

Update the schema so the following queries work:

```graphql
query future {
  trainings(
    filter: { startDate: future }
    orderBy: { field: title, direction: DESC }
  ) {
    totalCount
    edges {
      node {
        id
        code
      }
    }
  }
}
```

```graphql
query past {
  trainings(
    filter: { startDate: past }
    orderBy: { field: objectives, direction: DESC }
  ) {
    totalCount
    edges {
      node {
        id
        code
      }
    }
  }
}
```

## Exercise part 4 - Interfaces, Fragments, and the Global Object Identification

> Consistent object access enables simple caching and object lookups

https://graphql.org/learn/global-object-identification/

The goal of this exercise is to implement [Global Object Identification](https://relay.dev/graphql/objectidentification.htm). Why is this specification important? Some GraphQL clients, like [Relay](https://relay.dev/docs/en/graphql-server-specification.html#object-identification), require it to be implemented on the GraphQL server in order to be compliant.

- [ ] 1. Add the following Node interface to your schema

```graphql
interface Node {
  id: ID!
}
```

- [ ] 2. Add the following field to the `Query` type

```graphql
node(id: ID!): Node
```

- [ ] 2. The `Training` and `Discount` types should implement the interface. More on how to implement an interface in GraphQL [here](https://graphql.org/learn/schema/#interfaces).

## License

This material is available for private, non-commercial use under the [GPL version 3](http://www.gnu.org/licenses/gpl-3.0-standalone.html).
