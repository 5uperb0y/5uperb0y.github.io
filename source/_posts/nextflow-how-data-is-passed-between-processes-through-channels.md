---
title: Nextflow｜資料是怎麼在 Process 間傳遞？
date: 2023-01-05 20:58:15
tags: nextflow 
categories: [bioinformatics]
---

Process 是 nextflow 管理分析流程的基本單位，可包含能在 linux shell 執行的程式碼（例如 linux command, python code）、腳本（例如自訂的 hello-world.sh）與軟體（例如 GATK 或 FastQC）。 Process 之間彼此獨立，各有各的工作目錄，也可以分別設定其執行環境（例如 docker container 或 conda environment）。

Channel 則媒介了 process 間的資料交流，若不透過 channels 串接，process 間的檔案或變項無法共享。舉凡字串、數值、檔案路徑乃至標準輸出等，皆有對應的 qualifier 讓 nextflow 知道怎麼處理得自於 channel 的各式資料。
<!--more-->
在以下案例，我在 process A 定義了兩條字串，並透過不同途徑傳遞給 process B。

首先，string 1 被寫入檔案 (`str.txt`)，再以 path qualifier 告訴 Nextflow，把檔案路徑用軟連結 (symbolic link) 掛到 process B 的工作目錄。至於 string 2 則被定義為變項 (strAsEnv, a bash variable)，再以 env qualifier 告訴 nextflow 把變項寫到 process A 工作目錄內的 `.command.env`。

接著，分別以 strInFile 及 strAsEnv 命名由 process A 輸出 channel，傳遞給 process B 印出。
```groovy
// 提供 process B 所需的字串
process A {
    output:
        path "str.txt", emit: strInFile
        env strAsEnv, emit: strAsEnv
    """
        echo "string 1" > "str.txt"
        strAsEnv="string 2"
    """
}
// 印出 process A 提供的字串
process B {
    input:
        path strInFile
        val strAsEnv 
    output:
        stdout
    """
        echo "\$(cat ${strInFile}) is retrieved from a file passed by a queue channel"
        echo "${strAsEnv} is retrieved from a environment variable passed by a value channel"
    """ 
}
// 執行 process A & B
workflow {
    A()
    B(A.out.strInFile, A.out.strAsEnv).view()
}
```

```bash
$ nextflow run demoChannel.nf 
Launching `demoChannel.nf` [compassionate_shannon] - revision: 319cfe997a
executor >  local (2)
[f3/e5a365] process > A [100%] 1 of 1 ✔
[b0/f79f8e] process > B [100%] 1 of 1 ✔
string 1 is retrieved from a file passed by a queue channel
string 2 is retrieved from a environment variable passed by a value channel
```
各 process 的工作目錄預設建立在執行路徑下的`work`，直接觀察工作目錄內的暫存檔有助於了解 path 與 env 兩種資料傳遞方式的差異[^1]。
[^1]: 此處只列舉 process A 和 B 工作目錄內檔名或型別有別的檔案

在 process A 的工作目錄 ([f3/e5a365]) 裡，除了輸出的 str.txt，還有 `.command.env`。Nextflow 在執行時會將所有用 env qualifier 標記的變項定義式（即　`var=value` 這種格式）寫入 `.command.env`，以利隨後可輸出到其它 process。（可查看 `.command.sh` 查看寫入變項定義式的程式碼）

目前，只有 bash 變項才能透過 env qualifier 輸出，其它語言的變項只能透過某些方式存成檔案輸出（例如 R 的 RDS 檔）。
```bash
$ cd /work/f3/e5a365e3bb7b645cf4e13cf15eb235
$ ls -al
-rw-r--r-- 1 user group   18 Jan  5 22:26 .command.env      # environment variables of process A
-rw-r--r-- 1 user group  146 Jan  5 22:26 .command.sh       # commands to execute process A
-rw-r--r-- 1 user group    9 Jan  5 22:26 str.txt           # output of process A

$ cat .command.env
strAsEnv=string 2

$ cat .command.sh　 # nextflow 執行 process 的腳本，只擷取跟 `.command.env` 相關的部分
# capture process environment
set +u
echo strAsEnv=${strAsEnv[@]} > .command.env
```

而在 process B 的工作目錄可留意到 str.txt 的資料型別為 `l` (link)，檔案名稱也標示了來源。值得留意的是，如果使用 `cat` 或 `cp` 等指令通常不會有什麼問題，但如果用 `ls` 顯示檔案大小時，要注意預設顯示的是軟連結的大小，而非原檔的大小。相似的狀況也可能發生在使用 `find` 搜尋時，因為限制了搜索標的為檔案而意外地排除軟連結的情形。 
```bash
/work/b0/f79f8e550a4e1af11e2b03af8b4aa4$ ls -al
lrw-r--r-- 1 user group   62 Jan  5 22:26 str.txt -> /work/f3/e5a365e3bb7b645cf4e13cf15eb235/str.txt
```

綜上所述，要怎麼利用 channel 傳遞資料端視資料型別而異。無論想傳遞表格、文檔還是變項，都可以將其寫入檔案，再以 path qualifier 讓 nextflow 依據檔案路徑在下游 process 的工作目錄建立軟連結。若想傳遞的是 bash 變項，那還可以使用 env qualifier 將變項定義式記錄在當前 process 的工作目錄供後續取用。