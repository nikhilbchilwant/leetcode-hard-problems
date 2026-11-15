# 2561. Rearranging Fruits

> Source: https://algo.monster/liteproblems/2561

## Problem Statement

You have two fruit baskets, each containing `n` fruits. The cost of each fruit is given in two arrays: `basket1` and `basket2` (both 0-indexed). Your goal is to make both baskets **equal** by swapping fruits between them.

**Swapping Rules:**
- You can swap the `i`-th fruit from `basket1` with the `j`-th fruit from `basket2`
- The cost of each swap is `min(basket1[i], basket2[j])` - you pay the cost of the cheaper fruit being swapped

**What makes baskets equal?**
Two baskets are considered equal if, after sorting both baskets by fruit cost, they contain exactly the same sequence of values.

**Example:** If after swapping, `basket1 = [1, 2, 3]` and `basket2 = [3, 1, 2]`, they are equal because when sorted, both become `[1, 2, 3]`.

**Your task:** Find the minimum total cost to make both baskets equal through swapping. If it's impossible to make them equal, return `-1`.

**Key Insights:**
- You need to redistribute the fruits so both baskets have the same multiset of values
- Each fruit value must appear an even number of times in total (across both baskets) for equal distribution to be possible
- The swapping cost depends on choosing which fruits to swap - you want to minimize the total cost

## Intuition

Let's think about what needs to happen for both baskets to become equal. If we count how many times each fruit value appears across both baskets, we need to redistribute them so each basket gets exactly half of each fruit type.

For example, if the value `5` appears 4 times total (2 in basket1, 2 in basket2), we're already balanced for that value. But if `3` appears 4 times total with 3 in basket1 and 1 in basket2, we need to move one `3` from basket1 to basket2.

This leads to our first key insight: **For any fruit value, if it appears an odd number of times total, it's impossible to split evenly** - we'd return `-1`.

Now, how do we track what needs to be moved? We can use a counter that tracks the "imbalance" for each fruit value. If basket1 has excess of a fruit, we mark it positive; if basket2 has excess, we mark it negative. The fruits with non-zero counts need to be swapped.

For the swapping strategy, we have two options:
1. **Direct swap**: Pick a fruit from basket1 that needs to move and swap it with a fruit from basket2 that needs to move. The cost is the minimum of the two fruit values.
2. **Indirect swap using a middleman**: Sometimes it's cheaper to use the globally cheapest fruit as an intermediary. We swap the fruit we want to move with the cheapest fruit (cost = fruit value), then swap that cheap fruit to the other basket (cost = cheap fruit value). Total cost = `2 * cheapest_fruit_value`.

Here's the clever part: when we have a list of all fruits that need to be moved (half from each basket to the other), we want to pair them optimally. If we sort all the excess fruits that need swapping, we can pair the smallest with the largest, second smallest with second largest, and so on. But for each pair, we also consider if using the middleman (cheapest fruit overall) would be cheaper.

The final insight: we only need to process half of the fruits that need swapping because each swap fixes the position of two fruits simultaneously - one moves from basket1 to basket2, and another moves from basket2 to basket1.

**Learn more about [Greedy](/problems/greedy_intro) patterns.**

## Solution Approach

Let's walk through the implementation step by step:

**Step 1: Count the imbalance**
We use a `Counter` to track which fruits need to be moved. For each position `i`, we increment the counter for `basket1[i]` and decrement for `basket2[i]`. This gives us the net difference for each fruit value.

```python
cnt = Counter()
for a, b in zip(basket1, basket2):
    cnt[a] += 1
    cnt[b] -= 1
```

After this, if `cnt[x] = 4`, it means fruit `x` appears 4 more times in basket1 than needed, so we need to move 2 of them to basket2.

**Step 2: Check feasibility and find minimum fruit value**
While iterating through the counter, we:
- Check if any fruit has an odd count difference (impossible to balance)
- Track the minimum fruit value `mi` (for potential use as middleman)

**Step 3: Build the list of fruits to swap**
For each fruit with imbalance, we add half of the excess fruits to our swap list:

```python
nums = []
for x, v in cnt.items():
    if v % 2:
        return -1  # Odd count means impossible to balance
    nums.extend([x] * (abs(v) // 2))
```

If fruit `x` has count `+4`, we add two `x`s to `nums` (need to move 2 out of basket1).
If fruit `y` has count `-4`, we also add two `y`s to `nums` (need to move 2 into basket1).

**Step 4: Sort and calculate minimum cost**
We sort `nums` to optimally pair fruits for swapping:

```python
nums.sort()
m = len(nums) // 2
```

We only process the first half because:
- The first half represents the cheapest fruits to move
- Each swap involves two fruits (one from each basket), so processing half covers all swaps

**Step 5: Calculate the cost**
For each fruit in the first half, we choose the cheaper option:
- Direct swap cost: the fruit's value `x`
- Indirect swap through middleman: `2 * mi`

```python
return sum(min(x, mi * 2) for x in nums[:m])
```

**Why this works:**
- Sorting ensures we're always choosing the cheapest possible swaps
- The middleman strategy (`2 * mi`) is beneficial when the fruit we want to swap is expensive
- By processing only the first half of sorted `nums`, we automatically pair cheap fruits with expensive ones, minimizing total cost

## Example Walkthrough

Let's walk through a concrete example to illustrate the solution approach.

**Given:**
- `basket1 = [4, 2, 2, 2]`
- `basket2 = [1, 4, 1, 2]`

**Step 1: Count the imbalance**

We create a counter tracking the difference for each fruit:
- Position 0: `cnt[4] += 1`, `cnt[1] -= 1`
- Position 1: `cnt[2] += 1`, `cnt[4] -= 1`  
- Position 2: `cnt[2] += 1`, `cnt[1] -= 1`
- Position 3: `cnt[2] += 1`, `cnt[2] -= 1`

Result: `cnt = {4: 0, 1: -2, 2: +2}`

This means:
- Fruit `4` is balanced (appears equally in both baskets)
- Fruit `1` has deficit of 2 in basket1 (basket2 has 2 extra)
- Fruit `2` has excess of 2 in basket1 (basket1 has 2 extra)

**Step 2: Check feasibility and find minimum**

All counts are even (0, -2, +2), so it's possible to balance.
The minimum fruit value across both baskets is `mi = 1`.

**Step 3: Build swap list**

For each imbalanced fruit, add half the absolute count:
- Fruit `1`: `|-2| // 2 = 1`, add one `1` to nums
- Fruit `2`: `|+2| // 2 = 1`, add one `2` to nums

`nums = [1, 2]`

**Step 4: Sort and prepare for cost calculation**

`nums = [1, 2]` (already sorted)
`m = len(nums) // 2 = 1`

We only need to process the first fruit (`nums[:1] = [1]`)

**Step 5: Calculate minimum cost**

For the fruit with value `1`:
- Direct swap cost: `1`
- Indirect swap using middleman: `2 * mi = 2 * 1 = 2`
- Choose minimum: `min(1, 2) = 1`

Total cost: `1`

**What actually happens:**
We swap one `2` from basket1 with one `1` from basket2. The cost is `min(2, 1) = 1`.

After swapping:
- `basket1 = [4, 1, 2, 2]` → sorted: `[1, 2, 2, 4]`
- `basket2 = [2, 4, 1, 2]` → sorted: `[1, 2, 2, 4]`

Both baskets are now equal when sorted!

## Python Solution

```python
class Solution:
    def minCost(self, basket1: List[int], basket2: List[int]) -> int:
        # Count the frequency difference between basket1 and basket2
        # Positive count means excess in basket1, negative means excess in basket2
        frequency_diff = Counter()
        for fruit1, fruit2 in zip(basket1, basket2):
            frequency_diff[fruit1] += 1
            frequency_diff[fruit2] -= 1
      
        # Find the minimum value fruit across both baskets
        # This will be used for indirect swapping optimization
        min_fruit_value = min(frequency_diff)
      
        # Collect all fruits that need to be swapped
        fruits_to_swap = []
        for fruit_value, count_diff in frequency_diff.items():
            # If the difference is odd, it's impossible to balance
            if count_diff % 2:
                return -1
            # Add half of the excess fruits to the swap list
            # We only need to swap half because each swap fixes two positions
            fruits_to_swap.extend([fruit_value] * (abs(count_diff) // 2))
      
        # Sort fruits by value to minimize swap costs
        fruits_to_swap.sort()
      
        # We only need to consider the first half of swaps
        # The second half is implicitly handled by the first half
        num_swaps_needed = len(fruits_to_swap) // 2
      
        # Calculate minimum cost by choosing between direct swap and indirect swap
        # Direct swap: use the fruit value as cost
        # Indirect swap: use minimum fruit twice (swap with min, then min with target)
        return sum(min(fruit_value, min_fruit_value * 2) for fruit_value in fruits_to_swap[:num_swaps_needed])
```

## Time and Space Complexity

**Time Complexity:** `O(n × log n)`

The time complexity is dominated by the following operations:
- Creating the counter by iterating through both baskets: `O(n)` where `n` is the length of each basket
- Finding the minimum value in the counter: `O(k)` where `k` is the number of unique elements
- Building the `nums` list by iterating through counter items: `O(k)` for iteration, and the total elements added to `nums` is at most `n`
- Sorting the `nums` list: `O(m × log m)` where `m` is the length of `nums`, and in the worst case `m = n`
- Computing the sum for the first half of sorted `nums`: `O(m/2) = O(n)`

Since the sorting operation `O(n × log n)` dominates all other operations, the overall time complexity is `O(n × log n)`.

**Space Complexity:** `O(n)`

The space complexity comes from:
- Counter dictionary `cnt`: `O(k)` where `k` is the number of unique elements, at most `O(n)`
- List `nums`: Can contain at most `n` elements in the worst case when all swaps are needed
- Other variables like `mi` and `m`: `O(1)`

Therefore, the overall space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Misunderstanding the Swap Pairing Logic**

**Pitfall:** A common mistake is thinking that we need to explicitly pair fruits from basket1 with fruits from basket2 when calculating costs. Developers might try to track which specific fruits are being swapped between baskets.

**Why it happens:** The problem statement mentions swapping fruit `i` from basket1 with fruit `j` from basket2, leading to overthinking about which exact fruits to pair.

**The Issue:**
```python
# INCORRECT approach - trying to track specific swaps
basket1_excess = []
basket2_excess = []
for fruit, count in frequency_diff.items():
    if count > 0:
        basket1_excess.extend([fruit] * (count // 2))
    else:
        basket2_excess.extend([fruit] * (-count // 2))
# Then trying to pair them somehow...
```

**Solution:** The key insight is that after collecting all fruits that need swapping and sorting them, we only need to consider the first half. Each fruit in the first half implicitly pairs with a corresponding fruit in the second half, and we only need to count the cost once per swap (using the minimum of the two fruits being swapped).

### 2. **Incorrect Minimum Value Calculation**

**Pitfall:** Using `min(basket1 + basket2)` to find the minimum fruit value instead of `min(frequency_diff)`.

**Why it happens:** It seems logical to find the minimum across all original fruits.

**The Issue:**
```python
# INCORRECT
min_fruit_value = min(basket1 + basket2)  # Wrong!

# CORRECT
min_fruit_value = min(frequency_diff)  # Uses keys from the counter
```

**Solution:** The `min(frequency_diff)` correctly gets the minimum fruit value that appears in either basket because `frequency_diff` contains all unique fruit values as keys. Using `min(basket1 + basket2)` would work but is less efficient.

### 3. **Forgetting the Indirect Swap Optimization**

**Pitfall:** Only considering direct swaps and missing the optimization of using a cheap fruit as a middleman.

**The Issue:**
```python
# INCORRECT - only considering direct swaps
return sum(fruits_to_swap[:num_swaps_needed])  # Missing optimization!
```

**Why it matters:** If you have expensive fruits to swap (e.g., cost 100) but there's a cheap fruit (e.g., cost 1), it's cheaper to do two swaps through the cheap fruit (cost 2) than one direct swap (cost 100).

**Solution:** Always compare the direct swap cost with the indirect swap cost:
```python
return sum(min(fruit_value, min_fruit_value * 2) for fruit_value in fruits_to_swap[:num_swaps_needed])
```

### 4. **Processing the Entire Swap List Instead of Half**

**Pitfall:** Summing costs for all fruits in `fruits_to_swap` instead of just the first half.

**The Issue:**
```python
# INCORRECT - counts each swap twice
return sum(min(x, min_fruit_value * 2) for x in fruits_to_swap)  # Wrong!
```

**Why it happens:** Each swap involves two fruits, but we add both fruits to our list. If we process all fruits, we're essentially counting each swap twice.

**Solution:** Only process the first half after sorting:
```python
num_swaps_needed = len(fruits_to_swap) // 2
return sum(min(fruit_value, min_fruit_value * 2) for fruit_value in fruits_to_swap[:num_swaps_needed])
```

This ensures we count each swap exactly once, using the cheaper fruit from each swap pair.

