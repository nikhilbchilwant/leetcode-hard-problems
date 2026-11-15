# 3203. Find Minimum Diameter After Merging Two Trees

> Source: https://algo.monster/liteproblems/3203

## Problem Statement

You are given two undirected trees with `n` and `m` nodes respectively. The first [tree](/problems/tree_intro) has nodes numbered from `0` to `n-1`, and the second tree has nodes numbered from `0` to `m-1`. The edges of these trees are provided in two arrays: `edges1` (with `n-1` edges) and `edges2` (with `m-1` edges).

Your task is to connect these two trees by adding exactly one edge between any node from the first [tree](/problems/tree_intro) and any node from the second tree. After connecting the trees, you need to find the minimum possible diameter of the resulting combined tree.

The **diameter** of a [tree](/problems/tree_intro) is defined as the length of the longest path between any two nodes in the tree (measured by the number of edges in the path).

For example:
- If the first [tree](/problems/tree_intro) has edges `[[0,1],[0,2],[0,3]]` (a star with center 0) and the second tree has edges `[[0,1]]` (just two connected nodes), connecting any node from the first tree to any node from the second tree will result in a combined tree with diameter 3.

- In another example with two identical larger trees, connecting node 0 from the first [tree](/problems/tree_intro) with node 0 from the second tree results in a diameter of 5.

The key insight is that to minimize the diameter of the merged [tree](/problems/tree_intro), you need to consider:
1. The original diameters `d1` and `d2` of the two trees
2. The optimal connection point, which involves finding the "center" of each tree (the node that minimizes the maximum distance to any other node)

The final answer will be the maximum of:
- The original diameter of [tree](/problems/tree_intro) 1 (`d1`)
- The original diameter of tree 2 (`d2`)  
- The new diameter created by connecting the two trees optimally: `⌈d1/2⌉ + ⌈d2/2⌉ + 1`

## Intuition

When we connect two trees with a single edge, the diameter of the resulting [tree](/problems/tree_intro) depends on how we make that connection. Let's think about what happens:

The diameter of the merged [tree](/problems/tree_intro) could come from three possible scenarios:
1. The longest path stays entirely within the first tree (original diameter `d1`)
2. The longest path stays entirely within the second tree (original diameter `d2`)
3. The longest path crosses through our new connecting edge

For scenario 3, we need to minimize the longest path that goes through the connection. The key insight is that we should connect nodes that are as "central" as possible in each [tree](/problems/tree_intro). 

Think of it this way: if we connect leaf nodes or nodes near the edge of each [tree](/problems/tree_intro), we'd create very long paths. But if we connect nodes near the "center" of each tree, we minimize the maximum distance to any other node.

The "radius" of a [tree](/problems/tree_intro) is the minimum eccentricity (maximum distance from a node to any other node) among all nodes. For a tree with diameter `d`, the radius is `⌈d/2⌉`. This represents the distance from the center to the farthest node.

When we connect the two trees at their optimal points (near their centers), the new diameter through the connection would be:
- Distance from farthest node in [tree](/problems/tree_intro) 1 to connection point: `⌈d1/2⌉`
- The connecting edge itself: `1`
- Distance from connection point to farthest node in tree 2: `⌈d2/2⌉`
- Total: `⌈d1/2⌉ + ⌈d2/2⌉ + 1`

To find the diameter of each [tree](/problems/tree_intro) efficiently, we use a clever two-pass DFS approach:
- First DFS: Start from any node and find the farthest node from it (call it node `a`)
- Second DFS: Start from node `a` and find the farthest node from it (call it node `b`)
- The path from `a` to `b` is guaranteed to be a diameter of the tree

This works because in a [tree](/problems/tree_intro), one endpoint of a diameter must be the farthest node from any given starting point, and the other endpoint must be the farthest from that first endpoint.

Finally, we take the maximum of all three possible diameters: `max(d1, d2, ⌈d1/2⌉ + ⌈d2/2⌉ + 1)`.

**Learn more about [Tree](/problems/tree_intro), [Depth-First Search](/problems/dfs_intro), [Breadth-First Search](/problems/bfs_intro) and [Graph](/problems/graph_intro) patterns.**

## Solution Approach

The implementation consists of two main parts: finding the diameter of each [tree](/problems/tree_intro) and calculating the minimum diameter after merging.

**Finding [Tree](/problems/tree_intro) Diameter with Two DFS Passes:**

The `treeDiameter` function implements the two-pass DFS algorithm:

1. **Build the adjacency list**: Convert the edge list into a [graph](/problems/graph_intro) representation using a dictionary where each node maps to its neighbors.
   ```python
   g = defaultdict(list)
   for a, b in edges:
       g[a].append(b)
       g[b].append(a)
   ```

2. **First DFS pass**: Start from node 0 and find the farthest node from it.
   - The `dfs` function takes three parameters: current node `i`, parent node `fa` (to avoid revisiting), and current distance `t`
   - We track the maximum distance `ans` and the corresponding farthest node `a`
   ```python
   ans = a = 0
   dfs(0, -1, 0)  # Start from node 0 with no parent and distance 0
   ```

3. **Second DFS pass**: Start from the farthest node `a` found in the first pass.
   ```python
   dfs(a, -1, 0)  # Start from node a to find the actual diameter
   ```
   The maximum distance found in this pass is the diameter of the [tree](/problems/tree_intro).

**DFS Implementation Details:**

The recursive DFS function explores all neighbors except the parent:
```python
def dfs(i: int, fa: int, t: int):
    for j in g[i]:
        if j != fa:  # Don't go back to parent
            dfs(j, i, t + 1)  # Increment distance by 1
    nonlocal ans, a
    if ans < t:  # Update if we found a farther node
        ans = t
        a = i
```

**Calculating Minimum Diameter After Merge:**

Once we have both diameters `d1` and `d2`, we calculate the minimum possible diameter of the merged [tree](/problems/tree_intro):

```python
return max(d1, d2, (d1 + 1) // 2 + (d2 + 1) // 2 + 1)
```

This formula accounts for three cases:
- `d1`: The diameter remains within the first [tree](/problems/tree_intro)
- `d2`: The diameter remains within the second tree  
- `(d1 + 1) // 2 + (d2 + 1) // 2 + 1`: The diameter passes through the connecting edge

The expression `(d1 + 1) // 2` computes `⌈d1/2⌉` (ceiling division) in Python, which gives us the radius of each [tree](/problems/tree_intro). Adding 1 accounts for the connecting edge itself.

**Time Complexity**: `O(n + m)` where `n` and `m` are the number of nodes in the two trees, as we perform DFS on each [tree](/problems/tree_intro) twice.

**Space Complexity**: `O(n + m)` for storing the adjacency lists and the recursion stack during DFS.

## Example Walkthrough

Let's walk through a concrete example to illustrate the solution approach.

**Example Input:**
- Tree 1: edges1 = `[[0,1],[0,2]]` (3 nodes, shaped like a V with node 0 at center)
- Tree 2: edges2 = `[[0,1],[1,2],[2,3]]` (4 nodes, a straight line)

**Step 1: Find diameter of Tree 1**

First, build adjacency list for Tree 1:
```
0: [1, 2]
1: [0]
2: [0]
```

First DFS from node 0:
- Visit 0 (distance 0)
- Visit 1 (distance 1) 
- Visit 2 (distance 1)
- Farthest node is either 1 or 2 with distance 1 (let's say node 1)

Second DFS from node 1:
- Visit 1 (distance 0)
- Visit 0 (distance 1)
- Visit 2 (distance 2)
- Farthest node is 2 with distance 2

**Diameter of Tree 1 (d1) = 2**

**Step 2: Find diameter of Tree 2**

Build adjacency list for Tree 2:
```
0: [1]
1: [0, 2]
2: [1, 3]
3: [2]
```

First DFS from node 0:
- Visit 0 (distance 0)
- Visit 1 (distance 1)
- Visit 2 (distance 2)
- Visit 3 (distance 3)
- Farthest node is 3 with distance 3

Second DFS from node 3:
- Visit 3 (distance 0)
- Visit 2 (distance 1)
- Visit 1 (distance 2)
- Visit 0 (distance 3)
- Farthest node is 0 with distance 3

**Diameter of Tree 2 (d2) = 3**

**Step 3: Calculate minimum diameter after merge**

We need to find the maximum of:
- d1 = 2
- d2 = 3
- ⌈d1/2⌉ + ⌈d2/2⌉ + 1 = ⌈2/2⌉ + ⌈3/2⌉ + 1 = 1 + 2 + 1 = 4

Using integer arithmetic: `(2+1)//2 + (3+1)//2 + 1 = 1 + 2 + 1 = 4`

**Result: max(2, 3, 4) = 4**

The minimum possible diameter when connecting these two trees is 4. This occurs when we connect node 0 from Tree 1 (its center) with node 1 or 2 from Tree 2 (near its center). The longest path would be from node 2 in Tree 1, through node 0, across the connecting edge, through nodes 1/2 in Tree 2, to node 0 or 3 in Tree 2.

## Python Solution

```python
from typing import List
from collections import defaultdict

class Solution:
    def minimumDiameterAfterMerge(
        self, edges1: List[List[int]], edges2: List[List[int]]
    ) -> int:
        """
        Find the minimum possible diameter after connecting two trees with one edge.
      
        The strategy is to connect the centers of both trees to minimize the resulting diameter.
        The new diameter will be the maximum of:
        1. Original diameter of tree 1
        2. Original diameter of tree 2  
        3. Ceiling(d1/2) + ceiling(d2/2) + 1 (path through the connecting edge)
        """
        # Calculate diameters of both trees
        diameter1 = self.treeDiameter(edges1)
        diameter2 = self.treeDiameter(edges2)
      
        # The minimum diameter after merge is the maximum of:
        # - Original diameter of tree 1
        # - Original diameter of tree 2
        # - Path from center of tree 1 to center of tree 2
        # (d + 1) // 2 gives us the radius (distance from center to farthest node)
        return max(diameter1, diameter2, (diameter1 + 1) // 2 + (diameter2 + 1) // 2 + 1)

    def treeDiameter(self, edges: List[List[int]]) -> int:
        """
        Calculate the diameter of a tree using two DFS traversals.
        First DFS finds one end of the longest path.
        Second DFS from that end finds the actual diameter.
        """
        # Build adjacency list representation of the tree
        graph = defaultdict(list)
        for node_a, node_b in edges:
            graph[node_a].append(node_b)
            graph[node_b].append(node_a)
      
        # Variables to track the maximum distance and farthest node
        self.max_distance = 0
        self.farthest_node = 0
      
        def dfs(current_node: int, parent: int, distance: int) -> None:
            """
            Depth-first search to find the farthest node from the starting point.
          
            Args:
                current_node: Current node being visited
                parent: Parent node to avoid revisiting
                distance: Distance from the starting node
            """
            # Explore all neighbors except the parent
            for neighbor in graph[current_node]:
                if neighbor != parent:
                    dfs(neighbor, current_node, distance + 1)
          
            # Update the farthest node if current distance is greater
            if self.max_distance < distance:
                self.max_distance = distance
                self.farthest_node = current_node
      
        # First DFS: Find one end of the diameter starting from node 0
        dfs(0, -1, 0)
      
        # Reset max_distance for second DFS
        self.max_distance = 0
      
        # Second DFS: Find the actual diameter starting from the farthest node
        dfs(self.farthest_node, -1, 0)
      
        return self.max_distance
```

## Time and Space Complexity

The time complexity of this algorithm is `O(n + m)`, where `n` is the number of nodes in the first tree and `m` is the number of nodes in the second tree.

**Time Complexity Analysis:**
- The `treeDiameter` function is called twice, once for each tree
- For the first tree with `n` nodes:
  - Building the adjacency list from edges takes `O(n-1) = O(n)` time (since a tree with `n` nodes has `n-1` edges)
  - The DFS is performed twice from the function, each visiting all `n` nodes exactly once, taking `O(n)` time per DFS
  - Total for first tree: `O(n) + O(n) + O(n) = O(n)`
- For the second tree with `m` nodes:
  - Similarly, this takes `O(m)` time
- The final `max` operation takes `O(1)` time
- Overall time complexity: `O(n) + O(m) = O(n + m)`

**Space Complexity Analysis:**
The space complexity is `O(n + m)`.

- For the first tree:
  - The adjacency list `g` stores all edges, requiring `O(n)` space
  - The recursive DFS call stack can go up to depth `O(n)` in the worst case (when the tree is a chain)
  - Space for first tree: `O(n)`
- For the second tree:
  - Similarly requires `O(m)` space
- The variables `ans`, `a`, `d1`, `d2` use `O(1)` space
- Overall space complexity: `O(n) + O(m) = O(n + m)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Handling Single-Node Trees**
One of the most common pitfalls is not properly handling edge cases where one or both trees consist of a single node (no edges). When `edges` is empty, the tree has only one node, and its diameter should be 0.

**Problem Code:**
```python
def treeDiameter(self, edges: List[List[int]]) -> int:
    graph = defaultdict(list)
    for node_a, node_b in edges:
        graph[node_a].append(node_b)
        graph[node_b].append(node_a)
  
    # This will fail if edges is empty!
    dfs(0, -1, 0)  # Node 0 exists but has no neighbors
```

**Solution:**
Add an early return check for empty edge lists:
```python
def treeDiameter(self, edges: List[List[int]]) -> int:
    # Handle single-node tree
    if not edges:
        return 0
  
    graph = defaultdict(list)
    # ... rest of the code
```

### 2. **Integer Division vs Ceiling Division**
Another pitfall is incorrectly calculating the ceiling division for the radius. Using regular division or floor division can give wrong results.

**Problem Code:**
```python
# Wrong: This gives floor division
return max(d1, d2, d1 // 2 + d2 // 2 + 1)

# Also wrong: This doesn't handle odd diameters correctly
return max(d1, d2, math.ceil(d1 / 2) + math.ceil(d2 / 2) + 1)
```

**Solution:**
Use the formula `(d + 1) // 2` which correctly computes ceiling division for non-negative integers:
```python
return max(d1, d2, (d1 + 1) // 2 + (d2 + 1) // 2 + 1)
```

### 3. **Using Instance Variables Instead of Closure Variables**
Using `self.max_distance` and `self.farthest_node` as instance variables can cause issues in concurrent scenarios or when the method is called multiple times.

**Problem Code:**
```python
class Solution:
    def treeDiameter(self, edges):
        self.max_distance = 0  # Instance variable - persists between calls
        self.farthest_node = 0
        # ... rest of code
```

**Solution:**
Use local variables with `nonlocal` keyword or pass them as parameters:
```python
def treeDiameter(self, edges):
    max_distance = 0
    farthest_node = 0
  
    def dfs(current, parent, distance):
        nonlocal max_distance, farthest_node
        # ... rest of code
```

### 4. **Incorrect Starting Node Assumption**
Assuming node 0 always exists can fail when the tree nodes don't include 0.

**Problem Code:**
```python
# Assumes node 0 exists
dfs(0, -1, 0)
```

**Solution:**
Start from any valid node in the graph:
```python
if not graph:
    return 0
  
# Start from any node in the graph
start_node = next(iter(graph))
dfs(start_node, -1, 0)
```

