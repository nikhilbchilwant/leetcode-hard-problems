# 1044. Longest Duplicate Substring

> Source: https://algo.monster/liteproblems/1044

## Problem Statement

You are given a string `s` and need to find the longest substring that appears at least twice in the string. These duplicate substrings can overlap with each other.

A duplicated substring is a contiguous sequence of characters from `s` that occurs 2 or more times anywhere in the string. For example, in the string "banana", the substring "ana" appears twice (at positions 1-3 and 3-5), and these occurrences overlap at position 3.

Your task is to return any duplicated substring that has the maximum possible length. If no substring appears more than once in `s`, return an empty string `""`.

The solution uses [binary search](/problems/binary_search_intro) on the length of the potential duplicate substring. The `check` function verifies if there exists a duplicate substring of a given length `l` by iterating through all possible substrings of that length and using a set to track which substrings have been seen. If a substring is encountered that's already in the set, it means we've found a duplicate.

The [binary search](/problems/binary_search_intro) works by:
- Setting the search range from 0 to the length of the string
- For each middle value `mid`, checking if a duplicate substring of length `mid` exists
- If found, searching for longer duplicates (moving `left` pointer up)
- If not found, searching for shorter duplicates (moving `right` pointer down)
- Keeping track of the longest valid duplicate found so far in `ans`

## Intuition

The key insight is that if a substring of length `k` appears twice in the string, then we know the answer is at least `k` characters long. However, there might be an even longer duplicate substring of length `k+1`, `k+2`, and so on. We want to find the maximum such length.

A brute force approach would check all possible substrings, but this would be inefficient. Instead, we can observe that the problem has a monotonic property: if there exists a duplicate substring of length `k`, then there must also exist duplicate substrings of all lengths from `1` to `k-1`. This is because any substring of a duplicate substring is also a duplicate substring.

This monotonic property suggests using [binary search](/problems/binary_search_intro) on the answer. We can search for the maximum length `L` such that there exists a duplicate substring of length `L`.

For each candidate length during [binary search](/problems/binary_search_intro), we need to verify if any substring of that length appears twice. We can do this efficiently by:
1. Generating all substrings of the given length
2. Using a hash set to track which substrings we've seen
3. If we encounter a substring that's already in the set, we've found a duplicate

The [binary search](/problems/binary_search_intro) narrows down the search space from `O(n)` possible lengths to `O(log n)` checks. For each check, we examine `O(n)` substrings, and comparing strings takes `O(L)` time where `L` is the current length being checked. This gives us a more efficient solution than checking all possible substring pairs.

The approach elegantly combines the efficiency of [binary search](/problems/binary_search_intro) with the simplicity of hash-based duplicate detection, making it practical even for longer strings.

**Learn more about [Binary Search](/problems/binary_search_intro) and [Sliding Window](/problems/sliding_window_maximum) patterns.**

## Solution Approach

The solution implements a [binary search](/problems/binary_search_intro) combined with a hash set-based duplicate detection function.

**[Binary Search](/problems/binary_search_intro) Setup:**
- Initialize `left = 0` and `right = n` (length of string) as the search boundaries for possible substring lengths
- Use `ans` to keep track of the longest duplicate substring found so far

**[Binary Search](/problems/binary_search_intro) Logic:**
The main loop continues while `left < right`:
1. Calculate `mid = (left + right + 1) >> 1` (equivalent to `(left + right + 1) // 2`)
   - The `+1` ensures we round up to avoid infinite loops when `left` and `right` are adjacent
2. Call `check(mid)` to verify if a duplicate substring of length `mid` exists
3. Update the answer: `ans = t or ans` keeps the non-empty result (either the new found substring `t` or the previous `ans`)
4. Adjust search boundaries:
   - If a duplicate of length `mid` is found, search for longer substrings: `left = mid`
   - Otherwise, search for shorter substrings: `right = mid - 1`

**The `check` Function:**
This helper function determines if any substring of length `l` appears at least twice:
1. Initialize an empty set `vis` to track seen substrings
2. Iterate through all possible starting positions from `0` to `n - l`
3. For each position `i`, extract substring `t = s[i : i + l]`
4. If `t` is already in `vis`, we found a duplicate - return it immediately
5. Otherwise, add `t` to `vis` and continue
6. If no duplicate is found after checking all positions, return empty string

**Time Complexity Analysis:**
- [Binary search](/problems/binary_search_intro) performs `O(log n)` iterations
- Each `check` call examines `O(n)` substrings
- String slicing and hashing takes `O(L)` time for length `L`
- Overall: `O(n * L * log n)` where `L` is the average substring length checked

The algorithm efficiently narrows down the search space using [binary search](/problems/binary_search_intro) while maintaining simplicity through straightforward string comparison using Python's built-in set operations.

## Example Walkthrough

Let's trace through the algorithm with the string `s = "banana"`.

**Initial Setup:**
- String: "banana" (length n = 6)
- Binary search range: left = 0, right = 6
- Answer: ans = ""

**Iteration 1:**
- mid = (0 + 6 + 1) >> 1 = 3
- Check if any substring of length 3 appears twice:
  - i=0: "ban" → add to vis = {"ban"}
  - i=1: "ana" → add to vis = {"ban", "ana"}
  - i=2: "nan" → add to vis = {"ban", "ana", "nan"}
  - i=3: "ana" → already in vis! Return "ana"
- Found duplicate "ana", so ans = "ana"
- Search for longer: left = 3, right = 6

**Iteration 2:**
- mid = (3 + 6 + 1) >> 1 = 5
- Check if any substring of length 5 appears twice:
  - i=0: "banan" → add to vis = {"banan"}
  - i=1: "anana" → add to vis = {"banan", "anana"}
- No duplicates found, return ""
- No duplicate found, search for shorter: left = 3, right = 4

**Iteration 3:**
- mid = (3 + 4 + 1) >> 1 = 4
- Check if any substring of length 4 appears twice:
  - i=0: "bana" → add to vis = {"bana"}
  - i=1: "anan" → add to vis = {"bana", "anan"}
  - i=2: "nana" → add to vis = {"bana", "anan", "nana"}
- No duplicates found, return ""
- No duplicate found, search for shorter: left = 3, right = 3

**Termination:**
- left = right = 3, loop ends
- Return ans = "ana"

The algorithm correctly identifies "ana" as the longest duplicated substring. It appears at positions [1:4] and [3:6], with an overlap at position 3. Through binary search, we efficiently narrowed down from checking all possible lengths to finding the optimal length in just 3 iterations.

## Python Solution

```python
class Solution:
    def longestDupSubstring(self, s: str) -> str:
        """
        Find the longest duplicate substring in a given string.
        Uses binary search on the length of substring.
        """

        def has_duplicate_of_length(length: int) -> str:
            """Check if a duplicate substring of given length exists."""
            seen_substrings = set()

            for start_idx in range(len(s) - length + 1):
                current_substring = s[start_idx : start_idx + length]
                if current_substring in seen_substrings:
                    return current_substring
                seen_substrings.add(current_substring)

            return ''

        def feasible(length: int) -> bool:
            """
            Returns True if NO duplicate of this length exists.
            This inverts the condition so we can find "first true" = first length with no duplicate.
            """
            return has_duplicate_of_length(length) == ''

        n = len(s)
        left, right = 1, n - 1
        first_true_index = -1
        longest_duplicate = ''

        # Binary search template: find first length where no duplicate exists
        while left <= right:
            mid = (left + right) // 2
            if feasible(mid):
                first_true_index = mid
                right = mid - 1  # Search for smaller length with no duplicate
            else:
                # Duplicate exists at this length, record it and search longer
                longest_duplicate = has_duplicate_of_length(mid)
                left = mid + 1

        return longest_duplicate
```

## Time and Space Complexity

**Time Complexity:** `O(n² * log n)`

The algorithm uses binary search to find the longest duplicate substring. The binary search runs for `O(log n)` iterations where `n` is the length of the string. In each iteration, the `check` function is called.

The `check` function iterates through all possible substrings of length `l`, which takes `O(n - l + 1) ≈ O(n)` iterations. For each substring, it performs:
- String slicing `s[i : i + l]` which takes `O(l)` time
- Set membership check and insertion which takes `O(l)` time on average (due to string hashing)

Therefore, the `check` function has time complexity `O(n * l)`. In the worst case, `l` can be up to `n/2` (when the longest duplicate substring is half the string length), making each `check` call `O(n²)` in the worst case.

Overall time complexity: `O(log n) * O(n²) = O(n² * log n)`

**Space Complexity:** `O(n²)`

The space complexity is dominated by the `vis` set in the `check` function. In the worst case:
- The set can store up to `O(n)` different substrings
- Each substring can have length up to `O(n)`
- Therefore, the total space used can be `O(n²)`

Additionally, the `ans` variable stores a string of length up to `O(n)`, but this doesn't change the overall space complexity.

Overall space complexity: `O(n²)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Using Wrong Binary Search Template Variant**

A common mistake is using `while left < right` with `left = mid` when a duplicate is found:

```python
# WRONG - different template variant
while left < right:
    mid = (left + right + 1) >> 1
    if has_duplicate(mid):
        left = mid
    else:
        right = mid - 1
return result
```

**Correct template:**
```python
first_true_index = -1
while left <= right:
    mid = (left + right) // 2
    if feasible(mid):  # no duplicate exists
        first_true_index = mid
        right = mid - 1
    else:  # duplicate exists, record and search longer
        longest_duplicate = has_duplicate_of_length(mid)
        left = mid + 1
return longest_duplicate
```

For "find maximum length" problems, invert the feasible condition to find "first length where no duplicate exists".

### 2. **Memory Limit Exceeded with Large Strings**
Storing full substring strings in a set consumes excessive memory. Use rolling hash (Rabin-Karp algorithm) instead.

### 3. **Off-by-One Error in Substring Extraction**
Using `range(n - length)` instead of `range(n - length + 1)` misses the last valid substring.

```python
for start_idx in range(string_length - length + 1):  # Correct
```

### 4. **Hash Collision Issues (When Using Rolling Hash)**
Hash collisions can lead to false positives. Perform secondary verification or use multiple hash functions when a hash match is found.

