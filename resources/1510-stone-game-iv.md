# 1510. Stone Game IV

> Source: https://algo.monster/liteproblems/1510

## Problem Statement

Alice and Bob are playing a turn-based game with a pile of `n` stones. Alice goes first.

On each turn, a player must remove a **square number** of stones from the pile. A square number is any number that can be expressed as `k²` where `k` is a positive integer (examples: 1, 4, 9, 16, 25, ...). The player can choose any valid square number that doesn't exceed the current number of stones in the pile.

The game continues with players alternating turns until one player cannot make a valid move (when there are 0 stones left). The player who cannot make a move loses the game.

Given a positive integer `n` representing the initial number of stones, you need to determine if Alice will win the game, assuming both players play optimally (making the best possible moves). Return `true` if Alice wins, `false` if Bob wins.

**Example scenarios:**
- If `n = 1`: Alice can remove 1 stone (1 = 1²), leaving 0 stones for Bob. Bob cannot move, so Alice wins.
- If `n = 2`: Alice can only remove 1 stone, leaving 1 stone for Bob. Bob removes 1 stone, leaving 0 for Alice. Alice cannot move, so Bob wins.
- If `n = 4`: Alice can remove all 4 stones (4 = 2²), leaving 0 for Bob. Bob cannot move, so Alice wins.

The solution uses [dynamic programming](/problems/dynamic_programming_intro) with memoization. The `dfs(i)` function returns `true` if the current player can win starting with `i` stones. For each state with `i` stones, the function tries all possible square number moves (1, 4, 9, ... up to `i`) and checks if any move leads to a losing position for the opponent. If such a move exists, the current player can win.

## Intuition

The key insight is recognizing this as a **game theory problem** where we need to determine winning and losing positions. In optimal play, each player tries to force their opponent into a losing position.

Let's think about this problem from the ground up:
- When there are 0 stones, the current player has already lost (they can't make a move)
- For any other number of stones, the current player wins if they can make at least one move that puts their opponent in a losing position

This naturally leads to a recursive approach: to know if position `i` is winning, we need to check all possible moves from that position. If we can remove `j²` stones (where `j² ≤ i`), we reach position `i - j²`. If **any** of these resulting positions is a losing position for our opponent, then our current position is a winning position.

Why does this work? Because in optimal play:
- If the current player can force the opponent into even one losing position, they will choose that move and win
- If all possible moves lead to winning positions for the opponent, then the current player is doomed to lose

The recursive structure becomes clear:
- `dfs(0)` returns `False` (base case - no moves available, current player loses)
- `dfs(i)` returns `True` if there exists any `j` where `j² ≤ i` and `dfs(i - j²)` returns `False`
- Otherwise, `dfs(i)` returns `False`

The `@cache` decorator is crucial here because the same game states will be encountered multiple times through different paths. For example, reaching 5 stones could happen from starting with 6 (removing 1) or starting with 9 (removing 4). Memoization ensures we only compute each state once, making the solution efficient.

**Learn more about [Math](/problems/math-basics) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution implements a **top-down [dynamic programming](/problems/dynamic_programming_intro)** approach with memoization using Python's `@cache` decorator.

**Implementation Details:**

1. **Main Function Structure:**
   - The `winnerSquareGame` function takes `n` (number of stones) as input
   - It defines an inner recursive function `dfs(i)` that determines if the current player can win with `i` stones remaining
   - Returns the result of `dfs(n)` to determine if Alice (the first player) wins

2. **Recursive Function `dfs(i)`:**
   ```python
   @cache
   def dfs(i: int) -> bool:
   ```
   - The `@cache` decorator automatically memoizes results, preventing redundant calculations
   - Parameter `i` represents the current number of stones
   - Returns `True` if the current player can win from this position, `False` otherwise

3. **Base Case:**
   ```python
   if i == 0:
       return False
   ```
   - When there are 0 stones, the current player has no valid moves and loses

4. **Recursive Case - Try All Possible Moves:**
   ```python
   j = 1
   while j * j <= i:
       if not dfs(i - j * j):
           return True
       j += 1
   return False
   ```
   - Iterate through all possible square numbers from 1² to the largest square ≤ `i`
   - For each valid move removing `j²` stones, check the resulting position `i - j²`
   - If `dfs(i - j²)` returns `False`, it means the opponent will lose from that position, so the current player wins by making this move
   - If we find any such winning move, immediately return `True`
   - If all moves lead to winning positions for the opponent, return `False`

**Time and Space Complexity:**
- **Time Complexity:** `O(n√n)` - For each of the `n` possible states, we check up to `√n` possible moves
- **Space Complexity:** `O(n)` - The cache stores results for up to `n` different states, plus the recursion stack depth

The elegance of this solution lies in its simplicity: it directly models the game's win/lose conditions and uses memoization to efficiently handle overlapping subproblems.

## Example Walkthrough

Let's trace through the algorithm with `n = 5` stones to see how the solution determines the winner.

**Starting with dfs(5):**
- We need to check if the current player (Alice) can win with 5 stones
- Possible moves: remove 1 (1²) or 4 (2²) stones
- Try removing 1: Check `dfs(5 - 1) = dfs(4)`
  - **Computing dfs(4):**
    - Possible moves: remove 1 or 4 stones
    - Try removing 1: Check `dfs(4 - 1) = dfs(3)`
      - **Computing dfs(3):**
        - Possible moves: remove 1 stone only
        - Try removing 1: Check `dfs(3 - 1) = dfs(2)`
          - **Computing dfs(2):**
            - Possible moves: remove 1 stone only
            - Try removing 1: Check `dfs(2 - 1) = dfs(1)`
              - **Computing dfs(1):**
                - Possible moves: remove 1 stone only
                - Try removing 1: Check `dfs(1 - 1) = dfs(0)`
                - `dfs(0)` returns `False` (base case)
                - Since opponent loses, `dfs(1)` returns `True`
            - `dfs(1)` returned `True`, so opponent wins
            - No winning move found, `dfs(2)` returns `False`
        - `dfs(2)` returned `False`, so opponent loses
        - Found a winning move! `dfs(3)` returns `True`
    - `dfs(3)` returned `True`, so opponent wins
    - Try removing 4: Check `dfs(4 - 4) = dfs(0)`
    - `dfs(0)` returns `False` (base case)
    - Found a winning move! `dfs(4)` returns `True`
  - `dfs(4)` returned `True`, so opponent (Bob) wins if we leave 4 stones
- Try removing 4: Check `dfs(5 - 4) = dfs(1)`
  - We already computed `dfs(1) = True` (cached)
  - Opponent (Bob) wins if we leave 1 stone
- Both moves lead to winning positions for Bob
- No winning move found, `dfs(5)` returns `False`

**Result:** Alice loses with 5 stones, so Bob wins the game.

The key insight: Alice can only leave Bob with either 4 stones or 1 stone. In both cases, Bob has a winning strategy (removing all 4 stones from 4, or removing 1 stone from 1), so Alice is doomed to lose from position 5.

## Python Solution

```python
class Solution:
    def winnerSquareGame(self, n: int) -> bool:
        """
        Determine if Alice wins the square game with optimal play.
      
        Game rules: Players take turns removing a perfect square number of stones.
        The player who removes the last stone wins.
      
        Args:
            n: Number of stones initially in the pile
          
        Returns:
            True if Alice (first player) wins, False otherwise
        """
        from functools import cache
      
        @cache
        def can_win(remaining_stones: int) -> bool:
            """
            Check if the current player can win from this game state.
          
            Args:
                remaining_stones: Number of stones left in the pile
              
            Returns:
                True if current player can force a win, False otherwise
            """
            # Base case: no stones left means previous player took the last stone
            if remaining_stones == 0:
                return False
          
            # Try all possible moves (removing perfect square number of stones)
            square_root = 1
            while square_root * square_root <= remaining_stones:
                stones_to_remove = square_root * square_root
              
                # If opponent cannot win after our move, then we can win
                if not can_win(remaining_stones - stones_to_remove):
                    return True
                  
                square_root += 1
          
            # No winning move found
            return False
      
        # Alice starts first, check if she can win
        return can_win(n)
```

## Time and Space Complexity

**Time Complexity:** `O(n * √n)`

The analysis follows from the memoized recursive approach:
- The `dfs` function can be called with at most `n` different states (from `0` to `n`).
- Due to the `@cache` decorator, each state is computed only once.
- For each state `i`, we iterate through all perfect squares less than or equal to `i`, which requires checking approximately `√i` values.
- In the worst case, when `i = n`, we check up to `√n` perfect squares.
- Therefore, the total time complexity is `O(n) * O(√n) = O(n * √n)`.

**Space Complexity:** `O(n)`

The space complexity consists of:
- **Memoization cache:** The `@cache` decorator stores up to `n` different states, requiring `O(n)` space.
- **Recursion call stack:** In the worst case, the recursion depth can reach `O(n)` when we repeatedly subtract 1 (which is a perfect square), leading to a chain of calls like `dfs(n) → dfs(n-1) → dfs(n-2) → ... → dfs(0)`.
- Since both the cache and the call stack can use `O(n)` space, the overall space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Inefficient Square Number Generation**
A common mistake is recalculating square numbers repeatedly or using inefficient methods to check if a number is a perfect square.

**Pitfall Example:**
```python
# Inefficient: Checking all numbers up to n to find squares
for remove in range(1, i + 1):
    if int(remove ** 0.5) ** 2 == remove:  # Floating point issues!
        if not dfs(i - remove):
            return True
```

**Problems:**
- Iterates through all numbers instead of just perfect squares
- Using floating-point operations (`** 0.5`) can lead to precision errors
- Time complexity becomes O(n²) instead of O(n√n)

**Solution:**
Generate perfect squares directly by incrementing the base and squaring:
```python
j = 1
while j * j <= i:
    if not dfs(i - j * j):
        return True
    j += 1
```

### 2. **Missing or Incorrect Memoization**
Without proper memoization, the solution will have exponential time complexity due to recalculating the same game states.

**Pitfall Example:**
```python
def dfs(i):  # No memoization!
    if i == 0:
        return False
    j = 1
    while j * j <= i:
        if not dfs(i - j * j):
            return True
        j += 1
    return False
```

**Problems:**
- Same game states are computed multiple times
- Time complexity becomes exponential
- Will timeout for larger values of n

**Solution:**
Use `@cache` decorator or implement manual memoization:
```python
from functools import cache

@cache
def dfs(i):
    # ... rest of the code
```

Or with manual memoization:
```python
memo = {}
def dfs(i):
    if i in memo:
        return memo[i]
    # ... compute result
    memo[i] = result
    return result
```

### 3. **Incorrect Base Case Logic**
Misunderstanding when the game ends and who wins.

**Pitfall Example:**
```python
def dfs(i):
    if i == 0:
        return True  # Wrong! Current player already lost
```

**Problems:**
- When i = 0, the current player has no moves available
- The previous player took the last stone and won
- Returning True would incorrectly indicate the current player wins

**Solution:**
Return False when no stones remain, as the current player cannot move:
```python
if i == 0:
    return False  # Current player loses (cannot make a move)
```

### 4. **Off-by-One Errors in Square Calculation**
Incorrectly handling the boundary conditions for valid square numbers.

**Pitfall Example:**
```python
j = 0  # Starting from 0 instead of 1
while j * j <= i:
    if not dfs(i - j * j):  # When j=0, removes 0 stones!
        return True
    j += 1
```

**Problems:**
- Starting j from 0 means considering removing 0 stones as a valid move
- This creates an infinite loop or incorrect game logic

**Solution:**
Always start from j = 1 (smallest perfect square):
```python
j = 1  # Start from 1² = 1
while j * j <= i:
    # ... rest of the code
```

