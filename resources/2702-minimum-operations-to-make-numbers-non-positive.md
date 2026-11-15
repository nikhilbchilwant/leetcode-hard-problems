# 2702. Minimum Operations to Make Numbers Non-positive

> Source: https://algo.monster/liteproblems/2702

## Problem Statement

You have an array of integers `nums` (0-indexed) and two integer values `x` and `y`. You can perform operations on this array to reduce all numbers to zero or below.

In each operation, you:
1. Choose any index `i` from the array (where `0 <= i < nums.length`)
2. Decrease the value at `nums[i]` by `x`
3. Decrease the values at all other indices (every position except `i`) by `y`

Your goal is to find the minimum number of operations needed to make every integer in `nums` less than or equal to zero.

For example, if you have `nums = [10, 5, 8]`, `x = 4`, and `y = 1`, and you perform one operation choosing index 0:
- `nums[0]` decreases by 4: `10 - 4 = 6`
- `nums[1]` decreases by 1: `5 - 1 = 4`
- `nums[2]` decreases by 1: `8 - 1 = 7`
- Result after operation: `[6, 4, 7]`

You would continue performing operations until all values become `<= 0`. The problem asks for the minimum number of such operations required.

## Intuition

The key observation is that if we can make all numbers `<= 0` in `t` operations, then we can definitely do it in `t+1` operations or any number greater than `t`. This monotonic property suggests we can use [binary search](/problems/binary_search_intro) to find the minimum number of operations.

Let's think about what happens after `t` operations:
- Every number gets decreased by `y` in each operation (since it's not chosen), so each number decreases by at least `t * y`
- Some numbers might need additional reduction beyond `t * y` to become `<= 0`
- We can provide this extra reduction by choosing those specific indices during our `t` operations

For a number `nums[i]` to become `<= 0` after `t` operations:
- If `nums[i] <= t * y`, it's already handled (since every position gets reduced by `y` in each operation)
- If `nums[i] > t * y`, we need extra reduction of `nums[i] - t * y`
- Each time we choose index `i`, we get an extra reduction of `x - y` (since we reduce by `x` instead of `y`)
- So we need to choose index `i` at least `ceil((nums[i] - t * y) / (x - y))` times

The crucial insight: if the total number of times we need to choose specific indices (sum of all required selections) is `<= t`, then `t` operations are sufficient. Otherwise, we need more operations.

This gives us a way to check if a given number of operations `t` is feasible, which we can use with [binary search](/problems/binary_search_intro) to find the minimum valid `t`.

**Learn more about [Binary Search](/problems/binary_search_intro) patterns.**

## Solution Approach

We implement a [binary search](/problems/binary_search_intro) solution to find the minimum number of operations needed. The solution follows the standard binary search template for finding the first value where a condition becomes true.

**Feasible Function:**
The `feasible(t)` function determines if `t` operations are sufficient to make all numbers `<= 0`:

1. For each number `v` in `nums`:
   - After `t` operations, every number is reduced by at least `t * y`
   - If `v > t * y`, the remaining amount `v - t * y` must be eliminated by choosing this index
   - When we choose an index, we get extra reduction of `x - y` per operation
   - So we need `ceil((v - t * y) / (x - y))` operations where this index is chosen

2. Count the total number of times we need to choose specific indices:
   ```python
   cnt = 0
   for v in nums:
       if v > t * y:
           cnt += ceil((v - t * y) / (x - y))
   ```

3. If `cnt <= t`, then `t` operations are sufficient (return `True`), otherwise we need more operations (return `False`)

**[Binary Search](/problems/binary_search_intro) Template:**
```python
left, right = 0, max(nums)
first_true_index = -1

while left <= right:
    mid = (left + right) // 2
    if feasible(mid):
        first_true_index = mid
        right = mid - 1  # Try to find smaller valid value
    else:
        left = mid + 1

return first_true_index
```

The template tracks `first_true_index` to store the answer when the feasible condition is met. Since we're looking for the minimum operations where `feasible(t)` becomes true, we update `first_true_index = mid` when feasible and continue searching left for a potentially smaller value. The algorithm efficiently narrows down the search space by halving it in each iteration, resulting in `O(n * log(max(nums)))` time complexity.

## Example Walkthrough

Let's walk through a concrete example to illustrate the solution approach.

**Example:** `nums = [5, 3]`, `x = 3`, `y = 1`

**Step 1: Set up binary search with template**
- `left = 0` (minimum possible operations)
- `right = max(nums) = 5` (maximum operations we might need)
- `first_true_index = -1` (tracks our answer)

**Step 2: Binary search iterations using `while left <= right`**

**Iteration 1:** `left = 0, right = 5`
- `mid = (0 + 5) // 2 = 2`
- Check if `feasible(2)`:
  - For `nums[0] = 5`: remaining = `5 - 2*1 = 3`, need `ceil(3/2) = 2` selections
  - For `nums[1] = 3`: remaining = `3 - 2*1 = 1`, need `ceil(1/2) = 1` selection
  - Total selections needed: `2 + 1 = 3 > 2` → Not feasible
- `feasible(2) = False`, so `left = mid + 1 = 3`

**Iteration 2:** `left = 3, right = 5`
- `mid = (3 + 5) // 2 = 4`
- Check if `feasible(4)`:
  - For `nums[0] = 5`: remaining = `5 - 4*1 = 1`, need `ceil(1/2) = 1` selection
  - For `nums[1] = 3`: remaining = `3 - 4*1 = -1 <= 0`, need 0 selections
  - Total selections needed: `1 <= 4` → Feasible!
- `feasible(4) = True`, so `first_true_index = 4`, `right = mid - 1 = 3`

**Iteration 3:** `left = 3, right = 3`
- `mid = (3 + 3) // 2 = 3`
- Check if `feasible(3)`:
  - For `nums[0] = 5`: remaining = `5 - 3*1 = 2`, need `ceil(2/2) = 1` selection
  - For `nums[1] = 3`: remaining = `3 - 3*1 = 0 <= 0`, need 0 selections
  - Total selections needed: `1 <= 3` → Feasible!
- `feasible(3) = True`, so `first_true_index = 3`, `right = mid - 1 = 2`

**Iteration 4:** `left = 3, right = 2`
- `left > right`, loop terminates

**Step 3: Return result**
- `first_true_index = 3`, so the minimum number of operations is **3**

**Verification:**
- Operation 1: Choose index 0 → `[5-3, 3-1] = [2, 2]`
- Operation 2: Choose index 0 → `[2-3, 2-1] = [-1, 1]`
- Operation 3: Choose index 1 → `[-1-1, 1-3] = [-2, -2]`

All values are now `<= 0`, confirming that 3 operations are sufficient and minimal.

## Python Solution

```python
class Solution:
    def minOperations(self, nums: List[int], x: int, y: int) -> int:
        """
        Find minimum operations needed to reduce all elements to 0 or less.
        Uses binary search template to find first feasible number of operations.
        """

        def feasible(time: int) -> bool:
            """
            Check if all elements can be reduced to 0 or less within given time.
            Returns True if 'time' operations are sufficient, False otherwise.
            """
            targeted_operations_needed = 0

            for value in nums:
                # After 'time' operations, value becomes (value - time * y)
                remaining_value = value - time * y

                if remaining_value > 0:
                    # Each targeted operation reduces by (x - y) more than global
                    # Ceiling division: (a + b - 1) // b
                    operations_for_this_element = (remaining_value + (x - y) - 1) // (x - y)
                    targeted_operations_needed += operations_for_this_element

            return targeted_operations_needed <= time

        # Binary search template
        left, right = 0, max(nums)
        first_true_index = -1

        while left <= right:
            mid = (left + right) // 2

            if feasible(mid):
                first_true_index = mid
                right = mid - 1  # Try to find smaller valid value
            else:
                left = mid + 1

        return first_true_index
```

## Time and Space Complexity

**Time Complexity:** `O(n * log(max(nums)))`

The algorithm uses binary search on the range `[0, max(nums)]`, which requires `O(log(max(nums)))` iterations. In each iteration, the `check` function is called, which iterates through all `n` elements in the `nums` array to calculate the required operations. Each element requires `O(1)` time for the arithmetic operations and ceiling division. Therefore, the overall time complexity is `O(n * log(max(nums)))`.

**Space Complexity:** `O(1)`

The algorithm uses only a constant amount of extra space. The variables `l`, `r`, `mid`, `t`, `cnt`, and `v` are all scalar values that don't depend on the input size. The `check` function doesn't use any additional data structures that scale with input size. Therefore, the space complexity is `O(1)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Using Wrong Binary Search Template Variant**
A common mistake is using the `while left < right` with `right = mid` pattern instead of the standard template.

**Incorrect Implementation:**
```python
left, right = 0, max(nums)
while left < right:
    mid = (left + right) // 2
    if feasible(mid):
        right = mid
    else:
        left = mid + 1
return left
```

**Correct Implementation (Template-compliant):**
```python
left, right = 0, max(nums)
first_true_index = -1

while left <= right:
    mid = (left + right) // 2
    if feasible(mid):
        first_true_index = mid
        right = mid - 1
    else:
        left = mid + 1

return first_true_index
```

**Why the template matters:** The standard template explicitly tracks the answer with `first_true_index`, uses `while left <= right`, and `right = mid - 1` when feasible. This provides clearer semantics and handles edge cases more predictably.

### 2. **Integer Division vs Ceiling Division Error**
Another common mistake is incorrectly calculating the ceiling division when determining how many targeted operations are needed for each element.

**Incorrect Implementation:**
```python
operations_for_this_element = remaining_value // (x - y)  # Wrong! This is floor division
```

**Why it's wrong:** When we have a remaining value that isn't perfectly divisible by `(x - y)`, floor division will underestimate the operations needed. For example, if `remaining_value = 5` and `(x - y) = 2`, we need 3 operations (not 2).

**Correct Implementation:**
```python
# Method 1: Using math.ceil
import math
operations_for_this_element = math.ceil(remaining_value / (x - y))

# Method 2: Manual ceiling division (avoiding floating point)
operations_for_this_element = (remaining_value + (x - y) - 1) // (x - y)
```

### 3. **Incorrect Binary Search Boundary**
Setting the wrong upper bound for binary search can lead to incorrect results or unnecessary iterations.

**Incorrect Implementation:**
```python
right = sum(nums)  # Wrong! This could be unnecessarily large
```

**Why it's wrong:** The maximum operations needed is when we only use global operations (reducing by `y`), which would be `max(nums) // y + 1`. Using `sum(nums)` as the upper bound works but is inefficient.

**Correct Implementation:**
```python
right = max(nums)  # Sufficient upper bound
# Or more precisely:
right = (max(nums) + y - 1) // y  # Exact maximum operations needed
```

### 4. **Edge Case: When x equals y**
If `x == y`, the problem becomes trivial but the division by `(x - y)` would cause a division by zero error.

**Solution:**
```python
def minOperations(self, nums: List[int], x: int, y: int) -> int:
    if x == y:
        # All operations have the same effect
        return (max(nums) + x - 1) // x

    # Continue with binary search approach...
```

### 5. **Overflow in Ceiling Division Formula**
When using the manual ceiling division formula `(a + b - 1) // b`, the addition might cause integer overflow in languages with fixed integer sizes.

**Solution for Python:** Python handles arbitrary precision integers, so this isn't an issue. However, if implementing in other languages:
```python
# Alternative ceiling division without overflow risk
operations_for_this_element = remaining_value // (x - y)
if remaining_value % (x - y) != 0:
    operations_for_this_element += 1
```

