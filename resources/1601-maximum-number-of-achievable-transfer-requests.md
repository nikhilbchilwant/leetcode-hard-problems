# 1601. Maximum Number of Achievable Transfer Requests

> Source: https://algo.monster/liteproblems/1601

## Problem Statement

You have `n` buildings numbered from `0` to `n - 1`, and each building has employees who want to transfer to different buildings during transfer season.

You're given an array `requests` where each element `requests[i] = [from_i, to_i]` represents an employee's request to move from building `from_i` to building `to_i`.

The key constraint is that **all buildings are full**. This means a set of transfer requests can only be approved if every building maintains the same number of employees after all transfers are complete. In other words, for each building, the number of employees leaving must equal the number of employees entering (net change = 0).

For example, if `n = 3`:
- If 2 employees leave building 0, 1 leaves building 1, and 1 leaves building 2
- Then exactly 2 employees must enter building 0, 1 must enter building 1, and 1 must enter building 2

Your task is to find the **maximum number of requests** that can be fulfilled while maintaining this balance constraint for all buildings.

## Intuition

When we first look at this problem, we need to find which transfer requests we can approve while keeping all buildings balanced. The challenge is that approving one request affects two buildings - one loses an employee, another gains one.

The key insight comes from recognizing that we need to find a subset of requests where the movements form "cycles" or balanced patterns. For instance, if person A moves from building 0 to 1, person B moves from 1 to 2, and person C moves from 2 to 0, this forms a cycle where each building loses one and gains one employee.

Since we want the maximum number of requests, we might think about greedy approaches or dynamic programming. However, the constraint that each building must maintain exactly the same number of employees makes it difficult to make local optimal choices. Approving one request might require approving several others to maintain balance, and there's no clear way to predict which combination will be optimal.

The breakthrough comes when we notice the constraint: at most 16 requests. With such a small number, we can actually check every possible combination! If we have `m` requests, there are `2^m` ways to select or not select each request. For 16 requests, that's only 65,536 combinations - completely feasible for a computer to check.

We can represent each combination as a binary number where bit `i` indicates whether we include request `i`. For example, if we have 3 requests and the binary number is `101`, we include requests 0 and 2 but not request 1.

For each combination, we simulate the transfers: subtract 1 from the "from" building and add 1 to the "to" building for each selected request. If all buildings end up with a net change of 0, this combination is valid. Among all valid combinations, we pick the one with the most requests (most 1s in the binary representation).

This brute force approach works perfectly here because the small constraint size allows us to exhaustively search the entire solution space, guaranteeing we find the optimal answer.

**Learn more about [Backtracking](/problems/backtracking) patterns.**

## Solution Approach

The solution uses binary enumeration to explore all possible combinations of transfer requests. Here's how the implementation works:

**Binary Mask Representation:**
We use a bitmask to represent which requests are selected. For `m` requests, we iterate through all numbers from `0` to `2^m - 1`. Each number's binary representation tells us which requests to include - if bit `i` is 1, we include request `i`.

**Main Loop Structure:**
```python
for mask in range(1 << len(requests)):
```
This iterates through all possible combinations. The expression `1 << len(requests)` equals `2^len(requests)`.

**Counting Selected Requests:**
```python
cnt = mask.bit_count()
```
The `bit_count()` method returns the number of 1s in the binary representation of `mask`, which equals the number of selected requests.

**Optimization Check:**
```python
if ans < cnt and check(mask):
```
We only check validity if the current combination has more requests than our current best answer. This saves unnecessary computation.

**Validation Function:**
The `check(mask)` function verifies if a combination maintains balance:
```python
def check(mask: int) -> bool:
    cnt = [0] * n
    for i, (f, t) in enumerate(requests):
        if mask >> i & 1:
            cnt[f] -= 1
            cnt[t] += 1
    return all(v == 0 for v in cnt)
```

- Initialize a counter array `cnt` with zeros for each building
- For each request at index `i`, check if bit `i` is set in `mask` using `mask >> i & 1`
- If the request is selected, decrement the count for the "from" building and increment for the "to" building
- Finally, verify all buildings have a net change of 0 using `all(v == 0 for v in cnt)`

**Time Complexity:** `O(2^m * m)` where `m` is the number of requests. We check `2^m` combinations, and for each, we iterate through `m` requests.

**Space Complexity:** `O(n)` for the counter array to track building changes.

The algorithm guarantees finding the optimal solution by exhaustively checking all possibilities, which is feasible due to the small constraint on the number of requests (≤ 16).

## Example Walkthrough

Let's walk through a concrete example with `n = 3` buildings and `requests = [[0,1], [1,2], [0,2]]`.

We need to find the maximum number of these transfer requests we can approve while keeping all buildings balanced (same number of employees).

**Step 1: Understanding the Binary Enumeration**

With 3 requests, we have 2³ = 8 possible combinations to check:
- `000` (binary) = 0 (decimal): Select no requests
- `001` (binary) = 1 (decimal): Select only request[2] = [0,2]
- `010` (binary) = 2 (decimal): Select only request[1] = [1,2]
- `011` (binary) = 3 (decimal): Select request[1] and request[2]
- `100` (binary) = 4 (decimal): Select only request[0] = [0,1]
- `101` (binary) = 5 (decimal): Select request[0] and request[2]
- `110` (binary) = 6 (decimal): Select request[0] and request[1]
- `111` (binary) = 7 (decimal): Select all three requests

**Step 2: Checking Each Combination**

Let's trace through mask = 6 (binary: `110`), which selects requests [0,1] and [1,2]:

1. Initialize `cnt = [0, 0, 0]` for buildings 0, 1, and 2
2. Process request[0] = [0,1] (bit 0 is set in mask 110):
   - `cnt[0] -= 1` → cnt = [-1, 0, 0]
   - `cnt[1] += 1` → cnt = [-1, 1, 0]
3. Process request[1] = [1,2] (bit 1 is set in mask 110):
   - `cnt[1] -= 1` → cnt = [-1, 0, 0]
   - `cnt[2] += 1` → cnt = [-1, 0, 1]
4. Skip request[2] (bit 2 is not set)
5. Check if all values in cnt equal 0: No! Building 0 has -1, building 2 has +1

This combination is **invalid**.

**Step 3: Finding a Valid Combination**

Let's check mask = 7 (binary: `111`), which selects all three requests:

1. Initialize `cnt = [0, 0, 0]`
2. Process request[0] = [0,1]:
   - `cnt[0] -= 1` → cnt = [-1, 0, 0]
   - `cnt[1] += 1` → cnt = [-1, 1, 0]
3. Process request[1] = [1,2]:
   - `cnt[1] -= 1` → cnt = [-1, 0, 0]
   - `cnt[2] += 1` → cnt = [-1, 0, 1]
4. Process request[2] = [0,2]:
   - `cnt[0] -= 1` → cnt = [-2, 0, 1]
   - `cnt[2] += 1` → cnt = [-2, 0, 2]
5. Check if all values equal 0: No! Building 0 has -2, building 2 has +2

This is also **invalid**.

**Step 4: Finding the Maximum Valid Combination**

After checking all combinations, we find that mask = 0 (no requests) is trivially valid with 0 requests. 

For this particular example, no other combination maintains balance. If we had requests like [[0,1], [1,2], [2,0]], then selecting all three would form a cycle where each building loses one and gains one employee, resulting in a valid combination with 3 requests.

The algorithm returns the maximum number of requests from all valid combinations found.

## Python Solution

```python
class Solution:
    def maximumRequests(self, n: int, requests: List[List[int]]) -> int:
        """
        Find the maximum number of achievable requests among all requests.
        A set of requests is achievable if all buildings have net zero change in employees.
      
        Args:
            n: Number of buildings (indexed from 0 to n-1)
            requests: List of [from_building, to_building] transfer requests
          
        Returns:
            Maximum number of requests that can be fulfilled while maintaining balance
        """
      
        def is_valid_combination(bitmask: int) -> bool:
            """
            Check if the selected requests result in balanced employee counts.
          
            Args:
                bitmask: Binary representation where bit i indicates if request i is selected
              
            Returns:
                True if all buildings have net zero change, False otherwise
            """
            # Track net change in employee count for each building
            building_balance = [0] * n
          
            # Process each request based on the bitmask
            for request_idx, (from_building, to_building) in enumerate(requests):
                # Check if current request is selected (bit is set)
                if bitmask >> request_idx & 1:
                    building_balance[from_building] -= 1  # Employee leaves
                    building_balance[to_building] += 1     # Employee arrives
          
            # Check if all buildings have balanced employee count (net zero change)
            return all(balance == 0 for balance in building_balance)
      
        max_achievable_requests = 0
      
        # Try all possible combinations of requests using bitmask
        # Total combinations = 2^(number of requests)
        for bitmask in range(1 << len(requests)):
            # Count number of selected requests in current combination
            selected_count = bitmask.bit_count()
          
            # Only check validity if this combination has more requests than current best
            if max_achievable_requests < selected_count and is_valid_combination(bitmask):
                max_achievable_requests = selected_count
      
        return max_achievable_requests
```

## Time and Space Complexity

**Time Complexity:** `O(2^m × (m + n))`

The algorithm uses bit manipulation to enumerate all possible subsets of requests. The outer loop iterates through all `2^m` possible masks (where `m` is the length of requests). For each mask:
- The `bit_count()` operation takes `O(1)` time in Python 3.10+
- The `check()` function is called, which:
  - Iterates through all `m` requests to update the count array: `O(m)`
  - Checks if all `n` values in the count array equal zero: `O(n)`

Therefore, the total time complexity is `O(2^m × (m + n))`.

**Space Complexity:** `O(n)`

The space usage comes from:
- The `cnt` array inside the `check()` function, which has size `n`: `O(n)`
- A few integer variables (`ans`, `mask`, `cnt`) which take constant space: `O(1)`

The space complexity is `O(n)` as the dominant factor is the count array used to track the net change in occupancy for each room.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Misunderstanding the Balance Constraint**
Many developers initially think that simply matching the total number of outgoing and incoming transfers is sufficient. However, the constraint requires **each individual building** to maintain its exact employee count.

**Incorrect approach:**
```python
# Wrong: Only checking total balance
total_in = sum(1 for _, to_building in selected_requests)
total_out = sum(1 for from_building, _ in selected_requests)
return total_in == total_out  # This is NOT sufficient!
```

**Correct approach:**
```python
# Right: Check balance for EACH building independently
building_balance = [0] * n
for from_building, to_building in selected_requests:
    building_balance[from_building] -= 1
    building_balance[to_building] += 1
return all(balance == 0 for balance in building_balance)
```

### 2. **Incorrect Bit Manipulation**
A common mistake is incorrectly checking if a bit is set in the mask, leading to wrong request selections.

**Incorrect approaches:**
```python
# Wrong: Using wrong bit shift direction
if mask << i & 1:  # Should be >> not <<

# Wrong: Forgetting the & 1 operation
if mask >> i:  # This checks if mask >> i is non-zero, not if bit i is set

# Wrong: Using wrong bit index
if mask & (1 << (len(requests) - i - 1)):  # Unnecessarily reversing bit order
```

**Correct approach:**
```python
# Right: Shift right by i positions and check the least significant bit
if mask >> i & 1:
    # Process request i
```

### 3. **Off-by-One Error in Enumeration Range**
Developers might incorrectly calculate the total number of combinations.

**Incorrect approach:**
```python
# Wrong: Missing the last combination
for mask in range((1 << len(requests)) - 1):  # Misses the all-1s combination

# Wrong: Including unnecessary iteration
for mask in range(1 << len(requests) + 1):  # One iteration too many
```

**Correct approach:**
```python
# Right: Range from 0 to 2^m - 1 (inclusive)
for mask in range(1 << len(requests)):
    # This covers all 2^m combinations
```

### 4. **Inefficient Order of Operations**
Checking validity before comparing with the current best answer wastes computation.

**Inefficient approach:**
```python
for mask in range(1 << len(requests)):
    if is_valid_combination(mask):  # Expensive check done first
        selected_count = mask.bit_count()
        if selected_count > max_achievable_requests:
            max_achievable_requests = selected_count
```

**Efficient approach:**
```python
for mask in range(1 << len(requests)):
    selected_count = mask.bit_count()  # Cheap operation first
    if selected_count > max_achievable_requests:  # Quick filter
        if is_valid_combination(mask):  # Expensive check only when needed
            max_achievable_requests = selected_count
```

### 5. **Handling Self-Transfers Incorrectly**
When an employee requests to transfer from building A to building A (same building), this still counts as a valid request that can be fulfilled without affecting the balance.

**Potential confusion:**
```python
# Some might think self-transfers need special handling
if from_building == to_building:
    # Special case? No! The balance logic handles this automatically
    # building_balance[from_building] -= 1
    # building_balance[from_building] += 1
    # Net effect is 0, which is correct
```

**Correct understanding:**
The standard balance tracking automatically handles self-transfers correctly since subtracting 1 and adding 1 to the same building results in no net change.

