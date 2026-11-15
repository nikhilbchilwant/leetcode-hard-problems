# 3229. Minimum Operations to Make Array Equal to Target

> Source: https://algo.monster/liteproblems/3229

## Problem Statement

You have two positive integer arrays `nums` and `target` that have the same length.

In one operation, you can:
- Choose any contiguous subarray from `nums`
- Either add 1 to every element in that subarray, OR subtract 1 from every element in that subarray

Your goal is to transform `nums` into `target` using the minimum number of operations.

For example, if `nums = [1, 3, 2]` and `target = [2, 4, 5]`, you need to:
- Increase the first element by 1 (from 1 to 2)
- Increase the second element by 1 (from 3 to 4)  
- Increase the third element by 3 (from 2 to 5)

You could perform these operations separately, but you can optimize by selecting subarrays strategically. For instance, you could select the entire array and increment by 1, then select just the third element and increment by 2 more times.

## Intuition

Let's first think about what we're actually doing. We need to transform each `nums[i]` to `target[i]`. The difference at each position is `diff[i] = target[i] - nums[i]`. This tells us exactly how much we need to change each element.

Now, imagine we have a difference array like `[2, 3, 1, -2, -1, 3]`. We need to apply operations to make all these differences become 0.

The key observation is that when we perform an operation on a subarray, we're essentially reducing the absolute values of all differences in that range by 1 (making positive values smaller or negative values closer to 0).

Think of it like a mountain range where positive differences are peaks above ground and negative differences are valleys below ground. Our goal is to flatten everything to ground level (0). 

When consecutive differences have the same sign, we can "share" operations between them. For example, if we have differences `[2, 3, 4]`, we can:
- Apply 2 operations to the entire range `[2, 3, 4]` → `[0, 1, 2]`
- Apply 1 operation to the range `[1, 2]` → `[0, 0, 1]`
- Apply 1 operation to just `[1]` → `[0, 0, 0]`

Total: 2 + 1 + 1 = 4 operations.

Notice that we need at least `abs(2)` operations for the first element. For the second element, since it's larger than the first by 1, we need 1 additional operation. For the third element, since it's larger than the second by 1, we need 1 more additional operation.

When signs change (like going from positive to negative), we can't share operations anymore - we need to start fresh. If we have `[3, 2, -1]`, the operations for the positive part can't help with the negative part.

This leads us to the pattern: 
- Start with the absolute value of the first difference
- For each subsequent element with the same sign, only add the extra amount needed beyond what the previous element already required
- When the sign changes, add the full absolute value of the new difference

**Learn more about [Stack](/problems/stack_intro), [Greedy](/problems/greedy_intro), [Dynamic Programming](/problems/dynamic_programming_intro) and [Monotonic Stack](/problems/mono_stack_intro) patterns.**

## Solution Approach

The solution implements a [dynamic programming](/problems/dynamic_programming_intro) approach that processes the difference array element by element, tracking the minimum operations needed.

First, we calculate the initial operations needed as `f = abs(target[0] - nums[0])`. This represents the minimum operations required to transform the first element.

Then, for each subsequent position `i` from 1 to n-1, we:

1. Calculate the current difference: `x = target[i] - nums[i]`
2. Get the previous difference: `y = target[i-1] - nums[i-1]`

Now we check if the signs of `x` and `y` are the same by testing if `x * y > 0`:

**Case 1: Same sign (both positive or both negative)**
- When differences have the same sign, we can extend operations from the previous element
- We calculate `d = abs(x) - abs(y)`, which represents the additional operations needed
- If `d > 0`, it means the current element needs more operations than the previous one, so we add `d` to our total
- If `d ≤ 0`, the current element is already covered by the operations from the previous element, so we add nothing

**Case 2: Different signs or one is zero**
- When signs change, we can't reuse any operations from the previous element
- We need to add the full `abs(x)` operations to handle this element

The algorithm essentially tracks "layers" of operations. When consecutive differences have the same sign and increasing absolute values, we only count the additional layers needed. When the sign changes, we start counting new layers from scratch.

Time Complexity: `O(n)` - single pass through the array  
Space Complexity: `O(1)` - only using a few variables to track state

## Example Walkthrough

Let's walk through an example with `nums = [3, 1, 5, 4]` and `target = [4, 6, 3, 7]`.

**Step 1: Calculate the difference array**
- `diff[0] = target[0] - nums[0] = 4 - 3 = 1`
- `diff[1] = target[1] - nums[1] = 6 - 1 = 5`
- `diff[2] = target[2] - nums[2] = 3 - 5 = -2`
- `diff[3] = target[3] - nums[3] = 7 - 4 = 3`

So our difference array is `[1, 5, -2, 3]`.

**Step 2: Process each element**

*Initial:* 
- Start with `f = abs(diff[0]) = abs(1) = 1`
- We need at least 1 operation for the first element.

*Position 1:*
- Current difference `x = 5`, previous difference `y = 1`
- Check signs: `5 * 1 = 5 > 0` (same sign - both positive)
- Calculate additional operations needed: `d = abs(5) - abs(1) = 4`
- Since `d > 0`, add 4 to our total: `f = 1 + 4 = 5`
- Interpretation: We can extend the 1 operation from position 0 to cover position 1, but need 4 more operations to reach 5.

*Position 2:*
- Current difference `x = -2`, previous difference `y = 5`
- Check signs: `(-2) * 5 = -10 < 0` (different signs)
- Since signs are different, add the full amount: `f = 5 + abs(-2) = 7`
- Interpretation: We can't reuse any operations from the positive differences. We need 2 fresh operations to handle this negative difference.

*Position 3:*
- Current difference `x = 3`, previous difference `y = -2`
- Check signs: `3 * (-2) = -6 < 0` (different signs)
- Since signs are different, add the full amount: `f = 7 + abs(3) = 10`
- Interpretation: Again, sign change means we need 3 new operations.

**Final answer: 10 operations**

To verify, here's one way to achieve this with 10 operations:
1. Add 1 to subarray [0,1] → nums becomes [4,2,5,4], diff becomes [0,4,-2,3]
2. Add 1 to position [1] (4 times) → nums becomes [4,6,5,4], diff becomes [0,0,-2,3]
3. Subtract 1 from position [2] (2 times) → nums becomes [4,6,3,4], diff becomes [0,0,0,3]
4. Add 1 to position [3] (3 times) → nums becomes [4,6,3,7], diff becomes [0,0,0,0]

Total: 1 + 4 + 2 + 3 = 10 operations ✓

## Python Solution

```python
class Solution:
    def minimumOperations(self, nums: List[int], target: List[int]) -> int:
        """
        Calculate minimum operations to transform nums array to target array.
        Operations can increment/decrement contiguous subarrays by 1.
      
        Strategy: Process differences between arrays and track when we need
        additional operations based on sign changes or magnitude increases.
        """
        n = len(nums)
      
        # Initialize with the absolute difference at first position
        total_operations = abs(target[0] - nums[0])
      
        # Process remaining positions
        for i in range(1, n):
            # Calculate differences at current and previous positions
            current_diff = target[i] - nums[i]
            previous_diff = target[i - 1] - nums[i - 1]
          
            # Check if differences have the same sign (both positive or both negative)
            if current_diff * previous_diff > 0:
                # Same sign: only add operations if current magnitude exceeds previous
                magnitude_increase = abs(current_diff) - abs(previous_diff)
                if magnitude_increase > 0:
                    total_operations += magnitude_increase
            else:
                # Different signs or one is zero: need full magnitude of current difference
                total_operations += abs(current_diff)
      
        return total_operations
```

## Time and Space Complexity

**Time Complexity:** `O(n)`

The algorithm iterates through the array once with a single for loop from index 1 to n-1, where n is the length of the input arrays. Each iteration performs constant time operations:
- Array access operations: `target[i]`, `nums[i]`, `target[i-1]`, `nums[i-1]`
- Arithmetic operations: subtraction, multiplication, absolute value calculation
- Conditional checks and assignments

Since all operations inside the loop take `O(1)` time and the loop runs `n-1` times, the overall time complexity is `O(n)`.

**Space Complexity:** `O(1)`

The algorithm uses only a fixed amount of extra space:
- Variable `n` to store the array length
- Variable `f` to accumulate the result
- Loop variable `i`
- Temporary variables `x`, `y`, and `d` inside the loop

These variables occupy constant space regardless of the input size, making the space complexity `O(1)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Misunderstanding When Operations Can Be Reused

**The Problem:**
A common mistake is thinking that operations from the previous element can always be partially reused when differences have the same sign. Consider this scenario:

```python
nums = [1, 2, 3]
target = [4, 3, 5]
differences = [3, 1, 2]  # All positive
```

You might incorrectly think: "Since all differences are positive, I can apply +3 to the entire array, then handle the remaining differences." This leads to counting operations as: 3 (for entire array) + 0 (second element already done) + 0 (third element needs less than first).

**Why It's Wrong:**
The second element needs +1, but after applying +3 to everything, it would be at 5 (not the target 3). You'd need to subtract 2, requiring additional operations.

**The Correct Understanding:**
Operations can only be reused when moving from left to right if the current element needs at least as many operations in the same direction as the previous one. The algorithm correctly handles this by only reusing operations when `abs(current_diff) >= abs(previous_diff)` for same-sign differences.

### Pitfall 2: Incorrectly Handling Zero Differences

**The Problem:**
When either `current_diff` or `previous_diff` is zero, the sign comparison `current_diff * previous_diff > 0` evaluates to false (since 0 * anything = 0). Developers might mistakenly try to "optimize" this case:

```python
# Incorrect optimization attempt
if previous_diff == 0:
    total_operations += abs(current_diff)
elif current_diff == 0:
    # Wrong: assuming no operations needed
    continue  
```

**Why It's Wrong:**
A zero difference means that position is already correct. The current algorithm handles this correctly by treating zero as a "sign change" scenario, which properly accounts for starting new operations when needed.

### Pitfall 3: Over-complicating the Solution with Explicit Subarray Tracking

**The Problem:**
Some developers try to explicitly track and merge subarrays:

```python
# Over-complicated approach
operations = []
for i in range(n):
    diff = target[i] - nums[i]
    if diff != 0:
        operations.append((i, i, diff))

# Then try to merge overlapping operations...
```

**Why It's Wrong:**
This approach becomes unnecessarily complex and error-prone. The key insight is that we don't need to track actual subarrays - we only need to count the minimum operations by understanding when we can extend existing operations versus when we need new ones.

**The Solution:**
The provided algorithm elegantly sidesteps this by processing differences incrementally and only tracking the additional operations needed at each step, without explicitly managing subarray boundaries.

