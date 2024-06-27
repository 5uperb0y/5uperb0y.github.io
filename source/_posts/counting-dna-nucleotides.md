---
title: ROSALIND｜Counting DNA Nucleotides (DNA)
date: 2024-05-01 23:00:05
tags: rosalind 
categories:
---

給定 DNA 字串，依照 "A"、"C"、"G"、"T" 的順序，印出四種鹼基符號的數量。

> Given: A DNA string s of length at most 1000 nt.
> 
> Return: Four integers (separated by spaces) counting the respective number of times that the symbols 'A', 'C', 'G', and 'T' occur in s.

(https://rosalind.info/problems/dna/)

<!--more-->

在此例使用 dictionary 的 key 記錄鹼基符號，利用 value 記錄出現頻率，透過迴圈遍歷整個字串，每逢與 key 相同的字符即增加該 key 之 value。

儘管統計鹼基的方法很直觀，但輸出的順序要留意。題目要求以 A、C、G、T 之順序輸出鹼基出現頻率，但 python 的 *dictionary* 為無排序的雜湊表，所以值不會依照字母的順序或定義時的順序印出。為了回傳排序後的統計量，要在輸出前調整順序。

```python
def nt_freq(dna):
    """Counting DNA nucleotide
    """
    freq = {"A": 0, "T": 0, "C": 0, "G": 0}
    for nt in dna:
        freq[nt] = freq[nt] + 1
    output = " ".join( str(freq[nt]) for nt in "ACGT" )
    return output
```

也可以不自己寫迴圈，套用 python 的字串方法 *count()* 來統計特定鹼基的數量。
```python
def nt_freq(dna):
    """Counting DNA nucleotide
    """
    return " ".join(str(dna.count(nt)) for nt in "ACGT")
```

shell script 要尋找字串也有許多寫法，其中一種做法是透過 grep -o 取得匹配的鹼基，再傳給 wc -l 計算列數（即字串中，指定鹼基符號的數量）。四種鹼基符號的統計值儲存於 array 並以空格間隔，以便輸出時辨識 array 各項目的數值。
```bash
s="AGCTTTTCATTCTGACTGCAACGGGCAATATGTCTCTGTGTGGATTAAAAAAAGAGTGTCTGATAGCAGC"
d=()
for ch in A C G T;
do
    d+=$(echo $(echo $s | grep -o $ch | wc -l) "")
done
printf  "${d[@]}"
```
