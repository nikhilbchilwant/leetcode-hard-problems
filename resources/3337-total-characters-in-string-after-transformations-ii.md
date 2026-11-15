# 3337. Total Characters in String After Transformations II

> Source: https://algo.monster/liteproblems/3337

## Problem Statement

You are given a string `s` consisting of lowercase English letters, an integer `t` representing the number of transformations to perform, and an array `nums` of size 26.

In each transformation, every character in the string is replaced according to specific rules:

1. Each character `s[i]` is replaced with the next `nums[s[i] - 'a']` consecutive characters in the alphabet. For example:
   - If `s[i] = 'a'` and `nums[0] = 3`, the character `'a'` transforms into the next 3 consecutive characters ahead of it, resulting in `"bcd"`
 
2. The transformation wraps around the alphabet if it goes beyond `'z'`. For example:
   - If `s[i] = 'y'` and `nums[24] = 3`, the character `'y'` transforms into the next 3 consecutive characters ahead of it, which wraps around to give `"zab"`

The task is to determine the length of the resulting string after performing exactly `t` transformations on the original string `s`.

Since the answer can be very large, return it modulo `10^9 + 7`.

## Intuition

The first observation is that we don't need to track the actual string content - we only need to count how many of each character we have. Each character transforms independently based on its position in the alphabet.

Let's think about what happens in one transformation. If we have 5 'a's and `nums[0] = 3`, then each 'a' becomes "bcd". So after one transformation, we get 5 'b's, 5 'c's, and 5 'd's from those original 'a's. This suggests we can track character frequencies instead of the actual string.

We can represent this transformation as a relationship: the count of each character after transformation depends on the counts of all characters before transformation. For character `i`, it contributes to characters `(i+1) % 26`, `(i+2) % 26`, ..., `(i+nums[i]) % 26`.

This relationship can be expressed as matrix multiplication. We create a transformation matrix `M` where `M[i][j] = 1` if character `i` transforms to include character `j`, and `0` otherwise. Then, if we have a frequency vector `cnt` representing current character counts, `cnt * M` gives us the character counts after one transformation.

For `t` transformations, we need to compute `cnt * M * M * ... * M` (t times), which is `cnt * M^t`. Computing `M^t` directly would take O(t) matrix multiplications, which is too slow for large `t`.

The key insight is to use fast matrix exponentiation. Just like we can compute `x^n` in O(log n) time using binary exponentiation (`x^8 = ((x^2)^2)^2`), we can compute `M^t` in O(log t) matrix multiplications. This reduces the time complexity from O(t) to O(log t), making it feasible even for very large values of `t`.

Finally, the length of the resulting string is simply the sum of all character frequencies after `t` transformations.

**Learn more about [Math](/problems/math-basics) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution uses fast matrix exponentiation to accelerate the recurrence relationship. Let's break down the implementation:

**1. Initialize Character Frequency Count**

First, we count the frequency of each character in the input string `s`:
```python
cnt = [0] * 26
for c in s:
    cnt[ord(c) - ord("a")] += 1
```

**2. Build the Transformation Matrix**

We create a 26×26 transformation matrix where `matrix[i][j] = 1` if character `i` (0-indexed) transforms to include character `j`:
```python
matrix = [[0] * 26 for _ in range(26)]
for i, x in enumerate(nums):
    for j in range(1, x + 1):
        matrix[i][(i + j) % 26] = 1
```

For example, if `nums[0] = 3` (character 'a' expands to 3 characters), then `matrix[0][1] = matrix[0][2] = matrix[0][3] = 1`, meaning 'a' transforms to 'b', 'c', and 'd'.

**3. Matrix Multiplication Function**

The `matmul` function multiplies two matrices with modulo operations to prevent overflow:
```python
def matmul(a, b):
    n, p, q = len(a), len(b), len(b[0])
    res = [[0] * q for _ in range(n)]
    for i in range(n):
        for k in range(p):
            if a[i][k]:  # Optimization: skip if zero
                for j in range(q):
                    res[i][j] = (res[i][j] + a[i][k] * b[k][j]) % mod
    return res
```

**4. Fast Matrix Exponentiation**

The `matpow` function computes `matrix^t` using binary exponentiation:
```python
def matpow(mat, power):
    res = [[int(i == j) for j in range(26)] for i in range(26)]  # Identity matrix
    while power:
        if power % 2:
            res = matmul(res, mat)
        mat = matmul(mat, mat)
        power //= 2
    return res
```

This reduces the time complexity from O(t) to O(log t). The algorithm works by:
- Starting with an identity matrix as the result
- If the current bit of `power` is 1, multiply the result by the current matrix
- Square the matrix for the next bit position
- Repeat until all bits are processed

**5. Apply Transformations and Calculate Result**

Finally, we compute the character frequencies after `t` transformations:
```python
cnt = [cnt]  # Convert to 1×26 matrix
factor = matpow(matrix, t)  # Get matrix^t
result = matmul(cnt, factor)[0]  # Apply transformations
ans = sum(result) % mod  # Sum all frequencies
```

The multiplication `cnt * matrix^t` gives us the frequency of each character after `t` transformations. The sum of these frequencies is the total length of the resulting string.

**Time Complexity:** O(26³ × log t) for matrix exponentiation
**Space Complexity:** O(26²) for storing matrices

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Input:** `s = "ab"`, `t = 2`, `nums = [1, 2, 0, 0, ..., 0]` (remaining values are 0)

**Step 1: Count initial character frequencies**
- 'a' appears 1 time
- 'b' appears 1 time
- Frequency vector: `cnt = [1, 1, 0, 0, ..., 0]`

**Step 2: Build transformation matrix**
- 'a' (index 0) with `nums[0] = 1` transforms to the next 1 character → 'b'
- 'b' (index 1) with `nums[1] = 2` transforms to the next 2 characters → 'c', 'd'
- All other characters have `nums[i] = 0`, so they don't transform

Transformation matrix (showing only first 4×4 for clarity):
```
     a  b  c  d
a [[ 0  1  0  0 ]
b  [ 0  0  1  1 ]
c  [ 0  0  0  0 ]
d  [ 0  0  0  0 ]]
```

**Step 3: First transformation (t=1)**
Apply `cnt × matrix`:
- 'a' (count=1) → produces 1 'b'
- 'b' (count=1) → produces 1 'c' and 1 'd'
- Result after 1 transformation: `[0, 1, 1, 1, 0, ..., 0]`
- String would be "bcd" (length = 3)

**Step 4: Second transformation (t=2)**
We need `matrix²` for two transformations.

Computing `matrix²`:
```
     a  b  c  d
a [[ 0  0  1  1 ]
b  [ 0  0  0  0 ]
c  [ 0  0  0  0 ]
d  [ 0  0  0  0 ]]
```

Apply original `cnt × matrix²`:
- Original 'a' (count=1) → after 2 transformations produces 1 'c' and 1 'd'
- Original 'b' (count=1) → after 2 transformations produces nothing (since 'c' and 'd' have nums=0)
- Final frequency: `[0, 0, 1, 1, 0, ..., 0]`
- String would be "cd" (length = 2)

**Result:** The length after 2 transformations is 2.

This example demonstrates how:
1. We track character frequencies instead of the actual string
2. The transformation matrix encodes how each character expands
3. Matrix exponentiation efficiently computes multiple transformations
4. The final length is the sum of all character frequencies

## Python Solution

```python
class Solution:
    def lengthAfterTransformations(self, s: str, t: int, nums: List[int]) -> int:
        """
        Calculate the total length of string after t transformations.
        Each character can transform into multiple characters based on nums array.
      
        Args:
            s: Initial string
            t: Number of transformations to perform
            nums: Array where nums[i] indicates character 'a'+i transforms to next nums[i] characters
      
        Returns:
            Total length after transformations modulo 10^9 + 7
        """
        MOD = 10**9 + 7
        ALPHABET_SIZE = 26
      
        # Count frequency of each character in the initial string
        char_counts = [0] * ALPHABET_SIZE
        for char in s:
            char_index = ord(char) - ord('a')
            char_counts[char_index] += 1
      
        # Build transformation matrix
        # transformation_matrix[i][j] = 1 if character i can transform to character j
        transformation_matrix = [[0] * ALPHABET_SIZE for _ in range(ALPHABET_SIZE)]
        for char_idx, transform_count in enumerate(nums):
            # Character at index char_idx transforms into the next transform_count characters
            for offset in range(1, transform_count + 1):
                target_idx = (char_idx + offset) % ALPHABET_SIZE
                transformation_matrix[char_idx][target_idx] = 1
      
        def matrix_multiply(matrix_a: List[List[int]], matrix_b: List[List[int]]) -> List[List[int]]:
            """
            Multiply two matrices and return the result modulo MOD.
          
            Args:
                matrix_a: First matrix (n x p)
                matrix_b: Second matrix (p x q)
          
            Returns:
                Result matrix (n x q)
            """
            rows_a, cols_a, cols_b = len(matrix_a), len(matrix_b), len(matrix_b[0])
            result = [[0] * cols_b for _ in range(rows_a)]
          
            for i in range(rows_a):
                for k in range(cols_a):
                    if matrix_a[i][k]:  # Skip if element is 0 for optimization
                        for j in range(cols_b):
                            result[i][j] = (result[i][j] + matrix_a[i][k] * matrix_b[k][j]) % MOD
          
            return result
      
        def matrix_power(matrix: List[List[int]], exponent: int) -> List[List[int]]:
            """
            Calculate matrix raised to the power of exponent using binary exponentiation.
          
            Args:
                matrix: Square matrix to exponentiate
                exponent: Power to raise the matrix to
          
            Returns:
                Matrix raised to the given power
            """
            # Initialize result as identity matrix
            result = [[int(i == j) for j in range(ALPHABET_SIZE)] for i in range(ALPHABET_SIZE)]
          
            # Binary exponentiation
            while exponent > 0:
                if exponent % 2 == 1:
                    result = matrix_multiply(result, matrix)
                matrix = matrix_multiply(matrix, matrix)
                exponent //= 2
          
            return result
      
        # Convert char_counts to 1xN matrix format for multiplication
        char_counts_matrix = [char_counts]
      
        # Calculate transformation matrix raised to power t
        transformation_factor = matrix_power(transformation_matrix, t)
      
        # Apply transformations: multiply initial counts by transformation factor
        final_counts = matrix_multiply(char_counts_matrix, transformation_factor)[0]
      
        # Sum all character counts to get total length
        total_length = sum(final_counts) % MOD
      
        return total_length
```

## Time and Space Complexity

**Time Complexity:** `O(n + log t × |Σ|³)`

The time complexity breaks down as follows:
- `O(n)` for counting character frequencies in the initial string, where `n` is the length of string `s`
- Building the initial transformation matrix takes `O(|Σ|²)` time, where `|Σ| = 26` (the alphabet size)
- The matrix exponentiation `matpow(matrix, t)` dominates the complexity:
  - It performs `O(log t)` iterations due to binary exponentiation
  - Each iteration involves matrix multiplication via `matmul`
  - Each matrix multiplication of two `|Σ| × |Σ|` matrices takes `O(|Σ|³)` time
  - Total for matrix exponentiation: `O(log t × |Σ|³)`
- Final matrix multiplication between `cnt` (1 × 26) and `factor` (26 × 26) takes `O(|Σ|²)`
- Computing the final sum takes `O(|Σ|)`

Since `|Σ|³ > |Σ|²` and `log t × |Σ|³` typically dominates `n` for large `t`, the overall time complexity is `O(n + log t × |Σ|³)`.

**Space Complexity:** `O(|Σ|²)`

The space complexity analysis:
- `cnt` array uses `O(|Σ|)` space
- `matrix` uses `O(|Σ|²)` space for the transformation matrix
- `matmul` creates temporary matrices of size `O(|Σ|²)`
- `matpow` maintains matrices of size `O(|Σ|²)` throughout execution
- The identity matrix in `matpow` uses `O(|Σ|²)` space

The dominant space requirement is `O(|Σ|²)` for storing the matrices during computation.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Integer Overflow in Matrix Multiplication**

**Pitfall:** When performing matrix multiplication, the intermediate calculations can exceed integer limits before applying the modulo operation. Even though Python handles arbitrary precision integers, in other languages or when optimizing, developers might forget to apply modulo at each step.

**Problem Code:**
```python
# Incorrect - might overflow in some languages or cause performance issues
res[i][j] = res[i][j] + a[i][k] * b[k][j]
res[i][j] %= mod  # Modulo applied too late
```

**Solution:** Apply modulo operation immediately after each multiplication and addition:
```python
# Correct approach
res[i][j] = (res[i][j] + a[i][k] * b[k][j]) % mod
```

### 2. **Incorrect Matrix Indexing in Transformation Matrix Construction**

**Pitfall:** The transformation should map character `i` to characters `(i+1)` through `(i+nums[i])`, not starting from `i` itself. A common mistake is to include the original character or start from index 0.

**Problem Code:**
```python
# Incorrect - includes the original character
for j in range(nums[i] + 1):  # Should be range(1, nums[i] + 1)
    matrix[i][(i + j) % 26] = 1

# Or incorrect - starts from 0 instead of i+1
for j in range(nums[i]):
    matrix[i][j] = 1  # Wrong! Should be (i + j + 1) % 26
```

**Solution:** Start the range from 1 and add it to the current character index:
```python
# Correct approach
for j in range(1, nums[i] + 1):
    matrix[i][(i + j) % 26] = 1
```

### 3. **Identity Matrix Initialization Error**

**Pitfall:** When initializing the identity matrix for matrix exponentiation, using shallow copy or incorrect initialization can lead to wrong results.

**Problem Code:**
```python
# Incorrect - creates references to the same list
res = [[0] * 26] * 26
for i in range(26):
    res[i][i] = 1  # This modifies multiple rows!

# Or forgetting to set diagonal elements
res = [[0] * 26 for _ in range(26)]  # Missing diagonal = 1
```

**Solution:** Use list comprehension with proper conditional logic:
```python
# Correct approach
res = [[int(i == j) for j in range(26)] for i in range(26)]
```

### 4. **Off-by-One Error in Character Frequency Counting**

**Pitfall:** Using incorrect ASCII value calculations when converting characters to indices.

**Problem Code:**
```python
# Incorrect - might use wrong base
cnt[ord(c) - ord('A')] += 1  # Wrong if lowercase letters
# Or
cnt[ord(c) - 97] += 1  # Magic number, less readable
```

**Solution:** Always use `ord('a')` for lowercase letters:
```python
# Correct approach
cnt[ord(c) - ord('a')] += 1
```

### 5. **Forgetting Edge Cases with t = 0**

**Pitfall:** The algorithm should handle `t = 0` correctly, returning the original string length without any transformations.

**Problem Code:**
```python
# Might fail if matrix_power doesn't handle exponent = 0
factor = matrix_power(matrix, t)  # What if t = 0?
```

**Solution:** The matrix_power function correctly handles this by returning the identity matrix when exponent = 0, which leaves the character counts unchanged. Ensure the while loop condition properly handles zero:
```python
# Correct - returns identity matrix for exponent = 0
while exponent > 0:  # Loop doesn't execute when exponent = 0
    # ... exponentiation logic
return result  # Returns identity matrix
```

