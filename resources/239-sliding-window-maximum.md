# 239. Sliding Window Maximum

> Source: https://algo.monster/liteproblems/239

## Problem Statement

You have an array of integers called `nums` and a sliding window of size `k` that moves from left to right across the array. At each position, you can only see the `k` numbers currently inside the window. The window moves one position to the right at a time.

Your task is to find the maximum value in the window at each position and return all these maximum values as an array.

For example, if `nums = [1, 3, -1, -3, 5, 3, 6, 7]` and `k = 3`:
- Window position `[1, 3, -1]` → maximum is `3`
- Window position `[3, -1, -3]` → maximum is `3`  
- Window position `[-1, -3, 5]` → maximum is `5`
- Window position `[-3, 5, 3]` → maximum is `5`
- Window position `[5, 3, 6]` → maximum is `6`
- Window position `[3, 6, 7]` → maximum is `7`

The result would be `[3, 3, 5, 5, 6, 7]`.

The solution uses a max-heap (priority queue) to efficiently track the maximum value in each window. The heap stores tuples of `(-value, index)` where the negative value ensures the heap behaves as a max-heap. As the window slides, elements are added to the heap, and outdated elements (those outside the current window based on their index) are removed from the top of the heap. The maximum value for each window position is then extracted from the heap's top element.

## Intuition

The naive approach would be to check all `k` elements in each window position to find the maximum, giving us `O(n*k)` time complexity. We need something more efficient.

The key insight is that we want to quickly access the maximum element as the window slides. This naturally points us toward using a data structure that maintains elements in sorted order - a heap (priority queue).

However, there's a challenge: as the window slides, some elements become invalid (they fall outside the window). We can't just keep all elements in the heap forever. We need a way to identify and remove outdated elements.

This is where storing both the value and its index becomes crucial. By pairing each value with its original index `(value, index)`, we can determine whether an element is still within the current window. If an element's index is less than or equal to `i - k` (where `i` is the current right boundary of the window), it's outside the window and should be discarded.

Since Python's `heapq` implements a min-heap by default, we use a clever trick: store negative values `(-value, index)` to simulate a max-heap. This way, the smallest negative number (which corresponds to the largest positive number) stays at the top.

The algorithm flow becomes clear:
1. Pre-fill the heap with the first `k-1` elements
2. For each new position, add the new element to the heap
3. Clean up outdated elements from the heap top
4. The heap top now contains the maximum value for the current window

This approach reduces our time complexity to `O(n log k)` since we perform heap operations for each element, and heap operations take `O(log k)` time.


### Watch it run

Step through it below. The heap keeps the largest value on top; stale indices are popped off only when they surface.

<iframe src="/widgets/sliding-window-max-heap.html" width="100%" height="580" frameborder="0" style="border:1px solid #e5e7eb;border-radius:12px;width:100%;min-height:160px;touch-action:pan-y;" title="Sliding Window Maximum walkthrough"></iframe>

## Solution Approach

The solution uses a max-heap implemented with Python's `heapq` module to efficiently track the maximum value in each sliding window.

**Step 1: Initialize the heap**
```python
q = [(-v, i) for i, v in enumerate(nums[: k - 1])]
heapify(q)
```
We create a list of tuples containing `(-value, index)` for the first `k-1` elements. The negative value trick converts Python's min-heap into a max-heap. We then heapify this list to create our initial heap structure.

**Step 2: Process each window position**
```python
ans = []
for i in range(k - 1, len(nums)):
```
We start from index `k-1` (where we have our first complete window of size `k`) and iterate through the rest of the array. The variable `i` represents the right boundary of our current window.

**Step 3: Add new element to heap**
```python
heappush(q, (-nums[i], i))
```
For each position, we push the current element into the heap as a tuple `(-nums[i], i)`. This ensures our heap always contains all potential maximum values.

**Step 4: Remove outdated elements**
```python
while q[0][1] <= i - k:
    heappop(q)
```
Before extracting the maximum, we check if the top element of the heap is still within our current window. If `q[0][1] <= i - k`, it means the element at the heap's top is outside the current window (its index is too far to the left). We keep popping such elements until we find one that's within the window.

**Step 5: Record the maximum**
```python
ans.append(-q[0][0])
```
After cleaning up outdated elements, the heap's top contains the maximum value for the current window. We negate it back to get the original positive value and add it to our result array.

The algorithm maintains the invariant that after cleanup, the heap's top always contains the maximum value within the current window of size `k`. The time complexity is `O(n log n)` where `n` is the array length: each element is pushed and popped at most once, but outdated elements are removed only when they reach the top, so the heap can hold up to `n` entries and each operation costs `O(log n)`.

## Example Walkthrough

<details>
<summary>Prefer text? Read the step-by-step walkthrough.</summary>

Let's walk through a small example with `nums = [1, 3, -1, -3, 5]` and `k = 3`.

**Initial Setup:**
- First, we create a heap with the first `k-1 = 2` elements
- `q = [(-1, 0), (-3, 1)]` → After heapify: `[(-3, 1), (-1, 0)]`
- The heap is organized as a min-heap, so `(-3, 1)` is at the top (representing value 3)

**Window 1: [1, 3, -1]** (i = 2)
- Add `(-(-1), 2)` = `(1, 2)` to heap
- Heap becomes: `[(-3, 1), (-1, 0), (1, 2)]`
- Check top element: index 1 > 2 - 3 = -1, so it's valid (within window)
- Maximum = -(-3) = 3
- Result: [3]

**Window 2: [3, -1, -3]** (i = 3)
- Add `(-(-3), 3)` = `(3, 3)` to heap
- Heap becomes: `[(-3, 1), (-1, 0), (1, 2), (3, 3)]`
- Check top element: index 1 > 3 - 3 = 0, so it's valid
- Maximum = -(-3) = 3
- Result: [3, 3]

**Window 3: [-1, -3, 5]** (i = 4)
- Add `(-5, 4)` to heap
- Heap becomes: `[(-5, 4), (-3, 1), (1, 2), (3, 3), (-1, 0)]`
- Check top element: index 4 > 4 - 3 = 1, so it's valid
- Maximum = -(-5) = 5
- Result: [3, 3, 5]

The key insight is that we always maintain elements in the heap sorted by value (through the negative trick), and we lazily remove outdated elements only when they appear at the top. This ensures we always have quick access to the maximum valid element in O(log k) time.


</details>

## Python Solution

```python
from typing import List
from heapq import heapify, heappush, heappop

class Solution:
    def maxSlidingWindow(self, nums: List[int], k: int) -> List[int]:
        # Initialize max heap with first k-1 elements
        # Use negative values to simulate max heap (Python has min heap by default)
        # Store tuples of (negative_value, index) to track element positions
        max_heap = [(-value, index) for index, value in enumerate(nums[:k - 1])]
        heapify(max_heap)
      
        # Result list to store maximum values of each window
        result = []
      
        # Slide the window from position k-1 to the end
        for i in range(k - 1, len(nums)):
            # Add current element to the heap
            heappush(max_heap, (-nums[i], i))
          
            # Remove elements that are outside the current window
            # The window's valid range is [i - k + 1, i]
            while max_heap[0][1] <= i - k:
                heappop(max_heap)
          
            # The maximum element is at the top of the heap
            # Convert back to positive value and add to result
            result.append(-max_heap[0][0])
      
        return result
```

## Time and Space Complexity

**Time Complexity:** `O(n × log n)`

The algorithm uses a max heap to track the maximum element in each sliding window. Breaking down the operations:
- Initial heap construction with `k-1` elements takes `O(k)` time using `heapify`
- The main loop runs `n - k + 1` times (from index `k-1` to `n-1`)
- For each iteration:
  - `heappush` operation: `O(log m)` where `m` is the heap size
  - While loop for removing outdated elements: Each element is removed at most once throughout the entire execution
  - `heappop` operations: `O(log m)` per removed element

Outdated elements are removed lazily, only when they reach the top. An outdated element that is smaller than the current maximum stays buried in the heap. For an increasing array such as `[1, 2, 3, ..., n]`, every new element becomes the top, nothing is ever popped, and the heap grows to `n` entries. So `m` can be as large as `n`, and with `n` pushes and at most `n` pops the total is `O(n log n)`.

**Space Complexity:** `O(n)`

The heap `q` stores the current window plus any outdated elements that have not reached the top yet. In the increasing-array case above it holds all `n` elements. The answer list `ans` is not counted as auxiliary space since it's the required output. A monotonic deque solves the problem in `O(n)` time and `O(k)` space, because it removes outdated and dominated elements immediately.

## Common Pitfalls

### Pitfall 1: Not Properly Handling Elements Outside the Window

**The Problem:**
A common mistake is to only check once if the top element is outside the window, rather than using a while loop. This can leave outdated elements at the top of the heap.

**Incorrect Implementation:**
```python
# Wrong: Only checks once
if max_heap[0][1] <= i - k:
    heappop(max_heap)
result.append(-max_heap[0][0])
```

**Why It Fails:**
Multiple consecutive elements at the top of the heap might be outside the current window. For example, if the heap contains `[(-10, 0), (-9, 1), (-8, 5)]` and the current window starts at index 4, both elements with indices 0 and 1 need to be removed.

**Correct Solution:**
```python
# Correct: Uses while loop to remove ALL outdated elements
while max_heap[0][1] <= i - k:
    heappop(max_heap)
result.append(-max_heap[0][0])
```

### Pitfall 2: Incorrect Window Boundary Calculation

**The Problem:**
Misunderstanding the window boundaries leads to incorrect index comparisons when checking if an element is within the window.

**Common Mistakes:**
```python
# Wrong: Off-by-one error
while max_heap[0][1] < i - k:  # Should be <=
    heappop(max_heap)

# Wrong: Incorrect window start calculation
while max_heap[0][1] < i - k + 1:  # Overly complex and error-prone
    heappop(max_heap)
```

**Why It Matters:**
For a window ending at index `i` with size `k`, the valid indices are `[i - k + 1, i]`. An element at index `j` is outside the window if `j <= i - k` (equivalently, `j < i - k + 1`).

**Correct Solution:**
Use the clearer condition `max_heap[0][1] <= i - k` to identify elements outside the window.

### Pitfall 3: Forgetting to Store Index Information

**The Problem:**
Storing only values in the heap without their indices makes it impossible to determine which elements are still within the current window.

**Incorrect Implementation:**
```python
# Wrong: Only stores values
max_heap = [-v for v in nums[:k-1]]
heapify(max_heap)
for i in range(k-1, len(nums)):
    heappush(max_heap, -nums[i])
    # Cannot determine which elements to remove!
    result.append(-max_heap[0])
```

**Correct Solution:**
Always store tuples of `(negative_value, index)` to track both the value and position of each element:
```python
max_heap = [(-value, index) for index, value in enumerate(nums[:k - 1])]
```

### Pitfall 4: Initializing with Full Window Instead of k-1 Elements

**The Problem:**
Initializing the heap with all `k` elements before starting the main loop complicates the logic and can lead to off-by-one errors.

**Problematic Approach:**
```python
# Confusing: Starting with full window
max_heap = [(-v, i) for i, v in enumerate(nums[:k])]
heapify(max_heap)
result = [-max_heap[0][0]]  # Need special handling for first window
for i in range(k, len(nums)):
    # Complex index management
    ...
```

**Better Solution:**
Initialize with `k-1` elements and let the main loop handle all windows uniformly:
```python
max_heap = [(-v, i) for i, v in enumerate(nums[:k - 1])]
heapify(max_heap)
for i in range(k - 1, len(nums)):
    heappush(max_heap, (-nums[i], i))
    # Clean and consistent logic for all windows
    ...
```

