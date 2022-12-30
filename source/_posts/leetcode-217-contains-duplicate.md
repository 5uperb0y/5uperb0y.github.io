---
title: LeetCode 筆記｜217. Contains Duplicate
date: 2022-11-28 23:23:51
tags: array
categories: [programming, leetcode]
---

> Given an integer array `nums`, return `true` if any value appears at least twice in the array, and return `false` if every element is distinct.

(給定一整數陣列，判斷其中是否含重複的數字。)

Example:
> Input: nums = [1,2,3,1]
> Output: true

<!--more-->

## 解題思路

若 array 含重複數字，則有以下特性：

*   數字重複出現
*   array 長度大於其中數字種類數
*   若將 array 排序，則重複的數字比鄰出現

根據第一種特性，可用 hash table 存儲讀過的數字，再判斷讀入的數字是否已存在 hash table 中。此處，我以 python 的 `dict` 充作 hash table 來實踐這個想法。

```python
class Solution:
    def containsDuplicate(self, nums: List[int]) -> bool:
        occur = {}
        for n in nums:
            if n in occur.keys():
                return(True)
            else:
                occur[n] = n
        return(False)
```    

由於計算時間和空間需求都隨陣列大小線性增長，所以時間與空間複雜度都是 $O(n)$

除了 `dict`，python 的 `set` 也能體現 hash table 的特性。`set` 可想像為僅有 key 的 `dict`，由於 `set` 的元素皆獨一無二，故可將 array 轉為 `set`，再比較兩者的長度。若 `set` 長度小於 array，則表示 array 含重複值。
```python
class Solution:
    def containsDuplicate(self, nums: List[int]) -> bool:
        if len(set(nums)) != len(nums):
            return(True)
        else:
            return(False)
``` 

至於先排序，再比較數值兩側是否有重複值的做法，可參考：[LeetCode 刷題紀錄 ｜217. Contains Duplicate (Easy)](https://medium.com/roywannago-%E6%97%85%E8%A1%8C%E4%B8%8D%E9%9C%80%E7%90%86%E7%94%B1/leetcode-%E5%88%B7%E9%A1%8C%E7%B4%80%E9%8C%84-217-contains-duplicate-easy-647bc2ccdcf6)

## 延伸討論

* 使用 list, dict, set 存讀過的數字有什麼差異？（參考：[廖雪峰的官方網站：使用dict和set](https://www.liaoxuefeng.com/wiki/1016959663602400/1017104324028448)）

* python 是怎麼體現 `set` 的概念？（參考：[How is set() implemented?](https://stackoverflow.com/questions/3949310/how-is-set-implemented)）

* 排序算法跟 hash table 算法相比有什麼優點？（參考：[LeetCode 刷題紀錄 ｜217. Contains Duplicate (Easy)](https://medium.com/roywannago-%E6%97%85%E8%A1%8C%E4%B8%8D%E9%9C%80%E7%90%86%E7%94%B1/leetcode-%E5%88%B7%E9%A1%8C%E7%B4%80%E9%8C%84-217-contains-duplicate-easy-647bc2ccdcf6)）