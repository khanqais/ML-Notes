Explain the concept of UTXO model of Bitcoin in detail with example

The **UTXO model** is Bitcoin’s accounting system. Instead of storing a single account balance like `Qais = 1 BTC`, Bitcoin tracks individual **Unspent Transaction Outputs (UTXOs)**—spendable chunks of bitcoin created by earlier transactions.[1][2]

## What is a UTXO?

A UTXO is an output from a previous transaction that has not yet been used as an input in another transaction. Each UTXO has:

- A value, such as 0.4 BTC
- A locking condition, usually tied to the owner’s public key/address
- A reference to the earlier transaction that created it[3][4]

A wallet balance is simply the total value of all UTXOs the wallet can spend. For example, a wallet showing 0.45 BTC may actually hold one 0.40 BTC UTXO and one 0.05 BTC UTXO.[3][5]

## Important rule

A UTXO cannot be spent partially. It must be consumed completely, and the transaction creates new UTXOs for the recipient and the sender’s change.[3][6][7]

Think of it like paying with a ₹500 note for a ₹300 item: the ₹500 note is used fully, and you receive ₹200 as change. Bitcoin works similarly, except the “change” is a newly created UTXO.[1][6]

## Transaction structure

A Bitcoin transaction has:

```text
Inputs  -> UTXOs being spent
Outputs -> New UTXOs being created
Fee     -> Inputs - Outputs
```

When a transaction is confirmed:

- The input UTXOs are marked as spent.
- New output UTXOs are created.
- The difference between inputs and outputs becomes the miner’s transaction fee.[3][7][8]

## Example: sending 0.3 BTC

Suppose Alice has one UTXO:

```text
UTXO A: 0.3 BTC
Owner: Alice
```

Alice wants to send **0.25 BTC** to Bob.

She cannot spend only 0.25 BTC from UTXO A. She must spend the entire 0.3 BTC UTXO.[8]

### Transaction

```text
Inputs:
  UTXO A: 0.3 BTC

Outputs:
  Output 1: 0.25 BTC -> Bob
  Output 2: 0.0499 BTC -> Alice (change)

Fee:
  0.3 - (0.25 + 0.0499) = 0.0001 BTC
```

### Before transaction

```text
Alice's UTXOs:
  UTXO A: 0.3 BTC

Bob's UTXOs:
  None
```

### After transaction

```text
Alice's UTXOs:
  New change UTXO: 0.0499 BTC

Bob's UTXOs:
  New UTXO: 0.25 BTC

Spent UTXO:
  UTXO A: 0.3 BTC  -> no longer usable
```

UTXO A is destroyed, while two new UTXOs are created: one for Bob and one for Alice.[3][8]

## Example: using multiple UTXOs

Suppose Alice has:

```text
UTXO A: 0.4 BTC
UTXO B: 0.05 BTC
Total balance: 0.45 BTC
```

She wants to send **0.3 BTC** to Bob.

Her wallet selects UTXO A because 0.4 BTC is enough:

```text
Inputs:
  UTXO A: 0.4 BTC

Outputs:
  Bob: 0.3 BTC
  Alice change: 0.0999 BTC

Fee:
  0.4 - (0.3 + 0.0999) = 0.0001 BTC
```

After confirmation:

```text
Alice's UTXOs:
  0.05 BTC (UTXO B)
  0.0999 BTC (new change UTXO)

Bob's UTXOs:
  0.3 BTC
```

The original 0.4 BTC UTXO is spent and cannot be reused.[3][5]

## How it prevents double spending

When a transaction is broadcast, nodes check that every input refers to an existing, unspent UTXO. If two transactions try to spend the same UTXO, only one can be accepted; the other is rejected.[3][4]

Example:

```text
UTXO A: 0.3 BTC

TX1: Spend UTXO A -> Amit
TX2: Spend UTXO A -> Riya
```

If TX1 is confirmed first, UTXO A becomes spent. Therefore, TX2 is invalid because it refers to a UTXO that no longer exists in the UTXO set.[4][8]

## UTXO vs account model

| Feature | UTXO model | Account model |
|---|---|---|
| Used by | Bitcoin | Ethereum |
| Balance storage | Collection of unspent outputs | Single account balance |
| Transaction | Consumes old UTXOs, creates new UTXOs | Debits sender, credits receiver |
| Partial spending | Not possible; change UTXO is created | Possible |
| Privacy | Can use new addresses for change | Addresses/accounts are easier to link |

In the UTXO model, Bitcoin does not maintain a table of user balances; it maintains the set of all currently spendable outputs.[2][9]