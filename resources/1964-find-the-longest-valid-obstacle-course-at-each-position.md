# 1964. Find the Longest Valid Obstacle Course at Each Position

> Source: https://algo.monster/liteproblems/1964

## Problem Statement

You are given an array `obstacles` where each element represents the height of an obstacle. Your task is to find the longest obstacle course that can be built ending at each position.

For each index `i` in the array, you need to determine the longest sequence of obstacles that:
- Ends with the obstacle at position `i` (this obstacle must be included)
- Only includes obstacles from positions `0` to `i`
- Maintains the original order of obstacles from the array
- Each obstacle in the sequence is greater than or equal to the previous one (non-decreasing heights)

The output should be an array where `ans[i]` represents the length of the longest valid obstacle course ending at position `i`.

For example, if `obstacles = [1, 2, 3, 2]`:
- At index 0: The longest course is just `[1]`, length = 1
- At index 1: The longest course is `[1, 2]`, length = 2
- At index 2: The longest course is `[1, 2, 3]`, length = 3
- At index 3: The longest course is `[1, 2, 2]` or `[2, 2]`, length = 3

The solution uses a Binary Indexed Tree (Fenwick Tree) to efficiently track the longest increasing subsequence ending at or before each height value. For each obstacle:
1. It finds all previously seen obstacles with height ≤ current obstacle height
2. Queries the maximum length among those sequences
3. Extends that sequence by 1 to include the current obstacle
4. Updates the tree with this new length

The coordinate compression technique (sorting unique values) is used to map obstacle heights to indices in the Binary Indexed Tree, allowing efficient range queries and updates in `O(log n)` time.

## Intuition

This problem is essentially asking for the Longest Non-Decreasing Subsequence (LIS variant) ending at each position. The key insight is that for each obstacle at position `i`, we need to find the longest valid course among all previous obstacles that have height ≤ current obstacle's height.

The naive approach would be to check all previous obstacles for each position, giving us `O(n²)` complexity. However, we can optimize this by recognizing a pattern: we're repeatedly asking "what's the maximum length among all sequences ending with height ≤ h?"

This leads us to think about maintaining some data structure that can:
1. Query the maximum value in a range (all heights ≤ current height)
2. Update a value at a specific position (update the length for current height)

A Binary Indexed Tree (BIT) is perfect for this because it supports both range maximum queries and point updates in `O(log n)` time.

The next challenge is handling the range of possible heights. Since obstacle heights could be large, we use coordinate compression - mapping the actual heights to indices `1, 2, 3, ...` based on their relative order. This ensures our BIT only needs to be as large as the number of unique heights.

The algorithm flow becomes:
- For each obstacle, find its compressed index (position in sorted unique values)
- Query BIT for maximum length among all heights up to this index
- The answer for current position is this maximum + 1
- Update BIT at this index with the new length

This way, we maintain a running record of the best possible length for each height value, allowing us to build upon previous results efficiently.

**Learn more about [Binary Search](/problems/binary_search_intro) patterns.**

## Solution Approach

The solution uses a Binary Indexed Tree (Fenwick Tree) with coordinate compression to efficiently solve this problem.

**Step 1: Coordinate Compression**
```python
nums = sorted(set(obstacles))
```
We extract unique obstacle heights and sort them. This maps potentially large height values to indices `[1, 2, ..., n]`. For example, if obstacles contain heights `[5, 100, 5, 50]`, the unique sorted values would be `[5, 50, 100]`, which map to indices `[1, 2, 3]`.

**Step 2: Binary Indexed Tree Setup**
```python
tree = BinaryIndexedTree(n)
```
We initialize a BIT of size `n` (number of unique heights). The BIT maintains the maximum length of obstacle courses for each height level.

The BIT implementation provides two key operations:
- `query(x)`: Returns the maximum value from indices `1` to `x`. This gives us the longest course among all heights ≤ current height.
- `update(x, v)`: Updates position `x` with value `v` if `v` is greater than the current value at `x`.

**Step 3: Process Each Obstacle**
```python
for x in obstacles:
    i = bisect_left(nums, x) + 1
    ans.append(tree.query(i) + 1)
    tree.update(i, ans[-1])
```

For each obstacle height `x`:
1. Find its compressed index using [binary search](/problems/binary_search_intro): `bisect_left(nums, x) + 1`. We add 1 because BIT uses 1-based indexing.
2. Query the maximum length among all heights ≤ `x`: `tree.query(i)`. This gives us the longest valid course we can extend.
3. The length at current position is `query_result + 1` (adding current obstacle).
4. Update the BIT at index `i` with this new length, so future obstacles can build upon it.

**Time Complexity**: `O(n log n)` where `n` is the number of obstacles. The sorting takes `O(n log n)`, and for each obstacle, we perform `O(log n)` operations ([binary search](/problems/binary_search_intro), BIT query, and BIT update).

**Space Complexity**: `O(n)` for storing the unique heights and the BIT structure.

## Example Walkthrough

Let's walk through the solution with `obstacles = [2, 5, 1, 5, 3]`.

**Step 1: Coordinate Compression**
- Extract unique heights: `{2, 5, 1, 3}`
- Sort them: `[1, 2, 3, 5]`
- These map to BIT indices: `1, 2, 3, 4` (1-based indexing)

**Step 2: Initialize BIT**
- Create BIT of size 4 (for 4 unique heights)
- Initially all values are 0: `BIT = [0, 0, 0, 0]`

**Step 3: Process Each Obstacle**

**Position 0: height = 2**
- Find compressed index: 2 is at position 1 in `[1, 2, 3, 5]`, so index = 2
- Query BIT(2): maximum among indices 1-2 = 0
- Length at position 0 = 0 + 1 = 1
- Update BIT at index 2 with value 1
- `BIT state: [0, 1, 0, 0]`, `ans = [1]`

**Position 1: height = 5**
- Find compressed index: 5 is at position 3 in `[1, 2, 3, 5]`, so index = 4
- Query BIT(4): maximum among indices 1-4 = 1 (from height 2)
- Length at position 1 = 1 + 1 = 2
- Update BIT at index 4 with value 2
- `BIT state: [0, 1, 0, 2]`, `ans = [1, 2]`

**Position 2: height = 1**
- Find compressed index: 1 is at position 0 in `[1, 2, 3, 5]`, so index = 1
- Query BIT(1): maximum among indices 1-1 = 0
- Length at position 2 = 0 + 1 = 1
- Update BIT at index 1 with value 1
- `BIT state: [1, 1, 0, 2]`, `ans = [1, 2, 1]`

**Position 3: height = 5**
- Find compressed index: 5 is at position 3 in `[1, 2, 3, 5]`, so index = 4
- Query BIT(4): maximum among indices 1-4 = 2 (previous sequence ending at height 5)
- Length at position 3 = 2 + 1 = 3
- Update BIT at index 4 with value 3
- `BIT state: [1, 1, 0, 3]`, `ans = [1, 2, 1, 3]`

**Position 4: height = 3**
- Find compressed index: 3 is at position 2 in `[1, 2, 3, 5]`, so index = 3
- Query BIT(3): maximum among indices 1-3 = 1 (can build on height 1 or 2)
- Length at position 4 = 1 + 1 = 2
- Update BIT at index 3 with value 2
- `BIT state: [1, 1, 2, 3]`, `ans = [1, 2, 1, 3, 2]`

**Final Result: `[1, 2, 1, 3, 2]`**

This means:
- Position 0: Course `[2]` has length 1
- Position 1: Course `[2, 5]` has length 2
- Position 2: Course `[1]` has length 1
- Position 3: Course `[2, 5, 5]` or `[1, 5, 5]` has length 3
- Position 4: Course `[2, 3]` or `[1, 3]` has length 2

## Python Solution

```python
from typing import List
from bisect import bisect_left


class BinaryIndexedTree:
    """
    Binary Indexed Tree (Fenwick Tree) modified for range maximum queries.
    Instead of sum operations, this BIT maintains maximum values.
    """
    __slots__ = ["size", "tree"]

    def __init__(self, size: int):
        """
        Initialize the Binary Indexed Tree.

        Args:
            size: The size of the tree (number of elements)
        """
        self.size = size
        self.tree = [0] * (size + 1)  # 1-indexed array for BIT operations

    def update(self, index: int, value: int) -> None:
        """
        Update the tree by propagating the maximum value upwards.

        Args:
            index: The position to update (1-indexed)
            value: The value to compare and potentially update with
        """
        while index <= self.size:
            # Update current position with maximum value
            self.tree[index] = max(self.tree[index], value)
            # Move to next index affected by this position
            # index & -index gives the lowest set bit
            index += index & -index

    def query(self, index: int) -> int:
        """
        Query the maximum value in range [1, index].

        Args:
            index: The right boundary of the query range (1-indexed)

        Returns:
            The maximum value in the range [1, index]
        """
        max_value = 0
        while index > 0:
            # Take maximum of current value and tree value at this index
            max_value = max(max_value, self.tree[index])
            # Move to previous range by removing lowest set bit
            index -= index & -index
        return max_value


class Solution:
    def longestObstacleCourseAtEachPosition(self, obstacles: List[int]) -> List[int]:
        """
        Find the length of the longest obstacle course ending at each position.
        An obstacle course is a subsequence where each element is >= the previous one.

        Args:
            obstacles: List of obstacle heights

        Returns:
            List where each element represents the length of the longest
            non-decreasing subsequence ending at that position
        """
        # Create a sorted list of unique obstacle heights for coordinate compression
        unique_heights = sorted(set(obstacles))
        num_unique = len(unique_heights)

        # Initialize Binary Indexed Tree with size equal to number of unique heights
        fenwick_tree = BinaryIndexedTree(num_unique)

        result = []

        for obstacle_height in obstacles:
            # Map the obstacle height to its compressed coordinate (1-indexed)
            # bisect_left finds the position where height would be inserted
            compressed_index = bisect_left(unique_heights, obstacle_height) + 1

            # Query the maximum length of subsequence ending with height <= current
            max_length_before = fenwick_tree.query(compressed_index)

            # Current position extends the best subsequence by 1
            current_length = max_length_before + 1
            result.append(current_length)

            # Update the tree with the new length at this height
            fenwick_tree.update(compressed_index, current_length)

        return result
```

## Time and Space Complexity

**Time Complexity: `O(n × log n)`**

The time complexity analysis breaks down as follows:
- Creating the sorted unique values: `sorted(set(obstacles))` takes `O(n log n)` time, where `n` is the number of obstacles
- The main loop iterates through all `n` obstacles
- For each obstacle:
  - `bisect_left()` performs binary search on the sorted array, taking `O(log n)` time
  - `tree.query(i)` traverses up the BIT using the operation `x -= x & -x`, which visits at most `O(log n)` nodes
  - `tree.update(i, ans[-1])` traverses up the BIT using the operation `x += x & -x`, which also visits at most `O(log n)` nodes
- Total for the loop: `n × O(log n) = O(n log n)`
- Overall time complexity: `O(n log n) + O(n log n) = O(n log n)`

**Space Complexity: `O(n)`**

The space complexity analysis:
- `nums` array stores at most `n` unique values: `O(n)`
- Binary Indexed Tree stores an array `c` of size `n + 1`: `O(n)`
- `ans` array stores `n` results: `O(n)`
- Total space complexity: `O(n) + O(n) + O(n) = O(n)`

Where `n` is the number of obstacles in the input array.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect BIT Indexing (0-based vs 1-based)**
One of the most common mistakes is mixing up 0-based and 1-based indexing. Binary Indexed Trees traditionally use 1-based indexing because the bit manipulation operations (`index & -index`) don't work correctly with index 0.

**Incorrect Implementation:**
```python
# Wrong: Using 0-based indexing
compressed_index = bisect_left(unique_heights, obstacle_height)  # Returns 0-based index
fenwick_tree.query(compressed_index)  # BIT expects 1-based index
```

**Correct Implementation:**
```python
# Correct: Convert to 1-based indexing
compressed_index = bisect_left(unique_heights, obstacle_height) + 1
fenwick_tree.query(compressed_index)
```

### 2. **Using Standard BIT Sum Operations Instead of Maximum**
The traditional BIT is designed for prefix sum queries. This problem requires finding the maximum value in a range, not the sum.

**Incorrect Implementation:**
```python
class BinaryIndexedTree:
    def update(self, index: int, value: int) -> None:
        while index <= self.size:
            self.tree[index] += value  # Wrong: Adding instead of taking maximum
            index += index & -index

    def query(self, index: int) -> int:
        sum_value = 0
        while index > 0:
            sum_value += self.tree[index]  # Wrong: Summing instead of taking maximum
            index -= index & -index
        return sum_value
```

**Correct Implementation:**
```python
class BinaryIndexedTree:
    def update(self, index: int, value: int) -> None:
        while index <= self.size:
            self.tree[index] = max(self.tree[index], value)  # Take maximum
            index += index & -index

    def query(self, index: int) -> int:
        max_value = 0
        while index > 0:
            max_value = max(max_value, self.tree[index])  # Take maximum
            index -= index & -index
        return max_value
```

### 3. **Forgetting Coordinate Compression for Large Values**
Without coordinate compression, if obstacle heights have large values (e.g., up to 10^7), creating a BIT of that size would cause memory issues and timeout.

**Incorrect Approach:**
```python
# Wrong: Using actual height values as indices
max_height = max(obstacles)
fenwick_tree = BinaryIndexedTree(max_height)  # Could be 10^7!
for obstacle_height in obstacles:
    fenwick_tree.query(obstacle_height)  # Direct use of height
```

**Correct Approach:**
```python
# Correct: Compress coordinates first
unique_heights = sorted(set(obstacles))
fenwick_tree = BinaryIndexedTree(len(unique_heights))  # Size is at most n
for obstacle_height in obstacles:
    compressed_index = bisect_left(unique_heights, obstacle_height) + 1
    fenwick_tree.query(compressed_index)  # Use compressed index
```

### 4. **Handling Duplicate Heights Incorrectly**
When multiple obstacles have the same height, they should all be able to extend sequences ending at that height. The BIT update should use `max` to preserve the best result.

**Potential Issue:**
If you mistakenly overwrite instead of taking the maximum during updates, you might lose optimal solutions for duplicate heights.

**Prevention:**
The current implementation correctly uses `max(self.tree[index], value)` in the update method, ensuring that we keep the best (longest) sequence length for each height level.

