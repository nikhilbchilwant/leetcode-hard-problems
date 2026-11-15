# 2081. Sum of k-Mirror Numbers

> Source: https://algo.monster/liteproblems/2081

## Problem Statement

A **k-mirror number** is a positive integer without leading zeros that is a palindrome (reads the same forwards and backwards) in both base-10 and base-k representations.

For example:
- `9` is a 2-mirror number because:
  - In base-10: `9` reads the same forwards and backwards
  - In base-2: `9` = `1001`, which also reads the same forwards and backwards

- `4` is NOT a 2-mirror number because:
  - In base-10: `4` reads the same forwards and backwards
  - In base-2: `4` = `100`, which does NOT read the same forwards and backwards

Given two parameters:
- `k`: the base to check alongside base-10
- `n`: the count of k-mirror numbers needed

The task is to find the `n` smallest k-mirror numbers and return their sum.

## Intuition

The key insight is that checking all positive integers sequentially would be inefficient. Instead, we can be smarter by generating only base-10 palindromes first, then checking if they're also palindromes in base-k.

Why does this work? Because k-mirror numbers must be palindromes in base-10, we're guaranteed that every k-mirror number will be found in our generated base-10 palindromes. This dramatically reduces our search space.

To generate base-10 palindromes efficiently, we use a "half construction" technique. Any palindrome can be built by taking its first half and mirroring it. For example:
- For even-length palindrome `1221`: take first half `12`, mirror it to get `21`, combine to get `1221`
- For odd-length palindrome `12321`: take first half `123`, mirror all but the middle digit to get `21`, combine to get `12321`

By systematically generating palindromes this way (starting from length 1 and increasing), we ensure we get them in ascending order. This is crucial because we need the `n` smallest k-mirror numbers.

The approach becomes:
1. Generate palindromes in ascending order by length
2. For each length, generate all possible first halves
3. Construct the full palindrome from each first half
4. Check if it's also a palindrome in base-k
5. Keep track until we find `n` such numbers

This method is much more efficient than brute force because we're only checking numbers that are already palindromes in base-10, eliminating the vast majority of numbers that could never be k-mirror numbers.

**Learn more about [Math](/problems/math-basics) patterns.**

## Solution Approach

The implementation follows a systematic approach to generate and validate k-mirror numbers:

**1. Main Loop Structure**

We use an infinite counter `count(1)` to iterate through palindrome lengths starting from 1. For each length `l`, we determine the range of values for the first half of potential palindromes:
- Starting value: `x = 10^((l-1)//2)` 
- Ending value: `y = 10^((l+1)//2)`

For example:
- Length 1: range is `[1, 10)` 
- Length 2: range is `[1, 10)`
- Length 3: range is `[10, 100)`
- Length 4: range is `[10, 100)`

**2. Palindrome Construction**

For each number `i` in the first half range, we construct the complete palindrome `v`:
- Initialize `v = i` (this becomes the first half)
- Determine what to mirror: 
  - If length is even (`l % 2 == 0`): mirror the entire `i`
  - If length is odd: mirror `i // 10` (exclude the middle digit from mirroring)
- Build the second half by reversing digits:
  ```python
  while j > 0:
      v = v * 10 + j % 10  # Append last digit of j to v
      j //= 10              # Remove last digit from j
  ```

**3. Base-k Palindrome Check**

The `check` function verifies if a number is a palindrome in base-k:
```python
def check(x: int, k: int) -> bool:
    s = []
    while x:
        s.append(x % k)  # Extract digits in base-k
        x //= k
    return s == s[::-1]  # Check if palindrome
```

This converts the number to base-k representation and stores digits in a list, then checks if the list reads the same forwards and backwards.

**4. Result Accumulation**

When a k-mirror number is found:
- Add it to the running sum `ans`
- Decrement the counter `n`
- When `n` reaches 0, return the accumulated sum

**Example Walkthrough**

For `k=2, n=5`, the algorithm would:
1. Start with length 1: generate palindromes 1, 2, ..., 9
2. Check each in base-2: 
   - `1` → `1` in base-2 (palindrome ✓)
   - `3` → `11` in base-2 (palindrome ✓)
   - `5` → `101` in base-2 (palindrome ✓)
   - `7` → `111` in base-2 (palindrome ✓)
   - `9` → `1001` in base-2 (palindrome ✓)
3. Found 5 k-mirror numbers, return their sum: `1 + 3 + 5 + 7 + 9 = 25`

This approach efficiently finds k-mirror numbers by generating only base-10 palindromes and filtering those that are also palindromes in base-k, avoiding unnecessary checks on non-palindromic numbers.

## Example Walkthrough

Let's find the first 3 k-mirror numbers where k=3, and calculate their sum.

**Step 1: Generate palindromes of length 1**

First half range for length 1: `[1, 10)`
- Generate palindrome from `i=1`: Result is `1`
  - Check if `1` is palindrome in base-3: `1` → `1` in base-3 ✓
  - Found 1st k-mirror number: `1`

- Generate palindrome from `i=2`: Result is `2`
  - Check if `2` is palindrome in base-3: `2` → `2` in base-3 ✓
  - Found 2nd k-mirror number: `2`

- Generate palindrome from `i=3`: Result is `3`
  - Check if `3` is palindrome in base-3: `3` → `10` in base-3 ✗ (not palindrome)

- Generate palindrome from `i=4`: Result is `4`
  - Check if `4` is palindrome in base-3: `4` → `11` in base-3 ✓
  - Found 3rd k-mirror number: `4`

**Result**: We found our 3 k-mirror numbers: `1, 2, 4`

**Detailed construction for one palindrome (length 3 example):**

If we had continued to length 3 looking for more numbers:
- For `i=10` (first value in range `[10, 100)`):
  - Start with `v = 10`
  - Since length is odd, mirror `i // 10 = 1`
  - Build palindrome: `v = 10 * 10 + 1 = 101`
  - Check `101` in base-3: `101` → `10202` (palindrome ✓)

**Base-3 conversion demonstration for `4`:**
```
4 ÷ 3 = 1 remainder 1  → rightmost digit is 1
1 ÷ 3 = 0 remainder 1  → next digit is 1
Result: 4 in base-3 = 11 (reads same forwards/backwards)
```

**Final answer**: Sum = `1 + 2 + 4 = 7`

## Python Solution

```python
from itertools import count

class Solution:
    def kMirror(self, k: int, n: int) -> int:
        """
        Find the sum of the first n k-mirror numbers.
        A k-mirror number is a positive integer that is a palindrome in both base 10 and base k.
      
        Args:
            k: The base to check for palindrome property (2 <= k <= 9)
            n: The number of k-mirror numbers to find
          
        Returns:
            The sum of the first n k-mirror numbers
        """
      
        def is_palindrome_in_base_k(number: int, base: int) -> bool:
            """
            Check if a number is a palindrome when represented in base k.
          
            Args:
                number: The number to check
                base: The base for representation
              
            Returns:
                True if the number is a palindrome in the given base, False otherwise
            """
            digits = []
            temp = number
          
            # Convert number to base k representation
            while temp > 0:
                digits.append(temp % base)
                temp //= base
          
            # Check if digits form a palindrome
            return digits == digits[::-1]
      
        total_sum = 0
      
        # Iterate through palindrome lengths starting from 1
        for palindrome_length in count(1):
            # Calculate the range for the left half of the palindrome
            # For odd length palindromes, we need fewer digits in the left half
            left_half_start = 10 ** ((palindrome_length - 1) // 2)
            left_half_end = 10 ** ((palindrome_length + 1) // 2)
          
            # Generate all palindromes of current length
            for left_half in range(left_half_start, left_half_end):
                palindrome = left_half
              
                # Determine what part to mirror based on odd/even length
                # For even length: mirror all digits
                # For odd length: mirror all but the middle digit
                digits_to_mirror = left_half if palindrome_length % 2 == 0 else left_half // 10
              
                # Build the complete palindrome by appending reversed digits
                while digits_to_mirror > 0:
                    palindrome = palindrome * 10 + digits_to_mirror % 10
                    digits_to_mirror //= 10
              
                # Check if this palindrome is also a palindrome in base k
                if is_palindrome_in_base_k(palindrome, k):
                    total_sum += palindrome
                    n -= 1
                  
                    # Return when we've found n k-mirror numbers
                    if n == 0:
                        return total_sum
```

## Time and Space Complexity

The time complexity of this algorithm depends on finding `n` k-mirror numbers (numbers that are palindromes in both base 10 and base `k`).

For each length `l` of base-10 palindromes:
- We iterate through `10^((l-1)/2)` to `10^((l+1)/2)` possible first halves, which is approximately `10^(l/2)` iterations
- For each candidate, we construct a palindrome in `O(l)` time
- We check if it's a palindrome in base `k` using the `check` function, which takes `O(log_k(v))` time where `v` is the palindrome value

Since we need to find exactly `n` k-mirror numbers and `n ≤ 30`, the actual number of iterations is bounded. In practice, we'll examine palindromes of increasing lengths until we find `n` valid k-mirror numbers. The worst-case involves checking many palindromes before finding the required `n` k-mirror numbers.

Time Complexity: `O(n × 10^(l/2) × l)` where `l` is the maximum length of palindromes needed to find `n` k-mirror numbers. Since `n` is at most 30, this is effectively bounded in practice.

Space Complexity: `O(log_k(v))` for the list `s` used in the `check` function to store the base-`k` representation. Since we only use a constant amount of additional variables and the list `s` is recreated for each check, the space complexity is `O(log_k(v))` where `v` is the current palindrome value being checked.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Palindrome Generation for Odd vs Even Lengths**

**Pitfall:** A common mistake is incorrectly handling the middle digit when generating odd-length palindromes. Developers often mirror the entire left half for odd-length palindromes, which creates incorrect results.

**Example of the mistake:**
```python
# WRONG: Always mirroring the entire left_half
digits_to_mirror = left_half  # This is incorrect for odd lengths!
```

For a 3-digit palindrome starting with `12`, this would incorrectly generate `12221` (5 digits) instead of `121` (3 digits).

**Solution:** 
```python
# CORRECT: Exclude middle digit for odd-length palindromes
digits_to_mirror = left_half if palindrome_length % 2 == 0 else left_half // 10
```

### 2. **Off-by-One Errors in Range Calculation**

**Pitfall:** Incorrectly calculating the range for the left half can miss palindromes or generate invalid ones. A common error is using `10 ** (palindrome_length // 2)` for both start and end calculations.

**Example of the mistake:**
```python
# WRONG: This misses single-digit palindromes and has incorrect ranges
left_half_start = 10 ** (palindrome_length // 2)  # Would start at 10 for length 1!
```

**Solution:**
```python
# CORRECT: Proper range calculation
left_half_start = 10 ** ((palindrome_length - 1) // 2)
left_half_end = 10 ** ((palindrome_length + 1) // 2)
```

### 3. **Base Conversion Edge Cases**

**Pitfall:** The base-k palindrome check might fail for edge cases like when the number is 0 or when not properly handling the digit extraction.

**Example of the mistake:**
```python
# WRONG: Doesn't handle number = 0 correctly
def is_palindrome_in_base_k(number: int, base: int) -> bool:
    digits = []
    while number > 0:  # This loop never executes for number = 0
        digits.append(number % base)
        number //= base
    return digits == digits[::-1]  # Returns True for empty list!
```

**Solution:**
```python
# CORRECT: Handle edge cases explicitly
def is_palindrome_in_base_k(number: int, base: int) -> bool:
    if number == 0:
        return True  # 0 is a palindrome in any base
    digits = []
    temp = number
    while temp > 0:
        digits.append(temp % base)
        temp //= base
    return digits == digits[::-1]
```

### 4. **Performance Degradation with Large n**

**Pitfall:** Not breaking out of the inner loop early when enough k-mirror numbers are found, leading to unnecessary palindrome generation.

**Example of the mistake:**
```python
# WRONG: Continuing to generate palindromes even after finding n numbers
for left_half in range(left_half_start, left_half_end):
    # ... generate palindrome ...
    if is_palindrome_in_base_k(palindrome, k):
        total_sum += palindrome
        n -= 1
# Missing early termination!
```

**Solution:**
```python
# CORRECT: Return immediately when n reaches 0
if is_palindrome_in_base_k(palindrome, k):
    total_sum += palindrome
    n -= 1
    if n == 0:
        return total_sum  # Early termination
```

### 5. **Integer Overflow in Other Languages**

**Pitfall:** While Python handles arbitrary precision integers, implementing this in languages like Java or C++ requires careful consideration of integer overflow when calculating palindromes or their sum.

**Solution for other languages:**
- Use appropriate data types (`long long` in C++, `long` in Java)
- Consider using BigInteger classes for very large values
- Add overflow checks when multiplying by 10 or adding digits

