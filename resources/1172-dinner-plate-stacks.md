# 1172. Dinner Plate Stacks

> Source: https://algo.monster/liteproblems/1172

## Problem Statement

You need to implement a system that manages dinner plates across multiple stacks. Imagine you have an infinite number of stacks arranged in a row, numbered from left to right starting at 0. Each [stack](/problems/stack_intro) can hold a maximum number of plates (the capacity).

The `DinnerPlates` class should support the following operations:

1. **`DinnerPlates(int capacity)`**: Initialize the system with a given maximum capacity for each [stack](/problems/stack_intro).

2. **`push(int val)`**: Add a plate with value `val` to the leftmost [stack](/problems/stack_intro) that isn't full. This means you should find the first stack (with the smallest index) that has room for another plate and add the plate there.

3. **`pop()`**: Remove and return the value from the top of the rightmost non-empty [stack](/problems/stack_intro). If all stacks are empty, return `-1`.

4. **`popAtStack(int index)`**: Remove and return the value from the top of the [stack](/problems/stack_intro) at the given `index`. If the stack at that index is empty or doesn't exist, return `-1`.

The key challenge is efficiently tracking which stacks are not full (for push operations) and managing the removal of empty stacks from the right side (for pop operations). The solution uses a sorted set to maintain indices of non-full stacks, allowing for quick identification of where to push new plates. When popping from the rightmost [stack](/problems/stack_intro), the implementation ensures that any trailing empty stacks are removed to maintain the correct "rightmost non-empty stack" for future pop operations.

## Intuition

The main challenge in this problem is efficiently finding the leftmost non-full [stack](/problems/stack_intro) for push operations and managing the rightmost non-empty stack for pop operations. Let's think through what data structures we need.

First, we need to store the actual stacks. A simple array of stacks would work since we're numbering them sequentially from 0.

For the `push` operation, we need to quickly find the leftmost [stack](/problems/stack_intro) that isn't full. If we had to scan through all stacks from left to right each time, it would be inefficient. Instead, we can maintain a collection of indices of all non-full stacks. But we also need this collection to be ordered so we can always get the smallest index quickly. This suggests using an ordered set (like `SortedSet` in Python).

When we push to a [stack](/problems/stack_intro), if it becomes full, we remove its index from our ordered set. When we pop from a stack, it becomes non-full, so we add its index back to the ordered set.

The tricky part comes with the `pop` operation. We need to pop from the rightmost non-empty [stack](/problems/stack_intro). A naive approach would be to always keep track of the rightmost non-empty stack index, but this gets complicated when stacks become empty.

The key insight is that we can simply try to pop from the last [stack](/problems/stack_intro) in our array. If successful, great! But if that stack becomes empty after popping, we need to clean up. We should remove all trailing empty stacks from our array to maintain the invariant that the last stack in our array is the rightmost stack we care about. This cleanup step is crucial - it ensures that for the next `pop` operation, we can again just look at the last stack in our array.

This approach elegantly handles both the efficient lookup requirements (using the ordered set for push) and the rightmost [stack](/problems/stack_intro) management (by maintaining a clean array without trailing empty stacks).

**Learn more about [Stack](/problems/stack_intro) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

Let's implement the solution using a [stack](/problems/stack_intro) array combined with an ordered set to efficiently manage our dinner plates system.

### Data Structures

We maintain three key components:
- `capacity`: The maximum capacity for each [stack](/problems/stack_intro)
- `stacks`: An array of stacks to store all plates
- `not_full`: A `SortedSet` to track indices of stacks that aren't at full capacity

### Implementation Details

**Initialization (`__init__`)**
- Store the capacity limit
- Initialize an empty array for stacks
- Create an empty `SortedSet` for tracking non-full [stack](/problems/stack_intro) indices

**Push Operation (`push`)**

When pushing a value:
1. Check if `not_full` is empty:
   - If empty, no existing stacks have room, so create a new [stack](/problems/stack_intro) with the value
   - Add the new stack to `stacks` array
   - If `capacity > 1`, add this new stack's index to `not_full` (since it can hold more plates)

2. If `not_full` has indices:
   - Get the smallest index from `not_full` (the leftmost non-full [stack](/problems/stack_intro))
   - Push the value to `stacks[index]`
   - If the stack reaches capacity, remove its index from `not_full`

**PopAtStack Operation (`popAtStack`)**

When popping from a specific [stack](/problems/stack_intro) at `index`:
1. Validate the index:
   - Return `-1` if `index` is out of bounds or `stacks[index]` is empty

2. Pop the value from `stacks[index]`

3. Handle the aftermath:
   - If this was the last [stack](/problems/stack_intro) (`index == len(stacks) - 1`) and it's now empty:
     - Clean up all trailing empty stacks
     - Remove their indices from `not_full`
     - Keep removing empty stacks from the end until we hit a non-empty stack or run out of stacks
   - Otherwise, add `index` to `not_full` since the stack now has room

4. Return the popped value

**Pop Operation (`pop`)**

Simply delegate to `popAtStack(len(stacks) - 1)` to pop from the rightmost [stack](/problems/stack_intro).

### Key Optimizations

The use of `SortedSet` for `not_full` allows:
- O(log n) insertion and removal of indices
- O(1) access to the minimum index (leftmost non-full [stack](/problems/stack_intro))

The cleanup of trailing empty stacks ensures:
- The last [stack](/problems/stack_intro) in our array is always the rightmost relevant stack
- We don't waste memory on empty stacks at the end
- The `pop()` operation remains simple

This design achieves efficient time complexity for all operations while maintaining clean state management throughout the lifetime of the data structure.

## Example Walkthrough

Let's walk through a concrete example with `capacity = 2` to see how the solution works:

**Initial State:**
- `capacity = 2`
- `stacks = []`
- `not_full = {}`

**Operation 1: `push(1)`**
- `not_full` is empty, so we create a new stack
- `stacks = [[1]]`
- Since this stack can hold one more plate (capacity is 2), add index 0 to `not_full`
- `not_full = {0}`

**Operation 2: `push(2)`**
- `not_full = {0}`, so we push to the leftmost non-full stack at index 0
- `stacks = [[1, 2]]`
- Stack 0 is now full (has 2 plates), remove it from `not_full`
- `not_full = {}`

**Operation 3: `push(3)`**
- `not_full` is empty, create a new stack at index 1
- `stacks = [[1, 2], [3]]`
- Stack 1 can hold one more plate, add it to `not_full`
- `not_full = {1}`

**Operation 4: `push(4)`**
- Push to stack at index 1 (the leftmost non-full)
- `stacks = [[1, 2], [3, 4]]`
- Stack 1 is now full, remove from `not_full`
- `not_full = {}`

**Operation 5: `push(5)`**
- Create new stack at index 2
- `stacks = [[1, 2], [3, 4], [5]]`
- `not_full = {2}`

**Operation 6: `popAtStack(0)`**
- Pop from stack 0: returns 2
- `stacks = [[1], [3, 4], [5]]`
- Stack 0 now has room, add to `not_full`
- `not_full = {0, 2}`

**Operation 7: `pop()`**
- Calls `popAtStack(2)` (rightmost stack)
- Pop from stack 2: returns 5
- `stacks = [[1], [3, 4], []]`
- Stack 2 is empty and it's the last stack, so clean up trailing empty stacks
- Remove stack 2: `stacks = [[1], [3, 4]]`
- Remove index 2 from `not_full`: `not_full = {0}`

**Operation 8: `push(6)`**
- Push to stack 0 (leftmost non-full)
- `stacks = [[1, 6], [3, 4]]`
- Stack 0 is full, remove from `not_full`
- `not_full = {}`

This example demonstrates:
- How `not_full` tracks which stacks have room (enabling efficient push to leftmost)
- How trailing empty stacks are cleaned up during pop operations
- How the rightmost stack is always at `len(stacks) - 1` due to cleanup

## Python Solution

```python
from sortedcontainers import SortedSet
from typing import List


class DinnerPlates:
    def __init__(self, capacity: int):
        """
        Initialize the DinnerPlates data structure.
      
        Args:
            capacity: Maximum number of values each stack can hold
        """
        self.capacity = capacity
        self.stacks: List[List[int]] = []  # List of stacks to store plates
        self.not_full = SortedSet()  # Sorted set of indices of non-full stacks
  
    def push(self, val: int) -> None:
        """
        Push a value onto the leftmost stack with available capacity.
      
        Args:
            val: The value to push onto a stack
        """
        if not self.not_full:
            # No non-full stacks exist, create a new stack
            self.stacks.append([val])
          
            # If the new stack still has room, add its index to not_full set
            if self.capacity > 1:
                self.not_full.add(len(self.stacks) - 1)
        else:
            # Push to the leftmost non-full stack
            index = self.not_full[0]
            self.stacks[index].append(val)
          
            # If stack is now full, remove it from not_full set
            if len(self.stacks[index]) == self.capacity:
                self.not_full.discard(index)
  
    def pop(self) -> int:
        """
        Pop a value from the rightmost non-empty stack.
      
        Returns:
            The popped value, or -1 if all stacks are empty
        """
        return self.popAtStack(len(self.stacks) - 1)
  
    def popAtStack(self, index: int) -> int:
        """
        Pop a value from a specific stack at the given index.
      
        Args:
            index: The index of the stack to pop from
          
        Returns:
            The popped value, or -1 if the operation is invalid
        """
        # Check if index is valid and stack at index is not empty
        if index < 0 or index >= len(self.stacks) or not self.stacks[index]:
            return -1
      
        # Pop the value from the specified stack
        val = self.stacks[index].pop()
      
        if index == len(self.stacks) - 1 and not self.stacks[-1]:
            # If we popped from the last stack and it's now empty,
            # remove all trailing empty stacks
            while self.stacks and not self.stacks[-1]:
                self.not_full.discard(len(self.stacks) - 1)
                self.stacks.pop()
        else:
            # Stack at index now has space, add to not_full set
            self.not_full.add(index)
      
        return val


# Your DinnerPlates object will be instantiated and called as such:
# obj = DinnerPlates(capacity)
# obj.push(val)
# param_2 = obj.pop()
# param_3 = obj.popAtStack(index)
```

## Time and Space Complexity

**Time Complexity:** `O(n × log n)` where `n` is the number of operations.

- **`push(val)`**: The operation involves checking and manipulating the `not_full` SortedSet. Adding to or accessing the first element of a SortedSet takes `O(log m)` time where `m` is the size of the set. In the worst case, `m` can be proportional to the number of stacks, which grows with the number of operations. Therefore, each push operation is `O(log n)`.

- **`pop()`**: This calls `popAtStack()` with the last index, so its complexity is the same as `popAtStack()`.

- **`popAtStack(index)`**: The main operations involve:
  - Popping from a list: `O(1)`
  - Adding to or discarding from the SortedSet: `O(log m)` where `m` is the size of the set
  - In the worst case when popping from the last stack, we might need to remove multiple empty stacks. Each removal involves a discard operation from the SortedSet (`O(log n)`) and a list pop (`O(1)`). While this cleanup can remove multiple stacks, amortized over all operations, each stack is removed at most once.

  The dominant operation is the SortedSet manipulation, giving `O(log n)` per operation.

Since we perform `n` operations total, and each operation has `O(log n)` complexity, the overall time complexity is `O(n × log n)`.

**Space Complexity:** `O(n)` where `n` is the number of operations.

- The `stacks` list can grow to store all pushed elements, which in the worst case is `O(n)`.
- The `not_full` SortedSet can contain at most one entry per stack, which is bounded by `O(n)`.
- Therefore, the total space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Not Cleaning Up Empty Stacks After `popAtStack`**

One of the most critical pitfalls is forgetting to remove trailing empty stacks when popping from the rightmost stack. This can lead to incorrect behavior in subsequent `pop()` operations.

**Problem Example:**
```python
# Without proper cleanup:
plates = DinnerPlates(2)
plates.push(1)  # stacks: [[1]]
plates.push(2)  # stacks: [[1, 2]]
plates.push(3)  # stacks: [[1, 2], [3]]
plates.popAtStack(1)  # Returns 3, stacks: [[1, 2], []]
plates.pop()  # Should pop from [1, 2], but might try to pop from empty stack []
```

**Solution:**
Always check if we're popping from the last stack and clean up any trailing empty stacks:
```python
if index == len(self.stacks) - 1 and not self.stacks[-1]:
    while self.stacks and not self.stacks[-1]:
        self.not_full.discard(len(self.stacks) - 1)
        self.stacks.pop()
```

### 2. **Incorrect Management of the `not_full` Set**

Failing to properly maintain the `not_full` set can cause the `push` operation to behave incorrectly.

**Common Mistakes:**
- Forgetting to add a stack's index to `not_full` after popping from it
- Not removing indices when deleting trailing empty stacks
- Adding index 0 to `not_full` when creating the first stack with capacity 1

**Solution:**
Ensure proper synchronization:
```python
# When creating a new stack:
if self.capacity > 1:  # Only add if stack can hold more than 1 plate
    self.not_full.add(len(self.stacks) - 1)

# When popping creates space:
if not (index == len(self.stacks) - 1 and not self.stacks[-1]):
    self.not_full.add(index)

# When removing trailing stacks:
self.not_full.discard(len(self.stacks) - 1)  # Remove before popping the stack
```

### 3. **Edge Case: Capacity of 1**

When capacity is 1, each stack can only hold one plate. A common mistake is not handling this special case properly when managing the `not_full` set.

**Problem:**
```python
plates = DinnerPlates(1)
plates.push(1)  # Creates stack [1]
# If we incorrectly add index 0 to not_full, next push might try to add to full stack
```

**Solution:**
Check capacity before adding to `not_full`:
```python
if self.capacity > 1:  # Stack has room for more plates
    self.not_full.add(len(self.stacks) - 1)
```

### 4. **Using Regular Set Instead of SortedSet**

Using a regular `set` instead of `SortedSet` breaks the requirement to push to the leftmost non-full stack.

**Problem:**
```python
# With regular set, iteration order is not guaranteed
not_full = {2, 0, 1}  # Regular set
index = min(not_full)  # O(n) operation each time
```

**Solution:**
Use `SortedSet` for O(1) access to minimum:
```python
from sortedcontainers import SortedSet
self.not_full = SortedSet()
index = self.not_full[0]  # O(1) access to minimum
```

### 5. **Not Validating Index Bounds in `popAtStack`**

Failing to check if the index is valid can cause runtime errors.

**Problem:**
```python
plates.popAtStack(10)  # When only 3 stacks exist
```

**Solution:**
Always validate before accessing:
```python
if index < 0 or index >= len(self.stacks) or not self.stacks[index]:
    return -1
```

