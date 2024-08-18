---
title: ROSALIND｜Edit Distance (EDIT)
date: 2024-07-20 00:54:47
tags: rosalind 
categories:
---

給定兩條以 FASTA 格式儲存的蛋白質字串，計算兩者互相轉換所需的最少編輯次數。編輯方式包含置換、刪除與插入。

> Given: Two protein strings s and t in FASTA format (each of length at most 1000 aa).
>
> Return: The edit distance d(s,t).

(https://rosalind.info/problems/edit/)

<!--more-->

# 背景知識

在先前的 Rosalind 練習題，我已經陸續介紹兩種評估序列差異的方式，它們各自反映了特定的基因體變化，因此可用於推論親緣關係或演化途徑。

- [Hamming distance](https://5uperb0y.com/counting-point-mutations/)：兩等長序列彼此轉換所需的鹼基置換次數
- [reversal distance](https://5uperb0y.com/reversal-distance/)：兩序列彼此轉換所需的子序列反轉次數

這一題在置換（substitution）的基礎上，額外納入插入（insertion）與刪除（deletion）兩種點突變類型，將 Hamming distance 推廣為編輯距離（edit distance，字串互相轉換所需的編輯次數），以適用於長度不等的序列。

由於引進了新的突變類型，計算距離時也要考慮不同突變類別的權重。舉例來說，插入或刪除鹼基可能導致 readframe 移位，所以它們對基因體的影響可能比置換更為顯著。不過，這道題目只考慮最簡單的狀況，即所有突變類型對編輯距離的權重一致。

# 解題觀念

假設有兩個蛋白質字串 $S$ 與 $T$，兩者之間的最短編輯距離記為 $d(S,T)$。原則上，$S$ 和 $T$ 的長度越長，$d(S,T)$ 的上限也越高。為了評估延長這兩個字串對編輯距離的影響，可以由後往前，逐一比較各自的最後一位字符 $s$ 和 $t$，將字串重新表示為 $S=S_{sub}+s$ 與 $T=T_{sub}+t$。

如果 $s$ 與 $t$ 相同，表示添加它們不會增加編輯距離，所以 $d(S,T)=d(S_{sub},T_{sub})$。

如果 $s$ 與 $t$ 相異，則有三種可能的編輯方式，每種方式都會讓編輯距離增加 1。

- 把 $s$ 置換為 $t$，所以 $d(S,T)=d(S_{sub},T_{sub}) + 1$
- 從 $S$ 刪除 $s$（或在 $T_{sub}$ 末端插入 $t$），所以 $d(S,T)=d(S_{sub}, T_{sub} + t) + 1$
- 從 $T$ 刪除 $t$（或在 $S_{sub}$ 末端插入 $s$），所以 $d(S,T)=d(S_{sub} + s, T_{sub}) + 1$

每次延長字串時，我們都選擇增加最少編輯距離的方式，最終可得 $d(S,T)$。

# Python 實作

我使用 `dist` 儲存 $S$ 和 $T$ 各個子字串的編輯距離。`dist` 是一個雙層嵌套的 list，它的大小比兩個字串的長度多 1，以包含空字串的狀況。

由於空字串只能透過插入轉換為其它字串，所以 `dist` 的第 0 列和第 0 欄被設定為各子串的長度，其餘元素則初始化為 0，表示未知的編輯距離。

接著，由上至下，由左至右遍歷 `dist`，比較兩個子字串的最後一位字符，依據前述關係判斷當前的最小編輯距離。最終，`dist` 右下角的元素即為兩個完整字串的最小編輯距離。

```python
def edit(S, T):
    # The edit distance between S's and T's substrings
    dist = [
        [0] * (len(T) + 1)
        for _ in range(len(S) + 1)
        ]
    for s in range(1, len(S) + 1):
        dist[s][0] = s
    for t in range(1, len(T) + 1):
        dist[0][t] = t
    for s in range(len(S)):
        for t in range(len(T)):
            if S[s] == T[t]:
                dist[s + 1][t + 1] = dist[s][t]
            else:
                dist[s + 1][t + 1] = min(dist[s][t], dist[s + 1][t], dist[s][t + 1]) + 1
    return dist[-1][-1]
```