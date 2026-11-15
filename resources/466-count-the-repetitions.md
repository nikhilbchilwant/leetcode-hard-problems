# 466. Count The Repetitions

> Source: https://algo.monster/liteproblems/466

## Problem Statement

This problem asks you to find how many times a repeated string can be obtained from another repeated string through character removal.

First, let's understand the notation `[s, n]` which means string `s` repeated `n` times. For example, `["abc", 3]` gives us `"abcabcabc"`.

The problem defines that string `s1` can be "obtained from" string `s2` if we can remove some characters from `s2` to get `s1`. For instance, `"abc"` can be obtained from `"ab**dbe**c"` by removing the characters 'd', 'b', and 'e'.

Given:
- Two strings `s1` and `s2`
- Two integers `n1` and `n2`
- This creates `str1 = [s1, n1]` (string `s1` repeated `n1` times)
- And `str2 = [s2, n2]` (string `s2` repeated `n2` times)

The task is to find the maximum integer `m` such that `[str2, m]` (which means `str2` repeated `m` times) can be obtained from `str1`.

In other words, you need to determine how many complete copies of `str2` can be formed by removing characters from `str1`, where both `str1` and `str2` are themselves repeated strings.

For example, if `s1 = "acb"`, `n1 = 4`, `s2 = "ab"`, `n2 = 2`:
- `str1 = "acbacbacbacb"` (repeating "acb" 4 times)
- `str2 = "abab"` (repeating "ab" 2 times)
- From `str1`, we can obtain multiple copies of `str2` by removing appropriate characters
- The answer would be the maximum number of times `str2` can be obtained from `str1`

## Intuition

The key insight is that when we're matching characters from `s1` to form `s2`, there's a repeating pattern that emerges. Since we're dealing with repeated strings, after processing one complete `s1`, we'll be at some position in `s2`. This position and the number of complete `s2` strings we've formed create a cycle.

Think about it this way: when we scan through one complete `s1`, we're essentially asking "starting from position `i` in `s2`, how many complete `s2` strings can I form, and where will I end up in `s2`?" Since `s2` has a finite length, there are only a limited number of starting positions (0 to `len(s2)-1`).

The brilliant observation is that we can precompute this information for all possible starting positions in `s2`. For each starting position `i` in `s2`, we can determine:
- How many complete `s2` strings we can form by going through one `s1` (let's call this `cnt`)
- What position in `s2` we'll be at after processing one `s1` (let's call this `j`)

This creates a mapping: `d[i] = (cnt, j)`. This tells us that if we start at position `i` in `s2` and process one complete `s1`, we'll form `cnt` complete `s2` strings and end up at position `j` in `s2`.

Once we have this mapping, simulating the entire process becomes straightforward. We just need to iterate `n1` times (for each repetition of `s1`), and for each iteration, we look up our current position in the precomputed table to find how many `s2` strings we formed and where we'll be next.

The final step is dividing by `n2` because we're looking for complete `str2` strings (which is `s2` repeated `n2` times), not just complete `s2` strings.

This approach transforms what could be a character-by-character simulation of potentially very long strings into a simple iteration with precomputed state transitions.

## Solution Approach

The implementation follows a two-phase approach: preprocessing and iteration.

**Phase 1: Preprocessing**

We create a dictionary `d` to store the state transitions. For each possible starting position `i` in `s2` (where `i` ranges from 0 to `len(s2)-1`), we simulate matching one complete `s1`:

```python
for i in range(n):  # n is the length of s2
    cnt = 0  # count of complete s2 strings formed
    j = i    # current position in s2
    for c in s1:
        if c == s2[j]:
            j += 1
        if j == n:  # reached the end of s2
            cnt += 1
            j = 0   # wrap around to start of s2
    d[i] = (cnt, j)
```

For each starting position `i`, we:
- Initialize a counter `cnt` to track complete `s2` strings formed
- Set current position `j` to `i`
- Iterate through each character `c` in `s1`:
  - If `c` matches the current character in `s2` at position `j`, advance `j`
  - If we've completed `s2` (when `j == n`), increment `cnt` and reset `j` to 0
- Store the result as `d[i] = (cnt, j)`, where `cnt` is the number of complete `s2` strings and `j` is the final position in `s2`

**Phase 2: Iteration**

Now we simulate processing `n1` copies of `s1`:

```python
ans = 0
j = 0  # start at position 0 in s2
for _ in range(n1):
    cnt, j = d[j]  # lookup precomputed values
    ans += cnt     # add complete s2 strings formed
```

We:
- Start with position `j = 0` in `s2`
- For each of the `n1` repetitions of `s1`:
  - Look up `d[j]` to get the number of complete `s2` strings and the next position
  - Add `cnt` to our running total
  - Update `j` to the new position for the next iteration

**Final Calculation**

Since we want complete `str2` strings (where `str2 = [s2, n2]`), we divide the total count of `s2` strings by `n2`:

```python
return ans // n2
```

This approach has a time complexity of `O(|s2| × |s1| + n1)` where the first term is for preprocessing and the second for iteration. The space complexity is `O(|s2|)` for storing the transition table.

## Example Walkthrough

Let's walk through a concrete example with `s1 = "acb"`, `n1 = 4`, `s2 = "ab"`, `n2 = 2`.

First, we need to understand what we're looking for:
- `str1 = "acbacbacbacb"` (repeating "acb" 4 times)
- `str2 = "abab"` (repeating "ab" 2 times)
- We want to find how many complete `str2` strings can be obtained from `str1`

**Phase 1: Build the transition table**

We'll create a dictionary `d` where `d[i]` tells us what happens when we start at position `i` in `s2` and process one complete `s1`.

For `s2 = "ab"`, we have positions 0 and 1:

*Starting at position 0 in s2:*
- We need to match 'a' (at position 0 in s2)
- Process s1 = "acb":
  - 'a' matches s2[0='a'] → advance to position 1
  - 'c' doesn't match s2[1='b'] → stay at position 1
  - 'b' matches s2[1='b'] → advance to position 2 (which wraps to 0), we completed one s2!
- Result: `d[0] = (1, 0)` (completed 1 copy of s2, ended at position 0)

*Starting at position 1 in s2:*
- We need to match 'b' (at position 1 in s2)
- Process s1 = "acb":
  - 'a' doesn't match s2[1='b'] → stay at position 1
  - 'c' doesn't match s2[1='b'] → stay at position 1
  - 'b' matches s2[1='b'] → advance to position 2 (which wraps to 0), we completed one s2!
- Result: `d[1] = (1, 0)` (completed 1 copy of s2, ended at position 0)

**Phase 2: Simulate n1 iterations**

Now we process 4 copies of s1, starting at position 0 in s2:

- Iteration 1: Start at position 0 → `d[0] = (1, 0)` → gained 1 s2, now at position 0
- Iteration 2: Start at position 0 → `d[0] = (1, 0)` → gained 1 s2, now at position 0
- Iteration 3: Start at position 0 → `d[0] = (1, 0)` → gained 1 s2, now at position 0
- Iteration 4: Start at position 0 → `d[0] = (1, 0)` → gained 1 s2, now at position 0

Total s2 strings formed: 1 + 1 + 1 + 1 = 4

**Final Calculation**

We formed 4 complete s2 strings. Since str2 = [s2, 2] (meaning 2 copies of s2), we have:
- Number of complete str2 strings = 4 // 2 = 2

Therefore, the answer is 2. We can obtain str2 = "abab" exactly 2 times from str1 = "acbacbacbacb".

To verify, number the characters of "acbacbacbacb" from 0 to 11 (`a` at 0, 3, 6, 9 and `b` at 2, 5, 8, 11):
- First "abab": characters 0, 2, 3, 5 (`a`, `b`, `a`, `b`)
- Second "abab": characters 6, 8, 9, 11 (`a`, `b`, `a`, `b`)

## Python Solution

```python
class Solution:
    def getMaxRepetitions(self, s1: str, n1: int, s2: str, n2: int) -> int:
        """
        Find the maximum number of times str2 = s2 * n2 can be obtained from str1 = s1 * n1.
      
        Args:
            s1: The base string to be repeated n1 times
            n1: Number of times s1 is repeated
            s2: The pattern string to be repeated n2 times  
            n2: Number of times s2 is repeated
          
        Returns:
            Maximum number of str2 that can be obtained from str1
        """
        s2_length = len(s2)
      
        # Build a transition table for each starting position in s2
        # Key: starting index in s2
        # Value: (number of complete s2 cycles, ending index in s2) after processing one s1
        transition_table = {}
      
        for start_index in range(s2_length):
            complete_cycles = 0
            current_index = start_index
          
            # Process one complete s1 string
            for char in s1:
                if char == s2[current_index]:
                    current_index += 1
                  
                # Check if we completed one cycle of s2
                if current_index == s2_length:
                    complete_cycles += 1
                    current_index = 0
                  
            transition_table[start_index] = (complete_cycles, current_index)
      
        # Simulate processing n1 copies of s1
        total_s2_cycles = 0
        current_s2_index = 0
      
        for _ in range(n1):
            # Get transition result from current position
            cycles_gained, next_index = transition_table[current_s2_index]
            total_s2_cycles += cycles_gained
            current_s2_index = next_index
      
        # Calculate how many complete str2 (s2 * n2) we can form
        return total_s2_cycles // n2
```

## Time and Space Complexity

**Time Complexity:** `O(m × n + n₁)`

The algorithm consists of two main parts:

1. **Preprocessing phase:** Building the dictionary `d` by iterating through all `n` possible starting positions in `s2`. For each starting position `i` (where `i` ranges from `0` to `n-1`), we iterate through all characters in `s1` (length `m`) to count how many complete `s2` sequences can be formed. This gives us `O(m × n)` time complexity for the preprocessing.

2. **Simulation phase:** We then iterate `n1` times to simulate going through `s1` for `n1` repetitions. In each iteration, we perform a constant-time lookup in the dictionary `d` to get the precomputed values `(cnt, j)`. This phase takes `O(n₁)` time.

Therefore, the total time complexity is `O(m × n + n₁)`, where `m = len(s1)`, `n = len(s2)`, and `n₁` is the repetition count for `s1`.

**Space Complexity:** `O(n)`

The algorithm uses a dictionary `d` to store precomputed values for each possible starting position in `s2`. Since there are exactly `n` possible starting positions (indices `0` to `n-1`), and each entry stores a tuple of two integers `(cnt, j)`, the space complexity is `O(n)`, where `n = len(s2)`.

## Common Pitfalls

### 1. **Not Handling Empty or Invalid Input**
The code assumes both `s1` and `s2` contain valid characters and are non-empty. If `s2` is empty or `n1`/`n2` are zero, the code will fail with division by zero or other errors.

**Solution:**
```python
def getMaxRepetitions(self, s1: str, n1: int, s2: str, n2: int) -> int:
    # Add input validation
    if not s1 or not s2 or n1 == 0 or n2 == 0:
        return 0
  
    # Check if s1 contains all characters needed for s2
    if not all(c in s1 for c in set(s2)):
        return 0
  
    # Rest of the implementation...
```

### 2. **Cycle Detection Optimization Missed**
The current solution simulates all `n1` iterations linearly, which can be inefficient when `n1` is very large. Since the state transitions form a cycle, we could detect and skip the cycle.

**Solution:**
```python
def getMaxRepetitions(self, s1: str, n1: int, s2: str, n2: int) -> int:
    s2_length = len(s2)
    transition_table = {}
  
    # Build transition table (same as before)
    for start_index in range(s2_length):
        # ... (preprocessing code)
        transition_table[start_index] = (complete_cycles, current_index)
  
    # Detect cycle using Floyd's algorithm or state tracking
    visited = {}
    total_s2_cycles = 0
    current_s2_index = 0
  
    for iteration in range(n1):
        if current_s2_index in visited:
            # Cycle detected
            cycle_start = visited[current_s2_index][0]
            cycle_length = iteration - cycle_start
            cycles_in_cycle = total_s2_cycles - visited[current_s2_index][1]
          
            # Calculate remaining iterations
            remaining = n1 - iteration
            full_cycles = remaining // cycle_length
            remainder = remaining % cycle_length
          
            # Add cycles from full repetitions
            total_s2_cycles += full_cycles * cycles_in_cycle
          
            # Process remainder
            for _ in range(remainder):
                cycles_gained, current_s2_index = transition_table[current_s2_index]
                total_s2_cycles += cycles_gained
          
            return total_s2_cycles // n2
      
        visited[current_s2_index] = (iteration, total_s2_cycles)
        cycles_gained, current_s2_index = transition_table[current_s2_index]
        total_s2_cycles += cycles_gained
  
    return total_s2_cycles // n2
```

### 3. **Integer Overflow for Large Values**
When `n1` is very large (up to 10^6), the multiplication and accumulation of cycles could potentially cause issues in languages with fixed integer sizes (though Python handles arbitrary precision integers).

**Solution:**
Early termination when we know we can't form any more complete `str2`:
```python
def getMaxRepetitions(self, s1: str, n1: int, s2: str, n2: int) -> int:
    # ... (previous code)
  
    # During simulation
    for iteration in range(n1):
        cycles_gained, current_s2_index = transition_table[current_s2_index]
        total_s2_cycles += cycles_gained
      
        # Early termination: if no progress is possible
        if cycles_gained == 0 and current_s2_index == 0:
            # We've completed a full cycle without any matches
            break
  
    return total_s2_cycles // n2
```

