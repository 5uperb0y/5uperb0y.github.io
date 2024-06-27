---
title: ROSALIND｜Transcribing DNA into RNA (RNA)
date: 2022-02-03 15:55:48
tags: rosalind
categories:
---

模擬 DNA 轉錄 RNA 的過程，將給定 DNA 字串中的 T 替換為 U。

> Given: A DNA string t having length at most 1000 nt.
>
> Return: The transcribed RNA string of t.

(https://rosalind.info/problems/rna/)

<!--more-->

此題為字符替換問題，可使用 *str.replace(old, new, max)*，將字串中的 old（舊字符串） 替換為 new（新字符串）。若指定第三個參數 max，還可設定最大替換次數。
```python
def transcribe(dna):
	"""Transcribe DNA into RNA
	"""
	return dna.replace("T", "U")
```

R 語言與 replace 相應的 function 為 gsub，然而 gsub 可以接受向量，批次替換字符串。
```r
dna <- c("GATGGAACTTGACTACGTAAATT", "AAATTTT")
rna <- gsub(pattern = "T", replacement = "U", x =  dna)
print(rna)
```

bash 的 tr (transform) 可用於替換、刪除、修改等多種字符操作。此例使用的是其替換功能，參數設定為：$string | tr old new，意指將字串中的 old（舊字符） 替換為 new（新字符）。相較於 R 的 gsub() 和 python 的 replace()，tr 只針對字符的操作（即只能 A→T，不能 AT→CG），若要替換字符串則要使用 sed，其參數設定為：sed "s/old/new/g"。

```bash
# Transcribing DNA into RNA using tr
dna="GATGGAACTTGACTACGTAAATT"
rna=$(echo "$dna" | tr T U)
echo "$rna"

# Translating RNA into protein using sed
dna="GGTGGTGGTGGT"
rna=$(echo "$dna" | tr T U)
protein=$(echo "$rna" |  sed "s/GGU/G/g")
echo "$protein"
```