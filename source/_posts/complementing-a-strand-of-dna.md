---
title: ROSALIND｜Complementing a strand of DNA (REVC)
date: 2024-05-03 19:11:32
tags: rosalind 
categories:
---

給定 DNA 序列，回傳其反向互補序列。
> Given: A DNA string s of length at most 1000 bp.
>
> Return: The reverse complement sc of s.

(https://rosalind.info/problems/revc/)

<!--more-->

# 知識點

2017 年交大某生物相關研究所入學考試其中一題為：「若 DNA 其中一股序列為 ATCGATCG，求其 complementary DNA 序列為何？ (A) AUCGAUCG (B) ATCGATCG (C) TAGCTACG (D) GCTAGCTA。」
解題的關鍵在了解分子生物的專有名詞定義，以下是各種 DNA 序列的簡介
- DNA: 去氧核醣核酸
- RNA: 轉錄 DNA 之產物
- complementary DNA (cDNA)：反轉錄 RNA 之產物
- complementary strand of DNA：DNA 兩股互為彼此的 complementary strand of DNA
- reverse complement of DNA：順序反轉的 complementary strand of DNA

若 DNA 在轉錄後未經加工，原則上對於 DNA 的其中一股而言有以下關係：
- DNA 序列 = complementary DNA 序列
- RNA 序列 = 把 T 換成 U 的 complementary strand of DNA 序列
- complementary DNA 的序列和 complementary strand of DNA 互補

因此，答案為 (B) ATCGATCG，若混淆 complementary DNA 和 complementary strand of DNA 會選到完全相反的答案。

# 題解

取得反向互補序列的任務可分解為替換鹼基和反轉序列順序。

## python

python 的字串可以像 list 一樣以索引值提取數值，因此得以 `str[::-1]` 取得反向序列，此表達式的涵義為「由字串的最後一個字符，一次一個取出所有字符」。而替換互補鹼基則要先建立密碼表（即 A to T, T to A, C to G, G to C），再參考密碼表替換字串中的字符。我使用的第一個策略是以 dictionary 儲存互補鹼基的密碼表，接著建立逐一替換輸入的序列。

```python
def reverse_complement(dna):
    """complementing a strand of dna
    """
    comp = {"A": "T", "T": "A", "C": "G", "G": "C"}
    return "".join(comp[nt] for nt in dna[::-1])
```

第二個方法則是使用 maketrans(old, new) 建立字符替換的密碼表，再以 translate() 依據密碼表同時替換字串中相應的字符。由於maketrans 是字串這個物件的靜態方法，所以需要在前面加字串來調用之。既然這字串跟副程式其它功能無關，就用空字串表示以免混淆，不然理論上用什麼字串都行，可參考 [Purpose of string in front of maketrans](https://stackoverflow.com/questions/64513034/purpose-of-string-in-front-of-maketrans)
```python
def reverse_complement(dna):
    """complementing a strand of dna
    """
    return dna[::-1].translate("".maketrans("ATCG", "TAGC"))

```

以下則是我最初的錯誤嘗試，原先打算連續使用 replace() 替換鹼基，但後來發現第一次迴圈的結果會被第二次會圈抵銷掉（第一次：A to T, 第二次 T to A），所以才會採用前述的方法迴避問題。
```
# WRONG METHOD!
code = {"A": "T", "T": "A", "C": "G", "G": "C"}
rcdna = dna
for i, j in code.items():
    rcdna = rcdna.replace(i, j)
print rcdna
```
## R

相較於 python，R 無法以索引值擷取字符，所以需要將字串轉變為其他格式再處理，以下方法參考 Four ways to reverse a string in R[https://www.r-bloggers.com/2019/05/four-ways-to-reverse-a-string-in-r/]

1. 利用 strsplit() 將字串分割為 list，再以 rev() 反轉 list 順序，最後使用 paste0() 將 list 合併為字串。
2. 利用 utf8ToInt() 將字串中 utf-8 編碼的字符轉變為 UTF-32 編碼的整數向量，再以 rev() 反轉向量順序，最後用 intToUtf8() 恢復 utf-8 編碼的字符組成之字串

```
dna <- "AAAACCCGGT"
# strsplit() method
reversed <- paste0(rev(strsplit(dna, "")[[1]]), collapse = "")
# utf8ToInt() method
reversed <- intToUtf8(rev(utf8ToInt(dna)))
```

由於 gsub() 只能把多種匹配到的字符串替換為一種字符串，
，所以得倚重可同時替換多種指定字符的 chartr()。  chartr() 的用法接近 bash 的 tr，會將字串中的 old (舊字符) 替換為 new (新字符)，例如 chartr("AC", "TG", str) 表示 A to T, C to G。
```
rcdna <- chartr(old = "ATCG", new = "TAGC", reversed)
print(rcdna)
```

## bash
bash 的作法較 python 和 R 簡潔，讀取字串，以 rev 反轉字串順序，再以 tr 替換互補鹼基等步驟可寫成一行。
```
dna="AAAACCCGGT"
echo "$dna"| rev | tr ATCG TAGC
```

