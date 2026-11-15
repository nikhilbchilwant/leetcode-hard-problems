# 3691. Maximum Total Subarray Value II

> Source: https://algo.monster/liteproblems/3691

## Problem Statement

You are given an integer array `nums` of length `n` and an integer `k`.

Your task is to select **exactly** `k` **distinct** non-empty subarrays `nums[l..r]` from `nums`. Two important rules apply to the selection:

- Subarrays **may overlap** with each other.
- The exact same subarray (identical `l` and `r`) **cannot** be chosen more than once.

Each subarray `nums[l..r]` has a **value**, defined as:

`max(nums[l..r]) - min(nums[l..r])`

In other words, the value of a subarray is the difference between its largest and smallest elements.

The **total value** of your selection is the sum of the values of all `k` chosen subarrays.

Return the **maximum** possible total value you can achieve.

**Key points to understand:**

- A subarray is a contiguous portion of the array, so it is uniquely identified by its left boundary `l` and right boundary `r`.
- An array of length `n` has `n * (n + 1) / 2` distinct subarrays in total, and you must pick exactly `k` of them.
- Since you want to maximize the sum, the goal is essentially to find the `k` subarrays with the largest values (max-min differences) and add those values together.
- A subarray consisting of a single element always has a value of `0`, because its maximum and minimum are the same element.

**Example:**

Suppose `nums = [1, 3, 2]` and `k = 2`.

- The subarray `[1, 3, 2]` has value `3 - 1 = 2`.
- The subarray `[1, 3]` has value `3 - 1 = 2`.
- The subarray `[3, 2]` has value `3 - 2 = 1`.

Choosing the two subarrays with the largest values, `[1, 3, 2]` and `[1, 3]`, gives a total value of `2 + 2 = 4`, which is the maximum achievable.

## Intuition

The brute-force idea would be to compute the value of every possible subarray, sort them all, and take the top `k`. But there are `n * (n + 1) / 2` subarrays, which can be far too many to enumerate and sort when `n` is large. We need a way to find the `k` largest values **without** generating all of them.

The key observation is a **monotonicity property**. Fix the left boundary `l` and let the right boundary `r` grow to the right. As the subarray extends:

- The maximum `max(nums[l..r])` can only **stay the same or increase**.
- The minimum `min(nums[l..r])` can only **stay the same or decrease**.

Therefore, the value `max(nums[l..r]) - min(nums[l..r])` is **non-decreasing** as `r` grows. For every fixed `l`, the largest value is always achieved at `r = n - 1`.

This restructures the problem nicely: instead of one giant unordered pile of subarray values, we now have `n` **sorted (non-decreasing) sequences** — one per left endpoint `l`. The task becomes a classic one:

> Given `n` sorted sequences, find the sum of the top `k` elements overall.

This is exactly the pattern of "merge `k` sorted lists" solved with a **max-heap**:

1. Push the largest element of each sequence into the heap — that is, push `(value of nums[l..n-1], l, n-1)` for every `l`.
2. Pop the heap `k` times. Each pop gives the current global maximum among all remaining candidates. After popping `(val, l, r)`, the next-largest candidate from that same sequence is the subarray `nums[l..r-1]` (if `r > l`), so push it in.

This works because of the monotonicity: when we pop `(l, r)`, every longer subarray with the same `l` has already been considered, and `(l, r-1)` is guaranteed to be the next candidate from that sequence with a value no greater than the popped one. So the heap always contains the correct frontier of candidates, and each pop is guaranteed to be the true next-largest value.

The last piece is efficiency: each heap operation requires knowing `max(nums[l..r]) - min(nums[l..r])` for an arbitrary range. Computing this naively would cost `O(n)` per query. Since the array never changes, we can precompute a **Sparse Table**, which answers any range-max and range-min query in `O(1)` after `O(n log n)` preprocessing.

Putting it together: the heap holds at most `n` entries, we perform `k` pops, and each step does `O(1)` range queries — giving roughly `O((n + k) log n)` total time, which avoids ever touching all `O(n^2)` subarrays.

**Learn more about [Greedy](/problems/greedy_intro), [Segment Tree](/problems/segment_tree_intro) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

The solution combines two components: a **Sparse Table** for `O(1)` range max/min queries, and a **max-heap** to greedily extract the top `k` subarray values.

### Step 1: Build the Sparse Table

The `SparseTableRMQ` class precomputes answers for range maximum and range minimum queries using the doubling technique.

- `f_max[i][j]` stores the maximum of the range starting at index `i` with length `2^j`, i.e., `nums[i..i + 2^j - 1]`.
- `f_min[i][j]` stores the minimum of the same range.

**Initialization (base case `j = 0`):** a range of length `1` is just the element itself:

```python
self.f_max[i][0] = data[i]
self.f_min[i][0] = data[i]
```

**Transition:** a range of length `2^j` is split into two halves of length `2^(j-1)`:

```python
self.f_max[i][j] = max(self.f_max[i][j - 1], self.f_max[i + (1 << (j - 1))][j - 1])
self.f_min[i][j] = min(self.f_min[i][j - 1], self.f_min[i + (1 << (j - 1))][j - 1])
```

We also precompute a logarithm table `lg`, where `lg[i] = floor(log2(i))`, so queries avoid calling a log function:

```python
self.lg[i] = self.lg[i >> 1] + 1
```

**Querying `[l, r]`:** let `k = lg[r - l + 1]`. Two ranges of length `2^k` — one starting at `l` and one ending at `r` — together cover `[l, r]` (they may overlap, which is harmless for max/min):

```python
def query_max(self, l, r):
    k = self.lg[r - l + 1]
    return max(self.f_max[l][k], self.f_max[r - (1 << k) + 1][k])
```

Preprocessing takes `O(n log n)` time and space; each query takes `O(1)`.

### Step 2: Initialize the Max-Heap

For each left endpoint `l`, the largest value among all subarrays starting at `l` is achieved by the longest one, `nums[l..n-1]` (due to the monotonicity property). We push all `n` of these into the heap:

```python
pq = []
for l in range(n):
    val = st.query_max(l, n - 1) - st.query_min(l, n - 1)
    heappush(pq, (-val, l, n - 1))
```

Python's `heapq` is a min-heap, so values are negated to simulate a max-heap. Each entry is a tuple `(-val, l, r)` recording the subarray's value and its boundaries.

### Step 3: Greedily Extract the Top `k` Values

We pop from the heap exactly `k` times. Each pop yields the current global maximum among all remaining candidate subarrays:

```python
ans = 0
for _ in range(k):
    val, l, r = heappop(pq)
    ans += -val
    if r > l:
        val = st.query_max(l, r - 1) - st.query_min(l, r - 1)
        heappush(pq, (-val, l, r - 1))
return ans
```

After popping `(val, l, r)`:

- Add `val` to the answer.
- If `r > l`, the sequence for left endpoint `l` still has shorter subarrays left. The next candidate is `nums[l..r-1]`, whose value is no greater than the one just popped. Compute it with the sparse table and push it in.
- If `r == l`, the sequence for this `l` is exhausted (the single-element subarray with value `0` was the last one).

This is the classic "merge `n` sorted lists, take top `k`" pattern: the heap always holds at most one *frontier* candidate per sequence, and monotonicity guarantees that no skipped subarray can ever exceed a popped one.

### Complexity Analysis

- **Time:** `O((n + k) log n)` — building the sparse table costs `O(n log n)`, the initial heap construction performs `n` pushes, and the main loop performs `k` pops/pushes, each costing `O(log n)` with `O(1)` range queries.
- **Space:** `O(n log n)` for the sparse table, plus `O(n)` for the heap.

## Example Walkthrough

Let's trace the algorithm with `nums = [1, 3, 2]` and `k = 3`.

**All 6 subarrays and their values (for reference):**

| Subarray | `(l, r)` | max | min | Value |
|---|---|---|---|---|
| `[1]` | `(0, 0)` | 1 | 1 | 0 |
| `[3]` | `(1, 1)` | 3 | 3 | 0 |
| `[2]` | `(2, 2)` | 2 | 2 | 0 |
| `[1, 3]` | `(0, 1)` | 3 | 1 | 2 |
| `[3, 2]` | `(1, 2)` | 3 | 2 | 1 |
| `[1, 3, 2]` | `(0, 2)` | 3 | 1 | 2 |

The top 3 values are `2 + 2 + 1 = 5` — this is the answer the algorithm should produce **without** enumerating all six subarrays.

**Step 1: Build the Sparse Table**

- Log table: `lg[1] = 0`, `lg[2] = 1`, `lg[3] = 1`.
- Length-1 ranges (`j = 0`): `f_max[i][0] = f_min[i][0] = nums[i]` → `[1, 3, 2]`.
- Length-2 ranges (`j = 1`):
  - `f_max[0][1] = max(1, 3) = 3`, `f_min[0][1] = 1`
  - `f_max[1][1] = max(3, 2) = 3`, `f_min[1][1] = 2`

A query on `[0, 2]` uses `k = lg[3] = 1`: it combines the two length-2 ranges `[0, 1]` and `[1, 2]`, giving `max(3, 3) = 3` and `min(1, 2) = 1` in `O(1)`.

**Step 2: Initialize the max-heap**

For each left endpoint `l`, push the longest subarray `nums[l..2]` (the largest candidate for that `l`, by monotonicity):

- `l = 0`: query `[0, 2]` → `3 − 1 = 2` → push `(-2, 0, 2)`
- `l = 1`: query `[1, 2]` → `3 − 2 = 1` → push `(-1, 1, 2)`
- `l = 2`: query `[2, 2]` → `2 − 2 = 0` → push `(0, 2, 2)`

Heap (as a max-heap on value): `{(2, 0, 2), (1, 1, 2), (0, 2, 2)}`.

**Step 3: Pop `k = 3` times**

| Pop # | Popped `(val, l, r)` | Running `ans` | Pushed next (shrink `r` by 1) |
|---|---|---|---|
| 1 | `(2, 0, 2)` | `0 + 2 = 2` | Query `[0, 1]` → `3 − 1 = 2` → push `(2, 0, 1)` |
| 2 | `(2, 0, 1)` | `2 + 2 = 4` | Query `[0, 0]` → `0` → push `(0, 0, 0)` |
| 3 | `(1, 1, 2)` | `4 + 1 = 5` | Query `[1, 1]` → `0` → push `(0, 1, 1)` |

Notice what happened:

- After popping `[1, 3, 2]` (value 2), its shorter sibling `[1, 3]` immediately entered the heap and was popped next — the monotonicity guarantee at work: shrinking `r` can never increase the value, so the frontier candidate per `l` is always sufficient.
- The single-element subarrays (value 0) entered the heap but were never popped, since better candidates remained.
- The remaining `O(n²)` subarrays were never even generated.

**Result:** `ans = 5`, matching the brute-force answer of selecting `[1, 3, 2]`, `[1, 3]`, and `[3, 2]`.

## Python Solution

```python
from typing import List
from heapq import heappush, heappop


class SparseTableRMQ:
    """Sparse Table supporting O(1) range max/min queries after O(n log n) preprocessing."""

    def __init__(self, data: List[int]):
        self.n = len(data)
        # Maximum power of two needed to cover any interval length.
        self.max_log = self.n.bit_length() + 1

        # f_max[i][j]: maximum of the interval starting at i with length 2^j.
        # f_min[i][j]: minimum of the interval starting at i with length 2^j.
        self.f_max = [[0] * self.max_log for _ in range(self.n)]
        self.f_min = [[0] * self.max_log for _ in range(self.n)]

        # Precompute floor(log2(x)) for every interval length 1..n.
        self.lg = [0] * (self.n + 1)
        for i in range(2, self.n + 1):
            self.lg[i] = self.lg[i >> 1] + 1

        # Base case: intervals of length 2^0 = 1 are the elements themselves.
        for i in range(self.n):
            self.f_max[i][0] = data[i]
            self.f_min[i][0] = data[i]

        # Build the table by doubling: an interval of length 2^j is the
        # combination of two adjacent intervals of length 2^(j-1).
        for j in range(1, self.max_log):
            for i in range(self.n - (1 << j) + 1):
                self.f_max[i][j] = max(
                    self.f_max[i][j - 1], self.f_max[i + (1 << (j - 1))][j - 1]
                )
                self.f_min[i][j] = min(
                    self.f_min[i][j - 1], self.f_min[i + (1 << (j - 1))][j - 1]
                )

    def query_max(self, left: int, right: int) -> int:
        """Return the maximum value in the inclusive range [left, right]."""
        k = self.lg[right - left + 1]
        # Cover [left, right] with two (possibly overlapping) blocks of length 2^k.
        return max(self.f_max[left][k], self.f_max[right - (1 << k) + 1][k])

    def query_min(self, left: int, right: int) -> int:
        """Return the minimum value in the inclusive range [left, right]."""
        k = self.lg[right - left + 1]
        # Cover [left, right] with two (possibly overlapping) blocks of length 2^k.
        return min(self.f_min[left][k], self.f_min[right - (1 << k) + 1][k])


class Solution:
    def maxTotalValue(self, nums: List[int], k: int) -> int:
        n = len(nums)
        sparse_table = SparseTableRMQ(nums)

        # Max-heap (simulated with negated values) of candidate subarrays.
        # Key observation: for a fixed left endpoint, the value
        # (max - min) is non-increasing as the right endpoint shrinks.
        # So for each left endpoint, the widest subarray [left, n-1]
        # gives the largest value; we push those first.
        max_heap = []
        for left in range(n):
            value = sparse_table.query_max(left, n - 1) - sparse_table.query_min(left, n - 1)
            heappush(max_heap, (-value, left, n - 1))

        ans = 0
        # Pop the k largest values; after taking [left, right], the next-best
        # candidate with the same left endpoint is [left, right - 1].
        for _ in range(k):
            neg_value, left, right = heappop(max_heap)
            ans += -neg_value
            if right > left:
                value = sparse_table.query_max(left, right - 1) - sparse_table.query_min(left, right - 1)
                heappush(max_heap, (-value, left, right - 1))
        return ans
```

## Time and Space Complexity

- **Time Complexity:** `O(n log n + k log n)`, where `n` is the length of the array `nums`.
  - **Sparse Table Construction:** Building `f_max` and `f_min` requires filling a table of size `n × O(log n)`, where each entry is computed in `O(1)` time, costing `O(n log n)` in total. The `lg` array is precomputed in `O(n)`.
  - **Heap Initialization:** For each left endpoint `l` (there are `n` of them), one RMQ query is performed in `O(1)` and one `heappush` is performed in `O(log n)`, costing `O(n log n)` in total.
  - **Extracting the Top-k Values:** The loop runs `k` times; each iteration performs one `heappop` and at most one `heappush` on a heap of size at most `n` (each in `O(log n)`), plus `O(1)` RMQ queries, costing `O(k log n)` in total.
  - Summing these parts gives `O(n log n) + O(n log n) + O(k log n) = O(n log n + k log n)`.

- **Space Complexity:** `O(n log n)`, where `n` is the length of the array `nums`.
  - The sparse tables `f_max` and `f_min` each occupy `O(n log n)` space, which dominates.
  - The `lg` array takes `O(n)` space, and the heap `pq` holds at most `n` entries, i.e., `O(n)` space.
  - Therefore, the overall space usage is `O(n log n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Assuming subarray values are independent per left endpoint without the monotonicity guarantee

The heap-based "merge n sorted lists" approach is only correct because of a key property: **for a fixed left endpoint `l`, the value `max(nums[l..r]) - min(nums[l..r])` is non-decreasing as `r` grows** (extending a range can only raise the max or lower the min, never the opposite). A common mistake is to apply this pattern to a different value function (e.g., subarray sum, which is *not* monotonic in `r`) or to enumerate candidates in an order that breaks monotonicity (e.g., shrinking from the left while the heap entries are keyed by left endpoint). If the per-sequence values are not sorted, the heap no longer guarantees that the `k` popped values are the global top `k`.

**Solution:** Verify the monotonicity argument before reusing this pattern. Here, each "sorted list" is the family of subarrays `[l, n-1], [l, n-2], ..., [l, l]` for a fixed `l`, whose values are non-increasing. After popping `(l, r)`, push only its direct successor `(l, r-1)` — never skip ahead or push multiple successors, or you risk duplicates.

### Pitfall 2: Pushing duplicate subarrays into the heap

If, after popping `(l, r)`, you push **both** `(l, r-1)` and `(l+1, r)` (a tempting "expand in both directions" idea borrowed from other problems), the same subarray can enter the heap via multiple paths — e.g., `(l+1, r-1)` is reachable from both `(l, r-1)` and `(l+1, r)`. Since the problem forbids choosing the identical subarray twice, duplicates inflate the answer.

**Solution:** Either stick to the one-dimensional successor scheme used in this code (each `l` owns its own chain, and only `(l, r-1)` is pushed), or, if expanding in two directions, maintain a `visited` set of `(l, r)` pairs and skip already-seen entries before pushing.

### Pitfall 3: Computing values eagerly for all `O(n²)` subarrays

A naive approach precomputes the value of every subarray, sorts them, and takes the top `k`. With `n` up to typical constraint sizes (e.g., `10^5`), this requires `O(n²)` time and memory and will TLE/MLE.

**Solution:** Use the lazy heap approach shown: keep at most one frontier candidate per left endpoint, so the heap never exceeds `n` entries and total work is `O((n + k) log n)`.

### Pitfall 4: Sparse table indexing and sizing errors

Two frequent off-by-one mistakes when hand-rolling the sparse table:

- **Inner loop bound:** the start index `i` must satisfy `i + 2^j - 1 <= n - 1`, i.e., iterate `for i in range(n - (1 << j) + 1)`. Writing `range(n - (1 << j))` silently drops the last valid interval, producing wrong query results near the array's right edge.
- **Query block placement:** the second block must *end* at `right`, so it starts at `right - (1 << k) + 1`. Forgetting the `+ 1` shifts the block and may read out of the covered range.

Note that overlapping blocks are harmless **only for idempotent operations** like max/min/gcd. Reusing this exact two-block query for sums would double-count the overlap.

**Solution:** Carefully derive both bounds from the invariant "block of length `2^j` starting at `i` covers `[i, i + 2^j - 1]`", and test queries at the boundaries (`l = 0`, `r = n - 1`, and `l == r`).

### Pitfall 5: Forgetting that Python's `heapq` is a min-heap

Pushing `(value, l, r)` instead of `(-value, l, r)` extracts the `k` *smallest* values, yielding a drastically wrong (often zero-heavy) answer. Conversely, after negating, forgetting to negate back when accumulating (`ans += -neg_value`) produces a negative total.

**Solution:** Negate consistently on both push and pop, or wrap values in a comparator-flipping class. A quick sanity check: the answer must be non-negative, and with `k = 1` it must equal `max(nums) - min(nums)`.

### Pitfall 6: Not terminating the chain at single-element subarrays

After popping `(l, r)` with `r == l`, pushing `(l, r - 1)` creates an invalid subarray with `r < l`, which either crashes the sparse table query or silently adds garbage values.

**Solution:** Guard the push with `if right > left:` as the code does. Single-element subarrays (value `0`) are valid selections — they are still counted toward `k` when popped — but they have no successor.

