# 2040. Kth Smallest Product of Two Sorted Arrays

> Source: https://algo.monster/liteproblems/2040

## Problem Statement

You are given two sorted integer arrays `nums1` and `nums2` (both 0-indexed), and an integer `k`. Your task is to find the `k`-th smallest product among all possible products formed by multiplying one element from `nums1` with one element from `nums2`.

Specifically, you need to consider all products of the form `nums1[i] * nums2[j]` where `0 <= i < nums1.length` and `0 <= j < nums2.length`. After calculating all these products, sort them in ascending order and return the `k`-th value (using 1-based indexing, meaning `k=1` refers to the smallest product).

For example, if `nums1 = [2, 5]` and `nums2 = [3, 4]`, the possible products are:
- `2 * 3 = 6`
- `2 * 4 = 8`
- `5 * 3 = 15`
- `5 * 4 = 20`

Sorted in ascending order: `[6, 8, 15, 20]`. If `k = 2`, the answer would be `8`.

Note that the arrays can contain negative numbers, zero, or positive numbers, and the arrays are already sorted in non-decreasing order.

## Intuition

The naive approach would be to generate all possible products `nums1[i] * nums2[j]`, sort them, and return the `k`-th element. However, with arrays of size up to `10^5`, this would create up to `10^10` products, which is too memory-intensive and time-consuming.

The key insight is that we don't need to generate all products explicitly. Instead, we can use [binary search](/problems/binary_search_intro) on the answer space. If we pick a candidate value `p`, we can efficiently count how many products are less than or equal to `p` without generating all products.

Why [binary search](/problems/binary_search_intro)? Because if we can count how many products are `<= p` for any given `p`, we can find the smallest `p` such that exactly `k` products are `<= p`. This `p` would be our `k`-th smallest product.

The challenge becomes: given a value `p`, how do we count products `<= p` efficiently?

For each element `x` in `nums1`, we need to count how many elements `y` in `nums2` satisfy `x * y <= p`. This depends on the sign of `x`:

- If `x > 0`: We need `y <= p/x`. Since `nums2` is sorted and multiplication by positive `x` preserves order, we can use [binary search](/problems/binary_search_intro) to find the rightmost position where `nums2[j] <= p/x`.

- If `x < 0`: We need `y >= p/x` (inequality flips when dividing by negative). Since `nums2` is sorted and multiplication by negative `x` reverses order, larger values of `y` give smaller products. We can use [binary search](/problems/binary_search_intro) to find the leftmost position where `nums2[j] >= p/x`.

- If `x = 0`: The product is always `0`. If `p >= 0`, all elements in `nums2` contribute to the count.

By efficiently counting products for each candidate `p`, we can [binary search](/problems/binary_search_intro) for the exact `k`-th smallest product value. The search range is bounded by the maximum possible product magnitude, which is the product of the maximum absolute values from both arrays.

**Learn more about [Binary Search](/problems/binary_search_intro) patterns.**

## Solution Approach

The solution uses [binary search](/problems/binary_search_intro) on the answer space combined with a counting function. Here's the step-by-step implementation:

**1. Define the Search Range:**
First, we calculate the maximum possible absolute product value:
```python
mx = max(abs(nums1[0]), abs(nums1[-1])) * max(abs(nums2[0]), abs(nums2[-1]))
```
Since the arrays are sorted, the extreme values are at the endpoints. The search range becomes `[-mx, mx]`.

**2. Implement the Counting Function:**
The `count(p)` function calculates how many products are less than or equal to `p`:

```python
def count(p: int) -> int:
    cnt = 0
    n = len(nums2)
    for x in nums1:
        if x > 0:
            cnt += bisect_right(nums2, p / x)
        elif x < 0:
            cnt += n - bisect_left(nums2, p / x)
        else:
            cnt += n * int(p >= 0)
    return cnt
```

For each element `x` in `nums1`:
- **When `x > 0`**: We find how many elements in `nums2` satisfy `y <= p/x`. Using `bisect_right(nums2, p/x)` gives us the count of such elements since it returns the insertion position after all elements `<= p/x`.

- **When `x < 0`**: We need elements where `y >= p/x` (inequality flips). Using `bisect_left(nums2, p/x)` finds the first position where `nums2[j] >= p/x`. Therefore, `n - bisect_left(nums2, p/x)` gives the count of valid elements.

- **When `x = 0`**: All products equal `0`. If `p >= 0`, all `n` elements of `nums2` produce valid products; otherwise, none do.

**3. [Binary Search](/problems/binary_search_intro) for the k-th Smallest:**
The main binary search uses Python's `bisect_left` with a custom key function:

```python
return bisect_left(range(-mx, mx + 1), k, key=count) - mx
```

This searches for the leftmost position in the range `[-mx, mx]` where `count(p) >= k`. The `key=count` parameter transforms each value `p` in the range to `count(p)` before comparison.

The function effectively finds the smallest `p` such that at least `k` products are `<= p`. Since we're searching in `range(-mx, mx + 1)` which starts from 0, we subtract `mx` from the result to get the actual value.

**Time Complexity:** `O(m * log(n) * log(P))` where `m = len(nums1)`, `n = len(nums2)`, and `P` is the range of possible products. The outer [binary search](/problems/binary_search_intro) takes `O(log(P))` iterations, and each iteration calls `count()` which takes `O(m * log(n))` time.

**Space Complexity:** `O(1)` as we only use constant extra space beyond the input arrays.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Example:** `nums1 = [-2, 1, 3]`, `nums2 = [-1, 2, 4]`, `k = 5`

**Step 1: Calculate all possible products (for understanding)**
Let's first see what all products look like:
- `-2 * -1 = 2`
- `-2 * 2 = -4`
- `-2 * 4 = -8`
- `1 * -1 = -1`
- `1 * 2 = 2`
- `1 * 4 = 4`
- `3 * -1 = -3`
- `3 * 2 = 6`
- `3 * 4 = 12`

Sorted: `[-8, -4, -3, -1, 2, 2, 4, 6, 12]`
The 5th smallest is `2`.

**Step 2: Apply our binary search solution**

Calculate search range:
- `mx = max(|-2|, |3|) * max(|-1|, |4|) = 3 * 4 = 12`
- Search range: `[-12, 12]`

**Step 3: Binary search process**

We'll binary search for the smallest value `p` where `count(p) >= 5`.

Let's trace through some key iterations:

**When p = 0:**
- For `x = -2` (negative): Need `y >= 0/-2 = 0`. Elements in `nums2` that are `>= 0`: `[2, 4]`. Count = 2
- For `x = 1` (positive): Need `y <= 0/1 = 0`. Elements in `nums2` that are `<= 0`: `[-1]`. Count = 1
- For `x = 3` (positive): Need `y <= 0/3 = 0`. Elements in `nums2` that are `<= 0`: `[-1]`. Count = 1
- Total count(0) = 2 + 1 + 1 = 4 (less than k=5)

**When p = 2:**
- For `x = -2` (negative): Need `y >= 2/-2 = -1`. Elements in `nums2` that are `>= -1`: `[-1, 2, 4]`. Count = 3
- For `x = 1` (positive): Need `y <= 2/1 = 2`. Elements in `nums2` that are `<= 2`: `[-1, 2]`. Count = 2
- For `x = 3` (positive): Need `y <= 2/3 = 0.67`. Elements in `nums2` that are `<= 0.67`: `[-1]`. Count = 1
- Total count(2) = 3 + 2 + 1 = 6 (greater than or equal to k=5)

**When p = 1:**
- For `x = -2`: Need `y >= -0.5`. Elements `>= -0.5`: `[2, 4]`. Count = 2
- For `x = 1`: Need `y <= 1`. Elements `<= 1`: `[-1]`. Count = 1
- For `x = 3`: Need `y <= 0.33`. Elements `<= 0.33`: `[-1]`. Count = 1
- Total count(1) = 2 + 1 + 1 = 4 (less than k=5)

The binary search finds that `p = 2` is the smallest value where `count(p) >= 5`, which matches our expected answer.

**Key Insight:** We never actually computed all 9 products. Instead, we efficiently counted how many products were less than or equal to candidate values, allowing us to pinpoint the k-th smallest product through binary search.

## Python Solution

```python
class Solution:
    def kthSmallestProduct(self, nums1: List[int], nums2: List[int], k: int) -> int:
        def count_products_less_than_or_equal(target_product: int) -> int:
            """
            Count how many products from nums1[i] * nums2[j] are <= target_product
            """
            total_count = 0
            nums2_length = len(nums2)

            for num1 in nums1:
                if num1 > 0:
                    # For positive num1, find how many nums2[j] satisfy num1 * nums2[j] <= target_product
                    # This means nums2[j] <= target_product / num1
                    total_count += bisect_right(nums2, target_product / num1)
                elif num1 < 0:
                    # For negative num1, find how many nums2[j] satisfy num1 * nums2[j] <= target_product
                    # This means nums2[j] >= target_product / num1 (inequality flips for negative)
                    total_count += nums2_length - bisect_left(nums2, target_product / num1)
                else:
                    # For num1 = 0, product is always 0
                    # Count all of nums2 if target_product >= 0, otherwise count 0
                    total_count += nums2_length * int(target_product >= 0)

            return total_count

        # Calculate the maximum possible absolute value of products
        # This will be our search range boundary
        max_absolute_value = max(abs(nums1[0]), abs(nums1[-1])) * max(abs(nums2[0]), abs(nums2[-1]))

        # Binary search for the k-th smallest product in range [-max_absolute_value, max_absolute_value]
        # bisect_left finds the leftmost position where count >= k
        # Subtract max_absolute_value to convert from index to actual value
        return bisect_left(range(-max_absolute_value, max_absolute_value + 1), k,
                          key=count_products_less_than_or_equal) - max_absolute_value
```

## Time and Space Complexity

**Time Complexity:** `O(m × log n × log M)`

The time complexity breaks down as follows:
- The outer binary search using `bisect_left` on the range `[-mx, mx + 1]` performs `O(log M)` iterations, where `M = max(abs(nums1[0]), abs(nums1[-1])) × max(abs(nums2[0]), abs(nums2[-1]))` represents the maximum possible absolute value of products.
- For each binary search iteration, the `count` function is called, which iterates through all `m` elements in `nums1`, taking `O(m)` time.
- Within each iteration of `count`, either `bisect_right` or `bisect_left` is called on `nums2`, which takes `O(log n)` time where `n` is the length of `nums2`.
- Combining these operations: `O(log M) × O(m) × O(log n) = O(m × log n × log M)`

**Space Complexity:** `O(1)`

The space complexity analysis:
- The `count` function uses only a constant amount of extra space for variables `cnt`, `n`, and `x`.
- The `bisect_left` and `bisect_right` functions operate in-place and use `O(1)` extra space.
- The range object `range(-mx, mx + 1)` is a lazy iterator that doesn't create the entire list in memory, using `O(1)` space.
- No additional data structures are created that scale with input size.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Using the Wrong Binary Search Template Variant**

A common mistake is using `while left < right` with `right = mid` instead of the standard boundary-finding template. The correct template uses `while left <= right` with `right = mid - 1` and tracks the answer with `firstTrueIndex`.

**Incorrect Implementation:**
```java
while (left < right) {
    long mid = (left + right) >> 1;
    if (count(mid) >= k) {
        right = mid;  // Wrong pattern
    } else {
        left = mid + 1;
    }
}
return left;  // Returns left directly
```

**Correct Implementation:**
```java
long firstTrueValue = right;  // Default if not found
while (left <= right) {
    long mid = left + (right - left) / 2;
    if (count(mid) >= k) {
        firstTrueValue = mid;  // Track the answer
        right = mid - 1;
    } else {
        left = mid + 1;
    }
}
return firstTrueValue;
```

### 2. **Floating-Point Precision Issues**

The most critical pitfall in this solution is using floating-point division (`p / x`) when working with potentially large integers. This can lead to precision errors that cause incorrect counting.

**Problem Example:**
```python
# When p = 10^9 and x = 3
# p / x = 333333333.3333...
# Due to floating-point representation, this might become 333333333.33333334
# This can cause bisect_right/bisect_left to return wrong positions
```

**Solution:**
Replace floating-point division with integer arithmetic:

```python
def count_products_less_than_or_equal(target_product: int) -> int:
    total_count = 0
    nums2_length = len(nums2)

    for num1 in nums1:
        if num1 > 0:
            # Find how many nums2[j] satisfy num1 * nums2[j] <= target_product
            left, right = 0, nums2_length
            while left < right:
                mid = (left + right) // 2
                if num1 * nums2[mid] <= target_product:
                    left = mid + 1
                else:
                    right = mid
            total_count += left
        elif num1 < 0:
            # Find how many nums2[j] satisfy num1 * nums2[j] <= target_product
            left, right = 0, nums2_length
            while left < right:
                mid = (left + right) // 2
                if num1 * nums2[mid] <= target_product:
                    right = mid
                else:
                    left = mid + 1
            total_count += nums2_length - left
        else:
            total_count += nums2_length * int(target_product >= 0)

    return total_count
```

### 2. **Integer Overflow in Product Calculation**

When calculating `max_absolute_value` or products during comparison, the multiplication might overflow in languages with fixed integer sizes (though Python handles arbitrary precision integers).

**Problem Example:**
```python
# If nums1[-1] = 10^5 and nums2[-1] = 10^5
# Their product is 10^10, which exceeds 32-bit integer range
```

**Solution:**
For languages with overflow concerns, use long/int64 types or implement overflow-safe multiplication checks:

```python
def safe_multiply(a: int, b: int, limit: int) -> int:
    """Returns min(a * b, limit) without overflow"""
    if a == 0 or b == 0:
        return 0
    if abs(a) > limit // abs(b):
        return limit if (a > 0) == (b > 0) else -limit
    return a * b
```

### 3. **Incorrect Handling of Mixed Sign Arrays**

The initial calculation of `max_absolute_value` assumes the maximum absolute product comes from endpoint values, but with mixed signs, the actual range might be different.

**Problem Example:**
```python
# nums1 = [-10, -1, 1, 10]
# nums2 = [-10, -1, 1, 10]
# Maximum product: 10 * 10 = 100
# Minimum product: -10 * 10 = -100
# But if we only check endpoints, we might miss interior combinations
```

**Solution:**
Calculate the actual minimum and maximum possible products:

```python
# Calculate all possible extreme products
candidates = [
    nums1[0] * nums2[0],
    nums1[0] * nums2[-1],
    nums1[-1] * nums2[0],
    nums1[-1] * nums2[-1]
]
min_product = min(candidates)
max_product = max(candidates)

# Then use range [min_product, max_product] for binary search
return bisect_left(range(min_product, max_product + 1), k,
                  key=count_products_less_than_or_equal) - min_product
```

### 4. **Off-by-One Error in Binary Search Range**

Using `range(-mx, mx + 1)` assumes all products fall within `[-mx, mx]`, but the actual minimum product might not be `-mx`.

**Solution:**
Always calculate the exact minimum and maximum products rather than assuming symmetric ranges around zero.

