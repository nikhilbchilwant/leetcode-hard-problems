# 2973. Find Number of Coins to Place in Tree Nodes

> Source: https://algo.monster/liteproblems/2973

## Problem Statement

You are given an undirected [tree](/problems/tree_intro) with `n` nodes labeled from `0` to `n - 1`, rooted at node `0`. The tree structure is defined by a 2D integer array `edges` of length `n - 1`, where `edges[i] = [ai, bi]` indicates an edge between nodes `ai` and `bi`.

Each node has an associated cost given in a 0-indexed integer array `cost` of length `n`, where `cost[i]` represents the cost value of node `i`.

Your task is to determine how many coins to place at each node according to these rules:

1. **If the subtree of node `i` contains fewer than 3 nodes**: Place exactly `1` coin at node `i`.

2. **If the subtree of node `i` contains 3 or more nodes**: 
   - Select 3 distinct nodes from the subtree of node `i`
   - Calculate the product of their cost values
   - Place coins equal to the **maximum possible product** from all combinations of 3 nodes
   - If this maximum product is negative, place `0` coins instead

The subtree of a node includes the node itself and all its descendants.

Return an array `coin` of size `n` where `coin[i]` represents the number of coins to be placed at node `i`.

**Example considerations:**
- When calculating the maximum product of 3 nodes, you need to consider that negative costs can exist
- The maximum product might come from 3 positive values (largest three) or from 2 negative values and 1 positive value (two most negative × one most positive)
- The [tree](/problems/tree_intro) is undirected but rooted at node `0`, meaning we traverse it as a rooted tree structure

## Intuition

When we need to calculate something for each node based on its subtree, DFS is the natural choice because it processes children before parents, allowing us to aggregate information bottom-up.

The key insight is recognizing what information we actually need from each subtree. For any node, we need to find the maximum product of 3 distinct cost values from its subtree. 

At first, you might think we need to keep track of all cost values in each subtree, but that would be inefficient. Instead, let's think about which values could possibly contribute to the maximum product:

1. **If all costs are positive**: The maximum product comes from the 3 largest values.
2. **If we have negative costs**: The maximum could come from either:
   - The 3 largest positive values (if they exist)
   - 2 smallest (most negative) values × 1 largest positive value (negative × negative = positive)

This means we don't need all values - we only need to track the 2 smallest and 3 largest values from each subtree. This is a crucial optimization.

The DFS approach naturally builds up this information:
1. Start at a node and recursively visit all children
2. Collect the important cost values (smallest 2 and largest 3) from each child's subtree
3. Add the current node's cost to this collection
4. Sort to identify the smallest and largest values
5. Calculate the maximum product for this node using the formula: `max(0, largest3 × largest2 × largest1, smallest1 × smallest2 × largest1)`
6. Pass up only the 2 smallest and 3 largest values to the parent (at most 5 values total)

This pruning strategy (keeping only 5 values) ensures that even as we traverse up the [tree](/problems/tree_intro), we maintain constant space per recursive call while still having all the information needed to calculate the maximum product at any ancestor node.

**Learn more about [Tree](/problems/tree_intro), [Depth-First Search](/problems/dfs_intro), [Dynamic Programming](/problems/dynamic_programming_intro), [Sorting](/problems/sorting_summary) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

The implementation follows a DFS pattern with optimization for maintaining only the necessary cost values from each subtree.

**Data Structure Setup:**
- Build an adjacency list `g` where `g[a]` contains all neighbor nodes of node `a`
- Initialize answer array `ans` with all values set to `1` (default for subtrees with less than 3 nodes)

**Core DFS Function:**
The `dfs(a, fa)` function traverses the [tree](/problems/tree_intro) and returns an array containing the most relevant cost values from the subtree rooted at node `a`:

1. **Initialize result array**: Start with `res = [cost[a]]` containing the current node's cost

2. **Traverse children**: For each neighbor `b` of node `a`:
   - Skip if `b` is the parent node `fa` (to avoid going back up the [tree](/problems/tree_intro))
   - Recursively call `dfs(b, a)` and extend `res` with the returned values

3. **Sort the collected costs**: After collecting all costs from the subtree, sort `res` to easily identify smallest and largest values

4. **Calculate coins for current node**:
   - If `len(res) >= 3`: Calculate the maximum product using two possible combinations:
     - `res[-3] × res[-2] × res[-1]`: Product of 3 largest values
     - `res[0] × res[1] × res[-1]`: Product of 2 smallest and 1 largest value
     - Take `max(0, combination1, combination2)` to handle negative products
   - Otherwise: Keep the default value of `1` coin

5. **Optimize the return values**: If `len(res) > 5`, prune the array to keep only:
   - `res[:2]`: The 2 smallest values
   - `res[-3:]`: The 3 largest values
 
   This ensures we pass at most 5 values up the [tree](/problems/tree_intro) while maintaining all necessary information for parent calculations

**Algorithm Execution:**
- Start DFS from root node `0` with parent `-1`: `dfs(0, -1)`
- The function populates the `ans` array during traversal
- Return the completed `ans` array

**Time Complexity**: `O(n × log n)` where `n` is the number of nodes, due to [sorting](/problems/sorting_summary) at each node (but each value participates in at most `O(log n)` sorts as it moves up the [tree](/problems/tree_intro))

**Space Complexity**: `O(n)` for the adjacency list and recursion stack, with each recursive call maintaining at most 5 cost values

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Example Tree:**
```
edges = [[0,1], [0,2], [2,3], [2,4]]
cost = [2, -1, 3, 5, -2]

Tree structure:
      0(2)
     / \
   1(-1) 2(3)
        / \
      3(5) 4(-2)
```

**Step-by-step DFS traversal:**

1. **Start at node 0, visit child 1:**
   - Node 1 is a leaf, subtree has only 1 node
   - `res = [-1]`, length < 3, so `ans[1] = 1`
   - Return `[-1]` to parent

2. **From node 0, visit child 2:**
 
   2a. **From node 2, visit child 3:**
   - Node 3 is a leaf
   - `res = [5]`, length < 3, so `ans[3] = 1`
   - Return `[5]` to parent
 
   2b. **From node 2, visit child 4:**
   - Node 4 is a leaf
   - `res = [-2]`, length < 3, so `ans[4] = 1`
   - Return `[-2]` to parent
 
   2c. **Back at node 2:**
   - Collect costs from subtree: `res = [3]` (node 2's cost)
   - Add child 3's values: `res = [3, 5]`
   - Add child 4's values: `res = [3, 5, -2]`
   - Sort: `res = [-2, 3, 5]`
   - Length = 3, calculate products:
     - Three largest: `3 × 5 × (-2) = -30`
     - Two smallest × one largest: `(-2) × 3 × 5 = -30`
     - `ans[2] = max(0, -30, -30) = 0`
   - Return `[-2, 3, 5]` to parent

3. **Back at node 0:**
   - Collect costs from subtree: `res = [2]` (node 0's cost)
   - Add child 1's values: `res = [2, -1]`
   - Add child 2's values: `res = [2, -1, -2, 3, 5]`
   - Sort: `res = [-2, -1, 2, 3, 5]`
   - Length = 5, calculate products:
     - Three largest: `2 × 3 × 5 = 30`
     - Two smallest × one largest: `(-2) × (-1) × 5 = 10`
     - `ans[0] = max(0, 30, 10) = 30`
   - Since this is the root, we don't need to return values

**Final Result:** `ans = [30, 1, 0, 1, 1]`

**Key Observations:**
- Leaf nodes (1, 3, 4) have subtrees with < 3 nodes, so they get 1 coin
- Node 2's subtree has exactly 3 nodes, but the maximum product is negative, so it gets 0 coins
- Node 0's subtree has 5 nodes, and the maximum product (2 × 3 × 5 = 30) comes from the three largest positive values
- The optimization of keeping only 5 values (2 smallest + 3 largest) when returning from node 2 would still preserve all necessary information for calculating node 0's result

## Python Solution

```python
class Solution:
    def placedCoins(self, edges: List[List[int]], cost: List[int]) -> List[int]:
        """
        Calculate the maximum product of 3 costs for each subtree in a tree.
      
        Args:
            edges: List of edges representing the tree structure
            cost: List of costs for each node
          
        Returns:
            List where each element is either 1 (if subtree has < 3 nodes) 
            or the maximum product of 3 costs in that subtree
        """
      
        def dfs(node: int, parent: int) -> List[int]:
            """
            Perform DFS to collect costs from subtree and calculate maximum product.
          
            Args:
                node: Current node being visited
                parent: Parent of current node (to avoid revisiting)
              
            Returns:
                List of important costs from this subtree (at most 5 values)
            """
            # Start with current node's cost
            subtree_costs = [cost[node]]
          
            # Traverse all children (excluding parent)
            for child in adjacency_list[node]:
                if child != parent:
                    # Recursively get costs from child's subtree
                    child_costs = dfs(child, node)
                    subtree_costs.extend(child_costs)
          
            # Sort costs to easily find minimum and maximum values
            subtree_costs.sort()
          
            # Calculate maximum product if we have at least 3 nodes
            if len(subtree_costs) >= 3:
                # Maximum product could be:
                # 1. Product of 3 largest values (all positive)
                # 2. Product of 2 smallest (negative) and 1 largest (positive)
                # 3. Zero (if no positive product possible)
                max_product = max(
                    subtree_costs[-3] * subtree_costs[-2] * subtree_costs[-1],  # 3 largest
                    subtree_costs[0] * subtree_costs[1] * subtree_costs[-1],    # 2 smallest, 1 largest
                    0  # Non-negative constraint
                )
                result[node] = max_product
          
            # Optimization: Keep only the most relevant values for parent calculations
            # We only need 2 smallest and 3 largest values for future products
            if len(subtree_costs) > 5:
                subtree_costs = subtree_costs[:2] + subtree_costs[-3:]
          
            return subtree_costs
      
        # Initialize variables
        n = len(cost)
        adjacency_list = [[] for _ in range(n)]
      
        # Build adjacency list from edges
        for node_a, node_b in edges:
            adjacency_list[node_a].append(node_b)
            adjacency_list[node_b].append(node_a)
      
        # Initialize result array with 1 (default for subtrees with < 3 nodes)
        result = [1] * n
      
        # Start DFS from node 0 with no parent (-1)
        dfs(0, -1)
      
        return result
```

## Time and Space Complexity

**Time Complexity:** `O(n × log n)`

The algorithm performs a depth-first search (DFS) traversal of the tree starting from node 0. During the traversal:
- Each node is visited exactly once, contributing `O(n)` to the traversal itself
- At each node, we collect costs from all descendants in its subtree and sort them
- The sorting operation at each node takes `O(k × log k)` where `k` is the size of the subtree rooted at that node
- In the worst case (a linear tree), the root node would need to sort all `n` elements, giving `O(n × log n)`
- While multiple sorts occur across different nodes, the amortized analysis shows that each element participates in sorting operations along its path to the root
- The dominant operation is the sorting at nodes with large subtrees, particularly near the root

Therefore, the overall time complexity is `O(n × log n)`.

**Space Complexity:** `O(n)`

The space usage consists of:
- The adjacency list `g` which stores all edges: `O(n)` (since it's a tree with `n-1` edges)
- The answer array `ans`: `O(n)`
- The recursion call stack depth: `O(n)` in the worst case (linear tree)
- The `res` list at each recursive call: Although we collect costs from subtrees, we trim the list to at most 5 elements (`res = res[:2] + res[-3:]`), so each call maintains at most `O(1)` additional space for the trimmed result
- The total space for all `res` lists across the recursion is bounded by `O(n)`

Therefore, the overall space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Forgetting to Handle Negative Products

**Issue:** A common mistake is only considering the product of the three largest values when calculating the maximum product. This fails when negative values are present in the cost array.

**Why it happens:** With negative costs, the maximum product might come from multiplying two negative values (which gives a positive) with one positive value. For example, if costs are `[-10, -5, 1, 2, 3]`, the maximum product is `(-10) × (-5) × 3 = 150`, not `1 × 2 × 3 = 6`.

**Solution:** Always check both combinations:
```python
# Incorrect approach
max_product = subtree_costs[-3] * subtree_costs[-2] * subtree_costs[-1]

# Correct approach
max_product = max(
    subtree_costs[-3] * subtree_costs[-2] * subtree_costs[-1],  # 3 largest
    subtree_costs[0] * subtree_costs[1] * subtree_costs[-1],    # 2 smallest, 1 largest
    0  # Handle negative products
)
```

### Pitfall 2: Not Optimizing the Returned Cost Array

**Issue:** Returning all collected costs from each subtree without pruning leads to exponential growth in the size of arrays being passed up the tree, causing time and memory limit exceeded errors.

**Why it happens:** In a deep tree, without pruning, each node would accumulate all costs from its entire subtree. A parent node would then need to sort and process increasingly large arrays.

**Solution:** After sorting, keep only the values that matter for future calculations:
```python
# Inefficient - returns entire subtree
return subtree_costs

# Efficient - returns only necessary values
if len(subtree_costs) > 5:
    subtree_costs = subtree_costs[:2] + subtree_costs[-3:]
return subtree_costs
```

### Pitfall 3: Incorrect Tree Traversal

**Issue:** Treating the undirected edges as a directed tree without properly tracking the parent node, leading to infinite recursion or incorrect subtree calculations.

**Why it happens:** The edges represent an undirected graph, but we need to traverse it as a rooted tree. Without tracking the parent, DFS might revisit nodes.

**Solution:** Pass the parent node in DFS and skip it during traversal:
```python
def dfs(node, parent):
    for child in adjacency_list[node]:
        if child != parent:  # Critical check to avoid going back to parent
            child_costs = dfs(child, node)
```

### Pitfall 4: Off-by-One Errors in Array Indexing

**Issue:** Using incorrect indices when accessing the smallest or largest values after sorting, especially when the array has exactly 3 elements.

**Why it happens:** Confusion about zero-based indexing and negative indexing in Python.

**Solution:** Remember that for a sorted array:
- `arr[0]` and `arr[1]` are the two smallest values
- `arr[-1]`, `arr[-2]`, and `arr[-3]` are the three largest values (in ascending order)
```python
# For array [1, 2, 3, 4, 5]:
# arr[0] = 1 (smallest)
# arr[1] = 2 (second smallest)
# arr[-3] = 3 (third largest)
# arr[-2] = 4 (second largest)
# arr[-1] = 5 (largest)
```

