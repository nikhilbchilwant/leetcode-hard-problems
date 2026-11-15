# 4059. Lexicographically Largest Power Array

> Source: https://leetcode.com/problems/lexicographically-largest-power-array/

> AlgoMonster does not have a page for this problem number; this file contains the LeetCode statement only.

**Difficulty:** Hard  
**LeetCode topics:** Array, Greedy, Bit Manipulation

## Problem Statement

You are given an integer array `nums` of length `n`. You may rearrange its elements to form any permutation `perm`.

Define an array `power` of length 15. For each `0 set.

Bit positions are indexed from right to left, starting with the `0th` bit.

Return the lexicographically largest possible `power` array.

 

**Example 1:**

**Input:** nums = [7,5]

**Output:** [0,0,0,0,0,0,0,0,0,0,0,0,2,1,2]

**Explanation:**

Choose `perm = [7, 5]`.

	- Both elements have bit 2 set, so `power[12] = 2`.

	- The first element has bit 1 set, but the second does not, so `power[13] = 1`.

	- Both elements have bit 0 set, so `power[14] = 2`.

All higher bits are unset in the first element, so the remaining entries are 0.

**Example 2:**

**Input:** nums = [3,1,7]

**Output:** [0,0,0,0,0,0,0,0,0,0,0,0,1,2,3]

**Explanation:**

Choose `perm = [7, 3, 1]`.

	- The first element has bit 2 set, but the second does not, so `power[12] = 1`.

	- The first two elements have bit 1 set, but the third does not, so `power[13] = 2`.

	- All three elements have bit 0 set, so `power[14] = 3`.

All higher bits are unset in the first element, so the remaining entries are 0.

 

**Constraints:**

	- `1 4`

	- `0 15`
