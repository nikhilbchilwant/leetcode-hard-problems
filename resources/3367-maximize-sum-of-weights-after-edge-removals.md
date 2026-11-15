# 3367. Maximize Sum of Weights after Edge Removals

> Source: https://algo.monster/liteproblems/3367

## Problem Statement

You are given an undirected [tree](/problems/tree_intro) with `n` nodes numbered from `0` to `n - 1`. The tree is represented by a 2D array `edges` of length `n - 1`, where each element `edges[i] = [ui, vi, wi]` represents an edge between nodes `ui` and `vi` with weight `wi`.

Your goal is to remove some edges (or none) from the [tree](/problems/tree_intro) such that:
1. After removal, each node can be connected to **at most** `k` other nodes
2. The sum of weights of the remaining edges is **maximized**

The task is to find the maximum possible sum of weights that can be achieved after removing edges according to these constraints.

For example, if a node initially has 5 edges connected to it and `k = 3`, you must remove at least 2 edges from that node. You want to remove the edges with the smallest weights to keep the total sum as large as possible.

The solution uses [dynamic programming](/problems/dynamic_programming_intro) on the [tree](/problems/tree_intro) structure. For each node, it calculates two values:
- The maximum sum when this node can use up to `k` edges to its children
- The maximum sum when this node can use up to `k - 1` edges to its children (reserving one edge for its parent)

The algorithm performs a [depth-first search](/problems/dfs_intro) starting from node 0, computing these values bottom-up. For each node, it greedily selects the most profitable edges (those that add positive value) up to the limit `k` or `k - 1`, sorting them in descending order by their contribution to the total sum.

The final answer is the maximum of the two values computed for the root node (which has no parent, so both scenarios are valid).

## Intuition

The key insight is that we need to make local decisions at each node about which edges to keep, but these decisions affect the entire [tree](/problems/tree_intro) structure. Since each node can have at most `k` edges, we need to choose the most valuable edges at each node.

Think of it this way: imagine you're standing at any node in the [tree](/problems/tree_intro). You can see all the edges connecting to your neighbors. If you have more than `k` neighbors, you must cut some edges. Which ones should you cut? Naturally, you want to keep the edges that contribute the most to your total sum.

But here's the twist - the value of keeping an edge isn't just the edge weight itself. When you keep an edge to a child node, you're also keeping the entire subtree rooted at that child. So the real value of keeping an edge is: `edge_weight + optimal_sum_of_child_subtree`.

This leads us to a bottom-up [dynamic programming](/problems/dynamic_programming_intro) approach using DFS:

1. Start from the leaves and work our way up to the root
2. At each node, calculate the maximum sum we can get from its subtree
3. For each child, we have a choice: include the edge to that child or not
4. If we include the edge, we gain: `edge_weight + child_subtree_sum`

The clever part is recognizing that each node needs to return two values:
- `include_parent`: Maximum sum when this node uses at most `k` edges (can connect to parent)
- `exclude_parent`: Maximum sum when this node uses at most `k-1` edges (reserving one edge for parent connection)

Why two values? Because when a parent node is considering whether to keep the edge to this child, it needs to know the child's optimal sum when the child reserves an edge for the parent connection (`k-1` case).

At each node, we greedily select the best edges by calculating the "profit" of each edge: `edge_weight + child_exclude_parent - child_include_parent`. We sort these profits in descending order and take the top `k` (or `k-1`) most profitable edges.

The root node is special because it has no parent, so we can use all `k` edges for its children. That's why the final answer is the maximum of both values computed for the root.

**Learn more about [Tree](/problems/tree_intro), [Depth-First Search](/problems/dfs_intro) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The implementation uses a DFS-based [dynamic programming](/problems/dynamic_programming_intro) approach with the following key components:

**1. Graph Representation**

First, we build an adjacency list representation of the [tree](/problems/tree_intro):
```python
g: List[List[Tuple[int, int]]] = [[] for _ in range(n)]
for u, v, w in edges:
    g[u].append((v, w))
    g[v].append((u, w))
```
Each node stores a list of tuples `(neighbor, weight)` representing its connected edges.

**2. DFS Function with Two Return Values**

The core of the solution is the `dfs(u, fa)` function that returns a tuple `(a, b)`:
- `a`: Maximum sum when node `u` can use up to `k` edges to its children
- `b`: Maximum sum when node `u` can use up to `k-1` edges to its children

```python
def dfs(u: int, fa: int) -> Tuple[int, int]:
```

**3. Processing Each Node**

For each node `u`, we:
- Initialize a base sum `s = 0` (sum without any edges to children)
- Create a list `t` to store the profit of each child edge

```python
s = 0
t = []
for v, w in g[u]:
    if v == fa:  # Skip the parent edge
        continue
    a, b = dfs(v, u)  # Recursively process child
    s += a  # Add child's contribution without the connecting edge
```

**4. Calculating Edge Profits**

For each child `v` with edge weight `w`:
- `a` = child's max sum using up to `k` edges (not including parent edge)
- `b` = child's max sum using up to `k-1` edges (reserving for parent edge)
- Profit of including this edge = `w + b - a`

```python
if (d := (w + b - a)) > 0:
    t.append(d)
```

We only store positive profits because keeping an edge with negative profit would decrease our total sum.

**5. Greedy Selection**

Sort profits in descending order and greedily select the best edges:
```python
t.sort(reverse=True)
return s + sum(t[:k]), s + sum(t[:k-1])
```

- For the first return value: take up to `k` best edges
- For the second return value: take up to `k-1` best edges (reserving one for parent)

**6. Final Answer**

Start DFS from node 0 (arbitrary root) with parent `-1`:
```python
x, y = dfs(0, -1)
return max(x, y)
```

Since the root has no parent, both scenarios (using `k` edges or `k-1` edges) are valid, so we take the maximum.

**Time Complexity:** `O(n log n)` where `n` is the number of nodes. We visit each node once, and at each node, we sort its children's profits.

**Space Complexity:** `O(n)` for the adjacency list and recursion stack.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Example:** Consider a tree with 4 nodes and `k = 2`:
```
edges = [[0, 1, 10], [0, 2, 15], [1, 3, 20]]
```

This creates the following tree:
```
       0
      / \
   10/   \15
    /     \
   1       2
   |
  20|
   |
   3
```

Each node can have at most `k = 2` edges. Let's trace through the DFS:

**Step 1: DFS reaches leaf node 3**
- Node 3 has only 1 edge (to parent 1)
- No children to process
- Returns `(0, 0)` - no edges to children in either case

**Step 2: DFS backtracks to node 1**
- Node 1 has 2 edges: to parent 0 and child 3
- Process child 3: `dfs(3, 1)` returns `(0, 0)`
- Calculate profit of edge (1,3): `20 + 0 - 0 = 20`
- Profit list `t = [20]`
- Returns:
  - Using k=2 edges: `0 + sum([20]) = 20`
  - Using k-1=1 edge: `0 + sum([20]) = 20`
- Returns `(20, 20)`

**Step 3: DFS reaches leaf node 2**
- Node 2 has only 1 edge (to parent 0)
- No children to process
- Returns `(0, 0)`

**Step 4: DFS backtracks to root node 0**
- Node 0 has 2 edges: to children 1 and 2
- Process child 1: `dfs(1, 0)` returns `(20, 20)`
  - Base sum without edge: `s = 20`
  - Profit of edge (0,1): `10 + 20 - 20 = 10`
- Process child 2: `dfs(2, 0)` returns `(0, 0)`
  - Update base sum: `s = 20 + 0 = 20`
  - Profit of edge (0,2): `15 + 0 - 0 = 15`
- Profit list `t = [10, 15]`, sorted descending: `[15, 10]`
- Returns:
  - Using k=2 edges: `20 + sum([15, 10]) = 45`
  - Using k-1=1 edge: `20 + sum([15]) = 35`

**Final Answer:** `max(45, 35) = 45`

The optimal solution keeps all edges since node 0 has exactly 2 children (≤ k), node 1 has 2 edges total (≤ k), and leaf nodes have only 1 edge each.

**Key Observations:**
1. At node 1, the profit of keeping edge (1,3) is positive (20), so we include it
2. At node 0, both child edges have positive profit, and since we can keep k=2 edges, we keep both
3. The base sum at each node accumulates the "no edge" scenario, then we add back profitable edges
4. The algorithm naturally handles the constraint by limiting selections to k or k-1 edges at each step

## Python Solution

```python
class Solution:
    def maximizeSumOfWeights(self, edges: List[List[int]], k: int) -> int:
        """
        Maximize the sum of weights in a tree by selecting at most k edges per node.
      
        Args:
            edges: List of [u, v, weight] representing tree edges
            k: Maximum number of edges that can be selected per node
      
        Returns:
            Maximum sum of weights achievable
        """
      
        def dfs(current_node: int, parent_node: int) -> Tuple[int, int]:
            """
            Perform DFS to calculate maximum weights.
          
            Args:
                current_node: Current node being processed
                parent_node: Parent of current node (-1 if root)
          
            Returns:
                Tuple of (max_sum_with_k_edges, max_sum_with_k_minus_1_edges)
                - First value: max sum when current node can use k edges
                - Second value: max sum when current node can use k-1 edges (parent edge used)
            """
          
            # Base sum without including any edges from current node
            base_sum = 0
          
            # Store potential gains from including edges to children
            edge_gains = []
          
            # Process all neighbors (children in the DFS tree)
            for neighbor, edge_weight in adjacency_list[current_node]:
                # Skip the parent edge to avoid revisiting
                if neighbor == parent_node:
                    continue
              
                # Recursively get the maximum sums for the subtree rooted at neighbor
                sum_with_k, sum_with_k_minus_1 = dfs(neighbor, current_node)
              
                # Add the best sum without considering the edge to this child
                base_sum += sum_with_k
              
                # Calculate the gain from including the edge to this child
                # gain = (edge_weight + child's sum with k-1 edges) - (child's sum with k edges)
                gain = edge_weight + sum_with_k_minus_1 - sum_with_k
              
                # Only store positive gains (edges worth including)
                if gain > 0:
                    edge_gains.append(gain)
          
            # Sort gains in descending order to greedily select the best edges
            edge_gains.sort(reverse=True)
          
            # Calculate maximum sums:
            # 1. When we can use k edges from current node
            max_sum_k_edges = base_sum + sum(edge_gains[:k])
          
            # 2. When we can use k-1 edges (one slot reserved for parent edge)
            max_sum_k_minus_1_edges = base_sum + sum(edge_gains[:k - 1])
          
            return max_sum_k_edges, max_sum_k_minus_1_edges
      
        # Calculate number of nodes (tree has n-1 edges for n nodes)
        num_nodes = len(edges) + 1
      
        # Build adjacency list representation of the tree
        adjacency_list: List[List[Tuple[int, int]]] = [[] for _ in range(num_nodes)]
        for node_u, node_v, weight in edges:
            adjacency_list[node_u].append((node_v, weight))
            adjacency_list[node_v].append((node_u, weight))
      
        # Start DFS from node 0 as root (parent = -1)
        max_with_k, max_with_k_minus_1 = dfs(0, -1)
      
        # Return the maximum of both possibilities
        return max(max_with_k, max_with_k_minus_1)
```

## Time and Space Complexity

**Time Complexity:** `O(n log n)`

The algorithm performs a depth-first search (DFS) traversal of the tree:
- The DFS visits each node exactly once, contributing `O(n)` to traverse the tree
- At each node `u`, we iterate through all its adjacent edges, which across all nodes totals `O(n-1)` edge traversals (since there are `n-1` edges in a tree)
- For each node, we collect positive differences `d = w + b - a` in list `t` and sort them in descending order
- In the worst case, a node could have up to `n-1` children, making the sorting operation `O(n log n)`
- Since sorting dominates the complexity at each node and we have `n` nodes, the overall time complexity is `O(n log n)`

**Space Complexity:** `O(n)`

The space usage consists of:
- The adjacency list `g` stores all edges twice (once for each direction), using `O(n)` space
- The recursion stack for DFS can go up to `O(n)` depth in the worst case (skewed tree)
- At each recursive call, list `t` stores at most the number of children for that node, which across all recursive calls simultaneously can total `O(n)` space
- The sorting operation uses `O(log n)` auxiliary space in Python's Timsort implementation
- Overall space complexity is `O(n)` as the adjacency list and recursion stack dominate

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrectly Handling the Edge Count Constraint**

**Pitfall:** A common mistake is misunderstanding what "at most k edges" means. Some might interpret this as k edges in total across the entire tree, rather than k edges **per node**.

**Example of Wrong Thinking:**
```python
# WRONG: Trying to limit total edges globally
def wrong_approach(edges, k):
    # Sort all edges by weight and take top k
    edges.sort(key=lambda x: x[2], reverse=True)
    return sum(edge[2] for edge in edges[:k])
```

**Solution:** Remember that the constraint applies to each node individually. Each node can have at most k edges connected to it after removal.

### 2. **Forgetting to Account for Bidirectional Edges**

**Pitfall:** When building the adjacency list, forgetting that the tree is undirected and each edge should be added in both directions.

**Wrong Implementation:**
```python
# WRONG: Only adding edge in one direction
for u, v, w in edges:
    g[u].append((v, w))
    # Missing: g[v].append((u, w))
```

**Solution:** Always add edges in both directions for undirected graphs:
```python
for u, v, w in edges:
    g[u].append((v, w))
    g[v].append((u, w))
```

### 3. **Not Handling the Parent Edge Correctly in DFS**

**Pitfall:** The most subtle pitfall is incorrectly handling the parent edge case. When a node uses k-1 edges for its children, it's reserving one slot for its parent edge. However, this parent edge has already been accounted for in the parent's calculation.

**Wrong Thinking:**
```python
# WRONG: Trying to add parent edge weight again
def dfs(u, fa, parent_weight):
    # ... calculate for children ...
    return max_sum_k + parent_weight, max_sum_k_minus_1 + parent_weight  # WRONG!
```

**Solution:** The parent edge weight is already included when the parent node makes its decision. The child only needs to reserve a slot (use k-1 instead of k edges to children) but doesn't add the parent edge weight again.

### 4. **Incorrect Profit Calculation**

**Pitfall:** Misunderstanding what "profit" means when deciding whether to include an edge to a child.

**Wrong Calculation:**
```python
# WRONG: Just using edge weight as profit
profit = w  # This ignores the child's contribution
```

**Correct Calculation:**
```python
# CORRECT: Profit = gain from including edge vs not including it
profit = (w + child_sum_with_k_minus_1) - child_sum_with_k
```

The profit represents the **difference** between:
- Including the edge (weight w + child uses k-1 edges)
- Not including the edge (child uses k edges)

### 5. **Including Negative or Zero Profit Edges**

**Pitfall:** Including all edges to children regardless of their profit value.

**Wrong Implementation:**
```python
# WRONG: Including all edges even if they decrease total sum
for v, w in g[u]:
    if v != fa:
        a, b = dfs(v, u)
        t.append(w + b - a)  # Might be negative!
```

**Solution:** Only include edges with positive profit:
```python
for v, w in g[u]:
    if v != fa:
        a, b = dfs(v, u)
        profit = w + b - a
        if profit > 0:  # Only positive profits
            t.append(profit)
```

### 6. **Off-by-One Error with Number of Nodes**

**Pitfall:** Incorrectly calculating the number of nodes from the edges array.

**Wrong:**
```python
# WRONG: Using len(edges) as number of nodes
n = len(edges)  # This gives n-1, not n
```

**Correct:**
```python
# CORRECT: A tree with n nodes has n-1 edges
n = len(edges) + 1
```

### 7. **Not Considering Both Cases for the Root Node**

**Pitfall:** Only returning one value for the root instead of taking the maximum of both possibilities.

**Wrong:**
```python
# WRONG: Only considering one case
x, _ = dfs(0, -1)
return x
```

**Correct:**
```python
# CORRECT: Root has no parent, so both cases are valid
x, y = dfs(0, -1)
return max(x, y)
```

The root node is special because it has no parent edge to reserve a slot for, so both the k-edge and (k-1)-edge scenarios are valid final configurations.

