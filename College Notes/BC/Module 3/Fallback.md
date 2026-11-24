Explain Fallback function in Solidity with an example."

## What is a fallback function?

A `fallback()` function is a special catch-all that runs when a call to your contract doesn't match any defined function, or when ETH is sent with data attached but no `receive()` exists [](https://solidity-by-example.org/fallback/). It's the "default handler" when nothing else matches.[1]

value is the ETH amount in wei sent with a transaction, accessible as msg.value. data is the calldata payload — the function selector plus ABI-encoded arguments — accessible as msg.data or msg.sig. A plain ETH transfer has value but empty data; a contract call can have both. Token transfers like USDT use data for the call, not value, since no ETH actually moves
## When does it run?

The logic is a decision tree based on whether calldata (the raw bytes sent with a call) is empty or not [](https://0xmowgli.com/blog/receive-fallback):[2]

| Calldata | Contract has | What runs |
|---|---|---|
| Empty | `receive()` | `receive()` |
| Empty | No `receive`, payable `fallback()` | `fallback()` |
| Empty, with ETH | No `receive`, non-payable `fallback()` | Revert |
| Non-empty (unknown function) | Any | `fallback()` |

The key distinction: `receive()` handles plain ETH transfers with empty calldata, while `fallback()` handles everything else — unknown function calls, or ETH with calldata attached [](https://www.soliditylang.org/blog/2020/03/23/fallback-receive-split/).[3]
![[Screenshot_2026-11-23-17-08-56-101.png]]

## Syntax rules

- Written without the `function` keyword: `fallback() external payable { ... }`
- Must be `external`; can optionally be `payable` if it should accept ETH [](https://www.soliditylang.org/blog/2020/03/23/fallback-receive-split/).[3]
- No arguments, no return value in the basic form. An advanced form takes `bytes calldata input` and returns `bytes memory` for proxy/upgradeable patterns [](https://andreyobruchkov1996.substack.com/p/solidity-execution-overview-payable).[4]
- Only one `fallback()` per contract, and only one `receive()` per contract [](https://arxiv.org/pdf/2203.14850.pdf).[5]

## Example: donation contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract DonationBox {
    uint public totalReceived;
    event Received(address sender, uint amount, bool usedFallback);

    receive() external payable {
        // Plain ETH transfer, no calldata
        totalReceived += msg.value;
        emit Received(msg.sender, msg.value, false);
    }

    fallback() external payable {
        // Unknown function call, or ETH with calldata
        totalReceived += msg.value;
        emit Received(msg.sender, msg.value, true);
    }
}
```

**How it behaves:**

- Someone calls the contract with no data (e.g. a plain ETH send) → `receive()` runs, `usedFallback` is `false`.
- Someone calls a function name that doesn't exist, like `donate()` which isn't defined → `fallback()` runs, `usedFallback` is `true`.
- Someone sends ETH along with calldata → `fallback()` runs, since calldata isn't empty [](https://solidity-by-example.org/fallback/).[1]

## Why it exists

Before Solidity 0.6, there was only one unnamed `function() payable` that handled both cases. The 0.6.x update split it into `receive()` and `fallback()` so developers could clearly separate "just accept ETH" from "handle unknown calls" [](https://www.soliditylang.org/blog/2020/03/23/fallback-receive-split/). This matters for security: if your fallback is only meant to log ETH, you should check that calldata is empty, otherwise it also fires on accidental/unknown function calls [](https://blog.solidityscan.com/understanding-security-of-fallback-recieve-function-in-solidity-9d18c8cad337/).[3][6]

## Security notes

- If a contract has neither `receive()` nor a `fallback()`, any ETH sent to it is rejected and the transaction reverts [](https://docs.soliditylang.org/en/latest/security-considerations.html).[7]
- `transfer()` and `send()` forward only 2300 gas to the fallback, which is barely enough to log an event but not enough for complex logic. `call` forwards all available gas [](https://solidity-by-example.org/fallback/).[1]
- A common vulnerability is reentrancy: if `fallback()` sends ETH back to the caller, the caller's `receive()`/`fallback()` can re-enter your contract before state is updated. Always update state before sending ETH (checks-effects-interactions pattern).
- If you only want to accept plain ETH and reject unknown calls, define `receive()` but skip `fallback()` entirely [](https://ethereum.stackexchange.com/questions/142184/fallback-and-receive-functions-in-solidity).[8]

