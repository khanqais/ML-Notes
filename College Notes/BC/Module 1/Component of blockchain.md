Explain the components of Blockchain

Blockchain has several core components that work together to record, validate, and secure transactions in a decentralized way. The main components are **blocks, distributed ledger, nodes, cryptography, consensus mechanism, peer-to-peer network, and smart contracts**.[1][2]

## Blocks

A **block** is a container that stores a group of transactions. Each block contains transaction data, a timestamp, its own cryptographic hash, and the hash of the previous block.[3][4]

```text
Block N
------------------
Transactions
Timestamp
Previous Block Hash
Current Block Hash
Merkle Root
```

The previous-block hash links blocks in chronological order. If someone changes an old transaction, that block’s hash changes, which breaks the link to the next block.[3][4]

## Distributed ledger

A **distributed ledger** is the shared database that stores all transactions. Instead of one central server holding the data, every participating node keeps a synchronized copy of the ledger.[1][2]

For example, if there are 1,000 nodes in a public blockchain, many of them store the same transaction history. This reduces the risk that one compromised server can alter the record.[2][5]

## Nodes

A **node** is a computer participating in the blockchain network. Nodes receive transactions, validate them, store the ledger, and share valid blocks with other nodes.[1][6]

Common node types:

- **Full node:** Stores the complete blockchain and validates transactions/blocks.
- **Light node:** Stores only essential headers, not the full blockchain.
- **Miner/validator node:** Creates or validates new blocks according to the consensus rules.[1]

## Transactions

A **transaction** is the basic unit of data in a blockchain. It usually records a transfer of value or information, such as sending cryptocurrency from one wallet address to another.[1]

A typical transaction includes:

- Sender’s public address
- Receiver’s public address
- Amount/value
- Timestamp
- Digital signature created using the sender’s private key[1][2]

Example: “Qais sends 0.5 ETH to Amit” is a transaction. The network verifies Qais’s signature before adding it to a block.[1]

## Cryptography

**Cryptography** provides security, identity, and integrity in blockchain.

- **Hash functions:** Convert data into a fixed-length fingerprint. A tiny change in input produces a completely different hash.
- **Public/private key pairs:** A public key acts like an account address; a private key is used to sign transactions.
- **Digital signatures:** Prove that the owner of a private key authorized a transaction.[2][5]

For example, Qais signs a transaction with his private key. Other nodes verify it using Qais’s public key, but they cannot recover or use his private key.[2]

## Consensus mechanism

A **consensus mechanism** is the set of rules that lets decentralized nodes agree on which transactions are valid and which block should be added next. It prevents different nodes from having conflicting versions of the ledger.[1][6]

| Mechanism | How it works | Example |
|---|---|---|
| Proof of Work (PoW) | Miners solve a computational puzzle to add a block | Bitcoin |
| Proof of Stake (PoS) | Validators stake tokens and are selected to propose/validate blocks | Ethereum |
| Delegated Proof of Stake (DPoS) | Token holders vote for delegates who validate blocks | Some enterprise/public chains |

PoW uses computation, while PoS uses economic stake as the main security mechanism.[1]

## Peer-to-peer network

Blockchain runs on a **peer-to-peer (P2P) network**, meaning nodes communicate directly rather than through one central server. When a node receives a new transaction or block, it broadcasts it to connected peers.[1][7]

This is why blockchain is decentralized: there is no single server that all users must trust or depend on.[5][7]

## Smart contracts

A **smart contract** is a program stored and executed on a blockchain. It automatically performs actions when predefined conditions are met—for example, releasing payment after delivery is confirmed.[1][2]

```text
if (deliveryConfirmed == true) {
    releasePaymentToSeller();
}
```

Smart contracts remove the need for a trusted middleman for certain automated business rules.[2][5]

## Merkle tree

A **Merkle tree** organizes the hashes of all transactions inside a block. Its final output, the **Merkle root**, is stored in the block header and acts as a compact fingerprint for every transaction in that block. 

```text
Merkle Root
     /       \
  H(1,2)   H(3,4)
   /  \      /  \
 H1   H2   H3   H4
```

It allows efficient verification that a transaction belongs to a block without downloading every transaction. 

## How they work together

1. A user creates and signs a transaction with a private key.[1][2]
2. The transaction is broadcast through the P2P network to nodes.[1][7]
3. Nodes validate the signature, balance, and transaction rules.[1]
4. Valid transactions are grouped into a block.[3]
5. The block’s transactions are hashed into a Merkle root. 
6. Nodes reach consensus and add the block to the distributed ledger.[1][6]
7. Every node updates its copy of the blockchain.[1][2]

# With a suitable diagram, explain the structure of a block header with a list of transactions

A blockchain block has two main parts: a **block header** (metadata) and a **transaction list** (actual records). The header summarizes and secures the transactions using a Merkle root, while the previous-block hash links this block to the earlier block.[1][2]

## Block structure diagram

```text
+--------------------------------------------------------------+
|                         BLOCK N                              |
+--------------------------------------------------------------+
|                       BLOCK HEADER                           |
|                                                              |
|  Version: 1                                                  |
|  Previous Block Hash: 0000000000000000000a1b2c...            |
|  Merkle Root: 9f8e7d6c5b4a...                                |
|  Timestamp: 22-Nov-2026 17:30:00                             |
|  Difficulty Target: 0000000000000000000f...                  |
|  Nonce: 184729                                               |
+--------------------------------------------------------------+
|                 TRANSACTION COUNTER: 4                       |
+--------------------------------------------------------------+
|                     LIST OF TRANSACTIONS                     |
|                                                              |
|  T1: Qais -> Amit : ₹100                                     |
|  T2: Amit -> Riya : ₹50                                      |
|  T3: Riya -> Sam  : ₹30                                      |
|  T4: Sam -> Neha  : ₹20                                      |
+--------------------------------------------------------------+
```

The transaction list is stored in the block body, while the header contains only a compact summary of that body.[2][3]

## Block header fields

| Field | Purpose |
|---|---|
| **Version** | Identifies the blockchain protocol/software version used to create the block. [1][2] |
| **Previous block hash** | Stores the hash of the immediately preceding block. This creates the cryptographic chain. [1][2] |
| **Merkle root** | A single hash that represents all transactions in the current block. [1][2] |
| **Timestamp** | Records the approximate time when the block was created. [1][2] |
| **Difficulty target / bits** | Defines how difficult it is to find a valid block hash in Proof-of-Work systems. [1][4] |
| **Nonce** | A number miners change repeatedly while searching for a valid Proof-of-Work hash. [1][4] |

In Bitcoin, the header is typically 80 bytes and contains version, previous block hash, Merkle root, time, bits, and nonce.[2][3]

## Transactions and Merkle root

The transaction list is hashed into a Merkle tree. The root of that tree is stored in the block header.[1][2]

```text
                    Merkle Root
                   /           \
              H(T1,T2)       H(T3,T4)
              /      \        /      \
          H(T1)   H(T2)   H(T3)   H(T4)
            |       |        |       |
            T1      T2       T3      T4
```

Here:

```text
H(T1) = hash("Qais -> Amit : ₹100")
H(T2) = hash("Amit -> Riya : ₹50")
H(T3) = hash("Riya -> Sam : ₹30")
H(T4) = hash("Sam -> Neha : ₹20")

H(T1,T2) = hash(H(T1) + H(T2))
H(T3,T4) = hash(H(T3) + H(T4))

Merkle Root = hash(H(T1,T2) + H(T3,T4))
```

The Merkle root is placed in the block header.[1][2]

## How blocks are linked

The current block stores the hash of the previous block:

```text
Block N-1                       Block N
+-----------------+             +-------------------+
| Block hash: H1  | <---------- | Previous Hash: H1 |
| Transactions    |             | Merkle Root: MR2  |
+-----------------+             | Block hash: H2    |
                                +-------------------+
```

If even one transaction in Block N changes, its Merkle root changes, which changes the block hash. The next block’s `previous hash` will no longer match, so tampering becomes detectable.[1][5]

## 