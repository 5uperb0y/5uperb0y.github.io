---
title: Rosalind，生物資訊的 LeetCode
date: 2024-06-25 18:45:09
tags: rosalind
categories:
---

Rosalind 是一個以生物資訊為主題的程式解題平台，它與 LeetCode 等解題網站類似，能提供測試資料並且自動核對用戶上傳的答案。不過，Rosalind 的特色在於它收錄了生物資訊領域的經典問題，例如序列比對、譜系分析與基因重組等。

因此，在解決這些問題的同時，不僅能熟悉程式語言特性和了解演算法內涵，還能學習如何將生物學問題轉換為資訊科學問題，培養建模思考的方式。

<!--more-->

目前，Rosalind 平台有近三百道題目，這些題目依據其性質、解法與來源分為以下五類。

- [Python Village]：涵蓋解題所需的 Python 基礎知識，內容包含資料型態、條件判斷、迴圈語法與檔案讀寫等核心概念。
- [Bioinformatics Stronghold]：透過解題來學習解決生物資訊常用的演算法與其應用情境。適合已具備 Python 基礎的用戶學習。
- [Bioinformatics Armory]：介紹生物資訊常用的第三方軟體，學習使用這些工具解決實務難題。
- [Bioinformatics Textbook Track]：收錄 [Compeau & Pevzner (2014) Bioinformatics Algorithms](https://www.bioinformaticsalgorithms.org/) 的練習題，可以配合教科書的進度練習。
- [Algorithmic Heights]：收錄 [Dasgupta et al. (2006) Algorithm](http://algorithmics.lsi.upc.edu/docs/Dasgupta-Papadimitriou-Vazirani.pdf) 的練習題，可以配合教科書進度練習。

[Python Village]: https://rosalind.info/problems/list-view/?location=python-village
[Bioinformatics Stronghold]: https://rosalind.info/problems/list-view/
[Bioinformatics Armory]: https://rosalind.info/problems/list-view/?location=bioinformatics-armory
[Bioinformatics Textbook Track]: https://rosalind.info/problems/list-view/?location=bioinformatics-textbook-track
[Algorithmic Heights]: https://rosalind.info/problems/list-view/?location=algorithmic-heights


Rosalind 收錄的題目依據其所需的知識與所考量的前提，可以分為基礎、進階以及綜合三個層次。基礎項目是隨後幾個項目的鋪陳，用戶需要先完成基礎題目才能解鎖進階題目，而通過必要的進階題目後，才能挑戰綜合題目。

透過清單列的 "Problems" 選項，用戶可以切換題目的陳列模式。[List](https://rosalind.info/problems/list-view/) 模式依照題目釋出時間排序，提供題目回答率與正確率等資訊。[Topics](https://rosalind.info/problems/topics/) 模式則依照題目性質分類，方便用戶鑽研特定住提。[Tree](https://rosalind.info/problems/tree-view/) 模式則呈現了題目彼此依賴的關係，有助於用戶沿特定途徑學習，了解特定知識點的脈絡與變化。

我個人喜歡透過 Tree 模式學習，透過深度優先策略來解題。這方法的優點在於，進階題目建立在基礎題目的概念之上，所以解題時不至於毫無頭緒。此外，當觀念一再出現，也能夠達到複習的目的，強化學習效果。

![](https://raw.githubusercontent.com/5uperb0y/blog-media/main/rosalind-a-platform-for-learning-bioinformatics-through-problem-solving/rosalind-a-platform-for-learning-bioinformatics-through-problem-solving_problems.png)

在陳列頁面點擊題目名稱即可瀏覽詳細的題目說明，每道題目都包含一段背景知識描述、名詞釋義、題目陳述以及輸出入範例。

![](https://raw.githubusercontent.com/5uperb0y/blog-media/main/rosalind-a-platform-for-learning-bioinformatics-through-problem-solving/rosalind-a-platform-for-learning-bioinformatics-through-problem-solving_description.png)

由於 Rosalind 的伺服器沒有程式碼編譯器，所以作答時需要點擊 "Download Dataset" 手動下載測試資料，接著在自己的電腦完成計算，再把答案上傳或貼到解答區域。

每道題目的作答時間為五分鐘，相對於 LeetCode 的時間限制是相當寬鬆。目前我寫過的題目中，只有 [Reversal Distance](https://5uperb0y.com/reversal-distance/) 這道題需要考量效能問題，其餘題目即使使用暴力法都能在時限內解決。

點選 "Submit" 提交答案之後，Rosalind 就會檢查答案是否正確。由於系統僅檢查用戶上傳的答案，而不是上傳的 function，所以實際上不限制用戶以特定程式語言作答。

然而，也因為這項特性，Rosalind 平台不像 LeetCode 能用許多邊緣案例來測試 function 的穩健性。因此，除了自行設計測試資料，也建議多下載幾筆平台提供的資料進行測試，確保解法的正確性。


一旦解題成功，系統會解鎖這道題的討論區，其中包含解法分享與問題討論，這些內容都是由 Rosalind 的熱心用戶共同維護。

![](https://raw.githubusercontent.com/5uperb0y/blog-media/main/rosalind-a-platform-for-learning-bioinformatics-through-problem-solving/rosalind-a-platform-for-learning-bioinformatics-through-problem-solving_answering.png)

綜上所述，Rosalind 平台的宗旨是讓用戶透過解題來學習生物資訊學。雖然在學界和產業界，比起從頭構思演算法和重新撰寫程式，開發人員其實花更多時間在選擇與調試既有的工具。

可是，我覺得這樣的練習仍有其價值，因為它能幫助我們掌握必要的生物學知識、熟悉程式語言特性、學習演算法核心概念、培養問題建模的思考方式。這些都是解決實際生物資訊問題的重要素養。

以下是我的解題記錄與學習筆記，希望能為讀者（還有未來的自己）提供一些參考和啟發。

| ID   | Title                                                | Information                                  | Solution                    | Note                                                                               |
| ---- | ---------------------------------------------------- | -------------------------------------------- | --------------------------- | ---------------------------------------------------------------------------------- |
| DNA  | Counting DNA Nucleotides                             | [Info](https://rosalind.info/problems/dna/)  | [Code](./code/dna/dna.py)   | [Note](https://5uperb0y.com/counting-dna-nucleotides/)                             |
| RNA  | Transcribing DNA into RNA                            | [Info](https://rosalind.info/problems/rna/)  | [Code](./code/rna/rna.py)   | [Note](https://5uperb0y.com/transcribing-dna-into-rna/)                            |
| HAMM | Counting Point Mutations                             | [Info](https://rosalind.info/problems/hamm/) | [Code](./code/hamm/hamm.py) | [Note](https://5uperb0y.com/counting-point-mutations/)                             |
| REVC | Complementing a Strand of DNA                        | [Info](https://rosalind.info/problems/revc/) | [Code](./code/revc/revc.py) | [Note](https://5uperb0y.com/complementing-a-strand-of-dna/)                        |
| PERM | Enumerating Gene Orders                              | [Info](https://rosalind.info/problems/perm/) | [Code](./code/perm/perm.py) | [Note](https://5uperb0y.com/enumerating-gene-orders/)                              |
| SIGN | Enumerating Oriented Gene Orderings                  | [Info](https://rosalind.info/problems/sign/) | [Code](./code/sign/sign.py) | [Note](https://5uperb0y.com/enumerating-oriented-gene-orderings/)                  |
| LEXF | Enumerating k-mers Lexicographically                 | [Info](https://rosalind.info/problems/lexf/) | [Code](./code/lexf/lexf.py) | [Note](https://5uperb0y.com/enumerating-k-mers-lexicographically/)                 |
| LEXV | Ordering Strings of Varying Length Lexicographically | [Info](https://rosalind.info/problems/lexv/) | [Code](./code/lexv/lexv.py) | [Note](https://5uperb0y.com/ordering-strings-of-varying-length-lexicographically/) |
| SUBS | Finding a Motif in DNA                               | [Info](https://rosalind.info/problems/subs/) | [Code](./code/subs/subs.py) | [Note](https://5uperb0y.com/finding-a-motif-in-dna/)                               |
| LCSM | Finding a Shared Motif                               | [Info](https://rosalind.info/problems/lcsm/) | [Code](./code/lcsm/lcsm.py) | [Note](https://5uperb0y.com/finding-a-shared-motif/)                               |
| LGIS | Longest Increasing Subsequence                       | [Info](https://rosalind.info/problems/lgis/) | [Code](./code/lgis/lgis.py) | [Note](https://5uperb0y.com/longest-increasing-subsequence/)                       |
| REAR | Reversal Distance                                    | [Info](https://rosalind.info/problems/rear/) | [Code](./code/rear/rear.py) | [Note](https://5uperb0y.com/reversal-distance/)                                    |
| SORT | Sorting by Reversals                                 | [Info](https://rosalind.info/problems/sort/) | [Code](./code/sort/sort.py) | [Note](https://5uperb0y.com/sorting-by-reversals/)                                 |
| PROT | Translating RNA into Protein                         | [Info](https://rosalind.info/problems/prot/) | [Code](./code/prot/prot.py) | [Note](https://5uperb0y.com/translating-rna-into-protein/)                         |
| ORF  | Open Reading Frames                                  | [Info](https://rosalind.info/problems/orf/)  | [Code](./code/orf/orf.py)   | [Note](https://5uperb0y.com/open-reading-frames/)                                  |
| REVP | Locating Restriction Sites                           | [Info](https://rosalind.info/problems/revp/) | [Code](./code/revp/revp.py) | [Note](https://5uperb0y.com/locating-restriction-sites/)                           |
| SPLC | RNA Splicing                                         | [Info](https://rosalind.info/problems/splc/) | [Code](./code/splc/splc.py) | [Note](https://5uperb0y.com/rna-splicing/)                                         |
| SSEQ | Finding a Spliced Motif                              | [Info](https://rosalind.info/problems/sseq/) | [Code](./code/sseq/sseq.py) | [Note](https://5uperb0y.com/finding-a-spliced-motif/)                              |
| LCSQ | Finding a Shared Spliced Motif                       | [Info](https://rosalind.info/problems/scsq/) | [Code](./code/lcsq/lcsq.py) | [Note](https://5uperb0y.com/finding-a-shared-spliced-motif/)                       |
| EDIT | Edit Distance                                        | [Info](https://rosalind.info/problems/edit/) | [Code](./code/edit/edit.py) | [Note](https://5uperb0y.com/edit-distance/)                                        |