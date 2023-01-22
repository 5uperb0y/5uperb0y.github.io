---
title: VScode｜無助於開發，但有助於身心的套件
date: 2023-01-20 16:43:08
tags: [vscode] 
categories: [programming, tools]
---
最近調查 VSCode 有哪些能幫助軟體開發的工具時，發現不少有趣的擴充套件，這些套件也許不能提升開發速度，但說不定能為百無聊賴、獨自 coding 的下午增添風格與生氣。

<!--more-->

# 像駭客一樣寫程式
首先是能增加打字回饋感並且裝模作樣的套件。
## Power Mode
> This is the most necessary and MUST TO plugin EVER. Now I can feel the power of my typing. Recommended to all my friends and colleges. 1000000/10

![powermode](https://raw.githubusercontent.com/hoovercj/vscode-power-mode/7bbc4f68dd46da883b24011ae67516c861d09d1b/images/demo-presets-particles.gif)

Power Mode 能大幅增加 coding 的回饋感，除了隨著打字噴發的特效與劇烈震動以外，在頁面右上角也會顯示連續打字次數，讓打字除了聽覺與觸覺的回饋感以外，還增加了視覺的激情與火爆。

Power Mode 的設定健全，無論特效的形式、時間、幅度、頻率或觸發條件都可調整。以震動特效為例，若覺得太過干擾，可以透過以下方式關閉，
1. `Ctrl` + `,`，開啟設置
2. 搜尋 "Powermode > Shake: Enabled"
3. 關閉震動特效

## VSCode HackerTyper
>  Writing Code Like a Real Hacker 

[HackerTyper](https://hackertyper.net/) 是模擬駭客寫程式的網頁。一點開網頁，映入眼簾的即典型的駭客工作環境：黑底綠字的終端畫面、等寬的程式碼字體、閃爍的文字游標等。

在這畫面中，無論敲打哪個按鍵，網頁都會印出虛構的迴圈、變數、判斷，營造出用戶正在快速寫程式的模樣。

![VSCode HackerTyper 讓你像個駭客一樣 coding](https://github.com/jevakallio/vscode-hacker-typer/raw/master/docs/hackertyper-video.gif)

相較之下，VSCode HackerTyper 可以印出「真正」的程式碼。這款套件能錄製文件編輯行為，再透過用戶每次按鍵撥放來達到網頁版的效果。由於是採用自己的程式碼，所以這項套件也挺適合在現場示範時博其它人一笑的（詳見開發者親自示範如何[像個駭客一樣現場 demo](https://www.youtube.com/watch?v=ulnC-SDBDKE)。

1. `Ctrl` + `Shift` + `P`，開啟 VSCode 命令列。（接下來指令皆於命令列執行)
2. 輸入 "HackerTyper: Record Macro"，接著開始編輯文件
3. 編輯完文件後，輸入 "HackerTyper: Save Macro" 紀錄並命名
4. 要展示駭客技能時，開啟文件並輸入 "HackerTyper: Play Macro" 

*已知 VSCodeVim 與此套件衝突，所以要使用這工具前，得先停用 VSCodeVim 並且重啟視窗*

## Cursor smooth setting
> Watching those lines of codes flow like butter along with your cursor is just another satisfying thing to watch for any programmer like the satisfying sounds of their mechanical keyboards. 😌 (Trishiraj)

除了使用套件，VSCode 也有許多內建設置能調整編輯的視覺效果，其中我覺得最有效提升視覺舒適感的是一系列平滑設定。

好比說啟用 "Cursor Smooth Caret Animation" 之後，VSCode 會在移動過程添加殘影動畫，讓游標在字元與行列間跳動時的移動軌跡更接近肉眼習慣的模式，看起來更為滑順與流暢。

1. `Ctrl` + `,`
2. 搜尋 "smooth"
3. 勾選 "Cursor smooth Caret Animation"


## hacker theme
有了 Power Mode、HackerTyper、smooth cursor，再加上 VScode 商店許多駭客佈景主題，就能打造一座自嗨小天地，或是秀一回盛大而且浮誇的表演囉。
![pro hacker theme](https://github.com/thorerik/vscode-hacker-theme/raw/HEAD/media/Code_2020-08-12_01-55-19.png)

# 寫程式時不再孤獨 
接著讓 coding 在鍵盤敲擊聲、鍵盤的觸感與閃爍的游標之外，還增添人造互動的套件。

## vscode-pet
> Coding can be lonely sometimes but these pets always keep me company !! (Aya Lahrech)

vscode-pets 會在 VSCode 側欄新增寵物窗格，可以在其中添加貓咪、小狗與小雞等寵物。除了看著寵物閒晃以外，也能透過點擊或擲球與他們互動。

![VSCode 小動物會追著丟出的球跑，也會以表情符號回應點擊。](https://tonybaloney.github.io/vscode-pets/_images/throw-ball.gif)

若覺得預設的背景太單調，可以在設定更換背景
1. `Ctrl` + `,`，開啟設定
2. 搜尋 "vscode-pets.theme"，設定想要的主題

![除了森林，還有城堡與海灘背景](https://tonybaloney.github.io/vscode-pets/_images/forest.gif)

## Encourage
> There are times when writing code is drudgery. In those dark times, bathed in the soft glow of your monitor, engrossed in the rhythmic ticky tacka sound of of your keyboard, a few kind words can make a big difference. And who better to give you those kind words than your partner in crime - your editor. (Haacked)

![Encourage 會在文件儲存時，根據程式碼正確與否給予對應的回饋。](https://user-images.githubusercontent.com/7860985/79793932-66320380-831f-11ea-8188-fb4a627f670a.gif)

Encourage 是款文字互動套件[^1]，會根據文件儲存當下，程式碼是否語法錯誤給予對應的回饋。假如沒有錯誤，則會隨機印出鼓勵的話；反之，則會吐出吐槽或挖苦的句子。

[^1]: 此處介紹的是 Nicollas R. 承襲 Haacked 所開發的 Encourage 套件，後者的版本目前在 VSCode 1.74.3 的 MarketPlace 已經找不到了。 

這套件支援自訂語錄，所以可以自行添加喜歡的座右銘、名言或幹話等，
1. `Ctrl` + `Shift` + `P`，開啟設定介面 
2. 搜尋 "Preferences: Open User settings (JSON)"
3. 在大括號內添加以下內容，即可新增自訂語錄：
    ```json
        // [extensions]: Encourages
        "encourage.additionalEncouragements": [
            "Wake the fuck up, Samurai. We have a city to burn 🔥🔥🔥",
            "Just focus, you are better than them.",
            "Fuck yeah!!!",
            "Death to Corpos!!!💀",
            "Never stop fighting.",
            "Holy FUCK!!!"
        ],
    ```
4. 若只想聽到鼓勵的話，可以用空字串覆蓋預設的挖苦內容。
   ```json
    "encourage.discouragements": [""],
   ```

此處我加入了一些 Cyberpunk 2077 的台詞，但你也可以調整成從飲料封膜看到的冷笑話或農民曆頁緣的生活訣竅。

## Hotheaded VS Code
> love how it kic..., corrects me. (Venipa)

相較於 Encourage 的文字訊息，Hotheaded VS Code 會在語法出錯時放出嫌棄的語音（而且只有嫌棄，沒有鼓勵）。雖然我個人是比較喜歡鼓勵大於責備啦，但這在業界是種獎勵也說不定。

*聲線可以參考配音員[dalfaria 的網站試聽](https://www.fiverr.com/dalfaria)*

# 讓人哭笑不得
最後則是能讓自己的 IDE 與他人很不一樣的主題。此處推薦兩則 VSCode 官方推特也說讚的主題套件[^2]。

[^2]:https://twitter.com/code/status/928315998075215873

## Cage Icons
> With this I'm gonna steal the Declaration of Independence. (Kishore Newton)
 
我覺得尼可拉斯凱吉是詮釋「身不由己」最出色的演員，他的表情也是迷因的重要來源。這套件會將原生圖標替換那些常在梗圖裡看到的尼可拉斯凱吉表情。
![用膩了 Materials Icon Theme？何不試試 Cage Icons！](https://raw.githubusercontent.com/GabeStep/cage-icons/master/example.png)

## Hot dog stand

> This is AMAZING! I can hardly see anymore, recently started using a walking stick. couldn't recommend more! (Franklin Volcic)

Hot dog stand 套件復刻了 Win 3.1 史上最糟糕的視覺設計[^3]。套件開發者介紹說，這套主題能在 15 秒內毀了用戶雙眼，但根據 VSCode 推特小編的說法，其實大約只需兩秒鐘......

![安裝這款主題後，我只堅持了一秒鐘就移除了。](https://raw.githubusercontent.com/SomeKittens/VSC-HDS/master/theme.png)

[^3]: https://blog.codinghorror.com/a-tribute-to-the-windows-31-hot-dog-stand-color-scheme/

# 結論

有時，認真研究點沒意義的事，其實也挺好玩的啊！
