---
title: Nextflow｜讀取儲存於 JSON 的輸入資料
date: 2024-10-28 22:26:31
tags: nextflow
categories:
---

目前，nf-core 認證的分析流程皆[使用 CSV 管理輸入的樣本資料](https://nextflow-io.github.io/patterns/process-per-csv-record/)，這方法會逐列分析 CSV 檔每一列的資料。不過，其它任務排程或系統監控軟體可能更傾向使用 JSON 作為資料傳遞媒介。為了讓 Nextflow 能直接與那些軟體互動，需要能解析 JSON 格式並且逐條執行分析的功能。

<!-- more -->

Nextflow 讀取 JSON 的方法取決於輸入資料的鍵值設計。首先，我們能依照數據類型彙整資料，記錄該屬性在各樣本的確切數值（我稱為資料框模式）。在資料框模式中，key 對應樣本的一種屬性，value 則是紀錄各樣本屬性數值的列表。Nextflow 的 `-params-file` 可以直接讀取這種格式，以 `channel.of` 將參數轉換為 nextflow channel 之後，便能用 `transpose` operator 集中各樣本的屬性資訊。

`params.json`
```json
{
	"uuid": ["a", "b"],
	"fq1": ["/path/to/a_r1.fq", "/path/to/b_r1.fq"],
	"fq2": ["/path/to/a_r2.fq", "/path/to/b_r2.fq"],
	"meta": [{"gender":"f", "age":12}, {"gender":"m", "age":19}]
}
```

我們也可以先集中各樣本的屬性，讓用戶更容易理解特定樣本的相關資訊（我稱為物件模式）。在物件模式裡，一個樣本的所有屬性資訊都以字典形式儲存在列表的其中一項元素內，它依賴 `splitJson` operator 讀取檔案，再用 `map` 取出需要的數值，整理成分析所需的格式。

`input.json` 
```json
[
	{
		"uuid": "a",
		"fq1": "/path/to/a_r1.fq",
		"fq2": "/path/to/a_r2.fq",
		"meta": {
			"gender": "f",
			"age": 12
		}
	},
	{
		"uuid": "b",
		"fq1": "/path/to/b_r1.fq",
		"fq2": "/path/to/b_r2.fq",
		"meta": {
			"gender": "m",
			"age": 19
		}
	}
]
```

以下範例依據輸入的選項決定讀取 JSON 檔的方式：`-params-file <JSON>` 對應資料框模式，`--input <JSON>` 對應物件模式。順利讀取 JSON 檔後，程式將逐條印出每個樣本的屬性。

```groovy
process SHOW_INPUTS(){
/*
在標準輸出印出各樣本的所有資訊。
*/
input:
tuple val(uuid), path(fq1), path(fq2), val(meta)
output:
stdout
""" 
echo "uuid: ${uuid}, gender: ${meta.gender}, age: ${meta.age}, fq1: ${fq1}, fq2: ${fq2}"
"""
}
workflow {
	/* 依據輸入選項決定讀取 JSON 的方式
	*/
	params.input = false
	if ( params.input )  {
		// 以物件模式讀取 JSON 檔
		channel.fromPath(params.input)
		| splitJson()
		| map { m -> tuple(m.uuid, m.fq1, m.fq2, m.meta)}
		| set{ch_input}
		SHOW_INPUTS(ch_input).view()
	} else {
		// 以資料框模式讀取 JSON 檔
		ch_input = channel.of( [params.uuid, params.fq1, params.fq2, params.meta] ).transpose()
		SHOW_INPUTS(ch_input).view()
	}
}
```

實際執行的結果如下，兩種模式都能順利讀取 JSON，並且逐條執行分析。

```bash
$ nextflow run main.nf --input input.json (or nextflow run main.nf -params-file params.json)
 N E X T F L O W   ~  version 24.04.4
Launching `main.nf` [desperate_wiles] DSL2 - revision: f5b87f1e88
executor >  local (2)
[c3/14477d] SHOW_INPUTS (2) [100%] 2 of 2 ✔
uuid: a, gender: f, age: 12, fq1: a_r1.fq, fq2: a_r2.fq
uuid: b, gender: m, age: 19, fq1: b_r1.fq, fq2: b_r2.fq
```

---

- [Process per CSV record](https://nextflow-io.github.io/patterns/process-per-csv-record/)
- [splitJson operator](https://www.nextflow.io/docs/latest/reference/operator.html#splitjson)
- [Nextflow｜如何為輸入的資料建立辨識碼](https://5uperb0y.com/nextflow-combine-data-into-a-tuple-with-key/)