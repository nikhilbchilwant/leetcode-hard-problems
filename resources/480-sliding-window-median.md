# 480. Sliding Window Median

> Source: https://algo.monster/liteproblems/480

## Problem Statement

You are given an integer array `nums` and an integer `k`. You need to find the median of every contiguous subarray of size `k` using a sliding window approach.

The **median** is defined as:
- For an odd-sized list: the middle element when sorted
- For an even-sized list: the average of the two middle elements when sorted

For example:
- If `arr = [2, 3, 4]`, the median is `3` (the middle element)
- If `arr = [1, 2, 3, 4]`, the median is `(2 + 3) / 2 = 2.5` (average of two middle elements)

The sliding window works as follows:
- Start with a window of size `k` at the leftmost position of the array
- Calculate the median of the current window
- Move the window one position to the right (remove the leftmost element, add the next element from the right)
- Continue until the window reaches the rightmost position

Your task is to return an array containing the median of each window position as the window slides from left to right across the array. The answers will be accepted if they are within `10^-5` of the actual value.

For instance, if `nums = [1, 3, -1, -3, 5, 3, 6, 7]` and `k = 3`:
- Window `[1, 3, -1]` → median = `1`
- Window `[3, -1, -3]` → median = `-1`
- Window `[-1, -3, 5]` → median = `-1`
- And so on...

The result would be an array of all these medians.

## Intuition

Finding the median requires knowing the middle element(s) of a sorted sequence. If we were to sort the window every time it slides, we'd need `O(k log k)` time for each position, which is inefficient.

The key insight is that we don't need the entire sorted order - we only need to efficiently access the middle element(s). This naturally leads us to think about dividing our elements into two halves: the smaller half and the larger half.

If we maintain these two halves such that:
- The smaller half contains the smallest `⌈k/2⌉` elements
- The larger half contains the remaining elements
- We can quickly access the maximum of the smaller half and the minimum of the larger half

Then the median would be:
- For odd `k`: the maximum of the smaller half
- For even `k`: the average of the maximum of the smaller half and the minimum of the larger half

This structure points us toward using two heaps:
- A max-heap for the smaller half (to quickly get the largest element in this half)
- A min-heap for the larger half (to quickly get the smallest element in this half)

However, the sliding window introduces a challenge: we need to remove elements that fall out of the window. Removing arbitrary elements from a heap is expensive (`O(k)` time). 

Here's where lazy deletion comes in. Instead of immediately removing an element from the heap, we:
1. Mark it for deletion in a separate dictionary
2. Only actually remove it when it appears at the top of a heap
3. Keep track of the "logical" sizes of our heaps (actual size minus elements marked for deletion)

This way, elements that are buried in the heap and marked for deletion don't affect our median calculation (since they're not at the top), and we only pay the cost of removal when absolutely necessary.

The rebalancing ensures that our two halves maintain the correct size relationship, so the median can always be computed from the tops of the heaps.

## Solution Approach

The solution implements a `MedianFinder` class that maintains two heaps with lazy deletion to efficiently track the median of a sliding window.

### Data Structures Used

1. **Two Heaps**:
   - `small`: A max-heap (implemented using negative values in Python's min-heap) storing the smaller half of elements
   - `large`: A min-heap storing the larger half of elements

2. **Lazy Deletion Dictionary**:
   - `delayed`: A dictionary tracking elements to be removed and their counts
   - `small_size` and `large_size`: Logical sizes of the heaps (excluding delayed elements)

### Key Methods Implementation

**1. `add_num(num)`**:
- If `small` is empty or `num ≤ -small[0]` (max of smaller half), add to `small`
- Otherwise, add to `large`
- Increment the appropriate size counter
- Call `rebalance()` to maintain heap size invariant

**2. `remove_num(num)`**:
- Mark `num` for lazy deletion by incrementing `delayed[num]`
- Determine which heap logically contains `num` by comparing with `-small[0]`
- Decrement the appropriate size counter
- If `num` is at the top of its heap, call `prune()` to clean it up immediately

**3. `prune(pq)`**:
- While the top element of heap `pq` is in the `delayed` dictionary:
  - Decrement its count in `delayed`
  - If count reaches 0, remove from `delayed`
  - Pop the element from the heap
- This ensures the heap's top is always a valid (non-delayed) element

**4. `rebalance()`**:
- Maintains the invariant: `small_size` should be either equal to `large_size` or `large_size + 1`
- If `small_size > large_size + 1`: Move top of `small` to `large`
- If `small_size < large_size`: Move top of `large` to `small`
- After moving elements, prune the source heap to ensure its new top is valid

**5. `find_median()`**:
- If `k` is odd: Return `-small[0]` (the max of smaller half)
- If `k` is even: Return `(-small[0] + large[0]) / 2` (average of two middle elements)

### Main Algorithm Flow

1. Initialize `MedianFinder` with window size `k`
2. Add the first `k` elements to build the initial window
3. Calculate the first median
4. For each remaining element in `nums`:
   - Add the new element entering the window
   - Remove the element leaving the window (lazy deletion)
   - Calculate and store the new median

### Time Complexity Analysis

- **Adding an element**: `O(log k)` for heap insertion
- **Removing an element**: `O(1)` for marking deletion, `O(log k)` amortized for actual removal
- **Finding median**: `O(1)` as we just access heap tops
- **Overall**: `O(n log k)` where `n` is the array length

The lazy deletion optimization is crucial here - it prevents us from having to search through the heap to remove elements, which would be `O(k)` per removal.

## Example Walkthrough

Let's walk through the solution with `nums = [1, 3, -1, -3, 5, 3]` and `k = 3`.

Heap contents below are written as actual values. In the code, `small` stores negated values so that Python's min-heap behaves as a max-heap.

**Initial Window: [1, 3, -1]**

1. Start with empty heaps: `small = []`, `large = []`
2. Add `1`: Since `small` is empty, add to `small`.
   - `small = {1}`, `small_size = 1`
3. Add `3`: Compare with max of `small` (1). Since 3 > 1, add to `large`.
   - `large = {3}`, `large_size = 1`
   - Rebalance check: sizes 1 and 1 are valid
4. Add `-1`: Compare with max of `small` (1). Since -1 ≤ 1, add to `small`.
   - `small = {1, -1}`, `small_size = 2`
   - Rebalance check: `small_size (2)` is not greater than `large_size (1) + 1`, so nothing moves
5. Find median: k is odd, so median = max of `small` = **1**

For each slide, the code first adds the entering element and then removes the leaving one.

**Slide to [3, -1, -3]**

1. Add `-3` (entering element): Since -3 ≤ 1, add to `small`.
   - `small = {1, -1, -3}`, `small_size = 3`
   - Rebalance: `small_size (3) > large_size (1) + 1`, so move the top of `small` (1) to `large`
   - `small = {-1, -3}`, `small_size = 2`; `large = {1, 3}`, `large_size = 2`
2. Remove `1` (leaving element): Mark `delayed = {1: 1}`. Since 1 > -1 (max of `small`), it belongs to `large`, so `large_size = 1`. It is also the top of `large`, so `prune` pops it immediately and clears it from `delayed`.
   - `large = {3}`
   - Rebalance check: sizes 2 and 1 are valid
3. Find median: max of `small` = **-1**

**Slide to [-1, -3, 5]**

1. Add `5` (entering element): Since 5 > -1, add to `large`.
   - `large = {3, 5}`, `large_size = 2`
   - Rebalance check: sizes 2 and 2 are valid
2. Remove `3` (leaving element): Mark `delayed = {3: 1}`. Since 3 > -1, it belongs to `large`, so `large_size = 1`. It is the top of `large`, so it is pruned immediately.
   - `large = {5}`
3. Find median: max of `small` = **-1**

**Slide to [-3, 5, 3]**

1. Add `3` (entering element): Since 3 > -1, add to `large`.
   - `large = {3, 5}`, `large_size = 2`
   - Rebalance check: sizes 2 and 2 are valid
2. Remove `-1` (leaving element): Mark `delayed = {-1: 1}`. Since -1 ≤ -1, it belongs to `small`, so `small_size = 1`. It is the top of `small`, so it is pruned immediately.
   - `small = {-3}`
   - Rebalance: `small_size (1) < large_size (2)`, so move the top of `large` (3) to `small`
   - `small = {3, -3}`, `small_size = 2`; `large = {5}`, `large_size = 1`
3. Find median: max of `small` = **3**

Final result: `[1, -1, -1, 3]`

The key insight is how lazy deletion allows us to mark elements for removal without searching through the heaps, and how the two-heap structure with rebalancing maintains quick access to the median at all times.

## Python Solution

```python
from collections import defaultdict
from heapq import heappush, heappop
from typing import List

class MedianFinder:
    """
    A data structure that maintains the median of a sliding window of size k.
    Uses two heaps (max heap for smaller half, min heap for larger half) with lazy deletion.
    """
  
    def __init__(self, k: int):
        """
        Initialize the MedianFinder with window size k.
      
        Args:
            k: The size of the sliding window
        """
        self.k = k
        # Max heap for smaller half (negated values for max heap behavior)
        self.small = []
        # Min heap for larger half
        self.large = []
        # Track elements to be lazily deleted with their counts
        self.delayed = defaultdict(int)
        # Actual size of elements in small heap (excluding delayed deletions)
        self.small_size = 0
        # Actual size of elements in large heap (excluding delayed deletions)
        self.large_size = 0

    def add_num(self, num: int) -> None:
        """
        Add a number to the data structure.
      
        Args:
            num: The number to add
        """
        # Determine which heap to add the number to
        if not self.small or num <= -self.small[0]:
            # Add to small heap (negate for max heap behavior)
            heappush(self.small, -num)
            self.small_size += 1
        else:
            # Add to large heap
            heappush(self.large, num)
            self.large_size += 1
      
        # Ensure heaps remain balanced
        self.rebalance()

    def find_median(self) -> float:
        """
        Find the median of current elements in the window.
      
        Returns:
            The median value as a float
        """
        # If k is odd, median is the top of small heap
        # If k is even, median is average of tops of both heaps
        return -self.small[0] if self.k & 1 else (-self.small[0] + self.large[0]) / 2

    def remove_num(self, num: int) -> None:
        """
        Remove a number from the data structure using lazy deletion.
      
        Args:
            num: The number to remove
        """
        # Mark the number for lazy deletion
        self.delayed[num] += 1
      
        # Update size and prune if the number is at the top
        if num <= -self.small[0]:
            self.small_size -= 1
            if num == -self.small[0]:
                self.prune(self.small)
        else:
            self.large_size -= 1
            if num == self.large[0]:
                self.prune(self.large)
      
        # Ensure heaps remain balanced
        self.rebalance()

    def prune(self, heap: List[int]) -> None:
        """
        Remove all delayed elements from the top of the heap.
      
        Args:
            heap: The heap to prune (either self.small or self.large)
        """
        # Determine sign for value conversion (-1 for small heap, 1 for large heap)
        sign = -1 if heap is self.small else 1
      
        # Remove all delayed elements from the top
        while heap and sign * heap[0] in self.delayed:
            # Decrease the delay count
            self.delayed[sign * heap[0]] -= 1
            # Remove from delayed dict if count reaches 0
            if self.delayed[sign * heap[0]] == 0:
                self.delayed.pop(sign * heap[0])
            # Remove from heap
            heappop(heap)

    def rebalance(self) -> None:
        """
        Rebalance the two heaps to maintain the median property.
        Small heap should have equal or one more element than large heap.
        """
        # If small heap has too many elements, move one to large heap
        if self.small_size > self.large_size + 1:
            heappush(self.large, -heappop(self.small))
            self.small_size -= 1
            self.large_size += 1
            # Clean up any delayed elements exposed at the top
            self.prune(self.small)
        # If large heap has more elements, move one to small heap
        elif self.small_size < self.large_size:
            heappush(self.small, -heappop(self.large))
            self.large_size -= 1
            self.small_size += 1
            # Clean up any delayed elements exposed at the top
            self.prune(self.large)


class Solution:
    """
    Solution for the Sliding Window Median problem.
    """
  
    def medianSlidingWindow(self, nums: List[int], k: int) -> List[float]:
        """
        Find the median of each sliding window of size k in the array.
      
        Args:
            nums: The input array of numbers
            k: The size of the sliding window
          
        Returns:
            List of medians for each window position
        """
        # Initialize the median finder with window size k
        finder = MedianFinder(k)
      
        # Add the first k elements to initialize the window
        for num in nums[:k]:
            finder.add_num(num)
      
        # Calculate the median for the first window
        result = [finder.find_median()]
      
        # Slide the window through the rest of the array
        for i in range(k, len(nums)):
            # Add new element entering the window
            finder.add_num(nums[i])
            # Remove element leaving the window
            finder.remove_num(nums[i - k])
            # Calculate and store the median for current window
            result.append(finder.find_median())
      
        return result
```

## Time and Space Complexity

**Time Complexity:** `O(n × log k)`

The algorithm uses a sliding window approach with two heaps to maintain the median. For each of the `n` elements in the array:
- `add_num()`: Inserting into a heap takes `O(log k)` time since the heap size is bounded by `k`
- `remove_num()`: Uses lazy deletion - marking for deletion is `O(1)`, but pruning may trigger multiple heap pops in worst case
- `prune()`: In amortized analysis, each element is popped at most once, contributing `O(log k)` per element
- `rebalance()`: Involves at most one heap pop and push operation, which is `O(log k)`
- `find_median()`: Returns the top element(s) of heaps in `O(1)` time

Since we process `n` elements and each operation involves heap operations on heaps of size at most `k`, the total time complexity is `O(n × log k)`.

Because the heaps hold the current window (plus delayed elements that are removed lazily), heap operations take `O(log k)` amortized, and the overall complexity is `O(n × log k)`.

**Space Complexity:** `O(n)`

- The two heaps (`small` and `large`) together store at most `k` elements at any time: `O(k)`
- The `delayed` dictionary in worst case could store all unique elements from the array if they all get delayed for deletion: `O(n)` 
- The result array `ans` stores `n - k + 1` median values: `O(n)`

The dominant space usage is `O(n)` from the delayed dictionary and result array.

## Common Pitfalls

### 1. **Letting a Delayed Element Stay at a Heap Top**

**Pitfall**: `remove_num` decides which heap holds `num` by comparing it with `-self.small[0]`, and `find_median` reads both heap tops. Both are correct only if the top of each heap is always a live (non-delayed) element. Forgetting to call `prune` after a removal hits the top, or after `rebalance` pops the top of a heap, breaks this.

**Example Scenario**: With window `[1, 3, -1]`, `small = {1, -1}` and `large = {3}`. Suppose a version of `remove_num` marks `delayed = {1: 1}` but never prunes. The top of `small` is still the deleted value 1, so `find_median` returns 1 for the next window even though 1 has left it.

A related mistake is pruning `small` before deciding which heap `num` belongs to. When `num` is itself the top of `small`, the prune pops it, the comparison then runs against the next element, and `num` is counted as a removal from `large`, so `large_size` drops instead of `small_size`.

**Solution**: Keep the order used in the solution: mark the element as delayed, classify it against the current top, update the matching size counter, and prune only the heap whose top equals `num`. `rebalance` then prunes the heap it popped from:
```python
def remove_num(self, num: int) -> None:
    self.delayed[num] += 1
    if num <= -self.small[0]:
        self.small_size -= 1
        if num == -self.small[0]:
            self.prune(self.small)
    else:
        self.large_size -= 1
        if num == self.large[0]:
            self.prune(self.large)
    self.rebalance()
```

### 2. **Integer Overflow in Median Calculation**

**Pitfall**: For very large integers, calculating `(-small[0] + large[0]) / 2` might cause overflow in languages with fixed integer sizes.

**Solution**: Use floating-point division from the start:
```python
def find_median(self) -> float:
    if self.k & 1:
        return float(-self.small[0])
    else:
        return (float(-self.small[0]) + float(self.large[0])) / 2.0
```

### 3. **Not Handling Edge Cases in Rebalancing**

**Pitfall**: The rebalance function might try to pop from an empty heap after pruning, especially when all elements in a heap are marked for deletion.

**Solution**: Add safety checks:
```python
def rebalance(self) -> None:
    if self.small_size > self.large_size + 1:
        # Ensure small heap has valid elements before popping
        self.prune(self.small)
        if self.small:
            heappush(self.large, -heappop(self.small))
            self.small_size -= 1
            self.large_size += 1
            self.prune(self.small)
    elif self.small_size < self.large_size:
        # Ensure large heap has valid elements before popping
        self.prune(self.large)
        if self.large:
            heappush(self.small, -heappop(self.large))
            self.large_size -= 1
            self.small_size += 1
            self.prune(self.large)
```

### 4. **Inefficient Pruning Strategy**

**Pitfall**: Only pruning when necessary (at heap tops) can lead to heap bloat with many delayed elements accumulating in the middle of heaps, causing increased memory usage and slower heap operations.

**Solution**: Implement periodic full cleanup or track a threshold:
```python
def add_num(self, num: int) -> None:
    # Existing add logic...
  
    # Periodic cleanup when delayed elements exceed threshold
    if len(self.delayed) > self.k // 2:
        self.full_cleanup()
  
    self.rebalance()

def full_cleanup(self) -> None:
    # Rebuild heaps without delayed elements
    valid_nums = []
    for val in [-v for v in self.small] + self.large:
        if self.delayed.get(val, 0) > 0:
            self.delayed[val] -= 1  # Skip one deleted copy only
        else:
            valid_nums.append(val)
  
    # Clear delayed dictionary and rebuild
    self.delayed.clear()
    self.small = []
    self.large = []
    self.small_size = 0
    self.large_size = 0
  
    for num in valid_nums:
        self.add_num(num)
```

### 5. **Duplicate Values Handling**

**Pitfall**: When the same value appears multiple times and some instances are delayed while others are active, the lazy deletion counting can become inconsistent.

**Solution**: The current implementation handles this correctly by using a count-based approach in the `delayed` dictionary, but ensure the count never goes negative:
```python
def prune(self, heap: List[int]) -> None:
    sign = -1 if heap is self.small else 1
  
    while heap and sign * heap[0] in self.delayed:
        # Ensure we don't over-decrement
        if self.delayed[sign * heap[0]] > 0:
            self.delayed[sign * heap[0]] -= 1
            if self.delayed[sign * heap[0]] == 0:
                self.delayed.pop(sign * heap[0])
        heappop(heap)
```

