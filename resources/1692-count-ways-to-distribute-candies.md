# 1692. Count Ways to Distribute Candies

> Source: https://algo.monster/liteproblems/1692

## Problem Statement

You have `n` unique candies (labeled from `1` to `n`) and `k` bags. Your task is to distribute **all** the candies into the bags such that every bag contains **at least** one candy.

Two distribution methods are considered **different** if there exists at least one bag in the first distribution whose candies are not all together in the same bag in the second distribution. The order of bags and the order of candies within each bag don't matter.

For example:
- `(1), (2,3)` and `(2), (1,3)` are **different** distributions because candies `2` and `3` are together in bag `(2,3)` in the first distribution, but they are separated into different bags in the second distribution.
- `(1), (2,3)` and `(3,2), (1)` are the **same** distribution because the candies in each bag remain together (bag contents are `{1}` and `{2,3}` in both cases, just reordered).

Given two integers `n` (number of candies) and `k` (number of bags), return the number of different ways to distribute the candies. Since the answer can be very large, return it modulo `10^9 + 7`.

The problem essentially asks you to find the number of ways to partition a set of `n` distinct elements into exactly `k` non-empty subsets, which is known as the Stirling number of the second kind `S(n,k)`.

## Intuition

Let's think about how to build up the solution step by step. When we have `i` candies to distribute into `j` bags, we can focus on what happens with the `i`-th candy.

For the `i`-th candy, we have two choices:

1. **Put it in a new bag**: If we already have `i-1` candies distributed into `j-1` bags, we can add the `i`-th candy in a new (the `j`-th) bag. The number of ways to do this is exactly `f[i-1][j-1]`.

2. **Put it in an existing bag**: If we already have `i-1` candies distributed into `j` bags, we can add the `i`-th candy to any of these `j` existing bags. Since we have `j` choices for which bag to put it in, the number of ways is `f[i-1][j] * j`.

This gives us the recurrence relation: `f[i][j] = f[i-1][j-1] + f[i-1][j] * j`

The base case is `f[0][0] = 1` (one way to distribute zero candies into zero bags - do nothing), and all other `f[0][j]` and `f[i][0]` values are 0 (can't distribute candies into zero bags, or can't have positive bags with zero candies).

By building up from smaller subproblems to larger ones using [dynamic programming](/problems/dynamic_programming_intro), we can efficiently compute `f[n][k]`, which gives us the total number of ways to distribute `n` candies into `k` bags.

This approach works because each state `f[i][j]` only depends on previously computed states with fewer candies (`i-1`), allowing us to build the solution bottom-up.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

We implement the solution using a 2D [dynamic programming](/problems/dynamic_programming_intro) table `f` where `f[i][j]` represents the number of ways to distribute `i` candies into `j` bags.

**Initialization:**
- Create a 2D array `f` of size `(n+1) × (k+1)` initialized with zeros
- Set `f[0][0] = 1` as the base case (one way to distribute 0 candies into 0 bags)

**State Transition:**
For each candy `i` from `1` to `n` and each number of bags `j` from `1` to `k`, we apply the recurrence relation:

`f[i][j] = (f[i-1][j-1] + f[i-1][j] * j) % mod`

This formula combines:
- `f[i-1][j-1]`: Ways to place the `i`-th candy in a new bag
- `f[i-1][j] * j`: Ways to place the `i`-th candy in one of the `j` existing bags

**Implementation Details:**
```python
class Solution:
    def waysToDistribute(self, n: int, k: int) -> int:
        mod = 10**9 + 7
        f = [[0] * (k + 1) for _ in range(n + 1)]
        f[0][0] = 1
        for i in range(1, n + 1):
            for j in range(1, k + 1):
                f[i][j] = (f[i - 1][j] * j + f[i - 1][j - 1]) % mod
        return f[n][k]
```

The algorithm iterates through all states in a bottom-up manner, ensuring that when we compute `f[i][j]`, the required previous states `f[i-1][j-1]` and `f[i-1][j]` have already been computed.

**Time Complexity:** `O(n * k)` - We fill in a table of size `n × k`

**Space Complexity:** `O(n * k)` - For storing the DP table

The modulo operation `% (10^9 + 7)` is applied at each step to prevent integer overflow and return the result in the required format.

## Example Walkthrough

Let's walk through a small example with `n = 4` candies and `k = 2` bags to illustrate how the dynamic programming solution works.

We want to find the number of ways to distribute 4 distinct candies (labeled 1, 2, 3, 4) into exactly 2 non-empty bags.

**Step 1: Initialize the DP table**
We create a table `f[i][j]` where `i` ranges from 0 to 4 and `j` ranges from 0 to 2.

Initial state:
```
    j=0  j=1  j=2
i=0  1    0    0
i=1  0    ?    ?
i=2  0    ?    ?
i=3  0    ?    ?
i=4  0    ?    ?
```

**Step 2: Fill the table row by row**

For `i=1` (1 candy):
- `f[1][1] = f[0][0] + f[0][1] * 1 = 1 + 0 * 1 = 1`
  - One way: put candy 1 in bag 1: `{1}`
- `f[1][2] = f[0][1] + f[0][2] * 2 = 0 + 0 * 2 = 0`
  - Can't have 2 bags with only 1 candy

For `i=2` (2 candies):
- `f[2][1] = f[1][0] + f[1][1] * 1 = 0 + 1 * 1 = 1`
  - One way: both candies in same bag: `{1,2}`
- `f[2][2] = f[1][1] + f[1][2] * 2 = 1 + 0 * 2 = 1`
  - One way: each candy in separate bag: `{1}, {2}`

For `i=3` (3 candies):
- `f[3][1] = f[2][0] + f[2][1] * 1 = 0 + 1 * 1 = 1`
  - One way: all in same bag: `{1,2,3}`
- `f[3][2] = f[2][1] + f[2][2] * 2 = 1 + 1 * 2 = 3`
  - Three ways: `{1,2}, {3}` or `{1,3}, {2}` or `{1}, {2,3}`

For `i=4` (4 candies):
- `f[4][1] = f[3][0] + f[3][1] * 1 = 0 + 1 * 1 = 1`
  - One way: all in same bag: `{1,2,3,4}`
- `f[4][2] = f[3][1] + f[3][2] * 2 = 1 + 3 * 2 = 7`
  - Seven ways total:
    - From `f[3][1]`: candy 4 creates new bag from `{1,2,3}` → `{1,2,3}, {4}`
    - From `f[3][2] * 2`: candy 4 joins either existing bag:
      - `{1,2}, {3}` → `{1,2,4}, {3}` or `{1,2}, {3,4}`
      - `{1,3}, {2}` → `{1,3,4}, {2}` or `{1,3}, {2,4}`
      - `{1}, {2,3}` → `{1,4}, {2,3}` or `{1}, {2,3,4}`

**Final table:**
```
    j=0  j=1  j=2
i=0  1    0    0
i=1  0    1    0
i=2  0    1    1
i=3  0    1    3
i=4  0    1    7
```

**Answer:** `f[4][2] = 7`

The seven distinct distributions are:
1. `{1,2,3}, {4}`
2. `{1,2,4}, {3}`
3. `{1,2}, {3,4}`
4. `{1,3,4}, {2}`
5. `{1,3}, {2,4}`
6. `{1,4}, {2,3}`
7. `{1}, {2,3,4}`

This example demonstrates how the recurrence relation builds up the solution by considering whether each new candy creates a new bag or joins an existing one.

## Python Solution

```python
class Solution:
    def waysToDistribute(self, n: int, k: int) -> int:
        """
        Calculate the number of ways to distribute n distinct items into k non-empty groups.
        This implements Stirling numbers of the second kind.

        Args:
            n: Number of distinct items to distribute
            k: Number of non-empty groups

        Returns:
            Number of ways to distribute items modulo 10^9 + 7
        """
        MOD = 10**9 + 7

        # dp[i][j] represents the number of ways to distribute i items into j non-empty groups
        # Initialize dp table with dimensions (n+1) x (k+1)
        dp = [[0] * (k + 1) for _ in range(n + 1)]

        # Base case: 0 items into 0 groups has exactly 1 way (empty distribution)
        dp[0][0] = 1

        # Fill the dp table using the recurrence relation
        for items in range(1, n + 1):
            for groups in range(1, k + 1):
                # Two cases for distributing 'items' into 'groups':
                # 1. Add the new item to any existing group: dp[items-1][groups] * groups
                #    (we have 'groups' choices for which group to add the item to)
                # 2. Create a new group with just the new item: dp[items-1][groups-1]
                #    (we need to have distributed items-1 items into groups-1 groups first)
                dp[items][groups] = (dp[items - 1][groups] * groups +
                                     dp[items - 1][groups - 1]) % MOD

        return dp[n][k]
```

## Time and Space Complexity

The time complexity is `O(n × k)`, where `n` is the number of candies and `k` is the number of bags. This is because the code uses two nested loops: the outer loop iterates from 1 to `n` (n iterations), and the inner loop iterates from 1 to `k` (up to k iterations for each outer loop iteration). Each operation inside the nested loops takes constant time `O(1)`, resulting in a total time complexity of `O(n × k)`.

The space complexity is `O(n × k)`. The code creates a 2D array `f` with dimensions `(n + 1) × (k + 1)` to store the dynamic programming states. This array requires `O((n + 1) × (k + 1))` space, which simplifies to `O(n × k)`. The remaining variables (`mod`, `i`, `j`) only use constant space `O(1)`, so the overall space complexity is dominated by the 2D array, giving us `O(n × k)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Integer Overflow When Forgetting Modulo Operation**

**The Pitfall:**
A common mistake is applying the modulo operation only at the final return statement instead of during each calculation step. This can cause integer overflow for large values of `n` and `k`.

**Incorrect Implementation:**
```python
# WRONG - May cause overflow
dp[items][groups] = dp[items - 1][groups] * groups + dp[items - 1][groups - 1]
return dp[n][k] % MOD  # Modulo only at the end
```

**Why It's a Problem:**
The intermediate multiplication `dp[items - 1][groups] * groups` can become extremely large before the modulo is applied, potentially exceeding integer limits even in Python (though Python handles big integers, it affects performance).

**The Solution:**
Apply modulo operation immediately after each arithmetic operation:
```python
# CORRECT - Prevents overflow
dp[items][groups] = (dp[items - 1][groups] * groups +
                     dp[items - 1][groups - 1]) % MOD
```

### 2. **Incorrect Base Case Initialization**

**The Pitfall:**
Setting incorrect base cases, such as initializing `dp[1][1] = 1` or forgetting to handle edge cases properly.

**Common Mistakes:**
```python
# WRONG - Missing proper base case
dp = [[0] * (k + 1) for _ in range(n + 1)]
dp[1][1] = 1  # This alone is insufficient
```

**Why It's a Problem:**
The recurrence relation depends on `dp[i-1][j-1]` and `dp[i-1][j]`. Without `dp[0][0] = 1`, the entire table will remain zeros because the recurrence has no valid starting point.

**The Solution:**
Always set `dp[0][0] = 1` as the base case:
```python
# CORRECT
dp[0][0] = 1  # One way to distribute 0 items into 0 groups
```

### 3. **Off-by-One Errors in Loop Bounds**

**The Pitfall:**
Using incorrect loop ranges, especially starting from 0 instead of 1 for items and groups.

**Incorrect Implementation:**
```python
# WRONG - Starting from 0 causes issues
for items in range(0, n + 1):  # Should start from 1
    for groups in range(0, k + 1):  # Should start from 1
        dp[items][groups] = ...
```

**Why It's a Problem:**
- When `items = 0` and `groups > 0`: There's no way to distribute 0 items into positive number of groups
- When `groups = 0` and `items > 0`: There's no way to distribute positive items into 0 groups
- The recurrence relation references `groups - 1`, which would be negative if `groups = 0`

**The Solution:**
Start both loops from 1:
```python
# CORRECT
for items in range(1, n + 1):
    for groups in range(1, k + 1):
        dp[items][groups] = (dp[items - 1][groups] * groups +
                             dp[items - 1][groups - 1]) % MOD
```

### 4. **Not Handling Edge Cases Where k > n**

**The Pitfall:**
Not considering that when `k > n`, it's impossible to distribute `n` items into `k` non-empty groups.

**Why It's a Problem:**
The algorithm will correctly return 0 due to the DP logic, but it's inefficient to compute the entire table when we know the answer is 0.

**The Solution:**
Add an early return for this edge case:
```python
def waysToDistribute(self, n: int, k: int) -> int:
    if k > n:
        return 0  # Can't distribute n items into more than n non-empty groups
    if k == 0 or n == 0:
        return 1 if k == n else 0
    # ... rest of the implementation
```

