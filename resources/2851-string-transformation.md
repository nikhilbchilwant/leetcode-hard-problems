# 2851. String Transformation

> Source: https://algo.monster/liteproblems/2851

## Problem Statement

You have two strings `s` and `t` of equal length `n`. You can perform a specific operation on string `s`:

**Operation**: Remove a suffix from `s` of length `l` (where `0 < l < n`) and append it to the beginning of `s`. For example, if `s = 'abcd'`, you can remove the suffix `'cd'` and place it at the front, resulting in `s = 'cdab'`.

Given an integer `k`, you need to find the number of ways to transform string `s` into string `t` using **exactly** `k` operations.

Since the answer can be very large, return the result modulo `10^9 + 7`.

**Key Points:**
- Each operation is essentially a rotation of the string
- You must use exactly `k` operations (not fewer, not more)
- The strings `s` and `t` have the same length `n`
- In each operation, you must remove at least 1 character but not all characters from the end

**Example to understand the operation:**
- Starting with `s = 'abcd'`
- Remove suffix of length 1: `'d'` → Result: `'dabc'`
- Remove suffix of length 2: `'cd'` → Result: `'cdab'`  
- Remove suffix of length 3: `'bcd'` → Result: `'bcda'`

The problem asks you to count all possible sequences of `k` such operations that can transform `s` into `t`.

## Intuition

The key insight is recognizing that each operation is simply a **rotation** of the string. When we remove a suffix and prepend it, we're rotating the string. For a string of length `n`, there are exactly `n` possible rotations (including the original position).

We can represent each rotation by where the original first character ends up. If the first character of `s` moves to position `i`, we can use `i` as the unique identifier for that rotation state.

**First observation**: We need to identify which rotations of `s` match string `t`. There could be multiple rotations that match (or none at all). To find these efficiently, we can use the Z-algorithm on the concatenated string `s + t + t`. When we find a substring starting at position `i` (where `n ≤ i < 2n`) that matches the prefix of length `n`, then rotation `i - n` transforms `s` into `t`.

**Second observation**: This is a path counting problem. We need to count paths of exactly `k` steps from the initial state (rotation 0) to any target state (rotations that match `t`).

**Third observation**: The transition structure is special. From rotation 0, we can reach any of the `n-1` non-zero rotations in one step. From any non-zero rotation, we can reach rotation 0 or any of the other `n-2` non-zero rotations. This symmetry means all non-zero rotations behave identically!

This leads us to simplify the problem to just two states:
- State 0: rotation is 0
- State 1: rotation is non-zero

The transitions become:
- From state 0: can go to state 1 with `n-1` ways
- From state 1: can go to state 0 with 1 way, or stay in state 1 with `n-2` ways

This is now a simple 2-state [dynamic programming](/problems/dynamic_programming_intro) problem that can be solved with matrix exponentiation for efficiency. We compute `dp[k][0]` and `dp[k][1]` using the transition matrix `[[0, 1], [n-1, n-2]]` raised to the `k`-th power.

Finally, we sum up the contributions: for each rotation that matches `t`, we add `dp[k][0]` if it's rotation 0, or `dp[k][1]` if it's a non-zero rotation.

**Learn more about [Math](/problems/math-basics) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution implements several key algorithms and techniques to efficiently solve this problem:

### 1. **Z-Algorithm for Pattern Matching**

The `getZ` method implements the Z-algorithm to find all rotations of `s` that match `t`. We concatenate the strings as `s + t + t` (length `3n`) and compute the Z-array:

- For each position `i` in range `[n, 2n)`, if `z[i] >= n`, then rotation `(i - n)` of `s` equals `t`
- The Z-algorithm efficiently computes the longest prefix match at each position in `O(n)` time
- It maintains a window `[left, right]` representing the rightmost Z-box to avoid redundant comparisons

### 2. **[Dynamic Programming](/problems/dynamic_programming_intro) with State Reduction**

Instead of tracking all `n` possible rotations, we reduce to 2 states:
- `dp[t][0]` = number of ways to reach rotation 0 after exactly `t` steps
- `dp[t][1]` = number of ways to reach any specific non-zero rotation after exactly `t` steps

The recurrence relations are:
- `dp[t][0] = dp[t-1][1] * (n-1)` (from any non-zero rotation, we can reach 0)
- `dp[t][1] = dp[t-1][0] * 1 + dp[t-1][1] * (n-2)` (from 0 or from other non-zero rotations)

Initial conditions: `dp[0][0] = 1`, `dp[0][1] = 0`

### 3. **Matrix Exponentiation for Fast Computation**

To compute `dp[k]` efficiently for large `k`, we use matrix multiplication:

```
[dp[t][0]]   [0      1   ] [dp[t-1][0]]
[dp[t][1]] = [n-1  n-2] * [dp[t-1][1]]
```

The transition matrix `A = [[0, 1], [n-1, n-2]]` is raised to the `k`-th power using fast exponentiation:
- `matrixPower` implements binary exponentiation: repeatedly square the matrix and multiply when needed
- This reduces time complexity from `O(k)` to `O(log k)`

### 4. **Modular Arithmetic**

All operations use modular arithmetic with `MOD = 10^9 + 7`:
- `add(x, y)`: adds two numbers and applies modulo efficiently
- `mul(x, y)`: multiplies two numbers with modulo
- These helper functions prevent integer overflow and maintain correctness

### 5. **Final Computation**

The `numberOfWays` method orchestrates the solution:
1. Compute the transition matrix raised to power `k`
2. Extract `dp[k][0]` and `dp[k][1]` from the first row
3. Use Z-algorithm to find all valid target rotations
4. For each valid rotation `r`:
   - If `r = 0`, add `dp[k][0]` to result
   - If `r ≠ 0`, add `dp[k][1]` to result
5. Return the total count modulo `10^9 + 7`

**Time Complexity**: `O(n + log k)` - Z-algorithm takes `O(n)`, matrix exponentiation takes `O(log k)`  
**Space Complexity**: `O(n)` - for storing the concatenated string and Z-array

## Example Walkthrough

Let's walk through a concrete example with `s = "aba"`, `t = "baa"`, and `k = 2`.

**Step 1: Identify which rotations of s match t**

First, let's see all possible rotations of `s = "aba"`:
- Rotation 0: "aba" (original)
- Rotation 1: "aab" (moved 1 char from end to front)
- Rotation 2: "baa" (moved 2 chars from end to front)

We can see that rotation 2 matches our target `t = "baa"`.

Using the Z-algorithm on `s + t + t = "aba" + "baa" + "baa"`:
- At position 5 (which is index 2 in the second copy), we find a match of length 3
- This confirms rotation `5 - 3 = 2` transforms `s` into `t`

**Step 2: Set up the state transition model**

With `n = 3`, we have:
- From rotation 0: can reach rotations 1 or 2 (n-1 = 2 ways)
- From rotation 1: can reach rotations 0 or 2 (move 1 or 2 chars)
- From rotation 2: can reach rotations 0 or 1 (move 1 or 2 chars)

We simplify to two states:
- State 0: at rotation 0
- State 1: at any non-zero rotation (1 or 2)

**Step 3: Calculate dp values for k = 2**

Initial state: `dp[0][0] = 1`, `dp[0][1] = 0` (we start at rotation 0)

After 1 operation:
- `dp[1][0] = dp[0][1] × 1 = 0 × 1 = 0`
- `dp[1][1] = dp[0][0] × 1 + dp[0][1] × (3-2) = 1 × 1 + 0 × 1 = 1`

After 2 operations:
- `dp[2][0] = dp[1][1] × 1 = 1 × 1 = 1`
- `dp[2][1] = dp[1][0] × 1 + dp[1][1] × (3-2) = 0 × 1 + 1 × 1 = 1`

Using matrix form:
```
[0  1] × [0  1]   [2  1]
[2  1]   [2  1] = [2  3]
```
First row gives us: `dp[2][0] = 2`, `dp[2][1] = 1`

**Step 4: Count valid paths**

Our target rotation is 2 (non-zero), so we use `dp[2][1] = 1`.

**Result: 1 way**

Let's verify by listing all possible 2-operation sequences:
1. 0 → 1 → 0 (doesn't end at target)
2. 0 → 1 → 2 ✓ (ends at rotation 2 = "baa")
3. 0 → 2 → 0 (doesn't end at target)
4. 0 → 2 → 1 (doesn't end at target)

Indeed, there's exactly 1 way to reach `t = "baa"` in exactly 2 operations.

---


Consider `s = "aa"`, `t = "aa"`, `k = 1`.

**Step 1: Identify matching rotations**
- Rotation 0: "aa" ✓
- Rotation 1: "aa" ✓

Both rotations match `t`!

**Step 2: Calculate dp values for k = 1**
- `dp[1][0] = 0` (can't return to rotation 0 in 1 step)
- `dp[1][1] = 1` (can reach rotation 1 from rotation 0)

**Step 3: Count valid paths**
- Rotation 0 contributes: `dp[1][0] = 0`
- Rotation 1 contributes: `dp[1][1] = 1`

**Result: 0 + 1 = 1 way**

This shows how the algorithm handles cases where multiple rotations of `s` equal `t`.

## Python Solution

```python
class Solution:
    MOD: int = 1000000007

    def add(self, x: int, y: int) -> int:
        """Add two numbers with modulo operation."""
        x += y
        if x >= self.MOD:
            x -= self.MOD
        return x

    def mul(self, x: int, y: int) -> int:
        """Multiply two numbers with modulo operation."""
        return (x * y) % self.MOD

    def getZ(self, s: str) -> List[int]:
        """
        Compute Z-array using Z-algorithm.
        Z[i] represents the length of the longest substring starting from s[i] 
        which is also a prefix of s.
        """
        n = len(s)
        z_array = [0] * n
        left = right = 0
      
        for i in range(1, n):
            # If i is within the Z-box, we can use previously computed values
            if i <= right and z_array[i - left] <= right - i:
                z_array[i] = z_array[i - left]
            else:
                # Calculate Z[i] from scratch or extend beyond the Z-box
                z_value = max(0, right - i + 1)
                while i + z_value < n and s[i + z_value] == s[z_value]:
                    z_value += 1
                z_array[i] = z_value
          
            # Update the Z-box if we've found a longer match
            if i + z_array[i] - 1 > right:
                left = i
                right = i + z_array[i] - 1
              
        return z_array

    def matrixMultiply(self, matrix_a: List[List[int]], matrix_b: List[List[int]]) -> List[List[int]]:
        """
        Multiply two matrices with modulo operation.
        matrix_a is m x n, matrix_b is n x p, result is m x p.
        """
        rows_a = len(matrix_a)
        cols_a = len(matrix_a[0])
        cols_b = len(matrix_b[0])
      
        # Initialize result matrix with zeros
        result = [[0] * cols_b for _ in range(rows_a)]
      
        # Perform matrix multiplication
        for i in range(rows_a):
            for j in range(cols_b):
                for k in range(cols_a):
                    result[i][j] = self.add(result[i][j], 
                                           self.mul(matrix_a[i][k], matrix_b[k][j]))
        return result

    def matrixPower(self, matrix: List[List[int]], exponent: int) -> List[List[int]]:
        """
        Calculate matrix raised to the power of exponent using fast exponentiation.
        """
        size = len(matrix)
      
        # Initialize result as identity matrix
        result = [[0] * size for _ in range(size)]
        for i in range(size):
            result[i][i] = 1
      
        # Create a copy of the input matrix for manipulation
        base = [row[:] for row in matrix]
      
        # Fast exponentiation
        while exponent > 0:
            if exponent & 1:  # If exponent is odd
                result = self.matrixMultiply(result, base)
            base = self.matrixMultiply(base, base)
            exponent >>= 1  # Divide exponent by 2
          
        return result

    def numberOfWays(self, s: str, t: str, k: int) -> int:
        """
        Count the number of ways to transform string s to string t in exactly k operations.
        Each operation rotates the string by moving some prefix to the end.
        """
        n = len(s)
      
        # Calculate dp values using matrix exponentiation
        # dp[0] = ways to reach position 0 (original string)
        # dp[1] = ways to reach any non-zero position
        transition_matrix = [[0, 1], [n - 1, n - 2]]
        power_matrix = self.matrixPower(transition_matrix, k)
        dp_values = power_matrix[0]  # First row gives us [dp[k][0], dp[k][1]]
      
        # Concatenate strings for Z-algorithm: s + t + t
        concatenated = s + t + t
        z_array = self.getZ(concatenated)
      
        # Check all possible rotations of t that match s
        result = 0
        double_n = n + n
      
        for i in range(n, double_n):
            # If the substring starting at position i matches at least n characters,
            # it means rotating s by (i - n) positions gives us t
            if z_array[i] >= n:
                rotation_amount = i - n
                # Add the number of ways to achieve this rotation in k steps
                if rotation_amount == 0:
                    result = self.add(result, dp_values[0])
                else:
                    result = self.add(result, dp_values[1])
                  
        return result
```

## Time and Space Complexity

**Time Complexity: `O(n + log k)`**

The time complexity breaks down into several components:

1. **Z-algorithm (`getZ` function):** `O(n)` where n is the length of the concatenated string `s + t + t` (which is `3n`). The Z-algorithm processes each character at most twice (once when extending right, once when using previously computed values), resulting in linear time complexity.

2. **Matrix Power (`matrixPower` function):** `O(log k)` for computing the k-th power of a 2×2 matrix using fast exponentiation. Since the matrix is fixed size (2×2), each matrix multiplication takes `O(1)` time, and we perform `O(log k)` multiplications due to the binary exponentiation approach.

3. **Matrix Multiplication (`matrixMultiply` function):** For 2×2 matrices, this is `O(1)` per multiplication.

4. **Final loop to calculate result:** `O(n)` to iterate through positions from `n` to `2n` and check Z-values.

Overall: `O(3n) + O(log k) + O(n) = O(n + log k)`

**Space Complexity: `O(n)`**

The space complexity consists of:

1. **Concatenated string:** `O(3n) = O(n)` for storing `s + t + t`

2. **Z-array:** `O(3n) = O(n)` for storing the Z-values

3. **Matrix operations:** `O(1)` since we're working with fixed 2×2 matrices

4. **Other variables:** `O(1)` for counters, indices, and the result

Overall: `O(n) + O(n) + O(1) = O(n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Understanding of Valid Rotations**

**Pitfall**: A common mistake is assuming that we need to check if rotating `s` by some amount gives us `t`, when actually we need to check if rotating `t` gives us `s`. This confusion arises because the operation moves a suffix of the current string to the front.

**Why it happens**: The problem states we transform `s` into `t`, so intuitively we think about rotating `s`. However, after `k` operations starting from `s`, we need to end up at `t`. The Z-algorithm checks which rotations of `s` equal `t`.

**Solution**: The code correctly handles this by concatenating `s + t + t` and checking positions in range `[n, 2n)`. If `z[i] >= n` for position `i`, it means the rotation of `s` by `(i - n)` positions equals `t`.

### 2. **Off-by-One Error in Rotation Count**

**Pitfall**: Miscounting the number of valid rotations, especially forgetting that rotation by 0 (identity) and rotation by `n` (full rotation) are different cases.

**Example**: For `s = "abc"`, there are exactly 3 unique rotations:
- Rotation by 0: `"abc"` (identity)
- Rotation by 1: `"cab"` 
- Rotation by 2: `"bca"`

**Solution**: The code correctly handles this by:
- Treating rotation 0 separately with `dp[k][0]`
- Having exactly `n-1` non-zero rotations, each tracked by `dp[k][1]`
- The transition matrix correctly uses `n-1` and `n-2` for the coefficients

### 3. **Integer Overflow in Matrix Multiplication**

**Pitfall**: When multiplying large numbers in the matrix operations, intermediate results can overflow even before applying modulo, leading to incorrect results.

```python
# Incorrect - can overflow
result[i][j] = (result[i][j] + matrix_a[i][k] * matrix_b[k][j]) % MOD

# Correct - uses helper functions
result[i][j] = self.add(result[i][j], self.mul(matrix_a[i][k], matrix_b[k][j]))
```

**Solution**: The code uses dedicated `add()` and `mul()` helper functions that handle modulo arithmetic safely, preventing overflow at each step.

### 4. **Inefficient DP State Representation**

**Pitfall**: Tracking all `n` possible rotation states individually, leading to:
- Space complexity of `O(n × k)` for the DP table
- Time complexity of `O(n² × k)` for computation

**Why it's tempting**: It seems natural to track `dp[step][rotation]` for each possible rotation.

**Solution**: The code brilliantly reduces states to just 2:
- State 0: at original position
- State 1: at any non-zero rotation position

This optimization is valid because all non-zero rotations have symmetric transition probabilities, reducing complexity dramatically.

### 5. **Misunderstanding the Operation Constraint**

**Pitfall**: Forgetting that each operation must move at least 1 character but not all `n` characters. Some might incorrectly allow:
- Moving 0 characters (no operation)
- Moving all `n` characters (returns to original position)

**Solution**: The code enforces this by having exactly `n-1` valid operations from any position, as seen in the transition matrix. From position 0, we can reach any of the `n-1` non-zero positions. From any non-zero position, we can reach 0 or any of the other `n-2` non-zero positions.

