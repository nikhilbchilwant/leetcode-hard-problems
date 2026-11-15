# 3548. Equal Sum Grid Partition II

> Source: https://algo.monster/liteproblems/3548

## Problem Statement

You are given an `m x n` matrix `grid` filled with positive integers. Your goal is to figure out whether you can make a **single straight cut** on the grid—either **one horizontal cut** or **one vertical cut**—so that the grid is divided into two pieces satisfying a set of conditions.

When you make a cut, think of it as drawing a line that completely separates the grid into two parts:

- A **horizontal cut** splits the grid between two rows, producing a **top section** and a **bottom section**.
- A **vertical cut** splits the grid between two columns, producing a **left section** and a **right section**.

After making the cut, the two resulting sections must satisfy all of the following rules:

1. **Both sections must be non-empty.** This means you cannot cut along the very edge of the grid; each side must contain at least one full row (for a horizontal cut) or at least one full column (for a vertical cut).

2. **The sums must match (with one allowed exception).** Let the sum of all elements in one section be compared with the sum of all elements in the other section. The cut is valid if either:
   - The two sums are already **equal**, or
   - You are allowed to **discount at most one single cell** (from either of the two sections) so that the remaining sums become equal. In other words, you may remove the value of one cell from one of the sections, and after that removal, the two sums should be equal.

3. **Connectivity after discounting.** If you choose to discount a cell, the section it was removed from must **stay connected**. A section is connected when every remaining cell can be reached from any other remaining cell by moving up, down, left, or right through cells that are still in that section. Removing a cell from a corner or edge typically keeps the section connected, but removing an interior cell could break it apart, which is not allowed.

Your task is to return `true` if there exists at least one valid cut (horizontal or vertical) that meets all of the above conditions, and `false` otherwise.

**Note:** The discount counts as **at most one cell in total**, meaning across both sections combined you may skip the value of a single cell, not one per section.

## Intuition

The key observation is that a cut is always a **full straight line**—it goes all the way across the grid. This means the number of possible cuts is small: for a horizontal cut, we only have `m - 1` places to cut (between each pair of adjacent rows), and for a vertical cut, we only have `n - 1` places. So instead of guessing, we can simply **try every possible cut** and check whether any of them works.

Let's focus on horizontal cuts first. Imagine sweeping a cut line from top to bottom. As the line moves down by one row, the top section grows by that row, and the bottom section shrinks by that row. This suggests we can keep a **running sum** of the two parts: start with everything in the bottom, then move rows one at a time into the top. At each step we know `s1` (top sum) and `s2` (bottom sum) instantly, without recomputing from scratch.

Now, for each cut position we need to answer the partition question. There are two easy cases and one tricky case:

- If `s1 == s2`, the two halves already match, so we return `true` right away.
- If they differ, let `diff` be the absolute difference. To fix this gap by discounting **one cell**, that cell must come from the **larger** section, and its value must be exactly `diff`. Removing a cell of value `diff` from the bigger side lowers its sum by `diff`, making the two sides equal.

To check "does the larger section contain a cell with value `diff`?" quickly, we maintain two **hash maps** (`cnt1` and `cnt2`) that count how many times each value appears in the top and bottom parts. As rows move from bottom to top, we update both counts. This lets us test for the existence of a `diff`-valued cell in `O(1)`.

But existence alone isn't enough—rule 3 demands the section stays **connected** after the removal. A cell can be safely removed without breaking connectivity only when:

- The section has **more than one row and more than one column**, so it's a "thick" rectangle. Removing any single interior or edge cell from such a block never disconnects it (the surrounding cells still link up).
- The section is a **single row**, and the removed cell sits at one of the two **ends** (first or last column). Removing an end keeps the rest in one piece, but removing a middle cell would split the row into two.
- The section is a **single column** (width `1`), and the removed cell is at the **top or bottom end**. Same reasoning as the single-row case.

These conditions are exactly what the code checks when it finds a `diff`-valued cell: it verifies the shape of the larger section and where a removable cell could sit.

Finally, to handle **vertical cuts** without writing a second nearly identical function, we use a clever trick: **transpose the grid** (swap rows and columns). A vertical cut on the original grid becomes a horizontal cut on the transposed grid. So we just run the same `check` function twice—once on `grid` and once on its transpose `zip(*grid)`—and return `true` if either one succeeds.

**Learn more about [Prefix Sum](/problems/subarray_sum) patterns.**

## Solution Approach

The implementation follows the **enumerate partition lines** method, combined with **prefix sums** and **hash maps** for fast lookups. Let's walk through it piece by piece.

### The `check` helper

The whole logic for horizontal cuts is wrapped in a helper function `check(g)`. Since vertical cuts can be turned into horizontal cuts by transposing, this single function handles both cases.

```python
def check(g: List[List[int]]) -> bool:
    m, n = len(g), len(g[0])
    s1 = s2 = 0
    cnt1 = defaultdict(int)
    cnt2 = defaultdict(int)
```

- `s1` and `s2` are the running sums of the **top** and **bottom** sections.
- `cnt1` and `cnt2` are hash maps counting how many times each value appears in the top and bottom sections, used for `O(1)` existence checks.

### Initializing the bottom section

We start with **everything in the bottom**, so `s1 = 0` and `s2` holds the total of all cells. We also fill `cnt2` with the count of every value:

```python
for i, row in enumerate(g):
    for j, x in enumerate(row):
        s2 += x
        cnt2[x] += 1
```

### Sweeping the cut line downward

Now we move rows one at a time from the bottom into the top. We only iterate over the first `m - 1` rows (`g[: m - 1]`), because the cut must leave **at least one row** in the bottom—both sections must be non-empty.

```python
for i, row in enumerate(g[: m - 1]):
    for x in row:
        s1 += x
        s2 -= x
        cnt1[x] += 1
        cnt2[x] -= 1
```

After processing row `i`, the top section contains rows `0..i` and the bottom contains rows `i+1..m-1`. The sums and counts are kept in sync incrementally.

### Checking each cut position

First, the easy case—if the two sums already match, we're done:

```python
if s1 == s2:
    return True
```

Otherwise, we look at which side is larger and compute the difference `diff`. The cell we discount must come from the **larger** side and have value exactly `diff`.

**When the bottom is larger (`s1 < s2`):**

```python
diff = s2 - s1
if cnt2[diff]:
    if (
        (m - i - 1 > 1 and n > 1)
        or (i == m - 2 and (g[i + 1][0] == diff or g[i + 1][-1] == diff))
        or (n == 1 and (g[i + 1][0] == diff or g[-1][0] == diff))
    ):
        return True
```

Here `cnt2[diff]` checks whether a cell of value `diff` exists in the bottom. The three connectivity conditions correspond exactly to the shapes discussed:

- `m - i - 1 > 1 and n > 1`: the bottom section has more than one row **and** more than one column—a thick rectangle, so any cell can be removed safely.
- `i == m - 2 and (...)`: the bottom is a **single row** (`g[i+1]`), so the removable cell must be at an end—the first column `g[i+1][0]` or last column `g[i+1][-1]`.
- `n == 1 and (...)`: the bottom is a **single column**, so the removable cell must be at the top end `g[i+1][0]` or the bottom end `g[-1][0]`.

**When the top is larger (`s1 > s2`):**

```python
else:
    diff = s1 - s2
    if cnt1[diff]:
        if (
            (i + 1 > 1 and n > 1)
            or (i == 0 and (g[0][0] == diff or g[0][-1] == diff))
            or (n == 1 and (g[0][0] == diff or g[i][0] == diff))
        ):
            return True
```

This mirrors the previous case for the top section:

- `i + 1 > 1 and n > 1`: the top has more than one row (rows `0..i`) and more than one column.
- `i == 0 and (...)`: the top is a **single row** `g[0]`, so removal must be at an end.
- `n == 1 and (...)`: the top is a **single column**, so removal must be at the top end `g[0][0]` or bottom end `g[i][0]`.

If no cut position works, the helper returns `False`.

### Handling vertical cuts via transpose

Finally, the main function runs `check` twice:

```python
return check(grid) or check(list(zip(*grid)))
```

- `check(grid)` tries all **horizontal** cuts.
- `check(list(zip(*grid)))` transposes the grid, turning **vertical** cuts into horizontal ones, then reuses the same logic.

If either returns `True`, a valid partition exists.

### Complexity

- **Time complexity:** `O(m × n)`. Each call to `check` does a constant amount of work per cell (one initialization pass and one sweep pass), and we call it twice. The transpose also costs `O(m × n)`.
- **Space complexity:** `O(m × n)` in the worst case, dominated by the hash maps `cnt1` and `cnt2` storing value counts, plus the transposed grid.

## Example Walkthrough

Let's trace through the solution with a small grid where a **horizontal cut** with a single discounted cell makes the partition work.

```
grid = [[1, 4],
        [2, 3],
        [5, 1]]
```

This is a `3 x 2` grid (`m = 3`, `n = 2`). The total sum of all cells is `1 + 4 + 2 + 3 + 5 + 1 = 16`.

We call `check(grid)` to try all horizontal cuts.

---

#### Step 1: Initialize the bottom section

Everything starts in the bottom, so `s1 = 0` and `s2 = 16`. We also fill `cnt2` with every value's count:

```
cnt2 = {1: 2, 4: 1, 2: 1, 3: 1, 5: 1}
s1 = 0, s2 = 16
```

---

#### Step 2: Sweep the cut line — move row 0 (`[1, 4]`) into the top

We iterate over the first `m - 1 = 2` rows. After moving row `0`:

```
s1 = 0 + 1 + 4 = 5
s2 = 16 - 1 - 4 = 7
cnt1 = {1: 1, 4: 1}
cnt2 = {1: 1, 2: 1, 3: 1, 5: 1}   (the 4 dropped to 0)
```

Now the cut sits between row 0 and row 1:
- **Top** = `[[1, 4]]` (rows `0`)
- **Bottom** = `[[2, 3], [5, 1]]` (rows `1..2`)

**Check this cut position (`i = 0`):**

- `s1 == s2`? `5 == 7` → no.
- Bottom is larger (`s1 < s2`), so `diff = s2 - s1 = 2`.
- Does a cell of value `2` exist in the bottom? `cnt2[2] = 1` → **yes**.
- Now verify connectivity. The bottom section is rows `1..2`, which has `m - i - 1 = 2` rows and `n = 2` columns.
  - Condition `m - i - 1 > 1 and n > 1` → `2 > 1 and 2 > 1` → **True** (a thick rectangle).

The cut succeeds. Removing the cell with value `2` (at position `grid[1][0]`) from the bottom leaves:

```
Bottom after discount = [[_, 3], [5, 1]]  → sum = 3 + 5 + 1 = 9... 
```

Wait — let's recheck the arithmetic. The discount must equalize the two sides:
- Top sum = `5`, Bottom sum = `7`, `diff = 2`.
- Remove the value-`2` cell from the **bottom**: new bottom sum = `7 - 2 = 5`.
- Now Top = `5`, Bottom = `5` → **equal**. ✓

The bottom still forms a connected shape (a `2 x 2` block missing one corner cell stays connected), so all rules are satisfied. `check(grid)` returns `True`, and the overall answer is `true`.

---

#### What if this cut had failed?

For completeness, had we continued to `i = 1` (cut between rows 1 and 2):
- **Top** = rows `0..1`, sum `= 5 + 2 + 3 = 10`; **Bottom** = row `2`, sum `= 6`.
- `diff = 4`. The bottom is a single row `[5, 1]`, with no cell of value `4`, so `cnt1[4]`/`cnt2[4]` lookups fail this position.

And vertical cuts would be tried via `check(list(zip(*grid)))`, transposing the grid into:

```
[[1, 2, 5],
 [4, 3, 1]]
```

so a vertical cut on the original becomes a horizontal sweep here — handled by the exact same logic.

Since a valid horizontal cut was already found at `i = 0`, the function short-circuits and returns `true`.

## Python Solution

```python
from typing import List
from collections import defaultdict


class Solution:
    def canPartitionGrid(self, grid: List[List[int]]) -> bool:
        # Try to find a valid horizontal cut in the given grid.
        # A horizontal cut splits the grid into a top part (first k rows)
        # and a bottom part (remaining rows). The cut is valid if the two
        # parts have equal sums, OR if removing exactly one element equal to
        # the difference (from the heavier part) balances the two parts.
        def check(g: List[List[int]]) -> bool:
            rows, cols = len(g), len(g[0])

            top_sum = bottom_sum = 0
            # Frequency of values currently in the top / bottom parts.
            top_count = defaultdict(int)
            bottom_count = defaultdict(int)

            # Initially, everything belongs to the bottom part.
            for row in g:
                for value in row:
                    bottom_sum += value
                    bottom_count[value] += 1

            # Move rows one by one from the bottom part to the top part.
            # After processing row i, the top part is rows [0..i] and the
            # bottom part is rows [i+1..rows-1].
            for i in range(rows - 1):
                for value in g[i]:
                    top_sum += value
                    bottom_sum -= value
                    top_count[value] += 1
                    bottom_count[value] -= 1

                # Case 1: the two parts already have equal sums.
                if top_sum == bottom_sum:
                    return True

                # Case 2: the bottom part is heavier. We may remove one
                # element equal to `diff` from the bottom part to balance.
                if top_sum < bottom_sum:
                    diff = bottom_sum - top_sum
                    if bottom_count[diff]:
                        # The removed element must be at a "corner-removable"
                        # position so that both parts stay non-empty/valid:
                        # - bottom part has at least 2 rows and grid wider than 1
                        #   column (an interior element can be removed), or
                        # - bottom part is exactly the last row and the element
                        #   is at one of its two ends, or
                        # - the grid has a single column and the element is at
                        #   the top or bottom end of the bottom part.
                        if (
                            (rows - i - 1 > 1 and cols > 1)
                            or (
                                i == rows - 2
                                and (g[i + 1][0] == diff or g[i + 1][-1] == diff)
                            )
                            or (cols == 1 and (g[i + 1][0] == diff or g[-1][0] == diff))
                        ):
                            return True
                # Case 3: the top part is heavier. Symmetric to case 2;
                # we may remove one element equal to `diff` from the top part.
                else:
                    diff = top_sum - bottom_sum
                    if top_count[diff]:
                        if (
                            (i + 1 > 1 and cols > 1)
                            or (i == 0 and (g[0][0] == diff or g[0][-1] == diff))
                            or (cols == 1 and (g[0][0] == diff or g[i][0] == diff))
                        ):
                            return True
            return False

        # Check horizontal cuts on the original grid, then check vertical cuts
        # by transposing the grid and reusing the same horizontal-cut logic.
        return check(grid) or check([list(row) for row in zip(*grid)])
```

## Time and Space Complexity

- **Time Complexity**: `O(m × n)`, where `m` and `n` are the number of rows and columns of the matrix, respectively. The function `check` performs two passes over the grid: the first nested loop iterates over all `m × n` elements to compute the total sum `s2` and build the count dictionary `cnt2`, taking `O(m × n)` time. The second loop iterates over the first `m - 1` rows, processing each element once to update `s1`, `s2`, `cnt1`, and `cnt2`, also taking `O(m × n)` time; the constant-time boundary checks inside add no extra cost. The `check` function is invoked twice — once on the original `grid` and once on its transpose `list(zip(*grid))`. Transposing also costs `O(m × n)`. Hence the overall time complexity remains `O(m × n)`.

- **Space Complexity**: `O(m × n)`, where `m` and `n` are the number of rows and columns of the matrix, respectively. The dominant space usage comes from the dictionaries `cnt1` and `cnt2`, which in the worst case store up to `O(m × n)` distinct values, and from constructing the transposed grid `list(zip(*grid))`, which occupies `O(m × n)` space.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Misjudging Connectivity When Discounting an Interior Cell

The most subtle and error-prone part of this problem is rule 3—**after discounting a cell, the section it was removed from must stay connected**. A naive implementation simply checks whether a cell of value `diff` exists in the heavier section and immediately returns `True`:

```python
# WRONG: ignores connectivity entirely
diff = bottom_sum - top_sum
if bottom_count[diff]:
    return True
```

This is incorrect because removing an **interior** cell from a section can split it into two disconnected pieces. For example, in a "plus"-shaped or single-row-with-a-hole layout, deleting the middle cell breaks the path between its left and right neighbors, violating the rule. The check above would wrongly accept such a cut.

**Why the correct code works:** The key insight is reasoning about *which* cells are always safe to remove rather than running an expensive connectivity test for every candidate cell:

- If a section has **more than one row AND more than one column** (`rows - i - 1 > 1 and cols > 1`), it is a "thick" rectangle. Removing *any* single cell from a rectangle of size ≥ 2×2 always leaves it connected, so we don't even need to check *which* cell—existence of value `diff` is enough.
- If the section is a **single row**, only the two **end cells** can be removed safely; removing a middle cell splits the row. Hence we must verify `g[i+1][0] == diff or g[i+1][-1] == diff`.
- If the grid is a **single column**, the same logic applies vertically—only the top/bottom ends are safe.

**Takeaway:** Don't reach for a generic BFS/DFS connectivity check per candidate; instead, classify the section's *shape* (thick rectangle vs. single line) and apply the corresponding rule. This keeps the algorithm at `O(m × n)` instead of degrading to `O((m × n)²)`.

### Pitfall 2: Forgetting That the Discount Is "At Most One Cell Total"

A second common mistake is treating the discount as *one cell per section* rather than *one cell across both sections combined*. The correct logic always removes the cell from the **heavier** side with value exactly equal to the difference:

```python
diff = bottom_sum - top_sum   # the gap to close
if bottom_count[diff]:         # cell must live in the heavier (bottom) side
    ...
```

Removing a cell of value `diff` from the heavier side closes the gap exactly. Trying to remove a cell from the lighter side, or attempting two removals, both violate the problem's constraint and lead to wrong answers. Always anchor on: *which side is bigger, and does it contain a single cell equal to the surplus?*

### Pitfall 3: Off-By-One Errors on the "Both Sections Non-Empty" Rule

The loop runs `for i in range(rows - 1)` rather than `range(rows)`. Iterating up to `rows - 1` ensures the bottom always retains at least one row, satisfying rule 1 (both sections non-empty). A frequent bug is writing `range(rows)`, which allows a "cut" that leaves the bottom empty—an invalid partition that should never be considered. When adapting this pattern, double-check that the sweep stops one position before the last row/column.

