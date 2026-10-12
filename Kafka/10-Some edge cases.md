## Kafka: Replicated Log and Offsets

Example: your order service publishes **“order 101 placed.”** Kafka stores it in a topic partition. A consumer reads it and records its progress with an **offset**.

1. **While publishing:** With `acks=all`, Kafka waits for all currently in-sync replicas to receive the message. Pair it with `min.insync.replicas=2` and replication factor 3: if Kafka cannot keep at least two in-sync copies, it rejects the write instead of accepting it with only one copy.
    
2. **While processing:** The consumer reads the message, updates the database, then commits its offset to Kafka.
    
3. **If the consumer crashes before committing the offset:** Kafka reads that message again after restart or reassignment. That avoids skipping it, but the database update could happen twice. Kafka stores committed offsets so a consumer can resume after a restart.
    
4. **If the consumer commits before updating the database:** A crash in between can make Kafka skip the message on restart, even though the database was never updated. So commit only after the work succeeds.
    
5. **If the consumer stays down too long:** Kafka keeps messages only for the topic's configured retention period. If the messages expire before the consumer returns, it can no longer read them.

## At-Least-Once, Simply

**At-least-once means: “Retry rather than risk silently skipping the work.”** That protects against many crashes, but a retry can cause the same action to run twice.

For example, the consumer adds ₹500 to a wallet, then crashes before ACKing in RabbitMQ or committing its Kafka offset. The message is delivered again, and a naive consumer adds another ₹500.

Make the handler **idempotent**: assign the message a unique ID, record that ID when applying the wallet update, and ignore the message if that ID was already processed.
