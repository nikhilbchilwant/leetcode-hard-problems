# 2897. Apply Operations on Array to Maximize Sum of Squares

> Source: https://algo.monster/liteproblems/2897

## Problem Statement

You have an array of integers `nums` (0-indexed) and a positive integer `k`.

You can perform the following operation any number of times:
- Pick any two different indices `i` and `j`
- Simultaneously update:
  - `nums[i]` becomes `nums[i] AND nums[j]`
  - `nums[j]` becomes `nums[i] OR nums[j]`

After performing operations (or choosing not to perform any), you need to select `k` elements from the final array and calculate the sum of their squares.

Your goal is to find the maximum possible sum of squares of `k` selected elements.

Return the result modulo `10^9 + 7`.

**Key Insight**: The AND and OR operations have an interesting property at the bit level. When you apply these operations to two numbers:
- If both bits at a position are the same (both 0 or both 1), they remain unchanged
- If the bits are different (one is 0, one is 1), the AND operation produces 0 and the OR operation produces 1

This means you can effectively "move" 1-bits from one number to another. The strategy is to concentrate as many 1-bits as possible into fewer numbers to create larger values. Since we want to maximize the sum of squares, and `(a+c)^2 + (b-c)^2 > a^2 + b^2` when `a > b` and `c > 0`, it's optimal to make the largest numbers as large as possible.

## Intuition

Let's first understand what happens when we perform the operation on two numbers at the bit level. Consider two bits at the same position:
- `1 AND 1 = 1`, `1 OR 1 = 1` (both stay as 1)
- `0 AND 0 = 0`, `0 OR 0 = 0` (both stay as 0)
- `1 AND 0 = 0`, `1 OR 0 = 1` (the 1 moves from first number to second)
- `0 AND 1 = 0`, `0 OR 1 = 1` (the 1 stays in the second number)

The key observation is that **we can redistribute 1-bits among numbers, but we cannot create or destroy them**. The total count of 1-bits at each bit position remains constant across all operations.

Now, why do we want to concentrate bits? Consider a simple example with two numbers that have value 3 each: `3^2 + 3^2 = 18`. If we could somehow move bits to make them 6 and 0: `6^2 + 0^2 = 36`. The sum of squares increased significantly!

This happens because squaring amplifies differences. When we have `x^2 + y^2` and transfer some value `c` from `y` to `x`, we get `(x+c)^2 + (y-c)^2 = x^2 + y^2 + 2c(x-y) + 2c^2`. Since `x > y` after the transfer, the term `2c(x-y)` is positive, making the total sum larger.

Therefore, the optimal strategy is to make numbers as large as possible by concentrating all available 1-bits. We count how many 1-bits exist at each bit position across all numbers (since this total is invariant). Then, we greedily construct the `k` largest possible numbers by assembling these bits - taking one bit from each position's count until we've built `k` numbers, always building the largest numbers we can first.

This [greedy](/problems/greedy_intro) approach works because we want to maximize `sum(x_i^2)` for `k` numbers, and concentrating bits into fewer, larger numbers always produces a higher sum of squares than spreading them out evenly.

**Learn more about [Greedy](/problems/greedy_intro) patterns.**

## Solution Approach

The implementation follows a bit manipulation strategy combined with [greedy](/problems/greedy_intro) selection:

**Step 1: Count the bits**

First, we create an array `cnt` of size 31 (enough to handle 32-bit integers) to count how many numbers have a 1-bit at each bit position.

```python
cnt = [0] * 31
for x in nums:
    for i in range(31):
        if x >> i & 1:
            cnt[i] += 1
```

For each number `x` in the array, we check each bit position `i` from 0 to 30. The expression `x >> i & 1` shifts `x` right by `i` positions and checks if the least significant bit is 1. If it is, we increment `cnt[i]`.

**Step 2: Greedily construct the k largest numbers**

Now we build `k` numbers by using the available 1-bits:

```python
ans = 0
for _ in range(k):
    x = 0
    for i in range(31):
        if cnt[i]:
            x |= 1 << i
            cnt[i] -= 1
    ans = (ans + x * x) % mod
```

For each of the `k` numbers we need to select:
- Initialize `x = 0` to build a new number
- For each bit position `i`, if there's still a 1-bit available (`cnt[i] > 0`):
  - Set that bit in `x` using `x |= 1 << i` (OR operation with a number that has only bit `i` set)
  - Decrement `cnt[i]` to mark that we've used one 1-bit from this position
- Square the constructed number and add it to our answer

**Why this [greedy](/problems/greedy_intro) approach works:**

By taking one bit from each available position for each number, we ensure that:
1. The first number gets all possible bits (one from each position that has any)
2. The second number gets all remaining bits (one from each position that still has any)
3. And so on...

This naturally creates the largest possible numbers first, which when squared, give us the maximum sum. The modulo operation `% (10^9 + 7)` is applied to keep the result within bounds as required by the problem.

## Example Walkthrough

Let's walk through a small example with `nums = [5, 6, 3]` and `k = 2`.

**Step 1: Understand the initial binary representation**
- 5 = 101₂ (bits at positions 2 and 0)
- 6 = 110₂ (bits at positions 2 and 1)  
- 3 = 011₂ (bits at positions 1 and 0)

**Step 2: Count bits at each position**
- Position 0: Two 1-bits (from 5 and 3)
- Position 1: Two 1-bits (from 6 and 3)
- Position 2: Two 1-bits (from 5 and 6)

So our bit count array is `cnt = [2, 2, 2]` (only showing relevant positions).

**Step 3: Greedily construct k=2 largest numbers**

Building the **first number**:
- Check position 0: `cnt[0] = 2` > 0, so add bit 0 → number becomes 001₂
- Check position 1: `cnt[1] = 2` > 0, so add bit 1 → number becomes 011₂  
- Check position 2: `cnt[2] = 2` > 0, so add bit 2 → number becomes 111₂ = 7

After taking these bits: `cnt = [1, 1, 1]`

Building the **second number**:
- Check position 0: `cnt[0] = 1` > 0, so add bit 0 → number becomes 001₂
- Check position 1: `cnt[1] = 1` > 0, so add bit 1 → number becomes 011₂
- Check position 2: `cnt[2] = 1` > 0, so add bit 2 → number becomes 111₂ = 7

After taking these bits: `cnt = [0, 0, 0]`

**Step 4: Calculate the sum of squares**
- First number: 7² = 49
- Second number: 7² = 49
- Total sum: 49 + 49 = 98

**Verification**: We've created two numbers (7, 7) from the original array. The third number would be 0 since no bits remain. This is optimal because we concentrated all six 1-bits into two numbers rather than spreading them across three, maximizing the sum of squares.

## Python Solution

```python
class Solution:
    def maxSum(self, nums: List[int], k: int) -> int:
        MOD = 10**9 + 7
      
        # Count the number of set bits at each position across all numbers
        bit_count = [0] * 31
        for num in nums:
            for bit_position in range(31):
                # Check if the bit at position 'bit_position' is set
                if num >> bit_position & 1:
                    bit_count[bit_position] += 1
      
        total_sum = 0
      
        # Greedily construct k numbers to maximize the sum of squares
        for _ in range(k):
            current_number = 0
          
            # For each bit position, if there are available bits, set them
            for bit_position in range(31):
                if bit_count[bit_position] > 0:
                    # Set the bit at this position in the current number
                    current_number |= 1 << bit_position
                    # Decrement the available count for this bit position
                    bit_count[bit_position] -= 1
          
            # Add the square of the constructed number to the total sum
            total_sum = (total_sum + current_number * current_number) % MOD
      
        return total_sum
```

## Time and Space Complexity

**Time Complexity:** `O(n × log M + k × log M)`

The algorithm consists of two main parts:
1. Counting bits across all numbers: The outer loop iterates through `n` numbers, and for each number, we check `31` bits (which represents `log M` where `M` is the maximum value that can be represented in 31 bits). This gives us `O(n × log M)`.

2. Constructing `k` maximum numbers: We iterate `k` times, and in each iteration, we check all `31` bit positions to construct the maximum possible number. This gives us `O(k × log M)`.

The total time complexity is `O(n × log M + k × log M)`, which can be simplified to `O((n + k) × log M)`. However, since typically `n ≥ k` in this problem context, and following the reference answer's notation, this is often expressed as `O(n × log M)`.

**Space Complexity:** `O(log M)`

The algorithm uses a fixed-size array `cnt` of length `31` to store the count of set bits at each position. Since `31` represents the number of bits needed to represent the maximum value `M` (approximately `log₂ M`), the space complexity is `O(log M)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Integer Overflow When Calculating Squares**

**Pitfall**: In some programming languages (like C++ or Java), calculating `current_number * current_number` directly can cause integer overflow before applying the modulo operation. Since we're dealing with potentially large numbers (up to 31 bits set), the square could exceed the maximum integer value.

**Solution**: Apply modulo operation more frequently or use appropriate data types:
```python
# Instead of:
total_sum = (total_sum + current_number * current_number) % MOD

# For languages prone to overflow, consider:
# Cast to long/int64 first, or apply modulo in steps:
square = (current_number % MOD) * (current_number % MOD) % MOD
total_sum = (total_sum + square) % MOD
```

### 2. **Incorrect Bit Count Array Size**

**Pitfall**: Using an incorrect size for the bit count array. Some might use 32 bits thinking about 32-bit integers, but Python's integers can be arbitrarily large. However, the problem constraints typically limit the input values.

**Solution**: Check the problem constraints carefully. If numbers can be up to 2^31-1, you need 31 bit positions (0-30). Always verify:
```python
# Ensure bit_count size matches the maximum possible bit position
max_bits = max(num.bit_length() for num in nums) if nums else 0
bit_count = [0] * max(31, max_bits)  # Use at least 31 or the actual maximum needed
```

### 3. **Misunderstanding the Operation's Effect**

**Pitfall**: Thinking that the AND/OR operations change the total number of 1-bits in the system. Some might try to "create" new 1-bits or think bits are lost.

**Solution**: Remember that AND/OR operations only redistribute existing 1-bits:
- Total count of 1-bits at each position remains constant
- We're only moving bits between numbers, not creating or destroying them
- The bit counting approach correctly captures this invariant

### 4. **Not Handling Edge Cases**

**Pitfall**: Failing to handle cases where `k` is larger than the number of elements in `nums`, or when `nums` is empty.

**Solution**: Add validation:
```python
if not nums:
    return 0
  
# If k > len(nums), we can still construct k numbers
# The extra numbers will just be 0 after all bits are used
```

### 5. **Inefficient Bit Checking**

**Pitfall**: Using string conversion or other inefficient methods to check bits:
```python
# Inefficient:
for i, bit in enumerate(bin(num)[2:][::-1]):
    if bit == '1':
        bit_count[i] += 1

# Efficient (as shown in solution):
if num >> bit_position & 1:
    bit_count[bit_position] += 1
```

The bitwise operations are much faster than string manipulation and avoid potential issues with string indexing.

