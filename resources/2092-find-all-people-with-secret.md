# 2092. Find All People With Secret

> Source: https://algo.monster/liteproblems/2092

## Problem Statement

You have `n` people numbered from `0` to `n - 1`. Person `0` initially has a secret that they share with person `firstPerson` at time `0`.

You're given a list of meetings where each meeting is represented as `[xi, yi, timei]`, meaning person `xi` and person `yi` meet at time `timei`. 

The key rules for how the secret spreads are:

1. **During a meeting**: If one person knows the secret when they meet, they immediately share it with the other person
2. **Simultaneous sharing**: A person can attend multiple meetings at the same time
3. **Instant propagation**: If someone learns the secret at a particular time, they can share it with others in their meetings at that same time

For example, if persons A, B, and C all have meetings at time 5:
- If A knows the secret and meets B at time 5, B learns it
- If B also meets C at time 5, B can immediately share it with C
- This all happens instantaneously at time 5

The goal is to determine which people know the secret after all meetings have concluded. Return a list of all person IDs who know the secret, in any order.

## Intuition

The key insight is that the secret spreads like a contagion through meetings, but with a crucial twist: it can only spread among people who meet at the same time. This time constraint is what shapes our approach.

Initially, we might think to process meetings one by one, but this would miss an important aspect: if persons A, B, and C all meet at time 5 (A meets B, B meets C), and A knows the secret, then B learns it from A and can immediately share it with C - all at time 5. This "instant propagation" means we need to consider all meetings at the same timestamp together.

Think of each timestamp as creating a temporary network. At time 5, all people meeting at that time form a [graph](/problems/graph_intro) where edges represent meetings. The secret flows through this entire connected component instantly. If anyone in this component knows the secret at the start of time 5, everyone connected to them will know it by the end of time 5.

This leads us to a natural approach:
1. Group all meetings by their timestamp
2. For each timestamp, build a [graph](/problems/graph_intro) of who meets whom
3. Find everyone who already knows the secret in this group
4. Use graph traversal (DFS/BFS) to spread the secret to all people connected to secret holders

Why DFS/BFS? Because we need to explore all paths from secret holders to mark everyone reachable. It's essentially a reachability problem within each time-based subgraph.

The temporal ordering is critical - we must process earlier times first because someone who learns the secret at time 3 can share it in their meetings at time 5, but not vice versa. This is why we sort meetings by time before processing.

By breaking the problem into time-sliced subproblems and using [graph](/problems/graph_intro) traversal within each slice, we efficiently track how the secret propagates through the entire network of meetings.

**Learn more about [Depth-First Search](/problems/dfs_intro), [Breadth-First Search](/problems/bfs_intro), [Union Find](/problems/dsu_intro), [Graph](/problems/graph_intro) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

The implementation follows the intuition of processing meetings in temporal groups and using BFS to propagate the secret within each time slice.

**Initial Setup:**
- Create a boolean array `vis` of size `n` to track who knows the secret
- Mark `vis[0] = True` (person 0 has the secret initially) and `vis[firstPerson] = True` (they learn it at time 0)
- Sort all meetings by timestamp to process them chronologically

**Time-Grouped Processing:**
The core algorithm uses a two-pointer technique to group meetings by time:
```python
i = 0
while i < m:
    j = i
    # Find all meetings at the same timestamp
    while j + 1 < m and meetings[j + 1][2] == meetings[i][2]:
        j += 1
```
This groups meetings `[i, j]` that occur at the same time.

**Building the [Graph](/problems/graph_intro) for Each Timestamp:**
For each time group, we:
1. Create an adjacency list `g` using a defaultdict
2. Track all people involved in meetings at this time in set `s`
3. Build bidirectional edges for each meeting:
```python
for x, y, _ in meetings[i : j + 1]:
    g[x].append(y)
    g[y].append(x)
    s.update([x, y])
```

**BFS Propagation:**
Starting from people who already know the secret, we spread it through the connected component:
```python
q = deque([u for u in s if vis[u]])  # Start with secret holders
while q:
    u = q.popleft()
    for v in g[u]:  # Check all neighbors
        if not vis[v]:
            vis[v] = True  # Mark as knowing the secret
            q.append(v)   # Add to queue to spread further
```

The BFS ensures that if anyone in a connected component knows the secret, everyone in that component will learn it.

**Why This Works:**
- By processing times in order, we ensure people who learn the secret earlier can share it in later meetings
- The [graph](/problems/graph_intro) representation at each timestamp captures all possible paths for secret propagation
- BFS guarantees we find all reachable people from secret holders
- The `vis` array persists across time groups, maintaining the cumulative knowledge state

**Time Complexity:** `O(M log M + M + N)` where M is the number of meetings and N is the number of people. The [sorting](/problems/sorting_summary) takes `O(M log M)`, and processing all meetings takes `O(M)`.

**Space Complexity:** `O(N + M)` for the visited array and the temporary graphs created at each timestamp.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Input:**
- `n = 6` (people numbered 0-5)
- `firstPerson = 1` (person 1 learns the secret at time 0)
- `meetings = [[1,2,5], [2,3,8], [1,5,10], [3,4,5]]`

**Step 1: Initial Setup**
- Create `vis = [False, False, False, False, False, False]`
- Mark person 0 and person 1 as knowing the secret: `vis = [True, True, False, False, False, False]`
- Sort meetings by time: `[[3,4,5], [1,2,5], [2,3,8], [1,5,10]]`

**Step 2: Process Time 5 (meetings at index 0-1)**
- Meetings: `[3,4,5]` and `[1,2,5]`
- Build graph:
  ```
  g = {3: [4], 4: [3], 1: [2], 2: [1]}
  s = {1, 2, 3, 4}
  ```
- Find who knows the secret in this group: person 1
- BFS from person 1:
  - Visit person 1 → spread to person 2
  - Mark `vis[2] = True`
  - Persons 3 and 4 form a separate component (no one knows secret there)
- After time 5: `vis = [True, True, True, False, False, False]`

**Step 3: Process Time 8 (meeting at index 2)**
- Meeting: `[2,3,8]`
- Build graph:
  ```
  g = {2: [3], 3: [2]}
  s = {2, 3}
  ```
- Find who knows the secret in this group: person 2
- BFS from person 2:
  - Visit person 2 → spread to person 3
  - Mark `vis[3] = True`
- After time 8: `vis = [True, True, True, True, False, False]`

**Step 4: Process Time 10 (meeting at index 3)**
- Meeting: `[1,5,10]`
- Build graph:
  ```
  g = {1: [5], 5: [1]}
  s = {1, 5}
  ```
- Find who knows the secret in this group: person 1
- BFS from person 1:
  - Visit person 1 → spread to person 5
  - Mark `vis[5] = True`
- After time 10: `vis = [True, True, True, True, False, True]`

**Final Result:** People who know the secret are `[0, 1, 2, 3, 5]`

Note how person 4 never learns the secret because they only met person 3 at time 5, before person 3 knew the secret. This demonstrates the importance of processing meetings chronologically.

## Python Solution

```python
from collections import defaultdict, deque
from typing import List

class Solution:
    def findAllPeople(
        self, n: int, meetings: List[List[int]], firstPerson: int
    ) -> List[int]:
        # Track who knows the secret (initially person 0 and firstPerson)
        knows_secret = [False] * n
        knows_secret[0] = True
        knows_secret[firstPerson] = True
      
        # Sort meetings by time to process them chronologically
        meetings.sort(key=lambda meeting: meeting[2])
      
        meeting_idx = 0
        total_meetings = len(meetings)
      
        # Process all meetings grouped by time
        while meeting_idx < total_meetings:
            # Find all meetings happening at the same time
            end_idx = meeting_idx
            current_time = meetings[meeting_idx][2]
          
            while end_idx + 1 < total_meetings and meetings[end_idx + 1][2] == current_time:
                end_idx += 1
          
            # Build graph for people meeting at this time
            people_at_current_time = set()
            adjacency_list = defaultdict(list)
          
            for person_x, person_y, _ in meetings[meeting_idx : end_idx + 1]:
                # Create bidirectional edges for people meeting
                adjacency_list[person_x].append(person_y)
                adjacency_list[person_y].append(person_x)
                people_at_current_time.add(person_x)
                people_at_current_time.add(person_y)
          
            # BFS to spread the secret among connected people at this time
            queue = deque()
          
            # Start BFS from people who already know the secret
            for person in people_at_current_time:
                if knows_secret[person]:
                    queue.append(person)
          
            # Spread the secret through the connected component
            while queue:
                current_person = queue.popleft()
              
                for neighbor in adjacency_list[current_person]:
                    if not knows_secret[neighbor]:
                        knows_secret[neighbor] = True
                        queue.append(neighbor)
          
            # Move to next time group
            meeting_idx = end_idx + 1
      
        # Return list of all people who know the secret
        return [person_id for person_id in range(n) if knows_secret[person_id]]
```

## Time and Space Complexity

**Time Complexity:** `O(m log m + m + E)` where `m` is the number of meetings and `E` is the total number of edges across all time groups.

- Sorting the meetings takes `O(m log m)`
- The outer while loop processes each meeting exactly once: `O(m)`
- For each time group, we:
  - Build the adjacency list: `O(k)` where `k` is the number of meetings in that time group
  - Perform BFS traversal: `O(V + E)` where `V` is the number of unique people in that time group and `E` is the number of edges (connections) in that time group
- Since each meeting is processed once and contributes to exactly one time group's graph, the total BFS work across all time groups is `O(m + E)` where `E` can be at most `2m` (since each meeting creates at most 2 directed edges)
- Overall: `O(m log m + m + E)` = `O(m log m + m)` = `O(m log m)`

**Space Complexity:** `O(n + m)`

- `vis` array: `O(n)` 
- For each time group, we create:
  - Set `s`: `O(V)` where `V` is the number of unique people in that time group
  - Graph `g`: `O(V + E)` where `E` is the number of edges in that time group
  - Queue `q`: `O(V)` in the worst case
- Since we process one time group at a time and the temporary structures are recreated for each group, the maximum space used at any point is `O(n)` for the `vis` array plus `O(V + E)` for the largest time group
- In the worst case, all meetings could happen at the same time, making `V = O(n)` and `E = O(m)`
- Overall: `O(n + m)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Not Handling Simultaneous Secret Propagation Within Same Time**

**The Pitfall:** A common mistake is to process meetings at the same time individually rather than as a group. If you process meetings one by one, you might miss the chain reaction of secret spreading that should happen instantaneously.

**Example Scenario:**
```python
meetings = [[1, 2, 5], [2, 3, 5], [3, 4, 5]]
# If person 1 knows the secret:
# - Person 1 meets 2 at time 5 (2 learns secret)
# - Person 2 meets 3 at time 5 (3 should learn secret immediately)
# - Person 3 meets 4 at time 5 (4 should learn secret immediately)
```

**Wrong Approach:**
```python
# Processing meetings individually
for x, y, time in meetings:
    if knows_secret[x]:
        knows_secret[y] = True
    elif knows_secret[y]:
        knows_secret[x] = True
# This fails because person 2 learns at time 5 but can't immediately share with person 3
```

**Correct Solution:** Group all meetings by time and use BFS to propagate the secret through the entire connected component at once, as shown in the provided solution.

### 2. **Forgetting to Sort Meetings by Time**

**The Pitfall:** Processing meetings in arbitrary order instead of chronological order means people who learn the secret later might not be able to share it in their subsequent meetings.

**Example:**
```python
meetings = [[1, 2, 10], [0, 1, 5]]  # Unsorted
# If processed as-is:
# - Meeting at time 10: persons 1 and 2 meet (neither knows secret yet)
# - Meeting at time 5: person 0 shares with person 1
# Person 2 never learns the secret!
```

**Solution:** Always sort meetings by timestamp before processing:
```python
meetings.sort(key=lambda meeting: meeting[2])
```

### 3. **Creating a Global Graph Instead of Per-Timestamp Graphs**

**The Pitfall:** Building one graph for all meetings ignores the temporal aspect - people can only share secrets during their actual meeting times.

**Wrong Approach:**
```python
# Building a single graph for all meetings
graph = defaultdict(list)
for x, y, _ in meetings:
    graph[x].append(y)
    graph[y].append(x)
# Then doing one BFS - this incorrectly allows secret sharing across different times
```

**Correct Solution:** Create a fresh graph for each timestamp group and only propagate the secret among people meeting at that specific time.

### 4. **Not Maintaining Persistent Knowledge State**

**The Pitfall:** Resetting or not properly maintaining the `knows_secret` array between different timestamps means people "forget" the secret they learned earlier.

**Example of Wrong Implementation:**
```python
while meeting_idx < total_meetings:
    # Wrong: Creating new array for each time group
    temp_knows = knows_secret.copy()  
    # ... BFS propagation ...
    knows_secret = temp_knows  # This might lose information
```

**Correct Approach:** Use a single `knows_secret` array throughout the entire algorithm that accumulates knowledge over time. Once someone learns the secret, they keep it forever.

### 5. **Incorrect BFS Initialization**

**The Pitfall:** Starting BFS from all people in meetings rather than only those who already know the secret.

**Wrong:**
```python
queue = deque(people_at_current_time)  # Start with everyone
```

**Correct:**
```python
queue = deque([person for person in people_at_current_time if knows_secret[person]])
```

Only people who already know the secret can spread it to others in their meetings.

