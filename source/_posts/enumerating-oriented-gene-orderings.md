---
title: ROSALIND｜Enumerating Oriented Gene Orderings (SIGN)
date: 2024-05-06 14:45:59
tags: rosalind 
categories:
---

給定一正整數 n，求包含數字 1 到 n 與 -1 到 -n 的所有可能數列與其總數。

> A signed permutation of length n is some ordering of the positive integers {1,2,…,n} in which each integer is then provided with either a positive or negative sign (for the sake of simplicity, we omit the positive sign). For example, π=(5,−3,−2,1,4) is a signed permutation of length 5
> 
> Given: A positive integer n≤6
>
> Return: The total number of signed permutations of length n, followed by a list of all such permutations (you may list the signed permutations in any order).

<!--more-->

# 知識點

Synteny blocks 是相異物種因為基因體重排而分離的同源片段。在 [Enumerating Gene Orders](https://5uperb0y.com/enumerating-gene-orders/) 當中介紹了以數列排列模擬 synteny blocks 重排的想法，我們可以額外考慮 blocks 的方向性來改善模型的精細度。DNA 雖然由兩股去氧核酸構成，但轉錄僅發生在其中一股。

由於 synteny blocks 在演化過程中除了易位，也可能發生轉置，顛倒基因的排列和方向性，改變轉錄與基因調控等行為，從而影響個體的表徵。例如，Factor VIII Intron 22 的轉置便是血友病A的成因之一。

[^lakich1993]:Lakich et al. (1993). Inversions disrupting the factor VIII gene are a common cause of severe haemophilia A. Nature genetics, 5(3), 236-241. 

# 題解
這題的解法近似[數列的排列](https://5uperb0y.com/enumerating-gene-orders/)，只是透過正負號來表示 synteny blocks 的方向。因為要考慮正負符號，所以需要兩個調整。

1. 遞迴的 base case 要回傳數字與其相反數。例如輸入數列 `[1]`，要回傳 `[[1], [-1]]`。
2. 在把數字放回排列後的數列時，也要分別加入數字和其相反數。

```python
def slperm(l):
    """permutations of a numberic list, including -/+ of each number.
    """
    if len(l) == 1:
        return [[l[0]], [-l[0]]]
    else:
        return [
            p + [sign_e]
            for i, e in enumerate(l)
            for p in slperm(l[:i] + l[i+1:])
            for sign_e in [e, -e]
        ]
```

