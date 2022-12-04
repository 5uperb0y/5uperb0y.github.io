---
title: Nextflow｜背景執行 workflow 的方法
date: 2022-11-30 00:47:46
tags: nextflow
categories: bioinformatics
---
一般而言，若想要背景執行 linux 指令，可在指令末端添加 `&`，或是透過 `ctrl + z` 配合 `bg %n` 將執行中的指令挪到背景執行。然而，nextflow 腳本卻不適用這種做法（version >= 21.10.6），指令挪到背景後會陷入停止狀態。

```bash
$ nextflow run workflow.nf &
[1] 533
Launching `workflow.nf` [sick_waddington] - revision: 123b1ec198
[2]+  Stopped                 nextflow run workflow.nf
```

一旦陷入停止狀態，會變得異常難清，要用 `kill %n && fg` 才能一次清掉（參考[論壇的討論](https://gitter.im/nextflow-io/nextflow/archives/2020/09/16)）。

<!--more-->
## 情境
即使透過 `.sh` 來執行 nextflow 腳本也會遭遇相同的狀況。
```
#!/bin/bash
# run_workflow.sh
nextflow run workflow.nf
```

```bash
$ ./run_workflow.sh &
N E X T F L O W  ~  version 21.10.6
Launching `workflow.nf` [trusting_hamilton] - revision: 936bafe285
[1]-  Stopped                 nohup nextflow run paramInput.nf
[2]+  Stopped                 ./run_workflow.sh
```

## 解決辦法
此時，若要背景執行 nextflow 腳本，可在指令或是腳本內添加 nextflow 內建的 `-bg` 選項。
```bash
$ nextflow run workflow.nf -bg
 N E X T F L O W  ~  version 21.10.6
Launching `workflow.nf` [furious_kowalevski] - revision: 936bafe285
[8b/21d003] Submitted process > sayHi
Hi
```

這選項觸發的行為類似 `nohup`，能確保用戶退出 terminal 後仍能持續執行 nextflow 腳本（可參考官方[文件](https://www.nextflow.io/docs/latest/cli.html#execution-as-a-background-job)及[部落格](https://www.nextflow.io/blog/2021/5-more-tips-for-nextflow-user-on-hpc.html)）。

除此之外，添加 `-bg` 也會輸出 `.nextflow.pid` 檔，紀錄此指令的 pid，以便用戶追蹤 nextflow 腳本的執行狀況。
```bash
ps -p <pid>
```

若隨時將指令切到前台，並且用 `jobs` 查看執行狀態，也可以在指令末端補 `&`。只是這樣退出 terminal 後，nextflow 腳本也跟著結束了。

```bash
nextflow run workflow.nf -bg > log.txt &
```

## 結論
至於為什麼 nextflow 有這特性，我仍沒有頭緒。我試著比較添加 `-bg` 前後，`.nextflow.log` 和 `.command.run` 等檔案的內容，卻沒有發現相關差異。

另外，雖然官方說明 `-bg` 的行為類似 `nohup`，但卻無法用 `nohup` 達到同樣效果。可能還需要對 nextflow 和 linux 有更深的理解才能解釋吧。

