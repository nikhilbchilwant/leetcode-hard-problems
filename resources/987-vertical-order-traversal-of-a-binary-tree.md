# 987. Vertical Order Traversal of a Binary Tree

> Source: https://algo.monster/liteproblems/987

## Problem Statement

You are given the root of a binary [tree](/problems/tree_intro) and need to perform a **vertical order traversal** of the tree.

Each node in the [tree](/problems/tree_intro) has a position defined by coordinates `(row, col)`:
- The root node starts at position `(0, 0)`
- For any node at position `(row, col)`:
  - Its left child is at position `(row + 1, col - 1)`
  - Its right child is at position `(row + 1, col + 1)`

The vertical order traversal requires you to:
1. Group all nodes by their column index (from leftmost to rightmost column)
2. Within each column, order nodes from top to bottom (by row)
3. If multiple nodes share the same position `(row, col)`, sort them by their values in ascending order

The output should be a list of lists, where each inner list contains the values of nodes in a single column, ordered according to the rules above.

For example, if you have nodes at:
- Column -1: nodes with values [9, 7]
- Column 0: nodes with values [3, 15]  
- Column 1: nodes with values [20]

The vertical order traversal would return: `[[9, 7], [3, 15], [20]]`

The key challenge is handling nodes that occupy the same position - these must be sorted by their values before being added to the result.

## Intuition

The key insight is that we need to track two pieces of information for each node: its position in 2D space and its value. Since we're asked for a vertical order traversal (column by column), we need to know which column each node belongs to.

Think of the [tree](/problems/tree_intro) as being placed on a 2D grid. The root sits at position `(0, 0)`. As we move down the tree, going left means moving one column to the left (`col - 1`), and going right means moving one column to the right (`col + 1`). The row always increases by 1 as we go deeper.

The natural approach is to traverse the entire [tree](/problems/tree_intro) and record where each node sits. DFS works perfectly here because:
1. We can easily pass the current position `(row, col)` as parameters during recursion
2. When visiting a left child, we update to `(row + 1, col - 1)`
3. When visiting a right child, we update to `(row + 1, col + 1)`

Once we've collected all nodes with their positions, the problem becomes a [sorting](/problems/sorting_summary) problem. We need to group nodes by column, and within each column, order them by row (top to bottom). The tricky part is when multiple nodes share the same exact position - in this case, we sort by their values.

The clever trick in the solution is to store each node as a tuple `(column, row, value)`. When we sort this list, Python naturally sorts by the first element (column), then by the second element (row) for ties, and finally by the third element (value) for nodes at the same position. This gives us exactly the ordering we need.

After [sorting](/problems/sorting_summary), we simply iterate through the sorted list and group consecutive nodes with the same column value into the same sublist. This produces our final vertical order traversal.

**Learn more about [Tree](/problems/tree_intro), [Depth-First Search](/problems/dfs_intro), [Breadth-First Search](/problems/bfs_intro), [Binary Tree](/problems/binary_tree_min_depth) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

The solution uses DFS combined with [sorting](/problems/sorting_summary) to achieve the vertical order traversal. Let's break down the implementation:

**1. DFS Traversal with Position Tracking**

We define a recursive function `dfs(root, i, j)` where:
- `root` is the current node
- `i` represents the row (depth level)
- `j` represents the column

During the DFS traversal:
- For each non-null node, we record its position and value as a tuple `(j, i, root.val)` in the `nodes` list
- Note the order: column first, then row, then value - this is intentional for [sorting](/problems/sorting_summary) later
- Recursively visit the left child with position `(i + 1, j - 1)` 
- Recursively visit the right child with position `(i + 1, j + 1)`

**2. [Sorting](/problems/sorting_summary) the Collected Nodes**

After collecting all nodes with their positions, we call `nodes.sort()`. Python's default tuple [sorting](/problems/sorting_summary) behavior works perfectly here:
- First sorts by column `j` (leftmost to rightmost)
- For nodes in the same column, sorts by row `i` (top to bottom)
- For nodes at the exact same position, sorts by value

**3. Grouping by Column**

We iterate through the sorted `nodes` list and group nodes by column:
- Initialize `prev = -2000` (a value outside possible column range)
- For each node `(j, _, val)` in the sorted list:
  - If the column `j` differs from `prev`, start a new group by appending an empty list to `ans`
  - Update `prev = j` to track the current column
  - Append the node's value to the last group in `ans`

The algorithm has a time complexity of `O(n log n)` where `n` is the number of nodes, dominated by the [sorting](/problems/sorting_summary) step. The space complexity is `O(n)` for storing the nodes list.

This approach elegantly handles all the requirements: vertical ordering by column, top-to-bottom ordering within columns, and value-based [sorting](/problems/sorting_summary) for nodes at the same position.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

Consider this binary tree:
```
       3
      / \
     9   20
        /  \
       15   7
```

**Step 1: DFS Traversal with Position Tracking**

Starting from the root at position (0, 0), we traverse the tree and record each node's position:

- Visit node 3 at (row=0, col=0) → Record: (0, 0, 3)
- Visit left child 9 at (row=1, col=-1) → Record: (-1, 1, 9)
- Visit right child 20 at (row=1, col=1) → Record: (1, 1, 20)
- Visit 20's left child 15 at (row=2, col=0) → Record: (0, 2, 15)
- Visit 20's right child 7 at (row=2, col=2) → Record: (2, 2, 7)

Our `nodes` list now contains:
```
[(0, 0, 3), (-1, 1, 9), (1, 1, 20), (0, 2, 15), (2, 2, 7)]
```

**Step 2: Sorting**

Sort the list by (column, row, value):
```
[(-1, 1, 9), (0, 0, 3), (0, 2, 15), (1, 1, 20), (2, 2, 7)]
```

Notice how the sorting groups nodes by column first:
- Column -1: [(9)]
- Column 0: [(3), (15)] - ordered by row (0 before 2)
- Column 1: [(20)]
- Column 2: [(7)]

**Step 3: Grouping by Column**

Iterate through the sorted list and group by column:

- Process (-1, 1, 9): Column -1 is new, create group [9]
- Process (0, 0, 3): Column 0 is new, create group [3]
- Process (0, 2, 15): Column 0 continues, add to current group [3, 15]
- Process (1, 1, 20): Column 1 is new, create group [20]
- Process (2, 2, 7): Column 2 is new, create group [7]

**Final Result:** `[[9], [3, 15], [20], [7]]`

This represents the vertical order traversal from left to right, with nodes in each column ordered from top to bottom.

## Python Solution

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right

from typing import Optional, List

class Solution:
    def verticalTraversal(self, root: Optional[TreeNode]) -> List[List[int]]:
        """
        Performs vertical traversal of a binary tree.
        Returns nodes grouped by their vertical column from left to right.
        Within each column, nodes are ordered by row (top to bottom),
        and nodes at the same position are ordered by value.
        """
      
        def dfs(node: Optional[TreeNode], row: int, col: int) -> None:
            """
            Depth-first search to collect all nodes with their positions.
          
            Args:
                node: Current tree node
                row: Current row position (depth in tree)
                col: Current column position (horizontal distance from root)
            """
            if node is None:
                return
          
            # Store node information as (column, row, value) for proper sorting
            node_positions.append((col, row, node.val))
          
            # Traverse left subtree (row increases, column decreases)
            dfs(node.left, row + 1, col - 1)
          
            # Traverse right subtree (row increases, column increases)
            dfs(node.right, row + 1, col + 1)
      
        # Collect all nodes with their positions
        node_positions = []
        dfs(root, 0, 0)
      
        # Sort by column first, then row, then value
        node_positions.sort()
      
        # Group nodes by column
        result = []
        previous_col = -2000  # Initialize with value outside possible column range
      
        for col, _, val in node_positions:
            # Start a new column group if column changed
            if previous_col != col:
                result.append([])
                previous_col = col
          
            # Add node value to current column group
            result[-1].append(val)
      
        return result
```

## Time and Space Complexity

**Time Complexity:** `O(n × log n)`

The algorithm consists of two main phases:
1. **DFS traversal:** The `dfs` function visits each node exactly once, collecting node information with coordinates `(j, i, val)`. This takes `O(n)` time where `n` is the number of nodes.
2. **Sorting:** After collecting all nodes, the `nodes` list containing `n` elements is sorted. Python's sort operation has a time complexity of `O(n × log n)`.
3. **Building result:** The final loop iterates through the sorted nodes once to construct the answer, taking `O(n)` time.

The overall time complexity is dominated by the sorting step: `O(n) + O(n × log n) + O(n) = O(n × log n)`.

**Space Complexity:** `O(n)`

The space usage includes:
1. **Nodes list:** Stores information for all `n` nodes as tuples `(j, i, val)`, requiring `O(n)` space.
2. **Recursion stack:** In the worst case (skewed tree), the DFS recursion depth can be `O(n)`.
3. **Answer list:** Stores all `n` node values in the final result, using `O(n)` space.

The total space complexity is `O(n) + O(n) + O(n) = O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall: Incorrect Handling of Same-Position Nodes

**The Problem:**
A common mistake is forgetting to sort nodes by their values when they share the exact same position `(row, col)`. Developers often assume that nodes at the same position should maintain their traversal order or be added in the order they're discovered during DFS.

**Example of Incorrect Implementation:**
```python
def verticalTraversal(self, root: Optional[TreeNode]) -> List[List[int]]:
    from collections import defaultdict
  
    columns = defaultdict(list)
  
    def dfs(node, row, col):
        if not node:
            return
        # Wrong: Simply appending nodes without considering same-position sorting
        columns[col].append((row, node.val))
        dfs(node.left, row + 1, col - 1)
        dfs(node.right, row + 1, col + 1)
  
    dfs(root, 0, 0)
  
    result = []
    for col in sorted(columns.keys()):
        # Wrong: Only sorting by row, not handling same-position value sorting
        column_nodes = sorted(columns[col], key=lambda x: x[0])
        result.append([val for _, val in column_nodes])
  
    return result
```

**Why This Fails:**
Consider a tree where two nodes with values 5 and 3 both end up at position `(2, 1)`. The incorrect implementation might output them as `[5, 3]` based on traversal order, but the correct output should be `[3, 5]` (sorted by value).

**The Correct Solution:**
The provided solution avoids this pitfall by:
1. Storing all three pieces of information together: `(col, row, node.val)`
2. Using Python's tuple sorting which naturally handles all three sorting criteria:
   - Primary sort by column (left to right)
   - Secondary sort by row (top to bottom)
   - Tertiary sort by value (ascending order for same position)

```python
# Correct approach - tuple includes all sorting criteria
node_positions.append((col, row, node.val))
node_positions.sort()  # Sorts by all three fields automatically
```

**Alternative Correct Approach Using Custom Sorting:**
If you prefer to use a dictionary-based approach, ensure proper sorting:
```python
from collections import defaultdict

columns = defaultdict(list)

def dfs(node, row, col):
    if not node:
        return
    columns[col].append((row, node.val))
    dfs(node.left, row + 1, col - 1)
    dfs(node.right, row + 1, col + 1)

dfs(root, 0, 0)

result = []
for col in sorted(columns.keys()):
    # Correct: Sort by both row AND value
    column_nodes = sorted(columns[col], key=lambda x: (x[0], x[1]))
    result.append([val for _, val in column_nodes])
```

**Test Case to Verify:**
```
Tree:        1
           /   \
          2     3
         / \   / \
        4   6 5   7
      
Where nodes 6 and 5 end up at the same position.
Expected output should have 5 before 6 in their column.
```

