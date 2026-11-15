# 1724. Checking Existence of Edge Length Limited Paths II

> Source: https://algo.monster/liteproblems/1724

## Problem Statement

You are given an undirected [graph](/problems/graph_intro) with `n` nodes and a list of edges. Each edge is represented as `[u, v, dis]`, where `u` and `v` are nodes connected by an edge with distance `dis`. The graph may have multiple edges between the same pair of nodes and may not be fully connected.

You need to implement a class `DistanceLimitedPathsExist` with two methods:

1. **Constructor** `DistanceLimitedPathsExist(int n, int[][] edgeList)`: Initializes the class with the [graph](/problems/graph_intro) structure, where `n` is the number of nodes and `edgeList` contains all the edges.

2. **Query Method** `boolean query(int p, int q, int limit)`: Returns `true` if there exists a path from node `p` to node `q` where every edge on the path has a distance **strictly less than** `limit`. Returns `false` otherwise.

The solution uses a Persistent Union-Find data structure to efficiently handle queries. The approach works as follows:

1. **Initialization Phase**:
   - Sort all edges by their distances in ascending order
   - Process edges one by one, union-ing nodes with each edge's distance as a "version" timestamp
   - The Persistent Union-Find maintains a history of when nodes became connected

2. **Query Phase**:
   - For a query with limit `limit`, the Persistent Union-Find checks if nodes `p` and `q` belong to the same component when considering only edges with distance less than `limit`
   - This is done by finding the root of each node at "time" `limit` - if they share the same root, a valid path exists

## Intuition

The key observation is that we need to answer multiple queries about path existence with different distance limits. A naive approach would be to rebuild the [graph](/problems/graph_intro) for each query using only edges below the limit, then check connectivity - but this would be inefficient.

Let's think about what happens as we gradually increase the distance limit from 0 to infinity. Initially, no nodes are connected (limit = 0). As we increase the limit, edges start becoming "valid" for use in paths. When an edge with distance `d` becomes valid, it potentially connects two previously disconnected components.

This suggests a timeline perspective: at each "moment" (distance value), the [graph](/problems/graph_intro)'s connectivity structure changes. If we sort edges by distance and process them in order, we're essentially watching the graph evolve from completely disconnected to its final form.

For a query asking "can we go from `p` to `q` with edges < `limit`?", we're really asking: "are `p` and `q` in the same connected component when we only consider edges with distance < `limit`?"

This naturally leads to using Union-Find, which excels at tracking connected components. But standard Union-Find would require rebuilding for each query. The breakthrough is realizing we can make Union-Find "persistent" - recording WHEN each union operation happened.

By tagging each union with the edge's distance as a timestamp, we create a historical record. During a query with limit `L`, we can ask the Union-Find to show us the state of connectivity "at time `L`" - effectively traveling back in time to see which nodes were connected using only edges with distance < `L`.

This transforms the problem from repeatedly building graphs to a single preprocessing step (sorting and union operations) followed by efficient O(log n) queries using the persistent structure.

**Learn more about [Union Find](/problems/dsu_intro), [Graph](/problems/graph_intro) and [Minimum Spanning Tree](/problems/mst_forest) patterns.**

## Solution Approach

The solution implements a **Persistent Union-Find** data structure to efficiently handle connectivity queries with distance constraints.

### Persistent Union-Find Implementation

The `PersistentUnionFind` class maintains three key arrays:
- `p[]`: Parent pointers for each node (standard Union-Find)
- `rank[]`: Rank/height of each tree for union by rank optimization
- `version[]`: Stores the "timestamp" (edge distance) when a node's parent was changed

**Find Operation with Time Travel:**
```python
def find(self, x, t=inf):
    if self.p[x] == x or self.version[x] >= t:
        return x
    return self.find(self.p[x], t)
```
The find operation takes an additional parameter `t` (time/distance limit). It stops traversing up the parent chain if:
1. We reach a root node (`self.p[x] == x`)
2. We encounter a parent link created at or after time `t` (`self.version[x] >= t`)

This effectively shows us the component structure as it existed before time `t`.

**Union Operation with Versioning:**
```python
def union(self, a, b, t):
    pa, pb = self.find(a), self.find(b)
    if pa == pb:
        return False
    if self.rank[pa] > self.rank[pb]:
        self.version[pb] = t
        self.p[pb] = pa
    else:
        self.version[pa] = t
        self.p[pa] = pb
        if self.rank[pa] == self.rank[pb]:
            self.rank[pb] += 1
```
When uniting two components, the algorithm:
1. Finds current roots of both nodes
2. Uses union by rank to decide which root becomes the parent
3. **Crucially**: Records the timestamp `t` in the version array for the node that changes parent

### Main Class Implementation

**Initialization:**
```python
def __init__(self, n: int, edgeList: List[List[int]]):
    self.puf = PersistentUnionFind(n)
    edgeList.sort(key=lambda x: x[2])
    for u, v, dis in edgeList:
        self.puf.union(u, v, dis)
```
1. Creates a Persistent Union-Find with `n` nodes
2. **Sorts edges by distance** in ascending order - this is critical
3. Processes edges sequentially, using each edge's distance as its timestamp

The sorting ensures that when we query with limit `L`, all edges with distance < `L` have been processed in chronological order.

**Query Operation:**
```python
def query(self, p: int, q: int, limit: int) -> bool:
    return self.puf.find(p, limit) == self.puf.find(q, limit)
```
To check if nodes `p` and `q` are connected using only edges with distance < `limit`:
1. Find the root of `p` at time `limit`
2. Find the root of `q` at time `limit`
3. If they have the same root, they're in the same component

The beauty of this approach is that each query runs in O(α(n)) time (nearly constant), where α is the inverse Ackermann function, while preprocessing takes O(E log E) time for sorting the edges.

## Example Walkthrough

Let's walk through a small example to illustrate how the Persistent Union-Find solution works.

**Given:**
- 4 nodes (labeled 0, 1, 2, 3)
- Edges: `[[0,1,2], [1,2,4], [2,3,1], [0,3,5]]`

**Step 1: Initialization and Sorting**

First, we sort edges by distance:
```
[2,3,1] -> distance 1
[0,1,2] -> distance 2
[1,2,4] -> distance 4
[0,3,5] -> distance 5
```

Initial state - all nodes are their own parent:
```
Node:     0  1  2  3
Parent:   0  1  2  3
Version:  ∞  ∞  ∞  ∞
```

**Step 2: Processing Edges (Building History)**

Process edge `[2,3,1]` with timestamp 1:
- Find(2) = 2, Find(3) = 3 (different components)
- Union them: make 3's parent = 2
```
Node:     0  1  2  3
Parent:   0  1  2  2
Version:  ∞  ∞  ∞  1  ← Node 3 changed parent at time 1
```

Process edge `[0,1,2]` with timestamp 2:
- Find(0) = 0, Find(1) = 1 (different components)
- Union them: make 1's parent = 0
```
Node:     0  1  2  3
Parent:   0  0  2  2
Version:  ∞  2  ∞  1  ← Node 1 changed parent at time 2
```

Process edge `[1,2,4]` with timestamp 4:
- Find(1) = 0, Find(2) = 2 (different components)
- Union them: make 2's parent = 0
```
Node:     0  1  2  3
Parent:   0  0  0  2
Version:  ∞  2  4  1  ← Node 2 changed parent at time 4
```

Process edge `[0,3,5]` with timestamp 5:
- Find(0) = 0, Find(3) = 0 (already in same component!)
- No union needed

**Step 3: Query Examples**

**Query(2, 3, limit=3):** Can we go from node 2 to 3 using edges < 3?
- Find(2, time=3):
  - Check parent[2] = 0, but version[2] = 4 ≥ 3
  - So ignore this link and return 2
- Find(3, time=3):
  - Check parent[3] = 2, version[3] = 1 < 3 ✓
  - Continue to Find(2, time=3) = 2
- Result: Find(2,3) = 2, Find(3,3) = 2 → **TRUE** (same component)

**Query(0, 2, limit=3):** Can we go from node 0 to 2 using edges < 3?
- Find(0, time=3): Returns 0 (it's a root)
- Find(2, time=3):
  - Check parent[2] = 0, but version[2] = 4 ≥ 3
  - So ignore this link and return 2
- Result: Find(0,3) = 0, Find(2,3) = 2 → **FALSE** (different components)

**Query(0, 2, limit=5):** Can we go from node 0 to 2 using edges < 5?
- Find(0, time=5): Returns 0
- Find(2, time=5):
  - Check parent[2] = 0, version[2] = 4 < 5 ✓
  - Continue to Find(0, time=5) = 0
- Result: Find(0,5) = 0, Find(2,5) = 0 → **TRUE** (same component)

The key insight: by tracking WHEN each parent link was created, we can "time travel" to see the connectivity state at any distance threshold without rebuilding the graph!

## Python Solution

```python
from typing import List
from math import inf


class PersistentUnionFind:
    """
    A persistent union-find data structure that maintains version history.
    Each union operation is associated with a timestamp/version.
    """

    def __init__(self, n: int) -> None:
        """
        Initialize the persistent union-find structure.

        Args:
            n: Number of elements (nodes) in the structure
        """
        # Rank array for union by rank optimization
        self.rank: List[int] = [0] * n
        # Parent array - initially each element is its own parent
        self.parent: List[int] = list(range(n))
        # Version/timestamp when each element was last modified
        # Initialize with infinity to indicate no modifications yet
        self.version: List[float] = [inf] * n

    def find(self, x: int, timestamp: float = inf) -> int:
        """
        Find the root of element x at a given timestamp.

        Args:
            x: The element to find the root for
            timestamp: The version/timestamp to query at (default: infinity for latest)

        Returns:
            The root element of x at the given timestamp
        """
        # If x is its own parent, or the modification happened after the query timestamp,
        # then x is the root at this timestamp
        if self.parent[x] == x or self.version[x] >= timestamp:
            return x
        # Otherwise, recursively find the parent at the given timestamp
        return self.find(self.parent[x], timestamp)

    def union(self, a: int, b: int, timestamp: float) -> bool:
        """
        Union two elements at a given timestamp using union by rank.

        Args:
            a: First element to union
            b: Second element to union
            timestamp: The timestamp/version for this union operation

        Returns:
            True if union was performed, False if elements were already connected
        """
        # Find roots of both elements
        root_a = self.find(a)
        root_b = self.find(b)

        # If already in the same set, no union needed
        if root_a == root_b:
            return False

        # Union by rank - attach smaller rank tree under root of higher rank tree
        if self.rank[root_a] > self.rank[root_b]:
            # Make root_a the parent of root_b
            self.version[root_b] = timestamp
            self.parent[root_b] = root_a
        else:
            # Make root_b the parent of root_a
            self.version[root_a] = timestamp
            self.parent[root_a] = root_b
            # If ranks were equal, increment the rank of the new root
            if self.rank[root_a] == self.rank[root_b]:
                self.rank[root_b] += 1

        return True


class DistanceLimitedPathsExist:
    """
    Data structure to answer queries about whether two nodes are connected
    using only edges with distance less than a given limit.
    """

    def __init__(self, n: int, edgeList: List[List[int]]) -> None:
        """
        Initialize the data structure with nodes and edges.

        Args:
            n: Number of nodes
            edgeList: List of edges, each edge is [node1, node2, distance]
        """
        # Create persistent union-find structure
        self.persistent_union_find = PersistentUnionFind(n)

        # Sort edges by distance (ascending order)
        edgeList.sort(key=lambda edge: edge[2])

        # Process edges in order of increasing distance
        # Union nodes with the edge distance as timestamp
        for u, v, distance in edgeList:
            self.persistent_union_find.union(u, v, distance)

    def query(self, p: int, q: int, limit: int) -> bool:
        """
        Check if nodes p and q are connected using only edges with distance < limit.

        Args:
            p: First node
            q: Second node
            limit: Distance limit for edges

        Returns:
            True if p and q are connected with edges having distance < limit
        """
        # Two nodes are connected if they have the same root
        # when considering only edges with distance < limit
        return self.persistent_union_find.find(p, limit) == self.persistent_union_find.find(q, limit)
```

## Time and Space Complexity

**Time Complexity:**

- **Initialization (`__init__`):**
  - Creating the PersistentUnionFind takes `O(n)` time to initialize arrays
  - Sorting the edgeList takes `O(m log m)` time where `m` is the number of edges
  - Performing union operations for all edges takes `O(m × α(n))` time, where `α(n)` is the inverse Ackermann function (practically constant)
  - Total initialization time: `O(n + m log m + m × α(n))` = `O(n + m log m)`

- **Query operation:**
  - Each query calls `find` twice with a time limit parameter
  - The `find` operation with time limit traverses the parent chain until it finds a root or a version that exceeds the time limit
  - In the worst case, this takes `O(n)` time per find operation (when the tree forms a long chain)
  - Per query time complexity: `O(n)`

**Space Complexity:**

- **PersistentUnionFind class:**
  - `rank` array: `O(n)` space
  - `p` (parent) array: `O(n)` space
  - `version` array: `O(n)` space
  - Total: `O(n)` space

- **DistanceLimitedPathsExist class:**
  - Stores a reference to PersistentUnionFind: `O(n)` space
  - The sorted edgeList is created temporarily during initialization but not stored
  - Total space: `O(n)`

- **Overall space complexity:** `O(n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Handling of Multiple Edges Between Same Nodes**

**The Pitfall:**
The graph may contain multiple edges between the same pair of nodes with different distances. A common mistake is assuming that we should only keep the edge with minimum distance between any two nodes, leading to incorrect preprocessing like:

```python
# WRONG: Deduplicating edges by keeping only minimum distance
edge_map = {}
for u, v, dis in edgeList:
    key = (min(u,v), max(u,v))
    if key not in edge_map or edge_map[key] > dis:
        edge_map[key] = dis
edgeList = [[u, v, d] for (u,v), d in edge_map.items()]
```

**Why It's Wrong:**
While it seems logical to only keep the minimum distance edge, the Persistent Union-Find actually handles multiple edges correctly by processing them in sorted order. When we union nodes that are already connected, the `union` method returns `False` and doesn't modify the structure. The version timestamp of their first connection is preserved, which is exactly what we want.

**The Solution:**
Simply process all edges as given without deduplication:
```python
def __init__(self, n: int, edgeList: List[List[int]]) -> None:
    self.persistent_union_find = PersistentUnionFind(n)
    edgeList.sort(key=lambda edge: edge[2])  # Just sort, no deduplication
    for u, v, distance in edgeList:
        self.persistent_union_find.union(u, v, distance)
```

### 2. **Using Wrong Comparison in Query (< vs <=)**

**The Pitfall:**
Misunderstanding the problem requirement and using `<=` instead of `<` for the limit comparison:

```python
# WRONG: Including edges with distance equal to limit
def query(self, p: int, q: int, limit: int) -> bool:
    return self.persistent_union_find.find(p, limit + 1) == self.persistent_union_find.find(q, limit + 1)
```

**Why It's Wrong:**
The problem specifically asks for edges with distance **strictly less than** the limit. Using `limit + 1` or any adjustment that includes edges with distance equal to `limit` will produce incorrect results.

**The Solution:**
Use the limit directly as the timestamp in the find operation:
```python
def query(self, p: int, q: int, limit: int) -> bool:
    return self.persistent_union_find.find(p, limit) == self.persistent_union_find.find(q, limit)
```

### 3. **Path Compression Breaking Persistence**

**The Pitfall:**
Implementing standard path compression in the find operation, which modifies the parent pointers:

```python
# WRONG: Path compression destroys version history
def find(self, x: int, timestamp: float = inf) -> int:
    if self.parent[x] != x and self.version[x] < timestamp:
        self.parent[x] = self.find(self.parent[x], timestamp)  # Path compression
    return self.parent[x]
```

**Why It's Wrong:**
Path compression modifies the tree structure by making nodes point directly to the root. This destroys the version history we carefully maintained. Future queries with different timestamps would get incorrect results because the intermediate parent relationships have been lost.

**The Solution:**
Never modify the parent pointers during find operations. Accept the slightly higher time complexity in exchange for correctness:
```python
def find(self, x: int, timestamp: float = inf) -> int:
    if self.parent[x] == x or self.version[x] >= timestamp:
        return x
    return self.find(self.parent[x], timestamp)  # No path compression
```

### 4. **Forgetting to Handle Disconnected Components**

**The Pitfall:**
Assuming all nodes will eventually be connected and not properly initializing isolated nodes:

```python
# WRONG: Not initializing version array properly
def __init__(self, n: int) -> None:
    self.version: List[float] = [0] * n  # Wrong initial value
```

**Why It's Wrong:**
Nodes that never get connected to anything should never match with other components at any timestamp. Using 0 or any finite value as initial version could cause incorrect matches in queries.

**The Solution:**
Initialize version array with infinity to ensure unconnected nodes stay independent:
```python
def __init__(self, n: int) -> None:
    self.version: List[float] = [inf] * n  # Correct initialization
```

