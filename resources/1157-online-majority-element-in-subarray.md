# 1157. Online Majority Element In Subarray

> Source: https://algo.monster/liteproblems/1157

## Problem Statement

You need to design a data structure that can efficiently find the **majority element** within any given subarray of an array.

A **majority element** in a subarray is defined as an element that appears at least `threshold` times in that subarray.

The `MajorityChecker` class should support the following operations:

1. **Constructor** `MajorityChecker(int[] arr)`: Initialize the data structure with the given array `arr`.

2. **Query Method** `int query(int left, int right, int threshold)`:
   - Find an element in the subarray `arr[left...right]` (inclusive) that occurs at least `threshold` times
   - Return that element if it exists
   - Return `-1` if no such element exists

For example, if you have an array `[1, 1, 2, 2, 1, 1]` and you query the subarray from index 0 to 5 with threshold 4, the method should return `1` since it appears 4 times in that range. If you query the subarray from index 2 to 3 with threshold 2, it should return `2` since it appears 2 times in that range.

The challenge is to handle multiple queries efficiently after the initial array is provided, making this a problem that requires preprocessing and clever data structure design rather than a naive approach of counting elements for each query.

## Intuition

When we need to find a majority element in a subarray that appears at least `threshold` times, a naive approach would be to count every element in the range for each query, which would be inefficient for multiple queries.

The key insight comes from the **Boyer-Moore Voting Algorithm**, which can find a candidate majority element in linear time. This algorithm works by maintaining a candidate element and a counter - when we see the same element, we increment the counter, and when we see a different element, we decrement it. The element that survives is a potential majority element.

But how do we apply this to arbitrary subarrays efficiently? This is where the **[Segment Tree](/problems/segment_tree_intro)** comes in. We can build a segment tree where each node stores:
- A candidate majority element for that segment
- A "net count" representing how dominant that element is

When merging two segments, we apply Boyer-Moore logic:
- If both segments have the same candidate, we add their counts
- If they have different candidates, the one with the higher count "wins" and we subtract the smaller count from the larger

This gives us a potential majority element for any range in `O(log n)` time. However, the Boyer-Moore algorithm only gives us a **candidate** - it might not actually be the majority element with the required threshold.

To verify if this candidate actually appears `threshold` times, we need to count its actual occurrences in the range. This is where **[binary search](/problems/binary_search_intro)** comes in. By preprocessing and storing all positions where each element appears (in sorted order), we can use binary search to quickly find how many times the candidate appears in our query range:
- Find the leftmost position `>= left`
- Find the rightmost position `<= right`
- The difference gives us the actual count

This combination of [segment tree](/problems/segment_tree_intro) (for finding candidates), Boyer-Moore voting (for merging segments), and [binary search](/problems/binary_search_intro) (for verification) gives us an efficient solution that handles queries in `O(log n)` time after `O(n log n)` preprocessing.

**Learn more about [Segment Tree](/problems/segment_tree_intro) and [Binary Search](/problems/binary_search_intro) patterns.**

## Solution Approach

The solution implements three main components: a [Segment Tree](/problems/segment_tree_intro) with Boyer-Moore voting logic, preprocessing of element positions, and [binary search](/problems/binary_search_intro) for verification.

### 1. [Segment Tree](/problems/segment_tree_intro) Implementation

The `Node` class represents each node in the [segment tree](/problems/segment_tree_intro) with:
- `l, r`: Left and right boundaries of the segment (1-indexed)
- `x`: The candidate majority element for this segment
- `cnt`: The net count of the candidate element

The `SegmentTree` class builds the tree in the constructor:
- `build(u, l, r)`: Recursively builds the tree
  - For leaf nodes (when `l == r`), sets the element value and count to 1
  - For internal nodes, builds left and right children, then merges using `pushup`

### 2. Boyer-Moore Voting Logic in `pushup`

The `pushup(u)` method merges information from child nodes using Boyer-Moore logic:
- If both children have the same candidate element `x`:
  - Parent's candidate = `x`
  - Parent's count = sum of both counts
- If children have different candidates:
  - Parent's candidate = element with higher count
  - Parent's count = difference between the counts (higher - lower)

This maintains the Boyer-Moore invariant: the surviving element with its net count represents the potential majority element.

### 3. Query Operation in [Segment Tree](/problems/segment_tree_intro)

The `query(u, l, r)` method retrieves the candidate for any range:
- If current node's range is completely within query range, return its `(x, cnt)`
- If query range is entirely in left or right half, recurse to that child
- If query spans both children:
  - Get candidates from both sides
  - Merge them using Boyer-Moore logic (same as `pushup`)

### 4. Main Class Implementation

The `MajorityChecker` class combines everything:

**Initialization:**
- Creates the [segment tree](/problems/segment_tree_intro) from the input array
- Builds a dictionary `d` where `d[x]` contains all indices where element `x` appears (sorted)

**Query Method:**
1. Call [segment tree](/problems/segment_tree_intro)'s `query(1, left + 1, right + 1)` to get candidate `x`
   - Note: Adding 1 because segment tree uses 1-based indexing
2. Use [binary search](/problems/binary_search_intro) to verify the candidate:
   - `bisect_left(d[x], left)`: Find first occurrence of `x` at or after `left`
   - `bisect_left(d[x], right + 1)`: Find first occurrence of `x` after `right`
   - The difference `r - l` gives the actual count of `x` in the range
3. Return `x` if count `>= threshold`, otherwise return `-1`

### Time Complexity

- **Preprocessing:** `O(n log n)` to build the [segment tree](/problems/segment_tree_intro) and `O(n)` to build the position dictionary
- **Query:** `O(log n)` for segment tree query + `O(log n)` for two binary searches = `O(log n)` total

### Space Complexity

- `O(n)` for the [segment tree](/problems/segment_tree_intro) (4n nodes)
- `O(n)` for the position dictionary
- Total: `O(n)`

This approach efficiently handles multiple queries by preprocessing the data structure once and then answering each query in logarithmic time.

## Example Walkthrough

Let's walk through the solution with array `[2, 2, 1, 3, 1, 1, 1, 3]` and query `(2, 6, 2)` (find majority element in indices 2-6 with threshold 2).

**Step 1: Build Segment Tree**

The segment tree builds recursively with Boyer-Moore voting at each node:

```
Tree structure (showing [candidate, count]):
                [1,1] (covers indices 1-8)
               /                        \
        [2,1] (1-4)                  [1,3] (5-8)
        /         \                  /         \
    [2,2] (1-2)  [1,1] (3-4)    [1,2] (5-6)  [1,1] (7-8)
    /     \      /     \        /     \      /     \
[2,1]   [2,1]  [1,1]  [3,1]  [1,1]  [1,1]  [1,1]  [3,1]
(1)     (2)    (3)    (4)    (5)    (6)    (7)    (8)
```

Notice how node (1-4) merges its children:
- Left child has [2,2] (element 2 with count 2)
- Right child has [1,1] (element 1 with count 1, and 3 with count 1 cancels out)
- Since candidates differ: winner is 2 with count 2-1=1, so node stores [2,1]

**Step 2: Build Position Dictionary**

Create a map of each element to its positions:
```
d = {
    2: [0, 1],
    1: [2, 4, 5, 6],
    3: [3, 7]
}
```

**Step 3: Query (2, 6, 2)**

Convert to 1-based indexing: query segment tree for range (3, 7).

The segment tree query traverses:
1. Start at root [1,1] covering (1-8)
2. Query range (3-7) spans both children, so query both:
   - Left child (1-4): Only positions 3-4 needed, returns [1,1]
   - Right child (5-8): Only positions 5-7 needed, returns [1,3]
3. Merge results using Boyer-Moore:
   - Both have candidate 1
   - Combined count: 1 + 3 = 4
   - Return candidate [1,4]

**Step 4: Verify Candidate**

Check if element 1 actually appears ≥2 times in original range [2,6]:
```python
positions = d[1] = [2, 4, 5, 6]
left_idx = bisect_left([2, 4, 5, 6], 2) = 0  # First position ≥ 2
right_idx = bisect_left([2, 4, 5, 6], 7) = 4  # First position > 6
count = 4 - 0 = 4
```

Since count (4) ≥ threshold (2), return 1.

**Another Query Example: (0, 2, 3)**

1. Query segment tree for range (1, 3):
   - Returns candidate [2,1] (element 2 with net count 1)
2. Verify element 2:
   ```python
   positions = d[2] = [0, 1]
   left_idx = bisect_left([0, 1], 0) = 0
   right_idx = bisect_left([0, 1], 3) = 2
   count = 2 - 0 = 2
   ```
3. Since count (2) < threshold (3), return -1.

This example demonstrates how the segment tree efficiently finds candidates using Boyer-Moore voting, and binary search verifies the actual count.

## Python Solution

```python
from typing import List
from collections import defaultdict
from bisect import bisect_left


class Node:
    """Node for segment tree storing potential majority element and its count."""
    __slots__ = ("left", "right", "candidate", "count")

    def __init__(self):
        self.left = 0          # Left boundary of segment
        self.right = 0         # Right boundary of segment
        self.candidate = 0     # Potential majority element in this segment
        self.count = 0         # Net count of candidate (using Boyer-Moore logic)


class SegmentTree:
    """
    Segment tree that maintains potential majority elements using Boyer-Moore voting.
    Each node stores a candidate element and its net count after cancellation.
    """

    def __init__(self, nums: List[int]):
        self.nums = nums
        n = len(nums)
        # Allocate 4n nodes for the segment tree (standard practice)
        self.nodes = [Node() for _ in range(n << 2)]
        self.build(1, 1, n)

    def build(self, node_idx: int, left: int, right: int) -> None:
        """Build the segment tree recursively."""
        self.nodes[node_idx].left = left
        self.nodes[node_idx].right = right

        # Base case: leaf node
        if left == right:
            self.nodes[node_idx].candidate = self.nums[left - 1]  # 1-indexed to 0-indexed
            self.nodes[node_idx].count = 1
            return

        # Recursive case: build left and right children
        mid = (left + right) >> 1
        left_child = node_idx << 1
        right_child = (node_idx << 1) | 1

        self.build(left_child, left, mid)
        self.build(right_child, mid + 1, right)

        # Update current node based on children
        self._push_up(node_idx)

    def query(self, node_idx: int, query_left: int, query_right: int) -> tuple[int, int]:
        """
        Query the potential majority element in range [query_left, query_right].
        Returns (candidate, net_count) using Boyer-Moore voting logic.
        """
        current_node = self.nodes[node_idx]

        # Case 1: Current segment is fully within query range
        if current_node.left >= query_left and current_node.right <= query_right:
            return current_node.candidate, current_node.count

        mid = (current_node.left + current_node.right) >> 1
        left_child = node_idx << 1
        right_child = (node_idx << 1) | 1

        # Case 2: Query range is entirely in left subtree
        if query_right <= mid:
            return self.query(left_child, query_left, query_right)

        # Case 3: Query range is entirely in right subtree
        if query_left > mid:
            return self.query(right_child, query_left, query_right)

        # Case 4: Query range spans both subtrees - merge results
        left_candidate, left_count = self.query(left_child, query_left, query_right)
        right_candidate, right_count = self.query(right_child, query_left, query_right)

        # Apply Boyer-Moore voting logic to merge
        if left_candidate == right_candidate:
            return left_candidate, left_count + right_count
        elif left_count >= right_count:
            return left_candidate, left_count - right_count
        else:
            return right_candidate, right_count - left_count

    def _push_up(self, node_idx: int) -> None:
        """
        Update parent node based on its children using Boyer-Moore voting.
        This merges two segments' majority candidates.
        """
        left_child = node_idx << 1
        right_child = (node_idx << 1) | 1

        left_node = self.nodes[left_child]
        right_node = self.nodes[right_child]
        current_node = self.nodes[node_idx]

        # If both children have same candidate, add their counts
        if left_node.candidate == right_node.candidate:
            current_node.candidate = left_node.candidate
            current_node.count = left_node.count + right_node.count
        # Otherwise, the one with higher count survives (with reduced count)
        elif left_node.count >= right_node.count:
            current_node.candidate = left_node.candidate
            current_node.count = left_node.count - right_node.count
        else:
            current_node.candidate = right_node.candidate
            current_node.count = right_node.count - left_node.count


class MajorityChecker:
    """
    Data structure to efficiently check if an element appears >= threshold times
    in any subarray. Uses segment tree to find potential majority candidates.
    """

    def __init__(self, arr: List[int]):
        # Build segment tree for finding potential majority elements
        self.tree = SegmentTree(arr)

        # Store positions of each element for verification
        self.element_positions = defaultdict(list)
        for index, value in enumerate(arr):
            self.element_positions[value].append(index)

    def query(self, left: int, right: int, threshold: int) -> int:
        """
        Check if any element appears >= threshold times in arr[left:right+1].
        Returns the element if found, otherwise -1.
        """
        # Get potential majority candidate from segment tree (1-indexed query)
        candidate, _ = self.tree.query(1, left + 1, right + 1)

        # Verify if candidate actually appears >= threshold times
        # Binary search to count occurrences of candidate in range
        positions = self.element_positions[candidate]
        left_bound = bisect_left(positions, left)
        right_bound = bisect_left(positions, right + 1)

        actual_count = right_bound - left_bound
        return candidate if actual_count >= threshold else -1


# Your MajorityChecker object will be instantiated and called as such:
# obj = MajorityChecker(arr)
# param_1 = obj.query(left,right,threshold)
```

## Time and Space Complexity

**Time Complexity:**
- **Initialization (`__init__`):** `O(n log n)`
  - Building the segment tree takes `O(n)` time as each node is visited once during the build process
  - Creating the dictionary `d` with position lists requires iterating through the array once, which is `O(n)`
  - However, if we consider that each element might need to be appended to a list (which could involve resizing), the worst case remains `O(n)`
  - But actually, the reference answer states `O(n)`, which is correct as the segment tree build is `O(n)` and dictionary creation is `O(n)`, giving total `O(n)`

- **Query (`query`):** `O(log n)`
  - The segment tree query operation traverses at most `O(log n)` nodes from root to leaf
  - The binary search operations (`bisect_left`) on the position list also take `O(log n)` time
  - Total query complexity is `O(log n) + O(log n) = O(log n)`

**Space Complexity:** `O(n)`
- The segment tree uses `4n` nodes (allocated as `n << 2`), which is `O(n)` space
- The dictionary `d` stores the positions of all elements, using `O(n)` space in total (each index appears exactly once across all lists)
- Each `Node` object has a fixed number of attributes due to `__slots__`, maintaining constant space per node
- Total space complexity is `O(n) + O(n) = O(n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Using Wrong Binary Search Template Variant**

A critical mistake is using the wrong binary search variant. The correct template uses `while left <= right` with `right = mid - 1` and tracks the answer with `firstTrueIndex`:

**Incorrect (different variant):**
```java
while (left < right) {
    int mid = (left + right) >> 1;
    if (indices.get(mid) >= target) {
        right = mid;
    } else {
        left = mid + 1;
    }
}
return left;
```

**Correct (template-compliant):**
```java
int firstTrueIndex = indices.size();  // Default if not found
while (left <= right) {
    int mid = left + (right - left) / 2;
    if (indices.get(mid) >= target) {
        firstTrueIndex = mid;
        right = mid - 1;
    } else {
        left = mid + 1;
    }
}
return firstTrueIndex;
```

### 2. **Misunderstanding Boyer-Moore Voting Algorithm Limitations**

**Pitfall:** The Boyer-Moore voting algorithm only guarantees finding an element that appears more than n/2 times if such an element exists. It doesn't guarantee finding elements that appear frequently but less than n/2 times. Many developers mistakenly believe the candidate returned by the segment tree is always the answer.

**Example of the issue:**
```python
# Array: [1, 2, 3, 1, 1]
# Query: left=0, right=4, threshold=3
# Boyer-Moore might return 1 as candidate (correct in this case)
# But for Array: [1, 2, 3, 4, 5, 1, 1]
# Query: left=0, right=6, threshold=3
# Boyer-Moore might not return 1 as the candidate
```

**Solution:** Always verify the candidate by counting its actual occurrences using the position dictionary and binary search. Never skip the verification step:
```python
def query(self, left: int, right: int, threshold: int) -> int:
    candidate, _ = self.tree.query(1, left + 1, right + 1)

    # CRITICAL: Must verify the candidate
    positions = self.element_positions[candidate]
    left_bound = bisect_left(positions, left)
    right_bound = bisect_left(positions, right + 1)
    actual_count = right_bound - left_bound

    # Only return if verified
    return candidate if actual_count >= threshold else -1
```

### 2. **Index Conversion Errors Between 0-based and 1-based**

**Pitfall:** The segment tree uses 1-based indexing internally while the input array and query parameters use 0-based indexing. Mixing these up causes incorrect range queries.

**Common mistakes:**
```python
# WRONG: Forgetting to convert indices
def query(self, left: int, right: int, threshold: int) -> int:
    candidate, _ = self.tree.query(1, left, right)  # Bug!

# WRONG: Converting in the wrong place
def build(self, node_idx: int, left: int, right: int) -> None:
    if left == right:
        # Should be [left - 1], not [left]
        self.nodes[node_idx].candidate = self.nums[left]  # Bug!
```

**Solution:** Consistently handle index conversion at interface boundaries:
```python
def query(self, left: int, right: int, threshold: int) -> int:
    # Convert 0-based to 1-based when calling segment tree
    candidate, _ = self.tree.query(1, left + 1, right + 1)

def build(self, node_idx: int, left: int, right: int) -> None:
    if left == right:
        # Convert 1-based segment tree index to 0-based array index
        self.nodes[node_idx].candidate = self.nums[left - 1]
```

### 3. **Incorrect Binary Search Boundary for Counting Occurrences**

**Pitfall:** Using the wrong boundary in binary search when counting occurrences in a range, especially forgetting that `bisect_left` finds the insertion point, not necessarily an existing element.

**Common mistake:**
```python
# WRONG: Using right instead of right + 1
right_bound = bisect_left(positions, right)  # Bug! Misses element at index 'right'

# WRONG: Using bisect_right for left boundary
left_bound = bisect_right(positions, left)  # Bug! May skip valid occurrences
```

**Solution:** Use `bisect_left` correctly for both boundaries:
```python
# Find first position >= left
left_bound = bisect_left(positions, left)
# Find first position > right (equivalent to >= right + 1)
right_bound = bisect_left(positions, right + 1)
# Count = number of positions in [left, right]
actual_count = right_bound - left_bound
```

### 4. **Memory Allocation for Segment Tree**

**Pitfall:** Allocating insufficient memory for the segment tree. A common mistake is allocating exactly 2n nodes, which isn't enough for all cases.

**Wrong approach:**
```python
# WRONG: Not enough nodes
self.nodes = [Node() for _ in range(n * 2)]  # Bug!
```

**Solution:** Always allocate 4n nodes for a segment tree to handle worst-case scenarios:
```python
# Correct: 4n nodes ensures enough space
self.nodes = [Node() for _ in range(n << 2)]  # n * 4
```

### 5. **Handling Edge Cases with Empty Results**

**Pitfall:** Not handling the case where no element meets the threshold requirement, or assuming the Boyer-Moore candidate always exists.

**Solution:** Always return -1 when no valid element is found:
```python
def query(self, left: int, right: int, threshold: int) -> int:
    candidate, _ = self.tree.query(1, left + 1, right + 1)

    # Handle case where candidate might not exist in element_positions
    if candidate not in self.element_positions:
        return -1

    positions = self.element_positions[candidate]
    left_bound = bisect_left(positions, left)
    right_bound = bisect_left(positions, right + 1)
    actual_count = right_bound - left_bound

    # Explicitly return -1 if threshold not met
    return candidate if actual_count >= threshold else -1
```

