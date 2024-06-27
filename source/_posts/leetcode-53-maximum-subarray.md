---
title: LeetCode 筆記｜53. Maximum Subarray
date: 2022-12-09 21:04:29
tags: leetcode
categories:
---

## 問題描述
> Given an integer array nums, find the subarray which has the largest sum and return its sum.

給一整數數列，求其子數列級數的最大值。

**樣例**
> Input: nums = [-2,1,-3,4,-1,2,1,-5,4]
> Output: 6

<!--more-->

**限制**
- $1$$\leq$ nums.length $\leq$$10^5$：數列長度介於 $1$ 至 $10^5$ 間
- $-10^4$$\leq$ nums[i] $\leq$$10^4$：數值介於 $-10^4$ 至 $10^4$ 間

**知識點**
- dynamic programming
- Kadane's Algorithm

## 思路與題解
這題常見的作法是 Kadane's algorithm，我在解題時並不知道這套演算法，所以一開始是用極端情形勾勒 function 的輪廓，再透過更多樣例「擬合」出可能的解法，最後才整合為提交的答案。

- **數列僅包含一個數字**：解為該項數字，表示即使只有一個數字也能回傳
- **數列全為正值**：解為數列各項之和，所以程式碼內含有累加操作
- **數列全為負值**：解為數列最大的一項數字，所以程式碼內含有暫存最大值的變項與比較數值的判斷式

由此分析可知問題癥結在於，累加的級數在碰到負數時要如何處置？
- 若級數於累加後增加，則將這些數字納入計算。例如 [2, -1, -3, 5]，由於 5 > -1 + -3，所以仍有涵蓋這兩個負數的價值。
- 若級數於累加後減少，則跳過這些數字，從新的位置開始計算。例如 [2, -1, -4, 1]，累加新的數字無法抵銷兩個負數值，所以不如跳過。

至此，我雖然對計算方式有個概念，但還是經過了一連串測試，才歸納出以下算法。依序累加數列各項，每次累加後 (1) 若累加後級數沒增加，則從當前項重新計算級數；(2) 更新當前級數的最大值。

遍歷整條數列後，當前級數的最大值即為題解。只須走一趟循環，空間用量也固定，所以時間複雜度和空間複雜度分別為 $O(n)$ 與 $O(1)$。

```python
class Solution:
    def maxSubArray(self, nums: List[int]) -> int:
        sums = 0
        maxVal = nums[0]
        for i in range(len(nums)):
            sums += nums[i]
            if sums < nums[i]:
                sums = nums[i]
            if sums > maxVal:
                maxVal = sums
        return(maxVal)
```
*當初以為要回傳子數列的位置，所以才使用 index 取值*
- `sums`：紀錄當前累加值
- `maxVal`：紀錄當前累加的最大值