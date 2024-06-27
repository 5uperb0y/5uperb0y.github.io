---
title: ROSALIND | Ordering Strings of Varying Length Lexicographically (LEXV)
date: 2024-05-08 20:59:02
tags: rosalind
categories:
mathjax: true
---

給定字符序列 $\mathscr{A}$，求由這些字符組成且長度小於等於 k 的所有字串。這些字串須依照 $\mathscr{A}$ 字典序排列。

> Given: A permutation of at most 12 symbols defining an ordered alphabet $\mathscr{A}$ and a positive integer n (n≤4).
>
> Return: All strings of length at most n formed from $\mathscr{A}$, ordered lexicographically. (Note: As in “Enumerating k-mers Lexicographically”, alphabet order is based on the order in which the symbols are given.)

(https://rosalind.info/problems/lexv/)

<!--more-->

這題是 ["Enumerating k-mers lexicographically"](https://5uperb0y.com/enumerating-k-mers-lexicographically/) 的變體。在原始題目中，只需列出長度為 k 的所有可能字串，這些字串都能直接從長度為 k - 1 的字串延伸而來。

都是從長度為 k - 1 的字串延伸而來。是以，最終解是遞迴的直接產物，也能追溯到遞迴的基本狀況。

然而，這題則要求列出所有長度「小於等於 k」的字串，例如 $\mathscr{A}$ 為 `["A", "B"]` 而且 k = 2 時，則輸出應包含 `["A", "AA", "AB", "B", "BA", "BB"]`。

雖然 "AA" 和 "AB" 是透過添加字符組成，但是 "A" 和 "B" 則否，輸出結果之前來自不同的關係。為了解決這問題，每次遞迴不只要生成指定長度的字串，還要保留每次遞迴的中繼結果。

```python
def lexv(elements, k):
    """Ordering Strings of Varying Length Lexicographically
    """
    if k == 1:
        return elements
    else:
        output = []
        for e in elements:
            output.append(e)
            output.extend([f"{e}{k_str}" for k_str in lexv(elements, k - 1) ])
        return output
```