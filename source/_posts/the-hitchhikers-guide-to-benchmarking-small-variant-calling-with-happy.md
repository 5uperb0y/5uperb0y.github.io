---
title: 如何使用 hap.py 評估小片段變異分析的效度？
date: 2024-11-05 21:30:51
tags: bioinformatics
categories: 
---

**hap.py** 是比較 VCF 的工具，常配合標準資料集來評估變異分析的效度，它整合了資料處理、集合運算與指標統計等功能，方便用戶解決效度評估常見的問題。

<!--more-->

# 急用的時候讀這裡

準備 `truth.vcf`、`query.vcf`、`reference.fa`、`reference.fa.fai`、`confident.bed` 放入工作目錄，下載並進入 hap.py 分析環境，執行腳本後檢查結果。

```bash
$ docker pull quay.io/biocontainers/hap.py:0.3.15--py27hcb73b3d_0
$ docker run -it -v </path/to/workdir>:</path/to/workdir> -w </path/to/workdir> quay.io/biocontainers/hap.py:0.3.15--py27hcb73b3d_0 bash
$ hap.py truth.vcf query.vcf -f confident.bed -o output_prefix -r reference.fa
$ awk '{print $1,$14}' output_prefix.summary

Type  METRIC.F1_Score 

INDEL 0.5                ← 😫 結果很不好

SNP   1                  ← 😁 結果非常好
```

等待程式運算期間可以泡杯咖啡繼續讀，了解 hap.py 的用途與結果詮釋。

# 想比較 VCFs 的話，為什麼不直接取交集和差集？

因為相同變異可能有不同的表示方式，所以變異間非一一對應關係，若僅取交集和差集會錯估變異分析的效度，例如：

## multi-allelic vs. bi-allelic variants
```
chr1	20000	.	T	G,C	1000	PASS	.	GT	2/1
```
or
```
chr1	20000	.	T	C	1000	PASS	.	GT	0/1
chr1	20000	.	T	G	1000	PASS	.	GT	0/1
```

## multiple SNVs vs. single MNV

```
chr1	20000	.	TC	CA	1000	PASS	.	GT	0/1
```
or
```
chr1	20000	.	T	C	1000	PASS	.	GT	0/1
chr1	20001	.	C	A	1000	PASS	.	GT	0/1
```

## left-aligned Indels vs. right-aligned Indels
```
chr1	20002	.	TCCT	TCT	1000	PASS	.	GT	0/1
```
or
```
chr1	20001	.	TC	T	1000	PASS	.	GT	0/1
```

## multiple short Indels vs. single long Indel
```
# ref:   TC-C-TG
# input: TCTCCTG
chr1	20001	.	C	CT	1000	PASS	.	GT	0/1
chr1	20002	.	C	CC	1000	PASS	.	GT	0/1
```
or

```
# ref:   T--CCTG
# input: TCTCCTG
chr1	20000	.	T	TCT	1000	PASS	.	GT	0/1
```

# 具體來說，hap.py 是什麼？

hap.py ≈ vcf preprocessor + comparison engine + metric summarizer

- vcf prprocessor：統一 VCF 格式，標準化變異表示法，方便比較引擎運作
- comparison engine：考量變異表示法差異，正確統計混淆矩陣的各項元素（true positive, false positive, false negative）
- metric sumarizer：依照混淆矩陣計算精準度、召回率和 F1 score 等效度指標。

# 使用 hap.py 需要準備什麼？

- `truth.vcf` 記錄了你認為可靠的變異，用以和 query.vcf 比較。經過 Genome in a Bottle (GIAB) Consortium 認證的變異資料集可在 [NCBI FTP 資料庫](https://ftp-trace.ncbi.nlm.nih.gov/ReferenceSamples/giab/release/)下載。
- `query.vcf` 是你打算評估的 VCF 檔，標頭至少要含有 VCF 格式版本 (`##fileformat`)、contigs（`##contig=<...>`）、genotype（`##FORMAT=<ID=GT,...>`）及欄位名稱（`#CHROM ...`）。
- `confident.bed` 記錄了你要求 hap.py 檢查的基因體區域，這些區域以外的變異會標記為 UNK，不納入效度計算。"confident" 這名稱沿襲 GIAB 的慣例，位於 confident region 的變異經多項技術驗證，任何額外或遺漏的變異皆可視為誤報。
- `reference.fa` & `reference.fa.fai` 是變異分析使用的參考基因體及其索引檔。`hap.py` 需要這些資訊來依據基因體區域計算效度，還有計算 het-hom 與 Ti/Tv ratio 等品管指標。因此，要使用生成 truth.vcf 和 query.vcf 的參考基因體版本。
- `hap.py` & its environment 可以參考[官方教學](https://github.com/Illumina/hap.py?tab=readme-ov-file#installation)安裝，或是使用 biocontainer 上的 [image](https://biocontainers.pro/tools/hap.py)。dockerhub 也有一份 [image](https://hub.docker.com/r/pkrusche/hap.py?ref=https://githubhelp.com) ，可是已數年沒有更新，所以內容落後 hap.py 的教學文件。

執行分析前可以檢查一下 truth.vcf 和 query.vcf 的染色體表示法是否一致（`chrX` or `X`）、標頭內容和 INFO 欄位是否匹配，這些都是 hap.py 報錯的常見原因。

# 如何解讀結果？

hap.py 會在你使用 `-o` 指定的目錄輸出比對結果。精確度、召回率與 F1 score 等校度統計量紀錄在 `*.summary.csv`，其它次要的統計量則保留在 `*.extended.csv`。

`*.summary.csv` 的欄位可分為數據分層和效度指標兩類，我習慣先看 Type 和 METRIC.F1_Score，只要 SNV 和 INDEL 的 F1 score 接近 1，那麼大家都會幸福。

| Type  | METRIC.F1_Score |
| ----- | --------------- |
| INDEL | 0.99            |
| SNP   | 0.99            |

## 數據分層 (Stratification)

用戶能要求 hap.py 獨立計算特定類別的變異之效度指標。因此，如果想更仔細檢驗分析結果，可以先查看數據分層的方式（詳情參考 [hap.py 的官方簡介](https://github.com/Illumina/hap.py/blob/master/doc/happy.md#introduction)）。

- **Type**：目前，hap.py 支援 SNV 和 INDEL 的效度評估，暫時沒有其它變異類型。
- **Subtype**：SNV 可細分為 transition （腺嘌呤與鳥糞嘌呤，胞嘧啶與胸腺嘧啶互換）和 transversion（嘌呤與嘧啶互換），INDEL 也有不同長度片段的插入與刪除。
- **Filter**：hap.py 能依據 VCF 檔 FILTER 欄位，獨立列出數值為 PASS 的變異之效度指標。比較合格與全數變異的效度指標，便能評估變異品管的成效。
- **Subset**：如果使用 `--stratification <BED>` 要求 hap.py 獨立計算特定區域的效度指標，就會多出 *Subset* 欄位註記分析結果所屬區域的名稱，這些名稱對應到 BED 檔的 ID 欄（第四欄）。

## 混淆矩陣項目 (Confusion matrix elements)

比對之前，hap.py 會先統一 `truth.vcf` 和 `query.vcf` 的格式（例如 #CHROM 是否包含 `chr`），再標準化 `query.vcf` 以利比較引擎運作。標準化的方式包含將 MNV 拆分為獨立的 SNV、拆分 multi-allelic 變異、依據基因型合併相同位置的變異等，詳見[官方文件](https://github.com/Illumina/hap.py/blob/master/doc/normalisation.md)）。

hap.py 預設的比較引擎為 *xmap*，它會考量變異表示法的差異，從而避免低估分析的正確性。假設 `truth.vcf` 包含一個 MNV，例如：

```
chr1	20000	.	TC	CA	1000	PASS	.	GT	0/1
```

標準化之後， `query.vcf` 會將 MNV 拆分為兩個獨立的 SNV。

```
chr1	20000	.	T	C	1000	PASS	.	GT	0/1
chr1	20001	.	C	A	1000	PASS	.	GT	0/1
```

這兩組 VCF 檔實際上是同一種變異的不同表示方式。如果只取交集會得到空集合，但 *xmap* 能辨識這種狀況並正確標示出來。

- **TRUTH.TP** 是出現於 `truth.vcf`，而且能在標準化 `query.vcf` 找到的變異數量。
- **QUERY.TP** 是出現於標準化 `query.vcf`，而且能在`truth.vcf` 找到的變異數量。

TRUTH.TP 和 QUERY.TP 的數值有可能不一樣，這是因為在不同表示方式下，變異之間非一一對應。在上述案例中，`truth.vcf` 的一個 MNV 對應 `query.vcf` 的兩個 SNV，所以 TRUTH.TP 為 1，QUERY.TP 為 2。除了 MNV 和 SNV，序列比對以及基因型也跟變異的表示法有關。多種要素混合在一起就變得沒那麼直觀，所以我也還沒掌握所有案例。不過，如果對結果有疑慮都可以查看 `output_prefix.vcf.gz` 的 *TRUTH* 和 *QUERY* 欄位。它們註記了變異真偽的判定結果，可以依此推論 hap.py 標準化和比較變異的方式。

- **TRUTH.FN**：`truth.vcf` 獨有的變異數量
- **QUERY.FP**：`query.vcf` 獨有的變異數量
- **QUERY.UNK** 是 `query.vcf` 位在 *confident.bed* 以外的變異數量。
- **TRUTH.TOTAL**：*truch.vcf* 的變異總數，相當於 *TRUTH.TP* + *TRUTH.FN*。
- **QUERY.TOTAL**：`query.vcf` 的變異總數，相當於 *QUERY.TP* + *QUERY.FP* + *QUERY.UNK*。

hap.py 另外將 FP 分為兩類，但其實很少用到。
- **FP.gt**：Genotype errors，相同位置 (`POS`)、相同變異 (`REF`/`ALT`)、不同基因型 (`GT`)，例如 query GT = 1/1，truth GT = 0/1。
- **FP.al**：Allele errors，相同位置、不同變異、不同基因型。

## 效度指標 (Accuracy metrics)

- **METRIC.Recall**：召回率，真實變異被檢出的比率，計算方式為 TRUTH.TP / (TRUTH.TP + TRUTH.FN)。評估變異分析的效度時，通常會用多筆 `query.vcf` 對照單一 `truth.vcf`。召回率採計 TRUTH.TP 和 TRUTH.FN 以免數值因為 `query.vcf` 的表示法差異而浮動[^1]。
- **METRIC.Precision**：精準度，檢出變異為真實變異的比率，計算方式為 QUERY.TP / (QUERY.TP + QUERY.FP)。
- **METRIC.F1_Score**：精準度與召回率的調和平均，計算方式為 2 * METRIC.Recall* METRIC.Precision / (METRIC.Recall + METRIC.Precision)。
- **METRIC.Frac_NA**：QUERY.UNK / QUERY.TOTAL。

召回率高但精準度低的分析過度寬鬆，納入過多偽變異徒增診斷成本（全部個案都判陽性的檢驗有百分之百的召回率，卻一點用都沒有）；精準度高但召回率低的分析過度保守，遺漏過多真實變異限縮診斷效果（不會報錯卻驗不出真實變異的檢驗也沒什麼用）。F1 score 同時衡量了召回率和精準度，提供更符合直覺的效度估計。F1 score 使用調和平均的理由可以參考 [why is the f measure a harmonic mean and not an arithmetic mean of the precision](https://stackoverflow.com/questions/26355942/why-is-the-f-measure-a-harmonic-mean-and-not-an-arithmetic-mean-of-the-precision)。簡言之，調和平均對於差異懸殊數值的權重較低，所以唯有同時確保召回率和精準度，才能取得較高的 F1 score。

[^1]: 假設兩組 `query.vcf` 用不同表示方式記錄相同的變異，它們照理要有相同的召回率，所以才會用不受 `query.vcf` 表示法影響的 `truth.vcf` 來計算。

## 品管指標 (Quality metrics)

Ti/Tv ratio 和 het/hom ratio 是評估變異分析品質的指標，和 `truth.vcf`/`query.vcf` 的比較無關。這兩種指標在生物基因體內有一定範圍，而且其數值遠高於隨機值。通常，指標數值越高，變異分析的品質也越好。Ti/Tv ratio 是 transition 和 transversion 兩種 SNV 數量的比值。假設鹼基發生 transition 和 transversion 的機率一致，那麼 Ti/Tv ratio 應該要等於 0.5。因為 transition 有兩種變化途徑，但 transversions 有四種變化途徑，2 比 4 得 0.5。

依據轉譯密碼表，造成胺基酸變化的突變往往屬於 transversion，這種突變若發生在重要的蛋白質可能導致生物死亡。存在選擇壓力的情況下，生物體的 transversion 數量往往少於 transition，使得基因體的 Ti/Tv ratio 遠高於 0.5。人類全基因體定序結果的 Ti/Tv ratio 接近 2.1，全外顯子定序的 Ti/Tv ratio 接近 3（因為 exon 比 intron 暴露到更多選擇壓力下）。假設定序或分析的錯誤皆為隨機發生，若全基因體變異分析的 Ti/Tv ratio 比率遠低於 2.1，便可合理懷疑其中混入了太多隨機發生的錯誤，導致數值偏離預期。
 
het/hom ratio (heterozygous/homozygous variant ratio) 也是基於相似的原理。依據族群遺傳學的哈溫定律，het/hom ratio 會等於 2，如果變異分析的結果偏離預期太多，也可能反映了某種錯誤存在。這兩種指標都只能用來評估一整批變異分析的結果，細節可參考 Guo et al. (2014). Three-stage quality control strategies for DNA re-sequencing data. Briefings in bioinformatics, 15(6), 879-889.

- **TRUE.TOTAL.TiTv_ratio**：truth.vcf 的 Ti/Tv ratio。
- **QUERY.TOTAL.TiTv_ratio**：query.vcf 的 Ti/Tv ratio。
- **TRUTH.TOTAL.het_hom_ratio**：truth.vcf 的 het/hom ratio。
- **QUERY.TOTAL.het_hom_ratio**：query.vcf 的 het/hom ratio。

在有正確答案的情況下，直接比較 `truth.vcf` 和 `query.vcf` 更為直觀且具體。不過，Ti/Tv ratio 和 het/hom ratio 能作為額外的查核資訊。如果它們的數值和 F1 score 呈不同趨勢，能提醒我們細查資料，確認是否發生非預期的錯誤。

# 如果還有興趣，可以研讀以下論文與簡介

- [Karl Sebby. (2021). Demystifying benchmarking: A guide to germline variant calling metrics.](https://medium.com/truwl/demystifying-benchmarking-a-guide-to-germline-variant-calling-metrics-9d6eee4f8e80)：簡單明瞭的部落格文章，把變異分析效度評估解釋得很清楚。
- [Krusche et al. (2019). Best practices for benchmarking germline small-variant calls in human genomes. Nature biotechnology, 37(5), 555-560.](https://pmc.ncbi.nlm.nih.gov/articles/PMC6699627/)：GA4GH Benchmarking Team 發表的效度分析指引，可以了解變異表示法相關的偏誤，以及各項指標的設計原則。
- [Supplementary of Krusche et al. (2019)](https://pmc.ncbi.nlm.nih.gov/articles/instance/6699627/bin/NIHMS1533783-supplement-1.pdf)：前一則論文的補充資料，更詳盡地解釋了指標的公式。
- [Illumina/hap.py/doc/happy.md](https://github.com/Illumina/hap.py/blob/master/doc/happy.md) 以及 [Illumina/hap.py: Haplotype VCF comparison tools.](https://github.com/Illumina/hap.py)：hap.py 官方手冊，能了解程式具體做了那些處理。
- [Yih-Chii Hwang. (2020). Benchmarking state-of-the-art secondary variant calling pipelines.](https://medium.com/dnanexus/benchmarking-state-of-the-art-secondary-variant-calling-pipelines-5472ca6bace7)：說明如何進行效度分析的部落格文章。