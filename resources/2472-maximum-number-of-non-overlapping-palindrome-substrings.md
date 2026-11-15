# 2472. Maximum Number of Non-overlapping Palindrome Substrings

> Source: https://algo.monster/liteproblems/2472

## Problem Statement

You are given a string `s` and a positive integer `k`. Your task is to select a set of non-overlapping substrings from the string that meet specific criteria and maximize the count of such substrings.

The selected substrings must satisfy two conditions:
1. Each substring must have a length of at least `k` characters
2. Each substring must be a palindrome (reads the same forwards and backwards)

The substrings you select cannot overlap with each other - meaning they cannot share any character positions in the original string.

Your goal is to find the maximum number of such palindromic substrings you can select from the string `s`.

For example, if you have a string like "abaccdbbd" and `k = 3`, you could potentially select palindromes like "aba" and "bdb" (which don't overlap), giving you a count of 2. The challenge is to find the optimal selection that maximizes this count.

A substring is defined as a contiguous sequence of characters within the string, meaning the characters must appear consecutively in the original string.

## Intuition

To solve this problem, we need to think about two main challenges: identifying all possible palindromes and then selecting the maximum number of non-overlapping ones.

The first insight is that we need to know which substrings are palindromes before we can make any selection decisions. Checking if a substring is a palindrome repeatedly would be inefficient, so we can precompute this information. We can use [dynamic programming](/problems/dynamic_programming_intro) to build a 2D table `dp[i][j]` that tells us whether the substring from index `i` to index `j` is a palindrome. The key observation here is that a substring `s[i...j]` is a palindrome if and only if `s[i] == s[j]` and the inner substring `s[i+1...j-1]` is also a palindrome.

Once we know all the palindromes, the second challenge is selecting the maximum number of non-overlapping ones. This is where the problem becomes interesting - it's essentially an optimization problem where we need to make choices. At each position in the string, we have a decision to make: should we include a palindrome starting at this position, or skip it and look for palindromes starting later?

This decision-making process naturally leads to a recursive approach. For any position `i` in the string, we can either:
1. Skip position `i` and find the best solution starting from position `i+1`
2. Choose a palindrome starting at position `i` (if one exists with length at least `k`), and then find the best solution starting after that palindrome ends

The key insight is that once we choose a palindrome from position `i` to position `j`, the next palindrome can only start from position `j+1` or later (to avoid overlapping). This creates a natural recursive structure where each decision leads to a smaller subproblem.

By using memoization with our recursive function `dfs(i)`, we avoid recalculating the same subproblems multiple times. The function `dfs(i)` represents the maximum number of palindromes we can select starting from position `i` onwards, considering all the constraints.

**Learn more about [Greedy](/problems/greedy_intro), [Two Pointers](/problems/two_pointers_intro) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution consists of two main phases: preprocessing to identify all palindromes and a memoized recursive search to find the maximum number of non-overlapping palindromes.

**Phase 1: Preprocessing Palindromes**

We create a 2D boolean table `dp[i][j]` where `dp[i][j] = True` if the substring `s[i...j]` is a palindrome. We build this table using [dynamic programming](/problems/dynamic_programming_intro):

- Initialize all single characters as palindromes: `dp[i][i] = True`
- For each substring, we check from the end of the string backwards (starting from `i = n-1` down to `0`)
- For each starting position `i`, we check all ending positions `j > i`
- A substring `s[i...j]` is a palindrome if: `s[i] == s[j]` and `dp[i+1][j-1]` is `True`

This preprocessing takes `O(n^2)` time and space, where `n` is the length of the string.

**Phase 2: Memoized Recursive Search**

We define a recursive function `dfs(i)` that returns the maximum number of non-overlapping palindromes that can be selected from `s[i...]` (the substring starting at position `i`).

The recurrence relation is:

```
dfs(i) = max(
    dfs(i + 1),  // Skip position i
    max{1 + dfs(j + 1) for all j where j >= i + k - 1 and dp[i][j] is True}
)
```

Breaking this down:
- Base case: If `i >= n`, return `0` (no more characters to process)
- We have two choices at each position:
  1. Skip the current position and solve for `dfs(i + 1)`
  2. Choose a palindrome starting at position `i` with length at least `k`

For the second choice, we iterate through all possible ending positions `j` where:
- `j >= i + k - 1` (ensuring the palindrome has at least `k` characters)
- `dp[i][j] = True` (ensuring the substring is actually a palindrome)

If we choose the palindrome from `i` to `j`, we add `1` to our count and continue searching from position `j + 1` (the first position after our chosen palindrome).

The `@cache` decorator (memoization) ensures that each unique state `dfs(i)` is computed only once, reducing the time complexity from exponential to `O(n^2)`.

The final answer is `dfs(0)`, which gives us the maximum number of non-overlapping palindromes starting from the beginning of the string.

## Example Walkthrough

Let's walk through the solution with the string `s = "abaccdbbd"` and `k = 3`.

**Phase 1: Building the Palindrome Table**

First, we build our `dp` table to identify all palindromes. We'll create a 9×9 table (string length is 9).

Starting with single characters (all are palindromes):
- `dp[0][0] = True` ('a')
- `dp[1][1] = True` ('b')
- ... and so on for all positions

Then we check longer substrings, working backwards from position 8 to 0:

For position 7 (character 'b'):
- Check `s[7...8]`: "bd" → 'b' ≠ 'd', so `dp[7][8] = False`

For position 6 (character 'b'):
- Check `s[6...7]`: "bb" → 'b' = 'b', so `dp[6][7] = True`
- Check `s[6...8]`: "bbd" → 'b' ≠ 'd', so `dp[6][8] = False`

Continuing this process, we identify key palindromes of length ≥ 3:
- `dp[0][2] = True`: "aba" (positions 0-2)
- `dp[2][4] = True`: "acc" (positions 2-4) - Wait, this is False ('a' ≠ 'c')
- `dp[3][5] = True`: "ccd" - False ('c' ≠ 'd')
- `dp[5][7] = True`: "dbb" - False ('d' ≠ 'b')
- `dp[6][8] = True`: "bbd" - False ('b' ≠ 'd')

Actually, let me recalculate the palindromes correctly:
- "aba" (0-2): 'a' = 'a' and 'b' is palindrome → `dp[0][2] = True`
- "bac" (1-3): 'b' ≠ 'c' → `dp[1][3] = False`
- "acc" (2-4): 'a' ≠ 'c' → `dp[2][4] = False`
- "ccd" (3-5): 'c' ≠ 'd' → `dp[3][5] = False`
- "cdb" (4-6): 'c' ≠ 'b' → `dp[4][6] = False`
- "dbb" (5-7): 'd' ≠ 'b' → `dp[5][7] = False`
- "bbd" (6-8): 'b' ≠ 'd' → `dp[6][8] = False`

Let's check for "bdb" (positions 1,5,6): This isn't contiguous! We need contiguous substrings.

Actually, looking more carefully at "abaccdbbd":
- Position 5-7: "dbb" → 'd' ≠ 'b', not a palindrome
- Position 4-6: "cdb" → 'c' ≠ 'b', not a palindrome

The valid palindromes of length ≥ 3 are:
- "aba" (positions 0-2): `dp[0][2] = True`

**Phase 2: Finding Maximum Non-overlapping Palindromes**

Now we use our recursive function `dfs(i)` to find the maximum count:

`dfs(0)`: Starting from position 0
- Option 1: Skip position 0 → `dfs(1)`
- Option 2: Take palindrome "aba" (0-2) → `1 + dfs(3)`

Let's evaluate `dfs(1)`: Starting from position 1
- No palindromes of length ≥ 3 start at position 1
- Return `dfs(2)`

`dfs(2)`: Starting from position 2
- No palindromes of length ≥ 3 start at position 2
- Continue similarly...

`dfs(3)`: Starting from position 3
- No palindromes of length ≥ 3 start here
- Continue checking...

Since we don't find any other palindromes of length ≥ 3 in the remaining string, most `dfs` calls return 0.

Therefore:
- `dfs(3)` = 0 (no more palindromes found)
- `dfs(0)` = max(dfs(1), 1 + dfs(3)) = max(0, 1 + 0) = 1

The maximum number of non-overlapping palindromes of length at least 3 is **1** (just "aba").

This example shows how the algorithm:
1. Precomputes all palindromes efficiently
2. Uses memoized recursion to avoid overlapping selections
3. Maximizes the count by trying all valid combinations

## Python Solution

```python
class Solution:
    def maxPalindromes(self, s: str, k: int) -> int:
        """
        Find the maximum number of non-overlapping palindromic substrings of length at least k.
      
        Args:
            s: Input string
            k: Minimum length of palindromic substrings
          
        Returns:
            Maximum number of non-overlapping palindromic substrings
        """
        from functools import cache
      
        @cache
        def find_max_palindromes(start_index: int) -> int:
            """
            Dynamic programming function to find maximum palindromes starting from index.
          
            Args:
                start_index: Current starting position in the string
              
            Returns:
                Maximum number of palindromes from this position
            """
            # Base case: reached end of string
            if start_index >= string_length:
                return 0
          
            # Option 1: Skip current position and check from next index
            max_count = find_max_palindromes(start_index + 1)
          
            # Option 2: Try to include palindromes starting at current position
            # Check all possible ending positions that form palindromes of length >= k
            for end_index in range(start_index + k - 1, string_length):
                if is_palindrome[start_index][end_index]:
                    # Include this palindrome and continue from position after it
                    max_count = max(max_count, 1 + find_max_palindromes(end_index + 1))
          
            return max_count
      
        string_length = len(s)
      
        # Build 2D DP table to check if substring s[i:j+1] is a palindrome
        # is_palindrome[i][j] = True if s[i:j+1] is a palindrome
        is_palindrome = [[True] * string_length for _ in range(string_length)]
      
        # Fill the palindrome table bottom-up
        # Start from the end of string and work backwards
        for start in range(string_length - 1, -1, -1):
            for end in range(start + 1, string_length):
                # A substring is palindrome if:
                # 1. First and last characters match
                # 2. Inner substring is also a palindrome (or length <= 2)
                is_palindrome[start][end] = (s[start] == s[end] and 
                                            is_palindrome[start + 1][end - 1])
      
        # Find the maximum number of palindromes
        result = find_max_palindromes(0)
      
        # Clear the cache to free memory
        find_max_palindromes.cache_clear()
      
        return result
```

## Time and Space Complexity

The time complexity is `O(n^2)` and the space complexity is `O(n^2)`, where `n` is the length of the string `s`.

**Time Complexity Analysis:**
- Building the palindrome DP table: The nested loops iterate through all pairs `(i, j)` where `i < j`, which takes `O(n^2)` time.
- The DFS function with memoization: Each state `i` (from 0 to n-1) is computed at most once due to the `@cache` decorator. For each state, we iterate through positions from `i + k - 1` to `n - 1`, which is at most `O(n)` operations. Therefore, the DFS contributes `O(n) * O(n) = O(n^2)` time.
- Overall time complexity: `O(n^2) + O(n^2) = O(n^2)`.

**Space Complexity Analysis:**
- The 2D DP table `dp[i][j]` stores palindrome information for all substring pairs, requiring `O(n^2)` space.
- The memoization cache for the DFS function stores at most `n` states, each taking `O(1)` space, contributing `O(n)` space.
- The recursion stack depth is at most `O(n)` in the worst case.
- Overall space complexity: `O(n^2) + O(n) + O(n) = O(n^2)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. Greedy Selection of Palindromes

**The Pitfall:**
A common mistake is attempting to solve this problem greedily by always selecting the first valid palindrome encountered. For example, given the string "aabaa" with k=2, a greedy approach might select "aa" at position 0-1, then "aa" at position 3-4, yielding 2 palindromes. However, the optimal solution is to select the single palindrome "aabaa" (positions 0-4), then potentially find more palindromes afterward.

**Why This Fails:**
Selecting shorter palindromes early can block the selection of longer palindromes that might lead to better overall solutions. The problem requires global optimization, not local optimization.

**Solution:**
The recursive approach with memoization correctly explores all possibilities - both taking and skipping palindromes at each position - ensuring we find the globally optimal solution.

### 2. Incorrect Palindrome DP Table Initialization

**The Pitfall:**
When building the palindrome checking table, incorrectly handling the base cases can lead to wrong results. Specifically:
- Forgetting that single characters are palindromes
- Not properly handling adjacent characters (length 2 substrings)
- Accessing `dp[i+1][j-1]` when `i+1 > j-1`, which can give incorrect results

**Example of Incorrect Code:**
```python
# WRONG: This doesn't handle edge cases properly
for start in range(string_length):
    for end in range(start + 1, string_length):
        is_palindrome[start][end] = (s[start] == s[end] and 
                                    is_palindrome[start + 1][end - 1])
```

**Solution:**
Initialize the diagonal (single characters) as True and process the table in the correct order (bottom-up from the end of the string):
```python
# Initialize all positions as True (handles single characters and edge cases)
is_palindrome = [[True] * string_length for _ in range(string_length)]

# Process from end to beginning to ensure dp[i+1][j-1] is computed before dp[i][j]
for start in range(string_length - 1, -1, -1):
    for end in range(start + 1, string_length):
        is_palindrome[start][end] = (s[start] == s[end] and 
                                    is_palindrome[start + 1][end - 1])
```

### 3. Off-by-One Errors in Index Handling

**The Pitfall:**
Confusion between inclusive and exclusive indices when:
- Calculating substring length (should be `end - start + 1`)
- Checking minimum length constraint (should be `end >= start + k - 1`)
- Continuing recursion after selecting a palindrome (should continue from `end + 1`)

**Example of Incorrect Code:**
```python
# WRONG: Off-by-one error in length check
for end_index in range(start_index + k, string_length):  # Should be start_index + k - 1
    if is_palindrome[start_index][end_index]:
        max_count = max(max_count, 1 + find_max_palindromes(end_index))  # Should be end_index + 1
```

**Solution:**
Be explicit about index calculations:
```python
# Correct: end_index is inclusive, so length = end - start + 1
# For length >= k, we need end >= start + k - 1
for end_index in range(start_index + k - 1, string_length):
    if is_palindrome[start_index][end_index]:
        # Continue from the position after the selected palindrome
        max_count = max(max_count, 1 + find_max_palindromes(end_index + 1))
```

### 4. Memory Optimization Opportunities Missed

**The Pitfall:**
While the solution works correctly, it might use more memory than necessary. The palindrome table uses O(n²) space, but we could potentially optimize this.

**Solution:**
For very large strings, consider using a set to store only valid palindrome ranges instead of a full 2D table:
```python
# Memory-optimized approach for sparse palindromes
valid_palindromes = set()
for start in range(string_length):
    for end in range(start + k - 1, string_length):
        if is_palindrome_check(s, start, end):  # Check palindrome on-the-fly
            valid_palindromes.add((start, end))
```

This is particularly useful when the number of valid palindromes of length ≥ k is much smaller than n².

