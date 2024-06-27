---
title: hexo 部屬後網頁報錯 "cannot get /"
date: 2024-01-02 20:47:12
tags: blog
categories: 
---

新的一年好不容易提振精神來寫文章，想說先把文章的調整更新到網站上。不料，輸入完 `hexo clean && hexo delopy` 之後，雖然 Github action 顯示部屬正常，但網頁卻無法瀏覽，分別顯示 "This is not the web page you are looking for" 以及 "There isn't a GitHub Pages site here"

![This is not the web page you are looking for.](https://i.stack.imgur.com/Hp57y.png)

- [404 There isn't a GitHub Pages site here. #4098](https://github.com/hexojs/hexo/issues/4098)
- [GitHub what causes gaps in issue #Number (404 This is Not...)](https://stackoverflow.com/questions/66569047/github-what-causes-gaps-in-issue-number-404-this-is-not)

**TL;DR: 如果你也碰到這情形，可能是 hexo 部屬工具不全或是，上網隨便找一篇教學，或參照以下指令把相關套件更新後再部屬看看。**
```cmd
npm install -g npm-upgrade
npm-upgrade
npm update
```

<!-- more -->

# 環境設置
忘了，只是想一寫文章啊，眼看元旦就要消失了，急著解問題沒記錄環境設置。不過這倒能衍伸出一項新年展望，下次碰到問題記得留意環境問題喔。

# 調查過程
使用 `hexo server` 檢查，發現網站根本起不來，只有一串的 `Cannot GET /` 掛在蒼白的網頁上。

通常，這項錯誤肇因於 [`source/_posts` 沒有任何文章](https://blog.csdn.net/qq_45593330/article/details/116702792)，可是查看專案資料夾卻發現，所有文章都好端端留在那啊，究竟部屬的時候發生了什麼事？

回頭翻部屬訊息才注意到，哇乾，hexo 不知何故把 `gh-page` 大多數檔案移除（慘況可參考 repository 上的 [commit 歷史](https://github.com/5uperb0y/5uperb0y.github.io/commit/3929872560d5bb2db64fd0a68eb441cac377e355)）。接下來我推測了各種可能性：

- token 過期等權限問題？但都能順利推上 github 了，先排除這項可能。
- 可能手賤刪除文章不自知？嘗試回到舊 commit 再部屬，問題依舊。
- 可能 `_config.yml` 的分支設錯？檢查設置檔，也確定 origin 分支了，問題依舊

到了最後才懷疑套件可能安裝不全或有問題？找了這篇 [Hexo 紀錄更新過程](https://feifacunzai.github.io/2020/06/29/Hexo-%E7%B4%80%E9%8C%84%E6%9B%B4%E6%96%B0%E9%81%8E%E7%A8%8B/)，照裡面講的步驟更新套件後再部屬後，網頁和文章都回來啦。

# 結論

人生難免碰到困難與障礙，有時並非個人不努力，而是大環境的問題。既然錯不在己，過度糾結也無濟於事，不如更新環境或給自己重新開機的機會，也許問題會迎刃而解呢。
