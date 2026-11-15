# 913. Cat and Mouse

> Source: https://algo.monster/liteproblems/913

## Problem Statement

This is a game theory problem where a Mouse and a Cat play a turn-based game on an undirected [graph](/problems/graph_intro).

**Game Setup:**
- The game is played on an undirected [graph](/problems/graph_intro) represented as `graph[a]`, which contains a list of all nodes `b` such that there's an edge between node `a` and node `b`
- Mouse starts at node `1` and moves first
- Cat starts at node `2` and moves second  
- There's a hole at node `0`
- Players alternate turns

**Movement Rules:**
- On each turn, a player **must** move along one edge from their current position to an adjacent node
- For example, if Mouse is at node `1`, it must move to one of the nodes in `[graph](/problems/graph_intro)[1]`
- **Important restriction:** The Cat cannot move to node `0` (the hole)

**Winning Conditions:**
The game can end in three ways:
1. **Cat wins:** If the Cat ever occupies the same node as the Mouse (catches the Mouse)
2. **Mouse wins:** If the Mouse reaches node `0` (the hole)
3. **Draw:** If the same game state repeats (same positions for both players with the same player's turn to move)

**Problem Task:**
Given the [graph](/problems/graph_intro), and assuming both players play optimally (make the best possible moves), determine the outcome of the game:
- Return `1` if the Mouse wins
- Return `2` if the Cat wins
- Return `0` if the game ends in a draw

The key challenge is to analyze all possible game states and determine the optimal strategy for both players, considering that each player will make the best move possible to achieve their winning condition.

## Intuition

To solve this game theory problem, we need to think backwards from the end states. We know for certain who wins in specific situations - when the Mouse reaches the hole (Mouse wins) or when the Cat catches the Mouse (Cat wins). These are our "boundary states" with known outcomes.

The key insight is that we can work backwards from these known winning/losing states to determine the outcome of earlier game states. If we know the outcomes of all possible next states from a given position, we can determine the outcome of the current position.

Think of it like this: if it's the Mouse's turn and the Mouse can move to at least one position where the Mouse wins, then the Mouse will choose that move (playing optimally). Conversely, if all possible moves lead to Cat winning, then the current position is a losing position for the Mouse.

This naturally leads us to a **reverse topological traversal** approach. We start from the known end states and propagate the results backwards through the game state [graph](/problems/graph_intro). Each game state is represented by three components: `(mouse_position, cat_position, whose_turn)`.

The challenge is handling the "degree" concept - we need to know when all possible moves from a state lead to a loss for the current player. We track this by counting how many moves are available from each state. When we find a losing move for a player, we decrease the degree. If the degree reaches 0, it means all moves lead to a loss, so the current state is a losing state for that player.

The draw condition is handled implicitly - if a state doesn't get marked as a win for either player after we've processed all states, it must be a draw. This happens when neither player can force a win from that position.

By using BFS from the boundary states and carefully tracking which states lead to wins/losses, we can determine the outcome of the initial state `(1, 2, MOUSE_TURN)` - which is exactly what the problem asks for.

**Learn more about [Graph](/problems/graph_intro), [Topological Sort](/problems/topo_intro), [Memoization](/problems/memoization_intro), [Math](/problems/math-basics) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution implements a **topological sorting approach using BFS** to determine the game outcome by working backwards from known end states.

**State Representation:**
- Each game state is represented as `(mouse_position, cat_position, turn)`
- `turn` is either `MOUSE_TURN (0)` or `CAT_TURN (1)`
- Game outcomes are represented as `MOUSE_WIN (1)`, `CAT_WIN (2)`, or `TIE (0)`

**Data Structures:**
1. **Answer Array:** `ans[m][c][t]` - 3D array storing the outcome for each state
2. **Degree Array:** `degree[m][c][t]` - tracks the number of possible moves from each state
3. **Queue:** Used for BFS traversal of states

**Algorithm Steps:**

1. **Initialize Degree Array:**
   - For each state `(i, j, turn)`:
     - If it's Mouse's turn: `degree[i][j][MOUSE_TURN] = len([graph](/problems/graph_intro)[i])` (number of Mouse's adjacent nodes)
     - If it's Cat's turn: `degree[i][j][CAT_TURN] = len(graph[j])` (number of Cat's adjacent nodes)
   - Special case: Subtract 1 from Cat's degree if the hole (node 0) is adjacent, since Cat cannot move there

2. **Initialize Boundary States:**
   - **Mouse wins:** When Mouse reaches hole: `ans[0][j][turn] = MOUSE_WIN` for all `j` and both turns
   - **Cat wins:** When Cat catches Mouse: `ans[i][i][turn] = CAT_WIN` for all `i` and both turns
   - Add all boundary states to the queue

3. **BFS Propagation:**
   - Process each state from the queue
   - For each current state, find all possible previous states using `get_prev_states()`:
     - If current turn is Mouse's, previous turn was Cat's: Cat could have been at any adjacent position
     - If current turn is Cat's, previous turn was Mouse's: Mouse could have been at any adjacent position

4. **Update Previous States:**
   - For each previous state `(pm, pc, pt)`:
     - If already determined (`ans[pm][pc][pt] != TIE`), skip
     - **Immediate win condition:** If the player in the previous turn matches the winner of current state (e.g., Mouse's turn and current state is Mouse win), mark previous state as a win for that player
     - **All moves lose condition:** Otherwise, decrement the degree. If `degree[pm][pc][pt]` reaches 0, all moves from that state lead to loss, so mark it as a win for the opponent

5. **Return Result:**
   - After processing all states, return `ans[MOUSE_START][CAT_START][MOUSE_TURN]`
   - This gives the outcome when Mouse starts at node 1, Cat at node 2, and it's Mouse's turn

The algorithm ensures that all reachable states are evaluated, and states that remain unmarked after the BFS are draws (neither player can force a win).

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

Consider a simple graph with 4 nodes:
```
    0 (hole)
   / \
  1---3
   \ /
    2
```

Graph representation: `graph = [[1,3], [0,2,3], [1,3], [0,1,2]]`
- Node 0 connects to nodes 1 and 3
- Node 1 connects to nodes 0, 2, and 3  
- Node 2 connects to nodes 1 and 3
- Node 3 connects to nodes 0, 1, and 2

Initial setup: Mouse at node 1, Cat at node 2, Mouse moves first.

**Step 1: Initialize Degree Array**

For each state (mouse_pos, cat_pos, turn), count possible moves:
- State (1, 2, MOUSE_TURN): Mouse at 1 can move to [0,2,3] → degree = 3
- State (1, 2, CAT_TURN): Cat at 2 can move to [1,3] → degree = 2 (node 2 is not adjacent to the hole)
- State (3, 2, MOUSE_TURN): Mouse at 3 can move to [0,1,2] → degree = 3
- And so on for all states...

**Step 2: Initialize Boundary States**

Mouse wins (reaches hole):
- (0, 1, MOUSE_TURN) = MOUSE_WIN
- (0, 1, CAT_TURN) = MOUSE_WIN
- (0, 2, MOUSE_TURN) = MOUSE_WIN
- (0, 2, CAT_TURN) = MOUSE_WIN
- (0, 3, MOUSE_TURN) = MOUSE_WIN
- (0, 3, CAT_TURN) = MOUSE_WIN

Cat wins (catches Mouse):
- (1, 1, MOUSE_TURN) = CAT_WIN
- (1, 1, CAT_TURN) = CAT_WIN
- (2, 2, MOUSE_TURN) = CAT_WIN
- (2, 2, CAT_TURN) = CAT_WIN
- (3, 3, MOUSE_TURN) = CAT_WIN
- (3, 3, CAT_TURN) = CAT_WIN

**Step 3: BFS Propagation**

A state's turn tells who moves next, so the previous state belongs to the other player. For a state with `CAT_TURN`, the Mouse just moved; for a state with `MOUSE_TURN`, the Cat just moved.

Process boundary state (0, 2, CAT_TURN) = MOUSE_WIN:
- The Mouse just moved into the hole from a neighbor of node 0, so the previous states are (1, 2, MOUSE_TURN) and (3, 2, MOUSE_TURN)
- Both are Mouse turns and the current state is a Mouse win, so the Mouse can choose this move: mark (1, 2, MOUSE_TURN) = MOUSE_WIN and (3, 2, MOUSE_TURN) = MOUSE_WIN and add them to the queue

This already settles the starting state (1, 2, MOUSE_TURN).

Process boundary state (1, 1, MOUSE_TURN) = CAT_WIN:
- The Cat just moved onto node 1 from a neighbor of node 1 other than the hole, so the previous states are (1, 2, CAT_TURN) and (1, 3, CAT_TURN)
- Both are Cat turns and the current state is a Cat win, so mark both as CAT_WIN and add them to the queue

**Step 4: Continue Processing**

The BFS keeps popping states and repeating the same two rules. When the current result is a win for the player who moved into it, the previous state is marked immediately. Otherwise, the previous state's degree is decreased by 1, and the state is marked as a loss for its player only when the degree reaches 0. For example, (2, 3, MOUSE_TURN) and (2, 1, MOUSE_TURN) are eventually marked CAT_WIN this way, because every move of the Mouse from node 2 (to node 1 or node 3) leads to a state the Cat wins.

**Step 5: Read the Initial State**

The starting state (1, 2, MOUSE_TURN) was marked MOUSE_WIN in Step 3, because the Mouse can move straight from node 1 to the hole at node 0.

The algorithm returns 1, indicating the Mouse wins with optimal play.

## Python Solution

```python
from collections import deque
from typing import List

# Game constants
HOLE, MOUSE_START, CAT_START = 0, 1, 2
MOUSE_TURN, CAT_TURN = 0, 1
MOUSE_WIN, CAT_WIN, TIE = 1, 2, 0


class Solution:
    def catMouseGame(self, graph: List[List[int]]) -> int:
        """
        Determines the outcome of the cat and mouse game.
      
        Args:
            graph: Adjacency list representation of the game board
          
        Returns:
            0 if tie, 1 if mouse wins, 2 if cat wins
        """
      
        def get_previous_states(current_state: tuple) -> List[tuple]:
            """
            Get all possible previous states that could lead to current state.
          
            Args:
                current_state: (mouse_pos, cat_pos, turn)
              
            Returns:
                List of previous states (mouse_pos, cat_pos, turn)
            """
            mouse_pos, cat_pos, turn = current_state
            previous_turn = turn ^ 1  # Toggle turn using XOR
            previous_states = []
          
            if previous_turn == CAT_TURN:
                # If it was cat's turn, cat could have moved from any adjacent position
                for previous_cat_pos in graph[cat_pos]:
                    if previous_cat_pos != HOLE:  # Cat cannot be at hole
                        previous_states.append((mouse_pos, previous_cat_pos, previous_turn))
            else:
                # If it was mouse's turn, mouse could have moved from any adjacent position
                for previous_mouse_pos in graph[mouse_pos]:
                    previous_states.append((previous_mouse_pos, cat_pos, previous_turn))
          
            return previous_states
      
        n = len(graph)
      
        # Initialize result array: result[mouse][cat][turn] = game outcome
        result = [[[0, 0] for _ in range(n)] for _ in range(n)]
      
        # Initialize degree array: tracks number of possible moves from each state
        degree = [[[0, 0] for _ in range(n)] for _ in range(n)]
      
        # Calculate degrees for each state
        for mouse_pos in range(n):
            for cat_pos in range(1, n):  # Cat cannot be at position 0 (hole)
                degree[mouse_pos][cat_pos][MOUSE_TURN] = len(graph[mouse_pos])
                degree[mouse_pos][cat_pos][CAT_TURN] = len(graph[cat_pos])
          
            # Adjust cat's degree since it cannot move to hole
            for adjacent_to_hole in graph[HOLE]:
                degree[mouse_pos][adjacent_to_hole][CAT_TURN] -= 1
      
        # BFS queue for processing states
        queue = deque()
      
        # Initialize known winning states: mouse at hole
        for cat_pos in range(1, n):
            result[0][cat_pos][MOUSE_TURN] = MOUSE_WIN
            result[0][cat_pos][CAT_TURN] = MOUSE_WIN
            queue.append((0, cat_pos, MOUSE_TURN))
            queue.append((0, cat_pos, CAT_TURN))
      
        # Initialize known winning states: cat catches mouse
        for pos in range(1, n):
            result[pos][pos][MOUSE_TURN] = CAT_WIN
            result[pos][pos][CAT_TURN] = CAT_WIN
            queue.append((pos, pos, MOUSE_TURN))
            queue.append((pos, pos, CAT_TURN))
      
        # Process states using BFS
        while queue:
            current_state = queue.popleft()
            current_mouse, current_cat, current_turn = current_state
            current_result = result[current_mouse][current_cat][current_turn]
          
            # Check all states that could lead to current state
            for previous_state in get_previous_states(current_state):
                prev_mouse, prev_cat, prev_turn = previous_state
              
                # Skip if this previous state already has a determined outcome
                if result[prev_mouse][prev_cat][prev_turn] != TIE:
                    continue
              
                # Check if previous player can force a win
                is_winning_move = (
                    (current_result == MOUSE_WIN and prev_turn == MOUSE_TURN) or
                    (current_result == CAT_WIN and prev_turn == CAT_TURN)
                )
              
                if is_winning_move:
                    # Previous player found a winning move
                    result[prev_mouse][prev_cat][prev_turn] = current_result
                    queue.append(previous_state)
                else:
                    # Decrease the degree (number of unexplored moves)
                    degree[prev_mouse][prev_cat][prev_turn] -= 1
                  
                    # If all moves lead to loss, previous player loses
                    if degree[prev_mouse][prev_cat][prev_turn] == 0:
                        result[prev_mouse][prev_cat][prev_turn] = current_result
                        queue.append(previous_state)
      
        # Return the result for initial state
        return result[MOUSE_START][CAT_START][MOUSE_TURN]
```

## Time and Space Complexity

**Time Complexity: `O(n^3)`**

The algorithm uses BFS to traverse game states. Each state is represented by `(mouse_position, cat_position, turn)`, giving us `n × n × 2` possible states (mouse can be at any of `n` positions, cat can be at any of `n` positions, and there are 2 possible turns). 

For each state processed in the queue, we examine its previous states through `get_prev_states()`:
- When looking at previous cat turns, we iterate through all neighbors of the current cat position (at most `n-1` neighbors)
- When looking at previous mouse turns, we iterate through all neighbors of the current mouse position (at most `n` neighbors)

Since each state can be added to the queue at most once (when its outcome is determined), and for each state we do `O(n)` work to process previous states, the total time complexity is `O(n^2 × 2 × n) = O(n^3)`.

**Space Complexity: `O(n^2)`**

The algorithm uses several 3D arrays:
- `ans[n][n][2]`: stores the outcome for each state, requiring `O(n^2)` space
- `degree[n][n][2]`: stores the out-degree for each state, requiring `O(n^2)` space
- The queue can contain at most `O(n^2)` states since each state is added at most once

The total space complexity is `O(n^2)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Handling of Cat's Movement Restriction at Node 0**

**The Pitfall:**
A common mistake is forgetting to properly handle the restriction that the Cat cannot move to node 0 (the hole). This affects two critical parts of the algorithm:

1. **Degree Calculation:** When calculating the number of possible moves for the Cat, developers often use `degree[m][c][CAT_TURN] = len(graph[c])` without accounting for the fact that if node 0 is adjacent to the Cat's position, it should not be counted as a valid move.

2. **Previous State Generation:** When generating previous states where the Cat could have come from, developers might include node 0 as a possible previous position for the Cat.

**Incorrect Implementation Example:**
```python
# WRONG: Not adjusting degree for cat's restriction
for mouse_pos in range(n):
    for cat_pos in range(1, n):
        degree[mouse_pos][cat_pos][CAT_TURN] = len(graph[cat_pos])  # Includes node 0!

# WRONG: Not filtering out node 0 in previous states
if previous_turn == CAT_TURN:
    for previous_cat_pos in graph[cat_pos]:
        # Missing check for HOLE!
        previous_states.append((mouse_pos, previous_cat_pos, previous_turn))
```

**Correct Solution:**
```python
# Correctly adjust degree by subtracting 1 if node 0 is adjacent
for adjacent_to_hole in graph[HOLE]:
    degree[mouse_pos][adjacent_to_hole][CAT_TURN] -= 1

# Correctly filter out node 0 when generating previous cat positions
if previous_turn == CAT_TURN:
    for previous_cat_pos in graph[cat_pos]:
        if previous_cat_pos != HOLE:  # Essential check!
            previous_states.append((mouse_pos, previous_cat_pos, previous_turn))
```

### 2. **Misunderstanding the Win Condition Logic in BFS**

**The Pitfall:**
Another common error is incorrectly determining when a previous state should inherit the current state's outcome. The logic is subtle:

- If the current state is a win for a player AND it was that player's turn in the previous state, then the previous state is also a win for that player (they can choose to move to this winning position).
- Otherwise, we decrement the degree, and only when ALL moves lead to losses (degree becomes 0) do we mark it as a loss for that player.

**Incorrect Implementation Example:**
```python
# WRONG: Oversimplified win condition check
if current_result == MOUSE_WIN:
    result[prev_mouse][prev_cat][prev_turn] = MOUSE_WIN
    queue.append(previous_state)
```

**Correct Solution:**
```python
# Check if the player whose turn it was can force a win
is_winning_move = (
    (current_result == MOUSE_WIN and prev_turn == MOUSE_TURN) or
    (current_result == CAT_WIN and prev_turn == CAT_TURN)
)

if is_winning_move:
    result[prev_mouse][prev_cat][prev_turn] = current_result
    queue.append(previous_state)
else:
    degree[prev_mouse][prev_cat][prev_turn] -= 1
    if degree[prev_mouse][prev_cat][prev_turn] == 0:
        # All moves lead to loss for the player whose turn it was
        result[prev_mouse][prev_cat][prev_turn] = current_result
        queue.append(previous_state)
```

### 3. **Initializing States Where Cat is at Node 0**

**The Pitfall:**
Since the Cat can never be at node 0, states with `cat_pos = 0` are meaningless. Looping over them in the degree calculation is harmless on its own (those states are never generated as previous states, so their degrees are never read), but it does wasted work. It becomes a real bug if `cat_pos = 0` also enters the boundary initialization, for example a Cat-win loop `for pos in range(n)` would mark `(0, 0, turn)` as `CAT_WIN`, overwriting the Mouse win at the hole.

**Incorrect Implementation Example:**
```python
# Wasteful: includes cat_pos = 0, a state that can never occur
for mouse_pos in range(n):
    for cat_pos in range(n):  # Start from 1 instead
        degree[mouse_pos][cat_pos][MOUSE_TURN] = len(graph[mouse_pos])
```

**Correct Solution:**
```python
# Correctly skip cat_pos = 0
for mouse_pos in range(n):
    for cat_pos in range(1, n):  # Start from 1, not 0
        degree[mouse_pos][cat_pos][MOUSE_TURN] = len(graph[mouse_pos])
        degree[mouse_pos][cat_pos][CAT_TURN] = len(graph[cat_pos])
```

The first two pitfalls lead to incorrect game outcomes because they either allow invalid moves or incorrectly propagate win/loss states through the game tree.

