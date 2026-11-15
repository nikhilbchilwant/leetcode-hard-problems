# 2842. Count K-Subsequences of a String With Maximum Beauty

> Source: https://algo.monster/liteproblems/2842

## Problem Statement

You are given a string `s` and an integer `k`. Your task is to find k-subsequences with maximum beauty value.

A **k-subsequence** is defined as:
- A subsequence of string `s` with exactly length `k`
- All characters in the subsequence must be unique (each character appears only once in the subsequence)

The **beauty** of a k-subsequence is calculated as the sum of `f(c)` for every character `c` in that subsequence, where `f(c)` is the total number of times character `c` appears in the original string `s` (not in the subsequence).

For example, with `s = "abbbdd"` and `k = 2`:
- Character frequencies: `f('a') = 1`, `f('b') = 3`, `f('d') = 2`
- Some possible k-subsequences:
  - `"ab"` has beauty = `f('a') + f('b') = 1 + 3 = 4`
  - `"ad"` has beauty = `f('a') + f('d') = 1 + 2 = 3`
  - `"bd"` has beauty = `f('b') + f('d') = 3 + 2 = 5`

You need to:
1. Find the maximum possible beauty value among all k-subsequences
2. Count how many k-subsequences achieve this maximum beauty
3. Return the count modulo `10^9 + 7`

Important notes:
- Two k-subsequences are considered different if they use different indices from the original string, even if they form the same character sequence
- If it's impossible to form a k-subsequence (when there are fewer than `k` distinct characters in `s`), return `0`

The challenge involves identifying which characters to include in the k-subsequence to maximize beauty, then counting all possible ways to form such subsequences from the original string positions.

## Intuition

To maximize the beauty of a k-subsequence, we need to choose characters that have the highest frequencies in the original string. Think of it this way: since beauty is the sum of `f(c)` for each character in our subsequence, we want to include characters with the largest `f(c)` values.

The [greedy](/problems/greedy_intro) approach works here - we should always pick the `k` characters with the highest frequencies. If we have characters with frequencies `[5, 4, 3, 2, 1]` and need to pick `k=3` characters, we'd pick those with frequencies `[5, 4, 3]` to get maximum beauty of `5 + 4 + 3 = 12`.

However, there's a complication when multiple characters have the same frequency. For example, if frequencies are `[5, 4, 3, 3, 3, 2]` and `k=4`, we must pick the characters with frequencies `[5, 4]` and then choose 2 out of the 3 characters that have frequency 3.

This leads us to the key insight: 
1. First, sort all character frequencies in descending order
2. The k-th largest frequency (`val`) determines our cutoff point
3. All characters with frequency greater than `val` must be included
4. For characters with frequency exactly equal to `val`, we need to choose some of them

The counting part requires combinatorial mathematics:
- For characters with frequency > `val`: each occurrence in the string can be chosen independently, giving us `frequency` choices per character
- For characters with frequency = `val`: we need to choose exactly `remaining_k` characters from `x` available characters (where `x` is the count of characters with this frequency), giving us `C(x, remaining_k)` ways
- Each selected character with frequency `val` contributes `val` choices for positions in the original string

The final answer is the product of all these choices: `ans = (product of frequencies > val) × C(x, remaining_k) × val^remaining_k`

This approach ensures we find all k-subsequences with maximum beauty by systematically counting all valid combinations of the highest-frequency characters.

**Learn more about [Greedy](/problems/greedy_intro), [Math](/problems/math-basics) and [Combinatorics](/problems/backtracking) patterns.**

## Solution Approach

The implementation follows these key steps:

1. **Character Frequency Counting**: Use a Counter (hash table) `f` to count the occurrence of each character in string `s`. This gives us `f[c]` for each character.

2. **Early Termination Check**: If the number of distinct characters (`len(f)`) is less than `k`, it's impossible to form a k-subsequence with unique characters, so return `0`.

3. **Sort Frequencies**: Extract all frequency values from `f` and sort them in descending order into array `vs`. This allows us to greedily select characters with highest frequencies.

4. **Find Cutoff Frequency**: The k-th largest frequency `val = vs[k-1]` determines our cutoff. Characters with this frequency may or may not be fully included in the optimal subsequences.

5. **Count Characters at Cutoff**: Count how many characters have frequency exactly equal to `val`. Store this as `x = vs.count(val)`.

6. **Calculate Answer in Two Parts**:
 
   **Part 1 - Characters with frequency > val**: 
   - Iterate through `vs` and for each frequency `v > val`:
     - Multiply answer by `v` (each occurrence can be chosen)
     - Decrement `k` to track remaining slots
   - After this loop, `k` represents how many characters with frequency `val` we need to select
 
   **Part 2 - Characters with frequency = val**:
   - We need to choose `k` characters from `x` available characters: multiply by `comb(x, k)`
   - Each selected character contributes `val` possible positions: multiply by `pow(val, k, mod)`

7. **Modular Arithmetic**: All multiplications are done modulo `10^9 + 7` to prevent overflow.

The final formula becomes:
```
ans = (∏ v for all v > val) × C(x, k) × val^k (mod 10^9 + 7)
```

This approach efficiently counts all k-subsequences with maximum beauty by:
- Using [greedy](/problems/greedy_intro) selection to identify which characters must be included
- Using [combinatorics](/problems/backtracking) to count different ways to select characters at the cutoff frequency
- Using modular exponentiation for efficient computation of large powers

## Example Walkthrough

Let's walk through an example with `s = "aabbccd"` and `k = 3`.

**Step 1: Count character frequencies**
- `f('a') = 2`
- `f('b') = 2` 
- `f('c') = 2`
- `f('d') = 1`

**Step 2: Check if k-subsequence is possible**
- We have 4 distinct characters, and k = 3, so it's possible (4 ≥ 3) ✓

**Step 3: Sort frequencies in descending order**
- `vs = [2, 2, 2, 1]`

**Step 4: Find the cutoff frequency**
- The k-th (3rd) largest frequency is `val = vs[k-1] = vs[2] = 2`
- This means we need at least characters with frequency ≥ 2

**Step 5: Count characters at cutoff**
- Characters with frequency exactly 2: `'a'`, `'b'`, `'c'` → count `x = 3`
- Characters with frequency > 2: none

**Step 6: Calculate the answer**

Since all top frequencies equal `val = 2`, we skip Part 1 and go directly to Part 2:
- We need to choose k = 3 characters from x = 3 available characters
- Number of ways to choose: `C(3, 3) = 1`
- Each chosen character has 2 occurrences in the string, so each contributes factor of 2
- Total combinations: `1 × 2^3 = 1 × 8 = 8`

**Verification of the 8 subsequences:**
The maximum beauty is 2 + 2 + 2 = 6 (choosing any 3 characters from {a, b, c})

Since we must choose all three characters {a, b, c}, and each appears twice in the string:
- For 'a': can choose index 0 or 1 (2 ways)
- For 'b': can choose index 2 or 3 (2 ways)  
- For 'c': can choose index 4 or 5 (2 ways)
- Total: 2 × 2 × 2 = 8 different k-subsequences

All 8 k-subsequences have the same character set "abc" but use different index combinations from the original string, each achieving the maximum beauty of 6.

## Python Solution

```python
from collections import Counter
from math import comb

class Solution:
    def countKSubsequencesWithMaxBeauty(self, s: str, k: int) -> int:
        # Count frequency of each character in the string
        char_frequency = Counter(s)
      
        # If there are fewer unique characters than k, no valid subsequence exists
        if len(char_frequency) < k:
            return 0
      
        MOD = 10**9 + 7
      
        # Sort character frequencies in descending order
        frequency_values = sorted(char_frequency.values(), reverse=True)
      
        # Find the k-th largest frequency value (threshold frequency)
        threshold_freq = frequency_values[k - 1]
      
        # Count how many characters have the threshold frequency
        num_chars_with_threshold = frequency_values.count(threshold_freq)
      
        # Initialize result to 1 (multiplicative identity)
        result = 1
      
        # Process all frequencies greater than the threshold
        # These characters must be included in every valid subsequence
        for freq in frequency_values:
            if freq == threshold_freq:
                break
            k -= 1  # Reduce remaining slots to fill
            result = (result * freq) % MOD
      
        # For characters with threshold frequency, we need to choose k from num_chars_with_threshold
        # Multiply by combinations * (threshold_freq ^ k) for the remaining slots
        result = (result * comb(num_chars_with_threshold, k) * pow(threshold_freq, k, MOD)) % MOD
      
        return result
```

## Time and Space Complexity

The time complexity is `O(n + |Σ| log |Σ|)`, where `n` is the length of the string and `|Σ|` is the size of the character set (at most 26 for lowercase letters).

- Creating the frequency counter takes `O(n)` time as we iterate through the string once
- Sorting the frequency values takes `O(|Σ| log |Σ|)` time, which is `O(26 log 26) = O(1)` for lowercase letters
- Counting occurrences of `val` takes `O(|Σ|)` time
- The first loop iterates at most `k` times, which is at most `|Σ|` times, taking `O(|Σ|)` time
- The combination calculation `comb(x, k)` takes `O(min(k, x-k))` time, which is at most `O(|Σ|)`
- The modular exponentiation `pow(val, k, mod)` takes `O(log k)` time

Since `|Σ| = 26` is a constant for lowercase letters, the overall time complexity simplifies to `O(n)`.

The space complexity is `O(|Σ|)`, where `|Σ|` is the size of the character set.

- The Counter object stores at most `|Σ|` key-value pairs
- The sorted values list `vs` contains at most `|Σ|` elements
- Other variables use `O(1)` space

For lowercase letters where `|Σ| = 26`, this is effectively `O(1)` constant space relative to the input size.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Handling of Integer Overflow**
The most critical pitfall is forgetting to apply modulo operations consistently throughout the calculation. The combination value `comb(num_chars_with_threshold, k)` can become extremely large, and multiplying it with other values without proper modulo handling will cause integer overflow.

**Problem Code:**
```python
# This will overflow for large values
result = (result * comb(num_chars_with_threshold, k) * pow(threshold_freq, k, MOD)) % MOD
```

**Solution:**
Apply modulo to the combination result before multiplication:
```python
combination_value = comb(num_chars_with_threshold, k) % MOD
result = (result * combination_value % MOD * pow(threshold_freq, k, MOD)) % MOD
```

### 2. **Misunderstanding the Beauty Calculation**
A common mistake is thinking that beauty is calculated based on character frequency within the subsequence itself, rather than the frequency in the original string.

**Wrong Interpretation:**
- Counting how many times 'b' appears in subsequence "bd" 
- This would give beauty = 1 + 1 = 2

**Correct Interpretation:**
- Using original string frequencies: f('b') = 3, f('d') = 2
- Beauty of "bd" = 3 + 2 = 5

### 3. **Off-by-One Error in Finding Threshold**
Using the wrong index when finding the k-th largest frequency can lead to incorrect results.

**Problem Code:**
```python
# Wrong: using k instead of k-1
threshold_freq = frequency_values[k]  # This gets the (k+1)-th element
```

**Solution:**
```python
# Correct: arrays are 0-indexed
threshold_freq = frequency_values[k - 1]
```

### 4. **Not Handling the Case Where All Characters Have Same Frequency**
When all characters have the same frequency and k equals the number of distinct characters, the algorithm might incorrectly calculate combinations.

**Example:** s = "abc", k = 3
- All frequencies are 1
- The loop for frequencies > threshold won't execute
- Must ensure k is properly maintained for the combination calculation

**Solution:**
Track k carefully through the algorithm and ensure the final combination calculation uses the correct remaining value of k.

### 5. **Confusion Between Character Count and Occurrence Count**
Mixing up the number of distinct characters with a certain frequency versus the frequency value itself.

**Problem Code:**
```python
# Wrong: using frequency value instead of count of characters
num_chars_with_threshold = threshold_freq  # This is the frequency, not the count
```

**Solution:**
```python
# Correct: count how many characters have this frequency
num_chars_with_threshold = frequency_values.count(threshold_freq)
```

### 6. **Inefficient Power Calculation**
Using regular Python exponentiation without modulo can cause performance issues and overflow.

**Problem Code:**
```python
# Inefficient and may overflow
result = (result * (threshold_freq ** k)) % MOD
```

**Solution:**
```python
# Use built-in modular exponentiation
result = (result * pow(threshold_freq, k, MOD)) % MOD
```

