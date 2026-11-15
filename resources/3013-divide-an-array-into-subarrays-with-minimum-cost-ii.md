# 3013. Divide an Array Into Subarrays With Minimum Cost II

> Source: https://algo.monster/liteproblems/3013

## Problem Statement

You are given a 0-indexed integer array `nums` of length `n`, and two positive integers `k` and `dist`.

The **cost** of an array is defined as the value of its **first** element. For example, the cost of `[1,2,3]` is `1`, while the cost of `[3,4,1]` is `3`.

Your task is to divide `nums` into exactly `k` **disjoint contiguous subarrays** with a specific constraint on their positions. If you divide `nums` into subarrays starting at indices `0`, `i₁`, `i₂`, ..., `i_{k-1}`, then the constraint is:
- The difference between the starting index of the **second** subarray (`i₁`) and the starting index of the **k-th** subarray (`i_{k-1}`) must be **at most** `dist`
- In other words: `i_{k-1} - i₁ ≤ dist`

The subarrays would look like:
- First subarray: `nums[0..(i₁ - 1)]`
- Second subarray: `nums[i₁..(i₂ - 1)]`
- ...
- k-th subarray: `nums[i_{k-1}..(n - 1)]`

The total cost is the sum of the costs of all `k` subarrays (sum of the first element of each subarray).

Return the **minimum** possible sum of the costs of these `k` subarrays.

**Example interpretation:** If you have `nums = [1,3,2,6,4,2]`, `k = 3`, and `dist = 3`, you need to split the array into 3 parts where the starting positions of the 2nd and 3rd parts are at most 3 indices apart. One possible split could be `[1]`, `[3,2]`, `[6,4,2]` with costs 1, 3, and 6 respectively, giving a total cost of 10.

## Intuition

Let's think about what we're actually choosing when we divide the array into `k` subarrays. We need to pick `k-1` cutting points (since the first subarray always starts at index 0). These cutting points become the starting indices of our subarrays, and the first element at each starting index contributes to our total cost.

The key insight is that `nums[0]` is always included in the cost (it's the first element of the first subarray). So we need to choose `k-1` more starting positions for the remaining subarrays.

The constraint `i_{k-1} - i_1 ≤ dist` tells us that all our `k-1` cutting points must fit within a window of size `dist + 1`. Why? Because if the difference between the last and first cutting point is at most `dist`, then all cutting points must be within `dist + 1` consecutive positions.

This transforms our problem into: **find a [sliding window](/problems/sliding_window_maximum) of size `dist + 1` starting from index 1, and within each window, select the `k-1` smallest elements** (since we want to minimize the sum of costs).

Think of it this way:
- We must include `nums[0]` in our answer (cost of first subarray)
- We need to choose `k-1` more indices as starting points for the remaining subarrays
- These `k-1` indices must all fall within some window of size `dist + 1`
- To minimize cost, we should pick the `k-1` positions with the smallest values in that window

So the problem becomes: slide a window of size `dist + 1` through the array (starting from index 1), and for each window position, find the sum of the `k-1` smallest elements. Add `nums[0]` to this sum, and track the minimum across all window positions.

This naturally leads us to use two ordered sets: one to maintain the `k-1` smallest elements (whose sum we need), and another to hold the remaining elements in the window. As we slide the window, we efficiently update these sets and recompute the sum.

**Learn more about [Sliding Window](/problems/sliding_window_maximum) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

We implement the [sliding window](/problems/sliding_window_maximum) approach using two ordered sets (SortedList in Python) to efficiently maintain the `k-1` smallest elements in our window.

**Initial Setup:**
- Decrease `k` by 1 since `nums[0]` is always included
- Initialize sum `s` with the first `dist + 2` elements (including `nums[0]`)
- Create two SortedLists:
  - `l`: stores the `k` smallest elements in the current window
  - `r`: stores the remaining elements in the window
- Initially add all elements from indices `[1, dist + 1]` to `l`

**Helper Functions:**
- `l2r()`: Moves the largest element from `l` to `r` and updates sum `s`
- `r2l()`: Moves the smallest element from `r` to `l` and updates sum `s`

**Initial Balance:**
- If `l` has more than `k` elements, repeatedly call `l2r()` until `|l| = k`
- Set initial answer `ans = s`

**[Sliding Window](/problems/sliding_window_maximum) Process:**
For each position `i` from `dist + 2` to `n-1`:

1. **Remove the element leaving the window:**
   - Element to remove: `x = nums[i - dist - 1]`
   - If `x` is in `l`, remove it and subtract from `s`
   - Otherwise, remove it from `r`

2. **Add the new element entering the window:**
   - New element: `y = nums[i]`
   - If `y < l[-1]` (smaller than the largest in `l`), add to `l` and update `s`
   - Otherwise, add to `r`

3. **Rebalance the sets:**
   - While `|l| < k`: call `r2l()` to move smallest from `r` to `l`
   - While `|l| > k`: call `l2r()` to move largest from `l` to `r`

4. **Update answer:**
   - `ans = min(ans, s)`

The algorithm maintains the invariant that `l` always contains exactly the `k` smallest elements in the current window, and `s` always equals `nums[0]` plus the sum of elements in `l`. This ensures we efficiently compute the minimum possible sum across all valid window positions.

**Time Complexity:** `O(n * log(dist))` where each insertion/deletion in the SortedList takes `O(log(dist))` time.

**Space Complexity:** `O(dist)` for storing elements in the two SortedLists.

## Example Walkthrough

Let's walk through a small example with `nums = [10, 8, 18, 9]`, `k = 3`, and `dist = 1`.

**Goal:** Split the array into 3 subarrays where the starting indices of the 2nd and 3rd subarrays are at most 1 position apart.

**Initial Setup:**
- First subarray always starts at index 0, so `nums[0] = 10` is always in our cost
- We need to choose 2 more starting positions (k-1 = 2)
- These 2 positions must be within a window of size `dist + 1 = 2`

**Possible Windows:**
Since we need positions for the 2nd and 3rd subarrays, our window can start at:
- Window 1: indices [1, 2] → elements [8, 18]
- Window 2: indices [2, 3] → elements [18, 9]

**Window 1 Analysis:**
- Elements in window: [8, 18]
- Choose the 2 smallest: 8 and 18
- Subarrays would be: `[10]`, `[8]`, `[18, 9]`
- Total cost: 10 + 8 + 18 = 36

**Window 2 Analysis:**
- Elements in window: [18, 9]
- Choose the 2 smallest: 9 and 18
- Subarrays would be: `[10, 8]`, `[18]`, `[9]`
- Total cost: 10 + 18 + 9 = 37

**Algorithm Execution:**

1. **Initialize:** 
   - `k = 2` (decreased by 1)
   - `s = 10` (starting with nums[0])
   - Window starts with indices [1, 2]: add 8 and 18 to set `l`
   - `l = [8, 18]`, `r = []`
   - `s = 10 + 8 + 18 = 36`
   - `ans = 36`

2. **Slide to next window (i = 3):**
   - Remove `nums[1] = 8` from window (it's in `l`)
     - `l = [18]`, `s = 10 + 18 = 28`
   - Add `nums[3] = 9` to window
     - Since 9 < 18, add to `l`: `l = [9, 18]`, `s = 28 + 9 = 37`
   - Balance: `|l| = 2 = k`, no rebalancing needed
   - Update: `ans = min(36, 37) = 36`

**Result:** The minimum cost is 36, achieved by choosing indices 1 and 2 as starting positions, creating subarrays `[10]`, `[8]`, `[18, 9]`.

## Python Solution

```python
class Solution:
    def minimumCost(self, nums: List[int], k: int, dist: int) -> int:
        """
        Find the minimum cost by selecting k elements where the first element is always included,
        and subsequent elements must be within 'dist' distance from each other.
      
        Args:
            nums: List of numbers to select from
            k: Number of elements to select
            dist: Maximum distance constraint between consecutive selected elements
      
        Returns:
            Minimum sum of k selected elements
        """
        from sortedcontainers import SortedList
      
        # Helper function to move the largest element from left set to right set
        def move_from_left_to_right():
            nonlocal current_sum
            element = left_set.pop()  # Remove largest element from left set
            current_sum -= element
            right_set.add(element)
      
        # Helper function to move the smallest element from right set to left set
        def move_from_right_to_left():
            nonlocal current_sum
            element = right_set.pop(0)  # Remove smallest element from right set
            left_set.add(element)
            current_sum += element
      
        # Adjust k since the first element is always included
        k -= 1
      
        # Initialize with the first window (elements from index 1 to dist+1)
        # nums[0] is always included, so we start the window from index 1
        current_sum = sum(nums[:dist + 2])  # Sum including nums[0]
        left_set = SortedList(nums[1:dist + 2])  # Window elements (excluding nums[0])
        right_set = SortedList()  # Elements outside the k smallest in current window
      
        # Ensure left_set contains exactly k elements (the k smallest in window)
        while len(left_set) > k:
            move_from_left_to_right()
      
        # Initialize answer with the first window's sum
        min_cost = current_sum
      
        # Slide the window through the rest of the array
        for i in range(dist + 2, len(nums)):
            # Remove the element that's going out of the window
            outgoing_element = nums[i - dist - 1]
            if outgoing_element in left_set:
                left_set.remove(outgoing_element)
                current_sum -= outgoing_element
            else:
                right_set.remove(outgoing_element)
          
            # Add the new element entering the window
            incoming_element = nums[i]
            if left_set and incoming_element < left_set[-1]:
                # If new element is smaller than largest in left_set, add to left_set
                left_set.add(incoming_element)
                current_sum += incoming_element
            else:
                # Otherwise, add to right_set
                right_set.add(incoming_element)
          
            # Rebalance to ensure left_set has exactly k elements
            while len(left_set) < k:
                move_from_right_to_left()
            while len(left_set) > k:
                move_from_left_to_right()
          
            # Update the minimum cost
            min_cost = min(min_cost, current_sum)
      
        return min_cost
```

## Time and Space Complexity

**Time Complexity:** `O(n × log(dist))`

The algorithm uses a sliding window approach with two SortedLists to maintain the k smallest elements within a window of size `dist + 1`. 

- Initial setup takes `O(dist × log(dist))` to build the SortedList and organize elements between `l` and `r`
- The main loop runs `n - dist - 2` times (approximately `O(n)` iterations)
- In each iteration:
  - Removing element `x` from either `l` or `r`: `O(log(dist))` 
  - Adding element `y` to either `l` or `r`: `O(log(dist))`
  - The while loops for rebalancing (`r2l` and `l2r`) may execute, but each element moves between lists at most once per iteration, with each move taking `O(log(dist))`
- Since the window size is bounded by `dist + 1`, the SortedLists contain at most `O(dist)` elements

Overall time complexity: `O(n × log(dist))`

**Space Complexity:** `O(dist)`

The space is used for:
- SortedList `l`: contains at most `k` elements where `k ≤ dist + 1`
- SortedList `r`: contains remaining elements from the window, at most `dist + 1 - k` elements
- Together, `l` and `r` store at most `dist + 1` elements at any time

Therefore, the space complexity is `O(dist)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Window Boundaries**
One of the most common mistakes is misunderstanding the window boundaries. The window should contain elements that can be chosen as starting positions for subarrays 2 through k.

**Pitfall:** Thinking the window should be of size `dist` instead of `dist + 1`.

**Why it happens:** The constraint `i_{k-1} - i_1 ≤ dist` means if `i_1` is at position `p`, then `i_{k-1}` can be at most at position `p + dist`. This gives us `dist + 1` possible positions, not `dist`.

**Solution:** The window for possible starting positions should span indices `[i, i + dist]`, which contains `dist + 1` elements.

### 2. **Off-by-One Error in Initial Window Setup**
**Pitfall:** Including `nums[0]` in the sliding window data structures.

**Why it happens:** Since `nums[0]` is always the first element and its cost is always included, it shouldn't be part of the sliding window that tracks the k-1 smallest elements for positions 2 through k.

**Solution:** Start the window from index 1, not index 0. Initialize with `nums[1:dist + 2]` for the window elements.

### 3. **Forgetting to Adjust k**
**Pitfall:** Using the original `k` value throughout the algorithm instead of `k-1`.

**Why it happens:** Since `nums[0]` is always selected as the first subarray's starting position, we only need to find `k-1` more positions from the remaining array.

**Solution:** Immediately decrease `k` by 1 at the beginning: `k -= 1`

### 4. **Incorrect Element Removal Logic**
**Pitfall:** Not checking which set (left or right) contains the element being removed from the window.

```python
# Wrong approach:
left_set.remove(outgoing_element)  # Assumes it's always in left_set
current_sum -= outgoing_element

# Correct approach:
if outgoing_element in left_set:
    left_set.remove(outgoing_element)
    current_sum -= outgoing_element
else:
    right_set.remove(outgoing_element)
```

### 5. **Improper Sum Maintenance**
**Pitfall:** Forgetting to update `current_sum` when elements move between sets or are added/removed.

**Why it happens:** The sum should always represent `nums[0]` plus the sum of all elements in `left_set`. Any operation that changes `left_set` must update the sum accordingly.

**Solution:** Ensure every operation on `left_set` has a corresponding sum update:
- Adding to `left_set`: add to sum
- Removing from `left_set`: subtract from sum
- Moving from `left_set` to `right_set`: subtract from sum
- Moving from `right_set` to `left_set`: add to sum

### 6. **Incorrect Rebalancing Logic**
**Pitfall:** Not maintaining exactly `k` elements in `left_set` after each window slide.

**Solution:** After adding/removing elements, always rebalance:
```python
while len(left_set) < k and right_set:
    move_from_right_to_left()
while len(left_set) > k:
    move_from_left_to_right()
```

### 7. **Edge Case: When dist + 1 ≥ n - 1**
**Pitfall:** Not handling the case where the window size exceeds the available elements.

**Why it happens:** If `dist + 1 ≥ n - 1`, we have fewer than `dist + 1` elements to choose from after index 0.

**Solution:** The algorithm naturally handles this by iterating only up to `len(nums)`, but be aware that in such cases, we might not need the sliding window approach at all - we could simply select the k smallest elements from `nums[1:]` and add `nums[0]`.

