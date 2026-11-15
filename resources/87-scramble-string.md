# 87. Scramble String

> Source: https://algo.monster/liteproblems/87

## Problem Statement

You are given two strings `s1` and `s2` of the same length. You need to determine if `s2` is a scrambled version of `s1`.

A string can be scrambled using the following recursive algorithm:

1. If the string has length 1, stop (base case).

2. If the string has length > 1:
   - Split the string into two non-empty parts at any position. For example, if `s = "great"`, you could split it into `x = "gr"` and `y = "eat"`, making `s = x + y`.
   - Choose to either keep the two parts in the same order (`s = x + y`) or swap them (`s = y + x`).
   - Recursively apply this scrambling process to each part `x` and `y`.

Through this process, a string can be transformed into many different scrambled versions. For example, starting with `"great"`:
- Split into `"gr"` and `"eat"`
- Swap to get `"eat" + "gr" = "eatgr"`
- Recursively scramble `"gr"` to get `"rg"` (by swapping)
- Result: `"eatrg"` is a scrambled version of `"great"`

The task is to return `true` if `s2` can be obtained by scrambling `s1` using this algorithm, otherwise return `false`.

The solution uses [dynamic programming](/problems/dynamic_programming_intro) with memoization. The function `dfs(i, j, k)` checks if a substring of length `k` starting at index `i` in `s1` can be scrambled to match a substring of length `k` starting at index `j` in `s2`. For each possible split position `h`, it checks two scenarios: 
- No swap: the first `h` characters match and the remaining `k-h` characters match
- Swap: the first `h` characters of `s1` match the last `h` characters of `s2`, and vice versa

The base case is when `k=1`, where we simply check if the single characters are equal.

## Intuition

The key insight is that if `s2` is a scrambled version of `s1`, then there must exist a way to recursively partition both strings such that the corresponding parts match (either directly or after swapping).

Think about the scrambling process in reverse. If `s2` came from `s1`, then at some point during the scrambling, `s1` was split into two parts. These two parts were either kept in order or swapped, and then each part was independently scrambled. This means we can check if `s2` matches `s1` by trying all possible split points and checking if the resulting subproblems are valid.

For any substring of `s1` and corresponding substring of `s2` with the same length `k`, we need to find if there's a valid split point `h` (where `1 ≤ h < k`) such that:

1. **No swap case**: The first `h` characters of both substrings can be scrambled to match each other, AND the remaining `k-h` characters can also be scrambled to match. This corresponds to checking `dfs(i, j, h) && dfs(i+h, j+h, k-h)`.

2. **Swap case**: The first `h` characters of `s1` can be scrambled to match the last `h` characters of `s2`, AND the last `k-h` characters of `s1` can be scrambled to match the first `k-h` characters of `s2`. This corresponds to checking `dfs(i+h, j, k-h) && dfs(i, j+k-h, h)`.

The reason we need to check all possible split points is that we don't know where the original split occurred during scrambling. By trying all possibilities and using memoization to avoid recalculating the same subproblems, we can efficiently determine if the transformation is possible.

The base case is straightforward: when we're comparing single characters (`k=1`), they either match or they don't - no scrambling is possible at this level.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution implements a top-down [dynamic programming](/problems/dynamic_programming_intro) approach with memoization using Python's `@cache` decorator.

**Core Function Definition:**
The function `dfs(i, j, k)` determines whether the substring `s1[i:i+k]` can be scrambled to match `s2[j:j+k]`. The parameters represent:
- `i`: starting index in `s1`
- `j`: starting index in `s2`
- `k`: length of the substring to compare

**Base Case:**
When `k = 1`, we're comparing single characters. Simply check if `s1[i] == s2[j]`. If they match, return `True`; otherwise, return `False`.

**Recursive Case:**
For `k > 1`, we try all possible split points `h` where `1 ≤ h < k`. For each split point, we check two scenarios:

1. **No Swap Scenario:** 
   - Check if `dfs(i, j, h)` returns `True` (first part matches)
   - AND `dfs(i + h, j + h, k - h)` returns `True` (second part matches)
   - This represents keeping the original order after splitting

2. **Swap Scenario:**
   - Check if `dfs(i + h, j, k - h)` returns `True` (second part of `s1` matches first part of `s2`)
   - AND `dfs(i, j + k - h, h)` returns `True` (first part of `s1` matches second part of `s2`)
   - This represents swapping the two parts after splitting

If either scenario returns `True` for any split point `h`, we return `True`. If all split points fail both scenarios, we return `False`.

**Memoization:**
The `@cache` decorator automatically stores the results of `dfs(i, j, k)` calls. This prevents redundant calculations when the same subproblem appears multiple times, reducing the time complexity from exponential to polynomial.

**Main Function:**
The solution starts by calling `dfs(0, 0, len(s1))`, which checks if the entire string `s1` can be scrambled to match the entire string `s2`.

**Time Complexity:** `O(n^4)` where `n` is the length of the strings. There are `O(n^3)` possible states `(i, j, k)`, and for each state, we iterate through up to `n` split points.

**Space Complexity:** `O(n^3)` for the memoization cache storing all possible states.

## Example Walkthrough

Let's trace through how the algorithm determines if `s2 = "rgeat"` is a scrambled version of `s1 = "great"`.

We start with `dfs(0, 0, 5)`, checking if `"great"` can be scrambled to match `"rgeat"`.

**Step 1: Initial call `dfs(0, 0, 5)`**
- We're comparing `s1[0:5] = "great"` with `s2[0:5] = "rgeat"`
- Since `k = 5 > 1`, we try different split points `h = 1, 2, 3, 4`

**Step 2: Try `h = 1` (split at position 1)**
- No swap: Check `dfs(0, 0, 1)` AND `dfs(1, 1, 4)`
  - `dfs(0, 0, 1)`: Compare `"g"` with `"r"` → returns `False`
  - Since first part is `False`, no need to check second part
- Swap: Check `dfs(1, 0, 4)` AND `dfs(0, 4, 1)`
  - `dfs(1, 0, 4)`: Check if `"reat"` can scramble to `"rgea"`
  - `dfs(0, 4, 1)`: Compare `"g"` with `"t"` → returns `False`
  - Since second part is `False`, this fails

**Step 3: Try `h = 2` (split at position 2)**
- No swap: Check `dfs(0, 0, 2)` AND `dfs(2, 2, 3)`
  - `dfs(0, 0, 2)`: Check if `"gr"` can scramble to `"rg"` 
    - Try `h = 1`: Swap case works! `"g"` matches `"g"` and `"r"` matches `"r"`
    - Returns `True`
  - `dfs(2, 2, 3)`: Check if `"eat"` can scramble to `"eat"`
    - Direct match, returns `True`
  - Both parts are `True`, so this split works!

**Step 4: Return `True`**

The algorithm found that splitting `"great"` into `"gr"` and `"eat"`, then scrambling `"gr"` to `"rg"` (by swapping at split point 1), produces `"rgeat"`. Therefore, `"rgeat"` is indeed a scrambled version of `"great"`.

Key observations:
- The algorithm systematically tries all possible ways the original string could have been split
- Memoization ensures that when we encounter `dfs(0, 0, 2)` again in other branches, we don't recalculate it
- The recursion naturally handles nested scrambling (like scrambling within `"gr"` to get `"rg"`)

## Python Solution

```python
class Solution:
    def isScramble(self, s1: str, s2: str) -> bool:
        """
        Determines if s2 is a scrambled string of s1.
        A scrambled string is formed by recursively dividing the string into two non-empty substrings
        and potentially swapping them.
      
        Args:
            s1: The original string
            s2: The string to check if it's a scrambled version of s1
          
        Returns:
            True if s2 is a scrambled string of s1, False otherwise
        """
        from functools import cache
      
        @cache
        def is_scramble_substring(start_s1: int, start_s2: int, length: int) -> bool:
            """
            Recursively checks if a substring of s2 is a scrambled version of a substring of s1.
          
            Args:
                start_s1: Starting index in s1
                start_s2: Starting index in s2
                length: Length of the substring to check
              
            Returns:
                True if the substrings match as scrambled strings, False otherwise
            """
            # Base case: single character comparison
            if length == 1:
                return s1[start_s1] == s2[start_s2]
          
            # Try all possible split points
            for split_point in range(1, length):
                left_length = split_point
                right_length = length - split_point
              
                # Case 1: No swap occurred at this level
                # Check if left parts match and right parts match
                if (is_scramble_substring(start_s1, start_s2, left_length) and 
                    is_scramble_substring(start_s1 + left_length, start_s2 + left_length, right_length)):
                    return True
              
                # Case 2: Swap occurred at this level
                # Check if left part of s1 matches right part of s2 and vice versa
                if (is_scramble_substring(start_s1 + left_length, start_s2, right_length) and 
                    is_scramble_substring(start_s1, start_s2 + right_length, left_length)):
                    return True
          
            # No valid split found
            return False
      
        # Start the recursive check from the beginning of both strings
        return is_scramble_substring(0, 0, len(s1))
```

## Time and Space Complexity

The time complexity is `O(n^4)`, and the space complexity is `O(n^3)`, where `n` is the length of the string.

**Time Complexity Analysis:**
- The function `dfs(i, j, k)` has three parameters, each ranging from `0` to `n`, creating `O(n^3)` possible states.
- For each state with `k > 1`, we iterate through `h` from `1` to `k-1`, which takes `O(k)` iterations in the worst case, where `k` can be up to `n`.
- Within each iteration, we make recursive calls that are memoized, so each recursive call takes `O(1)` time to retrieve cached results.
- Therefore, the total time complexity is `O(n^3) × O(n) = O(n^4)`.

**Space Complexity Analysis:**
- The `@cache` decorator stores the results of all unique calls to `dfs(i, j, k)`.
- There are at most `n` possible values for each of `i`, `j`, and `k`, resulting in `O(n^3)` unique states to cache.
- The recursion depth is at most `O(n)` (when `k` starts at `n` and decreases by 1 each level).
- The dominant factor is the cache storage, giving us a space complexity of `O(n^3)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Forgetting Early Optimization: Character Frequency Check**

A common pitfall is diving straight into the recursive solution without first checking if the two strings have the same character frequencies. If `s1` and `s2` don't contain the exact same characters with the same frequencies, `s2` cannot possibly be a scrambled version of `s1`.

**Why this matters:** Without this check, the algorithm will perform expensive recursive calls even when the answer is obviously `False`, leading to unnecessary computation.

**Solution:**
Add a preliminary check before starting the recursion:

```python
from collections import Counter

class Solution:
    def isScramble(self, s1: str, s2: str) -> bool:
        # Early termination: check if both strings have same characters
        if Counter(s1) != Counter(s2):
            return False
          
        # If strings are identical, no need for complex checks
        if s1 == s2:
            return True
          
        # Proceed with the recursive solution...
        from functools import cache
      
        @cache
        def is_scramble_substring(start_s1: int, start_s2: int, length: int) -> bool:
            # ... rest of the implementation
```

### 2. **Index Calculation Errors in Swap Case**

The most error-prone part is calculating the correct indices when checking the swap scenario. It's easy to mix up:
- Which part of `s1` should match which part of `s2`
- The correct starting indices after swapping
- The correct lengths for each recursive call

**Common mistakes:**
- Using `j + h` instead of `j + k - h` for the swapped second part
- Using `i + k - h` instead of `i + h` for the first part's starting position
- Swapping the length parameters incorrectly

**Solution:**
Visualize the swap operation clearly:
```
Original: s1[i:i+k] vs s2[j:j+k]
Split s1 at position h: [i:i+h] and [i+h:i+k]
Split s2 for swap: [j:j+(k-h)] and [j+(k-h):j+k]

After swap matching:
- s1[i+h:i+k] (right part, length k-h) matches s2[j:j+(k-h)] (left part, length k-h)
- s1[i:i+h] (left part, length h) matches s2[j+(k-h):j+k] (right part, length h)
```

### 3. **Not Handling Edge Cases Properly**

**Common oversights:**
- Not checking if the strings have the same length initially
- Not handling empty strings (though typically not part of the problem constraints)
- Missing the case where both strings are identical

**Solution:**
Add comprehensive initial checks:

```python
class Solution:
    def isScramble(self, s1: str, s2: str) -> bool:
        # Edge case: different lengths
        if len(s1) != len(s2):
            return False
          
        # Edge case: empty strings
        if not s1:
            return True
          
        # Optimization: identical strings
        if s1 == s2:
            return True
          
        # Character frequency check
        if Counter(s1) != Counter(s2):
            return False
          
        # Proceed with main algorithm...
```

### 4. **Inefficient Base Case Check**

Some implementations check character frequency at every recursive level as a pruning technique, but doing this for every substring can actually slow down the algorithm due to the overhead of creating Counter objects repeatedly.

**Better approach:** Only do the frequency check at the top level, and trust the recursive structure to handle mismatches through the base case and split logic.

