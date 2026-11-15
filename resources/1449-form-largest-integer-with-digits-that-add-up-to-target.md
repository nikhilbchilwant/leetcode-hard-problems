# 1449. Form Largest Integer With Digits That Add up to Target

> Source: https://algo.monster/liteproblems/1449

## Problem Statement

You need to form the largest possible integer by "painting" digits, where each digit has an associated cost.

The problem gives you:
- An array `cost` where `cost[i]` represents the cost to paint digit `(i + 1)`
  - For example, `cost[0]` is the cost to paint digit 1, `cost[1]` is the cost to paint digit 2, and so on up to digit 9
- A `target` value representing the exact total cost you must spend

Your goal is to find the maximum integer you can create by selecting and arranging digits such that:
1. The sum of costs for all selected digits equals exactly `target`
2. The resulting number doesn't contain the digit 0 (only digits 1-9 are available)
3. You want the largest possible integer value

The key insights for maximizing the integer value:
- First, maximize the number of digits (more digits means a larger number)
- Among numbers with the same digit count, prefer using larger digits (9 is better than 1)
- Digits can be reused multiple times

For example, if `cost = [4,3,2,5,6,7,2,5,5]` and `target = 9`:
- Digit 3 costs 2, and digit 7 costs 2
- You could paint "777" (cost: 2+2+2 = 6) or "7777" (cost: 2+2+2+2 = 8) or "77777" (cost: 2+2+2+2+2 = 10)
- But since target is 9, you could paint "7773" (cost: 2+2+2+3 = 9)

The solution uses [dynamic programming](/problems/dynamic_programming_intro) where:
- `f[i][j]` tracks the maximum number of digits achievable using digits 1 through `i` with exact cost `j`
- `g[i][j]` helps reconstruct which digits were chosen by storing the previous state
- The algorithm fills the DP table considering whether to use digit `i` or not at each step
- Finally, it reconstructs the answer by backtracking through the states, preferring larger digits when possible

Return the result as a string since the number can be very large. If it's impossible to spend exactly `target` cost, return "0".

## Intuition

To get the largest possible number, we need to think about what makes one number larger than another. A number with more digits is always larger than a number with fewer digits (e.g., 1000 > 999). So our first priority is maximizing the number of digits we can paint with our budget.

This leads us to a key observation: if we want the most digits for our money, we should look for the cheapest digit available. However, once we know how many digits we can afford, we want those digits to be as large as possible (preferring 9s over 1s).

This creates a two-phase optimization problem:
1. First, find the maximum number of digits we can paint with exactly `target` cost
2. Then, among all ways to paint that many digits, choose the combination that gives the largest number

The challenge is that we can't just greedily pick the cheapest digit repeatedly. We need to spend exactly `target` cost, not just stay under it. For example, if the cheapest digit costs 3 and our target is 10, we can't paint 3 digits (cost 9) because we'd have 1 unit left over that we can't spend.

This "exact change" constraint naturally suggests [dynamic programming](/problems/dynamic_programming_intro). We can build up solutions by asking: "If I have budget `j`, what's the maximum number of digits I can paint using digits 1 through `i`?"

For each digit `i` and budget `j`, we have two choices:
- Don't use digit `i` at all: inherit the best solution from digits 1 through `i-1`
- Use digit `i`: if we have enough budget (`j >= cost[i-1]`), we can paint digit `i` and then solve the subproblem with remaining budget `j - cost[i-1]`

The DP table `f[i][j]` stores the maximum number of digits achievable. We also need `g[i][j]` to track our decisions so we can reconstruct which digits to use.

Once we've filled the DP table and know the maximum number of digits possible, we reconstruct the answer by working backwards. At each step, we greedily choose to use the largest digit possible (starting from 9 and going down). This ensures that our final number has the largest possible value among all numbers with the maximum digit count.

The reconstruction process examines `g[i][j]` to determine whether digit `i` was used. If `g[i][j] == j`, it means we didn't use digit `i` at that state, so we move to `i-1`. Otherwise, we used digit `i`, add it to our answer, and move to the state with reduced budget `g[i][j]`.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution uses a 2D [dynamic programming](/problems/dynamic_programming_intro) approach with state reconstruction. Let's walk through the implementation step by step:

**DP Table Setup:**
- `f[i][j]` represents the maximum number of digits we can paint using digits 1 through `i` with exact cost `j`
- Initialize `f` with `-inf` to mark impossible states
- Set `f[0][0] = 0` as base case (0 digits with 0 cost)
- `g[i][j]` stores the previous budget state to help reconstruct the solution

**Filling the DP Table:**
```python
for i, c in enumerate(cost, 1):  # i represents digit i (1-indexed)
    for j in range(target + 1):  # j represents current budget
```

For each digit `i` and budget `j`, we make a decision:

1. **Option 1: Don't use digit `i`**
   - Inherit the solution from `f[i-1][j]`
   - Set `g[i][j] = j` (budget remains unchanged)

2. **Option 2: Use digit `i`**
   - Only possible if `j >= c` (enough budget)
   - New digit count would be `f[i][j-c] + 1`
   - Set `g[i][j] = j - c` (remaining budget after using digit `i`)

The decision logic:
```python
if j < c or f[i][j - c] + 1 < f[i - 1][j]:
    f[i][j] = f[i - 1][j]    # Don't use digit i
    g[i][j] = j
else:
    f[i][j] = f[i][j - c] + 1  # Use digit i
    g[i][j] = j - c
```

We choose Option 2 (use digit `i`) only if:
- We have enough budget (`j >= c`)
- AND using digit `i` gives us more total digits (`f[i][j-c] + 1 >= f[i-1][j]`)

**Checking Feasibility:**
```python
if f[9][target] < 0:
    return "0"
```
If `f[9][target]` is negative, it's impossible to spend exactly `target` cost.

**Reconstructing the Answer:**
Starting from state `(i=9, j=target)`, we work backwards to build the largest number:

```python
ans = []
i, j = 9, target
while i:
    if j == g[i][j]:
        i -= 1  # Digit i wasn't used, move to i-1
    else:
        ans.append(str(i))  # Digit i was used
        j = g[i][j]  # Move to previous budget state
```

The reconstruction starts from digit 9 and works down. This ensures we use the largest digits possible for the maximum digit count. When `j != g[i][j]`, it means we used digit `i` at this state, so we:
1. Add digit `i` to our answer
2. Update `j` to the previous budget state `g[i][j]`

The algorithm continues until `i = 0`, at which point we've reconstructed all the digits used.

**Time Complexity:** `O(9 * target)` for filling the DP table and `O(max_digits)` for reconstruction
**Space Complexity:** `O(9 * target)` for the two 2D arrays

The key insight is that by processing digits from 1 to 9 and reconstructing from 9 to 1, we automatically get the largest possible number among all numbers with the maximum digit count.

## Example Walkthrough

Let's walk through a small example with `cost = [2, 4, 6, 2, 4, 6, 4, 4, 4]` and `target = 6`.

This means:
- Digit 1 costs 2, Digit 2 costs 4, Digit 3 costs 6
- Digit 4 costs 2, Digit 5 costs 4, Digit 6 costs 6
- Digit 7 costs 4, Digit 8 costs 4, Digit 9 costs 4

**Step 1: Initialize DP tables**
- `f[i][j]` tracks max digits using digits 1-i with cost j
- `f[0][0] = 0`, all other entries start as `-inf`

**Step 2: Fill DP table for key states**

For digit 1 (cost=2):
- At j=2: Can use digit 1 once, `f[1][2] = 1`
- At j=4: Can use digit 1 twice, `f[1][4] = 2`
- At j=6: Can use digit 1 three times, `f[1][6] = 3`

For digit 4 (cost=2):
- At j=2: Can use digit 4 once, `f[4][2] = 1`
- At j=4: Can use digit 4 twice, `f[4][4] = 2`
- At j=6: Can use digit 4 three times, `f[4][6] = 3`

Since digits 1 and 4 have the same cost (2), they achieve the same max digit counts. But digit 4 is larger, so when we have a choice, we prefer it.

For digit 9 (cost=4):
- At j=4: Can use digit 9 once, `f[9][4] = 1`
- At j=6: Best option is still three digits (using cheaper digits), `f[9][6] = 3`

**Step 3: Check feasibility**
`f[9][6] = 3`, which is positive, so a solution exists.

**Step 4: Reconstruct the answer**
Starting from state (i=9, j=6):

1. i=9, j=6: Check if we used digit 9
   - `g[9][6]` tells us whether we used digit 9 here
   - Since using digit 9 (cost 4) would leave us with j=2, and we can get 1 more digit with cost 2, total would be 2 digits
   - But not using digit 9 and using cheaper digits gives us 3 digits
   - So `g[9][6] = 6` (didn't use digit 9)
   - Move to i=8, j=6

2. Continue checking digits 8, 7, 6, 5 (all cost 4 or 6)
   - Similar logic: using them gives fewer than 3 digits
   - Keep moving down without using them

3. i=4, j=6: Check if we used digit 4
   - Using digit 4 (cost 2) leaves j=4, where we can get 2 more digits
   - Total: 3 digits, same as not using it
   - But digit 4 is valuable! `g[4][6] = 4` (we did use digit 4)
   - Add "4" to answer, move to i=4, j=4

4. i=4, j=4: Check if we used digit 4 again
   - Using digit 4 leaves j=2, where we can get 1 more digit
   - Total: 2 digits. `g[4][4] = 2` (we did use digit 4)
   - Add "4" to answer, move to i=4, j=2

5. i=4, j=2: Check if we used digit 4 again
   - Using digit 4 leaves j=0, where we have 0 more digits
   - Total: 1 digit. `g[4][2] = 0` (we did use digit 4)
   - Add "4" to answer, move to i=4, j=0

6. j=0, so we're done

**Final answer: "444"**

This makes sense because:
- We maximized the number of digits (3 digits)
- Among all ways to get 3 digits with cost 6, we used the largest possible digits (all 4s instead of mixing with 1s)
- The cost is exactly 2+2+2 = 6

## Python Solution

```python
class Solution:
    def largestNumber(self, cost: List[int], target: int) -> str:
        # dp[i][j] = maximum count of digits we can use with digits 1..i and exact cost j
        # Initialize with negative infinity to mark invalid states
        dp = [[-float('inf')] * (target + 1) for _ in range(10)]
        dp[0][0] = 0  # Base case: 0 digits with 0 cost is valid (0 digits used)
      
        # parent[i][j] = the cost from which we transitioned to reach state (i, j)
        # Used for backtracking to construct the answer
        parent = [[0] * (target + 1) for _ in range(10)]
      
        # Fill the DP table
        for digit in range(1, 10):  # Digits 1-9 (index i corresponds to digit i)
            digit_cost = cost[digit - 1]  # Cost of current digit
          
            for current_cost in range(target + 1):
                # Option 1: Don't use current digit, inherit from previous digit
                dont_use = dp[digit - 1][current_cost]
              
                # Option 2: Use current digit (if we have enough budget)
                if current_cost >= digit_cost:
                    use_digit = dp[digit][current_cost - digit_cost] + 1
                else:
                    use_digit = -float('inf')
              
                # Choose the option that gives more digits
                if use_digit > dont_use:
                    dp[digit][current_cost] = use_digit
                    parent[digit][current_cost] = current_cost - digit_cost
                else:
                    dp[digit][current_cost] = dont_use
                    parent[digit][current_cost] = current_cost
      
        # Check if it's possible to achieve the target cost
        if dp[9][target] < 0:
            return "0"
      
        # Backtrack to construct the largest number
        result = []
        current_digit, remaining_cost = 9, target
      
        while current_digit > 0:
            if remaining_cost == parent[current_digit][remaining_cost]:
                # Current digit was not used, move to previous digit
                current_digit -= 1
            else:
                # Current digit was used, add it to result
                result.append(str(current_digit))
                remaining_cost = parent[current_digit][remaining_cost]
      
        return "".join(result)
```

## Time and Space Complexity

**Time Complexity:** `O(n * target)` where `n` is the length of the cost array (which is 9 in this problem since digits are 1-9).

The algorithm uses dynamic programming with two nested loops:
- The outer loop iterates through each digit from 1 to 9 (i.e., through the cost array)
- The inner loop iterates through all possible values from 0 to target
- Each iteration performs constant time operations (comparisons and assignments)
- The reconstruction phase (while loop at the end) runs in `O(target)` time in the worst case, as we can select at most `target` digits (when all costs are 1)

Therefore, the overall time complexity is `O(9 * target) + O(target) = O(target)`.

**Space Complexity:** `O(n * target) = O(target)`

The algorithm uses:
- Two 2D arrays `f` and `g`, each of size `10 × (target + 1)`, requiring `O(10 * target) = O(target)` space
- An array `ans` to store the result digits, which in the worst case could have `target` elements (when minimum cost is 1), requiring `O(target)` space
- A few constant variables `i`, `j`, and `c`

The total space complexity is `O(target) + O(target) = O(target)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect DP Initialization - Using 0 Instead of Negative Infinity**

**The Pitfall:**
A common mistake is initializing the DP table with 0 instead of negative infinity:
```python
# WRONG - This makes invalid states appear valid
dp = [[0] * (target + 1) for _ in range(10)]
```

This causes the algorithm to incorrectly consider impossible states as valid (having 0 digits), leading to wrong answers when backtracking.

**Why This Happens:**
When we check `dp[digit][current_cost - digit_cost] + 1`, if this cell contains 0 (but should be impossible), we'd incorrectly think we can add 1 digit to a valid 0-digit solution.

**The Fix:**
Always initialize with negative infinity except for the base case:
```python
dp = [[-float('inf')] * (target + 1) for _ in range(10)]
dp[0][0] = 0  # Only valid base case
```

### 2. **Allowing Digit Reuse in the Wrong DP Dimension**

**The Pitfall:**
Using `dp[digit - 1][current_cost - digit_cost]` instead of `dp[digit][current_cost - digit_cost]`:
```python
# WRONG - Prevents digit reuse
use_digit = dp[digit - 1][current_cost - digit_cost] + 1
```

This would mean each digit can only be used once, drastically limiting our solutions.

**The Fix:**
Allow the same digit to be reused by referencing the same digit row:
```python
# CORRECT - Allows digit reuse
use_digit = dp[digit][current_cost - digit_cost] + 1
```

### 3. **Incorrect Backtracking Logic**

**The Pitfall:**
During reconstruction, incorrectly determining when a digit was used:
```python
# WRONG - Comparing with wrong value
if parent[current_digit][remaining_cost] != remaining_cost - cost[current_digit - 1]:
    current_digit -= 1
else:
    result.append(str(current_digit))
    remaining_cost = parent[current_digit][remaining_cost]
```

**Why This Fails:**
The parent array stores the previous cost state, not the cost difference. Misunderstanding this leads to incorrect digit selection.

**The Fix:**
Check if the cost remained the same (digit not used) or changed (digit used):
```python
if remaining_cost == parent[current_digit][remaining_cost]:
    # Cost unchanged - digit not used
    current_digit -= 1
else:
    # Cost changed - digit was used
    result.append(str(current_digit))
    remaining_cost = parent[current_digit][remaining_cost]
```

### 4. **Not Handling the Greedy Choice Correctly**

**The Pitfall:**
Breaking ties incorrectly when multiple options give the same digit count:
```python
# WRONG - Using >= instead of >
if use_digit >= dont_use:  # This might not give largest number
    dp[digit][current_cost] = use_digit
```

**Why This Matters:**
When `use_digit == dont_use`, we have two ways to achieve the same digit count. Since we process digits from 1 to 9 and reconstruct from 9 to 1, using `>=` would prefer smaller digits when counts are equal, giving a smaller final number.

**The Fix:**
Use strict inequality to prefer the "don't use" option when counts are equal:
```python
if use_digit > dont_use:  # Only use digit if it gives MORE digits
    dp[digit][current_cost] = use_digit
```

This ensures that during backtracking from digit 9 to 1, we naturally prefer larger digits when multiple options exist for the same digit count.

### 5. **Off-by-One Errors with Digit Indexing**

**The Pitfall:**
Forgetting that digit `i` corresponds to `cost[i-1]`:
```python
# WRONG - Using wrong index
digit_cost = cost[digit]  # Should be cost[digit - 1]
```

**The Fix:**
Always remember the mapping: digit 1 → cost[0], digit 2 → cost[1], ..., digit 9 → cost[8]:
```python
digit_cost = cost[digit - 1]
```

