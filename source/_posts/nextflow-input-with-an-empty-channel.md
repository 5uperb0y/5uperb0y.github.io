---
title: Nextflow｜輸入 empty channel 會發生什麼事？
date: 2022-12-27 18:10:49
tags: nextflow
categories: bioinformatics
---

上週碰到一個離奇的 bug，有支 nextflow 腳本可以順利執行，但會無聲無息地略過其中一個 process。該腳本執行後於終端吐出的訊息類似以下形式：

```
$ nextflow run workflow.nf
[3c/9ab742] process > a1 [100%] 1 of 1 ✔
[8e/4eg429] process > a2 [100%] 1 of 1 ✔
[-        ] process > b1
[2f/0c0b71] process > c1 [100%] 1 of 1 ✔
```
<!--more-->
- `a1` 與 `a2` 輸出的 channel 經 `join` 之後輸入 `b1`，而 `c1` 則獨立於其他三個 process。
- nextflow 沒有任何錯誤訊息（即終端看不到紅色的字）
- nextflow 沒有為 `b1` 開啟工作目錄，所以也無從查看 `.command.log/err/out` 等與腳本執行相關的資訊
- 查看 `.nextflow.log` 只能找到啟動 `b1` 的訊息：`[main] DEBUG nextflow.processor.TaskProcessor - Starting process > b1`，但沒有其他執行細節。

# 解決策略

由於 nextflow 幾乎沒有給出任何提示，所以只能自己想辦法在腳本內安排中斷點印出 channel 的內容。當時的策略是先不用 `join`，而是單獨輸入 `a1` 與 `a2` 的輸出，以便用 `stdout` 和 `view()`檢查各個 channel 是否與我們預期相同。

結果問題還真的出在 channel 的合併，原來 `a1` 與 `a2` 的輸出結果沒有共同的 key，所以執行 `join` 操作後會生成一個 empty channel。

# Empty channel

在 Nextflow 裏頭，可使用 `channel.empty()`建立 empty channel，其作用相當於對 process 傳遞終止訊號（參考 [Nextflow Channel Class 的說明](https://javadoc.io/static/io.nextflow/nextflow/0.28.2/nextflow/Channel.html)）。

舉例來說，以下這支 nextflow 腳本會依序印出同一時間輸入的數對。雖然 `a_ch` 含有四項數字，但因為 `b_ch` 其中一項為 empty channel 的終止訊號，所以這個 process 只遞交並執行了三次[^1]。

```groovy
// demoEmptyChannel.nf
process p {
    input:
        val a
        val b
    output:
        stdout
    """
    echo "(${a}, ${b})"
    """
}

workflow {
    a_ch = channel.of("a1", "a2", "a3", "a4")
    b_ch = channel.of("b1", "b2").concat(channel.empty()).concat(channel.of("b4")) // append items with concat() and channel.of()
    p(a_ch, b_ch).view()
}
```
```bash
$ nextflow run demoEmptyChannel.nf
[a8/4b2c55] process > p (3) [100%] 3 of 3 ✔
(a3, b4)
(a2, b2)
(a1, b1)
```

# 使用 `ifEmpty` 為輸入 channel 設定預設值
為了避免 process 被跳過或是沒有執行的狀況，[nextflow 官方文件](https://www.nextflow.io/docs/latest/operator.html#operator-ifempty)建議以 `ifEmpty` 為 channel 設定預設值。如以下案例，由於 `b_ch` 為 empty channel，經 `ifEmpty` operator 賦予預設值，使得 process 能順利運行。
```groovy
//demoIfEmpty.nf
process p {
    input:
        val a
        val b
    output:
        stdout
    """
    echo "($a, $b)"
    """}
workflow {   
    a_ch = channel.of("a1")
    // <defalut value> if ch.isempty() else ch
    b_ch = channel.empty()
    p(a_ch, b_ch.ifEmpty("b_default")).view()
}
```
```bash
$ nextflow run demoIfEmpty.nf
[47/459729] process > p (1) [100%] 1 of 1 ✔
(a1, b_default)
```

# 結論
Nextflow 的 process 會跳過含有 empty channel 的任務，若想避免這種狀況在非預期的情形發生（例如 `join` 失敗），可使用 `ifEmpty` 為 process 的輸入 channel 設定預設值。

# 延伸閱讀
- [NextFlow: How to fail if channel is empty ( .ifEmpty() )](https://stackoverflow.com/questions/70888844/nextflow-how-to-fail-if-channel-is-empty-ifempty)
- [check if nextflow channel is empty](https://stackoverflow.com/questions/64042860/check-if-nextflow-channel-is-empty))
- [Nextflow Workshop Hackaton 2018: 07_processes](https://github.com/nextflow-io/nf-hack18/blob/master/asciidocs/07_processes.adoc)

[^1]: 執行腳本時，nextflow 會先以 `nextflow.processor.TaskProcessor` 啟動 process，透過 `nextflow.executor.LocalHandler` 一一提交輸入 channel 的內容給 process 執行。詳細的流程可參閱 `.nextflow.log`。




