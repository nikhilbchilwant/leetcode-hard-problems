# 2836. Maximize Value of Function in a Ball Passing Game

> Source: https://algo.monster/liteproblems/2836

## Problem Statement

You have `n` players numbered from `0` to `n-1` playing a ball-passing game. Each player `i` has a predetermined receiver stored in `receiver[i]`, which indicates the player they will pass the ball to.

The game works as follows:
- You choose any starting player `i`
- Player `i` passes the ball to player `receiver[i]`
- Player `receiver[i]` then passes to player `receiver[receiver[i]]`
- This continues for exactly `k` passes total

The score of a game is calculated as the sum of all player indices who touched the ball during these `k` passes, including the starting player. If a player touches the ball multiple times, their index is counted each time.

For example, if the ball goes through players `i → receiver[i] → receiver[receiver[i]] → ...` for `k` passes, the score would be `i + receiver[i] + receiver[receiver[i]] + ...` (summing `k+1` terms total, since we include the starting player).

Important notes:
- The `receiver` array may contain duplicates (multiple players can pass to the same player)
- A player can pass to themselves (`receiver[i]` can equal `i`)

Your task is to find the **maximum possible score** by choosing the optimal starting player.

## Intuition

The naive approach would be to simulate the ball passing for each starting player, following the path for `k` passes and calculating the sum. However, with `k` potentially being very large (up to `10^10` based on typical constraints), this would result in `O(n * k)` time complexity, which is too slow.

The key observation is that we're essentially following a path in a directed graph where each node has exactly one outgoing edge. Starting from any player `i`, we need to find the sum of indices along a path of length `k`.

When dealing with problems that require traversing a large number of steps along a fixed path, binary lifting is a powerful technique. The idea is to precompute jumps of powers of 2: instead of moving one step at a time, we can jump `2^0, 2^1, 2^2, ...` steps.

Why does this help? Any number `k` can be represented as a sum of powers of 2 (its binary representation). For example, if `k = 13 = 1101₂ = 8 + 4 + 1`, we can decompose the `k` passes into jumps of 8 passes, 4 passes, and 1 pass.

For each player `i` and jump size `2^j`, we need to track two things:
1. **Where do we end up** after `2^j` passes starting from player `i` - this helps us chain jumps together
2. **What's the sum of indices** for those `2^j` passes (excluding the final position to avoid double counting when chaining)

By building these tables using [dynamic programming](/problems/dynamic_programming_intro) (where `2^j` passes = `2^(j-1)` passes followed by another `2^(j-1)` passes), we can then reconstruct any path of length `k` by combining the appropriate power-of-2 jumps based on `k`'s binary representation. This reduces our time complexity from `O(k)` per starting player to `O(log k)` per starting player.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

We implement the solution using [dynamic programming](/problems/dynamic_programming_intro) with binary lifting. Let's break down the implementation step by step:

**1. Initialize the DP Tables**

We create two 2D arrays:
- `f[i][j]`: The player we reach after making `2^j` passes starting from player `i`
- `g[i][j]`: The sum of player indices visited during `2^j` passes starting from player `i` (excluding the final player to avoid double counting)

The dimensions are `n × m` where `n` is the number of players and `m = k.bit_length()` (the number of bits needed to represent `k`).

**2. Base Case (j = 0)**

For single passes (`2^0 = 1` pass):
- `f[i][0] = receiver[i]`: After 1 pass from player `i`, we reach `receiver[i]`
- `g[i][0] = i`: The sum includes only the starting player `i` (not the receiver)

**3. Build the DP Tables**

For `j > 0`, we compute `2^j` passes by combining two `2^(j-1)` passes:
- `f[i][j] = f[f[i][j-1]][j-1]`: First jump `2^(j-1)` steps to reach `f[i][j-1]`, then jump another `2^(j-1)` steps from there
- `g[i][j] = g[i][j-1] + g[f[i][j-1]][j-1]`: The sum is the sum from the first half plus the sum from the second half

**4. Calculate Maximum Score**

For each potential starting player `i`:
- Initialize position `p = i` and sum `t = 0`
- Decompose `k` into its binary representation
- For each bit `j` that is set in `k` (i.e., `k >> j & 1` is true):
  - Add `g[p][j]` to the running sum `t`
  - Update position to `p = f[p][j]`
- After processing all bits, add the final position `p` to get the total score
- Track the maximum score across all starting players

**Example:**

If `k = 5 = 101₂`, we need to make jumps of `2^0 = 1` and `2^2 = 4` passes. Starting from player `i`:
- Jump 4 passes: accumulate `g[i][2]`, move to `f[i][2]`
- Jump 1 pass: accumulate `g[current_position][0]`, move to `f[current_position][0]`
- Add the final position to get the complete sum

The time complexity is `O(n × log k)` for building the tables and finding the maximum, and space complexity is `O(n × log k)` for storing the DP tables.

## Example Walkthrough

Let's walk through a concrete example to illustrate the binary lifting solution.

**Given:**
- `n = 4` players (indexed 0, 1, 2, 3)
- `receiver = [1, 2, 3, 0]` (player 0 passes to 1, player 1 passes to 2, etc.)
- `k = 5` passes

**Step 1: Build the DP tables**

First, let's understand what paths look like from each player:
- From player 0: 0 → 1 → 2 → 3 → 0 → 1 → ...
- From player 1: 1 → 2 → 3 → 0 → 1 → 2 → ...
- From player 2: 2 → 3 → 0 → 1 → 2 → 3 → ...
- From player 3: 3 → 0 → 1 → 2 → 3 → 0 → ...

Since `k = 5 = 101₂`, we need tables for `j = 0, 1, 2` (powers of 2 up to 4).

**Base case (j = 0, representing 2^0 = 1 pass):**
- `f[0][0] = 1`, `g[0][0] = 0` (from 0, one pass reaches 1, sum = 0)
- `f[1][0] = 2`, `g[1][0] = 1` (from 1, one pass reaches 2, sum = 1)
- `f[2][0] = 3`, `g[2][0] = 2` (from 2, one pass reaches 3, sum = 2)
- `f[3][0] = 0`, `g[3][0] = 3` (from 3, one pass reaches 0, sum = 3)

**For j = 1 (representing 2^1 = 2 passes):**
- `f[0][1] = f[f[0][0]][0] = f[1][0] = 2` (0 → 1 → 2)
- `g[0][1] = g[0][0] + g[1][0] = 0 + 1 = 1` (sum of indices 0, 1)
- `f[1][1] = f[f[1][0]][0] = f[2][0] = 3` (1 → 2 → 3)
- `g[1][1] = g[1][0] + g[2][0] = 1 + 2 = 3` (sum of indices 1, 2)
- Similarly: `f[2][1] = 0`, `g[2][1] = 5` and `f[3][1] = 1`, `g[3][1] = 3`

**For j = 2 (representing 2^2 = 4 passes):**
- `f[0][2] = f[f[0][1]][1] = f[2][1] = 0` (0 → 1 → 2 → 3 → 0)
- `g[0][2] = g[0][1] + g[2][1] = 1 + 5 = 6` (sum of indices 0, 1, 2, 3)
- Similarly for other players...

**Step 2: Calculate score for each starting player**

Let's trace through starting from player 0 with k = 5 = 101₂:

1. Initialize: `position = 0`, `sum = 0`
2. Process bit 0 (2^0 = 1 pass):
   - Add `g[0][0] = 0` to sum → `sum = 0`
   - Move to `f[0][0] = 1` → `position = 1`
3. Skip bit 1 (not set in 101₂)
4. Process bit 2 (2^2 = 4 passes):
   - Add `g[1][2] = 7` to sum → `sum = 7`
   - Move to `f[1][2] = 1` → `position = 1`
5. Add final position: `sum = 7 + 1 = 8`

The path taken: 0 → 1 → 2 → 3 → 0 → 1
Score = 0 + 1 + 2 + 3 + 0 + 1 = 7... Wait, let me recalculate.

Actually, with 5 passes starting from player 0:
- Pass 1: 0 → 1 (ball at 1)
- Pass 2: 1 → 2 (ball at 2)
- Pass 3: 2 → 3 (ball at 3)
- Pass 4: 3 → 0 (ball at 0)
- Pass 5: 0 → 1 (ball at 1)

Players who touched the ball: 0, 1, 2, 3, 0, 1
Score = 0 + 1 + 2 + 3 + 0 + 1 = 7

**Step 3: Compare all starting positions**

- Starting from 0: Score = 7
- Starting from 1: Score = 8 (path: 1 → 2 → 3 → 0 → 1 → 2, sum = 1+2+3+0+1+2 = 9)
- Starting from 2: Score = 10 (path: 2 → 3 → 0 → 1 → 2 → 3, sum = 2+3+0+1+2+3 = 11)  
- Starting from 3: Score = 9 (path: 3 → 0 → 1 → 2 → 3 → 0, sum = 3+0+1+2+3+0 = 9)

The maximum score is 11, achieved by starting from player 2.

## Python Solution

```python
class Solution:
    def getMaxFunctionValue(self, receiver: List[int], k: int) -> int:
        n = len(receiver)
        max_power = k.bit_length()  # Number of bits needed to represent k
      
        # Binary lifting tables:
        # next_node[i][j] = destination after 2^j jumps starting from node i
        # sum_values[i][j] = sum of node values when making 2^j jumps from node i
        next_node = [[0] * max_power for _ in range(n)]
        sum_values = [[0] * max_power for _ in range(n)]
      
        # Initialize base case: 2^0 = 1 jump
        for i, next_receiver in enumerate(receiver):
            next_node[i][0] = next_receiver  # After 1 jump from i, we reach receiver[i]
            sum_values[i][0] = i  # Sum for 1 jump starting from i is just i itself
      
        # Build binary lifting tables for powers of 2
        for power in range(1, max_power):
            for node in range(n):
                # To jump 2^power times from node:
                # First jump 2^(power-1) times to reach intermediate_node
                intermediate_node = next_node[node][power - 1]
                # Then jump another 2^(power-1) times from intermediate_node
                next_node[node][power] = next_node[intermediate_node][power - 1]
              
                # Sum for 2^power jumps = sum of first half + sum of second half
                sum_values[node][power] = (sum_values[node][power - 1] + 
                                          sum_values[intermediate_node][power - 1])
      
        # Find the maximum function value across all starting nodes
        max_result = 0
      
        for start_node in range(n):
            current_node = start_node
            total_sum = 0
          
            # Decompose k into sum of powers of 2 and make corresponding jumps
            for bit_position in range(max_power):
                if (k >> bit_position) & 1:  # Check if bit at position is set
                    # Add sum for 2^bit_position jumps from current_node
                    total_sum += sum_values[current_node][bit_position]
                    # Move to the destination after 2^bit_position jumps
                    current_node = next_node[current_node][bit_position]
          
            # Function value = sum of all intermediate nodes + final destination node
            function_value = total_sum + current_node
            max_result = max(max_result, function_value)
      
        return max_result
```

## Time and Space Complexity

**Time Complexity:** `O(n * log k)`

The algorithm uses binary lifting technique where:
- Building the jump tables requires two nested loops: the outer loop runs `m = k.bit_length()` times (which is `O(log k)`), and the inner loop runs `n` times for each iteration. This gives us `O(n * log k)` for preprocessing.
- Computing the answer requires iterating through all `n` starting positions, and for each position, we iterate through at most `m = O(log k)` bits of `k` to perform the jumps. This also gives us `O(n * log k)`.
- Overall time complexity is `O(n * log k) + O(n * log k) = O(n * log k)`.

**Space Complexity:** `O(n * log k)`

The algorithm maintains two 2D arrays:
- Array `f` of size `n × m` where `m = k.bit_length() = O(log k)`, storing the destination after `2^j` jumps from each position.
- Array `g` of size `n × m`, storing the sum of values collected after `2^j` jumps from each position.
- Both arrays require `O(n * log k)` space.
- Additional variables use `O(1)` space.
- Overall space complexity is `O(n * log k)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Off-by-One Error in Counting Nodes**

**The Pitfall:** A critical misunderstanding is how many nodes are involved when making `k` passes. If we make `k` passes, we actually touch `k+1` nodes total (the starting node plus `k` nodes we pass through). Many implementations incorrectly assume only `k` nodes are involved.

**Why This Happens:** The problem states "k passes" but the score includes the starting player. When player `i` makes a pass to `receiver[i]`, that's 1 pass but 2 players touched the ball.

**The Fix:** The provided solution handles this correctly by:
- Storing partial sums in `sum_values[i][j]` that exclude the final destination 
- Adding the final node (`current_node`) separately after all jumps: `function_value = total_sum + current_node`

### 2. **Integer Overflow for Large k Values**

**The Pitfall:** When `k` is very large (up to 10^10 as per typical constraints), and we're summing player indices multiple times, the total sum can exceed standard integer limits.

**Example:** If `k = 10^10` and players keep passing in a small cycle like `0 → 1 → 0`, the sum could be approximately `5 × 10^9`, which exceeds 32-bit integer limits.

**The Fix:** 
- In Python, integers automatically handle arbitrary precision, so this isn't an issue
- In languages like Java/C++, use 64-bit integers (`long` in Java, `long long` in C++)

### 3. **Incorrect Binary Decomposition Logic**

**The Pitfall:** When decomposing `k` into powers of 2, developers might accidentally process bits in the wrong order or forget to update the current position correctly.

**Wrong approach example:**
```python
# INCORRECT: Forgetting to update position
for bit in range(max_power):
    if (k >> bit) & 1:
        total_sum += sum_values[start_node][bit]  # Always using start_node!
```

**The Fix:** Always update the current position after each jump:
```python
for bit_position in range(max_power):
    if (k >> bit_position) & 1:
        total_sum += sum_values[current_node][bit_position]
        current_node = next_node[current_node][bit_position]  # Update position!
```

### 4. **Misunderstanding the Sum Storage in Binary Lifting**

**The Pitfall:** Storing the wrong values in `sum_values[i][j]`. Some might think it should include the destination node, leading to double-counting.

**Wrong interpretation:** `sum_values[i][j]` = sum of all `2^j + 1` nodes (including both start and end)

**Correct interpretation:** `sum_values[i][j]` = sum of the first `2^j` nodes (excluding the final destination)

**Why This Matters:** When combining two jumps of size `2^(j-1)`, the intermediate node would be counted twice if we included destinations in our sums.

### 5. **Edge Case: k = 0**

**The Pitfall:** The problem might allow `k = 0` (no passes), which means only the starting player touches the ball.

**The Fix:** The current solution handles this correctly because:
- When `k = 0`, no bits are set, so the loop doesn't execute
- `function_value = total_sum + current_node` = `0 + start_node`, which is correct

However, it's worth adding an explicit check for clarity:
```python
if k == 0:
    return max(range(n))  # Maximum player index
```

