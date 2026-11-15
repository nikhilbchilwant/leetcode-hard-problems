# 2935. Maximum Strong Pair XOR II

> Source: https://algo.monster/liteproblems/2935

## Problem Statement

You are given a 0-indexed integer array `nums`. Your task is to find the maximum XOR value among all **strong pairs** in the array.

A pair of integers `(x, y)` is called a **strong pair** if it satisfies the condition:
- `|x - y| <= min(x, y)`

This means the absolute difference between `x` and `y` should not exceed the smaller value of the two numbers.

You need to:
1. Select two integers from `nums` that form a strong pair
2. Calculate their bitwise XOR
3. Return the maximum XOR value possible among all valid strong pairs

**Important Notes:**
- You can pick the same integer twice to form a pair (e.g., `(nums[i], nums[i])` is valid)
- The indices of the elements don't matter - you're looking at the values themselves

**Example of strong pairs:**
- `(2, 3)` is a strong pair because `|2 - 3| = 1 <= min(2, 3) = 2`
- `(5, 10)` is a strong pair because `|5 - 10| = 5 <= min(5, 10) = 5`
- `(1, 5)` is NOT a strong pair because `|1 - 5| = 4 > min(1, 5) = 1`

## Intuition

Let's think about the strong pair condition `|x - y| <= min(x, y)`. The absolute value and minimum make this condition a bit complex to work with directly. 

A key insight is that we can simplify this by ordering the pairs. Without loss of generality, let's assume `x <= y`. Then:
- `|x - y| = y - x` (since `y >= x`)
- `min(x, y) = x` (since `x <= y`)

So the condition becomes: `y - x <= x`, which simplifies to `y <= 2x`.

This is much cleaner! For any value `y`, we need to find all values `x` where `x <= y` and `y <= 2x`. Rearranging the second inequality: `x >= y/2`.

So for each `y`, valid `x` values must satisfy: `y/2 <= x <= y`.

Now, how do we efficiently find the maximum XOR? If we sort the array and process elements as `y` from smallest to largest, we can maintain a "window" of valid `x` candidates. As we move to a larger `y`:
- We add `y` itself as a potential `x` candidate (since `x` can equal `y`)
- We remove any `x` values that are now too small (where `y > 2x`)

This [sliding window](/problems/sliding_window_maximum) approach ensures we always have exactly the valid candidates for the current `y`.

For finding maximum XOR efficiently, a binary [trie](/problems/trie_intro) is perfect. It allows us to:
- Insert numbers in `O(log max_value)` time
- Remove numbers in `O(log max_value)` time  
- Find the maximum XOR with a given number in `O(log max_value)` time

The [trie](/problems/trie_intro) works by storing numbers in binary representation. When searching for maximum XOR with `y`, we greedily try to take the opposite bit at each level (if `y` has 0, we try to go to 1, and vice versa) to maximize the XOR result.

By combining the sorted array with [sliding window](/problems/sliding_window_maximum) and binary [trie](/problems/trie_intro), we get an efficient solution that examines each element once as `y` and maintains only valid `x` candidates in the trie.

**Learn more about [Trie](/problems/trie_intro) and [Sliding Window](/problems/sliding_window_maximum) patterns.**

## Solution Approach

The solution implements the sorted array + [sliding window](/problems/sliding_window_maximum) + binary [trie](/problems/trie_intro) approach:

### 1. Binary [Trie](/problems/trie_intro) Implementation

The `[Trie](/problems/trie_intro)` class maintains a binary representation of numbers with three key operations:

**Insert Operation:**
- Inserts a number by traversing from the most significant bit (bit 20) to the least significant bit (bit 0)
- For each bit position `i`, extracts the bit value using `v = x >> i & 1`
- Creates new [trie](/problems/trie_intro) nodes as needed and increments the count at each node
- The `cnt` field tracks how many numbers pass through each node

**Search Operation:**
- Given a number `x`, finds the maximum XOR value possible with numbers in the [trie](/problems/trie_intro)
- At each bit position, tries to take the opposite bit (`v ^ 1`) to maximize XOR
- If the opposite bit path exists and has numbers (`node.children[v ^ 1].cnt > 0`), takes it and sets the corresponding bit in the answer
- Otherwise, follows the same bit path
- Returns the maximum XOR value achievable

**Remove Operation:**
- Removes a number by traversing its bit path
- Decrements the `cnt` at each node along the path
- This effectively marks the number as removed without actually deleting nodes

### 2. Main Algorithm

```python
def maximumStrongPairXor(self, nums: List[int]) -> int:
    nums.sort()  # Sort array to enable x <= y assumption
    tree = [Trie](/problems/trie_intro)()
    ans = i = 0  # ans stores max XOR, i is left pointer of window
  
    for y in nums:  # Enumerate each element as y
        tree.insert(y)  # Add y to valid candidates
      
        # Remove elements that violate y <= 2x (i.e., y > 2*nums[i])
        while y > nums[i] * 2:
            tree.remove(nums[i])
            i += 1
      
        # Find max XOR between y and valid candidates in trie
        ans = max(ans, tree.search(y))
  
    return ans
```

**Key Steps:**

1. **Sort the array**: This ensures we can process elements in order and maintain the `x <= y` relationship

2. **[Sliding window](/problems/sliding_window_maximum) maintenance**: For each `y`:
   - First insert `y` into the [trie](/problems/trie_intro) (it can pair with itself or future elements)
   - Remove elements from the left that no longer satisfy `y <= 2x`
   - The window `[nums[i], ..., y]` contains all valid `x` values for current `y`

3. **XOR maximization**: Use the [trie](/problems/trie_intro)'s `search` method to find the maximum XOR between `y` and any valid `x` in the window

**Time Complexity**: `O(n log n + n * log M)` where `n` is the array length and `M` is the maximum value (for bit operations)
- Sorting: `O(n log n)`
- For each element: `O(log M)` for insert, remove, and search operations

**Space Complexity**: `O(n * log M)` for the [trie](/problems/trie_intro) structure storing at most `n` numbers with `log M` bits each

## Example Walkthrough

Let's trace through the algorithm with `nums = [1, 2, 3, 4, 5]`.

**Step 1: Sort the array**
Array is already sorted: `[1, 2, 3, 4, 5]`

**Step 2: Initialize**
- `tree = empty Trie`
- `ans = 0` (maximum XOR found so far)
- `i = 0` (left pointer of sliding window)

**Step 3: Process each element as y**

**When y = 1 (index 0):**
- Insert 1 into trie: `tree = {1}`
- Check window: Is `1 > 2 * nums[0]`? Is `1 > 2 * 1 = 2`? No, so keep nums[0]
- Valid x candidates in trie: `{1}`
- Calculate XOR: `1 ⊕ 1 = 0`
- Update ans: `max(0, 0) = 0`

**When y = 2 (index 1):**
- Insert 2 into trie: `tree = {1, 2}`
- Check window: Is `2 > 2 * nums[0]`? Is `2 > 2 * 1 = 2`? No, so keep nums[0]
- Valid x candidates in trie: `{1, 2}`
- Calculate XORs: `2 ⊕ 1 = 3` (binary: 10 ⊕ 01 = 11), `2 ⊕ 2 = 0`
- Trie finds maximum: 3
- Update ans: `max(0, 3) = 3`

**When y = 3 (index 2):**
- Insert 3 into trie: `tree = {1, 2, 3}`
- Check window: Is `3 > 2 * nums[0]`? Is `3 > 2 * 1 = 2`? Yes!
  - Remove nums[0] = 1 from trie: `tree = {2, 3}`
  - Increment i to 1
- Valid x candidates in trie: `{2, 3}`
- Calculate XORs: `3 ⊕ 2 = 1`, `3 ⊕ 3 = 0`
- Trie finds maximum: 1
- Update ans: `max(3, 1) = 3`

**When y = 4 (index 3):**
- Insert 4 into trie: `tree = {2, 3, 4}`
- Check window: Is `4 > 2 * nums[1]`? Is `4 > 2 * 2 = 4`? No, so keep nums[1]
- Valid x candidates in trie: `{2, 3, 4}`
- Calculate XORs: `4 ⊕ 2 = 6` (binary: 100 ⊕ 010 = 110), `4 ⊕ 3 = 7` (binary: 100 ⊕ 011 = 111), `4 ⊕ 4 = 0`
- Trie finds maximum: 7
- Update ans: `max(3, 7) = 7`

**When y = 5 (index 4):**
- Insert 5 into trie: `tree = {2, 3, 4, 5}`
- Check window: Is `5 > 2 * nums[1]`? Is `5 > 2 * 2 = 4`? Yes!
  - Remove nums[1] = 2 from trie: `tree = {3, 4, 5}`
  - Increment i to 2
- Check window: Is `5 > 2 * nums[2]`? Is `5 > 2 * 3 = 6`? No, so keep nums[2]
- Valid x candidates in trie: `{3, 4, 5}`
- Calculate XORs: `5 ⊕ 3 = 6`, `5 ⊕ 4 = 1`, `5 ⊕ 5 = 0`
- Trie finds maximum: 6
- Update ans: `max(7, 6) = 7`

**Result:** The maximum XOR among all strong pairs is **7**, achieved by the pair (3, 4).

**Verification:** Is (3, 4) a strong pair?
- `|3 - 4| = 1`
- `min(3, 4) = 3`
- Is `1 ≤ 3`? Yes! ✓

The trie efficiently finds the maximum XOR by trying to set opposite bits at each position. For example, when y = 4 (binary: 100), the trie searches for numbers that would maximize XOR by preferring paths with opposite bits, ultimately finding 3 (binary: 011) which gives XOR = 111 (decimal 7).

## Python Solution

```python
from typing import List, Optional


class Trie:
    """Binary Trie data structure for XOR operations on integers."""
  
    __slots__ = ("children", "count")
  
    def __init__(self):
        # Binary trie: children[0] for bit 0, children[1] for bit 1
        self.children: List[Optional['Trie']] = [None, None]
        # Count of numbers passing through this node
        self.count: int = 0
  
    def insert(self, number: int) -> None:
        """Insert a number into the trie by its binary representation.
      
        Args:
            number: The integer to insert into the trie
        """
        current_node = self
        # Process bits from most significant (bit 20) to least significant (bit 0)
        for bit_position in range(20, -1, -1):
            # Extract the bit at current position
            bit_value = (number >> bit_position) & 1
          
            # Create new node if path doesn't exist
            if current_node.children[bit_value] is None:
                current_node.children[bit_value] = Trie()
          
            # Move to child node and increment count
            current_node = current_node.children[bit_value]
            current_node.count += 1
  
    def search(self, number: int) -> int:
        """Find the maximum XOR value with the given number.
      
        Args:
            number: The integer to find maximum XOR with
          
        Returns:
            Maximum XOR value achievable with stored numbers
        """
        current_node = self
        max_xor = 0
      
        # Process bits from most significant to least significant
        for bit_position in range(20, -1, -1):
            # Extract the bit at current position
            bit_value = (number >> bit_position) & 1
          
            # Try to go opposite direction for maximum XOR
            opposite_bit = bit_value ^ 1
          
            # Check if opposite path exists and has valid numbers
            if current_node.children[opposite_bit] and current_node.children[opposite_bit].count:
                # Set the bit in result and follow opposite path
                max_xor |= (1 << bit_position)
                current_node = current_node.children[opposite_bit]
            else:
                # Follow the same bit path if opposite doesn't exist
                current_node = current_node.children[bit_value]
      
        return max_xor
  
    def remove(self, number: int) -> None:
        """Remove a number from the trie by decrementing counts.
      
        Args:
            number: The integer to remove from the trie
        """
        current_node = self
        # Process bits from most significant to least significant
        for bit_position in range(20, -1, -1):
            # Extract the bit at current position
            bit_value = (number >> bit_position) & 1
          
            # Move to child node and decrement count
            current_node = current_node.children[bit_value]
            current_node.count -= 1


class Solution:
    """Solution for finding maximum strong pair XOR."""
  
    def maximumStrongPairXor(self, nums: List[int]) -> int:
        """Find the maximum XOR of a strong pair in the array.
      
        A strong pair (x, y) satisfies: |x - y| <= min(x, y)
        Which is equivalent to: max(x, y) <= 2 * min(x, y)
      
        Args:
            nums: List of positive integers
          
        Returns:
            Maximum XOR value of any strong pair
        """
        # Sort array to maintain sliding window of valid pairs
        nums.sort()
      
        # Initialize trie and tracking variables
        trie = Trie()
        max_xor_result = 0
        left_pointer = 0
      
        # Process each number as the larger element in potential pairs
        for current_num in nums:
            # Add current number to trie
            trie.insert(current_num)
          
            # Remove numbers that are too small to form strong pairs
            # Strong pair condition: current_num <= 2 * nums[left_pointer]
            while current_num > nums[left_pointer] * 2:
                trie.remove(nums[left_pointer])
                left_pointer += 1
          
            # Find maximum XOR with valid numbers in trie
            max_xor_result = max(max_xor_result, trie.search(current_num))
      
        return max_xor_result
```

## Time and Space Complexity

**Time Complexity:** `O(n × log M)`

The algorithm sorts the array first, which takes `O(n log n)` time. Then for each element in the sorted array:
- **Insert operation:** Traverses from bit position 20 to 0, taking `O(log M)` time where `M = 2^20`
- **Remove operation:** May be called multiple times per iteration, but each element is removed at most once throughout the entire algorithm, contributing `O(n × log M)` total
- **Search operation:** Traverses from bit position 20 to 0, taking `O(log M)` time

Since we iterate through `n` elements and perform operations that take `O(log M)` time each, and considering that `log M = 20` (constant) while `n` can vary, the dominant term is `O(n × log M)`. The sorting time `O(n log n)` is absorbed since `log n ≤ log M` for the given constraints.

**Space Complexity:** `O(n × log M)`

The Trie stores binary representations of numbers:
- Each number requires a path of length `log M = 20` bits from root to leaf
- In the worst case, all `n` numbers are stored in the Trie simultaneously
- Each unique path segment requires a new Trie node
- Maximum space needed is proportional to the number of nodes, which is bounded by `O(n × log M)`

The sliding window approach ensures that at any time, only valid strong pairs are maintained in the Trie, but this doesn't change the worst-case space complexity analysis.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Strong Pair Condition Simplification**

**Pitfall:** A common mistake is incorrectly simplifying the strong pair condition. Developers often assume that for sorted array with `x <= y`, the condition `|x - y| <= min(x, y)` becomes `y - x <= x`, leading to `y <= 2x`. However, they might forget to verify this works for all cases, especially when numbers can be negative or zero.

**Why it happens:** The mathematical transformation seems straightforward, but the condition relies on `x` being positive. With `x <= y`:
- `|x - y| = y - x` (since y ≥ x)
- `min(x, y) = x`
- So the condition becomes: `y - x <= x`, which gives `y <= 2x`

**Issue:** This only works correctly when `x > 0`. If the array contains 0 or negative numbers, the logic breaks down.

**Solution:**
```python
def maximumStrongPairXor(self, nums: List[int]) -> int:
    # Add validation for non-negative numbers
    if any(num < 0 for num in nums):
        raise ValueError("This implementation assumes non-negative numbers")
  
    # For arrays with 0, handle edge case
    nums = [num for num in nums if num > 0] or [0]
  
    nums.sort()
    # ... rest of the implementation
```

### 2. **Bit Range Assumption in Trie**

**Pitfall:** The trie implementation assumes numbers fit within 21 bits (processing from bit 20 to 0). If the input contains numbers larger than 2^21 - 1 (2,097,151), the trie will not correctly represent these numbers, leading to incorrect XOR calculations.

**Why it happens:** The code uses a hardcoded range `range(20, -1, -1)` without considering the actual maximum value in the input array.

**Solution:**
```python
class Trie:
    def __init__(self, max_bits=21):
        self.children = [None, None]
        self.count = 0
        self.max_bits = max_bits
  
    def insert(self, number: int) -> None:
        current_node = self
        # Use dynamic bit range based on initialization
        for bit_position in range(self.max_bits - 1, -1, -1):
            bit_value = (number >> bit_position) & 1
            # ... rest of the logic

class Solution:
    def maximumStrongPairXor(self, nums: List[int]) -> int:
        if not nums:
            return 0
          
        # Calculate required bits based on maximum value
        max_val = max(nums)
        max_bits = max_val.bit_length() if max_val > 0 else 1
      
        nums.sort()
        trie = Trie(max_bits)  # Pass calculated bit count
        # ... rest of the implementation
```

### 3. **Sliding Window Pointer Management**

**Pitfall:** Not properly handling the case when all elements violate the strong pair condition with the current element. The while loop might cause `left_pointer` to exceed array bounds if not careful.

**Why it happens:** When processing a very large number `y`, all previous smaller numbers might need to be removed from the trie, potentially causing the left pointer to go beyond valid indices.

**Solution:**
```python
def maximumStrongPairXor(self, nums: List[int]) -> int:
    nums.sort()
    trie = Trie()
    max_xor_result = 0
    left_pointer = 0
  
    for right_pointer, current_num in enumerate(nums):
        trie.insert(current_num)
      
        # Ensure left_pointer doesn't exceed right_pointer
        while left_pointer <= right_pointer and current_num > nums[left_pointer] * 2:
            trie.remove(nums[left_pointer])
            left_pointer += 1
      
        # Only search if trie has valid elements
        if left_pointer <= right_pointer:
            max_xor_result = max(max_xor_result, trie.search(current_num))
  
    return max_xor_result
```

### 4. **Empty Trie Node Access**

**Pitfall:** The `search` method doesn't handle the case where a child node is `None`, which can cause `AttributeError` when trying to access `current_node.children[bit_value]` after taking the opposite path.

**Why it happens:** After checking the opposite bit path and finding it doesn't exist or has count 0, the code assumes the same bit path exists, but this might not always be true if the trie is in an inconsistent state.

**Solution:**
```python
def search(self, number: int) -> int:
    current_node = self
    max_xor = 0
  
    for bit_position in range(20, -1, -1):
        bit_value = (number >> bit_position) & 1
        opposite_bit = bit_value ^ 1
      
        # Check both paths for validity
        if current_node.children[opposite_bit] and current_node.children[opposite_bit].count > 0:
            max_xor |= (1 << bit_position)
            current_node = current_node.children[opposite_bit]
        elif current_node.children[bit_value] and current_node.children[bit_value].count > 0:
            current_node = current_node.children[bit_value]
        else:
            # Handle case where both paths are invalid
            return max_xor  # Return partial result
  
    return max_xor
```

