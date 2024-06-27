---
title: LeetCode 筆記｜1. Two Sum
date: 2022-12-15 00:07:15
tags: leetcode
categories:
---

## 問題描述
> Given an array of integers nums and an integer target, return indices of the two numbers such that they add up to target.

求數列內，和為目標值的兩數之座標

**樣例**
> Input: nums = [3,2,4], target = 6
> Output: [1,2]

**限制**
- `2 <= nums.length <= 104`：數列長度介於 2 至 104 之間
- `-109 <= nums[i] <= 109`：數列各項可能有負值
- `-109 <= target <= 109`：目標值可能為負值
- Only one valid answer exists.：僅有一組解

<!--more-->

**知識點**
hash table

## 思路與題解
此題最直觀的解法是將數列各項兩兩相加，最遲到比對所有組合後才找到題解。然而，目標值在這算法裡僅作為判斷結果的依據，沒有充分利用其提供的資訊。


為了充分利用目標值，可以各項與目標值之差為 keys，各項座標為 values，建立 hash table 存儲讀過的數字。

若現在的數字為先前出現過的數字與目標值之差，則回傳這兩數字的座標即為題解；若否，則新增現在的數字與目標值之差到 hash table 內。由於題目設計保證有唯一解，所以最遲遍歷整個數列即可找到題解。 

```python
class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        record = {}  
        for i in range(len(nums)):
            diff = target - nums[i]
            if nums[i] in record.keys() :
                return([record[nums[i]], i])
            else: 
                record[diff] = i
```
- 此以 python dict 實踐 hash table 的功能。
- `record.keys()` 可寫成 `record`，但標出來比較清楚