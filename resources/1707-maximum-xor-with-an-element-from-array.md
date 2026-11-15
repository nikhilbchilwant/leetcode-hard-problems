# 1707. Maximum XOR With an Element From Array

> Source: https://algo.monster/liteproblems/1707

## Problem Statement

You are given an array `nums` containing non-negative integers and a `queries` array where each query is represented as `queries[i] = [xi, mi]`.

For each query `i`, you need to find the maximum XOR value between `xi` and any element in `nums` that is less than or equal to `mi`. Specifically:
- Find all elements `nums[j]` where `nums[j] <= mi`
- Calculate `nums[j] XOR xi` for each valid element
- Return the maximum XOR value among all these calculations
- If no element in `nums` satisfies `nums[j] <= mi`, return `-1` for that query

The final result should be an array `answer` where `answer[i]` contains the result for the `i-th` query.

**Key Points:**
- XOR operation is performed bitwise between two numbers
- Each query has a value constraint `mi` that limits which elements from `nums` can be considered
- The goal is to maximize the XOR result while respecting the constraint
- Queries are independent of each other

**Example Understanding:**
If `nums = [0, 1, 2, 3, 4]` and we have a query `[3, 2]`:
- Valid elements from `nums` are those `<= 2`, which are `[0, 1, 2]`
- Calculate XOR: `3 XOR 0 = 3`, `3 XOR 1 = 2`, `3 XOR 2 = 1`
- Maximum XOR value is `3`, so the answer for this query is `3`

## Intuition

To maximize XOR between two numbers, we want their bits to differ as much as possible, especially in the higher-order bits since they contribute more to the final value. For example, if we can make the most significant bit different, that alone gives us a value of at least `2^30` (for 31-bit numbers).

The challenge here is the constraint `mi` - we can only use elements from `nums` that don't exceed this limit. A naive approach would process each query independently by filtering valid elements and checking XOR with each one, but this would be inefficient.

The key insight is that we can process queries in a specific order to avoid redundant work. If we sort queries by their `mi` values in ascending order, we can incrementally add elements from `nums` to our data structure. Once an element is valid for a query with limit `mi`, it remains valid for all subsequent queries with larger limits.

This leads us to think about using a [Trie](/problems/trie_intro) (prefix tree) to store binary representations of numbers. Why a Trie? Because when finding maximum XOR, we want to greedily choose the opposite bit at each position if possible. Starting from the most significant bit, if `xi` has a `0` at position `k`, we want to find a number with `1` at that position (and vice versa).

The [Trie](/problems/trie_intro) allows us to make these greedy choices efficiently:
- Insert numbers from `nums` as paths in the binary trie (each edge represents a bit: 0 or 1)
- For each query value `xi`, traverse the trie by trying to take the opposite bit at each level
- This greedy approach guarantees the maximum XOR value

By combining offline query processing (sorting queries by `mi`) with the binary [trie](/problems/trie_intro) structure, we can:
1. Sort both `nums` and queries
2. Process queries in order of increasing `mi`
3. Incrementally add valid elements to the trie
4. Use the trie to find maximum XOR efficiently

This way, each element is inserted into the [trie](/problems/trie_intro) exactly once, and each query is answered in `O(31)` time (the number of bits).

**Learn more about [Trie](/problems/trie_intro) patterns.**

## Solution Approach

The solution uses **offline query processing** combined with a **binary [trie](/problems/trie_intro)** data structure. Here's how the implementation works:

### 1. Binary [Trie](/problems/trie_intro) Structure

The `[Trie](/problems/trie_intro)` class maintains a binary tree where each node has at most two children (representing bits 0 and 1):
- `children[0]` points to the subtree for bit 0
- `children[1]` points to the subtree for bit 1

**Insert Operation**: For each number `x`, we extract its bits from most significant (bit 30) to least significant (bit 0) and create a path in the [trie](/problems/trie_intro):
```python
def insert(self, x: int):
    node = self
    for i in range(30, -1, -1):
        v = x >> i & 1  # Extract bit at position i
        if node.children[v] is None:
            node.children[v] = Trie()
        node = node.children[v]
```

**Search Operation**: To find maximum XOR with `x`, we greedily try to take the opposite bit at each level:
```python
def search(self, x: int) -> int:
    node = self
    ans = 0
    for i in range(30, -1, -1):
        v = x >> i & 1  # Current bit of x
        if node.children[v ^ 1]:  # Try opposite bit first
            ans |= 1 << i  # Set bit i in result
            node = node.children[v ^ 1]
        elif node.children[v]:  # Settle for same bit
            node = node.children[v]
        else:
            return -1  # [Trie](/problems/trie_intro) is empty
```

### 2. Offline Query Processing

Instead of processing queries in their original order, we sort them by `mi` values:

1. **Sort nums array**: `nums.sort()` - This allows us to add elements incrementally
2. **Sort queries with indices**: Create tuples `(index, [xi, mi])` and sort by `mi`
3. **Process queries in sorted order**:
   ```python
   for i, (x, m) in sorted(zip(range(n), queries), key=lambda x: x[1][1]):
   ```

### 3. Incremental [Trie](/problems/trie_intro) Building

We maintain a pointer `j` to track which elements from `nums` have been inserted:
```python
while j < len(nums) and nums[j] <= m:
    [trie](/problems/trie_intro).insert(nums[j])
    j += 1
```

For each query with limit `m`:
- Insert all elements from `nums` that are `<= m` into the [trie](/problems/trie_intro)
- These elements remain in the trie for subsequent queries (which have larger `m` values)
- Search for maximum XOR with `x` using only the elements currently in the trie

### 4. Preserving Original Order

Since queries are processed out of order, we use the original index `i` to place results in the correct position:
```python
ans = [-1] * n  # Initialize all answers to -1
for i, (x, m) in sorted(...):
    # Process query
    ans[i] = [trie](/problems/trie_intro).search(x)  # Place result at original index
```

### Time Complexity
- Sorting `nums`: `O(n log n)` where `n = len(nums)`
- Sorting queries: `O(q log q)` where `q = len(queries)`
- [Trie](/problems/trie_intro) operations: Each insert/search takes `O(31)` = `O(1)`
- Total: `O(n log n + q log q + (n + q) * 31)` = `O(n log n + q log q)`

### Space Complexity
- [Trie](/problems/trie_intro) can store at most `n` numbers, each creating a path of length 31: `O(31n)` = `O(n)`
- Answer array: `O(q)`
- Total: `O(n + q)`

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Input:**
- `nums = [5, 2, 8, 4]`
- `queries = [[3, 5], [2, 3], [7, 8]]`

**Step 1: Sort nums and prepare queries with indices**
- Sorted `nums = [2, 4, 5, 8]`
- Queries with indices: `[(0, [3, 5]), (1, [2, 3]), (2, [7, 8])]`
- Sort by `mi` values: `[(1, [2, 3]), (0, [3, 5]), (2, [7, 8])]`

**Step 2: Process queries in sorted order**

Initialize: `j = 0`, `ans = [-1, -1, -1]`, empty Trie

**Query (1, [2, 3]):** `xi = 2`, `mi = 3`
- Add elements ≤ 3 to Trie: insert `2` (binary: 010)
- Trie now contains: `2`
- Find max XOR of `2` with Trie elements:
  - `2` (010) XOR `2` (010) = `0`
- `ans[1] = 0`

**Query (0, [3, 5]):** `xi = 3`, `mi = 5`
- Add elements ≤ 5 to Trie: insert `4` (binary: 100) and `5` (binary: 101)
- Trie now contains: `2, 4, 5`
- Find max XOR of `3` with Trie elements:
  - For `3` (011), traverse Trie seeking opposite bits:
    - Bit 2: `3` has 0, try 1 → found (leads to 4 or 5)
    - Bit 1: `3` has 1, try 0 → found (leads to 4)
    - Bit 0: `3` has 1, try 0 → found (4)
  - Path gives us `4`, so `3` XOR `4` = `7`
- `ans[0] = 7`

**Query (2, [7, 8]):** `xi = 7`, `mi = 8`
- Add elements ≤ 8 to Trie: insert `8` (binary: 1000)
- Trie now contains: `2, 4, 5, 8`
- Find max XOR of `7` with Trie elements:
  - For `7` (0111), traverse Trie seeking opposite bits:
    - Bit 3: `7` has 0, try 1 → found (leads to 8)
    - Bit 2: `7` has 1, try 0 → found (8)
    - Continue traversal...
  - Best path gives us `8`, so `7` XOR `8` = `15`
- `ans[2] = 15`

**Final Result:** `[7, 0, 15]`

The key insight is that by processing queries in ascending order of `mi`, we can incrementally build our Trie and never need to remove elements, making the solution efficient.

## Python Solution

```python
class Trie:
    """Binary trie to store numbers and find maximum XOR"""
    __slots__ = ["children"]

    def __init__(self):
        # Each node has two children: 0 and 1
        self.children = [None] * 2

    def insert(self, x: int) -> None:
        """Insert a number into the trie by its binary representation"""
        node = self
        # Process bits from most significant to least significant (30th to 0th bit)
        for i in range(30, -1, -1):
            # Extract the i-th bit of x
            bit = (x >> i) & 1
            # Create new node if path doesn't exist
            if node.children[bit] is None:
                node.children[bit] = Trie()
            # Move to the next node
            node = node.children[bit]

    def search(self, x: int) -> int:
        """Find the maximum XOR value with x among all inserted numbers"""
        node = self
        max_xor = 0
      
        # Process bits from most significant to least significant
        for i in range(30, -1, -1):
            # Extract the i-th bit of x
            bit = (x >> i) & 1
            # Try to go opposite direction for maximum XOR
            toggled_bit = bit ^ 1
          
            if node.children[toggled_bit]:
                # If opposite bit exists, add this bit position to result
                max_xor |= (1 << i)
                node = node.children[toggled_bit]
            elif node.children[bit]:
                # If only same bit exists, follow that path
                node = node.children[bit]
            else:
                # No valid path exists (trie is empty at this point)
                return -1
              
        return max_xor


class Solution:
    def maximizeXor(self, nums: List[int], queries: List[List[int]]) -> List[int]:
        """
        Find maximum XOR for each query where query[i] = [xi, mi]
        Returns max(xi XOR num) for all num in nums where num <= mi
        """
        trie = Trie()
        nums.sort()  # Sort nums to process them in ascending order
      
        num_index = 0
        num_queries = len(queries)
        result = [-1] * num_queries
      
        # Process queries in ascending order of their limits (mi)
        # Enumerate queries with their original indices to maintain result order
        sorted_queries = sorted(enumerate(queries), key=lambda item: item[1][1])
      
        for original_index, (xi, mi) in sorted_queries:
            # Insert all numbers <= mi into the trie
            while num_index < len(nums) and nums[num_index] <= mi:
                trie.insert(nums[num_index])
                num_index += 1
          
            # Find maximum XOR with xi among inserted numbers
            result[original_index] = trie.search(xi)
          
        return result
```

## Time and Space Complexity

**Time Complexity:** `O(n * log n + m * (log m + log M))`

Breaking down the operations:
- Sorting `nums`: `O(n * log n)` where `n = len(nums)`
- Sorting queries with their indices: `O(m * log m)` where `m = len(queries)`
- For each query:
  - Potentially insert multiple numbers into the Trie. In worst case, all `n` numbers are inserted across all queries, each insertion takes `O(log M)` time (31 bits for numbers up to `10^9`)
  - Search operation in Trie: `O(log M)` per query
- Total for Trie operations: `O(n * log M + m * log M) = O((n + m) * log M)`

Since we process each number at most once for insertion and perform one search per query, the overall time complexity is `O(n * log n + m * (log m + log M))`. When `n` is comparable to `m`, this can be simplified to `O(m * log m + n * (log n + log M))`.

**Space Complexity:** `O(n * log M)`

The space is dominated by the Trie structure:
- Each number insertion creates at most `O(log M)` nodes (31 nodes for 31 bits)
- In worst case, all `n` numbers are inserted into the Trie
- Each node stores an array of size 2 for children
- Additional space: `O(m)` for the answer array and `O(m)` for sorting queries

Therefore, the total space complexity is `O(n * log M)` where `M ≤ 10^9`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Bit Range Assumption**
A common mistake is assuming 32-bit integers and iterating from bit 31 to 0, when the problem constraints might specify different ranges. If `nums` contains values up to 10^9, we only need 30 bits (since 2^30 > 10^9).

**Pitfall Code:**
```python
for i in range(31, -1, -1):  # Wrong if max value is 10^9
    bit = (x >> i) & 1
```

**Solution:**
Check the problem constraints and adjust the bit range accordingly:
```python
# For values up to 10^9, use 30 bits (0-indexed from 29 to 0)
for i in range(29, -1, -1):  # Correct for max value 10^9
    bit = (x >> i) & 1
```

### 2. **Not Handling Empty Trie in Search**
The search function might not properly handle the case when no elements have been inserted yet (empty trie), leading to AttributeError when accessing None children.

**Pitfall Code:**
```python
def search(self, x: int) -> int:
    node = self
    ans = 0
    for i in range(30, -1, -1):
        v = x >> i & 1
        # Doesn't check if both children are None
        if node.children[v ^ 1]:
            ans |= 1 << i
            node = node.children[v ^ 1]
        else:
            node = node.children[v]  # Could be None!
```

**Solution:**
Add proper null checking:
```python
def search(self, x: int) -> int:
    node = self
    ans = 0
    for i in range(30, -1, -1):
        v = x >> i & 1
        if node.children[v ^ 1]:
            ans |= 1 << i
            node = node.children[v ^ 1]
        elif node.children[v]:
            node = node.children[v]
        else:
            return -1  # No valid path exists
    return ans
```

### 3. **Processing Queries in Original Order**
A critical mistake is processing queries in their original order instead of sorting by `mi`, which defeats the purpose of offline processing and makes the solution inefficient.

**Pitfall Code:**
```python
for i, (x, m) in enumerate(queries):  # Wrong! Processes in original order
    j = 0
    trie = Trie()  # Rebuilding trie for each query!
    while j < len(nums) and nums[j] <= m:
        trie.insert(nums[j])
        j += 1
    ans[i] = trie.search(x)
```

**Solution:**
Sort queries by `mi` while preserving original indices:
```python
# Create tuples with original indices and sort by mi
sorted_queries = sorted(enumerate(queries), key=lambda item: item[1][1])
for original_index, (xi, mi) in sorted_queries:
    # Process incrementally
    while num_index < len(nums) and nums[num_index] <= mi:
        trie.insert(nums[num_index])
        num_index += 1
    result[original_index] = trie.search(xi)
```

### 4. **Resetting the Trie or Index for Each Query**
Another mistake is resetting the trie or the nums index for each query, which eliminates the efficiency gain of offline processing.

**Pitfall Code:**
```python
for i, (x, m) in sorted_queries:
    j = 0  # Wrong! Resets index for each query
    temp_trie = Trie()  # Wrong! Creates new trie each time
    while j < len(nums) and nums[j] <= m:
        temp_trie.insert(nums[j])
        j += 1
```

**Solution:**
Maintain a single trie and index throughout:
```python
trie = Trie()
num_index = 0  # Initialize once outside the loop
for i, (x, m) in sorted_queries:
    # Continue from where we left off
    while num_index < len(nums) and nums[num_index] <= m:
        trie.insert(nums[num_index])
        num_index += 1  # Increments persistently
```

### 5. **Forgetting to Initialize Result Array with -1**
The problem requires returning -1 when no valid elements exist, but forgetting to initialize the result array with -1 values can lead to incorrect outputs.

**Pitfall Code:**
```python
result = [0] * num_queries  # Wrong! Should be -1 for no valid elements
```

**Solution:**
Initialize with -1:
```python
result = [-1] * num_queries  # Correct initialization
```

