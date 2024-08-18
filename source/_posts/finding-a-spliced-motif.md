---
title: ROSALIND｜Finding a Spliced Motif (SSEQ)
date: 2024-07-06 21:31:59
tags: rosalind 
categories:
---

透過 FASTQ 格式給定 s 與 t 兩個 DNA 字串，求 t 的各個字符依序出現在 s 中的位置，使得這些字符構成 s 的子序列（subsequence）。（只需提供其中一組解，不用列出所有可能的子序列位置。）

> Given: Two DNA strings s and t (each of length at most 1 kbp) in FASTA format.
> 
> Return: One collection of indices of s in which the symbols of t appear as a subsequence of s. If multiple solutions exist, you may return any one.

(https://rosalind.info/problems/sseq/)

<!--more-->

# 背景知識

在真核生物基因體中，introns 會在轉錄後加工時被移除，所以 DNA 序列不會完全呈現在 mRNA 上。因此，某些 motif 可能以不連續的方式出現基因體上。因此，透過 DNA 序列尋找這些 motif 時，不僅要考慮彼此相連的序列，還要納入可能被 introns 分隔的序列。

# 解題觀念

這題涉及「子字串」和「子序列」的區別。在生物資訊領域，我們常用序列來描述 DNA，所以這兩個概念可能產生歧義。

- 子字串（substring）：字串的一部分，要求字符間彼此相鄰。例如 "abc" 是 "abcde" 的子字串，但 "ac" 不是。
- 子序列（subsequence）：序列的一部份，僅要求元素出現的順序與原序列一致，不要求子序列各元素彼此相鄰。例如 [a,b,c] 和 [a,c] 都是 [a,b,c,d,e] 的子序列。

這一題等價於尋找子序列的位置，而且我們只須找到其中一組解。其中一個方法是依序檢查序列，當碰到與子序列相符的字符時，紀錄它的位置，然後繼續在序列中尋找子序列的下一個元素，直到找到子序列的所有元素。

# Python 實作

實作時，從 motif 的第一個字符開始，逐一檢查 DNA 序列中的每個字符。一旦發現 DNA 中的字符與當前檢查的 motif 字符相同時，就把該字符的位置記錄到 motif_pos（因為題目要求從 1 開始計數，所以索引值要加 1）。接著，繼續在 DNA 中尋找與 motif 下一個字符相符的字符，直到找到 motif 中所有字符為止。

```python
def sseq(dna, motif):
    motif_pos = []
    for pos, nt in enumerate(dna):
        if motif:
            if nt == motif[0]:
                motif_pos.append(pos + 1) 
                motif = motif[1:]
            else:
                pass
        else:
            break
    return motif_pos
```

# 討論：找出所有 spliced motif 的位置

為了挑戰自己，我在解完這題之後，開始研究怎麼找出所有 spliced motif 的位置。不過最終仍沒想出來，也看不太懂 AI 建議的解答。因此，我暫時先把程式碼和自己的見解記錄下來，期待自己在練習更多題目之後能夠開竅😥。

## 動態規劃

使用動態規劃解題的關鍵在於使用 dp[j] 記錄 motif[:j] 在 DNA 中所有可能的子序列位置。對於 DNA 序列中的每個字符，從後往前檢查 motif 的字符是否匹配。一旦兩者當前檢查的字符一致時，舊更新 dp。這個更新的過程利用了一個特性：由於已知 motif[:j] 在 DNA 中所有可能的子序列位置，那麼 motif[:j+1] 的所有可能子序列就可以透過將當前匹配的字符位置添加到 motif[:j] 所有的解來獲得。

這裡有個技術細節是，因為 tple 無法像是 list 使用 append 添加元素，所以用 `prev + (i + 1, )` 來創建新 tuple。

```python
def find_spliced_motifs_dp(dna, motif):
    dp = [set() for _ in range(len(motif) + 1)]
    dp[0].add(())
    for i in range(len(dna)):
        for j in reversed(range(len(motif))):
            if dna[i] == motif[j]:
                for prev in dp[j]:
                    dp[j + 1].add(prev + (i + 1, ) )
            else:
                pass
    return dp[-1]
```

## 遞迴

至於遞迴，先擺著等著開悟= =。

```python
def find_spliced_motifs_rc(dna, motif):
    def search(start, path, index):
        if index == len(motif):
            return {path}
        else:
            pass
        results = set()
        for i in range(start, len(dna)):
            if dna[i] == motif[index]:
                results.update(search(i + 1, path + (i + 1, ), index + 1))
            else:
                pass
        return results
    return search(0, (), 0)
```