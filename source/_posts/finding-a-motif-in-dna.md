---
title: ROSALIND | Finding a Motif in DNA (SUBS)
date: 2024-05-09 06:39:46
tags: rosalind
categories:
---

## Problem
給定字串 s 與 t，其中 t 的長度小於等於 s，求 s 與 t 一致之子序列的起始位置。

> Given: Two DNA strings s and t (each of length at most 1 kbp).
>
> Return: All locations of t as a substring of s.

(https://rosalind.info/problems/subs/)

<!--more-->
# 知識點
Motif 是指在一範圍內反覆出現且具鮮明特質的物體或概念，其指涉的對象因脈絡而異 。在分子生物中，motif 被用以形容一段為生物體共享的核酸或蛋白質序列，這些序列可能有相似的功能，且對各生物都至關重要。在網路生物學中，motif 則可用以形容一出現頻率高於隨機分布的子圖，這些子圖的出現頻率很高且有獨特的網路結構，因此能在欠缺相關生物學知識的情況被辨識出來。而在結構生物學中，motif 則代表核酸或蛋白質上呈規律結構的區段，這些區段可能由多個二級結構所構成。

雖然 motif 定義多元，但皆不脫「重複」與「特別」兩項屬性。是以，透過在核酸或蛋白質序列尋找重複序列來辨識 motif，也算在找尋某種特別的屬性。在結構生物學中，domain 是與 motif 相關的概念。Domain 也具有規律的結構和鮮明的特質，但與 motif 不同，domain 能獨立發揮某項功能。換句話說，motif 強調結構的實體，而 domain 則強調功能的實體。

# 題解
此題意在尋找與 motif 相符的子字串之位置。在 Python 中，我首先建立 list 儲存子字串位置，接著以 for 迴圈依序取出長度與 motif 一致的子字串，再比對兩者的序列。若序列一致，則將子字串的起始位置儲存至 list。由於 python 計數由 0 開始，所以輸出結果還要 +1 才符合題目要求。因為最後要回傳 list，所以也能用 list comprehension 呈現。

```python
def subs(s, t):
	return [
		i + 1
		for i, _ in enumerate(s[:-len(t)])
		if s[i:i+len(t)] == t
	]
```

最後再把結果依照題目要求印出，

```python
print(" ".join(map(subs(s,t))))
```

在 R 中，我採取了另一種策略：先列出所有與 motif 長度一致的子字串，再回傳序列與 motif 相同的子字串位置。因此，我先定義了能從一字串取出所有特定長度子字串的 function。接著再以 grep 搜索子字串中，與 motif 相符者的位置。
```
# exemplified data
dna <- "GATATATGCATATACTT"
motif <- "ATAT"
# Generate kmer from a string
kmer <- function (s, k) {
  char <- unlist(strsplit(s, ""))
  kmer <- unlist(lapply(0:(nchar(s) - k), function (x) {paste0(char[seq(k) + x], collapse = "")}))
  return(kmer)
}
# identify motif from kmer with grep 
find_motif <- function (dna, motif) {
  mer <- kmer(dna, nchar(motif))
  return(grep(motif, mer))
}
```

# 討論
## 1. motif 相關知識來源
本文關於 motif 的定義參考以下文獻與網站
* https://www.merriam-webster.com/dictionary/motif
* [莊榮輝，生物化學基礎 Biochemistry Basics 2008] (http://juang.bst.ntu.edu.tw/BC2008/slides/Proteinx3a.htm)
* Pizzuti, & Rombo. (2018). Algorithms for Graph and Network Analysis: Clustering and Search of Motifs in Graphs. Encyclopedia of Bioinformatics and Computational Biology: ABC of Bioinformatics, 95.

## 2. R: kmer function
K-mers 是指一字串所有長度為 k 的子字串，長度為 l 之字串會有 l - k + 1 個 k-mers，此概念常用於基因體學的序列比對和品質篩選的演算法。

我的 kmer function 的運作方式是先將字串分割為獨立字符，再用 lapply 依序取出 k 個字符以 paste0 合併為長度為 k 的子字串。由於僅有 l - k + 1 個 k-mers，所以 lapply 迴圈僅執行  l - k + 1 次。

```
kmer <- function (s, k) {
  char <- unlist(strsplit(s, ""))
  kmer <- unlist(lapply(0:(nchar(s) - k), function (x) {paste0(char[seq(k) + x], collapse = "")}))
  return(kmer)
}
```

## 3. R: grep 和 grepl
grep 用以尋找文字向量中符合特定模式的元素之索引值
```
> grep(pattern = "A", x = c("A", "B", "C", "D", "A"))
[1] 1 5
```
grepl 用以判斷文字向量的各元素是否符合特定模式
```
> grepl(pattern = "A", x = c("A", "B", "C", "D", "A"))
[1]  TRUE FALSE FALSE FALSE  TRUE
```
由於此題要求回報子字串的位置，因此使用 grep 實踐之。