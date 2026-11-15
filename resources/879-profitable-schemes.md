# 879. Profitable Schemes

> Source: https://algo.monster/liteproblems/879

## Problem Statement

You have a group of `n` members who can participate in various crimes. Each crime has two properties:
- The `i-th` crime generates `profit[i]` amount of profit
- The `i-th` crime requires exactly `group[i]` members to participate

There are important constraints to consider:
- If a member participates in one crime, they cannot participate in any other crime
- A **profitable scheme** is defined as any subset of crimes that:
  - Generates at least `minProfit` total profit
  - Uses at most `n` total members across all selected crimes

Your task is to count how many different profitable schemes can be formed. Since the answer can be very large, return it modulo `10^9 + 7`.

For example, if you have 5 members total, and you need at least 3 profit, you could select:
- Crime A that needs 2 members and gives 2 profit
- Crime B that needs 2 members and gives 2 profit
This would use 4 members total and generate 4 profit, which satisfies the requirements (≤5 members and ≥3 profit).

The problem asks you to find all possible combinations of crimes that meet these criteria and return the count of such combinations.

## Intuition

This problem is essentially asking us to find all valid subsets of crimes that satisfy our constraints. For each crime, we have a choice: either include it in our scheme or skip it. This binary choice pattern immediately suggests a recursive approach where we explore both possibilities at each step.

Let's think about what information we need to track as we make these decisions:
1. **Which crime we're currently considering** - we need an index `i` to track our position in the crime list
2. **How many members we've already used** - we need `j` to ensure we don't exceed `n` members
3. **How much profit we've accumulated** - we need `k` to check if we've reached `minProfit`

At each crime, we face two choices:
- **Skip the crime**: Move to the next crime without changing our member count or profit
- **Take the crime**: Add its required members to our count and its profit to our total (but only if we have enough members available)

The base case occurs when we've considered all crimes. At that point, we check if our accumulated profit meets the minimum requirement. If yes, we've found one valid scheme; if not, this path doesn't contribute to our answer.

A key optimization insight: once we've reached `minProfit`, any additional profit doesn't create a "different" scheme - we only care about meeting the threshold. So we can cap our profit tracking at `minProfit` using `min(k + profit[i], minProfit)`. This reduces the state space and prevents unnecessary computation.

The recursive nature with overlapping subproblems (same states reached through different paths) makes this perfect for memoization. By caching results for each unique `(i, j, k)` state, we avoid recalculating the same scenarios multiple times.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution implements a recursive approach with memoization using Python's `@cache` decorator. Let's walk through the implementation step by step.

We define a recursive function `dfs(i, j, k)` where:
- `i` represents the index of the current crime being considered
- `j` represents the total number of members already used
- `k` represents the total profit accumulated so far

**Base Case:**
When `i >= len(group)`, we've considered all crimes. At this point, we check if our accumulated profit `k` equals `minProfit`. If yes, we've found a valid scheme and return `1`. Otherwise, return `0`.

**Recursive Cases:**
For each crime at index `i`, we have two choices:

1. **Skip the current crime:** We move to the next crime without changing our member count or profit:
   ```python
   ans = dfs(i + 1, j, k)
   ```

2. **Take the current crime:** We can only do this if adding the required members doesn't exceed our limit:
   ```python
   if j + group[i] <= n:
       ans += dfs(i + 1, j + group[i], min(k + profit[i], minProfit))
   ```
 
   Notice the crucial optimization here: we use `min(k + profit[i], minProfit)` instead of just `k + profit[i]`. This caps the profit at `minProfit` because any profit beyond the minimum threshold doesn't create a different scheme - we only care whether we've met the threshold or not. This significantly reduces the state space.

**Memoization:**
The `@cache` decorator automatically memoizes the function, storing results for each unique `(i, j, k)` combination. This prevents redundant calculations when the same state is reached through different paths.

**Modulo Operation:**
Since the answer can be very large, we apply modulo `10^9 + 7` before returning:
```python
return ans % (10**9 + 7)
```

**Initial Call:**
We start the recursion with `dfs(0, 0, 0)`, meaning we begin at the first crime with no members used and zero profit.

The time complexity is `O(m * n * minProfit)` where `m` is the number of crimes, `n` is the number of members, and `minProfit` is the profit threshold. The space complexity is the same due to memoization storage.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Example:**
- `n = 5` (total members available)
- `minProfit = 3` (minimum profit required)
- `group = [2, 2]` (members needed for each crime)
- `profit = [2, 3]` (profit from each crime)

We'll trace through the recursive calls starting with `dfs(0, 0, 0)`:

**Step 1: dfs(0, 0, 0)**
- Currently at crime 0, with 0 members used and 0 profit
- Two choices:
  1. Skip crime 0 → `dfs(1, 0, 0)`
  2. Take crime 0 (needs 2 members, gives 2 profit) → `dfs(1, 2, 2)`

**Step 2a: dfs(1, 0, 0)** (from skipping crime 0)
- Currently at crime 1, with 0 members used and 0 profit
- Two choices:
  1. Skip crime 1 → `dfs(2, 0, 0)`
  2. Take crime 1 (needs 2 members, gives 3 profit) → `dfs(2, 2, 3)`

**Step 2b: dfs(1, 2, 2)** (from taking crime 0)
- Currently at crime 1, with 2 members used and 2 profit
- Two choices:
  1. Skip crime 1 → `dfs(2, 2, 2)`
  2. Take crime 1 (needs 2 members, gives 3 profit) → `dfs(2, 4, min(2+3, 3))` = `dfs(2, 4, 3)`

**Step 3: Evaluating base cases (i = 2)**
- `dfs(2, 0, 0)`: profit = 0 < minProfit = 3 → returns 0
- `dfs(2, 2, 3)`: profit = 3 = minProfit → returns 1 ✓
- `dfs(2, 2, 2)`: profit = 2 < minProfit = 3 → returns 0
- `dfs(2, 4, 3)`: profit = 3 = minProfit → returns 1 ✓

**Step 4: Backtracking and summing results**
- `dfs(1, 0, 0)` = 0 + 1 = 1
- `dfs(1, 2, 2)` = 0 + 1 = 1
- `dfs(0, 0, 0)` = 1 + 1 = 2

**Result:** There are 2 profitable schemes:
1. Take only crime 1 (uses 2 members, generates 3 profit)
2. Take both crimes (uses 4 members, generates 5 profit, capped at 3 for state tracking)

The key insight is how we cap the profit at `minProfit` when taking crime 1 in Step 2b. Even though the actual profit would be 5, we track it as 3 because any profit ≥ 3 satisfies our requirement, reducing the state space.

## Python Solution

```python
class Solution:
    def profitableSchemes(
        self, n: int, minProfit: int, group: List[int], profit: List[int]
    ) -> int:
        from functools import cache
        from typing import List
      
        MOD = 10**9 + 7
      
        @cache
        def dfs(scheme_idx: int, people_used: int, current_profit: int) -> int:
            """
            Dynamic programming function to count valid schemes.
          
            Args:
                scheme_idx: Current index in the schemes being considered
                people_used: Number of people already used
                current_profit: Total profit accumulated so far (capped at minProfit)
          
            Returns:
                Number of valid schemes from this state
            """
            # Base case: processed all schemes
            if scheme_idx >= len(group):
                # Valid scheme if we've reached minimum profit requirement
                return 1 if current_profit == minProfit else 0
          
            # Option 1: Skip current scheme
            ways = dfs(scheme_idx + 1, people_used, current_profit)
          
            # Option 2: Include current scheme if we have enough people
            if people_used + group[scheme_idx] <= n:
                # Cap profit at minProfit to avoid unnecessary states
                new_profit = min(current_profit + profit[scheme_idx], minProfit)
                ways += dfs(scheme_idx + 1, people_used + group[scheme_idx], new_profit)
          
            # Return result modulo MOD to prevent overflow
            return ways % MOD
      
        # Start DFS with: first scheme, 0 people used, 0 profit
        return dfs(0, 0, 0)
```

## Time and Space Complexity

The time complexity is `O(m × n × minProfit)`, where:
- `m` is the length of the `group` array (number of crimes/jobs)
- `n` is the maximum number of members that can be used
- `minProfit` is the minimum profit threshold

This is because the recursive function `dfs(i, j, k)` has three parameters:
- `i` ranges from `0` to `m` (length of group array)
- `j` ranges from `0` to `n` (number of members used)
- `k` ranges from `0` to `minProfit` (accumulated profit, capped at minProfit)

Due to the `@cache` decorator, each unique combination of `(i, j, k)` is computed only once, resulting in `O(m × n × minProfit)` total states.

The space complexity is `O(m × n × minProfit)` as well, which comes from:
- The memoization cache storing all possible states: `O(m × n × minProfit)`
- The recursion call stack depth which is at most `O(m)` (one call per crime/job)

Since `O(m × n × minProfit) > O(m)`, the overall space complexity is `O(m × n × minProfit)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Not Capping the Profit State**

**The Pitfall:** A natural first approach tracks the exact profit without capping it. It returns correct counts, but it is far too slow:

```python
# Correct results, but will cause TLE or MLE
def dfs(i, j, k):
    if i >= len(group):
        return 1 if k >= minProfit else 0  # Check if profit meets threshold
  
    ans = dfs(i + 1, j, k)
    if j + group[i] <= n:
        ans += dfs(i + 1, j + group[i], k + profit[i])  # Accumulate exact profit
  
    return ans % MOD
```

**Why This Fails:** This creates too many unique states. If profits can be large (e.g., up to 100) and you have many crimes, the total profit could reach thousands, creating thousands of distinct states for the profit dimension alone. This leads to:
- Time Limit Exceeded (TLE) due to exploring too many states
- Memory Limit Exceeded (MLE) from storing too many memoized results

**The Solution:** Cap the profit at `minProfit`:
```python
new_profit = min(current_profit + profit[scheme_idx], minProfit)
```

This works because once we've achieved `minProfit`, any additional profit doesn't create a "different" valid scheme - we only care about meeting the threshold. This optimization reduces the profit dimension from potentially unbounded values to at most `minProfit + 1` distinct states (0 through minProfit).

### 2. **Incorrect Base Case Logic**

**The Pitfall:** Writing the base case as:
```python
# WRONG - counts schemes with profit < minProfit as valid
if i >= len(group):
    return 1
```

**Why This Fails:** This version counts every subset whose member total fits within `n`, including those that do not meet the profit requirement. For `n = 5, minProfit = 3, group = [2, 2], profit = [2, 3]` it returns 4 (all subsets) instead of 2.

**The Solution:** Check the profit in the base case. Because profit is capped at `minProfit`, the test can be written as exact equality:
```python
if scheme_idx >= len(group):
    return 1 if current_profit == minProfit else 0
```

Since the capped profit never exceeds `minProfit`, reaching exactly `minProfit` means we've met or exceeded the threshold. Writing `current_profit >= minProfit` gives the same result.

### 3. **Forgetting the Modulo Operation**

**The Pitfall:** Only applying modulo at the final return:
```python
# RISKY - intermediate calculations might overflow
def dfs(i, j, k):
    # ... base case ...
  
    ans = dfs(i + 1, j, k)
    if j + group[i] <= n:
        ans += dfs(i + 1, j + group[i], min(k + profit[i], minProfit))
  
    return ans  # No modulo here

# Only modulo at the end
return dfs(0, 0, 0) % MOD
```

**Why This Fails:** While Python handles arbitrary precision integers, in other languages this could cause integer overflow. Additionally, even in Python, very large numbers can slow down arithmetic operations.

**The Solution:** Apply modulo at each step:
```python
return ways % MOD
```

This keeps numbers manageable and ensures consistency across different programming languages.

