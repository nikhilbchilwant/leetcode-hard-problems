# 2565. Subsequence With the Minimum Score

> Source: https://algo.monster/liteproblems/2565

## Problem Statement

You have two strings `s` and `t`. Your goal is to make `t` a subsequence of `s` by removing some characters from `t`.

When you remove characters from `t`, you incur a score based on the positions of the removed characters:
- If you don't remove any characters, the score is `0`
- If you do remove characters, the score is calculated as `right - left + 1`, where:
  - `left` is the minimum index among all removed characters
  - `right` is the maximum index among all removed characters

Your task is to find the minimum possible score needed to make `t` a subsequence of `s`.

A subsequence means that the remaining characters in `t` (after removal) appear in `s` in the same relative order, though not necessarily consecutively. For example, `"ace"` is a subsequence of `"abcde"` because you can find `a`, `c`, and `e` in that order within `"abcde"`.

## Intuition

The key observation is that when we remove characters from `t`, we're actually removing a contiguous substring. This means we keep a prefix of `t` (before the removed part) and a suffix of `t` (after the removed part), and both these parts need to be subsequences of `s`.

Think about it this way: if we remove characters from index `left` to `right` in `t`, we're left with:
- `t[0...left-1]` (the prefix we keep)
- `t[right+1...n-1]` (the suffix we keep)

For `t` to become a subsequence of `s` after this removal, we need:
1. The kept prefix must match some prefix of `s` as a subsequence
2. The kept suffix must match some suffix of `s` as a subsequence
3. These matches in `s` must not overlap

This leads us to precompute two important pieces of information:
- For each position `i` in `t`, what's the earliest position in `s` where we can match `t[0...i]` as a subsequence? Let's call this `f[i]`.
- For each position `j` in `t`, what's the latest position in `s` where we can start matching `t[j...n-1]` as a subsequence? Let's call this `g[j]`.

With these precomputed values, for any removal range `[left, right]`, we can quickly check if it's valid: we keep prefix `t[0...left-1]` which needs `s[0...f[left-1]]`, and suffix `t[right+1...n-1]` which needs `s[g[right+1]...m-1]`. The removal is valid if `f[left-1] < g[right+1]` (no overlap in `s`).

Since we want to minimize the score `right - left + 1`, which is essentially the length of the removed substring, and longer removals are always valid if shorter ones are valid (monotonicity property), we can use [binary search](/problems/binary_search_intro) on the length of the removal to find the minimum valid length.

**Learn more about [Two Pointers](/problems/two_pointers_intro) and [Binary Search](/problems/binary_search_intro) patterns.**

## Solution Approach

The solution consists of three main phases: preprocessing, [binary search](/problems/binary_search_intro), and validation.

**Phase 1: Preprocessing Arrays `f` and `g`**

First, we build array `f` where `f[i]` stores the minimum index in string `s` needed to match the prefix `t[0...i]` as a subsequence:

```python
f = [inf] * n  # Initialize with infinity
i, j = 0, 0
while i < m and j < n:
    if s[i] == t[j]:
        f[j] = i  # Record position in s where t[j] is matched
        j += 1
    i += 1
```

This greedy approach matches each character of `t` with the earliest possible character in `s`.

Similarly, we build array `g` where `g[i]` stores the maximum starting index in string `s` from where we can match the suffix `t[i...n-1]` as a subsequence:

```python
g = [-1] * n  # Initialize with -1
i, j = m - 1, n - 1
while i >= 0 and j >= 0:
    if s[i] == t[j]:
        g[j] = i  # Record position in s where t[j] is matched
        j -= 1
    i -= 1
```

This processes from right to left, matching each character of `t` with the latest possible character in `s`.

**Phase 2: [Binary Search](/problems/binary_search_intro) on Removal Length**

Since removing more characters always makes it easier to form a subsequence (monotonicity property), we can [binary search](/problems/binary_search_intro) on the length of the substring to remove:

```python
return bisect_left(range(n + 1), True, key=check)
```

This searches for the minimum length `x` (from 0 to n) where removing a substring of length `x` makes `t` a subsequence of `s`.

**Phase 3: Validation Function**

The `check(x)` function determines if we can remove a substring of length `x` from `t` and still form a subsequence of `s`:

```python
def check(x):
    for k in range(n):
        i, j = k - 1, k + x
        l = f[i] if i >= 0 else -1
        r = g[j] if j < n else m + 1
        if l < r:
            return True
    return False
```

For each possible starting position `k` of removal:
- We keep prefix `t[0...k-1]` which requires `s[0...f[k-1]]`
- We keep suffix `t[k+x...n-1]` which requires `s[g[k+x]...m-1]`
- The removal is valid if `f[k-1] < g[k+x]` (no overlap in `s`)

The boundary cases are handled by:
- If `i < 0` (no prefix), set `l = -1`
- If `j >= n` (no suffix), set `r = m + 1`

The [binary search](/problems/binary_search_intro) finds the minimum valid removal length, which is our answer.

## Example Walkthrough

Let's walk through a concrete example with `s = "abacaba"` and `t = "bzaa"`.

**Step 1: Build the prefix array `f`**

We need to find for each position in `t`, the earliest position in `s` where we can match `t[0...i]` as a subsequence.

- `t[0] = 'b'`: Found at `s[1]`, so `f[0] = 1`
- `t[1] = 'z'`: No 'z' in `s` after matching 'b', so `f[1] = inf`
- `t[2] = 'a'`: Would need 'bza' but 'z' doesn't exist, so `f[2] = inf`
- `t[3] = 'a'`: Would need 'bzaa' but 'z' doesn't exist, so `f[3] = inf`

Result: `f = [1, inf, inf, inf]`

**Step 2: Build the suffix array `g`**

We need to find for each position in `t`, the latest starting position in `s` where we can match `t[i...n-1]` as a subsequence.

Working backwards:
- `t[3] = 'a'`: Last 'a' in `s` is at position 6, so `g[3] = 6`
- `t[2] = 'a'`: Need "aa", can start at position 4 (matches positions 4,6), so `g[2] = 4`
- `t[1] = 'z'`: Need "zaa", but 'z' doesn't exist in `s`, so `g[1] = -1`
- `t[0] = 'b'`: Need "bzaa", but can't match 'z', so `g[0] = -1`

Result: `g = [-1, -1, 4, 6]`

**Step 3: Binary search on removal length**

We try different removal lengths to find the minimum:

**Check length 0 (no removal):**
- Keep entire string "bzaa"
- Need `s` to contain "bzaa" as subsequence
- From `g[0] = -1`, we know this is impossible
- Result: FALSE

**Check length 1 (remove 1 character):**

Try removing at each position:
- Remove position 0: Keep suffix "zaa", need `g[1] = -1` (impossible)
- Remove position 1: Keep prefix "b" and suffix "aa"
  - Prefix "b" needs `s[0...f[0]] = s[0...1]`
  - Suffix "aa" needs `s[g[2]...6] = s[4...6]`
  - Check: `f[0] < g[2]` → `1 < 4` ✓ Valid!
- Remove position 2: Keep prefix "bz", but `f[1] = inf` (impossible)
- Remove position 3: Keep prefix "bza", but `f[2] = inf` (impossible)

Result: TRUE (we found a valid removal of length 1)

**Answer:** The minimum score is 1 (removing one character at position 1).

We can verify: removing 'z' from "bzaa" gives us "baa", which is indeed a subsequence of "abacaba" (taking the 'b' at position 1 and the 'a's at positions 2 and 4).

## Python Solution

```python
from math import inf

class Solution:
    def minimumScore(self, s: str, t: str) -> int:
        """
        Find the minimum length of substring to remove from t
        so that the remaining parts form a subsequence of s.
        """

        def can_remove_length(removal_length):
            """
            Check if we can remove a substring of given length from t
            and still have the remaining parts be a subsequence of s.
            """
            for start_pos in range(target_len):
                left_end = start_pos - 1
                right_start = start_pos + removal_length

                last_s_pos_for_left = left_match_positions[left_end] if left_end >= 0 else -1
                first_s_pos_for_right = right_match_positions[right_start] if right_start < target_len else source_len + 1

                if last_s_pos_for_left < first_s_pos_for_right:
                    return True
            return False

        source_len, target_len = len(s), len(t)

        # left_match_positions[i]: rightmost index in s where t[0:i+1] can be matched
        left_match_positions = [inf] * target_len

        # right_match_positions[i]: leftmost index in s where t[i:] can be matched
        right_match_positions = [-1] * target_len

        # Build left_match_positions: match t from left to right in s
        s_idx, t_idx = 0, 0
        while s_idx < source_len and t_idx < target_len:
            if s[s_idx] == t[t_idx]:
                left_match_positions[t_idx] = s_idx
                t_idx += 1
            s_idx += 1

        # Build right_match_positions: match t from right to left in s
        s_idx, t_idx = source_len - 1, target_len - 1
        while s_idx >= 0 and t_idx >= 0:
            if s[s_idx] == t[t_idx]:
                right_match_positions[t_idx] = s_idx
                t_idx -= 1
            s_idx -= 1

        # Binary search for minimum removal length
        # Feasible condition: can_remove_length returns True
        left, right = 0, target_len
        first_true_index = -1

        while left <= right:
            mid = (left + right) // 2
            if can_remove_length(mid):
                first_true_index = mid
                right = mid - 1
            else:
                left = mid + 1

        return first_true_index
```

## Time and Space Complexity

**Time Complexity:** `O(n × log n)`

The algorithm consists of three main parts:

1. **First forward pass** to build array `f`: This iterates through string `s` (length `m`) and string `t` (length `n`) once, taking `O(m + n)` time.

2. **Second backward pass** to build array `g`: Similarly iterates through both strings once, taking `O(m + n)` time.

3. **Binary search with check function**: The `bisect_left` performs a binary search on the range `[0, n]`, which has `n + 1` elements. This results in `O(log n)` iterations. For each iteration, the `check` function is called, which loops through `n` elements (from `k = 0` to `k = n-1`), performing `O(1)` operations in each iteration. Therefore, each `check` call takes `O(n)` time.

The total time complexity is: `O(m + n) + O(m + n) + O(n × log n) = O(m + n + n × log n)`

Since we're typically looking for the subsequence `t` in string `s` where `n ≤ m`, and the dominant term is `n × log n`, the overall time complexity simplifies to `O(n × log n)`.

**Space Complexity:** `O(n)`

The algorithm uses:
- Array `f` of size `n`: `O(n)` space
- Array `g` of size `n`: `O(n)` space
- A few integer variables (`i`, `j`, `k`, `l`, `r`, etc.): `O(1)` space

The total space complexity is: `O(n) + O(n) + O(1) = O(n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. Using Wrong Binary Search Template

**Pitfall:** Using `while left < right` with `right = mid` instead of the standard template. This can lead to confusion and potential bugs.

**Solution:** Use the standard binary search template with `while left <= right`, `first_true_index` tracking, and `right = mid - 1`:
```python
left, right = 0, target_len
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

### 2. Incorrect Boundary Handling in Preprocessing Arrays

**Pitfall:** Not properly handling cases where the entire prefix or suffix of `t` cannot be matched in `s`. If `t` cannot be fully matched as a subsequence of `s`, the preprocessing arrays may contain uninitialized or incorrect values.

**Example:** If `s = "abc"` and `t = "xyz"`, none of the characters match, so the arrays remain at their initial values (`inf` and `-1`). This could lead to incorrect comparisons in the validation function.

**Solution:** The validation function correctly handles these extreme values by checking boundary conditions before accessing the arrays.

### 3. Off-by-One Errors in Index Calculations

**Pitfall:** When calculating the removal range, it's easy to make off-by-one errors. For instance, if we remove characters from index `k` to `k + x - 1` (total `x` characters), the remaining suffix starts at index `k + x`, not `k + x - 1`.

**Example:** If `t = "abcd"` and we remove 2 characters starting at index 1, we remove indices 1 and 2 (`"bc"`), leaving `"a"` (index 0) and `"d"` (index 3).

**Solution:** Be explicit about index ranges and use clear variable names:

```python
def can_remove_length(removal_length):
    for removal_start in range(target_len):
        removal_end = removal_start + removal_length - 1  # inclusive end
        prefix_end = removal_start - 1  # last index of prefix (-1 if no prefix)
        suffix_start = removal_end + 1  # first index of suffix (>= target_len if no suffix)

        # Now the logic is clearer
        left_bound = left_match_positions[prefix_end] if prefix_end >= 0 else -1
        right_bound = right_match_positions[suffix_start] if suffix_start < target_len else source_len

        if left_bound < right_bound:
            return True
    return False
```

### 4. Misunderstanding the Score Calculation

**Pitfall:** The score is `right - left + 1` where `left` and `right` are indices in `t`, not in `s`. Some might mistakenly use indices from `s` or forget the `+1` in the formula.

**Solution:** Remember that we're always removing a contiguous substring from `t`, and the score is simply the length of that substring. The binary search directly searches for this length, making the score calculation implicit and avoiding confusion.

