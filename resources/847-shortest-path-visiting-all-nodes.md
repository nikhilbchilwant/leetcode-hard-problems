# 847. Shortest Path Visiting All Nodes

> Source: https://algo.monster/liteproblems/847

## Problem Statement

You are given an undirected, connected [graph](/problems/graph_intro) with `n` nodes labeled from `0` to `n - 1`. The graph is represented as an adjacency list where `graph[i]` contains a list of all nodes that are directly connected to node `i` by an edge.

Your task is to find the **shortest path length that visits every node in the [graph](/problems/graph_intro)**. The path must visit all nodes at least once, but it can:
- Start and end at any node (they don't have to be the same)
- Revisit nodes multiple times if needed
- Reuse edges multiple times

For example, if you have 4 nodes with connections forming a [graph](/problems/graph_intro), you need to find the minimum number of edges to traverse such that you visit all 4 nodes at least once. The path doesn't need to be a simple path - you can backtrack and revisit nodes as necessary to reach all nodes with the minimum total distance.

## Intuition

The key challenge in this problem is that we need to visit all nodes while finding the shortest path. This isn't a standard shortest path problem because:

1. We're not going from point A to point B - we need to visit **every** node
2. We can revisit nodes and edges multiple times
3. We can start from any node

The first insight is that we need to track **two pieces of information** as we explore: which node we're currently at AND which nodes we've visited so far. A path that reaches node 3 having visited nodes {0, 1, 2} is different from reaching node 3 having only visited {0, 3}.

Since we need the **shortest** path and the [graph](/problems/graph_intro) is unweighted (each edge has cost 1), BFS is the natural choice. But how do we handle the "visit all nodes" requirement?

This is where **[bitmask](/problems/bitmask_intro)** comes in. With `n` nodes, we can use an n-bit integer where bit `i` represents whether node `i` has been visited. For example, with 4 nodes:
- `0001` means only node 0 visited
- `0101` means nodes 0 and 2 visited  
- `1111` means all 4 nodes visited (our goal!)

The state space becomes `(current_node, visited_bitmask)`. We use BFS to explore all possible states, and when we reach any state where the [bitmask](/problems/bitmask_intro) equals `(1 << n) - 1` (all bits set), we've found the shortest path that visits all nodes.

Why start from all nodes simultaneously? Since we can start from any node, instead of running BFS `n` times (once from each starting node), we initialize the queue with all possible starting states: `(i, 1 << i)` for each node `i`. This multi-source BFS efficiently finds the optimal solution regardless of which starting node leads to the shortest path.

The `visited_states` set prevents revisiting the same state `(node, bitmask)`, ensuring we don't get stuck in infinite loops while allowing us to revisit nodes with different visit patterns.

**Learn more about [Breadth-First Search](/problems/bfs_intro), [Graph](/problems/graph_intro), [Dynamic Programming](/problems/dynamic_programming_intro) and [Bitmask](/problems/bitmask_intro) patterns.**

## Solution Approach

The solution implements a multi-source BFS with state tracking using a [bitmask](/problems/bitmask_intro). Let's walk through the implementation step by step:

**1. Initialization:**
```python
num_nodes = len(graph)
queue = deque()
visited_states = set()
```
- `num_nodes` stores the number of nodes
- `queue` is a deque for BFS traversal
- `visited_states` is a set to track visited states (not just nodes, but `(node, bitmask)` pairs)

**2. Multi-source BFS Setup:**
```python
for node in range(num_nodes):
    initial_state = 1 << node
    queue.append((node, initial_state))
    visited_states.add((node, initial_state))
```
- For each node, we add the initial state `(node, 1 << node)` to the queue
- `1 << node` creates a bitmask with only that node's bit set (e.g., node 2 → `0100`)
- This allows us to start from any node simultaneously

**3. BFS Level-by-Level Traversal:**
```python
steps = 0
while True:
    for _ in range(len(queue)):
        current_node, visited_mask = queue.popleft()
```
- `steps` tracks the current distance (number of edges traversed)
- The inner loop processes all states at the current distance level
- Each state consists of the current node `current_node` and the visited bitmask `visited_mask`

**4. Goal Check:**
```python
if visited_mask == (1 << num_nodes) - 1:
    return steps
```
- `(1 << num_nodes) - 1` creates a bitmask with all `n` bits set (e.g., for n=4: `1111`)
- If the current state's bitmask equals this, we've visited all nodes
- Return the current distance as the answer

**5. State Expansion:**
```python
for neighbor in graph[current_node]:
    new_visited_mask = visited_mask | (1 << neighbor)
    if (neighbor, new_visited_mask) not in visited_states:
        visited_states.add((neighbor, new_visited_mask))
        queue.append((neighbor, new_visited_mask))
```
- For each `neighbor` of `current_node`
- Create the new bitmask by setting the neighbor's bit: `visited_mask | (1 << neighbor)`
- If this new state hasn't been visited, add it to the queue
- This ensures we explore all possible paths while avoiding redundant states

**6. Distance Increment:**
```python
steps += 1
```
- After processing all states at the current level, increment the distance

The algorithm guarantees finding the shortest path because:
- BFS explores states level by level (shortest distances first)
- The first time we dequeue a state with all nodes visited, its level is the shortest possible path length
- The state tracking prevents infinite loops while allowing node revisits with different visit patterns
- The graph is connected, so a full-mask state is always reached and the `while True` loop always returns

## Example Walkthrough

Let's walk through a small example with 4 nodes to illustrate the solution approach. Bitmasks are written with node 3 on the left and node 0 on the right.

**Graph:**
```
Node 0 connects to: [1, 2]
Node 1 connects to: [0, 2, 3]
Node 2 connects to: [0, 1]
Node 3 connects to: [1]
```

Visualized:
```
    0 --- 1 --- 3
     \   /
      \ /
       2
```

**Step 1: Initialization**
- Create queue with all starting states:
  - State (0, 0001) - at node 0, visited {0}
  - State (1, 0010) - at node 1, visited {1}
  - State (2, 0100) - at node 2, visited {2}
  - State (3, 1000) - at node 3, visited {3}
- `steps = 0`

**Step 2: Process Level 0 (`steps = 0`)**
- Process (0, 0001): not all nodes visited (0001 ≠ 1111)
  - Add (1, 0011) and (2, 0101) to queue
- Process (1, 0010): not all nodes visited
  - Add (0, 0011), (2, 0110), (3, 1010) to queue
- Process (2, 0100): not all nodes visited
  - Add (0, 0101), (1, 0110) to queue
- Process (3, 1000): not all nodes visited
  - Add (1, 1010) to queue
- `steps = 1`

**Step 3: Process Level 1 (`steps = 1`)**
The queue holds 8 states. The ones that produce new states are:
- (1, 0011) → adds (2, 0111) and (3, 1011)
- (2, 0101) → adds (1, 0111)
- (2, 0110) → adds (0, 0111)
- (1, 0110) → adds (3, 1110)
- (1, 1010) → adds (0, 1011) and (2, 1110)

The other states, such as (0, 0011) and (3, 1010), only lead to states that are already in `visited_states`. No state has mask 1111 yet. `steps = 2`

**Step 4: Process Level 2 (`steps = 2`)**
The queue holds 7 states, each with three nodes visited. Several of them reach the full mask for the first time:
- (3, 1011) → adds (1, 1011)
- (1, 0111) → moves to node 3, adds (3, 1111)
- (3, 1110) → adds (1, 1110)
- (0, 1011) → moves to node 2, adds (2, 1111)
- (2, 1110) → moves to node 0, adds (0, 1111)

These full-mask states are only added to the queue here; the goal check happens when a state is dequeued. `steps = 3`

**Step 5: Process Level 3 (`steps = 3`)**
- Dequeue (1, 1011): not all nodes visited
- Dequeue (3, 1111): bitmask = 1111 = (1 << 4) - 1
- **All nodes visited! Return `steps = 3`**

The state (3, 1111) came from (1, 0111), which came from (2, 0101), which came from the start state (0, 0001). So one shortest path is 0 → 2 → 1 → 3, which uses 3 edges and visits every node.

## Python Solution

```python
class Solution:
    def shortestPathLength(self, graph: List[List[int]]) -> int:
        # Get the number of nodes in the graph
        num_nodes = len(graph)
      
        # Initialize BFS queue with all possible starting nodes
        # Each entry is (current_node, visited_state_bitmask)
        queue = deque()
        visited_states = set()
      
        # Start BFS from every node as a potential starting point
        for node in range(num_nodes):
            initial_state = 1 << node  # Set bit for this node as visited
            queue.append((node, initial_state))
            visited_states.add((node, initial_state))
      
        # Track the number of steps taken
        steps = 0
      
        # BFS to find shortest path visiting all nodes
        while True:
            # Process all nodes at current distance level
            for _ in range(len(queue)):
                current_node, visited_mask = queue.popleft()
              
                # Check if all nodes have been visited
                # (all bits set means visited_mask equals 2^n - 1)
                if visited_mask == (1 << num_nodes) - 1:
                    return steps
              
                # Explore all neighbors of current node
                for neighbor in graph[current_node]:
                    # Update visited state by setting neighbor's bit
                    new_visited_mask = visited_mask | (1 << neighbor)
                  
                    # Only process this state if we haven't seen it before
                    if (neighbor, new_visited_mask) not in visited_states:
                        visited_states.add((neighbor, new_visited_mask))
                        queue.append((neighbor, new_visited_mask))
          
            # Increment step counter after processing current level
            steps += 1
```

## Time and Space Complexity

**Time Complexity:** `O(n^2 * 2^n)`

The algorithm uses BFS to explore all possible states. Each state is represented by a tuple `(current_node, visited_mask)` where:
- `current_node` can be any of the `n` nodes
- `visited_mask` is a bitmask representing which nodes have been visited, with `2^n` possible combinations

Since we track visited states in a set to avoid revisiting them, each unique state `(node, mask)` is dequeued at most once. There are `n * 2^n` possible states total.

When a state is dequeued, we iterate through all neighbors of its node. A node has at most `n - 1` neighbors, so each state does `O(n)` work. Multiplying, the total work is `O(n * 2^n * n) = O(n^2 * 2^n)`. Put another way, for each mask, the neighbor loops over all nodes together cost `O(m)` where `m` is the number of edges, giving `O(2^n * (n + m))`, and `m` is at most `O(n^2)`.

**Space Complexity:** `O(n * 2^n)`

The space is dominated by:
- The `visited_states` set which stores all visited states `(node, mask)`. In the worst case, this stores all `n * 2^n` possible states.
- The `queue` which in the worst case could contain `O(n * 2^n)` states, though typically it would be much smaller.

Therefore, the space complexity is `O(n * 2^n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Confusing State with Just Node Tracking**
A common mistake is trying to track only visited nodes instead of the combination of (node, visited_state). This leads to incorrect solutions because the same node might need to be visited multiple times with different sets of previously visited nodes.

**Incorrect approach:**
```python
visited = set()  # Only tracking nodes
for neighbor in graph[current]:
    if neighbor not in visited:  # Wrong! This prevents revisiting
        visited.add(neighbor)
        queue.append(neighbor)
```

**Correct approach:**
```python
visited_states = set()  # Track (node, bitmask) pairs
for neighbor in graph[current_node]:
    new_state = (neighbor, new_mask)
    if new_state not in visited_states:  # Allows revisiting nodes with different states
        visited_states.add(new_state)
        queue.append(new_state)
```

### 2. **Incorrect Bitmask Operations**
Many developers make errors when working with bitmasks, especially confusing bit positions with node values.

**Common mistakes:**
- Using `1 << (i+1)` instead of `1 << i` (off-by-one error)
- Using `visited_mask + (1 << neighbor)` instead of `visited_mask | (1 << neighbor)` for setting bits
- Checking completion with `visited_mask == n` instead of `visited_mask == (1 << n) - 1`

**Example of correct bitmask usage:**
```python
# Setting bit for node i
initial_mask = 1 << i  # NOT: 1 << (i+1)

# Adding a new visited node
new_mask = current_mask | (1 << neighbor)  # NOT: current_mask + (1 << neighbor)

# Checking if all n nodes visited
if visited_mask == (1 << n) - 1:  # NOT: if visited_mask == n
    return steps
```

### 3. **Starting from Only One Node**
Another pitfall is implementing single-source BFS instead of multi-source BFS, which can miss the optimal solution where the shortest path doesn't start from node 0.

**Incorrect single-source:**
```python
queue = deque([(0, 1 << 0)])  # Only starting from node 0
visited_states.add((0, 1 << 0))
```

**Correct multi-source:**
```python
for node in range(num_nodes):
    initial_state = 1 << node
    queue.append((node, initial_state))
    visited_states.add((node, initial_state))
```

### 4. **Checking the Goal Only When Generating Neighbors**
The solution checks for the full mask when a state is dequeued. Some implementations move this check into the neighbor loop to return one level earlier. That version never checks the start states, so a graph with a single node never returns.

**Problematic structure:**
```python
full_mask = (1 << num_nodes) - 1
while True:
    for _ in range(len(queue)):
        current_node, visited_mask = queue.popleft()
        for neighbor in graph[current_node]:
            new_visited_mask = visited_mask | (1 << neighbor)
            if new_visited_mask == full_mask:
                return steps + 1
            # ... add the new state to the queue
    steps += 1
```

For `graph = [[]]` (one node, no edges), the answer is `0`. The only start state already has the full mask, but the check inside the neighbor loop never runs because node 0 has no neighbors. The queue becomes empty and the `while True` loop runs forever.

**Correct structure:**
```python
while True:
    for _ in range(len(queue)):
        current_node, visited_mask = queue.popleft()
        if visited_mask == (1 << num_nodes) - 1:
            return steps  # Start states are checked too
        # ... expand neighbors
    steps += 1
```

### 5. **Storing Visited States as Tuples in a Set**
The set of `(node, mask)` tuples is correct, but each tuple carries Python object overhead. With the LeetCode constraint `1 <= n <= 12` there are at most `12 * 2^12 = 49,152` states, so the set fits in memory. A 2D list indexed by node and mask stores the same information with less overhead and faster lookups:

```python
visited = [[False] * (1 << num_nodes) for _ in range(num_nodes)]
for node in range(num_nodes):
    visited[node][1 << node] = True
    queue.append((node, 1 << node))

# Inside the BFS loop:
new_visited_mask = visited_mask | (1 << neighbor)
if not visited[neighbor][new_visited_mask]:
    visited[neighbor][new_visited_mask] = True
    queue.append((neighbor, new_visited_mask))
```

