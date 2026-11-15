# 1655. Distribute Repeating Integers

> Source: https://algo.monster/liteproblems/1655

## Problem Statement

You have an array `nums` containing `n` integers, where there are at most 50 unique values in the array. You also have an array `quantity` containing `m` customer orders, where `quantity[i]` represents how many integers the `i`-th customer wants.

You need to distribute the integers from `nums` to satisfy all customers with the following conditions:

1. Each customer `i` must receive exactly `quantity[i]` integers
2. All integers given to a single customer must have the same value (for example, if customer 1 gets 3 integers, they must all be the same number like [5, 5, 5])
3. Every customer must be satisfied with their allocation

The task is to determine if it's possible to distribute all integers from `nums` according to these rules. Return `true` if such a distribution exists, otherwise return `false`.

For example, if `nums = [1, 1, 2, 2, 2]` and `quantity = [2, 3]`, you could give customer 0 two 1's and customer 1 three 2's, making it possible to satisfy all customers, so the answer would be `true`.

## Intuition

The key insight is recognizing that we only care about the **counts** of each unique value in `nums`, not the actual values themselves. If we have five 2's and three 7's, what matters is that we have a group of 5 identical numbers and a group of 3 identical numbers.

Since each customer wants all their integers to be the same value, we can think of this as: "Can we assign the available groups of identical numbers to satisfy all customers?"

With at most 50 unique values and typically around 10 customers (small constraints), we can explore different ways to distribute these groups. But naive [backtracking](/problems/backtracking) would be inefficient because we'd repeatedly solve the same subproblems.

Here's where the breakthrough comes: We can represent which customers have been satisfied using a [bitmask](/problems/bitmask_intro). If we have 3 customers, the bitmask `101` means customers 0 and 2 are satisfied. This allows us to use [dynamic programming](/problems/dynamic_programming_intro) to remember: "Can we satisfy this specific subset of customers using the first `i` groups of numbers?"

The state `f[i][j]` answers: "Can we satisfy the customer subset `j` using only the first `i` unique numbers from our array?"

For each state, we try all possible ways to use the current group of identical numbers:
- We could use it to satisfy customer subset `k` (if the group is large enough)
- The remaining customers (`j XOR k`) should be satisfiable using the previous groups

By trying all possible subsets `k` of `j`, we're essentially doing optimized [backtracking](/problems/backtracking) - exploring all distributions but memorizing results to avoid redundant work.

The preprocessing step `s[j]` that calculates the total demand for each customer subset helps us quickly check if a group of identical numbers is large enough to satisfy that subset.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro), [Backtracking](/problems/backtracking) and [Bitmask](/problems/bitmask_intro) patterns.**

## Solution Approach

The solution uses **state compression [dynamic programming](/problems/dynamic_programming_intro)** combined with **subset enumeration** to efficiently explore all possible distributions.

**Step 1: Count frequencies and prepare data**
- Count occurrences of each number in `nums` using a Counter: `cnt = Counter(nums)`
- Extract just the counts into array `arr` since actual values don't matter
- Store `n = len(arr)` (number of unique values) and `m = len(quantity)` (number of customers)

**Step 2: Precompute subset sums**
- Create array `s` where `s[mask]` represents the total quantity demanded by the customer subset represented by `mask`
- For each mask from 1 to `(1 << m) - 1`:
  - Find the first set bit `j` in the mask
  - Calculate `s[mask] = s[mask ^ (1 << j)] + quantity[j]`
- This allows O(1) lookup of total demand for any customer subset

**Step 3: Initialize DP table**
- Create 2D DP table `f[n][1 << m]` where `f[i][j]` represents whether we can satisfy customer subset `j` using only the first `i+1` unique numbers
- Initialize `f[i][0] = True` for all `i` (empty subset is always satisfiable)

**Step 4: Fill DP table**
For each unique number `i` with count `x = arr[i]`:
- For each customer subset `j` from 1 to `(1 << m) - 1`:
  - **Option 1**: Don't use current number for subset `j`
    - If `i > 0` and `f[i-1][j] = True`, then `f[i][j] = True`
  - **Option 2**: Use current number for some subset `k` of `j`
    - Enumerate all subsets `k` of `j` using `k = (k - 1) & j`
    - Check if:
      - Previous numbers can satisfy remaining customers: `f[i-1][j ^ k] = True` (or `j == k` if `i == 0`)
      - Current number has enough count: `s[k] <= x`
    - If both conditions met, set `f[i][j] = True`

**Step 5: Return result**
- Return `f[n-1][(1 << m) - 1]`, which indicates whether all customers can be satisfied using all available unique numbers

The subset enumeration technique `k = (k - 1) & j` efficiently iterates through all subsets of `j` in decreasing order, ensuring we check all possible ways to use the current group of identical numbers.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Example:** `nums = [1, 1, 2, 2, 2]`, `quantity = [2, 3]`

**Step 1: Count frequencies and prepare data**
- Count occurrences: `{1: 2, 2: 3}`
- Extract counts: `arr = [2, 3]` (we have 2 ones and 3 twos)
- `n = 2` (two unique values), `m = 2` (two customers)

**Step 2: Precompute subset sums**
Calculate total demand for each customer subset:
- `s[00] = 0` (no customers)
- `s[01] = quantity[0] = 2` (customer 0 only)
- `s[10] = quantity[1] = 3` (customer 1 only)  
- `s[11] = quantity[0] + quantity[1] = 5` (both customers)

**Step 3: Initialize DP table**
Create `f[2][4]` table (2 unique numbers × 4 possible customer subsets):
```
f[0][00] = True  (empty subset always satisfiable)
f[1][00] = True
```

**Step 4: Fill DP table**

*Processing first unique number (2 ones):*
- `f[0][01]`: Can we satisfy customer 0 using 2 ones?
  - Check subset `01`: Need 2 items, have 2 ones ✓
  - Set `f[0][01] = True`
- `f[0][10]`: Can we satisfy customer 1 using 2 ones?
  - Check subset `10`: Need 3 items, have 2 ones ✗
  - Set `f[0][10] = False`
- `f[0][11]`: Can we satisfy both customers using 2 ones?
  - Check subset `01`: Need 2 items, have 2 ones ✓, but remaining subset `10` needs 3 items
  - Check subset `10`: Need 3 items, have 2 ones ✗
  - Check subset `11`: Need 5 items, have 2 ones ✗
  - Set `f[0][11] = False`

*Processing second unique number (3 twos):*
- `f[1][01]`: Can we satisfy customer 0 using first two groups?
  - Inherit from `f[0][01] = True` (already satisfied with ones)
  - Set `f[1][01] = True`
- `f[1][10]`: Can we satisfy customer 1 using first two groups?
  - Check if we can use 3 twos for customer 1: Need 3, have 3 ✓
  - Previous groups must satisfy empty set: `f[0][00] = True` ✓
  - Set `f[1][10] = True`
- `f[1][11]`: Can we satisfy both customers using first two groups?
  - Try using twos for subset `10` (customer 1):
    - Need 3 items, have 3 twos ✓
    - Remaining subset `01` must be satisfied by ones: `f[0][01] = True` ✓
  - Set `f[1][11] = True`

**Step 5: Return result**
- Check `f[1][11] = True`, meaning we can satisfy all customers using all available numbers
- Return `true`

The solution correctly identifies that we can give customer 0 the two 1's and customer 1 the three 2's, satisfying everyone's requirements.

## Python Solution

```python
class Solution:
    def canDistribute(self, nums: List[int], quantity: List[int]) -> bool:
        num_customers = len(quantity)
      
        # Precompute sum of quantities for each subset of customers
        # subset_sum[mask] = total quantity needed for customers in mask
        subset_sum = [0] * (1 << num_customers)
        for mask in range(1, 1 << num_customers):
            # Find the first set bit and calculate sum incrementally
            for customer_idx in range(num_customers):
                if mask >> customer_idx & 1:
                    # Remove this customer from mask and add their quantity
                    subset_sum[mask] = subset_sum[mask ^ (1 << customer_idx)] + quantity[customer_idx]
                    break
      
        # Count frequency of each number in nums
        frequency_map = Counter(nums)
        unique_values = list(frequency_map.values())
        num_unique = len(unique_values)
      
        # dp[i][mask] = True if we can satisfy customers in mask using first i+1 unique numbers
        dp = [[False] * (1 << num_customers) for _ in range(num_unique)]
      
        # Base case: empty set of customers can always be satisfied
        for i in range(num_unique):
            dp[i][0] = True
      
        # Fill the DP table
        for idx, count in enumerate(unique_values):
            for customer_mask in range(1, 1 << num_customers):
                # Case 1: If we could satisfy this mask without current number
                if idx > 0 and dp[idx - 1][customer_mask]:
                    dp[idx][customer_mask] = True
                    continue
              
                # Case 2: Try to use current number for some subset of customers
                subset = customer_mask
                while subset:
                    # Check if we can satisfy 'subset' with current number
                    # and remaining customers with previous numbers
                    can_use_previous = (customer_mask == subset if idx == 0 
                                       else dp[idx - 1][customer_mask ^ subset])
                    can_satisfy_subset = subset_sum[subset] <= count
                  
                    if can_use_previous and can_satisfy_subset:
                        dp[idx][customer_mask] = True
                        break
                  
                    # Move to next subset of customer_mask
                    subset = (subset - 1) & customer_mask
      
        # Return whether we can satisfy all customers using all unique numbers
        return dp[-1][-1]
```

## Time and Space Complexity

**Time Complexity:** `O(n * 3^m)`

The time complexity analysis breaks down as follows:
- Computing the sum array `s` takes `O(2^m * m)` time, where we iterate through all `2^m` subsets and for each subset, we may iterate through `m` positions to find the first set bit.
- The main dynamic programming section has:
  - Outer loop iterating through `n` different values (unique counts in `nums`)
  - Middle loop iterating through all `2^m` possible subsets `j`
  - Inner while loop that iterates through all subsets `k` of `j`

For the inner loop, the key insight is that for a fixed `j`, the number of subsets `k` of `j` is `2^(number of 1s in j)`. Summing over all possible `j` values:
- Each element in `quantity` can be in one of 3 states relative to subsets `j` and `k`: not in `j`, in `j` but not in `k`, or in both `j` and `k`
- This gives us `3^m` total iterations when summing across all `j` and their subsets `k`

Therefore, the overall time complexity is `O(2^m * m + n * 3^m) = O(n * 3^m)` since `3^m` dominates `2^m * m`.

**Space Complexity:** `O(n * 2^m)`

The space complexity comes from:
- Array `s` storing sums for all `2^m` subsets: `O(2^m)`
- 2D DP array `f` with dimensions `n × 2^m`: `O(n * 2^m)`
- Counter and array storage: `O(n)`

The dominant term is `O(n * 2^m)` from the DP table.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Inefficient Subset Enumeration Leading to TLE**

A common mistake is using inefficient methods to enumerate subsets, such as generating all possible subsets and checking if each is a subset of the target mask. This approach has exponential time complexity that becomes prohibitive even for moderate input sizes.

**Incorrect approach:**
```python
# Inefficient: Checking all 2^m subsets for each mask
for subset in range(1 << num_customers):
    if (subset & customer_mask) == subset:  # Check if subset of customer_mask
        # Process subset
```

**Correct approach:**
```python
# Efficient: Only enumerate actual subsets of customer_mask
subset = customer_mask
while subset:
    # Process subset
    subset = (subset - 1) & customer_mask
```

The key insight is that `(subset - 1) & customer_mask` generates the next smaller subset directly, visiting only actual subsets rather than all possible combinations.

### 2. **Incorrect DP State Initialization**

Another pitfall is forgetting to properly handle the base case when `idx == 0`. When processing the first unique number, there are no "previous numbers" to rely on, so the condition for using this number differs from subsequent iterations.

**Incorrect initialization:**
```python
# Wrong: Always checking dp[idx-1] even when idx == 0
if dp[idx - 1][customer_mask ^ subset] and subset_sum[subset] <= count:
    dp[idx][customer_mask] = True
```

**Correct initialization:**
```python
# Correct: Special handling for first unique number
can_use_previous = (customer_mask == subset if idx == 0 
                   else dp[idx - 1][customer_mask ^ subset])
if can_use_previous and subset_sum[subset] <= count:
    dp[idx][customer_mask] = True
```

When `idx == 0`, we can only satisfy `customer_mask` if we use all of it with the current number (hence `customer_mask == subset`), since there are no previous numbers available.

### 3. **Memory Optimization Oversight**

While not necessarily incorrect, failing to optimize memory usage can be problematic for larger inputs. Since each DP state only depends on the previous row, we can use rolling arrays to reduce space complexity from O(n × 2^m) to O(2^m).

**Memory-optimized version:**
```python
# Use two 1D arrays instead of 2D array
prev = [False] * (1 << num_customers)
curr = [False] * (1 << num_customers)
prev[0] = True

for count in unique_values:
    curr[0] = True
    for customer_mask in range(1, 1 << num_customers):
        curr[customer_mask] = prev[customer_mask]  # Don't use current number
      
        subset = customer_mask
        while subset and not curr[customer_mask]:
            if prev[customer_mask ^ subset] and subset_sum[subset] <= count:
                curr[customer_mask] = True
            subset = (subset - 1) & customer_mask
  
    prev, curr = curr, prev

return prev[-1]
```

