# Questions

1. **List and explain the core business drivers behind the NoSQL movement**? (5M)
Ans:
The core business drivers behind the NoSQL movement are **volume, velocity, variability, and agility**. They explain why organizations moved beyond traditional relational databases for applications that need massive scale, fast data handling, flexible schemas, and rapid development.

## 1. Volume

Modern businesses generate enormous amounts of data from social media, IoT devices, logs, transactions, mobile apps, and analytics systems. Traditional RDBMS systems often scale vertically—by adding more CPU, RAM, or storage to one server—which becomes expensive and eventually limited.

NoSQL databases are designed for **horizontal scaling** (scale-out): data is distributed across many commodity servers. This makes it easier and more cost-effective to store terabytes or petabytes of data.

**Example:** An e-commerce platform storing millions of product views, user sessions, and clickstream events can distribute this data across a NoSQL cluster instead of forcing one large relational server to handle everything.

## 2. Velocity

Velocity means both the **speed at which data is generated** and the **speed at which it must be read and written**. Public-facing applications often face sudden traffic bursts and require low-latency responses for real-time feeds, recommendations, messaging, gaming, and fraud detection.

A single-node RDBMS may struggle to sustain very high concurrent read/write throughput. NoSQL systems use distributed, parallel architectures to handle high throughput and provide fast access to frequently used data

**Example:** A live cricket-score app or stock-trading dashboard needs thousands of updates per second; NoSQL databases such as Redis or Cassandra are commonly used for such high-speed workloads.

## 3. Variability

Variability refers to the diversity and changing structure of data. Modern applications handle structured records, semi-structured JSON, logs, user profiles, images, videos, and sensor data—many of which do not fit neatly into fixed relational tables.

Relational databases require a predefined schema, so adding a new field often means altering tables and migrating data. NoSQL databases are typically schema-flexible, allowing different records to have different fields and making it easier to store evolving or heterogeneous data

**Example:** In a user-profile collection, one user may have `phone` and `address`, while another may have `socialLinks` and `preferences`. A document database such as MongoDB can store these variations without forcing every user record into the same columns.

## 4. Agility

Agility is the business need to develop, change, and release applications quickly. Agile teams frequently modify features, data models, and product requirements; rigid schemas and complex object-relational mapping can slow this process down

NoSQL reduces the “impedance mismatch” between application objects and database storage, especially for nested or hierarchical data such as JSON documents. This can improve developer productivity and shorten development cycles.

**Example:** If a product team adds “wishlist” or “recently viewed items” to a user profile, a document database can often accommodate the change with minimal schema migration, helping the team ship faster.


2. **Explain the CAP theorem and how it differs from ACID properties in databases**(5M)


3. **Define a Graph store and provide an example of solving a business problem effectively using a Graph store**
Ans: 

A **graph store** (or graph database) is a NoSQL database that stores data as a network of **nodes**, **relationships (edges)**, and **properties**. It is best when the most important question is not just “what data do we have?” but “how are these entities connected?”

## What is a graph store?

A graph store represents entities as **nodes** and their connections as **edges**. Both nodes and edges can contain properties, such as a user’s name or a relationship’s date

For example:

```
(Qais) --FOLLOWS--> (Rahul)
(Qais) --LIKED--> (Post 101)
(Rahul) --AUTHORED--> (Post 101)
```

Here, `Qais`, `Rahul`, and `Post 101` are nodes; `FOLLOWS`, `LIKED`, and `AUTHORED` are relationships; and details such as `name`, `post_id`, or `liked_at` are properties. Graph databases treat relationships as first-class data, so they can directly traverse multi-hop connections instead of repeatedly joining tables.

## Why graph stores are useful

|Feature|Meaning|Business benefit|
|---|---|---|
|Relationship-first model|Connections are stored directly, not reconstructed through foreign keys|Faster and simpler queries involving relationships. [[tigergraph](https://www.tigergraph.com/blog/graph-database-vs-nosql/)][[cognee](https://www.cognee.ai/graph-databases-explained)]|
|Multi-hop traversal|The database can follow chains such as user → friend → friend → product|Finds indirect patterns, such as fraud rings or recommendations. [[tigergraph](https://www.tigergraph.com/blog/graph-database-vs-nosql/)][[cognee](https://www.cognee.ai/graph-databases-explained)]|
|Flexible schema|New node types, relationships, and properties can be added easily|Supports evolving business domains without major redesign. [[nebula-graph](https://nebula-graph.io/posts/review-on-graph-databases)][[cognee](https://www.cognee.ai/graph-databases-explained)]|
|Natural modeling|Data is modeled like real-world networks|Suitable for social networks, supply chains, payments, and security. [[tigergraph](https://www.tigergraph.com/blog/graph-database-vs-nosql/)][[cognee](https://www.cognee.ai/graph-databases-explained)]|

## Example: fraud detection in a bank

A strong business use case is **detecting fraud rings in a banking or payment platform**. Fraudsters often create multiple accounts and hide their connection by using shared phone numbers, email IDs, devices, IP addresses, or transaction paths

### Graph model

Nodes can represent:

- Customers
- Bank accounts
- Phone numbers
- Email addresses
- Devices
- IP addresses
- Transactions

Edges can represent:

- `OWNS`
- `USES_PHONE`
- `USES_EMAIL`
- `LOGGED_IN_FROM`
- `TRANSFERRED_TO`

For example:

```
(Account A) --USES_PHONE--> (Phone 9876543210) <--USES_PHONE-- (Account B)
(Account B) --LOGGED_IN_FROM--> (Device X) <--LOGGED_IN_FROM-- (Account C)
(Account C) --TRANSFERRED_TO--> (Account D)
```

### Business problem solved

Suppose the bank wants to answer:

> “Is this new account connected, directly or indirectly, to accounts already flagged for fraud?”

A graph query can start from the new account and traverse its connected phone numbers, devices, emails, and transaction counterparts. If it reaches a known fraudulent account within a few hops, the system can flag the account for review or block a suspicious transaction in real time

### Why it is effective

In a relational database, this requires multiple joins across account, phone, device, IP, and transaction tables. As the number of hops increases, the query becomes complex and slow. A graph store follows the stored relationships directly, making pattern discovery such as “accounts linked through shared devices and common recipients” much more natural and efficient.

This helps the bank:

- Detect organized fraud instead of only individual suspicious accounts
- Identify indirect connections that simple rule-based checks may miss
- Reduce financial losses by acting before or during suspicious transactions
- Provide investigators with a visual network of related accounts and devices


4. List the architectural patterns in NoSQL databases and discuss the Key-Value and Document-Oriented patterns with their characteristics, use cases, and examples

The main architectural patterns in NoSQL databases are **key-value, document-oriented, column-family, graph, and object-oriented**. The first four are the most commonly used; each is designed for a different data model and workload.[[livebook.manning](https://livebook.manning.com/book/making-sense-of-nosql/chapter-4/)][[tutorialspoint](https://www.tutorialspoint.com/article/nosql-data-architecture-patterns)][[guru99](https://www.guru99.com/nosql-tutorial.html)]

## NoSQL architectural patterns

|Pattern|Data model|Best suited for|Examples|
|---|---|---|---|
|**Key-value**|Unique key maps to an opaque value|Caching, sessions, fast lookups|Redis, Memcached, DynamoDB, Riak [[docs.aws.amazon](https://docs.aws.amazon.com/zh_cn/whitepapers/latest/choosing-an-aws-nosql-database/types-of-nosql-databases.md)][[tutorialspoint](https://www.tutorialspoint.com/article/nosql-data-architecture-patterns)]|
|**Document-oriented**|Self-contained JSON/BSON/XML documents|User profiles, catalogs, content management|MongoDB, CouchDB, Firestore [[docs.aws.amazon](https://docs.aws.amazon.com/zh_cn/whitepapers/latest/choosing-an-aws-nosql-database/types-of-nosql-databases.md)][[geeksforgeeks](https://www.geeksforgeeks.org/nosql/types-of-nosql-databases/)]|
|**Column-family**|Rows partitioned by key, with column families|Time-series, IoT, event logs, high-write workloads|Cassandra, HBase, ScyllaDB [[airbyte](https://airbyte.com/top-etl-tools-for-sources/types-of-nosql-databases)][[tutorialspoint](https://www.tutorialspoint.com/article/nosql-data-architecture-patterns)]|
|**Graph**|Nodes, edges, and properties|Social networks, recommendations, fraud detection|Neo4j, OrientDB [[tutorialspoint](https://www.tutorialspoint.com/article/nosql-data-architecture-patterns)][[scribd](https://www.scribd.com/presentation/958794765/NoSQL-Data-Architecture-Patterns)]|
|**Object-oriented**|Objects similar to OOP classes|Applications needing direct object persistence|db4o, Perst [[tutorialspoint](https://www.tutorialspoint.com/article/nosql-data-architecture-patterns)]|

## Key-value pattern

A key-value store saves data as a pair of a **unique key** and a **value**. The key identifies the record, while the value may be a string, number, JSON object, serialized object, or binary blob; the database generally treats the value as opaque.[[docs.aws.amazon](https://docs.aws.amazon.com/zh_cn/whitepapers/latest/choosing-an-aws-nosql-database/types-of-nosql-databases.md)][[cs.clarku](https://cs.clarku.edu/~cs220/slides/22-NoSQL.pdf)]

### Characteristics

- **Simple API:** Applications mainly use operations such as `GET`, `PUT`, `SET`, and `DELETE`.
- **Fast key-based access:** Data is retrieved directly using its key, often through hashing or indexing, giving very low latency.
- **Schema-less values:** The value can have any structure, and different keys can store different data formats.[[cs.clarku](https://cs.clarku.edu/~cs220/slides/22-NoSQL.pdf)]
- **Limited querying:** The database usually cannot efficiently filter, join, or query inside the value because it does not understand its structure.[[geeksforgeeks](https://www.geeksforgeeks.org/nosql/types-of-nosql-databases/)][[cs.clarku](https://cs.clarku.edu/~cs220/slides/22-NoSQL.pdf)]
- **High scalability:** Keys can be partitioned across multiple servers, supporting horizontal scale-out.
- **High throughput:** Simple operations and in-memory designs make key-value stores suitable for millions of small concurrent operations.

### Use cases

- **Session storage:** Store logged-in user sessions, authentication tokens, and temporary login state.
- **Caching:** Cache database query results, API responses, or frequently accessed product details.
- **Rate limiting:** Track request counts per user, API key, or IP address.
- **Feature flags:** Store configuration such as `feature:newCheckout = true`.
- **Real-time leaderboards:** Maintain scores and rankings using Redis sorted sets.
- **Distributed locks:** Coordinate background jobs and prevent duplicate processing.

### Example

In an e-commerce application, a user’s session can be stored in Redis:

```
Key: session:abc123

Value:
{
  "userId": "u42",
  "role": "customer",
  "cart": [
    { "productId": "p101", "quantity": 2 }
  ],
  "expiresAt": "2026-11-10T18:30:00Z"
}
```

When the user makes a request, the backend calls:

```
GET session:abc123
```

Redis returns the complete session immediately. This avoids querying relational tables for user, cart, and cart-item data on every request, reducing latency and database load.

If the user adds an item, the backend updates the value:

```
SET session:abc123 -> updated cart JSON
EXPIRE session:abc123 3600
```

The `EXPIRE` command removes the session automatically after one hour of inactivity.

## Document-oriented pattern

A document-oriented database stores data as self-contained **documents**, commonly in JSON or BSON format. Each document contains fields and values, and related documents are grouped in collections; unlike relational tables, documents in the same collection do not need to have identical fields.[[docs.aws.amazon](https://docs.aws.amazon.com/zh_cn/whitepapers/latest/choosing-an-aws-nosql-database/types-of-nosql-databases.md)][[geeksforgeeks](https://www.geeksforgeeks.org/nosql/types-of-nosql-databases/)]

### Characteristics

- **Document-based storage:** Data is stored as JSON, BSON, or XML rather than rows and columns.[[docs.aws.amazon](https://docs.aws.amazon.com/zh_cn/whitepapers/latest/choosing-an-aws-nosql-database/types-of-nosql-databases.md)][[geeksforgeeks](https://www.geeksforgeeks.org/nosql/types-of-nosql-databases/)]
- **Flexible schema:** Documents can have different fields, so the data model can evolve easily.[[geeksforgeeks](https://www.geeksforgeeks.org/nosql/types-of-nosql-databases/)][[javatechonline](https://javatechonline.com/types-of-nosql-databases-and-examples/)]
- **Nested data support:** Arrays and nested objects can be stored inside one document, matching real-world application objects.[[docs.aws.amazon](https://docs.aws.amazon.com/zh_cn/whitepapers/latest/choosing-an-aws-nosql-database/types-of-nosql-databases.md)][[geeksforgeeks](https://www.geeksforgeeks.org/nosql/types-of-nosql-databases/)]
- **Queryable fields:** Unlike key-value stores, document databases can query, index, filter, and aggregate based on fields inside documents.[[docs.aws.amazon](https://docs.aws.amazon.com/zh_cn/whitepapers/latest/choosing-an-aws-nosql-database/types-of-nosql-databases.md)][[javatechonline](https://javatechonline.com/types-of-nosql-databases-and-examples/)]
- **Partial retrieval:** Applications can retrieve selected fields instead of always loading an entire document.[[docs.aws.amazon](https://docs.aws.amazon.com/zh_cn/whitepapers/latest/choosing-an-aws-nosql-database/types-of-nosql-databases.md)]
- **Reduced object-relational mismatch:** JSON-like documents often map naturally to objects in JavaScript, Node.js, and modern application code.[[geeksforgeeks](https://www.geeksforgeeks.org/nosql/types-of-nosql-databases/)]

### Use cases

- **Product catalogs:** Products have varying attributes, such as size for clothing or processor details for laptops.
- **User profiles:** Users may have different optional fields, preferences, addresses, and social links.
- **Content management systems:** Articles, blog posts, and media metadata fit naturally into documents.
- **Mobile and real-time apps:** Firestore is commonly used for chat apps and synchronized mobile data.[[geeksforgeeks](https://www.geeksforgeeks.org/nosql/types-of-nosql-databases/)]
- **REST APIs:** JSON documents map directly to API request and response bodies.
- **Event and activity storage:** Store semi-structured logs, notifications, and user activity records.

### Example

In MongoDB, a product can be stored as:

```
{
  "_id": "p101",
  "name": "Wireless Headphones",
  "brand": "SoundMax",
  "price": 2499,
  "category": "Electronics",
  "tags": ["audio", "bluetooth", "over-ear"],
  "specs": {
    "batteryLifeHours": 30,
    "noiseCancellation": true
  },
  "variants": [
    { "color": "Black", "stock": 25 },
    { "color": "White", "stock": 12 }
  ]
}
```

The application can query documents using fields:

```
db.products.find({
  category: "Electronics",
  price: { $lte: 3000 }
})
```

It can also create an index on `category` or `price` for faster queries. This is possible because the document database understands the internal structure of the document, unlike a key-value store where the value is opaque.[[docs.aws.amazon](https://docs.aws.amazon.com/zh_cn/whitepapers/latest/choosing-an-aws-nosql-database/types-of-nosql-databases.md)][[javatechonline](https://javatechonline.com/types-of-nosql-databases-and-examples/)]

