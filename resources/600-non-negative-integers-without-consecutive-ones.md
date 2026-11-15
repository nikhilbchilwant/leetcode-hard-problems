# 600. Non-negative Integers without Consecutive Ones

> Source: https://algo.monster/liteproblems/600

## Problem Statement

Given a positive integer `n`, you need to count how many integers in the range `[0, n]` have binary representations that **do not** contain consecutive ones (i.e., no "11" pattern).

For example:
- If `n = 5`, the binary representations are:
  - `0` → `0` (valid)
  - `1` → `1` (valid)
  - `2` → `10` (valid)
  - `3` → `11` (invalid - has consecutive ones)
  - `4` → `100` (valid)
  - `5` → `101` (valid)

The answer would be `5` since only the number `3` has consecutive ones in its binary representation.

The solution uses **Digit DP ([Dynamic Programming](/problems/dynamic_programming_intro))** technique. The key idea is to build valid numbers digit by digit from the most significant bit, ensuring we never place two consecutive `1`s.

The `dfs` function has three parameters:
- `i`: Current bit position (starting from the most significant bit)
- `pre`: The previous bit value (`0` or `1`)
- `limit`: Whether we're constrained by the upper bound `n`

The algorithm works as follows:

1. Start from the most significant bit of `n` and work downward
2. At each position, try placing either `0` or `1`
3. If the previous bit was `1` and we want to place `1`, skip this case (would create "11")
4. The `limit` flag ensures we don't exceed `n`:
   - If `limit` is `True`, we can only place bits up to the corresponding bit in `n`
   - If `limit` is `False`, we can freely choose `0` or `1`
5. Use memoization (`@cache`) to avoid recalculating the same states

The time complexity is `O(log n)` since we process each bit position once, and space complexity is also `O(log n)` for the recursion stack and memoization.

## Intuition

When we need to count numbers with specific digit patterns in a range, a brute force approach would be to check every number from `0` to `n`. However, this becomes inefficient for large values of `n`.

The key insight is that we can build valid numbers **digit by digit** from left to right (most significant to least significant bit). At each step, we only need to track:
1. What digit we placed previously (to avoid consecutive `1`s)
2. Whether we're still bounded by `n` (to ensure we don't exceed it)

Think of it like filling in blanks: `_ _ _ _` for a 4-bit number. When filling each position:
- If the previous position has `0`, the current position can be either `0` or `1`
- If the previous position has `1`, the current position must be `0` (to avoid "11")

The "limit" concept is crucial here. Consider `n = 1010` (binary):
- If we've built `10__` so far and stayed within the limit, the next bit can only be `0` or `1` (following `n`'s pattern)
- But if we've built `00__`, we've already gone below `n`, so we can freely choose any valid combination for remaining bits

This naturally leads to a recursive solution where we:
1. Process one bit at a time from left to right
2. Make decisions based on the previous bit and whether we're still at the upper bound
3. Count all valid combinations

The beauty of this approach is that many subproblems repeat. For instance, once we know how many valid numbers can be formed with the last `k` bits (without limit), we can reuse this result. This is why memoization dramatically improves performance - we solve each unique state only once.

The approach transforms an `O(n)` enumeration problem into an `O(log n)` digit-building problem, making it efficient even for very large values of `n`.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution implements **Digit DP** using memoized recursion. Let's walk through the implementation step by step:

### Core Function Design

The main recursive function `dfs(i, pre, limit)` has three parameters:
- `i`: Current bit position index (counting from right, starting at the most significant bit)
- `pre`: The previous bit value (`0` or `1`)
- `limit`: Boolean flag indicating if we're still bounded by `n`

### Implementation Details

1. **Getting the bit length**: 
   ```python
   n.bit_length() - 1
   ```
   This gives us the index of the most significant bit. For example, if `n = 5` (binary: `101`), the bit length is `3`, so we start at index `2`.

2. **Base case**:
   ```python
   if i < 0:
       return 1
   ```
   When we've processed all bits (index becomes negative), we've successfully formed one valid number.

3. **Determining the upper bound for current bit**:
   ```python
   up = (n >> i & 1) if limit else 1
   ```
   - If `limit` is `True`: We can only place bits up to what `n` has at position `i`
   - If `limit` is `False`: We can place either `0` or `1`

4. **Iterating through possible digits**:
   ```python
   for j in range(up + 1):
       if pre and j:
           continue
       ans += dfs(i - 1, j, limit and j == up)
   ```
   - We try each possible bit value `j` from `0` to `up`
   - Skip if both `pre` and `j` are `1` (would create consecutive ones)
   - Recurse with:
     - Next bit position: `i - 1`
     - Current bit becomes the new previous: `j`
     - Update limit: remains `True` only if we're still at limit AND placed the maximum allowed bit

5. **Memoization with `@cache`**:
   The decorator automatically caches results for each unique `(i, pre, limit)` combination, preventing redundant calculations.

### Example Walkthrough

For `n = 5` (binary: `101`):
- Start with `dfs(2, 0, True)` (bit position 2, no previous bit, limited by n)
- At position 2: Can place `0` or `1` (since `n` has `1` at position 2)
  - If we place `1`: Continue with `dfs(1, 1, True)`
  - If we place `0`: Continue with `dfs(1, 0, False)` (no longer at limit)
- At position 1: Depends on previous choices
  - If previous was `1` and limit is `True`: Can only place `0` (since `n` has `0` here)
  - If previous was `0` and no limit: Can place `0` or `1`
- Continue until all positions are filled

The algorithm efficiently counts all valid numbers by exploring the decision tree of bit placements while pruning invalid branches (consecutive ones) and using memoization to avoid recomputation.

## Example Walkthrough

Let's trace through the algorithm with `n = 5` (binary: `101`).

**Initial Setup:**
- `n = 5` → binary: `101` (3 bits)
- Start position: `i = 2` (most significant bit index)
- Initial call: `dfs(2, 0, True)`

**Step-by-step execution:**

```
dfs(2, 0, True)  // Position 2, no previous bit, limited by n
├─ Bit at position 2 in n = 1, so up = 1
├─ Try j = 0:
│  └─ dfs(1, 0, False)  // Placed 0, no longer at limit (0 < 1)
│     ├─ No limit, so up = 1
│     ├─ Try j = 0:
│     │  └─ dfs(0, 0, False)
│     │     ├─ Try j = 0: dfs(-1, 0, False) → returns 1 (number: 000 = 0)
│     │     └─ Try j = 1: dfs(-1, 1, False) → returns 1 (number: 001 = 1)
│     │     Total: 2
│     └─ Try j = 1:
│        └─ dfs(0, 1, False)
│           └─ Try j = 0: dfs(-1, 0, False) → returns 1 (number: 010 = 2)
│           └─ Try j = 1: SKIP (consecutive 1s)
│           Total: 1
│     Total from dfs(1, 0, False): 3
│
└─ Try j = 1:
   └─ dfs(1, 1, True)  // Placed 1, still at limit
      ├─ Bit at position 1 in n = 0, so up = 0
      └─ Try j = 0:
         └─ dfs(0, 0, True)  // Placed 0 = up, so still at limit (prefix 10)
            ├─ Bit at position 0 in n = 1, so up = 1
            ├─ Try j = 0: dfs(-1, 0, False) → returns 1 (number: 100 = 4)
            └─ Try j = 1: dfs(-1, 1, True) → returns 1 (number: 101 = 5)
            Total: 2
      (j = 1 is never tried here because up = 0; it would also create 11)
      Total from dfs(1, 1, True): 2

Final result: 3 + 2 = 5
```

**Valid numbers found:** 
- `000` (0) ✓
- `001` (1) ✓
- `010` (2) ✓
- `100` (4) ✓
- `101` (5) ✓

**Invalid number skipped:**
- `011` (3) ✗ - contains "11"

The algorithm correctly returns 5, counting all integers from 0 to 5 that don't contain consecutive 1s in their binary representation.

## Python Solution

```python
class Solution:
    def findIntegers(self, n: int) -> int:
        """
        Count the number of integers in range [0, n] that don't have 
        consecutive 1s in their binary representation.
      
        Args:
            n: Upper bound of the range (inclusive)
          
        Returns:
            Count of valid integers without consecutive 1s in binary
        """
        from functools import cache
      
        @cache
        def count_valid_numbers(bit_position: int, 
                                previous_bit: int, 
                                is_limit: bool) -> int:
            """
            Recursive function to count valid numbers using digit DP.
          
            Args:
                bit_position: Current bit position being processed (from MSB to LSB)
                previous_bit: The bit value placed at the previous (higher) position
                is_limit: Whether we're still bounded by the original number n
              
            Returns:
                Count of valid numbers from this state
            """
            # Base case: processed all bits successfully
            if bit_position < 0:
                return 1
          
            # Determine the upper bound for current bit
            # If we're at limit, we can only go up to the corresponding bit in n
            # Otherwise, we can use 0 or 1
            upper_bound = (n >> bit_position & 1) if is_limit else 1
          
            # Count valid numbers by trying each possible bit value
            total_count = 0
            for current_bit in range(upper_bound + 1):
                # Skip if we would have consecutive 1s (previous was 1 and current is 1)
                if previous_bit == 1 and current_bit == 1:
                    continue
              
                # Recursively count valid numbers with:
                # - Next bit position (moving right)
                # - Current bit becomes the previous for next recursion
                # - Update limit flag (remains true only if we're at limit AND chose maximum bit)
                total_count += count_valid_numbers(
                    bit_position - 1, 
                    current_bit, 
                    is_limit and (current_bit == upper_bound)
                )
          
            return total_count
      
        # Start from the most significant bit
        # Initial previous bit is 0 (no bit before MSB)
        # Initially we're bounded by n (is_limit = True)
        return count_valid_numbers(n.bit_length() - 1, 0, True)
```

## Time and Space Complexity

**Time Complexity:** `O(log n * 2 * 2)` = `O(log n)`

The time complexity is determined by the recursive DFS function with memoization. Let's analyze the state space:
- Parameter `i` ranges from `n.bit_length() - 1` down to `-1`, giving us `O(log n)` possible values
- Parameter `pre` can only be `0` or `1`, giving us `2` possible values  
- Parameter `limit` can only be `True` or `False`, giving us `2` possible values

The total number of unique states is `O(log n * 2 * 2)` = `O(log n)`. Due to memoization with `@cache`, each state is computed only once. Within each state, we iterate through at most 2 values (j from 0 to up, where up is at most 1), which is `O(1)` work per state.

Therefore, the overall time complexity is `O(log n)`.

**Space Complexity:** `O(log n)`

The space complexity consists of:
- **Recursion stack depth:** The maximum recursion depth is `n.bit_length()` which is `O(log n)`
- **Memoization cache:** The cache stores all unique states, which as analyzed above is `O(log n * 2 * 2)` = `O(log n)`

Therefore, the overall space complexity is `O(log n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Bit Indexing**
A common mistake is using the wrong bit indexing scheme, either starting from 0 instead of `n.bit_length() - 1`, or incrementing instead of decrementing the index.

**Wrong approach:**
```python
def dfs(i, pre, limit):
    if i >= n.bit_length():  # Wrong termination
        return 1
    # ... rest of logic
    dfs(i + 1, j, ...)  # Wrong direction
```

**Correct approach:**
```python
def dfs(i, pre, limit):
    if i < 0:  # Correct termination
        return 1
    # ... rest of logic
    dfs(i - 1, j, ...)  # Correct direction
```

### 2. **Forgetting to Handle the Limit Flag Properly**
The limit flag update is crucial but often misunderstood. A common error is to simply pass `limit` unchanged or always set it to `False` after the first non-maximum digit.

**Wrong approach:**
```python
# Incorrectly maintaining the limit flag
ans += dfs(i - 1, j, limit)  # Wrong: doesn't update based on current choice
# OR
ans += dfs(i - 1, j, False)  # Wrong: always removes limit after first bit
```

**Correct approach:**
```python
# Limit remains True only if we were at limit AND chose the maximum allowed bit
ans += dfs(i - 1, j, limit and j == up)
```

### 3. **Not Using Memoization or Using It Incorrectly**
Without memoization, the solution becomes exponentially slow. Also, using incorrect cache keys can lead to wrong results.

**Wrong approach:**
```python
# No memoization - exponential time complexity
def dfs(i, pre, limit):
    # ... recursive logic without caching
  
# Or wrong cache key
memo = {}
def dfs(i, pre, limit):
    if (i, pre) in memo:  # Missing 'limit' in cache key
        return memo[(i, pre)]
```

**Correct approach:**
```python
from functools import cache

@cache  # Automatically handles all parameters as cache key
def dfs(i, pre, limit):
    # ... recursive logic
```

### 4. **Incorrect Consecutive Ones Check**
Some implementations check for consecutive ones incorrectly by looking at the wrong conditions or positions.

**Wrong approach:**
```python
# Wrong: checking if current and next will be 1 (we don't know next yet)
if j == 1 and next_bit == 1:  
    continue

# Wrong: only checking when pre is 1, missing when j is 0
if pre == 1:
    continue
```

**Correct approach:**
```python
# Skip only when BOTH previous and current are 1
if pre == 1 and j == 1:
    continue
# Or more concisely:
if pre and j:
    continue
```

### 5. **Off-by-One Error in Bit Extraction**
When extracting the bit at position `i` from `n`, using the wrong shift amount is a common mistake.

**Wrong approach:**
```python
# Wrong: using bit_length() directly without -1
up = (n >> n.bit_length() & 1) if limit else 1

# Wrong: shifting in wrong direction
up = (n << i & 1) if limit else 1
```

**Correct approach:**
```python
# Correct: right shift by i positions and mask with 1
up = (n >> i & 1) if limit else 1
```

### 6. **Edge Case: n = 0**
While the code handles this correctly, it's worth noting that when `n = 0`, `n.bit_length()` returns 0, so we start with `dfs(-1, 0, True)` which immediately returns 1, giving the correct answer.

### Prevention Tips:
1. **Test with small examples**: Trace through `n = 5` manually to verify bit positions
2. **Add debugging output**: Print `(i, pre, limit)` states to understand the recursion flow
3. **Verify bit operations**: Test bit extraction separately with known values
4. **Check memoization effectiveness**: Count cache hits vs. misses to ensure it's working

