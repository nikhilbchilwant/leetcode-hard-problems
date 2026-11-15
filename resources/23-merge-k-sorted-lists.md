# 23. Merge k Sorted Lists

> Source: https://algo.monster/liteproblems/23

## Problem Statement

You have `k` sorted linked lists, where each [linked list](/problems/linked_list_cycle) contains nodes arranged in ascending order. Your task is to merge all these linked lists into a single sorted linked list.

For example, if you have 3 linked lists:
- List 1: `1 → 4 → 5`
- List 2: `1 → 3 → 4`  
- List 3: `2 → 6`

The merged result should be: `1 → 1 → 2 → 3 → 4 → 4 → 5 → 6`

The solution uses a min heap (priority queue) to efficiently track and extract the smallest node among all list heads. Here's how it works:

1. **Initialize the heap**: Add all non-null head nodes from the input lists to a min heap. The code adds a comparison method to `ListNode` so nodes can be compared by their values in the heap.

2. **Build the result**: Create a dummy node to simplify list construction. While the heap isn't empty:
   - Extract the node with minimum value from the heap
   - If this node has a next node, add it to the heap
   - Append the extracted node to the result list
   - Move the current pointer forward

3. **Return the result**: Return `dummy.next`, which points to the head of the merged list.

The time complexity is `O(n * log k)` where `n` is the total number of nodes across all lists and `k` is the number of lists, since each node is processed once and heap operations take `O(log k)` time. The space complexity is `O(k)` for the heap.

## Intuition

When merging two sorted lists, we repeatedly compare the heads and take the smaller one. But with `k` lists, comparing all `k` heads each time would be inefficient. We need a data structure that can quickly give us the minimum value among `k` elements and allow us to update it efficiently.

Think about the merge process: at any moment, we only care about the current smallest node among all the list heads. Once we take that node, we move to its next node in that particular list. This pattern suggests we need:
1. A way to quickly find the minimum among `k` elements
2. A way to remove that minimum
3. A way to add a new element (the next node)

A min heap perfectly fits these requirements. It maintains the minimum element at the root with `O(1)` access, and both insertion and deletion take `O(log k)` time.

The key insight is that we don't need to store all nodes in the heap at once - that would use too much memory and wouldn't use the fact that each list is already sorted. Instead, we only keep track of the "frontier" - the current head of each list. As we consume nodes from one list, we advance that list's representative in the heap.

The comparison function `__lt__` is added to `ListNode` because Python's heap needs to know how to order the nodes. By defining it to compare values, we ensure the heap maintains nodes in ascending order by their values.

This approach elegantly reduces the problem from "merge `k` sorted lists" to "repeatedly find and extract the minimum from `k` elements", which is exactly what a heap is designed to do efficiently.

**Learn more about [Linked List](/problems/linked_list_cycle), [Divide and Conquer](/problems/divide_and_conquer_intro), [Heap (Priority Queue)](/problems/heap_intro) and [Merge Sort](/problems/merge_k_sorted_lists) patterns.**

## Solution Approach

The solution implements a priority queue (min heap) approach to efficiently merge `k` sorted linked lists.

**Step 1: Enable Node Comparison**
```python
setattr(ListNode, "__lt__", lambda a, b: a.val < b.val)
```
We add a less-than comparison method to the `ListNode` class. This allows Python's heap to compare nodes based on their values when maintaining heap order.

**Step 2: Initialize the Min Heap**
```python
pq = [head for head in lists if head]
heapify(pq)
```
We create a list containing all non-null head nodes from the input lists, then convert it into a min heap using `heapify()`. This operation takes `O(k)` time where `k` is the number of lists.

**Step 3: Create Dummy Node for Result**
```python
dummy = cur = ListNode()
```
A dummy node simplifies list construction by eliminating edge cases when adding the first node. The `cur` pointer tracks where to append the next node.

**Step 4: Process Nodes from the Heap**
```python
while pq:
    node = heappop(pq)
    if node.next:
        heappush(pq, node.next)
    cur.next = node
    cur = cur.next
```
The main merging loop:
- Extract the minimum node from the heap using `heappop()` - `O(log k)` time
- If this node has a successor in its original list, add that successor to the heap with `heappush()` - `O(log k)` time
- Append the extracted node to the result list
- Move the `cur` pointer forward

This maintains the invariant that the heap always contains exactly one node from each non-exhausted list, ensuring we always select the global minimum among all current heads.

**Step 5: Return Result**
```python
return dummy.next
```
Return the head of the merged list, which is `dummy.next`.

**Time Complexity**: `O(n * log k)` where `n` is the total number of nodes across all lists. Each node is processed exactly once, and each heap operation costs `O(log k)`.

**Space Complexity**: `O(k)` for the heap, which stores at most `k` nodes at any time.

## Example Walkthrough

Let's walk through merging 3 sorted linked lists:
- List 1: `1 → 4 → 5`
- List 2: `1 → 3 → 4`
- List 3: `2 → 6`

**Initial Setup:**
- Add all head nodes to the min heap: `[1(List1), 1(List2), 2(List3)]`
- After heapify, the heap maintains min property with `1` at the root
- Create dummy node and cur pointer pointing to it

**Iteration 1:**
- Extract min node: `1` from List 1
- Add its next node `4` to heap: heap becomes `[1(List2), 2(List3), 4(List1)]`
- Append `1` to result: `dummy → 1`
- Move cur pointer to node `1`

**Iteration 2:**
- Extract min node: `1` from List 2
- Add its next node `3` to heap: heap becomes `[2(List3), 3(List2), 4(List1)]`
- Append `1` to result: `dummy → 1 → 1`
- Move cur pointer forward

**Iteration 3:**
- Extract min node: `2` from List 3
- Add its next node `6` to heap: heap becomes `[3(List2), 4(List1), 6(List3)]`
- Append `2` to result: `dummy → 1 → 1 → 2`
- Move cur pointer forward

**Iteration 4:**
- Extract min node: `3` from List 2
- Add its next node `4` to heap: heap becomes `[4(List1), 4(List2), 6(List3)]`
- Append `3` to result: `dummy → 1 → 1 → 2 → 3`
- Move cur pointer forward

**Iteration 5:**
- Extract min node: `4` from List 1
- Add its next node `5` to heap: heap becomes `[4(List2), 5(List1), 6(List3)]`
- Append `4` to result: `dummy → 1 → 1 → 2 → 3 → 4`
- Move cur pointer forward

**Iteration 6:**
- Extract min node: `4` from List 2
- No next node to add to heap: heap becomes `[5(List1), 6(List3)]`
- Append `4` to result: `dummy → 1 → 1 → 2 → 3 → 4 → 4`
- Move cur pointer forward

**Iteration 7:**
- Extract min node: `5` from List 1
- No next node to add to heap: heap becomes `[6(List3)]`
- Append `5` to result: `dummy → 1 → 1 → 2 → 3 → 4 → 4 → 5`
- Move cur pointer forward

**Iteration 8:**
- Extract min node: `6` from List 3
- No next node to add to heap: heap becomes empty `[]`
- Append `6` to result: `dummy → 1 → 1 → 2 → 3 → 4 → 4 → 5 → 6`
- Move cur pointer forward

**Final Result:**
Return `dummy.next` which points to the head of the merged list: `1 → 1 → 2 → 3 → 4 → 4 → 5 → 6`

The heap ensures we always extract the globally smallest node among all current list heads, maintaining the sorted order in the final result.

## Python Solution

```python
from typing import List, Optional
from heapq import heapify, heappop, heappush

# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next

class Solution:
    def mergeKLists(self, lists: List[Optional[ListNode]]) -> Optional[ListNode]:
        """
        Merge k sorted linked lists into one sorted linked list.
      
        Args:
            lists: List of head nodes of k sorted linked lists
          
        Returns:
            Head of the merged sorted linked list
        """
        # Add comparison method to ListNode for heap operations
        # This allows heap to compare ListNode objects by their values
        setattr(ListNode, "__lt__", lambda self, other: self.val < other.val)
      
        # Initialize priority queue with all non-null head nodes
        priority_queue = [head for head in lists if head]
      
        # Convert list into a min-heap based on node values
        heapify(priority_queue)
      
        # Create dummy node to simplify list construction
        dummy_head = ListNode()
        current_node = dummy_head
      
        # Process nodes from heap until empty
        while priority_queue:
            # Extract node with minimum value
            min_node = heappop(priority_queue)
          
            # If extracted node has a next node, add it to heap
            if min_node.next:
                heappush(priority_queue, min_node.next)
          
            # Append the minimum node to result list
            current_node.next = min_node
            current_node = current_node.next
      
        # Return the head of merged list (skip dummy node)
        return dummy_head.next
```

## Time and Space Complexity

**Time Complexity:** `O(n × log k)`

The algorithm uses a min-heap to efficiently merge `k` sorted linked lists. Breaking down the operations:
- Initially, we add at most `k` heads to the heap, which takes `O(k)` time for heapify operation
- For each of the `n` total nodes across all lists:
  - We perform one `heappop` operation: `O(log k)`
  - We potentially perform one `heappush` operation: `O(log k)`
  - Other operations (pointer updates) are `O(1)`
- Since we process all `n` nodes and each node requires `O(log k)` heap operations, the total time complexity is `O(n × log k)`

**Space Complexity:** `O(k)`

The space usage comes from:
- The min-heap `pq` stores at most `k` nodes at any given time (one from each list)
- The dummy node and cur pointer use `O(1)` additional space
- The `setattr` operation modifies the class but doesn't add space proportional to input size
- Therefore, the overall space complexity is `O(k)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Heap Comparison Error with Duplicate Values**
When multiple nodes have the same value, Python's heapq might try to compare the entire ListNode objects beyond just their values, leading to a TypeError since ListNode objects don't have a natural ordering.

**Problem Example:**
```python
# If two nodes have the same value, heapq might compare the objects themselves
# This happens when using tuples like (node.val, node) in the heap
priority_queue = [(node.val, node) for node in lists if node]
heapify(priority_queue)  # May raise TypeError if two nodes have same value
```

**Solution:**
The provided code correctly handles this by adding the `__lt__` method directly to the ListNode class. However, an alternative approach using a wrapper or index-based comparison is also valid:

```python
# Alternative 1: Use unique index as tiebreaker
priority_queue = []
for i, head in enumerate(lists):
    if head:
        heappush(priority_queue, (head.val, i, head))

while priority_queue:
    val, idx, node = heappop(priority_queue)
    if node.next:
        heappush(priority_queue, (node.next.val, idx, node.next))
    # ... rest of logic
```

### 2. **Pushing Empty Lists Into the Heap**
The input may contain empty lists, which appear as `None` heads. Putting `None` into the heap breaks the comparisons, because `None` has no `val` attribute and cannot be ordered against a `ListNode`.

**Problem:**
```python
priority_queue = list(lists)  # lists = [None, node] keeps the None
heapify(priority_queue)       # TypeError when None is compared with a ListNode
```

**Solution:**
Filter out empty lists before building the heap, and only push `node.next` when it exists:
```python
priority_queue = [head for head in lists if head]
heapify(priority_queue)
while priority_queue:
    node = heappop(priority_queue)
    if node.next:
        heappush(priority_queue, node.next)
    cur.next = node
    cur = cur.next
```

### 3. **Modifying Global ListNode Class**
Using `setattr` to modify the ListNode class affects all instances globally, which could cause issues in environments where multiple solutions run concurrently or in test frameworks.

**Problem:**
```python
setattr(ListNode, "__lt__", lambda self, other: self.val < other.val)
# This modification persists beyond this function call
```

**Solution:**
Use a wrapper class or custom comparison function instead:
```python
class HeapNode:
    def __init__(self, node):
        self.node = node
  
    def __lt__(self, other):
        return self.node.val < other.node.val

# Use HeapNode wrapper in heap
priority_queue = [HeapNode(head) for head in lists if head]
heapify(priority_queue)

while priority_queue:
    heap_node = heappop(priority_queue)
    node = heap_node.node
    if node.next:
        heappush(priority_queue, HeapNode(node.next))
    # ... rest of logic
```

### 4. **Edge Case: Empty Lists Array**
If all input lists are None or the lists array itself is empty, the code handles it correctly, but it's worth being explicit about this edge case.

**Verification:**
```python
# These cases are handled correctly:
lists = []  # Returns None
lists = [None, None, None]  # Returns None
```

The current implementation handles these cases well due to the list comprehension filter `[head for head in lists if head]`, but developers should be aware of these edge cases when testing.

