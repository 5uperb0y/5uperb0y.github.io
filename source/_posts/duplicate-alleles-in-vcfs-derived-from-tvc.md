---
title: Duplicate alleles in VCFs derived from tvc
date: 2024-08-07 22:39:18
tags: bioinformatics
categories:
---

最近，我們碰到一個棘手的技術問題。其他同仁反映，我們開發的變異分析工具在執行 GATK 時意外中止，並呈現以下錯誤訊息：

```
The provided VCF file is malformed at approximately line number 1: Duplicate allele added to VariantContext: TA
```

我檢查該步驟使用的 VCF 檔，發現其中一列的 REF 與 ALT 欄位內容重複，導致 GATK 報錯：

```
chr1	100	.	TA	GT,TA
```

標準的 VCF 檔只記錄實際存在的變異，不應出現 REF 和 ALT 欄位相同的狀況。即使該位置沒有變異，通常也會以點號（`.`）標記。因此，我推測問題的源頭不是分析工具本身，而是上游的 variant caller。

<!--more-->

由於這份 VCF 檔源於 Torrent variant caller 5.18 (tvc)，所以我研究了它的說明書。tvc 允許用戶透過 hotspot VCF，自訂分析區域以及必須報告的變異。在醫療檢驗服務中，我們通常會把具有臨床價值的變異位點納入 hotspot VCF，提升 tvc 對這些變異位點的靈敏度。由於出錯的條目恰好是這些臨床位點，我推測異常行為可能跟 hotspot VCF 的內容有關。

經過一系列測試，我發現這項錯誤可以在特定情境重現：

1. 樣本中存在 MNV (multi-nucleotide variant)，例如：
   ```
   chr1	100	.	TA	GT
   ```
2. Hotspot VCF 在相同位置存在兩個必須回報的 SNV (single nucleotide variant)，而且這兩個 SNV 的 REF 和 ALT 一致。（其中，`*` 表示任何非 "T" 鹼基）
   ```
   chr1	100	.	T	G
   chr1	100	.	*	T
   ```

在這種情境，tvc 表現出一些令人意外的行為

- 當偵測到 MNV 時，tvc 不會單獨報告 SNV，而是合併相同位置的變異。
- 在偵測變異時，tvc 只考慮 hotspot VCF 的 POSITION 和 ALT 欄位。

這些因素共同作用，導致 tvc 產生了以下異常結果：

```
chr1	100	.	TA	GT,TA
```

在這個結果中，"GT" 是樣本原有的 MNV，而異常的 "TA" 則分別來自第二個 SNV 的 ALT（"T"），以及參考基因體上的 "A"。

成功重現這個錯誤之後，我曾懷疑是否在使用 tvc 時誤植了 hotspot VCF 內容。然而，經過實際測試，我發現在最新版的 tvc 中，即使使用完整的 hotspot 也會面臨同樣問題。這表示可能有其他未考慮的因素與這個錯誤有關。

總而言之，鑒於網路上關於 tvc 的資源匱乏，希望這些紀錄能幫助陷入類似問題的同行，感謝大家。