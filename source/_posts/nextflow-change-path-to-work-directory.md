---
title: Nextflow｜調整工作目錄的路徑
date: 2023-01-30 22:36:46
tags: nextflow
categories: [bioinformatics]
---
Nextflow 會為分析流程的每一步建立工作目錄，再以工作目錄為中心接收資料、儲存暫存檔和匯出結果。工作目錄預設在 nextflow 腳本執行路徑下的 `work/`，可透過添加執行選項、調整參數設定、設置環境變項等三種方式自訂工作目錄的路徑。
<!--more-->

# 添加執行選項
執行 Nextflow 腳本時，加上 `-w` 便能指定工作目錄的存放路徑與名稱，是優先度最高的方式。

```bash
$ nextflow run workflow.nf -w "/path/to/workdir"
```

# 調整參數設定
其次，也能在[設置檔內定義工作目錄](https://www.nextflow.io/docs/latest/config.html)，再使用 `-c` 導入設置檔。（除此之外，使用 `-params-file` 導入參數或是直接將`workDir` 定義在 nextflow 腳本內都是無效的手法。）

```json
// config.json
workDir = "/path/to/workdir"
```
```bash
$ nextflow run workflow.nf -c config.json
```

要留意的是，雖然在設置檔內常使用 `params.var` 的方式來定義變項，但 `workDir` 不等同 `params.workDir`，因此以下兩種寫法都無法調整工作目錄的存放路徑。
```json
params.workDir = "/path/to/workdir"
params {
    workDir = "/path/to/workdir"
}
```

# 設置環境變項
第三種方法則是在 nextflow 的執行環境重設環境變數，是優先度最低的方式。（Nextflow 相關的環境變項可參照 [Workflow introspection](https://www.nextflow.io/docs/latest/metadata.html)）
```bash
$ export NXF_WORK="/path/to/workdir/"
$ nextflow run workflow.nf
```

# 結論
工作目錄的內容有助於追蹤程式錯誤，為了管理方便，可透過執行指令、內製變項和環境變項來調整工作目錄的路徑。

除了工作目錄外，或許也有人想調整 `.nextflow` 等暫存目錄的位置，但根據[開發者所述](https://github.com/nextflow-io/nextflow/issues/1747)，目前應該是無法更動。
> The `.nextflow` has to be in the launching directory to properly maintain the history of the executions.
