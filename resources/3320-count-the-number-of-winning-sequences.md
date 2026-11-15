# 3320. Count The Number of Winning Sequences

> Source: https://algo.monster/liteproblems/3320

## Problem Statement

This is a game theory problem where Alice and Bob play a fantasy battle game for `n` rounds. In each round, both players simultaneously summon one of three magical creatures:
- Fire Dragon (F)
- Water Serpent (W)  
- Earth Golem (E)

The scoring rules follow a rock-paper-scissors pattern:
- Fire Dragon beats Earth Golem (Fire Dragon player gets 1 point)
- Water Serpent beats Fire Dragon (Water Serpent player gets 1 point)
- Earth Golem beats Water Serpent (Earth Golem player gets 1 point)
- If both players summon the same creature, no points are awarded

You're given a string `s` of length `n` containing characters 'F', 'W', and 'E', representing Alice's predetermined sequence of creatures for each round.

Bob's strategy has one constraint: he cannot summon the same creature in two consecutive rounds. Bob wins if his total points are strictly greater than Alice's points after all `n` rounds.

The task is to count how many distinct valid sequences Bob can use to beat Alice. Since this number can be very large, return the result modulo `10^9 + 7`.

For example, if Alice's sequence is "FWE", Bob needs to find sequences where:
1. He doesn't repeat creatures consecutively
2. His total score against Alice's sequence is positive (Bob's points > Alice's points)

The solution uses [dynamic programming](/problems/dynamic_programming_intro) with memoization, tracking:
- Current round position
- Score difference (Alice's score minus Bob's score)
- Bob's last summoned creature (to enforce the no-consecutive-repeat rule)

The function explores all valid sequences Bob can play and counts those where he ends with more points than Alice.

## Intuition

The key insight is that we need to count all possible sequences Bob can play that result in him winning. Since Bob's choices depend on what he played previously (no consecutive repeats), this suggests a [dynamic programming](/problems/dynamic_programming_intro) approach where we track the game state.

Think about what information we need to determine if Bob can win from any given position:
1. **Current round** (`i`): We need to know which round we're in to access Alice's move from string `s`
2. **Score difference** (`j`): We need to track the running score to know if Bob is winning
3. **Bob's last move** (`k`): We need this to ensure Bob doesn't repeat creatures consecutively

For each state, we can try all three possible creatures Bob could summon (F, W, E), but skip any that match his previous move. Each choice leads to a new state with updated score difference.

The scoring can be viewed as a circular relationship: F → E → W → F, where each creature beats the next one in the cycle. We can encode this as numbers (F=0, W=1, E=2) and create a scoring function `calc(x, y)` that returns:
- `1` if `x` beats `y` 
- `-1` if `y` beats `x`
- `0` if it's a tie

An important optimization comes from recognizing when Bob can no longer win: if the remaining rounds `(n - i)` are less than or equal to the current score deficit `j` (where `j` represents Alice's lead), Bob cannot catch up even if he wins every remaining round. This allows us to prune the search tree early.

The memoization helps avoid recalculating the same game states multiple times. Starting from round 0 with score difference 0 and no previous move for Bob (represented as -1), we recursively explore all valid sequences and count those where Bob ends with a negative score difference (meaning Bob has more points than Alice).

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution uses memoization search ([dynamic programming](/problems/dynamic_programming_intro) with recursion) to count all valid sequences where Bob beats Alice.

**Data Structures:**
- Dictionary `d = {"F": 0, "W": 1, "E": 2}` maps characters to numbers for easier calculation
- `@cache` decorator for memoization to store already computed states

**Helper Function:**
The `calc(x, y)` function determines the scoring outcome between creatures `x` and `y`:
- If `x == y`: return `0` (tie)
- If `x < y`: 
  - If `x == 0` and `y == 2` (Fire vs Earth), return `1` (Fire wins)
  - Otherwise return `-1` (the smaller number loses)
- If `x > y`:
  - If `x == 2` and `y == 0` (Earth vs Fire), return `-1` (Earth loses)
  - Otherwise return `1` (the larger number wins)

**Main DFS Function:**
The `dfs(i, j, k)` function computes the number of winning sequences for Bob:
- **Parameters:**
  - `i`: current round index (0 to n-1)
  - `j`: score difference (Alice's score - Bob's score)
  - `k`: Bob's last summoned creature (-1 if no previous creature)

- **Base Cases:**
  1. If `len(s) - i <= j`: Bob cannot catch up even if he wins all remaining rounds, return `0`
  2. If `i >= len(s)`: All rounds completed
     - Return `1` if `j < 0` (Bob has more points)
     - Return `0` otherwise

- **Recursive Case:**
  For each possible creature Bob can summon (0, 1, 2):
  - Skip if it equals `k` (previous creature)
  - Calculate new score difference: `j + calc(d[s[i]], l)`
    - `d[s[i]]` is Alice's creature this round
    - `l` is Bob's chosen creature
    - The result updates the score difference
  - Recursively call `dfs(i + 1, new_j, l)`
  - Sum all valid sequences modulo `10^9 + 7`

**Algorithm Flow:**
1. Start with `dfs(0, 0, -1)`: round 0, score difference 0, no previous creature
2. At each state, try all valid creatures for Bob
3. Update score based on the matchup
4. Recursively explore the next round
5. Count sequences where Bob wins (final score difference < 0)
6. Clear cache after computation and return the result

The time complexity is `O(n² × 3)` where `n` is the length of string `s`, as we have at most `n` rounds, `O(n)` possible score differences, and 3 possible previous creatures for each state.

## Example Walkthrough

Let's walk through a small example where Alice's sequence is `s = "FW"` (2 rounds).

**Round-by-round analysis:**
- Round 1: Alice plays Fire (F)
- Round 2: Alice plays Water (W)

We need to count all sequences where Bob wins (gets more points than Alice).

**Starting state:** `dfs(0, 0, -1)`
- Round 0, score difference 0, Bob has no previous creature

**Round 1 (i=0):** Alice plays F
Bob can play F, W, or E (no restriction since k=-1):

1. **Bob plays F:** 
   - F vs F → tie (0 points)
   - New score: 0 + 0 = 0
   - Continue with `dfs(1, 0, 0)` (Bob's last = F)

2. **Bob plays W:**
   - W vs F → W wins (+1 for Bob)
   - New score: 0 + (-1) = -1 (negative means Bob is ahead)
   - Continue with `dfs(1, -1, 1)` (Bob's last = W)

3. **Bob plays E:**
   - E vs F → F wins (+1 for Alice)
   - New score: 0 + 1 = 1 (positive means Alice is ahead)
   - Continue with `dfs(1, 1, 2)` (Bob's last = E)

**Round 2 (i=1):** Alice plays W
For each branch from Round 1:

**From branch 1** `dfs(1, 0, 0)` (Bob played F, score tied):
- Bob cannot play F again (k=0)
- Bob plays W: W vs W → tie (0), final score: 0 (tie) ❌
- Bob plays E: E vs W → E wins (+1 for Bob), final score: -1 (Bob wins) ✓

**From branch 2** `dfs(1, -1, 1)` (Bob played W, Bob ahead by 1):
- Bob cannot play W again (k=1)
- Bob plays F: F vs W → W wins (+1 for Alice), final score: 0 (tie) ❌
- Bob plays E: E vs W → E wins (+1 for Bob), final score: -2 (Bob wins) ✓

**From branch 3** `dfs(1, 1, 2)` (Bob played E, Alice ahead by 1):
- Bob cannot play E again (k=2)
- Bob plays F: F vs W → W wins (+1 for Alice), final score: 2 (Alice wins) ❌
- Bob plays W: W vs W → tie (0), final score: 1 (Alice wins) ❌

**Valid winning sequences for Bob:**
1. F→E (tie in round 1, Bob wins round 2) → Bob wins 1-0
2. W→E (Bob wins round 1, Bob wins round 2) → Bob wins 2-0

Answer: **2** valid sequences where Bob beats Alice.

## Python Solution

```python
class Solution:
    def countWinningSequences(self, s: str) -> int:
        """
        Count the number of winning sequences for Bob against Alice's sequence.
        Alice's sequence is given as string s containing 'F', 'W', 'E'.
        Bob needs to create a sequence that beats Alice's with constraints:
        - No consecutive elements can be the same in Bob's sequence
        - Bob wins if his total score is positive
        """
      
        def calculate_score(alice_move: int, bob_move: int) -> int:
            """
            Calculate the score for a single round.
            Returns: 1 if Bob wins, -1 if Alice wins, 0 if tie
          
            Game rules (Rock-Paper-Scissors style):
            - Fire (0) beats Earth (2)
            - Water (1) beats Fire (0)  
            - Earth (2) beats Water (1)
            """
            if alice_move == bob_move:
                return 0  # Tie
          
            if alice_move < bob_move:
                # Special case: Fire (0) vs Earth (2) - Fire wins
                if alice_move == 0 and bob_move == 2:
                    return 1
                return -1  # Otherwise Bob wins this comparison
          
            # alice_move > bob_move
            # Special case: Earth (2) vs Fire (0) - Fire wins
            if alice_move == 2 and bob_move == 0:
                return -1
            return 1  # Otherwise Alice wins this comparison
      
        @cache
        def count_sequences(position: int, score_difference: int, last_bob_move: int) -> int:
            """
            Dynamic programming function to count valid winning sequences.
          
            Args:
                position: Current position in Alice's sequence
                score_difference: Current score (Alice's score - Bob's score)
                last_bob_move: Bob's last move (0=F, 1=W, 2=E, -1=no move yet)
          
            Returns:
                Number of valid sequences from this state
            """
            # Pruning: If remaining rounds can't overcome the deficit
            remaining_rounds = len(s) - position
            if remaining_rounds <= score_difference:
                return 0
          
            # Base case: Reached end of sequence
            if position >= len(s):
                # Bob wins if his score is higher (score_difference < 0)
                return int(score_difference < 0)
          
            total_sequences = 0
            alice_current_move = move_mapping[s[position]]
          
            # Try all possible moves for Bob (0=Fire, 1=Water, 2=Earth)
            for bob_move in range(3):
                # Skip if Bob would repeat his last move
                if bob_move == last_bob_move:
                    continue
              
                # Calculate new score after this round
                round_score = calculate_score(alice_current_move, bob_move)
                new_score_difference = score_difference + round_score
              
                # Recursively count sequences from next position
                sequences_from_here = count_sequences(position + 1, new_score_difference, bob_move)
                total_sequences = (total_sequences + sequences_from_here) % MOD
          
            return total_sequences
      
        # Constants
        MOD = 10**9 + 7
        move_mapping = {"F": 0, "W": 1, "E": 2}  # Map characters to integers
      
        # Start the recursion from position 0, score 0, no previous move (-1)
        result = count_sequences(0, 0, -1)
      
        # Clear the cache to free memory
        count_sequences.cache_clear()
      
        return result
```

## Time and Space Complexity

The time complexity is `O(n^2 × k)`, where `n` is the length of the string `s`, and `k = 3` represents the number of possible moves (F, W, E).

**Time Complexity Analysis:**
- The `dfs` function has three parameters: `i` (position in string), `j` (score difference), and `k` (last move used)
- Parameter `i` ranges from `0` to `n-1`, giving us `n` possible values
- Parameter `j` represents the score difference between Bob and Alice. In the worst case, Bob wins or loses all rounds, so `j` ranges from `-n` to `n`, giving us `2n+1 = O(n)` possible values
- Parameter `k` represents the last move and has `k = 3` possible values (plus the initial `-1`)
- For each state `(i, j, k)`, we iterate through at most `k-1 = 2` moves (excluding the previous move)
- Total number of states: `O(n × n × k) = O(n^2 × k)`
- Work per state: `O(k)` for the loop
- Total time complexity: `O(n^2 × k × k) = O(n^2 × k^2)` where `k = 3`

The space complexity is `O(n^2 × k)`, where `n` is the length of the string `s`, and `k = 3`.

**Space Complexity Analysis:**
- The `@cache` decorator memoizes all unique calls to `dfs(i, j, k)`
- As analyzed above, there are `O(n × n × k) = O(n^2 × k)` unique states
- Each cached entry stores an integer result
- The recursion depth is at most `n` (going through the entire string)
- Total space complexity: `O(n^2 × k)` for the cache plus `O(n)` for recursion stack, which simplifies to `O(n^2 × k)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Score Calculation Logic**
The most common pitfall is misunderstanding the scoring rules and implementing the wrong winner determination in the `calculate_score` function. The rock-paper-scissors pattern with Fire, Water, and Earth can be confusing when mapping to numerical comparisons.

**Pitfall Example:**
```python
# WRONG: Assuming simple circular comparison
def calculate_score(alice_move, bob_move):
    if alice_move == bob_move:
        return 0
    return 1 if (bob_move - alice_move) % 3 == 1 else -1
```

**Solution:**
The correct implementation needs to handle the special cases where Fire (0) beats Earth (2):
```python
def calculate_score(alice_move, bob_move):
    if alice_move == bob_move:
        return 0
  
    # Handle the wrap-around case: Fire (0) beats Earth (2)
    if alice_move == 0 and bob_move == 2:
        return -1  # Alice wins
    if alice_move == 2 and bob_move == 0:
        return 1   # Bob wins
  
    # Normal cases: higher number beats lower number
    return 1 if alice_move > bob_move else -1
```

### 2. **Score Difference Sign Confusion**
Another critical pitfall is mixing up whether the score difference represents "Alice - Bob" or "Bob - Alice", leading to incorrect win condition checks.

**Pitfall Example:**
```python
# WRONG: Confusing the sign of score difference
if position >= len(s):
    return int(score_difference > 0)  # Wrong condition!
```

**Solution:**
Be consistent with the score difference representation. If tracking `Alice's score - Bob's score`, then Bob wins when this value is negative:
```python
# Correct: Bob wins when Alice's score - Bob's score < 0
if position >= len(s):
    return int(score_difference < 0)
```

### 3. **Forgetting the Modulo Operation**
When counting large numbers of sequences, intermediate results can overflow. Forgetting to apply modulo at each step can cause incorrect results or overflow errors.

**Pitfall Example:**
```python
# WRONG: Only applying modulo at the end
total_sequences = 0
for bob_move in range(3):
    if bob_move != last_bob_move:
        total_sequences += count_sequences(...)
return total_sequences % MOD  # Too late!
```

**Solution:**
Apply modulo operation during accumulation to prevent overflow:
```python
total_sequences = 0
for bob_move in range(3):
    if bob_move != last_bob_move:
        sequences = count_sequences(...)
        total_sequences = (total_sequences + sequences) % MOD
return total_sequences
```

### 4. **Incorrect Pruning Condition**
The pruning optimization can backfire if implemented incorrectly, causing valid winning sequences to be missed.

**Pitfall Example:**
```python
# WRONG: Too aggressive pruning
if remaining_rounds < score_difference:  # Should be <=
    return 0
```

**Solution:**
The correct pruning condition should check if Bob can't win even if he wins all remaining rounds:
```python
# Bob needs to overcome a deficit of 'score_difference'
# Each win gives him +2 relative points (he gains 1, Alice gains 0)
if remaining_rounds <= score_difference:
    return 0
```

### 5. **Off-by-One Error in Base Case**
Checking the wrong boundary condition can lead to missing the last round or processing beyond the sequence length.

**Pitfall Example:**
```python
# WRONG: Using wrong comparison
if position > len(s):  # Should be >=
    return int(score_difference < 0)
```

**Solution:**
Use the correct boundary check:
```python
if position >= len(s):  # Correct: position starts at 0
    return int(score_difference < 0)
```

