# 2478. Number of Beautiful Partitions

> Source: https://algo.monster/liteproblems/2478

## Problem Statement

You are given a string `s` containing only digits from '1' to '9', along with two integers `k` and `minLength`. Your task is to find the number of ways to partition the string into exactly `k` non-overlapping substrings where each partition is considered "beautiful".

A partition is **beautiful** if it satisfies all of these conditions:
- The string is divided into exactly `k` substrings
- Each substring has a length of at least `minLength` characters
- Each substring must start with a prime digit ('2', '3', '5', or '7')
- Each substring must end with a non-prime digit ('1', '4', '6', '8', or '9')

For example, if we have a substring "241", it would be valid because:
- It starts with '2' (prime)
- It ends with '1' (non-prime)

The problem asks you to count all possible beautiful partitions of the given string `s`. Since the answer can be very large, return the result modulo `10^9 + 7`.

Key points to note:
- The substrings must be contiguous (you cannot rearrange characters)
- The substrings cannot overlap
- All `k` substrings must satisfy the prime/non-prime conditions
- Each substring must have at least `minLength` characters

If it's impossible to create a beautiful partition (for instance, if the first character is non-prime or the last character is prime), the answer would be 0.

## Intuition

To solve this problem, we need to think about how to count valid ways to partition the string. The key insight is that this is a classic [dynamic programming](/problems/dynamic_programming_intro) problem where we build up solutions for smaller subproblems.

Let's think step by step. We need to partition string `s` into exactly `k` parts, where each part satisfies certain conditions. At any position `i` in the string, we need to decide: "Can this position be the end of a partition?"

For position `i` to be a valid ending position of a partition, it must satisfy:
- The character at position `i` must be non-prime (since partitions end with non-prime digits)
- Either `i` is the last position in the string, OR the next character at position `i+1` must be prime (since the next partition must start with a prime digit)

This naturally leads us to define `f[i][j]` as the number of ways to partition the first `i` characters into exactly `j` valid partitions.

For each valid ending position `i` and partition count `j`, we need to consider where the previous partition ended. The previous partition could have ended at any position `t` where:
- `t` is at least `minLength` characters before `i` (since each partition needs at least `minLength` characters)
- There are `j-1` valid partitions up to position `t`

This gives us the recurrence relation: `f[i][j] = sum of all f[t][j-1]` where `t` ranges from `0` to `i-minLength`.

However, computing this sum repeatedly would be inefficient. Here's where the optimization comes in - we can use a prefix sum array `g[i][j]` to store the cumulative sum of `f[0][j]` through `f[i][j]`. This way, instead of summing up multiple values each time, we can directly access `g[i-minLength][j-1]` to get the sum we need.

The base case is `f[0][0] = 1` (one way to partition zero characters into zero parts), and we build up from there until we reach `f[n][k]`, which gives us the final answer.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The implementation uses [dynamic programming](/problems/dynamic_programming_intro) with prefix sum optimization. Let's walk through the solution step by step:

**Initial Validation:**
First, we check if a beautiful partition is even possible. If the first character is not prime or the last character is prime, we immediately return 0 since no valid partition exists.

**State Definition:**
We define two 2D arrays:
- `f[i][j]`: Number of ways to partition the first `i` characters into exactly `j` beautiful partitions
- `g[i][j]`: Prefix sum array where `g[i][j] = f[0][j] + f[1][j] + ... + f[i][j]`

Both arrays have dimensions `(n+1) × (k+1)` where `n` is the length of string `s`.

**Base Case:**
Initialize `f[0][0] = g[0][0] = 1`, representing one way to partition zero characters into zero parts.

**State Transition:**
For each position `i` from 1 to `n`, we:

1. **Check if position `i` can be a partition endpoint:**
   - Position `i` must have at least `minLength` characters (`i >= minLength`)
   - Character at position `i-1` (0-indexed) must be non-prime
   - Either `i = n` (last position) OR character at position `i` must be prime (start of next partition)

2. **If valid, calculate `f[i][j]` for all `j` from 1 to `k`:**
   ```
   f[i][j] = g[i-minLength][j-1]
   ```
   This formula works because:
   - We need the sum of all `f[t][j-1]` where `t` ranges from 0 to `i-minLength`
   - The prefix sum `g[i-minLength][j-1]` gives us exactly this sum

3. **Update the prefix sum array:**
   ```
   g[i][j] = (g[i-1][j] + f[i][j]) % mod
   ```
   This maintains the cumulative sum for future calculations.

**Time Complexity:** 
`O(n × k)` where `n` is the length of the string and `k` is the number of partitions.

**Space Complexity:** 
`O(n × k)` for the two 2D arrays.

**Final Answer:**
Return `f[n][k]`, which represents the number of ways to partition all `n` characters into exactly `k` beautiful partitions.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Example:** `s = "23542185131"`, `k = 3`, `minLength = 2`

**Step 1: Initial Validation**
- First character '2' is prime ✓
- Last character '1' is non-prime ✓
- Beautiful partition is possible!

**Step 2: Identify Valid Partition Endpoints**
We need to find positions where a partition can end:
- Character at that position must be non-prime
- Next character (if exists) must be prime

Looking at our string with 0-based indexing:
```
Index:    0 1 2 3 4 5 6 7 8 9 10
String:   2 3 5 4 2 1 8 5 1 3 1
Type:     P P P N P N N P N P N
```
(P = Prime, N = Non-prime)

Valid endpoints (1-indexed for DP array):
- Position 4: s[3]='4' is non-prime, s[4]='2' is prime ✓
- Position 6: s[5]='1' is non-prime, s[6]='8' is non-prime ✗
- Position 7: s[6]='8' is non-prime, s[7]='5' is prime ✓
- Position 9: s[8]='1' is non-prime, s[9]='3' is prime ✓
- Position 11: s[10]='1' is non-prime, end of string ✓

**Step 3: Build DP Table**

Initialize:
- `f[0][0] = 1` (base case)
- `g[0][0] = 1` (prefix sum)

For each valid endpoint `i` and partition count `j`:

**At position 4 (can form 1st partition "2354"):**
- Length = 4 ≥ minLength(2) ✓
- `f[4][1] = g[4-2][0] = g[2][0] = 1`
- This means: 1 way to partition first 4 chars into 1 partition

**At position 7 (can form 2nd partition):**
- For j=1: `f[7][1] = g[7-2][0] = g[5][0] = 1` (partition: "2354218")
- For j=2: `f[7][2] = g[7-2][1] = g[5][1] = 1` (partitions: "2354" + "218")

**At position 9 (can form 3rd partition):**
- For j=1: `f[9][1] = g[9-2][0] = g[7][0] = 1` 
- For j=2: `f[9][2] = g[9-2][1] = g[7][1] = 2` (either "2354" + "21851" or "2354218" + "51")
- For j=3: `f[9][3] = g[9-2][2] = g[7][2] = 1` (partitions: "2354" + "218" + "51")

**At position 11 (final position):**
- For j=1: `f[11][1] = g[11-2][0] = g[9][0] = 1`
- For j=2: `f[11][2] = g[11-2][1] = g[9][1] = 3`
- For j=3: `f[11][3] = g[11-2][2] = g[9][2] = 3`

**Step 4: Final Answer**
`f[11][3] = 3`

The three valid partitions are:
1. "2354" + "218" + "5131"
2. "2354" + "21851" + "31"  
3. "2354218" + "51" + "31"

Each partition satisfies:
- Starts with prime digit
- Ends with non-prime digit
- Has at least minLength characters
- Total of k=3 partitions

The answer is **3** (modulo 10^9 + 7).

## Python Solution

```python
class Solution:
    def beautifulPartitions(self, s: str, k: int, minLength: int) -> int:
        # Define prime digits for checking partition boundaries
        PRIME_DIGITS = '2357'
        MOD = 10**9 + 7
      
        # Early validation: first char must be prime, last char must be non-prime
        if s[0] not in PRIME_DIGITS or s[-1] in PRIME_DIGITS:
            return 0
      
        n = len(s)
      
        # dp[i][j] = number of ways to partition s[0:i] into j beautiful partitions
        # where position i is a valid partition end
        dp = [[0] * (k + 1) for _ in range(n + 1)]
      
        # prefix_sum[i][j] = cumulative sum of dp values up to position i with j partitions
        # Used for optimization to avoid recalculating sums
        prefix_sum = [[0] * (k + 1) for _ in range(n + 1)]
      
        # Base case: empty string with 0 partitions has 1 way
        dp[0][0] = prefix_sum[0][0] = 1
      
        # Iterate through each position in the string
        for i in range(1, n + 1):
            current_char = s[i - 1]
          
            # Check if position i can be a valid partition endpoint:
            # 1. Must have at least minLength characters
            # 2. Current character must be non-prime
            # 3. Either at the end of string OR next character is prime
            is_valid_end = (i >= minLength and 
                           current_char not in PRIME_DIGITS and 
                           (i == n or s[i] in PRIME_DIGITS))
          
            if is_valid_end:
                # For each number of partitions j
                for j in range(1, k + 1):
                    # Sum all valid ways to form j-1 partitions ending at least
                    # minLength positions before current position
                    dp[i][j] = prefix_sum[i - minLength][j - 1]
          
            # Update prefix sums for current position
            for j in range(k + 1):
                prefix_sum[i][j] = (prefix_sum[i - 1][j] + dp[i][j]) % MOD
      
        # Return number of ways to partition entire string into exactly k parts
        return dp[n][k]
```

## Time and Space Complexity

**Time Complexity:** `O(n × k)`

The algorithm uses nested loops where:
- The outer loop iterates through each character in string `s`, running `n` times
- For each position `i`, we iterate through all possible partition counts from `1` to `k`
- Both the update of `f[i][j]` and `g[i][j]` involve constant-time operations

Therefore, the total time complexity is `O(n × k)`, where `n` is the length of string `s` and `k` is the number of partitions.

**Space Complexity:** `O(n × k)`

The algorithm allocates:
- A 2D array `f` of size `(n + 1) × (k + 1)` to store the number of valid partitions ending at position `i` with exactly `j` partitions
- A 2D array `g` of size `(n + 1) × (k + 1)` to store the prefix sum of valid partitions
- A few constant space variables like `primes`, `mod`, and `n`

The dominant space usage comes from the two 2D arrays, resulting in a space complexity of `O(n × k)`, where `n` is the length of string `s` and `k` is the number of partitions.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Off-by-One Errors in Index Validation**

One of the most common mistakes is incorrectly checking whether a position can be a valid partition endpoint. Developers often confuse 0-based and 1-based indexing when validating partition boundaries.

**Pitfall Example:**
```python
# WRONG: Checking the wrong character for the next position
is_valid_end = (i >= minLength and 
               s[i] not in PRIME_DIGITS and  # Wrong! Should be s[i-1]
               (i == n or s[i+1] in PRIME_DIGITS))  # Wrong! Should be s[i]
```

**Solution:**
Remember that when using 1-based DP indexing (where `i` ranges from 1 to n), the corresponding character in the 0-based string is `s[i-1]`. The next character would be `s[i]`, not `s[i+1]`.

### 2. **Forgetting to Handle Edge Cases in Validation**

Another frequent error is not properly handling the boundary condition when checking if the next character is prime.

**Pitfall Example:**
```python
# WRONG: May cause index out of bounds
is_valid_end = (i >= minLength and 
               s[i-1] not in PRIME_DIGITS and 
               s[i] in PRIME_DIGITS)  # Fails when i == n
```

**Solution:**
Always check if you're at the string's end before accessing the next character:
```python
is_valid_end = (i >= minLength and 
               s[i-1] not in PRIME_DIGITS and 
               (i == n or s[i] in PRIME_DIGITS))
```

### 3. **Incorrect Prefix Sum Range**

A subtle but critical mistake is using the wrong range when calculating the sum of previous DP states.

**Pitfall Example:**
```python
# WRONG: Including positions that are too close
dp[i][j] = prefix_sum[i - 1][j - 1]  # Should ensure minLength gap
```

**Solution:**
The correct approach ensures that the previous partition ended at least `minLength` positions before:
```python
dp[i][j] = prefix_sum[i - minLength][j - 1]
```
This guarantees that the current partition has at least `minLength` characters.

### 4. **Not Applying Modulo Consistently**

Forgetting to apply the modulo operation can lead to integer overflow in languages with fixed integer sizes, or incorrect results even in Python.

**Pitfall Example:**
```python
# WRONG: Missing modulo operation
prefix_sum[i][j] = prefix_sum[i - 1][j] + dp[i][j]
```

**Solution:**
Always apply modulo when updating cumulative values:
```python
prefix_sum[i][j] = (prefix_sum[i - 1][j] + dp[i][j]) % MOD
```

### 5. **Misunderstanding the Problem Constraints**

Developers sometimes misinterpret what constitutes a "beautiful" partition, particularly regarding which digits are prime.

**Pitfall Example:**
```python
# WRONG: Including '1' as prime or forgetting '2'
PRIME_DIGITS = '357'  # Missing '2'
# or
PRIME_DIGITS = '12357'  # '1' is not prime
```

**Solution:**
Carefully verify the prime digits according to mathematical definition:
```python
PRIME_DIGITS = '2357'  # Exactly these four single-digit primes
```

