# 1627. Graph Connectivity With Threshold

> Source: https://algo.monster/liteproblems/1627

## Problem Statement

You are given `n` cities labeled from `1` to `n`. Two cities are connected by a road if they share a common divisor greater than a given `threshold`. Specifically, cities `x` and `y` are directly connected if there exists an integer `z` such that:
- `x % z == 0` (z divides x)
- `y % z == 0` (z divides y)  
- `z > threshold`

Given `n`, `threshold`, and an array of `queries` where each query is `[ai, bi]`, you need to determine if cities `ai` and `bi` are connected either directly or indirectly (through a path of roads).

The solution uses Union-Find to efficiently group connected cities. The key insight is to iterate through all possible common divisors `a` starting from `threshold + 1` up to `n`. For each divisor `a`, we connect all its multiples within the range `[1, n]` by unioning `a` with `2a`, `3a`, `4a`, and so on. This is done in the nested loop where `b` iterates through multiples of `a` (starting from `a + a` and incrementing by `a`).

After building the connected components, each query `[a, b]` can be answered by checking if cities `a` and `b` belong to the same component using the `find` operation. If `find(a) == find(b)`, the cities are connected and we return `true`; otherwise, we return `false`.

The Union-Find data structure with path compression and union by size ensures efficient operations, making the overall solution time-efficient for handling multiple queries.

## Intuition

The key observation is that if two cities share a common divisor greater than the threshold, they are connected. Moreover, if city `a` is connected to city `b`, and city `b` is connected to city `c`, then `a` and `c` are also connected through `b`. This transitive property suggests we're dealing with connected components in a graph.

Instead of checking every pair of cities to see if they share a common divisor (which would be inefficient), we can think about the problem from the perspective of the divisors themselves. For any divisor `d > threshold`, all multiples of `d` that are ≤ `n` will be connected to each other because they all share `d` as a common divisor.

For example, if `threshold = 2` and `n = 12`, then for divisor `3`, cities `3, 6, 9, 12` all share divisor `3` and should be in the same connected component. Similarly, for divisor `4`, cities `4, 8, 12` should be connected.

This leads us to the approach: for each potential divisor `d` from `threshold + 1` to `n`, we can connect all its multiples. The clever part is that we don't need to connect every pair of multiples directly - we can just connect each multiple to the divisor itself (or connect consecutive multiples). This is because Union-Find will automatically handle the transitive connections.

The Union-Find data structure is perfect for this problem because:
1. We need to build groups of connected cities efficiently
2. We need to quickly query whether two cities are in the same group
3. Cities might be connected through multiple different divisors (e.g., `6` and `12` share both `3` and `6` as common divisors), and Union-Find handles these overlapping groups naturally by merging them

**Learn more about [Union Find](/problems/dsu_intro) and [Math](/problems/math-basics) patterns.**

## Solution Approach

The solution uses the Union-Find (Disjoint Set Union) data structure to efficiently group connected cities and answer connectivity queries.

**Union-Find Implementation:**

The `UnionFind` class maintains two arrays:
- `p`: Parent array where `p[x]` stores the parent of node `x`
- `size`: Size array to track the size of each component for union by size optimization

The `find(x)` method finds the root of the component containing `x` and applies path compression by directly connecting `x` to its root during the traversal. This optimization ensures nearly constant time operations.

The `union(a, b)` method merges two components by connecting the root of the smaller component to the root of the larger one (union by size). This keeps the tree balanced and prevents degradation to a linked list.

**Main Algorithm:**

1. **Initialize Union-Find**: Create a Union-Find structure with `n + 1` elements (to handle 1-indexed cities from `1` to `n`).

2. **Build Connected Components**: 
   - Iterate through all potential common divisors `a` from `threshold + 1` to `n`
   - For each divisor `a`, iterate through its multiples: `b = 2a, 3a, 4a, ...` up to `n`
   - Union `a` with each of its multiples `b` using `uf.union(a, b)`
   - This effectively groups all numbers that share `a` as a common divisor

   The nested loop structure:
   ```python
   for a in range(threshold + 1, n + 1):
       for b in range(a + a, n + 1, a):
           uf.union(a, b)
   ```
 
   For example, when `a = 3`, we union `3` with `6`, `9`, `12`, etc. This connects all multiples of `3`.

3. **Answer Queries**: For each query `[a, b]`, check if cities `a` and `b` are in the same connected component by comparing their roots: `uf.find(a) == uf.find(b)`. Return `True` if they share the same root (connected), `False` otherwise.

**Time Complexity**: 
- Building components: `O(n * log(n))` for iterating through divisors and their multiples
- Answering queries: `O(q * α(n))` where `q` is the number of queries and `α` is the inverse Ackermann function (nearly constant)

**Space Complexity**: `O(n)` for the Union-Find data structure

## Example Walkthrough

Let's walk through a small example with `n = 6`, `threshold = 1`, and queries `[[4, 6], [3, 5], [3, 4]]`.

**Step 1: Initialize Union-Find**
We create a Union-Find structure with parent array `p = [0, 1, 2, 3, 4, 5, 6]` where each city initially points to itself.

**Step 2: Build Connected Components**
We iterate through divisors from `threshold + 1 = 2` to `n = 6`:

- **Divisor 2**: Connect all multiples of 2
  - Union(2, 4): Cities 2 and 4 now share component
  - Union(2, 6): Cities 2, 4, and 6 now share component
  - Component: {2, 4, 6}

- **Divisor 3**: Connect all multiples of 3
  - Union(3, 6): City 3 joins the component containing 6
  - Since 6 was already connected to {2, 4}, we now have {2, 3, 4, 6}

- **Divisor 4**: Connect all multiples of 4
  - No multiples of 4 within range (next would be 8 > 6)

- **Divisor 5**: Connect all multiples of 5
  - No multiples of 5 within range (next would be 10 > 6)

- **Divisor 6**: Connect all multiples of 6
  - No multiples of 6 within range (next would be 12 > 6)

After building, we have two components:
- Component 1: {2, 3, 4, 6} (connected through shared divisors)
- Component 2: {1} (no divisor > 1 divides 1)
- Component 3: {5} (only divisible by 1 and 5, no connections)

**Step 3: Answer Queries**
- Query `[4, 6]`: Find(4) = 2, Find(6) = 2 → Same root → **True**
  - Cities 4 and 6 share divisor 2 (both are even numbers)

- Query `[3, 5]`: Find(3) = 2, Find(5) = 5 → Different roots → **False**
  - Cities 3 and 5 share no divisor greater than 1

- Query `[3, 4]`: Find(3) = 2, Find(4) = 2 → Same root → **True**
  - Cities 3 and 4 are connected through 6 (3 and 6 share divisor 3, while 4 and 6 share divisor 2)

The key insight is that we don't need to find all common divisors between every pair of cities. Instead, by connecting each divisor to all its multiples, the Union-Find structure automatically creates the transitive connections. For instance, even though 3 and 4 don't share a common divisor > 1 directly, they're connected through their mutual connection to 6.

## Python Solution

```python
from typing import List


class UnionFind:
    """
    Union-Find (Disjoint Set Union) data structure with path compression 
    and union by size optimization.
    """
  
    def __init__(self, n: int) -> None:
        """
        Initialize the Union-Find structure with n elements (0 to n-1).
      
        Args:
            n: Number of elements in the structure
        """
        # Each element is initially its own parent (represents n disjoint sets)
        self.parent = list(range(n))
        # Track the size of each component for union by size optimization
        self.component_size = [1] * n
  
    def find(self, x: int) -> int:
        """
        Find the root/representative of the component containing element x.
        Uses path compression to optimize future queries.
      
        Args:
            x: The element whose root we want to find
          
        Returns:
            The root element of the component containing x
        """
        # Path compression: make all nodes on the path point directly to the root
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]
  
    def union(self, a: int, b: int) -> bool:
        """
        Unite the components containing elements a and b.
        Uses union by size to keep the tree balanced.
      
        Args:
            a: First element
            b: Second element
          
        Returns:
            True if the elements were in different components (union performed),
            False if they were already in the same component
        """
        # Find the roots of both elements
        root_a, root_b = self.find(a), self.find(b)
      
        # Already in the same component
        if root_a == root_b:
            return False
      
        # Union by size: attach smaller tree under the root of larger tree
        if self.component_size[root_a] > self.component_size[root_b]:
            self.parent[root_b] = root_a
            self.component_size[root_a] += self.component_size[root_b]
        else:
            self.parent[root_a] = root_b
            self.component_size[root_b] += self.component_size[root_a]
      
        return True


class Solution:
    def areConnected(
        self, n: int, threshold: int, queries: List[List[int]]
    ) -> List[bool]:
        """
        Determine if pairs of numbers share a common divisor greater than threshold.
        Two numbers are connected if they share a common divisor > threshold.
      
        Args:
            n: Range of numbers from 1 to n
            threshold: Minimum value for a valid common divisor
            queries: List of [a, b] pairs to check for connectivity
          
        Returns:
            List of boolean values indicating if each query pair is connected
        """
        # Initialize Union-Find with n+1 elements (to handle 1-indexed numbers)
        union_find = UnionFind(n + 1)
      
        # Connect all numbers that share a common divisor > threshold
        # For each potential divisor greater than threshold
        for divisor in range(threshold + 1, n + 1):
            # Connect all multiples of this divisor
            # Start from 2*divisor and increment by divisor to get all multiples
            for multiple in range(divisor + divisor, n + 1, divisor):
                # Union the divisor with its multiple
                # This creates connected components of numbers sharing common divisors
                union_find.union(divisor, multiple)
      
        # Check each query pair to see if they're in the same component
        # Two numbers are connected if they have the same root in the Union-Find
        return [union_find.find(a) == union_find.find(b) for a, b in queries]
```

## Time and Space Complexity

The time complexity is `O(n × log n × α(n) + q × α(n))`, which can be simplified to `O(n × log n × α(n) + q)` when considering that `α(n)` is nearly constant for practical values of `n`.

Breaking down the time complexity:
- The nested loops in `areConnected` iterate through divisors: for each number `a` from `threshold + 1` to `n`, we iterate through its multiples `b`. This creates approximately `n/1 + n/2 + n/3 + ... + n/n` iterations, which equals `n × (1 + 1/2 + 1/3 + ... + 1/n)` = `O(n × log n)` operations.
- Each `union` operation takes `O(α(n))` time due to path compression in the `find` operation, where `α(n)` is the inverse Ackermann function.
- Processing `q` queries, each requiring two `find` operations, takes `O(q × α(n))` time.

The space complexity is `O(n)`:
- The UnionFind data structure uses two arrays (`p` and `size`), each of size `n + 1`, requiring `O(n)` space.
- No additional significant space is used beyond the input and output.

Here, `n` is the number of nodes, `q` is the number of queries, and `α` is the inverse Ackermann function, which grows extremely slowly and is effectively constant (≤ 4) for all practical values of `n`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Inefficient Connection Strategy**
A common mistake is trying to connect cities by checking every pair of cities to see if they share a common divisor greater than the threshold. This would result in O(n²) comparisons and checking their GCD, leading to time complexity issues.

**Incorrect Approach:**
```python
# Don't do this - inefficient O(n²) approach
for i in range(1, n + 1):
    for j in range(i + 1, n + 1):
        if gcd(i, j) > threshold:
            union_find.union(i, j)
```

**Solution:** Instead of checking all pairs, iterate through potential divisors and connect their multiples. This is more efficient because each number is visited only by its divisors.

### 2. **Off-by-One Errors with Indexing**
Since cities are labeled from 1 to n (1-indexed), but Python arrays are 0-indexed, it's easy to make indexing errors.

**Common Mistake:**
```python
# Wrong: Using n elements for 1-indexed cities
union_find = UnionFind(n)  # This creates indices 0 to n-1
# Later accessing city n would cause IndexError
```

**Solution:** Initialize Union-Find with `n + 1` elements to accommodate 1-indexed cities, effectively ignoring index 0.

### 3. **Incorrect Loop Boundaries for Multiples**
When connecting multiples of a divisor, starting from the wrong value or using wrong increment can miss connections.

**Common Mistake:**
```python
# Wrong: Starting from divisor instead of 2*divisor
for divisor in range(threshold + 1, n + 1):
    for multiple in range(divisor, n + 1, divisor):  # Includes divisor itself
        union_find.union(divisor, multiple)
```

This unnecessarily tries to union a number with itself and can cause confusion.

**Solution:** Start from `divisor + divisor` (which is 2*divisor) to avoid self-unions and ensure we're only connecting distinct multiples.

### 4. **Missing Path Compression in Find Operation**
Implementing find without path compression leads to degraded performance over multiple queries.

**Inefficient Implementation:**
```python
def find(self, x):
    # No path compression - can degrade to O(n) per operation
    while self.parent[x] != x:
        x = self.parent[x]
    return x
```

**Solution:** Use recursive path compression to flatten the tree structure:
```python
def find(self, x):
    if self.parent[x] != x:
        self.parent[x] = self.find(self.parent[x])  # Path compression
    return self.parent[x]
```

### 5. **Forgetting Union by Size/Rank**
Without union by size or rank, the Union-Find tree can become unbalanced, leading to poor performance.

**Poor Implementation:**
```python
def union(self, a, b):
    root_a, root_b = self.find(a), self.find(b)
    if root_a != root_b:
        self.parent[root_a] = root_b  # Always attach a to b
```

**Solution:** Maintain component sizes and always attach the smaller tree to the larger one to keep the tree balanced.

