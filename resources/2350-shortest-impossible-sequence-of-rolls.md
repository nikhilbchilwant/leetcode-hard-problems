# 2350. Shortest Impossible Sequence of Rolls

> Source: https://algo.monster/liteproblems/2350

## Problem Statement

You are given an array `rolls` representing the results of rolling a k-sided dice n times, where each die face is numbered from 1 to k. The i-th element `rolls[i]` represents the result of the i-th roll.

Your task is to find the length of the shortest sequence of dice rolls that cannot be found as a subsequence within the given `rolls` array.

Key points to understand:
- A **sequence of rolls** of length `len` means rolling the k-sided dice exactly `len` times
- A **subsequence** is formed by selecting elements from `rolls` in their original order (not necessarily consecutive)
- You need to find the smallest length such that at least one possible sequence of that length doesn't appear as a subsequence in `rolls`

For example, if k=3 (dice with faces 1,2,3) and `rolls = [1,2,3,1,2]`:
- All sequences of length 1 (namely [1], [2], [3]) can be found as subsequences
- All sequences of length 2 (like [1,1], [1,2], [1,3], [2,1], etc.) can be found as subsequences
- But some sequence of length 3 cannot be found as a subsequence
- Therefore, the answer would be 3

## Intuition

The key insight is to think about what makes a sequence impossible to form as a subsequence. 

Let's start with sequences of length 1. For any sequence of length 1 to be missing, we would need at least one number from 1 to k to never appear in `rolls`. But if all k numbers appear at least once in `rolls`, then every possible sequence of length 1 can be formed.

Now consider sequences of length 2. For every possible sequence of length 2 to exist as a subsequence, we need to be able to pick any first number, then find any second number appearing after it. This is possible if, after picking any starting position for our first number, we can still find all k different values appearing somewhere later in the array.

This leads us to a crucial observation: we can form all sequences of length `L` if and only if we can partition `rolls` into `L` segments where each segment contains all k distinct values.

Why? Because:
- To form any sequence `[a₁, a₂, ..., aₗ]`, we pick `a₁` from the first segment (which has all k values, so `a₁` is guaranteed to exist)
- Then pick `a₂` from the second segment (which also has all k values)
- And so on...

The first length where we **cannot** form all possible sequences is when we can only create `L-1` complete segments (segments with all k values), but not `L` segments.

This naturally leads to the [greedy](/problems/greedy_intro) approach: go through `rolls` and collect distinct values. Once we've seen all k distinct values, we've completed one segment and can start collecting for the next segment. The number of complete segments we can form plus 1 gives us our answer - that's the shortest sequence length that cannot be formed as a subsequence.

For example, if we can form exactly 2 complete segments, it means:
- All sequences of length 1 can be formed (using the first segment)
- All sequences of length 2 can be formed (using both segments)
- But not all sequences of length 3 can be formed (we'd need 3 complete segments)

**Learn more about [Greedy](/problems/greedy_intro) patterns.**

## Solution Approach

The implementation uses a [greedy](/problems/greedy_intro) algorithm with a set data structure to efficiently track distinct values:

1. **Initialize variables:**
   - `ans = 1`: Start with assuming the shortest impossible sequence has length 1
   - `s = set()`: Use a set to track distinct dice values in the current segment

2. **Iterate through the rolls array:**
   - For each value `v` in `rolls`:
     - Add `v` to the set `s`
     - Check if `len(s) == k` (i.e., we've collected all k distinct values)

3. **When a complete segment is found:**
   - If the set contains all k values, it means we've completed one segment
   - Increment `ans` by 1 (now we can form all sequences of this new length)
   - Clear the set with `s.clear()` to start collecting for the next segment

4. **Return the result:**
   - After processing all rolls, `ans` represents the length of the shortest sequence that cannot be formed

**Why this works:**
- Each time we collect all k distinct values, we guarantee that all sequences of length equal to the number of complete segments can be formed
- The first incomplete segment (where we don't collect all k values) determines the shortest impossible sequence length

**Time Complexity:** `O(n)` where n is the length of rolls array - we iterate through the array once

**Space Complexity:** `O(k)` for the set, which can contain at most k distinct values

**Example walkthrough:**
If `rolls = [1,2,3,2,1]` and `k = 3`:
- First segment: collect {1,2,3} → complete, `ans = 2`
- Second segment: collect {2,1} → incomplete (missing 3)
- Return `ans = 2`

This means all sequences of length 1 can be formed, but not all sequences of length 2 (specifically, any sequence ending with 3 cannot be formed).

## Example Walkthrough

Let's walk through a concrete example with `rolls = [2, 1, 3, 2, 1, 1, 3]` and `k = 3` (a 3-sided dice).

**Step-by-step execution:**

Initial state: `ans = 1`, `s = {}`

1. **Process rolls[0] = 2:**
   - Add 2 to set: `s = {2}`
   - Is `len(s) == 3`? No, continue

2. **Process rolls[1] = 1:**
   - Add 1 to set: `s = {2, 1}`
   - Is `len(s) == 3`? No, continue

3. **Process rolls[2] = 3:**
   - Add 3 to set: `s = {2, 1, 3}`
   - Is `len(s) == 3`? **Yes!** We've found our first complete segment
   - Increment `ans = 2` (we can now form all sequences of length 1)
   - Clear the set: `s = {}`

4. **Process rolls[3] = 2:**
   - Add 2 to set: `s = {2}`
   - Is `len(s) == 3`? No, continue

5. **Process rolls[4] = 1:**
   - Add 1 to set: `s = {2, 1}`
   - Is `len(s) == 3`? No, continue

6. **Process rolls[5] = 1:**
   - Add 1 to set: `s = {2, 1}` (1 already exists, set remains unchanged)
   - Is `len(s) == 3`? No, continue

7. **Process rolls[6] = 3:**
   - Add 3 to set: `s = {2, 1, 3}`
   - Is `len(s) == 3`? **Yes!** We've found our second complete segment
   - Increment `ans = 3` (we can now form all sequences of length 2)
   - Clear the set: `s = {}`

**Result:** Return `ans = 3`

**Verification:**
- **Length 1 sequences:** [1], [2], [3] - all can be found as subsequences ✓
- **Length 2 sequences:** We found 2 complete segments, so all 9 possible sequences ([1,1], [1,2], [1,3], [2,1], [2,2], [2,3], [3,1], [3,2], [3,3]) can be formed ✓
- **Length 3 sequences:** We only have 2 complete segments, so not all 27 possible sequences can be formed. For example, any sequence requiring us to pick from a third segment cannot be formed ✗

Therefore, 3 is the shortest sequence length that cannot be found as a subsequence.

## Python Solution

```python
class Solution:
    def shortestSequence(self, rolls: List[int], k: int) -> int:
        # Initialize the length of shortest sequence that cannot be obtained
        sequence_length = 1
      
        # Set to track unique values seen in current subsequence
        seen_values = set()
      
        # Iterate through each die roll
        for value in rolls:
            # Add current value to the set
            seen_values.add(value)
          
            # When we've seen all k possible values (1 to k)
            if len(seen_values) == k:
                # We can form any sequence of current length using these values
                # So increment the target sequence length
                sequence_length += 1
              
                # Reset the set to start collecting for next length
                seen_values.clear()
      
        # Return the shortest sequence length that cannot be obtained
        return sequence_length
```

## Time and Space Complexity

**Time Complexity:** `O(n)` where `n` is the length of the rolls array. The algorithm iterates through the rolls array exactly once. For each element, it performs constant time operations: adding to a set (average `O(1)`), checking the set size (`O(1)`), and clearing the set (`O(k)` which happens at most `n/k` times, amortizing to `O(1)` per element).

**Space Complexity:** `O(k)` where `k` is the number of distinct values possible on the dice. The set `s` can contain at most `k` distinct elements before being cleared. This is the maximum space used at any point during the execution.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Misunderstanding the Problem as Finding Consecutive Elements

**The Mistake:** Many people initially interpret this problem as finding consecutive elements in the array rather than subsequences. They might think that `[1,2,3,1,2]` needs elements to be adjacent to form sequences.

**Why It's Wrong:** The problem asks for subsequences, not subarrays. A subsequence maintains the relative order but doesn't require elements to be consecutive. For example, in `[1,2,3,1,2]`, we can form the subsequence `[1,3,2]` by taking the 1st, 3rd, and 5th elements.

**Solution:** Remember that subsequences allow "jumping" over elements. When tracking whether a sequence can be formed, you only need to ensure the elements appear in the correct order somewhere in the array, not necessarily next to each other.

### Pitfall 2: Incorrectly Resetting the Tracking Set

**The Mistake:** Some implementations might reset the set after processing each element, or forget to reset it at all after finding all k values:

```python
# Wrong approach 1: Resetting too early
for value in rolls:
    seen_values.add(value)
    seen_values.clear()  # This clears after every element!
  
# Wrong approach 2: Never resetting
for value in rolls:
    seen_values.add(value)
    if len(seen_values) == k:
        sequence_length += 1
        # Forgot to clear the set!
```

**Why It's Wrong:** 
- Resetting too early prevents you from accumulating all k distinct values
- Never resetting means you'll only increment the answer once, giving an incorrect result

**Solution:** Only clear the set immediately after detecting that all k values have been collected:

```python
if len(seen_values) == k:
    sequence_length += 1
    seen_values.clear()  # Clear only after finding all k values
```

### Pitfall 3: Off-by-One Error in Initial Value

**The Mistake:** Starting with `sequence_length = 0` instead of `sequence_length = 1`:

```python
# Wrong
sequence_length = 0  # Starting from 0
```

**Why It's Wrong:** The minimum possible sequence length is 1 (a single dice roll). If we start from 0, we'd be claiming that an empty sequence cannot be formed, which doesn't make sense in the context of this problem.

**Solution:** Always initialize with `sequence_length = 1` since we're looking for the shortest non-empty sequence that cannot be formed.

### Pitfall 4: Confusing Dice Values with Array Indices

**The Mistake:** Assuming dice values are 0-indexed (0 to k-1) instead of 1-indexed (1 to k):

```python
# Wrong assumption in validation
if value < 0 or value >= k:  # Assumes 0-indexed
    # Handle invalid value
```

**Why It's Wrong:** The problem states dice faces are numbered from 1 to k, not 0 to k-1.

**Solution:** Remember that valid dice values range from 1 to k inclusive. This doesn't affect the core algorithm but is important for understanding test cases and validation.

