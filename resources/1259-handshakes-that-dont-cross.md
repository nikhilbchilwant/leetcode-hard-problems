# 1259. Handshakes That Don't Cross

> Source: https://algo.monster/liteproblems/1259

## Problem Statement

You have an even number of people standing in a circle, represented by `numPeople`. Each person needs to shake hands with exactly one other person, resulting in `numPeople / 2` total handshakes.

The key constraint is that when people shake hands, their arms cannot cross. Imagine drawing lines between pairs of people who shake hands - these lines must not intersect with each other.

Your task is to find the total number of valid ways to arrange these handshakes such that no two handshakes cross each other. Since the answer can be very large, return the result modulo `10^9 + 7`.

For example, if you have 4 people in a circle:
- One valid arrangement: person 1 shakes hands with person 2, and person 3 shakes hands with person 4
- Another valid arrangement: person 1 shakes hands with person 4, and person 2 shakes hands with person 3 (their arms would go around the outside and inside respectively, not crossing)
- Invalid arrangement: person 1 shakes hands with person 3, and person 2 shakes hands with person 4 (these would cross)

The problem essentially asks you to count all possible non-crossing perfect matchings in a circle of people.

## Intuition

When we think about arranging non-crossing handshakes in a circle, let's focus on one specific person - say person 1. This person must shake hands with someone, and this choice has a profound impact on the problem structure.

If person 1 shakes hands with person `k`, this handshake divides the circle into two separate groups:
- People between person 1 and person `k` (on one side)
- People between person `k` and person 1 (on the other side)

The crucial insight is that because handshakes cannot cross, people in one group can only shake hands with others in the same group. If someone from the left group tried to shake hands with someone from the right group, their handshake would have to cross the handshake between person 1 and person `k`.

This observation leads us to a recursive structure. For person 1 to shake hands with person `k`, there must be an even number of people on each side (since everyone needs a partner). If there are `l` people on the left side and `r` people on the right side, then:
- The `l` people on the left can arrange their handshakes in `dfs(l)` ways
- The `r` people on the right can arrange their handshakes in `dfs(r)` ways
- These arrangements are independent, so we multiply them: `dfs(l) * dfs(r)`

To find the total number of arrangements for `i` people, we sum up all possible choices of who person 1 shakes hands with. Person 1 can only shake hands with person 2, 4, 6, ... (to ensure even numbers of people on both sides), which is why we iterate `l` from 0 to `i-1` in steps of 2.

This recursive pattern is actually computing the Catalan numbers, a famous sequence that appears in many counting problems involving non-crossing structures.

**Learn more about [Math](/problems/math-basics) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution implements a [dynamic programming](/problems/dynamic_programming_intro) approach using memoization to avoid redundant calculations.

**Function Design:**
We define a recursive function `dfs(i)` that calculates the number of valid handshake arrangements for `i` people. The base case is when `i < 2`, where there's only one way (no handshakes), so we return `1`.

**Recursive Logic:**
For `i` people, we iterate through all possible positions where person 1 can shake hands. Since person 1 needs to shake hands with someone at an even distance to ensure both sides have even numbers of people, we use:
```python
for l in range(0, i, 2):
    r = i - l - 2
```

Here:
- `l` represents the number of people on the left side
- `r = i - l - 2` represents the number of people on the right side (subtracting 2 for person 1 and their handshake partner)
- We iterate `l` from 0 to `i-1` in steps of 2 to maintain even counts

**Calculation:**
For each split, the total arrangements equals the product of arrangements on both sides:
```python
ans += dfs(l) * dfs(r)
```

We accumulate all possible splits and apply modulo `10^9 + 7` to handle large numbers:
```python
ans %= mod
```

**Memoization:**
The `@cache` decorator automatically memoizes the function results. This is crucial because the same subproblems appear multiple times in different recursive branches. For example, `dfs(4)` might be calculated when solving `dfs(6)`, `dfs(8)`, etc. Without memoization, the time complexity would be exponential.

**Time Complexity:** `O(n²)` where n is `numPeople`, as we compute each state once and each state requires `O(n)` work.

**Space Complexity:** `O(n)` for the recursion stack and memoization cache.

## Example Walkthrough

Let's walk through the solution with `numPeople = 6` (6 people in a circle).

**Step 1: Initial Call**
We call `dfs(6)` to find arrangements for 6 people.

**Step 2: Person 1's Handshake Options**
Person 1 can shake hands with persons 2, 4, or 6 (only even positions to maintain even groups).

**Option A: Person 1 shakes hands with Person 2**
- Left side: 0 people (between 1 and 2)
- Right side: 4 people (persons 3, 4, 5, 6)
- Contribution: `dfs(0) * dfs(4) = 1 * dfs(4)`

To calculate `dfs(4)`:
- Person 3 shakes with Person 4: `dfs(0) * dfs(2) = 1 * 1 = 1`
- Person 3 shakes with Person 6: `dfs(2) * dfs(0) = 1 * 1 = 1`
- Total: `dfs(4) = 2`

So Option A contributes: `1 * 2 = 2`

**Option B: Person 1 shakes hands with Person 4**
- Left side: 2 people (persons 2, 3)
- Right side: 2 people (persons 5, 6)
- Contribution: `dfs(2) * dfs(2) = 1 * 1 = 1`

**Option C: Person 1 shakes hands with Person 6**
- Left side: 4 people (persons 2, 3, 4, 5)
- Right side: 0 people
- Contribution: `dfs(4) * dfs(0) = 2 * 1 = 2`

**Step 3: Final Result**
Total arrangements: `2 + 1 + 2 = 5`

The 5 valid arrangements are:
1. (1-2), (3-4), (5-6)
2. (1-2), (3-6), (4-5)
3. (1-4), (2-3), (5-6)
4. (1-6), (2-3), (4-5)
5. (1-6), (2-5), (3-4)

Each arrangement ensures no handshakes cross when drawn in the circle.

## Python Solution

```python
class Solution:
    def numberOfWays(self, numPeople: int) -> int:
        """
        Calculate the number of ways people can shake hands without crossing.

        Args:
            numPeople: Even number of people standing in a circle

        Returns:
            Number of valid handshake arrangements modulo 10^9 + 7
        """
        from functools import cache

        MOD = 10**9 + 7

        @cache
        def count_ways(num_people: int) -> int:
            """
            Dynamic programming function to count valid handshake arrangements.

            Args:
                num_people: Number of people to arrange handshakes for

            Returns:
                Number of valid arrangements for num_people
            """
            # Base case: 0 or 2 people have exactly 1 way to shake hands
            if num_people < 2:
                return 1

            total_ways = 0

            # Try pairing person 0 with every other person at odd positions
            # This ensures we always have even number of people on each side
            for left_count in range(0, num_people, 2):
                # Calculate number of people on the right side after pairing
                right_count = num_people - left_count - 2

                # Multiply ways to arrange left side with ways to arrange right side
                # Use Catalan number property: C(n) = sum(C(i) * C(n-1-i))
                total_ways += count_ways(left_count) * count_ways(right_count)
                total_ways %= MOD

            return total_ways

        return count_ways(numPeople)
```

## Time and Space Complexity

The time complexity is `O(n^2)` where `n` is the value of `numPeople`.

The function uses memoization with `@cache` decorator. For each value `i` from 0 to `numPeople`, we compute `dfs(i)` exactly once due to caching. When computing `dfs(i)`, we iterate through all possible left subtree sizes `l` from 0 to `i-2` in steps of 2, which gives us `i/2` iterations. For each iteration, we perform constant time operations (multiplication and modulo).

The total number of operations across all cached calls is:
- `dfs(0)`: 0 iterations
- `dfs(2)`: 1 iteration
- `dfs(4)`: 2 iterations
- ...
- `dfs(n)`: `n/2` iterations

This sums to `0 + 1 + 2 + ... + n/2 = O(n^2/4) = O(n^2)`.

The space complexity is `O(n)` where `n` is the value of `numPeople`.

The space is used for:
1. The memoization cache which stores results for values from 0 to `numPeople`, requiring `O(n)` space
2. The recursion call stack which can go up to depth `O(n)` in the worst case

Therefore, the total space complexity is `O(n) + O(n) = O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Partitioning Logic**
A common mistake is incorrectly calculating how to partition the circle when person 1 shakes hands with another person. Many developers mistakenly think they can pair person 1 with any other person, but this can lead to odd numbers of people on one or both sides.

**Wrong approach:**
```python
for partner in range(2, num_people + 1):
    left = partner - 2
    right = num_people - partner
    # This can result in odd numbers on either side
```

**Correct approach:**
```python
for left_count in range(0, num_people, 2):
    right_count = num_people - left_count - 2
    # Ensures both sides have even numbers
```

### 2. **Forgetting to Apply Modulo Operation**
When dealing with large numbers, forgetting to apply the modulo operation at each step can cause integer overflow in languages with fixed integer sizes, or incorrect results due to precision issues.

**Wrong approach:**
```python
total_ways = 0
for left_count in range(0, num_people, 2):
    right_count = num_people - left_count - 2
    total_ways += count_ways(left_count) * count_ways(right_count)
return total_ways % MOD  # Only applying modulo at the end
```

**Correct approach:**
```python
total_ways = 0
for left_count in range(0, num_people, 2):
    right_count = num_people - left_count - 2
    total_ways += count_ways(left_count) * count_ways(right_count)
    total_ways %= MOD  # Apply modulo after each addition
```

### 3. **Not Using Memoization**
Without memoization, the recursive solution has exponential time complexity because the same subproblems are solved multiple times. This will cause time limit exceeded for larger inputs.

**Wrong approach:**
```python
def count_ways(num_people):
    if num_people < 2:
        return 1
    total_ways = 0
    for left_count in range(0, num_people, 2):
        right_count = num_people - left_count - 2
        total_ways += count_ways(left_count) * count_ways(right_count)
    return total_ways % MOD
```

**Correct approach:**
Use `@cache` decorator or implement manual memoization:
```python
@cache
def count_ways(num_people):
    # ... rest of the implementation
```

Or with manual memoization:
```python
memo = {}
def count_ways(num_people):
    if num_people in memo:
        return memo[num_people]
    # ... calculate result
    memo[num_people] = result
    return result
```

### 4. **Incorrect Base Case**
Some might incorrectly handle the base case, especially for 2 people, thinking it needs special handling.

**Wrong approach:**
```python
if num_people == 0:
    return 0  # Wrong: 0 people means 1 valid arrangement (empty)
if num_people == 2:
    return 1  # This is actually correct but redundant
```

**Correct approach:**
```python
if num_people < 2:  # Handles both 0 and negative cases
    return 1
```

The beauty of this solution is that it naturally handles the case of 2 people through the recursive formula without special casing.

