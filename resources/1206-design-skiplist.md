# 1206. Design Skiplist

> Source: https://algo.monster/liteproblems/1206

## Problem Statement

Design and implement a **Skiplist** data structure from scratch without using any built-in libraries.

A skiplist is a probabilistic data structure that provides `O(log(n))` average time complexity for adding, erasing, and searching elements. It achieves similar performance to balanced tree structures like treaps and red-black trees but with simpler implementation logic based on linked lists.

The skiplist consists of multiple layers where:
- Each layer is a sorted [linked list](/problems/linked_list_cycle)
- The bottom layer contains all elements
- Higher layers act as "express lanes" containing fewer elements, allowing faster traversal
- Elements are promoted to higher layers randomly, creating a hierarchical structure

You need to implement the `Skiplist` class with the following methods:

1. **`Skiplist()`**: Initializes an empty skiplist object.

2. **`search(int target)`**: Returns `true` if the integer `target` exists in the skiplist, `false` otherwise.

3. **`add(int num)`**: Inserts the value `num` into the skiplist. The method should handle duplicate values.

4. **`erase(int num)`**: Removes one occurrence of the value `num` from the skiplist and returns `true`. If `num` doesn't exist in the skiplist, returns `false` without modifying the structure. When multiple instances of `num` exist, removing any single instance is acceptable.

**Important Note**: The skiplist must support duplicate values - multiple instances of the same number can exist in the data structure.

The implementation uses a probabilistic approach where each new element is assigned a random height (number of levels it appears in). This randomization ensures the expected logarithmic time complexity for operations while maintaining the simplicity of the structure compared to self-balancing trees.

## Intuition

Think of a skiplist like a multi-story building with express elevators. Instead of checking every floor (element) one by one, we can take express elevators that skip multiple floors, then switch to local elevators for fine-grained navigation.

The key insight is that we can speed up searching in a sorted [linked list](/problems/linked_list_cycle) by adding "shortcuts" at higher levels. Imagine we have a sorted linked list at the bottom level containing all elements. If we had to search for element 80 in `[30, 40, 50, 60, 70, 90]`, we'd need to traverse through all elements sequentially.

But what if we create another [linked list](/problems/linked_list_cycle) above it that contains only some elements, say `[30, 60, 90]`? Now we can first search in this sparse upper level - we quickly jump from 30 to 60, realize 80 is between them, then drop down to the lower level to search between 60 and 90. This reduces the number of comparisons.

We can extend this idea to multiple levels, where each higher level contains progressively fewer elements. The beauty is in how we decide which elements get promoted to higher levels - we use **randomization**. When inserting a new element, we flip a coin (with probability `p = 0.25`) to decide if it should also appear in the next level up. This random promotion ensures that statistically, about 1/4 of elements reach level 2, 1/16 reach level 3, and so on.

The randomization is crucial because:
1. It keeps the structure balanced probabilistically without complex rebalancing operations
2. It ensures `O(log n)` expected height since the probability of reaching level `k` is `p^k`
3. It simplifies insertion compared to deterministic balanced trees

For searching, we start from the highest level and move forward as far as possible without overshooting our target, then drop down a level and repeat. This "drop and continue" pattern naturally exploits the express lanes we've built.

The `find_closest` helper method embodies this navigation strategy - at each level, it finds the node just before where our target would be, setting us up perfectly for either finding the target (search), inserting after it (add), or removing the next node if it matches (erase).

**Learn more about [Linked List](/problems/linked_list_cycle) patterns.**

## Solution Approach

The implementation uses two classes: `Node` to represent skiplist nodes and `Skiplist` for the main data structure.

### Node Structure
Each `Node` contains:
- `val`: The integer value stored in the node
- `next`: An array of pointers where `next[i]` points to the next node at level `i`

The node's level determines how many layers it appears in. A node with level 3 will have pointers at levels 0, 1, and 2.

### Skiplist Class Constants
- `max_level = 32`: Maximum possible levels in the skiplist (prevents unbounded growth)
- `p = 0.25`: Probability of promoting a node to the next level (1/4 chance)

### Core Components

**1. Initialization**
The skiplist starts with a sentinel head node with value `-1` and `max_level` pointers. The `level` variable tracks the current maximum level being used (starts at 0).

**2. Helper Method: `find_closest(curr, level, target)`**
This method traverses horizontally at a given level to find the rightmost node whose value is less than `target`. It's the workhorse for navigation:
```python
while curr.next[level] and curr.next[level].val < target:
    curr = curr.next[level]
```

**3. Search Operation**
Starting from the highest active level, we:
- Use `find_closest` to get as close as possible to the target
- Check if the next node at this level equals the target
- If not found, drop down a level and repeat
- Return `true` if found at any level, `false` if we exhaust all levels

**4. Add Operation**
First, we randomly determine the new node's level using `random_level()`:
```python
level = 1
while level < max_level and random.random() < p:
    level += 1
```

Then, we traverse from the highest level downward:
- At each level `i`, find the insertion position using `find_closest`
- If `i < level` (node appears at this level), update pointers:
  - New node's `next[i]` points to current's `next[i]`
  - Current's `next[i]` points to the new node
- Update the skiplist's maximum level if necessary

**5. Erase Operation**
We search for the target value at each level:
- Use `find_closest` to position just before potential target
- If the next node matches the target, bypass it by updating pointers
- Set `ok = True` to indicate successful deletion
- After deletion, clean up empty top levels by reducing `self.level`

The key pattern is the top-down traversal: we always start from the highest level and work our way down, using `find_closest` to navigate horizontally before dropping levels. This ensures we take maximum advantage of the express lanes at higher levels.

### Time Complexity Analysis
- **Search**: `O(log n)` expected - we visit `O(log n)` levels and traverse `O(1)` nodes per level on average
- **Add**: `O(log n)` expected - similar to search plus `O(1)` pointer updates per level
- **Erase**: `O(log n)` expected - same traversal pattern as search with pointer updates

### Space Complexity
`O(n)` expected - each element appears in `O(log n)` levels on average, giving `n * O(log n) / n = O(n)` total space.

## Example Walkthrough

Let's walk through building a skiplist and performing operations on it. We'll use `p = 0.5` for this example (instead of 0.25) to make it easier to visualize.

**Initial State:**
```
Level 2: HEAD(-1) → None
Level 1: HEAD(-1) → None  
Level 0: HEAD(-1) → None
```

**Step 1: Add 3**
- `random_level()` returns 2 (let's say we got lucky with coin flips)
- Insert 3 at levels 0 and 1:
```
Level 2: HEAD(-1) → None
Level 1: HEAD(-1) → 3 → None
Level 0: HEAD(-1) → 3 → None
```

**Step 2: Add 7**
- `random_level()` returns 1 (only appears at level 0)
- Start from level 1: `find_closest(HEAD, 1, 7)` returns HEAD (since 3 < 7)
- At level 0: Insert 7 after 3
```
Level 2: HEAD(-1) → None
Level 1: HEAD(-1) → 3 -------→ None
Level 0: HEAD(-1) → 3 → 7 → None
```

**Step 3: Add 5**
- `random_level()` returns 3 (appears at levels 0, 1, and 2)
- Start from level 2: Insert 5 after HEAD
- At level 1: Insert 5 after 3
- At level 0: Insert 5 between 3 and 7
```
Level 2: HEAD(-1) → 5 -----------→ None
Level 1: HEAD(-1) → 3 → 5 -------→ None
Level 0: HEAD(-1) → 3 → 5 → 7 → None
```

**Step 4: Search for 7**
- Start at level 2: `find_closest(HEAD, 2, 7)` returns HEAD
  - Next node is 5 (< 7), move to 5
  - Next node is None, drop to level 1
- At level 1 (starting from node 5): Next is None, drop to level 0
- At level 0 (starting from node 5): Next is 7 - **Found!**
- Return `true`

**Step 5: Search for 6**
- Start at level 2: Move to 5, then drop (5 < 6 < None)
- At level 1 (from node 5): Next is None, drop to level 0
- At level 0 (from node 5): Next is 7 (7 > 6), so 6 is not found
- Return `false`

**Step 6: Add 5 again (duplicate)**
- `random_level()` returns 1
- Navigate to find insertion points
- Insert the second 5 after the first 5 at level 0
```
Level 2: HEAD(-1) → 5 ----------------→ None
Level 1: HEAD(-1) → 3 → 5 -----------→ None
Level 0: HEAD(-1) → 3 → 5 → 5 → 7 → None
```

**Step 7: Erase 5**
- Start at level 2: Find 5, remove it by updating HEAD.next[2] = None
- At level 1: Find 5 after 3, remove it
- At level 0: Find first 5, remove it (leaving the second 5)
```
Level 2: HEAD(-1) → None
Level 1: HEAD(-1) → 3 -----------→ None
Level 0: HEAD(-1) → 3 → 5 → 7 → None
```

**Key Observations:**

1. **Express Lane Effect**: When searching for 7, we could skip directly from HEAD to 5 at level 2, avoiding nodes 3. This demonstrates the "express lane" concept.

2. **Random Heights**: Each node got a different random height (3: level 2, 7: level 1, 5: level 3), creating a balanced structure probabilistically.

3. **Top-Down Navigation**: We always start from the highest level and work our way down, moving forward as much as possible at each level before dropping.

4. **Duplicate Handling**: The second 5 was inserted normally, and when erasing, we removed only one occurrence.

5. **Find Closest Pattern**: At each level, `find_closest` positions us at the rightmost node less than our target, setting up perfect positioning for the next operation (search check, insertion point, or deletion candidate).

This walkthrough illustrates how the skiplist achieves logarithmic performance: higher levels let us skip large portions of the list, while lower levels provide fine-grained access to all elements.

## Python Solution

```python
import random
from typing import List, Optional


class Node:
    """Node class for skip list implementation."""
    __slots__ = ['val', 'next']
  
    def __init__(self, val: int, level: int) -> None:
        """
        Initialize a node with a value and level.
      
        Args:
            val: The value stored in the node
            level: The number of forward pointers this node will have
        """
        self.val: int = val
        self.next: List[Optional['Node']] = [None] * level


class Skiplist:
    """
    Skip list data structure implementation.
  
    A probabilistic data structure that allows O(log n) search, insertion and deletion.
    Each node can have multiple forward pointers at different levels.
    """
  
    MAX_LEVEL: int = 32  # Maximum number of levels in the skip list
    PROBABILITY: float = 0.25  # Probability for determining node levels
  
    def __init__(self) -> None:
        """Initialize an empty skip list with a sentinel head node."""
        # Create head node with maximum possible level
        self.head: Node = Node(-1, self.MAX_LEVEL)
        # Current maximum level in use
        self.level: int = 0
  
    def search(self, target: int) -> bool:
        """
        Search for a target value in the skip list.
      
        Args:
            target: The value to search for
          
        Returns:
            True if the target exists, False otherwise
        """
        current: Node = self.head
      
        # Start from the highest level and move down
        for i in range(self.level - 1, -1, -1):
            # Find the node just before where target would be at this level
            current = self._find_closest_node(current, i, target)
            # Check if the next node at this level contains the target
            if current.next[i] and current.next[i].val == target:
                return True
      
        return False
  
    def add(self, num: int) -> None:
        """
        Add a number to the skip list.
      
        Args:
            num: The number to add
        """
        current: Node = self.head
        # Randomly determine the level for the new node
        node_level: int = self._generate_random_level()
        new_node: Node = Node(num, node_level)
      
        # Update the maximum level if necessary
        self.level = max(self.level, node_level)
      
        # Insert the node at all levels from top to bottom
        for i in range(self.level - 1, -1, -1):
            # Find the position to insert at this level
            current = self._find_closest_node(current, i, num)
          
            # Only update pointers for levels within the new node's range
            if i < node_level:
                new_node.next[i] = current.next[i]
                current.next[i] = new_node
  
    def erase(self, num: int) -> bool:
        """
        Remove the first occurrence of a number from the skip list.
      
        Args:
            num: The number to remove
          
        Returns:
            True if the number was found and removed, False otherwise
        """
        current: Node = self.head
        found: bool = False
      
        # Remove the node from all levels where it appears
        for i in range(self.level - 1, -1, -1):
            # Find the node just before the target at this level
            current = self._find_closest_node(current, i, num)
          
            # If the next node contains the target value, remove it
            if current.next[i] and current.next[i].val == num:
                current.next[i] = current.next[i].next[i]
                found = True
      
        # Decrease the maximum level if top levels are now empty
        while self.level > 1 and self.head.next[self.level - 1] is None:
            self.level -= 1
      
        return found
  
    def _find_closest_node(self, current: Node, level: int, target: int) -> Node:
        """
        Find the rightmost node whose value is less than the target at a given level.
      
        Args:
            current: The starting node
            level: The level to search at
            target: The target value
          
        Returns:
            The node just before where the target would be positioned
        """
        # Move forward at this level while the next value is less than target
        while current.next[level] and current.next[level].val < target:
            current = current.next[level]
      
        return current
  
    def _generate_random_level(self) -> int:
        """
        Generate a random level for a new node using geometric distribution.
      
        Returns:
            A random level between 1 and MAX_LEVEL
        """
        level: int = 1
      
        # Keep increasing level with probability PROBABILITY
        while level < self.MAX_LEVEL and random.random() < self.PROBABILITY:
            level += 1
      
        return level


# Your Skiplist object will be instantiated and called as such:
# obj = Skiplist()
# param_1 = obj.search(target)
# obj.add(num)
# param_3 = obj.erase(num)
```

## Time and Space Complexity

**Time Complexity:**

The skip list operations (`search`, `add`, and `erase`) all have an average time complexity of `O(log n)`, where `n` is the number of nodes in the skip list.

- **Search Operation:** Starting from the highest level, we traverse down through the levels. At each level, we move forward using `find_closest` until we find a node whose next element is greater than or equal to the target. With probability `p = 0.25`, each node appears at higher levels, creating approximately `log₁/ₚ(n) = log₄(n)` levels on average. At each level, we traverse `O(1/p) = O(4)` nodes on average. Therefore, the total time complexity is `O(log n) × O(1) = O(log n)`.

- **Add Operation:** Similar to search, we traverse from the highest level to find the correct insertion positions at each level. The `random_level()` function runs in `O(1)` average time (geometric distribution with `p = 0.25`). The insertion process takes `O(log n)` time to find positions and `O(1)` to update pointers at each level, resulting in `O(log n)` total time.

- **Erase Operation:** We traverse all levels from top to bottom to find and remove the target node, which takes `O(log n)` time. The additional cleanup to reduce the skip list level when the top levels become empty takes `O(log n)` in the worst case.

**Space Complexity:**

The space complexity is `O(n)`, where `n` is the number of nodes in the skip list.

- Each node is created with a random level determined by `random_level()`. The expected number of pointers per node follows a geometric distribution: `1 + p + p² + p³ + ... = 1/(1-p) = 1/(1-0.25) = 4/3`.
- With `n` nodes and an average of `4/3` pointers per node, the total space for pointers is `O(n × 4/3) = O(n)`.
- The `head` node uses `O(max_level) = O(32) = O(1)` space.
- Therefore, the overall space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Level Management During Deletion**

One of the most common mistakes is failing to properly update the skiplist's maximum level after deletion operations. When we delete nodes, the top levels might become empty, but if we don't adjust `self.level`, future operations will unnecessarily traverse these empty levels, degrading performance.

**Problematic Code:**
```python
def erase(self, num: int) -> bool:
    current = self.head
    found = False
  
    for i in range(self.level - 1, -1, -1):
        current = self._find_closest_node(current, i, num)
        if current.next[i] and current.next[i].val == num:
            current.next[i] = current.next[i].next[i]
            found = True
  
    # Forgot to clean up empty levels!
    return found
```

**Solution:**
Always check and reduce the maximum level after deletion:
```python
# After deletion, clean up empty top levels
while self.level > 1 and self.head.next[self.level - 1] is None:
    self.level -= 1
```

### 2. **Deleting All Occurrences Instead of One**

The problem specifically asks to delete only ONE occurrence when duplicates exist, but it's easy to accidentally delete all matching nodes at once.

**Problematic Code:**
```python
def erase(self, num: int) -> bool:
    current = self.head
    found = False
  
    for i in range(self.level - 1, -1, -1):
        current = self._find_closest_node(current, i, num)
        # This continues deleting even after finding the first match
        if current.next[i] and current.next[i].val == num:
            current.next[i] = current.next[i].next[i]
            found = True
```

**Solution:**
Track which specific node to delete across all levels:
```python
def erase(self, num: int) -> bool:
    current = self.head
    target_node = None
  
    # First, find the node to delete
    for i in range(self.level - 1, -1, -1):
        current = self._find_closest_node(current, i, num)
        if current.next[i] and current.next[i].val == num:
            if target_node is None:
                target_node = current.next[i]
            # Only delete if it's the same node instance
            if current.next[i] is target_node:
                current.next[i] = current.next[i].next[i]
  
    # Clean up levels...
    return target_node is not None
```

### 3. **Off-by-One Errors in Level Indexing**

Confusion between level count and level indices is common. If `level = 3`, the valid indices are 0, 1, and 2, not 1, 2, and 3.

**Problematic Code:**
```python
def __init__(self):
    self.head = Node(-1, self.MAX_LEVEL)
    self.level = 1  # Wrong! Should start at 0 for empty list
  
def add(self, num):
    node_level = self._generate_random_level()
    # Wrong range - should be (self.level - 1, -1, -1)
    for i in range(self.level, 0, -1):  
        # This will skip level 0 and might access invalid indices
```

**Solution:**
- Initialize `self.level = 0` for an empty skiplist
- Always use `range(self.level - 1, -1, -1)` to iterate from highest to lowest level
- Remember that a node with `level = k` has indices from 0 to k-1

### 4. **Not Handling the Case When Random Level Exceeds Current Maximum**

When adding a node with a level higher than the current maximum, we must update `self.level` BEFORE traversing levels.

**Problematic Code:**
```python
def add(self, num):
    node_level = self._generate_random_level()
    new_node = Node(num, node_level)
  
    # Using old self.level value - won't connect higher levels!
    for i in range(self.level - 1, -1, -1):
        # If node_level > self.level, higher levels won't be connected
        if i < node_level:
            new_node.next[i] = current.next[i]
            current.next[i] = new_node
  
    # Too late to update here
    self.level = max(self.level, node_level)
```

**Solution:**
Update the maximum level before the traversal:
```python
self.level = max(self.level, node_level)
# Now traverse using the updated level
for i in range(self.level - 1, -1, -1):
    # ...
```

