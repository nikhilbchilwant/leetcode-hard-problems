# 1563. Stone Game V

> Source: https://algo.monster/liteproblems/1563

## Problem Statement

You are given an array `stoneValue` where each element represents the value of a stone arranged in a row. Alice and Bob play a game with these stones.

The game proceeds in rounds:
1. **Alice's turn**: She divides the current row of stones into two non-empty parts (a left row and a right row)
2. **Bob's turn**: He calculates the sum of values in each row, then discards the row with the maximum sum
3. **Alice's score**: The sum of the remaining row is added to Alice's score
4. **Special case**: If both rows have equal sums, Bob lets Alice choose which row to discard

The game continues with the remaining row until only one stone is left (at which point no more divisions are possible). Alice starts with a score of 0.

Your task is to find the maximum score Alice can achieve if she plays optimally.

**Example walkthrough:**
If `stoneValue = [6, 2, 3, 4, 5, 5]`:
- Alice could split into `[6, 2, 3]` (sum = 11) and `[4, 5, 5]` (sum = 14)
- Bob discards `[4, 5, 5]` since 14 > 11
- Alice gets 11 points and continues with `[6, 2, 3]`
- The process repeats until one stone remains

The goal is to determine Alice's optimal splitting strategy to maximize her total score across all rounds.

## Intuition

The key insight is that this is a [dynamic programming](/problems/dynamic_programming_intro) problem where Alice wants to maximize her score by choosing optimal split points. Since Alice controls how to split the stones and wants to maximize her score, she needs to consider all possible ways to divide the current row.

For any subarray of stones, Alice needs to try every possible split point. When she splits at position `k`, she creates two parts: left `[i, k]` and right `[k+1, j]`. Bob will discard the part with the larger sum, so Alice gets the smaller sum added to her score. Then the game continues recursively with the remaining part.

The recursive nature becomes clear: if Alice keeps the left part (because it has smaller or equal sum), she gets `left_sum + dfs(i, k)` total score. If she keeps the right part, she gets `right_sum + dfs(k+1, j)` total score. When both parts have equal sums, Alice can choose which one to keep, so she picks the option that gives her the maximum future score.

To optimize this recursive approach, we can use memoization to avoid recalculating the same subproblems. The state is defined by the boundaries `(i, j)` of the current subarray.

The pruning optimization comes from recognizing that if we've already found a good score `ans`, and the current split can at most give us `2 * smaller_sum` (the smaller sum now plus at most the same amount in future rounds), we can skip this split if `ans >= 2 * smaller_sum`. This is because even in the best case, this split won't improve our answer.

Using prefix sums `s` allows us to quickly calculate the sum of any subarray: `sum[i to j] = s[j+1] - s[i]`, which makes the implementation more efficient.

**Learn more about [Math](/problems/math-basics) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution uses [dynamic programming](/problems/dynamic_programming_intro) with memoization and pruning optimizations. Here's how the implementation works:

**1. Prefix Sum Preprocessing:**
First, we build a prefix sum array `s` where `s[i]` represents the sum of the first `i` elements of `stoneValue`. This allows us to calculate the sum of any subarray in O(1) time: `sum[i to j] = s[j+1] - s[i]`.

**2. Recursive Function with Memoization:**
We define `dfs(i, j)` to represent the maximum score Alice can obtain from stones in the range `[i, j]`. The `@cache` decorator provides automatic memoization to avoid redundant calculations.

**3. Base Case:**
If `i >= j`, there's only one stone (or no stones) left, so Alice can't make any splits and returns 0.

**4. Enumeration of Split Points:**
For each possible split point `k` where `i ≤ k < j`, we:
- Calculate `l` (left sum) incrementally by adding `stoneValue[k]`
- Calculate `r` (right sum) by subtracting from the total sum of the range

**5. Decision Making Based on Sums:**
- **If `l < r`:** Bob discards the right part, Alice keeps the left part
  - Score = `l + dfs(i, k)`
  - Pruning: Skip if `ans >= l * 2` (current answer already better than best possible)

- **If `l > r`:** Bob discards the left part, Alice keeps the right part
  - Score = `r + dfs(k+1, j)`
  - Pruning: Break the loop if `ans >= r * 2` (no future splits can improve)

- **If `l == r`:** Alice chooses which part to keep
  - Score = `max(l + dfs(i, k), r + dfs(k+1, j))`

**6. Pruning Logic:**
The pruning works because:
- When keeping a part with sum `x`, the maximum possible score is `x` (current) + `x` (best case future) = `2x`
- If our current best `ans >= 2x`, this split can't improve our answer
- For the `l > r` case, we can break early since as `k` increases, `r` only gets smaller

**7. Final Answer:**
The function returns `dfs(0, len(stoneValue) - 1)`, which represents the maximum score for the entire array.

The time complexity is O(n³) with memoization (n² states, each taking O(n) to compute), and space complexity is O(n²) for the memoization cache.

## Example Walkthrough

Let's walk through a small example with `stoneValue = [7, 3, 4, 5]` to illustrate how the solution works.

**Initial Setup:**
- Prefix sum array: `s = [0, 7, 10, 14, 19]`
- We call `dfs(0, 3)` to find the maximum score for the entire array

**Round 1: `dfs(0, 3)` - stones [7, 3, 4, 5]**

We try each possible split point:

*Split at k=0:* [7] | [3, 4, 5]
- Left sum `l = 7`, Right sum `r = 12`
- Since `l < r`, Bob discards right part, Alice keeps left
- Score = 7 + `dfs(0, 0)` = 7 + 0 = 7
- Update `ans = 7`

*Split at k=1:* [7, 3] | [4, 5]
- Left sum `l = 10`, Right sum `r = 9`
- Since `l > r`, Bob discards left part, Alice keeps right
- Score = 9 + `dfs(2, 3)` = 9 + ? (need to compute)
- First, check pruning: Since `ans = 7` and `2 * r = 18`, no pruning

**Compute `dfs(2, 3)` - stones [4, 5]:**
- Only one split possible at k=2: [4] | [5]
- Left sum = 4, Right sum = 5
- Since 4 < 5, Alice keeps left part
- Score = 4 + `dfs(2, 2)` = 4 + 0 = 4
- Return 4

Back to *Split at k=1:*
- Score = 9 + 4 = 13
- Update `ans = 13`

*Split at k=2:* [7, 3, 4] | [5]
- Left sum `l = 14`, Right sum `r = 5`
- Since `l > r`, Bob discards left part, Alice keeps right
- Score = 5 + `dfs(3, 3)` = 5 + 0 = 5
- `ans` remains 13 (no improvement)
- Pruning check: Since `ans = 13 >= 2 * 5 = 10`, we could break here

**Final Result:** `dfs(0, 3)` returns 13

**Optimal Strategy:**
Alice's optimal play is to split [7, 3] | [4, 5] in the first round:
- Bob discards [7, 3] (sum = 10)
- Alice scores 9 points and continues with [4, 5]
- Next, she splits [4] | [5]
- Bob discards [5]
- Alice scores 4 more points
- Total score: 9 + 4 = 13

The memoization ensures that if we encounter the same subarray again (e.g., `dfs(2, 3)`), we don't recalculate it. The pruning optimization helps skip splits that can't possibly improve our current best answer, making the algorithm more efficient.

## Python Solution

```python
from typing import List
from functools import cache
from itertools import accumulate


class Solution:
    def stoneGameV(self, stoneValue: List[int]) -> int:
        """
        Stone Game V: Alice and Bob play a game with stones in a row.
        Each turn, a player divides stones into two non-empty groups.
        The player with smaller sum gets points equal to that sum.
        If sums are equal, player chooses which sum to take as points.
        Returns maximum points Alice can get with optimal play.
        """
      
        @cache
        def dp(left: int, right: int) -> int:
            """
            Dynamic programming function to find maximum score in range [left, right].
          
            Args:
                left: Starting index of the range (inclusive)
                right: Ending index of the range (inclusive)
          
            Returns:
                Maximum score achievable from stones in range [left, right]
            """
            # Base case: if range is invalid or contains single stone, no score
            if left >= right:
                return 0
          
            max_score = 0
            left_sum = 0
            # Calculate right sum using prefix sum array
            right_sum = prefix_sum[right + 1] - prefix_sum[left]
          
            # Try all possible split points
            for split_point in range(left, right):
                # Update sums after including current stone in left group
                left_sum += stoneValue[split_point]
                right_sum -= stoneValue[split_point]
              
                if left_sum < right_sum:
                    # Left group has smaller sum, Alice gets left_sum points
                    # Pruning: skip if current max_score is already >= 2 * left_sum
                    if max_score >= left_sum * 2:
                        continue
                    max_score = max(max_score, left_sum + dp(left, split_point))
                  
                elif left_sum > right_sum:
                    # Right group has smaller sum, Alice gets right_sum points
                    # Pruning: break early if max_score is already >= 2 * right_sum
                    if max_score >= right_sum * 2:
                        break
                    max_score = max(max_score, right_sum + dp(split_point + 1, right))
                  
                else:
                    # Both groups have equal sum, Alice chooses the better option
                    max_score = max(
                        max_score,
                        max(left_sum + dp(left, split_point), 
                            right_sum + dp(split_point + 1, right))
                    )
          
            return max_score
      
        # Build prefix sum array for efficient range sum queries
        # prefix_sum[i] = sum of stoneValue[0:i]
        prefix_sum = list(accumulate(stoneValue, initial=0))
      
        # Start the recursive solution from the entire array
        return dp(0, len(stoneValue) - 1)
```

## Time and Space Complexity

**Time Complexity:** `O(n³)`

The function uses dynamic programming with memoization through the `@cache` decorator. The `dfs` function explores all possible split points for subarrays defined by indices `(i, j)`.

- There are `O(n²)` possible unique states `(i, j)` where `0 ≤ i < j < n`, representing all possible subarrays.
- Due to memoization, each state is computed only once.
- For each state `(i, j)`, the function iterates through all possible split points `k` from `i` to `j-1`, which takes `O(j - i) = O(n)` time in the worst case.
- Within each iteration, operations like updating `l` and `r`, comparisons, and recursive calls (which return cached results) take `O(1)` time.

Therefore, the total time complexity is `O(n²) × O(n) = O(n³)`.

**Space Complexity:** `O(n²)`

The space complexity consists of:
- The memoization cache stores results for all `O(n²)` possible states `(i, j)`.
- The recursion stack depth is at most `O(n)` in the worst case when the array is split into increasingly smaller subarrays.
- The prefix sum array `s` takes `O(n)` space.

Since `O(n²)` dominates, the overall space complexity is `O(n²)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Sum Calculation Without Prefix Sum**
A common mistake is recalculating the sum of subarrays repeatedly within the recursive function, leading to O(n⁴) time complexity instead of O(n³).

**Pitfall Example:**
```python
def dp(left, right):
    # Wrong: Recalculating sum every time
    for split_point in range(left, right):
        left_sum = sum(stoneValue[left:split_point+1])  # O(n) operation
        right_sum = sum(stoneValue[split_point+1:right+1])  # O(n) operation
```

**Solution:**
Use prefix sums for O(1) range sum queries, or maintain running sums incrementally as shown in the correct implementation.

### 2. **Missing or Incorrect Pruning Logic**
Without proper pruning, the solution may time out on larger inputs. The key insight is that if we already have a score `ans`, and the current partition gives us sum `x`, the maximum we can possibly achieve is `x + x = 2x` (current score + best possible future score).

**Pitfall Example:**
```python
# Missing pruning entirely
for split_point in range(left, right):
    if left_sum < right_sum:
        max_score = max(max_score, left_sum + dp(left, split_point))
    # No early termination or skip conditions
```

**Solution:**
- When `left_sum < right_sum`: Skip if `max_score >= 2 * left_sum`
- When `left_sum > right_sum`: Break early if `max_score >= 2 * right_sum` (since right_sum only decreases)

### 3. **Confusion About Index Boundaries**
The recursive function uses inclusive boundaries `[left, right]`, but it's easy to mix this up with exclusive boundaries or off-by-one errors.

**Pitfall Example:**
```python
# Wrong: Treating right as exclusive
def dp(left, right):
    if left >= right - 1:  # Wrong base case
        return 0
    for split_point in range(left, right - 1):  # Wrong range
```

**Solution:**
Be consistent with inclusive boundaries:
- Base case: `if left >= right` (single stone or invalid range)
- Split range: `for split_point in range(left, right)`
- Recursive calls: `dp(left, split_point)` and `dp(split_point + 1, right)`

### 4. **Misunderstanding the Equal Sum Case**
When both partitions have equal sums, Alice gets to choose which partition to keep. A common mistake is to always pick one side or to add both scores.

**Pitfall Example:**
```python
if left_sum == right_sum:
    # Wrong: Adding both scores
    max_score = max(max_score, left_sum + dp(left, split_point) + 
                              right_sum + dp(split_point + 1, right))
  
    # Wrong: Always choosing left
    max_score = max(max_score, left_sum + dp(left, split_point))
```

**Solution:**
Alice chooses the partition that maximizes her total score:
```python
max_score = max(max_score, 
                max(left_sum + dp(left, split_point),
                    right_sum + dp(split_point + 1, right)))
```

