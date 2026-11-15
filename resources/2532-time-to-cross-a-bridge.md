# 2532. Time to Cross a Bridge

> Source: https://algo.monster/liteproblems/2532

## Problem Statement

This problem simulates workers moving boxes across a bridge between two warehouses. You have `n` boxes that need to be moved from the right (old) warehouse to the left (new) warehouse by `k` workers.

Each worker `i` has four time attributes given in `time[i] = [right_i, pick_i, left_i, put_i]`:
- `right_i`: Time to cross the bridge from left to right
- `pick_i`: Time to pick up a box from the right warehouse
- `left_i`: Time to cross the bridge from right to left
- `put_i`: Time to put down a box in the left warehouse

The complete cycle for a worker to move one box involves:
1. Cross bridge to the right side (`right_i` minutes)
2. Pick up a box (`pick_i` minutes)
3. Cross bridge back to the left side (`left_i` minutes)
4. Put down the box (`put_i` minutes)

**Worker Efficiency Rules:**
Worker `i` is less efficient than worker `j` if:
- `left_i + right_i > left_j + right_j`, OR
- `left_i + right_i == left_j + right_j` AND `i > j`

In other words, workers are ranked by their total bridge crossing time, with ties broken by worker index.

**Bridge Usage Rules:**
1. Only one worker can use the bridge at any time
2. When multiple workers are waiting to cross:
   - Prioritize workers on the **right side** (carrying boxes) over workers on the left side
   - Among workers on the same side, the **least efficient** worker crosses first
3. Workers stop being sent from the left side once enough workers have been dispatched to handle all remaining boxes

**Initial Setup:**
- All `k` workers start on the left side of the bridge
- All `n` boxes are in the right warehouse

The goal is to find the total time when the last box reaches the left warehouse (i.e., when the last worker carrying a box finishes crossing the bridge from right to left).

## Intuition

The key insight is that this is an event-driven simulation problem where workers transition between different states: waiting to cross, crossing the bridge, and working in warehouses.

Since only one worker can use the bridge at a time, the bridge becomes a bottleneck. We need to track which workers are available to cross at any given moment and choose the right one based on the priority rules.

The problem has four distinct states for workers:
1. **Waiting on the left** to go pick up boxes
2. **Working on the right** (picking up a box)
3. **Waiting on the right** to return with a box
4. **Working on the left** (putting down a box)

Workers move through these states in a cycle: wait left → cross → work right → wait right → cross → work left → wait left.

The priority rules tell us we need to:
- Always let workers with boxes (on the right) cross before empty-handed workers (on the left)
- Among workers on the same side, let the least efficient one cross first

Since "least efficient" means highest crossing time (or highest index as tiebreaker), we can use max-heaps to automatically get the least efficient worker when needed. We use negative indices in Python's min-heap to simulate a max-heap.

For workers who are currently working (not waiting), we need to know when they'll finish. This is a time-based event, so we use min-heaps to track the earliest completion time.

The simulation advances by:
1. Moving any workers who finished their warehouse work to the waiting queues
2. Checking if anyone is waiting to cross
3. If someone is waiting, let the appropriate worker cross based on priorities
4. If no one is waiting, jump forward in time to when the next worker finishes their warehouse work

The clever optimization is sorting workers by efficiency first. This way, worker indices naturally represent efficiency rankings, making heap operations straightforward.

The simulation ends when all boxes have been moved and the last worker carrying a box crosses to the left side. We track this by checking if `n == 0` (all boxes assigned) and no workers remain on the right side.

**Learn more about [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

The implementation uses priority queues and simulation to track worker movements across the bridge.

**Initial Setup:**
First, we sort the workers by their efficiency metric `left[i] + right[i]` in ascending order. This clever preprocessing means that worker index `i` directly represents their efficiency ranking - higher index means less efficient.

```python
time.sort(key=lambda x: x[0] + x[2])
```

**Data Structures:**
We maintain four priority queues to track worker states:
- `wait_in_left`: Max-heap storing indices of workers waiting on the left bank (use negative indices to simulate max-heap)
- `wait_in_right`: Max-heap storing indices of workers waiting on the right bank
- `work_in_left`: Min-heap storing `(finish_time, worker_index)` for workers putting boxes in left warehouse
- `work_in_right`: Min-heap storing `(finish_time, worker_index)` for workers picking boxes in right warehouse

Initially, all workers are waiting on the left:
```python
for i in range(k):
    heappush(wait_in_left, -i)
```

**Simulation Loop:**
The main simulation uses `cur` to track current time and processes events:

1. **Update waiting queues**: Check if any workers have finished their warehouse work at current time
   ```python
   while work_in_left and work_in_left[0][0] <= cur:
       t, i = heappop(work_in_left)
       heappush(wait_in_left, -i)
   ```
   Similar logic applies for `work_in_right`.

2. **Determine crossing eligibility**:
   - `left_to_go = n > 0 and wait_in_left`: Workers can go from left only if boxes remain
   - `right_to_go = bool(wait_in_right)`: Workers on right always need to return

3. **Handle no waiting workers**: If nobody is waiting to cross, fast-forward time to next worker completion
   ```python
   if not left_to_go and not right_to_go:
       nxt = min(work_in_left[0][0], work_in_right[0][0])
       cur = nxt
   ```

4. **Process bridge crossing with priorities**:
   - **Right side has priority** (carrying boxes):
     ```python
     i = -heappop(wait_in_right)  # Get least efficient worker
     cur += time[i][2]  # Add crossing time
     heappush(work_in_left, (cur + time[i][3], i))  # Schedule put completion
     ```
     Check termination: if `n == 0` and no workers remain on right, return `cur`
 
   - **Left side crossing** (going to pick boxes):
     ```python
     i = -heappop(wait_in_left)
     cur += time[i][0]  # Add crossing time
     n -= 1  # One box assigned
     heappush(work_in_right, (cur + time[i][1], i))  # Schedule pick completion
     ```

The simulation continues until all boxes are moved. The answer is the time when the last worker carrying a box finishes crossing to the left, which is captured by the termination condition when `n == 0` and both `wait_in_right` and `work_in_right` are empty.

## Example Walkthrough

Let's walk through a small example with `n = 3` boxes and `k = 2` workers:
- Worker 0: `[1, 1, 2, 1]` (right=1, pick=1, left=2, put=1)
- Worker 1: `[1, 2, 3, 1]` (right=1, pick=2, left=3, put=1)

**Step 1: Sort workers by efficiency**
- Worker 0: total crossing time = 1 + 2 = 3
- Worker 1: total crossing time = 1 + 3 = 4
- After sorting: Worker 0 stays at index 0, Worker 1 at index 1 (Worker 1 is less efficient)

**Step 2: Initialize data structures**
- `cur = 0` (current time)
- `wait_in_left = [-1, -0]` (both workers waiting on left, -1 has priority as least efficient)
- `wait_in_right = []` (empty)
- `work_in_left = []` (empty)
- `work_in_right = []` (empty)

**Step 3: Simulation begins**

**Time 0:**
- Check workers: No one finished working yet
- Can go left→right? Yes (n=3 > 0 and workers waiting)
- Can go right→left? No (no one waiting on right)
- Worker 1 (least efficient) crosses left→right
- `cur = 0 + 1 = 1` (crossing time)
- Worker 1 starts picking at time 1, finishes at time 3
- `work_in_right = [(3, 1)]`
- `n = 2` (one box assigned)

**Time 1:**
- Check workers: No one finished working yet
- Can go left→right? Yes (n=2 > 0 and worker 0 waiting)
- Can go right→left? No (no one waiting on right)
- Worker 0 crosses left→right
- `cur = 1 + 1 = 2` (crossing time)
- Worker 0 starts picking at time 2, finishes at time 3
- `work_in_right = [(3, 1), (3, 0)]`
- `n = 1` (another box assigned)

**Time 2:**
- No one waiting to cross
- Fast-forward to next event at time 3

**Time 3:**
- Workers 0 and 1 finish picking
- `wait_in_right = [-1, -0]` (both ready to return, -1 has priority)
- Can go right→left? Yes
- Worker 1 (least efficient) crosses right→left with box
- `cur = 3 + 3 = 6` (crossing time)
- Worker 1 starts putting at time 6, finishes at time 7
- `work_in_left = [(7, 1)]`

**Time 6:**
- Worker 0 still waiting on right
- Can go right→left? Yes
- Worker 0 crosses right→left with box
- `cur = 6 + 2 = 8` (crossing time)
- Worker 0 starts putting at time 8, finishes at time 9
- `work_in_left = [(7, 1), (9, 0)]`

**Time 8:**
- Worker 1 finishes putting at time 7 (already passed)
- `wait_in_left = [-1]` (worker 1 back on left)
- Can go left→right? Yes (n=1 > 0)
- Worker 1 crosses left→right
- `cur = 8 + 1 = 9`
- Worker 1 starts picking at time 9, finishes at time 11
- `work_in_right = [(11, 1)]`
- `n = 0` (last box assigned)

**Time 9:**
- Worker 0 finishes putting
- `wait_in_left = [-0]`
- Can go left→right? No (n=0, no more boxes)
- Fast-forward to time 11

**Time 11:**
- Worker 1 finishes picking last box
- `wait_in_right = [-1]`
- Worker 1 crosses right→left with final box
- `cur = 11 + 3 = 14`
- Check termination: n=0 and no workers remain on right
- **Return 14** (time when last box reaches left warehouse)

The simulation successfully tracks all worker movements and correctly identifies that the last box reaches the left warehouse at time 14, when Worker 1 completes their final crossing.

## Python Solution

```python
class Solution:
    def findCrossingTime(self, n: int, k: int, time: List[List[int]]) -> int:
        """
        Find the time when the last box reaches the right side.
      
        Args:
            n: Number of boxes to transport
            k: Number of workers
            time: List of [leftToRight, pickOld, rightToLeft, putNew] times for each worker
      
        Returns:
            Time when the last box arrives at the right side
        """
        from heapq import heappush, heappop
        from math import inf
      
        # Sort workers by their efficiency (less efficient workers have higher priority)
        # This ensures less efficient workers get priority when multiple are waiting
        time.sort(key=lambda x: x[0] + x[2])
      
        current_time = 0
      
        # Priority queues for workers waiting on each side (negative index for max-heap)
        waiting_left = []  # Workers waiting on left side
        waiting_right = []  # Workers waiting on right side
      
        # Priority queues for workers currently working on each side
        # Format: (finish_time, worker_index)
        working_left = []  # Workers picking/putting boxes on left side
        working_right = []  # Workers picking/putting boxes on right side
      
        # Initially, all workers are waiting on the left side
        for worker_id in range(k):
            heappush(waiting_left, -worker_id)
      
        while True:
            # Move workers who finished their work to waiting queues
            # Check left side workers
            while working_left:
                finish_time, worker_id = working_left[0]
                if finish_time > current_time:
                    break
                heappop(working_left)
                heappush(waiting_left, -worker_id)
          
            # Check right side workers
            while working_right:
                finish_time, worker_id = working_right[0]
                if finish_time > current_time:
                    break
                heappop(working_right)
                heappush(waiting_right, -worker_id)
          
            # Determine if there are workers ready to cross
            can_cross_from_left = n > 0 and waiting_left  # Still boxes to move and workers available
            can_cross_from_right = bool(waiting_right)  # Workers waiting on right side
          
            # If no one can cross right now, advance time to next event
            if not can_cross_from_left and not can_cross_from_right:
                next_event_time = inf
              
                if working_left:
                    next_event_time = min(next_event_time, working_left[0][0])
                if working_right:
                    next_event_time = min(next_event_time, working_right[0][0])
              
                current_time = next_event_time
                continue
          
            # Priority: workers on right side cross first (to free up space)
            if can_cross_from_right:
                # Get the least efficient worker (highest priority in our max-heap)
                worker_id = -heappop(waiting_right)
              
                # Worker crosses from right to left
                current_time += time[worker_id][2]  # rightToLeft time
              
                # Check if this was the last box being delivered
                if n == 0 and not waiting_right and not working_right:
                    return current_time
              
                # Worker starts putting the box on the left side
                finish_time = current_time + time[worker_id][3]  # putNew time
                heappush(working_left, (finish_time, worker_id))
            else:
                # Worker crosses from left to right with a box
                worker_id = -heappop(waiting_left)
              
                # Worker crosses from left to right
                current_time += time[worker_id][0]  # leftToRight time
                n -= 1  # One box is being transported
              
                # Worker starts picking up the box on the right side
                finish_time = current_time + time[worker_id][1]  # pickOld time
                heappush(working_right, (finish_time, worker_id))
```

## Time and Space Complexity

**Time Complexity:** `O(n × log k)`

The algorithm processes each of the `n` boxes exactly once. For each box:
- A worker is selected from the waiting queue (either `wait_in_left` or `wait_in_right`) using heap operations: `O(log k)`
- The worker is then added to a work queue (`work_in_left` or `work_in_right`) using heap operations: `O(log k)`
- Workers are moved from work queues back to waiting queues using heap operations: `O(log k)`

Since each box requires a constant number of heap operations and there are `n` boxes to process, the total time complexity is `O(n × log k)`.

**Space Complexity:** `O(k)`

The algorithm maintains four heaps:
- `wait_in_left`: stores indices of workers waiting on the left bank, at most `k` workers
- `wait_in_right`: stores indices of workers waiting on the right bank, at most `k` workers  
- `work_in_left`: stores workers currently working on the left bank, at most `k` workers
- `work_in_right`: stores workers currently working on the right bank, at most `k` workers

At any point in time, each of the `k` workers is in exactly one of these four heaps. Therefore, the total space used by all heaps combined is `O(k)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Priority Queue Implementation for Worker Efficiency**

**Pitfall**: Using the original worker indices directly in the priority queues without considering efficiency ordering. This leads to incorrect worker selection when multiple workers are waiting to cross.

**Why it happens**: The problem states that less efficient workers have priority, but if you use original indices, you might select the wrong worker when multiple are waiting.

**Incorrect approach**:
```python
# Wrong: Using original indices directly
for i in range(k):
    heappush(wait_in_left, -i)  # This assumes worker 0 is least efficient
```

**Solution**: Sort workers by efficiency first, then use the sorted indices:
```python
# Correct: Sort by efficiency first
time.sort(key=lambda x: x[0] + x[2])
# Now index i represents efficiency rank (higher i = less efficient)
for i in range(k):
    heappush(wait_in_left, -i)
```

### 2. **Mishandling the Termination Condition**

**Pitfall**: Terminating the simulation too early or too late, particularly forgetting to check if workers are still working on the right side.

**Why it happens**: The simulation should end when the last box reaches the left warehouse, but it's easy to miss edge cases where workers are still picking boxes on the right side.

**Incorrect approach**:
```python
# Wrong: Only checking if boxes remain
if n == 0:
    return cur
```

**Solution**: Check all conditions - no boxes left AND no workers on the right side:
```python
# Correct: Comprehensive termination check
if n == 0 and not wait_in_right and not work_in_right:
    return cur
```

### 3. **Time Advancement Logic Errors**

**Pitfall**: Not properly advancing time when no workers are waiting to cross the bridge, causing infinite loops or incorrect timing.

**Why it happens**: When all workers are busy in warehouses, the simulation must jump to the next completion time, but it's easy to miss this case.

**Incorrect approach**:
```python
# Wrong: Stuck in infinite loop when no one is waiting
while True:
    # Process workers...
    # Missing time advancement when everyone is busy
```

**Solution**: Explicitly handle the case when no one can cross:
```python
# Correct: Advance time to next worker completion
if not left_to_go and not right_to_go:
    nxt = inf
    if work_in_left:
        nxt = min(nxt, work_in_left[0][0])
    if work_in_right:
        nxt = min(nxt, work_in_right[0][0])
    cur = nxt
    continue
```

### 4. **Incorrect Box Count Management**

**Pitfall**: Decrementing the box count at the wrong time - either when a worker starts crossing or when they finish putting the box down.

**Why it happens**: The box count should decrease when a worker is assigned to transport a box (starts crossing from left), not when they complete delivery.

**Incorrect approach**:
```python
# Wrong: Decrementing when box is delivered
if wait_in_right:
    i = -heappop(wait_in_right)
    cur += time[i][2]
    n -= 1  # Wrong timing!
```

**Solution**: Decrement when a worker is assigned to pick up a box:
```python
# Correct: Decrement when worker goes to pick up a box
if wait_in_left and n > 0:
    i = -heappop(wait_in_left)
    cur += time[i][0]
    n -= 1  # Correct: Box is now assigned to this worker
```

### 5. **Bridge Priority Rules Violation**

**Pitfall**: Not properly implementing the priority that workers on the right side (carrying boxes) always cross before workers on the left side.

**Why it happens**: The condition logic can be confusing, especially when determining who can cross.

**Incorrect approach**:
```python
# Wrong: Checking left side first
if left_to_go:
    # Process left crossing
elif right_to_go:
    # Process right crossing
```

**Solution**: Always prioritize right side crossings:
```python
# Correct: Right side has priority
if right_to_go:
    # Process right crossing first
elif left_to_go:
    # Process left crossing only if no one on right
```

