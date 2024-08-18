---
title: 'VSCode: Failed to parse remote port from server output'
date: 2023-01-07 14:20:08
tags: vscode 
categories:
---
## 狀況描述
原本在 Windows 都能正常用 VSCode Remote SSH 連線伺服器，但一早卻發現連線失敗。當時，VSCode 不斷要求輸入密碼卻無法登入，"OUTPUT" 頁面提示的訊息節錄如下：

> Failed to parse remote port from server output

> Acquiring lock on /home/username/.vscode-server/bin/some-hash-code-here/vscode-remote-lock.username.some-hash-code-here

*版本資訊：windows 21H2, VSCode 1.74.2*
<!--more-->
## 原因與解決辦法
參考 [VSCode remote ssh connection not working](https://stackoverflow.com/questions/64034813/vs-code-remote-ssh-connection-not-working)，簡言之，VSCode 遠端連線前，需要在伺服器端的 `.vscode-server/bin` 安裝連線相關的工具並進行一些連線設置。假設伺服器已有這些套件，"OUTPUT" 會提示 `Found existing installation at (.vscode-server/bin 的路徑)`；若沒有這些套件，則會從官方網站下載與安裝。

若伺服器的套件和配置沒有隨本地 Vscode 更新，便可能因為連線兩端的資訊不相符而無法順利連線，此時可刪除並重新安裝這些工具與配置檔來解決問題。
1. `Ctrl` + `Shift` + `P`，開啟指令視窗
2. 搜尋並執行 "Remote-SSH: Kill VS Code Server on Host"，刪除 `vscode-server` 檔案與目錄
3. 選取欲清除既有工具的伺服器
4. 重新連線，VSCode 便會重新安裝相關軟體了

## 檢查方式

1. 開啟 command line 工具，以 `ping` 確認連線正常
2. 使用 putty 或 mobaXtern 等工具連線伺服器，確認問題出在 VSCode 
3. 重新開機、重新安裝 Remote SSH、移除 `C:/Users/username/.ssh/Known_host` 的內容，確認問題不在本地端
4. 按照前述步驟刪除伺服器內 `vscode-server` 的內容，重新連線
   
## 其他可能解法
若此處提及的方式無法解決問題，可參考以下連結試試看其他人的解法。
- [首次遠端連線且因為網路問題下載工具受阻](https://mushding.space/2021/12/22/vscode%20remote-ssh%20%E5%95%8F%E9%A1%8C%E8%B8%A9%E5%9D%91%E5%BF%83%E5%BE%97/) 
- [本地與伺服器的 SSH key 因為更新等緣故沒有同步](https://www.cnblogs.com/netsa/p/14857577.html)
- [任何可能與遠端連線配置或安裝的問題](https://stackoverflow.com/questions/64034813/vs-code-remote-ssh-connection-not-working)
- [伺服器上的 `vscode-server` 文件過時](https://blog.csdn.net/myWorld001/article/details/119443079)

## 結語

IDE 及其擴充套件相當方面，可以免去不少開發環境設置的功夫。畢竟我們不會等到摸熟了 IDE 所有功能及其實踐方式才開始開發，所以功能故障算是讓了解 IDE 機制或潛在設定的好機會。在本文的案例哩，可以關閉自動更新來避免這種狀況發生。

1. `Ctrl` + `,`，開啟 Settings
2. 搜尋 "update: Mode"
3. 設定為 "manual" 或 "none"
4. 重新啟動 VScode