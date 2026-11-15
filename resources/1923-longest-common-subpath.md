# 1923. Longest Common Subpath

> Source: https://algo.monster/liteproblems/1923

## Problem Statement

You have a country with `n` cities numbered from `0` to `n - 1`. Every pair of cities in this country is connected by a road.

There are `m` friends (numbered from `0` to `m - 1`) traveling through the country. Each friend follows a specific path through the cities, represented as an array of city numbers in the order they visit them. A friend can visit the same city multiple times in their journey, but the same city won't appear consecutively in their path.

Given:
- An integer `n` (number of cities)
- A 2D array `paths` where `paths[i]` represents the travel path of the `i-th` friend

Your task is to find the length of the longest common subpath that appears in every friend's path. If no common subpath exists among all friends, return `0`.

A **subpath** is defined as a contiguous sequence of cities within a path. For example, if a friend's path is `[1, 2, 3, 4]`, then `[2, 3]` is a subpath, but `[1, 3]` is not (since it's not contiguous).

## Intuition

The key insight is recognizing a monotonic property: if a common subpath of length `k` exists among all friends' paths, then any common subpath of length less than `k` also exists. This creates a feasibility sequence:

```
Length k:    0    1    2    3    4    5    6
Feasible:    T    T    T    T    F    F    F
                            ↑
                       first_true_index (maximum feasible)
```

We want to find the **maximum** length where a common subpath exists. Using our standard binary search template, we track `first_true_index` as we search.

For each candidate length `k`, we define `feasible(k)` as: does there exist a subpath of length `k` that appears in all friends' paths?

To check efficiently, we use **rolling hash**:
1. Compute hash values for all subpaths of length `k` in each path
2. Track unique hashes per friend (avoid double-counting)
3. Count occurrences across all friends
4. If any hash appears in all `m` paths, return true

The rolling hash formula: for subpath `[a, b, c]`, hash = `a * base^2 + b * base^1 + c * base^0`. Adjacent hashes can be computed in `O(1)` using prefix hashes.

**Learn more about [Binary Search](/problems/binary_search_intro) patterns.**

## Solution Approach

The solution uses [binary search](/problems/binary_search_intro) with the standard template, combined with rolling hash to check feasibility.

**Template Structure:**

```
left, right = 0, min(len(path) for path in paths)
first_true_index = 0  # Default: length 0 always works (empty subpath)

while left <= right:
    mid = (left + right) // 2
    if feasible(mid):
        first_true_index = mid  # Record this feasible length
        left = mid + 1          # Try larger lengths
    else:
        right = mid - 1         # Try smaller lengths

return first_true_index
```

**Preprocessing - Hash Arrays:**

Compute prefix hashes for rolling hash calculation:
```python
for path in paths:
    prefix_hash = [0] * (len(path) + 1)
    for i, city in enumerate(path, 1):
        prefix_hash[i] = (prefix_hash[i-1] * BASE + city) % MOD
```

This allows O(1) subpath hash: `hash(i, j) = (h[j] - h[i-1] * base^(j-i+1)) % mod`

**Feasibility Check:**

For length `k`, check if any subpath appears in all paths:
1. For each path, compute hashes of all subpaths of length `k`
2. Track unique hashes per path (avoid double-counting)
3. Count occurrences across all paths
4. If any hash appears in all `m` paths, return true

**Time Complexity:** `O(S × log(min_len))` where `S` = total path length.

**Space Complexity:** `O(S)` for prefix hash arrays.

## Example Walkthrough

Let's walk through with `paths = [[0,1,2,3,4], [2,3,4], [4,0,1,2,3]]`.

**Step 1: Initialize**
- `left = 0`, `right = 3` (min path length)
- `first_true_index = 0`

**Step 2: Binary search iterations**

**Iteration 1:** `left = 0, right = 3`
- `mid = (0 + 3) // 2 = 1`
- Check for common subpath of length 1:
  - All paths contain cities, so any shared city works
  - Hash counts: city 2 appears in all 3 paths ✓
- Update: `first_true_index = 1`, `left = 1 + 1 = 2`

**Iteration 2:** `left = 2, right = 3`
- `mid = (2 + 3) // 2 = 2`
- Check for common subpath of length 2:
  - Path 0: hashes for [0,1], [1,2], [2,3], [3,4]
  - Path 1: hashes for [2,3], [3,4]
  - Path 2: hashes for [4,0], [0,1], [1,2], [2,3]
  - Hash for [2,3] appears in all 3 paths ✓
- Update: `first_true_index = 2`, `left = 2 + 1 = 3`

**Iteration 3:** `left = 3, right = 3`
- `mid = (3 + 3) // 2 = 3`
- Check for common subpath of length 3:
  - Path 0: [0,1,2], [1,2,3], [2,3,4]
  - Path 1: [2,3,4] only
  - Path 2: [4,0,1], [0,1,2], [1,2,3]
  - No hash appears in all 3 paths ✗
- Update: `right = 3 - 1 = 2`

**Step 3: Loop terminates**
- `left = 3 > right = 2`
- Return `first_true_index = 2`

The longest common subpath has length **2** (subpath `[2,3]`).

## Python Solution

```python
class Solution:
    def longestCommonSubpath(self, n: int, paths: List[List[int]]) -> int:
        num_paths = len(paths)
        max_len = max(len(path) for path in paths)

        # Hash parameters
        BASE = 133331
        MOD = 2**64 + 1

        # Precompute powers of base
        base_powers = [1] * (max_len + 1)
        for i in range(1, len(base_powers)):
            base_powers[i] = base_powers[i - 1] * BASE % MOD

        # Compute prefix hashes for all paths
        path_hashes = []
        for path in paths:
            h = [0] * (len(path) + 1)
            for i, city in enumerate(path, 1):
                h[i] = (h[i - 1] * BASE + city) % MOD
            path_hashes.append(h)

        def feasible(length: int) -> bool:
            """Check if common subpath of given length exists."""
            if length == 0:
                return True
            hash_count = Counter()
            for h in path_hashes:
                seen = set()
                for i in range(1, len(h) - length + 1):
                    j = i + length - 1
                    subhash = (h[j] - h[i - 1] * base_powers[length]) % MOD
                    if subhash not in seen:
                        seen.add(subhash)
                        hash_count[subhash] += 1
            return max(hash_count.values()) == num_paths

        # Binary search with standard template
        left, right = 0, min(len(path) for path in paths)
        first_true_index = 0

        while left <= right:
            mid = (left + right) // 2

            if feasible(mid):
                first_true_index = mid
                left = mid + 1
            else:
                right = mid - 1

        return first_true_index
```

## Time and Space Complexity

**Time Complexity:** `O(S * log(min_len))` where `S` is the total sum of all path lengths and `min_len` is the length of the shortest path.

The analysis breaks down as follows:
- Binary search runs for `O(log(min_len))` iterations
- For each binary search iteration, we call the `check(k)` function
- Inside `check(k)`:
  - We iterate through all `m` paths
  - For each path of length `len(path)`, we compute rolling hashes for all subpaths of length `k`, which takes `O(len(path) - k + 1)` iterations
  - Each hash computation is `O(1)` due to the precomputed prefix hashes
  - Total work in `check(k)` is `O(sum(len(path) - k + 1))` for all paths, which is `O(S)`
- Preprocessing step computes prefix hashes for all paths in `O(S)` time
- Overall: `O(S) + O(S * log(min_len)) = O(S * log(min_len))`

**Space Complexity:** `O(S + mx)` where `S` is the total sum of all path lengths and `mx` is the length of the longest path.

The space breakdown:
- Array `p` stores powers of base up to `mx + 1`: `O(mx)`
- Array `hh` stores prefix hashes for all paths: `O(S)` total space
- Inside `check(k)`:
  - `cnt` Counter can store at most `O(S)` unique hash values
  - `vis` set for each path stores at most `O(len(path))` hashes
- Overall: `O(S + mx)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. Using the Wrong Binary Search Template
**The Pitfall**: Using `while left < right` with `left = mid` instead of the standard `while left <= right` template.

**Solution**: Use the standard template:
```python
left, right = 0, min(len(path) for path in paths)
first_true_index = 0

while left <= right:
    mid = (left + right) // 2
    if feasible(mid):
        first_true_index = mid
        left = mid + 1
    else:
        right = mid - 1

return first_true_index
```

### 2. Hash Collision Issues
**The Pitfall**: Using a small modulo or base value leads to collisions where different subpaths produce the same hash.

**Solution**: Use a large prime base (e.g., `133331`) and large modulo (e.g., `2^64 + 1`). For extra safety, consider double hashing.

### 3. Duplicate Counting Within a Path
**The Pitfall**: Counting the same subpath multiple times within one path.

**Solution**: Use a `seen` set per path to ensure each unique subpath hash is counted only once:
```python
seen = set()
for start in range(1, len(h) - length + 1):
    subhash = compute_hash(...)
    if subhash not in seen:
        seen.add(subhash)
        hash_count[subhash] += 1
```

### 4. Off-by-One Errors in Index Calculations
**The Pitfall**: Confusion between 0-based and 1-based indexing for prefix hashes.

**Solution**: Be consistent. Use 1-based indexing where `h[i]` = hash of `path[0:i]`.

