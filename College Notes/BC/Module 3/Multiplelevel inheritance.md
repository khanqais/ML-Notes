Write a program in solidity to implement multi-level inheritance

## What is multi-level inheritance?

Multi-level inheritance is when a contract inherits from another contract, which itself inherits from a third — forming a chain like `A → B → C`. The middle contract (`B`) acts as both a child (of `A`) and a parent (of `C`) [](https://www.geeksforgeeks.org/solidity/solidity-inheritance/).[1]

```
A (grandparent)
  ↓
B (parent, child of A)
  ↓
C (child of B, grandchild of A)
```

## The program

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

// ---- Level 1: Grandparent ----
contract Animal {
    string public name;

    constructor(string memory _name) {
        name = _name;
    }

    function eat() public pure returns (string memory) {
        return "I can eat";
    }
}

// ---- Level 2: Parent (inherits Animal) ----
contract Dog is Animal {
    constructor(string memory _name) Animal(_name) {
        // Animal's constructor is called with _name
    }

    function bark() public pure returns (string memory) {
        return "Woof Woof";
    }
}

// ---- Level 3: Child (inherits Dog, which inherits Animal) ----
contract Puppy is Dog {
    constructor(string memory _name) Dog(_name) {
        // Dog's constructor is called, which calls Animal's
    }

    function play() public pure returns (string memory) {
        return "I love playing";
    }

    // Override a parent function
    function eat() public pure override returns (string memory) {
        return "Puppy eats puppy food";
    }
}
```

## How it works

- `Animal` is the base (grandparent) — defines `name` and `eat()`.
- `Dog is Animal` — `Dog` inherits `name` and `eat()`, adds `bark()`.
- `Puppy is Dog` — `Puppy` inherits everything from both `Dog` and `Animal`, adds `play()` and overrides `eat()`.

The `is` keyword creates the inheritance link. `Puppy` can call `bark()` (from `Dog`) and access `name` (from `Animal`) even though it only directly inherits from `Dog` [](https://dev.to/shlok2740/multi-level-inheritance-in-solidity-3hpe).[2]

## Constructor chaining

Notice the constructor syntax:

```solidity
constructor(string memory _name) Animal(_name) { }
```

When `Puppy` is deployed, its constructor runs first, then calls `Dog`'s constructor, which calls `Animal`'s constructor. This is **constructor chaining** — each level must pass required arguments up the chain [](https://solidity.in/practice/inheritance-and-interfaces.html).[3]

## Function overriding

`Puppy` overrides `eat()` from `Animal`. For this to work:

- The parent function must be marked `virtual`.
- The child function must be marked `override`.

```solidity
// In Animal
function eat() public pure virtual returns (string memory) { ... }

// In Puppy
function eat() public pure override returns (string memory) { ... }
```

Without `virtual` on the parent, the compiler throws an error. Without `override` on the child, same thing [](https://github.com/argotorg/solidity/blob/develop/docs/contracts/inheritance.rst).[4]

## Access control across levels

| Member | Animal | Dog | Puppy | Outside |
|---|---|---|---|---|
| `name` (public) | Yes | Yes | Yes | Yes |
| `eat()` (public) | Yes | Yes | Yes (overridden) | Yes |
| `bark()` (public) | — | Yes | Yes | Yes |
| `play()` (public) | — | — | Yes | Yes |

A `private` member in `Animal` would **not** be accessible in `Dog` or `Puppy` — only `public` and `internal` members cross inheritance boundaries [](https://github.com/argotorg/solidity/blob/develop/docs/contracts/inheritance.rst).[5]

## Deploying and testing

Deploy `Puppy` (not `Animal` or `Dog` directly) with a name like `"Rocky"`. Then:

- Call `name()` → returns `"Rocky"` (inherited from Animal).
- Call `bark()` → returns `"Woof Woof"` (inherited from Dog).
- Call `play()` → returns `"I love playing"` (defined in Puppy).
- Call `eat()` → returns `"Puppy eats puppy food"` (overridden version).

## Interview one-liner

"Multi-level inheritance is a chain like `A → B → C`, where `B` is both a child of `A` and a parent of `C`. Use `is` to inherit, chain constructors with `Parent(args)`, and use `virtual`/`override` for function overriding. Only `public` and `internal` members cross inheritance boundaries — `private` stays locked to the defining contract."

Citations:
[1] Solidity - Inheritance - GeeksforGeeks https://www.geeksforgeeks.org/solidity/solidity-inheritance/
[2] Multi-level Inheritance in Solidity - DEV Community https://dev.to/shlok2740/multi-level-inheritance-in-solidity-3hpe
[3] Inheritance and interfaces – Solidity lesson | Solidity.in https://solidity.in/practice/inheritance-and-interfaces.html
[4] Inheritance | Solidity by Example | 0.8.26 https://solidity-by-example.org/inheritance/
[5] solidity/docs/contracts/inheritance.rst at develop - GitHub https://github.com/argotorg/solidity/blob/develop/docs/contracts/inheritance.rst
[6] 30 Days Of Solidity: Multi-level Inheritance - GitHub https://github.com/Satellite0719/30-Days-of-Solidity/blob/main/Day%2022%20-%20Multi-level%20Inheritance/readme.md
[7] Day 22 - Multi-level Inheritance - DEV Community https://dev.to/envoy_/day-22-multi-level-inheritance-1m9e
[8] Solidity Inheritance Examples | PDF https://www.scribd.com/document/834180932/blockchain
[9] inheritance - Multiple inherintance: solidity docs example ... https://ethereum.stackexchange.com/questions/56094/multiple-inherintance-solidity-docs-example-explanation
[10] 100-Days-of-Solidity/Day 23 - Hierarchical Inheritance/readme ... https://github.com/KcPele/100-Days-of-Solidity/blob/main/Day%2023%20-%20Hierarchical%20Inheritance/readme.md
[11] Solidity 0.7.0 documentation https://docs.soliditylang.org/en/v0.7.0/
[12] Solidity — Solidity 0.8.39-develop documentation https://docs.solidity.org/en/latest/
