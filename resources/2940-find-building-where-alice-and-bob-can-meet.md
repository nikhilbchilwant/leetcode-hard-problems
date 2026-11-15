# 2940. Find Building Where Alice and Bob Can Meet

> Source: https://algo.monster/liteproblems/2940

## Problem Statement

You are given a **0-indexed** array `heights` of positive integers, where `heights[i]` represents the height of the `i-th` building.

The movement rule for a person in the buildings is: If a person is in building `i`, they can move to any other building `j` if and only if both conditions are met:
- `i < j` (can only move to buildings with higher indices)
- `heights[i] < heights[j]` (can only move to taller buildings)

You are also given an array `queries` where `queries[i] = [ai, bi]`. For each query:
- Alice starts in building `ai`
- Bob starts in building `bi`

Your task is to find the **leftmost building** (minimum index) where Alice and Bob can meet for each query. Both Alice and Bob need to be able to reach this common building following the movement rules.

Return an array `ans` where `ans[i]` is the index of the leftmost building where Alice and Bob can meet on the `i-th` query. If they cannot meet at any building, set `ans[i]` to `-1`.

For example, if Alice is at building 2 and Bob is at building 5:
- Alice can move from building 2 to any building `j > 2` where `heights[j] > heights[2]`
- Bob can move from building 5 to any building `j > 5` where `heights[j] > heights[5]`
- The answer would be the smallest index building that both can reach

Note that if Alice and Bob start at the same building, or if one person is already at a building the other can reach directly, that building itself could be the meeting point.

## Intuition

Let's think about when Alice and Bob can meet. For a query `[a, b]`, we can assume `a ≤ b` without loss of generality (if not, we can swap them).

There are a few cases to consider:

1. **If `a == b`**: They're already at the same building, so the answer is `b`.

2. **If `heights[a] < heights[b]`**: Alice can directly jump to building `b` (since `a < b` and `heights[a] < heights[b]`), so they meet at building `b`.

3. **If `heights[a] ≥ heights[b]`**: This is the tricky case. Bob cannot jump backwards to Alice's building. Alice cannot jump to Bob's building because `heights[a] ≥ heights[b]`. They need to find a common building `j` where `j > b` and `heights[j] > heights[a]` (so Alice can reach it) and `heights[j] > heights[b]` (so Bob can reach it). Since `heights[a] ≥ heights[b]`, we just need `heights[j] > heights[a]`.

The key insight is that for case 3, we need to find the **leftmost** (smallest index) building `j` such that `j > b` and `heights[j] > heights[a]`.

Now, how do we efficiently find this for multiple queries? If we process queries independently, we'd need to scan the array for each query, which would be inefficient.

The clever approach is to **process queries in descending order of their right endpoint `b`**. Why? Because as we process queries with smaller `b` values, we can incrementally add buildings from right to left into a data structure. This way, when we process a query with right endpoint `b`, we already have information about all buildings with index greater than `b`.

We use a **Binary Indexed Tree (Fenwick Tree)** to maintain the minimum index for each height value. As we move from right to left in the buildings array, we insert each building's index into the BIT, indexed by its height (after discretization to handle the range). When we need to find the meeting point for Alice at position `a`, we query the BIT for the minimum index among all buildings taller than `heights[a]`.

The discretization step (using `sorted(set(heights))`) is needed because heights can be large values, but we need them as indices for the BIT. We map each height to its rank among all unique heights.

**Learn more about [Stack](/problems/stack_intro), [Segment Tree](/problems/segment_tree_intro), [Binary Search](/problems/binary_search_intro), [Monotonic Stack](/problems/mono_stack_intro) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

The solution uses a **Binary Indexed Tree (Fenwick Tree)** combined with query sorting to efficiently find the leftmost meeting building for each query.

**Step 1: Query Preprocessing**
- First, normalize each query so that `queries[i] = [min(ai, bi), max(ai, bi)]`. This ensures `l ≤ r` for each query `[l, r]`.
- Sort query indices in descending order of their right endpoint `r`. This allows us to process buildings from right to left and reuse computed information.

**Step 2: Height Discretization**
- Create a sorted list of unique heights: `s = sorted(set(heights))`.
- This is used to map actual height values to ranks (1 to n) for indexing in the BIT.
- For a height `h`, its rank is `n - bisect_left(s, h)`. We use reverse ranking so that larger heights get smaller indices, which aligns with our query needs.

**Step 3: Binary Indexed Tree Setup**
- Initialize a BIT of size `n` that maintains the minimum index for each height rank.
- The BIT supports:
  - `update(x, v)`: Update position `x` with value `v` (keeping minimum)
  - `query(x)`: Get the minimum value among positions 1 to `x`

**Step 4: Process Queries in Order**
- Use pointer `j` starting from `n-1` (rightmost building).
- For each query `[l, r]` (processed in descending order of `r`):

  a. **Add buildings to BIT**: While `j > r`, insert building `j` into the BIT:
     - Calculate height rank: `k = n - bisect_left(s, heights[j]) + 1`
     - Update BIT at position `k` with value `j`
     - Decrement `j`

  b. **Answer the query**:
     - If `l == r` or `heights[l] < heights[r]`: Answer is `r` (direct meeting)
     - Otherwise:
       - Calculate rank of `heights[l]`: `k = n - bisect_left(s, heights[l])`
       - Query BIT for minimum index among buildings taller than `heights[l]`
       - The BIT returns the leftmost valid building index, or `-1` if none exists

**Why this works:**
- By processing queries with larger `r` values first, we ensure that when processing a query with endpoint `r`, the BIT already contains all buildings with index `> r`.
- The BIT efficiently maintains the minimum index for each height range, allowing O(log n) queries.
- The reverse ranking ensures that querying rank `k` gives us the minimum index among all buildings with height strictly greater than the height at rank `k`.

**Time Complexity:** `O((n + m) × log n + m × log m)`
- Sorting queries: `O(m log m)`
- Processing each building and query: `O((n + m) × log n)` due to BIT operations

**Space Complexity:** `O(n + m)` for the BIT and auxiliary arrays.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Input:**
- `heights = [5, 3, 8, 2, 6, 1, 4, 6]`
- `queries = [[0, 7], [3, 5], [5, 2], [3, 0], [1, 6]]`

**Step 1: Query Preprocessing**

Normalize queries so left ≤ right:
- `[0, 7]` → `[0, 7]` (already normalized)
- `[3, 5]` → `[3, 5]` (already normalized)
- `[5, 2]` → `[2, 5]` (swapped)
- `[3, 0]` → `[0, 3]` (swapped)
- `[1, 6]` → `[1, 6]` (already normalized)

Sort query indices by descending right endpoint:
- Query 0: `[0, 7]` (r=7)
- Query 4: `[1, 6]` (r=6)
- Query 1: `[3, 5]` (r=5)
- Query 2: `[2, 5]` (r=5)
- Query 3: `[0, 3]` (r=3)

**Step 2: Height Discretization**

Unique heights sorted: `s = [1, 2, 3, 4, 5, 6, 8]`

Height rankings (using reverse order):
- Height 8 → rank 1
- Height 6 → rank 2
- Height 5 → rank 3
- Height 4 → rank 4
- Height 3 → rank 5
- Height 2 → rank 6
- Height 1 → rank 7

**Step 3: Process Queries**

Initialize empty BIT and pointer `j = 7` (rightmost building).

**Processing Query 0: `[0, 7]`**
- Add buildings from index 7 down to index 8 (none to add since j=7 and r=7)
- Check: `l=0, r=7`, `heights[0]=5 < heights[7]=6`
- Direct jump possible! Answer: `7`

**Processing Query 4: `[1, 6]`**
- Add building 7 to BIT:
  - `heights[7]=6` → rank 2
  - BIT.update(2, 7)
- Check: `l=1, r=6`, `heights[1]=3 < heights[6]=4`
- Direct jump possible! Answer: `6`

**Processing Query 1: `[3, 5]`**
- Add building 6 to BIT:
  - `heights[6]=4` → rank 4
  - BIT.update(4, 6)
- Check: `l=3, r=5`, `heights[3]=2 > heights[5]=1`
- Need to find building > index 5 with height > 2
- Query BIT for heights > 2 (rank 6): BIT.query(5) returns 6
- Answer: `6` (building at index 6 has height 4 > 2)

**Processing Query 2: `[2, 5]`**
- No new buildings to add (r=5 same as previous)
- Check: `l=2, r=5`, `heights[2]=8 > heights[5]=1`
- Need to find building > index 5 with height > 8
- Query BIT for heights > 8 (rank 0): No buildings taller than 8
- Answer: `-1`

**Processing Query 3: `[0, 3]`**
- Add buildings 5, 4, 3 to BIT:
  - Building 5: `heights[5]=1` → rank 7, BIT.update(7, 5)
  - Building 4: `heights[4]=6` → rank 2, BIT.update(2, 4)
  - Building 3: Already processed as part of query
- Check: `l=0, r=3`, `heights[0]=5 > heights[3]=2`
- Need to find building > index 3 with height > 5
- Query BIT for heights > 5 (rank 2): BIT.query(2) returns 4
- Answer: `4` (building at index 4 has height 6 > 5)

**Final Results:**
Reorder answers to match original query order:
- Query 0: `7`
- Query 1: `6`
- Query 2: `-1`
- Query 3: `4`
- Query 4: `6`

Output: `[7, 6, -1, 4, 6]`

## Python Solution

```python
from typing import List
from bisect import bisect_left

class BinaryIndexedTree:
    """
    A Binary Indexed Tree (Fenwick Tree) modified to support range minimum queries.
    Used to efficiently find the minimum index for heights greater than a threshold.
    """
    __slots__ = ["size", "tree"]

    def __init__(self, size: int):
        """
        Initialize the BIT with given size.

        Args:
            size: The size of the array to index
        """
        self.size = size
        self.tree = [float('inf')] * (size + 1)

    def update(self, index: int, value: int):
        """
        Update the BIT by setting minimum value at position index.

        Args:
            index: The position to update (1-indexed)
            value: The value to set as minimum
        """
        while index <= self.size:
            self.tree[index] = min(self.tree[index], value)
            # Move to next index affected by this position
            index += index & -index

    def query(self, index: int) -> int:
        """
        Query the minimum value from index 1 to the given index.

        Args:
            index: The right boundary of the query range (1-indexed)

        Returns:
            The minimum value in range [1, index], or -1 if no valid value exists
        """
        minimum = float('inf')
        while index > 0:
            minimum = min(minimum, self.tree[index])
            # Move to parent node in BIT
            index -= index & -index
        return -1 if minimum == float('inf') else minimum


class Solution:
    def leftmostBuildingQueries(
        self, heights: List[int], queries: List[List[int]]
    ) -> List[int]:
        """
        Find the leftmost building where pairs of people can meet.

        For each query [a, b], find the leftmost building c where:
        - c >= max(a, b)
        - heights[c] > max(heights[a], heights[b])

        Args:
            heights: List of building heights
            queries: List of pairs [a, b] representing people positions

        Returns:
            List of leftmost meeting positions for each query, -1 if impossible
        """
        num_buildings = len(heights)
        num_queries = len(queries)

        # Normalize queries so that first element is always smaller
        for i in range(num_queries):
            queries[i] = [min(queries[i]), max(queries[i])]

        # Create sorted unique heights for coordinate compression
        sorted_unique_heights = sorted(set(heights))

        # Initialize result array
        result = [-1] * num_queries

        # Initialize BIT for efficient range minimum queries
        fenwick_tree = BinaryIndexedTree(num_buildings)

        # Current building index being processed (right to left)
        current_building = num_buildings - 1

        # Process queries in descending order of right endpoint
        # This allows us to incrementally build the BIT as we go
        sorted_query_indices = sorted(range(num_queries),
                                     key=lambda i: -queries[i][1])

        for query_idx in sorted_query_indices:
            left_person, right_person = queries[query_idx]

            # Add all buildings to the right of current query's right endpoint
            while current_building > right_person:
                # Compress height coordinate (1-indexed for BIT)
                # Higher heights get lower compressed values for minimum queries
                compressed_height = num_buildings - bisect_left(sorted_unique_heights,
                                                               heights[current_building]) + 1
                fenwick_tree.update(compressed_height, current_building)
                current_building -= 1

            # Check if people can meet at the right person's position
            if left_person == right_person or heights[left_person] < heights[right_person]:
                result[query_idx] = right_person
            else:
                # Need to find a building taller than left person's building
                # Query for buildings strictly taller than left person's height
                compressed_threshold = num_buildings - bisect_left(sorted_unique_heights,
                                                                  heights[left_person])
                result[query_idx] = fenwick_tree.query(compressed_threshold)

        return result
```

## Time and Space Complexity

**Time Complexity:** `O(m log m + (m + n) log n)`

The time complexity breaks down as follows:
- Sorting queries by their right endpoint: `O(m log m)` where `m` is the number of queries
- Creating sorted set of unique heights: `O(n log n)` where `n` is the length of heights array
- Main loop iterates through all `m` queries:
  - For each query, we potentially process buildings from position `j` down to `r`
  - Each building is processed at most once across all queries
  - For each building processed:
    - Binary search using `bisect_left`: `O(log n)`
    - BIT update operation: `O(log n)`
  - Total for processing all buildings: `O(n log n)`
  - For each query answer:
    - Binary search: `O(log n)`
    - BIT query operation: `O(log n)`
  - Total for all query answers: `O(m log n)`
- Overall: `O(m log m + n log n + n log n + m log n)` = `O(m log m + (m + n) log n)`

**Space Complexity:** `O(n + m)`

The space complexity consists of:
- Binary Indexed Tree array `c`: `O(n)`
- Sorted set of heights `s`: `O(n)` in worst case when all heights are unique
- Answer array: `O(m)`
- Sorting operations use `O(log m)` stack space for sorting queries
- Total: `O(n + m)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Height Comparison for Direct Meeting**
**Pitfall:** A common mistake is using `heights[left_person] <= heights[right_person]` instead of `heights[left_person] < heights[right_person]` when checking if two people can meet at the right person's position.

**Why it's wrong:** When `heights[left_person] == heights[right_person]`, the left person cannot move to the right person's building because the movement rule requires moving to a **strictly taller** building.

**Example:**
- `heights = [3, 5, 5, 2]`
- Query `[1, 2]` where both buildings have height 5
- The left person at index 1 cannot move to index 2 since `heights[1] = heights[2] = 5`

**Solution:** Always use strict inequality (`<`) when checking height conditions:
```python
if left_person == right_person or heights[left_person] < heights[right_person]:
    result[query_idx] = right_person
```

### 2. **Off-by-One Error in BIT Querying**
**Pitfall:** Using `compressed_threshold = num_buildings - bisect_left(sorted_unique_heights, heights[left_person]) + 1` instead of without the `+1` when querying the BIT.

**Why it's wrong:** The BIT query should find buildings **strictly taller** than `heights[left_person]`. Adding 1 would include buildings with the same height, violating the movement rule.

**Example:**
- If `heights[left_person] = 10` and we want buildings with height > 10
- The compressed rank for height 10 is at position `k`
- We need to query positions `1` to `k-1` (heights > 10), not `1` to `k` (heights >= 10)

**Solution:** Query the BIT with the correct threshold:
```python
compressed_threshold = num_buildings - bisect_left(sorted_unique_heights, heights[left_person])
result[query_idx] = fenwick_tree.query(compressed_threshold)
```

### 3. **Not Handling the Case When Both Start at Same Building**
**Pitfall:** Forgetting to handle `left_person == right_person` as a special case.

**Why it matters:** When both people start at the same building, they're already "meeting" at that building, regardless of height comparisons.

**Solution:** Always check this condition first:
```python
if left_person == right_person:  # Already at same building
    result[query_idx] = right_person
elif heights[left_person] < heights[right_person]:  # Can meet at right
    result[query_idx] = right_person
else:  # Need to find a taller building
    # ... BIT query logic
```

### 4. **Incorrect Query Normalization**
**Pitfall:** Not normalizing queries to ensure `left <= right`, or normalizing incorrectly.

**Why it's wrong:** The algorithm assumes the left person has a smaller or equal index. Without normalization, the logic breaks because we process buildings from right to left based on the rightmost person's position.

**Solution:** Always normalize at the beginning:
```python
for i in range(num_queries):
    queries[i] = [min(queries[i]), max(queries[i])]
```

### 5. **Using Wrong Comparison in BIT Update**
**Pitfall:** Using maximum instead of minimum in the BIT update operation.

**Why it's wrong:** We need the **leftmost** (minimum index) building that satisfies the height requirement. Using maximum would give us the rightmost building instead.

**Solution:** Ensure the BIT maintains minimum values:
```python
def update(self, index: int, value: int):
    while index <= self.size:
        self.tree[index] = min(self.tree[index], value)  # Use min, not max
        index += index & -index
```

