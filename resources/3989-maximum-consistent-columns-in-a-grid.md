# 3989. Maximum Consistent Columns in a Grid

> Source: https://leetcode.com/problems/maximum-consistent-columns-in-a-grid/

> AlgoMonster does not have a page for this problem number; this file contains the LeetCode statement only.

**Difficulty:** Hard  
**LeetCode topics:** Array, Dynamic Programming, Matrix

## Problem Statement

You are given a 2D integer array `grid` of size `m x n`, and an integer `limit`.

You may remove zero or more columns from the grid, but at least one column must remain. The **relative** order of the remaining columns must be preserved.

A grid is called **consistent** if for every row `i`, and for every pair of adjacent remaining columns `a` and `b` with `a Return the **maximum** number of columns that can remain such that the resulting grid is **consistent**.

 

**Example 1:**

**Input:** grid = [[-2,0,3]], limit = 2

**Output:** 2

**Explanation:**

	- Remove column 2 and keep columns 0 and 1, which gives `|grid[0][1] − grid[0][0]| = |0 − (−2)| = 2 

**Example 2:**

**Input:** grid = [[1,-1,1],[2,2,2]], limit = 1

**Output:** 2

**Explanation:**

	- Remove column 1 and keep columns 0 and 2, which gives
	
		- `|grid[0][2] − grid[0][0]| = |1 − 1| = 0 
	

	- Thus, the maximum number of columns that can remain is 2.

**Example 3:**

**Input:** grid = [[-5,5]], limit = 9

**Output:** 1

**Explanation:**

	- Remove either column 0 or column 1, since `|grid[0][1] − grid[0][0]| = |5 − (−5)| = 10 > limit`.

	- Thus, the maximum number of columns that can remain is 1.

 

**Constraints:**

	- `1
