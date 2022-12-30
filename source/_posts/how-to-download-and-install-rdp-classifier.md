---
title: 如何下載和安裝 RDP classifier
date: 2022-03-07 22:50:09
tags: database
categories: [bioinformatics, metagenomics]
---

RDP classifier 是基於 Naive Bayes 的物種分類器，常用於註解 16S rRNA 基因序列的分類資訊。除了使用內建的 RDP 資料庫以外，RDP classifier 也允許使用自訂資料庫來訓練分類器。除了內建的 RDP database，也支援以其他資料庫來訓練分類器。

目前，RDP classifier 的核心演算法已被整合到 Mothur 和 DADA2 等流程的副程式庫，所以只要有安裝這些流程軟體，即使沒有安裝 RDP classifier，也能以相同的演算法註解物種資訊。然而，若想要依據自訂或最新的資料庫註解序列，或是使用 copy number 校正等功能，仍有獨立使用 RDP classifier 的必要。

本文將介紹三種下載和安裝 RDP classifier 的方式。

<!--more-->

## Conda
透過 conda 安裝的方式可參考 [Anaconda 的教學](https://anaconda.org/bioconda/rdp_classifier)。目前，conda 與 RDP 官網提供的 classifier 皆為最新的 2.13 版。

```bash
conda install -c bioconda rdp_classifier
conda install -c bioconda/label/cf201901 rdp_classifier
```

## 官方網站
若不想要裝一堆附加的軟體或剛好 conda 沒有需要的版本，也可以直接到官網提供的[連結](https://sourceforge.net/projects/rdp-classifier/)下載可執行檔。

```bash
wget https://jaist.dl.sourceforge.net/project/rdp-classifier/rdp-class
ifier/rdp_classifier_2.13.zip
unzip rdp_classifier_2.13.zip
chmod u+x path/to/rdp_classifier_2.13/dist/classifier.jar
```

由於 RDP classifier 是依賴 JAVA 的軟體，所以執行時要輸入 .jar 的絕對路徑。

```bash
java -jar /path/to/rdp_classifier_2.13/dist/classifier.jar <command>  
<parameters>
```
為了簡化指令，可以創造名為 rdp_classifier 的腳本，並將腳本所在的目錄加入環境變數。

```
touch rdp_classifier
chmod u+x rdp_classifier
vim rdp_classifier
```
接著把落落長的指令放到腳本中。
```
#!/usr/bin/env bash
java -jar /path/to/rdp_classifier_2.13/dist/classifier.jar $*
```
如此一來，往後執行時就只需要輸入 rdp_classifier即可。

```bash
rdp_classifier <command> <parameters>
```
## Ubuntu
若是 Ubuntu，則可以從套件庫安裝。只是相較於 conda，ubuntu 套件庫內的 RDP classifier 還[停在 2.10 版](https://packages.ubuntu.com/impish/rdp-classifier)。
```bash
sudo apt-get -y install rdp-classifier
```
除了前述方法，也能夠安裝集合了 RDP 團隊包含了 classifier 在內各種開發工具的 [RDPTools](https://github.com/rdpstaff/RDPTools)。只是 RDPTools 是透過 make 和 makefile 來編譯與安裝。由於依賴的軟體和環境設定等問題，我到現在還沒成功過，暫時無法整理出相關的筆記。