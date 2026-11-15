# 2999. Count the Number of Powerful Integers

> Source: https://algo.monster/liteproblems/2999

## Problem Statement

You need to find how many "powerful" integers exist within a given range `[start, finish]`.

A positive integer is considered **powerful** if it meets two conditions:
1. It ends with the string `s` (meaning `s` is a suffix of the number when viewed as a string)
2. Every digit in the number is at most `limit`

For example, if `s = "25"` and `limit = 5`, then:
- `525` would be powerful (ends with "25" and all digits ≤ 5)
- `625` would NOT be powerful (ends with "25" but has digit 6 > 5)
- `523` would NOT be powerful (doesn't end with "25")

The solution uses **Digit [Dynamic Programming](/problems/dynamic_programming_intro) (Digit DP)** to efficiently count valid numbers. The key insight is to transform the problem into finding the difference between two counts:

`answer = count([1, finish]) - count([1, start-1])`

The recursive function `dfs(pos, lim)` builds numbers digit by digit from left to right:
- `pos`: current position being processed
- `lim`: whether we're still bounded by the upper limit number

The algorithm works as follows:

1. If the number being built has fewer digits than `s`, it can't possibly end with `s`, so return 0
2. When we've built enough digits to potentially match `s`, check if the remaining positions can form `s` as a suffix
3. Otherwise, try placing each valid digit (0 to min of current limit and `limit` parameter) at the current position and recursively count valid numbers

The `@cache` decorator memorizes results to avoid recalculating the same states, making the solution efficient even for large ranges.

## Intuition

When we need to count numbers in a range with specific digit constraints, a brute force approach of checking every number would be too slow for large ranges. This is where we recognize a pattern that leads us to Digit DP.

The key observation is that we're dealing with constraints on individual digits and a suffix requirement. Instead of generating all numbers, we can build valid numbers digit by digit from left to right, keeping track of whether we're still constrained by the upper bound.

Think of it like filling in blanks: `_ _ _ _ 2 5` where the last two positions must be "25" (our suffix `s`), and each blank can only be filled with digits from 0 to `limit`. 

The insight to use `count([1, finish]) - count([1, start-1])` comes from the principle of complementary counting - it's easier to count from 1 to a number than to count within an arbitrary range directly.

As we build numbers digit by digit, we face decisions at each position:
- If we haven't placed enough digits yet to accommodate the suffix, some numbers will be too short
- Once we reach the point where the remaining positions exactly match the suffix length, we must check if placing the suffix would create a valid number
- The `lim` flag is crucial - it tells us whether we're still bounded by the original number's digits or if we can freely choose any digit up to `limit`

For example, if we're counting up to 5234 and we've placed "52" so far, the next digit can only be 0-3 to stay within bounds. But if we had placed "51", then for the next position we could use any digit from 0 to our `limit`.

Memoization naturally fits here because the same sub-problems (remaining positions with same constraints) appear multiple times. Once we know how many valid numbers can be formed from a certain position with certain constraints, we can reuse that result.

**Learn more about [Math](/problems/math-basics) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution implements Digit DP with memoization to efficiently count powerful integers. Here's how the implementation works:

**Core Function Design:**

The main recursive function `dfs(pos, lim)` represents:
- `pos`: Current digit position we're filling (0-indexed from left)
- `lim`: Boolean flag indicating if we're still bounded by the upper limit number `t`

**Step-by-step Implementation:**

1. **Base Case - Insufficient Digits:**
   ```python
   if len(t) < n:
       return 0
   ```
   If the target number `t` has fewer digits than the suffix `s`, it's impossible to form a valid powerful integer.

2. **Base Case - Suffix Position Reached:**
   ```python
   if len(t) - pos == n:
       return int(s <= t[pos:]) if lim else 1
   ```
   When we've filled enough positions and only the suffix length remains:
   - If `lim` is True (still bounded), check if placing suffix `s` keeps us within bounds
   - If `lim` is False (already smaller than upper bound), we can always place the suffix

3. **Recursive Case - Fill Current Position:**
   ```python
   up = min(int(t[pos]) if lim else 9, limit)
   ans = 0
   for i in range(up + 1):
       ans += dfs(pos + 1, lim and i == int(t[pos]))
   ```
   - Calculate upper bound `up` for current digit: minimum of the digit constraint and the limit parameter
   - Try each valid digit from 0 to `up`
   - Update `lim` for next position: remains True only if we chose the exact digit from `t[pos]`

4. **Main Algorithm - Range Calculation:**
   ```python
   n = len(s)
   t = str(start - 1)
   a = dfs(0, True)
   dfs.cache_clear()
   t = str(finish)
   b = dfs(0, True)
   return b - a
   ```
   - Convert the range problem to: `count([1, finish]) - count([1, start-1])`
   - Clear cache between calculations to avoid interference
   - Return the difference as the final answer

**Key Optimizations:**

- **Memoization (`@cache`):** Stores results of `dfs(pos, lim)` to avoid recalculating identical subproblems
- **Early Termination:** Returns 0 immediately when the number can't possibly satisfy the suffix requirement
- **Tight Bounds:** The `lim` flag ensures we only explore valid numbers within the range

**Time Complexity:** `O(D × 10 × 2)` where D is the number of digits in `finish`. Each position has at most 10 digit choices and 2 states for `lim`.

**Space Complexity:** `O(D × 2)` for the memoization cache.

## Example Walkthrough

Let's walk through finding powerful integers in the range `[50, 120]` where `s = "5"` and `limit = 6`.

**Step 1: Transform the problem**
- We need: `count([1, 120]) - count([1, 49])`
- First calculate `count([1, 49])`, then `count([1, 120])`

**Calculating count([1, 49]):**

We build numbers digit by digit, with `t = "49"`:

Starting at `dfs(0, True)` (position 0, still bounded):
- Since `len("49") - 0 = 2` and suffix length is 1, we continue
- Upper bound: `min(4, 6) = 4` (digit at position 0 is '4', limit is 6)
- Try digits 0-4:
  - Digit 0: `dfs(1, False)` - now unbounded since 0 < 4
  - Digit 1: `dfs(1, False)` - unbounded since 1 < 4  
  - Digit 2: `dfs(1, False)` - unbounded since 2 < 4
  - Digit 3: `dfs(1, False)` - unbounded since 3 < 4
  - Digit 4: `dfs(1, True)` - still bounded since 4 = 4

For `dfs(1, False)` (position 1, unbounded):
- Now `len("49") - 1 = 1` equals suffix length
- Since unbounded, we can place suffix "5" → returns 1

For `dfs(1, True)` (position 1, bounded):
- Now `len("49") - 1 = 1` equals suffix length
- Check if "5" ≤ "9" (remaining part of "49") → True → returns 1

Total for `count([1, 49])`: 0 + 1 + 1 + 1 + 1 + 1 = **5**
(Numbers: 5, 15, 25, 35, 45)

**Calculating count([1, 120]):**

With `t = "120"`:

Starting at `dfs(0, True)`:
- Since `len("120") - 0 = 3` and suffix length is 1, we continue
- Upper bound: `min(1, 6) = 1`
- Try digits 0-1:
  - Digit 0: `dfs(1, False)` 
  - Digit 1: `dfs(1, True)`

For `dfs(1, False)` (position 1, unbounded):
- `len("120") - 1 = 2` > suffix length 1, continue
- Upper bound: `min(9, 6) = 6` (unbounded, so use limit)
- Try digits 0-6, each leads to `dfs(2, False)`
- Each `dfs(2, False)` returns 1 (can place suffix "5")
- Subtotal: 7

For `dfs(1, True)` (position 1, bounded by "20"):
- Upper bound: `min(2, 6) = 2`
- Try digits 0-2:
  - Digit 0: `dfs(2, False)` → returns 1
  - Digit 1: `dfs(2, False)` → returns 1
  - Digit 2: `dfs(2, True)` → checks "5" ≤ "0" → False → returns 0
- Subtotal: 2

Total for `count([1, 120])`: 7 + 2 = **9**
(Numbers: 5, 15, 25, 35, 45, 55, 65 from first branch + 105, 115 from second branch)

**Final Answer:** 9 - 5 = **4** powerful integers in range [50, 120]
(These are: 55, 65, 105, 115)

## Python Solution

```python
class Solution:
    def numberOfPowerfulInt(self, start: int, finish: int, limit: int, s: str) -> int:
        """
        Count powerful integers in range [start, finish] where:
        - Each digit doesn't exceed 'limit'
        - The number ends with suffix 's'
        """
        from functools import cache
      
        @cache
        def count_valid_numbers(position: int, is_tight: int) -> int:
            """
            Digit DP to count valid numbers.
          
            Args:
                position: Current position in the number being built
                is_tight: Whether we're still bounded by the upper limit number
          
            Returns:
                Count of valid numbers from this state
            """
            # If current number has fewer digits than suffix, it's invalid
            if len(current_bound) < suffix_length:
                return 0
          
            # Check if we've reached the position where suffix should start
            if len(current_bound) - position == suffix_length:
                # If tight, check if remaining part is >= suffix
                # If not tight, any number with suffix is valid
                return int(s <= current_bound[position:]) if is_tight else 1
          
            # Determine the maximum digit we can place at current position
            if is_tight:
                max_digit = min(int(current_bound[position]), limit)
            else:
                max_digit = limit
          
            # Try all possible digits from 0 to max_digit
            total_count = 0
            for digit in range(max_digit + 1):
                # Recurse to next position
                # Remain tight only if we're currently tight and chose the maximum possible digit
                total_count += count_valid_numbers(
                    position + 1, 
                    is_tight and digit == int(current_bound[position])
                )
          
            return total_count
      
        # Length of the required suffix
        suffix_length = len(s)
      
        # Count numbers <= (start - 1)
        current_bound = str(start - 1)
        count_below_start = count_valid_numbers(0, True)
      
        # Clear cache before computing for the upper bound
        count_valid_numbers.cache_clear()
      
        # Count numbers <= finish
        current_bound = str(finish)
        count_up_to_finish = count_valid_numbers(0, True)
      
        # Return count of numbers in range [start, finish]
        return count_up_to_finish - count_below_start
```

## Time and Space Complexity

The time complexity is `O(log M × D)`, where `M` represents the maximum value between `start` and `finish`, and `D = 10` (the number of possible digits 0-9).

**Time Complexity Analysis:**
- The recursive function `dfs` explores digit positions from left to right in the number representation
- The maximum depth of recursion is `O(log M)` since we process each digit position of the number (the number of digits in a number `M` is proportional to `log M`)
- At each position, we iterate through at most `min(limit + 1, 10)` choices, which is bounded by `D = 10`
- The function is called twice (once for `start - 1` and once for `finish`)
- With memoization via `@cache`, each unique state `(pos, lim)` is computed only once
- There are at most `O(log M)` positions and 2 possible values for `lim` (True/False), giving us `O(log M)` unique states
- Therefore, the total time complexity is `O(log M × D)`

**Space Complexity Analysis:**
- The recursion depth is at most `O(log M)` (the number of digits)
- The memoization cache stores at most `O(log M)` unique states (considering `pos` ranges from 0 to number of digits, and `lim` has 2 values)
- The string `t` takes `O(log M)` space to store the number representation
- Therefore, the total space complexity is `O(log M)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Suffix Comparison with Leading Zeros**
**Problem:** When comparing if the suffix `s` can be placed at the end, the code directly compares strings: `s <= current_bound[position:]`. This can fail when the suffix has leading zeros.

For example, if `s = "007"` and we're checking if it fits in the remaining part "123", the string comparison `"007" <= "123"` returns `True`, but numerically placing "007" at those positions would create an invalid number.

**Solution:** Convert to integers for proper numerical comparison:
```python
if len(current_bound) - position == suffix_length:
    if is_tight:
        # Compare as integers to handle leading zeros correctly
        return 1 if int(s) <= int(current_bound[position:]) else 0
    else:
        return 1
```

### 2. **Not Validating Suffix Digits Against Limit**
**Problem:** The code assumes the suffix `s` itself is valid (all digits ≤ limit), but doesn't validate this. If `s = "789"` and `limit = 5`, the algorithm would still try to count numbers ending with "789", which is impossible.

**Solution:** Add validation before starting the computation:
```python
def numberOfPowerfulInt(self, start: int, finish: int, limit: int, s: str) -> int:
    # Validate that suffix itself satisfies the digit limit
    for digit_char in s:
        if int(digit_char) > limit:
            return 0  # No valid powerful integers possible
  
    # Rest of the implementation...
```

### 3. **Cache Pollution Between Test Cases**
**Problem:** Using `@cache` without proper cleanup between different test cases can cause incorrect results when the same `Solution` instance is reused with different parameters.

**Solution:** Use local caching or ensure proper cleanup:
```python
def numberOfPowerfulInt(self, start: int, finish: int, limit: int, s: str) -> int:
    from functools import lru_cache
  
    # Create a new cache for each function call
    @lru_cache(maxsize=None)
    def count_valid_numbers(position: int, is_tight: int) -> int:
        # Implementation...
  
    # Compute results
    current_bound = str(start - 1)
    count_below_start = count_valid_numbers(0, True)
  
    # Clear cache between bounds
    count_valid_numbers.cache_clear()
  
    current_bound = str(finish)
    count_up_to_finish = count_valid_numbers(0, True)
  
    # Clear cache after computation for next test case
    count_valid_numbers.cache_clear()
  
    return count_up_to_finish - count_below_start
```

### 4. **Edge Case: When start = 1**
**Problem:** Computing `start - 1` when `start = 1` gives 0, and `str(0) = "0"`. This edge case might not be handled correctly if the suffix length equals 1 and suffix is "0".

**Solution:** Handle the zero case explicitly:
```python
if start == 1:
    # Special handling for lower bound
    current_bound = "0"
    count_below_start = 0 if suffix_length > 1 or s != "0" else 1
else:
    current_bound = str(start - 1)
    count_below_start = count_valid_numbers(0, True)
```

### 5. **Integer Overflow for Large Ranges**
**Problem:** Although Python handles arbitrary precision integers, converting between strings and integers repeatedly for very large numbers can impact performance.

**Solution:** Keep computations in string format when possible and only convert when necessary for comparisons:
```python
# Instead of converting entire numbers, work with digit comparisons
if len(current_bound) - position == suffix_length:
    if is_tight:
        # Compare digit by digit instead of full integer conversion
        for i in range(suffix_length):
            if s[i] < current_bound[position + i]:
                return 1
            elif s[i] > current_bound[position + i]:
                return 0
        return 1  # Equal case
    else:
        return 1
```

