---
title: '移除不必要元件，打造精簡的 JupyterLab 使用者介面'
date: 2024-07-05 17:41:36
tags: 
categories:
---

JupyterLab 是數據分析領域廣泛使用的互動式編輯器，它整合了文件編輯、程式碼編譯、圖像呈現與檔案管理等功能。這個編輯器最大的特色是支援 Notebook 這種多媒體文件格式，能夠將程式碼、說明文字與輸出結果同時呈現在同一份文件中。

這種設計帶來兩個主要優勢：首先，簡化了數據分析過程中的參數調試流程。開發人員可以在同一頁面上檢視結果並即時更新參數。其次，它促進了資訊與分析結果的交流。用戶可以通過文件內的描述了解分析背景與相關知識，從程式碼中掌握具體執行方式，同時能夠即時查看結果和相應說明。

JupyterLab 的另一個優勢在於它能夠建立伺服器，讓用戶通過瀏覽器進行操作，提供了一個現成的使用者介面。這一特性使得開發者無需額外花費精力在 UI 設計上，因此除了作為開發編輯器和資訊傳遞媒介外，JupyterLab 也被數據分析團隊或各類平台用作分析服務的工具。開發者可以在雲端伺服器上架設基於 JupyterLab/JupyterHub 的服務，使非技術背景的用戶也能夠連線使用這些分析工具。

我在本文要探討的，是如何在提供分析服務的情景下，透過調整系統設置來建立更精簡的 JupyterLab 介面，從而改善用戶的使用體驗。

<!--more-->

# JupyterLab 預設的使用者介面

JupyterLab 預設的使用者介面包含多個功能區塊：

- 頂部的選單列（main menu）陳列了檔案存取、程式碼編輯、分頁管理、外觀設置與介面調整等選單。
- 左側的功能欄（side bar）顯示當前使用的擴充套件。預設的擴充套件為檔案瀏覽器（File browser），它讓用戶可以在這裡管理工作目錄中的資料夾和檔案。
- 中央則是瀏覽和編輯檔案的主要工作區，它允許用戶使用分頁管理不同工作視窗，例如 python/R 互動視窗、Linux 終端或是任何一種文件。工作區的文件管理器（Document manager）能把文件渲染為 notebook、html、markdown 或是純文字等方便瀏覽與操作的格式。

![](https://raw.githubusercontent.com/5uperb0y/blog-media/main/remove-unnecessary-elements-to-make-the-jupyterlab-ui-cleaner/interface-elements.png)

除了主要功能區塊以外，JupyterLab 的使用者介面還有其他資訊面板。左右兩側的活動欄（activity bar）會列出了已安裝的擴充套件。以 JupyterLab 4.2.3 版為例，內建的套件包含文件目錄（table of contents, 列出文件的章節標題）、終端與核心管理器（running terminals & kernels，查看當前正在執行的核心與終端機）、除錯器（debugger，支援單步執行與變項追蹤等除錯功能）與屬性檢查器（property inspector，可以查看程式碼區塊的屬性）。

而位於底部的狀態列（status bar）則顯示了程式執行狀況、標準輸出訊息、游標位置與當前核心等資訊。如果不習慣分頁管理，也可以在此切換為簡易介面（simple interface），讓系統一次只呈現單一文件。

JupyterLab 整合了這些豐富的功能元件，讓開發者能集中管理與調用所需的工具；它也提供多元的自訂選項，容許開發者依據個人喜好調整介面風格。然而，這些彈性可能對非技術用戶而言太過複雜。因此，為了打造更適合非技術用戶使用的分析服務，有必要調整 JupyterLab 的預設介面。

# 理想的分析介面與其實踐方案

JupyterLab 的預設介面有數個引導不佳的地方。首先，如果啟動 JupyterLab 伺服器時沒有指定要開啟的文件，那麼系統預設會跳出啟動頁（Launcher），用戶需要自行挑選要打開的視窗。其次，系統預設用 notebook 打開 .jpynb 檔案。notebook 模式能讓程式碼與輸出結果並陳，有助於開發者在調整參數時即時得到回饋。不過，這項設計要求用戶具備調整程式碼與執行程式的能力。最後，JupyterLab 內建許多開發相關套件，這些功能與使用分析無關，反而可能混淆用戶，降低應用軟體分析的體驗。

考量到這些可能導致混淆的狀況，我們可以從以下幾個方面著手以改善使用者介面。

- 自動載入：啟動服務時，自動開啟對應的 notebook。
- 圖形介面：提供圖形操作介面，避免用戶直接接觸程式碼。
- 精簡功能：移除與用戶無關的按鈕、選單與資訊，只保留分析服務本身。

以下是調整後的範例，它只保留系統內建的檔案管理器，以及位於主要工作區的分析服務介面。

![](https://raw.githubusercontent.com/5uperb0y/blog-media/main/remove-unnecessary-elements-to-make-the-jupyterlab-ui-cleaner/modified-jupyterlab-user-interface.png)


# 調整使用者介面的途徑

為了達到精簡使用者介面的目的，不只要調整 JupyterLab 的系統設置，也需要改動程式碼的觸發方式。

## --LabApp.default_url=<url>：指定開啟的頁面

啟動 JupyterLab 時，只要添加 `--LabApp.default_url=<url>` 就能指定要開啟的文件和瀏覽模式，假設工作目錄底下有一個 `row_counter.ipynb` 的話，那麼不同的 URL 會以不同方式打開文件。

- 以 notebook 模式打開文件：`jupyter lab --LabApp.default_url=/notebook/tree/row_counter.ipynb`
- 以 lab 模式打開文件：`juptyer lab --LabApp.default_url=/lab/tree/row_counter.ipynb`
- 以 simple 模式打開文件：`jupyter lab --LabApp.default_url=/doc/tree/row_counter.ipytnb`

## ipywidgets：建立互動元件與圖形介面

[IPyWidgets](https://ipywidgets.readthedocs.io/en/stable/) 允許開發者在 Jupyter notebook 建立按鈕、拉桿、文字框等可互動元件。這些元件能夠控制 Python 程式碼的行為，讓用戶能透過圖形介面執行分析，避免直接接觸程式碼與底層運作。IPyWidgets 的效果如下圖所示，執行程式之後，互動元件會呈現在程式碼區塊底下的輸出區塊。

![](https://i.sstatic.net/YHGEd.png)
（圖片來源：https://stackoverflow.com/questions/37013489/how-to-align-and-place-ipywidgets）

## Voila：呈現渲染過的使用者介面

[Voila](https://voila.readthedocs.io/en/stable/) 可以把 notebook 轉換獨立的網頁應用程式。如同官方網站的範例，Voila 會隱藏 JupyterLab 原生介面和程式碼區塊，只呈現 notebook 輸出內容，從而建立精簡的圖形介面。

![](https://voila.readthedocs.io/en/stable/_images/voila-basics.gif)
（圖片來源：https://voila.readthedocs.io/en/stable/）

## JupyterLab：調整介面與套件設置

假如你的分析服務不仰賴 JupyterLab 內建的互動元件，那麼只要使用 Voila 就能建立一個精簡的服務介面了。但是，如果想要保留一些好用預設工具（例如檔案瀏覽器與內容目錄），又不希望額外的元件影響用戶體驗，就需要調整 JupyterLab 的參數設置。

JupyterLab 的每個區塊都能在頂部的 Settings 選單調整。依序點選 "Settings"、"Settings Editor"、"Plugin Manger" or "JSON Settings Editor" 即可打開介面編輯器。不過，如果想管理這些異動，並且讓自訂的參數長期生效，就需要透過修改設置檔來調整介面。

### 關閉不必要的擴充套件

我們可以編輯 `~/.jupyter/labconfig/page_config.json` 來停用不必要的擴充套件，並且移除它們在活動欄的圖示。（如果家目錄內沒有這個目錄與檔案，那就自行建立一組。）
```json
{
    // 停用除錯器、目錄、屬性檢查器、終端與核心檢查器
    "disabledExtensions":
    {
        "@jupyterlab/debugger": true,
        "@jupyterlab/debugger-extension": true,
        "@jupyterlab/toc-extension": true,
        "@jupyterlab/application-extension:property-inspector": true,
        "@jupyterlab/running-extension": true
    }
}
```

### 移除不需要的功能元件

至於 JupyterLab 的功能元件則可以編輯 `<sys.prefix>/share/jupyter/lab/settings/overrides.json` 來調整（如前所述，如果沒有這個目錄與檔案，就自行建立）。因為每部主機的系統路徑都不一樣，所以要以 `jupyter lab path` 這個指令顯示的 Application directory 為準。關於設置檔的路徑說明與調整，可以參考[官方文件](https://jupyterlab.readthedocs.io/en/stable/user/directories.html)。

```json
{
    // 啟用 Simple mode 介面
    "@jupyterlab/application-extension:shell":
    {
        "startMode":"single"
    },
    // 以 voila 開啟 notebook
    "@jupyterlab/docmanager-extension:plugin":
    {
        "defaultViewers":
        {
            "notebook": "Voila Preview"
        }
    },
    // 關閉位於畫面底部的 status bar
    "@jupyterlab/statusbar-extension:plugin":
    {
        "visible": false
    },
    // 關閉 extension manager，隱藏它在側欄的圖示
    "@jupyterlab/extensionmanager-extension:plugin":
    {
        "enabled": false
    },
    // 隱藏 new launcher 按鈕，以免用戶誤觸
    "@jupyterlab/filebrowser-extension:widget":
    {
        "toolbar":
        [
            {
                "name": "new-launcher",
                "disabled": true
            }
        ]
    },
    // 移除 main menu
    "@jupyterlab/mainmenu-extension:plugin":
    {
        "menus":
        [
            {
                "id": "jp-mainmenu-file",
                "disabled": true
            },
            {
                "id": "jp-mainmenu-run",
                "disabled": true
            },
            {
                "id": "jp-mainmenu-kernel",
                "disabled": true
            },
            {
                "id": "jp-mainmenu-edit",
                "disabled": true
            },
            {
                "id": "jp-mainmenu-help",
                "disabled": true
            },
            {
                "id": "jp-mainmenu-tabs",
                "disabled": true
            },
            {
                "id": "jp-mainmenu-view",
                "disabled": true
            },
            {
                "id": "jp-mainmenu-settings",
                "disabled": true
            }
        ]
    }
}
```

# 結論

JupyterLab 是功能豐富的互動式編輯器，不僅適用於開發者，也能提供分析介面供非技術用戶使用。然而，它的預設介面含有許多可能造成混淆的功能元件。為了改善 JupyterLab 分析服務介面的引導性與清晰度，可以從調整以下四個項目：

- 啟動伺服器時，添加 `--LabApp.default_url=<url>` 選項，自動載入目標頁面。
- 以 IpyWidgets 設計互動式元件，再用 Voila 將 notebook 轉換為網頁應用程式。
- 編輯 `override.json` 移除不必要的功能元件，例如按鈕、選單與資訊欄。
- 編輯 `page_config.json` 停用開發者擴充套件。

透過這些方法，便能精簡 JupyterLab 的介面，降低分析服務的使用門檻，提升用戶體驗。

