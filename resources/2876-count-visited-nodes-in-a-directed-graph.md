# 2876. Count Visited Nodes in a Directed Graph

> Source: https://algo.monster/liteproblems/2876

## Problem Statement

You have a directed [graph](/problems/graph_intro) with `n` nodes labeled from `0` to `n - 1`. The graph has exactly `n` directed edges, where each node has exactly one outgoing edge. The graph is represented by a 0-indexed array `edges` where `edges[i]` indicates there is a directed edge from node `i` to node `edges[i]`.

The problem asks you to simulate a specific process for each starting node:
- Start from a node `x`
- Keep following the edges to visit other nodes
- Stop when you reach a node that you've already visited in this current process

Your task is to return an array `answer` where `answer[i]` represents the total number of different nodes you will visit when performing this process starting from node `i`.

For example, if you start from a node and follow edges: `2 → 3 → 4 → 3`, you would visit nodes `2, 3, 4` and then `3` again (which was already visited), so you stop. The count of different nodes visited would be 3 (nodes 2, 3, and 4).

Since each node has exactly one outgoing edge, following the edges will eventually lead to a cycle. The traversal from any starting node will either:
1. Enter directly into a cycle (if the starting node is part of a cycle)
2. Follow a path that eventually leads into a cycle

The challenge is to efficiently compute the number of distinct nodes visited for all possible starting nodes from `0` to `n - 1`.

## Intuition

Since each node has exactly one outgoing edge, the [graph](/problems/graph_intro) structure forms what's called a "functional graph" - following edges from any starting point will eventually lead to a cycle. Think of it like a comet: there's a "tail" (path leading to the cycle) and a "head" (the cycle itself).

The key insight is that once we know the answer for one node, we can use that information to help calculate answers for other nodes. When we traverse from a node `i`, we're essentially walking along a path until we hit a cycle. There are two scenarios we need to handle:

1. **We hit a cycle that hasn't been processed yet**: This means we've discovered a new cycle. Nodes within the cycle will visit all nodes in the cycle (they go round and round), while nodes on the path leading to the cycle will visit all nodes from their position to the cycle, plus all nodes in the cycle.

2. **We hit a node whose answer we already know**: This is great! If we know node `j` visits `ans[j]` nodes, and we're currently at a node that's `k` steps away from `j`, then our current node visits `k + ans[j]` nodes.

To implement this efficiently, we use a `vis` array to track the order in which we visit nodes during our current traversal. When we revisit a node, we can determine:
- If `vis[j] > 0`: We've found a cycle in our current traversal. The cycle length is `current_count - vis[j] + 1`.
- If `ans[j] > 0`: We've reached a node that was already processed in a previous traversal.

By processing each unvisited node exactly once and reusing previously computed results, we achieve an efficient `O(n)` solution. The trick is carefully tracking whether we're inside or outside a cycle, and properly calculating distances based on our position relative to the cycle.

**Learn more about [Graph](/problems/graph_intro), [Memoization](/problems/memoization_intro) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

We use two arrays to track our progress:
- `ans[i]`: stores the final answer for node `i` (number of distinct nodes visited starting from `i`)
- `vis[i]`: temporarily stores the visit order for node `i` during the current traversal (0 means unvisited in current traversal)

For each unprocessed node `i` (where `ans[i] == 0`), we perform a traversal:

1. **First Pass - Mark Visit Order**: 
   - Start from node `i` and follow edges, incrementing a counter `cnt` for each step
   - Mark `vis[j] = cnt` for each visited node `j`
   - Continue until we hit a node that's already been visited in this traversal (`vis[j] > 0`) or has a known answer (`ans[j] > 0`)

2. **Determine Cycle Properties**:
   - If we stopped because `ans[j] > 0`: We've reached a previously processed node
     - `total = cnt + ans[j]` (our path length plus the known answer)
     - `cycle = 0` (no new cycle discovered)
 
   - If we stopped because `vis[j] > 0`: We've found a new cycle
     - `cycle = cnt - vis[j] + 1` (the cycle length)
     - `total = cnt` (total nodes in our complete traversal)

3. **Second Pass - Fill Answers**:
   - Traverse the path again from node `i`
   - For each node on the path:
     - If it's in the cycle: `ans[j] = cycle`
     - If it's outside the cycle: `ans[j] = total`, then decrement `total` for the next node
   - The `max(total, cycle)` ensures nodes in the cycle get the cycle length

The algorithm cleverly handles two cases:
- **Case 1**: When we discover a new cycle, all nodes in the cycle get `cycle` as their answer, while nodes on the tail get progressively larger values as they're further from the cycle
- **Case 2**: When we hit a known node, we can immediately calculate answers by adding distances

Time Complexity: `O(n)` - each node is processed at most twice
Space Complexity: `O(n)` - for the `ans` and `vis` arrays

## Example Walkthrough

Let's walk through a small example with `edges = [1, 2, 3, 1, 2]`:

```
Graph structure:
0 → 1 → 2 → 3 → 1 (cycle: 1→2→3→1)
        ↑
        4
```

**Initial State:**
- `ans = [0, 0, 0, 0, 0]` (all unprocessed)
- `vis = [0, 0, 0, 0, 0]` (reset for each traversal)

**Processing node 0:**

*First Pass - Mark Visit Order:*
- Start at 0, `cnt = 1`, set `vis[0] = 1`
- Move to 1, `cnt = 2`, set `vis[1] = 2`
- Move to 2, `cnt = 3`, set `vis[2] = 3`
- Move to 3, `cnt = 4`, set `vis[3] = 4`
- Move to 1, but `vis[1] = 2 > 0` (already visited!)
- Stop with `cnt = 4`

*Determine Cycle Properties:*
- Since `vis[1] > 0`, we found a cycle
- `cycle = cnt - vis[1] + 1 = 4 - 2 + 1 = 3` (nodes 1, 2, 3 form the cycle)
- `total = cnt = 4`

*Second Pass - Fill Answers:*
- Start at 0 again:
  - Node 0: Not in cycle (vis[0]=1 < vis[1]=2), so `ans[0] = total = 4`, then `total = 3`
  - Node 1: In cycle (vis[1]=2 >= 2), so `ans[1] = max(3, 3) = 3`
  - Node 2: In cycle (vis[2]=3 >= 2), so `ans[2] = max(3, 3) = 3`
  - Node 3: In cycle (vis[3]=4 >= 2), so `ans[3] = max(3, 3) = 3`
- Result: `ans = [4, 3, 3, 3, 0]`, `vis` reset to `[0, 0, 0, 0, 0]`

**Processing node 4:**

*First Pass - Mark Visit Order:*
- Start at 4, `cnt = 1`, set `vis[4] = 1`
- Move to 2, but `ans[2] = 3 > 0` (already processed!)
- Stop with `cnt = 1`

*Determine Cycle Properties:*
- Since `ans[2] > 0`, we hit a known node
- `total = cnt + ans[2] = 1 + 3 = 4`
- `cycle = 0` (no new cycle)

*Second Pass - Fill Answers:*
- Start at 4 again:
  - Node 4: `ans[4] = total = 4`, then `total = 3`
- Result: `ans = [4, 3, 3, 3, 4]`

**Final Answer:** `[4, 3, 3, 3, 4]`

This demonstrates how the algorithm:
1. Discovers the cycle 1→2→3→1 when processing node 0
2. Assigns cycle length (3) to nodes in the cycle
3. Assigns increasing values (4) to nodes outside the cycle based on their distance
4. Reuses the known answer for node 2 when processing node 4, avoiding redundant traversal

## Python Solution

```python
class Solution:
    def countVisitedNodes(self, edges: List[int]) -> List[int]:
        """
        For each node in a functional graph, count how many nodes are visited
        when following edges starting from that node.
      
        Since each node has exactly one outgoing edge, paths either lead to
        a cycle or merge into a cycle.
      
        Args:
            edges: List where edges[i] represents the destination from node i
          
        Returns:
            List where result[i] is the count of visited nodes starting from node i
        """
        n = len(edges)
        result = [0] * n  # Store the final answer for each node
        visit_order = [0] * n  # Track visit order during traversal (0 means unvisited)
      
        for start_node in range(n):
            # Skip if we already computed the result for this node
            if result[start_node]:
                continue
              
            # Phase 1: Traverse and mark visit order until we hit a visited node
            visit_count = 0
            current_node = start_node
          
            while visit_order[current_node] == 0:
                visit_count += 1
                visit_order[current_node] = visit_count
                current_node = edges[current_node]
          
            # Phase 2: Determine cycle length and total path length
            cycle_length = 0
            total_path_length = visit_count + result[current_node]
          
            # If the node we stopped at doesn't have a result yet,
            # it means we found a new cycle
            if result[current_node] == 0:
                # Calculate cycle length: current visit count - visit order of cycle start + 1
                cycle_length = visit_count - visit_order[current_node] + 1
                total_path_length = visit_count
          
            # Phase 3: Assign results to all nodes in the path
            current_node = start_node
          
            while result[current_node] == 0:
                # Nodes get either the full path length or cycle length (if in cycle)
                result[current_node] = max(total_path_length, cycle_length)
                total_path_length -= 1
                current_node = edges[current_node]
      
        return result
```

## Time and Space Complexity

**Time Complexity:** `O(n)`

Each node is visited at most twice in the algorithm:
- Once during the initial traversal to detect cycles (the first while loop)
- Once during the backtracking to assign answer values (the second while loop)

For each starting node `i`, the algorithm follows edges until it either hits a visited node or a node with a computed answer. Since each node can only be the starting point once (due to the `if not ans[i]` check), and each edge is traversed a constant number of times throughout the entire algorithm, the total time complexity is `O(n)`.

**Space Complexity:** `O(n)`

The algorithm uses three arrays:
- `ans`: stores the final answer for each node, size `n`
- `vis`: tracks visitation order during cycle detection, size `n`  
- No recursive calls or additional data structures that grow with input size

Therefore, the space complexity is `O(n)`, where `n` is the length of the array edges.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Not Resetting `visit_order` Between Different Starting Nodes

**The Problem:**
The `visit_order` array is used to track the visiting sequence during traversal from each starting node. A critical mistake is forgetting that this array needs to be "logically reset" for each new traversal. While the code doesn't explicitly reset the entire array (which would be O(n) per starting node), it relies on the fact that `visit_order[node] == 0` means "unvisited in current traversal."

**Why This Breaks:**
If you don't properly handle the visit_order values from previous traversals, you might:
- Incorrectly detect cycles where none exist
- Calculate wrong cycle lengths
- Miss actual cycles because nodes appear "already visited"

**The Solution:**
The current code handles this correctly by only checking `visit_order[current_node] == 0` in the while loop. However, there's an implicit assumption that needs attention: after processing a path, the `visit_order` values remain non-zero. This works because:
1. We only process nodes where `result[node] == 0` (unprocessed nodes)
2. Once a node gets a result, we never traverse from it again
3. The leftover `visit_order` values don't interfere with future traversals

**Better Approach for Clarity:**
```python
# Option 1: Reset visit_order for the current path after processing
visited_nodes = []
while visit_order[current_node] == 0:
    visit_count += 1
    visit_order[current_node] = visit_count
    visited_nodes.append(current_node)
    current_node = edges[current_node]

# ... process the path ...

# Clean up visit_order for these nodes
for node in visited_nodes:
    visit_order[node] = 0
```

**Or Option 2: Use a different tracking mechanism:**
```python
# Use a set for the current path instead of reusing visit_order
current_path = {}
visit_count = 0
current_node = start_node

while current_node not in current_path and result[current_node] == 0:
    visit_count += 1
    current_path[current_node] = visit_count
    current_node = edges[current_node]
```

### Pitfall 2: Incorrect Cycle Length Calculation

**The Problem:**
When calculating `cycle_length = visit_count - visit_order[current_node] + 1`, it's easy to get the formula wrong by:
- Forgetting the `+1`
- Using the wrong node's visit_order
- Confusing visit_count with visit_order

**Example of the Error:**
If we have a path: `0 → 1 → 2 → 3 → 1` (cycle: 1→2→3→1)
- visit_order: [1, 2, 3, 4, ...]
- When we reach node 1 again, visit_count = 4
- Cycle length should be: 4 - 2 + 1 = 3 (nodes 1, 2, 3)
- Common mistake: 4 - 2 = 2 (missing one node!)

**The Solution:**
Always remember that cycle length = (current position - first occurrence position + 1). Draw out small examples to verify your formula.

### Pitfall 3: Mishandling the Two Different Stop Conditions

**The Problem:**
The traversal can stop for two different reasons:
1. We hit a node with a known result (`result[node] > 0`)
2. We hit a node we've seen in the current traversal (`visit_order[node] > 0`)

Mixing up these conditions leads to incorrect total_path_length calculations.

**The Solution:**
Structure your conditions clearly:
```python
if result[current_node] > 0:
    # Case 1: Hit a processed node
    cycle_length = 0
    total_path_length = visit_count + result[current_node]
elif visit_order[current_node] > 0:
    # Case 2: Found a new cycle
    cycle_length = visit_count - visit_order[current_node] + 1
    total_path_length = visit_count
```

This makes it explicit which case you're handling and prevents mixing up the logic.

