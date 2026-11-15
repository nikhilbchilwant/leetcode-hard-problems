# 960. Delete Columns to Make Sorted III

> Source: https://algo.monster/liteproblems/960

## Problem Statement

You have an array of `n` strings `strs`, where all strings have the same length.

You can select any set of column indices and delete those columns from all strings. When you delete a column at index `i`, you remove the character at position `i` from every string.

For example, if `strs = ["abcdef", "uvwxyz"]` and you delete columns at indices `{0, 2, 3}`:
- "abcdef" becomes "bef" (removing 'a' at index 0, 'c' at index 2, 'd' at index 3)
- "uvwxyz" becomes "vyz" (removing 'u' at index 0, 'w' at index 2, 'x' at index 3)

Your goal is to find the minimum number of columns to delete so that each remaining string is in lexicographic (alphabetical) order. This means for each string, every character should be less than or equal to the next character in that string.

For instance, after deletions:
- `strs[0]` should satisfy: `strs[0][0] <= strs[0][1] <= ... <= strs[0][k]`
- `strs[1]` should satisfy: `strs[1][0] <= strs[1][1] <= ... <= strs[1][k]`
- And so on for all strings

Return the minimum number of columns that need to be deleted to achieve this.

## Intuition

Instead of thinking about which columns to delete, we can flip the problem and think about which columns to keep. If we keep certain columns, they must form a valid subsequence where each string remains sorted.

The key insight is that this becomes a longest increasing subsequence (LIS) problem, but with a twist. We're looking for the longest subsequence of columns such that:
1. Within each string, the characters in the kept columns are in non-decreasing order
2. The kept columns maintain their relative positions

For example, if we keep columns 1, 3, and 5, then for every string `s`, we need `s[1] <= s[3] <= s[5]`.

To find this longest valid subsequence, we use [dynamic programming](/problems/dynamic_programming_intro). For each column `i`, we ask: "What's the longest valid subsequence that ends at column `i`?" 

To compute this, we look at all previous columns `j < i`. If column `j` can precede column `i` (meaning for all strings, the character at column `j` is less than or equal to the character at column `i`), then we can extend the subsequence ending at `j` by adding column `i`.

The condition `all(s[j] <= s[i] for s in strs)` checks whether keeping both columns `j` and `i` maintains the sorted order for all strings.

Once we find the maximum length of valid subsequences across all ending positions, the minimum deletions needed is simply `total columns - maximum valid subsequence length`. This transforms a deletion problem into a maximization problem, making it easier to solve with standard [dynamic programming](/problems/dynamic_programming_intro) techniques.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution implements a [dynamic programming](/problems/dynamic_programming_intro) approach to find the longest valid subsequence of columns.

**Step 1: Initialize the DP array**

We create an array `f` where `f[i]` represents the length of the longest valid subsequence ending at column `i`. Initially, each column by itself forms a valid subsequence of length 1, so `f = [1] * n`.

**Step 2: Build the DP table**

For each column `i` from 0 to `n-1`:
- We examine all previous columns `j` where `j < i`
- For each pair `(j, i)`, we check if column `j` can precede column `i` in a valid subsequence

**Step 3: Check validity condition**

The condition `all(s[j] <= s[i] for s in strs)` verifies that for every string in the array, the character at column `j` is less than or equal to the character at column `i`. This ensures that if we keep both columns `j` and `i`, all strings remain sorted.

**Step 4: Update the DP value**

If the validity condition is satisfied, we can extend the subsequence ending at column `j` by including column `i`. We update:
```
f[i] = max(f[i], f[j] + 1)
```

This means the longest subsequence ending at `i` is either:
- Its current value `f[i]`
- Or the subsequence ending at `j` plus one more column: `f[j] + 1`

**Step 5: Calculate the result**

After computing all `f[i]` values, `max(f)` gives us the length of the longest valid subsequence of columns. Since we need to delete all other columns, the minimum number of deletions is:
```
n - max(f)
```

**Time Complexity:** `O(n² × m)` where `n` is the length of each string and `m` is the number of strings. We have two nested loops over columns, and for each pair, we check all `m` strings.

**Space Complexity:** `O(n)` for the DP array `f`.

## Example Walkthrough

Let's walk through a small example with `strs = ["cba", "daf", "ghi"]`.

Each string has length 3, so we have columns 0, 1, and 2. Our goal is to find the minimum columns to delete so each string becomes sorted.

**Initial Setup:**
- Column 0: ['c', 'd', 'g']
- Column 1: ['b', 'a', 'f']  
- Column 2: ['a', 'f', 'i']
- Initialize `f = [1, 1, 1]` (each column alone forms a valid subsequence of length 1)

**Building the DP Table:**

For `i = 1`:
- Check if column 0 can precede column 1
- Compare: 'c' <= 'b'? No (for string 0)
- Since the condition fails, we can't extend. `f[1]` remains 1

For `i = 2`:
- Check if column 0 can precede column 2
  - 'c' <= 'a'? No (for string 0)
  - Can't extend from column 0

- Check if column 1 can precede column 2
  - 'b' <= 'a'? No (for string 0)
  - Can't extend from column 1

- `f[2]` remains 1

**Final DP array:** `f = [1, 1, 1]`

The maximum value in `f` is 1, meaning the longest valid subsequence has length 1 (we can keep at most one column).

**Result:** Minimum deletions = 3 - 1 = 2

We need to delete 2 columns. For instance:
- Keep only column 2: ["a", "f", "i"] - all strings become single characters (trivially sorted)
- Or keep only column 0: ["c", "d", "g"] - all strings become single characters

Let's verify: If we delete columns 0 and 1, we get ["a", "f", "i"]. Each string has only one character, so they're all sorted. ✓

## Python Solution

```python
class Solution:
    def minDeletionSize(self, strs: List[str]) -> int:
        # Get the length of each string (all strings have same length)
        string_length = len(strs[0])
      
        # dp[i] represents the maximum length of increasing subsequence ending at column i
        # Initialize all positions with 1 (each column itself forms a subsequence of length 1)
        dp = [1] * string_length
      
        # For each column position
        for current_col in range(string_length):
            # Check all previous columns
            for prev_col in range(current_col):
                # Check if we can extend the subsequence from prev_col to current_col
                # This is valid if for ALL strings, character at prev_col <= character at current_col
                if all(string[prev_col] <= string[current_col] for string in strs):
                    # Update the maximum subsequence length ending at current_col
                    dp[current_col] = max(dp[current_col], dp[prev_col] + 1)
      
        # The minimum columns to delete = total columns - maximum columns we can keep
        return string_length - max(dp)
```

## Time and Space Complexity

**Time Complexity:** `O(n^2 × m)`

The algorithm uses nested loops to iterate through all pairs of column indices `(i, j)` where `j < i`. The outer loop runs `n` times (where `n` is the length of each string), and the inner loop runs up to `i` times for each iteration of the outer loop, resulting in `O(n^2)` iterations total. For each pair `(i, j)`, the `all()` function checks a condition across all `m` strings in the array, which takes `O(m)` time. Therefore, the overall time complexity is `O(n^2 × m)`.

**Space Complexity:** `O(n)`

The algorithm uses an array `f` of size `n` to store the dynamic programming values, where `n` is the length of each string. No other data structures that scale with the input size are used. The space required for loop variables and temporary storage is constant. Therefore, the space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Misunderstanding the Problem Goal

**The Mistake:** A common misunderstanding is thinking we need to make the strings sorted relative to each other (like `strs[0] <= strs[1] <= strs[2]`), rather than making each individual string internally sorted.

**Why it happens:** The term "lexicographic order" can be ambiguous, and it's easy to confuse sorting between strings versus sorting within each string.

**Correct Understanding:** We need each string to be individually sorted. For example, if we have `["bac", "cab", "xyz"]`, we want to delete columns so that what remains of each string is sorted:
- "bac" might become "bc" (delete 'a')
- "cab" might become "ab" (delete 'c')  
- "xyz" stays "xyz" (no deletions needed)

### Pitfall 2: Incorrect Validity Check

**The Mistake:** Writing the condition as `strs[0][j] <= strs[0][i]` instead of checking ALL strings:

```python
# WRONG - only checks first string
if strs[0][prev_col] <= strs[0][current_col]:
    dp[current_col] = max(dp[current_col], dp[prev_col] + 1)
```

**Why it happens:** Testing with simple examples where checking just one string seems to work, or forgetting that the condition must hold for every string simultaneously.

**Correct Implementation:**
```python
# CORRECT - checks all strings
if all(string[prev_col] <= string[current_col] for string in strs):
    dp[current_col] = max(dp[current_col], dp[prev_col] + 1)
```

### Pitfall 3: Off-by-One Error in Final Calculation

**The Mistake:** Returning `max(dp)` directly instead of `string_length - max(dp)`:

```python
# WRONG - returns columns to keep, not columns to delete
return max(dp)
```

**Why it happens:** The DP array tracks the columns we're keeping (the longest valid subsequence), but the problem asks for the number of columns to delete.

**Correct Calculation:**
```python
# CORRECT - converts "columns kept" to "columns deleted"
return string_length - max(dp)
```

### Pitfall 4: Not Handling Edge Cases

**The Mistake:** Not considering edge cases like:
- Single character strings (`["a", "b"]`)
- Already sorted strings (`["abc", "xyz"]`)
- Strings requiring all columns deleted except one

**Solution:** The DP approach naturally handles these cases since:
- Each column starts with `dp[i] = 1` (can always keep at least one column)
- Already sorted strings will have `dp = [1, 2, 3, ..., n]`, giving `n - n = 0` deletions
- The algorithm correctly identifies when columns can't be paired together

