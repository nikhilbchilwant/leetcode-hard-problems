# 1872. Stone Game VIII

> Source: https://algo.monster/liteproblems/1872

## Problem Statement

Alice and Bob are playing a stone game where they take turns, with Alice going first.

The game starts with `n` stones arranged in a row, where each stone has a value given in the array `stones`.

On each player's turn (while there's more than one stone remaining), the player must:
1. Choose an integer `x > 1` and remove the leftmost `x` stones from the row
2. Add the sum of the removed stones' values to their score
3. Place a new stone on the left side of the row, with a value equal to the sum of the removed stones

The game continues until only one stone remains.

The goal is to find the score difference (Alice's score - Bob's score) when both players play optimally. Alice wants to maximize this difference, while Bob wants to minimize it.

For example, if `stones = [7, 90, 5, 1, 100, 10, 10, 2]`:
- A player might take the first 2 stones `[7, 90]`, score 97 points, and place a stone with value 97 on the left
- The new row becomes `[97, 5, 1, 100, 10, 10, 2]`
- The next player continues from this state

The key insight is that taking `x` leftmost stones and replacing them with their sum is equivalent to merging those stones while maintaining the [prefix sum](/problems/subarray_sum). Each move essentially chooses a prefix of length at least 2, scores its sum, and passes the modified array to the opponent.

## Intuition

The first key observation is that when we take the leftmost `x` stones and replace them with their sum, we're essentially merging stones while keeping the total sum unchanged. This means we can think of each move as choosing a prefix of the array to "collapse" into a single stone.

Since we must take at least 2 stones (`x > 1`), the minimum valid move is taking the first 2 stones. After any move, the resulting array has the same prefix sums - just with fewer elements. For instance, if we take stones `[7, 90, 5]` with sum 102, the new array starts with `[102, ...]`, and the [prefix sum](/problems/subarray_sum) at position 0 is still 102.

This leads to a crucial insight: instead of tracking the actual stones, we can work with prefix sums. When a player takes the first `i+1` stones (indices 0 to i), they score `s[i]` points, where `s` is the [prefix sum](/problems/subarray_sum) array.

Now, let's think about the game from a minimax perspective. Each player wants to maximize their own advantage, which is equivalent to maximizing (their score - opponent's score). When it's my turn, I want to maximize this difference; when it's the opponent's turn, they want to maximize their difference, which means minimizing mine.

Consider the decision at any state: if I can choose to take stones starting from position `i`, I have two main options:
1. Take the minimum allowed stones (positions `i` and `i+1`), scoring `s[i]`, then pass the remaining game to my opponent
2. Pass on taking position `i`, letting my opponent potentially take it

If I take stones up to position `i`, I score `s[i]` points, but then my opponent plays optimally from position `i+1`. So my net advantage becomes `s[i] - dfs(i+1)`.

If I don't take position `i` now, I'm essentially passing the decision to take from position `i+1` onwards, giving me advantage `dfs(i+1)`.

Therefore, `dfs(i) = max(s[i] - dfs(i+1), dfs(i+1))` - I choose whichever option gives me the better score difference.

The base case is when `i >= n-1`: there's only one way to end the game (take all remaining stones), giving a score of `s[n-1]`.

Starting from `dfs(1)` makes sense because Alice must take at least 2 stones on her first move, meaning she effectively chooses whether to take stones starting from index 1 or later.

**Learn more about [Math](/problems/math-basics), [Dynamic Programming](/problems/dynamic_programming_intro) and [Prefix Sum](/problems/subarray_sum) patterns.**

## Solution Approach

The solution implements a top-down [dynamic programming](/problems/dynamic_programming_intro) approach with memoization using Python's `@cache` decorator.

**Step 1: Calculate Prefix Sums**

First, we compute the [prefix sum](/problems/subarray_sum) array using `accumulate(stones)`:
```python
s = list(accumulate(stones))
```
This gives us `s[i]` = sum of `stones[0]` to `stones[i]`, which represents the score when taking the first `i+1` stones.

**Step 2: Define the Recursive Function**

We define `dfs(i)` to represent the maximum score difference the current player can achieve when they can choose to take stones starting from index `i` or later:

```python
@cache
def dfs(i: int) -> int:
    if i >= len(stones) - 1:
        return s[-1]
    return max(dfs(i + 1), s[i] - dfs(i + 1))
```

**Understanding the Recursion:**

- **Base Case**: When `i >= n - 1`, only one stone remains (or we're at the last stone). The current player must take all remaining stones, scoring `s[-1]` (the total sum).

- **Recursive Case**: The current player has two choices:
  1. **Skip taking stones at position `i`**: Pass the decision to position `i+1`, getting advantage `dfs(i + 1)`
  2. **Take stones up to position `i`**: Score `s[i]` points, then the opponent plays optimally from position `i+1`. The net advantage is `s[i] - dfs(i + 1)` (my score minus opponent's best advantage)

The player chooses the maximum of these two options.

**Step 3: Start the Game**

We call `dfs(1)` because Alice's first move must take at least 2 stones (indices 0 and 1 at minimum), so she effectively decides whether to take stones starting from index 1 or beyond.

**Memoization:**

The `@cache` decorator automatically memoizes the results of `dfs(i)`, preventing redundant calculations. Each state `i` is computed only once, reducing the time complexity from exponential to O(n).

**Time and Space Complexity:**
- Time: O(n) - we compute each state from 1 to n-1 exactly once
- Space: O(n) - for the [prefix sum](/problems/subarray_sum) array and memoization cache

## Example Walkthrough

Let's walk through a small example with `stones = [3, 7, 2, 3]`.

**Step 1: Calculate prefix sums**
- `s = [3, 10, 12, 15]`
- `s[0] = 3`, `s[1] = 10`, `s[2] = 12`, `s[3] = 15`

**Step 2: Build the solution bottom-up using our recursive logic**

We'll trace through `dfs(i)` for each position, starting from the base cases:

**Base case: `dfs(3)`**
- Since `i = 3 >= len(stones) - 1`, we return `s[-1] = 15`
- This means if the game reaches a state where only index 3 can be taken, the current player scores 15

**Calculate `dfs(2)`**
- We have two choices:
  1. Skip position 2: get `dfs(3) = 15`
  2. Take up to position 2: score `s[2] = 12`, opponent gets `dfs(3) = 15`, net = `12 - 15 = -3`
- `dfs(2) = max(15, -3) = 15`
- The optimal play is to skip and let the opponent take first

**Calculate `dfs(1)`**
- We have two choices:
  1. Skip position 1: get `dfs(2) = 15`
  2. Take up to position 1: score `s[1] = 10`, opponent gets `dfs(2) = 15`, net = `10 - 15 = -5`
- `dfs(1) = max(15, -5) = 15`
- Again, optimal play is to skip

**Game Play Simulation:**
Starting with `dfs(1) = 15`, let's see how the game actually unfolds:
1. Alice starts and `dfs(1) = 15` tells us she should skip taking at position 1
2. Since Alice skips position 1, she's effectively choosing to skip position 2 as well (from `dfs(2)`)
3. Alice ends up taking all 4 stones on her first move: `[3, 7, 2, 3]`
4. Alice scores 15, Bob scores 0 (game ends after one move)
5. Score difference = 15 - 0 = 15 ✓

This matches our calculated result of `dfs(1) = 15`.

**Alternative scenario:** What if Alice took only the first 2 stones?
1. Alice takes `[3, 7]`, scores 10, places stone with value 10
2. New array: `[10, 2, 3]`
3. Bob now plays optimally from this state, which corresponds to our `dfs(2)` calculation
4. Bob would take all remaining stones, scoring 15
5. Score difference = 10 - 15 = -5

This confirms why Alice chooses not to take at position 1 (would result in -5 instead of 15).

## Python Solution

```python
from typing import List
from functools import cache
from itertools import accumulate

class Solution:
    def stoneGameVIII(self, stones: List[int]) -> int:
        # Calculate prefix sums for quick range sum calculation
        # prefix_sums[i] represents sum of stones[0] to stones[i]
        prefix_sums = list(accumulate(stones))
      
        @cache
        def calculate_max_score_difference(current_index: int) -> int:
            """
            Calculate the maximum score difference the current player can achieve
            starting from current_index.
          
            Args:
                current_index: The index from which the current player can choose
          
            Returns:
                Maximum score difference (current player score - opponent score)
            """
            # Base case: if we're at or past the second-to-last stone,
            # the only option is to take all remaining stones
            if current_index >= len(stones) - 1:
                return prefix_sums[-1]
          
            # The current player has two choices:
            # Option 1: Skip this position and let the decision move to next index
            skip_current = calculate_max_score_difference(current_index + 1)
          
            # Option 2: Take stones from 0 to current_index (score = prefix_sums[current_index])
            # Then the opponent plays optimally from current_index + 1
            # Score difference = our score - opponent's best score
            take_current = prefix_sums[current_index] - calculate_max_score_difference(current_index + 1)
          
            # Return the maximum score difference we can achieve
            return max(skip_current, take_current)
      
        # Start from index 1 (at least 2 stones must be taken initially)
        return calculate_max_score_difference(1)
```

## Time and Space Complexity

The time complexity is `O(n)`, where `n` is the length of the array `stones`. The memoized recursive function `dfs` computes the result for each index from 1 to n-1 exactly once due to the `@cache` decorator. Each recursive call either returns immediately (base case) or makes one recursive call and performs `O(1)` operations. Since there are `n-1` possible states (indices 1 through n-1) and each state is computed once, the total time complexity is `O(n)`.

The space complexity is `O(n)`. This consists of:
- `O(n)` for the prefix sum array `s` created by `list(accumulate(stones))`
- `O(n)` for the memoization cache storing up to `n-1` computed values
- `O(n)` for the recursion call stack in the worst case

Therefore, the overall space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Starting from Wrong Index**

**Pitfall**: A common mistake is calling `dfs(0)` instead of `dfs(1)`, which would incorrectly allow taking just one stone on the first move.

**Why it's wrong**: The game rules explicitly state that players must choose `x > 1`, meaning at least 2 stones must be taken. Starting from index 0 would allow taking just `stones[0]`, violating this constraint.

**Solution**: Always start the recursion from `dfs(1)` since the minimum valid move is taking stones at indices 0 and 1.

```python
# Wrong
return calculate_max_score_difference(0)

# Correct
return calculate_max_score_difference(1)
```

### 2. **Incorrect Base Case Boundary**

**Pitfall**: Using `i == len(stones) - 1` instead of `i >= len(stones) - 1` in the base case.

**Why it's wrong**: During recursion, we might call `dfs(i + 1)` when `i` is already at `len(stones) - 1`, resulting in `dfs(len(stones))`. Without the `>=` check, this would bypass the base case and cause index out of bounds errors or incorrect results.

**Solution**: Use `>=` to handle all cases where no meaningful choice remains:

```python
# Wrong
if current_index == len(stones) - 1:
    return prefix_sums[-1]

# Correct
if current_index >= len(stones) - 1:
    return prefix_sums[-1]
```

### 3. **Misunderstanding Score Calculation**

**Pitfall**: Confusing what `prefix_sums[i]` represents and incorrectly calculating the score difference.

**Why it's wrong**: Some might think they need to calculate `prefix_sums[i] - prefix_sums[start]` for a range, but since we're always taking from the leftmost position (index 0), the score is simply `prefix_sums[i]`.

**Solution**: Remember that after each move, the stones are "merged" at position 0, so the next player always takes from index 0. The score for taking up to index `i` is exactly `prefix_sums[i]`.

### 4. **Forgetting Memoization**

**Pitfall**: Implementing the recursive solution without memoization.

**Why it's wrong**: Without memoization, the same states are recalculated multiple times, leading to exponential time complexity O(2^n) instead of linear O(n).

**Solution**: Always use `@cache` or implement manual memoization:

```python
# Without memoization - TLE for large inputs
def dfs(i: int) -> int:
    if i >= len(stones) - 1:
        return prefix_sums[-1]
    return max(dfs(i + 1), prefix_sums[i] - dfs(i + 1))

# With memoization - O(n) time
@cache
def dfs(i: int) -> int:
    if i >= len(stones) - 1:
        return prefix_sums[-1]
    return max(dfs(i + 1), prefix_sums[i] - dfs(i + 1))
```

### 5. **Confusion About Player Perspective**

**Pitfall**: Getting confused about whose perspective the score difference represents at each recursive call.

**Why it's wrong**: In game theory problems, it's crucial to maintain consistency about which player's advantage we're calculating. Mixing perspectives leads to incorrect signs in the score calculation.

**Solution**: Always think of `dfs(i)` as "the maximum advantage the current player can get over their opponent." When the current player scores `prefix_sums[i]` and passes to the opponent, the net advantage is `prefix_sums[i] - dfs(i + 1)` (my score minus opponent's best advantage from that point).

