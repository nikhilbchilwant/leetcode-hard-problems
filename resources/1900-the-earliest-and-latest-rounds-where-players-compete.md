# 1900. The Earliest and Latest Rounds Where Players Compete

> Source: https://algo.monster/liteproblems/1900

## Problem Statement

You have a tournament with `n` players standing in a row, numbered from `1` to `n` based on their initial positions (player `1` is first, player `2` is second, etc.).

The tournament consists of multiple rounds, and in each round:
- The `i-th` player from the front competes against the `i-th` player from the end
- Winners advance to the next round
- If there's an odd number of players, the middle player automatically advances

For example, with players `[1, 2, 4, 6, 7]`:
- Player `1` competes against player `7`
- Player `2` competes against player `6`  
- Player `4` (middle) automatically advances

After each round, winners are re-arranged in their original numerical order (ascending) before the next round begins.

Two special players, `firstPlayer` and `secondPlayer`, are the best in the tournament - they will always win against any other player except each other. For all other matches between regular players, you can choose who wins.

Given `n` (total players), `firstPlayer`, and `secondPlayer`, you need to find:
1. The **earliest** possible round where these two players could compete
2. The **latest** possible round where these two players could compete

Return these two values as an array `[earliest_round, latest_round]`.

## Intuition

The key observation is that we need to track the positions of our two special players as they advance through rounds. Since they always win their matches (except against each other), we can control when they meet by strategically choosing winners in other matches.

Think about what happens in each round:
- Players pair up from opposite ends of the row
- Winners maintain their original ordering for the next round
- The two special players will definitely advance until they meet

The challenge is that after each round, the number of players reduces and their relative positions change. If we have `n` players in a round, roughly `n/2` advance to the next round (plus one more if `n` is odd).

To find both earliest and latest meeting rounds, we need to explore all possible outcomes of matches between regular players. For each round, we can represent match outcomes as a binary choice - for each pair of regular players, either the front player wins (bit = 1) or the back player wins (bit = 0).

This leads us to a recursive approach with [memoization](/problems/memoization_intro):
- State: positions of our two special players (`l` and `r`) among `n` current players
- Base case: when `l + r = n - 1`, they're paired against each other (one from front, one from back)
- Transition: try all possible match outcomes for regular players, track where our special players end up in the next round

By using binary enumeration (`1 << m` combinations for `m` pairs), we can systematically explore all possible tournament progressions. For each possibility:
1. Determine which players advance based on our choice
2. Find the new positions of our special players among winners
3. Recursively solve for the smaller tournament
4. Track minimum rounds (earliest) and maximum rounds (latest)

The [memoization](/problems/memoization_intro) with `@cache` prevents recalculating the same state multiple times, as the same configuration of players can be reached through different paths.

**Learn more about [Memoization](/problems/memoization_intro) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution uses [memoization](/problems/memoization_intro) with binary enumeration to explore all possible tournament outcomes.

**Core Function: `dfs(l, r, n)`**

This function returns `[earliest_round, latest_round]` for players at positions `l` and `r` among `n` total players.

**Base Case:**
- When `l + r = n - 1`, the two players face each other (one from position `l` from front, other from position `n-1-r` from back, which equals `l`)
- Return `[1, 1]` since they meet in the current round

**Recursive Case:**

1. **Calculate number of matches:** `m = n >> 1` (equivalent to `n // 2`)
   - This gives us the number of player pairs competing

2. **Binary enumeration of match outcomes:** Loop through `i` from `0` to `2^m - 1`
   - Each bit in `i` represents one match outcome
   - If bit `j` is 1: front player `j` wins
   - If bit `j` is 0: back player `n-1-j` wins

3. **Determine winners for each configuration:**
   ```python
   win = [False] * n
   for j in range(m):
       if i >> j & 1:  # Check if j-th bit is 1
           win[j] = True
       else:
           win[n - 1 - j] = True
   ```
   - If `n` is odd, middle player automatically advances: `win[m] = True`
   - Force our special players to win: `win[l] = True` and `win[r] = True`
   - Their opponents must lose: `win[n - 1 - l] = False` and `win[n - 1 - r] = False`

4. **Calculate new positions in next round:**
   ```python
   a = b = c = 0
   for j in range(n):
       if j == l: a = c  # New position of first special player
       if j == r: b = c  # New position of second special player
       if win[j]: c += 1  # Count winners before current position
   ```
   - Variable `c` counts winners up to position `j`
   - When we reach special player positions, record their new positions among winners

5. **Recursive call and update results:**
   ```python
   x, y = dfs(a, b, c)  # Get results for next round
   res[0] = min(res[0], x + 1)  # Update earliest (minimum)
   res[1] = max(res[1], y + 1)  # Update latest (maximum)
   ```
   - Add 1 to account for the current round

**[Memoization](/problems/memoization_intro):**
- The `@cache` decorator automatically memoizes results
- Prevents recalculating the same state `(l, r, n)` multiple times

**Main Function:**
- Converts 1-indexed player numbers to 0-indexed positions
- Calls `dfs(firstPlayer - 1, secondPlayer - 1, n)`

The algorithm explores all possible tournament trees where the two special players advance, finding both the shortest and longest paths to their eventual meeting.

## Example Walkthrough

Let's walk through a small example with `n = 5`, `firstPlayer = 1`, `secondPlayer = 5`.

Initial setup: Players `[1, 2, 3, 4, 5]` where players 1 and 5 are special (always win).

**Round 1 Analysis:**
- We call `dfs(0, 4, 5)` (positions 0 and 4 for players 1 and 5)
- Check base case: `0 + 4 = 4 = 5 - 1`, so they face each other now!
- Player 1 (position 0) faces Player 5 (position 4 from back, which is position 0 from end)
- Return `[1, 1]` - they meet in round 1

Now let's try `n = 7`, `firstPlayer = 2`, `secondPlayer = 6`:

Initial: Players `[1, 2, 3, 4, 5, 6, 7]` where players 2 and 6 are special.

**Round 1:**
- Call `dfs(1, 5, 7)` (positions 1 and 5)
- Base case check: `1 + 5 = 6 ≠ 7 - 1`, so continue
- Number of matches: `m = 7 >> 1 = 3`
- Player 4 (middle) auto-advances

We explore different match outcomes using binary enumeration (8 possibilities):

**Example Configuration (i = 3, binary: 011):**
- Bit 0 = 1: Player 1 beats Player 7
- Bit 1 = 1: Player 2 beats Player 6 (Player 2 must win anyway)
- Bit 2 = 0: Player 5 beats Player 3
- Player 4 auto-advances

Winners: `[1, 2, 4, 5]` → Player 2 is at position 1, Player 6 didn't advance (eliminated)

This path is invalid since Player 6 must win.

**Valid Configuration (i = 2, binary: 010):**
- Bit 0 = 0: Player 7 beats Player 1
- Bit 1 = 1: Player 2 beats Player 6 (forced - Player 2 must win)
- Bit 2 = 0: Player 5 beats Player 3
- Player 4 auto-advances

But wait - Player 6 at position 5 faces Player 2 at position 1, so Player 6 must also win!
Let's correct this:

**Valid Configuration for Earliest Meeting:**
Choose outcomes to bring players 2 and 6 closer:
- Player 2 (pos 1) vs Player 6 (pos 5) - they don't face each other yet
- Player 1 (pos 0) vs Player 7 (pos 6) - let Player 1 win
- Player 2 (pos 1) vs Player 6 (pos 5) - NO, Player 2 faces Player 6's opponent
- Actually: Player 2 (pos 1) faces Player 6 (pos 5) from back = position 1 from end

Let me recalculate the pairings:
- Position 0 vs Position 6 (Player 1 vs Player 7)
- Position 1 vs Position 5 (Player 2 vs Player 6)
- Position 2 vs Position 4 (Player 3 vs Player 5)
- Position 3 (Player 4) auto-advances

Wait, `1 + 5 = 6 = 7 - 1`, so Players 2 and 6 DO face each other in round 1!

Let's try `n = 6`, `firstPlayer = 2`, `secondPlayer = 5`:

**Round 1:**
- Call `dfs(1, 4, 6)`
- Base case: `1 + 4 = 5 ≠ 6 - 1`, continue
- Matches: Player 1 vs 6, Player 2 vs 5, Player 3 vs 4
- Player 2 faces Player 5! Check: position 1 from front vs position 4 from back
- Position 4 from back is position 6-1-4 = 1 from end... that's position 4!
- So Player at position 1 faces player at position 4

Actually, the pairing works as: position `i` faces position `n-1-i`.
- Position 1 faces position 6-1-1 = 4 ✓

So players 2 and 5 meet in round 1.

For a case where they don't meet immediately, consider `n = 7`, `firstPlayer = 1`, `secondPlayer = 4`:

**Round 1:**
- Call `dfs(0, 3, 7)`
- Base case: `0 + 3 = 3 ≠ 6`, continue
- Pairings: 1 vs 7, 2 vs 6, 3 vs 5, 4 auto-advances
- Players 1 and 4 don't face each other

By controlling who wins between 2 vs 6 and 3 vs 5, we can influence when players 1 and 4 eventually meet. The algorithm explores all such possibilities to find both the earliest and latest rounds where they can meet.

## Python Solution

```python
from functools import cache
from typing import List

class Solution:
    def earliestAndLatest(
        self, n: int, firstPlayer: int, secondPlayer: int
    ) -> List[int]:
        """
        Find the earliest and latest rounds when two players can meet in a tournament.
      
        Args:
            n: Total number of players
            firstPlayer: Position of the first player (1-indexed)
            secondPlayer: Position of the second player (1-indexed)
      
        Returns:
            List containing [earliest_round, latest_round]
        """
      
        @cache
        def find_min_max_rounds(left_pos: int, right_pos: int, remaining_players: int) -> List[int]:
            """
            Recursively find minimum and maximum rounds for two players to meet.
          
            Args:
                left_pos: 0-indexed position of left player
                right_pos: 0-indexed position of right player
                remaining_players: Number of players remaining in current round
          
            Returns:
                List containing [min_rounds, max_rounds]
            """
            # Base case: if players are adjacent in the pairing, they meet next round
            if left_pos + right_pos == remaining_players - 1:
                return [1, 1]
          
            # Initialize result with worst case bounds
            min_rounds = float('inf')
            max_rounds = float('-inf')
          
            # Calculate number of matches in this round
            num_matches = remaining_players >> 1  # Integer division by 2
          
            # Try all possible match outcomes (2^num_matches possibilities)
            for outcome_mask in range(1 << num_matches):
                # Create array to track which players win their matches
                winners = [False] * remaining_players
              
                # Determine winners based on outcome_mask bits
                for match_idx in range(num_matches):
                    if outcome_mask >> match_idx & 1:
                        # Player at position match_idx wins
                        winners[match_idx] = True
                    else:
                        # Player at symmetric position wins
                        winners[remaining_players - 1 - match_idx] = True
              
                # If odd number of players, middle player gets a bye
                if remaining_players & 1:
                    winners[num_matches] = True
              
                # Force our two tracked players to win their matches
                winners[remaining_players - 1 - left_pos] = False
                winners[remaining_players - 1 - right_pos] = False
                winners[left_pos] = True
                winners[right_pos] = True
              
                # Calculate new positions after this round
                new_left_pos = 0
                new_right_pos = 0
                position_counter = 0
              
                for player_idx in range(remaining_players):
                    if player_idx == left_pos:
                        new_left_pos = position_counter
                    if player_idx == right_pos:
                        new_right_pos = position_counter
                    if winners[player_idx]:
                        position_counter += 1
              
                # Recursively find min/max rounds for next round
                sub_min, sub_max = find_min_max_rounds(
                    new_left_pos, new_right_pos, position_counter
                )
              
                # Update overall min and max
                min_rounds = min(min_rounds, sub_min + 1)
                max_rounds = max(max_rounds, sub_max + 1)
          
            return [min_rounds, max_rounds]
      
        # Convert to 0-indexed positions and start recursion
        return find_min_max_rounds(firstPlayer - 1, secondPlayer - 1, n)
```

## Time and Space Complexity

**Time Complexity:** `O(n^3 * 2^(n/2))`

The analysis breaks down as follows:
- The function `dfs` has at most `O(n^2)` unique states since parameters `l` and `r` can each range from `0` to `n-1`, and `n` decreases with each recursive call
- For each state, the function iterates through `2^(n/2)` possibilities (the for loop with `range(1 << m)` where `m = n >> 1`)
- Inside this loop, there's an `O(n)` operation to process the `win` array and calculate values `a`, `b`, and `c`
- Due to memoization with `@cache`, each unique state is computed only once
- The recursion depth is at most `O(log n)` since `n` roughly halves with each recursive level
- Total complexity: `O(n^2) * O(2^(n/2)) * O(n) = O(n^3 * 2^(n/2))`

**Space Complexity:** `O(n^3)`

The space analysis includes:
- The memoization cache stores at most `O(n^3)` states (considering all possible combinations of `l`, `r`, and `n`)
- The recursion stack depth is `O(log n)`
- Each recursive call uses `O(n)` space for the `win` array
- Total space: `O(n^3)` for the cache dominates, giving us `O(n^3)` overall space complexity

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Winner Assignment for Symmetric Matches**
A critical pitfall is incorrectly handling the constraint that when two players compete, exactly one must win and one must lose. When forcing the special players to win, you must ensure their opponents lose.

**Incorrect approach:**
```python
# Only setting winners without considering opponents
winners[left_pos] = True
winners[right_pos] = True
```

**Correct approach:**
```python
# First set opponents to lose, then set special players to win
winners[remaining_players - 1 - left_pos] = False
winners[remaining_players - 1 - right_pos] = False
winners[left_pos] = True
winners[right_pos] = True
```

### 2. **Edge Case: Special Players Competing Against Each Other**
When the two special players face each other in the current round (i.e., `left_pos + right_pos == remaining_players - 1`), trying to force both to win creates a logical contradiction.

**Solution:** Check this condition first as the base case and return `[1, 1]` immediately, avoiding the winner assignment logic entirely.

### 3. **Bit Manipulation Confusion**
Understanding how the outcome mask works can be tricky. Each bit represents a match outcome, not a player position.

**Common mistake:**
```python
# Thinking bit i represents player i
if outcome_mask & (1 << player_idx):  # Wrong!
```

**Correct understanding:**
```python
# Bit j represents match j (between player j and player n-1-j)
if outcome_mask >> match_idx & 1:
    winners[match_idx] = True
else:
    winners[remaining_players - 1 - match_idx] = True
```

### 4. **Position Tracking After Elimination**
After each round, players are re-ordered based on their original numbers (1 to n), not their current positions. The new position depends on how many winners appear before them in the original ordering.

**Incorrect:**
```python
# Assuming positions are reassigned sequentially
new_positions = list(range(num_winners))  # Wrong!
```

**Correct:**
```python
position_counter = 0
for player_idx in range(remaining_players):
    if player_idx == left_pos:
        new_left_pos = position_counter
    if player_idx == right_pos:
        new_right_pos = position_counter
    if winners[player_idx]:
        position_counter += 1
```

### 5. **Initialization Bounds for Min/Max**
Using incorrect initial values can lead to wrong results.

**Pitfall:**
```python
min_rounds = 0  # Too small
max_rounds = n   # Might be too large or too small
```

**Solution:**
```python
min_rounds = float('inf')
max_rounds = float('-inf')
```

These ensure any valid value will properly update the bounds during iteration.

