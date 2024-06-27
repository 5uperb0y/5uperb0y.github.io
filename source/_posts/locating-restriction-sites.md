---
title: ROSALIND｜Locating Restriction Sites (REVP)
date: 2024-06-18 23:23:46
tags: rosalind 
categories:
---

給定一條以 FASTA 儲存的 DNA 序列，求長度介於 4 到 12 之間的反向迴文（reverse palindrome）的起始位置與長度。

> A DNA string is a reverse palindrome if it is equal to its reverse complement. For instance, GCATGC is a reverse palindrome because its reverse complement is GCATGC. See Figure 2.
>
> Given: A DNA string of length at most 1 kbp in FASTA format.
> 
> Return: The position and length of every reverse palindrome in the string having length between 4 and 12. You may return these pairs in any order.

(https://rosalind.info/problems/revp/)

<!--more-->

# 背景知識
限制酶（restriction enzyme）是一種存在於細菌中的酵素，它能夠識別並切割特定的 DNA 序列。這些酵素是細菌免疫系統的一環，主要的功能是抵禦外源 DNA 的威脅，例如噬菌體（bacterial phage）或是質體（plasmid）等。

限制酶的發現可追溯到 1950 年代 [^1]。當時，Luria 和 Human 發現，有些噬菌體感染新宿主的能力會受到前宿主的影響，而且這種影響是可逆的，不依賴基因突變。這一現象後來被證明是因為宿主細菌的限制酶切割了噬菌體的核酸，從而「限制」了它們的感染能力。

細菌和外源 DNA 都可能成為限制酶的作用對象，而細菌和噬菌體也各自演化出能對抗限制酶的機制。細菌能透過甲基化酶，修飾限制酶識別位置的核苷酸，讓自身的 DNA 免受傷害。限制酶與甲基化酶組成了細菌的限制與修飾系統，兩者的平衡影響著細菌基因體的穩定性[^2]。另一方面，在部分細菌和病毒基因體中，限制酶識別序列的出現頻率偏低，這可能是一種適應性的演化，能減少 DNA 被限制酶切割的機率[^3][^4]。

[^1]: Loenen et al. (2014). Highlights of the DNA cutters: a short history of the restriction enzymes. Nucleic acids research, 42(1), 3-19.
[^2]: Shaw et al. (2023). Restriction-modification systems have shaped the evolution and distribution of plasmids across bacteria. Nucleic acids research, 51(13), 6806-6818.
[^3]: Rusinov et al. (2018). Avoidance of recognition sites of restriction-modification systems is a widespread but not universal anti-restriction strategy of prokaryotic viruses. BMC genomics, 19, 1-11.
[^4]: Rusinov et al. (2015). Lifespan of restriction-modification systems critically affects avoidance of their recognition sites in host genomes. BMC genomics, 16, 1-15.

限制酶切割後可能留下黏性端（Sticky end），這些黏性端能讓不同來源 DNA 片段互相配對，如果使用 DNA 接合酶連接比次，就能達成重組基因的目的。

不同類型的限制酶識別和切割 DNA 的模式與反應條件不同。其中，II 型限制酶的識別和切割位置相近，這種切割的精準性與可預測性，使其在生物科技上有廣泛的應用。

```
# Sticky End
5'-...A   TAT...-3'
3'-...TAT   A...-5'

# Blunt End
5'-...AT   AT...-3'
3'-...TA   TA...-5'
```

其次，由於個體間基因體的微小差異，若這些差異恰好出現在限制酶辨識位置，那麼使用相同的限制酶切割 DNA 時，會產生長度不同的片段。

這些 DNA 片段能透過電泳技術分離，提供基因體比較的資訊，這使得限制酶也成為物種鑑定、遺傳變異分析、鑑識科學等的工具 [^5]。

[^5]: Di Felice et al. (2019). Restriction enzymes and their use in molecular biology: An overview. Journal of biosciences, 44(2), 38.

在基因體學，反向迴文（reverse palindom）是指一段 DNA 序列與其反向互補序列一致。許多 II 型限制酶會識別基因體上這種迴文序列。

儘管不是所有限制酶都是如此，不過迴文序列的對稱性讓它能夠與同一股 DNA 序列的其它部分配對，形成可能有利於酵素作用的特殊結構，例如髮夾（hairpin）結構等。

![DNA palindromes](https://upload.wikimedia.org/wikipedia/commons/thumb/7/75/DNA_palindrome.svg/1280px-DNA_palindrome.svg.png)
（圖片源自：https://en.wikipedia.org/wiki/Palindromic_sequence）

其次，迴文序列表示，酵素能透過相同序列來辨識相同位置。這有助於限制酶單體聚集在鄰近位置而形成更有效率的雙聯體，同時切斷兩股序列，造成難以修補的傷害。

```
5'-...ATAT...-3'
3'-...TATA...-5'
```

反之，如果限制酶辨識的不是迴文序列，那麼兩股 DNA 的識別位置可能相距很遠，這不僅降低切割的效率。即使順利切割，也可能形成較長的黏性端，使其更容易被連接酶修復。

```
5'-GAGA......-3'
3'-......AGAG-5'
```

若想了解限制酶偏好迴文序列的解釋，可參考以下幾個討論串：
- [Why do Type II Restriction Endonucleases cleave at palindromic sequences?](https://biology.stackexchange.com/questions/9913/why-do-type-ii-restriction-endonucleases-cleave-at-palindromic-sequences)
- [Why are restriction sites palindromic in nature?](https://biology.stackexchange.com/questions/81626/why-are-restriction-sites-palindromic-in-nature)
- [Why do restriction enzymes tend to have an even number of bases in their recognition site?](https://biology.stackexchange.com/questions/1167/why-do-restriction-enzymes-tend-to-have-an-even-number-of-bases-in-their-recogni)


綜上所述，雖然不是所有限制酶都辨識迴文序列 [^6]，但迴文序列對於限制酶而言，似乎有一些結構上的優勢。

[^6]: https://en.wikipedia.org/wiki/List_of_restriction_enzyme_cutting_sites

# 解題觀念

嚴格來說，這道題其實不是找出限制酶的位置，畢竟限制酶是透過實驗決定的。題目的要求是尋找長度介於 4 - 12 之間的反向迴文序列，這是 II 型限制酶識別位置的常見長度。我們可以先設計出能找出所有反向迴文序列的方式，再移除超出範圍的序列即可滿足題目要求。

窮舉法是解決這個問題最直觀的方式，也就是逐一生成 DNA 所有子字串，判斷它們是否為反向迴文。生成子字串的過程需要 $O(n^2)$ 的時間複雜度，加上判斷反向迴文所需的 $O(n)$，共計 $O(n^3)$。因此，窮舉法雖然直觀而且全面，可是效率不高。

然而，依據迴文的對稱特性，我們其實沒有必要檢查所有的子字串。迴文字串兩側的字符必然完全對應且相等。因此，如果一個字串不是迴文，那麼從相同中心往兩側延長的字串也必然不是迴文。

| 座標         | 子字串     | 反向迴文 |
| ------------ | ---------- | -------- |
| `s[i-1:j-1]` | `...11...` | 是       |
| `s[i:j]`     | `..2110..` | 否       |
| `s[i-1:j+1]` | `.321103.` | 否       |

（為了方便，這一節暫時使用一般的迴文定義來解說演算法）

基於這項特性，中心擴展法（middle expansion method）應運而生。這個方法逐一檢查每個位置，將其視為潛在的迴文中心。接著，從中心往兩側擴展，直到兩側的字符不再對稱相等。

擴展過程中得到的子字串，就是以該位置為中心的迴文。由於這過程需要遍歷所有字符兩次，所以它的時間複雜度為$O(n^2)$。

1975 年，Manacher 演算法的發現進一步提升了搜尋迴文的效率。這個算法仍以中心擴展法為核心，但利用迴文的對稱特性，避免巢狀迴文的重複計算，使得時間複雜度降至 $O(n)$。

具體來說，在使用中心擴展法時，如果檢查點位於已知迴文 $S$ 內，就能透過 $S$ 的中心找到與檢查點的對稱點。

```
   |-迴文字串-|
...ooooo.ooooo...
     |  |  └ 檢查點
     |  └ 迴文中心
     └ 對稱點
```

根據迴文的特性，中心兩側字符必然彼此對應且相等。因此，如果左側存在迴文字串，那麼右側對應片段也會有完全相同的迴文字串。

假設檢查點與對稱點分別是迴文 $s$ 和 $s'$ 的中心，那麼根據它們的長度，可能存在兩種狀況。

第一，對稱點迴文 $s'$ 完全被 $S$ 覆蓋。根據兩側對稱特性，$s'$ 與 $s$ 的組成和長度完全一致，所以可以透過對稱點來確認檢查點的迴文資訊。

```
...xoooooooo.oooooooox...
    |<-s'>|   |<-s->| 
   |<-------S------->|
```

第二，對稱點迴文只有部分在 $S$ 內。這表示不能完全依賴對稱點來確認檢查點的迴文資訊，因為 $s'$ 有部分擴展到 $S$ 的邊界外。

此時，仍要回歸中心擴展法，但這次是從 $S$ 的邊界開始檢查，確認 $s$ 能否繼續延伸並形成新的迴文。如果 $s$ 能繼續延伸，形成更長的迴文，那麼它就會成為新的 $S$，能用來確認其後檢查點的迴文資訊。

```
...ooooooo.oooooox...
   |<-s'>|  <-s->
    |<----S---->|
```

這種作法確保了尋找迴文時，只在必要的情形使用中心擴展，其餘情形都可以透過儲存的資訊取得結果。查詢迴文資訊為常數時間，而且中心擴展始於 $S$ 的邊界，而非從頭開始。

即使沒有對應的迴文資訊，這方法仍持續延伸並記錄各位置的迴文資訊。因此，它才能避免重複計算，達到 $O(n)$ 的時間複雜度。

| 方法                                | 改進                     | 時間複雜度 | 空間複雜度 |
| ----------------------------------- | ------------------------ | ---------- | ---------- |
| 窮舉法（brute force）               |                          | $O(n^3)$   | $O(1)$     |
| 中心擴展法（middle expansion）      | 跳過不可能形成迴文的字串 | $O(n^2)$   | $O(1)$     |
| Manacher 法（Manacher's algorithm） | 只在必要時使用中心擴展法 | $O(n)$     | $O(n)$     |

# python 實作

如前所述，根據基因體學的定義，迴文序列與其反向互補序列一致。為了讓程式碼易讀，我新增了 `complement` function 來生成 DNA 的互補序列。這意味著如果字串 `dna` 是迴文，那麼它應該滿足 `dna == complement(dna)[::-1]`。

```python
def complement(dna):
    """Complementing a DNA sequence."""
    return dna.translate(str.maketrans("ATCG", "TAGC"))
```

考慮到反向迴文的定義，只有偶數長度的 DNA 序列才能形成完全一致的反向互補序列。因此這裡我們只尋找偶數長度的迴文。

## 窮舉法

窮舉法的步驟是列出 DNA 序列的所有子序列，同時確認它們是否滿足反向迴文。根據題目要求，DNA 序列的位置由 1 開始計數，所以在記錄迴文起始位置時，需要額外加 1 以滿足要求。

```python
def revp_bf(dna):
    """Find a list of all palindromes' positions and their length, in the DNA
    sequence using brute force.

    Args:
        dna (str): A DNA string composed of nucleotides ("A", "T", "C", "G").

    Returns:
        list: A list of tuple, recording the position and length of every
              reverse palindrome, like [(<position>, <length>)]
    """
    return [
        (i + 1, j - i)
        for j in range(len(dna) + 1)
        for i in range(j)
        if dna[i:j] == complement(dna[i:j])[::-1]
    ]
```

## 中心擴展法
應用中心擴展法時，我們要逐一檢查序列各個位置，將其視為潛在的迴文中心點 `mid`，再從中心往兩側檢查序列的對稱互補性。

在這個題目的條件下，DNA 迴文的長度必為偶數，所以 `mid` 不是特定字符的位置，而是字符間隙的位置。

其次，DNA 首尾兩個位置無法作為中心點，所以 `mid` 的數值介於 1 到 `len(dna) - 1`之間。

```
DNA   A T C G A G G A G
Pos  0 1 2 3 4 5 6 7 8 9 
      mid->
```

`span` 則是從 `mid` 向兩側擴展的距離（或說潛在迴文的半徑）。因為這個距離不會超出 DNA 序列的邊界，所以 `span` 它的最大值取決於 `mid` 到 DNA 序列兩端的最短距離。

```
DNA   A T C G A G G A G
Pos  0 1 2 3 4 5 6 7 8 9 
Span |<--------->|
          mid
```

在每個 `mid` 對應的 `span` 值域內，我們逐步增加 `span` 的長度，再依次檢查擴展範圍內的 DNA 序列是否滿足迴文（即左側與右側的核苷酸對稱互補）。

如果兩側核苷酸彼此互補，則記錄它的起始位置和長度，並且持續擴展 `span`；兩者不匹配，則中斷當前 `mid` 的擴展，轉向下一個中心點，直到所有位置都檢查過為止。

```python
def revp_me(dna):
    """Find a list of all palindromes' positions and their length, in the DNA
    sequence using middle expansion approach.

    Args:
        dna (str): A DNA string composed of nucleotides ("A", "T", "C", "G").

    Returns:
        list: A list of tuple, recording the position and length of every
              reverse palindrome, like [(<position>, <length>)]
    """
    palindromes = []
    for mid in range(1, len(dna)):
        for span in range(1, min(mid, len(dna) - mid) + 1):
            left_nt = dna[mid - span]
            right_nt = dna[mid - 1 + span]
            if left_nt == complement(right_nt):
                palindromes.append((mid - span + 1, span * 2))
            else:
                break
    return palindromes
```

## Manacher 演算法

Manacher 演算法最初是設計來尋找最長的迴文字串。不過，如果知道每個中心點的最長迴文字串，其實也能重建出以同樣位置為中心，只是長度較短的迴文字串。因此，這個方法只需一些調整就能用來解這這題。

另外，迴文中心在偶數長度字串位於兩字符間，而在奇數長度字串則位於特定字符上。為了解決這項分歧，原始的 Manacher 演算法會在字符間安插特殊符號（例如 `#`，`abc` -> `#a#b#c#`），將任何字串統一為奇數長度的形式。

對於這題，由於已知所有迴文長度都為偶數，所以可以省略插入特殊符號的步驟，直接處理原始的 DNA 序列。

為了讓程式碼的結構更清晰，我把中心擴展法的功能獨立到 `extend` function。這個 function 負責從指定的中心點開始，向兩側擴展來檢查序列的對稱互補性。`extend` function 允許用戶從已知的迴文邊界開始擴展，減少了重複檢查的狀況。

```python
def extend(dna, mid, start_span=1):
    """Extend the palindrome from the middle point to the maximum span.

    Args:
        dna (str): A DNA string composed of nucleotides ("A", "T", "C", "G").
        mid (int): Index of the middle point from which to check symmetry.
        start_span (int): Initial distance from the middle to start checking.

    Returns:
        int: Maximum span where the sequence remains a palindrome.
    """
    max_span = start_span
    for span in range(start_span, min(mid, len(dna) - mid) + 1):
        left_nt = dna[mid - span]
        right_nt = dna[mid - 1 + span]
        if left_nt == complement(right_nt):
            max_span = span
        else:
            break
    return max_span
```
Manacher 演算法的核心是利用已知的迴文資訊來避免重複檢查，它需要額外的空間來記錄以下三項資訊：

- `max_spans` 儲存字串各個中心點的最大迴文半徑。它們的數值預設為 -1，表示該位置未經檢查。隨著檢查進度，相應位置的數值會逐步更新。
- `right` 是最靠近 DNA 序列右端的已知迴文的右邊界位置。我們可以透過它來判斷檢查點是否位於已知的迴文內，從而利用已知的迴文資訊。
- `center` 是 `right` 對應迴文的中心位置。它能幫助我們找到檢查點的對稱點位置。

Manacher 演算法的計算流程與中心擴展法相似，仍須依序檢查 DNA 序列各個位置。不過在這基礎上，還要額外判斷當前位置是否位於某個已知的迴文內。

假設當前的檢查點 `mid` 小於 `right`，表示我們有機會利用已知的回文資訊。此時，要先找到檢查點在較大迴文中的對稱點 `mirror`。

如果對稱點的最大迴文半徑小於 `right - mid`，表示以對稱點為中心的迴文完全被較大的迴文覆蓋。因此，檢查點的迴文半徑就等於對稱點的迴文半徑。

如果對稱點的最大迴文半徑大於 `right - mid`，表示較大的迴文沒有完全覆蓋對稱點的迴文，所以檢查點的迴文半徑最多只能延伸到 `right` 的位置。

反之，如果 `mid` 在 `right` 之外，表示沒有任何可參考的資訊，所以檢查點的迴文半徑設定為 0，意味著得從頭開始進行中心擴展。

在判斷了 `mid` 與 `right` 的關係後，就能從已知的迴文半徑開始執行中心擴展，避免重複檢查已經擴展過的片段。擴展結束後，需要判斷擴展後的迴文半徑是否超出 `right`。若是，則要更新 `right` 與 `center` 的值。

最後，從 `max_span` 可以得到所有位置的最大迴文半徑，進而依據這些資訊還原其他較短迴文的位置和長度。

```python
def revp_manacher(dna):
    """Find a list of all palindromes' positions and their length, in the DNA
    sequence using the Manacher's algorithm.
    
    Args:
        dna (str): A DNA string composed of nucleotides ("A", "T", "C", "G").

    Returns:
        list: A list of tuple, recording the position and length of every
              reverse palindrome, like [(<position>, <length>)]
    """
    max_spans = [-1] * len(dna)
    right = center = 0
    for mid in range(1, len(dna)):
        if mid < right:
            mirror = 2 * center - mid
            max_spans[mid] = min(max_spans[mirror], right - mid)
        else:
            max_spans[mid] = 0
        max_spans[mid] = extend(dna, mid, max_spans[mid])
        if max_spans[mid] + mid > right:
            right = max_spans[mid] + mid
            center = mid
        else:
            pass
    return [
        (idx - span + 1, span * 2)
        for idx, max_span in enumerate(max_spans)
        for span in range(1, max_span + 1)
    ]
```

# 討論：為什麼 Manacher 演算法好像沒快很多？

為了檢查寫出來的 function 是否實現了 Manacher 演算法，所以我也使用題目提供的範例數據，比較了不同窮舉法、中心擴展法和 Manacher 法的計算效率。

窮舉法顯然是三者最慢的，不過為什麼 Manacher 法（$O(n)$）好像沒有比中心擴展法（$O(n^2)$）快？

```
Using DNA sequnce TCAATGCATGCGGGTCTATATGCAT * 100
Timing revp_bf:
5.213197014000002
Timing revp_me:
0.0027632010000502305
Timing revp_manacher:
0.004045999999959804
```

雖然我有添加一些程式碼來重建所有長度的迴文序列，但這些應該不至於拖累太多吧？後來回想起來，時間複雜度是基於最差情況的計算，題目提供的示範數據可能無法區分不同算法的效率差異。

II 型限制酶的長度多半在 4 到 12 之間。由於示範數據是依據這項觀察所設計，所以其中的迴文序列彼此嵌套或重疊的狀況也比較少。因此，即使使用 Manacher 演算法，多數片段仍要依賴中心擴展法。這導致 Manacher 演算法和中心擴展法在這個案例的效率相似。

為了驗證這個假說，我用 ATAT 組成的序列進行測試，因為它在所有位置都能形成迴文。而結果也如預期，對於多重嵌套的迴文結構，Manacher 演算法展現了它應有的效能。

```
Using DNA sequnce ATAT * 100
Timing revp_bf:
0.06110990399997718
Timing revp_me:
0.018740201000014167
Timing revp_manacher:
0.004075799999952778
```

測試的程式碼可參考我的 [Github](https://github.com/5uperb0y/rosalind/blob/main/code/revp/performance.py)。