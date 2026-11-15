# 1606. Find Servers That Handled Most Number of Requests

> Source: https://algo.monster/liteproblems/1606

## Problem Statement

You have `k` servers (numbered 0 to k-1) that handle requests. Each server can only process one request at a time, though they have unlimited computational power otherwise.

When the `i`-th request arrives, it gets assigned to a server following these rules:

1. If all servers are busy, the request is dropped (not handled)
2. If the server at position `(i % k)` is available, assign the request to that server
3. If that preferred server is busy, check the next server `(i+1) % k`, then `(i+2) % k`, and so on, wrapping around if needed, until finding an available server

You're given two arrays:
- `arrival[i]`: the arrival time of the `i`-th request (strictly increasing)
- `load[i]`: how long the `i`-th request takes to complete

A server becomes free again at time `arrival[i] + load[i]` after starting to handle request `i`.

Your task is to find which server(s) handled the most requests successfully. Return a list of server IDs that handled the maximum number of requests. The order of IDs in the output doesn't matter.

For example, if servers 0 and 2 each handled 5 requests while all other servers handled fewer, you would return `[0, 2]` or `[2, 0]`.

## Intuition

The key insight is that we need to efficiently track which servers are free and which are busy at any given time. When a new request arrives, we need to:

1. First, free up any servers that have finished their work (their end time is <= current arrival time)
2. Then find the next available server starting from position `i % k`

The challenge is efficiently finding "the next available server starting from position `i % k`" when servers can be scattered as available or busy.

We can think of this as maintaining two groups:
- **Free servers**: We need to quickly find the first server >= `i % k`, or wrap around to find the smallest server if no such server exists
- **Busy servers**: We need to know when each becomes free, so we can move them back to the free group

For free servers, we need a data structure that:
- Maintains sorted order (to find the next server >= some value)
- Supports binary search (to quickly locate position `i % k`)
- Allows insertion and deletion

This points us toward using a sorted list or balanced tree structure for free servers.

For busy servers, we need to know which ones finish first. Since we only care about freeing servers that have finished by the current arrival time, we can use a min-heap ordered by end time. This way, we can efficiently pop all servers that should be freed.

The wrap-around logic becomes elegant with binary search: if we can't find any server >= `i % k` (meaning our search position is at the end of the sorted list), we simply take the first server in the list (index 0), which effectively wraps around.

By maintaining these two data structures and transferring servers between them as they become busy or free, we can efficiently simulate the entire process while counting how many requests each server handles.

**Learn more about [Greedy](/problems/greedy_intro) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

The solution uses two main data structures to track server states:

1. **`free`**: A `SortedList` containing IDs of available servers, maintained in sorted order
2. **`busy`**: A min-heap containing tuples of `(end_time, server_id)` for busy servers

We also maintain a `cnt` array to track how many requests each server has handled.

**Step-by-step implementation:**

1. **Initialize data structures:**
   - `free = SortedList(range(k))` - Initially all servers 0 to k-1 are free
   - `busy = []` - Empty heap for busy servers
   - `cnt = [0] * k` - Counter array initialized to zeros

2. **Process each request** at time `start` with duration `t`:
 
   a. **Free up finished servers:**
   ```python
   while busy and busy[0][0] <= start:
       free.add(busy[0][1])
       heappop(busy)
   ```
   Pop all servers from the heap whose end time <= current start time and add them back to the free list.

   b. **Check if any servers are available:**
   ```python
   if not free:
       continue
   ```
   If no servers are free, drop this request and continue to the next one.

   c. **Find the appropriate server using the assignment algorithm:**
   ```python
   j = free.bisect_left(i % k)
   if j == len(free):
       j = 0
   server = free[j]
   ```
   - Use binary search to find the first server >= `i % k`
   - If no such server exists (j equals the length of free list), wrap around by setting `j = 0`
   - This elegantly handles the circular search pattern

   d. **Assign the request to the selected server:**
   ```python
   cnt[server] += 1
   heappush(busy, (start + t, server))
   free.remove(server)
   ```
   - Increment the request count for this server
   - Add the server to busy heap with its end time (`start + t`)
   - Remove the server from the free list

3. **Find the busiest servers:**
   ```python
   mx = max(cnt)
   return [i for i, v in enumerate(cnt) if v == mx]
   ```
   Find the maximum count and return all server IDs that have this maximum count.

**Time Complexity:** O(n log k) where n is the number of requests, as each request involves heap and sorted list operations that take O(log k) time.

**Space Complexity:** O(k) for storing the server states and counts.

## Example Walkthrough

Let's walk through a small example with `k = 3` servers and the following requests:
- `arrival = [1, 2, 3, 4, 8]`
- `load = [5, 2, 3, 3, 3]`

**Initial State:**
- `free = [0, 1, 2]` (all servers available)
- `busy = []` (no busy servers)
- `cnt = [0, 0, 0]` (no requests handled yet)

**Request 0** (i=0, arrival=1, load=5):
- Free up servers: None busy yet
- Check availability: free list has servers
- Find server: `0 % 3 = 0`, bisect_left(0) finds server 0 at position 0
- Assign to server 0: 
  - `cnt = [1, 0, 0]`
  - `busy = [(6, 0)]` (server 0 busy until time 6)
  - `free = [1, 2]`

**Request 1** (i=1, arrival=2, load=2):
- Free up servers: Server 0's end time is 6 > 2, so none freed
- Check availability: free list has servers
- Find server: `1 % 3 = 1`, bisect_left(1) finds server 1 at position 0
- Assign to server 1:
  - `cnt = [1, 1, 0]`
  - `busy = [(4, 1), (6, 0)]` (heap ordered by end time)
  - `free = [2]`

**Request 2** (i=2, arrival=3, load=3):
- Free up servers: Earliest end time is 4 > 3, so none freed
- Check availability: free list has servers
- Find server: `2 % 3 = 2`, bisect_left(2) finds server 2 at position 0
- Assign to server 2:
  - `cnt = [1, 1, 1]`
  - `busy = [(4, 1), (6, 0), (6, 2)]`
  - `free = []`

**Request 3** (i=3, arrival=4, load=3):
- Free up servers: Server 1 ends at time 4, so it becomes free
  - Pop (4, 1) from busy heap
  - Add server 1 to free: `free = [1]`
  - `busy = [(6, 0), (6, 2)]`
- Check availability: free list has servers
- Find server: `3 % 3 = 0`, bisect_left(0) in `[1]` returns position 0
  - Since we need server >= 0, and only server 1 is available
  - bisect_left returns position 1 (end of list)
  - Wrap around: `j = 0`, so we select server 1
- Assign to server 1:
  - `cnt = [1, 2, 1]`
  - `busy = [(6, 0), (6, 2), (7, 1)]`
  - `free = []`

**Request 4** (i=4, arrival=8, load=3):
- Free up servers: All servers have end time <= 8
  - Pop (6, 0), add server 0 to free
  - Pop (6, 2), add server 2 to free  
  - Pop (7, 1), add server 1 to free
  - `free = [0, 1, 2]` (sorted)
  - `busy = []`
- Check availability: free list has servers
- Find server: `4 % 3 = 1`, bisect_left(1) finds server 1 at position 1
- Assign to server 1:
  - `cnt = [1, 3, 1]`
  - `busy = [(11, 1)]`
  - `free = [0, 2]`

**Final Result:**
- `cnt = [1, 3, 1]`
- Maximum count is 3
- Return `[1]` (server 1 handled the most requests)

This example demonstrates:
- How servers are freed when their end time is reached
- The circular assignment pattern with wrap-around (Request 3)
- How the bisect_left operation efficiently finds the next available server
- The coordination between the free list and busy heap to track server states

## Python Solution

```python
class Solution:
    def busiestServers(self, k: int, arrival: List[int], load: List[int]) -> List[int]:
        # Initialize a sorted list of available servers (0 to k-1)
        available_servers = SortedList(range(k))
      
        # Min heap to track busy servers: (end_time, server_id)
        busy_servers = []
      
        # Counter array to track number of requests handled by each server
        request_count = [0] * k
      
        # Process each request
        for request_id, (arrival_time, processing_time) in enumerate(zip(arrival, load)):
            # Free up servers that have completed their tasks by current arrival time
            while busy_servers and busy_servers[0][0] <= arrival_time:
                end_time, server_id = heappop(busy_servers)
                available_servers.add(server_id)
          
            # Skip this request if no servers are available
            if not available_servers:
                continue
          
            # Find the next available server using round-robin strategy
            # Start looking from server (request_id % k)
            target_server = request_id % k
            server_index = available_servers.bisect_left(target_server)
          
            # If no server >= target_server exists, wrap around to the first server
            if server_index == len(available_servers):
                server_index = 0
          
            # Assign the request to the selected server
            selected_server = available_servers[server_index]
            request_count[selected_server] += 1
          
            # Mark server as busy until arrival_time + processing_time
            heappush(busy_servers, (arrival_time + processing_time, selected_server))
            available_servers.remove(selected_server)
      
        # Find servers with maximum request count
        max_requests = max(request_count)
        return [server_id for server_id, count in enumerate(request_count) if count == max_requests]
```

## Time and Space Complexity

**Time Complexity:** `O(n * (log k + k))` where `n` is the number of requests and `k` is the number of servers.

- The main loop iterates through all `n` requests
- For each request:
  - The while loop for freeing servers: In the worst case, we might pop all `k` servers from the heap, each pop operation takes `O(log k)`, and each add to SortedList takes `O(log k)`. This gives `O(k * log k)` in the worst case
  - `bisect_left` operation on SortedList: `O(log k)`
  - `heappush` operation: `O(log k)`
  - `remove` operation on SortedList: `O(k)` in the worst case (as it needs to find and remove the element)
- The final max operation: `O(k)`
- The list comprehension: `O(k)`

The dominant operation in each iteration is the `remove` operation which is `O(k)`, making the overall time complexity `O(n * k)`. However, considering all operations more precisely, it's `O(n * (log k + k))` which simplifies to `O(n * k)`.

**Space Complexity:** `O(k)`

- `free` SortedList: stores at most `k` servers, so `O(k)`
- `busy` heap: stores at most `k` servers (each server can only be busy once at a time), so `O(k)`
- `cnt` array: stores exactly `k` counters, so `O(k)`
- The final result list: at most `k` servers, so `O(k)`

The overall space complexity is `O(k)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Server Selection After Wrap-Around**

**The Pitfall:** A common mistake is incorrectly implementing the circular search for the next available server. Many developers might try something like:

```python
# INCORRECT approach
target = request_id % k
for offset in range(k):
    server = (target + offset) % k
    if server in available_servers:
        selected_server = server
        break
```

This linear search approach is inefficient (O(k) per request) and doesn't use the sorted nature of available servers.

**Why It's Wrong:** 
- Time complexity degrades to O(n*k) for n requests
- Doesn't utilize the efficiency of binary search
- Can timeout on large inputs

**The Correct Solution:**
```python
target_server = request_id % k
server_index = available_servers.bisect_left(target_server)

# Critical: Handle wrap-around by resetting to 0
if server_index == len(available_servers):
    server_index = 0

selected_server = available_servers[server_index]
```

This uses binary search (O(log k)) and elegantly handles wrap-around by checking if the index equals the list length.

### 2. **Not Freeing All Completed Servers**

**The Pitfall:** Using `if` instead of `while` when freeing servers:

```python
# INCORRECT - only frees one server
if busy_servers and busy_servers[0][0] <= arrival_time:
    end_time, server_id = heappop(busy_servers)
    available_servers.add(server_id)
```

**Why It's Wrong:** Multiple servers might finish at or before the current arrival time. Using `if` only frees one server, leaving others incorrectly marked as busy.

**The Correct Solution:**
```python
# Correct - frees ALL completed servers
while busy_servers and busy_servers[0][0] <= arrival_time:
    end_time, server_id = heappop(busy_servers)
    available_servers.add(server_id)
```

### 3. **Confusion Between Request Index and Server Index**

**The Pitfall:** Mixing up `i` (request index) with server IDs:

```python
# INCORRECT - using wrong variable
target_server = i % k  # Should be request_id or index of current request
```

**Why It's Wrong:** The formula `(i % k)` determines the preferred server based on the request's position in the sequence, not based on any server property. Using the wrong variable breaks the round-robin assignment logic.

**The Correct Solution:** Ensure you're using the request index (position in the arrival array) for the modulo operation:
```python
for request_id, (arrival_time, processing_time) in enumerate(zip(arrival, load)):
    # ... 
    target_server = request_id % k  # Correct: using request index
```

### 4. **Forgetting to Remove Server from Available List**

**The Pitfall:** After assigning a request to a server, forgetting to remove it from the available list:

```python
# INCORRECT - server remains in available list
selected_server = available_servers[server_index]
request_count[selected_server] += 1
heappush(busy_servers, (arrival_time + processing_time, selected_server))
# Missing: available_servers.remove(selected_server)
```

**Why It's Wrong:** The server would appear available for subsequent requests even though it's busy, leading to incorrect request assignments and counts.

**The Correct Solution:** Always remove the server from available list after assignment:
```python
selected_server = available_servers[server_index]
request_count[selected_server] += 1
heappush(busy_servers, (arrival_time + processing_time, selected_server))
available_servers.remove(selected_server)  # Critical step
```

