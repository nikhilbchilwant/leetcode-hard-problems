# 3369. Design an Array Statistics Tracker

> Source: https://algo.monster/liteproblems/3369

## Problem Statement

You need to design a data structure that can track numbers and efficiently answer statistical queries about them. The data structure should support adding numbers, removing the oldest number, and computing the mean, median, and mode of the current numbers.

The `StatisticsTracker` class must implement these operations:

1. **`StatisticsTracker()`**: Creates an empty statistics tracker.

2. **`addNumber(int number)`**: Adds a new number to the data structure.

3. **`removeFirstAddedNumber()`**: Removes the earliest added number (following FIFO order).

4. **`getMean()`**: Returns the floored mean of all current numbers. The mean is calculated as the sum of all values divided by the count, with the result floored (rounded down).

5. **`getMedian()`**: Returns the median of all current numbers. The median is the middle value when numbers are sorted in non-decreasing order. When there's an even count of numbers (two middle values), return the larger one.

6. **`getMode()`**: Returns the mode of all current numbers. The mode is the most frequently occurring number. If multiple numbers have the same highest frequency, return the smallest one.

The solution uses multiple data structures to efficiently handle these operations:
- A [queue](/problems/queue_intro) (`deque`) to maintain the order of insertion for FIFO removal
- A variable to track the running sum for mean calculation
- A hash table to count occurrences of each number
- Two sorted lists: one for finding the median and another for tracking modes (sorted by frequency descending, then by value ascending)

Each operation is optimized: `getMean()` runs in `O(1)`, `getMedian()` and `getMode()` use sorted data structures for efficient access, while `addNumber()` and `removeFirstAddedNumber()` run in `O(log n)` due to the sorted list operations.

## Intuition

The key insight is that we need to efficiently support different statistical operations while maintaining the ability to remove elements in FIFO order. This suggests we need multiple data structures, each optimized for a specific operation.

For tracking insertion order, a [queue](/problems/queue_intro) is the natural choice since we need FIFO removal. We can't rely solely on the queue for statistics though, because finding the median or mode in an unsorted queue would require `O(n log n)` sorting or `O(n)` scanning for each query.

For the mean, we realize we don't need to sum all elements every time - we can maintain a running sum. When we add a number, we add it to the sum; when we remove a number, we subtract it. The mean is simply `sum // count`.

For the median, we need quick access to the middle element of a sorted sequence. A sorted list allows us to maintain sorted order during insertions and deletions (`O(log n)`), and gives us `O(1)` access to any element by index. The median is always at index `len(q) // 2` due to the problem's specific median definition.

For the mode, we need to track frequencies and quickly find the number with the highest frequency (breaking ties by smallest value). We use two complementary structures:
- A hash table (`cnt`) for `O(1)` frequency lookups and updates
- A sorted list (`sl2`) that maintains tuples of `(number, frequency)` sorted by frequency descending, then by number ascending

The clever part is how we update `sl2`: when a number's frequency changes, we first remove the old `(number, old_frequency)` tuple, then add the new `(number, new_frequency)` tuple. This keeps `sl2` always sorted correctly, and the mode is always the first element.

This design trades some space complexity (maintaining multiple structures) for optimal time complexity on each operation, which is crucial when these statistical queries are called frequently.

**Learn more about [Queue](/problems/queue_intro), [Binary Search](/problems/binary_search_intro), [Data Stream](/problems/median_of_data_stream) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

The implementation uses five main components to efficiently handle all operations:

**Data Structures:**
1. **[Queue](/problems/queue_intro) (`q`)**: A deque to maintain insertion order for FIFO removal
2. **Sum variable (`s`)**: Tracks the running sum of all numbers
3. **Hash table (`cnt`)**: Maps each number to its occurrence count
4. **Sorted list (`sl`)**: Maintains all numbers in sorted order for median calculation
5. **Sorted list with custom key (`sl2`)**: Stores `(number, frequency)` tuples, sorted by frequency descending then by number ascending for mode calculation

**Implementation Details:**

**`addNumber(number)`:**
- Append the number to [queue](/problems/queue_intro) `q` for maintaining insertion order
- Add the number to sorted list `sl` to maintain sorted order
- Update the frequency tracking in `sl2`:
  - First discard the old tuple `(number, cnt[number])` from `sl2`
  - Increment `cnt[number]` by 1
  - Add the new tuple `(number, cnt[number])` to `sl2`
- Add the number to running sum `s`
- Time complexity: `O(log n)` due to sorted list operations

**`removeFirstAddedNumber()`:**
- Pop the first element from [queue](/problems/queue_intro) `q` using `popleft()`
- Remove this number from sorted list `sl`
- Update the frequency tracking in `sl2`:
  - Discard the old tuple `(number, cnt[number])` from `sl2`
  - Decrement `cnt[number]` by 1
  - Add the updated tuple `(number, cnt[number])` to `sl2`
- Subtract the number from running sum `s`
- Time complexity: `O(log n)` due to sorted list operations

**`getMean()`:**
- Simply return `s // len(q)` (integer division for floored mean)
- Time complexity: `O(1)`

**`getMedian()`:**
- Access the element at index `len(q) // 2` in sorted list `sl`
- This works because for even-length arrays, `len(q) // 2` gives us the larger of the two middle elements
- Time complexity: `O(1)` in Python (direct index access in SortedList)

**`getMode()`:**
- Return the first element's number from `sl2[0][0]`
- Since `sl2` is sorted by frequency descending (using negative frequency in the key), the first element has the highest frequency
- When frequencies are tied, the sorting by number ascending ensures we get the smallest number
- Time complexity: `O(1)`

The key insight in the sorting key for `sl2` is using `lambda x: (-x[1], x[0])`, which sorts by negative frequency (highest first) and then by the number itself (smallest first) for tie-breaking.

## Example Walkthrough

Let's walk through a sequence of operations to see how the data structures work together:

**Initial State:**
- Queue `q = []`
- Sum `s = 0`
- Count map `cnt = {}`
- Sorted list `sl = []`
- Mode list `sl2 = []`

**Operation 1: `addNumber(5)`**
- Add 5 to queue: `q = [5]`
- Add 5 to sorted list: `sl = [5]`
- Update frequency tracking:
  - Old tuple (5, 0) doesn't exist in sl2, so no removal
  - Update cnt: `cnt = {5: 1}`
  - Add new tuple: `sl2 = [(5, 1)]` sorted by `(-1, 5)`
- Update sum: `s = 5`

**Operation 2: `addNumber(3)`**
- Add 3 to queue: `q = [5, 3]`
- Add 3 to sorted list: `sl = [3, 5]` (maintains sorted order)
- Update frequency tracking:
  - cnt becomes `{5: 1, 3: 1}`
  - Add (3, 1): `sl2 = [(3, 1), (5, 1)]` sorted by `(-1, 3)` and `(-1, 5)`
- Update sum: `s = 8`

**Operation 3: `addNumber(5)`**
- Add 5 to queue: `q = [5, 3, 5]`
- Add 5 to sorted list: `sl = [3, 5, 5]`
- Update frequency tracking:
  - Remove old (5, 1) from sl2
  - cnt becomes `{5: 2, 3: 1}`
  - Add new (5, 2): `sl2 = [(5, 2), (3, 1)]` sorted by `(-2, 5)` then `(-1, 3)`
- Update sum: `s = 13`

**Query: `getMean()`**
- Return `s // len(q) = 13 // 3 = 4`

**Query: `getMedian()`**
- Index = `len(q) // 2 = 3 // 2 = 1`
- Return `sl[1] = 5`

**Query: `getMode()`**
- Return `sl2[0][0] = 5` (highest frequency is 2)

**Operation 4: `removeFirstAddedNumber()`**
- Remove first element (5) from queue: `q = [3, 5]`
- Remove 5 from sorted list: `sl = [3, 5]`
- Update frequency tracking:
  - Remove old (5, 2) from sl2
  - cnt becomes `{5: 1, 3: 1}`
  - Add new (5, 1): `sl2 = [(3, 1), (5, 1)]` (both have frequency 1, sorted by value)
- Update sum: `s = 8`

**Query: `getMode()`**
- Return `sl2[0][0] = 3` (both have frequency 1, return smallest value)

This example demonstrates how:
- The queue maintains insertion order for FIFO removal
- The sorted list provides O(1) median access
- The frequency tracking with sl2 ensures the mode is always at index 0
- The running sum allows O(1) mean calculation

## Python Solution

```python
from collections import deque, defaultdict
from sortedcontainers import SortedList


class StatisticsTracker:
    def __init__(self):
        # Queue to maintain insertion order for FIFO removal
        self.queue = deque()

        # Running sum for mean calculation
        self.total_sum = 0

        # Dictionary to track frequency of each number
        self.frequency_map = defaultdict(int)

        # Sorted list for median calculation
        self.sorted_numbers = SortedList()

        # Sorted list for mode calculation
        # Key sorts by: highest frequency first (negative), then smallest number
        self.frequency_sorted = SortedList(key=lambda x: (-x[1], x[0]))

    def addNumber(self, number: int) -> None:
        """Add a number to the tracker and update all statistics"""
        # Add to queue for FIFO tracking
        self.queue.append(number)

        # Add to sorted list for median
        self.sorted_numbers.add(number)

        # Update frequency tracking for mode
        # Remove old frequency entry if exists
        self.frequency_sorted.discard((number, self.frequency_map[number]))
        # Increment frequency
        self.frequency_map[number] += 1
        # Add new frequency entry
        self.frequency_sorted.add((number, self.frequency_map[number]))

        # Update running sum
        self.total_sum += number

    def removeFirstAddedNumber(self) -> None:
        """Remove the oldest number (FIFO) and update all statistics"""
        # Remove oldest number from queue
        number = self.queue.popleft()

        # Remove from sorted list for median
        self.sorted_numbers.remove(number)

        # Update frequency tracking for mode
        # Remove old frequency entry
        self.frequency_sorted.discard((number, self.frequency_map[number]))
        # Decrement frequency
        self.frequency_map[number] -= 1
        # Add new frequency entry (even if frequency is 0)
        self.frequency_sorted.add((number, self.frequency_map[number]))

        # Update running sum
        self.total_sum -= number

    def getMean(self) -> int:
        """Return the mean (average) of all numbers, using integer division"""
        return self.total_sum // len(self.queue)

    def getMedian(self) -> int:
        """Return the median (middle value when sorted)"""
        # For even number of elements, returns the upper middle element
        return self.sorted_numbers[len(self.queue) // 2]

    def getMode(self) -> int:
        """Return the mode (most frequent number, smallest if tie)"""
        # First element has highest frequency due to sorting key
        return self.frequency_sorted[0][0]


# Your StatisticsTracker object will be instantiated and called as such:
# obj = StatisticsTracker()
# obj.addNumber(number)
# obj.removeFirstAddedNumber()
# param_3 = obj.getMean()
# param_4 = obj.getMedian()
# param_5 = obj.getMode()
```

## Time and Space Complexity

**Time Complexity:**
- `addNumber(number)`: `O(log n)` where `n` is the current number of elements
  - `deque.append()`: `O(1)`
  - `SortedList.add()`: `O(log n)`
  - `SortedList.discard()`: `O(log n)`
  - `SortedList.add()` for sl2: `O(log n)`
  - Dictionary operations and arithmetic: `O(1)`

- `removeFirstAddedNumber()`: `O(log n)` where `n` is the current number of elements
  - `deque.popleft()`: `O(1)`
  - `SortedList.remove()`: `O(log n)`
  - `SortedList.discard()`: `O(log n)`
  - `SortedList.add()` for sl2: `O(log n)`
  - Dictionary operations and arithmetic: `O(1)`

- `getMean()`: `O(1)`
  - Simple division operation

- `getMedian()`: `O(log n)`
  - Accessing element by index in SortedList: `O(log n)`

- `getMode()`: `O(log n)`
  - Accessing element by index in SortedList: `O(log n)`

**Space Complexity:** `O(n)` where `n` is the number of added numbers
- `deque`: stores all `n` numbers - `O(n)`
- `SortedList sl`: stores all `n` numbers - `O(n)`
- `defaultdict cnt`: stores at most `n` unique numbers and their counts - `O(n)`
- `SortedList sl2`: stores at most `n` unique (number, count) pairs - `O(n)`
- Variable `s`: single integer - `O(1)`

The overall space complexity is `O(n)` as all data structures grow linearly with the number of elements stored.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Median Calculation for Even-Length Arrays**
A frequent mistake is using `(len(q) - 1) // 2` or misunderstanding which middle element to return when there's an even count of numbers. The problem specifically asks for the **larger** of the two middle elements.

**Incorrect approach:**
```python
def getMedian(self) -> int:
    # Wrong: This returns the smaller middle element for even counts
    return self.sorted_numbers[(len(self.queue) - 1) // 2]
```

**Correct approach:**
```python
def getMedian(self) -> int:
    # Correct: len(q) // 2 gives the upper middle element
    # For n=4: indices [0,1,2,3], len//2 = 2 (correct)
    # For n=5: indices [0,1,2,3,4], len//2 = 2 (correct middle)
    return self.sorted_numbers[len(self.queue) // 2]
```

### 2. **Not Handling Zero Frequency in removeFirstAddedNumber()**
When removing a number, developers often forget that after decrementing, a number might have zero occurrences. Leaving entries with zero frequency in the frequency_sorted list can cause incorrect mode calculations.

**Problematic approach:**
```python
def removeFirstAddedNumber(self) -> None:
    number = self.queue.popleft()
    self.frequency_sorted.discard((number, self.frequency_map[number]))
    self.frequency_map[number] -= 1
    # Bug: Only add back if frequency > 0
    if self.frequency_map[number] > 0:
        self.frequency_sorted.add((number, self.frequency_map[number]))
```

**Better approach:**
Either add all entries (including zero frequency) and filter when getting mode, or properly clean up zero-frequency entries:
```python
def removeFirstAddedNumber(self) -> None:
    number = self.queue.popleft()
    self.frequency_sorted.discard((number, self.frequency_map[number]))
    self.frequency_map[number] -= 1

    if self.frequency_map[number] > 0:
        self.frequency_sorted.add((number, self.frequency_map[number]))
    else:
        # Clean up zero frequency from the map
        del self.frequency_map[number]
```

### 3. **Forgetting to Update frequency_sorted Before Changing Frequency**
The order of operations is crucial. You must remove the old (number, frequency) tuple from frequency_sorted **before** updating the frequency in frequency_map.

**Wrong order:**
```python
def addNumber(self, number: int) -> None:
    # Wrong: Incrementing before removing old tuple
    self.frequency_map[number] += 1
    # This won't find the tuple since frequency has changed!
    self.frequency_sorted.discard((number, self.frequency_map[number] - 1))
    self.frequency_sorted.add((number, self.frequency_map[number]))
```

**Correct order:**
```python
def addNumber(self, number: int) -> None:
    # Remove old tuple first (using current frequency)
    self.frequency_sorted.discard((number, self.frequency_map[number]))
    # Then update frequency
    self.frequency_map[number] += 1
    # Add new tuple with updated frequency
    self.frequency_sorted.add((number, self.frequency_map[number]))
```

### 4. **Using Regular Division Instead of Integer Division for getMean()**
The problem explicitly asks for the **floored** mean, which requires integer division.

**Incorrect:**
```python
def getMean(self) -> int:
    # Wrong: Returns float, not int
    return self.total_sum / len(self.queue)
```

**Correct:**
```python
def getMean(self) -> int:
    # Correct: Integer division floors the result
    return self.total_sum // len(self.queue)
```

### 5. **Incorrect Sorting Key for Mode Calculation**
The mode requires finding the most frequent number, with ties broken by selecting the smallest number. Using the wrong sorting key will produce incorrect results.

**Common mistakes:**
```python
# Wrong: Sorts by frequency ascending (smallest first)
self.frequency_sorted = SortedList(key=lambda x: (x[1], x[0]))

# Wrong: Doesn't handle tie-breaking correctly
self.frequency_sorted = SortedList(key=lambda x: -x[1])
```

**Correct approach:**
```python
# Correct: -x[1] for descending frequency, x[0] for ascending number
self.frequency_sorted = SortedList(key=lambda x: (-x[1], x[0]))
```

