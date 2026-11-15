# 458. Poor Pigs

> Source: https://algo.monster/liteproblems/458

## Problem Statement

You have a certain number of `buckets` filled with liquid, and exactly one bucket contains poison. Your goal is to identify which bucket is poisonous using the minimum number of test pigs within a given time limit.

The testing process works as follows:

1. **Test Setup**: You have `minutesToTest` total minutes available for testing, and each test round takes `minutesToDie` minutes to see results.

2. **Testing Rules**:
   - You can select any number of live pigs for each test round
   - Each pig can drink from multiple buckets simultaneously 
   - Multiple pigs can drink from the same bucket
   - Drinking takes no time - pigs consume all assigned buckets instantly

3. **Test Results**: After waiting `minutesToDie` minutes:
   - Any pig that drank from the poisonous bucket will die
   - All other pigs survive
   - You cannot feed pigs during the waiting period

4. **Test Rounds**: You can repeat this process multiple times as long as you have time remaining (`minutesToTest` allows for `minutesToTest // minutesToDie` rounds of testing).

The problem asks you to find the **minimum number of pigs** needed to guarantee you can identify the poisonous bucket within the time limit.

**Example Logic**: If you have 4 test rounds available (based on the time constraints), each pig can effectively be in one of 5 states after all rounds: died in round 1, died in round 2, died in round 3, died in round 4, or survived all rounds. This means each pig can distinguish between 5 different outcomes, so `n` pigs can distinguish between `5^n` different buckets.

## Intuition

The key insight is to think of this problem as an **information theory** problem. Each pig can provide us with information based on when it dies (or if it survives).

Consider what happens with one pig over the testing period:
- If we have `minutesToTest = 60` and `minutesToDie = 15`, we can conduct `60 / 15 = 4` test rounds
- This single pig can die in round 1, round 2, round 3, round 4, or survive all rounds
- That's **5 different outcomes** (4 death times + 1 survival)

This means one pig can distinguish between 5 different buckets by assigning each bucket to a specific outcome. But what if we have more buckets?

With two pigs, we can think of it as a **coordinate system**:
- Pig 1 has 5 possible states (dies in rounds 1-4 or survives)
- Pig 2 also has 5 possible states
- Combined, they create a 5×5 grid = **25 different combinations**
- Each combination can identify a unique bucket

This extends to multiple dimensions. With `n` pigs, where each pig has `base` possible states (number of test rounds + 1), we can identify `base^n` different buckets.

The formula becomes:
- `base = (minutesToTest // minutesToDie) + 1` (number of possible states per pig)
- We need `base^n >= buckets`
- Solving for `n`: we need the minimum `n` such that `base^n >= buckets`

The solution implements this by starting with `p = 1` and repeatedly multiplying by `base` while counting iterations until `p >= buckets`. This gives us the minimum number of pigs needed.

## Solution Approach

The implementation uses a simple iterative approach to find the minimum number of pigs needed:

1. **Calculate the base (number of states per pig)**:
   ```python
   base = minutesToTest // minutesToDie + 1
   ```
   This represents how many different states each pig can have (dying in each round or surviving all rounds).

2. **Initialize variables**:
   ```python
   res, p = 0, 1
   ```
   - `res`: counts the number of pigs needed
   - `p`: tracks the total number of buckets we can test with `res` pigs (starts at 1)

3. **Iteratively find minimum pigs**:
   ```python
   while p < buckets:
       p *= base
       res += 1
   ```
   - Each iteration adds one more pig
   - Multiplying `p` by `base` represents adding another dimension to our testing space
   - When `p >= buckets`, we have enough pigs to test all buckets

4. **Return the result**:
   ```python
   return res
   ```

**Example walkthrough**:
- If `buckets = 25`, `minutesToDie = 15`, `minutesToTest = 60`
- `base = 60 // 15 + 1 = 5`
- Iteration 1: `p = 1 * 5 = 5`, `res = 1` (1 pig can test 5 buckets)
- Iteration 2: `p = 5 * 5 = 25`, `res = 2` (2 pigs can test 25 buckets)
- Since `p >= 25`, we return `res = 2`

The algorithm essentially computes `⌈log_base(buckets)⌉`, which is the minimum power of `base` needed to equal or exceed the number of buckets.

## Example Walkthrough

Let's walk through a concrete example with **8 buckets**, where `minutesToDie = 15` and `minutesToTest = 30`.

**Step 1: Calculate the base (states per pig)**
- We can conduct `30 // 15 = 2` test rounds
- Each pig has `2 + 1 = 3` possible states:
  - Dies in round 1
  - Dies in round 2  
  - Survives both rounds
- So `base = 3`

**Step 2: Determine minimum pigs needed**

Starting with `res = 0` (no pigs) and `p = 1` (can test 1 bucket):

*First iteration:*
- `p = 1 < 8`, so we need more pigs
- Add a pig: `p = 1 * 3 = 3`, `res = 1`
- With 1 pig, we can test 3 buckets

*Second iteration:*
- `p = 3 < 8`, still need more pigs
- Add another pig: `p = 3 * 3 = 9`, `res = 2`
- With 2 pigs, we can test 9 buckets

Since `p = 9 >= 8`, we have enough capacity. We need **2 pigs**.

**Step 3: How the actual testing works**

With 2 pigs and 8 buckets, we can arrange them in a 3×3 grid (using only 8 of the 9 positions):

```
        Pig 2 dies    Pig 2 dies    Pig 2
        round 1       round 2       survives
Pig 1   Bucket 1     Bucket 2      Bucket 3
round 1

Pig 1   Bucket 4     Bucket 5      Bucket 6
round 2

Pig 1   Bucket 7     Bucket 8      (unused)
survives
```

**Testing protocol:**
- Round 1: Pig 1 drinks from buckets 1,2,3; Pig 2 drinks from buckets 1,4,7
- Round 2: Pig 1 drinks from buckets 4,5,6; Pig 2 drinks from buckets 2,5,8

Based on when each pig dies (or survives), we can pinpoint the exact poisonous bucket. For example, if Pig 1 dies in round 2 and Pig 2 survives both rounds, the poison is in Bucket 6.

## Python Solution

```python
class Solution:
    def poorPigs(self, buckets: int, minutesToDie: int, minutesToTest: int) -> int:
        # Calculate the number of test rounds possible
        # Each pig can be tested (minutesToTest // minutesToDie) times
        # Plus 1 for the case where the pig doesn't drink and survives
        test_rounds = minutesToTest // minutesToDie + 1
      
        # Initialize the number of pigs needed
        num_pigs = 0
      
        # Calculate the maximum number of buckets that can be tested
        # with the current number of pigs
        max_testable_buckets = 1
      
        # Keep adding pigs until we can test all buckets
        # Each pig can distinguish between 'test_rounds' states
        # N pigs can distinguish between test_rounds^N buckets
        while max_testable_buckets < buckets:
            max_testable_buckets *= test_rounds
            num_pigs += 1
      
        return num_pigs
```

## Time and Space Complexity

**Time Complexity:** `O(log_base(buckets))` where `base = minutesToTest // minutesToDie + 1`

The algorithm uses a while loop that continues until `p >= buckets`. In each iteration, `p` is multiplied by `base`, starting from 1. The loop terminates when `p` reaches or exceeds `buckets`. This means the loop runs approximately `log_base(buckets)` times, since we're essentially finding the smallest integer `res` such that `base^res >= buckets`.

**Space Complexity:** `O(1)`

The algorithm only uses a constant amount of extra space. It maintains three variables: `base`, `res`, and `p`, regardless of the input size. No additional data structures like arrays or recursive call stacks are used, making the space complexity constant.

## Common Pitfalls

### 1. **Misunderstanding the Base Calculation**

**Pitfall**: A common mistake is calculating the base as `minutesToTest // minutesToDie` without adding 1, forgetting that a pig surviving all rounds is also a valid state.

**Incorrect Implementation**:
```python
# Wrong: Missing the "+1" for the survival state
test_rounds = minutesToTest // minutesToDie  # ❌
```

**Why it's wrong**: If you have 60 minutes total and each test takes 15 minutes, you can do 4 test rounds. But each pig has 5 possible states:
- Dies in round 1
- Dies in round 2  
- Dies in round 3
- Dies in round 4
- Survives all rounds (doesn't drink poison)

**Solution**: Always add 1 to account for the survival state:
```python
test_rounds = minutesToTest // minutesToDie + 1  # ✓
```

### 2. **Replacing the Loop With Floating-Point Logarithms**

**Pitfall**: The answer is `⌈log_base(buckets)⌉`, so it is tempting to compute it directly with `math.log`. Floating-point rounding makes this unreliable at exact powers of the base:

```python
import math

# WRONG: floating-point error at exact powers
return math.ceil(math.log(buckets) / math.log(test_rounds))
```

For `buckets = 125` and `test_rounds = 5`, `math.log(125) / math.log(5)` evaluates to `3.0000000000000004`, so `math.ceil` returns 4 instead of 3.

**Solution**: Keep the integer loop from the solution. It multiplies exact integers, needs only a few iterations for the problem's limits (`buckets ≤ 1000`), and cannot overflow in Python. In languages with fixed-size integers, the product stays below `buckets × base`, which fits easily in a 64-bit integer.

### 3. **Edge Case: Single Bucket**

**Pitfall**: Not handling the edge case where `buckets = 1` properly. If there's only one bucket, it must be poisonous, so no pigs are needed.

**Solution**: Add an explicit check:
```python
def poorPigs(self, buckets: int, minutesToDie: int, minutesToTest: int) -> int:
    if buckets == 1:
        return 0
  
    # Rest of the implementation...
```

### 4. **Conceptual Misunderstanding: Sequential vs. Parallel Testing**

**Pitfall**: Thinking that pigs must test buckets sequentially or that each pig can only test one bucket per round. This leads to vastly overestimating the number of pigs needed.

**Wrong Mental Model**: "Each pig tests one bucket at a time"

**Correct Understanding**: Each pig can drink from multiple buckets in a single round, and the combination of which rounds different pigs die in creates a unique "death pattern" that identifies the poisonous bucket. Think of it as an n-dimensional coordinate system where each pig represents a dimension.

