# 1526. Minimum Number of Increments on Subarrays to Form a Target Array

> Source: https://algo.monster/liteproblems/1526

## Problem Statement

You start with an array `initial` that has the same size as a given `target` array, where all elements in `initial` are zeros.

In each operation, you can:
- Select any contiguous subarray from `initial` 
- Increment every element in that subarray by 1

Your goal is to transform the `initial` array into the `target` array using the minimum number of operations.

For example, if `target = [1, 2, 3, 2, 1]`:
- Starting with `initial = [0, 0, 0, 0, 0]`
- You could select subarray from index 0 to 4 and increment: `[1, 1, 1, 1, 1]`
- Then select subarray from index 1 to 3 and increment: `[1, 2, 2, 2, 1]`
- Then select subarray from index 2 to 2 and increment: `[1, 2, 3, 2, 1]`
- Total operations: 3

The key insight is that when you look at consecutive elements in the target array:
- If `target[i] <= target[i-1]`, you don't need extra operations since you can reuse operations from previous elements
- If `target[i] > target[i-1]`, you need `target[i] - target[i-1]` additional operations to reach the higher value

## Intuition

Think of the target array as a mountain range where each value represents the height at that position. We need to build this mountain range from ground level (all zeros) using horizontal layers.

When we perform an operation on a subarray, we're essentially adding a horizontal layer of height 1 across that range. The key observation is that we can be [greedy](/problems/greedy_intro) and extend our operations as far as possible to minimize the total count.

Consider walking through the array from left to right:
- At the first position, we need exactly `target[0]` operations to reach that height
- Moving to the next position, two scenarios arise:
  - If `target[i] <= target[i-1]`: The height decreased or stayed the same. We can reuse the operations from the previous position - no new operations needed
  - If `target[i] > target[i-1]`: The height increased. We need `target[i] - target[i-1]` additional operations to build up from `target[i-1]` to `target[i]`

Visualize it like painting horizontal stripes on a wall. When the wall gets taller, you need extra stripes for the new section. When it gets shorter, the stripes you already painted cover that height.

For example, with `target = [3, 1, 2]`:
- Position 0: Need 3 operations to reach height 3
- Position 1: Height drops to 1, covered by existing operations
- Position 2: Height rises to 2, need 1 extra operation (2 - 1 = 1)
- Total: 3 + 0 + 1 = 4 operations

This [greedy](/problems/greedy_intro) approach works because extending an operation to cover more positions doesn't cost extra, so we always extend operations as far as the heights allow.

**Learn more about [Stack](/problems/stack_intro), [Greedy](/problems/greedy_intro), [Dynamic Programming](/problems/dynamic_programming_intro) and [Monotonic Stack](/problems/mono_stack_intro) patterns.**

## Solution Approach

The solution uses a [dynamic programming](/problems/dynamic_programming_intro) approach that can be optimized to use constant space.

**[Dynamic Programming](/problems/dynamic_programming_intro) Formulation:**

Define `f[i]` as the minimum number of operations needed to form `target[0..i]` from the initial array.

Base case: `f[0] = target[0]` - we need exactly `target[0]` operations to reach the first element's value.

Transition: For each position `i` from 1 to n-1:
- If `target[i] <= target[i-1]`: Then `f[i] = f[i-1]` (no additional operations needed)
- If `target[i] > target[i-1]`: Then `f[i] = f[i-1] + (target[i] - target[i-1])` (need extra operations for the increase)

**Space Optimization:**

Since `f[i]` only depends on `f[i-1]`, we don't need to store the entire array. We can maintain just the running sum of operations.

**Implementation:**

The code implements this elegantly in one line:
```python
return target[0] + sum(max(0, b - a) for a, b in pairwise(target))
```

Breaking it down:
1. `target[0]` - Initial operations for the first element
2. `pairwise(target)` - Creates pairs of consecutive elements `(target[i-1], target[i])`
3. `b - a` - Calculates the difference between consecutive elements
4. `max(0, b - a)` - Only counts positive differences (when height increases)
5. `sum(...)` - Adds up all the additional operations needed

The `max(0, b - a)` effectively implements our transition logic:
- When `b <= a` (height decreases or stays same): contributes 0
- When `b > a` (height increases): contributes `b - a`

**Time and Space Complexity:**
- Time: `O(n)` - single pass through the array
- Space: `O(1)` - only using variables for the sum calculation

## Example Walkthrough

Let's walk through a small example with `target = [2, 3, 1, 4, 2]` to illustrate the solution approach.

**Initial Setup:**
- Start with `initial = [0, 0, 0, 0, 0]`
- Need to transform it to `target = [2, 3, 1, 4, 2]`

**Step-by-step Analysis:**

Position 0 (target = 2):
- Starting from 0, need 2 operations to reach height 2
- Operations so far: **2**

Position 1 (target = 3):
- Previous height: 2, current height: 3
- Height increased by 1 (3 - 2 = 1)
- Need 1 additional operation
- Operations so far: 2 + 1 = **3**

Position 2 (target = 1):
- Previous height: 3, current height: 1
- Height decreased (1 - 3 = -2)
- No additional operations needed (we can reuse existing operations)
- Operations so far: 3 + 0 = **3**

Position 3 (target = 4):
- Previous height: 1, current height: 4
- Height increased by 3 (4 - 1 = 3)
- Need 3 additional operations
- Operations so far: 3 + 3 = **6**

Position 4 (target = 2):
- Previous height: 4, current height: 2
- Height decreased (2 - 4 = -2)
- No additional operations needed
- Operations so far: 6 + 0 = **6**

**Calculation using the formula:**
- Initial operations: `target[0] = 2`
- Consecutive differences:
  - (2, 3): max(0, 3-2) = 1
  - (3, 1): max(0, 1-3) = 0
  - (1, 4): max(0, 4-1) = 3
  - (4, 2): max(0, 2-4) = 0
- Total: 2 + 1 + 0 + 3 + 0 = **6 operations**

**Visual representation of operations:**
```
Operation 1: Select [0,4], increment → [1, 1, 1, 1, 1]
Operation 2: Select [0,4], increment → [2, 2, 2, 2, 2]
Operation 3: Select [1,1], increment → [2, 3, 2, 2, 2]
Operation 4: Select [3,3], increment → [2, 3, 2, 3, 2]
Operation 5: Select [3,3], increment → [2, 3, 2, 4, 2]
Operation 6: Select [3,3], increment → [2, 3, 2, 5, 2] (wait, this goes beyond)
```

Actually, let me correct the operations:
```
Operation 1: Select [0,1] and [3,4], increment → [1, 1, 0, 1, 1]
Operation 2: Select [0,1] and [3,4], increment → [2, 2, 0, 2, 2]
Operation 3: Select [1,1], increment → [2, 3, 0, 2, 2]
Operation 4: Select [3,3], increment → [2, 3, 0, 3, 2]
Operation 5: Select [3,3], increment → [2, 3, 0, 4, 2]
Operation 6: Select [2,2], increment → [2, 3, 1, 4, 2]
```

The key insight: we only need additional operations when the height increases from one position to the next.

## Python Solution

```python
class Solution:
    def minNumberOperations(self, target: List[int]) -> int:
        """
        Calculate minimum number of operations to build the target array.
      
        The key insight: we only need to add operations when the height increases
        from one element to the next. When height decreases, we can reuse 
        operations from the previous higher level.
      
        Args:
            target: List of positive integers representing target heights
          
        Returns:
            Minimum number of increment operations needed
        """
        # Start with the first element's height as initial operations needed
        total_operations = target[0]
      
        # For each consecutive pair, add the increase in height (if any)
        for i in range(1, len(target)):
            previous_height = target[i - 1]
            current_height = target[i]
          
            # Only add operations when height increases
            # If height decreases or stays same, no new operations needed
            height_increase = max(0, current_height - previous_height)
            total_operations += height_increase
      
        return total_operations
```

## Time and Space Complexity

**Time Complexity:** `O(n)` where `n` is the length of the target array.

The algorithm iterates through the array once using `pairwise(target)`, which generates consecutive pairs of elements. The `pairwise` function creates `n-1` pairs for an array of length `n`. For each pair `(a, b)`, we compute `max(0, b - a)` in constant time `O(1)`. The `sum()` function then accumulates these `n-1` values, resulting in a total time complexity of `O(n-1) = O(n)`.

**Space Complexity:** `O(1)` auxiliary space.

The `pairwise` function returns an iterator that generates pairs on-the-fly without creating a new list to store all pairs at once. The only additional space used is for:
- The iterator object itself: `O(1)`
- Variables to store intermediate calculations (`a`, `b`, and the running sum): `O(1)`

Therefore, the algorithm uses constant auxiliary space beyond the input array, giving us `O(1)` space complexity.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Misunderstanding the Problem as Requiring Individual Increments

**The Mistake:**
Many people initially think they need to track each individual increment operation separately, leading to overly complex solutions that try to simulate the actual building process step by step.

```python
# INCORRECT approach - trying to simulate each operation
def minNumberOperations_wrong(target):
    operations = 0
    current = [0] * len(target)
  
    while current != target:
        # Find a range to increment
        for i in range(len(target)):
            if current[i] < target[i]:
                # Increment this position and adjacent ones
                j = i
                while j < len(target) and current[j] < target[j]:
                    current[j] += 1
                    j += 1
                operations += 1
                break
  
    return operations
```

**Why It's Wrong:**
This approach is inefficient (O(n * max(target))) and unnecessarily complex. The problem doesn't require you to simulate the actual operations - just count how many are needed.

**The Fix:**
Recognize that you can calculate the minimum operations mathematically without simulation. Focus on the pattern: operations are only added when moving from a lower height to a higher height.

### Pitfall 2: Incorrectly Handling Decreasing Heights

**The Mistake:**
Some might think that when the height decreases, you need to subtract operations or handle it specially:

```python
# INCORRECT - wrong interpretation of decreasing heights
def minNumberOperations_wrong2(target):
    operations = target[0]
  
    for i in range(1, len(target)):
        # WRONG: thinking we need to handle decreases differently
        diff = target[i] - target[i-1]
        operations += abs(diff)  # This counts decreases as additions!
  
    return operations
```

**Why It's Wrong:**
When height decreases, we don't need ANY new operations. The operations that built the previous taller element already cover the current shorter element. Using `abs(diff)` would incorrectly add operations for decreases.

**The Fix:**
Only add the positive differences: `max(0, target[i] - target[i-1])`. This ensures decreases contribute 0 additional operations.

### Pitfall 3: Off-by-One Errors with Array Indexing

**The Mistake:**
Forgetting to handle the first element properly or iterating incorrectly:

```python
# INCORRECT - missing the first element
def minNumberOperations_wrong3(target):
    operations = 0  # WRONG: should start with target[0]
  
    for i in range(1, len(target)):
        operations += max(0, target[i] - target[i-1])
  
    return operations
```

**Why It's Wrong:**
The first element requires exactly `target[0]` operations to build from 0. Starting with `operations = 0` misses these initial operations.

**The Fix:**
Always initialize with `operations = target[0]` to account for building the first element from zero.

### Pitfall 4: Edge Case - Empty or Single Element Arrays

**The Mistake:**
Not handling edge cases properly:

```python
# INCORRECT - doesn't handle edge cases
def minNumberOperations_wrong4(target):
    # Crashes on empty array
    operations = target[0]
  
    # No check for single element
    for i in range(1, len(target)):
        operations += max(0, target[i] - target[i-1])
  
    return operations
```

**The Fix:**
Add proper edge case handling:

```python
def minNumberOperations_correct(target):
    if not target:
        return 0
  
    if len(target) == 1:
        return target[0]
  
    operations = target[0]
    for i in range(1, len(target)):
        operations += max(0, target[i] - target[i-1])
  
    return operations
```

