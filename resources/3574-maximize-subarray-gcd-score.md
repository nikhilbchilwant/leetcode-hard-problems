# 3574. Maximize Subarray GCD Score

> Source: https://algo.monster/liteproblems/3574

## Problem Statement

You have an array of positive integers `nums` and an integer `k`.

You can perform up to `k` operations on the array. In each operation, you select one element and double its value. However, each element can only be doubled once throughout all operations.

After modifying the array (or keeping it unchanged), you need to find a contiguous subarray that maximizes a specific score. The score of a subarray is calculated as:

`score = length of subarray × GCD of all elements in the subarray`

Where GCD (Greatest Common Divisor) is the largest integer that evenly divides all elements in the subarray.

Your goal is to return the maximum possible score that can be achieved by:
1. First, strategically doubling up to `k` elements (each at most once)
2. Then, selecting the contiguous subarray that gives the highest score

For example, if you have `nums = [2, 3, 6]` and `k = 1`, you might double one element to get `[4, 3, 6]` or `[2, 6, 6]` or `[2, 3, 12]`, then find the subarray with the maximum score. The subarray could be a single element, multiple consecutive elements, or the entire array.

## Intuition

Since the array length is limited to `n ≤ 1500`, we can afford to check all possible contiguous subarrays - there are `O(n²)` of them. This brute force enumeration is feasible given the constraints.

The key observation is about how doubling affects GCD. When we double a number, we're multiplying it by 2, which means we're adding one more factor of 2 to it. For a subarray's GCD to benefit from doubling operations, the GCD itself must be able to increase by a factor of 2.

Here's the critical insight: The GCD of a set of numbers can only have as many factors of 2 as the number with the **fewest** factors of 2 in that set. For example, if we have `[4, 8, 12]` where:
- `4 = 2² × 1` (2 factors of 2)
- `8 = 2³ × 1` (3 factors of 2)  
- `12 = 2² × 3` (2 factors of 2)

The GCD is limited by the numbers with only 2 factors of 2, so `GCD = 4`.

If we double some elements, we add one factor of 2 to them. To maximize the GCD, we should double the elements that have the minimum number of factors of 2 in our chosen subarray. If we have `k` operations and there are exactly `k` or fewer elements with the minimum factor count, we can increase all of them by one factor of 2, effectively doubling the GCD.

This leads to our strategy:
1. Precompute how many factors of 2 each element has
2. For each possible subarray `[l, r]`:
   - Calculate the base GCD of original elements
   - Find the minimum factor of 2 count among elements in this subarray
   - Count how many elements have this minimum
   - If this count ≤ `k`, we can double all of them, so the final GCD is `base_GCD × 2`
   - Otherwise, the GCD stays as `base_GCD`
   - Calculate score as `GCD × subarray_length`

This approach ensures we optimally use our doubling operations for each subarray we consider.

**Learn more about [Math](/problems/math-basics) patterns.**

## Solution Approach

The implementation follows the intuition by breaking down the problem into manageable steps:

**Step 1: Preprocess Factor of 2 Counts**

First, we create an array `cnt` to store how many times 2 divides each element:

```python
cnt = [0] * n
for i, x in enumerate(nums):
    while x % 2 == 0:
        cnt[i] += 1
        x //= 2
```

For each number, we repeatedly divide by 2 until it's odd, counting the divisions. For example, `12 = 2² × 3`, so `cnt[i] = 2`.

**Step 2: Enumerate All Subarrays**

We use nested loops where `l` is the left boundary and `r` is the right boundary:

```python
for l in range(n):
    g = 0  # GCD accumulator
    mi = inf  # Minimum factor of 2 count
    t = 0  # Count of elements with minimum factors
    for r in range(l, n):
        # Process subarray [l, r]
```

**Step 3: Maintain Subarray Properties**

For each subarray `[l, r]`, we maintain three key variables:

- `g`: The running GCD of elements in the current subarray. We use `gcd(g, nums[r])` to incrementally update it as we extend the subarray.
- `mi`: The minimum number of factors of 2 among all elements in the current subarray.
- `t`: The count of elements that have exactly `mi` factors of 2.

```python
g = gcd(g, nums[r])
if cnt[r] < mi:
    mi = cnt[r]  # Found a new minimum
    t = 1        # Reset count to 1
elif cnt[r] == mi:
    t += 1       # Another element with minimum factors
```

**Step 4: Calculate Score with Optimal Doubling**

The key decision is whether we can double the GCD:

```python
ans = max(ans, (g if t > k else g * 2) * (r - l + 1))
```

- If `t > k`: We have more elements with minimum factors than operations available. We can't double all of them, so the GCD remains `g`.
- If `t ≤ k`: We can double all elements with minimum factors, effectively adding one factor of 2 to the GCD, making it `g * 2`.

The score is then `GCD × length`, where length is `(r - l + 1)`.

**Time Complexity**: `O(n² × log(max(nums)))` - We enumerate `O(n²)` subarrays, and for each, we compute GCD which takes `O(log(max(nums)))` time.

**Space Complexity**: `O(n)` for the `cnt` array storing factor counts.

## Example Walkthrough

Let's work through a concrete example with `nums = [6, 12, 8]` and `k = 2`.

**Step 1: Count factors of 2 for each element**
- `6 = 2¹ × 3` → `cnt[0] = 1` (one factor of 2)
- `12 = 2² × 3` → `cnt[1] = 2` (two factors of 2)
- `8 = 2³ × 1` → `cnt[2] = 3` (three factors of 2)

So `cnt = [1, 2, 3]`

**Step 2: Enumerate all subarrays and calculate scores**

Let's trace through each subarray:

**Subarray [0, 0]: just `[6]`**
- Base GCD = 6
- Minimum factors of 2: `mi = 1`
- Count with minimum: `t = 1`
- Since `t = 1 ≤ k = 2`, we can double the GCD
- Score = `(6 × 2) × 1 = 12`

**Subarray [0, 1]: `[6, 12]`**
- Base GCD = gcd(6, 12) = 6
- Minimum factors of 2: `mi = 1` (from element 6)
- Count with minimum: `t = 1` (only element 6 has 1 factor)
- Since `t = 1 ≤ k = 2`, we can double the GCD
- Score = `(6 × 2) × 2 = 24`

**Subarray [0, 2]: `[6, 12, 8]`**
- Base GCD = gcd(gcd(6, 12), 8) = gcd(6, 8) = 2
- Minimum factors of 2: `mi = 1` (from element 6)
- Count with minimum: `t = 1` (only element 6)
- Since `t = 1 ≤ k = 2`, we can double the GCD
- Score = `(2 × 2) × 3 = 12`

**Subarray [1, 1]: just `[12]`**
- Base GCD = 12
- Minimum factors of 2: `mi = 2`
- Count with minimum: `t = 1`
- Since `t = 1 ≤ k = 2`, we can double the GCD
- Score = `(12 × 2) × 1 = 24`

**Subarray [1, 2]: `[12, 8]`**
- Base GCD = gcd(12, 8) = 4
- Minimum factors of 2: `mi = 2` (from element 12)
- Count with minimum: `t = 1` (only element 12 has 2 factors)
- Since `t = 1 ≤ k = 2`, we can double the GCD
- Score = `(4 × 2) × 2 = 16`

**Subarray [2, 2]: just `[8]`**
- Base GCD = 8
- Minimum factors of 2: `mi = 3`
- Count with minimum: `t = 1`
- Since `t = 1 ≤ k = 2`, we can double the GCD
- Score = `(8 × 2) × 1 = 16`

**Maximum score = 24** (achieved by either subarray [0, 1] or [1, 1])

The optimal strategy for subarray [0, 1] would be to double element 6 (which has the minimum factors of 2), transforming `[6, 12]` to `[12, 12]` with GCD = 12 and score = 12 × 2 = 24.

## Python Solution

```python
from typing import List
from math import gcd, inf


class Solution:
    def maxGCDScore(self, nums: List[int], k: int) -> int:
        """
        Calculate the maximum GCD score for subarrays of nums.
        The score is GCD * subarray_length, with potential doubling based on factors of 2.
      
        Args:
            nums: List of integers
            k: Threshold for determining whether to double the GCD
      
        Returns:
            Maximum possible score
        """
        n = len(nums)
      
        # Count the number of factors of 2 in each number
        factors_of_two = [0] * n
        for i, num in enumerate(nums):
            while num % 2 == 0:
                factors_of_two[i] += 1
                num //= 2
      
        max_score = 0
      
        # Try all possible subarrays starting from index left
        for left in range(n):
            current_gcd = 0
            min_factor_count = inf
            elements_with_min_factors = 0
          
            # Extend the subarray to index right
            for right in range(left, n):
                # Update GCD of current subarray
                current_gcd = gcd(current_gcd, nums[right])
              
                # Track minimum factor of 2 count and how many elements have it
                if factors_of_two[right] < min_factor_count:
                    min_factor_count = factors_of_two[right]
                    elements_with_min_factors = 1
                elif factors_of_two[right] == min_factor_count:
                    elements_with_min_factors += 1
              
                # Calculate score: double GCD if elements with minimum factors exceed k
                subarray_length = right - left + 1
                if elements_with_min_factors > k:
                    score = current_gcd * subarray_length
                else:
                    score = current_gcd * 2 * subarray_length
              
                max_score = max(max_score, score)
      
        return max_score
```

## Time and Space Complexity

**Time Complexity:** `O(n² × log n)`

The algorithm consists of:
- A preprocessing loop that counts trailing zeros (factors of 2) for each element, which takes `O(n × log max(nums))` time, where each number is divided by 2 until odd.
- A nested loop structure where:
  - The outer loop runs `n` times (for each starting position `l`)
  - The inner loop runs up to `n` times (for each ending position `r` from `l` to `n-1`)
  - Inside the inner loop, the `gcd(g, nums[r])` operation takes `O(log min(g, nums[r]))` time
  - Since we're computing GCD iteratively and the value of `g` can be at most `max(nums)`, each GCD operation is bounded by `O(log max(nums))`

The dominant operation is the nested loops with GCD computation, giving us `O(n² × log n)` time complexity, where `n` is the length of the array and the logarithmic factor comes from the GCD operations.

**Space Complexity:** `O(n)`

The algorithm uses:
- An array `cnt` of size `n` to store the count of factors of 2 for each element: `O(n)`
- A few constant variables (`ans`, `l`, `g`, `mi`, `t`, `r`): `O(1)`

Therefore, the total space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Misunderstanding the Problem Constraints

**The Issue**: The solution assumes we can only double the GCD by doubling elements with the minimum number of factors of 2, but the problem states we can double ANY k elements (each at most once). The current approach doesn't consider all possible combinations of which k elements to double.

**Example Where It Fails**:
- `nums = [3, 6, 12]`, `k = 2`
- Current approach: Counts factors of 2 as `[0, 1, 2]`
- For subarray `[6, 12]`: GCD = 6, min factors = 1, count = 1
- Since count (1) ≤ k (2), it doubles GCD to 12, giving score = 12 × 2 = 24
- **But**: We could double both 3 and 6 to get `[6, 12, 12]`, then subarray `[12, 12]` has GCD = 12, score = 12 × 2 = 24
- Or double 3 and 12 to get `[6, 6, 24]`, subarray `[6, 6]` has GCD = 6, score = 6 × 2 = 12

The real issue is that we need to consider which specific k elements to double to maximize the score, not just check if we can double all elements with minimum factors.

### Pitfall 2: Incorrect GCD Doubling Logic

**The Issue**: The code doubles the GCD when `elements_with_min_factors ≤ k`, but this doesn't correctly model the problem. We can't simply "double the GCD" - we need to actually double specific elements and then recalculate the GCD.

**Correct Approach**:
```python
def maxGCDScore(self, nums: List[int], k: int) -> int:
    n = len(nums)
    max_score = 0
  
    # Generate all possible states with up to k elements doubled
    # Using bit manipulation to represent which elements are doubled
    for mask in range(1 << n):
        if bin(mask).count('1') > k:
            continue
      
        # Create modified array based on mask
        modified = nums[:]
        for i in range(n):
            if mask & (1 << i):
                modified[i] *= 2
      
        # Find best subarray in this modified array
        for left in range(n):
            current_gcd = 0
            for right in range(left, n):
                current_gcd = gcd(current_gcd, modified[right])
                length = right - left + 1
                score = current_gcd * length
                max_score = max(max_score, score)
  
    return max_score
```

### Pitfall 3: Efficiency Concerns

**The Issue**: The corrected approach has exponential time complexity O(2^n × n² × log(max_val)) which becomes impractical for large arrays.

**Optimization Strategy**:
For larger inputs, consider:
1. **Pruning**: Skip masks that can't improve the current best score
2. **Dynamic Programming**: Cache GCD calculations for repeated subarrays
3. **Heuristics**: Focus on doubling elements that appear in high-scoring subarrays first

```python
def maxGCDScore(self, nums: List[int], k: int) -> int:
    n = len(nums)
    if n <= 15:  # Use brute force for small arrays
        return self.bruteForce(nums, k)
    else:  # Use heuristic approach for larger arrays
        return self.heuristicApproach(nums, k)
```

The original solution's approach of counting factors of 2 is an interesting optimization attempt, but it makes incorrect assumptions about how doubling operations affect the GCD of subarrays.

