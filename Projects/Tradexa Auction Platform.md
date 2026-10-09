# Tradexa — Placement Interview Cheat Sheet

> [!abstract] Resume lines covered
> - High-throughput proxy bidding in Go/Gin: 1,600 TPS, 0.43 ms latency.
> - PostgreSQL row-level pessimistic locking: ACID, 590+ concurrent bids/sec, zero data races.
> - QStash serverless webhooks: auction-expiry scheduling and decoupled event timing.
> - SSE live bid feeds, Cashfree escrow lifecycle, and webhook signature verification.

---

## How to use this sheet

Every question is tagged with the difficulty and resume claim it defends. Read the “What they’re really testing” note first; that is what separates a good answer from a great one.

> [!warning] Verify every claim
> Treat metrics, file paths, implementation details, and provider behavior below as interview-prep notes to verify against your actual code, benchmark data, and current configuration. Do not present suggested answers as facts unless you implemented or measured them.

---

## 1. Proxy Bidding System

### Q1. What is proxy bidding? How did you implement it?

**What they’re really testing:** Do you understand the domain and the technical design?

**Answer framework:**

- **What:** Proxy bidding lets a user set a secret maximum they are willing to pay. The system bids automatically on their behalf up to that maximum, showing only the minimum amount needed to beat competing bids.
- **How it works in Tradexa:**
  1. The user submits `{listing_id, amount}` as their secret maximum bid.
  2. The system starts a database transaction and locks the listing row with `SELECT FOR UPDATE`.
  3. It checks whether a `proxy_bid` record exists for the listing.
  4. **No existing proxy:** Create a proxy with maximum `X` and record a public bid at the starting price.
  5. **Same bidder raises their maximum:** Update the maximum; the public price remains unchanged unless the rules require a price change.
  6. **New bidder competes with the existing proxy:**
     - `newMax > oldMax`: the new bidder wins; public price becomes `oldMax + increment`.
     - `newMax < oldMax`: the old proxy holder remains ahead; public price becomes `newMax + increment`.
     - `newMax == oldMax`: the existing proxy holder wins the tie under a first-come rule.

**Key code:** `BidHandler()` in `handlers/listing.go`; the notes say the logic is handled in one database transaction.

### Q2. How did you handle race conditions in bidding?

**What they’re really testing:** Do you understand concurrency control, or did you just happen to get a working result?

- PostgreSQL row-level pessimistic locking is used with `SELECT FOR UPDATE` (GORM: `clause.Locking{Strength: "UPDATE"}`).
- A transaction that locks a listing row prevents other transactions from concurrently modifying that same row until the lock is released.
- This serializes the read-modify-write section for bids on the same listing, assuming every write path follows the same locking protocol.
- The resume claim is **zero data races** at **590+ concurrent bids/sec**; be ready to explain how you tested this and what “data race” means in this context.

**Why not optimistic locking?** Optimistic locking uses a version column and retries after conflicts. Under contention on one listing, retries may add load and latency. Whether pessimistic locking is better depends on contention, transaction length, and workload; avoid claiming optimistic locking necessarily causes a retry storm without benchmark evidence.

**Why not an in-memory mutex?** A process-local `sync.Mutex` does not coordinate separate application instances. Database locking works across instances that use the same database.

### Q3. Walk me through what happens when 590 bids arrive simultaneously for one listing

1. Requests reach the bid endpoint and begin transactions.
2. Transactions attempt to acquire a row lock on the same listing.
3. PostgreSQL allows one transaction at a time to hold the conflicting row lock; others wait or fail according to configured timeouts and cancellation behavior.
4. The lock holder reads the current proxy state, resolves the bid, writes the necessary records, and commits.
5. The lock is released, allowing another waiting transaction to proceed.
6. This prevents conflicting read-modify-write operations when all relevant code paths lock consistently.

**Trade-off:** High contention can increase wait time, queueing, and timeouts. Be specific about the load-test setup and measured tail latency. A 0.43 ms average should not be described as the latency of 590 serialized requests unless the benchmark actually measured that scenario.

### Q4. What is ACID, and how does your system ensure it?

- **Atomicity:** The proxy update and related bid writes should commit together or roll back together. If external effects such as SSE broadcasts are sent outside the database transaction, they are not automatically part of that atomic commit.
- **Consistency:** Database constraints and application validation preserve invariants, such as valid foreign keys and permitted bid transitions.
- **Isolation:** `SELECT FOR UPDATE` serializes conflicting work on the locked listing row. It does not, by itself, make every transaction globally serializable; explain the configured isolation level and locking scope accurately.
- **Durability:** PostgreSQL's write-ahead logging and configured durability settings protect committed transactions against failures, subject to the database's storage and replication configuration.

### Q5. What is the bid increment? Why $5? What happens on a tie?

- The notes say the increment is `$5.00`, hardcoded in `BidHandler` around line 421; verify the current code and currency handling.
- A fixed increment can reduce tiny bid increases, but the product rationale should match the actual design decision.
- **Tie rule:** When `newMax == oldMax`, the existing proxy holder wins, giving priority to the earlier bid.
- **Possible improvement:** Make increments configurable per listing or define a tiered increment schedule for different price ranges.

---

## 2. Performance: 1,600 TPS and 0.43 ms Latency

### Q6. How did you achieve 1,600 transactions per second?

**What they’re really testing:** Do you understand the actual performance factors, and did you benchmark the claim?

Be honest and specific about what the benchmark measured:

- Go's HTTP server and Gin can handle concurrent requests using goroutines, but throughput still depends on CPU, database capacity, connection-pool settings, network latency, and lock contention.
- A reused PostgreSQL connection pool avoids establishing a new connection for every request.
- Keep the critical path short: parse the request, validate it, perform the transaction, and return a response.
- If SSE broadcasting occurs after commit and does not block the request, explain how that behavior is implemented and how subscriber backpressure is handled.
- If notifications run asynchronously, explain how failures are handled; a fire-and-forget goroutine can lose work if the process exits.

**On 0.43 ms latency:** State the actual statistic (mean, p50, or another percentile), endpoint, load profile, environment, sample duration, and whether database/network time was included. Do not call a value “average” if it is actually p50.

### Q7. How would you scale this system to 100 times more traffic?

1. Scale stateless API instances behind a load balancer.
2. Measure the database bottleneck. Row locks serialize bids for the same listing; use partitioning or other approaches only after measuring the workload.
3. Send read-heavy listing queries to replicas only when replication lag is acceptable for the query's consistency needs.
4. Cache listing data where appropriate and invalidate or update it reliably after bids.
5. Shard by `listing_id` only if scale requires it; listings with hot bidding activity remain hot keys even after sharding.
6. Keep bid state updates durable and ordered. Moving proxy resolution to Redis and writing to the database asynchronously changes the consistency and recovery design; it is not a drop-in optimization.

### Q8. What is a goroutine? How is it different from a thread?

- A goroutine is a lightweight execution unit scheduled by the Go runtime.
- Go uses an M:N scheduling model, multiplexing goroutines over operating-system threads.
- Goroutine stacks start small and can grow; exact memory use depends on runtime version and workload, so avoid presenting one fixed stack size as universal.
- A goroutine is cheaper to create than an OS thread in many workloads, but large numbers of goroutines still consume resources and need lifecycle control.
- In Tradexa, HTTP handlers run concurrently, and `go sendOutbidNotifications()` launches asynchronous work if that call exists in the code.

---

## 3. PostgreSQL and Database Design

### Q9. Why pessimistic locking instead of optimistic locking?

- **Optimistic locking:** Add a version column; update with `WHERE id = X AND version = Y`. If no row is updated, another transaction won and the caller must retry or return a conflict.
- **Pessimistic locking:** Acquire a database lock before reading and changing the contested state.
- Pessimistic locking can suit highly contended updates to a single listing, but it makes requests wait and can reduce throughput for that hot listing.
- The choice should be defended with observed contention, transaction duration, failure rates, and latency—not a blanket claim that one strategy is always better.

### Q10. Explain your schema. Why is `ProxyBid` separate from `Bid`?

- **`bids` table:** Public bid history or ledger—the records shown in the auction UI.
- **`proxy_bids` table:** Secret maximum amounts used to calculate the winning price.
- Keeping the two concepts separate supports different access controls and query paths. It does not automatically guarantee confidentiality: database permissions, API serialization, logs, backups, and administrative access also matter.
- A unique constraint on `proxy_bids(listing_id)` may be appropriate if the model guarantees one active proxy record per listing; verify the actual schema and business rules.

### Q11. What indexes would you add to `bids`?

```sql
-- Useful for listing bid history ordered by amount.
CREATE INDEX idx_bids_listing_amount
    ON bids (listing_id, amount DESC);

-- Useful for filtering a bidder's bids.
CREATE INDEX idx_bids_bidder
    ON bids (bidder_id);

-- Useful for looking up a proxy by listing.
CREATE INDEX idx_proxy_bids_listing
    ON proxy_bids (listing_id);
```

Choose indexes based on real query patterns and `EXPLAIN (ANALYZE, BUFFERS)`. An index can speed up reads but adds storage and write overhead. The composite index may need additional columns or a different order depending on the exact query and tie-breaking rules.

### Q12. What is GORM? What are the trade-offs of an ORM?

- GORM is a Go ORM that maps Go structs to database tables and provides query, association, and migration helpers.
- **Benefits:** Faster development, convenient CRUD and associations, and support for common query patterns.
- **Risks:** N+1 queries, generated SQL that may not match performance needs, implicit behavior, and migrations that need review.
- In Tradexa, the notes mention `Preload("Seller").Preload("Images")` and a batch query with `GROUP BY` to avoid an N+1-style highest-bid lookup; verify the implementation.
- `clause.Locking` is GORM's way to express a locking clause, but the SQL and semantics are determined by the database.

---

## 4. Real-Time Updates: SSE and WebSocket

### Q13. Why use SSE for bid updates instead of WebSocket?

**What they’re really testing:** Do you understand the protocol trade-offs and can you justify the choice?

- **SSE:** Server-to-client event stream over HTTP; text-based and supported by the browser `EventSource` API, including automatic reconnect behavior.
- **WebSocket:** Full-duplex connection that supports communication in both directions after an HTTP upgrade.
- For one-way live bid updates, SSE can be simpler. Clients still submit bids through ordinary HTTP requests.
- For chat or targeted bidirectional interaction, WebSocket may fit better. Notifications can also be implemented through SSE or other mechanisms; explain why your actual design uses WebSocket.

**Architecture in the notes:**

- `GET /api/stream/:id` opens an SSE stream for a listing; `StreamHub` broadcasts a payload such as `{type: "new_bid", amount, listing_id}`.
- `GET /api/ws/notifications` opens a per-user WebSocket; `HubManager.NotifyUser(userID, ...)` sends targeted messages.

### Q14. How does `StreamHub` work, and how do you avoid race conditions?

- The notes describe an in-memory map from `listingID` to subscribers.
- Protect concurrent map and subscription changes with a mutex or another synchronization strategy.
- Avoid holding a global lock while performing potentially blocking network writes; use buffered channels, bounded queues, or another backpressure strategy.
- Remove disconnected subscribers and close resources safely to avoid leaks and send-on-closed-channel panics.
- **Scale limitation:** In-memory subscribers exist only in one process. With multiple instances, a bid handled by one instance will not automatically reach clients connected to another. Redis Pub/Sub or a message broker can distribute events, with delivery and recovery trade-offs.

### Q15. What happens when a user disconnects from SSE?

- The handler can observe request-context cancellation or a write error.
- It should remove the subscriber and release related resources.
- Browser `EventSource` typically reconnects automatically, subject to browser behavior and server retry configuration.
- A client may miss events while disconnected. Send an initial snapshot or support event IDs and replay if the product requires stronger catch-up guarantees.

---

## 5. Cashfree Payments and Escrow

### Q16. Explain the payment or escrow flow

1. The auction closes and an order is created with a pending-payment status.
2. The winner requests an order or payment session from the backend.
3. The backend calls Cashfree and returns the required session information to the client.
4. The buyer completes payment through the provider flow.
5. Cashfree sends a webhook to the backend.
6. The backend verifies the webhook signature and validates the event before updating order state.
7. On verified payment success, the order moves to a paid or escrow-related state according to the provider's actual product and account configuration.
8. The seller is notified to ship; shipment and delivery transitions update the order.
9. Any eventual release or refund flow must follow the actual payment product, settlement rules, and implemented business logic.

> [!warning] Be precise about “escrow”
> Do not say funds are legally held in escrow or that Cashfree holds funds in a particular way unless the product integration, account configuration, and applicable terms support that statement. Describe the actual payment and settlement flow.

### Q17. How did you verify Cashfree webhook signatures? Why?

- The notes say the webhook uses HMAC-SHA256 and a shared secret.
- Verification should use the provider's documented algorithm, required headers, timestamp or raw-body rules, and constant-time comparison where applicable. Do not assume the signed bytes are simply the raw body; check the current Cashfree documentation and your implementation.
- Verification prevents an unauthenticated caller from forging a payment-success event.
- The webhook may be exempt from user JWT authentication because the provider cannot present the user's JWT. It still needs provider-signature verification and replay/idempotency protections.

### Q18. Why is `/payment/webhook` public but `/internal/auction-close` is not?

- A payment webhook is called by the payment provider, not a logged-in user, so it generally cannot use the user's JWT. Authenticate it with the provider's webhook-verification mechanism.
- The notes say QStash calls `/internal/auction-close` and the backend verifies its signature with `QStashReceiver`.
- “Public route” means reachable without user JWT; it should not mean unauthenticated. Validate signatures, timestamps, payloads, and idempotency for server-to-server routes.

---

## 6. QStash and Event Scheduling

### Q19. What is QStash? Why use it instead of cron or `time.AfterFunc`?

**What they’re really testing:** Do you understand durable scheduling versus in-process timers?

- `time.AfterFunc` is an in-process timer. A restart or crash can lose the scheduled callback unless the schedule is reconstructed.
- A cron job can periodically query for expired auctions, but its precision and database load depend on the polling interval and query design.
- QStash is an external messaging/scheduling service that can deliver a scheduled HTTP request independently of the application process, subject to its delivery, retry, and retention semantics.
- The notes say Tradexa falls back to `time.AfterFunc` if `QSTASH_TOKEN` is missing. If true, describe this as a local-development fallback, not a production-equivalent durable scheduler.
- Verify the configured retry count and delivery guarantees in the current provider configuration; do not assume a fixed number of retries.

### Q20. What does `NotBefore` do in the QStash publish call?

```go
NotBefore: fmt.Sprintf("%d", at.Unix()),
```

The intended meaning is to delay delivery until a specified time. Confirm the exact field format, units, and limits against the version of the QStash SDK and API actually used.

When an auction is created, the notes say `ScheduleAuctionClose(listing.ID, *listing.AuctionEndsAt)` schedules the close event for the end time.

---

## 7. Authentication and Security

### Q21. How does JWT authentication work?

- On login, the server creates a signed token with claims such as user ID, email, and expiry.
- The notes say the token uses HS256 and is stored in an HTTP-only cookie; confirm the signing algorithm, cookie flags, and whether cookies are also used for other auth flows.
- Protected middleware extracts the token, validates its signature and expiry, and places the authenticated user ID in request context.
- An HTTP-only cookie prevents JavaScript from reading the cookie, but it does not alone make an application “XSS safe.” Use appropriate `Secure`, `SameSite`, CSRF, and XSS defenses.
- The notes describe a Redis-backed token blacklist using a `jti` and remaining TTL. Verify the actual logout and revocation behavior.

### Q22. Authentication vs authorization: where does each happen?

- **Authentication:** “Who are you?”—for example, JWT validation in `AuthRequired()` middleware.
- **Authorization:** “May you perform this action?”—for example, checking that the authenticated seller owns the listing before updating it.
- The notes say `UpdateListing` checks ownership and `BidHandler` prevents sellers bidding on their own listing. Confirm exact status codes and validation paths in the code.

### Q23. How does the rate limiter work?

- The notes describe Redis `INCR` plus `EXPIRE`, keyed by a prefix and IP address.
- On a request, increment the counter; set expiry for a new window; reject once the configured limit is exceeded.
- A plain counter with a fixed expiry is a fixed-window counter, not a true sliding-window counter. Clarify the algorithm accurately.
- Redis allows instances to share counters, but atomicity of increment-plus-expiry and handling of Redis failure should be considered.
- The notes list login at 10 attempts per 15 minutes per IP and OTP send at 5 per hour per IP; verify actual values and any account-based limits.

### Q24. What is the difference between `AuthRequired()` and `OptionalAuth()`?

- `AuthRequired()` rejects requests without a valid authenticated identity; it is used for actions that require login.
- `OptionalAuth()` attempts authentication but allows an anonymous request to continue; a valid user identity may enable personalized response data.
- The notes use `GET /listings/:id` as an example where guests can view the listing while authenticated users may see their own proxy-bid information. Ensure secret data is never exposed for another user.

---

## 8. System Design and Architecture

### Q25. Explain Tradexa's overall architecture

Use the mnemonic **C-B-D-E**:

- **Client:** React/Vite SPA; REST/JSON for API requests, WebSocket for chat or notifications, and SSE for bid updates.
- **Backend:** Go/Gin API, WebSocket manager, SSE hub, and background work.
- **Data:** PostgreSQL (noted as Supabase) for persistent data; Redis (noted as Upstash) for rate limiting and related ephemeral state.
- **External services:** Cashfree for payments, QStash for scheduling, Cloudinary for images, Gmail SMTP for email, and Google OAuth for social login.
- **Deployment:** The notes say Render.com hosts the static frontend and Go web service. Confirm current deployment and provider configuration.

### Q26. Why Go/Gin instead of Node.js or Python?

- Go has goroutines and runtime scheduling that can support many concurrent I/O-bound requests efficiently; Node.js also handles I/O concurrency efficiently, so avoid oversimplifying it as unable to serve concurrent requests.
- Go can compile to a single deployable binary, though the final container still depends on build and runtime choices.
- Static typing can catch certain errors at compile time, but it does not guarantee correct financial logic.
- The standard library provides HTTP, synchronization, and cryptographic packages; external dependencies still need review.
- Explain why Go/Gin matched this project's team, performance, deployment, and learning constraints.

### Q27. What would you do differently if rebuilding Tradexa?

1. Use a transactional outbox or durable queue for emails and other important post-commit work rather than fire-and-forget goroutines.
2. Consider event sourcing only if immutable auditability, replay, or dispute investigation justifies the added complexity.
3. Use Redis Pub/Sub or a broker to fan out live events across instances, while planning for reconnect and missed-event recovery.
4. Add structured logs, request/correlation IDs, metrics, tracing, and alerting.
5. Revisit the bidding write path with measured contention data before replacing database locking with distributed locks.

---

## 9. Deployment and Infrastructure

### Q28. How is Tradexa deployed? What is Render.com?

- Render is a cloud platform that can host web services and static sites.
- The notes describe two services in `render.yaml`: a static frontend and a long-running Go backend.
- The notes say Supabase hosts PostgreSQL, Upstash hosts Redis, and Cloudinary handles image storage and delivery.
- Confirm regions, connection pooling, free-tier behavior, service sleep policies, and actual production configuration before quoting them.

### Q29. What is the keep-alive endpoint for?

- The notes describe `GET /api/keep-alive` as checking the Go service and running `SELECT 1` against the database.
- If a hosting plan sleeps inactive services, an external monitor may periodically request a health endpoint, subject to the platform's current policies.
- Return health information carefully: expose only what is safe publicly, and avoid leaking detailed infrastructure status.
- The example response in the notes is `{status: "Alive and kicking", db: "Connected"}`; verify actual response content.

---

## 10. Behavioral and Resume Defense

### Q30. The resume claims 0.43 ms average response latency. How did you measure this?

**Honest answer; do not bluff:**

- Name the tool you actually used (`autocannon`, `k6`, `wrk`, or another tool).
- State the endpoint, request mix, concurrency, duration, hardware/hosting environment, database location, and whether warm-up was included.
- Clarify whether 0.43 ms is mean, p50, or another statistic, and whether it measures server processing time or end-to-end latency.
- Report p95/p99 and error rate too, especially under contention.
- The draft answer proposes “p50 at 1,600 req/sec,” but use that only if your benchmark output confirms it.

### Q31. Did you write tests for the bidding logic?

**Honest answer and pivot:**

- State which unit, integration, and load tests exist today.
- The proxy-bidding state machine has branches for no proxy, same-bidder increase, new bidder wins, existing proxy remains ahead, and tie.
- If branch tests are missing, say so and explain how you would test them using a transaction-aware test database or an appropriate repository abstraction.
- `testify/assert` and `sqlmock` are possible tools, but mocks may not reproduce PostgreSQL row-locking semantics. Use integration tests with PostgreSQL for locking and concurrency behavior.

### Q32. What if the auction-expiry webhook fires after the auction is already sold?

- The handler should check the current auction state and return safely if the auction is already closed or sold.
- The operation should be idempotent: repeated deliveries should not create duplicate orders, notifications, or state transitions.
- The notes say `/internal/auction-close` checks `listing.IsSold` and returns early. Verify whether the state check and update are protected against concurrent close events.
- Webhook delivery is often at-least-once, so verify provider retry semantics and design the handler for duplicates.

### Q33. How does bidder-name masking work, and why?

- The notes describe masking a bidder's name for privacy while preserving a stable pseudonymous tag across bids.
- The example is `A***n-3f2a`, with a short deterministic hash derived from a secret salt and user ID.
- Do not use an unsalted hash of predictable user IDs. Protect the secret, use a keyed construction such as HMAC if appropriate, and consider whether the stable tag enables unwanted tracking or linkability.
- Explain what identity information is shown and to whom.

---

## Quick-Fire Round

| Question | Answer |
|---|---|
| What status code for unauthenticated access? | Usually `401 Unauthorized`. |
| What status code when authenticated but not allowed? | Usually `403 Forbidden`; some APIs use `404` to avoid resource disclosure. |
| What status code for rate-limit exceeded? | `429 Too Many Requests`. |
| What is the SSE content type? | `text/event-stream`. |
| What Go keyword starts a goroutine? | `go`. |
| What does `defer tx.Rollback()` do? | Schedules rollback when the function returns; after a successful commit, rollback has no effect on the committed transaction. |
| What is `clause.Locking{Strength: "UPDATE"}`? | A GORM expression for a locking clause such as `SELECT ... FOR UPDATE` on supported databases. |
| What does QStash `NotBefore` do? | Intended to delay message delivery until a specified time; verify exact SDK semantics. |
| What is HMAC? | A keyed message-authentication code built from a cryptographic hash function. |
| SSE vs WebSocket? | SSE is primarily server-to-client over HTTP; WebSocket supports full-duplex communication. |
| What is a connection pool? | A reusable pool of database connections that avoids establishing one for every request. |
| What is GORM `AutoMigrate`? | A helper that attempts to align database schema with model definitions; review its behavior and use explicit migrations for controlled production changes. |
| What does `c.Abort()` do in Gin? | Stops the remaining handler chain from running; it does not itself write a response. |
| Why use an HTTP-only cookie for a JWT? | JavaScript cannot read it directly, which reduces one route for token theft; it does not eliminate XSS or CSRF risks. |

---

## Concepts to nail

| Concept                        | Why it matters for Tradexa                                                                                                                                         |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| CAP theorem                    | Explain consistency and availability trade-offs during partitions. Avoid labeling the whole system simply “CP” without specifying the operation and failure model. |
| ACID vs BASE                   | Bid-state updates need strong transactional invariants; search and analytics may tolerate stale or eventual results.                                               |
| Deadlocks                      | Transactions locking multiple rows can deadlock. Keep lock order consistent and handle deadlock retries safely.                                                    |
| N+1 query problem              | Loading listings and then querying each highest bid separately can create many database round trips; batch or aggregate where appropriate.                         |
| Idempotency                    | Duplicate webhook deliveries should not duplicate payment or auction state transitions.                                                                            |
| Horizontal vs vertical scaling | Vertical scaling adds resources to a machine; horizontal scaling adds instances and requires coordination for shared state.                                        |
| Load balancer                  | Distributes requests; long-lived SSE/WebSocket connections and in-memory hubs affect routing and cross-instance delivery.                                          |
| CDN                            | Serves static assets or images from distributed edge locations, reducing origin load and often improving delivery latency.                                         |

---

## Source files to review

The original notes say this sheet was built by analyzing:

- `handlers/listing.go`
- `websocket/manager.go`
- `config/scheduler.go`
- `config/qstash.go`
- `middleware/rate_limit.go`
- `routes/route.go`
- `tradexa_system_design.md`

> [!success] Final interview checklist
> - Replace every suggested detail with what your code actually does.
> - Separate measured results from targets and estimates.
> - Be ready to explain lock scope, transaction boundaries, isolation level, and benchmark methodology.
> - Distinguish current implementation from proposed improvements.
> - Never describe a payment flow as escrow, a security boundary as complete, or a performance number as a benchmark result unless you can substantiate it.
