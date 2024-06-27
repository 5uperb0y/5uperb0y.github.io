---
title: Nextflow｜模仿 docstring 來註解 process and workflow
date: 2024-01-11 21:04:22
tags: nextflow 
categories:
---

本文介紹使用 docstring 的好處，並且探討如何在 Nextflow 使用多行註解模仿 python 的 docstring，從而改善撰寫與維護開發文件的流暢性。
<!-- more -->

python 的 docstring 是 function 的強大特性，能讓開發者在參數類別之外，補充關於 function 的用途、假設條件以及運作原理等額外資訊。透過 `__doc__` 屬性，docstring 能夠被其他程式讀取，實現開發文檔自動化等功能。

更重要的是，docstring 位於 function 內部，這讓開發者更動程式碼後，能夠輕易地同步更新文檔，減少文檔與程式碼不一致的情形。

然而，Nextflow 目前並不支援 docstring 功能。不過，我們可使用多行註解（即介於 `/*` 與 `*/` 之間的文字）來模仿 docstring 的效果。這對於協助開發者了解當前的 module（無論是 process 或 workflow）都很有幫助。

鑒於 Nextflow 的註解無法像 python docstring 那樣透過物件方法呼叫，我寫了一支小腳本來[節錄這些資訊](https://github.com/5uperb0y/nf-tools/tree/main/src/get_docstring)。

簡言之，我將位於 module 名稱與 directive 之間的多行註解定義為 Nextflow 的 docstring，例如：

```groovy
process <name> {
	/* your comments */
	directives
	"""
	your code
	"""
}
```
這支腳本運用正則表達式（感謝 chatgpt 幫忙想出裏頭可怕的正則表達式）來抓取 docstring，將其與 module 類型（即 workflow 或 process）和名稱一併輸出。
```bash
$ python get_docstring.py workflow.nf
process FASTQC
this text is from FASTQC
workflow VARIANT_CALLING
this text is from VARIANT_CALLING
```
如此一來，我們就能取得 docstring 的內容，不僅方便瀏覽 module 功能，也便於自動生成文檔。

雖然目前功能還蠻陽春的，但之後也許可以新增其他實用功能，例如支援以名稱篩遠 docstring，或是使用 groovy 開發，使之能在 Nextflow 內部調用。