# 964. Least Operators to Express Number

> Source: https://algo.monster/liteproblems/964

## Problem Statement

You are given a positive integer `x` and need to create a mathematical expression using only the number `x` and the operators `+`, `-`, `*`, `/` to equal a target value. The expression must follow the format: `x (op1) x (op2) x (op3) x ...` where each operator is one of the four basic operations.

For example, if `x = 3` and you want to reach a certain target, you could write expressions like:
- `3 * 3 / 3 + 3 - 3` which equals `3`
- `3 + 3 + 3` which equals `9`
- `3 * 3 - 3 / 3` which equals `8`

The expression must follow these rules:
1. Division returns rational numbers (not just integers)
2. No parentheses are allowed anywhere in the expression
3. Standard order of operations applies (multiplication and division before addition and subtraction)
4. Unary negation is not allowed (you cannot write `-x`, only operations like `x - x` are valid)

Your task is to find the **minimum number of operators** needed to create an expression that equals the given `target`. 

For instance, if `x = 3` and `target = 19`, one possible expression is `3 + 3 * 3 + 3 + 3 / 3`, which uses 5 operators and equals 19. The function should return the minimum number of operators required.

## Intuition

The key insight is to think about what values we can create using `x` and how many operators each requires. Let's break this down:

- `x` itself requires 0 operators
- `x + x = 2x` requires 1 operator
- `x - x = 0` requires 1 operator  
- `x * x = x²` requires 1 operator
- `x / x = 1` requires 1 operator

Notice that we can build powers of `x`:
- `x² = x * x` (1 operator)
- `x³ = x * x * x` (2 operators)
- `x^k` requires `k-1` operators

We can also create `1` using `x / x` (1 operator), and from there build any small integer by adding `1`s together. For example, to get `3` when `x = 5`, we could do `5 / 5 + 5 / 5 + 5 / 5`.

The crucial realization is that any target number can be represented as a combination of powers of `x`. Think of it like converting the target to base `x`, but with the twist that we can use both positive and negative coefficients.

For any value `v` we want to reach:
1. If `v` is small (less than or equal to `x`), we can either:
   - Build it using `1`s: `x/x + x/x + ...` (requires `2v - 1` operators total)
   - Or subtract from `x`: `x - (something)` where we recursively find how to build `x - v`

2. If `v` is large, we find the appropriate power of `x`:
   - Find `k` such that `x^k` is close to `v`
   - Either overshoot with `x^k` and subtract the difference: `x^k - (x^k - v)`
   - Or undershoot with `x^(k-1)` and add the difference: `x^(k-1) + (v - x^(k-1))`

The [dynamic programming](/problems/dynamic_programming_intro) approach with [memoization](/problems/memoization_intro) naturally emerges because we're breaking down the problem of reaching `target` into subproblems of reaching smaller values. Each recursive call represents finding the optimal way to express a particular value, and we cache these results to avoid redundant calculations.

The algorithm essentially performs a search through different decompositions of the target, always choosing the path that minimizes the total number of operators used.

**Learn more about [Memoization](/problems/memoization_intro), [Math](/problems/math-basics) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution uses a recursive approach with [memoization](/problems/memoization_intro) to find the minimum number of operators needed to express the target value.

The main function `dfs(v)` computes the minimum operators needed to create value `v`. Here's how it works:

**Base Case: When `x >= v`**

When the value we want to create (`v`) is less than or equal to `x`, we have two strategies:
1. **Build using ones**: Create `v` by adding `1`s together. Since each `1` is created using `x/x` (1 operator), and we need `v` of them with `v-1` addition operators between them, the total is `2v - 1` operators.
   - Example: If `x = 5` and `v = 3`, we get `5/5 + 5/5 + 5/5` using 5 operators total.

2. **Subtract from x**: Express `v` as `x - (x - v)`. This uses 1 subtraction operator plus the operators needed to create `x - v`. The total is `2 * (x - v)` operators (since creating `x - v` using ones would need `2(x - v) - 1` operators, plus 1 for the subtraction).

The algorithm chooses the minimum of these two options: `min(v * 2 - 1, 2 * (x - v))`.

**Recursive Case: When `x < v`**

For larger values, we need to use powers of `x`:

1. **Find the appropriate power**: The algorithm finds the smallest `k` such that `x^k >= v`. This is done through a simple loop:
   ```python
   k = 2
   while x**k < v:
       k += 1
   ```

2. **Decision point**: Once we have `x^k`, we can either:
   - **Overshoot and subtract**: Use `x^k - (x^k - v)` to get `v`. This requires `k` operators to build `x^k` (since `x^k` needs `k-1` multiplications), plus the operators to build `x^k - v`.
   - **Undershoot and add**: Use `x^(k-1) + (v - x^(k-1))` to get `v`. This requires `k-1` operators for `x^(k-1)`, plus the operators to build `v - x^(k-1)`.

3. **Optimization**: The code includes a smart optimization. If `x^k - v < v`, meaning the overshoot difference is smaller than `v` itself, it considers both options. Otherwise, it only uses the undershoot approach since the overshoot would require building a larger value recursively.

**[Memoization](/problems/memoization_intro) with @cache**

The `@cache` decorator stores previously computed results for each value of `v`. This prevents redundant calculations when the same subproblem appears multiple times during recursion, significantly improving performance.

**Final Result**

The function returns `dfs(target)`, which gives the minimum number of operators needed to express the target value using only the number `x`.

The elegance of this solution lies in its recursive decomposition - it transforms the problem of expressing any number into smaller subproblems, using the fact that any target can be efficiently expressed as a combination of powers of `x` with appropriate additions and subtractions.

## Example Walkthrough

Let's walk through finding the minimum operators needed when `x = 3` and `target = 10`.

**Initial Call: `dfs(10)`**

Since `x = 3 < 10`, we enter the recursive case and need to find an appropriate power of 3:
- `3^2 = 9` (less than 10)
- `3^3 = 27` (greater than 10)

So `k = 3`, meaning `3^3 = 27` is our overshoot power.

**Exploring Two Options:**

1. **Overshoot approach**: Express 10 as `27 - 17`
   - We need `k - 1 = 2` operators to build `3^3` (i.e., `3 * 3 * 3`)
   - We need 1 subtraction operator
   - We need `dfs(17)` operators to build 17
 
2. **Undershoot approach**: Express 10 as `9 + 1`
   - We need `k - 2 = 1` operator to build `3^2` (i.e., `3 * 3`)
   - We need 1 addition operator
   - We need `dfs(1)` operators to build 1

**Computing `dfs(1)`:**

Since `x = 3 >= 1`, we use the base case:
- Option 1: Build 1 using ones: `3/3` requires 1 operator
- Option 2: Subtract from 3: `3 - 2` requires `2 * (3 - 1) = 4` operators

Minimum is 1 operator, so `dfs(1) = 1`.

**Computing `dfs(17)`:**

Since `3 < 17`, we find powers:
- `3^2 = 9` (less than 17)
- `3^3 = 27` (greater than 17)

So `k = 3`.

Checking overshoot: `27 - 17 = 10`. Since `10 < 17`, we consider both options:
- Overshoot: `27 - 10` needs `2 + dfs(10)` operators
- Undershoot: `9 + 8` needs `1 + dfs(8)` operators

For `dfs(8)`:
- Since `3 < 8`, we find `3^2 = 9 > 8`, so `k = 2`
- Overshoot: `9 - 1` needs `1 + dfs(1) = 1 + 1 = 2` operators
- Since we only need the minimum and `9 - 8 = 1 < 8`, this gives us `dfs(8) = 2`

Back to `dfs(17)`:
- Overshoot would create a cycle (needs `dfs(10)` which we're computing)
- Undershoot: `1 + dfs(8) = 1 + 2 = 3` operators

So `dfs(17) = 3`.

**Final Calculation for `dfs(10)`:**

1. Overshoot: `2 + dfs(17) = 2 + 3 = 5` operators
2. Undershoot: `1 + dfs(1) = 1 + 1 = 2` operators

The minimum is 2 operators.

**Verifying the Result:**

The expression `3 * 3 + 3 / 3` equals `9 + 1 = 10` and uses exactly 2 operators (one multiplication and one division, with addition connecting them). This confirms our answer of 2 operators.

## Python Solution

```python
class Solution:
    def leastOpsExpressTarget(self, x: int, target: int) -> int:
        """
        Find the minimum number of operations needed to build target using x.
        Operations allowed: +, -, *, / (where / means x/x = 1)
      
        Args:
            x: The base number to use in expressions
            target: The target value to build
          
        Returns:
            Minimum number of operators needed (excluding the first term)
        """
        from functools import cache
      
        @cache
        def dfs(current_value: int) -> int:
            """
            Recursively find minimum operations to build current_value.
          
            Args:
                current_value: The value we need to construct
              
            Returns:
                Minimum number of operations needed
            """
            # Base case: when x is greater than or equal to current_value
            if x >= current_value:
                # Option 1: Use addition with x/x (1) repeated current_value times
                # Each 1 needs one division operator, plus (current_value - 1) additions
                option_add_ones = current_value * 2 - 1
              
                # Option 2: Use x minus some number of 1s
                # One subtraction for x, then (x - current_value) divisions for 1s,
                # plus (x - current_value - 1) subtractions between 1s
                option_subtract_from_x = 2 * (x - current_value)
              
                return min(option_add_ones, option_subtract_from_x)
          
            # Find the smallest power k where x^k >= current_value
            power = 2
            while x ** power < current_value:
                power += 1
          
            # Check if it's better to overshoot and subtract or undershoot and add
            if x ** power - current_value < current_value:
                # If overshooting is closer, we have two options:
                # Option 1: Use x^power and subtract the difference
                option_overshoot = power + dfs(x ** power - current_value)
              
                # Option 2: Use x^(power-1) and add the difference
                option_undershoot = power - 1 + dfs(current_value - x ** (power - 1))
              
                return min(option_overshoot, option_undershoot)
            else:
                # If undershooting is closer or equal, use x^(power-1) and add
                return power - 1 + dfs(current_value - x ** (power - 1))
      
        return dfs(target)
```

## Time and Space Complexity

**Time Complexity:** `O(log(target) * log(target))` or `O(log²(target))`

The recursion depth is bounded by `O(log(target))` because at each recursive call, we're working with values that are differences between powers of `x` and the current value `v`. The value `v` gets reduced significantly in each recursive step - either by subtracting a power of `x` close to it, or by working with `x^k - v` where `x^k` is the smallest power greater than `v`.

At each recursion level, the while loop to find the appropriate power `k` takes `O(log(v))` time in the worst case, where `v ≤ target`. Since we're using memoization with `@cache`, each unique value of `v` is computed only once.

The number of unique subproblems is bounded by `O(target)` in the worst case, but due to the nature of the problem where we work with powers and differences, the actual number of unique states visited is much smaller, approximately `O(log(target))`.

Therefore, the overall time complexity is `O(log(target) * log(target))` = `O(log²(target))`.

**Space Complexity:** `O(log(target))`

The space complexity consists of two components:
1. The recursion stack depth: `O(log(target))` - as the recursion depth is logarithmic in the target value
2. The memoization cache: `O(log(target))` - storing results for the unique subproblems encountered

The dominant factor is `O(log(target))` for both the recursion stack and the cache storage.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Integer Overflow with Large Powers**

When computing `x ** power` for large values of `x` and `power`, the result can quickly exceed Python's practical memory limits or cause performance issues. For instance, if `x = 999` and we're looking for a large target, computing `999^10` would result in an astronomically large number.

**Solution:**
Add an early termination condition to prevent unnecessary large power calculations:

```python
@cache
def dfs(current_value: int) -> int:
    if x >= current_value:
        return min(current_value * 2 - 1, 2 * (x - current_value))
  
    power = 2
    power_value = x * x  # Start with x^2
  
    # Add upper bound check to prevent overflow
    while power_value < current_value and power < 40:  # 40 is a reasonable upper bound
        power += 1
        power_value *= x
        if power_value > 10**9:  # Safeguard against extremely large values
            break
  
    # Rest of the logic remains the same
```

### 2. **Incorrect Base Case Calculation**

A common mistake is misunderstanding how operators are counted in the base case. When building a value using ones (x/x), developers often forget that:
- The first `x/x` uses 1 operator (division)
- Each additional `x/x` needs 1 division operator PLUS 1 addition operator to connect it

**Incorrect implementation:**
```python
# Wrong: Forgetting the addition operators between terms
option_add_ones = current_value  # This only counts divisions!
```

**Correct implementation:**
```python
# Right: Count both divisions and additions
option_add_ones = current_value * 2 - 1  # v divisions + (v-1) additions
```

### 3. **Off-by-One Error in Power Calculation**

The recursive case needs `k-1` operators to build `x^k` (not `k` operators), since `x` itself requires no operators. This is a subtle but critical distinction.

**Incorrect:**
```python
# Wrong: Counting x as needing 1 operator
option_overshoot = power + 1 + dfs(x ** power - current_value)
```

**Correct:**
```python
# Right: x^k needs k-1 multiplications
option_overshoot = power - 1 + 1 + dfs(x ** power - current_value)
# Simplified to:
option_overshoot = power + dfs(x ** power - current_value)
```

### 4. **Missing Edge Case: target = x**

When `target` equals `x`, the answer should be 0 (no operators needed), but the base case logic might not handle this correctly if not carefully implemented.

**Solution:**
Add an explicit check at the beginning:

```python
def leastOpsExpressTarget(self, x: int, target: int) -> int:
    if target == x:
        return 0
  
    # Rest of the implementation...
```

### 5. **Inefficient Repeated Power Calculations**

Computing `x ** power` multiple times in the same recursive call wastes computational resources.

**Inefficient:**
```python
if x ** power - current_value < current_value:
    option_overshoot = power + dfs(x ** power - current_value)  # Recomputes x^power
    option_undershoot = power - 1 + dfs(current_value - x ** (power - 1))  # Recomputes x^(power-1)
```

**Efficient:**
```python
power_value = x ** power
prev_power_value = x ** (power - 1)

if power_value - current_value < current_value:
    option_overshoot = power + dfs(power_value - current_value)
    option_undershoot = power - 1 + dfs(current_value - prev_power_value)
```

