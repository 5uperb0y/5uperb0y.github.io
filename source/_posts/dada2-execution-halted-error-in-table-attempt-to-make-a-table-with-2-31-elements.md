---
title: 'DADA2 execution halted (Error in table: attempt to make a table with >= 2^31 elements)'
date: 2024-05-04 13:13:25
tags: metagenomics
categories:
---

本文純粹是技術問題。我之前以 Qiime2 外掛的 DADA2 處理已切除轉接子的 16S rRNA 基因序列 (V4 region) 時，因為以下錯誤而中斷了執行四天的程式。

```r
Plugin error from dada2: An error was encountered while running DADA2 in R (return code 1), please inspect stdout and stderr to learn more.
```
<!--more-->
我檢查錯誤報告後發現以下敘述：
```r
Error in table(pairdf$forward, pairdf$reverse) :
attempt to make a table with >= 2^31 elements
Calls: mergePairs -> lapply -> FUN -> table
Execution halted
Running external command line application(s). This may print messages to stdout and/or stderr.
The command(s) being run are below. These commands cannot be manually re-run as they will depend on temporary files that no longer exist.
```

於是根據關鍵字查詢只找到[一起相似案例](https://github.com/benjjneb/dada2/issues/641)，但該討論串沒有提供可行的解決辦法。簡言之，提問者的資料中可能有人類序列汙染，導致獨特序列的數量超出 DADA2 演算法的上限，所以程式才異常中斷。

然而，抽檢資料卻沒發現人類序列汙染，而且使用小量資料可以順利執行，讓我難以判斷問題的癥結點。因此，我蒐集 Qiime2 論壇上常提到的出錯狀況，一一測試來確認能否解決問題。

首先，我[調整了線程](https://forum.qiime2.org/t/dada2-error-return-code-1/7140/3)以免記憶體用罄，接著[檢查格式](https://forum.qiime2.org/t/dada2-return-code-1/2972)確保資料完整無誤，也重新安裝以排除未知問題，但這些方法都無濟於事，所以我只好 Qiime2 論壇[求助](https://forum.qiime2.org/t/dada2-execution-halted-error-in-table-attempt-to-make-a-table-with-2-31-elements/11845)。

開發團隊指出，這狀況可能是資料中有未清除的技術片段、定序品質太差或是存在汙染序列，使得 DADA2 在其中一步驟時，因數據量過載而中斷（這也可以解釋少量資料可以順利執行的原因）。

在我移除潛在的文庫汙染，切除末端的低品質序列後，DADA2 便能順利執行了。綜上所述，此錯誤源於數量過多的低品質/汙染序列，在移除汙染和品質控管後，即可解決問題。

而關於除錯，我也學到了幾個經驗：

- 仔細閱讀錯誤訊息，鎖定問題癥結點後才能有效排除障礙。胡亂猜測出錯原因無濟於事，反而浪費時間和精力。
- 要以錯誤訊息為關鍵字，上網找尋類似問題的解決方式；若找不到直接相關的問題，則重新檢視演算法，說不定可以找到問題來源。
- 把蒐集到的方法整理成表，一一測試結果。
- 若查詢和測試皆無果，則準備詳盡資料到專門論壇求助。
- 解決問題後把過程記錄下來，幫助其他人克服困難🥰

