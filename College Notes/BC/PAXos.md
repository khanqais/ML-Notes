Explain PAXOS in detail with example 

# Paxos Explained, with an Example

Paxos is a consensus algorithm that lets a group of unreliable nodes agree on a single value. It is safe even if nodes crash or messages are delayed. It works in two phases, Prepare/Promise and Accept/Accepted, and a value is chosen once a majority of acceptors accept it.

## Roles

- **Proposer:** Suggests a value and drives the protocol.
- **Acceptor:** Votes on proposals. A majority (quorum) of acceptors decides the outcome.
- **Learner:** Finds out which value was chosen.

In most deployments each node plays all three roles, which reduces message complexity without hurting correctness.[1]

## The Two Phases

**Phase 1: Prepare and Promise**
- The proposer picks a unique, increasing proposal number `n` and sends `Prepare(n)` to a majority of acceptors.[2]
- If `n` is higher than any number the acceptor has already responded to, it replies with a Promise. The promise says it will never again accept a proposal numbered below `n`. The reply also includes the highest-numbered proposal it has already accepted, if any.[2][3]

**Phase 2: Accept and Accepted**
- After getting promises from a majority, the proposer picks a value. If any promise reported an already-accepted value, it must use the value with the highest accepted number. If none did, it can use its own value.[2][4]
- It sends `Accept(n, v)`. An acceptor accepts it unless it has already promised a higher number.[1]
- When a majority of acceptors have accepted `(n, v)`, `v` is chosen permanently.[2]

The rule in Phase 2 is what makes Paxos safe. A later proposer is forced to adopt any value that might already have been chosen, rather than overwrite it.

## Example 1: Simple Case (No Conflict)

Three acceptors (A, B, C) and one proposer (P1) who wants to store "potato = 12".

1. P1 sends `Prepare(1)` to A, B and C.
2. All three reply with a Promise and report no previously accepted value.
3. P1 sends `Accept(1, 12)`.
4. All three accept. A majority has accepted, so 12 is chosen.
5. The acceptors tell the learners that 12 was chosen.[5]

## Example 2: Competing Proposers

This example shows why the safety rule matters. I built it from the protocol rules above. The sources don't contain this exact scenario.

Acceptors are A, B and C. P1 wants the value **12**, and P2 wants **23**.

1. P1 sends `Prepare(1)`. A, B and C promise. None has accepted anything yet.
2. P1 sends `Accept(1, 12)`. A and B accept it. C doesn't receive it because of a network delay. A majority (A and B) has accepted, so **12 is now chosen**, though P1 may crash before telling anyone.
3. P2 now starts with `Prepare(2)` to B and C. Both promise, because 2 is higher than 1.
4. B's promise includes its accepted proposal `(1, 12)`. C's promise has nothing.
5. P2 must use the value from the highest-numbered accepted proposal it saw. It abandons its own 23 and sends `Accept(2, 12)`.
6. B and C accept. Now all nodes agree on **12**.

Any two majorities overlap in at least one acceptor. So a new proposer's Phase 1 quorum always contains at least one acceptor that knows about a chosen value, and the proposer can't pick a conflicting one.

## Failure and Liveness

- If an acceptor has promised a higher number, it rejects older Accept requests. The old proposer then retries with a higher number.
- **Dueling proposers** can livelock. P1 and P2 keep preempting each other with higher numbers, and nobody finishes. The fix is a distinguished proposer (a leader), which is what Multi-Paxos does.[6]
- Acceptors must write promises and accepted values to disk (persistent storage) before replying, or a crash could break safety.[3]

## Basic Paxos vs Multi-Paxos vs Raft

Basic Paxos agrees on one value. Real systems need a log of many values. Multi-Paxos runs Paxos once per log slot. After an initial leader election, a stable leader can propose many values with one quorum round trip each, instead of two.[7]

| Aspect | Paxos | Raft |
|---|---|---|
| Leader | Not explicit in basic Paxos; Multi-Paxos adds one [8] | Always explicit [8] |
| Phases per decision | 2 (1 once a leader is stable) [8] | 1 once the leader is stable [8] |
| Specification | Practical pieces under-specified, easy to get subtly wrong [9] | Explicit algorithms for election, replication and membership [7] |
| Correctness and speed | Same in the common case [9] | Same in the common case [9] |

Raft is basically Multi-Paxos with the details spelled out, which is why it's easier to teach and implement.

