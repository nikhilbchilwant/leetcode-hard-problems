# 296. Best Meeting Point

> Source: https://algo.monster/liteproblems/296

## Problem Statement

You are given an `m x n` binary grid where each cell contains either 0 or 1. Each cell with value 1 represents the home of a friend. Your task is to find a meeting point such that the total travel distance for all friends to reach this meeting point is minimized.

The total travel distance is calculated as the sum of individual distances from each friend's home to the meeting point. The distance between any two points is measured using Manhattan Distance, which is defined as `distance(p1, p2) = |p2.x - p1.x| + |p2.y - p1.y|`.

For example, if you have a grid with friends at positions `(0,0)`, `(0,2)`, and `(2,1)`, and you choose meeting point `(0,1)`, the total travel distance would be:
- Friend at `(0,0)` to `(0,1)`: `|0-0| + |1-0| = 1`
- Friend at `(0,2)` to `(0,1)`: `|0-0| + |1-2| = 1`
- Friend at `(2,1)` to `(0,1)`: `|0-2| + |1-1| = 2`
- Total: `1 + 1 + 2 = 4`

The meeting point can be any cell in the grid (not necessarily a cell with value 1). Your goal is to return the minimum possible total travel distance.

## Intuition

The key insight is that Manhattan Distance allows us to treat the x and y coordinates independently. Since `distance = |x2 - x1| + |y2 - y1|`, we can minimize the sum of x-distances and y-distances separately.

Consider a simpler 1D problem first: if friends live at positions on a line, where should they meet to minimize total travel distance? Let's say friends are at positions `[1, 3, 5, 7]`. If we choose the meeting point at position `4` (the median), the total distance is `|1-4| + |3-4| + |5-4| + |7-4| = 3 + 1 + 1 + 3 = 8`.

Why does the median work? When we have an even number of points, any position between the two middle points gives the same minimum total distance. When we move the meeting point away from the median, we increase the distance for more friends than we decrease it for others. For instance, if we move from the median towards the left, we get closer to friends on the left but farther from friends on the right - and there are at least as many friends on the right as on the left.

This principle extends to 2D: we can find the median of all row coordinates (treating each friend's row position independently) and the median of all column coordinates. The optimal meeting point is at `(median_row, median_column)`.

The beauty of this approach is that we don't need to check every possible cell in the grid. By decomposing the 2D Manhattan Distance problem into two independent 1D problems, we can directly calculate the optimal meeting point as the median position in each dimension.

## Solution Approach

The implementation follows the median-finding approach we identified:

1. **Extract all friend positions**: First, we iterate through the entire grid to find all cells with value 1. For each friend found at position `(i, j)`, we store their row index `i` in a `rows` list and column index `j` in a `cols` list.

2. **Sort the column positions**: The `rows` list is naturally sorted since we iterate through the grid row by row. However, the `cols` list needs explicit sorting because we collect column indices as we traverse left to right within each row.

3. **Find the median positions**:
   - The median row is found at index `len(rows) >> 1` (equivalent to `len(rows) // 2`)
   - The median column is found at index `len(cols) >> 1` after sorting
   - For even-length lists, this gives us the upper-middle element, which works as the median for our purpose

4. **Calculate total distance**: The helper function `f(arr, x)` computes the sum of distances from all points in `arr` to position `x`. We apply this function twice:
   - `f(rows, i)`: calculates the total vertical distance from all friends to the median row `i`
   - `f(cols, j)`: calculates the total horizontal distance from all friends to the median column `j`
   - The final answer is the sum of these two values

The time complexity is `O(m*n + k*log(k))` where `k` is the number of friends (cells with value 1), due to the grid traversal and sorting of columns. The space complexity is `O(k)` for storing the friend positions.

This solution elegantly uses the property that Manhattan Distance can be decomposed into independent dimensions, allowing us to solve a 2D optimization problem by solving two simpler 1D median-finding problems.

## Example Walkthrough

Let's walk through a concrete example to illustrate the solution approach.

Consider this 3×3 grid:
```
1 0 0
0 1 0
1 1 0
```

**Step 1: Extract friend positions**
We scan the grid and find friends (cells with value 1) at:
- (0, 0) → add row 0 to `rows`, column 0 to `cols`
- (1, 1) → add row 1 to `rows`, column 1 to `cols`
- (2, 0) → add row 2 to `rows`, column 0 to `cols`
- (2, 1) → add row 2 to `rows`, column 1 to `cols`

After extraction:
- `rows = [0, 1, 2, 2]` (naturally sorted from our row-by-row traversal)
- `cols = [0, 1, 0, 1]` (unsorted, in order of discovery)

**Step 2: Sort column positions**
- `cols` after sorting: `[0, 0, 1, 1]`

**Step 3: Find median positions**
- Number of friends: 4
- Median index: `4 >> 1 = 2` (the upper-middle element for even-length lists)
- Median row: `rows[2] = 2`
- Median column: `cols[2] = 1`
- Meeting point: `(2, 1)`

**Step 4: Calculate total distance**

For row distances to median row 2:
- `|0 - 2| = 2`
- `|1 - 2| = 1`
- `|2 - 2| = 0`
- `|2 - 2| = 0`
- Row distance sum: `2 + 1 + 0 + 0 = 3`

For column distances to median column 1:
- `|0 - 1| = 1`
- `|0 - 1| = 1`
- `|1 - 1| = 0`
- `|1 - 1| = 0`
- Column distance sum: `1 + 1 + 0 + 0 = 2`

**Total minimum distance: 3 + 2 = 5**

We can verify this is optimal by checking the meeting point (2, 1):
- Friend at (0, 0) travels: `|2-0| + |1-0| = 2 + 1 = 3`
- Friend at (1, 1) travels: `|2-1| + |1-1| = 1 + 0 = 1`
- Friend at (2, 0) travels: `|2-2| + |1-0| = 0 + 1 = 1`
- Friend at (2, 1) travels: `|2-2| + |1-1| = 0 + 0 = 0`
- Total: `3 + 1 + 1 + 0 = 5` ✓

Notice how the median approach naturally finds the optimal meeting point without checking all possible cells in the grid. The meeting point (2, 1) happens to be where a friend lives, but this isn't always the case - the algorithm would work equally well if the optimal point were an empty cell.

## Python Solution

```python
class Solution:
    def minTotalDistance(self, grid: List[List[int]]) -> int:
        """
        Find the minimum total distance for all people to meet at one point.
        The optimal meeting point is the median of all x-coordinates and y-coordinates.

        Args:
            grid: 2D grid where 1 represents a person's location, 0 represents empty space

        Returns:
            Minimum total Manhattan distance for all people to meet
        """

        def calculate_distance_sum(positions: List[int], meeting_point: int) -> int:
            """
            Calculate sum of distances from all positions to the meeting point.

            Args:
                positions: List of coordinate values (either row or column indices)
                meeting_point: The target coordinate to measure distance to

            Returns:
                Sum of absolute distances
            """
            return sum(abs(position - meeting_point) for position in positions)

        # Collect all row and column indices where people are located
        row_indices = []
        column_indices = []

        for row_index, row in enumerate(grid):
            for column_index, cell_value in enumerate(row):
                if cell_value == 1:  # Person found at this location
                    row_indices.append(row_index)
                    column_indices.append(column_index)

        # Sort column indices to find median (row indices already sorted due to iteration order)
        column_indices.sort()

        # Find median positions (optimal meeting point)
        # Using bit shift for integer division by 2
        median_row = row_indices[len(row_indices) >> 1]
        median_column = column_indices[len(column_indices) >> 1]

        # Calculate total distance as sum of row distances and column distances
        total_distance = calculate_distance_sum(row_indices, median_row) + \
                        calculate_distance_sum(column_indices, median_column)

        return total_distance
```

## Time and Space Complexity

**Time Complexity:** `O(m * n * log(m * n))`

The algorithm iterates through the entire grid once to collect positions of all `1`s, which takes `O(m * n)` time where `m` is the number of rows and `n` is the number of columns. The `rows` list is already sorted as we traverse row by row, but the `cols` list needs to be sorted explicitly using `cols.sort()`, which takes `O(k * log k)` time where `k` is the number of `1`s in the grid. In the worst case, `k = m * n` when all cells contain `1`. Finding the median takes `O(1)` time, and calculating the total distance using function `f` takes `O(k)` time for each dimension. Therefore, the overall time complexity is `O(m * n + k * log k + k) = O(m * n * log(m * n))` in the worst case.

**Space Complexity:** `O(m * n)`

The algorithm uses two lists `rows` and `cols` to store the coordinates of all `1`s in the grid. In the worst case, when all cells in the grid contain `1`, both lists will have `m * n` elements. Therefore, the space complexity is `O(k)` where `k` is the number of `1`s, which is `O(m * n)` in the worst case.

## Common Pitfalls

### 1. **Assuming the Meeting Point Must Be at a Friend's Location**

A common misconception is thinking the optimal meeting point must be at one of the cells containing a 1 (friend's home). This can lead to inefficient solutions that only check friend locations as potential meeting points.

**Why it's wrong:** The optimal meeting point can be any cell in the grid, including empty cells (0s). For example, if friends are at positions (0,0) and (0,2), the optimal meeting point is (0,1), which might be an empty cell.

**Solution:** Use the mathematical property that the median minimizes the sum of absolute deviations. Don't restrict the search to only cells with value 1.

### 2. **Incorrectly Handling the 2D Median Calculation**

Some might try to find a "2D median" by treating the problem as finding a single point that minimizes distance in both dimensions simultaneously, leading to complex calculations or incorrect results.

**Why it's wrong:** Manhattan distance allows us to decompose the problem into two independent 1D problems. The optimal x-coordinate and y-coordinate can be found separately.

**Solution:** Separate the row and column coordinates into two lists, find the median of each independently, and sum the distances:

```python
# Correct approach - handle dimensions independently
median_row = row_indices[len(row_indices) // 2]
median_column = column_indices[len(column_indices) // 2]
```

### 3. **Forgetting to Sort the Column Indices**

When collecting friend positions by iterating through the grid row by row, the row indices are naturally sorted, but column indices are not. Forgetting to sort the column list before finding the median will give incorrect results.

**Why it's wrong:** The median requires a sorted list. While row indices are collected in order (0, 0, 1, 1, 2...), column indices might be collected as (0, 2, 1, 3...) depending on friend positions.

**Solution:** Always sort the column indices before finding the median:

```python
# Row indices are already sorted due to iteration order
# Column indices MUST be sorted
column_indices.sort()
median_column = column_indices[len(column_indices) // 2]
```

### 4. **Using Mean Instead of Median**

Some might intuitively use the average (mean) of all positions instead of the median, thinking it represents the "center" of all friends.

**Why it's wrong:** The mean minimizes squared distances, not absolute distances. For Manhattan distance (sum of absolute differences), the median is the optimal choice.

**Solution:** Always use median for minimizing sum of absolute distances:

```python
# Wrong - using mean
mean_row = sum(row_indices) / len(row_indices)

# Correct - using median
median_row = row_indices[len(row_indices) // 2]
```

