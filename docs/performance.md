---
title: Performance
---

Basically, **TypeGraphQL** is an abstraction layer built on top of the reference GraphQL implementation for Javascript - [`graphql-js`](https://github.com/graphql/graphql-js). It not only allows for building the GraphQL schema using classes and decorators but also gives a set of tools that focuses on developer experience and allow for making common tasks easy - authorization, validation, custom middlewares and others.

While this allows for easy and convenient development, it's sometimes a tradeoff in performance.

## Benchmarks

To measure the overhead of the abstraction, a few demo examples were made to compare the usage of TypeGraphQL against the implementations using bare metal - `graphql-js`. The benchmarks are located in a [folder on the GitHub repo](https://github.com/MichalLytek/type-graphql/tree/master/benchmarks).

The most demanding cases like returning an array of 10 000 nested objects shows that it's about 4 times slower. In real apps (with db access, etc.) it usually is a much lower factor but still not negligible. That's why TypeGraphQL has some built-in performance optimization options.

## Optimizations

When we have a query that returns a huge amount of JSON-like data and we don't need any field-level access restrictions, we can turn off whole authorization and middlewares stack for selected field resolvers using a `{ simple: true }` decorator option, e.g.:

```typescript
@ObjectType()
class SampleObject {
  @Field()
  sampleField: string;

  @Field(type => [SuperComplexObject], { simple: true })
  superComplexData: SuperComplexObject[];
}
```

This simple trick can speed up the execution up to 72%! The benchmarks shows that using simple resolvers allows for as fast execution as with bare `graphql-js` - the measured overhead is only about ~20%, which is a much more reasonable value than 400%.

You can also apply this behavior for all field of selected object types, using a `{ simpleResolvers: true }` decorator option, e.g.:

```typescript
@ObjectType({ simpleResolvers: true })
class Post {
  @Field()
  title: string;

  @Field()
  createdAt: Date;

  @Field()
  isPublished: boolean;
}
```

This optimization is not turned on by default mostly because of the global middlewares and auth feature. By using "simple resolvers" we are turning them off, so we have to be aware about the consequences - `@Authorized` guard on fields won't work for that fields and global middlewares won't be executed for that fields.

That's why we should **be really careful with using this feature**. The rule of thumb is to use "simple resolvers" only when it's really needed, like returning huge array of nested objects.
