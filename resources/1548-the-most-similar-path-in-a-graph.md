# 1548. The Most Similar Path in a Graph

> Source: https://algo.monster/liteproblems/1548

## Problem Statement

You are given a [graph](/problems/graph_intro) with `n` cities numbered from 0 to n-1, connected by `m` bidirectional roads. Each road in `roads[i] = [ai, bi]` connects city `ai` with city `bi`. Every city has a name consisting of exactly three uppercase English letters, provided in the array `names`. The graph is fully connected, meaning you can reach any city from any other city.

Your task is to find a path through the [graph](/problems/graph_intro) that:
1. Has the **same length** as a given `targetPath` array
2. Has the **minimum edit distance** compared to `targetPath`

The edit distance between two paths is calculated as the number of positions where the city names differ. For example:
- If your path visits cities with names ["AAA", "BBB", "CCC"]
- And the target path is ["AAA", "BBB", "DDD"]
- The edit distance is 1 (only the third position differs)

The path you return must be valid, meaning there must be a direct road between consecutive cities in your path (`ans[i]` and `ans[i+1]` must be connected by a road).

You need to return an array containing the city indices (not names) that form the path with minimum edit distance. If multiple paths have the same minimum edit distance, you can return any one of them.

**Example:** If `targetPath = ["ATL", "PEK", "LAX"]` and your [graph](/problems/graph_intro) has cities with similar names connected appropriately, you need to find a sequence of 3 connected cities whose names most closely match this target sequence.

## Intuition

The key insight is that we need to find the best matching path of a specific length, which suggests using [dynamic programming](/problems/dynamic_programming_intro). Think of it like aligning two sequences - our actual path through the [graph](/problems/graph_intro) and the target path.

At each step along the target path, we need to decide which city to visit. The challenge is that we can't just greedily pick the city with the best name match at each position because:
1. We need to ensure cities are connected by roads (path validity)
2. A suboptimal choice early on might lead to better overall matching later

This naturally leads to a [dynamic programming](/problems/dynamic_programming_intro) formulation where we consider: "What's the minimum edit distance if we've matched the first `i` positions of the target path and we're currently at city `j`?"

For each position in the target path, we need to try all possible cities we could be at, but we can only reach a city from its neighbors. So for city `j` at position `i`, we look at all its neighbors `k` and check: what was the best edit distance to reach neighbor `k` at position `i-1`? Then we add the cost of being at city `j` for position `i` (which is 0 if names match, 1 if they don't).

The formula `f[i][j] = min(f[i-1][k] + cost)` for all neighbors `k` captures this idea - we're building up optimal solutions for longer paths based on optimal solutions for shorter paths.

To reconstruct the actual path (not just the minimum distance), we need to remember which predecessor city gave us the optimal value at each step. This is why we maintain a `pre` array to track the optimal previous city for each state, allowing us to backtrack from the end to build the complete path.

**Learn more about [Graph](/problems/graph_intro) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The implementation uses [dynamic programming](/problems/dynamic_programming_intro) with the following components:

**1. [Graph](/problems/graph_intro) Construction:**
First, we build an adjacency list `g` where `g[i]` contains all cities directly connected to city `i`:
```python
g = [[] for _ in range(n)]
for a, b in roads:
    g[a].append(b)
    g[b].append(a)
```

**2. DP Table Initialization:**
We create a 2D DP table `f[i][j]` where:
- `i` represents the position in the target path (0 to m-1)
- `j` represents the current city (0 to n-1)
- `f[i][j]` stores the minimum edit distance to match the first `i+1` elements of targetPath when ending at city `j`

We also maintain a predecessor table `pre[i][j]` to track which city we came from to achieve the optimal value at `f[i][j]`.

**3. Base Case:**
For the first position (i=0), we can start at any city. The cost is 0 if the city name matches `targetPath[0]`, otherwise 1:
```python
for j, s in enumerate(names):
    f[0][j] = targetPath[0] != s  # 0 if match, 1 if not
```

**4. State Transition:**
For each subsequent position `i` in the target path and each possible current city `j`, we check all neighbors `k` of city `j`:
```python
for i in range(1, m):
    for j in range(n):
        for k in g[j]:
            t = f[i-1][k] + (targetPath[i] != names[j])
            if t < f[i][j]:
                f[i][j] = t
                pre[i][j] = k
```

The transition formula is:
`f[i][j] = min(f[i-1][k] + cost)` where `k` is a neighbor of `j` and `cost = 1` if `names[j] != targetPath[i]`, otherwise `cost = 0`.

**5. Finding the Optimal End City:**
After filling the DP table, we find which city at the last position gives the minimum edit distance:
```python
k = 0
mi = inf
for j in range(n):
    if f[-1][j] < mi:
        mi = f[-1][j]
        k = j
```

**6. Path Reconstruction:**
Using the predecessor array, we backtrack from the optimal end city to reconstruct the entire path:
```python
ans = [0] * m
for i in range(m - 1, -1, -1):
    ans[i] = k
    k = pre[i][k]
```

The time complexity is `O(m × n × d)` where `m` is the length of targetPath, `n` is the number of cities, and `d` is the average degree of cities. The space complexity is `O(m × n)` for the DP and predecessor tables.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Given:**
- Cities: `[0, 1, 2, 3]` with names `["AAA", "BBB", "CCC", "AAA"]`
- Roads: `[[0,1], [1,2], [2,3], [1,3]]`
- Target Path: `["AAA", "CCC", "AAA"]`

**Graph Structure:**
```
    0(AAA)---1(BBB)---2(CCC)
              |         |
              +----3(AAA)+
```

**Step 1: Initialize DP Table**

For position 0 (matching "AAA"):
- `f[0][0] = 0` (city 0 has name "AAA" - matches!)
- `f[0][1] = 1` (city 1 has name "BBB" - no match)
- `f[0][2] = 1` (city 2 has name "CCC" - no match)
- `f[0][3] = 0` (city 3 has name "AAA" - matches!)

**Step 2: Fill position 1 (matching "CCC")**

For city 0 at position 1:
- Neighbors of 0: [1]
- From city 1: `f[0][1] + (targetPath[1] != "AAA") = 1 + 1 = 2`
- So `f[1][0] = 2`, `pre[1][0] = 1`

For city 1 at position 1:
- Neighbors of 1: [0, 2, 3]
- From city 0: `f[0][0] + (targetPath[1] != "BBB") = 0 + 1 = 1`
- From city 2: `f[0][2] + (targetPath[1] != "BBB") = 1 + 1 = 2`
- From city 3: `f[0][3] + (targetPath[1] != "BBB") = 0 + 1 = 1`
- Best is 1, so `f[1][1] = 1`, `pre[1][1] = 0` (or 3)

For city 2 at position 1:
- Neighbors of 2: [1, 3]
- From city 1: `f[0][1] + (targetPath[1] != "CCC") = 1 + 0 = 1`
- From city 3: `f[0][3] + (targetPath[1] != "CCC") = 0 + 0 = 0`
- Best is 0, so `f[1][2] = 0`, `pre[1][2] = 3`

For city 3 at position 1:
- Neighbors of 3: [1, 2]
- From city 1: `f[0][1] + (targetPath[1] != "AAA") = 1 + 1 = 2`
- From city 2: `f[0][2] + (targetPath[1] != "AAA") = 1 + 1 = 2`
- So `f[1][3] = 2`, `pre[1][3] = 1` (or 2)

**Step 3: Fill position 2 (matching "AAA")**

For city 0 at position 2:
- Neighbors of 0: [1]
- From city 1: `f[1][1] + (targetPath[2] != "AAA") = 1 + 0 = 1`
- So `f[2][0] = 1`, `pre[2][0] = 1`

For city 1 at position 2:
- Neighbors of 1: [0, 2, 3]
- From city 0: `f[1][0] + (targetPath[2] != "BBB") = 2 + 1 = 3`
- From city 2: `f[1][2] + (targetPath[2] != "BBB") = 0 + 1 = 1`
- From city 3: `f[1][3] + (targetPath[2] != "BBB") = 2 + 1 = 3`
- Best is 1, so `f[2][1] = 1`, `pre[2][1] = 2`

For city 2 at position 2:
- Neighbors of 2: [1, 3]
- From city 1: `f[1][1] + (targetPath[2] != "CCC") = 1 + 1 = 2`
- From city 3: `f[1][3] + (targetPath[2] != "CCC") = 2 + 1 = 3`
- So `f[2][2] = 2`, `pre[2][2] = 1`

For city 3 at position 2:
- Neighbors of 3: [1, 2]
- From city 1: `f[1][1] + (targetPath[2] != "AAA") = 1 + 0 = 1`
- From city 2: `f[1][2] + (targetPath[2] != "AAA") = 0 + 0 = 0`
- Best is 0, so `f[2][3] = 0`, `pre[2][3] = 2`

**Step 4: Find optimal end and reconstruct path**

Minimum at position 2: `f[2][3] = 0` (city 3 is best)

Backtracking:
- Position 2: city 3
- Position 1: `pre[2][3] = 2` → city 2
- Position 0: `pre[1][2] = 3` → city 3

**Final path: [3, 2, 3]** with edit distance 0 (perfect match: "AAA", "CCC", "AAA")

## Python Solution

```python
class Solution:
    def mostSimilar(
        self, n: int, roads: List[List[int]], names: List[str], targetPath: List[str]
    ) -> List[int]:
        # Build adjacency list for the graph
        adjacency_list = [[] for _ in range(n)]
        for city_a, city_b in roads:
            adjacency_list[city_a].append(city_b)
            adjacency_list[city_b].append(city_a)

        path_length = len(targetPath)

        # dp[i][j] = minimum edit distance to reach city j at position i in targetPath
        dp = [[float('inf')] * n for _ in range(path_length)]

        # predecessor[i][j] = previous city when reaching city j at position i with minimum cost
        predecessor = [[-1] * n for _ in range(path_length)]

        # Initialize first position: cost is 0 if name matches, 1 if it doesn't
        for city in range(n):
            dp[0][city] = 0 if names[city] == targetPath[0] else 1

        # Fill DP table for remaining positions
        for position in range(1, path_length):
            for current_city in range(n):
                # Check all neighbors of current_city
                for previous_city in adjacency_list[current_city]:
                    # Calculate cost: previous cost + mismatch penalty
                    mismatch_cost = 0 if names[current_city] == targetPath[position] else 1
                    total_cost = dp[position - 1][previous_city] + mismatch_cost

                    # Update if we found a better path
                    if total_cost < dp[position][current_city]:
                        dp[position][current_city] = total_cost
                        predecessor[position][current_city] = previous_city

        # Find the city with minimum cost at the last position
        min_cost = float('inf')
        best_last_city = 0
        for city in range(n):
            if dp[path_length - 1][city] < min_cost:
                min_cost = dp[path_length - 1][city]
                best_last_city = city

        # Reconstruct the path by backtracking through predecessors
        result_path = [0] * path_length
        current_city = best_last_city
        for position in range(path_length - 1, -1, -1):
            result_path[position] = current_city
            current_city = predecessor[position][current_city]

        return result_path
```

## Time and Space Complexity

**Time Complexity:** `O(m × n²)`

The algorithm uses dynamic programming where:
- The outer loop iterates through the targetPath of length `m` (from index 1 to m-1)
- For each position `i`, we iterate through all `n` cities (variable `j`)
- For each city `j`, we iterate through all its neighbors in the adjacency list `g[j]`

In the worst case, a city can be connected to all other `n-1` cities, making the innermost loop run up to `n-1` times. Therefore, the total time complexity is `O(m × n × n) = O(m × n²)`.

Additionally, building the adjacency list takes `O(E)` where E is the number of edges (roads), and the final path reconstruction takes `O(m)` time, but these don't affect the overall complexity.

**Space Complexity:** `O(m × n)`

The space is used for:
- The DP table `f`: a 2D array of size `m × n` storing minimum edit distances
- The predecessor table `pre`: a 2D array of size `m × n` storing the previous city in the optimal path
- The adjacency list `g`: takes `O(n + E)` space where E is the number of edges
- The answer array `ans`: takes `O(m)` space

The dominant space usage comes from the two 2D arrays (`f` and `pre`), each requiring `O(m × n)` space, resulting in an overall space complexity of `O(m × n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Graph Connectivity Assumption**
A critical pitfall is assuming the graph is always fully connected without validation. If the graph has disconnected components, the DP approach might fail to find valid paths or produce incorrect results.

**Problem Example:**
```python
# Disconnected graph scenario
n = 4
roads = [[0, 1], [2, 3]]  # Cities 0-1 are connected, 2-3 are connected, but no path from 0/1 to 2/3
names = ["AAA", "BBB", "CCC", "DDD"]
targetPath = ["AAA", "CCC"]  # Requires going from city 0 to city 2
```

**Solution:**
Add validation to ensure reachable paths exist:
```python
# After filling DP table, check if any valid path exists
valid_path_exists = False
for city in range(n):
    if dp[path_length - 1][city] < float('inf'):
        valid_path_exists = True
        break

if not valid_path_exists:
    # Handle the case - could return empty list or raise exception
    return []
```

### 2. **Predecessor Array Not Properly Initialized for Starting Cities**
The predecessor array for the first position remains -1, which can cause issues during path reconstruction if not handled carefully.

**Problem Example:**
When backtracking, accessing `predecessor[0][current_city]` returns -1, which might be used incorrectly if the reconstruction loop isn't properly bounded.

**Solution:**
Ensure the path reconstruction loop properly handles the first position:
```python
# Modified reconstruction that explicitly handles the boundary
result_path = [0] * path_length
current_city = best_last_city
for position in range(path_length - 1, 0, -1):  # Stop at position 1, not 0
    result_path[position] = current_city
    current_city = predecessor[position][current_city]
result_path[0] = current_city  # Set the first city
```

### 3. **Self-Loop Roads Breaking DP Logic**
If the input contains self-loops (a city connected to itself), the DP might incorrectly consider staying in the same city as a valid transition.

**Problem Example:**
```python
roads = [[0, 0], [0, 1], [1, 2]]  # City 0 has a self-loop
```

**Solution:**
Filter out self-loops when building the adjacency list:
```python
adjacency_list = [[] for _ in range(n)]
for city_a, city_b in roads:
    if city_a != city_b:  # Exclude self-loops
        adjacency_list[city_a].append(city_b)
        adjacency_list[city_b].append(city_a)
```

### 4. **Memory Optimization Overlooked**
For large inputs, the full DP table might consume excessive memory when only the previous row is needed for computation.

**Solution:**
Use rolling array optimization:
```python
# Instead of dp[path_length][n], use two arrays
prev_dp = [float('inf')] * n
curr_dp = [float('inf')] * n

# Initialize first position
for city in range(n):
    prev_dp[city] = 0 if names[city] == targetPath[0] else 1

# Process remaining positions
for position in range(1, path_length):
    curr_dp = [float('inf')] * n
    for current_city in range(n):
        for previous_city in adjacency_list[current_city]:
            mismatch_cost = 0 if names[current_city] == targetPath[position] else 1
            curr_dp[current_city] = min(curr_dp[current_city],
                                       prev_dp[previous_city] + mismatch_cost)
    prev_dp = curr_dp
```
Note: This optimization requires maintaining the full predecessor array for path reconstruction, so it only saves partial memory.

