# 2534. Time Taken to Cross the Door

> Source: https://algo.monster/liteproblems/2534

## Problem Statement

You have `n` people numbered from `0` to `n - 1` and a single door that can only be used by one person at a time. Each person takes exactly one second to pass through the door (either entering or exiting).

You're given two arrays:
- `arrival[i]`: the time (in seconds) when person `i` arrives at the door
- `state[i]`: whether person `i` wants to enter (`0`) or exit (`1`)

When multiple people want to use the door at the same time, they must follow these priority rules:

1. **If the door was not used in the previous second**: People who want to **exit** go first
2. **If someone entered in the previous second**: People who want to **enter** go first
3. **If someone exited in the previous second**: People who want to **exit** go first
4. **Among people going in the same direction**: The person with the **smallest index** goes first

The task is to determine the exact time (in seconds) when each person passes through the door. Return an array `answer` where `answer[i]` is the time when person `i` crosses the door.

**Example walkthrough:**

For `arrival = [0,1,1,2,4]` and `state = [0,1,0,0,1]`:
- At `t=0`: Only person 0 wants to enter → person 0 enters at time 0
- At `t=1`: Person 1 wants to exit, person 2 wants to enter. Since someone entered at `t=0`, person 2 (entering) has priority → person 2 enters at time 1
- At `t=2`: Person 1 still wants to exit, person 3 wants to enter. Since someone entered at `t=1`, person 3 (entering) has priority → person 3 enters at time 2
- At `t=3`: Only person 1 wants to exit → person 1 exits at time 3
- At `t=4`: Only person 4 wants to exit → person 4 exits at time 4

Result: `[0,3,1,2,4]` (person 0 at time 0, person 1 at time 3, person 2 at time 1, etc.)

## Intuition

The key insight is that this is a simulation problem where we need to process people in the order they arrive, but with specific priority rules when conflicts occur. Since people can arrive at the same time but need to be processed one by one, we need a way to [queue](/problems/queue_intro) them up based on their desired direction.

Think of it like having two lines at the door - one for people wanting to enter and one for people wanting to exit. When someone arrives, they join the appropriate line. Then at each second, we decide which line gets to send someone through based on the priority rules.

The natural data structure for this is two queues - one for each direction. Why queues? Because among people going in the same direction, we need to maintain the order by their index (first-in-first-out based on person index).

For the state tracking, we need to remember what happened in the previous second to apply the correct priority rule. We can use a simple variable `st` where:
- `st = 1` means either the door wasn't used or someone exited last (favoring exit)
- `st = 0` means someone entered last (favoring entry)

The simulation then becomes straightforward:
1. At each time `t`, add all people who have arrived by this time to their respective queues
2. Check which queues have people waiting
3. If both queues have people, use the state `st` to decide who goes
4. If only one [queue](/problems/queue_intro) has people, that person goes and we update the state accordingly
5. If no one is waiting but more people will arrive later, we reset the state to favor exits (door becomes unused)
6. Move to the next second and repeat

This approach ensures we handle all the priority rules correctly while maintaining the chronological order of arrivals and processing everyone exactly once.

**Learn more about [Queue](/problems/queue_intro) patterns.**

## Solution Approach

We implement the simulation using two queues and a time-based processing loop:

**Data Structures:**
- `q = [deque(), deque()]`: Two queues where `q[0]` stores indices of people wanting to enter, and `q[1]` stores indices of people wanting to exit
- `ans`: Result array to store the time each person passes through the door

**Key Variables:**
- `t`: Current time (starts at 0)
- `i`: Index pointer for traversing the arrival array
- `st`: Door state (`1` = unused/exit last, `0` = enter last)

**Algorithm Steps:**

1. **Initialize the simulation:**
   ```python
   t = i = 0
   st = 1  # Door starts as "unused", favoring exits
   ans = [0] * n
   ```

2. **Main simulation loop** - Continue while there are people to process:
   ```python
   while i < n or q[0] or q[1]:
   ```

3. **Add arrivals to queues** - At current time `t`, add all people who have arrived:
   ```python
   while i < n and arrival[i] <= t:
       q[state[i]].append(i)
       i += 1
   ```
   Each person joins the [queue](/problems/queue_intro) corresponding to their desired direction.

4. **Process the door usage** based on who's waiting:

   - **Both queues have people:** Use the state `st` to decide priority
     ```python
     if q[0] and q[1]:
         ans[q[st].popleft()] = t
     ```
     The person from [queue](/problems/queue_intro) `q[st]` goes through at time `t`.

   - **Only one queue has people:** That person goes, update state
     ```python
     elif q[0] or q[1]:
         st = 0 if q[0] else 1
         ans[q[st].popleft()] = t
     ```
     If only entrants are waiting, `st = 0`; if only exits, `st = 1`.

   - **No one waiting:** Reset state to favor exits
     ```python
     else:
         st = 1
     ```

5. **Advance time** and repeat:
   ```python
   t += 1
   ```

**Why this works:**
- The queues naturally maintain the index-based priority within each direction (smaller indices are added first and dequeued first)
- The state variable `st` correctly implements the three priority rules:
  - When `st = 1` (unused/exit last), exits have priority
  - When `st = 0` (enter last), entries have priority
- Processing people as they arrive ensures we handle them in chronological order
- The time advances linearly, simulating each second of the door's operation

**Time Complexity:** `O(n)` where `n` is the number of people, as each person is added to and removed from a [queue](/problems/queue_intro) exactly once.

**Space Complexity:** `O(n)` for the queues and answer array.

## Example Walkthrough

Let's trace through a small example with `arrival = [0, 0, 2]` and `state = [1, 0, 1]`:
- Person 0 arrives at time 0 and wants to exit
- Person 1 arrives at time 0 and wants to enter
- Person 2 arrives at time 2 and wants to exit

**Initial Setup:**
- `t = 0`, `i = 0`, `st = 1` (door unused, favors exit)
- `q[0] = []` (enter queue), `q[1] = []` (exit queue)
- `ans = [0, 0, 0]`

**Time t=0:**
1. Add arrivals: Both person 0 and 1 arrive at time 0
   - Person 0 wants to exit → add to `q[1]`: `q[1] = [0]`
   - Person 1 wants to enter → add to `q[0]`: `q[0] = [1]`
   - Now `i = 2`
2. Process door: Both queues have people, check state `st = 1` (favors exit)
   - Person from `q[1]` goes → Person 0 exits at time 0
   - `ans[0] = 0`, `q[1] = []`
3. State remains `st = 1` (someone exited)
4. Advance to `t = 1`

**Time t=1:**
1. Add arrivals: No one arrives at time 1 (person 2 arrives at time 2)
2. Process door: Only `q[0]` has people (person 1)
   - Person 1 enters at time 1
   - `ans[1] = 1`, `q[0] = []`
   - Update `st = 0` (someone entered)
3. Advance to `t = 2`

**Time t=2:**
1. Add arrivals: Person 2 arrives at time 2
   - Person 2 wants to exit → add to `q[1]`: `q[1] = [2]`
   - Now `i = 3` (all people processed)
2. Process door: Only `q[1]` has people
   - Person 2 exits at time 2
   - `ans[2] = 2`, `q[1] = []`
   - Update `st = 1` (someone exited)
3. Advance to `t = 3`

**Time t=3:**
- No more people to process (`i = 3` and both queues empty)
- Loop terminates

**Final Result:** `ans = [0, 1, 2]`
- Person 0 exits at time 0 (had priority due to unused door)
- Person 1 enters at time 1 (only person waiting)
- Person 2 exits at time 2 (only person waiting)

This demonstrates how the state variable correctly manages priorities and how the queues maintain the order of people wanting to go in each direction.

## Python Solution

```python
from typing import List
from collections import deque

class Solution:
    def timeTaken(self, arrival: List[int], state: List[int]) -> List[int]:
        # Two queues: queue[0] for entering, queue[1] for exiting
        queues = [deque(), deque()]

        # Total number of people
        num_people = len(arrival)

        # Current time and index of next person to arrive
        current_time = 0
        person_index = 0

        # Previous state: 1 means exit (default when door not used previously)
        previous_state = 1

        # Result array to store the time each person passes through
        result = [0] * num_people

        # Process until all people have passed through the door
        while person_index < num_people or queues[0] or queues[1]:
            # Add all people who have arrived by current time to their respective queues
            while person_index < num_people and arrival[person_index] <= current_time:
                queues[state[person_index]].append(person_index)
                person_index += 1

            # Process the queues based on priority rules
            if queues[0] and queues[1]:
                # Both queues have people: use previous state for priority
                person_to_process = queues[previous_state].popleft()
                result[person_to_process] = current_time
            elif queues[0] or queues[1]:
                # Only one queue has people: process that queue
                previous_state = 0 if queues[0] else 1
                person_to_process = queues[previous_state].popleft()
                result[person_to_process] = current_time
            else:
                # No one is waiting: reset to exit priority for next use
                previous_state = 1

            # Move to next time unit
            current_time += 1

        return result
```

## Time and Space Complexity

**Time Complexity:** `O(n)`

The algorithm processes each person exactly once. The main while loop continues until all `n` people have been processed (`i < n`) and both queues are empty. In each iteration, we either:
- Add people to queues (each person is added once)
- Remove one person from a queue and assign their time (each person is removed once)
- Increment the time counter

Since each of the `n` people is added to a queue once and removed once, and all operations inside the loop (append, popleft, array access) are `O(1)`, the total time complexity is `O(n)`.

**Space Complexity:** `O(n)`

The space usage includes:
- Two deques `q[0]` and `q[1]` which together can hold at most `n` elements (all people)
- The answer array `ans` of size `n`
- A few constant variables (`t`, `i`, `st`, `n`)

The dominant space usage comes from the queues and the answer array, both of which are proportional to `n`. Therefore, the space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Time Advancement When No One is Waiting**

**Pitfall:** A common mistake is advancing time by 1 second even when nobody is waiting at the door. This causes inefficiency when there's a gap between arrivals.

**Example scenario:**
- `arrival = [0, 10]`, `state = [0, 1]`
- Person 0 arrives at t=0 and passes through
- The naive approach would increment time second by second (1, 2, 3, ..., 10) even though no one is waiting from t=1 to t=9

**Incorrect approach:**
```python
# Always incrementing by 1
current_time += 1  # This wastes iterations when no one is waiting
```

**Solution:** When both queues are empty but there are still people to arrive, jump directly to the next arrival time:

```python
if not queues[0] and not queues[1]:
    if person_index < num_people:
        # Jump to next arrival if queues are empty
        current_time = max(current_time, arrival[person_index])
    else:
        # All people processed, can exit loop
        break
else:
    # Normal time increment when processing someone
    current_time += 1
```

### 2. **Forgetting to Update State When Nobody is Waiting**

**Pitfall:** Not resetting the door state to "unused" (favoring exits) when the door isn't used for a second can lead to incorrect priorities.

**Example scenario:**
- Person enters at t=0 (state becomes "enter")
- No one uses door at t=1 (state should reset to "exit" priority)
- At t=2, if both enter and exit people arrive, exit should have priority

**Incorrect approach:**
```python
# Only updating state when someone uses the door
if queues[0] or queues[1]:
    previous_state = 0 if queues[0] else 1
    # ... process person
# Missing the else case to reset state!
```

**Solution:** Always handle the case when no one is waiting:

```python
if queues[0] and queues[1]:
    # Both queues have people
    result[queues[previous_state].popleft()] = current_time
elif queues[0] or queues[1]:
    # Only one queue has people
    previous_state = 0 if queues[0] else 1
    result[queues[previous_state].popleft()] = current_time
else:
    # No one waiting - reset state to favor exits
    previous_state = 1
```

### 3. **Processing Arrivals After Determining Door Usage**

**Pitfall:** Adding people to queues after processing the current time slot can cause people who arrive at the current time to miss their turn.

**Incorrect order:**
```python
# Process door first (WRONG!)
if queues[0] or queues[1]:
    # ... process someone

# Then add arrivals (TOO LATE!)
while person_index < num_people and arrival[person_index] <= current_time:
    queues[state[person_index]].append(person_index)
    person_index += 1
```

**Solution:** Always add all arrivals for the current time before processing the door:

```python
# First: Add all people who have arrived by current time
while person_index < num_people and arrival[person_index] <= current_time:
    queues[state[person_index]].append(person_index)
    person_index += 1

# Then: Process the door based on who's waiting
if queues[0] and queues[1]:
    # ... handle door usage
```

This ensures that everyone who arrives at time `t` is considered for passing through the door at time `t`.

