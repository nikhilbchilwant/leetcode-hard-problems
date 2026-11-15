# 2218. Maximum Value of K Coins From Piles

> Source: https://algo.monster/liteproblems/2218

## Problem Statement

You have `n` piles of coins on a table. Each pile contains a positive number of coins with different values.

The coins in each pile are stacked, and you can only take coins from the **top** of any pile. In one move, you can:
- Choose any pile
- Remove the top coin from that pile
- Add that coin to your wallet

You're given:
- `piles`: A list where `piles[i]` represents the `i-th` pile, with coins ordered from **top to bottom**
- `k`: A positive integer representing the exact number of coins you must take

Your goal is to find the **maximum total value** of coins you can collect by taking exactly `k` coins optimally.

For example, if you have a pile `[5, 3, 2]`, the coin with value `5` is on top, and you must take it before you can access the coin with value `3`.

The solution uses [dynamic programming](/problems/dynamic_programming_intro) with a grouped knapsack approach. The state `f[i][j]` represents the maximum value obtainable by taking exactly `j` coins from the first `i` piles. For each pile, we consider taking `0` to `min(j, pile_size)` coins from the top, using a [prefix sum](/problems/subarray_sum) array to efficiently calculate the total value of taking the first `h` coins from pile `i`.

The state transition is: `f[i][j] = max(f[i][j], f[i-1][j-h] + s[h])`, where `s[h]` is the sum of the first `h` coins from pile `i`.

## Intuition

The key insight is recognizing this as a **grouped knapsack problem**. We need to select exactly `k` items (coins), but with a constraint: coins from each pile must be taken in order from top to bottom.

Think of each pile as a "group" where we can take 0, 1, 2, or more items, but always starting from the top. This is different from a standard knapsack where items are independent - here, taking the 3rd coin from a pile requires taking the 1st and 2nd coins first.

This constraint naturally leads us to [dynamic programming](/problems/dynamic_programming_intro). We need to make decisions pile by pile: "For the current pile, how many coins should I take from the top to maximize my total value, given that I've already made optimal decisions for previous piles?"

The state representation becomes clear: `f[i][j]` = "maximum value when considering the first `i` piles and taking exactly `j` coins total". 

For each pile `i` and target count `j`, we try all valid options:
- Take 0 coins from this pile: inherit the best result from previous piles with the same coin count
- Take 1 coin from this pile: add the top coin's value to the best result from previous piles with `j-1` coins
- Take 2 coins from this pile: add the sum of top 2 coins to the best result from previous piles with `j-2` coins
- And so on...

The [prefix sum](/problems/subarray_sum) optimization comes from noticing that we repeatedly need sums like "value of first 1 coin", "value of first 2 coins", etc. Instead of recalculating these sums each time, we precompute them using `accumulate`.

This builds up our solution incrementally, ensuring we always make the optimal choice at each step while respecting the constraint that coins must be taken from the top.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) and [Prefix Sum](/problems/subarray_sum) patterns.**

## Solution Approach

We implement a [dynamic programming](/problems/dynamic_programming_intro) solution using a 2D table to solve this grouped knapsack problem.

**State Definition:**
- Create a 2D DP table `f[i][j]` where `f[i][j]` represents the maximum value obtainable by taking exactly `j` coins from the first `i` piles
- Initialize with dimensions `(n+1) × (k+1)` where `n` is the number of piles

**[Prefix Sum](/problems/subarray_sum) Optimization:**
For each pile, we precompute prefix sums to efficiently calculate the total value of taking the first `h` coins:
```python
s = list(accumulate(nums, initial=0))
```
This gives us `s[h]` = sum of first `h` coins from the current pile, with `s[0] = 0` (taking 0 coins).

**State Transition:**
For each pile `i` (1-indexed) and each possible total coin count `j`:
1. Iterate through all possible choices `h` of how many coins to take from pile `i`
2. The constraint is `h ≤ min(j, len(pile_i))` - we can't take more coins than available in the pile or more than our remaining budget
3. Update the state using:
   ```
   f[i][j] = max(f[i][j], f[i-1][j-h] + s[h])
   ```
   This means: "The best value for `j` coins from first `i` piles is either keeping the previous best, or taking `h` coins from pile `i` (worth `s[h]`) plus the best value for `j-h` coins from the first `i-1` piles"

**Implementation Details:**
- The outer loop iterates through piles using `enumerate(piles, 1)` for 1-based indexing
- The middle loop iterates through all possible total coin counts from 0 to `k`
- The inner loop tries all valid choices for the current pile, breaking early when `j < h` (can't take more coins than our budget)
- Base case is implicitly handled as `f[0][j] = 0` for all `j` (no value from 0 piles)

**Final Answer:**
Return `f[n][k]` - the maximum value from taking exactly `k` coins from all `n` piles.

The time complexity is `O(n × k × m)` where `m` is the average pile size, and space complexity is `O(n × k)` for the DP table.

## Example Walkthrough

Let's walk through a concrete example to illustrate the solution approach.

**Input:**
- `piles = [[10, 5], [3, 7, 1]]` (two piles)
- `k = 3` (must take exactly 3 coins)

**Step 1: Initialize DP Table**
Create a table `f[i][j]` with dimensions `(3 × 4)`:
```
    j=0  j=1  j=2  j=3
i=0  0    0    0    0
i=1  0    0    0    0  
i=2  0    0    0    0
```

**Step 2: Process Pile 1 [10, 5]**

Calculate prefix sums: `s = [0, 10, 15]`
- `s[0] = 0` (take 0 coins)
- `s[1] = 10` (take top coin: 10)
- `s[2] = 15` (take both coins: 10+5)

For each `j` from 0 to 3:
- `j=0`: Can only take 0 coins → `f[1][0] = 0`
- `j=1`: Can take 0 or 1 coin
  - Take 0: `f[0][1] + s[0] = 0 + 0 = 0`
  - Take 1: `f[0][0] + s[1] = 0 + 10 = 10` ✓
  - `f[1][1] = 10`
- `j=2`: Can take 0, 1, or 2 coins
  - Take 0: `f[0][2] + s[0] = 0 + 0 = 0`
  - Take 1: `f[0][1] + s[1] = 0 + 10 = 10`
  - Take 2: `f[0][0] + s[2] = 0 + 15 = 15` ✓
  - `f[1][2] = 15`
- `j=3`: Can take 0, 1, or 2 coins (pile only has 2)
  - Take 0: `f[0][3] + s[0] = 0 + 0 = 0`
  - Take 1: `f[0][2] + s[1] = 0 + 10 = 10`
  - Take 2: `f[0][1] + s[2] = 0 + 15 = 15` ✓
  - `f[1][3] = 15`

Updated table:
```
    j=0  j=1  j=2  j=3
i=0  0    0    0    0
i=1  0    10   15   15
i=2  0    0    0    0
```

**Step 3: Process Pile 2 [3, 7, 1]**

Calculate prefix sums: `s = [0, 3, 10, 11]`
- `s[0] = 0` (take 0 coins)
- `s[1] = 3` (take top coin: 3)
- `s[2] = 10` (take top 2 coins: 3+7)
- `s[3] = 11` (take all 3 coins: 3+7+1)

For each `j` from 0 to 3:
- `j=0`: Can only take 0 coins → `f[2][0] = 0`
- `j=1`: Can take 0 or 1 coin
  - Take 0: `f[1][1] + s[0] = 10 + 0 = 10` ✓
  - Take 1: `f[1][0] + s[1] = 0 + 3 = 3`
  - `f[2][1] = 10`
- `j=2`: Can take 0, 1, or 2 coins
  - Take 0: `f[1][2] + s[0] = 15 + 0 = 15` ✓
  - Take 1: `f[1][1] + s[1] = 10 + 3 = 13`
  - Take 2: `f[1][0] + s[2] = 0 + 10 = 10`
  - `f[2][2] = 15`
- `j=3`: Can take 0, 1, 2, or 3 coins
  - Take 0: `f[1][3] + s[0] = 15 + 0 = 15`
  - Take 1: `f[1][2] + s[1] = 15 + 3 = 18`
  - Take 2: `f[1][1] + s[2] = 10 + 10 = 20` ✓
  - Take 3: `f[1][0] + s[3] = 0 + 11 = 11`
  - `f[2][3] = 20`

Final table:
```
    j=0  j=1  j=2  j=3
i=0  0    0    0    0
i=1  0    10   15   15
i=2  0    10   15   20
```

**Answer:** `f[2][3] = 20`

The optimal strategy is to take both coins from pile 1 (10+5=15) and the top 2 coins from pile 2 (3+7=10), giving a total value of 25. Wait, let me recalculate...

Actually, checking the calculation for `j=3` in pile 2:
- Take 2 from pile 2: We need `f[1][1]` (1 coin from pile 1) which is 10
- Adding `s[2] = 10` gives us 10 + 10 = 20

This means we take 1 coin from pile 1 (value 10) and 2 coins from pile 2 (values 3+7=10), for a total of 20.

## Python Solution

```python
class Solution:
    def maxValueOfCoins(self, piles: List[List[int]], k: int) -> int:
        num_piles = len(piles)
      
        # dp[i][j] represents max value using first i piles with j coins taken
        dp = [[0] * (k + 1) for _ in range(num_piles + 1)]
      
        # Process each pile
        for pile_idx, current_pile in enumerate(piles, 1):
            # Calculate prefix sums for current pile (0, pile[0], pile[0]+pile[1], ...)
            prefix_sums = list(accumulate(current_pile, initial=0))
          
            # For each possible number of coins to take (0 to k)
            for coins_to_take in range(k + 1):
                # Try taking different amounts from current pile
                for coins_from_pile, sum_value in enumerate(prefix_sums):
                    # Can't take more coins than we have budget for
                    if coins_to_take < coins_from_pile:
                        break
                  
                    # Update dp: either skip current pile or take coins_from_pile coins
                    dp[pile_idx][coins_to_take] = max(
                        dp[pile_idx][coins_to_take],
                        dp[pile_idx - 1][coins_to_take - coins_from_pile] + sum_value
                    )
      
        # Return maximum value when using all piles and taking k coins
        return dp[num_piles][k]
```

## Time and Space Complexity

**Time Complexity:** `O(k × L)`

The algorithm uses a triple nested loop structure:
- The outer loop iterates through `n` piles
- The middle loop iterates through `k + 1` possible coins to take
- The inner loop iterates through each pile's coins (up to the size of the current pile)

Since we're iterating through all coins across all piles in the worst case, and for each coin position we perform `O(k)` work, the total time complexity is `O(k × L)`, where `L` is the total number of coins across all piles.

**Space Complexity:** `O(n × k)`

The algorithm uses:
- A 2D DP table `f` of size `(n + 1) × (k + 1)` to store the maximum values
- A temporary prefix sum array `s` for each pile, but this is reused for each pile

The dominant space usage is the DP table, giving us a space complexity of `O(n × k)`, where `n` is the number of piles and `k` is the maximum number of coins we can take.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Index Boundaries When Taking Coins from Current Pile**

**The Pitfall:**
A common mistake is incorrectly handling the loop boundary when deciding how many coins to take from the current pile. Developers often write:
```python
for coins_from_pile in range(len(current_pile) + 1):
    if coins_to_take < coins_from_pile:
        break
    # ... update dp
```

This seems logical but can cause an **IndexError** when accessing `prefix_sums[coins_from_pile]` because `prefix_sums` has length `len(current_pile) + 1`, and we need to ensure we don't exceed the pile size.

**Why This Happens:**
- The confusion arises because we want to consider taking 0 to `len(current_pile)` coins (inclusive)
- The prefix sum array has `len(current_pile) + 1` elements (including the initial 0)
- Developers might iterate beyond valid indices if they don't carefully match the loop with the prefix sum array

**The Solution:**
Use `enumerate(prefix_sums)` to naturally align the number of coins taken with the corresponding prefix sum:
```python
for coins_from_pile, sum_value in enumerate(prefix_sums):
    if coins_to_take < coins_from_pile:
        break
    # Now coins_from_pile ranges from 0 to len(current_pile) automatically
    # and sum_value is the correct corresponding sum
```

### 2. **Not Initializing DP Table Correctly for Edge Cases**

**The Pitfall:**
Some implementations might initialize the DP table incorrectly, particularly for impossible states:
```python
# Wrong: initializing with -1 or negative infinity for all cells
dp = [[-float('inf')] * (k + 1) for _ in range(num_piles + 1)]
dp[0][0] = 0  # Only setting base case
```

**Why This Happens:**
- The assumption that impossible states should be marked with negative infinity
- Not considering that taking 0 coins from any number of piles should yield 0 value

**The Solution:**
Initialize all values to 0, which correctly represents:
- Taking 0 coins from any number of piles gives 0 value
- Any state naturally starts with 0 as the minimum possible value
```python
dp = [[0] * (k + 1) for _ in range(num_piles + 1)]
```

### 3. **Inefficient Prefix Sum Calculation**

**The Pitfall:**
Recalculating the sum of coins for each possible choice within the inner loop:
```python
for coins_from_pile in range(min(coins_to_take, len(current_pile)) + 1):
    # Wrong: calculating sum each time
    sum_value = sum(current_pile[:coins_from_pile]) if coins_from_pile > 0 else 0
    dp[pile_idx][coins_to_take] = max(
        dp[pile_idx][coins_to_take],
        dp[pile_idx - 1][coins_to_take - coins_from_pile] + sum_value
    )
```

**Why This Happens:**
- Not recognizing that we're repeatedly calculating overlapping sums
- This adds unnecessary O(m) complexity for each state transition

**The Solution:**
Pre-compute prefix sums once per pile using `accumulate`:
```python
prefix_sums = list(accumulate(current_pile, initial=0))
# Now prefix_sums[h] gives sum of first h coins in O(1) time
```

### 4. **Forgetting Early Termination Optimization**

**The Pitfall:**
Continuing to iterate through all possible coins from the current pile even when it exceeds the budget:
```python
for coins_from_pile in range(min(len(current_pile), k) + 1):
    if coins_from_pile <= coins_to_take:  # Only process valid states
        # ... update dp
```

**Why This Happens:**
- Using a fixed range that doesn't adapt to the current `coins_to_take` value
- This wastes iterations when `coins_to_take` is small

**The Solution:**
Use early `break` when the number of coins to take from current pile exceeds budget:
```python
for coins_from_pile, sum_value in enumerate(prefix_sums):
    if coins_to_take < coins_from_pile:
        break  # No point checking further
    # ... update dp
```

