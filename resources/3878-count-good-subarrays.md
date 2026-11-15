# 3878. Count Good Subarrays

> Source: https://algo.monster/liteproblems/3878

## Problem Statement

You are given an integer array `nums`.

A **subarray** is a contiguous, non-empty sequence of elements within the array.

A subarray is called **good** if the **bitwise OR** of all its elements is equal to **at least one** element present in that subarray.

Your task is to return the number of good subarrays in `nums`.

Here, the bitwise OR of two integers `a` and `b` is denoted by `a | b`.

For example, consider a subarray whose elements OR together to produce some value `v`. This subarray is "good" only if `v` itself appears as one of the elements inside that subarray. Since the bitwise OR operation can only set bits (never clear them), the OR of all elements in a subarray will always be greater than or equal to each individual element. This means the OR result equals one of the subarray's elements precisely when that element already contains all the bits of every other element in the subarray — in other words, every other element is a "subset" (in terms of bits) of that element.

You need to count how many such good subarrays exist in `nums`.

## Intuition

The brute-force idea would be to look at every subarray, compute the bitwise OR of all its elements, and check whether that OR value equals one of the elements inside. But there are `O(n^2)` subarrays, and computing the OR for each is costly, so we need a smarter way.

The key observation comes from a property of the bitwise OR operation: ORing numbers can only **turn bits on**, never off. This means the OR of all elements in a subarray is always **greater than or equal to** every single element in that subarray (in terms of which bits are set). Therefore, the OR result of a subarray can equal one of its elements only when that element already contains all the bits of every other element. In other words, that element is the "largest superset" of all the rest — every other element `nums[k]` must satisfy `nums[k] | nums[i] == nums[i]`.

This insight flips the problem around. Instead of asking *"what is the OR of this subarray?"*, we ask *"for each element `nums[i]`, how many subarrays have `nums[i]` as their OR result?"* This is a **contribution enumeration** strategy: we fix `nums[i]` as the OR value and count the subarrays it accounts for.

For `nums[i]` to be the OR of a subarray, **all** elements in that subarray must be bit-subsets of `nums[i]`. So for each index `i`, we want to find how far we can extend to the left and to the right while every element stays a subset of `nums[i]`. If `l[i]` is the boundary just before the valid left region and `r[i]` is the boundary just after the valid right region, then:

- We can choose the left endpoint among `i - l[i]` positions (from `l[i] + 1` up to `i`).
- We can choose the right endpoint among `r[i] - i` positions (from `i` up to `r[i] - 1`).

This gives `(i - l[i]) * (r[i] - i)` subarrays where `nums[i]` serves as the OR value. Summing this over all `i` gives the total count.

To find these left and right boundaries efficiently, we use a **monotonic [stack](/problems/stack_intro)**. As we scan the array, the stack helps us quickly skip over elements that are already subsets of the current value, letting us determine each boundary in amortized constant time. One subtle detail is handling **equal or duplicate** values carefully so that no subarray is counted twice — this is why the left-boundary condition uses a strict comparison (`nums[stk[-1]] < x`) while the right-boundary condition does not, breaking ties consistently to one side.

**Learn more about [Stack](/problems/stack_intro) and [Monotonic Stack](/problems/mono_stack_intro) patterns.**

## Solution Approach

### Solution 1: Monotonic [Stack](/problems/stack_intro) + Contribution Enumeration

We enumerate each element `nums[i]` as the bitwise OR result of a subarray, and count how many subarrays have a bitwise OR exactly equal to `nums[i]`.

If the bitwise OR of a subarray equals `nums[i]`, then every element `nums[k]` in that subarray must satisfy:

`nums[k] | nums[i] == nums[i]`

That is, every element in the subarray must be a **bit-subset** of `nums[i]`. So for each index `i`, we need to find the left boundary `l[i]` and right boundary `r[i]`, such that all elements in the open interval `(l[i], r[i])` are subsets of `nums[i]`, while `nums[l[i]]` and `nums[r[i]]` are not. The number of subarrays with `nums[i]` as the bitwise OR result is then `(i - l[i]) * (r[i] - i)`.

We use a **monotonic [stack](/problems/stack_intro)** to compute these boundaries. To avoid counting the same subarray more than once when there are duplicate or equal values, we break ties asymmetrically: the left side uses a **strict** condition, while the right side uses a **non-strict** condition.

**Step 1: Compute the left boundaries `l`.**

We initialize `l` as an array of `-1` and process the array from left to right with a [stack](/problems/stack_intro):

- For each index `i` with value `x`, we pop from the [stack](/problems/stack_intro) while the top element `nums[stk[-1]]` is both **strictly less than** `x` and a subset of `x`, i.e. `nums[stk[-1]] < x and (nums[stk[-1]] | x) == x`.
- After popping, `l[i]` is set to the current top of the stack (or `-1` if the stack is empty). This is the nearest index to the left that is **not** absorbed under `nums[i]`.
- We then push `i` onto the stack.

```python
l = [-1] * n
stk = []
for i, x in enumerate(nums):
    while stk and nums[stk[-1]] < x and (nums[stk[-1]] | x) == x:
        stk.pop()
    l[i] = stk[-1] if stk else -1
    stk.append(i)
```

**Step 2: Compute the right boundaries `r`.**

We initialize `r` as an array of `n` and process the array from right to left with a fresh [stack](/problems/stack_intro):

- For each index `i`, we pop from the [stack](/problems/stack_intro) while the top element is a subset of `nums[i]`, i.e. `(nums[stk[-1]] | nums[i]) == nums[i]`. Note this condition is **non-strict** — it pops even on equality, which complements the strict left condition to prevent double counting.
- After popping, `r[i]` is set to the current top of the stack (or `n` if the stack is empty), the nearest index to the right not absorbed under `nums[i]`.
- We then push `i` onto the stack.

```python
r = [n] * n
stk = []
for i in range(n - 1, -1, -1):
    while stk and (nums[stk[-1]] | nums[i]) == nums[i]:
        stk.pop()
    r[i] = stk[-1] if stk else n
    stk.append(i)
```

**Step 3: Sum up the contributions.**

For each index `i`, the number of good subarrays where `nums[i]` is the OR result is the product of the count of valid left endpoints and valid right endpoints:

`(i - l[i]) * (r[i] - i)`

Summing over all indices gives the final answer:

```python
return sum((i - l[i]) * (r[i] - i) for i in range(n))
```

**Complexity Analysis.**

- **Time complexity:** `O(n)`, where `n` is the length of `nums`. Each element is pushed and popped from each [stack](/problems/stack_intro) at most once, so the [monotonic stack](/problems/mono_stack_intro) passes are linear, and the final summation is also linear.
- **Space complexity:** `O(n)`, for the boundary arrays `l` and `r` and the stack.

## Example Walkthrough

Consider `nums = [1, 1, 2]`. The expected answer is **4** (the good subarrays are `[1]@0`, `[1]@1`, `[2]@2`, and `[1,1]@(0,1)`; the subarrays containing both a `1` and a `2` OR to `3`, which never appears, so they are not good).

**Step 1 — Left boundaries `l` (left→right, strict pop `nums[stk[-1]] < x and (nums[stk[-1]] | x) == x`):**

- `i=0, x=1`: stack empty → `l[0] = -1`. Stack: `[0]`
- `i=1, x=1`: top `1 < 1`? No → `l[1] = 0`. Stack: `[0,1]`
- `i=2, x=2`: top `1 < 2` yes but `1|2 == 2`? No → `l[2] = 1`. Stack: `[0,1,2]`

Result: `l = [-1, 0, 1]`

**Step 2 — Right boundaries `r` (right→left, non-strict pop `(nums[stk[-1]] | nums[i]) == nums[i]`):**

- `i=2, x=2`: stack empty → `r[2] = 3`. Stack: `[2]`
- `i=1, x=1`: top `2|1 == 1`? No → `r[1] = 2`. Stack: `[2,1]`
- `i=0, x=1`: top `1|1 == 1`? Yes → pop; new top `2|1 == 1`? No → `r[0] = 2`. Stack: `[2,0]`

Result: `r = [2, 2, 3]`

**Step 3 — Sum contributions `(i - l[i]) * (r[i] - i)`:**

| i | nums[i] | l[i] | r[i] | (i−l[i]) | (r[i]−i) | contribution |
|---|---------|------|------|----------|----------|--------------|
| 0 | 1       | −1   | 2    | 1        | 2        | 2            |
| 1 | 1       | 0    | 2    | 1        | 1        | 1            |
| 2 | 2       | 1    | 3    | 1        | 1        | 1            |

Total = `2 + 1 + 1 = 4`, matching the brute-force count.

**Why tie-breaking matters:** The subarray `[1,1]@(0,1)` is counted once, under `i=0`. It is not recounted under `i=1` because the strict left condition stops `l[1]` at `0`, restricting `i=1`'s left endpoints to index `1` only. Pairing a strict left condition with a non-strict right condition ensures each subarray of equal values is attributed to exactly one index.

## Python Solution

```python
class Solution:
    def countGoodSubarrays(self, nums: list[int]) -> int:
        n = len(nums)

        # left[i] : index of the nearest element to the LEFT of i that is the
        #           "boundary" for i (exclusive). Used to count how many
        #           subarrays ending in the left direction keep nums[i] dominant
        #           under the OR/bitwise-subset relationship.
        left = [-1] * n
        stack = []
        for i, x in enumerate(nums):
            # Pop elements that are strictly smaller than x AND are a bitwise
            # subset of x (i.e. (nums[top] | x) == x). These elements cannot be
            # a valid left boundary for the current element.
            while stack and nums[stack[-1]] < x and (nums[stack[-1]] | x) == x:
                stack.pop()
            # The remaining top of the stack (if any) is the left boundary.
            left[i] = stack[-1] if stack else -1
            stack.append(i)

        # right[i] : index of the nearest element to the RIGHT of i that is the
        #            "boundary" for i (exclusive).
        right = [n] * n
        stack = []
        for i in range(n - 1, -1, -1):
            # Pop elements on the right that are a bitwise subset of nums[i]
            # ((nums[top] | nums[i]) == nums[i]), since they extend the valid
            # range for the current element.
            while stack and (nums[stack[-1]] | nums[i]) == nums[i]:
                stack.pop()
            # The remaining top of the stack (if any) is the right boundary.
            right[i] = stack[-1] if stack else n
            stack.append(i)

        # For each index i, the number of subarrays in which nums[i] acts as the
        # dominant element is (count of valid left endpoints) *
        # (count of valid right endpoints). Summing over all i gives the answer.
        return sum((i - left[i]) * (right[i] - i) for i in range(n))
```

## Time and Space Complexity

**Time Complexity: `O(n)`**

The code consists of three main parts:

1. **First loop (computing `l`)**: This iterates through all `n` elements. For each element, there is a `while` loop that pops from the stack `stk`. Although the inner `while` loop may execute multiple times in a single iteration, each index is pushed onto the stack exactly once and popped at most once across the entire loop. Therefore, the total number of push/pop operations is bounded by `O(n)`, giving this loop a total cost of `O(n)`.

2. **Second loop (computing `r`)**: By the same amortized analysis, this loop iterates through all `n` elements in reverse, with each index pushed and popped at most once. The total cost is also `O(n)`.

3. **Final summation**: The `sum((i - l[i]) * (r[i] - i) for i in range(n))` expression iterates over all `n` indices once, performing constant work per index, costing `O(n)`.

Combining these parts gives a total time complexity of `O(n) + O(n) + O(n) = O(n)`, where `n` is the length of the array `nums`.

**Space Complexity: `O(n)`**

The algorithm uses the following auxiliary data structures:

- The arrays `l` and `r`, each of size `n`, requiring `O(n)` space.
- The monotonic stack `stk`, which in the worst case can hold up to `n` indices simultaneously, requiring `O(n)` space.

Therefore, the overall space complexity is `O(n)`, where `n` is the length of the array `nums`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Symmetric Tie-Breaking Causes Double Counting

The most subtle and dangerous mistake in this problem is using the **same condition** on both the left and right passes. When duplicate or bit-equal elements exist, a single subarray can be counted multiple times (once for each equal element that could serve as the "OR result").

**Buggy version** — using a strict condition on *both* sides:

```python
# LEFT pass (correct: strict)
while stack and nums[stack[-1]] < x and (nums[stack[-1]] | x) == x:
    stack.pop()

# RIGHT pass (WRONG: also strict)
while stack and nums[stack[-1]] < nums[i] and (nums[stack[-1]] | nums[i]) == nums[i]:
    stack.pop()
```

**Why it fails.** Consider `nums = [2, 2]`. Both elements are equal, and the subarray `[2, 2]` has OR `= 2`, which appears in the subarray, so it is *good* — but it should be counted **exactly once**.

With strict conditions on both sides, neither the left pass (index 1 sees index 0 with `nums[0] < nums[1]` false) nor the right pass (index 0 sees index 1 with `nums[1] < nums[0]` false) absorbs the equal neighbor. As a result, both index 0 and index 1 claim the subarray `[2, 2]`, producing a count of 2 instead of 1.

**The fix.** Break ties **asymmetrically**: make exactly one side non-strict. The reference solution uses:

- **Left pass:** strict — `nums[stack[-1]] < x and (nums[stack[-1]] | x) == x`
- **Right pass:** non-strict — `(nums[stack[-1]] | nums[i]) == nums[i]` (pops even on equality)

This guarantees that among several bit-equal elements, only the **leftmost** one "owns" any subarray of equal values, so each good subarray is attributed to a unique pivot index.

---

### Pitfall 2: Forgetting the Subset Check and Treating It Like "Sum of Subarray Minimums/Maximums"

This problem resembles the classic *Sum of Subarray Minimums* template, so it is tempting to pop purely on the numeric comparison and drop the bitwise-subset guard:

```python
# WRONG: ignores the OR/subset relationship
while stack and nums[stack[-1]] < x:
    stack.pop()
```

**Why it fails.** A larger value is **not** automatically a bit-superset of a smaller one. For example, `4 (100)` is larger than `3 (011)`, but `3 | 4 = 7 ≠ 4`, so `3` is *not* a subset of `4`. Popping on magnitude alone would incorrectly extend the dominance region of `nums[i]` across elements it does not actually absorb, inflating the count.

**The fix.** Always combine the magnitude comparison with the subset check `(nums[stack[-1]] | x) == x`. The numeric `<` is only a tie-breaker; the **subset relation is the real correctness condition**.

---

### Pitfall 3: Off-by-One Errors in Boundary Initialization

A frequent slip is initializing `left` or `right` with the wrong sentinel, or mixing up the contribution formula.

```python
# WRONG sentinels / formula
left  = [0] * n      # should be -1
right = [n - 1] * n  # should be n
...
total += (i - left[i] + 1) * (right[i] - i + 1)  # wrong: +1 over-counts
```

**Why it fails.** The boundaries `left[i]` and `right[i]` point to the **exclusive** elements just outside the dominance region. The valid left endpoints are `left[i]+1 .. i` (that's `i - left[i]` choices) and the valid right endpoints are `i .. right[i]-1` (that's `right[i] - i` choices). Using `-1` and `n` as sentinels makes the formula `(i - left[i]) * (right[i] - i)` work uniformly even when no boundary exists.

**The fix.** Initialize `left = [-1] * n` and `right = [n] * n`, and use the product `(i - left[i]) * (right[i] - i)` **without** any `+1` adjustments.

---

### Pitfall 4: Integer Overflow (Language-Dependent)

In Python this is a non-issue since integers are arbitrary precision, but when porting to **Java/C++**, the result can exceed a 32-bit integer. For `n` up to `10^5`, the count of subarrays can approach `n*(n+1)/2 ≈ 5×10^9`, which overflows `int`.

**The fix.** Use a 64-bit accumulator (`long` in Java, `long long` in C++) for the running total and for the per-index product `(i - left[i]) * (right[i] - i)`.

