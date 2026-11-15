# 123. Best Time to Buy and Sell Stock III

> Source: https://algo.monster/liteproblems/123

## Problem Statement

You are given an array `prices` where `prices[i]` represents the price of a stock on day `i`.

Your goal is to find the maximum profit you can achieve by completing **at most two transactions**. A transaction consists of buying and then selling one share of the stock.

**Important constraints:**
- You can complete at most 2 transactions (buy-sell pairs)
- You cannot hold multiple stocks at the same time - you must sell your current stock before buying again
- You cannot sell a stock before buying it

The solution uses dynamic programming with four state variables:
- `f1`: Maximum profit after completing the first buy operation
- `f2`: Maximum profit after completing the first sell operation  
- `f3`: Maximum profit after completing the second buy operation
- `f4`: Maximum profit after completing the second sell operation

For each day's price, the code updates these states:
- `f1 = max(f1, -price)`: Either keep the previous buy state or buy at today's price
- `f2 = max(f2, f1 + price)`: Either keep the previous first-sell state or sell the first stock today
- `f3 = max(f3, f2 - price)`: Either keep the previous second-buy state or buy the second stock today using profit from first transaction
- `f4 = max(f4, f3 + price)`: Either keep the previous second-sell state or sell the second stock today

The final answer is `f4`, which represents the maximum profit after completing at most two transactions.

## Intuition

The key insight is that with at most two transactions, we need to track the best profit at each possible stage of trading. Since we can't hold multiple stocks simultaneously, our trading follows a strict sequence: buy → sell → buy → sell.

At any point in time, we can be in one of four states:
1. After first buy (holding first stock)
2. After first sell (completed one transaction, holding cash)
3. After second buy (holding second stock)
4. After second sell (completed two transactions)

The challenge is determining when to make each move to maximize profit. For each day, we need to decide: should we maintain our current state or transition to the next state?

This naturally leads to a dynamic programming approach where we track the maximum profit achievable at each state. The transitions are:
- To buy first stock: We spend `price`, so profit becomes `-price`
- To sell first stock: We gain `price` added to our profit from buying
- To buy second stock: We spend `price` from our first transaction's profit
- To sell second stock: We gain `price` added to our second buy profit

The beauty of this approach is that we don't need to explicitly track which days we buy and sell. Instead, we just maintain the best possible profit at each state as we process each day. If buying or selling on a particular day doesn't improve our profit, we simply keep the previous state's value.

By considering all prices sequentially and updating our four states optimally, we ensure we capture the maximum profit possible with at most two transactions. The final answer is `f4` because it represents the state after completing all possible transactions.

## Solution Approach

The solution implements a state-based dynamic programming approach with four variables representing different trading states.

**Initialization:**
We start by initializing the four state variables:
- `f1 = -prices[0]`: After buying the first stock on day 0, our profit is negative (we spent money)
- `f2 = 0`: We haven't sold anything yet, so profit is 0
- `f3 = -prices[0]`: If we were to buy a second stock on day 0 (hypothetically after a zero-profit first transaction)
- `f4 = 0`: No second sale yet, profit is 0

**State Transitions:**
For each subsequent day (starting from day 1), we update all four states:

1. **First Buy (`f1`):** 
   - `f1 = max(f1, -price)`
   - Either keep the previous buy state (we already bought earlier at a better price) or buy today (starting fresh with `-price` profit)

2. **First Sell (`f2`):**
   - `f2 = max(f2, f1 + price)`
   - Either keep the previous sell state (we already sold at a better price) or sell today (adding today's price to our first buy profit)

3. **Second Buy (`f3`):**
   - `f3 = max(f3, f2 - price)`
   - Either keep the previous second buy state or buy today using profits from the first transaction (`f2 - price`)

4. **Second Sell (`f4`):**
   - `f4 = max(f4, f3 + price)`
   - Either keep the previous second sell state or sell today (adding today's price to our second buy profit)

**Why This Works:**
The algorithm ensures that at each step, we maintain the maximum possible profit for each state. The `max` operations guarantee we only transition to a new state if it's profitable. If completing fewer than two transactions yields better profit, the algorithm naturally handles this - `f4` will simply equal `f2` (the profit from one transaction).

**Time and Space Complexity:**
- Time: `O(n)` where n is the length of the prices array (single pass)
- Space: `O(1)` using only four variables regardless of input size

The final answer `f4` represents the maximum profit achievable with at most two complete transactions.

## Example Walkthrough

Let's walk through the algorithm with `prices = [3, 2, 6, 5, 0, 3]`. Within each day, the four updates run in order, so each line uses the values already updated on the lines above it.

**Initial Setup (Day 0, price = 3):**
- `f1 = -3` (bought first stock at price 3)
- `f2 = 0` (haven't sold yet)
- `f3 = -3` (hypothetical second buy)
- `f4 = 0` (haven't completed second transaction)

**Day 1 (price = 2):**
- `f1 = max(-3, -2) = -2` (better to buy at price 2)
- `f2 = max(0, -2 + 2) = 0` (uses the `f1` just updated; selling today would give 0)
- `f3 = max(-3, 0 - 2) = -2` (better to buy second stock now)
- `f4 = max(0, -2 + 2) = 0` (uses the `f3` just updated; no gain yet)

**Day 2 (price = 6):**
- `f1 = max(-2, -6) = -2` (keep previous buy at price 2)
- `f2 = max(0, -2 + 6) = 4` (sell first stock for profit of 4)
- `f3 = max(-2, 4 - 6) = -2` (keep previous second buy)
- `f4 = max(0, -2 + 6) = 4` (could sell second stock for profit of 4)

**Day 3 (price = 5):**
- `f1 = max(-2, -5) = -2` (keep buy at price 2)
- `f2 = max(4, -2 + 5) = 4` (previous sell at 6 was better)
- `f3 = max(-2, 4 - 5) = -1` (buy second stock using first transaction's profit)
- `f4 = max(4, -1 + 5) = 4` (keep previous state)

**Day 4 (price = 0):**
- `f1 = max(-2, -0) = 0` (buying at 0 is better)
- `f2 = max(4, 0 + 0) = 4` (keep previous sell)
- `f3 = max(-1, 4 - 0) = 4` (buy second stock at price 0 using profit of 4)
- `f4 = max(4, 4 + 0) = 4` (keep previous state)

**Day 5 (price = 3):**
- `f1 = max(0, -3) = 0` (keep buy at price 0)
- `f2 = max(4, 0 + 3) = 4` (keep previous sell)
- `f3 = max(4, 4 - 3) = 4` (keep second buy at price 0)
- `f4 = max(4, 4 + 3) = 7` (sell second stock for total profit of 7)

**Final Result: 7**

The optimal strategy was:
- First transaction: Buy at price 2 (day 1), sell at price 6 (day 2) → profit = 4
- Second transaction: Buy at price 0 (day 4), sell at price 3 (day 5) → profit = 3
- Total profit = 4 + 3 = 7

## Python Solution

```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        """
        Calculate maximum profit from at most two stock transactions.
        Uses dynamic programming with four states to track optimal profits.
      
        Args:
            prices: List of daily stock prices
          
        Returns:
            Maximum profit achievable with at most two transactions
        """
        # Initialize four states for tracking profits
        # first_buy: Max profit after first stock purchase
        # first_sell: Max profit after first stock sale
        # second_buy: Max profit after second stock purchase  
        # second_sell: Max profit after second stock sale
        first_buy = -prices[0]
        first_sell = 0
        second_buy = -prices[0]
        second_sell = 0
      
        # Iterate through remaining prices starting from day 2
        for price in prices[1:]:
            # Updating in order reuses today's values; a same-day buy+sell adds 0, so this is safe
            # Max profit after first buy: either keep previous state or buy at current price
            first_buy = max(first_buy, -price)
          
            # Max profit after first sell: either keep previous state or sell at current price
            first_sell = max(first_sell, first_buy + price)
          
            # Max profit after second buy: either keep previous state or buy again after first sell
            second_buy = max(second_buy, first_sell - price)
          
            # Max profit after second sell: either keep previous state or sell second stock
            second_sell = max(second_sell, second_buy + price)
      
        # Return maximum profit after at most two complete transactions
        return second_sell
```

## Time and Space Complexity

**Time Complexity:** `O(n)`, where `n` is the length of the `prices` array. The algorithm iterates through the prices array once using a single for loop that runs from index 1 to the end of the array. Each iteration performs a constant number of operations (four `max` comparisons and arithmetic operations), resulting in linear time complexity.

**Space Complexity:** `O(1)`. The algorithm uses only four variables (`f1`, `f2`, `f3`, `f4`) to track the states of buying and selling stocks, regardless of the input size. No additional data structures that scale with input size are used, making the space complexity constant.

## Common Pitfalls

### 1. **Initializing the Buy States to 0**
A common mistake is starting every state at `0`, as if nothing had happened yet:

**Problematic Code:**
```python
first_buy = first_sell = second_buy = second_sell = 0  # Wrong for the buy states
for price in prices:
    first_buy = max(first_buy, -price)
    first_sell = max(first_sell, first_buy + price)
    second_buy = max(second_buy, first_sell - price)
    second_sell = max(second_sell, second_buy + price)
```

**Why This Is Wrong:**
`first_buy = 0` means "holding a share that cost nothing." Since `-price` is never above `0`, `first_buy` stays at `0`, and selling it gives `first_sell = price` for free. For `prices = [3, 2]`, this returns `3` (a free share sold at 3 on day 0) instead of `0`.

**Solution:**
Start the buy states at `-prices[0]` (or `-inf`), so a share must be paid for before it can be sold:
```python
first_buy, first_sell = -prices[0], 0
second_buy, second_sell = -prices[0], 0
```

**Note on update order:** The reference solution updates the four states in order, so `first_sell` uses the `first_buy` value from the same day. This is safe. Buying and selling on the same day changes profit by `0`, so it never creates a better answer than a real strategy. Updating in reverse order or through temporary variables gives the same result.

### 2. **Misunderstanding the First Buy State**
Another common mistake is initializing or updating `first_buy` incorrectly by thinking it should account for previous profits.

**Incorrect:**
```python
first_buy = max(first_buy, prev_profit - price)  # Wrong!
```

**Correct:**
```python
first_buy = max(first_buy, -price)  # Correct - first buy starts from 0 profit
```

The first buy always starts from zero profit since it's the beginning of our trading sequence. Only the second buy (`second_buy`) should consider profits from previous transactions.

### 3. **Edge Case: Single Element Array**
Failing to handle arrays with only one price can cause index out of bounds errors if not using the initialization approach shown.

**Explicit Check (also correct):**
```python
if len(prices) < 2:
    return 0  # Need at least 2 days to make a transaction
```

**Without the Check:**
The provided solution naturally handles this by initializing all states with `prices[0]` and then iterating from index 1, which works even for single-element arrays (the loop simply doesn't execute, returning 0).

