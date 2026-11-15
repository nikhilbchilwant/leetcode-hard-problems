# 233. Number of Digit One

> Source: https://algo.monster/liteproblems/233

## Problem Statement

Given an integer `n`, you need to count how many times the digit `1` appears in all non-negative integers from `0` to `n` (inclusive).

For example:
- If `n = 13`, you would count the digit `1` in: 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13
- The digit `1` appears in: 1 (once), 10 (once), 11 (twice), 12 (once), 13 (once)
- Total count = 1 + 1 + 2 + 1 + 1 = 6

The problem asks you to return the total count of how many times the digit `1` appears across all integers in the range `[0, n]`.

## Intuition

When counting digit occurrences across a range of numbers, a brute force approach would iterate through every number and count the `1`s, but this becomes inefficient for large values of `n`.

The key insight is that we can solve this problem digit by digit, considering the patterns of how `1`s appear in each position. This naturally leads us to think about Digit Dynamic Programming (Digit DP), where we build numbers digit by digit and keep track of our count along the way.

Think of it like this: when we're constructing numbers from `0` to `n`, we start from the leftmost digit and work our way right. At each position, we have choices for what digit to place (0-9), but we need to be careful not to exceed `n`. 

For example, if `n = 234`:
- At the first position, we can choose digits `0`, `1`, or `2`
- If we chose `2`, at the second position we can choose `0`, `1`, `2`, or `3`
- If we chose `2` then `3`, at the third position we can only choose `0`, `1`, `2`, `3`, or `4`

The crucial observation is that once we place a digit that's smaller than the corresponding digit in `n`, all subsequent positions become "free" - we can choose any digit from `0` to `9` without worrying about exceeding `n`.

This is where the `limit` flag comes in - it tracks whether we're still bounded by `n` or if we've already gone below it and have freedom in our choices.

As we build these numbers digit by digit, we maintain a count (`cnt`) of how many `1`s we've encountered so far. When we place a `1` at any position, we increment this count. By exploring all valid number constructions this way and accumulating the counts, we get our final answer.

The memoization (`@cache`) helps us avoid recalculating the same subproblems - if we've already computed how many `1`s appear when starting from a certain digit position with a certain count and limit state, we can reuse that result.

## Solution Approach

The solution uses Digit DP with memoized recursion. We convert the number `n` to a string `s` to easily access individual digits, then use a depth-first search function to explore all valid numbers.

**Core Function Design:**

The function `dfs(i, cnt, limit)` has three parameters:
- `i`: Current digit position being processed (0 is the leftmost/highest digit)
- `cnt`: Count of digit `1` seen so far in the current number being constructed
- `limit`: Boolean flag indicating if we're still bounded by the upper limit `n`

**Implementation Steps:**

1. **Base Case**: When `i >= len(s)`, we've processed all digit positions and return the accumulated count `cnt`.

2. **Determine Upper Bound**: 
   - If `limit` is `True`, we can only place digits from `0` to `s[i]` (the i-th digit of n)
   - If `limit` is `False`, we can place any digit from `0` to `9`
 
   ```python
   up = int(s[i]) if limit else 9
   ```

3. **Try All Valid Digits**: For each valid digit `j` from `0` to `up`:
   - Recursively call `dfs` for the next position
   - If `j == 1`, increment the count by 1
   - Update the `limit` flag: it remains `True` only if both current `limit` is `True` AND we chose the maximum possible digit (`j == up`)
 
   ```python
   for j in range(up + 1):
       ans += dfs(i + 1, cnt + (j == 1), limit and j == up)
   ```

4. **Memoization**: The `@cache` decorator automatically memoizes the results, preventing redundant calculations for the same `(i, cnt, limit)` state.

**Example Walkthrough:**

For `n = 13`:
- Start with `dfs(0, 0, True)`
- At position 0: can choose 0 or 1 (since first digit of 13 is 1)
  - If we choose 0: `dfs(1, 0, False)` - no longer limited
  - If we choose 1: `dfs(1, 1, True)` - still limited, count increases by 1
- At position 1 (when still limited after choosing 1): can choose 0, 1, 2, or 3
  - Each choice leads to different recursive calls
- The recursion aggregates all counts from valid number constructions

**Time and Space Complexity:**

- Time: `O(m² × D)` where `m` is the number of digits in `n` and `D = 10` (possible digit values)
- Space: `O(m²)` for the memoization cache

The solution efficiently counts all occurrences of digit `1` by exploring the number space systematically while avoiding redundant computations through memoization.

## Example Walkthrough

Let's trace through the solution with `n = 23` to understand how the Digit DP approach works.

**Setup:**
- Convert `n = 23` to string: `s = "23"`
- Start with `dfs(0, 0, True)` - position 0, count 0, limited by n

**Step 1: First digit (tens place)**
- Current state: `dfs(0, 0, True)`
- Since `limit = True`, we can choose digits 0, 1, or 2 (up to s[0] = '2')

**Choice 1a: Place '0' at tens place**
- Call `dfs(1, 0, False)` - move to position 1, count stays 0, no longer limited
- At position 1, since `limit = False`, we can choose any digit 0-9
- This represents numbers 00-09 (effectively 0-9)
- For digit '1' at ones place: `dfs(2, 1, False)` returns 1
- Total from this branch: 1 (only the number 01)

**Choice 1b: Place '1' at tens place**
- Call `dfs(1, 1, False)` - move to position 1, count becomes 1, no longer limited
- At position 1, since `limit = False`, we can choose any digit 0-9
- This represents numbers 10-19
- Each number already has one '1' from tens place
- For digit '1' at ones place (number 11): `dfs(2, 2, False)` returns 2
- For other digits at ones place: `dfs(2, 1, False)` returns 1 each
- Total from this branch: 1×9 + 2×1 = 11 (ten '1's from tens place, plus one more from 11)

**Choice 1c: Place '2' at tens place**
- Call `dfs(1, 0, True)` - move to position 1, count stays 0, still limited
- At position 1, since `limit = True`, we can only choose digits 0-3 (up to s[1] = '3')
- This represents numbers 20-23
- For digit '1' at ones place (number 21): `dfs(2, 1, True/False)` returns 1
- For other digits: `dfs(2, 0, True/False)` returns 0
- Total from this branch: 1 (only from number 21)

**Step 2: Aggregation**
- Total count = 1 + 11 + 1 = 13

**Verification:**
Numbers from 0 to 23 containing '1':
- 1 (one '1')
- 10, 11, 12, 13, 14, 15, 16, 17, 18, 19 (ten numbers, with 11 having two '1's)
- 21 (one '1')

Total: 1 + 10 + 1 + 1 = 13 ✓

The memoization ensures that identical subproblems (like counting from certain positions with the same count and limit values) are only computed once, making the algorithm efficient even for large values of n.

## Python Solution

```python
class Solution:
    def countDigitOne(self, n: int) -> int:
        """
        Count the total number of digit 1 appearing in all non-negative integers less than or equal to n.
        Uses digit DP with memoization to solve the problem.
        """
        from functools import cache
      
        @cache
        def count_ones_dp(position: int, ones_count: int, is_limited: bool) -> int:
            """
            Dynamic programming function to count digit 1s.
          
            Args:
                position: Current position in the number string (0-indexed from left)
                ones_count: Count of 1s accumulated so far in the current number being formed
                is_limited: Whether the current number is still bounded by the original number n
          
            Returns:
                Total count of digit 1s in all valid numbers from current state
            """
            # Base case: reached the end of the number
            if position >= len(number_str):
                return ones_count
          
            # Determine the upper bound for current digit
            # If limited by original number, use the digit at current position
            # Otherwise, we can use any digit from 0-9
            upper_bound = int(number_str[position]) if is_limited else 9
          
            total_count = 0
          
            # Try all possible digits from 0 to upper_bound
            for digit in range(upper_bound + 1):
                # Recursively calculate for next position
                # Update ones_count if current digit is 1
                # Update is_limited flag: remains True only if we're still at the limit and chose the max digit
                total_count += count_ones_dp(
                    position + 1, 
                    ones_count + (1 if digit == 1 else 0),
                    is_limited and (digit == upper_bound)
                )
          
            return total_count
      
        # Convert number to string for digit-by-digit processing
        number_str = str(n)
      
        # Start DFS from position 0, with 0 ones counted, and limited by n
        return count_ones_dp(0, 0, True)
```

## Time and Space Complexity

**Time Complexity:** `O(log(n) * 10 * log(n))` which simplifies to `O(log²(n))`

The recursive function explores a tree-like structure where:
- The depth of recursion is `len(s)`, which equals `O(log(n))` since the string representation of `n` has `log₁₀(n)` digits
- At each recursive level `i`, we iterate through at most 10 possible digits (0-9)
- Each state `(i, cnt, limit)` is computed only once due to memoization with `@cache`
- The total number of unique states is `O(log(n) * log(n) * 2)` because:
  - `i` ranges from 0 to `len(s) - 1`: `O(log(n))` possibilities
  - `cnt` represents the count of digit 1s encountered so far, which can be at most `log(n)`
  - `limit` is a boolean with 2 possible values
- Each state performs `O(10)` work in the loop

Therefore, the overall time complexity is `O(log(n) * log(n) * 2 * 10) = O(log²(n))`

**Space Complexity:** `O(log²(n))`

The space is consumed by:
- The recursion call stack depth: `O(log(n))`
- The memoization cache storing all unique states: `O(log(n) * log(n) * 2) = O(log²(n))`
- The string `s` storing the digits: `O(log(n))`

The dominant factor is the cache storage, giving us `O(log²(n))` space complexity.

## Common Pitfalls

### 1. **Incorrect Limit Flag Update**

**Pitfall**: A common mistake is incorrectly updating the `limit` flag when transitioning to the next digit position. Developers often write:
```python
# WRONG: This doesn't properly track the boundary
total_count += count_ones_dp(position + 1, ones_count + (1 if digit == 1 else 0), is_limited)
```

**Why it's wrong**: Once we choose a digit less than the maximum allowed digit at any position, all subsequent positions are no longer limited by the original number. The limit should only remain `True` if we're currently limited AND we chose the maximum possible digit.

**Solution**:
```python
# CORRECT: Properly update the limit flag
total_count += count_ones_dp(
    position + 1, 
    ones_count + (1 if digit == 1 else 0),
    is_limited and (digit == upper_bound)  # Only stay limited if we chose the max digit
)
```

### 2. **Accumulating Count vs. Returning Count**

**Pitfall**: Confusing whether to accumulate the count of 1s during recursion or return it at the base case. Some implementations try to add 1 to the result every time they encounter a digit 1:
```python
# WRONG: Trying to accumulate in the wrong way
if digit == 1:
    total_count += 1  # This counts individual digits, not complete numbers
total_count += count_ones_dp(position + 1, ones_count, is_limited and digit == upper_bound)
```

**Why it's wrong**: The function should track how many 1s are in the current number being formed and return that count only when a complete number is formed (at the base case).

**Solution**:
```python
# CORRECT: Track count in parameter, return at base case
if position >= len(number_str):
    return ones_count  # Return accumulated count for this complete number

# Pass updated count to next recursion level
total_count += count_ones_dp(
    position + 1, 
    ones_count + (1 if digit == 1 else 0),  # Update count in parameter
    is_limited and (digit == upper_bound)
)
```

### 3. **Missing or Incorrect Memoization State**

**Pitfall**: Not including all necessary state variables in the memoization, particularly forgetting the `ones_count` parameter:
```python
# WRONG: Missing ones_count in memoization
@cache
def count_ones_dp(position: int, is_limited: bool) -> int:
    # This won't correctly memoize different paths with different 1 counts
```

**Why it's wrong**: Different paths through the recursion tree can reach the same position with different counts of 1s. Without including `ones_count` in the memoization key, we'll incorrectly reuse results from paths with different 1 counts.

**Solution**: Include all state variables that affect the result:
```python
# CORRECT: Include all necessary state
@cache
def count_ones_dp(position: int, ones_count: int, is_limited: bool) -> int:
    # Properly memoizes based on position, count, and limit
```

### 4. **Off-by-One Error with Range**

**Pitfall**: Using incorrect range bounds when iterating through possible digits:
```python
# WRONG: Missing the upper_bound digit itself
for digit in range(upper_bound):  # This excludes upper_bound
```

**Why it's wrong**: When `is_limited` is `True`, we need to include the digit at the current position of `n` as a valid choice. Using `range(upper_bound)` would exclude it.

**Solution**:
```python
# CORRECT: Include upper_bound in the range
for digit in range(upper_bound + 1):  # Includes 0 through upper_bound
```

