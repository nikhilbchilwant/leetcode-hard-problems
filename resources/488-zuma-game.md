# 488. Zuma Game

> Source: https://algo.monster/liteproblems/488

## Problem Statement

This problem is based on a variation of the game Zuma where you need to clear all colored balls from a board.

You have:
- A **board** represented as a string containing a row of colored balls, where each ball can be one of five colors: Red ('R'), Yellow ('Y'), Blue ('B'), Green ('G'), or White ('W')
- A **hand** represented as a string containing balls you can insert into the board

The game mechanics work as follows:

1. **Insert Phase**: On each turn, you pick any ball from your hand and insert it at any position in the board (between any two balls or at either end)

2. **Removal Phase**: After insertion, if there are **three or more consecutive balls of the same color**, they are automatically removed from the board. This removal can trigger a chain reaction - if removing a group causes another group of three or more same-colored balls to become adjacent, those are removed too. This continues until no more groups can be removed.

3. **Win Condition**: You win when the board becomes empty (all balls cleared)

4. **Game Continuation**: Keep inserting balls from your hand until either:
   - You clear the board (win), or
   - You run out of balls in your hand but the board still has balls (lose)

The task is to find the **minimum number of balls** you need to insert from your hand to clear the entire board. If it's impossible to clear the board with the balls you have, return `-1`.

For example:
- Board: `"WRRBBW"`, Hand: `"RB"` 
- You could insert 'R' between the two existing R's to make `"WRRRBW"`, which removes the three R's, leaving `"WBBW"`
- Then insert 'B' to make `"WBBBW"`, which removes the three B's, leaving `"WW"`
- The board still has balls but you're out of moves, so more insertions would be needed

## Intuition

The key insight is to think of this problem as exploring different game states to find the optimal sequence of moves. Each state represents a snapshot of the game: what balls are on the board and what balls remain in your hand.

Starting from the initial state, we want to try all possible moves and see which path leads to an empty board with the fewest insertions. This naturally suggests a state-space search problem.

Why BFS over other approaches? Consider these observations:

1. **Each move has equal cost**: Inserting any ball costs exactly 1 move, regardless of which ball or where we insert it. This makes it an unweighted graph problem.

2. **We need the minimum**: Among all possible ways to clear the board, we want the one using the fewest balls. BFS explores states level by level, guaranteeing that the first time we reach the winning state (empty board), we've used the minimum number of moves.

3. **State representation matters**: A state is uniquely defined by the current board configuration and remaining balls in hand. However, we can optimize by only tracking visited board states (not the hand), since if we've seen a board configuration before, we've already explored the optimal ways to clear it.

The chain reaction mechanism (removing groups of 3+ same-colored balls repeatedly) is handled by a helper function `remove()` that simulates the cascading removals after each insertion. This is crucial because one strategic insertion might trigger multiple removals, dramatically changing the board state.

The algorithm systematically tries:
- Each unique ball color from our hand (no need to try duplicate balls of the same color in the same round)
- Each possible insertion position on the board
- Applies the removal rules to get the resulting board state
- Continues exploration if this board state hasn't been visited before

By using BFS with this state exploration strategy, we ensure finding the shortest path (minimum insertions) from the initial state to the victory condition (empty board), or determine it's impossible if no such path exists.

## Solution Approach

The solution implements BFS using a queue and a visited set to track explored board states. Let's walk through the implementation details:

**1. Chain Removal Function (`remove`)**

The helper function handles the cascading removal of consecutive balls:
```python
def remove(s):
    while len(s):
        next = re.sub(r'B{3,}|G{3,}|R{3,}|W{3,}|Y{3,}', '', s)
        if len(next) == len(s):
            break
        s = next
    return s
```
- Uses regex to find and remove groups of 3 or more consecutive same-colored balls
- The pattern `r'B{3,}|G{3,}|R{3,}|W{3,}|Y{3,}'` matches 3+ consecutive occurrences of any color
- Continues removing until no more groups can be removed (when string length stops changing)

**2. BFS Initialization**

```python
visited = set()
q = deque([(board, hand)])
```
- `visited`: Set to track board states we've already explored (prevents infinite loops)
- `q`: Queue storing tuples of `(current_board_state, remaining_balls_in_hand)`
- Starts with the initial board and full hand

**3. BFS Main Loop**

```python
while q:
    state, balls = q.popleft()
    if not state:
        return len(hand) - len(balls)
```
- Dequeues states in FIFO order (ensuring level-by-level exploration)
- If we reach an empty board (`not state`), we've won!
- Returns the number of balls used: `len(hand) - len(balls)`

**4. Move Generation and Exploration**

```python
for ball in set(balls):
    b = balls.replace(ball, '', 1)
    for i in range(1, len(state) + 1):
        s = state[:i] + ball + state[i:]
        s = remove(s)
        if s not in visited:
            visited.add(s)
            q.append((s, b))
```

This nested loop structure generates all possible moves:
- **Outer loop**: Iterates through unique ball colors in hand using `set(balls)`
  - `balls.replace(ball, '', 1)` removes exactly one instance of the chosen ball
- **Inner loop**: Tries inserting the ball at every position (0 to len(state))
  - `state[:i] + ball + state[i:]` inserts the ball at position `i`
  - Applies `remove()` to handle chain reactions
  - If the resulting board state is new, adds it to visited and enqueues for exploration

**5. Termination**

```python
return -1
```
If the queue empties without finding an empty board state, it's impossible to clear the board with the given hand.

**Key Optimizations:**
- Using `set(balls)` avoids redundant attempts with duplicate balls in the same round
- Tracking only board states (not hand states) in `visited` reduces memory usage
- The regex-based removal is efficient for pattern matching and replacement

The algorithm explores O(n! × m^n) states in the worst case, where n is hand size and m is board positions, but the visited set pruning significantly reduces actual exploration.

## Example Walkthrough

Let's trace through a small example to see how the BFS solution works:

**Initial State:**
- Board: `"RBYYBBR"`
- Hand: `"YRBR"`

**Goal:** Clear the board using minimum insertions.

**BFS Exploration:**

**Round 1 (Initial state):**
- Queue: `[("RBYYBBR", "YRBR")]`
- We explore all possible first moves

Let's trace one promising path:

**Option 1A: Insert 'Y' at position 3**
- Take 'Y' from hand → Remaining hand: `"RBR"`
- Insert at position 3: `"RBY|Y|YBBR"` → `"RBYYYBBR"`
- Apply remove(): `"RBYYYBBR"` → `"RBBBR"` (3 Y's removed)
- Apply remove() again: `"RBBBR"` → `"RR"` (3 B's removed!)
- New state: `("RR", "RBR")`
- This triggers a chain reaction removing 6 balls with just 1 insertion!

**Round 2 (From state "RR"):**
- Current board: `"RR"`
- Remaining hand: `"RBR"`
- Try inserting 'R' between the two R's:
  - Insert at position 1: `"R|R|R"` → `"RRR"`
  - Apply remove(): `"RRR"` → `""` (3 R's removed)
  - Board is empty! Victory!

**Result:** Minimum insertions = 2

**Why BFS finds this optimal solution:**

1. **Level 1 exploration:** BFS tries all possible first insertions:
   - Insert each unique ball ('Y', 'R', 'B') at each position (0-7)
   - Many paths lead nowhere productive
   - The path inserting 'Y' at position 3 creates the chain reaction

2. **Level 2 exploration:** From each resulting state, try all second insertions:
   - The state `"RR"` reached by our path needs only one more move
   - Other paths might still have longer boards requiring more moves

3. **First success = optimal:** Since BFS explores level-by-level, finding an empty board at level 2 guarantees no solution exists using fewer insertions.

**Key observations from this example:**
- Strategic placement matters: Inserting 'Y' at position 3 (not 2 or 4) triggers the optimal chain
- Chain reactions are powerful: One insertion removed 6 balls
- The visited set prevents re-exploring `"RR"` if other paths reach it later
- Using `set(balls)` in Round 1 means we only try 'Y', 'R', 'B' once each (not 'R' twice)

## Python Solution

```python
import re
from collections import deque

class Solution:
    def findMinStep(self, board: str, hand: str) -> int:
        def remove_consecutive_balls(board_state):
            """
            Remove groups of 3 or more consecutive same-colored balls.
            Keep removing until no more groups can be removed.
            """
            while board_state:
                # Remove any sequence of 3 or more consecutive same-colored balls
                new_board = re.sub(r'B{3,}|G{3,}|R{3,}|W{3,}|Y{3,}', '', board_state)
              
                # If nothing was removed, we're done
                if len(new_board) == len(board_state):
                    break
                  
                board_state = new_board
          
            return board_state
      
        # Track visited board states to avoid revisiting
        visited_states = set()
      
        # BFS queue: each element is (current_board_state, remaining_balls_in_hand)
        queue = deque([(board, hand)])
      
        while queue:
            current_board, remaining_balls = queue.popleft()
          
            # If board is empty, we've won - return number of balls used
            if not current_board:
                return len(hand) - len(remaining_balls)
          
            # Try each unique ball color in our hand
            for ball_color in set(remaining_balls):
                # Remove one ball of this color from hand
                new_hand = remaining_balls.replace(ball_color, '', 1)
              
                # Try inserting the ball at each position in the board (including after last position)
                for insert_position in range(len(current_board) + 1):
                    # Insert ball at current position
                    new_board = current_board[:insert_position] + ball_color + current_board[insert_position:]
                  
                    # Remove any groups of 3+ consecutive balls
                    new_board = remove_consecutive_balls(new_board)
                  
                    # If we haven't seen this board state before, explore it
                    if new_board not in visited_states:
                        visited_states.add(new_board)
                        queue.append((new_board, new_hand))
      
        # If we exhausted all possibilities without clearing the board, return -1
        return -1
```

## Time and Space Complexity

**Time Complexity:** `O(m! * m * n^2 * k)`

Where:
- `n` is the length of the board string
- `m` is the length of the hand string (number of balls in hand)
- `k` is the average number of iterations needed for the `remove` function to stabilize

The analysis breaks down as follows:
- The BFS explores different states where we try placing balls from the hand into different positions on the board
- In the worst case, we can have up to `m!` different orderings of using the balls from the hand
- For each state, we iterate through unique balls in the hand (at most `m` balls)
- For each ball, we try inserting it at `n+1` possible positions in the board
- After each insertion, the `remove` function is called which takes `O(n*k)` time, where `k` iterations are needed for the regex substitution to stabilize (typically small constant)
- The regex substitution itself takes `O(n)` per iteration
- Overall: `O(m! * m * n * n * k) = O(m! * m * n^2 * k)`

**Space Complexity:** `O(m! * n^2)`

The space complexity is determined by:
- The `visited` set which can store up to `O(m! * n)` different board states (different combinations of balls placed at different positions)
- The BFS queue which in the worst case can hold `O(m! * n)` states
- Each state string takes `O(n)` space
- The recursive call stack is not present here as the solution uses iterative BFS
- Total space: `O(m! * n * n) = O(m! * n^2)`

## Common Pitfalls

### 1. **Keying the Visited Set on the Board Only**
The solution stores only the board string in `visited_states`. Using `set(remaining_balls)` already prevents trying two balls of the same color from the same state, so duplicate balls are not the issue. The subtle point is that two different paths can reach the same board with different balls left in hand. The board-only key keeps whichever arrives first and discards the other, even though the discarded hand might be the one that can finish the board.

Both paths are at the same BFS level, so the two states use the same number of balls, but their remaining hands differ. On random tests this board-only key gives the same answers as a full-state search, yet nothing in the code guarantees that, so keying on the full state is the safe choice. It costs more memory but never discards a reachable state.

**Solution:**
Track states as `(board, sorted_hand)` pairs instead of just board states:

```python
visited_states = set()
queue = deque([(board, hand)])

while queue:
    current_board, remaining_balls = queue.popleft()
  
    # Create a canonical representation of the state
    state_key = (current_board, ''.join(sorted(remaining_balls)))
    if state_key in visited_states:
        continue
    visited_states.add(state_key)
  
    if not current_board:
        return len(hand) - len(remaining_balls)
  
    # rest of the code...
```

### 2. **Pruning Insertion Positions Too Aggressively**
The code tries every position from 0 to `len(current_board)`. A tempting optimization is to skip positions between two identical balls when the inserted ball has a different color, on the assumption that splitting a pair never helps.

```python
# INCORRECT pruning
if (0 < insert_position < len(current_board) and
        current_board[insert_position - 1] == current_board[insert_position] and
        current_board[insert_position] != ball_color):
    continue
```

**The Problem:**
Splitting a pair can be the only way to win. For `board = "RRWWRRBBRR"` and `hand = "WB"`, the answer is 2: insert 'W' between the last two R's to get `"RRWWRRBBRWR"`, then insert 'B' to clear `BBB`, which joins `RR` and `R` into `RRR`, and the chain clears the whole board. With the pruning above, that first move is skipped and the search returns -1.

**Safe Alternative:**
Skip only insertions that produce exactly the same string as another insertion. Inserting a ball right after a ball of the same color gives the same board as inserting it right before that ball:

```python
for insert_position in range(len(current_board) + 1):
    # Same string as inserting at insert_position - 1
    if insert_position > 0 and current_board[insert_position - 1] == ball_color:
        continue
    new_board = current_board[:insert_position] + ball_color + current_board[insert_position:]
    # rest of the code...
```

### 3. **Memory Explosion with Large Inputs**
The visited set can grow exponentially with board and hand size, potentially causing memory issues.

**The Problem:**
Each unique board configuration is stored, and with many possible intermediate states, memory usage can balloon quickly.

**Solution:**
Implement state pruning by limiting exploration depth or using iterative deepening:

```python
def findMinStep(self, board: str, hand: str) -> int:
    # Try with increasing depth limits
    for max_depth in range(1, len(hand) + 1):
        result = self.bfs_with_depth_limit(board, hand, max_depth)
        if result != -1:
            return result
    return -1

def bfs_with_depth_limit(self, board, hand, max_depth):
    visited_states = set()
    queue = deque([(board, hand, 0)])  # Added depth counter
  
    while queue:
        current_board, remaining_balls, depth = queue.popleft()
      
        if depth > max_depth:
            continue
          
        if not current_board:
            return depth
      
        # Continue with normal BFS logic...
```

### 4. **Regex Performance Issues**
The regex pattern matching can be slow for very long strings or when called repeatedly.

**The Problem:**
Regex compilation happens on every call to `remove_consecutive_balls`, and pattern matching can be expensive.

**Solution:**
Pre-compile the regex pattern and consider a non-regex approach for better performance:

```python
# Pre-compile regex (class-level or module-level)
REMOVAL_PATTERN = re.compile(r'B{3,}|G{3,}|R{3,}|W{3,}|Y{3,}')

def remove_consecutive_balls(board_state):
    while board_state:
        new_board = REMOVAL_PATTERN.sub('', board_state)
        if len(new_board) == len(board_state):
            break
        board_state = new_board
    return board_state

# Alternative non-regex approach
def remove_consecutive_balls_fast(board_state):
    while True:
        result = []
        i = 0
        changed = False
      
        while i < len(board_state):
            j = i
            while j < len(board_state) and board_state[j] == board_state[i]:
                j += 1
          
            if j - i < 3:
                result.extend(board_state[i:j])
            else:
                changed = True
          
            i = j
      
        board_state = ''.join(result)
        if not changed:
            break
  
    return board_state
```

These optimizations can significantly improve both time and space complexity, especially for larger inputs where the state space explosion becomes problematic.

