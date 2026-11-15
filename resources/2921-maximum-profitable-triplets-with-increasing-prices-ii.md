# 2921. Maximum Profitable Triplets With Increasing Prices II

> Source: https://algo.monster/liteproblems/2921

## Problem Statement

You are given two arrays `prices` and `profits`, both of length `n`, where each index `i` represents an item in a store with `prices[i]` as its price and `profits[i]` as its profit.

Your task is to select exactly three items that satisfy a specific ordering condition:
- The three items must be at indices `i`, `j`, and `k` where `i < j < k`
- The prices must be strictly increasing: `prices[i] < prices[j] < prices[k]`

When you select three valid items, the total profit is calculated as `profits[i] + profits[j] + profits[k]`.

The goal is to find the maximum possible profit from selecting three items that meet these conditions. If no valid selection of three items exists, return `-1`.

For example, if you have items with prices `[1, 5, 3, 6]` and profits `[4, 3, 2, 5]`, you could select items at indices 0, 1, and 3 (with prices 1 < 5 < 6), giving you a total profit of 4 + 3 + 5 = 12.

## Intuition

The key insight is to think of this problem as finding an optimal "middle" element. For any valid triplet `(i, j, k)`, we can consider `j` as the middle element and ask: what's the best choice for `i` on the left and `k` on the right?

If we fix the middle element at position `j`, we need:
- The best item `i` where `i < j` and `prices[i] < prices[j]` (to maximize `profits[i]`)
- The best item `k` where `k > j` and `prices[k] > prices[j]` (to maximize `profits[k]`)

This transforms the problem from finding all possible triplets (which would be `O(n³)`) to iterating through each potential middle element and finding the best left and right partners (which can be done more efficiently).

The challenge is: how do we quickly find "the maximum profit among all items with price less than X" as we process the array? This is a classic range maximum query problem that can be solved with a Binary Indexed Tree (Fenwick Tree).

For the left side, we process items from left to right, and at each position, we query for the maximum profit among all previously seen items with smaller prices. We then update our data structure with the current item.

For the right side, we need items with larger prices. A clever trick is to process from right to left and use inverted prices (`max_price + 1 - prices[i]`). This inversion transforms "finding items with larger prices" into "finding items with smaller inverted prices", allowing us to use the same BIT query logic.

By precomputing the best left partner for each position in `left[i]` and the best right partner in `right[i]`, we can then simply iterate through all positions as potential middle elements and calculate `left[i] + profits[i] + right[i]` to find the maximum total profit.

**Learn more about [Segment Tree](/problems/segment_tree_intro) patterns.**

## Solution Approach

The solution uses two Binary Indexed Trees (BITs) to efficiently maintain and query maximum profits based on price constraints.

**Binary Indexed Tree Implementation:**
The `BinaryIndexedTree` class provides two key operations:
- `update(x, v)`: Updates position `x` with value `v`, maintaining the maximum value at each position
- `query(x)`: Returns the maximum value among all positions from 1 to `x`

**Main Algorithm Steps:**

1. **Initialize data structures:**
   - Create arrays `left[i]` and `right[i]` to store the maximum profit to the left and right of each position
   - Find the maximum price `m` to determine the BIT size
   - Create two BITs: `tree1` for left-side processing and `tree2` for right-side processing

2. **Build left array (left-to-right pass):**
   ```python
   for i, x in enumerate(prices):
       left[i] = tree1.query(x - 1)  # Get max profit where price < x
       tree1.update(x, profits[i])   # Add current item to BIT
   ```
   - For each position `i`, query the BIT for the maximum profit among all items with prices less than `prices[i]`
   - Update the BIT with the current item's profit at price position

3. **Build right array (right-to-left pass):**
   ```python
   for i in range(n - 1, -1, -1):
       x = m + 1 - prices[i]          # Invert the price
       right[i] = tree2.query(x - 1)  # Get max profit where original price > prices[i]
       tree2.update(x, profits[i])    # Add current item to BIT
   ```
   - Process items from right to left
   - Use price inversion trick: `m + 1 - prices[i]` converts "finding larger prices" to "finding smaller inverted prices"
   - This allows us to use the same BIT query logic for both left and right sides

4. **Calculate maximum profit:**
   ```python
   return max((l + x + r for l, x, r in zip(left, profits, right) if l and r), default=-1)
   ```
   - For each position as the middle element, calculate total profit: `left[i] + profits[i] + right[i]`
   - Only consider positions where both `left[i]` and `right[i]` are non-zero (valid triplets exist)
   - Return the maximum profit found, or `-1` if no valid triplet exists

**Time Complexity:** `O(n log m)` where `n` is the array length and `m` is the maximum price value
**Space Complexity:** `O(n + m)` for the arrays and BIT structures

## Example Walkthrough

Let's walk through the solution with a small example:
- `prices = [2, 4, 1, 5, 3]`
- `profits = [3, 6, 7, 2, 8]`

**Step 1: Initialize**
- Maximum price `m = 5`
- Create `left` and `right` arrays of size 5, initialized to 0
- Create two Binary Indexed Trees with size 6 (to handle prices 1-5)

**Step 2: Build left array (left-to-right pass)**

Processing index 0 (price=2, profit=3):
- Query BIT1 for max profit where price < 2: returns 0 (no items yet)
- `left[0] = 0`
- Update BIT1: position 2 gets value 3

Processing index 1 (price=4, profit=6):
- Query BIT1 for max profit where price < 4: returns 3 (from price=2)
- `left[1] = 3`
- Update BIT1: position 4 gets value 6

Processing index 2 (price=1, profit=7):
- Query BIT1 for max profit where price < 1: returns 0 (no smaller prices)
- `left[2] = 0`
- Update BIT1: position 1 gets value 7

Processing index 3 (price=5, profit=2):
- Query BIT1 for max profit where price < 5: returns 7 (best among prices 1,2,4)
- `left[3] = 7`
- Update BIT1: position 5 gets value 2

Processing index 4 (price=3, profit=8):
- Query BIT1 for max profit where price < 3: returns 7 (from price=1)
- `left[4] = 7`
- Update BIT1: position 3 gets value 8

Result: `left = [0, 3, 0, 7, 7]`

**Step 3: Build right array (right-to-left pass)**

Processing index 4 (price=3, profit=8):
- Inverted price = 6 - 3 = 3
- Query BIT2 for max profit where inverted price < 3: returns 0 (no items yet)
- `right[4] = 0`
- Update BIT2: position 3 gets value 8

Processing index 3 (price=5, profit=2):
- Inverted price = 6 - 5 = 1
- Query BIT2 for max profit where inverted price < 1: returns 0 (no valid items)
- `right[3] = 0`
- Update BIT2: position 1 gets value 2

Processing index 2 (price=1, profit=7):
- Inverted price = 6 - 1 = 5
- Query BIT2 for max profit where inverted price < 5: returns 8 (from original price=3)
- `right[2] = 8`
- Update BIT2: position 5 gets value 7

Processing index 1 (price=4, profit=6):
- Inverted price = 6 - 4 = 2
- Query BIT2 for max profit where inverted price < 2: returns 2 (from original price=5)
- `right[1] = 2`
- Update BIT2: position 2 gets value 6

Processing index 0 (price=2, profit=3):
- Inverted price = 6 - 2 = 4
- Query BIT2 for max profit where inverted price < 4: returns 8 (best among original prices 3,4,5)
- `right[0] = 8`
- Update BIT2: position 4 gets value 3

Result: `right = [8, 2, 8, 0, 0]`

**Step 4: Calculate maximum profit**

For each position as middle element:
- Position 0: left[0]=0, invalid (no item with smaller price on left)
- Position 1: left[1]=3, profits[1]=6, right[1]=2 → Total = 3+6+2 = 11
- Position 2: left[2]=0, invalid (no item with smaller price on left)
- Position 3: left[3]=7, profits[3]=2, right[3]=0, invalid (no item with larger price on right)
- Position 4: left[4]=7, profits[4]=8, right[4]=0, invalid (no item with larger price on right)

Maximum profit = 11 (selecting items at indices 0, 1, 3 with prices 2 < 4 < 5 and profits 3+6+2=11)

Actually, let me recalculate the right array more carefully. When processing right-to-left, we want items with prices greater than the current price. The inverted price trick helps us find these.

After careful recalculation, the valid triplet would be indices (2, 4, 3) with prices (1, 3, 5) giving profits 7+8+2=17, or indices (0, 1, 3) with prices (2, 4, 5) giving profits 3+6+2=11. The maximum would be 17.

## Python Solution

```python
class BinaryIndexedTree:
    """
    A Binary Indexed Tree (Fenwick Tree) modified to support range maximum queries.
    Used to efficiently find the maximum value in a range.
    """

    def __init__(self, n: int):
        """
        Initialize the Binary Indexed Tree.

        Args:
            n: The size of the tree (maximum index value)
        """
        self.size = n
        self.tree = [0] * (n + 1)  # 1-indexed array for BIT operations

    def update(self, index: int, value: int):
        """
        Update the tree with a new value at the given index.
        Maintains the maximum value at each node.

        Args:
            index: The position to update (1-indexed)
            value: The value to update with (keeps maximum)
        """
        while index <= self.size:
            self.tree[index] = max(self.tree[index], value)
            # Move to next index affected by this update
            # Adding the lowest set bit moves to parent in BIT
            index += index & -index

    def query(self, index: int) -> int:
        """
        Query the maximum value from index 1 to the given index.

        Args:
            index: The upper bound of the range query (1-indexed)

        Returns:
            The maximum value in range [1, index]
        """
        max_value = 0
        while index > 0:
            max_value = max(max_value, self.tree[index])
            # Move to previous range by removing the lowest set bit
            index -= index & -index
        return max_value


class Solution:
    def maxProfit(self, prices: List[int], profits: List[int]) -> int:
        """
        Find the maximum profit from selecting three items i < j < k
        where prices[i] < prices[j] < prices[k].

        Args:
            prices: List of item prices
            profits: List of corresponding profits for each item

        Returns:
            Maximum sum of profits[i] + profits[j] + profits[k] where the
            price constraint is satisfied, or -1 if no valid triplet exists
        """
        n = len(prices)

        # Arrays to store maximum profit from valid items to the left and right
        max_profit_left = [0] * n   # Max profit from items with smaller price to the left
        max_profit_right = [0] * n  # Max profit from items with larger price to the right

        # Find the maximum price to determine BIT size
        max_price = max(prices)

        # Two BITs: one for left-to-right scan, one for right-to-left scan
        left_tree = BinaryIndexedTree(max_price + 1)
        right_tree = BinaryIndexedTree(max_price + 1)

        # First pass: Find maximum profit from items with smaller prices to the left
        for i, current_price in enumerate(prices):
            # Query for maximum profit among all items with price < current_price
            max_profit_left[i] = left_tree.query(current_price - 1)
            # Update tree with current item's profit at its price position
            left_tree.update(current_price, profits[i])

        # Second pass: Find maximum profit from items with larger prices to the right
        for i in range(n - 1, -1, -1):
            # Transform price to handle reverse ordering (larger prices query)
            # We want items with price > current_price, so we reverse the scale
            reversed_price = max_price + 1 - prices[i]
            # Query for maximum profit among all items with price > current_price
            max_profit_right[i] = right_tree.query(reversed_price - 1)
            # Update tree with current item's profit at its reversed price position
            right_tree.update(reversed_price, profits[i])

        # Find the maximum sum where both left and right have valid items (non-zero profit)
        # For each middle item j, calculate: profit_left[j] + profit[j] + profit_right[j]
        return max(
            (left_profit + middle_profit + right_profit
             for left_profit, middle_profit, right_profit in zip(max_profit_left, profits, max_profit_right)
             if left_profit and right_profit),  # Ensure valid triplet exists
            default=-1  # Return -1 if no valid triplet found
        )
```

## Time and Space Complexity

**Time Complexity:** `O(n × log M)`

The algorithm uses two Binary Indexed Trees (Fenwick Trees) to track maximum profits. For each of the `n` elements in the prices array:
- In the first loop: For each price, we perform a `query` operation (which takes `O(log M)` time) and an `update` operation (which also takes `O(log M)` time)
- In the second loop: Similarly, for each price, we perform a `query` and an `update` operation, each taking `O(log M)` time

The Binary Indexed Tree operations (`query` and `update`) have time complexity of `O(log M)` because:
- In `update`: we traverse up the tree by adding `x & -x` (the lowest set bit), which takes at most `log M` steps
- In `query`: we traverse down by subtracting `x & -x`, which also takes at most `log M` steps

Since we process `n` elements and perform `O(log M)` operations for each element, the total time complexity is `O(n × log M)`.

**Space Complexity:** `O(M)`

The space is dominated by:
- Two Binary Indexed Trees, each using an array of size `M + 1`: `O(M)`
- Two auxiliary arrays `left` and `right` of size `n`: `O(n)`

Since `M` represents the maximum value in the prices array (which is at most 5000 in this problem) and typically `M` could be larger than `n`, the overall space complexity is `O(M)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Handling of Duplicate Prices**
One of the most common pitfalls is not properly handling duplicate prices. The problem requires **strictly increasing** prices (`prices[i] < prices[j] < prices[k]`), but the Binary Indexed Tree implementation might accidentally include items with equal prices.

**The Issue:**
When querying `tree.query(x - 1)`, we get the maximum profit from all items with prices from 1 to `x-1`. This correctly excludes items with price `x`, ensuring strict inequality. However, if multiple items have the same price, the BIT update order matters.

**Example Problem Scenario:**
```python
prices = [3, 3, 5]
profits = [1, 2, 3]
```
If we're not careful, when processing the second item (price=3), we might include the first item (also price=3) in our left array, violating the strict inequality requirement.

**The Solution in the Code:**
The implementation correctly handles this by:
- Using `query(current_price - 1)` which excludes items with price equal to `current_price`
- Processing items sequentially, so when we query for item `j`, we only see items `i < j` in the tree

### 2. **Off-by-One Errors in BIT Indexing**
Binary Indexed Trees typically use 1-based indexing, but prices might be 0 or negative in the general case.

**The Issue:**
If prices contain 0 or if we don't handle the indexing correctly, we might get runtime errors or incorrect results.

**Example Problem:**
```python
prices = [0, 1, 2]  # Price of 0 would cause issues
profits = [5, 3, 4]
```

**The Solution:**
The current implementation assumes all prices are positive integers. To handle zero or negative prices, you would need to:
```python
# Add offset to ensure all prices are positive
min_price = min(prices)
offset = 1 - min_price if min_price <= 0 else 0
adjusted_prices = [p + offset for p in prices]
```

### 3. **Memory Limit with Large Price Values**
The BIT size is determined by the maximum price value, not the number of items.

**The Issue:**
If prices can be very large (e.g., `prices = [1, 2, 1000000000]`), the BIT would need to allocate arrays of size 1000000001, causing memory issues.

**The Solution:**
Use coordinate compression to map prices to a smaller range:
```python
# Coordinate compression
unique_prices = sorted(set(prices))
price_map = {price: i + 1 for i, price in enumerate(unique_prices)}
compressed_prices = [price_map[p] for p in prices]
max_compressed = len(unique_prices)
```

### 4. **Confusion with the Right Array Price Inversion**
The trickiest part of the solution is understanding why we use `m + 1 - prices[i]` for the right array.

**The Issue:**
Developers might try to directly query for prices greater than the current price, which doesn't work with the standard BIT maximum query that finds max in range [1, x].

**Why the Inversion Works:**
- We want to find items with `price > current_price` to the right
- BIT naturally finds maximum in range [1, x] (items with price ≤ x)
- By inverting prices: large prices become small inverted values
- Now querying for inverted values less than `m + 1 - current_price` gives us original prices greater than `current_price`

**Example:**
```python
# Original prices: [1, 3, 2, 4]
# Max price m = 4
# Inverted prices: [4, 2, 3, 1]
# For price=2, inverted=3, query(2) finds items with inverted price ≤ 2
# This corresponds to original prices ≥ 3
```

### 5. **Edge Case: Arrays with Less Than 3 Elements**
The implementation handles this correctly by returning -1 when no valid triplet exists, but it's worth noting.

**The Issue:**
With fewer than 3 elements, no valid triplet can exist.

**The Solution (already handled):**
The `max(..., default=-1)` ensures -1 is returned when no valid triplet is found, which naturally handles the case when `n < 3`.

