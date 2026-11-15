# 3405. Count the Number of Arrays with K Matching Adjacent Elements

> Source: https://algo.monster/liteproblems/3405

## Problem Statement

You need to create arrays of size `n` where each element is an integer between 1 and `m` (inclusive). However, these arrays must satisfy a specific condition about consecutive equal elements.

A **good array** must meet these requirements:
- It has exactly `n` elements
- Each element is in the range `[1, m]`
- There are **exactly** `k` positions where an element equals its previous element

More specifically, when checking pairs of consecutive elements `(arr[i-1], arr[i])` for `i` from 1 to `n-1`, exactly `k` of these pairs should have equal elements.

For example:
- If `n=4`, `m=2`, `k=1`: The array `[1,1,2,1]` would be good because there's exactly one position (index 1) where `arr[1] == arr[0]`
- The array `[1,2,2,2]` would have `k=2` since there are two positions where consecutive elements are equal

Your task is to count how many different good arrays can be formed with the given parameters. Since this number can be very large, return the result modulo `10^9 + 7`.

The problem essentially asks: Given the array length, the range of values, and the required number of consecutive equal pairs, how many valid arrays exist?

## Intuition

Let's think about what it means to have exactly `k` positions where consecutive elements are equal. If we visualize the array, we can think of it as having "breakpoints" and "segments".

When two consecutive elements are different, we can imagine there's a "break" between them. When consecutive elements are the same, they belong to the same "segment". 

If we have `k` positions where consecutive elements are equal, then we have `n - 1 - k` positions where consecutive elements are different (since there are `n - 1` total pairs of consecutive elements).

This means our array is divided into `n - k` segments, where all elements within each segment are the same, but adjacent segments have different values.

For example, with array `[2,2,2,1,1,3,3]`:
- We have segments: `[2,2,2]`, `[1,1]`, `[3,3]`
- There are 4 positions with equal consecutive elements (positions 1,2,4,6)
- There are 2 breaks (between segments)

Now, how many ways can we create such an array?

**Step 1: Where to place the breaks?**
We need to choose which `n - 1 - k` positions out of `n - 1` possible positions will be "breaks". This is a combination problem: `C(n-1, n-1-k)` which equals `C(n-1, k)`.

**Step 2: What values to assign to each segment?**
- The first segment can be any value from `1` to `m`, giving us `m` choices
- Each subsequent segment must be different from the previous one (to maintain the break), giving us `m - 1` choices for each
- Since we have `n - k` segments total, we have `n - k - 1` segments after the first one

Therefore, the total number of ways to assign values is: `m × (m-1)^(n-k-1)`

Combining both parts: `C(n-1, k) × m × (m-1)^(n-k-1)`

This formula captures the essence of constructing all possible good arrays by first deciding where segments break, then assigning valid values to each segment.

**Learn more about [Math](/problems/math-basics) and [Combinatorics](/problems/backtracking) patterns.**

## Solution Approach

The implementation uses [combinatorics](/problems/backtracking) with precomputed factorials and modular arithmetic to efficiently calculate the result.

**Precomputation of Factorials and Inverses:**

The code starts by precomputing factorials and their modular inverses up to a maximum value:
- `f[i]` stores `i!` modulo `10^9 + 7`
- `g[i]` stores the modular inverse of `f[i]`

```python
mx = 10**5 + 10
mod = 10**9 + 7
f = [1] + [0] * mx  # f[i] = i! mod (10^9 + 7)
g = [1] + [0] * mx  # g[i] = inverse of f[i]

for i in range(1, mx):
    f[i] = f[i - 1] * i % mod
    g[i] = pow(f[i], mod - 2, mod)  # Fermat's little theorem for inverse
```

The modular inverse is calculated using Fermat's Little Theorem: if `p` is prime, then `a^(p-2) ≡ a^(-1) (mod p)`.

**Combination Function:**

The combination function `C(m, n)` is implemented using the precomputed factorials:

```python
def comb(m: int, n: int) -> int:
    return f[m] * g[n] * g[m - n] % mod
```

This calculates `C(m, n) = m! / (n! × (m-n)!)` using modular arithmetic.

**Main Solution:**

The solution applies the formula we derived:

```python
class Solution:
    def countGoodArrays(self, n: int, m: int, k: int) -> int:
        return comb(n - 1, k) * m * pow(m - 1, n - k - 1, mod) % mod
```

Breaking down the calculation:
1. `comb(n - 1, k)`: Number of ways to choose which `k` positions have equal consecutive elements
2. `m`: Number of choices for the first segment's value
3. `pow(m - 1, n - k - 1, mod)`: Number of ways to assign values to the remaining `n - k - 1` segments, where each must differ from the previous
4. All multiplications are done with modulo `10^9 + 7` to prevent overflow

The use of fast exponentiation (`pow` with three arguments) ensures efficient computation of large powers while maintaining the modular constraint.

**Time Complexity:** 
- Precomputation: `O(mx)` where `mx = 10^5 + 10`
- Per query: `O(log n)` for the power calculation
- Overall: `O(mx + log n)`

**Space Complexity:** `O(mx)` for storing the factorial and inverse arrays.

## Example Walkthrough

Let's walk through a concrete example with `n=5`, `m=3`, `k=2`.

We want arrays of length 5, with values from 1 to 3, and exactly 2 positions where consecutive elements are equal.

**Step 1: Understanding the structure**

With `k=2` positions having equal consecutive elements, we have:
- Total consecutive pairs: `n-1 = 4`
- Positions with different consecutive elements: `4-2 = 2`
- Number of segments: `n-k = 5-2 = 3`

So our array will have 3 segments where values within each segment are the same, but adjacent segments differ.

**Step 2: Choosing where to place the equal pairs**

We need to choose which 2 positions out of 4 will have equal consecutive elements.
This is `C(4,2) = 6` ways.

For example, if positions 1 and 3 have equal pairs:
- Position 0→1: equal (segment continues)
- Position 1→2: different (new segment)  
- Position 2→3: equal (segment continues)
- Position 3→4: different (new segment)

This gives us the pattern: `[a,a,b,b,c]` where a≠b and b≠c.

**Step 3: Assigning values to segments**

For our 3 segments:
- First segment: can be any of the 3 values (1, 2, or 3)
- Second segment: must differ from first, so 2 choices
- Third segment: must differ from second, so 2 choices

Total value assignments: `3 × 2 × 2 = 12`

**Step 4: Final calculation**

Total good arrays = `C(4,2) × 3 × 2² = 6 × 3 × 4 = 72`

Using our formula: `C(n-1,k) × m × (m-1)^(n-k-1)`
= `C(4,2) × 3 × 2²`
= `6 × 3 × 4`
= `72`

Some example good arrays:
- `[1,1,2,2,3]` - segments: [1,1], [2,2], [3]
- `[2,2,1,1,3]` - segments: [2,2], [1,1], [3]
- `[3,1,1,2,2]` - segments: [3], [1,1], [2,2]
- `[1,1,1,2,3]` - segments: [1,1,1], [2], [3]

Each follows the pattern of having exactly 2 positions where consecutive elements are equal.

## Python Solution

```python
# Constants for array size and modulo value
MAX_SIZE = 10**5 + 10
MOD = 10**9 + 7

# Precompute factorials and their modular inverses
factorial = [1] + [0] * MAX_SIZE
factorial_inverse = [1] + [0] * MAX_SIZE

for i in range(1, MAX_SIZE):
    # Compute factorial[i] = i! mod MOD
    factorial[i] = factorial[i - 1] * i % MOD
    # Compute modular inverse of factorial[i] using Fermat's Little Theorem
    # Since MOD is prime, inverse of a is a^(MOD-2) mod MOD
    factorial_inverse[i] = pow(factorial[i], MOD - 2, MOD)


def combination(total: int, choose: int) -> int:
    """
    Calculate C(total, choose) = total! / (choose! * (total-choose)!) mod MOD
    Uses precomputed factorials and their modular inverses for efficiency
  
    Args:
        total: Total number of items
        choose: Number of items to choose
  
    Returns:
        The combination value modulo MOD
    """
    return factorial[total] * factorial_inverse[choose] * factorial_inverse[total - choose] % MOD


class Solution:
    def countGoodArrays(self, n: int, m: int, k: int) -> int:
        """
        Count the number of good arrays of length n with values from 1 to m
        where exactly k adjacent pairs have equal values
      
        Args:
            n: Length of the array
            m: Maximum value in array (values range from 1 to m)
            k: Number of adjacent pairs with equal values
      
        Returns:
            Number of good arrays modulo MOD
        """
        # Choose k positions out of (n-1) adjacent pairs to have equal values
        positions = combination(n - 1, k)
      
        # First element can be any of m values
        first_element = m
      
        # For the remaining (n-k-1) positions where values must differ from previous,
        # each can be any of (m-1) values
        remaining_positions = pow(m - 1, n - k - 1, MOD)
      
        return positions * first_element * remaining_positions % MOD
```

## Time and Space Complexity

The preprocessing phase computes factorial values `f[i]` and their modular inverses `g[i]` for all values up to `mx`. This takes `O(mx)` time for computing factorials and `O(mx * log(mod))` time for computing modular inverses using `pow(f[i], mod - 2, mod)`. The space complexity for preprocessing is `O(mx)` to store arrays `f` and `g`.

For the main function `countGoodArrays`:
- The `comb(n - 1, k)` function performs `O(1)` operations since it just accesses precomputed values and does constant multiplications.
- Computing `pow(m - 1, n - k - 1, mod)` takes `O(log(n - k - 1))` time using fast exponentiation, which simplifies to `O(log(n - k))`.
- The remaining operations (multiplications and modulo) are `O(1)`.

Therefore, ignoring the preprocessing time and space:
- **Time Complexity**: `O(log(n - k))` due to the modular exponentiation operation
- **Space Complexity**: `O(1)` as the function only uses a constant amount of additional space for storing intermediate results

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Integer Overflow in Intermediate Calculations**

**Pitfall:** Even though we're using modular arithmetic, intermediate multiplication results can overflow before applying the modulo operation. For example, when calculating `factorial[total] * factorial_inverse[choose] * factorial_inverse[total - choose]`, the intermediate product might exceed Python's integer limits in other languages or cause precision issues.

**Solution:** Apply modulo after each multiplication operation rather than at the end:
```python
# Instead of:
return factorial[total] * factorial_inverse[choose] * factorial_inverse[total - choose] % MOD

# Use:
result = factorial[total] * factorial_inverse[choose] % MOD
result = result * factorial_inverse[total - choose] % MOD
return result
```

### 2. **Edge Case: k = 0**

**Pitfall:** When `k = 0`, no consecutive elements should be equal. The formula `pow(m - 1, n - k - 1, MOD)` becomes `pow(m - 1, n - 1, MOD)`. However, if not careful with the logic, one might forget that the first element can still be any of the `m` values.

**Solution:** The current formula handles this correctly, but it's important to verify:
- For `k = 0`: Result should be `m * (m-1)^(n-1)` (first element: m choices, rest: m-1 choices each)
- Test with small examples: `n=3, m=2, k=0` should give `2 * 1 * 1 = 2` arrays: `[1,2,1]` and `[2,1,2]`

### 3. **Edge Case: k = n - 1**

**Pitfall:** When `k = n - 1`, all consecutive pairs must be equal, meaning all elements must be the same. The formula `pow(m - 1, n - k - 1, MOD)` becomes `pow(m - 1, -1, MOD)`, which would cause an error or unexpected behavior.

**Solution:** Handle this special case explicitly:
```python
def countGoodArrays(self, n: int, m: int, k: int) -> int:
    if k == n - 1:
        # All elements must be the same
        return m
    return combination(n - 1, k) * m * pow(m - 1, n - k - 1, MOD) % MOD
```

### 4. **Invalid Input: k > n - 1**

**Pitfall:** If `k > n - 1`, it's impossible to have more equal consecutive pairs than there are consecutive pairs in total. The combination function would fail or return 0.

**Solution:** Add input validation:
```python
def countGoodArrays(self, n: int, m: int, k: int) -> int:
    if k > n - 1 or k < 0:
        return 0
    if k == n - 1:
        return m
    return combination(n - 1, k) * m * pow(m - 1, n - k - 1, MOD) % MOD
```

### 5. **Precomputation Array Size**

**Pitfall:** The precomputed factorial arrays have a fixed size (`MAX_SIZE = 10**5 + 10`). If `n` exceeds this limit, accessing `factorial[n-1]` would cause an index error.

**Solution:** Either:
- Increase `MAX_SIZE` based on problem constraints
- Add dynamic computation for values beyond the precomputed range
- Add bounds checking:
```python
def countGoodArrays(self, n: int, m: int, k: int) -> int:
    if n - 1 >= MAX_SIZE:
        # Handle large n values with dynamic computation
        # or raise an appropriate error
        raise ValueError(f"n value {n} exceeds precomputed limits")
    # ... rest of the solution
```

### 6. **Modular Inverse Computation**

**Pitfall:** Using `pow(factorial[i], MOD - 2, MOD)` assumes MOD is prime (which it is for `10^9 + 7`). If the modulo value changes to a non-prime number, this method won't work.

**Solution:** Document the assumption or use the Extended Euclidean Algorithm for a more general solution:
```python
def mod_inverse(a, mod):
    """Compute modular inverse using Extended Euclidean Algorithm"""
    def extended_gcd(a, b):
        if a == 0:
            return b, 0, 1
        gcd, x1, y1 = extended_gcd(b % a, a)
        x = y1 - (b // a) * x1
        y = x1
        return gcd, x, y
  
    _, x, _ = extended_gcd(a % mod, mod)
    return (x % mod + mod) % mod
```

