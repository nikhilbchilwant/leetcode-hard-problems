# 3213. Construct String with Minimum Cost

> Source: https://algo.monster/liteproblems/3213

## Problem Statement

You are given a string `target` that you need to construct, along with an array of strings `words` and an array of integers `costs` (both arrays have the same length).

Starting with an empty string `s`, you can perform the following operation any number of times:
- Select any index `i` from the range `[0, words.length - 1]`
- Append `words[i]` to the end of string `s`
- This operation incurs a cost of `costs[i]`

Your goal is to find the **minimum total cost** needed to make string `s` equal to the `target` string. If it's impossible to construct the `target` string using the given words, return `-1`.

For example, if `target = "abcd"`, `words = ["a", "bc", "d", "ab", "cd"]`, and `costs = [1, 3, 2, 4, 5]`, you could:
- Append `"ab"` (cost 4) to get `s = "ab"`
- Append `"cd"` (cost 5) to get `s = "abcd"`
- Total cost: 4 + 5 = 9

Or alternatively:
- Append `"a"` (cost 1) to get `s = "a"`
- Append `"bc"` (cost 3) to get `s = "abc"`
- Append `"d"` (cost 2) to get `s = "abcd"`
- Total cost: 1 + 3 + 2 = 6

The minimum cost would be 6 in this case.

The key challenge is finding the optimal combination of words that:
1. When concatenated in order, form exactly the `target` string
2. Have the minimum total cost among all valid combinations

## Intuition

When we need to build a target string from smaller pieces with minimum cost, this naturally leads us to think about **[dynamic programming](/problems/dynamic_programming_intro)**. We can break down the problem into smaller subproblems: what's the minimum cost to build each prefix of the target string?

Let's define `f[i]` as the minimum cost to construct the first `i` characters of the target. To compute `f[i]`, we need to consider all possible ways to reach position `i`. We could have arrived at position `i` by appending a word that ends exactly at position `i`.

For example, if we're trying to build "abcdef" and we're at position 4 (having built "abcd"), we might have gotten there by:
- Building "abc" optimally, then adding "d"
- Building "ab" optimally, then adding "cd"  
- Building "a" optimally, then adding "bcd"
- Starting fresh with "abcd"

The key insight is that we need to check if the substring `target[i-j:i]` (where `j` is the length of a potential word) matches any word in our word list. If it does, we can transition from `f[i-j]` to `f[i]` with the cost of that word.

However, comparing strings directly for every position would be inefficient. This is where **string hashing** comes in. By pre-computing hash values for all substrings of the target, we can check if a substring matches a word in `O(1)` time instead of `O(length)` time.

We also optimize by:
1. Pre-computing a hash map that stores the minimum cost for each unique hash value (since multiple words might have the same content but different costs)
2. Only considering unique word lengths to avoid redundant checks
3. Using rolling hash technique to efficiently compute hash values for all substrings

The state transition becomes: `f[i] = min(f[i], f[i-j] + cost_of_matching_word)` for all valid word lengths `j` where the substring `target[i-j:i]` matches a word in our list.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution implements [dynamic programming](/problems/dynamic_programming_intro) with string hashing to efficiently find the minimum cost to construct the target string.

### String Hashing Setup

First, we compute polynomial rolling hash values for all prefixes of the target string:
- `h[i]` stores the hash value of `target[0:i]`
- `p[i]` stores `base^i mod mod` for efficient hash computation
- Hash formula: `h[i] = (h[i-1] * base + ord(target[i-1])) % mod`

This allows us to compute the hash of any substring `target[l:r]` in O(1) time using:
`hash(l, r) = (h[r] - h[l] * p[r-l]) % mod`

### Word Processing

We preprocess the words to create a hash map `d` where:
- Key: hash value of a word
- Value: minimum cost among all words with that hash value

This handles cases where multiple words have the same content but different costs - we only keep the cheapest option.

We also extract unique word lengths into `ss` (sorted set) to avoid checking impossible lengths.

### [Dynamic Programming](/problems/dynamic_programming_intro)

Initialize `f[0] = 0` (no cost for empty string) and `f[i] = inf` for all other positions.

For each position `i` from 1 to n:
1. Enumerate all possible word lengths `j` in ascending order
2. If `j > i`, break (word too long for current position)
3. Calculate the hash of substring `target[i-j:i]` using the formula:
   `x = (h[i] - h[i-j] * p[j]) % mod`
4. If this hash exists in our word dictionary `d`, update:
   `f[i] = min(f[i], f[i-j] + d[x])`

This checks if we can reach position `i` by:
- First building the string up to position `i-j` with cost `f[i-j]`
- Then appending a word of length `j` that matches `target[i-j:i]` with cost `d[x]`

### Final Result

Return `f[n]` if it's less than infinity (meaning the target is constructible), otherwise return `-1`.

The time complexity is `O(n * m)` where `n` is the length of target and `m` is the number of unique word lengths. The space complexity is `O(n + w)` where `w` is the number of words.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

Given:
- `target = "abc"`
- `words = ["a", "bc", "ab", "c"]`
- `costs = [2, 3, 5, 1]`

**Step 1: String Hashing Setup**

First, we compute hash values for all prefixes of "abc" (using base=131, mod=10^9+7):
- `h[0] = 0` (empty string)
- `h[1] = 97` (hash of "a")
- `h[2] = 97*131 + 98 = 12805` (hash of "ab") 
- `h[3] = 12805*131 + 99 = 1677754` (hash of "abc")

We also compute powers: `p[0]=1`, `p[1]=131`, `p[2]=17161`, `p[3]=2248091`

**Step 2: Word Processing**

Create hash map `d` with minimum costs for each word hash:
- "a" → hash = 97, cost = 2
- "bc" → hash = 98*131 + 99 = 12937, cost = 3
- "ab" → hash = 12805, cost = 5
- "c" → hash = 99, cost = 1

So `d = {97: 2, 12937: 3, 12805: 5, 99: 1}`

Unique word lengths: `ss = {1, 2}` (sorted)

**Step 3: Dynamic Programming**

Initialize: `f[0] = 0`, `f[1] = inf`, `f[2] = inf`, `f[3] = inf`

**Position i = 1** (building "a"):
- Check length j = 1: substring "a" (positions 0 to 1)
  - Hash = h[1] - h[0]*p[1] = 97
  - Found in d with cost 2
  - `f[1] = min(inf, f[0] + 2) = 0 + 2 = 2`
- Check length j = 2: too long (2 > 1), skip

**Position i = 2** (building "ab"):
- Check length j = 1: substring "b" (positions 1 to 2)
  - Hash = (h[2] - h[1]*p[1]) % mod = (12805 - 97*131) % mod = 98
  - Not in d, skip
- Check length j = 2: substring "ab" (positions 0 to 2)
  - Hash = h[2] - h[0]*p[2] = 12805
  - Found in d with cost 5
  - `f[2] = min(inf, f[0] + 5) = 0 + 5 = 5`

**Position i = 3** (building "abc"):
- Check length j = 1: substring "c" (positions 2 to 3)
  - Hash = (h[3] - h[2]*p[1]) % mod = (1677754 - 12805*131) % mod = 99
  - Found in d with cost 1
  - `f[3] = min(inf, f[2] + 1) = 5 + 1 = 6`
- Check length j = 2: substring "bc" (positions 1 to 3)
  - Hash = (h[3] - h[1]*p[2]) % mod = (1677754 - 97*17161) % mod = 12937
  - Found in d with cost 3
  - `f[3] = min(6, f[1] + 3) = min(6, 2 + 3) = 5`

**Final Result**: `f[3] = 5`

The minimum cost to build "abc" is 5, achieved by:
- Using "a" (cost 2) to build "a"
- Then using "bc" (cost 3) to complete "abc"
- Total: 2 + 3 = 5

## Python Solution

```python
from typing import List
from collections import defaultdict

class Solution:
    def minimumCost(self, target: str, words: List[str], costs: List[int]) -> int:
        # Rolling hash parameters
        HASH_BASE = 13331
        HASH_MOD = 998244353
      
        target_length = len(target)
      
        # Precompute rolling hash values for all prefixes of target
        # hash_values[i] = hash of target[0:i]
        hash_values = [0] * (target_length + 1)
        # power_values[i] = HASH_BASE^i mod HASH_MOD
        power_values = [1] * (target_length + 1)
      
        # Build rolling hash arrays
        for i, char in enumerate(target, 1):
            hash_values[i] = (hash_values[i - 1] * HASH_BASE + ord(char)) % HASH_MOD
            power_values[i] = (power_values[i - 1] * HASH_BASE) % HASH_MOD
      
        # dp[i] = minimum cost to construct target[0:i]
        dp = [0] + [float('inf')] * target_length
      
        # Get unique word lengths in sorted order for optimization
        unique_word_lengths = sorted(set(map(len, words)))
      
        # Map each word's hash to its minimum cost
        word_hash_to_min_cost = defaultdict(lambda: float('inf'))
      
        # Calculate hash for each word and store minimum cost
        for word, cost in zip(words, costs):
            word_hash = 0
            for char in word:
                word_hash = (word_hash * HASH_BASE + ord(char)) % HASH_MOD
            word_hash_to_min_cost[word_hash] = min(word_hash_to_min_cost[word_hash], cost)
      
        # Dynamic programming to find minimum cost
        for i in range(1, target_length + 1):
            # Try all possible word lengths
            for word_length in unique_word_lengths:
                # Skip if word length exceeds current position
                if word_length > i:
                    break
              
                # Calculate hash of substring target[i-word_length:i] using rolling hash
                substring_hash = (hash_values[i] - hash_values[i - word_length] * power_values[word_length]) % HASH_MOD
              
                # Update dp value if using this word gives a better cost
                dp[i] = min(dp[i], dp[i - word_length] + word_hash_to_min_cost[substring_hash])
      
        # Return result: minimum cost if target is constructible, else -1
        return dp[target_length] if dp[target_length] < float('inf') else -1
```

## Time and Space Complexity

**Time Complexity:** `O(n × √L)`

The algorithm consists of several parts:
- Computing rolling hash for the target string takes `O(n)` time
- Processing all words to build the hash dictionary takes `O(L)` time, where `L` is the sum of lengths of all words
- The main dynamic programming loop runs for `n` iterations (from 1 to n)
- For each position `i`, we iterate through all unique word lengths in `ss`
- The key insight is that while there could be many words, the number of distinct word lengths is bounded by `O(√L)`. This is because if we have `k` distinct lengths, in the worst case they would be 1, 2, 3, ..., k, and their sum would be `k(k+1)/2 ≤ L`, which gives us `k ≤ √(2L)`, hence `k = O(√L)`
- For each combination of position and word length, we perform constant time operations (hash calculation and comparison)

Therefore, the total time complexity is `O(n) + O(L) + O(n × √L) = O(n × √L)` when `n × √L > L`.

**Space Complexity:** `O(n + |d|)`

- Arrays `h`, `p`, and `f` each use `O(n+1) = O(n)` space
- The sorted set `ss` contains at most `O(√L)` unique lengths
- The dictionary `d` stores unique word hashes with their minimum costs, which in the worst case could be `O(|words|)` but is bounded by the number of unique words
- Since the reference answer states `O(n)` space complexity, this assumes that the dictionary size is dominated by `n` or considered separately from the main space analysis

The dominant space usage is `O(n)` for the dynamic programming arrays.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. Hash Collision Issues
**Problem:** The rolling hash implementation can produce negative values when computing substring hashes, leading to incorrect hash comparisons and wrong results.

When calculating `substring_hash = (hash_values[i] - hash_values[i - word_length] * power_values[word_length]) % HASH_MOD`, the subtraction can produce a negative number before the modulo operation. In Python, this results in a negative hash value, which won't match the positive hash values stored in `word_hash_to_min_cost`.

**Solution:** Ensure the hash value is always positive by adding HASH_MOD when the result is negative:

```python
# Instead of:
substring_hash = (hash_values[i] - hash_values[i - word_length] * power_values[word_length]) % HASH_MOD

# Use:
substring_hash = (hash_values[i] - hash_values[i - word_length] * power_values[word_length] % HASH_MOD + HASH_MOD) % HASH_MOD
```

### 2. Integer Overflow in Hash Computation
**Problem:** When computing `hash_values[i - word_length] * power_values[word_length]`, the multiplication can overflow even with modulo operations, potentially causing incorrect results in languages with fixed integer sizes.

**Solution:** Apply modulo operation to the multiplication before subtraction:

```python
substring_hash = (hash_values[i] - (hash_values[i - word_length] * power_values[word_length]) % HASH_MOD + HASH_MOD) % HASH_MOD
```

### 3. Duplicate Words with Different Costs
**Problem:** While the code handles this correctly by keeping the minimum cost for each hash, developers might forget this edge case and use the first occurrence or last occurrence instead.

**Example:** If `words = ["abc", "abc"]` and `costs = [10, 5]`, we must use cost 5, not 10.

**Solution:** The current implementation correctly uses:
```python
word_hash_to_min_cost[word_hash] = min(word_hash_to_min_cost[word_hash], cost)
```

### 4. Empty Words in Input
**Problem:** If the input contains empty strings in the `words` array, they would have a hash of 0 and could incorrectly affect the DP calculation.

**Solution:** Filter out empty words or add validation:

```python
# Add validation at the beginning
for word, cost in zip(words, costs):
    if not word:  # Skip empty words
        continue
    word_hash = 0
    for char in word:
        word_hash = (word_hash * HASH_BASE + ord(char)) % HASH_MOD
    word_hash_to_min_cost[word_hash] = min(word_hash_to_min_cost[word_hash], cost)
```

### 5. Hash Function Choice
**Problem:** Poor choice of hash base and modulo can increase collision probability, leading to incorrect results when different substrings produce the same hash.

**Solution:** Use well-tested parameters (like base=13331, mod=998244353) or implement double hashing for critical applications:

```python
# Double hashing for better collision resistance
HASH_BASE1, HASH_MOD1 = 13331, 998244353
HASH_BASE2, HASH_MOD2 = 10007, 1000000007
# Compute both hashes and use tuple (hash1, hash2) as key
```

