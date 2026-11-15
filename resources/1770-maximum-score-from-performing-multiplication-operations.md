# 1770. Maximum Score from Performing Multiplication Operations

> Source: https://algo.monster/liteproblems/1770

## Problem Statement

You have two arrays: `nums` and `multipliers`. The `nums` array has `n` elements, and the `multipliers` array has `m` elements, where `n >= m`.

Starting with a score of 0, you need to perform exactly `m` operations. For each operation `i` (where `i` goes from 0 to m-1):

1. **Pick an element** from either the beginning or the end of the `nums` array. Let's call this element `x`.

2. **Calculate points** by multiplying the chosen element `x` with `multipliers[i]` and add this product to your total score.

3. **Remove** the chosen element `x` from the `nums` array (so the array shrinks by one element after each operation).

The goal is to find the maximum possible score you can achieve after completing all `m` operations.

**Example walkthrough:**
- If `nums = [1, 2, 3]` and `multipliers = [3, 2]`
- You need to perform 2 operations
- Operation 1: You could pick 1 (from start) or 3 (from end), multiply by `multipliers[0] = 3`
- Operation 2: After removing your first choice, pick from the remaining elements at either end, multiply by `multipliers[1] = 2`
- Different choices lead to different total scores, and you want the maximum possible score

The challenge is determining which elements to pick from which end of the array and in what order to maximize the final score.

## Intuition

When we look at this problem, we need to make a series of choices: at each step, should we pick from the left end or the right end of the array? This creates a decision tree where each path represents a different sequence of choices.

The key insight is that we can think of this problem in terms of **states**. At any point during our operations, we can describe our current situation using just a few pieces of information:
- How many elements we've taken from the left side
- How many elements we've taken from the right side  
- Which operation we're currently on

But wait - if we know how many elements we've taken from the left (`i` elements) and which operation number we're on (`k`), we can figure out how many we've taken from the right (it would be `k - i` elements). So we really only need to track the left pointer and the operation number.

This observation leads us to **[dynamic programming](/problems/dynamic_programming_intro)**. For any given state (left index, right index, operation number), we have two choices:
1. Take from the left: multiply `nums[left]` by `multipliers[operation]` and move to the next state
2. Take from the right: multiply `nums[right]` by `multipliers[operation]` and move to the next state

We want to choose whichever option gives us the maximum score.

The recursive nature becomes clear: the best score from any state equals the current operation's score plus the best score from the remaining operations. By using memoization (the `@cache` decorator), we avoid recalculating the same states multiple times.

The function `f(i, j, k)` represents: "What's the maximum score we can get starting from index `i` on the left, index `j` on the right, and operation `k`?" We recursively try both options (taking from left or right) and return the maximum.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution implements a top-down [dynamic programming](/problems/dynamic_programming_intro) approach using memoization. Let's walk through the implementation step by step:

**Function Definition:**
```python
@cache
def f(i, j, k):
```
The function `f(i, j, k)` represents the maximum score we can achieve where:
- `i` is the current left index of the `nums` array
- `j` is the current right index of the `nums` array  
- `k` is the current operation number (index in `multipliers`)

The `@cache` decorator automatically memoizes the results, preventing redundant calculations for the same state.

**Base Case:**
```python
if k >= m or i >= n or j < 0:
    return 0
```
We return 0 when:
- We've completed all `m` operations (`k >= m`)
- The left pointer goes out of bounds (`i >= n`)
- The right pointer goes out of bounds (`j < 0`)

**Recursive Cases:**
```python
a = f(i + 1, j, k + 1) + nums[i] * multipliers[k]
b = f(i, j - 1, k + 1) + nums[j] * multipliers[k]
return max(a, b)
```

For each state, we calculate two options:
1. **Take from left** (`a`): Add `nums[i] * multipliers[k]` to the score from the remaining subproblem `f(i + 1, j, k + 1)`
2. **Take from right** (`b`): Add `nums[j] * multipliers[k]` to the score from the remaining subproblem `f(i, j - 1, k + 1)`

We return the maximum of these two choices.

**Initial Call:**
```python
n = len(nums)
m = len(multipliers)
return f(0, n - 1, 0)
```
We start with:
- Left pointer at index 0
- Right pointer at index `n - 1` 
- Operation number 0

The algorithm explores all possible combinations of taking elements from either end, but memoization ensures each unique state is computed only once. The time complexity is `O(m^2)` since there are at most `m` possible values for the left pointer (we can take at most `m` elements from the left), `m` possible values for the right pointer, and `m` operations. However, since the number of elements taken from left and right must sum to the operation number, the actual number of valid states is `O(m^2)`.

## Example Walkthrough

Let's work through a concrete example with `nums = [3, 1, 5, 2]` and `multipliers = [2, 4, 3]`.

We need to perform 3 operations total. Let's trace through the recursive solution:

**Initial State:** `f(0, 3, 0)` - left pointer at index 0, right pointer at index 3, operation 0

At this state, we have two choices:
- Take from left: `nums[0] * multipliers[0] = 3 * 2 = 6`, then solve `f(1, 3, 1)`
- Take from right: `nums[3] * multipliers[0] = 2 * 2 = 4`, then solve `f(0, 2, 1)`

Let's explore the left branch first:

**Path 1: Take left (3)**
State: `f(1, 3, 1)` - remaining array looks like `[1, 5, 2]` from indices 1 to 3
- Take from left: `nums[1] * multipliers[1] = 1 * 4 = 4`, then solve `f(2, 3, 2)`
- Take from right: `nums[3] * multipliers[1] = 2 * 4 = 8`, then solve `f(1, 2, 2)`

Following the better option (take right with value 8):
State: `f(1, 2, 2)` - remaining array looks like `[1, 5]` from indices 1 to 2
- Take from left: `nums[1] * multipliers[2] = 1 * 3 = 3`, then solve `f(2, 2, 3)`
- Take from right: `nums[2] * multipliers[2] = 5 * 3 = 15`, then solve `f(1, 1, 3)`

Taking right (better option with 15):
State: `f(1, 1, 3)` - base case reached (k >= m), returns 0

So this path gives: `6 + 8 + 15 = 29`

**Path 2: Take right (2) initially**
State: `f(0, 2, 1)` - remaining array looks like `[3, 1, 5]` from indices 0 to 2
- Take from left: `nums[0] * multipliers[1] = 3 * 4 = 12`, then solve `f(1, 2, 2)`
- Take from right: `nums[2] * multipliers[1] = 5 * 4 = 20`, then solve `f(0, 1, 2)`

Following the better option (take right with value 20):
State: `f(0, 1, 2)` - remaining array looks like `[3, 1]` from indices 0 to 1
- Take from left: `nums[0] * multipliers[2] = 3 * 3 = 9`, then solve `f(1, 1, 3)`
- Take from right: `nums[1] * multipliers[2] = 1 * 3 = 3`, then solve `f(0, 0, 3)`

Taking left (better option with 9):
State: `f(1, 1, 3)` - base case reached, returns 0

This path gives: `4 + 20 + 9 = 33`

The algorithm explores all paths (with memoization preventing recalculation) and returns the maximum score of **33**, achieved by taking elements in the order: right (2), right (5), left (3).

## Python Solution

```python
class Solution:
    def maximumScore(self, nums: List[int], multipliers: List[int]) -> int:
        """
        Calculate maximum score by selecting elements from start or end of nums array
        and multiplying with multipliers in order.
      
        Args:
            nums: Array of integers to select from
            multipliers: Array of multipliers to apply in order
          
        Returns:
            Maximum possible score
        """
        from functools import cache
      
        @cache
        def dp(left_index: int, right_index: int, mult_index: int) -> int:
            """
            Dynamic programming function to find maximum score.
          
            Args:
                left_index: Current left pointer in nums array
                right_index: Current right pointer in nums array  
                mult_index: Current index in multipliers array
              
            Returns:
                Maximum score achievable from current state
            """
            # Base case: all multipliers have been used
            if mult_index >= num_multipliers:
                return 0
          
            # Invalid state check (shouldn't occur with correct logic)
            if left_index >= nums_length or right_index < 0:
                return 0
          
            # Option 1: Take element from left side
            take_left = dp(left_index + 1, right_index, mult_index + 1) + nums[left_index] * multipliers[mult_index]
          
            # Option 2: Take element from right side
            take_right = dp(left_index, right_index - 1, mult_index + 1) + nums[right_index] * multipliers[mult_index]
          
            # Return maximum of both options
            return max(take_left, take_right)
      
        # Store array lengths for easier access
        nums_length = len(nums)
        num_multipliers = len(multipliers)
      
        # Start recursion with full nums array and first multiplier
        return dp(0, nums_length - 1, 0)
```

## Time and Space Complexity

**Time Complexity:** `O(m^2)` where `m` is the length of the multipliers array.

The recursive function `f(i, j, k)` uses three parameters, but there's a key relationship: at any step `k`, we've taken exactly `k` elements total from either the left (tracked by `i`) or right (tracked by `j`) ends. This means `i + (n - 1 - j) = k`, which reduces the state space from three dimensions to two. 

The memoization with `@cache` ensures each unique state `(i, j, k)` is computed only once. Since `k` ranges from `0` to `m-1`, and for each `k`, the value of `i` can range from `0` to `k` (as we can take at most `k` elements from the left), there are approximately `O(m^2)` unique states to compute. Each state computation takes `O(1)` time.

**Space Complexity:** `O(m^2)`

The space is used for:
1. **Memoization cache:** Stores up to `O(m^2)` unique states as analyzed above
2. **Recursion call stack:** Maximum depth is `m`, contributing `O(m)` space

The dominant factor is the memoization cache, giving us `O(m^2)` total space complexity.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. Memory Limit Exceeded (MLE) - Using 3D DP State

**The Pitfall:**
Many developers initially try to track all three parameters (left_index, right_index, mult_index) independently in their DP state. However, this creates unnecessary redundancy because these three values are actually interdependent. If you know how many elements were taken from the left and which operation you're on, you can calculate how many were taken from the right.

Using all three parameters leads to:
- Excessive memory usage: O(n × n × m) space in worst case
- Many invalid states being cached (where left_index + (n - 1 - right_index) ≠ mult_index)
- Potential Memory Limit Exceeded errors on large inputs

**The Solution:**
Optimize the state representation by using only two parameters. Since we know:
- Total operations performed = mult_index
- Elements taken from left = left_index
- Elements taken from right = mult_index - left_index

We can reconstruct the right pointer: `right_index = n - 1 - (mult_index - left_index)`

```python
class Solution:
    def maximumScore(self, nums: List[int], multipliers: List[int]) -> int:
        from functools import cache
      
        n = len(nums)
        m = len(multipliers)
      
        @cache
        def dp(left: int, mult_idx: int) -> int:
            # Base case: all multipliers used
            if mult_idx >= m:
                return 0
          
            # Calculate right pointer from left and mult_idx
            right = n - 1 - (mult_idx - left)
          
            # Take from left
            take_left = nums[left] * multipliers[mult_idx] + dp(left + 1, mult_idx + 1)
          
            # Take from right  
            take_right = nums[right] * multipliers[mult_idx] + dp(left, mult_idx + 1)
          
            return max(take_left, take_right)
      
        return dp(0, 0)
```

This optimization reduces:
- Space complexity from O(n × n × m) to O(m × m)
- Number of states from potentially n² × m to just m²
- Risk of MLE on large inputs where n is significantly larger than m

### 2. Integer Overflow Concerns

**The Pitfall:**
When nums and multipliers contain large values, their products and accumulated sums might exceed standard integer limits in some languages (though Python handles arbitrary precision integers automatically).

**The Solution:**
While Python doesn't have this issue, be aware when implementing in other languages. Use appropriate data types (long long in C++, long in Java) to handle the range of possible values according to problem constraints.

