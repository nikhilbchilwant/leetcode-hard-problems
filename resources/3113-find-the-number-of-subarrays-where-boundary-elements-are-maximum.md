# 3113. Find the Number of Subarrays Where Boundary Elements Are Maximum

> Source: https://algo.monster/liteproblems/3113

## Problem Statement

You are given an array of positive integers `nums`.

Your task is to count the number of subarrays where the first element and the last element are both equal to the largest element in that subarray.

In other words, for each subarray you consider, you need to check if:
- The maximum value in the subarray appears at both the beginning and end positions
- This means the boundary elements (first and last) must be the same value, and this value must be the largest in the entire subarray

For example:
- A subarray `[3, 1, 2, 3]` would count if `3` is the maximum value in this subarray
- A subarray `[5]` of length 1 always counts since the first and last element are the same
- A subarray `[2, 3, 2]` would not count because `3` is the maximum but doesn't appear at both ends

Return the total count of all such valid subarrays.

## Intuition

Let's think about what makes a valid subarray. For a subarray to be valid, its first and last elements must be equal to the maximum element. This means:
1. The first and last elements must be the same value
2. No element in between can be larger than this boundary value

A key observation is that when we fix an element `x` as our boundary (first/last) element, any valid subarray ending at `x` must have all its elements less than or equal to `x`.

This leads us to think about processing the array from left to right. For each position, we want to know: how many valid subarrays can end at this position?

When we encounter an element `x`:
- Any subarray of length 1 (just `[x]` itself) is always valid
- For longer subarrays, we need `x` to also appear somewhere earlier, and all elements between those two occurrences of `x` must be `≤ x`

The challenge is tracking which previous occurrences of values can form valid subarrays with our current element. If we encounter a value larger than some previous value, that larger value "blocks" the previous value from forming valid subarrays that extend to future positions.

This blocking behavior suggests using a monotonic [stack](/problems/stack_intro). We maintain a decreasing stack where:
- Each entry tracks a value and how many valid subarrays can end with that value
- When we see a new element `x`, we pop all smaller elements (they're blocked by `x`)
- If we find an equal element in the stack, we can extend all its valid subarrays to include the current position
- We track the count of valid subarrays ending at each position

The monotonic decreasing property ensures that any value in our [stack](/problems/stack_intro) can potentially form valid subarrays with future equal values, as there's no larger "blocking" element between them.

**Learn more about [Stack](/problems/stack_intro), [Binary Search](/problems/binary_search_intro) and [Monotonic Stack](/problems/mono_stack_intro) patterns.**

## Solution Approach

We implement the solution using a monotonic decreasing [stack](/problems/stack_intro). Each element in the stack is a pair `[x, cnt]` where `x` is the value and `cnt` is the count of valid subarrays ending with `x` as the rightmost element.

Here's how the algorithm works:

1. **Initialize an empty [stack](/problems/stack_intro) and answer counter**
   - `stk = []` to maintain our [monotonic stack](/problems/mono_stack_intro)
   - `ans = 0` to accumulate the total count

2. **Process each element `x` in the array**

   For each element `x`, we perform three key operations:

   **a) Pop smaller elements:**
   ```python
   while stk and stk[-1][0] < x:
       stk.pop()
   ```
   We remove all elements from the [stack](/problems/stack_intro) that are smaller than `x`. These elements cannot form valid subarrays that extend beyond `x` because `x` would be larger than them, violating the condition that boundary elements must be the maximum.

   **b) Update or add the current element:**
   ```python
   if not stk or stk[-1][0] > x:
       stk.append([x, 1])
   else:
       stk[-1][1] += 1
   ```
   - If the [stack](/problems/stack_intro) is empty or the top element is greater than `x`, we push `[x, 1]` onto the stack. The count is `1` because at minimum, `[x]` itself forms a valid subarray.
   - If the top element equals `x`, we've found another occurrence of `x` that can extend all previous valid subarrays ending with `x`. We increment the count by `1` to include the new subarray formed between this `x` and the previous `x`.

   **c) Add to answer:**
   ```python
   ans += stk[-1][1]
   ```
   We add the count of valid subarrays ending at the current position to our answer.

3. **Return the total count**

The key insight is that the monotonic decreasing property of the [stack](/problems/stack_intro) ensures that:
- Any value in the stack can potentially form valid subarrays with future occurrences of the same value
- When we see value `x` again, all elements between the two occurrences of `x` are guaranteed to be `≤ x` (due to our popping mechanism)
- The count accumulates as we find more occurrences of the same value, representing all possible valid subarrays ending at each position

Time Complexity: `O(n)` where `n` is the length of the array, as each element is pushed and popped at most once.
Space Complexity: `O(n)` for the [stack](/problems/stack_intro) in the worst case when all elements are in decreasing order.

## Example Walkthrough

Let's walk through the solution with the array `nums = [3, 1, 3, 5, 1]`.

We'll track our monotonic decreasing stack and count valid subarrays at each step.

**Initial state:**
- Stack: `[]`
- Answer: `0`

**Step 1: Process element 3**
- Stack is empty, so push `[3, 1]` (value 3 with count 1)
- Stack: `[[3, 1]]`
- Add count to answer: `ans = 0 + 1 = 1`
- Valid subarrays so far: `[3]`

**Step 2: Process element 1**
- 1 < 3 (top of stack), so push `[1, 1]` as new element
- Stack: `[[3, 1], [1, 1]]`
- Add count to answer: `ans = 1 + 1 = 2`
- Valid subarrays ending here: `[1]`

**Step 3: Process element 3**
- Pop `[1, 1]` since 1 < 3 (removing blocked element)
- Stack after popping: `[[3, 1]]`
- Top element equals 3, so increment its count: `[[3, 2]]`
- Add count to answer: `ans = 2 + 2 = 4`
- Valid subarrays ending here: `[3]` and `[3,1,3]`

**Step 4: Process element 5**
- Pop `[3, 2]` since 3 < 5 (removing blocked element)
- Stack after popping: `[]`
- Stack is empty, so push `[5, 1]`
- Stack: `[[5, 1]]`
- Add count to answer: `ans = 4 + 1 = 5`
- Valid subarrays ending here: `[5]`

**Step 5: Process element 1**
- 1 < 5 (top of stack), so push `[1, 1]` as new element
- Stack: `[[5, 1], [1, 1]]`
- Add count to answer: `ans = 5 + 1 = 6`
- Valid subarrays ending here: `[1]`

**Final Answer: 6**

The valid subarrays are:
1. `[3]` at index 0
2. `[1]` at index 1
3. `[3,1,3]` from index 0 to 2
4. `[3]` at index 2
5. `[5]` at index 3
6. `[1]` at index 4

Notice how when we encountered the second `3`, we could form 2 valid subarrays: the single element `[3]` and the longer subarray `[3,1,3]`. The stack helped us track that we had seen `3` before and that all elements between were ≤ 3, making `[3,1,3]` valid.

## Python Solution

```python
class Solution:
    def numberOfSubarrays(self, nums: List[int]) -> int:
        # Stack to maintain elements in decreasing order
        # Each element is [value, count of subarrays ending at this position]
        stack = []
        total_subarrays = 0

        for current_num in nums:
            # Remove all elements from stack that are smaller than current number
            # This maintains the decreasing order property
            while stack and stack[-1][0] < current_num:
                stack.pop()

            # If stack is empty or top element is greater than current number,
            # add new element with count 1 (the subarray containing just itself)
            if not stack or stack[-1][0] > current_num:
                stack.append([current_num, 1])
            else:
                # If top element equals current number, increment its count
                # This counts additional subarrays ending at current position
                stack[-1][1] += 1

            # Add the count of valid subarrays ending at current position
            total_subarrays += stack[-1][1]

        return total_subarrays
```

## Time and Space Complexity

**Time Complexity:** `O(n)`

The algorithm iterates through the array once with a for loop that processes each element. For each element, it performs:
- A while loop that pops elements from the stack
- At most one append operation or one increment operation
- Adding to the answer counter

Although there's a nested while loop inside the for loop, each element can be pushed onto the stack at most once and popped at most once throughout the entire execution. This means the total number of operations across all iterations is bounded by `2n` (each element is pushed once and popped at most once), resulting in an amortized `O(1)` operation per element and overall `O(n)` time complexity.

**Space Complexity:** `O(n)`

The space complexity is determined by the stack `stk` which stores pairs of `[value, count]`. In the worst case scenario, when the array is strictly decreasing, no elements would be popped from the stack, and we would have one entry in the stack for each distinct element. Since we could have up to `n` distinct elements in the array, the stack could grow to size `n`, resulting in `O(n)` space complexity.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Misunderstanding the Count Accumulation

**The Problem:**
A common mistake is thinking that when we encounter a duplicate value, we should only add 1 to the count, rather than incrementing and then adding the total count. This misunderstanding leads to undercounting valid subarrays.

**Incorrect Implementation:**
```python
if not stack or stack[-1][0] > current_num:
    stack.append([current_num, 1])
    total_subarrays += 1  # Wrong: only adding 1
else:
    stack[-1][1] += 1
    total_subarrays += 1  # Wrong: only adding 1 instead of stack[-1][1]
```

**Why It's Wrong:**
When we see a value `x` for the second time, we're not just forming one new subarray. We're forming:
1. The subarray `[x]` by itself
2. The subarray from the previous `x` to current `x`

When we see it a third time, we form three subarrays ending at that position, and so on.

**Correct Approach:**
Always add `stack[-1][1]` to the answer, which represents all valid subarrays ending at the current position.

### Pitfall 2: Not Handling the Stack Pop Condition Correctly

**The Problem:**
Using `<=` instead of `<` in the while loop condition, which would incorrectly pop equal elements.

**Incorrect Implementation:**
```python
while stack and stack[-1][0] <= current_num:  # Wrong: using <= instead of <
    stack.pop()
```

**Why It's Wrong:**
If we pop elements that are equal to the current number, we lose the ability to track consecutive occurrences of the same value. For example, with array `[3, 3, 3]`:
- Processing first `3`: stack becomes `[[3, 1]]`
- Processing second `3`: if we use `<=`, we'd pop the first `3`, losing track of valid subarrays
- This would give us count 3 instead of the correct count 6

**Correct Approach:**
Only pop elements strictly smaller than the current number: `stack[-1][0] < current_num`

### Pitfall 3: Forgetting Edge Cases

**The Problem:**
Not properly handling single-element subarrays or assuming the stack will always have elements.

**Example Scenario:**
For array `[5, 4, 3, 2, 1]` (strictly decreasing), each element forms only one valid subarray (itself).

**Potential Bug:**
```python
# Forgetting to check if stack is empty before accessing stack[-1]
if stack[-1][0] > current_num:  # Error if stack is empty
    stack.append([current_num, 1])
```

**Correct Approach:**
Always check if the stack is not empty before accessing its elements:
```python
if not stack or stack[-1][0] > current_num:
    stack.append([current_num, 1])
```

