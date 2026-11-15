# 2322. Minimum Score After Removals on a Tree

> Source: https://algo.monster/liteproblems/2322

## Problem Statement

You are given an undirected connected [tree](/problems/tree_intro) with `n` nodes labeled from `0` to `n - 1` and `n - 1` edges. Each node `i` has a value `nums[i]`, and the edges are represented as a 2D array where `edges[i] = [ai, bi]` indicates an edge between nodes `ai` and `bi`.

Your task is to remove two distinct edges from the [tree](/problems/tree_intro), which will split it into three connected components. For each pair of removed edges, you need to:

1. Calculate the XOR of all node values in each of the three components
2. Find the difference between the largest and smallest XOR values among the three components - this is the "score" for that pair of removed edges

For example, if after removing two edges you get three components with node values `[4,5,7]`, `[1,9]`, and `[3,3,3]`:
- Component 1 XOR: `4 ^ 5 ^ 7 = 6`
- Component 2 XOR: `1 ^ 9 = 8`  
- Component 3 XOR: `3 ^ 3 ^ 3 = 3`
- The score would be `8 - 3 = 5` (largest XOR minus smallest XOR)

You need to find and return the minimum possible score among all valid pairs of edge removals.

## Intuition

When we remove two edges from a [tree](/problems/tree_intro), we split it into exactly three connected components. The key insight is that we need to systematically explore all possible ways to remove two edges and track the resulting XOR values.

Let's think about what happens when we remove edges. If we remove edge `(u, v)`, we split the [tree](/problems/tree_intro) into two parts. If we then remove another edge from one of these parts, we get three components total. The challenge is computing the XOR sum for each component efficiently.

The first observation is that XOR has a useful property: `a ^ a = 0`. This means if we know the total XOR sum `s` of all nodes and the XOR sum `s1` of one component, we can get the XOR sum of the remaining part as `s ^ s1`.

Here's the strategy that emerges:
1. First, calculate the total XOR sum `s` of all node values in the [tree](/problems/tree_intro)
2. For each possible first edge to remove, we get two components - one with XOR sum `s1` and the other with XOR sum `s ^ s1`
3. Within the component with XOR sum `s1`, we explore removing a second edge, which creates a sub-component with XOR sum `s2`
4. This gives us three components with XOR sums: `s ^ s1` (the untouched component), `s2` (the sub-component), and `s1 ^ s2` (the remainder after removing `s2` from `s1`)

The beauty of using DFS is that it naturally computes subtree XOR sums. When we do DFS from a node, we can accumulate the XOR values from all its descendants. By treating each edge as a potential cut point and using the parent-child relationship in DFS traversal, we can efficiently enumerate all possible ways to remove two edges.

We need two DFS functions: one to compute the XOR sum after the first edge removal, and another to explore all possible second edge removals within that component while keeping track of the minimum score.

**Learn more about [Tree](/problems/tree_intro) and [Depth-First Search](/problems/dfs_intro) patterns.**

## Solution Approach

The implementation uses two DFS functions to systematically explore all possible ways of removing two edges from the [tree](/problems/tree_intro).

**Step 1: Build the Graph Representation**
We first construct an adjacency list `g` using a `defaultdict(list)` to represent the [tree](/problems/tree_intro) structure. Each edge `[a, b]` creates bidirectional connections.

**Step 2: Calculate Total XOR Sum**
We compute `s`, the XOR sum of all node values using `reduce(lambda x, y: x ^ y, nums)`. This gives us the baseline XOR value for the entire [tree](/problems/tree_intro).

**Step 3: First DFS Function - `dfs(i, fa)`**
This function calculates the XOR sum of a subtree rooted at node `i`, where `fa` is the parent node (to avoid revisiting):
- Start with `res = nums[i]` (the node's own value)
- For each child `j` of node `i` (excluding parent `fa`), recursively compute and XOR the subtree values
- Return the accumulated XOR sum for this subtree

**Step 4: Second DFS Function - `dfs2(i, fa)`**
This function explores removing a second edge within the subtree and calculates the minimum score:
- Similar to `dfs`, it computes the XOR sum of the subtree rooted at `i`
- For each child `j`, it gets the XOR sum `s2` of that child's subtree
- With `s2` known, we now have three components:
  - Component 1: `s ^ s1` (everything outside the first cut)
  - Component 2: `s2` (the subtree below the second cut)
  - Component 3: `s1 ^ s2` (remainder after removing `s2` from `s1`)
- Calculate `mx = max(s ^ s1, s2, s1 ^ s2)` and `mn = min(s ^ s1, s2, s1 ^ s2)`
- Update the global minimum: `ans = min(ans, mx - mn)`

**Step 5: Main Loop - Enumerate All First Cuts**
For each node `i` and each of its neighbors `j`:
- Treat the edge `(i, j)` as the first edge to remove
- Calculate `s1 = dfs(i, j)` - the XOR sum of the component containing node `i` after removing edge `(i, j)`
- Call `dfs2(i, j)` to explore all possible second edge removals within this component

The algorithm uses the XOR properties and [tree](/problems/tree_intro) structure to efficiently compute all possible three-way partitions. By systematically trying each edge as the first cut and then exploring second cuts within the resulting components, we ensure we find the minimum possible score.

The time complexity is `O(n²)` where `n` is the number of nodes, as we potentially explore each edge as both first and second cut. The space complexity is `O(n)` for the recursion stack and adjacency list storage.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

Consider a tree with 5 nodes where:
- `nums = [6, 4, 2, 3, 5]` (node values)
- `edges = [[0,1], [1,2], [1,3], [3,4]]`

The tree structure looks like:
```
      0(6)
       |
      1(4)
     / | \
   2(2) 3(3)
         |
        4(5)
```

**Step 1: Calculate Total XOR Sum**
`s = 6 ^ 4 ^ 2 ^ 3 ^ 5 = 0`

**Step 2: Try First Edge Removal - Edge (1,3)**
Let's remove edge (1,3) as our first cut. This splits the tree into:
- Component A: nodes {0,1,2} 
- Component B: nodes {3,4}

Calculate `s1` (XOR of Component A containing node 1):
- `dfs(1, 3)` computes: `4 ^ 6 ^ 2 = 0`
- So `s1 = 0`
- The other component has XOR: `s ^ s1 = 0 ^ 0 = 0`

**Step 3: Explore Second Edge Removals within Component A**
Now we call `dfs2(1, 3)` to try removing a second edge within Component A.

When exploring edge (0,1) as the second cut:
- `s2 = dfs(0, 1) = 6` (just node 0)
- Three components now have XOR values:
  - Component 1: `s ^ s1 = 0` (nodes {3,4})
  - Component 2: `s2 = 6` (node {0})
  - Component 3: `s1 ^ s2 = 0 ^ 6 = 6` (nodes {1,2})
- Score: `max(0, 6, 6) - min(0, 6, 6) = 6 - 0 = 6`

When exploring edge (1,2) as the second cut:
- `s2 = dfs(2, 1) = 2` (just node 2)
- Three components:
  - Component 1: `s ^ s1 = 0` (nodes {3,4})
  - Component 2: `s2 = 2` (node {2})
  - Component 3: `s1 ^ s2 = 0 ^ 2 = 2` (nodes {0,1})
- Score: `max(0, 2, 2) - min(0, 2, 2) = 2 - 0 = 2`

**Step 4: Try Other First Edge Removals**
We repeat this process for all other edges as potential first cuts:

When first removing edge (3,4):
- `s1 = dfs(3, 4) = 6 ^ 4 ^ 2 ^ 3 = 3`
- Then exploring second cuts within this component gives various scores

When first removing edge (0,1):
- `s1 = dfs(0, 1) = 6`
- Component B has XOR: `0 ^ 6 = 6`
- Then exploring second cuts in Component B (which has nodes {1,2,3,4})

**Step 5: Find Minimum Score**
After trying all possible pairs of edge removals, we track the minimum score encountered. In this example, one optimal solution might be removing edges (1,3) and (1,2), giving us a score of 2.

The algorithm systematically explores all `O(n²)` possible pairs of edge removals, using the XOR properties to efficiently compute component values and find the minimum possible score.

## Python Solution

```python
class Solution:
    def minimumScore(self, nums: List[int], edges: List[List[int]]) -> int:
        from collections import defaultdict
        from functools import reduce
      
        def calculate_subtree_xor(node: int, parent: int) -> int:
            """
            Calculate XOR of all values in subtree rooted at 'node'.
          
            Args:
                node: Current node being processed
                parent: Parent node to avoid revisiting
          
            Returns:
                XOR value of the subtree
            """
            xor_result = nums[node]
            for neighbor in graph[node]:
                if neighbor != parent:
                    xor_result ^= calculate_subtree_xor(neighbor, node)
            return xor_result
      
        def find_minimum_score(node: int, parent: int) -> int:
            """
            Traverse tree and calculate minimum score for all possible second edge removals.
          
            Args:
                node: Current node being processed
                parent: Parent node to avoid revisiting
          
            Returns:
                XOR value of the subtree rooted at 'node'
            """
            nonlocal total_xor, first_subtree_xor, min_score
          
            subtree_xor = nums[node]
            for neighbor in graph[node]:
                if neighbor != parent:
                    # Calculate XOR for subtree rooted at 'neighbor'
                    second_subtree_xor = find_minimum_score(neighbor, node)
                    subtree_xor ^= second_subtree_xor
                  
                    # Calculate XOR values for three components after removing two edges
                    # Component 1: first_subtree_xor (from first edge removal)
                    # Component 2: second_subtree_xor (from second edge removal)
                    # Component 3: remaining part (total_xor ^ first_subtree_xor ^ second_subtree_xor)
                    remaining_xor = total_xor ^ first_subtree_xor
                    component1 = remaining_xor ^ second_subtree_xor
                    component2 = second_subtree_xor
                    component3 = first_subtree_xor
                  
                    # Find maximum and minimum among three components
                    max_xor = max(component1, component2, component3)
                    min_xor = min(component1, component2, component3)
                  
                    # Update minimum score
                    min_score = min(min_score, max_xor - min_xor)
          
            return subtree_xor
      
        # Build adjacency list representation of the tree
        graph = defaultdict(list)
        for node_a, node_b in edges:
            graph[node_a].append(node_b)
            graph[node_b].append(node_a)
      
        # Calculate total XOR of all nodes
        total_xor = reduce(lambda x, y: x ^ y, nums)
      
        n = len(nums)
        min_score = float('inf')
      
        # Try each edge as the first edge to remove
        for node in range(n):
            for neighbor in graph[node]:
                # Remove edge between 'node' and 'neighbor'
                # Calculate XOR of subtree rooted at 'node' (with 'neighbor' as parent)
                first_subtree_xor = calculate_subtree_xor(node, neighbor)
              
                # Find optimal second edge to remove
                find_minimum_score(node, neighbor)
      
        return min_score
```

## Time and Space Complexity

The time complexity is `O(n²)`, where `n` is the number of nodes in the tree.

**Time Complexity Analysis:**
- The outer loop iterates through all `n` nodes
- For each node `i`, we iterate through its adjacent nodes (neighbors in `g[i]`)
- For each edge `(i, j)`, we call `dfs(i, j)` and `dfs2(i, j)`
- Both `dfs` and `dfs2` traverse the entire tree once, taking `O(n)` time each
- Since a tree with `n` nodes has exactly `n-1` edges, and we process each edge from both directions, we perform `2(n-1)` iterations of the inner loop across all outer loop iterations
- Total time: `O(n) × O(n) = O(n²)`

The space complexity is `O(n)`, where `n` is the number of nodes in the tree.

**Space Complexity Analysis:**
- The graph adjacency list `g` stores all edges, requiring `O(n)` space for a tree
- The recursion stack depth for both `dfs` and `dfs2` can go up to `O(n)` in the worst case (when the tree is a linear chain)
- Variables `s`, `s1`, `s2`, and `ans` use `O(1)` space
- Total space: `O(n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Component XOR Calculation After Edge Removal**

One of the most common mistakes is incorrectly calculating the XOR values of the three components after removing two edges. The code has a subtle bug in how it computes the three component XOR values.

**The Issue:**
In the `find_minimum_score` function, after removing two edges, the three components are calculated as:
- `component1 = remaining_xor ^ second_subtree_xor` 
- `component2 = second_subtree_xor`
- `component3 = first_subtree_xor`

However, this is incorrect. When we remove the first edge (between some node and its neighbor), we split the tree into two parts with XOR values `first_subtree_xor` and `total_xor ^ first_subtree_xor`. When we then remove a second edge within the `first_subtree_xor` component, creating a subtree with XOR `second_subtree_xor`, the three final components should be:
- Component containing the second removed subtree: `second_subtree_xor`
- Remainder of the first component: `first_subtree_xor ^ second_subtree_xor`
- The untouched component from the first split: `total_xor ^ first_subtree_xor`

**The Fix:**
```python
# Correct calculation of three components
component1 = total_xor ^ first_subtree_xor  # Untouched component
component2 = second_subtree_xor              # Second removed subtree
component3 = first_subtree_xor ^ second_subtree_xor  # Remainder of first component
```

### 2. **Not Exploring Both Directions for Edge Removal**

Another pitfall is only exploring second edge removals in one of the two components created by the first edge removal. The current code only explores removing a second edge from the component containing `node` (with XOR value `first_subtree_xor`), but it should also explore removing edges from the other component.

**The Fix:**
After removing edge `(node, neighbor)`, explore second edge removals in both resulting components:
```python
# Explore second edge removal in first component
first_subtree_xor = calculate_subtree_xor(node, neighbor)
find_minimum_score(node, neighbor)

# Also explore second edge removal in the other component
other_subtree_xor = calculate_subtree_xor(neighbor, node)
find_minimum_score(neighbor, node)
```

### 3. **Variable Naming Confusion**

The variable names in the original code can be confusing and lead to logical errors. Using generic names like `component1`, `component2`, `component3` or the original `s`, `s1`, `s2` makes it hard to track which component is which.

**The Fix:**
Use more descriptive variable names:
```python
def find_minimum_score_with_second_cut(root: int, parent: int) -> int:
    nonlocal total_xor, first_component_xor, min_score
  
    current_subtree_xor = nums[root]
    for child in graph[root]:
        if child != parent:
            second_cut_subtree_xor = find_minimum_score_with_second_cut(child, root)
            current_subtree_xor ^= second_cut_subtree_xor
          
            # Three components after two cuts:
            outside_first_cut = total_xor ^ first_component_xor
            second_cut_component = second_cut_subtree_xor
            first_cut_remainder = first_component_xor ^ second_cut_subtree_xor
          
            max_xor = max(outside_first_cut, second_cut_component, first_cut_remainder)
            min_xor = min(outside_first_cut, second_cut_component, first_cut_remainder)
            min_score = min(min_score, max_xor - min_xor)
  
    return current_subtree_xor
```

These improvements make the code more correct, complete, and maintainable.

