Kafka is a distributed event streaming platfrom 
**Not a Queue** 

storing a stream of events so many consumers can read it indepndently
order-service -> write an event to kafka

```json
{
  "eventType":"Order_placed",
  "orderID":"1141"
  "UserID":1234
}
```
![[Pasted image 20261011081938.png|586]]

order.placed topic
consumers do not remove msg from the kafka
 ![[Pasted image 20261011085154.png]]

analytic-service -> was down for 20min ->kafka can still keep the events
once analytics service is up again -kafka can continue from where it stopped 

new service -> fraud service - read old order related event also
producer doesnt know anything about consumer 
![[Pasted image 20261011090322.png]]![[Pasted image 20261011090332.png|149]]











