# 2953. Count Complete Substrings

> Source: https://algo.monster/liteproblems/2953

## Problem Statement

You are given a string `word` and an integer `k`.

A substring is considered **complete** if it satisfies two conditions:

1. **Equal frequency condition**: Every character that appears in the substring must occur exactly `k` times.

2. **Adjacent character condition**: For any two adjacent characters in the substring, their positions in the alphabet must differ by at most 2. In other words, if you have two adjacent characters `c1` and `c2`, then `|ord(c1) - ord(c2)| ≤ 2`.

Your task is to count how many complete substrings exist in the given string `word`.

For example, if we have adjacent characters 'a' and 'c', their difference is `|97 - 99| = 2`, which satisfies the condition. However, 'a' and 'd' would have a difference of `|97 - 100| = 3`, which violates the condition.

## Intuition

The key insight is recognizing that the two conditions create natural boundaries in our string. The second condition (adjacent characters must differ by at most 2) acts as a "breaking point" - whenever we encounter two adjacent characters that differ by more than 2, we know that no complete substring can span across this boundary.

This observation leads us to split the original string into segments where all adjacent characters satisfy the alphabet distance constraint. For example, if we have "abcfgh", we'd split it at 'c' and 'f' since `|ord('c') - ord('f')| = 3 > 2`.

Once we have these segments, we need to find all substrings within each segment where every character appears exactly `k` times. But how do we efficiently check this?

Here's where the mathematical insight comes in: if a substring has `i` unique characters and each must appear exactly `k` times, then the substring must have exactly `i × k` characters. This gives us fixed-size windows to check!

For instance, if `k = 2`:
- A substring with 1 unique character must have length 2 (like "aa")
- A substring with 2 unique characters must have length 4 (like "aabb")
- A substring with 3 unique characters must have length 6 (like "aabbcc")

Since we're dealing with lowercase English letters, we have at most 26 unique characters, so we only need to check window sizes from `k` to `26k`.

The [sliding window](/problems/sliding_window_maximum) technique becomes natural here - for each possible window size `l = i × k`, we slide a window of that size across the segment. We maintain:
- `cnt`: frequency of each character in the current window
- `freq`: frequency of frequencies (how many characters appear exactly `j` times)

When `freq[k] == i`, it means we have exactly `i` characters, each appearing exactly `k` times - a complete substring!

This approach elegantly combines the problem's constraints with fixed-size sliding windows, making it efficient to count all complete substrings.

**Learn more about [Sliding Window](/problems/sliding_window_maximum) patterns.**

## Solution Approach

The implementation consists of two main parts: splitting the string into valid segments and counting complete substrings within each segment.

**Step 1: Splitting the String into Valid Segments**

We use two pointers `i` and `j` to identify segments where all adjacent characters differ by at most 2:

```python
while i < n:
    j = i + 1
    while j < n and abs(ord(word[j]) - ord(word[j - 1])) <= 2:
        j += 1
    ans += f(word[i:j])
    i = j
```

Starting from position `i`, we extend `j` as long as the adjacent character condition is satisfied. Once we find a breaking point, we process the segment `word[i:j]` and move to the next segment.

**Step 2: Counting Complete Substrings in Each Segment**

The helper function `f(s)` counts complete substrings within a segment `s`. For each possible number of unique characters `i` (from 1 to 26):

1. **Calculate window size**: `l = i * k` (since we need `i` characters, each appearing `k` times)

2. **Initialize the first window**: Create a frequency counter `cnt` for the first `l` characters, and a frequency-of-frequencies counter `freq` that tracks how many characters appear exactly `j` times.

3. **Check initial window**: If `freq[k] == i`, we have found a complete substring (all `i` characters appear exactly `k` times).

4. **Slide the window**: For each new position `j` from `l` to `m-1`:
   - **Add new character** `s[j]`:
     ```python
     freq[cnt[s[j]]] -= 1  # Remove old frequency count
     cnt[s[j]] += 1         # Increment character count
     freq[cnt[s[j]]] += 1   # Add new frequency count
     ```
 
   - **Remove old character** `s[j-l]`:
     ```python
     freq[cnt[s[j-l]]] -= 1  # Remove old frequency count
     cnt[s[j-l]] -= 1        # Decrement character count
     freq[cnt[s[j-l]]] += 1  # Add new frequency count
     ```
 
   - **Check condition**: If `freq[k] == i`, increment the answer.

**Key Data Structures:**

- `cnt`: A `Counter` that maintains character frequencies in the current window
- `freq`: A `Counter` that maintains how many characters have each frequency value

**Why This Works:**

The `freq` counter is the key optimization. Instead of checking if all characters in the window appear exactly `k` times (which would require iterating through `cnt`), we maintain a count of frequencies. When `freq[k] == i`, it means exactly `i` characters have frequency `k`, and since our window has size `i * k`, these must be the only characters in the window.

**Time Complexity:** 

- For each segment, we iterate through at most 26 different window sizes
- For each window size, we slide through the segment once
- Overall: `O(26 * n) = O(n)` where `n` is the length of the string

## Example Walkthrough

Let's walk through a concrete example with `word = "ababcd"` and `k = 2`.

**Step 1: Split into valid segments**

We check adjacent characters:
- 'a' and 'b': `|97 - 98| = 1 ≤ 2` ✓
- 'b' and 'a': `|98 - 97| = 1 ≤ 2` ✓
- 'a' and 'b': `|97 - 98| = 1 ≤ 2` ✓
- 'b' and 'c': `|98 - 99| = 1 ≤ 2` ✓
- 'c' and 'd': `|99 - 100| = 1 ≤ 2` ✓

All adjacent characters satisfy the condition, so we have one segment: `"ababcd"`.

**Step 2: Count complete substrings in segment "ababcd"**

We check different window sizes based on the number of unique characters:

**For i = 1 (window size = 1 × 2 = 2):**
- Window `"ab"`: cnt = {'a':1, 'b':1}, freq = {1:2}, freq[2] = 0 ≠ 1 ✗
- Window `"ba"`: cnt = {'b':1, 'a':1}, freq = {1:2}, freq[2] = 0 ≠ 1 ✗
- Window `"ab"`: cnt = {'a':1, 'b':1}, freq = {1:2}, freq[2] = 0 ≠ 1 ✗
- Window `"bc"`: cnt = {'b':1, 'c':1}, freq = {1:2}, freq[2] = 0 ≠ 1 ✗
- Window `"cd"`: cnt = {'c':1, 'd':1}, freq = {1:2}, freq[2] = 0 ≠ 1 ✗

No complete substrings of size 2.

**For i = 2 (window size = 2 × 2 = 4):**
- Window `"abab"`: cnt = {'a':2, 'b':2}, freq = {2:2}, freq[2] = 2 ✓ (Found one!)
- Window `"babc"`: cnt = {'b':2, 'a':1, 'c':1}, freq = {1:2, 2:1}, freq[2] = 1 ≠ 2 ✗
- Window `"abcd"`: cnt = {'a':1, 'b':1, 'c':1, 'd':1}, freq = {1:4}, freq[2] = 0 ≠ 2 ✗

Found 1 complete substring: `"abab"`.

**For i = 3 (window size = 3 × 2 = 6):**
- Window `"ababcd"`: cnt = {'a':2, 'b':2, 'c':1, 'd':1}, freq = {1:2, 2:2}, freq[2] = 2 ≠ 3 ✗

No complete substrings of size 6.

**Total count: 1**

The only complete substring is `"abab"` where:
- Each character ('a' and 'b') appears exactly k=2 times
- All adjacent characters differ by at most 2 (|'a'-'b'| = 1)

Let's trace through `word = "aabaab"` with `k = 3`.

**Phase 1: Segment identification**
Check each adjacent pair:
- 'a','a': |97-97| = 0 ≤ 2 ✓
- 'a','b': |97-98| = 1 ≤ 2 ✓
- 'b','a': |98-97| = 1 ≤ 2 ✓
- 'a','a': |97-97| = 0 ≤ 2 ✓
- 'a','b': |97-98| = 1 ≤ 2 ✓

Result: One segment `"aabaab"` (length 6)

**Phase 2: Window sliding for each possible unique character count**

*Window size for i=1: 1×3=3*
- Position 0-2: `"aab"` → cnt={'a':2,'b':1}, freq={1:1,2:1}, need freq[3]=1 but have 0 ✗
- Position 1-3: `"aba"` → cnt={'a':2,'b':1}, freq={1:1,2:1}, freq[3]=0 ✗
- Position 2-4: `"baa"` → cnt={'b':1,'a':2}, freq={1:1,2:1}, freq[3]=0 ✗
- Position 3-5: `"aab"` → cnt={'a':2,'b':1}, freq={1:1,2:1}, freq[3]=0 ✗

*Window size for i=2: 2×3=6*
- Position 0-5: `"aabaab"` → cnt={'a':4,'b':2}, freq={2:1,4:1}, need freq[3]=2 but have 0 ✗

**Result: 0 complete substrings**

None of the substrings satisfy both conditions - no character appears exactly 3 times in any valid window.

## Python Solution

```python
from collections import Counter

class Solution:
    def countCompleteSubstrings(self, word: str, k: int) -> int:
        def count_complete_in_segment(segment: str) -> int:
            """
            Count complete substrings in a segment where adjacent chars differ by at most 2.
            A complete substring has each character appearing exactly k times.
            """
            segment_length = len(segment)
            result = 0
          
            # Try all possible numbers of unique characters (1 to 26)
            for num_unique_chars in range(1, 27):
                # Calculate the required window size
                window_size = num_unique_chars * k
              
                # Skip if window size exceeds segment length
                if window_size > segment_length:
                    break
              
                # Initialize counters for the first window
                char_count = Counter(segment[:window_size])
                frequency_count = Counter(char_count.values())
              
                # Check if first window is complete (all chars appear exactly k times)
                result += frequency_count[k] == num_unique_chars
              
                # Slide the window through the rest of the segment
                for right in range(window_size, segment_length):
                    # Character entering the window
                    entering_char = segment[right]
                  
                    # Update frequency counter before changing char count
                    frequency_count[char_count[entering_char]] -= 1
                    char_count[entering_char] += 1
                    frequency_count[char_count[entering_char]] += 1
                  
                    # Character leaving the window
                    leaving_char = segment[right - window_size]
                  
                    # Update frequency counter before changing char count
                    frequency_count[char_count[leaving_char]] -= 1
                    char_count[leaving_char] -= 1
                    frequency_count[char_count[leaving_char]] += 1
                  
                    # Check if current window is complete
                    result += frequency_count[k] == num_unique_chars
                  
            return result
      
        # Main function logic
        total_count = 0
        string_length = len(word)
        start_index = 0
      
        # Split the string into segments where adjacent chars differ by at most 2
        while start_index < string_length:
            end_index = start_index + 1
          
            # Extend segment while adjacent characters differ by at most 2
            while end_index < string_length and abs(ord(word[end_index]) - ord(word[end_index - 1])) <= 2:
                end_index += 1
          
            # Count complete substrings in this segment
            total_count += count_complete_in_segment(word[start_index:end_index])
          
            # Move to next segment
            start_index = end_index
          
        return total_count
```

## Time and Space Complexity

**Time Complexity:** `O(n × |Σ|)` where `n` is the length of the string `word` and `|Σ| = 26` represents the size of the character set (lowercase English letters).

The analysis breaks down as follows:
- The outer while loop in the main function iterates through the string `word` once, partitioning it into segments where consecutive characters differ by at most 2. This takes `O(n)` time total.
- For each segment, the function `f(s)` is called, which:
  - Iterates through at most 26 possible values of `i` (representing 1 to 26 unique characters)
  - For each `i`, it uses a sliding window of size `l = i × k`
  - The sliding window traverses the segment once, performing `O(1)` operations per position (updating counters and frequency maps)
  - Each segment is processed in `O(m × 26)` where `m` is the segment length
- Since the sum of all segment lengths equals `n`, the total time across all segments is `O(n × 26) = O(n × |Σ|)`

**Space Complexity:** `O(|Σ|)` where `|Σ| = 26`.

The space usage consists of:
- `cnt`: A Counter that stores at most 26 different characters (lowercase letters), requiring `O(26)` space
- `freq`: A Counter that stores frequency counts, with at most 26 different frequency values, requiring `O(26)` space
- Other variables use `O(1)` space
- The recursive calls and string slicing `word[i:j]` create new strings, but at most one segment is processed at a time, and the space for the segment is bounded by `O(n)` in the worst case, but the dominant space usage for the algorithm's working memory is `O(|Σ|)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Frequency Counter Updates When Sliding Window**

**The Pitfall:** When sliding the window, developers often update the character count first and then the frequency counter, but forget that the order matters. A common mistake is:

```python
# WRONG: Updates char_count before updating freq counter
char_count[entering_char] += 1
frequency_count[char_count[entering_char]] += 1
frequency_count[char_count[entering_char] - 1] -= 1  # Wrong old frequency
```

**Why It's Wrong:** After incrementing `char_count[entering_char]`, you've lost the original frequency value. Trying to decrement the old frequency becomes error-prone.

**The Solution:** Always update the frequency counter BEFORE modifying the character counter:

```python
# CORRECT: Update frequency counter first
frequency_count[char_count[entering_char]] -= 1  # Remove old frequency
char_count[entering_char] += 1                   # Update character count
frequency_count[char_count[entering_char]] += 1  # Add new frequency
```

### 2. **Forgetting to Handle Zero Frequencies**

**The Pitfall:** When a character's count becomes 0 (it leaves the window completely), the frequency counter still tracks that there's a character with frequency 0. This can lead to incorrect validation:

```python
# This check might fail even when valid
if frequency_count[k] == num_unique_chars:  # frequency_count[0] might be > 0
```

**The Solution:** The current implementation handles this correctly by maintaining the frequency counter properly. When a character count becomes 0, `frequency_count[0]` increases, but this doesn't affect our check since we only care about `frequency_count[k]`.

### 3. **Off-by-One Error in Window Sliding**

**The Pitfall:** Incorrectly calculating which character is leaving the window:

```python
# WRONG: Off by one
leaving_char = segment[right - window_size + 1]  # Wrong index

# WRONG: Using wrong boundary
for right in range(window_size - 1, segment_length):  # Wrong starting point
```

**The Solution:** The leaving character should be at index `right - window_size`:

```python
# CORRECT
for right in range(window_size, segment_length):
    leaving_char = segment[right - window_size]  # Correct index
```

### 4. **Not Breaking Early When Window Size Exceeds Segment**

**The Pitfall:** Continuing to check larger window sizes even when they exceed the segment length, leading to unnecessary iterations:

```python
# INEFFICIENT: No early termination
for num_unique_chars in range(1, 27):
    window_size = num_unique_chars * k
    # Continues even when window_size > segment_length
```

**The Solution:** Break early when the window size becomes too large:

```python
# EFFICIENT: Early termination
if window_size > segment_length:
    break
```

### 5. **Misunderstanding the Complete Substring Condition**

**The Pitfall:** Checking if AT LEAST `num_unique_chars` characters have frequency `k`, rather than EXACTLY:

```python
# WRONG: Allows more than num_unique_chars distinct characters
if frequency_count[k] >= num_unique_chars:  # Wrong condition
```

**Why It's Wrong:** If we have a window of size `3k` (expecting 3 unique chars), but 4 different characters appear with some having frequency `k`, this would incorrectly count as valid.

**The Solution:** Use strict equality to ensure EXACTLY the expected number of unique characters:

```python
# CORRECT: Exactly num_unique_chars characters must have frequency k
if frequency_count[k] == num_unique_chars:
```

This works because if the window has size `num_unique_chars * k` and exactly `num_unique_chars` characters have frequency `k`, then these must be the only characters in the window (no room for other characters).

