# 2927. Distribute Candies Among Children III

> Source: https://algo.monster/liteproblems/2927

## Problem Statement

You have `n` candies that need to be distributed among 3 children. Each child can receive between 0 and `limit` candies (inclusive).

The task is to find the total number of different ways to distribute all `n` candies among the 3 children such that no child receives more than `limit` candies.

For example, if you have 5 candies and each child can get at most 2 candies, you need to count all valid distributions like (2, 2, 1), (2, 1, 2), (1, 2, 2), etc., where each tuple represents the number of candies given to child 1, child 2, and child 3 respectively.

## Intuition

The key insight is to transform this problem into a classic "stars and bars" combinatorial problem. Imagine we have `n` identical candies (stars) that need to be separated into 3 groups (for 3 children) using 2 dividers (bars).

Without any restrictions, this is equivalent to arranging `n` stars and 2 bars in a line. The number of stars before the first bar goes to child 1, between the two bars goes to child 2, and after the second bar goes to child 3. Using the stars and bars formula, we get `C(n+2, 2)` total arrangements.

However, we have a constraint: no child can get more than `limit` candies. The initial formula counts invalid distributions where some children get too many candies. This is where the inclusion-exclusion principle comes in.

First, we subtract the "bad" cases where at least one child gets more than `limit` candies. If we force one child to get at least `limit + 1` candies, we're left with `n - (limit + 1)` candies to distribute freely among 3 children, giving us `C(n-limit+1, 2)` ways. Since any of the 3 children could be the one exceeding the limit, we multiply by 3.

But wait - by subtracting these cases, we've double-counted scenarios where two children both exceed the limit. These cases were subtracted twice (once for each child exceeding), but should only be subtracted once. So we need to add them back using `3 * C(n-2*limit, 2)` when such cases exist (when `n >= 2*limit + 2`).

This inclusion-exclusion pattern ensures we count each valid distribution exactly once by carefully accounting for overlapping constraint violations.

**Learn more about [Math](/problems/math-basics) and [Combinatorics](/problems/backtracking) patterns.**

## Solution Approach

The solution implements the inclusion-exclusion principle using combinatorial mathematics. Here's how the implementation works:

First, we check if it's impossible to distribute the candies. If `n > 3 * limit`, it means even if all three children get the maximum allowed candies, we still can't distribute all `n` candies, so we return 0.

Next, we calculate the total number of ways without any restrictions using the stars and bars formula:
- We need to place `n` candies into 3 boxes (children)
- This is equivalent to choosing 2 positions out of `n + 2` positions to place dividers
- The formula is `C(n+2, 2)` which represents all possible distributions

Then we apply the inclusion-exclusion principle to remove invalid cases:

**Step 1: Subtract cases where at least one child exceeds the limit**
- If `n > limit`, it's possible for a child to get more than `limit` candies
- For each child that must get at least `limit + 1` candies, we have `n - limit - 1` remaining candies to distribute
- Using stars and bars again: `C(n - limit + 1, 2)` ways for one specific child to exceed
- Since any of the 3 children could be the one exceeding, we subtract `3 * C(n - limit + 1, 2)`

**Step 2: Add back over-subtracted cases where two children exceed the limit**
- If `n - 2 >= 2 * limit` (equivalently `n >= 2 * limit + 2`), two children can both exceed the limit
- When two specific children each get at least `limit + 1` candies, we use `2 * (limit + 1) = 2 * limit + 2` candies
- Remaining candies: `n - 2 * limit - 2`, which gives `C(n - 2 * limit, 2)` distributions
- There are 3 ways to choose which 2 children exceed the limit, so we add back `3 * C(n - 2 * limit, 2)`

The final answer combines these calculations using the inclusion-exclusion formula:
`ans = C(n+2, 2) - 3 * C(n-limit+1, 2) + 3 * C(n-2*limit, 2)`

The `comb` function from Python's [math](/problems/math-basics) library efficiently computes the binomial coefficients needed for this solution.

## Example Walkthrough

Let's walk through a concrete example where we have **n = 5 candies** and each child can receive at most **limit = 2 candies**.

**Step 1: Calculate total distributions without restrictions**

Using the stars and bars formula C(n+2, 2):
- We need C(5+2, 2) = C(7, 2) = 21 total ways
- This counts ALL possible distributions like (5,0,0), (4,1,0), (3,2,0), etc.

**Step 2: Identify and subtract invalid distributions**

Since n = 5 > limit = 2, some distributions are invalid (a child gets more than 2 candies).

Let's enumerate what we're subtracting:
- Child 1 gets > 2: distributions like (3,2,0), (3,1,1), (4,1,0), (5,0,0)
- Child 2 gets > 2: distributions like (2,3,0), (1,3,1), (1,4,0), (0,5,0)
- Child 3 gets > 2: distributions like (2,0,3), (1,1,3), (1,0,4), (0,0,5)

For each child exceeding the limit:
- Force that child to get at least 3 candies (limit + 1)
- Distribute remaining 5 - 3 = 2 candies among all 3 children
- This gives C(2+2, 2) = C(4, 2) = 6 ways per child
- Total to subtract: 3 × 6 = 18

**Step 3: Add back over-subtracted cases**

Check if n ≥ 2×limit + 2: Is 5 ≥ 2×2 + 2 = 6? No!

Since 5 < 6, it's impossible for two children to both exceed the limit (they would need at least 3+3=6 candies total). So we add back 0.

**Final Calculation:**
- Total ways = 21 - 18 + 0 = 3

**Verification:**
The 3 valid distributions are:
1. (2, 2, 1) - Child 1 gets 2, Child 2 gets 2, Child 3 gets 1
2. (2, 1, 2) - Child 1 gets 2, Child 2 gets 1, Child 3 gets 2
3. (1, 2, 2) - Child 1 gets 1, Child 2 gets 2, Child 3 gets 2

Each distribution uses exactly 5 candies and no child gets more than 2 candies, confirming our answer of 3.

## Python Solution

```python
from math import comb

class Solution:
    def distributeCandies(self, n: int, limit: int) -> int:
        """
        Calculate the number of ways to distribute n candies among 3 children
        where each child can receive at most 'limit' candies.

        Uses inclusion-exclusion principle to count valid distributions.

        Args:
            n: Total number of candies to distribute
            limit: Maximum candies any single child can receive

        Returns:
            Number of valid distribution ways
        """
        # If total candies exceed maximum possible distribution (3 children * limit each)
        # then no valid distribution exists
        if n > 3 * limit:
            return 0

        # Start with total ways to distribute n candies among 3 children
        # without any limit (stars and bars formula: C(n+2, 2))
        total_ways = comb(n + 2, 2)

        # Subtract cases where at least one child gets more than limit candies
        # (Inclusion-Exclusion: subtract violations for each child)
        if n > limit:
            # For each of 3 children, subtract ways where that child gets > limit candies
            # If child i gets limit+1 or more, distribute remaining n-(limit+1) candies
            total_ways -= 3 * comb(n - limit + 1, 2)

        # Add back cases where at least two children exceed the limit
        # (Inclusion-Exclusion: add back over-subtracted cases)
        if n - 2 >= 2 * limit:
            # Cases where 2 children each get at least limit+1 candies
            # Remaining candies: n - 2*(limit+1) = n - 2*limit - 2
            total_ways += 3 * comb(n - 2 * limit, 2)

        # Note: Cases where all 3 children exceed limit are impossible
        # since that would require n > 3*limit, which we already handled

        return total_ways
```

## Time and Space Complexity

The time complexity is `O(1)` because the algorithm performs a fixed number of operations regardless of the input size. The function only executes:
- A constant number of comparisons (`if n > 3 * limit`, `if n > limit`, `if n - 2 >= 2 * limit`)
- A fixed number of `comb()` function calls (at most 3 calls)
- Basic arithmetic operations (additions, subtractions, multiplications)

Even though the `comb(n, k)` function might seem like it could affect complexity, in this specific implementation `k` is always 2, making each combination calculation `comb(n, 2) = n * (n - 1) / 2` which is computed in constant time through direct formula evaluation.

The space complexity is `O(1)` because the algorithm only uses a fixed amount of extra space:
- One variable `ans` to store the result
- Temporary variables for arithmetic calculations
- No recursive calls or dynamic data structures that grow with input size

The memory usage remains constant regardless of the values of `n` and `limit`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Boundary Conditions for Inclusion-Exclusion Terms**

A common mistake is incorrectly determining when to apply each term of the inclusion-exclusion principle. Specifically:

**Pitfall Example:**
```python
# INCORRECT: Using wrong conditions
if n >= limit:  # Wrong condition!
    total_ways -= 3 * comb(n - limit + 1, 2)

if n >= 2 * limit:  # Wrong condition!
    total_ways += 3 * comb(n - 2 * limit, 2)
```

**Why it's wrong:**
- For the first subtraction, we need `n > limit` (not `n >= limit`) because when `n = limit`, it's still valid for one child to receive all candies.
- For the second addition, using `n >= 2 * limit` would cause errors when `n = 2 * limit` or `n = 2 * limit + 1`, as the remaining candies would be negative or too few to distribute.

**Correct Solution:**
```python
if n > limit:
    total_ways -= 3 * comb(n - limit + 1, 2)

if n >= 2 * limit + 2:  # or equivalently: n - 2 >= 2 * limit
    total_ways += 3 * comb(n - 2 * limit, 2)
```

### 2. **Forgetting to Handle Edge Cases in Combinatorial Calculations**

**Pitfall Example:**
```python
# INCORRECT: Not checking if arguments to comb() are valid
total_ways -= 3 * comb(n - limit + 1, 2)  # What if n - limit + 1 < 2?
```

**Why it's wrong:** The `comb(n, k)` function returns 0 when `n < k`, but relying on this without explicit checks can make the code harder to understand and debug.

**Correct Solution:**
Always ensure the conditions properly guard against invalid combinatorial calculations:
```python
if n > limit:  # This ensures n - limit + 1 >= 2
    total_ways -= 3 * comb(n - limit + 1, 2)
```

### 3. **Misunderstanding the Stars and Bars Formula**

**Pitfall Example:**
```python
# INCORRECT: Using wrong formula for distributing n items among k groups
total_ways = comb(n + 3 - 1, 3 - 1)  # This would be C(n+2, 2) - correct
# But some might incorrectly write:
total_ways = comb(n, 3)  # WRONG!
```

**Why it's wrong:** The stars and bars formula for distributing `n` identical items into `k` groups is `C(n+k-1, k-1)`, not `C(n, k)`.

**Correct Solution:**
For 3 children (k=3), the formula is:
```python
total_ways = comb(n + 3 - 1, 3 - 1)  # = comb(n + 2, 2)
```

### 4. **Not Considering the Case Where Distribution is Impossible**

**Pitfall Example:**
```python
# INCORRECT: Forgetting the impossible case
def distributeCandies(self, n: int, limit: int) -> int:
    total_ways = comb(n + 2, 2)
    # ... rest of inclusion-exclusion
    return total_ways  # Could return negative number!
```

**Why it's wrong:** When `n > 3 * limit`, it's impossible to distribute all candies within the constraints, but the inclusion-exclusion formula might still produce a non-zero (possibly negative) result.

**Correct Solution:**
Always check for impossibility first:
```python
if n > 3 * limit:
    return 0
```

