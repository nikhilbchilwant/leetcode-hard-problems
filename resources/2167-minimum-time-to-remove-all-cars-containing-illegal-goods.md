# 2167. Minimum Time to Remove All Cars Containing Illegal Goods

> Source: https://algo.monster/liteproblems/2167

## Problem Statement

You have a train represented as a binary string `s` where each character represents a train car. A `'0'` means the car is clean (no illegal goods), and a `'1'` means the car contains illegal goods.

Your goal is to remove all cars containing illegal goods (`'1'`s) from the train using the minimum total time. You have three operations available:

1. **Remove from left end**: Remove the leftmost car (`s[0]`) - costs 1 unit of time
2. **Remove from right end**: Remove the rightmost car (`s[s.length - 1]`) - costs 1 unit of time  
3. **Remove from middle**: Remove any car from anywhere in the sequence - costs 2 units of time

You can perform any of these operations any number of times. The task is to find the minimum time needed to remove all cars with illegal goods.

For example, if `s = "1100101"`, you need to remove all the `'1'`s. You could:
- Remove 2 cars from the left (removes "11"), taking 2 units of time
- Remove 3 cars from the right (removes "101"), taking 3 units of time
- Total: 5 units of time

## Intuition

The key insight is that we want to find an optimal "splitting point" in the train. We can remove all illegal goods from the left side of this split point using some strategy, and all illegal goods from the right side using another strategy.

For any segment of the train, we have two choices:
1. Remove the entire segment from one end (costs equal to the segment length)
2. Keep some cars and remove individual illegal cars from the middle (each costs 2)

This leads us to think about the problem as: at each position `i`, what's the minimum cost to clear all illegal goods from `[0, i]` and from `[i, n-1]`?

For the prefix part (left side), at each position with an illegal good, we can either:
- Remove all cars from the left up to this point (cost = `i + 1`)
- Keep the previous cars and remove this one individually (cost = `pre[i] + 2`)

We choose the minimum of these two options.

Similarly, for the suffix part (right side), at each position with an illegal good, we can either:
- Remove all cars from this point to the right end (cost = `n - i`)
- Remove this car individually and handle the rest (cost = `suf[i + 1] + 2`)

The beauty of this approach is that it considers all possible strategies: removing everything from the left, removing everything from the right, or some combination where we remove from both ends and handle middle cars individually.

The final answer is the minimum sum of `pre[i] + suf[i]` across all positions, which represents the optimal split point where we've minimized the total cost of clearing illegal goods from both sides.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution uses [dynamic programming](/problems/dynamic_programming_intro) with two arrays to track the minimum cost from both directions.

**Data Structures:**
- `pre[i+1]`: Stores the minimum cost to remove all illegal goods from the first `i` cars
- `suf[i]`: Stores the minimum cost to remove all illegal goods from car `i` to the end
- Both arrays have size `n+1` to handle boundary cases

**Algorithm Implementation:**

1. **Build the prefix array (`pre`):**
   ```python
   for i, c in enumerate(s):
       pre[i + 1] = pre[i] if c == '0' else min(pre[i] + 2, i + 1)
   ```
   - If car `i` is clean (`c == '0'`), the cost remains the same as `pre[i]`
   - If car `i` has illegal goods (`c == '1'`), we choose the minimum between:
     - `pre[i] + 2`: Keep previous arrangement and remove this car individually
     - `i + 1`: Remove all cars from the left up to and including this position

2. **Build the suffix array (`suf`):**
   ```python
   for i in range(n - 1, -1, -1):
       suf[i] = suf[i + 1] if s[i] == '0' else min(suf[i + 1] + 2, n - i)
   ```
   - Process from right to left
   - If car `i` is clean, cost remains `suf[i + 1]`
   - If car `i` has illegal goods, choose minimum between:
     - `suf[i + 1] + 2`: Remove this car individually and handle the rest
     - `n - i`: Remove all cars from this position to the right end

3. **Find the optimal split:**
   ```python
   return min(a + b for a, b in zip(pre[1:], suf[1:]))
   ```
   - For each possible split position, calculate `pre[i] + suf[i]`
   - The minimum sum represents the optimal strategy
   - We use `pre[1:]` and `suf[1:]` to align the arrays properly (avoiding the boundary indices)

**Time Complexity:** `O(n)` - Single pass for each array construction and one pass to find minimum
**Space Complexity:** `O(n)` - Two arrays of size `n+1`

## Example Walkthrough

Let's walk through a small example with `s = "0110"` to illustrate the solution approach.

**Step 1: Initialize arrays**
- `n = 4`
- `pre = [0, 0, 0, 0, 0]` (size n+1)
- `suf = [0, 0, 0, 0, 0]` (size n+1)

**Step 2: Build prefix array (left to right)**

Processing each character:
- `i=0, s[0]='0'`: Clean car, so `pre[1] = pre[0] = 0`
- `i=1, s[1]='1'`: Illegal goods! 
  - Option 1: Remove individually: `pre[1] + 2 = 0 + 2 = 2`
  - Option 2: Remove all from left: `i + 1 = 1 + 1 = 2`
  - `pre[2] = min(2, 2) = 2`
- `i=2, s[2]='1'`: Illegal goods!
  - Option 1: Remove individually: `pre[2] + 2 = 2 + 2 = 4`
  - Option 2: Remove all from left: `i + 1 = 2 + 1 = 3`
  - `pre[3] = min(4, 3) = 3`
- `i=3, s[3]='0'`: Clean car, so `pre[4] = pre[3] = 3`

Result: `pre = [0, 0, 2, 3, 3]`

**Step 3: Build suffix array (right to left)**

Processing from the end:
- `i=3, s[3]='0'`: Clean car, so `suf[3] = suf[4] = 0`
- `i=2, s[2]='1'`: Illegal goods!
  - Option 1: Remove individually: `suf[3] + 2 = 0 + 2 = 2`
  - Option 2: Remove all from right: `n - i = 4 - 2 = 2`
  - `suf[2] = min(2, 2) = 2`
- `i=1, s[1]='1'`: Illegal goods!
  - Option 1: Remove individually: `suf[2] + 2 = 2 + 2 = 4`
  - Option 2: Remove all from right: `n - i = 4 - 1 = 3`
  - `suf[1] = min(4, 3) = 3`
- `i=0, s[0]='0'`: Clean car, so `suf[0] = suf[1] = 3`

Result: `suf = [3, 3, 2, 0, 0]`

**Step 4: Find optimal split**

We need to find the minimum of `pre[i] + suf[i]` for valid split points:
- Split at position 1: `pre[1] + suf[1] = 0 + 3 = 3`
- Split at position 2: `pre[2] + suf[2] = 2 + 2 = 4`
- Split at position 3: `pre[3] + suf[3] = 3 + 0 = 3`
- Split at position 4: `pre[4] + suf[4] = 3 + 0 = 3`

**Answer: 3**

This means the optimal strategy costs 3 units of time. One optimal approach would be to remove the 3 rightmost cars (positions 1, 2, 3), which removes both '1's and costs 3 units total.

## Python Solution

```python
class Solution:
    def minimumTime(self, s: str) -> int:
        """
        Find minimum time to remove all '1's from string s.
        Operations allowed:
        - Remove from left end: cost 1
        - Remove from right end: cost 1  
        - Remove from middle: cost 2
      
        Args:
            s: Binary string containing '0's and '1's
          
        Returns:
            Minimum time to remove all '1's
        """
        n = len(s)
      
        # prefix_cost[i] = minimum cost to remove all '1's in s[0:i]
        prefix_cost = [0] * (n + 1)
      
        # suffix_cost[i] = minimum cost to remove all '1's in s[i:n]
        suffix_cost = [0] * (n + 1)
      
        # Build prefix costs: for each position, decide whether to:
        # 1. Remove all elements from left (cost = i + 1)
        # 2. Keep previous strategy and remove current element from middle if it's '1' (cost = previous + 2)
        for i, char in enumerate(s):
            if char == '0':
                # No '1' at current position, carry forward previous cost
                prefix_cost[i + 1] = prefix_cost[i]
            else:
                # '1' at current position, choose minimum between:
                # - Remove as middle element (previous cost + 2)
                # - Remove all from left up to current position (i + 1)
                prefix_cost[i + 1] = min(prefix_cost[i] + 2, i + 1)
      
        # Build suffix costs: for each position from right, decide whether to:
        # 1. Remove all elements from right (cost = n - i)
        # 2. Keep previous strategy and remove current element from middle if it's '1' (cost = previous + 2)
        for i in range(n - 1, -1, -1):
            if s[i] == '0':
                # No '1' at current position, carry forward previous cost
                suffix_cost[i] = suffix_cost[i + 1]
            else:
                # '1' at current position, choose minimum between:
                # - Remove as middle element (previous cost + 2)
                # - Remove all from right starting from current position (n - i)
                suffix_cost[i] = min(suffix_cost[i + 1] + 2, n - i)
      
        # Find optimal split point: remove all '1's in left part and right part
        # The total cost at split point i is prefix_cost[i] + suffix_cost[i]
        return min(left_cost + right_cost 
                   for left_cost, right_cost in zip(prefix_cost[1:], suffix_cost[1:]))
```

## Time and Space Complexity

**Time Complexity:** `O(n)` where `n` is the length of the string `s`.

The algorithm consists of three main parts:
- First loop: Iterates through the string once to build the `pre` array, performing `O(1)` operations for each element - `O(n)` time
- Second loop: Iterates through the string once in reverse to build the `suf` array, performing `O(1)` operations for each element - `O(n)` time  
- Final computation: Uses `zip` and `min` to iterate through `n` elements once, performing `O(1)` addition for each pair - `O(n)` time

Total time complexity: `O(n) + O(n) + O(n) = O(n)`

**Space Complexity:** `O(n)` where `n` is the length of the string `s`.

The algorithm uses:
- `pre` array: stores `n + 1` elements - `O(n)` space
- `suf` array: stores `n + 1` elements - `O(n)` space
- The `zip` operation creates an iterator (not a new list) - `O(1)` space
- Other variables (`i`, `c`, `n`) use constant space - `O(1)` space

Total space complexity: `O(n) + O(n) + O(1) = O(n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Off-by-One Errors in Array Indexing**

The most common mistake is misaligning the prefix and suffix arrays when calculating the final answer. Since `pre[i]` represents the cost for the first `i` cars (0-indexed), and `suf[i]` represents the cost from car `i` to the end, developers often incorrectly combine them.

**Incorrect Implementation:**
```python
# Wrong: This double-counts or misses elements at boundaries
return min(pre[i] + suf[i] for i in range(n))
```

**Correct Implementation:**
```python
# Correct: pre[i] covers [0, i) and suf[i] covers [i, n)
return min(pre[i] + suf[i] for i in range(n + 1))
```

### 2. **Forgetting to Handle Empty String or All-Zero Cases**

If the string contains no '1's or is empty, the algorithm should return 0, but improper initialization can cause issues.

**Problem Example:**
```python
# If s = "000", without proper handling:
# Some implementations might still calculate non-zero costs
```

**Solution:**
Initialize both arrays with zeros and ensure the logic handles '0' characters by carrying forward the previous cost without modification.

### 3. **Incorrect Cost Calculation for Removing from Ends**

A frequent error is using the wrong formula for the cost of removing all cars from an end.

**Incorrect:**
```python
# Wrong: Using i instead of i+1 for left removal
pre[i + 1] = min(pre[i] + 2, i)  # Should be i+1

# Wrong: Using n-i-1 instead of n-i for right removal  
suf[i] = min(suf[i + 1] + 2, n - i - 1)  # Should be n-i
```

**Correct:**
```python
# Removing first i+1 cars costs i+1 units
pre[i + 1] = min(pre[i] + 2, i + 1)

# Removing cars from position i to end costs n-i units
suf[i] = min(suf[i + 1] + 2, n - i)
```

### 4. **Not Considering the Option to Remove Everything from One Side**

Some implementations forget that removing all cars from one side might be optimal, especially for strings like "00001" or "10000".

**Solution:**
Always include the boundary cases in the final comparison:
- `pre[n] + suf[n]` represents removing all from the left (cost = n)
- `pre[0] + suf[0]` represents removing all from the right (cost = n)

These should be naturally included when iterating through all split points from 0 to n.

