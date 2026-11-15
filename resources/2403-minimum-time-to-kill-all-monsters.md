# 2403. Minimum Time to Kill All Monsters

> Source: https://algo.monster/liteproblems/2403

## Problem Statement

You have an array `power` where `power[i]` represents the power level of the `i-th` monster that you need to defeat.

You begin the game with `0` mana points. Each day, you gain mana points equal to a value called `gain`, which starts at `1`.

On any given day, after gaining your daily mana, you can choose to fight and defeat a monster if your current mana points are greater than or equal to that monster's power. When you defeat a monster:
- Your mana points reset to `0`
- Your daily `gain` value increases by `1`

The goal is to find the minimum number of days required to defeat all the monsters.

The solution uses state compression with memoization. Since there are at most 17 monsters, we can represent which monsters are still alive using a [bitmask](/problems/bitmask_intro) (a binary number where the `i-th` bit being `1` means the `i-th` monster is alive).

The recursive function `dfs(mask)` calculates the minimum days needed when the current state of monsters is represented by `mask`. For each alive monster, we calculate how many days it would take to accumulate enough mana to defeat it: `⌈power[i] / gain⌉` days, where `gain = 1 + (number of monsters already defeated)`.

## Intuition

The key insight is recognizing that this is an optimization problem where the order in which we defeat monsters matters significantly. Each time we defeat a monster, our daily mana gain increases, making it easier to defeat subsequent monsters. This creates a dependency where defeating weaker monsters first might allow us to build up our gain value, making stronger monsters easier to defeat later.

Since we need to find the optimal order to defeat all monsters, we need to explore different sequences. With at most 17 monsters, trying all possible permutations would be `17!` which is computationally expensive. However, we can observe that what matters is not the exact sequence, but rather which monsters have been defeated at any point (since this determines our current `gain` value).

This leads us to think about states - at any point in our journey, we can represent our progress by which monsters are still alive. With a small number of monsters (≤17), we can use bit manipulation where each bit represents whether a specific monster is alive (`1`) or defeated (`0`).

The [dynamic programming](/problems/dynamic_programming_intro) insight comes from realizing that once we know the minimum days to defeat a certain subset of monsters, we can use this information to calculate the minimum days for larger subsets. For any state with multiple alive monsters, we try defeating each one and see which choice leads to the minimum total days.

The formula `⌈power[i] / gain⌉` for calculating days needed emerges naturally - if we gain `gain` mana per day and need `power[i]` total mana, we need to wait this many days. The ceiling function handles cases where the power isn't perfectly divisible by the gain.

By using memoization with our recursive approach, we ensure each state is calculated only once, reducing our time complexity from factorial to `O(2^n × n)`, making the solution efficient enough for the given constraints.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) and [Bitmask](/problems/bitmask_intro) patterns.**

## Solution Approach

The solution uses **state compression with memoization** to efficiently explore all possible orders of defeating monsters.

**State Representation:**
We use a [bitmask](/problems/bitmask_intro) `mask` to represent which monsters are still alive. For example, if `mask = 1011` (in binary), it means monsters at positions 0, 1, and 3 are alive, while the monster at position 2 has been defeated.

**Recursive Function Design:**
The function `dfs(mask)` returns the minimum number of days needed to defeat all monsters represented by the current `mask`. The base case is when `mask = 0` (all monsters defeated), which returns `0` days.

**Implementation Steps:**

1. **Initial Call:** Start with `dfs((1 << n) - 1)` where all `n` bits are set to 1, representing all monsters being alive initially.

2. **Calculate Current Gain:** For any state `mask`, the daily mana gain is calculated as:
   ```python
   gain = 1 + (n - mask.bit_count())
   ```
   Here, `mask.bit_count()` gives the number of alive monsters, so `n - mask.bit_count()` is the number of defeated monsters. Since gain starts at 1 and increases by 1 for each defeated monster, we add 1 to get the current gain value.

3. **Try Each Monster:** For each alive monster `i` (where bit `i` in `mask` is 1):
   - Calculate days needed to defeat it: `(power[i] + gain - 1) // gain`
     - This is equivalent to `⌈power[i] / gain⌉` using integer arithmetic
   - Recursively calculate the minimum days for the remaining monsters: `dfs(mask ^ (1 << i))`
     - The XOR operation `mask ^ (1 << i)` flips the `i`-th bit from 1 to 0, representing that monster `i` is now defeated
   - Update the answer with the total days: current days + recursive result

4. **Memoization:** The `@cache` decorator stores results for each unique `mask` value, preventing redundant calculations. Since there are `2^n` possible states and we compute each at most once, this significantly improves efficiency.

5. **Return Minimum:** After trying all possible next monsters to defeat, return the minimum days found.

**Time Complexity:** `O(2^n × n)` where `n` is the number of monsters. We have `2^n` possible states, and for each state, we iterate through up to `n` monsters.

**Space Complexity:** `O(2^n)` for the memoization cache storing results for each possible state.

## Example Walkthrough

Let's walk through a small example with `power = [3, 1, 4]` to illustrate the solution approach.

**Initial Setup:**
- We have 3 monsters with powers [3, 1, 4]
- Starting state: `mask = 111` (binary) = 7 (decimal), meaning all 3 monsters are alive
- Initial gain = 1 (no monsters defeated yet)

**Step 1: Call `dfs(111)`**
- Current gain = 1 + (3 - 3) = 1
- We can choose to defeat any of the 3 monsters first:

**Option A: Defeat Monster 0 (power = 3) first**
- Days needed: ⌈3/1⌉ = 3 days
- New state after defeating: `111 ^ 001 = 110` (monsters 1 and 2 remain)
- Total days = 3 + `dfs(110)`

**Option B: Defeat Monster 1 (power = 1) first**
- Days needed: ⌈1/1⌉ = 1 day
- New state after defeating: `111 ^ 010 = 101` (monsters 0 and 2 remain)
- Total days = 1 + `dfs(101)`

**Option C: Defeat Monster 2 (power = 4) first**
- Days needed: ⌈4/1⌉ = 4 days
- New state after defeating: `111 ^ 100 = 011` (monsters 0 and 1 remain)
- Total days = 4 + `dfs(011)`

Let's explore **Option B** (defeating the weakest monster first):

**Step 2: Call `dfs(101)` - Monsters 0 and 2 alive**
- Current gain = 1 + (3 - 2) = 2 (one monster defeated)
- Choose between monsters 0 and 2:

**Option B1: Defeat Monster 0 (power = 3)**
- Days needed: ⌈3/2⌉ = 2 days
- New state: `101 ^ 001 = 100` (only monster 2 remains)
- Total days = 2 + `dfs(100)`

**Option B2: Defeat Monster 2 (power = 4)**
- Days needed: ⌈4/2⌉ = 2 days
- New state: `101 ^ 100 = 001` (only monster 0 remains)
- Total days = 2 + `dfs(001)`

Following **Option B1**:

**Step 3: Call `dfs(100)` - Only Monster 2 alive**
- Current gain = 1 + (3 - 1) = 3 (two monsters defeated)
- Only choice: defeat Monster 2 (power = 4)
- Days needed: ⌈4/3⌉ = 2 days
- New state: `100 ^ 100 = 000` (all defeated)
- Total days = 2 + `dfs(000)` = 2 + 0 = 2

**Backtracking:**
- `dfs(100)` = 2
- `dfs(101)` = min(2 + 2, 2 + other_option) = 4
- `dfs(111)` = min(1 + 4, other_options) = 5

**Optimal Path Found:**
The minimum is 5 days total:
1. Day 1: Gain 1 mana, defeat Monster 1 (power = 1)
2. Days 2-3: Gain 2 mana/day for 2 days = 4 total, defeat Monster 0 (power = 3)
3. Days 4-5: Gain 3 mana/day for 2 days = 6 total, defeat Monster 2 (power = 4)

The algorithm explores all possible orderings through the recursive calls and memoization ensures we don't recalculate the same state twice, ultimately finding that defeating monsters in order [1, 0, 2] takes the minimum 5 days.

## Python Solution

```python
from typing import List
from functools import cache
from math import inf

class Solution:
    def minimumTime(self, power: List[int]) -> int:
        """
        Calculate minimum time to defeat all monsters.
        Each monster has a power value, and gain increases by 1 for each defeated monster.

        Args:
            power: List of integers representing power of each monster

        Returns:
            Minimum time required to defeat all monsters
        """

        @cache
        def dfs(mask: int) -> int:
            """
            Dynamic programming function to find minimum time for given state.

            Args:
                mask: Bitmask representing which monsters are still alive (1 = alive, 0 = defeated)

            Returns:
                Minimum time to defeat all monsters represented by mask
            """
            # Base case: no monsters left to defeat
            if mask == 0:
                return 0

            # Initialize minimum time to infinity
            min_time = inf

            # Calculate current gain (1 + number of already defeated monsters)
            num_defeated = n - mask.bit_count()
            current_gain = 1 + num_defeated

            # Try defeating each remaining monster
            for i, monster_power in enumerate(power):
                # Check if this monster is still alive (bit at position i is 1)
                if mask >> i & 1:
                    # Calculate new mask after defeating this monster
                    new_mask = mask ^ (1 << i)

                    # Calculate time to defeat this monster with current gain
                    # Using ceiling division: (monster_power + current_gain - 1) // current_gain
                    time_to_defeat = (monster_power + current_gain - 1) // current_gain

                    # Update minimum time
                    min_time = min(min_time, dfs(new_mask) + time_to_defeat)

            return min_time

        # Number of monsters
        n = len(power)

        # Start with all monsters alive (all bits set to 1)
        initial_mask = (1 << n) - 1

        return dfs(initial_mask)
```

## Time and Space Complexity

**Time Complexity:** `O(2^n × n)`

The algorithm uses dynamic programming with bitmask to explore all possible orderings of defeating monsters. The `dfs` function is memoized using `@cache`, which stores results for each unique mask state.

- There are `2^n` possible mask states (each bit represents whether a monster is defeated or not)
- For each mask state, we iterate through all `n` monsters to check which ones can be defeated next (the for loop iterates `n` times)
- Each state is computed only once due to memoization
- Therefore, the total time complexity is `O(2^n × n)`

**Space Complexity:** `O(2^n)`

The space complexity comes from:
- The memoization cache stores up to `2^n` different mask states
- The recursion stack depth is at most `n` (we defeat one monster per recursive call), which is `O(n)`
- Since `O(2^n) > O(n)` for `n > 1`, the dominant space complexity is `O(2^n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Gain Calculation**
A frequent mistake is miscalculating the current gain value. Since gain starts at 1 and increases by 1 for each defeated monster, developers often write:
```python
# WRONG: This gives the number of alive monsters, not defeated ones
current_gain = mask.bit_count()
```

**Correct approach:**
```python
# Calculate number of defeated monsters first
num_defeated = n - mask.bit_count()
current_gain = 1 + num_defeated
```

### 2. **Integer Division Instead of Ceiling Division**
When calculating days needed to defeat a monster, using simple integer division truncates the result:
```python
# WRONG: This rounds down, potentially underestimating days needed
time_to_defeat = power[i] // current_gain
```

**Correct approach:**
```python
# Use ceiling division formula: ⌈a/b⌉ = (a + b - 1) // b
time_to_defeat = (power[i] + current_gain - 1) // current_gain
```

### 3. **Incorrect Bit Manipulation for State Transitions**
Using the wrong operation to update the mask after defeating a monster:
```python
# WRONG: Using OR operation adds the bit instead of removing it
new_mask = mask | (1 << i)

# WRONG: Using AND with the bit itself keeps only that bit
new_mask = mask & (1 << i)
```

**Correct approach:**
```python
# XOR flips the bit from 1 to 0, removing the defeated monster
new_mask = mask ^ (1 << i)
```

### 4. **Checking Monster Availability Incorrectly**
Improperly checking if a monster at position `i` is still alive:
```python
# WRONG: This checks if ANY bit is set, not the specific i-th bit
if mask & 1:
    # process monster i

# WRONG: Forgetting to shift before checking
if mask & (1 << i) == 1:  # This compares the entire result with 1
    # process monster i
```

**Correct approach:**
```python
# Shift mask right by i positions and check the least significant bit
if mask >> i & 1:
    # process monster i

# Alternative: Check if the i-th bit is set
if mask & (1 << i):
    # process monster i
```

### 5. **Missing or Incorrect Base Case**
Forgetting to handle the base case or returning the wrong value:
```python
# WRONG: Forgetting base case causes infinite recursion
def dfs(mask):
    min_time = inf
    for i in range(n):
        # ... recursive calls
    return min_time

# WRONG: Returning wrong value for empty mask
if mask == 0:
    return 1  # Should be 0, as no monsters means 0 days needed
```

**Correct approach:**
```python
if mask == 0:
    return 0  # No monsters left = 0 days needed
```

### 6. **Not Using Memoization**
Implementing the recursive solution without caching results leads to exponential time complexity:
```python
# WRONG: No memoization causes repeated calculations
def dfs(mask):
    # ... implementation without @cache
```

**Correct approach:**
```python
from functools import cache

@cache
def dfs(mask):
    # ... implementation with automatic memoization
```

