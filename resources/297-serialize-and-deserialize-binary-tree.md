# 297. Serialize and Deserialize Binary Tree

> Source: https://algo.monster/liteproblems/297

## Problem Statement

This problem asks you to design an algorithm that can convert a binary tree into a string representation (serialization) and then convert that string back into the original binary tree structure (deserialization).

**Serialization** means taking a binary tree data structure and converting it into a string format that can be easily stored or transmitted. 

**Deserialization** means taking that string and reconstructing the exact same binary tree from it.

The key requirements are:
- You need to implement two methods: `serialize` and `deserialize`
- The `serialize` method takes a binary tree root node and returns a string
- The `deserialize` method takes a string and returns the root of the reconstructed binary tree
- After serializing and then deserializing a tree, you should get back the exact same tree structure

You have complete freedom in choosing how to represent the tree as a string. The only constraint is that your serialization method must produce a string that your deserialization method can correctly interpret to rebuild the original tree.

For example, if you have a tree:
```
    1
   / \
  2   3
     / \
    4   5
```

Your serialize method might produce a string like `"1,2,3,#,#,4,5,#,#,#,#"` (where `#` represents null nodes), and your deserialize method should be able to take this string and reconstruct the exact same tree structure.

## Intuition

To serialize and deserialize a binary tree, we need to find a way to capture all the information about the tree structure in a string format that we can later decode.

The key insight is that we need to record two things:
1. The values of all nodes
2. The structure/relationships between nodes (which nodes are children of which parent)

When we traverse a tree, we visit nodes in a specific order. If we can record this order along with markers for missing nodes (null children), we can reconstruct the exact same tree structure later.

Think about how you would describe a tree to someone over the phone. You might say: "Start with node 1, its left child is 2, its right child is 3. Node 2 has no children. Node 3's left child is 4, right child is 5..." This is essentially what serialization does - it creates a systematic description of the tree.

The challenge is handling null nodes. In a complete binary tree, we know exactly where each node should be. But in an incomplete tree, we need to mark where nodes are missing. This is why we use a special character like `#` to represent null nodes.

Whether we use BFS (level-order) or DFS (pre/in/post-order), the principle remains the same:
- **For serialization**: Visit nodes in a systematic order and record their values, using a special marker for nulls
- **For deserialization**: Read the values in the same order and reconstruct the tree following the same traversal pattern

BFS is intuitive because it processes the tree level by level, making it easy to understand which nodes are siblings and which are parent-child. We process all nodes at depth 0, then depth 1, then depth 2, and so on. When deserializing, we can assign children to parents in the same level-by-level manner.

The string format `"1,2,3,#,#,4,5"` tells us: root is 1, its children are 2 and 3, node 2 has no children (marked by `#,#`), node 3 has children 4 and 5. This information is sufficient to perfectly reconstruct the original tree.

## Solution Approach

The solution uses **Level Order Traversal (BFS)** to serialize and deserialize the binary tree. Here's how each part works:

### Serialization Process

1. **Initialize a queue** with the root node and an empty result list
2. **Process nodes level by level**:
   - Dequeue a node from the front
   - If the node exists, append its value to the result and enqueue both its children (even if they're `None`)
   - If the node is `None`, append `"#"` as a placeholder
3. **Join all values** with commas to create the final serialized string

```python
q = deque([root])
ans = []
while q:
    node = q.popleft()
    if node:
        ans.append(str(node.val))
        q.append(node.left)   # Add even if None
        q.append(node.right)  # Add even if None
    else:
        ans.append("#")
return ",".join(ans)
```

### Deserialization Process

1. **Split the string** by commas to get an array of values
2. **Create the root node** from the first value
3. **Use a queue** to track nodes that need children assigned
4. **Process values in pairs** (left child, right child):
   - For each parent node in the queue
   - Read the next two values from the array
   - If a value is not `"#"`, create a new node and add it to the queue
   - Assign the nodes as left and right children

```python
vals = data.split(",")
root = TreeNode(int(vals[0]))
q = deque([root])
i = 1
while q:
    node = q.popleft()
    # Process left child
    if vals[i] != "#":
        node.left = TreeNode(int(vals[i]))
        q.append(node.left)
    i += 1
    # Process right child
    if vals[i] != "#":
        node.right = TreeNode(int(vals[i]))
        q.append(node.right)
    i += 1
```

### Key Data Structures

- **Queue (deque)**: Used for level-order traversal in both serialization and deserialization
- **List**: Temporarily stores node values during serialization
- **String**: Final serialized format with comma-separated values

### Why This Works

The algorithm maintains a one-to-one correspondence between the serialization and deserialization process:
- During serialization, we process parent nodes before children and record nulls
- During deserialization, we read values in the same order and reconstruct relationships
- The queue ensures we process nodes level by level in both operations
- The `"#"` markers preserve the tree structure by indicating where nodes are missing

This approach has **O(n)** time complexity for both operations (visiting each node once) and **O(n)** space complexity (for the queue and result string).

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach. Consider this binary tree:

```
    1
   / \
  2   3
 /
4
```

**Serialization Process:**

1. **Initialize**: Queue = [1], Result = []

2. **Level 0 (root)**: 
   - Dequeue node 1
   - Add "1" to result → Result = ["1"]
   - Enqueue its children → Queue = [2, 3]

3. **Level 1**: 
   - Dequeue node 2
   - Add "2" to result → Result = ["1", "2"]
   - Enqueue its children → Queue = [3, 4, None]
 
   - Dequeue node 3
   - Add "3" to result → Result = ["1", "2", "3"]
   - Enqueue its children → Queue = [4, None, None, None]

4. **Level 2**:
   - Dequeue node 4
   - Add "4" to result → Result = ["1", "2", "3", "4"]
   - Enqueue its children → Queue = [None, None, None, None, None]
 
   - Dequeue None (from node 2's right)
   - Add "#" to result → Result = ["1", "2", "3", "4", "#"]
   - No children to enqueue → Queue = [None, None, None, None]

5. **Continue processing remaining Nones**:
   - Result = ["1", "2", "3", "4", "#", "#", "#", "#", "#"]

6. **Final string**: `"1,2,3,4,#,#,#,#,#"`

**Deserialization Process:**

1. **Split string**: vals = ["1", "2", "3", "4", "#", "#", "#", "#", "#"]

2. **Create root**: node(1), Queue = [node(1)], index i = 1

3. **Process node(1)'s children**:
   - vals[1] = "2" → Create node(2) as left child
   - vals[2] = "3" → Create node(3) as right child
   - Queue = [node(2), node(3)], i = 3

4. **Process node(2)'s children**:
   - vals[3] = "4" → Create node(4) as left child
   - vals[4] = "#" → No right child
   - Queue = [node(3), node(4)], i = 5

5. **Process node(3)'s children**:
   - vals[5] = "#" → No left child
   - vals[6] = "#" → No right child
   - Queue = [node(4)], i = 7

6. **Process node(4)'s children**:
   - vals[7] = "#" → No left child
   - vals[8] = "#" → No right child
   - Queue = [], i = 9

7. **Result**: The original tree is reconstructed:
```
    1
   / \
  2   3
 /
4
```

The key insight is that we process nodes in the same order during both serialization and deserialization, maintaining the parent-child relationships through the level-order traversal pattern.

## Python Solution

```python
from collections import deque
from typing import Optional

# Definition for a binary tree node.
class TreeNode:
    def __init__(self, x: int):
        self.val = x
        self.left: Optional['TreeNode'] = None
        self.right: Optional['TreeNode'] = None


class Codec:
    def serialize(self, root: Optional[TreeNode]) -> str:
        """
        Encodes a tree to a single string using level-order traversal.
      
        :type root: TreeNode
        :rtype: str
        """
        # Handle empty tree case
        if root is None:
            return ""
      
        # Initialize queue for BFS traversal
        queue = deque([root])
        result = []
      
        # Perform level-order traversal
        while queue:
            node = queue.popleft()
          
            if node:
                # Add node value to result
                result.append(str(node.val))
                # Add children to queue (even if None)
                queue.append(node.left)
                queue.append(node.right)
            else:
                # Use "#" to represent None nodes
                result.append("#")
      
        # Join all values with comma separator
        return ",".join(result)

    def deserialize(self, data: str) -> Optional[TreeNode]:
        """
        Decodes the encoded string back to a binary tree.
      
        :type data: str
        :rtype: TreeNode
        """
        # Handle empty string case
        if not data:
            return None
      
        # Split the string into individual values
        values = data.split(",")
      
        # Create root node from first value
        root = TreeNode(int(values[0]))
        queue = deque([root])
        index = 1
      
        # Reconstruct tree using BFS
        while queue:
            node = queue.popleft()
          
            # Process left child
            if values[index] != "#":
                node.left = TreeNode(int(values[index]))
                queue.append(node.left)
            index += 1
          
            # Process right child
            if values[index] != "#":
                node.right = TreeNode(int(values[index]))
                queue.append(node.right)
            index += 1
      
        return root


# Your Codec object will be instantiated and called as such:
# codec = Codec()
# codec.deserialize(codec.serialize(root))
```

## Time and Space Complexity

**Time Complexity:** `O(n)`

For the `serialize` method:
- We traverse each node in the tree exactly once using BFS (level-order traversal)
- For each node, we perform constant time operations: appending to the queue, appending to the result list, and string conversion
- The final `join` operation takes `O(m)` where `m` is the total number of elements in the list, which is at most `2n + 1` (including null nodes)
- Overall: `O(n)`

For the `deserialize` method:
- We process each value in the serialized string exactly once
- For each non-null value, we create a node and add it to the queue in constant time
- The `split` operation takes `O(k)` where `k` is the length of the string, which is proportional to `n`
- Overall: `O(n)`

**Space Complexity:** `O(n)`

For the `serialize` method:
- The queue stores at most `O(n)` nodes (in the worst case of a complete binary tree, the last level contains approximately `n/2` nodes)
- The result list `ans` stores all nodes plus null markers, which is `O(n)`
- The final string representation also takes `O(n)` space
- Overall: `O(n)`

For the `deserialize` method:
- The `vals` array after splitting stores `O(n)` elements
- The queue stores at most `O(n)` nodes
- The reconstructed tree itself contains `n` nodes, taking `O(n)` space
- Overall: `O(n)`

Where `n` is the number of nodes in the binary tree.

## Common Pitfalls

### 1. **Not Handling Empty Tree Edge Case**
A frequent mistake is forgetting to handle the case when the input tree is `None`/empty. This causes errors during serialization or deserialization.

**Problem Code:**
```python
def serialize(self, root):
    queue = deque([root])  # Will add None to queue
    result = []
    while queue:
        node = queue.popleft()
        result.append(str(node.val))  # Error: None has no attribute 'val'
```

**Solution:**
Always check for empty tree at the beginning of both methods:
```python
def serialize(self, root):
    if root is None:
        return ""
    # ... rest of the code

def deserialize(self, data):
    if not data:
        return None
    # ... rest of the code
```

### 2. **Index Out of Bounds in Deserialization**
When deserializing, accessing array indices without proper bounds checking can cause IndexError, especially with trailing null nodes.

**Problem Code:**
```python
while queue:
    node = queue.popleft()
    # Might exceed array bounds
    if values[index] != "#":  # IndexError if index >= len(values)
        node.left = TreeNode(int(values[index]))
    index += 1
```

**Solution:**
Add bounds checking or ensure serialization doesn't include unnecessary trailing nulls:
```python
while queue and index < len(values):
    node = queue.popleft()
    if index < len(values) and values[index] != "#":
        node.left = TreeNode(int(values[index]))
    index += 1
    if index < len(values) and values[index] != "#":
        node.right = TreeNode(int(values[index]))
    index += 1
```

### 3. **Forgetting to Add None Children to Queue During Serialization**
A critical mistake is only adding non-null children to the queue during serialization, which breaks the position mapping needed for deserialization.

**Problem Code:**
```python
def serialize(self, root):
    while queue:
        node = queue.popleft()
        if node:
            result.append(str(node.val))
            if node.left:  # Wrong: skips None nodes
                queue.append(node.left)
            if node.right:  # Wrong: skips None nodes
                queue.append(node.right)
```

**Solution:**
Always add both children to the queue, even if they're None:
```python
if node:
    result.append(str(node.val))
    queue.append(node.left)   # Add even if None
    queue.append(node.right)  # Add even if None
else:
    result.append("#")
```

### 4. **Using Wrong Delimiter or Conflicting with Node Values**
If node values can contain the delimiter character (e.g., negative numbers with "-" delimiter), the deserialization will fail.

**Problem Code:**
```python
# If using space as delimiter but values might contain spaces
return " ".join(result)  # Problematic if values have spaces

# Or not handling negative numbers properly
values = data.split("-")  # Will split "-5" incorrectly
```

**Solution:**
Use a delimiter that won't appear in node values (comma is safe for integers):
```python
return ",".join(result)  # Safe for integer values
# For more complex data, consider using JSON or escape sequences
```

### 5. **Inefficient String Concatenation**
Building the serialized string with repeated concatenation instead of joining a list causes O(n²) time complexity.

**Problem Code:**
```python
def serialize(self, root):
    result = ""
    while queue:
        node = queue.popleft()
        if node:
            result += str(node.val) + ","  # Inefficient string concatenation
        else:
            result += "#,"
```

**Solution:**
Collect values in a list and join once at the end:
```python
result = []
while queue:
    # ... append to list
    result.append(str(node.val))
return ",".join(result)  # Single join operation
```

