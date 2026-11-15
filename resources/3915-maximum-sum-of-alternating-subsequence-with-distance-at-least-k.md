# 3915. Maximum Sum of Alternating Subsequence With Distance at Least K

> Source: https://algo.monster/liteproblems/3915

## Problem Statement

You are given an integer array `nums` of length `n` and an integer `k`. Your task is to pick a **subsequence** from `nums` that satisfies certain constraints, and then return the **maximum** possible **score** of such a subsequence.

When you pick a subsequence, you choose indices `0 <= i₁ < i₂ < ... < iₘ < n`. These chosen indices must follow two rules:

1. **Minimum gap constraint**: For every consecutive pair of chosen indices, the gap between them must be at least `k`. In other words, for every `1 <= t < m`, we need `i_{t+1} - i_t >= k`. This means you cannot pick two elements that are too close together in terms of their positions.

2. **Strictly alternating constraint**: The selected values must form a **strictly alternating** sequence. This means the values must go up and down in a zigzag pattern, following one of these two shapes:
   - `nums[i₁] < nums[i₂] > nums[i₃] < ...` (starts by going up), or
   - `nums[i₁] > nums[i₂] < nums[i₃] > ...` (starts by going down)

   Note that the comparisons are **strict** (using `<` and `>`), so two adjacent selected values can never be equal.

A subsequence of length `1` (a single element) is automatically considered **strictly** alternating, since there are no adjacent pairs to compare.

The **score** of a valid subsequence is defined as the **sum** of all its selected values. Among all possible valid subsequences, you should return the one with the **maximum** total score.

To summarize, you must:
- Select indices that are spaced at least `k` apart.
- Ensure the corresponding values strictly zigzag (alternately increasing and decreasing).
- Maximize the sum of the selected values.

The final answer is a single integer representing this maximum score.

## Intuition

When we look at this problem, the key observation is that we are building a subsequence step by step, and at each step we need to know two things about the element we just placed: its **value** and whether it currently sits as a **valley** (the next element must be larger) or as a **peak** (the next element must be smaller). This naturally points us toward **[dynamic programming](/problems/dynamic_programming_intro)**, where the state captures the position of the last chosen element and the direction the sequence is heading.

Let's define two states for each index `i`:

- `f[i][0]`: the maximum sum of a valid subsequence ending at index `i`, where `nums[i]` is a **valley**. To extend this, the next chosen value must be **larger** than `nums[i]`.
- `f[i][1]`: the maximum sum of a valid subsequence ending at index `i`, where `nums[i]` is a **peak**. To extend this, the next chosen value must be **smaller** than `nums[i]`.

Why split into two states? Because the alternating rule means the requirement for the next element depends entirely on whether the current element is a high point or a low point. By tracking this, we always know what kind of element can follow.

Now think about how to transition. If `nums[i]` is a valley (`f[i][0]`), then the element before it in the subsequence must have been a **peak** with a value **greater** than `nums[i]`. So we look back at all earlier indices `j` that satisfy two conditions:

- The position gap is valid: `j <= i - k`.
- The value condition holds: `nums[j] > nums[i]`.

Among all such `j`, we want the best `f[j][1]`, then add `nums[i]`:

`f[i][0] = nums[i] + max(0, max over valid j of f[j][1])`

Similarly, if `nums[i]` is a peak (`f[i][1]`), the previous element must have been a **valley** with a smaller value:

`f[i][1] = nums[i] + max(0, max over valid j of f[j][0])`

The `max(0, ...)` part allows a subsequence to start fresh at index `i` (a length-1 subsequence), which is always valid.

If we computed these transitions naively, for each `i` we would scan all earlier valid `j`, giving an `O(n²)` solution. That is too slow when `n` is large. The bottleneck is the repeated query: *"find the maximum DP value among earlier elements whose value is greater than (or less than) the current value."*

This is exactly the kind of query a **Binary Indexed Tree (Fenwick Tree)** handles well. If we index the BIT by **value** instead of position, then:

- Querying "maximum `f[·][0]` among values **less than** `nums[i]`" becomes a **prefix maximum** query in a BIT keyed by value.
- Querying "maximum `f[·][1]` among values **greater than** `nums[i]`" becomes a **suffix maximum** query, which we turn into a prefix query by reversing the value order (indexing by `M + 1 - val`).

We use two BITs, one for each state. Since the values can be large, we first **compress** them: collect all distinct values, sort them, and map each value to a small rank. This keeps the BIT size proportional to the number of distinct values.

Finally, we must respect the position constraint `j <= i - k`. The trick is a **sliding insertion**: while processing index `i`, we only insert the state of index `i - k` into the BITs (using a delayed pointer). This guarantees that every element currently inside the BITs is at least `k` positions before `i`, so every query automatically respects the gap rule without any extra checking. As we move forward, elements become eligible exactly when they fall `k` steps behind, and they enter the trees right on time.

**Learn more about [Segment Tree](/problems/segment_tree_intro) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

We implement the idea as **[Dynamic Programming](/problems/dynamic_programming_intro) combined with two Binary Indexed Trees (Fenwick Trees)**. Let's walk through the pieces step by step.

### Step 1: Coordinate Compression

The values in `nums` can be large, but a BIT needs to be indexed over a compact range. So we first build a sorted list of the **distinct** values:

```python
stl = sorted(set(nums))
rank = {v: i + 1 for i, v in enumerate(stl)}
```

Here `rank[v]` maps each value to a `1`-based index. Using `1`-based indices is important because Fenwick Trees do not work with index `0`. After this, every value `nums[i]` can be referred to by a small integer `rank[nums[i]]` in the range `[1, len(stl)]`.

### Step 2: A Max-Based Fenwick Tree

We need a BIT that maintains **prefix maximums** rather than the usual prefix sums. The structure is:

```python
def update(self, index, val):
    while index <= self.n:
        self.tree[index] = max(self.tree[index], val)
        index += index & (-index)

def preSum(self, pos):
    ans = 0
    while pos >= 1:
        ans = max(ans, self.tree[pos])
        pos -= pos & (-pos)
    return ans
```

- `update(index, val)` records `val` at position `index` by climbing upward with `index += index & (-index)`, taking the `max` at each touched node.
- `preSum(pos)` returns the maximum stored value over positions `[1, pos]` by walking down with `pos -= pos & (-pos)`.

We create two such trees:

- `fwt0` keyed by **value in ascending order**, storing `f[·][0]` (valley states).
- `fwt1` keyed by **value in reversed order**, storing `f[·][1]` (peak states).

### Step 3: The DP States

We keep `dp[i] = [dp[i][0], dp[i][1]]`, where `dp[i][0]` is the best valley-ending sum and `dp[i][1]` is the best peak-ending sum at index `i`. Each one starts as a length-`1` subsequence:

```python
dp[i][0] = dp[i][1] = nums[i]
```

### Step 4: Querying the Trees (Transitions)

We process indices `i` from left to right. When `i >= k`, there is at least one eligible predecessor already inserted, so we perform the transitions:

```python
indx = rank[nums[i]]
dp[i][1] = max(dp[i][1], fwt0.preSum(indx - 1) + nums[i])
dp[i][0] = max(dp[i][0], fwt1.preSum(len(stl) - indx) + nums[i])
```

- For `dp[i][1]` (peak): we need an earlier valley with a **smaller** value. In `fwt0` (ascending order), values strictly less than `nums[i]` occupy positions `[1, indx - 1]`, so `fwt0.preSum(indx - 1)` gives the best `f[·][0]` among them.
- For `dp[i][0]` (valley): we need an earlier peak with a **larger** value. In `fwt1`, values are reversed, so values strictly greater than `nums[i]` map to the first `len(stl) - indx` positions, and `fwt1.preSum(len(stl) - indx)` gives the best `f[·][1]` among them.

### Step 5: Sliding Insertion to Enforce the Gap

The position constraint `j <= i - k` is handled by a **delayed insertion**. While we are at index `i`, we insert the state of index `i - k + 1`... but notice the query above only runs when `i >= k`, meaning the earliest queried predecessor is at most `i - k`. The insertion happens as:

```python
if i - k + 1 >= 0:
    indx = rank[nums[i - k + 1]]
    fwt0.update(indx, dp[i - k + 1][0])
    fwt1.update(len(stl) - indx + 1, dp[i - k + 1][1])
```

Because insertion is offset by `k`, every element living inside the trees when we query at index `i` is guaranteed to be at least `k` positions earlier. This means the gap rule `i_{t+1} - i_t >= k` is enforced automatically — no element that is too close ever gets a chance to contribute. Note that `fwt1` is updated at the **reversed** position `len(stl) - indx + 1` to match its reversed ordering.

### Step 6: Tracking the Answer

After computing both states for index `i`, we fold them into the running answer:

```python
res = max(res, dp[i][0], dp[i][1])
```

Since a single element is always a valid subsequence, `res` is initialized to `nums[0]`, ensuring the answer is never empty.

### Complexity

- **Time**: `O(n log n)`. Each index triggers a constant number of BIT operations, and each BIT operation costs `O(log n)`. Sorting for compression also costs `O(n log n)`.
- **Space**: `O(n)` for the two Fenwick Trees, the `dp` table, and the rank map.

## Example Walkthrough

Let's trace through a small example to see how the DP and two Fenwick Trees work together.

**Input:** `nums = [3, 1, 4, 2]`, `k = 2`

We want to pick a subsequence whose chosen indices are at least `k = 2` apart, whose values strictly zigzag, and whose sum is maximized.

---

#### Step 1: Coordinate Compression

Collect distinct values and sort them:

```
stl  = [1, 2, 3, 4]
rank = {1: 1, 2: 2, 3: 3, 4: 4}   (1-based)
```

So `len(stl) = 4`. For the reversed tree `fwt1`, a value with rank `r` is stored at position `4 - r + 1 = 5 - r`.

---

#### Step 2: Initialize

- `dp[i][0] = dp[i][1] = nums[i]` (every element is a valid length-1 subsequence).
- `fwt0` (ascending, stores valley states `dp[·][0]`).
- `fwt1` (reversed, stores peak states `dp[·][1]`).
- `res = nums[0] = 3`.

Initial DP:

| i | nums[i] | dp[i][0] (valley) | dp[i][1] (peak) |
|---|---------|-------------------|-----------------|
| 0 | 3       | 3                 | 3               |
| 1 | 1       | 1                 | 1               |
| 2 | 4       | 4                 | 4               |
| 3 | 2       | 2                 | 2               |

---

#### Step 3: Process each index (left to right)

The rule: **query** only when `i >= k` (i.e. `i >= 2`), and **insert** index `i - k + 1` when `i - k + 1 >= 0` (i.e. `i >= 1`).

---

**i = 0** (`nums[0] = 3`)

- Query? `0 >= 2`? No. Skip.
- Insert? `0 - 2 + 1 = -1 >= 0`? No. Skip.
- `res = max(3, 3, 3) = 3`.

Both trees still empty.

---

**i = 1** (`nums[1] = 1`)

- Query? `1 >= 2`? No. Skip.
- Insert? `1 - 2 + 1 = 0 >= 0`? Yes → insert index `0` (`nums[0] = 3`, rank `3`):
  - `fwt0.update(3, dp[0][0] = 3)`
  - `fwt1.update(5 - 3 = 2, dp[0][1] = 3)`
- `res = max(3, 1, 1) = 3`.

Now the trees hold index 0's states. Notice index 0 enters the tree exactly when we're at index 1, but it only becomes *queryable* at index `2` (gap of 2). 

---

**i = 2** (`nums[2] = 4`, rank = 4)

- Query? `2 >= 2`? Yes.
  - **Peak** `dp[2][1]`: need an earlier **valley with smaller value**. Query `fwt0.preSum(rank - 1 = 3)` → best valley among values `{1,2,3}`. Index 0 (value 3, valley sum 3) is stored at position 3, so this returns `3`.
    - `dp[2][1] = max(4, 3 + 4) = 7` → subsequence `[3, 4]` (valley 3 → peak 4). ✓
  - **Valley** `dp[2][0]`: need an earlier **peak with larger value**. Query `fwt1.preSum(len(stl) - rank = 4 - 4 = 0)` → empty range → returns `0`.
    - `dp[2][0] = max(4, 0 + 4) = 4` (stays length-1; no larger peak exists).
- Insert? `2 - 2 + 1 = 1 >= 0`? Yes → insert index `1` (`nums[1] = 1`, rank `1`):
  - `fwt0.update(1, dp[1][0] = 1)`
  - `fwt1.update(5 - 1 = 4, dp[1][1] = 1)`
- `res = max(3, 4, 7) = 7`.

Updated DP row: `dp[2] = [4, 7]`.

---

**i = 3** (`nums[3] = 2`, rank = 2)

- Query? `3 >= 2`? Yes.
  - **Peak** `dp[3][1]`: query `fwt0.preSum(rank - 1 = 1)` → best valley among values `{1}`. Index 1 (value 1, valley sum 1) sits at position 1 → returns `1`.
    - `dp[3][1] = max(2, 1 + 2) = 3` → subsequence `[1, 2]` (valley 1 → peak 2). ✓
  - **Valley** `dp[3][0]`: query `fwt1.preSum(len(stl) - rank = 4 - 2 = 2)` → best peak among values strictly greater than `2`, i.e. `{3, 4}`, mapped to reversed positions `[1, 2]`. Index 0 (value 3, peak sum 3) is at position 2 → returns `3`.
    - `dp[3][0] = max(2, 3 + 2) = 5` → subsequence `[3, 2]` (peak 3 → valley 2). ✓
- Insert? Index `3 - 2 + 1 = 2` would be inserted, but we're done querying after this, so it has no further effect.
- `res = max(7, 5, 3) = 7`.

Updated DP row: `dp[3] = [5, 3]`.

---

#### Step 4: Final Answer

```
res = 7
```

The best subsequence is `[3, 4]` — indices `0` and `2`, gap `= 2 >= k`, values `3 < 4` (a valid up-zigzag), sum `= 7`.

---

#### Why the gap is respected automatically

Observe the **delayed insertion** in action:

- Index 0 was inserted while processing **i = 1**, but the earliest query that could read it happened at **i = 2** — exactly `k = 2` positions away.
- When we queried at **i = 2**, index 1 had *not yet* been inserted (it goes in *during* i = 2's insertion step), so index 1 (only 1 apart) could never illegally contribute to index 2.

This offset-by-`k` insertion guarantees every element inside the trees is at least `k` positions behind the current index, so the gap constraint `i_{t+1} - i_t >= k` holds without any explicit position checks.

## Python Solution

```python
from typing import List


class FenwickTree:
    """Binary Indexed Tree that maintains prefix maximums (instead of prefix sums)."""

    def __init__(self, size: int) -> None:
        self.size = size
        self.tree = [0] * (size + 1)

    def update(self, index: int, value: int) -> None:
        """Set the maximum value at `index`, propagating forward through the tree."""
        while index <= self.size:
            self.tree[index] = max(self.tree[index], value)
            index += index & (-index)  # move to the next responsible node

    def query_prefix_max(self, position: int) -> int:
        """Return the maximum value over the prefix [1, position]."""
        result = 0
        while position >= 1:
            result = max(result, self.tree[position])
            position -= position & (-position)  # move to the previous block
        return result


class Solution:
    def maxAlternatingSum(self, nums: List[int], k: int) -> int:
        # Coordinate compression: map each distinct value to a 1-based rank.
        sorted_values = sorted(set(nums))
        rank = {value: idx + 1 for idx, value in enumerate(sorted_values)}
        num_distinct = len(sorted_values)

        # fenwick_up   : queried for transitions where the new element is larger.
        # fenwick_down : queried for transitions where the new element is smaller
        #                (handled via reversed ranks).
        fenwick_up = FenwickTree(num_distinct)
        fenwick_down = FenwickTree(num_distinct)

        n = len(nums)
        # dp[i][0]: best alternating sum ending at i, next expected move is "down".
        # dp[i][1]: best alternating sum ending at i, next expected move is "up".
        dp = [[0, 0] for _ in range(n)]
        result = nums[0]

        for i in range(n):
            # Base case: start a new subsequence at index i.
            dp[i][0] = dp[i][1] = nums[i]

            if i >= k:
                current_rank = rank[nums[i]]

                # Extend a chain whose previous element is smaller than nums[i]:
                # query prefix [1, current_rank - 1] over the "up" tree.
                dp[i][1] = max(
                    dp[i][1],
                    fenwick_up.query_prefix_max(current_rank - 1) + nums[i],
                )

                # Extend a chain whose previous element is larger than nums[i]:
                # in reversed coordinates that is prefix [1, num_distinct - current_rank].
                dp[i][0] = max(
                    dp[i][0],
                    fenwick_down.query_prefix_max(num_distinct - current_rank) + nums[i],
                )

            # Delayed update: only positions at least k indices behind index i
            # may serve as the previous element, satisfying the distance constraint.
            if i - k + 1 >= 0:
                source = i - k + 1
                source_rank = rank[nums[source]]
                # Register dp[source][0] in normal order for future "up" queries.
                fenwick_up.update(source_rank, dp[source][0])
                # Register dp[source][1] in reversed order for future "down" queries.
                fenwick_down.update(num_distinct - source_rank + 1, dp[source][1])

            # Track the global best across both states.
            result = max(result, dp[i][0], dp[i][1])

        return result
```

## Time and Space Complexity

- **Time Complexity**: `O(n log n)`, where `n` is the length of the array `nums`.

  The analysis is as follows:
  - Building the sorted set of distinct values via `sorted(set(nums))` takes `O(n log n)`.
  - Constructing the `rank` dictionary takes `O(n)`.
  - The main loop iterates `n` times. In each iteration, the operations `fwt0.preSum`, `fwt1.preSum`, `fwt0.update`, and `fwt1.update` each take `O(log M)` time, where `M` is the number of distinct values in `nums` (`M ≤ n`).

  Thus the loop costs `O(n log M)`. Combined with the initial sort `O(n log n)`, since `M ≤ n`, the overall time complexity is `O(n log n)`. Following the reference convention where `M = max(nums)`, this is expressed as `O(n log M)` for the Fenwick Tree portion (assuming values are bounded by `M`).

- **Space Complexity**: `O(M)`, where `M` is the number of distinct values in `nums` (bounded by `max(nums)`).

  The analysis is as follows:
  - The `stl` list and `rank` dictionary store the distinct values, requiring `O(M)` space.
  - The two Fenwick Trees `fwt0` and `fwt1` each use arrays of size `O(M)`.
  - The `dp` array uses `O(n)` space.

  Since `M ≤ n`, the dominant term gives an overall space complexity of `O(M)` (or equivalently `O(n)` in the worst case where all values are distinct).

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Using a Sum-Based Fenwick Tree Initialized to `0` with Negative Values

The most dangerous pitfall in this solution is the **interaction between the `0`-initialized Fenwick Tree and negative numbers**.

The `FenwickTree` is built with `self.tree = [0] * (size + 1)`, and `query_prefix_max` starts with `result = 0`. This silently assumes that **`0` is a safe "nothing found" sentinel** — i.e., that any real stored value is strictly greater than `0`.

This assumption breaks the moment `nums` contains negative values. Consider:

```python
nums = [-5, -3, -8]
k = 1
```

A valid valley→peak extension would want `dp[·][1] = (-5) + (-3) = -8`. But during the transition:

```python
dp[i][1] = max(dp[i][1], fenwick_up.query_prefix_max(current_rank - 1) + nums[i])
```

`query_prefix_max` may return `0` (the sentinel) instead of the true stored negative value (e.g., `-5`). Adding `nums[i]` to a phantom `0` produces a result that **was never built from a real predecessor**, corrupting the DP. Worse, the `update` may refuse to overwrite the initial `0` because `max(0, -5) = 0`, so negative valley/peak states are *never even stored*.

**Solution:** Initialize the tree and query sentinel with negative infinity, and treat "no predecessor" explicitly so phantom transitions are impossible:

```python
NEG_INF = float("-inf")

class FenwickTree:
    def __init__(self, size: int) -> None:
        self.size = size
        self.tree = [NEG_INF] * (size + 1)

    def update(self, index: int, value: int) -> None:
        while index <= self.size:
            self.tree[index] = max(self.tree[index], value)
            index += index & (-index)

    def query_prefix_max(self, position: int) -> int:
        result = NEG_INF
        while position >= 1:
            result = max(result, self.tree[position])
            position -= position & (-position)
        return result
```

Then guard the transition so a missing predecessor (`NEG_INF`) cannot pollute the DP:

```python
best_smaller = fenwick_up.query_prefix_max(current_rank - 1)
if best_smaller != NEG_INF:
    dp[i][1] = max(dp[i][1], best_smaller + nums[i])

best_larger = fenwick_down.query_prefix_max(num_distinct - current_rank)
if best_larger != NEG_INF:
    dp[i][0] = max(dp[i][0], best_larger + nums[i])
```

This way, an empty query contributes nothing instead of a spurious `0`.

---

### Pitfall 2: Off-by-One in the Reversed Coordinate Mapping

The "valley" transition relies on querying **strictly greater** values via reversed coordinates. It is extremely easy to mismatch the `update` offset and the `query` offset, since one uses `num_distinct - rank + 1` and the other uses `num_distinct - rank`.

- **Update** (storing peak state): `fenwick_down.update(num_distinct - source_rank + 1, ...)`
- **Query** (finding strictly larger): `fenwick_down.query_prefix_max(num_distinct - current_rank)`

If you accidentally make these symmetric (both `+1` or both without `+1`), you will either:
- Include values **equal** to `nums[i]` (violating the **strict** alternation rule), or
- Exclude the largest legitimate value, missing valid transitions.

**Verification:** A value `v` with rank `r` maps to reversed position `num_distinct - r + 1`. The largest reversed position corresponds to the smallest value. To fetch all values strictly greater than `nums[i]` (rank `current_rank`), those have ranks `current_rank + 1 ... num_distinct`, which map to reversed positions `1 ... num_distinct - current_rank`. Hence the query bound is exactly `num_distinct - current_rank` — confirming the asymmetry is intentional, not a bug. Always sanity-check with a tiny example like `nums = [1, 2]` before trusting the indices.

---

### Pitfall 3: Misaligning the Delayed Insertion Window

The gap constraint is enforced *implicitly* by the offset between querying (at index `i`) and inserting (index `i - k + 1`). A subtle trap is reasoning that the query at `i` sees the element at `i - k + 1`, when in fact the **insertion for `source = i - k + 1` happens *after* the query block in the same iteration**.

So during iteration `i`:
1. The query runs first → it sees everything inserted up to iteration `i - 1`, whose latest source was `(i - 1) - k + 1 = i - k`.
2. *Then* `source = i - k + 1` is inserted.

This means the closest usable predecessor is at distance exactly `i - (i - k) = k` ✓.

**Pitfall:** If you reorder the two blocks (insert before query), the closest predecessor becomes distance `k - 1`, **violating** the constraint. The ordering — *query first, insert second* — is load-bearing and must not be swapped.

---

### Pitfall 4: Empty / Single-Element Initialization

`result = nums[0]` correctly seeds the answer with a length-`1` subsequence (always valid). A common mistake is initializing `result = 0`, which:
- Returns `0` for an all-negative array instead of the maximum single element (e.g., `nums = [-3, -1, -7]` should return `-1`, not `0`).

Always initialize to a guaranteed-valid candidate (`nums[0]`), never to a neutral `0`.

