1)Write short note on:
a) Corda
b) Ripple
c) Hyperledger Fabric
d) Defi Architecture
e) Ethereum Virtual Machine
explain each one in detail

Ans:
Corda:
**Corda** is an specifically designed for **financial institutions** and regulated industries. Unlike public blockchains, Corda focuses on **privacy**, **legal identity**, and **direct transaction agreement** between parties
![[Pasted image 20261003201919.png|404]]
- Corda is used mainly for **financial transactions**.[scribd](https://www.scribd.com/document/991780604/5)
    
- Typical use cases:
    
    - Bank-to-bank transfers
        
    - Trade finance
        
    - Syndicated loans
        
    - Insurance contracts
        
    - Capital market deals
## How Corda is Different from Normal Blockchain

## Normal (Public) Blockchain:

- Data is stored in **blocks**.
    
- Blocks are **linked in order** (each block has the hash of the previous block).
    
- Each block contains **many transactions**.
    
- To do a transaction:
    1. Initiate transaction
    2. Broadcast to all nodes
    3. All nodes verify and validate
    4. If majority (e.g., >50%) accept, transaction is confirmed.
- This is done for **security and to prevent fraud**, but it is **slower**.
## Corda:
- Corda **does not use a global chain of blocks** in the same way.
- Instead, it focuses on **transactions linked to each other**.
- Each transaction contains the **hash of its previous transaction(s)** – so they form a chain of transactions, not necessarily a global block chain.
- Only the **parties involved** in a transaction see its details

- Question: If other nodes don’t verify, how is security maintained?
- Answer: Corda uses a special node called a **Notary Node**.
## Notary Node in Corda
- A **Notary Node** is a trusted node that has special power:
    - It can **accept or reject** a transaction.
    - It checks if the transaction is **valid and not double-spent**.
- Process for a transaction between two nodes (say, Node A and Node B):
    1. A and B create the transaction.
    2. Transaction is sent to the **Notary Node**.
    3. Notary verifies:
        - Are inputs valid?
        - Has this state already been used?
        - Are signatures correct?
    4. If everything is fine, Notary **signs/accepts** → transaction is successful.
    5. If something is wrong, Notary **rejects** → transaction fails.
- Because only trusted notaries validate, there is **no need to broadcast to everyone**, so it is **faster** and still **secure**
Corda is faster than public blockchains because it avoids the heavy communication and computation that public networks require. In a public blockchain, every transaction is broadcast to all nodes in the network. Each node then verifies the transaction, and the network must reach majority consensus before the transaction is confirmed. This process involves a lot of messages, computation, and waiting, which makes it slower. In Corda, by contrast, a transaction is shared only with the parties directly involved and a trusted Notary node. There is no need to broadcast the transaction to the entire network, and there is no need for global majority consensus. The Notary validates the transaction for uniqueness and correctness, and if it is valid, the transaction is finalized. Because Corda uses this targeted, permissioned approach with far fewer messages and less computation, transactions can be completed much more quickly than on a public blockchain