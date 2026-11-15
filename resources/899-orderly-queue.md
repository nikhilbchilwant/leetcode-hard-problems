# 899. Orderly Queue

> Source: https://algo.monster/liteproblems/899

## Problem Statement

You have a string `s` and an integer `k`. You can perform the following operation any number of times:
- Choose one of the first `k` letters in the string
- Remove it from its current position
- Append it to the end of the string

Your goal is to find the lexicographically smallest string possible after performing any number of these operations.

For example, if `s = "cba"` and `k = 1`, you can only move the first character each time:
- Starting with `"cba"`
- Move 'c' to end: `"bac"`
- Move 'b' to end: `"acb"`
- Move 'a' to end: `"cba"` (back to original)

The smallest string achievable would be `"acb"`.

If `k ≥ 2`, you have more flexibility. You can choose from the first 2 (or more) characters to move, which gives you the ability to rearrange the string more freely.

## Intuition

Let's think about what operations we can actually perform with different values of `k`.

When `k = 1`, we're severely limited - we can only take the first character and move it to the end. This is essentially a rotation operation. If we keep doing this, we'll cycle through all possible rotations of the string. For a string of length `n`, after `n` operations we're back where we started. So with `k = 1`, we can only achieve `n` different strings (all rotations), and we need to find the lexicographically smallest one.

When `k = 2`, something interesting happens. We can now choose between the first two characters to move. This gives us much more power. Consider what happens when we want to swap two adjacent characters in the middle of the string, say we want to swap `x` and `y` in `abc[xy]def`:

1. Move `a`, `b`, `c` to the end one by one: `[xy]defabc`
2. Now `x` and `y` are at the front. Move `y` to the end: `[x]defabcy`
3. Move `x` to the end: `defabcy[x]`
4. Move `d`, `e`, `f` to the end: `abcy[x]def`
5. Continue rotating until we get: `abc[yx]def`

We've successfully swapped `x` and `y`! This means with `k ≥ 2`, we can swap any two adjacent characters.

And if we can swap adjacent characters, we can perform bubble sort! We can keep swapping adjacent characters until the entire string is sorted. This is why when `k > 1`, the answer is simply the sorted version of the string - it's the lexicographically smallest possible arrangement, and we can achieve it through a series of moves.

The solution elegantly handles both cases: rotate through all possibilities when `k = 1`, or return the sorted string when `k > 1`.

**Learn more about [Math](/problems/math-basics) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

The solution implements a case-by-case approach based on the value of `k`:

**Case 1: When `k = 1`**

Since we can only move the first character to the end, we need to check all possible rotations of the string. The implementation:

1. Initialize `ans` with the original string `s`
2. Perform `len(s) - 1` rotations (we don't need to do all `n` rotations since the last one would bring us back to the original)
3. In each iteration:
   - Move the first character to the end: `s = s[1:] + s[0]`
   - Compare with the current answer and keep the smaller one: `ans = min(ans, s)`
4. Return the lexicographically smallest string found

```python
if k == 1:
    ans = s
    for _ in range(len(s) - 1):
        s = s[1:] + s[0]
        ans = min(ans, s)
    return ans
```

**Case 2: When `k > 1`**

As established in the intuition, when `k > 1`, we can achieve any permutation of the string. The lexicographically smallest permutation is simply the sorted string.

```python
return "".join(sorted(s))
```

The `sorted(s)` function converts the string into a list of characters, sorts them in ascending order, and `"".join()` concatenates them back into a string.

**Time Complexity:**
- For `k = 1`: `O(n²)` where `n` is the length of the string (we perform `n - 1` rotations, each taking `O(n)` time for string operations and comparison)
- For `k > 1`: `O(n log n)` due to [sorting](/problems/sorting_summary)

**Space Complexity:**
- For `k = 1`: `O(n)` for storing the answer string
- For `k > 1`: `O(n)` for the sorted character list

## Example Walkthrough

Let's walk through two examples to illustrate how the solution works for different values of `k`.

**Example 1: `s = "dcba"`, `k = 1`**

Since `k = 1`, we can only move the first character to the end. Let's trace through all rotations:

1. Start with `"dcba"` (initialize `ans = "dcba"`)
2. First rotation: Move 'd' to end → `"cbad"`
   - Compare: `min("dcba", "cbad")` = `"cbad"` (update `ans = "cbad"`)
3. Second rotation: Move 'c' to end → `"badc"`
   - Compare: `min("cbad", "badc")` = `"badc"` (update `ans = "badc"`)
4. Third rotation: Move 'b' to end → `"adcb"`
   - Compare: `min("badc", "adcb")` = `"adcb"` (update `ans = "adcb"`)

We've checked all meaningful rotations. The answer is `"adcb"`.

**Example 2: `s = "dcba"`, `k = 2`**

Since `k > 1`, we can achieve any permutation of the string through a series of moves. The lexicographically smallest permutation is the sorted string.

1. Sort the characters: ['d', 'c', 'b', 'a'] → ['a', 'b', 'c', 'd']
2. Join them: `"abcd"`

The answer is `"abcd"`.

Notice how with `k = 2`, we get a better result (`"abcd"`) than with `k = 1` (`"adcb"`). This demonstrates the increased flexibility when we can choose from multiple characters at the front of the string.

## Python Solution

```python
class Solution:
    def orderlyQueue(self, s: str, k: int) -> str:
        """
        Find the lexicographically smallest string that can be obtained by repeatedly:
        - Choose one of the first k letters and move it to the end of the string
      
        Args:
            s: Input string to be rearranged
            k: Number of first letters we can choose from to move to the end
          
        Returns:
            Lexicographically smallest string possible
        """
      
        if k == 1:
            # When k=1, we can only rotate the string
            # Try all possible rotations and find the minimum
            min_string = s
          
            # Generate all rotations by moving first character to end
            for _ in range(len(s) - 1):
                s = s[1:] + s[0]  # Move first character to end
                min_string = min(min_string, s)  # Keep track of minimum
              
            return min_string
      
        else:
            # When k >= 2, we can achieve any permutation
            # The smallest permutation is the sorted string
            return "".join(sorted(s))
```

## Time and Space Complexity

The time complexity depends on the value of `k`:

- When `k = 1`: The code performs `n-1` rotations of the string, where each rotation involves string slicing and concatenation operations that take `O(n)` time. Additionally, each comparison with `min()` takes `O(n)` time. Therefore, the total time complexity is `O(n^2)`.

- When `k > 1`: The code simply sorts the string using `sorted()`, which has a time complexity of `O(n log n)`.

Overall, the worst-case time complexity is `O(n^2)` when `k = 1`.

The space complexity is `O(n)` in both cases:
- When `k = 1`: The variable `ans` stores a copy of the string, and string slicing operations create new strings, requiring `O(n)` space.
- When `k > 1`: The `sorted()` function creates a new list of characters requiring `O(n)` space, and `join()` creates a new string of length `n`.

Therefore, the time complexity is `O(n^2)` and the space complexity is `O(n)`, where `n` is the length of the string.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Misunderstanding the k > 1 Case

**The Mistake:**
Many people initially think that when `k = 2`, you can only perform limited rearrangements, similar to `k = 1`. They might try to implement complex logic to track which characters can be moved based on the current first `k` positions.

**Why It Happens:**
The problem statement makes it seem like `k = 2` is just slightly more flexible than `k = 1`, but the reality is that `k ≥ 2` allows you to achieve ANY permutation of the string through a series of moves.

**The Mathematical Insight:**
When `k ≥ 2`, you can effectively "bubble sort" the string:
- You can always bring any character to the front by strategically moving other characters to the back
- This is possible because with 2 positions to choose from, you can preserve one character while cycling others

**Solution:**
Recognize that for `k ≥ 2`, the answer is always the sorted string:
```python
if k >= 2:
    return "".join(sorted(s))
```

### Pitfall 2: Inefficient Rotation Implementation for k = 1

**The Mistake:**
Using string concatenation in a loop without considering the number of iterations needed:
```python
# Inefficient: Checking all n rotations when n-1 is sufficient
ans = s
for i in range(len(s)):  # Should be range(len(s) - 1)
    s = s[1:] + s[0]
    ans = min(ans, s)
```

**Why It Happens:**
After `n` rotations (where `n = len(s)`), the string returns to its original form. The last rotation is redundant.

**Solution:**
Only perform `n-1` rotations since the `n`-th rotation brings you back to the original string:
```python
ans = s
for _ in range(len(s) - 1):  # Note: len(s) - 1, not len(s)
    s = s[1:] + s[0]
    ans = min(ans, s)
```

### Pitfall 3: Not Considering the Original String for k = 1

**The Mistake:**
Forgetting to include the original string in the comparison:
```python
# Wrong: Starting with the first rotation and doing only n - 1 rotations
ans = s[1:] + s[0]
for _ in range(len(s) - 1):
    s = s[1:] + s[0]
    ans = min(ans, s)
# For s = "abc", this checks "bca" and "cab" only and returns "bca" instead of "abc"
```

**Why It Happens:**
The original string might already be the lexicographically smallest rotation, and skipping it would give an incorrect answer. (A loop that runs all `n` rotations does include the original string, since the `n`-th rotation brings it back, so starting from an empty `ans` in that case still works.)

**Solution:**
Always initialize the answer with the original string before starting rotations:
```python
ans = s  # Start with original string
for _ in range(len(s) - 1):
    s = s[1:] + s[0]
    ans = min(ans, s)
```

