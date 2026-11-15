# 1231. Divide Chocolate

> Source: https://algo.monster/liteproblems/1231

## Problem Statement

You have a chocolate bar made up of chunks, where each chunk has a sweetness value given in an array `sweetness`.

You want to share this chocolate with `k` friends. To do this, you need to make `k` cuts to divide the chocolate into `k + 1` pieces. Each piece must consist of consecutive chunks (you can't rearrange the chunks).

After dividing the chocolate:
- You will eat the piece with the **minimum total sweetness**
- Your friends get the remaining pieces

Your goal is to maximize the total sweetness of the piece you eat by choosing where to make the cuts optimally.

For example, if `sweetness = [1, 2, 3, 4, 5, 6, 7, 8, 9]` and `k = 5`, you need to make 5 cuts to create 6 pieces. The algorithm will try to maximize the minimum piece's sweetness. One possible division could be `[1,2,3]`, `[4,5]`, `[6]`, `[7]`, `[8]`, `[9]` where the pieces have sweetness 6, 9, 6, 7, 8, and 9 respectively. You would eat the piece with minimum sweetness (6), so your goal is to find a cutting strategy that maximizes this minimum value.

The function should return the maximum possible total sweetness of the piece you can get.

## Intuition

The key insight is recognizing that this problem has a monotonic property: if we can achieve a minimum piece sweetness of `x`, then we can definitely achieve any minimum piece sweetness less than `x`. Why? Because if we can divide the chocolate such that every piece has at least sweetness `x`, we could make the same cuts (or adjust them slightly) to ensure every piece has at least sweetness `x-1`.

This monotonic property immediately suggests [binary search](/problems/binary_search_intro) on the answer. Instead of trying to figure out where to cut directly, we can ask: "What's the maximum sweetness `x` such that we can divide the chocolate into `k+1` pieces where each piece has at least sweetness `x`?"

The search space for our answer is between `0` (worst case, we get nothing) and `sum(sweetness)` (best case, we get the entire chocolate bar - only possible when `k=0`).

For any candidate value `mid` in our [binary search](/problems/binary_search_intro), we need to check: "Can we make `k` cuts such that we get `k+1` pieces, each with sweetness at least `mid`?"

To check this feasibility, we use a greedy approach: traverse the chocolate bar from left to right, accumulating sweetness. Whenever our accumulated sweetness reaches or exceeds `mid`, we "make a cut" (form a piece) and reset our accumulator. If we can form more than `k` pieces this way, it means we can definitely achieve a minimum sweetness of `mid` (we have extra pieces to spare). If we can't form enough pieces, then `mid` is too ambitious.

The greedy approach works because if there's any way to divide the chocolate to achieve minimum sweetness `mid`, then greedily taking pieces as soon as they reach `mid` will also work. There's no benefit to making pieces larger than necessary when we're trying to maximize the count of valid pieces.

**Learn more about [Binary Search](/problems/binary_search_intro) patterns.**

## Solution Approach

The solution uses the **[binary search](/problems/binary_search_intro) template** combined with a **greedy validation function**.

**Key Insight: Inverting the Feasible Function**
Since we're searching for the **maximum** achievable value, we invert the feasible function. Instead of "can we achieve sweetness X?", we define `feasible(x)` as "can we NOT achieve sweetness X?":

```python
def feasible(min_sweetness):
    current_sum = 0
    pieces_count = 0
    for chunk in sweetness:
        current_sum += chunk
        if current_sum >= min_sweetness:
            current_sum = 0
            pieces_count += 1
    # Cannot achieve if pieces_count <= k (need > k pieces)
    return pieces_count <= k
```

This creates a pattern like: `[false, false, false, true, true]` where `false` means "achievable" and `true` means "not achievable".

**Binary Search Using the Template:**
```python
left, right = 1, total + 1
first_true_index = total + 1  # Default if all values achievable

while left <= right:
    mid = (left + right) // 2
    if feasible(mid):
        first_true_index = mid
        right = mid - 1
    else:
        left = mid + 1

return first_true_index - 1
```

The template finds the **first infeasible sweetness**. The answer is `first_true_index - 1` (the last achievable sweetness).

**Why the Greedy Check Works:**
- We greedily form pieces from left to right
- Each piece is formed as soon as its total sweetness reaches `mid`
- If we can form more than `k` pieces (i.e., at least `k+1` pieces), then `mid` is achievable
- The greedy approach is optimal: if there's any way to achieve sweetness `mid`, greedy will find it

**Time Complexity:** `O(n × log(sum(sweetness)))` where `n` is the length of the sweetness array.

**Space Complexity:** `O(1)` as we only use a constant amount of extra space.

## Example Walkthrough

Let's walk through a small example with `sweetness = [1, 2, 2, 1, 2, 2, 1, 2, 2]` and `k = 2` (we need to make 2 cuts to create 3 pieces).

**Step 1: Initialize Binary Search Range**
- `l = 0` (minimum possible sweetness)
- `r = sum([1, 2, 2, 1, 2, 2, 1, 2, 2]) = 15` (maximum possible sweetness)

**Step 2: Binary Search Process**

**Iteration 1:**
- `mid = (0 + 15 + 1) // 2 = 8`
- Check if we can achieve minimum sweetness of 8:
  - Traverse array: `[1, 2, 2, 1, 2, 2, 1, 2, 2]`
  - Accumulate: 1 → 3 → 5 → 6 → 8 (≥8, form piece 1, reset)
  - Continue: 2 → 3 → 5 → 7 (can't reach 8, end of array)
  - Pieces formed: 1 (need 3)
  - Return `false` since 1 ≤ 2
- Result: `r = 8 - 1 = 7`

**Iteration 2:**
- `mid = (0 + 7 + 1) // 2 = 4`
- Check if we can achieve minimum sweetness of 4:
  - Traverse: `[1, 2, 2, 1, 2, 2, 1, 2, 2]`
  - Accumulate: 1 → 3 → 5 (≥4, form piece 1, reset)
  - Continue: 1 → 3 → 5 (≥4, form piece 2, reset)
  - Continue: 1 → 3 → 5 (≥4, form piece 3, reset)
  - Pieces formed: 3 (need 3)
  - Return `true` since 3 > 2
- Result: `l = 4`

**Iteration 3:**
- `mid = (4 + 7 + 1) // 2 = 6`
- Check if we can achieve minimum sweetness of 6:
  - Traverse: `[1, 2, 2, 1, 2, 2, 1, 2, 2]`
  - Accumulate: 1 → 3 → 5 → 6 (≥6, form piece 1, reset)
  - Continue: 2 → 4 → 5 → 7 (≥6, form piece 2, reset)
  - Continue: 2 (can't reach 6, end of array)
  - Pieces formed: 2 (need 3)
  - Return `false` since 2 ≤ 2
- Result: `r = 6 - 1 = 5`

**Iteration 4:**
- `mid = (4 + 5 + 1) // 2 = 5`
- Check if we can achieve minimum sweetness of 5:
  - Traverse: `[1, 2, 2, 1, 2, 2, 1, 2, 2]`
  - Accumulate: 1 → 3 → 5 (≥5, form piece 1, reset)
  - Continue: 1 → 3 → 5 (≥5, form piece 2, reset)
  - Continue: 1 → 3 → 5 (≥5, form piece 3, reset)
  - Pieces formed: 3 (need 3)
  - Return `true` since 3 > 2
- Result: `l = 5`

**Step 3: Convergence**
- Now `l = 5` and `r = 5`, so the loop exits
- Return `l = 5`

**Final Answer: 5**

The optimal division would be: `[1, 2, 2]` (sweetness = 5), `[1, 2, 2]` (sweetness = 5), `[1, 2, 2]` (sweetness = 5). You eat the piece with minimum sweetness, which is 5.

## Python Solution

```python
class Solution:
    def maximizeSweetness(self, sweetness: List[int], k: int) -> int:
        total = sum(sweetness)

        def feasible(min_sweetness: int) -> bool:
            """
            Returns True if we CANNOT achieve this minimum sweetness.
            (Inverted for standard binary search template)
            """
            current_sum = 0
            pieces_count = 0

            for chunk in sweetness:
                current_sum += chunk
                if current_sum >= min_sweetness:
                    current_sum = 0
                    pieces_count += 1

            # Can NOT achieve if pieces_count <= k (need > k pieces)
            return pieces_count <= k

        # Binary search for the first infeasible minimum sweetness
        # Pattern: [false, false, true, true] where false = achievable
        left, right = 1, total + 1
        first_true_index = total + 1  # Default if all values achievable

        while left <= right:
            mid = (left + right) // 2

            if feasible(mid):
                first_true_index = mid
                right = mid - 1
            else:
                left = mid + 1

        # Answer is the largest achievable = first_true_index - 1
        return first_true_index - 1
```

## Time and Space Complexity

**Time Complexity:** `O(n × log(sum(sweetness)))`

The algorithm uses binary search on the answer space. The search range is from `0` to `sum(sweetness)`, which means the binary search will perform `O(log(sum(sweetness)))` iterations. In each iteration, the `check` function is called, which iterates through all `n` elements of the sweetness array exactly once, taking `O(n)` time. Therefore, the overall time complexity is `O(n × log(sum(sweetness)))`, where `n` is the length of the sweetness array.

**Space Complexity:** `O(1)`

The algorithm only uses a constant amount of extra space. The variables `l`, `r`, `mid` in the main function and `s`, `cnt`, `v` in the check function are all scalar variables that don't depend on the input size. No additional data structures are created that scale with the input, so the space complexity is `O(1)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Using Wrong Binary Search Template Variant**
**Pitfall:** For "find maximum" problems, using the non-standard `while left < right` with `left = mid` variant requires careful midpoint calculation.

**Example of the Problem:**
```python
# Non-standard variant - prone to infinite loops
while left < right:
    mid = (left + right + 1) >> 1  # Must use +1 to avoid infinite loop
    if can_achieve(mid):
        left = mid
    else:
        right = mid - 1
```

**Solution:** Invert the feasible function and use the standard template:
```python
def feasible(x):
    # Returns True if we CANNOT achieve x
    return pieces_count <= k

first_true_index = total + 1
while left <= right:
    mid = (left + right) // 2
    if feasible(mid):
        first_true_index = mid
        right = mid - 1
    else:
        left = mid + 1
return first_true_index - 1
```

### 2. **Wrong Piece Count Comparison**
**Pitfall:** Using `pieces_count >= k` instead of `pieces_count > k` in the validation.

**Why it's problematic:** We need `k+1` pieces total. The inverted feasible returns true when we CANNOT achieve, so the condition should be `pieces_count <= k`.

**Solution:** Use `return pieces_count <= k` for the inverted feasible function.

### 3. **Edge Case: k = 0**
**Pitfall:** Not understanding that when `k = 0`, we need exactly 1 piece, so the answer is the sum of all sweetness.

**Solution:** The template handles this correctly - when `k = 0`, `feasible` returns false for all values up to the total sum, so `first_true_index = total + 1` and the answer is `total`.

### 4. **Integer Overflow in Large Inputs**
**Pitfall:** In languages with fixed integer sizes, `sum(sweetness)` might overflow.

**Solution:** In Python this isn't an issue. In other languages, use `long long` in C++ or `long` in Java.

### 5. **Misunderstanding the Greedy Strategy**
**Pitfall:** Trying to optimize piece formation with DP or looking ahead.

**Solution:** Trust the greedy approach - cutting as soon as we reach the threshold is optimal for checking feasibility.

