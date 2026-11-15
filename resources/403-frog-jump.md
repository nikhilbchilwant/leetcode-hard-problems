# 403. Frog Jump

> Source: https://algo.monster/liteproblems/403

## Problem Statement

You have a frog that needs to cross a river by jumping on stones. The river is divided into units, and stones are placed at certain unit positions. The frog starts on the first stone and must reach the last stone without jumping into the water.

The key rules for the frog's movement are:

1. **Starting position**: The frog begins on the first stone (at position `stones[0]`)
2. **First jump**: The first jump must be exactly `1` unit
3. **Jump distance rule**: If the frog's last jump was `k` units, the next jump must be either:
   - `k - 1` units
   - `k` units  
   - `k + 1` units
4. **Direction**: The frog can only jump forward (to higher position values)
5. **Valid landing**: The frog must land on a stone - it cannot jump into water

Given an array `stones` containing the positions of stones in ascending order, you need to determine if the frog can successfully jump from the first stone to the last stone following these rules.

For example, if `stones = [0, 1, 3, 5, 6, 8, 12, 17]`, the frog can cross by following this path:
- Jump 1 unit from stone 0 to stone 1
- Jump 2 units from stone 1 to stone 3  
- Jump 2 units from stone 3 to stone 5
- Jump 3 units from stone 5 to stone 8
- Jump 4 units from stone 8 to stone 12
- Jump 5 units from stone 12 to stone 17

The function should return `true` if the frog can reach the last stone, and `false` otherwise.

## Intuition

The problem asks us to find if there's a valid path from the first stone to the last stone. At each stone, we have multiple choices for our next jump (based on the previous jump distance), which creates a branching decision tree. This naturally suggests a recursive exploration approach.

Think about standing on any stone during the journey. We need to know two things:
1. **Which stone we're currently on** (position `i`)
2. **How we got here** (the last jump distance `k`)

Why do we need the last jump distance? Because it determines our next possible moves. If we jumped `k` units to get here, we can only jump `k-1`, `k`, or `k+1` units next.

The key insight is that the same stone can be reached with different jump distances. For example, stone at position 5 could be reached by:
- Jumping 2 units from position 3
- Jumping 4 units from position 1

Each arrival creates a different state because the allowed next jumps are different. This means our state is defined by the pair `(current_stone_index, last_jump_distance)`.

Since we might visit the same state multiple times through different paths, we can use memoization to avoid recalculating whether a particular `(stone, jump)` combination can lead to the destination.

The hash table `pos` is used for quick lookups - given a position value, we can instantly find if there's a stone there and get its index. This is crucial because when we're at stone `i` and want to jump `j` units, we need to check if position `stones[i] + j` has a stone.

Starting from stone 0 with an initial jump of 0 (which forces the first actual jump to be 1), we recursively explore all possible paths until we either reach the last stone or exhaust all possibilities.

## Solution Approach

The solution uses **Dynamic Programming with Memoization** combined with **Depth-First Search (DFS)** to explore all possible jumping paths.

### Data Structures Used

1. **Hash Table (`pos`)**: Maps each stone position to its index in the array. This allows O(1) lookup to check if a target position has a stone.
   ```python
   pos = {s: i for i, s in enumerate(stones)}
   ```

2. **Memoization Cache**: The `@cache` decorator automatically memoizes the results of `dfs(i, k)` to avoid redundant calculations.

### Core Algorithm - DFS Function

The recursive function `dfs(i, k)` represents:
- `i`: Current stone index
- `k`: Last jump distance used to reach stone `i`
- Returns: `True` if we can reach the last stone from position `i`, `False` otherwise

### Implementation Steps

1. **Base Case**: If we're at the last stone (`i == n - 1`), we've successfully crossed the river, return `True`.

2. **Recursive Exploration**: For the current stone at index `i`, try all possible next jumps:
   - Generate jump distances: `j ∈ [k-1, k, k+1]`
   - For each valid jump distance `j`:
     - Check if `j > 0` (must be positive)
     - Calculate target position: `stones[i] + j`
     - Check if target position has a stone using `pos`
     - If stone exists, recursively check `dfs(pos[stones[i] + j], j)`
     - If any recursive call returns `True`, we've found a valid path

3. **Return False**: If no valid jump leads to success, return `False`.

### Starting Point

The algorithm starts with `dfs(0, 0)`:
- Start at stone index 0
- Initial `k = 0` ensures the first actual jump must be 1 (since next jumps can be `k-1=(-1)`, `k=0`, or `k+1=1`, only 1 is valid)

### Time and Space Complexity

- **Time Complexity**: O(n²) - At most n stones, and from each stone, we can have at most n different jump distances
- **Space Complexity**: O(n²) - For the memoization cache storing results for each `(stone_index, jump_distance)` pair

The memoization is crucial here - without it, the algorithm would explore the same states multiple times, leading to exponential time complexity.

## Example Walkthrough

Let's trace through a small example with `stones = [0, 1, 3, 6, 10]`.

**Initial Setup:**
- Create position map: `pos = {0:0, 1:1, 3:2, 6:3, 10:4}`
- Start with `dfs(0, 0)` (at stone index 0, last jump was 0)

**Step 1: From stone 0 (position 0), k=0**
- Possible jumps: k-1=-1 (invalid), k=0 (invalid), k+1=1 ✓
- Jump 1 unit → position 0+1=1 
- Stone exists at position 1 (index 1)
- Call `dfs(1, 1)`

**Step 2: From stone 1 (position 1), k=1**
- Possible jumps: k-1=0 (invalid), k=1 ✓, k+1=2 ✓
- Try jump 1: position 1+1=2 → no stone, skip
- Try jump 2: position 1+2=3 → stone exists (index 2)
- Call `dfs(2, 2)`

**Step 3: From stone 2 (position 3), k=2**
- Possible jumps: k-1=1 ✓, k=2 ✓, k+1=3 ✓
- Try jump 1: position 3+1=4 → no stone, skip
- Try jump 2: position 3+2=5 → no stone, skip  
- Try jump 3: position 3+3=6 → stone exists (index 3)
- Call `dfs(3, 3)`

**Step 4: From stone 3 (position 6), k=3**
- Possible jumps: k-1=2 ✓, k=3 ✓, k+1=4 ✓
- Try jump 2: position 6+2=8 → no stone, skip
- Try jump 3: position 6+3=9 → no stone, skip
- Try jump 4: position 6+4=10 → stone exists (index 4)
- Call `dfs(4, 4)`

**Step 5: At stone 4 (position 10), k=4**
- This is the last stone (index 4 = n-1)
- Return `True`

The recursion unwinds, returning `True` all the way back. The frog successfully crosses!

**Path taken:** 0 → 1 (jump 1) → 3 (jump 2) → 6 (jump 3) → 10 (jump 4)

Note how memoization helps: in larger inputs the same state `(stone index, last jump)` is often reached along different paths. After the first visit, `@cache` returns the stored result for that state instead of exploring it again. The jump size matters too: with `stones = [0, 1, 3, 6, 7]`, the frog reaches position 6 with a last jump of 3, and its next jump must be 2, 3, or 4, so the stone at 7 (one unit away) is unreachable and the answer is `False`.

## Python Solution

```python
class Solution:
    def canCross(self, stones: List[int]) -> bool:
        """
        Determine if a frog can cross the river by jumping on stones.
        The frog starts at the first stone and each jump must be k-1, k, or k+1 units,
        where k is the distance of the previous jump.
      
        Args:
            stones: List of stone positions in ascending order
          
        Returns:
            True if the frog can reach the last stone, False otherwise
        """
        from functools import cache
      
        @cache
        def dfs(current_index: int, last_jump_distance: int) -> bool:
            """
            Depth-first search to check if frog can reach the last stone.
          
            Args:
                current_index: Current stone index the frog is on
                last_jump_distance: Distance of the previous jump
              
            Returns:
                True if frog can reach the last stone from current position
            """
            # Base case: reached the last stone
            if current_index == num_stones - 1:
                return True
          
            # Try all possible jump distances: k-1, k, k+1
            for next_jump_distance in range(last_jump_distance - 1, last_jump_distance + 2):
                # Calculate the target position after jumping
                target_position = stones[current_index] + next_jump_distance
              
                # Check if jump is valid (distance > 0) and target position has a stone
                if next_jump_distance > 0 and target_position in stone_position_to_index:
                    # Recursively check if we can reach the end from the new position
                    target_index = stone_position_to_index[target_position]
                    if dfs(target_index, next_jump_distance):
                        return True
          
            # No valid path found from current position
            return False
      
        # Initialize variables
        num_stones = len(stones)
        # Create a mapping from stone position to its index for O(1) lookup
        stone_position_to_index = {position: index for index, position in enumerate(stones)}
      
        # Start DFS from first stone (index 0) with initial jump distance 0
        return dfs(0, 0)
```

## Time and Space Complexity

The time complexity is `O(n²)` where `n` is the number of stones.

**Time Complexity Analysis:**
- The algorithm uses memoized DFS with two parameters: position index `i` and jump distance `k`
- There are at most `n` possible positions (from 0 to n-1)
- For each position, the maximum jump distance `k` is bounded by `n` (in the worst case, the frog could jump at most `n` units to reach the last stone)
- Therefore, there are at most `O(n × n) = O(n²)` unique states in the memoization
- Each state is computed once due to the `@cache` decorator
- For each state, we explore at most 3 jump options (`k-1`, `k`, `k+1`), which is `O(1)`
- Total time complexity: `O(n²) × O(1) = O(n²)`

The space complexity is `O(n²)` where `n` is the number of stones.

**Space Complexity Analysis:**
- The `pos` dictionary stores `n` key-value pairs: `O(n)`
- The memoization cache stores at most `O(n²)` states (as analyzed above)
- The recursion stack depth is at most `n` in the worst case (jumping through all stones): `O(n)`
- Total space complexity: `O(n) + O(n²) + O(n) = O(n²)`

The dominant factor in both time and space complexity is the memoization cache which can store up to `n²` states.

## Common Pitfalls

### 1. **Incorrect Handling of the First Jump**

**Pitfall**: A common mistake is misunderstanding how to enforce that the first jump must be exactly 1 unit. Some implementations might:
- Start with `dfs(0, 1)` thinking the first jump is already 1
- Forget to validate that the first jump can only be 1 unit

**Why it's wrong**: Starting with `dfs(0, 1)` would mean the frog has already made a jump of 1 unit, which would allow the next jump to be 0, 1, or 2 units. This violates the constraint that the first actual jump must be exactly 1.

**Correct approach**: Start with `dfs(0, 0)`. When `k=0`, the possible next jumps are `k-1=-1` (invalid), `k=0` (invalid), and `k+1=1` (valid). This naturally enforces that the first jump must be 1 unit.

### 2. **Missing Edge Case: Second Stone Not at Position 1**

**Pitfall**: Not checking if it's even possible to make the first jump. If `stones = [0, 2, 3, 4, 8]`, the frog cannot make the first jump since position 1 has no stone.

**Solution**: Add an early termination check:
```python
# Early termination: if second stone is not at position 1, impossible to cross
if len(stones) > 1 and stones[1] != 1:
    return False
```

### 3. **Integer Overflow in Large Jump Calculations**

**Pitfall**: For very large stone positions, calculating `stones[i] + j` might cause issues in some languages (though Python handles big integers well).

**Solution**: Add bounds checking or use the constraint that jump distances are limited:
```python
# The maximum jump distance from any stone cannot exceed n (number of stones)
if next_jump_distance > num_stones:
    continue  # Skip impossibly large jumps
```

### 4. **Not Using Memoization or Using It Incorrectly**

**Pitfall**: Forgetting to memoize the DFS function or using incorrect memoization keys. Without memoization, the time complexity becomes exponential.

**Wrong approach**:
```python
# Wrong: only memoizing by current index
@cache
def dfs(current_index):  # Missing last_jump_distance parameter
    ...
```

**Why it's wrong**: The same stone can be reached with different previous jump distances, leading to different possible next jumps. The state must include both position AND how we got there.

**Correct approach**: Always include both `(current_index, last_jump_distance)` in the memoization key.

### 5. **Allowing Zero or Negative Jumps**

**Pitfall**: Forgetting to check `next_jump_distance > 0` before attempting a jump. This could lead to:
- Infinite loops (jumping 0 units stays at the same position)
- Invalid backward jumps (negative distances)

**Solution**: Always validate jump distance:
```python
if next_jump_distance > 0 and target_position in stone_position_to_index:
    # Process valid jump
```

### Complete Improved Solution with Pitfall Fixes:

```python
class Solution:
    def canCross(self, stones: List[int]) -> bool:
        from functools import cache
      
        # Early termination: impossible if first jump can't be made
        if len(stones) > 1 and stones[1] != 1:
            return False
      
        num_stones = len(stones)
        stone_position_to_index = {position: index for index, position in enumerate(stones)}
      
        @cache
        def dfs(current_index: int, last_jump_distance: int) -> bool:
            if current_index == num_stones - 1:
                return True
          
            for next_jump_distance in range(last_jump_distance - 1, last_jump_distance + 2):
                # Critical checks: positive jump and not exceeding reasonable bounds
                if next_jump_distance > 0 and next_jump_distance <= num_stones:
                    target_position = stones[current_index] + next_jump_distance
                  
                    if target_position in stone_position_to_index:
                        target_index = stone_position_to_index[target_position]
                        if dfs(target_index, next_jump_distance):
                            return True
          
            return False
      
        return dfs(0, 0)
```

