# 3209. Number of Subarrays With AND Value of K

> Source: https://algo.monster/liteproblems/3209

## Problem Statement

Given an array of integers `nums` and an integer `k`, you need to find the total number of contiguous subarrays where the bitwise AND of all elements in the subarray equals `k`.

A subarray is a contiguous part of the array. For example, if `nums = [1, 2, 3]`, the subarrays are `[1]`, `[2]`, `[3]`, `[1,2]`, `[2,3]`, and `[1,2,3]`.

The bitwise AND operation takes two numbers and performs AND on each bit position. For a subarray from index `l` to `r`, you calculate `nums[l] & nums[l+1] & ... & nums[r]` and check if this result equals `k`.

For example, if `nums = [1, 1, 1]` and `k = 1`:
- Subarray `[1]` has AND value = `1` (appears 3 times)
- Subarray `[1, 1]` has AND value = `1 & 1 = 1` (appears 2 times)
- Subarray `[1, 1, 1]` has AND value = `1 & 1 & 1 = 1` (appears 1 time)
- Total count = 6

## Intuition

The key insight is that bitwise AND has a special property: it can only decrease or stay the same as we add more elements, never increase. This is because AND operation can only turn bits from 1 to 0, never from 0 to 1.

Consider what happens when we fix the right endpoint of our subarray at position `i`. All possible subarrays ending at `i` have left endpoints ranging from `0` to `i`. As we extend the left boundary further left (including more elements), the AND value either stays the same or decreases.

Since each number in the array is at most `10^9` (roughly 30 bits), and each AND operation can potentially turn off at least one bit, there can be at most 30 distinct AND values for all subarrays ending at any position. This is a crucial observation that makes the problem tractable.

Instead of checking all possible subarrays (which would be O(n²)), we can maintain a frequency map of all distinct AND values for subarrays ending at the current position. When we move to the next position `i+1`:
- Each existing AND value gets combined with `nums[i+1]` through AND operation
- We also start a new subarray containing just `nums[i+1]`

This way, we only need to update at most 30 values per position, making the algorithm efficient. We use a Counter (hash table) to track how many subarrays produce each AND value. Whenever we see the value `k` in our counter, we add its frequency to our answer.

The transition from position `i` to `i+1` naturally maintains all possible AND values and their counts, allowing us to solve the problem in a single pass through the array.

**Learn more about [Segment Tree](/problems/segment_tree_intro) and [Binary Search](/problems/binary_search_intro) patterns.**

## Solution Approach

The solution uses a hash table to track all possible AND values for subarrays ending at each position. Here's how the implementation works:

We maintain two counters:
- `pre`: stores the frequency of AND values for subarrays ending at the previous position
- `cur`: stores the frequency of AND values for subarrays ending at the current position

For each element `x` in `nums`:

1. **Create a new counter for current position**: Initialize `cur = Counter()` to store AND values ending at this position.

2. **Update existing subarrays**: For each AND value `y` in `pre` with frequency `v`:
   - Calculate `x & y` - this extends all subarrays ending at the previous position to include the current element
   - Add the frequency to `cur`: `cur[x & y] += v`
   - This handles all subarrays that started before the current position

3. **Add single-element subarray**: Increment `cur[x] += 1` to account for the subarray containing only the current element.

4. **Count subarrays with AND = k**: Add `cur[k]` to the answer. This gives us the number of subarrays ending at the current position with AND value equal to `k`.

5. **Move to next position**: Set `pre = cur` to prepare for the next iteration.

The algorithm processes each element once, and for each element, it updates at most 30 distinct AND values (due to the bit limitation). This gives us a time complexity of O(n × 30) = O(n).

For example, with `nums = [1, 1, 2]` and `k = 0`:
- Position 0: `cur = {1: 1}`, no subarrays with AND = 0
- Position 1: `cur = {1: 2}` (from `1&1=1` twice), no subarrays with AND = 0
- Position 2: `cur = {0: 2, 2: 1}` (from `1&2=0` twice, and single `2`), add 2 to answer

The space complexity is O(30) = O(1) since we store at most 30 distinct values in the counter.

## Example Walkthrough

Let's walk through the solution with `nums = [3, 7, 3, 1]` and `k = 3`.

**Initial Setup:**
- `ans = 0` (our answer counter)
- `pre = Counter()` (empty, tracks AND values from previous position)

**Processing nums[0] = 3:**
- Create new `cur = Counter()`
- No previous values in `pre` to extend
- Add single-element subarray: `cur[3] += 1`, so `cur = {3: 1}`
- Check if k=3 exists: `cur[3] = 1`, so `ans += 1` → `ans = 1`
- Update `pre = cur = {3: 1}`

**Processing nums[1] = 7:**
- Create new `cur = Counter()`
- Extend previous subarrays: For `3` in `pre` with frequency 1:
  - Calculate `7 & 3 = 3` (binary: 111 & 011 = 011)
  - Add to cur: `cur[3] += 1`, so `cur = {3: 1}`
- Add single-element subarray: `cur[7] += 1`, so `cur = {3: 1, 7: 1}`
- Check if k=3 exists: `cur[3] = 1`, so `ans += 1` → `ans = 2`
- Update `pre = cur = {3: 1, 7: 1}`

**Processing nums[2] = 3:**
- Create new `cur = Counter()`
- Extend previous subarrays:
  - For `3` in `pre`: `3 & 3 = 3`, so `cur[3] += 1`
  - For `7` in `pre`: `3 & 7 = 3`, so `cur[3] += 1`
  - Now `cur = {3: 2}`
- Add single-element subarray: `cur[3] += 1`, so `cur = {3: 3}`
- Check if k=3 exists: `cur[3] = 3`, so `ans += 3` → `ans = 5`
- Update `pre = cur = {3: 3}`

**Processing nums[3] = 1:**
- Create new `cur = Counter()`
- Extend previous subarrays:
  - For `3` in `pre` with frequency 3: `1 & 3 = 1` (binary: 001 & 011 = 001)
  - So `cur[1] += 3`, giving `cur = {1: 3}`
- Add single-element subarray: `cur[1] += 1`, so `cur = {1: 4}`
- Check if k=3 exists: `cur[3] = 0`, so `ans += 0` → `ans = 5`

**Final Answer: 5**

The 5 subarrays with AND value equal to 3 are:
1. `[3]` at index 0
2. `[3, 7]` from indices 0-1
3. `[7, 3]` from indices 1-2
4. `[3]` at index 2
5. `[3, 7, 3]` from indices 0-2

## Python Solution

```python
from typing import List
from collections import Counter

class Solution:
    def countSubarrays(self, nums: List[int], k: int) -> int:
        # Total count of valid subarrays
        result = 0

        # Store AND results of subarrays ending at previous position
        # Key: AND result, Value: count of subarrays with that AND result
        previous_and_counts = Counter()

        # Process each element in the array
        for current_num in nums:
            # Store AND results of subarrays ending at current position
            current_and_counts = Counter()

            # Extend all previous subarrays by including current element
            # For each previous AND result, compute new AND with current element
            for and_result, count in previous_and_counts.items():
                new_and_result = current_num & and_result
                current_and_counts[new_and_result] += count

            # Add subarray containing only the current element
            current_and_counts[current_num] += 1

            # Count how many subarrays ending at current position have AND equal to k
            result += current_and_counts[k]

            # Update previous counts for next iteration
            previous_and_counts = current_and_counts

        return result
```

## Time and Space Complexity

The time complexity is `O(n × log M)`, where `n` is the length of the array `nums` and `M` is the maximum value in the array `nums`.

The key insight is that for any number, the bitwise AND operation can only decrease or maintain the value, never increase it. Starting from any value `x`, repeatedly applying AND operations will create at most `O(log M)` distinct values before reaching 0, since each bit position can only change from 1 to 0 (never from 0 to 1).

In the algorithm:
- The outer loop iterates through all `n` elements
- For each element, we iterate through the `pre` Counter, which contains at most `O(log M)` distinct AND results
- Each operation inside the loops takes `O(1)` time

Therefore, the total time complexity is `O(n × log M)`.

The space complexity is `O(log M)`. At any point in the algorithm, the Counter `cur` (and subsequently `pre`) stores at most `O(log M)` distinct values, representing all possible AND results of subarrays ending at the current position. Since we only maintain two Counters (`pre` and `cur`) and each can have at most `O(log M)` entries, the space complexity is `O(log M)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Not Handling the Case Where k Has Bits That Don't Exist in Any Array Element**

A common mistake is not recognizing that if `k` has any bit set to 1 that is never set to 1 in any element of the array, the answer should immediately be 0. The bitwise AND operation can only preserve or clear bits, never set new ones.

**Pitfall Example:**
```python
# If nums = [4, 2, 8] (binary: [100, 010, 1000]) and k = 3 (binary: 011)
# k requires bits 0 and 1 to be set, but no element has both bits set
# The AND of any subarray cannot produce 3
```

**Solution:**
Add an early termination check:
```python
def countSubarrays(self, nums: List[int], k: int) -> int:
    # Check if k is achievable by ANDing all elements
    all_bits_or = 0
    for num in nums:
        all_bits_or |= num

    # If k has bits that don't exist in any element, return 0
    if k & ~all_bits_or != 0:
        return 0

    # Continue with the original algorithm...
```

### 2. **Assuming AND Values Always Decrease Monotonically**

While it's true that extending a subarray can only maintain or decrease the AND value, developers might incorrectly assume they can break early when the AND becomes less than `k`. However, this is wrong because we need to count ALL subarrays, not just find one.

**Pitfall Example:**
```python
# WRONG: Breaking early when AND < k
for and_result, count in previous_and_counts.items():
    new_and_result = current_num & and_result
    if new_and_result < k:  # WRONG!
        break  # This skips valid counting
    current_and_counts[new_and_result] += count
```

**Solution:**
Process all entries without early termination based on the AND value:
```python
# CORRECT: Process all previous AND values
for and_result, count in previous_and_counts.items():
    new_and_result = current_num & and_result
    current_and_counts[new_and_result] += count
```

### 3. **Memory Inefficiency with Large Numbers**

While the Counter approach works well for 32-bit integers (limiting distinct AND values to ~30), using a regular dictionary without cleanup could accumulate entries with zero counts over iterations, potentially causing memory issues in edge cases.

**Solution:**
Clean up zero-count entries or use defaultdict more carefully:
```python
def countSubarrays(self, nums: List[int], k: int) -> int:
    result = 0
    previous_and_counts = {}  # Use regular dict for better control

    for current_num in nums:
        current_and_counts = {}

        # Process previous AND values
        for and_result, count in previous_and_counts.items():
            new_and_result = current_num & and_result
            current_and_counts[new_and_result] = current_and_counts.get(new_and_result, 0) + count

        # Add single element subarray
        current_and_counts[current_num] = current_and_counts.get(current_num, 0) + 1

        # Count subarrays with AND = k
        result += current_and_counts.get(k, 0)

        # Only keep non-zero counts
        previous_and_counts = {key: val for key, val in current_and_counts.items() if val > 0}

    return result
```

