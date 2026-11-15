# 1298. Maximum Candies You Can Get from Boxes

> Source: https://algo.monster/liteproblems/1298

## Problem Statement

You are given `n` boxes labeled from `0` to `n - 1`. Each box has properties defined by four arrays:

- `status[i]`: Indicates whether box `i` is initially open (`1`) or closed (`0`)
- `candies[i]`: The number of candies contained in box `i`
- `keys[i]`: A list of box labels that you can unlock after opening box `i`
- `containedBoxes[i]`: A list of other boxes found inside box `i`

You start with an array `initialBoxes` containing the labels of boxes you initially possess.

The rules for collecting candies are:
1. You can take candies from any **open** box that you have
2. When you open a box, you obtain all the keys inside it (which can be used to open other closed boxes)
3. When you open a box, you also obtain all the boxes contained within it

A box can be opened if:
- It was already open (status is `1`), OR
- You have a key for it (obtained from another box)

The goal is to find the maximum number of candies you can collect by strategically opening boxes and using the keys and contained boxes you find.

For example, you might initially have a closed box that you cannot open. But if you open another box that contains a key to the first box, you can then go back and open it to collect its candies. Similarly, a box might contain other boxes inside it, which you can then open if they are unlocked or if you have their keys.

The challenge is to explore all possible boxes you can access through this chain of opening boxes, finding keys, and discovering new boxes, ultimately maximizing the total number of candies collected.

## Intuition

The key insight is to think of this problem as an exploration game where we gradually expand our access to more boxes. Initially, we have some boxes but not all of them can be opened - some are locked and need keys.

Consider what happens when we open a box:
1. We collect the candies inside it
2. We obtain keys that might unlock other boxes we already have
3. We discover new boxes that were contained inside

This creates a ripple effect - opening one box can trigger a chain reaction of newly accessible boxes. A box that was previously inaccessible might become available through two different ways:
- We find its key in another box
- We find the box itself inside another box (and it's already open or we have its key)

The challenge is that we might encounter boxes before we have the ability to open them. For example, we might have a locked box from the beginning, but only find its key after opening several other boxes. This means we need to keep track of:
- Which boxes we currently possess (but might not be able to open yet)
- Which boxes we've already opened (to avoid processing them twice)
- The current status of each box (open or locked)

BFS naturally fits this exploration pattern because:
- We process boxes in waves - first the initially accessible boxes, then the boxes that become accessible after opening those, and so on
- Each "level" of BFS represents boxes that became accessible at the same stage of exploration
- We don't need to find an optimal path; we just need to explore all reachable boxes

The algorithm mimics how you would solve this in real life: start with what you can open immediately, collect resources (keys and new boxes), then check if any previously inaccessible boxes can now be opened with your new keys. Repeat this process until no new boxes can be opened.

The beauty of this approach is that it guarantees we'll find all accessible candies because we systematically explore every possible box that can be reached through any combination of keys and contained boxes.

**Learn more about [Breadth-First Search](/problems/bfs_intro) and [Graph](/problems/graph_intro) patterns.**

## Solution Approach

We implement the BFS exploration using a queue and hash sets to efficiently track our progress through the boxes.

**Data Structures:**
- Queue `q`: Stores boxes that are currently accessible and ready to be processed
- Set `has`: Tracks all boxes we currently possess (whether open or closed)
- Set `took`: Records boxes we've already processed to avoid duplicates
- Variable `ans`: Accumulates the total candies collected

**Initial Setup:**
We start by adding all `initialBoxes` to the `has` set since these are the boxes we possess from the beginning. For each initial box, if it's already open (`status[box] == 1`), we:
- Add it to the queue `q` for processing
- Mark it as processed in `took`
- Collect its candies and add to `ans`

**BFS Processing:**
While the queue is not empty, we process each box:

1. **Process Keys:** For each key in `keys[box]`:
   - Unlock the corresponding box by setting `status[k] = 1`
   - Check if this newly unlocked box is one we already possess (`k in has`) but haven't processed yet (`k not in took`)
   - If so, add it to the queue, mark as processed, and collect its candies

2. **Process Contained Boxes:** For each box in `containedBoxes[box]`:
   - Add it to our possession (`has.add(b)`)
   - If this box is open (`status[b] == 1`) and we haven't processed it yet (`b not in took`):
     - Add it to the queue for processing
     - Mark as processed
     - Collect its candies

**Key Insights:**
- A box enters the queue only when two conditions are met: we possess it AND it's open
- Keys permanently unlock boxes by changing their status, making them available for future processing
- The `took` set prevents processing the same box multiple times, even if we obtain it through different paths
- We collect candies exactly once per box, at the moment we add it to the queue

The algorithm terminates when the queue is empty, meaning we've explored all accessible boxes. The total candies collected in `ans` represents the maximum possible candies we can obtain.

This approach ensures completeness because:
- Every box we can access will eventually be processed
- Keys found later can unlock boxes we possessed earlier
- Boxes found inside other boxes are immediately checked for accessibility

The time complexity is `O(n)` where `n` is the number of boxes, as each box is processed at most once. The space complexity is also `O(n)` for the queue and sets.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Given:**
- 5 boxes labeled 0 to 4
- `status = [1, 0, 1, 0, 0]` (boxes 0 and 2 are open, others are closed)
- `candies = [7, 5, 4, 100, 6]`
- `keys = [[1,2], [], [4], [], []]` (box 0 has keys to boxes 1 and 2, box 2 has key to box 4)
- `containedBoxes = [[1,2], [3], [], [], []]` (box 0 contains boxes 1 and 2, box 1 contains box 3)
- `initialBoxes = [0]` (we start with only box 0)

**Step-by-step BFS process:**

**Initial Setup:**
- Add box 0 to `has`: `has = {0}`
- Box 0 is open (status[0] = 1), so:
  - Add to queue: `q = [0]`
  - Mark as processed: `took = {0}`
  - Collect candies: `ans = 7`

**Iteration 1: Process box 0**
- Dequeue box 0
- Process keys [1, 2]:
  - Key 1: Set `status[1] = 1` (unlock box 1)
    - Box 1 not in `has` yet, so no action
  - Key 2: Set `status[2] = 1` (box 2 already open, no change)
    - Box 2 not in `has` yet, so no action
- Process contained boxes [1, 2]:
  - Box 1: Add to `has` → `has = {0, 1}`
    - Box 1 is now open (we just unlocked it) and not in `took`
    - Add to queue: `q = [1]`
    - Mark as processed: `took = {0, 1}`
    - Collect candies: `ans = 7 + 5 = 12`
  - Box 2: Add to `has` → `has = {0, 1, 2}`
    - Box 2 is open (status[2] = 1) and not in `took`
    - Add to queue: `q = [1, 2]`
    - Mark as processed: `took = {0, 1, 2}`
    - Collect candies: `ans = 12 + 4 = 16`

**Iteration 2: Process box 1**
- Dequeue box 1
- Process keys [] (no keys)
- Process contained boxes [3]:
  - Box 3: Add to `has` → `has = {0, 1, 2, 3}`
    - Box 3 is closed (status[3] = 0) and we have no key
    - Cannot add to queue

**Iteration 3: Process box 2**
- Dequeue box 2
- Process keys [4]:
  - Key 4: Set `status[4] = 1` (unlock box 4)
    - Box 4 not in `has` yet, so no action
- Process contained boxes [] (no contained boxes)

**Queue is empty, algorithm terminates**

**Final Result:** 
- We collected candies from boxes 0, 1, and 2
- Total candies = 7 + 5 + 4 = 16
- Box 3 remains inaccessible (we have it but no key)
- Box 4 was never obtained (we have its key but not the box itself)

This example demonstrates how:
1. Opening box 0 gave us access to boxes 1 and 2, plus the key to box 1
2. The key from box 0 allowed us to immediately open box 1 when we found it
3. Box 3 was discovered but remained locked throughout
4. Having a key (for box 4) doesn't help if we never obtain the box itself

## Python Solution

```python
from collections import deque
from typing import List

class Solution:
    def maxCandies(
        self,
        status: List[int],
        candies: List[int],
        keys: List[List[int]],
        containedBoxes: List[List[int]],
        initialBoxes: List[int],
    ) -> int:
        """
        Calculate maximum candies that can be collected from boxes.
      
        Args:
            status: List where status[i] = 1 if box i is open, 0 if locked
            candies: List where candies[i] is the number of candies in box i
            keys: List where keys[i] contains keys found in box i
            containedBoxes: List where containedBoxes[i] contains boxes found inside box i
            initialBoxes: List of boxes available at the start
          
        Returns:
            Maximum number of candies that can be collected
        """
        # Initialize queue for BFS traversal
        queue = deque()
      
        # Track boxes we have and boxes we've already opened
        boxes_owned = set(initialBoxes)
        boxes_opened = set()
      
        # Total candies collected
        total_candies = 0
      
        # Process initial boxes - open any that are already unlocked
        for box in initialBoxes:
            if status[box] == 1:  # Box is open/unlocked
                queue.append(box)
                boxes_opened.add(box)
                total_candies += candies[box]
      
        # BFS to process all reachable boxes
        while queue:
            current_box = queue.popleft()
          
            # Process keys found in current box
            for key in keys[current_box]:
                if status[key] == 0:  # Box was locked
                    status[key] = 1  # Unlock the box
                  
                    # If we own this box and haven't opened it yet, open it now
                    if key in boxes_owned and key not in boxes_opened:
                        queue.append(key)
                        boxes_opened.add(key)
                        total_candies += candies[key]
          
            # Process boxes found inside current box
            for contained_box in containedBoxes[current_box]:
                boxes_owned.add(contained_box)
              
                # If this box is unlocked and we haven't opened it yet, open it
                if status[contained_box] == 1 and contained_box not in boxes_opened:
                    queue.append(contained_box)
                    boxes_opened.add(contained_box)
                    total_candies += candies[contained_box]
      
        return total_candies
```

## Time and Space Complexity

**Time Complexity: `O(n)`**

The algorithm uses BFS to process boxes. Each box can be added to the queue at most once (tracked by the `took` set), and for each box processed:
- We iterate through its keys list: `O(keys[box])` operations
- We iterate through its contained boxes list: `O(containedBoxes[box])` operations

Since each box is processed at most once and the total number of keys and contained boxes across all boxes is bounded by `O(n)`, the overall time complexity is `O(n)`.

**Space Complexity: `O(n)`**

The algorithm uses the following data structures:
- `q` (deque): Can contain at most `n` boxes
- `has` (set): Can contain at most `n` boxes  
- `took` (set): Can contain at most `n` boxes
- `status` array: Modified in-place, but originally size `n`

The total space used is `O(n)` where `n` is the total number of boxes.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Not Re-checking Previously Owned but Locked Boxes After Finding Keys

A critical mistake is failing to check if a newly found key unlocks a box we already possessed but couldn't open earlier. When we find a key, we must check ALL boxes we currently own, not just newly discovered ones.

**Incorrect approach:**
```python
# Wrong: Only checking if the key unlocks future boxes
for key in keys[current_box]:
    status[key] = 1  # Unlock the box
    # Missing: Not checking if we already own this box!
```

**Correct approach:**
```python
for key in keys[current_box]:
    if status[key] == 0:
        status[key] = 1  # Unlock the box
        # Critical: Check if we already own this now-unlocked box
        if key in boxes_owned and key not in boxes_opened:
            queue.append(key)
            boxes_opened.add(key)
            total_candies += candies[key]
```

### Pitfall 2: Processing Boxes Multiple Times

Another common error is adding the same box to the queue multiple times, leading to counting candies more than once. This can happen when:
- A box is contained in multiple other boxes
- Multiple keys unlock the same box
- The same box appears in both initial boxes and contained boxes

**Incorrect approach:**
```python
# Wrong: Not tracking which boxes have been processed
for contained_box in containedBoxes[current_box]:
    if status[contained_box] == 1:
        queue.append(contained_box)  # May add duplicates!
        total_candies += candies[contained_box]  # May count twice!
```

**Correct approach:**
```python
# Use a set to track processed boxes
for contained_box in containedBoxes[current_box]:
    boxes_owned.add(contained_box)
    if status[contained_box] == 1 and contained_box not in boxes_opened:
        queue.append(contained_box)
        boxes_opened.add(contained_box)  # Mark as processed
        total_candies += candies[contained_box]
```

### Pitfall 3: Modifying Status Array Without Checking Current State

Blindly setting `status[key] = 1` without checking if the box was already open can lead to inefficient operations and potential logic errors in more complex scenarios.

**Better practice:**
```python
for key in keys[current_box]:
    if status[key] == 0:  # Only modify if currently locked
        status[key] = 1
        # ... rest of logic
```

This optimization avoids unnecessary operations and makes the code's intent clearer.

