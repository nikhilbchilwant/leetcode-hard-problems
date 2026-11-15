# 3244. Shortest Distance After Road Addition Queries II

> Source: https://algo.monster/liteproblems/3244

## Problem Statement

You have `n` cities numbered from `0` to `n - 1`. Initially, these cities are connected by unidirectional roads where each city `i` has a road going to city `i + 1` (for all `0 <= i < n - 1`). This forms a simple path from city `0` to city `n - 1` with initial length `n - 1`.

You are given a list of `queries` where each query `queries[i] = [u_i, v_i]` represents adding a new unidirectional road from city `u_i` to city `v_i`. These new roads act as shortcuts that can potentially reduce the distance from city `0` to city `n - 1`.

After processing each query (adding each new road), you need to determine the length of the shortest path from city `0` to city `n - 1`.

The problem guarantees that there are no two queries with overlapping but non-nested intervals. Specifically, there won't be cases where `queries[i][0] < queries[j][0] < queries[i][1] < queries[j][1]`.

Your task is to return an array `answer` where `answer[i]` represents the shortest path length from city `0` to city `n - 1` after processing the first `i + 1` queries.

For example, if `n = 5`, you initially have roads: `0→1→2→3→4`, giving a path length of 4. If you add a road from city `1` to city `3`, you create a shortcut `0→1→3→4`, reducing the path length to 3.

## Intuition

The key insight is that when we add a new road from city `u` to city `v`, we're essentially creating a shortcut that allows us to skip cities `u+1, u+2, ..., v-1`. This shortcut reduces the total path length.

Initially, the shortest path from city `0` to city `n-1` requires visiting every city in sequence, giving us a path length of `n-1`. Each time we add a shortcut, we can potentially reduce this length.

Think of it this way: if we maintain a record of "which city comes next" for each city, we can track the actual path. The array `nxt[i]` tells us the next city we can reach from city `i`. Initially, `nxt[i] = i + 1` because we can only go to the next consecutive city.

When we add a road from `u` to `v`, we update `nxt[u] = v`, meaning from city `u`, we can now jump directly to city `v`. But here's the crucial part: the cities that were previously between `u` and `v` in our path are now bypassed. We need to mark these bypassed cities somehow.

The clever trick is to set `nxt[i] = 0` for all cities that get bypassed. This indicates that these cities are no longer part of any shortest path from city `0` to city `n-1`. 

By counting how many cities remain "active" (not bypassed) in our path, we can maintain the current shortest distance. Each time we bypass a city, we decrease our path length counter by 1.

There's also an optimization: if we already have a shortcut that covers a larger range, adding a smaller shortcut within that range won't help. For instance, if we already have a road from city `1` to city `5`, adding a road from city `2` to city `4` won't reduce the path length further. This is why we check if `0 < nxt[u] < v` before processing a query.

**Learn more about [Greedy](/problems/greedy_intro) and [Graph](/problems/graph_intro) patterns.**

## Solution Approach

Let's walk through the implementation step by step:

**1. Initialize the tracking array:**
We create an array `nxt` where `nxt[i]` represents the next city reachable from city `i`. Initially, `nxt = [1, 2, 3, ..., n-1]`, meaning each city `i` connects to city `i+1`.

**2. Initialize the path length counter:**
We maintain `cnt = n - 1`, which represents the current shortest path length from city `0` to city `n-1`.

**3. Process each query `[u, v]`:**

For each new road from city `u` to city `v`, we first check if this shortcut is meaningful:
- The condition `0 < nxt[u] < v` checks whether:
  - `nxt[u] > 0`: City `u` is still active (not bypassed)
  - `nxt[u] < v`: The new road actually creates a shortcut (jumps further than the current next city)

**4. Update bypassed cities:**
If the shortcut is valid, we need to mark all cities that will be bypassed. Starting from `i = nxt[u]` (the current next city from `u`), we iterate through all cities until we reach `v`:
```python
i = nxt[u]
while i < v:
    cnt -= 1          # Decrease path length for each bypassed city
    nxt[i], i = 0, nxt[i]  # Mark city as bypassed (set to 0) and move to next
```

The clever use of simultaneous assignment `nxt[i], i = 0, nxt[i]` ensures we:
- Save the next city to visit before overwriting it
- Mark the current city as bypassed by setting `nxt[i] = 0`
- Move to the next city in the original path

**5. Update the shortcut:**
After marking all bypassed cities, we set `nxt[u] = v` to establish the new shortcut.

**6. Record the result:**
After processing each query, we append the current `cnt` to our answer array.

**Example walkthrough:**
If `n = 5` and we add a road from city `1` to city `4`:
- Initial: `nxt = [1, 2, 3, 4]`, `cnt = 4`
- Cities `2` and `3` will be bypassed
- We set `nxt[2] = 0`, `nxt[3] = 0`, and `nxt[1] = 4`
- `cnt` decreases by 2 (for the two bypassed cities), becoming `2`
- The new shortest path is: `0 → 1 → 4`, with length 2

This [greedy](/problems/greedy_intro) approach works because once a city is bypassed, it will never be part of any future shortest path, allowing us to maintain the answer efficiently in `O(1)` amortized time per query.

## Example Walkthrough

Let's walk through a concrete example with `n = 6` cities (numbered 0 to 5) and queries `[[0, 3], [2, 5], [0, 4]]`.

**Initial Setup:**
- Cities: `0 → 1 → 2 → 3 → 4 → 5`
- `nxt = [1, 2, 3, 4, 5]` (each city points to the next)
- `cnt = 5` (need 5 steps to go from city 0 to city 5)
- Path length: 5

**Query 1: [0, 3] - Add road from city 0 to city 3**
- Check if shortcut is valid: `0 < nxt[0] < 3` → `0 < 1 < 3` ✓
- Cities to bypass: 1 and 2
  - Start with `i = nxt[0] = 1`
  - While `1 < 3`: Set `nxt[1] = 0`, decrease `cnt` to 4, move to `i = 2`
  - While `2 < 3`: Set `nxt[2] = 0`, decrease `cnt` to 3, move to `i = 3`
  - Stop (3 is not < 3)
- Set `nxt[0] = 3` (create the shortcut)
- New path: `0 → 3 → 4 → 5`
- `nxt = [3, 0, 0, 4, 5]`, `cnt = 3`
- Answer so far: `[3]`

**Query 2: [2, 5] - Add road from city 2 to city 5**
- Check if shortcut is valid: `0 < nxt[2] < 5` → `0 < 0 < 5` ✗
- City 2 is already bypassed (nxt[2] = 0), so this shortcut has no effect
- Path remains: `0 → 3 → 4 → 5`
- `nxt = [3, 0, 0, 4, 5]`, `cnt = 3`
- Answer so far: `[3, 3]`

**Query 3: [0, 4] - Add road from city 0 to city 4**
- Check if shortcut is valid: `0 < nxt[0] < 4` → `0 < 3 < 4` ✓
- Cities to bypass: 3
  - Start with `i = nxt[0] = 3`
  - While `3 < 4`: Set `nxt[3] = 0`, decrease `cnt` to 2, move to `i = 4`
  - Stop (4 is not < 4)
- Set `nxt[0] = 4` (create the shortcut)
- New path: `0 → 4 → 5`
- `nxt = [4, 0, 0, 0, 5]`, `cnt = 2`
- Answer so far: `[3, 3, 2]`

**Final Result:** `[3, 3, 2]`

The algorithm efficiently tracks which cities are active in the shortest path. Once a city is bypassed (marked with 0), it never becomes part of the path again, allowing us to maintain the shortest distance by simply counting active transitions.

## Python Solution

```python
from typing import List

class Solution:
    def shortestDistanceAfterQueries(
        self, n: int, queries: List[List[int]]
    ) -> List[int]:
        # Initialize next_node array where next_node[i] points to the next reachable node from i
        # Initially, each node i points to i+1 (forming a path 0->1->2->...->n-1)
        next_node = list(range(1, n))
      
        # Result array to store shortest distance after each query
        result = []
      
        # Current shortest path length (initially n-1 edges from 0 to n-1)
        current_distance = n - 1
      
        # Process each query which adds a shortcut from u to v
        for start_node, end_node in queries:
            # Check if adding this edge creates a valid shortcut
            # The edge is valid if:
            # 1. next_node[start_node] is not 0 (not already bypassed)
            # 2. next_node[start_node] < end_node (creates an actual shortcut)
            if 0 < next_node[start_node] < end_node:
                # Start from the node that start_node currently points to
                current = next_node[start_node]
              
                # Mark all nodes between start_node and end_node as bypassed
                # These nodes are no longer part of the shortest path
                while current < end_node:
                    # Decrease the distance count for each bypassed edge
                    current_distance -= 1
                    # Mark the node as bypassed (0) and move to the next node
                    temp = next_node[current]
                    next_node[current] = 0
                    current = temp
              
                # Update start_node to directly point to end_node (add the shortcut)
                next_node[start_node] = end_node
          
            # Append the current shortest distance after processing this query
            result.append(current_distance)
      
        return result
```

## Time and Space Complexity

**Time Complexity:** `O(n + q)`

The algorithm initializes a `nxt` array in `O(n)` time. For each query, it processes nodes between positions `u` and `v`. The key insight is that each node can only be "skipped over" (set to 0) at most once throughout all queries. When `nxt[i]` is set to 0, that node is effectively removed from future traversals. Since there are `n-1` nodes that can be skipped (excluding the first node), and each query takes constant time when no nodes are skipped, the total time across all `q` queries is bounded by `O(n + q)`.

**Space Complexity:** `O(n)`

The algorithm uses a `nxt` array of size `n-1` to track the next reachable city from each position, and an `ans` array to store results (which has size `q`). The dominant space usage is the `nxt` array, giving us `O(n)` space complexity. The temporary variables `i`, `u`, `v`, and `cnt` use constant space.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Loop Termination When Marking Bypassed Nodes**

**The Pitfall:**
A common mistake is incorrectly handling the loop that marks bypassed nodes. Developers might write:

```python
# WRONG - This can cause infinite loop or skip nodes
current = next_node[start_node]
while current < end_node:
    current_distance -= 1
    next_node[current] = 0
    current = next_node[current]  # Bug: next_node[current] was just set to 0!
```

The problem here is that after setting `next_node[current] = 0`, we try to use `next_node[current]` to get the next node, but it's now 0, causing the loop to terminate prematurely or behave unexpectedly.

**The Solution:**
Use simultaneous assignment or save the next node before modifying:

```python
# CORRECT - Option 1: Simultaneous assignment
current = next_node[start_node]
while current < end_node:
    current_distance -= 1
    next_node[current], current = 0, next_node[current]

# CORRECT - Option 2: Save next node first
current = next_node[start_node]
while current < end_node:
    current_distance -= 1
    temp = next_node[current]
    next_node[current] = 0
    current = temp
```

### 2. **Not Checking if a Shortcut is Actually Valid**

**The Pitfall:**
Blindly adding every query as a shortcut without checking if it actually improves the path:

```python
# WRONG - Processes invalid shortcuts
for start_node, end_node in queries:
    current = next_node[start_node]
    while current < end_node:
        current_distance -= 1
        next_node[current], current = 0, next_node[current]
    next_node[start_node] = end_node
    result.append(current_distance)
```

This fails when:
- The node is already bypassed (`next_node[start_node] == 0`)
- The shortcut doesn't actually skip any nodes (`next_node[start_node] >= end_node`)

**The Solution:**
Always validate the shortcut before processing:

```python
# CORRECT - Validates shortcut conditions
for start_node, end_node in queries:
    if 0 < next_node[start_node] < end_node:
        # Process valid shortcut
        current = next_node[start_node]
        while current < end_node:
            current_distance -= 1
            next_node[current], current = 0, next_node[current]
        next_node[start_node] = end_node
    result.append(current_distance)
```

### 3. **Off-by-One Error in Initial Setup**

**The Pitfall:**
Incorrectly initializing the `next_node` array size or values:

```python
# WRONG - Creates array of wrong size
next_node = list(range(n))  # Should be range(1, n)

# WRONG - Includes n as a destination
next_node = list(range(1, n+1))  # Node n-1 shouldn't point to n
```

**The Solution:**
Remember that:
- We have n cities numbered 0 to n-1
- City n-1 is the destination (doesn't need a next pointer)
- The array should have n-1 elements

```python
# CORRECT
next_node = list(range(1, n))  # [1, 2, 3, ..., n-1]
```

This creates an array where `next_node[i]` represents the next city from city `i` for `i` in range `[0, n-2]`.

