# 1681. Minimum Incompatibility

> Source: https://algo.monster/liteproblems/1681

## Problem Statement

You have an integer array `nums` and an integer `k`. Your task is to divide this array into `k` subsets where:
- Each subset must have **equal size** (same number of elements)
- No subset can contain duplicate elements (all elements in a subset must be distinct)

The **incompatibility** of a subset is defined as the difference between its maximum and minimum elements. For example, if a subset contains `[2, 5, 7]`, its incompatibility is `7 - 2 = 5`.

Your goal is to find the **minimum possible sum** of incompatibilities across all `k` subsets after optimally distributing the array elements. If it's impossible to create such a partition (for instance, if there are too many duplicate elements to form distinct subsets), return `-1`.

Key points to understand:
- The array must be completely partitioned (all elements must be assigned to exactly one subset)
- Each subset must have exactly `n/k` elements where `n` is the length of `nums`
- Elements can appear in any order within a subset
- You want to minimize the total sum of (max - min) values across all subsets

For example, if `nums = [1, 2, 1, 4]` and `k = 2`, you need to create 2 subsets of size 2 each. One valid partition could be `[1, 2]` and `[1, 4]`, giving incompatibilities of `1` and `3` respectively, for a total sum of `4`.

## Intuition

The key insight is that we need to explore different ways to partition the array into valid subsets and find the optimal one. Since each subset must have exactly `m = n/k` elements and no duplicates, we can think of this as a combinatorial optimization problem.

First, let's consider what makes a valid subset. A subset of size `m` is valid if it contains no duplicate elements. For any valid subset, we can immediately calculate its incompatibility by finding the difference between its maximum and minimum elements. This suggests we should precompute the incompatibility for all possible valid subsets.

Since the array size is small (constraints typically allow up to 16 elements), we can use **bit manipulation** to represent subsets. Each subset can be encoded as a binary number where the `j-th` bit indicates whether `nums[j]` is included. For example, if `nums = [1, 2, 3]`, the binary `101` represents the subset containing `nums[0]` and `nums[2]`, which is `[1, 3]`.

With this representation, we can precompute array `g` where `g[i]` stores the incompatibility of subset `i` if it's a valid subset of size `m`, or `-1` if it's invalid (has duplicates or wrong size).

Now for the main problem: how do we find the optimal way to partition the entire array? This is where **[dynamic programming](/problems/dynamic_programming_intro)** comes in. We define `f[i]` as the minimum sum of incompatibilities when we've assigned elements according to [bitmask](/problems/bitmask_intro) `i`. Starting from `f[0] = 0` (no elements assigned), we build up to `f[2^n - 1]` (all elements assigned).

The transition works as follows: for each state `i`, we look at the remaining unassigned elements. Among these, we find elements that are distinct (no duplicates) and try to form the next subset. We enumerate all possible valid subsets from these remaining elements and update our DP state accordingly. If we can form a valid subset `j` from the remaining elements, then `f[i | j] = min(f[i | j], f[i] + g[j])`.

The reason this works is that we're essentially trying all possible ways to partition the array while maintaining the constraints, and the DP ensures we don't recompute the same subproblems. The bit manipulation allows us to efficiently track which elements have been assigned and which are still available.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) and [Bitmask](/problems/bitmask_intro) patterns.**

## Solution Approach

The solution uses preprocessing combined with state compression and [dynamic programming](/problems/dynamic_programming_intro). Let's walk through the implementation step by step.

**Step 1: Preprocessing Valid Subsets**

First, we calculate `m = n/k`, which is the required size of each subset. We create an array `g` of size `2^n` to store incompatibilities, initialized with `-1`.

For each possible subset `i` from `1` to `2^n - 1`:
- Check if the subset has exactly `m` elements using `bit_count()`
- If yes, iterate through all elements in `nums` and check if bit `j` is set in `i`
- Track the elements in a set to detect duplicates
- Keep track of minimum and maximum values
- If we successfully add `m` distinct elements, calculate `g[i] = max - min`
- If duplicates are found, `g[i]` remains `-1`

```python
for i in range(1, 1 << n):
    if i.bit_count() != m:
        continue
    # Check for duplicates and calculate min/max
    if len(s) == m:  # Valid subset with no duplicates
        g[i] = mx - mi
```

**Step 2: [Dynamic Programming](/problems/dynamic_programming_intro)**

Initialize DP array `f` where `f[i]` represents the minimum sum of incompatibilities for state `i`. Set `f[0] = 0` and all others to infinity.

For each state `i` where `f[i]` is not infinity:
1. Find all unassigned elements that are distinct (no duplicates among remaining elements)
2. Create a [bitmask](/problems/bitmask_intro) `mask` representing these available distinct elements
3. If we have fewer than `m` distinct elements available, skip this state

```python
for j, x in enumerate(nums):
    if (i >> j & 1) == 0 and x not in s:  # Unassigned and distinct
        s.add(x)
        mask |= 1 << j
```

**Step 3: Enumerate Subsets**

For the available elements represented by `mask`, enumerate all its subsets using the technique `j = (j - 1) & mask`. This efficiently generates all subsets of `mask`.

For each subset `j`:
- Check if `g[j] != -1` (valid subset)
- Update `f[i | j] = min(f[i | j], f[i] + g[j])`

This transition means: if we're at state `i` and add subset `j`, the new state is `i | j` with cost `f[i] + g[j]`.

```python
j = mask
while j:
    if g[j] != -1:
        f[i | j] = min(f[i | j], f[i] + g[j])
    j = (j - 1) & mask
```

**Step 4: Return Result**

The final answer is `f[2^n - 1]`, which represents the state where all elements are assigned. If this value is still infinity, it means no valid partition exists, so return `-1`.

The time complexity is `O(3^n)` due to iterating through all subsets and their subsets, and space complexity is `O(2^n)` for the DP and preprocessing arrays.

## Example Walkthrough

Let's walk through a concrete example with `nums = [6, 3, 8, 1, 3, 1, 2, 2]` and `k = 4`.

**Initial Setup:**
- Array length `n = 8`, so each subset must have `m = n/k = 8/4 = 2` elements
- We need to create 4 subsets, each with 2 distinct elements

**Step 1: Preprocessing Valid Subsets**

We'll compute incompatibilities for all possible 2-element subsets. Using binary representation where bit position corresponds to array index:

Some examples of valid subsets:
- Binary `00000011` (decimal 3) = indices [0,1] = elements [6,3]
  - No duplicates, incompatibility = 6 - 3 = 3
  - `g[3] = 3`

- Binary `00001001` (decimal 9) = indices [0,3] = elements [6,1]  
  - No duplicates, incompatibility = 6 - 1 = 5
  - `g[9] = 5`

- Binary `00011000` (decimal 24) = indices [3,4] = elements [1,3]
  - No duplicates, incompatibility = 3 - 1 = 2
  - `g[24] = 2`

- Binary `00101000` (decimal 40) = indices [3,5] = elements [1,1]
  - Contains duplicates! 
  - `g[40] = -1` (invalid)

**Step 2: Dynamic Programming**

Starting with `f[0] = 0` (no elements assigned), we build up our solution:

*State `i = 0` (binary `00000000`):*
- All elements available: [6,3,8,1,3,1,2,2]
- Distinct available elements: {6,3,8,1,2} at positions [0,1,2,3,6]
- Mask = `01001111` (positions where distinct elements exist)

We try forming 2-element subsets from this mask:
- Subset `00000011` (elements [6,3]): `f[3] = 0 + 3 = 3`
- Subset `00000101` (elements [6,8]): `f[5] = 0 + 2 = 2`
- Subset `00001001` (elements [6,1]): `f[9] = 0 + 5 = 5`
- And so on...

*State `i = 3` (binary `00000011`):*
- Already used: positions [0,1] (elements [6,3])
- Remaining: [8,1,3,1,2,2] at positions [2,3,4,5,6,7]
- Distinct remaining: {8,1,3,2} at positions [2,3,4,6]
- Mask = `01011100`

Valid transitions:
- Subset `00001100` (elements [8,1]): `f[15] = min(f[15], 3 + 7) = 10`
- Subset `00010100` (elements [8,3]): `f[23] = min(f[23], 3 + 5) = 8`
- Subset `01001000` (elements [1,2]): `f[75] = min(f[75], 3 + 1) = 4`

*Continuing the process...*

Eventually we reach state `i = 255` (binary `11111111`) where all elements are assigned.

**Step 3: Finding the Optimal Partition**

After exploring all possibilities, we find the minimum sum is achieved with:
- Subset 1: [1,3] (indices 3,4) → incompatibility = 2
- Subset 2: [1,2] (indices 5,6) → incompatibility = 1  
- Subset 3: [2,3] (indices 7,1) → incompatibility = 1
- Subset 4: [6,8] (indices 0,2) → incompatibility = 2

Total sum = 2 + 1 + 1 + 2 = 6

Therefore, `f[255] = 6` is our answer.

This walkthrough demonstrates how the algorithm systematically explores all valid partitions using bit manipulation to track states and dynamic programming to avoid redundant calculations.

## Python Solution

```python
class Solution:
    def minimumIncompatibility(self, nums: List[int], k: int) -> int:
        n = len(nums)
        subset_size = n // k  # Size of each subset
      
        # incompatibility[mask] stores the incompatibility of the subset represented by mask
        # -1 means invalid subset (has duplicates or wrong size)
        incompatibility = [-1] * (1 << n)
      
        # Precompute incompatibility for all valid subsets of size subset_size
        for mask in range(1, 1 << n):
            if bin(mask).count('1') != subset_size:
                continue
          
            subset_elements = set()
            min_val, max_val = 20, 0  # Initialize with extreme values (nums[i] <= 16)
          
            # Check if this subset is valid (no duplicates) and find min/max
            for idx, num in enumerate(nums):
                if mask >> idx & 1:  # If idx-th bit is set in mask
                    if num in subset_elements:
                        # Duplicate found, invalid subset
                        break
                    subset_elements.add(num)
                    min_val = min(min_val, num)
                    max_val = max(max_val, num)
          
            # If subset has no duplicates, store its incompatibility
            if len(subset_elements) == subset_size:
                incompatibility[mask] = max_val - min_val
      
        # dp[mask] = minimum sum of incompatibilities for elements selected by mask
        dp = [float('inf')] * (1 << n)
        dp[0] = 0  # Base case: empty set has 0 incompatibility
      
        # Try all possible states
        for used_mask in range(1 << n):
            if dp[used_mask] == float('inf'):
                continue
          
            # Find all unused elements and their first occurrence positions
            unused_elements = set()
            available_mask = 0
            for idx, num in enumerate(nums):
                if (used_mask >> idx & 1) == 0 and num not in unused_elements:
                    unused_elements.add(num)
                    available_mask |= 1 << idx
          
            # Need at least subset_size unused distinct elements to form next subset
            if len(unused_elements) < subset_size:
                continue
          
            # Try all possible subsets from available positions
            subset_mask = available_mask
            while subset_mask:
                if incompatibility[subset_mask] != -1:
                    # Update dp for state including this new subset
                    dp[used_mask | subset_mask] = min(
                        dp[used_mask | subset_mask], 
                        dp[used_mask] + incompatibility[subset_mask]
                    )
                # Generate next subset of available_mask
                subset_mask = (subset_mask - 1) & available_mask
      
        # Return result for all elements used, or -1 if impossible
        return dp[-1] if dp[-1] != float('inf') else -1
```

## Time and Space Complexity

The time complexity of this algorithm is `O(3^n)`, where `n` is the length of the input array.

The analysis breaks down as follows:
- The first loop iterates through all `2^n` possible subsets to precompute valid groups of size `m`. For each subset, we check if it contains exactly `m` elements and if all elements are distinct. This takes `O(2^n × n)` time.
- The dynamic programming part uses bitmask DP where `f[i]` represents the minimum incompatibility for elements selected by mask `i`.
- For each state `i` (there are `2^n` states), we enumerate all possible valid subsets `j` that can be added to the current state. The key insight is that for each position in the array, it can be in one of three states: already used (in mask `i`), included in the next subset `j`, or not included in either. This gives us the `3^n` complexity.
- The line `j = (j - 1) & mask` efficiently iterates through all subsets of `mask`, which contributes to the `3^n` bound when summed across all states.

The space complexity is `O(2^n)` due to:
- Array `g` of size `2^n` storing the incompatibility values for all valid subsets of size `m`
- Array `f` of size `2^n` storing the DP states for all possible selections of elements
- Additional space used is `O(n)` for temporary variables, which doesn't affect the overall complexity

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Incorrectly Handling Duplicate Elements in Available Mask

**The Problem:**
When building the `available_mask` for unused elements, a common mistake is including all positions of duplicate elements instead of just the first occurrence. This leads to incorrect subset enumeration and potentially invalid states.

```python
# INCORRECT approach - includes all positions of duplicates
available_mask = 0
for idx, num in enumerate(nums):
    if (used_mask >> idx & 1) == 0:  # Just checking if unused
        available_mask |= 1 << idx
```

This would mark all occurrences of duplicate values as available, causing the algorithm to consider invalid subsets that contain the same value multiple times.

**The Solution:**
Track which values have already been seen and only include the first occurrence of each unused value:

```python
# CORRECT approach - only first occurrence of each distinct value
unused_elements = set()
available_mask = 0
for idx, num in enumerate(nums):
    if (used_mask >> idx & 1) == 0 and num not in unused_elements:
        unused_elements.add(num)
        available_mask |= 1 << idx
```

### Pitfall 2: Inefficient Subset Enumeration

**The Problem:**
Attempting to enumerate all possible subsets of size `subset_size` from the available elements using recursion or nested loops, which is inefficient and complex:

```python
# INEFFICIENT - checking all 2^n subsets repeatedly
for subset_mask in range(1, 1 << n):
    if bin(subset_mask).count('1') == subset_size:
        # Check if subset is valid...
```

**The Solution:**
Use the bit manipulation technique `(j - 1) & mask` to efficiently enumerate only the subsets of the available mask:

```python
# EFFICIENT - only enumerate subsets of available_mask
subset_mask = available_mask
while subset_mask:
    if incompatibility[subset_mask] != -1:
        # Process valid subset
    subset_mask = (subset_mask - 1) & available_mask
```

### Pitfall 3: Not Validating Subset Size Early

**The Problem:**
Forgetting to check if there are enough distinct unused elements before attempting to form the next subset:

```python
# MISSING validation - may attempt impossible transitions
for used_mask in range(1 << n):
    # Directly trying to form subsets without checking feasibility
    subset_mask = available_mask
    while subset_mask:
        # ...
```

**The Solution:**
Always verify that you have at least `subset_size` distinct elements available before attempting to form new subsets:

```python
# With proper validation
if len(unused_elements) < subset_size:
    continue  # Skip this state as it's impossible to form a valid subset
```

### Pitfall 4: Incorrect Base Case or Initialization

**The Problem:**
Setting incorrect initial values for the DP array or incompatibility array:

```python
# INCORRECT - wrong initialization
dp = [0] * (1 << n)  # Should be infinity except dp[0]
incompatibility = [0] * (1 << n)  # Should be -1 for invalid subsets
```

**The Solution:**
Properly initialize arrays to distinguish between valid and invalid states:

```python
# CORRECT initialization
incompatibility = [-1] * (1 << n)  # -1 indicates invalid subset
dp = [float('inf')] * (1 << n)     # infinity for unvisited states
dp[0] = 0                          # Base case: empty set has 0 cost
```

These pitfalls can cause the algorithm to produce incorrect results or fail to find valid partitions when they exist. Careful attention to duplicate handling, efficient enumeration, and proper validation ensures the solution works correctly for all edge cases.

