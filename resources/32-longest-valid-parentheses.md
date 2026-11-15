# 32. Longest Valid Parentheses

> Source: https://algo.monster/liteproblems/32

## Problem Statement

Given a string that contains only the characters `'('` and `')'`, you need to find the length of the longest valid (well-formed) parentheses substring.

A valid parentheses substring means that:
- Every opening parenthesis `'('` has a corresponding closing parenthesis `')'`
- The parentheses are properly nested and balanced
- It forms a continuous substring within the original string

For example:
- In the string `"(()"`, the longest valid parentheses substring is `"()"` with length 2
- In the string `")()())"`, the longest valid parentheses substring is `"()()"` with length 4
- In the string `"(())"`, the entire string is valid with length 4

The goal is to return the maximum length among all valid parentheses substrings that can be found in the given string. If no valid parentheses substring exists, return 0.

## Intuition

To find the longest valid parentheses substring, we need to think about what makes parentheses valid and how we can track their lengths efficiently.

The key insight is that valid parentheses have a building pattern - they either extend from previous valid sequences or form new valid pairs. When we encounter a closing parenthesis `')'`, it might complete a valid sequence in one of two ways:

1. It directly pairs with the immediate preceding opening parenthesis `'('`, forming a simple pair like `"()"`. In this case, we get a valid length of 2, but we also need to check if there was a valid sequence right before this pair that we can connect to.

2. It pairs with an opening parenthesis that comes before a already-validated sequence. For instance, in `"((...))"`, when we process the last `')'`, it pairs with the first `'('`, wrapping around an inner valid sequence.

This suggests using [dynamic programming](/problems/dynamic_programming_intro) where `f[i]` represents the length of the longest valid parentheses ending at position `i-1`. 

Why ending at a specific position? Because as we scan through the string, we need to know if we can extend or connect previous valid sequences. By tracking the length of valid parentheses ending at each position, we can:
- Look back to find matching opening parentheses
- Connect adjacent valid sequences
- Build upon previously computed results

The critical observation is that when we find a valid pair, we need to check:
- The length of the valid sequence that this pair completes (if any)
- The length of any valid sequence that comes right before this newly formed sequence

This way, we can concatenate separate valid sequences that become adjacent after forming new pairs, like how `"()"` and `"()"` can be recognized as the continuous `"()()"` with length 4.

**Learn more about [Stack](/problems/stack_intro) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

We implement the solution using [dynamic programming](/problems/dynamic_programming_intro) with a 1D array `f` where `f[i]` represents the length of the longest valid parentheses ending at position `i-1` in the string.

**Initialization:**
- Create an array `f` of size `n+1` initialized with zeros, where `n` is the length of the string
- We use 1-based indexing for easier handling of boundary conditions

**Processing each character:**
We iterate through the string with index `i` starting from 1. For each character at position `i-1` in the original string:

**Case 1: Current character is `'('`**
- No valid parentheses can end with an opening bracket
- `f[i]` remains 0 (already initialized)

**Case 2: Current character is `')'`**
We have two sub-cases to consider:

**Sub-case 2.1:** The previous character `s[i-2]` is `'('`
- We have a direct pair `"()"`
- The length is `f[i-2] + 2`
- `f[i-2]` accounts for any valid sequence ending right before this pair

**Sub-case 2.2:** The previous character `s[i-2]` is `')'`
- We might have a pattern like `"((...))"`
- First, find the position `j` that might contain the matching `'('`
- Calculate `j = i - f[i-1] - 1`, where `f[i-1]` is the length of valid parentheses ending at `i-2`
- If `j > 0` and `s[j-1] = '('`, we found a matching pair
- The total length is: `f[i-1] + 2 + f[j-1]`
  - `f[i-1]`: length of the inner valid sequence
  - `2`: the newly matched pair
  - `f[j-1]`: length of any valid sequence ending right before position `j-1`

**Finding the answer:**
After processing all characters, the maximum value in array `f` gives us the length of the longest valid parentheses substring.

**Time Complexity:** `O(n)` - single pass through the string  
**Space Complexity:** `O(n)` - for the dp array

The beauty of this approach is that it systematically builds upon previously computed results, ensuring we capture all possible valid sequences and their connections.

## Example Walkthrough

Let's walk through the solution with the string `"(())"`:

**Setup:**
- String: `"(())"`
- Length n = 4
- Create dp array `f` of size 5: `[0, 0, 0, 0, 0]`
- Using 1-based indexing where `f[i]` represents the longest valid parentheses ending at position `i-1` in the string

**Step-by-step processing:**

**i = 1, character = '(' at position 0:**
- Opening parenthesis cannot end a valid sequence
- `f[1] = 0`
- Array: `[0, 0, 0, 0, 0]`

**i = 2, character = '(' at position 1:**
- Another opening parenthesis
- `f[2] = 0`
- Array: `[0, 0, 0, 0, 0]`

**i = 3, character = ')' at position 2:**
- Previous character at position 1 is '('
- This forms a direct pair "()"
- `f[3] = f[1] + 2 = 0 + 2 = 2`
- Array: `[0, 0, 0, 2, 0]`

**i = 4, character = ')' at position 3:**
- Previous character at position 2 is ')'
- Need to find matching '(' for this ')'
- Calculate position: `j = i - f[i-1] - 1 = 4 - 2 - 1 = 1`
- Check if `j > 0` (yes, 1 > 0) and `s[j-1] = '('` (yes, s[0] = '(')
- We found a match! The first '(' matches with this last ')'
- `f[4] = f[3] + 2 + f[0] = 2 + 2 + 0 = 4`
  - `f[3] = 2`: length of inner valid sequence "()"
  - `2`: for the outer matching pair
  - `f[0] = 0`: no valid sequence before position 0
- Array: `[0, 0, 0, 2, 4]`

**Result:**
The maximum value in the array is 4, which correctly identifies that the entire string "(())" is valid.

This example demonstrates how the algorithm:
1. Identifies the inner pair "()" first
2. Then recognizes that the outer parentheses wrap around this valid sequence
3. Combines these to get the total length of 4

## Python Solution

```python
class Solution:
    def longestValidParentheses(self, s: str) -> int:
        # Get the length of the input string
        n = len(s)
      
        # dp[i] represents the length of the longest valid parentheses substring
        # ending at position i-1 in the original string (1-indexed for easier calculation)
        dp = [0] * (n + 1)
      
        # Iterate through each character with 1-based indexing
        for i, char in enumerate(s, 1):
            # Only closing parentheses can form valid pairs
            if char == ")":
                # Case 1: Current ')' matches with previous '(' to form "()"
                if i > 1 and s[i - 2] == "(":
                    # Add 2 for the new pair and include any valid substring before it
                    dp[i] = dp[i - 2] + 2
              
                # Case 2: Current ')' might match with a '(' before a valid substring
                else:
                    # Find the position before the valid substring ending at i-1
                    j = i - dp[i - 1] - 1
                  
                    # Check if there's a matching '(' at position j-1
                    if j > 0 and s[j - 1] == "(":
                        # Length = previous valid substring + 2 for new pair + 
                        # any valid substring before the matching '('
                        dp[i] = dp[i - 1] + 2 + dp[j - 1]
      
        # Return the maximum length found
        return max(dp)
```

## Time and Space Complexity

The time complexity is `O(n)`, where `n` is the length of the string. The algorithm iterates through the string once using a single for loop that processes each character exactly once. Inside the loop, all operations (array access, comparisons, and arithmetic operations) are performed in constant time `O(1)`.

The space complexity is `O(n)`, where `n` is the length of the string. The algorithm creates an auxiliary array `f` of size `n + 1` to store the dynamic programming states. This array stores the length of the longest valid parentheses substring ending at each position, requiring linear space proportional to the input string length.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Off-by-One Errors with Index Mapping**
The most common pitfall in this solution is confusion between the 1-based indexing of the DP array and 0-based indexing of the string. This leads to incorrect index calculations when looking for matching parentheses.

**Problem Example:**
```python
# Incorrect: Mixing up indices
j = i - dp[i-1] - 1  # This is correct for 1-based dp
if j > 0 and s[j] == "(":  # Wrong! Should be s[j-1]
    dp[i] = dp[i-1] + 2 + dp[j]  # Wrong! Should be dp[j-1]
```

**Solution:**
Always remember that `dp[i]` corresponds to `s[i-1]`. When calculating position `j` in the dp array, access the string at `s[j-1]`.

### 2. **Missing Boundary Checks**
Failing to verify that indices are within valid bounds before accessing array elements can cause runtime errors in Java and C++, and silently wrong answers in Python.

**Problem Example:**
```python
# Without proper boundary checking
if s[i-2] == "(":  # When i = 1, s[-1] wraps around to the LAST character
    dp[i] = dp[i-2] + 2  # and dp[-1] reads dp[n]
```

For `s = ")("`, the first `)` sees `s[-1] = '('` and sets `dp[1] = 2`, so the function returns `2` even though the string contains no valid pair.

**Solution:**
Always add boundary checks:
```python
if i > 1 and s[i-2] == "(":
    dp[i] = dp[i-2] + 2
```

### 3. **Forgetting to Accumulate Previous Valid Sequences**
A critical mistake is only counting the current matching pair without including adjacent valid sequences.

**Problem Example:**
For string `"()()"`:
```python
# Incorrect: Only counting the current pair
if s[i-2] == "(":
    dp[i] = 2  # Wrong! Misses previous valid sequences
```
This would give `dp = [0, 0, 2, 0, 2]` instead of `[0, 0, 2, 0, 4]`.

**Solution:**
Always add the length of any valid sequence that comes before:
```python
dp[i] = dp[i-2] + 2  # Correctly accumulates previous sequences
```

### 4. **Incorrect Calculation of Matching Position**
When looking for the matching opening parenthesis for a closing one, using the wrong formula to calculate position `j`.

**Problem Example:**
```python
# Incorrect ways to find the matching position
j = i - dp[i-1]      # Missing the -1 for the current ')'
j = i - dp[i] - 1    # Using dp[i] which hasn't been calculated yet
j = dp[i-1] - 1      # Completely wrong logic
```

**Solution:**
The correct formula accounts for:
- Current position `i`
- Length of valid substring ending at `i-1`: `dp[i-1]`
- The current `)` itself: `-1`

Correct calculation: `j = i - dp[i-1] - 1`

### 5. **Not Handling Empty String or Edge Cases**
The code might fail on edge cases like empty strings or strings with only opening/closing parentheses.

**Solution:**
The current implementation handles these well:
- Empty string: `max([0])` returns 0
- All `(`: All dp values remain 0
- All `)`: No valid pairs formed, all dp values remain 0

### 6. **Using 0-based DP Array Without Adjustment**
Some might try to use 0-based indexing for the DP array, which requires careful adjustment of all index calculations.

**Problem Example:**
```python
# Using 0-based dp array without proper adjustments
dp = [0] * n
for i in range(n):
    if s[i] == ")":
        if i > 0 and s[i-1] == "(":
            dp[i] = dp[i-2] + 2  # Wrong! i-2 could be negative
```

**Solution:**
Either stick with 1-based indexing as shown in the original solution, or carefully adjust all index calculations and add appropriate boundary checks for 0-based indexing.

