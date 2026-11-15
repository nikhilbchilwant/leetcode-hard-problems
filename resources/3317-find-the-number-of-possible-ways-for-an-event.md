# 3317. Find the Number of Possible Ways for an Event

> Source: https://algo.monster/liteproblems/3317

## Problem Statement

You have `n` performers who need to be assigned to stages for an event. There are `x` stages available, and each performer must be assigned to exactly one stage. When performers are assigned to the same stage, they form a band and will perform together. Note that some stages can remain empty (no performers assigned).

After all performances are done, the jury gives each band (not each stage, but each band that has at least one performer) a score between `1` and `y` inclusive.

You need to count the total number of different ways this event can happen. Two events are considered different if:
- Any performer is assigned to a different stage than in another arrangement, OR
- Any band receives a different score than in another arrangement

For example, if you have 2 performers and 2 stages:
- Both performers could go to stage 1 (forming 1 band)
- Both performers could go to stage 2 (forming 1 band)  
- Performer 1 to stage 1, performer 2 to stage 2 (forming 2 bands)
- Performer 1 to stage 2, performer 2 to stage 1 (forming 2 bands)

Each of these arrangements would then have different scoring possibilities based on how many bands were formed and the value of `y`.

The answer should be returned modulo `10^9 + 7` since it can be very large.

## Intuition

The key insight is to break this problem into two independent parts: arranging performers into bands, and then assigning scores to those bands.

First, let's think about how performers get distributed. We need to track not just which stage each performer goes to, but how many stages actually end up with performers (become bands). This is important because only non-empty stages receive scores.

Consider building the arrangement incrementally. When we place performer `i`, we have two choices:
1. Add them to an existing band (a stage that already has performers)
2. Start a new band (place them on an empty stage)

This leads us to think about [dynamic programming](/problems/dynamic_programming_intro) with states `f[i][j]` representing the number of ways to arrange `i` performers into exactly `j` non-empty stages.

For the transition, when we have already arranged `i-1` performers into `j-1` bands and want to add the `i`-th performer:
- To keep `j-1` bands: we can't do this (we need to form `j` bands)
- To form `j` bands from `j-1` bands: the new performer must go to a new stage. We have `x - (j-1)` empty stages available
- To keep `j` bands when we already have `j` bands: the new performer joins one of the existing `j` bands

This gives us the recurrence: `f[i][j] = f[i-1][j] * j + f[i-1][j-1] * (x - (j-1))`

Once we know how many ways to form `j` bands with all `n` performers, scoring becomes straightforward. Each band independently gets a score from `1` to `y`, so for `j` bands, there are `y^j` scoring possibilities.

The final answer combines both parts: for each possible number of bands `j` from `1` to `x`, we multiply the number of ways to form `j` bands (`f[n][j]`) by the number of ways to score them (`y^j`), and sum everything up.

**Learn more about [Math](/problems/math-basics), [Dynamic Programming](/problems/dynamic_programming_intro) and [Combinatorics](/problems/backtracking) patterns.**

## Solution Approach

We implement the solution using a 2D [dynamic programming](/problems/dynamic_programming_intro) table `f[i][j]` where `i` represents the number of performers and `j` represents the number of non-empty stages (bands).

**Initialization:**
- Create a 2D array `f` of size `(n+1) × (x+1)` initialized with zeros
- Set `f[0][0] = 1` as the base case (0 performers in 0 bands is one valid way)

**Building the DP table:**
For each performer `i` from `1` to `n` and each possible number of bands `j` from `1` to `x`:

```python
f[i][j] = (f[i-1][j] * j + f[i-1][j-1] * (x - (j-1))) % mod
```

This formula captures two scenarios:
- `f[i-1][j] * j`: The `i`-th performer joins one of the existing `j` bands. There are `j` choices.
- `f[i-1][j-1] * (x - (j-1))`: The `i`-th performer starts a new band. We had `j-1` bands using `j-1` stages, so there are `x - (j-1)` empty stages available.

**Calculating the final answer:**
After filling the DP table, we calculate the total number of ways by considering all possible numbers of bands:

```python
ans = 0
p = 1
for j in range(1, x + 1):
    p = p * y % mod  # p = y^j
    ans = (ans + f[n][j] * p) % mod
```

For each valid number of bands `j`:
- `f[n][j]` gives the number of ways to arrange all `n` performers into exactly `j` bands
- Each of these `j` bands can receive a score from `1` to `y`, giving `y^j` scoring combinations
- We accumulate `p = y^j` iteratively to avoid repeated exponentiation
- The product `f[n][j] * y^j` is added to our answer

**Modulo Operation:**
Throughout the computation, we take modulo `10^9 + 7` to prevent integer overflow and return the final result within the required range.

The time complexity is `O(n × x)` for filling the DP table and `O(x)` for calculating the final answer, giving an overall complexity of `O(n × x)`. The space complexity is `O(n × x)` for the DP table.

## Example Walkthrough

Let's walk through a small example with `n = 3` performers, `x = 3` stages, and `y = 2` possible scores.

**Step 1: Initialize DP table**
We create a table `f[i][j]` where `f[i][j]` = ways to arrange `i` performers into exactly `j` non-empty stages.
- Base case: `f[0][0] = 1` (0 performers in 0 bands is one valid way)

**Step 2: Fill the DP table**

For performer 1 (`i = 1`):
- `f[1][1] = f[0][0] * (3-0) = 1 * 3 = 3`
  - Performer 1 can go to any of the 3 empty stages

For performer 2 (`i = 2`):
- `f[2][1] = f[1][1] * 1 = 3 * 1 = 3`
  - Both performers in the same band (3 ways to choose which stage)
- `f[2][2] = f[1][1] * (3-1) = 3 * 2 = 6`
  - Performers in different bands (first performer picks 1 of 3 stages, second picks 1 of 2 remaining)

For performer 3 (`i = 3`):
- `f[3][1] = f[2][1] * 1 = 3 * 1 = 3`
  - All 3 performers in one band
- `f[3][2] = f[2][1] * (3-1) + f[2][2] * 2 = 3 * 2 + 6 * 2 = 6 + 12 = 18`
  - Either: start with all in 1 band, then split one off (3 * 2 ways)
  - Or: start with 2 bands, add third to either band (6 * 2 ways)
- `f[3][3] = f[2][2] * (3-2) = 6 * 1 = 6`
  - Each performer in their own band

**Step 3: Calculate final answer**
For each possible number of bands `j`, multiply by scoring possibilities `y^j`:

- 1 band: `f[3][1] * 2^1 = 3 * 2 = 6` ways
  - (3 ways to form 1 band) × (2 possible scores)
- 2 bands: `f[3][2] * 2^2 = 18 * 4 = 72` ways
  - (18 ways to form 2 bands) × (4 ways to score: each band gets 1 or 2)
- 3 bands: `f[3][3] * 2^3 = 6 * 8 = 48` ways
  - (6 ways to form 3 bands) × (8 ways to score: each of 3 bands gets 1 or 2)

**Total: 6 + 72 + 48 = 126 ways**

This counts all possible combinations of:
1. Assigning the 3 performers to stages (forming 1, 2, or 3 bands)
2. Assigning scores (1 or 2) to each formed band

## Python Solution

```python
class Solution:
    def numberOfWays(self, n: int, x: int, y: int) -> int:
        MOD = 10**9 + 7
      
        # dp[i][j] represents the number of ways to arrange i items 
        # using exactly j distinct values from x available values
        dp = [[0] * (x + 1) for _ in range(n + 1)]
        dp[0][0] = 1  # Base case: 0 items with 0 distinct values has 1 way
      
        # Fill the DP table
        for items in range(1, n + 1):
            for distinct_values in range(1, x + 1):
                # Two choices:
                # 1. Use an existing distinct value (multiply by distinct_values)
                # 2. Use a new distinct value (multiply by remaining unused values)
                use_existing = dp[items - 1][distinct_values] * distinct_values
                use_new = dp[items - 1][distinct_values - 1] * (x - (distinct_values - 1))
                dp[items][distinct_values] = (use_existing + use_new) % MOD
      
        # Calculate the final answer by considering y choices for each group
        answer = 0
        y_power = 1  # Tracks y^j
      
        for groups in range(1, x + 1):
            y_power = (y_power * y) % MOD  # y^groups
            # Add contribution of having 'groups' distinct values, 
            # each mapped to one of y possible outcomes
            answer = (answer + dp[n][groups] * y_power) % MOD
      
        return answer
```

## Time and Space Complexity

The time complexity is `O(n × x)`, where `n` represents the number of performers and `x` represents the number of programs. This is because the code uses a nested loop structure where the outer loop iterates from 1 to `n` (n iterations) and the inner loop iterates from 1 to `x` (x iterations). Each operation inside the nested loops takes constant time `O(1)`, resulting in a total time complexity of `O(n × x)`.

The space complexity is `O(n × x)`. The code creates a 2D array `f` with dimensions `(n + 1) × (x + 1)` to store the dynamic programming states. This array dominates the space usage, requiring `O(n × x)` space. The additional variables (`ans`, `p`, `mod`) use only constant space `O(1)`, which doesn't affect the overall space complexity.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Power Calculation for Scoring**

**Pitfall:** A common mistake is incorrectly computing `y^j` for each number of bands. Developers might:
- Use `y**j` directly without modulo, causing integer overflow for large values
- Recalculate `y^j` from scratch in each iteration using `pow(y, j, MOD)`, which is inefficient
- Initialize `y_power` incorrectly or update it in the wrong place

**Incorrect Implementation:**
```python
# Wrong: Overflow risk
answer = (answer + dp[n][groups] * (y ** groups)) % MOD

# Wrong: Inefficient repeated calculation
answer = (answer + dp[n][groups] * pow(y, groups, MOD)) % MOD

# Wrong: Incorrect initialization/update
y_power = y  # Should start at 1
for groups in range(1, x + 1):
    answer = (answer + dp[n][groups] * y_power) % MOD
    y_power = (y_power * y) % MOD  # Update after use instead of before
```

**Correct Solution:**
```python
y_power = 1  # Start with y^0 = 1
for groups in range(1, x + 1):
    y_power = (y_power * y) % MOD  # Update to y^groups before use
    answer = (answer + dp[n][groups] * y_power) % MOD
```

### 2. **Confusion Between Stages and Bands**

**Pitfall:** Mixing up the concept of stages (physical locations) and bands (groups of performers). The DP state `dp[i][j]` represents `j` non-empty stages (bands), not total stages used.

**Incorrect Thinking:**
- Treating `j` as the stage number instead of the count of non-empty stages
- Forgetting that multiple stages can remain empty
- Incorrectly calculating available stages for a new band

**Example of Wrong Logic:**
```python
# Wrong: Thinking j represents specific stage assignments
use_new = dp[items - 1][distinct_values - 1] * distinct_values  # Incorrect!
```

**Correct Understanding:**
- `j` represents the number of bands (non-empty stages)
- When adding a new band, we have `x - (j-1)` empty stages to choose from
- The formula `(x - (j-1))` correctly counts remaining empty stages

### 3. **Off-by-One Errors in DP Transitions**

**Pitfall:** Incorrectly handling indices when transitioning between DP states, especially when dealing with the "add to existing band" vs "create new band" logic.

**Common Mistakes:**
```python
# Wrong: Using wrong indices
use_new = dp[items - 1][distinct_values] * (x - distinct_values)  # Should be distinct_values - 1

# Wrong: Incorrect range for loops
for distinct_values in range(0, x + 1):  # Should start from 1, not 0
```

**Correct Implementation:**
```python
for distinct_values in range(1, x + 1):
    use_existing = dp[items - 1][distinct_values] * distinct_values
    use_new = dp[items - 1][distinct_values - 1] * (x - (distinct_values - 1))
```

### 4. **Missing or Incorrect Modulo Operations**

**Pitfall:** Forgetting to apply modulo at intermediate steps, leading to integer overflow even before the final result.

**Wrong:**
```python
# Missing modulo in intermediate calculations
dp[items][distinct_values] = use_existing + use_new  # Can overflow!
answer = answer + dp[n][groups] * y_power  # Can overflow!
```

**Correct:**
```python
dp[items][distinct_values] = (use_existing + use_new) % MOD
answer = (answer + dp[n][groups] * y_power) % MOD
```

### 5. **Incorrect Base Case Initialization**

**Pitfall:** Setting wrong initial values for the DP table, particularly for `dp[0][0]`.

**Wrong:**
```python
dp = [[1] * (x + 1) for _ in range(n + 1)]  # All initialized to 1
# or
dp[0][1] = 1  # Wrong base case
```

**Correct:**
```python
dp = [[0] * (x + 1) for _ in range(n + 1)]
dp[0][0] = 1  # Only this should be 1: 0 performers in 0 bands
```

These pitfalls can lead to incorrect results or runtime errors. Always verify the logic by working through small examples by hand and ensure all modulo operations are properly applied.

