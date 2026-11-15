# 1944. Number of Visible People in a Queue

> Source: https://algo.monster/liteproblems/1944

## Problem Statement

You have `n` people standing in a queue, numbered from `0` to `n - 1` from left to right. Each person has a distinct height given in an array `heights`, where `heights[i]` represents the height of the `i`-th person.

A person can see another person to their right if everyone between them is shorter than both of them. Specifically, person `i` can see person `j` if:
- `i < j` (person `j` is to the right of person `i`)
- `min(heights[i], heights[j]) > max(heights[i+1], heights[i+2], ..., heights[j-1])` (all people between them are shorter than both person `i` and person `j`)

Your task is to return an array `answer` of length `n`, where `answer[i]` represents the number of people that the `i`-th person can see to their right in the queue.

For example, if we have heights `[10, 6, 8, 5, 11, 9]`:
- Person at index 0 (height 10) can see persons at indices 1 (height 6), 2 (height 8), and 4 (height 11), so `answer[0] = 3`
- Person at index 1 (height 6) can see persons at indices 2 (height 8) and 4 (height 11), so `answer[1] = 2`
- And so on for the remaining people

## Intuition

Let's think about what happens when person `i` looks to their right. They can see person `j` if everyone between them is shorter than both of them. This means person `i` will see a sequence of people with increasing heights until they encounter someone taller than themselves, at which point their view is blocked.

Consider this key observation: from person `i`'s perspective, the people they can see must form a strictly increasing sequence of heights. Why? Because if person `j` is visible to person `i`, and person `k` (where `k > j`) is also visible, then person `k` must be taller than person `j`. Otherwise, person `j` would block the view to person `k`.

Now, let's think about this problem from right to left. If we process people from right to left and maintain a [stack](/problems/stack_intro) of heights we've seen so far, we can efficiently determine how many people each person can see.

When we're at position `i`, we need to find:
1. All people to the right who are shorter than `heights[i]` - these are definitely visible
2. The first person who is taller than or equal to `heights[i]` - this person is also visible but blocks the view beyond

This naturally leads to a monotonic [stack](/problems/stack_intro) approach. As we traverse from right to left, we maintain a stack where heights increase from top to bottom. For each person at position `i`:
- We pop all elements from the stack that are smaller than `heights[i]` (these people are visible)
- If the stack still has elements after popping, the top element represents the first person taller than or equal to `heights[i]` (also visible but blocks further view)
- We then push `heights[i]` onto the stack for processing future positions

This way, the [stack](/problems/stack_intro) always maintains the "skyline" of people that could potentially be seen by people further to the left, with shorter people being removed as they get hidden behind taller ones.

**Learn more about [Stack](/problems/stack_intro) and [Monotonic Stack](/problems/mono_stack_intro) patterns.**

## Solution Approach

We implement the solution using a monotonic [stack](/problems/stack_intro) that processes the array from right to left. The stack maintains heights in increasing order from top to bottom, representing the "visible skyline" for people looking from the left.

Here's the step-by-step implementation:

1. **Initialize data structures:**
   - Create an answer array `ans` of size `n` initialized with zeros
   - Create an empty [stack](/problems/stack_intro) `stk` to maintain the monotonic property

2. **Traverse from right to left** (`i` from `n-1` to `0`):
 
   For each person at position `i`:
 
   a. **Count visible shorter people:**
   ```python
   while stk and stk[-1] < heights[i]:
       ans[i] += 1
       stk.pop()
   ```
   - Pop all elements from the [stack](/problems/stack_intro) that are smaller than `heights[i]`
   - Each popped element represents a person visible to person `i`
   - Increment `ans[i]` for each person popped
 
   b. **Check for blocking person:**
   ```python
   if stk:
       ans[i] += 1
   ```
   - If the stack is not empty after popping, the top element is >= `heights[i]`
   - This person is also visible but blocks the view beyond
   - Increment `ans[i]` by 1
 
   c. **Update the stack:**
   ```python
   stk.append(heights[i])
   ```
   - Push the current height onto the stack
   - This maintains the stack for processing people to the left

3. **Return the result:**
   - After processing all positions, return the `ans` array

**Example walkthrough** with `heights = [10, 6, 8, 5, 11, 9]`:

- `i=5` (height 9): [Stack](/problems/stack_intro) empty, `ans[5] = 0`, stack = `[9]`
- `i=4` (height 11): Pop 9 (smaller), `ans[4] = 1`, stack = `[11]`
- `i=3` (height 5): Stack has 11 (larger), `ans[3] = 1`, stack = `[11, 5]`
- `i=2` (height 8): Pop 5 (smaller), `ans[2] = 1`, then 11 remains (larger), `ans[2] = 2`, stack = `[11, 8]`
- `i=1` (height 6): Stack has 8 (larger), `ans[1] = 1`, then 11 remains, `ans[1] = 2`, stack = `[11, 8, 6]`
- `i=0` (height 10): Pop 6, 8 (smaller), `ans[0] = 2`, then 11 remains (larger), `ans[0] = 3`, stack = `[11, 10]`

Final answer: `[3, 2, 2, 1, 1, 0]`

The time complexity is `O(n)` as each element is pushed and popped at most once, and the space complexity is `O(n)` for the [stack](/problems/stack_intro).

## Example Walkthrough

Let's walk through a small example with `heights = [5, 3, 8, 3, 2, 5]` to illustrate the monotonic stack approach.

We'll process from right to left, maintaining a stack where heights increase from top to bottom.

**Initial state:** `ans = [0, 0, 0, 0, 0, 0]`, `stack = []`

**i = 5 (height = 5):**
- Stack is empty, no one to see
- `ans[5] = 0`
- Push 5 onto stack: `stack = [5]`

**i = 4 (height = 2):**
- Stack top (5) > 2, so person 4 can see person 5
- `ans[4] = 1` 
- Push 2 onto stack: `stack = [5, 2]`

**i = 3 (height = 3):**
- Pop 2 from stack (2 < 3), person 3 sees this person: `ans[3] = 1`
- Stack top (5) > 3, so person 3 also sees person at height 5: `ans[3] = 2`
- Push 3 onto stack: `stack = [5, 3]`

**i = 2 (height = 8):**
- Pop 3 from stack (3 < 8), person 2 sees this: `ans[2] = 1`
- Pop 5 from stack (5 < 8), person 2 sees this: `ans[2] = 2`
- Stack is now empty, no more people to see
- Push 8 onto stack: `stack = [8]`

**i = 1 (height = 3):**
- Stack top (8) > 3, so person 1 can see the person at height 8
- `ans[1] = 1`
- Push 3 onto stack: `stack = [8, 3]`

**i = 0 (height = 5):**
- Pop 3 from stack (3 < 5), person 0 sees this: `ans[0] = 1`
- Stack top (8) > 5, so person 0 also sees person at height 8: `ans[0] = 2`
- Push 5 onto stack: `stack = [8, 5]`

**Final result:** `ans = [2, 1, 2, 2, 1, 0]`

Let's verify person 0's view (height 5):
- Can see person 1 (height 3): ✓ (3 < 5)
- Can see person 2 (height 8): ✓ (min(5,8) = 5 > max(3) = 3)
- Cannot see beyond person 2 as 8 > 5 blocks the view

The stack maintains the "visible skyline" - at each step, it contains the people who could potentially be seen by someone further to the left, ordered by height.

## Python Solution

```python
class Solution:
    def canSeePersonsCount(self, heights: List[int]) -> List[int]:
        """
        Calculate how many people each person can see to their right.
        A person can see another person if all people between them are shorter.
      
        Args:
            heights: List of integers representing heights of people
          
        Returns:
            List where ans[i] represents number of people person i can see to their right
        """
        n = len(heights)
        result = [0] * n
      
        # Monotonic decreasing stack to track visible people from right
        # Stack stores heights of people that haven't been blocked yet
        stack = []
      
        # Process people from right to left
        for i in range(n - 1, -1, -1):
            current_height = heights[i]
          
            # Count and remove all people shorter than current person
            # These people are visible to current person but will be blocked for people to the left
            while stack and stack[-1] < current_height:
                result[i] += 1
                stack.pop()
          
            # If stack is not empty, current person can see one more person
            # (the first person taller than or equal to them)
            if stack:
                result[i] += 1
          
            # Add current person to stack for processing people to the left
            stack.append(current_height)
      
        return result
```

## Time and Space Complexity

**Time Complexity:** `O(n)`

The algorithm iterates through the array once from right to left (index `n-1` to `0`). For each element at position `i`:
- The while loop pops elements from the stack that are smaller than `heights[i]`
- Each element is pushed onto the stack exactly once and popped at most once

Since each element is processed at most twice (one push and one pop), the total number of operations across all iterations is `O(n)`. Even though there's a nested while loop, the amortized time complexity remains `O(n)` because each element can only be popped once throughout the entire execution.

**Space Complexity:** `O(n)`

The space complexity consists of:
- The answer array `ans` of size `n`: `O(n)`
- The stack `stk` which can contain at most `n` elements in the worst case (when heights are in increasing order from right to left): `O(n)`

Therefore, the total space complexity is `O(n) + O(n) = O(n)`, where `n` is the length of the array `heights`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Incorrectly Processing from Left to Right

**The Problem:**
A natural instinct is to iterate from left to right since we're looking for people each person can see to their right. However, this approach makes it difficult to efficiently determine visibility because we don't know what's ahead.

**Incorrect Approach:**
```python
def canSeePersonsCount(self, heights: List[int]) -> List[int]:
    n = len(heights)
    result = [0] * n
  
    for i in range(n):
        max_height_seen = 0
        for j in range(i + 1, n):
            if heights[j] > max_height_seen:
                result[i] += 1
                max_height_seen = heights[j]
            if heights[j] >= heights[i]:
                break
  
    return result
```

**Why It Fails:**
This O(n²) solution is inefficient and doesn't properly handle the visibility rules. It counts people based on increasing heights but misses the nuance that a person can see someone shorter if there's no taller person blocking them.

**The Fix:**
Process from right to left with a monotonic stack. This allows us to maintain information about future people while processing current positions.

### Pitfall 2: Misunderstanding Stack Monotonicity

**The Problem:**
Confusion about whether the stack should be monotonically increasing or decreasing, and from which direction (top to bottom or bottom to top).

**Incorrect Implementation:**
```python
# Wrong: Trying to maintain increasing stack from bottom to top
while stack and stack[-1] > current_height:  # Wrong comparison
    result[i] += 1
    stack.pop()
```

**Why It Fails:**
This reverses the logic and would count people that are actually blocked from view. The stack should contain people in a way that represents the "visible skyline" - we pop shorter people because they're visible but will be blocked for anyone further left.

**The Fix:**
The stack should be monotonically decreasing from bottom to top (increasing visibility). We pop elements that are **shorter** than the current person:
```python
while stack and stack[-1] < current_height:  # Correct comparison
    result[i] += 1
    stack.pop()
```

### Pitfall 3: Forgetting the Blocking Person

**The Problem:**
After popping all shorter people from the stack, forgetting to count the first person who is taller than or equal to the current person (if they exist).

**Incorrect Implementation:**
```python
def canSeePersonsCount(self, heights: List[int]) -> List[int]:
    n = len(heights)
    result = [0] * n
    stack = []
  
    for i in range(n - 1, -1, -1):
        # Only counting popped elements
        while stack and stack[-1] < heights[i]:
            result[i] += 1
            stack.pop()
        # Missing: if stack: result[i] += 1
        stack.append(heights[i])
  
    return result
```

**Why It Fails:**
This misses counting the person who blocks the view. For example, if person A (height 10) looks right and sees person B (height 8) and person C (height 12), the code would only count person B (popped from stack) but not person C (who remains in stack and blocks further view).

**The Fix:**
Always check if there's a remaining person in the stack after popping:
```python
while stack and stack[-1] < heights[i]:
    result[i] += 1
    stack.pop()

if stack:  # Don't forget this check!
    result[i] += 1
```

### Pitfall 4: Using Indices Instead of Heights in Stack

**The Problem:**
Storing indices in the stack instead of heights, which complicates the comparison logic.

**Incorrect Implementation:**
```python
stack = []  # Storing indices
for i in range(n - 1, -1, -1):
    while stack and heights[stack[-1]] < heights[i]:  # Extra indexing
        result[i] += 1
        stack.pop()
    if stack:
        result[i] += 1
    stack.append(i)  # Storing index instead of height
```

**Why It Fails:**
While this can work, it adds unnecessary complexity and potential for index errors. Since we only need height values for comparison, storing indices adds an extra layer of indirection.

**The Fix:**
Store heights directly in the stack for cleaner, more readable code:
```python
stack.append(heights[i])  # Store height directly
```

