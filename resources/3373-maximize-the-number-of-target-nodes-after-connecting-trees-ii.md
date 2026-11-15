# 3373. Maximize the Number of Target Nodes After Connecting Trees II

> Source: https://algo.monster/liteproblems/3373

## Problem Statement

You are given two undirected trees with `n` and `m` nodes respectively. The first [tree](/problems/tree_intro) has nodes labeled from `[0, n-1]` and the second tree has nodes labeled from `[0, m-1]`.

The trees are represented by two 2D arrays:
- `edges1` of length `n-1` representing edges in the first [tree](/problems/tree_intro), where `edges1[i] = [ai, bi]` means there's an edge between nodes `ai` and `bi`
- `edges2` of length `m-1` representing edges in the second tree, where `edges2[i] = [ui, vi]` means there's an edge between nodes `ui` and `vi`

A key concept in this problem is the "target" relationship: Node `u` is **target** to node `v` if the number of edges on the path from `u` to `v` is even. Note that every node is always target to itself (0 edges, which is even).

Your task is to determine, for each node `i` in the first [tree](/problems/tree_intro), what is the maximum possible number of nodes that can be target to node `i` if you connect exactly one node from the first tree to one node from the second tree. This connection creates a combined graph where you need to count target nodes.

The output should be an array of `n` integers where `answer[i]` represents the maximum possible number of target nodes for node `i` in the first [tree](/problems/tree_intro).

Important: Each query is independent - after calculating the answer for one node, the added edge is removed before processing the next node.

## Intuition

Let's think about what happens when we connect two trees. When we add an edge between node `x` from [tree](/problems/tree_intro) 1 and node `y` from tree 2, we create paths between all nodes in both trees.

The key insight is understanding the "target" relationship - node `u` is target to node `v` if the path between them has an even number of edges. This is essentially asking: which nodes are at even distances from each other?

In a [tree](/problems/tree_intro), we can classify nodes into two groups based on their distance parity from any reference node. If we pick node 0 as reference and do a DFS:
- Nodes at even distances (0, 2, 4, ...) form one group
- Nodes at odd distances (1, 3, 5, ...) form another group

Here's the crucial observation: **within a single [tree](/problems/tree_intro)**, all nodes in the same parity group are at even distances from each other, and nodes in different groups are at odd distances from each other.

When we connect two trees by adding an edge between node `i` from [tree](/problems/tree_intro) 1 and node `j` from tree 2:
- If the edge connects nodes of the **same parity** (both even or both odd depth), then nodes of the same parity across both trees will be at even distances
- If the edge connects nodes of **different parity**, then nodes of opposite parity across trees will be at even distances

For any node `i` in [tree](/problems/tree_intro) 1, the target nodes come from:
1. Nodes in tree 1 that have the same parity as node `i` (these are always at even distance regardless of how we connect to tree 2)
2. Nodes from tree 2 that we can make "target" by choosing the right connection point

To maximize the total, we should connect to [tree](/problems/tree_intro) 2 in a way that adds the maximum possible nodes from tree 2. Since tree 2 has two parity groups, we want to connect such that the larger group becomes target to node `i`.

This leads to our approach: 
- Use DFS to classify nodes in both trees by parity (0 or 1)
- Count how many nodes belong to each parity group in both trees
- For each node `i` in [tree](/problems/tree_intro) 1, the answer is: (nodes with same parity as `i` in tree 1) + (maximum parity group size in tree 2)

The beauty of this solution is that it reduces a complex path-counting problem to a simple parity classification problem using DFS.

**Learn more about [Tree](/problems/tree_intro), [Depth-First Search](/problems/dfs_intro) and [Breadth-First Search](/problems/bfs_intro) patterns.**

## Solution Approach

The solution implements the parity-based approach using DFS to classify nodes in both trees.

**Step 1: Build Adjacency Lists**

The `build` function converts the edge lists into adjacency list representations for both trees:
```python
def build(edges: List[List[int]]) -> List[List[int]]:
    n = len(edges) + 1  # [Tree](/problems/tree_intro) has n-1 edges for n nodes
    g = [[] for _ in range(n)]
    for a, b in edges:
        g[a].append(b)
        g[b].append(a)
    return g
```

**Step 2: DFS for Parity Classification**

The `dfs` function traverses the [tree](/problems/tree_intro) and assigns parity values to each node:
```python
def dfs(g: List[List[int]], a: int, fa: int, c: List[int], d: int, cnt: List[int]):
    c[a] = d           # Assign parity value (0 or 1) to current node
    cnt[d] += 1        # Increment count for this parity group
    for b in g[a]:
        if b != fa:    # Avoid going back to parent
            dfs(g, b, a, c, d ^ 1, cnt)  # Flip parity for child (0→1 or 1→0)
```

Key parameters:
- `g`: adjacency list of the [tree](/problems/tree_intro)
- `a`: current node
- `fa`: parent node (to avoid cycles)
- `c`: array storing parity value for each node
- `d`: current depth parity (0 or 1)
- `cnt`: array counting nodes in each parity group

**Step 3: Main Algorithm**

```python
g1 = build(edges1)  # Build adjacency list for [tree](/problems/tree_intro) 1
g2 = build(edges2)  # Build adjacency list for tree 2
n, m = len(g1), len(g2)

c1 = [0] * n  # Parity values for [tree](/problems/tree_intro) 1 nodes
c2 = [0] * m  # Parity values for tree 2 nodes
cnt1 = [0, 0]  # Count of nodes with parity 0 and 1 in tree 1
cnt2 = [0, 0]  # Count of nodes with parity 0 and 1 in tree 2

dfs(g2, 0, -1, c2, 0, cnt2)  # Classify [tree](/problems/tree_intro) 2 nodes
dfs(g1, 0, -1, c1, 0, cnt1)  # Classify tree 1 nodes
```

**Step 4: Calculate Results**

```python
t = max(cnt2)  # Maximum parity group size in [tree](/problems/tree_intro) 2
return [t + cnt1[c1[i]] for i in range(n)]
```

For each node `i` in [tree](/problems/tree_intro) 1:
- `c1[i]` gives its parity (0 or 1)
- `cnt1[c1[i]]` gives the count of nodes with the same parity in tree 1
- `t` is the maximum possible nodes we can get from tree 2
- Total target nodes = nodes from tree 1 with same parity + maximum from tree 2

**Time Complexity**: O(n + m) where n and m are the sizes of the two trees. We perform DFS once on each [tree](/problems/tree_intro).

**Space Complexity**: O(n + m) for storing the adjacency lists and parity arrays.

The elegance of this solution lies in recognizing that the problem reduces to counting parity groups, transforming a complex graph distance problem into a simple counting problem using DFS.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Example Input:**
- Tree 1: edges1 = [[0,1], [0,2], [2,3], [2,4]] (5 nodes)
- Tree 2: edges2 = [[0,1], [0,2], [0,3]] (4 nodes)

```
Tree 1:          Tree 2:
    0                1
   / \               |
  1   2              0
     / \            / \
    3   4          2   3
```

**Step 1: Build Adjacency Lists**
- g1: {0: [1,2], 1: [0], 2: [0,3,4], 3: [2], 4: [2]}
- g2: {0: [1,2,3], 1: [0], 2: [0], 3: [0]}

**Step 2: DFS for Parity Classification**

For Tree 2 (starting from node 0 with parity 0):
- Node 0: parity = 0 (depth 0)
- Node 1: parity = 1 (depth 1)  
- Node 2: parity = 1 (depth 1)
- Node 3: parity = 1 (depth 1)
- Result: c2 = [0,1,1,1], cnt2 = [1,3] (1 node with parity 0, 3 nodes with parity 1)

For Tree 1 (starting from node 0 with parity 0):
- Node 0: parity = 0 (depth 0)
- Node 1: parity = 1 (depth 1)
- Node 2: parity = 1 (depth 1)
- Node 3: parity = 0 (depth 2)
- Node 4: parity = 0 (depth 2)
- Result: c1 = [0,1,1,0,0], cnt1 = [3,2] (3 nodes with parity 0, 2 nodes with parity 1)

**Step 3: Calculate Maximum from Tree 2**
- t = max(cnt2) = max(1,3) = 3

**Step 4: Calculate Results for Each Node**

For node 0 in Tree 1:
- c1[0] = 0 (parity 0)
- Nodes in Tree 1 with same parity: cnt1[0] = 3 (nodes 0,3,4)
- Maximum from Tree 2: t = 3
- answer[0] = 3 + 3 = 6

For node 1 in Tree 1:
- c1[1] = 1 (parity 1)
- Nodes in Tree 1 with same parity: cnt1[1] = 2 (nodes 1,2)
- Maximum from Tree 2: t = 3
- answer[1] = 2 + 3 = 5

For node 2 in Tree 1:
- c1[2] = 1 (parity 1)
- Nodes in Tree 1 with same parity: cnt1[1] = 2 (nodes 1,2)
- Maximum from Tree 2: t = 3
- answer[2] = 2 + 3 = 5

For node 3 in Tree 1:
- c1[3] = 0 (parity 0)
- Nodes in Tree 1 with same parity: cnt1[0] = 3 (nodes 0,3,4)
- Maximum from Tree 2: t = 3
- answer[3] = 3 + 3 = 6

For node 4 in Tree 1:
- c1[4] = 0 (parity 0)
- Nodes in Tree 1 with same parity: cnt1[0] = 3 (nodes 0,3,4)
- Maximum from Tree 2: t = 3
- answer[4] = 3 + 3 = 6

**Final Answer:** [6, 5, 5, 6, 6]

**Why This Works:**
- When we want to maximize target nodes for node i, we need nodes at even distances from i
- In Tree 1, nodes with the same parity as i are always at even distances (they're "free" targets)
- From Tree 2, we can choose to connect in a way that makes either parity group 0 or group 1 be at even distances from node i
- Since we want to maximize, we always choose the larger group from Tree 2 (which is 3 nodes with parity 1)
- The total is the sum of same-parity nodes in Tree 1 plus the maximum group from Tree 2

## Python Solution

```python
class Solution:
    def maxTargetNodes(
        self, edges1: List[List[int]], edges2: List[List[int]]
    ) -> List[int]:
        def build_adjacency_list(edges: List[List[int]]) -> List[List[int]]:
            """Build adjacency list representation of a tree from edge list."""
            num_nodes = len(edges) + 1
            adjacency_list = [[] for _ in range(num_nodes)]
          
            for node_a, node_b in edges:
                adjacency_list[node_a].append(node_b)
                adjacency_list[node_b].append(node_a)
          
            return adjacency_list

        def dfs_color_nodes(
            graph: List[List[int]], 
            current_node: int, 
            parent_node: int, 
            node_colors: List[int], 
            current_color: int, 
            color_counts: List[int]
        ) -> None:
            """
            DFS to color nodes with alternating colors (0 or 1) based on distance parity.
            Also counts nodes of each color.
          
            Args:
                graph: Adjacency list of the tree
                current_node: Current node being processed
                parent_node: Parent of current node (-1 if root)
                node_colors: Array to store color of each node
                current_color: Color to assign to current node (0 or 1)
                color_counts: Array to count nodes of each color
            """
            # Assign color to current node
            node_colors[current_node] = current_color
            # Increment count for this color
            color_counts[current_color] += 1
          
            # Visit all neighbors except parent
            for neighbor in graph[current_node]:
                if neighbor != parent_node:
                    # Recursively color children with opposite color (XOR with 1 flips 0->1, 1->0)
                    dfs_color_nodes(
                        graph, 
                        neighbor, 
                        current_node, 
                        node_colors, 
                        current_color ^ 1, 
                        color_counts
                    )

        # Build adjacency lists for both trees
        graph1 = build_adjacency_list(edges1)
        graph2 = build_adjacency_list(edges2)
      
        # Get number of nodes in each tree
        num_nodes_tree1 = len(graph1)
        num_nodes_tree2 = len(graph2)
      
        # Initialize color arrays for both trees (0 or 1 based on distance parity from root)
        colors_tree1 = [0] * num_nodes_tree1
        colors_tree2 = [0] * num_nodes_tree2
      
        # Initialize counters for nodes of each color in both trees
        color_counts_tree1 = [0, 0]  # [count of color 0, count of color 1]
        color_counts_tree2 = [0, 0]  # [count of color 0, count of color 1]
      
        # Color nodes in tree2 starting from node 0 as root
        dfs_color_nodes(graph2, 0, -1, colors_tree2, 0, color_counts_tree2)
      
        # Color nodes in tree1 starting from node 0 as root
        dfs_color_nodes(graph1, 0, -1, colors_tree1, 0, color_counts_tree1)
      
        # Get the maximum count between the two colors in tree2
        max_color_count_tree2 = max(color_counts_tree2)
      
        # For each node in tree1, calculate the result:
        # max nodes from tree2 + nodes of same color in tree1
        result = []
        for node_idx in range(num_nodes_tree1):
            node_color = colors_tree1[node_idx]
            total = max_color_count_tree2 + color_counts_tree1[node_color]
            result.append(total)
      
        return result
```

## Time and Space Complexity

**Time Complexity:** `O(n + m)`

The algorithm consists of the following operations:
- Building graph `g1` from `edges1`: iterates through `n-1` edges, taking `O(n)` time
- Building graph `g2` from `edges2`: iterates through `m-1` edges, taking `O(m)` time  
- DFS traversal on `g2`: visits each of the `m` nodes exactly once, taking `O(m)` time
- DFS traversal on `g1`: visits each of the `n` nodes exactly once, taking `O(n)` time
- Finding maximum of `cnt2`: `O(1)` since `cnt2` has only 2 elements
- Building the result list: iterates through `n` nodes with `O(1)` work per node, taking `O(n)` time

Total time complexity: `O(n) + O(m) + O(m) + O(n) + O(1) + O(n) = O(n + m)`

**Space Complexity:** `O(n + m)`

The algorithm uses the following space:
- Adjacency list `g1`: stores `n` nodes with a total of `2(n-1)` edges (each edge appears twice), taking `O(n)` space
- Adjacency list `g2`: stores `m` nodes with a total of `2(m-1)` edges, taking `O(m)` space
- Arrays `c1` and `c2`: `O(n)` and `O(m)` space respectively
- Arrays `cnt1` and `cnt2`: `O(1)` space each (fixed size of 2)
- DFS recursion stack: at most `O(n)` for `g1` and `O(m)` for `g2` in the worst case (when the tree is a chain)
- Result array: `O(n)` space

Total space complexity: `O(n) + O(m) + O(n) + O(m) + O(1) = O(n + m)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Tree Root Selection Assumption**

**The Pitfall:**
The most critical pitfall in this solution is assuming that we can arbitrarily choose node 0 as the root for both trees when performing DFS coloring. While this works for tree2 (since we're only interested in the maximum count of either color group), it can produce incorrect results for tree1 because the parity classification of nodes depends on which node is chosen as the reference point.

**Why It Happens:**
- When we color tree1 starting from node 0, we're essentially measuring distances from node 0
- But the problem asks for the maximum target nodes for each node `i`, meaning node `i` should be the reference point
- The parity relationship between nodes changes depending on the reference node

**Example Scenario:**
```
Tree1: 0 -- 1 -- 2 -- 3
```
- If we color from node 0: colors = [0, 1, 0, 1]
- If we color from node 1: colors = [1, 0, 1, 0]
- If we color from node 2: colors = [0, 1, 0, 1]

When calculating target nodes for node 1, using node 0 as reference gives wrong parity relationships.

**The Solution:**
We need to perform DFS coloring for each query node separately:

```python
def maxTargetNodes(self, edges1: List[List[int]], edges2: List[List[int]]) -> List[int]:
    def build_adjacency_list(edges):
        num_nodes = len(edges) + 1
        adjacency_list = [[] for _ in range(num_nodes)]
        for node_a, node_b in edges:
            adjacency_list[node_a].append(node_b)
            adjacency_list[node_b].append(node_a)
        return adjacency_list

    def dfs_color_nodes(graph, current_node, parent_node, node_colors, current_color, color_counts):
        node_colors[current_node] = current_color
        color_counts[current_color] += 1
      
        for neighbor in graph[current_node]:
            if neighbor != parent_node:
                dfs_color_nodes(graph, neighbor, current_node, node_colors, current_color ^ 1, color_counts)

    graph1 = build_adjacency_list(edges1)
    graph2 = build_adjacency_list(edges2)
  
    num_nodes_tree1 = len(graph1)
    num_nodes_tree2 = len(graph2)
  
    # For tree2, we can use any root since we only need the max count
    colors_tree2 = [0] * num_nodes_tree2
    color_counts_tree2 = [0, 0]
    dfs_color_nodes(graph2, 0, -1, colors_tree2, 0, color_counts_tree2)
    max_color_count_tree2 = max(color_counts_tree2)
  
    result = []
  
    # For each node in tree1, perform DFS with that node as root
    for query_node in range(num_nodes_tree1):
        colors_tree1 = [0] * num_nodes_tree1
        color_counts_tree1 = [0, 0]
      
        # Color tree1 with query_node as the root
        dfs_color_nodes(graph1, query_node, -1, colors_tree1, 0, color_counts_tree1)
      
        # Nodes with color 0 (same color as root) are at even distance
        # These are the target nodes from tree1
        target_nodes_tree1 = color_counts_tree1[0]
      
        # Total = target nodes from tree1 + max possible from tree2
        result.append(target_nodes_tree1 + max_color_count_tree2)
  
    return result
```

### 2. **Optimization Consideration**

While the corrected solution above works, it has O(n²) time complexity because we run DFS for each node. If this is too slow, a more sophisticated approach would involve:
- Pre-computing parity relationships between all pairs of nodes
- Or using the fact that we can derive parity relationships transitively

However, for moderate-sized trees (n ≤ 1000), the O(n²) solution should be acceptable.

### 3. **Edge Case: Single Node Trees**

Another pitfall is not handling the edge case where one or both trees have only a single node (empty edges list). The solution should handle this gracefully since a single node is always target to itself.

