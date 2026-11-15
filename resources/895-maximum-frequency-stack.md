# 895. Maximum Frequency Stack

> Source: https://algo.monster/liteproblems/895

## Problem Statement

You need to design a special [stack](/problems/stack_intro) data structure called `FreqStack` that supports two operations:

1. **Push**: Add an element to the [stack](/problems/stack_intro)
2. **Pop**: Remove and return the element that appears most frequently in the stack

The key challenge is that when you pop, you don't just remove any occurrence of the most frequent element - you need to follow specific rules:
- Find the element(s) with the highest frequency count
- If multiple elements share the same highest frequency, remove the one that was pushed most recently (closest to the top)

For example, if you push the sequence `[5, 7, 5, 7, 4, 5]`:
- Element `5` appears 3 times (highest frequency)
- Elements `7` appears 2 times
- Element `4` appears 1 time

When you call `pop()`, it should return `5` because it has the highest frequency. The specific `5` removed would be the last one pushed.

If after that pop you have `[5, 7, 5, 7, 4]` and call `pop()` again:
- Both `5` and `7` now appear 2 times (tied for highest frequency)
- Since `7` was pushed more recently than the remaining `5`, it returns `7`

The implementation uses:
- A hash table `cnt` to track the frequency of each element
- A max heap `q` that stores tuples of `(-frequency, -timestamp, value)` to efficiently find the most frequent and most recent element
- A timestamp counter `ts` to track the order of insertions

The negative values in the heap ensure that higher frequencies and more recent timestamps have priority when popping elements.

## Intuition

When thinking about this problem, we need to handle two key requirements: tracking frequency and maintaining recency when frequencies are tied.

The first thought might be to use a simple frequency counter, but that alone won't help us efficiently find the element with maximum frequency. We'd have to scan through all elements every time we pop, which would be inefficient.

This naturally leads us to consider a heap (priority queue) since it can give us the maximum element in `O(log n)` time. But what should we store in the heap?

We need to prioritize by frequency first, so that should be our primary sorting criterion. However, when frequencies are equal, we need the most recently pushed element. This means we need to track when each element was pushed - hence the need for timestamps.

The clever insight is to combine both frequency and timestamp into a single tuple for the heap. By using `(-frequency, -timestamp, value)`, we create a max heap where:
- Higher frequencies come first (more negative means higher priority in a min heap)
- Among equal frequencies, larger timestamps come first (more recent pushes)

Why don't we remove elements from the heap when their frequency decreases? This would require searching through the heap, which is expensive. Instead, we use a "lazy deletion" approach - we keep all entries in the heap and only check if they're valid when we pop them. The frequency counter `cnt` tells us the current actual frequency of each element.

When we pop, we get the top element from the heap. Since we decrement its frequency in `cnt`, any outdated entries for this element in the heap (with higher frequencies) will naturally be ignored in future operations because we always use the most recent frequency when pushing new entries.

This approach elegantly handles both requirements with `O(log n)` complexity for both push and pop operations.

**Learn more about [Stack](/problems/stack_intro) patterns.**

## Solution Approach

The solution uses a hash table combined with a priority queue (max heap) to efficiently track and retrieve the most frequent elements.

### Data Structures Used:

1. **Hash Table `cnt`**: A `defaultdict(int)` that maintains the current frequency count for each element in the [stack](/problems/stack_intro).

2. **Priority Queue `q`**: A max heap implemented using Python's `heapq` (which is a min heap by default, so we use negative values). Each entry is a tuple `(-frequency, -timestamp, value)`.

3. **Timestamp Counter `ts`**: An integer that increments with each push operation to track the order of insertions.

### Implementation Details:

**Initialization (`__init__`)**:
- Initialize `cnt` as an empty frequency counter
- Initialize `q` as an empty list (will be used as a heap)
- Set `ts = 0` as the starting timestamp

**Push Operation (`push`)**:
1. Increment the timestamp: `self.ts += 1`
2. Update the frequency count: `self.cnt[val] += 1`
3. Add a new entry to the heap: `heappush(self.q, (-self.cnt[val], -self.ts, val))`
   - The negative frequency ensures higher frequencies have priority
   - The negative timestamp ensures more recent elements have priority when frequencies are equal
   - The actual value is stored as the third element

**Pop Operation (`pop`)**:
1. Extract the top element from the heap: `val = heappop(self.q)[2]`
   - This gives us the element with the highest frequency
   - If frequencies are tied, it gives us the most recently pushed one
2. Decrement the frequency count: `self.cnt[val] -= 1`
3. Return the popped value: `return val`

### Key Insights:

- **No Stale Entries**: Each heap entry stands for one copy of a value that is still in the stack. If `5` was pushed three times, the heap holds `(-1, ..., 5)`, `(-2, ..., 5)`, and `(-3, ..., 5)`. Popping removes the `(-3, ..., 5)` entry and lowers `cnt[5]` to 2, and the remaining entry with frequency 2 now describes the current top copy of `5` exactly. Because pops always remove the highest-frequency copy of a value first, the heap never needs cleanup.

- **Time Complexity**: 
  - Push: `O(log n)` where n is the number of entries in the heap
  - Pop: `O(log n)` for the heap operation

- **Space Complexity**: `O(n)` where n is the number of elements currently in the stack (at most the total number of pushes), since the heap holds one entry per element.

## Example Walkthrough

Let's walk through a concrete example with the sequence of operations:
- `push(4)`, `push(5)`, `push(4)`, `push(6)`, `push(5)`, `push(4)`, `pop()`, `pop()`

**Initial State:**
- `cnt = {}` (empty frequency counter)
- `q = []` (empty heap)
- `ts = 0` (timestamp counter)

**Step 1: `push(4)`**
- Increment timestamp: `ts = 1`
- Update frequency: `cnt[4] = 1`
- Add to heap: `heappush(q, (-1, -1, 4))`
- State: `cnt = {4: 1}`, `q = [(-1, -1, 4)]`

**Step 2: `push(5)`**
- Increment timestamp: `ts = 2`
- Update frequency: `cnt[5] = 1`
- Add to heap: `heappush(q, (-1, -2, 5))`
- State: `cnt = {4: 1, 5: 1}`, `q = [(-1, -2, 5), (-1, -1, 4)]`
  - Note: 5 has higher priority due to more recent timestamp

**Step 3: `push(4)`**
- Increment timestamp: `ts = 3`
- Update frequency: `cnt[4] = 2`
- Add to heap: `heappush(q, (-2, -3, 4))`
- State: `cnt = {4: 2, 5: 1}`, `q = [(-2, -3, 4), (-1, -1, 4), (-1, -2, 5)]`
  - The heap now has 4 at the top (frequency 2 beats frequency 1)

**Step 4: `push(6)`**
- Increment timestamp: `ts = 4`
- Update frequency: `cnt[6] = 1`
- Add to heap: `heappush(q, (-1, -4, 6))`
- State: `cnt = {4: 2, 5: 1, 6: 1}`, heap has 4 with frequency 2 at top

**Step 5: `push(5)`**
- Increment timestamp: `ts = 5`
- Update frequency: `cnt[5] = 2`
- Add to heap: `heappush(q, (-2, -5, 5))`
- State: `cnt = {4: 2, 5: 2, 6: 1}`, heap has 5 with (-2, -5) at top
  - Both 4 and 5 have frequency 2, but 5 was pushed more recently

**Step 6: `push(4)`**
- Increment timestamp: `ts = 6`
- Update frequency: `cnt[4] = 3`
- Add to heap: `heappush(q, (-3, -6, 4))`
- State: `cnt = {4: 3, 5: 2, 6: 1}`, heap has 4 with frequency 3 at top

**Step 7: `pop()`**
- Extract from heap: `heappop(q)` returns `(-3, -6, 4)`, so `val = 4`
- Decrement frequency: `cnt[4] = 2`
- Return: `4`
- State: `cnt = {4: 2, 5: 2, 6: 1}`
  - Note: The entries `(-2, -3, 4)` and `(-1, -1, 4)` remain in the heap; they stand for the two copies of 4 that are still in the stack

**Step 8: `pop()`**
- Extract from heap: Top is `(-2, -5, 5)` (5 with frequency 2, timestamp 5)
  - This beats `(-2, -3, 4)` (4 with frequency 2, timestamp 3) because 5 is more recent
- Pop returns `val = 5`
- Decrement frequency: `cnt[5] = 1`
- Return: `5`

The heap holds exactly one entry per element still in the stack, and the combination of frequency and timestamp ensures each pop returns the correct element.

## Python Solution

```python
from collections import defaultdict
from heapq import heappush, heappop
from typing import Optional


class FreqStack:
    """
    A stack-like data structure that pops the most frequently occurring element.
    When multiple elements have the same frequency, the most recently pushed element is popped.
    """
  
    def __init__(self):
        """Initialize the frequency stack with necessary data structures."""
        # Dictionary to track the frequency count of each element
        self.frequency_count = defaultdict(int)
      
        # Min heap (priority queue) to maintain elements by frequency and recency
        # Stored as tuples: (-frequency, -timestamp, value)
        # Negative values used to simulate max heap behavior
        self.priority_queue = []
      
        # Timestamp counter to track insertion order
        self.timestamp = 0

    def push(self, val: int) -> None:
        """
        Push an element onto the frequency stack.
      
        Args:
            val: The integer value to push onto the stack
        """
        # Increment timestamp for this operation
        self.timestamp += 1
      
        # Update frequency count for this value
        self.frequency_count[val] += 1
      
        # Add to priority queue with negative frequency and timestamp for max heap behavior
        # Priority order: highest frequency first, then most recent timestamp
        heappush(self.priority_queue, 
                (-self.frequency_count[val], -self.timestamp, val))

    def pop(self) -> int:
        """
        Pop and return the most frequent element from the stack.
        If there's a tie in frequency, return the most recently pushed element.
      
        Returns:
            The popped integer value
        """
        # Extract the value with highest priority (most frequent, most recent)
        _, _, value = heappop(self.priority_queue)
      
        # Decrease the frequency count for this value
        self.frequency_count[value] -= 1
      
        return value


# Your FreqStack object will be instantiated and called as such:
# obj = FreqStack()
# obj.push(val)
# param_2 = obj.pop()
```

## Time and Space Complexity

**Time Complexity:**
- `__init__()`: `O(1)` - Initializing empty data structures takes constant time.
- `push(val)`: `O(log n)` - Incrementing the counter and timestamp are `O(1)` operations, but inserting into the heap using `heappush` takes `O(log n)` time where `n` is the number of elements in the heap.
- `pop()`: `O(log n)` - Removing the top element from the heap using `heappop` takes `O(log n)` time, and decrementing the counter is `O(1)`.

**Space Complexity:**
- `O(n)` where `n` is the total number of push operations performed.
- The heap `q` stores one entry for each push operation, never removing entries until they are popped, so it can grow up to size `n`.
- The counter dictionary `cnt` stores at most `O(m)` entries where `m` is the number of unique values pushed, which is bounded by `n`.
- The timestamp variable `ts` uses `O(1)` space.
- Overall space complexity is `O(n)` as the heap dominates the space usage.

**Note:** The heap stores tuples of `(-frequency, -timestamp, value)` using negative values to achieve max-heap behavior since Python's `heapq` implements a min-heap. This ensures elements with higher frequency are popped first, and among elements with the same frequency, the most recently pushed element (larger timestamp) is popped first.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Breaking Frequency Ties by Value Instead of Recency

**The Problem:**
The heap key must break ties between equal frequencies by push order. A version that stores `(-frequency, -value)` and drops the timestamp picks the larger value instead of the most recent one.

```python
# WRONG: ties are broken by value, not by push order
heappush(self.priority_queue, (-self.frequency_count[val], -val))
```

With `push(2)`, `push(1)`, `pop()`, both values have frequency 1. The correct answer is `1` (pushed last), but this version returns `2`.

**Solution:**
Store a timestamp that increases with every push and use `-timestamp` as the second key, as the solution does: `(-frequency, -timestamp, val)`.

### Pitfall 2: Assuming the Heap Fills Up with Stale Entries

**The Problem:**
It is easy to assume that after a pop, the older entries of the same value (with lower frequencies) are outdated and must be cleaned up or validated. They are not. Push `5` three times: the heap holds entries with frequencies 1, 2, and 3. A pop removes the frequency-3 entry and sets `cnt[5] = 2`, and the remaining frequency-2 entry describes the current top copy of `5` exactly. Each heap entry always matches one element that is still in the stack, so the heap size equals the stack size and no cleanup step is needed.

Extra code such as rebuilding the heap with one entry per value is actually harmful: it drops the entries for the lower copies of each value, so later pops return the wrong elements.

### Pitfall 3: Using a Heap When an O(1) Design Exists

**The Problem:**
The heap gives `O(log n)` per operation. A design based on one stack per frequency level achieves `O(1)` per operation:

```python
class FreqStack:
    def __init__(self):
        self.freq = {}  # element -> frequency
        self.group = defaultdict(list)  # frequency -> stack of elements
        self.max_freq = 0
  
    def push(self, val: int) -> None:
        freq = self.freq.get(val, 0) + 1
        self.freq[val] = freq
        self.group[freq].append(val)
        self.max_freq = max(self.max_freq, freq)
  
    def pop(self) -> int:
        val = self.group[self.max_freq].pop()
        self.freq[val] -= 1
        if not self.group[self.max_freq]:
            self.max_freq -= 1
        return val
```

A value with frequency `f` appears once in each of the stacks `group[1]` through `group[f]`, so popping from `group[max_freq]` always returns the most recent element among those with the highest frequency.

