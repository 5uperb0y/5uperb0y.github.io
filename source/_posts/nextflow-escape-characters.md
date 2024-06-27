---
title: Nextflow｜程式碼區塊的跳脫符號 (escape characters)
date: 2022-11-10 00:37:46
tags: nextflow
categories:
---
本文介紹程式碼定義變項的符號與 nextflow 內建語法衝突時，有哪些選項可以解決之。

<!--more-->
## Nextflow 管理程式的單位
Process 是 Nextflow 管理程式的單位，其中必然包含 script 區塊來定義想執行的程式。除了 script 區塊，還有 directives（環境設置）、inputs（輸入資料）、outputs（輸出資料）等非必要但有助流程控管的區塊。

在執行程式前，Nextflow 會解讀 script 區塊帶有 `$` 前綴的變項，代入 inputs、parameters 或 config files 的對應內容。
以 process fastQC 為例，nextflow 將 `fq` (input) 的內容代入 `${fq}` (script) 後才執行程式，再將輸出結果透過 `fqc`(output) 傳遞。

```
process fastQC {
    publishDir "${params.outdir}/qc", mode: 'copy', overwrite: true
    input:
        path fq
    output:
        path "*_fastqc.{zip,html}" into fqc
    """
    fastqc --nogroup -q ${fq}
    """       
}
```
## 程式碼與nextflow 語法衝突
值得留意的是，perl 或 bash 等語言也使用 `$` 標記變項（例如：`$i`、`$path`）。Nextflow 無法區分以 `$` 為前綴的變項是定義在 script 內（script 變項），還是得自於 inputs, parameters 或 config files（nextflow 變項）。

因此，如果程式碼出現以 `$` 定義或呼叫的 script 變項 ，nextflow 便有可能因為無法在 inputs、parameters 或 config files 找到對應內容而報錯。

舉以下案例來說，`title`/`$title` 是使用 bash 定義的 script 變項。然而，nextflow 卻誤判該變項的來源，以至於找不到其內容（`No such variable`）

```
# printPath.nf
processs printDir {
    output:
        stdout
    """
    title="The current directory is,"
    echo "$title \n $PWD"
    """
}
workflow {
    printDir().view()
}
```

```
$ nextflow run printPath.nf
Error executing process > 'printDir'

Caused by:
    No such variable: title -- Check script 'printDir.nf' at line: 4
```

對於這個問題，主要的解法是透過更換標記符號來區分 script 變項和 nextflow 變項。

## 更換 nextflow 變項的標記符號
第一種方法是改以三個單引號夾註程式碼，提示 nextflow 改以 `！{}` 標記 nextflow 變項。

```
processs printDir {
    output:
        stdout
    ```
    title="The current directory is,"
    echo "$title \n $PWD"
    ```
}
```

## 更換 (跳脫) script 變項的標記符號
若 script 變項不多，也可以在 `＄` 前面加上反斜線(`\$`)，提示 nextflow 忽略 `$` 符號。
```
processs printDir {
    output:
        stdout
    ```
    title="The current directory is,"
    echo "\$title \n $PWD"
    ```
}
```

## 更動跳脫字元
由於 nextflow 的底層是 groovy，所以除了 script 與 nextflow 的語法衝突，也可能碰到 script 與 groovy 語法衝突的狀況。

以下兩個案例都在 script 區塊使用反斜線來編輯文字。由於反斜線是 groovy 預設的跳脫符號，所以執行這些 processes 時也可能發生編譯錯誤。
```
# sepReplace.nf
process sepReplace {
    output:
        stdout
    '''
    str="A,B,C"
    echo $str | sed "s/,/\"/g"
    '''
}
workflow {
    sepReplace().view()
}
```
```
$ nextflow run sepReplace.nf
(skip)
Command error:
    .command.sh line3: unexpected EOF while looking for matching `"'
```

或是[這個案例](https://github.com/nextflow-io/nextflow/issues/67)。

```
# strRemove.nf
process strRemove {
    outpur:
        stdout
    '''
    echo "Hello lg:en" | sed "s/.*lg:\(.*\).*/\1/"
    '''
}
workflow {
    strRemove().view()
}
```

```
$ nextflow run strRemove.nf
Script compilation error
- file : /path/to/workdir/strRemove.nf
- cause: Unexpected character: '\'' @ line 4, column 7.
       '''
         ^
```

碰到這種情形，首先要使用 `$/` 和 `/$` 夾註程式碼，提示 groovy 改以 `$` 作為跳脫符號，以區分 script 和 groovy 語法。

接著，在 script 變項的 `$` 前面再加一個 `$` (`$$`)，提示 nextflow 忽略 `$` 符號，以區分 script 和 nextflow 語法。

```
process sepReplace {
    output:
        stdout
    $/
    str="A,B,C"
    echo $$str | sed "s/,/\"/g"
    /$
}
```

## 封裝為腳本
不過，如果程式碼用到大量變項，前述方法會讓程式碼顯得囉嗦又不易讀。此時，可以考慮將程式碼封裝為腳本，再從 script 區塊呼叫。

```
#!/bin/bash
# sepReplace.sh
str="A,B,C"
echo $$str | sed "s/,/\"/g"
```

```
process sepReplace {
    output:
        stdout
    """
    $baseDir/sepReplace.sh
    """
}
```


## 結論
因為 nextflow、bash、perl 都以 `$` 標記變項，所以執行 process 時可能會無法正確辨識 script 區塊內的變項來源而報錯。可能的解決途徑如下，

- **nextflow 變項少**：以三個單引號夾註程式碼，再以 `！{}` 標記 nextflow 變項
- **script 變項少**：以 `\` 跳脫 script 變項的 `$`
- **出現反斜線**：以 `$/` 和 `/$` 夾註程式碼，再以 `$` 跳脫 script 變項的 `$`
- **程式碼龐雜**：將程式碼封裝為腳本，再從 process 呼叫腳本

