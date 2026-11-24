Explain different visibility specifier of functions in solidity with example

## The four visibility specifiers

Solidity has four function visibility specifiers: `public`, `external`, `internal`, and `private`. They control **who can call** a function — from inside the contract, from a derived (inherited) contract, or from outside entirely [](https://docs.soliditylang.org/en/latest/contracts.html?highlight=visibility).[1]

| Specifier | Same contract | Derived contract | Outside (other contract / user) |
|---|---|---|---|
| `public` | Yes | Yes | Yes |
| `external` | No (only via `this.f()`) | Yes | Yes |
| `internal` | Yes | Yes | No |
| `private` | Yes | No | No |

## `public`

Callable from anywhere — inside the contract, from derived contracts, and from other contracts or users via a transaction [](https://docs.soliditylang.org/en/latest/contracts.html?highlight=visibility). It's the default choice when you want a function exposed in the contract's ABI (its external interface).[1]

```solidity
contract Counter {
    uint public count;

    function increment() public {
        count += 1;
    }
}
```

Anyone can call `increment()` — a user, another contract, or the contract itself.

## `external`

Part of the contract's interface, so it can be called from other contracts and via transactions, but **not** directly from inside the same contract — you'd need `this.f()` syntax, which creates an external message call [](https://docs.soliditylang.org/en/latest/contracts.html?highlight=visibility). This is slightly cheaper on gas than `public` for functions that are only meant to be called externally, since parameters are read directly from calldata.[1]

```solidity
contract TokenSale {
    function buyTokens(address buyer) external payable {
        // logic
    }
}
```

Calling `buyTokens()` from inside the same contract directly would fail; you'd need `this.buyTokens(...)`.

## `internal`

Callable only from within the current contract **or** contracts that inherit from it. It's not exposed in the ABI, so it can't be called externally [](https://docs.soliditylang.org/en/latest/contracts.html?highlight=visibility). Think of it as "family access" — the contract and its children can use it, but no one else.[1]

```solidity
contract Base {
    uint internal itemCount;

    function incrementCount() internal {
        itemCount++;
    }
}

contract Child is Base {
    function doSomething() public {
        incrementCount();  // works, because Child inherits Base
    }
}
```

An outside contract calling `incrementCount()` directly would fail.

## `private`

Only callable from within the contract that defines it — not even derived contracts can see it [](https://docs.soliditylang.org/en/latest/contracts.html?highlight=visibility). It's the most restrictive.[1]

```solidity
contract BankAccount {
    uint private balance;

    function updateBalance(uint amount) private {
        balance += amount;
    }
}
```

Even a contract inheriting from `BankAccount` can't call `updateBalance()`.

## Important note: "private" ≠ secret

A common misconception: `private` doesn't mean the data is hidden from the blockchain. Everything on-chain is public and readable by anyone. `private` only restricts **who can call** the function or access the variable **through the contract's interface** — anyone can still read the raw storage directly using low-level tools [](https://dev.to/oxrajesh/a-deep-dive-into-solidity-visibility-specifiers-public-private-internal-and-external-8a). Never store sensitive data (like passwords or keys) thinking `private` protects it.[2]

## When to use which

- `public` — when you want the function callable from anywhere (most common for user-facing actions).
- `external` — when it's only meant to be called from outside, saves a bit of gas.
- `internal` — helper functions shared with child contracts.
- `private` — internal helpers that shouldn't leak to child contracts.

The official style guide recommends ordering functions by visibility: constructor, receive, fallback, external, public, internal, private [](https://docs.soliditylang.org/en/latest/style-guide.html).[3]

