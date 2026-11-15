# 1675. Minimize Deviation in Array

> Source: https://algo.monster/liteproblems/1675

## Problem Statement

You have an array `nums` containing `n` positive integers. You can perform two types of operations on any element any number of times:

1. **If the element is even**: Divide it by 2
   - Example: Array `[1,2,3,4]` → perform on last element → `[1,2,3,2]`

2. **If the element is odd**: Multiply it by 2  
   - Example: Array `[1,2,3,4]` → perform on first element → `[2,2,3,4]`

The **deviation** of the array is defined as the maximum difference between any two elements in the array. In other words, `deviation = max(nums) - min(nums)`.

Your task is to find the **minimum possible deviation** after performing any number of operations on the array elements.

For example:
- If `nums = [1,2,3,4]`, the initial deviation is `4 - 1 = 3`
- After operations, you might achieve a smaller deviation by strategically increasing small odd numbers or decreasing large even numbers

The key insight is that:
- Odd numbers can only be multiplied by 2 (making them larger)
- Even numbers can only be divided by 2 (making them smaller)
- Each odd number can be multiplied at most once (since it becomes even)
- Each even number can be divided multiple times until it becomes odd

The challenge is determining which operations to perform and in what order to minimize the final deviation.

## Intuition

To minimize the deviation (the difference between max and min values), we need to bring the maximum and minimum values as close together as possible. We can either decrease the maximum value or increase the minimum value.

Let's first observe the constraints of our operations:
- Odd numbers can only go up (multiply by 2), and only once (then they become even)
- Even numbers can only go down (divide by 2), potentially multiple times

This asymmetry makes the problem tricky. Here's the key insight: since odd numbers can only be doubled once, let's **pre-process all odd numbers by doubling them first**. This transforms our problem into a simpler one where we only need to consider division operations.

After this pre-processing:
- All numbers in our array are now even
- We can only perform division operations (making numbers smaller)
- The only way to reduce deviation is to reduce the maximum value (since we can't increase the minimum anymore)

Now the strategy becomes clear:
1. Start with all odd numbers doubled (so everything is even)
2. Keep track of the current minimum value
3. Repeatedly take the maximum value and divide it by 2
4. Update our minimum value if the new divided value is smaller
5. Calculate the deviation at each step
6. Stop when the maximum value becomes odd (can't divide anymore)

We use a max heap to efficiently track and extract the maximum value at each step. The reason we stop when we hit an odd maximum is that we can't reduce it further, and all other values are either already smaller or would need to pass through this odd value if we kept dividing them.

This [greedy](/problems/greedy_intro) approach works because we're always reducing the largest possible value, which gives us the best chance of minimizing the overall deviation.

**Learn more about [Greedy](/problems/greedy_intro) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

Let's walk through the implementation step by step:

**1. Pre-processing: Convert all odd numbers to even**

First, we iterate through the array and double all odd numbers. This is done using the bit operation `v & 1` to check if a number is odd, and `v <<= 1` (left shift) to multiply by 2:

```python
for v in nums:
    if v & 1:  # Check if odd
        v <<= 1  # Multiply by 2
```

**2. Initialize data structures**

We use a max heap to maintain the maximum value. Since Python's `heapq` is a min heap, we store negative values to simulate a max heap:

```python
h = []  # Max heap (storing negative values)
mi = inf  # Track minimum value

for v in nums:
    if v & 1:
        v <<= 1
    h.append(-v)  # Store negative for max heap
    mi = min(mi, v)  # Update minimum
heapify(h)
```

**3. Calculate initial deviation**

After pre-processing, calculate the initial deviation:

```python
ans = -h[0] - mi  # max_value - min_value
```

**4. [Greedy](/problems/greedy_intro) reduction of maximum values**

Keep extracting the maximum value (top of heap), divide it by 2, and push it back:

```python
while h[0] % 2 == 0:  # While max is even
    x = heappop(h) // 2  # Extract and divide by 2
    heappush(h, x)       # Push back the new value
    mi = min(mi, -x)     # Update minimum if needed
    ans = min(ans, -h[0] - mi)  # Update answer
```

**Key observations:**
- We only continue while the maximum value is even (`h[0] % 2 == 0`)
- When dividing a negative number by 2, we're effectively halving the absolute value
- After each operation, we update both the minimum value and the minimum deviation

**5. Why this works**

The algorithm guarantees finding the minimum deviation because:
- We've explored all possible ways to increase odd numbers (by doubling them initially)
- We systematically reduce the maximum value whenever possible
- We stop when no further reduction of the maximum is possible (it's odd)
- At each step, we track the minimum deviation achieved

The time complexity is `O(n log n log M)` where `M` is the maximum value, as each number can be divided at most `log M` times, and each heap operation takes `O(log n)`.

## Example Walkthrough

Let's trace through the algorithm with `nums = [4, 1, 5, 20, 3]`:

**Step 1: Pre-processing (double all odd numbers)**
- Original: `[4, 1, 5, 20, 3]`
- After doubling odds: `[4, 2, 10, 20, 6]`
- All numbers are now even
- Track minimum: `mi = 2`

**Step 2: Initialize max heap**
- Heap (negative values): `[-20, -10, -6, -4, -2]`
- Initial deviation: `20 - 2 = 18`

**Step 3: Iteratively reduce maximum values**

*Iteration 1:*
- Extract max: `20` (even, can divide)
- Divide by 2: `20 → 10`
- New heap: `[-10, -10, -6, -4, -2]`
- Update minimum: `mi = 2` (unchanged)
- New deviation: `10 - 2 = 8`
- Best deviation so far: `8`

*Iteration 2:*
- Extract max: `10` (even, can divide)
- Divide by 2: `10 → 5`
- New heap: `[-10, -6, -5, -4, -2]`
- Update minimum: `mi = 2` (unchanged)
- New deviation: `10 - 2 = 8`
- Best deviation so far: `8`

*Iteration 3:*
- Extract max: `10` (even, can divide)
- Divide by 2: `10 → 5`
- New heap: `[-6, -5, -5, -4, -2]`
- Update minimum: `mi = 2` (unchanged)
- New deviation: `6 - 2 = 4`
- Best deviation so far: `4`

*Iteration 4:*
- Extract max: `6` (even, can divide)
- Divide by 2: `6 → 3`
- New heap: `[-5, -5, -4, -3, -2]`
- Update minimum: `mi = 2` (unchanged)
- New deviation: `5 - 2 = 3`
- Best deviation so far: `3`

*Iteration 5:*
- Extract max: `5` (odd, cannot divide)
- **STOP** - Maximum is odd, no more operations possible

**Final Result: Minimum deviation = 3**

The final array would be `[4, 2, 5, 5, 3]` with max = 5 and min = 2, giving deviation = 3.

Notice how we:
1. Started by making all numbers even (so we could only decrease from there)
2. Greedily reduced the largest values
3. Tracked the minimum deviation at each step
4. Stopped when the maximum became odd (no further reduction possible)

## Python Solution

```python
class Solution:
    def minimumDeviation(self, nums: List[int]) -> int:
        # Use a max heap to track the largest element
        # Python only has min heap, so we negate values
        max_heap = []
        minimum_value = float('inf')
      
        # Preprocess: maximize all odd numbers (they can only be doubled once)
        # This ensures we only need to handle division operations later
        for value in nums:
            if value & 1:  # Check if odd using bitwise AND
                value <<= 1  # Double the odd number (multiply by 2)
            max_heap.append(-value)  # Negate for max heap behavior
            minimum_value = min(minimum_value, value)
      
        # Convert list to heap structure
        heapify(max_heap)
      
        # Initial deviation: difference between max and min
        min_deviation = -max_heap[0] - minimum_value
      
        # Keep reducing the maximum value while it's even
        # We can only divide even numbers by 2
        while max_heap[0] % 2 == 0:
            # Pop the maximum (most negative) value and divide by 2
            max_value = heappop(max_heap) // 2
            heappush(max_heap, max_value)
          
            # Update the minimum value if necessary
            minimum_value = min(minimum_value, -max_value)
          
            # Update the minimum deviation found so far
            min_deviation = min(min_deviation, -max_heap[0] - minimum_value)
      
        return min_deviation
```

## Time and Space Complexity

**Time Complexity:** `O(n log n × log m)`, where `n` is the length of the array `nums` and `m` is the maximum element in the array.

- Initial preprocessing loop: `O(n)` - iterates through all elements once, performing constant time operations (bit manipulation and comparison)
- Heapify operation: `O(n)` - builds the heap from the array
- Main while loop: 
  - Each even number can be divided by 2 at most `O(log m)` times before becoming odd
  - Total number of heap operations across all elements: `O(n log m)`
  - Each heap operation (pop and push): `O(log n)`
  - Combined: `O(n log m × log n)` = `O(n log n × log m)`

**Space Complexity:** `O(n)`

- The heap `h` stores exactly `n` elements (one for each number in the input array)
- Additional variables (`mi`, `ans`, `x`, `v`) use `O(1)` space
- Total space: `O(n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Integer Division with Negative Numbers**

**The Pitfall:** When working with negative numbers in the max heap, dividing by 2 using `//` operator can produce unexpected results due to Python's floor division behavior.

```python
# Problematic code:
max_value = heappop(max_heap) // 2  # Wrong for negative numbers!
```

For negative numbers, Python's `//` operator rounds toward negative infinity:
- `-5 // 2 = -3` (not `-2` as you might expect)
- `-4 // 2 = -2` (this one works correctly)

**The Solution:** Since we're storing negative values to simulate a max heap, we need to handle the division carefully:

```python
# Correct approach:
max_value = -heappop(max_heap)  # Convert to positive first
max_value //= 2                  # Now divide
heappush(max_heap, -max_value)   # Push back as negative
```

Or use a more explicit conversion:
```python
max_value = heappop(max_heap)
max_value = -((-max_value) // 2)  # Convert to positive, divide, then negate
heappush(max_heap, max_value)
```

### 2. **Forgetting to Update Minimum Value During Operations**

**The Pitfall:** Only tracking the initial minimum value and forgetting that dividing large even numbers might create new minimum values.

```python
# Problematic code:
minimum_value = min(nums)  # Set once and never update
```

**The Solution:** Update the minimum value every time we perform a division operation:

```python
while max_heap[0] % 2 == 0:
    max_value = -heappop(max_heap)
    max_value //= 2
    heappush(max_heap, -max_value)
    minimum_value = min(minimum_value, max_value)  # Must update here!
    min_deviation = min(min_deviation, -max_heap[0] - minimum_value)
```

### 3. **Not Pre-processing Odd Numbers**

**The Pitfall:** Trying to handle both multiplication and division operations during the main loop, which significantly complicates the logic and can lead to suboptimal solutions.

```python
# Problematic approach:
while True:
    if min_value % 2 == 1:  # Try to increase minimum
        min_value *= 2
    if max_value % 2 == 0:  # Try to decrease maximum
        max_value //= 2
    # Complex logic to determine when to stop...
```

**The Solution:** Always double all odd numbers first, then only handle division operations:

```python
# Preprocess all odd numbers
for i in range(len(nums)):
    if nums[i] & 1:
        nums[i] <<= 1
# Now only handle divisions in the main loop
```

### 4. **Incorrect Heap Initialization**

**The Pitfall:** Forgetting to call `heapify()` after building the initial heap array, or trying to use `heappush()` for each element which is less efficient.

```python
# Inefficient:
max_heap = []
for value in processed_nums:
    heappush(max_heap, -value)  # O(n log n) total
```

**The Solution:** Build the array first, then heapify in O(n) time:

```python
max_heap = [-value for value in processed_nums]
heapify(max_heap)  # O(n) time
```

