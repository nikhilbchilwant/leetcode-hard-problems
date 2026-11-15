# 1494. Parallel Courses II

> Source: https://algo.monster/liteproblems/1494

## Problem Statement

You have `n` courses numbered from `1` to `n` that need to be taken. Some courses have prerequisites - you must complete certain courses before you can take others. These prerequisite relationships are given in the `relations` array, where `relations[i] = [prevCourse_i, nextCourse_i]` means you must complete `prevCourse_i` before taking `nextCourse_i`.

Each semester, you can take at most `k` courses, but you can only take a course if you've already completed all its prerequisites in previous semesters.

Your task is to find the minimum number of semesters needed to complete all `n` courses.

For example, if you have 4 courses with prerequisites `[[1,3], [2,3]]` and can take `k=2` courses per semester:
- Semester 1: Take courses 1 and 2 (no prerequisites)
- Semester 2: Take course 3 (prerequisites 1 and 2 are completed)
- Semester 3: Take course 4 (no prerequisites)
- Total: 3 semesters minimum

The problem guarantees that it's always possible to complete all courses (no circular dependencies).

## Intuition

The key insight is that this is a shortest path problem where we need to find the minimum number of steps (semesters) to reach a goal state (all courses completed) from an initial state (no courses completed).

Think of each possible combination of completed courses as a state. We can represent this efficiently using bits - if course `i` is completed, bit `i` is set to 1. This gives us a compact way to track which courses are done.

For each state, we need to determine which courses we can take next. A course is available if all its prerequisites are satisfied (completed in the current state). We can precompute the prerequisites for each course as a [bitmask](/problems/bitmask_intro) - for course `i`, we store `d[i]` as a bitmask of all courses that must be completed before taking course `i`.

To check if course `i` can be taken in the current state `cur`, we verify if `(cur & d[i]) == d[i]` - this means all prerequisite bits for course `i` are set in the current state.

Once we know which courses are available, we face a choice problem: if more than `k` courses are available, which `k` should we take? Since we want the minimum number of semesters, we should explore all possible combinations of taking up to `k` courses. This is where the BFS comes in - it ensures we find the shortest path by exploring all states level by level (semester by semester).

The clever part of the solution is generating all subsets of size `k` when more than `k` courses are available. It uses bit manipulation: `nxt = (nxt - 1) & x` generates all subsets of the original available courses `x` by systematically turning off bits.

By using BFS with visited state tracking, we guarantee finding the minimum number of semesters while avoiding revisiting the same state multiple times.

**Learn more about [Graph](/problems/graph_intro), [Dynamic Programming](/problems/dynamic_programming_intro) and [Bitmask](/problems/bitmask_intro) patterns.**

## Solution Approach

The solution uses BFS with bit manipulation to explore all possible ways of taking courses while respecting prerequisites and the semester limit `k`.

**Step 1: Preprocess Prerequisites**
```python
d = [0] * (n + 1)
for x, y in relations:
    d[y] |= 1 << x
```
For each course `y`, we create a [bitmask](/problems/bitmask_intro) `d[y]` that represents all its prerequisites. If course `x` is a prerequisite of course `y`, we set bit `x` in `d[y]` using `|= 1 << x`.

**Step 2: Initialize BFS**
```python
q = deque([(0, 0)])
vis = {0}
```
Start with state `0` (no courses completed) at semester `0`. The queue stores tuples of `(current_state, semester_count)`. The `vis` set tracks visited states to avoid redundant exploration.

**Step 3: BFS Exploration**
```python
while q:
    cur, t = q.popleft()
    if cur == (1 << (n + 1)) - 2:
        return t
```
Check if we've completed all courses. The target state is `(1 << (n + 1)) - 2`, which sets bits 1 through n (we skip bit 0 since courses are 1-indexed).

**Step 4: Find Available Courses**
```python
nxt = 0
for i in range(1, n + 1):
    if (cur & d[i]) == d[i]:
        nxt |= 1 << i
nxt ^= cur
```
For each course `i`, check if its prerequisites are satisfied by testing `(cur & d[i]) == d[i]`. This means all prerequisite bits match. Build `nxt` as a [bitmask](/problems/bitmask_intro) of all available courses. Then `nxt ^= cur` removes already completed courses, leaving only newly available ones.

**Step 5: Generate Next States**

If available courses ≤ `k`, take all of them:
```python
if nxt.bit_count() <= k:
    if (nxt | cur) not in vis:
        vis.add(nxt | cur)
        q.append((nxt | cur, t + 1))
```

If available courses > `k`, generate all combinations of exactly `k` courses:
```python
else:
    x = nxt
    while nxt:
        if nxt.bit_count() == k and (nxt | cur) not in vis:
            vis.add(nxt | cur)
            q.append((nxt | cur, t + 1))
        nxt = (nxt - 1) & x
```
The trick `(nxt - 1) & x` generates all subsets of `x`. Subtracting 1 from a number flips all trailing 1s to 0s and the first 0 to 1. ANDing with `x` ensures we only keep bits that were in the original set.

The BFS guarantees we find the minimum number of semesters since we explore states level by level, and the first time we reach the goal state gives us the shortest path.

## Example Walkthrough

Let's walk through a small example with `n = 4` courses, `k = 2` courses per semester, and prerequisites `relations = [[1,3], [2,3]]`.

**Step 1: Build Prerequisites Bitmasks**
- Course 3 has prerequisites 1 and 2: `d[3] = 0110` (bits 1 and 2 set)
- Other courses have no prerequisites: `d[1] = 0000`, `d[2] = 0000`, `d[4] = 0000`

**Step 2: BFS Initialization**
- Start state: `cur = 00000` (no courses completed)
- Queue: `[(00000, 0)]`
- Visited: `{00000}`

**Semester 1 (t = 0):**
- Current state: `cur = 00000`
- Check available courses:
  - Course 1: `(00000 & 0000) == 0000` ✓ (no prerequisites)
  - Course 2: `(00000 & 0000) == 0000` ✓ (no prerequisites)
  - Course 3: `(00000 & 0110) == 0110` ✗ (needs courses 1,2)
  - Course 4: `(00000 & 0000) == 0000` ✓ (no prerequisites)
- Available courses: `nxt = 11010` (courses 1, 2, 4)
- Since we have 3 available courses but k=2, generate all 2-course combinations:
  - Take courses 1,2: new state = `00110`
  - Take courses 1,4: new state = `10010`
  - Take courses 2,4: new state = `10100`
- Add all three states to queue with t=1

**Semester 2 (t = 1):**
Let's follow the path where we took courses 1,2:
- Current state: `cur = 00110` (courses 1,2 completed)
- Check available courses:
  - Course 3: `(00110 & 0110) == 0110` ✓ (prerequisites 1,2 satisfied)
  - Course 4: `(00110 & 0000) == 0000` ✓ (no prerequisites)
- Available courses: `nxt = 11000` (courses 3, 4)
- Since we have exactly 2 courses and k=2, take both
- New state: `11110` with t=2

**Semester 3 (t = 2):**
- Current state: `cur = 11110` (all courses completed)
- Target state is `11110` (bits 1-4 set)
- We've completed all courses! Return t=2

**Alternative paths:**
- If we had taken courses 1,4 in semester 1, we'd need:
  - Semester 2: Take course 2 (now course 3's prerequisites are met)
  - Semester 3: Take course 3
  - Total: 3 semesters

- If we had taken courses 2,4 in semester 1:
  - Semester 2: Take course 1
  - Semester 3: Take course 3
  - Total: 3 semesters

The BFS explores all paths and finds that taking courses 1,2 in the first semester leads to the minimum of 2 semesters total.

## Python Solution

```python
class Solution:
    def minNumberOfSemesters(self, n: int, relations: List[List[int]], k: int) -> int:
        # Build prerequisite bitmask for each course
        # prerequisites[i] stores a bitmask of all courses that must be taken before course i
        prerequisites = [0] * (n + 1)
        for prereq, course in relations:
            prerequisites[course] |= 1 << prereq
      
        # BFS queue: (completed_courses_bitmask, semester_count)
        queue = deque([(0, 0)])
        visited = {0}
      
        while queue:
            completed_courses, semester_count = queue.popleft()
          
            # Check if all courses are completed (bits 1 to n are set)
            # Target: 111...110 (n ones followed by a zero at bit 0)
            if completed_courses == (1 << (n + 1)) - 2:
                return semester_count
          
            # Find all courses available to take this semester
            # A course is available if all its prerequisites are completed
            available_courses = 0
            for course_id in range(1, n + 1):
                if (completed_courses & prerequisites[course_id]) == prerequisites[course_id]:
                    available_courses |= 1 << course_id
          
            # Remove already completed courses from available courses
            available_courses ^= completed_courses
          
            # If we can take all available courses this semester (count <= k)
            if available_courses.bit_count() <= k:
                new_state = available_courses | completed_courses
                if new_state not in visited:
                    visited.add(new_state)
                    queue.append((new_state, semester_count + 1))
            else:
                # Need to enumerate all possible k-sized subsets of available courses
                subset = available_courses
                while subset:
                    # Only consider subsets of exactly k courses
                    if subset.bit_count() == k:
                        new_state = subset | completed_courses
                        if new_state not in visited:
                            visited.add(new_state)
                            queue.append((new_state, semester_count + 1))
                    # Generate next subset using bit manipulation trick
                    # This iterates through all subsets of available_courses
                    subset = (subset - 1) & available_courses
```

## Time and Space Complexity

**Time Complexity:** `O(3^n)`

The algorithm uses BFS to explore different states of completed courses. Each state is represented by a bitmask where bit `i` indicates whether course `i` is completed.

- The total number of possible states is `2^n` (each course can be either taken or not taken)
- For each state, we need to find available courses (courses whose prerequisites are satisfied), which takes `O(n)` time
- When the number of available courses exceeds `k`, we enumerate all subsets of size `k`. The number of such subsets is `C(available_courses, k)`, which in the worst case can be `C(n, k)`
- The critical operation is the subset enumeration using `nxt = (nxt - 1) & x`, which iterates through all subsets of the available courses. For a set with `m` elements, this generates `2^m` subsets
- In the worst case, we might have all `n` courses available at once, leading to `2^n` subset iterations
- Since we visit each state at most once (using the `vis` set), and for each state we might enumerate up to `2^n` subsets, the overall time complexity is `O(2^n * 2^n) = O(4^n)`
- However, due to the BFS nature and the fact that we're only considering valid transitions (taking at most `k` courses at a time), the practical complexity is closer to `O(3^n)`

**Space Complexity:** `O(2^n)`

- The `vis` set stores all visited states, which can be at most `2^n` different bitmask states
- The BFS queue `q` can contain at most `O(2^n)` states in the worst case
- The dependency array `d` uses `O(n)` space
- Therefore, the overall space complexity is `O(2^n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Target State Calculation**
A frequent mistake is miscalculating the target state for course completion. Since courses are numbered from 1 to n (not 0-indexed), the target bitmask should have bits 1 through n set, not bits 0 through n-1.

**Wrong:**
```python
if completed_courses == (1 << n) - 1:  # This checks bits 0 to n-1
    return semester_count
```

**Correct:**
```python
if completed_courses == (1 << (n + 1)) - 2:  # Sets bits 1 to n
    return semester_count
# Or alternatively:
if completed_courses == ((1 << (n + 1)) - 1) ^ 1:  # More explicit
    return semester_count
```

### 2. **Inefficient Subset Generation When Available Courses > k**
When generating k-sized subsets, developers often enumerate ALL subsets first and then filter by size, which is inefficient.

**Inefficient approach:**
```python
all_subsets = []
subset = available_courses
while subset:
    all_subsets.append(subset)
    subset = (subset - 1) & available_courses

for subset in all_subsets:
    if subset.bit_count() == k:
        # Process subset
```

**Better approach (as in solution):**
Check the bit count inline to avoid storing unnecessary subsets:
```python
subset = available_courses
while subset:
    if subset.bit_count() == k:  # Check immediately
        new_state = subset | completed_courses
        if new_state not in visited:
            visited.add(new_state)
            queue.append((new_state, semester_count + 1))
    subset = (subset - 1) & available_courses
```

### 3. **Not Handling the Case Where All Available Courses Can Be Taken**
A critical optimization that's often missed: when the number of available courses is ≤ k, we should take all of them in one semester rather than generating all possible subsets.

**Inefficient (always generating subsets):**
```python
# Always enumerate subsets, even when unnecessary
subset = available_courses
while subset:
    if subset.bit_count() <= k:  # Note: using <= instead of ==
        # Process subset
    subset = (subset - 1) & available_courses
```

**Correct optimization:**
```python
if available_courses.bit_count() <= k:
    # Take all available courses at once
    new_state = available_courses | completed_courses
    if new_state not in visited:
        visited.add(new_state)
        queue.append((new_state, semester_count + 1))
else:
    # Only enumerate k-sized subsets when necessary
    # ... subset generation code
```

### 4. **Forgetting to Remove Already Completed Courses from Available Set**
After determining which courses have their prerequisites satisfied, you must remove courses that are already completed.

**Wrong:**
```python
available_courses = 0
for course_id in range(1, n + 1):
    if (completed_courses & prerequisites[course_id]) == prerequisites[course_id]:
        available_courses |= 1 << course_id
# Forgot to remove completed courses!
```

**Correct:**
```python
available_courses = 0
for course_id in range(1, n + 1):
    if (completed_courses & prerequisites[course_id]) == prerequisites[course_id]:
        available_courses |= 1 << course_id
# Remove already completed courses
available_courses ^= completed_courses  # XOR removes completed courses
```

### 5. **Using Wrong Bit Manipulation for Subset Generation**
The subset generation formula `(subset - 1) & original` is subtle and easy to get wrong.

**Common mistakes:**
```python
# Wrong: This doesn't generate proper subsets
subset = (subset - 1) & available_courses - 1

# Wrong: This might skip subsets
subset = subset - 1 & available_courses  # Operator precedence issue

# Wrong: Infinite loop
subset = (subset + 1) & available_courses
```

**Correct:**
```python
subset = (subset - 1) & available_courses
```
The parentheses around `(subset - 1)` are crucial due to operator precedence.

