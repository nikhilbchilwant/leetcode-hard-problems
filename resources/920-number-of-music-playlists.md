# 920. Number of Music Playlists

> Source: https://algo.monster/liteproblems/920

## Problem Statement

You have a music player with `n` different songs. You want to create a playlist to listen to exactly `goal` songs during your trip (songs can repeat in the playlist). To keep things interesting and avoid boredom, your playlist must follow these rules:

1. **Every song must be played at least once** - All `n` songs must appear in your playlist at least one time.

2. **Replay restriction** - If you want to play a song again, you must wait until at least `k` other different songs have been played since the last time you played it.

For example, if `k = 2` and you play song A, you need to play at least 2 other different songs before you can play song A again.

Given three integers:
- `n`: the number of different songs available
- `goal`: the total length of the playlist (total number of songs to play)
- `k`: the minimum number of different songs that must be played before repeating a song

Return the number of different valid playlists you can create. Since this number can be very large, return the answer modulo `10^9 + 7`.

The problem uses [dynamic programming](/problems/dynamic_programming_intro) where `f[i][j]` represents the number of ways to create a playlist of length `i` using exactly `j` different songs. The transition considers two cases:
- Adding a new song that hasn't been played yet: there are `(n - j + 1)` choices
- Repeating a song that was already played: there are `(j - k)` choices (only if `j > k`)

The final answer is `f[goal][n]`, representing playlists of length `goal` that use all `n` songs.

## Intuition

The key insight is to think about building the playlist step by step, keeping track of how many songs we've used so far. When constructing a playlist, at each position we need to make a choice: either play a new song we haven't played before, or replay a song we've already used.

Let's think about what state we need to track. At any point in building our playlist, we care about:
- How many songs we've added to the playlist so far (let's call this `i`)
- How many different songs we've used (let's call this `j`)

This naturally leads to a [dynamic programming](/problems/dynamic_programming_intro) approach where `f[i][j]` represents the number of ways to create a playlist of length `i` using exactly `j` different songs.

Now, when we're at position `i` in our playlist and have used `j` different songs, we have two choices:

**Choice 1: Play a new song**
- We can only do this if `j < n` (we haven't used all songs yet)
- We had `j - 1` different songs before, and now we're adding the `j`-th one
- How many new songs can we choose from? We have `n` total songs, and we've already used `j - 1`, so we have `n - (j - 1) = n - j + 1` options
- This contributes `f[i - 1][j - 1] * (n - j + 1)` to our count

**Choice 2: Replay an existing song**
- We can only do this if we've already used some songs (`j > 0`)
- Due to the constraint, we can only replay songs that were played more than `k` songs ago
- If we have `j` different songs in our rotation and need to skip the last `k` songs, we have `j - k` songs available to replay
- This only works if `j > k` (otherwise we don't have enough songs to satisfy the gap requirement)
- This contributes `f[i - 1][j] * (j - k)` to our count

The base case is `f[0][0] = 1` - there's exactly one way to create an empty playlist with zero songs.

Since we need all `n` songs to appear at least once in our `goal`-length playlist, our answer is `f[goal][n]`.

**Learn more about [Math](/problems/math-basics), [Dynamic Programming](/problems/dynamic_programming_intro) and [Combinatorics](/problems/backtracking) patterns.**

## Solution Approach

We implement the [dynamic programming](/problems/dynamic_programming_intro) solution using a 2D array `f` where `f[i][j]` represents the number of ways to create a playlist of length `i` using exactly `j` different songs.

**Initialization:**
- Create a 2D array `f` of size `(goal + 1) × (n + 1)` initialized with zeros
- Set the base case `f[0][0] = 1` (one way to create an empty playlist)

**State Transition:**
For each position `i` from `1` to `goal` and for each number of different songs `j` from `1` to `n`, we calculate `f[i][j]` based on two scenarios:

1. **Adding a new song:** 
   - Previous state: `f[i - 1][j - 1]` (playlist of length `i - 1` with `j - 1` different songs)
   - Number of choices: `(n - j + 1)` new songs available
   - Contribution: `f[i][j] += f[i - 1][j - 1] * (n - j + 1)`

2. **Replaying an existing song:** 
   - Previous state: `f[i - 1][j]` (playlist of length `i - 1` with `j` different songs)
   - Condition: Only possible if `j > k` (we have enough songs to satisfy the gap requirement)
   - Number of choices: `(j - k)` songs that can be replayed
   - Contribution: `f[i][j] += f[i - 1][j] * (j - k)`

**Implementation Details:**
```python
for i in range(1, goal + 1):
    for j in range(1, n + 1):
        # Add a new song
        f[i][j] = f[i - 1][j - 1] * (n - j + 1)
      
        # Replay an existing song (if possible)
        if j > k:
            f[i][j] += f[i - 1][j] * (j - k)
      
        # Apply modulo to prevent overflow
        f[i][j] %= mod
```

**Time Complexity:** `O(goal × n)` as we iterate through all states in our DP table.

**Space Complexity:** `O(goal × n)` for storing the DP table.

The final answer is `f[goal][n]`, which represents the number of playlists of length `goal` that include all `n` songs at least once.

## Example Walkthrough

Let's walk through a small example with `n = 3` songs, `goal = 3` songs in playlist, and `k = 1` (must play 1 different song before repeating).

We'll build our DP table `f[i][j]` where `i` is playlist length and `j` is number of different songs used.

**Initial State:**
```
f[0][0] = 1 (one way to make empty playlist)
All other entries start at 0
```

**Building the DP Table:**

For `i = 1` (playlist of length 1):
- `f[1][1]`: Use 1 different song
  - Add new song: `f[0][0] × (3 - 1 + 1) = 1 × 3 = 3`
  - Can't replay (j ≤ k)
  - Result: `f[1][1] = 3`

For `i = 2` (playlist of length 2):
- `f[2][1]`: Use 1 different song (must repeat the same song)
  - Add new song: `f[1][0] × 3 = 0` (f[1][0] doesn't exist/is 0)
  - Can't replay (j = 1 ≤ k = 1)
  - Result: `f[2][1] = 0`

- `f[2][2]`: Use 2 different songs
  - Add new song: `f[1][1] × (3 - 2 + 1) = 3 × 2 = 6`
  - Replay existing: `f[1][2] × (2 - 1) = 0` (f[1][2] is 0)
  - Result: `f[2][2] = 6`

For `i = 3` (playlist of length 3):
- `f[3][1]`: Use 1 different song
  - Add new song: `f[2][0] × 3 = 0`
  - Can't replay (j ≤ k)
  - Result: `f[3][1] = 0`

- `f[3][2]`: Use 2 different songs
  - Add new song: `f[2][1] × 2 = 0 × 2 = 0`
  - Replay existing: `f[2][2] × (2 - 1) = 6 × 1 = 6`
  - Result: `f[3][2] = 6`

- `f[3][3]`: Use 3 different songs (all songs used)
  - Add new song: `f[2][2] × (3 - 3 + 1) = 6 × 1 = 6`
  - Replay existing: `f[2][3] × (3 - 1) = 0` (f[2][3] is 0)
  - Result: `f[3][3] = 6`

**Final DP Table:**
```
     j=0  j=1  j=2  j=3
i=0   1    0    0    0
i=1   0    3    0    0
i=2   0    0    6    0
i=3   0    0    6    6
```

**Answer:** `f[3][3] = 6`

The 6 valid playlists are (using songs A, B, C):
1. [A, B, C] - each song played once
2. [A, C, B] - each song played once
3. [B, A, C] - each song played once
4. [B, C, A] - each song played once
5. [C, A, B] - each song played once
6. [C, B, A] - each song played once

Note that we can't have playlists like [A, A, B] because that would only use 2 different songs, not all 3 as required.

## Python Solution

```python
class Solution:
    def numMusicPlaylists(self, n: int, goal: int, k: int) -> int:
        """
        Calculate the number of possible playlists of length 'goal' containing all 'n' songs,
        where each song must be separated by at least 'k' other songs before being repeated.
      
        Args:
            n: Total number of unique songs available
            goal: Target length of the playlist
            k: Minimum number of other songs between repetitions of the same song
      
        Returns:
            Number of valid playlists modulo 10^9 + 7
        """
        MOD = 10**9 + 7
      
        # dp[i][j] represents the number of playlists of length i containing exactly j unique songs
        dp = [[0] * (n + 1) for _ in range(goal + 1)]
      
        # Base case: empty playlist with 0 songs
        dp[0][0] = 1
      
        # Fill the DP table
        for playlist_length in range(1, goal + 1):
            for unique_songs in range(1, n + 1):
                # Case 1: Add a new song that hasn't been played yet
                # We have (n - unique_songs + 1) choices for the new song
                dp[playlist_length][unique_songs] = dp[playlist_length - 1][unique_songs - 1] * (n - unique_songs + 1)
              
                # Case 2: Replay an existing song (if allowed by the k constraint)
                # We can only replay if we have more than k unique songs already
                # and we can choose from (unique_songs - k) songs that are eligible for replay
                if unique_songs > k:
                    dp[playlist_length][unique_songs] += dp[playlist_length - 1][unique_songs] * (unique_songs - k)
              
                # Apply modulo to prevent overflow
                dp[playlist_length][unique_songs] %= MOD
      
        # Return the number of playlists of length 'goal' containing all 'n' unique songs
        return dp[goal][n]
```

## Time and Space Complexity

The time complexity is `O(goal × n)`. The code uses two nested loops - the outer loop iterates from 1 to `goal` (which runs `goal` times), and the inner loop iterates from 1 to `n` (which runs `n` times). Inside the nested loops, all operations including arithmetic calculations, array access, and modulo operations take constant time `O(1)`. Therefore, the overall time complexity is `O(goal × n)`.

The space complexity is `O(goal × n)`. The code creates a 2D array `f` with dimensions `(goal + 1) × (n + 1)`, which requires `O((goal + 1) × (n + 1))` space. This simplifies to `O(goal × n)` since we drop the constant additions when expressing big-O notation. No other data structures are created that would affect the space complexity, so the total space complexity remains `O(goal × n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Off-by-One Error in Counting Available Songs**

**The Pitfall:**
When adding a new song that hasn't been played yet, developers often make mistakes in calculating how many songs are available to choose from. A common error is using `(n - j)` instead of `(n - j + 1)`.

**Why It Happens:**
- If we have `n` total songs and have already used `j - 1` different songs, we're now choosing the `j`-th unique song
- The number of remaining songs is `n - (j - 1) = n - j + 1`
- It's easy to incorrectly think "we have used `j` songs, so `n - j` remain"

**Incorrect Code:**
```python
# Wrong: This assumes we've already used j songs
dp[i][j] = dp[i - 1][j - 1] * (n - j)  # ❌
```

**Correct Code:**
```python
# Right: We're choosing the j-th song from (n - j + 1) available options
dp[i][j] = dp[i - 1][j - 1] * (n - j + 1)  # ✅
```

### 2. **Integer Overflow from Not Applying Modulo**

**The Pitfall:**
Leaving out the modulo operation entirely, or applying it only to the final answer, lets the table values grow without bound.

**Why It Happens:**
- The number of playlists grows exponentially with `goal`
- Python handles arbitrary precision integers, but the computation becomes much slower as the numbers get longer
- In Java or C++, the `long` values overflow and the answer is wrong

**Incorrect Code:**
```python
# Wrong: no modulo inside the loops
dp[i][j] = dp[i - 1][j - 1] * (n - j + 1)
if j > k:
    dp[i][j] += dp[i - 1][j] * (j - k)
# ...
return dp[goal][n] % MOD  # Too late in Java/C++ ❌
```

**Correct Code:**
```python
# Right: reduce every cell before it is used again
dp[i][j] = dp[i - 1][j - 1] * (n - j + 1)
if j > k:
    dp[i][j] += dp[i - 1][j] * (j - k)
dp[i][j] %= MOD  # ✅
```

Reducing once per cell is enough: each stored value is below `10^9 + 7` and each multiplier is at most `n <= 100`, so the sum of the two products fits easily in a 64-bit `long`.

### 3. **Incorrect Base Case Initialization**

**The Pitfall:**
Setting `dp[0][0] = 0` or forgetting to initialize the base case entirely, which propagates zeros throughout the entire DP table.

**Why It Happens:**
- It seems counterintuitive that there's "one way" to create an empty playlist
- Developers might think an empty playlist should have zero ways

**Incorrect Code:**
```python
# Wrong: No valid playlists will be counted
dp[0][0] = 0  # ❌
# Or forgetting initialization entirely
```

**Correct Code:**
```python
# Right: There's exactly one way to have an empty playlist
dp[0][0] = 1  # ✅
```

### 4. **Misunderstanding the Replay Constraint**

**The Pitfall:**
Dropping the `j > k` guard on the replay case, or miscounting the number of replayable songs.

**Why It Happens:**
- The constraint states we need `k` OTHER songs between replays
- The last `k` songs played are all different and are blocked, so only `j - k` of the `j` used songs can be replayed
- When `j < k`, the factor `j - k` is negative, so without the guard the code adds negative counts

**Incorrect Code:**
```python
# Wrong: no guard, so j < k contributes a negative count
dp[i][j] += dp[i - 1][j] * (j - k)  # ❌
```

For `n = 3, goal = 4, k = 2`, this version returns `0` instead of the correct `6` (only `[A, B, C, A]` and its relabelings are valid). Note that writing `j >= k` instead of `j > k` happens to give the same results, because when `j == k` the factor `j - k` is 0.

**Correct Code:**
```python
# Right: replay only when more than k distinct songs have been used
if j > k:  # ✅
    dp[i][j] += dp[i - 1][j] * (j - k)
    # When j = k+1, we have 1 song that can be replayed
```

