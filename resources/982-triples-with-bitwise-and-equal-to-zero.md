# 982. Triples with Bitwise AND Equal To Zero

> Source: https://algo.monster/liteproblems/982

## Problem Statement

You are given an integer array `nums`. Your task is to count the number of **AND triples**.

An **AND triple** is a set of three indices `(i, j, k)` that satisfy the following conditions:

- All three indices `i`, `j`, and `k` must be valid positions in the array (between `0` and `nums.length - 1`)
- The bitwise AND operation of the three elements at these positions must equal zero: `nums[i] & nums[j] & nums[k] == 0`

The `&` operator represents the bitwise AND operation, which compares each bit position of the numbers and returns `1` only if both bits are `1`, otherwise returns `0`.

For example, if `nums = [2, 1, 3]`:
- The triple `(0, 1, 2)` gives us `2 & 1 & 3 = 0`, so this is a valid AND triple
- We need to check all possible combinations of three indices

Note that the indices don't need to be distinct - you can use the same index multiple times in a triple. For instance, `(0, 0, 1)` is a valid triple if `nums[0] & nums[0] & nums[1] == 0`.

Return the total count of all valid AND triples.

## Intuition

The naive approach would be to check all possible triples `(i, j, k)` using three nested loops, computing `nums[i] & nums[j] & nums[k]` for each combination. However, this would require `O(n³)` operations, which could be inefficient for larger arrays.

The key insight is that we can optimize this by breaking down the problem into two stages:

1. **Pre-compute partial results**: Instead of computing `nums[i] & nums[j] & nums[k]` for every triple, we can first compute all possible values of `nums[i] & nums[j]` for every pair `(i, j)`. Since the bitwise AND operation is associative, we have `(nums[i] & nums[j]) & nums[k] = nums[i] & nums[j] & nums[k]`.

2. **Count occurrences**: We store how many times each partial result `nums[i] & nums[j]` appears. This is important because if multiple pairs produce the same AND result, they will all contribute to the answer when combined with a suitable third element.

3. **Match with third element**: For each unique partial result `xy` from step 1, we check against every element `z` in the array. If `xy & z == 0`, then all pairs that produced `xy` can form valid triples with this `z`. So we add the count of such pairs to our answer.

This approach is more efficient because:
- We avoid redundant calculations by storing intermediate results
- We use counting to handle multiple occurrences of the same partial AND result
- We effectively reduce the problem from checking `n³` combinations to `n²` for pre-computation plus checking at most `n² × n` conditions (but often much fewer due to the counting optimization)

The beauty of this solution lies in recognizing that many different pairs `(i, j)` might produce the same AND result, and all of them would behave identically when combined with any third element `k`.

## Solution Approach

The implementation follows a two-phase approach using enumeration and counting:

**Phase 1: Pre-compute all pairwise AND results**

```python
cnt = Counter(x & y for x in nums for y in nums)
```

- We use a nested loop (via list comprehension) to iterate through all pairs of elements in `nums`
- For each pair `(x, y)`, we compute their bitwise AND: `x & y`
- We use Python's `Counter` from the collections module to count how many times each AND result appears
- For example, if `nums = [2, 1, 3]`, we compute:
  - `2 & 2 = 2`, `2 & 1 = 0`, `2 & 3 = 2`
  - `1 & 2 = 0`, `1 & 1 = 1`, `1 & 3 = 1`
  - `3 & 2 = 2`, `3 & 1 = 1`, `3 & 3 = 3`
  - The counter would store: `{2: 3, 0: 2, 1: 3, 3: 1}`

**Phase 2: Count valid triples**

```python
return sum(v for xy, v in cnt.items() for z in nums if xy & z == 0)
```

- We iterate through each unique pairwise AND result `xy` and its count `v` from our counter
- For each `xy`, we check every element `z` in the original array
- If `xy & z == 0`, this means we found valid triples:
  - All `v` pairs that produced the AND result `xy` can form valid triples with this `z`
  - We add `v` to our running total
- The sum accumulates all valid triple counts

**Why this works:**
- If `(nums[i] & nums[j]) & nums[k] == 0`, then any indices `(i', j')` where `nums[i'] & nums[j'] == nums[i] & nums[j]` will also satisfy `(nums[i'] & nums[j']) & nums[k] == 0`
- By grouping pairs by their AND result and counting them, we avoid redundant calculations
- Time complexity: `O(n²)` for the first phase + `O(n × unique_AND_results)` for the second phase, which is better than the naive `O(n³)` approach

## Example Walkthrough

Let's walk through the solution with `nums = [2, 1, 3]`.

**Step 1: Pre-compute all pairwise AND results**

We need to compute `x & y` for every pair of elements (including pairs with the same element):

- For `x = 2` (binary: 10):
  - `2 & 2 = 2` (10 & 10 = 10)
  - `2 & 1 = 0` (10 & 01 = 00)
  - `2 & 3 = 2` (10 & 11 = 10)

- For `x = 1` (binary: 01):
  - `1 & 2 = 0` (01 & 10 = 00)
  - `1 & 1 = 1` (01 & 01 = 01)
  - `1 & 3 = 1` (01 & 11 = 01)

- For `x = 3` (binary: 11):
  - `3 & 2 = 2` (11 & 10 = 10)
  - `3 & 1 = 1` (11 & 01 = 01)
  - `3 & 3 = 3` (11 & 11 = 11)

Our Counter becomes: `{0: 2, 1: 3, 2: 3, 3: 1}`
- AND result `0` appears 2 times (from pairs at indices (0,1) and (1,0))
- AND result `1` appears 3 times (from pairs at indices (1,1), (1,2), (2,1))
- AND result `2` appears 3 times (from pairs at indices (0,0), (0,2), (2,0))
- AND result `3` appears 1 time (from pair at indices (2,2))

**Step 2: Count valid triples**

Now for each unique AND result and its count, check against every element in `nums`:

- For `xy = 0` (count = 2):
  - Check `0 & 2 = 0` ✓ → Add 2 to answer
  - Check `0 & 1 = 0` ✓ → Add 2 to answer
  - Check `0 & 3 = 0` ✓ → Add 2 to answer
  - Subtotal: 6

- For `xy = 1` (count = 3):
  - Check `1 & 2 = 0` ✓ → Add 3 to answer
  - Check `1 & 1 = 1` ✗ → Add 0
  - Check `1 & 3 = 1` ✗ → Add 0
  - Subtotal: 3

- For `xy = 2` (count = 3):
  - Check `2 & 2 = 2` ✗ → Add 0
  - Check `2 & 1 = 0` ✓ → Add 3 to answer
  - Check `2 & 3 = 2` ✗ → Add 0
  - Subtotal: 3

- For `xy = 3` (count = 1):
  - Check `3 & 2 = 2` ✗ → Add 0
  - Check `3 & 1 = 1` ✗ → Add 0
  - Check `3 & 3 = 3` ✗ → Add 0
  - Subtotal: 0

**Total: 6 + 3 + 3 + 0 = 12**

This means there are 12 valid AND triples where `nums[i] & nums[j] & nums[k] == 0`.

## Python Solution

```python
class Solution:
    def countTriplets(self, nums: List[int]) -> int:
        # Step 1: Pre-compute frequency of all possible bitwise AND results for pairs
        # For each pair (x, y) from nums, calculate x & y and count occurrences
        pair_and_frequency = Counter(x & y for x in nums for y in nums)
      
        # Step 2: Count valid triplets
        # For each pre-computed pair AND result and its frequency,
        # check against each number z in nums
        # If (x & y) & z == 0, then the triplet (x, y, z) has AND result of 0
        triplet_count = sum(
            frequency 
            for pair_and_result, frequency in pair_and_frequency.items() 
            for z in nums 
            if pair_and_result & z == 0
        )
      
        return triplet_count
```

## Time and Space Complexity

The time complexity is `O(n^2 + n × M)`, where `n` is the length of the array `nums` and `M` is the maximum value in the array `nums` (with `M ≤ 2^16` in this problem).

Breaking down the time complexity:
- The first part `Counter(x & y for x in nums for y in nums)` iterates through all pairs of elements in `nums`, performing a bitwise AND operation for each pair. This takes `O(n^2)` time.
- The second part iterates through all key-value pairs in the counter (at most `M` unique values from the bitwise AND operations) and for each pair, iterates through all elements `z` in `nums` to check if `xy & z == 0`. This takes `O(M × n)` time.
- Combined: `O(n^2 + n × M)`

The space complexity is `O(M)`, where `M` is the maximum possible value in `nums`.

The space is primarily used by the `Counter` object `cnt`, which stores the frequency of each unique result from `x & y`. In the worst case, there could be up to `M` different values resulting from the bitwise AND operations between pairs of elements, hence `O(M)` space complexity.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

**Pitfall 1: Misunderstanding the index requirement - treating indices as distinct**

A common mistake is assuming that the three indices `i`, `j`, and `k` must all be different. The problem actually allows using the same index multiple times. For instance, `(0, 0, 0)` is a valid triple if `nums[0] & nums[0] & nums[0] == 0`.

**Wrong approach:**
```python
# Incorrectly filtering out cases where indices are the same
count = 0
for i in range(len(nums)):
    for j in range(i+1, len(nums)):  # Wrong: skipping i == j
        for k in range(j+1, len(nums)):  # Wrong: skipping j == k
            if nums[i] & nums[j] & nums[k] == 0:
                count += 1
```

**Correct approach:**
```python
# Allow all index combinations including duplicates
for x in nums:
    for y in nums:
        for z in nums:
            if x & y & z == 0:
                count += 1
```

**Pitfall 2: Integer overflow concerns in other languages**

While Python handles arbitrary-precision integers automatically, implementing this in languages like C++ or Java requires attention to data types. The bitwise AND operation itself won't overflow, but the counting might.

**Solution for other languages:**
- Use appropriate data types (e.g., `long long` in C++ or `long` in Java) for the counter
- The maximum count would be `n³` where n is the array length, so ensure your counter variable can handle this

**Pitfall 3: Inefficient Counter implementation**

Some might try to manually implement the counting logic instead of using Python's efficient `Counter` class, leading to verbose and potentially slower code.

**Inefficient manual approach:**
```python
# Manual dictionary management
pair_and_frequency = {}
for x in nums:
    for y in nums:
        result = x & y
        if result in pair_and_frequency:
            pair_and_frequency[result] += 1
        else:
            pair_and_frequency[result] = 1
```

**Better approach (as shown in solution):**
```python
# Concise and efficient using Counter
pair_and_frequency = Counter(x & y for x in nums for y in nums)
```

**Pitfall 4: Attempting premature optimization with bit manipulation tricks**

Some might try to optimize by analyzing bit patterns or using advanced bit manipulation techniques, but this often complicates the code without significant performance gains for this problem's constraints.

**Example of over-optimization:**
```python
# Trying to skip combinations based on bit patterns
# This adds complexity without meaningful performance improvement
max_val = max(nums)
bit_positions = []
while max_val:
    bit_positions.append(max_val & 1)
    max_val >>= 1
# ... complex logic to prune combinations
```

The straightforward two-phase approach with `O(n²)` complexity is already optimal enough for typical constraints and much clearer to understand and maintain.

