# 1815. Maximum Number of Groups Getting Fresh Donuts

> Source: https://algo.monster/liteproblems/1815

## Problem Statement

You have a donut shop that bakes donuts in batches of size `batchSize`. The shop has a strict rule: all donuts from one batch must be served before any donuts from the next batch can be served.

Multiple groups of customers will visit your shop, where `groups[i]` represents the number of customers in the i-th group. Each customer wants exactly one donut.

When a group arrives at the shop, all customers in that group must be served before any customers from the next group. A group is considered "happy" if they receive fresh donuts - meaning the first customer of their group doesn't get a donut that was left over from serving the previous group.

For example, if `batchSize = 4` and a group of 3 customers is served, there will be 1 leftover donut. If the next group arrives, they would get that 1 leftover donut first, making them unhappy since they didn't get all fresh donuts.

You can rearrange the order in which groups visit the shop to maximize happiness. Your goal is to find the maximum number of groups that can be happy.

**Key points:**
- Groups must be served completely before the next group
- A group is happy only if they start with a fresh batch (no leftovers from previous group)
- You can rearrange the visiting order of groups
- Return the maximum possible number of happy groups

## Intuition

The key insight is that a group is happy when they start with no leftover donuts. This happens when the total number of donuts served to all previous groups is a multiple of `batchSize`.

Think about it this way: if we've served exactly `12` donuts and `batchSize = 4`, we've used exactly `3` complete batches with no leftovers. The next group will be happy because they'll get fresh donuts.

This means we care about the remainder when dividing the cumulative sum of customers by `batchSize`. A group starting at position `i` is happy if the sum of all group sizes before position `i` modulo `batchSize` equals `0`.

We can make several observations:

1. **Groups with size divisible by `batchSize`**: These groups are special. If a group has exactly `batchSize` customers (or any multiple), they consume complete batches and leave no leftovers. We can greedily place all such groups first - they'll all be happy and won't affect subsequent groups.

2. **Remainder matters, not actual size**: For groups whose size isn't divisible by `batchSize`, only the remainder matters. A group of `7` customers with `batchSize = 4` has the same effect as a group of `3` customers - both leave `3` donuts for the next batch.

3. **The problem becomes arrangement of remainders**: After handling complete batches, we need to arrange groups based on their remainders `[1, 2, ..., batchSize-1]` to maximize happy groups. The first group in any arrangement is always happy (starts fresh), and subsequent groups are happy when the cumulative remainder equals `0`.

4. **State representation**: Since we only care about how many groups have each possible remainder value, we can represent the state as a frequency count of remainders. With at most `30` groups and remainder values from `1` to `batchSize-1`, we can encode this state efficiently using bit manipulation where every `5` bits represent the count of one remainder value.

The solution uses [dynamic programming](/problems/dynamic_programming_intro) with [memoization](/problems/memoization_intro) to try different arrangements of these remainder groups, tracking the running remainder to determine when a new group would be happy.

**Learn more about [Memoization](/problems/memoization_intro), [Dynamic Programming](/problems/dynamic_programming_intro) and [Bitmask](/problems/bitmask_intro) patterns.**

## Solution Approach

The solution uses **greedy strategy** combined with **state compression** and **memoized search** ([dynamic programming](/problems/dynamic_programming_intro)).

### Step 1: Handle Groups Divisible by BatchSize

First, we identify groups whose size is already a multiple of `batchSize`. These groups consume complete batches and leave no leftovers, so they can all be placed at the beginning and will all be happy:

```python
ans = 0
for v in groups:
    i = v % batchSize
    ans += i == 0  # Count groups with remainder 0
```

### Step 2: State Compression for Remaining Groups

For groups not divisible by `batchSize`, we only care about their remainder when divided by `batchSize`. We encode the frequency of each remainder into a single integer `state`:

- Use `5` bits to represent the count of each remainder value
- Remainder `i` is stored at bit positions `[i*5, i*5+4]`
- Maximum `batchSize` is `9`, so we need at most `8 * 5 = 40` bits

```python
state = 0
for v in groups:
    i = v % batchSize
    if i:  # If remainder is not 0
        state += 1 << (i * 5)  # Increment count for remainder i
```

### Step 3: Memoized Search with DFS

The core algorithm uses a recursive function `dfs(state, mod)` where:
- `state`: Current frequency distribution of remainders
- `mod`: Current cumulative remainder (sum of served groups mod `batchSize`)
- Returns: Maximum number of happy groups possible from this state

```python
@cache
def dfs(state, mod):
    res = 0
    x = int(mod == 0)  # 1 if current group will be happy, 0 otherwise
  
    for i in range(1, batchSize):
        if state >> (i * 5) & 31:  # Check if remainder i exists
            # Try using a group with remainder i
            t = dfs(state - (1 << (i * 5)), (mod + i) % batchSize)
            res = max(res, t + x)
  
    return res
```

**How the DFS works:**

1. If `mod == 0`, the next group to be served will be happy (gets fresh donuts), so `x = 1`
2. Try each possible remainder `i` from `1` to `batchSize-1`:
   - Check if we have groups with remainder `i`: `(state >> (i * 5)) & 31` extracts the 5-bit count
   - If available, remove one group with remainder `i` from state: `state - (1 << (i * 5))`
   - Update cumulative remainder: `(mod + i) % batchSize`
   - Recursively solve the subproblem
   - Add `x` to account for whether the current group is happy
3. Return the maximum happiness achievable

### Step 4: Final Answer

The total answer combines:
- Groups with size divisible by `batchSize` (all happy)
- Optimal arrangement of remaining groups found by `dfs(state, 0)`

```python
return ans + dfs(state, 0)
```

The `@cache` decorator provides [memoization](/problems/memoization_intro), preventing redundant calculations for the same `(state, mod)` pairs, which significantly improves performance.

## Example Walkthrough

Let's walk through an example with `batchSize = 4` and `groups = [1, 3, 2, 5, 2, 2, 1, 6]`.

**Step 1: Identify and handle groups divisible by batchSize**

First, we check which groups have sizes divisible by 4:
- Group of 1: `1 % 4 = 1` (remainder 1)
- Group of 3: `3 % 4 = 3` (remainder 3)
- Group of 2: `2 % 4 = 2` (remainder 2)
- Group of 5: `5 % 4 = 1` (remainder 1)
- Group of 2: `2 % 4 = 2` (remainder 2)
- Group of 2: `2 % 4 = 2` (remainder 2)
- Group of 1: `1 % 4 = 1` (remainder 1)
- Group of 6: `6 % 4 = 2` (remainder 2)

No groups are divisible by 4, so `ans = 0`.

**Step 2: Build state representation**

Count the frequency of each remainder:
- Remainder 1: 3 groups (sizes 1, 5, 1)
- Remainder 2: 4 groups (sizes 2, 2, 2, 6)
- Remainder 3: 1 group (size 3)

Encode this into state using 5 bits per remainder:
- Remainder 1 at bits [5-9]: `3 << 5 = 96`
- Remainder 2 at bits [10-14]: `4 << 10 = 4096`
- Remainder 3 at bits [15-19]: `1 << 15 = 32768`
- Total state: `96 + 4096 + 32768 = 36960`

**Step 3: Execute DFS search**

Start with `dfs(36960, 0)`:

Since `mod = 0`, the first group we place will be happy (`x = 1`).

We try different first groups:

**Option A: Start with remainder 1**
- Remove one group with remainder 1: new state = `36960 - 32 = 36928`
- New mod = `(0 + 1) % 4 = 1`
- Recursively solve `dfs(36928, 1)`
- In this recursive call, `mod = 1`, so next group won't be happy (`x = 0`)

**Option B: Start with remainder 2**
- Remove one group with remainder 2: new state = `36960 - 1024 = 35936`
- New mod = `(0 + 2) % 4 = 2`
- Recursively solve `dfs(35936, 2)`

**Option C: Start with remainder 3**
- Remove one group with remainder 3: new state = `36960 - 32768 = 4192`
- New mod = `(0 + 3) % 4 = 3`
- Recursively solve `dfs(4192, 3)`

Let's trace one promising path: Start with remainder 3, then remainder 1:
1. Place group with remainder 3: `mod = 3`, this group is happy
2. Place group with remainder 1: `mod = (3 + 1) % 4 = 0`, this group is happy
3. Place group with remainder 2: `mod = (0 + 2) % 4 = 2`, this group is happy
4. Place group with remainder 2: `mod = (2 + 2) % 4 = 0`, this group is not happy
5. Continue placing remaining groups...

The algorithm explores all possible arrangements. One optimal arrangement might be:
`[3] → [1] → [2, 2] → [1, 1, 2, 2]`

Where groups in brackets are served together to reach `mod = 0`:
- Group with remainder 3: happy (starts fresh)
- Group with remainder 1: happy (cumulative = 4, mod = 0)
- First group with remainder 2: happy (starts fresh after mod = 0)
- Second group with remainder 2: not happy (cumulative = 2)
- Continue analysis...

**Step 4: Return final answer**

The DFS will find that the maximum number of happy groups is 4, achieved by carefully arranging the groups so that the cumulative remainder becomes 0 as often as possible.

Final answer: `0 + 4 = 4` happy groups.

## Python Solution

```python
class Solution:
    def maxHappyGroups(self, batchSize: int, groups: List[int]) -> int:
        from functools import cache
        from typing import List
      
        @cache
        def dfs(compressed_state: int, current_remainder: int) -> int:
            """
            Dynamic programming function to find maximum happy groups.
          
            Args:
                compressed_state: Bit-packed representation of remaining groups
                current_remainder: Current remainder from previous groups
          
            Returns:
                Maximum number of happy groups achievable from this state
            """
            max_happy_groups = 0
          
            # If current remainder is 0, the next group will be happy
            bonus_if_happy = int(current_remainder == 0)
          
            # Try using each possible remainder group (1 to batchSize-1)
            for remainder in range(1, batchSize):
                # Extract count of groups with this remainder from compressed state
                # Each remainder uses 5 bits in the state
                bit_position = remainder * 5
                count_of_remainder = (compressed_state >> bit_position) & 31
              
                if count_of_remainder > 0:
                    # Remove one group with this remainder from state
                    new_state = compressed_state - (1 << bit_position)
                  
                    # Calculate new remainder after adding this group
                    new_remainder = (current_remainder + remainder) % batchSize
                  
                    # Recursively solve for remaining groups
                    happy_from_remaining = dfs(new_state, new_remainder)
                  
                    # Update maximum happy groups
                    max_happy_groups = max(max_happy_groups, 
                                          happy_from_remaining + bonus_if_happy)
          
            return max_happy_groups
      
        # Initialize compressed state and count groups divisible by batchSize
        compressed_state = 0
        happy_groups_count = 0
      
        # Process each group
        for group_size in groups:
            remainder = group_size % batchSize
          
            # Groups divisible by batchSize are always happy
            if remainder == 0:
                happy_groups_count += 1
            else:
                # Pack non-zero remainder groups into compressed state
                # Each remainder gets 5 bits (can store count up to 31)
                compressed_state += 1 << (remainder * 5)
      
        # Add maximum happy groups from non-zero remainder groups
        happy_groups_count += dfs(compressed_state, 0)
      
        return happy_groups_count
```

## Time and Space Complexity

**Time Complexity:** `O(batchSize * 31^(batchSize-1))`

The analysis breaks down as follows:
- The state is represented using 5 bits for each remainder value from 1 to `batchSize-1`, allowing up to 31 occurrences of each remainder
- The total number of possible states is at most `31^(batchSize-1)` since we have `batchSize-1` different remainder values (excluding 0)
- For each state, the DFS function iterates through `batchSize-1` possible remainders
- Each state is computed only once due to memoization with `@cache`
- Therefore, the time complexity is `O(batchSize * 31^(batchSize-1))`

For typical constraints where `batchSize ≤ 9`, this gives us `31^8 * 9 ≈ 8.5 * 10^11` in the worst case, but in practice, many states are unreachable (since the total number of groups is limited), keeping the actual complexity well below `O(10^7)`.

**Space Complexity:** `O(31^(batchSize-1))`

The space analysis:
- The memoization cache stores at most `31^(batchSize-1)` different states
- Each cached entry stores the state tuple and the result value
- The recursion depth is at most the total number of groups, which is typically much smaller than the number of states
- For `batchSize ≤ 9`, this gives us `31^8 ≈ 9.5 * 10^10` theoretical states, but the actual number of valid states encountered is significantly smaller due to the limited number of groups

In practice, with reasonable constraints on the number of groups (typically ≤ 30), the actual space used is much less than the theoretical maximum, keeping it below `O(10^6)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect State Representation - Using Lists or Tuples Instead of Bit Compression**

Many developers initially try to use a list or tuple to track remainder frequencies:

```python
# WRONG APPROACH - Will cause Time Limit Exceeded
@cache
def dfs(remainder_counts, current_remainder):
    remainder_counts = list(remainder_counts)  # Convert tuple for modification
    # ... rest of logic
```

**Why this fails:**
- Python's `@cache` requires hashable arguments
- Converting between lists and tuples repeatedly is inefficient
- The state space becomes much larger with tuple representations

**Solution:** Use bit compression as shown in the correct solution. Each remainder gets 5 bits, allowing efficient state representation in a single integer.

### 2. **Forgetting to Handle Groups Divisible by BatchSize Separately**

A common mistake is including groups with remainder 0 in the DFS state:

```python
# WRONG - Including remainder 0 in DFS
for group_size in groups:
    remainder = group_size % batchSize
    compressed_state += 1 << (remainder * 5)  # Includes remainder 0!
```

**Why this fails:**
- Groups with remainder 0 always leave the "plate clean" for the next group
- They can all be placed at the beginning and all be happy
- Including them in DFS unnecessarily complicates the state space

**Solution:** Count groups with remainder 0 separately and add them directly to the answer.

### 3. **Incorrect Bit Manipulation for Extracting/Updating Counts**

Developers often make mistakes with bit operations:

```python
# WRONG - Incorrect extraction
count = compressed_state >> (remainder * 5)  # Missing mask!

# WRONG - Incorrect decrement
new_state = compressed_state - remainder  # Should subtract (1 << bit_position)
```

**Solution:** 
- Extract count: `(compressed_state >> (remainder * 5)) & 31`
- Decrement count: `compressed_state - (1 << (remainder * 5))`

### 4. **Not Understanding When a Group is Happy**

A critical misunderstanding is when to count a group as happy:

```python
# WRONG - Counting happiness after serving the group
def dfs(state, mod):
    for remainder in range(1, batchSize):
        if has_remainder(state, remainder):
            new_mod = (mod + remainder) % batchSize
            # Wrong: checking if new_mod == 0
            bonus = int(new_mod == 0)  # INCORRECT!
```

**Why this fails:**
- A group is happy if they START with fresh donuts (mod == 0 BEFORE serving)
- Not if they FINISH exactly at a batch boundary

**Solution:** Check `mod == 0` at the beginning of each recursive call, before choosing which group to serve next.

### 5. **Off-by-One Error with Remainder Range**

```python
# WRONG - Including batchSize as a possible remainder
for remainder in range(1, batchSize + 1):  # WRONG!
    # Process remainder...
```

**Why this fails:**
- Remainders range from 0 to batchSize-1
- A remainder of batchSize would actually be 0

**Solution:** Use `range(1, batchSize)` since remainder 0 is handled separately.

