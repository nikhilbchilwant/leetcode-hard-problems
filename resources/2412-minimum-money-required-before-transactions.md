# 2412. Minimum Money Required Before Transactions

> Source: https://algo.monster/liteproblems/2412

## Problem Statement

You have a list of transactions that you need to complete. Each transaction is represented as `[cost, cashback]`, where:
- `cost` is the amount of money you need to have to perform the transaction
- `cashback` is the amount of money you receive back after completing the transaction

The key rules are:
1. Each transaction must be completed exactly once
2. To complete a transaction with cost `cost_i`, you must have at least `cost_i` money available
3. After completing a transaction, your money changes from `money` to `money - cost + cashback`
4. You can complete the transactions in any order you choose

The problem asks: What is the minimum initial amount of money you need to guarantee that you can complete ALL transactions, regardless of the order you choose to complete them?

For example, if you have a transaction `[10, 3]`:
- You need at least 10 money to start this transaction
- After completing it, you'll have `money - 10 + 3 = money - 7` (you lose 7 money overall)

The challenge is finding the minimum starting money that works for ANY possible ordering of the transactions, since we want to ensure we can complete all transactions no matter which order we pick.

## Intuition

Let's think about what happens with our money as we complete transactions. Each transaction `[cost, cashback]` has a net effect of `cashback - cost` on our money. If `cost > cashback`, we lose money overall (negative profit). If `cashback > cost`, we gain money (positive profit).

The key insight is that we want to minimize our initial money requirement. To do this optimally, we should complete all losing transactions (where `cost > cashback`) first, then complete the profitable ones. Why? Because profitable transactions increase our money, making it easier to afford subsequent transactions.

So first, let's calculate the total loss from all losing transactions: `s = sum(max(0, cost - cashback))`. This represents how much money we'll lose from all the bad transactions.

Now, which transaction should be the last one? This is crucial because the last transaction determines our peak money requirement. We need to consider two cases:

1. **If the last transaction is a losing one `(cost > cashback)`:** We need enough money to cover all previous losses plus this transaction's cost. But wait - this transaction's loss is already included in `s`, so we actually need `s - (cost - cashback) + cost = s + cashback`.

2. **If the last transaction is a profitable one `(cost <= cashback)`:** We need enough money to cover all previous losses (the full `s`) plus this transaction's cost, giving us `s + cost`.

Since we don't know which order is actually worst-case, we try each transaction as the potential last one and take the maximum requirement. This ensures we have enough money for any possible ordering.

The formula `max(ans, s + b)` when `a > b` and `max(ans, s + a)` otherwise captures exactly this logic - finding the worst-case initial money needed across all possible final transactions.

**Learn more about [Greedy](/problems/greedy_intro) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

The implementation follows a [greedy](/problems/greedy_intro) strategy based on our intuition about handling losing transactions first.

**Step 1: Calculate total losses**
```python
s = sum(max(0, a - b) for a, b in transactions)
```
We iterate through all transactions and accumulate the net loss from each losing transaction. For a transaction `[a, b]`:
- If `a > b` (losing transaction), we add `a - b` to our sum
- If `a <= b` (profitable transaction), we add `0` (using `max(0, a - b)`)

This gives us `s`, the total amount we'll lose from all losing transactions.

**Step 2: Find the worst-case scenario**
```python
ans = 0
for a, b in transactions:
    if a > b:
        ans = max(ans, s + b)
    else:
        ans = max(ans, s + a)
```

We consider each transaction as potentially being the last one executed:

- **When `a > b` (losing transaction as last):** 
  - The loss `a - b` from this transaction is already included in `s`
  - To execute this as the last transaction, we need `s - (a - b) + a = s + b` money
  - We update answer with `max(ans, s + b)`

- **When `a <= b` (profitable transaction as last):**
  - This transaction wasn't included in `s` since it's profitable
  - To execute this as the last transaction after all losses, we need `s + a` money
  - We update answer with `max(ans, s + a)`

The maximum value across all these scenarios gives us the minimum initial money needed to guarantee we can complete all transactions in some order.

**Time Complexity:** `O(n)` where n is the number of transactions (two passes through the array)  
**Space Complexity:** `O(1)` as we only use a few variables

## Example Walkthrough

Let's walk through a concrete example with transactions: `[[3, 0], [2, 1], [5, 8]]`

**Step 1: Calculate total losses**

First, we identify which transactions lose money:
- Transaction `[3, 0]`: cost=3, cashback=0 → loses 3-0 = 3 money
- Transaction `[2, 1]`: cost=2, cashback=1 → loses 2-1 = 1 money  
- Transaction `[5, 8]`: cost=5, cashback=8 → gains 8-5 = 3 money (profitable!)

Total losses `s = 3 + 1 + 0 = 4`

**Step 2: Consider each transaction as potentially the last one**

Now we check how much initial money we'd need if each transaction were executed last:

1. **If `[3, 0]` is last (losing transaction):**
   - We need to handle the other transactions first, losing money from `[2, 1]`
   - The loss from `[3, 0]` is already in `s=4`, so we need: `s + cashback = 4 + 0 = 4`
   - This means: start with 4 → after other transactions we'd have at least 3 → can afford cost of 3

2. **If `[2, 1]` is last (losing transaction):**
   - The loss from `[2, 1]` is already in `s=4`, so we need: `s + cashback = 4 + 1 = 5`
   - This means: start with 5 → after other transactions we'd have at least 2 → can afford cost of 2

3. **If `[5, 8]` is last (profitable transaction):**
   - This wasn't included in `s`, so we need: `s + cost = 4 + 5 = 9`
   - This means: start with 9 → lose 4 from the losing transactions → have 5 left → can afford cost of 5

**Step 3: Take the maximum**

The worst case is `max(4, 5, 9) = 9`

**Verification:** Let's verify that 9 works for any order:
- Order 1: `[3,0]` → `[2,1]` → `[5,8]`: 9→6→5→10 ✓
- Order 2: `[2,1]` → `[3,0]` → `[5,8]`: 9→8→5→10 ✓
- Order 3: `[5,8]` → `[3,0]` → `[2,1]`: 9→12→9→8 ✓
- Order 4: `[5,8]` → `[2,1]` → `[3,0]`: 9→12→11→8 ✓
- Order 5: `[3,0]` → `[5,8]` → `[2,1]`: 9→6→9→8 ✓
- Order 6: `[2,1]` → `[5,8]` → `[3,0]`: 9→8→11→8 ✓

All orderings work with initial money of 9, confirming our answer!

## Python Solution

```python
class Solution:
    def minimumMoney(self, transactions: List[List[int]]) -> int:
        # Calculate total net loss from all transactions where cost exceeds cashback
        # This represents the minimum amount we'll definitely lose
        total_net_loss = sum(max(0, cost - cashback) for cost, cashback in transactions)
      
        # Initialize the minimum money needed
        min_money_needed = 0
      
        # Check each transaction as if it were the last one performed
        # This ensures we have enough money for any ordering of transactions
        for cost, cashback in transactions:
            if cost > cashback:
                # For losing transactions: need total losses plus this transaction's cashback
                # (since we get cashback after paying, we need it available before)
                min_money_needed = max(min_money_needed, total_net_loss + cashback)
            else:
                # For gaining/neutral transactions: need total losses plus this transaction's cost
                # (since we gain/break even, we just need enough to start this transaction)
                min_money_needed = max(min_money_needed, total_net_loss + cost)
      
        return min_money_needed
```

## Time and Space Complexity

The time complexity is `O(n)`, where `n` is the number of transactions. The algorithm iterates through the transactions list twice - once in the list comprehension to calculate the sum `s` with `sum(max(0, a - b) for a, b in transactions)`, and once in the for loop to find the maximum value. Each iteration performs constant-time operations (`max`, subtraction, and comparison), resulting in `O(n) + O(n) = O(n)` overall time complexity.

The space complexity is `O(1)`. The algorithm only uses a fixed amount of extra space for variables `s` and `ans`, regardless of the input size. The list comprehension in the sum function is implemented as a generator expression, which doesn't create an intermediate list but instead computes values on-the-fly, maintaining constant space usage.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Misunderstanding the Problem Constraint
**The Mistake:** Many people initially think they need to find the optimal order of transactions that minimizes the required starting money. They might try complex ordering strategies or dynamic programming approaches.

**Why It's Wrong:** The problem asks for the minimum money needed to guarantee completion of ALL transactions for ANY order. We're not looking for the best order; we're looking for an amount that works regardless of order choice.

**Correct Understanding:** We need to find the worst-case scenario across all possible orderings and ensure we have enough money for that worst case.

### Pitfall 2: Incorrectly Calculating Required Money for Each Transaction
**The Mistake:** When checking a transaction as the last one, using the wrong formula:
```python
# WRONG: Just using the cost of the transaction
if cost > cashback:
    min_money_needed = max(min_money_needed, total_net_loss + cost)  # Wrong!
```

**Why It's Wrong:** For losing transactions, when we place them last, their loss is already counted in `total_net_loss`. If we have `total_net_loss` money and execute all losing transactions, we end with 0. To then execute a transaction `[cost, cashback]` where `cost > cashback`, we need exactly `cashback` money (since after all losses we have `total_net_loss - total_net_loss = 0`, and we need `cost` to start, but the loss `cost - cashback` was already accounted for).

**Correct Approach:**
```python
if cost > cashback:
    # The loss (cost - cashback) is already in total_net_loss
    # So we need: total_net_loss - (cost - cashback) + cost = total_net_loss + cashback
    min_money_needed = max(min_money_needed, total_net_loss + cashback)
else:
    # Profitable transaction not in total_net_loss, need full cost on top
    min_money_needed = max(min_money_needed, total_net_loss + cost)
```

### Pitfall 3: Not Considering All Transactions as Potential Last Transactions
**The Mistake:** Only checking losing transactions or only checking profitable transactions as the last one:
```python
# WRONG: Only considering losing transactions
for cost, cashback in transactions:
    if cost > cashback:
        min_money_needed = max(min_money_needed, total_net_loss + cashback)
# Missing the check for profitable transactions!
```

**Why It's Wrong:** Both losing and profitable transactions could require the maximum initial money when placed last. A profitable transaction with very high cost might require more initial money than any losing transaction.

**Example:** 
- Transactions: `[[10, 5], [100, 101]]`
- Total loss: `10 - 5 = 5`
- Checking `[10, 5]` last: need `5 + 5 = 10`
- Checking `[100, 101]` last: need `5 + 100 = 105` ← This is the maximum!

**Correct Approach:** Always check every transaction as a potential last transaction, regardless of whether it's losing or profitable.

