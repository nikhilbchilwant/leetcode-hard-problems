# 1916. Count Ways to Build Rooms in an Ant Colony

> Source: https://algo.monster/liteproblems/1916

## Problem Statement

You're building `n` rooms numbered from `0` to `n-1` for an ant colony. You're given an array `prevRoom` where `prevRoom[i]` tells you which room must be built before room `i`. Room `0` is already built (so `prevRoom[0] = -1`), and once you build a room, it's directly connected to its previous room.

**Key constraints:**
- You can only build one room at a time
- To build room `i`, room `prevRoom[i]` must already be built
- You can only travel between rooms that are already built and connected
- The rooms form a [tree](/problems/tree_intro) structure with room `0` as the root

**Task:** Count how many different orders you can build all the rooms. Return the answer modulo `10^9 + 7`.

**Example visualization:**
If `prevRoom = [-1, 0, 0, 1]`, the structure looks like:
```
    0 (already built)
   / \
  1   2
 /
3
```

Valid build orders include:
- Build 1, then 3, then 2
- Build 1, then 2, then 3  
- Build 2, then 1, then 3

## Intuition

Let's think about this problem step by step. Since we have a [tree](/problems/tree_intro) structure with dependencies, we need to build parent rooms before their children. This naturally suggests a recursive approach starting from the root.

Consider a simple case: if room 0 has two children (rooms 1 and 2), we can build either room 1 first or room 2 first. Both are valid since their parent (room 0) is already built.

Now, what if room 1 has its own subtree with 3 rooms total, and room 2 has a subtree with 2 rooms total? We need to figure out how many ways we can interleave the building of these two subtrees while maintaining their internal ordering.

This is where [combinatorics](/problems/backtracking) comes in. If we have two sequences that must maintain their internal order, the number of ways to interleave them is the binomial coefficient `C(n+m, m)` where `n` and `m` are the lengths of the sequences. This is because we're choosing `m` positions out of `n+m` total positions for one sequence, and the other sequence fills the remaining positions.

For example, if subtree 1 has rooms [A, B] and subtree 2 has rooms [X, Y, Z], we can interleave them as:
- A, B, X, Y, Z
- A, X, B, Y, Z
- A, X, Y, B, Z
- X, A, B, Y, Z
- ... and so on

The total number of such interleavings is `C(2+3, 3) = C(5, 3) = 10`.

When a node has multiple children, we process them one by one. First, we calculate the ways to build the first child's subtree. Then for each subsequent child, we multiply our current result by the number of ways to interleave that child's subtree with all previously processed subtrees. This multiplication principle works because the choices for each interleaving are independent.

The recursive function returns the size of each subtree (including the root), which we need to calculate the correct binomial coefficients when combining subtrees at higher levels of the [tree](/problems/tree_intro).

**Learn more about [Tree](/problems/tree_intro), [Graph](/problems/graph_intro), [Topological Sort](/problems/topo_intro), [Math](/problems/math-basics), [Dynamic Programming](/problems/dynamic_programming_intro) and [Combinatorics](/problems/backtracking) patterns.**

## Solution Approach

The implementation uses a depth-first search (DFS) approach with combinatorial mathematics to count the valid building orders.

**Data Structure Setup:**
- `ingoing`: A dictionary mapping each room to its parent (though not used in the main logic)
- `outgoing`: A dictionary mapping each room to a set of its children
- These dictionaries represent the [tree](/problems/tree_intro) structure as an adjacency list

**Building the [Tree](/problems/tree_intro) Representation:**
```python
for i in range(1, len(prevRoom)):
    ingoing[i].add(prevRoom[i])
    outgoing[prevRoom[i]].add(i)
```
This loop constructs the parent-child relationships from the `prevRoom` array.

**Recursive DFS Function:**
The `recurse(i)` function processes each node and returns the total number of nodes in its subtree (including itself).

For leaf nodes (rooms with no children):
```python
if len(outgoing[i]) == 0:
    return 1
```

For internal nodes, we iterate through all children:
```python
nodes_in_tree = 0
for v in outgoing[i]:
    cn = recurse(v)  # Get size of child's subtree
    if nodes_in_tree != 0:
        ans[0] *= comb(nodes_in_tree + cn, cn)
        ans[0] %= modulo
    nodes_in_tree += cn
```

**Key Algorithm - Combining Subtrees:**
When we have processed some children (with `nodes_in_tree` total nodes) and encounter a new child subtree (with `cn` nodes), we multiply our current answer by `C(nodes_in_tree + cn, cn)`.

This binomial coefficient represents the number of ways to interleave:
- The `nodes_in_tree` nodes from previously processed subtrees
- The `cn` nodes from the current child's subtree

The multiplication works because:
1. Each subtree internally has its own number of valid orderings (calculated recursively)
2. The ways to interleave different subtrees are independent choices
3. By the multiplication principle, total ways = product of all these choices

**Modular Arithmetic:**
Since the answer can be very large, we take modulo `10^9 + 7` after each multiplication to prevent overflow.

**Final Result:**
The function starts with `ans = [1]` (using a list to maintain reference in nested function) and returns `ans[0] % modulo` as the final count of valid building orders.

## Example Walkthrough

Let's walk through the example where `prevRoom = [-1, 0, 0, 1]`.

First, let's visualize the tree structure:
```
    0 (already built)
   / \
  1   2
 /
3
```

**Step 1: Build Tree Representation**
- `outgoing = {0: {1, 2}, 1: {3}, 2: {}, 3: {}}`
- Room 0 has children 1 and 2
- Room 1 has child 3
- Rooms 2 and 3 are leaves

**Step 2: Start DFS from Room 0**
- Call `recurse(0)`
- Room 0 has 2 children: 1 and 2
- Initialize `nodes_in_tree = 0`, `ans = [1]`

**Step 3: Process First Child (Room 1)**
- Call `recurse(1)`
  - Room 1 has 1 child: 3
  - Call `recurse(3)`
    - Room 3 is a leaf, returns 1
  - For room 1: `nodes_in_tree = 0` initially
  - After processing child 3: `nodes_in_tree = 1`
  - Room 1's subtree has 2 nodes total (itself + room 3)
  - Returns 2

- Back in `recurse(0)`:
  - Child 1's subtree has 2 nodes
  - Since `nodes_in_tree = 0` (first child), no multiplication needed
  - Update `nodes_in_tree = 2`

**Step 4: Process Second Child (Room 2)**
- Call `recurse(2)`
  - Room 2 is a leaf, returns 1

- Back in `recurse(0)`:
  - Child 2's subtree has 1 node
  - Now we need to interleave:
    - 2 nodes from room 1's subtree (rooms 1 and 3)
    - 1 node from room 2's subtree (room 2)
  - Calculate: `C(2+1, 1) = C(3, 1) = 3`
  - Update answer: `ans = 1 × 3 = 3`

**Step 5: Final Result**
The function returns 3, meaning there are 3 valid building orders:

1. **Build 1 → Build 3 → Build 2**
   - After building 1, we can access and build 3
   - Finally build 2

2. **Build 1 → Build 2 → Build 3**
   - Build 1 first, then 2
   - Finally build 3 (accessible through 1)

3. **Build 2 → Build 1 → Build 3**
   - Build 2 first
   - Then build 1, finally 3

The key insight: The binomial coefficient `C(3, 1) = 3` tells us there are 3 ways to choose 1 position out of 3 total positions for room 2, while rooms 1 and 3 must maintain their order (1 before 3).

## Python Solution

```python
from typing import List
from collections import defaultdict
from math import comb

class Solution:
    def waysToBuildRooms(self, prevRoom: List[int]) -> int:
        """
        Calculate the number of ways to build rooms given their dependencies.
      
        Args:
            prevRoom: List where prevRoom[i] is the room that must be built before room i
      
        Returns:
            Number of valid build orders modulo 10^9 + 7
        """
        MOD = 10**9 + 7
      
        # Build adjacency lists for the tree structure
        # ingoing[i] contains the parent of room i (not actually used in computation)
        # outgoing[i] contains all children of room i
        ingoing = defaultdict(set)
        outgoing = defaultdict(set)
      
        # Construct the tree edges (room 0 is the root with no parent)
        for room_id in range(1, len(prevRoom)):
            ingoing[room_id].add(prevRoom[room_id])
            outgoing[prevRoom[room_id]].add(room_id)
      
        # Use list to store answer to allow modification in nested function
        result = [1]
      
        def count_subtree_arrangements(room: int) -> int:
            """
            Recursively count arrangements for the subtree rooted at 'room'.
          
            Args:
                room: Current room being processed
          
            Returns:
                Total number of nodes in the subtree rooted at 'room'
            """
            # Base case: leaf node (room with no children)
            if len(outgoing[room]) == 0:
                return 1
          
            total_nodes_in_subtrees = 0
          
            # Process each child subtree
            for child_room in outgoing[room]:
                # Get the size of child's subtree
                child_subtree_size = count_subtree_arrangements(child_room)
              
                # Calculate combinations for merging this subtree with previous ones
                # We need to choose positions for the new subtree nodes among all positions
                if total_nodes_in_subtrees != 0:
                    # Number of ways to interleave the sequences
                    result[0] *= comb(total_nodes_in_subtrees + child_subtree_size, child_subtree_size)
                    result[0] %= MOD
              
                total_nodes_in_subtrees += child_subtree_size
          
            # Return total nodes in this subtree (including current room)
            return total_nodes_in_subtrees + 1
      
        # Start DFS from root (room 0)
        count_subtree_arrangements(0)
      
        return result[0] % MOD
```

## Time and Space Complexity

**Time Complexity:** `O(n²)`

The algorithm performs a depth-first traversal of the tree starting from room 0. For each node visited:
- The recursive function `recurse()` is called exactly once for each of the `n` rooms
- At each node with children, the algorithm computes combinations using `comb(nodes_in_tree + cn, cn)`
- The `comb()` function has a time complexity of `O(n)` in the worst case when computing binomial coefficients
- In the worst case (e.g., a linear tree structure), the sum of all combination computations across all nodes can reach `O(n²)`

Therefore, the overall time complexity is `O(n²)`.

**Space Complexity:** `O(n)`

The space usage consists of:
- `ingoing` dictionary: stores at most `n-1` entries (one parent per room except room 0), using `O(n)` space
- `outgoing` dictionary: stores all parent-child relationships, totaling `n-1` edges, using `O(n)` space
- Recursive call stack: in the worst case (linear tree), the recursion depth can be `O(n)`
- `ans` list: stores a single value, using `O(1)` space

The dominant factor is the recursive call stack and the dictionaries, resulting in `O(n)` space complexity.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Integer Overflow in Combinatorial Calculations**

The most critical pitfall in this solution is handling large combinatorial values. The `comb(n, k)` function can produce extremely large numbers that exceed standard integer limits, especially when dealing with large tree structures.

**The Problem:**
```python
# This can cause overflow or incorrect results
result[0] *= comb(total_nodes_in_subtrees + child_subtree_size, child_subtree_size)
result[0] %= MOD
```

When `comb()` returns a very large number, multiplying it with `result[0]` before taking modulo can cause overflow issues in some implementations or languages.

**Solution:**
Implement a modular arithmetic version of the combination function:

```python
def comb_mod(n, k, mod):
    """Calculate C(n,k) % mod using modular arithmetic"""
    if k > n or k < 0:
        return 0
    if k == 0 or k == n:
        return 1
  
    # Calculate using modular arithmetic to prevent overflow
    numerator = 1
    denominator = 1
  
    for i in range(min(k, n - k)):
        numerator = (numerator * (n - i)) % mod
        denominator = (denominator * (i + 1)) % mod
  
    # Use modular inverse for division
    return (numerator * pow(denominator, mod - 2, mod)) % mod
```

### 2. **Incorrect Tree Construction**

Another common mistake is misunderstanding the parent-child relationships when building the adjacency list.

**The Problem:**
```python
# Wrong direction - treating prevRoom[i] as child of i
outgoing[i].add(prevRoom[i])  # INCORRECT!
```

**Solution:**
Always remember that `prevRoom[i]` is the parent of room `i`, not the other way around:
```python
# Correct: prevRoom[i] is the parent, so i is a child of prevRoom[i]
outgoing[prevRoom[i]].add(i)
```

### 3. **Not Handling Single Child Optimization**

The current code has an unnecessary check that could be simplified:

**The Problem:**
```python
if total_nodes_in_subtrees != 0:
    result[0] *= comb(total_nodes_in_subtrees + child_subtree_size, child_subtree_size)
```

This check prevents multiplication on the first child, but `comb(0 + cn, cn)` equals 1 anyway, making the check redundant.

**Solution:**
Remove the conditional for cleaner code:
```python
# Always multiply - when total_nodes_in_subtrees is 0, comb(cn, cn) = 1
result[0] = (result[0] * comb(total_nodes_in_subtrees + child_subtree_size, 
                              child_subtree_size)) % MOD
```

### 4. **Using Mutable Default Arguments**

While not present in this code, a common Python pitfall when using `defaultdict` is accidentally sharing mutable defaults across function calls.

**Solution:**
Always create new instances inside the function rather than using default parameters:
```python
# Good practice
def waysToBuildRooms(self, prevRoom: List[int]) -> int:
    ingoing = defaultdict(set)  # Fresh instance each call
    outgoing = defaultdict(set)
```

