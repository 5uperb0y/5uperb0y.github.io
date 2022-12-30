---
title: LeetCode 筆記｜724. Find Pivot Index
date: 2022-12-20 20:34:20
tags: array
categories: [programming, leetcode]
---

若數列某項兩側各數字的總和相同，則該項為數列的樞紐 (pivot)。給定一整數數列，判斷其是否含樞紐項。若有，求樞紐之索引值；若無，則回傳 `-1`。
<!--more-->
## 問題描述

> Given an array of integers `nums`, calculate the pivot index of this array.
> The pivot index is the index where the sum of all the numbers strictly to the left of the index is equal to the sum of all the numbers strictly to the index's right.
> **If the index is on the left edge of the array, then the left sum is `0`** because there are no elements to the left. This also applies to the right edge of the array.
> Return the leftmost pivot index. If no such index exists, return `-1`.

**樣例**
> Input: nums = [1,7,3,6,5,6]
> Output: 3

> Input: nums = [-1,-1,-1,-1,-1,0]
> Output: 2

> Input: nums = [-1,-1,-1,0,1,1]
> Output: 0

*留意負數項、樞紐位於數列首尾、數列只含一項或兩項的案例*

**限制**

- `1 <= nums.length <= 10^4`
- `-1000 <= nums[i] <= 1000`：留意負數項

**知識點**
prefix sum & array

## 思路與題解
此處借鑑[官方解法](https://leetcode.com/problems/find-pivot-index/solutions/127676/find-pivot-index/)。首先計算數列的級數 (`S`)，接著依序讀過數列，並計算讀過數字之和 (`leftSum`，即當前項左側的子數列總和)。

由於已算過 `S`，所以當前項右側的子數列總和可透過 `S - nums[i] - leftSum` 得知。

若當前項左右兩側的子數列總和一致，表示當前項為樞紐，其索引值便為題解；若總和不一致，則累計 `leftSum` 的值。若遍歷數列仍未求得樞紐，即可確認此數列無樞紐，依題目要求回傳 `-1`。

```python
class Solution:
    def pivotIndex(self, nums: List[int]) -> int:
        leftSum = 0
        S = sum(nums)
        for i in range(len(nums)):
            if leftSum == S - nums[i] - leftSum:
                return(i)
            else:
                leftSum += nums[i]
        return(-1)
```

## 延伸討論
### 曾嘗試但失敗的解法
我原先的想法是定義兩個指標，在一次迴圈中從數列兩側夾擠，同時計算 `leftSum` 和 `rightSum`。當兩指標重合，且左右數列總和一致時，指標的位置即為樞紐的索引值；反之，則表示此數列沒有樞紐。

這種方法的關鍵在於判斷何時要調動兩側的指標，所以在程式碼中間我寫了不少判斷式。然而，當數列正負數穿插的時候，判斷式便會很難寫。

舉下方的程式碼為例，碰到 `[-1,-1,-1,0,1,1]` 和 `[-1,-1,-1,0,1,1,0]` 這兩個案例時，右側指標會在循環中不斷往左側移動，直接錯過樞紐的位置。
```python
class Solution:
    def pivotIndex(self, nums: List[int]) -> int:
        l = 0
        r = len(nums) - 1
        leftSum = nums[l]
        rightSum = nums[r]
        while l < r:
            if  abs(leftSum) < abs(rightSum):
                l += 1
                leftSum += nums[l]
            else:
                r -= 1
                rightSum += nums[r]
        
        if leftSum == rightSum:
            return(l)
        else:
            return(-1)
```

### 練習題目時，適合使用程式語言定義好的 function 嗎？
我覺得這算是我其中一種心態糾結，即「為了打好基礎，必須手刻所有功能」。不過追根究柢，做題的目的是透過題目學習該題相關知識。

以這題為例，`sum`並非解題的關鍵，也不是這題的核心知識，如果使用這個 function 能改善程式碼的可讀性和簡潔性，那麼我覺得使用程式語言定義好的 function 也不錯。