# 552. Student Attendance Record II

> Source: https://algo.monster/liteproblems/552

## Problem Statement

You need to count how many valid attendance records of length `n` can be created for a student to be eligible for an attendance award.

An attendance record is a string of length `n` where each character represents the student's status for that day:
- `'A'` means Absent
- `'L'` means Late  
- `'P'` means Present

For a student to be eligible for the attendance award, their record must satisfy **both** conditions:
1. **Less than 2 total absences**: The student can have at most 1 `'A'` in their entire record
2. **No 3 consecutive late days**: The student cannot have 3 or more `'L'`s in a row anywhere in the record

Given an integer `n`, you need to find the total number of different attendance records of length `n` that would make a student eligible for the award. Since this number can be very large, return the result modulo `10^9 + 7`.

For example, if `n = 2`, valid records include: `"PP"`, `"PL"`, `"LP"`, `"LL"`, `"AP"`, `"PA"`, `"AL"`, `"LA"`, `"LP"`. Records like `"AA"` (2 absences) would be invalid.

The solution uses [dynamic programming](/problems/dynamic_programming_intro) with memoization where `dfs(i, j, k)` represents:
- `i`: current position in the record being built
- `j`: number of absences used so far (0 or 1)
- `k`: current consecutive late count

At each position, we try adding:
- An absence `'A'` (if we haven't used one yet)
- A late `'L'` (if we haven't had 2 consecutive lates)
- A present `'P'` (always allowed, resets consecutive late count)

## Intuition

The key insight is that we need to track two critical pieces of information as we build each attendance record: how many absences we've used (since we can have at most 1), and how many consecutive lates we currently have (since we can't have 3 in a row).

Think about building the attendance record day by day. At each day, we have three choices - add `'A'`, `'L'`, or `'P'`. But these choices are constrained by our current state:
- We can only add `'A'` if we haven't already used our one allowed absence
- We can only add `'L'` if we don't already have 2 consecutive lates
- We can always add `'P'`, and doing so resets our consecutive late counter to 0

This naturally leads to a recursive approach where we explore all valid possibilities. The state space is manageable because:
- Position ranges from `0` to `n-1`
- Absence count is either `0` or `1` (once we hit 1, we can't add more)
- Consecutive late count ranges from `0` to `2` (once we hit 2, we can't add another `'L'`)

So we have at most `n × 2 × 3` unique states, making [dynamic programming](/problems/dynamic_programming_intro) with memoization efficient.

The recursive function `dfs(i, j, k)` counts all valid records starting from position `i` with `j` absences already used and `k` consecutive lates ending at position `i-1`. At each step:
- If `j == 0`, we can add an `'A'` and continue with `j = 1` and reset `k = 0`
- If `k < 2`, we can add an `'L'` and continue with `k + 1` consecutive lates
- We can always add a `'P'` and reset `k = 0`

The base case is when we've built a complete record of length `n`, which counts as 1 valid record. The sum of all paths gives us the total number of valid attendance records.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution implements a top-down [dynamic programming](/problems/dynamic_programming_intro) approach with memoization using Python's `@cache` decorator.

**State Definition:**
The recursive function `dfs(i, j, k)` represents:
- `i`: Current position in the attendance record (0-indexed)
- `j`: Number of absences used so far (0 or 1)
- `k`: Current count of consecutive lates

**Base Case:**
When `i >= n`, we've successfully built a valid attendance record of length `n`, so we return `1`.

**Recursive Transitions:**
At each position `i`, we explore three possible actions:

1. **Add an Absence ('A'):**
   - Only possible if `j == 0` (haven't used our one allowed absence)
   - Transition: `dfs(i + 1, j + 1, 0)`
   - We increment `j` to 1 and reset `k` to 0 (consecutive late count resets)

2. **Add a Late ('L'):**
   - Only possible if `k < 2` (don't already have 2 consecutive lates)
   - Transition: `dfs(i + 1, j, k + 1)`
   - We keep `j` the same and increment `k`

3. **Add a Present ('P'):**
   - Always possible
   - Transition: `dfs(i + 1, j, 0)`
   - We keep `j` the same and reset `k` to 0

**Implementation Details:**
```python
ans = 0
if j == 0:
    ans += dfs(i + 1, j + 1, 0)  # Add 'A'
if k < 2:
    ans += dfs(i + 1, j, k + 1)  # Add 'L'
ans += dfs(i + 1, j, 0)          # Add 'P'
return ans % mod
```

The modulo operation `% (10^9 + 7)` is applied at each step to prevent integer overflow.

**Time Complexity:** `O(n × 2 × 3) = O(n)` since we have at most `n` positions, 2 possible values for absences (0 or 1), and 3 possible values for consecutive lates (0, 1, or 2).

**Space Complexity:** `O(n)` for the recursion stack depth and the memoization cache.

The `cache_clear()` at the end is used to free up memory after computation, though it's not strictly necessary for correctness.

## Example Walkthrough

Let's walk through a small example with `n = 3` to illustrate how the solution builds valid attendance records.

Starting with `dfs(0, 0, 0)` - position 0, no absences used, no consecutive lates:

**Position 0 choices:**
1. Add 'A': `dfs(1, 1, 0)` - move to position 1, used 1 absence, reset consecutive lates
2. Add 'L': `dfs(1, 0, 1)` - move to position 1, no absences, 1 consecutive late
3. Add 'P': `dfs(1, 0, 0)` - move to position 1, no absences, reset consecutive lates

Let's trace one path: **Starting with 'A'** → `dfs(1, 1, 0)`:

**Position 1 choices (after 'A'):**
1. ~~Add 'A'~~: Not allowed (j = 1, already used our one absence)
2. Add 'L': `dfs(2, 1, 1)` - creates "AL_"
3. Add 'P': `dfs(2, 1, 0)` - creates "AP_"

Following "AL" path → `dfs(2, 1, 1)`:

**Position 2 choices (after 'AL'):**
1. ~~Add 'A'~~: Not allowed (already used absence)
2. Add 'L': `dfs(3, 1, 2)` - creates "ALL"
3. Add 'P': `dfs(3, 1, 0)` - creates "ALP"

Both paths reach `dfs(3, _, _)` which returns 1 (base case: i >= n).

So from "AL", we get 2 valid records: "ALL" and "ALP".

Similarly, from "AP" → `dfs(2, 1, 0)`:
- Can't add 'A' (used absence)
- Can add 'L' → "APL" ✓
- Can add 'P' → "APP" ✓

This gives us 2 more valid records.

**Complete enumeration for n = 3:**

Starting with 'A' (4 valid records):
- "ALL", "ALP", "APL", "APP"

Starting with 'L' (7 valid records):
- "LAL", "LAP", "LLA", "LLP", "LPA", "LPL", "LPP"
- "LLL" is excluded because it has 3 consecutive lates

Starting with 'P' (8 valid records):
- "PAL", "PAP", "PLA", "PLL", "PLP", "PPA", "PPL", "PPP"

Through the recursive process, each path that doesn't violate our constraints (≤1 absence, <3 consecutive lates) contributes 1 to our count. The memoization ensures we don't recompute states like `dfs(2, 0, 1)` when they are reached by different prefixes.

The final answer for n = 3 is 4 + 7 + 8 = 19 valid attendance records.

## Python Solution

```python
class Solution:
    def checkRecord(self, n: int) -> int:
        """
        Calculate the number of valid attendance records of length n.
        A valid record has:
        - At most 1 absence ('A')
        - No more than 2 consecutive late days ('L')
      
        Args:
            n: Length of the attendance record
          
        Returns:
            Number of valid attendance records modulo 10^9 + 7
        """
        from functools import cache
      
        MOD = 10**9 + 7
      
        @cache
        def dp(day: int, absence_count: int, consecutive_late: int) -> int:
            """
            Dynamic programming function to count valid attendance records.
          
            Args:
                day: Current day index (0-indexed)
                absence_count: Number of absences used so far (0 or 1)
                consecutive_late: Number of consecutive late days ending at current position
              
            Returns:
                Number of valid attendance records from this state
            """
            # Base case: reached the end of the record
            if day >= n:
                return 1
          
            result = 0
          
            # Option 1: Mark as Absent ('A') - only if no absence used yet
            if absence_count == 0:
                result += dp(day + 1, absence_count + 1, 0)
          
            # Option 2: Mark as Late ('L') - only if less than 2 consecutive lates
            if consecutive_late < 2:
                result += dp(day + 1, absence_count, consecutive_late + 1)
          
            # Option 3: Mark as Present ('P') - always allowed, resets consecutive late count
            result += dp(day + 1, absence_count, 0)
          
            return result % MOD
      
        # Calculate the answer starting from day 0 with no absences and no consecutive lates
        answer = dp(0, 0, 0)
      
        # Clear the cache to free memory
        dp.cache_clear()
      
        return answer
```

## Time and Space Complexity

**Time Complexity:** `O(n)`

The function uses memoization with `@cache` decorator. The state space is defined by three parameters:
- `i`: ranges from `0` to `n` (n possible values)
- `j`: can only be `0` or `1` (2 possible values for absence count)
- `k`: can be `0`, `1`, or `2` (3 possible values for consecutive late count)

Total number of unique states = `n × 2 × 3 = 6n`

Each state is computed only once due to memoization, and each state computation involves at most 3 recursive calls with constant time operations. Therefore, the time complexity is `O(6n) = O(n)`.

**Space Complexity:** `O(n)`

The space complexity consists of:
- **Recursion stack depth:** The maximum recursion depth is `n` (when `i` goes from `0` to `n`), contributing `O(n)` space.
- **Cache storage:** The memoization cache stores at most `6n` states, each taking constant space, contributing `O(n)` space.

Overall space complexity is `O(n) + O(n) = O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Forgetting to Apply Modulo at Each Step**

**Pitfall:** A common mistake is only applying the modulo operation at the final return, rather than at each recursive step. This can lead to integer overflow issues for large values of `n`.

**Incorrect approach:**
```python
def dp(day, absence_count, consecutive_late):
    if day >= n:
        return 1
  
    result = 0
    if absence_count == 0:
        result += dp(day + 1, absence_count + 1, 0)
    if consecutive_late < 2:
        result += dp(day + 1, absence_count, consecutive_late + 1)
    result += dp(day + 1, absence_count, 0)
  
    return result  # Missing modulo here!

# Only applying modulo at the end
return dp(0, 0, 0) % MOD  # Too late - overflow may have already occurred
```

**Solution:** Apply modulo at each recursive return to keep numbers manageable:
```python
return result % MOD  # Apply modulo at each step
```

### 2. **Incorrect State Transition for Consecutive Lates**

**Pitfall:** Forgetting to reset the consecutive late count when adding a non-late character ('A' or 'P'). This would incorrectly track consecutive lates across non-late days.

**Incorrect approach:**
```python
# Wrong: Not resetting consecutive_late when adding 'A'
if absence_count == 0:
    result += dp(day + 1, absence_count + 1, consecutive_late)  # Should be 0, not consecutive_late

# Wrong: Not resetting consecutive_late when adding 'P'
result += dp(day + 1, absence_count, consecutive_late)  # Should be 0, not consecutive_late
```

**Solution:** Always reset `consecutive_late` to 0 when adding 'A' or 'P':
```python
if absence_count == 0:
    result += dp(day + 1, absence_count + 1, 0)  # Reset to 0
result += dp(day + 1, absence_count, 0)  # Reset to 0
```

### 3. **Off-by-One Error in Consecutive Late Check**

**Pitfall:** Using the wrong comparison operator for checking consecutive lates. The condition should be `consecutive_late < 2` (allowing 0 or 1 previous consecutive lates before adding another), not `consecutive_late <= 2`.

**Incorrect approach:**
```python
# Wrong: This would allow 3 consecutive lates
if consecutive_late <= 2:  # Incorrect!
    result += dp(day + 1, absence_count, consecutive_late + 1)
```

**Solution:** Use strict less than (`<`) to ensure we never have 3 consecutive lates:
```python
if consecutive_late < 2:  # Correct - only allows 0 or 1 previous consecutive lates
    result += dp(day + 1, absence_count, consecutive_late + 1)
```

### 4. **Not Using Memoization or Using It Incorrectly**

**Pitfall:** Implementing the recursive solution without memoization leads to exponential time complexity due to repeated calculations of the same states.

**Incorrect approach:**
```python
def dp(day, absence_count, consecutive_late):
    # No @cache decorator or manual memoization
    if day >= n:
        return 1
    # ... rest of the logic
```

**Solution:** Use Python's `@cache` decorator or implement manual memoization:
```python
from functools import cache

@cache  # Essential for avoiding repeated calculations
def dp(day, absence_count, consecutive_late):
    # ... implementation
```

