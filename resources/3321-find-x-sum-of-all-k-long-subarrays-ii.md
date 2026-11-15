# 3321. Find X-Sum of All K-Long Subarrays II

> Source: https://algo.monster/liteproblems/3321

## Problem Statement

You are given an array `nums` of `n` integers and two integers `k` and `x`.

The **x-sum** of an array is calculated using the following procedure:
1. Count the occurrences of all elements in the array
2. Keep only the occurrences of the top `x` most frequent elements
   - If two elements have the same number of occurrences, the element with the **bigger** value is considered more frequent
3. Calculate the sum of the resulting array

**Note:** If an array has less than `x` distinct elements, its x-sum is simply the sum of the entire array.

Your task is to return an integer array `answer` of length `n - k + 1` where `answer[i]` is the x-sum of the subarray `nums[i..i + k - 1]`.

For example, if we have a subarray and need to find its x-sum with `x = 2`:
- First, count how many times each element appears
- Select the top 2 most frequent elements (using value as tiebreaker if frequencies are equal)
- Sum up all occurrences of these top 2 elements

The problem asks you to compute this x-sum for every [sliding window](/problems/sliding_window_maximum) of size `k` in the array, moving from left to right one position at a time.

## Intuition

The key insight is that we need to efficiently maintain the top `x` most frequent elements in a [sliding window](/problems/sliding_window_maximum) of size `k`. As the window slides, elements enter and leave, which changes their frequencies and potentially affects which elements belong in the top `x`.

Think of this problem as maintaining two groups of elements:
- The "elite" group: the top `x` most frequent elements whose occurrences contribute to the x-sum
- The "waiting" group: all other elements that might get promoted to the elite group

When we slide the window:
1. A new element enters on the right - its frequency increases
2. An old element leaves on the left - its frequency decreases

These frequency changes might cause elements to move between the elite and waiting groups. An element with increased frequency might deserve promotion to the elite group, while an element with decreased frequency might need demotion.

To handle the tie-breaking rule (larger values are preferred when frequencies are equal), we can use tuples `(frequency, value)` as our comparison key. This naturally ensures that when frequencies are equal, the larger value wins.

The challenge is efficiently maintaining these two groups and quickly calculating the x-sum. Using ordered data structures (like sorted lists) allows us to:
- Always know the weakest member of the elite group (smallest frequency/value pair)
- Always know the strongest member of the waiting group (largest frequency/value pair)
- Efficiently transfer elements between groups when needed

By maintaining a running sum `s` of the elite group, we avoid recalculating the entire x-sum from scratch for each window. We only update `s` when elements move in or out of the elite group, making each window's calculation very efficient.

The pattern of "remove old occurrence count, update frequency, add new occurrence count" ensures we properly handle the frequency changes without double-counting or missing elements during transitions.

**Learn more about [Sliding Window](/problems/sliding_window_maximum) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

The solution uses a hash table combined with two ordered sets to efficiently track and update the x-sum as we slide the window across the array.

**Data Structures:**
- `cnt`: A Counter (hash table) to track the frequency of each element in the current window
- `l`: A SortedList storing the top `x` most frequent elements as tuples `(frequency, value)`
- `r`: A SortedList storing the remaining elements not in the top `x`
- `s`: Running sum of all elements in `l` (the x-sum)

**Helper Functions:**

The `remove(v)` function handles removing an element's old state before updating its frequency:
- Creates a tuple `p = (cnt[v], v)` representing the element's current state
- If `p` is in the elite set `l`, removes it and subtracts its contribution from `s`
- Otherwise, removes it from the waiting set `r`

The `add(v)` function handles adding an element's new state after updating its frequency:
- Creates a tuple `p = (cnt[v], v)` representing the element's new state
- If `l` is not empty and `p` is greater than the smallest element in `l`, it belongs in the elite group
  - Adds `p` to `l` and updates the sum `s`
- Otherwise, adds `p` to the waiting set `r`

**Main Algorithm:**

1. **Initialize the first window**: Add the first `k` elements to build the initial window
   - For each element, remove its old state, increment its count, then add its new state

2. **Balance the sets**: After the window is built, ensure `l` contains exactly `x` elements (or all distinct elements if less than `x`)
   - While `l` has fewer than `x` elements and `r` is not empty:
     - Move the largest element from `r` to `l` and update `s`
   - While `l` has more than `x` elements:
     - Move the smallest element from `l` to `r` and update `s`

3. **Record the x-sum**: The current value of `s` is the x-sum for this window position

4. **Slide the window**: Remove the leftmost element and continue to the next position
   - Remove the old state of the departing element
   - Decrement its count in `cnt`
   - Add its new state (which might be with count 0)

The process repeats for each window position, giving us an x-sum for each of the `n - k + 1` possible windows.

**Time Complexity**: `O(n × log k)` - Each element is added and removed once, with each operation taking `O(log k)` time due to the sorted lists.

**Space Complexity**: `O(n)` - For storing the counter and sorted lists.

## Example Walkthrough

Let's walk through a concrete example with `nums = [1, 1, 2, 2, 3, 4, 2, 3]`, `k = 6`, and `x = 2`.

**Initial Setup:**
- We need to find the x-sum for each window of size 6
- The x-sum includes only the top 2 most frequent elements

**Window 1: [1, 1, 2, 2, 3, 4]**

Building the window element by element:
- Add 1: `cnt = {1:1}`, move (1,1) to `l`, `s = 1`
- Add 1: `cnt = {1:2}`, update to (2,1) in `l`, `s = 2`
- Add 2: `cnt = {1:2, 2:1}`, add (1,2) to `l`, `s = 3`
- Add 2: `cnt = {1:2, 2:2}`, update to (2,2) in `l`, `s = 5`
- Add 3: `cnt = {1:2, 2:2, 3:1}`, add (1,3) to `r` (since we already have 2 elements in `l`)
- Add 4: `cnt = {1:2, 2:2, 3:1, 4:1}`, add (1,4) to `r`

Current state:
- `l = [(2,1), (2,2)]` - top 2 frequent elements (both appear twice)
- `r = [(1,3), (1,4)]` - remaining elements
- `s = 2×1 + 2×2 = 6` (x-sum for this window)

**Sliding to Window 2: [1, 2, 2, 3, 4, 2]**

Remove leftmost element (1):
- Remove (2,1) from `l`, `s = 4`
- Update `cnt = {1:1, 2:2, 3:1, 4:1}`
- Add (1,1) back, but now it goes to `r` since (1,1) < (2,2)

Add rightmost element (2):
- Remove (2,2) from `l`, `s = 0`
- Update `cnt = {1:1, 2:3, 3:1, 4:1}`
- Add (3,2) to `l`, `s = 6`

Balance sets (move highest from `r` to `l` until we have 2 elements):
- Move (1,4) from `r` to `l`, `s = 7`

Current state:
- `l = [(1,4), (3,2)]` - element 2 appears 3 times, element 4 appears once
- `r = [(1,1), (1,3)]`
- `s = 1×4 + 3×2 = 10` (x-sum for this window)

**Sliding to Window 3: [2, 2, 3, 4, 2, 3]**

Remove leftmost element (1):
- Remove (1,1) from `r`
- Update `cnt = {2:3, 3:1, 4:1}` (element 1 is removed entirely)
- No need to add anything back since count is 0

Add rightmost element (3):
- Remove (1,3) from `r`
- Update `cnt = {2:3, 3:2, 4:1}`
- Add (2,3) to `l`, `s = 12`

Balance sets (we now have 3 elements in `l`, need to demote one):
- Move (1,4) from `l` to `r`, `s = 11`

Current state:
- `l = [(2,3), (3,2)]` - element 3 appears 2 times, element 2 appears 3 times
- `r = [(1,4)]`
- `s = 2×3 + 3×2 = 12` (x-sum for this window)

**Final Result:** `answer = [6, 10, 12]`

This walkthrough demonstrates how:
1. The sorted lists maintain the top x elements dynamically
2. The sum `s` is updated incrementally as elements move between sets
3. The tie-breaking rule favors larger values when frequencies are equal
4. Elements smoothly transition between the elite and waiting groups as their frequencies change

## Python Solution

```python
class Solution:
    def findXSum(self, nums: List[int], k: int, x: int) -> List[int]:
        """
        Find the sum of top x frequent elements for each sliding window of size k.
        Elements are ranked by (frequency, value) in descending order.
      
        Args:
            nums: Input array
            k: Window size
            x: Number of top frequent elements to sum
      
        Returns:
            List of sums for each window
        """
      
        def add_to_sets(value: int) -> None:
            """Add element to either top_x_set or remaining_set based on its priority."""
            if frequency_map[value] == 0:
                return
          
            priority_tuple = (frequency_map[value], value)
          
            # If top_x_set is empty or this element has higher priority than the minimum in top_x_set
            if top_x_set and priority_tuple > top_x_set[0]:
                nonlocal current_sum
                current_sum += priority_tuple[0] * priority_tuple[1]  # frequency * value
                top_x_set.add(priority_tuple)
            else:
                remaining_set.add(priority_tuple)
      
        def remove_from_sets(value: int) -> None:
            """Remove element from whichever set it belongs to."""
            if frequency_map[value] == 0:
                return
          
            priority_tuple = (frequency_map[value], value)
          
            if priority_tuple in top_x_set:
                nonlocal current_sum
                current_sum -= priority_tuple[0] * priority_tuple[1]  # frequency * value
                top_x_set.remove(priority_tuple)
            else:
                remaining_set.remove(priority_tuple)
      
        # Initialize data structures
        top_x_set = SortedList()      # Maintains top x elements by (frequency, value)
        remaining_set = SortedList()   # Maintains remaining elements
        frequency_map = Counter()       # Tracks frequency of each element in current window
        current_sum = 0                 # Sum of top x elements
        n = len(nums)
        result = [0] * (n - k + 1)     # Result array for each window
      
        # Process each element
        for i, current_value in enumerate(nums):
            # Update frequency for current element
            remove_from_sets(current_value)
            frequency_map[current_value] += 1
            add_to_sets(current_value)
          
            # Calculate window start index
            window_start = i - k + 1
          
            # Skip if we haven't formed a complete window yet
            if window_start < 0:
                continue
          
            # Balance the sets: ensure top_x_set has exactly x elements
            # Move elements from remaining_set to top_x_set if needed
            while remaining_set and len(top_x_set) < x:
                element = remaining_set.pop()
                top_x_set.add(element)
                current_sum += element[0] * element[1]
          
            # Move excess elements from top_x_set to remaining_set
            while len(top_x_set) > x:
                element = top_x_set.pop(0)  # Remove smallest element
                current_sum -= element[0] * element[1]
                remaining_set.add(element)
          
            # Store result for current window
            result[window_start] = current_sum
          
            # Remove the leftmost element of the window for next iteration
            leftmost_element = nums[window_start]
            remove_from_sets(leftmost_element)
            frequency_map[leftmost_element] -= 1
            if frequency_map[leftmost_element] > 0:
                add_to_sets(leftmost_element)
      
        return result
```

## Time and Space Complexity

**Time Complexity:** `O(n * k * log(k))`

The algorithm uses a sliding window approach with two SortedLists to maintain the top `x` elements by frequency and value. Breaking down the operations:

- The outer loop runs `n` times (once for each element in `nums`)
- For each iteration:
  - `remove(v)` operation: Removing from a SortedList takes `O(log(k))` time in the worst case, as the SortedList can contain at most `k` distinct elements
  - `add(v)` operation: Adding to a SortedList takes `O(log(k))` time
  - The while loops for balancing between `l` and `r` SortedLists:
    - Each element can move between `l` and `r` at most once per window
    - Pop and add operations each take `O(log(k))` time
  - For the window at position `j`:
    - Remove operation for `nums[j]`: `O(log(k))`
    - Add operation for `nums[j]`: `O(log(k))`

Since we perform a constant number of `O(log(k))` operations per element, and we process `n` elements total, with potential rebalancing operations that could involve up to `k` elements in the worst case, the overall time complexity is `O(n * k * log(k))`.

**Space Complexity:** `O(k)`

The space usage consists of:
- `l` and `r` SortedLists: Combined they store at most `k` distinct elements (one entry per distinct value in the current window), so `O(k)` space
- `cnt` Counter: Stores at most `k` distinct elements from the current window, so `O(k)` space
- `ans` array: Stores `n - k + 1` results, which is `O(n)` space
- Other variables (`s`, `i`, `j`, etc.): `O(1)` space

The dominant space complexity excluding the output array is `O(k)`. Including the output array, the total space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Handling of Element Priority When Frequencies Are Equal**

A critical pitfall is misunderstanding how elements should be prioritized when they have the same frequency. The problem states that when frequencies are equal, the element with the **bigger value** should be considered more frequent. This means the comparison tuple should be `(frequency, value)` in that order, which naturally gives us the correct sorting behavior.

**Incorrect approach:**
```python
# Wrong: Using (value, frequency) would prioritize value over frequency
priority_tuple = (value, frequency_map[value])
```

**Correct approach:**
```python
# Correct: Using (frequency, value) prioritizes frequency first, then value as tiebreaker
priority_tuple = (frequency_map[value], value)
```

### 2. **Not Handling Zero-Frequency Elements Properly**

When an element's frequency becomes 0 (after being removed from the window), attempting to add it to the sorted sets creates unnecessary overhead and potential bugs. The element with frequency 0 shouldn't contribute to any calculations.

**Problem:**
```python
def add_to_sets(value: int) -> None:
    # Missing check for zero frequency
    priority_tuple = (frequency_map[value], value)
    # This would add (0, value) to the sets unnecessarily
```

**Solution:**
```python
def add_to_sets(value: int) -> None:
    if frequency_map[value] == 0:
        return  # Skip elements with zero frequency
    priority_tuple = (frequency_map[value], value)
    # ... rest of the logic
```

### 3. **Incorrect Sum Calculation**

A common mistake is forgetting that the sum should include **all occurrences** of the top x elements, not just counting each unique element once.

**Incorrect:**
```python
# Wrong: Only adds the value once
current_sum += priority_tuple[1]  # Just adds the value
```

**Correct:**
```python
# Correct: Multiplies value by its frequency
current_sum += priority_tuple[0] * priority_tuple[1]  # frequency * value
```

### 4. **Improper Set Balancing After Window Updates**

After adding or removing elements, the sets might become unbalanced. The top_x_set should always contain exactly x elements (or all distinct elements if fewer than x exist). Failing to rebalance after each window update leads to incorrect results.

**Issue:** Not moving elements between sets when needed:
```python
# After updating the window, forgetting to rebalance:
result[window_start] = current_sum  # Wrong if sets are unbalanced
```

**Solution:** Always rebalance before calculating the result:
```python
# Move elements from remaining_set to top_x_set if needed
while remaining_set and len(top_x_set) < x:
    element = remaining_set.pop()
    top_x_set.add(element)
    current_sum += element[0] * element[1]

# Move excess elements from top_x_set to remaining_set
while len(top_x_set) > x:
    element = top_x_set.pop(0)
    current_sum -= element[0] * element[1]
    remaining_set.add(element)

# Now calculate the result
result[window_start] = current_sum
```

### 5. **Edge Case: When Window Has Fewer Than x Distinct Elements**

The problem states that if a window has fewer than x distinct elements, the x-sum is the sum of the entire array. The balancing logic handles this automatically by keeping all elements in top_x_set when there are fewer than x distinct elements, but it's important to understand this behavior.

**Key insight:** The while loop condition `while remaining_set and len(top_x_set) < x` ensures we only move elements if they exist in remaining_set, naturally handling the case where we have fewer than x distinct elements.

