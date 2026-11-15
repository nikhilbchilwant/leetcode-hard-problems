# 1531. String Compression II

> Source: https://algo.monster/liteproblems/1531

## Problem Statement

This problem asks you to find the minimum length of a run-length encoded string after deleting at most `k` characters from the original string `s`.

**Run-length encoding** compresses a string by replacing consecutive identical characters (that appear 2 or more times) with the character followed by its count. For example:
- `"aabccc"` becomes `"a2bc3"` 
- `"aa"` becomes `"a2"`
- `"ccc"` becomes `"c3"`
- Single characters like `"b"` remain as `"b"` (not `"b1"`)

Given a string `s` and an integer `k`, you can delete **at most** `k` characters from `s` to minimize the length of the resulting run-length encoded string.

**The goal**: Find the minimum possible length of the run-length encoded version after optimally deleting up to `k` characters.

**Example scenarios:**
- If you have `"aaabcccd"` and `k=2`, you might delete `"b"` and `"d"` to get `"aaaccc"`, which encodes to `"a3c3"` with length 4
- The length calculation for encoded strings:
  - A single character has length 1 (e.g., `"a"`)
  - 2-9 consecutive characters have length 2 (e.g., `"a2"` to `"a9"`)
  - 10-99 consecutive characters have length 3 (e.g., `"a10"` to `"a99"`)
  - 100 consecutive characters have length 4 (e.g., `"a100"`)

## Intuition

The key insight is that run-length encoding benefits from having consecutive identical characters. When we can delete characters, we want to create longer runs of the same character or eliminate characters that break up existing runs.

Consider the string `"aabaac"`. If we delete the `'b'`, we get `"aaaac"` which encodes to `"a4c"` (length 3). Without deletion, it would be `"a2ba2c"` (length 6). This shows how strategic deletions can significantly reduce the encoded length.

The challenge is deciding which characters to delete. We need to think about this problem in terms of segments. For any substring, we can choose to:
1. Keep all characters of one type and delete the rest to form a single run
2. Split the string at some point and handle the parts separately

This naturally leads to a [dynamic programming](/problems/dynamic_programming_intro) approach where we consider all possible ways to handle a substring starting at position `i` with `k` deletions remaining.

For each starting position `i`, we examine all possible ending positions `j`. In the range `[i, j]`, we can choose to keep only the most frequent character and delete all others. This transforms the range into a single run. The cost of this operation is `(j - i + 1 - maxFreq)` deletions, where `maxFreq` is the count of the most frequent character in this range.

The beauty of this approach is that it considers all possible segmentation strategies:
- Making the entire remaining string into one character type
- Making just the first few characters the same, then optimally handling the rest
- Any combination in between

By tracking the frequency of characters as we extend our range from `i` to `j`, we can efficiently calculate the cost of making that range uniform. The encoded length of a uniform segment depends on its frequency: 1 character for single occurrences, 2 for frequencies 2-9, 3 for 10-99, and 4 for 100.

The recursion with memoization ensures we don't recalculate the same subproblems, making the solution efficient despite considering all possible segmentation strategies.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution uses [dynamic programming](/problems/dynamic_programming_intro) with memoization to find the minimum encoded length. Here's how the implementation works:

**Data Structures:**
- `dp[i][k]`: A 2D memoization array where `dp[i][k]` stores the minimum encoded length for substring `s[i..n)` with at most `k` deletions allowed
- `K_MAX = 101`: A sentinel value representing an impossible/invalid state

**Main Algorithm Flow:**

1. **Initialization**: The `dp` array is initialized with `K_MAX` values to indicate uncomputed states.

2. **Recursive Function** `compression(s, i, k)`:
   - **Base Cases:**
     - If `k < 0`: Return `K_MAX` (invalid - too many deletions)
     - If `i == s.length()` or `s.length() - i <= k`: Return 0 (can delete entire remaining string)
   - **Memoization Check**: If `dp[i][k]` already computed, return cached value

3. **Core Logic - Segment Processing:**
   ```java
   int maxFreq = 0;
   int[] count = new int[128];  // ASCII character frequency counter
 
   for (int j = i; j < s.length(); ++j) {
       maxFreq = Math.max(maxFreq, ++count[s.charAt(j)]);
       dp[i][k] = Math.min(dp[i][k], 
           getLength(maxFreq) + compression(s, j + 1, k - (j - i + 1 - maxFreq)));
   }
   ```
 
   For each starting position `i`, the algorithm:
   - Iterates through all possible ending positions `j`
   - Maintains character frequencies in the range `[i, j]` using the `count` array
   - Tracks `maxFreq`: the highest frequency character in current range
   - Calculates the cost of making range `[i, j]` uniform:
     - Keep `maxFreq` characters of the same type
     - Delete `(j - i + 1 - maxFreq)` other characters
     - Encoded length becomes `getLength(maxFreq)`
   - Recursively solves for the remaining string `s[j+1..n)`

4. **Length Calculation** `getLength(maxFreq)`:
   ```java
   if (maxFreq == 1) return 1;      // "a"
   if (maxFreq < 10) return 2;      // "a2" to "a9"
   if (maxFreq < 100) return 3;     // "a10" to "a99"
   return 4;                        // "a100"
   ```
   This helper function computes the encoded length based on run frequency.

**Example Walkthrough:**

For `s = "aaabcccd"` and `k = 2`:
- At `i=0`, we explore making segments:
  - `[0,2]` all 'a': length = 2 ("a3") + solve rest with k=2
  - `[0,5]` keep 'a' or 'c': delete 3 chars, need k≥3 (invalid)
  - `[0,3]` keep 'a': delete 'b', length = 2 ("a3") + solve rest with k=1

The algorithm explores all valid segmentations and returns the minimum encoded length found.

**Time Complexity:** `O(n²k)` where n is string length - for each of `n*k` states, we iterate through up to `n` positions.

**Space Complexity:** `O(nk)` for the memoization table.

## Example Walkthrough

Let's walk through a small example with `s = "aabb"` and `k = 1`:

**Initial Setup:**
- String: "aabb" (length 4)
- Maximum deletions allowed: k = 1
- Goal: Find minimum encoded length after deleting at most 1 character

**Step-by-step Process:**

Starting at position `i = 0` with `k = 1` deletion available:

1. **Consider segment [0,0] = "a"**:
   - Keep 1 'a', delete 0 characters
   - Encoded length of this segment: 1 (single 'a')
   - Remaining string: "abb" starting at index 1
   - Total: 1 + compression("abb", k=1)

2. **Consider segment [0,1] = "aa"**:
   - Keep 2 'a's, delete 0 characters  
   - Encoded length of this segment: 2 ("a2")
   - Remaining string: "bb" starting at index 2
   - Total: 2 + compression("bb", k=1)

3. **Consider segment [0,2] = "aab"**:
   - Option: Keep 2 'a's, delete 1 'b'
   - Cost: 1 deletion (within budget)
   - Encoded length: 2 ("a2")
   - Remaining string: "b" starting at index 3
   - Total: 2 + compression("b", k=0)

4. **Consider segment [0,3] = "aabb"**:
   - Option: Keep 2 'a's or 2 'b's, delete 2 others
   - Cost: 2 deletions (exceeds k=1)
   - This option is invalid

**Recursive Calls:**

For `compression("bb", k=1)` at index 2:
- Can make entire "bb" into "b2" (length 2) with 0 deletions
- Result: 2

For `compression("b", k=0)` at index 3:
- Single 'b' has length 1
- Result: 1

**Final Calculation:**

Comparing all valid options from position 0:
- Option 1: 1 + compression("abb", k=1) = 1 + 3 = 4
  - "abb" becomes "a" + "bb" = "a" + "b2" = 3
- Option 2: 2 + compression("bb", k=1) = 2 + 2 = 4  
  - "aa" + "bb" = "a2" + "b2" = 4
- Option 3: 2 + compression("b", k=0) = 2 + 1 = 3
  - Delete middle 'b', get "aa" + "b" = "a2b" = 3

**Optimal Solution:** 
Delete the 'b' at index 2, resulting in "aab" which encodes to "a2b" with length 3.

This example demonstrates how the algorithm explores different ways to segment the string, calculating the cost of making each segment uniform and recursively solving for the remainder.

## Python Solution

```python
class Solution:
    def getLengthOfOptimalCompression(self, s: str, k: int) -> int:
        # Maximum value used to represent invalid/impossible states
        MAX_VALUE = 101
      
        # Memoization table: dp[i][j] represents the minimum length of compressed string
        # starting from index i with at most j deletions allowed
        dp = [[MAX_VALUE] * (k + 1) for _ in range(len(s))]
      
        def get_compression_length(frequency):
            """
            Calculates the length of run-length encoding for a given frequency
            For example: 'a' -> 1, 'aa' -> 'a2' -> 2, 'aaa' -> 'a3' -> 2
          
            Args:
                frequency: The frequency of a character
          
            Returns:
                The length after run-length encoding
            """
            if frequency == 1:
                return 1  # Single character: 'c'
            if frequency < 10:
                return 2  # Character + single digit: 'c2' to 'c9'
            if frequency < 100:
                return 3  # Character + two digits: 'c10' to 'c99'
            return 4      # Character + three digits: 'c100'
      
        def find_min_compression(start_idx, deletions_left):
            """
            Recursively finds the minimum compression length starting from index start_idx
            with at most deletions_left deletions allowed
          
            Args:
                start_idx: The starting index for compression
                deletions_left: The number of deletions still allowed
          
            Returns:
                The minimum compression length
            """
            # Base case: if we've exceeded deletion limit, return invalid state
            if deletions_left < 0:
                return MAX_VALUE
          
            # Base case: if we've reached the end or can delete all remaining characters
            if start_idx == len(s) or len(s) - start_idx <= deletions_left:
                return 0
          
            # Return memoized result if already computed
            if dp[start_idx][deletions_left] != MAX_VALUE:
                return dp[start_idx][deletions_left]
          
            # Track character frequencies in the current window
            char_count = {}
            max_frequency = 0
          
            # Try all possible windows starting from start_idx
            for end_idx in range(start_idx, len(s)):
                # Update frequency of current character
                current_char = s[end_idx]
                char_count[current_char] = char_count.get(current_char, 0) + 1
                max_frequency = max(max_frequency, char_count[current_char])
              
                # Calculate deletions needed: total chars in window - max frequency
                window_size = end_idx - start_idx + 1
                deletions_needed = window_size - max_frequency
              
                # Calculate compression length for keeping the most frequent character
                compression_length = get_compression_length(max_frequency)
              
                # Recursively calculate the rest and update minimum
                total_length = compression_length + find_min_compression(
                    end_idx + 1, deletions_left - deletions_needed
                )
              
                dp[start_idx][deletions_left] = min(
                    dp[start_idx][deletions_left], total_length
                )
          
            return dp[start_idx][deletions_left]
      
        # Start the recursive compression from index 0 with k deletions allowed
        return find_min_compression(0, k)
```

## Time and Space Complexity

**Time Complexity:** `O(n^2 * k)` where `n` is the length of string `s` and `k` is the maximum number of deletions allowed.

The analysis breaks down as follows:
- The memoization table `dp[i][k]` has `n * k` states to compute
- For each state `dp[i][k]`, we iterate through all positions from `i` to `n-1` in the inner loop (the `j` loop)
- This inner loop runs at most `n - i` times, which is `O(n)` in the worst case
- For each iteration of the inner loop, we perform constant time operations: updating the frequency count, calculating `getLength()`, and making a recursive call
- The recursive call either returns a memoized result in `O(1)` or computes a new state
- Since each state is computed only once due to memoization, the total time complexity is `O(n * k * n) = O(n^2 * k)`

**Space Complexity:** `O(n * k)` 

The space complexity consists of:
- The memoization table `dp[n][k+1]` requires `O(n * k)` space
- The recursion call stack can go up to depth `n` in the worst case, contributing `O(n)` space
- The `count` array in each recursive call uses `O(128) = O(1)` space
- Overall space complexity is dominated by the memoization table: `O(n * k)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Handling of Single Character Runs**
A frequent mistake is treating single characters incorrectly in the encoding length calculation. Remember that a single character like 'a' has length 1, not 2 (it's encoded as 'a', not 'a1').

**Incorrect:**
```python
def get_compression_length(frequency):
    if frequency <= 1:
        return 2  # Wrong! This would treat 'a' as 'a1'
    # ...
```

**Correct:**
```python
def get_compression_length(frequency):
    if frequency == 1:
        return 1  # Single character remains as-is
    # ...
```

### 2. **Off-by-One Error in Deletion Count**
When calculating the number of deletions needed to make a segment uniform, developers often miscalculate by forgetting that we keep the most frequent character and delete everything else.

**Incorrect:**
```python
# Wrong calculation - might subtract 1 unnecessarily
deletions_needed = window_size - max_frequency - 1
```

**Correct:**
```python
# Keep max_frequency chars, delete the rest
deletions_needed = window_size - max_frequency
```

### 3. **Not Considering All Possible Segmentations**
A critical mistake is prematurely optimizing by breaking the loop early or not considering all possible ending positions for each starting position. The algorithm needs to explore all valid windows to find the optimal solution.

**Incorrect:**
```python
for end_idx in range(start_idx, len(s)):
    # ... calculate compression ...
    if deletions_needed > deletions_left:
        break  # Wrong! Later characters might form better segments
```

**Correct:**
```python
for end_idx in range(start_idx, len(s)):
    # ... calculate compression ...
    # Continue exploring even if current window seems suboptimal
    total_length = compression_length + find_min_compression(
        end_idx + 1, deletions_left - deletions_needed
    )
```

### 4. **Improper Base Case for Remaining String**
Failing to handle the case where we can delete all remaining characters leads to incorrect results.

**Incorrect:**
```python
# Only checking if we've reached the end
if start_idx == len(s):
    return 0
```

**Correct:**
```python
# Can delete entire remaining string if we have enough deletions
if start_idx == len(s) or len(s) - start_idx <= deletions_left:
    return 0
```

### 5. **Integer Overflow in Length Calculation**
While Python handles large integers well, in other languages or when optimizing, assuming the string length won't exceed 100 can cause issues.

**Solution:** Always validate input constraints and handle edge cases:
```python
def get_compression_length(frequency):
    if frequency == 1:
        return 1
    if frequency < 10:
        return 2
    if frequency < 100:
        return 3
    if frequency == 100:  # Explicitly handle the upper bound
        return 4
    # Handle unexpected cases
    return 4  # Or raise an exception for invalid input
```

### 6. **Not Initializing Memoization Table Properly**
Using 0 or -1 as the initial value instead of a sentinel value can cause confusion between "not computed" and "valid result of 0".

**Incorrect:**
```python
dp = [[0] * (k + 1) for _ in range(len(s))]  # 0 could be a valid result!
```

**Correct:**
```python
dp = [[MAX_VALUE] * (k + 1) for _ in range(len(s))]  # Clear sentinel value
```

