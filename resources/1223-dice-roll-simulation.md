# 1223. Dice Roll Simulation

> Source: https://algo.monster/liteproblems/1223

## Problem Statement

You have a special die simulator that generates random numbers from 1 to 6. However, this simulator has a constraint: for each number `i` (from 1 to 6), it cannot appear more than `rollMax[i-1]` times consecutively.

Given:
- An integer `n` representing the number of times you'll roll the die
- An array `rollMax` of length 6, where `rollMax[i]` represents the maximum consecutive times that number `i+1` can appear

Your task is to find the total number of distinct valid sequences of length `n` that can be obtained. Since this number can be very large, return the result modulo `10^9 + 7`.

For example, if `rollMax = [1,1,2,2,2,3]` and `n = 2`:
- The sequence `[1,1]` is invalid because 1 can only appear at most 1 time consecutively
- The sequence `[1,2]` is valid
- The sequence `[3,3]` is valid because 3 can appear up to 2 times consecutively

Two sequences are considered different if they differ in at least one position.

## Intuition

When we think about building valid sequences, we need to keep track of three key pieces of information at each step:
1. How many rolls we've completed so far
2. What number was rolled last
3. How many times that number has appeared consecutively

This naturally leads to a recursive approach where we try all possible next rolls and check if they're valid based on our constraints.

For each position in our sequence, we have up to 6 choices (rolling 1 through 6). The validity of each choice depends on our current state:
- If we roll a different number than the last one, we can always do it (resetting the consecutive count to 1)
- If we roll the same number as the last one, we can only do it if we haven't exceeded the maximum consecutive limit for that number

The key insight is that we can use [dynamic programming](/problems/dynamic_programming_intro) with memoization because the same state `(position, last_number, consecutive_count)` will be encountered multiple times through different paths. For instance, reaching position 3 with the last roll being 4 and having it appear twice consecutively could happen through sequences like `[1,4,4,...]` or `[2,4,4,...]`. The number of valid ways to complete the sequence from this state is the same regardless of how we got there.

By defining our state as `dfs(i, j, x)` where:
- `i` = current position (how many dice we've rolled)
- `j` = the last number rolled
- `x` = consecutive count of number `j`

We can recursively compute all valid sequences and use memoization to avoid recalculating the same states. The base case is when we've rolled all `n` dice (`i >= n`), which represents one valid complete sequence.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution uses [dynamic programming](/problems/dynamic_programming_intro) with memoization through a recursive function `dfs(i, j, x)` that represents:
- `i`: the current position (number of dice rolls completed)
- `j`: the last number rolled (0 initially, 1-6 for actual rolls)
- `x`: the consecutive count of number `j`

**Implementation Details:**

1. **Base Case**: When `i >= n`, we've completed all `n` rolls successfully, so we return `1` to count this valid sequence.

2. **Recursive Case**: For each position, we try rolling each number from 1 to 6:
   - If the new number `k` is different from the last rolled number `j`:
     - We can always roll it, calling `dfs(i + 1, k, 1)` 
     - The consecutive count resets to 1 since we're starting a new number
 
   - If the new number `k` equals the last rolled number `j`:
     - We check if `x < rollMax[j - 1]` (haven't exceeded the limit)
     - If valid, we call `dfs(i + 1, j, x + 1)` with incremented consecutive count
     - If not valid, we skip this choice

3. **Memoization**: The `@cache` decorator automatically memoizes the function results, preventing redundant calculations for the same state `(i, j, x)`.

4. **Initial Call**: We start with `dfs(0, 0, 0)` where:
   - Position 0 (no rolls yet)
   - Last number 0 (special value indicating no previous roll)
   - Consecutive count 0

5. **Modulo Operation**: Since the answer can be very large, we take modulo `10^9 + 7` before returning from each recursive call.

The time complexity is `O(n × 6 × max(rollMax))` since we have at most that many unique states, and the space complexity is the same for storing the memoization cache.

## Example Walkthrough

Let's walk through a small example with `n = 3` and `rollMax = [1, 1, 2, 2, 2, 3]`.

We want to find all valid sequences of length 3 where:
- Number 1 can appear at most 1 time consecutively
- Number 2 can appear at most 1 time consecutively  
- Number 3 can appear at most 2 times consecutively
- Number 4 can appear at most 2 times consecutively
- Number 5 can appear at most 2 times consecutively
- Number 6 can appear at most 3 times consecutively

Starting with `dfs(0, 0, 0)` (position 0, no previous number, count 0):

**First Roll (position 0):**
Since there's no previous number (j=0), we can roll any number 1-6:
- Roll 1: → `dfs(1, 1, 1)`
- Roll 2: → `dfs(1, 2, 1)`
- Roll 3: → `dfs(1, 3, 1)`
- Roll 4: → `dfs(1, 4, 1)`
- Roll 5: → `dfs(1, 5, 1)`
- Roll 6: → `dfs(1, 6, 1)`

**Second Roll (position 1):** Let's trace the path starting with rolling 3:
From `dfs(1, 3, 1)` (last roll was 3, appearing once):
- Roll 1: Different from 3 → `dfs(2, 1, 1)` ✓
- Roll 2: Different from 3 → `dfs(2, 2, 1)` ✓
- Roll 3: Same as last, check if 1 < rollMax[2]=2 → Yes → `dfs(2, 3, 2)` ✓
- Roll 4: Different from 3 → `dfs(2, 4, 1)` ✓
- Roll 5: Different from 3 → `dfs(2, 5, 1)` ✓
- Roll 6: Different from 3 → `dfs(2, 6, 1)` ✓

**Third Roll (position 2):** Let's continue with the path [3, 3]:
From `dfs(2, 3, 2)` (last roll was 3, appearing twice consecutively):
- Roll 1: Different from 3 → `dfs(3, 1, 1)` → Base case: return 1 ✓
- Roll 2: Different from 3 → `dfs(3, 2, 1)` → Base case: return 1 ✓
- Roll 3: Same as last, check if 2 < rollMax[2]=2 → No → Skip ✗
- Roll 4: Different from 3 → `dfs(3, 4, 1)` → Base case: return 1 ✓
- Roll 5: Different from 3 → `dfs(3, 5, 1)` → Base case: return 1 ✓
- Roll 6: Different from 3 → `dfs(3, 6, 1)` → Base case: return 1 ✓

So from state `dfs(2, 3, 2)`, we get 5 valid sequences:
- [3, 3, 1]
- [3, 3, 2]
- [3, 3, 4]
- [3, 3, 5]
- [3, 3, 6]

Note that [3, 3, 3] is invalid because 3 cannot appear more than 2 times consecutively.

Similarly, sequences like [1, 1, 1] and [2, 2, 2] would be invalid because numbers 1 and 2 can only appear once consecutively.

The memoization ensures that when we encounter the same state (like reaching position 2 with last roll 4 and count 1) through different paths, we only compute it once and reuse the result.

## Python Solution

```python
class Solution:
    def dieSimulator(self, n: int, rollMax: List[int]) -> int:
        MOD = 10**9 + 7
      
        @cache
        def dfs(roll_index: int, last_number: int, consecutive_count: int) -> int:
            """
            Dynamic programming function to count valid dice roll sequences.
          
            Args:
                roll_index: Current position in the sequence (0 to n-1)
                last_number: The last rolled number (1-6, or 0 if no previous roll)
                consecutive_count: How many times the last_number appeared consecutively
          
            Returns:
                Number of valid sequences from this state
            """
            # Base case: completed all n rolls
            if roll_index >= n:
                return 1
          
            total_sequences = 0
          
            # Try rolling each die face (1 through 6)
            for die_face in range(1, 7):
                if die_face != last_number:
                    # Rolling a different number - reset consecutive count to 1
                    total_sequences += dfs(roll_index + 1, die_face, 1)
                elif consecutive_count < rollMax[last_number - 1]:
                    # Rolling the same number - check if within rollMax limit
                    # Note: rollMax is 0-indexed, so we use last_number - 1
                    total_sequences += dfs(roll_index + 1, last_number, consecutive_count + 1)
          
            return total_sequences % MOD
      
        # Start with roll_index=0, last_number=0 (no previous roll), consecutive_count=0
        return dfs(0, 0, 0)
```

## Time and Space Complexity

The time complexity is `O(n × k × M)`, where:
- `n` is the number of dice rolls
- `k = 6` is the number of faces on the die (constant)
- `M` is the maximum value in `rollMax` array

The space complexity is `O(n × k × M)` due to the memoization cache.

**Analysis:**

For the time complexity:
- The recursive function `dfs(i, j, x)` has three parameters:
  - `i` ranges from `0` to `n-1` (n possible values)
  - `j` ranges from `0` to `6` (7 possible values, including the initial 0)
  - `x` ranges from `0` to at most `max(rollMax)` (M possible values)
- Due to the `@cache` decorator, each unique state `(i, j, x)` is computed only once
- Total number of unique states: `n × 7 × M`
- For each state, we iterate through 6 possible dice values (constant work)
- Therefore, time complexity is `O(n × 7 × M)` which simplifies to `O(n × k × M)` where `k` represents the dice range

For the space complexity:
- The memoization cache stores all computed states
- Maximum number of states stored: `n × 7 × M`
- Additionally, the recursion stack depth can go up to `n`
- The dominant factor is the cache storage: `O(n × k × M)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Index Mapping for rollMax Array**
One of the most common mistakes is confusion about the indexing between die faces (1-6) and the rollMax array (0-indexed). When checking the constraint for die face `k`, you need to use `rollMax[k-1]` not `rollMax[k]`.

**Incorrect:**
```python
elif consecutive_count < rollMax[last_number]:  # Wrong! Array out of bounds when last_number=6
```

**Correct:**
```python
elif consecutive_count < rollMax[last_number - 1]:  # Correct 0-based indexing
```

### 2. **Forgetting to Apply Modulo Operation**
The problem requires the answer modulo 10^9 + 7, but it's easy to forget applying it, especially if you're accumulating results in a loop.

**Incorrect:**
```python
def dfs(roll_index, last_number, consecutive_count):
    if roll_index >= n:
        return 1
    total_sequences = 0
    for die_face in range(1, 7):
        # ... logic ...
        total_sequences += dfs(...)
    return total_sequences  # Missing modulo!
```

**Correct:**
```python
return total_sequences % MOD
```

### 3. **Improper Initial State Handling**
Starting with wrong initial values can lead to incorrect logic. A common mistake is starting with `last_number=1` or `consecutive_count=1`.

**Incorrect:**
```python
return dfs(0, 1, 1)  # Wrong! This assumes we already rolled a 1
```

**Correct:**
```python
return dfs(0, 0, 0)  # Start with no previous roll (last_number=0)
```

### 4. **Not Handling the First Roll Correctly**
When `last_number=0` (initial state), the logic for checking consecutive counts shouldn't apply. The current solution handles this correctly because `die_face != last_number` will always be true for the first roll.

**Potential Bug (if logic is restructured):**
```python
# If someone tries to optimize by checking rollMax first
if consecutive_count >= rollMax[die_face - 1]:
    continue  # This would incorrectly skip valid first rolls
```

### 5. **Memory Limit Issues with Large Inputs**
While `@cache` is convenient, for very large inputs, the memoization table might consume too much memory. In such cases, consider using iterative DP with space optimization.

**Alternative Approach for Memory Optimization:**
```python
def dieSimulator(self, n: int, rollMax: List[int]) -> int:
    MOD = 10**9 + 7
    # dp[i][j] = number of ways to roll i times with last number being j+1
    # and considering all valid consecutive counts
    dp = [[0] * 6 for _ in range(n + 1)]
    # Additional tracking for consecutive counts can be added if needed
```

