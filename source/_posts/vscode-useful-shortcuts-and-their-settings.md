---
title: VSCode｜常用快捷鍵與個人設置
date: 2023-01-08 02:20:58
tags: vscode 
categories: 
---
VSCode 有許多快捷鍵設置，本文記錄一些我覺得挺實用的設置

<!--more-->

# 開啟個人快捷鍵配置

1. `Ctrl` + `Shift` + `P`
2. 搜尋 "Preferences: Open Keyboard Shortcuts (JSON)" 

# 調整當前子頁尺寸

```json
    {
        "key": "ctrl+shift+d",
        "command": "workbench.action.decreaseViewSize"
    },
    {
        "key": "ctrl+shift+i",
        "command": "workbench.action.increaseViewSize"
    }
```

# 開啟連結

設定條件是為了避免快捷鍵彼此衝突，點連結的行為大概只有在編輯 md 檔案時發生，所以才這樣設置。

```json

    {
        "key": "ctrl+l",
        "command":"editor.action.openLink",
        "when": "editorTextFocus && editorLangId == 'markdown'" 
    }

```
# 分頁管理
`Alt` + `[0-9]`：切換至指定分頁

# 刷新側欄檔案目錄
`Ctrl` + `0`：開啟檔案目錄，再按一次則聚焦檔案目錄

```json
    {
        "key": "ctrl+F5",
        "command":"workbench.files.action.refreshFilesExplorer",
        "when": "workbench.explorer.fileView.focus"
    }
```
# 游標動畫
VSCode 有一系列滑順特效動畫，能讓游標移動與視窗滑動的軌跡帶有殘引，看起來更加滑順。我自己覺得彷彿是影片從 720p 改善到 4k 的滑溜溜感。
1. `Ctrl` + `,`
2. 搜尋 "smooth"
3. 勾選 "Cursor smooth Caret Animation"
