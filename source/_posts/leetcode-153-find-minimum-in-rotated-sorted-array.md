---
title: LeetCode 筆記｜153. Find Minimum in Rotated Sorted Array
date: 2022-12-13 22:45:15
tags: leetcode
categories: programming
---

## 問題描述
> Given the sorted rotated[^1] array `nums` of unique elements, return the *minimum element of this array*.

平移一嚴格遞增數列各項 n 單位後，求此數列的最小值

**樣例**
> Input: nums = [3,4,5,1,2]
> Output: 1

*須留意幾個樣例：數列只含一項、數列只含兩項、平移後與原數列相同、最小值位於數列尾部*

<!--more-->

**限制**
- `n == nums.length`：數列長度為 n
- `1 <= n <= 5000`：數列長度介於 1 至 5000 項
- `-5000 <= nums[i] <= 5000`：數列各項介於 -5000 至 5000，因此兩項相加不至於超出整數大小的上下界
- All the integers of `nums` are **unique**：數列各項唯一
- `nums` is sorted and rotated between `1` and `n` times：數列已排序，且至多平移 n 次，最少平移 1 次
- You must write an algorithm that runs in `O(log n)` time：此題有時間複雜度限制

**知識點**
binary search

## 思路與題解
此題的關鍵在於判斷最小值，並使用二分搜尋法簡化找尋最小值的步驟。

由於數列已由小到大排序，所以如果某項數值在平移後小於前一項，該項即為數列的最小值。依照這項原則，可以先定義 `left`、`mid`、`right` 三個指標，其中 `mid` 是 `left` 與 `right` 的中間值。

若 `nums[mid]` 小於其前一項，則 `nums[mid]` 為最小值，即可中斷迴圈並回傳題解。若尚未找到最小值，則可比較 `nums[mid]` 與 `nums[right]`，選定下一個搜索標的。

假設 `nums[right]` > `nums[mid]`，表示數列於 `mid` 至 `right` 之間嚴格遞增，表示原數列的首尾交接處不在此區間，所以可以縮小搜索範圍到 `left` 至 `mid - 1` 之間。反之，則表示數列於 `left` 至 `mid` 之間嚴格遞增，則可縮小搜索範圍至 `mid + 1` 至 `right` 之間。

由於嚴格遞增數列一定有唯一的最小值，所以最遲在 `left` 與 `right` 指向同一座標時，會找到數列最小值。

```python
class Solution:
    def findMin(self, nums: List[int]) -> int:
        left = 0
        right = len(nums) - 1
        while left <= right:
            mid = right - (right - left)//2
            if nums[mid] < nums[mid - 1]:
                break
            else:
                if nums[mid] > nums[right]:
                    left = mid + 1
                else:
                    right = mid - 1
        return(nums[mid])
```

## 延伸討論

### `mid` 的定義方式

`mid` 亦可定義為 `(left + right)//2`，但聽同事討論，若兩數的數值極大，則有數值超出整數範圍的風險。以相減的方式撰寫則一定不會超出範圍。

### 判斷首尾交界處的方式
有看過同事在比較時，不比對 `nums[right]`，而是比對 `nums[-1]`，有一樣的效果。


[^1]: rotation 是指數列各項座標平移相同單位，例如 [1, 2, 3, 4] 平移 1 單位即為 [4, 1, 2, 3]，平移 2 單位則為 [3, 4, 1, 2]