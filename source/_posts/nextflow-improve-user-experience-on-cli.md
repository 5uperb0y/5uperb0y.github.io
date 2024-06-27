---
title: Nextflow｜如何改善用戶的命令列使用體驗
date: 2024-01-08 22:10:02
tags: nextflow 
categories:
---
Nextflow 的命令列介面對開發者而言已相當全面，但對一般用戶而言，仍有可以改善的空間。本文介紹一些 Nextflow 的內建功能，可以因應不同的專案結構，改善用戶的命令列使用體驗。

<!-- more -->
# 如何執行 Nextflow 腳本？

```bash
nextflow run /path/to/workflow.nf\
		 -c  /path/to/params.config\
		 -params-file /path/to/params.json\
		 [--something <value>]
```
Nextflow 的命令包含幾個關鍵選項，

- `run`：啟動腳本的選項，用於執行特定的 Nextflow 腳本
- `-c`：pipeline 的設置檔案，可調整運算環境、計算資源和日誌檔等 pipeline 相關參數，也能批次導入腳本中以 `params`定義的參數
- `-params-file`：參數批次檔，經由 `-params-file` 導入的參數優先於 `-c` 導入的參數。
- `--something`：由命令列輸入的參數，可滿足特定的運算需求，具最高優先度。

# 可能的改進空間

Nextflow 的命令列設計對開發者而言相當全面，但從一般用戶的角度來說，仍有一些使用上的挑戰。這些挑戰中，最明顯的是腳本與設置檔的選擇問題。對於只包含單一腳本與配置檔案的專案而言，清楚的使用說明與文件標示足以確保腳本正確使用。然而，一旦涉及多個腳本與配置檔案，容易造成使用者困惑，尤其是在必須針對不同計算環境或分析場域選擇適當設置檔的情境。

其次，雖然 `-params-file` 提供了參數輸入的靈活性，允許用戶批次輸入樣本並且自訂 pipeline 的參數，但這同時也將 pipeline 的核心參數和樣本相關的參數混合在一起，有可能會造成一定程度的混淆。

再者，自定義參數檔需要以 JSON 格式輸入，可能對一些用戶不夠友善。


# 解決方案

## 預設設置檔
為了簡化設置檔的指定過程，可善用 Nextflow [讀取參數的機制](https://www.nextflow.io/docs/latest/config.html#configuration-file)，將預設參數寫在腳本內部，或是存放在腳本同層目錄下的 `nextflow.config` [^1]。 如此一來，若用戶未使用 `-c` 指定，Nextflow 會自動讀取這兩個參數來源以執行腳本。
```
workflows
|- main.nf
|- nextflow.config
```
[^1]: Nextflow 還不支援在腳本內指定設置檔路徑，所以暫時只有這兩套預設方式。

## 管理多元設置
跨平台與情境的設置檔可以透過 Nextflow 的 [Configuration Profile](https://www.nextflow.io/docs/latest/config.html#config-profiles) 來管理，從而避免因少量參數異動而建立多個設置檔。例如：

```config
profiles {
    docker {
        process.executor = 'docker'
    }
    k8s {
        process.executor = 'k8s'
    }
}
```
使用時，可以在命令列指定相應的 profile 切換執行環境。
```bash
nextflow run workflow.nf -profile "k8s"
```

## 採用表格輸入
為了改善輸入檔案與 pipeline 參數的管理，可以考慮使用表格形式紀錄輸入檔案的資訊，將 pipeline 相關參數另外以命令列傳入，達到區分樣本相關資訊與 pipeline 核心參數的效果。


讀入表格資料的方式可參考 [Process per CSV record](https://nextflow-io.github.io/patterns/process-per-csv-record/)。假設有批定序資料要分析，其檔案路徑記錄在 `input.csv`
```csv
"uuid","fq1","fq2"
"smp1","/path/to/smp1_r1.fq","/path/to/smp1_r2.fq"
"smp2","/path/to/smp2_r1.fq","/path/to/smp2_r2.fq"
```
則讀取方式可寫作：
```groovy
input_ch = Channel.fromPath(params.input) |
	| splitCsv(header:true) \
	| map { row -> tuple(row.uuid, file(row.fq1), file(row.fq2)) }
```
用戶便能透過 CSV 格式管理批次輸入的資料，並透過命令列指定其餘參數。
```bash
nextflow run workflow.nf --input input.csv --output /path/to/output
```

## 使用入口腳本
在包含多個 pipeline 的專案中，可以在專案根目錄新增 `main.nf` 腳本。在這個入口腳本中，可以編寫一些幫助函數，協助用戶了解所有可用的 pipeline，或是讓用戶透過別名來快速呼叫特定的 pipeline，實踐各式各樣的客製功能。

```groovy
if (params.help) {
	// print help messages
	// select pipeline to run by user-input key
}
```
這樣做的好處是，用戶可以用統一的名字呼叫 pipeline，不用記憶各支腳本的路徑。而且，用戶可以使用 `--help` 等參數來了解專案中各 pipeline 的用途與使用方式。

```bash
nextflow run workflow --help
```

# 結論

綜合以上的改進方案後，用戶可以在不深入研究專案目錄結構的情況下，了解該專案能提供的分析及其使用方式，免除指定設置檔的負擔，並且以慣用的表格來管理輸入檔案。
```bash
nextflow run project --workflow <name> --input input.csv [--something <value>]
``` 

若需要額外的封裝和彈性，也能在 Nextflow 框架之外，開發一個能包裝其 API 的軟體，進一步簡化用戶的操作。
```bash
wapper <name> --input input.csv [--something <value>]
```

