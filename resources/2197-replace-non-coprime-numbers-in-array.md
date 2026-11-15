# 2197. Replace Non-Coprime Numbers in Array

> Source: https://algo.monster/liteproblems/2197

## Problem Statement

You are given an array of integers `nums`. Your task is to repeatedly merge adjacent numbers that share a common factor greater than 1 (non-coprime numbers) until no such pairs exist.

The process works as follows:

1. **Find any two adjacent numbers** in the array where `GCD(x, y) > 1` (they are non-coprime)
2. **If no such pair exists**, stop the process
3. **Otherwise, replace the pair** with their Least Common Multiple (LCM)
4. **Repeat** steps 1-3 until no more non-coprime adjacent pairs can be found

Two numbers are considered **non-coprime** if their Greatest Common Divisor (GCD) is greater than 1. For example:
- Numbers 6 and 9 are non-coprime because `GCD(6, 9) = 3 > 1`
- Numbers 5 and 7 are coprime because `GCD(5, 7) = 1`

When you find a non-coprime pair, you calculate their LCM using the formula: `LCM(x, y) = x * y / GCD(x, y)`

The problem guarantees that:
- The order in which you merge non-coprime pairs doesn't matter - you'll always get the same final result
- All values in the final array will be at most `10^8`

Return the final modified array after all possible merges have been performed.

**Example walkthrough:**
If `nums = [6, 4, 3, 2, 7, 6, 2]`:
- 6 and 4 are non-coprime (`GCD = 2`), merge to get `[12, 3, 2, 7, 6, 2]`
- 12 and 3 are non-coprime (`GCD = 3`), merge to get `[12, 2, 7, 6, 2]`
- 12 and 2 are non-coprime (`GCD = 2`), merge to get `[12, 7, 6, 2]`
- 6 and 2 are non-coprime (`GCD = 2`), merge to get `[12, 7, 6]`
- No more non-coprime adjacent pairs exist
- Final result: `[12, 7, 6]`

## Intuition

The key insight is that when we have three consecutive numbers that could potentially be merged, the order of merging doesn't matter - we always end up with the same result. Consider three numbers `x`, `y`, `z` where both pairs `(x,y)` and `(y,z)` are non-coprime. Whether we merge `x` and `y` first (getting `LCM(x,y)`) and then merge with `z`, or merge `y` and `z` first and then merge with `x`, we get the same final value: `LCM(x, y, z)`.

This property suggests we can use a **greedy approach** - we can always choose to merge from left to right without worrying about missing a better merging strategy.

Think of it like building a wall where some bricks need to be combined when they don't fit well together. As we place each new brick, we check if it fits well with the previous one. If not, we combine them into a single larger brick. Then we check if this new combined brick fits with the one before it, and so on.

A **[stack](/problems/stack_intro)** is perfect for this scenario because:
1. We need to constantly check and potentially modify the most recent elements (the top of the stack)
2. After merging two numbers, the result might need to be merged with the number that came before them
3. We only ever need to look at the last two elements at any given time

The process becomes:
- Add each number to the [stack](/problems/stack_intro)
- After adding, repeatedly check if the top two numbers are non-coprime
- If they are non-coprime, replace them with their LCM
- This new LCM might be non-coprime with the number below it, so we keep checking
- Continue until the top two numbers are coprime or we have less than two numbers

This way, we maintain an invariant: at any point, all numbers currently in the [stack](/problems/stack_intro) (except possibly the last two) are pairwise coprime with their neighbors. When we finish processing all numbers, the entire stack contains the final answer where no adjacent pairs can be merged.

**Learn more about [Stack](/problems/stack_intro) and [Math](/problems/math-basics) patterns.**

## Solution Approach

We implement the solution using a **[stack](/problems/stack_intro)-based approach** to efficiently manage the merging process.

**Algorithm Steps:**

1. **Initialize an empty [stack](/problems/stack_intro)** to store the processed numbers.

2. **Process each number** in the input array:
   - Push the current number `x` onto the [stack](/problems/stack_intro)
   - After pushing, enter a loop to check for potential merges

3. **Merge adjacent non-coprime numbers**:
   - While the [stack](/problems/stack_intro) has at least 2 elements:
     - Get the top two elements: `x` (second from top) and `y` (top)
     - Calculate their GCD: `g = gcd(x, y)`
     - If `g == 1`, they are coprime, so break the loop
     - Otherwise, merge them:
       - Pop the top element
       - Replace the new top with their LCM: `x * y // g`
     - Continue checking (the new LCM might be non-coprime with the element below it)

4. **Return the [stack](/problems/stack_intro)** as the final result.

**Implementation Details:**

```python
def replaceNonCoprimes(self, nums: List[int]) -> List[int]:
    stk = []
    for x in nums:
        stk.append(x)
        while len(stk) > 1:
            x, y = stk[-2:]  # Get last two elements
            g = gcd(x, y)
            if g == 1:  # Coprime, no merge needed
                break
            stk.pop()  # Remove the top element
            stk[-1] = x * y // g  # Replace with LCM
    return stk
```

**Key Points:**

- The **LCM formula** used is: `LCM(x, y) = x * y / GCD(x, y)`
- We use integer division `//` to ensure the result remains an integer
- The `gcd` function (from Python's [math](/problems/math-basics) module) efficiently computes the Greatest Common Divisor
- The inner `while` loop ensures we cascade merges - after merging two numbers, we immediately check if the result can merge with its new neighbor

**Time Complexity:** O(n × log(max_value)), where n is the length of the input array. Each number is pushed once and can be involved in at most one merge operation. The GCD calculation takes O(log(max_value)) time.

**Space Complexity:** O(n) for the [stack](/problems/stack_intro) in the worst case when no numbers can be merged.

The beauty of this approach is that it maintains the invariant that all adjacent pairs in the [stack](/problems/stack_intro) (except possibly the last two being processed) are already coprime, ensuring we find the optimal final configuration.

## Example Walkthrough

Let's trace through a small example: `nums = [2, 6, 3, 9, 2]`

**Initial state:** Stack = `[]`

**Step 1: Process 2**
- Push 2: Stack = `[2]`
- Check merges: Only 1 element, no merge possible

**Step 2: Process 6**
- Push 6: Stack = `[2, 6]`
- Check top two: GCD(2, 6) = 2 > 1 (non-coprime!)
- Merge: LCM(2, 6) = 2 × 6 ÷ 2 = 6
- Stack = `[6]`
- Check again: Only 1 element, done

**Step 3: Process 3**
- Push 3: Stack = `[6, 3]`
- Check top two: GCD(6, 3) = 3 > 1 (non-coprime!)
- Merge: LCM(6, 3) = 6 × 3 ÷ 3 = 6
- Stack = `[6]`
- Check again: Only 1 element, done

**Step 4: Process 9**
- Push 9: Stack = `[6, 9]`
- Check top two: GCD(6, 9) = 3 > 1 (non-coprime!)
- Merge: LCM(6, 9) = 6 × 9 ÷ 3 = 18
- Stack = `[18]`
- Check again: Only 1 element, done

**Step 5: Process 2**
- Push 2: Stack = `[18, 2]`
- Check top two: GCD(18, 2) = 2 > 1 (non-coprime!)
- Merge: LCM(18, 2) = 18 × 2 ÷ 2 = 18
- Stack = `[18]`
- Check again: Only 1 element, done

**Final Result:** `[18]`

Notice how the stack approach naturally handles cascading merges - each time we merge, we immediately check if the result can merge with what came before it. In this example, all numbers ended up sharing common factors, resulting in a single merged value.

## Python Solution

```python
from typing import List
from math import gcd

class Solution:
    def replaceNonCoprimes(self, nums: List[int]) -> List[int]:
        # Use a stack to process elements and merge non-coprime adjacent pairs
        stack = []
      
        for num in nums:
            # Add current number to stack
            stack.append(num)
          
            # Keep merging the last two elements if they are not coprime
            while len(stack) > 1:
                # Get the last two elements
                second_last, last = stack[-2:]
              
                # Calculate the greatest common divisor
                common_divisor = gcd(second_last, last)
              
                # If GCD is 1, they are coprime, no merge needed
                if common_divisor == 1:
                    break
              
                # Remove the last element
                stack.pop()
              
                # Replace the second-to-last element with LCM of the pair
                # LCM = (a * b) / GCD(a, b)
                stack[-1] = second_last * last // common_divisor
      
        return stack
```

## Time and Space Complexity

**Time Complexity:** `O(n × log M)`

The algorithm iterates through each element in the input array once (`O(n)`). For each element, it performs a while loop that merges adjacent non-coprime numbers. The key operation inside the while loop is computing the GCD using the Euclidean algorithm, which takes `O(log M)` time where `M` is the maximum value being compared.

While the inner while loop might seem to suggest a higher complexity, each element can be involved in at most `O(1)` amortized merge operations. This is because once two elements are merged, they form a larger element that won't be revisited for the same comparison. The total number of GCD computations across all iterations is bounded by `O(n)`, and each GCD computation takes `O(log M)` time.

Therefore, the overall time complexity is `O(n × log M)`.

**Space Complexity:** `O(n)`

The algorithm uses a stack `stk` to store the processed elements. In the worst case, when all consecutive elements are coprime (GCD = 1), no merging occurs, and the stack will contain all `n` elements from the input array. Thus, the space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Not Continuously Checking for Cascading Merges**

**The Pitfall:** A common mistake is to only check once if the newly added element can merge with its predecessor, without considering that the merged result might also be non-coprime with its new neighbor.

**Incorrect Implementation:**
```python
def replaceNonCoprimes(self, nums: List[int]) -> List[int]:
    stack = []
    for num in nums:
        if stack and gcd(stack[-1], num) > 1:
            g = gcd(stack[-1], num)
            stack[-1] = stack[-1] * num // g
        else:
            stack.append(num)
    return stack
```

**Why it fails:** Consider `nums = [2, 2, 2]`:
- First 2 goes into stack: `[2]`
- Second 2 merges with first: `[4]` (LCM of 2 and 2)
- Third 2 is added: `[4, 2]` ❌ (Should continue merging!)

The correct result should be `[8]`, not `[4, 2]`.

**Solution:** Use a `while` loop to continuously check and merge until no more merges are possible:
```python
while len(stack) > 1:
    x, y = stack[-2:]
    g = gcd(x, y)
    if g == 1:
        break
    stack.pop()
    stack[-1] = x * y // g
```

### 2. **Integer Overflow When Computing LCM**

**The Pitfall:** Computing `x * y` before dividing by GCD can cause integer overflow for large numbers, even though the final LCM might be within bounds.

**Problem Example:** If `x = 10^6` and `y = 10^6` with `gcd(x, y) = 10^3`, then:
- `x * y = 10^12` (might overflow in some languages)
- But `LCM = 10^12 / 10^3 = 10^9` (within bounds)

**Solution:** In Python, this isn't an issue due to arbitrary precision integers. However, in languages with fixed-size integers (like C++ or Java), you should:
1. First divide one number by the GCD, then multiply: `(x // g) * y`
2. Or use careful ordering: `x // g * y` (relying on left-to-right evaluation)

```python
# Safe approach for languages with overflow concerns:
stack[-1] = (second_last // common_divisor) * last
```

### 3. **Forgetting to Import GCD Function**

**The Pitfall:** Attempting to use `gcd()` without importing it from the `math` module.

**Solution:** Always include the import statement:
```python
from math import gcd
```

Or implement your own GCD function using Euclidean algorithm:
```python
def gcd(a, b):
    while b:
        a, b = b, a % b
    return a
```

