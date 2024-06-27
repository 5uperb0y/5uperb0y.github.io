---
title: Vim｜嘗試修改 Vimium 來關閉網頁 pop-up
date: 2023-01-22 20:54:05
tags: vim
categories:
---
彈出視窗是以鍵盤瀏覽網頁的最大阻礙。這些視窗的內容不外乎會員邀請、推薦連結、蓋板廣告與網站公告等，是網站而定，通常沒辦法以 tab 來點擊，之前介紹的 Vimium 功能也沒辦法為這些視窗上的連結打標籤。由於彈出視窗並非廣告，所以下載 adblock 也無法阻擋之。所以往往只能乖乖移動滑鼠關閉之。

因為這些彈出視窗設計有別於html 內建的元素，所以有些更糟糕的案例是連關閉按鈕都沒有顯示在頁面上，強迫使用這點開網頁才能關閉之。

在得知 Vimium 點擊連結的方便之後，我嘗試修改 Vimium 的程式碼，讓他可以為原先沒辦法點擊的連結標註標籤

<!--more-->

# 想法與策略
Vimium 能為網頁中可點擊的元素標註標籤，讓用戶得以用這些標籤的快捷鍵點開連結，不用經手滑鼠或用tab 遍歷。其原理是透過html元素，辨識可能可點擊的元素，例如連結、按鈕等。

然而，網頁中仍能有自訂元素，這些元素的名稱和行為未必與 html 原生元素一致，所以 Vimium 只能嘗試搜索可能可點擊的元素名稱，例如 buttom 或是 link，間接推論這些元素可點擊。不過實際的網頁往往不會使用這麼直白的名稱，可能使用縮寫或是帶好來命名這些元素，導致 vimium 無法標註這些元素。

讀了[Vimium 介紹與支持 Plurk (噗浪)](https://ithelp.ithome.com.tw/articles/10091999)之後，覺得最直觀的想法是 修改 Vimium 程式碼的判斷式，看能不能找到彈出視窗共同的特徵，例如名稱、類別或是辨識碼等，拓展 Vimium 能辨識的元素，讓程式也能夠為彈出視窗標記快捷鍵。

不過這篇文章距今已十年，Vimium 也經歷多次改版，所以可能要找一下 github page 找到辨識網頁元素的區域在何處。所幸在[這篇文章](https://stackoverflow.com/questions/53918093/how-can-i-make-my-element-clickable-for-vimium)找到了一些參考。

我嘗試各大新聞網都失敗，只有知乎成功，因此以下會用知乎為例，說明要怎麼修改程式碼讓 Vimium 可以關閉彈出連結。
![Vimium 無法辨識知乎登入視窗的關閉符號](https://github.com/5uperb0y/blog-media/blob/main/modify-vimium-to-label-and-close-pop-up_before.png?raw=true)
# 調查彈出視窗是否有共同的特徵
下一步是觀察彈出視窗的網頁元素的構建方式，歸納出彈出視窗的共通特徵，以便讓 Vimium 辨識。首先，在瀏覽器按 `F12` 開啟開發人員視窗來檢視網頁原始碼，接著於彈出視窗依序點擊右鍵和檢查，即可鎖定對應位置的網頁原始碼。

我檢查了幾個網站後發現，每個網頁的彈出視窗設計都不一樣，例如元素的組成、對應的滑鼠行為(e.g. hover or click)，乃至於元素的 class 與 id 也因網頁而異。這意味著，也許沒有共通的值可以辨識這些自訂元素，只能針對常用的網頁來做修改。
![](https://github.com/5uperb0y/blog-media/blob/main/modify-vimium-to-label-and-close-pop-up_check.png?raw=true)
# 下載擴充套件程式碼
下載擴充套件的方式可參考[此文](https://stackoverflow.com/questions/16680682/how-to-modify-an-extension-from-the-chrome-web-store)
1. 安裝 [Chrome extension source viewer](https://chrome.google.com/webstore/detail/chrome-extension-source-v/jifpbeccnghkjeaalbbjmodiffmgedin)
2. 點開 Vimium 頁面
3. 點擊 "Chrome extension source viewer"
4. 點擊 "Download as zip"
![於擴充套件頁面點擊 "CRX" 即可下載套件原始碼（圖：Chrome extension source viewer 頁面）](https://lh3.googleusercontent.com/dCvSGp9T3LZP6PLfzgxNpxlX5qxizpjKN8g9R3eNKTg98e4MfAEDc5IRVFGr5O4VtC4nq62HNKpGxkNaDLscK1VXLw=w640-h400-e365-rj-sc0x00ffffff)

# 修改程式碼並套用於瀏覽器

解壓縮後，接著點開，在這段加入
https://github.com/philc/vimium/blob/v1.67.4/content_scripts/link_hints.js#L987-L992
```js
    if (!isClickable && className && className.toLowerCase().includes("zdi"))
      possibleFalsePositive = isClickable = true;
```

1. 開啟 Chrome，點選工具列的擴充功能圖示
2. 點選「管理擴充管理」
3. 點選「載入未封裝項目」
4. 重新整理頁面，查看是否順利修改

![修改 Vimium 的程式碼後，關閉視窗的符號被標上標籤了。](https://github.com/5uperb0y/blog-media/blob/main/modify-vimium-to-label-and-close-pop-up_after.png?raw=true)
# 困難與限制
實作之後才發現，許多網站不是用 Html 既有物件來構建彈出視窗，而是自訂許多 class 與 id，再透過 Javascript 控制。這些自定義物件的行為由 Javascript 控制，每個網頁的命名、行為與能見度都不一樣，所以 Vimium 也無法一次搞定所有可能性。

換句話說，本文陳述的嘗試只適用於常用的網站，好比說常看的新聞網或社群論壇等，不能一勞永逸地克服所有彈出視窗。
# 結論

目前，彈出視窗仍是鍵盤瀏覽網頁的最大阻礙，原因是彈出是往往是自訂元素。，諸如 Vimium 等擴充套件也不易辨識多樣的自訂元素。鍵盤的可及性受限於網頁元素的設計共識，所以政府對於無障礙網頁設計才有相關的規範與建議吧。
