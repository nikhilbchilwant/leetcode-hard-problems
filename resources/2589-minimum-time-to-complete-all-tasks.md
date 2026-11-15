# 2589. Minimum Time to Complete All Tasks

> Source: https://algo.monster/liteproblems/2589

## Problem Statement

You have a computer that can run multiple tasks simultaneously. Given a 2D array `tasks` where each `tasks[i] = [start_i, end_i, duration_i]` represents:
- `start_i`: the earliest time the task can start
- `end_i`: the latest time the task must finish
- `duration_i`: the total seconds needed to complete the task

Each task must run for exactly `duration_i` seconds within its time window `[start_i, end_i]` (inclusive). The task doesn't need to run continuously - it can be split across multiple time intervals.

The computer can be turned on and off as needed. When it's on, it can run any number of tasks simultaneously. Your goal is to find the minimum total time the computer needs to be turned on to complete all tasks.

For example, if you have two tasks:
- Task 1: `[1, 3, 2]` - needs 2 seconds between time 1 and 3
- Task 2: `[2, 5, 3]` - needs 3 seconds between time 2 and 5

One optimal solution would be to turn on the computer at times 2 and 3 (2 seconds total) for task 1, and at times 2, 3, and 5 (reusing times 2 and 3) for task 2. The total time the computer is on would be 3 seconds (times 2, 3, and 5).

## Intuition

The key observation is that when multiple tasks have overlapping time windows, they can share the same time slots when the computer is on. This means we want to maximize the reuse of time slots across different tasks.

Consider which time slots we should prioritize for each task. If we have a task that needs to run for `duration` seconds within `[start, end]`, which specific seconds should we choose?

The [greedy](/problems/greedy_intro) insight is to choose time slots as late as possible within each task's window. Why? Because later time slots are more likely to overlap with future tasks' time windows. If we sort tasks by their end times and process them in that order, choosing the latest possible slots for each task increases the chances that subsequent tasks (which have equal or later end times) can reuse those same slots.

For example, if Task A runs from `[1, 5]` and needs 2 seconds, and Task B runs from `[3, 7]` and needs 2 seconds:
- If we run Task A at times 1 and 2, Task B would need new slots at times 6 and 7 (total: 4 seconds)
- If we run Task A at times 4 and 5, Task B can reuse those same slots (total: 2 seconds)

This leads to our strategy:
1. Sort tasks by end time to ensure we process tasks with earlier deadlines first
2. For each task, select the required time slots starting from the end of its window and working backwards
3. Mark selected time slots as "used" so subsequent tasks can reuse them
4. Count only the newly selected time slots toward our total

By always selecting the latest possible slots and processing tasks in order of end time, we create maximum opportunity for slot reuse, thus minimizing the total time the computer needs to be on.

**Learn more about [Stack](/problems/stack_intro), [Greedy](/problems/greedy_intro), [Binary Search](/problems/binary_search_intro) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

The implementation follows the [greedy](/problems/greedy_intro) strategy we identified:

**Step 1: Sort tasks by end time**
```python
tasks.sort(key=lambda x: x[1])
```
We sort all tasks in ascending order of their end times. This ensures we process tasks with earlier deadlines first, allowing later tasks to potentially reuse the time slots we select.

**Step 2: Track selected time slots**
```python
vis = [0] * 2010
```
We create a boolean array `vis` where `vis[i] = 1` indicates that time slot `i` has been selected (computer is on at time `i`). The size 2010 covers all possible time values based on the problem constraints.

**Step 3: Process each task greedily**
For each task `[start, end, duration]`:

First, we check how many time slots in the task's window have already been selected:
```python
duration -= sum(vis[start : end + 1])
```
This reduces the `duration` by the number of already-selected slots that this task can reuse.

Then, we select additional time slots if needed, starting from the end and working backwards:
```python
i = end
while i >= start and duration > 0:
    if not vis[i]:
        duration -= 1
        vis[i] = 1
        ans += 1
    i -= 1
```

The algorithm iterates from `end` down to `start`:
- If time slot `i` is not yet selected (`vis[i] == 0`), we select it
- Mark `vis[i] = 1` to indicate this slot is now used
- Increment our answer counter `ans` (total time computer is on)
- Decrease `duration` as we've satisfied one more second of this task's requirement
- Continue until we've selected enough slots (`duration == 0`)

**Step 4: Return the result**
The variable `ans` tracks the total number of unique time slots selected across all tasks, which represents the minimum time the computer needs to be on.

The time complexity is `O(n log n + n * m)` where `n` is the number of tasks and `m` is the maximum time range. The space complexity is `O(m)` for the `vis` array.

## Example Walkthrough

Let's walk through a concrete example with 3 tasks:
- Task A: `[1, 4, 2]` - needs 2 seconds between time 1 and 4
- Task B: `[2, 6, 3]` - needs 3 seconds between time 2 and 6
- Task C: `[5, 7, 1]` - needs 1 second between time 5 and 7

**Step 1: Sort by end time**
After sorting: `[[1,4,2], [2,6,3], [5,7,1]]` (already sorted in this example)

**Step 2: Initialize tracking array**
`vis = [0,0,0,0,0,0,0,0,...]` (all zeros initially)

**Step 3: Process each task**

**Processing Task A `[1,4,2]`:**
- No slots used yet in range [1,4], so duration remains 2
- Select slots from end: choose times 4 and 3
- `vis = [0,0,0,1,1,0,0,0,...]`
- Computer on time += 2 (total: 2)

**Processing Task B `[2,6,3]`:**
- Check existing slots in [2,6]: times 3 and 4 are already selected
- Remaining duration = 3 - 2 = 1
- Select 1 more slot from end: choose time 6
- `vis = [0,0,0,1,1,0,1,0,...]`
- Computer on time += 1 (total: 3)

**Processing Task C `[5,7,1]`:**
- Check existing slots in [5,7]: time 6 is already selected
- Remaining duration = 1 - 1 = 0
- No new slots needed! Task C can reuse time 6
- `vis = [0,0,0,1,1,0,1,0,...]`
- Computer on time += 0 (total: 3)

**Result:** The computer needs to be on for 3 seconds total (at times 3, 4, and 6).

This demonstrates the key insight: by selecting slots as late as possible (starting from the end of each task's window), we maximized reuse. Task B reused times 3 and 4 from Task A, and Task C completely reused time 6 from Task B, minimizing the total running time.

## Python Solution

```python
class Solution:
    def findMinimumTime(self, tasks: List[List[int]]) -> int:
        # Sort tasks by their end time to process them in optimal order
        # This ensures we can greedily assign time slots from the end
        tasks.sort(key=lambda task: task[1])

        # Track which time slots have been used (0 = unused, 1 = used)
        # Size 2010 assumes end times won't exceed 2009
        time_slot_used = [0] * 2010

        # Total number of time units needed
        total_time = 0

        # Process each task
        for start_time, end_time, required_duration in tasks:
            # Calculate how many time slots within this task's window are already used
            already_used_slots = sum(time_slot_used[start_time : end_time + 1])

            # Reduce the required duration by the already used slots
            remaining_duration = required_duration - already_used_slots

            # Greedily assign time slots from the end of the task's window
            # This maximizes potential sharing with future tasks
            current_time = end_time
            while current_time >= start_time and remaining_duration > 0:
                # If this time slot hasn't been used yet
                if not time_slot_used[current_time]:
                    # Mark it as used
                    time_slot_used[current_time] = 1
                    # Decrease remaining duration needed
                    remaining_duration -= 1
                    # Increment total time used
                    total_time += 1
                # Move to earlier time slot
                current_time -= 1

        return total_time
```

## Time and Space Complexity

**Time Complexity:** `O(n × log n + n × m)`

The time complexity consists of two main parts:
- `O(n × log n)` for sorting the tasks array, where `n` is the number of tasks
- `O(n × m)` for processing all tasks, where:
  - For each of the `n` tasks, we perform:
    - `sum(vis[start : end + 1])` which takes `O(m)` time in the worst case (when `end - start` can be up to `m`)
    - A while loop that iterates from `end` to `start`, which is also `O(m)` in the worst case
  - Therefore, processing all tasks takes `O(n × m)` time

The overall time complexity is `O(n × log n + n × m)`, where `m = 2010` (the size of the `vis` array).

**Space Complexity:** `O(m)`

The space complexity is determined by:
- The `vis` array of size `2010`, which takes `O(m)` space where `m = 2010`
- The sorting operation uses `O(log n)` space for the recursive call stack in Python's Timsort
- Since `m` is a constant (2010) and typically larger than `log n`, the dominant space complexity is `O(m)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Array Slicing for Already-Used Slots Calculation**

**The Pitfall:**
The line `sum(time_slot_used[start_time : end_time + 1])` creates a new list slice every time, which can be inefficient. More critically, developers often make off-by-one errors with Python slicing, forgetting that the end index is exclusive. While the code correctly uses `end_time + 1`, it's easy to mistakenly write `sum(time_slot_used[start_time : end_time])` which would miss the last time slot.

**Solution:**
Use an explicit loop to make the logic clearer and avoid slicing errors:
```python
already_used_slots = 0
for t in range(start_time, end_time + 1):
    if time_slot_used[t]:
        already_used_slots += 1
```

### 2. **Fixed Array Size Assumption**

**The Pitfall:**
The code uses `time_slot_used = [0] * 2010` with a hardcoded size. If the problem constraints change or if end times exceed 2009, this will cause an IndexError. This is a brittle assumption that makes the code less maintainable.

**Solution:**
Calculate the maximum end time dynamically:
```python
max_time = max(task[1] for task in tasks) if tasks else 0
time_slot_used = [0] * (max_time + 1)
```

### 3. **Not Validating Task Duration Against Window Size**

**The Pitfall:**
The code assumes that `duration <= (end_time - start_time + 1)` for all tasks. If a task has a duration longer than its available window, the greedy algorithm will fail silently by not allocating enough time slots, leading to incorrect results.

**Solution:**
Add validation before processing:
```python
for start_time, end_time, required_duration in tasks:
    window_size = end_time - start_time + 1
    if required_duration > window_size:
        # Handle impossible task - could return -1 or raise exception
        return -1  # Task cannot be completed within its window
    # ... rest of the processing
```

### 4. **Inefficient Time Slot Selection for Large Time Ranges**

**The Pitfall:**
When iterating backwards from `end_time` to `start_time`, if the time range is large and most slots are already used, the algorithm wastes time checking many already-used slots.

**Solution:**
Maintain a set or list of available time slots within each window:
```python
# Collect available slots first
available_slots = []
for t in range(start_time, end_time + 1):
    if not time_slot_used[t]:
        available_slots.append(t)

# Select from the end of available slots
slots_to_use = min(remaining_duration, len(available_slots))
for i in range(len(available_slots) - slots_to_use, len(available_slots)):
    time_slot_used[available_slots[i]] = 1
    total_time += 1
```

### 5. **Not Handling Edge Cases**

**The Pitfall:**
The code doesn't explicitly handle edge cases like empty task lists, tasks with zero duration, or negative time values.

**Solution:**
Add edge case handling at the beginning:
```python
if not tasks:
    return 0

# Filter out invalid tasks
valid_tasks = [task for task in tasks
               if task[2] > 0 and task[0] <= task[1] and task[0] >= 0]
```

