Explain the concept of double spending with a suitable example in detail

**Double spending** is an attempt to spend the same digital currency or token more than once. It is a major problem for digital money because digital data can be copied, unlike physical cash, which can exist in only one place at a time.[1][2]

## Why it matters

With a ₹100 note, if you give it to a shopkeeper, you no longer have that note, so you cannot give the same physical note to another person. But a digital coin is essentially data; without a reliable system, someone could copy it and send the “same” coin to two different people.[3][4]

Double spending breaks trust because a merchant may deliver goods after seeing a payment, while the same coins were already spent elsewhere.[2][5]

## Detailed example

Suppose Qais has **1 BTC** and wants to buy a laptop from Amit for 1 BTC.

### Step 1: First transaction

Qais creates this transaction:

```text
TX1: Qais -> Amit : 1 BTC
```

He signs it using his private key and broadcasts it to the blockchain network. At this stage, the transaction is still **unconfirmed** and waits in the network’s transaction pool, called the mempool.[1][6]

### Step 2: Double-spend attempt

Before TX1 is confirmed, Qais creates another transaction using the same 1 BTC:

```text
TX2: Qais -> Riya : 1 BTC
```

Now the network sees two conflicting transactions:

```text
TX1: Qais -> Amit : 1 BTC
TX2: Qais -> Riya : 1 BTC
```

Both use the same available balance, so only one can ultimately be valid.[1][4]

### Step 3: Network validation

Nodes check the available unspent balance. Since Qais has only 1 BTC, the network cannot accept both transactions.

```text
Qais balance: 1 BTC

TX1 spends 1 BTC -> valid if selected
TX2 spends same 1 BTC -> invalid if TX1 is selected
```

The blockchain records only one transaction and rejects the other.[1][4]

### Step 4: Final outcome

If TX1 is included in a block first:

```text
Block N
------------------
TX1: Qais -> Amit : 1 BTC
------------------
```

Then TX2 is rejected because Qais no longer has 1 BTC to spend. Amit receives the money, and Riya does not.[1]

If TX2 is included first, then TX1 is rejected and Amit does not receive the payment. Therefore, a merchant should wait for confirmations before delivering expensive goods.[1][7]

## How blockchain prevents it

Blockchain prevents double spending through:

- **Transaction validation:** Nodes check whether the sender actually owns enough unspent coins.
- **Consensus mechanism:** Miners or validators agree on one valid version of the transaction history.
- **Block confirmation:** Once a transaction is added to a block, later blocks build on top of it, making reversal increasingly difficult.
- **Cryptographic signatures:** A transaction must be signed with the sender’s private key, proving ownership.[4][6][8]

For example, in Bitcoin, a transaction becomes more secure as more blocks are added after it. Merchants commonly wait for several confirmations before treating a large payment as final.[6][7]

## Common double-spend attacks

| Attack | How it works |
|---|---|
| **Race attack** | The attacker sends two conflicting transactions quickly, hoping the merchant accepts the one that will later be rejected. [1][9] |
| **Finney attack** | The attacker pre-mines a block containing a transaction, spends the same coins elsewhere, then tries to release the pre-mined block to reverse the payment. [10] |
| **51% attack** | An attacker controls more than half of the network’s mining/staking power and can reorganize recent blocks, potentially reversing a transaction. [8][9] |

What are the methods of solving the double spending problem in Bitcoin?Explain those methods with examples each.

Bitcoin solves double spending through **UTXO validation, public distributed ledger, transaction broadcasting, Proof-of-Work consensus, longest-chain rule, and transaction confirmations**. These methods ensure that the same bitcoin can be included in only one valid transaction.[1][2][3]

## UTXO validation

Bitcoin does not store a simple account balance. It tracks **Unspent Transaction Outputs (UTXOs)**. To spend bitcoin, a transaction must reference an existing UTXO as its input. Once that UTXO is spent, it cannot be used again. 

### Example

Suppose Qais has one UTXO:

```text
UTXO A: 0.5 BTC
```

He tries to send the same UTXO to two people:

```text
TX1: UTXO A -> Amit : 0.5 BTC
TX2: UTXO A -> Riya : 0.5 BTC
```

If TX1 is confirmed first, UTXO A becomes spent. Therefore, TX2 is invalid because it refers to a UTXO that no longer exists. 

```text
UTXO A spent by TX1
        |
        v
TX2 rejected: input already spent
```

## Public distributed ledger

Every full Bitcoin node maintains a copy of the blockchain and its transaction history. When a node receives a transaction, it checks whether the referenced UTXO has already been spent in an earlier confirmed transaction.[2][3]

### Example

Qais sends 1 BTC to Amit in Block 100:

```text
Block 100
TX1: Qais -> Amit : 1 BTC
```

Later, Qais tries to send the same 1 BTC to Riya:

```text
TX2: Qais -> Riya : 1 BTC
```

Nodes compare TX2 against the public ledger. They see that the input UTXO was already consumed by TX1, so they reject TX2.[2][3]

## Transaction broadcasting

Bitcoin transactions are publicly announced to the peer-to-peer network before being confirmed. This allows many nodes to see conflicting transactions quickly and reject duplicates.[1][2]

### Example

Qais creates two conflicting transactions:

```text
TX1: Qais -> Amit : 1 BTC
TX2: Qais -> Riya : 1 BTC
```

Both transactions use the same input UTXO. When they are broadcast:

```text
Node 1 sees TX1 and TX2
Node 2 sees TX1 and TX2
Node 3 sees TX1 and TX2
```

Each node recognizes that only one can be valid. Miners normally include only one of the conflicting transactions in a block; the other remains unconfirmed or is rejected.[1][2]

## Proof-of-Work consensus

Bitcoin uses **Proof of Work (PoW)**. Miners compete to find a valid block hash by changing a value called the nonce. A block is accepted only if its hash meets the network difficulty target.[1][2]

This makes it expensive to create an alternative blockchain history or reverse confirmed transactions. An attacker would need to redo the Proof of Work for the attacked block and all blocks after it.[1][2]

### Example

Suppose Qais sends 1 BTC to Amit, and the transaction is included in Block 100. Qais wants to reverse it and send the same 1 BTC to Riya.

He must create an alternative Block 100 containing TX2, then mine new blocks faster than the rest of the network:

```text
Honest chain:
Block 100: Qais -> Amit
Block 101
Block 102
...

Attacker chain:
Block 100': Qais -> Riya
Block 101'
Block 102'
...
```

To replace the honest chain, Qais must produce a longer valid chain than all honest miners combined. This is extremely costly and unlikely unless he controls more than 50% of the network’s mining power.[1][2]

## Longest valid chain rule

If two miners create valid blocks at nearly the same time, the network may temporarily have two branches. Bitcoin resolves this by treating the chain with the most accumulated Proof of Work—usually the longest valid chain—as the correct history.[3][4]

### Example

```text
Block 99
     / \
Block 100A   Block 100B
     |           |
Block 101A   Block 101B
     |           |
Block 102A   Block 101C
```

If the branch through `100A` becomes longer first:

```text
Block 99
   |
Block 100A
   |
Block 101A
   |
Block 102A
```

Then nodes continue building on `100A`. The shorter branch is abandoned, and its transactions return to the mempool for confirmation again.[3][4]

This prevents an attacker from easily replacing a confirmed transaction with a conflicting one.[3][4]

## Transaction confirmations

A transaction is included in one block after the first confirmation. However, users often wait for additional blocks to be added on top of that block. More confirmations make a transaction harder to reverse.[3][5]

### Example

Qais sends 1 BTC to Amit:

```text
Block 100: TX1: Qais -> Amit
```

After more blocks are mined:

```text
Block 100: TX1: Qais -> Amit
Block 101
Block 102
Block 103
Block 104
Block 105
Block 106
```

TX1 now has six confirmations. To reverse it, an attacker would need to replace Block 100 and mine six additional blocks faster than the honest network. This is practically infeasible for a normal attacker.[3][5]

For high-value payments, merchants commonly wait for 2–6 confirmations before delivering goods.[3][5]

## Cryptographic signatures

Every Bitcoin transaction is signed with the sender’s private key. Other nodes verify the signature using the sender’s public key. This proves that the owner authorized the transaction, but it does not by itself prevent double spending; it works together with UTXO validation and consensus.[1][2]

### Example

Qais signs:

```text
TX1: Qais -> Amit : 1 BTC
```

with his private key. Nodes verify the signature using Qais’s public key. If someone tries to create a fake transaction from Qais’s address without the private key, nodes reject it.[1][2]

##