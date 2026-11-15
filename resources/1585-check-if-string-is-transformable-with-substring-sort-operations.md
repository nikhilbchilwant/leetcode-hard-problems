# 1585. Check If String Is Transformable With Substring Sort Operations

> Source: https://algo.monster/liteproblems/1585

## Problem Statement

You are given two strings `s` and `t` of equal length containing only digits. Your task is to determine if you can transform string `s` into string `t` by performing a specific operation any number of times.

The allowed operation is:
- Select any non-empty substring in `s` and sort it in ascending order in place

For example, if you have the string `"14234"` and you select the substring `"4234"` (from index 1 to 4), [sorting](/problems/sorting_summary) it would give you `"12344"`.

The problem asks you to return `true` if it's possible to transform `s` into `t` through any sequence of these [sorting](/problems/sorting_summary) operations, and `false` otherwise.

The key insight is that this [sorting](/problems/sorting_summary) operation is similar to bubble sort - you can only move smaller digits to the left by sorting substrings. This means:
- A digit can move left past larger digits (by sorting a substring containing both)
- A digit cannot move left past smaller digits (sorting would keep the smaller digit on the left)

For instance, in `"321"`, you could transform it to `"123"` by [sorting](/problems/sorting_summary) the entire string. But you could never transform `"123"` into `"321"` because the sorting operation only moves smaller values leftward.

## Intuition

The key observation is that the [sorting](/problems/sorting_summary) operation has a specific restriction: it can only move smaller elements to the left. This is exactly how bubble sort works - smaller elements "bubble up" to earlier positions.

Think about what happens when we sort a substring. If we have `"42"` and sort it, we get `"24"`. The smaller digit `2` moved left, and the larger digit `4` moved right. We can never use [sorting](/problems/sorting_summary) to move a larger digit past a smaller digit to the left.

This means if we want to transform `s` into `t`, for each digit in `t` (reading left to right), we need to find that digit in `s` and "bubble" it to the correct position. But here's the catch - we can only move a digit leftward if there are no smaller digits blocking its path.

For example, if `s = "8940"` and `t = "0984"`, to get the `0` to the front in `t`, we need to move the `0` from position 3 in `s` to position 0. This is possible because `0` is the smallest digit - it can bubble past `8`, `9`, and `4`.

However, if `s = "8940"` and `t = "9804"`, we cannot achieve this transformation. Why? To get `9` at position 0 in `t`, we'd use the `9` at position 1 in `s`. But to get `8` at position 1 in `t`, we'd need to move the `8` from position 0 in `s` to the right, past the `9`. This is impossible - [sorting](/problems/sorting_summary) never moves larger digits left past smaller ones.

This leads to our strategy: process `t` from left to right. For each digit we need, find its leftmost available occurrence in `s`. Check if any smaller digit appears before this position - if yes, the transformation is impossible. If no, we can safely "use" this occurrence and move on.

By tracking positions of each digit in `s` using queues (to maintain order and easily access/remove the leftmost occurrence), we can efficiently verify if the transformation is possible.

**Learn more about [Greedy](/problems/greedy_intro) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

The solution implements the bubble sort intuition using a [greedy](/problems/greedy_intro) approach with position tracking.

**Data Structure Setup:**
We use a dictionary `pos` where each key (digit 0-9) maps to a deque containing all positions where that digit appears in string `s`. A deque is used because we need efficient removal from the front (leftmost positions).

```python
pos = defaultdict(deque)
for i, c in enumerate(s):
    pos[int(c)].append(i)
```

For example, if `s = "8940"`, then:
- `pos[8] = deque([0])`
- `pos[9] = deque([1])` 
- `pos[4] = deque([2])`
- `pos[0] = deque([3])`

**Main Algorithm:**
We iterate through string `t` character by character. For each character in `t`, we convert it to digit `x` and perform two crucial checks:

1. **Availability Check:** `if not pos[x]`
   - If there are no more occurrences of digit `x` in `s`, transformation is impossible
   - Return `False`

2. **Blocking Check:** `any(pos[i] and pos[i][0] < pos[x][0] for i in range(x))`
   - Check if any smaller digit (0 to x-1) has its leftmost occurrence before the leftmost occurrence of `x`
   - If yes, we cannot bubble `x` to the current position because a smaller digit blocks it
   - Return `False`

If both checks pass, we remove the leftmost occurrence of `x` from its deque using `pos[x].popleft()`, marking it as "used" for the current position in `t`.

**Why This Works:**
- By processing `t` from left to right and always using the leftmost available occurrence of each digit from `s`, we ensure we're making the most optimal choices
- The blocking check ensures we respect the bubble sort constraint: we can only move a digit left if no smaller digits are in its way
- Using deques maintains the order of positions and allows O(1) access to the leftmost position

**Time Complexity:** O(n × d) where n is the length of strings and d is the number of distinct digits (at most 10)
**Space Complexity:** O(n) for storing positions

## Example Walkthrough

Let's walk through an example where `s = "8940"` and `t = "0894"`.

**Step 1: Build position tracking**
First, we create a dictionary of deques to track where each digit appears in `s`:
- `pos[8] = deque([0])` - digit 8 is at position 0
- `pos[9] = deque([1])` - digit 9 is at position 1  
- `pos[4] = deque([2])` - digit 4 is at position 2
- `pos[0] = deque([3])` - digit 0 is at position 3

**Step 2: Process t = "0894" character by character**

*Processing t[0] = '0':*
- We need digit 0, check `pos[0] = deque([3])` - it exists at position 3
- Check if any smaller digits (none exist, since 0 is the smallest) block it: No blockers
- We can use the 0 at position 3. Remove it: `pos[0].popleft()` → `pos[0] = deque([])`

*Processing t[1] = '8':*
- We need digit 8, check `pos[8] = deque([0])` - it exists at position 0
- Check if any smaller digits block it:
  - Check digits 0-7: `pos[0]` is empty (we used it), others don't exist
  - No blockers found
- We can use the 8 at position 0. Remove it: `pos[8].popleft()` → `pos[8] = deque([])`

*Processing t[2] = '9':*
- We need digit 9, check `pos[9] = deque([1])` - it exists at position 1
- Check if any smaller digits block it:
  - Check digits 0-8: All are either empty or don't exist
  - No blockers found
- We can use the 9 at position 1. Remove it: `pos[9].popleft()` → `pos[9] = deque([])`

*Processing t[3] = '4':*
- We need digit 4, check `pos[4] = deque([2])` - it exists at position 2
- Check if any smaller digits block it:
  - Check digits 0-3: All are either empty or don't exist
  - No blockers found
- We can use the 4 at position 2. Remove it: `pos[4].popleft()` → `pos[4] = deque([])`

**Result:** All characters in `t` were successfully matched without any blocking issues. Return `True`.

The transformation is possible because we can sort the entire string "8940" to get "0489", then sort substring "489" to get "0849", and finally sort substring "49" to get "0894".

## Python Solution

```python
from collections import defaultdict, deque
from typing import Deque, DefaultDict

class Solution:
    def isTransformable(self, s: str, t: str) -> bool:
        # Create a dictionary to store positions of each digit in string s
        # Key: digit (0-9), Value: deque of positions where this digit appears
        digit_positions: DefaultDict[int, Deque[int]] = defaultdict(deque)
      
        # Populate the position queues for each digit in string s
        for index, char in enumerate(s):
            digit = int(char)
            digit_positions[digit].append(index)
      
        # Process each character in target string t
        for char in t:
            target_digit = int(char)
          
            # Check if target digit exists in remaining positions
            if not digit_positions[target_digit]:
                return False
          
            # Get the leftmost position of the target digit
            target_position = digit_positions[target_digit][0]
          
            # Check if any smaller digit appears before the target digit's position
            # This would block the transformation (smaller digits can't move right past larger ones)
            for smaller_digit in range(target_digit):
                if digit_positions[smaller_digit] and digit_positions[smaller_digit][0] < target_position:
                    return False
          
            # Remove the used position of the target digit
            digit_positions[target_digit].popleft()
      
        # All characters successfully matched
        return True
```

## Time and Space Complexity

**Time Complexity:** `O(n × C)` where `n` is the length of string `s` and `C` is the size of the digit set (10 for digits 0-9).

- Building the position queues takes `O(n)` time as we iterate through string `s` once
- For each character in string `t` (total `n` characters):
  - We check `any(pos[i] and pos[i][0] < pos[x][0] for i in range(x))`, which iterates through at most `C` digits (0 to x-1)
  - Each check `pos[i] and pos[i][0] < pos[x][0]` takes `O(1)` time
  - The `popleft()` operation takes `O(1)` time
- Overall: `O(n) + O(n × C) = O(n × C)`

**Space Complexity:** `O(n)`

- The `pos` dictionary stores at most `n` positions distributed across up to `C` deques
- Since each position in string `s` is stored exactly once, the total space used is `O(n)`
- The number of digits `C` is constant (10), so it doesn't affect the asymptotic space complexity

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Misunderstanding the Direction of Movement**
A critical pitfall is misunderstanding how the sorting operation affects digit movement. Many developers incorrectly assume digits can move in both directions freely.

**The Pitfall:** Thinking that sorting allows any rearrangement, when actually:
- Digits can only move **left** past **larger** digits
- Digits **cannot** move left past **smaller** digits
- This is a one-way constraint similar to bubble sort

**Example of the mistake:**
```python
# WRONG: Checking if larger digits block movement
for larger_digit in range(target_digit + 1, 10):
    if digit_positions[larger_digit] and digit_positions[larger_digit][0] < target_position:
        return False
```

**Correct approach:**
```python
# RIGHT: Checking if smaller digits block movement
for smaller_digit in range(target_digit):
    if digit_positions[smaller_digit] and digit_positions[smaller_digit][0] < target_position:
        return False
```

### 2. **Using the Wrong Data Structure**
Using a list instead of a deque for storing positions can lead to performance issues.

**The Pitfall:** Using `list.pop(0)` which is O(n) operation:
```python
# INEFFICIENT: O(n) removal from front
digit_positions = defaultdict(list)
# Later in code:
digit_positions[target_digit].pop(0)  # O(n) operation!
```

**Solution:** Use `deque.popleft()` which is O(1):
```python
# EFFICIENT: O(1) removal from front
digit_positions = defaultdict(deque)
# Later in code:
digit_positions[target_digit].popleft()  # O(1) operation!
```

### 3. **Not Using the Leftmost Available Position**
A subtle but crucial mistake is not always selecting the leftmost occurrence of the required digit.

**The Pitfall:** Trying to be "clever" by selecting a different occurrence:
```python
# WRONG: Trying to find "best" position instead of leftmost
for pos in digit_positions[target_digit]:
    if can_reach(pos):  # Some custom logic
        use_position(pos)
        break
```

**Why this fails:** The greedy approach of always using the leftmost available position is optimal. Using any other position could block future transformations unnecessarily.

**Correct approach:** Always use the leftmost position:
```python
# RIGHT: Always use the leftmost available position
target_position = digit_positions[target_digit][0]
```

### 4. **Forgetting to Check Digit Availability**
Assuming both strings have the same character frequency without verification.

**The Pitfall:** Only checking the blocking condition:
```python
# INCOMPLETE: Missing availability check
def isTransformable(self, s: str, t: str) -> bool:
    # ... setup code ...
    for char in t:
        target_digit = int(char)
        # MISSING: if not digit_positions[target_digit]: return False
      
        # Directly accessing without checking availability
        target_position = digit_positions[target_digit][0]  # Could raise IndexError!
```

**Solution:** Always check availability before accessing:
```python
# COMPLETE: Check availability first
if not digit_positions[target_digit]:
    return False
target_position = digit_positions[target_digit][0]
```

### 5. **Incorrect Blocking Condition Logic**
Using the wrong comparison or range when checking for blocking digits.

**The Pitfall:** Off-by-one errors or wrong comparison operators:
```python
# WRONG: Including the target digit itself in the blocking check
for digit in range(target_digit + 1):  # Should be range(target_digit)
  
# WRONG: Using <= instead of <
if digit_positions[smaller_digit][0] <= target_position:  # Should be <
```

**Correct implementation:**
```python
# RIGHT: Check only smaller digits with correct comparison
for smaller_digit in range(target_digit):
    if digit_positions[smaller_digit] and digit_positions[smaller_digit][0] < target_position:
        return False
```

