---
title: ROSALIND｜Finding a Shared Spliced Motif (LCSQ)
date: 2024-07-14 23:03:26
tags: rosalind 
categories:
---

給定兩條以 FASTA 格式儲存的 DNA 序列，求兩序列最長的共同子序列。

> Given: Two DNA strings s and t (each having length at most 1 kbp) in FASTA format.
>
> Return: A longest common subsequence of s and t. (If more than one solution exists, you may return any one.)

(https://rosalind.info/problems/lcsq/)

<!--more-->

# 背景知識

在實際的基因體研究中，我們可能需要比對多條 DNA 序列，來找出重複出現的 motif 序列。這題要求找出兩條序列最長的共同子序列，它代表了被 introns 分隔的最長 mofif。

較長的序列通常涵蓋更多功能基因，但也因此更容易受到突變和基因重排的影響。如果一段涉及多種功能的長序列能在物種的基因體長期保存，沒有因為演化過程中的偶發事件而被打斷或消失，就暗示著這段序列可能與該物種的核心功能有關。

# 解題觀念

這題是最長共同子序列（LCS, longest common subsequnce）問題的變體，我推薦閱讀[演算法筆記](https://web.ntnu.edu.tw/~algo/Subsequence2.html)的介紹，它清楚解釋了 LCS 的核心觀念與不同解題方法，也有程式實作可供參考。這裡我會依據自己的理解，重述其中一種基於動態規劃的解題方法。

假設有兩個序列 $S$ 與 $T$，它們的最長共同子序列記為 $LCS(S, T)$。如果單獨考慮兩序列的最後一個元素，可以將 $S$ 與 $T$ 重新表示為 $S = S_{sub} + s$ 和 $T = T_{sub} + t$。

那麼根據最後一個元素的狀況，我們可以得出以下關係：

- 如果 $s = t$，那麼這兩個元素都可以納入 $LCS(S, T)$，即 $LCS(S, T) = LCS(S_{sub}, T_{sub}) + s$。
- 如果 $s \neq t$，表示不需要檢查完整的序列即可找到 $LCS(S, T)$，
    - $LCS(S, T)$ 不包含 $s$ 或 $t$，即 $LCS(S, T) = LongerOne(LCS(S_{sub}, T), LCS(S, T_{sub}))$
    - $LCS(S, T)$ 不包含 $s$ 與 $t$，即 $LCS(S, T) = LCS(S_{sub}, T_{sub})$

因為 $LCS(S_{sub}, T_{sub})$ 小於等於 $LCS(S_{sub}, T)$ 和 $LCS(S, T_{sub})$，所以這些關係可以化簡為：

$$ LCS(S, T)=
\begin{cases}
LCS(S_{sub}, T_{sub}) + s & when & s = t\\
LongerOne(LCS(S_{sub}, T), LCS(S, T_{sub})) & when & s\neq t \
\end{cases}
$$

這個關係式表明，兩序列的 LCS 可以從各自子序列的 LCS 推論出來，這構成了應用動態規劃的基礎。

我們可以使用一個二維陣列來記錄 $S$ 和 $T$ 所有子序列的 LCS，再逐一檢視各個子序列的最後一個元素，參考前述關係式填滿這個陣列。最終，陣列右下角的元素就是題目要求的最常共同子序列。

# Python 實作

動態規劃的實作通常包含三個步驟：定義資料結構、釐清子結構間的關係，確認初始值[^1]。考量這題要求找出 DNA 序列的最長共同子序列（為了方便討論，此處字串視為廣義的序列），我選擇以巢狀 list 來模擬二維陣列。每個元素代表對應的 DNA 子序列的最長共同子序列。

這樣方法雖然需要額外的儲存空間，但我覺得比較直觀易懂，因為它需要不額外的步驟來回溯最長共同子序列的內容。

以下實作需要注意幾點：

- 二維矩陣的長與寬都要比兩條 DNA 序列的長度各多 1，因為它儲存的是 `S[:index]` 的最長共同子序列
- 兩條 DNA 序列最短的子序列是空序列，所以它們的初始值是一個空字串。
- 填表時依照前述的關係式來進行，這裡用了 `max(a, b, key=len)` 來簡化程式碼。


[^1]: 我覺得動態規劃難在看得懂但是想不到。不過別人花了一輩子研究才想出來的演算法，要我們在周末下午憑空刻出來也是強人所難。因此，我覺得適度原諒遲鈍的自己也是很值得鼓勵的行為喔。順道一提，我蠻喜歡這篇關於動態規劃的文章，推薦給大家:) [动态规划很难？DP连刷40道题，我总结出了这些套路](https://github.com/iamshuaidi/algo-basic/blob/master/%E5%AD%A6%E7%AE%97%E6%B3%95/%E5%AD%A6%E5%8A%A8%E6%80%81%E8%A7%84%E5%88%92/%E5%8A%A8%E6%80%81%E8%A7%84%E5%88%92%E5%BE%88%E9%9A%BE%EF%BC%9FDP%E8%BF%9E%E5%88%B740%E9%81%93%E9%A2%98%EF%BC%8C%E6%88%91%E6%80%BB%E7%BB%93%E5%87%BA%E4%BA%86%E8%BF%99%E4%BA%9B%E5%A5%97%E8%B7%AF.md)

```python
def lcsq(S, T):
    """Calculate and return the longest common subsequence (LCS) of two strings S and T.

    Args:
        S, T (str): The two input DNA strings.

    Return:
        str: A longest common subsequence of s1 and t.
    """
    # lcs denotes the longest subsequence of all s1's and s2's substring
    lcs = [
        [""] * (len(T) + 1)
        for _ in range(len(S) + 1)
        ]
    for s in range(len(S)):
        for t in range(len(T)):
            if S[s] == T[t]:
                lcs[s + 1][t + 1] = lcs[s][t] + S[s]
            else:
                lcs[s + 1][t + 1] = max(lcs[s + 1][t], lcs[s][t + 1], key=len)
    return lcs[-1][-1]
```