I love popcorn. 🍿

As I'm writing this, I'm snacking on a bowl of multi-colored kernels, lightly seasoned with Flavacol (hint: this is the movie theater's secret) and I'm feeling a twinge of hubris in my eye...

"I should start a popcorn company..."

Well, that's a terrible idea, Ryan! But, for the sake of this tutorial, it's _the best idea you've had all week_. **In this tutorial, we're going to learn how to create a simple, GraphQL-based API for a popcorn company**.

All we need is a name. Seeing as how I'm quite fond of sarcasm, how about...

_Sarcastic Kernels: The Popcorn You Love to Hate to Eat™_

Perfect! And like any self-respecting nerd, it only seems right to start writing some code before we come up with a sound business plan. To get started, let’s wrap our heads around what GraphQL is and how it compares to REST.

# What is GraphQL?

GraphQL is a client-side query language coupled with a pattern — formally known as a "schema" — for organizing the creation, reading, updating, and deleting of data in your application (yeah, that CRUD).

We say "application", here, and not database because GraphQL is data-source-agnostic, meaning it doesn't care _where_ your data lives.

From the outside looking in, GraphQL can seem quite scary. Does the "Graph" part mean I have to learn about graph databases? Does the QL (query language) mean I have to learn I have to learn a brand new programming language?!

Not quite. To calm your nerves, the harsh truth is: GraphQL is just a dressed up `GET` or `POST` request.

Wait, what?!

Yep. While GraphQL as a whole _does_ introduce some new concepts for organizing and interacting with your data, behind the curtain, GraphQL still relies on a good ol' fashioned HTTP request to do its magic.

### Rethinking REST

Where GraphQL separates from a more familiar REST API is due to its _flexibility_. With REST, done properly, endpoints are typically designed from the perspective of a resource, or, a type of data in our application.

For example, a `GET` request to `/api/v1/flavors` would be expected to send us back a response that looks something like this:

```json
[
  {
   "id": 1,
    "name": "The Lazy Person's Movie Theater",
    "description": "That elusive flavor that you begrudgingly carted yourself to the theater for, now in the comfort of your own home, you slob!"
  }, {
    "id": 2,
    "name": "What's Wrong With You Caramel",
    "description": "You're a crazy person that likes sweet popcorn. Congratulations."
  }, {
    "id": 3,
    "name": "Gnarly Chili Lime",
    "description": "The kind of popcorn you make when you need a good smack in the face."}
]
```

Now, there's nothing terribly wrong with this, but let's consider our UI, or rather, how we intend to _consume_ this data.

If we wanted to display a simple list UI where all we had were the kinds of popcorn that are available (and nothing else), we might end up with a design like the one below.

![](https://cleverbeagle-cdn.s3.amazonaws.com/ponyfoo/graphql-example-list-ui.png)

And… we'd be in a bit of a jam. We can choose not to use the `description` field, sure, but are we just going to sit around and act like we _didn't_ send that to the client as well?!

Haha! What do you think this is? _Fantasy land_? Of course we are! And when someone asks "why is the app so slow for users?" in a few months, we'll just say "be right back, going to grab a coffee!" and then split town, never to be seen again.

It's not entirely our fault, though. To be fair, REST _is_ the data-fetching equivalent of going to a restaurant and being asked "What do you want? We'll give you what we have."

Jokes aside, in a real application this can be problematic. For example, we may display different traits for each kind of popcorn like pricing information, brand details, or dietary restrictions ("vegan popcorn!"). Rigid REST endpoints make delivering specific traits based on context a headache, leading to unnecessary performance overhead and frustration.

## How GraphQL improves REST

On the surface, this may seem like a small problem. "Who cares if we're sending unnecessary data to the client?" Well, let's add some context. GraphQL was invented at Facebook. Facebook serves millions of requests _per second_.

Translation? Every optimization counts.

Instead of saying "here's what's available," **GraphQL inverts the problem and asks "what do you need?"**

We can get back a response from GraphQL specific to the context where we're consuming data _without_ having to add a one-off endpoint, perform multiple requests, or write convoluted conditional code.

# How does GraphQL work?

Like we hinted at above, at it's core GraphQL relies on a simple `GET` or `POST` request for moving data to and _from_ the client. Unpacking that, GraphQL has two types of requests when it comes to reading (the R in CRUD), creating, updating, and deleting (The CUD in CRUD): queries and mutations.

All of those queries and mutations are sent as either a `GET` or `POST` request to a GraphQL server at a URL like `https://myapp.com/graphql`. More on that below.

### Understanding Queries

Queries are what you'd expect: a request for some data. We have the UI, and we need to fill it with data, so we make a _query_ to the server. With a traditional REST API, our query would come in the form of a GET request. With GraphQL, we introduce a new syntax for requesting data:

```
{
  flavors {
    name
  }
}
```

Wait, you mean JSON? Or is that a JavaScript object?

Neither.

The "QL" part of GraphQL stands for _query language_. Quite literally, this is a brand new language for writing data queries. That sounds more complicated than it is. Let's break down the above query.

```
{
  // The fields we want to query are written here.
}
```

All queries start from the "root query" and are known as fields. To save yourself a headache: it's best to refer to this as "query fields in my schema." Don't fret, we'll learn more about how those are defined in a bit. Here, we ask to query the `flavors` field on the root query.

```
{
  flavors {
    // The sub-fields we want for each flavor are written here.
  }
}
```

When querying a field, we also need to specify the sub-fields we want for each object in the response (even if we expect just a single object to be returned).

```
{
  flavors {
    name
  }
}
```

The end result? When we send this query to a GraphQL server, we get back a neat, tidy response like this:

```json
{
  "data": {
    "flavors": [
      { "name": "The Lazy Person's Movie Theater" },
      { "name": "What's Wrong With You Caramel" },
      { "name": "Gnarly Chili Lime" }
    ]
  }
}
```

Neat, right? To make this clear, if we ran the following query on another page:

```
{
  flavors {
    id
    name
    description
  }
}
```

we'd get back a response like this:

```json
{
  "data": {
    "flavors": [
      { "id": 1, "name": "The Lazy Person's Movie Theater", description: "That elusive flavor that you begrudgingly carted yourself to the theater for, now in the comfort of your own home, you slob!" },
      { "id": 2, "name": "What's Wrong With You Caramel", description: "You're a crazy person that likes sweet popcorn. Congratulations." },
      { "id": 3, "name": "Gnarly Chili Lime", description: "A friend told me this would taste good. It didn't. It burned my kernels. I haven't had the heart to tell him." }
    ]
  }
}
```

Super powerful! _Same endpoint, with a different response tailored to the context_.

If we wanted to get a single flavor, GraphQL queries accept arguments, too:

```
{
  flavors(id: "1") {
    id
    name
    description
  }
}
```

Here, we've hardcoded the specific `id` of a flavor we want to query, but we can also have a dynamic `id`:

```
query getFlavor($id: ID) {
  flavors(id: $id) {
    id
    name
    description
  }
}
```

On the first line, we give our query a name (this is arbitrary; we could replace `getFlavor` with `pizza` and this would still work) and define the variables that query expects. Here, we expect a possible variable `id` to be passed as an `ID` scalar type (more on these below).

Regardless of using a static or dynamic `id` to make our request, here's the response we can expect:

```json
{
  "data": {
    "flavors": [
      { "id": 1, "name": "The Lazy Person's Movie Theater", description: "That elusive flavor that you begrudgingly carted yourself to the theater for, now in the comfort of your own home, you slob!" }
    ]
  }
}
```

Nice! Hopefully your hamster is starting to spin. This is already great, but where GraphQL really starts to shine is with _nested fields_. Let's assume we had another field in our schema called `nutrition` that told us just how unhealthy our sarcastic kernels are:

```
{
  flavors {
    id
    name
    nutrition {
      calories
      fat
      sodium
    }
  }
}
```

What this _may_ look like is us having a nested `nutrition` object on each of our `flavors`. Not quite! With GraphQL, we can combine separate but related data sources into a single query, getting a response that gives us the benefit of nested data without having to denormalize everything in the database:

```json
{
  "data": {
    "flavors": [
      {
        "id": 1,
        "name": "The Lazy Person's Movie Theater",
        "nutrition": {
          "calories": 500,
          "fat": 12,
          "sodium": 1000
        }
      },
      ...
    ]
  }
}
```

That's a serious boon on productivity. But what about _updating_ data, does GraphQL give the same advantages?

### Understanding Mutations

Where _queries_ fetch data, _mutations_ are responsible for making changes to data. Alternatively, too, mutations can be used for a generic RPC (remote procedure call) for miscellaneous tasks like sending a user's data to a third-party API.

```
mutation updateFlavor($id: ID!, $name: String, $description: String) {
  updateFlavor(id: $id, name: $name, description: $description) {
    id
    name
    description
  }
}
```

Mutations rely on a similar syntax to queries. Here, we define a mutation `updateFlavor` with some variables: `id`, `name`, and `description`. Just like with our queries, we "wrap" a mutation field (defined on a similar _root mutation_) using the keyword `mutation`, followed by a name describing the mutation and a set of variables to pass along.

Those variables include _what we're trying to change_ or _mutate_. Notice, too, that after executing a mutation, we can ask for some fields _back_.

In this case, we want to get back the `id`, `name`, and `description` _after_ they've been mutated. This helps with things like optimistic UI, negating the need for a request following the update.

### Writing a schema and attaching it to a GraphQL server

So far, what we've been looking at is how GraphQL functions _on the client_. This is how we _make_ requests, but how do we respond to them?

#### The GraphQL server

In order to make a GraphQL request, we need to have a GraphQL _server_ to send it to. A GraphQL server is a regular ol' HTTP server (if you're a JavaScripter, think Express or Hapi) with a GraphQL _schema_ attached to it.

```js
import express from 'express'
import graphqlHTTP from 'express-graphql'
import schema from './schema'

const app = express()

app.use('/graphql', graphqlHTTP({
  schema: schema,
  graphiql: true
}))

app.listen(4000)
```

By "attached," we mean that requests received by that server are passed through the schema and then back to the client, kind of like an air filter in your house.

The "filtering" process that takes place is relative to the query or mutation you send from the client. Both queries and mutations are _resolved_ using functions associated with the fields defined on our _root query_ or _root mutation_ in our schema.

Above, we can see a mock HTTP server being created with the JavaScript library Express. Utilizing the `graphqlHTTP` function from the `express-graphql` package by Facebook, we "attach" our schema (here, assumed to be defined in another file) and start our server on port `4000` (i.e., `http://localhost:4000/graphql` is where we'll send requests from the client).

#### Types and resolvers

With a running server, we need to define the schema that we attach to it.

Recall that earlier, we talked about defining _fields_ on either a _root query_ or _root mutation_.

```
import gql from 'graphql-tag'
import mongodb from '/path/to/mongodb’ // For example. Assuming `mongodb` gives us a MongoDB connection.

const schema = {
  typeDefs: gql`
    type Nutrition {
      flavorId: ID
      calories: Int
      fat: Int
      sodium: Int
    }

    type Flavor {
      id: ID
      name: String
      description: String
      nutrition: Nutrition
    }

    type Query {
      flavors(id: ID): [Flavor]
    }

    type Mutation {
      updateFlavor(id: ID!, name: String, description: String): Flavor
    }
  `,
  resolvers: {
    Query: {
      flavors: (parent, args) => {
        // Assuming args equals an object like { id: '1' }
        return mongodb.collection('flavors').find(args).toArray()
      },
    },
    Mutation: {
      updateFlavor: (parent, args) => {
        // Assuming args equals an object like { id: '1', name: 'Movie Theater Clone', description: 'Bring the movie theater taste home!' }

        // Perform the update.
        mongodb.collection('flavors').update(args)

        // Return the flavor after the update.
        return mongodb.collection('flavors').findOne(args.id)
      },
    },
    Flavor: {
      nutrition: (parent) => {
        return mongodb.collection('nutrition').findOne({
          flavorId: parent.id,
        })
      }
    },
  },
}

export default schema
```

When it comes to defining fields in a GraphQL schema, there are two parts: `typeDefs` and `resolvers`.

`typeDefs` contain the _type definitions_ for the data in our application. For example, earlier we talked about retrieving a list of `flavors`. In order to do that, we need to do three things:

1. Tell our schema what a flavor's data looks like (above, by defining the `type Flavor` type).
2. Define a field on the root `type Query` field (above, the `flavors` property on the `type Query` value).
3. Define a _resolver function_ on the `resolvers.Query` object corresponding to the field we defined on the root `type Query` field.

Focusing on the `typeDefs`, this is where we tell our schema about the _shape_ of our data. In other words, we tell GraphQL about the different properties a piece of data might contain.

```
type Flavor {
  id: ID
  name: String
  description: String
  nutrition: Nutrition
}
```

The `type Flavor` definition says that "a flavor can contain an `id` as an `ID`, a `name` as a `String`, a `description` as a `String`, and `nutrition` as `Nutrition`."

For that last one, `nutrition`, we pass the name of _another type defined in our `typeDefs`_. Here, `type Nutrition` describes how nutrition data is shaped in our application.

> Notice that we're not saying in our database. A database is assumed in the example above, but your data can come from _any_ data source. Even a third-party API or a static file!

Just like we did for `type Flavor` we specify the names of the field a piece of `nutrition` data will have, assigning what GraphQL refers to as _scalar types_ to each property. As of writing, GraphQL recognizes [five built-in scalar types](https://graphql.org/learn/schema/#scalar-types):

- `Int`: A signed 32‐bit integer.
- `Float`: A signed double-precision floating-point value.
- `String`: A UTF‐8 character sequence.
- `Boolean`: `true` or `false`.
- `ID`: A unique identifier, often used to refetch an object or as the key for a cache. The ID type is serialized in the same way as a String; however, defining it as an ID signifies that it is not intended to be human‐readable.

In addition to these scalar types, we can also assign _custom types_ to a property like we did with the `Nutrition` type on the `nutrition` property of the `type Flavor` above.

```
type Query {
  flavors(id: ID): [Flavor]
}
```

On our root `type Query` (the "root query" we talked about earlier), we define the name of a _field_ that can be queried. When defining that field, too, we specify any arguments we might expect along with the type of data we expect to be returned.

In this example, we expect a possible `id` argument to be passed as an `ID` scalar type and expect an array of objects resembling the `Flavor` type in response.

### Wiring up a query resolver

With our `flavors` field defined on our root `type Query`, next, we define what's known as a _resolver function_.

This is where GraphQL more or less "stops." If we look in the `resolvers` object of our schema file and then in the `Query` object nested under that, we can see a property `flavors` assigned to a function. This `flavors` here is the _resolver function_ for the `flavors` field defined on our root `type Query`.

```
typeDefs: gql`…`,
resolvers: {
  Query: {
    flavors: (parent, args) => {
      // Assuming args equals an object like { id: '1' }
      return mongodb.collection('flavors').find(args).toArray()
    },
  },
  …
},
```

This resolver function takes a few different arguments: the `parent` query if one exists, the `args` passed to the query if any exist, and a missing `context` argument which gives us miscellaneous "context" data (e.g., the current user if we provide them when our server starts).

Inside our resolver, we do _whatever we need to do to resolve the query_. This is where GraphQL "quits caring" and leaves retrieving and returning data up to us. Again, this could be a call to a database, an API, a static file..._anything_.

> While GraphQL doesn't care where our data comes from, it _does _ care about what we return. We can return a JSON object, an array of JSON objects, or a Promise (which GraphQL will resolve for us).

Here, we use a mock call to a MongoDB database collection called `flavors`, passing in our `args` (if any exist) to a `.find()` call and returning what it finds as an array.

### Resolving nested fields

What may not be clear above is how our nested `nutrition` data is resolved. Remember: we're not actually storing the nutrition data _on_ each `flavor` but assume it lives in another databse collection/table.

While we _did_ tell GraphQL that our `type Flavor` might include some `nutrition` data in the shape of the `type Nutrition`, we didn't explain how to actually _resolve_ that data. Again: **the  `nutrition` data for a flavor is assumed to be in a different collection than the flavor data**.

```
  typeDefs: gql`
    type Nutrition {
      flavorId: ID
      calories: Int
      fat: Int
      sodium: Int
    }

    type Flavor {
      […]
      nutrition: Nutrition
    }

    type Query {…}

    type Mutation {…}
  `,
  resolvers: {
    Query: {
      flavors: (parent, args) => {…},
    },
    Mutation: {…},
    Flavor: {
      nutrition: (parent) => {
        return mongodb.collection('nutrition').findOne({
          flavorId: parent.id,
        })
      }
    },
  },
```

If we look close at the `resolvers` object on our schema, notice that we have `Query`, `Mutation`, and `Flavor`. These correspond to the types we defined in the `typeDefs` above.

Looking at the `Flavor` object, we can see the field `nutrition` being defined as a _resolver function_. What's unique about this is that we're defining this _on the `Flavor` type directly. In other words, we're saying "this is how we want you to resolve the `nutrition` field for any queries utilizing the `type Flavor`."

Inside, we do a familiar MongoDB query, but notice that we utilize the `parent` argument passed to the resolver function. The `parent` here is each iteration of the `flavors` field. For example, if we ask for all flavors at once like this:

```
{
  flavors {
    id
    name
    nutrition {
      calories
    }
  }
}
```

For each `flavor` returned by `flavors`, we'd pass it through the `nutrition` resolver defined on `resolvers.Flavor` as `parent`. If we look close, we can see that we utilize the `parent.id` field, referring to the `id` of the flavor we're currently iterating (looping) over.

We pass that `parent.id` to our database query, matching it to a (presumed) `flavorId` property on each `nutrition` item.

### Wiring up mutations

Conveniently, our knowledge of wiring up queries maps over perfectly to mutations. In fact, the process is nearly identical. If we take a look at our root `type Mutation`, we can see that we define a field `updateFlavor` accepting the arguments we specified on the client:

```
type Mutation {
  updateFlavor(id: ID!, name: String, description: String): Flavor
}
```

Here, we're saying "we expect the `updateFlavor` mutation to accept a possible `id` as an `ID` (the `!` tells GraphQL that this is _required_), `name` as a `String`, and `description` as a `String`." Additionally, once our mutation completes, we expect some data in return resembling the `Flavor` type (i.e., containing an `id`, `name`, `description`, and/or `nutrition`).

```
{
  typeDefs: gql`…`,
  resolvers: {
    Mutation: {
      updateFlavor: (parent, args) => {
        // Assuming args equals an object like { id: '1', name: 'Movie Theater Clone', description: 'Bring the movie theater taste home!' }

        // Perform the update.
        mongodb.collection('flavors').update(
          { id: args.id },
          {
            $set: {
              ...args,
            },
          },
        )

        // Return the flavor after the update.
        return mongodb.collection('flavors').findOne(args.id)
      },
    },
  },
}
```

Inside our resolver function for the `updateFlavor` mutation, we do what you might expect: interact with our database to change or _update_ the flavor.

Notice that immediately after we perform the update, we make a call back to our database to find the same flavor again and return it from our resolver. Why?

Remember: on the client, we expect a return value _after_ our mutation completes. In this example, we expect the `flavor` we just updated to be returned.

Couldn't we just return the `args` object? Yep! We could. The reason we choose _not_ to in this case is that we want to be 100% certain our database update succeeded. If we go fetch the data again and see that it's changed: all is well!

# Why would I want to use GraphQL?

Though it may not look like much, at this point we have a functioning—albeit simple—GraphQL API up and running!

As with any new tech, though, what may not be abundantly clear is why you'd even want to use this. To be fair, this _is_ a lot of moving parts. Why shouldn't we just stick to REST or talking to the database directly?

### You want to reduce the number of requests made from the client

Where a lot of apps get bogged down is in the number, frequency, and complexity of HTTP requests. While GraphQL doesn't completely eliminate requests, utilized properly, it can _significantly_ reduce the number of requests you make from the client (in many cases, down to just one).

Whether you're running an app with tons of users or an app with lots of data (e.g., an app for handling medical records), using GraphQL will definitely speed up client performance.

### You want to avoid denormalizing data just to compensate the UI

In applications with a lot of relational data, the "denormalization trap" can be quite common. While this works, it's by no means ideal and can slow things down unnecessarily. With GraphQL and nested queries, the need to denormalize your data is significantly reduced.

### You have multiple data sources to talk to from different apps

This problem can be solved in part with a traditional REST API, but it still leaves a problem: consistent querying from the client. Assuming you have a product with a web app, iOS app, Android app, and developer API, it's likely that you'll have to rig up querying _for each of those platforms_.

This translates to developing knowledge of multiple client implementations for HTTP requests, an inconsistent means for performing queries, and messy, platform-specific endpoints in your API (don't you "holier than thou" me, friend, you know you've done this before!).

## Is GraphQL perfect? Should I ditch my REST API today and switch to it?

Of course not! _Nothing is perfect_.

Where GraphQL most obviously falls short is its complexity. Writing a GraphQL schema introduces a lot of _mandatory_ steps in order to wire up your data. As you’re learning, this can be frustrating because what is missing from your schema may not always be obvious and errors on the client and server may be unhelpful.

Further, consuming GraphQL on the client is not standardized beyond the GraphQL query language. Different libraries exists for this, the most popular being Apollo and Relay, each with their own idiosyncrasies.

GraphQL is also just a specification. Packages like `graphql` (used internally by the `express-graphql` package from our examples) are just an _implementation_ of that specification. In other words, different implementations for different programming languages may interpret the specification differently. This can lead to problems for you and your team if you use multiple languages across projects.

GraphQL is an impressive step forward in handling data, though. It's by no means a silver bullet, but it's certainly worth experimenting with. A great way to get started is to think about a particularly messy part of your data process and try to implement it using GraphQL.

This is the great news: GraphQL can be implemented incrementally. _You don't have to go all in to start taking advantage of it_. This is a great way to get buy in from your team and stakeholders and create an opportunity to get your hands dirty.

Keep in mind: GraphQL is ultimately just a tool for doing a job. It's not "killing" anything. That said, it's worth familiarizing yourself with it and starting to apply in your apps. Anywhere you struggle with performance overhead or complex UIs like data dashboards, news feeds, and user profiles is a great place to get started.

Happy coding!
