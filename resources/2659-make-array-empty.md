# 2659. Make Array Empty

> Source: https://algo.monster/liteproblems/2659

## Problem Statement

You have an integer array `nums` containing distinct numbers. You need to empty this array by repeatedly performing one of two operations:

1. If the first element is the smallest value in the entire array, remove it from the array
2. Otherwise, move the first element to the end of the array

Your task is to count how many total operations it takes to completely empty the array.

For example, if you have `nums = [3, 1, 2]`:
- First element is 3, which is not the smallest (1 is), so move 3 to the end: `[1, 2, 3]` (1 operation)
- First element is 1, which is the smallest, so remove it: `[2, 3]` (1 operation)
- First element is 2, which is the smallest, so remove it: `[3]` (1 operation)
- First element is 3, which is the smallest (and only), so remove it: `[]` (1 operation)
- Total: 4 operations

The challenge is to efficiently calculate this count without actually simulating the entire process, since the array could be large. The solution uses the fact that elements will be removed in sorted order (smallest to largest), and tracks how many rotations occur between removing consecutive elements.

## Intuition

Since elements can only be removed when they're the smallest and at the front, we'll always remove elements in ascending order. This means if we have `[3, 1, 4, 2]`, we'll remove them in the order: 1, 2, 3, 4, regardless of their initial positions.

The key insight is that we don't need to simulate the actual rotations. Instead, we can calculate how many operations it takes to bring each element to the front position when it's its turn to be removed.

Consider the sorted sequence of elements to be removed. For the first (smallest) element, we need to rotate the array until it reaches the front. That's simply its initial position + 1 operations (the positions ahead of it plus the removal itself).

For subsequent elements, the calculation becomes more complex. When moving from removing element `a` to element `b`, we need to consider:
1. The distance between their original positions
2. Which elements between them have already been removed (they no longer contribute to the rotation count)
3. Whether we need to "wrap around" the array (if `b`'s position comes before `a`'s position in the original array)

The "wrap around" case is crucial: if element `b` was originally positioned before element `a`, it means after removing `a`, we continue rotating past the end of the array and back to the beginning to reach `b`. This adds `n - k` extra operations, where `k` is the number of elements already removed.

To efficiently track which elements have been removed between any two positions, we use a data structure like a Fenwick tree or sorted list. This allows us to query "how many elements between positions `i` and `j` have been removed?" in `O(log n)` time.

The formula for operations between consecutive removals becomes:
- Basic distance: `j - i`
- Subtract already removed elements between them: `- (removed elements in range)`
- Add wrap-around cost if needed: `+ (n - k)` if `i > j`

**Learn more about [Greedy](/problems/greedy_intro), [Segment Tree](/problems/segment_tree_intro), [Binary Search](/problems/binary_search_intro) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

First, we create a hash table `pos` to map each element to its original index in the array: `pos = {x: i for i, x in enumerate(nums)}`. This allows us to quickly look up where each element was initially positioned.

Next, we sort the `nums` array. Since elements must be removed in ascending order (smallest first), this sorted array represents the order in which elements will be removed.

We initialize the answer with the operations needed to remove the first (smallest) element: `ans = pos[nums[0]] + 1`. This represents rotating the array until the smallest element reaches the front, then removing it.

To track which positions have been processed (removed), we use a `SortedList` data structure. This allows us to efficiently:
- Add removed positions with `sl.add(i)`
- Query how many positions below a certain value have been removed using `sl.bisect(j)` and `sl.bisect(i)`

For each consecutive pair of elements `(a, b)` in the sorted array:

1. Get their original positions: `i = pos[a]` and `j = pos[b]`

2. Calculate the distance between positions, adjusting for already-removed elements:
   - `d = j - i - sl.bisect(j) + sl.bisect(i)`
   - This computes the raw distance `j - i`, then subtracts the count of removed elements between them

3. Add wrap-around cost if necessary:
   - If `i > j` (meaning `b` was originally before `a`), we need to wrap around the array
   - This adds `(n - k) * int(i > j)` operations, where `k` is the current iteration index (number of elements already removed minus 1)

4. Update the total: `ans += d + (n - k) * int(i > j)`

5. Mark position `i` as removed: `sl.add(i)`

The time complexity is `O(n log n)` due to [sorting](/problems/sorting_summary) and the log-time operations on the SortedList. The space complexity is `O(n)` for the hash table and SortedList.

## Example Walkthrough

Let's trace through the solution with `nums = [4, 2, 1, 3]`.

**Step 1: Create position mapping**
- `pos = {4: 0, 2: 1, 1: 2, 3: 3}`
- This tells us: 4 is at index 0, 2 is at index 1, 1 is at index 2, 3 is at index 3

**Step 2: Sort the array**
- Sorted `nums = [1, 2, 3, 4]`
- This is the order we'll remove elements (smallest to largest)

**Step 3: Remove first element (1)**
- Element 1 is at position 2 (from `pos[1] = 2`)
- Operations needed: `2 + 1 = 3` (rotate twice to bring to front, then remove)
- `ans = 3`
- Add position 2 to removed set: `sl = [2]`

**Step 4: Remove second element (2)**
- Previous element 1 was at position `i = 2`
- Current element 2 is at position `j = 1`
- Calculate distance: `d = 1 - 2 - bisect(1) + bisect(2)`
  - `bisect(1)` in `[2]` = 0 (no removed positions less than 1)
  - `bisect(2)` in `[2]` = 0 (no removed positions less than 2)
  - `d = 1 - 2 - 0 + 0 = -1`
- Since `i > j` (2 > 1), we need wrap-around: add `n - k = 4 - 1 = 3`
- Total operations: `-1 + 3 = 2`
- `ans = 3 + 2 = 5`
- Add position 1 to removed set: `sl = [1, 2]`

**Step 5: Remove third element (3)**
- Previous element 2 was at position `i = 1`
- Current element 3 is at position `j = 3`
- Calculate distance: `d = 3 - 1 - bisect(3) + bisect(1)`
  - `bisect(3)` in `[1, 2]` = 2 (two removed positions less than 3)
  - `bisect(1)` in `[1, 2]` = 0 (no removed positions less than 1)
  - `d = 3 - 1 - 2 + 0 = 0`
- Since `i < j` (1 < 3), no wrap-around needed
- Total operations: `0 + 1 = 1` (the +1 for removal)
- `ans = 5 + 1 = 6`
- Add position 3 to removed set: `sl = [1, 2, 3]`

**Step 6: Remove fourth element (4)**
- Previous element 3 was at position `i = 3`
- Current element 4 is at position `j = 0`
- Calculate distance: `d = 0 - 3 - bisect(0) + bisect(3)`
  - `bisect(0)` in `[1, 2, 3]` = 0 (no removed positions less than 0)
  - `bisect(3)` in `[1, 2, 3]` = 2 (two removed positions less than 3)
  - `d = 0 - 3 - 0 + 2 = -1`
- Since `i > j` (3 > 0), we need wrap-around: add `n - k = 4 - 3 = 1`
- Total operations: `-1 + 1 + 1 = 1` (the last +1 for removal)
- `ans = 6 + 1 = 7`

**Final answer: 7 operations**

The algorithm efficiently calculates that we need 7 total operations to empty the array `[4, 2, 1, 3]` without actually simulating the rotations.

## Python Solution

```python
class Solution:
    def countOperationsToEmptyArray(self, nums: List[int]) -> int:
        # Create a mapping from each value to its original position in the array
        value_to_original_position = {value: index for index, value in enumerate(nums)}

        # Sort the values to determine the removal order (smallest to largest)
        sorted_values = sorted(nums)

        # SortedList to track which positions have been removed
        removed_positions = SortedList()

        # Initialize answer with operations to remove the first (smallest) element
        # We need original_position + 1 operations to reach and remove it
        total_operations = value_to_original_position[sorted_values[0]] + 1

        # Total number of elements
        array_length = len(nums)

        # Process each consecutive pair of values in sorted order
        for removal_index, (current_value, next_value) in enumerate(pairwise(sorted_values)):
            # Get original positions of current and next values
            current_position = value_to_original_position[current_value]
            next_position = value_to_original_position[next_value]

            # Calculate how many unremoved elements are between current and next positions
            # This accounts for already removed elements that don't affect the distance
            removed_between = removed_positions.bisect(next_position) - removed_positions.bisect(current_position)
            distance = next_position - current_position - removed_between

            # Add the distance to move from current to next position
            total_operations += distance

            # If next position is before current position (wraparound case)
            # We need to account for cycling through remaining elements
            if current_position > next_position:
                remaining_elements = array_length - removal_index
                total_operations += remaining_elements

            # Mark current position as removed
            removed_positions.add(current_position)

        return total_operations
```

## Time and Space Complexity

**Time Complexity:** `O(n × log n)`

The time complexity is dominated by the following operations:
- Creating the position dictionary `pos`: `O(n)` where we iterate through all elements once
- Sorting the `nums` array: `O(n × log n)` using Python's built-in sort
- The main loop iterates through `n-1` pairs of consecutive elements: `O(n)` iterations
- Inside each iteration:
  - Two `bisect` operations on the `SortedList`: each takes `O(log k)` where `k` is the current size of the sorted list (at most `n`)
  - One `add` operation to the `SortedList`: `O(log k)`
  - Other operations like dictionary lookups and arithmetic: `O(1)`

Since the loop runs `n` times and each iteration performs operations that cost `O(log n)` in the worst case, the total time complexity for the loop is `O(n × log n)`.

Therefore, the overall time complexity is `O(n) + O(n × log n) + O(n × log n) = O(n × log n)`.

**Space Complexity:** `O(n)`

The space complexity comes from:
- The `pos` dictionary storing position mappings for all `n` elements: `O(n)`
- The sorted copy of `nums` (in-place sort still requires `O(n)` for Timsort in worst case): `O(n)`
- The `SortedList` which can store up to `n-1` elements: `O(n)`
- Other variables like `ans`, `i`, `j`, `d`, etc.: `O(1)`

The total space complexity is `O(n) + O(n) + O(n) + O(1) = O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrectly Calculating Distance Between Elements**

A critical pitfall is misunderstanding how to calculate the actual distance between two elements after some positions have been removed. Many attempts incorrectly compute this as simply `j - i` without accounting for the gaps created by removed elements.

**Incorrect approach:**
```python
distance = next_position - current_position  # Wrong! Doesn't account for removed elements
```

**Correct approach:**
```python
removed_between = removed_positions.bisect(next_position) - removed_positions.bisect(current_position)
distance = next_position - current_position - removed_between
```

The key insight is that when elements are removed, they create "gaps" in the array that don't contribute to the movement count. We must subtract the number of already-removed positions between our current and target positions.

### 2. **Mishandling the Wrap-Around Case**

Another common mistake is incorrectly determining when and how to handle wrap-around scenarios. When the next element to remove is at a position earlier than the current one (in the original array), we need to wrap around to reach it.

**Incorrect approach:**
```python
if current_position > next_position:
    total_operations += array_length  # Wrong! Should use remaining elements
```

**Correct approach:**
```python
if current_position > next_position:
    remaining_elements = array_length - removal_index
    total_operations += remaining_elements
```

The wrap-around cost isn't the full array length but rather the number of elements still remaining in the array at that point. Since `removal_index` represents how many elements we've already removed minus one, the remaining count is `array_length - removal_index`.

### 3. **Off-by-One Error in Initial Operation Count**

A subtle mistake is forgetting to add 1 when counting the initial operations needed to remove the first element.

**Incorrect approach:**
```python
total_operations = value_to_original_position[sorted_values[0]]  # Missing the removal operation itself
```

**Correct approach:**
```python
total_operations = value_to_original_position[sorted_values[0]] + 1
```

The position gives us the number of rotations needed to bring the element to the front, but we also need one additional operation to actually remove it.

### 4. **Using Wrong Data Structure for Tracking Removed Positions**

Using a regular set or list instead of a SortedList leads to inefficient queries when determining how many elements have been removed in a range.

**Incorrect approach:**
```python
removed_positions = set()  # Can't efficiently count elements in a range
# Later...
removed_between = sum(1 for p in removed_positions if current_position < p < next_position)  # O(n) per query!
```

**Correct approach:**
```python
removed_positions = SortedList()  # Maintains sorted order
removed_between = removed_positions.bisect(next_position) - removed_positions.bisect(current_position)  # O(log n)
```

The SortedList allows binary search operations, making range queries logarithmic rather than linear in complexity.

