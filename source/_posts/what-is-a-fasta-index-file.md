---
title: Fasta index file (.fai) 是什麼？
date: 2024-10-15 21:21:21
tags: bioinformatics
categories:
---


使用參考基因體進行分析時，如果沒有提供對應的 Fasta index 檔 (*.fai*)，很有可能跳出以下這類錯誤訊息：

```
# HaplotypeCaller
ERROR MESSAGE: Fasta index file ref.fasta.fai for reference ref.fasta does not exist.
# freebayes
unable to find FASTA index entry for '1'
```

這個 *.fai* 究竟是什麼？為什麼許多分析需要它？

<!--more-->

# Fasta index file

*.fai* 是 Fasta 的索引，記錄了各序列的位置，能提升軟體擷取子序列的效率。假設我們想從一萬條 DNA 序列取出位於中間的鹼基。在沒有索引檔的情況下，每一次都得從第一個鹼基讀取，直到抵達指定位置後再回報結果（循序存取，sequential access）。循序存取的效率取決於 DNA　長度，因此其時間複雜度為 $O(n)$。

不過，如果事先掃描整個 Fasta 並建立索引檔，便能依據索引檔直接取用目標子序列，達到 $O(1)$ 的時間複雜度（隨機存取，random access）

# samtools faidx
[samtools faidx](https://www.htslib.org/doc/faidx.html) 可用於建立參考基因體的索引檔。使用時應確保 Fasta 每條序列等長（除了最後一列）。輸出檔名預設為原 Fasta 檔名加上 *.fai* 後綴。

```bash
samtools faidx ref.fa
```

以此指令建立的 *.fai* 共有五欄，

- NAME：序列名稱，即 ">" 之後的字串
- LENGTH：序列的鹼基總數
- OFFSET：由 0 開始計算，序列第一個鹼基為 FASTA 檔第幾個字符（含換行符）
- LINEBASES：每一列的鹼基總數
- LINEWIDTH：每一列的字符總數（含換行符）

舉例來說，如果換行符為 "\n"，那麼序列 s1 的總長為 10 鹼基，始於第 4 個字符，每一列有 5 鹼基、6 字符。

**example.fasta**
```
>s1
AAAAA
CCCCC
```

**example.fasta.fai**
```
#NAME   LENGTH   OFFSET  LINEBASES LINEWIDTH （實際不存在此列）
s1      10       4       5         6
```

獨到這裡，可能有人會問為什麼 OFFSET 是 4 而不是 1？這是因為，OFFSET 是序列第一個鹼基在整個檔案的位置，而非它在序列中的座標。因此，計算位置時連序列的名稱都要一起計算。從 0 開始計算，序列 s1 的一個鹼基正好位於第 4 個字符，所以 OFFSET 才會等於 4。

# Python 實作：模仿 bedtools getfasta
[bedtools getfasta](https://bedtools.readthedocs.io/en/latest/content/tools/getfasta.html) 的功能是依據 BED 檔從 Fasta 檔擷取子字串。BED 檔是紀錄區域的格式，普遍含有染色體、起始位置、終止位置和辨識碼四個欄位。比較需要留意的是，BED 檔的位置是從 0 開始計算，和 GTF 之類的格式不同。

```bash
$ cat ref.fa
>chr1
AAAAAAAACCCCCCCCCCCCCGCTACTGGGGGGGGGGGGGGGGGG

$ cat test.bed
chr1 5 10 myseq

$ bedtools getfasta -fi ref.fa -bed test.bed -tab
chr1 5 10 AAACC
```

以下使用 python 模仿 bedtools getfasta，利用 *.fai* 計算子序列在 Fasta 檔的位置，再透過 f.seek() 實踐隨機存取（參考[Complexity of f.seek() in Python](https://stackoverflow.com/questions/51801213/complexity-of-f-seek-in-python)）。簡言之，我首先讀取 BED 檔各列，存為 list 備用。*.fai* 則存為 dict，以 name 為 key，其它為 value，這樣就能依照 name 找到座標資訊。

每次讀進 BED 一個區域的 start 和 end，再用以下關係式找出該區域在 Fasta 檔的位置。

- 目標區域在 Fasta 的起始位置 = offset + linewidth * (start // linebases) + start % linebases
- 目標區域在 Fasta 的終點位置 = 子序列的起始位置 + region_length + end // linebases

實際執行時，先用 f.seek() 跳到起始位置，再用 f.read 讀到終點位置，刪除中間的換行符號，便能獲得目標區域的子序列了。

```python
def read_bed(path):
    bed = []
    with open(path, "r") as file:
        for line in file:
            chrom, start, end, name, *_ = line.strip().split()
            bed.append([chrom, int(start), int(end)])
    return bed
def read_fai(path):
    fai = {}
    with open(path, "r") as file:
        for line in file:
            name, length, offset, linebases, linewidth = line.strip().split()
            fai[name] = [int(length), int(offset), int(linebases), int(linewidth)]
    return fai
def get_fasta(fa_path, fai_path, bed_path):
    bed = read_bed(bed_path)
    fai = read_fai(fai_path)
    results = []
    for chrom, start, end in bed:
        length, offset, linewidth, linebases = fai[chrom]
        region_length = end - start
        region_start = offset + linewidth * (start // linebases) + start % linebases
        with open(fa_path, "r") as f:
            f.seek(region_start)
            substring = f.read(region_length + end // linebases).replace("\n", "")
            results.append([chrom, start, end, substring])
    return results
            
get_fasta("ref.fa", "ref.fa.fai", "test.bed")
```

最後附上 samtools 使用手冊和國外論壇的簡介供參考：

- [samtools faidx](https://www.htslib.org/doc/faidx.html)
- [what is in the fasta.fai](https://www.biostars.org/p/98885/#98966)
- [Can You Please Tell Me Where I Find Information About .Fai File Format?](https://www.biostars.org/p/1495/)
- 