# 3333. Find the Original Typed String II

> Source: https://algo.monster/liteproblems/3333

## Problem Statement

Alice is trying to type a string on her computer, but she's clumsy and sometimes holds a key down too long, causing a character to be typed multiple times in a row.

You're given:
- `word`: the final string that appears on Alice's screen (containing the repeated characters)
- `k`: a positive integer representing the minimum length of the original string Alice intended to type

Your task is to find how many different original strings Alice could have been trying to type, given that:
- The original string must have at least `k` characters
- Any character in `word` could be the result of Alice pressing that key one or more times
- Consecutive identical characters in `word` could have come from Alice intending to type that character any number of times (from 1 up to the total count of consecutive occurrences)

For example, if `word = "aaa"`, Alice could have intended to type:
- "a" (pressed 'a' once but held it too long, resulting in "aaa")
- "aa" (pressed 'a' twice, with at least one held too long)
- "aaa" (pressed 'a' three times)

The answer should be returned modulo `10^9 + 7` since it can be very large.

The problem essentially asks you to count all possible ways to "compress" the given string by reducing consecutive character groups, while ensuring the resulting string has at least `k` characters.

## Intuition

The key insight is to recognize that this is a counting problem with constraints. Let's think about it step by step.

First, observe that consecutive identical characters form groups. For example, `"aaabbcc"` has three groups: `"aaa"`, `"bb"`, and `"cc"`. For each group of size `n`, Alice could have intended to type anywhere from `1` to `n` characters. So a group of `3` identical characters gives us `3` choices.

Without any length constraint, the total number of possibilities would simply be the product of choices for each group. If we have groups of sizes `3`, `2`, and `2`, we'd have `3 × 2 × 2 = 12` total possibilities.

But here's the twist: we need the original string to have at least `k` characters. This constraint makes things tricky. We can reformulate this as:
- Total possibilities = (All possibilities) - (Possibilities with length < k)

Let's denote:
- `a` = total number of ways without length restriction
- `b` = number of ways with length less than `k`
- Answer = `a - b`

For calculating `a`, it's straightforward - multiply the number of choices for each group.

For calculating `b`, we need [dynamic programming](/problems/dynamic_programming_intro). Why? Because we need to count all combinations where the total selected characters is less than `k`. This is a classic DP problem where we need to track:
- How many groups we've processed
- How many total characters we've selected so far

The DP state `f[i][j]` represents the number of ways to select exactly `j` characters from the first `i` groups. For each group with `x` extra selectable characters (group size - 1, since we must select at least 1), we can choose anywhere from `0` to `min(x, j)` additional characters.

The [prefix sum](/problems/subarray_sum) optimization comes from the observation that when transitioning from group `i-1` to group `i`, we're essentially summing over a range of previous states. Instead of iterating through each possibility, we can use prefix sums to calculate this range sum in constant time.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) and [Prefix Sum](/problems/subarray_sum) patterns.**

## Solution Approach

Let's walk through the implementation step by step:

**Step 1: Group Consecutive Characters**

First, we iterate through `word` and identify groups of consecutive identical characters. For each group:
- Count its size
- If the group has more than 1 character, it means we have choices (we can select 1 to `size` characters)
- Store `size - 1` in the `nums` array (representing extra selectable characters beyond the mandatory 1)
- Multiply `ans` by the group size to accumulate total possibilities without restriction
- Decrement `k` by 1 for each group (since we must select at least 1 character per group)

```python
for i, c in enumerate(word):
    cur += 1
    if i == len(word) - 1 or c != word[i + 1]:
        if cur > 1:
            if k > 0:
                nums.append(cur - 1)
            ans = ans * cur % mod
        cur = 0
        k -= 1
```

**Step 2: Check Early Termination**

If `k < 1` after selecting one character from each group, it means we already satisfy the minimum length requirement. Return `ans` directly.

```python
if k < 1:
    return ans
```

**Step 3: [Dynamic Programming](/problems/dynamic_programming_intro) Setup**

Create a 2D DP table `f[m+1][k]` where:
- `m` is the number of groups with extra selectable characters
- `f[i][j]` = number of ways to select exactly `j` characters from the first `i` groups
- Initialize `f[0][0] = 1` (one way to select 0 characters from 0 groups)

```python
m = len(nums)
f = [[0] * k for _ in range(m + 1)]
f[0][0] = 1
```

**Step 4: DP Transition with [Prefix Sum](/problems/subarray_sum) Optimization**

For each group `i` with `x` extra selectable characters:
- Build a [prefix sum](/problems/subarray_sum) array `s` from the previous row `f[i-1]`
- For each target sum `j`, we can select `l ∈ [0, min(x, j)]` characters from this group
- This means `f[i][j]` = sum of `f[i-1][j-l]` for all valid `l`
- Using prefix sums: `f[i][j] = s[j+1] - s[j-min(x,j)]`

```python
for i, x in enumerate(nums, 1):
    s = list(accumulate(f[i - 1], initial=0))
    for j in range(k):
        f[i][j] = (s[j + 1] - s[j - min(x, j)] + mod) % mod
```

**Step 5: Calculate Final Answer**

The number of ways with length less than `k` is the sum of `f[m][j]` for all `j < k`. Subtract this from the total possibilities:

```python
return (ans - sum(f[m][j] for j in range(k))) % mod
```

The time complexity is `O(n + m × k)` where `n` is the length of `word` and `m` is the number of groups. The space complexity is `O(m × k)` for the DP table.

## Example Walkthrough

Let's walk through a concrete example with `word = "aaabbc"` and `k = 3`.

**Step 1: Identify Groups**
- Group 1: "aaa" (size 3)
- Group 2: "bb" (size 2) 
- Group 3: "c" (size 1)

**Step 2: Calculate Total Possibilities Without Restriction**
- Group "aaa": can type 1, 2, or 3 'a's → 3 choices
- Group "bb": can type 1 or 2 'b's → 2 choices
- Group "c": can only type 1 'c' → 1 choice
- Total: 3 × 2 × 1 = 6 possibilities

These 6 possibilities are:
1. "abc" (1a + 1b + 1c)
2. "abbc" (1a + 2b + 1c)
3. "aabc" (2a + 1b + 1c)
4. "aabbc" (2a + 2b + 1c)
5. "aaabc" (3a + 1b + 1c)
6. "aaabbc" (3a + 2b + 1c)

**Step 3: Adjust k and Build nums Array**
- We must select at least 1 character from each group
- After mandatory selections: k = 3 - 3 = 0
- Since k = 0, we need exactly the minimum (1 from each group)
- nums = [2, 1] (extra characters we can select: 3-1=2 for "aaa", 2-1=1 for "bb")

**Step 4: DP to Count Strings with Length < 3**
We need to count combinations with total length less than 3 (i.e., length 0, 1, or 2 after mandatory selections).

DP table f[i][j] = ways to select exactly j extra characters from first i groups:

Initial: f[0][0] = 1

Processing Group 1 (can select 0-2 extra):
- f[1][0] = 1 (select 0 extra from "aaa")
- f[1][1] = 1 (select 1 extra from "aaa")
- f[1][2] = 1 (select 2 extra from "aaa")

Processing Group 2 (can select 0-1 extra):
- f[2][0] = f[1][0] = 1 (select 0 from both)
- f[2][1] = f[1][1] + f[1][0] = 2 (select 1 total: either 1 from first or 1 from second)
- f[2][2] = f[1][2] + f[1][1] = 2 (select 2 total: either 2 from first, or 1 from each)

Since k was adjusted to 0, we're looking for combinations with negative total length, which is 0.

**Step 5: Final Answer**
- Total possibilities: 6
- Possibilities with length < 3: 0
- Answer: 6 - 0 = 6

All 6 strings have at least 3 characters, so the answer is 6.

---

Let's try another example with `word = "aaaa"` and `k = 2`:

**Groups:** Just one group "aaaa" (size 4)
**Total possibilities:** 4 (can type 1, 2, 3, or 4 'a's)
**After mandatory selection:** k = 2 - 1 = 1
**nums = [3]** (can select 0-3 extra characters)

**DP to count strings with length < 2:**
- f[0][0] = 1
- f[1][0] = 1 (select 0 extra → string "a" has length 1)
- f[1][1] = 1 (select 1 extra → string "aa" has length 2)

Strings with length < 2: f[1][0] = 1 (just "a")

**Answer:** 4 - 1 = 3

The 3 valid strings are: "aa", "aaa", "aaaa" (all have length ≥ 2)

## Python Solution

```python
class Solution:
    def possibleStringCount(self, word: str, k: int) -> int:
        MOD = 10**9 + 7
      
        # Store the count of extra characters for each group (count - 1)
        extra_chars_per_group = []
        total_combinations = 1
        current_group_size = 0
      
        # Group consecutive identical characters
        for i, char in enumerate(word):
            current_group_size += 1
          
            # Check if we're at the end of a group (last char or next char is different)
            if i == len(word) - 1 or char != word[i + 1]:
                # If group has more than 1 character, it contributes to combinations
                if current_group_size > 1:
                    if k > 0:
                        # Store how many extra chars we can choose from this group
                        extra_chars_per_group.append(current_group_size - 1)
                    # Multiply total combinations by group size
                    total_combinations = (total_combinations * current_group_size) % MOD
              
                # Reset for next group
                current_group_size = 0
                k -= 1  # Decrease k for each group (minimum 1 char per group)
      
        # If k is already satisfied with minimum characters, return total combinations
        if k < 1:
            return total_combinations
      
        # Dynamic programming to count invalid combinations (those with < k total chars)
        num_groups = len(extra_chars_per_group)
      
        # dp[i][j] = number of ways to select chars from first i groups with exactly j extra chars
        dp = [[0] * k for _ in range(num_groups + 1)]
        dp[0][0] = 1  # Base case: 0 groups, 0 extra chars = 1 way
      
        # Process each group
        for i, max_extra in enumerate(extra_chars_per_group, 1):
            # Use prefix sum for optimization
            prefix_sum = list(accumulate(dp[i - 1], initial=0))
          
            for j in range(k):
                # We can choose 0 to min(max_extra, j) extra chars from current group
                # This uses prefix sum to calculate range sum efficiently
                dp[i][j] = (prefix_sum[j + 1] - prefix_sum[j - min(max_extra, j)] + MOD) % MOD
      
        # Subtract invalid combinations (those with < k chars) from total
        invalid_combinations = sum(dp[num_groups][j] for j in range(k))
        return (total_combinations - invalid_combinations) % MOD
```

## Time and Space Complexity

**Time Complexity:** `O(n + m*k^2)` where `n` is the length of the string `word`, `m` is the number of consecutive character groups (size of `nums` array), and `k` is the input parameter.

- The first loop iterates through the string once: `O(n)`
- The nested loops for dynamic programming iterate `m` times (for each element in `nums`), and for each iteration:
  - Computing prefix sum using `accumulate`: `O(k)`  
  - Inner loop runs `k` times, and each iteration involves constant time operations with prefix sum lookup: `O(k)`
  - Total for this part: `O(m*k)`
- Final sum computation: `O(k)`
- Overall: `O(n + m*k)`, and since `m ≤ n`, this simplifies to `O(n + m*k)`

However, the reference answer states `O(n + k^2)`. This is correct when considering that `m ≤ k` in the worst case that matters (when we need to compute the DP table). If `m > k`, the function returns early with just `ans`, making the complexity `O(n)`. When `m ≤ k`, the DP computation dominates with `O(m*k) ≤ O(k^2)`.

**Space Complexity:** `O(m*k)` which can be simplified to `O(k^2)` under the same reasoning.

- The `nums` array stores at most `m` elements: `O(m)`
- The DP table `f` has dimensions `(m+1) × k`: `O(m*k)`
- The prefix sum array `s` has size `k+1`: `O(k)`
- Total: `O(m*k)`, which becomes `O(k^2)` when `m ≤ k`

The reference answer's `O(k^2)` space complexity is correct considering that when the DP table is actually computed (i.e., when `k` is not reduced to below 1), we have `m ≤ k`, making the space complexity `O(k^2)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Modulo Operation During Subtraction**

**Pitfall:** When calculating the final answer by subtracting invalid combinations from total combinations, a common mistake is:
```python
# WRONG - can produce negative results
return (total_combinations - invalid_combinations) % MOD
```

If `invalid_combinations > total_combinations` (which shouldn't happen logically but can occur due to modulo operations during intermediate calculations), this produces a negative number, leading to incorrect results in languages like Python where `%` preserves sign.

**Solution:** Always add MOD before taking modulo when subtracting:
```python
# CORRECT - ensures positive result
return (total_combinations - invalid_combinations % MOD + MOD) % MOD
```

### 2. **Off-by-One Error in k Adjustment**

**Pitfall:** Forgetting to decrement `k` for each group or decrementing it incorrectly:
```python
# WRONG - only decrements k when group size > 1
if current_group_size > 1:
    k -= 1
```

Every group (even single characters) contributes at least 1 character to the minimum length, so `k` should be decremented for each group regardless of size.

**Solution:** Decrement `k` outside the conditional check:
```python
# CORRECT - decrements for every group
if i == len(word) - 1 or char != word[i + 1]:
    if current_group_size > 1:
        # handle extra characters
    k -= 1  # This should be outside the if statement
```

### 3. **Overflow in Prefix Sum Range Calculation**

**Pitfall:** When calculating the range sum using prefix sums, accessing negative indices:
```python
# WRONG - can access negative index when j < min(max_extra, j)
dp[i][j] = prefix_sum[j + 1] - prefix_sum[j - min(max_extra, j)]
```

**Solution:** Ensure the index calculation is correct:
```python
# CORRECT - max(0, ...) ensures non-negative index
start_idx = max(0, j - min(max_extra, j))
dp[i][j] = (prefix_sum[j + 1] - prefix_sum[start_idx] + MOD) % MOD
```

### 4. **Not Handling Edge Case When No Groups Have Extra Characters**

**Pitfall:** If all groups have exactly 1 character and `k` is still positive after processing, the DP won't execute (empty `extra_chars_per_group`), but we still need to return 0 since we can't meet the minimum length requirement.

**Solution:** Check if it's possible to meet the requirement:
```python
if k > 0 and len(extra_chars_per_group) == 0:
    return 0  # Impossible to reach k characters
```

### 5. **Integer Overflow in Total Combinations Calculation**

**Pitfall:** Forgetting to apply modulo during multiplication:
```python
# WRONG - can overflow in languages with fixed integer sizes
total_combinations = total_combinations * current_group_size
```

**Solution:** Apply modulo after each multiplication:
```python
# CORRECT - prevents overflow
total_combinations = (total_combinations * current_group_size) % MOD
```

