# 1063. Number of Valid Subarrays

> Source: https://algo.monster/liteproblems/1063

## Problem Statement

You are given an integer array `nums`. Your task is to count how many non-empty subarrays have the property that their leftmost element is not larger than any other element in that subarray.

A subarray is a contiguous sequence of elements from the array. For example, if `nums = [1, 4, 2, 5, 3]`, then `[4, 2, 5]` is a subarray, but `[1, 2, 5]` is not (since the elements are not contiguous).

For a valid subarray, the first element (leftmost) must be the minimum element or tied for the minimum in that subarray. In other words, no element in the subarray should be strictly smaller than the first element.

For example:
- If `nums = [1, 4, 2, 5, 3]`
  - Subarray `[1]` is valid (single element)
  - Subarray `[1, 4]` is valid (1 ≤ 4)
  - Subarray `[1, 4, 2]` is valid (1 ≤ 2 and 1 ≤ 4)
  - Subarray `[4]` is valid (single element)
  - Subarray `[4, 2]` is NOT valid (4 > 2)
  - Subarray `[2, 5, 3]` is valid (2 ≤ 5 and 2 ≤ 3)

## Intuition

The key insight is that for each position `i` in the array, we need to find how many valid subarrays can start from that position. A subarray starting at position `i` is valid as long as `nums[i]` remains the minimum (or tied for minimum) element.

Think about it this way: if we start a subarray at position `i`, we can keep extending it to the right as long as we don't encounter an element smaller than `nums[i]`. The moment we hit a smaller element, any subarray that includes both `nums[i]` and that smaller element would be invalid.

For example, if `nums = [3, 1, 2, 4]` and we start at index 0 (value 3), we can form:
- `[3]` - valid (single element)
- `[3, 1]` - invalid because 3 > 1

So from index 0, we can only form 1 valid subarray. The element at index 1 (value 1) blocks us from extending further.

This leads us to the core idea: for each position `i`, we need to find the first position `j` to its right where `nums[j] < nums[i]`. Then, all subarrays starting at `i` and ending anywhere from `i` to `j-1` would be valid. That gives us exactly `j - i` valid subarrays starting at position `i`.

The challenge becomes efficiently finding this "blocking position" for each element. A monotonic [stack](/problems/stack_intro) is perfect for this - we can traverse the array from right to left, maintaining a stack of indices in increasing order of their values. For each position, we pop elements from the stack that are greater than or equal to the current element (they can't block us), and the remaining top element (if any) is our nearest smaller element to the right.

By precomputing these blocking positions for all indices, we can then simply sum up `(blocking_position - current_position)` for each position to get the total count of valid subarrays.

**Learn more about [Stack](/problems/stack_intro) and [Monotonic Stack](/problems/mono_stack_intro) patterns.**

## Solution Approach

The solution uses a monotonic [stack](/problems/stack_intro) to efficiently find, for each position, the nearest smaller element to its right.

**Step 1: Initialize data structures**
- Create a `right` array of size `n`, initialized with `n` (representing no blocking element found)
- Create an empty [stack](/problems/stack_intro) `stk` to maintain indices

**Step 2: Traverse from right to left**
We iterate through the array from the last index to the first (`n-1` to `0`). For each position `i`:

1. **Clean the [stack](/problems/stack_intro)**: While the stack is not empty and `nums[stk[-1]] >= nums[i]`, pop elements from the stack. These elements have values greater than or equal to `nums[i]`, so they cannot block subarrays starting at `i`.

2. **Find the blocking position**: After cleaning, if the [stack](/problems/stack_intro) is not empty, the top element `stk[-1]` represents the nearest position to the right of `i` where there's a smaller element. Set `right[i] = stk[-1]`.

3. **Add current index**: Push the current index `i` onto the [stack](/problems/stack_intro) for processing future elements.

**Step 3: Calculate the result**
For each position `i`, the number of valid subarrays starting at `i` is `right[i] - i`. Sum these values for all positions.

**Example walkthrough:**
Let's trace through `nums = [1, 4, 2, 5, 3]`:

- `i = 4` (value 3): [Stack](/problems/stack_intro) is empty, `right[4] = 5`, push 4. Stack: `[4]`
- `i = 3` (value 5): Pop 4 (since 3 < 5), stack empty, `right[3] = 5`, push 3. Stack: `[3]`
- `i = 2` (value 2): Stack has 3, `nums[3] = 5 >= 2`, pop it. Stack empty, `right[2] = 5`, push 2. Stack: `[2]`
- `i = 1` (value 4): Stack has 2, `nums[2] = 2 < 4`, so `right[1] = 2`, push 1. Stack: `[2, 1]`
- `i = 0` (value 1): Pop 1 (since 4 >= 1), pop 2 (since 2 >= 1). Stack empty, `right[0] = 5`, push 0. Stack: `[0]`

Final `right = [5, 2, 5, 5, 5]`

Count of valid subarrays: `(5-0) + (2-1) + (5-2) + (5-3) + (5-4) = 5 + 1 + 3 + 2 + 1 = 12`

The time complexity is `O(n)` since each element is pushed and popped at most once. The space complexity is `O(n)` for the `right` array and [stack](/problems/stack_intro).

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Example: `nums = [3, 1, 2, 4]`**

**Step 1: Build the `right` array using monotonic stack**

We traverse from right to left (indices 3 to 0), maintaining a stack of indices in increasing order of their values.

Initial setup:
- `right = [4, 4, 4, 4]` (initialized with n=4)
- `stk = []` (empty stack)

**Index 3 (value 4):**
- Stack is empty
- `right[3] = 4` (no smaller element to the right)
- Push 3 onto stack
- Stack: `[3]`

**Index 2 (value 2):**
- Check stack top: `nums[3] = 4 >= 2`, so pop 3
- Stack is now empty
- `right[2] = 4` (no smaller element to the right)
- Push 2 onto stack
- Stack: `[2]`

**Index 1 (value 1):**
- Check stack top: `nums[2] = 2 >= 1`, so pop 2
- Stack is now empty
- `right[1] = 4` (no smaller element to the right)
- Push 1 onto stack
- Stack: `[1]`

**Index 0 (value 3):**
- Check stack top: `nums[1] = 1 < 3`, so don't pop
- `right[0] = 1` (element at index 1 blocks us)
- Push 0 onto stack
- Stack: `[1, 0]`

Final `right` array: `[1, 4, 4, 4]`

**Step 2: Count valid subarrays**

For each index `i`, we can form `right[i] - i` valid subarrays starting at that position:

- Index 0: `right[0] - 0 = 1 - 0 = 1` valid subarray
  - `[3]` ✓ (stops before index 1 where value is 1 < 3)

- Index 1: `right[1] - 1 = 4 - 1 = 3` valid subarrays
  - `[1]` ✓
  - `[1, 2]` ✓ (1 ≤ 2)
  - `[1, 2, 4]` ✓ (1 ≤ 2 and 1 ≤ 4)

- Index 2: `right[2] - 2 = 4 - 2 = 2` valid subarrays
  - `[2]` ✓
  - `[2, 4]` ✓ (2 ≤ 4)

- Index 3: `right[3] - 3 = 4 - 3 = 1` valid subarray
  - `[4]` ✓

**Total count: 1 + 3 + 2 + 1 = 7 valid subarrays**

The key insight is that the monotonic stack efficiently identifies "blocking positions" - for each starting position, we know exactly where we must stop to keep all subarrays valid. This transforms the problem from checking every possible subarray to a simple calculation based on these precomputed boundaries.

## Python Solution

```python
class Solution:
    def validSubarrays(self, nums: List[int]) -> int:
        """
        Count the number of valid subarrays where each subarray's first element
        is the minimum element in that subarray.

        Args:
            nums: List of integers

        Returns:
            Total count of valid subarrays
        """
        n = len(nums)

        # Initialize array to store the next smaller element's index for each position
        # Default to n (beyond array bounds) if no smaller element exists
        next_smaller_index = [n] * n

        # Monotonic stack to track indices in increasing order of their values
        stack = []

        # Traverse array from right to left to find next smaller element
        for i in range(n - 1, -1, -1):
            # Pop elements from stack that are greater than or equal to current element
            # These cannot be the next smaller element for position i
            while stack and nums[stack[-1]] >= nums[i]:
                stack.pop()

            # If stack has elements, top element is the next smaller element's index
            if stack:
                next_smaller_index[i] = stack[-1]

            # Add current index to stack for processing future elements
            stack.append(i)

        # Calculate total valid subarrays
        # For each position i, valid subarrays are from i to (next_smaller_index[i] - 1)
        total_valid_subarrays = sum(j - i for i, j in enumerate(next_smaller_index))

        return total_valid_subarrays
```

## Time and Space Complexity

**Time Complexity:** `O(n)`

The algorithm iterates through the array once from right to left (index `n-1` to `0`). For each element at index `i`:
- The while loop pops elements from the stack. Although there's a nested loop structure, each element is pushed onto the stack exactly once and popped at most once throughout the entire execution. This amortized analysis gives us `O(1)` operations per element.
- The final sum operation iterates through the `right` array once, taking `O(n)` time.

Therefore, the overall time complexity is `O(n) + O(n) = O(n)`.

**Space Complexity:** `O(n)`

The algorithm uses:
- The `right` array of size `n` to store the next smaller element indices: `O(n)`
- The `stk` (stack) which can contain at most `n` elements in the worst case when the array is strictly increasing: `O(n)`

Therefore, the total space complexity is `O(n) + O(n) = O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Misunderstanding the Comparison Logic

**The Problem:** A common mistake is incorrectly handling the comparison when popping from the stack. Developers might use `nums[stack[-1]] > nums[i]` instead of `nums[stack[-1]] >= nums[i]`.

**Why it's Wrong:** Using strict inequality (`>`) would keep equal elements in the stack, which is incorrect. When we have equal elements, a subarray starting from an earlier position cannot extend past a later position with the same value, as the condition requires the leftmost element to be no larger than others.

**Example:** Consider `nums = [3, 3, 2]`
- With incorrect logic (`>`): The subarray `[3, 3]` would be counted as valid, but it shouldn't be since the first 3 is not smaller than or equal to all elements (it equals the second 3, but we need the leftmost to be the minimum or tied for minimum).
- With correct logic (`>=`): We properly identify that position 0 can only form valid subarray `[3]` before hitting the equal element at position 1.

**Solution:**
```python
# Correct - use >= to pop equal elements too
while stack and nums[stack[-1]] >= nums[i]:
    stack.pop()
```

### Pitfall 2: Stack Direction Confusion

**The Problem:** Developers might traverse the array from left to right instead of right to left, thinking they're building the "next smaller" array incorrectly.

**Why it's Wrong:** Traversing left to right would find the *previous* smaller element, not the *next* smaller element. We need to know where each subarray starting at position `i` must stop, which requires finding the next (rightward) position with a smaller value.

**Example:** For `nums = [4, 2, 5]`
- Left-to-right traversal would tell us that position 2 (value 5) has position 1 (value 2) as its previous smaller element, which doesn't help determine valid subarrays starting at position 2.
- Right-to-left traversal correctly identifies that position 0 (value 4) has position 1 (value 2) as its next smaller element, limiting valid subarrays starting at 0.

**Solution:**
```python
# Correct - traverse from right to left
for i in range(n - 1, -1, -1):
    # Process elements from right to left
```

### Pitfall 3: Incorrect Result Calculation

**The Problem:** Some might calculate the result as `sum(next_smaller_index)` or use `next_smaller_index[i] - i - 1` for counting.

**Why it's Wrong:**
- Just summing indices gives meaningless results
- Using `next_smaller_index[i] - i - 1` undercounts by excluding the subarray ending exactly at position `next_smaller_index[i] - 1`

**Example:** For position `i = 0` with `next_smaller_index[0] = 3`:
- Incorrect: `3 - 0 - 1 = 2` (missing one subarray)
- Correct: `3 - 0 = 3` (counts subarrays `[0:1]`, `[0:2]`, `[0:3]`)

**Solution:**
```python
# Correct - the number of valid subarrays starting at i
total_valid_subarrays = sum(j - i for i, j in enumerate(next_smaller_index))
```

