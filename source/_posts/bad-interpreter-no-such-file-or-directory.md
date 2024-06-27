---
title: 'bad interpreter: No such file or directory'
date: 2020-02-19 16:56:10
tags: linux
categories:
---

在 Linux 執行 Perl、R 或 Python 腳本時，有幾種情況可能會跳出 "bad interpreter: No such file or directory"。

<!--more-->

# 打錯字

檢查 #!/usr/bin/env 有沒有拼對，如果拼錯的話也會報錯。

# 路徑不對

即「[執行Python "/bin/usr/python: bad interpreter: No such file or directory" 錯誤](https://www.cnblogs.com/xuxianren/p/4612440.html)」中提到的問題：當腳本開頭寫成 #!/usr/bin/python 時，如果沒有主動連結到安裝的 python 版本，有可能找不到。改寫成 #!/usr/bin/env python，就會自動尋找 python 的路徑了。

# 腳本的編碼格式不相容

Windows 記事本的格式和 Linux 的腳本不同，所以會因為隱藏的字元而無法判讀，解決方法可參考「[sh腳本異常：/bin/sh^M:bad interpreter: No such file or directory](https://www.cnblogs.com/xuxianren/p/4612440.html)」，在 vi 或 vim 編輯器下，以 :set fileformat=unix 修改腳本編碼。
