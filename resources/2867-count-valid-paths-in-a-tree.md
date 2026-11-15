# 2867. Count Valid Paths in a Tree

> Source: https://algo.monster/liteproblems/2867

## Problem Statement

You have an undirected [tree](/problems/tree_intro) with `n` nodes labeled from `1` to `n`. The tree is defined by a 2D integer array `edges` of length `n - 1`, where each `edges[i] = [ui, vi]` represents an edge between nodes `ui` and `vi`.

Your task is to count the number of **valid paths** in the [tree](/problems/tree_intro). A path `(a, b)` is considered **valid** if it contains **exactly one** prime number among all the node labels in the path from `a` to `b`.

Important details about paths:
- A path `(a, b)` is a sequence of distinct nodes starting at node `a` and ending at node `b`, where every two consecutive nodes in the sequence are connected by an edge in the [tree](/problems/tree_intro)
- Paths `(a, b)` and `(b, a)` are considered the same and should only be counted once
- The path includes both endpoints `a` and `b`

For example, if you have a path from node 2 to node 6 that goes through nodes `2 → 4 → 6`, and only node 2 is prime, then this path is valid because it contains exactly one prime number.

The goal is to return the total count of all such valid paths in the [tree](/problems/tree_intro).

## Intuition

The key insight is that we need paths with **exactly one prime number**. This means we can categorize nodes into two groups: prime nodes and non-prime nodes.

Let's think about what makes a valid path:
- If a path contains exactly one prime, that prime node acts as a "special point" in the path
- All other nodes in the path must be non-prime

This observation leads to an important realization: **every valid path must pass through exactly one prime node**, and this prime node can be either:
1. An endpoint of the path
2. Somewhere in the middle of the path

Now, here's the clever part: if we focus on each prime node individually, we can count all valid paths that include that specific prime. Since each valid path has exactly one prime, we won't double-count any paths.

For a prime node `p`, what are the valid paths involving it?
- **Case 1: `p` is an endpoint** - The path starts at `p` and extends into a connected group of non-prime nodes
- **Case 2: `p` is in the middle** - The path goes through `p`, connecting two different groups of non-prime nodes on either side

This brings us to another insight: non-prime nodes that are connected to each other (without going through any prime node) form a connected component. We can use Union-Find to efficiently group these non-prime nodes together.

Why Union-Find? Because:
- We can quickly merge non-prime nodes that share an edge
- We can instantly know the size of each connected component
- We can identify which component any non-prime node belongs to

The counting strategy becomes:
1. Build connected components of non-prime nodes using Union-Find
2. For each prime node `p`:
   - Look at all its non-prime neighbors
   - Each neighbor belongs to a connected component of size `s`
   - Paths where `p` is an endpoint: add `s` for each component
   - Paths where `p` is in the middle: add `s1 × s2` for each pair of components

This approach is efficient because we process each prime node once and use the precomputed component sizes to count paths in constant time per neighbor.

**Learn more about [Tree](/problems/tree_intro), [Depth-First Search](/problems/dfs_intro), [Math](/problems/math-basics) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The implementation consists of three main components: preprocessing prime numbers, building connected components of non-prime nodes, and counting valid paths.

**Step 1: Prime Number Preprocessing**

We use the Sieve of Eratosthenes to precompute all prime numbers up to `10^5 + 10`:
```python
prime[0] = prime[1] = False  # 0 and 1 are not prime
for i in range(2, mx + 1):
    if prime[i]:
        for j in range(i * i, mx + 1, i):
            prime[j] = False
```
This gives us O(1) lookup to check if any node value is prime.

**Step 2: Build Graph and Union-Find Structure**

First, we create an adjacency list representation of the [tree](/problems/tree_intro):
```python
g = [[] for _ in range(n + 1)]
for u, v in edges:
    g[u].append(v)
    g[v].append(u)
```

Then, we use Union-Find to merge non-prime nodes that are directly connected:
```python
uf = UnionFind(n + 1)
for u, v in edges:
    if prime[u] + prime[v] == 0:  # both nodes are non-prime
        uf.union(u, v)
```

The Union-Find data structure maintains:
- `p[x]`: parent of node x (for finding the root)
- `size[x]`: size of the component if x is a root
- Path compression in `find()` for efficiency
- Union by size in `union()` to keep the [tree](/problems/tree_intro) balanced

**Step 3: Count Valid Paths**

For each prime node `i`, we count paths in two scenarios:

```python
for i in range(1, n + 1):
    if prime[i]:
        t = 0  # accumulator for previously seen component sizes
        for j in g[i]:  # check all neighbors
            if not prime[j]:
                cnt = uf.size[uf.find(j)]  # size of j's component
                ans += cnt  # paths where i is endpoint
                ans += t * cnt  # paths where i is in middle
                t += cnt  # update accumulator
```

The counting logic:
- `cnt`: For each non-prime neighbor `j`, we get the size of its connected component
- `ans += cnt`: These are paths from prime `i` to any node in `j`'s component (prime as endpoint)
- `ans += t * cnt`: These are paths connecting nodes from previously seen components through prime `i` (prime in middle)
- `t += cnt`: We accumulate the total size of components seen so far

**Why This Works:**

1. Each valid path has exactly one prime, so by iterating through all primes and counting paths involving each, we cover all valid paths exactly once

2. The Union-Find ensures that non-prime nodes in the same component are reachable from each other without passing through any prime

3. The accumulator pattern (`t`) elegantly handles counting paths through the prime node by multiplying sizes of different components, avoiding the need for nested loops

**Time Complexity:** O(n × α(n)) where α is the inverse Ackermann function (practically constant)
**Space Complexity:** O(n) for the graph and Union-Find structures

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

Consider a tree with 6 nodes and the following edges:
```
edges = [[1,2], [1,3], [2,4], [3,5], [3,6]]
```

This creates the tree:
```
      1
     / \
    2   3
    |   |\
    4   5 6
```

**Step 1: Identify Prime Nodes**
- Node 1: Not prime
- Node 2: Prime ✓
- Node 3: Prime ✓
- Node 4: Not prime
- Node 5: Prime ✓
- Node 6: Not prime

**Step 2: Build Connected Components of Non-Prime Nodes**

We use Union-Find to merge non-prime nodes that are directly connected:
- Edge [1,2]: Node 2 is prime, so no merge
- Edge [1,3]: Node 3 is prime, so no merge
- Edge [2,4]: Node 2 is prime, so no merge
- Edge [3,5]: Node 3 is prime, so no merge
- Edge [3,6]: Node 3 is prime, so no merge

Result: Each non-prime node (1, 4, 6) is in its own component:
- Component of node 1: {1}, size = 1
- Component of node 4: {4}, size = 1
- Component of node 6: {6}, size = 1

**Step 3: Count Valid Paths for Each Prime Node**

**For prime node 2:**
- Neighbors: 1 (non-prime), 4 (non-prime)
- Initialize t = 0
- Process neighbor 1:
  - Component size = 1
  - Add paths with 2 as endpoint: ans += 1 (path: 2-1)
  - Add paths through 2: ans += 0 × 1 = 0
  - Update t = 0 + 1 = 1
- Process neighbor 4:
  - Component size = 1
  - Add paths with 2 as endpoint: ans += 1 (path: 2-4)
  - Add paths through 2: ans += 1 × 1 = 1 (path: 1-2-4)
  - Update t = 1 + 1 = 2
- Total from node 2: 3 paths

**For prime node 3:**
- Neighbors: 1 (non-prime), 5 (prime), 6 (non-prime)
- Initialize t = 0
- Process neighbor 1:
  - Component size = 1
  - Add paths with 3 as endpoint: ans += 1 (path: 3-1)
  - Add paths through 3: ans += 0 × 1 = 0
  - Update t = 0 + 1 = 1
- Skip neighbor 5 (it's prime)
- Process neighbor 6:
  - Component size = 1
  - Add paths with 3 as endpoint: ans += 1 (path: 3-6)
  - Add paths through 3: ans += 1 × 1 = 1 (path: 1-3-6)
  - Update t = 1 + 1 = 2
- Total from node 3: 3 paths

**For prime node 5:**
- Neighbors: 3 (prime)
- No non-prime neighbors, so no paths counted here
- Total from node 5: 0 paths

**Final Answer:** 3 + 3 + 0 = 6 valid paths

The valid paths are:
1. 1-2 (contains only prime 2)
2. 2-4 (contains only prime 2)
3. 1-2-4 (contains only prime 2)
4. 1-3 (contains only prime 3)
5. 3-6 (contains only prime 3)
6. 1-3-6 (contains only prime 3)

Notice how the algorithm correctly:
- Counts paths where a prime is an endpoint (like 1-2, 2-4, 1-3, 3-6)
- Counts paths where a prime is in the middle (like 1-2-4, 1-3-6)
- Avoids counting invalid paths like 2-1-3 (two primes) or 1-3-5 (two primes)
- Uses the Union-Find component sizes to efficiently count without explicitly enumerating all paths

## Python Solution

```python
from typing import List


class UnionFind:
    """Union-Find (Disjoint Set Union) data structure with path compression and union by size."""
  
    def __init__(self, n: int):
        """Initialize Union-Find with n elements (0 to n-1)."""
        self.parent = list(range(n))  # Each element is initially its own parent
        self.size = [1] * n  # Size of each component
  
    def find(self, x: int) -> int:
        """Find the root of element x with path compression."""
        if self.parent[x] != x:
            # Path compression: make x point directly to root
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]
  
    def union(self, a: int, b: int) -> bool:
        """
        Union two elements a and b by size.
        Returns True if they were in different components, False otherwise.
        """
        root_a, root_b = self.find(a), self.find(b)
      
        # Already in the same component
        if root_a == root_b:
            return False
      
        # Union by size: attach smaller tree under larger tree
        if self.size[root_a] > self.size[root_b]:
            self.parent[root_b] = root_a
            self.size[root_a] += self.size[root_b]
        else:
            self.parent[root_a] = root_b
            self.size[root_b] += self.size[root_a]
      
        return True


# Precompute prime numbers using Sieve of Eratosthenes
MAX_VALUE = 10**5 + 10
is_prime = [True] * (MAX_VALUE + 1)
is_prime[0] = is_prime[1] = False  # 0 and 1 are not prime

# Sieve of Eratosthenes algorithm
for i in range(2, MAX_VALUE + 1):
    if is_prime[i]:
        # Mark all multiples of i as non-prime
        for j in range(i * i, MAX_VALUE + 1, i):
            is_prime[j] = False


class Solution:
    def countPaths(self, n: int, edges: List[List[int]]) -> int:
        """
        Count the number of valid paths where exactly one node is a prime number.
      
        Args:
            n: Number of nodes (labeled from 1 to n)
            edges: List of edges in the tree
      
        Returns:
            Number of valid paths with exactly one prime node
        """
        # Build adjacency list representation of the graph
        adjacency_list = [[] for _ in range(n + 1)]
        union_find = UnionFind(n + 1)
      
        for u, v in edges:
            adjacency_list[u].append(v)
            adjacency_list[v].append(u)
          
            # Union non-prime nodes that are connected
            # is_prime[u] + is_prime[v] == 0 means both are non-prime
            if is_prime[u] + is_prime[v] == 0:
                union_find.union(u, v)
      
        total_paths = 0
      
        # For each prime node, count valid paths passing through it
        for node in range(1, n + 1):
            if is_prime[node]:
                # Count of non-prime nodes connected to current prime node
                accumulated_count = 0
              
                for neighbor in adjacency_list[node]:
                    if not is_prime[neighbor]:
                        # Get size of the non-prime component containing neighbor
                        component_size = union_find.size[union_find.find(neighbor)]
                      
                        # Add paths from this component to the prime node
                        total_paths += component_size
                      
                        # Add paths through prime node connecting this component 
                        # with previously seen non-prime components
                        total_paths += accumulated_count * component_size
                      
                        accumulated_count += component_size
      
        return total_paths
```

## Time and Space Complexity

**Time Complexity:** `O(n × α(n))`

The time complexity breaks down as follows:
- The sieve of Eratosthenes preprocessing takes `O(mx × log(log(mx)))` where `mx = 10^5 + 10`, which is a constant operation independent of input size `n`.
- Building the adjacency list from edges takes `O(n)` since there are at most `n-1` edges in a tree.
- The UnionFind operations: For each edge, we perform a `union` operation if both endpoints are non-prime. Each `union` involves two `find` operations, which take `O(α(n))` amortized time due to path compression, where `α` is the inverse Ackermann function.
- The main counting loop iterates through all `n` nodes. For each prime node, we iterate through its neighbors and perform a `find` operation for non-prime neighbors, taking `O(α(n))` per operation.
- Since the graph is a tree with `n-1` edges, the total number of neighbor checks across all nodes is `O(n)`.
- Overall, the dominant operations are the UnionFind operations which total to `O(n × α(n))`.

**Space Complexity:** `O(n)`

The space complexity consists of:
- The UnionFind structure uses two arrays `p` and `size`, each of size `n+1`, contributing `O(n)`.
- The adjacency list `g` stores all edges twice (undirected graph), using `O(n)` space for a tree.
- The prime sieve array uses `O(mx)` space, but since `mx` is a constant (`10^5 + 10`), this is `O(1)` with respect to input size `n`.
- Therefore, the overall space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Union-Find Implementation for Node Indices**

**The Pitfall:** The most common mistake is misaligning the Union-Find indices with the node labels. Since nodes are labeled from `1` to `n`, but Union-Find typically uses 0-based indexing, developers often create a Union-Find of size `n` instead of `n+1`, leading to index out of bounds errors or incorrect component tracking.

**Example of Wrong Implementation:**
```python
# WRONG: Creates Union-Find with indices 0 to n-1
union_find = UnionFind(n)  
for u, v in edges:
    # This will fail when u or v equals n
    if is_prime[u] + is_prime[v] == 0:
        union_find.union(u, v)  # Index error when node = n
```

**Solution:**
```python
# CORRECT: Creates Union-Find with indices 0 to n (inclusive)
union_find = UnionFind(n + 1)
# Now indices 1 to n can be safely used
```

### 2. **Double Counting Paths Through Prime Nodes**

**The Pitfall:** When counting paths that pass through a prime node, developers might accidentally count the same path multiple times by processing it from both endpoints.

**Example of Wrong Logic:**
```python
# WRONG: May count paths twice
for node in range(1, n + 1):
    if is_prime[node]:
        for neighbor1 in adjacency_list[node]:
            if not is_prime[neighbor1]:
                for neighbor2 in adjacency_list[node]:
                    if not is_prime[neighbor2] and neighbor1 != neighbor2:
                        # This counts path (a,b) and (b,a) separately
                        total_paths += union_find.size[union_find.find(neighbor1)] * \
                                      union_find.size[union_find.find(neighbor2)]
```

**Solution:** Use an accumulator pattern to ensure each pair is counted exactly once:
```python
# CORRECT: Count each pair only once
accumulated_count = 0
for neighbor in adjacency_list[node]:
    if not is_prime[neighbor]:
        component_size = union_find.size[union_find.find(neighbor)]
        # Multiply with previously seen components only
        total_paths += accumulated_count * component_size
        accumulated_count += component_size
```

### 3. **Forgetting to Handle Single-Node Paths**

**The Pitfall:** The problem asks for paths with exactly one prime, but developers might forget that a single prime node by itself is also a valid path (from the node to itself).

**Solution:** The current implementation correctly handles this by adding `component_size` for each non-prime component connected to a prime node. However, if you're counting single-node paths separately, ensure prime nodes themselves are counted:
```python
# If counting single nodes as paths (depends on problem interpretation)
for node in range(1, n + 1):
    if is_prime[node]:
        total_paths += 1  # The prime node itself as a path
```

### 4. **Inefficient Prime Checking Without Preprocessing**

**The Pitfall:** Checking if a number is prime inside the main algorithm without preprocessing leads to massive performance degradation.

**Example of Wrong Approach:**
```python
# WRONG: O(sqrt(n)) check for each query
def is_prime_number(num):
    if num < 2:
        return False
    for i in range(2, int(num**0.5) + 1):
        if num % i == 0:
            return False
    return True

# Using this in the main loop is inefficient
if is_prime_number(node):  # This is called O(n) times
    # process...
```

**Solution:** Always precompute primes using the Sieve of Eratosthenes for O(1) lookup as shown in the original solution.

