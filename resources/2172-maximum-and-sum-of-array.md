# 2172. Maximum AND Sum of Array

> Source: https://algo.monster/liteproblems/2172

## Problem Statement

You have an integer array `nums` with `n` elements and a number of slots `numSlots`. The constraint guarantees that `2 * numSlots >= n`, meaning you have enough space to place all numbers.

Each slot is numbered from `1` to `numSlots`, and each slot can hold **at most two numbers** from the array.

Your task is to distribute all `n` numbers from `nums` into these slots to maximize the **AND sum**.

The **AND sum** is calculated by:
- Taking each number you've placed
- Performing a bitwise AND operation between that number and its slot number
- Summing all these AND results

For example, if you place numbers `[1, 3]` in slot `1` and `[4, 6]` in slot `2`:
- `1 AND 1 = 1`
- `3 AND 1 = 1` 
- `4 AND 2 = 0`
- `6 AND 2 = 2`
- Total AND sum = `1 + 1 + 0 + 2 = 4`

The goal is to find the optimal placement of all numbers that gives you the maximum possible AND sum.

## Intuition

The key insight is that we need to try different assignments of numbers to slots to find the maximum AND sum. Since each slot can hold up to 2 numbers and we have `numSlots` slots, we essentially have `2 * numSlots` positions available.

We can think of this as having `2 * numSlots` distinct positions where we can place our numbers. Position `0` and `1` belong to slot `1`, positions `2` and `3` belong to slot `2`, and so on. For position `j`, the corresponding slot number is `j // 2 + 1`.

The challenge is exploring all possible placements efficiently. This is where [bitmask](/problems/bitmask_intro) [dynamic programming](/problems/dynamic_programming_intro) comes in. We can use a bitmask of length `2 * numSlots` where each bit represents whether a position is occupied or not.

For a given [bitmask](/problems/bitmask_intro) `i`, the number of `1` bits tells us how many numbers from `nums` we've placed so far. If we've placed `cnt` numbers, they must be the first `cnt` numbers from the array (we can process them in order since the final AND sum doesn't depend on which specific number goes where, only on the pairing of numbers with slots).

The [dynamic programming](/problems/dynamic_programming_intro) approach works backwards: for each state with `cnt` numbers placed, we try removing the last placed number (`nums[cnt-1]`) from each possible position. If position `j` is occupied in the current state (bit `j` is set), we can compute what the maximum sum would be if we remove that number from position `j`. The AND contribution of that number would be `nums[cnt-1] & (j // 2 + 1)`.

By iterating through all valid states (where the number of placed items doesn't exceed `n`) and trying all possible last placements, we build up the maximum AND sum for each configuration. The answer is the maximum value among all states in our DP table.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) and [Bitmask](/problems/bitmask_intro) patterns.**

## Solution Approach

The solution uses [bitmask](/problems/bitmask_intro) [dynamic programming](/problems/dynamic_programming_intro) to explore all possible placements of numbers into slots.

**State Representation:**
- We create an array `f` of size `1 << m` where `m = numSlots * 2` (total positions available)
- `f[i]` stores the maximum AND sum achievable when the positions indicated by [bitmask](/problems/bitmask_intro) `i` are occupied
- Each bit in the bitmask represents whether a position is filled (1) or empty (0)

**Implementation Steps:**

1. **Initialize variables:**
   - `n = len(nums)` - number of elements to place
   - `m = numSlots << 1` - total positions (2 per slot)
   - `f = [0] * (1 << m)` - DP array initialized with zeros

2. **Iterate through all possible states:**
   ```python
   for i in range(1 << m):
   ```
   Each `i` represents a different configuration of filled positions.

3. **Count placed numbers:**
   ```python
   cnt = i.bit_count()
   if cnt > n:
       continue
   ```
   - `bit_count()` returns the number of 1s in the [bitmask](/problems/bitmask_intro)
   - Skip states where we've placed more numbers than available

4. **Try removing each placed number:**
   ```python
   for j in range(m):
       if i >> j & 1:
   ```
   - Check if position `j` is occupied in state `i`
   - If yes, calculate the maximum sum by considering this as the last placement

5. **Update DP value:**
   ```python
   f[i] = max(f[i], f[i ^ (1 << j)] + (nums[cnt - 1] & (j // 2 + 1)))
   ```
   - `i ^ (1 << j)` gives the previous state with position `j` empty
   - `nums[cnt - 1]` is the last number placed (we place numbers in order)
   - `j // 2 + 1` converts position index to slot number (positions 0,1 → slot 1, positions 2,3 → slot 2, etc.)
   - The AND operation `nums[cnt - 1] & (j // 2 + 1)` calculates the contribution

6. **Return the maximum:**
   ```python
   return max(f)
   ```
   The answer is the maximum value across all valid states in the DP table.

**Time Complexity:** `O(2^(2*numSlots) * 2*numSlots)` - we iterate through all states and for each state, check all positions.

**Space Complexity:** `O(2^(2*numSlots))` - for storing the DP array.

## Example Walkthrough

Let's walk through a small example with `nums = [3, 5]` and `numSlots = 2`.

**Setup:**
- We have 2 numbers to place: [3, 5]
- We have 2 slots (slot 1 and slot 2), each can hold up to 2 numbers
- Total positions available: `m = 2 * 2 = 4` positions
  - Positions 0,1 belong to slot 1
  - Positions 2,3 belong to slot 2
- DP array size: `2^4 = 16` states

**Key States Walkthrough:**

Initially, `f[0000] = 0` (no numbers placed, AND sum = 0)

**Placing 1 number (cnt = 1, placing nums[0] = 3):**
- State `0001` (position 0 filled): 
  - `f[0001] = f[0000] + (3 & 1) = 0 + 1 = 1`
- State `0010` (position 1 filled):
  - `f[0010] = f[0000] + (3 & 1) = 0 + 1 = 1`
- State `0100` (position 2 filled):
  - `f[0100] = f[0000] + (3 & 2) = 0 + 2 = 2`
- State `1000` (position 3 filled):
  - `f[1000] = f[0000] + (3 & 2) = 0 + 2 = 2`

**Placing 2 numbers (cnt = 2, placing nums[1] = 5):**
- State `0011` (positions 0,1 filled - both in slot 1):
  - From `0001`: `f[0011] = f[0001] + (5 & 1) = 1 + 1 = 2`
  - From `0010`: `f[0011] = f[0010] + (5 & 1) = 1 + 1 = 2`
- State `0101` (positions 0,2 filled - one in each slot):
  - From `0001`: `f[0101] = f[0001] + (5 & 2) = 1 + 0 = 1`
  - From `0100`: `f[0101] = f[0100] + (5 & 1) = 2 + 1 = 3`
  - Maximum: `f[0101] = 3`
- State `0110` (positions 1,2 filled):
  - From `0010`: `f[0110] = f[0010] + (5 & 2) = 1 + 0 = 1`
  - From `0100`: `f[0110] = f[0100] + (5 & 1) = 2 + 1 = 3`
  - Maximum: `f[0110] = 3`
- State `1001` (positions 0,3 filled):
  - From `0001`: `f[1001] = f[0001] + (5 & 2) = 1 + 0 = 1`
  - From `1000`: `f[1001] = f[1000] + (5 & 1) = 2 + 1 = 3`
  - Maximum: `f[1001] = 3`
- State `1100` (positions 2,3 filled - both in slot 2):
  - From `0100`: `f[1100] = f[0100] + (5 & 2) = 2 + 0 = 2`
  - From `1000`: `f[1100] = f[1000] + (5 & 2) = 2 + 0 = 2`

**Result:**
The maximum value in the DP array is `3`, which occurs when:
- We place 3 in slot 2 (3 & 2 = 2)
- We place 5 in slot 1 (5 & 1 = 1)
- Total AND sum = 2 + 1 = 3

This demonstrates how the algorithm explores all possible placements and finds the optimal assignment by building up from smaller states to larger ones.

## Python Solution

```python
class Solution:
    def maximumANDSum(self, nums: List[int], numSlots: int) -> int:
        """
        Assigns numbers to slots to maximize the sum of (number AND slot_number).
        Each slot can hold at most 2 numbers.
      
        Args:
            nums: List of integers to assign to slots
            numSlots: Number of available slots (numbered 1 to numSlots)
      
        Returns:
            Maximum possible sum of (number AND slot_number) for all assignments
        """
        num_count = len(nums)
        # Total positions available (2 positions per slot)
        total_positions = numSlots * 2
      
        # dp[mask] = maximum AND sum when positions indicated by mask are filled
        # mask bit i represents whether position i is occupied
        dp = [0] * (1 << total_positions)
      
        # Iterate through all possible states (masks)
        for mask in range(1 << total_positions):
            # Count how many positions are filled in current mask
            filled_positions = mask.bit_count()
          
            # Skip if we're trying to place more numbers than available
            if filled_positions > num_count:
                continue
          
            # Try placing the current number at each possible position
            for position in range(total_positions):
                # Check if this position is occupied in current mask
                if (mask >> position) & 1:
                    # Calculate which slot this position belongs to (1-indexed)
                    slot_number = (position // 2) + 1
                  
                    # Previous state without this position filled
                    previous_mask = mask ^ (1 << position)
                  
                    # Current number being placed (0-indexed in nums)
                    current_num = nums[filled_positions - 1]
                  
                    # Update dp with maximum value
                    dp[mask] = max(
                        dp[mask], 
                        dp[previous_mask] + (current_num & slot_number)
                    )
      
        # Return the maximum value across all valid states
        return max(dp)
```

## Time and Space Complexity

**Time Complexity:** `O(2^(2*numSlots) * numSlots)`

The algorithm iterates through all possible states represented by a bitmask of size `2 * numSlots`. For each state `i` in the range `[0, 2^(2*numSlots))`, it performs the following operations:
- `bit_count()` operation: `O(1)` in Python 3.10+
- Inner loop iterating through `m = 2 * numSlots` positions: `O(numSlots)`
- Inside the inner loop: bit operations and array access are `O(1)`

Therefore, the total time complexity is `O(2^(2*numSlots) * numSlots)`.

**Space Complexity:** `O(2^(2*numSlots))`

The algorithm uses:
- Array `f` of size `2^(2*numSlots)` to store the DP states: `O(2^(2*numSlots))`
- A few auxiliary variables (`n`, `m`, `cnt`, `i`, `j`): `O(1)`

The dominant factor is the DP array, resulting in a space complexity of `O(2^(2*numSlots))`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Incorrect Slot Number Calculation

**Problem:** A common mistake is incorrectly mapping position indices to slot numbers. Since each slot has 2 positions, developers often confuse the zero-based position indexing with one-based slot numbering.

**Incorrect Implementation:**
```python
# Wrong: forgetting to add 1 for 1-based slot numbering
slot_number = position // 2  # This gives 0-based slot index

# Wrong: incorrect division for position-to-slot mapping
slot_number = position / 2 + 1  # Using float division instead of integer
```

**Correct Implementation:**
```python
slot_number = (position // 2) + 1  # Correctly converts to 1-based slot number
```

### Pitfall 2: Misunderstanding Number Placement Order

**Problem:** The algorithm assumes numbers are placed in the order they appear in the input array. Some developers mistakenly think they can place numbers in any order, leading to incorrect indexing when accessing `nums[filled_positions - 1]`.

**Why This Matters:**
- The DP state only tracks which positions are filled, not which specific numbers are in those positions
- By placing numbers in order, we know that if `k` positions are filled, we've placed `nums[0]` through `nums[k-1]`
- Trying to optimize by reordering numbers would require a completely different state representation

**Solution:** Always place numbers sequentially from the input array. If you want to explore different orderings, you'd need to modify the entire DP approach to track which specific numbers have been placed.

### Pitfall 3: Overflow with Large numSlots

**Problem:** The space complexity is `O(2^(2*numSlots))`, which grows exponentially. For `numSlots = 16`, this requires `2^32` array elements, consuming gigabytes of memory.

**Symptoms:**
- Memory allocation failures
- Program crashes or timeouts
- Python MemoryError exceptions

**Solution:** 
1. Check problem constraints carefully - most problems limit `numSlots` to reasonable values (typically ≤ 9)
2. If larger values are needed, consider alternative approaches like branch-and-bound or greedy heuristics
3. Add validation:
```python
if numSlots > 15:  # Practical limit for bitmask DP
    raise ValueError("numSlots too large for bitmask DP approach")
```

### Pitfall 4: Confusion About State Representation

**Problem:** Misunderstanding what each bit in the mask represents. Some developers think each bit represents a slot (not a position), leading to incorrect state transitions.

**Incorrect Mental Model:**
- Thinking mask bit `i` represents whether slot `i` has any numbers
- This would only allow tracking whether slots are used, not how many numbers each contains

**Correct Understanding:**
- Each slot has exactly 2 positions (even if not fully utilized)
- Positions 0,1 belong to slot 1; positions 2,3 belong to slot 2, etc.
- The mask tracks individual positions, allowing us to know if a slot has 0, 1, or 2 numbers

**Debugging Tip:** Add helper visualization:
```python
def visualize_mask(mask, total_positions):
    positions = []
    for i in range(total_positions):
        if (mask >> i) & 1:
            slot = (i // 2) + 1
            pos_in_slot = i % 2
            positions.append(f"Slot{slot}_Pos{pos_in_slot}")
    return positions
```

