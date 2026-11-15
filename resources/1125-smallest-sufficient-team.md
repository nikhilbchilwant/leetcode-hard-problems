# 1125. Smallest Sufficient Team

> Source: https://algo.monster/liteproblems/1125

## Problem Statement

You are given a list of required skills `req_skills` and a list of people. Each person `people[i]` has a list of skills they possess.

Your task is to form a **sufficient team** - a set of people such that every skill in `req_skills` is covered by at least one person in the team. The team should be represented by the indices of the people selected.

For example:
- If `req_skills = ["java", "nodejs", "reactjs"]`
- And `people = [["java"], ["nodejs"], ["nodejs", "reactjs"]]`
- Then `team = [0, 2]` would be a sufficient team because person 0 has "java" and person 2 has both "nodejs" and "reactjs", covering all required skills

The goal is to find the **smallest possible team** that covers all required skills. You need to return the indices of the people in this minimal team. The answer can be returned in any order.

Key points:
- Every skill in `req_skills` must be covered by at least one team member
- One person can contribute multiple skills to the team
- You want to minimize the total number of people in the team
- The solution is guaranteed to exist
- Return the indices (positions) of the selected people, not their skill lists

## Intuition

The key observation is that we need to track which skills are covered as we build our team. Since there are at most 16 skills, we can represent any combination of skills as a binary number where each bit indicates whether a specific skill is covered or not. For example, if we have 3 skills, `101` in binary means the 1st and 3rd skills are covered.

This naturally leads to a [dynamic programming](/problems/dynamic_programming_intro) approach where our state is the set of skills currently covered. We can think of this as: "What's the minimum number of people needed to cover a specific set of skills?"

Starting from an empty skill set (0 in binary), we can progressively build up to covering all skills by considering adding each person one at a time. When we add a person, they contribute their skills to our current skill set using the bitwise OR operation.

The challenge is that we also need to track **which people** form the optimal team, not just the count. So alongside tracking the minimum number of people for each skill set, we need to remember:
- Who was the last person added to achieve this skill set
- What was the previous skill set before adding this person

This allows us to backtrack from the final state (all skills covered) to reconstruct the actual team members.

The reason this works efficiently is that:
1. The number of possible skill sets is bounded by `2^m` where `m ≤ 16`, making it computationally feasible
2. Each person's skills can be pre-computed as a [bitmask](/problems/bitmask_intro) for quick combination with existing skill sets
3. We only update a state when we find a better (smaller) team to achieve it, ensuring optimality

By the time we've processed all possible combinations, `f[2^m - 1]` (all skills covered) will contain the minimum team size, and we can trace back through our recorded transitions to find exactly which people form this optimal team.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) and [Bitmask](/problems/bitmask_intro) patterns.**

## Solution Approach

First, we create a mapping dictionary `d` where each skill in `req_skills` gets mapped to its index position. This allows us to quickly convert skills to bit positions.

Next, we preprocess each person's skills into a [bitmask](/problems/bitmask_intro). For each person `i`, we create `p[i]` which represents all their skills as a single integer. If person `i` has skill `s`, we set the bit at position `d[s]` using `p[i] |= 1 << d[s]`.

We then initialize three arrays for our [dynamic programming](/problems/dynamic_programming_intro):
- `f[i]`: Minimum number of people needed to cover skill set `i`. Initialize all to infinity except `f[0] = 0` (no people needed for no skills)
- `g[i]`: The index of the last person added when achieving skill set `i` with minimum people
- `h[i]`: The previous skill set state before adding the last person to achieve state `i`

The core DP transition works as follows:
1. Iterate through all possible skill sets from `0` to `(1 << m) - 1`
2. For each valid state `i` (where `f[i] != inf`), try adding each person `j`
3. The new state after adding person `j` is `i | p[j]` (combining current skills with person j's skills)
4. If `f[i] + 1 < f[i | p[j]]`, we found a better way to achieve state `i | p[j]`:
   - Update `f[i | p[j]] = f[i] + 1` (one more person than state `i`)
   - Record `g[i | p[j]] = j` (person `j` was added last)
   - Record `h[i | p[j]] = i` (came from state `i`)

After filling the DP table, we reconstruct the answer by backtracking:
1. Start from the final state `i = (1 << m) - 1` (all skills covered)
2. Add `g[i]` (the last person for this state) to our answer
3. Move to the previous state: `i = h[i]`
4. Repeat until `i = 0`

This backtracking gives us the indices of all people in the minimum sufficient team.

The time complexity is `O(2^m * n)` where `m` is the number of required skills and `n` is the number of people. The space complexity is `O(2^m)` for the DP arrays.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Input:**
- `req_skills = ["java", "nodejs", "reactjs"]`
- `people = [["java"], ["nodejs"], ["nodejs", "reactjs"]]`

**Step 1: Create skill mapping**
```
d = {"java": 0, "nodejs": 1, "reactjs": 2}
```

**Step 2: Convert people's skills to bitmasks**
- Person 0 has ["java"]: `p[0] = 001` (bit 0 set)
- Person 1 has ["nodejs"]: `p[1] = 010` (bit 1 set)  
- Person 2 has ["nodejs", "reactjs"]: `p[2] = 110` (bits 1 and 2 set)

**Step 3: Initialize DP arrays**
We have 2^3 = 8 possible skill states (000 to 111 in binary).
```
f = [0, inf, inf, inf, inf, inf, inf, inf]  // minimum people needed
g = [-1, -1, -1, -1, -1, -1, -1, -1]        // last person added
h = [-1, -1, -1, -1, -1, -1, -1, -1]        // previous state
```

**Step 4: DP transitions**

Starting from state 000 (no skills):
- Add person 0: 000 | 001 = 001
  - `f[001] = 1`, `g[001] = 0`, `h[001] = 000`
- Add person 1: 000 | 010 = 010
  - `f[010] = 1`, `g[010] = 1`, `h[010] = 000`
- Add person 2: 000 | 110 = 110
  - `f[110] = 1`, `g[110] = 2`, `h[110] = 000`

From state 001 (java covered):
- Add person 1: 001 | 010 = 011
  - `f[011] = 2`, `g[011] = 1`, `h[011] = 001`
- Add person 2: 001 | 110 = 111
  - `f[111] = 2`, `g[111] = 2`, `h[111] = 001`

From state 010 (nodejs covered):
- Add person 0: 010 | 001 = 011
  - `f[011]` is already 2, no update
- Add person 2: 010 | 110 = 110
  - `f[110]` is already 1, no update

From state 110 (nodejs, reactjs covered):
- Add person 0: 110 | 001 = 111
  - `f[111]` is already 2, no update

After processing all transitions:
```
State  Binary  f    g    h
0      000     0    -1   -1
1      001     1    0    0
2      010     1    1    0
3      011     2    1    1
4      100     inf  -1   -1
5      101     inf  -1   -1
6      110     1    2    0
7      111     2    2    1
```

**Step 5: Backtrack from state 111 (all skills covered)**
- Start at state 111: Add person `g[111] = 2` to team
- Move to state `h[111] = 001`
- At state 001: Add person `g[001] = 0` to team
- Move to state `h[001] = 000`
- State 000 is our base case, stop

**Result:** Team = [2, 0] (or [0, 2] when reversed)

This team covers all skills: Person 0 has "java", Person 2 has "nodejs" and "reactjs".

## Python Solution

```python
class Solution:
    def smallestSufficientTeam(
        self, req_skills: List[str], people: List[List[str]]
    ) -> List[int]:
        # Create a mapping from skill name to its bit position
        skill_to_bit = {skill: bit_pos for bit_pos, skill in enumerate(req_skills)}
      
        num_skills = len(req_skills)
        num_people = len(people)
      
        # Convert each person's skills to a bitmask representation
        # person_skillmasks[i] represents all skills of person i as a bitmask
        person_skillmasks = [0] * num_people
        for person_idx, person_skills in enumerate(people):
            for skill in person_skills:
                person_skillmasks[person_idx] |= 1 << skill_to_bit[skill]
      
        # DP arrays for tracking the smallest team
        # min_team_size[mask] = minimum number of people needed to cover skills represented by mask
        min_team_size = [float('inf')] * (1 << num_skills)
      
        # last_person[mask] = the last person added to achieve this skill mask
        last_person = [0] * (1 << num_skills)
      
        # prev_mask[mask] = the previous skill mask before adding last_person[mask]
        prev_mask = [0] * (1 << num_skills)
      
        # Base case: 0 people needed to cover 0 skills
        min_team_size[0] = 0
      
        # Try all possible skill combinations
        for current_mask in range(1 << num_skills):
            # Skip if this mask is not reachable
            if min_team_size[current_mask] == float('inf'):
                continue
          
            # Try adding each person to the current team
            for person_idx in range(num_people):
                # Calculate new skill mask after adding this person
                new_mask = current_mask | person_skillmasks[person_idx]
              
                # If adding this person results in a smaller team for new_mask
                if min_team_size[current_mask] + 1 < min_team_size[new_mask]:
                    min_team_size[new_mask] = min_team_size[current_mask] + 1
                    last_person[new_mask] = person_idx
                    prev_mask[new_mask] = current_mask
      
        # Reconstruct the team by backtracking from the full skill mask
        result_team = []
        current_mask = (1 << num_skills) - 1  # All skills covered
      
        while current_mask:
            result_team.append(last_person[current_mask])
            current_mask = prev_mask[current_mask]
      
        return result_team
```

## Time and Space Complexity

**Time Complexity:** `O(2^m × n)`

The algorithm uses dynamic programming with bitmask states. The outer loop iterates through all possible skill combinations, which is `2^m` states (where `m` is the number of required skills). For each valid state `i`, the inner loop iterates through all `n` people to try adding each person to the current team. Therefore, the total time complexity is `O(2^m × n)`.

**Space Complexity:** `O(2^m)`

The algorithm maintains three arrays (`f`, `g`, and `h`), each of size `2^m` to store:
- `f[i]`: minimum number of people needed to achieve skill set `i`
- `g[i]`: the last person added to achieve skill set `i`
- `h[i]`: the previous state before adding the last person

Additionally, the array `p` of size `n` stores the skill bitmask for each person, but this is `O(n)` which is dominated by `O(2^m)` in most practical cases. The overall space complexity is `O(2^m)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Redundant State Transitions and Performance Issues**

A critical pitfall in this solution is that it processes **every possible person for every reachable state**, even when adding a person doesn't contribute any new skills. This creates unnecessary iterations and can significantly slow down the algorithm.

**The Problem:**
- If a person's skills are already fully covered by the current mask (i.e., `person_skillmasks[person_idx] & current_mask == person_skillmasks[person_idx]`), then `new_mask` will equal `current_mask`
- The algorithm still performs the comparison and update checks, wasting computational resources
- For large inputs with many people having overlapping skills, this becomes a performance bottleneck

**Solution:**
Add a check to skip persons who don't contribute new skills:

```python
for person_idx in range(num_people):
    # Skip if this person doesn't add any new skills
    if person_skillmasks[person_idx] & ~current_mask == 0:
        continue
  
    # Calculate new skill mask after adding this person
    new_mask = current_mask | person_skillmasks[person_idx]
  
    # Rest of the logic remains the same...
```

### 2. **Memory Optimization Opportunity Missed**

Another pitfall is using three separate arrays (`min_team_size`, `last_person`, `prev_mask`) when a single dictionary could be more memory-efficient for sparse cases.

**The Problem:**
- When the number of required skills is large (e.g., 15-16), the arrays allocate `2^m` elements even if many states are never reached
- This wastes memory for impossible or unreachable skill combinations

**Solution:**
Use a dictionary-based approach for sparse state representation:

```python
# Instead of arrays, use dictionaries
dp = {0: 0}  # skill_mask -> min_team_size
parent = {}   # skill_mask -> (prev_mask, person_added)

for current_mask in list(dp.keys()):
    for person_idx in range(num_people):
        new_mask = current_mask | person_skillmasks[person_idx]
      
        if new_mask not in dp or dp[current_mask] + 1 < dp[new_mask]:
            dp[new_mask] = dp[current_mask] + 1
            parent[new_mask] = (current_mask, person_idx)
```

### 3. **Edge Case: Empty Required Skills**

The code doesn't explicitly handle the case where `req_skills` is empty, which could lead to unexpected behavior.

**The Problem:**
- If `req_skills = []`, then `num_skills = 0` and `(1 << num_skills) - 1 = 0`
- The reconstruction loop `while current_mask:` would immediately exit, returning an empty team
- While technically correct, it might be worth adding explicit handling for clarity

**Solution:**
Add an early return for edge cases:

```python
def smallestSufficientTeam(self, req_skills, people):
    if not req_skills:
        return []
  
    # Rest of the implementation...
```

