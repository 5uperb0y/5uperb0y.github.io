---
title: ROSALIND | Enumerating k-mers Lexicographically (LEXF)
date: 2024-05-07 17:48:09
tags: rosalind
categories:
---

給定字符序列 $\mathscr{A}$，求由這些字符組成且長度為 k 的所有字串。這些字串須依照 $\mathscr{A}$ 字典序排列。

> Given: A permutation of at most 12 symbols defining an ordered alphabet $\mathscr{A}$ and a positive integer n (n≤4).
>
> Return: All strings of length at most n formed from $\mathscr{A}$ , ordered lexicographically. (Note: As in “Enumerating k-mers Lexicographically”, alphabet order is based on the order in which the symbols are given.)

(https://rosalind.info/problems/lexv/)

<!--more-->
$\mathscr{A}$ 是一個字符序列，除了規範了可用的字符外，也定義了這些字符的字典序。像是英文字母的字典序是從 A 到 Z，所以 "AA" 會排在 "AB" 前面。這題要求從 $\mathscr{A}$ 生成指定長度的所有可能字串，並確保這些字串依照字典序排序。

假設今有 $\mathscr{A}$ 為 `["A", "B", "C"]`，而且指定長度為 3，則生成步驟為：

1. 從 $\mathscr{A}$ 依序取出字符來組成字串（"A"、"B"、"C"）
2. 對於每個字符，追加 $\mathscr{A}$ 中的每个字符，生成長度為 2 的所有可能字串（"AA"、"AB"、"AC"、"BA"、"BB"、……"CC"）
3. 對於長度為 2 的字串，再度追加 $\mathscr{A}$ 中的每个字符，生成長度為 3 的所有可能字串（"AAA"、"AAB"、"AAC"、"ABA"、……"CCC"）
4. 最終將生成 $3 \times 3 \times 3 = 27$ 個字串。

由上述案例，我們可以歸納遞迴關係如下：

- 終止條件：當 `k=1` 時，回傳 $\mathscr{A}$。
- 遞迴關係：對於 $\mathscr{A}$ 每個字符，生成長度為 `k - 1` 的所有可能字串，再讓字符作為這些字串的前綴。

```python
def lexf(elements, k):
    """Generate all permutations of a specified length from a given set of elements.
    """
    if k == 1:
        return elements
    else:
        return [
            f"{e}{k_str}"
            for e in elements
            for k_str in lexf(elements, k - 1)
        ]
```

最後再把結果依題目要求的格式印出來。

```python
def print_results(elements, k):
    for k_str in lexf(elements, k):
        print(k_str)
```
