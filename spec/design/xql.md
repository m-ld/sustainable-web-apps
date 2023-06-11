# "xQL": A query language

To make such an application suitably easy to build well, we believe developers
need a language for interacting with data which is:

- **Expressive**—able express all the data access and mutation an application
  needs, without too much fuss;
- **Readable**—with meaning clearly understood to those reading; and
- **Resilient**—unlikely to lead to mistakes, both when writing code the first
  time and when modifying code later.

In this section, we outline a query language which fulfils these goals. It
supports both read queries and writes. For the purposes of this document, we
name this query language "xQL", where "x" is meant to be a placeholder. It is
expected that this language will merge with [json-rql](https://json-rql.org/)
eventually, but for now it will be considered as its own specification with its
own (temporary) name.

We take inspiration from several existing tools:

- Like [GraphQL](https://graphql.org/), xQL has a recursive, composable query
  structure (whereas SPARQL is often hard to compose well).

- Like GraphQL, xQL's results match the structure of the query.

- Like [Datomic Pull](https://docs.datomic.com/pro/query/pull.html), xQL is
  built as data structures, and natively integrates with existing JavaScript
  tools (whereas GraphQL has a novel string representation, requiring custom
  tools).
- Like [SPARQL](https://www.w3.org/TR/sparql11-query/), xQL queries match over
  the entire set of data, so they can "begin" anywhere (whereas GraphQL schemas
  are always rooted at a common `Query` node).

- Like [JSON-LD Frames](https://www.w3.org/TR/json-ld11-framing/), xQL queries
  are themselves JSON-LD, and are patterns to match in the data, resulting in
  similarly-shaped data. (But whereas JSON-LD Frames, generally return all of
  the data in a new shape, xQL queries are designed to limit the data to exactly
  what is asked for.)

The language is described below with varying degrees of precision. Bear in mind,
however, that this is a draft for analysis purposes and subject to change during
implementation. We have focused on the areas we believe are most fundamental.

## Reactive Observable Queries

Our data store presents a graph of data which can change (through the actions of
other collaborators) at any moment. Therefore, any application will need:

1. To be able to query the graph to get exactly the data they need, and
2. To get updates to that data as soon as they become available, in a form that
   makes them easy to present to the user.

Therefore, we will build a query language for this data store which can address
any data required, and a reactive interface to use that query language which
presents an [RxJS Observable](https://rxjs.dev/guide/observable) of successive
query results. This interface is convenient to use in any web framework or
library, including React, Angular, and the Web Components ecosystem.

### Queries

First, let's look at the form of xQL queries and the data they return. An xQL
query takes the form of a kind of JSON-LD document, much like JSON-LD Frames do.
The _result_ of a query always has the same structure as the query itself: an
object in the query corresponds to an object in the result with the same keys,
and an array in the query corresponds to an array in the result.

xQL defines certain keys with special meaning, beginning with `@`, just like
JSON-LD itself does. When these keys appear in queries, they also appear in
results, because the structures must match. Importantly, because these keywords
are not defined by JSON-LD, these key-value pairs are ignored entirely by
JSON-LD processors. This supports an important principle of xQL's design:

**The result of any xQL query is a valid JSON-LD document describing a subset of
the original data.** That is, the results of two queries run against the same
graph will never appear to "disagree". While in some query languages, the
meaning of query results depends on an understanding of the query that found
them ("The name is 'Luke Skywalker'"), xQL results carry their full context and
can be understood in isolation ("The SWAPI-name of
`https://swapi.dev/api/people/1/` is 'Luke Skywalker'"). Results can be
transported, cached, and merged with other results and always remain a correct
subset of the information in the original data source.

Let's illustrate with some examples. We'll use the data offered by
[SWAPI](https://swapi.dev/), the Star Wars API. This API offers a rich set of
data in a JSON format which is easily read as JSON-LD by attaching a context to
each resource.

The context:

```json
{
  "@context": {
    "@vocab": "http://swapi.dev/documentation#",
    "url": "@id",
    "height": { "@type": "xsd:integer" },
    "mass": { "@type": "xsd:integer" },
    "vehicles": { "@type": "@id" }
  }
}
```

Additionally, we add an additional `@type` key to each resource naming its type,
as this is not present in SWAPI's responses and [not possible to add with a
`@context`](https://github.com/w3c/json-ld-syntax/issues/386). The values for
`@type` are `Planet`, `Spaceship`, `Vehicle`, `Person`, `Film`, and `Species`.

#### Match by `@id`: What color is Luke's hair?

Suppose we have the IRI for [Luke Skywalker](https://swapi.dev/api/people/1/). Then we might ask:

```json
{
  "@context": { "@vocab": "http://swapi.dev/documentation#" },
  "@id": "https://swapi.dev/api/people/1/",
  "hair_color": "?"
}
```

Note the placeholder at `hair_color`. The query engine will attempt to find a
value for this key. We will get the result:

```json
{
  "@context": { "@vocab": "http://swapi.dev/documentation#" },
  "@id": "https://swapi.dev/api/people/1/",
  "hair_color": "blond"
}
```

As JSON-LD, this describes exactly one fact, "Luke's hair is blond", written in Turtle as:

```turtle
<https://swapi.dev/api/people/1/> <http://swapi.dev/documentation#hair_color> "blond" .
```

This is a (very small) subgraph of the original graph: a single triple which was
present in the original data.

#### Match by `name`: What color are Luke's eyes?

Suppose we _don't_ know Luke's IRI yet. Then we might ask:

```json
{
  "@context": { "@vocab": "http://swapi.dev/documentation#" },
  "name": "Luke Skywalker",
  "eye_color": "?"
}
```

Then we expect to get the following:

```json
{
  "@context": { "@vocab": "http://swapi.dev/documentation#" },
  "name": "Luke Skywalker",
  "eye_color": "blue"
}
```

Notice that the result, once again, has the same shape as the query. The query
serves as a template which the query engine "fills in". Notice also that what
this means in JSON-LD, "Something named 'Luke Skywalker' has blue eyes," is
again a subgraph of the original data:

```turtle
[] <http://swapi.dev/documentation#name> "Luke Skywalker" ;
   <http://swapi.dev/documentation#eye_color> "blue" .
```

(Techincally speaking, it's more proper to say that the original graph
[_entails_](https://w3c.github.io/rdf-semantics/spec/#simpleentailment) this
one, because of the blank node, but the idea is the same—it's a subset of the
original information.)

#### Graph traversal: What vehicles has Luke piloted?

What if we want know about related entities, such as the vehicles Luke has piloted?

```json
{
  "@context": { "@vocab": "http://swapi.dev/documentation#" },
  "name": "Luke Skywalker",
  "vehicles": [
    {
      "name": "?",
      "model": "?",
      "manufacturer": "?"
    }
  ]
}
```

The result:

```json
{
  "@context": { "@vocab": "http://swapi.dev/documentation#" },
  "name": "Luke Skywalker",
  "vehicles": [
    {
      "name": "Snowspeeder",
      "model": "t-47 airspeeder",
      "manufacturer": "Incom corporation"
    },
    {
      "name": "Imperial Speeder Bike",
      "model": "74-Z speeder bike",
      "manufacturer": "Aratech Repulsor Company"
    }
  ]
}
```

Notice that `vehicles` was an array in the query, and remains an array in the
result. By JSON-LD semantics, an array of a single value is equivalent to the
single value itself. However, in xQL, the result value will always appear in an
array exactly when it was in an array in the query, regardless of the number of
results. This makes the format of the result object easier to anticipate.

#### Filtering with operators

<!-- markdownlint-disable no-space-in-code -->

Suppose we want to find all of the Skywalkers—that is, all people whose name
ends in ` Skywalker`:

<!-- markdownlint-restore -->

```json
{
  "@context": { "@vocab": "http://swapi.dev/documentation#" },
  "@graph": [
    {
      "@id": "?",
      "@type": "Person",
      "name": { "@value": "?", "@strends": " Skywalker" }
    }
  ]
}
```

`@strends` here is our first xQL operator. It corresponds to [the SPARQL
function `STRENDS`](https://www.w3.org/TR/sparql11-query/#func-strends). This
query will give us:

```json
{
  "@context": { "@vocab": "http://swapi.dev/documentation#" },
  "@graph": [
    {
      "@id": "https://swapi.dev/api/people/1/",
      "@type": "Person",
      "name": { "@value": "Luke Skywalker", "@strends": " Skywalker" }
    },
    {
      "@id": "https://swapi.dev/api/people/11/",
      "@type": "Person",
      "name": { "@value": "Anakin Skywalker", "@strends": " Skywalker" }
    },
    {
      "@id": "https://swapi.dev/api/people/43/",
      "@type": "Person",
      "name": { "@value": "Shmi Skywalker", "@strends": " Skywalker" }
    }
  ]
}
```

Notice what this document means in JSON-LD: that there are three resources, all
people, with these three names. The `@strends` key describes to the application
(correct) information about the names, but doesn't represent any facts
explicitly in the original data—correspondingly, JSON-LD isn't aware of the
`@strends` key, and ignores it. Thus, the result (under JSON-LD) is still a
subgraph of the original data.

### Reactive Updates

Using pure RxJS observables, we can observe a query and subscribe to the
results:

```ts
const result$ = observeQuery({
  "@context": { "@vocab": "http://swapi.dev/documentation#" },
  "@graph": [
    {
      "@id": "?",
      "@type": "Person",
      hair_color: "?",
      name: { "@value": "?", "@strends": " Skywalker" },
    },
  ],
});

result$.subscribe((result) => {
  // Do something with the result
});
```

The query is executed immediately, and the observer receives the result
(asynchronously, because reading from the graph is itself always async):

```json
{
  "@context": { "@vocab": "http://swapi.dev/documentation#" },
  "@graph": [
    {
      "@id": "https://swapi.dev/api/people/1/",
      "@type": "Person",
      "hair_color": "blond",
      "name": { "@value": "Luke Skywalker", "@strends": " Skywalker" }
    },
    {
      "@id": "https://swapi.dev/api/people/11/",
      "@type": "Person",
      "hair_color": "blond",
      "name": { "@value": "Anakin Skywalker", "@strends": " Skywalker" }
    },
    {
      "@id": "https://swapi.dev/api/people/43/",
      "@type": "Person",
      "hair_color": "black",
      "name": { "@value": "Shmi Skywalker", "@strends": " Skywalker" }
    }
  ]
}
```

Next, either locally or remotely, the `hair_color` of
`https://swapi.dev/api/people/1/` is changed to "ash-brown". The observable
reacts, and emits a new result:

```json
{
  "@context": { "@vocab": "http://swapi.dev/documentation#" },
  "@graph": [
    {
      "@id": "https://swapi.dev/api/people/1/",
      "@type": "Person",
      "hair_color": "ash-brown",
      "name": { "@value": "Luke Skywalker", "@strends": " Skywalker" }
    },
    {
      "@id": "https://swapi.dev/api/people/11/",
      "@type": "Person",
      "hair_color": "blond",
      "name": { "@value": "Anakin Skywalker", "@strends": " Skywalker" }
    },
    {
      "@id": "https://swapi.dev/api/people/43/",
      "@type": "Person",
      "hair_color": "black",
      "name": { "@value": "Shmi Skywalker", "@strends": " Skywalker" }
    }
  ]
}
```

Then, the name of `https://swapi.dev/api/people/11/` is changed to "Darth
Vader". Again, the observable reacts, and emits:

```json
{
  "@context": { "@vocab": "http://swapi.dev/documentation#" },
  "@graph": [
    {
      "@id": "https://swapi.dev/api/people/1/",
      "@type": "Person",
      "hair_color": "ash-brown",
      "name": { "@value": "Luke Skywalker", "@strends": " Skywalker" }
    },
    {
      "@id": "https://swapi.dev/api/people/43/",
      "@type": "Person",
      "hair_color": "black",
      "name": { "@value": "Shmi Skywalker", "@strends": " Skywalker" }
    }
  ]
}
```

Note that `observeQuery()` as proposed here must already have some awareness of
the data store (the m-ld clone) it reads from. It could be constructed by the
application in terms of the clone (`const observeQuery =
queryObserverFor(clone)`), or it could be provided by the library and take the
clone as another argument (`observeQuery(clone, query)`). The exact API is still
to be determined based on what turns out to be most convenient.

In React:

```tsx
const Skywalkers = () => {
  const skywalkers = useQuery({
    "@context": { "@vocab": "http://swapi.dev/documentation#" },
    "@graph": [
      {
        "@id": "?",
        "@type": "Person",
        hair_color: "?",
        name: { "@value": "?", "@strends": " Skywalker" },
      },
    ],
  });

  return (
    <ul>
      {skywalkers.map((person) => (
        <li key={person["@id"]}>
          <a href={person["@id"]}>{person.name["@value"]}</a> has{" "}
          {person.hair_color} hair.
        </li>
      ))}
    </ul>
  );
};
```

In React, the `useQuery` hook is little more than a wrapper around
`observeQuery`. It simply returns each result emitted from the observable,
causing the component to re-render with new data. If we'd prefer the list items
to be their own component, we have two choices. In one approach, we can declare
`Person` to take `name` and `hair_color`, which ensures that `Skywalkers`
fetches those values and passes them along:

```tsx
const Person = ({
  id,
  name,
  hairColor,
}: {
  id: string;
  name: string;
  hairColor: string;
}) => (
  <li>
    <a href={id}>{name}</a> has {hairColor} hair.
  </li>
);

///

const Skywalkers = () => {
  const skywalkers = useQuery({
    "@context": { "@vocab": "http://swapi.dev/documentation#" },
    "@graph": [
      // This component is responsible for getting the bits we'll need to pass
      // to `Person`.
      {
        "@id": "?",
        "@type": "Person",
        hair_color: "?",
        name: { "@value": "?", "@strends": " Skywalker" },
      },
    ],
  });

  return (
    <ul>
      {skywalkers.map((person) => (
        <Person
          key={person["@id"]}
          id={person["@id"]}
          name={person.name["@value"]}
          hairColor={person.hair_color}
        />
      ))}
    </ul>
  );
};
```

This makes the `Person` component purely prop-driven, with no smarts of its own.
Alternatively, the child components can make and subscribe to their own queries
entirely:

```tsx
const Person = ({ id }: { id: string }) => {
  // Given an `id`, `Person` can query for its own data.
  const person = useQuery({
    "@context": { "@vocab": "http://swapi.dev/documentation#" },
    "@id": id,
    "@type": "Person",
    hair_color: "?",
    name: { "@value": "?", "@strends": " Skywalker" },
  });

  return (
    <li>
      <a href={person["@id"]}>{person.name["@value"]}</a> has{" "}
      {person.hair_color} hair.
    </li>
  );
};

///

const Skywalkers = () => {
  const skywalkers = useQuery({
    "@context": { "@vocab": "http://swapi.dev/documentation#" },
    "@graph": [
      // We only need the `@id` to pass to the child component.
      {
        "@id": "?",
        name: { "@value": "?", "@strends": " Skywalker" },
      },
    ],
  });

  return (
    <ul>
      {skywalkers.map((person) => (
        <Person key={person["@id"]} id={person["@id"]} />
      ))}
    </ul>
  );
};
```

Now the `Person` component is smarter, and depends on the query engine to
function. However, in this version, `Skywalkers` is not observing as many facts.
When Luke's hair turns ash-brown now, `Skywalkers` will not re-render, but
Luke's `Person` component will. But when Anakin changes his name to "Darth
Vader", `Skywalkers` _will_ re-render, and drop him from the list it displays.

## Writing Data

It is, of course, already possible to write data to a m-ld clone, and any
existing write APIs will trigger any observed queries as expected. However, the
API can be a bit clunky for UI-driven code.

```ts
const updateName = () => {
  meld.write({
    "@context": { "@vocab": "http://swapi.dev/documentation#" },
    // Must use `@delete` and `@insert` at the top level.
    "@delete": {
      "@id": "https://swapi.dev/api/people/11/",
      // Must explicitly delete any existing values of the `name`.
      name: "?",
    },
    "@insert": {
      "@id": "https://swapi.dev/api/people/11/",
      name: "Darth Vader",
    },
  });
};
```

Instead, we will introduce a newer write syntax which parallels the reading
queries above:

```ts
const updateName = () => {
  meld.write({
    "@context": { "@vocab": "http://swapi.dev/documentation#" },
    // Just like read queries, uses literal values for pattern-matching.
    "@id": "https://swapi.dev/api/people/11/",
    // Neatly replaces existing values with a new value with a single operator.
    name: { "@update": "Darth Vader" },
  });
};
```

These writes can describe more specific intentions, which allow them to handle
concurrency better:

```ts
const duelWithVader = () => {
  meld.write({
    "@context": { "@vocab": "http://swapi.dev/documentation#" },
    "@id": "https://swapi.dev/api/people/1/",
    mass: { "@minus": 1 },
  });
};
```

Whatever the value of Luke's `mass` is when this write reaches the domain, this
will reduce it by 1.

## TypeScript Support

The structure and syntax defined here has been selected for its utility for the
app developer. We can greatly ease the developer experience by offering strong,
useful types, in two ways:

1. In reads and writes themselves, we can offer types which make constructing
   the queries (and indeed _any_ JSON-LD document) more ergonomic in supportive editors.
2. In query _results_, we can offer sophisticated types based on the query
   itself, making the results object more ergonomic to work with and making it
   harder to introduce errors as the application changes over time.

### Useful types for JSON-LD documents

JSON-LD itself is not aware of the expected types of properties. However,
JSON-LD contexts do map properties from one name to another—thus, given the
expected types of various properties by their IRIs, a sophisticated system of
TypeScript types can map those types to their new names under a given JSON-LD
context.

The exact mechanics and API of this will be developed further during this
milestone, but so far we have built enough to confirm that this possible to
accomplish in TypeScript. The examples here demonstrate what is built so far.

Because of the current limitations of TypeScript's contextual typing inference,
we can only do this checking on an argument to a function call. In many cases,
this is just fine: if the document in question is a query, it will typically
appear as an argument to `observeQuery()` or `useQuery()`. For other cases, we
can provide an identity function which lets TypeScript apply the types properly
without actually doing anything at runtime. The examples below use such a
function, which we call here `jsonld()`.

First, we must define some property types. This can be in an interface which is
explicitly provided to the query mechanism, or it can be a global interface
which can be extended through declaration merging from anywhere in the
application. (This is reasonable in most cases, as the keys are IRIs, and
typically will only need a single type definition within a given application.)
The interface looks something like this:

```ts
interface PropertyTypes {
  "http://swapi.dev/documentation#name": string;
  "http://swapi.dev/documentation#height": number;
  "http://swapi.dev/documentation#mass": number;
}
```

That is, in this application, we expect that in any JSON-LD document, a
`"http://swapi.dev/documentation#height"` key maps to a `number`, and so on.
Now, if `jsonld()` is built with this `PropertyTypes` interface, we can expect:

```ts
// ✅ This has no problem:
jsonld({ "http://swapi.dev/documentation#height": 172 });

// ❌ But this is an error:
jsonld({ "http://swapi.dev/documentation#height": "Luke Skywalker" });
```

So far, this is not terribly interesting. But once contexts come into play, it's
much more interesting:

```ts
jsonld({
  "@context": {
    height: "http://swapi.dev/documentation#height",
  },
  // ✅ This has no problem:
  height: 172,
});

jsonld({
  "@context": {
    height: "http://swapi.dev/documentation#height",
  },
  // ❌ But this is an error:
  height: "Luke Skywalker",
});
```

That is, the JSON-LD context now allows TypeScript to correctly type _term_
keys, key names defined in the context, by mapping them to their IRIs.

This mapping not only provides type _checking_; it also enables _completions_ in
the editor, greatly improving ergonomics. For instance, suppose an editor
contains the following code:

```ts
jsonld({
  "@context": {
    /** The name of the entity. */
    name: "http://swapi.dev/documentation#name",
    /** A person's height. */
    height: "http://swapi.dev/documentation#height",
  },
  // <- Cursor here
});
```

Bringing up the editor's completion feature (`Ctrl-Space` by default in VS Code)
will display the options `name` and `height` for completion—and even better,
will display the docstrings given for each of them. (It would be preferable to
display a docstring defined within `PropertyTypes`, but unfortunately [this is
not currently possible in
TypeScript](https://github.com/microsoft/TypeScript/issues/50715). If this issue
is resolved, however, we will be able to add that ability.) possible.

JSON-LD contexts propagate to child nodes, and these types propagate in the same
way. Thus:

```ts
jsonld({
  "@context": {
    name: "http://swapi.dev/documentation#name",
    parent: "https://schema.org/parent",
  },

  name: "Luke Skywalker",
  // ❌ This is an error, because `height` isn't in context here.
  height: 172,

  // `parent` is in the context, and typed as `unknown` (not in `PropertyTypes)
  parent: {
    "@context": {
      // The term `height` is only defined under this node.
      height: "http://swapi.dev/documentation#height",
    },
    // ✅ This isn't an error, because `name` is still in context here.
    name: "Anakin Skywalker",
    // ✅ This isn't an error, because `height` is now in context.
    height: 188,
  },
});
```

JSON-LD contexts are defined with a great deal of complexity. We intend to
implement typing for the majority of that complexity, including [Default
Vocabulary](https://www.w3.org/TR/json-ld11/#default-vocabulary), [Compact
IRIs](https://www.w3.org/TR/json-ld11/#compact-iris) and [Scoped
Contexts](https://www.w3.org/TR/json-ld11/#scoped-contexts). We may leave less
common features for future work, such as [Imported
Contexts](https://www.w3.org/TR/json-ld11/#imported-contexts) and [Protected
Term Definitions](https://www.w3.org/TR/json-ld11/#protected-term-definitions).

Most notably, type-scoped contexts will yield a more schema-like definition, as
each `@type` can define its own set of properties.

### Query results

Once the application has query results, it has to do something with them.
Teasing apart the results object is error-prone without the support of type
checking and the editor ergonomics it affords.

```tsx
const data = useQuery({
  "@context": { "@vocab": "http://swapi.dev/documentation#" },
  name: "Luke Skywalker",
  height: "?",
  vehicles: [
    {
      name: "?",
      model: "?",
      manufacturer: "?",
    },
  ],
});

return (
  <div>
    {data.name} ({data.height / 100}m) has piloted{" "}
    {data.vehicles.map((v) => v.name).join(", ")}
  </div>
);
```

There's a strong coupling here between the structure of the query and the access
into the `data` object below. Without a well-typed `data` object, this code can
easily break in the face of a typo, and if either the query or the access
changes in the future, only human vigilence or testing will catch if they drift
apart. The problem is even worse if we, say, pass the vehicles to another
component to render, as we then have two, possibly distant components to keep in
sync.

In the GraphQL community, this is typically solved with types, but as TypeScript
doesn't understand GraphQL natively, this either requires manually typing `data`
and ensuring that type stays up to date as code changes, or running a separate
toolchain to automatically build type definitions from the queries and schema.

In xQL, we have the advantage of both the queries and the "schema" being native
to TypeScript: the query is an object, and the equivalent of a schema is defined
by the `PropertyTypes` interface. Since the result by definition has the same
structure as the query itself, and since we can already type the query, we can
also translate the query into a type for the result data. Thus, in the query
above, TypeScript derives the type for `data` as:

```ts
const data: {
  "@context": {
    "@vocab": "http://swapi.dev/documentation#";
  };
  name: "Luke Skywalker";
  height: number;
  vehicles: {
    name: string;
    model: string;
    manufacturer: string;
  }[];
};
```

As of this analysis, the implementation in progress of query result types are
quite bare-bones: they very simply map the structure of the query to a result
object type, taking query placeholders into account, and do no term resolution
using the context. Having established in the JSON-LD document types described
above that we can do that term resolution, we are confident we can reuse it for
result typing.
