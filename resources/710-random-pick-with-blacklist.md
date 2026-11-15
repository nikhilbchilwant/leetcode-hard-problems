# 710. Random Pick with Blacklist

> Source: https://algo.monster/liteproblems/710

## Problem Statement

You need to design a random number generator that picks integers from the range `[0, n-1]`, but excludes certain numbers specified in a blacklist.

Given:
- An integer `n` representing the upper bound of the range (exclusive)
- An array `blacklist` containing unique integers that should never be returned

The task requires implementing a `Solution` class with two methods:

1. **Constructor** `Solution(int n, int[] blacklist)`: Initializes the random number generator with the range size `n` and the list of forbidden numbers
2. **Pick method** `pick()`: Returns a random integer from `[0, n-1]` that is not in the blacklist

Key requirements:
- All valid numbers (those in range and not blacklisted) must have equal probability of being selected
- The algorithm should minimize calls to the built-in random function

The solution uses a remapping strategy:
- It calculates `k` as the count of valid numbers (`n - len(blacklist)`)
- Creates a mapping dictionary `d` to handle blacklisted numbers in the range `[0, k-1]`
- For each blacklisted number less than `k`, it maps it to a valid number in the range `[k, n-1]`
- When `pick()` is called, it generates a random number in `[0, k-1]` and returns either the number itself (if not blacklisted) or its mapped replacement

## Intuition

The key insight is that we want to generate random numbers efficiently without repeatedly calling random until we get a non-blacklisted number, which could be very inefficient if many numbers are blacklisted.

Think of it this way: if we have `n` total numbers and `len(blacklist)` forbidden ones, we actually have `k = n - len(blacklist)` valid numbers to choose from. The challenge is how to uniformly select from these `k` valid numbers.

The clever idea is to create a "virtual" array of size `k` where all elements are valid. We can generate a random index in `[0, k-1]` and map it to an actual valid number. But we don't want to physically create this array as it could be huge.

Here's the mapping strategy:
1. Divide the original range `[0, n-1]` into two parts: `[0, k-1]` and `[k, n-1]`
2. Some blacklisted numbers fall in `[0, k-1]` and some in `[k, n-1]`
3. The blacklisted numbers in `[k, n-1]` are already "out of the way" - we'll never generate them since we only pick from `[0, k-1]`
4. For blacklisted numbers in `[0, k-1]`, we need to swap them with valid numbers from `[k, n-1]`

This is like having a deck of cards where we move all the "bad" cards to the bottom. When we draw from the top `k` cards, if we would draw a bad card that's still in the top portion, we have a pre-computed mapping that tells us which good card from the bottom to use instead.

The beauty is that this requires only one random call per `pick()` operation - we generate a random number in `[0, k-1]` and either use it directly or look up its replacement in our mapping dictionary.

**Learn more about [Math](/problems/math-basics), [Binary Search](/problems/binary_search_intro) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

Let's walk through the implementation step by step:

**Initialization (`__init__` method):**

1. **Calculate valid count**: First, we compute `k = n - len(blacklist)` (stored as `self.valid_range_size`), which represents the total number of valid (non-blacklisted) integers we can return.

2. **Create mapping dictionary**: Initialize an empty dictionary `self.remap_dict` that will store mappings for blacklisted numbers in the range `[0, k-1]`.

3. **Convert blacklist to set**: Create a set `blacklist_set` from the blacklist for O(1) lookup time when checking if a number is blacklisted.

4. **Build the remapping**:
   - Start with a pointer `replacement_index = k` (beginning of the second half of our range)
   - For each blacklisted number `b`:
     - If `b < k` (it's in the first half where we'll generate random numbers):
       - Find the next valid number starting from `i`: Keep incrementing `replacement_index` while it is also blacklisted
       - Map `b` to this valid number: `self.remap_dict[b] = replacement_index`
       - Increment `replacement_index` for the next potential mapping

**Pick method (`pick`):**

1. **Generate random index**: Call `randrange(self.valid_range_size)` to get a random number `x` in `[0, k-1]`.

2. **Return mapped value**: Use `self.remap_dict.get(x, x)` which:
   - Returns `self.remap_dict[x]` if `x` is a blacklisted number that we've mapped
   - Returns `x` itself if it's not in the mapping (meaning it's already a valid number)

**Example walkthrough:**
If `n = 7` and `blacklist = [2, 3, 5]`:
- `k = 7 - 3 = 4` (we have 4 valid numbers)
- Valid numbers are: `[0, 1, 4, 6]`
- Blacklisted numbers in `[0, 3]`: `2` and `3`
- Valid numbers in `[4, 6]`: `4` and `6`
- Mappings created: `{2: 4, 3: 6}`
- When `pick()` generates:
  - `0` → returns `0` (not mapped)
  - `1` → returns `1` (not mapped)
  - `2` → returns `4` (mapped)
  - `3` → returns `6` (mapped)

This ensures uniform distribution across all valid numbers with only one random call per pick.

## Example Walkthrough

Let's walk through a concrete example with `n = 10` and `blacklist = [3, 5, 8, 9]`.

**Initialization Phase:**

1. Calculate valid count: `k = 10 - 4 = 6` (we have 6 valid numbers to choose from)

2. Our valid numbers are: `[0, 1, 2, 4, 6, 7]`

3. Split our perspective:
   - Range we'll generate from: `[0, 5]` (indices 0 through k-1)
   - Range we'll remap to if needed: `[6, 9]` (indices k through n-1)

4. Identify problematic blacklisted numbers:
   - Blacklisted in `[0, 5]`: `3` and `5` (these need remapping)
   - Blacklisted in `[6, 9]`: `8` and `9` (already out of the way)

5. Build remapping:
   - Start with `i = 6`
   - Process `b = 3`:
     - Since `3 < 6`, we need to map it
     - Check `i = 6`: not blacklisted ✓
     - Map: `d[3] = 6`
     - Increment: `i = 7`
   - Process `b = 5`:
     - Since `5 < 6`, we need to map it
     - Check `i = 7`: not blacklisted ✓
     - Map: `d[5] = 7`
     - Increment: `i = 8`
   - Process `b = 8`: Since `8 ≥ 6`, skip (no mapping needed)
   - Process `b = 9`: Since `9 ≥ 6`, skip (no mapping needed)

Final mapping dictionary: `{3: 6, 5: 7}`

**Pick Phase:**

When `pick()` is called, it generates a random number in `[0, 5]`:

- If it generates `0` → returns `0` (valid, not mapped)
- If it generates `1` → returns `1` (valid, not mapped)
- If it generates `2` → returns `2` (valid, not mapped)
- If it generates `3` → returns `6` (using mapping `d[3] = 6`)
- If it generates `4` → returns `4` (valid, not mapped)
- If it generates `5` → returns `7` (using mapping `d[5] = 7`)

Each of the 6 valid numbers `[0, 1, 2, 4, 6, 7]` has exactly 1/6 probability of being returned, achieving perfect uniform distribution with just one random call per pick operation.

## Python Solution

```python
from typing import List
from random import randrange

class Solution:
    def __init__(self, n: int, blacklist: List[int]):
        # Calculate the size of the valid range [0, k) where we'll pick from
        # All blacklisted numbers will be remapped if they fall in this range
        self.valid_range_size = n - len(blacklist)

        # Dictionary to remap blacklisted numbers in [0, k) to valid numbers in [k, n)
        self.remap_dict = {}

        # Start searching for replacement values from position k
        replacement_index = self.valid_range_size

        # Convert blacklist to set for O(1) lookup
        blacklist_set = set(blacklist)

        # Process each blacklisted number
        for blacklisted_num in blacklist:
            # Only remap if the blacklisted number is in our picking range [0, k)
            if blacklisted_num < self.valid_range_size:
                # Find the next non-blacklisted number in range [k, n)
                while replacement_index in blacklist_set:
                    replacement_index += 1

                # Map the blacklisted number to a valid replacement
                self.remap_dict[blacklisted_num] = replacement_index
                replacement_index += 1

    def pick(self) -> int:
        # Pick a random number from [0, k)
        random_num = randrange(self.valid_range_size)

        # If it's a blacklisted number (exists in remap dict), return its replacement
        # Otherwise, return the number itself
        return self.remap_dict.get(random_num, random_num)


# Your Solution object will be instantiated and called as such:
# obj = Solution(n, blacklist)
# param_1 = obj.pick()
```

## Time and Space Complexity

**Time Complexity:**
- **Initialization (`__init__`):** `O(B)` where `B` is the length of the blacklist.
  - Creating a set from blacklist: `O(B)`
  - Iterating through each element in blacklist: `O(B)`
  - For each blacklisted number less than `k`, we iterate through the while loop. In the worst case, we check at most `B` blacklisted numbers in the range `[k, n)`, so the while loop runs `O(B)` times total across all iterations.
  - Overall initialization: `O(B)`

- **Pick method:** `O(1)`
  - Generating a random number: `O(1)`
  - Dictionary lookup: `O(1)` average case
  - Return operation: `O(1)`

**Space Complexity:** `O(B)`
- The dictionary `self.remap_dict` stores at most `B` mappings (one for each blacklisted number that is less than `k`)
- The set `blacklist_set` stores `B` elements
- Additional variables (`valid_range_size`, `replacement_index`, etc.) use `O(1)` space
- Overall space complexity: `O(B)`

The algorithm uses a remapping strategy where it maps blacklisted numbers in the range `[0, k)` to valid numbers in the range `[k, n)`, where `k = n - len(blacklist)`. This allows for constant-time random selection from the valid numbers.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Forgetting to Skip Already Blacklisted Replacement Values**

**The Pitfall:** A common mistake is not checking if the replacement index itself is blacklisted when building the remapping dictionary. If you simply map blacklisted numbers in `[0, k)` to consecutive numbers starting from `k`, you might accidentally map to another blacklisted number.

**Incorrect Implementation:**
```python
def __init__(self, n: int, blacklist: List[int]):
    self.valid_range_size = n - len(blacklist)
    self.remap_dict = {}
    replacement_index = self.valid_range_size

    for blacklisted_num in blacklist:
        if blacklisted_num < self.valid_range_size:
            # WRONG: Not checking if replacement_index is also blacklisted!
            self.remap_dict[blacklisted_num] = replacement_index
            replacement_index += 1
```

**Why it fails:** If `n = 7` and `blacklist = [2, 3, 5]`, this would map `2 → 4` and `3 → 5`. But `5` is also blacklisted! This would cause `pick()` to potentially return `5`, violating the constraint.

**Solution:** Always check if the replacement index is in the blacklist set before using it:
```python
while replacement_index in blacklist_set:
    replacement_index += 1
self.remap_dict[blacklisted_num] = replacement_index
```

### 2. **Not Converting Blacklist to Set for Efficient Lookups**

**The Pitfall:** Using the blacklist array directly for membership checking instead of converting it to a set first.

**Inefficient Implementation:**
```python
def __init__(self, n: int, blacklist: List[int]):
    self.valid_range_size = n - len(blacklist)
    self.remap_dict = {}
    replacement_index = self.valid_range_size

    for blacklisted_num in blacklist:
        if blacklisted_num < self.valid_range_size:
            # INEFFICIENT: O(m) lookup for each check where m = len(blacklist)
            while replacement_index in blacklist:
                replacement_index += 1
            self.remap_dict[blacklisted_num] = replacement_index
            replacement_index += 1
```

**Why it's problematic:** This creates O(m²) time complexity in the worst case, where m is the blacklist size. Each `in blacklist` check takes O(m) time.

**Solution:** Convert to a set first for O(1) lookups:
```python
blacklist_set = set(blacklist)
while replacement_index in blacklist_set:  # O(1) lookup
    replacement_index += 1
```

### 3. **Mapping All Blacklisted Numbers Instead of Only Those in [0, k)**

**The Pitfall:** Creating unnecessary mappings for blacklisted numbers that are already outside the picking range.

**Wasteful Implementation:**
```python
def __init__(self, n: int, blacklist: List[int]):
    self.valid_range_size = n - len(blacklist)
    self.remap_dict = {}

    for blacklisted_num in blacklist:
        # WRONG: Mapping even numbers >= k which will never be picked
        self.remap_dict[blacklisted_num] = some_valid_number
```

**Why it's problematic:** Since `pick()` only generates numbers in `[0, k)`, blacklisted numbers >= k will never be generated anyway. Creating mappings for them wastes memory and initialization time.

**Solution:** Only create mappings for blacklisted numbers less than k:
```python
if blacklisted_num < self.valid_range_size:
    # Only then create the mapping
```

