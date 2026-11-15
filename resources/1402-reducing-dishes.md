# 1402. Reducing Dishes

> Source: https://algo.monster/liteproblems/1402

## Problem Statement

You are a chef with `n` dishes, where each dish has a `satisfaction` level. You can cook any dish in exactly 1 unit of time.

The **like-time coefficient** of a dish is calculated as the cooking time (including all previously cooked dishes) multiplied by its satisfaction level. For example:
- If you cook the first dish with satisfaction `s1`, its like-time coefficient is `1 * s1`
- If you cook a second dish with satisfaction `s2`, its like-time coefficient is `2 * s2`
- If you cook a third dish with satisfaction `s3`, its like-time coefficient is `3 * s3`

Your goal is to find the **maximum sum of like-time coefficients** you can achieve. You have complete flexibility in:
- Choosing which dishes to cook (you can discard some dishes)
- Deciding the order in which to cook the selected dishes

For example, if you have dishes with satisfaction levels `[-1, -8, 0, 5, -9]` and you choose to cook dishes with satisfaction `0` and `5` in that order, the total would be:
- First dish (satisfaction 0): `1 * 0 = 0`
- Second dish (satisfaction 5): `2 * 5 = 10`
- Total: `0 + 10 = 10`

But if you cook them in reverse order (5 first, then 0):
- First dish (satisfaction 5): `1 * 5 = 5`
- Second dish (satisfaction 0): `2 * 0 = 0`
- Total: `5 + 0 = 5`

The problem asks you to find the optimal selection and ordering of dishes to maximize the total like-time coefficient. Note that dishes with negative satisfaction can still contribute positively if cooked early and followed by dishes with high positive satisfaction.

## Intuition

Let's think about how to maximize the total like-time coefficient. Since each dish's contribution is `time * satisfaction`, dishes cooked later have their satisfaction multiplied by larger numbers. This immediately suggests we should cook dishes with higher satisfaction later to maximize their contribution.

Consider cooking dishes with satisfactions `[a, b, c]` in that order. The total would be:
- `1*a + 2*b + 3*c`

We can rewrite this as:
- `a + b + c` (each dish counted once)
- `+ b + c` (dishes 2 and 3 get counted again)  
- `+ c` (dish 3 gets counted a third time)

This reveals something interesting: the last dish gets counted 3 times, the second-to-last gets counted 2 times, and so on. So we definitely want to place dishes with higher satisfaction values at the end.

But there's another consideration: should we include dishes with negative satisfaction? A negative dish early in the sequence might still be worth including if it allows highly positive dishes to be cooked later with higher multipliers.

The key insight comes from looking at the rewritten sum above. When we add a new dish at the beginning of our sequence, it shifts all other dishes one position later (increasing their multipliers by 1). The net effect is that we add the sum of all currently selected dishes' satisfactions plus the new dish's satisfaction.

For example, if we're cooking dishes with satisfactions `[3, 5]` giving us `1*3 + 2*5 = 13`, and we want to add a dish with satisfaction `-1` at the beginning:
- New sequence: `[-1, 3, 5]`
- New total: `1*(-1) + 2*3 + 3*5 = -1 + 6 + 15 = 20`
- The increase is: `20 - 13 = 7 = (-1) + 3 + 5`

This means we should keep adding dishes (from highest to lowest satisfaction) as long as the cumulative sum remains positive. Once adding a dish would make the cumulative sum negative or zero, we stop, as it would decrease our total.

**Learn more about [Greedy](/problems/greedy_intro), [Dynamic Programming](/problems/dynamic_programming_intro) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

Based on our intuition, we implement a [greedy](/problems/greedy_intro) algorithm with the following steps:

1. **Sort the satisfaction array in descending order**: We start with `satisfaction.sort(reverse=True)`. This ensures we consider dishes from highest to lowest satisfaction.

2. **Initialize tracking variables**: 
   - `ans = 0`: Tracks the maximum sum of like-time coefficients
   - `s = 0`: Tracks the cumulative sum of satisfactions we're considering

3. **Iterate through sorted dishes**: For each dish with satisfaction `x`:
   - Add `x` to our cumulative sum: `s += x`
   - Check if including this dish is beneficial: `if s <= 0: break`
   - If beneficial, add the cumulative sum to our answer: `ans += s`

The magic happens in how `ans += s` works. When we include a new dish at the beginning of our cooking sequence, it increases the position (and thus the multiplier) of all previously selected dishes by 1. The net effect on our total is exactly the sum of all selected dishes' satisfactions, which is `s`.

**Example walkthrough** with `satisfaction = [-1, -8, 0, 5, -9]`:
1. After [sorting](/problems/sorting_summary): `[5, 0, -1, -8, -9]`
2. First iteration (`x = 5`):
   - `s = 0 + 5 = 5`
   - `s > 0`, so we continue
   - `ans = 0 + 5 = 5`
   - We're cooking `[5]` with total `1*5 = 5`
3. Second iteration (`x = 0`):
   - `s = 5 + 0 = 5`
   - `s > 0`, so we continue
   - `ans = 5 + 5 = 10`
   - We're cooking `[0, 5]` with total `1*0 + 2*5 = 10`
4. Third iteration (`x = -1`):
   - `s = 5 + (-1) = 4`
   - `s > 0`, so we continue
   - `ans = 10 + 4 = 14`
   - We're cooking `[-1, 0, 5]` with total `1*(-1) + 2*0 + 3*5 = 14`
5. Fourth iteration (`x = -8`):
   - `s = 4 + (-8) = -4`
   - `s <= 0`, so we break

The algorithm correctly identifies that we should cook dishes `[-1, 0, 5]` in that order for a maximum total of `14`.

The time complexity is `O(n log n)` due to [sorting](/problems/sorting_summary), and the space complexity is `O(1)` if we sort in-place.

## Example Walkthrough

Let's walk through a small example with `satisfaction = [-2, 5, -1, 0, 3, -3]`.

**Step 1: Sort in descending order**
After sorting: `[5, 3, 0, -1, -2, -3]`

**Step 2: Greedily select dishes**
We'll iterate through the sorted array, maintaining:
- `s`: cumulative sum of satisfactions
- `ans`: our running total of like-time coefficients

| Iteration | Dish (x) | s = s + x | Continue? | ans = ans + s | Dishes Selected | Actual Calculation |
|-----------|----------|-----------|-----------|---------------|-----------------|-------------------|
| 1 | 5 | 0 + 5 = 5 | Yes (s > 0) | 0 + 5 = 5 | [5] | 1×5 = 5 |
| 2 | 3 | 5 + 3 = 8 | Yes (s > 0) | 5 + 8 = 13 | [3, 5] | 1×3 + 2×5 = 13 |
| 3 | 0 | 8 + 0 = 8 | Yes (s > 0) | 13 + 8 = 21 | [0, 3, 5] | 1×0 + 2×3 + 3×5 = 21 |
| 4 | -1 | 8 + (-1) = 7 | Yes (s > 0) | 21 + 7 = 28 | [-1, 0, 3, 5] | 1×(-1) + 2×0 + 3×3 + 4×5 = 28 |
| 5 | -2 | 7 + (-2) = 5 | Yes (s > 0) | 28 + 5 = 33 | [-2, -1, 0, 3, 5] | 1×(-2) + 2×(-1) + 3×0 + 4×3 + 5×5 = 33 |
| 6 | -3 | 5 + (-3) = 2 | Yes (s > 0) | 33 + 2 = 35 | [-3, -2, -1, 0, 3, 5] | 1×(-3) + 2×(-2) + 3×(-1) + 4×0 + 5×3 + 6×5 = 35 |

**Why does `ans += s` work?**
When we add a new dish at the beginning, all existing dishes shift one position later (their multipliers increase by 1). The net increase equals the sum of all selected satisfactions, which is exactly `s`.

For instance, when adding `-1` in iteration 4:
- Previous: `1×0 + 2×3 + 3×5 = 21`
- After adding `-1`: `1×(-1) + 2×0 + 3×3 + 4×5 = 28`
- Increase: `28 - 21 = 7` (which equals `s = -1 + 0 + 3 + 5`)

**Result:** Maximum sum = 35, achieved by cooking all dishes in order `[-3, -2, -1, 0, 3, 5]`.

## Python Solution

```python
class Solution:
    def maxSatisfaction(self, satisfaction: List[int]) -> int:
        # Sort dishes by satisfaction in descending order (highest satisfaction first)
        satisfaction.sort(reverse=True)
      
        # Initialize result and cumulative sum
        max_like_time_coefficient = 0
        cumulative_sum = 0
      
        # Iterate through dishes from highest to lowest satisfaction
        for dish_satisfaction in satisfaction:
            # Add current dish to cumulative sum
            cumulative_sum += dish_satisfaction
          
            # If adding this dish (and all previous dishes) would decrease total satisfaction, stop
            if cumulative_sum <= 0:
                break
          
            # Add cumulative sum to result
            # This effectively adds each dish's satisfaction multiplied by its position
            # when dishes are served in ascending order of satisfaction
            max_like_time_coefficient += cumulative_sum
      
        return max_like_time_coefficient
```

## Time and Space Complexity

The time complexity is `O(n × log n)`, where `n` is the length of the satisfaction array. This is dominated by the sorting operation `satisfaction.sort(reverse=True)`, which uses Timsort algorithm with `O(n × log n)` complexity. The subsequent for loop iterates through the array once with `O(n)` operations, but this is absorbed by the sorting complexity.

The space complexity is `O(log n)`. While the sort operation is performed in-place and doesn't require additional space proportional to the input size, the sorting algorithm (Timsort) uses `O(log n)` space for its recursive call stack during the merge operations. The remaining variables (`ans`, `s`, `x`) use only `O(1)` constant space.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Misunderstanding the Cumulative Sum Logic

**The Problem:** Many developers struggle to understand why `ans += s` correctly calculates the like-time coefficients. They might think they need to explicitly track positions and multiply each dish by its position.

**Why It Happens:** The cumulative sum trick is counterintuitive. When we add a new dish at the beginning of our sequence, it shifts all other dishes one position later, increasing their multipliers by 1.

**Solution:** Understand that when we prepend a dish with satisfaction `x`:
- The new dish contributes `1 * x`
- All previously selected dishes shift one position right, each gaining their satisfaction value once more
- The total gain is `x + (sum of all previously selected dishes)` = current cumulative sum

### Pitfall 2: Incorrect Break Condition

**The Problem:** Using `if cumulative_sum < 0` instead of `if cumulative_sum <= 0` as the break condition.

**Why It Happens:** It seems logical to only stop when the sum becomes negative, not when it's zero.

**The Issue:** When cumulative sum equals 0, adding more dishes won't increase the total (it adds 0), but continuing to iterate wastes computation time. More critically, if you don't break at 0 and the next dish has negative satisfaction, you might incorrectly continue processing.

**Solution:** Always use `<= 0` as the break condition to ensure optimal performance and correctness.

### Pitfall 3: Forgetting to Sort or Sorting in Wrong Order

**The Problem:** Either forgetting to sort the array or sorting in ascending order instead of descending order.

**Why It Happens:** The greedy approach requires considering dishes from highest to lowest satisfaction, but this requirement can be easily overlooked.

**Solution:** Always sort in descending order first. The algorithm depends on evaluating high-satisfaction dishes first to make optimal decisions about including negative-satisfaction dishes.

### Pitfall 4: Attempting to Track Individual Dish Positions

**The Problem:** Creating complex data structures to track which dish goes in which position, leading to unnecessarily complicated code.

**Example of Incorrect Approach:**
```python
# Overly complex approach
selected_dishes = []
for dish in satisfaction:
    selected_dishes.append(dish)
    total = sum(i * dish for i, dish in enumerate(selected_dishes, 1))
    # ... more complex logic
```

**Solution:** Trust the cumulative sum approach. The mathematical relationship `ans += cumulative_sum` handles all position calculations implicitly.

