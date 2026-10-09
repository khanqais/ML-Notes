#  DSA Revision Platform — Placement Interview Cheat Sheet

> [!abstract] Project summary
> A DSA revision platform with a spaced-repetition algorithm (10% confidence decay, capped at 50%), an advanced code-judge pipeline with C++ signature parsing, auto-generated testing harnesses covering 17+ edge cases, and a problem-management system across 50+ DSA categories with full revision tracking.

---

## 📌 1. Project Overview Questions

### Q1. Walk me through your project in 2–3 minutes

> [!tip] Expected answer framework
> - **Problem:** Developers forget DSA problems after solving them. There is no structured revision system.
> - **Solution:** I built a platform where users solve problems, rate their confidence from 1–10, and the system reschedules revision using spaced repetition.
> - **Key features:**
>   1. **Spaced-repetition engine:** 10% decay, capped at 50%.
>   2. **Automatic code judge:** C++ signature parsing and harness generation.
>   3. **Problem management:** 50+ categories, with saved code, approach, and intuition.
> - **Impact:** Efficient revision scheduling and automatic coverage of 17+ edge-case scenarios.

### Q2. What problem does this project solve?

It addresses the forgetting-curve problem in DSA preparation. Without a system, users may forget a problem soon after solving it. This platform resurfaces problems when they are due for review, based on the user's confidence and revision history.

### Q3. What tech stack did you use, and why?

Be ready to justify every choice:

- **Frontend (React/Next.js?):** Component-driven UI for problem cards.
- **Backend (Node/Express or similar):** REST APIs for problem CRUD and the judge pipeline.
- **Database:** Explain why SQL or NoSQL fits problem and revision storage.
- **Judge sandbox:** Explain why C++ is supported and how execution is isolated.

> [!warning] Personalize before the interview
> Replace the question marks and alternatives with the exact stack and implementation you actually used. Do not claim features or impact you did not build or measure.

---

## 🧠 2. Spaced-Repetition Algorithm

### Q4. Explain your spaced-repetition algorithm in detail

```text
Next review date = current date + interval
Interval depends on confidence score (1–10)

Confidence decay:
  new_confidence = max(confidence × 0.90, confidence − cap)
  cap = 50% of original score
```

- Higher confidence means a longer interval before the next review.
- Lower confidence means the problem should be reviewed sooner.
- A 10% decay per missed or poor review is intended to model forgetting and increase review urgency.

> [!warning] Clarify the formula
> The expression above needs a precise definition of `cap` and when decay is applied. If the intent is “confidence cannot fall below 50% of its original/peak value,” state that explicitly and use a formula that enforces that floor.

### Q5. Why 10% confidence decay?

The proposed rationale is that it balances urgency against review overload. With multiplicative 10% decay, after seven decay events the score is about 48% of its original value: `0.9⁷ ≈ 0.478`. Be clear about whether this value was empirically tuned or chosen as an initial heuristic.

### Q6. Why have a 50% decay floor?

Without a floor, repeated decay could reduce confidence close to zero, even for a problem the user once knew well. A floor at half the peak score prevents the scheduler from treating that problem as entirely unfamiliar. Explain whether the floor is based on the original score, the highest score, or another stored value.

### Q7. How do you prioritize problems?

```text
urgency = (days_overdue × weight) + (1 / confidence)
```

Problems with low confidence and more overdue days receive higher urgency. If this is the actual ordering rule, describe whether larger or smaller urgency values are shown first. A priority queue or database ordering can retrieve the highest-priority items.

### Q8. How does this differ from Anki's SM-2 algorithm?

- SM-2 adjusts intervals using an ease factor and the user's repetition quality.
- This design uses a simpler confidence score and decay model tailored to DSA revision.
- SM-2 tracks repetition history and adjusts ease; this design emphasizes a user-facing confidence rating.
- Describe this as a simpler, product-specific heuristic—not as inherently more accurate.

### Q9. What data structure would you use for the review scheduler?

A min-heap keyed by `next_review_date` can retrieve the earliest due problem efficiently:

- Insert or update: `O(log n)`.
- Retrieve and remove earliest item: `O(log n)`.
- For a distributed implementation, a Redis sorted set can order items by score, such as a timestamp.

### Q10. How would you scale the scheduler to one million users?

- Partition or shard scheduler data by `user_id` where appropriate.
- Store due timestamps in a sorted structure such as Redis ZSET, while considering key cardinality and operational limits.
- Have background workers claim due work in bounded batches rather than scanning every user's full schedule.
- Send notifications asynchronously through a queue such as Kafka or SQS.
- Make jobs idempotent and monitor queue lag, retries, and duplicate delivery.

---

## ⚙️ 3. Code-Judge Pipeline

### Q11. Explain the code-judge pipeline end to end

```text
User submits C++ code
    ↓
Signature parser extracts function name, parameter types, and return type
    ↓
Harness generator wraps the solution with input/output handling
    ↓
Compilation with g++ and configured flags
    ↓
Execution against server-side test cases in an isolated environment
    ↓
Output comparison → verdict (AC / WA / TLE / CE)
```

### Q12. How does the C++ signature parser work?

It extracts the function name, parameter types, and return type. For example:

```cpp
// Input signature:
vector<int> twoSum(vector<int>& nums, int target)

// Parsed representation:
{
  name: "twoSum",
  params: [("nums", "vector<int>&"), ("target", "int")],
  return: "vector<int>"
}
```

A regex can handle simple signatures, but nested templates, qualifiers, default arguments, and overloads make robust parsing difficult. If the implementation uses regex, describe its supported subset honestly; a full C++ parser generally needs compiler tooling or a carefully defined grammar.

### Q13. What is an automatically generated testing harness?

A generated wrapper `main()` reads inputs in the required types, calls the user's function, and prints results in a format the judge can compare.

```cpp
#include <bits/stdc++.h>
using namespace std;

// User's solution is inserted here.

int main() {
    int n;
    cin >> n;

    vector<int> nums(n);
    for (auto& x : nums) cin >> x;

    int target;
    cin >> target;

    auto result = twoSum(nums, target);
    for (auto x : result) cout << x << ' ';
    return 0;
}
```

### Q14. How do you handle 17+ edge cases?

Explain that test cases should be derived from each problem's specification and constraints. Example categories include:

1. Empty input (`n = 0`), where allowed.
2. Single-element array.
3. All elements equal.
4. Sorted ascending or descending input.
5. Negative numbers.
6. Integer boundary values (`INT_MAX`, `INT_MIN`).
7. Duplicate values.
8. Large input for performance testing.
9. Strings with spaces or special characters.
10. Empty strings, where allowed.
11. Disconnected graph components.
12. Graph self-loops, where allowed.
13. Single-node tree.
14. Skewed tree (all-left or all-right).
15. One-row or one-column matrix.
16. Target absent from the input.
17. Multiple valid answers, such as in Two Sum.
18. Exact constraint boundaries, such as `n = 10⁵`.

> [!warning] Avoid overclaiming
> “17+ scenarios” should mean scenarios your system actually generates or tests—not a universal set applied to every problem. Some listed cases are invalid for particular constraints.

### Q15. How do you prevent malicious code execution?

A production judge should use defense in depth:

- Run each submission in a strongly isolated sandbox or container; containers alone are not a complete security boundary.
- Apply CPU, memory, process-count, file-size, and wall-clock limits.
- Disable network access and restrict filesystem access.
- Use syscall filtering and least-privilege execution where supported.
- Terminate timed-out processes and clean up their resources.
- Keep the judge host and runtime images patched; avoid exposing secrets to the sandbox.

### Q16. What is the time complexity of the judge pipeline?

- Signature parsing: typically `O(L)`, where `L` is source length, for a single-pass parser; regex behavior depends on the pattern and input.
- Compilation: depends on source size, compiler, libraries, and environment; it is not a constant-time operation.
- Execution: approximately the sum of runtime over all test cases, subject to process-startup and sandbox overhead.
- Output comparison: `O(M)` for output length `M`, assuming a linear comparison.

### Q17. How do you handle TLE?

Run the program as a child process under an external supervisor. Enforce CPU and wall-clock limits, detect expiry, terminate the process, and report TLE. A production implementation should also clean up descendants and temporary files; do not rely on a timeout mechanism that can be bypassed by child processes.

---

## 🗂️ 4. Problem Management System

### Q18. Describe the data model

```text
problems
  id, title, difficulty, category_id, constraints, description

user_solutions
  id, user_id, problem_id, code, approach, intuition,
  confidence (1–10), last_reviewed, next_review_date

revision_history
  id, user_solution_id, reviewed_at, confidence_before, confidence_after

categories
  id, name, parent_id
```

The `parent_id` field supports a hierarchy, for example `Graph → BFS → Multi-source BFS`.

### Q19. How did you design 50+ DSA categories?

Use a hierarchy of broad topics and specific patterns:

```text
Arrays & Strings
  ├── Two Pointers
  ├── Sliding Window
  └── Prefix Sum

Graphs
  ├── BFS / Multi-source BFS
  ├── DFS / Cycle Detection
  ├── Shortest Path (Dijkstra, Bellman–Ford)
  ├── Topological Sort
  └── Union-Find / DSU

Trees
  ├── Binary Search Tree
  ├── Segment Tree
  └── Trie

Dynamic Programming
  ├── 0/1 Knapsack
  ├── LCS / LIS
  ├── Interval DP
  └── Digit DP
```

### Q20. How do you store and retrieve approach and intuition fields?

- Store text in a `TEXT` column or an equivalent document field.
- Use a composite uniqueness constraint or index on `(user_id, problem_id)` for a user's solution to a problem.
- Add full-text search only if users need to search notes; PostgreSQL `tsvector` is one option.
- Choose storage based on the actual access patterns rather than adding multiple databases without a demonstrated need.

### Q21. How is confidence used in scheduling?

An example mapping is:

| Confidence | Example interval |
|---:|---:|
| 10 | 30 days |
| 8 | 14 days |
| 6 | 7 days |
| 4 | 3 days |
| 2 | 1 day |
| 1 | Today |

The confidence decay can shorten the interval or raise urgency. Explain how this mapping combines with due dates and review outcomes in the actual implementation.

---

## 🔧 5. System Design and Scale

### Q22. How would you design the system for 10 million users?

```text
Client → CDN → Load balancer → API gateway
                                  ↓
                              Auth service
                                  ↓
          ┌───────────────────────┼──────────────────────┐
          ↓                       ↓                      ↓
   Problem service          Judge service         Scheduler service
          ↓                       ↓                      ↓
     PostgreSQL          Isolated workers       Due-work store
                                  ↓                      ↓
                           Object storage       Queue → notifications
```

Discuss service boundaries, database access patterns, queue semantics, isolation, observability, and failure recovery. This is a proposed scale architecture; distinguish it from what the project currently runs.

### Q23. How would you make the judge horizontally scalable?

- Put submissions on a durable job queue.
- Run multiple isolated judge workers that claim jobs and publish results.
- Scale worker count based on queue depth and resource utilization.
- Keep workers stateless where possible, and store submission status/results in durable storage.
- Make retries safe and prevent a duplicate job from producing conflicting results.

### Q24. What database would you choose for revision history, and why?

A practical starting point is PostgreSQL for users, problems, solutions, categories, and revision events because these are structured and benefit from transactions and relational constraints. Add Redis for ephemeral scheduling or caching only if access patterns justify it. Add an analytics store or document database only when measured requirements call for one; avoid introducing several databases solely for architectural appearance.

### Q25. How would you implement notifications for due reviews?

- Identify due reviews in bounded batches using an indexed timestamp or sorted set.
- Enqueue notification jobs rather than sending messages inside the scheduler transaction.
- Have workers send email or push notifications through a provider.
- Use idempotency keys, retry policies, and a dead-letter path to manage failures and duplicates.

---

## 💻 6. DSA and CS Fundamentals

### Q26. What is the Ebbinghaus forgetting curve, and how did you use it?

The forgetting-curve idea describes how recall can decline over time without reinforcement. A commonly used illustrative model is:

```text
R = e^(−t / S)
```

Here, `R` is a retention measure, `t` is elapsed time, and `S` represents memory stability. The platform's confidence decay is a simplified scheduling heuristic inspired by forgetting and review—not a direct measurement or validated implementation of this psychological model.

### Q27. Implement a min-heap for the scheduler

```cpp
priority_queue<
    pair<long long, int>,
    vector<pair<long long, int>>,
    greater<>
> pq;

// {next_review_timestamp, problem_id}
pq.push({timestamp, problemId});

auto [time, problemId] = pq.top();
pq.pop();
```

### Q28. How would you parse a C++ function signature using regex?

```python
import re

pattern = r'([\w:<>&*, ]+)\s+(\w+)\s*\(([^)]*)\)'
match = re.search(pattern, signature)
```

This is only a starting point. It may fail on nested parentheses, function pointers, complex templates, attributes, namespaces, and other valid C++ syntax. State the supported grammar and test its limits.

### Q29. How does Union-Find (DSU) work?

```cpp
int parent[N], rankValue[N];

int find(int x) {
    if (parent[x] != x) parent[x] = find(parent[x]);
    return parent[x];
}

void unite(int a, int b) {
    a = find(a);
    b = find(b);
    if (a == b) return;

    if (rankValue[a] < rankValue[b]) swap(a, b);
    parent[b] = a;
    if (rankValue[a] == rankValue[b]) rankValue[a]++;
}
```

With path compression and union by rank/size, operations run in amortized `O(α(n))`, where `α` is the inverse Ackermann function.

### Q30. What is a segment tree, and when would a user revise it?

A segment tree supports range queries and updates in `O(log n)` for common operations such as range sum, minimum, or maximum. In this project, a user might revisit segment-tree problems when their review schedule marks them due; the data structure itself is not part of the scheduler unless the implementation actually uses it.

---

## 🧩 7. OOP and Design Patterns

### Q31. What design patterns did you use?

Only claim patterns that appear in the code. Possible designs include:

- **Strategy:** Interchangeable scheduling policies, such as a custom decay strategy and SM-2-style strategy.
- **Factory:** Select a harness generator based on a supported signature or problem type.
- **Observer / event-driven design:** A review event triggers schedule recalculation.
- **Repository:** Isolate persistence operations for problems and user solutions.

### Q32. How did you model the Problem entity?

```text
Problem
  ├── id, title, difficulty, category
  ├── constraints: Constraints { min_n, max_n, value_range }
  └── getEdgeCases(): List<TestCase>

UserSolution
  ├── problem_id, user_id
  ├── code, approach, intuition
  ├── confidence: float (1.0–10.0)
  ├── revisionHistory: List<RevisionEntry>
  └── getNextReviewDate(): Date
```

In a relational implementation, `UserSolution` is usually a separate entity referencing `Problem`, rather than literally inheriting from it. Explain the design that matches your codebase.

---

## 🔥 8. Behavioral and Impact Questions

### Q33. What was the hardest technical challenge?

> “The hardest part was handling C++ function signatures for harness generation. C++ types can include nested templates, references, and qualifiers—for example, `vector<pair<int, int>>&`. I had to define the subset of signatures the platform supports and make parsing and harness generation reliable for that subset. I would explain the specific parser approach and tricky cases I actually implemented.”

> [!warning] Make this your own
> The original suggested story mentions trying five regex patterns and then using recursive descent. Use those details only if they accurately describe your work.

### Q34. What would you improve with more time?

1. **Adaptive scheduling:** Evaluate scheduling policies using review outcomes before considering a learned model.
2. **More languages:** Add Python and Java with language-specific harnesses and sandboxes.
3. **Collaboration:** Let users share notes and approaches.
4. **Analytics:** Show weak topics and confidence trends.
5. **Judge hardening:** Improve sandboxing, observability, and test coverage.

### Q35. How do you measure success?

- Review-schedule adherence and return rate.
- Confidence changes over time, interpreted alongside review outcomes.
- Problems reviewed or mastered, with a clear definition of “mastered.”
- Judge correctness against known expected outputs.
- Submission latency, queue wait time, and failure rate.

---

## ⚡ 9. Quick-Fire Questions

| Question | Sharp answer |
|---|---|
| Why C++ for the judge? | It is widely used in competitive programming and provides fast execution. The judge still needs safe isolation and a clearly defined supported input format. |
| How do you handle infinite loops? | Enforce CPU and wall-clock limits in a supervised, isolated process; terminate it and report TLE. |
| What if two problems have the same urgency? | Use a deterministic secondary key, such as next-review timestamp and problem ID. |
| Can a user override confidence? | If implemented, let the user rate confidence on a 1–10 scale after review and recalculate the schedule. |
| How do you prevent test-case leakage? | Keep hidden test cases server-side and avoid returning them in client responses or logs. |
| What is your API rate-limit strategy? | Use a per-user and per-IP policy, such as a token bucket; tune limits to actual workload and abuse risk. |
| How do you handle concurrent submissions? | Give each run an isolated workspace and process/container, with resource limits and unique job IDs. |
| What happens on a compilation error? | Return a CE verdict with a safely truncated, sanitized compiler diagnostic. |

---

## 📊 10. Numbers to Remember

| Metric | Value / note |
|---|---|
| Confidence decay rate | 10% per review cycle (confirm actual behavior) |
| Decay floor | 50% (specify whether based on original or peak confidence) |
| Confidence scale | 1–10 |
| Edge-case scenarios | 17+ (describe what is actually generated/tested) |
| DSA categories | 50+ |
| Judge timeout | Add the real configured value; do not use an example as a claimed metric |

---

## 🎤 Closing One-Liners

- **Algorithm:** “I built a confidence-based revision scheduler inspired by spaced repetition, with a decay heuristic that brings weaker or overdue problems back sooner.”
- **Judge:** “I built a C++ judging pipeline that parses supported function signatures, generates a harness, and evaluates submissions against test cases.”
- **Scale:** “The judge can be scaled with queued submissions and multiple isolated workers; the architecture separates execution from the API request path.”
- **System:** “It is like a revision planner for competitive programming, combining problem tracking with an integrated code judge.”

> [!success] Final interview prep checklist
> - Replace every placeholder with your actual implementation details.
> - Separate what you built from what you would build at scale.
> - Be ready to explain the decay formula, floor, interval mapping, and tie-breaking precisely.
> - Never claim production-grade sandboxing, measured impact, or automatic edge-case generation unless you implemented and verified it.
