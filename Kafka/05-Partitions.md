Partition - will be a ordered append only log inside the topic  


Order.Placed is divided into 3 partitions:
partition 0 - event, event , event , event
partition 1 - event , event , event
partition 2 - event

Order.Placed- can receive lot of order events
partition allows Kafka to spread events across multiple logs so multiple consumers can process all events in order

partitions 0: order_1001, order_1004
partitions 1: order_1005, order_1003
partitions 2: order_1006

