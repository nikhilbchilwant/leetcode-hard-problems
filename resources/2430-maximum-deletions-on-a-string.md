# 2430. Maximum Deletions on a String

> Source: https://algo.monster/liteproblems/2430

## Problem Statement

You are given a string `s` consisting of only lowercase English letters. You need to delete the entire string using a series of operations and find the maximum number of operations possible.

In each operation, you can choose one of two actions:
1. Delete the **entire remaining string** `s` (this counts as 1 operation)
2. Delete the **first** `i` letters of `s` if these first `i` letters are **exactly equal** to the next `i` letters that follow them, where `i` can be any value from `1` to `s.length / 2`

For the second type of operation, you're essentially looking for a prefix that repeats immediately after itself. When you find such a repeating pattern, you can delete the first occurrence.

**Example:** 
- If `s = "ababc"`, you can observe that the first 2 letters `"ab"` are equal to the next 2 letters `"ab"`. So you can delete the first `"ab"` to get `"abc"` (1 operation).
- From `"abc"`, you cannot find any repeating prefix pattern, so you must delete the entire remaining string (1 operation).
- Total: 2 operations

The goal is to find the **maximum** number of operations needed to delete all characters from the string. You want to maximize the number of steps by choosing the optimal deletion strategy at each point.

## Intuition

The key insight is that we want to maximize the number of operations, which means we should try to make as many small deletions as possible rather than deleting large chunks or the entire string at once.

Think about it this way: if we can find a repeating pattern at the beginning of our string, we have a choice. We could either:
1. Delete the entire string (1 operation)
2. Delete just the first occurrence of the pattern and continue with the remaining string

The second option is often better because it gives us more opportunities to perform additional operations on the remaining string.

This naturally leads to a recursive approach. At each step, we look at the current string and ask: "What's the maximum number of operations I can perform starting from here?" 

For any position `i` in the string, we need to check all possible prefix lengths `j` (from 1 to half of the remaining string length). If we find that `s[i:i+j]` equals `s[i+j:i+2j]`, we've found a valid repeating pattern. We can delete the first `j` characters and then solve the same problem for the substring starting at position `i+j`.

The recursive nature suggests [dynamic programming](/problems/dynamic_programming_intro). We can use memoization to avoid recalculating the same subproblems. The function `dfs(i)` represents "what's the maximum number of operations needed to delete everything from position `i` onwards?"

The base case is when we've reached the end of the string (`i == n`), where no more operations are needed, so we return 0.

For each position, we always have the option to delete the entire remaining string (which gives us 1 operation). But we also check if we can do better by finding repeating prefixes. Among all valid options, we choose the one that gives us the maximum number of operations: `max(1, 1 + dfs(i+j))` for all valid `j`.

This greedy strategy of always choosing the option that maximizes operations, combined with exploring all possibilities through recursion and memoization, gives us the optimal solution.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution uses **memoization search** ([dynamic programming](/problems/dynamic_programming_intro) with recursion) to find the maximum number of operations.

### Core Function: `dfs(i)`

We define a recursive function `dfs(i)` that calculates the maximum number of operations needed to delete all characters from index `i` to the end of the string.

**Base Case:**
- When `i == n` (reached the end of string), return `0` since no characters remain to delete.

**Recursive Case:**
For each position `i`, we have two strategies:
1. Delete the entire remaining substring `s[i:]` - this gives us exactly 1 operation
2. Look for repeating prefixes and delete them

### Finding Repeating Prefixes

For the second strategy, we enumerate all possible prefix lengths `j` where:
- `1 ≤ j ≤ (n - i) // 2` (the prefix can be at most half of the remaining string)
- Check if `s[i:i+j] == s[i+j:i+2j]` (the prefix equals the immediately following substring)

When we find such a repeating pattern, we can:
- Delete the first `j` characters (the first occurrence of the pattern)
- Recursively solve for the remaining string starting at position `i+j`
- This gives us `1 + dfs(i+j)` operations total

### Maximizing Operations

Among all valid options (including deleting the entire remaining string), we choose the one that maximizes the number of operations:

```python
ans = 1  # Option to delete entire remaining string
for j in range(1, (n - i) // 2 + 1):
    if s[i:i+j] == s[i+j:i+2j]:
        ans = max(ans, 1 + dfs(i+j))
```

### Memoization with `@cache`

The `@cache` decorator is crucial for efficiency. Without it, we would recalculate `dfs(i)` for the same positions multiple times. The decorator automatically stores the results of function calls, so when `dfs(i)` is called with the same `i` value again, it returns the cached result instead of recalculating.

### Time Complexity Analysis

- For each position `i`, we check up to `(n-i)/2` possible prefix lengths
- String comparison `s[i:i+j] == s[i+j:i+2j]` takes `O(j)` time
- With memoization, each position is calculated only once
- Overall complexity: `O(n³)` where `n` is the length of the string

The solution starts by calling `dfs(0)` to process the entire string from the beginning, and returns the maximum number of operations possible.

## Example Walkthrough

Let's walk through the solution with `s = "aabbaabb"`.

**Starting at position 0:** `dfs(0)` processes the entire string "aabbaabb"
- Option 1: Delete entire string → 1 operation
- Option 2: Check for repeating prefixes:
  - j=1: "a" vs "a" ✓ (matches!) → Delete "a", get 1 + dfs(1)
  - j=2: "aa" vs "bb" ✗ (no match)
  - j=3: "aab" vs "baa" ✗ (no match)
  - j=4: "aabb" vs "aabb" ✓ (matches!) → Delete "aabb", get 1 + dfs(4)

Let's explore the j=4 path: `1 + dfs(4)`

**At position 4:** `dfs(4)` processes "aabb"
- Option 1: Delete entire string → 1 operation
- Option 2: Check for repeating prefixes:
  - j=1: "a" vs "a" ✓ (matches!) → Delete "a", get 1 + dfs(5)
  - j=2: "aa" vs "bb" ✗ (no match)

Following j=1: `1 + dfs(5)`

**At position 5:** `dfs(5)` processes "abb"
- Option 1: Delete entire string → 1 operation
- Option 2: Check for repeating prefixes:
  - j=1: "a" vs "b" ✗ (no match)
- Best option: Delete entire string → 1 operation

**Backtracking:**
- `dfs(5)` = 1
- `dfs(4)` = max(1, 1 + dfs(5)) = max(1, 1 + 1) = 2
- `dfs(0)` = max(1, ..., 1 + dfs(4)) = max(1, ..., 1 + 2) = 3

The maximum number of operations is **3**:
1. Delete "aabb" from "aabbaabb" → "aabb"
2. Delete "a" from "aabb" → "abb"  
3. Delete "abb" entirely

The memoization ensures that if we encounter the same substring position again (like if multiple paths lead to `dfs(4)`), we don't recalculate it.

## Python Solution

```python
class Solution:
    def deleteString(self, s: str) -> int:
        from functools import cache
      
        n = len(s)
      
        @cache
        def dfs(start_index: int) -> int:
            """
            Recursively find the maximum number of operations to delete the string.
          
            Args:
                start_index: Current starting position in the string
          
            Returns:
                Maximum number of delete operations from current position
            """
            # Base case: reached the end of string
            if start_index == n:
                return 0
          
            # At minimum, we can delete the entire remaining string as one operation
            max_operations = 1
          
            # Try all possible lengths for the first part (up to half of remaining string)
            # Since we need two equal consecutive parts, max length is half of remaining
            max_length = (n - start_index) // 2
          
            for length in range(1, max_length + 1):
                # Extract first part and second part of equal length
                first_part = s[start_index : start_index + length]
                second_part = s[start_index + length : start_index + 2 * length]
              
                # If the two consecutive parts are equal, we can delete the first part
                if first_part == second_part:
                    # Recursively solve for the remaining string after deleting first part
                    operations = 1 + dfs(start_index + length)
                    max_operations = max(max_operations, operations)
          
            return max_operations
      
        # Start the recursion from index 0
        return dfs(0)
```

## Time and Space Complexity

The time complexity is `O(n^3)`, and the space complexity is `O(n^2)`.

**Time Complexity Analysis:**
- The `dfs` function can be called with at most `n` different values of `i` (from 0 to n-1).
- For each call to `dfs(i)`, the loop iterates up to `(n-i)//2` times, which is `O(n)` in the worst case.
- Inside each iteration, the string comparison `s[i:i+j] == s[i+j:i+j+j]` takes `O(j)` time, where `j` can be up to `n/2`.
- Therefore, for each `dfs` call, the work done is `O(n^2)` (summing up all the string comparisons).
- Since we have `O(n)` unique calls due to memoization, the total time complexity is `O(n) * O(n^2) = O(n^3)`.

**Space Complexity Analysis:**
- The `@cache` decorator stores the results of all unique calls to `dfs`, which can be at most `n` different values.
- The recursion depth can be at most `O(n)` in the worst case.
- Each string slicing operation `s[i:i+j]` creates a new string, but these are temporary and don't accumulate.
- The dominant space usage comes from the cache storing `O(n)` results and the string slicing operations which can create strings of total length `O(n^2)` across all comparisons.
- Therefore, the space complexity is `O(n^2)`.

Note: The reference answer states `O(n^2)` for time complexity, which would be achievable with more efficient string comparison methods like using rolling hash or Z-algorithm, but the given implementation has `O(n^3)` time complexity due to the naive string comparison.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **String Slicing Performance Overhead**
The most common pitfall in this solution is the repeated string slicing operations `s[i:i+j] == s[i+j:i+2j]` inside the loop. Each string slice creates a new string object, which takes O(j) time and space. When this happens for every possible length at every position, it significantly impacts performance.

**Problem Example:**
```python
# This comparison creates two new string objects every time
if s[start_index : start_index + length] == s[start_index + length : start_index + 2 * length]:
```

**Solution:** Use character-by-character comparison or precompute a 2D table for substring equality:

```python
class Solution:
    def deleteString(self, s: str) -> int:
        from functools import cache
      
        n = len(s)
      
        # Precompute substring equality in O(n²)
        # equal[i][j] = True if s[i:i+j] == s[i+j:i+2j]
        equal = [[False] * (n // 2 + 1) for _ in range(n)]
      
        for i in range(n):
            for length in range(1, (n - i) // 2 + 1):
                is_equal = True
                for k in range(length):
                    if s[i + k] != s[i + length + k]:
                        is_equal = False
                        break
                equal[i][length] = is_equal
      
        @cache
        def dfs(start_index: int) -> int:
            if start_index == n:
                return 0
          
            max_operations = 1
            max_length = (n - start_index) // 2
          
            for length in range(1, max_length + 1):
                # Use precomputed result instead of string slicing
                if equal[start_index][length]:
                    operations = 1 + dfs(start_index + length)
                    max_operations = max(max_operations, operations)
          
            return max_operations
      
        return dfs(0)
```

### 2. **Forgetting to Clear Cache Between Test Cases**
When using `@cache` decorator in competitive programming platforms or when running multiple test cases, the cache persists between function calls. This can lead to incorrect results if the function is not properly isolated.

**Problem Example:**
```python
@cache
def dfs(i):  # Cache persists across different string inputs!
    # ...
```

**Solution:** Either define the cached function inside the main function (as shown in the original solution) or manually clear the cache:

```python
class Solution:
    def deleteString(self, s: str) -> int:
        # Define dfs inside deleteString so each call gets a fresh cache
        @cache
        def dfs(start_index: int) -> int:
            # implementation
            pass
      
        result = dfs(0)
        dfs.cache_clear()  # Optional: explicitly clear cache
        return result
```

### 3. **Off-by-One Errors in Range Calculation**
A subtle but common mistake is incorrectly calculating the maximum valid length for the prefix:

**Problem Example:**
```python
# Wrong: might go out of bounds
for length in range(1, n - start_index):  # Should be (n - start_index) // 2 + 1
```

**Solution:** Always ensure the range is `range(1, (n - start_index) // 2 + 1)` to guarantee both parts fit within the remaining string.

### 4. **Missing Base Case or Incorrect Return Value**
Forgetting to handle the base case when the entire string has been processed:

**Problem Example:**
```python
def dfs(start_index):
    # Missing base case - will cause infinite recursion
    max_operations = 1
    # ...
```

**Solution:** Always check if `start_index == n` and return 0 at the beginning of the function.

