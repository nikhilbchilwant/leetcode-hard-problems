# 2603. Collect Coins in a Tree

> Source: https://algo.monster/liteproblems/2603

## Problem Statement

You have an undirected [tree](/problems/tree_intro) with `n` nodes labeled from `0` to `n - 1`. The tree structure is given by an array `edges` where `edges[i] = [ai, bi]` represents an edge between nodes `ai` and `bi`.

Each node may or may not have a coin, represented by an array `coins` where:
- `coins[i] = 1` means node `i` has a coin
- `coins[i] = 0` means node `i` has no coin

You start at any node of your choice in the [tree](/problems/tree_intro). From your current position, you can:
1. **Collect coins**: Gather all coins that are within distance 2 from your current node (including the current node itself)
2. **Move**: Travel to any adjacent node

Your goal is to collect all the coins in the [tree](/problems/tree_intro) and return to your starting position. You need to find the **minimum number of edges** you must traverse to accomplish this task.

**Important notes:**
- You can collect coins up to distance 2 away without moving to those nodes
- Each edge traversal counts toward your total (if you cross the same edge multiple times, count each traversal)
- You must return to your starting position after collecting all coins

For example, if you're at node A and there's a coin at node B that's 2 edges away, you can collect that coin without moving to node B. However, if there's a coin at node C that's 3 edges away, you'd need to move closer to collect it.

The challenge is to find the optimal path that minimizes the total number of edge traversals while collecting all coins and returning to the start.

## Intuition

Let's think about what parts of the [tree](/problems/tree_intro) we actually need to visit. Since we can collect coins from distance 2, we don't need to physically visit every node that has a coin - we just need to get close enough.

First observation: **Leaf nodes without coins are useless**. If a leaf node has no coin, there's no reason to go there. We can remove these nodes from consideration. But this creates new leaf nodes, which might also be coinless and useless. So we can repeatedly remove coinless leaf nodes until we're left with a [tree](/problems/tree_intro) where every leaf has a coin.

Second observation: **We don't need to visit leaf nodes with coins either!** Since we can collect coins from distance 2, if we reach a node that's 2 edges away from a leaf with a coin, we can collect that coin without going further. This means we can "trim" the last layer of leaves from our [tree](/problems/tree_intro).

But wait - after removing one layer of leaves, we get new leaves. These new leaves are originally at distance 1 from the old leaves. So if we're at a node that's distance 1 from these new leaves, we're actually at distance 2 from the original leaves with coins. This means **we can trim another layer of leaves**.

After trimming these two layers of leaves (all of which have coins after our first pruning step), we're left with the "core" of the [tree](/problems/tree_intro) - the minimal set of nodes we must actually visit to be within collecting distance of all coins.

The final insight: In any [tree](/problems/tree_intro) traversal where we need to visit certain edges and return to start, we must traverse each required edge exactly twice (once going away from start, once coming back). So the answer is simply `2 × (number of edges in the trimmed tree)`.

The algorithm becomes:
1. Remove all coinless leaf nodes recursively
2. Trim 2 layers of remaining leaves (these all have coins)
3. Count edges in the remaining [tree](/problems/tree_intro) and multiply by 2

**Learn more about [Tree](/problems/tree_intro), [Graph](/problems/graph_intro) and [Topological Sort](/problems/topo_intro) patterns.**

## Solution Approach

The implementation uses **topological sorting** to progressively trim the [tree](/problems/tree_intro) from its leaves inward. Here's how the solution works step by step:

**Step 1: Build the adjacency list**
```python
g = defaultdict(set)
for a, b in edges:
    g[a].add(b)
    g[b].add(a)
```
We use a dictionary of sets to represent the [graph](/problems/graph_intro). Each node maps to a set of its neighbors, making it easy to add/remove connections and check degree.

**Step 2: Remove coinless leaf nodes**
```python
q = deque(i for i in range(n) if len(g[i]) == 1 and coins[i] == 0)
while q:
    i = q.popleft()
    for j in g[i]:
        g[j].remove(i)
        if coins[j] == 0 and len(g[j]) == 1:
            q.append(j)
    g[i].clear()
```
- Initialize a queue with all leaf nodes (`len(g[i]) == 1`) that have no coins (`coins[i] == 0`)
- Process each node in the queue:
  - Remove it from its neighbor's adjacency list
  - If the neighbor becomes a coinless leaf after removal, add it to the queue
  - Clear the processed node's adjacency list
- This continues until no more coinless leaves exist

**Step 3: Trim two layers of remaining leaves**
```python
for k in range(2):
    q = [i for i in range(n) if len(g[i]) == 1]
    for i in q:
        for j in g[i]:
            g[j].remove(i)
        g[i].clear()
```
- Run the trimming process twice
- Each iteration finds all current leaf nodes and removes them
- No need to check for coins here since all remaining leaves have coins after Step 2
- After two iterations, we've removed nodes that are within distance 2 of the original leaves

**Step 4: Count remaining edges**
```python
return sum(len(g[a]) > 0 and len(g[b]) > 0 for a, b in edges) * 2
```
- Check each original edge `[a, b]`
- If both endpoints still exist in the trimmed [graph](/problems/graph_intro) (`len(g[node]) > 0`), the edge is part of our required traversal
- Multiply by 2 because we traverse each edge twice (going and returning)

The time complexity is `O(n)` since we process each node and edge a constant number of times. The space complexity is `O(n)` for storing the adjacency list.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

Consider a tree with 7 nodes and the following structure:
```
Initial Tree:
      3(coin)
     /|\
    / | \
   /  |  \
  0   4   5(coin)
 /|   |
1 2   6(coin)
```

- Edges: `[[0,1], [0,2], [0,3], [3,4], [3,5], [4,6]]`
- Coins: `[0, 0, 0, 1, 0, 1, 1]` (nodes 3, 5, and 6 have coins)

**Step 1: Remove coinless leaf nodes**

Initial leaf nodes: 1, 2, 5, 6
- Node 1: leaf without coin → remove
- Node 2: leaf without coin → remove  
- Node 5: leaf with coin → keep
- Node 6: leaf with coin → keep

After removing nodes 1 and 2:
```
      3(coin)
       |\
       | \
       |  \
       4   5(coin)
       |
       6(coin)
```

Now node 0 becomes a coinless leaf → remove it

After removing node 0:
```
    3(coin)
     |\
     | \
     4  5(coin)
     |
     6(coin)
```

No more coinless leaves exist.

**Step 2: Trim first layer of leaves**

Current leaves: nodes 5 and 6
Remove these leaves:
```
    3(coin)
     |
     4
```

**Step 3: Trim second layer of leaves**

Current leaf: node 4
Remove this leaf:
```
    3(coin) (isolated node)
```

**Step 4: Count remaining edges**

Check original edges:
- `[0,1]`: node 0 removed, node 1 removed → edge not in final tree
- `[0,2]`: node 0 removed, node 2 removed → edge not in final tree
- `[0,3]`: node 0 removed → edge not in final tree
- `[3,4]`: node 4 removed → edge not in final tree
- `[3,5]`: node 5 removed → edge not in final tree
- `[4,6]`: node 4 removed, node 6 removed → edge not in final tree

Remaining edges: 0
Answer: 0 × 2 = 0

This makes sense! Starting from node 3, we can collect:
- The coin at node 3 (distance 0)
- The coin at node 5 (distance 1) 
- The coin at node 6 (distance 2, via path 3→4→6)

Since we can collect all coins without moving from node 3, we need 0 edge traversals.

## Python Solution

```python
class Solution:
    def collectTheCoins(self, coins: List[int], edges: List[List[int]]) -> int:
        # Build adjacency list representation of the tree
        graph = defaultdict(set)
        for node_a, node_b in edges:
            graph[node_a].add(node_b)
            graph[node_b].add(node_a)
      
        n = len(coins)
      
        # Step 1: Remove all leaf nodes that don't have coins
        # Initialize queue with leaf nodes (degree 1) that have no coins
        queue = deque(node for node in range(n) 
                     if len(graph[node]) == 1 and coins[node] == 0)
      
        # Keep removing leaf nodes without coins
        while queue:
            current_node = queue.popleft()
            # Remove this node from its neighbor's adjacency list
            for neighbor in graph[current_node]:
                graph[neighbor].remove(current_node)
                # If neighbor becomes a leaf and has no coin, add to queue
                if coins[neighbor] == 0 and len(graph[neighbor]) == 1:
                    queue.append(neighbor)
            # Clear the current node's connections
            graph[current_node].clear()
      
        # Step 2: Remove two layers of leaf nodes
        # This accounts for the collection distance of 2
        for layer in range(2):
            # Find all current leaf nodes
            leaf_nodes = [node for node in range(n) if len(graph[node]) == 1]
            # Remove each leaf node
            for leaf in leaf_nodes:
                for neighbor in graph[leaf]:
                    graph[neighbor].remove(leaf)
                graph[leaf].clear()
      
        # Step 3: Count remaining edges that need to be traversed
        # An edge is counted if both its endpoints still exist in the graph
        # Multiply by 2 because we need to traverse each edge twice (forward and back)
        remaining_edges = sum(len(graph[node_a]) > 0 and len(graph[node_b]) > 0 
                              for node_a, node_b in edges)
      
        return remaining_edges * 2
```

## Time and Space Complexity

**Time Complexity:** `O(n)`

The algorithm consists of three main phases:

1. **Building the graph:** Creating the adjacency list from edges takes `O(n)` time since there are `n-1` edges in a tree with `n` nodes.

2. **First BFS - Removing leaf nodes without coins:** 
   - Finding initial leaves without coins takes `O(n)` time
   - Each node is processed at most once in the queue
   - For each node, we iterate through its neighbors (at most `O(n)` operations total across all nodes)
   - Total: `O(n)`

3. **Two iterations of leaf removal:**
   - Each iteration finds all current leaves in `O(n)` time
   - Removes these leaves and updates neighbors in `O(n)` time total
   - Two iterations: `2 * O(n) = O(n)`

4. **Final edge counting:** Iterating through all edges to count valid ones takes `O(n)` time.

Overall time complexity: `O(n) + O(n) + O(n) + O(n) = O(n)`

**Space Complexity:** `O(n)`

The space usage includes:
- Graph adjacency list `g`: `O(n)` space for storing all edges
- Queue `q`: At most `O(n)` elements
- Temporary list in the second phase: `O(n)` in worst case
- Input storage for `coins` and `edges`: `O(n)`

Overall space complexity: `O(n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Incorrectly Handling Edge Cases with Few Nodes

**The Problem:**
The algorithm assumes there are enough nodes to form a meaningful tree structure. When the tree has very few nodes (n ≤ 2), the trimming process might remove all nodes, leading to incorrect results.

**Example Scenario:**
- Tree with 2 nodes: `[0]---[1]`, coins = `[1, 1]`
- After removing coinless leaves (none), we trim 2 layers
- First trim removes both nodes (they're both leaves)
- Result: 0 edges, but we should traverse the edge twice to collect both coins

**Solution:**
Add an early return for small trees:
```python
def collectTheCoins(self, coins: List[int], edges: List[List[int]]) -> int:
    n = len(coins)
  
    # Handle edge cases
    if n <= 1:
        return 0
    if n == 2:
        return 2 if any(coins) else 0
  
    # Rest of the algorithm...
```

### Pitfall 2: Modifying Sets During Iteration

**The Problem:**
When removing nodes from the graph, iterating over a set while modifying it can cause runtime errors or skip elements.

**Incorrect Code:**
```python
# This might cause issues
for neighbor in graph[current_node]:  # Iterating over the set
    graph[neighbor].remove(current_node)  # Modifying during iteration
```

**Solution:**
Convert the set to a list before iteration or use a copy:
```python
# Option 1: Convert to list
for neighbor in list(graph[current_node]):
    graph[neighbor].remove(current_node)

# Option 2: Use a copy
for neighbor in graph[current_node].copy():
    graph[neighbor].remove(current_node)
```

### Pitfall 3: Not Handling Disconnected Components

**The Problem:**
If all coins are removed during the trimming process (or the tree becomes disconnected), the algorithm might return 0 when it should handle the case differently.

**Example Scenario:**
- A long chain where all coins are at the leaves
- After trimming, no edges remain
- The algorithm returns 0, which is correct only if we can collect all coins from the starting position

**Solution:**
Check if any coins remain after trimming:
```python
# After all trimming is done
if not any(len(graph[i]) > 0 for i in range(n) if coins[i] == 1):
    # All coins were within collection distance from leaves
    # Verify this is the intended behavior for your use case
    return 0
```

### Pitfall 4: Confusion About Collection Distance

**The Problem:**
Developers might misunderstand that "distance 2" means we can collect coins from nodes up to 2 edges away WITHOUT moving to those nodes. This affects how many layers to trim.

**Common Mistake:**
- Trimming only 1 layer thinking we need to be within distance 1 to collect
- Or trimming 3 layers thinking we need an extra buffer

**Solution:**
Document clearly and stick to exactly 2 trim iterations:
```python
# Trim exactly 2 layers - this accounts for collection distance of 2
# After trimming, remaining nodes require actual traversal
for _ in range(2):  # Exactly 2, not 1 or 3
    leaf_nodes = [node for node in range(n) if len(graph[node]) == 1]
    # ... trimming logic
```

