# 25. Reverse Nodes in k-Group

> Source: https://algo.monster/liteproblems/25

## Problem Statement

You are given the head of a [linked list](/problems/linked_list_cycle) and a positive integer `k`. Your task is to reverse the nodes of the linked list in groups of `k` nodes at a time, then return the modified list.

The reversal should work as follows:
- Take `k` consecutive nodes from the [linked list](/problems/linked_list_cycle) and reverse their order
- Move to the next group of `k` nodes and reverse them
- Continue this process for all complete groups of `k` nodes

Important constraints and rules:
- `k` is guaranteed to be less than or equal to the length of the [linked list](/problems/linked_list_cycle)
- If the remaining nodes at the end of the list are fewer than `k`, leave them in their original order (don't reverse incomplete groups)
- You must modify the actual node connections, not just swap the values stored in the nodes
- Only the links between nodes can be changed; the values within nodes must remain unchanged

For example, if you have a [linked list](/problems/linked_list_cycle) `1 -> 2 -> 3 -> 4 -> 5` and `k = 2`:
- First group: reverse `1 -> 2` to get `2 -> 1`
- Second group: reverse `3 -> 4` to get `4 -> 3`
- Remaining nodes: `5` is less than `k` nodes, so leave it as is
- Final result: `2 -> 1 -> 4 -> 3 -> 5`

## Intuition

When we need to reverse nodes in groups, the key insight is that we're essentially performing multiple smaller reversals and then connecting them together. Think of it like reversing words in a sentence - we reverse each word individually, then put them back in their original positions.

The first challenge is determining whether we have enough nodes for a complete group. We need to look ahead `k` nodes before committing to a reversal. If we can't find `k` nodes, we know we've reached the end and should stop.

For the reversal itself, we can treat each group of `k` nodes as an independent [linked list](/problems/linked_list_cycle) segment. By temporarily "cutting" this segment from the main list (setting the last node's `next` to `None`), we can reverse it using a standard linked list reversal technique. The beauty of this approach is that we can reuse a simple reversal function for each group.

The trickiest part is maintaining the connections between reversed groups. After reversing a group, we need to:
1. Connect the previous group's tail to the new head of the reversed group
2. Connect the tail of the reversed group to the rest of the list
3. Move our pointer to the end of the reversed group to prepare for the next iteration

Using a dummy node simplifies edge cases, particularly when reversing the first group. The dummy node acts as a stable reference point - its `next` pointer will always point to the actual head of our result list, even after the first group is reversed.

The pattern emerges: for each iteration, we identify `k` nodes, detach them, reverse them, and reattach them. We keep track of where to connect each reversed segment using a `pre` pointer that marks the node just before the current group. After reversing, what was the first node of the group becomes the last, so that becomes our new `pre` for the next iteration.

**Learn more about [Recursion](/problems/recursion_intro) and [Linked List](/problems/linked_list_cycle) patterns.**

## Solution Approach

The implementation uses a simulation approach with a helper function for reversing linked lists. Let's walk through the solution step by step.

First, we define a helper function `reverse` that reverses an entire [linked list](/problems/linked_list_cycle):
- Create a dummy node to build the reversed list
- Iterate through the original list, taking each node and inserting it at the beginning of the new list
- Each node's `next` pointer is updated to point to the previous head of the reversed list
- Return the reversed list starting from `dummy.next`

For the main function `reverseKGroup`, we use the following approach:

1. **Initialize pointers**: Create a dummy node pointing to the original head. This dummy node helps handle edge cases and provides a stable reference to return the final result. We also initialize a `pre` pointer to track the node just before each group to be reversed.

2. **Check for k nodes**: Before reversing any group, we need to verify that we have at least `k` nodes remaining:
   - Start from `pre` and move `cur` pointer `k` times forward
   - If `cur` becomes `None` before completing `k` moves, we've reached the end with fewer than `k` nodes remaining
   - Return `dummy.next` as we don't reverse incomplete groups

3. **Extract and reverse the group**:
   - Save the first node of the group as `node` (which is `pre.next`)
   - Save the node after the group as `nxt` (which is `cur.next`)
   - Temporarily disconnect the group by setting `cur.next = None`
   - Call `reverse(node)` to reverse these `k` nodes

4. **Reconnect the reversed group**:
   - Connect `pre.next` to the new head of the reversed group (returned by `reverse`)
   - Connect the tail of the reversed group (which is the original `node`) to `nxt`
   - Update `pre` to `node` for the next iteration

5. **Continue iteration**: Repeat steps 2-4 until we can't find a complete group of `k` nodes.

The time complexity is `O(n)` where `n` is the number of nodes, as we visit each node a constant number of times. The space complexity is `O(1)` as we only use a fixed number of pointers regardless of input size.

The elegance of this solution lies in treating each group as an independent unit that can be reversed and reconnected, while the dummy node and pointer management handle all the edge cases seamlessly.

## Example Walkthrough

Let's walk through the solution with a concrete example: linked list `1 -> 2 -> 3 -> 4 -> 5` with `k = 3`.

**Initial Setup:**
```
dummy -> 1 -> 2 -> 3 -> 4 -> 5
  ^
 pre
```
We create a dummy node pointing to head (1) and set `pre = dummy`.

**First Iteration - Check for k nodes:**
Starting from `pre`, we move a `cur` pointer k times:
```
dummy -> 1 -> 2 -> 3 -> 4 -> 5
  ^                ^
 pre              cur
```
We successfully moved 3 times (1→2→3), so we have a complete group.

**Extract and Reverse First Group:**
- Save `node = pre.next` (node 1)
- Save `nxt = cur.next` (node 4)
- Set `cur.next = None` to isolate the group: `1 -> 2 -> 3 -> None`
- Reverse this group to get: `3 -> 2 -> 1 -> None`

**Reconnect:**
```
Before: dummy -> [1 -> 2 -> 3] -> 4 -> 5
After:  dummy -> [3 -> 2 -> 1] -> 4 -> 5
```
- Set `pre.next = 3` (head of reversed group)
- Set `node.next = 4` (connect tail of reversed group to rest)
- Update `pre = node` (node 1)

**Second Iteration - Check for k nodes:**
```
dummy -> 3 -> 2 -> 1 -> 4 -> 5
                   ^
                  pre
```
Starting from `pre` (node 1), try to move `cur` k times:
- Move 1: `cur` at node 4
- Move 2: `cur` at node 5
- Move 3: `cur` becomes `None` (out of bounds)

Since we can't find k nodes, we stop and return `dummy.next`.

**Final Result:** `3 -> 2 -> 1 -> 4 -> 5`

The first group (1, 2, 3) was reversed to (3, 2, 1), while the incomplete group (4, 5) remained unchanged.

## Python Solution

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next

from typing import Optional

class Solution:
    def reverseKGroup(self, head: Optional[ListNode], k: int) -> Optional[ListNode]:
        def reverse_linked_list(start_node: Optional[ListNode]) -> Optional[ListNode]:
            """
            Reverses a linked list and returns the new head.
          
            Args:
                start_node: Head of the linked list to reverse
              
            Returns:
                New head of the reversed linked list
            """
            dummy_head = ListNode()
            current = start_node
          
            # Reverse the linked list by inserting each node at the beginning
            while current:
                next_temp = current.next
                current.next = dummy_head.next
                dummy_head.next = current
                current = next_temp
              
            return dummy_head.next
      
        # Create dummy node to simplify edge cases
        dummy = ListNode(next=head)
        prev_group_tail = dummy
      
        while prev_group_tail:
            # Check if there are k nodes remaining
            current = prev_group_tail
            for _ in range(k):
                current = current.next
                if current is None:
                    # Less than k nodes remaining, return result
                    return dummy.next
          
            # Save pointers for group reversal
            group_head = prev_group_tail.next
            next_group_head = current.next
          
            # Disconnect current group from the rest of the list
            current.next = None
          
            # Reverse the current group and connect it back
            prev_group_tail.next = reverse_linked_list(group_head)
          
            # After reversal, original head becomes the tail
            group_head.next = next_group_head
          
            # Move to the next group
            prev_group_tail = group_head
          
        return dummy.next
```

## Time and Space Complexity

The time complexity is `O(n)`, where `n` is the total number of nodes in the linked list. This is because:
- The main while loop iterates through the list in groups of `k` nodes
- For each group, we traverse `k` nodes to check if a complete group exists
- If a complete group exists, we reverse those `k` nodes using the `reverse` function, which takes `O(k)` time
- Each node is visited at most twice: once during the checking phase and once during the reversal phase
- Total operations: `O(n/k) * O(k) + O(n/k) * O(k) = O(n) + O(n) = O(n)`

The space complexity is `O(1)` because:
- The algorithm only uses a constant amount of extra space for pointers (`dummy`, `pre`, `cur`, `node`, `nxt`)
- The `reverse` helper function also uses only a constant amount of extra space (`dummy`, `cur`, `nxt` local variables)
- No recursive calls are made that would use the call stack
- The reversal is done in-place by manipulating the existing node pointers

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Incorrect Pointer Management After Reversal

One of the most common mistakes is forgetting that after reversing a group, the original head of that group becomes its tail. Many developers incorrectly update the `prev` pointer or fail to properly reconnect the reversed group.

**Incorrect approach:**
```python
# Wrong: Trying to use the new head as the connection point for the next group
prev_group_tail.next = reverse_linked_list(group_head)
prev_group_tail = prev_group_tail.next  # This points to the new head, not the tail!
```

**Correct approach:**
```python
# Right: Remember that group_head becomes the tail after reversal
prev_group_tail.next = reverse_linked_list(group_head)
group_head.next = next_group_head  # group_head is now the tail
prev_group_tail = group_head  # Move to the actual tail of the reversed group
```

### Pitfall 2: Off-by-One Error When Counting k Nodes

Another frequent error occurs when checking if k nodes are available. Developers often start counting from the wrong node or include the previous group's tail in the count.

**Incorrect approach:**
```python
# Wrong: Starting count from prev_group_tail itself
current = prev_group_tail
for i in range(k):
    if current is None:
        return dummy.next
    current = current.next
```

**Correct approach:**
```python
# Right: Start from prev_group_tail and move k times
current = prev_group_tail
for _ in range(k):
    current = current.next
    if current is None:
        return dummy.next
```

### Pitfall 3: Modifying the List While Checking for k Nodes

Some implementations try to reverse nodes while checking if k nodes exist, leading to corrupted list structures when there are fewer than k remaining nodes.

**Incorrect approach:**
```python
# Wrong: Reversing while counting
count = 0
temp_head = None
current = prev_group_tail.next
while current and count < k:
    next_node = current.next
    current.next = temp_head
    temp_head = current
    current = next_node
    count += 1

if count < k:  # Too late! The list is already partially reversed
    # Need to reverse back - extra work and error-prone
```

**Correct approach:**
```python
# Right: Check first, then reverse
# First verify k nodes exist
current = prev_group_tail
for _ in range(k):
    current = current.next
    if current is None:
        return dummy.next

# Only then proceed with reversal
group_head = prev_group_tail.next
next_group_head = current.next
current.next = None
prev_group_tail.next = reverse_linked_list(group_head)
```

### Pitfall 4: Not Properly Isolating the Group Before Reversal

Forgetting to disconnect the group from the rest of the list before reversing can cause the reverse function to process more nodes than intended.

**Incorrect approach:**
```python
# Wrong: Forgetting to set current.next = None
group_head = prev_group_tail.next
next_group_head = current.next
# current.next is still pointing to next_group_head!
prev_group_tail.next = reverse_linked_list(group_head)  # This reverses beyond k nodes
```

**Correct approach:**
```python
# Right: Properly isolate the group
group_head = prev_group_tail.next
next_group_head = current.next
current.next = None  # Critical: disconnect the group
prev_group_tail.next = reverse_linked_list(group_head)
group_head.next = next_group_head  # Reconnect after reversal
```

