# 716. Max Stack

> Source: https://algo.monster/liteproblems/716

## Problem Statement

You need to design a max [stack](/problems/stack_intro) data structure that combines the functionality of a regular stack with the ability to efficiently retrieve and remove the maximum element.

The `MaxStack` class should support the following operations:

1. **`MaxStack()`**: Initialize an empty [stack](/problems/stack_intro) object.

2. **`push(int x)`**: Add element `x` to the top of the [stack](/problems/stack_intro).

3. **`pop()`**: Remove and return the element from the top of the [stack](/problems/stack_intro) (most recently added element).

4. **`top()`**: Return the element at the top of the [stack](/problems/stack_intro) without removing it.

5. **`peekMax()`**: Return the maximum element currently in the [stack](/problems/stack_intro) without removing it.

6. **`popMax()`**: Remove and return the maximum element from the [stack](/problems/stack_intro). If multiple elements have the same maximum value, remove the one that is closest to the top of the stack (the most recently added among the maximum elements).

**Performance Requirements:**
- The `top()` operation must run in `O(1)` time
- All other operations (`push`, `pop`, `peekMax`, `popMax`) must run in `O(log n)` time

The challenge is to maintain both [stack](/problems/stack_intro) ordering (LIFO - Last In First Out) and efficient access to the maximum element. When `popMax()` is called, you need to be able to remove an element that might not be at the top of the stack while preserving the relative order of all other elements.

For example, if you push `[5, 1, 5]` in that order, the [stack](/problems/stack_intro) from bottom to top is `[5, 1, 5]`. Calling `popMax()` should remove and return the top `5` (not the bottom one), leaving the stack as `[5, 1]`.

## Intuition

The key insight is that we need to maintain two different orderings of the same elements simultaneously: the [stack](/problems/stack_intro) order (insertion order) and the value order (for finding maximum). This immediately suggests using two data structures working together.

For a regular [stack](/problems/stack_intro), we could use a simple array or [linked list](/problems/linked_list_cycle). However, since `popMax()` requires removing an element from potentially anywhere in the stack (not just the top), we need a data structure that supports efficient removal from any position. This leads us to consider a doubly-linked list, where any node can be removed in `O(1)` time if we have a reference to it.

The second challenge is maintaining quick access to the maximum element. A sorted container like a balanced BST or a sorted list would allow us to find the maximum in `O(1)` or `O(log n)` time. But here's the clever part: instead of storing values in both structures separately, we can store **node references** in the sorted structure that point to nodes in our doubly-[linked list](/problems/linked_list_cycle).

This creates a beautiful synergy:
- When we `push`, we create a node in the doubly-[linked list](/problems/linked_list_cycle) and add a reference to that same node in our sorted structure
- When we `pop`, we remove the node from the tail of the linked list and also remove its reference from the sorted structure
- When we `popMax()`, we get the node reference from the sorted structure, remove it from there, and also unlink it from the doubly-linked list

The sorted structure (using Python's `SortedList`) maintains nodes sorted by their values, so the maximum is always at the end. When multiple nodes have the same maximum value, the `SortedList` maintains them in insertion order, ensuring that `popMax()` naturally removes the most recently added maximum.

This design elegantly solves the problem of maintaining both orderings while allowing efficient operations on both ends - [stack](/problems/stack_intro) operations on the [linked list](/problems/linked_list_cycle) side and max operations through the sorted structure.

**Learn more about [Stack](/problems/stack_intro) and [Linked List](/problems/linked_list_cycle) patterns.**

## Solution Approach

The solution uses a combination of a **doubly-[linked list](/problems/linked_list_cycle)** and a **sorted list** to achieve the required time complexities.

### Data Structure Design

**1. Node Class:**
```python
class Node:
    def __init__(self, val=0):
        self.val = val
        self.prev = None
        self.next = None
```
Each node stores a value and pointers to both previous and next nodes, enabling bidirectional traversal and `O(1)` removal.

**2. DoubleLinkedList Class:**
The doubly-[linked list](/problems/linked_list_cycle) uses sentinel nodes (dummy head and tail) to simplify edge cases:
- `head` and `tail` are dummy nodes that always exist
- Actual elements are inserted between `head` and `tail`
- The [stack](/problems/stack_intro)'s top is always at `tail.prev`

Key methods:
- `append(val)`: Creates a new node and inserts it just before the tail. Returns the node reference for use in the sorted list.
- `remove(node)`: Unlinks a node from the list by updating its neighbors' pointers (`node.prev.next = node.next` and `node.next.prev = node.prev`)
- `pop()`: Removes and returns the node at `tail.prev` (top of [stack](/problems/stack_intro))
- `peek()`: Returns the value at `tail.prev` without removal

**3. MaxStack Class:**
Combines two structures:
- `self.stk`: A `DoubleLinkedList` maintaining [stack](/problems/stack_intro) order
- `self.sl`: A `SortedList` containing node references, sorted by node values

### Operation Implementation

**`push(x)`:**
1. Create and append a new node to the doubly-[linked list](/problems/linked_list_cycle)
2. Add the same node reference to the sorted list
3. The sorted list maintains nodes in ascending order by their values

**`pop()`:**
1. Remove the last node from the doubly-[linked list](/problems/linked_list_cycle) ([stack](/problems/stack_intro) top)
2. Remove the same node reference from the sorted list
3. Return the node's value

**`top()`:**
Simply return `tail.prev.val` from the doubly-[linked list](/problems/linked_list_cycle) - this is `O(1)` since we're just accessing a value.

**`peekMax()`:**
Return the value of the last node in the sorted list (`self.sl[-1].val`). Since the list is sorted, the maximum is always at the end.

**`popMax()`:**
1. Pop the last node from the sorted list (the maximum)
2. Remove this same node from the doubly-[linked list](/problems/linked_list_cycle) using the static `remove` method
3. Return the node's value

### Time Complexity Analysis

- `top()`: `O(1)` - Direct access to the tail's previous node
- `push()`: `O(log n)` - Dominated by sorted list insertion
- `pop()`: `O(log n)` - Dominated by sorted list removal
- `peekMax()`: `O(1)` - Direct access to the last element in sorted list
- `popMax()`: `O(log n)` - Sorted list removal is `O(log n)`, doubly-[linked list](/problems/linked_list_cycle) removal is `O(1)`

The key insight is that by storing node references rather than values in the sorted list, we can maintain both orderings efficiently and perform cross-structure operations seamlessly.

## Example Walkthrough

Let's walk through a sequence of operations to see how the two data structures work together:

**Initial State:**
- Doubly-linked list: `[head] ↔ [tail]` (empty)
- Sorted list: `[]` (empty)

**Operation 1: `push(3)`**
1. Create a new node with value 3
2. Insert it into the doubly-linked list between head and tail
3. Add the node reference to the sorted list

- Doubly-linked list: `[head] ↔ [Node(3)] ↔ [tail]`
- Sorted list: `[Node(3)]` (sorted by value)

**Operation 2: `push(5)`**
1. Create a new node with value 5
2. Insert it before tail (after Node(3))
3. Add to sorted list, which maintains sorted order

- Doubly-linked list: `[head] ↔ [Node(3)] ↔ [Node(5)] ↔ [tail]`
- Sorted list: `[Node(3), Node(5)]` (sorted by value)

**Operation 3: `push(1)`**
1. Create Node(1) and insert before tail
2. Sorted list inserts it at the beginning due to its value

- Doubly-linked list: `[head] ↔ [Node(3)] ↔ [Node(5)] ↔ [Node(1)] ↔ [tail]`
- Sorted list: `[Node(1), Node(3), Node(5)]` (sorted by value)
- Stack order (bottom to top): [3, 5, 1]

**Operation 4: `peekMax()`**
- Look at the last element in sorted list: `Node(5)`
- Return its value: 5
- No changes to either structure

**Operation 5: `popMax()`**
1. Remove last node from sorted list: `Node(5)`
2. Use the node reference to unlink it from the doubly-linked list
3. Return value 5

- Doubly-linked list: `[head] ↔ [Node(3)] ↔ [Node(1)] ↔ [tail]`
- Sorted list: `[Node(1), Node(3)]`
- Stack now contains: [3, 1]

**Operation 6: `top()`**
- Access `tail.prev` which is `Node(1)`
- Return value: 1
- This is O(1) as we're just following a pointer

**Operation 7: `push(5)` again**
- Create new Node(5) and add to both structures

- Doubly-linked list: `[head] ↔ [Node(3)] ↔ [Node(1)] ↔ [Node(5)] ↔ [tail]`
- Sorted list: `[Node(1), Node(3), Node(5)]`

**Operation 8: `push(5)` once more**
- Another Node(5) is created (different node object, same value)

- Doubly-linked list: `[head] ↔ [Node(3)] ↔ [Node(1)] ↔ [Node(5)₁] ↔ [Node(5)₂] ↔ [tail]`
- Sorted list: `[Node(1), Node(3), Node(5)₁, Node(5)₂]`
- Note: The sorted list maintains insertion order for equal values

**Operation 9: `popMax()`**
- Removes `Node(5)₂` (the most recently added 5)
- This demonstrates that when multiple maximums exist, we remove the one closest to the top

- Doubly-linked list: `[head] ↔ [Node(3)] ↔ [Node(1)] ↔ [Node(5)₁] ↔ [tail]`
- Sorted list: `[Node(1), Node(3), Node(5)₁]`

The beauty of this approach is that both structures share the same node objects. When we remove a node from one structure, we can immediately locate and remove it from the other using the node reference, maintaining consistency between both orderings efficiently.

## Python Solution

```python
from typing import Optional
from sortedcontainers import SortedList


class Node:
    """Node for doubly linked list."""

    def __init__(self, val: int = 0) -> None:
        self.val: int = val
        self.prev: Optional['Node'] = None
        self.next: Optional['Node'] = None


class DoublyLinkedList:
    """Doubly linked list with sentinel nodes for efficient operations."""

    def __init__(self) -> None:
        # Create sentinel head and tail nodes
        self.head: Node = Node()
        self.tail: Node = Node()
        # Connect sentinels
        self.head.next = self.tail
        self.tail.prev = self.head

    def append(self, val: int) -> Node:
        """
        Append a new node with given value to the end of the list.
        Returns the newly created node.
        """
        node = Node(val)
        # Insert before tail sentinel
        node.next = self.tail
        node.prev = self.tail.prev
        self.tail.prev = node
        node.prev.next = node
        return node

    @staticmethod
    def remove(node: Node) -> Node:
        """
        Remove a node from the list by updating its neighbors' pointers.
        Returns the removed node.
        """
        node.prev.next = node.next
        node.next.prev = node.prev
        return node

    def pop(self) -> Node:
        """
        Remove and return the last node (before tail sentinel).
        """
        return self.remove(self.tail.prev)

    def peek(self) -> int:
        """
        Return the value of the last node without removing it.
        """
        return self.tail.prev.val


class MaxStack:
    """
    Stack data structure that supports retrieving the maximum element.
    Supports O(1) top and O(log n) push, pop, and popMax operations.
    """

    def __init__(self) -> None:
        # Doubly linked list to maintain stack order
        self.stack: DoublyLinkedList = DoublyLinkedList()
        # Sorted list to efficiently track maximum elements
        # Stores references to nodes, sorted by their values
        self.sorted_list: SortedList = SortedList(key=lambda x: x.val)

    def push(self, x: int) -> None:
        """
        Push element x onto the stack.
        """
        # Add node to the end of the stack
        node = self.stack.append(x)
        # Add node reference to sorted list for max tracking
        self.sorted_list.add(node)

    def pop(self) -> int:
        """
        Remove and return the top element from the stack.
        """
        # Remove from stack
        node = self.stack.pop()
        # Remove from sorted list
        self.sorted_list.remove(node)
        return node.val

    def top(self) -> int:
        """
        Get the top element without removing it.
        """
        return self.stack.peek()

    def peekMax(self) -> int:
        """
        Get the maximum element in the stack without removing it.
        """
        # Last element in sorted list is the maximum
        return self.sorted_list[-1].val

    def popMax(self) -> int:
        """
        Remove and return the maximum element from the stack.
        """
        # Get and remove max node from sorted list
        node = self.sorted_list.pop()
        # Remove the same node from the stack
        DoublyLinkedList.remove(node)
        return node.val


# Your MaxStack object will be instantiated and called as such:
# obj = MaxStack()
# obj.push(x)
# param_2 = obj.pop()
# param_3 = obj.top()
# param_4 = obj.peekMax()
# param_5 = obj.popMax()
```

## Time and Space Complexity

**Time Complexity:**
- `push(x)`: `O(log n)` - Appending to the doubly linked list is `O(1)`, but adding to the SortedList requires `O(log n)` for binary search insertion
- `pop()`: `O(log n)` - Removing from the tail of the doubly linked list is `O(1)`, and removing a node from the SortedList is `O(log n)` amortized
- `top()`: `O(1)` - Simply returns the value at the tail of the doubly linked list
- `peekMax()`: `O(1)` - Accessing the last element in the SortedList is constant time
- `popMax()`: `O(log n)` - Popping the last element from the SortedList is `O(log n)` amortized, and removing from the doubly linked list is `O(1)`

**Space Complexity:**
- `O(n)` where `n` is the number of elements in the stack
- The doubly linked list stores `n` nodes
- The SortedList stores references to the same `n` nodes
- Each node contains a value and two pointers (prev and next)
- Total space is `O(n) + O(n) = O(n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Storing Values Instead of Node References in SortedList**

One of the most common mistakes is storing just the values in the sorted list rather than the actual node references. This breaks the connection between the two data structures and makes it impossible to efficiently remove the correct element during `popMax()`.

**Incorrect approach:**
```python
class MaxStack:
    def __init__(self):
        self.stack = DoublyLinkedList()
        self.sorted_list = SortedList()  # Stores values, not nodes!

    def push(self, x):
        self.stack.append(x)
        self.sorted_list.add(x)  # Wrong: adding value instead of node

    def popMax(self):
        max_val = self.sorted_list.pop()
        # Problem: How do we find and remove the RIGHT occurrence
        # of max_val from the stack? If there are duplicates,
        # we might remove the wrong one!
```

**Why this fails:** When multiple elements have the same value, you can't determine which specific node to remove from the doubly-linked list. The problem requires removing the most recently added maximum (closest to top), but with only values, you'd have to traverse the entire stack to find it, breaking the O(log n) time complexity requirement.

**Solution:** Always store node references in the sorted list, using a key function for sorting:
```python
self.sorted_list = SortedList(key=lambda x: x.val)
```

### 2. **Forgetting to Handle Bidirectional Updates**

Another pitfall is updating only one data structure while forgetting the other, leading to inconsistency.

**Incorrect approach:**
```python
def pop(self):
    node = self.stack.pop()
    # Forgot to remove from sorted_list!
    return node.val
```

**Why this fails:** The sorted list will still contain a reference to a node that's no longer in the stack. Future `peekMax()` or `popMax()` operations might try to access this orphaned node, causing incorrect results or errors.

**Solution:** Always update both data structures in tandem:
```python
def pop(self):
    node = self.stack.pop()
    self.sorted_list.remove(node)  # Don't forget this!
    return node.val
```

### 3. **Using Built-in List Instead of Doubly-Linked List**

Some might try to use Python's built-in list for the stack, which seems simpler but creates problems.

**Incorrect approach:**
```python
class MaxStack:
    def __init__(self):
        self.stack = []  # Using regular list
        self.sorted_list = SortedList()

    def popMax(self):
        max_val = self.sorted_list.pop()
        # Problem: Removing from middle of list is O(n)!
        idx = self.stack.index(max_val)  # O(n) search
        return self.stack.pop(idx)  # O(n) removal
```

**Why this fails:** Removing an element from an arbitrary position in a Python list requires O(n) time because all subsequent elements must be shifted. This violates the O(log n) requirement for `popMax()`.

**Solution:** Use a doubly-linked list where node removal from any position is O(1) once you have the node reference.

### 4. **Not Handling Duplicate Values Correctly**

When the stack contains duplicate maximum values, the implementation must remove the correct one (most recently added).

**Incorrect approach:**
```python
def popMax(self):
    max_val = self.sorted_list[-1].val
    # Wrong: This might remove the first occurrence, not the last
    for node in self.sorted_list:
        if node.val == max_val:
            self.sorted_list.remove(node)
            DoublyLinkedList.remove(node)
            return max_val
```

**Why this fails:** This removes the first occurrence of the maximum value encountered in the sorted list, which might not be the most recently added one.

**Solution:** The sorted list should maintain stable ordering for equal elements (most implementations do this by default), and you should always remove from the end:
```python
def popMax(self):
    node = self.sorted_list.pop()  # Removes the last (most recent) max
    DoublyLinkedList.remove(node)
    return node.val
```

