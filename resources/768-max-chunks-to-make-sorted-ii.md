# 768. Max Chunks To Make Sorted II

> Source: https://algo.monster/liteproblems/768

## Problem Statement

You are given an integer array `arr`.

The task is to split this array into some number of chunks (partitions), where each chunk is sorted individually. After [sorting](/problems/sorting_summary) each chunk separately and then concatenating them back together in their original order, the final result should equal the fully sorted version of the original array.

For example, if we have `arr = [2, 1, 3, 4]`, we could split it into chunks `[2, 1]` and `[3, 4]`. After [sorting](/problems/sorting_summary) each chunk individually, we get `[1, 2]` and `[3, 4]`. Concatenating them gives us `[1, 2, 3, 4]`, which equals the sorted original array.

The goal is to find the **maximum** number of chunks we can create while still satisfying the condition that [sorting](/problems/sorting_summary) each chunk individually and concatenating them produces the sorted array.

## Intuition

To understand how to maximize the number of chunks, let's think about what makes a valid chunk boundary. When we split the array into chunks and sort each chunk individually, the chunks must "fit together" properly after [sorting](/problems/sorting_summary) to form the complete sorted array.

The critical observation is: for any valid chunk, all elements in that chunk must end up in the same positions in the final sorted array as they would if we just sorted that chunk alone. This means the maximum value in a chunk cannot be greater than any value that comes after it in a different chunk.

Consider walking through the array from left to right. As we process each element, we need to decide whether it can start a new chunk or must belong to the previous chunk. 

If we encounter a value that is greater than or equal to the maximum value seen so far in the current chunk, it could potentially start a new chunk. Why? Because it won't violate the ordering when chunks are sorted and concatenated.

However, if we encounter a value that is smaller than the maximum of the current chunk, we have a problem. This smaller value needs to be sorted within a chunk that includes the previous maximum. In fact, it might even need to be merged with earlier chunks if it's smaller than their maximums too.

This naturally leads to a [stack](/problems/stack_intro)-based approach where:
- Each element in the stack represents the maximum value of a chunk
- When we see a new element that's greater than or equal to the stack top, it can form its own chunk
- When we see a smaller element, we need to merge chunks by popping from the stack until we find where this element belongs, but we preserve the maximum of the merged chunks

The number of elements remaining in the [stack](/problems/stack_intro) at the end represents the maximum number of chunks we can create.

**Learn more about [Stack](/problems/stack_intro), [Greedy](/problems/greedy_intro), [Sorting](/problems/sorting_summary) and [Monotonic Stack](/problems/mono_stack_intro) patterns.**

## Solution Approach

The solution uses a **monotonic [stack](/problems/stack_intro)** to track the maximum values of each chunk. The stack maintains the property that elements are in non-decreasing order from bottom to top.

Here's how the algorithm works step by step:

1. **Initialize an empty [stack](/problems/stack_intro)** `stk` to store the maximum values of chunks.

2. **Iterate through each element** `v` in the array:
 
   - **Case 1: Can form a new chunk** - If the [stack](/problems/stack_intro) is empty OR the current element `v` is greater than or equal to the top of the stack (`v >= stk[-1]`):
     - Simply push `v` onto the stack
     - This means `v` can be the maximum of a new chunk
 
   - **Case 2: Need to merge chunks** - If `v` is less than the stack top:
     - First, save the current maximum by popping it: `mx = stk.pop()`
     - Keep popping elements from the stack while the stack is not empty AND the top element is greater than `v` (`stk[-1] > v`)
     - This merges all chunks whose maximum values are greater than the current element
     - Finally, push back the saved maximum `mx` onto the stack
     - This represents the maximum of the newly merged chunk

3. **Return the [stack](/problems/stack_intro) size** - The number of elements in the stack represents the maximum number of chunks.

**Why this works:**
- The [stack](/problems/stack_intro) maintains maximum values of chunks in increasing order
- When we encounter a smaller value, it must belong to a chunk with all previous elements that have maximums greater than it
- By merging chunks (popping from stack) and keeping the overall maximum, we ensure the chunk boundaries are valid
- Each element in the final stack represents one valid chunk

**Time Complexity:** `O(n)` where `n` is the length of the array, as each element is pushed and popped at most once.

**Space Complexity:** `O(n)` in the worst case when the array is already sorted and each element forms its own chunk.

## Example Walkthrough

Let's walk through the algorithm with `arr = [2, 1, 3, 4, 3]`.

**Initial state:** `stk = []`

**Step 1: Process element 2**
- Stack is empty, so push 2
- `stk = [2]`
- This creates our first chunk with maximum 2

**Step 2: Process element 1**
- 1 < 2 (top of stack), so we need to merge
- Save max: `mx = 2` (pop from stack)
- Stack is now empty, so stop popping
- Push back `mx = 2`
- `stk = [2]`
- Elements [2, 1] form one chunk with maximum 2

**Step 3: Process element 3**
- 3 ≥ 2 (top of stack), so we can form a new chunk
- Push 3
- `stk = [2, 3]`
- Now we have two chunks: [2, 1] with max 2, and [3] with max 3

**Step 4: Process element 4**
- 4 ≥ 3 (top of stack), so we can form a new chunk
- Push 4
- `stk = [2, 3, 4]`
- Three chunks: [2, 1], [3], and [4]

**Step 5: Process element 3**
- 3 < 4 (top of stack), so we need to merge
- Save max: `mx = 4` (pop from stack)
- Now check: 3 ≥ 3 (new top), so stop popping
- Push back `mx = 4`
- `stk = [2, 3, 4]`
- The last two elements [4, 3] merge into one chunk with maximum 4

**Final result:** `stk = [2, 3, 4]` has 3 elements, so maximum chunks = 3

The chunks are: `[2, 1]`, `[3]`, `[4, 3]`
- Sorting individually: `[1, 2]`, `[3]`, `[3, 4]`
- Concatenating: `[1, 2, 3, 3, 4]` ✓ (equals the sorted original array)

## Python Solution

```python
class Solution:
    def maxChunksToSorted(self, arr: List[int]) -> int:
        # Stack to maintain the maximum value of each chunk
        stack = []
      
        for value in arr:
            # If stack is empty or current value is greater than or equal to 
            # the max of the last chunk, start a new chunk
            if not stack or value >= stack[-1]:
                stack.append(value)
            else:
                # Current value is smaller than the max of the last chunk
                # Need to merge chunks
              
                # Save the maximum value of the current chunk being merged
                max_value = stack.pop()
              
                # Pop all chunks whose max value is greater than current value
                # These chunks need to be merged because the current smaller value
                # would need to be sorted before their elements
                while stack and stack[-1] > value:
                    stack.pop()
              
                # Push back the maximum value of all merged chunks
                stack.append(max_value)
      
        # The number of elements in stack represents the number of chunks
        return len(stack)
```

## Time and Space Complexity

**Time Complexity:** `O(n)` where `n` is the length of the input array.

The algorithm iterates through each element in the array exactly once. For each element, we perform stack operations (push and pop). While there's a nested while loop that pops elements from the stack, each element can be pushed to and popped from the stack at most once throughout the entire execution. Therefore, the total number of operations across all iterations is bounded by `2n` (each element is pushed once and popped at most once), resulting in `O(n)` time complexity.

**Space Complexity:** `O(n)` in the worst case.

The space is used by the stack `stk`. In the worst-case scenario, when the input array is already sorted in ascending order (e.g., `[1, 2, 3, 4, 5]`), each element will be pushed to the stack and remain there, resulting in the stack containing all `n` elements. Therefore, the space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Incorrectly Handling Equal Values

**The Problem:**
A common mistake is using strict inequality (`>`) instead of `>=` when deciding whether to create a new chunk. This would incorrectly merge chunks when encountering equal values.

**Incorrect Implementation:**
```python
if not stack or value > stack[-1]:  # Wrong: using > instead of >=
    stack.append(value)
```

**Why It's Wrong:**
Consider `arr = [1, 1, 2]`. With strict inequality:
- Push 1 (stack: [1])
- Second 1 is not greater than stack top, so merge (stack: [1])
- Push 2 (stack: [1, 2])
- Result: 2 chunks

But the correct answer is 3 chunks: [1], [1], [2], since equal values can be in separate chunks as long as they maintain sorted order.

**Correct Approach:**
```python
if not stack or value >= stack[-1]:  # Correct: using >=
    stack.append(value)
```

### Pitfall 2: Forgetting to Save the Maximum Before Merging

**The Problem:**
When merging chunks, forgetting to save the maximum value of the chunks being merged leads to incorrect results.

**Incorrect Implementation:**
```python
else:
    # Forgot to save the maximum!
    while stack and stack[-1] > value:
        stack.pop()
    stack.append(value)  # Wrong: pushing current value instead of maximum
```

**Why It's Wrong:**
Consider `arr = [3, 1, 2]`:
- Push 3 (stack: [3])
- 1 < 3, need to merge, but if we push 1 instead of 3, stack becomes [1]
- 2 >= 1, so push 2 (stack: [1, 2])
- Result: 2 chunks, but the correct answer is 1

The issue is that we lose track of the actual maximum value in the merged chunk, which should be 3, not 1. Because the stack top is now 1, the value 2 is wrongly allowed to start a new chunk, even though 3 must come after it in sorted order.

**Correct Approach:**
```python
else:
    max_value = stack.pop()  # Save the maximum first
    while stack and stack[-1] > value:
        stack.pop()
    stack.append(max_value)  # Push back the saved maximum
```

### Pitfall 3: Over-Merging Chunks

**The Problem:**
Popping too many elements from the stack by using wrong comparison conditions during the merge process.

**Incorrect Implementation:**
```python
else:
    max_value = stack.pop()
    while stack and stack[-1] >= value:  # Wrong: using >= instead of >
        stack.pop()
    stack.append(max_value)
```

**Why It's Wrong:**
Consider `arr = [1, 2, 1, 3]`:
- Push 1 (stack: [1])
- Push 2 (stack: [1, 2])
- For the second 1: max_value = 2, and with `>=`, we'd pop the first 1 too (since 1 >= 1)
- This incorrectly merges all three elements into one chunk

The correct behavior should only merge chunks whose maximum is strictly greater than the current value, allowing chunks with equal maximums to remain separate.

**Correct Approach:**
```python
while stack and stack[-1] > value:  # Correct: using strict >
    stack.pop()
```

