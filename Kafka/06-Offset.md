an offset - the position of a message inside a partition
topic -> partitions -> offsets

every message in a particular gets an offset number

topic order.placed
partition 0
offset 0 -> order_1001
offset 1 -> order_1004
offset 2 -> order_1007
offset 3 -> order_1010

**Offset is not the event ID. It is just a position in the event.**

nofi-service = offset 25
ana-service = offset 12
inventory-service= offset 30
these offsets will make sure that our events are tracked very clearly and consistently. Let's say some service goes down. If that service goes up, it will see that these offsets are processed and that I need to work on the next offset. 