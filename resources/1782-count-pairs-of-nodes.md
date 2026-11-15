# 1782. Count Pairs Of Nodes

> Source: https://algo.monster/liteproblems/1782

## Problem Statement

You have an undirected [graph](/problems/graph_intro) with `n` nodes (numbered from 1 to n) and a list of edges. Each edge `edges[i] = [ui, vi]` represents an undirected connection between nodes `ui` and `vi`. Note that there can be multiple edges between the same pair of nodes.

For any pair of nodes `(a, b)`, we define `incident(a, b)` as the total number of edges that are connected to **either** node `a` or node `b`. This counts:
- All edges connected to node `a`
- All edges connected to node `b`
- But edges between `a` and `b` are counted only once (not double-counted)

You're given an array of `queries`. For each `queries[j]`, you need to count how many pairs of nodes `(a, b)` satisfy both conditions:
1. `a < b` (to avoid counting the same pair twice)
2. `incident(a, b) > queries[j]` (the total number of edges connected to either node exceeds the query value)

Return an array `answers` where `answers[j]` contains the count of valid node pairs for the j-th query.

**Example Understanding:**
If node 1 has 3 edges connected to it, node 2 has 4 edges connected to it, and there are 2 edges directly between nodes 1 and 2, then:
- Node 1's degree = 3
- Node 2's degree = 4
- `incident(1, 2) = 3 + 4 - 2 = 5` (we subtract the edges between them since they were counted in both degrees)

## Intuition

The key insight is that for a pair of nodes `(a, b)`, the value `incident(a, b)` equals `degree[a] + degree[b] - edges_between(a, b)`. This is because when we add the degrees of both nodes, we count the edges between them twice, so we need to subtract them once.

A naive approach would be to check all possible pairs `(a, b)` for each query, but with up to `n = 20,000` nodes, this would require checking `n*(n-1)/2` pairs for each query, which is too slow.

The breakthrough comes from realizing we can use a two-step approach:

1. **Optimistic counting with [binary search](/problems/binary_search_intro)**: First, assume there are no direct edges between any pair of nodes. In this case, `incident(a, b) = degree[a] + degree[b]`. If we sort the degrees, for each node `a` with degree `degree[a]`, we can use binary search to find how many nodes `b` have `degree[b]` such that `degree[a] + degree[b] > query`. This gives us an upper bound on the answer.

2. **Correction for direct edges**: The optimistic count above is incorrect for pairs that actually have edges between them. Specifically, if nodes `a` and `b` have `k` edges between them, then the actual `incident(a, b) = degree[a] + degree[b] - k`. So we need to check: did we count this pair in step 1 (meaning `degree[a] + degree[b] > query`), but shouldn't have (meaning `degree[a] + degree[b] - k ≤ query`)? If so, we subtract 1 from our answer.

This approach is efficient because:
- Step 1 takes `O(n log n)` per query using [binary search](/problems/binary_search_intro) on sorted degrees
- Step 2 only needs to check pairs that actually have edges between them (much fewer than all possible pairs)
- We precompute degrees and edge counts between pairs using hash tables for quick lookups

**Learn more about [Graph](/problems/graph_intro), [Two Pointers](/problems/two_pointers_intro), [Binary Search](/problems/binary_search_intro) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

The implementation follows the two-step approach described in the intuition:

**Step 1: Preprocessing**
- Create a `cnt` array to store the degree of each node (converting to 0-indexed)
- Use a hash table `g` to store the count of edges between each pair of nodes
- For each edge `[a, b]`:
  - Convert to 0-indexed: `a = a - 1, b = b - 1`
  - Normalize the pair as `(min(a, b), max(a, b))` to avoid duplicates
  - Increment `cnt[a]` and `cnt[b]` (degree counting)
  - Increment `g[(a, b)]` (edge count between this pair)

**Step 2: Create sorted degree array**
- Create `s = sorted(cnt)` for efficient [binary search](/problems/binary_search_intro)
- This allows us to quickly find how many nodes have degree above a certain threshold

**Step 3: Process each query**
For each query value `t`:

1. **Optimistic counting with [binary search](/problems/binary_search_intro):**
   - For each degree value `x` at position `j` in the sorted array `s`
   - Find the smallest index `k` where `s[k] > t - x` using `bisect_right(s, t - x, lo=j + 1)`
   - The `lo=j + 1` ensures we only count pairs where `a < b` (avoiding duplicates)
   - Add `n - k` to the answer (number of valid pairs with node having degree `x`)

2. **Correction for actual edges:**
   - Iterate through all pairs `(a, b)` that have edges between them (stored in `g`)
   - For each pair with `v` edges between them:
     - Check if we counted this pair optimistically: `cnt[a] + cnt[b] > t`
     - But shouldn't have: `cnt[a] + cnt[b] - v <= t`
     - If both conditions are true, subtract 1 from the answer

**Time Complexity:**
- Preprocessing: `O(E)` where E is the number of edges
- Per query: `O(n log n)` for [binary search](/problems/binary_search_intro) + `O(unique pairs with edges)`
- Total: `O(E + Q * (n log n + P))` where Q is queries and P is unique pairs with edges

**Space Complexity:** `O(n + P)` for storing degrees and edge counts between pairs

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Given:**
- `n = 4` nodes (numbered 1 to 4)
- `edges = [[1,2], [2,3], [2,4], [3,4], [3,4]]` (note: two edges between nodes 3 and 4)
- `queries = [4, 5]`

**Step 1: Preprocessing**

First, calculate each node's degree and track edges between pairs:
- Node 1: connected to edge [1,2] → degree = 1
- Node 2: connected to edges [1,2], [2,3], [2,4] → degree = 3
- Node 3: connected to edges [2,3], [3,4], [3,4] → degree = 3
- Node 4: connected to edges [2,4], [3,4], [3,4] → degree = 3

Edge counts between pairs:
- (1,2): 1 edge
- (2,3): 1 edge
- (2,4): 1 edge
- (3,4): 2 edges

**Step 2: Create sorted degree array**
- Original degrees: [1, 3, 3, 3]
- Sorted degrees: [1, 3, 3, 3]

**Step 3: Process Query 1 (t = 4)**

*Optimistic counting:*
- For node with degree 1 (position 0): Find nodes with degree > 3. Using binary search from position 1 onward, we find no such nodes. Count = 0
- For node with degree 3 (position 1): Find nodes with degree > 1. From position 2 onward, we have 2 nodes. Count = 2
- For node with degree 3 (position 2): Find nodes with degree > 1. From position 3 onward, we have 1 node. Count = 1
- For node with degree 3 (position 3): No nodes after it. Count = 0
- Total optimistic count = 0 + 2 + 1 + 0 = 3

*Correction for actual edges:*
Check each pair with edges:
- Pair (1,2): degree[1] + degree[2] = 1 + 3 = 4, not > 4, so wasn't counted optimistically. No correction needed.
- Pair (2,3): degree[2] + degree[3] = 3 + 3 = 6 > 4, and 6 - 1 = 5 > 4. No correction needed.
- Pair (2,4): degree[2] + degree[4] = 3 + 3 = 6 > 4, and 6 - 1 = 5 > 4. No correction needed.
- Pair (3,4): degree[3] + degree[4] = 3 + 3 = 6 > 4, but 6 - 2 = 4 ≤ 4. We counted this pair but shouldn't have. Subtract 1.

Final answer for query 4: 3 - 1 = 2

**Step 3: Process Query 2 (t = 5)**

*Optimistic counting:*
- For node with degree 1 (position 0): Find nodes with degree > 4. No such nodes. Count = 0
- For node with degree 3 (position 1): Find nodes with degree > 2. From position 2 onward, we have 2 nodes. Count = 2
- For node with degree 3 (position 2): Find nodes with degree > 2. From position 3 onward, we have 1 node. Count = 1
- For node with degree 3 (position 3): No nodes after it. Count = 0
- Total optimistic count = 0 + 2 + 1 + 0 = 3

*Correction for actual edges:*
Check each pair with edges:
- Pair (1,2): degree[1] + degree[2] = 4, not > 5. Not counted optimistically. No correction.
- Pair (2,3): degree[2] + degree[3] = 6 > 5, and 6 - 1 = 5 ≤ 5. We counted this pair but shouldn't have. Subtract 1.
- Pair (2,4): degree[2] + degree[4] = 6 > 5, and 6 - 1 = 5 ≤ 5. We counted this pair but shouldn't have. Subtract 1.
- Pair (3,4): degree[3] + degree[4] = 6 > 5, and 6 - 2 = 4 ≤ 5. We counted this pair but shouldn't have. Subtract 1.

Final answer for query 5: 3 - 3 = 0

**Result:** `answers = [2, 0]`

The valid pairs for query 4 are (2,3) and (2,4), both with incident count of 5.
For query 5, no pairs have incident count > 5.

## Python Solution

```python
from collections import defaultdict
from bisect import bisect_right
from typing import List

class Solution:
    def countPairs(
        self, n: int, edges: List[List[int]], queries: List[int]
    ) -> List[int]:
        # Initialize degree count for each node (0-indexed)
        node_degrees = [0] * n

        # Dictionary to store edge frequencies between pairs of nodes
        # Key: (smaller_node, larger_node), Value: edge count
        edge_frequencies = defaultdict(int)

        # Process each edge to count degrees and edge frequencies
        for node1, node2 in edges:
            # Convert to 0-indexed
            node1, node2 = node1 - 1, node2 - 1

            # Ensure consistent ordering with smaller node first
            smaller_node, larger_node = min(node1, node2), max(node1, node2)

            # Increment degree for both nodes
            node_degrees[smaller_node] += 1
            node_degrees[larger_node] += 1

            # Track frequency of this specific edge
            edge_frequencies[(smaller_node, larger_node)] += 1

        # Create sorted copy of degrees for binary search
        sorted_degrees = sorted(node_degrees)

        # Initialize result array for each query
        result = [0] * len(queries)

        # Process each query
        for query_idx, threshold in enumerate(queries):
            # Count pairs where sum of degrees > threshold
            # Using two-pointer technique with binary search
            for j, current_degree in enumerate(sorted_degrees):
                # Find smallest index where sorted_degrees[k] + current_degree > threshold
                # Search only in elements after current position to avoid duplicate pairs
                k = bisect_right(sorted_degrees, threshold - current_degree, lo=j + 1)

                # All nodes from index k to n-1 form valid pairs with current node
                result[query_idx] += n - k

            # Adjust for multi-edges: some pairs were overcounted
            # If a pair has multiple edges, their actual incident count is less than
            # the sum of their degrees (since shared edges are counted twice)
            for (node_a, node_b), edge_count in edge_frequencies.items():
                # Total incident edges without removing duplicates
                total_with_duplicates = node_degrees[node_a] + node_degrees[node_b]

                # Actual incident edges (removing duplicate counting)
                actual_incident_edges = total_with_duplicates - edge_count

                # If we counted this pair but shouldn't have, decrement
                if total_with_duplicates > threshold and actual_incident_edges <= threshold:
                    result[query_idx] -= 1

        return result
```

## Time and Space Complexity

**Time Complexity:** `O(q × (n × log n + m))`

The time complexity breaks down as follows:
- Initial edge processing takes `O(m)` to iterate through all edges and update counts and the dictionary
- Sorting the count array takes `O(n log n)`
- For each query (total `q` queries):
  - The nested loop iterates through each element in sorted array `s` (size `n`), and for each element performs a binary search using `bisect_right`, which takes `O(log n)`. This gives `O(n × log n)` per query
  - The correction loop iterates through all unique edges in dictionary `g`, which can be at most `O(m)` edges
  - Total per query: `O(n × log n + m)`
- Overall: `O(q × (n × log n + m))`

**Space Complexity:** `O(n + m)`

The space complexity consists of:
- Array `cnt` of size `n`: `O(n)`
- Dictionary `g` storing at most `m` unique edges: `O(m)`
- Sorted array `s` (copy of `cnt`): `O(n)`
- Answer array `ans` of size `q`: `O(q)`
- Since `q` is typically smaller than or comparable to `n` and `m`, the dominant terms are `O(n + m)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Edge Counting Between Node Pairs**
A critical pitfall is miscounting or misunderstanding how edges between the same pair of nodes affect the incident count. Many implementations fail to properly handle multiple edges between the same two nodes.

**Problem Example:**
```python
# Incorrect approach - treating each edge independently
edge_count = defaultdict(int)
for u, v in edges:
    edge_count[(u, v)] += 1  # Wrong! (1,2) and (2,1) treated as different
```

**Solution:**
Always normalize the node pair by ordering them consistently:
```python
edge_frequencies = defaultdict(int)
for node1, node2 in edges:
    node1, node2 = node1 - 1, node2 - 1
    smaller_node, larger_node = min(node1, node2), max(node1, node2)
    edge_frequencies[(smaller_node, larger_node)] += 1
```

### 2. **Double-Counting Node Pairs in Binary Search**
When using binary search to find valid pairs, it's easy to count each pair twice (once as (a,b) and once as (b,a)).

**Problem Example:**
```python
# Incorrect - counts each pair twice
for j, degree in enumerate(sorted_degrees):
    k = bisect_right(sorted_degrees, threshold - degree)  # Wrong starting point!
    result += n - k
```

**Solution:**
Start the binary search from `j + 1` to ensure we only count pairs where the first node comes before the second:
```python
for j, current_degree in enumerate(sorted_degrees):
    k = bisect_right(sorted_degrees, threshold - current_degree, lo=j + 1)
    result[query_idx] += n - k
```

### 3. **Forgetting 1-Indexed to 0-Indexed Conversion**
The problem states nodes are numbered from 1 to n, but arrays in most programming languages are 0-indexed. Forgetting this conversion leads to index errors or incorrect results.

**Problem Example:**
```python
# Incorrect - using 1-indexed directly
node_degrees = [0] * (n + 1)  # Wastes space
for node1, node2 in edges:
    node_degrees[node1] += 1  # Using 1-indexed directly
    node_degrees[node2] += 1
```

**Solution:**
Convert to 0-indexed immediately when processing edges:
```python
node_degrees = [0] * n
for node1, node2 in edges:
    node1, node2 = node1 - 1, node2 - 1  # Convert to 0-indexed
    # ... rest of processing
```

### 4. **Incorrect Correction Logic for Multi-Edges**
The trickiest part is correctly adjusting for pairs that were optimistically counted but shouldn't be included due to shared edges.

**Problem Example:**
```python
# Incorrect correction logic
for (node_a, node_b), edge_count in edge_frequencies.items():
    if node_degrees[node_a] + node_degrees[node_b] - edge_count > threshold:
        result[query_idx] += 1  # Wrong! Should be subtracting overcounts
```

**Solution:**
The correction should subtract overcounted pairs:
```python
for (node_a, node_b), edge_count in edge_frequencies.items():
    total_with_duplicates = node_degrees[node_a] + node_degrees[node_b]
    actual_incident_edges = total_with_duplicates - edge_count

    # We counted this pair optimistically but it doesn't actually qualify
    if total_with_duplicates > threshold and actual_incident_edges <= threshold:
        result[query_idx] -= 1
```

### 5. **Performance Degradation with Repeated Sorting**
Sorting the degree array inside the query loop would cause unnecessary time complexity.

**Problem Example:**
```python
for query in queries:
    sorted_degrees = sorted(node_degrees)  # Sorting for each query - O(n log n) per query!
    # ... rest of logic
```

**Solution:**
Sort once before processing all queries:
```python
sorted_degrees = sorted(node_degrees)  # Sort once
for query_idx, threshold in enumerate(queries):
    # Use pre-sorted array for all queries
```

