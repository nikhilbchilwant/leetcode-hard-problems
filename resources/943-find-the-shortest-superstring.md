# 943. Find the Shortest Superstring

> Source: https://algo.monster/liteproblems/943

## Problem Statement

The challenge is to find the shortest possible string that contains all the given strings in the array `words` as substrings. It's important to note that none of the strings in `words` is a substring of any other string in the array, simplifying the problem. The task is to construct a superstring that concatenates all individual strings in such a way that they overlap wherever possible to reduce the overall length. If there are several shortest superstrings, returning any one of them is acceptable.

## Intuition

To solve this problem, we think about overlaps between pairs of strings to determine which strings should be concatenated together. To start, we need to know how to combine two strings together to achieve the maximum overlap, which means finding how a string can partially overlap the end of another string.

The intuition for the solution approach involves calculating the overlap between each pair of strings, which is the maximum prefix of one string that is also a suffix of the other. This forms a graph-like structure where the overlap value can represent the edge weight. To find the shortest superstring, we aim to visit each node (string) while maximizing the total weight of the edges we have traversed.

To find the order of strings that maximizes the overlaps, we can use [dynamic programming](/problems/dynamic_programming_intro). A dynamic programming table `dp[mask][i]` can be constructed where `mask` is a [bitmask](/problems/bitmask_intro) representing the set of strings included, and `i` is the last string added. The value represents the maximum overlap we can get so far using the strings in `mask`, ending with the string `i`.

However, since the exact order of strings that leads to this maximum value is also required to construct the superstring, we need a way to backtrack from the maximum overlap value to the start. This is done using another table `parent[mask][i]` that records the second to last string added before `i`.

Once the [dynamic programming](/problems/dynamic_programming_intro) table is filled, we can reconstruct the path from the maximum value, and then we add the remaining strings without overlaps to complete the superstring. The superstring is then constructed by taking the first string in order and adding the suffixes of all subsequent strings according to the overlap.

The final output is constructed by joining these strings. This way, we arrive at a superstring with the smallest possible length that still includes all strings from the `words` array as substrings.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) and [Bitmask](/problems/bitmask_intro) patterns.**

## Solution Approach

The solution uses a combination of [dynamic programming](/problems/dynamic_programming_intro) with bit masking and graph theory to construct the shortest superstring. Here's a detailed walkthrough of the implementation:

1. **Graph Creation with Overlaps**: 
    - A 2D array `overlap[][]` is created to store the overlaps. `overlap[i][j]` means the length of the overlap between the end of word `i` and the start of word `j`.
    - Nested loops iterate through each pair of words `word1` and `word2` in `words`, and for each pair, a third loop tries overlap lengths from longest to shortest and stops at the first match.

2. **[Dynamic Programming](/problems/dynamic_programming_intro) (DP) Table Initialization**:
    - A 2D DP array `dp[][]` is declared with `1 << n` rows and `n` columns. Each cell `dp[mask][i]` will store the maximum overlap obtainable with the last string in the subset `mask` being `words[i]`.
    - To reconstruct the optimal combination of string overlaps, a predecessor array `parent[][]` is maintained to track the choice of previous word in the optimal path.

3. **DP and Bitmasking**:
    - The algorithm iterates over all possible subsets `mask` (represented as bitmasks) and each word `j` in `mask`. For each subset-word pair, it calculates the maximum overlap by appending word `j` after some word `k` of `prev_mask` (the subset without word `j`): `dp[prev_mask][k] + overlap[k][j]`.
    - The idea is to transition from a smaller subset (without word `j`) to a larger subset (with word `j`) and update the DP table with the max overlap.

4. **Reconstructing the Path**:
    - After populating the DP table, the last word `j` in the optimal path (leading to the maximum overlap in the full set) is found.
    - From there, using the `parent` table, the algorithm reconstructs the order of strings, starting from the full set (all bits set) and moving backwards by removing the current word from the mask and jumping to its predecessor.

5. **Building the Shortest Superstring**:
    - Once the order is found, each word is added to the result in a specific way:
        - The first word is added in full.
        - Every subsequent word is added, minus the prefix that overlaps with the end of the current superstring (this overlap length is stored in `overlap[prev_index][index]`).
    - The backtracking stops early when `parent` is `-1`, which happens when the words before that point contribute no overlap at all (for example, when every overlap is 0). Any words not reached by the backtracking are appended after the traced words. This never makes the result longer than optimal, because the skipped words added zero overlap in the optimal arrangement.

This solution uses **[dynamic programming](/problems/dynamic_programming_intro) for subset optimization combined with graph theory concepts** (finding the longest path in an edge-weighted graph, where edges represent word overlaps). The superstring is formed by selecting substrings that give the maximum overlap when concatenated together. This results in space and time complexity that is exponential relative to the number of strings due to the bitmasking approach (`O(n^2 * 2^n)`), which is acceptable here given the problem's constraints.

## Example Walkthrough

Let's consider the `words` array `["catg", "ctaagt", "gcta", "ttca", "atgcatc"]` (indices 0 to 4). Here's how the solution operates on this example:

**Step 1: Graph Creation with Overlaps**
- For each ordered pair of words, we calculate the longest suffix of the first word that equals a prefix of the second. The nonzero overlaps are:
    - `gcta` → `ctaagt`: `cta`, length 3
    - `catg` → `atgcatc`: `atg`, length 3
    - `ttca` → `catg`: `ca`, length 2
    - `ctaagt` → `ttca`: `t`, length 1
    - `catg` → `gcta`, `gcta` → `atgcatc`, `ttca` → `atgcatc`, `atgcatc` → `catg`, `atgcatc` → `ctaagt`: length 1 each
- Every other pair, such as `catg` → `ctaagt`, has overlap 0.

This results in the 5×5 overlap array `overlap[][]`.

**Step 2: Dynamic Programming (DP) Table Initialization**
- Create a `dp` table with `1 << 5 = 32` rows (since there are 5 strings) and 5 columns.
- Create a `parent` table with the same dimensions to trace the optimal path.

**Step 3: DP and Bitmasking**
- Iterate over all subsets of `words` (represented by bitmasks) and try to extend each subset by adding one word at the end.
- For example, for the subset `{gcta, ctaagt}` ending with `ctaagt`, the value is `dp[{gcta}][gcta] + overlap[gcta][ctaagt] = 0 + 3 = 3`, and `parent` records `gcta`.
- For the full set `11111`, the final row is `dp[11111] = [6, 8, 6, 7, 9]`, so the largest total overlap is 9, ending with `atgcatc` (index 4).

**Step 4: Reconstructing the Path**
- Trace back from the full set `11111` and word 4 using `parent`. The order obtained after reversing is `["gcta", "ctaagt", "ttca", "catg", "atgcatc"]`.

**Step 5: Building the Shortest Superstring**
- Start by adding `gcta` in full: `gcta`.
- Add `ctaagt` minus the overlapping prefix `cta` (length 3): `gctaagt`.
- Add `ttca` minus the overlapping prefix `t` (length 1): `gctaagttca`.
- Add `catg` minus the overlapping prefix `ca` (length 2): `gctaagttcatg`.
- Add `atgcatc` minus the overlapping prefix `atg` (length 3): `gctaagttcatgcatc`.

The final superstring `gctaagttcatgcatc` has length 16, which equals the total length of all words (25) minus the total overlap (9). It contains every string from `words` as a substring.

The DP table `dp[mask][i]` holds the maximum overlap value, and `parent[mask][i]` lets us trace back the string sequence. We then build the superstring using these overlaps, so each string appears as a substring while the total length is minimized.

## Python Solution

```python
from typing import List

class Solution:
    def shortestSuperstring(self, words: List[str]) -> str:
        # Number of words
        num_words = len(words)
      
        # Graph g where g[i][j] represents the length of overlap if word i is followed by word j
        overlap = [[0] * num_words for _ in range(num_words)]
      
        # Calculate the maximum overlap of each pair of words
        for i, word1 in enumerate(words):
            for j, word2 in enumerate(words):
                if i != j:
                    # Check the maximum prefix of word2 that's a suffix of word1
                    for k in range(min(len(word1), len(word2)), 0, -1):
                        if word1[-k:] == word2[:k]:
                            overlap[i][j] = k
                            break
      
        # Dynamic programming table, where dp[mask][i] holds the highest overlap with mask and ending with word i
        dp = [[0] * num_words for _ in range(1 << num_words)]
      
        # Parent table to reconstruct the path
        parent = [[-1] * num_words for _ in range(1 << num_words)]
      
        # Fill dp[] and parent[]
        for mask in range(1 << num_words):
            for j in range(num_words):
                if (mask >> j) & 1:
                    # Previous mask before adding word j
                    prev_mask = mask ^ (1 << j)
                    for k in range(num_words):
                        if (prev_mask >> k) & 1:
                            value = dp[prev_mask][k] + overlap[k][j]
                            if value > dp[mask][j]:
                                dp[mask][j] = value
                                parent[mask][j] = k
      
        # Recover the last word in the optimal arrangement
        last = max(range(num_words), key=lambda i: dp[-1][i])
      
        # Recover the order of words in the optimal superstring
        order = []
        mask = (1 << num_words) - 1
        while last != -1:
            prev = parent[mask][last]
            order.append(last)
            mask ^= (1 << last)
            last = prev
        order.reverse()
      
        # Add words that are not in the optimal order (could happen if all overlaps are 0)
        order.extend(j for j in range(num_words) if j not in order)
      
        # Build the shortest superstring based on the order
        result = [words[order[0]]]
        for i in range(1, len(order)):
            index = order[i]
            prev_index = order[i - 1]
            result.append(words[index][overlap[prev_index][index]:])
      
        return ''.join(result)

# An example of how to use the class would be:
# solution = Solution()
# superstring = solution.shortestSuperstring(["abc", "bcd", "cde"])
```

## Time and Space Complexity

### Time Complexity

The time complexity of the code is `O(n^2 * 2^n + n^2 * W^2)`, where `n` is the number of words and `W` is the maximum length of a word.

1. The overlap computation runs over all `n^2` ordered pairs of words. For each pair it tries up to `W` overlap lengths, and each comparison `word1[-k:] == word2[:k]` costs `O(W)`, giving `O(n^2 * W^2)`.

2. The DP loop fills `dp` and `parent` in `O(n^2 * 2^n)` time: for each of the `2^n` masks it tries every last word `j` and every previous word `k`.

3. Finding the best last word in the full mask takes `O(n)`.

4. Reconstructing the order takes `O(n)` for the backtracking. The line that appends missing words uses `j not in order`, which is `O(n^2)` in the worst case.

5. Building the output string takes `O(n * W)`.

The DP loop dominates for the problem's constraints (`n <= 12`, `W <= 20`).

### Space Complexity

The space complexity of the code is `O(n * 2^n + n^2)`, which comprises:

1. `O(n^2)` for storing the overlap between pairs of words in `overlap`.

2. `O(n * 2^n)` for storing the DP values in `dp`.

3. `O(n * 2^n)` for storing the predecessors in `parent`.

The output string itself takes `O(n * W)` space.

**Learn more about [how to find time and space complexity quickly using problem constraints](/problems/runtime_summary).**

