# 2242. Maximum Score of a Node Sequence

> Source: https://algo.monster/liteproblems/2242

## Problem Statement

You have an undirected [graph](/problems/graph_intro) with `n` nodes labeled from `0` to `n - 1`. Each node has a score given in the array `scores`, where `scores[i]` is the score of node `i`. The graph's edges are provided in the array `edges`, where each `edges[i] = [ai, bi]` represents an undirected edge between nodes `ai` and `bi`.

A **valid node sequence** must satisfy two conditions:
1. Every pair of adjacent nodes in the sequence must be connected by an edge in the [graph](/problems/graph_intro)
2. Each node can appear at most once in the sequence (no repetitions)

The **score of a node sequence** is the sum of all individual node scores in that sequence.

Your task is to find the maximum possible score of a valid node sequence that has exactly 4 nodes. If no valid 4-node sequence exists in the [graph](/problems/graph_intro), return `-1`.

For example, if you have a sequence `[a, b, c, d]`, it's valid only if:
- There's an edge between `a` and `b`
- There's an edge between `b` and `c`  
- There's an edge between `c` and `d`
- All four nodes are distinct

The score would be `scores[a] + scores[b] + scores[c] + scores[d]`.

## Intuition

The key insight is that any 4-node path has a specific structure: it consists of two nodes in the middle connected by an edge, with one additional node attached to each end. In other words, for a path `[c, a, b, d]`, the edge `(a, b)` serves as the "backbone" of the path.

This observation transforms our problem: instead of searching for all possible 4-node paths in the [graph](/problems/graph_intro), we can iterate through each edge and try to extend it into a 4-node path by adding one node on each side.

For each edge `(a, b)`:
- We need to find a node `c` that connects to `a` (but isn't `b`)
- We need to find a node `d` that connects to `b` (but isn't `a` or `c`)
- All four nodes must be distinct

Since we want to maximize the total score, we should choose the highest-scoring neighbors for `c` and `d`. But here's the challenge: if we check all neighbors of `a` and all neighbors of `b`, the time complexity could be very high in dense graphs.

The clever optimization comes from realizing that we don't need to store all neighbors for each node. For any edge `(a, b)`, we're looking for:
- The best neighbor of `a` (excluding `b`)
- The best neighbor of `b` (excluding `a` and the chosen neighbor of `a`)

Even in the worst case where our first choice creates a conflict (same node chosen from both sides), we only need the top 3 neighbors for each node. Why 3? Because:
- We exclude one node (the other endpoint of the edge)
- We might exclude one more node due to overlap
- This leaves us needing at most the 3rd best option

By keeping only the top 3 neighbors (sorted by score) for each node, we guarantee that we'll find the optimal solution if it exists, while significantly reducing the search space and improving efficiency.

**Learn more about [Graph](/problems/graph_intro) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

The implementation follows these steps:

**Step 1: Build an adjacency list representation of the [graph](/problems/graph_intro)**
```python
g = defaultdict(list)
for a, b in edges:
    g[a].append(b)
    g[b].append(a)
```
We create a dictionary where each node maps to a list of its neighbors. Since the graph is undirected, we add both `a` to `b`'s neighbors and `b` to `a`'s neighbors.

**Step 2: Optimize the adjacency list by keeping only top 3 neighbors**
```python
for k in g.keys():
    g[k] = nlargest(3, g[k], key=lambda x: scores[x])
```
For each node, we use `nlargest` to keep only the 3 neighbors with the highest scores. This is the key optimization that reduces our search space while guaranteeing we won't miss the optimal solution.

**Step 3: Try to extend each edge into a 4-node path**
```python
ans = -1
for a, b in edges:
    for c in g[a]:
        for d in g[b]:
            if b != c != d != a:
                t = scores[a] + scores[b] + scores[c] + scores[d]
                ans = max(ans, t)
```

For each edge `(a, b)` in the original [graph](/problems/graph_intro):
- We iterate through all possible neighbors `c` of node `a` (at most 3 due to our optimization)
- We iterate through all possible neighbors `d` of node `b` (at most 3)
- We check if all four nodes are distinct: `b != c != d != a`
  - This ensures `c` is not `b`, `d` is not `a`, and `c` is not `d`
- If valid, we calculate the total score of the path `[c, a, b, d]`
- We update our answer with the maximum score found

**Time Complexity**: `O(m)` where `m` is the number of edges. For each edge, we check at most 3 × 3 = 9 combinations, which is constant.

**Space Complexity**: `O(n)` for storing the adjacency list, where each node stores at most 3 neighbors.

The algorithm returns `-1` if no valid 4-node sequence exists (when `ans` remains `-1`), otherwise returns the maximum score found.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Given:**
- `n = 5` nodes (labeled 0 to 4)
- `scores = [5, 2, 9, 8, 4]`
- `edges = [[0,1], [1,2], [2,3], [0,3], [2,4]]`

**Step 1: Build the adjacency list**

After processing all edges, we get:
- Node 0: neighbors [1, 3]
- Node 1: neighbors [0, 2]
- Node 2: neighbors [1, 3, 4]
- Node 3: neighbors [2, 0]
- Node 4: neighbors [2]

**Step 2: Keep only top 3 neighbors by score**

For each node, sort neighbors by their scores and keep top 3:
- Node 0: neighbors [3, 1] (scores: 8, 2)
- Node 1: neighbors [2, 0] (scores: 9, 5)
- Node 2: neighbors [3, 4, 1] (scores: 8, 4, 2)
- Node 3: neighbors [2, 0] (scores: 9, 5)
- Node 4: neighbors [2] (score: 9)

**Step 3: Try extending each edge**

Let's check edge (0, 1):
- For node 0's neighbors (excluding 1): we have node 3
- For node 1's neighbors (excluding 0): we have node 2
- Check path [3, 0, 1, 2]: all nodes distinct ✓
- Score = 8 + 5 + 2 + 9 = 24

Let's check edge (1, 2):
- For node 1's neighbors (excluding 2): we have node 0
- For node 2's neighbors (excluding 1): we have nodes 3 and 4
- Check path [0, 1, 2, 3]: all nodes distinct ✓
  - Score = 5 + 2 + 9 + 8 = 24
- Check path [0, 1, 2, 4]: all nodes distinct ✓
  - Score = 5 + 2 + 9 + 4 = 20

Let's check edge (2, 3):
- For node 2's neighbors (excluding 3): we have nodes 4 and 1
- For node 3's neighbors (excluding 2): we have node 0
- Check path [4, 2, 3, 0]: all nodes distinct ✓
  - Score = 4 + 9 + 8 + 5 = 26
- Check path [1, 2, 3, 0]: all nodes distinct ✓
  - Score = 2 + 9 + 8 + 5 = 24

Let's check edge (0, 3):
- For node 0's neighbors (excluding 3): we have node 1
- For node 3's neighbors (excluding 0): we have node 2
- Check path [1, 0, 3, 2]: all nodes distinct ✓
  - Score = 2 + 5 + 8 + 9 = 24

Let's check edge (2, 4):
- For node 2's neighbors (excluding 4): we have nodes 3 and 1
- For node 4's neighbors (excluding 2): no other neighbors
- No valid 4-node path can be formed from this edge

**Result:** The maximum score is 26 from the path [4, 2, 3, 0].

## Python Solution

```python
from collections import defaultdict
from heapq import nlargest
from typing import List

class Solution:
    def maximumScore(self, scores: List[int], edges: List[List[int]]) -> int:
        # Build adjacency list graph representation
        graph = defaultdict(list)
        for node_a, node_b in edges:
            graph[node_a].append(node_b)
            graph[node_b].append(node_a)
      
        # For each node, keep only the 3 neighbors with highest scores
        # This optimization reduces the search space while guaranteeing we won't miss the optimal solution
        for node in graph.keys():
            graph[node] = nlargest(3, graph[node], key=lambda neighbor: scores[neighbor])
      
        max_score = -1
      
        # Try all edges as the middle edge of a 4-node path
        for node_a, node_b in edges:
            # Try all combinations of neighbors of node_a and node_b
            for node_c in graph[node_a]:
                for node_d in graph[node_b]:
                    # Ensure all four nodes are distinct
                    if node_b != node_c and node_c != node_d and node_d != node_a:
                        # Calculate total score of the 4-node path: c-a-b-d
                        total_score = scores[node_a] + scores[node_b] + scores[node_c] + scores[node_d]
                        max_score = max(max_score, total_score)
      
        return max_score
```

## Time and Space Complexity

**Time Complexity:** `O(m + n log n + m)`

Breaking down the time complexity:
1. Building the adjacency list from edges takes `O(m)` where `m` is the number of edges
2. For each vertex, we use `nlargest(3, ...)` to keep only the 3 neighbors with highest scores. In the worst case, each vertex could be connected to all other vertices, so this operation is `O(n log n)` per vertex. With `n` vertices total, this step is `O(n * n log n)` in the worst case, but since the graph is built from `m` edges, the total degree sum is `2m`, making this step effectively `O(m log m)` or simplified to `O(m)` since we're only keeping 3 elements
3. The nested loops iterate through all edges (`m` iterations), and for each edge, check at most 3 neighbors from each endpoint (constant 3×3=9 combinations), giving `O(m * 9) = O(m)`

Overall time complexity: `O(m)` where `m` is the number of edges

**Space Complexity:** `O(n + m)`

Breaking down the space complexity:
1. The adjacency list `g` stores at most 3 neighbors for each vertex after filtering, requiring `O(n)` space
2. Before filtering, the adjacency list temporarily stores all edges, requiring `O(m)` space
3. Other variables use constant space

Overall space complexity: `O(n + m)` where `n` is the number of vertices and `m` is the number of edges

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Incorrect Neighbor Selection After Optimization

**The Problem:**
A critical pitfall occurs when we keep only the top 3 neighbors for each node. If we're not careful about *when* we select these top neighbors, we might accidentally filter out edges that are part of the optimal solution.

Consider this scenario:
- Node A has neighbors: [B, C, D, E] with scores [10, 20, 30, 40]
- We keep top 3 by score: [C, D, E]
- But the edge (A, B) still exists in our edge list!
- When processing edge (A, B), we try to find neighbors of A, but B is no longer in A's neighbor list
- This creates an inconsistency where we process an edge but can't properly extend it

**Incorrect Implementation:**
```python
# WRONG: This can cause issues
for a, b in edges:
    for c in g[a]:  # g[a] might not contain b anymore!
        for d in g[b]:  # g[b] might not contain a anymore!
            if b != c != d != a:
                # We might miss valid paths
```

**The Solution:**
The code handles this correctly by:
1. Building the complete adjacency list first
2. Then filtering to top 3 neighbors
3. When iterating through edges, we use the original edge list (which preserves all connections)
4. We only use the filtered adjacency list to find the *extension* nodes (c and d), not to verify the edge (a, b) itself

### Pitfall 2: Misunderstanding the 4-Node Path Structure

**The Problem:**
Developers might misunderstand what constitutes a valid 4-node path and try to find any 4 connected nodes, rather than specifically a path structure.

**Incorrect Approach:**
```python
# WRONG: Looking for any 4 connected nodes (could form a cycle or star)
for node in range(n):
    neighbors = g[node]
    if len(neighbors) >= 3:
        # Just picking any 3 neighbors plus the node itself
        total = scores[node] + sum(scores[n] for n in neighbors[:3])
```

**The Solution:**
The correct approach recognizes that we need a *path* specifically: c-a-b-d where:
- c connects to a (but not necessarily to b or d)
- a connects to b (the central edge)
- b connects to d (but not necessarily to a or c)
- All nodes are distinct

### Pitfall 3: Inefficient Duplicate Checking

**The Problem:**
When checking if all four nodes are distinct, using a set or multiple individual comparisons can be error-prone or inefficient.

**Incorrect Implementation:**
```python
# WRONG: Incomplete checking
if c != b and d != a:  # Misses checking if c == d or c == a or d == b
    total_score = ...

# ALSO WRONG: Over-complicated
if len(set([a, b, c, d])) == 4:  # Works but creates unnecessary overhead
    total_score = ...
```

**The Solution:**
The code uses the elegant Python chaining: `if b != c != d != a:`
This is equivalent to `if b != c and c != d and d != a:` but note that this still requires understanding that:
- We already know a != b (they form an edge, and we assume no self-loops)
- We need to explicitly check that c != a and d != b aren't guaranteed by this condition

A more explicit (and perhaps clearer) version would be:
```python
if c != b and c != a and d != a and d != b and c != d:
    total_score = ...
```

