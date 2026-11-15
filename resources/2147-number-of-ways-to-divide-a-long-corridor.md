# 2147. Number of Ways to Divide a Long Corridor

> Source: https://algo.monster/liteproblems/2147

## Problem Statement

You are given a string `corridor` representing a library corridor with seats (`'S'`) and plants (`'P'`). The corridor already has dividers at both ends (before index 0 and after index n-1).

Your task is to install additional room dividers to divide the corridor into sections, where:
- Each section must contain **exactly 2 seats**
- Each section can have any number of plants
- Dividers can only be placed between adjacent positions (between indices `i-1` and `i`)
- At most one divider can be installed at each position

You need to count the number of different ways to divide the corridor. Two ways are considered different if there exists at least one position where a divider is installed in one way but not in the other.

For example, if the corridor is `"SSPPSPS"`:
- One valid way could be to place dividers to create sections like: `[SS][PP][SP][S]` - but this is invalid as the last section has only 1 seat
- A valid division would ensure each section has exactly 2 seats

The solution uses [dynamic programming](/problems/dynamic_programming_intro) with memoization. The function `dfs(i, k)` represents:
- `i`: current position in the corridor
- `k`: number of seats in the current section being formed

The algorithm works by:
1. Traversing each position and counting seats in the current section
2. When a section has 2 seats, we can either:
   - Continue adding to the current section (if we encounter plants)
   - Start a new section (place a divider and reset seat count to 0)
3. If we ever have more than 2 seats in a section, that path is invalid

The answer should be returned modulo `10^9 + 7`. If no valid division exists (e.g., odd number of total seats), return `0`.

## Intuition

The key insight is that we need to make decisions about where to place dividers as we traverse the corridor from left to right. At each position, we need to track how many seats we've accumulated in our current section.

Think of it like walking down the corridor and counting seats. Every time we see a seat (`'S'`), we add it to our current section's count. When we have exactly 2 seats in our section, we have a choice:
- We can close off this section by placing a divider (start fresh with 0 seats)
- We can continue without placing a divider (keep the 2 seats and continue)

The second option only makes sense if we encounter plants (`'P'`) next, because if we encounter another seat without placing a divider, we'd have 3 seats in one section, which violates the constraint.

This naturally leads to a recursive approach where at each position `i` with `k` seats already counted:
1. If we've reached the end of the corridor, we check if `k = 2` (valid section)
2. If we see a seat, increment `k`
3. If `k > 2`, this path is invalid (return 0)
4. Otherwise, we always try continuing without a divider: `dfs(i + 1, k)`
5. Additionally, if `k = 2`, we can also try placing a divider: `dfs(i + 1, 0)`

The beauty of this approach is that it automatically handles all the edge cases:
- Odd number of total seats will never produce valid divisions
- Sections with too many seats are pruned immediately
- Plant positions naturally become decision points when we have 2 seats

Memoization (`@cache`) is crucial here because many subproblems repeat - we often reach the same position with the same number of seats through different division choices earlier in the corridor.

**Learn more about [Math](/problems/math-basics) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution uses **memoization search ([dynamic programming](/problems/dynamic_programming_intro) with recursion)** to explore all valid ways to partition the corridor.

### Implementation Details

We define a recursive function `dfs(i, k)` where:
- `i` represents the current position in the corridor string
- `k` represents the number of seats in the current section being formed

**Base Case:**
When `i >= len(corridor)`, we've traversed the entire corridor. We return `1` if `k == 2` (indicating the last section has exactly 2 seats), otherwise return `0`.

**Recursive Case:**
1. First, check if the current position has a seat: if `corridor[i] == 'S'`, increment `k` by 1
2. If `k > 2`, the current section has too many seats, making this path invalid - return `0`
3. Otherwise, we have two choices:
   - **Don't place a divider**: Continue to the next position with the current seat count: `dfs(i + 1, k)`
   - **Place a divider** (only if `k == 2`): Start a new section with 0 seats: `dfs(i + 1, 0)`

The total number of ways is the sum of both choices (when applicable), taken modulo `10^9 + 7`.

### Key Components

**Memoization with `@cache`:**
The decorator automatically caches results of `dfs(i, k)` calls. This prevents redundant calculations when we reach the same state `(i, k)` through different partition choices.

**Modulo Operation:**
Since the answer can be very large, we apply modulo `10^9 + 7` when combining results: `ans = (ans + dfs(i + 1, 0)) % mod`

**Algorithm Flow:**
```
Start at position 0 with 0 seats
For each position:
  - Count seats in current section
  - If section valid (≤2 seats):
    - Try continuing without divider
    - If exactly 2 seats, also try placing divider
  - Combine results with modulo
Return total number of valid partitions
```

The time complexity is `O(n)` with memoization (each state `(i, k)` is computed at most once, and `k` can only be 0, 1, or 2). The space complexity is also `O(n)` for the recursion stack and cache.

## Example Walkthrough

Let's walk through the corridor `"SSPPSPS"` to understand how the solution works.

**Initial Setup:**
- Start with `dfs(0, 0)` - position 0, with 0 seats in current section
- We need to explore all valid ways to divide this corridor

**Step-by-step traversal:**

1. **Position 0 ('S'):** `dfs(0, 0)`
   - Current char is 'S', so k becomes 1
   - k ≤ 2, so we continue: `dfs(1, 1)`

2. **Position 1 ('S'):** `dfs(1, 1)`
   - Current char is 'S', so k becomes 2
   - k = 2, we have two choices:
     - Continue without divider: `dfs(2, 2)`
     - Place divider and start new section: `dfs(2, 0)`

3. **Branch 1 - No divider after "SS":** `dfs(2, 2)`
   - Position 2 ('P'): k stays 2 (plants don't add seats)
   - k = 2, two choices again:
     - Continue: `dfs(3, 2)`
     - Place divider: `dfs(3, 0)`
 
   Following Branch 1.1: `dfs(3, 2)`
   - Position 3 ('P'): k stays 2
   - Continue with both choices...
 
   Following Branch 1.2: `dfs(3, 0)`
   - We've divided as "SSP|P..."
   - Position 3 ('P'): k stays 0
   - Continue...

4. **Branch 2 - Place divider after "SS":** `dfs(2, 0)`
   - We've divided as "SS|P..."
   - Position 2 ('P'): k stays 0
   - k < 2, only one choice: `dfs(3, 0)`

Continuing this process, let's trace one complete valid path:
- "SS|PPSPS" → First section "SS" has 2 seats ✓
- Continue from position 2 with k=0
- Positions 2-3 are plants, k stays 0
- Position 4 ('S'): k becomes 1
- Position 5 ('P'): k stays 1
- Position 6 ('S'): k becomes 2
- End of string: k = 2 ✓
- This gives us "SS|PPSPS" where second section has 2 seats

Another valid path:
- "SSPP|SPS" → First section "SSPP" has 2 seats ✓
- Divider placed after position 3
- Continue from position 4 with k=0
- Following similar logic leads to "SPS" with 2 seats ✓

The algorithm explores all such paths, counting only those where:
1. Every section has exactly 2 seats
2. The last section also has exactly 2 seats

Invalid paths (automatically pruned):
- Any path where k > 2 returns 0 immediately
- Any path ending with k ≠ 2 returns 0

The memoization ensures that when we reach the same `(position, seat_count)` state through different earlier choices, we don't recalculate - we just reuse the cached result.

For this example, the function would return 2, representing the two valid ways to divide the corridor.

## Python Solution

```python
class Solution:
    def numberOfWays(self, corridor: str) -> int:
        """
        Calculate the number of ways to divide a corridor into sections
        where each section contains exactly 2 seats.
      
        Args:
            corridor: String containing 'S' (seats) and 'P' (plants)
      
        Returns:
            Number of valid ways to divide the corridor, modulo 10^9 + 7
        """
        from functools import cache
      
        MOD = 10**9 + 7
      
        @cache
        def count_divisions(index: int, seats_in_section: int) -> int:
            """
            Recursively count valid divisions starting from given index.
          
            Args:
                index: Current position in the corridor
                seats_in_section: Number of seats in the current section (0, 1, or 2)
          
            Returns:
                Number of valid ways to divide from this position
            """
            # Base case: reached end of corridor
            if index >= len(corridor):
                # Valid only if current section has exactly 2 seats
                return 1 if seats_in_section == 2 else 0
          
            # Count seats if current position has a seat
            if corridor[index] == "S":
                seats_in_section += 1
          
            # Invalid case: more than 2 seats in a section
            if seats_in_section > 2:
                return 0
          
            # Option 1: Continue current section (don't place divider)
            ways = count_divisions(index + 1, seats_in_section)
          
            # Option 2: Place divider after current position if section has 2 seats
            if seats_in_section == 2:
                ways = (ways + count_divisions(index + 1, 0)) % MOD
          
            return ways
      
        # Calculate result starting from index 0 with 0 seats
        result = count_divisions(0, 0)
      
        # Clear cache to free memory
        count_divisions.cache_clear()
      
        return result
```

## Time and Space Complexity

The time complexity is `O(n)`, where `n` is the length of the corridor string. This is because the memoized DFS function has at most `O(n)` unique states. Each state is defined by the pair `(i, k)`, where `i` ranges from `0` to `n-1` and `k` can only be `0`, `1`, or `2`. This gives us at most `3n` unique states. Each state is computed only once due to the `@cache` decorator, and each computation takes `O(1)` time (excluding recursive calls). Therefore, the overall time complexity is `O(n)`.

The space complexity is `O(n)`. This comes from two sources: the recursion call stack which can go up to depth `n` in the worst case when traversing the entire corridor, and the cache storage which stores at most `O(n)` states (specifically at most `3n` states as analyzed above). Both contribute to `O(n)` space complexity, resulting in an overall space complexity of `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Divider Placement Logic**

A critical misunderstanding is thinking that when we have 2 seats in a section, we *must* place a divider immediately. This leads to missing valid configurations where plants appear after the second seat but before the divider.

**Wrong Approach:**
```python
if seats_in_section == 2:
    # Force divider placement - WRONG!
    return count_divisions(index + 1, 0)
```

**Correct Approach:**
```python
if seats_in_section == 2:
    # We have TWO choices: place divider OR continue
    ways = (ways + count_divisions(index + 1, 0)) % MOD
```

For example, with `"SSPP"`:
- We can divide as `[SS][PP]` (divider after position 1)
- We can also divide as `[SSPP]` (only boundary dividers)
Both are valid since each section has exactly 2 seats.

### 2. **Mishandling the Base Case**

Another common mistake is not properly validating the final section when reaching the end of the corridor.

**Wrong Base Case:**
```python
if index >= len(corridor):
    return 1  # Always returns 1 - WRONG!
```

**Correct Base Case:**
```python
if index >= len(corridor):
    # Only valid if the last section has exactly 2 seats
    return 1 if seats_in_section == 2 else 0
```

### 3. **Forgetting Edge Cases**

The algorithm might seem to work but fail on edge cases like:
- Corridors with odd number of total seats (impossible to divide into sections of 2)
- Corridors with only plants `"PPP"` (returns 0 correctly)
- Corridors with exactly 2 seats `"SS"` (should return 1)

**Solution:** The recursive approach naturally handles these cases:
- Odd seats: The base case will never have `seats_in_section == 2` for the final section
- No seats: The function returns 0 as no valid division exists
- Exactly 2 seats: Returns 1 as there's exactly one valid way (no internal dividers)

### 4. **Memory Limit Issues with Large Inputs**

Without clearing the cache after computation, the memoization decorator can consume significant memory for large inputs.

**Best Practice:**
```python
result = count_divisions(0, 0)
count_divisions.cache_clear()  # Clear cache to free memory
return result
```

### 5. **Integer Overflow Without Modulo**

Forgetting to apply modulo at each step can cause integer overflow for large results.

**Wrong:**
```python
ways = ways + count_divisions(index + 1, 0)  # May overflow
```

**Correct:**
```python
ways = (ways + count_divisions(index + 1, 0)) % MOD
```

