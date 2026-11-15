# 3569. Maximize Count of Distinct Primes After Split

> Source: https://leetcode.com/problems/maximize-count-of-distinct-primes-after-split/

> AlgoMonster does not have a page for this problem number; this file contains the LeetCode statement only.

**Difficulty:** Hard  
**LeetCode topics:** Array, Hash Table, Math, Segment Tree, Number Theory, Ordered Set

## Problem Statement

You are given an integer array `nums` having length `n` and a 2D integer array `queries` where `queries[i] = [idx, val]`.

For each query:

	- Update `nums[idx] = val`.

	- Choose an integer `k` with `1 prime values in each part is **maximum**.

**Note:** The changes made to the array in one query persist into the next query.

Return an array containing the result for each query, in the order they are given.

 

**Example 1:**

**Input:** nums = [2,1,3,1,2], queries = [[1,2],[3,3]]

**Output:** [3,4]

**Explanation:**

	- Initially `nums = [2, 1, 3, 1, 2]`.

	- After 1st query, `nums = [2, 2, 3, 1, 2]`. Split `nums` into `[2]` and `[2, 3, 1, 2]`. `[2]` consists of 1 distinct prime and `[2, 3, 1, 2]` consists of 2 distinct primes. Hence, the answer for this query is `1 + 2 = 3`.

	- After 2nd query, `nums = [2, 2, 3, 3, 2]`. Split `nums` into `[2, 2, 3]` and `[3, 2]` with an answer of `2 + 2 = 4`.

	- The output is `[3, 4]`.

**Example 2:**

**Input:** nums = [2,1,4], queries = [[0,1]]

**Output:** [0]

**Explanation:**

	- Initially `nums = [2, 1, 4]`.

	- After 1st query, `nums = [1, 1, 4]`. There are no prime numbers in `nums`, hence the answer for this query is 0.

	- The output is `[0]`.

 

**Constraints:**

	- `2 4`

	- `1 4`

	- `1 5`

	- `0
