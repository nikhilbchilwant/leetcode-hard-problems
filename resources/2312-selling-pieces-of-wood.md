# 2312. Selling Pieces of Wood

> Source: https://algo.monster/liteproblems/2312

## Problem Statement

You have a rectangular piece of wood with height `m` and width `n`. You want to cut this wood into smaller pieces and sell them to maximize your profit.

You're given a 2D array `prices` where each element `prices[i] = [hi, wi, pricei]` tells you that:
- You can sell a rectangular piece with height `hi` and width `wi` for `pricei` dollars
- You can sell multiple pieces of the same dimensions
- You don't have to sell all the pieces you cut

**Cutting Rules:**
- Each cut must go completely across the wood (either full height or full width)
- A vertical cut splits the wood into left and right pieces
- A horizontal cut splits the wood into top and bottom pieces
- You can make as many cuts as you want
- You cannot rotate pieces (a 2×3 piece is different from a 3×2 piece)

**Goal:** Find the maximum money you can earn by optimally cutting the `m × n` wood and selling the resulting pieces according to the given prices.

**Example:** If you have a 4×6 wood and prices include selling a 2×3 piece for $10, you could potentially cut the wood into multiple 2×3 pieces and sell them. The challenge is finding the optimal cutting strategy that maximizes total revenue.

The solution uses [dynamic programming](/problems/dynamic_programming_intro) with [memoization](/problems/memoization_intro), where `dfs(h, w)` calculates the maximum money from a piece of height `h` and width `w`. It considers:
1. Selling the piece as-is (if a price exists for those dimensions)
2. Making horizontal cuts at different positions and recursively solving for both pieces
3. Making vertical cuts at different positions and recursively solving for both pieces

## Intuition

When faced with this wood cutting problem, we need to find the optimal way to cut a piece of wood to maximize profit. This naturally leads us to think about [dynamic programming](/problems/dynamic_programming_intro) because:

1. **Optimal Substructure**: The maximum profit from a piece of wood depends on the maximum profits from smaller pieces. If we cut a 4×6 piece into two 2×6 pieces, the best total profit is the sum of the best profits from each 2×6 piece.

2. **Overlapping Subproblems**: When cutting different pieces, we'll often encounter the same dimensions repeatedly. For example, cutting a 4×6 piece horizontally in the middle gives two 2×6 pieces, and cutting a 6×6 piece after removing a 2×6 section also gives a 4×6 piece. We don't want to recalculate the same dimensions multiple times.

The key insight is that for any piece of wood with dimensions `(h, w)`, we have exactly these options:
- Sell it as-is if there's a price for those exact dimensions
- Cut it horizontally at any position from 1 to `h-1` and maximize the sum of profits from both pieces
- Cut it vertically at any position from 1 to `w-1` and maximize the sum of profits from both pieces

We want the maximum among all these choices.

This recursive structure suggests using [memoization](/problems/memoization_intro) (top-down DP). Starting with the full `m×n` piece, we recursively break it down into smaller pieces, trying all possible cuts. The function `dfs(h, w)` represents "what's the maximum money I can get from a piece of height `h` and width `w`?"

The base case is when we can sell the piece directly (if its dimensions match a price in our list). Otherwise, we try all possible cuts and take the maximum. The `@cache` decorator ensures we don't recalculate the same dimensions twice, making our solution efficient.

**Learn more about [Memoization](/problems/memoization_intro) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The implementation uses [memoization](/problems/memoization_intro) search (top-down [dynamic programming](/problems/dynamic_programming_intro)) to solve this problem efficiently.

**Data Structure Setup:**
First, we create a nested dictionary `d` to store the prices for quick lookup. We use `defaultdict(dict)` where `d[h][w]` gives us the price for a wood piece with height `h` and width `w`. This allows O(1) price lookups during our calculations.

```python
d = defaultdict(dict)
for h, w, p in prices:
    d[h][w] = p
```

**Recursive Function with [Memoization](/problems/memoization_intro):**
The core of the solution is the `dfs(h, w)` function that calculates the maximum money for a piece with height `h` and width `w`. The `@cache` decorator automatically memoizes results to avoid redundant calculations.

**Algorithm Steps:**

1. **Base Value**: Start with `ans = d[h].get(w, 0)` - this checks if we can sell the current piece as-is. If there's no price for these exact dimensions, we start with 0.

2. **Try Horizontal Cuts**: Iterate through all possible horizontal cut positions from 1 to `h // 2 + 1`:
   ```python
   for i in range(1, h // 2 + 1):
       ans = max(ans, dfs(i, w) + dfs(h - i, w))
   ```
   - Cut at position `i` creates two pieces: one with height `i` and another with height `h - i`
   - Both pieces maintain the same width `w`
   - We only iterate to `h // 2 + 1` because cutting at position `i` or `h - i` gives the same two pieces

3. **Try Vertical Cuts**: Similarly, iterate through all possible vertical cut positions from 1 to `w // 2 + 1`:
   ```python
   for i in range(1, w // 2 + 1):
       ans = max(ans, dfs(h, i) + dfs(h, w - i))
   ```
   - Cut at position `i` creates two pieces: one with width `i` and another with width `w - i`
   - Both pieces maintain the same height `h`

4. **Return Maximum**: The function returns the maximum value among selling as-is and all possible cuts.

**Optimization**: The solution only iterates to half of each dimension (`h // 2 + 1` and `w // 2 + 1`) because cutting a 6-unit piece at position 2 or position 4 produces the same two pieces (2-unit and 4-unit), just in different order. This reduces redundant calculations.

The final answer is obtained by calling `dfs(m, n)` for the original wood dimensions.

## Example Walkthrough

Let's walk through a small example with a 3×4 wood piece and the following prices:
- 1×2 piece sells for $3
- 2×2 piece sells for $7
- 3×4 piece sells for $10

We'll trace through `dfs(3, 4)` to find the maximum profit.

**Step 1: Initial call `dfs(3, 4)`**
- Check if we can sell 3×4 as-is: Yes, for $10. So `ans = 10`
- Try horizontal cuts (i from 1 to 2):
  - Cut at position 1: `dfs(1, 4) + dfs(2, 4)`
  - Cut at position 2: `dfs(2, 4) + dfs(1, 4)` (same pieces, different order)
- Try vertical cuts (i from 1 to 2):
  - Cut at position 1: `dfs(3, 1) + dfs(3, 3)`
  - Cut at position 2: `dfs(3, 2) + dfs(3, 2)`

**Step 2: Calculate `dfs(1, 4)`**
- No price for 1×4, so `ans = 0`
- No horizontal cuts possible (height is 1)
- Try vertical cuts:
  - Cut at 1: `dfs(1, 1) + dfs(1, 3)` = 0 + 0 = 0
  - Cut at 2: `dfs(1, 2) + dfs(1, 2)` = 3 + 3 = 6
- Result: `dfs(1, 4) = 6`

**Step 3: Calculate `dfs(2, 4)`**
- No price for 2×4, so `ans = 0`
- Try horizontal cut at 1: `dfs(1, 4) + dfs(1, 4)` = 6 + 6 = 12
- Try vertical cuts:
  - Cut at 1: `dfs(2, 1) + dfs(2, 3)` = 0 + 0 = 0
  - Cut at 2: `dfs(2, 2) + dfs(2, 2)` = 7 + 7 = 14
- Result: `dfs(2, 4) = 14`

**Step 4: Back to `dfs(3, 4)`**
- Horizontal cut at 1: `dfs(1, 4) + dfs(2, 4)` = 6 + 14 = 20
- Vertical cut at 2: `dfs(3, 2) + dfs(3, 2)`
  - `dfs(3, 2)`: No direct price, try cuts
    - Horizontal cuts give us combinations of 1×2 and 2×2 pieces
    - Best is cutting at position 1: `dfs(1, 2) + dfs(2, 2)` = 3 + 7 = 10
  - So vertical cut at 2: 10 + 10 = 20

**Final Result:**
The maximum of:
- Sell as-is: $10
- Best horizontal cut: $20 (splitting into 1×4 and 2×4)
- Best vertical cut: $20 (splitting into two 3×2 pieces)

Therefore, `dfs(3, 4) = 20`

The optimal strategy is to either:
1. Cut horizontally to get 1×4 and 2×4, then cut each into two 1×2 pieces (worth $3 each) and two 2×2 pieces (worth $7 each)
2. Cut vertically to get two 3×2 pieces, each optimally cut into one 1×2 and one 2×2 piece

Both strategies yield $20 total profit, which is better than selling the 3×4 piece as-is for $10.

## Python Solution

```python
from typing import List
from functools import cache
from collections import defaultdict

class Solution:
    def sellingWood(self, m: int, n: int, prices: List[List[int]]) -> int:
        # Create a nested dictionary to store prices for specific dimensions
        # price_map[height][width] = price
        price_map = defaultdict(dict)
        for height, width, price in prices:
            price_map[height][width] = price
      
        @cache
        def dfs(height: int, width: int) -> int:
            """
            Dynamic programming function to find maximum profit for a wood piece.
          
            Args:
                height: Current height of the wood piece
                width: Current width of the wood piece
          
            Returns:
                Maximum profit obtainable from this wood piece
            """
            # Base case: check if current dimensions have a direct price
            max_profit = price_map[height].get(width, 0)
          
            # Try all possible horizontal cuts
            for cut_height in range(1, height // 2 + 1):
                # Split into two pieces: (cut_height, width) and (height - cut_height, width)
                profit_from_horizontal_cut = dfs(cut_height, width) + dfs(height - cut_height, width)
                max_profit = max(max_profit, profit_from_horizontal_cut)
          
            # Try all possible vertical cuts
            for cut_width in range(1, width // 2 + 1):
                # Split into two pieces: (height, cut_width) and (height, width - cut_width)
                profit_from_vertical_cut = dfs(height, cut_width) + dfs(height, width - cut_width)
                max_profit = max(max_profit, profit_from_vertical_cut)
          
            return max_profit
      
        # Start the recursive search with the full dimensions
        return dfs(m, n)
```

## Time and Space Complexity

The time complexity is `O(m × n × (m + n) + p)`, where:
- `p` is the length of the prices array (for preprocessing the dictionary)
- `m × n` represents the number of unique subproblems (all possible height-width combinations)
- For each subproblem `(h, w)`, we iterate through:
  - `h/2` possible horizontal cuts (splitting by height)
  - `w/2` possible vertical cuts (splitting by width)
  - Total iterations per subproblem: `O(h + w)`, which in worst case is `O(m + n)`

The space complexity is `O(m × n)`, which accounts for:
- The memoization cache storing results for at most `m × n` unique `(height, width)` pairs
- The dictionary `d` storing the prices, which takes `O(p)` space but `p ≤ m × n`
- The recursion stack depth, which is at most `O(m + n)` in the worst case

Therefore, the overall space complexity is dominated by the cache storage of `O(m × n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Inefficient Loop Boundaries Leading to Redundant Calculations**

**The Pitfall:**
A common mistake is iterating through all possible cut positions instead of just half:

```python
# Inefficient approach - causes redundant calculations
for cut_height in range(1, height):  # Goes from 1 to height-1
    max_profit = max(max_profit, dfs(cut_height, width) + dfs(height - cut_height, width))
```

This creates duplicate work because cutting at position `i` or position `height - i` produces the same two pieces. For example, cutting a height-6 piece at position 2 or position 4 both create pieces of height 2 and 4.

**The Solution:**
Only iterate to the midpoint:
```python
for cut_height in range(1, height // 2 + 1):
    max_profit = max(max_profit, dfs(cut_height, width) + dfs(height - cut_height, width))
```

### 2. **Incorrect Price Lookup Data Structure**

**The Pitfall:**
Using a simple dictionary with tuple keys might seem intuitive but can lead to verbose code:

```python
# Less optimal approach
price_dict = {}
for h, w, p in prices:
    price_dict[(h, w)] = p

# Then in dfs function:
max_profit = price_dict.get((height, width), 0)
```

**The Solution:**
Use nested dictionaries for cleaner, more readable lookups:
```python
price_map = defaultdict(dict)
for height, width, price in prices:
    price_map[height][width] = price

# In dfs function:
max_profit = price_map[height].get(width, 0)
```

### 3. **Forgetting to Consider Not Cutting At All**

**The Pitfall:**
Starting with `max_profit = 0` and only considering cuts, forgetting that the optimal solution might be to sell the piece as-is:

```python
def dfs(height, width):
    max_profit = 0  # Wrong! Missing the direct sale option
  
    # Only considering cuts
    for cut_height in range(1, height // 2 + 1):
        max_profit = max(max_profit, dfs(cut_height, width) + dfs(height - cut_height, width))
    # ...
```

**The Solution:**
Always initialize with the direct sale price (if available):
```python
def dfs(height, width):
    # Start with the price of selling this piece as-is
    max_profit = price_map[height].get(width, 0)
  
    # Then consider cuts...
```

### 4. **Missing or Incorrect Memoization**

**The Pitfall:**
Implementing manual memoization incorrectly or forgetting it entirely:

```python
# Without memoization - exponential time complexity!
def dfs(height, width):
    # recursive calls without caching
  
# Or incorrect manual memoization
memo = {}
def dfs(height, width):
    if (height, width) in memo:
        return memo[(height, width)]
    # ... calculation ...
    # Forgetting to store result: memo[(height, width)] = result
    return result  # Missing memo storage!
```

**The Solution:**
Use Python's `@cache` decorator for automatic, correct memoization:
```python
from functools import cache

@cache
def dfs(height, width):
    # Automatically memoized
```

### 5. **Off-by-One Errors in Loop Ranges**

**The Pitfall:**
Using incorrect range boundaries that either miss valid cuts or create invalid pieces:

```python
# Wrong - misses the case where we cut exactly in half
for cut_height in range(1, height // 2):  # Should be height // 2 + 1
  
# Wrong - creates a piece with 0 height
for cut_height in range(0, height // 2 + 1):  # Should start from 1
```

**The Solution:**
Ensure cuts start from 1 (minimum piece size) and go up to and including the midpoint:
```python
for cut_height in range(1, height // 2 + 1):
    # Correctly handles all unique partitions
```

