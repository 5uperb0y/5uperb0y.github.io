---
title: 軟體測試的無限後退
date: 2024-08-01 00:04:59
tags: development
categories:
---
在修改程式碼之後，開發人員會執行測試以確保軟體的各項功能運行無誤。由於測試通常以程式碼實現，所以它們也可能存在 bug。鑒於這種風險，有些人可能考慮撰寫測試的測試，試圖為軟體的品質把關。

不過，依據這個想法，測試的測試也應該被檢驗其正確性，因此需要額外的測試來支持既有的測試。以此類推，它們會衍生出無窮個測試，形成軟體測試的無限後退難題（infinite regress）。

在實務上，無窮測試顯然不可行，那麼開發人員應該如何確保測試的可信度與有效性呢？
<!--more-->

這個挑戰來自我們公司的法規管制部門（Regulatory Affairs, RA）。在生醫軟體的開發環境裡，RA 部門負責監督各團隊的作業流程，以確保產品遵循 IVDs 或 LDTs 等政府規範。因此，他們特別在意軟體更新與架構異動的風險。

當我們為了引進自動化測試而諮詢 RA 部門時，雖然他們認同這項提議立意良善，但仍不免從風險管控的立場，要求我們驗證測試手段的有效性。在會議當下，我隱約感到這方法不實際，卻提不出完整的理由說服對方。

因此，我對驗證測試的方法做了一些調查。最終發現，比起撰寫測試的測試，善用其他手段更能保障測試的有效性。

首先，撰寫簡單易懂的測試程式碼，讓它不證自明，就不需要額外覆核，從而終止測試的後退。另外，簡潔的程式碼也有助於辨識錯誤和手動調試。

其次，讓軟體與測試互相支持，形成循環論證來避免無限後退。例如在測試導線開發（Test-Driven Development, TDD）裡，在設計一項功能前，會先建立對應的測試。如果功能尚未完成，卻通過其測試，就表示測試程式碼有誤。反之，如果完成功能後無法通過測試，就表示兩者之一需要修正。隨著軟體迭代，這些測試一方面保障既有的功能，另一方面也受到新增的功能挑戰，達到兩者彼此檢驗與支持的效果。

最後一種觀點則涉及測試的本質。測試是除錯的手段而非證成的依據，測試通過能提供的線索有限，但測試失敗卻直接指引了修改程式碼的方向。因此，與其去證明已經通過的測試，不如提升既有測試的覆蓋度與靈敏度，增加改善軟體品質的機會。

綜上所述，如果想要保障測試的可信度與有效性，我們不需要為測試撰寫測試。更有效的手段是維護簡要而精準的測試，讓軟體與測試隨著開發歷程彼此支持與改善。

- [How do you solve the infinite regress problem in TDD?](https://stackoverflow.com/questions/2113601/how-do-you-solve-the-infinite-regress-problem-in-tdd)
- [How to Improve Your Tests by Being an Evil Coder](https://coderefinery.nz/2019/08/15/an-adversarial-process-to-improve-your-tests/)
- [Pragmatic Unit Testing in Java 8 with JUnit](https://pragprog.com/titles/utj2/pragmatic-unit-testing-in-java-8-with-junit/)
- [To TDD or not to TDD](https://enterprisecraftsmanship.com/posts/tdd-not-tdd/)
- [卡爾．波柏（華文哲學百科）](https://mephilosophy.ccu.edu.tw/entry.php?entry_name=%E5%8D%A1%E7%88%BE%EF%BC%8E%E6%B3%A2%E6%9F%8F)