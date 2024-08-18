---
title: 運用 VSCode snippets 管理並活用個人寫作語料庫
date: 2024-07-18 22:43:21
tags: [writing, tools]
categories:
---

學習寫作的一個有效方法是透過閱讀優秀作品，並細心記錄值得借鑒的用字與段落，進而運用在自己的作品中。市面上已經有場景辭典或詞彙工具書幫我們分類詞條，但是自建語料庫能按照習慣規劃，更符合個人需求。

然而，無論是使用現成的還是自建的語料庫，查閱過程往往會打斷寫作節奏，降低其實用性。那麼，要如何管理這些精心收集的語料，才能及時更新又方便取用呢？這篇文章將介紹如何使用 VSCode snippets 的提示與管理功能，將語料庫的應用整合到 markdown 文檔的寫作流程中。

<!--more-->

VSCode snippets 原本用於管理常用程式碼片段，它允許用戶透過關鍵字快速查詢和插入這些片段。憑藉這些特性，再稍加調整後，VSCode snippets 同樣能用來管理寫作的常用詞條，幫助我們推敲用字與學習新詞彙。

# VSCode snippets 使用教學

## 啟用 VSCode markdown snippets

由於 markdown snippets 功能預設為關閉狀態，所以需要編輯設置檔來啟用它。

1. 按下 `Ctrl` + `Shift` + `p` 打開指令列
2. 查詢 "Preferences: Open Settings (JSON)" 並開啟 `setting.json`
3. 在 `setting.json` 添加以下設置[^1]：
   ```json
   // markdonw snippets activation
    "[markdown]": {
		"editor.quickSuggestions": true
    },
   ```
4. 儲存並離開

[^1]: 參考[[踩雷紀錄] Vscode markdown snippets 無作用](https://bingdoal.github.io/others/2021/12/markdown-snippets-not-working-in-vscode/)

## 編輯 user snippets

1. 依序點選 "File"、"Preferences"、"Configure User Snippets"
2. 在彈出的選單中選擇 "markdown.json"[^2]
3. `markdown.json` 使用以下格式記錄用戶的 snippets
   ```json
   "[提示文字]": {
		"prefix": "[觸發文字]",
		"body": "[詞條主體]",
	 	"description": "[詞條描述]"
	 }
   ```
   - 提示文字（key）：顯示於建議用字選單右側，概述詞條內容
   - 觸發文字（prefix）：輸入後，VSCode 會在它的右下方顯示相符的建議用字選單
   - 詞條主體（body）：將插入的實際詞條內容
   - 詞條描述（description）：詞條的補充說明，不會出現在建議用自選單中

[^2]: 這裡介紹的是編輯 global snippets 的方式。VSCode 也提供用戶依照專案資料夾調整 snippets 的彈性，所以可以為不同文章類型準備各自的語料庫，詳見官方說明：[Snippets in Visual Studio Code](https://code.visualstudio.com/docs/editor/userdefinedsnippets#_project-snippet-scope)

## 使用個人語料庫

在啟用提示功能和添加個人語料後，即可在 markdown 文檔使用個人語料庫。以下範例使用器官來分類詞彙[^3]，只要輸入 "@鼻"，VSCode 就會自動建議相關用字。因為 user snippets 支援模糊搜索，即使輸入不完整，也能提示部分匹配的詞條。

![](https://raw.githubusercontent.com/5uperb0y/thesaurus/main/demo.png)

這個示範使用以下的設置檔。為了提升建議選單的可讀性，我把詞條、類別與解釋三種資訊並列於 "prefix"，並在每個 "prefix" 前添加特殊符號，確保提示功能只在必要時被觸發。另外，我也省略了不會顯示的 "description" 以節省篇幅。

```json
// markdown.json
{
	"秀挺": {
		"prefix": "@鼻  秀挺：秀麗高挺",
		"body": "秀挺"
	},
	"高峙": {
		"prefix": "@鼻  高峙：高挺",
		"body": "高峙"
	},
	"齊勻高整": {
		"prefix": "@鼻  齊勻高整：形容鼻樑勻稱高挺",
		"body": "齊勻高整"
	},
}
```

[^3]: 範例取自於《如何捷進寫作詞彙》（黃淑貞。2023。商周出版）

# 改善個人語料庫的管理方式

收集到的詞條可以使用 [snippet generator](https://snippet-generator.app/?description=&tabtrigger=&snippet=&mode=vscode) 轉換為 JSON 格式，再添加到 `markdown.json` 當中。不過這個流程較為繁瑣，`markdown.json` 本身也不易瀏覽與操作。因此，我打算從以下幾個方向改善語料庫的管理方式。[^4]

[^4]: 這裡提到的程式碼與表格可到我的 [Github](https://github.com/5uperb0y/corpus) 查看。

## 改以表格紀錄個人語料庫

相較於 JSON 格式，純文字表格更容易閱讀與維護。畢竟，表格既能以 Git 進行版本控管，還相容於 Excel/Google sheet 等試算表工具。

我採用 TSV 格式儲存語料，把資訊分為觸發符號、類別、詞彙與解釋四欄。觸發符號除了防止誤用以外，也能用來篩選詞條。例如 "@" 表示詞彙，而"~"表示片語，以此區分不同語料性質。類別則參考《如何捷進寫作詞彙》和《場景設定創意辭海》的架構，但內容會由我自己補充。

```tsv
#觸發符號	類別	詞彙	解釋
@	鼻	秀挺	秀麗高挺
@	鼻	高峙	高挺
```

## 使用程式把表格轉換為 JSON 格式

由於表格需要額外轉檔才能符合 VSCode snippet 的格式要求，我寫了一個腳本。它能夠逐列切割表格，重新組合各欄元素，填入 JSON 檔中。

```bash
#!/bin/bash
echo "{"
grep "^[^#]" "$1" | while IFS=$'\t' read -r trigger category word explanation
do
    cat <<EOF
  "$word": {
    "prefix": "$trigger$category  $word：$explanation",
    "body": "$word"
  },
EOF
done
echo "}"
```

## 用 Git Hook 實現流程自動化

為了減輕格式轉換的負擔，可以在專案資料夾新增 `.git/hook/pre-commit`[^5]，讓程式在每次 commit 前，自動生成 VSCode snippet 要求的 JSON 格式。

[^5]: git hook 的自動化教學可參考[官方文件](https://git-scm.com/book/zh-tw/v2/Customizing-Git-Git-Hooks)

```bash
./snippet_generator.sh corpus.tsv > corpus.json
```

接著，為 `corpus.json` 建立軟連結，取代 VSCode 預設的 snippet 設置檔。如此一來，就能即時套用語料庫的修改。

```bash
ln -s /path/to/corpus.json /path/to/markdown.json
```

# 結論：閱讀、紀錄、寫作、潤稿

綜上所述，應用 VSCode snippets 提供的自動補全、建議選單以及詞條管理等功能，能夠降低管理與應用語料庫的成本，讓學習、紀錄與運用等過程更加緊密連結。理想的使用情境是在閱讀後，記錄值得值得的詞條與段落，並在撰寫文章時加以利用。透過定期回顧與編修語料庫，能逐步精煉其內容，讓它更符合個人所需。