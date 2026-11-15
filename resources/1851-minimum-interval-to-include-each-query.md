# 1851. Minimum Interval to Include Each Query

> Source: https://algo.monster/liteproblems/1851

## Problem Statement

You are given a 2D integer array `intervals` where each `intervals[i] = [left_i, right_i]` represents an interval that starts at `left_i` and ends at `right_i` (both endpoints are inclusive). The size of an interval is the number of integers it contains, calculated as `right_i - left_i + 1`.

You are also given an integer array `queries`. For each query `queries[j]`, you need to find the smallest interval that contains the query value. Specifically, you need to find the interval `i` with the minimum size such that `left_i <= queries[j] <= right_i`. If no interval contains the query value, the answer is `-1`.

Return an array containing the answers to all queries in their original order.

For example, if an interval is `[2, 5]`, its size is `5 - 2 + 1 = 4`, and it contains the integers 2, 3, 4, and 5. If you have a query value of 3, this interval would contain it since `2 <= 3 <= 5`.

The key challenge is to efficiently find the smallest interval for each query when dealing with potentially large numbers of intervals and queries.

## Intuition

The key observation is that the order in which we process queries doesn't affect the final answer - we just need to return results in the original query order. This opens up an optimization opportunity: we can sort the queries and process them in ascending order.

Why is [sorting](/problems/sorting_summary) helpful? Consider processing queries from smallest to largest. As we move to larger query values, we can incrementally add intervals that might be relevant. An interval becomes relevant when its left endpoint is less than or equal to the current query value. Once an interval's left endpoint satisfies this condition for a query value `x`, it will continue to satisfy it for all larger query values.

This suggests a sweep line approach: as we process queries in ascending order, we can maintain a set of "active" intervals - those whose left endpoints we've already passed. We add intervals to this set as their left endpoints become less than or equal to the current query.

However, not all active intervals are valid for the current query. An interval is only valid if it also contains the query value on the right side, meaning `right_i >= query`. Intervals that end before the current query value should be removed from consideration.

To efficiently track and retrieve the smallest valid interval, we use a min heap. The heap stores intervals by their size (as the priority) along with their right endpoint. This allows us to:
1. Quickly access the smallest interval (heap top)
2. Remove intervals that have become invalid (right endpoint < current query)

The algorithm naturally handles the incremental nature of the problem: as we move through sorted queries, we incrementally add new intervals that become relevant and remove old intervals that are no longer valid, maintaining an optimal set of candidates at each step.

**Learn more about [Binary Search](/problems/binary_search_intro), [Sorting](/problems/sorting_summary), [Line Sweep](/problems/line_sweep_intro) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

The implementation follows a [sorting](/problems/sorting_summary) + offline query + priority queue strategy:

**1. Preprocessing:**
- Sort `intervals` by their left endpoints in ascending order
- Create a list of tuples `(query_value, original_index)` for each query and sort by query value
- Initialize answer array `ans` with `-1` values (default for queries with no valid interval)

**2. Main Processing Loop:**

We iterate through sorted queries `(x, j)` where `x` is the query value and `j` is its original index:

**Adding relevant intervals to the heap:**
```python
while i < n and intervals[i][0] <= x:
    a, b = intervals[i]
    heappush(pq, (b - a + 1, b))
    i += 1
```
- Use pointer `i` to track unprocessed intervals
- For each interval `[a, b]` whose left endpoint `a <= x`, add it to the min heap
- Store tuple `(interval_size, right_endpoint)` where `interval_size = b - a + 1`
- The heap is ordered by interval size (smallest first)

**Removing invalid intervals:**
```python
while pq and pq[0][1] < x:
    heappop(pq)
```
- Remove intervals from the heap whose right endpoint is less than current query `x`
- These intervals no longer contain the query value and are invalid

**Recording the answer:**
```python
if pq:
    ans[j] = pq[0][0]
```
- If heap is not empty, the top element has the minimum size among all valid intervals
- Store this size at the original query position `j` in the answer array

**3. Key Data Structures:**
- **Min Heap (`pq`)**: Maintains valid intervals ordered by size. Each element is `(size, right_endpoint)`
- **Pointer (`i`)**: Tracks the next unprocessed interval to avoid redundant checks
- **Answer Array (`ans`)**: Stores results in original query order using the preserved index `j`

The algorithm efficiently processes each interval and query exactly once, using the sorted order to maintain only the relevant intervals in memory at any given time.

## Example Walkthrough

Let's walk through a concrete example to illustrate the solution approach.

**Input:**
- `intervals = [[1, 4], [2, 3], [3, 6], [5, 7]]`
- `queries = [4, 2, 6, 3]`

**Step 1: Preprocessing**

First, sort intervals by left endpoint:
- `intervals = [[1, 4], [2, 3], [3, 6], [5, 7]]` (already sorted)

Create query tuples with original indices and sort by value:
- Original: `queries = [4, 2, 6, 3]` with indices `[0, 1, 2, 3]`
- Query tuples: `[(4, 0), (2, 1), (6, 2), (3, 3)]`
- Sorted: `[(2, 1), (3, 3), (4, 0), (6, 2)]`

Initialize answer array: `ans = [-1, -1, -1, -1]`

**Step 2: Process Sorted Queries**

Initialize: `i = 0`, `pq = []` (min heap)

**Query (2, 1):** value = 2, original index = 1
- Add intervals with left ≤ 2:
  - `[1, 4]`: size = 4, add `(4, 4)` to heap
  - `[2, 3]`: size = 2, add `(2, 3)` to heap
- Heap state: `[(2, 3), (4, 4)]`
- Remove intervals with right < 2: none
- Smallest valid interval: size = 2
- `ans[1] = 2`

**Query (3, 3):** value = 3, original index = 3
- Add intervals with left ≤ 3:
  - `[3, 6]`: size = 4, add `(4, 6)` to heap
- Heap state: `[(2, 3), (4, 4), (4, 6)]`
- Remove intervals with right < 3: none
- Smallest valid interval: size = 2 (from interval `[2, 3]`)
- `ans[3] = 2`

**Query (4, 0):** value = 4, original index = 0
- Add intervals with left ≤ 4: none (all processed)
- Heap state: `[(2, 3), (4, 4), (4, 6)]`
- Remove intervals with right < 4:
  - Remove `(2, 3)` since right endpoint 3 < 4
- Heap state: `[(4, 4), (4, 6)]`
- Smallest valid interval: size = 4 (from interval `[1, 4]`)
- `ans[0] = 4`

**Query (6, 2):** value = 6, original index = 2
- Add intervals with left ≤ 6:
  - `[5, 7]`: size = 3, add `(3, 7)` to heap
- Heap state: `[(3, 7), (4, 4), (4, 6)]`
- Remove intervals with right < 6:
  - Remove `(4, 4)` since right endpoint 4 < 6
- Heap state: `[(3, 7), (4, 6)]`
- Smallest valid interval: size = 3 (from interval `[5, 7]`)
- `ans[2] = 3`

**Step 3: Return Result**

Final answer array: `ans = [4, 2, 3, 2]`

This corresponds to:
- `queries[0] = 4`: smallest containing interval is `[1, 4]` with size 4
- `queries[1] = 2`: smallest containing interval is `[2, 3]` with size 2
- `queries[2] = 6`: smallest containing interval is `[5, 7]` with size 3
- `queries[3] = 3`: smallest containing interval is `[2, 3]` with size 2

The key insight demonstrated is how sorting queries allows us to incrementally maintain only the relevant intervals in the heap, efficiently finding the smallest valid interval for each query.

## Python Solution

```python
class Solution:
    def minInterval(self, intervals: List[List[int]], queries: List[int]) -> List[int]:
        """
        Find the smallest interval size containing each query point.

        Args:
            intervals: List of intervals [start, end]
            queries: List of query points

        Returns:
            List where each element is the size of smallest interval containing that query,
            or -1 if no interval contains it
        """
        # Get the number of intervals and queries
        num_intervals = len(intervals)
        num_queries = len(queries)

        # Sort intervals by start time for efficient processing
        intervals.sort()

        # Create indexed queries and sort by query value
        # This allows us to process queries in order while preserving original indices
        indexed_queries = sorted((query_value, index) for index, query_value in enumerate(queries))

        # Initialize result array with -1 (default when no interval contains the query)
        result = [-1] * num_queries

        # Min heap to store intervals: (interval_size, interval_end)
        # Smallest interval size will be at the top
        min_heap = []

        # Pointer to track which interval we're currently processing
        interval_index = 0

        # Process each query in sorted order
        for query_value, original_index in indexed_queries:
            # Add all intervals that start before or at the current query point
            while interval_index < num_intervals and intervals[interval_index][0] <= query_value:
                start, end = intervals[interval_index]
                interval_size = end - start + 1
                heappush(min_heap, (interval_size, end))
                interval_index += 1

            # Remove intervals from heap that end before the current query point
            # These intervals don't contain the query point
            while min_heap and min_heap[0][1] < query_value:
                heappop(min_heap)

            # If heap is not empty, the top element is the smallest valid interval
            if min_heap:
                result[original_index] = min_heap[0][0]

        return result
```

## Time and Space Complexity

**Time Complexity:** `O(n × log n + m × log m)`

The time complexity breaks down as follows:
- Sorting intervals takes `O(n × log n)` where `n` is the number of intervals
- Creating and sorting the queries list with indices takes `O(m × log m)` where `m` is the number of queries
- The main loop iterates through all `m` queries:
  - The inner while loop for adding intervals runs at most `n` times total across all iterations (each interval is added once)
  - Each heap push operation takes `O(log n)` time
  - The while loop for removing expired intervals also runs at most `n` times total across all iterations
  - Each heap pop operation takes `O(log n)` time
- Total heap operations: at most `n` pushes and `n` pops, giving `O(n × log n)`
- Overall: `O(n × log n + m × log m + n × log n)` = `O(n × log n + m × log m)`

**Space Complexity:** `O(n + m)`

The space complexity consists of:
- The sorted queries list with indices: `O(m)`
- The answer array: `O(m)`
- The priority queue can contain at most `n` intervals: `O(n)`
- Total: `O(n + m)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Not Handling the Original Query Order**

**Pitfall:** A common mistake is sorting the queries for efficient processing but forgetting to preserve their original indices, leading to results being returned in the wrong order.

**Wrong approach:**
```python
# Sorting queries without preserving indices
queries.sort()
# Process queries...
# Results will be in sorted query order, not original order!
```

**Correct approach:**
```python
# Create indexed queries to preserve original positions
indexed_queries = sorted((query_value, index) for index, query_value in enumerate(queries))
# Use original_index to place results in correct position
result[original_index] = min_heap[0][0]
```

### 2. **Incorrect Heap Cleanup Logic**

**Pitfall:** Using the wrong comparison when removing invalid intervals from the heap. The condition should check if the interval's right endpoint is **less than** the query (not less than or equal).

**Wrong approach:**
```python
# This removes intervals that could still be valid
while min_heap and min_heap[0][1] <= query_value:  # Wrong!
    heappop(min_heap)
```

**Correct approach:**
```python
# Only remove intervals that end before the query point
while min_heap and min_heap[0][1] < query_value:  # Correct!
    heappop(min_heap)
```

### 3. **Reprocessing Intervals for Each Query**

**Pitfall:** Resetting the interval index for each query, causing intervals to be processed multiple times and degrading performance to O(n*m).

**Wrong approach:**
```python
for query_value, original_index in indexed_queries:
    interval_index = 0  # Resetting for each query - inefficient!
    while interval_index < num_intervals and intervals[interval_index][0] <= query_value:
        # Process interval...
```

**Correct approach:**
```python
interval_index = 0  # Initialize once outside the loop
for query_value, original_index in indexed_queries:
    # Continue from where we left off
    while interval_index < num_intervals and intervals[interval_index][0] <= query_value:
        # Process interval...
        interval_index += 1  # Move forward, never reset
```

### 4. **Storing Incorrect Information in the Heap**

**Pitfall:** Storing only the interval size in the heap without the right endpoint, making it impossible to determine when an interval becomes invalid.

**Wrong approach:**
```python
# Missing right endpoint information
heappush(min_heap, interval_size)  # Can't check validity later!
```

**Correct approach:**
```python
# Store both size and right endpoint
heappush(min_heap, (interval_size, end))
# Now we can check if end < query_value to determine validity
```

### 5. **Off-by-One Error in Size Calculation**

**Pitfall:** Forgetting that intervals are inclusive on both ends, leading to incorrect size calculation.

**Wrong approach:**
```python
interval_size = end - start  # Missing the +1 for inclusive endpoints
```

**Correct approach:**
```python
interval_size = end - start + 1  # Correct for inclusive intervals
```

These pitfalls can cause incorrect results or significant performance degradation. The key insight is that the algorithm relies on maintaining state across queries (the interval pointer and heap) while properly tracking the original query order.

