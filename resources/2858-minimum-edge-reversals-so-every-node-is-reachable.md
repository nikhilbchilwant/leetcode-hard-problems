# 2858. Minimum Edge Reversals So Every Node Is Reachable

> Source: https://algo.monster/liteproblems/2858

## Problem Statement

You are given a directed [graph](/problems/graph_intro) with `n` nodes (labeled from `0` to `n - 1`) that would form a tree if all edges were bidirectional. The graph is represented by a 2D array `edges`, where `edges[i] = [ui, vi]` indicates a directed edge from node `ui` to node `vi`.

An edge reversal means changing the direction of an edge - turning an edge from `ui → vi` into `vi → ui`.

Your task is to find, for each node `i` (where `0 ≤ i ≤ n - 1`), the minimum number of edge reversals needed so that starting from node `i`, you can reach all other nodes in the [graph](/problems/graph_intro) by following directed edges.

Return an array `answer` where `answer[i]` represents the minimum number of edge reversals required when starting from node `i`.

**Key Points:**
- The [graph](/problems/graph_intro) forms a tree structure (connected and acyclic) if edges were bidirectional
- Each edge is initially directed (one-way)
- You need to calculate the answer independently for each starting node
- The goal is to make all nodes reachable from a given starting node with minimum edge reversals

**Example Understanding:**
If you have edges `[[0,1], [2,0], [3,2]]`, this creates a directed path: `3 → 2 → 0 → 1`. 
- Starting from node 0: You can reach node 1 directly, but to reach nodes 2 and 3, you'd need to reverse edges `2→0` and `3→2`, requiring 2 reversals.
- Starting from node 3: You can already reach all nodes following the existing directions, requiring 0 reversals.

## Intuition

The key insight is that if we know the answer for one node, we can efficiently compute the answer for all other nodes without starting from scratch.

Let's think about what happens when we pick node 0 as the root. We need to count how many edges need to be reversed to reach all nodes from node 0. An edge needs to be reversed if it points "away" from our root (going against the direction we want to traverse).

Now here's the clever part: what happens when we shift our perspective from node 0 to one of its neighbors, say node 1? 

- If the original edge was `0 → 1`, we didn't need to reverse it when starting from 0. But now starting from 1, we'd need to reverse it to reach 0. So the reversal count increases by 1.
- If the original edge was `1 → 0`, we had to reverse it when starting from 0. But now starting from 1, it's already pointing the right way. So the reversal count decreases by 1.

This observation leads to a beautiful relationship: when moving the "root" from node `i` to its adjacent node `j`, the answer changes by exactly ±1 depending on the original edge direction.

The solution brilliantly encodes this by creating a bidirectional representation where:
- Forward edges (`x → y`) are marked with weight `1`
- Reverse edges (`y → x`) are marked with weight `-1`

During the first DFS from node 0, we count edges with negative weight (these are the edges pointing away from 0 that need reversal).

In the second DFS, we propagate the answer using the formula: `ans[j] = ans[i] + k`, where `k` is the edge weight. This elegantly captures the ±1 adjustment:
- If `k = 1`: the edge originally pointed from `i` to `j`, so we need one more reversal when `j` is the root
- If `k = -1`: the edge originally pointed from `j` to `i`, so we need one fewer reversal when `j` is the root

This two-pass approach transforms an O(n²) naive solution (running DFS from each node) into an elegant O(n) solution.

**Learn more about [Depth-First Search](/problems/dfs_intro), [Breadth-First Search](/problems/bfs_intro), [Graph](/problems/graph_intro) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The implementation uses a two-pass DFS strategy with a clever edge weight encoding system.

**Step 1: [Graph](/problems/graph_intro) Construction**

First, we build an adjacency list where each edge is stored bidirectionally with different weights:
```python
g = [[] for _ in range(n)]
for x, y in edges:
    g[x].append((y, 1))   # Original direction: weight = 1
    g[y].append((x, -1))  # Reverse direction: weight = -1
```

This encoding allows us to track edge directions:
- Weight `1`: Edge needs to be reversed if we traverse it in this direction
- Weight `-1`: Edge is already in the correct direction

**Step 2: First DFS - Calculate reversals from node 0**

```python
def dfs(i: int, fa: int):
    for j, k in g[i]:
        if j != fa:
            ans[0] += int(k < 0)  # Count edges that need reversal
            dfs(j, i)
```

Starting from node 0, we traverse the tree and count how many edges point away from the root (have negative weight). These are the edges we'd need to reverse to reach all nodes from node 0.

- `i`: Current node
- `fa`: Parent node (to avoid revisiting)
- `k < 0`: Indicates the edge originally pointed from `j` to `i`, so it needs reversal when starting from node 0

**Step 3: Second DFS - Propagate results to all nodes**

```python
def dfs2(i: int, fa: int):
    for j, k in g[i]:
        if j != fa:
            ans[j] = ans[i] + k
            dfs2(j, i)
```

This phase uses [dynamic programming](/problems/dynamic_programming_intro) to calculate answers for all other nodes based on the result from node 0:

- When moving from node `i` to node `j`:
  - If `k = 1`: The edge originally goes `i → j`, so starting from `j` requires one additional reversal (to reach `i`)
  - If `k = -1`: The edge originally goes `j → i`, so starting from `j` requires one fewer reversal

The formula `ans[j] = ans[i] + k` elegantly captures this adjustment.

**Time Complexity:** O(n) - Each node is visited exactly twice
**Space Complexity:** O(n) - For the adjacency list and recursion stack

## Example Walkthrough

Let's walk through a small example with 4 nodes and edges: `[[0,1], [2,0], [3,2]]`

This creates the directed graph: `3 → 2 → 0 → 1`

**Step 1: Build the Graph**
We create a bidirectional adjacency list with weights:
```
Node 0: [(1, 1), (2, -1)]  // 0→1 original, 2→0 reversed
Node 1: [(0, -1)]          // 0→1 reversed view
Node 2: [(0, 1), (3, -1)]  // 2→0 original, 3→2 reversed
Node 3: [(2, 1)]           // 3→2 original
```

**Step 2: First DFS from Node 0**
Starting at node 0, we traverse and count negative weights (edges needing reversal):
- Visit node 0
  - Check edge to node 1: weight = 1 (positive, no reversal needed)
    - Visit node 1 (no more edges except back to parent)
  - Check edge to node 2: weight = -1 (negative, needs reversal! Count = 1)
    - Visit node 2
      - Check edge to node 3: weight = -1 (negative, needs reversal! Count = 2)
        - Visit node 3 (no more edges except back to parent)

Result: `ans[0] = 2` (need to reverse edges 2→0 and 3→2)

**Step 3: Second DFS to Propagate Results**
Starting from node 0 with ans[0] = 2:
- At node 0, process children:
  - To node 1: `ans[1] = ans[0] + 1 = 2 + 1 = 3`
    - Edge 0→1 exists, so from node 1 we need +1 reversal to reach node 0
  - To node 2: `ans[2] = ans[0] + (-1) = 2 - 1 = 1`
    - Edge 2→0 exists, so from node 2 we need -1 reversal (one less)
    - From node 2, process its child:
      - To node 3: `ans[3] = ans[2] + (-1) = 1 - 1 = 0`
        - Edge 3→2 exists, so from node 3 we need -1 reversal

**Final Answer:** `[2, 3, 1, 0]`

Verification:
- From node 0: Reverse 2→0 and 3→2 to reach all nodes (2 reversals) ✓
- From node 1: Reverse 0→1, 2→0, and 3→2 to reach all nodes (3 reversals) ✓
- From node 2: Reverse 3→2 to reach node 3, already can reach 0→1 (1 reversal) ✓
- From node 3: Can already reach all nodes via 3→2→0→1 (0 reversals) ✓

## Python Solution

```python
class Solution:
    def minEdgeReversals(self, n: int, edges: List[List[int]]) -> List[int]:
        # Initialize result array to store minimum reversals for each node as root
        min_reversals = [0] * n
      
        # Build adjacency list with weighted edges
        # Weight 1: original direction (no reversal needed)
        # Weight -1: reversed direction (reversal needed)
        adjacency_list = [[] for _ in range(n)]
        for source, destination in edges:
            adjacency_list[source].append((destination, 1))   # Original edge direction
            adjacency_list[destination].append((source, -1))   # Reversed edge direction
      
        def calculate_reversals_from_root(node: int, parent: int) -> None:
            """
            First DFS: Calculate the number of edge reversals needed
            when node 0 is the root to reach all other nodes.
            """
            for neighbor, edge_weight in adjacency_list[node]:
                if neighbor != parent:
                    # If edge_weight < 0, we need to reverse this edge
                    min_reversals[0] += int(edge_weight < 0)
                    calculate_reversals_from_root(neighbor, node)
      
        # Calculate reversals needed when node 0 is the root
        calculate_reversals_from_root(0, -1)
      
        def propagate_reversals_to_children(node: int, parent: int) -> None:
            """
            Second DFS: Use the result from node 0 to calculate
            the minimum reversals for all other nodes as roots.
          
            The key insight: when moving root from parent to child,
            the reversal count changes by the edge weight between them.
            """
            for neighbor, edge_weight in adjacency_list[node]:
                if neighbor != parent:
                    # Update reversal count for neighbor based on current node
                    # If edge_weight = 1: moving root from node to neighbor requires one less reversal
                    # If edge_weight = -1: moving root from node to neighbor requires one more reversal
                    min_reversals[neighbor] = min_reversals[node] + edge_weight
                    propagate_reversals_to_children(neighbor, node)
      
        # Propagate the reversal counts to all nodes
        propagate_reversals_to_children(0, -1)
      
        return min_reversals
```

## Time and Space Complexity

**Time Complexity:** `O(n)`

The algorithm consists of two depth-first search (DFS) traversals:
- The first DFS (`dfs`) visits each node exactly once, and for each node, it iterates through all its adjacent edges. Since this is a tree with `n` nodes, there are `n-1` edges, and each edge is stored twice in the adjacency list (once for each direction). This gives us `O(n + 2(n-1)) = O(n)`.
- The second DFS (`dfs2`) also visits each node exactly once and iterates through all adjacent edges, resulting in the same `O(n)` complexity.
- The graph construction loop iterates through all `n-1` edges once, which is `O(n)`.

Overall time complexity: `O(n) + O(n) + O(n) = O(n)`

**Space Complexity:** `O(n)`

The space usage includes:
- The adjacency list `g` which stores `2(n-1)` edge entries total across all nodes: `O(n)`
- The answer array `ans` of size `n`: `O(n)`
- The recursive call stack for DFS, which in the worst case (a linear tree) can go up to depth `n`: `O(n)`

Overall space complexity: `O(n) + O(n) + O(n) = O(n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Edge Weight Assignment**

**Pitfall:** One of the most common mistakes is confusing the edge weight assignment logic. Developers often reverse the weights, assigning `1` to edges that need reversal and `-1` to edges in the correct direction.

**Incorrect Implementation:**
```python
# WRONG: This reverses the meaning of weights
for source, destination in edges:
    adjacency_list[source].append((destination, -1))  # Incorrect!
    adjacency_list[destination].append((source, 1))   # Incorrect!
```

**Why it's wrong:** The weight should represent the cost adjustment when changing the root. When we traverse an edge in its original direction (source → destination), no reversal is needed from the perspective of the original root, but when we make the destination the new root, we'd need to reverse this edge.

**Solution:** Remember that the weight represents the change in reversal count when moving the root across that edge:
```python
# CORRECT: Weight represents the change when moving root
for source, destination in edges:
    adjacency_list[source].append((destination, 1))   # Moving root from source to dest adds 1 reversal
    adjacency_list[destination].append((source, -1))  # Moving root from dest to source removes 1 reversal
```

### 2. **Misunderstanding the Weight's Role in the Second DFS**

**Pitfall:** Incorrectly interpreting what the edge weight means during the propagation phase, leading to wrong formulas like `ans[j] = ans[i] - k` or `ans[j] = ans[i] + abs(k)`.

**Why it happens:** The dual meaning of edge weights can be confusing:
- In the first DFS: `k < 0` indicates an edge that needs reversal from node 0's perspective
- In the second DFS: `k` represents the change in reversal count when moving the root

**Solution:** Understand that in the second DFS, the weight directly represents the adjustment needed:
- If traversing edge `i → j` with weight `k = 1`: The edge goes from i to j originally, so making j the root requires one MORE reversal (to reach i from j)
- If traversing edge `i → j` with weight `k = -1`: The edge originally went from j to i, so making j the root requires one FEWER reversal

### 3. **Forgetting to Handle the Tree Property**

**Pitfall:** Not properly avoiding cycles by forgetting to track the parent node, causing infinite recursion.

**Incorrect Implementation:**
```python
def dfs(node):
    for neighbor, weight in adjacency_list[node]:
        # WRONG: No parent check, will revisit nodes
        ans[0] += int(weight < 0)
        dfs(neighbor)
```

**Solution:** Always pass and check the parent node to avoid revisiting:
```python
def dfs(node, parent):
    for neighbor, weight in adjacency_list[node]:
        if neighbor != parent:  # Essential check
            ans[0] += int(weight < 0)
            dfs(neighbor, node)
```

### 4. **Incorrect Initial Count in First DFS**

**Pitfall:** Counting edge reversals incorrectly by checking `k > 0` instead of `k < 0`, or forgetting the conversion to integer.

**Solution:** Remember that in the first DFS, we count edges that point AWAY from the root (need reversal):
```python
# Correct: Count edges with negative weight (originally pointing toward root)
ans[0] += int(k < 0)
```

The `int()` conversion is crucial because `k < 0` returns a boolean, and we need to add 1 or 0 to our count.

