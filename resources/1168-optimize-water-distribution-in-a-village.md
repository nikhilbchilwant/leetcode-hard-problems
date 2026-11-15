# 1168. Optimize Water Distribution in a Village

> Source: https://algo.monster/liteproblems/1168

## Problem Statement

You have `n` houses in a village that need water supply. To provide water to all houses, you can either build wells directly in houses or connect houses with pipes to share water from existing wells.

For each house `i`, you have two options:
- Build a well directly inside it at a cost of `wells[i - 1]` (using 0-based indexing)
- Connect it to another house that has water supply using pipes

The pipe connections are described in the `pipes` array, where each element `pipes[j] = [house1_j, house2_j, cost_j]` represents:
- A bidirectional pipe can be built between `house1_j` and `house2_j`
- The cost to build this pipe is `cost_j`
- Multiple pipes with different costs can exist between the same two houses

Your goal is to find the minimum total cost to ensure all houses have water supply. Houses can get water either from their own well or through pipe connections to other houses that have water (directly from a well or indirectly through other pipes).

The solution transforms this problem into a [minimum spanning tree](/problems/mst_forest) problem by creating a virtual node (node 0) representing a "super well". Each house's well-building cost becomes an edge weight between that house and node 0. The algorithm then uses Kruskal's approach with Union-Find to find the minimum cost to connect all houses to a water source.

## Intuition

The key insight is recognizing that this is actually a [graph](/problems/graph_intro) connectivity problem in disguise. Each house needs to be connected to a water source, either directly (by building a well) or indirectly (through pipes to other houses with water).

Think of it this way: if we build a well in a house, we're essentially "connecting" that house to a water source. If we lay pipes between houses, we're creating paths for water to flow. The challenge is that wells can be built in multiple houses, making it unclear which houses should have wells and which should rely on pipes.

The breakthrough comes from creating a virtual "super source" node (node 0) that represents all possible wells. Now, instead of deciding whether to build a well in house `i`, we can model it as connecting house `i` to this super source with an edge weight equal to `wells[i-1]`. This transforms the problem elegantly:

- Building a well in house `i` → Connecting house `i` to node 0 with cost `wells[i-1]`
- Laying a pipe between houses → Connecting two house nodes with the pipe cost
- Ensuring all houses have water → Ensuring all house nodes are connected to node 0

With this transformation, the problem becomes finding the minimum cost to connect all nodes in a [graph](/problems/graph_intro) - a classic [Minimum Spanning Tree](/problems/mst_forest) (MST) problem! We need exactly `n` edges to connect `n+1` nodes (n houses + 1 super source), and we want the minimum total cost.

Using Kruskal's algorithm with Union-Find is natural here: we sort all edges by cost and greedily pick the cheapest edges that connect different components until everything is connected. The beauty is that the algorithm automatically decides which houses get wells (direct edges to node 0) and which houses share water through pipes (edges between house nodes).

**Learn more about [Union Find](/problems/dsu_intro), [Graph](/problems/graph_intro), [Minimum Spanning Tree](/problems/mst_forest) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

The solution implements Kruskal's algorithm with Union-Find data structure to find the [minimum spanning tree](/problems/mst_forest). Here's how the implementation works:

**Step 1: Transform the Problem**
```python
for i, w in enumerate(wells, 1):
    pipes.append([0, i, w])
```
We add virtual edges from node 0 (super source) to each house `i` with cost `wells[i-1]`. This transforms building a well into a [graph](/problems/graph_intro) edge problem.

**Step 2: Sort Edges by Cost**
```python
pipes.sort(key=lambda x: x[2])
```
Kruskal's algorithm requires edges sorted by weight in ascending order. This ensures we consider cheaper connections first.

**Step 3: Initialize Union-Find**
```python
p = list(range(n + 1))
```
Create a parent array where `p[i] = i` initially, meaning each node (0 to n) is its own parent. Node 0 represents the super source, nodes 1 to n represent houses.

**Step 4: Find Operation with Path Compression**
```python
def find(x: int) -> int:
    if p[x] != x:
        p[x] = find(p[x])  # Path compression
    return p[x]
```
This finds the root of the component containing node `x`. Path compression optimization makes subsequent finds faster by directly connecting nodes to their root.

**Step 5: Process Edges Using Union-Find**
```python
ans = 0
for a, b, c in pipes:
    pa, pb = find(a), find(b)
    if pa != pb:
        p[pa] = pb  # Union operation
        n -= 1      # One less component
        ans += c    # Add edge cost
        if n == 0:
            return ans
```

For each edge `(a, b, cost)`:
- Find the roots of both endpoints
- If they're in different components (`pa != pb`), merge them:
  - Union: Set one root as parent of the other
  - Decrement `n` (number of components remaining)
  - Add the edge cost to our answer
  - If we've connected all houses (`n == 0`), we're done

The algorithm greedily selects minimum-cost edges that connect different components. Since we need exactly `n` edges to connect `n+1` nodes into one component, we stop when `n` reaches 0. The total cost accumulated is the minimum cost to supply water to all houses.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Given:**
- 3 houses in the village
- `wells = [1, 2, 2]` (costs to build wells in houses 1, 2, 3)
- `pipes = [[1, 2, 1], [2, 3, 1]]` (pipe connections with costs)

**Step 1: Transform the Problem**

First, we create a virtual node 0 (super source) and add edges from it to each house:
- Edge (0, 1) with cost 1 (well in house 1)
- Edge (0, 2) with cost 2 (well in house 2)
- Edge (0, 3) with cost 2 (well in house 3)

Our complete edge list becomes:
```
pipes = [[1, 2, 1], [2, 3, 1], [0, 1, 1], [0, 2, 2], [0, 3, 2]]
```

**Step 2: Sort Edges by Cost**

After sorting by cost (third element):
```
pipes = [[1, 2, 1], [2, 3, 1], [0, 1, 1], [0, 2, 2], [0, 3, 2]]
```

**Step 3: Initialize Union-Find**

Parent array: `p = [0, 1, 2, 3]`
- Each node is its own parent initially
- We have 4 components (nodes 0, 1, 2, 3)
- Counter `n = 3` (we need to connect 3 houses to water)

**Step 4: Process Edges with Kruskal's Algorithm**

**Edge 1: [1, 2, 1]**
- Find parents: `find(1) = 1`, `find(2) = 2`
- Different components, so union them: `p[1] = 2`
- Add cost: `ans = 0 + 1 = 1`
- Decrement counter: `n = 2`
- State: Houses 1 and 2 are connected

**Edge 2: [2, 3, 1]**
- Find parents: `find(2) = 2`, `find(3) = 3`
- Different components, so union them: `p[2] = 3`
- Add cost: `ans = 1 + 1 = 2`
- Decrement counter: `n = 1`
- State: Houses 1, 2, and 3 are all connected

**Edge 3: [0, 1, 1]**
- Find parents: `find(0) = 0`, `find(1) = 3` (through path compression)
- Different components, so union them: `p[0] = 3`
- Add cost: `ans = 2 + 1 = 3`
- Decrement counter: `n = 0`
- **All houses connected to water source! Return ans = 3**

**Result Interpretation:**

The minimum cost is 3, achieved by:
- Building a well in house 1 (edge 0→1, cost 1)
- Connecting house 1 to house 2 with a pipe (cost 1)
- Connecting house 2 to house 3 with a pipe (cost 1)

This gives all houses access to water at minimum total cost of 3.

## Python Solution

```python
class Solution:
    def minCostToSupplyWater(
        self, n: int, wells: List[int], pipes: List[List[int]]
    ) -> int:
        # Union-Find helper function to find the root of a node
        def find_root(node: int) -> int:
            # Path compression: make each node point directly to root
            if parent[node] != node:
                parent[node] = find_root(parent[node])
            return parent[node]

        # Add wells as edges from virtual node 0 to each house
        # wells[i-1] is the cost to build a well at house i
        for house_id, well_cost in enumerate(wells, start=1):
            pipes.append([0, house_id, well_cost])

        # Sort all edges (pipes + wells) by cost for Kruskal's algorithm
        pipes.sort(key=lambda edge: edge[2])

        # Initialize Union-Find parent array
        # parent[i] represents the parent of node i
        parent = list(range(n + 1))

        total_cost = 0
        edges_added = 0

        # Process edges in ascending order of cost (Kruskal's algorithm)
        for source, destination, cost in pipes:
            root_source = find_root(source)
            root_destination = find_root(destination)

            # If nodes are in different components, unite them
            if root_source != root_destination:
                # Union: merge the two components
                parent[root_source] = root_destination
                edges_added += 1
                total_cost += cost

                # Early termination: we need exactly n edges to connect n+1 nodes
                if edges_added == n:
                    return total_cost
```

## Time and Space Complexity

**Time Complexity:** `O((m + n) × log(m + n))`

The time complexity is dominated by the sorting operation. After adding all wells as virtual edges from node 0, we have `m + n` total edges (where `m` is the original number of pipes and `n` is the number of wells). Sorting these `m + n` edges takes `O((m + n) × log(m + n))` time. The Union-Find operations (find and union) with path compression have an amortized time complexity of nearly `O(1)` per operation, and we perform at most `m + n` such operations, contributing `O((m + n) × α(n))` where `α` is the inverse Ackermann function, which is effectively constant for practical purposes. Therefore, the overall time complexity is `O((m + n) × log(m + n))`.

**Space Complexity:** `O(m + n)`

The space complexity consists of several components:
- The modified `pipes` array after adding well connections: `O(m + n)` space
- The parent array `p` for Union-Find: `O(n + 1) = O(n)` space
- The recursion stack for the find operation with path compression: `O(n)` in the worst case

Since we're modifying the input `pipes` array in-place by appending to it, the additional space used is `O(n)` for the parent array and potential recursion stack. However, considering the total space including the modified input, it becomes `O(m + n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect House Indexing When Adding Wells**
A critical pitfall is mishandling the 1-based vs 0-based indexing when adding wells as edges. The problem states houses are numbered 1 to n, but the wells array uses 0-based indexing.

**Incorrect Implementation:**
```python
# Wrong: This creates edges from node 0 to nodes 0,1,2...n-1
for i in range(n):
    pipes.append([0, i, wells[i]])
```

**Correct Implementation:**
```python
# Correct: Creates edges from node 0 to nodes 1,2,3...n
for i, w in enumerate(wells, 1):
    pipes.append([0, i, w])
# OR
for i in range(n):
    pipes.append([0, i + 1, wells[i]])
```

### 2. **Forgetting to Account for the Virtual Node in Parent Array**
The parent array must have size `n + 1` to accommodate nodes 0 through n (virtual node 0 plus n houses).

**Incorrect Implementation:**
```python
# Wrong: Only accounts for n houses, missing the virtual node
parent = list(range(n))
```

**Correct Implementation:**
```python
# Correct: Includes space for nodes 0 to n
parent = list(range(n + 1))
```

### 3. **Wrong Termination Condition**
The algorithm needs exactly n edges to connect n+1 nodes (n houses + 1 virtual node). A common mistake is checking for n-1 edges or using the wrong counter.

**Incorrect Implementation:**
```python
# Wrong: Checking for n-1 edges
if edges_added == n - 1:
    return total_cost

# Wrong: Using the component counter incorrectly
components = n + 1  # Starting with n+1 components
for source, destination, cost in pipes:
    if find(source) != find(destination):
        union(source, destination)
        components -= 1
        if components == 1:  # Wrong: should check if we've added n edges
            return total_cost
```

**Correct Implementation:**
```python
# Correct: Need exactly n edges
edges_added = 0
for source, destination, cost in pipes:
    if find(source) != find(destination):
        union(source, destination)
        edges_added += 1
        total_cost += cost
        if edges_added == n:  # Connected all n houses to virtual node
            return total_cost
```

### 4. **Missing Path Compression in Find Operation**
Without path compression, the find operation can degrade to O(n) time complexity in worst case, making the overall algorithm inefficient.

**Incorrect Implementation:**
```python
# Wrong: No path compression
def find(x):
    while parent[x] != x:
        x = parent[x]
    return x
```

**Correct Implementation:**
```python
# Correct: With path compression
def find(x):
    if parent[x] != x:
        parent[x] = find(parent[x])  # Compress path
    return parent[x]
```

### 5. **Not Handling the Case When Input is Already Connected**
While less common in this specific problem, if the pipes array is modified or pre-processed, ensure the algorithm handles cases where some houses might already be connected.

**Solution:** The Union-Find structure naturally handles this by checking if nodes are already in the same component before adding an edge, preventing redundant connections and incorrect cost calculations.

