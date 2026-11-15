# 3666. Minimum Operations to Equalize Binary String

> Source: https://algo.monster/liteproblems/3666

## Problem Statement

You are given a binary string `s` (a string consisting only of the characters `'0'` and `'1'`), and an integer `k`.

In a single operation, you must perform the following:

- Choose **exactly** `k` **different** indices from the string.
- For each chosen index, **flip** the character: every `'0'` becomes `'1'`, and every `'1'` becomes `'0'`.

It is important to note that you must select **exactly** `k` indices in every operation—no more and no fewer. The chosen indices must be distinct within a single operation.

Your goal is to make **all** characters in the string equal to `'1'`.

Return the **minimum** number of operations required to achieve this. If it is **impossible** to make every character `'1'`, return `-1`.

**Example walk-through of an operation:**

Suppose `s = "010"` and `k = 2`. You must pick exactly `2` indices to flip. For instance:

- Choosing indices `0` and `1` transforms `"010"` into `"100"` (index `0`: `'0'` → `'1'`, index `1`: `'1'` → `'0'`).
- Choosing indices `0` and `2` transforms `"010"` into `"111"` (index `0`: `'0'` → `'1'`, index `2`: `'0'` → `'1'`).

In this case, a single operation choosing indices `0` and `2` makes all characters `'1'`, so the answer would be `1`.

The challenge lies in deciding, at each step, which `k` indices to flip so that you reach the all-`'1'` string in as few operations as possible.

## Intuition

The first key observation is that the **specific positions** of the `'0'`s and `'1'`s don't actually matter—only **how many** `'0'`s remain in the string. Why? Because in each operation we are free to choose any `k` indices we like. So the only thing that defines a "state" is the current count of `'0'`s, which we call `cur`. Our starting state is the number of `'0'`s in `s`, and our goal is to reach the state where `cur == 0`.

Next, let's think about what a single operation does to `cur`. We pick exactly `k` indices to flip. Suppose `x` of those chosen indices are `'0'`s (they become `'1'`s, reducing the count of `'0'`s), and the remaining `k - x` are `'1'`s (they become `'0'`s, increasing the count). After the flip, the new number of `'0'`s is:

`cur + (k - x) - x = cur + k - 2x`

So from state `cur`, one operation can land us on any value of the form `cur + k - 2x`, depending on how many `'0'`s we choose to flip.

Now we figure out the valid range of `x`:

1. We can't flip more `'0'`s than exist, so `0 <= x <= min(cur, k)`.
2. We can't flip more `'1'`s than exist either. The number of `'1'`s flipped is `k - x`, and there are `n - cur` ones available, so `k - x <= n - cur`, which gives `x >= k - n + cur`.

Combining, `x` ranges over `[max(k - n + cur, 0), min(cur, k)]`. Plugging the extremes of `x` into `cur + k - 2x`, the reachable new counts form a **contiguous range** `[l, r]` where:

- `l = cur + k - 2 * min(cur, k)`
- `r = cur + k - 2 * max(k - n + cur, 0)`

A neat detail: since each step changes `cur` by `k - 2x`, the **parity** of `cur` only changes based on the parity of `k`. More precisely, `cur + k - 2x` always has the same parity as `cur + k`, so consecutive reachable states skip by `2` and share a fixed parity. This means within the range `[l, r]`, only the values matching the correct parity are actually reachable in one step.

Putting it all together, this becomes a **shortest-path search** on a graph where each node is a possible count of `'0'`s, and edges connect `cur` to every reachable count in `[l, r]`. Since every operation counts as one step (unweighted edges), **BFS** finds the minimum number of operations. The BFS level acts as our operation counter, and the moment we dequeue state `0`, we return that level.

The final piece is **efficiency**. The range `[l, r]` could be huge, and naively iterating over all values inside it on every step would be far too slow. To keep things fast, we maintain two **ordered sets**—one holding the still-unvisited even counts and one holding the unvisited odd counts. When processing a state, we only need to look at the set matching the required parity, jump to the first value `>= l`, and sweep forward while values stay `<= r`. Each count is added to a set once and removed once, so we never process the same state twice, keeping the whole search efficient.

**Learn more about [Breadth-First Search](/problems/bfs_intro), [Union Find](/problems/dsu_intro) and [Math](/problems/math-basics) patterns.**

## Solution Approach

We solve this with **BFS over an ordered set** of states. Each state is the current number of `'0'`s in the string, and we search for the shortest path (fewest operations) from the initial count to state `0`.

**Step 1: Set up the ordered sets by parity.**

We let `n` be the length of `s`. As established in the intuition, every operation changes the count of `'0'`s by `k - 2x`, so reachable states always preserve a fixed parity relationship. To exploit this, we keep two `SortedSet`s, one for even counts and one for odd counts:

```python
ts = [SortedSet() for _ in range(2)]
for i in range(n + 1):
    ts[i % 2].add(i)
```

This pre-loads every possible count of `'0'`s from `0` to `n` into its corresponding parity bucket. `ts[0]` holds all even counts, and `ts[1]` holds all odd counts. These sets represent the **unvisited** states.

**Step 2: Initialize the BFS from the starting state.**

We compute the initial number of `'0'`s, mark it as visited by removing it from its ordered set, and push it into the BFS queue:

```python
cnt0 = s.count('0')
ts[cnt0 % 2].remove(cnt0)
q = deque([cnt0])
ans = 0
```

Here `ans` counts the number of operations, which equals the BFS depth.

**Step 3: Process the BFS level by level.**

We run a standard level-order BFS. The `for _ in range(len(q))` loop processes all states at the current depth before incrementing `ans`, so `ans` correctly tracks the number of operations:

```python
while q:
    for _ in range(len(q)):
        cur = q.popleft()
        if cur == 0:
            return ans
        ...
    ans += 1
```

The moment we dequeue state `0`, we have reached the all-`'1'` string, so we return `ans` immediately.

**Step 4: Compute the reachable range and expand.**

For each dequeued state `cur`, we calculate the contiguous range `[l, r]` of new counts reachable in one operation, using the formulas derived earlier:

```python
l = cur + k - 2 * min(cur, k)
r = cur + k - 2 * max(k - n + cur, 0)
```

- `l` corresponds to flipping as many `'0'`s as possible (`x = min(cur, k)`).
- `r` corresponds to flipping as few `'0'`s as possible (`x = max(k - n + cur, 0)`).

All reachable states share the parity of `l`, so we only look in the matching set `t = ts[l % 2]`. We use `bisect_left` to jump straight to the first unvisited value `>= l`, then sweep forward while values stay `<= r`:

```python
t = ts[l % 2]
j = t.bisect_left(l)
while j < len(t) and t[j] <= r:
    q.append(t[j])
    t.remove(t[j])
```

Each reachable, still-unvisited state in `[l, r]` is added to the queue and **removed** from the ordered set so it is never revisited. Because we remove from the front of the set after each `bisect_left`, the index `j` continues to point at the next candidate.

**Step 5: Handle the impossible case.**

If the BFS drains the queue without ever reaching state `0`, no sequence of operations can make the string all `'1'`, so we return `-1`:

```python
return -1
```

**Why this is efficient.**

The naive approach of iterating over every integer in `[l, r]` on each step could be `O(n)` per state, leading to `O(n^2)` total work. By storing unvisited states in ordered sets and **removing** each one the first time it is reached, every state is inserted once and removed once across the entire search. The `bisect_left` lets us skip directly to the relevant portion of the range. This keeps the overall complexity to roughly `O(n log n)`, where `n` is the length of the string, since each of the `n + 1` possible states is processed at most once with logarithmic-time ordered-set operations.

## Example Walkthrough

Let's trace the solution approach with a small example.

**Input:** `s = "0011"`, `k = 3`

**Setup:**
- `n = 4` (length of the string).
- Number of `'0'`s in `s`: `cnt0 = 2`. This is our starting state.
- Goal: reach state `0` (zero `'0'`s remaining).

**Step 1: Build parity-based ordered sets.**

Possible counts of `'0'`s range from `0` to `4`. We split them by parity:
- `ts[0]` (even) = `{0, 2, 4}`
- `ts[1]` (odd) = `{1, 3}`

These represent all **unvisited** states.

**Step 2: Initialize BFS.**

- Remove the start state `2` from its set: `ts[0]` becomes `{0, 4}`.
- Queue: `q = [2]`
- `ans = 0`

Current sets:
- `ts[0] = {0, 4}`
- `ts[1] = {1, 3}`

**Step 3 & 4: BFS Level 0 (`ans = 0`).**

Process `cur = 2`. It is not `0`, so we compute the reachable range.

- `l = cur + k - 2 * min(cur, k) = 2 + 3 - 2 * min(2, 3) = 2 + 3 - 4 = 1`
- `r = cur + k - 2 * max(k - n + cur, 0) = 2 + 3 - 2 * max(3 - 4 + 2, 0) = 5 - 2 * max(1, 0) = 5 - 2 = 3`

So from state `2`, one operation reaches counts in `[1, 3]` sharing parity with `l = 1` (odd). We look only in `ts[1] = {1, 3}`.

Sweep from the first value `>= 1`:
- `1` is `<= 3` → add to queue, remove from set.
- `3` is `<= 3` → add to queue, remove from set.

Now:
- `ts[1] = {}` (empty)
- `q = [1, 3]`

After finishing this level, `ans` becomes `1`.

**Verification of meaning:** From state `2` (two `'0'`s), flipping `k = 3` indices, we can choose `x` zeros to flip where `x ∈ [1, 2]`:
- `x = 2`: flip both `'0'`s and one `'1'` → new count `= 2 + 3 - 4 = 1`.
- `x = 1`: flip one `'0'` and two `'1'`s → new count `= 2 + 3 - 2 = 3`.

This matches the reachable range `{1, 3}`.

**BFS Level 1 (`ans = 1`).**

Process the two states `1` and `3`.

**Process `cur = 1`:** Not `0`. Compute range:
- `l = 1 + 3 - 2 * min(1, 3) = 4 - 2 = 2`
- `r = 1 + 3 - 2 * max(3 - 4 + 1, 0) = 4 - 2 * max(0, 0) = 4`

Range `[2, 4]`, parity of `l = 2` is even → look in `ts[0] = {0, 4}`.

Sweep from first value `>= 2`:
- `4` is `<= 4` → add to queue, remove from set.
- (`0` is below `2`, so skipped.)

Now:
- `ts[0] = {0}`
- `q = [3, 4]`

**Process `cur = 3`:** Not `0`. Compute range:
- `l = 3 + 3 - 2 * min(3, 3) = 6 - 6 = 0`
- `r = 3 + 3 - 2 * max(3 - 4 + 3, 0) = 6 - 2 * max(2, 0) = 6 - 4 = 2`

Range `[0, 2]`, parity of `l = 0` is even → look in `ts[0] = {0}`.

Sweep from first value `>= 0`:
- `0` is `<= 2` → add to queue, remove from set.

Now:
- `ts[0] = {}` (empty)
- `q = [4, 0]`

After finishing this level, `ans` becomes `2`.

**BFS Level 2 (`ans = 2`).**

Process states `4` and `0`.

**Process `cur = 4`:** Not `0`. (Its expansion would continue, but it's irrelevant now.)

**Process `cur = 0`:** This is the goal state! We return `ans = 2`.

**Result:** `2` operations.

**Sanity check of the path:** `2 → 3 → 0`
- Start: `"0011"` (2 zeros). Flip indices `0`, `1`, `2`: `"1101"` (1 zero)... 

Let me use the path `2 → 1 → 0` instead for a cleaner illustration:
- Start `"0011"` (2 zeros). Flip one `'0'` and two `'1'`s, e.g. indices `0, 2, 3`: `"1100"`. That gives 2 zeros, not 1.

The cleanest concrete path matching `2 → 3 → 0`:
- **Op 1:** `"0011"`, flip indices `0, 1, 2` (two `'0'`s... only flip one zero). Flip indices `0, 2, 3`: index `0` `'0'→'1'`, index `2` `'1'→'0'`, index `3` `'1'→'0'` → `"1000"` (3 zeros). ✓ State `3`.
- **Op 2:** `"1000"`, flip indices `1, 2, 3`: all `'0'→'1'` → `"1111"` (0 zeros). ✓ State `0`.

Two operations achieve the all-`'1'` string, confirming the answer of `2`.

## Python Solution

```python
from collections import deque
from sortedcontainers import SortedSet


class Solution:
    def minOperations(self, s: str, k: int) -> int:
        # Total length of the binary string
        length = len(s)

        # Maintain two SortedSets, partitioned by parity (even / odd).
        # available_by_parity[p] holds all reachable counts whose value % 2 == p.
        # These represent the possible "number of 1s" states we can still visit.
        available_by_parity = [SortedSet() for _ in range(2)]
        for value in range(length + 1):
            available_by_parity[value % 2].add(value)

        # Initial state: the number of zeros in the string.
        # (The BFS works on the count of zeros, aiming to reduce it to 0.)
        zero_count = s.count('0')

        # Remove the starting state from the available pool so it isn't revisited.
        available_by_parity[zero_count % 2].remove(zero_count)

        # BFS queue holds the current frontier of zero-count states.
        queue = deque([zero_count])

        # Number of operations performed so far (BFS level).
        operations = 0

        while queue:
            # Process every state in the current BFS level.
            for _ in range(len(queue)):
                current = queue.popleft()

                # Reaching zero zeros means the string is fully transformed.
                if current == 0:
                    return operations

                # Compute the reachable range of next states after one operation.
                # Each operation flips exactly k characters; the new zero count
                # depends on how many of those flips hit zeros versus ones.
                #
                # left  = current + k - 2 * (max zeros we can flip)
                # right = current + k - 2 * (min zeros we must flip)
                left = current + k - 2 * min(current, k)
                right = current + k - 2 * max(k - length + current, 0)

                # The parity of all reachable next states is fixed by `left`.
                target_set = available_by_parity[left % 2]

                # Collect every still-available state within [left, right].
                index = target_set.bisect_left(left)
                while index < len(target_set) and target_set[index] <= right:
                    next_state = target_set[index]
                    queue.append(next_state)
                    # Mark as visited by removing it from the available pool.
                    target_set.remove(next_state)
                    # After removal, the element at `index` shifts, so we don't
                    # advance the index manually—re-check the same position.

            # Completed one BFS level; one more operation has been applied.
            operations += 1

        # No sequence of operations reaches a zero count of 0.
        return -1
```

## Time and Space Complexity

- **Time Complexity**: `O(n log n)`. The algorithm performs a BFS over the possible states, where each state corresponds to a value in the range `[0, n]` (the number of '0's). Since each value can be visited at most once, there are at most `O(n)` distinct states processed throughout the entire BFS. The two `SortedSet` structures initially hold all `O(n)` values, and each value is removed exactly once. Each `bisect_left`, indexing, and `remove` operation on the `SortedSet` costs `O(log n)`. Because each element is added to the queue and removed from the set only once, the total work is bounded by `O(n)` operations each costing `O(log n)`, giving an overall time complexity of `O(n log n)`.

- **Space Complexity**: `O(n)`. The two `SortedSet` objects together store `O(n)` values (all integers from `0` to `n`). The BFS queue `q` can hold at most `O(n)` states at any time, since each value is enqueued at most once. Hence the total auxiliary space used is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall: Forgetting that the parity of the zero-count is invariant — and mishandling the "impossible" cases

The single most common mistake is treating this problem as if **any** target state can eventually be reached. In reality, every operation flips *exactly* `k` characters, so if an operation flips `x` zeros (turning them into ones) and `k - x` ones (turning them into zeros), the change in the zero-count is:

```
Δ = -x + (k - x) = k - 2x
```

Since `2x` is always even, **the parity of the zero-count changes by the parity of `k` on each operation**. This has two consequences people routinely miss:

1. **If `k` is even**, the parity of the zero-count *never changes*. So if you start with an odd number of zeros, you can **never** reach `0` (which is even) — the answer is `-1` immediately, no matter how many operations you perform.

2. **If `k` is odd**, the parity *alternates* every operation, so both even and odd counts are reachable over time.

A naive BFS that searches across **both** parity buckets without respecting this invariant will either:
- Explore unreachable states (wasting work, or worse, "reaching" `0` through an invalid path if the range math is buggy), or
- Fail to ever terminate correctly.

#### Why the provided code handles this correctly (and the subtle trap inside it)

The given solution sidesteps the issue elegantly: it computes `left` and `right` from the *actual* operation formula, and **all states in `[left, right]` share the parity of `left`**. It then only searches `available_by_parity[left % 2]`. Because the math is grounded in `k - 2x`, the parity invariant is enforced automatically — an even-`k` problem starting from an odd count will simply drain its queue without ever touching the even bucket, and the function returns `-1`.

The **trap** is if a developer "optimizes" or rewrites the expansion step and accidentally searches the wrong bucket, e.g.:

```python
# WRONG: searches the parity of `current`, not `left`
target_set = available_by_parity[current % 2]
```

After an odd-`k` operation, the reachable states have the *opposite* parity of `current`, so this lookup finds nothing valid (or the wrong things), silently breaking correctness.

#### Solution: assert the invariant explicitly and short-circuit the impossible case

Make the reasoning visible and bullet-proof by adding an early check, which also speeds up the obvious impossible case:

```python
class Solution:
    def minOperations(self, s: str, k: int) -> int:
        length = len(s)
        zero_count = s.count('0')

        # Parity invariant: each operation changes the zero-count by (k - 2x),
        # so the count shifts parity by (k % 2) each step.
        # If k is even, parity is frozen. Reaching 0 (even) requires an
        # even starting count; otherwise it is provably impossible.
        if k % 2 == 0 and zero_count % 2 == 1:
            return -1

        # ... proceed with the BFS exactly as before ...
        available_by_parity = [SortedSet() for _ in range(2)]
        for value in range(length + 1):
            available_by_parity[value % 2].add(value)

        available_by_parity[zero_count % 2].remove(zero_count)
        queue = deque([zero_count])
        operations = 0

        while queue:
            for _ in range(len(queue)):
                current = queue.popleft()
                if current == 0:
                    return operations
                left = current + k - 2 * min(current, k)
                right = current + k - 2 * max(k - length + current, 0)

                # Always key off `left`'s parity — this is the parity of the
                # entire reachable range, NOT necessarily `current`'s parity.
                target_set = available_by_parity[left % 2]
                index = target_set.bisect_left(left)
                while index < len(target_set) and target_set[index] <= right:
                    next_state = target_set[index]
                    queue.append(next_state)
                    target_set.remove(next_state)
            operations += 1

        return -1
```

### Related secondary pitfalls

To round out the picture, here are three smaller mistakes worth guarding against:

- **Mutating the SortedSet while iterating by a fixed index.** The original code relies on the fact that after `target_set.remove(...)`, the element that *was* at `index + 1` shifts down into `index`, so it re-checks the same `index`. If you mistakenly write `index += 1` after a removal, you'll **skip** every other element in the range. Don't add the increment; the removal already advances you.

- **Forgetting to remove the start state before the BFS.** If `zero_count` is left in `available_by_parity`, it can be re-discovered as a "next state" and re-enqueued, causing redundant work or, combined with a buggy visited check, an infinite loop. The line `available_by_parity[zero_count % 2].remove(zero_count)` is essential.

- **Off-by-one in the reachable range.** The bounds come from clamping `x` (zeros flipped) to `[max(k - length + current, 0), min(current, k)]`. Getting these clamps wrong — e.g., forgetting that you can't flip more ones than exist (`length - current`) or more zeros than exist (`current`) — produces an invalid range that either misses valid transitions or includes impossible ones. Verify with a tiny case like `s = "010", k = 2`: `current = 2`, `left = 2 + 2 - 2*2 = 0`, `right = 2 + 2 - 2*max(2-3+2,0) = 4 - 2 = 2`, giving range `[0, 2]` of even states — and `0` is reachable in one step, matching the expected answer of `1`.

