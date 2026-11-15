# 188. Best Time to Buy and Sell Stock IV

> Source: https://algo.monster/liteproblems/188

## Problem Statement

You are given an array `prices` where `prices[i]` represents the stock price on day `i`. You also have a limit `k` on the maximum number of transactions you can make.

Your goal is to find the maximum profit you can achieve by buying and selling the stock, with the following constraints:

- You can complete at most `k` transactions (buying and then selling counts as one transaction)
- You cannot hold multiple stocks at once - you must sell your current stock before buying again
- You cannot buy and sell on the same day

The solution uses dynamic programming with memoization. The function `dfs(i, j, k)` tracks three parameters:
- `i`: the current day index
- `j`: the number of remaining transactions allowed
- `k`: whether currently holding stock (0 = not holding, 1 = holding)

At each day, there are three possible decisions:
1. Do nothing and move to the next day
2. If holding stock (`k = 1`), sell it for `prices[i]` profit and decrement available transactions
3. If not holding stock (`k = 0`) and have transactions remaining (`j > 0`), buy stock for `-prices[i]` cost

The function recursively explores all valid paths and returns the maximum profit achievable. The `@cache` decorator prevents recalculating the same state multiple times, improving efficiency.

The answer is found by calling `dfs(0, k, 0)`, which starts from day 0 with `k` transactions available and no stock held initially.

## Intuition

The key insight is that at any given moment, we need to track three pieces of information: which day we're on, how many transactions we have left, and whether we're currently holding stock or not.

Think about the problem as a series of states. On each day, we're in one of two states: either holding stock or not holding stock. When we're holding stock, we can only sell. When we're not holding stock, we can only buy (if we have transactions remaining).

Why do we need to track transactions? Because each complete buy-sell cycle counts as one transaction, and we're limited to `k` transactions total. We decrement the transaction count when a cycle starts (at the buying point), so a buy is only allowed while a transaction is still available.

The recursive approach naturally emerges when we consider that at each day, we make a decision that leads to a new state. From day `i`, we want to know: "What's the maximum profit I can get from here onwards?" This depends on:
- Our current position (holding stock or not)
- How many transactions we can still make
- The future decisions we'll make

The three choices at each step represent our decision tree:
1. **Skip this day** - Sometimes the best action is no action. We simply move to the next day with the same state.
2. **Sell (if holding)** - If `k = 1` (holding stock), we can sell for `prices[i]` profit. After selling, we're back to not holding stock (`k = 0`) and we've used up one transaction.
3. **Buy (if not holding and have transactions left)** - If `k = 0` and `j > 0`, we can buy stock for `prices[i]` cost (hence the negative). After buying, we're holding stock (`k = 1`).

The beauty of this approach is that it naturally handles the constraint that we can't buy and sell simultaneously - the state `k` ensures we're either in "can buy" mode or "can sell" mode, never both.

By exploring all possible paths through these decisions and taking the maximum, we find the optimal trading strategy. The memoization (`@cache`) is crucial because many different sequences of decisions can lead to the same state (same day, same transactions left, same holding status), so we avoid recalculating these overlapping subproblems.

## Solution Approach

The solution implements a top-down dynamic programming approach using memoization. Let's walk through the implementation step by step:

**Function Definition:**
The core function `dfs(i, j, k)` represents:
- `i`: current day index (0 to n-1)
- `j`: number of transactions remaining
- `k`: stock holding state (0 = not holding, 1 = holding)

**Base Case:**
```python
if i >= len(prices):
    return 0
```
When we've processed all days, no more profit can be made, so return 0.

**State Transitions:**

1. **Default Action - Do Nothing:**
   ```python
   ans = dfs(i + 1, j, k)
   ```
   We always consider the option of skipping the current day and maintaining our current state.

2. **Selling Stock (when k = 1):**
   ```python
   if k:
       ans = max(ans, prices[i] + dfs(i + 1, j, 0))
   ```
   - If currently holding stock (`k = 1`), we can sell it
   - Selling gives us `prices[i]` profit
   - After selling, we transition to not holding (`k = 0`)
   - Transactions remain the same (`j`) because this transaction was already counted when we bought

3. **Buying Stock (when k = 0 and j > 0):**
   ```python
   elif j:
       ans = max(ans, -prices[i] + dfs(i + 1, j - 1, 1))
   ```
   - If not holding stock (`k = 0`) and have transactions available (`j > 0`)
   - Buying costs us `prices[i]` (hence negative)
   - After buying, we transition to holding (`k = 1`)
   - We decrement available transactions (`j - 1`) as we're starting a new transaction

**Key Design Decisions:**

- **Transaction Counting:** The transaction is decremented when buying (not selling). This simplifies tracking since each buy operation initiates a new transaction that will be completed upon selling.

- **Memoization with @cache:** The decorator automatically caches results for each unique `(i, j, k)` tuple, preventing redundant calculations. This is crucial as many different trading sequences can lead to the same state.

- **State Space:** The total state space is `O(n × k × 2)` where:
  - `n` is the number of days
  - `k` is the maximum transactions
  - `2` represents the two holding states

**Time and Space Complexity:**
- Time: `O(n × k)` as we compute each state at most once
- Space: `O(n × k)` for the memoization cache

The solution starts with `dfs(0, k, 0)` - beginning at day 0, with `k` transactions available, and not holding any stock. The function explores all valid trading paths and returns the maximum achievable profit.

## Example Walkthrough

Let's walk through a small example with `prices = [3, 2, 6, 5, 0, 3]` and `k = 2` (maximum 2 transactions).

We start with `dfs(0, 2, 0)` - day 0, 2 transactions available, not holding stock.

**Day 0 (price = 3):** Not holding stock, 2 transactions available
- Option 1: Skip day → `dfs(1, 2, 0)`
- Option 2: Buy stock → `-3 + dfs(1, 1, 1)` (spend 3, now holding, 1 transaction left)
- We explore both paths

**Following the buy path from Day 0:**
**Day 1 (price = 2):** Holding stock, 1 transaction available
- Option 1: Skip day → `dfs(2, 1, 1)`
- Option 2: Sell stock → `2 + dfs(2, 1, 0)` (gain 2, not holding)
- Selling would give us -1 profit so far (bought at 3, sold at 2)

**Better path - Skip Day 0:**
**Day 1 (price = 2):** Not holding stock, 2 transactions available
- Option 1: Skip day → `dfs(2, 2, 0)`
- Option 2: Buy stock → `-2 + dfs(2, 1, 1)` (spend 2, now holding, 1 transaction left)

**Following the buy at Day 1 path:**
**Day 2 (price = 6):** Holding stock, 1 transaction available
- Option 1: Skip day → `dfs(3, 1, 1)`
- Option 2: Sell stock → `6 + dfs(3, 1, 0)` (gain 6, not holding)
- Selling gives us 4 profit (bought at 2, sold at 6)

**After selling at Day 2:**
**Day 3 (price = 5):** Not holding stock, 1 transaction available
- Option 1: Skip day → `dfs(4, 1, 0)`
- Option 2: Buy stock → `-5 + dfs(4, 0, 1)` (spend 5, now holding, 0 transactions left)

**Day 4 (price = 0):** Following the skip path, not holding, 1 transaction available
- Option 1: Skip day → `dfs(5, 1, 0)`
- Option 2: Buy stock → `0 + dfs(5, 0, 1)` (buy at 0, now holding, 0 transactions left)

**Day 5 (price = 3):** If we bought at Day 4, holding stock, 0 transactions available
- Option 1: Skip day → `dfs(6, 0, 1)` → returns 0 (base case)
- Option 2: Sell stock → `3 + dfs(6, 0, 0)` → returns 3

**Optimal Path:**
1. Buy at day 1 (price = 2) → -2 profit, 1 transaction used
2. Sell at day 2 (price = 6) → +6 profit, total = 4
3. Buy at day 4 (price = 0) → -0 profit, 2nd transaction used
4. Sell at day 5 (price = 3) → +3 profit, total = 7

The function explores all possible paths through memoized recursion and returns the maximum profit of 7. The memoization ensures that when we encounter the same state (same day, transactions, and holding status) through different paths, we don't recalculate it.

## Python Solution

```python
class Solution:
    def maxProfit(self, k: int, prices: List[int]) -> int:
        """
        Find maximum profit from at most k stock transactions.
      
        Args:
            k: Maximum number of transactions allowed
            prices: List of stock prices
          
        Returns:
            Maximum profit achievable
        """
        from functools import cache
      
        @cache
        def dfs(day_index: int, transactions_left: int, holding_stock: int) -> int:
            """
            Dynamic programming helper to calculate maximum profit.
          
            Args:
                day_index: Current day index in prices array
                transactions_left: Number of buy-sell transactions remaining
                holding_stock: 1 if currently holding stock, 0 otherwise
              
            Returns:
                Maximum profit from current state onwards
            """
            # Base case: no more days left
            if day_index >= len(prices):
                return 0
          
            # Option 1: Do nothing today (skip to next day)
            max_profit = dfs(day_index + 1, transactions_left, holding_stock)
          
            if holding_stock:
                # Currently holding stock - can sell today
                max_profit = max(
                    max_profit, 
                    prices[day_index] + dfs(day_index + 1, transactions_left, 0)
                )
            elif transactions_left > 0:
                # Not holding stock and have transactions left - can buy today
                # Buying uses one transaction
                max_profit = max(
                    max_profit, 
                    -prices[day_index] + dfs(day_index + 1, transactions_left - 1, 1)
                )
          
            return max_profit
      
        # Start from day 0, with k transactions available, not holding any stock
        return dfs(0, k, 0)
```

## Time and Space Complexity

The time complexity is `O(n × k)`, where `n` is the length of the prices array and `k` is the maximum number of transactions allowed.

This is because the recursive function `dfs` uses memoization with `@cache`. The state space is defined by three parameters:
- `i`: ranges from `0` to `n-1` (n possible values)
- `j`: ranges from `0` to `k` (k+1 possible values) 
- `k` (the third parameter): can only be `0` or `1` (2 possible values)

Therefore, the total number of unique states is `n × (k+1) × 2 = O(n × k)`. Each state is computed only once due to memoization, and each computation takes `O(1)` time (excluding recursive calls).

The space complexity is `O(n × k)` as well. This accounts for:
- The memoization cache storing all possible states: `O(n × k)`
- The recursion call stack depth, which can go up to `O(n)` in the worst case

Since `O(n × k) + O(n) = O(n × k)` when `k ≥ 1`, the overall space complexity is `O(n × k)`.

## Common Pitfalls

### 1. **Counting Each Transaction Twice**

A transaction is one buy plus one sell, so the counter must change exactly once per cycle. The bug is decrementing it at both ends.

**Incorrect approach:**
```python
if holding_stock:
    # Wrong: decrementing when selling...
    max_profit = max(max_profit,
                    prices[day_index] + dfs(day_index + 1, transactions_left - 1, 0))
elif transactions_left > 0:
    # ...and again when buying
    max_profit = max(max_profit,
                    -prices[day_index] + dfs(day_index + 1, transactions_left - 1, 1))
```

**Why this fails:** Every completed buy-sell cycle consumes two units, so only about `k/2` transactions are allowed. For `prices = [3, 2, 6, 5, 0, 3]` and `k = 2`, this version can complete just one transaction and returns 4 instead of 7.

**Correct approach:** Decrement once per cycle. The reference solution decrements when buying (a buy is only allowed while `transactions_left > 0`):
```python
if holding_stock:
    # Selling completes the transaction, no decrement needed
    max_profit = max(max_profit,
                    prices[day_index] + dfs(day_index + 1, transactions_left, 0))
elif transactions_left > 0:
    # Buying starts a new transaction, decrement here
    max_profit = max(max_profit,
                    -prices[day_index] + dfs(day_index + 1, transactions_left - 1, 1))
```

Decrementing only at the sell (and still checking `transactions_left > 0` before a buy) is equally correct; what matters is doing it in exactly one place.

### 2. **Memory Optimization for Large k Values**

When k is very large (k ≥ n/2 where n is the number of days), the problem essentially becomes unlimited transactions, but the current solution still tracks all k states unnecessarily.

**Issue:** This wastes memory and computation for states that will never be reached.

**Solution:** Add an optimization check:
```python
def maxProfit(self, k: int, prices: List[int]) -> int:
    if not prices:
        return 0
  
    n = len(prices)
    # If k >= n/2, we can make as many transactions as we want
    if k >= n // 2:
        # Simplified unlimited transactions solution
        profit = 0
        for i in range(1, n):
            if prices[i] > prices[i-1]:
                profit += prices[i] - prices[i-1]
        return profit
  
    # Original DP solution for limited k
    @cache
    def dfs(day_index, transactions_left, holding_stock):
        # ... rest of the implementation
```

### 3. **Edge Cases**

The reference solution already handles these: an empty `prices` array hits the base case immediately and returns 0, and with `k = 0` the buy branch never runs, so the result is 0. An explicit check only makes this visible:

**Optional early return:**
```python
def maxProfit(self, k: int, prices: List[int]) -> int:
    # Should check for empty prices first
    if not prices or k == 0:
        return 0
  
    @cache
    def dfs(day_index, transactions_left, holding_stock):
        # ... implementation
```

This check is not needed for correctness, but it skips building the memoized recursion when the answer is obviously 0.

### 4. **State Parameter Order Confusion**

Mixing up the order of parameters in recursive calls is a subtle but critical error:

**Incorrect:**
```python
# Wrong parameter order in recursive call
max_profit = max(max_profit, 
                prices[day_index] + dfs(day_index + 1, 0, transactions_left))
# Should be: dfs(day_index + 1, transactions_left, 0)
```

**Solution:** Always maintain consistent parameter order: `(day_index, transactions_left, holding_stock)`. Using descriptive variable names instead of single letters (i, j, k) helps prevent this mistake.

