---
title: Vim｜Vimium，學習 Vim 的新途徑
date: 2022-12-31 01:06:11
tags: vim
categories:
---

[Vimium](https://chrome.google.com/webstore/detail/vimium/dbepggeogbaibhgnhhndojpepiihcmeb) 是 Chrome 的一款擴充套件，它借鑒了 Vim 的鍵位設計與操作邏輯，讓用戶只需要鍵盤便能執行分頁管理、連結點擊與頁面瀏覽等操作。

由於瀏覽器終究不等同編輯器，所以 Vimium 並沒有移植 Vim 所有的功能與指令。然而，即使只納入 Vim 的部分性質，Vimium 仍顯著改善了 Chrome 的鍵盤瀏覽體驗。以點擊連結為例，Chrome 的預設作法是[慢慢用 `Tab` 切到連結位置](https://5uperb0y.com/navigate-websites-with-keyboard/)；Vimium 則會自動標記視窗內的連結，用戶只要鍵入標記字符即可開啟連結。因為最多輸入三個字即可開啟連結，所以有時甚至比滑鼠點擊還迅速。

對於習慣鍵鼠操作卻有心學習 Vim 的用戶而言，一時要以 Vim 取代既有的工作流程，可能是繁瑣且費時的過程。若先從 Vimium 著手，在體驗鍵盤瀏覽網頁的流暢感之餘，也能讓 Vim 操作的抽象性質融入日常生活（例如：查資料與逛論壇）。待初探了 Vim 的設計緣由，再逐步精進其他操作模式，也不失為一種學習 Vim 的可行策略。

<!--more-->

# 能從 Vimium 
借鑒了 Vim 鍵位設計與操作邏輯，無論是管理分頁、點擊連結或瀏覽頁面都能夠只憑鍵盤操作。

- **習慣在 normal 模式瀏覽文件的方式**：
- **理解以 home row 為核心的鍵位安排**：Vim 的鍵位設計以 home row（`f` 和 `j` 那列）為核心，不用為了點選特殊控制鍵而過度伸展手指（例如：`Shift` + `Alt` + `F`），所有操作都不超過字母鍵盤，減少了手部位移的距離。
- **學習 vim 實用功能的邏輯與效果**：Vimium 移植了
- **了解自訂快捷鍵的意義**：相較於滑鼠，快捷鍵操作有更多空間能夠因應個人的需求調整。
- 
- 能夠更為習慣 normal mode 的瀏覽方式j，
- 體會以 home row 為中心的指令鍵位安排
- 習慣使用垂直移動游標，例如 `j` & `k`
- 了解 Vim easymotion 跳轉的邏輯與效果
- 學會使用 `/` 搜尋文字
- 體驗指令組合的效果
- 依據需求自訂快捷鍵
- 使用書籤功能

# 學會 Vimium 只要三分鐘
> Press `f` to master Vimium (and navigate internet without a mouse.)

Vimium 最大的特色是能透過「標記與跳轉」的模式開啟連結。一旦按下 `f`，Vimium 便會為視窗內所有可互動的項目（例如連結、欄位與按鈕）標註辨識碼，接著只要輸入辨識碼即可開啟連結。

![鍵入 `f` 後, Vimium 會為視窗內的可互動物件標記辨識碼，鍵入辨識碼即可開啟連結（使用 Vimium 操作 Google search)](https://github.com/5uperb0y/blog-media/blob/main/start-learning-vim-from-vimium_easymotion.png?raw=true)
- `L`：開啟「Vimium」
- `SK`：開啟「工具」，透過 Vimium 的連結辨識碼，免除網頁設定欄位快捷鍵的負擔
- `SA`：切換至「搜尋列」，可以編輯搜尋內容


雖然 Vimium 有一套基於 home row 指令，Chrome 與 Windows 內建快捷鍵已能直觀地管理分頁（例如 `Ctrl` + `T`）與滑動視窗（例如方向鍵與空白鍵），要配合瀏覽網頁的需求是綽綽有餘。例如Vimium 配合特殊控制鍵，即可在不同分頁開啟連結。
| 行為                       | 快捷鍵                 |
| -------------------------- | ---------------------- |
| 於當前分頁開啟             | `f`, then ID           |
| 於新分頁開啟               | `f`, then `Shift` + ID |
| 於新分頁開啟並切換至新分頁 | `f`, then `Ctrl` + ID  |


# 以 Vim style 瀏覽網頁 

鍵鼠操作以 `Ctrl`、`Home`、`PageUp` 等控制鍵，

## 移動游標的方式

| 行為             | Vimium 快捷鍵 | Vim 快捷鍵   | Chrome 快捷鍵 |
| ---------------- | ------------- | ------------ | ------------- |
| 移至頂部         | `gg`          | `gg`         | `Home`        |
| 向上移動半個視窗 | `u`           | `Ctrl` + `u` | -             |
| 向上移動一列     | `k`           | `k`          | `↑`           |
| 向下移動一列     | `j`           | `j`          | `↓`           |
| 向下移動半個視窗 | `d`           | `Ctrl` + `d` | -             |
| 移至底部         | `G`           | `G`          | `End`         |

## 分頁管理
分頁管理是網頁瀏覽的特殊需求，所以 Vimium 就沒有照搬 Vim 的快捷鍵，而是將那些負責編輯或是功能與其他移動指令相近的按紐分配給分頁管理。
![Vimium 分頁管理與頁面瀏覽的快捷鍵](https://github.com/5uperb0y/blog-media/blob/main/start-learning-vim-from-vimium_tab-operation.png?raw=true)
比對 Vimium 指令與 Chrome 快捷鍵會發現，後者需要用 `Ctrl` 和 `Alt` 等特殊控制鍵去定義。此外，觀察鍵盤位置分布可得知，Chrome 快捷鍵位置集中在兩側，似乎也是為了因應特殊控制鍵的分布以及持滑鼠的手的擺放位置。然而，瀏覽網頁時其實沒有文字輸入的需求，所以文字區塊的鍵盤等於閒置了。
| 行為     | Vimium 指令 | Chrome 快捷鍵           |
| -------- | ----------- | ----------------------- |
| 開新分頁 | `t`         | `Ctrl` + `T`            |
| 關閉分頁 | `x`         | `Ctrl` + `W`            |
| 上一頁   | `H`         | `Alt` + `←`             |
| 下一頁   | `L`         | `Alt` + `→`             |
| 直達分頁 | `#g0`       | `Ctrl` + `#`            |
| 上一分頁 | `J`         | `Alt` + `Shift` + `Tab` |
| 下一分頁 | `K`         | `Alt` + `Tab`           |

## 數值參數
就目前使用下來的想法，為什麼稱呼 Vimium 的行為為「指令」，而chrome的操作為「快捷鍵」，這是因為 Vimium 移植了 Vim 的機制，視每次操作為單元指令，指令可彼此疊加與重複創造出更複雜的行為。Vim 指令的行為分為 motion, replication factor, and operator。前述提及的所有指令都可算是 motion，亦即移動游標數個單位，例如一個視窗、一列、整個頁面等。這些指令都可以再前面冠上 replication factor 來增加執行次數 <replication factor> <motion>。

而 operator 則是對文字的編輯，這包含了 c (change)、y (yank拖)、d(delete)等，
Vimium 保留了 Vim 重複計數 (repeat count or replication factor) 的機制，用戶只要在命令前加上數字，便能指定命令的重複執行次數。

- `4t`：開啟四個分頁
- `3x`：關閉前三個分頁
- `2d`：下移一個視窗(半個視窗 * 2)
- `3g0`：直達第三個分頁

## 頁面書籤
1. `mc`，建立分頁內標記，`c` 可替換為其他小寫字母
2. `mC`，建立跨分頁標記，`C` 可替換為其他大寫字母
3. ``c`，跳轉至標記的位置


## mode 切換
其實 Vimium 也有 insert 與 visual 模式，但相當罕用，原因如下：
1. 網頁瀏覽唯一的編輯機會是調整網址
2. 若有大量編輯需求，也是在 google doc 或 Rstudio 這種瀏覽器編輯器編輯，裏頭應該有對應的 vim 套件才對
3. 調整網址或更新關鍵字幾乎沒有重複操作或複雜編輯的需求

Vimium 的 insert 模式與其說是用於編輯文字，不如說是為了暫時脫離 Vimium 指令模式。Normal 模式的指令與快捷鍵在 Insert 模式無效，在 youtube, gmail 等內建快捷鍵相當完善的網頁時，使用 insert mode 就能用網頁預設的快捷鍵，避免衝突。

# 設計自己專屬的快捷鍵組合

鼓勵用戶，Vimium 支援了自訂以下項目的功能，


首先鍵入 `?` 開啟 "Vimium Help"，再點選 "Options" 進入設定面板，即能看到以下設定

## "Excluded URLs and keys"
這設定能限定特定網域適用的 Vimium 快捷鍵，以避免其覆蓋網頁或瀏覽器預設的快捷鍵。舉例來說，Vimium 的 `/`（搜尋頁內文字）會覆蓋 Google search 的 `/`（重返搜尋列）。

如果我已經很習慣使用 `Ctrl` + `F` 來查找頁內文字或是使用 `/` 來編輯搜索關鍵字，可以透過編輯 "Patterns" 和 "Keys"，讓 Vimium 的 `/` 設定不套用在 Google search 上。

![編輯 "Excluded URLs and keys" 來設定 Vimium 快捷鍵的適用範圍（Keys 預設是禁用全數快捷鍵，而 Patterns 可用正則表達式編寫規則）](https://github.com/5uperb0y/blog-media/blob/main/start-learning-vim-from-vimium_excluded-urls.png?raw=true)


## "Custom key mapping"
這設定能調整預設的快捷鍵以符合用戶的使用習慣。好比說，如果不習慣 `h` `j` `k` `l` 的操作，可以透過以下方式改成動作遊戲常用的 `a` `s` `w` `d` (`map <key> <command>`)。
```bash
# 改以 wasd 配置移動視窗
map a scrollLeft 
map s scrollDown
map w scrollUp
map d scrollRight
```
進階一點也能為常用網址設置快捷鍵 (`map <key> createTab <url>`)
```bash
# 鍵入 @tl 以開啟 Google 翻譯
map @tl createTab https://translate.google.com.tw/?hl=zh-TW
```

## "Previous patterns and Next patterns"
輸入 `[[`（上一頁）或 `]]`（下一頁）之後，Vimium 會搜尋並開啟頁面內含「上一頁」或「下一頁」的連結。這功能完全解決了 Google search 無法迅速跳轉到下一頁搜索結果的問題。然而 Vimium 預設是辨識英文和箭頭符號，所以要應用於中文網站時得自行添加搜索標的。
1. 點選 "Show Advanced Options"
2. 編輯 "Previous patterns" 與 "Next patterns" 
3. 依照自己常用的網頁添加搜索標的（例如：Google search，上一頁/下一頁；ptt.cc，</>；巴哈姆特，◄/►）
   
![可以檢查常用網站的上一頁/下一頁來調整快捷鍵偵測的字串](https://github.com/5uperb0y/blog-media/blob/main/start-learning-vim-from-vimium_page-navigation.png?raw=true)
# 結論
《原子習慣》，這些訣竅也適用於學習新事物。
1. 善用 `f` 點擊連結
2. 維持原本的瀏覽網頁
3. 透過 `?` 查詢指令
4. 學習使用 vim-like 指令瀏覽與管理分頁
5. 配合習慣，自訂適合自己的快捷鍵組合 