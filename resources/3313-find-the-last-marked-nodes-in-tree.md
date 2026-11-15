# 3313. Find the Last Marked Nodes in Tree

> Source: https://algo.monster/liteproblems/3313

## Problem Statement

You have an undirected [tree](/problems/tree_intro) with `n` nodes labeled from `0` to `n - 1`. The tree structure is given through an array `edges` of length `n - 1`, where each `edges[i] = [ui, vi]` represents an edge connecting nodes `ui` and `vi`.

The problem involves a marking process that works as follows:
- Initially, all nodes in the [tree](/problems/tree_intro) are unmarked
- You start by marking exactly one node at time `t = 0`
- Every second after that, any unmarked node that has at least one marked neighbor becomes marked
- This process continues until all nodes are marked

Your task is to determine, for each possible starting node `i`, which node would be the last one to get marked if you began the marking process from node `i`.

The output should be an array `nodes` where `nodes[i]` represents the last node to be marked when starting from node `i`. If there are multiple nodes that get marked at the same final time (multiple valid answers), you can return any one of them.

For example, if you start marking from a node near one end of a long path in the [tree](/problems/tree_intro), the last node to be marked would likely be at the opposite end, as the marking would spread outward from your starting point like a wave, reaching the farthest nodes last.

## Intuition

Let's think about what happens when we start marking from any node. The marking spreads outward like a ripple in water - each second, the marking advances exactly one edge further from all currently marked nodes. This means the last node to be marked will be the one that's farthest away from our starting point.

Now here's the key insight: for any starting node in a [tree](/problems/tree_intro), which node would be farthest away? It must be one of the endpoints of the tree's diameter (the longest path in the tree). Why? Because the diameter endpoints are, by definition, the two nodes that are maximally far apart in the entire tree.

Consider this: if you start from any node that's not on the diameter, you'll eventually reach the diameter and then need to traverse to one of its endpoints. But if you start from a node on the diameter, you'll still end up reaching one of the endpoints last. This is because the diameter represents the "longest stretch" in the [tree](/problems/tree_intro).

Here's where it gets interesting - we don't need to compute the answer for each starting node independently. Since the last marked node must always be one of the two diameter endpoints, we only need to:

1. Find both endpoints of the diameter (let's call them `a` and `b`)
2. For each starting node `i`, determine which endpoint (`a` or `b`) is farther from it

To find the diameter endpoints, we use a well-known property: if we start a DFS from any node, the farthest node we reach is guaranteed to be one endpoint of the diameter. Then, if we start another DFS from that endpoint, the farthest node we reach is the other endpoint.

Once we have both endpoints `a` and `b`, we compute the distance from each node to both endpoints. For any starting node `i`, if `dist[i][a] > dist[i][b]`, then `a` is farther and will be marked last; otherwise, `b` will be marked last.

This elegant approach reduces what seems like an `O(n²)` problem (computing the farthest node for each starting point) to just `O(n)` with three DFS traversals.

**Learn more about [Tree](/problems/tree_intro) and [Depth-First Search](/problems/dfs_intro) patterns.**

## Solution Approach

The implementation follows the intuition we developed, using three DFS traversals to efficiently solve the problem:

**Step 1: Build the Graph Representation**
```python
g = [[] for _ in range(n)]
for u, v in edges:
    g[u].append(v)
    g[v].append(u)
```
We create an adjacency list representation of the [tree](/problems/tree_intro), where `g[i]` contains all neighbors of node `i`.

**Step 2: Define the DFS Function**
```python
def dfs(i: int, fa: int, dist: List[int]):
    for j in g[i]:
        if j != fa:
            dist[j] = dist[i] + 1
            dfs(j, i, dist)
```
This recursive DFS function:
- Takes the current node `i`, its parent `fa` (to avoid revisiting), and a distance array `dist`
- For each neighbor `j` that isn't the parent, it updates `dist[j]` to be one more than `dist[i]`
- Recursively explores from neighbor `j`

**Step 3: Find the First Diameter Endpoint**
```python
dist1 = [-1] * n
dist1[0] = 0
dfs(0, -1, dist1)
a = dist1.index(max(dist1))
```
- Start DFS from node 0 (arbitrary choice)
- Find node `a` with maximum distance from node 0
- Node `a` is guaranteed to be one endpoint of the diameter

**Step 4: Find the Second Diameter Endpoint and Calculate Distances from `a`**
```python
dist2 = [-1] * n
dist2[a] = 0
dfs(a, -1, dist2)
b = dist2.index(max(dist2))
```
- Start DFS from node `a`
- Find node `b` with maximum distance from `a`
- Node `b` is the other endpoint of the diameter
- `dist2` now contains distances from all nodes to node `a`

**Step 5: Calculate Distances from `b`**
```python
dist3 = [-1] * n
dist3[b] = 0
dfs(b, -1, dist3)
```
- Start DFS from node `b`
- `dist3` contains distances from all nodes to node `b`

**Step 6: Determine the Answer for Each Starting Node**
```python
return [a if x > y else b for x, y in zip(dist2, dist3)]
```
For each node `i`:
- If `dist2[i] > dist3[i]`, node `a` is farther from `i`, so return `a`
- Otherwise, node `b` is farther from `i`, so return `b`

**Time Complexity:** `O(n)` - We perform three DFS traversals, each taking `O(n)` time.

**Space Complexity:** `O(n)` - We store the graph adjacency list and three distance arrays, each of size `n`.

The beauty of this solution lies in recognizing that the answer for any starting node must be one of only two possibilities (the diameter endpoints), allowing us to precompute everything we need with just three traversals instead of `n` traversals.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

Consider a tree with 5 nodes and edges: `[[0,1], [1,2], [1,3], [3,4]]`

The tree structure looks like:
```
    0
    |
    1
   / \
  2   3
      |
      4
```

**Step 1: Build the adjacency list**
```
g[0] = [1]
g[1] = [0, 2, 3]
g[2] = [1]
g[3] = [1, 4]
g[4] = [3]
```

**Step 2: First DFS from node 0**
Starting from node 0 with distance 0:
- Visit node 1: dist1[1] = 1
- From node 1, visit node 2: dist1[2] = 2
- From node 1, visit node 3: dist1[3] = 2
- From node 3, visit node 4: dist1[4] = 3

Result: `dist1 = [0, 1, 2, 2, 3]`
The farthest node is 4 with distance 3, so `a = 4` (first diameter endpoint).

**Step 3: Second DFS from node 4**
Starting from node 4 with distance 0:
- Visit node 3: dist2[3] = 1
- From node 3, visit node 1: dist2[1] = 2
- From node 1, visit node 0: dist2[0] = 3
- From node 1, visit node 2: dist2[2] = 3

Result: `dist2 = [3, 2, 3, 1, 0]`
The farthest nodes are 0 and 2, both with distance 3. Let's say we pick `b = 0` (second diameter endpoint).

The diameter of the tree is the path from node 4 to node 0 (length 3).

**Step 4: Third DFS from node 0**
Starting from node 0 with distance 0:
- Visit node 1: dist3[1] = 1
- From node 1, visit node 2: dist3[2] = 2
- From node 1, visit node 3: dist3[3] = 2
- From node 3, visit node 4: dist3[4] = 3

Result: `dist3 = [0, 1, 2, 2, 3]`

**Step 5: Determine the answer for each starting node**
Now we compare distances to determine which endpoint (4 or 0) is farther from each starting node:

- Node 0: dist2[0]=3 (to node 4) vs dist3[0]=0 (to node 0) → 3 > 0, so answer is 4
- Node 1: dist2[1]=2 (to node 4) vs dist3[1]=1 (to node 0) → 2 > 1, so answer is 4
- Node 2: dist2[2]=3 (to node 4) vs dist3[2]=2 (to node 0) → 3 > 2, so answer is 4
- Node 3: dist2[3]=1 (to node 4) vs dist3[3]=2 (to node 0) → 1 < 2, so answer is 0
- Node 4: dist2[4]=0 (to node 4) vs dist3[4]=3 (to node 0) → 0 < 3, so answer is 0

Final answer: `[4, 4, 4, 0, 0]`

This makes intuitive sense:
- Starting from nodes 0, 1, or 2: The marking spreads toward node 4, which is the farthest endpoint
- Starting from node 3: The marking reaches node 4 quickly (distance 1) but takes longer to reach node 0 (distance 2)
- Starting from node 4: Node 0 is the farthest away and gets marked last

## Python Solution

```python
class Solution:
    def lastMarkedNodes(self, edges: List[List[int]]) -> List[int]:
        """
        Find the last marked node for each starting position in a tree.
        Uses the tree diameter endpoints to determine which node is farthest.

        Args:
            edges: List of edges representing an undirected tree

        Returns:
            List where index i contains the last marked node when starting from node i
        """

        def dfs(node: int, parent: int, distances: List[int]) -> None:
            """
            Depth-first search to calculate distances from a starting node.

            Args:
                node: Current node being visited
                parent: Parent node to avoid revisiting
                distances: Array to store distances from the starting node
            """
            for neighbor in adjacency_list[node]:
                if neighbor != parent:
                    distances[neighbor] = distances[node] + 1
                    dfs(neighbor, node, distances)

        # Build the tree structure
        num_nodes = len(edges) + 1
        adjacency_list = [[] for _ in range(num_nodes)]

        for u, v in edges:
            adjacency_list[u].append(v)
            adjacency_list[v].append(u)

        # Step 1: Find one endpoint of the diameter
        # Start DFS from node 0 to find the farthest node
        distances_from_0 = [-1] * num_nodes
        distances_from_0[0] = 0
        dfs(0, -1, distances_from_0)

        # Node 'endpoint_a' is the farthest from node 0
        endpoint_a = distances_from_0.index(max(distances_from_0))

        # Step 2: Find the other endpoint of the diameter
        # Start DFS from endpoint_a to find the farthest node from it
        distances_from_a = [-1] * num_nodes
        distances_from_a[endpoint_a] = 0
        dfs(endpoint_a, -1, distances_from_a)

        # Node 'endpoint_b' is the farthest from endpoint_a (other end of diameter)
        endpoint_b = distances_from_a.index(max(distances_from_a))

        # Step 3: Calculate distances from endpoint_b
        distances_from_b = [-1] * num_nodes
        distances_from_b[endpoint_b] = 0
        dfs(endpoint_b, -1, distances_from_b)

        # Step 4: For each node, determine which endpoint is farther
        # This gives us the last marked node when starting from each position
        result = []
        for dist_to_a, dist_to_b in zip(distances_from_a, distances_from_b):
            if dist_to_a > dist_to_b:
                result.append(endpoint_a)
            else:
                result.append(endpoint_b)

        return result
```

## Time and Space Complexity

**Time Complexity:** `O(n)`

The algorithm performs three depth-first searches (DFS) on the tree:
- First DFS from node 0 to find the farthest node `a`
- Second DFS from node `a` to find the farthest node `b`
- Third DFS from node `b` to compute distances

Each DFS traversal visits every node exactly once and processes each edge twice (once from each direction). Since a tree with `n` nodes has `n-1` edges, each DFS takes `O(n + 2(n-1)) = O(n)` time.

Additionally:
- Building the adjacency list `g` requires iterating through all edges: `O(n-1) = O(n)`
- Finding the maximum distance node using `index(max(dist))`: `O(n)`
- The final list comprehension with `zip`: `O(n)`

Total time complexity: `O(n) + 3 × O(n) + O(n) + O(n) = O(n)`

**Space Complexity:** `O(n)`

The space usage includes:
- Adjacency list `g`: stores all edges, requiring `O(n)` space
- Three distance arrays (`dist1`, `dist2`, `dist3`): each of size `n`, but only two exist simultaneously due to sequential execution, requiring `O(n)` space
- DFS recursion stack: maximum depth is `O(n)` in the worst case (skewed tree)
- Result list: `O(n)` space

Total space complexity: `O(n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Assumption About Multiple Farthest Nodes**

**Pitfall:** A common mistake is assuming that when starting from a node, there could be multiple nodes at the maximum distance that all get marked last simultaneously. This might lead to overthinking the problem and trying to return all such nodes or handling tie-breaking unnecessarily.

**Why it happens:** The problem statement mentions "If there are multiple nodes that get marked at the same final time (multiple valid answers), you can return any one of them," which might mislead developers into thinking they need to track all nodes at maximum distance.

**The Reality:** In a tree structure, one of the key properties is that the farthest node from any given starting point will always be one of the two endpoints of the tree's diameter. While there might be multiple nodes at the same maximum distance in theory, the algorithm correctly identifies that we only need to consider the diameter endpoints.

**Solution:** Trust the diameter property - the algorithm already handles this correctly by only considering the two diameter endpoints as possible answers.

### 2. **Off-by-One Error in Node Count**

**Pitfall:** Incorrectly calculating the number of nodes in the tree.

```python
# Wrong:
num_nodes = len(edges)  # This gives n-1 nodes

# Correct:
num_nodes = len(edges) + 1  # A tree with n nodes has n-1 edges
```

**Why it happens:** It's easy to forget that a tree with `n` nodes has exactly `n-1` edges. Using `len(edges)` directly would give you `n-1` instead of `n`.

**Solution:** Always remember the fundamental property of trees: `number_of_nodes = number_of_edges + 1`

### 3. **Forgetting to Handle the Parent in DFS**

**Pitfall:** Not tracking the parent node during DFS traversal, leading to infinite recursion.

```python
# Wrong - will cause infinite recursion:
def dfs(node: int, distances: List[int]) -> None:
    for neighbor in adjacency_list[node]:
        distances[neighbor] = distances[node] + 1
        dfs(neighbor, distances)  # Will revisit the parent!

# Correct:
def dfs(node: int, parent: int, distances: List[int]) -> None:
    for neighbor in adjacency_list[node]:
        if neighbor != parent:  # Skip the parent node
            distances[neighbor] = distances[node] + 1
            dfs(neighbor, node, distances)
```

**Why it happens:** In an undirected tree, edges are bidirectional. When traversing from node A to node B, B's adjacency list includes A, which would cause the DFS to go back to A infinitely.

**Solution:** Always pass and check the parent parameter in tree DFS to avoid revisiting the node you came from.

### 4. **Using Wrong Distance Arrays for Comparison**

**Pitfall:** Confusing which distance array corresponds to which endpoint when determining the final answer.

```python
# Wrong - using distances from node 0 instead of endpoint distances:
return [endpoint_a if distances_from_0[i] > distances_from_b[i] else endpoint_b
        for i in range(num_nodes)]

# Correct:
return [endpoint_a if distances_from_a[i] > distances_from_b[i] else endpoint_b
        for i in range(num_nodes)]
```

**Why it happens:** After performing three DFS traversals, it's easy to mix up which distance array should be used. The `distances_from_0` array was only used to find the first diameter endpoint and shouldn't be used in the final comparison.

**Solution:** Clearly name your variables and remember that you need distances from both diameter endpoints (not from the arbitrary starting node 0) to determine which endpoint is farther from each node.

### 5. **Misunderstanding the Diameter Finding Algorithm**

**Pitfall:** Thinking that any arbitrary farthest node from any starting point gives you a diameter endpoint.

**Why it happens:** While it's true that starting from *any* node and finding the farthest node gives you one diameter endpoint, some might think this process can be started from any previously found "farthest" node.

**The Correct Process:**
1. Start from ANY node (we use 0) → Find farthest node → This IS a diameter endpoint
2. Start from that diameter endpoint → Find farthest node → This IS the other diameter endpoint
3. Do NOT continue this process further - you now have both endpoints

**Solution:** Understand that the two-step process (arbitrary node → endpoint A → endpoint B) is sufficient and complete for finding the tree diameter.

