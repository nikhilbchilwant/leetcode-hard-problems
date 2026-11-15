# 1088. Confusing Number II

> Source: https://algo.monster/liteproblems/1088

## Problem Statement

A **confusing number** is a number that when rotated 180 degrees becomes a different number with each digit valid.

When we rotate individual digits by 180 degrees:
- `0` remains `0`
- `1` remains `1`
- `6` becomes `9`
- `8` remains `8`
- `9` becomes `6`
- `2`, `3`, `4`, `5`, and `7` become invalid (cannot be rotated)

To determine if a number is confusing:
1. Each digit in the number must be rotatable (only `0`, `1`, `6`, `8`, `9` are valid)
2. After rotating the entire number 180 degrees, it must result in a different number than the original

For example:
- `69` is a confusing number because when rotated it becomes `96`, which is different from `69`
- `88` is NOT a confusing number because when rotated it remains `88`
- `25` is NOT a confusing number because `2` and `5` cannot be rotated

When rotating a number, leading zeros in the result are ignored. For instance, `8000` rotated becomes `0008`, which is treated as just `8`.

Given an integer `n`, you need to count how many confusing numbers exist in the range `[1, n]` inclusive.

## Intuition

The key insight is that we need to count valid confusing numbers efficiently without checking every single number from 1 to n. Since only certain digits (0, 1, 6, 8, 9) can be rotated, we can build valid candidates digit by digit rather than checking all numbers.

Think of it like building a number from left to right. At each position, we only have 5 choices (the rotatable digits), not all 10 digits. This immediately reduces our search space significantly.

The challenge is ensuring we don't exceed `n` while building these numbers. This is where the "digit DP with limit" technique comes in. When we're building a number digit by digit, we need to track whether we're still bounded by the corresponding digit in `n`. For example, if `n = 234`, and we've built `2` so far, the next digit can only be 0, 1, or 3 (not 6, 8, or 9) to stay within the limit.

The [backtracking](/problems/backtracking) approach naturally fits here because:
1. We make a choice (pick a valid digit)
2. Explore further (move to the next position)
3. If we complete a valid number, check if it's confusing
4. Backtrack and try other digit choices

To check if a number is confusing, we rotate it by replacing each digit with its rotated counterpart (6→9, 9→6, etc.) and compare with the original. The mapping array `d = [0, 1, -1, -1, -1, -1, 9, -1, 8, 6]` cleverly encodes both which digits are valid (non-negative values) and what they become when rotated.

The `limit` flag in the DFS is crucial - it tells us whether we're still constrained by `n`'s digits or can freely use any rotatable digit. Once we use a digit smaller than the corresponding digit in `n`, all subsequent positions become unconstrained.

This approach is much more efficient than brute force because we're only generating potentially valid numbers and pruning invalid branches early, rather than checking all numbers from 1 to n.

**Learn more about [Math](/problems/math-basics) and [Backtracking](/problems/backtracking) patterns.**

## Solution Approach

The solution uses digit dynamic programming with [backtracking](/problems/backtracking) to efficiently count confusing numbers. Let's break down the implementation:

**1. Digit Mapping Setup**
```python
d = [0, 1, -1, -1, -1, -1, 9, -1, 8, 6]
```
This array serves dual purpose:
- Index represents the original digit (0-9)
- Value represents what it becomes when rotated 180°
- `-1` indicates the digit cannot be rotated (invalid)

**2. Check Function**
```python
def check(x: int) -> bool:
    y, t = 0, x
    while t:
        t, v = divmod(t, 10)
        y = y * 10 + d[v]
    return x != y
```
This function rotates a number and checks if it's different from the original:
- Extract each digit from right to left using `divmod(t, 10)`
- Build the rotated number `y` by appending rotated digits
- Return `True` if the rotated number differs from the original

**3. DFS with Digit DP**
```python
def dfs(pos: int, limit: bool, x: int) -> int:
```
The recursive function builds valid numbers digit by digit:
- `pos`: Current position in the number we're building
- `limit`: Whether we're still bounded by the digits of `n`
- `x`: The number built so far

**4. Base Case**
```python
if pos >= len(s):
    return int(check(x))
```
When we've built a complete number (reached all positions), check if it's confusing.

**5. Digit Selection**
```python
up = int(s[pos]) if limit else 9
ans = 0
for i in range(up + 1):
    if d[i] != -1:
        ans += dfs(pos + 1, limit and i == up, x * 10 + i)
```
- If `limit` is `True`, we can only use digits up to `s[pos]` (the digit at position `pos` in `n`)
- If `limit` is `False`, we can use any digit from 0-9
- Only consider rotatable digits (`d[i] != -1`)
- Update `limit` for the next position: remains `True` only if we used the maximum allowed digit

**6. Main Function**
```python
s = str(n)
return dfs(0, True, 0)
```
Convert `n` to string for easy digit access and start the DFS from position 0 with limit enabled.

The algorithm efficiently explores only valid number combinations while maintaining the constraint that numbers shouldn't exceed `n`. By building numbers using only rotatable digits and checking the confusing property at the end, we avoid unnecessary iterations through invalid numbers.

## Example Walkthrough

Let's trace through the algorithm with `n = 20` to see how it counts confusing numbers.

**Setup:**
- `s = "20"` (string representation of n)
- `d = [0, 1, -1, -1, -1, -1, 9, -1, 8, 6]` (rotation mapping)

**DFS Exploration:**

Starting with `dfs(pos=0, limit=True, x=0)`:
- At position 0, we can use digits 0-2 (since `s[0]='2'` and `limit=True`)
- Valid rotatable digits in range [0,2]: only 0 and 1 (2 maps to -1, so invalid)

**Branch 1: Choose digit 0**
- Call `dfs(pos=1, limit=False, x=0)`
- Since we chose 0 < 2, `limit` becomes `False`
- At position 1, we can now use any digit 0-9
- Valid rotatable digits: 0, 1, 6, 8, 9
- This generates numbers: 00→0, 01→1, 06→6, 08→8, 09→9
- Checking each:
  - 0: rotates to 0 (same) → NOT confusing
  - 1: rotates to 1 (same) → NOT confusing
  - 6: rotates to 9 (different) → CONFUSING ✓
  - 8: rotates to 8 (same) → NOT confusing
  - 9: rotates to 6 (different) → CONFUSING ✓

**Branch 2: Choose digit 1**
- Call `dfs(pos=1, limit=False, x=1)`
- At position 1, we can use any digit 0-9
- Valid rotatable digits: 0, 1, 6, 8, 9
- This generates numbers: 10, 11, 16, 18, 19
- Checking each:
  - 10: rotates to 01→1 (different) → CONFUSING ✓
  - 11: rotates to 11 (same) → NOT confusing
  - 16: rotates to 91 (different) → CONFUSING ✓
  - 18: rotates to 81 (different) → CONFUSING ✓
  - 19: rotates to 61 (different) → CONFUSING ✓

**Rotation Examples:**
- Number 16: digit 1→1, digit 6→9, reversed = 91
- Number 19: digit 1→1, digit 9→6, reversed = 61

**Final Count:** 6 confusing numbers (6, 9, 10, 16, 18, 19)

The algorithm efficiently explores only 10 valid candidates (2 first digits × 5 second digits) instead of checking all 20 numbers, demonstrating how digit DP prunes the search space by only building numbers with rotatable digits.

## Python Solution

```python
class Solution:
    def confusingNumberII(self, n: int) -> int:
        def is_confusing_number(num: int) -> bool:
            """
            Check if a number is confusing by rotating it 180 degrees.
            A confusing number becomes different after rotation.
            """
            rotated = 0
            temp = num

            # Rotate each digit and build the rotated number
            while temp:
                temp, digit = divmod(temp, 10)
                rotated = rotated * 10 + rotation_map[digit]

            # A confusing number must be different from its rotation
            return num != rotated

        def count_confusing_numbers(position: int, is_limit: bool, current_num: int) -> int:
            """
            Use digit DP to count confusing numbers.

            Args:
                position: Current digit position being processed
                is_limit: Whether we're still bounded by the original number's digits
                current_num: The number being built so far

            Returns:
                Count of confusing numbers
            """
            # Base case: finished building a number
            if position >= len(digits_str):
                return int(is_confusing_number(current_num))

            # Determine the maximum digit we can use at this position
            max_digit = int(digits_str[position]) if is_limit else 9

            result = 0
            # Try each valid digit (0-9 that can be rotated)
            for digit in range(max_digit + 1):
                # Skip digits that cannot be rotated (2, 3, 4, 5, 7)
                if rotation_map[digit] != -1:
                    result += count_confusing_numbers(
                        position + 1,
                        is_limit and (digit == max_digit),
                        current_num * 10 + digit
                    )

            return result

        # Mapping of digits to their 180-degree rotation
        # -1 means the digit cannot be rotated validly
        rotation_map = [0, 1, -1, -1, -1, -1, 9, -1, 8, 6]
        #               0  1   2   3   4   5   6   7   8  9

        # Convert number to string for digit-by-digit processing
        digits_str = str(n)

        # Start DFS from the first digit
        return count_confusing_numbers(0, True, 0)
```

## Time and Space Complexity

**Time Complexity:** `O(log(n) * 5^log(n))` or simplified to `O(n^0.7)`

The time complexity is determined by the DFS traversal:
- The recursion depth is `O(log(n))` since we process each digit position of the number `n`
- At each position, we can choose from at most 5 valid digits (0, 1, 6, 8, 9) that have valid rotations
- In the worst case, we explore `5^log(n)` different number combinations
- Since `5^log(n) = n^(log(5)/log(10)) ≈ n^0.699`, the overall time complexity can be approximated as `O(n^0.7)`
- The `check()` function takes `O(log(n))` time to verify if a number is confusing, but this doesn't change the overall complexity

**Space Complexity:** `O(log(n))`

The space complexity comes from:
- The recursion call stack depth is `O(log(n))` corresponding to the number of digits in `n`
- The string `s` stores the digits of `n`, which takes `O(log(n))` space
- The dictionary `d` is a fixed-size array of 10 elements, contributing `O(1)` space
- Local variables in each recursive call use `O(1)` space

Therefore, the total space complexity is `O(log(n))`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrectly Handling Leading Zeros in Rotation**

**Pitfall:** A common mistake is not properly handling the rotation of numbers that produce leading zeros. For example, when rotating `6000`, you get `0009`, which should be treated as `9`, not `0009`. If you compare the string representations or don't handle the numeric conversion correctly, you might incorrectly classify such numbers.

**Example of Incorrect Implementation:**
```python
def check(x: int) -> bool:
    # WRONG: String comparison doesn't handle leading zeros
    original = str(x)
    rotated = ""
    for digit in reversed(original):
        rotated += str(d[int(digit)])
    return original != rotated  # This fails for cases like 6000
```

**Solution:** Always work with integer values when comparing the final numbers, as shown in the correct implementation. The integer conversion automatically handles leading zeros:
```python
def check(x: int) -> bool:
    y, t = 0, x
    while t:
        t, v = divmod(t, 10)
        y = y * 10 + d[v]  # Building as integer naturally handles leading zeros
    return x != y  # Integer comparison: 6000 != 9 ✓
```

### 2. **Starting DFS with Leading Zeros**

**Pitfall:** When building numbers digit by digit, you might accidentally count numbers with leading zeros (like `069` as a separate number from `69`), which would give incorrect counts since we're counting integers in the range `[1, n]`.

**Example of Incorrect Start:**
```python
# WRONG: This could potentially count numbers with leading zeros
for first_digit in range(10):  # Starting from 0
    if d[first_digit] != -1:
        count += dfs(1, limit and first_digit == int(s[0]), first_digit)
```

**Solution:** There are two approaches to handle this:

**Approach 1:** Skip leading zeros explicitly in the DFS:
```python
def dfs(pos: int, limit: bool, x: int, started: bool) -> int:
    if pos >= len(s):
        return int(started and check(x))  # Only count if we've started the number

    up = int(s[pos]) if limit else 9
    ans = 0
    for i in range(up + 1):
        if d[i] != -1:
            if i == 0 and not started:
                # Leading zero - don't mark as started
                ans += dfs(pos + 1, limit and i == up, x, False)
            else:
                # Valid digit that starts or continues the number
                ans += dfs(pos + 1, limit and i == up, x * 10 + i, True)
    return ans
```

**Approach 2:** Handle smaller numbers separately:
```python
# For each length from 1 to len(str(n))
total = 0
for length in range(1, len(str(n)) + 1):
    if length < len(str(n)):
        # Count all confusing numbers of this length
        total += count_fixed_length(length)
    else:
        # Count confusing numbers of max length up to n
        total += count_up_to_n(n)
```

### 3. **Forgetting to Check Digit Validity Before Rotation**

**Pitfall:** Attempting to rotate invalid digits (2, 3, 4, 5, 7) or not properly filtering them during the DFS traversal.

**Example of Incorrect Check:**
```python
def check(x: int) -> bool:
    y, t = 0, x
    while t:
        t, v = divmod(t, 10)
        # WRONG: Not checking if digit can be rotated
        y = y * 10 + d[v]  # This could use -1 for invalid digits!
    return x != y
```

**Solution:** The DFS should only build numbers using valid rotatable digits, so the check function should never encounter invalid digits. This is ensured by the condition `if d[i] != -1` in the DFS loop. However, for robustness, you could add validation:
```python
def check(x: int) -> bool:
    y, t = 0, x
    while t:
        t, v = divmod(t, 10)
        if d[v] == -1:  # Safety check
            return False
        y = y * 10 + d[v]
    return x != y
```

