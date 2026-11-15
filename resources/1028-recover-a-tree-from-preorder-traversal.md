# 1028. Recover a Tree From Preorder Traversal

> Source: https://algo.monster/liteproblems/1028

## Problem Statement

You are given a string that represents the output of a preorder [depth-first search](/problems/dfs_intro) (DFS) traversal of a binary [tree](/problems/tree_intro). In this traversal output:

- Each node is represented by a certain number of dashes followed by its value
- The number of dashes indicates the depth of the node in the [tree](/problems/tree_intro)
- The root node has depth 0 (no dashes)
- Each child node has depth = parent's depth + 1

For example, if you see `"1-2--3"`, this means:
- `1` is at depth 0 (root)
- `2` is at depth 1 (child of 1)
- `3` is at depth 2 (child of 2)

The traversal follows preorder DFS, meaning nodes are visited in this order:
1. Visit the current node
2. Visit the left subtree
3. Visit the right subtree

An important constraint: if a node has only one child, that child must be the left child.

Your task is to reconstruct the original binary [tree](/problems/tree_intro) from this traversal string and return the root node.

**Example walkthrough:**

Given input `"1-2--3--4-5--6--7"`:
- `1` (0 dashes) - root
- `2` (1 dash) - left child of 1
- `3` (2 dashes) - left child of 2
- `4` (2 dashes) - right child of 2
- `5` (1 dash) - right child of 1
- `6` (2 dashes) - left child of 5
- `7` (2 dashes) - right child of 5

This reconstructs to:
```
      1
     / \
    2   5
   / \ / \
  3  4 6  7
```

## Intuition

The key insight is that in a preorder DFS traversal, we always process nodes in a specific order: parent first, then its children. When we see the traversal string, we're essentially seeing the exact path the DFS took through the [tree](/problems/tree_intro).

Think about how DFS works when traversing a [tree](/problems/tree_intro):
1. We start at the root
2. We go as deep as possible along the left path
3. When we can't go deeper, we backtrack and explore right branches

The number of dashes tells us exactly how deep we are in the [tree](/problems/tree_intro). This is crucial because:
- If we see a node with `d` dashes, it must be at depth `d`
- If the next node has `d+1` dashes, it's a child of the current node
- If the next node has the same or fewer dashes, we need to backtrack

Consider this simple example: `"1-2--3-4"`
- `1` (depth 0): root
- `2` (depth 1): must be child of 1
- `3` (depth 2): must be child of 2
- `4` (depth 1): can't be child of 3 (depth decreased), must backtrack to 1 and be its other child

This naturally suggests using a stack to track our current path from root:
- When depth increases: we're going deeper (current node's child)
- When depth stays same or decreases: we need to backtrack (pop from stack)

The stack always maintains the path from root to the current position. When we need to attach a new node:
1. We pop nodes until the stack size matches the target depth (this is our backtracking)
2. The top of the stack is now the parent of our new node
3. We attach as left child if it's empty, otherwise as right child (following the constraint that single children must be left children)

This approach directly mirrors how we would manually trace through the DFS traversal - keeping track of where we are in the [tree](/problems/tree_intro) and backtracking when necessary based on the depth indicators.

**Learn more about [Tree](/problems/tree_intro), [Depth-First Search](/problems/dfs_intro) and [Binary Tree](/problems/binary_tree_min_depth) patterns.**

## Solution Approach

The implementation uses a stack-based approach to reconstruct the [tree](/problems/tree_intro) from its preorder traversal string. Here's how the algorithm works:

**Data Structure:**
- A `Stack<TreeNode>` to maintain the current path from root to the node being processed

**Algorithm Steps:**

1. **Parse the traversal string** using an index pointer `i`:
   ```
   while (i < traversal.length())
   ```

2. **Extract depth information** by counting consecutive dashes:
   ```java
   int depth = 0;
   while (i < traversal.length() && traversal.charAt(i) == '-') {
       depth++;
       i++;
   }
   ```

3. **Extract the node value** by reading consecutive digits:
   ```java
   int num = 0;
   while (i < traversal.length() && Character.isDigit(traversal.charAt(i))) {
       num = num * 10 + (traversal.charAt(i) - '0');
       i++;
   }
   ```

4. **Create the new node** with the extracted value:
   ```java
   TreeNode newNode = new TreeNode(num);
   ```

5. **Adjust stack to match target depth** (backtracking):
   ```java
   while (stack.size() > depth) {
       stack.pop();
   }
   ```
   This ensures the stack size equals the depth, meaning the top of the stack is the parent of the new node.

6. **Attach the new node to its parent**:
   ```java
   if (!stack.isEmpty()) {
       if (stack.peek().left == null) {
           stack.peek().left = newNode;
       } else {
           stack.peek().right = newNode;
       }
   }
   ```
   Following the rule that if a node has only one child, it must be the left child.

7. **Push the new node onto the stack** for future processing:
   ```java
   stack.push(newNode);
   ```

8. **Return the root** (first element in the stack):
   ```java
   return stack.isEmpty() ? null : stack.get(0);
   ```

**Example Walkthrough:**

For input `"1-2--3"`:
- Parse `1` (depth=0): Create node(1), stack is empty so this is root, push to stack: `[1]`
- Parse `2` (depth=1): Create node(2), stack size (1) matches depth (1), attach as left child of node(1), push to stack: `[1, 2]`
- Parse `3` (depth=2): Create node(3), stack size (2) matches depth (2), attach as left child of node(2), push to stack: `[1, 2, 3]`

Final [tree](/problems/tree_intro):
```
    1
   /
  2
 /
3
```

The beauty of this approach is that the stack naturally handles the backtracking needed when moving between different branches of the [tree](/problems/tree_intro), perfectly mirroring the DFS traversal pattern.

## Example Walkthrough

Let's trace through the input `"1-2--4-3"` step by step to understand how the stack-based approach reconstructs the tree.

**Initial State:**
- Stack: `[]`
- Index `i = 0`

**Step 1: Process "1"**
- Count dashes: 0 dashes → depth = 0
- Extract value: 1
- Create node(1)
- Stack size (0) matches depth (0), so no popping needed
- Stack is empty, so this is our root
- Push node(1) to stack
- Stack: `[node(1)]`

**Step 2: Process "-2"**
- Count dashes: 1 dash → depth = 1
- Extract value: 2
- Create node(2)
- Stack size (1) matches depth (1), so no popping needed
- Parent is node(1) (top of stack)
- node(1).left is null, so attach node(2) as left child
- Push node(2) to stack
- Stack: `[node(1), node(2)]`
- Tree so far:
```
  1
 /
2
```

**Step 3: Process "--4"**
- Count dashes: 2 dashes → depth = 2
- Extract value: 4
- Create node(4)
- Stack size (2) matches depth (2), so no popping needed
- Parent is node(2) (top of stack)
- node(2).left is null, so attach node(4) as left child
- Push node(4) to stack
- Stack: `[node(1), node(2), node(4)]`
- Tree so far:
```
  1
 /
2
/
4
```

**Step 4: Process "-3"**
- Count dashes: 1 dash → depth = 1
- Extract value: 3
- Create node(3)
- Stack size (3) > depth (1), so pop until size equals 1
  - Pop node(4): Stack: `[node(1), node(2)]`
  - Pop node(2): Stack: `[node(1)]`
- Now stack size (1) matches depth (1)
- Parent is node(1) (top of stack)
- node(1).left is not null (it's node(2)), so attach node(3) as right child
- Push node(3) to stack
- Stack: `[node(1), node(3)]`

**Final Tree:**
```
    1
   / \
  2   3
 /
4
```

The algorithm successfully reconstructed the tree by:
1. Using depth to determine parent-child relationships
2. Using the stack to maintain the current path and handle backtracking
3. Following the left-child-first rule when attaching nodes

## Python Solution

```python
# Definition for a binary tree node
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right


def recoverFromPreorder(traversal: str) -> TreeNode:
    """
    Recovers a binary tree from its preorder traversal string representation.
    The traversal string uses dashes to indicate depth levels.
  
    Args:
        traversal: The preorder traversal string with depth indicators
      
    Returns:
        The root node of the recovered binary tree
    """
    # Stack to maintain the current path from root to the node being processed
    node_stack = []
    current_index = 0
  
    while current_index < len(traversal):
        # Count the number of dashes to determine the depth of the current node
        current_depth = 0
        while current_index < len(traversal) and traversal[current_index] == '-':
            current_depth += 1
            current_index += 1
      
        # Parse the node value from the traversal string
        node_value = 0
        while current_index < len(traversal) and traversal[current_index].isdigit():
            node_value = node_value * 10 + int(traversal[current_index])
            current_index += 1
      
        # Create a new tree node with the parsed value
        current_node = TreeNode(node_value)
      
        # Pop nodes from stack until we reach the correct depth
        # This ensures the stack contains only ancestors of the current node
        while len(node_stack) > current_depth:
            node_stack.pop()
      
        # Attach the current node to its parent
        if node_stack:
            parent_node = node_stack[-1]
            # If left child is empty, attach as left child; otherwise, attach as right child
            if parent_node.left is None:
                parent_node.left = current_node
            else:
                parent_node.right = current_node
      
        # Add the current node to the stack for future processing
        node_stack.append(current_node)
  
    # Return the root node (first element in stack) or None if stack is empty
    return node_stack[0] if node_stack else None
```

## Time and Space Complexity

**Time Complexity: `O(n)`**

The algorithm iterates through the input string `traversal` exactly once, where `n` is the length of the string. During this traversal:
- Each character is visited once to count dashes (depth calculation)
- Each character is visited once to parse numbers
- The stack operations (push/pop) for each node are performed at most once

While there's a nested loop for popping from the stack (`while (stack.size() > depth)`), each node is pushed once and popped at most once throughout the entire execution, making the amortized cost `O(1)` per node. Since the number of nodes is proportional to the length of the string (actually less than `n`), the overall time complexity remains `O(n)`.

**Space Complexity: `O(h)`**

The space complexity is determined by:
- The stack that stores TreeNode references, which at any point contains nodes along the current path from root to the node being processed
- The maximum size of the stack equals the height `h` of the tree being constructed
- In the worst case (completely skewed tree), `h = O(n)` where `n` is the number of nodes
- In the best case (balanced tree), `h = O(log n)`

The additional space used for variables like `i`, `depth`, and `num` is `O(1)`.

Therefore, the space complexity is `O(h)` where `h` is the height of the resulting tree.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Depth Calculation Due to Off-by-One Errors**

A common mistake is misunderstanding the relationship between stack size and depth. Developers often incorrectly pop nodes from the stack, leading to wrong parent-child relationships.

**Pitfall Example:**
```python
# WRONG: Popping until stack size equals depth - 1
while len(node_stack) > current_depth - 1:
    node_stack.pop()
```

**Why it's wrong:** The depth represents the exact level in the tree. If depth is 2, we need exactly 2 ancestors in the stack (root at depth 0, parent at depth 1). The current node will be at depth 2.

**Solution:** Always pop until `len(node_stack) == current_depth`:
```python
# CORRECT: Stack size should equal the depth
while len(node_stack) > current_depth:
    node_stack.pop()
```

### 2. **Parsing Multi-Digit Numbers Incorrectly**

Another frequent error is assuming all node values are single digits, which fails for values like 10, 123, etc.

**Pitfall Example:**
```python
# WRONG: Only reading a single digit
node_value = int(traversal[current_index])
current_index += 1
```

**Solution:** Continue parsing while encountering digits:
```python
# CORRECT: Parse all consecutive digits
node_value = 0
while current_index < len(traversal) and traversal[current_index].isdigit():
    node_value = node_value * 10 + int(traversal[current_index])
    current_index += 1
```

### 3. **Not Handling Edge Cases Properly**

Failing to handle empty input or single-node trees can cause index errors or null pointer exceptions.

**Pitfall Example:**
```python
# WRONG: Assuming stack always has elements
return node_stack[0]  # Crashes if traversal is empty string
```

**Solution:** Check for empty stack:
```python
# CORRECT: Handle empty input gracefully
return node_stack[0] if node_stack else None
```

### 4. **Misunderstanding the Single Child Rule**

Some implementations incorrectly place a single child as the right child, violating the constraint that single children must be left children.

**Pitfall Example:**
```python
# WRONG: Randomly choosing left or right for single child
if parent_node.left is None and parent_node.right is None:
    # Might incorrectly assign to right child
    parent_node.right = current_node
```

**Solution:** Always check left child first:
```python
# CORRECT: Single child must be left child
if parent_node.left is None:
    parent_node.left = current_node
else:
    parent_node.right = current_node
```

### 5. **Stack Management During Backtracking**

A subtle but critical mistake is not properly maintaining the stack when backtracking to different branches.

**Pitfall Example:**
```python
# WRONG: Forgetting to add current node to stack
while len(node_stack) > current_depth:
    node_stack.pop()
# Missing: node_stack.append(current_node)
```

This breaks the algorithm for subsequent nodes because the stack doesn't accurately represent the path from root to current position.

**Solution:** Always push the current node after processing:
```python
# CORRECT: Maintain stack integrity
while len(node_stack) > current_depth:
    node_stack.pop()
# ... attach node to parent ...
node_stack.append(current_node)  # Don't forget this!
```

