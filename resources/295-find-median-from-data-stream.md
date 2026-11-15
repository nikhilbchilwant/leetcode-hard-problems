# 295. Find Median from Data Stream

> Source: https://algo.monster/liteproblems/295

## Problem Statement

This problem asks you to design a data structure that efficiently finds the median value from a stream of integers.

The median is defined as:
- For an odd-sized list: the middle element when sorted
- For an even-sized list: the average of the two middle elements when sorted

For example:
- `[2,3,4]` has median `3` (the middle element)
- `[2,3]` has median `2.5` (average of 2 and 3)

You need to implement a `MedianFinder` class with three methods:

1. `MedianFinder()`: Initializes an empty data structure
2. `addNum(int num)`: Adds a new integer from the data stream to your data structure
3. `findMedian()`: Returns the current median of all numbers added so far

The challenge is to maintain the median efficiently as new numbers are continuously added. The solution uses two heaps:
- A max heap (`maxq`) to store the smaller half of numbers (with the largest of the small numbers at the top)
- A min heap (`minq`) to store the larger half of numbers (with the smallest of the large numbers at the top)

The key insight is that by maintaining these two heaps with balanced sizes (difference of at most 1), the median can always be found from the top elements of the heaps:
- When both heaps have equal size: median is the average of both tops
- When `minq` has one more element: median is the top of `minq`

The `addNum` operation maintains this balance by:
1. First adding the new number to `maxq` (smaller half)
2. Moving the largest element from `maxq` to `minq` to ensure proper ordering
3. Rebalancing if `minq` becomes too large by moving its smallest element back to `maxq`

This design allows `addNum` to run in `O(log n)` time and `findMedian` to run in `O(1)` time.

## Intuition

The naive approach would be to store all numbers in a list and sort it every time we need to find the median. However, this would be inefficient with `O(n log n)` time for each median query.

The key observation is that to find the median, we don't need the entire sorted array - we only need to know the middle element(s). This leads us to think: what if we could partition the numbers into two halves and only track the "boundary" between them?

Imagine splitting the sorted array into two parts:
- Left half: smaller numbers
- Right half: larger numbers

The median would always be at the boundary between these two halves. If we could efficiently maintain this partition and quickly access the maximum of the left half and minimum of the right half, we could find the median instantly.

This naturally leads to using heaps:
- A max heap for the left half (smaller numbers) - gives us the largest of the small numbers
- A min heap for the right half (larger numbers) - gives us the smallest of the large numbers

Why this works beautifully:
1. The top of the max heap is the largest element in the smaller half
2. The top of the min heap is the smallest element in the larger half
3. These two elements are exactly the middle elements of the entire dataset!

The balancing strategy ensures that:
- Both heaps have roughly equal size (differ by at most 1)
- Every element in the max heap ≤ every element in the min heap

When adding a new number, we maintain these invariants by:
1. First adding to the max heap (arbitrarily chosen as the entry point)
2. Moving the largest from max heap to min heap (ensures the ordering property)
3. Rebalancing if needed (ensures the size property)

This elegant design transforms the problem from "finding median in a sorted array" to "maintaining two balanced heaps", which is much more efficient for a stream of data.

## Solution Approach

The implementation uses two heaps to maintain the median efficiently:

**Data Structures:**
- `minq`: A min heap storing the larger half of numbers
- `maxq`: A max heap storing the smaller half of numbers

In Python, we use the `heapq` module which only provides min heap functionality. To simulate a max heap, we negate the values when inserting into `maxq`.

**Initialization (`__init__`):**
```python
self.minq = []  # Min heap for larger half
self.maxq = []  # Max heap for smaller half
```

**Adding Numbers (`addNum`):**

The algorithm ensures two invariants:
1. Every element in `maxq` ≤ every element in `minq`
2. Size difference between heaps is at most 1

```python
heappush(self.minq, -heappushpop(self.maxq, -num))
```
This line does two things:
- First, add `num` to `maxq` (negated to simulate max heap): `heappushpop(self.maxq, -num)`
- Then immediately pop the maximum from `maxq` and push it to `minq` (negated back to positive)

This ensures that even if `num` was supposed to go to `minq`, it will end up there after this operation, maintaining the ordering invariant.

After this operation, `minq` always gains one element. So we check if rebalancing is needed:
```python
if len(self.minq) - len(self.maxq) > 1:
    heappush(self.maxq, -heappop(self.minq))
```
If `minq` has more than one extra element compared to `maxq`, we move the smallest element from `minq` back to `maxq`.

**Finding Median (`findMedian`):**

Two cases based on the total number of elements:

1. **Even number of elements** (both heaps have equal size):
   ```python
   return (self.minq[0] - self.maxq[0]) / 2
   ```
   The median is the average of:
   - Top of `minq` (smallest of larger half): `self.minq[0]`
   - Top of `maxq` (largest of smaller half): `-self.maxq[0]` (remember values are negated)

2. **Odd number of elements** (`minq` has one extra element):
   ```python
   return self.minq[0]
   ```
   The median is simply the top of `minq`.

**Time Complexity:**
- `addNum`: `O(log n)` - heap operations
- `findMedian`: `O(1)` - just accessing heap tops

**Space Complexity:** `O(n)` - storing all n elements across two heaps

## Example Walkthrough

Let's walk through adding numbers `[5, 3, 8, 2, 10]` one by one and finding the median after each addition.

**Initial State:**
- `maxq = []` (max heap for smaller half)
- `minq = []` (min heap for larger half)

**Step 1: Add 5**
1. Add 5 to maxq: `maxq = [-5]` (negated for max heap)
2. Pop max from maxq (-5) and push to minq: `maxq = [], minq = [5]`
3. Check balance: `len(minq) - len(maxq) = 1 - 0 = 1` (balanced)
4. **Median = 5** (odd count, return top of minq)

**Step 2: Add 3**
1. Add 3 to maxq: `maxq = [-3]`
2. Pop max from maxq (-3) and push to minq: `maxq = [], minq = [3, 5]`
3. Check balance: `len(minq) - len(maxq) = 2 - 0 = 2` (unbalanced!)
4. Rebalance: Move min from minq to maxq: `maxq = [-3], minq = [5]`
5. **Median = (5 + 3) / 2 = 4** (even count, average of tops)

**Step 3: Add 8**
1. Add 8 to maxq: `maxq = [-8, -3]`
2. Pop max from maxq (-8) and push to minq: `maxq = [-3], minq = [5, 8]`
3. Check balance: `len(minq) - len(maxq) = 2 - 1 = 1` (balanced)
4. **Median = 5** (odd count, return top of minq)
   - Current sorted order: [3, 5, 8]

**Step 4: Add 2**
1. Add 2 to maxq: `maxq = [-3, -2]`
2. Pop max from maxq (-3) and push to minq: `maxq = [-2], minq = [3, 8, 5]` → heap adjusts to `[3, 5, 8]`
3. Check balance: `len(minq) - len(maxq) = 3 - 1 = 2` (unbalanced!)
4. Rebalance: Move min from minq to maxq: `maxq = [-3, -2], minq = [5, 8]`
5. **Median = (3 + 5) / 2 = 4** (even count, average of tops)
   - Current sorted order: [2, 3, 5, 8]

**Step 5: Add 10**
1. Add 10 to maxq: `maxq = [-10, -2, -3]`
2. Pop max from maxq (-10) and push to minq: `maxq = [-3, -2], minq = [5, 8, 10]`
3. Check balance: `len(minq) - len(maxq) = 3 - 2 = 1` (balanced)
4. **Median = 5** (odd count, return top of minq)
   - Current sorted order: [2, 3, 5, 8, 10]

Notice how the median is always efficiently found from the heap tops without needing to sort all elements!

## Python Solution

```python
from heapq import heappush, heappop, heappushpop

class MedianFinder:
    """
    A data structure that supports adding numbers and finding the median efficiently.
    Uses two heaps to maintain the median in O(log n) time for additions and O(1) for retrieval.
    """

    def __init__(self):
        """
        Initialize the MedianFinder with two heaps:
        - min_heap: stores the larger half of numbers (min heap)
        - max_heap: stores the smaller half of numbers (implemented as negative min heap)
        """
        self.min_heap = []  # Stores larger half of numbers
        self.max_heap = []  # Stores smaller half of numbers (negated for max heap behavior)

    def addNum(self, num: int) -> None:
        """
        Add a number to the data structure.
      
        Algorithm:
        1. Push num to max_heap (smaller half)
        2. Move the largest from max_heap to min_heap (larger half)
        3. Balance the heaps if min_heap has more than 1 extra element
      
        Args:
            num: Integer to add to the data structure
        """
        # Add to max_heap first, then move its maximum to min_heap
        # This ensures all elements in max_heap are <= all elements in min_heap
        heappush(self.min_heap, -heappushpop(self.max_heap, -num))
      
        # Balance the heaps: min_heap can have at most 1 more element than max_heap
        if len(self.min_heap) - len(self.max_heap) > 1:
            # Move smallest element from min_heap to max_heap
            heappush(self.max_heap, -heappop(self.min_heap))

    def findMedian(self) -> float:
        """
        Find the median of all numbers added so far.
      
        Returns:
            The median as a float
        """
        # If both heaps have equal size, median is average of both tops
        if len(self.min_heap) == len(self.max_heap):
            # min_heap[0] is smallest of larger half
            # -max_heap[0] is largest of smaller half
            return (self.min_heap[0] - self.max_heap[0]) / 2.0
      
        # If unequal size, min_heap has one extra element which is the median
        return float(self.min_heap[0])


# Your MedianFinder object will be instantiated and called as such:
# obj = MedianFinder()
# obj.addNum(num)
# param_2 = obj.findMedian()
```

## Time and Space Complexity

**Time Complexity:**
- `addNum(num)`: `O(log n)` where `n` is the number of elements already in the data structure. This is because:
  - `heappushpop` on `maxq` takes `O(log n)` time
  - `heappush` on `minq` takes `O(log n)` time  
  - The conditional rebalancing with `heappop` and `heappush` takes `O(log n)` time when executed
  - Overall: `O(log n)` for each insertion

- `findMedian()`: `O(1)` since we're only accessing the top elements of the heaps and performing constant-time arithmetic operations

**Space Complexity:**
- `O(n)` where `n` is the total number of elements added to the MedianFinder
- The space is used to store all `n` elements across two heaps (`minq` and `maxq`)
- The sum of elements in both heaps equals the total number of elements inserted

## Common Pitfalls

### 1. **Incorrect Heap Balancing Logic**

**Pitfall:** A common mistake is to always alternate between heaps or use simple size comparison without considering the actual values. For example:

```python
# WRONG: Simply alternating between heaps
def addNum(self, num):
    if len(self.min_heap) <= len(self.max_heap):
        heappush(self.min_heap, num)
    else:
        heappush(self.max_heap, -num)
```

This breaks the invariant that all elements in `max_heap` should be ≤ all elements in `min_heap`.

**Solution:** Always route through one heap first, then transfer to maintain ordering:
```python
# CORRECT: Ensures proper ordering
heappush(self.min_heap, -heappushpop(self.max_heap, -num))
```

### 2. **Forgetting to Negate Values for Max Heap**

**Pitfall:** Python's `heapq` only provides min heap. Forgetting to negate values when using it as a max heap leads to incorrect results:

```python
# WRONG: Treating max_heap as regular min heap
heappush(self.max_heap, num)  # Should be -num
median = (self.min_heap[0] + self.max_heap[0]) / 2  # Should be min_heap[0] - max_heap[0]
```

**Solution:** Always negate when pushing to and popping from the max heap:
```python
# CORRECT: Proper negation for max heap simulation
heappush(self.max_heap, -num)
median = (self.min_heap[0] - self.max_heap[0]) / 2
```

### 3. **Integer Division Instead of Float Division**

**Pitfall:** Using integer division when calculating the median of even-sized lists:

```python
# WRONG in Python 2 or with // operator
return (self.min_heap[0] - self.max_heap[0]) // 2  # Integer division
```

**Solution:** Use float division or explicitly convert to float:
```python
# CORRECT: Float division
return (self.min_heap[0] - self.max_heap[0]) / 2.0
# OR
return float(self.min_heap[0] - self.max_heap[0]) / 2
```

### 4. **Incorrect Heap Size Management**

**Pitfall:** Allowing the size difference between heaps to exceed 1, or always keeping min_heap larger:

```python
# WRONG: checks the wrong heap
if len(self.max_heap) - len(self.min_heap) > 1:
    heappush(self.min_heap, -heappop(self.max_heap))
```

With the routing `heappush(self.min_heap, -heappushpop(self.max_heap, -num))`, every call adds one element to `min_heap` and leaves `max_heap` the same size, so `max_heap` never grows and this condition is never true. After three calls `min_heap` holds all three numbers, `max_heap` is empty, and `findMedian` returns the smallest number instead of the middle one.

**Solution:** Consistently maintain that min_heap has either equal or one more element:
```python
# CORRECT: min_heap has at most 1 extra element
if len(self.min_heap) - len(self.max_heap) > 1:
    heappush(self.max_heap, -heappop(self.min_heap))
```

### 5. **Empty Heap Access**

**Pitfall:** Not handling the edge case when finding median with no elements added:

```python
def findMedian(self):
    # WRONG: Crashes if called before any addNum
    return self.min_heap[0]  # IndexError if empty
```

**Solution:** Either add a check or ensure the problem constraints guarantee at least one element:
```python
def findMedian(self):
    if not self.min_heap and not self.max_heap:
        return 0  # or raise an exception
    # ... rest of the logic
```

