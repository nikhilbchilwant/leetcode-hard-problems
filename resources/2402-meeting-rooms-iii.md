# 2402. Meeting Rooms III

> Source: https://algo.monster/liteproblems/2402

## Problem Statement

You have `n` meeting rooms numbered from `0` to `n - 1`.

You're given a list of meetings, where each meeting is represented as `[start_time, end_time]`. The time interval is half-closed, meaning `[start, end)` includes the start time but excludes the end time. All start times are unique.

The meeting room allocation follows these rules:

1. **Lowest available room first**: Each meeting gets assigned to the unused room with the smallest number.

2. **Delay if no rooms available**: If all rooms are busy, the meeting waits until a room becomes free. When it finally starts, it keeps its original duration. For example, if a meeting was supposed to run from time 2 to 5 (duration 3), but all rooms are busy until time 7, it will run from time 7 to 10 (still duration 3).

3. **Earlier meetings get priority**: When a room becomes available and multiple meetings are waiting, the meeting with the earlier original start time gets the room first.

Your task is to find which room hosted the most meetings. If multiple rooms hosted the same maximum number of meetings, return the room with the smallest number.

For example, if you have 2 rooms and meetings `[[0,10], [1,5], [2,7], [3,4]]`:
- Meeting `[0,10]` gets room 0 (runs 0-10)
- Meeting `[1,5]` gets room 1 (runs 1-5)  
- Meeting `[2,7]` waits until room 1 is free at time 5, then runs 5-10 in room 1
- Meeting `[3,4]` waits until a room is free at time 10, then runs 10-11 in room 0

Room 0 hosted 2 meetings, room 1 hosted 2 meetings, so we return room 0 (smaller number).

## Intuition

The key insight is that we need to simulate the meeting room allocation process while tracking which rooms are busy and which are idle at any given time.

Think about what information we need at each moment:
- Which rooms are currently free?
- Which rooms are occupied and when will they become free?
- When multiple rooms are free, which one should we pick? (the one with the smallest index)
- When no rooms are free, which meeting ends first?

This naturally leads us to use two data structures:

1. **A min-heap for idle rooms** - sorted by room index, so we can quickly get the smallest available room number
2. **A min-heap for busy rooms** - sorted by end time (and room index as tiebreaker), so we can quickly identify which room becomes free first

The simulation process becomes straightforward:
- Process meetings in chronological order (sort by start time first)
- Before assigning each meeting, check if any busy rooms have become free (their end time ≤ current meeting's start time) and move them to the idle heap
- If we have idle rooms, assign the smallest one
- If no idle rooms exist, we must wait - find the room that becomes free earliest, calculate the delay, and schedule the meeting with the adjusted time

The clever part is handling delays: when a meeting is delayed from time `s` to time `a` (where `a` is when a room becomes available), the meeting duration remains `e - s`, so the new end time becomes `a + (e - s)`.

By maintaining a counter array `cnt[i]` that tracks how many meetings each room hosted, we can easily find our answer at the end by scanning for the maximum count (with the smallest index as tiebreaker).

**Learn more about [Sorting](/problems/sorting_summary) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

The implementation uses two min-heaps as priority queues to manage room allocation efficiently:

**Data Structures:**
- `busy`: A min-heap storing `(end_time, room_index)` pairs for occupied rooms, sorted by end time first
- `idle`: A min-heap storing available room indices, sorted by room number
- `cnt`: An array to track the number of meetings held in each room

**Algorithm Steps:**

1. **Initialize and Sort:**
   - Sort meetings by start time using `meetings.sort()`
   - Initialize `idle` heap with all room indices `[0, 1, 2, ..., n-1]`
   - Create empty `busy` heap and counter array `cnt`

2. **Process Each Meeting:**
   For each meeting `[s, e]` in sorted order:
 
   a. **Free up rooms:** Check if any busy rooms have finished (end time ≤ current start time `s`):
   ```python
   while busy and busy[0][0] <= s:
       heappush(idle, heappop(busy)[1])
   ```
   This moves completed rooms from `busy` back to `idle`.

   b. **Assign room:**
   - **If idle rooms exist:** Take the smallest room index:
     ```python
     i = heappop(idle)
     cnt[i] += 1
     heappush(busy, (e, i))
     ```
     The room becomes busy until time `e`.
 
   - **If no idle rooms:** Find the room that becomes free earliest:
     ```python
     a, i = heappop(busy)  # a = earliest end time, i = room index
     cnt[i] += 1
     heappush(busy, (a + e - s, i))
     ```
     The meeting is delayed from time `s` to time `a`. Since the duration is `e - s`, the new end time becomes `a + (e - s)`.

3. **Find the Answer:**
   Scan through the counter array to find the room with maximum meetings (choosing the smallest index for ties):
   ```python
   ans = 0
   for i, v in enumerate(cnt):
       if cnt[ans] < v:
           ans = i
   ```

**Time Complexity:** `O(m × log m)` where `m` is the number of meetings
- [Sorting](/problems/sorting_summary) meetings: `O(m × log m)`
- Each meeting involves at most `O(log n)` heap operations
- Since `n ≤ m`, the overall complexity is `O(m × log m)`

**Space Complexity:** `O(n)` for the heaps and counter array

## Example Walkthrough

Let's walk through a concrete example with 3 rooms and meetings `[[0,4], [1,3], [2,5], [3,4], [4,6]]`.

**Initial Setup:**
- `idle` heap: `[0, 1, 2]` (all rooms available)
- `busy` heap: `[]` (no rooms occupied)
- `cnt`: `[0, 0, 0]` (meeting count for each room)

**Processing Meeting [0,4]:**
- No busy rooms to free up
- Idle rooms available, take room 0 (smallest)
- `idle`: `[1, 2]`, `busy`: `[(4, 0)]`, `cnt`: `[1, 0, 0]`
- Room 0 is busy until time 4

**Processing Meeting [1,3]:**
- Check busy rooms: room 0's end time (4) > current start (1), so no rooms freed
- Idle rooms available, take room 1
- `idle`: `[2]`, `busy`: `[(3, 1), (4, 0)]`, `cnt`: `[1, 1, 0]`
- Room 1 is busy until time 3

**Processing Meeting [2,5]:**
- Check busy rooms: earliest end time (3) > current start (2), so no rooms freed
- Idle rooms available, take room 2
- `idle`: `[]`, `busy`: `[(3, 1), (4, 0), (5, 2)]`, `cnt`: `[1, 1, 1]`
- All rooms now occupied

**Processing Meeting [3,4]:**
- Check busy rooms: room 1 ends at time 3 ≤ current start (3), so free it
- Move room 1 from busy to idle: `idle`: `[1]`, `busy`: `[(4, 0), (5, 2)]`
- Take room 1 from idle
- `idle`: `[]`, `busy`: `[(4, 0), (4, 1), (5, 2)]`, `cnt`: `[1, 2, 1]`
- Room 1 is busy until time 4

**Processing Meeting [4,6]:**
- Check busy rooms: both room 0 and 1 end at time 4 ≤ current start (4)
- Free both: `idle`: `[0, 1]`, `busy`: `[(5, 2)]`
- Take room 0 (smallest available)
- `idle`: `[1]`, `busy`: `[(5, 2), (6, 0)]`, `cnt`: `[2, 2, 1]`

**Final Result:**
- `cnt`: `[2, 2, 1]` (room 0: 2 meetings, room 1: 2 meetings, room 2: 1 meeting)
- Rooms 0 and 1 both hosted 2 meetings (maximum)
- Return room 0 (smaller index)

This example demonstrates how the algorithm efficiently manages room allocation using heaps, handles rooms becoming free, and tracks meeting counts to find the answer.

## Python Solution

```python
class Solution:
    def mostBooked(self, n: int, meetings: List[List[int]]) -> int:
        # Sort meetings by start time
        meetings.sort()
      
        # Min heap to track busy rooms: (end_time, room_number)
        busy_rooms = []
      
        # Min heap to track available rooms by room number
        available_rooms = list(range(n))
        heapify(available_rooms)
      
        # Counter for number of meetings held in each room
        meeting_count = [0] * n
      
        # Process each meeting
        for start_time, end_time in meetings:
            # Free up rooms that have finished their meetings
            while busy_rooms and busy_rooms[0][0] <= start_time:
                freed_room = heappop(busy_rooms)[1]
                heappush(available_rooms, freed_room)
          
            # If there's an available room, use it
            if available_rooms:
                room_id = heappop(available_rooms)
                meeting_count[room_id] += 1
                heappush(busy_rooms, (end_time, room_id))
            else:
                # No available rooms, wait for the earliest room to finish
                earliest_end_time, room_id = heappop(busy_rooms)
                meeting_count[room_id] += 1
                # Schedule meeting right after current meeting ends
                # Duration is (end_time - start_time)
                new_end_time = earliest_end_time + (end_time - start_time)
                heappush(busy_rooms, (new_end_time, room_id))
      
        # Find the room with the most meetings (smallest index if tie)
        most_used_room = 0
        for room_id, count in enumerate(meeting_count):
            if meeting_count[most_used_room] < count:
                most_used_room = room_id
      
        return most_used_room
```

## Time and Space Complexity

**Time Complexity:** `O(m * log m + m * log n)` where `m` is the number of meetings and `n` is the number of rooms.

- Sorting meetings takes `O(m * log m)`
- For each meeting, we perform:
  - While loop to free rooms: In the worst case, all rooms might become free, leading to `O(n * log n)` operations across all meetings (amortized `O(log n)` per meeting)
  - Push/pop operations on heaps: Each heap operation takes `O(log n)` time
  - Each meeting is processed exactly once with at most constant heap operations
- Finding the maximum count at the end takes `O(n)`
- Overall: `O(m * log m + m * log n + n)` = `O(m * log m + m * log n)` since typically `m ≥ n`

**Space Complexity:** `O(n)`

- `busy` heap can contain at most `n` rooms: `O(n)`
- `idle` heap contains at most `n` rooms: `O(n)`
- `cnt` array of size `n`: `O(n)`
- The sorting operation might use `O(log m)` space for the sorting algorithm's recursion stack (if using quicksort/mergesort)
- Overall: `O(n + log m)` = `O(n)` as `n` is typically the dominant factor

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Duration Calculation When Delaying Meetings**

**The Pitfall:** When all rooms are busy and a meeting needs to be delayed, a common mistake is to incorrectly calculate the new end time. Developers often make one of these errors:

- Setting the new end time as just `earliest_end_time + end_time` (adding the original end time instead of duration)
- Setting it as `earliest_end_time + 1` (assuming all meetings have duration 1)
- Using the original `end_time` without adjustment

**Wrong Implementation:**
```python
# INCORRECT - adds end_time instead of duration
new_end_time = earliest_end_time + end_time  

# INCORRECT - uses fixed duration
new_end_time = earliest_end_time + 1

# INCORRECT - doesn't account for delay
heappush(busy_rooms, (end_time, room_id))
```

**Correct Implementation:**
```python
# CORRECT - adds the actual duration (end_time - start_time)
new_end_time = earliest_end_time + (end_time - start_time)
heappush(busy_rooms, (new_end_time, room_id))
```

### 2. **Not Freeing All Available Rooms**

**The Pitfall:** Using `if` instead of `while` when freeing up rooms, which only frees one room even when multiple rooms have finished.

**Wrong Implementation:**
```python
# INCORRECT - only frees one room
if busy_rooms and busy_rooms[0][0] <= start_time:
    freed_room = heappop(busy_rooms)[1]
    heappush(available_rooms, freed_room)
```

**Correct Implementation:**
```python
# CORRECT - frees all rooms that have finished
while busy_rooms and busy_rooms[0][0] <= start_time:
    freed_room = heappop(busy_rooms)[1]
    heappush(available_rooms, freed_room)
```

### 3. **Forgetting to Sort Meetings**

**The Pitfall:** Processing meetings in the given order instead of chronological order by start time. The problem states that meetings should be assigned based on their start times.

**Wrong Implementation:**
```python
# INCORRECT - processes meetings in given order
for start_time, end_time in meetings:
    # process meeting
```

**Correct Implementation:**
```python
# CORRECT - sorts meetings first
meetings.sort()
for start_time, end_time in meetings:
    # process meeting
```

### 4. **Using Wrong Heap for Available Rooms**

**The Pitfall:** Using a regular list or not properly maintaining the min-heap property for available rooms, leading to incorrect room allocation (not choosing the lowest numbered available room).

**Wrong Implementation:**
```python
# INCORRECT - doesn't maintain sorted order
available_rooms = list(range(n))
# Later: available_rooms.append(freed_room) instead of heappush
```

**Correct Implementation:**
```python
# CORRECT - uses heap operations
available_rooms = list(range(n))
heapify(available_rooms)
# Later: heappush(available_rooms, freed_room)
```

