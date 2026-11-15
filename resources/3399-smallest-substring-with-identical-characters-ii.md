# 3399. Smallest Substring With Identical Characters II

> Source: https://algo.monster/liteproblems/3399

## Problem Statement

You have a binary string `s` consisting of only '0's and '1's with length `n`, and you're allowed to perform up to `numOps` operations on it.

Each operation allows you to:
- Choose any index `i` (where `0 <= i < n`)
- Flip the character at that position (change '0' to '1' or '1' to '0')

Your goal is to minimize the length of the longest substring where all characters are identical (all '0's or all '1's).

For example:
- In string "111000", the longest substring of identical characters has length 3 (either "111" or "000")
- After flipping the character at index 2 to get "110000", the longest substring now has length 4 ("0000")
- After flipping the character at index 3 to get "110100", the longest substring now has length 2 (either "11" or "00")

The function should return the minimum possible length of such a longest identical substring after performing at most `numOps` flips optimally.

## Intuition

The key insight is that we want to find the minimum possible value for the maximum length of consecutive identical characters. This is a classic optimization problem where we're minimizing a maximum value - a perfect candidate for [binary search](/problems/binary_search_intro).

Why [binary search](/problems/binary_search_intro)? The answer lies in the monotonic property: if we can achieve a maximum length of `m` with some number of operations, then we can definitely achieve any length greater than `m` with the same or fewer operations. Conversely, if we cannot achieve length `m`, we cannot achieve any length smaller than `m` either.

For any target maximum length `m`, we need to figure out the minimum number of flips required to ensure no consecutive identical substring exceeds this length. This breaks down into two distinct strategies:

1. **When `m = 1`**: We need a completely alternating pattern like "010101..." or "101010...". We can count how many positions differ from each pattern and choose the one requiring fewer flips. This is optimal because any substring of length 2 or more would contain different characters.

2. **When `m > 1`**: We need to break up long runs of identical characters. For a run of `k` consecutive identical characters, we need to place flips strategically to split it into segments of at most `m` characters. The optimal way is to place flips every `m+1` positions within the run, requiring `k // (m+1)` flips.

The [binary search](/problems/binary_search_intro) tries different values of `m` from 1 to `n`, checking for each whether it's achievable with at most `numOps` operations. The smallest achievable `m` is our answer.

This approach is efficient because instead of trying all possible combinations of flips (which would be exponential), we're using the problem's structure to directly compute the minimum flips needed for any given target length.

**Learn more about [Binary Search](/problems/binary_search_intro) patterns.**

## Solution Approach

The solution uses the **boundary-finding binary search template** to find the minimum achievable maximum length of consecutive identical characters.

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

Let's walk through the solution with `s = "111000"` and `numOps = 2`.

**Initial Analysis:**
- String: "111000" (length n = 6)
- Current longest consecutive identical substring: 3 (both "111" and "000")
- We have 2 operations available

**Binary Search Process:**

We'll binary search on the range [1, 6] to find the minimum achievable maximum length.

**Try m = 3 (midpoint of [1, 6]):**
- Check if we can limit all consecutive substrings to length 3
- Current groups: "111" (length 3), "000" (length 3)
- For group "111": 3 // (3+1) = 3 // 4 = 0 flips needed
- For group "000": 3 // (3+1) = 3 // 4 = 0 flips needed
- Total flips needed: 0 ≤ 2 ✓
- Since this works, try smaller values

**Try m = 1:**
- Need alternating pattern: either "010101" or "101010"
- Compare with "010101": positions 0,1,2 need flipping → 3 flips
- Compare with "101010": positions 3,4,5 need flipping → 3 flips
- Minimum flips needed: 3 > 2 ✗
- Cannot achieve m = 1

**Try m = 2:**
- Need to break groups longer than 2
- Group "111" (length 3): 3 // (2+1) = 3 // 3 = 1 flip needed
  - One flip at position 1 or 2 would give "101000" or "110000"
- Group "000" (length 3): 3 // (2+1) = 3 // 3 = 1 flip needed
  - One flip at position 3 or 4 would give "110100" or "111010"
- Total flips needed: 2 ≤ 2 ✓
- This works!

**Binary Search Conclusion:**
- m = 1 doesn't work (needs 3 flips)
- m = 2 works (needs 2 flips)
- Therefore, the minimum achievable maximum length is 2

**Verification:**
After flipping positions 1 and 4: "111000" → "101010"
- All consecutive identical substrings have length ≤ 1
Actually, we can also flip positions 2 and 3: "111000" → "110100"
- Consecutive identical substrings: "11", "01", "00" (max length = 2)

The answer is 2.

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

For each iteration of the binary search, the `check` function is called:
- When `m == 1`: The function iterates through the string once to count mismatches with alternating patterns "01" or "10", taking `O(n)` time.
- When `m > 1`: The function iterates through the string once to count consecutive segments and calculate required operations, taking `O(n)` time.

Since each `check` call takes `O(n)` time and binary search makes `O(log n)` calls, the overall time complexity is `O(n * log n)`.

**Space Complexity:** `O(1)`

The algorithm uses only a constant amount of extra space:
- The `check` function uses variables `cnt`, `t` (when `m == 1`), and `k` which require `O(1)` space.
- The binary search itself only uses a few variables for bounds and mid-point calculations.
- No additional data structures proportional to input size are created.

Therefore, the space complexity is `O(1)`.

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

