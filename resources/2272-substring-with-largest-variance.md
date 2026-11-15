# 2272. Substring With Largest Variance

> Source: https://algo.monster/liteproblems/2272

## Problem Statement

Given a string `s` consisting of only lowercase English letters, you need to find the **largest variance** among all possible substrings of `s`.

The **variance** of a string is defined as the largest difference between the number of occurrences of any two characters present in the string. These two characters may or may not be the same.

A **substring** is a contiguous sequence of characters within a string.

For example, if a substring contains character 'a' appearing 5 times and character 'b' appearing 2 times, and these are the maximum and minimum occurrences among all characters in that substring, then the variance would be `5 - 2 = 3`.

Your task is to find the maximum variance value across all possible substrings of the given string `s`.

The solution uses [dynamic programming](/problems/dynamic_programming_intro) with character enumeration. Since there are only 26 lowercase letters, we can enumerate all pairs of characters `(a, b)` where `a` represents the character we want to maximize and `b` represents the character we want to minimize in our count difference. For each pair, we use dynamic programming to track:
- `f[0]`: the count of consecutive character `a` occurrences ending at the current position
- `f[1]`: the maximum variance for substrings ending at the current position that contain both characters `a` and `b`

## Intuition

The key insight is to transform this problem into something simpler. Instead of trying to track all character frequencies in every possible substring, we can focus on just two characters at a time.

Think about it this way: the variance is the difference between the maximum and minimum character frequencies. In any substring, there will be exactly one character with the highest frequency and one with the lowest frequency. So we can enumerate all possible pairs of characters `(a, b)` where `a` is the character we want to maximize and `b` is the character we want to minimize.

For each pair `(a, b)`, we want to find the substring where the difference `count(a) - count(b)` is maximized. This transforms our problem into a classic maximum subarray sum problem, similar to Kadane's algorithm.

We can assign values to characters:
- Character `a` gets value `+1`
- Character `b` gets value `-1`  
- All other characters get value `0`

Now we need to find the maximum sum of any subarray, with the constraint that the subarray must contain at least one `b` (otherwise the variance wouldn't be valid).

This is where the [dynamic programming](/problems/dynamic_programming_intro) comes in. We maintain two states:
- `f[0]`: tracks consecutive `a`s without any `b` yet
- `f[1]`: tracks the maximum variance when we've seen at least one `b`

When we encounter an `a`, both values increase by 1. When we encounter a `b`, we have a choice: either extend the current sequence (decreasing by 1) or start fresh from the previous `f[0]` value (which represents starting a new sequence right after some `a`s). This ensures we always have at least one `b` in our considered substring.

By trying all possible character pairs and tracking the maximum variance for each, we guarantee finding the overall maximum variance across all substrings.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution uses enumeration combined with [dynamic programming](/problems/dynamic_programming_intro) to find the maximum variance.

**Step 1: Enumerate Character Pairs**

We use `permutations(ascii_lowercase, 2)` to generate all possible pairs of characters `(a, b)`. Since we want different characters for meaningful variance, we skip cases where `a == b`.

**Step 2: [Dynamic Programming](/problems/dynamic_programming_intro) States**

For each character pair `(a, b)`, we maintain two states:
- `f[0]`: Count of consecutive character `a` occurrences (without any `b`)
- `f[1]`: Maximum variance for substrings ending at current position that contain both `a` and `b`

Initially, `f[0] = 0` and `f[1] = -inf` (negative infinity ensures we only update the answer when we have a valid substring containing both characters).

**Step 3: Process Each Character**

For each character `c` in string `s`, we update our states based on three cases:

1. **If `c == a`**: Both states increase by 1
   - `f[0] = f[0] + 1` (one more `a` in the sequence)
   - `f[1] = f[1] + 1` (variance increases by 1)

2. **If `c == b`**: We need to decrease the variance
   - `f[1] = max(f[1] - 1, f[0] - 1)`
     - Either extend current sequence: `f[1] - 1`
     - Or start new sequence from previous `a`s: `f[0] - 1`
   - `f[0] = 0` (reset the count of consecutive `a`s)

3. **Otherwise**: Skip this character (it doesn't affect the variance for this pair)

**Step 4: Update Maximum Variance**

After processing each character, we check if `f[1]` is greater than our current maximum variance `ans`. The condition `if ans < f[1]` updates the answer whenever we find a larger variance.

**Why This Works**

The key insight is that `f[1]` always represents a valid substring containing at least one `b` (due to initialization with `-inf` and the update rules). The `max(f[1] - 1, f[0] - 1)` operation when encountering `b` ensures we either:
- Continue the current valid substring
- Start a new substring that includes the previous `a` occurrences and current `b`

This guarantees we find the maximum variance for each character pair, and by trying all pairs, we find the overall maximum.

**Time Complexity**: `O(26 × 25 × n) = O(n)` where `n` is the length of string `s`
**Space Complexity**: `O(1)` as we only use constant extra space

## Example Walkthrough

Let's walk through the algorithm with string `s = "aababbb"`.

We'll focus on one character pair to illustrate: `(a='a', b='b')` where we want to maximize count of 'a' and minimize count of 'b'.

**Initial State:**
- `f[0] = 0` (no consecutive 'a's yet)
- `f[1] = -inf` (no valid substring with both characters yet)
- `ans = 0` (global maximum variance)

**Processing each character:**

1. **s[0] = 'a'**: This is our 'a' character
   - `f[0] = 0 + 1 = 1` (one 'a')
   - `f[1] = -inf + 1 = -inf` (still no 'b' seen)
 
2. **s[1] = 'a'**: Another 'a'
   - `f[0] = 1 + 1 = 2` (two consecutive 'a's)
   - `f[1] = -inf + 1 = -inf` (still no 'b' seen)
 
3. **s[2] = 'b'**: This is our 'b' character
   - `f[1] = max(-inf - 1, 2 - 1) = max(-inf, 1) = 1`
     - We start a new valid substring: "aab" has variance 2-1=1
   - `f[0] = 0` (reset consecutive 'a' count)
   - Update `ans = max(0, 1) = 1`
 
4. **s[3] = 'a'**: Back to 'a'
   - `f[0] = 0 + 1 = 1` 
   - `f[1] = 1 + 1 = 2` (substring "aaba" has 3 'a's and 1 'b', variance = 3-1=2)
   - Update `ans = max(1, 2) = 2`
 
5. **s[4] = 'b'**: Another 'b'
   - `f[1] = max(2 - 1, 1 - 1) = max(1, 0) = 1`
     - Continuing "aabab" gives variance (3-2)=1
     - Starting fresh from the last 'a' gives "ab" with variance (1-1)=0
   - `f[0] = 0`
   - `ans` remains 2
 
6. **s[5] = 'b'**: Another 'b'
   - `f[1] = max(1 - 1, 0 - 1) = max(0, -1) = 0`
   - `f[0] = 0`
   - `ans` remains 2
 
7. **s[6] = 'b'**: Final 'b'
   - `f[1] = max(0 - 1, 0 - 1) = max(-1, -1) = -1`
   - `f[0] = 0`
   - `ans` remains 2

For this character pair (a='a', b='b'), the maximum variance found is 2, which corresponds to substring "aaba" with 3 'a's and 1 'b'.

The algorithm would then try all other 25×24 character pairs. For example:
- (a='b', b='a'): Would find substring "babbb" with variance 4-1=3
- Other pairs would yield lower or invalid results

The final answer would be the maximum across all pairs: **3**.

## Python Solution

```python
from itertools import permutations
from string import ascii_lowercase
from math import inf

class Solution:
    def largestVariance(self, s: str) -> int:
        max_variance = 0
      
        # Try all possible pairs of distinct characters
        for char_high, char_low in permutations(ascii_lowercase, 2):
            if char_high == char_low:
                continue
          
            # dp[0]: max difference when we haven't seen char_low yet
            # dp[1]: max difference when we have seen at least one char_low
            dp = [0, -inf]
          
            # Process each character in the string
            for current_char in s:
                if current_char == char_high:
                    # Increment both states when we see the high frequency character
                    dp[0] = dp[0] + 1
                    dp[1] = dp[1] + 1
                elif current_char == char_low:
                    # When we see the low frequency character:
                    # - Update dp[1] by either continuing from dp[1]-1 or starting fresh from dp[0]-1
                    # - Reset dp[0] to 0 since we've now seen char_low
                    dp[1] = max(dp[1] - 1, dp[0] - 1)
                    dp[0] = 0
              
                # Update maximum variance if current state with char_low is better
                if max_variance < dp[1]:
                    max_variance = dp[1]
      
        return max_variance
```

## Time and Space Complexity

**Time Complexity:** `O(n × |Σ|²)`

The algorithm iterates through all permutations of 2 characters from the alphabet using `permutations(ascii_lowercase, 2)`. Since `ascii_lowercase` contains 26 letters, this generates `26 × 25 = 650` pairs (excluding identical pairs). This gives us `|Σ|²` iterations where `|Σ| = 26` is the size of the character set.

For each character pair `(a, b)`, the algorithm traverses the entire string `s` once, which takes `O(n)` time where `n` is the length of the string. Within each iteration of the string traversal, only constant time operations are performed (comparisons, arithmetic operations, and max operations).

Therefore, the total time complexity is `O(|Σ|² × n)` which can also be written as `O(n × |Σ|²)`.

**Space Complexity:** `O(1)`

The algorithm uses only a fixed amount of extra space:
- The list `f` with exactly 2 elements: `f[0]` and `f[1]`
- A few scalar variables: `ans`, `a`, `b`, and `c`
- The permutation generator creates pairs on-the-fly without storing all of them

Since the space usage doesn't grow with the input size and remains constant regardless of the length of string `s`, the space complexity is `O(1)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Forgetting to Initialize `dp[1]` with Negative Infinity

**The Problem:**
A common mistake is initializing `dp[1]` with `0` instead of `-inf`. This would incorrectly consider substrings that don't contain the low-frequency character `char_low` as valid, leading to wrong answers.

```python
# WRONG: This initialization allows invalid substrings
dp = [0, 0]  # ❌ Incorrect

# CORRECT: Ensures we only count valid substrings with both characters
dp = [0, -inf]  # ✅ Correct
```

**Why It Matters:**
The variance is only meaningful when both characters are present in the substring. By initializing `dp[1]` with `-inf`, we ensure that it only becomes non-negative after encountering at least one `char_low`, guaranteeing our substring contains both characters.

### Pitfall 2: Incorrect State Transition When Encountering `char_low`

**The Problem:**
Some might incorrectly update the states when encountering `char_low`:

```python
# WRONG: These transitions miss important cases
if current_char == char_low:
    dp[1] = dp[0] - 1  # ❌ Ignores continuing from dp[1]
    dp[0] = 0

# OR
if current_char == char_low:
    dp[1] = dp[1] - 1  # ❌ Ignores starting fresh from dp[0]
    dp[0] = 0

# CORRECT: Consider both possibilities
if current_char == char_low:
    dp[1] = max(dp[1] - 1, dp[0] - 1)  # ✅ Takes the better option
    dp[0] = 0
```

**Why It Matters:**
The `max(dp[1] - 1, dp[0] - 1)` operation allows us to either:
- Continue an existing valid substring that already contains both characters (`dp[1] - 1`)
- Start a new valid substring using accumulated `char_high` occurrences (`dp[0] - 1`)

This flexibility is crucial for finding the maximum variance.

### Pitfall 3: Not Resetting `dp[0]` After Encountering `char_low`

**The Problem:**
Forgetting to reset `dp[0]` to `0` after encountering `char_low`:

```python
# WRONG: Keeps counting consecutive char_high incorrectly
if current_char == char_low:
    dp[1] = max(dp[1] - 1, dp[0] - 1)
    # Missing: dp[0] = 0  ❌

# CORRECT: Reset the consecutive count
if current_char == char_low:
    dp[1] = max(dp[1] - 1, dp[0] - 1)
    dp[0] = 0  # ✅ Essential reset
```

**Why It Matters:**
`dp[0]` represents the count of consecutive `char_high` occurrences without any `char_low`. Once we encounter `char_low`, this consecutive sequence is broken, so we must reset `dp[0]` to `0`.

### Complete Corrected Solution:

```python
from itertools import permutations
from string import ascii_lowercase
from math import inf

class Solution:
    def largestVariance(self, s: str) -> int:
        max_variance = 0
      
        for char_high, char_low in permutations(ascii_lowercase, 2):
            # Initialize with -inf to ensure valid substrings
            dp = [0, -inf]  # ✅ Correct initialization
          
            for current_char in s:
                if current_char == char_high:
                    dp[0] = dp[0] + 1
                    dp[1] = dp[1] + 1
                elif current_char == char_low:
                    # Consider both transition options
                    dp[1] = max(dp[1] - 1, dp[0] - 1)  # ✅ Correct transition
                    dp[0] = 0  # ✅ Reset consecutive count
              
                if max_variance < dp[1]:
                    max_variance = dp[1]
      
        return max_variance
```

