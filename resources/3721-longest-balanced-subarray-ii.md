# 3721. Longest Balanced Subarray II

> Source: https://algo.monster/liteproblems/3721

## Problem Statement

You are given an integer array `nums`.

A **subarray** (a contiguous, non-empty sequence of elements within the array) is called **balanced** if the number of **distinct even** numbers in it equals the number of **distinct odd** numbers in it.

Your task is to return the length of the **longest** balanced subarray.

Key points to keep in mind:

- Only **distinct** values are counted. For example, if the subarray is `[2, 2, 3]`, it contains one distinct even number (`2`) and one distinct odd number (`3`), so it is balanced — even though the value `2` appears twice.
- The comparison is between the *count of unique even values* and the *count of unique odd values* within the subarray, not the total occurrences of even and odd elements.
- A subarray with zero distinct even numbers and zero distinct odd numbers would trivially satisfy the condition, but since subarrays must be non-empty, every subarray contains at least one number.

For example:

- Given `nums = [2, 3, 2, 5]`, the subarray `[2, 3]` has one distinct even number (`2`) and one distinct odd number (`3`), so it is balanced. A longer balanced subarray may exist, such as `[3, 2, 5]`? No — that one has one distinct even (`2`) but two distinct odds (`3`, `5`), so it is not balanced. However, `[2, 3, 2]` has one distinct even (`2`) and one distinct odd (`3`), giving a balanced subarray of length `3`.
- If no balanced subarray exists (e.g., `nums = [1, 1, 1]`, which contains only odd numbers), the answer is `0`.

The goal is to find the maximum possible length among all such balanced subarrays.

## Intuition

The condition "distinct even count equals distinct odd count" naturally suggests turning the problem into a **balance/prefix-sum** problem. If we assign `+1` to every distinct odd number and `-1` to every distinct even number, then a subarray is balanced exactly when the sum of these contributions inside it is `0`.

This is similar to the classic problem of finding the longest subarray with equal numbers of `0`s and `1`s, where we map values to `+1`/`-1` and look for two positions with the **same [prefix sum](/problems/subarray_sum)**. If `now[i]` denotes the prefix balance up to index `i`, then the subarray `(pos, i]` is balanced whenever `now[i] = now[pos]`, and its length is `i - pos`. To maximize the length, for each right endpoint `i` we want the **earliest** position `pos` whose prefix balance equals the current one.

The tricky part is the word **distinct**. A number should only contribute once per subarray, no matter how many times it appears. This breaks the simple static prefix-sum trick, because whether an element "counts" depends on the left boundary of the subarray: an occurrence of `x` at index `i` matters only for subarrays that start *after* the previous occurrence of `x`.

The standard way to handle this is to think of each occurrence as contributing to a **range of prefix positions**:

- When we see value `x` at index `i`, it adds its contribution (`+1` if odd, `-1` if even) to all prefix sums at positions `i, i+1, \ldots, n`.
- But if `x` appeared before at position `last[x]`, that earlier occurrence's contribution must be **revoked** for the range starting at `last[x]`, because for any subarray containing both occurrences, only one should count. Effectively the contribution "moves" from the old occurrence to the new one.

So as the right endpoint `i` sweeps from left to right, the array of prefix balances is no longer fixed — it gets **range updates** (`add +1` or `-1` over a suffix). After each update, we need to find the smallest position `pos` such that the (dynamically maintained) prefix balance at `pos` equals the current total balance `now`.

This is exactly what a **lazy [segment tree](/problems/segment_tree_intro)** over the prefix positions can do:

- It supports **range addition** to handle adding/revoking contributions over suffixes.
- Each node stores the **minimum and maximum** value in its interval. Since each update changes values by `±1` on a contiguous range, the set of values within any interval is "dense" — every integer between the interval's min and max actually occurs. Therefore, checking `mn <= now <= mx` is enough to know whether the target value exists in a subtree, which lets us **binary search on the tree** for the leftmost position holding value `now` in `O(log n)` time.

Combining these pieces — prefix balance, a hash table `last` to detect repeated values, and a [segment tree](/problems/segment_tree_intro) with min/max plus lazy range updates for fast leftmost-equal queries — gives an `O(n \log n)` sweep: for each right endpoint, update contributions, query the earliest matching prefix position `pos`, and update the answer with `i - pos`.

**Learn more about [Segment Tree](/problems/segment_tree_intro), [Divide and Conquer](/problems/divide_and_conquer_intro) and [Prefix Sum](/problems/subarray_sum) patterns.**

## Solution Approach

**[Segment Tree](/problems/segment_tree_intro) + [Prefix Sum](/problems/subarray_sum) + Hash Table**

We maintain a [segment tree](/problems/segment_tree_intro) over the prefix positions `0, 1, \ldots, n`, where the value at position `p` represents the prefix balance after processing the first `p` elements (with contributions adjusted for distinctness). Position `0` represents the empty prefix with balance `0`.

**Step 1: Define the [segment tree](/problems/segment_tree_intro) node**

Each node covers an interval `[l, r]` and stores:

- `mn` / `mx`: the minimum and maximum prefix balance within the interval;
- `lazy`: a pending range-addition tag.

The tree supports two operations:

- `modify(u, l, r, v)`: add `v` to every position in `[l, r]` (range addition with lazy propagation);
- `query(u, target)`: find the **leftmost** position whose value equals `target`.

The query works by walking down the tree: at each node, first push down the lazy tag, then check whether the left child satisfies `mn <= target <= mx`. Because all updates are `±1` over contiguous ranges, adjacent positions differ by at most `1`, so every integer in `[mn, mx]` is guaranteed to appear in that subtree — the membership check is exact. If the left child can contain `target`, descend left (to find the earliest position); otherwise descend right. A leaf returns its position. Note that the target is always present in the tree, since position `0` is never modified except by suffix updates that also affect `now` — more precisely, the current balance `now` always equals the value at the latest position `i`, so the query never fails.

**Step 2: Sweep the right endpoint and maintain contributions**

We iterate `i` from `1` to `n` over the elements `x = nums[i-1]`, maintaining:

- `last`: a hash table mapping each value to the index of its most recent occurrence;
- `now`: the current total balance, i.e., the value stored at position `i` (or `n`) in the tree.

For each element `x`, let its contribution be:

```
det = +1  if x is odd
det = -1  if x is even
```

Then:

1. **Revoke the previous occurrence.** If `x` appeared before at `last[x]`, its old contribution applied to all positions `>= last[x]`. Since only one occurrence should count per subarray, undo it with `modify(1, last[x], n, -det)` and update `now -= det`. Conceptually, this transfers the contribution of `x` from its old position to the new one.
2. **Add the current occurrence.** Record `last[x] = i`, then apply `modify(1, i, n, det)` and update `now += det`. Now, for every left boundary `pos`, the value at position `pos` in the tree is exactly the balance of the subarray `(pos, i]`... rather, the prefix value such that the subarray balance is `now - value(pos)`.
3. **Query the earliest matching prefix.** The subarray `(pos, i]` is balanced precisely when the tree value at `pos` equals `now`. Call `pos = query(1, now)` to get the leftmost such position, and update:

```
ans = max(ans, i - pos)
```

**Why the invariant holds**

For a fixed right endpoint `i`, the tree value at position `pos` equals:

```
(distinct odds) - (distinct evens) over the prefix, counting each value
only at its last occurrence within [1, i]
```

A value `x` whose last occurrence in `[1, i]` is at index `j` contributes to positions `pos >= j`. For the subarray `(pos, i]`, the value `x` is included if and only if `j > pos`, which means its contribution is present in `value(i) = now` but absent from `value(pos)` exactly when `x` lies inside the subarray. Hence:

```
balance of (pos, i] = now - value(pos)
```

and the subarray is balanced if and only if `value(pos) = now`.

**Step 3: Return the answer**

After processing all right endpoints, `ans` holds the length of the longest balanced subarray (it stays `0` if none exists).

**Complexity Analysis**

- **Time complexity:** `O(n log n)`. Each element triggers at most two range modifications and one tree binary search, each costing `O(log n)`.
- **Space complexity:** `O(n)` for the [segment tree](/problems/segment_tree_intro) (about `4(n+1)` nodes) and the hash table `last`.

**Walkthrough on `nums = [2, 3, 2, 5]`**

| `i` | `x` | action | `now` | `query(now)` → `pos` | `i - pos` |
|---|---|---|---|---|---|
| 1 | 2 | add `-1` on `[1,4]` | `-1` | leftmost `-1` is `1` | `0` |
| 2 | 3 | add `+1` on `[2,4]` | `0` | leftmost `0` is `0` | `2` |
| 3 | 2 | revoke `-1` at `[1,4]`, add `-1` on `[3,4]` | `0` | leftmost `0` is `0` | `3` |
| 4 | 5 | add `+1` on `[4,4]` | `1` | leftmost `1` is `4` | `0` |

The answer is `3`, corresponding to the balanced subarray `[2, 3, 2]` (one distinct even, one distinct odd).

## Example Walkthrough

Let's trace the algorithm on `nums = [2, 3, 2, 5]` (so `n = 4`).

We maintain a segment tree over prefix positions `0..4`. Conceptually, the tree holds an array `value[0..4]`, where `value[pos]` is the adjusted prefix balance (distinct odds minus distinct evens, with each value counted only at its **last** occurrence so far). Initially everything is `0`:

```
value = [0, 0, 0, 0, 0],  now = 0,  last = {},  ans = 0
```

---

**Step `i = 1`, `x = 2` (even, `det = -1`)**

- `2` has not been seen before, so there is nothing to revoke.
- Record `last[2] = 1`, apply `modify([1, 4], -1)`, and set `now = -1`.

```
value = [0, -1, -1, -1, -1],  now = -1
```

- `query(-1)` → leftmost position with value `-1` is `pos = 1`.
- `ans = max(0, 1 - 1) = 0`. The candidate subarray `(1, 1]` is empty, so no gain.

---

**Step `i = 2`, `x = 3` (odd, `det = +1`)**

- First occurrence of `3`; nothing to revoke.
- Record `last[3] = 2`, apply `modify([2, 4], +1)`, and set `now = 0`.

```
value = [0, -1, 0, 0, 0],  now = 0
```

- `query(0)` → during the descent, the left subtree reports `mn = -1, mx = 0`, and since values change by `±1` between neighbors, `0` is guaranteed to exist there — we descend left and land on `pos = 0`.
- `ans = max(0, 2 - 0) = 2`. This corresponds to subarray `(0, 2] = [2, 3]`: one distinct even, one distinct odd. Balanced ✓

---

**Step `i = 3`, `x = 2` (even, `det = -1`) — the interesting case**

- `2` was seen at `last[2] = 1`, so its old contribution must move:
  - **Revoke:** `modify([1, 4], +1)`, `now = 0 - (-1) = 1`:
    ```
    value = [0, 0, 1, 1, 1]
    ```
  - **Re-add at the new position:** `last[2] = 3`, `modify([3, 4], -1)`, `now = 1 + (-1) = 0`:
    ```
    value = [0, 0, 1, 0, 0],  now = 0
    ```
- Notice what this accomplished: position `1` no longer "remembers" the old `2`, because for any subarray starting at `pos = 1` or `pos = 2`, the second `2` at index `3` is the one that counts — and it must count **only once**.
- `query(0)` → leftmost position with value `0` is `pos = 0`.
- `ans = max(2, 3 - 0) = 3`. Subarray `(0, 3] = [2, 3, 2]`: distinct evens `{2}`, distinct odds `{3}` — sizes `1 = 1`. Balanced ✓ Length `3`.

---

**Step `i = 4`, `x = 5` (odd, `det = +1`)**

- First occurrence of `5`; record `last[5] = 4`, apply `modify([4, 4], +1)`, `now = 1`.

```
value = [0, 0, 1, 0, 1],  now = 1
```

- `query(1)` → leftmost position with value `1` is `pos = 2`.
- `i - pos = 4 - 2 = 2`. This finds subarray `(2, 4] = [2, 5]`: one distinct even, one distinct odd — balanced, but shorter than what we already have.
- `ans = max(3, 2) = 3`.

---

**Summary table**

| `i` | `x` | Tree updates | `value[0..4]` after updates | `now` | `pos` | `i - pos` | `ans` |
|---|---|---|---|---|---|---|---|
| 1 | 2 | `[1,4] -1` | `[0, -1, -1, -1, -1]` | `-1` | 1 | 0 | 0 |
| 2 | 3 | `[2,4] +1` | `[0, -1, 0, 0, 0]` | `0` | 0 | 2 | 2 |
| 3 | 2 | revoke `[1,4] +1`, then `[3,4] -1` | `[0, 0, 1, 0, 0]` | `0` | 0 | 3 | **3** |
| 4 | 5 | `[4,4] +1` | `[0, 0, 1, 0, 1]` | `1` | 2 | 2 | 3 |

The final answer is **`3`**, achieved by the balanced subarray `[2, 3, 2]`.

The key invariant visible throughout the trace: at every step, the subarray `(pos, i]` has balance `now - value[pos]`, so it is balanced exactly when `value[pos] = now` — and the segment tree's min/max-guided descent finds the **leftmost** such `pos` in `O(log n)`, maximizing the length for each right endpoint.

## Python Solution

```python
from typing import List


class Solution:
    def longestBalanced(self, nums: List[int]) -> int:
        """
        Find the length of the longest subarray in which the number of
        DISTINCT odd values equals the number of DISTINCT even values.

        Idea:
        - Assign each value a weight: +1 if odd, -1 if even.
        - For every prefix ending at index i, maintain a segment tree over
          positions [0, n] where tree[j] = sum of weights of all distinct
          values whose LAST occurrence lies in [1, j].
        - The distinct-value balance of subarray (j, i] is then
          total_balance - tree[j]. It is zero exactly when
          tree[j] == total_balance.
        - Since adjacent leaves differ by exactly +1 / -1, the leftmost
          position holding a target value can be located by walking down
          the tree (intermediate value property), maximizing i - j.
        """
        n = len(nums)

        class Node:
            """A segment tree node covering the index range [left, right]."""
            __slots__ = ("left", "right", "min_val", "max_val", "lazy")

            def __init__(self) -> None:
                self.left = self.right = 0       # range boundaries
                self.min_val = self.max_val = 0  # min / max leaf value in range
                self.lazy = 0                    # pending range-add value

        tree = [Node() for _ in range((n + 1) * 4)]

        def build(node: int, left: int, right: int) -> None:
            """Initialize the node for range [left, right] and recurse."""
            tree[node].left, tree[node].right = left, right
            tree[node].min_val = tree[node].max_val = tree[node].lazy = 0
            if left == right:
                return
            mid = (left + right) >> 1
            build(node << 1, left, mid)
            build(node << 1 | 1, mid + 1, right)

        def apply_update(node: int, value: int) -> None:
            """Apply a range-add of `value` to this node."""
            tree[node].min_val += value
            tree[node].max_val += value
            tree[node].lazy += value

        def push_down(node: int) -> None:
            """Propagate the pending lazy value to both children."""
            if tree[node].lazy:
                apply_update(node << 1, tree[node].lazy)
                apply_update(node << 1 | 1, tree[node].lazy)
                tree[node].lazy = 0

        def push_up(node: int) -> None:
            """Recompute this node's aggregates from its children."""
            tree[node].min_val = min(tree[node << 1].min_val, tree[node << 1 | 1].min_val)
            tree[node].max_val = max(tree[node << 1].max_val, tree[node << 1 | 1].max_val)

        def update(node: int, left: int, right: int, value: int) -> None:
            """Add `value` to every position in [left, right]."""
            if tree[node].left >= left and tree[node].right <= right:
                apply_update(node, value)
                return
            push_down(node)
            mid = (tree[node].left + tree[node].right) >> 1
            if left <= mid:
                update(node << 1, left, right, value)
            if right > mid:
                update(node << 1 | 1, left, right, value)
            push_up(node)

        def query(node: int, target: int) -> int:
            """
            Return the leftmost position whose value equals `target`.
            Valid because adjacent leaf values differ by exactly 1,
            so target lies in a subtree iff min <= target <= max.
            """
            if tree[node].left == tree[node].right:
                return tree[node].left
            push_down(node)
            if tree[node << 1].min_val <= target <= tree[node << 1].max_val:
                return query(node << 1, target)
            return query(node << 1 | 1, target)

        # Build the tree over positions 0..n (position 0 = empty prefix).
        build(1, 0, n)

        last_pos = {}            # value -> index (1-based) of its last occurrence
        total_balance = 0        # distinct-value balance of the whole prefix
        ans = 0

        for i, x in enumerate(nums, start=1):
            delta = 1 if (x & 1) else -1  # +1 for odd, -1 for even

            # If x appeared before, cancel its previous contribution,
            # which covered the suffix [last_pos[x], n].
            if x in last_pos:
                update(1, last_pos[x], n, -delta)
                total_balance -= delta

            # Record x's new last occurrence and add its contribution
            # to the suffix [i, n].
            last_pos[x] = i
            update(1, i, n, delta)
            total_balance += delta

            # Leftmost j with tree[j] == total_balance gives the longest
            # balanced subarray (j, i].
            pos = query(1, total_balance)
            ans = max(ans, i - pos)

        return ans
```

## Time and Space Complexity

- **Time Complexity:** `O(n log n)`, where `n` is the length of the array `nums`.
  - The loop enumerates each right endpoint, running `n` iterations in total.
  - In each iteration, the code performs at most two range update operations (`modify`) and one binary-search-style query (`query`) on the segment tree.
  - Each `modify` call recurses down the tree to a depth of `O(log n)`, applying lazy tags and pushing values up/down along the path, so it costs `O(log n)` per call.
  - Each `query` call descends from the root to a leaf, choosing the left or right child based on whether `target` lies within `[mn, mx]` of the left subtree, which also takes `O(log n)`.
  - Building the segment tree with `build(1, 0, n)` visits every node exactly once, costing `O(n)`.
  - Therefore the total time is `O(n) + n × O(log n) = O(n log n)`.

- **Space Complexity:** `O(n)`, where `n` is the length of the array `nums`.
  - The segment tree array `tr` allocates `(n + 1) * 4` nodes, each with constant-size fields (`l`, `r`, `mn`, `mx`, `lazy`), occupying `O(n)` space.
  - The hash table `last`, which records the most recent index of each distinct value, stores at most `n` entries, taking `O(n)` space.
  - The recursion depth of `build`, `modify`, and `query` is bounded by the tree height `O(log n)`, which is dominated by the `O(n)` terms above.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Forgetting to revoke the previous occurrence of a duplicate value

This is the most frequent and most damaging mistake. The whole correctness of the
prefix-balance invariant rests on each value contributing **exactly once**, at its
**latest** occurrence. A tempting but wrong simplification looks like this:

```python
for i, x in enumerate(nums, start=1):
    delta = 1 if (x & 1) else -1
    update(1, i, n, delta)          # WRONG: old occurrence still counted
    total_balance += delta
    pos = query(1, total_balance)
    ans = max(ans, i - pos)
```

With this version, an input like `[2, 2, 3]` is mishandled: the value `2` adds
`-1` twice, so the tree behaves as if there were two distinct evens. The query
then looks for the wrong target and either misses the balanced subarray
`[2, 2, 3]` (one distinct even, one distinct odd) or reports a bogus length.

**Why it breaks:** the invariant states that `tree[j]` equals the distinct
balance of values whose *last* occurrence lies in `[1, j]`. If the old
occurrence is not cancelled, a duplicate value is counted at *both* positions,
so `total_balance - tree[j]` no longer equals the balance of subarray `(j, i]`.

**Fix:** before inserting the current occurrence, undo the old suffix update
covering `[last_pos[x], n]`, effectively *moving* the value's contribution to
its new position:

```python
if x in last_pos:
    update(1, last_pos[x], n, -delta)   # cancel the stale contribution
    total_balance -= delta
last_pos[x] = i
update(1, i, n, delta)
total_balance += delta
```

---

### Pitfall 2: Querying for `0` instead of `total_balance`

It is easy to think "a balanced subarray has balance 0, so search for the
leftmost position holding `0`." But `tree[j]` stores the **prefix** balance,
not the subarray balance. The subarray `(j, i]` has balance
`total_balance - tree[j]`, so the correct search target is:

```python
pos = query(1, total_balance)   # NOT query(1, 0)
```

Searching for `0` only works coincidentally when `total_balance == 0`, and it
silently returns wrong answers otherwise (the test `[2, 3, 2, 5]` exposes this
at `i = 3`, where `total_balance = 0` happens to mask the bug — try
`[2, 3, 5, 4]` instead).

---

### Pitfall 3: Building the tree over `[1, n]` and dropping position `0`

Position `0` represents the **empty prefix** and is the left boundary that
yields subarrays starting at index 1 (i.e., the full prefix `[1, i]`). If the
tree only covers `[1, n]`:

- subarrays that start at the very beginning of the array can never be found, and
- the query may even fail, because the target value `total_balance` is no
  longer guaranteed to exist anywhere in the tree.

Always build over `[0, n]`:

```python
build(1, 0, n)
```

and never apply any element update to position `0` itself (updates start at `i >= 1`).

---

### Pitfall 4: Missing `push_down` inside `query`

The query walks from the root to a leaf reading `min_val` / `max_val` of the
children. If a node still carries a pending `lazy` tag, its children's
aggregates are stale, and the descent direction (left vs. right child) can be
chosen incorrectly — producing a non-leftmost position or an outright wrong
leaf. The fix is to propagate before inspecting children:

```python
def query(node, target):
    if tree[node].left == tree[node].right:
        return tree[node].left
    push_down(node)                       # essential — do not skip
    if tree[node << 1].min_val <= target <= tree[node << 1].max_val:
        return query(node << 1, target)
    return query(node << 1 | 1, target)
```

A related subtlety: the membership test `min <= target <= max` is only exact
because every update is a `±1` range-add over a suffix, which keeps adjacent
leaf values differing by at most 1 (an intermediate-value property). If you
adapt this template to a problem with arbitrary deltas, this check becomes a
necessary-but-not-sufficient condition and the query logic must be redesigned.

---

### Pitfall 5: Recursion depth on large inputs

The recursive `build`, `update`, and `query` each descend `O(log n)` levels,
which is safe, but `build` recurses into every node and Python's default
recursion limit (1000) can be hit through deep call chains on very large `n`
combined with other recursion in the program. Two safeguards:

```python
import sys
sys.setrecursionlimit(2 * 10**5)
```

or convert `build` to an iterative initialization. Alternatively, since all
initial values are `0`, you can skip `build` entirely by setting boundaries
lazily — but then `tree[node].left/right` must be computed from parameters
passed down the recursion rather than read from the node.

