![[Pasted image 20261010223914.png|649]]
![[Pasted image 20261010230725.png|649]]

sync rest chain-
one service directly calls multiple other services during the same req 
order-service
immediately calls: noti-service, analytics-service, inventory-service
they all are tightly coupled
one-service : noti-service slows down- slow the full request
```ts
try{
  await createOrder(ordeData)
  await notiUser(ordeData)
  await analytics(ordeData)
  await updateInventory(ordeData)
  
}
```
create messy code
adding a new service will be problematic
how to solve this??


order-service ->