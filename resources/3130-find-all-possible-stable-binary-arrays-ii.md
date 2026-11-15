# 3130. Find All Possible Stable Binary Arrays II

> Source: https://algo.monster/liteproblems/3130

## Problem Statement

You need to count the number of valid binary arrays that meet specific requirements.

Given three positive integers:
- `zero`: the exact number of 0s that must appear in the array
- `one`: the exact number of 1s that must appear in the array  
- `limit`: the maximum size threshold for subarrays

A binary array is considered **stable** if it satisfies all three conditions:

1. It contains exactly `zero` number of 0s
2. It contains exactly `one` number of 1s
3. Every subarray with size greater than `limit` must contain at least one 0 and at least one 1

The third condition ensures that you cannot have long consecutive sequences of the same digit. For example, if `limit = 2`, then you cannot have subarrays like `[0,0,0]` or `[1,1,1]` since these subarrays of size 3 only contain one type of digit.

Your task is to find the total number of different stable binary arrays that can be formed. Since this number can be very large, return the result modulo `10^9 + 7`.

For instance, if `zero = 1`, `one = 1`, and `limit = 2`, the valid arrays would be `[0,1]` and `[1,0]`, giving an answer of 2.

## Intuition

The key insight is to think about building the array position by position while keeping track of what digit we place at each position. We need to ensure that no consecutive sequence of the same digit exceeds the `limit`.

Let's think recursively: if we have `i` zeros and `j` ones remaining to place, and we know the last digit we placed was `k` (where `k` is either 0 or 1), how many valid arrays can we form?

When placing the next digit, we have two choices - place a 0 or place a 1. However, we need to be careful about the constraint that no subarray larger than `limit` can contain only one type of digit.

Here's the crucial observation: if we place a digit `k` at the current position, we need to ensure that within the last `limit + 1` positions, there's at least one digit that's different from `k`. This means if we've placed more than `limit` consecutive same digits, the array becomes invalid.

To handle this constraint efficiently, we can use the principle of inclusion-exclusion. When calculating `dfs(i, j, k)`:
- We first count all possible ways by considering both placing another `k` and placing the opposite digit
- Then we subtract the invalid cases where we would have more than `limit` consecutive `k`s

For example, if `k = 0`, we calculate:
- `dfs(i - 1, j, 0)`: place another 0
- `dfs(i - 1, j, 1)`: place a 1 after the 0
- Subtract `dfs(i - limit - 1, j, 1)`: this represents the cases where we would have `limit + 1` consecutive 0s

The subtraction works because if we're at position with `i` zeros remaining and we place `limit + 1` consecutive zeros, we'd transition from a state where we had `i + limit + 1` zeros with the last digit being 1, to a state with `i` zeros. By subtracting these cases, we eliminate arrangements that violate the constraint.

The base cases are straightforward: when we have only zeros or only ones left, we can place them all consecutively only if their count doesn't exceed `limit`.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) and [Prefix Sum](/problems/subarray_sum) patterns.**

## Solution Approach

The solution uses [dynamic programming](/problems/dynamic_programming_intro) with memoization to count valid stable arrays. We implement a recursive function `dfs(i, j, k)` that represents the number of stable arrays when we have `i` zeros and `j` ones remaining to place, and the last placed digit is `k` (0 or 1).

**State Definition:**
- `i`: number of zeros remaining to place
- `j`: number of ones remaining to place  
- `k`: the last digit placed (0 or 1)

**Base Cases:**
1. If `i = 0` (no zeros left):
   - Return 1 if `k = 1` and `j ≤ limit` (we can place all remaining ones consecutively)
   - Return 0 otherwise
 
2. If `j = 0` (no ones left):
   - Return 1 if `k = 0` and `i ≤ limit` (we can place all remaining zeros consecutively)
   - Return 0 otherwise

**Recursive Transition:**

For `k = 0` (last placed digit was 0):
```
dfs(i, j, 0) = dfs(i - 1, j, 0) + dfs(i - 1, j, 1) - dfs(i - limit - 1, j, 1)
```
- `dfs(i - 1, j, 0)`: place another 0 after the current 0
- `dfs(i - 1, j, 1)`: place a 1 after the current 0
- `-dfs(i - limit - 1, j, 1)`: subtract invalid cases where we'd have `limit + 1` consecutive 0s

For `k = 1` (last placed digit was 1):
```
dfs(i, j, 1) = dfs(i, j - 1, 0) + dfs(i, j - 1, 1) - dfs(i, j - limit - 1, 0)
```
- `dfs(i, j - 1, 0)`: place a 0 after the current 1
- `dfs(i, j - 1, 1)`: place another 1 after the current 1
- `-dfs(i, j - limit - 1, 0)`: subtract invalid cases where we'd have `limit + 1` consecutive 1s

**Implementation Details:**
1. Use `@cache` decorator for memoization to avoid recalculating same states
2. Handle edge cases where `i - limit - 1 < 0` or `j - limit - 1 < 0` by returning 0
3. The final answer is `(dfs(zero, one, 0) + dfs(zero, one, 1)) % mod`, considering both starting with 0 or 1
4. Clear the cache after computation to free memory

The time complexity is `O(zero × one × 2)` for the number of unique states, and space complexity is `O(zero × one × 2)` for memoization storage.

## Example Walkthrough

Let's walk through a small example with `zero = 3`, `one = 2`, and `limit = 1`.

Since `limit = 1`, we cannot have more than 1 consecutive digit of the same type. This means our array must alternate between 0s and 1s.

Let's trace through the recursive calls starting with `dfs(3, 2, 0)` (3 zeros left, 2 ones left, last digit placed was 0):

**Step 1:** Calculate `dfs(3, 2, 0)`
- We can place another 0: `dfs(2, 2, 0)` 
- We can place a 1: `dfs(2, 2, 1)`
- Subtract invalid cases with 2 consecutive 0s: `-dfs(1, 2, 1)`

**Step 2:** Calculate `dfs(2, 2, 0)`
- Place another 0: `dfs(1, 2, 0)`
- Place a 1: `dfs(1, 2, 1)` 
- Subtract invalid: `-dfs(0, 2, 1)`

When we reach `dfs(0, 2, 1)`, this is a base case. Since we have 0 zeros left and 2 ones remaining, and `2 > limit`, we return 0 (cannot place 2 consecutive ones).

**Step 3:** Calculate `dfs(1, 2, 1)` 
- Place a 0: `dfs(0, 2, 0)`
- Place another 1: `dfs(0, 1, 1)`
- Subtract invalid: `-dfs(0, 0, 0)` (returns 0 since j = 0)

For `dfs(0, 2, 0)`, we have only ones left. Since `2 > limit`, we return 0.
For `dfs(0, 1, 1)`, we have 1 one left and last digit is 1. Since `1 ≤ limit`, we return 1.

**Step 4:** Continue unwinding
Working backward through our calculations:
- `dfs(1, 2, 1) = 0 + 1 - 0 = 1`
- `dfs(1, 2, 0) = dfs(0, 2, 0) + dfs(0, 2, 1) - dfs(-1, 2, 1) = 0 + 0 - 0 = 0`
- `dfs(2, 2, 0) = 0 + 1 - 0 = 1`

Similarly, we calculate `dfs(3, 2, 1)` starting with a 1.

The valid arrays for this example would be patterns like:
- `[0, 1, 0, 1, 0]` 
- `[1, 0, 1, 0, 0]`

Each valid array must alternate digits since we cannot have consecutive same digits (limit = 1). The total count would be `(dfs(3, 2, 0) + dfs(3, 2, 1)) % mod`.

## Python Solution

```python
class Solution:
    def numberOfStableArrays(self, zero: int, one: int, limit: int) -> int:
        from functools import cache
      
        MOD = 10**9 + 7
      
        @cache
        def count_arrays(zeros_left: int, ones_left: int, last_digit: int) -> int:
            """
            Count valid stable arrays with given zeros and ones remaining.
          
            Args:
                zeros_left: Number of zeros still to place
                ones_left: Number of ones still to place  
                last_digit: The last digit placed (0 or 1)
          
            Returns:
                Number of valid stable arrays
            """
            # Base case: only ones left
            if zeros_left == 0:
                # Valid if last digit was 1 and remaining ones don't exceed limit
                return int(last_digit == 1 and ones_left <= limit)
          
            # Base case: only zeros left
            if ones_left == 0:
                # Valid if last digit was 0 and remaining zeros don't exceed limit
                return int(last_digit == 0 and zeros_left <= limit)
          
            # If last digit was 0, we're placing another 0
            if last_digit == 0:
                # Add one more 0 to the sequence
                total = count_arrays(zeros_left - 1, ones_left, 0) + count_arrays(zeros_left - 1, ones_left, 1)
              
                # Subtract invalid cases where we exceed limit consecutive zeros
                # This happens when we have more than 'limit' consecutive zeros
                if zeros_left - limit - 1 >= 0:
                    total -= count_arrays(zeros_left - limit - 1, ones_left, 1)
              
                return total
          
            # If last digit was 1, we're placing another 1
            else:
                # Add one more 1 to the sequence
                total = count_arrays(zeros_left, ones_left - 1, 0) + count_arrays(zeros_left, ones_left - 1, 1)
              
                # Subtract invalid cases where we exceed limit consecutive ones
                # This happens when we have more than 'limit' consecutive ones
                if ones_left - limit - 1 >= 0:
                    total -= count_arrays(zeros_left, ones_left - limit - 1, 0)
              
                return total
      
        # Calculate total by considering both starting with 0 and starting with 1
        result = (count_arrays(zero, one, 0) + count_arrays(zero, one, 1)) % MOD
      
        # Clear the cache to free memory
        count_arrays.cache_clear()
      
        return result
```

## Time and Space Complexity

**Time Complexity:** `O(zero * one * 2)` = `O(zero * one)`

The function uses memoization with `@cache` decorator. The state space is defined by three parameters:
- `i`: ranges from `0` to `zero` (zero + 1 possible values)
- `j`: ranges from `0` to `one` (one + 1 possible values)  
- `k`: can be either `0` or `1` (2 possible values)

Each unique state `(i, j, k)` is computed at most once due to memoization. The total number of unique states is `(zero + 1) * (one + 1) * 2`. For each state, the function performs constant time operations (basic arithmetic and at most 2 recursive calls that are either cached or lead to new states).

Therefore, the time complexity is `O(zero * one * 2)` = `O(zero * one)`.

**Space Complexity:** `O(zero * one)`

The space complexity consists of:
1. **Memoization cache:** Stores all unique states visited, which is at most `(zero + 1) * (one + 1) * 2` = `O(zero * one)` entries.
2. **Recursion stack:** The maximum recursion depth is `zero + one` in the worst case (when we decrement either `i` or `j` by 1 in each recursive call until both reach 0). This contributes `O(zero + one)` to space complexity.

Since `O(zero * one)` dominates `O(zero + one)` for non-trivial inputs, the overall space complexity is `O(zero * one)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Subtraction Logic for Consecutive Elements**

**The Pitfall:** 
A common mistake is misunderstanding why we subtract `dfs(i - limit - 1, j, 1)` when the last digit is 0. Many developers incorrectly think this represents "having exactly limit+1 consecutive zeros," but it actually represents the count of arrays where we would create MORE than limit consecutive zeros.

**Why This Happens:**
When we have `i` zeros left and the last placed digit was 0, calling `dfs(i-1, j, 0)` means we're placing another 0. If we've already placed some zeros, and now we're adding the `(limit+1)`-th consecutive zero, we're violating the constraint. The subtraction removes cases where placing all remaining zeros would create an invalid sequence.

**Correct Understanding:**
- `dfs(i-1, j, 0) + dfs(i-1, j, 1)` counts ALL ways to place remaining elements
- `dfs(i - limit - 1, j, 1)` counts the INVALID cases where we'd have `limit+1` consecutive zeros
- The subtraction gives us only the VALID arrangements

### 2. **Forgetting to Handle Negative Indices**

**The Pitfall:**
Not checking if `i - limit - 1 < 0` or `j - limit - 1 < 0` before making the subtraction. This leads to incorrect recursive calls with negative parameters.

**Solution:**
```python
if last_digit == 0:
    total = count_arrays(zeros_left - 1, ones_left, 0) + count_arrays(zeros_left - 1, ones_left, 1)
    # Only subtract if the index is valid
    if zeros_left - limit - 1 >= 0:
        total -= count_arrays(zeros_left - limit - 1, ones_left, 1)
```

### 3. **Applying Modulo Operation Incorrectly**

**The Pitfall:**
Applying modulo only at the final return, not during intermediate calculations. This can cause integer overflow in languages with fixed integer sizes, or lead to incorrect results when subtracting.

**Solution:**
Apply modulo after each operation to keep numbers manageable:
```python
if last_digit == 0:
    total = (count_arrays(zeros_left - 1, ones_left, 0) + count_arrays(zeros_left - 1, ones_left, 1)) % MOD
    if zeros_left - limit - 1 >= 0:
        # Handle potential negative results from subtraction
        total = (total - count_arrays(zeros_left - limit - 1, ones_left, 1) + MOD) % MOD
```

### 4. **Misunderstanding the Initial Call**

**The Pitfall:**
Thinking that `dfs(zero, one, 0)` means "start the array with a 0." It actually means "we have zero zeros and one ones left to place, and the last placed digit was 0."

**Correct Interpretation:**
- `dfs(zero, one, 0)` represents: "Count arrays where we need to place 'zero' zeros and 'one' ones, assuming the previous (imaginary) digit was 0"
- `dfs(zero, one, 1)` represents: "Count arrays where we need to place 'zero' zeros and 'one' ones, assuming the previous (imaginary) digit was 1"
- The sum gives us all possible valid arrays regardless of what the first actual digit is

### 5. **Memory Leak with @cache**

**The Pitfall:**
Not clearing the cache after computation, which can cause memory issues when the function is called multiple times (e.g., in a testing environment).

**Solution:**
Always clear the cache after getting the result:
```python
result = (count_arrays(zero, one, 0) + count_arrays(zero, one, 1)) % MOD
count_arrays.cache_clear()  # Important for memory management
return result
```

