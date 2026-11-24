 

# Raft Consensus Algorithm: Detailed Notes

Raft keeps several servers in agreement by making them agree on one ordered log of operations. If every node has the same log and replays it, every node ends up with the same data state.

## 1. The Problem

- Storing data in one file on one server is simple. If the server dies, for example in a data-center flood, you lose the data.
- The fix is to replicate the data across several servers.
- Replication creates a new problem: inconsistency. One server may say the potato price is 12, another 23, and a third 10. You need to know which one to trust.
- Consensus means getting all nodes to agree on one state of the data. Raft is an algorithm that achieves this.

## 2. The Log (Source of Truth)

- Every node stores a log that records each operation: update, add, remove.
- Replaying the log from the start gives the current state of the data.
- If all nodes agree on the log, they automatically agree on the data state.
- Each log entry has an **index** (its position) and a **term** number (the leader's term when it was created).

## 3. Cluster Roles and Server States

Each node is in one of three states:

| State | Role |
|---|---|
| Leader | Handles client requests and replicates log entries to the others. |
| Follower | Passive. It accepts entries from the leader and votes in elections. |
| Candidate | A follower that has stopped hearing from the leader and is running for leader. |

- The cluster has one leader (for example, 1 leader and 4 followers in a 5-node cluster). Clients talk to the leader.
- The leader sends periodic **heartbeats** to say "I am alive."

## 4. Leader Election

1. A follower that gets no heartbeat within the timeout (for example, 100 ms) assumes the leader is dead. It becomes a **candidate** and starts an election.
2. Followers vote for a candidate, and the candidate that wins becomes the new leader.
3. A **term** is the period during which one leader serves. When a new leader is elected, a new term starts (term 1 becomes term 2).

### Randomized timeouts
If every node had the same timeout, they would all become candidates at once and vote for themselves, so no one would win. Each node therefore gets a **random** election timeout. Usually one node times out first and wins. Sometimes several become candidates, and the election is retried.

### Vote restriction (log freshness)
- A longer log means fresher data.
- A node will **not** vote for a candidate whose log is shorter or less up to date than its own.
- This stops nodes with stale data from becoming leader.

## 5. Log Replication (Two-Phase, Commit-Like)

1. The client sends a command to the leader.
2. The leader creates a log entry with an index and a term.
3. The leader sends it to the followers (an AppendEntries request).
4. Followers **append** the entry to their logs. The entry is appended, **not committed**. It isn't final, because a follower can't know whether the other nodes also received it.
5. The leader waits for responses. When a **majority (more than half)** of nodes have appended the entry, it is safely replicated.
6. The leader **commits** the entry and applies it.
7. The leader tells the followers to commit, and they do.

Why majority? Any two majorities overlap in at least one node, so a committed entry survives leader changes and failures. A 5-node cluster tolerates 2 failures.

## 6. Handling a Lagging or Recovered Follower

**Scenario:** A follower goes down, misses entries 4 and 5, then comes back. The leader wants to send entry 6, but the follower only has entries 1–3. Appending entry 6 would leave gaps, and the log must stay in order.

**Solution: the consistency check**
- Each AppendEntries request carries the **index and term of the previous entry**.
- If the follower doesn't have that previous entry, it returns an **error**.
- The leader then **decrements the index** and retries with an earlier entry and its previous entry.
- This repeats until the leader finds the point where the logs match.
- From there the leader sends all the missing entries in order, and the follower catches up.

## 7. Why Raft Matters

- Raft gives strong consistency by keeping the log identical across nodes.
- Databases such as **YugabyteDB** and **CockroachDB** use Raft or variations of it.
- Raft is not limited to databases. The same idea works with any **finite state machine (FSM)**. If all nodes run the same FSM and apply the same operations in the same order (1, 2, 3, 4...), they all end in the same state. This is called a *replicated state machine*.

## 8. Quick Revision Cheat Sheet

- **Problem:** Replication causes inconsistency, so we need consensus.
- **Log:** An ordered list of operations. The same log gives the same state.
- **Roles:** Leader, follower, candidate.
- **Heartbeat:** The leader's "I'm alive" signal. If it stops, an election starts.
- **Term:** The leader's tenure number, which increases with each election.
- **Random timeouts:** Prevent split votes.
- **Vote rule:** Don't vote for a candidate with a staler log.
- **Replication:** Append, then wait for a majority, then commit.
- **Repair:** Check the previous index and term, and walk back until the logs match.
- **Generalization:** Raft works with any FSM, not just databases.

## 9. Interview-Ready Answer (30 seconds)

"Raft is a consensus algorithm that keeps replicated nodes consistent by agreeing on an ordered log. One node is elected leader using randomized election timeouts and terms. The leader replicates entries to followers and commits them once a majority has appended them. Followers with missing entries are repaired through a previous-index and previous-term check. Because every node applies the same log, they reach the same state. CockroachDB and YugabyteDB use it."

**Note:** The video simplifies a few points. In the full Raft paper, a leader also only commits entries from its *current* term by counting replicas, and followers learn the commit index from later AppendEntries messages or heartbeats. These are useful details to mention in interviews.

