# 765. Couples Holding Hands

> Source: https://algo.monster/liteproblems/765

## Problem Statement

You have `n` couples (so `2n` people total) sitting in a row of `2n` seats. Each person has a unique ID number, and couples are paired sequentially: couple 0 consists of persons `(0, 1)`, couple 1 consists of persons `(2, 3)`, couple 2 consists of persons `(4, 5)`, and so on, with the last couple being persons `(2n - 2, 2n - 1)`.

The goal is to arrange the seating so that every couple sits next to each other (side by side). You're given an array `row` where `row[i]` represents the ID of the person currently sitting in seat `i`.

To achieve this arrangement, you can perform swaps. A swap involves selecting any two people in the row, having them stand up, and switching their seats.

Your task is to find the minimum number of swaps needed to ensure all couples are sitting together.

For example, if `row = [0, 2, 1, 3]`, person 0 is sitting in seat 0, person 2 in seat 1, person 1 in seat 2, and person 3 in seat 3. Since couple 0 (persons 0 and 1) are not sitting together, and couple 1 (persons 2 and 3) are not sitting together, we need to swap. One swap of persons in seats 1 and 2 would result in `[0, 1, 2, 3]`, where both couples are now sitting together.

## Intuition

The key insight is to think about what happens when couples are sitting in the wrong positions. If couple A has one person sitting with someone from couple B, then couple B must have their other person sitting somewhere else, possibly with someone from couple C, and so on. This creates a cycle of couples that are "entangled" with each other.

Consider this scenario: Person 0 sits with Person 2, Person 1 sits with Person 3. Here, couples 0 and 1 are entangled. To fix this, we need exactly 1 swap - we can swap Person 1 and Person 2, resulting in couples sitting correctly.

Now imagine three couples are entangled: Person 0 sits with Person 2, Person 1 sits with Person 4, and Person 3 sits with Person 5. This forms a cycle of 3 couples, and we need exactly 2 swaps to fix all of them.

The pattern emerges: if `k` couples form a cycle where they're all mixed up with each other, we need exactly `k-1` swaps to arrange them correctly. Why? Because each swap fixes one couple and reduces the problem size by 1. When we're down to the last couple in the cycle, they're automatically in the right position after all previous swaps.

To identify these cycles, we can use Union-Find. As we scan through the seats pair by pair (seats 0-1, seats 2-3, etc.), we check which couples are currently sitting together. If Person `i` from couple A is sitting with Person `j` from couple B, we union couples A and B in our Union-Find structure. This builds up the connected components representing our cycles.

If we have `n` couples total and they form `x` connected components (cycles), then the total number of swaps needed is `(size_1 - 1) + (size_2 - 1) + ... + (size_x - 1) = n - x`.

This is why the solution uses Union-Find to group couples and returns `n - (number of connected components)`.

**Learn more about [Greedy](/problems/greedy_intro), [Depth-First Search](/problems/dfs_intro), [Breadth-First Search](/problems/bfs_intro), [Union Find](/problems/dsu_intro) and [Graph](/problems/graph_intro) patterns.**

## Solution Approach

The implementation uses Union-Find (Disjoint Set Union) to track which couples form connected components (cycles). Here's how the solution works step by step:

**1. Initialize Union-Find Structure**
```python
n = len(row) >> 1  # Number of couples (2n people means n couples)
p = list(range(n))  # Parent array for Union-Find, initially each couple is its own parent
```
We create a parent array `p` where `p[i] = i` initially, meaning each couple is in its own set.

**2. Map People to Couples**
Each person's couple number can be calculated as `person_id >> 1` (equivalent to `person_id // 2`):
- Person 0 and 1 → Couple 0
- Person 2 and 3 → Couple 1
- Person 4 and 5 → Couple 2

**3. Build Connected Components**
```python
for i in range(0, len(row), 2):
    a, b = row[i] >> 1, row[i + 1] >> 1
    p[find(a)] = find(b)
```
We iterate through seats in pairs (0-1, 2-3, 4-5, etc.). For each pair of adjacent seats:
- Get the couple numbers of the two people sitting together: `a = row[i] >> 1` and `b = row[i + 1] >> 1`
- Union these two couples using `p[find(a)] = find(b)`

This unions couples that have their members mixed up in the current seating arrangement.

**4. Find Operation with Path Compression**
```python
def find(x: int) -> int:
    if p[x] != x:
        p[x] = find(p[x])  # Path compression
    return p[x]
```
The find operation locates the root of a set and applies path compression for efficiency.

**5. Count Connected Components and Calculate Result**
```python
return n - sum(i == find(i) for i in range(n))
```
- Count how many couples are roots of their components: `sum(i == find(i) for i in range(n))`
- This gives us the number of connected components `x`
- Return `n - x`, which is the minimum number of swaps needed

**Why This Works:**
If `k` couples form a cycle (connected component), they need `k-1` swaps to all sit correctly. With `x` total components having sizes `y₁, y₂, ..., yₓ`, the total swaps needed is:
`(y₁ - 1) + (y₂ - 1) + ... + (yₓ - 1) = (y₁ + y₂ + ... + yₓ) - x = n - x`

The Union-Find efficiently groups the entangled couples and allows us to calculate the answer in O(n) time with nearly O(1) operations for union and find (with path compression).

## Example Walkthrough

Let's walk through the solution with `row = [0, 5, 4, 3, 2, 1]` (3 couples, 6 people).

**Step 1: Identify the couples**
- Couple 0: persons (0, 1)
- Couple 1: persons (2, 3)  
- Couple 2: persons (4, 5)

**Step 2: Check current seating pairs**
- Seats 0-1: Person 0 (Couple 0) sits with Person 5 (Couple 2)
- Seats 2-3: Person 4 (Couple 2) sits with Person 3 (Couple 1)
- Seats 4-5: Person 2 (Couple 1) sits with Person 1 (Couple 0)

Notice all three couples are entangled in a cycle!

**Step 3: Build Union-Find structure**
Initialize: `p = [0, 1, 2]` (each couple is its own parent)

Process seat pairs:
- Seats 0-1: Person 0 (Couple 0) and Person 5 (Couple 2) → Union couples 0 and 2
  - `find(0) = 0`, `find(2) = 2`
  - Set `p[0] = 2`
  - Now `p = [2, 1, 2]`

- Seats 2-3: Person 4 (Couple 2) and Person 3 (Couple 1) → Union couples 2 and 1
  - `find(2) = 2`, `find(1) = 1`
  - Set `p[2] = 1`
  - Now `p = [2, 1, 1]`

- Seats 4-5: Person 2 (Couple 1) and Person 1 (Couple 0) → Union couples 1 and 0
  - `find(1) = 1`, `find(0) = find(2) = find(1) = 1`
  - Already in same component, no change needed

**Step 4: Count connected components**
Check which couples are roots:
- Couple 0: `find(0) = 1`, not a root
- Couple 1: `find(1) = 1`, is a root ✓
- Couple 2: `find(2) = 1`, not a root

Number of components = 1

**Step 5: Calculate result**
Minimum swaps = n - components = 3 - 1 = 2

**Verification with actual swaps:**
1. Swap persons in seats 1 and 5: `[0, 1, 4, 3, 2, 5]` - Couple 0 is fixed!
2. Swap persons in seats 2 and 4: `[0, 1, 2, 3, 4, 5]` - All couples are fixed!

Indeed, we needed exactly 2 swaps, confirming our formula works correctly.

## Python Solution

```python
class Solution:
    def minSwapsCouples(self, row: List[int]) -> int:
        """
        Find minimum swaps needed to seat couples together.
        Uses Union-Find to group couples that are sitting in wrong positions.
      
        Args:
            row: List of integers representing people (couples are 0-1, 2-3, 4-5, etc.)
      
        Returns:
            Minimum number of swaps needed
        """
      
        def find_root(node: int) -> int:
            """
            Find the root of the set containing the given node.
            Implements path compression for optimization.
          
            Args:
                node: The node to find the root for
          
            Returns:
                The root node of the set
            """
            if parent[node] != node:
                # Path compression: make every node point directly to root
                parent[node] = find_root(parent[node])
            return parent[node]
      
        # Calculate number of couples (each couple consists of 2 people)
        num_couples = len(row) // 2
      
        # Initialize parent array for Union-Find
        # Each couple initially forms its own set
        parent = list(range(num_couples))
      
        # Process each seat pair (positions 0-1, 2-3, 4-5, etc.)
        for seat_index in range(0, len(row), 2):
            # Get couple IDs for the two people sitting together
            # Person i belongs to couple i//2
            couple_a = row[seat_index] // 2
            couple_b = row[seat_index + 1] // 2
          
            # Union: Connect the two couples in the same component
            # If they're from different couples, they need to be swapped
            root_a = find_root(couple_a)
            root_b = find_root(couple_b)
            parent[root_a] = root_b
      
        # Count number of connected components
        # Each component requires (size - 1) swaps to fix
        num_components = sum(1 for couple_id in range(num_couples) 
                           if couple_id == find_root(couple_id))
      
        # Total swaps = total couples - number of components
        return num_couples - num_components
```

## Time and Space Complexity

**Time Complexity:** `O(n × α(n))`

The algorithm uses a Union-Find (Disjoint Set Union) data structure with path compression. The main operations are:
- Iterating through the array once: `O(n)` where `n` is the length of the row array
- For each pair of elements (n/2 pairs total), performing two `find` operations and one union operation
- The `find` operation with path compression has an amortized time complexity of `O(α(n))`, where `α(n)` is the inverse Ackermann function
- The final sum comprehension iterates through n/2 elements, each calling `find` once: `O(n × α(n))`

Since `α(n)` grows extremely slowly and is effectively constant for all practical values of `n` (it's less than 5 for all values of `n` less than 2^65536), the overall time complexity is `O(n × α(n))`.

**Space Complexity:** `O(n)`

The space usage includes:
- The parent array `p` of size `n/2`: `O(n)`
- The recursion stack for the `find` function, which can reach `O(n)` depth on a single call when the parent chain is long (path compression flattens the chain afterward, but does not bound the depth of that first call)
- Variables `a`, `b`, and loop counters: `O(1)`

The dominant factor is the parent array, giving us a total space complexity of `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Couple ID Calculation**
A common mistake is using modulo operation (`%`) instead of integer division to determine couple IDs. Some might incorrectly think that couples are paired as (0,1), (1,2), (2,3), etc., or try to use `person_id % 2` to find partners.

**Wrong approach:**
```python
# Incorrect: This doesn't properly identify couples
couple_id = row[i] % 2  # Wrong!
partner = row[i] + 1 if row[i] % 2 == 0 else row[i] - 1  # Overcomplicated
```

**Correct approach:**
```python
# Each couple ID is simply person_id // 2 (or person_id >> 1)
couple_id = row[i] // 2
# Person 0,1 → Couple 0; Person 2,3 → Couple 1; etc.
```

### 2. **Forgetting Path Compression in Union-Find**
Without path compression, the find operation can degrade to O(n) in the worst case, making the overall algorithm inefficient for large inputs.

**Inefficient version:**
```python
def find(x):
    while parent[x] != x:
        x = parent[x]  # No path compression
    return x
```

**Optimized version:**
```python
def find(x):
    if parent[x] != x:
        parent[x] = find(parent[x])  # Path compression
    return parent[x]
```

### 3. **Misunderstanding the Problem's Seat Pairing**
Some might think any two consecutive people in the array should be couples, but the problem specifically states that seats are paired as (0,1), (2,3), (4,5), etc. This means we should only check pairs at even indices.

**Wrong iteration:**
```python
# Incorrect: Checking every consecutive pair
for i in range(len(row) - 1):
    couple_a = row[i] // 2
    couple_b = row[i + 1] // 2
    # This would check pairs like (1,2), (3,4) which aren't seat pairs
```

**Correct iteration:**
```python
# Check only seat pairs: (0,1), (2,3), (4,5), etc.
for i in range(0, len(row), 2):
    couple_a = row[i] // 2
    couple_b = row[i + 1] // 2
```

### 4. **Counting Components Before All Unions Are Done**
There are two correct ways to count components. The solution performs all unions first and then counts the couples that are their own root. Another correct option starts with `n` components and subtracts one each time a union joins two different sets. The mistake is to count roots partway through, before every seat pair has been processed, because later unions can still merge components.

**Wrong timing:**
```python
for i in range(0, len(row), 2):
    parent[find(row[i] // 2)] = find(row[i + 1] // 2)
    if i == 0:
        # Wrong: counts roots after only the first union
        components = sum(1 for c in range(n) if c == find(c))
```

**Correct approaches:**
```python
# Option 1: perform all unions, then count roots
for i in range(0, len(row), 2):
    parent[find(row[i] // 2)] = find(row[i + 1] // 2)
components = sum(1 for c in range(n) if c == find(c))

# Option 2: start with n components and subtract on each successful merge
components = n
for i in range(0, len(row), 2):
    root_a, root_b = find(row[i] // 2), find(row[i + 1] // 2)
    if root_a != root_b:
        parent[root_a] = root_b
        components -= 1
```

### 5. **Not Handling Edge Cases**
While the problem guarantees valid input, in practice you might want to validate:
- Empty array or array with odd length
- Invalid person IDs (negative or >= 2n)
- Duplicate person IDs

**Robust version with validation:**
```python
def minSwapsCouples(self, row: List[int]) -> int:
    if not row or len(row) % 2 != 0:
        return 0  # or raise an exception
  
    n = len(row) // 2
    if len(set(row)) != len(row):  # Check for duplicates
        raise ValueError("Duplicate person IDs found")
  
    # Continue with main algorithm...
```

