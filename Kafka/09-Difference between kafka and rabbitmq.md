|Aspect|RabbitMQ|Kafka|
|---|---|---|
|Model|Smart broker, simple consumer|Simple broker, smart consumer|
|After consume|Message deleted|Message stays (retention-based)|
|Replay|No|Yes, by rewinding the offset|
|Multiple independent readers|Not the natural fit|Yes, via separate consumer groups|
|Ordering|Global order with a single consumer. Parallel consumers lose strict order.|Order only within a partition, chosen by partition key. No global order.|
|Throughput|About 4k–10k msg/s|1M+ msg/s|
|Latency|About 1–5 ms at low volume, because the broker pushes messages|About 5–50 ms, because consumers pull in batches. Stays steady under load.|
|Delivery|At-most-once and at-least-once|At-least-once. "Exactly-once" only for Kafka-to-Kafka within one cluster using transactions.|
|DLQ and retries|Built in|You build them yourself|
|Operations|Easy: single binary, simple clustering, management UI|Harder: rebalancing, broker failures, topic config, consumer groups. No ZooKeeper in newer versions (it uses Raft).|
