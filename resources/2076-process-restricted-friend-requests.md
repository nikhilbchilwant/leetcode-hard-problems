# 2076. Process Restricted Friend Requests

> Source: https://algo.monster/liteproblems/2076

## Problem Statement

You have a network of `n` people labeled from `0` to `n - 1`. Initially, no one is friends with each other.

You're given two lists:
1. **Restrictions**: A list of pairs `[xi, yi]` indicating that person `xi` and person `yi` cannot become friends, either directly or indirectly through other people. This means if `xi` becomes friends with someone who is friends with `yi` (through any chain of friendships), it violates the restriction.

2. **Requests**: A list of friend requests `[uj, vj]` that need to be processed in order.

For each friend request, you need to determine if it can be accepted:
- A request is **successful** if accepting it doesn't violate any restriction
- If two people are already friends (directly or through a chain), the request is still considered successful
- When a request is successful, the two people become direct friends

The key challenge is that when two people become friends, they join each other's friend groups. This means all friends of person `u` become indirectly connected to all friends of person `v`. You must ensure this merger doesn't cause any pair from the restrictions list to become connected.

For example, if person 0 and person 1 are restricted from being friends, and person 0 is already friends with person 2, then a friend request between person 1 and person 2 cannot be accepted because it would indirectly connect person 0 and person 1.

You need to return a boolean array where each element indicates whether the corresponding friend request was successful (`true`) or not (`false`).

## Intuition

The core insight is that we need to track connected components of friends, as friendship is transitive - if A is friends with B, and B is friends with C, then A and C are in the same friend group.

This immediately suggests using a Union-Find (Disjoint Set Union) data structure, which efficiently manages connected components and can quickly tell us if two people are in the same group.

For each friend request between persons `u` and `v`, we face two scenarios:

1. **They're already in the same friend group**: The request is automatically successful since they're already connected through some chain of friendships.

2. **They're in different friend groups**: This is where we need to be careful. Accepting this request would merge two entire friend groups together. We must check if this merger would violate any restrictions.

The key realization is that when we merge groups containing `u` and `v`, every person in `u`'s group becomes connected to every person in `v`'s group. So for each restriction `[x, y]`, we need to verify that we're not about to connect them.

How do we check this? For each restriction `[x, y]`, we find which groups `x` and `y` belong to (using the find operation). If `x` is in the same group as `u` and `y` is in the same group as `v` (or vice versa), then accepting the friend request would connect `x` and `y`, violating the restriction.

If no restrictions are violated, we can safely merge the two groups by making one group's root point to the other's root (`p[pu] = pv`), effectively combining all members of both groups into one larger friend group.

This approach works because Union-Find gives us constant-time group identification (with path compression) and efficient merging of groups, while we only need to check each restriction once per request.

**Learn more about [Union Find](/problems/dsu_intro) and [Graph](/problems/graph_intro) patterns.**

## Solution Approach

The solution uses the Union-Find data structure to efficiently manage friend groups and check restrictions.

**Initialization:**
- Create a parent array `p` where `p[i] = i` initially, meaning each person is their own representative (root of their group)
- Initialize an empty answer list to store results for each request

**Find Operation with Path Compression:**
```python
def find(x: int) -> int:
    if p[x] != x:
        p[x] = find(p[x])  # Path compression
    return p[x]
```
This function finds the root representative of a person's group. Path compression optimizes future lookups by making nodes point directly to the root.

**Processing Each Friend Request `[u, v]`:**

1. **Find the roots** of both people:
   ```python
   pu, pv = find(u), find(v)
   ```

2. **Check if already friends:**
   ```python
   if pu == pv:
       ans.append(True)
   ```
   If they have the same root, they're already in the same friend group, so the request is automatically successful.

3. **Check restrictions** if they're in different groups:
   - Initialize `ok = True` to track if the request can be accepted
   - For each restriction `[x, y]`:
     ```python
     px, py = find(x), find(y)
     if (pu == px and pv == py) or (pu == py and pv == px):
         ok = False
         break
     ```
   - This checks if accepting the request would connect restricted pairs:
     - `pu == px and pv == py`: Person `x` is in `u`'s group and person `y` is in `v`'s group
     - `pu == py and pv == px`: Person `y` is in `u`'s group and person `x` is in `v`'s group
   - Either condition means merging the groups would violate the restriction

4. **Union operation** if no restrictions are violated:
   ```python
   if ok:
       p[pu] = pv
   ```
   This merges the two groups by making one root point to the other.

5. **Record the result:**
   ```python
   ans.append(ok)
   ```

**Time Complexity:**
- For each request: `O(r)` where `r` is the number of restrictions (we check all restrictions in worst case)
- With path compression, find operations are nearly `O(1)` amortized
- Total: `O(m * r)` where `m` is the number of requests

**Space Complexity:** `O(n)` for the parent array

The elegance of this solution lies in how Union-Find naturally handles the transitive nature of friendships while making it easy to check if a merger would violate restrictions.

## Example Walkthrough

Let's walk through a concrete example to illustrate how the solution works.

**Given:**
- `n = 5` (people labeled 0 to 4)
- Restrictions: `[[0, 1], [1, 2], [3, 4]]`
- Requests: `[[0, 4], [1, 3], [3, 2]]`

**Initial State:**
- Parent array: `p = [0, 1, 2, 3, 4]` (everyone is their own root)
- Each person is in their own group

**Processing Request 1: [0, 4]**
- Find roots: `find(0) = 0`, `find(4) = 4` (different groups)
- Check restrictions:
  - `[0, 1]`: `find(0) = 0`, `find(1) = 1`
    - Is `(0 == 0 and 4 == 1)` or `(0 == 1 and 4 == 0)`? No ✓
  - `[1, 2]`: `find(1) = 1`, `find(2) = 2`
    - Is `(0 == 1 and 4 == 2)` or `(0 == 2 and 4 == 1)`? No ✓
  - `[3, 4]`: `find(3) = 3`, `find(4) = 4`
    - Is `(0 == 3 and 4 == 4)` or `(0 == 4 and 4 == 3)`? No ✓
- No violations found, so merge: `p[0] = 4`
- Parent array: `p = [4, 1, 2, 3, 4]`
- Groups: {0, 4}, {1}, {2}, {3}
- Result: `true`

**Processing Request 2: [1, 3]**
- Find roots: `find(1) = 1`, `find(3) = 3` (different groups)
- Check restrictions:
  - `[0, 1]`: `find(0) = 4`, `find(1) = 1`
    - Is `(1 == 4 and 3 == 1)` or `(1 == 1 and 3 == 4)`? No ✓
  - `[1, 2]`: `find(1) = 1`, `find(2) = 2`
    - Is `(1 == 1 and 3 == 2)` or `(1 == 2 and 3 == 1)`? No ✓
  - `[3, 4]`: `find(3) = 3`, `find(4) = 4`
    - Is `(1 == 3 and 3 == 4)` or `(1 == 4 and 3 == 3)`? No ✓
- No violations found, so merge: `p[1] = 3`
- Parent array: `p = [4, 3, 2, 3, 4]`
- Groups: {0, 4}, {1, 3}, {2}
- Result: `true`

**Processing Request 3: [3, 2]**
- Find roots: `find(3) = 3`, `find(2) = 2` (different groups)
- Check restrictions:
  - `[0, 1]`: `find(0) = 4`, `find(1) = 3`
    - Is `(3 == 4 and 2 == 3)` or `(3 == 3 and 2 == 4)`? No ✓
  - `[1, 2]`: `find(1) = 3`, `find(2) = 2`
    - Is `(3 == 3 and 2 == 2)` or `(3 == 2 and 2 == 3)`? 
    - First condition is TRUE! (3 == 3 and 2 == 2) ✗
- Violation detected! Person 1 (in group with root 3) would connect to person 2
- Cannot merge these groups
- Result: `false`

**Final Answer:** `[true, true, false]`

The key insight: When checking request [3, 2], we found that person 1 is already in person 3's group (from the previous request). Accepting this request would put person 2 in the same group, violating the restriction [1, 2] that says persons 1 and 2 cannot be friends directly or indirectly.

## Python Solution

```python
class Solution:
    def friendRequests(
        self, n: int, restrictions: List[List[int]], requests: List[List[int]]
    ) -> List[bool]:
        """
        Process friend requests with restrictions using Union-Find.
      
        Args:
            n: Number of people (0 to n-1)
            restrictions: List of pairs that cannot be in the same friend group
            requests: List of friend request pairs to process
          
        Returns:
            List of booleans indicating whether each request was approved
        """
      
        def find(node: int) -> int:
            """
            Find the root parent of a node with path compression.
          
            Args:
                node: The node to find the root for
              
            Returns:
                The root parent of the node
            """
            if parent[node] != node:
                # Path compression: make node point directly to root
                parent[node] = find(parent[node])
            return parent[node]
      
        # Initialize parent array where each person is their own parent initially
        parent = list(range(n))
      
        # Store results for each request
        result = []
      
        # Process each friend request
        for person_u, person_v in requests:
            # Find the root parents of both people
            parent_u = find(person_u)
            parent_v = find(person_v)
          
            if parent_u == parent_v:
                # Already in the same friend group, request automatically approved
                result.append(True)
            else:
                # Check if merging these groups would violate any restriction
                is_valid = True
              
                for restricted_x, restricted_y in restrictions:
                    # Find parents of the restricted pair
                    parent_x = find(restricted_x)
                    parent_y = find(restricted_y)
                  
                    # Check if merging would put restricted pair in same group
                    if (parent_u == parent_x and parent_v == parent_y) or \
                       (parent_u == parent_y and parent_v == parent_x):
                        is_valid = False
                        break
              
                result.append(is_valid)
              
                # If valid, perform the union operation
                if is_valid:
                    parent[parent_u] = parent_v
      
        return result
```

## Time and Space Complexity

**Time Complexity:** `O(q × m × α(n))` where `q` is the number of requests, `m` is the number of restrictions, and `α(n)` is the inverse Ackermann function.

The algorithm uses a Union-Find (Disjoint Set Union) data structure with path compression. For each of the `q` requests:
- We perform two `find()` operations to get `pu` and `pv`, which takes `O(α(n))` time each with path compression
- We iterate through all `m` restrictions, and for each restriction, we perform two `find()` operations to get `px` and `py`, taking `O(α(n))` time each
- If the request is approved, we perform one union operation by setting `p[pu] = pv`, which takes `O(1)` time

Therefore, for each request, the time complexity is `O(1 × α(n) + m × α(n)) = O(m × α(n))`. With `q` requests total, the overall time complexity is `O(q × m × α(n))`.

Note: The reference answer states `O(q × m × log(n))`, which would be accurate if using Union-Find without path compression. With path compression (as implemented in the code with `p[x] = find(p[x])`), the amortized time complexity for `find()` operations is `O(α(n))`, where `α(n)` is the inverse Ackermann function, which is effectively constant for all practical values of `n`.

**Space Complexity:** `O(n)`

The space is used for:
- The parent array `p` of size `n`: `O(n)`
- The answer list `ans` which stores boolean values for each request: `O(q)`
- Recursive call stack for `find()` function in worst case (before path compression): `O(n)`

Since typically `q ≤ n` in practical scenarios and we're looking at the dominant term, the overall space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Not Using Find() When Checking Restrictions**

**The Pitfall:**
A critical mistake is directly using the parent array instead of the `find()` function when checking restrictions. For example:

```python
# WRONG - This will fail after path compression changes intermediate nodes
for restricted_x, restricted_y in restrictions:
    if (parent[person_u] == parent[restricted_x] and parent[person_v] == parent[restricted_y]) or \
       (parent[person_u] == parent[restricted_y] and parent[person_v] == parent[restricted_x]):
        is_valid = False
        break
```

**Why It Fails:**
The parent array doesn't always directly point to the root. After path compression, intermediate nodes may point to ancestors but not necessarily the root. This leads to incorrect group membership checks.

**The Solution:**
Always use `find()` to get the actual root representative:

```python
# CORRECT - Always use find() to get the true root
parent_x = find(restricted_x)
parent_y = find(restricted_y)

if (parent_u == parent_x and parent_v == parent_y) or \
   (parent_u == parent_y and parent_v == parent_x):
    is_valid = False
    break
```

### 2. **Performing Union Before Checking All Restrictions**

**The Pitfall:**
Merging groups immediately upon finding no direct restriction between the requested pair:

```python
# WRONG - Merging too early
parent_u = find(person_u)
parent_v = find(person_v)

if parent_u != parent_v:
    parent[parent_u] = parent_v  # Union performed here
  
    # Then checking restrictions - but groups are already merged!
    for restricted_x, restricted_y in restrictions:
        # This check is meaningless now
```

**Why It Fails:**
Once the union is performed, you can't undo it. If you later discover a restriction violation, the data structure is already corrupted.

**The Solution:**
First check ALL restrictions, then perform union only if valid:

```python
# CORRECT - Check all restrictions first
is_valid = True
for restricted_x, restricted_y in restrictions:
    # Check restrictions without modifying the structure
    parent_x = find(restricted_x)
    parent_y = find(restricted_y)
  
    if (parent_u == parent_x and parent_v == parent_y) or \
       (parent_u == parent_y and parent_v == parent_x):
        is_valid = False
        break

# Only merge if all checks pass
if is_valid:
    parent[parent_u] = parent_v
```

### 3. **Forgetting to Handle Already-Connected Case**

**The Pitfall:**
Not checking if two people are already in the same group before validating restrictions:

```python
# WRONG - Missing the already-friends check
parent_u = find(person_u)
parent_v = find(person_v)

# Immediately checking restrictions without considering they might already be friends
is_valid = True
for restricted_x, restricted_y in restrictions:
    # ... restriction checks
```

**Why It Fails:**
If two people are already friends, the request should automatically succeed without checking restrictions (since no new connections are being made).

**The Solution:**
Check for existing friendship first:

```python
# CORRECT - Handle already-friends case separately
parent_u = find(person_u)
parent_v = find(person_v)

if parent_u == parent_v:
    result.append(True)  # Already friends, automatic success
else:
    # Only check restrictions if they're in different groups
    is_valid = True
    for restricted_x, restricted_y in restrictions:
        # ... restriction checks
```

These pitfalls are particularly dangerous because they may work correctly for simple test cases but fail on more complex scenarios with multiple groups and chained friendships.

