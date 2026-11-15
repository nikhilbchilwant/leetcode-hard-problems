# 1825. Finding MK Average

> Source: https://algo.monster/liteproblems/1825

## Problem Statement

You need to implement a data structure that calculates a special type of average called **MKAverage** from a stream of integers.

Given two integers `m` and `k`, the MKAverage is calculated as follows:

1. **Check stream size**: If the stream has fewer than `m` elements, the MKAverage is `-1`.

2. **Take last m elements**: Otherwise, consider only the last `m` elements from the stream.

3. **Remove extremes**: From these `m` elements, remove the `k` smallest values and the `k` largest values.

4. **Calculate average**: Compute the average of the remaining `m - 2k` elements, rounded down to the nearest integer.

You need to implement a `MKAverage` class with three methods:

- **`MKAverage(int m, int k)`**: Constructor that initializes the object with an empty stream and the parameters `m` and `k`.

- **`void addElement(int num)`**: Adds a new element `num` to the stream.

- **`int calculateMKAverage()`**: Returns the MKAverage of the current stream (considering only the last `m` elements if the stream has at least `m` elements). Returns `-1` if the stream has fewer than `m` elements.

For example, if `m = 5` and `k = 1`, and the stream contains `[3, 1, 10, 5, 7, 2]`:
- Take the last 5 elements: `[1, 10, 5, 7, 2]`
- Remove 1 smallest (`1`) and 1 largest (`10`): `[5, 7, 2]`
- Calculate average: `(5 + 7 + 2) / 3 = 14 / 3 = 4` (rounded down)

## Intuition

The key challenge is efficiently maintaining the middle elements (after removing `k` smallest and `k` largest) as we add new elements and potentially remove old ones from our sliding window of size `m`.

The naive approach would be to sort the last `m` elements every time we need to calculate the average, but this would be inefficient with `O(m log m)` time for each query.

Instead, we can maintain the elements in a way that keeps them already "partitioned" into three groups:
- The `k` smallest elements
- The middle `m - 2k` elements  
- The `k` largest elements

This leads us to think about using three separate ordered data structures. When a new element arrives, we can determine which group it belongs to by comparing it with the boundaries of these groups (the maximum of the smallest group and the minimum of the largest group).

Since we only care about the last `m` elements, we need a [queue](/problems/queue_intro) to track the order of insertion. When the queue exceeds size `m`, we remove the oldest element from whichever group contains it.

The crucial insight is that after adding or removing elements, we might need to rebalance the groups to ensure:
- The `lo` group has exactly `k` elements (or less if we don't have enough total elements)
- The `hi` group has exactly `k` elements (or less if we don't have enough total elements)
- The `mid` group contains everything else

This rebalancing involves moving elements between adjacent groups. For example, if `lo` has more than `k` elements, we move its largest element to `mid`. If `lo` has fewer than `k` elements and `mid` is not empty, we move the smallest element from `mid` to `lo`.

By maintaining the sum of only the middle group elements, we can calculate the MKAverage in `O(1)` time by simply dividing this sum by `m - 2k`.

**Learn more about [Queue](/problems/queue_intro), [Data Stream](/problems/median_of_data_stream) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

The solution uses three ordered sets and a [queue](/problems/queue_intro) to efficiently maintain the required data structure:

### Data Structures:
1. **[Queue](/problems/queue_intro) `q`**: Stores the last `m` elements in order of insertion, with the head being the oldest element
2. **Three SortedLists**: 
   - `lo`: Stores the `k` smallest elements
   - `hi`: Stores the `k` largest elements  
   - `mid`: Stores the remaining middle elements
3. **Variable `s`**: Maintains the running sum of all elements in `mid`

### Implementation Details:

**Adding an Element (`addElement`):**

1. **Initial Placement**: When a new element `num` arrives, determine which set it belongs to:
   - If `lo` is empty or `num ≤ max(lo)`, add to `lo`
   - Else if `hi` is empty or `num ≥ min(hi)`, add to `hi`
   - Otherwise, add to `mid` and update sum `s += num`

2. **Maintain Window Size**: Add `num` to [queue](/problems/queue_intro) `q`. If queue length exceeds `m`:
   - Remove the oldest element `x` from the queue
   - Find and remove `x` from whichever set (`lo`, `mid`, or `hi`) contains it
   - If removing from `mid`, update sum `s -= x`

3. **Rebalancing Operations**: After insertion/deletion, ensure each set has the correct size:
   - While `len(lo) > k`: Move `max(lo)` to `mid`, update `s += max(lo)`
   - While `len(hi) > k`: Move `min(hi)` to `mid`, update `s += min(hi)`
   - While `len(lo) < k` and `mid` is not empty: Move `min(mid)` to `lo`, update `s -= min(mid)`
   - While `len(hi) < k` and `mid` is not empty: Move `max(mid)` to `hi`, update `s -= max(mid)`

**Calculating MKAverage (`calculateMKAverage`):**
- If [queue](/problems/queue_intro) length < `m`, return `-1`
- Otherwise, return `s // (m - 2*k)` (integer division for rounding down)

### Time Complexity:
- `addElement`: `O(log m)` for insertions/removals in sorted lists
- `calculateMKAverage`: `O(1)` since we maintain the sum

### Space Complexity:
- `O(m)` to store at most `m` elements across all data structures

The key insight is that by maintaining these three partitioned sets and rebalancing them after each operation, we avoid having to sort the entire window repeatedly and can calculate the average in constant time.

## Example Walkthrough

Let's walk through an example with `m = 5`, `k = 1`, and the following sequence of operations:

**Initial State:**
- `lo = []`, `mid = []`, `hi = []`, `q = []`, `s = 0`

**addElement(3):**
1. Since `lo` is empty, add 3 to `lo`: `lo = [3]`
2. Add to queue: `q = [3]`
3. Queue size (1) < m (5), no removal needed
4. Rebalance: `len(lo) = 1 = k`, no rebalancing needed
- State: `lo = [3]`, `mid = []`, `hi = []`, `s = 0`

**addElement(1):**
1. 1 ≤ max(lo) = 3, so add to `lo`: `lo = [1, 3]`
2. Add to queue: `q = [3, 1]`
3. Queue size (2) < m (5), no removal
4. Rebalance: `len(lo) = 2 > k = 1`, move max(lo) = 3 to `mid`
- State: `lo = [1]`, `mid = [3]`, `hi = []`, `s = 3`

**addElement(10):**
1. 10 > max(lo) = 1 and `hi` is empty, add to `hi`: `hi = [10]`
2. Add to queue: `q = [3, 1, 10]`
3. Queue size (3) < m (5), no removal
4. Rebalance: All sets have correct sizes
- State: `lo = [1]`, `mid = [3]`, `hi = [10]`, `s = 3`

**addElement(5):**
1. 5 > max(lo) = 1 and 5 < min(hi) = 10, add to `mid`: `mid = [3, 5]`
2. Add to queue: `q = [3, 1, 10, 5]`, `s = 3 + 5 = 8`
3. Queue size (4) < m (5), no removal
4. Rebalance: All sets have correct sizes
- State: `lo = [1]`, `mid = [3, 5]`, `hi = [10]`, `s = 8`

**addElement(7):**
1. 7 > max(lo) = 1 and 7 < min(hi) = 10, add to `mid`: `mid = [3, 5, 7]`
2. Add to queue: `q = [3, 1, 10, 5, 7]`, `s = 8 + 7 = 15`
3. Queue size (5) = m (5), no removal yet
4. Rebalance: All sets have correct sizes
- State: `lo = [1]`, `mid = [3, 5, 7]`, `hi = [10]`, `s = 15`

**calculateMKAverage():**
- Queue size = 5 ≥ m = 5
- Return `s // (m - 2*k) = 15 // 3 = 5`

**addElement(2):**
1. 2 > max(lo) = 1 and 2 < min(hi) = 10, add to `mid`: `mid = [2, 3, 5, 7]`
2. Add to queue: `q = [3, 1, 10, 5, 7, 2]`, `s = 15 + 2 = 17`
3. Queue size (6) > m (5), remove oldest element 3:
   - Find 3 in `mid`, remove it: `mid = [2, 5, 7]`
   - Update sum: `s = 17 - 3 = 14`
   - Remove from queue: `q = [1, 10, 5, 7, 2]`
4. Rebalance: All sets have correct sizes
- Final state: `lo = [1]`, `mid = [2, 5, 7]`, `hi = [10]`, `s = 14`

**calculateMKAverage():**
- The last 5 elements are `[1, 10, 5, 7, 2]`
- After removing k=1 smallest (1) and k=1 largest (10): `[2, 5, 7]`
- Return `s // (m - 2*k) = 14 // 3 = 4`

This walkthrough demonstrates how the three sets maintain the partitioning, how rebalancing ensures correct sizes, and how the running sum enables O(1) average calculation.

## Python Solution

```python
from collections import deque
from sortedcontainers import SortedList

class MKAverage:
    def __init__(self, m: int, k: int):
        """
        Initialize the MKAverage data structure.
      
        Args:
            m: The number of most recent elements to consider
            k: The number of smallest and largest elements to exclude
        """
        self.m = m  # Size of sliding window
        self.k = k  # Number of elements to exclude from each end
        self.middle_sum = 0  # Sum of middle elements (excluding k smallest and k largest)
        self.queue = deque()  # Queue to maintain last m elements in order
      
        # Three sorted lists to maintain k smallest, middle, and k largest elements
        self.smallest = SortedList()  # Stores k smallest elements
        self.middle = SortedList()    # Stores middle elements (m - 2k elements)
        self.largest = SortedList()   # Stores k largest elements

    def addElement(self, num: int) -> None:
        """
        Add a new element to the data structure.
      
        Args:
            num: The number to add
        """
        # Determine which sorted list the new element belongs to
        if not self.smallest or num <= self.smallest[-1]:
            # Number belongs to smallest k elements
            self.smallest.add(num)
        elif not self.largest or num >= self.largest[0]:
            # Number belongs to largest k elements
            self.largest.add(num)
        else:
            # Number belongs to middle elements
            self.middle.add(num)
            self.middle_sum += num
      
        # Add element to queue
        self.queue.append(num)
      
        # Remove oldest element if queue exceeds m elements
        if len(self.queue) > self.m:
            oldest = self.queue.popleft()
          
            # Remove oldest element from appropriate sorted list
            if oldest in self.smallest:
                self.smallest.remove(oldest)
            elif oldest in self.largest:
                self.largest.remove(oldest)
            else:
                self.middle.remove(oldest)
                self.middle_sum -= oldest
      
        # Rebalance the three sorted lists to maintain invariants
      
        # Move excess elements from smallest to middle
        while len(self.smallest) > self.k:
            element = self.smallest.pop()  # Remove largest from smallest
            self.middle.add(element)
            self.middle_sum += element
      
        # Move excess elements from largest to middle
        while len(self.largest) > self.k:
            element = self.largest.pop(0)  # Remove smallest from largest
            self.middle.add(element)
            self.middle_sum += element
      
        # Move elements from middle to smallest if needed
        while len(self.smallest) < self.k and self.middle:
            element = self.middle.pop(0)  # Remove smallest from middle
            self.smallest.add(element)
            self.middle_sum -= element
      
        # Move elements from middle to largest if needed
        while len(self.largest) < self.k and self.middle:
            element = self.middle.pop()  # Remove largest from middle
            self.largest.add(element)
            self.middle_sum -= element

    def calculateMKAverage(self) -> int:
        """
        Calculate the MKAverage of current elements.
      
        Returns:
            The average of middle elements (excluding k smallest and k largest),
            or -1 if there are fewer than m elements
        """
        # Return -1 if we don't have enough elements
        if len(self.queue) < self.m:
            return -1
      
        # Calculate average of middle elements (integer division)
        middle_count = self.m - 2 * self.k
        return self.middle_sum // middle_count


# Your MKAverage object will be instantiated and called as such:
# obj = MKAverage(m, k)
# obj.addElement(num)
# param_2 = obj.calculateMKAverage()
```

## Time and Space Complexity

**Time Complexity:**
- `addElement(num)`: `O(log m)` per operation
  - Adding to `SortedList` (lo, mid, or hi): `O(log m)`
  - Appending to deque: `O(1)`
  - When queue exceeds size `m`, removing from deque: `O(1)` and removing from one of the SortedLists: `O(log m)`
  - Rebalancing operations (while loops): Each element movement involves removing from one SortedList (`O(log m)`) and adding to another (`O(log m)`). In the worst case, we might need to move `O(k)` elements, but amortized over many operations, each element is moved a constant number of times, keeping the overall complexity at `O(log m)`

- `calculateMKAverage()`: `O(1)`
  - Simply returns the pre-calculated sum `s` divided by `(m - 2 * k)`, which is a constant time operation

**Space Complexity:** `O(m)`
- The deque `q` stores at most `m` elements
- The three SortedLists (`lo`, `mid`, `hi`) together store exactly the same elements as in the deque, totaling `m` elements
- Additional variables (`s`, `m`, `k`) use `O(1)` space
- Total space complexity is `O(m)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Rebalancing Order**
One of the most critical pitfalls is performing the rebalancing operations in the wrong order. The rebalancing must follow a specific sequence to maintain the invariants correctly.

**Wrong Approach:**
```python
# Moving elements to smallest/largest before handling excess
while len(self.smallest) < self.k and self.middle:
    element = self.middle.pop(0)
    self.smallest.add(element)
    self.middle_sum -= element

while len(self.smallest) > self.k:
    element = self.smallest.pop()
    self.middle.add(element)
    self.middle_sum += element
```

**Why it's wrong:** If you try to fill `smallest` or `largest` before removing excess elements, you might move elements unnecessarily and corrupt the data structure state.

**Correct Approach:** Always handle excess elements first (moving from `smallest`/`largest` to `middle`), then handle deficits (moving from `middle` to `smallest`/`largest`).

### 2. **Incorrect Element Removal from SortedList**
When removing the oldest element from the queue, using the wrong removal method can lead to removing the wrong element or errors.

**Wrong Approach:**
```python
if oldest in self.smallest:
    self.smallest.pop(0)  # Wrong! This removes the minimum, not 'oldest'
```

**Correct Approach:**
```python
if oldest in self.smallest:
    self.smallest.remove(oldest)  # Removes the specific element
```

### 3. **Not Handling Edge Cases in Initial Placement**
When adding a new element, comparing with empty lists can cause errors.

**Wrong Approach:**
```python
if num <= self.smallest[-1]:  # Error if smallest is empty!
    self.smallest.add(num)
```

**Correct Approach:**
```python
if not self.smallest or num <= self.smallest[-1]:
    self.smallest.add(num)
```

### 4. **Forgetting to Update the Sum**
The `middle_sum` must be updated whenever elements move in or out of the `middle` list.

**Wrong Approach:**
```python
# Forgetting to update sum when removing from middle
if oldest in self.middle:
    self.middle.remove(oldest)
    # Missing: self.middle_sum -= oldest
```

**Correct Approach:** Always update `middle_sum` when:
- Adding to `middle`: `self.middle_sum += num`
- Removing from `middle`: `self.middle_sum -= num`
- Moving elements in/out during rebalancing

### 5. **Using Regular List Instead of SortedList**
Using Python's regular list and manually sorting will result in O(m log m) complexity for each operation.

**Wrong Approach:**
```python
self.smallest = []  # Regular list
# Later...
self.smallest.append(num)
self.smallest.sort()  # O(m log m) every time!
```

**Correct Approach:** Use `SortedList` from `sortedcontainers` which maintains sorted order with O(log m) insertions and removals.

### 6. **Integer Division vs Float Division**
The problem requires rounding down to the nearest integer.

**Wrong Approach:**
```python
return self.middle_sum / middle_count  # Returns float
```

**Correct Approach:**
```python
return self.middle_sum // middle_count  # Integer division (rounds down)
```

### Prevention Tips:
1. Test with edge cases where `k = 0` or `2k = m`
2. Verify the sum is correctly maintained by checking `sum(middle) == middle_sum` during debugging
3. Test with duplicate values to ensure removal works correctly
4. Always handle the "excess" case before the "deficit" case in rebalancing

