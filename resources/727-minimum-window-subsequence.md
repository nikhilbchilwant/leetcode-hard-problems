# 727. Minimum Window Subsequence

> Source: https://algo.monster/liteproblems/727

## Problem Statement

You are given two strings `s1` and `s2`. Your task is to find the minimum contiguous substring of `s1` such that `s2` is a subsequence of this substring.

A subsequence means that the characters of `s2` appear in the substring in the same order, but not necessarily consecutively. For example, "ace" is a subsequence of "aec**ace**" (the bold part shows one valid window).

The requirements are:
- Find the shortest contiguous substring of `s1` that contains all characters of `s2` as a subsequence
- If multiple substrings of the same minimum length exist, return the one that starts earliest (leftmost) in `s1`
- If no such substring exists, return an empty string `""`

For example:
- If `s1 = "abcdebdde"` and `s2 = "bde"`, the answer would be `"bcde"` because:
  - The substring `"bcde"` contains `s2 = "bde"` as a subsequence
  - While `"bdde"` at the end also contains `"bde"` as a subsequence and has the same length, `"bcde"` appears first

## Intuition

The key insight is that we need to track where valid subsequences start as we scan through `s1`. Since we're looking for the minimum window containing `s2` as a subsequence, we need to know: "If I'm at position `i` in `s1` and I've matched `j` characters of `s2`, where did this matching sequence start?"

Think about it this way: as we scan through `s1`, at each position we can potentially match characters from `s2`. When we match a character, we're either:
1. Starting a new subsequence (matching the first character of `s2`)
2. Extending an existing subsequence (matching the next character of `s2`)

This naturally leads to a [dynamic programming](/problems/dynamic_programming_intro) approach where we track the starting positions of valid subsequences.

The brilliant part is using `dp[i][j]` to store the starting position (not just a boolean or count). Why? Because once we've matched all characters of `s2` (reached `j = n`), we need to know where this subsequence started to calculate the window length.

Consider matching `s2 = "bde"` in `s1 = "abcdebdde"`:
- When we see 'b' at position 2, we can start matching `s2`
- When we see 'd' at position 4, we can extend our match if we had 'b' before
- When we see 'e' at position 5, we complete the match and know it started at position 2

The state transition naturally follows:
- If characters match and it's the first character of `s2`, record current position as start
- If characters match and it's not the first character, inherit the start position from the previous match
- If characters don't match, keep the previous best start position for this many matched characters

Finally, we scan through all positions where we've matched all of `s2` and pick the shortest window.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) and [Sliding Window](/problems/sliding_window_maximum) patterns.**

## Solution Approach

We implement the solution using a 2D [dynamic programming](/problems/dynamic_programming_intro) table `dp[i][j]` where:
- `i` represents the position in string `s1` (from 0 to m)
- `j` represents the position in string `s2` (from 0 to n)
- `dp[i][j]` stores the starting position (1-indexed) of the shortest substring of `s1[0...i-1]` that contains `s2[0...j-1]` as a subsequence

**Step 1: Initialize the DP table**
```python
dp = [[0] * (len_s2 + 1) for _ in range(len_s1 + 1)]
```
We create a table of size `(m+1) × (n+1)` initialized with zeros. Zero indicates no valid subsequence exists.

**Step 2: Fill the DP table**
For each position `i` in `s1` and each position `j` in `s2`:

```python
if s1[i - 1] == s2[j - 1]:
    dp[i][j] = i if j == 1 else dp[i - 1][j - 1]
else:
    dp[i][j] = dp[i - 1][j]
```

The state transition follows these rules:
- When `s1[i-1] == s2[j-1]`:
  - If `j == 1` (matching first character of `s2`): Set `dp[i][j] = i` (start a new subsequence here)
  - If `j > 1` (matching subsequent characters): Set `dp[i][j] = dp[i-1][j-1]` (inherit start position from previous match)
- When `s1[i-1] != s2[j-1]`:
  - Set `dp[i][j] = dp[i-1][j]` (keep the best start position found so far for matching `j` characters)

**Step 3: Find the minimum window**
```python
start_index, min_length = 0, len_s1 + 1
for i in range(1, len_s1 + 1):
    if s1[i - 1] == s2[len_s2 - 1] and dp[i][len_s2] != 0:
        window_start = dp[i][len_s2] - 1
        window_length = i - window_start
        if window_length < min_length:
            min_length = window_length
            start_index = window_start
```

We scan through `s1` looking for positions where:
- The character matches the last character of `s2`
- We have a valid subsequence ending here (`dp[i][len_s2] != 0`)

For each valid ending position, we calculate the window length as `i - (dp[i][len_s2] - 1)` and keep track of the minimum. Because the comparison is strict (`<`), a later window of the same length does not replace an earlier one.

**Step 4: Return the result**
```python
return "" if min_length > len_s1 else s1[start_index : start_index + min_length]
```

If no valid window was found (`min_length > len_s1`), return empty string. Otherwise, return the substring from position `start_index` with length `min_length`.

**Time Complexity:** O(m × n) where m = len(s1) and n = len(s2)
**Space Complexity:** O(m × n) for the DP table

## Example Walkthrough

Let's walk through the algorithm with `s1 = "abcdebdde"` and `s2 = "bde"`.

**Setup:**
- `m = 9` (length of s1), `n = 3` (length of s2)
- Initialize DP table `dp[10][4]` with all zeros

**Building the DP Table:**

We'll process each character of s1 and track where valid subsequences start.

| i | s1[i-1] | j=1 (need 'b') | j=2 (need 'd') | j=3 (need 'e') |
|---|---------|----------------|----------------|----------------|
| 1 | 'a' | dp[1][1]=0 (no match) | dp[1][2]=0 | dp[1][3]=0 |
| 2 | 'b' | dp[2][1]=2 (match! start here) | dp[2][2]=0 | dp[2][3]=0 |
| 3 | 'c' | dp[3][1]=2 (keep previous) | dp[3][2]=0 | dp[3][3]=0 |
| 4 | 'd' | dp[4][1]=2 | dp[4][2]=2 (match! inherit from dp[3][1]) | dp[4][3]=0 |
| 5 | 'e' | dp[5][1]=2 | dp[5][2]=2 | dp[5][3]=2 (match! inherit from dp[4][2]) |
| 6 | 'b' | dp[6][1]=6 (new 'b' starts here) | dp[6][2]=2 | dp[6][3]=2 |
| 7 | 'd' | dp[7][1]=6 | dp[7][2]=6 (match! inherit from dp[6][1]) | dp[7][3]=2 |
| 8 | 'd' | dp[8][1]=6 | dp[8][2]=6 (match! inherit from dp[7][1]) | dp[8][3]=2 |
| 9 | 'e' | dp[9][1]=6 | dp[9][2]=6 | dp[9][3]=6 (match! inherit from dp[8][2]) |

**Key observations:**
- At i=2: 'b' matches s2[0], so we start a new subsequence (dp[2][1]=2)
- At i=4: 'd' matches s2[1], and we had 'b' before, so dp[4][2]=2 (inherited from dp[3][1])
- At i=5: 'e' matches s2[2], completing "bde" starting from position 2
- At i=6: Another 'b' starts a new potential subsequence
- At i=9: 'e' completes another "bde" sequence starting from position 6

**Finding the Minimum Window:**

Now we scan for positions where we've matched all of s2 (j=3):

1. At i=5: s1[4]='e' matches s2[2], and dp[5][3]=2
   - Window: s1[1:5] = "bcde" (positions 2-5)
   - Length: 5 - 2 + 1 = 4

2. At i=9: s1[8]='e' matches s2[2], and dp[9][3]=6
   - Window: s1[5:9] = "bdde" (positions 6-9)
   - Length: 9 - 6 + 1 = 4

Both windows have length 4, but "bcde" starts earlier (position 1 vs position 5 in 0-indexed), so we return "bcde".

**Why the DP approach works:**
- `dp[i][j]` tracks where the shortest valid subsequence starts, allowing us to calculate window lengths
- By processing left-to-right, we naturally find the leftmost window when lengths are equal
- The state transitions ensure we only extend valid subsequences and always keep the best (earliest) starting position

## Python Solution

```python
class Solution:
    def minWindow(self, s1: str, s2: str) -> str:
        len_s1, len_s2 = len(s1), len(s2)

        # dp[i][j] stores the starting index (1-based) in s1 where
        # the subsequence s2[0:j] ends at position i in s1
        # 0 means no valid subsequence found
        dp = [[0] * (len_s2 + 1) for _ in range(len_s1 + 1)]

        # Fill the dp table
        for i in range(1, len_s1 + 1):
            for j in range(1, len_s2 + 1):
                if s1[i - 1] == s2[j - 1]:
                    # Characters match
                    if j == 1:
                        # First character of s2, start index is current position
                        dp[i][j] = i
                    else:
                        # Continue from previous subsequence
                        dp[i][j] = dp[i - 1][j - 1]
                else:
                    # Characters don't match, inherit from previous position in s1
                    dp[i][j] = dp[i - 1][j]

        # Find the minimum window
        start_index = 0
        min_length = len_s1 + 1

        for i in range(1, len_s1 + 1):
            # Check if s1[i-1] matches the last character of s2
            # and we have a valid subsequence ending at position i
            if s1[i - 1] == s2[len_s2 - 1] and dp[i][len_s2] != 0:
                # Calculate window start (0-based) and length
                window_start = dp[i][len_s2] - 1
                window_length = i - window_start

                # Update minimum window if current one is smaller
                if window_length < min_length:
                    min_length = window_length
                    start_index = window_start

        # Return the minimum window substring or empty string if not found
        return "" if min_length > len_s1 else s1[start_index : start_index + min_length]
```

## Time and Space Complexity

The time complexity is `O(m × n)`, where `m` is the length of string `s1` and `n` is the length of string `s2`. This is because:
- The code contains two nested loops that iterate through `s1` (length `m`) and `s2` (length `n`) to fill the DP table `dp`, resulting in `O(m × n)` operations.
- The second loop iterates through `s1` once more with `O(m)` operations, but this is dominated by the `O(m × n)` complexity.

The space complexity is `O(m × n)` due to the 2D DP table `dp` which has dimensions `(m + 1) × (n + 1)`, storing the starting positions for minimum window subsequences.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Index Conversion Between 1-based and 0-based Systems**

The most common pitfall in this solution is mixing up 1-based indexing (used in the DP table) with 0-based indexing (used for string access). The DP table stores 1-based indices to distinguish between "no valid subsequence" (0) and "valid subsequence starting at position 0" (which would be 1 in the table).

**Pitfall Example:**
```python
# WRONG: Forgetting to subtract 1 when converting from DP index to string index
window_start = dp[i][len_s2]  # This is 1-based!
return s1[window_start : window_start + min_length]  # Error: using 1-based index in 0-based string
```

**Solution:**
Always remember to subtract 1 when converting from DP table indices to string indices:
```python
window_start = dp[i][len_s2] - 1  # Convert to 0-based
```

### 2. **Not Checking for Valid Subsequence Before Calculating Window**

Another pitfall is attempting to calculate the window length when `dp[i][len_s2] == 0`, which means no valid subsequence exists.

**Pitfall Example:**
```python
# WRONG: Not checking if dp[i][len_s2] is non-zero
if s1[i - 1] == s2[len_s2 - 1]:  # Missing check for dp[i][len_s2] != 0
    window_start = dp[i][len_s2] - 1  # This could be -1 if dp[i][len_s2] is 0!
```

**Solution:**
Always verify that a valid subsequence exists before processing:
```python
if s1[i - 1] == s2[len_s2 - 1] and dp[i][len_s2] != 0:
    window_start = dp[i][len_s2] - 1
```

### 3. **Incorrect Window Length Calculation**

The window length should be calculated as the difference between the current ending position and the starting position.

**Pitfall Example:**
```python
# WRONG: Subtracting the 1-based start directly
window_length = i - dp[i][len_s2]  # One shorter than the real window
```

For `s1 = "abcdebdde"`, `s2 = "bde"`, at `i = 5` this gives `5 - 2 = 3`, but the window `"bcde"` has length 4. The slice `s1[start_index : start_index + 3]` would then return `"bcd"`, which does not contain `"bde"`.

**Solution:**
The correct calculation accounts for the fact that `i` is 1-based and `window_start` is converted to 0-based (equivalently, `i - dp[i][len_s2] + 1`):
```python
window_start = dp[i][len_s2] - 1  # 0-based start
window_length = i - window_start  # i is 1-based, so this gives the correct length
```

### 4. **Edge Case: Empty Strings or s2 Longer Than s1**

Failing to handle edge cases where `s2` is empty or longer than `s1` can cause issues.

**Solution:**
Add validation at the beginning:
```python
if not s2:
    return ""
if len(s2) > len(s1):
    return ""
```

### 5. **Misunderstanding the DP State Definition**

A conceptual pitfall is misunderstanding what `dp[i][j]` represents. It's crucial to remember that it stores the **starting position** (1-based) of the shortest substring ending at position `i` that contains the first `j` characters of `s2` as a subsequence.

**Pitfall Example:**
```python
# WRONG: Thinking dp[i][j] stores the ending position or length
min_length = dp[i][len_s2]  # This is NOT the length!
```

**Solution:**
Always remember that `dp[i][j]` is a starting position, and calculate the length as:
```python
window_length = i - (dp[i][len_s2] - 1)  # Current position minus start position
```

