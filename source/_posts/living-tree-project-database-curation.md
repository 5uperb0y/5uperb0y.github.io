---
title: All-Species Living Tree Project (LTP) 物種資訊的訂正和維護
date: 2022-11-22 00:03:33
tags: metagenomics
categories:
---

All-Species Living Tree Project (LTP)[^1] 旨在維護所有已知古菌和細菌 type strain 的 16S rRNA 基因序列，並基於這些序列，建立古菌和細菌的譜系樹。

LTP 主要由四個單位參與
- LPSN 提供分類與命名資訊 (https://www.bacterio.net/)
- ARB 支援譜系樹建立與分類 (http://www.arb-home.de/)
- Ribocon 則負責資料庫的管理 (https://www.ribocon.com/)
- SILVA 則提供序列資料庫、計算資源與網頁介面 (https://www.arb-silva.de/)

（自 2020 年起，LTP 的遷出 SILVA，獨立為一個網頁：https://imedea.uib-csic.es/mmg/ltp/）

LTP 的資料常用於 16S rRNA 基因增幅序列的物種註解，而基於其收錄序列所構建的譜系樹，也能作為 Phylogenetic placement 的基礎[^2]。

另外，由於 LTP與 RDP training set 都只收錄了 type strain 的序列資料，所以相較於透過序列預測建立的資料庫，LTP 的序列資料較為可靠，能避免在物種註釋時，要在分類錯誤之外面臨資料庫錯誤的問題。

儘管 LTP 已是人工維護且品質好的資料庫，但目前版本 ( LTP_01_2022) 的部分物種資訊仍有誤騰的狀況。在官方修正這些問題之前，資料庫的使用者得自己訂正這些錯誤。

<!--more-->

# 分類階層間少了分隔符

若使用 ";" 來分割這樣的字串，會得到較短的列表。
> AF251436 Ferrimicrobium acidiphilum Bacteria;ActinomycetotaAcidimicrobiia;Acidimicrobiales;Acidimicrobiaceae;Ferrimicrobium

# 分類名稱出現額外符號

文字前後出現空格、引號和句號等，可能會因字串不同而高估該分類階層的分類群數量，也可能在資料處理造成意外麻煩。
## 額外的雙引號 
> AY102612 Caedibacter taeniospiralis "Bacteria;Pseudomonadota;Alphaproteobacteria;Holosporales;""Caedimonadaceae"";Caedibacter 
## 額外的句號
> LC552068 Conexivisphaera calida Archaea;Crenarchaeota;Conexivisphaeria;Conexivisphaerales;Conexivisphaeraceae;Conexivisphaera.
## 額外的空格（位於 Hyphomonas adhaerens 與 Bacteria 之間） 
> KF863150 Hyphomonas adhaerens Bacteria;Pseudomonadota;Alphaproteobacteria;Hyphomonadales;Hyphomonadaceae;Hyphomonas Bacteria;Pseudomonadota;Alphaproteobacteria;Caulobacterales;Hyphomonadaceae;Hyphomonas

# 分類名稱間有額外的分隔符

會導致該紀錄多一個分類階層 
> HQ223108 Thermoleophilum minutum Bacteria;Actinomycetota;;Thermoleophilia;Thermoleophilales;Thermoleophilaceae;Thermoleophilum

# 重複出現的分類名稱

例如同樣的分類寫兩遍，或是整個條目寫了兩遍，會導致該紀錄增加多個分類階層
## 門名重複出現 
> AJ288899 Bacteriovorax stolpii Bacteria;Bdellovibrionota;Bdellovibrionota;Bacteriovoracia;Bacteriovoracales;Bacteriovoracaceae;Bacteriovorax
## 整串分類名稱重複出現 
> "AF082795 Hyphomonas rosenbergii Bacteria;Pseudomonadota;Alphaproteobacteria;Hyphomonadales;Hyphomonadaceae;Hyphomonas Bacteria;Pseudomonadota;Alphaproteobacteria;Caulobacterales;Hyphomonadaceae;Hyphomonas

# 誤植分類名稱

在此例中，屬名 Amnimonas 被誤謄為 Aminomonas（這是另一個屬）例如因為形似而寫成其他分類名、打錯字，會導致 convergent 現象（即相同子分類群卻有不同母分類群）
## 誤植 (Aminomonas → Amnimonas) 
> MF682432 Amnimonas aquatica Bacteria;Pseudomonadota;Gammaproteobacteria;Moraxellales;Moraxellaceae;Aminomonas
## 錯字 (Jatrophihabitantaless→Jatrophihabitantales) 
> LC323105 Jatrophihabitans telluris Bacteria;Actinomycetota;Actinobacteria;Jatrophihabitantaless;Jatrophihabitantaceae;Jatrophihabitans

# 異名

即條目間使用該分類群的不同名稱，會導致 convergent 現象（即相同子分類群卻有不同母分類群）
## 異名
> (Bdellovibrionia [synonym] → Oligoflexia [correct name])
## 俗名
> (Bdellovibrionota [not validly published] → Bdellovibrionota [correct name]) HM038000 Vampirovibrio chlorellavorus Bacteria;Bdellovibrionota;Oligoflexia;Bdellovibrionales;Bdellovibrionaceae;Vampirovibrio AJ292759 Bdellovibrio bacteriovorus Bacteria;Bdellovibrionota;Bdellovibrionia;Bdellovibrionales;Bdellovibrionaceae;Bdellovibrio

# 欠缺完整分類階層

比其它條目少了一個分類階層 
>  \# Lack information at Genus level (Ulvibacterium)
>  KF303137 Aestuariibaculum scopimerae Bacteria;Bacteroidota;Flavobacteriia;Flavobacteriales;Flavobacteriaceae
 

# 檢查方式以及我們的義務
這些問題的模式紛亂，其實不容易找到有效的偵測方法，以下枚舉幾個我曾用過的策略，
- 以分隔符切割字串，檢查輸出字串列表當中，長度異常的條目
- 下載 LPSN 物種資訊，檢查未出現在 LPSN 的條目
- 檢查出現分號、括號、空格以外的符號的條目
- 利用這些方法檢查完畢後，就順道把自己的發現寄給他們，當作對資料庫維護團隊的感謝吧。

最後，分享一個我原本以為是因為以下原因而產生的 typo .....

> VIFM01000001 Myxococcus llanfairpwllgwyngyllgogerychwyrndrobwllllantysiliogogogochensis Bacteria;Myxococcota;Myxococcia;Myxococcales;Myxococcaceae;Myxococcus

![這難道不是因為某個老梗產生的物種名稱嗎......](https://github.com/5uperb0y/blog-media/blob/main/living-tree-project-database-curation_ptt.png?raw=true)

拿去 Goolge 才知道，還真的有名字這麼長的生物[^3]，長到維基百科側欄都破圖了。

![名字太長了，網頁都破圖囉。](https://github.com/5uperb0y/blog-media/blob/main/living-tree-project-database-curation_wiki.png?raw=true)

Myxococcus llanfairpwllgwyngyllgogerychwyrndrobwllllantysiliogogogochensis 命名自其被分離的地點，位於威爾斯安格爾西島的 Llanfair­pwll­gwyn­gyll­go­gery­chwyrn­drobwll­llan­tysilio­gogo­goch 小鎮，分離地點附近還有一間四星級飯店😮。

![分離地點附近有間四星級飯店](https://github.com/5uperb0y/blog-media/blob/main/living-tree-project-database-curation_map.png?raw=true)

註
[^1]: All-Species Living Tree Project (LTP)  相關文獻，Yarza & Munoz. (2014). The all-species living tree project. *In Methods in Microbiology* (Vol. 41, pp. 45-59). Academic Press. 以及  Yarza et al. (2008). The All-Species Living Tree project: a 16S rRNA-based phylogenetic tree of all sequenced type strains. *Systematic and applied microbiology*, 31(4), 241-250.

[^2]: Phylogenetic placement 是指，依據枝節間的親緣關係，將輸入序列置放在既存譜系樹適當位置。由於有納入參考譜系樹，這方法比 de novo  building 更節省計算資源；也因為採用「插入」取代「捨棄」，也比 read mapping/recruitment 保留更多序列。參考下文以理解 phylogenetic placement 的優勢： Janssen et al. (2018). Phylogenetic placement of exact amplicon sequences improves associations with clinical information. *Msystems*, 3(3), e00021-18.

[^3]: Chambers et al. (2020). Comparative Genomics and Pan-Genomics of the Myxococcaceae, including a Description of Five Novel Species: Myxococcus eversor sp. nov., Myxococcus llanfairpwllgwyngyllgogerychwyrndrobwllllantysiliogogogochensis sp. nov., Myxococcus vastator sp. nov., Pyxidicoccus caerfyrddinensis sp. nov., and Pyxidicoccus trucidator sp. nov. *Genome biology and evolution*, 12(12), 2289-2302.

