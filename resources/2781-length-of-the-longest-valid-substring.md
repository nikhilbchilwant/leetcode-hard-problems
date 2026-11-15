# 2781. Length of the Longest Valid Substring

> Source: https://algo.monster/liteproblems/2781

## Problem Statement

You are given a string `word` and an array of strings `forbidden`.

A string is considered **valid** if it doesn't contain any of the forbidden strings as substrings. In other words, none of the strings from the `forbidden` array should appear anywhere within the valid string.

Your task is to find the **longest valid substring** that can be formed from the given `word`. You need to return the length of this longest valid substring.

A **substring** is defined as a contiguous sequence of characters within a string, which can also be empty.

For example:
- If `word = "cbaaaabc"` and `forbidden = ["aaa", "cb"]`, then the longest valid substring would be `"aabc"` (from index 2 to 5) with length 4, since it doesn't contain "aaa" or "cb" as substrings.
- The substring "cbaaaa" would be invalid because it contains "cb" at the beginning and "aaa" at the end.

The key challenge is to efficiently find the longest contiguous portion of the original word that avoids all forbidden patterns.

## Intuition

The natural first thought might be to check every possible substring of `word` to see if it's valid, but this would be extremely inefficient. Instead, we can think about this problem using a [sliding window](/problems/sliding_window_maximum) approach.

The key insight is that if we find a forbidden substring within our current window, we need to adjust our window to exclude it. When we encounter a forbidden pattern, the leftmost valid position for our window would be right after where that forbidden pattern starts.

Consider scanning through the string from left to right. At each position `j` (right boundary of our window), we want to maintain the largest valid window ending at `j`. If we find a forbidden substring that ends at position `j`, we must move our left boundary `i` to exclude this forbidden pattern.

Here's the crucial observation: forbidden strings have limited length. In most practical cases, forbidden strings are relatively short (the solution assumes a maximum length of 10). This means when checking if position `j` creates any forbidden substring, we only need to look back a limited number of positions.

For each position `j`, we check all possible substrings ending at `j` that could potentially be forbidden. We start checking from `j` and go backwards (checking substrings `word[j:j+1]`, `word[j-1:j+1]`, `word[j-2:j+1]`, etc.). If we find a forbidden substring `word[k:j+1]`, we know that any valid window must start after position `k`, so we update `i = k + 1`.

By checking backwards from `j` to `max(j - 10, i - 1)`, we ensure we're checking all relevant forbidden patterns while maintaining efficiency. The `i - 1` bound ensures we don't check positions before our current valid window start, as those have already been excluded for good reason.

This approach efficiently maintains the longest valid substring ending at each position, ultimately giving us the maximum length across all positions.

**Learn more about [Sliding Window](/problems/sliding_window_maximum) patterns.**

## Solution Approach

We implement a [sliding window](/problems/sliding_window_maximum) approach with an optimization based on the bounded length of forbidden strings.

**Data Structures:**
- Convert the `forbidden` list into a set `s` for O(1) lookup time when checking if a substring is forbidden
- Use two pointers: `i` (left boundary of the valid window) and `j` (right boundary being explored)
- Variable `ans` to track the maximum valid substring length found

**Algorithm Steps:**

1. **Initialize:** Create a set from forbidden strings, set `ans = 0` and `i = 0` (left boundary starts at beginning)

2. **Iterate through each position:** For each position `j` from 0 to `len(word) - 1`:
 
   a. **Check for forbidden substrings ending at j:**
      - Iterate backwards from position `j` to `max(j - 10, i - 1)`
      - The limit of 10 assumes forbidden strings have maximum length 10
      - We don't need to check before `i - 1` since our valid window starts at `i`
 
   b. **For each starting position k in the backward scan:**
      - Check if `word[k : j + 1]` is in the forbidden set
      - If found, update `i = k + 1` (move left boundary past the start of forbidden substring)
      - Break immediately since we've found the rightmost forbidden substring ending at `j`
 
   c. **Update maximum length:**
      - Calculate current valid window size: `j - i + 1`
      - Update `ans = max(ans, j - i + 1)`

3. **Return** the maximum length found

**Key Optimizations:**
- Using a set for O(1) forbidden string lookups
- Limiting the backward search to 10 positions (or less if near the valid window start)
- Breaking immediately when finding a forbidden substring, as this gives us the tightest constraint

**Time Complexity:** O(n × m) where n is the length of `word` and m is the maximum length of forbidden strings (assumed to be 10)

**Space Complexity:** O(p × m) where p is the number of forbidden strings and m is their average length (for storing the set)

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Example:** `word = "abacab"`, `forbidden = ["ab", "bac"]`

**Step-by-step process:**

1. **Initialize:** 
   - Create set `s = {"ab", "bac"}`
   - Set `i = 0` (left boundary), `ans = 0` (max length)

2. **j = 0** (examining 'a' at position 0):
   - Check substrings ending at position 0:
     - `word[0:1] = "a"` → not forbidden
   - Valid window: [0, 0] = "a", length = 1
   - Update `ans = 1`

3. **j = 1** (examining 'b' at position 1):
   - Check substrings ending at position 1:
     - `word[1:2] = "b"` → not forbidden
     - `word[0:2] = "ab"` → **FORBIDDEN!**
   - Found forbidden substring starting at position 0
   - Update `i = 0 + 1 = 1` (move left boundary past the forbidden substring)
   - Valid window: [1, 1] = "b", length = 1
   - `ans` remains 1

4. **j = 2** (examining 'a' at position 2):
   - Check substrings ending at position 2 (from j=2 down to max(2-10, i-1)=0):
     - `word[2:3] = "a"` → not forbidden
     - `word[1:3] = "ba"` → not forbidden
     - `word[0:3] = "aba"` → not forbidden
   - Valid window: [1, 2] = "ba", length = 2
   - Update `ans = 2`

5. **j = 3** (examining 'c' at position 3):
   - Check substrings ending at position 3:
     - `word[3:4] = "c"` → not forbidden
     - `word[2:4] = "ac"` → not forbidden
     - `word[1:4] = "bac"` → **FORBIDDEN!**
   - Found forbidden substring starting at position 1
   - Update `i = 1 + 1 = 2` (move left boundary past "bac")
   - Valid window: [2, 3] = "ac", length = 2
   - `ans` remains 2

6. **j = 4** (examining 'a' at position 4):
   - Check substrings ending at position 4 (from j=4 down to max(4-10, i-1)=1):
     - `word[4:5] = "a"` → not forbidden
     - `word[3:5] = "ca"` → not forbidden
     - `word[2:5] = "aca"` → not forbidden
     - `word[1:5] = "baca"` → not forbidden (we stop at i-1=1)
   - Valid window: [2, 4] = "aca", length = 3
   - Update `ans = 3`

7. **j = 5** (examining 'b' at position 5):
   - Check substrings ending at position 5:
     - `word[5:6] = "b"` → not forbidden
     - `word[4:6] = "ab"` → **FORBIDDEN!**
   - Found forbidden substring starting at position 4
   - Update `i = 4 + 1 = 5` (move left boundary past "ab")
   - Valid window: [5, 5] = "b", length = 1
   - `ans` remains 3

**Result:** The longest valid substring has length **3** (the substring "aca" from positions 2-4).

This example demonstrates how the sliding window adjusts its left boundary whenever a forbidden pattern is detected, always maintaining the longest possible valid substring at each step.

## Python Solution

```python
class Solution:
    def longestValidSubstring(self, word: str, forbidden: List[str]) -> int:
        # Convert forbidden list to set for O(1) lookup
        forbidden_set = set(forbidden)
      
        # Initialize variables
        max_length = 0  # Track the maximum valid substring length
        left_boundary = 0  # Left boundary of the current valid substring
      
        # Iterate through each position as the right endpoint of substring
        for right in range(len(word)):
            # Check substrings ending at 'right', starting from 'right' going backwards
            # Only check last 10 characters (constraint: forbidden strings have max length 10)
            # Stop at left_boundary - 1 to avoid rechecking invalid positions
            for left in range(right, max(right - 10, left_boundary - 1), -1):
                # Extract substring from left to right (inclusive)
                substring = word[left:right + 1]
              
                # If substring is forbidden, update left boundary to exclude it
                if substring in forbidden_set:
                    left_boundary = left + 1
                    break
          
            # Update maximum length with current valid substring length
            current_length = right - left_boundary + 1
            max_length = max(max_length, current_length)
      
        return max_length
```

## Time and Space Complexity

**Time Complexity:** `O(n * m)`

The algorithm uses a sliding window approach with two pointers. The outer loop runs `n` times where `n` is the length of the word. For each position `j`, the inner loop checks substrings ending at position `j`. The inner loop runs at most 10 iterations (bounded by `max(j - 10, i - 1)`), since we're checking substrings of maximum length 10. The substring extraction `word[k : j + 1]` takes `O(m)` time where `m` is the maximum length of forbidden strings (at most 10). The set lookup operation `in s` takes `O(m)` time on average for string comparison. Therefore, the overall time complexity is `O(n * 10 * m) = O(n * m)`.

**Space Complexity:** `O(k * m)`

The space complexity consists of:
- The set `s` storing all forbidden strings: `O(k * m)` where `k` is the number of forbidden strings and `m` is the average length of forbidden strings
- Variables `ans`, `i`, `j`, `k`: `O(1)`
- Temporary substring creation `word[k : j + 1]`: `O(m)` for each substring checked

The dominant factor is the storage of the forbidden strings set, giving us a total space complexity of `O(k * m)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Not Breaking After Finding a Forbidden Substring**

**The Pitfall:** 
When checking for forbidden substrings ending at position `right`, some implementations might continue checking all possible starting positions even after finding a forbidden substring. This leads to incorrect results because it might unnecessarily move the `left_boundary` further right than needed.

**Why It's Wrong:**
```python
# INCORRECT approach - continues checking after finding forbidden substring
for left in range(right, max(right - 10, left_boundary - 1), -1):
    substring = word[left:right + 1]
    if substring in forbidden_set:
        left_boundary = left + 1
        # Missing break here! Will continue and potentially update left_boundary again
```

If we don't break, we might find another forbidden substring that starts earlier, causing us to update `left_boundary` to an even more restrictive position, unnecessarily shrinking our valid window.

**Example Scenario:**
- `word = "abcdef"`, `forbidden = ["cde", "abcde"]`
- At position `right = 4` (character 'e')
- Without break: First finds "cde" (updates `left_boundary = 3`), then finds "abcde" (updates `left_boundary = 1`)
- With break: Finds "cde" first (updates `left_boundary = 3`) and stops
- The correct behavior is to stop at the first (rightmost starting) forbidden substring

### 2. **Incorrect Loop Boundary Calculation**

**The Pitfall:**
Using `max(right - 10, left_boundary)` instead of `max(right - 10, left_boundary - 1)` in the inner loop range.

**Why It's Wrong:**
```python
# INCORRECT - doesn't check position left_boundary itself
for left in range(right, max(right - 10, left_boundary), -1):
    # This skips checking if substring starting at left_boundary is forbidden
```

This misses checking substrings that start exactly at `left_boundary`, which could be forbidden. We need to check down to `left_boundary - 1` to ensure we examine all relevant positions.

### 3. **Assuming Fixed Maximum Length for Forbidden Strings**

**The Pitfall:**
Hard-coding the value 10 as the maximum forbidden string length without verifying this constraint or making it adaptable.

**Better Solution:**
```python
def longestValidSubstring(self, word: str, forbidden: List[str]) -> int:
    forbidden_set = set(forbidden)
    # Calculate actual maximum length of forbidden strings
    max_forbidden_length = max(len(f) for f in forbidden) if forbidden else 0
  
    max_length = 0
    left_boundary = 0
  
    for right in range(len(word)):
        # Use actual maximum length instead of hardcoded 10
        for left in range(right, max(right - max_forbidden_length, left_boundary - 1), -1):
            substring = word[left:right + 1]
            if substring in forbidden_set:
                left_boundary = left + 1
                break
      
        max_length = max(max_length, right - left_boundary + 1)
  
    return max_length
```

### 4. **Off-by-One Error in Substring Extraction**

**The Pitfall:**
Confusion with Python's slice notation can lead to incorrect substring extraction.

```python
# INCORRECT - might use wrong indices
substring = word[left:right]  # Missing the character at position 'right'
```

**Correct Approach:**
```python
substring = word[left:right + 1]  # Includes character at position 'right'
```

Remember that Python slicing `[start:end]` includes `start` but excludes `end`, so we need `right + 1` to include the character at position `right`.

