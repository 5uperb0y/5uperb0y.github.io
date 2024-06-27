---
title: ROSALIND | Finding a Shared Motif (LCSM)
date: 2024-05-11 18:44:14
tags: rosalind
categories:
---

給定一 FASTA 檔案，內含數條由 A、C、G、T 組成的字串，求這些字串共享的最長子字串。

> A common substring of a collection of strings is a substring of every member of the collection. We say that a common substring is a longest common substring if there does not exist a longer common substring. For example, "CG" is a common substring of "ACGTACGT" and "AACCGTATA", but it is not as long as possible; in this case, "CGTA" is a longest common substring of "ACGTACGT" and "AACCGTATA".
> Note that the longest common substring is not necessarily unique; for a simple example, "AA" and "CC" are both longest common substrings of "AACC" and "CCAA".
>
> Given: A collection of k (k≤100) DNA strings of length at most 1 kbp each in FASTA format.
>
> Return: A longest common substring of the collection. (If multiple solutions exist, you may return any single solution.)

(https://rosalind.info/problems/lcsm/)

<!--more-->

# 知識點
如果我們能夠透過理論計算或是資料庫數據得知某種 motif 的組成，便能按照 [Finding a motif in dna](https://5uperb0y.com/finding-a-motif-in-dna/) 這題的要求，在一條 DNA 中尋找已知 motif 的位置。然而，並非所有研究對象都能事前得知 motif 的資訊，所以更常見的方法是透過比較多組基因體，歸納出反覆出現且序列相似的 motif。

為了簡化問題，這題只要求找出最長的 motif，因為長度愈長，motif 涉及的功能往往越多。此外，題目還假設同一種 motif 的序列完全一致，這意味著這些 motif 在演化中沒發生任何突變，暗示其功能相當重要，具有相當的穩定性。換句話說，這些 motif 可能反映了各基因體共享的核心功能。

# 題解

這題相當於尋找各字串的最長共同子字串（longest common substring, LCS）。最直觀的解法在於了解「 LCS 的長度不大於最短字串的長度」，因此可以列出最短字串的所有子字串，再拿這些子字串由長至短一一與其它字串比較。第一個出現在所有字串的子字串就是題目要求的 LCS。

以下使用 python 實作這個算法，首先找到最短的子字串 `min_s`，求出它所有的子字串 `kmer`，最後由長至短檢查子字串是否出現在所有字串中，一旦符合條件，便直接回傳結果。（由於這裡是由短至長生成 `kmer`，所以後面把它倒置 `kmer[::-1]` 來改變比對的順序。）

```python
def lcs(strs):
    """Finding the longest common substring from multiple strings
    """
    min_s = min(strs, key = len)
    kmer = [
        min_s[i:i+k+1]
        for k in range(len(min_s) + 1)
        for i in range(len(min_s) - k)
    ]
    for mer in kmer[::-1]:
        if all(mer in s for s in strs):
            return mer
        else:
            pass
    return ""
```

由於題目要求輸入 FASTA 檔案，所以需要一個讀檔的副程式。FASTA 檔案是紀錄序列的通用文字格式，`>` 開頭的列紀錄 DNA、RNA或蛋白質序列的名稱，直到碰到下一個 `>` 符號之前的列則記錄了序列本身。為了方便在終端機用肉眼判讀，有些 FASTA 檔案中的序列會分成數列表示。

```
>Read_id
ACTGACTGATCGGATAGTAGTGATAGCGATCGATCAGCATATCGACTATCAGCTAGCTACAGCTGAGCATCGATCG
ACGATTACGACGATCAG
```

以下程式將 FASTA 讀進 python dictionary。凡是 `>` 開頭的列，就把它當作 dictionary 的 key，而其餘的列則串接起來成為 dictionary 的 value。

```python
def read_fasta(path):
    """Read a fasta file into a dictionary
    """
    with open(path, "r") as f:
        reads = {}
        for line in f:
            if line.startswith(">"):
                key = line[1:].strip()
                reads[key] = ""
            else:
                reads[key] = reads[key] + line.strip()
        return reads
```

結合讀檔與 LCS 主程式，便能滿足題目的要求了。

```python
def lcsm(fasta):
    """Finding the longest common nucleotide sequence from multiple sequences in a fasta file
    """
    seqs = list(read_fasta(fasta).values())
    return(lcs(seqs))
```

# 討論

## longest common substring & longest common subsequence

[Longest common substring] 和 [longest common subsequence] 是不同的問題："AB" 是 "ABXC" 和 "ABOC" 的 longest common substring，"ABC" 則否；`[A, B, C]` 是 `[A, B, X, C]` 和 `[A, B, O, C]` 的 longest common subsequence，`[A, B]` 則否。

因為在生物資訊的語境裡，序列（sequence）也用於描述 DNA/RNA/蛋白質，而這些序列資訊往往也用字串（string）紀錄。因此，當初想這題有什麼巧妙解法時，誤入歧途去查了 longest common "subsequence" 的解法。

寫出的程式沒有返回預期結果時，問問 chatgpt 怎麼解 longest common subsequence 還覺得是不是 AI 變笨了，怎麼給你 Rosalind 題目的完整輸出入結果還嘴硬。分清楚這兩個問題後，我才明白小丑是我自己🤡。

（如果想要找到更快的算法，可以點進 Longest common substring 連結看看維基百科的解釋）

[Longest common substring]: https://en.wikipedia.org/wiki/Longest_common_substring
[longest common subsequence]: https://en.wikipedia.org/wiki/Longest_common_subsequence

## 所有字串的 LCS 不見得與一對字串的 LCS 一致

明白小丑是我自己之後，我謄錄了一份利用動態規劃尋找兩字串 longest common substring 的方法。簡言之，這方法會一一比對兩字串每個字符，並且將最長子字串的長度紀錄在一個矩陣中。再比對過所有字符後，便能得出 LCS。

```python
def lcs(s1, s2):
    """Finding the longest common substring from two strings
    """
    dp = [[0] * (len(s2) + 1) for _ in range(len(s1) + 1)]
    max_len, end_pos = 0, 0
    for i in range(1, len(s1) + 1):
        for j in range(1, len(s2) + 1):
            if s1[i-1] == s2[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1
                if max_len < dp[i][j]:
                    max_len = dp[i][j]
                    end_pos = i
                else:
                    pass
            else:
                dp[i][j] = 0
    return s1[end_pos - max_len:end_pos]
```

我原先的以為，一對字串的 LCS 會與所有字串的 LCS 相同，所以我可以像是「找出最大的數字」一樣，找出「最長的共同子字串」，

```python
# 這是錯的!!!!!
def lcsm(fasta):
    """Finding the longest common nucleotide sequence from multiple sequences in a fasta file
    """
    seqs = list(read_fasta(fasta).values())
    m = seqs[0]
    for s in seqs[1:]:
        m = lcs(s, m)
    return m
```

雖然這支程式可順利通過 ROSALIND 的測資，但我在討論區立刻看到一則反例，才乖乖寫成題解那個形式。

> I'm not sure the LCS of all the strings would necessarily be a substring of the pairwise LCS's. For example, in this set of strings:
> AAAAATATATAACGT AAAAACCCCCACGT ACACACCCCCACGT
> The LCS of the first and second string will be AAAAA, the LCS of the second and third string will be CCCCC, but the LCS of all three strings will be ACGT.

