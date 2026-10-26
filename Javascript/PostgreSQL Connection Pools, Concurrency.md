

Tags: #nodejs #postgresql #backend #concurrency #system-design #interview

## 1. The problem

Suppose a CSV import endpoint receives up to 50,000 rows.

The application uses a shared PostgreSQL connection pool:

```js
const pool = new Pool({
  max: 10
});
```

All database-backed endpoints in this Node.js process use this same pool.

The original import handler is:

```js
app.post('/import', async (req, res) => {
  const rows = await parseCsv(req.file);

  await Promise.all(
    rows.map(row =>
      pool.query(
        'INSERT INTO leads(a, b) VALUES ($1, $2)',
        [row.a, row.b]
      )
    )
  );

  res.json({ ok: true });
});
```

The main problem is **unbounded concurrency**: the application submits all 50,000 operations immediately, although the pool has at most 10 connections.

> [!important] Main distinction
> A connection-pool limit controls database connections.
> An application concurrency limit controls outstanding operations.
> A batch size controls how many rows one operation handles.

These are three different controls.

---

## 2. What does `max: 10` mean?

```js
const pool = new Pool({
  max: 10
});
```

This means the pool can have at most 10 client connections.

It does not mean:

- Only 10 promises can exist.
- Only 10 requests can reach the application.
- Only 10 query operations can be submitted.
- Every request gets 10 connections.
- All 10 connections must always be open.

The pool creates connections as needed, up to the configured maximum.

### Example

If 50,000 queries are submitted and no other work is using the pool:

```text
Up to 10 queries use connections.
The remaining queries wait for a connection.
```

Conceptually:

```text
Application submits 50,000 queries
                  |
                  v
         PostgreSQL connection pool
         ┌────────────────────────┐
         │ Connection 1 → Query 1 │
         │ Connection 2 → Query 2 │
         │ ...                    │
         │ Connection 10 → Q10    │
         └────────────────────────┘

Waiting:
Q11, Q12, Q13, ..., Q50000
```

The exact number waiting changes as operations finish. Around 49,990 may be waiting initially.

### Connection reuse

When a query finishes:

1. The connection becomes available.
2. Another waiting operation receives it.
3. The connection executes that query.
4. The process repeats.

The application does not create 50,000 database connections.

---

## 3. What does `pool.query()` do?

```js
await pool.query(sql, params);
```

Conceptually:

```text
Acquire a client from the pool
        ↓
Execute the query
        ↓
Return the client to the pool
```

The pool manages acquisition and release automatically.

Example:

```js
const result = await pool.query(
  'SELECT * FROM leads WHERE id = $1',
  [leadId]
);
```

This is convenient for standalone queries.

---

## 4. What does `pool.connect()` do?

```js
const client = await pool.connect();
```

This checks out one specific client connection.

The application keeps that connection until it explicitly releases it:

```js
client.release();
```

Example:

```js
const client = await pool.connect();

try {
  await client.query('SELECT 1');
  await client.query('SELECT 2');
} finally {
  client.release();
}
```

Both queries use the same connection.

### Important distinction

```js
pool.query(...)
```

Uses automatic acquisition and release for each operation.

```js
client.query(...)
```

Uses the particular connection already checked out by the application.

> [!warning] Always release checked-out clients
> If clients are not released, the pool can eventually run out of available connections.
> New operations then wait for clients that never return.

---

## 5. Why `Promise.all(rows.map(...))` submits all operations

Consider:

```js
await Promise.all(
  rows.map(row => insertRow(row))
);
```

JavaScript evaluates the argument to `Promise.all()` first.

Therefore:

```js
rows.map(row => insertRow(row))
```

calls `insertRow()` for every row before `Promise.all()` starts waiting.

Conceptually:

```js
const p1 = insertRow(rows);
const p2 = insertRow(rows);[1]
const p3 = insertRow(rows);[2]

// ...

const p50000 = insertRow(rows[49999]);

await Promise.all([p1, p2, p3, /* ... */ p50000]);
```

### Key idea

`Promise.all()` observes promises. It does not limit how many operations are started.

Also, `Promise.all()` itself does not create threads or make database work run in parallel. Actual database concurrency depends on the available connections and database resources.

---

## 6. What happens to the original import request?

### Step 1: Parse the CSV

```js
const rows = await parseCsv(req.file);
```

The application obtains all parsed rows.

If the parser builds an array, all these rows are retained in application memory.

Writing `await` does not automatically make CPU-heavy parsing nonblocking. That depends on how the parser is implemented.

### Step 2: Submit all inserts

```js
rows.map(row => pool.query(...))
```

The application creates thousands of query promises and submits thousands of pool operations.

### Step 3: The pool distributes connections

At most 10 connections exist in this pool.

Queries that cannot acquire a connection wait in the pool queue.

### Step 4: The handler waits

```js
await Promise.all(...);
```

The handler waits for success or rejection.

While waiting for asynchronous I/O, the handler yields control. It does not block the JavaScript thread for the whole import.

### Step 5: Respond on complete success

```js
res.json({ ok: true });
```

This executes only if all input promises fulfill.

---

## 7. How other endpoints are affected

Suppose another endpoint uses the same pool:

```js
app.get('/profile', async (req, res, next) => {
  try {
    const result = await pool.query(
      'SELECT * FROM users WHERE id = $1',
      [req.user.id]
    );

    res.json(result.rows);
  } catch (error) {
    next(error);
  }
});
```

If the import has filled the pool and its waiting queue, the profile query may wait behind many import operations.

Possible consequences:

- Higher latency.
- Request timeouts.
- Pool-acquisition timeouts, if configured.
- Increased memory use from outstanding requests.
- Worse user experience across unrelated database-backed endpoints.

### Is the event loop blocked?

Not necessarily.

The event loop may still accept requests and execute other callbacks.

The bottleneck is often connection availability rather than a permanently blocked JavaScript thread.

```text
Request accepted
      ↓
Needs a DB connection
      ↓
Waits because the shared pool is busy
```

Non-database endpoints can still run, though memory pressure and heavy synchronous work may also affect them.

---

## 8. What if one insert fails?

Suppose one insert violates a constraint.

```js
await Promise.all(queryPromises);
```

rejects when it observes that failure.

The following success response is skipped:

```js
res.json({ ok: true });
```

### What happens to other queries?

`Promise.all()` does not automatically cancel them.

| Query state | What happens |
|---|---|
| Already successful | Its changes remain committed unless part of an uncommitted enclosing transaction |
| Currently executing | Continues unless separately cancelled or interrupted |
| Waiting in the pool | Can execute when a connection becomes available |
| Failed | Its promise rejects |

The remaining operations are not guaranteed to succeed. They simply are not cancelled by `Promise.all()`.

> [!important] Promise rejection is not database rollback
> `Promise.all()` is a JavaScript coordination mechanism.
> It does not provide database transaction atomicity.

### Partial import example

```text
Row 1 → committed
Row 2 → committed
Row 3 → failed
Row 4 → committed
Row 5 → committed
```

The request can fail while some rows remain inserted.

---

## 9. Does the original process crash, hang, or complete?

There is no unconditional answer.

### All inserts succeed

The request eventually completes, assuming no timeout or infrastructure failure.

### One insert fails

The route handler rejects.

- Express 5 forwards rejected async handlers to error handling.
- Express 4 requires explicit forwarding or an async wrapper.
- An unhandled rejection can terminate Node.js depending on runtime configuration and handlers.

### Database operations stall

Requests can appear stuck while waiting for connections or query completion.

### Memory becomes excessive

An out-of-memory failure is possible, but not inevitable.

### Safe route structure

```js
app.post('/import', async (req, res, next) => {
  try {
    // Async work here.

    res.json({ ok: true });
  } catch (error) {
    next(error);
  }
});
```

This explicitly forwards errors in both Express 4 and Express 5.

---

## 10. Three independent concepts

### Connection pool size

```js
max: 10
```

Maximum client connections in this pool.

### Application concurrency

```js
mapLimit(rows, 5, insertRow)
```

Maximum outstanding calls to `insertRow()` from this operation.

Some calls may be waiting for connections rather than actively executing SQL.

### Batch size

```js
const BATCH_SIZE = 5000;
```

Maximum rows handled by one batch statement.

### Example

```text
Pool size: 10
Import concurrency: 2
Batch size: 5,000
```

This could mean:

- Up to 2 outstanding batch operations from this import.
- Each operation handles up to 5,000 rows.
- The pool still has a maximum of 10 connections.

Batching and concurrency can be combined, but they solve different problems.

---

## 11. Fix 1: Bounded concurrency with `mapLimit()`

### Requirements

Create a function that:

1. Calls `fn` once for each item.
2. Never has more than `limit` calls in flight.
3. Returns results in the same order as the input.

### Failure policy

This implementation:

- Attempts every item.
- Records the first observed error.
- Waits for all workers to finish.
- Rejects after processing if any item failed.

This policy deliberately satisfies “call `fn` once for every item,” even when some items fail.

### Implementation

```js
async function mapLimit(items, limit, fn) {
  if (!Number.isInteger(limit) || limit < 1) {
    throw new RangeError('limit must be a positive integer');
  }

  const results = new Array(items.length);

  let nextIndex = 0;
  let hasError = false;
  let firstError;

  async function worker() {
    while (nextIndex < items.length) {
      const index = nextIndex++;

      try {
        results[index] = await fn(items[index], index);
      } catch (error) {
        if (!hasError) {
          hasError = true;
          firstError = error;
        }
      }
    }
  }

  const workerCount = Math.min(limit, items.length);

  const workers = Array.from(
    { length: workerCount },
    () => worker()
  );

  await Promise.all(workers);

  if (hasError) {
    throw firstError;
  }

  return results;
}
```

---

## 12. How `mapLimit()` works

### Shared index

```js
let nextIndex = 0;
```

Workers use this to find the next unclaimed item.

```js
const index = nextIndex++;
```

There is no `await` between checking the loop condition and claiming the index.

In a single JavaScript execution context, this synchronous section completes before another worker resumes, so workers do not claim the same index.

### One operation per worker

```js
results[index] = await fn(items[index], index);
```

The worker waits for the current call before claiming another item.

For a limit of 3:

```text
Worker A → at most 1 outstanding call
Worker B → at most 1 outstanding call
Worker C → at most 1 outstanding call

Total → at most 3 outstanding calls
```

### Why use `Promise.all()` here?

```js
await Promise.all(workers);
```

This is safe because the number of workers is bounded.

```text
Original:
Promise.all() waits for 50,000 immediately submitted queries.

Worker solution:
Promise.all() waits for a small fixed number of workers.
```

Each worker submits operations gradually.

### Why are results in input order?

```js
results[index] = result;
```

Results are assigned to their original positions.

Example:

```text
Input: A, B, C
Completion: C, A, B

Returned:
[resultOfA, resultOfB, resultOfC]
```

Using `results.push(result)` would record completion order instead.

### Empty input

For:

```js
mapLimit([], 5, fn)
```

there are zero workers, and the result is:

```js
[]
```

---

## 13. Worker pool versus fixed batches

A worker pool keeps using available capacity.

Example with limit 3:

```text
Start A, B, C

B finishes → start D
A finishes → start E
D finishes → start F
```

It does not wait for A, B, and C all to finish before starting D.

Fixed groups behave differently:

```text
Start A, B, C
Wait for all three

Start D, E, F
Wait for all three
```

If C is slow, fixed grouping leaves capacity unused after A and B finish.

---

## 14. Import handler using `mapLimit()`

```js
app.post('/import', async (req, res, next) => {
  try {
    const rows = await parseCsv(req.file);

    await mapLimit(rows, 5, row =>
      pool.query(
        'INSERT INTO leads(a, b) VALUES ($1, $2)',
        [row.a, row.b]
      )
    );

    res.json({
      ok: true,
      imported: rows.length
    });
  } catch (error) {
    next(error);
  }
});
```

### Connection usage

For one import:

```text
At most 5 outstanding insert calls.
At most 5 connections occupied by this import.
```

If other work already occupies the pool, some of those five calls may wait.

### Why use 5 if the pool has 10?

It limits this one import’s pressure on the shared pool.

However, this does not reserve five connections for other endpoints.

### Multiple imports

```text
1 import × limit 5 = up to 5 outstanding calls
2 imports × limit 5 = up to 10 outstanding calls
3 imports × limit 5 = up to 15 outstanding calls
```

The pool still has only 10 connections.

> [!warning] Per-request limit is not a global limit
> Several limited requests can collectively overload the shared pool.
> A global limiter or background-job system may still be needed.

### Transaction behavior

This handler still uses separate standalone inserts.

It does not make the import all-or-nothing.

---

## 15. Fix 2: Insert multiple rows per SQL statement

Instead of running:

```sql
INSERT INTO leads(a, b) VALUES ($1, $2);
```

once per row, send arrays:

```sql
INSERT INTO leads(a, b)
SELECT *
FROM unnest($1::text[], $2::text[])
ON CONFLICT DO NOTHING;
```

### Example parameters

```js
[
  ['A1', 'A2', 'A3'],
  ['B1', 'B2', 'B3']
]
```

The two arrays are expanded into paired rows:

```text
A1, B1
A2, B2
A3, B3
```

These rows are inserted using one SQL statement.

The code must construct both arrays from the same chunk so their positions correspond.

### Important

One statement still performs row-level work:

- Writing rows.
- Updating indexes.
- Checking constraints.
- Checking applicable conflicts.
- Running applicable triggers.

Batching reduces repeated statement and communication overhead. It does not eliminate the cost of storing rows.

---

## 16. Complete batched, transactional import handler

This version:

- Validates the file and rows.
- Acquires one client.
- Inserts sequential batches.
- Uses one transaction.
- Releases the client.
- Forwards errors.

```js
const BATCH_SIZE = 5000;
const MAX_ROWS = 50_000;

app.post(
  '/import',
  upload.single('file'),
  async (req, res, next) => {
    try {
      if (!req.file) {
        return res.status(400).json({
          error: 'CSV file is required'
        });
      }

      let rows;

      try {
        rows = await parseCsv(req.file);
      } catch {
        return res.status(400).json({
          error: 'Invalid CSV file'
        });
      }

      if (rows.length === 0 || rows.length > MAX_ROWS) {
        return res.status(400).json({
          error: `Row count must be 1-${MAX_ROWS}`
        });
      }

      const valid = [];
      const invalid = [];

      rows.forEach((row, index) => {
        const a = String(row.a ?? '').trim();
        const b = String(row.b ?? '').trim();

        if (
          !a ||
          !b ||
          a.length > 255 ||
          b.length > 255
        ) {
          invalid.push({
            record: index + 1,
            reason: 'missing or too long'
          });
        } else {
          valid.push({ a, b });
        }
      });

      if (invalid.length > 0) {
        return res.status(422).json({
          error: 'Invalid rows',
          invalid: invalid.slice(0, 100)
        });
      }

      const client = await pool.connect();

      let inserted = 0;

      try {
        await client.query('BEGIN');

        for (
          let index = 0;
          index < valid.length;
          index += BATCH_SIZE
        ) {
          const chunk = valid.slice(
            index,
            index + BATCH_SIZE
          );

          const result = await client.query(
            `
              INSERT INTO leads(a, b)
              SELECT *
              FROM unnest($1::text[], $2::text[])
              ON CONFLICT DO NOTHING
            `,
            [
              chunk.map(row => row.a),
              chunk.map(row => row.b)
            ]
          );

          inserted += result.rowCount;
        }

        await client.query('COMMIT');
      } catch (error) {
        await client.query('ROLLBACK').catch(() => {});
        throw error;
      } finally {
        client.release();
      }

      return res.json({
        ok: true,
        received: valid.length,
        inserted,
        skipped: valid.length - inserted
      });
    } catch (error) {
      next(error);
    }
  }
);
```

### Why is `pool.connect()` inside an outer `try`?

Connection acquisition can fail too.

The outer `try/catch` forwards that failure to `next(error)`, including in Express 4.

### Why report record number rather than physical CSV line?

CSV fields can contain embedded newlines.

Therefore, `index + 2` is not always the actual physical line number.

Use parser-provided line metadata if exact source lines are needed.

### Validation is not a guarantee that SQL cannot fail

Application validation cannot rule out:

- All database constraints.
- Concurrent changes.
- Trigger failures.
- Connection errors.
- Storage failures.

The transaction and error handling are still necessary.

---

## 17. How many connections does this batched handler use?

One connection per import request.

It is acquired once:

```js
const client = await pool.connect();
```

Every statement uses the same client:

```js
await client.query('BEGIN');
await client.query(/* batch 1 */);
await client.query(/* batch 2 */);
await client.query('COMMIT');
```

It is released once:

```js
client.release();
```

### For 50,000 rows

```text
50,000 rows ÷ 5,000 rows per batch = 10 batches
```

Execution:

```text
One connection
    |
    ├── BEGIN
    ├── INSERT batch 1
    ├── INSERT batch 2
    ├── ...
    ├── INSERT batch 10
    ├── COMMIT
    |
    └── Release
```

| Metric | Value |
|---|---:|
| Checked-out connections per import | 1 |
| Batch INSERT statements | 10 |
| Transaction-control statements on success | 2 |
| Total SQL statements on the success path | 12 |
| Concurrent query calls from this handler | At most 1 |

The loop uses `await`, so batches run sequentially.

### The connection remains occupied between batches

Finishing a batch does not return the client to the pool.

The handler holds it until `client.release()`.

---

## 18. Why all transaction statements need the same client

Incorrect pattern:

```js
await pool.query('BEGIN');
await pool.query('INSERT ...');
await pool.query('COMMIT');
```

Each `pool.query()` can acquire a different connection.

A PostgreSQL transaction belongs to a particular connection.

Correct pattern:

```js
const client = await pool.connect();

try {
  await client.query('BEGIN');
  await client.query('INSERT ...');
  await client.query('COMMIT');
} catch (error) {
  await client.query('ROLLBACK').catch(() => {});
  throw error;
} finally {
  client.release();
}
```

> [!important] Transaction rule
> Acquire one client, execute every transaction statement through it, and release it afterward.

---

## 19. What does `ON CONFLICT DO NOTHING` mean?

```sql
ON CONFLICT DO NOTHING
```

skips rows that encounter applicable uniqueness or exclusion conflicts.

It is not a general-purpose “ignore every database error” instruction.

Other problems can still fail a statement, such as:

- A `NOT NULL` violation.
- A `CHECK` violation.
- A foreign-key violation.
- An invalid value for the target type.
- A trigger raising an error.

### It requires an actual database conflict

Two business-duplicate rows are not automatically conflicts unless the schema has an applicable constraint or unique index.

Example:

```sql
CREATE UNIQUE INDEX leads_a_b_unique
ON leads(a, b);
```

Whether this is the correct uniqueness rule depends on the application.

### Inserted and skipped counts

```js
inserted += result.rowCount;
```

counts rows actually inserted.

```js
skipped = valid.length - inserted;
```

counts input rows not inserted by these statements.

### Transaction semantics

If a non-skipped SQL error occurs before commit:

```text
Roll back the transaction
        ↓
Earlier batches in this transaction are undone
```

If conflicts are skipped and all statements succeed:

```text
Commit successfully inserted rows
```

So “all-or-nothing” applies to the changes the transaction makes, not a promise that every input row must be inserted.

---

## 20. Is one connection slower than ten?

Not necessarily.

Connection count alone does not determine performance.

### Compare 50,000 input rows

| Approach | INSERT statements | Connections at once |
|---|---:|---:|
| Sequential one-row inserts | 50,000 | 1 |
| Unbounded one-row inserts | 50,000 | Up to 10 |
| `mapLimit(..., 5, ...)` one-row inserts | 50,000 | Up to 5 |
| Sequential batches of 5,000 | 10 | 1 |

The batched solution removes a large amount of repeated per-statement overhead.

It also uses one commit instead of committing each row separately.

### Why fewer statements help

Each statement involves work such as:

- Sending query data.
- Processing the statement.
- Executing it.
- Returning a result.
- Handling the result in the application.

The actual row writes still happen, but the surrounding overhead is amortized across many rows.

### Network example

Ignoring execution time for illustration:

```text
50,000 sequential single-row statements
→ 50,000 sequential query-response cycles

10 sequential batch statements
→ 10 sequential INSERT query-response cycles
```

The original concurrent version overlaps some of that waiting, so this is not an exact timing comparison. It illustrates why statement count matters.

### Why 10 connections may not give 10× speed

Connections share:

- CPU.
- Storage bandwidth.
- WAL bandwidth.
- Index structures.
- Locking resources.
- Database memory.

More concurrency can help up to a point, then provide diminishing returns or increase contention.

### Recommendation

For this 50,000-row endpoint, sequential batched inserts in one transaction are a sensible starting point.

Benchmark before increasing concurrency.

---

## 21. Why not parallelize batches on the same client?

Example:

```js
await Promise.all(
  chunks.map(chunk =>
    client.query(sql, makeParams(chunk))
  )
);
```

This does not create multiple PostgreSQL connections.

The client still represents one connection, and its queries are processed sequentially.

It can queue more work without providing the multi-connection parallelism someone might expect.

Use a straightforward awaited loop when sequential execution is intended.

### What if batches use separate clients?

Then they can run concurrently, but they do not participate in one ordinary shared transaction.

This changes failure behavior:

```text
Batch A commits
Batch B fails
Batch C commits
```

Do not trade away atomicity accidentally just to increase connection usage.

---

## 22. Multiple imports can still fill the pool

A single batched import holds one connection.

With a pool size of 10:

```text
1 simultaneous import  → up to 1 connection
5 simultaneous imports → up to 5 connections
10 simultaneous imports → up to 10 connections
```

An additional import may wait.

This is much better than each import immediately submitting 50,000 pool operations, but it does not eliminate global resource limits.

Possible controls:

- Limit simultaneous imports.
- Use a process-wide semaphore.
- Put imports into a background-job queue.
- Use a separately budgeted import pool.
- Apply suitable timeouts.
- Track progress and support retries.

A separate pool is not free capacity: its connections still count against the database server’s total connection budget.

---

## 23. Is batching constant-memory?

No.

The handler still retains:

- Parsed rows.
- Validated rows.
- Invalid-row metadata.
- Temporary chunk arrays.
- Parameter arrays.

It avoids thousands of outstanding query operations, but all input rows are still parsed into memory.

### Streaming alternative

For larger files:

```text
Read CSV incrementally
        ↓
Validate records incrementally
        ↓
Build one bounded batch
        ↓
Insert or stream batch
        ↓
Continue
```

This reduces memory growth.

However, streaming changes validation and error-reporting behavior. A transaction can still protect against partial committed changes if the design requires it.

Also, the row-count check happens after parsing in the shown code. Configure upload-size and parser limits too; a post-parse row limit does not protect the parser from an oversized file.

---

## 24. Faster bulk-loading option: COPY

For large loads, PostgreSQL's `COPY` mechanism is designed for efficient bulk transfer.

Conceptually:

```text
Application streams rows
        ↓
COPY protocol
        ↓
PostgreSQL bulk loading
```

It generally has less overhead than a series of INSERT statements.

### Conflict-handling limitation

`COPY` is not a direct replacement for:

```sql
INSERT ... ON CONFLICT DO NOTHING
```

A common design is:

```text
COPY into staging table
        ↓
INSERT INTO final_table
SELECT ...
FROM staging_table
ON CONFLICT DO NOTHING
        ↓
Commit
```

This adds complexity, so choose it when measurements or scale justify it.

For a moderate import endpoint, batched INSERTs may already be sufficient.

---

## 25. Measuring performance

Do not assume a particular execution time without testing.

Separate:

1. CSV parsing.
2. Validation.
3. Pool-acquisition wait.
4. Batch execution.
5. Commit.
6. Total request duration.

### Example instrumentation

```js
const acquireStarted = performance.now();
const client = await pool.connect();

const acquireMs = performance.now() - acquireStarted;
let inserted = 0;

try {
  await client.query('BEGIN');

  for (
    let index = 0;
    index < valid.length;
    index += BATCH_SIZE
  ) {
    const chunk = valid.slice(
      index,
      index + BATCH_SIZE
    );

    const batchStarted = performance.now();

    const result = await client.query(
      `
        INSERT INTO leads(a, b)
        SELECT *
        FROM unnest($1::text[], $2::text[])
        ON CONFLICT DO NOTHING
      `,
      [
        chunk.map(row => row.a),
        chunk.map(row => row.b)
      ]
    );

    inserted += result.rowCount;

    console.log({
      batch: Math.floor(index / BATCH_SIZE) + 1,
      attempted: chunk.length,
      inserted: result.rowCount,
      durationMs: performance.now() - batchStarted
    });
  }

  const commitStarted = performance.now();

  await client.query('COMMIT');

  console.log({
    acquireMs,
    commitMs: performance.now() - commitStarted,
    inserted
  });
} catch (error) {
  await client.query('ROLLBACK').catch(() => {});
  throw error;
} finally {
  client.release();
}
```

The caller must handle rejection, including failures from `pool.connect()`.

### Throughput

```text
Attempted rows per second =
attempted rows / elapsed seconds
```

```text
Inserted rows per second =
actually inserted rows / elapsed seconds
```

These differ if many rows are skipped.

### Benchmark fairly

Keep these conditions comparable:

- Same input.
- Same row sizes.
- Same indexes and constraints.
- Same duplicate rate.
- Same transaction behavior.
- Same database hardware and network.
- Similar background load.

Do not compare a mostly-skipped duplicate import with an all-new-row import as if they perform identical work.

---

## 26. Useful pool metrics

```js
console.log({
  total: pool.totalCount,
  idle: pool.idleCount,
  waiting: pool.waitingCount
});
```

### Meaning

| Metric | Meaning |
|---|---|
| `totalCount` | Clients currently in the pool |
| `idleCount` | Clients currently idle |
| `waitingCount` | Requests waiting to acquire a client |

A checked-out client is not necessarily executing a query at every instant.

For example, a transaction can hold a client while application code runs between statements.

Therefore:

```text
totalCount - idleCount
```

indicates checked-out clients, not an exact count of actively executing SQL statements.

---

## 27. Important limitations and correctness details

### Validation does not replace constraints

Keep database constraints as the final enforcement layer.

### Transaction does not mean zero operational risk

A transaction can hold locks and consume resources until it finishes.

Keep it no longer than necessary.

### Commit can have an uncertain outcome after a connection failure

If the connection fails around `COMMIT`, the application may not know whether PostgreSQL committed successfully.

A retry strategy should use an appropriate idempotency or uniqueness design instead of assuming every reported error means nothing committed.

### Client disconnection does not automatically cancel the import

If the HTTP caller times out or disconnects, database work may continue unless the application implements explicit cancellation behavior.

### Database import failure is not always an HTTP 500

Known validation or business errors may deserve specific responses.

Unexpected database or infrastructure failures should go through appropriate error handling without exposing sensitive details.

---

## 28. Common misconceptions

### “Pool size 10 means only 10 queries can be submitted.”

False.

More queries can be submitted and wait in the pool queue.

### “Promise.all starts the queries.”

Not precisely.

In this example, `map()` invokes `pool.query()` before `Promise.all()` receives the promises.

### “Promise.all limits concurrency.”

False.

It coordinates completion, not how many operations start.

### “One failed promise cancels every other query.”

False.

Cancellation must be implemented separately.

### “A rejected Promise.all rolls back successful inserts.”

False.

Only a database transaction provides database rollback behavior.

### “Using one connection means inserting one row at a time.”

False.

One statement on one connection can insert thousands of rows.

### “Batch size 5,000 means 5,000 connections.”

False.

It means up to 5,000 input rows per batch.

### “A concurrency limit of 5 reserves the other 5 connections.”

False.

Other imports and endpoints can use the remaining pool capacity.

### “Ten connections must be ten times faster than one.”

False.

Statement count, batching, transaction overhead, and shared database resources matter.

### “ON CONFLICT DO NOTHING ignores all errors.”

False.

It handles applicable conflicts, not arbitrary SQL errors.

---

## 29. Quick comparison

| Property | Unbounded single-row inserts | Limited single-row inserts | Sequential transactional batches |
|---|---|---|---|
| Example | `Promise.all(rows.map(...))` | `mapLimit(rows, 5, ...)` | Awaited batch loop |
| INSERT statements for 50k rows | 50,000 | 50,000 | 10 at batch size 5,000 |
| Outstanding calls per import | Potentially 50,000 | At most 5 | At most 1 |
| Connections per import | Up to pool capacity | Up to 5 | 1 |
| Huge per-import pool queue | Possible | Avoided | Avoided |
| Atomic by default | No | No | Yes, for transaction changes |
| Duplicate handling | Depends on SQL | Depends on SQL | `ON CONFLICT` in shown code |
| Input fully in memory | Yes, in shown code | Yes, in shown code | Yes, in shown code |

---

## 30. Interview-ready answers

### What is wrong with the original endpoint?

> It submits up to 50,000 individual query operations immediately. The pool limits connections to 10, but the remaining operations wait in application memory. This can delay other endpoints sharing the pool. Promise.all rejects on a failure but does not cancel outstanding queries or roll back successful inserts.

### What is the difference between pool size and concurrency?

> Pool size limits database connections. Application concurrency limits outstanding operations. A pool of 10 can still have thousands of operations waiting for connections.

### What does mapLimit solve?

> It bounds the number of outstanding calls by using a fixed number of workers. Each worker waits for its current operation before taking another item, preventing a huge per-request query queue.

### How many connections does the transactional batch solution use?

> One per import request. It checks out a client once, executes BEGIN, all batch inserts, and COMMIT on that client, then releases it.

### Why can one connection be efficient?

> Because it inserts thousands of rows per statement rather than issuing one statement per row. This reduces repeated query overhead, and one transaction avoids per-row commits.

### Why not use pool.query for transaction statements?

> A transaction belongs to one PostgreSQL connection. Separate pool.query calls can use different clients, so every transaction statement must use the same checked-out client.

### Which approach would you choose here?

> For this 50,000-row import, I would start with validated, sequential batch inserts on one client inside a transaction. I would also limit simultaneous imports and measure performance. For much larger loads, I would evaluate streaming COPY with a staging table if conflict handling is required.