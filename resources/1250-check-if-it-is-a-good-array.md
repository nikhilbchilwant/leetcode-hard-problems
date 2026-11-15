# 1250. Check If It Is a Good Array

> Source: https://algo.monster/liteproblems/1250

## Problem Statement

You are given an array `nums` containing positive integers. Your task is to determine if you can select any subset of numbers from this array, multiply each selected number by any integer (positive, negative, or zero), and then add all these products together to get a sum of exactly `1`.

For example, if you have an array `[3, 6]`, you could select both numbers and use multipliers like `3 × 1 + 6 × (-1) = 3 - 6 = -3`, but this doesn't give us `1`. However, if you have `[3, 5]`, you could use `3 × 2 + 5 × (-1) = 6 - 5 = 1`.

The array is called **good** if there exists at least one way to achieve a sum of `1` through this process. Return `True` if the array is good, otherwise return `False`.

The key insight is based on Bézout's Identity from number theory. This identity states that for any set of integers, you can form the value `1` as a linear combination (sum of products) if and only if the greatest common divisor (GCD) of all those integers equals `1`. 

Therefore, the problem reduces to checking whether the GCD of all numbers in the array equals `1`. If it does, then by Bézout's Identity, there exist integer coefficients (multipliers) that can be used with some subset of the array to produce a sum of `1`.

## Intuition

Let's start by thinking about the simplest case - what if we only had two numbers in our array, say `a` and `b`? We want to find integer multipliers `x` and `y` such that `a × x + b × y = 1`.

When can we find such integers? Consider the numbers `6` and `9`. Their common divisors are `1` and `3`, with GCD being `3`. No matter what integers we multiply them by, we'll always get multiples of `3`. We can get `3, 6, 9, 12, ...` but never `1`. This is because `6 = 3 × 2` and `9 = 3 × 3`, so any linear combination will be `3 × (2x + 3y)`, which is always divisible by `3`.

Now consider `5` and `7`. Their GCD is `1` (they're coprime). We can indeed find integers to get `1`: `5 × 3 + 7 × (-2) = 15 - 14 = 1`. This works because when GCD is `1`, the two numbers share no common factors other than `1`.

This observation leads us to a fundamental theorem in number theory - Bézout's Identity. It states that for integers `a` and `b`, the equation `a × x + b × y = d` has integer solutions if and only if `d` is divisible by `gcd(a, b)`. In our case, since we want `d = 1`, we need `gcd(a, b) = 1`.

What about more than two numbers? The beautiful thing is that this property extends. If we have numbers `a₁, a₂, ..., aₙ`, we can obtain `1` as their linear combination if and only if `gcd(a₁, a₂, ..., aₙ) = 1`. 

Why? Because the GCD of all numbers represents the largest value that divides all of them. If this GCD is greater than `1`, say it's `g`, then every linear combination of these numbers will be divisible by `g`, making it impossible to obtain `1`. But if the GCD is `1`, it means at least some subset of these numbers are collectively coprime, and we can use them to construct `1`.

This transforms our original problem into a much simpler one: instead of trying to find the actual subset and multipliers, we just need to check if the GCD of all numbers in the array equals `1`.

**Learn more about [Math](/problems/math-basics) patterns.**

## Solution Approach

The implementation is remarkably elegant once we understand the mathematical foundation. We need to find the GCD of all numbers in the array and check if it equals `1`.

The solution uses Python's `reduce` function from the `functools` module combined with the `gcd` function from the `[math](/problems/math-basics)` module. Here's how it works:

1. **Computing GCD of Multiple Numbers**: While `gcd` typically works with two numbers, we can extend it to multiple numbers using the property that `gcd(a, b, c) = gcd(gcd(a, b), c)`. This means we can compute the GCD of multiple numbers by repeatedly applying the GCD function pairwise.

2. **Using Reduce**: The `reduce` function is perfect for this task. It takes a binary function (in our case, `gcd`) and applies it cumulatively to the items in the array from left to right. For example:
   - If `nums = [6, 10, 15]`
   - `reduce(gcd, nums)` computes: `gcd(gcd(6, 10), 15) = gcd(2, 15) = 1`

3. **The Algorithm Flow**:
   - Start with the first element of the array
   - Compute GCD with the second element
   - Take this result and compute GCD with the third element
   - Continue this process until all elements are processed
   - The final result is the GCD of all numbers in the array

4. **Final Check**: Once we have the GCD of all numbers, we simply check if it equals `1`. If yes, return `True` (the array is good); otherwise, return `False`.

The time complexity is `O(n × log(min(nums)))` where `n` is the length of the array, as we perform `n-1` GCD operations, and each GCD operation using the Euclidean algorithm takes `O(log(min(a, b)))` time.

The space complexity is `O(1)` as we only need constant extra space for the GCD calculations.

This solution is optimal because we must examine all numbers at least once to determine if any subset can form `1`, and the GCD approach gives us the answer by examining each number exactly once.

## Example Walkthrough

Let's walk through the solution with the array `nums = [6, 10, 15]`.

**Step 1: Understanding what we're looking for**
We need to check if we can select any subset and multiply each number by some integer to get a sum of 1. For example, can we find integers x, y, z such that `6×x + 10×y + 15×z = 1`?

**Step 2: Apply the GCD approach**
According to Bézout's Identity, we can achieve a sum of 1 if and only if GCD(6, 10, 15) = 1.

**Step 3: Calculate GCD using reduce**
The `reduce(gcd, nums)` function will compute:
- First iteration: `gcd(6, 10)`
  - 10 = 6 × 1 + 4
  - 6 = 4 × 1 + 2  
  - 4 = 2 × 2 + 0
  - So `gcd(6, 10) = 2`

- Second iteration: `gcd(2, 15)`
  - 15 = 2 × 7 + 1
  - 2 = 1 × 2 + 0
  - So `gcd(2, 15) = 1`

**Step 4: Check the result**
Since the final GCD = 1, the array is **good** and we return `True`.

**Verification**: Indeed, we can find multipliers that work. For instance:
- Using all three numbers: `6×(-2) + 10×2 + 15×(-1) = -12 + 20 - 15 = -7` (not 1, but other combinations exist)
- Using just 10 and 15: `10×(-2) + 15×1 = -20 + 15 = -5` (not 1)
- Using 6 and 10: Since gcd(6,10) = 2, we can only get even numbers from these two
- But since gcd(2, 15) = 1, we know there exists some combination that works: `2×8 + 15×(-1) = 16 - 15 = 1`

This means we could use `(6×4 + 10×(-2)) + 15×(-1) = (24-20) + 15×(-1) = 4 + 15×(-1)`. Actually, let's verify with the subset {10, 15}: `10×3 + 15×(-2) = 30 - 30 = 0`. Let me recalculate: since gcd(10,15) = 5, we can't get 1 from just these two. But since gcd(6,10,15) = 1, we know some subset works. In fact, checking {6,15}: gcd(6,15) = 3, so not these two either. But {10,6} with gcd=2 combined with 15 gives us gcd=1 overall, meaning the full set or certain combinations will work.

The key insight is we don't need to find the actual multipliers - just knowing that GCD = 1 guarantees they exist!

## Python Solution

```python
from typing import List
from functools import reduce
from math import gcd

class Solution:
    def isGoodArray(self, nums: List[int]) -> bool:
        """
        Determines if the array is "good" based on Bézout's identity.
      
        An array is considered "good" if there exist integers x_i such that:
        x_1 * nums[0] + x_2 * nums[1] + ... + x_n * nums[n-1] = 1
      
        By Bézout's identity, this is possible if and only if gcd(nums) = 1.
      
        Args:
            nums: List of positive integers
          
        Returns:
            True if the array is good, False otherwise
        """
        # Calculate the GCD of all numbers in the array
        # If GCD equals 1, then linear combination can produce 1
        overall_gcd = reduce(gcd, nums)
      
        return overall_gcd == 1
```

## Time and Space Complexity

The time complexity is `O(n × log m)`, where `n` is the length of the array `nums` and `m` is the maximum value in the array.

- The `reduce` function iterates through all `n` elements in the array
- For each element, it performs a GCD operation with the accumulated result
- Each GCD operation between two numbers has complexity `O(log min(a, b))`, which in worst case is `O(log m)`
- Since we perform `n-1` GCD operations total, the overall time complexity is `O(n × log m)`

The space complexity is `O(1)`.

- The `reduce` function only maintains a single accumulated value (the running GCD)
- The `gcd` function uses constant extra space for its iterative or recursive implementation
- No additional data structures are created that scale with input size

Note: The reference answer states `O(n + log m)` for time complexity, but this appears to be incorrect. The GCD operations cannot be performed in amortized constant time - each of the `n-1` GCD calculations takes `O(log m)` time, resulting in `O(n × log m)` total time complexity.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Empty Array Handling**
The current implementation will raise an error if an empty array is passed to the `reduce` function, as `reduce` requires at least one element when no initial value is provided.

**Example of the issue:**
```python
nums = []
reduce(gcd, nums)  # Raises TypeError: reduce() of empty sequence with no initial value
```

**Solution:**
Add a check for empty arrays or provide an initial value to `reduce`:

```python
def isGoodArray(self, nums: List[int]) -> bool:
    if not nums:
        return False  # Empty array cannot form 1
  
    overall_gcd = reduce(gcd, nums)
    return overall_gcd == 1
```

### 2. **Single Element Arrays**
While the current code handles single-element arrays correctly, it's worth understanding the behavior. If `nums = [1]`, the GCD is 1 (good array). If `nums = [2]`, the GCD is 2 (not a good array).

**Potential confusion:** Some might think a single element array should always return `False` since you can't form a "combination" with one element. However, mathematically, if that single element is 1, then `1 × 1 = 1` is valid.

### 3. **Integer Overflow in Other Languages**
While Python handles arbitrary precision integers automatically, implementing this in languages like C++ or Java requires careful consideration of integer overflow during GCD calculations.

**Example issue in C++:**
```cpp
// Large numbers might overflow during intermediate GCD calculations
int gcd(int a, int b) {
    while (b != 0) {
        int temp = b;
        b = a % b;  // Potential overflow here
        a = temp;
    }
    return a;
}
```

**Solution for other languages:**
Use long long or BigInteger types when dealing with large numbers.

### 4. **Misunderstanding the Problem Statement**
A common misconception is thinking we need to find the actual coefficients (multipliers) that produce 1. The problem only asks whether such coefficients exist, which is answered by checking if GCD = 1.

**Incorrect approach:**
```python
# Trying to find actual coefficients - unnecessary and complex
def findCoefficients(nums):
    # Complex algorithm to find x_i values...
    pass
```

**Correct approach:**
Simply check if GCD equals 1, as shown in the solution.

### 5. **Assuming All Numbers Must Be Used**
The problem states we can select "any subset" of numbers. Some might incorrectly assume all numbers must participate in forming 1. However, the GCD approach correctly handles this because:
- If GCD of all numbers is 1, then there exists some subset whose GCD is also 1
- We don't need to check all possible subsets individually

**Example:** `nums = [6, 10, 15, 7]`
- GCD of all: `gcd(6, 10, 15, 7) = 1`
- Even though `gcd(6, 10, 15) = 1`, we don't need to check subsets separately

