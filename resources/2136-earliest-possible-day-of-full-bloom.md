# 2136. Earliest Possible Day of Full Bloom

> Source: https://algo.monster/liteproblems/2136

## Problem Statement

You have `n` flower seeds that need to be planted and grown. Each seed must be planted first before it can start growing and eventually bloom.

For each seed `i`:
- `plantTime[i]` represents the number of full days required to plant that seed. You can only work on planting one seed per day, and you don't need to plant the same seed on consecutive days - you just need to accumulate `plantTime[i]` total days of work for that seed to be fully planted.
- `growTime[i]` represents the number of full days the seed needs to grow after being completely planted. Once the growth period is complete, the flower blooms and remains bloomed forever.

Starting from day `0`, you can choose to plant the seeds in any order you want. Your goal is to find the earliest possible day when all flowers have bloomed.

For example, if you have a seed with `plantTime = 3` and `growTime = 2`, and you start planting it on day 0, you'll finish planting on day 3 (after working days 0, 1, and 2). The seed will then grow for 2 more days and bloom on day 5.

The key insight is that while you're planting seeds sequentially (one at a time), previously planted seeds can grow in parallel. This means the order in which you plant seeds matters - you want to minimize the overall time until the last flower blooms.

## Intuition

Let's think about what happens when we plant seeds. Since we can only plant one seed at a time, the total planting time is fixed - it's always the sum of all `plantTime[i]`. What we can control is the order of planting, which affects when each seed starts and finishes growing.

Consider two seeds A and B:
- Seed A: `plantTime = 2`, `growTime = 5`
- Seed B: `plantTime = 3`, `growTime = 1`

If we plant A first then B:
- A finishes planting on day 2, blooms on day 2 + 5 = 7
- B finishes planting on day 2 + 3 = 5, blooms on day 5 + 1 = 6
- All flowers bloom by day 7

If we plant B first then A:
- B finishes planting on day 3, blooms on day 3 + 1 = 4
- A finishes planting on day 3 + 2 = 5, blooms on day 5 + 5 = 10
- All flowers bloom by day 10

The first order is better! Why? Because seed A has a longer growth time, and by planting it first, we allow it to start its long growth period earlier while we're still planting other seeds.

This reveals the key insight: seeds with longer growth times should be planted first. While we're busy planting the remaining seeds, the early-planted seeds with long growth times are already growing in parallel. This minimizes the "tail" time - the growth period of the last seed that determines when all flowers have bloomed.

Think of it like cooking multiple dishes: you'd start with the dish that takes longest to cook first, so it can be cooking while you prepare the others. The same principle applies here - start the seeds with longest growth times first so they can grow while you're still planting others.

Therefore, the optimal strategy is to sort seeds by `growTime` in descending order and plant them in that sequence. For each seed, we track when it will bloom (`plantingEndTime + growTime`) and keep the maximum bloom time across all seeds.

**Learn more about [Greedy](/problems/greedy_intro) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

Based on our intuition that seeds with longer growth times should be planted first, we implement a [greedy](/problems/greedy_intro) algorithm with [sorting](/problems/sorting_summary).

**Step 1: Sort the seeds**
We pair each seed's `plantTime` and `growTime` together and sort them by `growTime` in descending order. This is achieved using:
```python
sorted(zip(plantTime, growTime), key=lambda x: -x[1])
```
The `zip` function pairs corresponding elements, and `key=lambda x: -x[1]` sorts by the second element (growTime) in descending order.

**Step 2: Simulate the planting process**
We iterate through the sorted seeds and track two key variables:
- `t`: The current day, representing when we finish planting the current seed
- `ans`: The earliest day when all planted seeds so far will have bloomed

For each seed with planting time `pt` and growth time `gt`:
1. We update the current day: `t += pt` (we spend `pt` days planting this seed)
2. We calculate when this seed will bloom: `t + gt` (finish planting day + growth time)
3. We update our answer: `ans = max(ans, t + gt)` (keep track of the latest bloom time)

**Why this works:**
- By planting seeds with longer growth times first, we ensure they start growing while we're still planting other seeds
- The `max` operation ensures we track the latest blooming flower, which determines when all flowers have bloomed
- Each seed's bloom time is its planting completion time plus its growth time

**Time Complexity:** `O(n log n)` due to [sorting](/problems/sorting_summary), where `n` is the number of seeds.

**Space Complexity:** `O(n)` for storing the zipped pairs during [sorting](/problems/sorting_summary).

The algorithm guarantees the minimum possible bloom time for all flowers by prioritizing seeds that need more time to grow, allowing them to grow in parallel while we plant the remaining seeds.

## Example Walkthrough

Let's walk through a concrete example with 3 seeds:
- Seed 0: `plantTime = 1`, `growTime = 4`
- Seed 1: `plantTime = 2`, `growTime = 3`
- Seed 2: `plantTime = 3`, `growTime = 1`

**Step 1: Sort by growth time (descending)**

After sorting by `growTime` in descending order:
- Seed 0: `plantTime = 1`, `growTime = 4` (highest growth time)
- Seed 1: `plantTime = 2`, `growTime = 3`
- Seed 2: `plantTime = 3`, `growTime = 1` (lowest growth time)

**Step 2: Simulate planting in this order**

Initialize: `t = 0` (current day), `ans = 0` (latest bloom day)

**Plant Seed 0:**
- Planting takes 1 day: `t = 0 + 1 = 1`
- This seed blooms on day: `1 + 4 = 5`
- Update answer: `ans = max(0, 5) = 5`
- Status: Seed 0 planted on days [0], will bloom on day 5

**Plant Seed 1:**
- Planting takes 2 days: `t = 1 + 2 = 3`
- This seed blooms on day: `3 + 3 = 6`
- Update answer: `ans = max(5, 6) = 6`
- Status: Seed 1 planted on days [1, 2], will bloom on day 6

**Plant Seed 2:**
- Planting takes 3 days: `t = 3 + 3 = 6`
- This seed blooms on day: `6 + 1 = 7`
- Update answer: `ans = max(6, 7) = 7`
- Status: Seed 2 planted on days [3, 4, 5], will bloom on day 7

**Result:** All flowers bloom by day 7.

**Visual Timeline:**
```
Day: 0  1  2  3  4  5  6  7
S0:  [P][---grow---][bloom]
S1:     [P  P][--grow--][bloom]
S2:           [P  P  P][g][bloom]
```
Where P = planting, grow = growing period, bloom = flower has bloomed

By planting the seed with the longest growth time first, we minimize the total time. If we had planted them in reverse order (shortest growth time first), the seed with `growTime = 4` would start growing much later, resulting in all flowers blooming on day 10 instead of day 7.

## Python Solution

```python
class Solution:
    def earliestFullBloom(self, plantTime: List[int], growTime: List[int]) -> int:
        # Initialize variables to track the earliest bloom completion time
        earliest_bloom_day = 0  # The earliest day when all flowers have bloomed
        current_planting_day = 0  # Current day when we finish planting
      
        # Create pairs of (plantTime, growTime) and sort by growTime in descending order
        # Strategy: Plant flowers with longer grow times first to minimize total time
        flower_pairs = zip(plantTime, growTime)
        sorted_flowers = sorted(flower_pairs, key=lambda flower: -flower[1])
      
        # Process each flower in the optimal order
        for plant_duration, grow_duration in sorted_flowers:
            # Update the day when we finish planting this flower
            current_planting_day += plant_duration
          
            # Calculate when this flower will bloom (planting end day + grow time)
            bloom_day = current_planting_day + grow_duration
          
            # Update the earliest day when all flowers have bloomed
            earliest_bloom_day = max(earliest_bloom_day, bloom_day)
      
        return earliest_bloom_day
```

## Time and Space Complexity

**Time Complexity:** `O(n log n)`

The dominant operation in this algorithm is the sorting step. The `sorted()` function is called on a zip object containing `n` elements (where `n` is the number of seeds), which requires `O(n log n)` time. The subsequent iteration through the sorted list takes `O(n)` time, but this is overshadowed by the sorting complexity. Therefore, the overall time complexity is `O(n log n)`.

**Space Complexity:** `O(n)`

The space complexity arises from creating the zipped list of tuples from `plantTime` and `growTime`, which contains `n` elements. The `sorted()` function creates a new list of size `n` to store the sorted result. Although the zip object itself is an iterator and doesn't consume extra space proportional to `n`, the sorted list does require `O(n)` additional space. Hence, the space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Incorrectly Sorting by Plant Time Instead of Grow Time

A common mistake is sorting by `plantTime` rather than `growTime`, or sorting in the wrong order (ascending instead of descending).

**Incorrect approach:**
```python
# Wrong: Sorting by plantTime
sorted_flowers = sorted(zip(plantTime, growTime), key=lambda x: x[0])

# Wrong: Sorting growTime in ascending order
sorted_flowers = sorted(zip(plantTime, growTime), key=lambda x: x[1])
```

**Why this fails:**
Consider two seeds:
- Seed A: plantTime = 1, growTime = 10
- Seed B: plantTime = 10, growTime = 1

If we plant A first: Day 1 (finish planting A) + 10 (grow) = Day 11 for A to bloom
Then plant B: Day 11 (finish planting B) + 1 (grow) = Day 12 for B to bloom
Total: 12 days

If we plant B first: Day 10 (finish planting B) + 1 (grow) = Day 11 for B to bloom
Then plant A: Day 11 (finish planting A) + 10 (grow) = Day 21 for A to bloom
Total: 21 days

The correct approach (sorting by growTime descending) gives us 12 days, while the wrong approach gives us 21 days.

### Pitfall 2: Forgetting to Track the Maximum Bloom Time

Another mistake is only tracking the last flower's bloom time instead of the maximum bloom time across all flowers.

**Incorrect approach:**
```python
for plant_duration, grow_duration in sorted_flowers:
    current_planting_day += plant_duration
    # Wrong: Only considering the last flower
    earliest_bloom_day = current_planting_day + grow_duration
```

**Why this fails:**
Earlier planted flowers might bloom later than subsequently planted ones. For example:
- Seed 1: plantTime = 1, growTime = 100 → blooms on day 101
- Seed 2: plantTime = 1, growTime = 1 → blooms on day 3

The last flower (Seed 2) blooms on day 3, but we need to wait until day 101 for all flowers to bloom.

**Correct approach:**
Always use `max()` to track the latest bloom time among all flowers:
```python
earliest_bloom_day = max(earliest_bloom_day, current_planting_day + grow_duration)
```

### Pitfall 3: Misunderstanding the Problem Constraints

Some might think you can plant multiple seeds simultaneously or that growth doesn't happen in parallel.

**Key clarifications:**
- You can only plant one seed at a time (sequential planting)
- Once a seed is planted, it grows independently (parallel growth)
- The planting of a seed requires `plantTime[i]` cumulative days of work, not necessarily consecutive

The solution correctly handles these constraints by accumulating planting time sequentially while allowing previously planted seeds to grow in parallel.

