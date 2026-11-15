# 2931. Maximum Spending After Buying Items

> Source: https://algo.monster/liteproblems/2931

## Problem Statement

You have `m` shops, and each shop has `n` items for sale. The value of each item is given in a 2D matrix `values`, where `values[i][j]` represents the value of the `j`-th item in the `i`-th shop.

Important constraints:
- Items in each shop are sorted in non-increasing order from left to right (i.e., `values[i][j] >= values[i][j + 1]`)
- All items across all shops are unique (different items, even if they have the same value)

You need to buy all `m * n` items over multiple days, following these rules:
- On day `d`, you can choose any shop `i`
- From that shop, you must buy the rightmost available item (the one with the highest index that hasn't been bought yet)
- The cost of buying an item with value `v` on day `d` is `v * d`

Your goal is to find the maximum total amount of money that can be spent when buying all items.

For example, if a shop has items with values `[5, 3, 1]`:
- You must buy item with value `1` before you can buy the item with value `3`
- You must buy item with value `3` before you can buy the item with value `5`

The strategy involves deciding which shop to buy from each day to maximize the total spending. Since the cost increases with the day number, you want to save higher-value items for later days while respecting the constraint that you must buy items from right to left within each shop.

## Intuition

Since the cost of an item is `value * day`, and the day number keeps increasing, we want to maximize the total spending by buying items with larger values on later days. This means we should buy items with smaller values first and save the expensive items for later.

The key insight is that we're trying to maximize `Σ(value[i] * day[i])`. To maximize this sum, we should pair the smallest values with the smallest day numbers and the largest values with the largest day numbers.

However, there's a constraint: within each shop, we must buy items from right to left (smallest to largest, since items are sorted in non-increasing order). We can't just pick any item from any shop - we can only buy the rightmost available item from each shop.

This constraint naturally leads us to think about using a min-heap (priority queue). At any point in time, we have access to exactly one item from each shop - the rightmost unbought item. Among these available items, we should always choose the one with the smallest value to buy next.

Here's the strategy:
1. Start by putting the rightmost item from each shop into a min-heap
2. On each day, extract the minimum value item from the heap (this ensures we buy cheaper items first)
3. After buying an item from shop `i` at position `j`, the next available item from that shop is at position `j-1`, so we add it to the heap
4. Continue until all items are purchased

This [greedy](/problems/greedy_intro) approach works because:
- We always have visibility of the currently available item from each shop
- By always choosing the minimum among available items, we ensure smaller values get smaller day multipliers
- The heap maintains the invariant that we're always considering only the valid, available items from each shop

The algorithm essentially simulates the optimal buying sequence while respecting the right-to-left constraint within each shop.

**Learn more about [Greedy](/problems/greedy_intro), [Sorting](/problems/sorting_summary) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

We implement the [greedy](/problems/greedy_intro) strategy using a min-heap (priority queue) to always select the item with the smallest value among all currently available items.

**Data Structure Setup:**
- Use a min-heap `pq` to store tuples of `(value, shop_index, item_index)`
- The heap is ordered by value, so the smallest value item is always at the top

**Initialization:**
```python
pq = [(row[-1], i, n - 1) for i, row in enumerate(values)]
heapify(pq)
```
- For each shop `i`, we add the rightmost item (at index `n-1`) to the heap
- Each entry contains: the item's value `row[-1]`, shop index `i`, and item position `n-1`

**Main Algorithm:**
```python
ans = d = 0
while pq:
    d += 1
    v, i, j = heappop(pq)
    ans += v * d
    if j:
        heappush(pq, (values[i][j - 1], i, j - 1))
```

The process works as follows:

1. **Day Counter**: Initialize day `d = 0` and increment it for each purchase

2. **Extract Minimum**: On each day, extract the item with minimum value from the heap using `heappop(pq)`
   - This gives us the value `v`, shop index `i`, and item position `j`

3. **Calculate Cost**: Add `v * d` to the total answer
   - This represents buying an item with value `v` on day `d`

4. **Add Next Item**: If there are more items in shop `i` (i.e., `j > 0`), add the next item from that shop to the heap
   - The next available item is at position `j-1` with value `values[i][j-1]`

5. **Repeat**: Continue until the heap is empty (all items have been purchased)

**Time Complexity**: `O(m*n * log(m))` where we perform `m*n` heap operations, each taking `O(log m)` time (since at most `m` items are in the heap at once)

**Space Complexity**: `O(m)` for the heap, which stores at most one item per shop at any time

This approach ensures we always buy the cheapest available item each day, maximizing the total spending by saving expensive items for later days when the multiplier is higher.

## Example Walkthrough

Let's walk through a small example with `m = 3` shops and `n = 3` items per shop:

```
values = [[10, 7, 2],    # Shop 0
          [9,  5, 1],    # Shop 1  
          [8,  6, 3]]    # Shop 2
```

Remember: items are sorted in non-increasing order (left to right), but we must buy from right to left in each shop.

**Initial Setup:**
We start by adding the rightmost item from each shop to our min-heap:
- Shop 0: Add item at index 2 with value 2
- Shop 1: Add item at index 2 with value 1
- Shop 2: Add item at index 2 with value 3

Min-heap: `[(1, shop1, idx2), (2, shop0, idx2), (3, shop2, idx2)]`

**Day 1:**
- Extract minimum: `(1, shop1, idx2)`
- Cost: 1 × 1 = 1
- Add next item from shop 1: value 5 at index 1
- Min-heap: `[(2, shop0, idx2), (3, shop2, idx2), (5, shop1, idx1)]`

**Day 2:**
- Extract minimum: `(2, shop0, idx2)`
- Cost: 2 × 2 = 4
- Add next item from shop 0: value 7 at index 1
- Min-heap: `[(3, shop2, idx2), (5, shop1, idx1), (7, shop0, idx1)]`

**Day 3:**
- Extract minimum: `(3, shop2, idx2)`
- Cost: 3 × 3 = 9
- Add next item from shop 2: value 6 at index 1
- Min-heap: `[(5, shop1, idx1), (6, shop2, idx1), (7, shop0, idx1)]`

**Day 4:**
- Extract minimum: `(5, shop1, idx1)`
- Cost: 5 × 4 = 20
- Add next item from shop 1: value 9 at index 0
- Min-heap: `[(6, shop2, idx1), (7, shop0, idx1), (9, shop1, idx0)]`

**Day 5:**
- Extract minimum: `(6, shop2, idx1)`
- Cost: 6 × 5 = 30
- Add next item from shop 2: value 8 at index 0
- Min-heap: `[(7, shop0, idx1), (8, shop2, idx0), (9, shop1, idx0)]`

**Day 6:**
- Extract minimum: `(7, shop0, idx1)`
- Cost: 7 × 6 = 42
- Add next item from shop 0: value 10 at index 0
- Min-heap: `[(8, shop2, idx0), (9, shop1, idx0), (10, shop0, idx0)]`

**Day 7:**
- Extract minimum: `(8, shop2, idx0)`
- Cost: 8 × 7 = 56
- No more items from shop 2
- Min-heap: `[(9, shop1, idx0), (10, shop0, idx0)]`

**Day 8:**
- Extract minimum: `(9, shop1, idx0)`
- Cost: 9 × 8 = 72
- No more items from shop 1
- Min-heap: `[(10, shop0, idx0)]`

**Day 9:**
- Extract minimum: `(10, shop0, idx0)`
- Cost: 10 × 9 = 90
- No more items from shop 0
- Min-heap: empty

**Total cost:** 1 + 4 + 9 + 20 + 30 + 42 + 56 + 72 + 90 = **324**

By always selecting the minimum value item available, we ensured that:
- Smaller values (1, 2, 3) were bought on early days (1, 2, 3)
- Larger values (8, 9, 10) were bought on later days (7, 8, 9)
- This maximizes the total spending since expensive items get multiplied by larger day numbers

## Python Solution

```python
from typing import List
from heapq import heapify, heappop, heappush

class Solution:
    def maxSpending(self, values: List[List[int]]) -> int:
        # Get the number of columns (items per shop)
        num_columns = len(values[0])
      
        # Initialize priority queue with the last (rightmost) item from each row
        # Each tuple contains: (value, row_index, column_index)
        priority_queue = [(row[-1], row_idx, num_columns - 1) 
                         for row_idx, row in enumerate(values)]
      
        # Convert list to min-heap
        heapify(priority_queue)
      
        # Initialize total spending and day counter
        total_spending = 0
        day = 0
      
        # Process items from smallest to largest value
        while priority_queue:
            day += 1
          
            # Extract minimum value item from heap
            current_value, row_idx, col_idx = heappop(priority_queue)
          
            # Add to total spending: value * day
            total_spending += current_value * day
          
            # If there are more items in this row (moving left)
            if col_idx > 0:
                # Add the next item from the same row to the heap
                heappush(priority_queue, 
                        (values[row_idx][col_idx - 1], row_idx, col_idx - 1))
      
        return total_spending
```

## Time and Space Complexity

**Time Complexity:** `O(m × n × log m)`

The algorithm uses a min-heap to process all elements from the 2D array `values` which has `m` rows and `n` columns. 

- Initial heap construction: Creating the initial heap with `m` elements (one from each row) takes `O(m)` time using `heapify`.
- Main loop iterations: The while loop runs exactly `m × n` times since we process every element in the matrix exactly once.
- Heap operations per iteration: In each iteration, we perform one `heappop` operation which takes `O(log m)` time. We also perform at most one `heappush` operation which also takes `O(log m)` time. The heap size never exceeds `m` elements since we maintain at most one element from each row.
- Total time: `O(m) + O(m × n × log m) = O(m × n × log m)`

**Space Complexity:** `O(m)`

The space is dominated by the priority queue `pq`. The heap maintains at most `m` elements at any given time - one element from each row of the matrix. Each element in the heap is a tuple containing three values `(value, row_index, column_index)`, but the number of such tuples is bounded by `m`. Therefore, the space complexity is `O(m)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Heap Initialization - Using Left-Most Items Instead of Right-Most**

A common mistake is initializing the heap with the left-most (first) items from each shop instead of the right-most (last) items.

**Incorrect Implementation:**
```python
# Wrong: Starting from index 0 (left-most items)
priority_queue = [(row[0], row_idx, 0) 
                 for row_idx, row in enumerate(values)]
```

**Why This Fails:**
- The problem requires buying items from right to left within each shop
- You must buy the rightmost available item before you can access items to its left
- Starting from the left violates this constraint

**Correct Implementation:**
```python
# Correct: Starting from the last index (right-most items)
priority_queue = [(row[-1], row_idx, len(row) - 1) 
                 for row_idx, row in enumerate(values)]
```

### 2. **Incorrectly Adding Next Items to the Heap**

Another pitfall is adding the wrong next item after purchasing from a shop.

**Incorrect Implementation:**
```python
# Wrong: Moving to the right (increasing index)
if col_idx < len(values[0]) - 1:
    heappush(priority_queue, 
            (values[row_idx][col_idx + 1], row_idx, col_idx + 1))
```

**Why This Fails:**
- After buying an item at position `j`, the next available item in that shop is at position `j-1` (moving left)
- Moving right would attempt to access already-purchased items

**Correct Implementation:**
```python
# Correct: Moving to the left (decreasing index)
if col_idx > 0:
    heappush(priority_queue, 
            (values[row_idx][col_idx - 1], row_idx, col_idx - 1))
```

### 3. **Starting Day Counter at 1 Instead of 0**

**Incorrect Implementation:**
```python
day = 1  # Wrong starting value
while priority_queue:
    # day is already 1 on first iteration
    current_value, row_idx, col_idx = heappop(priority_queue)
    total_spending += current_value * day
    day += 1
```

**Why This Matters:**
While this doesn't break the algorithm logic, it's cleaner to start at 0 and increment at the beginning of each iteration. This makes the code more intuitive and matches the pattern of "increment then use."

**Correct Implementation:**
```python
day = 0
while priority_queue:
    day += 1  # Increment first, then use
    current_value, row_idx, col_idx = heappop(priority_queue)
    total_spending += current_value * day
```

### 4. **Misunderstanding the Greedy Strategy**

Some might think we should buy the most expensive items first to maximize spending, but this is incorrect.

**Why the Greedy Approach Works:**
- Cost = value × day, so delaying expensive items increases their contribution
- By buying cheap items early (small day numbers), we save expensive items for later (large day numbers)
- This maximizes the total: small_value × small_day + large_value × large_day

**Example:**
With items [1, 5]:
- Strategy 1: Buy 5 on day 1, then 1 on day 2 → Total = 5×1 + 1×2 = 7
- Strategy 2: Buy 1 on day 1, then 5 on day 2 → Total = 1×1 + 5×2 = 11 ✓

