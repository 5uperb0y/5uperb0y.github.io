---
title: Nextflow｜怎麼從運行中的腳本取回參數？
date: 2022-12-06 21:45:37
tags: nextflow
categories: bioinformatics
---

在執行腳本前，Nextflow 會讀取設置檔 (configuration files) 中的參數，將之代入腳本的對應位置後再執行程式。這項特性有助於使用者管理複雜流程的輸入值與環境設定，也將具體數值從流程邏輯抽離，讓開發者專注於流程的梳理與串接。

然而，隨著流程腳本改版，設置檔的內容也可能跟著改變，是否有方法能記錄執行流程時使用的設置檔，以利往後重現分析或追蹤歷次設定？

在這篇文中，我首先介紹了 nextflow 導入參數的方式，再陳述取回參數的可行策略，並附上這些策略的最簡範例供參考。

<!--more-->

## Nextflow 導入參數的方式

以下是 nextflow 導入參數的常見方式，若一參數被多種方式定義，則會以順位較小者為準。

| 順位 | 方式                   | 範例                                                |
| ---- | ---------------------- | --------------------------------------------------- |
| 1    | 於命令列輸入           | `nextflow run workflow.nf --something value`        |
| 2    | 以 `-params-file` 導入 | `nextflow run workflow.nf -params-file params.json` |
| 3    | 以 `-c` 導入           | `nextflow run workflow.nf -c params.config`       |
| 4    | 於 nextflow 腳本內宣告 | `params.something = value`                          |

## 使用 `params` 取回參數
其中一個方法是讀取 `params` 的內容，`params`是 nextflow 的[隱變項](https://www.nextflow.io/docs/latest/script.html?highlight=implicit#implicit-variables)，採用 `[key1:value1, key2:value2,...]`格式存儲以不同方式導入的參數。

### params.json 
```json
{
    "paramsDerivedConfig" : "provided using the -params-file option"
}
```

### params.config
```bash
params {
    configDerivedConfig = "specified using the -c option"
}
```
### getConfig.nf
```java
params.inScriptConfig = "defined within the script itself"
process retrieveConfigFromParams {
    output:
        stdout
    """
    echo "$params"
    """
}
workflow {
    getConfig().view()
}
```

```bash
$ nextflow run getConfig.nf --commandSpecifiedConfig "specified on the command line" -params-file params.json -c params.config
[configDerivedConfig:specified using the -c option, config-derived-config:specified using the -c option, paramsDerivedConfig:provided using the -params-file option, params-derived-config:provided using the -params-file option, commandSpecifiedConfig:specified on the command line, command-specified-config:specified on the command line, inScriptConfig:defined within the script itself, in-script-config:defined within the script itself]
```

## 從輸入路徑取回參數

然而，前述方法只能取回以 `params` 存儲的參數，若想取得 `process`, `docker`, `report` 等[其它設置 (configuration scope)](https://www.nextflow.io/docs/latest/config.html?highlight=params#config-scopes)，可以讀取位於 `$launchDir` 的 `.nextflow.log`，從中獲得當初執行腳本時輸入的 `*.config` 路徑，再將設置擋複製出來。

```java
process getConfig {
    output:
        stdout
    shell:
    '''
    str=$(grep "User config file:" "!{launchDir}/.nextflow.log")
    config=${str##*:}
    echo ${config}
    '''
}
workflow {
    getConfig().view()
}
```

```
$ nextflow run getConfig.nf -c params.config
/home/user/workflow/params.config
```

## 結論

若能從運行中的 nextflow 腳本取回輸入參數，將有助於往後重現分析或追蹤設置。目前，nextflow 有兩個隱變項能協助我們達成這項任務，
- `params`：記錄了不同管道導入的參數，可以在 `*.nf` 檔各處呼叫以取得書入的參數和設定。
- `launchDir`：雖然本身與參數無關，但此變項記錄了腳本執行位置，其中的 `.nextflow.log` 記錄了 `*.config` 路徑。讀取該檔即可獲取輸入的參數和設定。