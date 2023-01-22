---
title: Nextflow｜如何為輸入的資料建立辨識碼 
date: 2023-01-09 23:50:16
tags: nextflow 
categories: [bioinformatics]
---
在 Nextflow 的框架，channel 媒介了 process 間的資料傳遞。雖然資料進出 channel 遵循先進先出 (first-in, first out, FIFO) 原則，但隨後 nextflow 會平行處理這些資料，所以 process 釋出資料的順序取決於執行速率，而不是原先輸入的順序（見下例）。

```groovy
// demoEmitOrder.nf
process A {
    input:
        val num
    output:
        stdout
    """
    echo "${num}"
    """
}

workflow {
    nums = channel.of("1", "2", "3")
    nums.view()
    A(nums).view() 
}
```
```bash
$ nextflow run demoEmitOrder.nf
1
2
3
1

3

2
```
Nextflow 這項特性使得輸入彼此相依的資料時要格外留意，因為可能在定序產物品管時，發生不同樣本的順逆序列（例如：a_r1.fa 與 b_r2.fa 合併）的狀況；又或是使用 GATK 這類需要參考基因體的工具時，無法為 `.fasta` 找到對應 `.dict` 和 `.fai` 檔的情形。

因此，本文將介紹如何將想要一起輸入的檔案以及其辨識碼組合成 tuple （例如：`["ID", "ID_r1.fastq", "ID_r2.fastq"]`），確保成對資料同進同出，還能透過辨識碼與其它資料合併。
<!--more-->

# 準備測試資料

首先建立雙端定序產物與人體基因體序列的測試資料，用以示範成對資料和多筆資料各自要如何組合成 tuple。

```bash
$ mkdir data
$ touch data/{a..c}_r{1,2}.fq           # 示範成對資料
$ touch data/{a..c}.{fasta,dict,fai}    # 示範多筆資料
```

接著，在 nextflow script (`*.nf`) 定義這些資料以建立 input channel。此處這麼做是為了簡便，但其實也可以將資料路徑寫到 JSON 檔，在執行 nextflow script 時，透過 `-c` 或 `-params-file` 讀入。要留意的是，欲組合在一起的資料於 nextflow 變項的順序要一致，

```groovy
inDir = "/path/to/data"
id = ["a", "b", "c"]
fq1 = ["$inDir/a_r1.fq", "$inDir/b_r1.fq", "$inDir/c_r1.fq"]
fq2 = ["$inDir/a_r2.fq", "$inDir/b_r2.fq", "$inDir/c_r2.fq"]
```

# 透過 nextflow operator
假設已經把辨識碼與輸入資料定義成前一節的形式，而且要組合在一起的資料於 nextflow 各變項內的順序也相同，那麼可以先建立包含全數資料的 tuple，再使用 `transpose` operator 轉置。

```groovy
workflow {
    channel.of([id, fq1, fq2]).transpose()
}
```

![`transpose` 能轉置 channel 的組成。](https://github.com/5uperb0y/blog-media/blob/main/combine-data-into-a-tuple-with-key_transpose.png?raw=true)

由於 channel 遵循 FIFO 原則，所以使用 `view()` 瀏覽結果時，確認資料釋出與輸入順序一致。
```
[a, /path/to/data/a_r1.fq, /path/to/data/a_r2.fq]
[b, /path/to/data/b_r1.fq, /path/to/data/b_r2.fq]
[c, /path/to/data/c_r1.fq, /path/to/data/c_r2.fq]
```
# 透過 bash in process 

假設辨識碼為檔名的一部份，那麼也可以用 bash script 擷取字串，並在 output channel 將擷取到的辨識碼與輸入的資料組合成 `tuple`。
```groovy
process addID {
    input:
        path fq1
        path fq2
    output:
        tuple env(id), path(fq1), path(fq2)
    """
        id=\$(basename -s _r1.fq ${fq1})
    """
}

workflow {
    addID(channel.fromPath(fq1), channel.fromPath(fq2))
}
```

由於經 process 處理，資料釋出順序已偏離輸入順序，紀錄的路徑也轉移到 process 的工作目錄了。
```
[a, /path/to/work/c2/3169d4c100067150f54629d968b7fb/a_r1.fq, /path/to/work/c2/3169d4c100067150f54629d968b7fb/a_r2.fq]  
[c, /path/to/work/2a/f8e10f5efe89d9c99d5814f5c7a151/c_r1.fq, /path/to/work/2a/f8e10f5efe89d9c99d5814f5c7a151/c_r2.fq]  
[b, /path/to/work/9b/b8295d80236b0871e5a197845b0ec1/b_r1.fq, /path/to/work/9b/b8295d80236b0871e5a197845b0ec1/b_r2.fq]  

```
# 透過 channel factory (限定成對資料)
如果辨識碼是檔名的一部份，而且資料又如雙端定序產物兩兩一組，那麼也可以使用 `fromFilePairs` 建立 `tuple`。簡言之，nextflow 會搜尋指定目錄下含有成對詞綴的檔案（此例為 `_r1.fq` 與 `_r2.fq`），擷取詞綴前的字串，將成對資料組合成帶有辨識碼的 `tuple`。

預設是組合成 `["id", ["path 1", "path 2"]]` 形式，所以此處使用 `flat:true` 攤平巢狀結構。
```groovy
workflow {
    channel.fromFilePairs("$inDir/*_r{1,2}.fq", flat:true)
}
```

```
[a, /path/to/data/a_r1.fq, /path/to/data/a_r2.fq]
[b, /path/to/data/b_r1.fq, /path/to/data/b_r2.fq]
[c, /path/to/data/c_r1.fq, /path/to/data/c_r2.fq]
```
# 結合 channel operator 和 groovy closure 
`fromFilePairs` 僅適用成對資料，若是多筆資料則要組合 nextflow operator 和 groovy closure 來完成。
1. 以 `fromPath` 建立 channel 時，搜尋不同詞綴的檔案
2. 以 groovy *getBaseName* 擷取檔名，再用 nextflow `map` 將檔名與檔案路徑組合成 tuple（例如：`a.fasta`→`［"a", "a.fasta"]`）
3. 以 `groupTuple` 組合含有相同辨識碼的 tuple（例如：`["a", "a.fasta"]` 與 `["a", "a.dict"]` 組合成 `["a", ["a.fasta", "a.dict"]]`）
4. 以 `map` 配合 groovy *flatten* 攤開巢狀 `tuple`

```groovy
workflow {
    channel.fromPath(["$inDir/*.dict", "$inDir/*.fasta", "$inDir/*.fai"])
        .map{it -> tuple(it.getBaseName(), it)}
        .groupTuple()
        .map{it -> it.flatten()}
}
```
此處假設檔名等同辨識碼，若辨識碼是檔名的子字串或其它組合，那就要參考 groovy 的字串處理方式。這裡使用 groovy 是為了要在 nextflow script 完成任務，若不限此條件，那也可以新增 process，改用熟悉的腳本語言來處理字串（參考 bash script 的方法）。
```
[a, /path/to/data/a.fasta, /path/to/data/a.dict, /path/to/data/a.fai]
[b, /path/to/data/b.fasta, /path/to/data/b.dict, /path/to/data/b.fai]
[c, /path/to/data/c.fasta, /path/to/data/c.dict, /path/to/data/c.fai]
```

# 怎麼讀取資料
前述建立的 `tuple` 可用以下方式讀入 process，也可以用 `join` 組合，確保成對或多筆資料能夠在 process 同進同出。
```groovy
process readInput {
    input:
        tuple val(id), path(fq1), path(fq2)
        tuple val(id), path(fasta), path(dict), path(fai)
    output:
        stdout
    """
    echo "${id}, ${fq1}, ${fq2}"
    echo "${id}, ${fasta}, ${dict}, ${fai}"
    """
}
```
# 結論

最後使用一張表總結將資料組合為 `tuple` 的方法。

| 比較項目 | operator       | bash in process | fromFilePairs | operator & closure |
| -------- | -------------- | --------------- | ------------- | ------------------ |
| 檔案數   | 不限           | 不限            | 兩筆          | 不限               |
| 定義     | 需定義檔案路徑 | 需定義檔案路徑  | 檔案存放路徑  | 檔案存放路徑       |
| 釋出順序 | 輸入順序       | 完成順序        | 輸入順序      | 輸入順序           |
| 檔案位置 | 輸入路徑       | 工作目錄        | 輸入路徑      | 輸入路徑           |



