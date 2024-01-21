---
title: 常見的生物資訊軟體開發文件
date: 2024-01-21 20:52:59
tags: 
categories:
---

撰寫軟體的理想目標是讓程式碼自我解釋，使得任何受過訓練的工程師能夠僅靠程式碼本身理解其含意。然而，隨著軟體功能與部件日益複雜，我們仍須依賴額外的文件來說明程式碼的商業邏輯、運作情境與潛在限制等背景知識。

在生物資訊軟體開發領域，尤其是醫療軟體開發中，文件的詳盡程度與正確性尤為重要。為了把關產品品質與和保障用戶權益，醫療產品驗證要求產品送審時提供完整的軟體設計、需求分析和產品驗證報告。這些文件的種類繁多且規範細緻，撰寫和維護這些文件往往成為開發者的重擔。

減輕這種負擔的一種方法是讓開發人專注於與開發相關的文件，並將這些文件作為合規文件的來源。透過簡化文件類別和統一內容來源，來簡化文件撰寫和維護的流程。

因此，本文整理了常見的生物資訊軟體開發文件，闡述與開發密切相關的文件類型，並列舉優良的撰寫指南，同時對各類文件的定位差異提出個人見解。

<!-- more -->

# 常見的生物資訊軟體開發文件清單
雖然文件內容會依專案性質與規模而有所不同，但在開發過程中通常會隨專案進展逐步納入以下幾種文件，


| 名稱                   | 內容                                            | 額外受眾 | 指引                                                                                 |
| ---------------------- | ----------------------------------------------- | -------- | ------------------------------------------------------------------------------------ |
| Issues                 | 紀錄 bugfix 或 feature request 等開發需求與問題 | 專案經理 | [Writing Good Feature Requests] & [Writing Good Bug Reports]                         |
| Docstring & Type hints | 描述程式模組的功能、參數與回傳資訊              | 開發者   | [PEP 257] & [PEP 484]                                                                |
| Code comments          | 程式碼的補充說明，包括任務標記和權益作法提示    | 開發者   | [Best practices for writing code comments]                                           |
| Built-in help          | 軟體內建的用戶幫助訊息                          | 用戶     | [docopt]                                                                             |
| Commit messages        | 摘要程式碼異動的內容、理由與背景                | 開發者   | [Conventional commits]                                                               |
| Pull requests          | 簡述為滿足需求對應的軟體異動及其理據            | 審查者   | [Writing A Great Pull Request Description] & [How to write the perfect pull request] |
| README                 | 介紹專案目標、安裝使用與授權等入門資訊          | 用戶     | [Make a README]                                                                      |
| CHANGELOG              | 紀錄專案開發歷程與各版本更新內容                | 用戶     | [Keep a CHANGELOG]                                                                   |

[Writing Good Feature Requests]: https://github.com/rstudio/rstudio/wiki/Writing-Good-Feature-Requests
[Writing Good Bug Reports]: https://github.com/rstudio/rstudio/wiki/Writing-Good-Bug-Reports
[PEP 257]: https://peps.python.org/pep-0257/
[PEP 484]: https://peps.python.org/pep-0484/
[Best practices for writing code comments]: https://stackoverflow.blog/2021/12/23/best-practices-for-writing-code-comments/
[docopt]: http://docopt.org/
[Conventional commits]: https://www.conventionalcommits.org/en/v1.0.0/
[Writing A Great Pull Request Description]: https://www.pullrequest.com/blog/writing-a-great-pull-request-description/
[How to write the perfect pull request]: https://github.blog/2015-01-21-how-to-write-the-perfect-pull-request/
[Make a README]: https://www.makeareadme.com/
[Keep a CHANGELOG]: https://keepachangelog.com/en/1.1.0/

除了專案進展，開發文件的內容和格式也會根據團隊或任務需求而異。例如，單一腳本構成的小型專案在初期可能僅需程式碼註解和內建幫助訊息；隨著專案成熟，可以逐漸增加 README 文件來補充安裝和授權資訊，並在軟體定版後定期更新CHANGELOG。而個人開發的專案可能沒有 Pull Request 資訊；但團隊開發除了需要 pull request 把關軟體品質外，還需要 issues 管理專案排程。

表中的連結已提供詳細的文件撰寫指引，以下我將補充一些對個別文件定位的看法。

# README 和 CHANGELOG 的差異
README 呈現專案的現況，包括專案簡介、安裝和使用方法、軟體授權等必要資訊，是用戶或開發者了解專案的首要途徑。除此之外，README 還充當軟體文件的目錄，連結到用戶手冊、常見問題和開發指南等資源，供進階用戶或開發者參考。

```
# README

## Introduction

## Quick start

## installation
```
CHANGELOG 則記錄專案的過往，包括軟體變更和版本號次等更新資訊，是用戶或開發者了解開發歷程的入門管道。此外，CHANGELOG 也充當專案歷程的目錄，連結到 issue 和 pull request 等資源，供進階用戶或開發者參考。

```
# CHANGELOG

## [1.1.0] - 2019-02-15

### Added

- Danish translation (#297).
- Georgian translation from (#337).
```

換言之，README 和 CHANGELOG 總結了專案的現況與過往，是用戶與開發者了解專案最基礎的媒介。因此，這兩份文件通常會擺在專案目錄的最頂層，確保所有人一眼就能找到。

# Commits 和 CHANGELOG 的差異
如前所述，CHANGELOG 主要記錄軟體的重要變更，這些變更可能影響軟體的功能或使用方式。典型的 CHANGELOG 會按時序編排條目來排列，並依版本和變更類型組織變更紀錄。每條記錄不僅簡要描述了變更內容，還會連接到相關的需求文檔，補充軟體變更背景資訊。

相較之下，commit messages 主要面向開發者，著重於個別程式碼的變更，而非對整體用戶的影響。標準的 commit messages 通常包括變更類型和簡要說明，其中類型方便篩選和查找相關資訊，而簡要說明則提供了變更的摘要。這些資訊讓查閱者能夠追蹤開發歷程（例如使用 `git log --onelines`），並在需要時回溯程式碼的變更。

```md
docs: correct spelling of CHANGELOG
```
更完整的 commit messages 則包含了變更的背景、原因或機制的詳細解釋，有時還會附上註腳標記相關的 issue 連結，補充程式碼變更的需求情境

```md
fix: prevent racing of requests

Introduce a request id and a reference to latest request. Dismiss
incoming responses other than from latest request.

Refs: #123
```
雖然兩者都是開發紀錄的一部分，但 CHANGELOG 著重於需求的處置，commit 則聚焦程式碼的具體變更；CHANGELOG 通常隨著軟體更新而更新，而 commit 則在提交後鮮少變更；CHANGELOG 強調變更的影響，而 commit 則更注重變更的細節。由於這兩種文件的目標受眾和內容尺度殊異，直接從 commits 生成 CHANGELOG 可能會導致不必要的冗餘資訊，從而無法達到 CHANGELOG 的設計目的。

# Code comments 和 Docstring 的差異
程式碼內的文字不被執行的文字片段可分為 code comments （註解）與 docstring。以 Python 為例，docstring 是記錄函數、類別或模塊的特殊屬性，它們被夾註在三重雙引號之中，用於描述功能、參數和回傳資訊等自訂資訊。透過內置的 `__doc__` 方法可以取得 docstring，使其能被語義化工具解析，用於自動化文件生成，如 docopt。

```
def complex(real=0.0, imag=0.0):
    """Form a complex number.

    Keyword arguments:
    real -- the real part (default 0.0)
    imag -- the imaginary part (default 0.0)
    """
    if imag == 0.0 and real == 0.0:
        return complex_zero
```
另一方面，註解也是程式碼中不執行的文字片段，可用於解釋程式碼的背景或機制。值得注意的是，註解通常無法以內置方法取得，因此較常用於標記任務（如：`TODO`）、提供權宜性解法的提示或對複雜程式碼進行補充說明，並會隨著程式碼的改進而被簡化或刪除。

```
# TODO: remove the deprecated method

# Use the name as the title if the properties did not include one (issue #1425)

# Magical formula taken from a stackoverflow post at <URL>, reputedly related to human vision perception.
```
總結來說，註解的內容較為多樣，可能包括臨時註記或任務提醒；而 docstring 則是程式碼正式文件的一部分，記錄了關於程式本身使用上的更多資訊。


# 開發文件應存放何處？

理想上，開發文件應具備好找、好讀和好維護等特性。這些特性可以這樣定義：

- **好找**：文件應支援全域搜尋或資料庫搜尋，方便獲取重要資訊。
- **好讀**：文件應能在網頁、圖形介面或終端機中打開，並且資訊清晰簡潔。
- **好維護**：文件應集中存放於單一位置並能進行版本控制，以減少文件與開發進度不一致的風險。

考量到這些原則，一些常見的文件格式可能不適合管理開發文件。例如，Office 系列文件不易進行搜尋和版本控制。將文件存放在品管系統 (ISO) 或多個資料夾中也不利於取得文件。而 HTML 和 LaTeX 等格式則包含過多無關的排版標記，使內容難以閱讀。

因此，目前最普遍的做法是使用 Markdown 進行純文本編輯，並將文件統一存放在專案目錄下。這樣不僅便於備份、版控和搜尋，還可以像程式碼一樣按照開發流程更新文件，從而將開發和文件緊密結合，降低不一致的風險。

常見的專案文件結構如下，例如 Python 的 [requests](https://github.com/psf/requests) 專案或生物資訊領域的 [nf-core](https://github.com/nf-core/sarek/tree/3.4.0) 等專案都採用了這樣的結構。

將 README 和 CHANGELOG 這兩種反映專案主要內容的文件放在專案根目錄，方便用戶或開發者取得這些資訊。再以單獨的目錄管理其他自訂文件，並共享多媒體資源，如圖片、影片或字體等。這種做法的好處除了方便管理外，還可以通過程式碼工具將文件編譯成網頁或 PDF 等易於閱讀與分享的格式。

```
project
|- README.md 					# 專案現況的學習途徑                   
|- CHANGELOG.md              	# 開發歷程的入門管道   
|- docs/                        # 自訂文件
    |- assets/                  # 共享的多媒體資源
    |- <usage/output/faq>.md    # 用戶、開發者或分享用的文件
    |- <makefile>               # 用以編譯文件檔為其他格式
```
# 結論
在生物資訊軟體開發的領域中，常見的文件類型可以分為三大類：程式碼本身 (Source Code)、軟體當前狀態 (README)、以及開發歷程 (CHANGELOG)。這些文件不僅涵蓋了軟體的關鍵資訊，還透過互相連結和註腳形式，串聯起相關的文件內容。這樣的做法有利於在必要時將內容改寫為合規文件的格式。最佳的文件管理方法是將其儲存在專案目錄下，採用 Markdown 格式進行純文本編輯。這不僅保證了文件的可檢索性和易讀性，還可通過 Git 進行版本控制，確保文件的維護和更新與程式碼開發流程同步，從而達到易於維護的目的。


# 延伸閱讀
- [Top consideratios for creating bioinformatics software documentation](https://academic.oup.com/bib/article/19/4/693/2907814)
- [Where should you put the documentation](https://understandlegacycode.com/blog/where-to-put-documentation/)
- [Lean/Agile Documnetation: strategies for agile teams](https://agilemodeling.com/essays/agiledocumentation.htm)
- [技術文件產能工具](https://www.ithome.com.tw/voice/95002)