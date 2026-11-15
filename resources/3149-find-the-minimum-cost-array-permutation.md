# 3149. Find the Minimum Cost Array Permutation

> Source: https://algo.monster/liteproblems/3149

## Problem Statement

You are given an array `nums` which is a permutation of `[0, 1, 2, ..., n - 1]`. Your task is to find another permutation `perm` of the same set `[0, 1, 2, ..., n - 1]` that minimizes a specific score.

The score of any permutation `perm` is calculated as:
`score(perm) = |perm[0] - nums[perm[1]]| + |perm[1] - nums[perm[2]]| + ... + |perm[n - 1] - nums[perm[0]]|`

In this formula:
- Each term calculates the absolute difference between an element in `perm` at position `i` and an element in `nums` at position `perm[i+1]`
- The calculation wraps around cyclically, so the last term uses `perm[n-1]` and `nums[perm[0]]`

The goal is to find the permutation `perm` that produces the minimum possible score. If multiple permutations have the same minimum score, you should return the lexicographically smallest one among them.

For example, if `nums = [1, 0, 2]`, you need to find a permutation of `[0, 1, 2]` that, when used as indices and values according to the scoring formula, gives the smallest total sum of absolute differences. The permutation should start with 0 when possible for lexicographic ordering, and among all minimum-score permutations, choose the one that comes first in dictionary order.

## Intuition

The first key observation is that the score formula creates a cycle: we're essentially creating a circular chain where each element in `perm` points to the next one, and we sum up the differences between each element and what it points to in `nums`. Since this is a cycle, we can rotate any valid permutation and get the same score. For example, `[0, 1, 2]` and `[1, 2, 0]` would have the same score value.

Given that rotations preserve the score and we want the lexicographically smallest result, we can fix the first element as `0`. This simplifies our problem significantly - we only need to find the optimal arrangement of the remaining elements `[1, 2, ..., n-1]`.

The small constraint (`n ≤ 14`) immediately suggests that we can use bit manipulation and [dynamic programming](/problems/dynamic_programming_intro). With at most 14 elements, we have `2^14 = 16384` possible states, which is manageable.

We can think of building the permutation step by step. At each step, we need to decide which unvisited number to visit next. The challenge is that our current choice affects future costs - if we choose number `cur` after `pre`, we pay `|pre - nums[cur]|` now, but this also determines where we'll be when making the next choice.

This naturally leads to a state-based [dynamic programming](/problems/dynamic_programming_intro) approach where:
- We track which numbers we've already used (using a [bitmask](/problems/bitmask_intro))
- We track what the last number we chose was (since this affects the cost of the next choice)
- We compute the minimum remaining cost from each state

Once we know the minimum costs, we can reconstruct the actual permutation by greedily choosing the next number that leads to the optimal total cost while maintaining lexicographic order (trying smaller numbers first).

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) and [Bitmask](/problems/bitmask_intro) patterns.**

## Solution Approach

The solution uses memoized recursion with state compression to find the optimal permutation. Here's how the implementation works:

**State Representation:**
We use a [bitmask](/problems/bitmask_intro) `mask` to represent which numbers have been selected. If the `i`-th bit of `mask` is 1, it means number `i` has been included in our permutation. For example, `mask = 5` (binary: `101`) means we've selected numbers 0 and 2.

**[Dynamic Programming](/problems/dynamic_programming_intro) Function `dfs(mask, pre)`:**
This function calculates the minimum score possible when:
- We've already selected the numbers indicated by `mask`
- The last selected number was `pre`

The function works as follows:
1. **Base case:** If `mask == (1 << n) - 1` (all bits set, meaning all numbers selected), we've completed the cycle. Return `|pre - nums[0]|` to close the cycle back to the first element.

2. **Recursive case:** For each unselected number `cur` (where `mask >> cur & 1 ^ 1` checks if bit `cur` is 0):
   - Calculate the cost of selecting `cur` next: `|pre - nums[cur]|`
   - Add the minimum cost for the remaining choices: `dfs(mask | 1 << cur, cur)`
   - Track the minimum total cost across all choices

The `@cache` decorator memoizes results to avoid recomputing the same states.

**Permutation Construction with `g(mask, pre)`:**
Once we know the minimum costs, we reconstruct the actual permutation:

1. Add the current number `pre` to our answer list
2. If we've selected all numbers (`mask == (1 << n) - 1`), we're done
3. Otherwise, get the minimum cost from the current state: `res = dfs(mask, pre)`
4. Try each unselected number `cur` in ascending order (for lexicographic ordering):
   - Check if selecting `cur` achieves the minimum cost: `|pre - nums[cur]| + dfs(mask | 1 << cur, cur) == res`
   - If yes, recursively build the rest of the permutation with `g(mask | 1 << cur, cur)` and break (ensuring lexicographically smallest choice)

**Main Execution:**
The solution starts by calling `g(1, 0)`, which means:
- Initial mask is `1` (binary: `0...001`), indicating only number 0 is selected
- Starting with `pre = 0` (first element is 0 for lexicographic ordering)

The time complexity is `O(n^2 * 2^n)` due to the state space and transitions, while the space complexity is `O(n * 2^n)` for memoization.

## Example Walkthrough

Let's walk through the solution with `nums = [1, 0, 2]`.

**Step 1: Understanding the Score Calculation**
For any permutation `perm`, the score is:
`|perm[0] - nums[perm[1]]| + |perm[1] - nums[perm[2]]| + |perm[2] - nums[perm[0]]|`

**Step 2: Initialize DFS Computation**
We start with `perm[0] = 0` (for lexicographic ordering). Now we need to find the minimum cost using `dfs(mask, pre)`:

- `dfs(001, 0)`: mask=1 (binary 001) means only 0 is selected, pre=0
  - Can choose 1: cost = `|0 - nums[1]| + dfs(011, 1)` = `|0 - 0| + dfs(3, 1)` = `0 + dfs(3, 1)`
  - Can choose 2: cost = `|0 - nums[2]| + dfs(101, 2)` = `|0 - 2| + dfs(5, 2)` = `2 + dfs(5, 2)`

- `dfs(011, 1)`: mask=3 means 0,1 selected, pre=1
  - Only 2 available: cost = `|1 - nums[2]| + dfs(111, 2)` = `|1 - 2| + dfs(7, 2)` = `1 + dfs(7, 2)`

- `dfs(101, 2)`: mask=5 means 0,2 selected, pre=2
  - Only 1 available: cost = `|2 - nums[1]| + dfs(111, 1)` = `|2 - 0| + dfs(7, 1)` = `2 + dfs(7, 1)`

- `dfs(111, 2)`: All selected, pre=2, return to start
  - Return `|2 - nums[0]|` = `|2 - 1|` = 1

- `dfs(111, 1)`: All selected, pre=1, return to start
  - Return `|1 - nums[0]|` = `|1 - 1|` = 0

**Step 3: Compute Minimum Costs**
Working backwards:
- `dfs(7, 2)` = 1
- `dfs(7, 1)` = 0
- `dfs(3, 1)` = 1 + 1 = 2
- `dfs(5, 2)` = 2 + 0 = 2
- `dfs(1, 0)` = min(0 + 2, 2 + 2) = min(2, 4) = 2

**Step 4: Construct the Permutation**
Now we build the permutation using `g(mask, pre)`:

Starting with `g(1, 0)`:
- Add 0 to answer: `ans = [0]`
- Check minimum cost: `dfs(1, 0)` = 2
- Try cur=1: `|0 - nums[1]| + dfs(3, 1)` = 0 + 2 = 2 ✓ (equals minimum)
- Recurse with `g(3, 1)`

Continue with `g(3, 1)`:
- Add 1 to answer: `ans = [0, 1]`
- Only option is cur=2: `|1 - nums[2]| + dfs(7, 2)` = 1 + 1 = 2
- Recurse with `g(7, 2)`

Finally `g(7, 2)`:
- Add 2 to answer: `ans = [0, 1, 2]`
- All bits set, we're done

**Result:** The optimal permutation is `[0, 1, 2]` with score = 2

We can verify: 
- `|0 - nums[1]| + |1 - nums[2]| + |2 - nums[0]|`
- `= |0 - 0| + |1 - 2| + |2 - 1|`
- `= 0 + 1 + 1 = 2`

## Python Solution

```python
class Solution:
    def findPermutation(self, nums: List[int]) -> List[int]:
        """
        Find a permutation of indices [0, 1, ..., n-1] that minimizes the total
        absolute difference when traversing the array in that order (forming a cycle).
      
        Args:
            nums: List of integers
          
        Returns:
            List of indices representing the optimal permutation
        """
      
        @cache
        def calculate_min_cost(visited_mask: int, last_index: int) -> int:
            """
            Calculate minimum cost to visit all remaining nodes starting from last_index.
          
            Args:
                visited_mask: Bitmask representing visited indices (1 = visited)
                last_index: The index of the last visited element
              
            Returns:
                Minimum cost to complete the tour
            """
            # If all nodes are visited, return cost to go back to start
            if visited_mask == (1 << n) - 1:
                return abs(nums[last_index] - nums[0])
          
            min_cost = float('inf')
          
            # Try visiting each unvisited node
            for next_index in range(1, n):
                # Check if next_index is not visited (bit is 0)
                if (visited_mask >> next_index) & 1 == 0:
                    # Calculate cost: edge cost + remaining tour cost
                    cost = abs(nums[last_index] - nums[next_index]) + \
                           calculate_min_cost(visited_mask | (1 << next_index), next_index)
                    min_cost = min(min_cost, cost)
          
            return min_cost
      
        def construct_path(visited_mask: int, current_index: int) -> None:
            """
            Reconstruct the optimal path by following the decisions that led to minimum cost.
          
            Args:
                visited_mask: Bitmask representing visited indices
                current_index: Current position in the tour
            """
            # Add current index to the result path
            result_path.append(current_index)
          
            # If all nodes are visited, we're done
            if visited_mask == (1 << n) - 1:
                return
          
            # Get the optimal cost from current state
            optimal_cost = calculate_min_cost(visited_mask, current_index)
          
            # Find which next move achieves this optimal cost
            for next_index in range(1, n):
                # Check if next_index is not visited
                if (visited_mask >> next_index) & 1 == 0:
                    # Calculate cost of moving to next_index
                    edge_cost = abs(nums[current_index] - nums[next_index])
                    remaining_cost = calculate_min_cost(visited_mask | (1 << next_index), next_index)
                  
                    # If this path gives optimal cost, follow it
                    if edge_cost + remaining_cost == optimal_cost:
                        construct_path(visited_mask | (1 << next_index), next_index)
                        break
      
        # Initialize variables
        n = len(nums)
        result_path = []
      
        # Start from index 0 (with bitmask having only bit 0 set)
        construct_path(1, 0)
      
        return result_path
```

## Time and Space Complexity

**Time Complexity:** `O(n² × 2ⁿ)`

The time complexity is determined by the dynamic programming function `dfs`:
- The function uses memoization with `@cache`, creating at most `2ⁿ × n` unique states (where `mask` has `2ⁿ` possible values representing subsets, and `pre` has `n` possible values representing the previous index)
- For each state `(mask, pre)`, the function iterates through all `n` positions to find unvisited nodes (the for loop from 1 to n)
- Each state is computed only once due to memoization
- Therefore, the total time complexity is `O(n × 2ⁿ × n) = O(n² × 2ⁿ)`

The reconstruction function `g` has the same traversal pattern but doesn't add to the overall complexity since it only traces back the optimal path once.

**Space Complexity:** `O(n × 2ⁿ)`

The space complexity consists of:
- The memoization cache storing up to `2ⁿ × n` states, each taking `O(1)` space
- The recursion stack depth which is at most `O(n)` (visiting each element once)
- The answer array `ans` which stores `O(n)` elements
- Since `2ⁿ × n` dominates, the overall space complexity is `O(n × 2ⁿ)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Cycle Closure in Base Case**

**Pitfall:** A common mistake is forgetting to close the cycle properly or using the wrong indices. In the base case, developers often return `abs(pre - nums[pre])` or `abs(nums[pre] - pre)` instead of the correct `abs(pre - nums[perm[0]])`.

**Why it happens:** The scoring formula `|perm[n-1] - nums[perm[0]]|` for the last term is confusing. When we reach the base case, `pre` represents `perm[n-1]`, and we need to connect back to `nums[perm[0]]`. Since we started with index 0, `perm[0] = 0`, so we should return `abs(pre - nums[0])`.

**Solution:** Always remember that in the cyclic tour, the last element connects back to wherever we started. Since we fix the starting point as 0, the base case should return `abs(pre - nums[0])`.

### 2. **Mixing Up Index and Value in the Scoring Formula**

**Pitfall:** Confusing what represents an index and what represents a value in the formula `|perm[i] - nums[perm[i+1]]|`. Developers might incorrectly implement it as `|nums[perm[i]] - nums[perm[i+1]]|` or `|perm[i] - perm[i+1]|`.

**Why it happens:** The formula uses `perm` both as a source of values (left side) and indices (right side), which is counterintuitive.

**Solution:** Think of it this way:
- `perm[i]` is the VALUE at position i in our permutation
- `perm[i+1]` is used as an INDEX into the nums array
- So we compare: value at position i vs. nums element at index given by position i+1

### 3. **Starting from Arbitrary Position Instead of Fixed Position**

**Pitfall:** Trying to consider all possible starting positions by iterating through different initial values, which breaks the lexicographical ordering requirement and increases complexity unnecessarily.

**Why it happens:** Developers might think they need to try all starting points to find the global minimum, not realizing that fixing the start at 0 is both correct and ensures lexicographical ordering.

**Solution:** Always start with 0 (calling `g(1, 0)`). Since it's a cycle, the minimum score is the same regardless of where we "cut" the cycle to represent it as a sequence. Starting with 0 guarantees the lexicographically smallest result among all minimum-score permutations.

### 4. **Incorrect Bitmask Operations**

**Pitfall:** Using wrong bit operations to check or set visited status:
- Using `mask & (1 << cur)` instead of `(mask >> cur) & 1` to check if bit is set
- Using `mask + (1 << cur)` instead of `mask | (1 << cur)` to set a bit

**Why it happens:** Bit manipulation can be tricky, and different equivalent forms can be confusing.

**Solution:** Stick to consistent patterns:
- Check if visited: `(mask >> cur) & 1 == 1` 
- Check if unvisited: `(mask >> cur) & 1 == 0`
- Mark as visited: `mask | (1 << cur)`

### 5. **Not Breaking After Finding First Valid Choice in Path Construction**

**Pitfall:** In the `construct_path` function, continuing to explore other options after finding the first valid next move, which could lead to incorrect permutations or violate lexicographical ordering.

**Why it happens:** Developers might think they need to explore all paths that achieve minimum cost.

**Solution:** Since we iterate through indices in ascending order (0 to n-1), the first valid choice we find is automatically the lexicographically smallest. Always break immediately after recursing into the first valid path.

