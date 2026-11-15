# 1220. Count Vowels Permutation

> Source: https://algo.monster/liteproblems/1220

## Problem Statement

You need to count how many valid strings of length `n` can be formed using only lowercase vowels ('a', 'e', 'i', 'o', 'u'), where each vowel must follow specific rules about what vowel can come after it.

The rules are:
- After 'a': can only place 'e'
- After 'e': can only place 'a' or 'i'  
- After 'i': can place any vowel except 'i' (so 'a', 'e', 'o', or 'u')
- After 'o': can only place 'i' or 'u'
- After 'u': can only place 'a'

Given an integer `n`, you need to find the total number of valid strings of length `n` that can be formed following these rules. Since the answer can be very large, return the result modulo `10^9 + 7`.

For example, if `n = 1`, you can form 5 strings (one for each vowel: "a", "e", "i", "o", "u").

If `n = 2`, you need to count all valid 2-character combinations where the second character follows the rules based on what the first character is. For instance, "ae" is valid (since 'e' can follow 'a'), but "aa" is not valid (since only 'e' can follow 'a').

## Intuition

The key insight is to think about this problem in terms of [dynamic programming](/problems/dynamic_programming_intro), where we track how many valid strings end with each specific vowel at each step.

Instead of trying to generate all possible strings (which would be exponentially expensive), we can observe that what matters for building a string of length `n` is only the last character of strings of length `n-1`. This is because the rules only care about consecutive pairs of vowels.

Let's think backwards - if we want to know how many strings of length `n` end with a particular vowel, we need to know which vowels could have come before it. By inverting the given rules:

- Strings ending in 'a' could have come from strings ending in 'e', 'i', or 'u'
- Strings ending in 'e' could have come from strings ending in 'a' or 'i'  
- Strings ending in 'i' could have come from strings ending in 'e' or 'o'
- Strings ending in 'o' could have come from strings ending in 'i'
- Strings ending in 'u' could have come from strings ending in 'i' or 'o'

This naturally leads to a [dynamic programming](/problems/dynamic_programming_intro) approach where we maintain counts for strings ending with each vowel. At each step (increasing string length by 1), we calculate the new counts based on the previous counts and the rules above.

We start with length 1, where we have exactly one string for each vowel. Then for each subsequent length, we update our counts using the transition rules. The beauty of this approach is that we only need to track 5 numbers (one for each vowel) at any given time, making the solution very efficient with `O(n)` time and `O(1)` space complexity.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

We implement the [dynamic programming](/problems/dynamic_programming_intro) solution using an array to track the count of strings ending with each vowel.

We use an array `f` of size 5 where:
- `f[0]` represents count of strings ending with 'a'
- `f[1]` represents count of strings ending with 'e'
- `f[2]` represents count of strings ending with 'i'
- `f[3]` represents count of strings ending with 'o'
- `f[4]` represents count of strings ending with 'u'

Initially, for strings of length 1, we set `f = [1, 1, 1, 1, 1]` since we can have one string for each vowel.

For each subsequent length (from 2 to n), we create a new array `g` and calculate the new counts based on the transition rules:

```
g[0] = (f[1] + f[2] + f[4]) % mod  # 'a' can come from 'e', 'i', 'u'
g[1] = (f[0] + f[2]) % mod         # 'e' can come from 'a', 'i'
g[2] = (f[1] + f[3]) % mod         # 'i' can come from 'e', 'o'
g[3] = f[2]                         # 'o' can only come from 'i'
g[4] = (f[2] + f[3]) % mod         # 'u' can come from 'i', 'o'
```

After calculating the new counts in `g`, we update `f = g` for the next iteration.

We repeat this process `n-1` times (since we already handled length 1 initially). At each step, we take modulo `10^9 + 7` to prevent integer overflow.

Finally, the total number of valid strings of length `n` is the sum of all counts in the final array: `sum(f) % mod`.

The time complexity is `O(n)` since we iterate `n-1` times with constant work in each iteration. The space complexity is `O(1)` as we only use two arrays of fixed size 5.

## Example Walkthrough

Let's walk through the solution for `n = 3` to see how the dynamic programming approach works.

**Step 1: Initialize for n = 1**
We start with `f = [1, 1, 1, 1, 1]`, representing one string each ending with 'a', 'e', 'i', 'o', 'u'.
These are the strings: "a", "e", "i", "o", "u" (total: 5 strings)

**Step 2: Calculate for n = 2**
Now we apply the transition rules to find strings of length 2:

```
g[0] = f[1] + f[2] + f[4] = 1 + 1 + 1 = 3
  (strings ending in 'a': "ea", "ia", "ua")

g[1] = f[0] + f[2] = 1 + 1 = 2  
  (strings ending in 'e': "ae", "ie")

g[2] = f[1] + f[3] = 1 + 1 = 2
  (strings ending in 'i': "ei", "oi")

g[3] = f[2] = 1
  (strings ending in 'o': "io")

g[4] = f[2] + f[3] = 1 + 1 = 2
  (strings ending in 'u': "iu", "ou")
```

Update: `f = [3, 2, 2, 1, 2]` (total: 10 strings of length 2)

**Step 3: Calculate for n = 3**
Apply the transition rules again:

```
g[0] = f[1] + f[2] + f[4] = 2 + 2 + 2 = 6
  (6 strings ending in 'a': from "ae"→"aea", "ie"→"iea", 
   "ei"→"eia", "oi"→"oia", "iu"→"iua", "ou"→"oua")

g[1] = f[0] + f[2] = 3 + 2 = 5
  (5 strings ending in 'e': from "ea"→"eae", "ia"→"iae", "ua"→"uae",
   "ei"→"eie", "oi"→"oie")

g[2] = f[1] + f[3] = 2 + 1 = 3
  (3 strings ending in 'i': from "ae"→"aei", "ie"→"iei", "io"→"ioi")

g[3] = f[2] = 2
  (2 strings ending in 'o': from "ei"→"eio", "oi"→"oio")

g[4] = f[2] + f[3] = 2 + 1 = 3
  (3 strings ending in 'u': from "ei"→"eiu", "oi"→"oiu", "io"→"iou")
```

**Final Result:** `f = [6, 5, 3, 2, 3]`
Total valid strings of length 3 = 6 + 5 + 3 + 2 + 3 = **19**

This demonstrates how we efficiently count all valid strings without generating them explicitly, using only the counts from the previous step.

## Python Solution

```python
class Solution:
    def countVowelPermutation(self, n: int) -> int:
        # Each index represents a vowel: 0='a', 1='e', 2='i', 3='o', 4='u'
        # dp[i] stores count of strings ending with vowel i
        dp = [1] * 5
        MOD = 10**9 + 7
      
        # Build strings of length 2 to n
        for _ in range(n - 1):
            # Create new dp array for next iteration
            new_dp = [0] * 5
          
            # 'a' can follow 'e', 'i', or 'u'
            new_dp[0] = (dp[1] + dp[2] + dp[4]) % MOD
          
            # 'e' can follow 'a' or 'i'
            new_dp[1] = (dp[0] + dp[2]) % MOD
          
            # 'i' can follow 'e' or 'o'
            new_dp[2] = (dp[1] + dp[3]) % MOD
          
            # 'o' can follow 'i'
            new_dp[3] = dp[2]
          
            # 'u' can follow 'i' or 'o'
            new_dp[4] = (dp[2] + dp[3]) % MOD
          
            # Update dp for next iteration
            dp = new_dp
      
        # Return total count of all valid strings
        return sum(dp) % MOD
```

## Time and Space Complexity

The time complexity is `O(n)`, where `n` is the input parameter representing the length of the string to be formed. The algorithm uses a single loop that iterates `n-1` times, and within each iteration, it performs a constant number of operations (updating 5 array elements with simple arithmetic operations). Therefore, the total time complexity is linear with respect to `n`.

The space complexity is `O(1)` or equivalently `O(C)` where `C = 5` is the number of vowels. The algorithm uses two fixed-size arrays `f` and `g`, each containing exactly 5 elements regardless of the input size `n`. Since the space usage remains constant and does not grow with the input, the space complexity is constant.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. Confusing the Transition Direction

**The Pitfall:** The most common mistake is misunderstanding which vowel can follow which. The problem states "After 'a': can only place 'e'", meaning if we have 'a', the next character must be 'e'. However, when building the DP solution, we need to think in reverse: to count strings ending with a particular vowel, we need to look at which vowels can come BEFORE it.

For example:
- The rule "After 'a': can only place 'e'" means 'a' → 'e'
- In DP terms: to get strings ending with 'e', we can extend strings ending with 'a'
- So when calculating `new_dp[1]` (strings ending with 'e'), we include `dp[0]` (strings ending with 'a')

**Incorrect Implementation:**
```python
# WRONG: Thinking forward instead of backward
new_dp[1] = dp[0]  # Only 'a' can lead to 'e' - WRONG interpretation
new_dp[0] = dp[4]  # Only 'u' can lead to 'a' - Missing 'e' and 'i'
```

**Correct Implementation:**
```python
# RIGHT: Think about what can come BEFORE each vowel
new_dp[0] = (dp[1] + dp[2] + dp[4]) % MOD  # 'a' can follow 'e', 'i', 'u'
new_dp[1] = (dp[0] + dp[2]) % MOD          # 'e' can follow 'a', 'i'
```

### 2. Forgetting Modulo Operations

**The Pitfall:** Since the answer can be very large, forgetting to apply modulo at intermediate steps can cause integer overflow in languages with fixed integer sizes, or produce incorrect results.

**Incorrect Implementation:**
```python
# WRONG: No modulo during intermediate calculations
new_dp[0] = dp[1] + dp[2] + dp[4]  # Can overflow
# ...
return sum(dp) % MOD  # Only applying modulo at the end
```

**Correct Implementation:**
```python
# RIGHT: Apply modulo at each step
new_dp[0] = (dp[1] + dp[2] + dp[4]) % MOD
# ...
return sum(dp) % MOD
```

### 3. Off-by-One Error in Loop Count

**The Pitfall:** Since we initialize the DP array for strings of length 1, we need to iterate exactly `n-1` times to build strings of length `n`. Running the loop `n` times would give us strings of length `n+1`.

**Incorrect Implementation:**
```python
# WRONG: Iterating n times instead of n-1
for _ in range(n):  # This builds strings of length n+1
    # ... transitions ...
```

**Correct Implementation:**
```python
# RIGHT: Iterate n-1 times since we start with length 1
for _ in range(n - 1):
    # ... transitions ...
```

