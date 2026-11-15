# 135. Candy

> Source: https://algo.monster/liteproblems/135

## Problem Statement

You have `n` children standing in a line, and each child has a rating value given in an array called `ratings`.

You need to distribute candies to these children following these rules:
- Every child must receive at least one candy
- If a child has a higher rating than their neighbor (the child directly to their left or right), they must receive more candies than that neighbor

Your task is to find the minimum total number of candies needed to satisfy both requirements.

For example, if you have children with ratings `[1, 0, 2]`:
- The first child (rating 1) is higher than the second child (rating 0), so they need more candies than the second child
- The third child (rating 2) is higher than the second child (rating 0), so they need more candies than the second child
- One valid distribution would be `[2, 1, 2]` candies respectively, giving a total of 5 candies

The challenge is to find the minimum number of candies while ensuring that any child with a higher rating than their immediate neighbor gets more candies than that neighbor.

## Intuition

The key insight is that each child's candy count is influenced by both their left and right neighbors. When we look at a child, they need to satisfy two potential constraints:
1. If they have a higher rating than their left neighbor, they need more candies than the left neighbor
2. If they have a higher rating than their right neighbor, they need more candies than the right neighbor

The challenge is that we can't determine the optimal candy count in a single pass because we don't know what's coming ahead. For instance, if we only traverse left to right and assign candies based on the left neighbor, we might violate the constraint with the right neighbor.

Consider the ratings `[1, 2, 3, 2, 1]`. If we only go left to right:
- We'd assign `[1, 2, 3, ?, ?]` based on increasing ratings
- But the child with rating 3 also needs more candies than the child with rating 2 to its right
- And that child with rating 2 needs more candies than the child with rating 1 to its right

This creates a cascading effect that we can't resolve in a single left-to-right pass.

The solution is to handle each direction separately:
- First, traverse left to right and ensure each child with a higher rating than their left neighbor gets more candies than them
- Then, traverse right to left and ensure each child with a higher rating than their right neighbor gets more candies than them
- Finally, for each child, take the maximum of these two requirements

Why take the maximum? Because a child must satisfy both constraints simultaneously. If the left constraint says they need 3 candies and the right constraint says they need 2 candies, they must have at least 3 candies to satisfy both conditions.

This two-pass approach ensures we capture all the dependencies from both directions and find the minimum candies needed while respecting all neighbor relationships.

## Solution Approach

We implement the two-pass approach using two arrays to track candy requirements from each direction.

**Step 1: Initialize Arrays**
- Create two arrays `left` and `right`, both of size `n` (number of children)
- Initialize all values to 1, since each child must have at least one candy
- `left[i]` will store the minimum candies needed considering only left neighbors
- `right[i]` will store the minimum candies needed considering only right neighbors

**Step 2: Left-to-Right Pass**
- Traverse from index 1 to `n-1`
- For each position `i`, check if `ratings[i] > ratings[i-1]`
- If true, set `left[i] = left[i-1] + 1`
- This ensures children with higher ratings than their left neighbor get more candies

**Step 3: Right-to-Left Pass**
- Traverse from index `n-2` down to 0
- For each position `i`, check if `ratings[i] > ratings[i+1]`
- If true, set `right[i] = right[i+1] + 1`
- This ensures children with higher ratings than their right neighbor get more candies

**Step 4: Calculate Final Result**
- For each child at position `i`, their actual candy count must be `max(left[i], right[i])`
- Taking the maximum ensures both left and right neighbor constraints are satisfied
- Sum up all these maximum values to get the total minimum candies needed

**Example Walkthrough:**
For `ratings = [1, 0, 2]`:
- After left-to-right pass: `left = [1, 1, 2]` (child at index 2 needs more than child at index 1)
- After right-to-left pass: `right = [2, 1, 1]` (child at index 0 needs more than child at index 1)
- Final candies: `[max(1,2), max(1,1), max(2,1)] = [2, 1, 2]`
- Total: 5 candies

The time complexity is `O(n)` as we make three passes through the array, and the space complexity is `O(n)` for the two auxiliary arrays.

## Example Walkthrough

Let's walk through the solution with `ratings = [1, 3, 2, 2, 1]`:

**Step 1: Initialize Arrays**
- Create `left = [1, 1, 1, 1, 1]` and `right = [1, 1, 1, 1, 1]`
- Every child starts with at least 1 candy

**Step 2: Left-to-Right Pass**
- Index 1: `ratings[1]=3 > ratings[0]=1`, so `left[1] = left[0] + 1 = 2`
- Index 2: `ratings[2]=2 < ratings[1]=3`, so `left[2]` stays 1
- Index 3: `ratings[3]=2 = ratings[2]=2`, so `left[3]` stays 1
- Index 4: `ratings[4]=1 < ratings[3]=2`, so `left[4]` stays 1
- Result: `left = [1, 2, 1, 1, 1]`

**Step 3: Right-to-Left Pass**
- Index 3: `ratings[3]=2 > ratings[4]=1`, so `right[3] = right[4] + 1 = 2`
- Index 2: `ratings[2]=2 = ratings[3]=2`, so `right[2]` stays 1
- Index 1: `ratings[1]=3 > ratings[2]=2`, so `right[1] = right[2] + 1 = 2`
- Index 0: `ratings[0]=1 < ratings[1]=3`, so `right[0]` stays 1
- Result: `right = [1, 2, 1, 2, 1]`

**Step 4: Calculate Final Result**
- Child 0: `max(left[0]=1, right[0]=1) = 1` candy
- Child 1: `max(left[1]=2, right[1]=2) = 2` candies
- Child 2: `max(left[2]=1, right[2]=1) = 1` candy
- Child 3: `max(left[3]=1, right[3]=2) = 2` candies
- Child 4: `max(left[4]=1, right[4]=1) = 1` candy

**Final Distribution:** `[1, 2, 1, 2, 1]`
**Total Candies:** 1 + 2 + 1 + 2 + 1 = **7 candies**

**Verification:**
- Child 1 (rating 3) has 2 candies, more than both neighbors (1 candy each) ✓
- Child 3 (rating 2) has 2 candies, more than child 4 (1 candy) ✓
- All other neighbor relationships are satisfied ✓

## Python Solution

```python
class Solution:
    def candy(self, ratings: List[int]) -> int:
        """
        Distribute minimum candies to children based on ratings.
        Each child must receive at least one candy.
        Children with higher ratings than neighbors must get more candies.
      
        Args:
            ratings: List of integers representing children's ratings
          
        Returns:
            Minimum number of candies needed
        """
        n = len(ratings)
      
        # Initialize arrays to track candy requirements from left and right traversals
        left_to_right = [1] * n  # Minimum candies needed considering left neighbor
        right_to_left = [1] * n  # Minimum candies needed considering right neighbor
      
        # Left to right pass: ensure each child with higher rating than left neighbor gets more candy
        for i in range(1, n):
            if ratings[i] > ratings[i - 1]:
                left_to_right[i] = left_to_right[i - 1] + 1
      
        # Right to left pass: ensure each child with higher rating than right neighbor gets more candy
        for i in range(n - 2, -1, -1):
            if ratings[i] > ratings[i + 1]:
                right_to_left[i] = right_to_left[i + 1] + 1
      
        # Take maximum of both requirements at each position to satisfy both constraints
        total_candies = sum(max(left_candies, right_candies) 
                           for left_candies, right_candies in zip(left_to_right, right_to_left))
      
        return total_candies
```

## Time and Space Complexity

**Time Complexity:** `O(n)`

The algorithm consists of three main parts:
- First loop: Iterates through the array once from left to right (`i` from `1` to `n-1`), performing constant time operations at each step - `O(n)`
- Second loop: Iterates through the array once from right to left (`i` from `n-2` to `0`), performing constant time operations at each step - `O(n)`
- Final sum calculation: Uses `zip` and `max` to iterate through both arrays once, taking the maximum of corresponding elements and summing them - `O(n)`

Total time complexity: `O(n) + O(n) + O(n) = O(n)`

**Space Complexity:** `O(n)`

The algorithm uses:
- `left` array: stores `n` elements - `O(n)`
- `right` array: stores `n` elements - `O(n)`
- The `zip` operation creates an iterator (not additional storage)
- Variables `n` and loop counter `i` - `O(1)`

Total space complexity: `O(n) + O(n) + O(1) = O(n)`

Where `n` is the length of the ratings array.

## Common Pitfalls

### Pitfall 1: Using a Single Array Instead of Two Separate Arrays

**The Problem:**
A common mistake is trying to optimize space by using a single array and updating it in place during both passes. This approach fails because the second pass can overwrite values needed for correct calculation.

**Incorrect Approach:**
```python
candies = [1] * n
# Left to right pass
for i in range(1, n):
    if ratings[i] > ratings[i-1]:
        candies[i] = candies[i-1] + 1

# Right to left pass - THIS IS WRONG!
for i in range(n-2, -1, -1):
    if ratings[i] > ratings[i+1]:
        candies[i] = candies[i+1] + 1  # Overwrites left-to-right values!
```

**Why It Fails:**
For ratings `[1, 2, 3, 4, 1]`, the left pass gives `[1, 2, 3, 4, 1]`. In the right pass, position 3 (rating 4) is higher than position 4, so it is set to `candies[4] + 1 = 2`, overwriting the 4 it needed from the left pass. The final distribution `[1, 2, 3, 2, 1]` gives the child with rating 4 fewer candies than the neighbor with rating 3, which breaks the rule.

**The Solution:**
Use two separate arrays as shown in the original solution, or if using a single array, modify the right-to-left pass to take the maximum:

```python
candies = [1] * n
# Left to right pass
for i in range(1, n):
    if ratings[i] > ratings[i-1]:
        candies[i] = candies[i-1] + 1

# Right to left pass with maximum check
for i in range(n-2, -1, -1):
    if ratings[i] > ratings[i+1]:
        candies[i] = max(candies[i], candies[i+1] + 1)
```

### Pitfall 2: Confusing Strict Inequality with Non-Strict Comparison

**The Problem:**
Using `>=` instead of `>` when comparing ratings, or incorrectly handling equal ratings.

**Incorrect Logic:**
```python
if ratings[i] >= ratings[i-1]:  # WRONG - equal ratings don't require more candies
    left_to_right[i] = left_to_right[i-1] + 1
```

**Why It Fails:**
The problem states that only children with *higher* ratings than neighbors need more candies. Children with equal ratings to their neighbors can have any relative candy amounts. Using `>=` would unnecessarily increase the candy count.

**The Solution:**
Always use strict inequality (`>`) when comparing ratings. Children with equal ratings don't have constraints relative to each other, which helps minimize the total candy count.

