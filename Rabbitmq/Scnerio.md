Suppose a message says: “Add ₹500 to customer A’s wallet.” The system needs to decide what to do if a service crashes while handling it. ????


Suppose a payment service confirms payment and publishes this message:
```json
{
  "event_id": "credit-abc123",
  "customer_id": "A",
  "amount_paise": 50000
}
```
A wallet worker handles it:

1. Receive the message from RabbitMQ; **do not ACK it yet**.
2. Start a PostgreSQL transaction.
3. Try to insert `credit-abc123` into a `processed_messages` or `wallet_transactions` table, where `event_id` has a **unique constraint**.
4. If it’s a new ID, add ₹500 to customer A’s wallet and insert the ledger entry, all in the **same database transaction**.
5. Commit the database transaction.
6. Only after commit, ACK the RabbitMQ message.
A database transaction ensures the ledger entry and balance update commit together or roll back together. A unique constraint prevents concurrent duplicate deliveries from applying the same credit twice.

| Failure point                                                         | Result with this design                                                                                                                                                                 |
| --------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Worker crashes before starting the database transaction               | No credit is applied. RabbitMQ redelivers the unacknowledged message.                                                                                                                   |
| Worker crashes during the transaction                                 | PostgreSQL rolls back the unfinished transaction. RabbitMQ redelivers; the worker tries again.                                                                                          |
| Database transaction commits, then worker crashes before RabbitMQ ACK | RabbitMQ may redeliver. The unique `event_id` already exists, so the worker does **not** credit the wallet again; it ACKs the duplicate.                                                |
| Worker ACKs before committing the database transaction                | **Danger:** if it crashes before commit, the message is gone but the credit was not saved. Always commit first, ACK second.                                                             |
| Two workers receive the same message around the same time             | The database unique constraint allows only one transaction to claim that `event_id`; the other treats it as already processed.                                                          |
| RabbitMQ publish times out and the producer retries                   | The same event may be published twice. The database’s unique ID makes the duplicate harmless. RabbitMQ publisher confirms help the producer know whether the broker accepted a publish. |
| Message repeatedly fails                                              | Retry with a limit and backoff; then send it to a dead-letter queue and alert someone. Don’t retry a permanent error forever.                                                           |
## The Main Rule

**Database commit first, RabbitMQ ACK second, and deduplicate every operation using a stable unique ID.**

