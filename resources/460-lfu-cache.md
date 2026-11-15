# 460. LFU Cache

> Source: https://algo.monster/liteproblems/460

## Problem Statement

This problem asks you to implement a Least Frequently Used (LFU) cache data structure with the following requirements:

**What is an LFU Cache?**
An LFU cache evicts the least frequently used item when the cache reaches its capacity. Each item has a use counter that tracks how many times it has been accessed.

**Required Operations:**

1. **`LFUCache(int capacity)`**: Initialize the cache with a given capacity.

2. **`get(int key)`**: 
   - If the key exists in the cache, return its value
   - If the key doesn't exist, return `-1`
   - When a key is accessed, its use counter increases by 1

3. **`put(int key, int value)`**: 
   - If the key already exists, update its value
   - If the key doesn't exist, insert it into the cache
   - When inserting a new key and the cache is at capacity, remove the least frequently used key first
   - A newly inserted key starts with a use counter of 1
   - Accessing a key (either updating or inserting) increases its use counter by 1

**Tie-Breaking Rule:**
When multiple keys have the same frequency (use counter), the Least Recently Used (LRU) among them should be evicted. This means if two keys have been used the same number of times, remove the one that was accessed least recently.

**Performance Requirement:**
Both `get` and `put` operations must run in `O(1)` average time complexity.

**Example Scenario:**
- If capacity is 2 and you insert keys A and B, both have frequency 1
- If you access A twice, A has frequency 3 and B has frequency 1
- When inserting a new key C at capacity, B gets evicted (lowest frequency)
- If both A and B had frequency 2, the one accessed least recently would be evicted

The challenge is designing a data structure that can track both frequency counts and recency of access while maintaining `O(1)` operations for get, put, and eviction.

## Intuition

To achieve `O(1)` operations for an LFU cache, we need to efficiently track two things: frequency counts and the order of access within each frequency group.

The key insight is that we need a two-level organization:
1. **First level**: Group nodes by their frequency
2. **Second level**: Within each frequency group, maintain the order of access (for LRU tie-breaking)

Think of it like organizing books on shelves - each shelf represents a frequency level (1 time used, 2 times used, etc.), and books on each shelf are ordered by when they were last touched.

**Why HashMap + Doubly Linked Lists?**

For `O(1)` access to any key, we need a HashMap that maps `key -> Node`. But we also need to quickly find and remove the least frequently used item.

The challenge is that when we need to evict, we must:
- Find the minimum frequency group instantly
- Within that group, find the least recently used item instantly
- Remove that item in `O(1)` time

This leads us to use:
- **A HashMap** (`map`): Maps each key to its node for `O(1)` lookup
- **Another HashMap** (`freq_map`): Maps each frequency to a doubly linked list of nodes with that frequency
- **Doubly Linked Lists**: For each frequency level, maintain nodes in LRU order (most recent at head, least recent at tail)

**Why track `min_freq`?**

When the cache is full and we need to evict, we must quickly identify which frequency group has the lowest count. Instead of searching through all frequencies, we maintain a `min_freq` variable that always points to the current minimum frequency level.

**The frequency increment pattern:**

When a node is accessed:
1. Remove it from its current frequency list
2. If that frequency list becomes empty and it was the minimum frequency, increment `min_freq`
3. Add the node to the `frequency + 1` list
4. Always add to the head of the new list (marking it as most recently used at that frequency)

This design ensures that at any moment, we can find the least frequently used (and least recently used in case of ties) item at the tail of the `freq_map[min_freq]` list in `O(1)` time.

## Solution Approach

The implementation uses three main components: `Node`, `DoublyLinkedList`, and `LFUCache` classes.

**1. Node Class:**
```python
class Node:
    def __init__(self, key: int, value: int):
        self.key = key
        self.value = value
        self.freq = 1  # Initial frequency is 1
        self.prev = None
        self.next = None
```
Each node stores the key-value pair, its frequency count, and pointers for the doubly linked list.

**2. DoublyLinkedList Class:**
```python
class DoublyLinkedList:
    def __init__(self):
        self.head = Node(-1, -1)  # Dummy head
        self.tail = Node(-1, -1)  # Dummy tail
        self.head.next = self.tail
        self.tail.prev = self.head
```
Uses dummy head and tail nodes to simplify edge cases. This list maintains nodes in LRU order within each frequency level.

Key operations:
- `add_first(node)`: Adds a node right after the head (most recently used position)
- `remove(node)`: Removes a node from the list in `O(1)` time
- `remove_last()`: Removes the node before tail (least recently used)
- `is_empty()`: Checks if the list contains any real nodes

**3. LFUCache Class:**

The cache maintains:
- `capacity`: Maximum number of items
- `min_freq`: Current minimum frequency in the cache
- `map`: HashMap mapping `key -> Node`
- `freq_map`: HashMap mapping `frequency -> DoublyLinkedList`

**Get Operation:**
```python
def get(self, key: int) -> int:
    if key not in self.map:
        return -1
    node = self.map[key]
    self.incr_freq(node)  # Increment frequency
    return node.value
```

**Put Operation:**
```python
def put(self, key: int, value: int) -> None:
    if key in self.map:
        # Update existing key
        node = self.map[key]
        node.value = value
        self.incr_freq(node)
    else:
        # Insert new key
        if len(self.map) == self.capacity:
            # Evict LFU item
            ls = self.freq_map[self.min_freq]
            node = ls.remove_last()  # Remove LRU from min frequency
            self.map.pop(node.key)
      
        # Create and add new node
        node = Node(key, value)
        self.add_node(node)
        self.map[key] = node
        self.min_freq = 1  # New node has frequency 1
```

**Increment Frequency Helper:**
```python
def incr_freq(self, node: Node) -> None:
    freq = node.freq
    ls = self.freq_map[freq]
    ls.remove(node)  # Remove from current frequency list
  
    if ls.is_empty():
        self.freq_map.pop(freq)  # Clean up empty frequency
        if freq == self.min_freq:
            self.min_freq += 1  # Update min_freq if needed
  
    node.freq += 1
    self.add_node(node)  # Add to new frequency list
```

**Add Node Helper:**
```python
def add_node(self, node: Node) -> None:
    freq = node.freq
    ls = self.freq_map[freq]  # Get or create list for this frequency
    ls.add_first(node)  # Add as most recently used
    self.freq_map[freq] = ls
```

**Time Complexity:**
- `get`: `O(1)` - HashMap lookup + list operations
- `put`: `O(1)` - HashMap operations + list operations
- Space: `O(capacity)` for storing the cache items

The solution elegantly handles the LFU eviction policy with LRU tie-breaking by maintaining a two-level structure where frequencies map to ordered lists, ensuring all operations remain `O(1)`.

## Example Walkthrough

Let's walk through a concrete example with capacity 2 to see how the LFU cache operates:

**Initial State:**
- Cache capacity: 2
- Cache is empty
- `min_freq = 0`

**Step 1: `put(1, 10)`**
- Cache is not full, create new node with key=1, value=10, freq=1
- Add node to frequency 1's linked list
- Update `min_freq = 1`
- State:
  ```
  map: {1: Node(1,10,freq=1)}
  freq_map: {1: [Node(1,10)]}
  min_freq: 1
  ```

**Step 2: `put(2, 20)`**
- Cache is not full, create new node with key=2, value=20, freq=1
- Add node to frequency 1's linked list (at head, so it's most recent)
- State:
  ```
  map: {1: Node(1,10,freq=1), 2: Node(2,20,freq=1)}
  freq_map: {1: [Node(2,20) -> Node(1,10)]}  // 2 is more recent
  min_freq: 1
  ```

**Step 3: `get(1)`**
- Find node with key=1 in map
- Increment its frequency from 1 to 2:
  - Remove Node(1,10) from frequency 1's list
  - Frequency 1's list now only has Node(2,20)
  - Add Node(1,10) to frequency 2's list
- Return value 10
- State:
  ```
  map: {1: Node(1,10,freq=2), 2: Node(2,20,freq=1)}
  freq_map: {1: [Node(2,20)], 2: [Node(1,10)]}
  min_freq: 1  // Still 1 because freq_map[1] is not empty
  ```

**Step 4: `put(3, 30)`**
- Cache is full (size = 2), need to evict
- Look at `freq_map[min_freq]` which is `freq_map[1]` containing [Node(2,20)]
- Remove the least recently used from minimum frequency (Node(2,20))
- Remove key=2 from map
- Add new Node(3,30) with freq=1
- Update `min_freq = 1`
- State:
  ```
  map: {1: Node(1,10,freq=2), 3: Node(3,30,freq=1)}
  freq_map: {1: [Node(3,30)], 2: [Node(1,10)]}
  min_freq: 1
  ```

**Step 5: `get(3)`**
- Find node with key=3 in map
- Increment its frequency from 1 to 2:
  - Remove Node(3,30) from frequency 1's list
  - Frequency 1's list becomes empty, so remove it from freq_map
  - Since freq=1 was min_freq and now empty, increment `min_freq = 2`
  - Add Node(3,30) to frequency 2's list (at head)
- Return value 30
- State:
  ```
  map: {1: Node(1,10,freq=2), 3: Node(3,30,freq=2)}
  freq_map: {2: [Node(3,30) -> Node(1,10)]}  // 3 is more recent
  min_freq: 2
  ```

**Step 6: `put(4, 40)`**
- Cache is full, need to evict
- Look at `freq_map[min_freq]` which is `freq_map[2]` containing [Node(3,30) -> Node(1,10)]
- Remove the least recently used from minimum frequency (tail = Node(1,10))
- Remove key=1 from map
- Add new Node(4,40) with freq=1
- Update `min_freq = 1` (new node always has frequency 1)
- State:
  ```
  map: {3: Node(3,30,freq=2), 4: Node(4,40,freq=1)}
  freq_map: {1: [Node(4,40)], 2: [Node(3,30)]}
  min_freq: 1
  ```

This example demonstrates:
1. How nodes move between frequency lists when accessed
2. How LRU ordering within each frequency list handles tie-breaking
3. How `min_freq` tracks the lowest frequency for efficient eviction
4. How the data structure maintains O(1) operations throughout

## Python Solution

```python
from collections import defaultdict
from typing import Optional


class Node:
    """Node for doubly linked list to store cache entries."""
  
    def __init__(self, key: int, value: int) -> None:
        self.key = key
        self.value = value
        self.freq = 1  # Frequency counter starts at 1
        self.prev: Optional[Node] = None
        self.next: Optional[Node] = None


class DoublyLinkedList:
    """Doubly linked list to maintain nodes with same frequency in LRU order."""
  
    def __init__(self) -> None:
        # Create dummy head and tail nodes for easier manipulation
        self.head = Node(-1, -1)
        self.tail = Node(-1, -1)
        self.head.next = self.tail
        self.tail.prev = self.head

    def add_first(self, node: Node) -> None:
        """Add node right after head (most recently used position)."""
        node.prev = self.head
        node.next = self.head.next
        self.head.next.prev = node
        self.head.next = node

    def remove(self, node: Node) -> Node:
        """Remove a specific node from the list."""
        node.next.prev = node.prev
        node.prev.next = node.next
        node.next = None
        node.prev = None
        return node

    def remove_last(self) -> Node:
        """Remove and return the least recently used node (before tail)."""
        return self.remove(self.tail.prev)

    def is_empty(self) -> bool:
        """Check if the list is empty (only contains dummy nodes)."""
        return self.head.next == self.tail


class LFUCache:
    """
    Least Frequently Used (LFU) cache implementation.
    Evicts least frequently used items when capacity is reached.
    For items with same frequency, evicts least recently used.
    """
  
    def __init__(self, capacity: int) -> None:
        self.capacity = capacity
        self.min_freq = 0  # Track minimum frequency for eviction
        self.node_map: dict[int, Node] = {}  # Map from key to node
        self.freq_map: defaultdict[int, DoublyLinkedList] = defaultdict(DoublyLinkedList)  # Map from frequency to list of nodes

    def get(self, key: int) -> int:
        """
        Get value for the given key.
        Returns -1 if key doesn't exist.
        """
        # Handle edge cases
        if self.capacity == 0 or key not in self.node_map:
            return -1
      
        # Get node and update its frequency
        node = self.node_map[key]
        self._increment_frequency(node)
        return node.value

    def put(self, key: int, value: int) -> None:
        """
        Insert or update key-value pair in cache.
        Evicts LFU item if cache is at capacity.
        """
        # Handle zero capacity
        if self.capacity == 0:
            return
      
        # Update existing key
        if key in self.node_map:
            node = self.node_map[key]
            node.value = value
            self._increment_frequency(node)
            return
      
        # Evict LFU item if at capacity
        if len(self.node_map) == self.capacity:
            freq_list = self.freq_map[self.min_freq]
            evicted_node = freq_list.remove_last()
            del self.node_map[evicted_node.key]
      
        # Add new node
        node = Node(key, value)
        self._add_node(node)
        self.node_map[key] = node
        self.min_freq = 1  # New node has frequency 1

    def _increment_frequency(self, node: Node) -> None:
        """
        Increment frequency of a node and move it to appropriate frequency list.
        """
        freq = node.freq
        freq_list = self.freq_map[freq]
      
        # Remove node from current frequency list
        freq_list.remove(node)
      
        # Clean up empty frequency list and update min_freq if needed
        if freq_list.is_empty():
            del self.freq_map[freq]
            if freq == self.min_freq:
                self.min_freq += 1
      
        # Increment frequency and add to new frequency list
        node.freq += 1
        self._add_node(node)

    def _add_node(self, node: Node) -> None:
        """
        Add node to the frequency list corresponding to its frequency.
        """
        freq = node.freq
        freq_list = self.freq_map[freq]
        freq_list.add_first(node)


# Your LFUCache object will be instantiated and called as such:
# obj = LFUCache(capacity)
# param_1 = obj.get(key)
# obj.put(key, value)
```

## Time and Space Complexity

**Time Complexity:**
- `get(key)`: `O(1)`
  - Checking if key exists in map: `O(1)` (hash map lookup)
  - Calling `incr_freq()`: `O(1)` (involves removing and adding node to doubly linked list)
  - Return value: `O(1)`

- `put(key, value)`: `O(1)`
  - Checking if key exists in map: `O(1)` (hash map lookup)
  - If key exists, update value and call `incr_freq()`: `O(1)`
  - If at capacity, remove LFU node: `O(1)` (remove from tail of min_freq list)
  - Create new node and call `add_node()`: `O(1)`
  - Update map and min_freq: `O(1)`

- Helper functions:
  - `incr_freq()`: `O(1)` - removes node from current frequency list, updates frequency, adds to new frequency list
  - `add_node()`: `O(1)` - adds node to head of frequency list
  - `add_first()`: `O(1)` - pointer manipulation in doubly linked list
  - `remove()`: `O(1)` - pointer manipulation in doubly linked list
  - `remove_last()`: `O(1)` - calls remove on tail's previous node
  - `is_empty()`: `O(1)` - simple comparison

**Space Complexity:** `O(capacity)`
- `self.map`: `O(n)` where `n ≤ capacity` (stores at most capacity nodes)
- `self.freq_map`: `O(n)` in worst case (each node could have different frequency)
- Each `Node`: `O(1)` space, total `O(n)` for all nodes
- Each `DoublyLinkedList`: `O(1)` overhead (just head and tail sentinels), but contains nodes already counted
- Total: `O(capacity)` since the cache never stores more than capacity items

## Common Pitfalls

### 1. **Forgetting to Handle Zero Capacity Edge Case**

One of the most common pitfalls is not properly handling when the cache is initialized with capacity 0. This can lead to unexpected behavior or errors when trying to perform operations.

**Problem Code:**
```python
class LFUCache:
    def __init__(self, capacity: int):
        self.capacity = capacity
        # ... other initialization
  
    def put(self, key: int, value: int) -> None:
        # Missing zero capacity check
        if key in self.node_map:
            # update logic
        else:
            if len(self.node_map) == self.capacity:  # This will evict when capacity=0!
                # eviction logic
            # add new node
```

**Solution:**
Always check for zero capacity at the beginning of both `get` and `put` methods:
```python
def get(self, key: int) -> int:
    if self.capacity == 0:  # Handle zero capacity
        return -1
    # ... rest of logic

def put(self, key: int, value: int) -> None:
    if self.capacity == 0:  # Handle zero capacity
        return
    # ... rest of logic
```

### 2. **Incorrectly Updating min_freq When Removing Nodes**

A critical pitfall is not properly maintaining the `min_freq` variable when removing nodes from frequency lists, especially when the minimum frequency list becomes empty.

**Problem Code:**
```python
def _increment_frequency(self, node: Node) -> None:
    freq = node.freq
    freq_list = self.freq_map[freq]
    freq_list.remove(node)
  
    # Forgot to update min_freq when the list becomes empty!
    if freq_list.is_empty():
        del self.freq_map[freq]
        # Missing: if freq == self.min_freq: self.min_freq += 1
  
    node.freq += 1
    self._add_node(node)
```

**Solution:**
Always update `min_freq` when the minimum frequency list becomes empty:
```python
if freq_list.is_empty():
    del self.freq_map[freq]
    if freq == self.min_freq:
        self.min_freq += 1  # Critical update!
```

### 3. **Not Resetting min_freq When Adding New Nodes**

When adding a new node to the cache, forgetting to reset `min_freq` to 1 can cause incorrect eviction behavior.

**Problem Code:**
```python
def put(self, key: int, value: int) -> None:
    if key not in self.node_map:
        # eviction logic if needed
        node = Node(key, value)
        self._add_node(node)
        self.node_map[key] = node
        # Forgot to set: self.min_freq = 1
```

**Solution:**
Always set `min_freq = 1` when adding a new node:
```python
node = Node(key, value)
self._add_node(node)
self.node_map[key] = node
self.min_freq = 1  # New nodes always have frequency 1
```

### 4. **Memory Leak from Not Properly Cleaning Up Node Pointers**

When removing nodes from the doubly linked list, failing to set the `prev` and `next` pointers to `None` can cause memory leaks.

**Problem Code:**
```python
def remove(self, node: Node) -> Node:
    node.next.prev = node.prev
    node.prev.next = node.next
    # Forgot to clean up node's pointers!
    return node
```

**Solution:**
Always clean up the removed node's pointers:
```python
def remove(self, node: Node) -> Node:
    node.next.prev = node.prev
    node.prev.next = node.next
    node.next = None  # Clean up
    node.prev = None  # Clean up
    return node
```

### 5. **Using Regular Dict Instead of defaultdict**

Using a regular dictionary for `freq_map` requires extra code to check and create new lists for each frequency.

**Problem Code:**
```python
self.freq_map: dict[int, DoublyLinkedList] = {}

def _add_node(self, node: Node) -> None:
    freq = node.freq
    if freq not in self.freq_map:  # Extra check needed
        self.freq_map[freq] = DoublyLinkedList()
    freq_list = self.freq_map[freq]
    freq_list.add_first(node)
```

**Solution:**
Use `defaultdict` to automatically create new lists:
```python
from collections import defaultdict

self.freq_map: defaultdict[int, DoublyLinkedList] = defaultdict(DoublyLinkedList)

def _add_node(self, node: Node) -> None:
    freq = node.freq
    freq_list = self.freq_map[freq]  # Automatically creates new list if needed
    freq_list.add_first(node)
```

These pitfalls can cause subtle bugs that are difficult to debug, especially under edge cases. Careful attention to state management and edge case handling is crucial for a correct LFU cache implementation.

