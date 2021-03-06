# Introduction

This is an introduction to the Neo4j GraphQL mapping library (@neo4j/graphql). It also outlines requirements and where to get support. If you are already familiar with @neo4j/graphql, feel free to jump directly to the [reference](./reference.md) section.

## Requirements

@neo4j/graphql 1.0.x at minimum, requires:

-   [Neo4j](https://neo4j.com/) Database 4.1.0 and above.

-   [Apoc](https://neo4j.com/labs/apoc/) 4.1.0 and above

## Additional Resources

-   Version Control - https://github.com/neo4j/graphql
-   Bug Tracker - https://github.com/neo4j/graphql/issues

## What is neo4j/graphql ?

GraphQL to Cypher query execution layer for Neo4j and JavaScript GraphQL implementations. This library makes it easier to use Neo4j and GraphQL together. Translating GraphQL queries into a single Cypher query means users do not need to understand the Cypher Query language & can let the library handle all the database talking.

Using this library users can focus on building great applications while just writing minimal backend code.

### Goals of @neo4j/graphql

-   Provide an abstraction for GraphQL developers ontop of Neo4j.

-   Enable the integration with common community library's and tools.

## What's the difference from `neo4j-graphql-js` ?

> Checkout the original Neo4j GraphQL implementation [here](https://grandstack.io/)

When using the new library you will feel right at home, we have taken familiar fundamentals and concepts from neo4j-graphql-js and extended them. This library @neo4j/graphql is a fully supported Neo4j product. Here we look at the changes in the two implementations and also suggest some ways to migrate over to the new library.

### New Features

The latest and greatest stuff exclusive to this new implementation.

#### Nested Mutations

Use nested mutations to perform operations such as; create a post and connect it to an author, with just one GraphQL call leads to ultimately one database trip;

```graphql
mutation {
    createPosts(
        input: [
            {
                title: "nested mutations are cool"
                author: { connect: { where: { name: "dan" } } }
            }
        ]
    ) {
        id
    }
}
```

#### @auth

Define complex, nested & related, authorization rules such as; “grant update access to all moderators of a post”.

```graphql
type User {
    id: ID!
    username: String!
}

type Post
    @auth(
        rules: [
            {
                allow: [{ moderator: { id: "sub" } }] # "sub" being "req.jwt.sub"
                operations: ["update"]
            }
        ]
    ) {
    id: ID!
    title: String!
    moderator: User @relationship(type: "MODERATES_POST", direction: "IN")
}
```

### Excluded ~~Features~~

Features we have chosen to exclude for the first version of @neo4j/graphql.

#### Relationship Properties

We found the existing implementation [here](https://grandstack.io/docs/graphql-relationship-types/), where you have to use the 'top-level' relation directive;

```graphql
type Rated @relation(name: "RATED") {
    ....
}
```

Tricky to reason about. Before adding this feature back in we want to explore some more expressive ideas and take any community feedback on board. **The library doesn't know the concept of relationship properties** meaning you cannot; create, read, or filter by properties on a relationship.

#### Top Level Union's

In `neo4j-graphql-js` users could query top-level Unions such as;

```graphql
union Search = Genre | Movie
```

```graphql
query {
	Search {
		... on Genre {}
		... on Movie {}
	}
}
```

In the new implementation, **you cannot do this.** We made this decision on the fact that we had to create nodes with multiple labels causing issues. **You can use unions on a `@relationship`.**

#### Interface Querying

Similar to the reasons states in the Top Level Unions... we found that adding multiple labels onto a node can sometimes cause more problems it's trying to solve plus if you take into consideration the complexity. In the version, **users cannot query top-level Interfaces Nor use them as a `@relationship`.** In this implementation interfaces give you no real database support therefor no; query, update, delete, filter support. But instead used as a language feature to safeguard your schema. Great for When dealing with repetitive or large schemas you can essentially put "The side railings up".

#### Further Excluded Features

1. Multiple Databases
1. Additional Labels
1. GraphQL Architect
1. Indexes and Constraints
1. Inferring a Schema
1. Federation Support - We found federation very specific to Apollo users & not beneficial for our greater audience
