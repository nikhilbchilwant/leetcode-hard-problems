# 3145. Find Products of Elements of Big Array

> Source: https://algo.monster/liteproblems/3145

## Problem Statement

Given the concept of a **powerful array** for a non-negative integer `x`, which is defined as the shortest sorted array of powers of two that sum up to `x`. For example:
- The powerful array of 1 is `[1]` (binary: 0001)
- The powerful array of 10 is `[2, 8]` (binary: 01010)
- The powerful array of 13 is `[1, 4, 8]` (binary: 01101)

The powerful array essentially represents the powers of 2 corresponding to the 1-bits in the binary representation of the number.

A special array called `big_nums` is created by concatenating the powerful arrays of all positive integers in ascending order:
- Powerful array of 1: `[1]`
- Powerful array of 2: `[2]`
- Powerful array of 3: `[1, 2]`
- Powerful array of 4: `[4]`
- Powerful array of 5: `[1, 4]`
- And so on...

So `big_nums = [1, 2, 1, 2, 4, 1, 4, 2, 4, 1, 2, 4, 8, ...]`

You are given a 2D integer matrix `queries`, where each query is of the form `[from_i, to_i, mod_i]`. For each query, you need to:
1. Take the subarray from `big_nums[from_i]` to `big_nums[to_i]` (inclusive)
2. Calculate the product of all elements in this subarray
3. Return the result modulo `mod_i`

The task is to return an array `answer` where `answer[i]` contains the result for the `i-th` query.

**Key Insight**: Since all elements in `big_nums` are powers of 2, the product of elements like `2^a × 2^b × 2^c` equals `2^(a+b+c)`. Therefore, the problem reduces to finding the sum of exponents in the specified range and then computing `2^sum % mod`.

## Intuition

Since every element in `big_nums` is a power of 2, multiplying them together means adding their exponents. For example, `2 × 8 × 4 = 2^1 × 2^3 × 2^2 = 2^(1+3+2) = 2^6`. This transforms our problem from finding products to finding sums of exponents.

The challenge is that `big_nums` can be extremely large (as we concatenate powerful arrays for all positive integers), so we can't build it explicitly. We need a way to directly calculate the sum of exponents for any range `[left, right]` without constructing the entire array.

The key observation is that there's a pattern in how powerful arrays are generated. Looking at the binary representations:
- Numbers from `[1, 1]` have at most 1 bit set
- Numbers from `[2, 3]` have at most 2 bits set
- Numbers from `[4, 7]` have at most 3 bits set
- Numbers from `[8, 15]` have at most 4 bits set

More generally, numbers in the range `[2^i, 2^(i+1) - 1]` share similar bit patterns. The powerful arrays for these numbers follow a recursive structure - they're similar to the powerful arrays of numbers in `[0, 2^i - 1]`, but with an additional `2^i` term.

This pattern allows us to precompute:
- `cnt[i]`: the total count of elements in powerful arrays for all numbers from `0` to `2^i - 1`
- `s[i]`: the sum of all exponents in these powerful arrays

For any arbitrary position in `big_nums`, we can use [binary search](/problems/binary_search_intro) to find which number's powerful array it belongs to. Once we know the number, we can decompose it bit by bit to find the exact exponent at that position.

The solution uses a helper function `f(i)` to calculate the sum of exponents from position `0` to position `i-1` in `big_nums`. Then for each query `[left, right, mod]`, the answer is `2^(f(right + 1) - f(left)) % mod`, which we compute using fast exponentiation.

**Learn more about [Binary Search](/problems/binary_search_intro) patterns.**

## Solution Approach

The solution consists of three main components: precomputation, [binary search](/problems/binary_search_intro) with bit manipulation, and query processing.

### 1. Precomputation of Pattern Tables

First, we precompute two arrays `cnt` and `s` for the first `m=50` groups (sufficient since `2^50` covers the problem constraints):

```python
m = 50
cnt = [0] * (m + 1)
s = [0] * (m + 1)
p = 1
for i in range(1, m + 1):
    cnt[i] = cnt[i - 1] * 2 + p
    s[i] = s[i - 1] * 2 + p * (i - 1)
    p *= 2
```

- `cnt[i]`: Total number of elements in powerful arrays for numbers `[0, 2^i - 1]`
- `s[i]`: Sum of all exponents in these powerful arrays
- The recurrence relations come from the fact that numbers in `[2^i, 2^(i+1) - 1]` have patterns similar to `[0, 2^i - 1]` but with an added `2^i` term

### 2. Computing Index and Sum for Any Number

The function `num_idx_and_sum(x)` calculates the total number of elements and sum of exponents for all powerful arrays up to number `x`:

```python
def num_idx_and_sum(x: int) -> tuple:
    idx = 0
    total_sum = 0
    while x:
        i = x.bit_length() - 1  # Find highest set bit
        idx += cnt[i]
        total_sum += s[i]
        x -= 1 << i  # Remove the highest bit
        total_sum += (x + 1) * i  # Add contribution of remaining numbers
        idx += x + 1
    return (idx, total_sum)
```

This works by processing `x` bit by bit from the highest bit. For example, if `x = 13 = 2^3 + 2^2 + 2^0`:
- First, handle numbers `[0, 2^3 - 1]` using precomputed values
- Then handle remaining numbers `[2^3, 13]`, which is like handling `[0, 5]` with each exponent increased by 3
- Recursively process until `x` becomes 0

### 3. Finding Prefix Sum at Position i

The function `f(i)` finds the sum of exponents from position `0` to position `i-1` in `big_nums`. We use [binary search](/problems/binary_search_intro) to find the largest number whose powerful array ends before position `i`.

The feasible condition is: "Does the count of elements up to number `mid` reach at least `i`?" We want the last position where this is FALSE (count < i), so we search for the first TRUE and take one position before.

```python
def f(i: int) -> int:
    left, right = 0, 1 << m
    first_true_index = -1

    while left <= right:
        mid = (left + right) // 2
        idx, _ = num_idx_and_sum(mid)
        if idx >= i:  # feasible: count reaches target
            first_true_index = mid
            right = mid - 1
        else:
            left = mid + 1

    # The answer is one position before first_true_index
    target = first_true_index - 1 if first_true_index != -1 else (1 << m)

    idx, total_sum = num_idx_and_sum(target)
    i -= idx
    x = target + 1
    for _ in range(i):
        y = x & -x  # Extract lowest set bit
        total_sum += y.bit_length() - 1
        x -= y
    return total_sum
```

This uses the standard binary search template to find the first number where the cumulative count reaches `i`. Then we take the number just before it (since we want the largest number whose powerful array ends before position `i`) and calculate the remaining elements using bit manipulation (`x & -x` extracts the lowest set bit).

### 4. Processing Queries

Finally, for each query `[left, right, mod]`:

```python
def findProductsOfElements(self, queries: List[List[int]]) -> List[int]:
    return [pow(2, f(right + 1) - f(left), mod) for left, right, mod in queries]
```

The answer is `2^(f(right + 1) - f(left)) % mod`, computed using Python's built-in `pow` function with three arguments for efficient modular exponentiation.

The time complexity is `O(q * log^2(n))` where `q` is the number of queries and `n` is the range of indices, due to the [binary search](/problems/binary_search_intro) and bit operations.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

Suppose we have a query `[7, 11, 13]`, meaning we want to find the product of elements from index 7 to 11 in `big_nums`, then take the result modulo 13.

**Step 1: Understanding big_nums**

First, let's build a portion of `big_nums` by finding powerful arrays for the first few numbers:
- 1 (binary: 0001) → powerful array: [1]
- 2 (binary: 0010) → powerful array: [2]
- 3 (binary: 0011) → powerful array: [1, 2]
- 4 (binary: 0100) → powerful array: [4]
- 5 (binary: 0101) → powerful array: [1, 4]
- 6 (binary: 0110) → powerful array: [2, 4]
- 7 (binary: 0111) → powerful array: [1, 2, 4]

So `big_nums = [1, 2, 1, 2, 4, 1, 4, 2, 4, 1, 2, 4, ...]`
- Indices:     [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, ...]

**Step 2: Extract the subarray**

For our query `[7, 11, 13]`, we need elements at indices 7-11:
- `big_nums[7:12] = [2, 4, 1, 2, 4]`

**Step 3: Convert to exponents**

Since these are all powers of 2:
- 2 = 2^1, 4 = 2^2, 1 = 2^0, 2 = 2^1, 4 = 2^2
- Exponents: [1, 2, 0, 1, 2]

**Step 4: Calculate sum of exponents**

Using the `f()` function:
- `f(12)` calculates sum of exponents from index 0 to 11
- `f(7)` calculates sum of exponents from index 0 to 6

Let's trace through `f(7)`:
1. Binary search finds that powerful arrays for numbers 1-5 contribute exactly 7 elements
2. So position 7 is at the start of number 6's powerful array
3. `f(7)` returns sum of exponents for indices 0-6, which equals:
   - Number 1: exponent 0 (for element 1)
   - Number 2: exponent 1 (for element 2)
   - Number 3: exponents 0, 1 (for elements 1, 2)
   - Number 4: exponent 2 (for element 4)
   - Number 5: exponents 0, 2 (for elements 1, 4)
   - Total: 0 + 1 + (0+1) + 2 + (0+2) = 6

Similarly, `f(12)` would calculate the sum including indices 7-11:
- Additional exponents from indices 7-11: 1, 2, 0, 1, 2
- `f(12) = f(7) + (1+2+0+1+2) = 6 + 6 = 12`

**Step 5: Compute the result**

The sum of exponents in range [7, 11] is:
- `f(12) - f(7) = 12 - 6 = 6`

Therefore, the product is:
- `2^6 = 64`

And the final answer:
- `64 % 13 = 12`

The beauty of this approach is that we never actually build `big_nums`. Instead, we use the pattern in binary representations to directly compute the sum of exponents for any range, making it efficient even for very large indices.

## Python Solution

```python
from typing import List


# Precompute values for optimization
MAX_BITS = 50
bit_count = [0] * (MAX_BITS + 1)  # Count of bits up to position i
bit_sum = [0] * (MAX_BITS + 1)    # Sum of bit positions up to position i
power_of_two = 1

# Precompute cumulative bit counts and sums for powers of 2
for i in range(1, MAX_BITS + 1):
    bit_count[i] = bit_count[i - 1] * 2 + power_of_two
    bit_sum[i] = bit_sum[i - 1] * 2 + power_of_two * (i - 1)
    power_of_two *= 2


def calculate_bit_index_and_sum(n: int) -> tuple:
    """
    Calculate the total number of bits and sum of bit positions
    for all numbers from 1 to n in their binary representation.

    Args:
        n: Upper bound of the range

    Returns:
        Tuple of (total_bit_count, sum_of_bit_positions)
    """
    total_count = 0
    total_sum = 0

    while n > 0:
        # Find the position of the most significant bit
        msb_position = n.bit_length() - 1

        # Add precomputed values for complete powers of 2
        total_count += bit_count[msb_position]
        total_sum += bit_sum[msb_position]

        # Remove the highest power of 2
        n -= 1 << msb_position

        # Add contribution from remaining numbers
        total_sum += (n + 1) * msb_position
        total_count += n + 1

    return (total_count, total_sum)


def find_bit_position_sum(index: int) -> int:
    """
    Find the sum of bit positions up to the given index in the
    sequence of all set bits from binary representations.

    Args:
        index: The target index in the bit sequence

    Returns:
        Sum of bit positions up to the index
    """
    # Binary search using the standard template
    # Find first number where cumulative count >= index
    left, right = 0, 1 << MAX_BITS
    first_true_index = -1

    while left <= right:
        mid = (left + right) // 2
        current_index, _ = calculate_bit_index_and_sum(mid)

        if current_index >= index:  # feasible: count reaches target
            first_true_index = mid
            right = mid - 1
        else:
            left = mid + 1

    # We want the largest number whose count is still < index
    # That's one position before first_true_index
    target = first_true_index - 1 if first_true_index != -1 else (1 << MAX_BITS)

    # Calculate sum up to the found number
    bit_index, position_sum = calculate_bit_index_and_sum(target)

    # Calculate remaining bits needed
    remaining_bits = index - bit_index
    current_number = target + 1

    # Add bit positions for remaining bits using lowbit technique
    for _ in range(remaining_bits):
        # Get the rightmost set bit (lowbit)
        lowest_bit = current_number & -current_number
        # Add the position of this bit (0-indexed)
        position_sum += lowest_bit.bit_length() - 1
        # Remove the rightmost set bit
        current_number -= lowest_bit

    return position_sum


class Solution:
    def findProductsOfElements(self, queries: List[List[int]]) -> List[int]:
        """
        Find products of elements in given ranges where elements are powers of 2
        based on bit positions in the binary representation sequence.

        Args:
            queries: List of [left, right, mod] queries

        Returns:
            List of results where each result is 2^(sum of bit positions) % mod
        """
        results = []

        for left, right, mod in queries:
            # Calculate sum of bit positions in range [left, right]
            # Note: Using 1-indexed positions, so add 1 to right
            sum_of_positions = find_bit_position_sum(right + 1) - find_bit_position_sum(left)

            # Result is 2^sum_of_positions % mod
            result = pow(2, sum_of_positions, mod)
            results.append(result)

        return results
```

## Time and Space Complexity

**Time Complexity:** `O(q × log M)`

The main time complexity comes from the `findProductsOfElements` method which processes `q` queries. For each query, it calls function `f` twice (for `right + 1` and `left`).

Inside function `f`:
- The binary search loop runs `O(log M)` iterations, where `M` is the upper bound (`1 << m` which represents `2^50`)
- Within each binary search iteration, `num_idx_and_sum` is called, which has a while loop that runs at most `O(log M)` times (based on the bit length of `x`)
- After the binary search, there's another loop that runs `i` times in the worst case, but since we're working with bit operations on numbers up to `M`, this is also bounded by `O(log M)`

Therefore, for each query: `O(log M × log M)` which simplifies to `O(log² M)` for the binary search portion, but the dominant operation is actually `O(log M)` due to the nature of the bit manipulation operations.

Overall: `O(q × log M)` where `q` is the number of queries and `M ≤ 10^15` in this problem.

**Space Complexity:** `O(log M)`

The space complexity is determined by:
- The precomputed arrays `cnt` and `s`, each of size `m + 1` where `m = 50`, which is `O(log M)` since `m = log₂(M)`
- The recursive depth and local variables in functions are `O(1)`
- No additional data structures that scale with input size are used

Therefore, the space complexity is `O(log M)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Using Wrong Binary Search Template Variant**

**The Pitfall:**
Using different binary search variants that don't match the standard template can lead to off-by-one errors and incorrect results:
```python
# WRONG: Non-standard variant
while left < right:
    mid = (left + right + 1) >> 1
    if count < index:
        left = mid
    else:
        right = mid - 1
return left  # Returns result directly without tracking
```

**The Solution:**
Use the standard template that tracks `first_true_index`:
```python
# CORRECT: Standard template
left, right = 0, max_value
first_true_index = -1

while left <= right:
    mid = (left + right) // 2
    if feasible(mid):  # count >= index
        first_true_index = mid
        right = mid - 1
    else:
        left = mid + 1

# Handle result based on first_true_index
target = first_true_index - 1 if first_true_index != -1 else max_value
```

### 2. **Off-by-One Errors in Index Calculation**

The most common pitfall in this problem is mishandling the 0-indexed vs 1-indexed conversion when calculating bit positions and array indices.

**The Pitfall:**
- The problem uses 0-indexed arrays for queries (`big_nums[from_i]` to `big_nums[to_i]`)
- The internal logic often works with 1-indexed counting for numbers (1, 2, 3, ...)
- Mixing these can lead to incorrect range calculations

**Example of the Error:**
```python
# WRONG: Forgetting to adjust for 0-indexing
def findProductsOfElements(self, queries):
    return [pow(2, f(right) - f(left), mod) for left, right, mod in queries]
    # This misses the last element in the range!
```

**The Solution:**
```python
# CORRECT: Properly handle inclusive range [left, right]
def findProductsOfElements(self, queries):
    return [pow(2, f(right + 1) - f(left), mod) for left, right, mod in queries]
    # f(right + 1) gives sum up to and including index right
```

### 3. **Integer Overflow in Intermediate Calculations**

**The Pitfall:**
When calculating `total_sum += (n + 1) * msb_position`, the multiplication can overflow for large values even though Python handles big integers well. The real issue is when using this pattern in other languages or when optimizing with fixed-size integers.

**The Solution:**
Always use appropriate data types and be mindful of the order of operations:
```python
# Better practice - accumulate incrementally if needed
total_sum += (n + 1) * msb_position  # Python handles this, but be careful in C++/Java
```

### 4. **Incorrect Lowbit Extraction Logic**

**The Pitfall:**
The lowbit operation `x & -x` extracts the rightmost set bit, but developers often misunderstand what this returns:
```python
# WRONG: Thinking lowbit gives the bit position directly
bit_position = x & -x  # This gives 2^position, not position!

# WRONG: Off-by-one in bit position calculation
position = (x & -x).bit_length()  # This is 1-indexed, not 0-indexed
```

**The Solution:**
```python
# CORRECT: Get the actual bit position (0-indexed)
lowest_bit = current_number & -current_number
position = lowest_bit.bit_length() - 1  # Subtract 1 for 0-indexing
```

### 5. **Modular Arithmetic Errors**

**The Pitfall:**
Computing large powers before taking modulo:
```python
# WRONG: Will cause overflow or be extremely slow
result = (2 ** sum_of_positions) % mod
```

**The Solution:**
```python
# CORRECT: Use built-in modular exponentiation
result = pow(2, sum_of_positions, mod)
```

### 6. **Precomputation Array Size**

**The Pitfall:**
Using insufficient precomputation size:
```python
# WRONG: May not cover all test cases
MAX_BITS = 30  # Too small for large indices
```

**The Solution:**
```python
# CORRECT: Use sufficient size based on constraints
MAX_BITS = 50  # Covers up to 2^50 which is more than enough
```

**Key Debugging Tips:**
1. Test with small examples where you can manually verify the `big_nums` array
2. Print intermediate values in `calculate_bit_index_and_sum` to verify correctness
3. Verify that `f(i)` returns the correct cumulative sum for known positions
4. Test edge cases: queries with left=0, right=0, and single-element ranges

