---
title: R/Rstudio console 的指令長度限制
date: 2022-03-03 22:38:31
tags: 
categories: [programming]
---

TL:DR：輸入 R/Rstudio console 的指令之容量上限為 4095 bytes。若超出上限，console 會忽略超出的部分，顯示 + 提醒使用者補完剩餘指令。

<!--more-->

今天測試定序前處理的 pipeline 時，發現總是無法順利讀檔。為了 debug，我把所有檔案的輸入路徑存為一個字串，並複製到 R console，重跑其中一個 function。結果 console 上印出一個 + 號，function 仍無法運作。
```r
> parse_path("....") # 這是長度約三千多個字符的字串
+
```
我第一個反應是檢查括弧的數量和位置是否正確，接著仔細看檔案路徑中有沒有跳脫字符。然而，怎麼檢查都沒有留意到語法錯誤，於是開始丟各種長度和內容的字串測試。


後來，我發現只要刪除部分字串，function 又可以順利執行了，所以我猜 R 的字串可能有長度上限？循著這猜想，我在網路上找到不少面臨相似問題的人，例如：

- [storing long strings (DNA sequence) in R](https://stackoverflow.com/questions/28399710/storing-long-strings-dna-sequence-in-r)
- [How to bypass RStudio console upper limit on character string length?](https://stackoverflow.com/questions/54974996/how-to-bypass-rstudio-console-upper-limit-on-character-string-length)

最終，我才在[這則問題](https://community.rstudio.com/t/does-console-impose-an-upper-limit-on-the-length-of-strings/12872)找到答案：輸入的字串被刪節確實是因為其大小超出限制，但並非超出 character 存儲上限，而是超出 R/Rstudio console 的上限。

舉例來說，我們可以用 paste0 和 rep 建立長字串。
```r
seq <- paste0(rep("ABCDEFG", 1000), collapse = "")
```

再使用 nchar 統計字串長度。
```r
> nchar(seq)
[1] 7000
```
然而，若把字串印出，再完整貼到 console 裏執行。由於字串大小超出 console 上限，所以 nchar 便無法順利運作了。

```r
> nchar( "ABCDEFGABCDEFGABCDEFG......" )
+ 
```

當然，字串也有其容量上限，這上限應該是取決於記憶體容量。這狀況，R 會直接告訴我們容量不足，而不是顯示讓人困惑的 +。
```r
> seq <- paste0( rep("ABCDEFG", 100000000000000), collapse = "")
Error: cannot allocate vector of size 745058.1 Gb
```
有趣的是，數字再大一點時數字會以科學記號表示，不符合 rep 的輸入規範，也會顯示錯誤。

```r
> seq <- paste0( rep("ABCDEFG", 1000000000000000000000), collapse = "")
Error in rep("ABCDEFG", 1e+21) : invalid 'times' argument
```
這些道理我都懂，可是，我的 pipeline 是以腳本執行啊，怎麼還會碰到這問題？經過一番苦思，發現 bug 出在我的程式邏輯上，跟本文提到的限制一點關係都沒有。

![meme, it's my fault, not R's](https://github.com/5uperb0y/blog-media/blob/main/r-console-upper-limit-on-the-string-length.png?raw=true)