# 3272. Find the Count of Good Integers

> Source: https://algo.monster/liteproblems/3272

## Problem Statement

You are given two positive integers `n` and `k`.

An integer `x` is called **k-palindromic** if it satisfies two conditions:
1. `x` is a palindrome (reads the same forwards and backwards)
2. `x` is divisible by `k`

An integer is called **good** if its digits can be rearranged to form a k-palindromic integer. For example, when `k = 2`, the number `2020` is good because it can be rearranged to form `2002`, which is both a palindrome and divisible by 2. However, `1010` is not good because no rearrangement of its digits can form a k-palindromic integer.

Your task is to count how many good integers have exactly `n` digits.

**Important constraints:**
- No integer can have leading zeros, neither in its original form nor after rearrangement
- For example, `1010` cannot be rearranged to form `0101` or `101` (different number of digits)

The problem asks you to return the total count of n-digit good integers.

## Intuition

The key insight is that instead of checking all n-digit numbers to see if they can be rearranged into k-palindromic numbers, we can work backwards: generate all possible n-digit palindromes and check if they're divisible by `k`.

Why does this work? Because if a number can be rearranged to form a palindrome, then that palindrome must have the same digit frequency as the original number. So we can:
1. Generate all n-digit palindromes
2. Check if each palindrome is divisible by `k` 
3. For valid palindromes, count how many different n-digit numbers can be rearranged to form them

To generate palindromes efficiently, we only need to enumerate the first half of the digits. For a palindrome of length `n`, we take the first `⌈n/2⌉` digits and mirror them. For example, if `n = 5` and we have `123` as the first half, we create `12321` by appending the reverse of `12` (excluding the middle digit since n is odd).

The range for the first half is `[10^((n-1)//2), 10^((n-1)//2 + 1))`. This ensures we generate all possible n-digit palindromes.

Once we find a valid k-palindromic number, we need to count how many different n-digit numbers share the same digits. This is a permutation problem with repetition. However, we must avoid counting the same set of digits multiple times. For instance, `2002`, `2020`, and `0220` all have the same digits `{0,0,2,2}`, so we should only count this digit set once.

To avoid duplicates, we use the sorted string representation of the palindrome as a unique identifier. If we've already processed a palindrome with the same sorted digits, we skip it.

For counting permutations, we use the formula: `(n - count_of_zeros) * (n-1)! / (product of factorials of digit frequencies)`. The `(n - count_of_zeros)` term accounts for the fact that we can't place a zero in the first position (no leading zeros allowed).

**Learn more about [Math](/problems/math-basics) and [Combinatorics](/problems/backtracking) patterns.**

## Solution Approach

The implementation follows these steps:

**1. Precompute Factorials**
```python
fac = [factorial(i) for i in range(n + 1)]
```
We precompute factorials from `0!` to `n!` for efficient permutation counting later.

**2. Initialize Variables**
- `ans`: Counter for the final result
- `vis`: Set to track processed digit combinations (avoids duplicates)
- `base = 10^((n-1)//2)`: Starting point for enumerating the first half of palindromes

**3. Generate Palindromes**
```python
for i in range(base, base * 10):
    s = str(i)
    s += s[::-1][n % 2:]
```
For each number `i` in the range `[base, base * 10)`:
- Convert it to string `s`
- Append its reverse to form a palindrome
- If `n` is odd (`n % 2 = 1`), skip the first character of the reverse to avoid duplicating the middle digit
- If `n` is even (`n % 2 = 0`), append the full reverse

Example: If `n = 5` and `i = 123`, we get `s = "123" + "21" = "12321"`

**4. Check Divisibility by k**
```python
if int(s) % k:
    continue
```
Skip palindromes that aren't divisible by `k`.

**5. Handle Duplicate Digit Sets**
```python
t = "".join(sorted(s))
if t in vis:
    continue
vis.add(t)
```
Sort the digits of the palindrome to create a unique identifier. If we've already processed this digit combination, skip it. Otherwise, add it to `vis`.

**6. Count Valid Permutations**
```python
cnt = Counter(t)
res = (n - cnt["0"]) * fac[n - 1]
for x in cnt.values():
    res //= fac[x]
ans += res
```

Using [combinatorics](/problems/backtracking) formula:
$$\frac{(n - x_0) \cdot (n - 1)!}{x_0! \cdot x_1! \cdots x_9!}$$

- `Counter(t)` counts the frequency of each digit
- `(n - cnt["0"])` represents valid choices for the first position (non-zero digits)
- `fac[n - 1]` represents `(n-1)!` permutations for remaining positions
- Divide by `fac[x]` for each digit frequency to eliminate duplicate permutations
- Add the result to the total answer

The algorithm efficiently counts all n-digit good integers by generating palindromes, checking divisibility, and using [combinatorics](/problems/backtracking) to count valid permutations while avoiding duplicates.

## Example Walkthrough

Let's walk through the solution with `n = 4` and `k = 2`.

**Step 1: Setup**
- Precompute factorials: `fac = [1, 1, 2, 6, 24]` (for 0! through 4!)
- Initialize `ans = 0`, `vis = set()`
- Calculate `base = 10^((4-1)//2) = 10^1 = 10`

**Step 2: Generate 4-digit palindromes**
We iterate `i` from 10 to 99 (first half of 4-digit palindromes):

For `i = 10`:
- `s = "10" + "01" = "1001"` (palindrome)
- Check: `1001 % 2 = 1` (odd, not divisible by 2)
- Skip this palindrome

For `i = 11`:
- `s = "11" + "11" = "1111"` (palindrome)
- Check: `1111 % 2 = 1` (odd, not divisible by 2)
- Skip this palindrome

For `i = 12`:
- `s = "12" + "21" = "1221"` (palindrome)
- Check: `1221 % 2 = 1` (odd, not divisible by 2)
- Skip this palindrome

...continuing...

For `i = 20`:
- `s = "20" + "02" = "2002"` (palindrome)
- Check: `2002 % 2 = 0` ✓ (divisible by 2)
- Sort digits: `t = "0022"`
- Not in `vis`, so add "0022" to `vis`
- Count permutations:
  - `cnt = Counter("0022") = {'0': 2, '2': 2}`
  - Valid first positions: `4 - 2 = 2` (can't use zeros)
  - Permutations: `res = 2 * 3! / (2! * 2!) = 2 * 6 / 4 = 3`
  - The 3 valid numbers are: 2002, 2020, 2200
- Add to answer: `ans = 0 + 3 = 3`

For `i = 22`:
- `s = "22" + "22" = "2222"` (palindrome)
- Check: `2222 % 2 = 0` ✓ (divisible by 2)
- Sort digits: `t = "2222"`
- Not in `vis`, so add "2222" to `vis`
- Count permutations:
  - `cnt = Counter("2222") = {'2': 4}`
  - Valid first positions: `4 - 0 = 4` (no zeros)
  - Permutations: `res = 4 * 3! / 4! = 4 * 6 / 24 = 1`
  - The only valid number is: 2222
- Add to answer: `ans = 3 + 1 = 4`

...continuing through all values...

For `i = 40`:
- `s = "40" + "04" = "4004"` (palindrome)
- Check: `4004 % 2 = 0` ✓ (divisible by 2)
- Sort digits: `t = "0044"`
- Not in `vis`, so add "0044" to `vis`
- Count permutations:
  - `cnt = Counter("0044") = {'0': 2, '4': 2}`
  - Valid first positions: `4 - 2 = 2`
  - Permutations: `res = 2 * 3! / (2! * 2!) = 2 * 6 / 4 = 3`
  - The 3 valid numbers are: 4004, 4040, 4400
- Add to answer: `ans = 4 + 3 = 7`

The process continues for all even palindromes from 10 to 99, accumulating the count of good integers. Each valid k-palindromic number contributes the count of its unique permutations (excluding those with leading zeros) to the final answer.

## Python Solution

```python
from math import factorial
from collections import Counter

class Solution:
    def countGoodIntegers(self, n: int, k: int) -> int:
        # Precompute factorials from 0 to n for permutation calculations
        factorials = [factorial(i) for i in range(n + 1)]
      
        # Total count of good integers
        total_count = 0
      
        # Set to track already processed digit combinations
        visited_combinations = set()
      
        # Calculate the starting point for generating n-digit palindromes
        # For n digits, we need to generate the first half
        half_start = 10 ** ((n - 1) // 2)
        half_end = half_start * 10
      
        # Generate all possible n-digit palindromes
        for half_number in range(half_start, half_end):
            # Convert half to string
            half_str = str(half_number)
          
            # Build palindrome based on whether n is odd or even
            # If n is odd, skip the middle character when reversing
            # If n is even, append the full reverse
            palindrome_str = half_str + half_str[::-1][n % 2:]
          
            # Check if palindrome is divisible by k
            palindrome_num = int(palindrome_str)
            if palindrome_num % k != 0:
                continue
          
            # Create a canonical form (sorted digits) to identify unique digit combinations
            sorted_digits = "".join(sorted(palindrome_str))
          
            # Skip if we've already processed this combination of digits
            if sorted_digits in visited_combinations:
                continue
          
            visited_combinations.add(sorted_digits)
          
            # Count frequency of each digit
            digit_counts = Counter(sorted_digits)
          
            # Calculate number of valid permutations (numbers that don't start with 0)
            # First position can be any non-zero digit: (n - count_of_zeros) choices
            # Remaining positions: (n-1)! ways to arrange
            valid_permutations = (n - digit_counts["0"]) * factorials[n - 1]
          
            # Divide by factorial of each digit count to account for duplicate digits
            for count in digit_counts.values():
                valid_permutations //= factorials[count]
          
            total_count += valid_permutations
      
        return total_count
```

## Time and Space Complexity

**Time Complexity:** `O(10^m × n × log n)` where `m = ⌊(n-1)/2⌋`

- The main loop iterates from `base` to `base * 10`, where `base = 10^((n-1)//2)`. This gives us `10^m` iterations where `m = ⌊(n-1)/2⌋`.
- For each iteration:
  - String concatenation and reversal operations: `O(n)` (since the resulting string has length `n`)
  - Modulo operation on an `n`-digit number: `O(n)`
  - Sorting the string `s` to create `t`: `O(n log n)`
  - Checking membership in `vis` set: `O(n)` (comparing strings of length `n`)
  - Creating Counter from string `t`: `O(n)`
  - Computing factorial division for permutations: `O(n)` (at most `n` distinct digits)
- The dominant operation per iteration is sorting: `O(n log n)`
- Total: `O(10^m × n × log n)`

**Space Complexity:** `O(10^m × n)` where `m = ⌊(n-1)/2⌋`

- `fac` array stores `n+1` factorial values: `O(n)`
- `vis` set can store at most `10^m` unique sorted strings, each of length `n`: `O(10^m × n)`
- Other variables like `s`, `t`, and `cnt` use `O(n)` space but are reused in each iteration
- The dominant space usage is the `vis` set: `O(10^m × n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Palindrome Generation for Odd-Length Numbers**

**Pitfall:** When generating palindromes of odd length, a common mistake is duplicating the middle digit incorrectly. For example, if `n = 5` and we start with `123`, we might incorrectly generate `12332321` (8 digits) instead of `12321` (5 digits).

**Why it happens:** The slicing logic `s[::-1][n % 2:]` can be confusing. When `n` is odd, we need to skip the first character of the reversed string to avoid duplicating the middle digit.

**Solution:** Ensure the palindrome generation logic correctly handles both odd and even lengths:
```python
# For odd n: append reverse without the first character
# For even n: append the full reverse
palindrome_str = half_str + half_str[::-1][n % 2:]
```

### 2. **Division by Zero in Permutation Calculation**

**Pitfall:** If the factorial array isn't properly initialized or if digit counts are incorrect, you might attempt to divide by `factorial[0]` when it's undefined or zero.

**Why it happens:** The code divides by `factorials[count]` for each digit count. If `factorials[0]` isn't properly set to 1, this will cause an error.

**Solution:** Always initialize `factorial(0) = 1`:
```python
factorials = [factorial(i) for i in range(n + 1)]  # factorial(0) = 1 by definition
```

### 3. **Integer Overflow in Permutation Counting**

**Pitfall:** For large values of `n`, the intermediate calculation `(n - digit_counts["0"]) * factorials[n - 1]` can overflow, even though the final result after division might be within bounds.

**Why it happens:** Python handles big integers well, but in other languages or if optimizing, you might face overflow issues.

**Solution:** Perform divisions as early as possible to keep numbers manageable:
```python
# Instead of computing the full numerator first
valid_permutations = factorials[n - 1]
for count in digit_counts.values():
    valid_permutations //= factorials[count]
valid_permutations *= (n - digit_counts["0"])
```

### 4. **Missing Edge Case: All Zeros in Digit Count**

**Pitfall:** When a palindrome consists entirely of zeros (which shouldn't happen for valid n-digit numbers), the formula `(n - digit_counts["0"])` would be 0, but this case should never occur since we're generating valid palindromes.

**Why it happens:** The palindrome generation starts from `10^((n-1)//2)`, which ensures at least one non-zero digit, but it's worth being aware of this constraint.

**Solution:** The current implementation naturally avoids this by starting palindrome generation from a non-zero base:
```python
half_start = 10 ** ((n - 1) // 2)  # Always starts with a non-zero digit
```

### 5. **Incorrect Handling of Single-Digit Numbers (n=1)**

**Pitfall:** When `n = 1`, the calculation `10 ** ((n - 1) // 2)` equals `10 ** 0 = 1`, which means the range would be `[1, 10)`. This works correctly, but the permutation counting logic might seem unnecessary for single digits.

**Solution:** The current implementation handles this correctly, but you could add a special case for clarity:
```python
if n == 1:
    # Count single-digit numbers divisible by k
    return len([i for i in range(1, 10) if i % k == 0])
```

