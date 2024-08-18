---
title: VSCodeVim 常用設定
date: 2022-12-28 22:09:40
tags: [vim, vscode]
categories:
---

本文紀錄一些有助於在 Vscode 使用 vim 的套件與快捷鍵設定。

<!--more-->
# 除了 `Esc`，還有哪些鍵適合作為模式切換快捷鍵？
Vim 被開發出來的時候，`Esc` 位於現今當代鍵盤 `Tab` 和 `Caps` 的位置。如今，`Esc` 被移到鍵盤左上角，已不若以往方便了。既然 VSCodeVim 支援重新設定快捷鍵，那麼有哪些鍵適合作為模式切換快捷鍵？

1. `jj`：位置很棒，只是打字打到 j 時，會有輕微的停頓感。缺點是編輯中文時，要頻繁切換輸入法。
2. `jk`：一次按兩顆鍵，但因為可以一起按，沒有連續按鈕的卡頓感，只是同樣要解決換模式前得切換輸入法的問題。
3. `Enter`*2：剛開始使用 vim 很容易忘記切回 normal 模式，既然中文輸入會頻繁輸入 enter讓文字寫入文件，enter 設定為切換模式的快捷鍵既不會干擾文字輸入也好記。只是碰到換行時會延遲等候第二次按鍵的訊號，有時還挺煩人的。
4. `Shift` + `Enter`：雖然要兩個按鈕，但位置其實不錯按，而且不會輸入新列的卡頓感。唯一的缺點是設置稍嫌麻煩。

## 前述三種方法要修改 setting.json
```json
"vim.insertModeKeyBindings": [
        {
            "before": ["CR", "CR"],
            "after": ["<Esc>"]
        }
    ],
    

```

## `Shift` + `Enter` 的方法要改 keybindings.json
```json
{
        //https://github.com/VSCodeVim/Vim/issues/2584#issuecomment-385561286
        "key": "shift+Enter",
        "command": "extension.vim_escape",
        "when": "editorTextFocus"
    }
```

# 使用相對列名
由於 vim 可以為指令添加數字以重複執行，所以為了計算方便，需要設置相對行號，設定方式為：
1. `Ctrl` + `,`，啟動 UI 設定
2. 搜尋 "Line numbers" 
3. 由 "on" 改為 "relative"，預設為絕對行號

然而，內建方式只能顯示絕對行號或相對行號，若要一併顯示，則要安裝 *Double Line Numbers* 套件。
1. `Ctrl` + `Shift` + `X`，開啟擴充套件。
2. 搜尋並安裝 *Double Line Numbers* 
3. `Ctrl` + `Shift` + `P`，輸入 `Double Line Numbers: Relative + Absolute`


# 怎樣設定快捷鍵
vscode 和 vim 有些彼此衝突的快捷鍵，例如常用的 `Ctrl` + `F` 或 `Ctrl` +`W` 等。依照個人習慣，可以選擇 vim 模式要啟用那些快捷鍵
1. `Ctrl` + `Shift` + `P`，開啟 VScode 擴充套件命令列
2. 搜尋 "open user settings"，輸入 `Enter` 開啟 `setting.json`
3. 新增以下設置來關掉 vim的快捷鍵
    ```json
        "vim.handleKeys": {
        "<C-d>": true,
        "<C-s>": false,
        "<C-w>": false,
        "<C-t>": false,
        "<C-z>": false
    },
    ```
這些是我常用的指令。

# 怎樣才不會忘記在切換模式後切為英文輸入法

由於 Normal mode 僅能以英文輸入法下指令，假如在 Insert mode 以中文輸入法編輯文件，使用預設的 `Esc` 切換模式後，仍維持中文輸入法，還要將輸入法切回英文，才能下達指令。為了解決這問題有兩個策略：

## 改變模式切換快捷鍵
第一種策略是把切換模式的快捷鍵設定為 `jj`，雖然不見得比較快，但一定能避免中文輸入法讓指令無法執行的問題。由於改用英文字母作為切換快捷鍵，所以在切換模式前一定要更換輸入法，所以能確保切換後一定是英文輸入法

1. `Shift`, then `jj`，切為 Normal mode
2. `i`，切為 Insert mode
3. `Shift`，切為中文輸入法

## 自動切換輸入法
另一種是呼叫自動切換輸入法的腳本，在切回 normal mode 時自動幫我們切換文字
1. 下載 (im.select.exe)[https://github.com/daipeihust/im-select] 放到任何你喜歡的路徑
2. `Ctrl` + `Shift` + `P`，開啟 VScode 擴充套件命令列
3. 搜尋 "open user settings"，輸入 `Enter` 開啟 `setting.json`
4. 新增以下設置以啟用輸入法自動切換
    ```json
    "vim.autoSwitchInputMethod.enable": true,
    "vim.autoSwitchInputMethod.defaultIM": "us",
    "vim.autoSwitchInputMethod.obtainIMCmd": "/path/to/im-select.exe",
    "vim.autoSwitchInputMethod.switchIMCmd": "/path/to/im-select.exe -s {im}"
    ```

透過這樣設置，在切回 normal mode 瞬間，就會變成英文輸入法。而進入 insert mode 時，就會回到當初離開時的輸入法。
# 關閉括號自動補全

VSCode 內建括號與引號自動補全，然而有時這還挺雞肋的，因為在括號內打完字還要使用方向鍵移到括號外才能繼續打字，若使用 Vim 還得切模式再移動，不管哪種方式都沒省太多功夫。因此，對於一般文本編輯或編輯程式碼的需求而言，似乎可以在 setting.json 添加以下設置來關閉自動補全。
```json
// automatical closing
    "editor.autoClosingQuotes": "never",
    "editor.autoClosingBrackets": "never",
```

# 調整視窗大小的快捷鍵
設定快捷鍵，就不用為了調整欄位大小，辛苦地讓游標對齊邊框了。
```json
	// put those into keybinding.json
    {
        "key": "ctrl+shift+d",
        "command": "workbench.action.decreaseViewSize"
    },
    {
        "key": "ctrl+shift+i",
        "command": "workbench.action.increaseViewSize"
    }
```

# 更新檔案瀏覽側欄
使用終端或指令新增或刪除檔案不會同步更新在檔案探索側欄，因為重新整理的按鈕太小了，所以我也設定快捷鍵輔助。
```json
	// put those into keybinding.json
	{
		"key": "ctrl+f5",
		"command":"workbench.files.action.refreshFilesExplorer",
		"when":"filesExplorerFocus"
	}
```
# 超棒的學習資源和網站
- [VsCodeVim 自動在normal mode切回英文輸入法的方法](https://ithelp.ithome.com.tw/articles/10291847)
- [vscode vim mode](https://www.blog.lasai.com.tw/2020/07/05/vscode-vim-mode/)