# 358. Rearrange String k Distance Apart

> Source: https://algo.monster/liteproblems/358

## Problem Statement

You are given a string `s` and an integer `k`. Your task is to rearrange the characters in the string such that any two identical characters are separated by **at least** `k` distance from each other.

For example, if you have the character 'a' at position `i`, then the next 'a' cannot appear until position `i + k` or later.

If it's impossible to create such a rearrangement where all identical characters maintain this minimum distance requirement, you should return an empty string `""`.

The key challenge is to strategically place characters to ensure the distance constraint is satisfied for all characters. Characters that appear more frequently in the original string will be harder to place since they need more positions while maintaining the required spacing.

Example scenarios:
- If `s = "aabbcc"` and `k = 3`, one valid rearrangement could be `"abcabc"` where each pair of identical characters is exactly 3 positions apart
- If `s = "aaabc"` and `k = 3`, it would be impossible to arrange since we have three 'a's that each need to be at least 3 positions apart, requiring more total positions than available
- If `k = 0` or `k = 1`, any arrangement works since there's no effective distance requirement

## Intuition

The key insight is that we need to greedily place the most frequent characters first to avoid getting stuck later. Think about it this way: if a character appears many times, it needs many "slots" with proper spacing. If we delay placing these high-frequency characters, we might run out of valid positions.

Consider a greedy strategy: always pick the character with the highest remaining count that is currently available (not in the recent `k-1` positions). This ensures we're always making progress on the characters that are hardest to place.

To implement this strategy, we need two main components:

1. **A max heap** to always know which character has the highest remaining count among those currently available to place
2. **A cooling queue** to temporarily hold characters that were recently used and cannot be placed yet

The process works like this:
- Pick the character with the highest count from the heap
- Place it in the result
- Decrease its count and put it in a "cooling" queue
- After exactly `k` positions, the character becomes available again and returns to the heap

The cooling mechanism is crucial: when we place a character at position `i`, it cannot be used again until position `i+k`. By using a queue of size `k`, we naturally enforce this constraint - a character enters the queue after being used and only exits (becoming available again) after `k` other characters have been placed.

If at any point we cannot place a character (the heap is empty but we still have characters to place), it means the arrangement is impossible. This happens when the cooling queue contains characters that still need to be placed, but there are no available characters to fill the gap.

The beauty of this approach is that it automatically handles the distance constraint through the queue mechanism while ensuring we prioritize characters that are most likely to cause problems if delayed.

## Solution Approach

Let's walk through the implementation step by step:

**1. Character Frequency Count and Max Heap Setup**
```python
h = [(-v, c) for c, v in Counter(s).items()]
heapify(h)
```
- First, we count the frequency of each character using `Counter(s)`
- We create a max heap by negating the frequencies (Python's heapq is a min heap by default)
- Each heap element is a tuple `(-frequency, character)`

**2. Initialize Data Structures**
```python
q = deque()
ans = []
```
- `q` is our cooling queue that holds characters temporarily unavailable
- `ans` accumulates our result string

**3. Main Processing Loop**
```python
while h:
    v, c = heappop(h)
    v *= -1
    ans.append(c)
    q.append((v - 1, c))
```
- While we have available characters in the heap:
  - Pop the character with highest frequency
  - Convert back to positive frequency (`v *= -1`)
  - Add this character to our result
  - Decrease its count and add to cooling queue

**4. Cooling Queue Management**
```python
if len(q) >= k:
    w, c = q.popleft()
    if w:
        heappush(h, (-w, c))
```
- Once the cooling queue reaches size `k`, the oldest character has "cooled down"
- We remove it from the queue
- If it still has remaining count (`w > 0`), push it back to the heap with negated frequency

**5. Validation and Return**
```python
return "" if len(ans) != len(s) else "".join(ans)
```
- If we successfully placed all characters, `len(ans)` equals `len(s)`
- Otherwise, some characters got stuck in the cooling queue, making arrangement impossible

**Why This Works:**

The algorithm ensures that after a character is placed at position `i`, it stays in the queue while the next `k - 1` characters are placed, so its earliest next position is `i + k`. The max heap guarantees we always choose the most frequent available character, preventing scenarios where high-frequency characters get blocked later.

**Time Complexity:** `O(n log m)` where `n` is the length of string and `m` is the number of unique characters
**Space Complexity:** `O(m)` for the heap and queue

## Example Walkthrough

Let's trace through the algorithm with `s = "aabbcc"` and `k = 3`.

**Initial Setup:**
- Character frequencies: `{'a': 2, 'b': 2, 'c': 2}`
- Max heap (negated): `[(-2, 'a'), (-2, 'b'), (-2, 'c')]`
- Cooling queue: `[]`
- Result: `[]`

**Iteration 1:**
- Pop from heap: `(-2, 'a')` → frequency = 2 (all counts tie, so the heap breaks the tie by character and `'a'` comes first)
- Add 'a' to result: `['a']`
- Add to cooling queue with decreased count: `[(1, 'a')]`
- Queue size (1) < k (3), so nothing returns to heap
- Heap: `[(-2, 'b'), (-2, 'c')]`

**Iteration 2:**
- Pop from heap: `(-2, 'b')` → frequency = 2
- Add 'b' to result: `['a', 'b']`
- Add to cooling queue: `[(1, 'a'), (1, 'b')]`
- Queue size (2) < k (3), so nothing returns to heap
- Heap: `[(-2, 'c')]`

**Iteration 3:**
- Pop from heap: `(-2, 'c')` → frequency = 2
- Add 'c' to result: `['a', 'b', 'c']`
- Add to cooling queue: `[(1, 'a'), (1, 'b'), (1, 'c')]`
- Queue size (3) = k (3), so first element `(1, 'a')` exits queue
- Since count = 1 > 0, push 'a' back to heap with `(-1, 'a')`
- Heap: `[(-1, 'a')]`

**Iteration 4:**
- Pop from heap: `(-1, 'a')` → frequency = 1
- Add 'a' to result: `['a', 'b', 'c', 'a']`
- Add to cooling queue: `[(1, 'b'), (1, 'c'), (0, 'a')]`
- Queue size (3) = k (3), so `(1, 'b')` exits queue
- Push 'b' back to heap with `(-1, 'b')`
- Heap: `[(-1, 'b')]`

**Iteration 5:**
- Pop from heap: `(-1, 'b')` → frequency = 1
- Add 'b' to result: `['a', 'b', 'c', 'a', 'b']`
- Add to cooling queue: `[(1, 'c'), (0, 'a'), (0, 'b')]`
- Queue size (3) = k (3), so `(1, 'c')` exits queue
- Push 'c' back to heap with `(-1, 'c')`
- Heap: `[(-1, 'c')]`

**Iteration 6:**
- Pop from heap: `(-1, 'c')` → frequency = 1
- Add 'c' to result: `['a', 'b', 'c', 'a', 'b', 'c']`
- Add to cooling queue: `[(0, 'a'), (0, 'b'), (0, 'c')]`
- Queue size (3) = k (3), so `(0, 'a')` exits queue
- Count = 0, so don't push back to heap
- Heap: `[]` (empty)

**Final Check:**
- Heap is empty, loop ends
- Result length (6) = original string length (6) ✓
- Return: `"abcabc"`

Each identical character is exactly 3 positions apart: a's at positions 0 and 3, b's at positions 1 and 4, c's at positions 2 and 5.

## Python Solution

```python
from collections import Counter, deque
from heapq import heapify, heappop, heappush

class Solution:
    def rearrangeString(self, s: str, k: int) -> str:
        # Edge case: if k is 0 or 1, any arrangement works
        if k <= 1:
            return s

        # Count frequency of each character and create max heap
        # Using negative values to simulate max heap (since Python has min heap by default)
        char_freq = Counter(s)
        max_heap = [(-freq, char) for char, freq in char_freq.items()]
        heapify(max_heap)

        # Queue to hold characters that are in cooldown period
        cooldown_queue = deque()

        # Result list to build the rearranged string
        result = []

        # Process characters from heap
        while max_heap:
            # Get the character with highest frequency
            neg_freq, char = heappop(max_heap)
            freq = -neg_freq

            # Add character to result
            result.append(char)

            # Add character to cooldown queue with decremented frequency
            cooldown_queue.append((freq - 1, char))

            # If cooldown period is over (queue has k elements),
            # the first element can be added back to heap
            if len(cooldown_queue) >= k:
                remaining_freq, cooldown_char = cooldown_queue.popleft()
                if remaining_freq > 0:
                    heappush(max_heap, (-remaining_freq, cooldown_char))

        # If we couldn't place all characters, return empty string
        # This happens when the cooldown constraint cannot be satisfied
        if len(result) != len(s):
            return ""

        return "".join(result)
```

## Time and Space Complexity

**Time Complexity:** `O(n log m)` where `n` is the length of the input string and `m` is the number of unique characters (at most 26 for lowercase English letters).

- Creating the frequency counter takes `O(n)` time
- Building the initial heap from the counter takes `O(m)` time to create the list and `O(m)` time for heapify operation
- The main while loop runs exactly `n` iterations (once for each character in the string)
- In each iteration:
  - `heappop()` operation: `O(log m)`
  - Append to answer list: `O(1)`
  - Append to queue: `O(1)`
  - Conditionally `heappush()` operation: `O(log m)`
- Total: `O(n)` + `O(m)` + `O(n × log m)` = `O(n log m)`

Since `m ≤ 26` for English letters, this can also be considered `O(n)` in practice.

**Space Complexity:** `O(m)` or `O(1)` if we consider the alphabet size as constant

- Counter storage: `O(m)` for storing frequency of unique characters
- Heap storage: `O(m)` for at most `m` unique characters
- Queue storage: `O(k)` but since `k ≤ m`, this is bounded by `O(m)`
- Answer list: `O(n)` but this is typically not counted as it's the output
- Total auxiliary space: `O(m)`

Since the number of unique characters is bounded by 26 for lowercase English letters, the space complexity can be considered `O(1)` constant space.

## Common Pitfalls

### 1. **Forgetting the Edge Case When k = 0**

**The Pitfall:**
Many implementations forget that when `k = 0`, there's no distance requirement between identical characters. The main algorithm with the cooldown queue becomes unnecessary work.

**Why It Happens:**
```python
# Without handling k = 0, this condition fails:
if len(cooldown_queue) >= k:  # When k = 0, this is always True
    # Characters immediately get popped from queue
```

When `k = 0`, each character goes into the queue and immediately comes back out to the heap, so the loop still returns a valid string, but it spends `O(n log m)` time on an input that needs no rearranging. The early return makes the case explicit and skips the heap entirely.

**The Solution:**
```python
# Add this check at the beginning
if k <= 1:
    return s  # Any arrangement is valid
```

### 2. **Incorrect Cooldown Queue Size Check**

**The Pitfall:**
Using `>` instead of `>=` when checking if the cooldown period has elapsed:

```python
# WRONG - This makes characters wait one extra position
if len(cooldown_queue) > k:
    remaining_freq, cooldown_char = cooldown_queue.popleft()

# CORRECT - Character becomes available exactly after k positions
if len(cooldown_queue) >= k:
    remaining_freq, cooldown_char = cooldown_queue.popleft()
```

**Why It Matters:**
- With `>`, a character at position `i` wouldn't be available until position `i + k + 1`
- This unnecessarily restricts placement options and can cause valid arrangements to fail

### 3. **Not Checking for Remaining Characters in Cooldown Queue**

**The Pitfall:**
Assuming that if the main loop completes, the arrangement is valid:

```python
# WRONG - Missing validation
while max_heap:
    # ... process characters
return "".join(result)  # Might return partial result!
```

**Why It Fails:**
When impossible arrangements occur, some characters remain stuck in the cooldown queue. Without checking, you might return a string shorter than the input.

**The Solution:**
```python
# Always validate that all characters were placed
if len(result) != len(s):
    return ""
return "".join(result)
```

### 4. **Heap Value Sign Confusion**

**The Pitfall:**
Forgetting to negate values when working with the max heap simulation:

```python
# WRONG - Forgets to negate when pushing back
if remaining_freq > 0:
    heappush(max_heap, (remaining_freq, cooldown_char))  # Should be negative!

# WRONG - Forgets to convert back to positive
neg_freq, char = heappop(max_heap)
cooldown_queue.append((neg_freq - 1, char))  # Should use positive freq
```

**The Solution:**
Always maintain consistency:
- Negate when pushing to heap: `heappush(max_heap, (-remaining_freq, cooldown_char))`
- Convert to positive after popping: `freq = -neg_freq`

### 5. **Inefficient String Concatenation**

**The Pitfall:**
Using string concatenation instead of a list:

```python
# INEFFICIENT - O(n²) due to string immutability
result = ""
while max_heap:
    result += char  # Creates new string each time
```

**The Solution:**
Use a list and join at the end:
```python
result = []
while max_heap:
    result.append(char)
return "".join(result)  # O(n) operation
```

