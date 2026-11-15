# 432. All O`one Data Structure

> Source: https://algo.monster/liteproblems/432

## Problem Statement

This problem asks you to design a data structure called `AllOne` that maintains string counts and can efficiently retrieve strings with minimum and maximum counts.

The data structure needs to support the following operations:

1. **`AllOne()`**: Initialize an empty data structure.

2. **`inc(String key)`**: Increase the count of string `key` by 1. If `key` doesn't exist yet, add it with count 1.

3. **`dec(String key)`**: Decrease the count of string `key` by 1. If the count reaches 0, remove the string from the data structure. The problem guarantees that `key` exists when `dec` is called.

4. **`getMaxKey()`**: Return any string that has the maximum count. Return `""` if the data structure is empty.

5. **`getMinKey()`**: Return any string that has the minimum count. Return `""` if the data structure is empty.

**Critical Constraint**: All operations must run in `O(1)` average time complexity.

The challenge lies in maintaining the ability to track both minimum and maximum counts while supporting increment and decrement operations, all in constant time. A simple hash map wouldn't suffice since finding min/max would require `O(n)` time. The solution uses a doubly-linked list where each node represents a count value and contains all strings with that count, combined with a hash map for `O(1)` access to each string's node. This allows all operations to maintain `O(1)` complexity by organizing strings by their counts in sorted order within the linked list structure.

## Intuition

The key insight starts with recognizing that we need `O(1)` access to both minimum and maximum count strings. A simple hash map `{key: count}` gives us `O(1)` increment/decrement but finding min/max would require scanning all entries in `O(n)` time.

To achieve `O(1)` min/max retrieval, we need to maintain some ordering. But traditional sorted structures like balanced trees give us `O(log n)` operations, not `O(1)`.

The breakthrough comes from observing that counts change incrementally - when we increment a key, its count goes from `n` to `n+1`, and when we decrement, it goes from `n` to `n-1`. We never jump directly from count 2 to count 10.

This incremental nature suggests using a doubly-linked list where each node represents a specific count value (1, 2, 3, ...) and maintains all strings having that count. The nodes are ordered by count value.

Why does this work for `O(1)`?
- When incrementing a string with count `n`, we only need to check if a node with count `n+1` exists right after the current node. If it does, move the string there; if not, create a new node.
- Similarly for decrement, we check for count `n-1` node right before.
- The min count strings are always in the first node after the dummy head.
- The max count strings are always in the last node before the dummy head.

We maintain a hash map `{key: node}` to directly access which node contains each string in `O(1)`. The doubly-linked list with a circular dummy head makes insertion/removal operations clean and avoids edge cases.

Each count value appears at most once in the list, and strings with the same count are grouped together in a set within their node. When a node's set becomes empty (no strings with that count), we remove the node entirely.

## Solution Approach

The solution uses a doubly-linked list of nodes where each node represents a unique count value and stores all strings with that count. Additionally, a hash map tracks which node each string belongs to.

**Data Structures:**

1. **Node Class**: Each node contains:
   - `cnt`: The count value this node represents
   - `keys`: A set of all strings with this count
   - `prev` and `next`: Pointers for the doubly-linked list
   - Helper methods `insert()` to add a new node after this one, and `remove()` to unlink this node

2. **AllOne Class**: Contains:
   - `root`: A dummy sentinel node that makes the list circular (`root.next` points to min count node, `root.prev` points to max count node)
   - `nodes`: A hash map `{string: node}` for `O(1)` access to each string's current node

**Implementation Details:**

**`inc(key)`:**
1. If `key` is new:
   - Check if the first node after root exists and has count 1
   - If yes, add `key` to that node's set
   - If no, create a new node with count 1 and insert it after root
2. If `key` exists:
   - Get current node from `nodes[key]`
   - Check if next node has count `curr.cnt + 1`
   - If yes, move `key` to that node
   - If no, create a new node with count `curr.cnt + 1` and insert it
   - Remove `key` from current node's set
   - If current node becomes empty, remove it from the list

**`dec(key)`:**
1. Get current node from `nodes[key]`
2. If count becomes 0 (curr.cnt == 1):
   - Remove `key` from `nodes` map entirely
3. Otherwise:
   - Check if previous node has count `curr.cnt - 1`
   - If yes, move `key` to that node
   - If no, create a new node with count `curr.cnt - 1` and insert it
4. Remove `key` from current node's set
5. If current node becomes empty, remove it from the list

**`getMinKey()` and `getMaxKey()`:**
- Min: Return any key from `root.next.keys` (first node after dummy)
- Max: Return any key from `root.prev.keys` (last node before dummy)
- Both use `next(iter(set))` to get an arbitrary element from the set in `O(1)`

The circular dummy node elegantly handles edge cases - when the list is empty, `root.next == root.prev == root`, and both min/max operations return empty string as required.

## Example Walkthrough

Let's walk through a sequence of operations to see how the data structure maintains O(1) operations:

**Initial state**: Empty structure with just the dummy root node pointing to itself.
```
root ↔ root (circular)
nodes = {}
```

**Operation 1: `inc("apple")`**
- "apple" is new, need a node with count 1
- Create new node with count=1, insert after root
- Add "apple" to this node's key set
```
root ↔ [count:1, keys:{"apple"}] ↔ root
nodes = {"apple": node1}
```

**Operation 2: `inc("banana")`**
- "banana" is new, check if first node has count 1 (yes!)
- Add "banana" to existing count=1 node
```
root ↔ [count:1, keys:{"apple","banana"}] ↔ root
nodes = {"apple": node1, "banana": node1}
```

**Operation 3: `inc("apple")`**
- "apple" exists in count=1 node
- Check if next node has count=2 (no, next is root)
- Create new node with count=2, insert after count=1 node
- Move "apple" from count=1 to count=2
```
root ↔ [count:1, keys:{"banana"}] ↔ [count:2, keys:{"apple"}] ↔ root
nodes = {"apple": node2, "banana": node1}
```

**Operation 4: `getMinKey()`**
- Return any key from root.next (first node)
- Returns "banana" (count=1)

**Operation 5: `getMaxKey()`**
- Return any key from root.prev (last node)
- Returns "apple" (count=2)

**Operation 6: `dec("banana")`**
- "banana" is in count=1 node
- Count becomes 0, so remove "banana" entirely
- Count=1 node becomes empty, remove it from list
```
root ↔ [count:2, keys:{"apple"}] ↔ root
nodes = {"apple": node2}
```

**Operation 7: `inc("apple")`**
- "apple" is in count=2 node
- Check if next node has count=3 (no, next is root)
- Create new node with count=3, insert after count=2 node
- Move "apple" to count=3, count=2 node becomes empty and is removed
```
root ↔ [count:3, keys:{"apple"}] ↔ root
nodes = {"apple": node3}
```

Notice how each operation only involves:
- Looking up a key in the hash map: O(1)
- Checking immediate neighbors (prev/next): O(1)
- Adding/removing from a set: O(1)
- Inserting/removing a node: O(1)

The doubly-linked list maintains counts in sorted order, but we never traverse it - we only look at immediate neighbors when incrementing/decrementing, and at the ends for min/max operations.

## Python Solution

```python
class Node:
    """
    Represents a node in the doubly-linked list.
    Each node stores a count value and all keys that have that count.
    """
    def __init__(self, key: str = '', count: int = 0):
        self.prev = None  # Previous node in the linked list
        self.next = None  # Next node in the linked list
        self.count = count  # The count value this node represents
        self.keys = {key} if key else set()  # Set of keys with this count
  
    def insert_after(self, new_node: 'Node') -> 'Node':
        """
        Insert a new node right after the current node.
        Returns the newly inserted node.
        """
        new_node.prev = self
        new_node.next = self.next
        self.next.prev = new_node
        self.next = new_node
        return new_node
  
    def remove(self) -> None:
        """
        Remove this node from the linked list by connecting 
        its previous and next nodes directly.
        """
        self.prev.next = self.next
        self.next.prev = self.prev


class AllOne:
    """
    A data structure that maintains string keys with their counts.
    Supports incrementing/decrementing counts and retrieving keys with min/max counts.
    All operations run in O(1) time.
    """
    def __init__(self):
        # Create a sentinel root node for the circular doubly-linked list
        self.root = Node()
        self.root.next = self.root
        self.root.prev = self.root
      
        # Map each key to its corresponding node in the linked list
        self.key_to_node = {}
  
    def inc(self, key: str) -> None:
        """
        Increment the count of the given key by 1.
        If the key doesn't exist, initialize it with count 1.
        """
        if key not in self.key_to_node:
            # Key doesn't exist, need to add it with count 1
            if self.root.next == self.root or self.root.next.count > 1:
                # No node with count 1 exists, create a new one
                new_node = Node(key, 1)
                self.key_to_node[key] = self.root.insert_after(new_node)
            else:
                # Node with count 1 already exists, add key to it
                self.root.next.keys.add(key)
                self.key_to_node[key] = self.root.next
        else:
            # Key exists, increment its count
            current_node = self.key_to_node[key]
            next_node = current_node.next
            new_count = current_node.count + 1
          
            # Check if we need to create a new node for the incremented count
            if next_node == self.root or next_node.count > new_count:
                # Create new node with incremented count
                new_node = Node(key, new_count)
                self.key_to_node[key] = current_node.insert_after(new_node)
            else:
                # Node with the target count already exists
                next_node.keys.add(key)
                self.key_to_node[key] = next_node
          
            # Remove key from current node
            current_node.keys.discard(key)
            if not current_node.keys:
                # If node has no keys left, remove it from the list
                current_node.remove()
  
    def dec(self, key: str) -> None:
        """
        Decrement the count of the given key by 1.
        If the count reaches 0, remove the key entirely.
        Assumes the key exists in the data structure.
        """
        current_node = self.key_to_node[key]
      
        if current_node.count == 1:
            # Count will become 0, remove the key entirely
            del self.key_to_node[key]
        else:
            # Decrement the count
            prev_node = current_node.prev
            new_count = current_node.count - 1
          
            # Check if we need to create a new node for the decremented count
            if prev_node == self.root or prev_node.count < new_count:
                # Create new node with decremented count
                new_node = Node(key, new_count)
                self.key_to_node[key] = prev_node.insert_after(new_node)
            else:
                # Node with the target count already exists
                prev_node.keys.add(key)
                self.key_to_node[key] = prev_node
      
        # Remove key from current node
        current_node.keys.discard(key)
        if not current_node.keys:
            # If node has no keys left, remove it from the list
            current_node.remove()
  
    def getMaxKey(self) -> str:
        """
        Return a key with the maximum count.
        If multiple keys have the maximum count, return any one of them.
        Returns empty string if no keys exist.
        """
        if self.root.prev == self.root:
            return ""
        # The last node (root.prev) contains keys with maximum count
        return next(iter(self.root.prev.keys))
  
    def getMinKey(self) -> str:
        """
        Return a key with the minimum count.
        If multiple keys have the minimum count, return any one of them.
        Returns empty string if no keys exist.
        """
        if self.root.next == self.root:
            return ""
        # The first node (root.next) contains keys with minimum count
        return next(iter(self.root.next.keys))


# Your AllOne object will be instantiated and called as such:
# obj = AllOne()
# obj.inc(key)
# obj.dec(key)
# param_3 = obj.getMaxKey()
# param_4 = obj.getMinKey()
```

## Time and Space Complexity

**Time Complexity:**
- `inc(key)`: `O(1)` - The operation involves hash table lookups (`O(1)`), set operations (`add` and `discard` are `O(1)`), and doubly linked list insertion/removal (`O(1)`). All operations are constant time.
- `dec(key)`: `O(1)` - Similar to `inc`, this involves hash table lookups, set operations, and doubly linked list manipulation, all of which are constant time operations.
- `getMaxKey()`: `O(1)` - Directly accesses the last node in the doubly linked list (`self.root.prev`) and retrieves an arbitrary key from its set using `next(iter())`, which is `O(1)`.
- `getMinKey()`: `O(1)` - Directly accesses the first node in the doubly linked list (`self.root.next`) and retrieves an arbitrary key from its set using `next(iter())`, which is `O(1)`.

**Space Complexity:**
- `O(n)` where `n` is the number of unique keys in the data structure.
- The `nodes` dictionary stores `n` key-node mappings.
- The doubly linked list contains at most `n` nodes (one for each unique count value, which is bounded by the number of keys).
- Each node stores a set of keys with the same count, and the total number of keys across all sets is `n`.
- Therefore, the overall space complexity is `O(n)`.

## Common Pitfalls

### 1. **Incorrectly Handling Node Insertion Order**
A critical pitfall is inserting new nodes in the wrong position when incrementing or decrementing counts. The doubly-linked list must maintain strict ordering where nodes are sorted by count value (ascending from root.next to root.prev).

**Pitfall Example:**
```python
# WRONG: Inserting after the wrong node
if next_node == self.root or next_node.count > new_count:
    new_node = Node(key, new_count)
    # Bug: inserting after next_node instead of current_node
    self.key_to_node[key] = next_node.insert_after(new_node)
```

**Solution:**
Always insert the new node immediately after the current node when incrementing, or after the previous node when decrementing:
```python
# CORRECT: Insert after current_node when incrementing
if next_node == self.root or next_node.count > new_count:
    new_node = Node(key, new_count)
    self.key_to_node[key] = current_node.insert_after(new_node)
```

### 2. **Forgetting to Update the key_to_node Mapping**
When moving a key to a different node, failing to update the `key_to_node` dictionary will cause subsequent operations on that key to reference the wrong node.

**Pitfall Example:**
```python
# WRONG: Not updating the mapping
if next_node.count == new_count:
    next_node.keys.add(key)
    # Bug: forgot to update key_to_node[key]
```

**Solution:**
Always update the mapping whenever a key moves to a different node:
```python
# CORRECT: Update the mapping
if next_node.count == new_count:
    next_node.keys.add(key)
    self.key_to_node[key] = next_node  # Essential update
```

### 3. **Memory Leak from Empty Nodes**
Not removing empty nodes from the linked list causes a memory leak and breaks the invariant that each node should contain at least one key.

**Pitfall Example:**
```python
# WRONG: Not removing empty nodes
current_node.keys.discard(key)
# Bug: forgot to check if node is now empty
```

**Solution:**
Always check and remove nodes that become empty after removing a key:
```python
# CORRECT: Remove empty nodes
current_node.keys.discard(key)
if not current_node.keys:
    current_node.remove()
```

### 4. **Edge Case in dec() When Count Becomes Zero**
A subtle pitfall is not properly handling the case when decrementing a key with count 1. The key should be completely removed from the data structure, not moved to a "count 0" node.

**Pitfall Example:**
```python
# WRONG: Trying to create a node with count 0
def dec(self, key: str) -> None:
    current_node = self.key_to_node[key]
    new_count = current_node.count - 1  # Could be 0!
    # Bug: might create a node with count 0
    new_node = Node(key, new_count)
```

**Solution:**
Handle count 1 as a special case:
```python
# CORRECT: Remove key entirely when count becomes 0
def dec(self, key: str) -> None:
    current_node = self.key_to_node[key]
  
    if current_node.count == 1:
        # Remove the key entirely
        del self.key_to_node[key]
    else:
        # Normal decrement logic
        new_count = current_node.count - 1
        # ... rest of the logic
```

### 5. **Calling getMinKey/getMaxKey on an Empty Structure**
Attempting to access `root.next.keys` or `root.prev.keys` when the structure is empty (root points to itself) raises `StopIteration`, because `next(iter(...))` is called on the root's empty key set.

**Pitfall Example:**
```python
# WRONG: Accessing keys without checking if structure is empty
def getMinKey(self) -> str:
    # Bug: root.next might be root itself (empty structure)
    return next(iter(self.root.next.keys))
```

**Solution:**
Always check if the structure is empty first:
```python
# CORRECT: Check for empty structure
def getMinKey(self) -> str:
    if self.root.next == self.root:
        return ""
    return next(iter(self.root.next.keys))
```

