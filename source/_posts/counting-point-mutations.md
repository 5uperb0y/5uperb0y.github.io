---
title: ROSALIND｜Counting Point Mutations (HAMM)
date: 2024-05-04 13:13:25
tags: rosalind 
categories:
---

給定兩等長字串 s 和 t，計算兩者的 Hamming distance dH(s,t)

> Given: Two DNA strings s and t of equal length (not exceeding 1 kbp).
>
> Return: The Hamming distance dH(s,t).

(https://rosalind.info/problems/hamm/)

<!--more-->

# 知識點

分子生物學的突變 (mutations) 是基因序列的改變。其中，點突變 (point mutations) 是指單一鹼基的變化，依照變化的模式可分為 transition（pyrimidine → pyrimidine, purine → purine）和 transversion (purine ←→ pyrimidine)。

點突變的常見原因為化學修飾或複製錯誤。例如亞硝酸 (nitrous acid) 可透過氧化脫胺將 cytosine 轉化為 uracil，導致該位置的鹼基在下一輪複製發生 transition：C≡G (initial base pair) → U≡G (after mutation) → U=A (after replication)。

由於 Hamming distance 是兩等長字串對應位置之字符不相符的數量，一條序列要轉變為另一條序列最少需要的點突變次數可用 Hamming distance 描述，藉此得以推論源於共同祖先的兩條基因序列的最簡演化途徑。
- ATCG 和 AAAA 的 Hamming distance 為 3
- 1234 和 1233 的 Hamming distance 為 1
- dog  和 god  的 Hamming distance 為 2

# 題解

## python
使用迴圈依序比對兩字串對應位置的鹼基是否一致，再透過記數變項統計不相符的數量即可計算 Hamming distance
```python
s = "GAGCCTACTAACGGGAT"
t = "CATCGTAATGACGGCCT"
hdist = 0
for i in range(0, len(s) - 1):
    if s[i] != t[i]:
        hdist = hdist + 1
print hdist
```
此任務亦可配合 zip 改寫為 list comprehension 形式，即合計兩字串對應位置鹼基符號不相符的數量。
```python
print sum([a != b for a, b in zip(s, t)])
```

簡言之，zip 可將兩 list 的元素依序取出配對形成新的 list，所以能配合 for 一次處理兩個 list 的元素。例如輸入 zip("abc", "def") 會回傳 [("a", "d"), ("b", "e"), ("c", "f")]。zip() 的介紹詳見 [Python 使用 zip 與 for 迴圈同時對多個 List 進行迭代](https://blog.gtwang.org/programming/python-iterate-through-multiple-lists-in-parallel/)

至於 List comprehension 則能以較簡潔的程式碼應用 for loop，也往往有較高的執行效率。此表達法的典型形式為 [expression for item in iterable]。例如要從字串中取出非A的序列，一般的 for loop 寫法為
```
# common method
str = []
for i in "AATCGG":
	if i != "A":
		str.append(i)
print str
```
使用 List comprehension 可將程式碼簡化為一行，
```
# list comprehension method
print [i for i in "ATCGG" if i != "A"]
```
使用 list comprehension 的其他特性，可參考[What is the advantage of a list comprehension over a for loop?](https://stackoverflow.com/questions/16341775/what-is-the-advantage-of-a-list-comprehension-over-a-for-loop) 

回到 hamming distance 的計算，另一種寫法是不直接加總 True/False，而是用判斷式把不相符時的距離加權記錄在程式中，這樣可以明確表達鹼基不符時，要計算多少距離。

```python
def hamming_distance(s1, s2):
	"""Calculating hamming distance of two strings with equal length
	"""
	return sum( 1 for c1, c2 in zip(s1, s2) if c1 != c2 )
```

## R
R 的作法則是使用 strsplit 將字串轉換為 list，再以向量運算對應位置的鹼基是否一致，最後以 sum 統計相異鹼基的數量。
```
s <- "GAGCCTACTAACGGGAT"
t <- "CATCGTAATGACGGCCT"
sum(strsplit(s, "")[[1]] != strsplit(t, "")[[1]])
```