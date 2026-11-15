# 517. Super Washing Machines

> Source: https://algo.monster/liteproblems/517

## Problem Statement

You have `n` super washing machines arranged in a line. Each washing machine initially contains some number of dresses or is empty.

In each move, you can select any `m` washing machines (where `1 <= m <= n`) and simultaneously pass one dress from each selected machine to one of its adjacent machines. This means:
- If a machine is selected, it must pass exactly one dress to either its left or right neighbor
- Multiple machines can be selected and operate in the same move
- All selected machines transfer dresses simultaneously in a single move

Given an integer array `machines` where `machines[i]` represents the number of dresses in the `i`-th washing machine (from left to right), your task is to find the minimum number of moves required to make all washing machines have the same number of dresses.

If it's impossible to distribute the dresses evenly among all machines, return `-1`.

For example, if you have machines with `[1, 0, 5]` dresses:
- The total is 6 dresses, which can be evenly distributed as 2 dresses per machine
- One possible sequence: Machine at index 2 passes dresses to its neighbor(s) over multiple moves
- The goal is to reach `[2, 2, 2]` in the minimum number of moves

## Intuition

First, let's check if a solution exists. If the total number of dresses cannot be divided evenly among all machines, it's impossible to balance them, so we return `-1`. Otherwise, each machine should eventually have `k = total_dresses / n` dresses.

The key insight is to think about the "flow" of dresses through the line of machines. For each machine `i`, we can calculate how many dresses it has in excess or deficit: `a[i] = machines[i] - k`. A positive value means it needs to give away dresses, while a negative value means it needs to receive dresses.

Now, imagine drawing a boundary between any two adjacent machines. All excess dresses on the left side of this boundary must eventually flow to the right side (if the left has surplus), or vice versa. The number of dresses that must cross this boundary is the absolute value of the cumulative sum up to that point: `|sum(a[0] to a[i])|`. Since only one dress can cross this boundary per move, this gives us a lower bound on the number of moves needed.

For example, if machines have `[4, 0, 0]` and target is `k = 1.33...` (impossible), but say `[3, 0, 0]` with `k = 1`, then:
- `a = [2, -1, -1]`
- After position 0: cumulative sum = `2` (2 dresses must flow right)
- After position 1: cumulative sum = `1` (1 dress must flow right)

There's another constraint: a single machine with too many excess dresses needs to distribute them to both neighbors. If machine `i` has `a[i]` extra dresses (where `a[i] > 0`), it needs at least `a[i]` moves to get rid of all its excess, even if both neighbors are taking dresses simultaneously.

The answer is the maximum of:
1. The maximum flow across any boundary: `max(|cumulative_sum|)` for all positions
2. The maximum excess at any single machine: `max(a[i])` for all `a[i] > 0`

This [greedy](/problems/greedy_intro) approach works because these two values represent independent bottlenecks that cannot be reduced - we need at least this many moves no matter how we schedule the transfers.

**Learn more about [Greedy](/problems/greedy_intro) patterns.**

## Solution Approach

Let's walk through the implementation step by step:

1. **Check if solution is possible**: First, we calculate the total number of dresses and check if it can be evenly distributed:
   ```python
   n = len(machines)
   k, mod = divmod(sum(machines), n)
   if mod:
       return -1
   ```
   If there's a remainder (`mod != 0`), even distribution is impossible, so we return `-1`. Otherwise, `k` represents the target number of dresses each machine should have.

2. **Initialize tracking variables**: We need two variables:
   - `ans`: tracks the maximum moves required (our answer)
   - `s`: tracks the cumulative sum of differences (represents flow across boundaries)
   ```python
   ans = s = 0
   ```

3. **Process each machine**: For each machine, we calculate three important values:
   ```python
   for x in machines:
       x -= k           # Calculate difference from target
       s += x           # Update cumulative sum
       ans = max(ans, abs(s), x)
   ```
 
   Here's what each calculation represents:
   - `x -= k`: This transforms the actual dress count to the difference from target. If `x > 0`, the machine has excess; if `x < 0`, it has deficit.
   - `s += x`: The cumulative sum `s` represents the net flow of dresses that must pass through the boundary after this machine. `abs(s)` tells us how many moves are needed for this flow.
   - `ans = max(ans, abs(s), x)`: We update our answer with the maximum of:
     - Current answer
     - `abs(s)`: The flow bottleneck at this boundary
     - `x`: The distribution bottleneck if this machine has excess (only positive `x` matters as a bottleneck)

4. **Return the result**: The final `ans` contains the minimum number of moves required.

The algorithm efficiently computes both bottlenecks in a single pass through the array with `O(n)` time complexity and `O(1)` extra space. The key insight is recognizing that the answer is determined by the maximum of either the flow bottleneck between groups or the distribution bottleneck at individual machines with excess dresses.

## Example Walkthrough

Let's walk through the solution with `machines = [0, 3, 0]`:

**Step 1: Check if solution is possible**
- Total dresses = 0 + 3 + 0 = 3
- Number of machines n = 3
- Target per machine k = 3 ÷ 3 = 1 (no remainder, so solution exists)

**Step 2: Calculate differences from target**
- Machine 0: 0 - 1 = -1 (needs 1 dress)
- Machine 1: 3 - 1 = +2 (has 2 extra dresses)
- Machine 2: 0 - 1 = -1 (needs 1 dress)

**Step 3: Process each machine and track bottlenecks**

Initialize: `ans = 0`, `s = 0` (cumulative sum)

- **Machine 0** (has 0 dresses):
  - Difference x = -1
  - Cumulative sum s = 0 + (-1) = -1
  - Update ans = max(0, |-1|, -1) = max(0, 1, -1) = 1
  - Interpretation: 1 dress must flow from right to left across this boundary

- **Machine 1** (has 3 dresses):
  - Difference x = +2
  - Cumulative sum s = -1 + 2 = 1
  - Update ans = max(1, |1|, 2) = max(1, 1, 2) = 2
  - Interpretation: Machine 1 has 2 excess dresses to distribute, creating a bottleneck

- **Machine 2** (has 0 dresses):
  - Difference x = -1
  - Cumulative sum s = 1 + (-1) = 0
  - Update ans = max(2, |0|, -1) = max(2, 0, -1) = 2
  - Interpretation: No net flow needed past this point (s = 0 confirms balance)

**Result: Minimum moves = 2**

This makes sense because Machine 1 needs to give away 2 dresses, and a selected machine can pass only one dress per move. Giving one dress to each side therefore takes two moves:
- Move 1: Machine 1 passes 1 dress left to Machine 0 → [1, 2, 0]
- Move 2: Machine 1 passes 1 dress right to Machine 2 → [1, 1, 1]

The algorithm correctly identifies that Machine 1's excess of 2 dresses is the bottleneck requiring 2 moves.

## Python Solution

```python
class Solution:
    def findMinMoves(self, machines: List[int]) -> int:
        # Get the number of machines
        n = len(machines)
      
        # Calculate the target number of dresses per machine
        total_dresses = sum(machines)
        target_per_machine, remainder = divmod(total_dresses, n)
      
        # If dresses cannot be evenly distributed, return -1
        if remainder:
            return -1
      
        # Track the maximum moves needed
        max_moves = 0
        # Track the cumulative excess/deficit as we traverse machines
        cumulative_balance = 0
      
        # Process each machine
        for current_dresses in machines:
            # Calculate excess/deficit for this machine
            excess = current_dresses - target_per_machine
          
            # Update cumulative balance (net flow through boundaries)
            cumulative_balance += excess
          
            # The answer is the maximum of:
            # 1. abs(cumulative_balance): max flow through any boundary
            # 2. excess: max dresses a single machine needs to give away
            max_moves = max(max_moves, abs(cumulative_balance), excess)
      
        return max_moves
```

## Time and Space Complexity

The time complexity is `O(n)`, where `n` is the number of washing machines. The algorithm performs a single pass through the `machines` array with one initial `sum()` operation that takes `O(n)` time, followed by a for loop that iterates through all `n` elements exactly once. Each operation inside the loop (subtraction, addition, and max comparison) takes constant time `O(1)`.

The space complexity is `O(1)`. The algorithm only uses a fixed number of variables (`n`, `k`, `mod`, `ans`, `s`, and `x`) regardless of the input size. No additional data structures that scale with the input are created.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Misunderstanding Why We Don't Use `abs(excess)` in the Max Comparison

**The Mistake:**
Many people intuitively think we should use `abs(excess)` instead of just `excess` when updating the maximum:
```python
# Incorrect approach
max_moves = max(max_moves, abs(cumulative_balance), abs(excess))
```

**Why This is Wrong:**
Machines with deficit (negative excess) can receive from multiple neighbors simultaneously, so they don't create a bottleneck. Only machines with excess dresses create a bottleneck because they must give away dresses one at a time.

**Example:**
- `machines = [3, 0, 3]`, target = 2
- The differences are `[1, -2, 1]` and the cumulative balances are `[1, -1, 0]`
- The middle machine has a deficit of 2, but both neighbors can pass it one dress in the same move, so the answer is 1
- Using `abs(excess)` returns 2, because it treats the deficit of 2 as a bottleneck
- Using `excess` returns `max(1, 1, -2) = 1`, which is correct

**Correct Approach:**
```python
# Only positive excess values matter as bottlenecks
max_moves = max(max_moves, abs(cumulative_balance), excess)
```

### Pitfall 2: Confusing Cumulative Balance with Individual Machine Transfers

**The Mistake:**
Thinking that `cumulative_balance` represents how many dresses a specific machine needs to transfer, rather than the net flow through a boundary.

**Why This Matters:**
`cumulative_balance` actually represents the total net flow of dresses that must pass through the boundary between processed and unprocessed machines. This is different from individual machine operations.

**Example:**
- `machines = [1, 0, 5]`, target = 2
- After processing index 0: cumulative_balance = -1 (the left part is short by 1, so 1 dress must flow left across the boundary between index 0 and 1)
- After processing index 1: cumulative_balance = -3 (3 dresses must flow left through the boundary between index 1 and 2)
- The abs(cumulative_balance) = 3 represents the bottleneck flow, not any single machine's operation

**Correct Understanding:**
The cumulative balance tracks the net flow requirement across boundaries, which determines one type of bottleneck in the system.

### Pitfall 3: Forgetting Edge Cases in Initial Validation

**The Mistake:**
Not properly handling or testing edge cases like:
- Single machine: `machines = [5]`
- All machines already balanced: `machines = [2, 2, 2]`
- Total sum is zero: `machines = [0, 0, 0]`

**Solution:**
Always validate these cases:
```python
# Handle single machine case (already balanced)
if n == 1:
    return 0

# Check if already balanced (optional optimization)
if all(d == target_per_machine for d in machines):
    return 0
```

Though the main algorithm handles these cases correctly, explicitly checking can improve clarity and potentially save computation.

