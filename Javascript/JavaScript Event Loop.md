# JavaScript Event Loop, Async/Await & Non-Blocking Code

## Core Idea

- JavaScript is **single-threaded**: it runs one task at a time.
- **Non-blocking** means the main thread is not stuck waiting for slow operations like DB queries, API calls, timers, or file reads.
- Async operations are handled by the runtime (browser/Node.js). When they finish, the **event loop** schedules the continuation to run.
- `await` pauses **only the current async function**, not the entire Node.js server.

## Event Loop

The event loop lets JavaScript perform non-blocking I/O even though it has one main JavaScript thread.

### Main parts

- **Call Stack**: Where synchronous JavaScript code runs.
- **Web APIs / Node APIs**: Handle timers, network requests, DB drivers, file I/O, etc.
- **Macrotask Queue / Task Queue**: Callbacks from `setTimeout`, `setInterval`, I/O events, DOM events.
- **Microtask Queue**: Callbacks from resolved Promises, `.then()`, `.catch()`, `.finally()`, and `queueMicrotask()`.
- **Event Loop**: Checks whether the call stack is empty, then moves queued work back to the stack.

### Event loop steps

1. Run all current synchronous code until the call stack is empty.
2. Drain the **entire microtask queue**.
3. In browsers, render/paint if needed.
4. Take **one macrotask** from the task queue and run it.
5. After that macrotask finishes, drain the microtask queue again.
6. Repeat.

So the order is:

```text
Synchronous code
→ Microtasks: Promise.then, queueMicrotask
→ Rendering (browser only)
→ One macrotask: setTimeout, I/O callback, DOM event
→ Microtasks again
→ Next macrotask
```

Microtasks have higher priority than macrotasks. The event loop drains all microtasks before starting the next macrotask. [17][18][22]

## Microtasks vs Macrotasks

| Type | Examples | Priority | How many run per event loop turn? |
|---|---|---|---|
| Microtask | `Promise.then`, `Promise.catch`, `Promise.finally`, `queueMicrotask` | Higher | All of them, until the queue is empty |
| Macrotask | `setTimeout`, `setInterval`, I/O callbacks, DOM events | Lower | Usually one, then microtasks run again |

### Important rule

Even `setTimeout(fn, 0)` does **not** run before a resolved Promise callback.

```js
setTimeout(() => {
  console.log("timeout");
}, 0);

Promise.resolve().then(() => {
  console.log("promise");
});

console.log("sync");

// Output:
// sync
// promise
// timeout
```

Why:

1. `console.log("sync")` runs immediately.
2. The Promise callback goes into the **microtask queue**.
3. The `setTimeout` callback goes into the **macrotask queue**.
4. After synchronous code finishes, all microtasks run first.
5. Then the timeout macrotask runs. [22][26][28]

## Node.js Extra: `process.nextTick`

Node.js has one more special queue:

```text
Current code
→ process.nextTick queue
→ Microtask queue: Promises
→ Next event loop phase / macrotask
```

- `process.nextTick()` has the highest priority.
- It runs after the current operation but **before Promise microtasks** in normal CommonJS code.
- Promise callbacks run after `process.nextTick()` callbacks. [24][25]

Example:

```js
Promise.resolve().then(() => console.log("promise"));

process.nextTick(() => console.log("nextTick"));

setTimeout(() => console.log("timeout"), 0);

console.log("sync");

// Output:
// sync
// nextTick
// promise
// timeout
```

## Why `await` Is Needed

If later code depends on an async result, we must wait for it.

```js
const rows = getDataFromDB(); // returns a Promise, not data
for (const row of rows) {}    // wrong: loop runs on a Promise
```

Correct:

```js
async function handleData() {
  const rows = await getDataFromDB(); // wait for actual data

  for (const row of rows) {
    process(row);
  }
}
```

- `await` waits for the Promise to resolve.
- The function resumes only after the result is available.
- Other requests/events can still run during the wait.

## GET Request Example

```js
app.get("/users", async (req, res) => {
  const rows = await db.query("SELECT * FROM users");
  res.json(rows);
});
```

- This request waits for its own DB result.
- It does **not** block other GET requests.
- Node can handle many requests concurrently through the event loop.

## Common Misconception

Missing `await` does **not** block other requests.

```js
app.get("/users", (req, res) => {
  const rows = db.query("SELECT * FROM users"); // Promise
  res.json(rows); // wrong response
});
```

- This is a **correctness bug**, not a blocking issue.
- The query still runs asynchronously.
- But the code does not wait for the result, so it sends a Promise instead of data.

## What Actually Blocks Node

Blocking happens with synchronous or CPU-heavy work:

```js
const data = fs.readFileSync("big.txt"); // blocks event loop

while (Date.now() < end) {} // busy loop blocks event loop
```

While this runs, Node cannot process any other request.

| Style | Blocks other requests? | Correct result? |
|---|---|---|
| `await db.query()` | No | Yes |
| `db.query()` without `await` | No | No, returns Promise |
| `readFileSync` / heavy CPU loop | Yes | Yes, but server stalls |

## Sequential vs Parallel Async Work

Use sequential `await` when the next call depends on the previous result:

```js
const user = await getUser();
const orders = await getOrders(user.id);
```

Use `Promise.all` for independent operations:

```js
const [users, products] = await Promise.all([
  getUsers(),
  getProducts(),
]);
```

- `Promise.all` starts both tasks together.
- It waits for all of them to finish.
- Usually faster than awaiting them one by one.

## Error Handling

```js
try {
  const rows = await db.query("SELECT * FROM users");
} catch (err) {
  console.error("DB query failed:", err);
}
```

- If a Promise rejects, `await` throws the error.
- `try/catch` works like normal synchronous error handling.

## CPU-Heavy Work

Even `async` functions can block if the actual work is CPU-heavy.

Examples:

- Image processing
- Large JSON parsing
- Complex calculations
- Big loops

Solutions:

- Worker Threads
- Job queues: RabbitMQ, BullMQ
- Separate microservice

## Interview Answer

> JavaScript is non-blocking because async operations like DB queries do not hold the main thread while waiting. If later code depends on the result, we use `await` inside an `async` function. `await` pauses only that function until the Promise resolves, then resumes it with the actual value. Meanwhile, the event loop continues handling other requests. The event loop first finishes synchronous code, then drains all microtasks such as Promise callbacks, then runs one macrotask such as a `setTimeout` or I/O callback, then drains microtasks again. Real blocking comes from synchronous, CPU-heavy, or `*Sync` operations because they occupy the single main thread.

## Key Takeaways

- `await` = wait for **this function's** result.
- Non-blocking = other requests can still run.
- Missing `await` = wrong data, not blocking.
- Microtasks run before macrotasks.
- `Promise.then` runs before `setTimeout(fn, 0)`.
- In Node, `process.nextTick` runs before Promise microtasks.
- `readFileSync` / CPU loops = real blocking.
- Use `Promise.all` for independent async calls.
- Use queues/workers for CPU-heavy tasks.