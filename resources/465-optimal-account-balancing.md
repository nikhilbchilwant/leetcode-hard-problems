# 465. Optimal Account Balancing

> Source: https://algo.monster/liteproblems/465

## Problem Statement

You have a group of people who have been lending money to each other, and now they need to settle all their debts. You're given a list of transactions where each transaction tells you: who paid, who received, and how much money was transferred.

Specifically, you receive an array `transactions` where each element `transactions[i] = [from_i, to_i, amount_i]` means:
- Person with ID `from_i` gave `amount_i` dollars to person with ID `to_i`

After all these transactions, some people might owe money while others are owed money. Your task is to find the **minimum number of new transactions** needed to completely settle all debts so that everyone's balance becomes zero.

For example, if person A owes $10 and person B is owed $10, they can settle with just 1 transaction (A pays B $10). But if there are multiple people with various debts and credits, finding the minimum number of transactions becomes more complex.

The key insight is that people with positive balances (those who are owed money) need to receive payments from people with negative balances (those who owe money). The challenge is to arrange these settlements in the most efficient way possible to minimize the total number of transactions.

## Intuition

Let's think about what happens after all the original transactions. Each person has a net balance - they either owe money (negative balance) or are owed money (positive balance). The key insight is that we only care about these final balances, not the original transactions.

For example, if Alice paid Bob $10, Bob paid Charlie $10, and Charlie paid Alice $10, everyone's net balance is zero - no further transactions needed! This tells us we should first calculate everyone's net balance.

Now, here's the crucial observation: when we settle debts, we're essentially trying to group people together such that the sum of their balances equals zero. Within each group, we can settle all debts with `(group_size - 1)` transactions. Why? Because in a group of `n` people whose balances sum to zero, we can pick one person as a "central bank" and have the other `n-1` people settle with them.

For instance, if we have three people with balances `[5, -2, -3]`, person 1 can receive $2 from person 2 and $3 from person 3, settling everything with just 2 transactions.

The challenge becomes: how do we optimally partition all people with non-zero balances into such groups? This is where the small constraints become crucial. With typically at most 12-15 people having non-zero balances, we can explore all possible subsets.

We use dynamic programming with bitmasks to avoid recalculating. For each subset of people (represented as a bitmask), we:
1. Check if their balances sum to zero
2. If yes, this subset can be settled with `(subset_size - 1)` transactions
3. We also consider splitting this subset into smaller zero-sum subsets to see if that requires fewer total transactions

The minimum across all these possibilities gives us our answer. This approach ensures we find the globally optimal way to group people for debt settlement.

## Solution Approach

The implementation follows our intuition of finding optimal groups of people whose balances sum to zero. Let's walk through the code step by step:

**Step 1: Calculate Net Balances**
```python
g = defaultdict(int)
for f, t, x in transactions:
    g[f] -= x  # person f loses x amount
    g[t] += x  # person t gains x amount
```
We use a dictionary to track each person's net balance. After processing all transactions, positive values mean the person is owed money, negative values mean they owe money.

**Step 2: Filter Non-Zero Balances**
```python
nums = [x for x in g.values() if x]
m = len(nums)
```
We only care about people with non-zero balances since those with zero balance are already settled. This significantly reduces our problem size.

**Step 3: Initialize DP Array**
```python
f = [inf] * (1 << m)
f[0] = 0
```
We create a DP array where `f[i]` represents the minimum number of transactions needed to settle debts for the subset of people represented by bitmask `i`. The empty set (0) requires 0 transactions.

**Step 4: Process Each Subset**
```python
for i in range(1, 1 << m):
    s = 0
    for j, x in enumerate(nums):
        if i >> j & 1:
            s += x
```
For each possible subset `i`, we first calculate the sum of balances. If bit `j` is set in `i`, we include `nums[j]` in our sum.

**Step 5: Handle Zero-Sum Subsets**
```python
if s == 0:
    f[i] = i.bit_count() - 1
```
If the subset sums to zero, we can settle it internally with `(size - 1)` transactions by having everyone settle with one central person.

**Step 6: Try Splitting the Subset**
```python
j = (i - 1) & i
while j > 0:
    f[i] = min(f[i], f[j] + f[i ^ j])
    j = (j - 1) & i
```
This clever bit manipulation iterates through all non-empty subsets `j` of `i`. For each split:
- `j` is one subset
- `i ^ j` is the complementary subset (elements in `i` but not in `j`)
- We check if splitting into these two groups gives fewer total transactions

The expression `(j - 1) & i` generates the next subset of `i` in decreasing order.

**Step 7: Return Result**
```python
return f[-1]
```
`f[-1]` or `f[(1 << m) - 1]` represents the subset containing all people with non-zero balances, which is our answer.

The time complexity is `O(3^n)` where `n` is the number of people with non-zero balances (due to iterating through all subsets and their subsets), and space complexity is `O(2^n)` for the DP array.

## Example Walkthrough

Let's walk through a concrete example to understand how the solution works.

**Given transactions:** `[[0,1,10], [1,2,5], [2,0,3]]`
- Person 0 gives $10 to Person 1
- Person 1 gives $5 to Person 2
- Person 2 gives $3 to Person 0

**Step 1: Calculate Net Balances**
- Person 0: -10 (gave) + 3 (received) = **-7** (owes $7)
- Person 1: +10 (received) - 5 (gave) = **+5** (is owed $5)
- Person 2: +5 (received) - 3 (gave) = **+2** (is owed $2)

So `nums = [-7, 5, 2]` (3 people with non-zero balances)

**Step 2: Initialize DP Array**
With 3 people, we have 2³ = 8 possible subsets.
`f = [0, inf, inf, inf, inf, inf, inf, inf]`

**Step 3: Process Each Subset**

Let's trace through key subsets (using binary representation):

- **Subset 001 (person 0 only):** Balance = -7
  - Not zero-sum, remains `f[1] = inf`

- **Subset 010 (person 1 only):** Balance = 5
  - Not zero-sum, remains `f[2] = inf`

- **Subset 011 (persons 0,1):** Balance = -7 + 5 = -2
  - Not zero-sum, remains `f[3] = inf`

- **Subset 100 (person 2 only):** Balance = 2
  - Not zero-sum, remains `f[4] = inf`

- **Subset 101 (persons 0,2):** Balance = -7 + 2 = -5
  - Not zero-sum, remains `f[5] = inf`

- **Subset 110 (persons 1,2):** Balance = 5 + 2 = 7
  - Not zero-sum, remains `f[6] = inf`

- **Subset 111 (all three):** Balance = -7 + 5 + 2 = 0 ✓
  - Zero-sum! Initial value: `f[7] = 3 - 1 = 2` transactions
  - Now check if splitting helps:
    - Split into {person 0} and {persons 1,2}: `f[1] + f[6] = inf + inf = inf` (no help)
    - Split into {person 1} and {persons 0,2}: `f[2] + f[5] = inf + inf = inf` (no help)
    - Split into {person 2} and {persons 0,1}: `f[4] + f[3] = inf + inf = inf` (no help)
    - Split into {persons 0,1} and {person 2}: `f[3] + f[4] = inf + inf = inf` (no help)
  - No split improves the result, so `f[7] = 2`

**Result:** The minimum number of transactions is **2**.

This makes sense! Person 0 owes $7 total. They can:
1. Pay $5 to Person 1 (settling Person 1's balance)
2. Pay $2 to Person 2 (settling Person 2's balance)

Both transactions together settle everyone's debt with just 2 transactions, which matches our algorithm's answer.

## Python Solution

```python
class Solution:
    def minTransfers(self, transactions: List[List[int]]) -> int:
        # Build a balance map for each person
        # Negative balance means they owe money, positive means they are owed money
        balance_map = defaultdict(int)
        for from_person, to_person, amount in transactions:
            balance_map[from_person] -= amount  # Person giving money
            balance_map[to_person] += amount    # Person receiving money

        # Extract non-zero balances (only people who owe or are owed money)
        balances = [balance for balance in balance_map.values() if balance != 0]
        num_people = len(balances)

        # dp[mask] = minimum number of transactions to settle debts for people in mask
        # mask is a bitmask where bit i represents whether person i is included
        dp = [float('inf')] * (1 << num_people)
        dp[0] = 0  # Base case: no people need 0 transactions

        # Iterate through all possible subsets of people
        for mask in range(1, 1 << num_people):
            # Calculate the sum of balances for people in current mask
            balance_sum = 0
            for person_idx, person_balance in enumerate(balances):
                if mask >> person_idx & 1:  # Check if person_idx is in the mask
                    balance_sum += person_balance

            # If sum is 0, this subset can be settled internally
            if balance_sum == 0:
                # A group with zero sum needs (group_size - 1) transactions
                dp[mask] = mask.bit_count() - 1

                # Try splitting this group into two subgroups
                # Iterate through all non-empty subsets of current mask
                submask = (mask - 1) & mask
                while submask > 0:
                    # Update minimum: current group split into submask and (mask ^ submask)
                    dp[mask] = min(dp[mask], dp[submask] + dp[mask ^ submask])
                    # Move to next submask
                    submask = (submask - 1) & mask

        # Return the minimum transactions needed for all people
        return dp[-1]
```

## Time and Space Complexity

**Time Complexity:** `O(3^m)` where `m` is the number of people with non-zero balance.

The algorithm uses dynamic programming with bitmask to find the minimum number of transactions. For each subset represented by mask `i`:
- We iterate through all possible subsets of `i` using `j = (i - 1) & i` operation, which generates all non-empty proper subsets
- For a set with `k` elements, there are `2^k - 1` non-empty subsets
- The total number of operations across all masks is bounded by `∑(i=1 to 2^m) 2^(bit_count(i))`, which equals `3^m`
- Additionally, for each mask, we compute the sum which takes `O(m)` time
- Therefore, the overall time complexity is `O(m * 2^m + 3^m) = O(3^m)`

**Space Complexity:** `O(2^m)` where `m` is the number of people with non-zero balance.

The space is dominated by:
- The array `f` of size `2^m` to store the minimum transactions for each subset
- The `nums` array of size `m` to store non-zero balances
- The dictionary `g` which stores at most `n` entries where `n` is the number of unique people in transactions
- Since we only consider non-zero balances, and `m ≤ n`, the space complexity is `O(2^m)`

## Common Pitfalls

### Pitfall 1: Incorrect Balance Calculation Direction

A frequent mistake is reversing the signs when calculating balances. Many developers intuitively think "if A gives money to B, A has positive balance" but this leads to incorrect results.

**Wrong approach:**
```python
for from_person, to_person, amount in transactions:
    balance_map[from_person] += amount  # WRONG!
    balance_map[to_person] -= amount    # WRONG!
```

**Why it's wrong:** This reverses the debt relationship. The person giving money should have a negative balance (they're out money), and the person receiving should have positive balance (they're up money).

**Correct approach:**
```python
for from_person, to_person, amount in transactions:
    balance_map[from_person] -= amount  # Giver loses money
    balance_map[to_person] += amount    # Receiver gains money
```

### Pitfall 2: Not Filtering Zero Balances

Including people with zero balance in the DP calculation significantly increases complexity without adding value.

**Wrong approach:**
```python
balances = list(balance_map.values())  # Includes zeros!
```

**Why it's wrong:** People with zero balance are already settled and don't need any transactions. Including them increases the state space from `2^n` to `2^m` where `m > n`, making the algorithm much slower.

**Correct approach:**
```python
balances = [balance for balance in balance_map.values() if balance != 0]
```

### Pitfall 3: Incorrect Subset Iteration

The subset enumeration logic using bit manipulation is tricky and easy to get wrong.

**Wrong approach:**
```python
# Trying to iterate through subsets incorrectly
for submask in range(1, mask):
    if submask & mask == submask:  # Check if submask is subset of mask
        dp[mask] = min(dp[mask], dp[submask] + dp[mask ^ submask])
```

**Why it's wrong:** This iterates through many unnecessary values and is inefficient. It checks `O(2^n)` values for each mask instead of just the actual subsets.

**Correct approach:**
```python
submask = (mask - 1) & mask
while submask > 0:
    dp[mask] = min(dp[mask], dp[submask] + dp[mask ^ submask])
    submask = (submask - 1) & mask
```

This efficiently iterates only through actual subsets of `mask` using the formula `(submask - 1) & mask`.

### Pitfall 4: Forgetting the Base Case for Zero-Sum Groups

When a subset sums to zero, forgetting to set the base case before trying splits leads to incorrect results.

**Wrong approach:**
```python
if balance_sum == 0:
    # Only try splitting, forget the direct settlement
    submask = (mask - 1) & mask
    while submask > 0:
        dp[mask] = min(dp[mask], dp[submask] + dp[mask ^ submask])
        submask = (submask - 1) & mask
```

**Why it's wrong:** This misses the fact that a zero-sum group can be settled directly with `(size - 1)` transactions, which might be better than any split.

**Correct approach:**
```python
if balance_sum == 0:
    # First set the base case
    dp[mask] = mask.bit_count() - 1
    # Then try splitting to see if we can do better
    submask = (mask - 1) & mask
    while submask > 0:
        dp[mask] = min(dp[mask], dp[submask] + dp[mask ^ submask])
        submask = (submask - 1) & mask
```

