---
title: ROSALIND｜Enumerating Gene Orders (PERM)
date: 2024-05-05 23:12:35
tags: rosalind 
categories:
---

給定一正整數 n，求包含數字 1 到 n 的所有可能數列與其總數。

> A permutation of length n is an ordering of the positive integers {1,2,…,n} . For example, π=(5,3,2,1,4) is a permutation of length 5.
>
> Given: A positive integer n≤7 .
>
> Return: The total number of permutations of length n , followed by a list of all such permutations (in any order).

(https://rosalind.info/problems/perm/)

<!--more-->

# 知識點

基因體重排（genome rearrangement）泛指基因體上發生的刪除（deletion）、插入（insertion）、重複（duplication）、反轉（inversion）或易位（translocation）等現象。這些變動通常涉及數百至數百萬對鹼基，相較於點突變，它們的影響更為廣泛。

基因體重排的發生機制主要有[^gu2008]：

- non-allelic homologous recombination (NAHR)：細胞分裂中，染色體對齊時發生錯位，導致片段重組。
- non-homologous end-joining (NHEJ)：DNA 斷鍊修復時，連結了非同源的染色體。
- Fork Stalling and Template Switching (FoSTeS)：DNA 複製時因故停滯時，新合成的 DNA 意外移動到其他模板進行複製。

這些變化可能導致基因無效而喪失功能、彼此融合而改變功能，或是副本數變動而加強/減弱功能，從而影響細胞結構與生理。許多疾病和癌症與基因體重排有關，例如 BRCA1 和 BRCA2 的副本數變化與乳癌發作密切相關。[^ewald2009]

基因體重排不僅是疾病基因體學的重要研究對象，也因為被視為種化的分子機制之一，而在演化生物學中占一席之地。相較於點突變，基因體重排的發生頻率較低，使得親緣關係相近的物種在染色體層級上保持一致，即使它們在局部基因序列上略有不同。

然而，親緣關係較遠的物種則可能因為基因體重排而有顯著差異。在演化過程中，這些沒有被分離到道不同區域或染色體的基因彼此聚集成 synteny block。在比較基因體學中，synteny block 提供了一種比染色體數量和大小更細緻的描述單位，有助於在不過度瑣碎的層級上討論基因體的變化。透過比較 synteny block 的繼承與變化，可以推輪物種分化的歷史。[^touchman2010]

這題的設計反映了基因體重排的概念，將每個數列視為一條染色體，其中每個數字代表一個 synteny block，數字的順序則代表這些 block 在餐考基因體上出現的順序或染色體編號。透過模擬單一染色體 synteny block 的不同排列，探索可能的重排情境。

[^ewald2009]: Ewald et al. (2009). Genomic rearrangements in BRCA1 and BRCA2: A literature review. Genetics and Molecular Biology, 32, 437-446.
[^gu2008]: Gu et al. (2008). Mechanisms for human genomic rearrangements. Pathogenetics, 1, 1-17.
[^touchman2010]: https://www.nature.com/scitable/knowledge/library/comparative-genomics-13239404/

# 解題

讓我們從最簡案例來推論怎麼用遞迴解這題。假設數列只有一個數字，也就只有一種可能的排列；而兩個數字的排列也很直觀。

```python
# n = 1
[[1]]
# n = 2
[[1, 2], [2, 1]]
```

現在考慮數列有三個數字的情形。由於沒辦法一眼得到解答，於是我們需要更系統的解決辦法，其中一個策略是：

1. 先取出其中一個數字
2. 排列剩下的數字
3. 把取出的數字加回排列結果
4. 重複取出所有數字後，便得到數列的所有可能

如同以下各數列所示，我首先依序取出 1、2、3 （數列的首項），然後再排列剩餘兩個數字，再把取出的數字與排列後的數列結合。

```python
# n = 3
[
    [1, 2, 3],
    [1, 3, 2],
    [2, 1, 3],
    [2, 3, 1],
    [3, 1, 2],
    [3, 2, 1]
]
```

歸納前述方法，便能用 python 實作以下關係：

- 若數列只有一個數字，回傳此數列
- 若數列不只一個數字，則依序取出各數字，排列剩餘數字，再把取出的數字加到排列的結果中。

```python
def lperm(l):
    """Permutations of a list
    """
    if len(l) == 1:
        return [l]
    else:
        return [
            p + [e]
            for i, e in enumerate(l)
            for p in lperm(l[:i] + l[i+1:])
        ]
```

隨後便可整理排列的結果為題目要求的格式。由於我們已經列出所有可能數列，所以直接計算數列總數即可，不用多做一次階乘計算。

```python
def print_permutations(n):
    num_perms = lperm(list(range(1, n+1)))
    print(len(num_perms))
    for p in num_perms:
        print(" ".join(map(str, p)))
```

# 討論

我解這題的靈感來自 [All Permutations of a String in Python (Recursive)](https://stackoverflow.com/questions/23116911/all-permutations-of-a-string-in-python-recursive) 這串問答，我只需要把原方法改寫為適合 list 的方式即可。不過一開始我寫成以下形式，能看出會報什麼錯誤嗎？

```python
# !!!! WRONG SOLUTION !!!!
def lperm(l):
    if len(l) == 1:
        return l
    else:
        return = [
            p + [e]
            for i, e in enumerate(l):
            for p in lperm(l[:i] + l[i+1:])
        ]
        return result
```

關鍵在於遞迴關係的初始狀態設定，這個 function 預期要回傳一個存儲所有可能數列的巢狀 list，但我在初始狀態的時候寫成 `return l`，導致遞迴調用 function 回傳了一個攤平的 list。於是 `p` 就成了整數，自然無法和 `[e]` 相連，讓程式無法運作。