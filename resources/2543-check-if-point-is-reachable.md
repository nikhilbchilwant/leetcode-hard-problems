# 2543. Check if Point Is Reachable

> Source: https://algo.monster/liteproblems/2543

## Problem Statement

You start at point `(1, 1)` on an infinitely large grid and need to determine if you can reach a target point `(targetX, targetY)` through a series of moves.

From any point `(x, y)`, you can move to exactly one of these four positions:
- `(x, y - x)` - subtract the x-coordinate from the y-coordinate
- `(x - y, y)` - subtract the y-coordinate from the x-coordinate  
- `(2 * x, y)` - double the x-coordinate
- `(x, 2 * y)` - double the y-coordinate

Given integers `targetX` and `targetY` representing your destination coordinates, return `true` if you can reach `(targetX, targetY)` from `(1, 1)` using any number of these moves, and `false` otherwise.

The key insight is that the first two operations preserve the greatest common divisor (gcd) of the coordinates while only changing their values through addition/subtraction. The last two operations can multiply the gcd by powers of 2. Starting from `(1, 1)` where `gcd(1, 1) = 1 = 2^0`, the only way to reach a target is if `gcd(targetX, targetY)` is a power of 2.

## Intuition

To understand why this problem reduces to checking if the GCD is a power of 2, let's analyze what each operation does mathematically.

Starting from `(1, 1)`, we notice that the GCD of our coordinates begins at 1. Let's track how each operation affects the GCD:

1. `(x, y) → (x, y - x)`: The GCD remains unchanged because `gcd(x, y - x) = gcd(x, y)`. This is a fundamental property - subtracting one number from another doesn't change their GCD.

2. `(x, y) → (x - y, y)`: Similarly, `gcd(x - y, y) = gcd(x, y)`. The GCD is preserved.

3. `(x, y) → (2x, y)`: Here `gcd(2x, y) = gcd(x, y)` if y is odd, or `2 × gcd(x, y)` if y is even. This can potentially double the GCD.

4. `(x, y) → (x, 2y)`: By the same logic, this can also potentially double the GCD.

The key realization is that operations 1 and 2 only rearrange the values while preserving the GCD, while operations 3 and 4 can only multiply the GCD by 2. Since we start with `gcd(1, 1) = 1 = 2^0`, the GCD can only ever become `2^k` for some non-negative integer k.

Working backwards helps verify this is also sufficient. If we can reach any point with `gcd = 2^k`, we can reverse the operations: divide by 2 when possible, and use addition/subtraction operations to reduce the coordinates. The process converges because we can always make the larger coordinate smaller while maintaining the GCD, eventually reaching `(1, 1)` if and only if the GCD was initially a power of 2.

The elegant bit manipulation check `x & (x - 1) == 0` works because a power of 2 in binary has exactly one bit set (like `1000`), and subtracting 1 flips all bits up to that position (giving `0111`). The AND operation between these yields 0 only for powers of 2.

**Learn more about [Math](/problems/math-basics) patterns.**

## Solution Approach

The solution implements a mathematical approach that uses the GCD property we discovered. Here's how the algorithm works:

**Step 1: Calculate the GCD**
```python
x = gcd(targetX, targetY)
```
We compute the greatest common divisor of the target coordinates using Python's built-in `gcd` function from the [math](/problems/math-basics) module. This gives us the fundamental invariant we need to check.

**Step 2: Check if GCD is a Power of 2**
```python
return x & (x - 1) == 0
```
This bit manipulation trick efficiently determines if `x` is a power of 2:
- If `x` is a power of 2, it has exactly one bit set in binary (e.g., `8 = 1000₂`)
- Subtracting 1 from a power of 2 flips all bits after the single set bit (e.g., `7 = 0111₂`)
- The bitwise AND of these two numbers equals 0 only when `x` is a power of 2
- For example: `8 & 7 = 1000₂ & 0111₂ = 0000₂ = 0`

**Why This Works:**

The reference solution explains that we can think about the problem in reverse - starting from `(targetX, targetY)` and working back to `(1, 1)`. 

When moving backward:
- `(x, y)` can come from `(x, x+y)`, `(x+y, y)`, `(x/2, y)` (if x is even), or `(x, y/2)` (if y is even)
- We keep dividing by 2 whenever possible until both coordinates are odd
- Once both are odd, we use the addition operations to reduce the larger coordinate
- Since `(x+y)/2 < max(x, y)` when both are odd and different, we can always make progress
- The process terminates at `(1, 1)` if and only if the original GCD was a power of 2

The algorithm's elegance lies in reducing a complex reachability problem to a simple GCD check with O(log(min(targetX, targetY))) time complexity for the GCD calculation and O(1) for the power-of-2 check.

## Example Walkthrough

Let's trace through two examples to understand how the solution works.

**Example 1: Can we reach (6, 9)?**

First, we calculate `gcd(6, 9)`:
- 9 = 1 × 6 + 3
- 6 = 2 × 3 + 0
- So `gcd(6, 9) = 3`

Now we check if 3 is a power of 2:
- 3 in binary: `011`
- 3 - 1 = 2 in binary: `010`
- 3 & 2 = `011 & 010 = 010 = 2 ≠ 0`

Since 3 is not a power of 2, we **cannot** reach (6, 9) from (1, 1).

To understand why, let's trace what happens to the GCD starting from (1, 1):
- Initial GCD = 1
- Operations (x, y-x) and (x-y, y) preserve the GCD
- Operations (2x, y) and (x, 2y) can only multiply GCD by 2
- So from GCD = 1, we can only ever reach points with GCD = 1, 2, 4, 8, 16, ...
- We can never reach a point with GCD = 3

**Example 2: Can we reach (10, 14)?**

First, we calculate `gcd(10, 14)`:
- 14 = 1 × 10 + 4
- 10 = 2 × 4 + 2
- 4 = 2 × 2 + 0
- So `gcd(10, 14) = 2`

Now we check if 2 is a power of 2:
- 2 in binary: `10`
- 2 - 1 = 1 in binary: `01`
- 2 & 1 = `10 & 01 = 00 = 0`

Since 2 is a power of 2, we **can** reach (10, 14) from (1, 1).

Here's one possible path:
1. Start at (1, 1), GCD = 1
2. (1, 1) → (2, 1) using operation (2x, y), GCD = 1
3. (2, 1) → (2, 2) using operation (x, 2y), GCD = 2
4. (2, 2) → (4, 2) using operation (2x, y), GCD = 2
5. (4, 2) → (4, 6) using operation (x, y-x) reversed, GCD = 2
6. (4, 6) → (4, 10) using operation (x, y-x) reversed, GCD = 2
7. (4, 10) → (4, 14) using operation (x, y-x) reversed, GCD = 2
8. (4, 14) → (10, 14) using operation (x-y, y) reversed, GCD = 2

The actual path doesn't matter for our algorithm - we only need to verify that the GCD is achievable, which it is since 2 = 2^1.

## Python Solution

```python
from math import gcd

class Solution:
    def isReachable(self, targetX: int, targetY: int) -> bool:
        # Find the greatest common divisor of targetX and targetY
        common_divisor = gcd(targetX, targetY)
      
        # Check if the GCD is a power of 2
        # A number is a power of 2 if it has only one bit set in binary representation
        # Using bitwise AND with (number - 1) will result in 0 only for powers of 2
        # Example: 8 (1000) & 7 (0111) = 0, but 6 (0110) & 5 (0101) = 4 (0100)
        return common_divisor & (common_divisor - 1) == 0
```

## Time and Space Complexity

The time complexity is `O(log(min(targetX, targetY)))`. This is because the GCD (Greatest Common Divisor) calculation using the Euclidean algorithm takes logarithmic time with respect to the smaller of the two input values. The Euclidean algorithm repeatedly applies the modulo operation, and the number of iterations is bounded by `O(log(min(targetX, targetY)))`. The subsequent bitwise operation `x & (x - 1) == 0` to check if x is a power of 2 takes constant time `O(1)`.

The space complexity is `O(1)`. The algorithm only uses a constant amount of extra space to store the variable `x` (the GCD result), regardless of the input size. The GCD calculation itself (assuming it uses the iterative Euclidean algorithm) also requires only constant space for temporary variables.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Integer Overflow in GCD Calculation**
When dealing with very large input values for `targetX` and `targetY`, intermediate calculations in the GCD algorithm could potentially cause issues in languages with fixed integer sizes. While Python handles arbitrary precision integers automatically, developers implementing this in other languages might face overflow.

**Solution:** In Python, this isn't a concern due to automatic big integer handling. In other languages, use appropriate data types (like `long long` in C++) or implement overflow-safe GCD algorithms.

### 2. **Misunderstanding the Bit Manipulation Check**
A common mistake is incorrectly implementing the power-of-2 check. Some developers might try:
- `x & (x + 1) == 0` (incorrect - this doesn't work)
- `x % 2 == 0` (incorrect - only checks if even, not if power of 2)
- Forgetting to handle the edge case where `x = 0`

**Solution:** Always use `x & (x - 1) == 0` for checking powers of 2. Note that this returns `true` for `x = 0` as well, but since GCD is always positive for positive inputs, this edge case doesn't affect our solution.

### 3. **Not Handling the Case Where Both Coordinates Equal 1**
While the problem states we start at `(1, 1)`, some might overlook that `(1, 1)` itself is a valid target. The GCD of `(1, 1)` is 1, which is `2^0`, a power of 2.

**Solution:** The current implementation correctly handles this case since `1 & 0 = 0`, returning `true` as expected.

### 4. **Attempting to Simulate the Actual Path**
A significant pitfall is trying to use BFS, DFS, or dynamic programming to find the actual path from `(1, 1)` to the target. This approach would be inefficient and could lead to:
- Stack overflow in recursive solutions
- Memory issues with large coordinates
- Time limit exceeded due to exploring exponentially many states

**Solution:** Trust the mathematical insight that checking if GCD is a power of 2 is sufficient. The proof guarantees that if this condition holds, a path exists, even if we don't explicitly construct it.

### 5. **Incorrect Reverse Logic Implementation**
Some might try to implement the reverse traversal mentioned in the explanation, working from `(targetX, targetY)` back to `(1, 1)`. This requires careful handling of:
- When to divide by 2 vs. when to subtract
- Avoiding infinite loops
- Handling negative numbers correctly

**Solution:** The GCD-based approach elegantly avoids all these complications by reducing the problem to a single mathematical check.

