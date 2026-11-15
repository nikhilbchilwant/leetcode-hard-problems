# 381. Insert Delete GetRandom O(1) - Duplicates allowed

> Source: https://algo.monster/liteproblems/381

## Problem Statement

This problem asks you to implement a data structure called `RandomizedCollection` that supports storing duplicate values (a multiset) with three main operations, all running in average O(1) time complexity.

The data structure needs to support:

1. **`insert(val)`**: Add a value to the collection. The value can be added even if it already exists in the collection (allowing duplicates). Returns `true` if this is the first occurrence of the value, `false` if the value already existed.

2. **`remove(val)`**: Remove one occurrence of a value from the collection. If the value has multiple copies, only one is removed. Returns `true` if the value was present and removed, `false` if the value wasn't in the collection.

3. **`getRandom()`**: Return a random element from the collection. Each element's probability of being selected should be proportional to how many times it appears in the collection. For example, if the collection contains [1, 1, 2], the value 1 should have twice the probability of being selected compared to 2.

The key challenge is achieving O(1) average time complexity for all operations while handling duplicates correctly. The solution uses:
- A list `l` to store all values (including duplicates) for O(1) random access
- A dictionary `m` mapping each value to a set of indices where that value appears in the list

For the `remove` operation, the trick is to swap the element to be removed with the last element in the list, then remove the last element (O(1) operation), while carefully updating the index tracking in the dictionary.

## Intuition

To achieve O(1) operations for insert, remove, and getRandom, we need to think about what data structures give us constant time access.

For `getRandom()`, we need to pick a random element with probability proportional to its frequency. The simplest way is to store all elements (including duplicates) in a list and use random indexing - this naturally gives us the right probability distribution since duplicates occupy more positions.

However, using just a list makes `remove()` problematic - finding and removing an arbitrary element would take O(n) time. This is where the key insight comes in: we can make removal O(1) by always removing from the end of the list. But how do we remove a specific value that might be in the middle?

The trick is to swap the element we want to remove with the last element, then pop the last element. This is O(1), but now we need to track where each value is located in our list.

This leads us to maintain a hash map that maps each value to a set of indices where it appears in the list. Why a set of indices? Because we have duplicates - a value might appear at multiple positions.

When we `insert(val)`:
- Add the value to the end of the list (O(1))
- Add this new index to the set of indices for this value in our map (O(1))

When we `remove(val)`:
- Get any index where this value appears from our map (O(1))
- Swap with the last element and pop (O(1))
- Update the indices in our map - remove the old index, add the new index for the swapped element (O(1))

This combination of a list (for random access) and a hash map of sets (for tracking positions) gives us all operations in O(1) average time while correctly handling duplicates.

## Solution Approach

The implementation uses two main data structures:
- `self.m`: A dictionary mapping each value to a set of indices where that value appears in the list
- `self.l`: A list storing all values (including duplicates)

**Initialization:**
```python
def __init__(self):
    self.m = {}  # value -> set of indices
    self.l = []  # list of all values
```

**Insert Operation:**
```python
def insert(self, val: int) -> bool:
    idx_set = self.m.get(val, set())  # Get existing indices or empty set
    idx_set.add(len(self.l))          # Add new index (end of list)
    self.m[val] = idx_set              # Update map
    self.l.append(val)                 # Add value to list
    return len(idx_set) == 1           # True if first occurrence
```
The insert adds the value to the end of the list and records its index in the map. It returns `True` only when the set size is 1 (first occurrence).

**Remove Operation:**
```python
def remove(self, val: int) -> bool:
    if val not in self.m:
        return False
  
    idx_set = self.m[val]
    idx = list(idx_set)[0]      # Get any index of val
    last_idx = len(self.l) - 1
    self.l[idx] = self.l[last_idx]  # Swap with last element
    idx_set.remove(idx)          # Remove old index from val's set
  
    # Update indices for the swapped element
    last_idx_set = self.m[self.l[last_idx]]
    if last_idx in last_idx_set:
        last_idx_set.remove(last_idx)  # Remove old last index
    if idx < last_idx:
        last_idx_set.add(idx)           # Add new index (only if not same position)
  
    if not idx_set:
        self.m.pop(val)  # Remove val from map if no more occurrences
  
    self.l.pop()  # Remove last element
    return True
```

The remove operation is the most complex:
1. Pick any index where `val` appears (we use the first one from the set)
2. Swap that element with the last element in the list
3. Update the index tracking: remove the old indices, add the new index for the swapped element
4. Handle edge case when removing the last element (no swap needed)
5. Clean up empty sets from the map

**GetRandom Operation:**
```python
def getRandom(self) -> int:
    return -1 if len(self.l) == 0 else random.choice(self.l)
```
Simply picks a random element from the list. Since duplicates appear multiple times in the list, the probability is naturally proportional to frequency.

All operations achieve O(1) average time complexity through careful use of hash maps and the swap-and-pop technique for removal.

## Example Walkthrough

Let's walk through a sequence of operations to see how the data structure maintains O(1) operations while handling duplicates.

**Initial State:**
- `l = []`
- `m = {}`

**Operation 1: `insert(10)`**
- Add 10 to end of list: `l = [10]`
- Update map: `m = {10: {0}}`
- Return `True` (first occurrence)

**Operation 2: `insert(20)`**
- Add 20 to end of list: `l = [10, 20]`
- Update map: `m = {10: {0}, 20: {1}}`
- Return `True` (first occurrence)

**Operation 3: `insert(10)`**
- Add 10 to end of list: `l = [10, 20, 10]`
- Update map: `m = {10: {0, 2}, 20: {1}}`
- Return `False` (10 already exists)

**Operation 4: `getRandom()`**
- Pick random from `[10, 20, 10]`
- 10 has 2/3 probability, 20 has 1/3 probability

**Operation 5: `remove(20)`**
- Find 20 at index 1 (from `m[20] = {1}`)
- Swap with last element: `l[1] = l[2] = 10`, so `l = [10, 10, 10]`
- Update indices:
  - Remove index 1 from 20's set: `m[20] = {}` → remove 20 from map
  - The swapped element (10) was at index 2, now at index 1
  - Remove index 2 from 10's set: `{0, 2}` → `{0}`
  - Add index 1 to 10's set: `{0}` → `{0, 1}`
- Pop last element: `l = [10, 10]`
- Final state: `m = {10: {0, 1}}`
- Return `True`

**Operation 6: `remove(10)`**
- Find 10 at index 0 (could pick any from `{0, 1}`)
- Swap with last element: `l[0] = l[1] = 10`, so `l = [10, 10]`
- Update indices:
  - Remove index 0 from 10's set: `{0, 1}` → `{1}`
  - The swapped element (also 10) was at index 1, now at index 0
  - Remove index 1 from 10's set: `{1}` → `{}`
  - Add index 0 to 10's set: `{}` → `{0}`
- Pop last element: `l = [10]`
- Final state: `m = {10: {0}}`
- Return `True`

This example shows how:
1. The list maintains all values for O(1) random access
2. The map tracks positions enabling O(1) removal
3. The swap-and-pop technique keeps removal at O(1)
4. Duplicate handling works correctly with sets of indices

## Python Solution

```python
import random
from typing import Set, List, Dict

class RandomizedCollection:
    def __init__(self):
        """
        Initialize your data structure here.
        """
        # Dictionary mapping values to sets of their indices in the list
        self.value_to_indices: Dict[int, Set[int]] = {}
        # List storing all values (allows duplicates)
        self.values_list: List[int] = []

    def insert(self, val: int) -> bool:
        """
        Inserts a value to the collection. Returns true if the collection did not already contain the specified element.
        """
        # Get the set of indices for this value, or create empty set if not exists
        indices_set = self.value_to_indices.get(val, set())
      
        # Add the new index (which will be at the end of the list)
        indices_set.add(len(self.values_list))
      
        # Update the mapping
        self.value_to_indices[val] = indices_set
      
        # Append the value to the list
        self.values_list.append(val)
      
        # Return True if this was the first occurrence of the value
        return len(indices_set) == 1

    def remove(self, val: int) -> bool:
        """
        Removes a value from the collection. Returns true if the collection contained the specified element.
        """
        # Check if value exists in the collection
        if val not in self.value_to_indices:
            return False
      
        # Get the set of indices for the value to remove
        indices_set = self.value_to_indices[val]
      
        # Get any index of the value to remove (convert set to list and take first)
        index_to_remove = list(indices_set)[0]
      
        # Get the last index in the list
        last_index = len(self.values_list) - 1
      
        # Swap the element to remove with the last element
        self.values_list[index_to_remove] = self.values_list[last_index]
      
        # Remove the index from the set of the value being removed
        indices_set.remove(index_to_remove)
      
        # Update indices for the value that was swapped from the end
        last_value_indices = self.value_to_indices[self.values_list[last_index]]
      
        # Remove the old last index from the swapped value's index set
        if last_index in last_value_indices:
            last_value_indices.remove(last_index)
      
        # If we didn't remove the last element, add the new index for the swapped value
        if index_to_remove < last_index:
            last_value_indices.add(index_to_remove)
      
        # If no more indices for the removed value, delete it from the mapping
        if not indices_set:
            self.value_to_indices.pop(val)
      
        # Remove the last element from the list
        self.values_list.pop()
      
        return True

    def getRandom(self) -> int:
        """
        Get a random element from the collection.
        """
        # Return -1 if empty, otherwise return a random element
        return -1 if len(self.values_list) == 0 else random.choice(self.values_list)


# Your RandomizedCollection object will be instantiated and called as such:
# obj = RandomizedCollection()
# param_1 = obj.insert(val)
# param_2 = obj.remove(val)
# param_3 = obj.getRandom()
```

## Time and Space Complexity

**Time Complexity:**
- `insert(val)`: `O(1)` - Getting/creating a set, adding to set, and appending to list are all constant time operations.
- `remove(val)`: `O(1)` - Dictionary lookup, set operations (remove/add), list element swap, and list pop from end are all constant time. Converting set to list with `list(idx_set)[0]` is `O(1)` since we only access the first element.
- `getRandom()`: `O(1)` - `random.choice()` on a list is constant time as it generates a random index and accesses that element directly.

**Space Complexity:**
- `O(n)` where `n` is the total number of elements in the collection.
  - The list `self.l` stores all `n` elements: `O(n)`
  - The dictionary `self.m` stores at most `n` index entries across all sets (each element in the list corresponds to one index in some set): `O(n)`
  - Total space: `O(n) + O(n) = O(n)`

## Common Pitfalls

### 1. **Incorrect Index Update When Removing the Last Element**

One of the most common mistakes occurs when removing an element that happens to be at the last position of the list. In this case, `index_to_remove` equals `last_index`, meaning we're trying to swap an element with itself.

**The Problem:**
```python
# Buggy version
def remove(self, val: int) -> bool:
    # ... earlier code ...
  
    # Swap the element
    self.values_list[index_to_remove] = self.values_list[last_index]
    indices_set.remove(index_to_remove)

    # Update indices for swapped value
    last_value_indices = self.value_to_indices[self.values_list[last_index]]
    last_value_indices.remove(last_index)  # PROBLEM: may already be gone
    last_value_indices.add(index_to_remove)  # PROBLEM: may re-add a removed index
```

After `insert(10)` and `insert(20)`, calling `remove(20)` gives `index_to_remove = last_index = 1`. The line `indices_set.remove(1)` already removed index 1 from the set for 20, and `last_value_indices` is that same set, so `last_value_indices.remove(1)` raises `KeyError`. Even with that fixed, the unconditional `add` would put index 1 back, pointing past the end of the list after the `pop()`.

**The Solution:**
Guard both updates:
```python
if last_index in last_value_indices:
    last_value_indices.remove(last_index)
if index_to_remove < last_index:
    last_value_indices.add(index_to_remove)
```

### 2. **Using List Instead of Set for Index Storage**

Another pitfall is using a list to store indices instead of a set:

**The Problem:**
```python
# Incorrect approach
self.value_to_indices = {}  # value -> list of indices (WRONG!)

def insert(self, val: int) -> bool:
    indices_list = self.value_to_indices.get(val, [])
    indices_list.append(len(self.values_list))  # O(1)
    # ...

def remove(self, val: int) -> bool:
    # ...
    indices_list.remove(index_to_remove)  # O(n) - Linear search!
```

Using a list makes the `remove` operation O(n) because removing a specific index requires searching through the list.

**The Solution:**
Use a set for O(1) average-case removal:
```python
self.value_to_indices = {}  # value -> set of indices
indices_set.remove(index_to_remove)  # O(1) average
```

### 3. **Not Handling the Case When Removing Value That Appears Multiple Times at Different Indices**

A subtle bug can occur when the value being removed appears multiple times, including at the last position:

**The Problem:**
```python
# Example: values_list = [1, 2, 1], removing 1
# value_to_indices = {1: {0, 2}, 2: {1}}

# If we remove index 0 (first 1), we swap with index 2 (also 1)
# After swap: values_list = [1, 2, 1]  # Looks the same!
# But indices need updating: {1: {0, 2}} -> {1: {0}}
```

If not careful with the index updates, you might incorrectly update the indices for the swapped value when it's the same as the removed value.

**The Solution:**
The code correctly handles this by getting the indices set for `self.values_list[last_index]` (the actual value at the last position) rather than assuming it's different from `val`:
```python
# Correct approach - uses the actual value at last_index
last_value_indices = self.value_to_indices[self.values_list[last_index]]
```

### 4. **Forgetting to Clean Up Empty Sets in the Dictionary**

**The Problem:**
After removing all occurrences of a value, leaving empty sets in the dictionary:
```python
# Incomplete removal
indices_set.remove(index_to_remove)
# Forgot to remove empty set from dictionary!
# self.value_to_indices might still have {val: set()}
```

The next `remove(val)` then passes the `val not in self.value_to_indices` check and reaches `list(indices_set)[0]` on an empty set, which raises `IndexError` instead of returning `False`. Re-insertion still works, because adding to the empty set makes its size 1, but the dictionary keeps stale keys.

**The Solution:**
Always clean up empty sets:
```python
if not indices_set:
    self.value_to_indices.pop(val)
```

