# 1349. Maximum Students Taking Exam

> Source: https://algo.monster/liteproblems/1349

## Problem Statement

You are given a `m × n` matrix `seats` representing the seat distribution in a classroom. Each seat is marked as:
- `'#'` if the seat is broken (cannot be used)
- `'.'` if the seat is in good condition (can be used)

Students taking an exam have a specific cheating pattern - they can see the answers of other students sitting:
- Directly to their left
- Directly to their right  
- Diagonally to their upper left
- Diagonally to their upper right

However, they cannot see the answers of students sitting directly in front or behind them.

Your task is to find the **maximum number of students** that can be seated in the classroom to take the exam such that no student can cheat from another. Students can only be placed in seats that are in good condition (marked with `'.'`).

For example, if we have a classroom layout:
```
. # . . # .
. . # . . .
```

We need to strategically place students so that:
1. No two students sit next to each other in the same row (left/right adjacency)
2. No student sits diagonally adjacent to another student in the row above (upper-left or upper-right positions)
3. Students are only placed in good seats (marked with `'.'`)

The goal is to maximize the total number of students that can take the exam under these constraints.

## Intuition

The key insight is that students in each row can only affect students in adjacent rows (the row above and below). Since students cannot see answers from those directly in front or behind them, we only need to worry about diagonal conflicts between consecutive rows.

This observation suggests we can process the classroom row by row. For each row, we need to decide which seats to fill while ensuring:
1. No two students sit adjacent in the same row
2. Students in the current row don't conflict with students in the previous row (diagonal positions)

Since each seat has only two states (occupied or empty), and the number of seats per row is relatively small (at most 8 based on typical constraints), we can use **bit manipulation** to represent seat arrangements. Each row's state can be encoded as a binary number where `1` means a student is seated and `0` means the seat is empty.

For example, if we have students in positions 0 and 3 of a row with 5 seats, we can represent this as `01001` (reading right to left in binary).

The problem now becomes: given the current row's available seats and the previous row's student arrangement, what's the maximum number of students we can place in all remaining rows?

This naturally leads to a **[dynamic programming](/problems/dynamic_programming_intro)** approach where:
- State: current row index and the student arrangement of the current row
- Decision: try all valid student arrangements for the current row
- Transition: move to the next row with updated constraints based on current placement

We can use **memoization** to avoid recalculating the same states. The recursive function `dfs(seat, i)` explores all valid ways to place students starting from row `i`, where `seat` represents which positions are available (considering both broken seats and diagonal conflicts from the previous row).

For each row, we enumerate all possible student arrangements (`mask`) and check:
- Students are only placed in available seats: `(seat | mask) == seat`
- No adjacent students in the same row: `!(mask & (mask << 1))`

The beauty of this approach is that it systematically explores all valid configurations while efficiently pruning invalid ones through bit operations.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) and [Bitmask](/problems/bitmask_intro) patterns.**

## Solution Approach

The solution uses **state compression with memoization search** to efficiently explore all valid seating arrangements.

### State Representation

First, we convert the classroom matrix into a compressed format. The function `f(seat)` transforms each row into a [bitmask](/problems/bitmask_intro) where:
- Bit position `i` is set to `1` if seat `i` is available (`.`)
- Bit position `i` is set to `0` if seat `i` is broken (`#`)

```python
def f(seat: List[str]) -> int:
    mask = 0
    for i, c in enumerate(seat):
        if c == '.':
            mask |= 1 << i
    return mask
```

This creates an array `ss` where `ss[i]` represents the available seats in row `i`.

### [Dynamic Programming](/problems/dynamic_programming_intro) with Memoization

The core algorithm uses a recursive function `dfs(seat, i)` that:
- `seat`: [bitmask](/problems/bitmask_intro) representing available positions in row `i` (considering broken seats and diagonal conflicts from previous row)
- `i`: current row index
- Returns: maximum students that can be seated from row `i` to the last row

### Enumerating Valid Arrangements

For each row, we try all possible student arrangements from `0` to `2^n - 1` (where `n` is the number of columns):

```python
for mask in range(1 << n):
```

### Validity Checks

For each arrangement `mask`, we verify:

1. **Students only in available seats**: `(seat | mask) != seat`
   - If this condition is true, skip this arrangement
   - This ensures we don't place students in broken seats or positions that conflict with the previous row

2. **No adjacent students**: `(mask & (mask << 1))`
   - If this is non-zero, students are sitting adjacent
   - `mask << 1` shifts all bits left by 1 position
   - The AND operation detects if any student has a neighbor to their left

### State Transition

If the arrangement is valid:

1. Count students in current row: `cnt = mask.bit_count()`

2. If it's the last row (`i == len(ss) - 1`):
   - Update answer: `ans = max(ans, cnt)`

3. Otherwise, prepare the next row:
   - Start with available seats: `nxt = ss[i + 1]`
   - Remove positions diagonally below current students:
     - `nxt &= ~(mask << 1)` (removes upper-right diagonal conflicts)
     - `nxt &= ~(mask >> 1)` (removes upper-left diagonal conflicts)
   - Recursively solve: `ans = max(ans, cnt + dfs(nxt, i + 1))`

### Memoization

The `@cache` decorator automatically memoizes the results of `dfs(seat, i)`, preventing redundant calculations when the same state is encountered again.

### Final Result

The algorithm starts with `dfs(ss[0], 0)`, exploring all valid arrangements starting from the first row with all initially available seats, and returns the maximum number of students that can be seated.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

Consider this 2×3 classroom:
```
. . #
# . .
```

**Step 1: Convert to bitmasks**

First, we convert each row to a bitmask (reading positions from right to left):
- Row 0: `. . #` → `011` (binary) = 3 (decimal)
  - Position 0: `.` → bit 0 = 1
  - Position 1: `.` → bit 1 = 1  
  - Position 2: `#` → bit 2 = 0
- Row 1: `# . .` → `110` (binary) = 6 (decimal)

So `ss = [3, 6]`

**Step 2: Start DFS from row 0**

Call `dfs(seat=3, i=0)` where seat=3 (`011`) represents available positions in row 0.

**Step 3: Try all possible arrangements for row 0**

We enumerate masks from 0 to 7 (2³-1):

- **mask=0** (`000`): No students placed
  - Valid (no adjacency, all positions available)
  - Count = 0
  - Next row's available seats: `ss[1] = 6` (`110`)
  - No diagonal conflicts to remove
  - Recursively solve: `0 + dfs(6, 1)`

- **mask=1** (`001`): Student at position 0
  - Check: `(3 | 1) = 3 == 3` ✓ (only using available seats)
  - Check: `(1 & 2) = 0` ✓ (no adjacency)
  - Count = 1
  - Next row available: `6 & ~(1<<1) & ~(1>>1) = 6 & ~2 & ~0 = 4` (`100`)
  - Removes position 1 from next row (diagonal conflict)
  - Recursively solve: `1 + dfs(4, 1)`

- **mask=2** (`010`): Student at position 1
  - Valid checks pass
  - Count = 1
  - Next row available: `6 & ~(2<<1) & ~(2>>1) = 6 & ~4 & ~1 = 2` (`010`)
  - Removes positions 0 and 2 from next row
  - Recursively solve: `1 + dfs(2, 1)`

- **mask=3** (`011`): Students at positions 0 and 1
  - Check: `(3 & 6) = 2` ✗ (adjacent students!)
  - Skip this arrangement

- **mask=4** (`100`): Student at position 2
  - Check: `(3 | 4) = 7 ≠ 3` ✗ (trying to use broken seat!)
  - Skip this arrangement

**Step 4: Process row 1 recursively**

For each valid arrangement from row 0, we solve row 1 (the last row).

For example, when row 0 has mask=1:
- Call `dfs(4, 1)` where seat=4 (`100`)
- Try mask=0: 0 students
- Try mask=4 (`100`): 1 student at position 2 (valid)
- Maximum for this path: 1 (from row 0) + 1 (from row 1) = 2

**Step 5: Combine results**

After exploring all paths:
- Path 1: No students in row 0, optimal arrangement in row 1 → total = 2
- Path 2: Student at position 0 in row 0, student at position 2 in row 1 → total = 2
- Path 3: Student at position 1 in row 0, student at position 1 in row 1 → total = 2

Maximum students = 2

One optimal arrangement:
```
X . #    (X = student placed)
# . X
```

This satisfies all constraints:
- No adjacent students in any row
- No diagonal conflicts between rows
- Students only in good seats

## Python Solution

```python
class Solution:
    def maxStudents(self, seats: List[List[str]]) -> int:
        def get_available_mask(row: List[str]) -> int:
            """
            Convert a row of seats to a bitmask where 1 means the seat is available ('.')
            and 0 means the seat is broken ('#')
            """
            mask = 0
            for position, seat in enumerate(row):
                if seat == '.':
                    mask |= 1 << position
            return mask

        @cache
        def dp(available_seats: int, row_index: int) -> int:
            """
            Dynamic programming function to find maximum students that can be seated
            from current row to the last row.
          
            Args:
                available_seats: Bitmask of available seats in current row
                row_index: Current row index
          
            Returns:
                Maximum number of students that can be seated from this row onwards
            """
            max_students = 0
          
            # Try all possible seating arrangements for current row
            for seating_mask in range(1 << num_cols):
                # Check if seating arrangement is valid:
                # 1. Students only sit in available seats (seating_mask is subset of available_seats)
                # 2. No two students sit adjacent to each other (no consecutive 1s in mask)
                if (available_seats | seating_mask) != available_seats or (seating_mask & (seating_mask << 1)):
                    continue
              
                students_in_row = seating_mask.bit_count()
              
                # Base case: last row
                if row_index == len(seat_masks) - 1:
                    max_students = max(max_students, students_in_row)
                else:
                    # Calculate available seats for next row considering current seating
                    # Students in current row block diagonal seats in next row
                    next_available = seat_masks[row_index + 1]
                    next_available &= ~(seating_mask << 1)  # Block left diagonal
                    next_available &= ~(seating_mask >> 1)  # Block right diagonal
                  
                    # Recursively solve for next row
                    max_students = max(max_students, students_in_row + dp(next_available, row_index + 1))
          
            return max_students

        num_cols = len(seats[0])
        # Convert each row to a bitmask representation
        seat_masks = [get_available_mask(row) for row in seats]
      
        # Start dynamic programming from first row
        return dp(seat_masks[0], 0)
```

## Time and Space Complexity

**Time Complexity:** `O(4^n × n × m)`

The analysis breaks down as follows:
- For each row (total `m` rows), we iterate through all possible mask values from `0` to `2^n - 1`, which gives us `2^n` iterations per recursive call
- For each mask, we check if it's valid by verifying:
  - `(seat | mask) != seat`: ensures students are only placed in available seats
  - `(mask & (mask << 1))`: ensures no two adjacent students in the same row
- The `bit_count()` operation takes `O(n)` time in the worst case
- Due to memoization with `@cache`, each unique state `(seat, i)` is computed only once
- The number of possible states for `seat` is at most `2^n` (all possible configurations of available/blocked seats after considering previous row's impact)
- There are `m` possible values for `i` (row index)
- Total states: `2^n × m`
- For each state, we try `2^n` masks, and each mask validation and bit counting takes `O(n)`
- Therefore: `O(2^n × m × 2^n × n) = O(4^n × n × m)`

**Space Complexity:** `O(2^n × m)`

The space usage comes from:
- The memoization cache stores results for each unique `(seat, i)` pair
- There are at most `2^n` different values for `seat` (representing different configurations of available seats after blocking positions based on previous row's students)
- There are `m` different values for `i` (row indices from 0 to m-1)
- The list `ss` uses `O(m)` space to store the seat configurations
- The recursion depth is at most `m`
- Total space: `O(2^n × m)` dominated by the cache storage

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Adjacency Check Logic**

One of the most common mistakes is incorrectly checking whether students are sitting adjacent to each other. The condition `(mask & (mask << 1))` only checks if a student has someone to their immediate left, but doesn't check for someone to their immediate right.

**Incorrect approach:**
```python
# Only checks left adjacency
if mask & (mask << 1):
    continue
```

**Why it fails:** Consider mask `0b110` (students in positions 1 and 2). The check `0b110 & 0b1100 = 0b100` is non-zero, correctly detecting adjacency. However, if we had mask `0b011` (students in positions 0 and 1), we need to ensure both left and right adjacencies are checked.

**Correct solution:**
The code actually handles this correctly because `mask & (mask << 1)` effectively checks if any bit has a neighbor to its left, which is sufficient. When bit `i` and bit `i+1` are both set, shifting left by 1 and ANDing will produce a non-zero result.

### 2. **Off-by-One Errors in Diagonal Blocking**

When blocking diagonal seats for the next row, it's easy to mix up the shift directions or forget one of the diagonals.

**Common mistake:**
```python
# Forgetting to block one diagonal or using wrong shift direction
next_available &= ~(seating_mask << 1)  # Only blocking one diagonal
```

**Correct approach:**
```python
next_available &= ~(seating_mask << 1)  # Block upper-left diagonal for next row
next_available &= ~(seating_mask >> 1)  # Block upper-right diagonal for next row
```

### 3. **Inefficient State Space Exploration**

Without memoization, the algorithm would repeatedly solve the same subproblems, leading to exponential time complexity.

**Pitfall:** Forgetting to use `@cache` or implementing manual memoization incorrectly:
```python
def dp(available_seats, row_index):  # Missing @cache decorator
    # ... recursive logic
```

**Solution:** Always use `@cache` decorator or implement proper memoization:
```python
@cache
def dp(available_seats, row_index):
    # ... recursive logic
```

### 4. **Incorrect Subset Check**

When verifying if students are only placed in available seats, the subset check can be confusing.

**Common mistake:**
```python
# Wrong subset check
if seating_mask & available_seats != seating_mask:
    continue
```

**Correct approach:**
```python
# Check if seating_mask is a subset of available_seats
if (available_seats | seating_mask) != available_seats:
    continue
```

The correct check uses OR operation: if `seating_mask` contains any bit that's not in `available_seats`, the OR result will differ from `available_seats`.

### 5. **Edge Case: Empty Classroom**

If all seats are broken or the classroom is empty, the algorithm should return 0. The current implementation handles this correctly, but it's important to verify:

**Potential issue:**
```python
# If seats is empty or all seats are broken
if not seats or all(all(seat == '#' for seat in row) for row in seats):
    return 0
```

The recursive approach naturally handles this because if no valid arrangements exist, the maximum will remain 0.

