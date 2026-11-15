# 1857. Largest Color Value in a Directed Graph

> Source: https://algo.monster/liteproblems/1857

## Problem Statement

You have a directed [graph](/problems/graph_intro) with `n` nodes (numbered from `0` to `n-1`) and `m` edges. Each node has a color represented by a lowercase English letter, given in the string `colors` where `colors[i]` is the color of node `i`.

The [graph](/problems/graph_intro) is defined by a 2D array `edges` where each `edges[j] = [aj, bj]` represents a directed edge from node `aj` to node `bj`.

A valid path in the [graph](/problems/graph_intro) is a sequence of nodes `x1 -> x2 -> x3 -> ... -> xk` where there exists a directed edge from each node to the next one in the sequence.

For any valid path, the **color value** is calculated as the maximum frequency of any single color among all nodes in that path. For example, if a path contains nodes with colors "a", "b", "a", "a", "c", then the color value would be `3` (since "a" appears 3 times, which is the maximum).

Your task is to find the **largest color value** among all possible valid paths in the [graph](/problems/graph_intro).

If the [graph](/problems/graph_intro) contains a cycle, return `-1` since a cycle would allow infinite traversal and make the color value unbounded.

**Example:** If you have a path with nodes colored "r", "r", "b", "r", the color "r" appears 3 times and "b" appears 1 time, so the color value of this path is `3`.

## Intuition

The key insight is that we need to track the maximum count of each color along all paths ending at each node. Since we want the largest color value across all valid paths, we need to explore paths systematically while avoiding cycles.

Think of this problem as asking: "For every possible path to reach a node, what's the maximum count of each color we can accumulate?" This naturally leads to a [dynamic programming](/problems/dynamic_programming_intro) approach where we build up solutions from shorter paths to longer ones.

Why [topological sort](/problems/topo_intro)? In a directed acyclic [graph](/problems/graph_intro) (DAG), topological sort gives us an ordering where we process nodes only after all their predecessors have been processed. This ensures that when we reach a node, we already know the best color counts from all paths leading to it.

The cycle detection comes naturally from [topological sort](/problems/topo_intro) - if we can't process all `n` nodes through topological sort, it means there's a cycle (some nodes have dependencies that can never be resolved).

For the [dynamic programming](/problems/dynamic_programming_intro) part, we maintain a 2D array `dp[node][color]` representing the maximum count of each color on any path ending at that node. When we process a node, we update all its neighbors by:
1. Taking the color counts we've accumulated so far (from the current node)
2. Adding 1 if the neighbor's color matches the color we're tracking
3. Keeping the maximum value if multiple paths lead to the same neighbor

This way, we systematically build up the answer by processing nodes in topological order, ensuring we always have complete information about all incoming paths before processing a node. The final answer is the maximum value in the entire `dp` array.

**Learn more about [Graph](/problems/graph_intro), [Topological Sort](/problems/topo_intro), [Memoization](/problems/memoization_intro) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution uses **[Topological Sort](/problems/topo_intro) with [Dynamic Programming](/problems/dynamic_programming_intro)** to solve this problem efficiently.

**Step 1: Build the [Graph](/problems/graph_intro) and Calculate In-degrees**
- Create an adjacency list `g` to represent the directed graph
- Track the in-degree of each node in array `indeg`
- For each edge `[a, b]`, add `b` to `g[a]`'s list and increment `indeg[b]`

**Step 2: Initialize [Topological Sort](/problems/topo_intro)**
- Create a queue `q` and add all nodes with in-degree 0 (nodes with no incoming edges)
- Initialize a 2D DP array `dp[n][26]` where `dp[i][j]` represents the maximum count of color `j` on any path ending at node `i`
- For each starting node (in-degree 0), set `dp[i][c] = 1` where `c` is that node's color

**Step 3: Process Nodes in Topological Order**
- Use BFS to process nodes level by level
- For each node `i` dequeued:
  - Increment counter `cnt` (tracks processed nodes for cycle detection)
  - For each neighbor `j` of node `i`:
    - Decrement `indeg[j]`
    - If `indeg[j]` becomes 0, add `j` to the queue
    - Update the DP values: `dp[j][k] = max(dp[j][k], dp[i][k] + (c == k))`
      - This means: take the maximum between the current value and the value from node `i`
      - Add 1 if color `k` matches node `j`'s color `c`
    - Track the maximum value seen so far in `ans`

**Step 4: Check for Cycles and Return Result**
- If `cnt < n`, not all nodes were processed, indicating a cycle exists - return `-1`
- Otherwise, return `ans` which holds the largest color value found

The algorithm runs in `O(V + E)` time for the [topological sort](/problems/topo_intro) plus `O(V * 26)` for the DP updates, giving overall `O(V * 26 + E * 26)` time complexity, where `V` is the number of nodes and `E` is the number of edges.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Example Graph:**
- `colors = "abaca"` (5 nodes with colors: 0→'a', 1→'b', 2→'a', 3→'c', 4→'a')
- `edges = [[0,1], [0,2], [2,3], [3,4]]`

This creates the graph:
```
    0(a) → 1(b)
     ↓
    2(a) → 3(c) → 4(a)
```

**Step 1: Build Graph and Calculate In-degrees**
- Adjacency list: `g[0] = [1,2]`, `g[2] = [3]`, `g[3] = [4]`
- In-degrees: `indeg = [0, 1, 1, 1, 1]` (node 0 has no incoming edges)

**Step 2: Initialize Topological Sort**
- Queue `q = [0]` (only node 0 has in-degree 0)
- Initialize DP array (showing only non-zero values):
  - `dp[0][0] = 1` (node 0 has color 'a', which is index 0)

**Step 3: Process Nodes in Topological Order**

*Processing node 0:*
- Remove node 0 from queue, `cnt = 1`
- Process neighbor 1:
  - `indeg[1] = 0`, add to queue
  - Update DP: `dp[1][0] = max(0, 1+0) = 1` (inherits 'a' count)
  - `dp[1][1] = max(0, 0+1) = 1` (node 1 is 'b')
- Process neighbor 2:
  - `indeg[2] = 0`, add to queue
  - Update DP: `dp[2][0] = max(0, 1+1) = 2` (node 2 is also 'a')
- Current max `ans = 2`

*Processing node 1:*
- Remove node 1 from queue, `cnt = 2`
- No neighbors, move on

*Processing node 2:*
- Remove node 2 from queue, `cnt = 3`
- Process neighbor 3:
  - `indeg[3] = 0`, add to queue
  - Update DP: `dp[3][0] = max(0, 2+0) = 2` (inherits 'a' count)
  - `dp[3][2] = max(0, 0+1) = 1` (node 3 is 'c')
- Current max `ans = 2`

*Processing node 3:*
- Remove node 3 from queue, `cnt = 4`
- Process neighbor 4:
  - `indeg[4] = 0`, add to queue
  - Update DP: `dp[4][0] = max(0, 2+1) = 3` (node 4 is 'a', adds to count)
  - `dp[4][2] = max(0, 1+0) = 1` (inherits 'c' count)
- Current max `ans = 3`

*Processing node 4:*
- Remove node 4 from queue, `cnt = 5`
- No neighbors, done

**Step 4: Check for Cycles and Return Result**
- `cnt = 5 = n`, so no cycle exists
- Return `ans = 3`

The answer is 3, which comes from the path 0→2→3→4 where color 'a' appears 3 times (at nodes 0, 2, and 4).

## Python Solution

```python
from collections import defaultdict, deque
from typing import List

class Solution:
    def largestPathValue(self, colors: str, edges: List[List[int]]) -> int:
        # Number of nodes in the graph
        num_nodes = len(colors)
      
        # Track in-degree for each node (for topological sort)
        in_degree = [0] * num_nodes
      
        # Build adjacency list representation of the graph
        adjacency_list = defaultdict(list)
        for source, destination in edges:
            adjacency_list[source].append(destination)
            in_degree[destination] += 1
      
        # Initialize queue with nodes having zero in-degree (starting nodes)
        queue = deque()
      
        # DP table: dp[node][color] = max count of 'color' ending at 'node'
        # 26 represents the 26 lowercase letters
        color_count_dp = [[0] * 26 for _ in range(num_nodes)]
      
        # Add all nodes with zero in-degree to queue and initialize their color count
        for node_id, degree in enumerate(in_degree):
            if degree == 0:
                queue.append(node_id)
                # Convert character to index (0-25) and increment count for this node's color
                color_index = ord(colors[node_id]) - ord('a')
                color_count_dp[node_id][color_index] += 1
      
        # Track number of processed nodes (to detect cycles)
        processed_nodes = 0
        # Track the maximum color count found so far
        max_color_count = 1
      
        # Process nodes in topological order
        while queue:
            current_node = queue.popleft()
            processed_nodes += 1
          
            # Process all neighbors of current node
            for neighbor in adjacency_list[current_node]:
                in_degree[neighbor] -= 1
              
                # If all dependencies processed, add to queue
                if in_degree[neighbor] == 0:
                    queue.append(neighbor)
              
                # Get the color index of the neighbor node
                neighbor_color = ord(colors[neighbor]) - ord('a')
              
                # Update DP values for the neighbor
                for color_idx in range(26):
                    # If current color matches neighbor's color, add 1; otherwise, just propagate
                    increment = 1 if color_idx == neighbor_color else 0
                    color_count_dp[neighbor][color_idx] = max(
                        color_count_dp[neighbor][color_idx],
                        color_count_dp[current_node][color_idx] + increment
                    )
                    # Update global maximum
                    max_color_count = max(max_color_count, color_count_dp[neighbor][color_idx])
      
        # If not all nodes were processed, there's a cycle
        if processed_nodes < num_nodes:
            return -1
      
        return max_color_count
```

## Time and Space Complexity

**Time Complexity:** `O((n + m) × 26)` or `O((n + m) × |Σ|)`

The algorithm uses topological sorting with dynamic programming. Breaking down the time complexity:
- Building the graph and calculating in-degrees takes `O(m)` time where `m` is the number of edges
- The BFS traversal visits each node exactly once, taking `O(n)` time
- For each node processed, we iterate through all its outgoing edges and update the DP table with 26 colors, which takes `O(degree(node) × 26)` time
- The total work across all nodes is `O(m × 26)` since each edge is processed once with 26 color updates
- Overall: `O(n + m × 26)` = `O((n + m) × 26)`

**Space Complexity:** `O(m + n × 26)` or `O(m + n × |Σ|)`

The space usage consists of:
- Graph adjacency list `g`: `O(m)` to store all edges
- In-degree array `indeg`: `O(n)`
- Queue `q`: `O(n)` in worst case
- DP table `dp`: `O(n × 26)` for storing the maximum count of each color for each node
- Overall: `O(m + n × 26)`

Where `n` is the number of nodes, `m` is the number of edges, and `|Σ| = 26` represents the size of the alphabet (26 lowercase letters).

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect DP Initialization for Starting Nodes**

**The Pitfall:** A common mistake is forgetting to initialize the DP values for nodes with zero in-degree (starting nodes). If you don't set `dp[node][color] = 1` for the starting node's own color, the algorithm will incorrectly calculate all path color values as 0.

**Why It Happens:** When processing nodes in topological order, the DP propagation relies on having correct base values. Starting nodes have no predecessors, so their initial state must be manually set.

**Solution:** Always initialize the color count for starting nodes:
```python
for node_id, degree in enumerate(in_degree):
    if degree == 0:
        queue.append(node_id)
        color_index = ord(colors[node_id]) - ord('a')
        color_count_dp[node_id][color_index] = 1  # Don't forget this!
```

### 2. **Updating DP Values Before Checking In-Degree**

**The Pitfall:** Updating the DP values for a neighbor node before verifying that all its dependencies have been processed (i.e., before its in-degree reaches 0) can lead to incorrect results.

**Why It Happens:** In topological sort with DP, a node's final DP value should only be computed after all possible paths leading to it have been considered. Premature updates may miss contributions from unprocessed predecessors.

**Solution:** Update DP values for all edges, but only add nodes to the queue when their in-degree reaches zero:
```python
for neighbor in adjacency_list[current_node]:
    in_degree[neighbor] -= 1
  
    # Update DP regardless of in-degree
    for color_idx in range(26):
        increment = 1 if color_idx == neighbor_color else 0
        color_count_dp[neighbor][color_idx] = max(
            color_count_dp[neighbor][color_idx],
            color_count_dp[current_node][color_idx] + increment
        )
  
    # Only add to queue when all dependencies are processed
    if in_degree[neighbor] == 0:
        queue.append(neighbor)
```

### 3. **Not Tracking Maximum During DP Updates**

**The Pitfall:** Computing the maximum color value only at the end by scanning the entire DP table can miss intermediate values for nodes that aren't leaf nodes in the DAG.

**Why It Happens:** The maximum color value might occur at an internal node rather than a terminal node. If you only check the final DP state, you might miss the actual maximum that occurred during processing.

**Solution:** Track the maximum value during each DP update:
```python
max_color_count = max(max_color_count, color_count_dp[neighbor][color_idx])
```

### 4. **Incorrect Cycle Detection**

**The Pitfall:** Using the wrong condition for cycle detection, such as checking if the queue is empty instead of counting processed nodes.

**Why It Happens:** An empty queue doesn't necessarily mean all nodes were processed—it could mean some nodes formed a cycle and never had their in-degree reduced to zero.

**Solution:** Count the number of processed nodes and compare with total nodes:
```python
if processed_nodes < num_nodes:
    return -1  # Cycle detected
```

