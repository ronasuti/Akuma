# **悪魔 : Akuma**
An opinionated Rust GraphQL API gateway that handles things... Differently.

## Differently?
These days, when making GraphQL APIs, you want three things:
1. Extensibility: How easy is it to add business logic, your own authentication/authorization, etc.
2. Algorithmic superiority: How well your GraphQL server avoids N+1 queries and, hopefully, pushes joins down to the DB
3. Gateway superiority: How many simple, often repeated queries can your API handle, testing gateway performance

Among current solutions, nothing provides all three out-of-the-box.

### Highly Integrated Server: Hasura
Hasura, for example, delivers 2 and 3 beautifully, but is a Brobdingnagian mass of Haskell code that, I suspect deliberately, isn't feasible to extend with features like caching, rate limiting, and business logic (without external servers). Hasura provides a cloud solution that *does* have these features if you desire, but, well, that's a cloud platform with the usual lock-in.

### Programming Framework: `async-graphql`
Then, there's solutions like async-graphql for Rust, which delivers 1 and 3 with aplomb. If you're using a fast web framework as the base like Actix Web, async-graphql absolutely *screams* performance-wise in contrived cases and, like many microframeworks in memory-safe languages, it's fairly ergonomic to use and extend. However, it doesn't have a good solution for nested querying which leaves the opportunity for N+1 database queries, which range from pretty bad for NoSQL databases like Cassandra to tragic for relational databases like PostgreSQL. Even with efficient handlers, however, joins still happen server-side unless you do extremely messy field inspection, which either gets very repetitive (coding every possible permutation of join you'll use) or basically involves writing Hasura from scratch.

### A Bit Of Both: Postgraphile
Lastly, you have solutions like Postgraphile, which is an alternative to Hasura that delivers 1 and 2. It's more flexible as it's a relatively simple Node.js server that has support for a menagerie of plugins while doing the same efficient GraphQL->SQL transpilation that gives Hasura a commanding performance advantage when using relational databases. However, Postgraphile *is* a Node.js server at the end of the day: translating GraphQL queries to SQL correctly and flexibly is compute-intensive enough that it will inevitably end up slower than Hasura.

## So what makes Akuma different?
To put it simply, it takes the "bit of both" style of Postgraphile and absolutely *cranks* it to 11.

To start with, this framework will begin as "just" an opinionated GraphQL framework, allowing a lot of the flexibility of other web API frameworks. You can create custom resolvers if you so desire, but you're totally fine just making simple REST endpoints. How? While the framework exclusively consumes GraphQL, it passes along the query generally in its entirety to Hasura. The advantage with this architecture is that Akuma can implement middleware, particularly caching, on the same system.

As this project continues, I will explore distributed caches like:
1. Redis client(s) for interop and managed solutions
2. Aerospike or Couchbase for easy in-mem distributed caching
3. Scylla for when a little more latency is okay, but you don't want to pay for tons of RAM