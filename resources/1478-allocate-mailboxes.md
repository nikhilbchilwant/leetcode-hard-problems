# 1478. Allocate Mailboxes

> Source: https://algo.monster/liteproblems/1478

## Problem Statement

You are given an array `houses` where `houses[i]` represents the location of the i-th house along a street. You need to place exactly `k` mailboxes on this street.

The goal is to minimize the total distance between each house and its nearest mailbox. In other words, for each house, you want to find the closest mailbox to it, and the sum of all these distances should be as small as possible.

**Key Points:**
- Houses are located at specific positions along a street (given as integers in the array)
- You must place exactly `k` mailboxes somewhere on this street
- Each house will be served by its nearest mailbox
- You want to minimize the sum of distances from all houses to their nearest mailboxes
- The answer is guaranteed to fit in a 32-bit integer

**Example Understanding:**
If you have houses at positions `[1, 4, 8, 10, 20]` and you can place `k = 3` mailboxes, you need to strategically place these 3 mailboxes so that when each house goes to its nearest mailbox, the total distance traveled by all houses combined is minimized.

The problem requires finding the optimal placement of mailboxes to achieve this minimum total distance.

## Intuition

Let's think about this problem step by step. First, when we have a group of houses and want to place one mailbox to serve them all, where should we put it? The optimal position is at the median of those houses. This minimizes the total distance because moving the mailbox away from the median would increase the total distance.

Now, with `k` mailboxes for `n` houses, we need to partition the houses into `k` groups, where each group is served by one mailbox. Since we want to minimize distances, it makes sense to sort the houses first - this way, each mailbox will serve a contiguous segment of houses along the street.

This leads us to a [dynamic programming](/problems/dynamic_programming_intro) approach. We can think of the problem as: "What's the minimum cost to serve the first `i+1` houses using `j` mailboxes?" 

For each state `f[i][j]`, we need to decide where the `j-1`th mailbox ends its service. If it ends at house `p`, then:
- Houses `0` to `p` are served by the first `j-1` mailboxes (cost = `f[p][j-1]`)
- Houses `p+1` to `i` are served by the `j`th mailbox (cost = `g[p+1][i]`)

The key insight for calculating `g[i][j]` (cost of serving houses from `i` to `j` with one mailbox) is that when we place a mailbox at the median position, the total distance equals the sum of pairwise distances between houses at opposite ends. For sorted houses, this can be computed efficiently using the recurrence:

`g[i][j] = g[i+1][j-1] + houses[j] - houses[i]`

This works because adding houses `i` and `j` to an existing group adds exactly `houses[j] - houses[i]` to the total distance (they pair up with each other when the mailbox is at the median).

By building up from smaller subproblems to larger ones, we can find the optimal placement strategy for all `k` mailboxes.

**Learn more about [Math](/problems/math-basics), [Dynamic Programming](/problems/dynamic_programming_intro) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

The implementation follows a [dynamic programming](/problems/dynamic_programming_intro) approach with preprocessing for efficient distance calculations.

**Step 1: Sort the houses**
```python
houses.sort()
```
[Sorting](/problems/sorting_summary) ensures that each mailbox will serve a contiguous range of houses, simplifying our problem.

**Step 2: Precompute distances using matrix `g`**
```python
g = [[0] * n for _ in range(n)]
for i in range(n - 2, -1, -1):
    for j in range(i + 1, n):
        g[i][j] = g[i + 1][j - 1] + houses[j] - houses[i]
```
`g[i][j]` represents the minimum total distance when placing one mailbox to serve houses from index `i` to `j`. We build this matrix bottom-up using the recurrence relation:
- `g[i][j] = g[i+1][j-1] + houses[j] - houses[i]`

This formula works because when we add houses at positions `i` and `j` to a group, they contribute `houses[j] - houses[i]` to the total distance (since the optimal mailbox position is at the median).

**Step 3: [Dynamic Programming](/problems/dynamic_programming_intro)**
```python
f = [[inf] * (k + 1) for _ in range(n)]
```
`f[i][j]` represents the minimum total distance to serve houses `0` to `i` using `j` mailboxes.

**Step 4: Base case initialization**
```python
for i in range(n):
    f[i][1] = g[0][i]
```
When using only 1 mailbox, the cost for houses `0` to `i` is simply `g[0][i]`.

**Step 5: Fill the DP table**
```python
for i in range(n):
    for j in range(2, min(k + 1, i + 2)):
        for p in range(i):
            f[i][j] = min(f[i][j], f[p][j - 1] + g[p + 1][i])
```
For each state `f[i][j]`, we try all possible positions `p` where the `(j-1)`th mailbox ends its service:
- Houses `0` to `p` are served by `j-1` mailboxes: cost = `f[p][j-1]`
- Houses `p+1` to `i` are served by the `j`th mailbox: cost = `g[p+1][i]`
- Total cost = `f[p][j-1] + g[p+1][i]`

We take the minimum over all possible splits.

**Step 6: Return the result**
```python
return f[-1][k]
```
The answer is `f[n-1][k]`, which represents the minimum distance to serve all `n` houses using exactly `k` mailboxes.

**Time Complexity:** `O(n³)` for preprocessing `g` and `O(n²k)` for the DP computation.
**Space Complexity:** `O(n²)` for matrix `g` and `O(nk)` for the DP table `f`.

## Example Walkthrough

Let's walk through a small example with houses at positions `[1, 4, 8]` and `k = 2` mailboxes.

**Step 1: Sort the houses**
Houses are already sorted: `[1, 4, 8]`

**Step 2: Build the distance matrix `g`**

We need to calculate `g[i][j]` - the cost of serving houses from index `i` to `j` with one mailbox placed at the median.

Starting from the bottom-right and working backwards:
- `g[1][2]` (houses at 4 and 8): The median is between 4 and 8, and the total distance = 8 - 4 = 4
- `g[0][1]` (houses at 1 and 4): The median is between 1 and 4, and the total distance = 4 - 1 = 3  
- `g[0][2]` (houses at 1, 4, and 8): Using the recurrence `g[0][2] = g[1][1] + houses[2] - houses[0] = 0 + 8 - 1 = 7`

So our `g` matrix is:
```
    0  1  2
0 [ 0  3  7 ]
1 [ 0  0  4 ]
2 [ 0  0  0 ]
```

**Step 3: Initialize DP table `f`**

Create `f[i][j]` where `i` represents serving houses 0 to i, and `j` represents using j mailboxes.

**Step 4: Base case (j = 1)**
When using only 1 mailbox for all houses:
- `f[0][1] = g[0][0] = 0` (one house at position 1)
- `f[1][1] = g[0][1] = 3` (houses at 1 and 4, mailbox at median)
- `f[2][1] = g[0][2] = 7` (houses at 1, 4, 8, mailbox at 4)

**Step 5: Fill DP for j = 2**

For `f[1][2]` (2 houses, 2 mailboxes):
- Split at house 0: `f[0][1] + g[1][1] = 0 + 0 = 0`
- So `f[1][2] = 0` (one mailbox per house)

For `f[2][2]` (3 houses, 2 mailboxes):
- Split at house 0: `f[0][1] + g[1][2] = 0 + 4 = 4`
  - First mailbox serves house at 1 (cost = 0)
  - Second mailbox serves houses at 4 and 8 (cost = 4)
- Split at house 1: `f[1][1] + g[2][2] = 3 + 0 = 3`
  - First mailbox serves houses at 1 and 4 (cost = 3)
  - Second mailbox serves house at 8 (cost = 0)
- So `f[2][2] = min(4, 3) = 3`

**Final DP table:**
```
    j=0  j=1  j=2
i=0 [∞    0    ∞]
i=1 [∞    3    0]
i=2 [∞    7    3]
```

**Step 6: Return result**
`f[2][2] = 3` is our answer.

The optimal placement is: one mailbox serving houses at positions 1 and 4 (placed at position 2.5 or anywhere between them), and another mailbox at position 8. Total distance = |1-2.5| + |4-2.5| + |8-8| = 1.5 + 1.5 + 0 = 3.

## Python Solution

```python
class Solution:
    def minDistance(self, houses: List[int], k: int) -> int:
        # Sort houses by position for optimal mailbox placement
        houses.sort()
        n = len(houses)
      
        # cost[i][j] = minimum cost to place one mailbox for houses[i] to houses[j]
        # The optimal position for one mailbox is the median of the houses
        cost = [[0] * n for _ in range(n)]
      
        # Build cost matrix from right-bottom to left-top
        # For sorted houses, placing mailbox at median minimizes total distance
        for i in range(n - 2, -1, -1):
            for j in range(i + 1, n):
                # Recurrence: extend the range by adding houses at both ends
                # The new cost adds the distance between the new pair of houses
                cost[i][j] = cost[i + 1][j - 1] + houses[j] - houses[i]
      
        # dp[i][j] = minimum distance for houses[0..i] using j mailboxes
        dp = [[float('inf')] * (k + 1) for _ in range(n)]
      
        # Process each house position
        for i in range(n):
            # Base case: one mailbox for houses[0..i]
            dp[i][1] = cost[0][i]
          
            # Try using 2 to min(k, i+1) mailboxes
            # Can't use more mailboxes than houses up to position i
            for j in range(2, min(k + 1, i + 2)):
                # Try all possible positions for the (j-1)th mailbox
                for p in range(i):
                    # Split: houses[0..p] use j-1 mailboxes
                    #         houses[p+1..i] use 1 mailbox
                    dp[i][j] = min(dp[i][j], dp[p][j - 1] + cost[p + 1][i])
      
        # Return minimum distance for all houses using k mailboxes
        return dp[-1][k]
```

## Time and Space Complexity

**Time Complexity:** `O(n^2 × k)`

The analysis breaks down as follows:
- Sorting the houses takes `O(n log n)` time
- Building the cost matrix `g`:
  - The nested loops iterate through all pairs `(i, j)` where `i < j`
  - This results in `O(n^2)` operations
- Computing the DP table `f`:
  - The outer loop runs `n` times (for each house index `i`)
  - The middle loop runs up to `k` times (for each number of mailboxes `j`)
  - The inner loop runs up to `i` times (for each split point `p`)
  - In the worst case, this gives us `O(n × k × n) = O(n^2 × k)`

Since `O(n^2 × k)` dominates `O(n log n)` and `O(n^2)`, the overall time complexity is `O(n^2 × k)`.

**Space Complexity:** `O(n^2)`

The space usage consists of:
- The cost matrix `g`: `n × n` array requiring `O(n^2)` space
- The DP table `f`: `n × (k+1)` array requiring `O(n × k)` space

Since `k ≤ n` (we cannot have more mailboxes than houses), and typically `k` is much smaller than `n`, the `O(n^2)` term from matrix `g` dominates, giving us a total space complexity of `O(n^2)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Cost Matrix Calculation for Single Mailbox**

**Pitfall:** A common mistake is incorrectly calculating the cost matrix `g[i][j]` (or `cost[i][j]` in the code). Developers often try to compute this by iterating through all houses and calculating distances to a median position explicitly, which is both inefficient and error-prone.

**Why it happens:** The recurrence relation `g[i][j] = g[i+1][j-1] + houses[j] - houses[i]` is not intuitive. Many attempt to calculate the median position first and then sum distances, leading to:
```python
# INCORRECT approach
median = houses[(i + j) // 2]
for h in range(i, j + 1):
    cost[i][j] += abs(houses[h] - median)
```

**Solution:** Trust the mathematical property that for sorted houses, when adding two houses at the ends of a range, they contribute exactly `houses[j] - houses[i]` to the total distance when the mailbox is at the median. The correct approach builds the cost matrix using the recurrence relation.

### 2. **Off-by-One Errors in DP Boundaries**

**Pitfall:** Incorrectly setting the loop boundaries, especially:
- Using `range(2, k + 1)` instead of `range(2, min(k + 1, i + 2))` for the number of mailboxes
- This causes attempting to place more mailboxes than houses available up to position `i`

**Why it happens:** Not considering that you cannot place more mailboxes than the number of houses in the current range.

**Solution:** Always bound the number of mailboxes by both `k` and the number of available houses:
```python
for j in range(2, min(k + 1, i + 2)):  # i+2 because we have houses 0 to i (inclusive)
```

### 3. **Forgetting to Sort the Houses Array**

**Pitfall:** Operating on the unsorted houses array, which breaks the fundamental assumption that each mailbox serves a contiguous range of houses.

**Why it happens:** The problem statement doesn't explicitly mention that houses need to be sorted, and developers might assume the input is already sorted or that order doesn't matter.

**Solution:** Always sort the houses array at the beginning:
```python
houses.sort()  # Critical first step
```

### 4. **Incorrect Base Case Initialization**

**Pitfall:** Not properly initializing the base case where one mailbox serves houses from index 0 to i:
```python
# INCORRECT - forgetting base case
dp[i][1] = 0  # Wrong!

# CORRECT
dp[i][1] = cost[0][i]
```

**Why it happens:** Confusion about what the base case represents - it's not zero cost, but the actual cost of placing one mailbox optimally for those houses.

**Solution:** Ensure the base case correctly uses the precomputed cost matrix for placing one mailbox.

### 5. **Integer Overflow in Other Languages**

**Pitfall:** While Python handles large integers automatically, implementing this in languages like Java or C++ might cause overflow when initializing with `Integer.MAX_VALUE` and performing additions.

**Solution:** Use appropriate data types (like `long` in Java) or check for overflow conditions before additions:
```python
# In Python, we use float('inf') which handles this gracefully
dp = [[float('inf')] * (k + 1) for _ in range(n)]
```

