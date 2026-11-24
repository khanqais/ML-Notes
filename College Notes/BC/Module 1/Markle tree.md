![[Screenshot_2026-11-22-17-23-13-077.png]]# What is a Merkle tree? Explain the structure of a Merkle tree in detail with example

A **Merkle tree** (hash tree) is a binary tree of hashes used to verify that a large set of data has not changed. Its top hash, called the **Merkle root**, acts as a compact fingerprint for all the underlying data.[1][2]

## Structure

A Merkle tree has three levels:

- **Leaves:** Each leaf stores the hash of one data item, such as one transaction.
- **Intermediate nodes:** Each stores the hash of its two child hashes.
- **Root node:** The final hash, called the **Merkle root**, summarizes every leaf.[3][4]

The tree is built **bottom-up**: first hash the data, then pair hashes and hash the pairs, and repeat until only one hash remains.[3][5]

```text
                 Merkle Root
                /           \
        Hash(AB)         Hash(CD)
          /   \             /   \
   Hash(A) Hash(B)   Hash(C) Hash(D)
       |       |         |       |
       A       B         C       D
```

Here, `A`, `B`, `C`, and `D` are the original data items; their hashes form the leaves.[4]

## Example with four transactions

Suppose a blockchain block contains four transactions:

```text
T1 = "Qais -> Amit: ₹100"
T2 = "Amit -> Riya: ₹50"
T3 = "Riya -> Sam: ₹30"
T4 = "Sam -> Neha: ₹20"
```

### Step 1: Hash each transaction

Each transaction is hashed individually:

```text
H1 = hash(T1)
H2 = hash(T2)
H3 = hash(T3)
H4 = hash(T4)
```

These four hashes are the leaf nodes.[3][5]

### Step 2: Pair and hash leaves

Combine adjacent leaf hashes in pairs:

```text
H12 = hash(H1 + H2)
H34 = hash(H3 + H4)
```

So:

```text
          H12        H34
         /   \      /   \
       H1     H2  H3     H4
```

`H12` proves that transactions T1 and T2 are unchanged; `H34` proves T3 and T4 are unchanged.[3][4]

### Step 3: Create the Merkle root

Combine the two intermediate hashes:

```text
Merkle Root = hash(H12 + H34)
```

Final structure:

```text
                 Merkle Root
                /           \
             H12             H34
            /   \           /   \
          H1     H2       H3     H4
          |      |        |      |
          T1     T2       T3     T4
```

The Merkle root is stored in the blockchain block header.[3][6]

## Why it is useful

If someone changes T2 from “₹50” to “₹500”:

```text
T2 changes
  -> H2 changes
  -> H12 changes
  -> Merkle Root changes
```

So the block’s stored Merkle root no longer matches the recalculated root. This makes tampering easy to detect without checking every transaction separately.[2][5]

It is also efficient: to prove that T2 belongs to the block, you do not need all four transaction hashes. You need T2’s hash plus the hashes along its path—`H1`, `H34`, and the root. This is called a **Merkle proof**.[6][7]

## Odd number of leaves

If there are 5 transactions, the last unpaired leaf is usually duplicated or paired with itself:

```text
H5 -> H5
```

Then:

```text
H56 = hash(H5 + H5)
