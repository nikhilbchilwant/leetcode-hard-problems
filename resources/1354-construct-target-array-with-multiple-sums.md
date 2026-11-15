# 1354. Construct Target Array With Multiple Sums

> Source: https://algo.monster/liteproblems/1354

## Problem Statement

You start with an array `arr` containing `n` ones: `[1, 1, 1, ..., 1]`. Your goal is to transform this array into a given `target` array of `n` integers.

The transformation follows these rules:
1. Calculate `x` as the sum of all elements currently in your array
2. Choose any index `i` (where `0 <= i < n`) and replace the value at that index with `x`
3. Repeat this process as many times as needed

You need to determine if it's possible to transform the initial array of ones into the `target` array using these operations. Return `true` if the transformation is possible, otherwise return `false`.

For example, if you start with `[1, 1, 1]`:
- Sum is 3, you could replace index 0 to get `[3, 1, 1]`
- Sum is 5, you could replace index 1 to get `[3, 5, 1]`
- Sum is 9, you could replace index 2 to get `[3, 5, 9]`

## Intuition

The key insight is that working forward from the initial array `[1, 1, 1, ...]` to the `target` array is difficult because at each step, we have multiple choices for which index to update, and it's unclear which choice leads to the target.

However, if we think about the problem in reverse, starting from the `target` array and working backwards to `[1, 1, 1, ...]`, the problem becomes much clearer. Why? Because in any array state, the largest element must have been the most recently created element. This is because when we replace an element with the sum of all elements, the new value is always larger than any individual element that existed before.

Consider how an element becomes the largest: if we have an array and perform the operation, the element we just updated becomes the sum of all elements, which is guaranteed to be the largest. So when looking at any array state, we can confidently say the largest element was produced in the most recent operation.

Working backwards, if the current largest element is `mx` and it was created by summing all elements, then before this operation:
- The element at this position had some smaller value
- The sum of all other elements was `t = current_sum - mx`
- The original value at this position was `mx - t`

We can repeatedly "undo" operations by:
1. Finding the largest element `mx`
2. Computing what it was before: `mx - t` where `t` is the sum of all other elements
3. Replacing `mx` with this previous value

If at any point we get invalid values (negative or zero), we know the transformation is impossible. We continue this reverse process until all elements become `1`, indicating we've successfully traced back to the initial state.

The modulo operation `mx % t` in the solution is an optimization - instead of repeatedly subtracting `t` from `mx` until we get a valid value, we can directly compute the remainder, which gives us the value before multiple operations were applied consecutively to the same index.

**Learn more about [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

The solution implements the reverse construction approach using a max heap (priority queue) to efficiently track and extract the largest element at each step.

**Data Structures Used:**
- **Max Heap**: Python's `heapq` provides a min heap by default, so we negate all values to simulate a max heap
- **Variable `s`**: Tracks the current sum of all elements in the array

**Implementation Steps:**

1. **Initialization**: 
   - Calculate the initial sum `s = sum(target)`
   - Create a max heap by negating all elements: `pq = [-x for x in target]`
   - Convert the list to a heap structure: `heapify(pq)`

2. **Main Loop** (`while -pq[0] > 1`):
   - Continue while the largest element is greater than 1
   - Extract the largest element: `mx = -heappop(pq)`
   - Calculate the sum of remaining elements: `t = s - mx`

3. **Validation Checks**:
   - If `t == 0`: This means all other elements sum to 0, which is impossible since we started with positive integers
   - If `mx - t < 1`: The previous value would be less than 1, which violates our starting condition
   - Return `False` if either condition is met

4. **Calculate Previous Value**:
   - Instead of repeatedly subtracting `t` from `mx`, use modulo: `x = mx % t`
   - Special case: If `mx % t == 0`, set `x = t` (this handles the case where `mx` is exactly divisible by `t`)
   - This optimization handles cases where the same index was updated multiple times consecutively

5. **Update State**:
   - Push the calculated previous value back: `heappush(pq, -x)`
   - Update the sum: `s = s - mx + x` (remove `mx`, add `x`)

6. **Termination**:
   - If the loop completes (all elements become 1), return `True`
   - The array has been successfully traced back to `[1, 1, 1, ...]`

**Time Complexity**: `O(n log n)` for initial heapification, and each operation involves heap operations which are `O(log n)`

**Space Complexity**: `O(n)` for storing the heap

The elegance of this solution lies in recognizing that the largest element uniquely identifies the last operation, allowing us to deterministically work backwards and verify if the transformation is possible.

## Example Walkthrough

Let's trace through the algorithm with `target = [9, 3, 5]`.

**Initial Setup:**
- Start with array of ones: `[1, 1, 1]`
- Can we reach `[9, 3, 5]`? Let's work backwards.
- Sum `s = 9 + 3 + 5 = 17`
- Max heap (negated): `[-9, -3, -5]`

**Step 1: Process largest element (9)**
- Extract max: `mx = 9`
- Sum of others: `t = s - mx = 17 - 9 = 8`
- Previous value: `mx % t = 9 % 8 = 1`
- Update array state: `[1, 3, 5]`
- Update sum: `s = 17 - 9 + 1 = 9`
- Heap becomes: `[-5, -3, -1]`

**Step 2: Process largest element (5)**
- Extract max: `mx = 5`
- Sum of others: `t = s - mx = 9 - 5 = 4`
- Previous value: `mx % t = 5 % 4 = 1`
- Update array state: `[1, 3, 1]`
- Update sum: `s = 9 - 5 + 1 = 5`
- Heap becomes: `[-3, -1, -1]`

**Step 3: Process largest element (3)**
- Extract max: `mx = 3`
- Sum of others: `t = s - mx = 5 - 3 = 2`
- Previous value: `mx % t = 3 % 2 = 1`
- Update array state: `[1, 1, 1]`
- Update sum: `s = 5 - 3 + 1 = 3`
- Heap becomes: `[-1, -1, -1]`

**Result:** All elements are now 1, so we return `true`.

**Forward Verification:**
Working forward from `[1, 1, 1]`:
1. Sum = 3, replace index 2: `[1, 1, 3]`
2. Sum = 5, replace index 1: `[1, 5, 3]` 
3. Sum = 9, replace index 1: `[1, 9, 3]`
4. Sum = 13, replace index 0: `[13, 9, 3]`
5. Sum = 25, replace index 0: `[25, 9, 3]`
6. Continue... eventually reaching `[9, 3, 5]` ✓

The backward approach efficiently validates that the transformation is possible without needing to explore all forward paths.

## Python Solution

```python
from typing import List
from heapq import heapify, heappop, heappush

class Solution:
    def isPossible(self, target: List[int]) -> bool:
        """
        Determine if we can transform an array of all 1s to the target array.
        In each operation, we can replace one element with the sum of all elements.
      
        Working backwards: the largest element must have been formed by summing all others.
        So we reverse the operation by subtracting the sum of others from the largest.
        """
        # Calculate total sum of all elements
        total_sum = sum(target)
      
        # Create max heap (negate values since Python has min heap by default)
        max_heap = [-num for num in target]
        heapify(max_heap)
      
        # Keep reducing the largest element until all elements become 1
        while -max_heap[0] > 1:
            # Get the current maximum element
            current_max = -heappop(max_heap)
          
            # Calculate sum of all other elements
            sum_of_others = total_sum - current_max
          
            # Edge cases where transformation is impossible
            if sum_of_others == 0 or current_max - sum_of_others < 1:
                return False
          
            # Calculate the previous value before this element became current_max
            # Use modulo for optimization when current_max >> sum_of_others
            # If remainder is 0, the previous value was sum_of_others
            previous_value = (current_max % sum_of_others) or sum_of_others
          
            # Push the previous value back to heap
            heappush(max_heap, -previous_value)
          
            # Update total sum
            total_sum = total_sum - current_max + previous_value
      
        # All elements are now 1, transformation is possible
        return True
```

## Time and Space Complexity

**Time Complexity:** `O(n log n + k log n)` where `n` is the length of the array `target` and `k` is the number of operations needed to reduce all elements to 1.

- Initial heapification takes `O(n)` time
- Each iteration involves:
  - `heappop()`: `O(log n)`
  - `heappush()`: `O(log n)`
- The number of iterations `k` depends on the values in the target array. In the worst case, when we have large values that need to be reduced step by step, `k` can be proportional to `log(max(target))` due to the modulo operation optimization
- For practical purposes and simplified analysis, this is often expressed as `O(n log n)` when considering typical constraints

**Space Complexity:** `O(n)`

- The priority queue `pq` stores all `n` elements from the target array, requiring `O(n)` space
- Additional variables (`s`, `mx`, `t`, `x`) use `O(1)` space
- Total space complexity is `O(n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Integer Overflow with Repeated Subtraction**

**The Pitfall:**
A naive implementation might use a while loop to repeatedly subtract `sum_of_others` from `current_max`:

```python
# WRONG - This causes Time Limit Exceeded!
while current_max > sum_of_others:
    current_max -= sum_of_others
previous_value = current_max
```

This approach fails catastrophically when `current_max` is significantly larger than `sum_of_others`. For example, with `target = [1, 1000000000]`, you'd need nearly a billion iterations!

**The Solution:**
Use the modulo operator for instant calculation:

```python
# CORRECT - O(1) operation
previous_value = current_max % sum_of_others
if previous_value == 0:
    previous_value = sum_of_others
```

### 2. **Incorrect Handling of Modulo Zero Case**

**The Pitfall:**
When `current_max % sum_of_others == 0`, using 0 as the previous value is incorrect:

```python
# WRONG - This will push 0 to the heap
previous_value = current_max % sum_of_others  # Could be 0!
heappush(max_heap, -previous_value)
```

This breaks the invariant that all values must be ≥ 1 since we started with an array of ones.

**The Solution:**
When the modulo result is 0, the previous value should be `sum_of_others`:

```python
# CORRECT - Handle the edge case
previous_value = (current_max % sum_of_others) or sum_of_others
```

### 3. **Missing Edge Case: Sum of Others is Zero**

**The Pitfall:**
Forgetting to check if `sum_of_others == 0` leads to division by zero error:

```python
# WRONG - Division by zero when sum_of_others = 0
previous_value = current_max % sum_of_others  # ZeroDivisionError!
```

**The Solution:**
Always validate before the modulo operation:

```python
# CORRECT - Check for zero first
if sum_of_others == 0:
    return False
previous_value = (current_max % sum_of_others) or sum_of_others
```

### 4. **Not Checking if Previous Value Would Be Less Than 1**

**The Pitfall:**
Even after modulo optimization, forgetting to verify that the calculated previous value is valid:

```python
# WRONG - Missing validation
previous_value = (current_max % sum_of_others) or sum_of_others
# What if current_max < sum_of_others + 1?
```

**The Solution:**
Add a validation check before proceeding:

```python
# CORRECT - Ensure we can reach a valid state
if current_max - sum_of_others < 1:
    return False
```

### 5. **Using Min Heap Instead of Max Heap**

**The Pitfall:**
Python's `heapq` is a min heap, so forgetting to negate values will process the smallest element instead of the largest:

```python
# WRONG - This processes minimum elements
max_heap = target.copy()
heapify(max_heap)
current_max = heappop(max_heap)  # Actually gets minimum!
```

**The Solution:**
Negate all values to simulate a max heap:

```python
# CORRECT - Negate to create max heap behavior
max_heap = [-num for num in target]
heapify(max_heap)
current_max = -heappop(max_heap)  # Gets maximum
```

