# 1406. Stone Game III

> Source: https://algo.monster/liteproblems/1406

## Problem Statement

Alice and Bob are playing a stone game with a row of stones, where each stone has an integer value given in the array `stoneValue`.

The game follows these rules:
- Alice and Bob take turns, with Alice going first
- On each turn, a player must take 1, 2, or 3 stones from the **beginning** of the remaining row
- Each player's score is the sum of values of all stones they've taken
- Both players start with a score of 0
- The game continues until all stones are taken
- Both players play optimally (making the best possible moves)

The goal is to determine who wins when both players play perfectly. Return:
- `"Alice"` if Alice wins (has a higher score)
- `"Bob"` if Bob wins (has a higher score)  
- `"Tie"` if both players end with the same score

For example, if `stoneValue = [1, 2, 3, 7]`:
- Alice could take 1 stone (value 1), then Bob might take 3 stones (values 2, 3, 7 = 12)
- Or Alice could take 2 stones (values 1, 2 = 3), leaving Bob with different options
- The key is that both players will choose the strategy that maximizes their own score

The challenge is to determine the outcome when both players make optimal decisions at every turn.

## Intuition

When both players play optimally, each player tries to maximize their own advantage. The key insight is that we can think about this game in terms of **score difference** rather than absolute scores.

Since players alternate turns, when it's my turn, I want to maximize the difference between my score and my opponent's score. Whatever stones I take add to my score, and then my opponent will play optimally on their turn to maximize their advantage.

This leads us to think recursively: at any position `i` in the array, the current player needs to decide whether to take 1, 2, or 3 stones. If they take `j+1` stones (where `j` can be 0, 1, or 2), they gain the sum of those stone values. But then the opponent gets to play from position `i+j+1`, and the opponent will also play optimally.

The crucial realization is that if we define `dfs(i)` as the maximum score difference the current player can achieve starting from position `i`, then:
- When I take some stones and gain their values, that's positive for my score difference
- When my opponent plays next, they'll achieve their best score difference from the remaining stones, which is negative for my score difference

So if I take `j+1` stones starting from position `i`, my net score difference is:
`(sum of stones I take) - (opponent's best score difference from remaining stones)`

This can be written as: `sum(stoneValue[i] to stoneValue[i+j]) - dfs(i+j+1)`

Since I want to maximize my advantage, I should choose the option (taking 1, 2, or 3 stones) that gives me the maximum score difference.

At the very beginning (`i=0`), Alice plays first. If `dfs(0) > 0`, it means Alice can achieve a positive score difference (she wins). If `dfs(0) < 0`, Bob wins. If `dfs(0) = 0`, it's a tie.

**Learn more about [Math](/problems/math-basics) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution implements a recursive approach with memoization to find the optimal outcome.

**Implementation Details:**

We define a recursive function `dfs(i)` that calculates the maximum score difference the current player can achieve when starting from index `i`:

1. **Base Case**: If `i >= n` (where `n` is the length of `stoneValue`), there are no more stones to take, so return `0`.

2. **Recursive Case**: For each valid move (taking 1, 2, or 3 stones), we:
   - Initialize `ans = -inf` to track the maximum score difference
   - Initialize `s = 0` to accumulate the sum of stones being taken
   - Try taking `j+1` stones (where `j` ranges from 0 to 2):
     - Add `stoneValue[i + j]` to our running sum `s`
     - Calculate the score difference as `s - dfs(i + j + 1)`
     - Update `ans` with the maximum value

3. **Memoization**: The `@cache` decorator is used to store previously computed results, preventing redundant calculations for the same state.

The formula for the recursive relation is:
```
dfs(i) = max(sum(stoneValue[i:i+j+1]) - dfs(i+j+1)) for j in {0, 1, 2}
```

**Final Result Determination:**

After computing `dfs(0)`, which represents the maximum score difference Alice can achieve as the first player:
- If `dfs(0) > 0`: Alice wins (return "Alice")
- If `dfs(0) < 0`: Bob wins (return "Bob")  
- If `dfs(0) == 0`: It's a tie (return "Tie")

**Time Complexity**: `O(n)` where `n` is the number of stones, since each state `i` is computed at most once due to memoization.

**Space Complexity**: `O(n)` for the memoization cache and recursion stack.

## Example Walkthrough

Let's walk through a small example with `stoneValue = [1, 2, 3, -9]` to illustrate how the solution works.

We'll trace through the recursive function `dfs(i)` which returns the maximum score difference the current player can achieve starting from index `i`.

**Starting from the end (building up our memoization):**

`dfs(4)`: Base case - no stones left, returns `0`

`dfs(3)`: Starting at index 3, only stone value `-9` remains
- Take 1 stone: gain `-9`, opponent gets `dfs(4) = 0`
  - Score difference: `-9 - 0 = -9`
- Can't take 2 or 3 stones (out of bounds)
- Maximum: `-9`

`dfs(2)`: Starting at index 2, stones `[3, -9]` remain
- Take 1 stone (value `3`): gain `3`, opponent gets `dfs(3) = -9`
  - Score difference: `3 - (-9) = 12`
- Take 2 stones (values `3, -9 = -6`): gain `-6`, opponent gets `dfs(4) = 0`
  - Score difference: `-6 - 0 = -6`
- Can't take 3 stones (out of bounds)
- Maximum: `12`

`dfs(1)`: Starting at index 1, stones `[2, 3, -9]` remain
- Take 1 stone (value `2`): gain `2`, opponent gets `dfs(2) = 12`
  - Score difference: `2 - 12 = -10`
- Take 2 stones (values `2, 3 = 5`): gain `5`, opponent gets `dfs(3) = -9`
  - Score difference: `5 - (-9) = 14`
- Take 3 stones (values `2, 3, -9 = -4`): gain `-4`, opponent gets `dfs(4) = 0`
  - Score difference: `-4 - 0 = -4`
- Maximum: `14`

`dfs(0)`: Starting at index 0, all stones `[1, 2, 3, -9]` available
- Take 1 stone (value `1`): gain `1`, opponent gets `dfs(1) = 14`
  - Score difference: `1 - 14 = -13`
- Take 2 stones (values `1, 2 = 3`): gain `3`, opponent gets `dfs(2) = 12`
  - Score difference: `3 - 12 = -9`
- Take 3 stones (values `1, 2, 3 = 6`): gain `6`, opponent gets `dfs(3) = -9`
  - Score difference: `6 - (-9) = 15`
- Maximum: `15`

**Result:** `dfs(0) = 15 > 0`, so Alice wins!

This means when both players play optimally, Alice can achieve a score that's 15 points higher than Bob's. Alice's optimal strategy is to take the first 3 stones (gaining 6 points), forcing Bob to take the last stone with value -9.

## Python Solution

```python
class Solution:
    def stoneGameIII(self, stoneValue: List[int]) -> str:
        from functools import cache
        from math import inf
      
        @cache
        def dfs(index: int) -> int:
            """
            Calculate the maximum score difference the current player can achieve
            starting from index position.
          
            Args:
                index: Current position in the stone array
              
            Returns:
                Maximum score difference (current player's score - opponent's score)
            """
            # Base case: no stones left
            if index >= n:
                return 0
          
            # Try taking 1, 2, or 3 stones and find the best outcome
            max_score_diff = -inf
            current_sum = 0
          
            for num_stones in range(3):
                # Check if we can take this many stones
                if index + num_stones >= n:
                    break
              
                # Add the stone value to current sum
                current_sum += stoneValue[index + num_stones]
              
                # Calculate score difference:
                # current_sum (what we take) - dfs(next) (opponent's best from remaining)
                score_diff = current_sum - dfs(index + num_stones + 1)
                max_score_diff = max(max_score_diff, score_diff)
          
            return max_score_diff
      
        # Initialize game parameters
        n = len(stoneValue)
      
        # Get Alice's maximum score difference
        alice_score_diff = dfs(0)
      
        # Determine winner based on score difference
        if alice_score_diff == 0:
            return 'Tie'
        elif alice_score_diff > 0:
            return 'Alice'
        else:
            return 'Bob'
```

## Time and Space Complexity

**Time Complexity:** `O(n)`

The algorithm uses memoization with the `@cache` decorator. The `dfs` function is called with parameter `i` ranging from `0` to `n-1`. Due to memoization, each unique state `dfs(i)` is computed only once. For each state, we iterate at most 3 times (constant time) to consider taking 1, 2, or 3 stones. Therefore, we have `n` unique states, each taking `O(1)` time to compute, resulting in a total time complexity of `O(n)`.

**Space Complexity:** `O(n)`

The space complexity consists of two components:
1. **Memoization cache:** The `@cache` decorator stores the results of `dfs(i)` for each unique value of `i` from `0` to `n-1`, requiring `O(n)` space.
2. **Recursion call stack:** In the worst case, the recursion depth can go up to `n` levels (when taking one stone at each step), requiring `O(n)` space for the call stack.

Both components contribute `O(n)` space, so the overall space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Score Calculation - Adding Opponent's Score Instead of Subtracting**

A common mistake is incorrectly calculating the score difference by adding the opponent's best score instead of subtracting it:

**Incorrect Implementation:**
```python
# WRONG: Adding opponent's score
score_diff = current_sum + dfs(index + num_stones + 1)
```

**Why it's wrong:** The recursive function returns the score difference from the current player's perspective. When we move to the next state, the opponent becomes the current player. So we need to subtract their best outcome from our current gain.

**Correct Implementation:**
```python
# CORRECT: Subtracting opponent's score
score_diff = current_sum - dfs(index + num_stones + 1)
```

### 2. **Off-by-One Error in Loop Range**

Another frequent mistake is using incorrect bounds when trying to take stones:

**Incorrect Implementation:**
```python
# WRONG: Loop goes from 1 to 3 instead of 0 to 2
for num_stones in range(1, 4):
    if index + num_stones > n:  # Also wrong boundary check
        break
    current_sum += stoneValue[index + num_stones - 1]
    score_diff = current_sum - dfs(index + num_stones)
```

**Why it's wrong:** 
- Using `range(1, 4)` with direct indexing creates confusion and potential index errors
- The boundary check `index + num_stones > n` is off by one

**Correct Implementation:**
```python
# CORRECT: Loop from 0 to 2, representing taking 1 to 3 stones
for num_stones in range(3):
    if index + num_stones >= n:
        break
    current_sum += stoneValue[index + num_stones]
    score_diff = current_sum - dfs(index + num_stones + 1)
```

### 3. **Forgetting to Accumulate Stone Values**

A subtle but critical error is recalculating the sum each iteration instead of accumulating:

**Incorrect Implementation:**
```python
for num_stones in range(3):
    if index + num_stones >= n:
        break
    # WRONG: Only considering single stone value, not cumulative
    stone_value = stoneValue[index + num_stones]
    score_diff = stone_value - dfs(index + num_stones + 1)
    max_score_diff = max(max_score_diff, score_diff)
```

**Why it's wrong:** When taking 2 or 3 stones, we need the sum of all stones taken, not just the last one.

**Correct Implementation:**
```python
current_sum = 0
for num_stones in range(3):
    if index + num_stones >= n:
        break
    # CORRECT: Accumulating all stone values
    current_sum += stoneValue[index + num_stones]
    score_diff = current_sum - dfs(index + num_stones + 1)
    max_score_diff = max(max_score_diff, score_diff)
```

### 4. **Using Global Variables Without Proper Initialization**

When using memoization with `@cache`, avoid relying on global variables that might change between test cases:

**Incorrect Implementation:**
```python
class Solution:
    def stoneGameIII(self, stoneValue: List[int]) -> str:
        n = len(stoneValue)
      
        @cache
        def dfs(index):
            # WRONG: Accessing stoneValue directly from outer scope
            # This can cause issues if the function is reused
            if index >= len(stoneValue):
                return 0
            # ... rest of the code
```

**Correct Implementation:**
```python
class Solution:
    def stoneGameIII(self, stoneValue: List[int]) -> str:
        n = len(stoneValue)
      
        @cache
        def dfs(index):
            # CORRECT: Using pre-computed n variable
            if index >= n:
                return 0
            # Access stoneValue through consistent scope
```

**Best Practice:** Clear the cache between different test cases if needed, or ensure the memoization key uniquely identifies the problem state.

