# 639. Decode Ways II

> Source: https://algo.monster/liteproblems/639

## Problem Statement

This problem asks you to count the number of ways to decode a string that contains digits and wildcard characters (`*`).

The encoding rules are:
- Letters A-Z are mapped to numbers 1-26 (A="1", B="2", ..., Z="26")
- To decode, you group consecutive digits and map them back to letters
- Valid groupings must form numbers between 1-26
- Leading zeros are not allowed (e.g., "06" is invalid)

The special wildcard character `*` can represent any digit from 1-9 (not 0).

For example, with input `"1*"`:
- The `*` could be any digit 1-9
- This gives possibilities: "11", "12", "13", ..., "19"
- Each can be decoded as either:
  - Two single digits (e.g., "11" → "AA")
  - One double digit if valid (e.g., "11" → "K")

The solution uses [dynamic programming](/problems/dynamic_programming_intro) with three variables:
- `a`: number of ways to decode up to position i-2
- `b`: number of ways to decode up to position i-1  
- `c`: number of ways to decode up to current position i

For each position, it considers:
1. **Single digit decoding**: Current character alone
   - If `*`: represents 9 possibilities (1-9)
   - If digit 1-9: one way
   - If '0': cannot be decoded alone

2. **Two digit decoding**: Previous and current character together
   - If both are `*`: 15 valid combinations (11-19, 21-26)
   - If previous is `*` and current is digit: depends on digit value
   - If previous is digit and current is `*`: depends on previous digit
   - If both are digits: check if they form a valid number ≤ 26

The answer is returned modulo `10^9 + 7` to handle large numbers.

## Intuition

The key insight is recognizing this as a [dynamic programming](/problems/dynamic_programming_intro) problem similar to the classic "Decode Ways" problem, but with the added complexity of wildcards.

Think of it like building a path from left to right through the string. At each position, you have choices about how to interpret the digits - either take one digit as a single letter, or combine it with the previous digit to form a two-digit letter.

The wildcard `*` essentially creates multiple parallel universes - each possible digit value (1-9) that `*` could represent. Instead of tracking each possibility separately, we can count them mathematically.

Why [dynamic programming](/problems/dynamic_programming_intro)? Because the number of ways to decode up to position `i` depends only on:
- The number of ways to decode up to position `i-1` (if we take current digit alone)
- The number of ways to decode up to position `i-2` (if we take last two digits together)

This creates overlapping subproblems with optimal substructure.

The space optimization comes from observing that we only need the previous two states at any point. Instead of maintaining a full DP array, we use three variables (`a`, `b`, `c`) that rotate through positions, similar to how Fibonacci numbers can be calculated with just two variables.

For handling wildcards:
- A single `*` represents 9 choices (digits 1-9)
- Two `*` together (`**`) can form 15 valid two-digit codes: 11-19 (9 choices) and 21-26 (6 choices)
- When `*` is paired with a regular digit, we count based on what valid combinations are possible

The modulo operation is applied at each step to prevent integer overflow, which is a common technique in counting problems with large results.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution uses a space-optimized [dynamic programming](/problems/dynamic_programming_intro) approach with three variables representing a sliding window of states.

**Variable Setup:**
- `a`: represents `dp[i-2]` - ways to decode up to 2 positions back
- `b`: represents `dp[i-1]` - ways to decode up to 1 position back  
- `c`: represents `dp[i]` - ways to decode up to current position
- Initialize `b = 1` as base case (empty string has 1 way to decode)

**Main Loop:** Process each character from index 0 to n-1:

**Step 1: Single Digit Decoding**
Check if current character can be decoded as a single digit:
- If `s[i-1] == '*'`: It can be any digit 1-9, so `c = 9 * b`
- If `s[i-1]` is '1'-'9': Valid single digit, so `c = b`
- If `s[i-1] == '0'`: Cannot be decoded alone, so `c = 0`

**Step 2: Two Digit Decoding** (only if `i > 1`)
Add ways to decode last two characters as a pair:

- **Case 1:** Both are wildcards (`**`)
  - Can form: 11-19 (9 numbers) and 21-26 (6 numbers)
  - Add `15 * a` to `c`

- **Case 2:** First is wildcard, second is digit (`*d`)
  - If digit > '6': Only '1' works as first digit → add `a`
  - If digit ≤ '6': Both '1' and '2' work → add `2 * a`

- **Case 3:** First is digit, second is wildcard (`d*`)
  - If first digit is '1': Can form 11-19 → add `9 * a`
  - If first digit is '2': Can form 21-26 → add `6 * a`
  - Otherwise: Invalid combination

- **Case 4:** Both are regular digits
  - Check if they form a valid number between 10-26
  - Calculate: `(s[i-2] - '0') * 10 + (s[i-1] - '0')`
  - If valid and first digit isn't '0': add `a`

**Step 3: State Update**
Shift variables for next iteration: `a, b = b, c`

**Modulo Operation:** Apply `% (10^9 + 7)` after each addition to prevent overflow.

The final answer is stored in `c` after processing all characters.

## Example Walkthrough

Let's walk through the solution with input `s = "1*"`:

**Initial Setup:**
- `a = 0` (dp[i-2])
- `b = 1` (dp[i-1], base case)
- `c = 0` (dp[i])

**Iteration 1 (i = 1, character = '1'):**

*Step 1: Single digit decoding*
- Current character is '1' (valid digit 1-9)
- `c = b = 1` (one way to decode '1' as 'A')

*Step 2: Two digit decoding*
- Skip (i = 1, need i > 1)

*Step 3: Update states*
- `a = b = 1`
- `b = c = 1`

**Iteration 2 (i = 2, character = '*'):**

*Step 1: Single digit decoding*
- Current character is '*' (can be 1-9)
- `c = 9 * b = 9 * 1 = 9`
- These represent: '1' followed by '1' → "AA", '1' followed by '2' → "AB", ..., '1' followed by '9' → "AI"

*Step 2: Two digit decoding (i > 1)*
- Previous character is '1', current is '*'
- This is Case 3: first is digit '1', second is wildcard
- '1*' can form: 11, 12, 13, 14, 15, 16, 17, 18, 19 (all valid)
- Add `9 * a = 9 * 1 = 9` to c
- `c = 9 + 9 = 18`
- These represent: "11" → 'K', "12" → 'L', ..., "19" → 'S'

*Step 3: Update states*
- `a = b = 1`
- `b = c = 18`

**Final Result:** `c = 18`

The 18 ways break down as:
- 9 ways treating '*' as single digit (1-9): "1,1" through "1,9"
- 9 ways treating '1*' as two-digit number: "11" through "19"

## Python Solution

```python
class Solution:
    def numDecodings(self, s: str) -> int:
        MOD = 10**9 + 7
        n = len(s)
      
        # Dynamic programming with space optimization
        # prev_prev: number of ways to decode up to position i-2
        # prev: number of ways to decode up to position i-1  
        # curr: number of ways to decode up to position i
        prev_prev, prev, curr = 0, 1, 0
      
        for i in range(1, n + 1):
            # Single digit decoding (using character at position i-1)
            if s[i - 1] == "*":
                # '*' can represent any digit from 1 to 9
                curr = (9 * prev) % MOD
            elif s[i - 1] != "0":
                # Non-zero digit can be decoded as itself
                curr = prev
            else:
                # '0' cannot be decoded as a single digit
                curr = 0
          
            # Two-digit decoding (using characters at positions i-2 and i-1)
            if i > 1:
                if s[i - 2] == "*" and s[i - 1] == "*":
                    # "**" can represent: 11-19 (9 ways) and 21-26 (6 ways)
                    curr = (curr + 15 * prev_prev) % MOD
                elif s[i - 2] == "*":
                    # "*X" where X is a digit
                    if s[i - 1] > "6":
                        # Only "1X" is valid (X > 6), so 1 way
                        curr = (curr + prev_prev) % MOD
                    else:
                        # Both "1X" and "2X" are valid (X <= 6), so 2 ways
                        curr = (curr + 2 * prev_prev) % MOD
                elif s[i - 1] == "*":
                    # "X*" where X is a digit
                    if s[i - 2] == "1":
                        # "1*" can represent 11-19, so 9 ways
                        curr = (curr + 9 * prev_prev) % MOD
                    elif s[i - 2] == "2":
                        # "2*" can represent 21-26, so 6 ways
                        curr = (curr + 6 * prev_prev) % MOD
                else:
                    # Both are regular digits
                    # Check if they form a valid two-digit number (10-26)
                    if s[i - 2] != "0":
                        two_digit_value = int(s[i - 2]) * 10 + int(s[i - 1])
                        if two_digit_value <= 26:
                            curr = (curr + prev_prev) % MOD
          
            # Shift values for next iteration
            prev_prev, prev = prev, curr
      
        return curr
```

## Time and Space Complexity

**Time Complexity:** `O(n)` where `n` is the length of the input string `s`. The algorithm iterates through the string once with a single for loop from index 1 to n. Within each iteration, all operations (checking characters, arithmetic operations, and modulo operations) are performed in constant time `O(1)`. Therefore, the overall time complexity is linear.

**Space Complexity:** `O(1)`. The algorithm uses only three variables (`a`, `b`, `c`) to store the dynamic programming states, regardless of the input size. These variables represent `dp[i-2]`, `dp[i-1]`, and `dp[i]` respectively, implementing a space-optimized version of dynamic programming. Additional variables like `mod` and `n` also require constant space. No additional data structures that scale with input size are used, making the space complexity constant.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Wildcard Counting for Two-Digit Combinations**

A frequent mistake is miscalculating the number of valid two-digit combinations when wildcards are involved. Developers often forget that:
- `"**"` represents exactly 15 valid combinations (11-19 and 21-26), not 9 × 9 = 81
- `"2*"` can only form 21-26 (6 combinations), not 21-29
- `"*X"` where X > 6 can only use '1' as the first digit, not both '1' and '2'

**Solution:** Carefully enumerate all valid combinations:
```python
if s[i-2] == '*' and s[i-1] == '*':
    # Valid: 11-19 (9) + 21-26 (6) = 15 total
    curr = (curr + 15 * prev_prev) % MOD
```

### 2. **Forgetting to Apply Modulo Operation Consistently**

The problem requires returning the result modulo 10^9 + 7. A common error is applying modulo only at the end or forgetting it in some calculations, which can cause integer overflow in languages with fixed integer sizes.

**Solution:** Apply modulo after every arithmetic operation that could produce a large number:
```python
curr = (9 * prev) % MOD  # Apply modulo immediately
curr = (curr + 15 * prev_prev) % MOD  # Apply again after addition
```

### 3. **Off-by-One Indexing Errors**

The dynamic programming iteration uses 1-based indexing (i from 1 to n), but string indexing is 0-based. This mismatch often leads to accessing wrong characters:
- Using `s[i]` instead of `s[i-1]` for current character
- Using `s[i-1]` instead of `s[i-2]` for previous character

**Solution:** Be consistent with index mapping:
```python
for i in range(1, n + 1):
    # Current character is at s[i-1]
    # Previous character is at s[i-2]
    if s[i-1] == '*':  # Correct: i-1 for current
        # ...
    if i > 1 and s[i-2] == '*':  # Correct: i-2 for previous
        # ...
```

### 4. **Incorrect Base Case Initialization**

Setting wrong initial values for the DP variables can propagate errors throughout the computation. Common mistakes include:
- Initializing `prev` (dp[0]) as 0 instead of 1
- Not properly handling the first character

**Solution:** Initialize correctly:
```python
prev_prev, prev, curr = 0, 1, 0
# prev = 1 represents the empty string having one way to decode
```

### 5. **Missing Edge Case: Leading Zero**

Failing to handle strings that start with '0' or have '0' after '*' can lead to incorrect counts. Remember that '0' cannot be decoded as a single digit and "00", "30", etc., are invalid two-digit codes.

**Solution:** Explicitly check for '0':
```python
if s[i-1] == '0':
    curr = 0  # '0' cannot be decoded alone
# For two-digit decoding:
if s[i-2] != '0':  # Ensure first digit isn't '0'
    two_digit_value = int(s[i-2]) * 10 + int(s[i-1])
    if 10 <= two_digit_value <= 26:  # Valid range check
        curr = (curr + prev_prev) % MOD
```

