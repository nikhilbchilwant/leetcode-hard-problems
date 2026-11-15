# 1595. Minimum Cost to Connect Two Groups of Points

> Source: https://algo.monster/liteproblems/1595

## Problem Statement

You have two groups of points. The first group contains `size₁` points and the second group contains `size₂` points, where `size₁ ≥ size₂`.

You're given a cost matrix of dimensions `size₁ × size₂` where `cost[i][j]` represents the cost to connect point `i` from the first group to point `j` from the second group.

Your task is to connect the two groups with the following requirements:
- **Every point in the first group** must be connected to at least one point in the second group
- **Every point in the second group** must be connected to at least one point in the first group

A single point can be connected to multiple points in the opposite group. The goal is to find the minimum total cost to satisfy these connection requirements.

For example, if you have:
- First group: 3 points (A, B, C)  
- Second group: 2 points (X, Y)
- Cost matrix showing connection costs between each pair

You need to ensure all points A, B, C are connected to at least one of {X, Y}, and both X and Y are connected to at least one of {A, B, C}. The connections you choose should minimize the total cost.

## Intuition

The key insight is recognizing this as an assignment problem with special constraints. Since every point must be connected to at least one point in the opposite group, we need to track which connections have been made.

Let's think about how to build the solution incrementally. As we process points from the first group one by one, we need to know which points in the second group have already been connected. This is where the [bitmask](/problems/bitmask_intro) comes in - we can use a binary number where each bit represents whether a point in the second group has been connected or not.

For example, if the second group has 3 points and our [bitmask](/problems/bitmask_intro) is `101`, it means points 0 and 2 are connected, but point 1 is not yet connected.

The [dynamic programming](/problems/dynamic_programming_intro) state `f[i][j]` represents: "What's the minimum cost to connect the first `i` points from group 1, such that the points in group 2 that are connected form the pattern `j` (as a [bitmask](/problems/bitmask_intro))?"

When we're at point `i` in the first group, we have choices:
1. We can connect it to a point `k` in the second group that wasn't connected before (flipping that bit in our mask)
2. We can connect it to a point `k` that was already connected (the mask doesn't change, but we still pay the connection cost)

The beauty of this approach is that by the time we reach `f[m][2^n - 1]` (where `m` is the size of group 1 and `2^n - 1` means all bits are set for group 2), we've ensured:
- All points in group 1 have been processed and connected
- All points in group 2 are connected (all bits in the mask are 1)

This guarantees both constraints are satisfied while tracking the minimum cost along the way.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) and [Bitmask](/problems/bitmask_intro) patterns.**

## Solution Approach

We implement a [dynamic programming](/problems/dynamic_programming_intro) solution with [bitmask](/problems/bitmask_intro) to track connections to the second group.

**State Definition:**
- `f[i][j]` = minimum cost to connect the first `i` points from group 1, where `j` is a [bitmask](/problems/bitmask_intro) representing which points in group 2 are connected
- `m` = size of first group, `n` = size of second group

**Initialization:**
```python
f = [[inf] * (1 << n) for _ in range(m + 1)]
f[0][0] = 0
```
- Create a 2D DP table with dimensions `(m+1) × 2^n`
- `f[0][0] = 0` means no points connected, zero cost
- All other states start with infinity

**State Transitions:**
For each point `i` in the first group (1 to m):
```python
for i in range(1, m + 1):
    for j in range(1 << n):  # iterate through all possible bitmasks
        for k in range(n):    # try connecting to each point in group 2
```

For each valid connection to point `k` in group 2 (when bit `k` is set in mask `j`):
```python
if (j >> k & 1) == 0:
    continue  # skip if bit k is not set in mask j
```

**Calculate minimum cost from three possible previous states:**
```python
c = cost[i - 1][k]  # cost to connect point i-1 to point k
x = min(
    f[i][j ^ (1 << k)],      # Case 1: point k wasn't connected before
    f[i - 1][j],              # Case 2: point k was already connected, keep it
    f[i - 1][j ^ (1 << k)]    # Case 3: point k wasn't connected, now connecting
) + c
f[i][j] = min(f[i][j], x)
```

The three cases represent:
1. **Same row, flip bit k**: We've already processed point `i`, but point `k` in group 2 wasn't connected
2. **Previous row, same mask**: Point `k` was already connected by previous points
3. **Previous row, flip bit k**: Point `k` gets its first connection from point `i`

**Final Answer:**
```python
return f[m][-1]  # f[m][2^n - 1]
```
Returns the minimum cost when all `m` points from group 1 are processed and all `n` points from group 2 are connected (all bits set in the mask).

The algorithm ensures both constraints are met: every point in group 1 is connected (we process all of them), and every point in group 2 is connected (final mask has all bits set).

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Example Setup:**
- First group: 3 points (indexed 0, 1, 2)
- Second group: 2 points (indexed 0, 1)
- Cost matrix:
```
cost = [[1, 4],
        [3, 2],
        [5, 1]]
```
This means:
- Connecting point 0 (group 1) to point 0 (group 2) costs 1
- Connecting point 0 (group 1) to point 1 (group 2) costs 4
- And so on...

**DP Table Initialization:**
We create `f[4][4]` since we have 3 points in group 1 (need indices 0-3) and 2^2=4 possible bitmasks for group 2.
- Bitmask `00` (decimal 0): no points in group 2 connected
- Bitmask `01` (decimal 1): only point 0 in group 2 connected
- Bitmask `10` (decimal 2): only point 1 in group 2 connected  
- Bitmask `11` (decimal 3): both points in group 2 connected

Initial state: `f[0][0] = 0`, all others = infinity

**Processing Point 0 from Group 1:**

When `i=1` (processing first point), we consider connecting it to points in group 2:

For mask `j=1` (binary `01` - point 0 in group 2 is connected):
- We can connect point 0 (group 1) to point 0 (group 2), cost = 1
- Previous state: `f[0][0] = 0` (no connections before)
- New state: `f[1][1] = 0 + 1 = 1`

For mask `j=2` (binary `10` - point 1 in group 2 is connected):
- We can connect point 0 (group 1) to point 1 (group 2), cost = 4
- Previous state: `f[0][0] = 0`
- New state: `f[1][2] = 0 + 4 = 4`

For mask `j=3` (binary `11` - both points in group 2 connected):
- Option 1: Connect to point 0 (cost 1), coming from `f[0][2]` or `f[1][2]`
- Option 2: Connect to point 1 (cost 4), coming from `f[0][1]` or `f[1][1]`
- Best: `f[1][3] = min(f[1][2] + 1, f[1][1] + 4) = min(4+1, 1+4) = 5`

**Processing Point 1 from Group 1:**

When `i=2`, key transitions:

For mask `j=3` (both points connected):
- Connect point 1 to point 0 (cost 3):
  - From `f[1][3] = 5` (point 0 already connected): total = 5 + 3 = 8
  - From `f[1][2] = 4` (connecting point 0 for first time): total = 4 + 3 = 7
- Connect point 1 to point 1 (cost 2):
  - From `f[1][3] = 5` (point 1 already connected): total = 5 + 2 = 7
  - From `f[1][1] = 1` (connecting point 1 for first time): total = 1 + 2 = 3
- Best: `f[2][3] = 3`

**Processing Point 2 from Group 1:**

When `i=3`, for the final mask `j=3`:
- Connect point 2 to point 0 (cost 5): best path gives 3 + 5 = 8
- Connect point 2 to point 1 (cost 1): best path gives 3 + 1 = 4
- Best: `f[3][3] = 4`

**Final Answer:** `f[3][3] = 4`

This corresponds to the optimal solution:
- Point 0 (group 1) connects to point 0 (group 2): cost 1
- Point 1 (group 1) connects to point 1 (group 2): cost 2  
- Point 2 (group 1) connects to point 1 (group 2): cost 1
- Total cost: 1 + 2 + 1 = 4

All points in both groups are connected, satisfying our constraints with minimum total cost.

## Python Solution

```python
class Solution:
    def connectTwoGroups(self, cost: List[List[int]]) -> int:
        # Get dimensions of the cost matrix
        group1_size, group2_size = len(cost), len(cost[0])
      
        # Initialize DP table
        # dp[i][mask] = minimum cost to connect first i points from group1
        # with points from group2 represented by mask
        dp = [[float('inf')] * (1 << group2_size) for _ in range(group1_size + 1)]
      
        # Base case: no points from group1 connected to empty set costs 0
        dp[0][0] = 0
      
        # Iterate through each point in group1
        for i in range(1, group1_size + 1):
            # Iterate through all possible subsets of group2 (represented as bitmask)
            for mask in range(1 << group2_size):
                # Try connecting current point from group1 to each point in group2
                for j in range(group2_size):
                    # Check if j-th point of group2 is in the current mask
                    if (mask >> j & 1) == 0:
                        continue
                  
                    # Cost of connecting (i-1)th point of group1 to j-th point of group2
                    connection_cost = cost[i - 1][j]
                  
                    # Calculate minimum cost from three possible transitions:
                    # 1. Current point reuses existing connection (dp[i][mask ^ (1 << j)])
                    # 2. Previous points already covered this mask (dp[i - 1][mask])
                    # 3. Add new connection from previous state (dp[i - 1][mask ^ (1 << j)])
                    min_previous_cost = min(
                        dp[i][mask ^ (1 << j)],  # Remove j from mask for current row
                        dp[i - 1][mask],          # All connections already made by previous points
                        dp[i - 1][mask ^ (1 << j)]  # Add connection from previous row
                    )
                  
                    # Update minimum cost for current state
                    dp[i][mask] = min(dp[i][mask], min_previous_cost + connection_cost)
      
        # Return minimum cost when all points from both groups are connected
        # (all bits set in the mask)
        return dp[group1_size][(1 << group2_size) - 1]
```

## Time and Space Complexity

**Time Complexity:** `O(m * 2^n * n)`

The algorithm uses dynamic programming with bitmask to track which points in the second group have been connected. There are three nested loops:
- The outer loop iterates through `m` points in the first group: `O(m)`
- The middle loop iterates through all possible bitmask states for the second group: `O(2^n)`
- The inner loop iterates through each point in the second group: `O(n)`

For each combination of `(i, j, k)`, we perform constant time operations to calculate the minimum cost. Therefore, the overall time complexity is `O(m * 2^n * n)`.

**Space Complexity:** `O(m * 2^n)`

The algorithm uses a 2D DP table `f` with dimensions `(m + 1) × 2^n`. Each cell stores a single integer value representing the minimum cost for a specific state. The space required is:
- DP table `f`: `(m + 1) * 2^n` entries
- Other variables use constant space

Therefore, the overall space complexity is `O(m * 2^n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Base Case Initialization**
A common mistake is initializing `dp[0][mask] = 0` for all masks, not just mask 0. This would incorrectly suggest that points from group 2 can be connected with zero cost when no points from group 1 have been processed.

**Wrong:**
```python
dp = [[0] * (1 << group2_size) for _ in range(group1_size + 1)]
```

**Correct:**
```python
dp = [[float('inf')] * (1 << group2_size) for _ in range(group1_size + 1)]
dp[0][0] = 0
```

### 2. **Missing State Transitions**
Developers often forget one of the three critical transitions, particularly the case where the current point in group 1 connects to an already-connected point in group 2. This leads to suboptimal solutions.

**Incomplete transition (missing reuse case):**
```python
min_previous_cost = min(
    dp[i - 1][mask],
    dp[i - 1][mask ^ (1 << j)]
)
```

**Complete transition:**
```python
min_previous_cost = min(
    dp[i][mask ^ (1 << j)],      # Current row, different mask
    dp[i - 1][mask],              # Previous row, same mask
    dp[i - 1][mask ^ (1 << j)]    # Previous row, different mask
)
```

### 3. **Incorrect Bit Manipulation**
Using wrong bit operations or checking bit positions incorrectly is a frequent error.

**Wrong bit check:**
```python
if mask & (1 << j) == 0:  # This checks if bit j is NOT set
    continue
```

**Correct for this algorithm:**
```python
if (mask >> j & 1) == 0:  # Skip if bit j is not set
    continue
```

### 4. **Off-by-One Errors in Indexing**
Since the DP table uses 1-based indexing for group 1 points but the cost matrix uses 0-based indexing, it's easy to mix them up.

**Wrong:**
```python
connection_cost = cost[i][j]  # Using i directly
```

**Correct:**
```python
connection_cost = cost[i - 1][j]  # Adjust for 0-based cost matrix
```

### 5. **Incorrect Final Answer Extraction**
Some implementations might check all possible masks instead of just the full mask where all group 2 points are connected.

**Wrong:**
```python
return min(dp[group1_size])  # This doesn't ensure all group2 points are connected
```

**Correct:**
```python
return dp[group1_size][(1 << group2_size) - 1]  # Ensures all bits are set
```

### 6. **Memory Optimization Confusion**
When trying to optimize space by using only two rows, developers might incorrectly handle the transition `dp[i][mask ^ (1 << j)]` which requires the current row.

**Solution:** If implementing space optimization, process masks in a specific order or use a temporary array to avoid overwriting needed values:
```python
# Process masks in decreasing order when space-optimizing
for mask in range((1 << group2_size) - 1, -1, -1):
    # transitions here
```

