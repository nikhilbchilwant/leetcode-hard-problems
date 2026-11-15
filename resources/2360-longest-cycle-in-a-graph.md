# 2360. Longest Cycle in a Graph

> Source: https://algo.monster/liteproblems/2360

## Problem Statement

You have a directed [graph](/problems/graph_intro) with `n` nodes labeled from `0` to `n - 1`. Each node can have at most one outgoing edge (meaning each node points to at most one other node).

The [graph](/problems/graph_intro) is given as an array `edges` of size `n`, where:
- `edges[i]` represents the node that node `i` points to
- If node `i` has no outgoing edge, then `edges[i] = -1`

Your task is to find the length of the longest cycle in this [graph](/problems/graph_intro). A cycle is a path that starts at some node and eventually returns to that same starting node. If no cycle exists in the graph, return `-1`.

For example:
- If `edges = [3, 3, 4, 2, 3]`, node 0 points to node 3, node 1 points to node 3, node 2 points to node 4, node 3 points to node 2, and node 4 points to node 3. There's a cycle: 2 → 4 → 3 → 2, which has length 3.
- If `edges = [2, -1, 3, 1]`, node 0 points to 2, node 1 has no outgoing edge, node 2 points to 3, and node 3 points to 1. There's no cycle in this [graph](/problems/graph_intro), so the answer would be -1.

## Intuition

Since each node has at most one outgoing edge, following edges from any starting node creates a single path (no branching). This path will eventually either:
1. Hit a dead end (node with no outgoing edge)
2. Enter a cycle

The key insight is that if we keep track of nodes in our current path, we can detect when we revisit a node - that's when we've found a cycle. The cycle length is the number of nodes from where we first encountered the repeated node until we reached it again.

Consider what happens when we traverse from a node:
- We build a path by following edges: `node → edges[node] → edges[edges[node]] → ...`
- We mark each visited node to avoid reprocessing
- If we reach a node we've already seen in our current path, we've found a cycle
- The cycle doesn't include all nodes in our path - only those from the repeated node onwards

For example, if our path is `1 → 2 → 3 → 4 → 2`, the cycle is `2 → 3 → 4 → 2` with length 3, not the entire path.

Why do we need to try every unvisited node as a starting point? Because the [graph](/problems/graph_intro) might have multiple disconnected components, and cycles could exist in any of them. A node might lead to a cycle even if it's not part of the cycle itself.

The algorithm naturally emerges:
1. For each unvisited node, start a new path traversal
2. Follow edges while recording the path
3. If we revisit a node from our current path, calculate the cycle length
4. Mark all traversed nodes as visited to avoid redundant work
5. Track the maximum cycle length found

This approach ensures we find all cycles in the [graph](/problems/graph_intro) with just one pass through all nodes, making it efficient with `O(n)` time complexity.

**Learn more about [Depth-First Search](/problems/dfs_intro), [Breadth-First Search](/problems/bfs_intro), [Graph](/problems/graph_intro) and [Topological Sort](/problems/topo_intro) patterns.**

## Solution Approach

The implementation uses a straightforward traversal approach where we visit each node as a potential starting point for finding cycles.

**Data Structures Used:**
- `vis`: A boolean array of size `n` to track visited nodes, preventing redundant traversals
- `cycle`: A list to store the current path being explored from each starting node

**Algorithm Steps:**

1. **Initialize tracking variables:**
   - Create a visited array `vis` of size `n`, all set to `False`
   - Initialize `ans = -1` to store the maximum cycle length

2. **Iterate through all nodes as potential starting points:**
   ```python
   for i in range(n):
       if vis[i]:
           continue  # Skip already visited nodes
   ```

3. **For each unvisited starting node, build a path:**
   - Start from node `i` and follow edges using variable `j`
   - Build the path in `cycle` list while marking nodes as visited
   ```python
   j = i
   cycle = []
   while j != -1 and not vis[j]:
       vis[j] = True
       cycle.append(j)
       j = edges[j]  # Move to next node
   ```

4. **Detect and measure cycles:**
   - After the loop, `j` will be either:
     - `-1`: Path ended at a dead node (no cycle)
     - A visited node: Either part of current path (cycle) or from previous traversal
 
   - If `j != -1`, check if it's in the current path:
   ```python
   k = next((k for k in range(m) if cycle[k] == j), inf)
   ```
   - If `j` is found in `cycle` at index `k`, then nodes from index `k` to the end form a cycle
   - Cycle length = `m - k` where `m = len(cycle)`

5. **Update maximum cycle length:**
   ```python
   ans = max(ans, m - k)
   ```

**Example Walkthrough:**

Consider `edges = [3, 3, 4, 2, 3]`:
- Starting from node 0: Path is `0 → 3 → 2 → 4 → 3`. Node 3 is revisited.
  - `cycle = [0, 3, 2, 4]`, `j = 3`
  - Find 3 in cycle at index 1
  - Cycle length = 4 - 1 = 3 (the cycle is `3 → 2 → 4 → 3`)

The algorithm efficiently handles all cases:
- Multiple disconnected components
- Nodes leading to cycles without being part of them
- Dead-end paths
- Already processed nodes from previous traversals

Time complexity: `O(n)` - each node is visited at most once
Space complexity: `O(n)` - for the visited array and cycle list

## Example Walkthrough

Let's walk through the algorithm with `edges = [1, 2, 0, 4, 5, 3]`.

This graph looks like:
- 0 → 1 → 2 → 0 (forms a cycle of length 3)
- 3 → 4 → 5 → 3 (forms a cycle of length 3)

**Step-by-step execution:**

**Iteration 1: Start from node 0**
- `vis = [False, False, False, False, False, False]`
- Build path: 0 → 1 → 2 → 0
- `cycle = [0, 1, 2]`, then `j = edges[2] = 0`
- Node 0 is already visited (it's in our current path)
- Find 0 in cycle: it's at index 0
- Cycle length = 3 - 0 = 3
- Update `ans = 3`
- Mark nodes visited: `vis = [True, True, True, False, False, False]`

**Iteration 2: Start from node 1**
- Skip (already visited)

**Iteration 3: Start from node 2**
- Skip (already visited)

**Iteration 4: Start from node 3**
- `vis = [True, True, True, False, False, False]`
- Build path: 3 → 4 → 5 → 3
- `cycle = [3, 4, 5]`, then `j = edges[5] = 3`
- Node 3 is already visited (it's in our current path)
- Find 3 in cycle: it's at index 0
- Cycle length = 3 - 0 = 3
- `ans = max(3, 3) = 3`
- Mark nodes visited: `vis = [True, True, True, True, True, True]`

**Iterations 5-6: Start from nodes 4 and 5**
- Skip (already visited)

**Result:** Maximum cycle length = 3

The algorithm correctly identifies both cycles in the graph with just one pass through all nodes, demonstrating its O(n) efficiency.

## Python Solution

```python
class Solution:
    def longestCycle(self, edges: List[int]) -> int:
        n = len(edges)
        visited = [False] * n
        max_cycle_length = -1
      
        # Try starting from each unvisited node
        for start_node in range(n):
            if visited[start_node]:
                continue
          
            # Track the path from current starting node
            current_node = start_node
            path = []
          
            # Follow edges until we hit a visited node or dead end
            while current_node != -1 and not visited[current_node]:
                visited[current_node] = True
                path.append(current_node)
                current_node = edges[current_node]
          
            # If we hit a dead end (no outgoing edge), skip
            if current_node == -1:
                continue
          
            # Check if the node we stopped at is part of current path
            # If yes, we found a cycle
            path_length = len(path)
          
            # Find the index of current_node in path
            # If current_node is in path, we have a cycle
            cycle_start_index = float('inf')
            for index in range(path_length):
                if path[index] == current_node:
                    cycle_start_index = index
                    break
          
            # Update max cycle length if we found a cycle
            cycle_length = path_length - cycle_start_index
            max_cycle_length = max(max_cycle_length, cycle_length)
      
        return max_cycle_length
```

## Time and Space Complexity

**Time Complexity:** `O(n)`

The algorithm visits each node at most once throughout the entire execution. In the outer loop, we iterate through all `n` nodes. For each unvisited node, we start a traversal that follows edges until we either hit a visited node or reach a dead end (`-1`). Since each node is marked as visited when first encountered and never revisited, the total number of edge traversals across all iterations is bounded by `n`. The inner operations, including the list comprehension to find the index of `j` in the cycle, run in `O(m)` where `m` is the length of the current path, but since the sum of all path lengths cannot exceed `n`, the overall time complexity remains `O(n)`.

**Space Complexity:** `O(n)`

The algorithm uses additional space for:
- The `vis` array of size `n` to track visited nodes: `O(n)`
- The `cycle` list that stores nodes in the current path, which in the worst case can contain up to `n` nodes: `O(n)`
- A few integer variables (`i`, `j`, `k`, `m`, `ans`): `O(1)`

Therefore, the total space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Incorrectly Identifying Cycles
**The Problem:** A common mistake is assuming that whenever we encounter a visited node, we've found a cycle. This is incorrect because the visited node might have been visited in a previous traversal from a different starting point, not in the current path.

**Example of the Issue:**
```python
# INCORRECT approach
for start in range(n):
    if visited[start]:
        continue
    current = start
    length = 0
    while current != -1 and not visited[current]:
        visited[current] = True
        length += 1
        current = edges[current]
  
    # WRONG: Assuming any visited node means a cycle
    if current != -1:  # current is visited
        max_cycle = max(max_cycle, length)  # This is wrong!
```

Consider `edges = [1, 2, -1, 4, 5, 3]`:
- Starting from node 0: `0 → 1 → 2 → -1` (no cycle)
- Starting from node 3: `3 → 4 → 5 → 3` (cycle of length 3)
- But if we start from node 4 after processing node 0, we'd see `4 → 5 → 3` (already visited from previous traversal), and incorrectly count this as a cycle of length 3.

**The Solution:** Always check if the visited node is part of the CURRENT path, not just any visited node:
```python
# CORRECT approach
path = []
while current != -1 and not visited[current]:
    visited[current] = True
    path.append(current)
    current = edges[current]

# Check if current is in the current path
if current != -1 and current in path:
    cycle_start = path.index(current)
    cycle_length = len(path) - cycle_start
    max_cycle = max(max_cycle, cycle_length)
```

### Pitfall 2: Using Dictionary/Set for Path Instead of List
**The Problem:** Using a set or dictionary to track the current path loses the order information needed to calculate cycle length.

**Example of the Issue:**
```python
# INCORRECT: Using set loses ordering
path_set = set()
while current != -1 and not visited[current]:
    visited[current] = True
    path_set.add(current)
    current = edges[current]

if current in path_set:
    # How do we calculate cycle length? We don't know the position!
    cycle_length = len(path_set)  # WRONG!
```

**The Solution:** Use a list to maintain order, or use a dictionary that maps nodes to their positions:
```python
# Option 1: List (as shown in correct solution)
path = []
# ... traverse and append to path ...
if current in path:
    cycle_length = len(path) - path.index(current)

# Option 2: Dictionary with positions
path_positions = {}
position = 0
while current != -1 and not visited[current]:
    visited[current] = True
    path_positions[current] = position
    position += 1
    current = edges[current]

if current in path_positions:
    cycle_length = position - path_positions[current]
```

### Pitfall 3: Not Handling the inf Return Value Properly
**The Problem:** Using `next()` with a default of `inf` without proper handling can cause issues in the max comparison.

**Example of the Issue:**
```python
# Potential issue with inf
k = next((k for k in range(m) if cycle[k] == j), inf)
ans = max(ans, m - k)  # If k is inf, m - k becomes negative infinity!
```

**The Solution:** Add a check before using the value:
```python
# Better approach
cycle_start = -1
for k in range(len(path)):
    if path[k] == current_node:
        cycle_start = k
        break

if cycle_start != -1:  # Only update if cycle found
    cycle_length = len(path) - cycle_start
    max_cycle_length = max(max_cycle_length, cycle_length)
```

This ensures we only calculate and compare cycle lengths when an actual cycle is detected in the current path.

