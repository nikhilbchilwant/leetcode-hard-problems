# 1359. Count All Valid Pickup and Delivery Options

> Source: https://algo.monster/liteproblems/1359

## Problem Statement

You have `n` orders, where each order consists of two services: a pickup service and a delivery service. You need to count how many different valid sequences exist for completing all these services.

The key constraint is that for each order `i`, the pickup service must happen before its corresponding delivery service. In other words, `pickup(i)` must always come before `delivery(i)` in any valid sequence.

Since you have `n` orders, you'll have a total of `2n` services to arrange (n pickups and n deliveries). You need to find the number of valid arrangements where each delivery happens after its corresponding pickup.

For example, if you have 2 orders:
- Order 1 has pickup P1 and delivery D1
- Order 2 has pickup P2 and delivery D2

Some valid sequences would be: P1, P2, D1, D2 or P1, P2, D2, D1 or P2, P1, D1, D2, etc.
But P1, D1, D2, P2 would be invalid because D2 comes before P2.

The answer can be very large, so you should return the result modulo `10^9 + 7`.

## Intuition

Let's think about how to build valid sequences step by step. When we have `i-1` orders already arranged in a valid sequence, we want to add the `i`-th order to this arrangement.

The key insight is to work backwards - instead of thinking about where to place the pickup first, let's think about where to place the delivery. Since we're building on top of a valid arrangement of `i-1` orders (which has `2(i-1)` services), adding one more order means we need to insert 2 new services: `P_i` and `D_i`.

Here's the clever part: if we fix the delivery `D_i` at a specific position, then the pickup `P_i` must come before it. This constraint actually makes counting easier!

Let's say we have `i` total orders. We can choose ANY of these `i` orders to be the last one delivered. This gives us `i` choices for which order's delivery appears at the very end of our sequence.

Once we've decided which order has its delivery at the end, where can we place its corresponding pickup? The pickup must come before the delivery, and since the delivery is at position `2i` (the last position), the pickup can be at any of the previous `2i-1` positions.

For the remaining `i-1` orders, they form a valid sub-problem that we've already solved: `f[i-1]`.

This gives us the recurrence: `f[i] = i × (2i - 1) × f[i-1]`

- `i`: number of ways to choose which order delivers last
- `(2i - 1)`: number of positions where we can place the pickup of that last-delivered order
- `f[i-1]`: number of valid arrangements for the remaining orders

Starting with `f[1] = 1` (only one way to arrange a single order: P1, D1), we can build up to `f[n]` by repeatedly applying this formula.

**Learn more about [Math](/problems/math-basics), [Dynamic Programming](/problems/dynamic_programming_intro) and [Combinatorics](/problems/backtracking) patterns.**

## Solution Approach

The implementation uses a [dynamic programming](/problems/dynamic_programming_intro) approach with space optimization. Instead of maintaining an array `f[1...n]` to store all intermediate results, we use a single variable `f` since each state only depends on the previous one.

Here's the step-by-step implementation:

1. **Initialize the base case**: Set `f = 1`, which represents `f[1] = 1` (one order has exactly one valid sequence: P1, D1).

2. **Define the modulo value**: `mod = 10^9 + 7` to handle large numbers and prevent overflow.

3. **Iterate from 2 to n**: For each number of orders `i` from 2 to `n`:
   - Apply the recurrence relation: `f = f × i × (2i - 1)`
   - Take modulo at each step: `f = (f × i × (2i - 1)) % mod`

The recurrence relation `f[i] = i × (2i - 1) × f[i-1]` is derived from:
- `i`: Number of ways to choose which order to deliver last
- `(2i - 1)`: Number of positions available for placing the pickup of the last-delivered order
- `f[i-1]`: Number of valid arrangements for the remaining `i-1` orders

**Space Optimization**: Since `f[i]` only depends on `f[i-1]`, we don't need to store all previous values. We update the same variable `f` in each iteration, reducing space complexity from `O(n)` to `O(1)`.

**Time Complexity**: `O(n)` - we iterate through numbers 2 to n once.

**Space Complexity**: `O(1)` - we only use a constant amount of extra space.

The final value of `f` after the loop completes is our answer - the total number of valid pickup/delivery sequences for `n` orders, modulo `10^9 + 7`.

## Example Walkthrough

Let's walk through calculating the number of valid sequences for n = 3 orders.

**Starting with n = 1:**
- Only one valid sequence: P1, D1
- f[1] = 1

**Building up to n = 2:**
- We have f[1] = 1 valid sequence for 1 order
- To add the 2nd order, we apply: f[2] = 2 × (2×2 - 1) × f[1]
- Breaking this down:
  - We can choose either order 1 or order 2 to deliver last (2 choices)
  - If we place a delivery at the end (position 4), its pickup can go in positions 1, 2, or 3 (that's 2×2 - 1 = 3 positions)
  - The remaining order forms f[1] = 1 valid arrangement
- f[2] = 2 × 3 × 1 = 6

Let's verify by listing all 6 sequences for 2 orders:
1. P1, P2, D1, D2 (order 2 delivered last, P2 at position 2)
2. P1, P2, D2, D1 (order 1 delivered last, P1 at position 1)
3. P2, P1, D1, D2 (order 2 delivered last, P2 at position 1)
4. P2, P1, D2, D1 (order 1 delivered last, P1 at position 2)
5. P1, D1, P2, D2 (order 2 delivered last, P2 at position 3)
6. P2, D2, P1, D1 (order 1 delivered last, P1 at position 3)

**Building up to n = 3:**
- We have f[2] = 6 valid sequences for 2 orders
- To add the 3rd order, we apply: f[3] = 3 × (2×3 - 1) × f[2]
- Breaking this down:
  - We can choose order 1, 2, or 3 to deliver last (3 choices)
  - If we place a delivery at position 6 (the end), its pickup can go in positions 1-5 (that's 2×3 - 1 = 5 positions)
  - The remaining 2 orders form f[2] = 6 valid arrangements
- f[3] = 3 × 5 × 6 = 90

**Implementation trace for n = 3:**
```
Initialize: f = 1, mod = 10^9 + 7

i = 2:
  f = (1 × 2 × 3) % mod = 6

i = 3:
  f = (6 × 3 × 5) % mod = 90

Return: 90
```

The algorithm correctly computes that there are 90 valid pickup/delivery sequences for 3 orders.

## Python Solution

```python
class Solution:
    def countOrders(self, n: int) -> int:
        """
        Calculate the number of valid pickup/delivery sequences for n orders.
      
        For n orders, we have n pickups and n deliveries (2n total actions).
        Each order's pickup must come before its delivery.
      
        The formula works by building up from 1 order to n orders:
        - For each new order i, we can insert its pickup and delivery into
          the existing sequence of 2*(i-1) actions in multiple ways
        - There are (2*i-1) positions to place the pickup (before or between existing actions)
        - Once pickup is placed, there are i positions for the delivery (after the pickup)
        - This gives us i * (2*i-1) ways to insert order i
      
        Args:
            n: Number of orders
          
        Returns:
            Number of valid sequences modulo 10^9 + 7
        """
        MOD = 10**9 + 7
      
        # Initialize result for 1 order (only 1 valid sequence: P1, D1)
        result = 1
      
        # For each additional order from 2 to n
        for i in range(2, n + 1):
            # Multiply by the number of ways to insert order i
            # i * (2*i - 1) represents:
            # - (2*i - 1) positions to place the pickup
            # - i positions to place the delivery after the pickup
            result = (result * i * (2 * i - 1)) % MOD
          
        return result
```

## Time and Space Complexity

The time complexity is `O(n)`, where `n` is the number of orders. This is because the algorithm uses a single for loop that iterates from 2 to n+1, performing a constant amount of work (multiplication and modulo operations) in each iteration.

The space complexity is `O(1)`. The algorithm only uses a fixed amount of extra space regardless of the input size - it maintains just two variables: `mod` (a constant) and `f` (which stores the running product). No additional data structures that scale with input size are used.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Integer Overflow Without Proper Modulo Application**

**Pitfall**: A common mistake is performing all multiplications first and then taking modulo at the end:

```python
# WRONG - This can cause integer overflow
result = 1
for i in range(2, n + 1):
    result = result * i * (2 * i - 1)
return result % MOD  # Taking modulo only at the end
```

**Why it's problematic**: For large values of n, the intermediate result can exceed Python's integer limits in other languages or become computationally expensive even in Python. While Python handles arbitrary precision integers, the multiplication becomes increasingly slow for very large numbers.

**Solution**: Apply modulo after each multiplication operation:

```python
# CORRECT - Apply modulo at each step
result = 1
for i in range(2, n + 1):
    result = (result * i * (2 * i - 1)) % MOD
```

### 2. **Incorrect Order of Operations in Modulo Arithmetic**

**Pitfall**: Breaking down the multiplication incorrectly when applying modulo:

```python
# WRONG - Can lead to incorrect results due to operation precedence
result = 1
for i in range(2, n + 1):
    result = result * (i % MOD) * ((2 * i - 1) % MOD) % MOD
```

**Why it's problematic**: Taking modulo of individual terms before multiplication is unnecessary and can sometimes lead to confusion. The correct approach is to multiply first, then take modulo of the entire product.

**Solution**: Multiply all terms together first, then apply modulo:

```python
# CORRECT - Cleaner and more efficient
result = 1
for i in range(2, n + 1):
    result = (result * i * (2 * i - 1)) % MOD
```

### 3. **Off-by-One Error in Loop Range**

**Pitfall**: Starting the loop from the wrong index or using incorrect range bounds:

```python
# WRONG - Missing the last order
for i in range(2, n):  # This only goes up to n-1
    result = (result * i * (2 * i - 1)) % MOD

# WRONG - Starting from wrong index
for i in range(1, n + 1):  # This incorrectly processes order 1
    result = (result * i * (2 * i - 1)) % MOD
```

**Solution**: Ensure the loop starts from 2 (since we've already handled order 1 as base case) and goes up to and including n:

```python
# CORRECT
for i in range(2, n + 1):  # Goes from 2 to n inclusive
    result = (result * i * (2 * i - 1)) % MOD
```

### 4. **Forgetting Edge Cases**

**Pitfall**: Not handling the case when n = 1 properly:

```python
# WRONG - Doesn't handle n=1 correctly
result = 0  # Wrong initialization
for i in range(2, n + 1):
    result = (result * i * (2 * i - 1)) % MOD
```

**Solution**: Initialize result to 1 (representing f[1] = 1), which correctly handles the base case:

```python
# CORRECT
result = 1  # Correct initialization for n=1
for i in range(2, n + 1):
    result = (result * i * (2 * i - 1)) % MOD
```

