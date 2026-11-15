# 1803. Count Pairs With XOR in a Range

> Source: https://algo.monster/liteproblems/1803

## Problem Statement

You are given a 0-indexed integer array `nums` and two integers `low` and `high`. Your task is to find the number of **nice pairs** in the array.

A **nice pair** is defined as a pair of indices `(i, j)` where:
- `0 <= i < j < nums.length` (i.e., `i` comes before `j` in the array)
- The XOR operation between `nums[i]` and `nums[j]` produces a result that falls within the range `[low, high]`, inclusive
- In other words: `low <= (nums[i] XOR nums[j]) <= high`

The XOR operation is a bitwise operation where each bit in the result is `1` if the corresponding bits in the two operands are different, and `0` if they are the same.

For example, if `nums = [1, 4, 2, 7]`, `low = 2`, and `high = 6`:
- Pair `(0, 1)`: `1 XOR 4 = 5`, which is in range `[2, 6]` ✓
- Pair `(0, 2)`: `1 XOR 2 = 3`, which is in range `[2, 6]` ✓
- Pair `(0, 3)`: `1 XOR 7 = 6`, which is in range `[2, 6]` ✓
- Pair `(1, 2)`: `4 XOR 2 = 6`, which is in range `[2, 6]` ✓
- Pair `(1, 3)`: `4 XOR 7 = 3`, which is in range `[2, 6]` ✓
- Pair `(2, 3)`: `2 XOR 7 = 5`, which is in range `[2, 6]` ✓

All 6 pairs satisfy the condition, so the answer would be 6.

The challenge is to efficiently count all such pairs without checking every possible combination, especially when dealing with large arrays.

## Intuition

When we need to count pairs whose XOR values fall within a range `[low, high]`, we can transform this into a simpler problem. Instead of directly counting pairs in a range, we can use the principle: count of pairs in `[low, high]` = count of pairs in `[0, high]` - count of pairs in `[0, low-1]`.

This transformation is powerful because counting pairs with XOR less than a certain value is easier to handle systematically than counting within a range.

For XOR operations on arrays, a [Trie](/problems/trie_intro) (prefix tree) data structure is particularly effective. Why? Because XOR operates on bits, and a Trie can efficiently represent binary numbers bit by bit. We can build a binary Trie (0-1 Trie) where each path from root to leaf represents a number in binary form.

The key insight is that as we traverse the array, for each number we encounter, we want to find how many previously seen numbers would produce an XOR result less than our limit. By storing numbers in the [Trie](/problems/trie_intro) as we go, we can efficiently query this information.

When searching for pairs with XOR less than a limit, we traverse the [Trie](/problems/trie_intro) bit by bit from the most significant bit. At each bit position:
- If the limit's current bit is `1`, we have flexibility. We can take all numbers that would give us a `0` at this position (making the XOR result definitely smaller), and then continue exploring the path that gives us a `1`.
- If the limit's current bit is `0`, we must follow the path that keeps our XOR result's bit as `0` to stay under the limit.

By processing numbers one by one and querying the [Trie](/problems/trie_intro) before inserting each number, we ensure we only count each pair once (since we only look at previously inserted numbers). This approach gives us an efficient `O(n * log(max_value))` solution where `n` is the array length and `log(max_value)` represents the number of bits we need to check.

**Learn more about [Trie](/problems/trie_intro) patterns.**

## Solution Approach

The solution uses a **0-1 [Trie](/problems/trie_intro)** (binary Trie) to efficiently count pairs with XOR values in the range `[low, high]`. Let's walk through the implementation step by step.

### [Trie](/problems/trie_intro) Node Structure

The [Trie](/problems/trie_intro) node contains:
- `children[0]` and `children[1]`: pointers to left (bit 0) and right (bit 1) child nodes
- `cnt`: counter tracking how many numbers pass through or end at this node

### Insert Function

The `insert(x)` function adds a number to the [Trie](/problems/trie_intro):
1. Start from the root node
2. Process bits from most significant (bit 15) to least significant (bit 0)
3. For each bit position `i`:
   - Extract the bit value: `v = x >> i & 1`
   - If the child node for this bit doesn't exist, create it
   - Move to that child node and increment its counter

### Search Function

The `search(x, limit)` function counts how many numbers in the [Trie](/problems/trie_intro) have XOR with `x` less than `limit`:
1. Start from the root with `ans = 0`
2. For each bit position from 15 to 0:
   - Get current bit of `x`: `v = x >> i & 1`
   - Check the current bit of `limit`: `limit >> i & 1`
 
   **If limit's bit is 1:**
   - We can take all numbers that would make XOR bit = 0 (same as `x`'s bit)
   - Add `node.children[v].cnt` to answer (all these paths give XOR < limit)
   - Continue exploring the path where XOR bit = 1: `node = node.children[v ^ 1]`
 
   **If limit's bit is 0:**
   - We must follow the path where XOR bit = 0 to stay under limit
   - Move to: `node = node.children[v]` (same bit as `x`)

### Main Algorithm

The `countPairs` function orchestrates the solution:
1. Initialize answer counter and empty [Trie](/problems/trie_intro)
2. For each number `x` in `nums`:
   - Calculate pairs with XOR in `[0, high]`: `tree.search(x, high + 1)`
   - Subtract pairs with XOR in `[0, low-1]`: `tree.search(x, low)`
   - The difference gives pairs with XOR in `[low, high]`
   - Insert `x` into the Trie for future queries
3. Return the total count

### Why This Works

By processing numbers sequentially and querying before inserting, we ensure:
- Each pair `(i, j)` is counted exactly once when processing `nums[j]`
- The [Trie](/problems/trie_intro) contains all numbers with indices less than current index
- The bit-by-bit comparison efficiently filters numbers based on XOR constraints

The time complexity is `O(n * 16)` where `n` is the array length and 16 is the number of bits processed (for numbers up to 2^16). The space complexity is `O(n * 16)` for storing the [Trie](/problems/trie_intro).

## Example Walkthrough

Let's walk through a small example with `nums = [3, 10, 5, 25]`, `low = 5`, and `high = 15`.

**Step 1: Process nums[0] = 3 (binary: 0011)**
- Query: How many numbers already in Trie have XOR with 3 in range [5, 15]?
- Trie is empty, so count = 0
- Insert 3 into Trie

**Step 2: Process nums[1] = 10 (binary: 1010)**
- Query: How many numbers in Trie have XOR with 10 in range [5, 15]?
- We need to check: 3 XOR 10 = 9 (binary: 1001)
- Is 9 in [5, 15]? Yes! Count = 1
- Insert 10 into Trie

Let's trace the search process for `search(10, 16)` - `search(10, 5)`:
- For `search(10, 16)` with limit = 16 (binary: 10000):
  - Starting from MSB, we traverse the Trie comparing bits
  - At each level, we check if we can count paths or need to continue
  - Result: 1 (the path representing 3)
- For `search(10, 5)` with limit = 5 (binary: 00101):
  - Similar traversal but with stricter limit
  - Result: 0 (3 XOR 10 = 9, which is ≥ 5)
- Difference: 1 - 0 = 1 valid pair

**Step 3: Process nums[2] = 5 (binary: 0101)**
- Query: How many numbers in Trie have XOR with 5 in range [5, 15]?
- Check: 3 XOR 5 = 6 (in range ✓), 10 XOR 5 = 15 (in range ✓)
- Count = 2
- Insert 5 into Trie

**Step 4: Process nums[3] = 25 (binary: 11001)**
- Query: How many numbers in Trie have XOR with 25 in range [5, 15]?
- Check: 3 XOR 25 = 26 (out of range ✗), 10 XOR 25 = 19 (out of range ✗), 5 XOR 25 = 28 (out of range ✗)
- Count = 0
- Insert 25 into Trie

**Final Answer: 0 + 1 + 2 + 0 = 3 nice pairs**

The pairs are: (0,1) with XOR=9, (0,2) with XOR=6, and (1,2) with XOR=15.

## Python Solution

```python
from typing import List


class Trie:
    """Binary trie node for storing numbers in binary representation."""
  
    def __init__(self):
        # children[0] represents bit 0, children[1] represents bit 1
        self.children = [None] * 2
        # Count of numbers passing through this node
        self.cnt = 0
  
    def insert(self, x: int) -> None:
        """Insert a number into the trie by its binary representation.
      
        Args:
            x: The number to insert into the trie.
        """
        node = self
        # Process bits from most significant (bit 15) to least significant (bit 0)
        for i in range(15, -1, -1):
            # Extract the i-th bit of x
            bit = (x >> i) & 1
          
            # Create new node if path doesn't exist
            if node.children[bit] is None:
                node.children[bit] = Trie()
          
            # Move to the child node
            node = node.children[bit]
            # Increment count for this path
            node.cnt += 1
  
    def search(self, x: int, limit: int) -> int:
        """Count numbers in trie where XOR with x is less than limit.
      
        Args:
            x: The number to XOR with existing numbers in trie.
            limit: The upper bound (exclusive) for XOR result.
          
        Returns:
            Count of numbers where (number XOR x) < limit.
        """
        node = self
        count = 0
      
        # Process bits from most significant to least significant
        for i in range(15, -1, -1):
            if node is None:
                return count
          
            # Extract the i-th bit of x and limit
            x_bit = (x >> i) & 1
            limit_bit = (limit >> i) & 1
          
            if limit_bit == 1:
                # If limit bit is 1, we can take all numbers with same bit as x
                # (which would give XOR bit = 0, making result smaller)
                if node.children[x_bit] is not None:
                    count += node.children[x_bit].cnt
              
                # Continue searching with opposite bit (XOR bit = 1)
                node = node.children[x_bit ^ 1]
            else:
                # If limit bit is 0, we must take same bit to keep XOR result small
                node = node.children[x_bit]
      
        return count


class Solution:
    def countPairs(self, nums: List[int], low: int, high: int) -> int:
        """Count pairs (i, j) where i < j and low <= nums[i] XOR nums[j] <= high.
      
        Args:
            nums: List of integers.
            low: Lower bound (inclusive) for XOR value.
            high: Upper bound (inclusive) for XOR value.
          
        Returns:
            Number of valid pairs.
        """
        pair_count = 0
        trie = Trie()
      
        # For each number, count valid pairs with previously seen numbers
        for num in nums:
            # Count pairs where XOR is in range [low, high]
            # This equals: (pairs with XOR < high+1) - (pairs with XOR < low)
            pairs_below_high = trie.search(num, high + 1)
            pairs_below_low = trie.search(num, low)
            pair_count += pairs_below_high - pairs_below_low
          
            # Insert current number for future pairs
            trie.insert(num)
      
        return pair_count
```

## Time and Space Complexity

**Time Complexity:** `O(n × log M)`

The algorithm iterates through each element in the `nums` array once. For each element `x`:
- It performs two `search` operations: `tree.search(x, high + 1)` and `tree.search(x, low)`. Each search traverses the Trie from root to leaf, examining exactly 16 bits (since we iterate from bit 15 down to bit 0), taking `O(log M)` time where `log M = 16`.
- It performs one `insert` operation: `tree.insert(x)`. The insert operation also traverses 16 bits to insert the number into the Trie, taking `O(log M)` time.

Since we process `n` elements and each element requires `O(log M)` operations, the total time complexity is `O(n × log M)`, where `n` is the length of the array `nums` and `M` is the maximum possible value (here `log M = 16` for 16-bit integers).

**Space Complexity:** `O(n × log M)`

The space is primarily consumed by the Trie structure:
- In the worst case, each number in `nums` creates a unique path in the Trie from root to leaf.
- Each path consists of 16 nodes (one for each bit position from 15 to 0).
- With `n` numbers, the maximum number of nodes created is `O(n × 16) = O(n × log M)`.
- Each Trie node contains an array of size 2 for children and an integer counter, which is `O(1)` space per node.

Therefore, the total space complexity is `O(n × log M)`, where `log M = 16` represents the bit length of the numbers being processed.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Bit Range Assumption**

**Pitfall:** The code assumes all numbers fit within 16 bits (processes bits 15 down to 0). If the input contains numbers larger than 2^16 - 1 (65535), the solution will produce incorrect results because higher-order bits are ignored.

**Example of Failure:**
```python
nums = [100000, 100001, 2]  # 100000 requires at least 17 bits
low = 1
high = 3
# The code would incorrectly handle the XOR of 100000 and 100001
```

**Solution:** Determine the actual bit range needed:
```python
def countPairs(self, nums: List[int], low: int, high: int) -> int:
    # Find the maximum value to determine bit range
    max_val = max(max(nums), high) if nums else 0
    bit_length = max_val.bit_length() if max_val > 0 else 1
  
    # Then modify insert and search to use bit_length instead of hardcoded 15
    # In insert and search methods:
    for i in range(bit_length - 1, -1, -1):  # Instead of range(15, -1, -1)
```

### 2. **Null Node Handling in Search**

**Pitfall:** The search function checks `if node is None` inside the loop, but if a child node doesn't exist when trying to follow a path, the code will crash with an AttributeError when trying to access `node.children`.

**Example of Failure:**
```python
# If the trie is sparse and certain paths don't exist
# node = node.children[x_bit ^ 1]  # This could set node to None
# Then on next iteration: node.children[x_bit]  # AttributeError!
```

**Solution:** Add proper null checks before accessing children:
```python
def search(self, x: int, limit: int) -> int:
    node = self
    count = 0
  
    for i in range(15, -1, -1):
        if node is None:
            return count
      
        x_bit = (x >> i) & 1
        limit_bit = (limit >> i) & 1
      
        if limit_bit == 1:
            # Check if child exists before accessing cnt
            if node.children[x_bit] is not None:
                count += node.children[x_bit].cnt
          
            # Check if the path we want to follow exists
            if node.children[x_bit ^ 1] is not None:
                node = node.children[x_bit ^ 1]
            else:
                return count  # No more valid paths
        else:
            if node.children[x_bit] is not None:
                node = node.children[x_bit]
            else:
                return count  # No more valid paths
  
    return count
```

### 3. **Edge Case: Empty Array or Single Element**

**Pitfall:** While the current code handles these cases correctly (returns 0), developers might overlook testing these scenarios or might try to optimize by adding early returns that break the logic.

**Solution:** Keep the implementation clean and let the loop naturally handle edge cases:
```python
# The current implementation correctly handles:
# - Empty array: loop doesn't execute, returns 0
# - Single element: first iteration finds 0 pairs, inserts, returns 0
# No special casing needed!
```

### 4. **Integer Overflow in Other Languages**

**Pitfall:** When porting this solution to languages with fixed-size integers (like C++ or Java), bit operations might behave differently for negative numbers or cause overflow.

**Solution for Other Languages:**
```python
# In Python, this isn't an issue, but if porting:
# 1. Use unsigned integers where possible
# 2. Explicitly mask results: (x >> i) & 1
# 3. Be careful with the XOR operation on signed integers
```

