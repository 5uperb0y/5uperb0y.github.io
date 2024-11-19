---
title: ROSALIND｜Edit Distance Alignment (EDTA)
date: 2024-11-19 22:09:11
tags: rosalind 
categories:
---

比對兩蛋白質字串，計算其最小編輯距離，並以橫槓表示間隔呈現比對結果。

> Given: Two protein strings s and t in FASTA format (with each string having length at most 1000 aa).
>
> Return: The edit distance dE(s,t) followed by two augmented strings s′ and t′ representing an optimal alignment of s and t.

(https://rosalind.info/problems/edta/)

<!--more-->

## 背景知識

在生物資訊學中，字串比對 (alignment) 的目的是確定兩條字串的字符間有意義的對應關係。字符的關係（例如匹配、替換、插入與刪除）反映了基因突變或定序錯誤等現象，透過比對蛋白質或核酸序列，能識別兩序列相似的片段，從而推論生物的功能、結構和演化關係。比對的結果通常以矩陣、圖示或字串形式呈現，以便研究人員比較各區域的異同。舉例來說，兩條蛋白質序列可表示為以下形式，其中 "-" 表示間隔 (gap)。

```
ref:  PRETT-
read: PR-TTY
```

BAM 檔則以更簡約的 CIGAR 格式呈現，以下文字表示輸入序列與參考序列相比，依序有「2bp 匹配、1bp 刪除、2bp 匹配、1bp 插入」。
```
2M1D2M1I
```

長序列的比對則能以 Dotplot 則能呈現，例如：

![](https://upload.wikimedia.org/wikipedia/commons/3/33/Zinc-finger-dot-plot.png)
(以 dotplot 呈現兩條 DNA 序列各片段的相似性。圖/wikipedia)

序列比對的方法可以依據其範圍分為全域比對 (global alignment) 和局部比對 (local alignment)。全域比對確認整段序列的關係，強調整體的相似性；局部比對則尋找彼此對應的子序列，只著重序列間相似的片段。因此，全域比對適合比較長度相近且序列相似的同源序列，局部比對則能應用在序列歧異或長度有很大落差的序列。

一對序列可能存在多種比對方式，各自的差異可用編輯距離 (edit distance or Levenshtein distance) 衡量。編輯距離是一字串經過系列編輯操作轉變為另一字串的最少操作次數，這些操作包含替換、刪除與插入。生物體內各類突變的發生機率並不相同，計算編輯距離時也需要依據採用的演化模型對不同編輯操作進行加權，以吻合我們對這些分子生物機制的理解和假設。[BLOSUM](https://en.wikipedia.org/wiki/BLOSUM) 便依據資料庫內不同胺基酸互相轉換的比率，訂立各類替換事件的加權分數。

## 解題觀念

這題要求計算兩條蛋白質序列的編輯距離，並以字串呈現全域比對的結果。先前我曾介紹如何用[動態規劃計算編輯距離](https://5uperb0y.com/edit-distance/)。簡言之，這方法一次只考慮一個字符，判斷轉換當前兩條子序列所需的最簡操作來計算編輯距離。接著，比對下一個字符，並重複此過程直到算出完整序列的編輯距離。為了避免重複計算，這方法會記錄子序列間的編輯距離。

這方法在計算編輯距離的同時，也完成了序列比對。因此，我們只要像記錄編輯距離一樣，保留每次選擇的操作（例如插入、刪除或替換），最後就能據此重建比對結果。

## Python 實作

為了重建比對結果，我擴充了原先計算編輯距離的[程式碼](https://github.com/5uperb0y/rosalind/blob/main/code/edit/edit.py)，新增了每一步的編輯操作紀錄。首先建立兩個用雙層嵌套 list 來模擬的表格：`dists` 記錄 `S` 和 `T` 各子字串的最小編輯距離，`edits` 則記錄對應的操作（"0"：匹配/替換，"1"：插入，"2"：刪除）。假設 `S` 是參考序列，`T` 是對應序列。那麼，「刪除」表示 `S` 長度增長，而 `T` 長度不變，對應 `dists[s + 1][t]`；「插入」表示 `T` 長度增長，而 `S` 長度不變，對應 `dists[s][t + 1]`。

因此，初始化表格的時候，`edits[s][0]` 全為 2（刪除），`edits[0][t]` 全為 1（插入）。

計算完編輯距離後，可以參照 `edits` 從表格最右下角的位置 `edits[len(S)][len(T)]` 開始，依照編輯操作重建比對結果。我使用 `x` 和 `y` 追蹤當前的位置，並依據 `edits[x][y]` 的值決定下一步操作：

- `edit[x][y] == 0` 表示「匹配/替換」，`S[x]` 對應 `T[y]`，接著比對 `S[x - 1]` 和 `T[y - 1]`。
- `edit[x][y] == 1` 表示「插入」，"-" 對應 `T[y]`，接著比對 `S[x - 1]` 和 `T[y]`。
- `edit[x][y] == 2` 表示「刪除」，`S[x]` 對應 "-"，接續比對 `S[x]` 和 `T[y - 1]`。

為了簡化重建流程，我把下一個要比對的位置記錄在 `moves = [(1, 1), (0, 1), (1, 0)]`，依序為「匹配/替換」、「插入」和「刪除」，所以我能用 `edits[x][y]` 的值來取得對應操作，並用來更新 x 和 y 的位置。

```python
def align(S, T):
	dists = [ [0] * (len(T) + 1) for _ in range(len(S) + 1) ]
	edits = [ [-1] * (len(T) + 1) for _ in range(len(S) + 1) ] # 0: match, 1: insertion, 2: deletion
	for s in range(1, len(S) + 1):
		dists[s][0] = s
		edits[s][0] = 2
	for t in range(1, len(T) + 1):
		dists[0][t] = t
		edits[0][t] = 1
	# Fill DP tables
	for s in range(len(S)):
		for t in range(len(T)):
			if S[s] == T[t]:
				dists[s + 1][t + 1] = dists[s][t]
				edits[s + 1][t + 1] = 0
			else:
				choices = [dists[s][t], dists[s + 1][t], dists[s][t + 1]]
				dists[s + 1][t + 1] = min(choices) + 1
				edits[s + 1][t + 1] = choices.index(min(choices))
	# Reconstruct aligned sequences
	aln_s, aln_t = "", ""
	x, y = len(S), len(T)
	moves = [(1, 1), (0, 1), (1, 0)] # [match, insertion, deletion]
	while edits[x][y] >= 0:
		move_x, move_y = moves[edits[x][y]]
		aln_s = (S[x - 1] if move_x else "-") + aln_s
		aln_t = (T[y - 1] if move_y else "-") + aln_t
		x, y = x - move_x, y - move_y
	return([str(dists[-1][-1]), aln_s, aln_t])
```
