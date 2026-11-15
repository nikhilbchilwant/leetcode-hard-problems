# 2448. Minimum Cost to Make Array Equal

> Source: https://algo.monster/liteproblems/2448

## Problem Statement

You have two arrays `nums` and `cost`, both containing `n` positive integers. Each element `nums[i]` can be increased or decreased by 1, and doing this operation on the `i`-th element costs `cost[i]`.

Your goal is to make all elements in the `nums` array equal by performing these operations. You need to find the minimum total cost to achieve this.

For example, if `nums = [1, 3, 5]` and `cost = [2, 1, 3]`:
- To change `nums[0]` from 1 to any other value, each increment/decrement costs 2
- To change `nums[1]` from 3 to any other value, each increment/decrement costs 1
- To change `nums[2]` from 5 to any other value, each increment/decrement costs 3

The task is to determine what value all elements should become and calculate the minimum total cost. The total cost is calculated as the sum of `|nums[i] - target| * cost[i]` for all elements, where `target` is the chosen equal value.

The solution uses a sorted approach combined with prefix sums. After [sorting](/problems/sorting_summary) the `(nums[i], cost[i])` pairs by `nums[i]`, it considers each unique value in `nums` as a potential target. For each potential target value `x`, it calculates:
- The cost to increase all smaller values to `x`: `x * Σ(cost[j]) - Σ(nums[j] * cost[j])` for all `j` where `nums[j] < x`
- The cost to decrease all larger values to `x`: `Σ(nums[j] * cost[j]) - x * Σ(cost[j])` for all `j` where `nums[j] > x`

The [prefix sum](/problems/subarray_sum) arrays `f` and `g` store cumulative weighted sums and cumulative costs respectively, allowing efficient calculation of these costs for each potential target.

## Intuition

The key insight is that the optimal target value must be one of the existing values in the `nums` array. Why? Consider the cost function for any target value `x`. The total cost is `Σ|nums[i] - x| * cost[i]`, which is a weighted sum of absolute differences. This function is piecewise linear - it changes its slope only at the points where `x` equals one of the values in `nums`. Between any two consecutive values in `nums`, the cost function is linear, so the minimum cannot occur in the middle of an interval; it must occur at one of the endpoints.

Once we realize we only need to check existing values in `nums` as potential targets, the next challenge is efficiently calculating the cost for each candidate. A naive approach would calculate `Σ|nums[i] - target| * cost[i]` for each target, taking O(n²) time.

To optimize this, we can sort the array and use prefix sums. After [sorting](/problems/sorting_summary) by `nums` values, for any target value `x`, we can split the cost calculation into two parts:
- Elements smaller than `x`: contribute `(x - nums[i]) * cost[i]`
- Elements larger than `x`: contribute `(nums[i] - x) * cost[i]`

By rearranging these sums:
- Left side cost: `x * Σcost[i] - Σ(nums[i] * cost[i])` for all `i` where `nums[i] ≤ x`
- Right side cost: `Σ(nums[i] * cost[i]) - x * Σcost[i]` for all `i` where `nums[i] > x`

We can precompute two [prefix sum](/problems/subarray_sum) arrays:
- `f[i]`: cumulative sum of `nums[j] * cost[j]` for `j < i`
- `g[i]`: cumulative sum of `cost[j]` for `j < i`

With these prefix sums, calculating the cost for any target value becomes O(1), reducing the overall complexity to O(n log n) due to [sorting](/problems/sorting_summary).

**Learn more about [Greedy](/problems/greedy_intro), [Binary Search](/problems/binary_search_intro), [Prefix Sum](/problems/subarray_sum) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

The implementation follows these steps:

1. **Create and Sort Pairs**: First, we zip the `nums` and `cost` arrays together and sort them by the `nums` values. This gives us `arr = sorted(zip(nums, cost))`, where each element is a tuple `(nums[i], cost[i])` sorted by the first element.

2. **Build [Prefix Sum](/problems/subarray_sum) Arrays**: We create two prefix sum arrays with length `n+1`:
   - `f[i]`: stores the cumulative sum of `nums[j] * cost[j]` for all `j < i`
   - `g[i]`: stores the cumulative sum of `cost[j]` for all `j < i`

   The construction loop:
   ```python
   for i in range(1, n + 1):
       a, b = arr[i - 1]  # a = nums value, b = cost value
       f[i] = f[i - 1] + a * b
       g[i] = g[i - 1] + b
   ```

3. **Enumerate Each Potential Target**: For each position `i` in the sorted array, we consider `arr[i-1][0]` as the target value `a`. We calculate the total cost as the sum of:

   - **Left side cost** (`l`): Cost to move all elements less than `a` up to `a`
     - Formula: `a * g[i-1] - f[i-1]`
     - `g[i-1]` is the sum of costs for elements before position `i`
     - `f[i-1]` is the sum of `(value * cost)` for elements before position `i`

   - **Right side cost** (`r`): Cost to move all elements greater than `a` down to `a`
     - Formula: `f[n] - f[i] - a * (g[n] - g[i])`
     - `f[n] - f[i]` is the sum of `(value * cost)` for elements after position `i`
     - `g[n] - g[i]` is the sum of costs for elements after position `i`

4. **Track Minimum Cost**: We keep track of the minimum total cost across all potential targets:
   ```python
   ans = min(ans, l + r)
   ```

The mathematical derivation for the cost formulas:
- For elements less than target `a`: Cost = `Σ(a - nums[j]) * cost[j] = a * Σcost[j] - Σ(nums[j] * cost[j])`
- For elements greater than target `a`: Cost = `Σ(nums[j] - a) * cost[j] = Σ(nums[j] * cost[j]) - a * Σcost[j]`

Time complexity: O(n log n) for [sorting](/problems/sorting_summary), O(n) for building prefix sums, and O(n) for enumeration, resulting in O(n log n) overall.
Space complexity: O(n) for the sorted array and [prefix sum](/problems/subarray_sum) arrays.

## Example Walkthrough

Let's walk through a small example with `nums = [1, 3, 5, 2]` and `cost = [2, 1, 3, 4]`.

**Step 1: Create and Sort Pairs**
We combine nums and cost, then sort by nums values:
- Original pairs: `[(1,2), (3,1), (5,3), (2,4)]`
- Sorted array: `arr = [(1,2), (2,4), (3,1), (5,3)]`

**Step 2: Build Prefix Sum Arrays**
We create arrays `f` and `g` where:
- `f[i]` = cumulative sum of `nums[j] * cost[j]` for j < i
- `g[i]` = cumulative sum of `cost[j]` for j < i

| i | arr[i-1] | f[i] | g[i] |
|---|----------|------|------|
| 0 | -        | 0    | 0    |
| 1 | (1,2)    | 0 + 1×2 = 2  | 0 + 2 = 2 |
| 2 | (2,4)    | 2 + 2×4 = 10 | 2 + 4 = 6 |
| 3 | (3,1)    | 10 + 3×1 = 13 | 6 + 1 = 7 |
| 4 | (5,3)    | 13 + 5×3 = 28 | 7 + 3 = 10 |

**Step 3: Calculate Cost for Each Potential Target**

For **target = 1** (i=1):
- Left cost: `1×g[0] - f[0] = 1×0 - 0 = 0` (no elements to the left)
- Right cost: `(f[4]-f[1]) - 1×(g[4]-g[1]) = (28-2) - 1×(10-2) = 26 - 8 = 18`
- Total: 0 + 18 = 18

For **target = 2** (i=2):
- Left cost: `2×g[1] - f[1] = 2×2 - 2 = 2` (move 1→2 costs 1×2)
- Right cost: `(f[4]-f[2]) - 2×(g[4]-g[2]) = (28-10) - 2×(10-6) = 18 - 8 = 10`
- Total: 2 + 10 = 12

For **target = 3** (i=3):
- Left cost: `3×g[2] - f[2] = 3×6 - 10 = 8` (move 1→3 costs 2×2, move 2→3 costs 1×4)
- Right cost: `(f[4]-f[3]) - 3×(g[4]-g[3]) = (28-13) - 3×(10-7) = 15 - 9 = 6`
- Total: 8 + 6 = 14

For **target = 5** (i=4):
- Left cost: `5×g[3] - f[3] = 5×7 - 13 = 22` (move all elements up to 5)
- Right cost: `(f[4]-f[4]) - 5×(g[4]-g[4]) = 0 - 0 = 0` (no elements to the right)
- Total: 22 + 0 = 22

**Step 4: Find Minimum**
The minimum cost is 12, achieved when target = 2.

We can verify: Making all elements equal to 2:
- Change 1→2: |1-2|×2 = 2
- Change 3→2: |3-2|×1 = 1
- Change 5→2: |5-2|×3 = 9
- Change 2→2: |2-2|×4 = 0
- Total: 2 + 1 + 9 + 0 = 12 ✓

## Python Solution

```python
class Solution:
    def minCost(self, nums: List[int], cost: List[int]) -> int:
        # Sort pairs of (number, cost) by number value
        sorted_pairs = sorted(zip(nums, cost))
        n = len(sorted_pairs)

        # prefix_weighted_sum[i] = sum of (number * cost) for first i elements
        prefix_weighted_sum = [0] * (n + 1)
        # prefix_cost_sum[i] = sum of costs for first i elements
        prefix_cost_sum = [0] * (n + 1)

        # Build prefix sums
        for i in range(1, n + 1):
            num_val, cost_val = sorted_pairs[i - 1]
            prefix_weighted_sum[i] = prefix_weighted_sum[i - 1] + num_val * cost_val
            prefix_cost_sum[i] = prefix_cost_sum[i - 1] + cost_val

        min_total_cost = float('inf')

        # Try each position as the target point
        for i in range(1, n + 1):
            target_num = sorted_pairs[i - 1][0]

            # Cost to move all elements on the left to target_num
            # For elements left of target: cost = sum(cost[j] * (target - num[j]))
            left_cost = target_num * prefix_cost_sum[i - 1] - prefix_weighted_sum[i - 1]

            # Cost to move all elements on the right to target_num
            # For elements right of target: cost = sum(cost[j] * (num[j] - target))
            right_cost = (prefix_weighted_sum[n] - prefix_weighted_sum[i]) - target_num * (prefix_cost_sum[n] - prefix_cost_sum[i])

            # Update minimum total cost
            min_total_cost = min(min_total_cost, left_cost + right_cost)

        return min_total_cost
```

## Time and Space Complexity

**Time Complexity:** `O(n × log n)`, where `n` is the length of the array `nums`.

- The dominant operation is sorting the array of tuples `(nums[i], cost[i])`, which takes `O(n × log n)` time.
- Creating the prefix sum arrays `f` and `g` requires iterating through the sorted array once, taking `O(n)` time.
- Finding the minimum cost by iterating through all possible median positions takes `O(n)` time.
- Overall: `O(n × log n) + O(n) + O(n) = O(n × log n)`

**Space Complexity:** `O(n)`

- The sorted array `arr` stores `n` tuples, requiring `O(n)` space.
- The prefix sum arrays `f` and `g` each require `O(n + 1) = O(n)` space.
- Additional variables use `O(1)` space.
- Overall: `O(n) + O(n) + O(n) = O(n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Not Considering Duplicate Values in the Sorted Array**

A common mistake is assuming that each unique value needs to be checked as a potential target. However, when there are duplicate values in `nums`, the current implementation correctly handles this by checking each position, even if the values are the same. This is actually optimal because:

- When multiple elements have the same value but different costs, the prefix sums naturally accumulate correctly
- The algorithm doesn't need special handling for duplicates

**Why this works:** For duplicate values at positions `i` and `i+1`, when we calculate the cost for position `i+1`, the element at position `i` (with the same value) contributes 0 to the total cost since `|nums[i] - target| = 0`.

### 2. **Integer Overflow in Prefix Sum Calculations**

The most critical pitfall is potential integer overflow when calculating `prefix_weighted_sum`. Since we're multiplying `nums[i] * cost[i]` and accumulating these products, the values can grow very large.

**Problem scenario:**
- If `nums[i]` and `cost[i]` can be up to `10^5` each, and we have `n = 10^5` elements
- The maximum value of a single product: `10^5 * 10^5 = 10^10`
- The maximum accumulated sum: `10^5 * 10^10 = 10^15`

**Solution:** In Python, integers have arbitrary precision, so overflow isn't an issue. However, in languages like Java or C++, you should:
```python
# Use long long in C++ or long in Java
# In Python, this is handled automatically
prefix_weighted_sum = [0] * (n + 1)  # Python handles large integers
```

### 3. **Off-by-One Errors in Prefix Sum Indexing**

A subtle but common mistake is incorrectly indexing the prefix arrays when calculating left and right costs.

**Incorrect approach:**
```python
# Wrong: Using wrong indices
left_cost = target_num * prefix_cost_sum[i] - prefix_weighted_sum[i]  # Should be i-1
right_cost = (prefix_weighted_sum[n] - prefix_weighted_sum[i-1]) - target_num * (prefix_cost_sum[n] - prefix_cost_sum[i-1])  # Should be i
```

**Correct approach:**
```python
# Correct: For position i (1-indexed), elements before it are at indices [0, i-1]
left_cost = target_num * prefix_cost_sum[i - 1] - prefix_weighted_sum[i - 1]
right_cost = (prefix_weighted_sum[n] - prefix_weighted_sum[i]) - target_num * (prefix_cost_sum[n] - prefix_cost_sum[i])
```

### 4. **Not Considering the Weighted Median Optimization**

While the current O(n log n) solution works, a more sophisticated approach using weighted median can solve this in O(n) time after sorting. The pitfall is not recognizing that this is essentially a weighted median problem.

**Insight:** The optimal target is the weighted median of the values, where weights are the costs. This can be found by:
1. Finding the point where cumulative cost reaches half of total cost
2. That position's value is the optimal target

**Optimized approach:**
```python
def minCost(self, nums: List[int], cost: List[int]) -> int:
    sorted_pairs = sorted(zip(nums, cost))
    total_cost = sum(cost)
    accumulated = 0

    # Find weighted median
    for num_val, cost_val in sorted_pairs:
        accumulated += cost_val
        if accumulated >= total_cost / 2:
            target = num_val
            break

    # Calculate minimum cost using the weighted median
    return sum(abs(num - target) * c for num, c in zip(nums, cost))
```

This optimization reduces the enumeration step from O(n) checks to O(1) by directly finding the optimal target.

