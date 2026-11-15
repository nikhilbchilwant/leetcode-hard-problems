# 3181. Maximum Total Reward Using Operations II

> Source: https://algo.monster/liteproblems/3181

## Problem Statement

You have an integer array `rewardValues` of length `n`, where each element represents a reward value. You start with a total reward `x = 0`, and all indices in the array are initially unmarked.

You can perform the following operation any number of times:
- Select any unmarked index `i` from the range `[0, n - 1]`
- If `rewardValues[i]` is greater than your current total reward `x`, you can:
  - Add `rewardValues[i]` to your total (i.e., `x = x + rewardValues[i]`)
  - Mark index `i` as used

The constraint is that you can only collect a reward if its value is strictly greater than your current total reward. Once you collect a reward, that index becomes marked and cannot be used again.

Your goal is to find the maximum possible total reward you can achieve by choosing rewards optimally.

For example, if `rewardValues = [1, 6, 4, 3, 2]`:
- You could start by taking reward value `1` (since `1 > 0`), making your total `1`
- Then take reward value `2` (since `2 > 1`), making your total `3`
- Then take reward value `4` (since `4 > 3`), making your total `7`
- You cannot take reward value `6` because `6 < 7`
- You cannot take reward value `3` because `3 < 7`

## Intuition

The key observation is that once we decide to take a reward value `v`, we can only take it if our current total is less than `v`. This means we need to track all possible total rewards we can achieve at each step.

First, let's think about the order of taking rewards. If we have two rewards `a` and `b` where `a < b`, and we want to take both, we must take `a` first. Why? Because if we take `b` first, our total becomes at least `b`, which means we can never take `a` afterwards (since `a < b` means `a` is not greater than our current total). This suggests we should consider rewards in sorted order.

Next, we notice that duplicate values don't help us - if we can take a reward of value `v`, we only need to take it once. Taking the same value multiple times doesn't change what future rewards we can access. So we can work with unique values only.

Now, the core insight: this is a reachability problem. We want to know which total reward values are reachable. We can think of it as a [dynamic programming](/problems/dynamic_programming_intro) problem where `f[j]` represents whether we can achieve a total reward of exactly `j`.

For each reward value `v` in sorted order:
- We can only add `v` to our total if our current total is less than `v`
- If we can reach total `j` where `j < v`, then we can also reach total `j + v`

This leads to the state transition: for each reward `v`, we look at all reachable totals less than `v` and mark `total + v` as reachable.

The bit manipulation optimization comes from representing the reachability state as a binary number. If bit `i` is set in our number `f`, it means total reward `i` is reachable. When processing reward `v`:
- `(1 << v) - 1` creates a mask for all bits less than `v`
- `f & ((1 << v) - 1)` extracts all reachable totals less than `v`
- Shifting this left by `v` bits represents adding `v` to each of these totals
- OR-ing with the original `f` combines the new reachable states with existing ones

The maximum reachable total is the position of the highest set bit, which we get using `bit_length() - 1`.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution uses [dynamic programming](/problems/dynamic_programming_intro) with bit manipulation optimization. Let's break down the implementation step by step:

**Step 1: Preprocessing**
```python
nums = sorted(set(rewardValues))
```
- Remove duplicates using `set()` since taking the same value multiple times doesn't provide additional benefit
- Sort the values in ascending order to ensure we process smaller rewards first

**Step 2: Initialize the DP state**
```python
f = 1
```
- We use a binary number `f` where the `i`-th bit being 1 means a total reward of `i` is reachable
- Initially, `f = 1` means only bit 0 is set, representing that a total of 0 is reachable (our starting state)

**Step 3: Process each reward value**
```python
for v in nums:
    f |= (f & ((1 << v) - 1)) << v
```
This is the core of the algorithm. For each reward value `v`, we:

1. **Create a mask for valid current totals**: `(1 << v) - 1`
   - This creates a number with all bits from 0 to `v-1` set to 1
   - For example, if `v = 5`, then `(1 << 5) - 1 = 31 = 11111` in binary

2. **Extract reachable totals less than v**: `f & ((1 << v) - 1)`
   - This AND operation keeps only the bits in `f` that represent totals less than `v`
   - These are the totals from which we can take reward `v`

3. **Calculate new reachable totals**: `<< v`
   - Shift the result left by `v` positions
   - If bit `j` was set (meaning total `j` was reachable and `j < v`), now bit `j + v` is set
   - This represents adding reward `v` to each valid current total

4. **Update the state**: `f |= ...`
   - OR the new reachable totals with the existing state
   - This preserves all previously reachable totals while adding the new ones

**Step 4: Extract the answer**
```python
return f.bit_length() - 1
```
- `bit_length()` returns the number of bits needed to represent `f`
- Subtracting 1 gives us the position of the highest set bit, which is the maximum reachable total reward

**Example Walkthrough:**
If `rewardValues = [1, 4, 3]`, after removing duplicates and sorting: `nums = [1, 3, 4]`

- Initially: `f = 1` (binary: `1`)
- Process `v = 1`: `f = 1 | ((1 & 0) << 1) = 1 | 0 = 1 | 10 = 11` (can reach 0 and 1)
- Process `v = 3`: `f = 11 | ((11 & 111) << 3) = 11 | (11 << 3) = 11 | 11000 = 11011` (can reach 0, 1, 3, 4)
- Process `v = 4`: `f = 11011 | ((11011 & 1111) << 4) = 11011 | (1011 << 4) = 11011 | 10110000 = 10111011` (can reach 0, 1, 3, 4, 5, 7)
- Answer: `bit_length(10111011) - 1 = 8 - 1 = 7`

The time complexity is `O(n * max(rewardValues))` for the bit operations, and space complexity is `O(1)` since we use a single integer to store the state.

## Example Walkthrough

Let's walk through a small example with `rewardValues = [1, 1, 3, 3]` to illustrate the solution approach.

**Step 1: Preprocessing**
- Remove duplicates and sort: `nums = [1, 3]`
- We only need unique values since taking duplicates doesn't help us reach higher totals

**Step 2: Initialize**
- Start with `f = 1` (binary: `0001`)
- This means we can reach a total of 0 (bit 0 is set)

**Step 3: Process each reward**

**Processing v = 1:**
- Current state: `f = 0001` (can reach total 0)
- Create mask: `(1 << 1) - 1 = 1` (binary: `0001`)
- Find valid totals: `f & 1 = 0001 & 0001 = 0001` (total 0 is valid since 0 < 1)
- Calculate new totals: `0001 << 1 = 0010` (if we add 1 to total 0, we get total 1)
- Update state: `f = 0001 | 0010 = 0011`
- Now we can reach totals: 0 and 1

**Processing v = 3:**
- Current state: `f = 0011` (can reach totals 0 and 1)
- Create mask: `(1 << 3) - 1 = 7` (binary: `0111`)
- Find valid totals: `f & 7 = 0011 & 0111 = 0011` (totals 0 and 1 are both < 3)
- Calculate new totals: `0011 << 3 = 0011000` (adding 3 to totals 0 and 1 gives us 3 and 4)
- Update state: `f = 0011 | 0011000 = 0011011`
- Now we can reach totals: 0, 1, 3, and 4

**Step 4: Extract answer**
- Final state: `f = 0011011` (binary)
- The highest set bit is at position 4 (representing total 4)
- `bit_length(0011011) - 1 = 5 - 1 = 4`
- Maximum total reward = 4

**How we achieved this total:**
Starting with x = 0:
1. Take the first occurrence of reward 1 (since 1 > 0), now x = 1
2. Take the first occurrence of reward 3 (since 3 > 1), now x = 4
3. We cannot take any more rewards since all remaining values (the duplicate 1 and 3) are not greater than our current total of 4

The bit manipulation elegantly tracks all possible totals we can reach, ensuring we find the optimal sequence to maximize our reward.

## Python Solution

```python
class Solution:
    def maxTotalReward(self, rewardValues: List[int]) -> int:
        # Remove duplicates and sort the reward values in ascending order
        unique_rewards = sorted(set(rewardValues))
      
        # Initialize a bitmask where bit i represents whether sum i is achievable
        # Start with bit 0 set (sum of 0 is always achievable)
        achievable_sums = 1
      
        # Process each reward value in ascending order
        for reward in unique_rewards:
            # Find all sums less than current reward: (1 << reward) - 1 creates a mask with reward rightmost bits set
            # AND with achievable_sums to get achievable sums less than reward
            valid_previous_sums = achievable_sums & ((1 << reward) - 1)
          
            # For each valid previous sum, we can add the current reward
            # Shifting left by reward adds reward to each of those sums
            new_achievable_sums = valid_previous_sums << reward
          
            # Update achievable_sums to include the new sums
            achievable_sums |= new_achievable_sums
      
        # The highest set bit position minus 1 gives the maximum achievable sum
        # bit_length() returns the number of bits needed to represent the number
        return achievable_sums.bit_length() - 1
```

## Time and Space Complexity

**Time Complexity:** `O(n × M / w)`

The algorithm first creates a sorted set from `rewardValues`, which takes `O(n log n)` time. Then it iterates through each unique value in the sorted set. For each value `v`, it performs bitwise operations on an integer `f`:
- `(1 << v) - 1` creates a bitmask with `v` bits set to 1
- `f & ((1 << v) - 1)` performs a bitwise AND operation
- The result is then left-shifted by `v` positions
- Finally, a bitwise OR operation updates `f`

Since `f` can grow up to `2M` bits (where `M` is the maximum value in `rewardValues`), and bitwise operations on large integers are handled in chunks of word size `w` (32 or 64 bits), each bitwise operation takes `O(M / w)` time. With `n` unique values to process, the total time complexity is `O(n × M / w)`.

**Space Complexity:** `O(n + M / w)`

The space is used for:
- Storing the sorted set of unique values: `O(n)`
- The integer `f` which can grow up to `2M` bits, requiring `O(M / w)` words of memory

Therefore, the total space complexity is `O(n + M / w)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Not Removing Duplicates**
One of the most common mistakes is forgetting to remove duplicate values from the input array. If you have multiple instances of the same reward value, you might think you can collect them multiple times, but the algorithm's logic doesn't work that way.

**Why it's a problem:** 
- If `rewardValues = [2, 2, 3]`, processing the second `2` is redundant since once your total exceeds 2, you can never collect a reward of value 2 again
- This leads to unnecessary computation and potentially incorrect state tracking

**Solution:**
Always use `set()` to remove duplicates before processing:
```python
unique_rewards = sorted(set(rewardValues))
```

### 2. **Integer Overflow with Large Values**
When reward values are very large (up to 2000 in typical constraints), the bitmask `achievable_sums` can become extremely large. The expression `(1 << reward)` creates a number with `reward` bits, which grows exponentially.

**Why it's a problem:**
- For large reward values, the bitmask can exceed typical integer limits in some languages
- Python handles arbitrary precision integers, but other languages might overflow

**Solution:**
- In Python, this is handled automatically
- In other languages, consider using alternative DP approaches with arrays or sets when values are large
- Add a check for maximum possible sum to avoid unnecessary computation:
```python
if reward > 2000:  # or whatever your constraint is
    # Handle edge case or use alternative approach
```

### 3. **Incorrect Mask Creation**
A subtle but critical error is using the wrong mask to find valid previous sums. Some might mistakenly write:
```python
# WRONG: This doesn't create the correct mask
valid_previous_sums = achievable_sums & (1 << (reward - 1))
```

**Why it's a problem:**
- `1 << (reward - 1)` creates a single bit at position `reward-1`, not a mask of all bits below `reward`
- This would only check if sum `reward-1` is achievable, missing all other valid sums

**Solution:**
Always use `(1 << reward) - 1` to create a mask with all bits from 0 to `reward-1` set:
```python
# CORRECT: Creates a mask with all bits 0 to reward-1 set to 1
valid_previous_sums = achievable_sums & ((1 << reward) - 1)
```

### 4. **Processing Values in Wrong Order**
Processing reward values without sorting them first will produce incorrect results.

**Why it's a problem:**
- The algorithm relies on processing smaller values first to build up the achievable sums correctly
- If you process a large value before smaller ones, you might miss valid combinations

**Solution:**
Always sort the values in ascending order:
```python
unique_rewards = sorted(set(rewardValues))
```

### 5. **Off-by-One Error in Final Answer**
Forgetting to subtract 1 from `bit_length()` is a common mistake:
```python
# WRONG: This returns the number of bits, not the maximum sum
return achievable_sums.bit_length()

# CORRECT: Subtract 1 to get the position of the highest bit
return achievable_sums.bit_length() - 1
```

**Why it's a problem:**
- `bit_length()` returns the number of bits needed to represent the number
- For example, if the maximum sum is 7 (binary: 111), `bit_length()` returns 3, but we want 7
- The highest set bit position is at index 7, which is `bit_length() - 1 = 8 - 1 = 7`

