# 2014. Longest Subsequence Repeated k Times

> Source: https://algo.monster/liteproblems/2014

## Problem Statement

You are given a string `s` of length `n` and an integer `k`. Your task is to find the **longest subsequence** that can be **repeated** `k` times in string `s`.

A **subsequence** is a string that can be derived from another string by deleting some or no characters without changing the order of the remaining characters.

A subsequence `seq` is considered **repeated** `k` times in string `s` if `seq * k` (which means concatenating `seq` with itself `k` times) is a subsequence of `s`.

For example, consider the string `"bababcba"` and the subsequence `"bba"`:
- If we concatenate `"bba"` 2 times, we get `"bbabba"`
- We can find `"bbabba"` as a subsequence in `"bababcba"` by selecting characters at positions: `b`(0)`b`(2)`ab`(3,4)`ba`(6,7)
- Therefore, `"bba"` is repeated 2 times in the string

The goal is to return the **longest** subsequence that can be repeated `k` times. If there are multiple subsequences of the same maximum length, return the **lexicographically largest** one. If no such subsequence exists, return an **empty string**.

## Intuition

When we need to find the longest subsequence that can be repeated `k` times, we face two key challenges: generating all possible subsequences and efficiently checking if each one can be repeated `k` times.

The first insight is that any character appearing in our answer subsequence must appear at least `k` times in the original string. Why? Because if we repeat the subsequence `k` times, each character in it will appear `k` times in the concatenated result. So we can immediately filter out characters that don't meet this frequency requirement.

The second insight is about how to systematically explore all possible subsequences. Since we want the longest one (and lexicographically largest if there are ties), we can use a level-by-level exploration approach. We start with an empty string and gradually build longer subsequences by appending one character at a time. This is where BFS shines - it naturally explores all subsequences of length 1, then length 2, and so on.

Why BFS instead of DFS? BFS guarantees that when we find a valid subsequence of length `n`, we've already explored all subsequences of length `n-1`. This means the last valid subsequence we find at each level is guaranteed to be one of the longest. Since we append characters in order from our filtered character set, we also ensure lexicographic ordering within each length.

The verification step is straightforward but crucial. For each candidate subsequence, we need to check if we can find it `k` times in the original string. We do this by scanning through the string with two pointers - one for the original string and one for our subsequence pattern. Each time we complete matching the pattern, we decrement `k`. If we can match the pattern `k` times before reaching the end of the string, the subsequence is valid.

This approach elegantly combines the exhaustive search needed for finding all subsequences with the efficiency of pruning invalid candidates early, making it practical despite the potentially large search space.

**Learn more about [Greedy](/problems/greedy_intro) and [Backtracking](/problems/backtracking) patterns.**

## Solution Approach

The solution implements a BFS-based approach to systematically explore all possible subsequences and find the longest one that can be repeated `k` times.

**Step 1: Character Filtering**

First, we count the frequency of each character in the string using a `Counter`. Then we filter out characters that appear less than `k` times, storing valid characters in a list `cs`:

```python
cnt = Counter(s)
cs = [c for c in ascii_lowercase if cnt[c] >= k]
```

This preprocessing step ensures we only consider characters that could possibly be part of a subsequence repeated `k` times.

**Step 2: BFS Initialization**

We initialize a queue with an empty string and set up our answer variable:

```python
q = deque([""])
ans = ""
```

The empty string serves as our starting point for building subsequences.

**Step 3: Level-by-Level Exploration**

We process the queue using BFS, where each level represents subsequences of a particular length:

```python
while q:
    cur = q.popleft()
    for c in cs:
        nxt = cur + c
        if check(nxt, k):
            ans = nxt
            q.append(nxt)
```

For each subsequence `cur` from the queue, we try appending each valid character `c` to create a new subsequence `nxt`. If `nxt` can be repeated `k` times (verified by the `check` function), we update our answer and add `nxt` to the queue for further extension.

**Step 4: Verification Function**

The `check` function uses a two-pointer approach to verify if a subsequence `t` can be found `k` times in the string `s`:

```python
def check(t: str, k: int) -> bool:
    i = 0
    for c in s:
        if c == t[i]:
            i += 1
            if i == len(t):
                k -= 1
                if k == 0:
                    return True
                i = 0
    return False
```

The function maintains pointer `i` for the current position in pattern `t`. As we scan through `s`, whenever we find a matching character, we advance `i`. When we complete matching the entire pattern (`i == len(t)`), we decrement `k` and reset `i` to start matching the pattern again. If we successfully match the pattern `k` times, we return `True`.

**Why This Works:**

- BFS ensures we explore subsequences in increasing order of length, so the last valid subsequence we find is among the longest
- By iterating through characters in `cs` (which contains characters in alphabetical order), we naturally handle the lexicographic requirement
- The two-pointer verification is efficient, running in `O(n)` time for each check
- The overall approach guarantees finding the optimal solution by exhaustively checking all valid possibilities

## Example Walkthrough

Let's walk through the solution with `s = "bababcba"` and `k = 2`.

**Step 1: Character Filtering**
First, count character frequencies:
- 'a': 3 times
- 'b': 4 times  
- 'c': 1 time

Since we need characters that appear at least `k=2` times, we filter to get `cs = ['a', 'b']`.

**Step 2: BFS Exploration**

We start with queue = `[""]` and begin exploring:

**Level 0:** Process `""`
- Try appending 'a': Check if "a" can repeat 2 times
  - Scan: `b[a]babcba` → found 1st 'a' at index 1
  - Continue: `bab[a]bcba` → found 2nd 'a' at index 3
  - ✓ Valid! Update `ans = "a"`, add to queue
- Try appending 'b': Check if "b" can repeat 2 times
  - Scan: `[b]ababcba` → found 1st 'b' at index 0
  - Continue: `ba[b]abcba` → found 2nd 'b' at index 2
  - ✓ Valid! Update `ans = "b"`, add to queue

Queue now: `["a", "b"]`

**Level 1:** Process `"a"`
- Try appending 'a': Check if "aa" can repeat 2 times (need "aaaa")
  - Scan for 4 'a's total, but only have 3 'a's in string
  - ✗ Invalid, skip
- Try appending 'b': Check if "ab" can repeat 2 times (need "abab")
  - Scan: `b[a]babcba` → found 1st 'a' at index 1
  - Continue: `ba[b]abcba` → found 1st 'b' at index 2
  - Continue: `bab[a]bcba` → found 2nd 'a' at index 3
  - Continue: `baba[b]cba` → found 2nd 'b' at index 4
  - ✓ Valid! Update `ans = "ab"`, add to queue

**Level 1 (continued):** Process `"b"`
- Try appending 'a': Check if "ba" can repeat 2 times (need "baba")
  - Scan: `[b]ababcba` → found 1st 'b' at index 0
  - Continue: `b[a]babcba` → found 1st 'a' at index 1
  - Continue: `ba[b]abcba` → found 2nd 'b' at index 2
  - Continue: `bab[a]bcba` → found 2nd 'a' at index 3
  - ✓ Valid! Update `ans = "ba"`, add to queue
- Try appending 'b': Check if "bb" can repeat 2 times (need "bbbb")
  - Need 4 'b's in order, but can only find: b(0), b(2), b(4), b(6)
  - ✓ Valid! Update `ans = "bb"`, add to queue

Queue now: `["ab", "ba", "bb"]`

**Level 2:** Process `"ab"`, `"ba"`, `"bb"`

For `"ab"`:
- Try "aba": Check if can repeat 2 times (need "abaaba")
  - After checking: ✗ Cannot find this pattern twice
- Try "abb": Check if can repeat 2 times (need "abbabb")  
  - After checking: ✗ Cannot find this pattern twice

For `"ba"`:
- Try "baa": ✗ Invalid
- Try "bab": ✗ Invalid

For `"bb"`:
- Try "bba": Check if can repeat 2 times (need "bbabba")
  - Scan: `[b]ababcba` → 1st 'b'
  - Continue: `ba[b]abcba` → 2nd 'b'
  - Continue: `bab[a]bcba` → 1st 'a'
  - Continue: `baba[b]cba` → 3rd 'b'
  - Continue: `babab[b]a` → 4th 'b'
  - Continue: `bababcb[a]` → 2nd 'a'
  - ✓ Valid! Update `ans = "bba"`
- Try "bbb": ✗ Invalid

The process continues, but no subsequence longer than 3 characters can be repeated twice.

**Final Answer:** `"bba"` - the longest subsequence that can be repeated 2 times in "bababcba".

## Python Solution

```python
from collections import Counter, deque
from string import ascii_lowercase

class Solution:
    def longestSubsequenceRepeatedK(self, s: str, k: int) -> str:
        def is_valid_subsequence(pattern: str, required_count: int) -> bool:
            """
            Check if the pattern can be found as a subsequence in string s
            at least required_count times.
            """
            pattern_index = 0
            for char in s:
                if char == pattern[pattern_index]:
                    pattern_index += 1
                    # If we've matched the entire pattern once
                    if pattern_index == len(pattern):
                        required_count -= 1
                        # If we've found the pattern k times, return True
                        if required_count == 0:
                            return True
                        # Reset to find the pattern again
                        pattern_index = 0
            return False

        # Count frequency of each character in the string
        char_frequency = Counter(s)
      
        # Only consider characters that appear at least k times
        # (necessary condition for being in a subsequence repeated k times)
        valid_chars = [char for char in ascii_lowercase if char_frequency[char] >= k]
      
        # BFS queue to build subsequences incrementally
        queue = deque([""])
        result = ""
      
        # BFS to find the longest valid subsequence
        while queue:
            current_subsequence = queue.popleft()
          
            # Try extending the current subsequence with each valid character
            for char in valid_chars:
                next_subsequence = current_subsequence + char
              
                # Check if this new subsequence can be repeated k times
                if is_valid_subsequence(next_subsequence, k):
                    # Update result (BFS guarantees we find longest first)
                    result = next_subsequence
                    # Add to queue for further extension
                    queue.append(next_subsequence)
      
        return result
```

## Time and Space Complexity

**Time Complexity:** `O(n * m * 2^m)` where `n` is the length of string `s` and `m` is the number of distinct characters that appear at least `k` times in `s`.

- The algorithm uses BFS to explore all possible subsequences built from characters that appear at least `k` times
- In the worst case, we generate all possible strings from `m` characters, which gives us `O(2^m)` possible strings (each character can be included or not, multiple times)
- For each generated string of length `l`, the `check` function takes `O(n)` time to verify if it's a valid subsequence repeated `k` times
- The maximum length of any valid subsequence is bounded by `n/k`, but more tightly by `m * (n/k)` considering character frequencies
- The total number of strings generated is approximately `m + m^2 + m^3 + ... + m^(n/k) ≈ O(m^(n/k))` in the worst case, but practically bounded by valid subsequences
- Since we're doing BFS level by level and only keeping valid subsequences, the effective complexity is `O(n * m * 2^m)`

**Space Complexity:** `O(m * 2^m)` 

- The queue `q` can contain at most all valid subsequences at any point
- In the worst case, this could be all possible strings formed from `m` characters
- Each string has maximum length `n/k`, but the number of strings dominates the space complexity
- The counter and character list `cs` take `O(26) = O(1)` space for lowercase letters
- Therefore, the overall space complexity is `O(m * 2^m)` for storing all possible valid subsequences in the queue

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Character Frequency Filtering**

A common mistake is assuming that if a character appears exactly `k` times, it can be part of the repeated subsequence. However, this is only a necessary condition, not sufficient.

**Pitfall Example:**
```python
# Incorrect assumption
valid_chars = [char for char in s if s.count(char) == k]  # Wrong!
```

**Why it's wrong:** A character that appears `k` times can only appear once in the subsequence pattern (not multiple times). If a character appears `2k` times, it could appear twice in the pattern.

**Correct Approach:**
```python
# A character must appear AT LEAST k times to be in the pattern
valid_chars = [char for char in ascii_lowercase if char_frequency[char] >= k]
```

### 2. **Not Handling Lexicographic Order Properly**

The problem asks for the lexicographically largest subsequence among those with maximum length. A common pitfall is trying to find this by sorting or by complex comparisons.

**Pitfall Example:**
```python
# Trying to track all max-length subsequences and then sort
max_length_subsequences = []
# ... collect all subsequences of max length
return max(max_length_subsequences)  # Inefficient and complex
```

**Why BFS naturally handles this:** Since we iterate through `valid_chars` in alphabetical order and BFS explores level by level, the last valid subsequence we find at each length is automatically the lexicographically largest for that length.

### 3. **Greedy Subsequence Verification**

A critical mistake is using a greedy approach that doesn't properly reset when matching the pattern multiple times.

**Pitfall Example:**
```python
def incorrect_check(pattern: str, k: int) -> bool:
    matches = 0
    pattern_idx = 0
    for char in s:
        if pattern_idx < len(pattern) and char == pattern[pattern_idx]:
            pattern_idx += 1
            if pattern_idx == len(pattern):
                matches += 1
                # Forgot to reset pattern_idx!
    return matches >= k
```

**Correct Implementation:**
```python
def is_valid_subsequence(pattern: str, required_count: int) -> bool:
    pattern_index = 0
    for char in s:
        if char == pattern[pattern_index]:
            pattern_index += 1
            if pattern_index == len(pattern):
                required_count -= 1
                if required_count == 0:
                    return True
                pattern_index = 0  # Critical: Reset to match pattern again
    return False
```

### 4. **Memory/Time Limit Issues with Large Inputs**

BFS can explore an exponential number of subsequences. Without proper pruning, this can lead to TLE or MLE.

**Pitfall Example:**
```python
# Adding all possible subsequences without checking validity first
for char in valid_chars:
    next_subsequence = current + char
    queue.append(next_subsequence)  # Adding without validation
```

**Optimized Approach:**
```python
# Only add to queue if it's a valid k-repeated subsequence
for char in valid_chars:
    next_subsequence = current + char
    if is_valid_subsequence(next_subsequence, k):
        queue.append(next_subsequence)  # Prune invalid branches early
```

### 5. **Edge Case: Empty String Result**

Don't forget that the result could be an empty string if no valid subsequence exists.

**Pitfall Example:**
```python
# Assuming there's always a valid answer
return result[0]  # Could cause IndexError if result is empty
```

**Correct Handling:**
```python
# Initialize with empty string and return as-is if no valid subsequence found
result = ""
# ... BFS logic
return result  # Correctly returns "" if no valid subsequence exists
```

