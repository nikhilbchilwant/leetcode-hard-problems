# 818. Race Car

> Source: https://algo.monster/liteproblems/818

## Problem Statement

You have a race car on an infinite number line starting at position `0` with initial speed `+1`. The car can move into negative positions and operates based on two commands:

**Command 'A' (Accelerate):**
- First, update position: `position += speed`
- Then, double the speed: `speed *= 2`

**Command 'R' (Reverse):**
- If current speed is positive: set `speed = -1`
- If current speed is negative: set `speed = 1`
- Position remains unchanged

For example, with the command sequence `"AAR"`:
- Start: position = `0`, speed = `1`
- After 'A': position = `0 + 1 = 1`, speed = `1 * 2 = 2`
- After 'A': position = `1 + 2 = 3`, speed = `2 * 2 = 4`
- After 'R': position = `3` (unchanged), speed = `-1` (reversed from positive)

Given a target position `target`, find the minimum number of commands needed to reach that exact position.

The solution uses [dynamic programming](/problems/dynamic_programming_intro) where `dp[i]` represents the minimum number of commands to reach position `i`. The algorithm considers different strategies:
- If position `i` equals `2^k - 1` for some `k`, we can reach it with exactly `k` accelerations
- Otherwise, we can either overshoot and reverse back, or approach from below with a combination of accelerations and reversals

## Intuition

The key observation is understanding the positions we can reach with consecutive accelerations. Starting from position `0` with speed `1`:
- After 1 'A': position = `1`, speed = `2`
- After 2 'A's: position = `3`, speed = `4`
- After 3 'A's: position = `7`, speed = `8`
- After k 'A's: position = `2^k - 1`, speed = `2^k`

Notice that consecutive accelerations give us positions `1, 3, 7, 15, 31...` which are all of the form `2^k - 1`. These are the "perfect" positions that can be reached most efficiently with exactly `k` accelerations.

For any other target position, we have two main strategies:

**Strategy 1: Overshoot and come back**
If our target is between `2^(k-1) - 1` and `2^k - 1`, we can:
- Accelerate `k` times to reach `2^k - 1` (overshooting the target)
- Reverse to change direction (1 command)
- Then solve the subproblem of reaching the remaining distance `(2^k - 1) - target` in the opposite direction

This gives us the formula: `dp[target] = k + 1 + dp[2^k - 1 - target]`

**Strategy 2: Approach from below**
Alternatively, we can:
- Accelerate `k-1` times to reach `2^(k-1) - 1` (stopping short of target)
- Reverse to change direction (1 command)
- Accelerate `j` times backward to position `2^(k-1) - 1 - (2^j - 1)`
- Reverse again to go forward (1 command)
- Solve the remaining distance to target

This gives us: `dp[target] = (k-1) + 1 + j + 1 + dp[target - (2^(k-1) - 2^j)]`

The optimal solution is the minimum among all these possible strategies. The bit length operation `k = target.bit_length()` helps us quickly find the smallest `k` such that `2^k > target`, which determines the range of strategies we need to consider.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution uses [dynamic programming](/problems/dynamic_programming_intro) with a bottom-up approach. We create an array `dp` where `dp[i]` stores the minimum number of commands needed to reach position `i`.

**Step 1: Initialize the DP array**
```python
dp = [0] * (target + 1)
```
We build solutions for all positions from `1` to `target`.

**Step 2: Process each position**
For each position `i` from `1` to `target`:

First, calculate `k = i.bit_length()`, which gives us the smallest integer such that `2^k > i`. This means `i` falls in the range `[2^(k-1), 2^k - 1]`.

**Step 3: Handle perfect positions**
```python
if i == 2**k - 1:
    dp[i] = k
    continue
```
If `i` equals `2^k - 1`, we can reach it with exactly `k` consecutive accelerations. This is optimal, so we set `dp[i] = k` and move to the next position.

**Step 4: Strategy 1 - Overshoot and return**
```python
dp[i] = dp[2**k - 1 - i] + k + 1
```
We accelerate `k` times to reach position `2^k - 1`, then reverse (adding 1 command), and solve the subproblem of traveling `(2^k - 1) - i` distance in the opposite direction. The total cost is `k` accelerations + `1` reverse + `dp[2^k - 1 - i]`.

**Step 5: Strategy 2 - Approach from below with backtracking**
```python
for j in range(k - 1):
    dp[i] = min(dp[i], dp[i - (2**(k - 1) - 2**j)] + k - 1 + j + 2)
```
We try all possible ways to approach from below:
- Accelerate `k-1` times to reach `2^(k-1) - 1`
- Reverse (1 command)
- Accelerate `j` times backward, moving `2^j - 1` units back
- Reverse again (1 command)
- Solve remaining distance: `i - (2^(k-1) - 1) + (2^j - 1) = i - (2^(k-1) - 2^j)`

The formula simplifies to:
- `(k-1)` forward accelerations
- `1` reverse
- `j` backward accelerations  
- `1` reverse
- `dp[i - (2^(k-1) - 2^j)]` for the remaining distance

Total: `(k-1) + 1 + j + 1 + dp[i - (2^(k-1) - 2^j)] = k - 1 + j + 2 + dp[i - (2^(k-1) - 2^j)]`

**Step 6: Return the result**
After computing all positions up to `target`, return `dp[target]` as the minimum number of commands needed.

## Example Walkthrough

Let's find the minimum number of commands to reach position `5`.

**Step 1: Build DP table from position 1 to 5**

**Position 1:**
- `k = 1.bit_length() = 1` (since `2^1 = 2 > 1`)
- Check if `1 == 2^1 - 1 = 1`: Yes! This is a perfect position
- `dp[1] = 1` (one 'A' command: pos 0→1)

**Position 2:**
- `k = 2.bit_length() = 2` (since `2^2 = 4 > 2`)
- Check if `2 == 2^2 - 1 = 3`: No
- Strategy 1 (overshoot): Go to 3, reverse, go back 1
  - `dp[2] = k + 1 + dp[3-2] = 2 + 1 + dp[1] = 2 + 1 + 1 = 4`
- Strategy 2 (approach from below): Go to 1, reverse, go back 0 positions, reverse, go forward 1
  - For `j = 0`: `dp[2] = (k-1) + 1 + j + 1 + dp[2-(1-0)] = 1 + 1 + 0 + 1 + dp[1] = 3 + 1 = 4`
- `dp[2] = min(4, 4) = 4` (for example "ARRA": go to 1, reverse twice, then accelerate to 2)

**Position 3:**
- `k = 3.bit_length() = 2` (since `2^2 = 4 > 3`)
- Check if `3 == 2^2 - 1 = 3`: Yes! This is a perfect position
- `dp[3] = 2` (two 'A' commands: pos 0→1→3)

**Position 4:**
- `k = 4.bit_length() = 3` (since `2^3 = 8 > 4`)
- Check if `4 == 2^3 - 1 = 7`: No
- Strategy 1 (overshoot to 7): `dp[4] = 3 + 1 + dp[7-4] = 3 + 1 + dp[3] = 3 + 1 + 2 = 6`
- Strategy 2 (from position 3):
  - For `j = 0`: Go to 3, reverse, don't go back, reverse, go 1 forward
    - `dp[4] = 2 + 1 + 0 + 1 + dp[4-3] = 2 + 1 + 0 + 1 + dp[1] = 4 + 1 = 5`
  - For `j = 1`: Go to 3, reverse, go back 1, reverse, go 2 forward
    - `dp[4] = 2 + 1 + 1 + 1 + dp[4-(3-1)] = 2 + 1 + 1 + 1 + dp[2] = 5 + 4 = 9`
- `dp[4] = min(6, 5, 9) = 5` (Commands: "AARRA" - go to 3, reverse twice, go to 4)

**Position 5:**
- `k = 5.bit_length() = 3` (since `2^3 = 8 > 5`)
- Check if `5 == 2^3 - 1 = 7`: No
- Strategy 1 (overshoot to 7): `dp[5] = 3 + 1 + dp[7-5] = 3 + 1 + dp[2] = 3 + 1 + 4 = 8`
- Strategy 2 (from position 3):
  - For `j = 0`: `dp[5] = 2 + 1 + 0 + 1 + dp[5-3] = 4 + dp[2] = 4 + 4 = 8`
  - For `j = 1`: `dp[5] = 2 + 1 + 1 + 1 + dp[5-(3-1)] = 5 + dp[3] = 5 + 2 = 7`
- `dp[5] = min(8, 8, 7) = 7`

**Answer: 7 commands minimum to reach position 5**

One optimal sequence: "AARARAA"
- Start: pos=0, speed=1
- A: pos=1, speed=2
- A: pos=3, speed=4
- R: pos=3, speed=-1
- A: pos=2, speed=-2
- R: pos=2, speed=1
- A: pos=3, speed=2
- A: pos=5, speed=4

## Python Solution

```python
class Solution:
    def racecar(self, target: int) -> int:
        # dp[i] stores the minimum number of instructions to reach position i
        dp = [0] * (target + 1)
      
        # Calculate minimum instructions for each position from 1 to target
        for position in range(1, target + 1):
            # Calculate the number of bits needed to represent the position
            # This represents the minimum accelerations needed if we go straight
            bits_needed = position.bit_length()
          
            # Check if position is exactly reachable by continuous acceleration
            # (i.e., position is of the form 2^k - 1)
            if position == 2**bits_needed - 1:
                # Can reach this position with exactly k accelerations
                dp[position] = bits_needed
                continue
          
            # Case 1: Overshoot the target and then reverse
            # Go to (2^k - 1), then reverse and cover the remaining distance
            overshoot_position = 2**bits_needed - 1
            remaining_distance = overshoot_position - position
            dp[position] = dp[remaining_distance] + bits_needed + 1
          
            # Case 2: Undershoot, reverse, go back some distance, then reverse again
            # Try different amounts of backward movement
            for backward_steps in range(bits_needed - 1):
                # Forward distance: 2^(k-1) - 1 (one bit less than full)
                # Backward distance: 2^j - 1 (where j is backward_steps)
                forward_distance = 2**(bits_needed - 1) - 1
                backward_distance = 2**backward_steps - 1
                net_distance = forward_distance - backward_distance
              
                # Calculate remaining distance to target
                remaining = position - net_distance
              
                # Total instructions: (k-1) forward + 1 reverse + j backward + 1 reverse + remaining
                instructions = (bits_needed - 1) + 1 + backward_steps + 1 + dp[remaining]
                dp[position] = min(dp[position], instructions)
      
        return dp[target]
```

## Time and Space Complexity

**Time Complexity:** `O(n * log n)`

The algorithm uses dynamic programming to compute the minimum number of instructions to reach each position from 1 to target.

- The outer loop iterates through all positions from 1 to `target`, which is `O(n)` where `n = target`.
- For each position `i`:
  - Computing `bit_length()` takes `O(log i)` time.
  - If `i = 2^k - 1`, the computation is `O(1)`.
  - Otherwise, there's an inner loop that runs from `j = 0` to `k - 2`, where `k ≈ log i`. This inner loop performs `O(log i)` iterations.
  - Each iteration of the inner loop performs constant time operations (array lookups and comparisons).
- Therefore, for each position `i`, the work done is `O(log i)`.
- The total time complexity is `∑(i=1 to n) O(log i) = O(n * log n)`.

**Space Complexity:** `O(n)`

The algorithm uses a single array `dp` of size `target + 1` to store the minimum number of instructions for each position from 0 to `target`. No other data structures scale with the input size, so the space complexity is `O(n)` where `n = target`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Considering Only the Overshoot Strategy**

**The Pitfall:**
It is tempting to assume that the fastest route always accelerates past the target and then comes back. Dropping the undershoot loop gives wrong answers.

```python
# Wrong: only the overshoot case
dp[position] = dp[2**bits_needed - 1 - position] + bits_needed + 1
```

For `target = 4`, overshooting costs `AAA` (to 7) + `R` + `dp[3] = 2`, which is 6 commands. The undershoot route `AARRA` (to 3, reverse twice, then 1 more step) needs only 5. The overshoot-only version also returns 8 instead of 6 for `target = 8`.

**Solution:**
Always try both strategies and keep the minimum, as the loop over `backward_steps` does.

### 2. **Skipping the Zero-Backward-Steps Case**

**The Pitfall:**
In the undershoot strategy, it can look pointless to reverse twice without moving (`backward_steps = 0`). Starting the loop at 1 removes that option:

```python
# Wrong: misses the "reverse twice in place" option
for backward_steps in range(1, bits_needed - 1):
```

Two reversals in a row reset the speed to `+1` without changing position, which is exactly how `AARRA` reaches 4. With the loop starting at 1, `target = 4` returns 6 instead of 5, and `target = 8` returns 8 instead of 6.

**Solution:**
Start the loop at `backward_steps = 0`: `for backward_steps in range(bits_needed - 1)`.

### 3. **Adding Unnecessary Bounds Checks or a Larger DP Array**

**The Pitfall:**
It is common to worry that `dp[remaining_distance]` or `dp[remaining]` could index past `target` or below 0, and to add bounds checks or allocate `dp` of size `2 * target`.

**Why it is not needed:**
Since `bits_needed = position.bit_length()`, the position satisfies `2^(k-1) <= position < 2^k - 1` in the non-perfect case. The overshoot remainder `2^k - 1 - position` is therefore at most `2^(k-1) - 1`, which is smaller than `position`. In the undershoot case, `remaining = position - (2^(k-1) - 2^j)` is at least `1` (because `2^(k-1) - 2^j < 2^(k-1) <= position`) and smaller than `position` (because `j < k-1`). Both indices always fall in `[0, position)`, so they are already computed when `position` is processed in increasing order.

**Solution:**
Keep `dp` of size `target + 1` and fill it in increasing order of position. No extra bounds checks are required.

