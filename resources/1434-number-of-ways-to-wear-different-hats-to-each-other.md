# 1434. Number of Ways to Wear Different Hats to Each Other

> Source: https://algo.monster/liteproblems/1434

## Problem Statement

You have `n` people and 40 types of hats (labeled from 1 to 40). Each person has a list of hats they prefer to wear.

Given a 2D array `hats` where `hats[i]` contains all the hat types that person `i` likes, you need to find the number of ways to assign hats to all `n` people such that:
- Each person wears exactly one hat
- Each person wears a hat they prefer (from their list)
- No two people wear the same hat

Since the answer can be very large, return the result modulo `10^9 + 7`.

For example, if person 0 likes hats [1, 2] and person 1 likes hats [2, 3], there are 2 valid ways:
- Person 0 wears hat 1, person 1 wears hat 2
- Person 0 wears hat 2, person 1 wears hat 3

The key constraint is that all `n` people must wear different hats from each other, and each person can only wear a hat from their preference list.

## Intuition

The first observation is that we have at most 10 people but up to 40 hats. This suggests we should think about the problem from the perspective of hats rather than people, since there are fewer people to track.

Since `n ≤ 10`, we can use a [bitmask](/problems/bitmask_intro) to represent which people have already been assigned hats. For example, if we have 3 people and persons 0 and 2 have been assigned hats, we can represent this state as `101` in binary (or 5 in decimal).

The key insight is to process hats one by one and track which combinations of people have been assigned hats so far. Instead of asking "which hat should person i wear?", we ask "for hat i, which person (if any) should wear it?"

We can build our solution incrementally:
- Start with hat 1, then hat 2, and so on
- For each hat `i`, we have two choices:
  1. Don't assign it to anyone (skip it)
  2. Assign it to one of the people who likes this hat (if they haven't been assigned a hat yet)

This leads to [dynamic programming](/problems/dynamic_programming_intro) where `f[i][j]` represents the number of ways to assign the first `i` hats to achieve state `j`, where `j` is a [bitmask](/problems/bitmask_intro) indicating which people have received hats.

The state transition works as follows:
- If we skip hat `i`, then `f[i][j]` includes all ways from `f[i-1][j]`
- If we assign hat `i` to person `k` (who likes it and is in state `j`), then we add the ways from the previous state where person `k` didn't have a hat, which is `f[i-1][j ^ (1 << k)]`

By processing all hats and tracking all possible assignment states, we can find the total number of ways to assign different hats to all `n` people, which is `f[m][(1 << n) - 1]` where `m` is the highest numbered hat and `(1 << n) - 1` represents all people having hats.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) and [Bitmask](/problems/bitmask_intro) patterns.**

## Solution Approach

We implement the [dynamic programming](/problems/dynamic_programming_intro) solution using a 2D array where the first dimension represents hats and the second dimension represents the state of people (using [bitmask](/problems/bitmask_intro)).

**Step 1: Build the preference mapping**
```python
g = defaultdict(list)
for i, h in enumerate(hats):
    for v in h:
        g[v].append(i)
```
We create a dictionary `g` where `g[hat]` contains the list of people who like that hat. This reverse mapping makes it easier to process hats one by one.

**Step 2: Initialize the DP table**
```python
n = len(hats)
m = max(max(h) for h in hats)
f = [[0] * (1 << n) for _ in range(m + 1)]
f[0][0] = 1
```
- `n` is the number of people
- `m` is the highest hat number that anyone likes
- `f[i][j]` represents the number of ways to assign the first `i` hats to achieve state `j`
- We initialize `f[0][0] = 1` (one way to assign 0 hats to 0 people)

**Step 3: Fill the DP table**
```python
for i in range(1, m + 1):
    for j in range(1 << n):
        f[i][j] = f[i - 1][j]  # Case 1: Skip hat i
        for k in g[i]:         # Case 2: Assign hat i to person k
            if j >> k & 1:     # Check if person k is in state j
                f[i][j] = (f[i][j] + f[i - 1][j ^ (1 << k)]) % mod
```

For each hat `i` and each state `j`:
- **Case 1**: We don't assign hat `i` to anyone, so `f[i][j]` inherits from `f[i-1][j]`
- **Case 2**: We try to assign hat `i` to each person `k` who likes it
  - `j >> k & 1` checks if the `k`-th bit of `j` is set (person `k` has a hat in state `j`)
  - If yes, we add the count from the previous state where person `k` didn't have a hat: `f[i-1][j ^ (1 << k)]`
  - The XOR operation `j ^ (1 << k)` flips the `k`-th bit, removing person `k` from the state

**Step 4: Return the result**
```python
return f[m][-1]
```
The answer is `f[m][(1 << n) - 1]`, which represents the number of ways to assign up to `m` hats such that all `n` people have hats. Since `(1 << n) - 1` is the last element in the array (all bits set), we can access it with `f[m][-1]`.

The time complexity is `O(m × 2^n × n)` where `m` is the number of hats and `n` is the number of people. The space complexity is `O(m × 2^n)`.

## Example Walkthrough

Let's walk through a small example where we have 2 people and their hat preferences:
- Person 0 likes hats [1, 2]
- Person 1 likes hats [2, 3]

**Step 1: Build the reverse mapping**
```
g[1] = [0]      # Hat 1 is liked by person 0
g[2] = [0, 1]   # Hat 2 is liked by persons 0 and 1
g[3] = [1]      # Hat 3 is liked by person 1
```

**Step 2: Initialize DP table**
- n = 2 people, m = 3 (highest hat number)
- We need states from 00 to 11 in binary (0 to 3 in decimal)
- f[0][0] = 1 (one way to assign 0 hats to 0 people)

**Step 3: Process each hat**

**Processing Hat 1:**
- State 00 (nobody has hats): f[1][00] = f[0][00] = 1 (skip hat 1)
- State 01 (person 0 has hat): 
  - Skip: f[0][01] = 0
  - Assign to person 0: f[0][00] = 1
  - Total: f[1][01] = 0 + 1 = 1
- State 10 (person 1 has hat): f[1][10] = f[0][10] = 0 (person 1 doesn't like hat 1)
- State 11 (both have hats): f[1][11] = f[0][11] = 0

After hat 1: f[1] = [1, 1, 0, 0]

**Processing Hat 2:**
- State 00: f[2][00] = f[1][00] = 1 (skip hat 2)
- State 01: 
  - Skip: f[1][01] = 1
  - Person 0 already has hat, can't assign
  - Total: f[2][01] = 1
- State 10:
  - Skip: f[1][10] = 0
  - Assign to person 1: f[1][00] = 1
  - Total: f[2][10] = 0 + 1 = 1
- State 11:
  - Skip: f[1][11] = 0
  - Assign to person 0: f[1][10] = 0
  - Assign to person 1: f[1][01] = 1
  - Total: f[2][11] = 0 + 0 + 1 = 1

After hat 2: f[2] = [1, 1, 1, 1]

**Processing Hat 3:**
- State 00: f[3][00] = f[2][00] = 1
- State 01: f[3][01] = f[2][01] = 1 (person 0 doesn't like hat 3)
- State 10:
  - Skip: f[2][10] = 1
  - Person 1 already has hat, can't assign
  - Total: f[3][10] = 1
- State 11:
  - Skip: f[2][11] = 1
  - Assign to person 1: f[2][01] = 1
  - Total: f[3][11] = 1 + 1 = 2

After hat 3: f[3] = [1, 1, 1, 2]

**Result:** f[3][11] = 2, meaning there are 2 ways to assign hats:
1. Person 0 wears hat 1, person 1 wears hat 2
2. Person 0 wears hat 2, person 1 wears hat 3

## Python Solution

```python
class Solution:
    def numberWays(self, hats: List[List[int]]) -> int:
        # Build a mapping from each hat to the list of people who can wear it
        hat_to_people = defaultdict(list)
        for person_id, person_hats in enumerate(hats):
            for hat in person_hats:
                hat_to_people[hat].append(person_id)
      
        MOD = 10**9 + 7
        num_people = len(hats)
        max_hat_id = max(max(person_hats) for person_hats in hats)
      
        # dp[i][mask] = number of ways to assign first i hats to people
        # where mask represents which people have been assigned hats
        # bit j in mask = 1 means person j has been assigned a hat
        dp = [[0] * (1 << num_people) for _ in range(max_hat_id + 1)]
      
        # Base case: 0 hats considered, no one has a hat = 1 way
        dp[0][0] = 1
      
        # Process each hat from 1 to max_hat_id
        for hat_id in range(1, max_hat_id + 1):
            for people_mask in range(1 << num_people):
                # Option 1: Don't use the current hat
                dp[hat_id][people_mask] = dp[hat_id - 1][people_mask]
              
                # Option 2: Assign current hat to one of the eligible people
                for person in hat_to_people[hat_id]:
                    # Check if this person already has a hat in current mask
                    if people_mask >> person & 1:
                        # Add ways from previous state where this person didn't have a hat
                        previous_mask = people_mask ^ (1 << person)
                        dp[hat_id][people_mask] = (dp[hat_id][people_mask] + 
                                                    dp[hat_id - 1][previous_mask]) % MOD
      
        # Return the number of ways where all hats are considered and all people have hats
        # The mask with all people having hats is (1 << num_people) - 1 = all bits set
        return dp[max_hat_id][(1 << num_people) - 1]
```

## Time and Space Complexity

**Time Complexity:** `O(m × 2^n × n)`

The algorithm uses dynamic programming with states `f[i][j]` where `i` represents hats from 1 to `m` and `j` represents a bitmask of size `2^n` for people assignments.

- The outer loop iterates through `m` hats: `O(m)`
- The middle loop iterates through all possible bitmasks: `O(2^n)`
- The inner loop iterates through people who can wear hat `i`, which in the worst case can be all `n` people: `O(n)`

Therefore, the overall time complexity is `O(m × 2^n × n)`.

**Space Complexity:** `O(m × 2^n)`

The algorithm uses a 2D DP array `f` with dimensions `(m + 1) × 2^n`:
- First dimension has size `m + 1` for hats from 0 to `m`
- Second dimension has size `2^n` for all possible bitmask states

Additionally, the graph `g` stores the mapping of hats to people, which takes `O(m × n)` space in the worst case.

Since `O(m × 2^n) > O(m × n)` when `n > 1`, the dominant space complexity is `O(m × 2^n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect State Representation - Processing People Instead of Hats**

A natural but inefficient approach is to use `dp[person][hats_used_mask]` where we process people one by one and track which hats have been used. This leads to a state space of `O(n × 2^40)` which is computationally infeasible since 2^40 is enormous.

**Why this happens:** It's intuitive to think "assign hats to people" and process people sequentially.

**Solution:** Reverse the perspective - process hats one by one and track which people have received hats. This gives us `O(40 × 2^n)` states, which is manageable since n ≤ 10.

### 2. **Off-by-One Error in DP Array Initialization**

A common mistake is initializing the DP array incorrectly:
```python
# Wrong - doesn't account for hat 0 (which doesn't exist)
dp = [[0] * (1 << num_people) for _ in range(max_hat_id)]
```

**Why this happens:** Forgetting that we need an extra row for the base case (0 hats considered).

**Solution:** Initialize with `max_hat_id + 1` rows:
```python
dp = [[0] * (1 << num_people) for _ in range(max_hat_id + 1)]
```

### 3. **Incorrect Bit Manipulation for Previous State**

When calculating the previous state, using wrong operations:
```python
# Wrong - using OR instead of XOR
previous_mask = people_mask | (1 << person)

# Wrong - using AND instead of XOR  
previous_mask = people_mask & ~(1 << person)
```

**Why this happens:** Confusion about how to remove a person from the bitmask.

**Solution:** Use XOR to flip the bit:
```python
previous_mask = people_mask ^ (1 << person)
```
This correctly removes person from the current mask to get the previous state.

### 4. **Forgetting the Modulo Operation**

Missing the modulo operation during state transitions:
```python
# Wrong - might cause integer overflow
dp[hat_id][people_mask] = dp[hat_id][people_mask] + dp[hat_id - 1][previous_mask]
```

**Why this happens:** Focusing on the logic and forgetting about the constraint that answers can be very large.

**Solution:** Apply modulo after each addition:
```python
dp[hat_id][people_mask] = (dp[hat_id][people_mask] + 
                           dp[hat_id - 1][previous_mask]) % MOD
```

### 5. **Using 1-indexed Hats with 0-indexed Arrays**

Confusion between hat numbering (1 to 40) and array indexing (0-based):
```python
# Wrong - treating hat numbers as 0-indexed
for hat_id in range(max_hat_id):  # Missing hat 40 if max_hat_id is 40
```

**Why this happens:** The problem states hats are numbered 1-40, but arrays are 0-indexed.

**Solution:** Be consistent - either adjust indices when building the mapping or ensure loops cover the correct range:
```python
for hat_id in range(1, max_hat_id + 1):  # Correctly processes hats 1 to max_hat_id
```

