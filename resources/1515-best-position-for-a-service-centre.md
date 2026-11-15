# 1515. Best Position for a Service Centre

> Source: https://algo.monster/liteproblems/1515

## Problem Statement

A delivery company needs to find the optimal location for a new service center in a city. The company has the coordinates of all customers on a 2D map and wants to position the service center to minimize the total distance to all customers.

Given an array `positions` where `positions[i] = [xi, yi]` represents the coordinates of the `i-th` customer, you need to find the location `[x_centre, y_centre]` for the service center that minimizes the sum of Euclidean distances to all customers.

The Euclidean distance between two points `(x1, y1)` and `(x2, y2)` is calculated as `√((x1-x2)² + (y1-y2)²)`.

Your goal is to minimize the total sum:
`Σ √((x_centre - xi)² + (y_centre - yi)²)` for all customers `i`.

The solution uses gradient descent optimization to find the optimal center position. Starting from the centroid (average position) of all customers, it iteratively adjusts the center location by following the negative gradient direction. The gradient indicates how the total distance changes with respect to small movements in x and y directions. The learning rate (`alpha`) gradually decreases to ensure convergence, and the algorithm stops when the position changes become smaller than a threshold (`eps`).

The answer should be accurate within `10^-5` of the actual minimum sum of distances.

## Intuition

The key insight is that finding the optimal service center location is a continuous optimization problem where we want to minimize a smooth, convex function. Unlike finding the median (which minimizes Manhattan distance), minimizing Euclidean distances doesn't have a simple closed-form solution.

Think of this problem as finding the lowest point in a valley. If you're standing anywhere on the valley slope, the steepest downhill direction tells you which way to move to get lower. This is exactly what gradient descent does - it calculates the "slope" (gradient) at our current position and moves in the opposite direction to go downhill.

The gradient at any point tells us how the total distance changes when we make small movements. For each customer at position `(xi, yi)`, their contribution to the gradient is proportional to the direction from our current center to that customer, weighted by `1/distance`. This makes intuitive sense: customers who are farther away have less "pull" on where the center should be.

Starting from the centroid (average of all customer positions) is a natural choice because it's already a reasonable approximation - it would be optimal if we were minimizing squared distances instead of regular distances.

The learning rate `alpha` starts at 0.5 and gradually decreases (multiplied by 0.999 each iteration). This is crucial because:
- Initially, we want to take larger steps to quickly approach the optimal region
- As we get closer to the minimum, smaller steps prevent overshooting
- The decaying rate ensures convergence to a stable solution

The algorithm terminates when the position updates become negligibly small (less than `eps = 1e-6`), indicating we've reached a local minimum which, due to the convex nature of the problem, is also the global minimum.

**Learn more about [Math](/problems/math-basics) patterns.**

## Solution Approach

The implementation uses gradient descent optimization to find the optimal service center location. Here's a detailed walkthrough of the algorithm:

**Initialization:**
- Calculate the centroid of all customer positions as the starting point: `x = Σxi/n`, `y = Σyi/n`
- Set the initial learning rate `alpha = 0.5`
- Define the decay factor `decay = 0.999` to gradually reduce the learning rate
- Set convergence threshold `eps = 1e-6`

**Gradient Descent Loop:**

1. **Calculate Gradient and Total Distance:**
   For each iteration, compute the gradient components and current total distance:
   ```python
   grad_x = grad_y = 0
   dist = 0
   for x1, y1 in positions:
       a = x - x1  # x-component of vector from customer to center
       b = y - y1  # y-component of vector from customer to center
       c = sqrt(a * a + b * b)  # Euclidean distance
       grad_x += a / (c + 1e-8)  # Partial derivative w.r.t x
       grad_y += b / (c + 1e-8)  # Partial derivative w.r.t y
       dist += c  # Accumulate total distance
   ```
 
   The gradient formula comes from the derivative of `√((x-xi)² + (y-yi)²)`:
   - `∂/∂x = (x-xi)/√((x-xi)² + (y-yi)²)`
   - `∂/∂y = (y-yi)/√((x-xi)² + (y-yi)²)`
 
   The `1e-8` term prevents division by zero when the center coincides with a customer location.

2. **Update Position:**
   Move the center in the negative gradient direction (downhill):
   ```python
   dx = grad_x * alpha
   dy = grad_y * alpha
   x -= dx
   y -= dy
   ```

3. **Decay Learning Rate:**
   ```python
   alpha *= decay
   ```
   This ensures smaller steps as we approach the minimum, improving convergence stability.

4. **Check Convergence:**
   ```python
   if abs(dx) <= eps and abs(dy) <= eps:
       return dist
   ```
   When position updates become smaller than the threshold, we've found the optimal location and return the minimum sum of distances.

The algorithm uses the convexity of the objective function - the sum of Euclidean distances forms a convex surface with a single global minimum, guaranteeing that gradient descent will find the optimal solution regardless of minor variations in the starting point.

## Example Walkthrough

Let's walk through a small example with 3 customers to illustrate how gradient descent finds the optimal service center location.

**Given:** `positions = [[0, 1], [1, 0], [1, 2]]`

**Step 1: Initialize at Centroid**
- Calculate starting position (centroid):
  - x = (0 + 1 + 1) / 3 = 0.667
  - y = (1 + 0 + 2) / 3 = 1.0
- Starting point: `(0.667, 1.0)`
- Initial learning rate: `alpha = 0.5`

**Step 2: First Iteration - Calculate Gradient**

For each customer, calculate their contribution to the gradient:

Customer 1 at `(0, 1)`:
- Vector from customer to center: `a = 0.667 - 0 = 0.667`, `b = 1.0 - 1 = 0`
- Distance: `c = √(0.667² + 0²) = 0.667`
- Gradient contribution: `grad_x += 0.667/0.667 = 1.0`, `grad_y += 0/0.667 = 0`

Customer 2 at `(1, 0)`:
- Vector: `a = 0.667 - 1 = -0.333`, `b = 1.0 - 0 = 1.0`
- Distance: `c = √(0.333² + 1.0²) = 1.054`
- Gradient contribution: `grad_x += -0.333/1.054 = -0.316`, `grad_y += 1.0/1.054 = 0.949`

Customer 3 at `(1, 2)`:
- Vector: `a = 0.667 - 1 = -0.333`, `b = 1.0 - 2 = -1.0`
- Distance: `c = √(0.333² + 1.0²) = 1.054`
- Gradient contribution: `grad_x += -0.333/1.054 = -0.316`, `grad_y += -1.0/1.054 = -0.949`

Total gradient: `grad_x = 1.0 - 0.316 - 0.316 = 0.368`, `grad_y = 0 + 0.949 - 0.949 = 0`
Total distance at this position: `0.667 + 1.054 + 1.054 = 2.775`

**Step 3: Update Position**
- Movement: `dx = 0.368 * 0.5 = 0.184`, `dy = 0 * 0.5 = 0`
- New position: `x = 0.667 - 0.184 = 0.483`, `y = 1.0 - 0 = 1.0`
- Update learning rate: `alpha = 0.5 * 0.999 = 0.4995`

**Step 4: Check Convergence**
- Since `|dx| = 0.184 > 1e-6`, continue iterating

**Subsequent Iterations:**

The algorithm continues this process:
- Iteration 2: Position moves to approximately `(0.395, 1.0)`, distance = 2.733
- Iteration 3: Position moves to approximately `(0.350, 1.0)`, distance = 2.718
- ...
- After many iterations: Position converges to approximately `(0.5, 1.0)`

**Final Result:**
The optimal service center location is near `(0.5, 1.0)` with a minimum total distance of approximately 2.707.

This makes intuitive sense - the optimal location is pulled towards the cluster of two customers at x=1 (customers 2 and 3) but balanced by customer 1 at x=0. The y-coordinate settles at 1.0, which is the median of the y-values (0, 1, 2), minimizing vertical distances.

The gradient descent successfully navigates from the initial centroid to the true geometric median, demonstrating how the algorithm efficiently finds the point that minimizes the sum of Euclidean distances to all customers.

## Python Solution

```python
from typing import List
from math import sqrt


class Solution:
    def getMinDistSum(self, positions: List[List[int]]) -> float:
        """
        Find the geometric median - a point that minimizes the sum of Euclidean distances
        to all given positions using gradient descent optimization.
      
        Args:
            positions: List of [x, y] coordinates
          
        Returns:
            Minimum sum of distances from the optimal point to all positions
        """
        # Calculate the centroid as initial guess for the optimal point
        num_positions = len(positions)
        center_x = sum(pos[0] for pos in positions) / num_positions
        center_y = sum(pos[1] for pos in positions) / num_positions
      
        # Gradient descent parameters
        learning_rate = 0.5  # Initial step size for gradient descent
        decay_factor = 0.999  # Decay factor to reduce learning rate over iterations
        convergence_threshold = 1e-6  # Threshold for convergence detection
        epsilon = 1e-8  # Small value to avoid division by zero
      
        # Iterative gradient descent optimization
        while True:
            # Calculate gradients and total distance
            gradient_x = 0.0
            gradient_y = 0.0
            total_distance = 0.0
          
            for position_x, position_y in positions:
                # Calculate distance components
                delta_x = center_x - position_x
                delta_y = center_y - position_y
                distance = sqrt(delta_x * delta_x + delta_y * delta_y)
              
                # Accumulate partial derivatives (gradient components)
                # Adding epsilon to avoid division by zero when point coincides with a position
                gradient_x += delta_x / (distance + epsilon)
                gradient_y += delta_y / (distance + epsilon)
              
                # Accumulate total distance
                total_distance += distance
          
            # Calculate step sizes based on gradients and learning rate
            step_x = gradient_x * learning_rate
            step_y = gradient_y * learning_rate
          
            # Update center point by moving against gradient direction
            center_x -= step_x
            center_y -= step_y
          
            # Decay the learning rate for finer adjustments
            learning_rate *= decay_factor
          
            # Check convergence: if steps are smaller than threshold, we've converged
            if abs(step_x) <= convergence_threshold and abs(step_y) <= convergence_threshold:
                return total_distance
```

## Time and Space Complexity

**Time Complexity:** `O(n * k)` where `n` is the number of positions and `k` is the number of iterations until convergence.

- The initial calculation of the centroid takes `O(n)` time to sum all coordinates.
- The main gradient descent loop runs for `k` iterations until convergence (when `abs(dx) <= eps and abs(dy) <= eps`).
- In each iteration, we iterate through all `n` positions to calculate:
  - The gradient components `grad_x` and `grad_y`
  - The total distance `dist`
  - This inner loop takes `O(n)` time per iteration
- The number of iterations `k` depends on the convergence criteria and decay rate. With `decay = 0.999` and `eps = 1e-6`, the algorithm typically converges in a constant number of iterations that doesn't depend on `n`, but in worst case could be `O(log(1/eps))`.
- Overall time complexity: `O(n * k)` where `k = O(log(1/eps))` in practice.

**Space Complexity:** `O(1)`

- The algorithm uses only a constant amount of extra space:
  - Variables for coordinates: `x`, `y`
  - Variables for gradients: `grad_x`, `grad_y`
  - Variables for updates: `dx`, `dy`
  - Other scalar variables: `dist`, `alpha`, `decay`, `eps`, `n`, `a`, `b`, `c`
- All these variables require constant space regardless of the input size.
- The input list `positions` is not counted as extra space since it's part of the input.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Numerical Instability When Distance Approaches Zero**

**Problem:** When the center point gets very close to or coincides with a customer position, the distance becomes zero or near-zero, causing division by zero in the gradient calculation. This leads to `NaN` or infinite values that corrupt the optimization process.

**Example Scenario:**
```python
# If center_x = 5, center_y = 3 and a customer is at [5, 3]
delta_x = 5 - 5 = 0
delta_y = 3 - 3 = 0
distance = sqrt(0 + 0) = 0
gradient_x += 0 / 0  # Division by zero!
```

**Solution:** Add a small epsilon value to the denominator:
```python
gradient_x += delta_x / (distance + epsilon)  # epsilon = 1e-8
gradient_y += delta_y / (distance + epsilon)
```

### 2. **Premature Convergence with Fixed Learning Rate**

**Problem:** Using a constant learning rate can cause the algorithm to oscillate around the minimum without converging, or converge too slowly. Large learning rates cause overshooting, while small rates make convergence extremely slow.

**Example of Oscillation:**
```python
# With fixed learning_rate = 1.0
# The center might jump back and forth across the optimal point
# Iteration 1: center_x = 10 → 2 (overshoot)
# Iteration 2: center_x = 2 → 9 (overshoot back)
# ... continues oscillating
```

**Solution:** Implement learning rate decay:
```python
learning_rate *= decay_factor  # Gradually reduce step size
# Start with learning_rate = 0.5
# After 100 iterations: 0.5 * (0.999^100) ≈ 0.45
# After 1000 iterations: 0.5 * (0.999^1000) ≈ 0.18
```

### 3. **Poor Initial Point Selection**

**Problem:** Starting from an arbitrary point (like origin [0,0]) when all customers are far from it can lead to slow convergence and numerical issues due to large gradients.

**Example:**
```python
# If all customers are around [1000, 1000] but we start at [0, 0]
# Initial gradients will be very large, causing unstable updates
```

**Solution:** Use the centroid (mean position) as the starting point:
```python
center_x = sum(pos[0] for pos in positions) / len(positions)
center_y = sum(pos[1] for pos in positions) / len(positions)
```

### 4. **Incorrect Convergence Criteria**

**Problem:** Using absolute distance threshold instead of step size can cause premature termination when far from optimum, or infinite loops when the threshold is too strict.

**Incorrect approach:**
```python
# Wrong: checking if total distance change is small
if abs(current_dist - previous_dist) < threshold:
    return  # Might stop when on a plateau, not at minimum
```

**Solution:** Check if the position updates (step sizes) are small:
```python
if abs(step_x) <= convergence_threshold and abs(step_y) <= convergence_threshold:
    return total_distance  # Converged when movement is negligible
```

### 5. **Accumulating Floating-Point Errors**

**Problem:** In problems with many customers or requiring high precision, small floating-point errors can accumulate and affect the final result accuracy.

**Solution:** Use appropriate data types and consider numerical stability:
```python
# Use double precision floats
gradient_x = 0.0  # Not 0 (which might be int)
gradient_y = 0.0

# Consider using higher precision libraries for extreme cases
# from decimal import Decimal (if needed)
```

