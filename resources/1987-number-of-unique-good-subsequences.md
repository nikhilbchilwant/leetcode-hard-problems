# 1987. Number of Unique Good Subsequences

> Source: https://algo.monster/liteproblems/1987

## Problem Statement

You are given a binary string containing only '0's and '1's. Your task is to find the number of **unique good subsequences** from this string.

A subsequence is considered **good** if:
- It is not empty
- It has no leading zeros (except for the single character `"0"` itself)

For example, given `binary = "001"`:
- All possible subsequences that can be formed are: `"0"` (from index 0), `"0"` (from index 1), `"1"` (from index 2), `"00"`, `"01"`, and `"001"`
- Among these, `"00"`, `"01"`, and `"001"` are **not good** because they have leading zeros
- The good subsequences are: `"0"`, `"0"`, `"1"`
- The **unique** good subsequences are: `"0"` and `"1"`
- Therefore, the answer is 2

Key points to understand:
- A subsequence maintains the relative order of characters but doesn't need to be consecutive
- The string `"0"` by itself is considered good (it's the exception to the "no leading zeros" rule)
- Any string starting with `"1"` is good
- Any string starting with `"0"` (except `"0"` itself) is not good
- We need to count only unique subsequences, not all occurrences

Since the answer can be very large, return it modulo `10^9 + 7`.

## Intuition

Let's think about how to build unique good subsequences as we scan through the binary string character by character.

The key insight is that we need to track subsequences based on what they end with, because when we encounter a new character, we can potentially append it to all existing subsequences to create new ones.

Since good subsequences cannot have leading zeros (except for `"0"` itself), we need to categorize our subsequences:
- Subsequences that end with `'1'` - these can always be extended
- Subsequences that end with `'0'` but start with `'1'` - these are also valid good subsequences

Why this categorization? Because:
1. Any subsequence starting with `'1'` is always good, regardless of what follows
2. The only good subsequence starting with `'0'` is `"0"` itself

As we traverse the string:
- When we see a `'0'`: We can append it to all existing good subsequences (both those ending in `'1'` and those ending in `'0'`). This creates new subsequences ending in `'0'`.
- When we see a `'1'`: We can append it to all existing good subsequences, and additionally, we can start a new subsequence with just this `'1'`.

Let's use two variables:
- `f`: count of unique good subsequences ending with `'1'`
- `g`: count of unique good subsequences ending with `'0'` (but starting with `'1'`)

When we encounter `'0'`:
- All subsequences in `f` can be extended with `'0'`, creating `f` new subsequences ending in `'0'`
- All subsequences in `g` can be extended with `'0'`, but they still end in `'0'`
- So `g = g + f`

When we encounter `'1'`:
- All subsequences in `f` can be extended with `'1'`, creating `f` new subsequences ending in `'1'`
- All subsequences in `g` can be extended with `'1'`, creating `g` new subsequences ending in `'1'`
- We can also start a new subsequence with just this `'1'`
- So `f = f + g + 1`

Finally, if the string contains at least one `'0'`, we add 1 to account for the single `"0"` subsequence, which is the special case of a good subsequence starting with `'0'`.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

We implement a [dynamic programming](/problems/dynamic_programming_intro) solution using two variables to track distinct good subsequences:

**Variables:**
- `f`: Count of unique good subsequences ending with `'1'`
- `g`: Count of unique good subsequences ending with `'0'` (but starting with `'1'`)
- `ans`: Flag to track if we've seen at least one `'0'` (to add the single `"0"` subsequence at the end)
- `mod = 10^9 + 7`: For modulo operation

**Algorithm:**

1. **Initialize** `f = g = 0` and `ans = 0`

2. **Traverse each character** `c` in the binary string:
 
   - **If `c = '0'`:**
     - Update `g = (g + f) % mod`
     - This means we can append `'0'` to all `f` subsequences ending with `'1'`, creating new subsequences ending with `'0'`
     - Set `ans = 1` to mark that we've seen a `'0'`
 
   - **If `c = '1'`:**
     - Update `f = (f + g + 1) % mod`
     - We can append `'1'` to:
       - All `f` subsequences ending with `'1'` (creates `f` new ones)
       - All `g` subsequences ending with `'0'` (creates `g` new ones)
       - Start a new subsequence with just this `'1'` (adds `1`)

3. **Calculate final answer:**
   - `ans = (ans + f + g) % mod`
   - This adds:
     - `f`: all unique subsequences ending with `'1'`
     - `g`: all unique subsequences ending with `'0'` (starting with `'1'`)
     - `ans`: the single `"0"` subsequence if we've seen any `'0'`

**Example walkthrough with `binary = "001"`:**

- Initially: `f = 0`, `g = 0`, `ans = 0`
- Character `'0'` at index 0:
  - `g = (0 + 0) = 0`
  - `ans = 1`
- Character `'0'` at index 1:
  - `g = (0 + 0) = 0`
  - `ans = 1`
- Character `'1'` at index 2:
  - `f = (0 + 0 + 1) = 1`
- Final: `ans = (1 + 1 + 0) = 2`

The answer is 2, representing the unique good subsequences `"0"` and `"1"`.

**Time Complexity:** `O(n)` where `n` is the length of the binary string
**Space Complexity:** `O(1)` as we only use a constant amount of extra space

## Example Walkthrough

Let's walk through the solution with `binary = "101"`:

**Initial state:**
- `f = 0` (subsequences ending with '1')
- `g = 0` (subsequences ending with '0', starting with '1')
- `ans = 0` (haven't seen a '0' yet)

**Step 1: Process '1' at index 0**
- Since it's a '1', we update: `f = f + g + 1 = 0 + 0 + 1 = 1`
- This represents the new subsequence "1"
- Current subsequences: {"1"}
- State: `f = 1, g = 0, ans = 0`

**Step 2: Process '0' at index 1**
- Since it's a '0', we update: `g = g + f = 0 + 1 = 1`
- We can append '0' to "1" to get "10"
- Set `ans = 1` (we've now seen a '0')
- Current subsequences: {"1", "10"}
- State: `f = 1, g = 1, ans = 1`

**Step 3: Process '1' at index 2**
- Since it's a '1', we update: `f = f + g + 1 = 1 + 1 + 1 = 3`
- We can:
  - Append '1' to "1" → "11" (from f)
  - Append '1' to "10" → "101" (from g)
  - Start new subsequence "1" (the +1)
- Current subsequences: {"1", "10", "11", "101"}
- State: `f = 3, g = 1, ans = 1`

**Final calculation:**
- `ans = ans + f + g = 1 + 3 + 1 = 5`
- The 5 unique good subsequences are:
  1. "0" (the special case, counted in ans)
  2. "1" (ending with '1', in f)
  3. "11" (ending with '1', in f)
  4. "101" (ending with '1', in f)
  5. "10" (ending with '0' but starting with '1', in g)

The answer is **5**.

## Python Solution

```python
class Solution:
    def numberOfUniqueGoodSubsequences(self, binary: str) -> int:
        # Dynamic programming approach to count unique subsequences
        # subsequences_ending_with_one: count of unique subsequences ending with '1'
        # subsequences_ending_with_zero: count of unique subsequences ending with '0'
        subsequences_ending_with_one = 0
        subsequences_ending_with_zero = 0
        has_zero = 0  # Flag to track if we've seen a '0' (to add the single "0" subsequence)
        MOD = 10**9 + 7
      
        # Process each character in the binary string
        for char in binary:
            if char == "0":
                # When we see a '0', we can append it to all subsequences ending with '1'
                # This creates new subsequences ending with '0'
                subsequences_ending_with_zero = (subsequences_ending_with_zero + subsequences_ending_with_one) % MOD
                # Mark that we've seen at least one '0' (for the single "0" subsequence)
                has_zero = 1
            else:  # char == "1"
                # When we see a '1', we can:
                # 1. Append it to all subsequences ending with '1'
                # 2. Append it to all subsequences ending with '0'
                # 3. Start a new subsequence with just this '1'
                subsequences_ending_with_one = (subsequences_ending_with_one + subsequences_ending_with_zero + 1) % MOD
      
        # Total count includes:
        # - All subsequences ending with '1'
        # - All subsequences ending with '0'
        # - The single "0" subsequence if we've seen any '0'
        total_subsequences = (has_zero + subsequences_ending_with_one + subsequences_ending_with_zero) % MOD
      
        return total_subsequences
```

## Time and Space Complexity

**Time Complexity:** `O(n)` where `n` is the length of the binary string.

The algorithm iterates through the binary string exactly once with a single for loop. Within each iteration, it performs a constant number of operations (comparisons, additions, and modulo operations). Therefore, the time complexity is linear with respect to the input size.

**Space Complexity:** `O(1)`

The algorithm uses only a fixed number of variables (`f`, `g`, `ans`, and `mod`) regardless of the input size. No additional data structures that scale with the input are created. The space usage remains constant throughout the execution, making the space complexity constant.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Incorrectly Handling the Single "0" Subsequence

**The Problem:**
A common mistake is forgetting that the single character "0" is a valid good subsequence (it's the only exception to the "no leading zeros" rule). Many solutions might incorrectly exclude all subsequences starting with '0', including the single "0" itself.

**Incorrect Approach:**
```python
# Wrong: This would miss counting the single "0" as a valid subsequence
def numberOfUniqueGoodSubsequences(self, binary: str) -> int:
    f = g = 0
    MOD = 10**9 + 7
  
    for char in binary:
        if char == "0":
            g = (g + f) % MOD
        else:
            f = (f + g + 1) % MOD
  
    # Missing the single "0" subsequence!
    return (f + g) % MOD
```

**Solution:**
Use a flag variable (`has_zero`) to track whether we've encountered at least one '0' in the string. Add this flag to the final count to include the single "0" subsequence when applicable.

### Pitfall 2: Double Counting Subsequences

**The Problem:**
When updating the counts for subsequences ending with '0' or '1', it's easy to accidentally count the same subsequence multiple times or update variables in the wrong order, leading to incorrect results.

**Incorrect Approach:**
```python
# Wrong: Updating both variables simultaneously can lead to using already-modified values
def numberOfUniqueGoodSubsequences(self, binary: str) -> int:
    f = g = 0
    has_zero = 0
    MOD = 10**9 + 7
  
    for char in binary:
        if char == "0":
            # Wrong: If we update f here too, we'd be counting incorrectly
            g = (g + f) % MOD
            f = (f + 1) % MOD  # This is wrong!
            has_zero = 1
        else:
            f = (f + g + 1) % MOD
  
    return (has_zero + f + g) % MOD
```

**Solution:**
Only update the appropriate variable for each character type:
- When seeing '0': Only update `g` (subsequences ending with '0')
- When seeing '1': Only update `f` (subsequences ending with '1')

### Pitfall 3: Forgetting the Modulo Operation

**The Problem:**
The problem states that the answer can be very large and should be returned modulo 10^9 + 7. Forgetting to apply the modulo operation at each step can lead to integer overflow in languages with fixed integer sizes, or simply returning the wrong answer.

**Incorrect Approach:**
```python
# Wrong: Missing modulo operations during intermediate calculations
def numberOfUniqueGoodSubsequences(self, binary: str) -> int:
    f = g = 0
    has_zero = 0
    MOD = 10**9 + 7
  
    for char in binary:
        if char == "0":
            g = g + f  # Missing modulo!
            has_zero = 1
        else:
            f = f + g + 1  # Missing modulo!
  
    return (has_zero + f + g) % MOD  # Only applying modulo at the end
```

**Solution:**
Apply the modulo operation after every arithmetic operation that could potentially exceed the modulo value. This ensures the numbers stay within manageable bounds throughout the computation.

