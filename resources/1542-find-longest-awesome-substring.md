# 1542. Find Longest Awesome Substring

> Source: https://algo.monster/liteproblems/1542

## Problem Statement

You are given a string `s` consisting of digit characters ('0' through '9'). An **awesome substring** is a non-empty substring that can be rearranged (by swapping characters any number of times) to form a palindrome.

Your task is to find the length of the longest awesome substring in `s`.

A palindrome has a special property: all characters appear an even number of times, except for at most one character which can appear an odd number of times (this would be the middle character in odd-length palindromes).

For example:
- "3242133" is awesome because it can be rearranged to "3241423" which is a palindrome
- "12321" is already a palindrome, so it's awesome
- "1221" can form palindrome "2112" or "1221"

The key insight is that for a substring to be rearrangeable into a palindrome:
- At most one digit can appear an odd number of times
- All other digits must appear an even number of times

The solution uses bit manipulation to track the parity (odd/even count) of each digit. A 10-bit integer `st` represents the state where bit `i` is 1 if digit `i` appears an odd number of times in the current prefix, and 0 if it appears an even number of times.

For a substring `s[j..i]` to be awesome, the XOR of states at positions `i` and `j-1` should either be:
- 0 (all digits appear even times)
- A power of 2 (exactly one digit appears odd times)

## Intuition

The key observation is that a string can be rearranged into a palindrome if and only if at most one character appears an odd number of times. Why? In a palindrome, characters are mirrored around the center. For even-length palindromes, every character must have a matching pair. For odd-length palindromes, exactly one character sits in the middle without a pair.

Since we're dealing with digits 0-9, we can track the parity (odd/even count) of each digit using bits. Think of a 10-bit number where each bit represents whether a digit (0-9) appears an odd or even number of times. If bit `i` is 1, digit `i` appears odd times; if 0, even times.

As we traverse the string, we can maintain this bit state using XOR operations. When we encounter digit `d`, we flip the `d`-th bit using `st ^= (1 << d)`. This naturally tracks parity because XOR toggles bits - if a digit appeared even times (bit = 0) and we see it again, it becomes odd (bit = 1), and vice versa.

Now comes the clever part: for any substring `s[j..i]` to be awesome, we need to check if the digits in that substring have the right parity pattern (at most one odd). Instead of checking every possible substring directly, we can use prefix states. 

If we know the parity state at position `i` and position `j-1`, the XOR of these two states gives us the parity of the substring `s[j..i]`. Why? Because XOR cancels out the common prefix `s[0..j-1]`, leaving only the substring we care about.

For the substring to be awesome, this XOR result should either be:
- `0` (all digits appear even times - forms an even-length palindrome)
- A power of 2 like `1`, `10`, `100` in binary (exactly one digit appears odd times - forms an odd-length palindrome with that digit in the middle)

By storing the first occurrence of each state in a hash map, we can efficiently find the longest awesome substring ending at each position. We check if the current state was seen before (gives all-even substring) or if flipping any single bit of the current state was seen before (gives exactly-one-odd substring).

## Solution Approach

The implementation uses **state compression** with **prefix sum** technique to efficiently find the longest awesome substring.

**Data Structures Used:**
- `st`: An integer representing the current parity state (10 bits for digits 0-9)
- `d`: A dictionary/hash map storing the first occurrence index of each state
- `ans`: Variable to track the maximum length found

**Algorithm Steps:**

1. **Initialize the state tracking:**
   - Set `st = 0` (initially all digits have even count, i.e., count = 0)
   - Initialize dictionary `d = {0: -1}` to handle substrings starting from index 0
   - Set `ans = 1` as minimum awesome substring has length 1

2. **Process each character:**
   For each character at index `i`:
   - Convert character to digit: `v = int(c)`
   - Update state by flipping the bit for this digit: `st ^= (1 << v)`
 
3. **Check for all-even digit counts:**
   - If current state `st` exists in dictionary `d`, it means we've seen this exact state before at index `d[st]`
   - The substring between `d[st] + 1` and `i` has all digits appearing even times
   - Update answer: `ans = max(ans, i - d[st])`
   - If `st` is new, store it: `d[st] = i`

4. **Check for exactly-one-odd digit count:**
   - For each possible digit `v` from 0 to 9:
     - Calculate the state that differs by only bit `v`: `st ^ (1 << v)`
     - If this modified state exists in dictionary, the substring between those positions has exactly digit `v` appearing odd times
     - Update answer: `ans = max(ans, i - d[st ^ (1 << v)])`

**Why this works:**
- When `st` matches a previous state, the XOR between them is 0, meaning all digits in the substring have even counts
- When `st ^ (1 << v)` matches a previous state, the XOR between current state and that previous state is `(1 << v)`, meaning only digit `v` has odd count
- By checking all 10 possible single-bit differences, we cover all cases where exactly one digit appears odd times

**Time Complexity:** `O(n × 10)` where n is the length of string, as we check 10 possible digit flips for each character.

**Space Complexity:** `O(2^10)` in worst case for the dictionary, as there are at most `2^10 = 1024` possible states.

## Example Walkthrough

Let's trace through the string `s = "3242133"` to find the longest awesome substring.

**Initial Setup:**
- `st = 0` (binary: `0000000000` - all digits have even count)
- `d = {0: -1}` (state 0 seen at position -1)
- `ans = 1`

**Step-by-step processing:**

**i = 0, char = '3':**
- Digit `v = 3`
- Update state: `st = 0 ^ (1 << 3) = 8` (binary: `0000001000`)
- Check if `st = 8` in `d`: No
- Check single-bit flips:
  - For digit 3: `8 ^ 8 = 0` is in `d` at position -1
  - Length = `0 - (-1) = 1`
  - Update `ans = max(1, 1) = 1`
- Add to dictionary: `d[8] = 0`

**i = 1, char = '2':**
- Digit `v = 2`
- Update state: `st = 8 ^ (1 << 2) = 12` (binary: `0000001100`)
- Check if `st = 12` in `d`: No
- Check single-bit flips:
  - For digit 2: `12 ^ 4 = 8` is in `d` at position 0
  - Length = `1 - 0 = 1`
  - For digit 3: `12 ^ 8 = 4` not in `d`
- Add to dictionary: `d[12] = 1`

**i = 2, char = '4':**
- Digit `v = 4`
- Update state: `st = 12 ^ (1 << 4) = 28` (binary: `0000011100`)
- Check if `st = 28` in `d`: No
- Check single-bit flips:
  - For digit 2: `28 ^ 4 = 24` not in `d`
  - For digit 3: `28 ^ 8 = 20` not in `d`
  - For digit 4: `28 ^ 16 = 12` is in `d` at position 1
  - Length = `2 - 1 = 1`
- Add to dictionary: `d[28] = 2`

**i = 3, char = '2':**
- Digit `v = 2`
- Update state: `st = 28 ^ (1 << 2) = 24` (binary: `0000011000`)
- Check if `st = 24` in `d`: No
- Check single-bit flips:
  - For digit 3: `24 ^ 8 = 16` not in `d`
  - For digit 4: `24 ^ 16 = 8` is in `d` at position 0
  - Length = `3 - 0 = 3` 
  - Update `ans = max(1, 3) = 3`
- Add to dictionary: `d[24] = 3`

**i = 4, char = '1':**
- Digit `v = 1`
- Update state: `st = 24 ^ (1 << 1) = 26` (binary: `0000011010`)
- Check if `st = 26` in `d`: No
- Check single-bit flips (only showing matches):
  - No matches that improve `ans`
- Add to dictionary: `d[26] = 4`

**i = 5, char = '3':**
- Digit `v = 3`
- Update state: `st = 26 ^ (1 << 3) = 18` (binary: `0000010010`)
- Check if `st = 18` in `d`: No
- Check single-bit flips:
  - For digit 1: `18 ^ 2 = 16` not in `d`
  - For digit 4: `18 ^ 16 = 2` not in `d`
- Add to dictionary: `d[18] = 5`

**i = 6, char = '3':**
- Digit `v = 3`
- Update state: `st = 18 ^ (1 << 3) = 26` (binary: `0000011010`)
- Check if `st = 26` in `d`: Yes, at position 4!
- Length = `6 - 4 = 2`
- Check single-bit flips:
  - For digit 1: `26 ^ 2 = 24` is in `d` at position 3
  - Length = `6 - 3 = 3`
  - For digit 3: `26 ^ 8 = 18` is in `d` at position 5
  - Length = `6 - 5 = 1`
  - For digit 4: `26 ^ 16 = 10` not in `d`

**Final Result:** `ans = 3`

The longest awesome substring is "242" (from index 1 to 3), which can be rearranged to form palindrome "424".

## Python Solution

```python
class Solution:
    def longestAwesome(self, s: str) -> int:
        # Bitmask to track parity (odd/even count) of each digit 0-9
        # Bit i is 1 if digit i appears odd number of times, 0 if even
        parity_mask = 0
      
        # Dictionary to store first occurrence index of each parity state
        # Key: parity mask, Value: earliest index where this mask occurred
        first_occurrence = {0: -1}  # Empty prefix has mask 0 at index -1
      
        # Initialize result with minimum possible length
        max_length = 1
      
        # Process each character in the string
        for current_index, char in enumerate(s):
            # Convert character to digit
            digit = int(char)
          
            # Toggle the bit for this digit in the parity mask
            # XOR flips the bit: 0->1 (even->odd) or 1->0 (odd->even)
            parity_mask ^= 1 << digit
          
            # Case 1: Check if we've seen this exact parity mask before
            # If yes, substring between these positions has all even counts (palindrome)
            if parity_mask in first_occurrence:
                max_length = max(max_length, current_index - first_occurrence[parity_mask])
            else:
                # Record first occurrence of this parity mask
                first_occurrence[parity_mask] = current_index
          
            # Case 2: Check for palindromes with exactly one odd-count digit
            # Try flipping each bit to see if that state was seen before
            for digit_to_flip in range(10):
                # Create mask with one bit different (one digit has different parity)
                target_mask = parity_mask ^ (1 << digit_to_flip)
              
                # If this mask exists, substring between positions forms palindrome
                # with exactly one odd-count digit
                if target_mask in first_occurrence:
                    max_length = max(max_length, current_index - first_occurrence[target_mask])
      
        return max_length
```

## Time and Space Complexity

The time complexity is `O(n × C)`, where `n` is the length of the string `s` and `C` is the number of types of digit characters (which is 10 for digits 0-9).

- The outer loop iterates through each character in the string, contributing `O(n)`.
- For each character, we perform XOR operations and dictionary lookups in `O(1)` time.
- The inner loop iterates through all 10 possible digits (0-9), contributing `O(C)` where `C = 10`.
- Therefore, the overall time complexity is `O(n × 10) = O(n × C)`.

The space complexity is `O(2^C)`, where `C` is the number of types of digit characters.

- The dictionary `d` stores states represented by bitmasks.
- Each digit (0-9) can be either present an odd or even number of times, represented by a bit being 1 or 0.
- With 10 different digits, there are `2^10 = 1024` possible states that can be stored in the dictionary.
- Therefore, the space complexity is `O(2^10) = O(2^C)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Forgetting to Initialize the Empty Prefix State**

**Pitfall:** A common mistake is forgetting to initialize `first_occurrence[0] = -1`. Without this initialization, substrings starting from index 0 won't be properly handled.

**Example of the bug:**
```python
# WRONG: Missing initialization
first_occurrence = {}  # Should be {0: -1}
```

**Why it matters:** When the entire prefix from index 0 to i forms a valid palindrome (all even counts), we need to calculate the length as `i - (-1) = i + 1`. Without the initial state, we'd miss these valid substrings.

**Solution:** Always initialize with `first_occurrence = {0: -1}` to handle substrings starting from the beginning.

### 2. **Updating Dictionary Before Checking**

**Pitfall:** Adding the current state to the dictionary before checking if it already exists will always find the current index, resulting in zero-length substrings.

**Example of the bug:**
```python
# WRONG: Update before check
parity_mask ^= 1 << digit
first_occurrence[parity_mask] = current_index  # Don't do this first!
if parity_mask in first_occurrence:  # This will always be True now
    max_length = max(max_length, current_index - first_occurrence[parity_mask])  # Always 0!
```

**Solution:** Always check for existing states first, then only update if it's a new state:
```python
if parity_mask in first_occurrence:
    max_length = max(max_length, current_index - first_occurrence[parity_mask])
else:
    first_occurrence[parity_mask] = current_index
```

### 3. **Overwriting Previous Occurrences**

**Pitfall:** Updating `first_occurrence[parity_mask]` every time we see a state, instead of keeping only the first occurrence.

**Example of the bug:**
```python
# WRONG: Always updating
first_occurrence[parity_mask] = current_index  # This overwrites the first occurrence
```

**Why it matters:** We want the longest substring, so we need the earliest occurrence of each state. Overwriting with later occurrences would give us shorter substrings.

**Solution:** Only store the first occurrence of each state using the conditional check shown in pitfall #2.

### 4. **Not Checking Single-Character Substrings**

**Pitfall:** Initializing `max_length = 0` instead of `max_length = 1`.

**Why it matters:** Every single character is a palindrome by itself, so the minimum answer should be 1 (unless the string is empty, which typically isn't a test case).

**Solution:** Initialize with `max_length = 1` to account for single-character palindromes.

