Explain fixed and dynamic arrays in solidity with suitable examples

## What are arrays in Solidity?

An array is a collection of elements of the same type, accessed by index. Solidity has two kinds: **fixed-size** (length set at compile time, can't change) and **dynamic** (length can grow or shrink at runtime) [](https://docs.solidity.org/en/latest/types.html).[1]

## Fixed-size arrays

Syntax: `T[k]` where `T` is the element type and `k` is the size. The size is part of the type itself and can't change [](https://docs.solidity.org/en/latest/types.html).[1]

```solidity
contract FixedArray {
    uint[5] public scores;  // exactly 5 uints, all initialized to 0

    function setScore(uint index, uint value) public {
        require(index < 5, "index out of bounds");
        scores[index] = value;
    }

    function getScore(uint index) public view returns (uint) {
        return scores[index];
    }

    function getLength() public pure returns (uint) {
        return 5;  // fixed, known at compile time
    }
}
```

**Key points:**
- Size is fixed forever — you can't `push` or `pop`.
- You can only overwrite existing slots: `scores[2] = 99`.
- Accessing an index ≥ 5 causes an error (array out of bounds).
- More gas-efficient since the layout is known at compile time [](https://mudassirkhan.me/blog/solidity-arrays-from-scratch).[2]

## Dynamic arrays

Syntax: `T[]` — empty brackets mean the size isn't fixed. It starts at length 0 and grows with `.push()` [](https://docs.solidity.org/en/latest/types.html).[1]

```solidity
contract DynamicArray {
    uint[] public numbers;

    function addNumber(uint value) public {
        numbers.push(value);  // adds to the end, length grows by 1
    }

    function removeLast() public {
        numbers.pop();  // removes last element, length shrinks by 1
    }

    function getLength() public view returns (uint) {
        return numbers.length;
    }

    function getAll() public view returns (uint[] memory) {
        return numbers;
    }
}
```

**Key points:**
- Starts empty (length 0).
- `.push(value)` adds to the end and increases length by 1.
- `.pop()` removes the last element and decreases length by 1.
- `.length` is dynamic — changes as you push/pop.
- Can grow indefinitely (limited by gas) [](https://www.alchemy.com/docs/how-solidity-arrays-work).[3]

## Side-by-side comparison

| Feature | Fixed (`T[k]`) | Dynamic (`T[]`) |
|---|---|---|
| Size | Set at compile time | Changes at runtime |
| `.push()` | Not available | Available |
| `.pop()` | Not available | Available |
| `.length` | Constant (= k) | Changes with push/pop |
| Gas efficiency | Better (no resizing) | Worse (resizing costs gas) |
| Use case | Known, fixed count | Unknown/growing count |

## Memory vs Storage arrays

An important distinction — arrays can live in different "locations":

- **Storage** — permanent, on-chain, expensive to write.
- **Memory** — temporary, exists only during function execution.
- **Calldata** — read-only input data for external functions.

```solidity
contract ArrayLocations {
    uint[] public storageArr;  // storage (permanent)

    function example() public pure returns (uint) {
        uint[3] memory fixedMem = [uint(1), 2, 3];  // memory (temporary)
        uint[] memory dynMem = new uint[](5);        // memory dynamic, fixed at 5
        return fixedMem[0] + dynMem[4];
    }
}
```

Note: a **memory** dynamic array must have its length set at creation (`new uint[](5)`), so it's actually fixed once allocated — only **storage** dynamic arrays can truly grow/shrink with push/pop [](https://mudassirkhan.me/blog/solidity-arrays).[4]

## Deleting elements

`delete arr[i]` doesn't remove the element — it resets it to the default value (0 for uint) but keeps the slot. To actually remove, you'd use `.pop()` (only removes last) or manually shift elements.

```solidity
contract DeleteExample {
    uint[] public arr = [1, 2, 3, 4, 5];

    function deleteAt(uint index) public {
        delete arr[index];  // arr[index] becomes 0, length stays 5
    }

    function removeLast() public {
        arr.pop();  // length becomes 4, last element gone
    }
}
```

## When to use which

- **Fixed** — when you know the exact count upfront (e.g. 5 team members, 12 months).
- **Dynamic** — when the count is unknown or changes (e.g. list of users, bids in an auction).

## Interview one-liner

"Fixed arrays have a compile-time size (`uint[5]`) and can't grow or shrink; dynamic arrays (`uint[]`) start empty and grow with `.push()` / shrink with `.pop()`. Fixed arrays are more gas-efficient since the layout is known upfront, while dynamic arrays offer flexibility at the cost of resizing gas."

Citations:
[1] Types — Solidity 0.8.39-develop documentation https://docs.solidity.org/en/latest/types.html
[2] Solidity Arrays From Scratch: A Practical Guide https://mudassirkhan.me/blog/solidity-arrays-from-scratch
[3] How do Solidity arrays work? - Alchemy Docs https://www.alchemy.com/docs/how-solidity-arrays-work
[4] Solidity Arrays — push, pop, length, and Safe Iteration https://mudassirkhan.me/blog/solidity-arrays
[5] Solidity - Arrays - GeeksforGeeks https://www.geeksforgeeks.org/solidity/solidity-arrays/
[6] Dynamic Arrays and its Operations in Solidity - GeeksforGeeks https://www.geeksforgeeks.org/solidity/dynamic-arrays-and-its-operations-in-solidity/
[7] Array | Solidity by Example | 0.8.26 https://solidity-by-example.org/array/
[8] Solidity - Reference Types - GeeksforGeeks https://www.geeksforgeeks.org/solidity/solidity-reference-types/
[9] learn-solidity/Tutorials/Beginners/17-arrays-in-solidity.md at main · raj-pranav/learn-solidity https://github.com/raj-pranav/learn-solidity/blob/main/Tutorials/Beginners/17-arrays-in-solidity.md
[10] Arrays (fixed and dynamic) in Blockchain / Solidity https://leyaa.ai/codefly/learn/blockchain/part-1/blockchain-arrays-fixed-and-dynamic
[11] Arrays in Solidity - ethereum-blockchain-developer.com https://www.ethereum-blockchain-developer.com/solidity-examples/arrays
[12] Miscellaneous — Solidity 0.6.2 documentation https://docs.soliditylang.org/en/v0.6.2/miscellaneous.html
[13] Types — Solidity 0.8.31-develop documentation https://docs.soliditylang.org/en/latest/types.html?highlight=bytes
[14] Fixed-Size/Dynamic Arrays - Conflux Documentation https://doc.confluxnetwork.org/docs/general/build/smart-contracts/gas-optimization/fixedSize/
[15] Types https://docs.soliditylang.org/en/v0.8.35-pre.1/types.html
