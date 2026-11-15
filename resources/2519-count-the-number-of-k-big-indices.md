# 2519. Count the Number of K-Big Indices

> Source: https://algo.monster/liteproblems/2519

## Problem Statement

You are given a 0-indexed integer array `nums` and a positive integer `k`.

An index `i` is called **k-big** if it satisfies both of these conditions:
1. There are at least `k` different indices before position `i` (indices less than `i`) where the values at those indices are smaller than `nums[i]`
2. There are at least `k` different indices after position `i` (indices greater than `i`) where the values at those indices are smaller than `nums[i]`

In other words, for an index to be k-big, the element at that index must have at least `k` smaller elements on its left side AND at least `k` smaller elements on its right side in the array.

Your task is to count how many indices in the array are k-big and return this count.

For example, if `nums = [2, 3, 6, 5, 2, 3]` and `k = 2`:
- Index 2 (value 6): Has 2 smaller values on the left (2, 3) and 2 smaller values on the right (5, 2, 3 where 2 and 3 are smaller). This is k-big.
- Index 3 (value 5): Has 2 smaller values on the left (2, 3) and 2 smaller values on the right (2, 3). This is k-big.
- Other indices don't satisfy both conditions with at least k=2 smaller elements on each side.

## Intuition

The key insight is that for each index, we need to count how many smaller elements exist on its left and right sides. A naive approach would check every pair of indices, resulting in O(n²) time complexity for each position, which is inefficient.

We can think of this problem as maintaining two dynamic sets of numbers - one growing from the left as we traverse, and one shrinking from the right. For each position, we need to quickly query: "How many elements in each set are smaller than the current element?"

This naturally leads us to consider data structures that support:
1. Efficient insertion/deletion of elements
2. Efficient range queries (counting elements less than a given value)

A Binary Indexed Tree (Fenwick Tree) is perfect for this scenario because it can:
- Update values in O(log n) time
- Query prefix sums (which can count elements) in O(log n) time

The clever approach is to use two trees:
- **Tree1**: Starts empty and gradually accumulates elements as we move from left to right. At position `i`, it contains all elements from indices `0` to `i-1`, allowing us to count smaller elements on the left.
- **Tree2**: Initially contains ALL elements, then removes them one by one as we traverse. At position `i`, it contains all elements from indices `i+1` to `n-1`, allowing us to count smaller elements on the right.

By traversing the array once and maintaining these two trees, we can determine for each position whether it's k-big by querying both trees for the count of elements smaller than `nums[i]`. The query `tree.query(v - 1)` gives us the count of all elements strictly less than `v`.

This transforms an O(n²) problem into an O(n log n) solution, as we perform O(log n) operations for each of the n elements.

**Learn more about [Segment Tree](/problems/segment_tree_intro), [Binary Search](/problems/binary_search_intro), [Divide and Conquer](/problems/divide_and_conquer_intro) and [Merge Sort](/problems/merge_k_sorted_lists) patterns.**

## Solution Approach

The solution implements two Binary Indexed Trees (Fenwick Trees) to efficiently track elements smaller than the current value on both sides.

**Binary Indexed Tree Implementation:**
The `BinaryIndexedTree` class provides two key operations:
- `update(x, delta)`: Adds `delta` to position `x` in O(log n) time
- `query(x)`: Returns the sum of elements from position 1 to `x` in O(log n) time

**Main Algorithm Steps:**

1. **Initialize Two Trees:**
   - `tree1 = BinaryIndexedTree(n)` - Initially empty, will track elements on the left
   - `tree2 = BinaryIndexedTree(n)` - Will track elements on the right

2. **Populate tree2 with All Elements:**
   ```python
   for v in nums:
       tree2.update(v, 1)
   ```
   This adds a count of 1 for each value in the array, so `tree2` initially represents all elements.

3. **Traverse and Check Each Position:**
   ```python
   for v in nums:
       tree2.update(v, -1)  # Remove current element from right side
       ans += tree1.query(v - 1) >= k and tree2.query(v - 1) >= k
       tree1.update(v, 1)   # Add current element to left side
   ```

   For each element `v` at position `i`:
   - **Remove from Right Side**: `tree2.update(v, -1)` decrements the count, effectively removing this element from the "right side" set
   - **Check k-big Condition**:
     - `tree1.query(v - 1)` counts elements less than `v` on the left
     - `tree2.query(v - 1)` counts elements less than `v` on the right
     - If both counts are at least `k`, increment the answer
   - **Add to Left Side**: `tree1.update(v, 1)` adds this element to the "left side" set for future positions

**Key Insight:** The algorithm cleverly maintains the invariant that at each position `i`:
- `tree1` contains exactly the elements from indices `0` to `i-1`
- `tree2` contains exactly the elements from indices `i+1` to `n-1`

This allows us to query both trees to get the exact counts needed to determine if position `i` is k-big.

**Time Complexity:** O(n log n) - We perform O(log n) operations for each of the n elements
**Space Complexity:** O(n) - Two trees each storing up to n elements

## Example Walkthrough

Let's walk through a small example with `nums = [2, 3, 6, 5, 2, 3]` and `k = 2`.

**Initial Setup:**
- Create two Binary Indexed Trees of size 6
- `tree1` starts empty (represents left side)
- `tree2` initially contains all elements: {2:2, 3:2, 5:1, 6:1} (value:count)

**Step-by-step traversal:**

**Position 0 (value = 2):**
- Remove 2 from `tree2`: {2:1, 3:2, 5:1, 6:1}
- Check left: `tree1.query(1)` = 0 elements < 2 (not enough, need k=2)
- Check right: `tree2.query(1)` = 1 element < 2 (not enough, need k=2)
- Not k-big ❌
- Add 2 to `tree1`: {2:1}

**Position 1 (value = 3):**
- Remove 3 from `tree2`: {2:1, 3:1, 5:1, 6:1}
- Check left: `tree1.query(2)` = 1 element < 3 (not enough, need k=2)
- Check right: `tree2.query(2)` = 2 elements < 3 (enough!)
- Not k-big ❌ (left side insufficient)
- Add 3 to `tree1`: {2:1, 3:1}

**Position 2 (value = 6):**
- Remove 6 from `tree2`: {2:1, 3:1, 5:1}
- Check left: `tree1.query(5)` = 2 elements < 6 (enough!)
- Check right: `tree2.query(5)` = 3 elements < 6 (enough!)
- Is k-big ✓
- Add 6 to `tree1`: {2:1, 3:1, 6:1}

**Position 3 (value = 5):**
- Remove 5 from `tree2`: {2:1, 3:1}
- Check left: `tree1.query(4)` = 2 elements < 5 (enough!)
- Check right: `tree2.query(4)` = 2 elements < 5 (enough!)
- Is k-big ✓
- Add 5 to `tree1`: {2:1, 3:1, 5:1, 6:1}

**Position 4 (value = 2):**
- Remove 2 from `tree2`: {3:1}
- Check left: `tree1.query(1)` = 1 element < 2 (not enough, need k=2)
- Check right: `tree2.query(1)` = 0 elements < 2 (not enough, need k=2)
- Not k-big ❌
- Add 2 to `tree1`: {2:2, 3:1, 5:1, 6:1}

**Position 5 (value = 3):**
- Remove 3 from `tree2`: {} (empty)
- Check left: `tree1.query(2)` = 2 elements < 3 (enough!)
- Check right: `tree2.query(2)` = 0 elements < 3 (not enough, need k=2)
- Not k-big ❌ (right side insufficient)
- Add 3 to `tree1`: {2:2, 3:2, 5:1, 6:1}

**Result:** 2 indices are k-big (positions 2 and 3)

The key observation is how the trees dynamically maintain the left and right sides: `tree1` grows as we add processed elements, while `tree2` shrinks as we remove the current element, ensuring accurate counts for each position.

## Python Solution

```python
from typing import List


class BinaryIndexedTree:
    """
    Binary Indexed Tree (Fenwick Tree) for efficient range sum queries
    and point updates in O(log n) time.
    """

    def __init__(self, n: int) -> None:
        """
        Initialize a Binary Indexed Tree with size n.

        Args:
            n: The maximum value that can be stored (1-indexed)
        """
        self.size = n
        # Tree array with size n+1 (1-indexed for easier bit manipulation)
        self.tree = [0] * (n + 1)

    def update(self, index: int, delta: int) -> None:
        """
        Add delta to the value at position index.

        Args:
            index: The position to update (1-indexed)
            delta: The value to add to the position
        """
        # Traverse all affected nodes in the tree
        while index <= self.size:
            self.tree[index] += delta
            # Move to the next node that this index affects
            # index & -index gives the lowest set bit
            index += index & -index

    def query(self, index: int) -> int:
        """
        Get the prefix sum from 1 to index (inclusive).

        Args:
            index: The end position of the range (1-indexed)

        Returns:
            The sum of elements from position 1 to index
        """
        result = 0
        # Traverse and accumulate values from the tree
        while index > 0:
            result += self.tree[index]
            # Move to the parent node by removing the lowest set bit
            index -= index & -index
        return result


class Solution:
    def kBigIndices(self, nums: List[int], k: int) -> int:
        """
        Find the number of k-big indices in the array.
        A k-big index i is an index where there are at least k elements
        smaller than nums[i] before position i, and at least k elements
        smaller than nums[i] after position i.

        Args:
            nums: The input array of integers
            k: The threshold for k-big indices

        Returns:
            The count of k-big indices in the array
        """
        n = len(nums)

        # Tree for tracking elements to the left of current position
        left_tree = BinaryIndexedTree(n)

        # Tree for tracking elements to the right of current position
        right_tree = BinaryIndexedTree(n)

        # Initially, all elements are to the right of position 0
        for value in nums:
            right_tree.update(value, 1)

        k_big_count = 0

        # Process each element in the array
        for value in nums:
            # Remove current element from the right tree
            # (as we're now at this position)
            right_tree.update(value, -1)

            # Check if current index is k-big:
            # - left_tree.query(value - 1): count of elements < value on the left
            # - right_tree.query(value - 1): count of elements < value on the right
            if left_tree.query(value - 1) >= k and right_tree.query(value - 1) >= k:
                k_big_count += 1

            # Add current element to the left tree
            # (as we're moving past this position)
            left_tree.update(value, 1)

        return k_big_count
```

## Time and Space Complexity

**Time Complexity:** `O(n × log n)`

The code uses two Binary Indexed Trees (Fenwick Trees) to solve the problem. Let's break down the time complexity:

1. **Initialization of BITs:** Creating two `BinaryIndexedTree` objects takes `O(n)` time (just initializing arrays).

2. **Initial population of tree2:** The first loop iterates through all `n` elements in `nums` and calls `update()` for each. The `update()` operation in a BIT takes `O(log n)` time because it updates at most `log n` nodes (following the path determined by `x += x & -x`). This gives us `O(n × log n)`.

3. **Main processing loop:** This loop iterates through all `n` elements and for each element:
   - Calls `tree2.update(v, -1)`: `O(log n)`
   - Calls `tree1.query(v - 1)`: `O(log n)` (the query traverses at most `log n` nodes via `x -= x & -x`)
   - Calls `tree2.query(v - 1)`: `O(log n)`
   - Calls `tree1.update(v, 1)`: `O(log n)`

   Total per iteration: `4 × O(log n) = O(log n)`
   For `n` iterations: `O(n × log n)`

Overall time complexity: `O(n × log n) + O(n × log n) = O(n × log n)`

**Space Complexity:** `O(n)`

The space usage consists of:
- Two BIT objects, each containing an array `c` of size `n + 1`: `2 × O(n) = O(n)`
- A few integer variables (`ans`, loop variables): `O(1)`

Total space complexity: `O(n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Handling of Value Range**

**The Pitfall:**
The current implementation assumes that all values in `nums` are within the range `[1, n]` where `n = len(nums)`. This is a critical assumption because the Binary Indexed Tree is initialized with size `n`, meaning it can only handle values from 1 to n. If the array contains:
- Values larger than `n`
- Zero or negative values
- Duplicate values that when counted exceed the array size

The code will either crash with an index out of bounds error or produce incorrect results.

**Example of Failure:**
```python
nums = [10, 20, 30, 15, 5]  # Values exceed array length
k = 1
# This will cause index out of bounds when trying to update(10, 1) on a tree of size 5
```

**Solution:**
Compress the values to the range `[1, unique_count]` using coordinate compression:

```python
def kBigIndices(self, nums: List[int], k: int) -> int:
    n = len(nums)

    # Coordinate compression: map values to ranks
    sorted_unique = sorted(set(nums))
    value_to_rank = {v: i + 1 for i, v in enumerate(sorted_unique)}

    # Convert original values to their ranks
    compressed = [value_to_rank[v] for v in nums]

    # Initialize trees with the number of unique values
    max_rank = len(sorted_unique)
    left_tree = BinaryIndexedTree(max_rank)
    right_tree = BinaryIndexedTree(max_rank)

    # Populate right tree with compressed values
    for rank in compressed:
        right_tree.update(rank, 1)

    k_big_count = 0

    for rank in compressed:
        right_tree.update(rank, -1)

        if left_tree.query(rank - 1) >= k and right_tree.query(rank - 1) >= k:
            k_big_count += 1

        left_tree.update(rank, 1)

    return k_big_count
```

### 2. **Handling Duplicate Values Incorrectly**

**The Pitfall:**
When multiple identical values exist in the array, the current approach correctly handles them because each occurrence is treated separately. However, developers might mistakenly think they need special handling for duplicates or might incorrectly try to "optimize" by grouping them.

**Why It Works As-Is:**
Each element is processed individually in sequence, so duplicates naturally get counted correctly:
- When we encounter a duplicate, previous occurrences are already in the left tree
- Future occurrences are still in the right tree
- The query correctly counts all instances smaller than the current value

**What NOT to Do:**
Don't try to batch-process duplicates or skip them thinking they're already handled. Each position needs individual evaluation for the k-big condition.

### 3. **Off-by-One Errors in Queries**

**The Pitfall:**
Using `tree.query(v)` instead of `tree.query(v - 1)` when counting elements strictly less than `v`. The query function returns the count of elements from 1 to the given index (inclusive), so to count elements strictly less than `v`, we need to query up to `v - 1`.

**Incorrect:**
```python
# This counts elements <= v, not < v
left_count = left_tree.query(value)
```

**Correct:**
```python
# This counts elements < v
left_count = left_tree.query(value - 1)
```

### 4. **Tree Initialization Size Error**

**The Pitfall:**
If implementing coordinate compression, initializing the trees with `n` (array length) instead of the number of unique values can waste memory or cause issues if the compressed ranks exceed expectations.

**Best Practice:**
Always size the Binary Indexed Tree based on the maximum value it needs to handle after any compression or transformation is applied.

