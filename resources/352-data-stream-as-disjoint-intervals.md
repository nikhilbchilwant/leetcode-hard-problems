# 352. Data Stream as Disjoint Intervals

> Source: https://algo.monster/liteproblems/352

## Problem Statement

This problem asks you to design a data structure that can efficiently summarize a stream of integers as a collection of disjoint intervals.

You need to implement a `SummaryRanges` class with the following functionality:

1. **Constructor** `SummaryRanges()`: Creates an empty object ready to receive integers from a data stream.

2. **Method** `addNum(int value)`: Takes a non-negative integer and adds it to the data structure. As numbers are added, they should be incorporated into existing intervals when possible.

3. **Method** `getIntervals()`: Returns the current summary of all numbers seen so far as a list of disjoint intervals. Each interval is represented as `[start, end]` where both start and end are inclusive. The intervals must be sorted by their starting values.

The key challenge is to maintain these intervals efficiently as new numbers arrive. When a new number is added:
- If it can extend an existing interval (adjacent to its boundaries), the interval should be extended
- If it can merge two separate intervals (filling a gap of exactly 1), those intervals should be merged
- If it falls within an existing interval, no change is needed
- If it's isolated from all existing intervals, it creates a new single-point interval `[value, value]`

For example, if you've seen numbers 1, 3, 7, 2, 6:
- After 1: intervals are `[[1,1]]`
- After 1,3: intervals are `[[1,1], [3,3]]`
- After 1,3,7: intervals are `[[1,1], [3,3], [7,7]]`
- After 1,3,7,2: intervals are `[[1,3], [7,7]]` (1,2,3 form a continuous range)
- After 1,3,7,2,6: intervals are `[[1,3], [6,7]]` (6,7 form a continuous range)

## Intuition

The core insight is that we need to efficiently maintain sorted, non-overlapping intervals while handling new numbers that could affect existing intervals in various ways. This naturally suggests using a sorted data structure where we can quickly find where a new number fits relative to existing intervals.

When we add a new number `val`, we need to determine its relationship with existing intervals. The key observation is that a number can interact with at most two intervals - the one immediately to its left and the one immediately to its right. This is because intervals are disjoint.

We can identify four scenarios when adding a number:
1. **Bridge scenario**: The number connects two adjacent intervals (when left interval ends at `val-1` and right interval starts at `val+1`)
2. **Extend left**: The number extends or falls within the left interval (when `val` is at most one more than the left interval's end)
3. **Extend right**: The number extends or falls within the right interval (when `val` is at most one less than the right interval's start)
4. **Isolated**: The number creates a new interval by itself

Using a `SortedDict` (ordered map) with interval start points as keys makes sense because:
- We can use binary search (`bisect_right`) to quickly find where a new number fits
- We maintain intervals sorted by start position automatically
- We can efficiently access and modify adjacent intervals

The clever part is storing each interval as `[start, end]` with the start as the key. When we find where `val` fits using `bisect_right`, we immediately know:
- The interval to the right (if it exists) starts at the key found by `bisect_right`
- The interval to the left (if it exists) is the one just before that position

This approach gives us `O(log n)` time for adding numbers and `O(n)` space, where `n` is the number of disjoint intervals, making it efficient for streams where numbers tend to cluster into ranges.

## Solution Approach

The solution uses a `SortedDict` data structure to maintain intervals sorted by their starting points. Here's how each component works:

**Initialization:**
```python
def __init__(self):
    self.mp = SortedDict()
```
We initialize an empty `SortedDict` where keys are interval start points and values are `[start, end]` pairs.

**Adding a Number:**

When `addNum(val)` is called, we follow these steps:

1. **Find position using binary search:**
   ```python
   ridx = self.mp.bisect_right(val)
   lidx = n if ridx == 0 else ridx - 1
   ```
   - `ridx` finds the index of the first interval whose start is greater than `val`
   - `lidx` is the index of the interval immediately to the left (or `n` if none exists)

2. **Handle the merge case (bridging two intervals):**
   ```python
   if (lidx != n and ridx != n and 
       values[lidx][1] + 1 == val and 
       values[ridx][0] - 1 == val):
       self.mp[keys[lidx]][1] = self.mp[keys[ridx]][1]
       self.mp.pop(keys[ridx])
   ```
   If `val` perfectly bridges two intervals (left ends at `val-1`, right starts at `val+1`), merge them by extending the left interval to include the right interval's end, then remove the right interval.

3. **Handle extending or falling within left interval:**
   ```python
   elif lidx != n and val <= values[lidx][1] + 1:
       self.mp[keys[lidx]][1] = max(val, self.mp[keys[lidx]][1])
   ```
   If `val` is at most one more than the left interval's end, extend the left interval. The `max` ensures we don't shrink an interval if `val` is already within it.

4. **Handle extending or falling within right interval:**
   ```python
   elif ridx != n and val >= values[ridx][0] - 1:
       self.mp[keys[ridx]][0] = min(val, self.mp[keys[ridx]][0])
   ```
   If `val` is at most one less than the right interval's start, extend the right interval leftward. Since we're modifying the start point (which is the key), we update the value's start component.

5. **Create new isolated interval:**
   ```python
   else:
       self.mp[val] = [val, val]
   ```
   If `val` doesn't connect with any existing intervals, create a new single-point interval `[val, val]`.

**Getting Intervals:**
```python
def getIntervals(self) -> List[List[int]]:
    return list(self.mp.values())
```
Simply return all interval values as a list. Since `SortedDict` maintains order by keys (start points), the result is automatically sorted.

**Time Complexity:**
- `addNum`: O(log n) for binary search and O(1) for updates, where n is the number of intervals
- `getIntervals`: O(n) to convert values to a list

**Space Complexity:** O(n) where n is the maximum number of disjoint intervals.

## Example Walkthrough

Let's walk through adding the sequence: 5, 2, 7, 6, 1

**Initial state:** `mp = {}` (empty SortedDict)

**Step 1: addNum(5)**
- `bisect_right(5)` returns 0 (no intervals exist)
- `ridx = 0`, `lidx = 0` (n = 0, so both are "out of bounds")
- No intervals to merge or extend
- Create new interval: `mp = {5: [5,5]}`
- Intervals: `[[5,5]]`

**Step 2: addNum(2)**
- Current: `mp = {5: [5,5]}`
- `bisect_right(2)` returns 0 (2 < 5)
- `ridx = 0` (interval at key 5), `lidx = 1` (no left interval)
- Check right interval: `[5,5]`
  - Is `2 >= 5-1`? No (2 < 4)
- Create new interval: `mp = {2: [2,2], 5: [5,5]}`
- Intervals: `[[2,2], [5,5]]`

**Step 3: addNum(7)**
- Current: `mp = {2: [2,2], 5: [5,5]}`
- `bisect_right(7)` returns 2 (7 > all existing keys)
- `ridx = 2` (no right interval), `lidx = 1` (interval at key 5)
- Check left interval: `[5,5]`
  - Is `7 <= 5+1`? No (7 > 6)
- Create new interval: `mp = {2: [2,2], 5: [5,5], 7: [7,7]}`
- Intervals: `[[2,2], [5,5], [7,7]]`

**Step 4: addNum(6)**
- Current: `mp = {2: [2,2], 5: [5,5], 7: [7,7]}`
- `bisect_right(6)` returns 2 (6 > 5 but < 7)
- `ridx = 2` (interval at key 7), `lidx = 1` (interval at key 5)
- Check merge condition:
  - Left interval ends at 5, right interval starts at 7
  - Is `5+1 == 6` and `7-1 == 6`? YES!
- Merge intervals: Extend `[5,5]` to `[5,7]`, remove `[7,7]`
- Result: `mp = {2: [2,2], 5: [5,7]}`
- Intervals: `[[2,2], [5,7]]`

**Step 5: addNum(1)**
- Current: `mp = {2: [2,2], 5: [5,7]}`
- `bisect_right(1)` returns 0 (1 < 2)
- `ridx = 0` (interval at key 2), `lidx = 2` (no left interval)
- Check right interval: `[2,2]`
  - Is `1 >= 2-1`? YES! (1 >= 1)
- Extend right interval leftward: `[2,2]` becomes `[1,2]`
- Update: `mp[2][0] = 1`
- Result: `mp = {2: [1,2], 5: [5,7]}`
- Intervals: `[[1,2], [5,7]]`

**Final Summary:**
The stream 5, 2, 7, 6, 1 produces intervals `[[1,2], [5,7]]`. Notice how:
- Individual numbers started as single-point intervals
- Number 6 bridged the gap between [5,5] and [7,7], merging them
- Number 1 extended the interval [2,2] leftward to become [1,2]

## Python Solution

```python
from sortedcontainers import SortedDict
from typing import List

class SummaryRanges:
    """
    A class to maintain disjoint intervals as numbers are added to a stream.
    When a number is added, it either creates a new interval, extends an existing one,
    or merges two adjacent intervals.
    """
  
    def __init__(self):
        """Initialize with an empty sorted dictionary to store intervals."""
        # SortedDict maintains intervals sorted by their start points (keys)
        # Each key maps to [start, end] of an interval
        self.intervals_map = SortedDict()

    def addNum(self, val: int) -> None:
        """
        Add a number to the data stream and update intervals accordingly.
      
        Args:
            val: The integer value to add to the stream
        """
        num_intervals = len(self.intervals_map)
      
        # Find the position where val would be inserted (right bisection)
        right_idx = self.intervals_map.bisect_right(val)
      
        # Find the index of the interval that might contain or be adjacent to val
        # If right_idx is 0, no interval starts before val, so set left_idx to num_intervals (invalid)
        # Otherwise, left_idx is the interval just before where val would be inserted
        left_idx = num_intervals if right_idx == 0 else right_idx - 1
      
        # Get references to keys and values for easier access
        interval_keys = self.intervals_map.keys()
        interval_values = self.intervals_map.values()
      
        # Case 1: val connects two adjacent intervals (merging case)
        # Check if val is exactly between two intervals (end of left + 1 == val == start of right - 1)
        if (left_idx != num_intervals and 
            right_idx != num_intervals and 
            interval_values[left_idx][1] + 1 == val and 
            interval_values[right_idx][0] - 1 == val):
            # Merge the two intervals by extending the left interval to include the right
            self.intervals_map[interval_keys[left_idx]][1] = self.intervals_map[interval_keys[right_idx]][1]
            # Remove the right interval as it's now merged
            self.intervals_map.pop(interval_keys[right_idx])
          
        # Case 2: val extends or is within the left interval
        elif left_idx != num_intervals and val <= interval_values[left_idx][1] + 1:
            # Extend the end of the left interval if val is beyond it
            # If val is already within the interval, keep the current end
            self.intervals_map[interval_keys[left_idx]][1] = max(val, self.intervals_map[interval_keys[left_idx]][1])
          
        # Case 3: val extends or is at the beginning of the right interval
        elif right_idx != num_intervals and val >= interval_values[right_idx][0] - 1:
            # Extend the start of the right interval to include val
            # Use min to handle case where val might already be in the interval
            self.intervals_map[interval_keys[right_idx]][0] = min(val, self.intervals_map[interval_keys[right_idx]][0])
          
        # Case 4: val creates a new isolated interval
        else:
            # Create a new interval containing only val
            self.intervals_map[val] = [val, val]

    def getIntervals(self) -> List[List[int]]:
        """
        Get all current disjoint intervals in sorted order.
      
        Returns:
            A list of intervals, where each interval is [start, end]
        """
        return list(self.intervals_map.values())


# Your SummaryRanges object will be instantiated and called as such:
# obj = SummaryRanges()
# obj.addNum(val)
# param_2 = obj.getIntervals()
```

## Time and Space Complexity

**Time Complexity:**

- `__init__()`: `O(1)` - Simply initializes an empty SortedDict.

- `addNum(val)`: `O(log n)` where `n` is the number of intervals in the SortedDict.
  - `bisect_right(val)`: `O(log n)` - Binary search in the sorted keys.
  - Accessing keys and values lists: `O(1)` - These are views in SortedDict.
  - All merge operations (checking conditions and updating intervals): `O(1)`.
  - `pop()` operation in worst case (merging two intervals): `O(log n)`.
  - Insertion of new interval: `O(log n)`.
  - Overall: `O(log n)` per operation.

- `getIntervals()`: `O(n)` where `n` is the number of intervals.
  - Converting the values view to a list requires iterating through all intervals.

**Space Complexity:**

- Overall space: `O(n)` where `n` is the number of disjoint intervals stored.
  - In the best case (all numbers form one continuous range), space is `O(1)`.
  - In the worst case (all numbers are non-consecutive), space is `O(k)` where `k` is the number of unique values added.
  - The SortedDict stores at most one interval per disjoint range of consecutive numbers.

## Common Pitfalls

### 1. **Incorrect Interval Merging Logic**

**Pitfall:** One of the most common mistakes is incorrectly handling the case where a new number should merge two existing intervals. Developers often forget to check both conditions properly or make off-by-one errors.

**Example of incorrect code:**
```python
# Wrong: Checking if val equals the boundaries instead of being adjacent
if (left_interval[1] == val and right_interval[0] == val):
    # This would only merge if val is exactly on both boundaries
    # Missing the case where val fills a gap of 1
```

**Solution:** Ensure you check if the value bridges the gap between two intervals:
```python
if (interval_values[left_idx][1] + 1 == val and 
    interval_values[right_idx][0] - 1 == val):
    # Correctly checks if val fills a gap of exactly 1
```

### 2. **Not Handling Already-Contained Values**

**Pitfall:** When a value falls within an existing interval, some implementations incorrectly try to modify the interval or create a duplicate.

**Example of incorrect code:**
```python
# Wrong: Always extending the interval even if val is already inside
if val <= interval_values[left_idx][1] + 1:
    self.intervals_map[interval_keys[left_idx]][1] = val  # Wrong!
```

**Solution:** Use `max()` to ensure the interval doesn't shrink:
```python
self.intervals_map[interval_keys[left_idx]][1] = max(val, self.intervals_map[interval_keys[left_idx]][1])
```

### 3. **Key Modification in SortedDict**

**Pitfall:** Attempting to directly modify a key in SortedDict breaks the sorted order. When extending an interval's start point, you cannot simply change the key.

**Example of incorrect approach:**
```python
# Wrong: Trying to change the key directly
if val < interval_start:
    del self.intervals_map[interval_start]
    self.intervals_map[val] = [val, interval_end]  # Inefficient and error-prone
```

**Solution:** The current implementation cleverly avoids this by only modifying the value's start component when extending leftward:
```python
# Correct: Only modify the value, not the key
self.intervals_map[interval_keys[right_idx]][0] = min(val, self.intervals_map[interval_keys[right_idx]][0])
```

### 4. **Binary Search Index Confusion**

**Pitfall:** Misunderstanding what `bisect_right` returns and incorrectly calculating the left interval index.

**Example of incorrect code:**
```python
# Wrong: Using bisect_left or incorrectly calculating left_idx
left_idx = self.intervals_map.bisect_left(val) - 1  # Can be -1!
if left_idx >= 0:  # Need to check bounds
    # process...
```

**Solution:** Use consistent index handling with a sentinel value:
```python
right_idx = self.intervals_map.bisect_right(val)
left_idx = num_intervals if right_idx == 0 else right_idx - 1
# Now left_idx == num_intervals serves as "no left interval" indicator
```

### 5. **Order of Condition Checking**

**Pitfall:** Checking conditions in the wrong order can lead to incorrect behavior. For example, checking single interval extensions before checking the merge case.

**Example of problematic ordering:**
```python
# Wrong order: Extending first might prevent proper merging
if extends_left_interval:
    extend_left()
elif extends_right_interval:
    extend_right()
elif merges_two_intervals:  # This might never be reached!
    merge()
```

**Solution:** Always check the merge case first, as it's the most specific condition:
```python
if merges_two_intervals:
    merge()
elif extends_left_interval:
    extend_left()
elif extends_right_interval:
    extend_right()
else:
    create_new_interval()
```

### 6. **Edge Case: Single-Point Intervals**

**Pitfall:** Not properly handling single-point intervals where start equals end.

**Solution:** The implementation correctly handles this by initializing new intervals as `[val, val]` and using inclusive boundaries throughout the logic.

