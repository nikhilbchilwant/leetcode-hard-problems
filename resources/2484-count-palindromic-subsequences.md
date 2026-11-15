# 2484. Count Palindromic Subsequences

> Source: https://algo.monster/liteproblems/2484

## Problem Statement

You are given a string `s` consisting only of digits (0-9). Your task is to count the number of **palindromic subsequences** of length exactly 5 that can be formed from this string.

A **palindromic** string reads the same forwards and backwards. For example, "12321" is palindromic because it's the same when reversed.

A **subsequence** is formed by deleting some (possibly zero) characters from the original string without changing the relative order of the remaining characters. For instance, if `s = "12345"`, then "135" is a subsequence (formed by deleting characters at positions 2 and 4), but "153" is not (because it changes the relative order).

Since the count can be very large, return the result modulo `10^9 + 7`.

**Example clarification:**
- A palindromic subsequence of length 5 has the pattern where the 1st character equals the 5th, and the 2nd character equals the 4th. The middle (3rd) character can be any digit.
- For a string like "123321", one valid palindromic subsequence of length 5 would be "12321" (taking indices 0,1,2,4,5).

The solution approach uses [dynamic programming](/problems/dynamic_programming_intro) to efficiently count these palindromes. It pre-computes:
- `pre[i][j][k]`: the number of pairs `(j, k)` that can be formed using characters before position `i`
- `suf[i][j][k]`: the number of pairs `(j, k)` that can be formed using characters after position `i`

Then, for each position as the middle character, it multiplies the matching pairs from before and after to get the total count of palindromes with that center.

## Intuition

To find palindromic subsequences of length 5, we need to understand their structure. A 5-character palindrome has the form `abcba`, where the first two characters mirror the last two, and `c` is the middle character.

The key insight is that we can fix the middle character and count how many valid palindromes can be formed around it. For each position `i` in the string as the potential middle character, we need to:
1. Count all possible pairs `(a, b)` that can be formed from characters before position `i`
2. Count all possible pairs `(b, a)` that can be formed from characters after position `i` 
3. Multiply these counts together

Why does this work? Because if we have `x` ways to form pair `(a, b)` on the left and `y` ways to form pair `(b, a)` on the right, then we have `x * y` ways to form the palindrome `abcba` with the character at position `i` as the center.

For example, if before position `i` we can form the pair `(1, 2)` in 3 different ways, and after position `i` we can form the pair `(2, 1)` in 2 different ways, then we can create `3 * 2 = 6` palindromes of the form `12c21` where `c` is the character at position `i`.

The challenge is efficiently counting these pairs. We use [dynamic programming](/problems/dynamic_programming_intro):
- `pre[i][j][k]` counts how many times we can pick digit `j` followed by digit `k` from the first `i` characters
- `suf[i][j][k]` counts how many times we can pick digit `j` followed by digit `k` from position `i` onwards

To build `pre[i][j][k]`, we observe that when we add character `s[i]` with value `v`:
- All previous counts remain valid: `pre[i][j][k] = pre[i-1][j][k]`
- For each digit `j` that appeared before position `i`, we can now form a new pair `(j, v)` by combining `j` with the current character `v`

The same logic applies for building `suf` but in reverse order. This preprocessing allows us to answer our counting query in `O(1)` time for each middle position and each pair of digits.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The implementation uses [dynamic programming](/problems/dynamic_programming_intro) with prefix and suffix counting to efficiently enumerate all palindromic subsequences of length 5.

**Data Structures:**
- `pre[i][j][k]`: 3D array storing the count of pairs `(j, k)` that can be formed using digits from positions `0` to `i-1`
- `suf[i][j][k]`: 3D array storing the count of pairs `(j, k)` that can be formed using digits from position `i` to `n-1`
- `c[d]`: Frequency counter for digit `d` seen so far during preprocessing

**Algorithm Steps:**

1. **Initialize arrays**: Create `pre` and `suf` arrays of size `(n+2) × 10 × 10` initialized to 0. The extra padding simplifies boundary handling.

2. **Build prefix counts (`pre`):**
   ```
   For each position i from 1 to n:
     - Copy all counts from pre[i-1] to pre[i]
     - Current digit v = s[i-1]
     - For each digit j (0-9):
         pre[i][j][v] += c[j]  // j appeared c[j] times before, now paired with v
     - Increment c[v] by 1
   ```
   This builds up pairs where the first element comes before the second in the string.

3. **Build suffix counts (`suf`):**
   ```
   For each position i from n down to 1:
     - Copy all counts from suf[i+1] to suf[i]
     - Current digit v = s[i-1]
     - For each digit j (0-9):
         suf[i][j][v] += c[j]  // v at position i paired with j that comes after
     - Increment c[v] by 1
   ```
   This counts pairs in reverse order from the end of the string.

4. **Count palindromes:**
   ```
   For each position i as middle character:
     For each pair (j, k):
       ans += pre[i-1][j][k] * suf[i+1][j][k]
   ```
   For palindrome pattern `jk-i-kj`, we multiply:
   - Number of `(j,k)` pairs before position `i`
   - Number of `(j,k)` pairs after position `i` (note: same order due to palindrome property)

5. **Return result modulo `10^9 + 7`**

**Time Complexity:** `O(n × 100)` = `O(n)` since we iterate through the string three times with constant work (10×10 digit pairs) at each position.

**Space Complexity:** `O(n × 100)` = `O(n)` for the `pre` and `suf` arrays.

The elegance of this solution lies in decomposing the palindrome counting problem into counting matching pairs on both sides of each potential center character, avoiding the need to explicitly enumerate all possible subsequences.

## Example Walkthrough

Let's walk through the solution with `s = "12321"`.

**Step 1: Build prefix counts (`pre`)**

We'll track pairs `(j,k)` that can be formed from left to right.

- Position 1 (digit '1'): 
  - No pairs yet, just record we've seen one '1'
  - `c[1] = 1`

- Position 2 (digit '2'):
  - Can form pair (1,2) once (using the '1' at position 1)
  - `pre[2][1][2] = 1`
  - `c[2] = 1`

- Position 3 (digit '3'):
  - Copy previous pairs: `pre[3][1][2] = 1`
  - Can form new pairs: (1,3) once, (2,3) once
  - `pre[3][1][3] = 1`, `pre[3][2][3] = 1`
  - `c[3] = 1`

- Position 4 (digit '2'):
  - Copy previous pairs
  - Can form new pairs: (1,2) again, (2,2) once, (3,2) once
  - `pre[4][1][2] = 2`, `pre[4][2][2] = 1`, `pre[4][3][2] = 1`
  - Other pairs remain: `pre[4][1][3] = 1`, `pre[4][2][3] = 1`

- Position 5 (digit '1'):
  - Copy and add new pairs with '1' as second element
  - Notable: `pre[5][2][1] = 2` (two '2's before this '1')

**Step 2: Build suffix counts (`suf`)**

Now track pairs from right to left.

- Position 5 (digit '1'): 
  - No pairs yet, `c[1] = 1`

- Position 4 (digit '2'):
  - Can form pair (2,1) once
  - `suf[4][2][1] = 1`

- Position 3 (digit '3'):
  - Copy previous: `suf[3][2][1] = 1`
  - New pairs: (3,2) once, (3,1) once
  - `suf[3][3][2] = 1`, `suf[3][3][1] = 1`

- Position 2 (digit '2'):
  - Notable: `suf[2][2][1] = 2` (can pick '2' at pos 2, then '1' at pos 5)
  - `suf[2][2][3] = 1`, `suf[2][2][2] = 1`

- Position 1 (digit '1'):
  - Multiple pairs possible including `suf[1][1][2] = 2`

**Step 3: Count palindromes**

For each position as middle:

- **Middle at position 1 (digit '1'):**
  - No characters before, so `pre[0][j][k] = 0` for all j,k
  - Result: 0 palindromes

- **Middle at position 2 (digit '2'):**
  - Need matching pairs like (1,2) before and (2,1) after
  - `pre[1][2][1]` = 0 (no (2,1) pairs before position 2)
  - Result: 0 palindromes

- **Middle at position 3 (digit '3'):**
  - Check pair (1,2): `pre[2][1][2] = 1`, `suf[4][1][2] = 0`
  - Check pair (2,1): `pre[2][2][1] = 0`, `suf[4][2][1] = 1`
  - Result: 0 palindromes

- **Middle at position 4 (digit '2'):**
  - Check pair (1,2): `pre[3][1][2] = 1`, `suf[5][1][2] = 0`
  - Check pair (2,1): `pre[3][2][1] = 0`, `suf[5][2][1] = 0`
  - Check pair (1,3): `pre[3][1][3] = 1`, `suf[5][1][3] = 0`
  - Result: 0 palindromes

- **Middle at position 5 (digit '1'):**
  - Check pair (1,2): `pre[4][1][2] = 2`, `suf[6][1][2] = 0`
  - Check pair (2,1): `pre[4][2][1] = 0`, `suf[6][2][1] = 0`
  - Result: 0 palindromes

Wait, this example doesn't yield palindromes because we need longer strings. Let me use `s = "123321"`:

For this string, when position 3 (digit '3') is the middle:
- `pre[2][1][2] = 1` (pair "12" from positions 0,1)
- `suf[4][1][2] = 1` (pair "12" from positions 4,5 → digits '2','1')

But actually we need `suf[4][2][1] = 1` for the palindrome pattern.

The key insight: for palindrome "12321", we need:
- Pair (1,2) before middle
- Pair (2,1) after middle (reverse order!)

So when checking position 3 as middle:
- `pre[2][1][2] × suf[4][2][1] = 1 × 1 = 1`

This gives us one palindrome: selecting indices 0,1,2,3,4 → "12321"

**Final answer: 1 palindromic subsequence of length 5**

## Python Solution

```python
class Solution:
    def countPalindromes(self, s: str) -> int:
        MOD = 10**9 + 7
        n = len(s)
      
        # prefix_pairs[i][j][k] = count of subsequences "jk" ending before position i
        prefix_pairs = [[[0] * 10 for _ in range(10)] for _ in range(n + 2)]
      
        # suffix_pairs[i][j][k] = count of subsequences "jk" starting after position i
        suffix_pairs = [[[0] * 10 for _ in range(10)] for _ in range(n + 2)]
      
        # Convert string to list of integers for easier processing
        digits = [int(char) for char in s]
      
        # Build prefix pairs: count all possible two-digit subsequences before each position
        digit_count = [0] * 10  # Count of each digit seen so far
        for pos in range(1, n + 1):
            current_digit = digits[pos - 1]
          
            # Copy previous counts
            for first_digit in range(10):
                for second_digit in range(10):
                    prefix_pairs[pos][first_digit][second_digit] = prefix_pairs[pos - 1][first_digit][second_digit]
          
            # Add new pairs formed with current digit as the second digit
            for first_digit in range(10):
                prefix_pairs[pos][first_digit][current_digit] += digit_count[first_digit]
          
            # Update count of current digit
            digit_count[current_digit] += 1
      
        # Build suffix pairs: count all possible two-digit subsequences after each position
        digit_count = [0] * 10  # Reset count for suffix processing
        for pos in range(n, 0, -1):
            current_digit = digits[pos - 1]
          
            # Copy previous counts
            for first_digit in range(10):
                for second_digit in range(10):
                    suffix_pairs[pos][first_digit][second_digit] = suffix_pairs[pos + 1][first_digit][second_digit]
          
            # Add new pairs formed with current digit as the first digit
            for second_digit in range(10):
                suffix_pairs[pos][current_digit][second_digit] += digit_count[second_digit]
          
            # Update count of current digit
            digit_count[current_digit] += 1
      
        # Count palindromes: for each position as middle character,
        # multiply matching prefix and suffix pairs
        total_palindromes = 0
        for middle_pos in range(1, n + 1):
            for first_digit in range(10):
                for second_digit in range(10):
                    # For palindrome "ab_ba", we need prefix "ab" and suffix "ba"
                    # Note: suffix uses [second_digit][first_digit] for mirror pattern
                    palindrome_count = prefix_pairs[middle_pos - 1][first_digit][second_digit] * \
                                     suffix_pairs[middle_pos + 1][second_digit][first_digit]
                    total_palindromes = (total_palindromes + palindrome_count) % MOD
      
        return total_palindromes
```

## Time and Space Complexity

The time complexity is `O(100 × n)`, which can be broken down as follows:
- The first loop iterates through `n` positions, and for each position, it copies a 10×10 matrix from the previous position and updates 10 entries, taking `O(100)` time per iteration, resulting in `O(100 × n)`.
- The second loop similarly iterates through `n` positions in reverse, performing the same `O(100)` operations per iteration, resulting in `O(100 × n)`.
- The final loop iterates through `n` positions, and for each position, it performs calculations for all 10×10 pairs of digits `(j, k)`, taking `O(100)` time per position, resulting in `O(100 × n)`.
- Total: `O(100 × n) + O(100 × n) + O(100 × n) = O(100 × n)`.

The space complexity is `O(100 × n)`, which comes from:
- The `pre` array: `(n + 2) × 10 × 10 = O(100 × n)` space.
- The `suf` array: `(n + 2) × 10 × 10 = O(100 × n)` space.
- Other variables (`t`, `c`) use `O(n)` and `O(10)` space respectively, which are negligible.
- Total: `O(100 × n) + O(100 × n) = O(100 × n)`.

Where `n` is the length of the string `s`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Index Mapping for Suffix Pairs**

The most critical pitfall in this problem is misunderstanding how to use the suffix pairs when counting palindromes. When forming a palindrome of pattern `ab-c-ba`, the suffix part needs to be `ba` (reversed order), but the suffix array stores pairs in their original left-to-right order.

**Incorrect approach:**
```python
# Wrong: Using the same indices as prefix
palindrome_count = prefix_pairs[middle_pos - 1][first_digit][second_digit] * \
                   suffix_pairs[middle_pos + 1][first_digit][second_digit]
```

**Correct approach:**
```python
# Correct: Swap indices for suffix to account for palindrome symmetry
palindrome_count = prefix_pairs[middle_pos - 1][first_digit][second_digit] * \
                   suffix_pairs[middle_pos + 1][second_digit][first_digit]
```

### 2. **Integer Overflow Without Proper Modulo Operations**

Since we're multiplying large numbers, intermediate results can exceed integer limits. Failing to apply modulo at each step can cause overflow.

**Incorrect approach:**
```python
# Wrong: Accumulating without modulo can cause overflow
total_palindromes += palindrome_count
# Apply modulo only at the end
return total_palindromes % MOD
```

**Correct approach:**
```python
# Correct: Apply modulo after each addition
total_palindromes = (total_palindromes + palindrome_count) % MOD
```

### 3. **Off-by-One Errors in Array Indexing**

The padding strategy (using n+2 size arrays) can lead to confusion about proper index usage.

**Common mistakes:**
- Using `prefix_pairs[middle_pos]` instead of `prefix_pairs[middle_pos - 1]` (includes the middle character in prefix)
- Using `suffix_pairs[middle_pos]` instead of `suffix_pairs[middle_pos + 1]` (includes the middle character in suffix)

**Solution:** Always remember that for position `i` as middle:
- Prefix should contain positions `[0, i-1]`
- Suffix should contain positions `[i+1, n-1]`

### 4. **Forgetting to Reset Counter Arrays**

When building suffix pairs, forgetting to reset the digit counter array will lead to incorrect counts.

**Incorrect approach:**
```python
digit_count = [0] * 10
# Build prefix pairs...
# Forgot to reset digit_count before building suffix pairs
for pos in range(n, 0, -1):
    # This will use leftover counts from prefix building
```

**Correct approach:**
```python
digit_count = [0] * 10  # For prefix
# Build prefix pairs...
digit_count = [0] * 10  # Reset for suffix
# Build suffix pairs...
```

### 5. **Misunderstanding Subsequence vs Substring**

A common conceptual error is treating this as a substring problem, where elements must be contiguous. Remember that subsequences allow skipping elements while maintaining relative order.

**Key insight:** The DP approach correctly handles this by accumulating all possible pairs regardless of their distance in the original string.

