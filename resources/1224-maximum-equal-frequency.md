# 1224. Maximum Equal Frequency

> Source: https://algo.monster/liteproblems/1224

## Problem Statement

You are given an array `nums` containing positive integers. Your task is to find the longest possible prefix of this array where you can remove **exactly one** element, and after removal, all remaining numbers have the same frequency (number of occurrences).

For example, if you have a prefix where some numbers appear 3 times and others appear 2 times, you need to check if removing one element can make all numbers have the same frequency - either all appearing 2 times or all appearing 3 times.

Key points to understand:
- You must remove exactly one element from the prefix
- After removal, every distinct number that remains must appear the same number of times
- If removing one element leaves the prefix empty, this is valid (all numbers have 0 occurrences)
- You want the longest such prefix

The solution tracks two important pieces of information:
- `cnt`: How many times each number appears in the current prefix
- `ccnt`: How many numbers have a specific count (e.g., how many numbers appear exactly 2 times)

The algorithm checks three main conditions for a valid prefix of length `i`:
1. **All numbers appear once** (`mx == 1`): Removing any single element leaves all others with 0 occurrences, which satisfies the requirement
2. **Numbers have counts of `mx` and `mx-1`, with only one number at count `mx`**: Removing one occurrence of the number with `mx` appearances makes all numbers have `mx-1` occurrences
3. **All numbers except one appear `mx` times, and one number appears once**: Removing the single occurrence makes all remaining numbers have `mx` occurrences

## Intuition

The key insight is recognizing that for a prefix to be valid after removing one element, the frequency distribution must follow very specific patterns. Let's think about what makes a removal valid.

When we remove one element from a prefix, we're essentially decreasing the count of that particular number by 1. For all remaining numbers to have equal frequency afterward, we need to start with a frequency distribution that's "almost uniform" - meaning it's just one element away from being uniform.

What patterns allow this? Let's think through the possibilities:

1. **Everything appears once**: If every number appears exactly once, removing any element leaves an empty occurrence for that number, and all others remain at 0 occurrences. This is trivially valid.

2. **Two frequency levels differing by 1**: Imagine we have some numbers appearing `k` times and others appearing `k-1` times. If exactly one number appears `k` times, we can remove one instance of it to bring it down to `k-1`, matching all others.

3. **One outlier with frequency 1**: If all numbers except one appear `k` times, and that outlier appears just once, we can remove that single occurrence entirely, leaving all remaining numbers at frequency `k`.

The clever part is realizing we don't need to track the entire frequency distribution in detail - we just need to know:
- The maximum frequency (`mx`)
- How many numbers have each frequency (`ccnt`)
- The total count of elements seen so far

By maintaining these counters as we traverse the array, we can check at each position whether the current prefix satisfies one of our valid patterns. The mathematical conditions in the code directly correspond to these three scenarios:

- `mx == 1` checks pattern 1
- `ccnt[mx] * mx + ccnt[mx - 1] * (mx - 1) == i and ccnt[mx] == 1` checks pattern 2
- `ccnt[mx] * mx + 1 == i and ccnt[1] == 1` checks pattern 3

This approach is efficient because we're not actually trying different removals - we're mathematically determining if the current frequency distribution allows for a valid removal.

## Solution Approach

We use two hash tables (Counters in Python) to track the frequency information:
- `cnt`: Maps each element value to its frequency in the current prefix
- `ccnt`: Maps each frequency to how many elements have that frequency
- `mx`: Tracks the maximum frequency among all elements

Let's walk through the implementation step by step:

**1. Initialize data structures:**
```python
cnt = Counter()    # element -> frequency
ccnt = Counter()   # frequency -> count of elements with that frequency
ans = mx = 0       # ans: result, mx: max frequency
```

**2. Process each element with 1-based indexing:**
For each element `v` at position `i` (1-indexed):

**3. Update the frequency counters:**
```python
if v in cnt:
    ccnt[cnt[v]] -= 1  # Decrease count for old frequency
cnt[v] += 1            # Increment frequency of v
mx = max(mx, cnt[v])   # Update maximum frequency
ccnt[cnt[v]] += 1     # Increase count for new frequency
```

This maintains the invariant that `ccnt[k]` tells us how many distinct elements appear exactly `k` times.

**4. Check the three valid conditions:**

**Condition 1: All elements appear once**
```python
if mx == 1:
    ans = i
```
When every element appears exactly once, removing any element results in uniform frequencies (0 for the removed element, 1 for others becomes 0 after removal).

**Condition 2: Two frequency levels (mx and mx-1) with one element at mx**
```python
elif ccnt[mx] * mx + ccnt[mx - 1] * (mx - 1) == i and ccnt[mx] == 1:
    ans = i
```
- `ccnt[mx] * mx + ccnt[mx - 1] * (mx - 1) == i` verifies that all elements have frequencies of either `mx` or `mx-1`
- `ccnt[mx] == 1` ensures only one element has the maximum frequency
- We can remove one occurrence of the element with frequency `mx` to make all frequencies equal to `mx-1`

**Condition 3: All elements at frequency mx except one outlier at frequency 1**
```python
elif ccnt[mx] * mx + 1 == i and ccnt[1] == 1:
    ans = i
```
- `ccnt[mx] * mx + 1 == i` checks that the total count equals all elements at frequency `mx` plus one extra
- `ccnt[1] == 1` confirms exactly one element appears once
- We can remove the single-occurrence element entirely

**5. Return the maximum valid prefix length:**
The algorithm continuously updates `ans` whenever a valid prefix is found, ensuring we get the longest possible valid prefix.

The time complexity is `O(n)` where `n` is the length of the array, as we process each element once. The space complexity is `O(n)` for the hash tables in the worst case where all elements are unique.

## Example Walkthrough

Let's walk through the solution with the array `nums = [2, 2, 1, 1, 5, 3, 3, 5]`.

We'll track `cnt` (element frequencies), `ccnt` (frequency counts), and check our three conditions at each step.

**Initial state:** `cnt = {}`, `ccnt = {}`, `mx = 0`, `ans = 0`

**i = 1, element = 2:**
- Update: `cnt = {2: 1}`, `ccnt = {1: 1}`, `mx = 1`
- Check conditions:
  - `mx == 1` ✓ (all elements appear once)
- `ans = 1`

**i = 2, element = 2:**
- Update: `cnt = {2: 2}`, `ccnt = {1: 0, 2: 1}`, `mx = 2`
- Check conditions:
  - `mx == 1` ✗
  - `ccnt[2] * 2 + ccnt[1] * 1 = 1*2 + 0*1 = 2 == i` ✓ and `ccnt[2] == 1` ✓
- `ans = 2` (we can remove one '2' to make all frequencies equal to 1)

**i = 3, element = 1:**
- Update: `cnt = {2: 2, 1: 1}`, `ccnt = {1: 1, 2: 1}`, `mx = 2`
- Check conditions:
  - `mx == 1` ✗
  - `ccnt[2] * 2 + ccnt[1] * 1 = 1*2 + 1*1 = 3 == i` ✓ and `ccnt[2] == 1` ✓
- `ans = 3` (remove one '2' to get all frequencies equal to 1)

**i = 4, element = 1:**
- Update: `cnt = {2: 2, 1: 2}`, `ccnt = {1: 0, 2: 2}`, `mx = 2`
- Check conditions:
  - `mx == 1` ✗
  - `ccnt[2] * 2 + ccnt[1] * 1 = 2*2 + 0*1 = 4 == i` ✓ but `ccnt[2] == 2` ✗
  - `ccnt[2] * 2 + 1 = 2*2 + 1 = 5 ≠ 4` ✗
- No update to `ans`

**i = 5, element = 5:**
- Update: `cnt = {2: 2, 1: 2, 5: 1}`, `ccnt = {1: 1, 2: 2}`, `mx = 2`
- Check conditions:
  - `mx == 1` ✗
  - `ccnt[2] * 2 + ccnt[1] * 1 = 2*2 + 1*1 = 5 == i` ✓ but `ccnt[2] == 2` ✗
  - `ccnt[2] * 2 + 1 = 2*2 + 1 = 5 == i` ✓ and `ccnt[1] == 1` ✓
- `ans = 5` (remove the single '5' to get all frequencies equal to 2)

**i = 6, element = 3:**
- Update: `cnt = {2: 2, 1: 2, 5: 1, 3: 1}`, `ccnt = {1: 2, 2: 2}`, `mx = 2`
- Check conditions:
  - `mx == 1` ✗
  - `ccnt[2] * 2 + ccnt[1] * 1 = 2*2 + 2*1 = 6 == i` ✓ but `ccnt[2] == 2` ✗
  - `ccnt[2] * 2 + 1 = 2*2 + 1 = 5 ≠ 6` ✗
- No update to `ans`

**i = 7, element = 3:**
- Update: `cnt = {2: 2, 1: 2, 5: 1, 3: 2}`, `ccnt = {1: 1, 2: 3}`, `mx = 2`
- Check conditions:
  - `mx == 1` ✗
  - `ccnt[2] * 2 + ccnt[1] * 1 = 3*2 + 1*1 = 7 == i` ✓ but `ccnt[2] == 3` ✗
  - `ccnt[2] * 2 + 1 = 3*2 + 1 = 7 == i` ✓ and `ccnt[1] == 1` ✓
- `ans = 7` (remove the single '5' to get all frequencies equal to 2)

**i = 8, element = 5:**
- Update: `cnt = {2: 2, 1: 2, 5: 2, 3: 2}`, `ccnt = {1: 0, 2: 4}`, `mx = 2`
- Check conditions: None satisfied
- Final `ans = 7`

The longest valid prefix has length 7, where we can remove the element '5' (which appears once) to make all remaining elements appear exactly 2 times.

## Python Solution

```python
class Solution:
    def maxEqualFreq(self, nums: List[int]) -> int:
        # freq_count: maps each number to its frequency
        freq_count = {}
      
        # freq_freq_count: maps each frequency to how many numbers have that frequency
        freq_freq_count = {}
      
        max_length = 0
        max_frequency = 0
      
        for index, num in enumerate(nums, 1):
            # If number already exists, update frequency of frequencies
            if num in freq_count:
                old_freq = freq_count[num]
                freq_freq_count[old_freq] -= 1
                if freq_freq_count[old_freq] == 0:
                    del freq_freq_count[old_freq]
          
            # Increment frequency of current number
            freq_count[num] = freq_count.get(num, 0) + 1
            new_freq = freq_count[num]
          
            # Update maximum frequency seen
            max_frequency = max(max_frequency, new_freq)
          
            # Update frequency of frequencies
            freq_freq_count[new_freq] = freq_freq_count.get(new_freq, 0) + 1
          
            # Check if current prefix is valid for removal of one element
          
            # Case 1: All elements appear exactly once (can remove any)
            if max_frequency == 1:
                max_length = index
          
            # Case 2: All elements have frequency max_frequency except one has (max_frequency - 1)
            # Remove one occurrence from the element with max_frequency
            elif (freq_freq_count.get(max_frequency, 0) * max_frequency + 
                  freq_freq_count.get(max_frequency - 1, 0) * (max_frequency - 1) == index and 
                  freq_freq_count.get(max_frequency, 0) == 1):
                max_length = index
          
            # Case 3: All elements have frequency max_frequency except one element appears once
            # Remove the element that appears once
            elif (freq_freq_count.get(max_frequency, 0) * max_frequency + 1 == index and 
                  freq_freq_count.get(1, 0) == 1):
                max_length = index
          
            # Case 4: All elements have same frequency f, and one has frequency f+1
            # This is handled by Case 2 when max_frequency = f+1
          
            # Case 5: Only one unique number (all same)
            elif len(freq_count) == 1:
                max_length = index
          
            # Case 6: All have same frequency and total elements = (frequency + 1) * unique_count
            # Remove one from any element
            elif (len(freq_freq_count) == 1 and 
                  (max_frequency + 1) * freq_freq_count.get(max_frequency, 0) == index):
                max_length = index
      
        return max_length
```

## Time and Space Complexity

The time complexity is `O(n)`, where `n` is the length of the `nums` array. The algorithm iterates through the array once with a single loop. Within each iteration, all operations are constant time: accessing and updating dictionary values (`cnt` and `ccnt`), arithmetic operations, and comparisons. Even though we use `Counter` objects and perform dictionary operations, each individual operation takes `O(1)` time.

The space complexity is `O(n)`. In the worst case, when all elements in `nums` are unique, the `cnt` dictionary will store `n` different keys. The `ccnt` dictionary stores the frequency of frequencies, which is bounded by `O(n)` as well (at most `n` different frequency values). The variables `ans`, `mx`, and `i` use constant space. Therefore, the overall space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Missing Edge Cases in Validation Conditions**

One of the most common pitfalls is not covering all valid scenarios where removing one element creates uniform frequencies. The code above actually has redundant conditions (Cases 5 and 6) that attempt to patch missing logic, making it harder to understand and potentially buggy.

**The Problem:**
The three main conditions might miss valid prefixes like:
- When all elements have the same frequency `f`, and removing one element from any of them creates uniform frequency `f-1`
- When there's only one unique element in the prefix

**The Fix:**
Consolidate the validation logic into clear, comprehensive conditions:

```python
def maxEqualFreq(self, nums: List[int]) -> int:
    cnt = {}      # element -> frequency
    ccnt = {}     # frequency -> count of elements with that frequency
    ans = mx = 0
  
    for i, v in enumerate(nums, 1):
        # Update frequencies
        if v in cnt:
            ccnt[cnt[v]] -= 1
            if ccnt[cnt[v]] == 0:
                del ccnt[cnt[v]]
      
        cnt[v] = cnt.get(v, 0) + 1
        mx = max(mx, cnt[v])
        ccnt[cnt[v]] = ccnt.get(cnt[v], 0) + 1
      
        # Check valid conditions
        # Case 1: All elements appear once
        if mx == 1:
            ans = i
      
        # Case 2: Only one unique element (can remove one occurrence)
        elif len(cnt) == 1:
            ans = i
      
        # Case 3: Two frequency levels (mx and mx-1) with one element at mx
        elif (len(ccnt) == 2 and 
              mx - 1 in ccnt and 
              (ccnt[mx] == 1 and ccnt[mx] * mx + ccnt[mx - 1] * (mx - 1) == i)):
            ans = i
      
        # Case 4: All at frequency mx except one element appears once
        elif (len(ccnt) == 2 and 
              1 in ccnt and 
              ccnt[1] == 1 and 
              ccnt[mx] * mx + 1 == i):
            ans = i
      
        # Case 5: All have same frequency, removing one creates uniform freq
        elif len(ccnt) == 1 and (mx * len(cnt) == i + 1):
            ans = i
  
    return ans
```

### 2. **Incorrect Frequency Counter Management**

**The Problem:**
Forgetting to remove entries from `ccnt` when their count becomes zero leads to incorrect validation checks. The dictionary might contain keys with value 0, causing conditions like `mx - 1 in ccnt` to incorrectly return True.

**The Fix:**
Always clean up zero-count entries:

```python
if v in cnt:
    ccnt[cnt[v]] -= 1
    if ccnt[cnt[v]] == 0:
        del ccnt[cnt[v]]  # Critical: remove zero entries
```

### 3. **Off-by-One Errors with Indexing**

**The Problem:**
Mixing 0-based and 1-based indexing when checking if the total count equals the prefix length. The enumerate starts at 1, but forgetting this can lead to incorrect validation.

**The Fix:**
Be consistent with 1-based indexing throughout:

```python
for i, v in enumerate(nums, 1):  # i is 1-based
    # All checks use i as the current prefix length
    if ccnt[mx] * mx + ccnt[mx - 1] * (mx - 1) == i:  # Correct
```

### 4. **Not Handling the "All Same Frequency" Case Properly**

**The Problem:**
When all elements have the same frequency `f`, we can remove one occurrence from any element to achieve uniform frequency `f-1`. This case is often missed or incorrectly implemented.

**The Fix:**
Add explicit check for this scenario:

```python
# All elements have same frequency f
# Total = f * unique_count
# After removing one: (f-1) * unique_count + (unique_count - 1)
elif len(ccnt) == 1 and mx * len(cnt) == i + 1:
    ans = i
```

This ensures we catch prefixes where all elements appear the same number of times and removing one element from any of them creates a valid uniform distribution.

