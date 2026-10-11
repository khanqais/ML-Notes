![[Pasted image 20261010231713.png]]
queue - buffer between services
order-service publish a message to a queue
order service does not wait for the noti-service to finish sending the email 

Once the order is placed:
```json
{
  "eventType":"Order_placed",
  "orderID":"1141"
  "UserID":1234
}
```

order service puts this msg into the queue
noti-service consume it and send the email

system is less tighly coupled 
but we need multiple queues for:
OrderService- NQ-Nofi-service
OrderService- Ana-Nofi-Analyservice


consumed is already consumed -> removed from the queue 
backgrounds jobs 

analytics-service —> new service —> process orders from the last 7 days is lost 


