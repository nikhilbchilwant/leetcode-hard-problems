# 2366. Minimum Replacements to Sort the Array

> Source: https://algo.monster/liteproblems/2366

## Problem Statement

You have an integer array `nums` indexed from 0. You can perform an operation where you replace any element in the array with **two** elements that add up to the original element's value.

For instance, if you have `nums = [5,6,7]`, you could replace the element `6` (at index 1) with `2` and `4`, transforming the array into `[5,2,4,7]`.

Your goal is to find the **minimum number of operations** needed to make the array sorted in **non-decreasing order** (each element is less than or equal to the next element).

The key insight is that when you replace an element with two smaller elements, you're essentially splitting it. You need to strategically decide which elements to split and how to split them so that the final array becomes sorted with the fewest operations possible.

For example:
- If `nums = [3,9,3]`, you need to split the middle element `9` to make the array non-decreasing
- You could split `9` into smaller parts that fit between or equal to `3` and `3`
- The minimum operations would be determined by how you optimally split such elements

The challenge is to determine which elements must be split and the optimal way to split them to achieve a sorted array with minimal operations.

## Intuition

Let's think about this problem backwards. If we process from left to right, we don't know what values we'll encounter later, making it hard to decide how to split current elements optimally. But if we process from right to left, we can make better decisions.

The key observation is that the last element of the final sorted array should be as large as possible. Why? Because a larger last element gives us more room to work with for the elements before it. Since we want a non-decreasing array, each element must be `≤` the next element. Therefore, there's no benefit in splitting the last element - we should keep it as large as possible.

Working backwards from index `n-1` to `0`, we maintain a variable `mx` representing the maximum value that the current position can have (initially `mx = nums[n-1]`).

For each element `nums[i]`:
- If `nums[i] ≤ mx`, perfect! We don't need to split it. We update `mx = nums[i]` since this becomes the new upper bound for elements to its left.
- If `nums[i] > mx`, we must split it into smaller pieces, each `≤ mx`.

When we need to split `nums[i]` into pieces each `≤ mx`, what's the optimal strategy? We want to:
1. Minimize the number of pieces (fewer pieces = fewer operations)
2. Make the smallest piece as large as possible (to give more room for elements to the left)

The minimum number of pieces needed is `k = ⌈nums[i] / mx⌉`. This requires `k-1` operations (splitting once creates 2 pieces, splitting twice creates 3 pieces, etc.).

To make the pieces as equal as possible (maximizing the minimum piece), we divide `nums[i]` into `k` roughly equal parts. The smallest piece will be `⌊nums[i] / k⌋`, which becomes our new `mx` for the next iteration.

This [greedy](/problems/greedy_intro) approach ensures we perform the minimum operations while maintaining the maximum possible values at each position, giving us the optimal solution.

**Learn more about [Greedy](/problems/greedy_intro) and [Math](/problems/math-basics) patterns.**

## Solution Approach

Let's implement the [greedy](/problems/greedy_intro) approach by traversing the array from right to left:

1. **Initialize variables:**
   - `ans = 0` to count the total operations
   - `n = len(nums)` for array length
   - `mx = nums[-1]` as the maximum allowed value (starting with the last element)

2. **Traverse backwards from index `n-2` to `0`:**
 
   For each element `nums[i]`:
 
   **Case 1: No split needed** (`nums[i] ≤ mx`)
   - The element fits within our constraint
   - Update `mx = nums[i]` as the new upper bound for elements to the left
   - Continue to the next element
 
   **Case 2: Split required** (`nums[i] > mx`)
   - Calculate minimum pieces needed: `k = ⌈nums[i] / mx⌉`
   - We can compute this using: `k = (nums[i] + mx - 1) // mx`
   - Add operations to answer: `ans += k - 1` (since `k` pieces require `k-1` splits)
   - Update `mx` to the smallest piece value: `mx = nums[i] // k`

3. **Return the total operations**

**Example walkthrough:**

Consider `nums = [3, 9, 3]`:
- Start with `mx = 3` (last element)
- At index 1: `nums[1] = 9 > mx = 3`
  - Need `k = ⌈9/3⌉ = 3` pieces
  - Operations: `3 - 1 = 2`
  - New `mx = ⌊9/3⌋ = 3`
- At index 0: `nums[0] = 3 ≤ mx = 3`
  - No split needed
- Total operations: `2`

**Time Complexity:** `O(n)` - single pass through the array
**Space Complexity:** `O(1)` - only using a few variables

The key insight is that by processing right-to-left and greedily minimizing splits while maximizing the smallest piece value, we ensure the optimal solution.

## Example Walkthrough

Let's walk through the solution with `nums = [4, 7, 2]`:

**Initial Setup:**
- `ans = 0` (operation counter)
- `mx = 2` (last element, which sets our initial maximum)
- We'll traverse from right to left: index 1 → index 0

**Step 1: Process index 1 (value = 7)**
- Current element: `nums[1] = 7`
- Check: Is `7 ≤ mx (2)`? No, `7 > 2`, so we need to split
- Calculate pieces needed: `k = ⌈7/2⌉ = 4` pieces
- Operations needed: `k - 1 = 4 - 1 = 3` operations
- Update `ans = 0 + 3 = 3`
- New `mx = ⌊7/4⌋ = 1` (smallest piece when dividing 7 into 4 parts)
- Array conceptually becomes: `[4, 1, 1, 2, 2, 2]` (though we don't actually modify it)

**Step 2: Process index 0 (value = 4)**
- Current element: `nums[0] = 4`
- Check: Is `4 ≤ mx (1)`? No, `4 > 1`, so we need to split
- Calculate pieces needed: `k = ⌈4/1⌉ = 4` pieces
- Operations needed: `k - 1 = 4 - 1 = 3` operations
- Update `ans = 3 + 3 = 6`
- New `mx = ⌊4/4⌋ = 1` (smallest piece when dividing 4 into 4 parts)
- Array conceptually becomes: `[1, 1, 1, 1, 1, 1, 2, 2, 2]`

**Result:** Total operations = `6`

The final conceptual array `[1, 1, 1, 1, 1, 1, 2, 2, 2]` is sorted in non-decreasing order, achieved with 6 operations (3 operations to split the 7, and 3 operations to split the 4).

## Python Solution

```python
class Solution:
    def minimumReplacement(self, nums: List[int]) -> int:
        """
        Minimize the number of operations to make the array non-increasing.
        Each operation replaces an element with two elements that sum to the original.
      
        Strategy: Process from right to left, ensuring each element is <= the next element.
        """
        total_operations = 0
        array_length = len(nums)
      
        # Start with the rightmost element as the maximum allowed value
        max_allowed_value = nums[-1]
      
        # Process elements from right to left (second-to-last to first)
        for index in range(array_length - 2, -1, -1):
            current_value = nums[index]
          
            # If current element is already <= max_allowed_value, no operations needed
            if current_value <= max_allowed_value:
                max_allowed_value = current_value
                continue
          
            # Calculate minimum number of parts to split current element
            # Using ceiling division: (current_value + max_allowed_value - 1) // max_allowed_value
            num_parts = (current_value + max_allowed_value - 1) // max_allowed_value
          
            # Number of operations = number of parts - 1 (splitting into k parts takes k-1 operations)
            total_operations += num_parts - 1
          
            # Update max_allowed_value to the smallest value after optimal splitting
            # This ensures the leftmost split value is minimized for future iterations
            max_allowed_value = current_value // num_parts
      
        return total_operations
```

## Time and Space Complexity

The time complexity is `O(n)`, where `n` is the length of the array `nums`. The algorithm iterates through the array exactly once from right to left (from index `n-2` to `0`), performing constant-time operations at each index. Even though there are arithmetic operations like division and modulo inside the loop, these operations take `O(1)` time for integer arithmetic.

The space complexity is `O(1)`. The algorithm only uses a fixed amount of extra space regardless of the input size. The variables used are `ans` (to store the result), `n` (to store the array length), `mx` (to track the maximum allowed value), `i` (loop counter), and `k` (temporary variable for calculations). No additional data structures that scale with input size are created.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Ceiling Division Formula**

**Pitfall:** Using standard ceiling division with floating-point operations like `math.ceil(current_value / max_allowed_value)` can lead to floating-point precision errors, especially with large integers.

**Why it's problematic:**
- Floating-point arithmetic can introduce rounding errors
- For large integers, precision loss might cause incorrect results
- Extra import of `math` module is unnecessary

**Solution:** Use integer-only ceiling division formula:
```python
# Correct: Integer ceiling division
num_parts = (current_value + max_allowed_value - 1) // max_allowed_value

# Avoid: Floating-point ceiling
num_parts = math.ceil(current_value / max_allowed_value)
```

### 2. **Wrong Update of Maximum Allowed Value After Splitting**

**Pitfall:** After splitting an element, incorrectly updating the `max_allowed_value` can break the algorithm. A common mistake is using the remainder or the ceiling division result instead of floor division.

**Why it's problematic:**
- Using `current_value % num_parts` gives the remainder, not the minimum piece value
- Using `(current_value + num_parts - 1) // num_parts` gives the maximum piece value, not minimum
- This leads to suboptimal splits in subsequent iterations

**Solution:** Always use floor division to get the minimum piece value:
```python
# Correct: Minimum piece value ensures optimal future splits
max_allowed_value = current_value // num_parts

# Wrong: Using remainder
max_allowed_value = current_value % num_parts

# Wrong: Using ceiling (maximum piece value)
max_allowed_value = (current_value + num_parts - 1) // num_parts
```

### 3. **Processing Array in Wrong Direction**

**Pitfall:** Processing the array from left to right instead of right to left.

**Why it's problematic:**
- When processing left to right, you don't know what constraints future elements will impose
- This can lead to unnecessary splits or inability to determine optimal split values
- The problem becomes significantly more complex or unsolvable with a greedy approach

**Solution:** Always process from right to left:
```python
# Correct: Right to left traversal
for index in range(len(nums) - 2, -1, -1):
    # Process nums[index]

# Wrong: Left to right traversal
for index in range(len(nums) - 1):
    # This won't work with the greedy approach
```

### 4. **Off-by-One Error in Operations Count**

**Pitfall:** Confusing the number of pieces with the number of operations needed.

**Why it's problematic:**
- Splitting into `k` pieces requires `k-1` operations (cuts)
- Adding `k` instead of `k-1` to the operation count gives wrong answer

**Solution:** Remember that operations = pieces - 1:
```python
# Correct: k pieces need k-1 operations
total_operations += num_parts - 1

# Wrong: Adding number of pieces directly
total_operations += num_parts
```

### 5. **Edge Case: Single Element Array**

**Pitfall:** Not handling arrays with only one element properly.

**Why it's problematic:**
- The loop `range(n-2, -1, -1)` becomes empty when `n=1`
- While this naturally returns 0 (correct answer), it's important to understand why

**Solution:** The current implementation handles this correctly, but be aware:
```python
# For nums = [5], the loop doesn't execute, returning 0 operations (correct)
# But explicitly handling might make code clearer:
if len(nums) <= 1:
    return 0
```

