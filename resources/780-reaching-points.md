# 780. Reaching Points

> Source: https://algo.monster/liteproblems/780

## Problem Statement

You are given four integers: starting coordinates `(sx, sy)` and target coordinates `(tx, ty)`. Your goal is to determine if you can transform the starting point into the target point through a series of allowed operations.

The allowed operations are:
- From any point `(x, y)`, you can move to `(x, x + y)` - this adds the x-coordinate to the y-coordinate
- From any point `(x, y)`, you can move to `(x + y, y)` - this adds the y-coordinate to the x-coordinate

Starting from `(sx, sy)`, you need to check if it's possible to reach exactly `(tx, ty)` by applying these operations any number of times.

For example:
- Starting from `(1, 1)`, you could go to `(1, 2)` using the first operation
- From `(1, 2)`, you could go to `(3, 2)` using the second operation
- From `(3, 2)`, you could go to `(3, 5)` using the first operation
- And so on...

The function should return `true` if the target point is reachable from the starting point, and `false` otherwise.

## Intuition

The key observation is that moving forward from `(sx, sy)` to `(tx, ty)` can create an exponentially large search space, making it inefficient to explore all possible paths. However, working backwards from `(tx, ty)` to `(sx, sy)` is much more efficient.

Why work backwards? When we're at point `(tx, ty)`, we can determine exactly which operation was used to reach it:
- If `tx > ty`, the last operation must have been `(tx - ty, ty) → (tx, ty)`
- If `ty > tx`, the last operation must have been `(tx, ty - tx) → (tx, ty)`

This gives us a unique path backwards, eliminating the branching factor that exists when moving forward.

The brilliant optimization comes from recognizing that if `tx >> ty` (much greater), we would need to subtract `ty` from `tx` many times repeatedly. Instead of doing this one by one, we can use the modulo operation: `tx % ty` gives us the result after all possible subtractions of `ty` from `tx`. This reduces the time complexity from linear to logarithmic.

For example, if we have `(1000, 3)` and need to reach `(1, 3)`, instead of subtracting 3 from 1000 repeatedly (333 operations), we can directly compute `1000 % 3 = 1`.

The algorithm stops when:
1. We reach `(sx, sy)` exactly - return `true`
2. One coordinate matches while the other is still larger - check if the difference is divisible by the matching coordinate
3. We go below either `sx` or `sy` - impossible to reach, return `false`

This approach transforms an exponential forward search into an efficient logarithmic backward traversal.

**Learn more about [Math](/problems/math-basics) patterns.**

## Solution Approach

The implementation uses a working-backwards approach with modulo optimization to efficiently determine if we can reach `(sx, sy)` from `(tx, ty)`.

**Main Loop - Backward Traversal:**
```python
while tx > sx and ty > sy and tx != ty:
    if tx > ty:
        tx %= ty
    else:
        ty %= tx
```

This loop continues as long as both coordinates are larger than their starting counterparts and they're not equal. In each iteration:
- If `tx > ty`, we know the previous state was `(tx - n*ty, ty)` for some positive integer `n`. Using `tx %= ty` directly computes the result after maximum possible subtractions.
- If `ty > tx`, similarly we use `ty %= tx` to reverse multiple operations at once.

The condition `tx != ty` stops the loop when the coordinates are equal. A point such as `(5, 5)` cannot be produced from any point with positive coordinates, so there is nothing further to undo; the terminal checks below then decide the answer.

**Terminal Cases Check:**

After the loop terminates, we have four possible scenarios:

1. **Exact Match:**
```python
if tx == sx and ty == sy:
    return True
```
We've successfully traced back to the starting point.

2. **X-coordinate Matches:**
```python
if tx == sx:
    return ty > sy and (ty - sy) % tx == 0
```
When `tx` equals `sx`, we can only perform operations that add `tx` to `ty`. We check if:
- `ty > sy` (we haven't gone past the target)
- `(ty - sy) % tx == 0` (the difference is achievable by adding `tx` multiple times)

3. **Y-coordinate Matches:**
```python
if ty == sy:
    return tx > sx and (tx - sx) % ty == 0
```
Similar logic when `ty` equals `sy` - we can only add `ty` to `tx` repeatedly.

4. **No Match:**
```python
return False
```
If none of the above conditions are met, it's impossible to reach the target.

**Time Complexity:** `O(log(max(tx, ty)))` due to the modulo operations which work similarly to the Euclidean algorithm for GCD.

**Space Complexity:** `O(1)` as we only use a constant amount of extra space.

## Example Walkthrough

Let's trace through an example where we start at `(sx, sy) = (1, 1)` and want to reach `(tx, ty) = (5, 12)`.

**Working Backwards from (5, 12) to (1, 1):**

**Step 1:** Start with `(tx, ty) = (5, 12)`
- Since `ty > tx` (12 > 5), the previous point must have been `(5, 12 - 5) = (5, 7)`
- We know this because to get to `(5, 12)`, we must have added 5 to the y-coordinate

**Step 2:** Now at `(5, 7)`
- Since `ty > tx` (7 > 5), the previous point was `(5, 7 - 5) = (5, 2)`

**Step 3:** Now at `(5, 2)`
- Since `tx > ty` (5 > 2), the previous point was `(5 - 2, 2) = (3, 2)`

**Step 4:** Now at `(3, 2)`
- Since `tx > ty` (3 > 2), the previous point was `(3 - 2, 2) = (1, 2)`

**Step 5:** Now at `(1, 2)`
- We have `tx = sx = 1` (x-coordinate matches)
- Check if we can reach `(1, 1)` from `(1, 2)`:
  - Is `ty > sy`? Yes, 2 > 1 ✓
  - Is `(ty - sy) % tx == 0`? Is `(2 - 1) % 1 == 0`? Yes, 1 % 1 = 0 ✓
- Therefore, we can reach `(1, 1)` by one more backward step

**Result:** Return `true` - we successfully traced a path from `(5, 12)` back to `(1, 1)`.

The code performs these steps with modulo, which merges repeated subtractions of the same value: the first loop iteration computes `ty = 12 % 5 = 2` (Steps 1-2), and the second computes `tx = 5 % 2 = 1` (Steps 3-4). The loop then stops because `tx > sx` is false, and the `tx == sx` check from Step 5 returns `true`.

**Optimization with Modulo:**
If we had `(sx, sy) = (1, 1)` and `(tx, ty) = (1000, 3)`, the algorithm would:
1. Start with `(1000, 3)`
2. Since `tx > ty` (1000 > 3), compute `tx = 1000 % 3 = 1`
3. Now we have `(1, 3)`
4. Check terminal case: `tx = sx = 1` matches
5. Check if `(3 - 1) % 1 == 0`? Yes, so return `true`

This avoided 333 individual subtractions by using the modulo operation!

## Python Solution

```python
class Solution:
    def reachingPoints(self, sx: int, sy: int, tx: int, ty: int) -> bool:
        """
        Determine if we can reach target point (tx, ty) from source point (sx, sy).
        At each step, we can transform (x, y) to either (x + y, y) or (x, x + y).
      
        The key insight is to work backwards from target to source, since going forward
        has many possible paths but going backward is deterministic.
        """
      
        # Work backwards from target point by reducing the larger coordinate
        # When tx > ty, the previous step must have been (tx - ty, ty)
        # We use modulo to skip multiple subtraction steps at once
        while tx > sx and ty > sy and tx != ty:
            if tx > ty:
                # Reduce tx by multiples of ty
                tx %= ty
            else:
                # Reduce ty by multiples of tx
                ty %= tx
      
        # Check if we've reached the source point exactly
        if tx == sx and ty == sy:
            return True
      
        # If x-coordinate matches, check if we can reach source by only changing y
        if tx == sx:
            # ty must be reachable from sy by adding tx multiple times
            return ty > sy and (ty - sy) % tx == 0
      
        # If y-coordinate matches, check if we can reach source by only changing x
        if ty == sy:
            # tx must be reachable from sx by adding ty multiple times
            return tx > sx and (tx - sx) % ty == 0
      
        # Cannot reach the source point
        return False
```

## Time and Space Complexity

**Time Complexity:** `O(log(max(tx, ty)))`

The algorithm works backwards from `(tx, ty)` to `(sx, sy)`. In each iteration of the while loop, we perform a modulo operation that significantly reduces the larger value:
- When `tx > ty`, we perform `tx %= ty`, which reduces `tx` to at most `ty - 1`
- When `ty > tx`, we perform `ty %= tx`, which reduces `ty` to at most `tx - 1`

This reduction pattern is similar to the Euclidean algorithm for finding GCD. In the worst case, the number of iterations is proportional to the logarithm of the maximum of `tx` and `ty`. Each modulo operation takes `O(1)` time, resulting in an overall time complexity of `O(log(max(tx, ty)))`.

**Space Complexity:** `O(1)`

The algorithm uses only a constant amount of extra space. We modify the input values `tx` and `ty` in-place without using any additional data structures or recursive calls. All operations are performed using the existing variables, requiring only `O(1)` auxiliary space.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Using `>=` in the Loop Condition**

**The Pitfall:**
Writing the loop as `while tx >= sx and ty >= sy and tx != ty` lets the modulo step keep running after one coordinate has already reached its source value. The loop then reduces past the source and the terminal checks fail.

For example, with `sx = 1, sy = 1, tx = 3, ty = 2`, the correct answer is `True` (`(1, 1) → (1, 2) → (3, 2)`). With `>=`, the loop computes `tx = 3 % 2 = 1`, and because `tx >= sx` still holds, it continues with `ty = 2 % 1 = 0`. The final check `ty > sy` fails and the function returns `False`.

**Solution:**
Use strict inequalities so the loop stops as soon as either coordinate reaches its source value, and let the terminal checks handle the remaining coordinate:
```python
while tx > sx and ty > sy and tx != ty:
```

The `tx != ty` part stops the loop at a point such as `(5, 5)`, which has no valid predecessor because a predecessor would need a `0` coordinate. Removing it does not crash the code: `ty %= tx` sets `ty` to `0`, the loop ends because `ty > sy` is false, and the function returns `False`, which is the right answer since the loop only runs while `tx > sx`. Keeping the check makes the intent explicit.

### 2. **Incorrect Terminal Case Validation**

**The Pitfall:**
A common mistake is to check only `(ty - sy) % tx == 0` without verifying `ty > sy` (or similarly for the x-coordinate case). This could incorrectly return `True` for cases where we've "overshot" the target in the backward traversal.

For example, if `sx = 3, sy = 7, tx = 3, ty = 4`, without the `ty > sy` check, we might incorrectly validate `(4 - 7) % 3 == 0` which gives `True` since `-3 % 3 == 0`, but we cannot actually reach `(3, 4)` from `(3, 7)` going forward.

**Solution:**
Always verify both conditions in terminal cases:
```python
if tx == sx:
    return ty > sy and (ty - sy) % tx == 0  # Must check ty > sy
if ty == sy:
    return tx > sx and (tx - sx) % ty == 0  # Must check tx > sx
```

### 3. **Not Handling Edge Cases Where Source Equals Target**

**The Pitfall:**
The algorithm might not immediately handle the case where `(sx, sy) == (tx, ty)` if the implementation doesn't check for this condition before or during the main loop.

**Solution:**
The current implementation handles this naturally - when source equals target, the while loop doesn't execute, and the first condition `if tx == sx and ty == sy: return True` catches this case. However, for clarity, you could add an early return:
```python
if sx == tx and sy == ty:
    return True  # Already at target
```

### 4. **Using Subtraction Instead of Modulo**

**The Pitfall:**
Using repeated subtraction instead of modulo operation:
```python
# Inefficient approach - DO NOT USE
while tx > ty:
    tx -= ty  # This could take millions of iterations
```
This would cause Time Limit Exceeded (TLE) for large differences between coordinates, such as `tx = 1000000000, ty = 1`.

**Solution:**
Always use modulo for efficiency:
```python
if tx > ty:
    tx %= ty  # Reduces tx in O(1) time
```
This reduces the time complexity from potentially O(max(tx, ty)) to O(log(max(tx, ty))).

