# 1847. Closest Room

> Source: https://algo.monster/liteproblems/1847

## Problem Statement

You are given information about a hotel with `n` rooms, where each room has a unique room number and a size. The rooms are represented as a 2D array `rooms` where `rooms[i] = [roomId_i, size_i]` means there is a room with number `roomId_i` and size `size_i`.

You need to process `k` queries about room assignments. Each query is given as `queries[j] = [preferred_j, minSize_j]`, where:
- `preferred_j` is the preferred room number for this query
- `minSize_j` is the minimum acceptable room size for this query

For each query, you need to find a room that:
1. Has a size of at least `minSize_j` (the room must be large enough)
2. Has a room number `id` that minimizes the absolute difference `abs(id - preferred_j)` (the room number should be as close as possible to the preferred room number)

If multiple rooms have the same absolute difference from the preferred room number (a tie), choose the room with the smaller room number. If no room meets the size requirement, return `-1` for that query.

The output should be an array `answer` of length `k`, where `answer[j]` contains the room number assigned for the `j-th` query, or `-1` if no suitable room exists.

For example, if the preferred room number is 5 and there are two rooms with numbers 3 and 7 that both meet the size requirement, both have an absolute difference of 2 from the preferred number. In this case, you would choose room 3 since it has the smaller room number.

## Intuition

The key insight is that we can process queries in a specific order to optimize our search. Since each query has a minimum size requirement, if we process queries from smallest to largest minimum size, we can progressively eliminate rooms that are too small, rather than checking all rooms for each query.

Think about it this way: if we sort queries by their `minSize` in ascending order, once a room becomes too small for a query, it will remain too small for all subsequent queries (since they have larger `minSize` requirements). This means we can remove small rooms from consideration permanently as we process queries.

Here's the clever approach:
1. Start by considering all rooms as available options
2. Sort queries by their `minSize` (smallest first)
3. For each query, remove all rooms that are now too small
4. Among the remaining valid rooms, find the one closest to the preferred room number

To efficiently find the closest room number, we need a data structure that:
- Allows us to remove elements (rooms that become too small)
- Supports [binary search](/problems/binary_search_intro) to find the closest value to our preferred room number

This leads us to use an ordered set (like `SortedList` in Python). With [binary search](/problems/binary_search_intro), we can find where the preferred room number would fit in the sorted list, then check the rooms immediately before and after that position to determine which is closest.

The reason we process queries "offline" (not in their original order) is that this [sorting](/problems/sorting_summary) optimization significantly reduces repeated work. After processing all queries in this optimized order, we rearrange the answers back to match the original query order.

This transforms what could be an `O(k * n)` problem (checking all rooms for each query) into a more efficient solution where each room is considered for removal only once, and finding the closest room uses [binary search](/problems/binary_search_intro) `O(log n)`.

**Learn more about [Binary Search](/problems/binary_search_intro) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

The solution implements an offline query approach with [binary search](/problems/binary_search_intro) template to find the closest room ID.

**1. Sort and prepare data:**
- Sort rooms by size in ascending order
- Create sorted query indices by minimum size requirement

**2. Binary Search Template for Finding Closest Room:**

We use the template to find the **first room ID >= preferred ID**:

```python
left, right = 0, n - 1
first_true_index = -1

while left <= right:
    mid = (left + right) // 2
    if ids[mid] >= preferred_id:
        first_true_index = mid
        right = mid - 1
    else:
        left = mid + 1
```

The feasibility condition `ids[mid] >= preferred_id` creates the pattern:
- Room IDs smaller than preferred: return `false`
- Room IDs at or after preferred: return `true`

**3. Handle the result:**

```python
# Check room at or after preferred_id
if first_true_index != -1:
    result[query_idx] = ids[first_true_index]

# Check room before preferred_id and choose closer one
if first_true_index == -1:
    # All rooms < preferred_id, closest is the last one
    result[query_idx] = ids[n - 1]
elif first_true_index > 0:
    left_room_id = ids[first_true_index - 1]
    if result[query_idx] - preferred_id >= preferred_id - left_room_id:
        result[query_idx] = left_room_id
```

When `first_true_index == -1`, all room IDs are less than preferred, so the closest is the largest (last) room ID.

**Time Complexity:** `O(n log n + k log k + k log n)`
**Space Complexity:** `O(n + k)`

## Example Walkthrough

Let's walk through a concrete example to illustrate the solution approach.

**Given:**
- Rooms: `[[1, 4], [2, 3], [3, 5], [4, 1]]`
- Queries: `[[2, 3], [3, 4], [5, 2]]`

**Step 1: Sort rooms by size**
After sorting: `[[4, 1], [2, 3], [1, 4], [3, 5]]`
- Room 4 has size 1
- Room 2 has size 3
- Room 1 has size 4
- Room 3 has size 5

**Step 2: Create sorted query indices**
Query minimum sizes: [3, 4, 2]
Sorted indices: `[2, 0, 1]` (process query 2 first with minSize=2, then query 0 with minSize=3, then query 1 with minSize=4)

**Step 3: Initialize SortedList with all room IDs**
`sl = [1, 2, 3, 4]` (sorted room numbers)

**Step 4: Process queries in sorted order**

**Processing Query 2** (preferred=5, minSize=2):
- Remove rooms with size < 2: Remove room 4 (size 1)
- `sl = [1, 2, 3]`
- Find closest to 5: `bisect_left(5) = 3` (would insert at end)
- Check candidate at p-1=2: room 3 has distance |3-5|=2
- Answer for query 2: room 3

**Processing Query 0** (preferred=2, minSize=3):
- Remove rooms with size < 3: No rooms to remove (room 2 has size 3)
- `sl = [1, 2, 3]`
- Find closest to 2: `bisect_left(2) = 1` (position of room 2)
- Check candidate at p=1: room 2 has distance |2-2|=0
- Check candidate at p-1=0: room 1 has distance |1-2|=1
- Room 2 is closer (distance 0 vs 1)
- Answer for query 0: room 2

**Processing Query 1** (preferred=3, minSize=4):
- Remove rooms with size < 4: Remove room 2 (size 3)
- `sl = [1, 3]`
- Find closest to 3: `bisect_left(3) = 1` (position of room 3)
- Check candidate at p=1: room 3 has distance |3-3|=0
- Check candidate at p-1=0: room 1 has distance |1-3|=2
- Room 3 is closer (distance 0 vs 2)
- Answer for query 1: room 3

**Step 5: Arrange answers in original query order**
- Query 0 → room 2
- Query 1 → room 3
- Query 2 → room 3

**Final Answer:** `[2, 3, 3]`

This example demonstrates how:
- Processing queries by ascending minSize allows us to progressively remove small rooms
- Binary search efficiently finds the closest room among valid options
- The offline approach avoids redundant work by removing each room at most once

## Python Solution

```python
from typing import List
from sortedcontainers import SortedList


class Solution:
    def closestRoom(
        self, rooms: List[List[int]], queries: List[List[int]]
    ) -> List[int]:
        rooms.sort(key=lambda room: room[1])
        num_queries = len(queries)
        sorted_query_indices = sorted(range(num_queries), key=lambda i: queries[i][1])
        result = [-1] * num_queries
        room_index = 0
        num_rooms = len(rooms)
        valid_room_ids = SortedList(room[0] for room in rooms)

        def find_first_ge(target: int) -> int:
            """Find first room ID >= target using binary search template."""
            ids = list(valid_room_ids)
            left, right = 0, len(ids) - 1
            first_true_index = -1

            while left <= right:
                mid = (left + right) // 2
                if ids[mid] >= target:
                    first_true_index = mid
                    right = mid - 1
                else:
                    left = mid + 1

            return first_true_index

        for query_idx in sorted_query_indices:
            preferred_id, min_size = queries[query_idx]

            while room_index < num_rooms and rooms[room_index][1] < min_size:
                valid_room_ids.remove(rooms[room_index][0])
                room_index += 1

            if room_index == num_rooms:
                break

            ids = list(valid_room_ids)
            n = len(ids)

            # Find first room ID >= preferred_id
            left, right = 0, n - 1
            first_true_index = -1

            while left <= right:
                mid = (left + right) // 2
                if ids[mid] >= preferred_id:
                    first_true_index = mid
                    right = mid - 1
                else:
                    left = mid + 1

            # Check room at or after preferred_id
            if first_true_index != -1:
                result[query_idx] = ids[first_true_index]

            # Check room before preferred_id and choose closer one
            if first_true_index == -1:
                # All rooms < preferred_id, closest is the last one
                result[query_idx] = ids[n - 1]
            elif first_true_index > 0:
                left_room_id = ids[first_true_index - 1]
                if result[query_idx] - preferred_id >= preferred_id - left_room_id:
                    result[query_idx] = left_room_id

        return result
```

## Time and Space Complexity

**Time Complexity:** `O(n × log n + k × log k + n × log n + k × log n)`

Breaking down the operations:
- Sorting rooms by size: `O(n × log n)`
- Sorting query indices by minSize: `O(k × log k)`
- Initial SortedList construction: `O(n × log n)` (inserting n elements, each taking `O(log n)`)
- For each query processing:
  - Removing rooms from SortedList: Each removal takes `O(log n)`, and we remove at most `n` rooms total across all queries, giving `O(n × log n)`
  - Binary search (bisect_left) on SortedList: `O(log n)` per query, totaling `O(k × log n)`
  - Accessing elements and comparisons: `O(1)` per query

The dominant terms simplify to: `O(n × log n + k × log k + k × log n)`

However, the reference answer gives `O(n × log n + k × log k)`, which appears to be overlooking the cost of SortedList operations. The actual complexity includes the additional `O(n × log n)` for SortedList construction and `O(k × log n)` for binary searches.

**Space Complexity:** `O(n + k)`

Breaking down the space usage:
- Sorted rooms list: `O(n)` (in-place sort)
- Query indices array: `O(k)`
- Answer array: `O(k)`
- SortedList: `O(n)` (stores at most n room IDs)

Total space complexity is `O(n + k)`, which matches the reference answer.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Using Wrong Binary Search Template Variant**

**Pitfall**: Using `bisect_left` directly without the standard template structure.

**Wrong Implementation**:
```python
p = valid_room_ids.bisect_left(preferred_id)
result[query_idx] = valid_room_ids[p]
```

**Correct Implementation** (using template):
```python
left, right = 0, n - 1
first_true_index = -1

while left <= right:
    mid = (left + right) // 2
    if ids[mid] >= preferred_id:
        first_true_index = mid
        right = mid - 1
    else:
        left = mid + 1
```

### 2. **Incorrect Tie-Breaking When Distances Are Equal**

**Pitfall**: Using `>` instead of `>=` when comparing distances.

**Solution**: Use `>=` to prefer smaller room ID when distances are equal:
```python
if result[query_idx] - preferred_id >= preferred_id - left_room_id:
    result[query_idx] = left_room_id
```

### 3. **Not Handling first_true_index = -1**

**Pitfall**: Not handling the case when all room IDs are less than preferred.

**Solution**: When `first_true_index == -1`, the closest room is the last one:
```python
if first_true_index == -1:
    result[query_idx] = ids[n - 1]
```

### 4. **Processing Queries in Original Order**

**Pitfall**: Processing queries in their original order would require re-adding rooms.

**Solution**: Process queries sorted by `minSize` and use an index array to maintain the mapping.

### 5. **Not Checking Bounds Before Accessing Elements**

**Pitfall**: Accessing `ids[first_true_index]` or `ids[first_true_index - 1]` without bounds checking.

**Solution**: Check `first_true_index != -1` and `first_true_index > 0` before accessing.

