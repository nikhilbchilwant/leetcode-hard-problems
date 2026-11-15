# 2318. Number of Distinct Roll Sequences

> Source: https://algo.monster/liteproblems/2318

## Problem Statement

You need to simulate rolling a 6-sided dice `n` times and count how many different valid sequences of rolls are possible.

A sequence of rolls is valid if it satisfies two conditions:

1. **Adjacent rolls must be coprime**: Any two consecutive dice values in the sequence must have a greatest common divisor (GCD) of 1. For example, if you roll a 2 followed by a 3, this is valid since `gcd(2,3) = 1`. But rolling a 2 followed by a 4 is invalid since `gcd(2,4) = 2`.

2. **Same values must be separated**: If the same number appears multiple times in the sequence, there must be at least 2 other rolls between them. In other words, if position `i` has the same value as position `j`, then `|i - j| > 2`. For example, the sequence `[1, 2, 3, 1]` is valid (the two 1's are separated by 2 positions), but `[1, 2, 1]` is invalid (the two 1's are only separated by 1 position).

The problem asks you to count all distinct valid sequences of length `n` where each roll can be any value from 1 to 6. Since this count can be very large, return the result modulo `10^9 + 7`.

For example, if `n = 1`, any single roll from 1 to 6 is valid, so the answer is 6. For larger values of `n`, you need to consider both constraints when building valid sequences.

## Intuition

When thinking about this problem, we need to track not just the current roll, but also the previous roll to ensure adjacent values are coprime, and potentially the roll before that to ensure we don't repeat a value too soon.

The key insight is that we need to know the last two rolls to determine if the next roll is valid. If we're at position `k`:
- We need to know the roll at position `k-1` to check if the new roll is coprime with it
- We need to know the roll at position `k-2` to ensure we're not repeating that value (violating the separation constraint)

This naturally leads to a [dynamic programming](/problems/dynamic_programming_intro) approach where our state is defined by:
- The current position in the sequence (from 1 to `n`)
- The value of the previous roll (position `k-1`)
- The value of the roll before that (position `k-2`)

We can build up our answer using `dp[k][i][j]` which represents the number of valid sequences of length `k` where the last roll is `i+1` and the second-to-last roll is `j+1` (using 0-based indexing for the array).

Starting with sequences of length 2, we can initialize all valid pairs where two adjacent values are coprime and different. Then for each subsequent position, we try all possible previous values that would make a valid sequence:
- The new value must be coprime with the immediately previous value
- The new value must be different from both the previous value and the value two positions back

By building up from length 2 to length `n`, we accumulate all possible valid sequences. The final answer is the sum of all `dp[n][i][j]` values, representing all possible ways to end an `n`-length sequence with any valid pair of final values.

**Learn more about [Memoization](/problems/memoization_intro) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The implementation uses a 3D [dynamic programming](/problems/dynamic_programming_intro) array to track valid sequences:

**Data Structure:**
- `dp[k][i][j]`: A 3D array where `dp[k][i][j]` represents the number of valid sequences of length `k` ending with dice value `i+1` at position `k` and dice value `j+1` at position `k-1`.

**Algorithm Steps:**

1. **Handle Base Case**: If `n = 1`, return 6 since any single dice roll (1-6) forms a valid sequence.

2. **Initialize DP Array**: Create a 3D array of dimensions `(n+1) × 6 × 6` initialized with zeros.

3. **Initialize Length-2 Sequences**: 
   ```python
   for i in range(6):
       for j in range(6):
           if gcd(i + 1, j + 1) == 1 and i != j:
               dp[2][i][j] = 1
   ```
   This sets `dp[2][i][j] = 1` for all valid 2-roll sequences where:
   - The two values are coprime (`gcd(i+1, j+1) == 1`)
   - The two values are different (`i != j`)

4. **Build Longer Sequences**: For each length `k` from 3 to `n`:
   ```python
   for k in range(3, n + 1):
       for i in range(6):  # Current last roll
           for j in range(6):  # Second-to-last roll
               if gcd(i + 1, j + 1) == 1 and i != j:
                   for h in range(6):  # Third-to-last roll
                       if gcd(h + 1, i + 1) == 1 and h != i and h != j:
                           dp[k][i][j] += dp[k - 1][h][i]
   ```
 
   For each valid ending pair `(i, j)`:
   - Check if they are coprime and different
   - Try all possible third-to-last values `h`
   - Validate that `h` is coprime with `i` (the new second-to-last)
   - Ensure `h ≠ i` (adjacent values must differ)
   - Ensure `h ≠ j` (values must be separated by at least 2 positions)
   - Add the count from the previous state `dp[k-1][h][i]`

5. **Calculate Final Answer**: Sum all valid sequences of length `n`:
   ```python
   ans = 0
   for i in range(6):
       for j in range(6):
           ans += dp[-1][i][j]
   return ans % mod
   ```

The solution uses modulo `10^9 + 7` to handle large numbers. The time complexity is `O(n × 6³) = O(216n)` and space complexity is `O(n × 36) = O(36n)`.

## Example Walkthrough

Let's walk through the solution with `n = 3` to find all valid 3-roll sequences.

**Step 1: Initialize for length 2**

First, we find all valid 2-roll sequences where values are coprime and different:
- (1,2): gcd(1,2)=1, different ✓ → dp[2][0][1] = 1
- (1,3): gcd(1,3)=1, different ✓ → dp[2][0][2] = 1  
- (1,5): gcd(1,5)=1, different ✓ → dp[2][0][4] = 1
- (2,1): gcd(2,1)=1, different ✓ → dp[2][1][0] = 1
- (2,3): gcd(2,3)=1, different ✓ → dp[2][1][2] = 1
- (2,5): gcd(2,5)=1, different ✓ → dp[2][1][4] = 1
- (3,1): gcd(3,1)=1, different ✓ → dp[2][2][0] = 1
- (3,2): gcd(3,2)=1, different ✓ → dp[2][2][1] = 1
- (3,4): gcd(3,4)=1, different ✓ → dp[2][2][3] = 1
- (3,5): gcd(3,5)=1, different ✓ → dp[2][2][4] = 1

(Continuing this process for all pairs gives us 17 valid 2-roll sequences)

**Step 2: Build length 3 sequences**

For each valid ending pair (i,j), we try adding a third value h at the beginning:

Example: Building sequences ending with (2,3):
- Current last two rolls: j=2 (value 3), i=1 (value 2)
- Need dp[3][1][2] (sequences ending with 2,3)
- Try each possible h (third-to-last roll):
  - h=0 (value 1): Check gcd(1,2)=1 ✓, 1≠2 ✓, 1≠3 ✓
    - Look up dp[2][0][1] = 1 (sequences "1,2")
    - Add to dp[3][1][2]: creates sequence [1,2,3]
  - h=2 (value 3): Check gcd(3,2)=1 ✓, 3≠2 ✓, but 3=3 ✗
    - Skip (violates separation rule)
  - h=3 (value 4): Check gcd(4,2)=2 ✗
    - Skip (not coprime)
  - h=4 (value 5): Check gcd(5,2)=1 ✓, 5≠2 ✓, 5≠3 ✓
    - Look up dp[2][4][1] = 1 (sequences "5,2")
    - Add to dp[3][1][2]: creates sequence [5,2,3]

So dp[3][1][2] = 2 (two valid sequences ending with 2,3)

**Step 3: Sum all valid length-3 sequences**

After processing all combinations:
- Valid sequences like [1,2,3], [1,2,5], [1,3,2], [1,3,4], [1,3,5], [1,5,2], [1,5,3], [1,5,4], [1,5,6]...
- Sum all dp[3][i][j] values where dp[3][i][j] > 0
- Total count = 48 valid sequences of length 3

The algorithm systematically builds longer sequences by extending shorter valid sequences, ensuring both the coprime constraint (through gcd checks) and the separation constraint (by tracking the last two values).

## Python Solution

```python
from math import gcd

class Solution:
    def distinctSequences(self, n: int) -> int:
        # Base case: if n is 1, we can roll any of the 6 dice values
        if n == 1:
            return 6
      
        MOD = 10**9 + 7
      
        # dp[length][last_die][second_last_die] = number of valid sequences
        # where last_die is the last rolled die (0-indexed: 0 represents die value 1)
        # and second_last_die is the second to last rolled die
        dp = [[[0] * 6 for _ in range(6)] for _ in range(n + 1)]
      
        # Initialize base case for sequences of length 2
        # Two consecutive dice must have gcd = 1 and be different
        for last_die in range(6):
            for second_last_die in range(6):
                # Convert 0-indexed to 1-indexed for gcd calculation
                if gcd(last_die + 1, second_last_die + 1) == 1 and last_die != second_last_die:
                    dp[2][last_die][second_last_die] = 1
      
        # Build up the dp table for sequences of length 3 to n
        for length in range(3, n + 1):
            for last_die in range(6):
                for second_last_die in range(6):
                    # Check if last two dice are valid (coprime and different)
                    if gcd(last_die + 1, second_last_die + 1) == 1 and last_die != second_last_die:
                        # Try all possible third-to-last dice
                        for third_last_die in range(6):
                            # Check constraints: 
                            # 1. third_last and second_last must be coprime
                            # 2. third_last must be different from second_last
                            # 3. third_last must be different from last (no same value at distance 2)
                            if (gcd(third_last_die + 1, second_last_die + 1) == 1 and 
                                third_last_die != second_last_die and 
                                third_last_die != last_die):
                                dp[length][last_die][second_last_die] += dp[length - 1][third_last_die][second_last_die]
      
        # Sum up all valid sequences of length n ending with any valid pair
        total_sequences = 0
        for last_die in range(6):
            for second_last_die in range(6):
                total_sequences += dp[n][last_die][second_last_die]
      
        return total_sequences % MOD
```

## Time and Space Complexity

**Time Complexity:** `O(n * 6^3)` = `O(216n)` = `O(n)`

The algorithm uses dynamic programming with three nested loops:
- The outermost loop iterates from 3 to n, contributing `O(n)` iterations
- The middle two loops iterate through all possible pairs of dice values (i, j), each from 0 to 5, contributing `O(6^2)` = `O(36)` iterations
- The innermost loop iterates through all possible previous dice values (h), contributing `O(6)` iterations
- Inside the loops, the GCD calculation and other operations are `O(1)`

Therefore, the total time complexity is `O(n * 6 * 6 * 6)` = `O(216n)` = `O(n)`.

**Space Complexity:** `O(n * 6^2)` = `O(36n)` = `O(n)`

The algorithm uses a 3D DP array:
- The first dimension has size `n + 1`
- The second dimension has size 6 (representing the previous dice value)
- The third dimension has size 6 (representing the current dice value)

Therefore, the total space complexity is `O((n + 1) * 6 * 6)` = `O(36n)` = `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. Incorrect Index Mapping in the Recurrence Relation

**The Pitfall:**
When building sequences of length `k`, the code uses `dp[k-1][third_last_die][second_last_die]` in the transition. This is incorrect because:
- In `dp[k-1]`, the last position is at index `k-1` and second-to-last is at `k-2`
- When extending to length `k`, the old "last" (position `k-1`) becomes the new "second-to-last"
- The old "second-to-last" (position `k-2`) becomes the new "third-to-last"

The current code incorrectly keeps `second_last_die` in the same position when it should shift to represent the third-to-last position.

**The Fix:**
```python
# Incorrect transition:
dp[length][last_die][second_last_die] += dp[length - 1][third_last_die][second_last_die]

# Correct transition:
dp[length][last_die][second_last_die] += dp[length - 1][second_last_die][third_last_die]
```

When extending from length `k-1` to `k`:
- The new last die is `last_die` (position `k`)
- The new second-to-last is `second_last_die` (position `k-1`)
- The old configuration had `second_last_die` as last (position `k-1`) and `third_last_die` as second-to-last (position `k-2`)

### 2. Constraint Validation Confusion

**The Pitfall:**
The constraint checking can be confusing because we're validating relationships between the wrong pairs. When extending the sequence, we need to ensure:
- The new die (`last_die`) is coprime with the previous die (`second_last_die`)
- The new die doesn't equal any die within distance 2

**The Fix:**
Structure the validation more clearly:
```python
for length in range(3, n + 1):
    for last_die in range(6):
        for second_last_die in range(6):
            # First validate the new pair being added
            if gcd(last_die + 1, second_last_die + 1) != 1 or last_die == second_last_die:
                continue
          
            # Then check all valid previous states
            for third_last_die in range(6):
                # Ensure no same value at distance 2
                if third_last_die == last_die:
                    continue
                # Add valid transitions from previous state
                dp[length][last_die][second_last_die] += dp[length - 1][second_last_die][third_last_die]
```

### 3. Missing Modulo Operations

**The Pitfall:**
The code only applies modulo at the final return, but intermediate values can overflow in languages with fixed integer sizes or cause performance issues with very large numbers.

**The Fix:**
Apply modulo during accumulation:
```python
dp[length][last_die][second_last_die] = (dp[length][last_die][second_last_die] + 
                                          dp[length - 1][second_last_die][third_last_die]) % MOD
```

### Complete Corrected Solution:
```python
from math import gcd

class Solution:
    def distinctSequences(self, n: int) -> int:
        if n == 1:
            return 6
      
        MOD = 10**9 + 7
      
        dp = [[[0] * 6 for _ in range(6)] for _ in range(n + 1)]
      
        # Initialize length 2 sequences
        for i in range(6):
            for j in range(6):
                if gcd(i + 1, j + 1) == 1 and i != j:
                    dp[2][i][j] = 1
      
        # Build longer sequences
        for length in range(3, n + 1):
            for last_die in range(6):
                for second_last_die in range(6):
                    if gcd(last_die + 1, second_last_die + 1) == 1 and last_die != second_last_die:
                        for third_last_die in range(6):
                            if third_last_die != last_die:
                                dp[length][last_die][second_last_die] = (
                                    dp[length][last_die][second_last_die] + 
                                    dp[length - 1][second_last_die][third_last_die]
                                ) % MOD
      
        total = 0
        for i in range(6):
            for j in range(6):
                total = (total + dp[n][i][j]) % MOD
      
        return total
```

