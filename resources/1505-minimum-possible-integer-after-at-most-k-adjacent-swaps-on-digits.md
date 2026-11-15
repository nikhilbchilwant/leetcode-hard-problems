# 1505. Minimum Possible Integer After at Most K Adjacent Swaps On Digits

> Source: https://algo.monster/liteproblems/1505

## Problem Statement

You are given a string `num` that represents the digits of a very large integer, and an integer `k` representing the maximum number of swaps you can perform.

In each swap operation, you can exchange any two **adjacent** digits in the number. You can perform at most `k` such swaps.

Your task is to find the minimum possible integer you can create by performing up to `k` adjacent swaps, and return it as a string.

For example:
- If `num = "4321"` and `k = 4`, you could swap adjacent digits to eventually get `"1234"` (the minimum possible)
- If `num = "100"` and `k = 1`, you could swap once to get `"010"` which equals `"10"`

The challenge involves strategically choosing which adjacent digits to swap to minimize the final number, while staying within the constraint of at most `k` swaps. Since you can only swap adjacent digits, moving a digit multiple positions requires multiple swaps.

## Intuition

To minimize the number, we want smaller digits to appear earlier in the result. The key insight is to greedily select the smallest available digit that can be moved to the current position within our remaining swap budget.

Consider building the result digit by digit from left to right. For each position, we want to find the smallest digit that we can afford to move there. Moving a digit from position `j` to position `i` (where `j > i`) requires `j - i` adjacent swaps to bubble it leftward.

However, there's a complication: as we select digits and "remove" them from their original positions, the distances between remaining digits change. If we've already selected some digits between positions `i` and `j`, the actual number of swaps needed is less than `j - i` because there are gaps.

This leads us to think about tracking which positions have been "used" and calculating the actual distance dynamically. The actual distance becomes: `(original position j) - (current position i) - (number of used positions between i and j)`.

We can use a Binary Indexed Tree (Fenwick Tree) to efficiently track and query how many positions have been used in any range. For each digit value (0-9), we maintain a queue of positions where that digit appears. Then, for each position in our result:
1. Try each digit from 0 to 9
2. Check if we can afford to move the first occurrence of that digit to the current position
3. If yes, select it, mark its position as used, and update our remaining swaps
4. The first affordable digit we find (starting from 0) is optimal for this position

The Binary Indexed Tree allows us to query "how many positions are used up to index `x`" in `O(log n)` time, making the distance calculation efficient.

**Learn more about [Greedy](/problems/greedy_intro) and [Segment Tree](/problems/segment_tree_intro) patterns.**

## Solution Approach

The solution uses a Binary Indexed Tree (Fenwick Tree) combined with a [greedy](/problems/greedy_intro) approach to efficiently find the minimum number.

**Data Structures Used:**

1. **Binary Indexed Tree (BIT)**: Tracks which positions have been "used" (digits already selected). It supports:
   - `update(x, delta)`: Mark position `x` as used by adding `delta` (1 in our case)
   - `query(x)`: Get count of used positions from 1 to `x`

2. **Position Queues**: A dictionary `pos` where `pos[d]` is a deque containing all positions where digit `d` appears in the original string (1-indexed).

**Algorithm Steps:**

1. **Initialize Data Structures:**
   ```python
   pos = defaultdict(deque)
   for i, v in enumerate(num, 1):
       pos[int(v)].append(i)
   ```
   Store all positions (1-indexed) where each digit appears.

2. **Build Result Greedily:**
   For each position `i` from 1 to `n`:
   - Try digits 0 through 9 in order
   - For each digit `v`, check if we have any occurrences left in `pos[v]`
   - If yes, get the leftmost occurrence at position `j = pos[v][0]`

3. **Calculate Actual Distance:**
   ```python
   dist = tree.query(n) - tree.query(j) + j - i
   ```
   This formula computes:
   - `tree.query(n) - tree.query(j)`: Number of used positions after `j`
   - `j - i`: Original distance between positions
   - The actual swaps needed = `(j - i) + (used positions after j)`

4. **Make Selection:**
   - If `dist <= k`, we can afford to move this digit:
     - Deduct `k -= dist`
     - Remove this position from `pos[v]` using `popleft()`
     - Add digit `v` to result
     - Mark position `j` as used: `tree.update(j, 1)`
     - Break to move to next position

5. **Return Result:**
   Join all selected digits to form the final minimum number.

**Time Complexity:** `O(n * 10 * log n)` where `n` is the length of the string. For each of `n` positions, we check up to 10 digits, and each BIT operation takes `O(log n)`.

**Space Complexity:** `O(n)` for storing positions and the BIT structure.

## Example Walkthrough

Let's walk through the solution with `num = "4321"` and `k = 4`.

**Initial Setup:**
- Original string: "4321"
- Position queues (1-indexed):
  - pos[1] = [4]
  - pos[2] = [3] 
  - pos[3] = [2]
  - pos[4] = [1]
- BIT initialized with all zeros
- Result = []

**Building Position 1:**
- Try digit 0: No occurrences available, skip
- Try digit 1: pos[1] has [4]
  - Position j = 4 (digit '1' is at position 4)
  - Calculate distance: `tree.query(4) - tree.query(4) + 4 - 1 = 0 - 0 + 3 = 3`
  - Cost is 3 swaps, we have k=4, so we can afford it!
  - Select digit '1', update k = 4 - 3 = 1
  - Remove position 4 from pos[1]: pos[1] = []
  - Mark position 4 as used: tree.update(4, 1)
  - Result = ['1']

**Building Position 2:**
- Try digit 0: No occurrences, skip
- Try digit 1: No occurrences left, skip
- Try digit 2: pos[2] has [3]
  - Position j = 3
  - Calculate distance: `tree.query(4) - tree.query(3) + 3 - 2 = 1 - 0 + 1 = 2`
    - Note: tree.query(4) = 1 because position 4 is marked as used
  - Cost is 2 swaps, but we only have k=1 remaining, cannot afford
- Try digit 3: pos[3] has [2]
  - Position j = 2
  - Calculate distance: `tree.query(4) - tree.query(2) + 2 - 2 = 1 - 0 + 0 = 1`
  - Cost is 1 swap, we have k=1, we can afford it!
  - Select digit '3', update k = 1 - 1 = 0
  - Mark position 2 as used: tree.update(2, 1)
  - Result = ['1', '3']

**Building Position 3:**
- k = 0 (no swaps left)
- Try digit 0: No occurrences, skip
- Try digit 1: No occurrences left, skip
- Try digit 2: pos[2] has [3]
  - Position j = 3
  - Calculate distance: `tree.query(4) - tree.query(3) + 3 - 3 = 1 - 1 + 0 = 0`
    - Note: Both positions 2 and 4 are marked as used
  - Cost is 0 swaps, we can afford it!
  - Select digit '2'
  - Result = ['1', '3', '2']

**Building Position 4:**
- Only digit '4' remains at position 1
- Distance = 0 (it's the only one left)
- Result = ['1', '3', '2', '4']

**Final Answer:** "1324"

This demonstrates how the algorithm greedily selects the smallest available digit that can be moved within the swap budget, using the BIT to track which positions have been "consumed" to accurately calculate movement costs.

## Python Solution

```python
from collections import defaultdict, deque
from typing import List


class BinaryIndexedTree:
    """
    Binary Indexed Tree (Fenwick Tree) for efficient range sum queries and updates.
    Used to track which positions have been used/moved.
    """
  
    def __init__(self, n: int) -> None:
        """
        Initialize a Binary Indexed Tree with size n.
      
        Args:
            n: Size of the tree (1-indexed)
        """
        self.size = n
        self.tree = [0] * (n + 1)  # 1-indexed array
  
    @staticmethod
    def lowbit(x: int) -> int:
        """
        Get the lowest set bit of x.
        This helps navigate the tree structure.
      
        Args:
            x: Input number
          
        Returns:
            The value of the lowest set bit
        """
        return x & -x
  
    def update(self, x: int, delta: int) -> None:
        """
        Add delta to position x and update all affected parent nodes.
      
        Args:
            x: Position to update (1-indexed)
            delta: Value to add
        """
        while x <= self.size:
            self.tree[x] += delta
            x += BinaryIndexedTree.lowbit(x)
  
    def query(self, x: int) -> int:
        """
        Get the prefix sum from position 1 to x.
      
        Args:
            x: End position for the prefix sum (1-indexed)
          
        Returns:
            Sum of elements from 1 to x
        """
        result = 0
        while x > 0:
            result += self.tree[x]
            x -= BinaryIndexedTree.lowbit(x)
        return result


class Solution:
    def minInteger(self, num: str, k: int) -> str:
        """
        Find the smallest possible integer by swapping adjacent digits at most k times.
      
        Args:
            num: Input number as string
            k: Maximum number of adjacent swaps allowed
          
        Returns:
            The lexicographically smallest string after at most k swaps
        """
        # Store positions of each digit (0-9) in the original string
        # Using 1-indexed positions for the Binary Indexed Tree
        digit_positions = defaultdict(deque)
        for index, digit_char in enumerate(num, 1):
            digit_value = int(digit_char)
            digit_positions[digit_value].append(index)
      
        result = []
        n = len(num)
      
        # Binary Indexed Tree to track which positions have been used
        # When a position is used, we mark it with 1
        used_positions_tree = BinaryIndexedTree(n)
      
        # Process each position in the result
        for current_position in range(1, n + 1):
            # Try digits from 0 to 9 to get the smallest possible digit
            for digit in range(10):
                positions_queue = digit_positions[digit]
              
                if positions_queue:
                    # Get the leftmost occurrence of this digit
                    original_position = positions_queue[0]
                  
                    # Calculate the actual distance to move this digit to current position
                    # Distance = original_position - current_position - (number of used positions before original_position)
                    used_before = used_positions_tree.query(original_position)
                    distance = original_position - current_position - used_before
                  
                    # Alternative calculation using the formula in original code:
                    # total_used = used_positions_tree.query(n)
                    # distance = total_used - used_before + original_position - current_position
                  
                    if distance <= k:
                        # We can afford to move this digit to the current position
                        k -= distance
                        positions_queue.popleft()
                        result.append(str(digit))
                      
                        # Mark this position as used
                        used_positions_tree.update(original_position, 1)
                        break
      
        return ''.join(result)
```

## Time and Space Complexity

**Time Complexity:** `O(n² log n)`

The algorithm consists of:
- Building position queues: `O(n)` where we iterate through the string once
- Main loop with nested operations:
  - Outer loop runs `n` times (for each position in result)
  - Inner loop checks 10 digits (constant)
  - For each valid digit:
    - `tree.query(n)` and `tree.query(j)`: Each query operation takes `O(log n)` due to the BIT structure
    - `tree.update(j, 1)`: Update operation takes `O(log n)`
  - In worst case, we perform BIT operations for each position

Since we have `n` iterations and each iteration potentially performs `O(log n)` BIT operations, the total time complexity is `O(n² log n)`.

The `n²` factor comes from the fact that for each of the `n` positions, we might need to check positions that require calculating distances using the BIT, and in worst case scenarios, we examine many positions before finding one within the allowed swaps `k`.

**Space Complexity:** `O(n)`

The space usage includes:
- `pos` dictionary with deques storing all positions: `O(n)` total across all digits
- `ans` list storing the result: `O(n)`
- BinaryIndexedTree with array `c` of size `n+1`: `O(n)`
- Other variables: `O(1)`

The total space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Distance Calculation Formula**

**Pitfall:** Many implementations incorrectly calculate the number of swaps needed to move a digit from position `j` to position `i`. A common mistake is using simple arithmetic like `j - i` without accounting for digits that have already been moved.

**Why it happens:** When we select and move digits earlier in the process, they effectively "disappear" from their original positions. This changes the actual distance between remaining digits.

**Example:**
- Original string: `"4321"`, k = 4
- After selecting '1' from position 4 and moving it to position 1 (3 swaps)
- The remaining digits are conceptually at positions 1, 2, 3 (not 2, 3, 4)
- If we now want to move '2' to position 2, it only needs 1 swap, not 2

**Solution:** Use the Binary Indexed Tree to track used positions and adjust the distance calculation:
```python
# Correct formula
used_after_j = tree.query(n) - tree.query(j)
actual_distance = j - i - (tree.query(j - 1))
# OR equivalently:
actual_distance = tree.query(n) - tree.query(j) + j - i
```

### 2. **Using 0-Indexed Positions with Binary Indexed Tree**

**Pitfall:** Binary Indexed Trees traditionally use 1-based indexing. Using 0-based indexing can cause index out of bounds errors or incorrect calculations.

**Why it happens:** The BIT's `lowbit` operation and tree navigation logic rely on 1-based indexing. Position 0 would cause infinite loops in queries/updates.

**Solution:** Always use 1-based indexing when working with BIT:
```python
# Correct: enumerate starting from 1
for i, v in enumerate(num, 1):
    pos[int(v)].append(i)

# Incorrect: using 0-based indexing
for i, v in enumerate(num):  # Don't do this
    pos[int(v)].append(i)
```

### 3. **Not Breaking After Finding a Valid Digit**

**Pitfall:** Forgetting to `break` after successfully placing a digit, causing the algorithm to try placing multiple digits at the same position.

**Solution:** Always include a `break` statement after successfully placing a digit:
```python
if dist <= k:
    k -= dist
    pos[v].popleft()
    ans.append(str(v))
    tree.update(j, 1)
    break  # Critical: move to next position
```

### 4. **Modifying the Deque While Iterating**

**Pitfall:** Some might try to iterate through all positions of a digit and modify the deque simultaneously, leading to unexpected behavior.

**Solution:** Always check only the first element (`pos[v][0]`) and use `popleft()` to remove it:
```python
# Correct approach
if pos[v]:  # Check if deque is not empty
    j = pos[v][0]  # Peek at first element
    # ... calculate distance ...
    if dist <= k:
        pos[v].popleft()  # Remove only after confirming we'll use it
```

### 5. **Integer vs String Confusion**

**Pitfall:** Mixing up integer values and string characters when building the result, leading to type errors or incorrect output format.

**Solution:** Be consistent with type conversions:
```python
# Store positions by integer value
pos[int(v)].append(i)  # Convert char to int for indexing

# But append string to result
ans.append(str(v))  # Convert back to string for output
```

