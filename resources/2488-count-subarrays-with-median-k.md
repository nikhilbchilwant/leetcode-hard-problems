# 2488. Count Subarrays With Median K

> Source: https://algo.monster/liteproblems/2488

## Problem Statement

You are given an array `nums` of size `n` containing distinct integers from `1` to `n`, and a positive integer `k`.

Your task is to find the number of non-empty subarrays in `nums` that have a median equal to `k`.

**Key definitions:**
- **Median**: The middle element of an array after sorting it in ascending order. If the array has an even number of elements, the median is the left middle element (the smaller of the two middle elements).
  - For example: 
    - The median of `[2,3,1,4]` is `2` (after sorting: `[1,2,3,4]`, the left middle element is `2`)
    - The median of `[8,4,3,5,1]` is `4` (after sorting: `[1,3,4,5,8]`, the middle element is `4`)
- **Subarray**: A contiguous portion of the original array

The problem asks you to count how many subarrays exist where, if you were to sort that subarray, the median would be exactly `k`.

For instance, if `nums = [3,2,1,4,5]` and `k = 4`, you need to find all contiguous subarrays where `4` would be the median after sorting that subarray.

## Intuition

Since the array contains distinct integers from `1` to `n`, and we're looking for subarrays with median `k`, we can use a key observation: for `k` to be the median of a subarray, the number of elements greater than `k` and the number of elements less than `k` must be balanced in a specific way.

For an odd-length subarray, `k` is the median when there are equal numbers of elements greater than and less than `k`. For an even-length subarray, `k` is the median when there's exactly one more element less than `k` than elements greater than `k`.

This leads us to think about the problem differently: instead of checking every possible subarray and computing its median, we can track the **balance** between elements greater than `k` and elements less than `k`.

Let's define a balance score: for each element, if it's greater than `k`, we add `+1`; if it's less than `k`, we add `-1`. For a subarray containing `k`:
- If the balance is `0`, we have an odd-length subarray with `k` as median
- If the balance is `-1`, we have an even-length subarray with `k` as median

Since every valid subarray must contain `k` itself, we can start from `k`'s position and expand outward. We traverse to the right of `k`, keeping track of the balance scores we encounter. Then, when we traverse to the left of `k`, we look for complementary balances on the right side that would make the total balance either `0` or `-1`.

For example, if we have a balance of `+2` on the left side of `k`, we need a balance of `-2` (for odd-length) or `-1` (for even-length) on the right side to create a valid subarray with median `k`.

This approach transforms the problem from checking all subarrays to efficiently counting valid combinations using the balance scores, reducing our time complexity significantly.

**Learn more about [Prefix Sum](/problems/subarray_sum) patterns.**

## Solution Approach

The implementation follows the intuition by tracking balance scores as we traverse from `k`'s position in both directions.

**Step 1: Find the position of k**
```python
i = nums.index(k)
```
First, we locate where `k` appears in the array, storing its index as `i`.

**Step 2: Initialize variables**
- `ans = 1`: Start with 1 because the single element `[k]` itself is a valid subarray with median `k`
- `cnt = Counter()`: A dictionary to store balance scores from the right side
- `x = 0`: Tracks the running balance score

**Step 3: Traverse right from k**
```python
for v in nums[i + 1 :]:
    x += 1 if v > k else -1
    ans += 0 <= x <= 1
    cnt[x] += 1
```
Starting from position `i + 1`, we traverse to the right:
- For each element `v`, update balance `x`: add `+1` if `v > k`, otherwise `-1`
- If `x` is `0` or `1`, the subarray from `k` to current position has median `k`, so increment `ans`
- Store the balance `x` in counter `cnt` for later use when combining with left side

**Step 4: Traverse left from k**
```python
x = 0
for j in range(i - 1, -1, -1):
    x += 1 if nums[j] > k else -1
    ans += 0 <= x <= 1
    ans += cnt[-x] + cnt[-x + 1]
```
Reset `x = 0` and traverse leftward from position `i - 1`:
- Update balance `x` the same way: `+1` if element > `k`, otherwise `-1`
- If `x` is `0` or `1`, the subarray from current position to `k` has median `k`, so increment `ans`
- Check for complementary balances on the right side:
  - `cnt[-x]`: Creates subarrays with total balance `0` (odd-length with median `k`)
  - `cnt[-x + 1]`: Creates subarrays with total balance `1` (even-length with median `k`)
  - Add both counts to `ans`

**Why this works:**
When we have a left balance of `x` and a right balance of `y`, the total balance of the subarray spanning both sides is `x + y`. For `k` to be the median:
- Total balance should be `0` (odd-length) → need `y = -x`
- Total balance should be `1` (even-length) → need `y = -x + 1`

The algorithm efficiently counts all valid subarrays in `O(n)` time using `O(n)` space for the counter.

## Example Walkthrough

Let's walk through the solution with `nums = [2, 5, 1, 4, 3]` and `k = 4`.

**Step 1: Find position of k**
- `k = 4` is at index `i = 3`

**Step 2: Traverse right from k (index 4)**
- Start with `x = 0`, `ans = 1` (counting `[4]` itself)
- Process element `3` at index 4:
  - Since `3 < 4`, add `-1`: `x = -1`
  - Check if `0 ≤ x ≤ 1`: No, so don't increment `ans`
  - Store in counter: `cnt[-1] = 1`

Right side balances collected: `cnt = {-1: 1}`

**Step 3: Traverse left from k (indices 2, 1, 0)**
- Reset `x = 0`

Process index 2 (element `1`):
- Since `1 < 4`, add `-1`: `x = -1`
- Check if `0 ≤ x ≤ 1`: No
- Look for complementary balances:
  - `cnt[-(-1)] = cnt[1]` = 0 (not found)
  - `cnt[-(-1) + 1] = cnt[2]` = 0 (not found)
- `ans` remains 1

Process index 1 (element `5`):
- Since `5 > 4`, add `+1`: `x = -1 + 1 = 0`
- Check if `0 ≤ x ≤ 1`: Yes! Increment `ans` to 2
  - This counts subarray `[5, 1, 4]` with balance 0
- Look for complementary balances:
  - `cnt[-0] = cnt[0]` = 0 (not found)
  - `cnt[-0 + 1] = cnt[1]` = 0 (not found)
- `ans` remains 2

Process index 0 (element `2`):
- Since `2 < 4`, add `-1`: `x = 0 - 1 = -1`
- Check if `0 ≤ x ≤ 1`: No
- Look for complementary balances:
  - `cnt[-(-1)] = cnt[1]` = 0 (not found)
  - `cnt[-(-1) + 1] = cnt[2]` = 0 (not found)
- `ans` remains 2

**Valid subarrays found:**
1. `[4]` - single element, median is 4
2. `[5, 1, 4]` - sorted: `[1, 4, 5]`, median is 4

Final answer: `2`

## Python Solution

```python
class Solution:
    def countSubarrays(self, nums: List[int], k: int) -> int:
        # Find the index of element k in the array
        k_index = nums.index(k)
      
        # Dictionary to store balance counts from the right side of k
        right_balance_count = Counter()
      
        # Initialize result with 1 (subarray containing only k itself)
        result = 1
      
        # Calculate balance for elements to the right of k
        # Balance: +1 for elements > k, -1 for elements < k
        balance = 0
        for value in nums[k_index + 1:]:
            if value > k:
                balance += 1
            else:
                balance -= 1
          
            # Count subarrays starting at k and ending at current position
            # Valid if balance is 0 (equal elements on both sides) or 1 (one more larger element)
            if 0 <= balance <= 1:
                result += 1
          
            # Store the balance count for later use with left side
            right_balance_count[balance] += 1
      
        # Calculate balance for elements to the left of k
        balance = 0
        for j in range(k_index - 1, -1, -1):
            if nums[j] > k:
                balance += 1
            else:
                balance -= 1
          
            # Count subarrays starting at current position and ending at k
            # Valid if balance is 0 or 1
            if 0 <= balance <= 1:
                result += 1
          
            # Count subarrays that span both sides of k
            # We need the total balance to be 0 or 1
            # So we look for right balances that complement the left balance
            result += right_balance_count[-balance] + right_balance_count[-balance + 1]
      
        return result
```

## Time and Space Complexity

**Time Complexity:** `O(n)`

The algorithm performs the following operations:
- Finding the index of `k` using `nums.index(k)`: `O(n)` in the worst case
- First loop iterating through elements from index `i+1` to the end: `O(n-i-1)` operations
- Second loop iterating from index `i-1` down to 0: `O(i)` operations
- Within each loop iteration, all operations (arithmetic, dictionary access/update, counter increments) are `O(1)`

Total time complexity: `O(n) + O(n-i-1) + O(i) = O(n)`

**Space Complexity:** `O(n)`

The algorithm uses:
- `cnt` Counter/dictionary that stores at most `n` different values of `x` (in the worst case, each element after index `i` produces a unique cumulative sum)
- A few integer variables (`i`, `ans`, `x`): `O(1)`

The Counter `cnt` can have at most `O(n)` entries since the value of `x` changes by ±1 at each step and we process at most `n` elements, leading to a space complexity of `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrectly Handling the Balance Calculation**
A common mistake is confusing when to add +1 or -1 to the balance. Some might incorrectly write:
```python
# WRONG: This reverses the logic
balance += 1 if value < k else -1
```
This error completely breaks the algorithm because the balance no longer correctly represents the difference between elements greater than k and elements less than k.

**Solution:** Remember that we want to track how many MORE elements are greater than k compared to those less than k. Always use `+1` for elements greater than k and `-1` for elements less than k.

### 2. **Forgetting to Initialize Result with 1**
Some implementations might start with `result = 0`, forgetting that the single-element subarray `[k]` itself is valid:
```python
# WRONG: Missing the single element [k] case
result = 0
```

**Solution:** Always initialize with `result = 1` since a subarray containing only k has k as its median by definition.

### 3. **Using Wrong Complement Values When Combining Left and Right**
A critical error is using the wrong complement when checking subarrays that span both sides:
```python
# WRONG: Using positive values instead of negative
result += right_balance_count[balance] + right_balance_count[balance + 1]
```
This fails because if the left side has balance `x`, we need the right side to have balance `-x` or `-x + 1` to achieve a total balance of 0 or 1.

**Solution:** Always use negative complements: `right_balance_count[-balance]` and `right_balance_count[-balance + 1]`.

### 4. **Not Handling the Case Where k Doesn't Exist in the Array**
While the problem states k exists in the array, defensive programming might add error handling:
```python
# WRONG: Will throw ValueError if k not in nums
k_index = nums.index(k)
```

**Solution:** For production code, consider adding validation:
```python
if k not in nums:
    return 0
k_index = nums.index(k)
```

### 5. **Confusing the Order of Operations**
Some might try to combine both traversals in a single loop or process left side before right side:
```python
# WRONG: Processing left side first won't have right balance counts ready
balance = 0
for j in range(k_index - 1, -1, -1):
    # ... trying to use right_balance_count which is empty
```

**Solution:** Always process the right side first to populate the `right_balance_count` dictionary, then process the left side to combine with the stored right balances.

