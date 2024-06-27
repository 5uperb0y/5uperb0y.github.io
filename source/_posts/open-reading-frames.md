---
title: ROSALIND｜Open Reading Frames (ORF)
date: 2024-06-11 22:11:25
tags: rosalind
categories:
---

給定一條以 FASTA 格式儲存的 DNA 序列，依據它的 open reading frames，列出所有可能由這條 DNA 序列轉譯出的蛋白質序列。

> An open reading frame (ORF) is one which starts from the start codon and ends by stop codon, without any other stop codons in between. Thus, a candidate protein string is derived by translating an open reading frame into amino acids until a stop codon is reached.
>
> Given: A DNA string s of length at most 1 kbp in FASTA format.
>
> Return: Every distinct candidate protein string that can be translated from ORFs of s. Strings can be returned in any order.

(https://rosalind.info/problems/orf/)

<!--more-->

# 背景知識

在合成蛋白質時，核醣體會以三個核苷酸一組讀取 RNA 序列，添加各組核苷酸對應的胺基酸。核醣體開始讀取 RNA 序列的位置，會影響核苷酸的分組方式以及隨後轉譯出的蛋白質種類。由於 RNA 密碼子的長度為三，所以依據開始讀取序列的位置，每條 RNA 都會有三種劃分核苷酸的方式。

如下所示，這樣將核酸序列依照密碼子長度劃分，讓核醣體讀取的方式，被稱作 reading frame。每個 reading frame 都定義了一個從不同核苷酸開始讀取的密碼子序列。

```
AAA GGG CCC
A AAG GGC CC
AA AGG GCC C
```

相較於 RNA，DNA 兩股序列的任一股都可能轉錄為 RNA 並且轉譯為蛋白質。因此，每條 DNA 有六種可能的 reading frame，其中三種來自順向股，另外三種則來自逆向股的 reverse complemnt。DNA 這六種 reading frame 對應的蛋白質序列叫做 six-frame translating sequences。

以下表格呈現了 DNA 序列 `AAAGGGCCC` 的 six-frame translating sequences，感興趣的人可以到[這個網頁](https://www.bioline.com/media/calculator/01_13.html)試試看怎麼計算。

|         | translation in forward direction | translation in reverse direction |
| ------- | -------------------------------- | -------------------------------- |
| frame 1 | KGP                              | GPF                              |
| frame 2 | KG                               | GP                               |
| frame 3 | RA                               | AL                               |

Open reading frames (ORFs) 則是指 DNA 序列中，可能轉錄為 RNA 並且轉譯為蛋白質的片段。具體地說，ORFs 是由起始密碼子（`ATG`）延伸至終止密碼子（`TAG`、`TGA`、`TAA`）的區域。

辨識基因體 ORFs 和 six-frame translating sequences 是尋找蛋白質譯碼基因的重要步驟。在許多臨床與研究場合，DNA 資料比起 RNA 或蛋白質資料更容易取得（或更經濟實惠），研究人員只能依賴 ORFs 和 six-frame translating sequences 來預測蛋白質譯碼基因，已進行功能分析或是基於蛋白質序列比較的親緣關係分析。

例如在癌症基因檢測中，通常只對部分腫瘤熱點的 DNA 進行定序，再透過變異分析和註解來研究致病或治療相關的基因體功能變化。在微生物總基因體研究（metagenomics）中，也是透過定序環境 DNA 以及隨後的物種註解來普查群落的組成。

此外，在缺乏足夠基因體註解資訊的情況下，ORFs 和 six-frame translating sequences 能成為尋找新的譯碼基因或預測基因功能的依據。而相較於 six-frame translating sequences，ORFs 更有機會反映真實的蛋白質譯碼基因，所以使用 ORFs 進行後續分析時，能減少數據分析的雜訊，降低計算的複雜性。

# 解題觀念

這題要求從 FASTA 檔案讀取一條 DNA 序列，再列出它所有的 ORFs 對應的蛋白質序列。這一種情況與[轉譯 mature mRNA 那題](https://5uperb0y.com/translating-rna-into-protein/)的情境不同。在真核生物的基因體中，有許多不參與蛋白質表現的 DNA，例如內含子、偽基因和調控子等非譯碼 DNA （non-coding DNA，它們有些不會被轉錄，有些則在轉錄後被切除，這導致 DNA 序列和實際會被轉譯出蛋白質的 mature mRNA 序列有所不同。

即使 mRNA 能忠實地保留 DNA 的大部分序列，read frame 也取決於起始密碼子的位置。以 DNA 序列 `ATGCATGTAA` 為例，從前後兩個起始密碼子轉譯會產生兩種不同的蛋白質序列，它們各自來自不同的 read frame。因此，直接從 DNA 序列預測蛋白質序列時，必須考慮所有可能的 read frames，以確保不遺漏任何潛在的 ORFs。

```
# 從第一個起始密碼子開始轉譯
ATG CAT GTA A
# 從第二個起始密碼子開始轉譯
. ... ATG TAA
```

在此提供一個辨識 ORFs 的策略：
1. 由 5' 到 3' 一一讀取 DNA 序列。
2. 讀到起始密碼子時，暫存它的座標，再記錄當下的 read frame。
3. 如果碰到多個起始密碼子，都按步驟 2. 依照所在的 read frame 儲存它們的座標。
4. 一旦碰到終止密碼子，就取出對應 read frame 的起始密碼子座標，直到取出所有暫存的座標。
5. ORFs 正是終止密碼子與起始密碼子之間的區域
6. 持續讀取 DNA 序列直到最後一個核苷酸為止。

接著再用同樣的流程讀取反向互補序列，就能得到 DNA 所有 read frame 的 ORFs。最後根據 DNA 密碼表把 ORFs 序列轉換為對應的蛋白質序列即可滿足題目要求。

# Python 實作

因為完成這題所需的幾項功能在先前的文章已經解釋過了，所以接下來我只會解釋要怎麼用 python 實作 ORFs 的辨識。解題的完整程式碼可參考[我的 github](https://github.com/5uperb0y/rosalind/blob/main/code/orf/orf.py)。

| 功能             | 題目                          | 連結                                                        |
| ---------------- | ----------------------------- | ----------------------------------------------------------- |
| 讀取 FASTA 檔案  | Finding a Shared Motif        | [Link](https://5uperb0y.com/finding-a-shared-motif/)        |
| 取得反向互補序列 | Complementing a Strand of DNA | [Link](https://5uperb0y.com/complementing-a-strand-of-dna/) |
| 轉譯RNA序列      | Translating RNA into Protein  | [Link](https://5uperb0y.com/translating-rna-into-protein/)  |

我使用 orf_locs (list) 儲存所有 ORFs 的座標。為了暫存不同 read frame 上發現的起始密碼子座標，我建立了一個由 3 個 lists 組成的 list，start_locs。其中的三個 list 分別對應 DNA 序列三種可能的 read frame (0, 1, 2)。

在讀取 DNA 序列的過程中，透過將當前核苷酸的位置索引值對 3 取模數 ($position \mod 3$)，可以得知當前所在的 read frame 索引值。一旦遇到起始密碼子，將其座標存入 start_locs 對應 read frame 的 list；直到碰到終止密碼子，才把 start_locs 對應 list 的起始密碼子座標清空，與當前終止密碼子的座標組成 ORFs 區段，存入 orf_locs。

讀取完整條序列後，再根據 ORFs 座標擷取所有 ORFs 對應的 DNA 子序列。

```python
def orf(dna):
    """
    Identifies all open reading frames (ORFs) from all read frames in a DNA sequence
    that start with 'ATG' and end with a stop codon ('TAA', 'TAG', 'TGA').

    Args:
        dna (str): A DNA sequence.

    Returns:
        list: A list of ORF sequences.
    """
    orf_locs = []
    start_locs = [[], [], []]
    for pos, _ in enumerate(dna):
        frame = pos % 3
        codon = dna[pos: pos + 3]
        if codon == "ATG":
            start_locs[frame].append(pos)
        elif codon in ["TGA", "TAG", "TAA"]:
            while start_locs[frame]:
                orf_locs.append((start_locs[frame].pop(), pos))
        else:
            pass
    return [ dna[i:j] for i, j in orf_locs ]
```

# 討論：open reading frame 的定義

Open reading frame 一詞的字面意思是："Frames that remain open to keep ribosome reading"，指的是核酸序列中未被終止密碼子中斷的一段區域。由於 ORFs 主要用於預測蛋白質譯碼基因，所以也衍伸出許多能用於設計演算法的操作型定義 (Sievar et al., 2018)

- 定義1：ORFs 是指長度為 3 的倍數，而且介於起始密碼子和終止密碼子之間的序列。
- 定義2：ORFs 是指長度為 3 的倍數，而且介於兩組終止密碼子之間的序列。
- 定義3：ORFs 是介於核酸裁切位點之間的序列。

![](https://ars.els-cdn.com/content/image/1-s2.0-S0168952517302299-gr1.jpg)

（當中橘色線條標記的區域即為各種定義下的 ORFs，圖片來源：Sieber et al., 2018.）

在這些定義中，第一種最為常見，包含這道題目都採用起始和終止密碼子來定義 ORFs 的範圍。定義 1 適用於基因體結構相對單純的原核生物，但在應用於真核生物時會有些問題。

首先，位於內含子中的終止密碼子會導致 ORFs 短於實際可以轉譯出蛋白質的序列。其次，由於起始密碼子既作為開始轉譯的訊號，也是甲硫胺酸的密碼子，定義 1 在實務上難以區分這兩種狀況。

相較之下，定義 2 只考慮終止密碼子，因此可以避免起始密碼子帶來的混淆問題。此外，定義 2 所構成的 ORFs 能夠較全面涵蓋到外顯子的位置，因此 Sievar et al. 推薦使用定義 2。至於第三種定義，則要仰賴基因體註解資訊（通常是在基因預測之後的步驟），而且只能用於真核生物，並不是偵測 ORF 演算法的常見方式。

Sieber et al. (2018). The definition of open reading frame revisited. Trends in Genetics, 34(3), 167-170.