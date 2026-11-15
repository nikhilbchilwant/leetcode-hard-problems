# 2193. Minimum Number of Moves to Make Palindrome

> Source: https://algo.monster/liteproblems/2193

## Problem Statement

You have a string `s` that contains only lowercase English letters. Your goal is to transform this string into a palindrome using the minimum number of moves possible.

A **move** consists of selecting any two **adjacent** characters in the string and swapping their positions. For example, in the string "abc", you could swap 'a' and 'b' to get "bac", or swap 'b' and 'c' to get "acb".

A palindrome is a string that reads the same forwards and backwards. Examples include "aba", "racecar", or "aa".

The problem guarantees that the input string can always be converted into a palindrome, meaning the character frequencies allow for palindrome formation (at most one character can have an odd frequency).

You need to find and return the minimum number of adjacent swaps required to rearrange the string into any valid palindrome.

For example:
- If `s = "aabb"`, one possible palindrome is "abba", which requires 2 moves: "aabb" → "abab" → "abba"
- If `s = "letelt"`, it can be rearranged to "lettel" with the minimum number of adjacent swaps

## Intuition

To build a palindrome, we need to ensure that characters are mirrored around the center. The key insight is that we can work from the outside in, fixing the palindrome structure layer by layer.

Think about how a palindrome looks: the first character must match the last character, the second must match the second-to-last, and so on. This naturally suggests a two-pointer approach where we try to match characters from both ends.

Starting with pointers at positions `i` (left) and `j` (right), we want to make `s[i] == s[j]`. Since we can only swap adjacent characters, we need to find a character that matches `s[i]` somewhere between positions `i+1` to `j`, then bubble it to position `j` through adjacent swaps.

Why search from right to left (from `j` down to `i`)? Because we want to find the matching character that's closest to position `j`. This minimizes the number of swaps needed to bring it to the correct position. Once we find the match at position `k`, we perform `j - k` adjacent swaps to move it to position `j`.

After successfully pairing characters at positions `i` and `j`, we move our pointers inward (`i++` and `j--`) and repeat the process for the next layer of the palindrome.

The special case occurs when we can't find a match for `s[i]` in the range. This only happens when `s[i]` is the odd-frequency character that belongs in the center of the palindrome. In this case, we calculate how many swaps are needed to eventually move it to the center position (`n // 2 - i` swaps) and continue processing other characters.

This [greedy](/problems/greedy_intro) approach works because each character pairing is independent - once we've correctly positioned characters at the outer positions, they don't need to be moved again when we process inner positions.

**Learn more about [Greedy](/problems/greedy_intro) and [Two Pointers](/problems/two_pointers_intro) patterns.**

## Solution Approach

The implementation uses a [greedy](/problems/greedy_intro) two-pointer approach with character list manipulation:

1. **Convert string to list**: We first convert the string `s` to a list `cs` since strings are immutable in Python and we need to perform swaps.

2. **Initialize variables**: 
   - `ans` tracks the total number of swaps
   - `n` stores the length of the string
   - `i` and `j` are pointers starting at `0` and `n-1` respectively

3. **Main loop** (`while i < j`):
   - For each character at position `i`, we search for its matching character from position `j` down to `i+1`
   - We use a flag `even` to track whether we found a match

4. **Finding and swapping the match**:
   - The inner loop `for k in range(j, i, -1)` searches from right to left
   - When we find `cs[i] == cs[k]`, we've found our match
   - We then bubble this character from position `k` to position `j` using adjacent swaps:
     ```python
     while k < j:
         cs[k], cs[k + 1] = cs[k + 1], cs[k]
         k += 1
         ans += 1
     ```
   - Each swap increments our answer counter
   - After successfully positioning the match, we decrement `j` since that position is now fixed

5. **Handling the odd character**:
   - If no match is found (`even` remains `False`), this means `cs[i]` is the odd-frequency character
   - This character needs to eventually reach the center position at index `n // 2`
   - We add `n // 2 - i` to our answer, which represents the number of swaps needed to move it from position `i` to the center
   - We don't actually perform these swaps now; we just count them and continue processing

6. **Move to next character**: After processing position `i`, we increment it to process the next character

The algorithm continues until `i >= j`, meaning we've processed all character pairs needed to form the palindrome. The total number of swaps accumulated in `ans` is returned as the minimum number of moves needed.

Time complexity: `O(n²)` where n is the length of the string, as we potentially scan through the remaining string for each character.
Space complexity: `O(n)` for the character list.

## Example Walkthrough

Let's walk through the algorithm with `s = "aabb"` to transform it into a palindrome.

**Initial Setup:**
- Convert string to list: `cs = ['a', 'a', 'b', 'b']`
- `ans = 0`, `n = 4`, `i = 0`, `j = 3`

**Step 1: Match cs[0] = 'a' with right side**
- `i = 0, j = 3`
- Search from `j=3` down to find a match for `cs[0] = 'a'`
- Check `cs[3] = 'b'` - no match
- Check `cs[2] = 'b'` - no match  
- Check `cs[1] = 'a'` - **match found!** at `k = 1`
- Swap `cs[1]` to position `j=3`:
  - Swap `cs[1]` with `cs[2]`: `['a', 'b', 'a', 'b']`, `ans = 1`
  - Swap `cs[2]` with `cs[3]`: `['a', 'b', 'b', 'a']`, `ans = 2`
- Decrement `j = 2`, increment `i = 1`

**Step 2: Match cs[1] = 'b' with right side**
- `i = 1, j = 2`
- Search from `j=2` down to find a match for `cs[1] = 'b'`
- Check `cs[2] = 'b'` - **match found!** at `k = 2`
- Already in correct position (no swaps needed)
- Decrement `j = 1`, increment `i = 2`

**Step 3: Termination**
- Now `i = 2` and `j = 1`, so `i > j`
- Loop terminates

**Result:**
- Final string: `['a', 'b', 'b', 'a']` which is "abba" (a palindrome)
- Total swaps: `ans = 2`

The algorithm successfully transformed "aabb" into the palindrome "abba" using 2 adjacent swaps, which is the minimum number possible.

## Python Solution

```python
class Solution:
    def minMovesToMakePalindrome(self, s: str) -> int:
        # Convert string to list for in-place modifications
        char_list = list(s)
        total_moves = 0
        string_length = len(s)
      
        # Use two pointers from both ends
        left_ptr = 0
        right_ptr = string_length - 1
      
        while left_ptr < right_ptr:
            # Flag to check if we found a matching character
            found_match = False
          
            # Search from right pointer backwards to find match for left character
            for search_idx in range(right_ptr, left_ptr, -1):
                if char_list[left_ptr] == char_list[search_idx]:
                    found_match = True
                  
                    # Bubble the matching character to the right pointer position
                    while search_idx < right_ptr:
                        # Swap adjacent characters
                        char_list[search_idx], char_list[search_idx + 1] = char_list[search_idx + 1], char_list[search_idx]
                        search_idx += 1
                        total_moves += 1
                  
                    # Move right pointer inward after placing the character
                    right_ptr -= 1
                    break
          
            # If no match found, this character must be the odd one (for odd-length palindromes)
            # Move it to the center position
            if not found_match:
                total_moves += string_length // 2 - left_ptr
          
            # Move left pointer forward
            left_ptr += 1
      
        return total_moves
```

## Time and Space Complexity

**Time Complexity:** `O(n²)`

The algorithm uses a two-pointer approach with nested loops:
- The outer while loop runs from `i = 0` to `i < j`, which iterates approximately `n/2` times where `n` is the length of the string
- For each iteration of the outer loop, there's an inner for loop that searches from position `j` down to `i+1` to find a matching character, which can take up to `O(n)` iterations in the worst case
- When a match is found, there's another while loop that performs swaps to move the character from position `k` to position `j`, which can perform up to `O(n)` swaps in the worst case
- The operations inside the loops (comparisons, swaps, increments) are all `O(1)`

Overall: The outer loop runs `O(n)` times, and for each iteration, the inner operations can take `O(n)` time, resulting in `O(n) × O(n) = O(n²)` time complexity.

**Space Complexity:** `O(n)`

- The algorithm creates a list `cs` from the input string `s`, which requires `O(n)` additional space
- Other variables (`ans`, `i`, `j`, `k`, `even`) use constant space `O(1)`
- No recursive calls or additional data structures are used

Therefore, the total space complexity is `O(n)` due to the list conversion of the input string.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Handling of the Odd Character**

**Pitfall**: A common mistake is immediately moving the odd-frequency character to the center position when it's encountered, which disrupts the remaining string structure and leads to incorrect swap counts.

**Why it happens**: When we encounter a character at position `i` that has no match from the right side, the intuitive response might be to immediately swap it to the center. However, doing this would shift all characters between position `i` and the center, making subsequent matching operations incorrect.

**Example of the mistake**:
```python
# WRONG APPROACH
if not found_match:
    # Don't actually move the character now!
    center = string_length // 2
    while left_ptr < center:
        char_list[left_ptr], char_list[left_ptr + 1] = char_list[left_ptr + 1], char_list[left_ptr]
        left_ptr += 1
        total_moves += 1
```

**Correct approach**: Only count the moves needed (`string_length // 2 - left_ptr`) without actually performing them. The odd character will naturally end up in the center after all other pairs are matched.

### 2. **Off-by-One Error in the Search Loop**

**Pitfall**: Using incorrect loop boundaries when searching for the matching character, such as `range(right_ptr, left_ptr - 1, -1)` or `range(right_ptr + 1, left_ptr, -1)`.

**Why it happens**: The search should start from `right_ptr` and go down to (but not including) `left_ptr`. The confusion arises from Python's range behavior and the need to avoid comparing a character with itself.

**Correct boundary**: `range(right_ptr, left_ptr, -1)` ensures we:
- Start searching from position `right_ptr`
- Stop before reaching position `left_ptr` 
- Never compare `char_list[left_ptr]` with itself

### 3. **Forgetting to Update Pointers After Operations**

**Pitfall**: After finding and swapping a match, forgetting to decrement `right_ptr`, or always incrementing `left_ptr` even when handling the odd character.

**Why it happens**: The algorithm has different behaviors for matched pairs versus the odd character, and it's easy to apply the wrong pointer update logic.

**Correct pointer management**:
```python
if found_match:
    # After bubbling match to position right_ptr
    right_ptr -= 1  # This position is now fixed
  
# Always move left_ptr forward regardless of match
left_ptr += 1
```

### 4. **Using the Original String Instead of the Modified List**

**Pitfall**: Attempting to perform comparisons or swaps on the original string `s` instead of the mutable list `char_list`.

**Example of the mistake**:
```python
# WRONG: comparing with original string
if s[left_ptr] == s[search_idx]:  # Should be char_list
    # ...
```

**Solution**: Always work with `char_list` after the initial conversion, as strings are immutable in Python and we need to track the changes made by our swaps.

