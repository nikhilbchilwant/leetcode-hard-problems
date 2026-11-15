# 2547. Minimum Cost to Split an Array

> Source: https://algo.monster/liteproblems/2547

## Problem Statement

You have an integer array `nums` and an integer `k`. Your task is to split this array into some number of non-empty subarrays and find the minimum possible total cost of the split.

To calculate the cost of a split, you need to understand two key concepts:

**Trimmed Subarray**: When you trim a subarray, you remove all numbers that appear only once in that subarray. For example:
- `trimmed([3,1,2,4,3,4]) = [3,4,3,4]` (numbers 1 and 2 appear only once, so they're removed)
- `trimmed([1,2,3,3,3,4,4]) = [3,3,3,4,4]` (numbers 1 and 2 appear only once, so they're removed)

**Importance Value**: The importance value of a subarray is calculated as `k + length of trimmed subarray`. For instance:
- If a subarray is `[1,2,3,3,3,4,4]`, its trimmed version is `[3,3,3,4,4]` with length 5
- The importance value would be `k + 5`

**Total Cost**: The total cost of a split is the sum of importance values of all subarrays in that split.

Your goal is to find a way to split the array `nums` into subarrays such that the total cost is minimized. Each subarray must be contiguous (elements next to each other in the original array) and non-empty.

For example, if you have `nums = [1,2,3,4]` and `k = 2`, you could:
- Keep it as one subarray `[1,2,3,4]` where all elements appear once, so trimmed is `[]`, giving importance value `2 + 0 = 2`
- Or split it into `[1,2]` and `[3,4]`, each having importance value `2 + 0 = 2`, for total cost `4`
- Or other combinations

The challenge is finding the optimal split that gives the minimum total cost.

## Intuition

The key insight is that this is an optimal subproblem structure where we need to make decisions about where to split the array. At each position, we face a choice: where should the current subarray end?

Think of it this way: if we're standing at index `i`, we need to decide where to make the next cut. We could cut immediately after `i`, or after `i+1`, or after `i+2`, and so on. Each choice creates a subarray from `i` to some position `j`, and then we need to solve the same problem for the remaining array starting from `j+1`.

This naturally leads to a recursive approach: `dfs(i)` represents "what's the minimum cost to split the array starting from index `i`?" For each starting position `i`, we try all possible ending positions `j` and calculate:
- The cost of the subarray from `i` to `j`
- Plus the minimum cost to handle the rest of the array from `j+1`

The tricky part is calculating the importance value efficiently. Instead of actually creating the trimmed array, we can be clever: the length of the trimmed array equals the total length minus the count of numbers that appear only once. So we track:
- `cnt`: frequency of each number in the current subarray
- `one`: count of numbers that appear exactly once

As we extend the subarray by including `nums[j]`:
- If `cnt[nums[j]]` becomes 1, we increment `one` (a new unique number)
- If `cnt[nums[j]]` becomes 2, we decrement `one` (a number is no longer unique)

The importance value is then `k + (j - i + 1) - one`, where `(j - i + 1)` is the subarray length and `one` is the count of unique numbers.

Since we might recalculate `dfs(i)` for the same `i` multiple times through different splitting paths, we use memoization to cache results and avoid redundant calculations.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution uses [dynamic programming](/problems/dynamic_programming_intro) with memoization to find the minimum cost. Here's how the implementation works:

**Main Function Structure:**
We define a recursive function `dfs(i)` that calculates the minimum cost to split the array starting from index `i`. The answer to our problem is `dfs(0)`.

**Base Case:**
When `i >= n` (reached or passed the end of array), we return `0` since there's nothing left to split.

**Recursive Case:**
For each starting position `i`, we try all possible ending positions `j` from `i` to `n-1`:

1. **Initialize tracking variables:**
   - `cnt = Counter()`: A hash table to track frequency of each number in current subarray
   - `one = 0`: Count of numbers appearing exactly once
   - `ans = inf`: Store the minimum cost found

2. **Extend the subarray:** For each `j` from `i` to `n-1`:
   - Add `nums[j]` to our frequency counter: `cnt[nums[j]] += 1`
   - Update the `one` counter:
     - If `cnt[nums[j]] == 1`: A new unique number, so `one += 1`
     - If `cnt[nums[j]] == 2`: A number is no longer unique, so `one -= 1`
 
3. **Calculate cost for this split:**
   - Length of current subarray: `j - i + 1`
   - Length of trimmed subarray: `(j - i + 1) - one`
   - Importance value: `k + (j - i + 1) - one`
   - Total cost: `k + j - i + 1 - one + dfs(j + 1)`

4. **Track minimum:** Update `ans = min(ans, current_cost)`

**Memoization:**
The `@cache` decorator automatically memoizes the results of `dfs(i)`. This prevents recalculating the same subproblem multiple times, reducing time complexity from exponential to `O(n²)`.

**Time Complexity:** `O(n²)` - We have `n` states for `dfs(i)`, and for each state, we iterate through at most `n` positions.

**Space Complexity:** `O(n)` - For the recursion stack and memoization cache.

## Example Walkthrough

Let's walk through the solution with `nums = [1,2,1,2,1]` and `k = 2`.

We start with `dfs(0)`, which needs to find the minimum cost to split the entire array starting from index 0.

**From index 0, we try different ending positions:**

**Option 1: Subarray [0,0] = [1]**
- `cnt = {1: 1}`, `one = 1` (number 1 appears once)
- Trimmed length = 1 - 1 = 0
- Importance = k + 0 = 2 + 0 = 2
- Total cost = 2 + `dfs(1)`

**Option 2: Subarray [0,1] = [1,2]**
- Start with `cnt = {1: 1}`, `one = 1`
- Add nums[1]=2: `cnt = {1: 1, 2: 1}`, `one = 2` (both appear once)
- Trimmed length = 2 - 2 = 0
- Importance = k + 0 = 2 + 0 = 2
- Total cost = 2 + `dfs(2)`

**Option 3: Subarray [0,2] = [1,2,1]**
- Start with `cnt = {1: 1, 2: 1}`, `one = 2`
- Add nums[2]=1: `cnt = {1: 2, 2: 1}`, `one = 1` (only 2 appears once now)
- Trimmed length = 3 - 1 = 2
- Importance = k + 2 = 2 + 2 = 4
- Total cost = 4 + `dfs(3)`

**Option 4: Subarray [0,3] = [1,2,1,2]**
- Continue from previous: add nums[3]=2
- `cnt = {1: 2, 2: 2}`, `one = 0` (no numbers appear once)
- Trimmed length = 4 - 0 = 4
- Importance = k + 4 = 2 + 4 = 6
- Total cost = 6 + `dfs(4)`

**Option 5: Subarray [0,4] = [1,2,1,2,1]**
- Continue: add nums[4]=1
- `cnt = {1: 3, 2: 2}`, `one = 0` (still no numbers appear once)
- Trimmed length = 5 - 0 = 5
- Importance = k + 5 = 2 + 5 = 7
- Total cost = 7 + `dfs(5)` = 7 + 0 = 7

Now let's evaluate `dfs(2)` (needed for Option 2):
- Subarray [2,2] = [1]: importance = 2, total = 2 + `dfs(3)`
- Subarray [2,3] = [1,2]: importance = 2, total = 2 + `dfs(4)`
- Subarray [2,4] = [1,2,1]: importance = 4, total = 4 + 0 = 4

And `dfs(3)`:
- Subarray [3,3] = [2]: importance = 2, total = 2 + `dfs(4)`
- Subarray [3,4] = [2,1]: importance = 2, total = 2 + 0 = 2

And `dfs(4)`:
- Subarray [4,4] = [1]: importance = 2, total = 2 + 0 = 2

Working backwards with memoization:
- `dfs(4)` = 2
- `dfs(3)` = min(2 + 2, 2) = 2
- `dfs(2)` = min(2 + 2, 2 + 2, 4) = 4
- `dfs(1)` = min(2 + 4, 2 + 2, 4 + 0, 5) = 4
- `dfs(0)` = min(2 + 4, 2 + 4, 4 + 2, 6 + 2, 7) = 6

The minimum cost is 6, achieved by splitting as [1] [2,1,2,1] or [1,2] [1,2,1].

## Python Solution

```python
class Solution:
    def minCost(self, nums: List[int], k: int) -> int:
        from functools import cache
        from collections import Counter
        from typing import List
      
        @cache
        def dp(start_idx):
            """
            Dynamic programming function to find minimum cost from index start_idx to end.
          
            Args:
                start_idx: Starting index of the current subarray
              
            Returns:
                Minimum cost to partition from start_idx to end of array
            """
            # Base case: if we've processed all elements
            if start_idx >= n:
                return 0
          
            # Track frequency of elements in current subarray
            frequency_map = Counter()
            # Count of elements that appear exactly once
            unique_count = 0
            # Initialize minimum cost to infinity
            min_cost = float('inf')
          
            # Try all possible end positions for current subarray
            for end_idx in range(start_idx, n):
                current_element = nums[end_idx]
                frequency_map[current_element] += 1
              
                # Update count of unique elements
                if frequency_map[current_element] == 1:
                    # Element appears for first time
                    unique_count += 1
                elif frequency_map[current_element] == 2:
                    # Element now appears twice, no longer unique
                    unique_count -= 1
              
                # Calculate cost for current subarray:
                # k (constant cost) + subarray_length - unique_elements + cost of remaining array
                subarray_length = end_idx - start_idx + 1
                importance_score = subarray_length - unique_count
                current_cost = k + importance_score + dp(end_idx + 1)
              
                # Update minimum cost
                min_cost = min(min_cost, current_cost)
          
            return min_cost
      
        # Store array length for easier access
        n = len(nums)
      
        # Start dynamic programming from index 0
        return dp(0)
```

## Time and Space Complexity

**Time Complexity:** `O(n^2)`

The algorithm uses dynamic programming with memoization. The `dfs(i)` function is called for each possible starting position from `0` to `n-1`, resulting in at most `n` unique states. For each state `dfs(i)`, the function iterates through all positions from `i` to `n-1`, performing `O(n-i)` operations. In the worst case, this gives us:
- State 0: iterates through positions 0 to n-1 → `n` operations
- State 1: iterates through positions 1 to n-1 → `n-1` operations
- ...
- State n-1: iterates through position n-1 → `1` operation

The total operations sum up to `n + (n-1) + ... + 1 = n(n+1)/2 = O(n^2)`.

Within each iteration, operations like updating the counter and checking conditions take `O(1)` time, so they don't affect the overall complexity.

**Space Complexity:** `O(n)`

The space complexity consists of:
1. **Recursion stack:** In the worst case, the recursion depth can be `O(n)` when each subarray contains only one element.
2. **Memoization cache:** The `@cache` decorator stores results for at most `n` different states (one for each starting position `i`).
3. **Counter object:** Within each recursive call, a Counter is used which can store at most `O(n)` distinct elements in the worst case, but only one Counter exists at any point in the recursion path.

The dominant factor is the recursion stack depth plus the memoization storage, both of which are `O(n)`, resulting in an overall space complexity of `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. Incorrect Unique Element Count Update Logic

**The Pitfall:**
A common mistake is incorrectly updating the `unique_count` when elements transition between different frequency states. Developers often forget to handle the case when an element's frequency increases beyond 2, leading to incorrect trimmed subarray calculations.

**Incorrect Implementation:**
```python
# WRONG: Only considering transitions between 1 and 2
if frequency_map[current_element] == 1:
    unique_count += 1
elif frequency_map[current_element] == 2:
    unique_count -= 1
# Missing: What if frequency goes from 2 to 3, 3 to 4, etc.?
```

**Why It's Wrong:**
When an element appears 3 or more times, the unique count shouldn't change, but some might mistakenly try to update it further or reset it.

**Correct Solution:**
The provided code is actually correct - it only updates `unique_count` when:
- Frequency changes from 0→1 (element becomes unique): increment
- Frequency changes from 1→2 (element no longer unique): decrement
- Frequency changes from 2→3+ (no change needed): do nothing

### 2. Off-by-One Error in Importance Value Calculation

**The Pitfall:**
Miscalculating the importance value by confusing the trimmed length formula.

**Incorrect Calculation:**
```python
# WRONG: Subtracting unique count from k instead of subarray length
importance_score = k - unique_count
current_cost = importance_score + dp(end_idx + 1)
```

**Correct Solution:**
```python
# CORRECT: Importance = k + (trimmed_length)
# Where trimmed_length = total_length - unique_elements
subarray_length = end_idx - start_idx + 1
trimmed_length = subarray_length - unique_count
importance_score = k + trimmed_length
current_cost = importance_score + dp(end_idx + 1)
```

### 3. Using Global Counter Instead of Local

**The Pitfall:**
Using a single Counter object across all recursive calls, causing frequency counts to bleed between different subarray considerations.

**Incorrect Implementation:**
```python
class Solution:
    def minCost(self, nums: List[int], k: int) -> int:
        frequency_map = Counter()  # WRONG: Global counter
      
        @cache
        def dp(start_idx):
            # Using the global frequency_map
            for end_idx in range(start_idx, n):
                frequency_map[nums[end_idx]] += 1  # Modifies global state
                # ... rest of logic
```

**Why It's Wrong:**
This would accumulate counts across different recursive paths, leading to incorrect frequency calculations.

**Correct Solution:**
Create a new Counter for each call to `dp()`:
```python
def dp(start_idx):
    frequency_map = Counter()  # Local to this function call
    unique_count = 0
    # ... rest of logic
```

### 4. Forgetting to Reset or Clear Cache Between Test Cases

**The Pitfall:**
When testing multiple cases or submitting to an online judge, the `@cache` decorator might retain values from previous test cases if the function is defined at class level.

**Solution:**
Either:
1. Define the cached function inside the main method (as shown in the correct code)
2. Explicitly clear the cache if needed: `dp.cache_clear()`
3. Use manual memoization with a dictionary that's initialized within the method

The provided solution correctly handles this by defining the `dp` function inside the `minCost` method, ensuring a fresh cache for each problem instance.

