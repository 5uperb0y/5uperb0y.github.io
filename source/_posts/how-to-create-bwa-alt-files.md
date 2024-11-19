---
title: 怎麼準備 bwa 所需的 .alt 檔？
date: 2024-10-14 18:47:53
tags: bioinformatics
categories:
---

因應 GRCh38 納入 alternate contigs (ALT contigs)，BWA 早在 [2014 的更新](https://sourceforge.net/p/bio-bwa/mailman/message/32845712/) 便開始支援 alt-aware mapping（詳見 [bwa/README-alt.md](https://github.com/lh3/bwa/blob/master/README-alt.md) 以及 [GATK 的教學](https://sites.google.com/a/broadinstitute.org/legacy-gatk-documentation/tutorials/8017-How-to-Map-reads-to-a-reference-with-alternate-contigs-like-GRCh38)）。

使用 alt-aware mapping 模式需要準備 *.alt* 檔，它記錄了 ALT contigs 在參考基因體 (primary contigs) 的位置，可以用 ALT contigs 拿 BWA 比對參考基因體生成。因此，*.alt* 實際上是 SAM 格式，而不是使用 bwa index 建立的索引格式。

```bash
bwa mem ref.fa alt.fa > ref.fa.alt
```
<!--more-->

ALT contigs 是同段基因在不同族群的變體，納入它們可以提升參考基因體的族群代表性。人類參考基因體的建置最初僅納用少數個體的定序資料。人類基因體有一些多樣性高的區域（例如 MHC），它們的序列往往迥異於參考基因體，所以比對軟體難以定位來自這些區域的定序產物，降低變異分析的靈敏度。為了減輕參考基因體相關的偏誤，GRCh38 版本基於千人基因體 (1000 Genomes) 等計畫的研究成果，納入了將近 109 Mb 長的 ALT contigs。

然而，ALT contigs 仍保留與參考基因體相似的片段。因此，一條序列可能同時吻合 ALT contigs 或原參考基因體，導致軟體無法區別序列來源，低估了比對的可信度（即 MAPQ, Mapping Qualities）。詳情可參考 illumina 簡介：[ALT-Aware mapping](https://jp.support.illumina.com/content/dam/illumina-support/help/Illumina_DRAGEN_Bio_IT_Platform_v3_7_1000000141465/Content/SW/Informatics/Dragen/GPipelineAltMap_fDG.htm)。

為此，需要額外資訊來記錄 ALT contigs 和參考基因體的關係，讓比對軟體能妥善處理 GRCh38 這類含有 ALT contigs 的參考基因體。在 bwa，額外資訊儲存在 *.alt*。

*.alt* 的格式為 SAM，記錄了每條 ALT contigs 在原本參考基因體的位置。提供 *.alt* 之後，bwa 會註記能比對到 ALT contigs 的結果。假如序列與 ALT contigs 契合，bwa 便會以 ALT contigs 為代表來計算比對分數，最後透過 *.alt* 的座標資訊找到輸入序列在參考基因體的正確位置。

建立 *.alt* 的具體做法如下：假設有參考基因體 *ref.fa* 和 ALT contigs *alt.fa*，首先使用 bwa 創建 *.alt* 檔，再合併 *ref.fa* 和 *alt.fa* 為含有 ALT contigs 的 *ref_alt.fa*。最後再用 bwa index 生成其他序列比對所需的索引檔。

```bash
bwa mem ref.fa alt.fa > ref.fa.alt
cat ref.fa > ref_alt.fa
cat alt.fa >> ref_alt.fa
bwa index ref_alt.fa
```

剩下步驟可參考 [How to Map reads to a reference with alternate contigs like GRCh38](https://sites.google.com/a/broadinstitute.org/legacy-gatk-documentation/tutorials/8017-How-to-Map-reads-to-a-reference-with-alternate-contigs-like-GRCh38) ，簡言之，使用 alt-aware mapping 只是第一步，還需要用 [bwa-postalt.js 腳本](https://github.com/lh3/bwa/tree/master/bwakit)處理那些有配對到 ALT contigs 的序列（這支腳本擺在 bwa/bwakit 裡面，所以要去 github 下載）。

---
*註 1： .alt 是採用 alt-mapping 的必要檔案，但本文提到的參考資料都沒講清楚它的生成方式，才會有那麼多貼文求助。*

- [How to create .alt index file?](https://github.com/lh3/bwa/issues/231)
- [BWA fasta alt file for GRCH38p14](https://www.reddit.com/r/bioinformatics/comments/17snuzk/bwa_fasta_alt_file_for_grch38p14/)
- [BWA .alt index file](https://www.biostars.org/p/432382/)

*註 2：當初這方法提出時，bwa 的作者其實不鼓勵使用含有 ALT contig 的參考基因體，不過現今 WGS/WES 成熟、註解資料庫相容、而且有許多 multi-genome mapper，風向也漸漸改變了。*