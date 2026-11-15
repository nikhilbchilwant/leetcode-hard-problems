# 3088. Make String Anti-palindrome

> Source: https://algo.monster/liteproblems/3088

## Problem Statement

You are given a string `s` of **even** length `n`. An **anti-palindrome** is defined as a string where for every position `i` (where `0 <= i < n`), the character at position `i` is different from the character at its mirror position `n - i - 1`. In other words, `s[i] != s[n - i - 1]` for all valid indices.

Your task is to transform the given string `s` into an anti-palindrome by performing any number of swap operations (including zero operations). In each operation, you can select any two characters in the string and swap their positions.

The goal is to return the resulting anti-palindrome string. If multiple valid anti-palindrome strings can be formed, you must return the **lexicographically smallest** one. If it's impossible to create an anti-palindrome from the given string, return `"-1"`.

For example:
- If `s = "abba"`, after swapping to get `"abab"`, we have an anti-palindrome because `s[0] != s[3]` ('a' != 'b') and `s[1] != s[2]` ('b' != 'a').
- The string must satisfy that no character at position `i` equals the character at position `n - i - 1`.
- Among all possible anti-palindromes that can be formed, you need to find the one that comes first in dictionary order.

## Intuition

To find the lexicographically smallest anti-palindrome, we should start by [sorting](/problems/sorting_summary) the string. This ensures we're working with the smallest possible arrangement from the beginning.

The key insight is that after [sorting](/problems/sorting_summary), we only need to focus on the middle portion of the string. Why? Because in a sorted string, if it's already an anti-palindrome, we're done. If not, the problematic pairs (where `s[i] == s[n-i-1]`) will be concentrated around the middle.

Consider a sorted string like `"aabbccdd"`. The potential issue occurs when the character at position `i` equals the character at position `n-i-1`. In a sorted string, this happens when we have too many of the same character clustered in the middle region.

The critical observation is that we only need to check if `s[m] == s[m-1]` (where `m = n/2`). If these middle characters are different, the string is already an anti-palindrome because:
- The left half is sorted in non-decreasing order
- The right half (when viewed from the mirror positions) is in non-increasing order
- If the two middle elements are different, all mirror pairs will be different

When `s[m] == s[m-1]`, we have a run of identical characters crossing the midpoint. We need to break this symmetry by swapping some characters from the right half with different characters. We look for the first different character `s[i]` in the right half and swap it with problematic positions. This maintains lexicographical order as much as possible while ensuring the anti-palindrome property.

If we can't find enough different characters to swap (meaning most or all characters in the string are the same), it's impossible to form an anti-palindrome, and we return `"-1"`.

**Learn more about [Greedy](/problems/greedy_intro) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

The implementation follows a [greedy](/problems/greedy_intro) approach combined with [sorting](/problems/sorting_summary):

1. **Sort the string**: Convert the input string to a sorted character array `cs = sorted(s)`. This gives us the lexicographically smallest starting point.

2. **Calculate middle position**: Find the middle index `m = n // 2` where `n` is the length of the string.

3. **Check if adjustment is needed**: Compare `cs[m]` with `cs[m-1]`. If they're different, the string is already an anti-palindrome and we can return the joined result.

4. **Handle the problematic case** when `cs[m] == cs[m-1]`:
   - Initialize pointer `i = m` to find the first different character in the right half
   - Move `i` forward while `cs[i] == cs[i-1]` to skip over identical characters
   - Initialize pointer `j = m` to track positions that need swapping

5. **Perform swaps to fix violations**:
   - While `j < n` and `cs[j] == cs[n-j-1]` (violation of anti-palindrome property):
     - If `i >= n`, no more different characters are available, return `"-1"`
     - Swap `cs[i]` with `cs[j]` to fix the violation
     - Increment both `i` and `j` to process the next position

6. **Return the result**: Join the character array back into a string and return it.

The algorithm ensures that we make minimal changes to maintain lexicographical order while satisfying the anti-palindrome constraint. The swapping strategy specifically targets positions where the anti-palindrome property is violated (`cs[j] == cs[n-j-1]`) and fixes them using different characters from the right half of the sorted string.

The time complexity is `O(n log n)` due to [sorting](/problems/sorting_summary), and the space complexity is `O(n)` for storing the sorted character array.

## Example Walkthrough

Let's walk through the solution with `s = "aabbcc"` (n = 6).

**Step 1: Sort the string**
- `cs = ['a', 'a', 'b', 'b', 'c', 'c']`

**Step 2: Calculate middle position**
- `m = 6 // 2 = 3`

**Step 3: Check if adjustment needed**
- Compare `cs[3]` with `cs[2]`: 'b' == 'b', so adjustment is needed

**Step 4: Find first different character**
- Initialize `i = 3` (at 'b')
- Move `i` forward while `cs[i] == cs[i-1]`:
  - `cs[3] == cs[2]`? 'b' == 'b'? Yes, so `i = 4`
  - `cs[4] == cs[3]`? 'c' == 'b'? No, stop
- Now `i = 4` (pointing to first 'c')

**Step 5: Fix violations**
- Initialize `j = 3`
- Check positions that violate anti-palindrome:

  **First iteration:**
  - `j = 3`, check if `cs[3] == cs[6-3-1]` → `cs[3] == cs[2]` → 'b' == 'b'? Yes (violation!)
  - Swap `cs[4]` with `cs[3]`: ['a', 'a', 'b', 'c', 'b', 'c']
  - Increment: `i = 5`, `j = 4`

  **Second iteration:**
  - `j = 4`, check if `cs[4] == cs[6-4-1]` → `cs[4] == cs[1]` → 'b' == 'a'? No (no violation)
  - No more violations, exit loop

**Step 6: Verify the result**
- Final string: "aabcbc"
- Check anti-palindrome property:
  - Position 0 and 5: 'a' != 'c' ✓
  - Position 1 and 4: 'a' != 'b' ✓
  - Position 2 and 3: 'b' != 'c' ✓
- All positions satisfy `s[i] != s[n-i-1]`

The result "aabcbc" is the lexicographically smallest anti-palindrome we can form from "aabbcc".

## Python Solution

```python
class Solution:
    def makeAntiPalindrome(self, s: str) -> str:
        # Convert string to sorted list of characters
        char_list = sorted(s)
        n = len(char_list)
        mid_index = n // 2

        # Check if the middle elements are the same (potential palindrome issue)
        if char_list[mid_index] == char_list[mid_index - 1]:
            # Find the end of the sequence of duplicate characters starting from middle
            duplicate_end = mid_index
            while duplicate_end < n and char_list[duplicate_end] == char_list[duplicate_end - 1]:
                duplicate_end += 1

            # Start swapping from middle position to break palindrome pattern
            swap_position = mid_index
            while swap_position < n and char_list[swap_position] == char_list[n - swap_position - 1]:
                # If we've exhausted all characters to swap with, no solution exists
                if duplicate_end >= n:
                    return "-1"

                # Swap characters to break the palindrome pattern
                char_list[duplicate_end], char_list[swap_position] = char_list[swap_position], char_list[duplicate_end]

                # Move both pointers forward
                duplicate_end += 1
                swap_position += 1

        # Convert list back to string and return
        return "".join(char_list)
```

## Time and Space Complexity

The time complexity of this algorithm is `O(n × log n)`, where `n` is the length of the string `s`. This is dominated by the sorting operation `sorted(s)` at the beginning of the function, which uses a comparison-based sorting algorithm (typically Timsort in Python) that has a time complexity of `O(n × log n)`. The subsequent operations include:
- The while loops that iterate through portions of the array, which take `O(n)` time in the worst case
- Character swapping operations that take `O(1)` time each
- The final join operation that takes `O(n)` time

Since `O(n × log n) + O(n) = O(n × log n)`, the overall time complexity is `O(n × log n)`.

The space complexity is `O(n)`, where `n` is the length of the string `s`. This is because:
- The `sorted(s)` operation creates a new list of characters that requires `O(n)` space
- The variable `cs` stores this sorted list of `n` characters
- Other variables (`i`, `j`, `m`) use only `O(1)` additional space
- The final `"".join(cs)` operation creates a new string of length `n`, but this is the return value

Therefore, the total space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Misunderstanding When Swapping is Necessary**

A critical pitfall is not recognizing that swapping is only needed when we have a "run" of identical characters crossing the middle boundary that creates palindromic pairs. The condition `char_list[mid_index] == char_list[mid_index - 1]` alone doesn't guarantee we need swaps.

**Problem Example:**
```python
s = "aabbccdd"  # After sorting: "aabbccdd"
# mid_index = 4
# char_list[3] = 'b', char_list[4] = 'c' (different)
# But we still need to check if char_list[i] == char_list[n-i-1] for all positions
```

In this case, even though the middle elements differ, we have:
- Position 0 ('a') vs Position 7 ('d') - OK
- Position 1 ('a') vs Position 6 ('d') - OK
- Position 2 ('b') vs Position 5 ('c') - OK
- Position 3 ('b') vs Position 4 ('c') - OK

The code would work, but the logic can be misleading.

### 2. **Incorrect Duplicate End Finding**

The loop to find `duplicate_end` can go out of bounds or miss edge cases:

```python
# Current problematic code:
duplicate_end = mid_index
while duplicate_end < n and char_list[duplicate_end] == char_list[duplicate_end - 1]:
    duplicate_end += 1
```

**Issue:** This finds the end of duplicates starting from `mid_index`, but what if we need characters that are different from `char_list[mid_index-1]`, not just the next different character in sequence?

**Example:**
```python
s = "aaaabbbb"  # After sorting: "aaaabbbb"
# We need to swap some 'b's into the first half to break the palindrome
# But duplicate_end will point to index 4 (first 'b'), when we actually need it
```

### 3. **Insufficient Validation for Impossible Cases**

The current code only returns "-1" when `duplicate_end >= n` during swapping, but there are other impossible cases:

**Example:**
```python
s = "aaaa"  # All same characters
# After sorting: "aaaa"
# No matter how we arrange, s[0] will always equal s[3], s[1] will equal s[2]
```

### **Improved Solution:**

```python
class Solution:
    def makeAntiPalindrome(self, s: str) -> str:
        char_list = sorted(s)
        n = len(char_list)

        # First, check if anti-palindrome is possible
        char_count = {}
        for c in char_list:
            char_count[c] = char_count.get(c, 0) + 1

        # If any character appears more than n/2 times, impossible
        for count in char_count.values():
            if count > n // 2:
                return "-1"

        # Now arrange to ensure anti-palindrome property
        mid = n // 2

        # Check and fix violations from the middle outward
        for i in range(mid):
            if char_list[i] == char_list[n - 1 - i]:
                # Find a different character to swap with
                for j in range(i + 1, n - i - 1):
                    if char_list[j] != char_list[i] and char_list[j] != char_list[n - 1 - i]:
                        # Check if swapping maintains anti-palindrome at position j
                        mirror_j = n - 1 - j
                        if j < mid and char_list[i] != char_list[mirror_j]:
                            char_list[i], char_list[j] = char_list[j], char_list[i]
                            break
                        elif j >= mid and char_list[i] != char_list[mirror_j]:
                            char_list[n - 1 - i], char_list[j] = char_list[j], char_list[n - 1 - i]
                            break

        # Final verification
        for i in range(mid):
            if char_list[i] == char_list[n - 1 - i]:
                return "-1"

        return "".join(char_list)
```

This improved solution:
1. Pre-validates if an anti-palindrome is possible by checking character frequencies
2. Systematically checks each position for violations
3. Ensures swaps don't create new violations
4. Performs final verification before returning the result

