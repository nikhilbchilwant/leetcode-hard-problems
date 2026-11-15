# 3398. Smallest Substring With Identical Characters I

> Source: https://algo.monster/liteproblems/3398

## Problem Statement

You have a binary string `s` consisting of only '0's and '1's, with length `n`. You're allowed to perform up to `numOps` operations on this string.

Each operation allows you to:
- Choose any index `i` (where `0 <= i < n`)
- Flip the character at that position (change '0' to '1' or '1' to '0')

Your goal is to minimize the length of the longest substring where all characters are identical (all '0's or all '1's).

For example:
- In string "111001", the longest substring with identical characters is "111" with length 3
- After some flips, you might reduce this maximum length

The task is to find the minimum possible length of such longest identical substring after performing at most `numOps` operations optimally.

The solution uses [binary search](/problems/binary_search_intro) to find this minimum length. For each candidate length `m`, it checks whether it's possible to achieve that maximum length using at most `numOps` flips:
- When `m = 1`, the string should alternate like "010101..." or "101010...", and we count how many flips are needed
- For `m > 1`, we count how many flips are needed to break up each consecutive group of identical characters into segments of at most length `m`

The [binary search](/problems/binary_search_intro) finds the smallest `m` where the required number of flips doesn't exceed `numOps`.

## Intuition

The key insight is that we're looking for the minimum possible value of the maximum consecutive identical characters. This "minimize the maximum" pattern naturally suggests [binary search](/problems/binary_search_intro) on the answer.

Think about it this way: if we can achieve a maximum length of `m`, then we can definitely achieve any length greater than `m` (we just need fewer or the same number of flips). Conversely, if we cannot achieve length `m` with our available operations, we cannot achieve any length smaller than `m` either. This monotonic property makes [binary search](/problems/binary_search_intro) applicable.

For any target maximum length `m`, we need to figure out the minimum number of flips required to ensure no consecutive identical substring exceeds length `m`:

1. **Special case when `m = 1`**: This means we want no two adjacent characters to be the same - the string must alternate perfectly like "010101..." or "101010...". We can calculate flips needed for both patterns and take the minimum.

2. **General case when `m > 1`**: For each group of consecutive identical characters of length `k`:
   - If `k <= m`, no flips needed for this group
   - If `k > m`, we need to break it up by flipping some characters within it
   - To break a group of length `k` into segments of at most `m`, we need `k // (m + 1)` flips strategically placed

The formula `k // (m + 1)` comes from the fact that each flip creates a breakpoint, and to divide a segment of length `k` into parts of at most length `m`, we need to place flips at intervals. For example, to break "11111" (length 5) into segments of at most 2, we need `5 // 3 = 1` flip in the middle.

By binary searching on the possible values of `m` from 1 to `n`, we find the smallest `m` where the required flips don't exceed `numOps`.

**Learn more about [Binary Search](/problems/binary_search_intro) patterns.**

## Solution Approach

The solution uses the **boundary-finding binary search template** to find the minimum possible maximum length of consecutive identical characters.

**Binary Search Template Structure:**
We search for the minimum `m` where `feasible(m)` returns `True`. The search space is `[1, n]`.

```
left, right = 1, n
first_true_index = -1

while left <= right:
    mid = (left + right) // 2
    if feasible(mid):
        first_true_index = mid
        right = mid - 1  # Look for smaller m
    else:
        left = mid + 1

return first_true_index
```

**The Feasible Function:**
`feasible(m)` determines if we can achieve a maximum consecutive length of `m` using at most `numOps` flips.

**Case 1: When `m = 1`**
- We need the string to alternate completely
- Count operations for pattern "010101..." using `sum(c == pattern[i & 1])`
- Take minimum of this count and `n - count` (for pattern "101010...")

**Case 2: When `m > 1`**
- For each group of `k` consecutive identical characters:
  - Calculate flips needed: `k // (m + 1)`
  - This formula optimally breaks a segment into parts of at most `m`

**Why This Template Works:**
The feasible function creates a monotonic pattern: `False, False, ..., True, True, True`. If we can achieve max length `m`, we can certainly achieve any length greater than `m`. We find the first `True` (minimum achievable m).

**Time Complexity:** `O(n log n)` where binary search runs `O(log n)` iterations, each check takes `O(n)`.

**Space Complexity:** `O(1)` - only using a few variables.

## Example Walkthrough

Let's walk through the solution with a concrete example:
- String `s = "111001"`
- `numOps = 2` (we can perform up to 2 flips)

**Initial Analysis:**
- Current consecutive groups: "111" (length 3), "00" (length 2), "1" (length 1)
- Maximum consecutive length = 3

**Binary Search Process:**

We'll binary search on the answer range [1, 6] (string length is 6).

**Iteration 1: Check if m = 3 is achievable**
- For each group:
  - "111" (length 3): needs `3 // (3+1) = 0` flips
  - "00" (length 2): needs `2 // (3+1) = 0` flips
  - "1" (length 1): needs `1 // (3+1) = 0` flips
- Total flips needed: 0 ≤ 2 ✓
- Since we can achieve m=3, try smaller values

**Iteration 2: Check if m = 2 is achievable**
- For each group:
  - "111" (length 3): needs `3 // (2+1) = 1` flip
  - "00" (length 2): needs `2 // (2+1) = 0` flips
  - "1" (length 1): needs `1 // (2+1) = 0` flips
- Total flips needed: 1 ≤ 2 ✓
- Since we can achieve m=2, try smaller values

**Iteration 3: Check if m = 1 is achievable**
- This requires alternating pattern
- Pattern "010101": compare with "111001"
  - Positions needing flips: indices 0, 1, 3 → 3 flips
- Pattern "101010": compare with "111001"
  - Positions needing flips: indices 2, 4, 5 → 3 flips
- Minimum flips needed: 3 > 2 ✗
- Cannot achieve m=1

**Result:** The minimum achievable maximum consecutive length is **2**.

To verify: With 1 flip at index 1, we get "101001" with maximum consecutive length of 2 ("00").

## Python Solution

```python
class Solution:
    def minLength(self, s: str, numOps: int) -> int:
        n = len(s)

        def feasible(m: int) -> bool:
            """Check if max consecutive length of m is achievable."""
            ops_needed = 0

            if m == 1:
                # Need alternating pattern
                pattern = "01"
                ops_needed = sum(c == pattern[i & 1] for i, c in enumerate(s))
                ops_needed = min(ops_needed, n - ops_needed)
            else:
                # Count ops to break long consecutive groups
                k = 0
                for i, c in enumerate(s):
                    k += 1
                    if i == n - 1 or c != s[i + 1]:
                        ops_needed += k // (m + 1)
                        k = 0

            return ops_needed <= numOps

        # Binary search template: find first m where feasible(m) is True
        left, right = 1, n
        first_true_index = -1

        while left <= right:
            mid = (left + right) // 2
            if feasible(mid):
                first_true_index = mid
                right = mid - 1  # Look for smaller m
            else:
                left = mid + 1

        return first_true_index
```

## Time and Space Complexity

**Time Complexity:** `O(n * log n)`

The algorithm uses binary search on the range `[1, n]` where `n` is the length of string `s`. The binary search performs `O(log n)` iterations.

For each iteration of binary search, the `check` function is called:
- When `m == 1`: The function iterates through the string once to count mismatches with alternating patterns, taking `O(n)` time.
- When `m > 1`: The function iterates through the string once to count consecutive segments and calculate required operations, taking `O(n)` time.

Therefore, the overall time complexity is `O(n * log n)`.

**Space Complexity:** `O(1)`

The algorithm uses only a constant amount of extra space:
- The `check` function uses variables `cnt`, `k`, and `t` (when `m == 1`)
- The string `t = "01"` when `m == 1` is constant size
- The binary search itself only requires a few variables for bounds and mid-point calculations

The space used does not scale with the input size, resulting in `O(1)` space complexity.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Using Wrong Binary Search Template Variant**

A critical mistake is using the `while left < right` with `right = mid` pattern instead of the correct template.

**Incorrect approach:**
```python
left, right = 1, n
while left < right:
    mid = (left + right) // 2
    if feasible(mid):
        right = mid
    else:
        left = mid + 1
return left
```

**Correct approach (template-compliant):**
```python
left, right = 1, n
first_true_index = -1

while left <= right:
    mid = (left + right) // 2
    if feasible(mid):
        first_true_index = mid
        right = mid - 1
    else:
        left = mid + 1

return first_true_index
```

The template uses `while left <= right` with `right = mid - 1` and tracks the answer in `first_true_index`.

### 2. **Incorrect Formula for Breaking Consecutive Groups**

Using the wrong formula to calculate flips needed:
- `k // m` - doesn't account for spacing correctly
- `(k - 1) // m` - underestimates

**Correct formula:** `k // (m + 1)` - places flips every `(m + 1)` positions optimally.

### 3. **Misunderstanding the Alternating Pattern Check**

When checking for `m = 1`, forgetting to consider both patterns ("010101..." and "101010..."):

```python
# Wrong: Only checking one pattern
cnt = sum(c == pattern[i & 1] for i, c in enumerate(s))

# Correct: Take minimum of both patterns
cnt = min(cnt, n - cnt)
```

### 4. **Not Resetting the Consecutive Counter**

Forgetting to reset `k = 0` after processing each group leads to incorrect calculations for subsequent groups.

