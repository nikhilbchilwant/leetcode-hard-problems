# 1537. Get the Maximum Score

> Source: https://algo.monster/liteproblems/1537

## Problem Statement

You have two sorted arrays of distinct integers, `nums1` and `nums2`. Your task is to find a path through these arrays that maximizes the sum of values you collect.

Here's how you can traverse:
- Start from index 0 of either `nums1` or `nums2`
- Move through your chosen array from left to right
- When you encounter a value that exists in both arrays (a common element), you can switch to the other array at that point
- Each common value is only counted once in your path sum

The goal is to find the path that gives you the maximum possible sum of all values visited.

For example, if `nums1 = [2, 4, 5, 8, 10]` and `nums2 = [4, 6, 8, 9]`:
- Common elements are 4 and 8
- You could start in `nums1`, collect 2, then at 4 you have a choice to switch or stay
- If you switch at 4, you'd continue in `nums2` with 6, then at 8 you could switch again
- The optimal path would be: 2 → 4 (switch) → 6 → 8 (switch) → 10, giving sum = 30

The solution uses [two pointers](/problems/two_pointers_intro) `i` and `j` to traverse both arrays simultaneously, and two variables `f` and `g` to track the maximum sum achievable up to the current position in each array. When a common element is found (where `nums1[i] == nums2[j]`), both paths converge to the better sum plus that common element, allowing the algorithm to effectively "switch" to the more profitable path.

Since the result could be very large, return the answer modulo `10^9 + 7`.

## Intuition

The key insight is that common elements in both arrays act as "bridges" or "junction points" where we can switch between paths. Between any two consecutive common elements (or from the start to the first common element, or from the last common element to the end), we're essentially choosing which array gives us a larger sum for that segment.

Think of it like having two parallel roads with bridges connecting them at certain points. At each bridge, you want to have taken the road that gave you more points up to that bridge. After crossing the bridge, you again have the freedom to choose which road to take until the next bridge.

This naturally leads to a two-pointer approach where we process both arrays simultaneously. We maintain two running sums:
- `f`: maximum sum we can achieve if we're currently in `nums1`
- `g`: maximum sum we can achieve if we're currently in `nums2`

As we traverse:
- When `nums1[i] < nums2[j]`, we add `nums1[i]` to `f` and move pointer `i` forward
- When `nums1[i] > nums2[j]`, we add `nums2[j]` to `g` and move pointer `j` forward
- When `nums1[i] == nums2[j]` (a common element), this is our bridge! We can now "merge" the paths by taking the maximum of `f` and `g` up to this point, add the common element, and set both `f` and `g` to this new value. This effectively allows us to switch to whichever path was better.

The beauty of this approach is that we don't need to explicitly track which array we're in or make switching decisions. By maintaining these two sums and merging them at common elements, we automatically get the optimal path. At the end, we return `max(f, g)` to get the maximum possible sum.

**Learn more about [Greedy](/problems/greedy_intro), [Two Pointers](/problems/two_pointers_intro) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution implements a two-pointer technique with [dynamic programming](/problems/dynamic_programming_intro) concepts to find the maximum sum path.

**Initialization:**
- Set `mod = 10^9 + 7` for the final modulo operation
- Get lengths `m` and `n` for `nums1` and `nums2` respectively
- Initialize pointers `i = j = 0` to traverse both arrays from the beginning
- Initialize `f = g = 0` where:
  - `f` tracks the maximum sum if we're currently following `nums1`
  - `g` tracks the maximum sum if we're currently following `nums2`

**Main Loop Logic:**
The algorithm processes elements while either array has unvisited elements (`i < m or j < n`):

1. **When `nums1` is exhausted** (`i == m`):
   - Add remaining elements from `nums2` to `g`
   - Increment `j`

2. **When `nums2` is exhausted** (`j == n`):
   - Add remaining elements from `nums1` to `f`
   - Increment `i`

3. **When `nums1[i] < nums2[j]`**:
   - This element only exists in `nums1`, so add it to `f`
   - Move pointer `i` forward

4. **When `nums1[i] > nums2[j]`**:
   - This element only exists in `nums2`, so add it to `g`
   - Move pointer `j` forward

5. **When `nums1[i] == nums2[j]`** (common element found):
   - This is a junction point where paths can merge
   - Set both `f` and `g` to `max(f, g) + nums1[i]`
   - This ensures we take the better path up to this point and continue with the optimal sum
   - Increment both `i` and `j` since we've processed this common element in both arrays

**Final Result:**
Return `max(f, g) % mod` to get the maximum sum among both possible ending paths, modulo `10^9 + 7`.

The time complexity is `O(m + n)` as we traverse each array at most once, and the space complexity is `O(1)` as we only use a constant amount of extra variables.

## Example Walkthrough

Let's trace through a small example to understand how the algorithm works.

**Given:** `nums1 = [1, 3, 5, 7]` and `nums2 = [2, 3, 6, 7, 8]`

**Initial State:**
- `i = 0, j = 0`
- `f = 0` (max sum if we're in nums1)
- `g = 0` (max sum if we're in nums2)

**Step 1:** Compare `nums1[0]=1` vs `nums2[0]=2`
- Since `1 < 2`, add 1 to path in nums1: `f = 0 + 1 = 1`
- Move `i` to 1
- State: `i=1, j=0, f=1, g=0`

**Step 2:** Compare `nums1[1]=3` vs `nums2[0]=2`
- Since `3 > 2`, add 2 to path in nums2: `g = 0 + 2 = 2`
- Move `j` to 1
- State: `i=1, j=1, f=1, g=2`

**Step 3:** Compare `nums1[1]=3` vs `nums2[1]=3`
- Common element found! This is a junction point
- Merge paths: both `f` and `g` become `max(1, 2) + 3 = 5`
- Move both pointers: `i` to 2, `j` to 2
- State: `i=2, j=2, f=5, g=5`

**Step 4:** Compare `nums1[2]=5` vs `nums2[2]=6`
- Since `5 < 6`, add 5 to path in nums1: `f = 5 + 5 = 10`
- Move `i` to 3
- State: `i=3, j=2, f=10, g=5`

**Step 5:** Compare `nums1[3]=7` vs `nums2[2]=6`
- Since `7 > 6`, add 6 to path in nums2: `g = 5 + 6 = 11`
- Move `j` to 3
- State: `i=3, j=3, f=10, g=11`

**Step 6:** Compare `nums1[3]=7` vs `nums2[3]=7`
- Common element found! Another junction point
- Merge paths: both `f` and `g` become `max(10, 11) + 7 = 18`
- Move both pointers: `i` to 4, `j` to 4
- State: `i=4, j=4, f=18, g=18`

**Step 7:** `i=4` means nums1 is exhausted, but `j=4 < 5`
- Add remaining element from nums2: `g = 18 + 8 = 26`
- Move `j` to 5
- State: `i=4, j=5, f=18, g=26`

**Final Result:** `max(f, g) = max(18, 26) = 26`

The optimal path was: **2 → 3 → 6 → 7 → 8** (starting in nums2, staying in nums2 at the first junction, and continuing in nums2 after the second junction).

Note how at each common element (3 and 7), the algorithm automatically "chose" the better path by taking the maximum of `f` and `g`. This ensures we always continue with the optimal sum without explicitly tracking which array we're in.

## Python Solution

```python
class Solution:
    def maxSum(self, nums1: List[int], nums2: List[int]) -> int:
        MOD = 10**9 + 7
        len1, len2 = len(nums1), len(nums2)
      
        # Two pointers for traversing both arrays
        pointer1 = pointer2 = 0
      
        # Running sums for paths through nums1 and nums2 respectively
        sum_path1 = sum_path2 = 0
      
        # Process both arrays using two pointers
        while pointer1 < len1 or pointer2 < len2:
            # If we've exhausted nums1, add remaining elements from nums2
            if pointer1 == len1:
                sum_path2 += nums2[pointer2]
                pointer2 += 1
            # If we've exhausted nums2, add remaining elements from nums1
            elif pointer2 == len2:
                sum_path1 += nums1[pointer1]
                pointer1 += 1
            # If current element in nums1 is smaller, add it to path1
            elif nums1[pointer1] < nums2[pointer2]:
                sum_path1 += nums1[pointer1]
                pointer1 += 1
            # If current element in nums2 is smaller, add it to path2
            elif nums1[pointer1] > nums2[pointer2]:
                sum_path2 += nums2[pointer2]
                pointer2 += 1
            # If elements are equal (intersection point)
            else:
                # At intersection, take the maximum sum so far and add current element
                # Both paths now have the same sum after this intersection
                sum_path1 = sum_path2 = max(sum_path1, sum_path2) + nums1[pointer1]
                pointer1 += 1
                pointer2 += 1
      
        # Return the maximum of the two path sums, modulo MOD
        return max(sum_path1, sum_path2) % MOD
```

## Time and Space Complexity

**Time Complexity:** `O(m + n)` where `m` is the length of `nums1` and `n` is the length of `nums2`.

The algorithm uses two pointers `i` and `j` to traverse through both arrays simultaneously. In each iteration of the while loop, at least one pointer moves forward by one position. Since pointer `i` can move at most `m` times and pointer `j` can move at most `n` times, the total number of iterations is bounded by `m + n`. Each operation inside the loop (comparisons, additions, and max operations) takes constant time `O(1)`. Therefore, the overall time complexity is `O(m + n)`.

**Space Complexity:** `O(1)`

The algorithm only uses a constant amount of extra space for variables:
- Two pointers `i` and `j`
- Two accumulator variables `f` and `g` to track the maximum sum paths
- Variables `m` and `n` to store array lengths
- Variable `mod` for the modulo operation

All these variables require constant space regardless of the input size. The algorithm modifies values in-place without creating any additional data structures that scale with input size. Therefore, the space complexity is `O(1)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Forgetting to Apply Modulo Operation**
A critical pitfall is applying the modulo operation incorrectly or forgetting it entirely. Many developers might apply modulo during intermediate calculations, which can lead to incorrect results.

**Incorrect approach:**
```python
# Wrong: Applying modulo during intermediate calculations
sum_path1 = (sum_path1 + nums1[pointer1]) % MOD
sum_path2 = (sum_path2 + nums2[pointer2]) % MOD
```

**Why it's wrong:** When we compare `max(sum_path1, sum_path2)` at intersection points, the modulo operation can distort the actual comparison. For example, if `sum_path1 = 10^9 + 8` and `sum_path2 = 10^9 + 6`, after modulo they become `1` and `10^9 - 1` respectively, incorrectly suggesting `sum_path2` is larger.

**Correct approach:**
```python
# Only apply modulo at the very end
return max(sum_path1, sum_path2) % MOD
```

### 2. **Misunderstanding the Intersection Logic**
Another common mistake is not properly handling the intersection points where both arrays have the same value.

**Incorrect approach:**
```python
# Wrong: Only updating one path at intersection
if nums1[pointer1] == nums2[pointer2]:
    sum_path1 = max(sum_path1, sum_path2) + nums1[pointer1]
    pointer1 += 1
    pointer2 += 1
```

**Why it's wrong:** This only updates `sum_path1` but leaves `sum_path2` with its old value. Future comparisons at the next intersection will be incorrect.

**Correct approach:**
```python
# Both paths should converge to the same optimal value at intersection
if nums1[pointer1] == nums2[pointer2]:
    sum_path1 = sum_path2 = max(sum_path1, sum_path2) + nums1[pointer1]
    pointer1 += 1
    pointer2 += 1
```

### 3. **Incorrect Handling of Array Exhaustion**
Some implementations might terminate the loop when one array is exhausted, missing elements from the other array.

**Incorrect approach:**
```python
# Wrong: Stopping when one array is exhausted
while pointer1 < len1 and pointer2 < len2:
    # process elements
    ...
# Missing elements from the remaining array
```

**Correct approach:**
```python
# Continue until BOTH arrays are exhausted
while pointer1 < len1 or pointer2 < len2:
    if pointer1 == len1:
        # Process remaining elements from nums2
    elif pointer2 == len2:
        # Process remaining elements from nums1
    # ... rest of the logic
```

### 4. **Not Recognizing That Arrays Are Already Sorted**
Some might attempt to find common elements using nested loops or sets, adding unnecessary complexity.

**Inefficient approach:**
```python
# Unnecessary: Using sets to find common elements
common = set(nums1) & set(nums2)
```

**Why it's inefficient:** Since both arrays are sorted, the two-pointer technique efficiently finds intersections in O(m+n) time without extra space for sets.

