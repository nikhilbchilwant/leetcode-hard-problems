# 3739. Count Subarrays With Majority Element II

> Source: https://algo.monster/liteproblems/3739

## Problem Statement

You are given an integer array `nums` and an integer `target`.

Return the number of **subarrays** of `nums` in which `target` is the **majority element**.

The **majority element** of a subarray is the element that appears **strictly more than half** of the times in that subarray.

In other words, for a subarray of length `len`, `target` is the majority element if the number of times `target` appears within that subarray is strictly greater than `len / 2`.

**Key points to understand:**

- A **subarray** is a contiguous, non-empty portion of the array.
- You need to count how many such subarrays exist where `target` dominates by appearing more than half the time.
- Different subarrays (different start or end positions) are counted separately, even if they contain the same values.

**Example walkthrough:**

Suppose `nums = [1, 2, 3, 2, 2]` and `target = 2`. Consider the subarray `[2, 2]` (the last two elements). It has length `2`, and `target = 2` appears `2` times. Since `2 > 2 / 2 = 1`, `target` is the majority element here, so this subarray counts.

Now consider the subarray `[2, 3, 2]`. It has length `3`, and `target = 2` appears `2` times. Since `2 > 3 / 2 = 1.5`, `target` is the majority element, so this subarray counts as well.

However, a subarray like `[1, 2]` has length `2` with `target = 2` appearing only `1` time. Since `1` is not strictly greater than `2 / 2 = 1`, `target` is **not** the majority element, so this subarray does **not** count.

## Intuition

The first challenge is the phrase "strictly more than half." Counting occurrences directly for every possible subarray would be too slow, so we need a smarter way to express this condition.

The key observation is a **transformation trick**: since we only care whether each element equals `target` or not, we can convert the array into `+1` and `-1` values:

- If an element equals `target`, treat it as `+1`.
- If an element does not equal `target`, treat it as `-1`.

Why does this help? In a subarray of length `len`, suppose `target` appears `k` times. Then the non-`target` elements appear `len - k` times. The sum of the transformed subarray becomes:

`k * (+1) + (len - k) * (-1) = 2k - len`

The condition "`target` is the majority element" means `k > len / 2`, which is the same as `2k > len`, or `2k - len > 0`. So `target` being the majority is **exactly equivalent** to the transformed subarray having a **sum strictly greater than `0`**.

Now the problem becomes: *count the number of subarrays whose sum is strictly greater than `0`*. This is a classic problem that can be solved with **prefix sums**.

Let `s[i]` denote the [prefix sum](/problems/subarray_sum) of the first `i` transformed elements (with `s[0] = 0`). A subarray from index `j+1` to `i` has sum `s[i] - s[j]`. We want this to be greater than `0`, meaning `s[i] - s[j] > 0`, or equivalently `s[j] < s[i]`.

So for each ending position `i`, we need to know **how many earlier prefix sums `s[j]` are strictly less than the current [prefix sum](/problems/subarray_sum) `s[i]`**. If we add up this count over all positions, we get the final answer.

To count "how many previous values are less than the current value" efficiently as we scan through the array, we use a **Binary Indexed Tree (Fenwick Tree)**. It lets us insert prefix sums and query how many of them are below a given value, each in `O(log n)` time.

One small detail: prefix sums can range from `-n` to `n` (negative values are possible). Since a Binary Indexed Tree only works with positive indices, we **shift every [prefix sum](/problems/subarray_sum) by `n + 1`**, mapping the range `[-n, n]` into `[1, 2n + 1]`. This way every value safely fits into the tree's index space. We start by inserting the shifted value of the initial prefix sum `s[0] = 0` (which becomes `n + 1`), and then process each element while updating and querying the tree.

**Learn more about [Segment Tree](/problems/segment_tree_intro), [Divide and Conquer](/problems/divide_and_conquer_intro), [Prefix Sum](/problems/subarray_sum) and [Merge Sort](/problems/merge_k_sorted_lists) patterns.**

## Solution Approach

We use a **Binary Indexed Tree (Fenwick Tree)** combined with the **[prefix sum](/problems/subarray_sum)** idea described above.

### Step 1: Build the Binary Indexed Tree

The prefix sums range from `-n` to `n`, and after shifting by `n + 1`, the values fall into the range `[1, 2n + 1]`. So we create a Binary Indexed Tree of size `2n + 1`:

```python
tree = BinaryIndexedTree(n * 2 + 1)
```

The tree supports two operations:

- `update(x, delta)`: add `delta` to the count at position `x`, and propagate upward using `x += x & -x`.
- `query(x)`: return the total count from position `1` to `x`, by accumulating downward using `x -= x & -x`.

Both operations run in `O(log n)` time.

### Step 2: Initialize the Starting [Prefix Sum](/problems/subarray_sum)

We let `s` represent the current running [prefix sum](/problems/subarray_sum) (already shifted). Initially, before processing any element, the prefix sum is `0`, which after shifting by `n + 1` becomes:

```python
s = n + 1
tree.update(s, 1)
```

This records that the [prefix sum](/problems/subarray_sum) `0` has occurred once.

### Step 3: Scan the Array

We iterate through each element `x` in `nums` and update the [prefix sum](/problems/subarray_sum) based on the `+1` / `-1` transformation:

```python
s += 1 if x == target else -1
```

After updating, `s` represents the [prefix sum](/problems/subarray_sum) at the current ending position (in shifted form).

### Step 4: Count Valid Subarrays Ending Here

For the current position, we want subarrays whose sum is strictly greater than `0`. As derived earlier, this means counting earlier prefix sums `s[j]` that are **strictly less than** the current `s`. In shifted terms, that is all recorded values in the range `[1, s - 1]`:

```python
ans += tree.query(s - 1)
```

The `query(s - 1)` returns how many previously inserted prefix sums are strictly less than `s`, which is exactly the number of valid subarrays ending at the current position.

### Step 5: Record the Current [Prefix Sum](/problems/subarray_sum)

After counting, we insert the current [prefix sum](/problems/subarray_sum) into the tree so it can be used by future positions:

```python
tree.update(s, 1)
```

### Step 6: Return the Answer

Once all elements are processed, `ans` holds the total number of subarrays in which `target` is the majority element:

```python
return ans
```

### Complexity Analysis

- **Time Complexity:** `O(n log n)`, since we process each of the `n` elements once, and each `update` and `query` operation on the Binary Indexed Tree takes `O(log n)` time.
- **Space Complexity:** `O(n)`, for the Binary Indexed Tree which stores up to `2n + 1` entries.

## Example Walkthrough

Let's trace through a small example to see the solution approach in action.

**Input:** `nums = [1, 2, 2]`, `target = 2`

**Setup:**
- `n = 3`, so the Binary Indexed Tree has size `2n + 1 = 7`.
- The shift amount is `n + 1 = 4`, mapping prefix sums from `[-3, 3]` to `[1, 7]`.
- Initialize `s = 4` (shifted value of prefix sum `0`) and `tree.update(4, 1)`.
- The tree now records that prefix sum `0` has appeared once. `ans = 0`.

**Transformation reminder:** element `== target` becomes `+1`, otherwise `-1`. So `[1, 2, 2]` transforms to `[-1, +1, +1]`.

---

**Step i = 0, element `x = 1`:**
- `x != target`, so `s += -1` → `s = 3` (this is the shifted prefix sum after the first element; the true prefix sum is `-1`).
- **Count:** `tree.query(s - 1) = tree.query(2)`. We ask: how many recorded prefix sums are strictly less than `3`? Only the value `4` is recorded, and `4` is not less than `3`. So the query returns `0`. `ans += 0` → `ans = 0`.
- **Record:** `tree.update(3, 1)`. Tree now holds prefix sums `{4: 1, 3: 1}`.

---

**Step i = 1, element `x = 2`:**
- `x == target`, so `s += 1` → `s = 4` (true prefix sum is `0`).
- **Count:** `tree.query(3)`. How many recorded prefix sums are strictly less than `4`? The recorded values are `4` and `3`; only `3` is less than `4`. So the query returns `1`. `ans += 1` → `ans = 1`.
  - This corresponds to the subarray `[2]` (just the element at index 1), where `target` appears once in a length-1 subarray — clearly the majority.
- **Record:** `tree.update(4, 1)`. Tree now holds `{4: 2, 3: 1}`.

---

**Step i = 2, element `x = 2`:**
- `x == target`, so `s += 1` → `s = 5` (true prefix sum is `1`).
- **Count:** `tree.query(4)`. How many recorded prefix sums are strictly less than `5`? Recorded values are `4` (count 2) and `3` (count 1), all less than `5`. So the query returns `3`. `ans += 3` → `ans = 4`.
  - These correspond to three subarrays ending at index 2:
    - `[2]` (index 2 alone) — majority.
    - `[2, 2]` (indices 1–2) — `2` appears twice in length 2, `2 > 1`, majority.
    - `[1, 2, 2]` (indices 0–2) — `2` appears twice in length 3, `2 > 1.5`, majority.
- **Record:** `tree.update(5, 1)`. Tree now holds `{5: 1, 4: 2, 3: 1}`.

---

**Result:** `ans = 4`.

Let's verify by listing every qualifying subarray directly:

| Subarray | Length | Count of `2` | Majority? |
|----------|--------|--------------|-----------|
| `[2]` (index 1) | 1 | 1 | ✅ `1 > 0.5` |
| `[2]` (index 2) | 1 | 1 | ✅ `1 > 0.5` |
| `[2, 2]` (1–2) | 2 | 2 | ✅ `2 > 1` |
| `[1, 2, 2]` (0–2) | 3 | 2 | ✅ `2 > 1.5` |

That's exactly **4** subarrays, matching our computed answer. Each time we processed an element, the Fenwick Tree query gave us the count of previous prefix sums smaller than the current one — which equals the number of valid subarrays ending at that position — and we accumulated those into `ans`.

## Python Solution

```python
from typing import List


class BinaryIndexedTree:
    """A Fenwick Tree (Binary Indexed Tree) supporting point updates and prefix-sum queries."""

    __slots__ = ("n", "c")

    def __init__(self, n: int) -> None:
        # n: the maximum index (size) of the tree.
        self.n = n
        # c: internal array holding partial sums; 1-indexed, so size is n + 1.
        self.c = [0] * (n + 1)

    def update(self, x: int, delta: int) -> None:
        # Add `delta` to position `x`, then propagate to all responsible parents.
        while x <= self.n:
            self.c[x] += delta
            x += x & -x  # move to the next index that includes position x.

    def query(self, x: int) -> int:
        # Return the prefix sum over [1, x].
        s = 0
        while x > 0:
            s += self.c[x]
            x -= x & -x  # move to the previous range that does not overlap.
        return s


class Solution:
    def countMajoritySubarrays(self, nums: List[int], target: int) -> int:
        # Count subarrays in which `target` is a strict majority element.
        #
        # Idea: map each element to +1 if it equals `target`, otherwise -1.
        # A subarray (l, r] has `target` as a strict majority iff its mapped
        # sum is positive, i.e. prefix[r] - prefix[l] > 0, equivalently
        # prefix[l] < prefix[r].
        #
        # So for each prefix value we count how many earlier prefixes are
        # strictly smaller, using a BIT for efficient counting.

        n = len(nums)

        # Prefix sums range from -n to n; shift by (n + 1) to keep indices
        # within [1, 2n + 1] for the 1-indexed BIT.
        tree = BinaryIndexedTree(n * 2 + 1)

        # `s` is the running, shifted prefix sum. The empty prefix maps to n + 1.
        s = n + 1
        tree.update(s, 1)  # register the empty prefix.

        ans = 0
        for x in nums:
            # Update the prefix sum: +1 when matching target, -1 otherwise.
            s += 1 if x == target else -1
            # Count earlier prefixes strictly smaller than the current one.
            ans += tree.query(s - 1)
            # Register the current prefix for future queries.
            tree.update(s, 1)

        return ans
```

## Time and Space Complexity

**Time Complexity:** `O(n log n)`, where `n` is the length of the array `nums`.

- The code iterates through each element of `nums` once, resulting in `n` iterations of the main loop.
- Within each iteration, two operations are performed on the Binary Indexed Tree (BIT): `tree.query(s - 1)` and `tree.update(s, 1)`.
- Both the `query` and `update` operations of a Binary Indexed Tree run in `O(log m)` time, where `m` is the size of the tree. Here the tree size is `2n + 1`, so each operation takes `O(log(2n + 1)) = O(log n)` time.
- Therefore, the overall time complexity is `n × O(log n) = O(n log n)`.

**Space Complexity:** `O(n)`, where `n` is the length of the array `nums`.

- The Binary Indexed Tree maintains an internal array `c` of size `2n + 2` (since it is initialized with `n * 2 + 1` and uses `n + 1` slots internally).
- This array dominates the auxiliary space usage, which is proportional to `n`, giving a space complexity of `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall: Off-by-One Error in the Strict Inequality Query

The most common mistake in this solution is querying `tree.query(s)` instead of `tree.query(s - 1)`.

**Why this happens:**

The condition for `target` being a strict majority is `prefix[r] - prefix[l] > 0`, which means `prefix[l] < prefix[r]` — a **strict** inequality. It's tempting to write `tree.query(s)` because we're "counting prefixes up to the current value," but `query(s)` includes prefixes that are **equal** to `s` (where the mapped subarray sum is exactly `0`).

A subarray with sum `0` has `target` appearing exactly half the time, which is **not** strictly more than half. Including these would overcount.

**Incorrect version:**

```python
ans += tree.query(s)  # WRONG: counts prefixes <= s, includes sum == 0 cases
```

**Correct version:**

```python
ans += tree.query(s - 1)  # counts prefixes strictly less than s
```

**Concrete example:**

Take `nums = [2, 1]` and `target = 2`. The mapped array is `[+1, -1]`.

- Empty prefix: `s = 0` (shifted), registered.
- After `2`: `s = +1`. The subarray `[2]` has sum `1 > 0`, valid. `query(s - 1)` correctly returns `1`.
- After `1`: `s = 0`. The subarray `[2, 1]` has sum `0`, **not** valid.
  - With `query(s - 1)`: returns count of prefixes strictly `< 0` → `0`. ✓ Correct.
  - With `query(s)`: returns count of prefixes `<= 0`, which includes the empty prefix at `0` → `1`. ✗ Wrongly counts `[2, 1]`.

The correct answer is `1`, but using `query(s)` would yield `2`.

---

### Secondary Pitfall: Incorrect Shift or Tree Sizing

Another subtle error is mis-sizing the BIT or choosing the wrong shift value.

**Why this happens:**

Prefix sums range from `-n` (all non-target) to `+n` (all target). To map these into the 1-indexed BIT domain `[1, 2n + 1]`, the shift must be `n + 1`:

- Minimum prefix `-n` → `-n + (n + 1) = 1` ✓
- Maximum prefix `+n` → `n + (n + 1) = 2n + 1` ✓

**Common mistakes:**

```python
s = n          # WRONG shift: -n maps to 0, which is invalid for a 1-indexed BIT
tree = BinaryIndexedTree(n * 2)  # WRONG size: max index 2n+1 exceeds the tree
```

If the shift is `n` instead of `n + 1`, the smallest prefix maps to index `0`. Calling `update(0, 1)` enters an **infinite loop** because `0 & -0 == 0`, so `x` never advances. Always verify both boundary cases map within `[1, 2n + 1]`.

**Correct setup:**

```python
s = n + 1
tree = BinaryIndexedTree(n * 2 + 1)
```

