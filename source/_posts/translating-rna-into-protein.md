---
title: ROSALIND｜Translating RNA into Protein (PROT)
date: 2024-06-09 21:32:14
tags: rosalind
categories:
---


轉譯RNA字串為蛋白質字串

> Given: An RNA string s corresponding to a strand of mRNA (of length at most 10 kbp).
>
> Return: The protein string encoded by s.

(https://rosalind.info/problems/prot/)

<!--more-->

# 背景知識

mRNA 紀錄了由 DNA 轉錄而來的基因資訊，其上每三個核苷酸會對應到一個胺基酸或終止密碼子。在轉譯時，tRNA 會攜帶對應的胺基酸，以 mRNA 為模板，於核醣體合成蛋白質。

- genetic code：核苷酸 codon 及其轉譯時對應的胺基酸。
- codon：密碼子，解讀遺傳密碼表的單位。mRNA 上每三個核苷酸為一個 codon。由於有 A、U、C、G 四種核苷酸，所以存在 4^3 = 64 種 codon，對應著 20 種胺基酸和終止密碼子。
- anticodon：即 codon 的互補序列。
- start codon：起始密碼子，代表轉譯開始的訊號，會轉譯出 methionine。AUG 是常見的 start codon，但也有其他序列可啟動轉譯。
- stop codons：終止密碼子，代表轉譯停止的訊號，不會轉譯出胺基酸。對應的核苷酸序列為 UAA/UAG/UGA。

# 解題觀念

轉譯的過程可分解為：構建密碼表、分割 mRNA 序列、查找密碼表、處理終止密碼子。

- 構建密碼表：遺傳密碼需要以特定的資料型態（例如：list、hash table、table 等），儲存的型態會影響翻譯的方式。
- 分割 mRNA 序列：將 mRNA 切成三個一組的 codon，以逐一查找密碼表。
- 查找密碼表：逐一查詢 codon 對應的胺基酸，再傳接所有得出的胺基酸。
- 處理終止密碼子：由於終止密碼子不轉譯胺基酸，所以要移除終止密碼子以後的序列。

# 題解
## Python
密碼表可以 dictionary 儲存。python 的 dictionary 由數個 items 組成，每個 item 則包含 key 和 value 兩部分：value 是欲儲存的資訊，其資料型態可為數值、字串、list 甚或其他的 dictionary (nested dictionary)；而 key 則為查找 value 的依據，每個 key 對應一個 value。

dictionary 的特性是item無序性和資料型態可變性，前者意味著 dictionary 的 item 間沒有依照順序排列，因此無法依照索引取值；後者則是指可以透過新增、刪除、取代等操作改變 dictionary 內各 item 的內容。

詳細的 dictionary 介紹可參考[Python字典（dictionary）基礎與16種操作](https://selflearningsuccess.com/python-dictionary/)和[Python Dictionary完全教學一次搞懂](https://www.learncodewithmike.com/2019/12/python-dictionary.html)。在此題中，主要是應用 dictionary 的 key 和 value 互相查找的特性來儲存密碼表。

首先，依照取值策略，可以兩種方式儲存密碼表。第一種取值策略為正向索引，即透過 key 來查找 value，可以 codon 為 key，胺基酸為 value 儲存，轉譯時每三個字符從 dictionary 取值。在這張密碼表中，終止密碼子對應到空字串，表示它們不會轉譯出任何胺基酸。

### Get values by keys
```python
def translate(rna):
    """Translate a RNA string into a proten string

    Args:
        rna (str): A RNA string composed of nucleotides ('U', 'C', 'A', 'G'), length must be a multiple of three.

    Returns:
        str: Amino acid sequence derived from the RNA
    """
    code = {
    "UUU": "F", "UUC": "F",
    "UUA": "L", "UUG": "L", "CUU": "L", "CUC": "L", "CUA": "L", "CUG": "L",
    "AUU": "I", "AUC": "I", "AUA": "I",
    "AUG": "M",
    "GUU": "V", "GUA": "V", "GUC": "V", "GUG": "V",
    "UCU": "S", "UCC": "S", "UCA": "S", "UCG": "S", "AGU": "S", "AGC": "S",
    "CCU": "P", "CCC": "P", "CCA": "P", "CCG": "P",
    "ACU": "T", "ACC": "T", "ACA": "T", "ACG": "T",
    "GCU": "A", "GCC": "A", "GCA": "A", "GCG": "A",
    "UAU": "Y", "UAC": "Y",
    "CAU": "H", "CAC": "H",
    "CAA": "Q", "CAG": "Q",
    "AAU": "N", "AAC": "N",
    "AAA": "K", "AAG": "K",
    "GAU": "D", "GAC": "D",
    "GAA": "E", "GAG": "E",
    "UGU": "C", "UGC": "C",
    "UGG": "W",
    "CGU": "R", "CGA": "R", "CGC": "R", "CGG": "R", "AGA": "R", "AGG": "R",
    "GGU": "G", "GGA": "G", "GGC": "G", "GGG": "G",
    "UGA": "", "UAG": "", "UAA": ""
    }
    pos = 0
    protein = ""
    while code.get(rna[pos: pos + 3], ""):
        protein = protein + code.get(rna[pos: pos + 3], "")
        pos = pos + 3
    return protein
```

終止密碼子會讓轉譯進度提前結束，為了處理這種情況，我使用 while 來控制迴圈的執行。

- 轉譯到終止密碼子：終止密碼子對應到空字串，於是迴圈停止。
- 轉譯到RNA尾端：list slicing 在索引值超出範圍時不會報錯，而只會回傳空字串，於是迴圈照樣停止。

透過這兩種機制，我就能讓轉譯在碰到終止密碼子或RNA尾端時自然結束。

### Get keys by values (in list form)

第二種取值策略則為逆向索引，即透過 value 來查找 key，可以胺基酸為 key，codon 為 value 儲存。由於 value 為 list 形式，無法直接取值，所以定義 function 來判斷 codon 是否位於 value 並回傳對應的 Key：

```python
def find_val(d, target):
    for key, val in d.items():
        if target in val:
            return key
```
而實踐轉譯的方式則是利用 for 依序找出所有 codon 對應的胺基酸，並在每次迴圈延伸已知的蛋白質序列。相較於第一種方式，這方法會用到兩個迴圈，第一個迴圈用以尋找 value，第二個迴圈則用以取出所有 codon 對應的 key。

```python
code = {
    "F": ["UUU", "UUC"],
    "L": ["UUA", "UUG", "CUU", "CUC", "CUA", "CUG"],
    "I": ["AUU", "AUC", "AUA"],
    "M": ["AUG"],
    "V": ["GUU", "GUA", "GUC", "GUG"],
    "S": ["UCU", "UCC", "UCA", "UCG", "AGU", "AGC"],
    "P": ["CCU", "CCC", "CCA", "CCG"],
    "T": ["ACU", "ACC", "ACA", "ACG"],
    "A": ["GCU", "GCC", "GCA", "GCG"],
    "Y": ["UAU", "UAC"],
    "H": ["CAU", "CAC"],
    "Q": ["CAA", "CAG"],
    "N": ["AAU", "AAC"],
    "K": ["AAA", "AAG"],
    "D": ["GAU", "GAC"],
    "E": ["GAA", "GAG"],
    "C": ["UGU", "UGC"],
    "W": ["UGG"],
    "R": ["CGU", "CGA", "CGC", "CGG", "AGA", "AGG"],
    "G": ["GGU", "GGA", "GGC", "GGG"],
    "" : ["UGA", "UAG", "UAA"]
    }

def translate(rna, code):
    p = ""
    for i in [s[i:i+3] for i in range(0, len(s), 3)]:
        p += find_val(code, i)
    return p
```
## R
在 R 中，則可以用 list 儲存密碼表。 list 可以儲存不同類型的資料型態（例如：整數、浮點數、邏輯、向量等），每個儲存的元素皆有其編號，可以透過元素的索引值取值。除了固有的編號，也能於創建 list 時為元素命名，以便使用名稱取值。

在此題中，是利用 list 能同時紀錄內容和其名稱的特性來儲存密碼表。

```r
# extract a element by index
> l <- list("dog", TRUE, 5)
> print(l[[2]])
[1] TRUE
# extract a element by name
> l <- list("A" = "dog", "B" = TRUE, "C" = 5)
> print(l[["A"]])
[1] "dog"
```

示範數據如下：
```r
rna <- "AUGGCCAUGGCGCCCAGAACUGAGAUCAAUAGUACCCGUAUUAACGGGUGA"
```
首先，定義將 mRNA 序列三個一組分為 codon 的副程式。簡言之，以 strsplit 將字串切割為由 list 儲存且長度為三字符的一批短字串[4]，再用 unlist 將之轉換為 vector [5]。 其他寫法可參考 [Split Character String into Chunks in R (2 Examples)](https://statisticsglobe.com/split-character-string-into-chunks-in-r)

```r
chunks <- function(s, n) {
  unlist(strsplit(s, paste0("(?<=.{", n, "})"), perl = TRUE))
}
```

接著同樣可依照取值策略循兩種方式儲存密碼表。第一種是正向索引，以 codon 為名，胺基酸為元素，將密碼表存為 list。接著，以 lapply 依序對照密碼表，取出 codon 對應的胺基酸字符，再用 paste0 接合為蛋白質字串。
```r
code <-  
  list(
  "UUU"= "F", "UUC"= "F",
  "UUA"= "L", "UUG"= "L", "CUU"= "L", "CUC"= "L", "CUA"= "L", "CUG"= "L",
  "AUU"= "I", "AUC"= "I", "AUA"= "I",
  "AUG"= "M",
  "GUU"= "V", "GUA"= "V", "GUC"= "V", "GUG"= "V",
  "UCU"= "S", "UCC"= "S", "UCA"= "S", "UCG"= "S", "AGU"= "S", "AGC"= "S",
  "CCU"= "P", "CCC"= "P", "CCA"= "P", "CCG"= "P",
  "ACU"= "T", "ACC"= "T", "ACA"= "T", "ACG"= "T",
  "GCU"= "A", "GCC"= "A", "GCA"= "A", "GCG"= "A",
  "UAU"= "Y", "UAC"= "Y",
  "CAU"= "H", "CAC"= "H",
  "CAA"= "Q", "CAG"= "Q",
  "AAU"= "N", "AAC"= "N",
  "AAA"= "K", "AAG"= "K",
  "GAU"= "D", "GAC"= "D",
  "GAA"= "E", "GAG"= "E",
  "UGU"= "C", "UGC"= "C",
  "UGG"= "W",
  "CGU"= "R", "CGA"= "R", "CGC"= "R", "CGG"= "R", "AGA"= "R", "AGG"= "R",
  "GGU"= "G", "GGA"= "G", "GGC"= "G", "GGG"= "G",
  "UGA"= "", "UAG"= "", "UAA"= ""
)
transl <- function (rna, code) {
  return(paste0(lapply(chunks(rna, 3), function(x) {code[[x]]}), collapse = ""))
}
```
第二種則是逆向索引，以胺基酸為名，codon 為元素，將密碼表存為 list。要注意的是，由於無法以空字符命名，所以這種儲存方法要把終止密碼子命名為其他符號，最後再把終止密碼子的符號從輸出的蛋白質序列中刪除。

簡言之，利用 lapply 遍歷所有 codon，以 grep 判斷 codon 位於 list 何處，藉此取出對應的胺基酸字符，使用 paste0 接合為蛋白質字串。最後以 gsub 配合正則表達式移除終止密碼子的符號及其後的序列。

```r
code <- 
  list("F" = c("UUU", "UUC"),
       "L" = c("UUA", "UUG", "CUU", "CUC", "CUA", "CUG"),
       "I" = c("AUU", "AUC", "AUA"),
       "M" = c("AUG"),
       "V" = c("GUU", "GUA", "GUC", "GUG"),
       "S" = c("UCU", "UCC", "UCA", "UCG", "AGU", "AGC"),
       "P" = c("CCU", "CCC", "CCA", "CCG"),
       "T" = c("ACU", "ACC", "ACA", "ACG"),
       "A" = c("GCU", "GCC", "GCA", "GCG"),
       "Y" = c("UAU", "UAC"),
       "H" = c("CAU", "CAC"),
       "Q" = c("CAA", "CAG"),
       "N" = c("AAU", "AAC"),
       "K" = c("AAA", "AAG"),
       "D" = c("GAU", "GAC"),
       "E" = c("GAA", "GAG"),
       "C" = c("UGU", "UGC"),
       "W" = c("UGG"),
       "R" = c("CGU", "CGA", "CGC", "CGG", "AGA", "AGG"),
       "G" = c("GGU", "GGA", "GGC", "GGG"),
       "*" = c("UGA", "UAG", "UAA")
       )
transl <- function(rna, code) {
  p <- paste0(lapply(chunks(rna, 3), function(x) {names(code)[grep(x, code)]}), collapse = "")
  p <- gsub("\\*.*?$","", p)
  return(p)
}
```

# 討論

## Biology: Translation
遺傳密碼的模式有很多故事可以探討，例如「為什麼遺傳密碼以三個一組為單位？」、「為什麼遺傳密碼與蛋白質非一對一關係？」、「為什麼遺傳密碼中，第三個鹼基比較無專一性？」等問題，這些問題反映了生命演化出遺傳密碼的蛛絲馬跡。這些資訊可參考書籍《生物決定論》或 Koonin & Novozhilov. (2009). Origin and evolution of the genetic code: the universal enigma. IUBMB life, 61(2), 99-111.

而遺傳密碼破譯前，也有許多高明的見解和理論想解釋轉譯的模式，可以參考《創世第八天》。

## Python: List comprehension
List comprehension 可簡化程式碼，將 for 迴圈輸出值儲存到 list 中，其結構為：
```
[expression for item in iterable (if-else statement)]
```
- expression：可為運算式或取值
- item：Iterable Object 之元素
- iterable：Iterable Object
- if-else statement：條件判斷，即 item 符合條件才執行 expression

以下列出不同類型 expression 的範例：

value extraction
```python
> l = ["a", "b", "c", "d", "e"]
> o = [l[i] for i in range(0, len(l), 2)]
['a', 'c', 'e']
```
computation
```python
> n = [i*3 for i in range(5)] 
> print n
[0, 3, 6, 9, 12]
```
computation with if-else statement
```python
> n = [i*3 for i in range(5) if i > 2] 
> print n
[9, 12]
```

詳細介紹可參考 [Python Comprehension語法應用教學](https://www.learncodewithmike.com/2020/01/python-comprehension.html)

## Python: "".join() 的使用方法
join() 可將 list 內的各元素依照指定分隔符連接為字串，寫法為

```python
# "sep".join(list)
> print ",".join(["A", "B", "C"])
A,B,C
> print "_".join(["A", "B", "C"])
A_B_C
print "".join(["A", "B", "C"])
ABC
```
## R: strsplit 如何切割無符號分隔的字串？
strsplit 可依照指定分隔符將字串切割為一批以 list 儲存的短字串，使用方法為：
```r
# strsplit(s, sep)
> strsplit("A,B,C,D", ",")
[[1]]
[1] "A" "B" "C" "D"
> strsplit("A_B_C_D", "_")
[[1]]
[1] "A" "B" "C" "D"
```

倘若字串中沒有規律出現的符號（例如：mRNA 序列，"AUCGAUCGA"），可以使用正則表達式的 positive lookbehind 概念來解決。

positive lookbehind 的形式如下，意思是尋找前位字串符合 pattern 的 y。
```
(?<=pattern)y
```
例如在以下範例中，就是把前位字符為「1」的「A」取代為「B」，而前位字符不符合條件者則不受影響
```r
> gsub("(?<=1)A", "B", "1A2A3A1A2A3A", perl = TRUE)
[1] "1B2A3A1B2A3A"
```
在了解 positive lookbehind 的意思之後，即可解釋以下程式碼的運作原理。
```r
> strsplit("AUCGAUCGA", "(?<=.{3})", perl = TRUE)
[[1]]
[1] "AUC" "GAU" "CGA"
```

此處 「(?<=.{3})」 的涵義其實是 「(?<=.{3})""」，表示「尋找前位為字串長三個字符的空字符」， 換句話說，此程式碼先以 positive lookbehind 表達式取出預計切割位置的空字符「""」，再以 strsplit 將這些空字符當作分隔符，將輸入字串切割為一批短字串。

## R: 為什麼要以 unlist 處理 strsplit 的輸出結果？
strsplit 原始輸出為 list 形式，所以需要使用 unlist 將之轉換為 vector 以利後續處理
```r
> strsplit("A,B,C,D", ",")
[[1]]
[1] "A" "B" "C" "D"
> unlist(strsplit("A,B,C,D", ","))
[1] "A" "B" "C" "D"
```
