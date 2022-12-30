---
title: LeetCode 筆記｜1480. Running Sum of 1d Array
date: 2022-12-19 21:08:37
tags: array
categories: [programming, leetcode]
---
計算數列逐項的累積和 (runningSum)

<!--more-->

## 問題描述
> Given an array nums. We define a running sum of an array as runningSum[i] = sum(nums[0]…nums[i]).
> Return the running sum of nums.

**樣例**
> Input: nums = [1,2,3,4]
> Output: [1,3,6,10]

**限制**
- `1 <= nums.length <= 1000`
- `-10^6 <= nums[i] <= 10^6`

**知識點**
prefix sum & array  

## 思路與題解
此題應該算是 for loop 練習，我覺得要留意 (1) indices out of range 以及 (2) len(nums) == 1 的狀況。
```python
class Solution:
    def runningSum(self, nums: List[int]) -> List[int]:
        for n in range(1, len(nums)):
            nums[n] = nums[n - 1] + nums[n]
        return(nums)
```