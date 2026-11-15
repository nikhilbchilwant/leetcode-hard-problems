# 2551. Put Marbles in Bags

> Source: https://algo.monster/liteproblems/2551

## Problem Statement

You have `k` bags and an array of marbles with given weights. You need to distribute all marbles into exactly `k` bags following specific rules:

1. **Every bag must contain at least one marble** - no empty bags allowed.

2. **Marbles must stay in consecutive order** - if marble at index `i` and marble at index `j` are in the same bag, then all marbles between indices `i` and `j` must also be in that bag. This means you're essentially splitting the array into `k` consecutive segments.

3. **Cost calculation** - for a bag containing marbles from index `i` to index `j` (inclusive), the cost is defined as `weights[i] + weights[j]` (sum of the first and last marble weights in that bag).

The **score** of a distribution is the sum of all bag costs.

Your task is to find the **difference** between the maximum possible score and the minimum possible score across all valid ways to distribute the marbles.

For example, if `weights = [1, 3, 5, 1]` and `k = 2`:
- One way to split: `[1, 3]` and `[5, 1]` 
  - Bag 1 cost: `1 + 3 = 4`
  - Bag 2 cost: `5 + 1 = 6`
  - Total score: `10`
- Another way: `[1]` and `[3, 5, 1]`
  - Bag 1 cost: `1 + 1 = 2`
  - Bag 2 cost: `3 + 1 = 4`
  - Total score: `6`

The problem asks for the difference between all possible maximum and minimum scores from different valid distributions.

## Intuition

Let's think about what happens when we distribute marbles into bags. When we split the array into `k` consecutive segments, we're essentially choosing `k-1` split points in the array.

Consider what contributes to the total score. If we split between index `i` and `i+1`, we create two adjacent bags:
- The bag ending at index `i` contributes `...+ weights[i]` to its cost
- The bag starting at index `i+1` contributes `weights[i+1] +...` to its cost

Here's the key insight: **every split point between positions `i` and `i+1` contributes exactly `weights[i] + weights[i+1]` to the total score**, regardless of where other splits are placed.

To see why, imagine we have marbles at positions `[a, b, c, d, e]` and split it into 3 bags: `[a, b] | [c, d] | [e]`
- Bag 1 cost: `a + b`
- Bag 2 cost: `c + d` 
- Bag 3 cost: `e + e` (single element)
- Total: `a + b + c + d + e + e`

Notice that the total score includes the first and last element of the entire array (which are always included regardless of how we split), plus the sum of elements at each split point. The split after `b` adds `b + c`, and the split after `d` adds `d + e`.

Since the first and last elements of the entire array are constant in every distribution, **the only variable part of the score comes from our choice of `k-1` split points**. Each potential split point between positions `i` and `i+1` has a fixed contribution of `weights[i] + weights[i+1]`.

To maximize the score, we choose the `k-1` split points with the largest contributions. To minimize the score, we choose the `k-1` split points with the smallest contributions. The difference between these gives us our answer.

This transforms the problem from finding optimal ways to partition an array into simply selecting the `k-1` largest and `k-1` smallest values from all possible split point costs.

**Learn more about [Greedy](/problems/greedy_intro), [Sorting](/problems/sorting_summary) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

Based on our intuition that each split point between positions `i` and `i+1` contributes `weights[i] + weights[i+1]` to the total score, we can implement the solution as follows:

**Step 1: Create array of split point costs**

We transform the original `weights` array into an array of all possible split point costs. For an array of length `n`, there are `n-1` possible split points.

```python
arr = [weights[i] + weights[i+1] for i in range(len(weights)-1)]
```

The solution uses `pairwise(weights)` which is a Python utility that generates consecutive pairs: `(weights[0], weights[1])`, `(weights[1], weights[2])`, etc. This elegantly creates the same result.

**Step 2: Sort the split point costs**

To find both the maximum and minimum scores, we need to identify the largest and smallest split point costs:

```python
arr = sorted(a + b for a, b in pairwise(weights))
```

**Step 3: Calculate the difference**

- For the **minimum score**: we choose the `k-1` smallest split points, which are the first `k-1` elements in the sorted array: `arr[:k-1]`
- For the **maximum score**: we choose the `k-1` largest split points, which are the last `k-1` elements in the sorted array: `arr[len(arr)-k+1:]` or `arr[-(k-1):]`

The difference between the maximum and minimum scores is:

```python
sum(arr[len(arr) - k + 1:]) - sum(arr[:k-1])
```

**Time Complexity**: `O(n log n)` where `n` is the length of the weights array, dominated by the [sorting](/problems/sorting_summary) operation.

**Space Complexity**: `O(n)` for storing the array of split point costs.

The elegance of this solution lies in recognizing that we don't need to actually partition the array in different ways - we just need to identify which split points to use for maximum and minimum scores.

## Example Walkthrough

Let's walk through the solution with `weights = [6, 2, 3, 4, 1]` and `k = 3` (we need to split into 3 bags).

**Step 1: Calculate all possible split point costs**

We need to find the cost of splitting between each consecutive pair:
- Split between index 0 and 1: `weights[0] + weights[1] = 6 + 2 = 8`
- Split between index 1 and 2: `weights[1] + weights[2] = 2 + 3 = 5`
- Split between index 2 and 3: `weights[2] + weights[3] = 3 + 4 = 7`
- Split between index 3 and 4: `weights[3] + weights[4] = 4 + 1 = 5`

So our array of split costs is: `[8, 5, 7, 5]`

**Step 2: Sort the split costs**

Sorted array: `[5, 5, 7, 8]`

**Step 3: Select splits for min and max scores**

Since `k = 3`, we need exactly `k-1 = 2` split points.

For **minimum score**, choose the 2 smallest costs:
- Use splits with costs `5` and `5` (the first two elements)
- These correspond to splitting at positions 1→2 and 3→4
- This creates bags: `[6] | [2, 3, 4] | [1]`
- Total score: `(6+6) + (2+4) + (1+1) = 12 + 6 + 2 = 20`

For **maximum score**, choose the 2 largest costs:
- Use splits with costs `7` and `8` (the last two elements)
- These correspond to splitting at positions 0→1 and 2→3
- This creates bags: `[6] | [2, 3] | [4, 1]`
- Total score: `(6+6) + (2+3) + (4+1) = 12 + 5 + 5 = 22`

**Step 4: Calculate the difference**

The difference is simply the sum of the 2 largest costs minus the sum of the 2 smallest costs:
- Sum of largest 2: `7 + 8 = 15`
- Sum of smallest 2: `5 + 5 = 10`
- Difference: `15 - 10 = 5`

We can verify: `22 (max score) - 20 (min score) = 2`... Wait, let me recalculate.

Actually, I made an error in understanding how the split costs relate to the total. The difference between max and min scores equals the difference between the sums of selected split costs, but we need to account for the constant terms (first and last elements of the entire array are always included).

Let me recalculate more carefully:
- Constant contribution: `weights[0] + weights[4] = 6 + 1 = 7` (always included)
- Variable contribution comes from the chosen splits
- Min score uses splits costing `5 + 5 = 10`, total = `7 + 10 = 17`
- Max score uses splits costing `7 + 8 = 15`, total = `7 + 15 = 22`
- Difference: `22 - 17 = 5`

This matches our calculation of `sum(arr[-(k-1):]) - sum(arr[:k-1]) = 15 - 10 = 5`.

## Python Solution

```python
from typing import List
from itertools import pairwise

class Solution:
    def putMarbles(self, weights: List[int], k: int) -> int:
        # Calculate sums of all adjacent pairs in the weights array
        # These represent the cost of making a split between two consecutive marbles
        adjacent_sums = sorted(a + b for a, b in pairwise(weights))
      
        # To get maximum sum: take the (k-1) largest adjacent sums
        # To get minimum sum: take the (k-1) smallest adjacent sums
        # The difference is computed by:
        # - Taking the top (k-1) sums starting from index (len-k+1)
        # - Subtracting the bottom (k-1) sums from index 0 to (k-2)
        max_sum = sum(adjacent_sums[len(adjacent_sums) - k + 1:])
        min_sum = sum(adjacent_sums[:k - 1])
      
        return max_sum - min_sum
```

## Time and Space Complexity

The time complexity is `O(n × log n)`, where `n` is the length of the array `weights`.

- Creating the array of pairwise sums using `pairwise(weights)` takes `O(n)` time and generates `n-1` pairs
- Computing `a + b` for each pair takes `O(n)` time total
- Sorting the array of `n-1` elements takes `O(n × log n)` time, which dominates the overall time complexity
- Array slicing `arr[len(arr) - k + 1:]` and `arr[:k - 1]` each takes `O(k)` time
- Computing the sums of the sliced arrays takes `O(k)` time each
- Since `k ≤ n`, the sorting operation dominates with `O(n × log n)`

The space complexity is `O(n)`, where `n` is the length of the array `weights`.

- The `arr` variable stores `n-1` pairwise sums, requiring `O(n)` space
- The slicing operations create new arrays but their sizes are bounded by `k-1`, which is at most `n-1`
- The sorting operation may require `O(n)` auxiliary space depending on the implementation
- Overall space complexity is `O(n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Misunderstanding the Score Calculation**

A common mistake is thinking that the score for a bag is the sum of ALL marbles in it, rather than just the first and last marble. This leads to completely incorrect approaches that try to minimize/maximize total sums.

**Wrong Understanding:**
```python
# INCORRECT: Thinking score is sum of all marbles in the bag
def wrong_score(weights, i, j):
    return sum(weights[i:j+1])  # This is NOT how the score is calculated!
```

**Correct Understanding:**
```python
# CORRECT: Score is only the sum of first and last marble
def correct_score(weights, i, j):
    return weights[i] + weights[j]  # Only endpoints matter
```

### 2. **Off-by-One Errors with Split Points**

When dealing with k bags, you need exactly k-1 split points, not k splits. This is a frequent source of errors.

**Wrong:**
```python
# INCORRECT: Using k splits instead of k-1
return sum(arr[-k:]) - sum(arr[:k])  # Wrong number of splits!
```

**Correct:**
```python
# CORRECT: Using k-1 splits for k bags
return sum(arr[-(k-1):]) - sum(arr[:k-1])
```

### 3. **Incorrect Slicing for Maximum Values**

Getting the last k-1 elements from a sorted array requires careful indexing. Using `arr[-k+1:]` is incorrect.

**Wrong:**
```python
# INCORRECT: This gives k elements, not k-1
max_sum = sum(arr[-k+1:])  # Gets too many elements
```

**Correct:**
```python
# CORRECT: Multiple valid ways to get last k-1 elements
max_sum = sum(arr[-(k-1):])  # Using negative indexing
# OR
max_sum = sum(arr[len(arr)-k+1:])  # Using positive indexing
```

### 4. **Not Handling Edge Cases**

Failing to consider when k equals 1 or equals the length of the array can cause issues.

**Problematic:**
```python
def putMarbles(weights, k):
    if len(weights) == 1:  # Forgot to handle k == 1
        return 0
    arr = sorted(a + b for a, b in pairwise(weights))
    # Rest of code...
```

**Better:**
```python
def putMarbles(weights, k):
    # When k == 1, no splits needed, max and min are the same
    if k == 1 or k == len(weights):
        return 0
    arr = sorted(a + b for a, b in pairwise(weights))
    # Rest of code...
```

### 5. **Using pairwise Without Import**

If using Python's `pairwise` function, forgetting to import it will cause a NameError.

**Wrong:**
```python
class Solution:
    def putMarbles(self, weights, k):
        # Forgot to import pairwise!
        arr = sorted(a + b for a, b in pairwise(weights))
```

**Correct:**
```python
from itertools import pairwise  # Don't forget this import!

class Solution:
    def putMarbles(self, weights, k):
        arr = sorted(a + b for a, b in pairwise(weights))
```

**Alternative without pairwise:**
```python
# If pairwise is not available or you prefer explicit iteration
arr = sorted(weights[i] + weights[i+1] for i in range(len(weights)-1))
```

