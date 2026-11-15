# 3017. Count the Number of Houses at a Certain Distance II

> Source: https://algo.monster/liteproblems/3017

## Problem Statement

You have a city with `n` houses numbered from `1` to `n`. These houses are connected by streets in the following way:

1. There's a street connecting each consecutive pair of houses: house `i` connects to house `i + 1` for all `1 ≤ i ≤ n - 1`
2. There's one additional street that directly connects house `x` to house `y`

This creates a [graph](/problems/graph_intro) where houses form a linear path from `1` to `n`, with one extra connection creating a potential shortcut between houses `x` and `y`.

Your task is to find, for each distance `k` from `1` to `n`, how many pairs of houses `(house₁, house₂)` have exactly `k` as the minimum number of streets needed to travel between them.

The output should be an array `result` of length `n` where:
- `result[k]` = the total number of house pairs that are exactly `k` streets apart (using the shortest path)
- The array is 1-indexed, meaning `result[1]` represents pairs that are 1 street apart, `result[2]` represents pairs that are 2 streets apart, and so on

Important notes:
- `x` and `y` can be equal (meaning the extra street connects a house to itself, which doesn't add any new path)
- Each pair `(house₁, house₂)` where `house₁ ≠ house₂` should be counted in the result
- The distance is measured as the minimum number of streets to traverse, considering both the linear connections and the extra street between `x` and `y`

For example, if `n = 4`, `x = 1`, `y = 3`, the houses are connected as: `1-2-3-4` with an extra street `1-3`. The shortest distances between all pairs would need to be calculated considering both the direct path and any path using the shortcut.

## Intuition

The key insight is that the extra street between houses `x` and `y` transforms part of the linear path into a cycle. Without this extra street, we'd have a simple line [graph](/problems/graph_intro) where the distance between any two houses `i` and `j` is just `|i - j|`. But with the extra street, we create a cycle that can serve as a shortcut.

Let's think about what happens when we add the edge between `x` and `y`:
- If `|x - y| ≤ 1`, the extra street doesn't create any shortcuts (it either connects a house to itself or connects two already adjacent houses), so we maintain the linear structure.
- If `|x - y| > 1`, we form a cycle of length `|x - y| + 1` (the houses from `x` to `y` plus the direct edge back).

The [graph](/problems/graph_intro) now consists of three parts:
1. A "tail" before the cycle: houses `1` to `x-1`
2. The cycle itself: houses from `x` to `y`
3. Another "tail" after the cycle: houses `y+1` to `n`

For any pair of houses, the shortest path might:
- Stay entirely within one tail (using the linear path)
- Go from one tail into the cycle
- Go through the cycle to reach the other tail
- Stay within the cycle (where we can go either clockwise or counterclockwise)

The solution breaks down the problem by:
1. First handling the case without the cycle (when `|x - y| ≤ 1`)
2. Computing distances as if we had a shorter linear [graph](/problems/graph_intro) of length `n - cycle_len + 2` (collapsing the cycle into just 2 nodes)
3. Adding the contribution from pairs within the cycle itself
4. Adjusting for the interactions between each tail and the cycle

The cycle creates interesting distance patterns. For example, in a cycle of even length, opposite nodes are equidistant going either direction, while in odd-length cycles, all pairs have a unique shortest path direction. This is why the solution treats even and odd cycle lengths differently with conditions like `cycle_len & 1`.

The approach systematically counts pairs at each distance by considering these three components and their interactions, building up the final count array by adding contributions from each part of the structure.

**Learn more about [Graph](/problems/graph_intro) and [Prefix Sum](/problems/subarray_sum) patterns.**

## Solution Approach

The implementation handles three distinct cases based on the structure created by the extra street:

### Case 1: No Effective Cycle (`|x - y| ≤ 1`)
When the distance between `x` and `y` is at most 1, the extra street doesn't create any shortcuts. The [graph](/problems/graph_intro) remains effectively linear, so for each distance `k`, there are exactly `n - k` pairs of houses that are `k` streets apart. Since we count both `(i, j)` and `(j, i)`, we multiply by 2:
```python
if abs(x - y) <= 1:
    return [2 * x for x in reversed(range(n))]
```
This gives us `[2*(n-1), 2*(n-2), ..., 2*1, 0]`.

### Case 2: [Graph](/problems/graph_intro) with Cycle
When `|x - y| > 1`, we form a cycle and decompose the problem:

**Step 1: Collapse the cycle**
```python
cycle_len = abs(x - y) + 1
n2 = n - cycle_len + 2
res = [2 * x for x in reversed(range(n2))]
```
We treat the cycle as if it were just 2 nodes, creating a linear [graph](/problems/graph_intro) of length `n2`. This accounts for distances in the "collapsed" version.

**Step 2: Add cycle internal distances**
```python
res2 = [cycle_len * 2] * (cycle_len >> 1)
if not cycle_len & 1:  # Even cycle
    res2[-1] = cycle_len
res2[0] -= 2
```
- For a cycle, pairs at distance `d` (where `d ≤ cycle_len/2`) appear with specific frequencies
- In even-length cycles, antipodal pairs (at distance `cycle_len/2`) have both paths equal, so we count them once
- We subtract 2 from `res2[0]` to avoid double-counting the cycle's entry/exit points

**Step 3: Handle tail-to-cycle interactions**
```python
tail1 = x - 1  # Left tail length
tail2 = n - y  # Right tail length
```

For each tail, we calculate additional distances created by paths that go from the tail into the cycle:
- `i_mx`: Maximum distance achievable from this tail through the cycle
- `val_mx`: Maximum contribution value based on cycle and tail sizes
- The pattern fills distances based on how far into the cycle we can reach

The algorithm builds `res3` array for each tail:
```python
res3 = [val_mx] * i_mx
# Set initial values
res3[0] = 0
res3[1] = 0
# Fill pattern based on cycle properties
for i, j in enumerate(range(4, val_mx, 4)):
    res3[i + 2] = j
    res3[i_mx2 - i - 1] = j
```

**Step 4: Adjust for direct tail connections**
```python
for i in range(1, tail + 1):
    res3[i] += 2
```
This adds the direct connections within each tail.

**Step 5: Handle even cycle special case**
```python
if not cycle_len & 1:
    mn = cycle_len >> 1
    for i in range(mn, mn + tail):
        res3[i] += 2
```
For even-length cycles, additional adjustments are needed for distances involving the midpoint of the cycle.

### Final Assembly
The solution accumulates all contributions:
1. Base distances from the collapsed [graph](/problems/graph_intro)
2. Internal cycle distances
3. Tail-to-cycle interaction distances for both tails

The result is a complete count of all pairs at each distance from 1 to `n`, properly accounting for the shortcut created by the extra street between houses `x` and `y`.

## Example Walkthrough

Let's walk through a concrete example with `n = 6`, `x = 2`, `y = 5`.

**Step 1: Visualize the graph**
```
Original linear path: 1 - 2 - 3 - 4 - 5 - 6
Extra street: 2 --- 5
Final graph: 1 - 2 - 3 - 4 - 5 - 6
                 └───────────┘
```
This creates a cycle of length 4 (houses 2, 3, 4, 5).

**Step 2: Identify the structure**
- Left tail: house 1 (length = 1)
- Cycle: houses 2, 3, 4, 5 (length = 4)  
- Right tail: house 6 (length = 1)

**Step 3: Calculate base distances (collapsed graph)**
Collapse the cycle into 2 nodes: `n2 = 6 - 4 + 2 = 4`
Base array for linear graph of length 4:
```
res = [2*3, 2*2, 2*1, 0] = [6, 4, 2, 0]
```

**Step 4: Add cycle internal distances**
For the 4-cycle, distances between pairs:
- 2↔3: 1, 3↔4: 1, 4↔5: 1, 5↔2: 1 (4 pairs at distance 1)
- 2↔4: 2, 3↔5: 2 (2 pairs at distance 2)

```
res2 = [4*2, 4/2] = [8, 2]
res2[0] -= 2 = [6, 2]  // Adjust for entry/exit points
```

**Step 5: Handle tail interactions**

For left tail (house 1):
- 1→2: distance 1 (direct)
- 1→3: distance 2 (via 2)
- 1→4: distance 2 (via 2→5→4, using shortcut)
- 1→5: distance 1 (via 2→5, using shortcut)

For right tail (house 6):
- 6→5: distance 1 (direct)
- 6→4: distance 2 (via 5)
- 6→3: distance 2 (via 5→2→3, using shortcut)
- 6→2: distance 1 (via 5→2, using shortcut)

**Step 6: Compute final distances**
Let's verify all pairs and their shortest distances:

| Pair | Direct Path | Using Shortcut | Min Distance |
|------|------------|----------------|--------------|
| 1↔2 | 1 | - | 1 |
| 1↔3 | 2 | 2 | 2 |
| 1↔4 | 3 | 2 (1→2→5→4) | 2 |
| 1↔5 | 4 | 1 (1→2→5) | 1 |
| 1↔6 | 5 | 2 (1→2→5→6) | 2 |
| 2↔3 | 1 | 3 | 1 |
| 2↔4 | 2 | 2 | 2 |
| 2↔5 | 3 | 1 (shortcut) | 1 |
| 2↔6 | 4 | 2 (2→5→6) | 2 |
| 3↔4 | 1 | 3 | 1 |
| 3↔5 | 2 | 2 | 2 |
| 3↔6 | 3 | 3 | 3 |
| 4↔5 | 1 | 3 | 1 |
| 4↔6 | 2 | 2 | 2 |
| 5↔6 | 1 | - | 1 |

**Final count by distance:**
- Distance 1: pairs (1,2), (1,5), (2,3), (2,5), (3,4), (4,5), (5,6) = 7 pairs × 2 = 14
- Distance 2: pairs (1,3), (1,4), (1,6), (2,4), (2,6), (3,5), (4,6) = 7 pairs × 2 = 14
- Distance 3: pairs (3,6) = 1 pair × 2 = 2
- Distance 4, 5, 6: 0 pairs

**Result:** `[14, 14, 2, 0, 0, 0]`

The algorithm efficiently computes this by:
1. Starting with the collapsed graph contribution
2. Adding cycle-internal pair counts
3. Adjusting for paths from tails through the cycle
4. Accounting for the shortcut's impact on all reachable pairs

## Python Solution

```python
class Solution:
    def countOfPairs(self, n: int, x: int, y: int) -> List[int]:
        # If x and y are adjacent or the same, no special edge effect
        if abs(x - y) <= 1:
            # Simple chain graph: each distance k appears 2*(n-k) times
            return [2 * distance for distance in reversed(range(n))]
      
        # Calculate the cycle length formed by the special edge
        cycle_length = abs(x - y) + 1
      
        # Number of nodes after contracting the cycle to a single supernode
        contracted_nodes = n - cycle_length + 2
      
        # Initialize result with distances in the contracted graph
        result = [2 * distance for distance in reversed(range(contracted_nodes))]
      
        # Pad with zeros to reach n elements
        while len(result) < n:
            result.append(0)
      
        # Calculate contribution from pairs within the cycle
        cycle_pairs = cycle_length // 2
        cycle_contribution = [cycle_length * 2] * cycle_pairs
      
        # Handle even cycle length case
        if cycle_length % 2 == 0:
            cycle_contribution[-1] = cycle_length
      
        # Adjust for endpoints
        cycle_contribution[0] -= 2
      
        # Add cycle contributions to result
        for i in range(len(cycle_contribution)):
            result[i] += cycle_contribution[i]
      
        # Ensure x < y for consistent processing
        if x > y:
            x, y = y, x
      
        # Calculate tail lengths (nodes outside the cycle)
        left_tail_length = x - 1
        right_tail_length = n - y
      
        # Process each tail
        for tail_length in (left_tail_length, right_tail_length):
            if tail_length == 0:
                continue
          
            # Maximum index affected by this tail
            max_index = tail_length + (cycle_length // 2)
          
            # Maximum value contribution
            max_value = 4 * min((cycle_length - 3) // 2, tail_length)
          
            # Secondary maximum index for even cycle case
            max_index_2 = max_index - (1 - (cycle_length % 2))
          
            # Initialize tail contribution array
            tail_contribution = [max_value] * max_index
            tail_contribution[0] = 0
            tail_contribution[1] = 0
          
            # Special case for even cycle length
            if cycle_length % 2 == 0:
                tail_contribution[-1] = 0
          
            # Build symmetric pattern of contributions
            for i, value in enumerate(range(4, max_value, 4)):
                tail_contribution[i + 2] = value
                tail_contribution[max_index_2 - i - 1] = value
          
            # Add linear contributions from tail nodes
            for i in range(1, tail_length + 1):
                tail_contribution[i] += 2
          
            # Handle even cycle length special case
            if cycle_length % 2 == 0:
                min_distance = cycle_length // 2
                for i in range(min_distance, min_distance + tail_length):
                    tail_contribution[i] += 2
          
            # Add tail contributions to final result
            for i in range(len(tail_contribution)):
                result[i] += tail_contribution[i]
      
        return result
```

## Time and Space Complexity

**Time Complexity:** `O(n)`

The algorithm consists of several main parts:
- Initial setup and list creation with `[2 * x for x in reversed(range(n2))]` takes `O(n2)` time where `n2 = n - cycle_len + 2`, which is `O(n)` in the worst case
- The first loop for `res2` runs for `cycle_len >> 1` iterations, which is at most `O(n)` 
- Processing two tails (`tail1` and `tail2`), each involving:
  - Creating `res3` list with size `i_mx` which is `O(tail + cycle_len/2)` = `O(n)`
  - Multiple loops that iterate over the tail length or portions of `res3`, each taking at most `O(n)` time
  - The nested structure with `enumerate(range(4, val_mx, 4))` iterates `O(tail)` times
- Final accumulation loops are all bounded by `O(n)`

Since all operations are sequential and each is bounded by `O(n)`, the overall time complexity is `O(n)`.

**Space Complexity:** `O(n)`

The space usage includes:
- `res` list of size `n`: `O(n)`
- `res2` list of size `cycle_len >> 1`: `O(n)` in worst case
- `res3` list created for each tail with size `i_mx = tail + (cycle_len >> 1)`: `O(n)` in worst case
- Other variables use constant space: `O(1)`

The lists are not all allocated simultaneously for their maximum sizes, but the dominant space usage is the `res` array and temporary arrays that can be up to `O(n)` in size. Therefore, the overall space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrectly Handling the Case When x > y**

One of the most common mistakes is not properly normalizing the input when `x > y`. The algorithm assumes `x < y` for consistent processing of left and right tails.

**Pitfall Example:**
```python
# Wrong: Using x and y directly without normalization
left_tail = x - 1
right_tail = n - y
```

**Solution:**
Always ensure `x < y` before processing:
```python
if x > y:
    x, y = y, x
left_tail = x - 1
right_tail = n - y
```

### 2. **Off-by-One Errors in Cycle Length Calculation**

The cycle length should include both endpoints plus the path between them.

**Pitfall Example:**
```python
# Wrong: Forgetting to add 1
cycle_length = abs(x - y)  # Missing the +1
```

**Solution:**
```python
cycle_length = abs(x - y) + 1
```

### 3. **Mishandling Even vs Odd Cycle Length**

Even-length cycles have special properties where antipodal nodes have two equal-length shortest paths. This requires different handling for distance counting.

**Pitfall Example:**
```python
# Wrong: Treating all cycles the same
cycle_contribution = [cycle_length * 2] * (cycle_length // 2)
# Missing the special case for even cycles
```

**Solution:**
```python
cycle_contribution = [cycle_length * 2] * (cycle_length // 2)
if cycle_length % 2 == 0:  # Even cycle special case
    cycle_contribution[-1] = cycle_length  # Antipodal pairs counted once
```

### 4. **Incorrect Array Indexing When Building Tail Contributions**

The symmetric pattern for tail contributions requires careful index calculation, especially for the secondary maximum index.

**Pitfall Example:**
```python
# Wrong: Incorrect calculation of max_index_2
max_index_2 = max_index - 1  # Should consider cycle parity
```

**Solution:**
```python
max_index_2 = max_index - (1 - (cycle_length % 2))
```

### 5. **Forgetting to Pad the Result Array**

The result array must have exactly `n` elements, but the contracted graph might produce fewer distances.

**Pitfall Example:**
```python
# Wrong: Returning array with incorrect length
result = [2 * distance for distance in reversed(range(contracted_nodes))]
return result  # May have fewer than n elements
```

**Solution:**
```python
result = [2 * distance for distance in reversed(range(contracted_nodes))]
while len(result) < n:
    result.append(0)
```

### 6. **Double-Counting Cycle Endpoints**

When calculating cycle contributions, the entry and exit points of the cycle should not be double-counted with the contracted graph distances.

**Pitfall Example:**
```python
# Wrong: Not adjusting for endpoints
cycle_contribution[0] = cycle_length * 2  # Should subtract 2
```

**Solution:**
```python
cycle_contribution[0] -= 2  # Avoid double-counting endpoints
```

### 7. **Misunderstanding the Problem's Distance Definition**

The problem asks for the number of pairs at each distance, not just counting unique distances. Each pair `(i, j)` where `i ≠ j` should be counted.

**Pitfall Example:**
```python
# Wrong: Counting only one direction
return [x for x in reversed(range(n))]  # Missing factor of 2
```

**Solution:**
```python
return [2 * x for x in reversed(range(n))]  # Count both (i,j) and (j,i)
```

