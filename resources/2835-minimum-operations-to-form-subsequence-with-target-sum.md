# 2835. Minimum Operations to Form Subsequence With Target Sum

> Source: https://algo.monster/liteproblems/2835

## Problem Statement

You have an array `nums` containing only non-negative powers of 2 (like 1, 2, 4, 8, 16, etc.) and a target value `target`.

You can perform the following operation any number of times:
- Pick any element `nums[i]` that is greater than 1
- Remove that element from the array
- Add two copies of `nums[i] / 2` to the end of the array

For example, if you pick the element 8, you remove it and add two 4's to the array.

Your goal is to find the minimum number of operations needed so that you can select some elements from the resulting array (a subsequence) that sum to exactly `target`. If it's impossible to achieve this, return -1.

A subsequence means you can pick any elements from the array while maintaining their relative order - you don't need to pick consecutive elements.

The key insight is that each operation splits a larger power of 2 into two smaller equal powers of 2. For instance:
- Splitting 16 gives you two 8's
- Splitting 8 gives you two 4's  
- Splitting 4 gives you two 2's
- Splitting 2 gives you two 1's

Since powers of 2 have unique binary representations (each has exactly one bit set to 1), this problem essentially becomes about manipulating bits. When you split a power of 2, you're moving a bit from a higher position to a lower position in binary representation.

## Intuition

Let's think about what happens when we perform an operation. When we split a power of 2, we're essentially breaking down a larger power into smaller ones. For example, splitting 8 (which is `2^3`) gives us two 4's (each is `2^2`). The total sum remains the same - we had 8 before, and we have 4 + 4 = 8 after.

This immediately tells us something important: if the sum of all elements in `nums` is less than `target`, it's impossible to reach the target no matter how many operations we perform. The sum never changes!

Now, since we're dealing with powers of 2, let's think in binary. Each power of 2 has exactly one bit set:
- 1 = `2^0` = binary `0001`
- 2 = `2^1` = binary `0010`  
- 4 = `2^2` = binary `0100`
- 8 = `2^3` = binary `1000`

When we split a number, we're moving a bit from position `i` to position `i-1` and creating two copies of it. This is like "breaking a larger bill into smaller denominations" - splitting a $20 bill into two $10 bills.

To build our target sum, we need certain bits to be available. If we look at `target` in binary, each bit position that's set to 1 requires us to have at least one number with that bit available.

The [greedy](/problems/greedy_intro) insight comes from realizing that we should always use the smallest available bit that satisfies our need. Why? Because splitting larger bits is more expensive (requires more operations). If we need a bit at position `i` and we have it available, we use it directly (0 operations). If we don't have it but have a bit at position `j` where `j > i`, we need to split down from position `j` to position `i`, which takes `j - i` operations.

We can also combine smaller bits to form larger ones - two bits at position `i` can combine to form one bit at position `i+1`. This is like combining two $10 bills to make a $20 bill.

The algorithm therefore:
1. Counts how many bits we have at each position across all numbers
2. For each bit needed in `target` (from low to high), finds the nearest available bit
3. Splits down if necessary, keeping track of the operations needed
4. Combines smaller bits when possible to prepare for future needs

**Learn more about [Greedy](/problems/greedy_intro) patterns.**

## Solution Approach

Let's walk through the implementation step by step:

**Step 1: Check if solution is possible**
```python
s = sum(nums)
if s < target:
    return -1
```
Since operations don't change the total sum, if the sum of all elements is less than `target`, it's impossible to form the target.

**Step 2: Count bits at each position**
```python
cnt = [0] * 32
for x in nums:
    for i in range(32):
        if x >> i & 1:
            cnt[i] += 1
```
We create an array `cnt` where `cnt[i]` represents how many times bit `i` appears across all numbers. For each number `x`, we check which bit position has a 1 (since each power of 2 has exactly one bit set).

**Step 3: Process target bits from low to high**
The main loop uses two pointers `i` and `j`:
- `i` points to the current bit position we need in `target`
- `j` points to the position we're currently processing in `cnt`

```python
while 1:
    # Skip bits in target that are 0
    while i < 32 and (target >> i & 1) == 0:
        i += 1
    if i == 32:
        break
```
We skip bit positions in `target` that are 0 since we don't need them.

**Step 4: Combine lower bits if needed**
```python
while j < i:
    cnt[j + 1] += cnt[j] // 2
    cnt[j] %= 2
    j += 1
```
If `j < i`, we have bits at positions lower than what we need. We can combine pairs of bits at position `j` to form bits at position `j+1`. This is like combining two 1's to make a 2, or two 2's to make a 4. We add `cnt[j] // 2` to the next position and keep the remainder at the current position.

**Step 5: Find the nearest available bit**
```python
while cnt[j] == 0:
    cnt[j] = 1
    j += 1
```
If we don't have a bit at position `j`, we need to find the next available bit at a higher position. As we move up, we mark each position with 1 because when we split from position `j` down to position `i`, we'll create bits at all intermediate positions.

**Step 6: Perform the split and count operations**
```python
ans += j - i
cnt[j] -= 1
j = i
i += 1
```
The number of operations needed is `j - i` (the distance we need to split down). We subtract 1 from `cnt[j]` since we used that bit, reset `j` to `i`, and move to the next bit position in `target`.

The algorithm continues until all required bits in `target` are satisfied. The key insight is that we process bits greedily from low to high, always using the nearest available bit and minimizing the number of split operations needed.

## Example Walkthrough

Let's walk through a concrete example to understand how the solution works.

**Example:** `nums = [1, 4, 16]`, `target = 9`

First, let's understand what we need:
- Target 9 in binary is `1001`, which means we need bits at positions 0 and 3 (representing 1 and 8)
- Our initial array has: 1 (bit at position 0), 4 (bit at position 2), 16 (bit at position 4)

**Step 1: Initial Setup**
- Sum check: 1 + 4 + 16 = 21 ≥ 9 ✓ (solution is possible)
- Count bits at each position:
  - `cnt[0] = 1` (from the number 1)
  - `cnt[2] = 1` (from the number 4)
  - `cnt[4] = 1` (from the number 16)
  - All other positions = 0

**Step 2: Process bit position 0 (need 1)**
- We need bit at position 0 for target
- We already have `cnt[0] = 1`
- Use it directly: 0 operations needed
- Update: `cnt[0] = 0`

**Step 3: Process bit position 3 (need 8)**
- Skip positions 1 and 2 in target (they're 0)
- We need bit at position 3 for target
- Check `cnt[3] = 0` (we don't have an 8)
- Look for the next available bit: `cnt[4] = 1` (we have a 16)
- Split 16 → two 8's: 1 operation needed
- Use one 8 for our target
- Update: `cnt[4] = 0`, `cnt[3] = 1` (one 8 remains)

**Final Result:** 1 operation

Let's verify: After 1 operation, our array becomes `[1, 4, 8, 8]`. We can select 1 and 8 to sum to 9.

---

**Another Example:** `nums = [1, 32]`, `target = 12`

Target 12 in binary is `1100`, which means we need bits at positions 2 and 3 (representing 4 and 8).

**Step 1: Initial Setup**
- Sum check: 1 + 32 = 33 ≥ 12 ✓
- Count bits: `cnt[0] = 1`, `cnt[5] = 1`

**Step 2: Process bit position 2 (need 4)**
- Skip positions 0 and 1 in target
- Need bit at position 2
- Check positions: `cnt[2] = 0`, `cnt[3] = 0`, `cnt[4] = 0`, `cnt[5] = 1`
- Split operations needed:
  - 32 → two 16's (position 5 to 4): 1 operation
  - 16 → two 8's (position 4 to 3): 1 operation  
  - 8 → two 4's (position 3 to 2): 1 operation
- Total: 3 operations to get a 4
- Update: `cnt[5] = 0`, `cnt[2] = 1` (one 4 used, one remains)

**Step 3: Process bit position 3 (need 8)**
- Need bit at position 3
- From our previous splits, we have `cnt[3] = 1` (one 8 remains from splitting)
- Use it directly: 0 additional operations

**Final Result:** 3 operations

The algorithm efficiently tracks available bits and performs the minimum splits needed to form the target sum.

## Python Solution

```python
class Solution:
    def minOperations(self, nums: List[int], target: int) -> int:
        # Check if the sum of all numbers is less than target (impossible case)
        total_sum = sum(nums)
        if total_sum < target:
            return -1
      
        # Count frequency of each bit position (0 to 31)
        # bit_count[i] represents how many numbers have bit i set
        bit_count = [0] * 32
        for num in nums:
            for bit_pos in range(32):
                if (num >> bit_pos) & 1:
                    bit_count[bit_pos] += 1
      
        # Initialize pointers and operation counter
        target_bit_pos = 0  # Current bit position in target we're trying to satisfy
        current_bit_pos = 0  # Current bit position we're processing
        operations = 0
      
        while True:
            # Find the next bit position that is set in target
            while target_bit_pos < 32 and ((target >> target_bit_pos) & 1) == 0:
                target_bit_pos += 1
          
            # If we've processed all bits in target, we're done
            if target_bit_pos == 32:
                break
          
            # Carry forward excess bits from lower positions to higher positions
            # Two bits at position i can combine to form one bit at position i+1
            while current_bit_pos < target_bit_pos:
                bit_count[current_bit_pos + 1] += bit_count[current_bit_pos] // 2
                bit_count[current_bit_pos] %= 2
                current_bit_pos += 1
          
            # Find the nearest available bit at or above current position
            # Each split operation increases the position difference
            while bit_count[current_bit_pos] == 0:
                bit_count[current_bit_pos] = 1
                current_bit_pos += 1
          
            # Add the number of split operations needed
            # (difference between where we found the bit and where we need it)
            operations += current_bit_pos - target_bit_pos
          
            # Use one bit from current position
            bit_count[current_bit_pos] -= 1
          
            # Reset current position to target position for next iteration
            current_bit_pos = target_bit_pos
            target_bit_pos += 1
      
        return operations
```

## Time and Space Complexity

**Time Complexity:** `O(n × log M + log target)`

The time complexity consists of two main parts:
- Building the count array: The first loop iterates through all `n` elements in `nums`. For each element `x`, it checks up to 32 bits (or `log M` bits where `M` is the maximum value), resulting in `O(n × log M)` time.
- Processing the target: The while loops process each bit position of the target. In the worst case, this involves iterating through all 32 bit positions with potential carry operations, contributing `O(log target)` time.

Since `log target` is typically bounded by 32 (for 32-bit integers) and is often smaller than `n × log M`, the overall time complexity is `O(n × log M)`.

**Space Complexity:** `O(log M)`

The space complexity is determined by:
- The `cnt` array which has a fixed size of 32 elements, representing bit positions up to `2^31`. This requires `O(32)` = `O(log M)` space, where `M` is the maximum possible value that can be represented.
- A few integer variables (`i`, `j`, `ans`, `s`) which require `O(1)` space.

Therefore, the overall space complexity is `O(log M)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Bit Carrying Logic**
A common mistake is not properly handling the carry-forward of excess bits from lower positions. When we have multiple bits at the same position, we can combine pairs to form bits at higher positions.

**Incorrect approach:**
```python
# Wrong: Only moving to the next position without carrying
while current_bit_pos < target_bit_pos:
    current_bit_pos += 1
```

**Correct approach:**
```python
# Correct: Carry pairs of bits to higher positions
while current_bit_pos < target_bit_pos:
    bit_count[current_bit_pos + 1] += bit_count[current_bit_pos] // 2
    bit_count[current_bit_pos] %= 2
    current_bit_pos += 1
```

### 2. **Not Marking Intermediate Positions During Split Search**
When searching for an available bit at a higher position, forgetting to mark intermediate positions with 1 leads to incorrect operation counting. When we split a higher bit down to a lower position, we create bits at all intermediate positions.

**Incorrect approach:**
```python
# Wrong: Just finding the position without marking intermediate bits
while bit_count[current_bit_pos] == 0:
    current_bit_pos += 1
```

**Correct approach:**
```python
# Correct: Mark each intermediate position with 1
while bit_count[current_bit_pos] == 0:
    bit_count[current_bit_pos] = 1  # This is crucial!
    current_bit_pos += 1
```

### 3. **Overflow with Large Numbers**
Using 31 or fewer bits might cause issues with the maximum possible values. Always use 32 bits to handle numbers up to 2^31.

**Incorrect approach:**
```python
bit_count = [0] * 30  # Not enough bits
```

**Correct approach:**
```python
bit_count = [0] * 32  # Sufficient for all valid inputs
```

### 4. **Not Resetting the Current Position Pointer**
After using a bit, failing to reset `current_bit_pos` to `target_bit_pos` causes the algorithm to skip necessary carry operations in subsequent iterations.

**Incorrect approach:**
```python
operations += current_bit_pos - target_bit_pos
bit_count[current_bit_pos] -= 1
# Forgot to reset current_bit_pos
target_bit_pos += 1
```

**Correct approach:**
```python
operations += current_bit_pos - target_bit_pos
bit_count[current_bit_pos] -= 1
current_bit_pos = target_bit_pos  # Reset to process next target bit
target_bit_pos += 1
```

### 5. **Misunderstanding the Problem Constraints**
Some might think we need to use ALL elements to form the target, but we only need to select a subsequence. The check should only ensure the total sum is at least the target, not exactly the target.

**Incorrect approach:**
```python
if sum(nums) != target:  # Wrong: We don't need exact sum
    return -1
```

**Correct approach:**
```python
if sum(nums) < target:  # Correct: Just need enough sum available
    return -1
```

