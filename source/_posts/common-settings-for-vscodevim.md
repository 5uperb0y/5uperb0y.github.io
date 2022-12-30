---
title: VscodeVim 常用設定
date: 2022-12-28 22:09:40
tags: vim, vscode 
categories: [programming, tools]
---

本文紀錄一些有助於在 Vscode 使用 vim 的套件與快捷鍵設定。

<!--more-->

## 怎樣設定快捷鍵
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

## 怎樣才不會忘記在切換模式後切為英文輸入法

由於 Normal mode 僅能以英文輸入法下指令，假如在 Insert mode 以中文輸入法編輯文件，使用預設的 `Esc` 切換模式後，仍維持中文輸入法，還要將輸入法切回英文，才能下達指令。為了解決這問題有兩個策略：

### 改變模式切換快捷鍵
第一種策略是把切換模式的快捷鍵設定為 `jj`，雖然不見得比較快，但一定能避免中文輸入法讓指令無法執行的問題。由於改用英文字母作為切換快捷鍵，所以在切換模式前一定要更換輸入法，所以能確保切換後一定是英文輸入法

1. `Shift`, then `jj`，切為 Normal mode
2. `i`，切為 Insert mode
3. `Shift`，切為中文輸入法

### 自動切換輸入法
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

## 超棒的學習資源和網站
- [VsCodeVim 自動在normal mode切回英文輸入法的方法](https://ithelp.ithome.com.tw/articles/10291847)
- [vscode vim mode](https://www.blog.lasai.com.tw/2020/07/05/vscode-vim-mode/)