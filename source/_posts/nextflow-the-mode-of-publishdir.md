---
title: Nextflow｜利用 `publishDir` 匯出分析檔案
date: 2023-01-14 08:50:46
tags: nextflow
categories:
---
Nextflow 透過 channel 媒介 process 間的檔案傳遞，讓輸出入檔案以軟連結集中到 process 的工作目錄。不過若想從工作目錄取出分析的最終結果，有賴 `publishDir` directive 的協助。

<!--more-->

`publishDir` 能匯出 process 輸出檔案到指定目錄，其語法與注意事項如下：
```groovy
process demoPub {
    publishDir "/path/to/output"
    output:
        path "example.txt"
    """
    touch "example.txt"
    """
}

workflow {
    demoPub()
}
```

- 所有且僅有定義在 output channel 的檔案會匯出到指定目錄
- `publishDir` 應使用絕對路徑 
- 預設會在指定目錄建立輸出檔案的軟連結
- 

```bash
project
├── work
|   └── workdir
|       └── example.txt
└── publishDir
    └── example.txt -> /project/work/workdir    
```

# 修改 `publishDir` 匯出檔案的方式

若想修改預設模式可以寫成
```groovy
publishDir "path/to/output", mode: "copy"
```
```bash
project
├── work
|   └── workdir
|       └── example.txt
└── publishDir
    └── example.txt
```
除了 "copy"，nextflow 亦有其它匯出檔案的方式，它們分別對應了 linux 的 symbolic link, copy, 與 hard link。詳情可參考軟連結、硬連結與複製的差異，此處僅列出與使用檔案較為相關的特性。
| mode | symlink | copy | link |
| ---- | ------- | ---- | ---- |
|linux指令|symbolic link| copy|hard link|
|適用對象|檔案與目錄|檔案與目錄|檔案|
|原檔更新|隨之更新|不隨之更新|隨之更新|
|容量大小|連結大小|與原檔一致|與原檔一致|
|刪除原檔|連結遺失|不受影響|不受影響|


# 篩選輸出的檔案
```groovy
publishDir "path/to/output", pattern: "*.txt"
publishDir "path/to/backup", pattern: "*.md"
```
使用 `pattern` 篩選，可以達到分流輸出檔案的目的。

# 更動輸出名稱
```groovy
publishDir "path/to/output", saveAs: "new_name"
```
使用 `saveAs` 重新命名，可用於重新命名、依照檔名調整輸出目錄等。

# 結論
Nextflow 最大的特色是以 channel 傳遞資料，process 是從前一步驟的工作目錄獲得資料，而非讀取輸出目錄的資料夾。換句話說，無論建立 output channel 還是匯出檔案到 publishDir，nextflow 都是以 process 的工作目錄為中心。由於每批次處理都會新建工作目錄，所以也確保了 process 輸出的檔案不會彼此覆寫，減少命名中間檔和管理暫存目錄的勞力（這些都由 nextflow 代勞了）。
