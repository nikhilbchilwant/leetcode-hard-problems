# 810. Chalkboard XOR Game

> Source: https://algo.monster/liteproblems/810

## Problem Statement

You have an array of integers `nums` representing numbers written on a chalkboard. Alice and Bob play a game where they take turns erasing exactly one number from the chalkboard, with Alice going first.

The key rules are:
- If erasing a number causes the bitwise XOR of all remaining elements to become `0`, the player who erased that number loses
- If a player starts their turn with the bitwise XOR of all elements already equal to `0`, that player wins immediately
- The XOR of a single element is the element itself, and the XOR of no elements is `0`

Your task is to determine if Alice wins the game, assuming both players play optimally. Return `true` if Alice wins, `false` otherwise.

The solution reveals an elegant pattern: Alice wins if and only if either:
1. The initial XOR of all numbers is already `0` (Alice wins immediately), OR
2. The array has an even number of elements

The mathematical proof shows that when the array length is even and initial XOR is non-zero, Alice can always make a move that avoids losing. This is because if every possible move Alice could make resulted in XOR becoming `0` (causing her to lose), it would create a mathematical contradiction. Specifically, if we XOR all possible resulting states together, we'd get `0 = S` (where `S` is the initial non-zero XOR), which is impossible.

Conversely, when the array has odd length and non-zero initial XOR, Alice's first move leaves Bob with an even-length array, guaranteeing Bob's victory by the same logic.

## Intuition

When first approaching this problem, we might think about simulating the game or using dynamic programming to track all possible game states. However, the key insight comes from recognizing this is a mathematical game theory problem where we can determine the winner without simulating actual gameplay.

Let's think about the winning and losing conditions:
- A player loses if they erase a number that makes XOR = `0`
- A player wins if it's their turn and XOR is already `0`

The first observation is straightforward: if the initial XOR of all numbers is `0`, Alice wins immediately before making any move.

For the more interesting case where initial XOR ≠ `0`, we need to think about what happens based on array length. The breakthrough comes from considering what it means for a player to be "forced to lose."

For Alice to be forced to lose with an even-length array, every single number she could potentially erase must result in XOR = `0`. But here's the clever part: if we XOR all these potential resulting states together, we get a contradiction. 

Why? Because when we XOR all possible states `S_i` (where `S_i` is the XOR after removing element `i`), each `S_i = S ⊕ nums[i]`. When we XOR all these together with an even number of terms, the `S` values cancel out (even number of identical values XORed equals `0`), leaving us with just the XOR of all original numbers, which equals `S`. This would mean `0 = S`, contradicting our assumption that `S ≠ 0`.

This mathematical impossibility tells us that with an even-length array, Alice can always find at least one safe move that doesn't result in XOR = `0`. She can keep making safe moves, and eventually, Bob will be forced into a position where he must lose.

With an odd-length array, after Alice's first move, Bob gets an even-length array - putting him in the winning position we just analyzed. This pattern recognition leads us to the surprisingly simple solution: check if the array length is even or if the initial XOR is `0`.

**Learn more about [Math](/problems/math-basics) patterns.**

## Solution Approach

The implementation is remarkably elegant once we understand the mathematical pattern. We need to check two conditions for Alice to win:

1. **Initial XOR equals 0**: If the XOR of all numbers is already `0` when the game starts, Alice wins immediately without making any move.

2. **Even array length**: If the array has an even number of elements, Alice is guaranteed to win regardless of the initial XOR value (as proven mathematically).

The solution uses Python's `reduce` function with the `xor` operator from the `operator` module to calculate the XOR of all elements efficiently:

```python
class Solution:
    def xorGame(self, nums: List[int]) -> bool:
        return len(nums) % 2 == 0 or reduce(xor, nums) == 0
```

**Implementation Details:**

- `len(nums) % 2 == 0`: Checks if the array length is even. If true, Alice wins.

- `reduce(xor, nums)`: Computes the XOR of all elements in the array. The `reduce` function applies the XOR operation cumulatively from left to right:
  - For `nums = [a, b, c, d]`, it computes `((a ⊕ b) ⊕ c) ⊕ d`

- The `or` operator ensures Alice wins if either condition is met.

**Time and Space Complexity:**

- **Time Complexity**: `O(n)` where `n` is the length of the array. We need to traverse all elements once to compute the XOR.

- **Space Complexity**: `O(1)` as we only use a constant amount of extra space for the XOR calculation.

The beauty of this solution lies in its simplicity - what appears to be a complex game theory problem reduces to a single line checking two simple conditions. No need for recursion, dynamic programming, or game tree exploration - just pure mathematical insight translated into code.

## Example Walkthrough

Let's walk through two examples to illustrate how the solution works:

**Example 1: `nums = [1, 1, 2]`**

First, let's check our two winning conditions for Alice:
1. Calculate initial XOR: `1 ⊕ 1 ⊕ 2 = 0 ⊕ 2 = 2` (not zero)
2. Check array length: 3 elements (odd)

Since the initial XOR is not 0 and the array length is odd, Alice will lose.

Let's verify this by understanding the game flow:
- Initial state: `[1, 1, 2]`, XOR = 2
- Alice must make a move. Let's examine her options:
  - Remove first 1: Remaining `[1, 2]`, XOR = `1 ⊕ 2 = 3` (safe move)
  - Remove second 1: Remaining `[1, 2]`, XOR = `1 ⊕ 2 = 3` (safe move)  
  - Remove 2: Remaining `[1, 1]`, XOR = `1 ⊕ 1 = 0` (Alice loses!)

Alice can avoid losing on her first move by removing either 1. But now Bob has an even-length array `[1, 2]` with non-zero XOR. According to our pattern, Bob is now in the winning position - he can always make safe moves and eventually force Alice to lose.

**Example 2: `nums = [1, 2, 3, 4]`**

Checking winning conditions:
1. Calculate initial XOR: `1 ⊕ 2 ⊕ 3 ⊕ 4 = 3 ⊕ 3 ⊕ 4 = 0 ⊕ 4 = 4` (not zero)
2. Check array length: 4 elements (even)

Since the array length is even, Alice wins!

The mathematical guarantee tells us that Alice can always find at least one safe move. Let's verify:
- If Alice removes 1: Remaining XOR = `2 ⊕ 3 ⊕ 4 = 5` (safe)
- If Alice removes 2: Remaining XOR = `1 ⊕ 3 ⊕ 4 = 6` (safe)
- If Alice removes 3: Remaining XOR = `1 ⊕ 2 ⊕ 4 = 7` (safe)
- If Alice removes 4: Remaining XOR = `1 ⊕ 2 ⊕ 3 = 0` (would lose)

Alice has three safe moves available. After she makes any safe move, Bob gets an odd-length array with non-zero XOR (the losing position), and the pattern continues with Alice maintaining her advantage.

**Example 3: `nums = [5, 5]`**

Quick check:
1. Initial XOR: `5 ⊕ 5 = 0` (Alice wins immediately!)
2. Array length: 2 (even, but we don't need to check this)

Alice wins before making any move because the XOR is already 0 at the start of her turn.

These examples demonstrate how the solution's simple conditions correctly predict the winner without simulating the entire game.

## Python Solution

```python
from typing import List
from functools import reduce
from operator import xor

class Solution:
    def xorGame(self, nums: List[int]) -> bool:
        """
        Determines if Alice wins the XOR game.
      
        Alice wins if either:
        1. The array has an even number of elements (Alice always wins with optimal play)
        2. The XOR of all elements equals 0 (Alice wins immediately)
      
        Args:
            nums: List of integers representing the game array
          
        Returns:
            True if Alice wins, False otherwise
        """
        # Calculate the XOR of all elements in the array
        total_xor = reduce(xor, nums)
      
        # Alice wins if array length is even OR if XOR of all elements is 0
        return len(nums) % 2 == 0 or total_xor == 0
```

## Time and Space Complexity

The time complexity is `O(n)`, where `n` is the length of the array `nums`. This is because the `reduce(xor, nums)` operation needs to iterate through all elements in the array exactly once to compute the XOR of all elements. The length check `len(nums) % 2 == 0` is `O(1)`, so the overall time complexity is dominated by the XOR reduction operation.

The space complexity is `O(1)`. The algorithm only uses a constant amount of extra space regardless of the input size. The `reduce` function computes the XOR result iteratively without creating any additional data structures that scale with the input size, maintaining only the running XOR result.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Misunderstanding the Winning Condition**
A common mistake is thinking that a player loses when they *make* the XOR equal to 0, rather than understanding that they lose *after* making a move that results in XOR = 0. This subtle difference is crucial - if the XOR is already 0 at the start of a player's turn, that player wins immediately without making a move.

**Incorrect interpretation:**
```python
def xorGame(self, nums: List[int]) -> bool:
    # Wrong: Checking if Alice can make a move that creates XOR = 0
    total_xor = reduce(xor, nums)
    if total_xor == 0:
        return False  # Incorrectly assumes Alice loses if XOR is 0
    return len(nums) % 2 == 0
```

**Correct approach:**
```python
def xorGame(self, nums: List[int]) -> bool:
    # Right: Alice wins if XOR is already 0 (wins immediately)
    return len(nums) % 2 == 0 or reduce(xor, nums) == 0
```

### 2. **Manually Computing XOR Instead of Using reduce()**
Implementing XOR calculation with a loop is more error-prone and less elegant:

**Less optimal approach:**
```python
def xorGame(self, nums: List[int]) -> bool:
    total_xor = 0
    for num in nums:
        total_xor ^= num  # Easy to forget initialization or use wrong operator
    return len(nums) % 2 == 0 or total_xor == 0
```

**Better approach using reduce:**
```python
def xorGame(self, nums: List[int]) -> bool:
    return len(nums) % 2 == 0 or reduce(xor, nums) == 0
```

### 3. **Edge Case: Single Element Array**
Some might worry about handling arrays with a single element specially, but the solution handles this correctly:
- Array length 1 is odd
- XOR of single element is the element itself
- Alice loses unless that element is 0

**Unnecessary special case handling:**
```python
def xorGame(self, nums: List[int]) -> bool:
    if len(nums) == 1:
        return nums[0] == 0  # Redundant check
    return len(nums) % 2 == 0 or reduce(xor, nums) == 0
```

### 4. **Attempting to Simulate the Game**
The biggest pitfall is trying to actually simulate the game with recursion or dynamic programming:

**Overly complex approach:**
```python
def xorGame(self, nums: List[int]) -> bool:
    def canWin(arr, is_alice_turn):
        current_xor = reduce(xor, arr) if arr else 0
        if current_xor == 0:
            return is_alice_turn
      
        for i in range(len(arr)):
            new_arr = arr[:i] + arr[i+1:]
            if not new_arr or reduce(xor, new_arr) == 0:
                continue  # This move loses (XOR of an empty board is 0)
            if not canWin(new_arr, not is_alice_turn):
                return True
        return False
  
    return canWin(nums, True)
```

This approach gives the right answers but is unnecessarily complex and has exponential time complexity, when the mathematical insight gives us an O(n) solution. Note the `not new_arr` guard: `reduce(xor, [])` without an initial value raises `TypeError`, so removing the last element must be handled separately.

