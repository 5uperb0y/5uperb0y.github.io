---
title: 在 Window 上使用 VSCode 建立 Tabletop simulator 的 Lua 腳本開發環境
date: 2023-07-22 23:35:58
tags: lua
categories:
---
# tabletop simulator VSCode 開發環境設定
Tabletop Simulator 是一款多人桌遊模擬器，提供強大的 Lua 腳本支援，讓玩家可以開發自訂遊戲。

然而，遊戲內建的腳本編輯器比記事本還難用，所以如果想要編輯複雜的腳本，仍有必要使用外部編輯器。鑒於網路上的資源稀少，我想在本文分享一下建立開發環境的經驗和總結。
<!-- more -->

# 必要軟體安裝與設定
首先，我們要完成必要軟體的安裝與設定。

1. 安裝 Steam 並購買 [Tabletop simulator](https://store.steampowered.com/app/286160/Tabletop_Simulator/)。
2. 安裝 [VSCode](https://code.visualstudio.com/)。雖然官方推薦的 Lua 開發環境是 Atom，但 VSCode 的功能更加強大，近幾年也比較廣泛被使用（參考 [Github 停止支援 ATOM IDE 的新聞](https://github.blog/2022-06-08-sunsetting-atom/)）。
3. 在 VSCode 當中安裝 [Tabletop Simulator Lua](https://marketplace.visualstudio.com/items?itemName=rolandostar.tabletopsimulator-lua) 套件。這款套件能從遊戲儲存檔擷取 Tabletop Simulator 的 Lua 腳本，讓我們直接能在 VSCode 中開發，並享有自動補全、檔案瀏覽以及其他擴充套件的功能。

# 連結遊戲與編輯器
接著要設定遊戲與編輯器的連結
1. 在 Steam 中啟動 Tabletop Simulator，並開啟任意一款遊戲（步驟為：`Create`、`Singleplayer`、`Classic/DLC/Workshop`、挑選遊戲）。
2. 打開 VSCode 時，Tabletop Simulator Lua 套件會自動辨識遊戲所在的資料夾（在 Windows 預設為 `%UserProfile%\Documents\My Games\Tabletop Simulator\Saves\`），將遊戲腳本 (`*.lua`)和設置檔 (`.xml`) 從遊戲儲存檔 (`*.json`) 獨立出來，並且放置到暫存資料夾中（預設為 `%UserProfile%\AppData\Local\Temp\TabletopSimulator\Tabletop Simulator Lua`。
3. 如果套件沒有自動更新或是想要更換其他遊戲時，可以鍵入 `Ctrl`+`Alt`+`L` 更新連結。

# 編輯遊戲腳本

現在，我們已經可以在 VSCode 當中編輯遊戲腳本了。Lua 腳本是遊戲儲存檔的一部份，而遊戲儲存檔則是一個 `JSON` 檔，記錄了啟動遊戲所需的所有元素。Tabletop Simulator Lua 的功能即是從儲存檔中擷取 Lua 腳本，並且轉換為適合閱讀與編輯的格式。

編輯完成後，按下 `Ctrl` + `Alt` + `S` 儲存並重啟腳本。此時，Tabletop Simulator 會將腳本變動更新到遊戲儲存檔，讓遊戲呈現腳本所做的變動。

如果希望在編輯腳本的同時能觀察遊戲畫面，有兩種方式可以達成：

首先，將 Tabletop Simulator 設定為視窗模式（`Menu`、`Configuration`、`Graphics`、取消勾選 `Fullscreen`）。
- 使用 Windows 視窗分割功能 （`Win` + `←`/`→`/`↑`），同時陳列編輯器與遊戲視窗。
- 下載 Windows PowerToys，這是一個功能強大的 Windows 工具集，其中包含了能將遊戲視窗固定在螢幕上的功能。([always on top feature](https://learn.microsoft.com/en-us/windows/powertoys/always-on-top))


# 建立專案資料夾

由於無法指定 Tabletop Simulator 遊戲儲存檔的存放位置，所以想要進行版本控管或專案管理時需要一些權變措施。

1. 首先，建立一個新的專案資料夾，例如命名為 `project/`。專案資料夾的結構會看起來像這樣：
    ```
    project/
    |- README.md
    |- .git
    |- .gitignore
    ```
2. 再次開啟 Tabletop Simulator，選擇任意遊戲並儲存。遊戲會自動建立包含遊戲所有設定與資訊的 JSON 儲存檔 (`TS_Save_*.json`)，並附上一張遊戲截圖 (`TS_Save_*.jpg`)。
    ```
    %UserProfile%\Documents\My Games\Tabletop Simulator\Saves\
    |- TS_Save_*.json
    |- TS_Save_*.jpg
    ```
3. 由於 Tabletop Simulator 並未提供選擇遊戲儲存檔位置的選項，我們可以先將儲存檔和截圖檔案移至專案資料夾，以便之後進行版本控管。
    ```
    project/
    |- README.md
    |- .git
    |- .gitignore
    |- TS_Save_*.json
    |- TS_Save_*.jpg
    ```
4. 接下來，在 Tabletop Simulator 的資料夾內建立一個指向專案資料夾的軟連結，以確保儲存檔可以在遊戲的載入遊戲清單中被找到。由於軟連結會連結到原始檔案，因此我們可以透過軟連結對原始檔案進行修改，並將變動同步到遊戲中。（Windows 系統在 cmd 當中使用 [mklink 建立軟連結](https://learn.microsoft.com/zh-tw/windows-server/administration/windows-commands/mklink)）
    ```
    %UserProfile%\Documents\My Games\Tabletop Simulator\Saves\
    |- project/ -> /path/to/project/
    ```
5. 再次開啟 VSCode 並啟動 Tabletop Simulator Lua 套件。此時 VSCode 的探索欄應該會出現 Tabletop Simulator Lua 資料夾以及由套件從遊戲儲存檔中提取出來的 Lua 腳本。
    ```
    Tabletop Simulator Lua/
    |- Global.-*.lua
    |- Global.-*.xml
    ```
6. 將專案資料夾加入 VSCode 的工作空間：選擇 `File` → `Add Folder to Workspace`，然後選擇專案資料夾。此時，探索欄的結構會看起來像這樣：
    ```
    Tabletop Simulator Lua/
    |- Global.-*.lua
    |- Global.-*.xml

    project/
    |- README.md
    |- .git
    |- .gitignore
    |- TS_Save_*.json
    |- TS_Save_*.jpg
    ```
7. 在專案資料夾中建立需要的檔案或資料夾，例如 `src/`（用來儲存遊戲腳本）。
    ```
    Tabletop Simulator Lua/
    |- Global.-*.lua
    |- Global.-*.xml

    project/
    |- README.md
    |- .git
    |- .gitignore
    |- TS_Save_*.json
    |- TS_Save_*.jpg
    |- src\
        |- script1.lua
        |- script2.lua
    ```
8. 接著開啟 `Global.-*.lua`，加入以下命令以指定要導入的腳本。由於套件會自動在工作空間的資料夾尋找副檔名為 `.ttslua`和`.lua` 的檔案，所以呼叫時就不需要加入副檔名了。
    ```lua
    require("script/script1")
    require("script/script2")
    ```

現在，已經成功設定 Tabletop Simulator 的 Lua 開發環境，也將遊戲依照專案資料夾的規劃納入版本控管，可以按照自己的習慣開發遊戲腳本了。

只是要記得在修改腳本或遊戲設定後，按下 `Ctrl` + `Alt` + `S` 以同步變動到遊戲儲存檔中，也不要忘記在新增腳本時更新導入的指令。

# 參考資料
- [TTS VSCode Docs](https://tts-vscode.rolandostar.com/)
- [Require and Path](https://github.com/rolandostar/tabletopsimulator-lua-vscode/issues/27)
- [Best way to commit the main json file?](https://github.com/rolandostar/tabletopsimulator-lua-vscode/issues/21)
- [Get/Send LUA scripts to/from workspace folder](https://github.com/rolandostar/tabletopsimulator-lua-vscode/issues/20)
