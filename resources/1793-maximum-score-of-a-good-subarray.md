# 1793. Maximum Score of a Good Subarray

> Source: https://algo.monster/liteproblems/1793

## Problem Statement

You have an array of integers `nums` (0-indexed) and an integer `k`.

The **score** of a subarray from index `i` to index `j` is calculated as: `min(nums[i], nums[i+1], ..., nums[j]) * (j - i + 1)`. In other words, the score equals the minimum element in the subarray multiplied by the length of the subarray.

A **good** subarray is one that contains the index `k`. This means for a subarray from index `i` to `j` to be good, it must satisfy `i ≤ k ≤ j`.

Your task is to find the maximum possible score among all good subarrays.

For example, if `nums = [1,4,3,7,4,5]` and `k = 3`, then:
- The subarray `[7]` (from index 3 to 3) has score `7 * 1 = 7`
- The subarray `[3,7,4]` (from index 2 to 4) has score `3 * 3 = 9`
- The subarray `[4,3,7,4,5]` (from index 1 to 5) has score `3 * 5 = 15`

All these subarrays contain index `k = 3`, making them good subarrays. You need to return the maximum score achievable.

## Intuition

The key insight is that for any subarray, its score is determined by the minimum element multiplied by the subarray length. This means that if we fix a particular element as the minimum value of a subarray, we want to extend that subarray as far as possible while maintaining that element as the minimum.

Think about it this way: if `nums[i]` is going to be the minimum element of our subarray, we should expand the subarray to the left and right as much as possible while `nums[i]` remains the smallest element. The farther we can extend, the larger the length becomes, and thus the larger the score.

For each element `nums[i]`, we need to find:
- How far left can we extend before encountering a smaller element?
- How far right can we extend before encountering a smaller element?

This naturally leads us to use a monotonic [stack](/problems/stack_intro) approach. For each position `i`, we can efficiently find:
- `left[i]`: the rightmost position to the left where `nums[left[i]] < nums[i]`
- `right[i]`: the leftmost position to the right where `nums[right[i]] < nums[i]`

Once we have these boundaries, the maximum subarray where `nums[i]` is the minimum would span from `left[i] + 1` to `right[i] - 1`, giving us a score of `nums[i] * (right[i] - left[i] - 1)`.

However, there's one crucial constraint: the subarray must be "good" - it must contain index `k`. So when considering `nums[i]` as the minimum, we can only count it if the subarray from `left[i] + 1` to `right[i] - 1` actually includes index `k`. This means we need `left[i] + 1 ≤ k ≤ right[i] - 1`.

By checking all possible minimum elements and their corresponding maximum subarrays (that include `k`), we can find the maximum score.

**Learn more about [Stack](/problems/stack_intro), [Two Pointers](/problems/two_pointers_intro), [Binary Search](/problems/binary_search_intro) and [Monotonic Stack](/problems/mono_stack_intro) patterns.**

## Solution Approach

The solution uses a **monotonic [stack](/problems/stack_intro)** to efficiently find the boundaries for each element when it serves as the minimum value of a subarray.

**Step 1: Find Left Boundaries**

We traverse the array from left to right with a monotonic increasing [stack](/problems/stack_intro):
- For each element `nums[i]`, we pop elements from the stack that are greater than or equal to `nums[i]`
- After popping, if the stack is not empty, the top element's index is `left[i]` - the rightmost position to the left where the element is smaller than `nums[i]`
- If the stack is empty, `left[i] = -1` (no smaller element to the left)
- Push the current index `i` onto the stack

```python
left = [-1] * n
stk = []
for i, v in enumerate(nums):
    while stk and nums[stk[-1]] >= v:
        stk.pop()
    if stk:
        left[i] = stk[-1]
    stk.append(i)
```

**Step 2: Find Right Boundaries**

We traverse the array from right to left with a similar approach:
- For each element `nums[i]`, we pop elements from the [stack](/problems/stack_intro) that are strictly greater than `nums[i]`
- After popping, if the stack is not empty, the top element's index is `right[i]` - the leftmost position to the right where the element is less than or equal to `nums[i]`
- If the stack is empty, `right[i] = n` (no smaller or equal element to the right)
- Push the current index `i` onto the stack

```python
right = [n] * n
stk = []
for i in range(n - 1, -1, -1):
    v = nums[i]
    while stk and nums[stk[-1]] > v:
        stk.pop()
    if stk:
        right[i] = stk[-1]
    stk.append(i)
```

Note the subtle difference: for the left boundary, we use `>=` to pop elements, while for the right boundary, we use `>`. This ensures we handle duplicate values correctly and avoid counting the same subarray multiple times.

**Step 3: Calculate Maximum Score**

For each element `nums[i]` as the potential minimum:
- The subarray where `nums[i]` is minimum spans from `left[i] + 1` to `right[i] - 1`
- Check if this subarray contains index `k`: `left[i] + 1 ≤ k ≤ right[i] - 1`
- If yes, calculate the score: `nums[i] * (right[i] - left[i] - 1)`
- Track the maximum score

```python
ans = 0
for i, v in enumerate(nums):
    if left[i] + 1 <= k <= right[i] - 1:
        ans = max(ans, v * (right[i] - left[i] - 1))
return ans
```

The time complexity is `O(n)` as each element is pushed and popped from the [stack](/problems/stack_intro) at most once. The space complexity is `O(n)` for storing the `left` and `right` arrays.

## Example Walkthrough

Let's walk through the solution with `nums = [1,4,3,7,4,5]` and `k = 3`.

**Step 1: Find Left Boundaries**

We traverse left to right with a monotonic increasing stack:

- i=0, v=1: Stack empty → left[0] = -1, push 0. Stack: [0]
- i=1, v=4: 1 < 4 → left[1] = 0, push 1. Stack: [0,1]
- i=2, v=3: Pop 1 (4≥3) → left[2] = 0, push 2. Stack: [0,2]
- i=3, v=7: 3 < 7 → left[3] = 2, push 3. Stack: [0,2,3]
- i=4, v=4: Pop 3 (7≥4) → left[4] = 2, push 4. Stack: [0,2,4]
- i=5, v=5: 4 < 5 → left[5] = 4, push 5. Stack: [0,2,4,5]

Result: `left = [-1, 0, 0, 2, 2, 4]`

**Step 2: Find Right Boundaries**

We traverse right to left with a monotonic stack:

- i=5, v=5: Stack empty → right[5] = 6, push 5. Stack: [5]
- i=4, v=4: Pop 5 (5>4) → right[4] = 6, push 4. Stack: [4]
- i=3, v=7: 4 ≤ 7 → right[3] = 4, push 3. Stack: [4,3]
- i=2, v=3: Pop 3 (7>3), Pop 4 (4>3) → right[2] = 6, push 2. Stack: [2]
- i=1, v=4: 3 ≤ 4 → right[1] = 2, push 1. Stack: [2,1]
- i=0, v=1: Pop 1 (4>1), Pop 2 (3>1) → right[0] = 6, push 0. Stack: [0]

Result: `right = [6, 2, 6, 4, 6, 6]`

**Step 3: Calculate Maximum Score**

For each element as minimum, check if subarray contains k=3:

- i=0: Subarray [0,5], contains k=3 ✓ → Score = 1 × 6 = 6
- i=1: Subarray [1,1], doesn't contain k=3 ✗
- i=2: Subarray [1,5], contains k=3 ✓ → Score = 3 × 5 = 15
- i=3: Subarray [3,3], contains k=3 ✓ → Score = 7 × 1 = 7
- i=4: Subarray [3,5], contains k=3 ✓ → Score = 4 × 3 = 12
- i=5: Subarray [5,5], doesn't contain k=3 ✗

Maximum score = 15 (when nums[2]=3 is the minimum of subarray [1,5])

## Python Solution

```python
class Solution:
    def maximumScore(self, nums: List[int], k: int) -> int:
        n = len(nums)

        # Arrays to store the nearest smaller element indices
        # left[i]: index of nearest smaller element to the left of i (-1 if none)
        # right[i]: index of nearest smaller element to the right of i (n if none)
        left_smaller = [-1] * n
        right_smaller = [n] * n

        # Monotonic stack to find nearest smaller element on the left
        stack = []
        for i, value in enumerate(nums):
            # Pop elements from stack that are greater than or equal to current value
            while stack and nums[stack[-1]] >= value:
                stack.pop()
            # If stack is not empty, top element is the nearest smaller on the left
            if stack:
                left_smaller[i] = stack[-1]
            stack.append(i)

        # Clear stack for right side processing
        stack = []

        # Monotonic stack to find nearest smaller element on the right
        for i in range(n - 1, -1, -1):
            value = nums[i]
            # Pop elements from stack that are greater than current value
            # Note: using > instead of >= to handle equal elements correctly
            while stack and nums[stack[-1]] > value:
                stack.pop()
            # If stack is not empty, top element is the nearest smaller on the right
            if stack:
                right_smaller[i] = stack[-1]
            stack.append(i)

        # Calculate maximum score
        max_score = 0
        for i, value in enumerate(nums):
            # Check if k is within the subarray where nums[i] is the minimum
            # The subarray spans from (left_smaller[i] + 1) to (right_smaller[i] - 1)
            if left_smaller[i] + 1 <= k <= right_smaller[i] - 1:
                # Calculate score: minimum value * subarray length
                subarray_length = right_smaller[i] - left_smaller[i] - 1
                score = value * subarray_length
                max_score = max(max_score, score)

        return max_score
```

## Time and Space Complexity

The time complexity is `O(n)`, where `n` is the length of the array `nums`. This is because:
- The first for loop iterates through all `n` elements to build the `left` array. Although there's a while loop inside, each element is pushed and popped from the stack at most once, resulting in `O(n)` operations total.
- The second for loop iterates through all `n` elements in reverse to build the `right` array. Similarly, each element is pushed and popped at most once, giving `O(n)` operations.
- The final for loop iterates through all `n` elements to calculate the maximum score, which takes `O(n)` time.
- Overall: `O(n) + O(n) + O(n) = O(n)`

The space complexity is `O(n)`, where `n` is the length of the array `nums`. This is because:
- The `left` array uses `O(n)` space
- The `right` array uses `O(n)` space
- The stack `stk` uses at most `O(n)` space in the worst case when all elements are in increasing order
- Overall: `O(n) + O(n) + O(n) = O(n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. Incorrect Handling of Duplicate Values

**The Pitfall:**
One of the most common mistakes is using the same comparison operator (`>=` or `>`) for both left and right boundary calculations. This can lead to:
- **Double counting**: The same subarray might be counted multiple times when there are duplicate minimum values
- **Missing subarrays**: Some valid subarrays might not be considered at all

For example, with `nums = [3, 3, 3]` and `k = 1`:
- If we use `>=` for both boundaries, multiple elements might claim ownership of the same subarray
- If we use `>` for both boundaries, we might miss valid subarrays entirely

**The Solution:**
Use asymmetric comparison operators:
- Left boundary: Use `>=` (pop elements greater than or equal)
- Right boundary: Use `>` (pop elements strictly greater)

This ensures each subarray is counted exactly once by assigning it to the leftmost occurrence of the minimum value.

### 2. Off-by-One Errors in Boundary Calculation

**The Pitfall:**
Confusing the boundary indices with the actual subarray indices. The common mistakes include:
- Using `left[i]` to `right[i]` as the subarray bounds directly
- Incorrectly calculating the subarray length as `right[i] - left[i]`
- Wrong condition check for whether `k` is in the subarray

**The Solution:**
Remember that:
- `left[i]` is the index of the element strictly smaller than `nums[i]` (not part of the subarray)
- `right[i]` is the index of the element smaller or equal to `nums[i]` (not part of the subarray)
- The actual subarray spans from `left[i] + 1` to `right[i] - 1`
- The length is `(right[i] - 1) - (left[i] + 1) + 1 = right[i] - left[i] - 1`

### 3. Not Checking if Subarray Contains k

**The Pitfall:**
Calculating the maximum score for all possible subarrays without verifying if they contain index `k`. This violates the "good subarray" requirement.

**The Solution:**
Always verify `left[i] + 1 <= k <= right[i] - 1` before considering the score of a subarray.

### 4. Stack Not Cleared Between Left and Right Processing

**The Pitfall:**
Reusing the same stack variable without clearing it between left and right boundary calculations, leading to incorrect boundary values.

**The Solution:**
Either:
- Explicitly clear the stack: `stack = []` or `stack.clear()`
- Use separate stack variables for left and right processing

