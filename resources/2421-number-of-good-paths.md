# 2421. Number of Good Paths

> Source: https://algo.monster/liteproblems/2421

## Problem Statement

You are given a [tree](/problems/tree_intro) with `n` nodes numbered from `0` to `n - 1`, connected by exactly `n - 1` edges. Each node `i` has a value `vals[i]`.

A **good path** is a simple path in the [tree](/problems/tree_intro) that satisfies two conditions:
1. The starting and ending nodes must have the **same value**
2. All nodes along the path (including start and end) must have values **less than or equal to** the value of the starting/ending nodes

In other words, if you walk from one endpoint to the other, the endpoints have the maximum value along the entire path, and both endpoints share this same maximum value.

Your task is to count the total number of distinct good paths in the [tree](/problems/tree_intro).

Important notes:
- A path and its reverse are considered the same (e.g., path `0 -> 1` is the same as `1 -> 0`)
- A single node by itself counts as a valid good path
- The [tree](/problems/tree_intro) is represented as an undirected [graph](/problems/graph_intro) with `edges[i] = [ai, bi]` indicating an edge between nodes `ai` and `bi`

For example, if we have nodes with values `[1, 3, 2, 1, 3]` and appropriate edges, we need to find all paths where:
- Both endpoints have value 1 (and all nodes on the path have values ≤ 1)
- Both endpoints have value 2 (and all nodes on the path have values ≤ 2)  
- Both endpoints have value 3 (and all nodes on the path have values ≤ 3)
- Plus all single nodes count as good paths

## Intuition

The key insight is that for a good path, the endpoints must have the maximum value along the entire path. This suggests we should process nodes in a specific order - from smallest to largest values.

Why does this ordering help? When we process nodes by increasing value, we can guarantee that any previously processed node has a value less than or equal to the current node. This means when we're at node `a` with value `v`, any adjacent node `b` that we've already processed (with value ≤ `v`) can potentially form good paths with `a` as one endpoint.

Here's the crucial observation: if we connect two groups of nodes through an edge where both endpoints have value `v` or less, then any node with value `v` in one group can form a good path with any node with value `v` in the other group. Why? Because:
1. The endpoints have the same value `v`
2. All intermediate nodes were processed earlier, so they have values ≤ `v`

This naturally leads to a Union-Find approach:
- Process nodes in increasing order of their values
- When processing node `a`, look at its neighbors `b` that have been processed (values ≤ `vals[a]`)
- If `a` and `b` are in different connected components, merge them
- Count the good paths formed: if component 1 has `x` nodes with value `vals[a]` and component 2 has `y` nodes with value `vals[a]`, then merging creates `x * y` new good paths

The beauty of this approach is that by processing nodes in sorted order, we ensure that when we're looking for good paths with maximum value `v`, we've already built all the necessary connections between nodes with values ≤ `v`. Each node also counts as its own good path (a path of length 0), so we start with `n` good paths.

**Learn more about [Tree](/problems/tree_intro), [Union Find](/problems/dsu_intro), [Graph](/problems/graph_intro) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

The implementation uses **[Sorting](/problems/sorting_summary) + Union-Find** to efficiently count good paths.

**Step 1: Build the adjacency list**
First, we create an adjacency list `g` to represent the [tree](/problems/tree_intro) structure, making it easy to find neighbors of any node.

**Step 2: Initialize Union-Find structures**
- `p`: parent array for Union-Find, initially `p[i] = i` (each node is its own parent)
- `size`: a dictionary where `size[i][v]` tracks how many nodes with value `v` exist in the connected component with root `i`
- Initially, `size[i][vals[i]] = 1` for each node `i`

**Step 3: Sort and process nodes**
We create pairs of `(vals[i], i)` and sort them. This ensures we process nodes in increasing order of their values.

**Step 4: Process each node and merge components**
For each node `a` with value `v`:
1. Check all neighbors `b` of `a`
2. Skip if `vals[b] > v` (neighbor hasn't been processed yet)
3. Find the root components `pa` and `pb` using path compression
4. If they're in different components (`pa != pb`):
   - Calculate contribution: `size[pa][v] * size[pb][v]`
   - This counts all good paths where one endpoint is a node with value `v` in component `pa` and the other endpoint is a node with value `v` in component `pb`
   - Merge components: set `p[pa] = pb`
   - Update size: `size[pb][v] += size[pa][v]`

**Step 5: Count total good paths**
- Start with `ans = n` (each single node is a good path)
- Add contributions from merging components as we process nodes
- Return the final count

The `find` function implements path compression for efficiency:
```python
def find(x):
    if p[x] != x:
        p[x] = find(p[x])  # Path compression
    return p[x]
```

**Time Complexity**: `O(n log n)` for [sorting](/problems/sorting_summary) + `O(n × α(n))` for Union-Find operations, where `α` is the inverse Ackermann function (practically constant).

**Space Complexity**: `O(n)` for the Union-Find structures and adjacency list.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

Consider a tree with 5 nodes and values `vals = [1, 3, 2, 1, 3]`:
```
    0(1)---1(3)
           /   \
        2(2)   3(1)
               /
            4(3)
```
Edges: `[[0,1], [1,2], [1,3], [3,4]]`

**Step 1: Initial Setup**
- Build adjacency list: `g = {0:[1], 1:[0,2,3], 2:[1], 3:[1,4], 4:[3]}`
- Initialize Union-Find: `p = [0,1,2,3,4]` (each node is its own parent)
- Initialize size tracker: `size = {0:{1:1}, 1:{3:1}, 2:{2:1}, 3:{1:1}, 4:{3:1}}`
- Start with `ans = 5` (each single node is a good path)

**Step 2: Sort nodes by value**
Sorted pairs: `[(1,0), (1,3), (2,2), (3,1), (3,4)]`

**Step 3: Process nodes in order**

**Processing node 0 (value=1):**
- Check neighbor 1: `vals[1]=3 > 1`, skip (not processed yet)
- No merges, `ans = 5`

**Processing node 3 (value=1):**
- Check neighbor 1: `vals[1]=3 > 1`, skip
- Check neighbor 4: `vals[4]=3 > 1`, skip
- No merges, `ans = 5`

**Processing node 2 (value=2):**
- Check neighbor 1: `vals[1]=3 > 2`, skip
- No merges, `ans = 5`

**Processing node 1 (value=3):**
- Check neighbor 0: `vals[0]=1 ≤ 3`, process it
  - `find(0)=0`, `find(1)=1` (different components)
  - Contribution: `size[0][3] × size[1][3] = 0 × 1 = 0`
  - Merge: `p[0]=1`, update `size[1][1] = 1+1 = 2`
- Check neighbor 2: `vals[2]=2 ≤ 3`, process it
  - `find(2)=2`, `find(1)=1` (different components)
  - Contribution: `size[2][3] × size[1][3] = 0 × 1 = 0`
  - Merge: `p[2]=1`
- Check neighbor 3: `vals[3]=1 ≤ 3`, process it
  - `find(3)=3`, `find(1)=1` (different components)
  - Contribution: `size[3][3] × size[1][3] = 0 × 1 = 0`
  - Merge: `p[3]=1`, update `size[1][1] = 2+1 = 3`
- `ans = 5 + 0 = 5`

**Processing node 4 (value=3):**
- Check neighbor 3: `vals[3]=1 ≤ 3`, process it
  - `find(3)=1` (through path compression), `find(4)=4`
  - Different components!
  - Contribution: `size[1][3] × size[4][3] = 1 × 1 = 1`
  - This represents the good path from node 1 to node 4 (both have value 3)
  - Merge: `p[1]=4`, update `size[4][3] = 1+1 = 2`
- `ans = 5 + 1 = 6`

**Final Answer: 6 good paths**
1. Single node 0: `[0]`
2. Single node 1: `[1]`
3. Single node 2: `[2]`
4. Single node 3: `[3]`
5. Single node 4: `[4]`
6. Path from 1 to 4: `[1-3-4]` (both endpoints have value 3, all intermediate nodes have values ≤ 3)

The key insight is that when we process node 4 (value=3), it can connect with node 1 (also value=3) through the already-processed nodes with smaller values, creating one additional good path.

## Python Solution

```python
class Solution:
    def numberOfGoodPaths(self, vals: List[int], edges: List[List[int]]) -> int:
        # Find operation with path compression for Union-Find
        def find(node: int) -> int:
            if parent[node] != node:
                parent[node] = find(parent[node])  # Path compression
            return parent[node]
      
        # Build adjacency list for the graph
        graph = defaultdict(list)
        for node_a, node_b in edges:
            graph[node_a].append(node_b)
            graph[node_b].append(node_a)
      
        # Initialize Union-Find structure
        n = len(vals)
        parent = list(range(n))  # Each node is initially its own parent
      
        # Track count of each value in each component
        # component_value_count[root][value] = count of nodes with that value
        component_value_count = defaultdict(Counter)
        for node_idx, node_val in enumerate(vals):
            component_value_count[node_idx][node_val] = 1
      
        # Start with n good paths (each node forms a path with itself)
        good_paths_count = n
      
        # Process nodes in ascending order of their values
        # This ensures we only connect nodes with values <= current value
        for current_val, current_node in sorted(zip(vals, range(n))):
            # Check all neighbors of current node
            for neighbor in graph[current_node]:
                # Skip neighbors with larger values (they'll be processed later)
                if vals[neighbor] > current_val:
                    continue
              
                # Find roots of both components
                root_current = find(current_node)
                root_neighbor = find(neighbor)
              
                # If they're in different components, merge them
                if root_current != root_neighbor:
                    # Add new good paths formed by connecting nodes with same max value
                    # from both components
                    good_paths_count += (component_value_count[root_current][current_val] * 
                                       component_value_count[root_neighbor][current_val])
                  
                    # Union: merge smaller component into larger
                    parent[root_current] = root_neighbor
                  
                    # Update value count in the merged component
                    component_value_count[root_neighbor][current_val] += component_value_count[root_current][current_val]
      
        return good_paths_count
```

## Time and Space Complexity

**Time Complexity:** `O(n × log n)`

The time complexity is dominated by the following operations:
- Sorting the values along with their indices: `O(n × log n)` where `n` is the number of nodes
- Building the adjacency list from edges: `O(E)` where `E` is the number of edges, and since it's a tree, `E = n - 1`, so this is `O(n)`
- The main loop iterates through all `n` nodes in sorted order: `O(n)`
  - For each node, we iterate through its neighbors (at most `O(n)` edges total across all iterations)
  - The `find` operation uses path compression, which has an amortized time complexity of `O(α(n))` where `α` is the inverse Ackermann function, practically constant
  - Union operations and size updates are `O(1)`

Since the sorting step dominates with `O(n × log n)` and all other operations are at most `O(n × α(n))` which is practically `O(n)`, the overall time complexity is `O(n × log n)`.

**Space Complexity:** `O(n)`

The space complexity consists of:
- Adjacency list `g`: `O(n + E) = O(n)` since `E = n - 1` for a tree
- Parent array `p`: `O(n)`
- Size dictionary with Counter objects: In the worst case, each component could have nodes with different values, but since we're tracking counts per value per component root, this is bounded by `O(n)`
- Sorting uses `O(n)` space for the sorted list of tuples

Therefore, the total space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Incorrect Path Counting When Merging Components

**The Problem:**
When merging two components, a common mistake is to incorrectly calculate the number of new good paths formed. Some might think to add `size[pa][v] + size[pb][v]` or use `(size[pa][v] + size[pb][v]) * (size[pa][v] + size[pb][v] - 1) // 2`.

**Why It's Wrong:**
- Adding the sizes only counts the nodes, not the paths between them
- The combination formula counts paths within the merged component, but we've already counted paths within each individual component in previous iterations

**The Correct Approach:**
Use multiplication: `size[pa][v] * size[pb][v]`. This counts exactly the new paths created between nodes with value `v` from component A and nodes with value `v` from component B.

**Example:**
If component A has 2 nodes with value 3 and component B has 3 nodes with value 3:
- New paths created = 2 × 3 = 6 paths
- Each node in A can form a path with each node in B

### Pitfall 2: Processing Nodes in Wrong Order

**The Problem:**
Processing nodes randomly or in decreasing order of values breaks the algorithm's logic.

**Why It's Wrong:**
The algorithm relies on the invariant that when processing a node with value `v`, all nodes with values less than `v` have already been processed and potentially merged. This ensures that when we check `vals[neighbor] <= v`, we know the neighbor has been processed.

**The Fix:**
Always sort nodes by their values in ascending order:
```python
for current_val, current_node in sorted(zip(vals, range(n))):
```

### Pitfall 3: Forgetting Path Compression

**The Problem:**
Implementing find without path compression:
```python
def find(x):
    while parent[x] != x:
        x = parent[x]
    return x
```

**Why It's Wrong:**
Without path compression, repeated find operations can degrade to O(n) time in the worst case, making the overall algorithm inefficient.

**The Fix:**
Always use path compression:
```python
def find(x):
    if parent[x] != x:
        parent[x] = find(parent[x])  # Compress the path
    return parent[x]
```

### Pitfall 4: Not Updating Component Size After Merge

**The Problem:**
Forgetting to update the size dictionary after merging:
```python
if root_current != root_neighbor:
    good_paths_count += size[root_current][v] * size[root_neighbor][v]
    parent[root_current] = root_neighbor
    # Missing: size[root_neighbor][v] += size[root_current][v]
```

**Why It's Wrong:**
Future merges involving this component will have incorrect counts, leading to wrong results.

**The Fix:**
Always update the size after merging:
```python
size[root_neighbor][current_val] += size[root_current][current_val]
```

### Pitfall 5: Double Counting or Missing Single-Node Paths

**The Problem:**
- Starting with `ans = 0` instead of `ans = n`
- Or trying to count single-node paths during the merge process

**Why It's Wrong:**
Each single node is a valid good path by definition, and these won't be counted during the merge process since merges only count paths between different components.

**The Fix:**
Initialize the answer with n to account for all single-node paths:
```python
good_paths_count = n  # Start with single-node paths
```

