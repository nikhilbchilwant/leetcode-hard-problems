# 1147. Longest Chunked Palindrome Decomposition

> Source: https://algo.monster/liteproblems/1147

## Problem Statement

You are given a string `text` that needs to be split into k substrings `(subtext₁, subtext₂, ..., textₖ)` with the following conditions:

1. Each substring must be **non-empty**
2. When concatenated together, all substrings must equal the original `text` (i.e., `subtext₁ + subtext₂ + ... + subtextₖ == text`)
3. The substrings must form a palindromic pattern where `subtextᵢ == subtextₖ₋ᵢ₊₁` for all valid values of i (where `1 ≤ i ≤ k`)

The third condition means that the first substring must equal the last substring, the second must equal the second-to-last, and so on. For example, if we split "ghiabcdefhelloadamhelloabcdefghi" into 5 parts: `["ghi", "abcdef", "hello", "adamhello", "abcdef", "ghi"]`, we can see that:
- 1st substring "ghi" equals 5th substring "ghi" 
- 2nd substring "abcdef" equals 4th substring "abcdef"
- 3rd substring "helloadamhello" is in the middle (equals itself when k is odd)

Your task is to find the **maximum possible value of k** - the largest number of substrings you can split the text into while satisfying all three conditions.

## Intuition

The key insight is that we want to maximize the number of pieces, which means we should try to create as many matching pairs as possible from the ends of the string.

Think about it this way: if we have a string and we can find a prefix that matches a suffix, we've created 2 pieces. The question becomes - should we take the shortest matching pair or look for longer ones?

Let's consider why choosing the **shortest matching prefix-suffix pair** is optimal:

Imagine we have two options:
- Option A: Take a short matching pair (like "a" from both ends)
- Option B: Take a longer matching pair (like "abc" from both ends)

If we choose the longer pair (Option B), we're consuming more characters, leaving less room for potential matches in the middle. By choosing the shorter pair (Option A), we leave more characters in the middle, giving us more opportunities to find additional matching pairs.

Here's a visual example: Consider the string "abcXYZabc"
- If we greedily take "a" and "c" as our first matching pair (both single characters 'a'), we're left with "bcXYZab" 
- If we instead took "abc" as our matching pair, we're left with just "XYZ"

The [greedy](/problems/greedy_intro) choice of shorter matches gives us more flexibility for the remaining string.

This naturally leads to a two-pointer approach:
1. Start with pointers at the beginning and end of the string
2. Try to find the shortest substring from the left that matches a substring from the right
3. When found, count it as 2 pieces and move both pointers inward
4. Repeat until the pointers meet or cross
5. If there's a remaining middle portion that couldn't be matched, it counts as 1 piece

The correctness of this [greedy](/problems/greedy_intro) approach stems from the fact that any valid decomposition can be rearranged to have shorter matches on the outside without reducing the total number of pieces.

**Learn more about [Greedy](/problems/greedy_intro), [Two Pointers](/problems/two_pointers_intro) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The implementation uses a **[greedy](/problems/greedy_intro) approach with [two pointers](/problems/two_pointers_intro)** to find the maximum number of palindromic chunks.

We maintain [two pointers](/problems/two_pointers_intro):
- `i` starting from the beginning of the string (index 0)
- `j` starting from the end of the string (index `len(text) - 1`)

The algorithm works as follows:

1. **Initialize variables:**
   - `ans = 0` to count the number of chunks
   - `i = 0` and `j = len(text) - 1` as our [two pointers](/problems/two_pointers_intro)

2. **Main loop** - while `i <= j`:
   - Set `k = 1` to represent the current substring length we're checking
   - Set `ok = False` to track whether we found a matching pair
 
3. **Inner loop** - search for the shortest matching prefix and suffix:
   - While `i + k - 1 < j - k + 1` (ensuring substrings don't overlap):
     - Check if `text[i : i + k] == text[j - k + 1 : j + 1]`
     - If they match:
       - Add 2 to the answer (we found a matching pair)
       - Move `i` forward by `k` positions: `i += k`
       - Move `j` backward by `k` positions: `j -= k`
       - Set `ok = True` and break from the inner loop
     - If no match, increment `k` to try a longer substring

4. **Handle the middle portion:**
   - If `ok` is still `False` after the inner loop (no matching pair found):
     - Add 1 to the answer (the remaining middle portion becomes a single chunk)
     - Break from the main loop

5. **Return** the final answer

**Example walkthrough with "ghiabcdefhelloadamhelloabcdefghi":**
- First iteration: Find "ghi" matches at both ends, `ans += 2`, move pointers inward
- Second iteration: Find "abcdef" matches at both ends, `ans += 2`, move pointers inward
- Third iteration: Remaining string "helloadamhello" has no matching prefix-suffix, `ans += 1`
- Final answer: 5

The time complexity is `O(n²)` in the worst case, where we might need to check multiple substring lengths at each step. The space complexity is `O(n)` for the string slicing operations.

## Example Walkthrough

Let's walk through the algorithm with the string `"abcab"` to find the maximum number of palindromic substrings.

**Initial Setup:**
- String: `"abcab"`
- `i = 0` (pointing to 'a' at the start)
- `j = 4` (pointing to 'b' at the end)
- `ans = 0`

**Iteration 1:**
- We're looking for matching prefix and suffix
- Try `k = 1`: Check if `text[0:1]` ("a") equals `text[4:5]` ("b")
  - "a" ≠ "b", no match
- Try `k = 2`: Check if `text[0:2]` ("ab") equals `text[3:5]` ("ab")
  - "ab" = "ab", **match found!**
- Add 2 to answer: `ans = 2`
- Move pointers: `i = 2`, `j = 2`

**Iteration 2:**
- Now `i = 2` and `j = 2` (both pointing to the middle 'c')
- Since `i <= j` is still true, we continue
- The remaining substring is just "c"
- Try `k = 1`: But `i + k - 1 = 2` is not less than `j - k + 1 = 2`
- No matching pair can be found (we're at a single character)
- Add 1 to answer: `ans = 3`
- Break from loop

**Result:**
The string "abcab" can be split into maximum **3 palindromic substrings**: `["ab", "c", "ab"]`

Let's verify:
- Concatenation: "ab" + "c" + "ab" = "abcab" ✓
- Palindromic pattern: 1st substring "ab" = 3rd substring "ab", 2nd substring "c" = itself ✓

This example demonstrates how the greedy approach finds the shortest matching pairs ("ab" instead of trying longer substrings first) to maximize the total number of pieces.

## Python Solution

```python
class Solution:
    def longestDecomposition(self, text: str) -> int:
        """
        Find the maximum number of chunks in a decomposition where the string
        can be split into (a1, a2, ..., ak, r, ak, ..., a2, a1) where r can be empty.
      
        Args:
            text: The input string to decompose
          
        Returns:
            The maximum number of chunks in the decomposition
        """
        chunk_count = 0
        left_pointer = 0
        right_pointer = len(text) - 1
      
        # Process the string from both ends towards the center
        while left_pointer <= right_pointer:
            substring_length = 1
            match_found = False
          
            # Try to find matching substrings of increasing length
            while left_pointer + substring_length - 1 < right_pointer - substring_length + 1:
                # Extract substring from left side
                left_substring = text[left_pointer:left_pointer + substring_length]
                # Extract substring from right side
                right_substring = text[right_pointer - substring_length + 1:right_pointer + 1]
              
                # Check if the substrings match
                if left_substring == right_substring:
                    # Found matching chunks, count them as 2 pieces
                    chunk_count += 2
                    # Move pointers inward by the length of matched substring
                    left_pointer += substring_length
                    right_pointer -= substring_length
                    match_found = True
                    break
              
                # Try longer substring in next iteration
                substring_length += 1
          
            # If no matching substrings found, the remaining middle part counts as 1 chunk
            if not match_found:
                chunk_count += 1
                break
      
        return chunk_count
```

## Time and Space Complexity

**Time Complexity:** `O(n²)`

The algorithm uses two pointers `i` and `j` starting from both ends of the string. In the worst case:
- The outer while loop runs until `i <= j`, which can iterate up to `n/2` times where `n` is the length of the string
- For each iteration of the outer loop, the inner while loop increments `k` to find matching substrings
- In the worst case, `k` can grow up to `n/2` (when no matching prefix-suffix pairs are found until the middle)
- String slicing `text[i : i + k]` and `text[j - k + 1 : j + 1]` takes `O(k)` time
- String comparison also takes `O(k)` time

Therefore, the worst-case scenario involves `O(n/2)` iterations of the outer loop, with each iteration potentially doing `O(n/2)` work in the inner loop for substring operations, resulting in `O(n²)` time complexity.

**Space Complexity:** `O(n)`

The space complexity is `O(n)` due to the string slicing operations. When Python performs slicing like `text[i : i + k]` and `text[j - k + 1 : j + 1]`, it creates new string objects that require additional memory proportional to the slice length. In the worst case, these slices can be up to `O(n/2)` in length.

If the implementation were modified to compare characters directly without creating substring copies (using index-based character comparison), the space complexity could be reduced to `O(1)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Incorrect Handling of the Middle Element

**The Problem:**
A common mistake is forgetting to count the middle portion when no matching prefix-suffix pair can be found. When the pointers meet or cross without finding any matches, the remaining string still counts as one valid chunk.

**Incorrect Implementation:**
```python
while left_pointer <= right_pointer:
    substring_length = 1
    match_found = False
  
    while left_pointer + substring_length - 1 < right_pointer - substring_length + 1:
        if left_substring == right_substring:
            chunk_count += 2
            match_found = True
            break
        substring_length += 1
  
    # WRONG: Forgetting to handle the case when no match is found
    if not match_found:
        break  # This loses the middle chunk!
```

**Correct Implementation:**
```python
if not match_found:
    chunk_count += 1  # Count the remaining middle portion
    break
```

### Pitfall 2: Off-by-One Errors in Pointer Boundaries

**The Problem:**
The condition for checking substring overlap is critical. Using `<=` instead of `<` in the inner loop condition can cause the algorithm to check overlapping substrings, leading to incorrect results.

**Incorrect Implementation:**
```python
# WRONG: This allows overlapping substrings
while left_pointer + substring_length - 1 <= right_pointer - substring_length + 1:
    # This might check overlapping regions
```

**Correct Implementation:**
```python
# Ensures substrings don't overlap
while left_pointer + substring_length - 1 < right_pointer - substring_length + 1:
    # Proper boundary checking
```

### Pitfall 3: Greedy Algorithm Correctness Assumption

**The Problem:**
Some might think we need to try all possible decompositions to find the maximum k, not trusting that the greedy approach (finding shortest matching prefix-suffix) always yields the optimal solution.

**Why Greedy Works:**
The key insight is that if we have a matching prefix and suffix of length L, splitting them off immediately is always optimal. Any alternative decomposition that keeps these L characters as part of larger chunks would result in fewer total chunks. The greedy choice of smallest matching chunks maximizes the number of pieces.

**Example to Illustrate:**
For string "abcabc":
- Greedy approach: Split as ["a", "b", "c", "a", "b", "c"] → 6 chunks (though this would actually be ["a","bc","a"] → 3 chunks following the algorithm)
- Non-greedy might try: ["abc", "abc"] → 2 chunks
- The greedy approach correctly identifies ["a", "bc", "a"] by finding the shortest match first

### Pitfall 4: String Slicing Performance

**The Problem:**
Creating new substring copies with slicing operations in Python can be inefficient for very long strings, potentially causing performance issues.

**Alternative Approach Using Indices:**
```python
def longestDecomposition(self, text: str) -> int:
    def matches(start1, end1, start2, end2):
        # Compare characters without creating substrings
        if end1 - start1 != end2 - start2:
            return False
        for i in range(end1 - start1):
            if text[start1 + i] != text[start2 + i]:
                return False
        return True
  
    chunk_count = 0
    left = 0
    right = len(text) - 1
  
    while left <= right:
        length = 1
        found = False
      
        while left + length - 1 < right - length + 1:
            if matches(left, left + length, right - length + 1, right + 1):
                chunk_count += 2
                left += length
                right -= length
                found = True
                break
            length += 1
      
        if not found:
            chunk_count += 1
            break
  
    return chunk_count
```

This approach avoids creating substring copies and can be more memory-efficient for large inputs.

