---
title: ROSALIND｜Reversal Distance (REAR)
date: 2024-06-03 08:50:08
tags: rosalind
categories:
---

給定 5 對長度為 10 的數列，計算將各對數列互相轉換所需的最少反轉次數。

> A reversal of a permutation creates a new permutation by inverting some interval of the permutation; (5,2,3,1,4) , (5,3,4,1,2) , and (4,1,2,3,5) are all reversals of (5,3,2,1,4) . The reversal distance between two permutations π and σ , written drev(π,σ) , is the minimum number of reversals required to transform π into σ (this assumes that π and σ have the same length).
>
> Given: A collection of at most 5 pairs of permutations, all of which have length 10.
>
> Return: The reversal distance between each permutation pair.

(https://rosalind.info/problems/rear/)

<!--more-->

# 題解

染色體特定片段前後翻轉的現象稱為 inversion，它是基因體重排最常見的形式。計算兩條染色體透過 inversion 互相轉換所需的最少次數，可視同求解兩數列之間的反轉距離（reversal distance）。

假設我們有一個起始數列和一個目標數列。若兩數列完全相同，則反轉距離為 0，不須進行任何反轉操作。但如果兩者不一致，可先列出起始數列經過一次反轉後可能產生的所有數列，再檢查目標數列是否出現在新生成的數列中。

如果找到目標數列，就回報所需的反轉次數；如果沒找到，就列出這些數列再次返喘後可能產生的所有數列。

重複上述過程，列出經過 n 次反轉後可能產生的所有數列，並且檢查目標數列是否出現其中，直到找到目標數列或窮盡所有可能為止。

為了實現這個算法，首先需要一個副程式，用於列出一個數列經過一次反轉後可能產生的所有數列。簡言之，在數列上任選兩個位置，用 `[::-1]` 反轉居於其間的子數列，再連結反轉數列與剩餘數列。

這些數列可以儲存在 list、tuple、或 set 等資料型別中。我選擇使用 set 是因為它能自動排除重複的數列，減少多餘的比較。其次，與 list 和 tuple 相比，判斷一個元素是否在 set 之中只需要常數時間，能減少比較的時間成本。最後，set 支援交集、聯集和差集等操作，在比較數列集合時有比較多彈性。

```python
def swap(l):
    """
    Generate all possible subsequences of a tuple by reversing segments.

    Args:
        l (tuple): Input tuple, e.g., (1, 2).

    Returns:
        set: Set of tuples with reversed segments, e.g., {(1, 2), (2, 1)}.
    """
    return {
        l[:i] + l[i:j][::-1] + l[j:]
        for j in range(len(l) + 1)
        for i in range(j)
    }
```

除此之外，計算反轉距離還需要以下三項資訊：
- curr：記錄準備與目標數列比較的數列集合
- visited：記錄已經檢查過的數列集合
- dist：記錄當前的反轉次數

具體的流程和程式碼如下：
1. 檢查目標數列是否出現在 curr 中。若有找到目標數列，則回報當前的反轉次數  dist。
2. 若沒有找到，則依序反轉 curr 中每個數列，但要排除掉已經檢查過的數列visited。
3. 將新生成的反轉數列納入已檢查過的數列集合 visited，以免重複比較。
4. 將所有可能的反轉結果更新到 curr，以便下一輪比較。同時，將當前反轉次數 dist 的值加 1。
5. 更新完畢後，回到步驟 1. 繼續搜索，直到找出目標數列（回傳 dist），或是檢查過所有可能性仍未找到目標數列（回傳 -1）

```python
def rev_dist(s1, s2):
    """Calculate the minimum number of reversals required to transform tuple s1 to tuple s2.

    Args:
        s1 (tuple): Starting tuple.
        s2 (tuple): Target tuple.

    Returns:
        int: Minimum number of swaps required, or -1 if transformation is not possible.
    """
    curr, visited = {s1}, {s1}
    dist = 0
    while curr:
        if s2 in curr:
            return dist
        else:
            next_step = set()
            for e in curr:
                next_step.update(swap(e).difference(visited))
                visited.update(next_step)
            curr = next_step
            dist = dist + 1
    return -1
```

# 討論

這題的觀念其實不難理解，但真正的挑戰在於計算效率。隨著數列長度和反轉次數增加，算法的時間複雜度會呈指數增長。依照題目要求的數列長度測試，我的電腦大約只能處理到 4-5 次反轉，再多的反轉次數就無法在合理的時間內算出來。

想要評估計算反轉距離的成本可以考慮以下三個因素：比較成本（$c$, Cost）、可能的數列數量（$n$, Number）、檢查的深度（$d$, Depth or Distance），即 $cn^d$。為了提升計算效率，可以從這三項因素著手。

- 降低每次比較的成本，例如以 set 取代 list。
- 降低可能的數列數量，例如排除已檢查過的數列或是只生成有用的數列等。
- 降低檢查深度，例如同時從數列的前端和後端開始檢查。

這些方式都能提升計算效率，但適用的情境不盡相同。如果數列長度很短，可能的反轉結果會比較少，反轉距離也不長，那麼關鍵就在於降低每次比較的成本。反之，如果像這題一樣需要處理較長的數列，那麼重點就轉移到了控制可能的數列數量和檢查深度上。

雖然排除已檢查過的數列能起到一定作用，但問題在於反轉距離還是太長了，相較於指數增長的待檢查數列，重複數列的數量顯得微不足道。

另一種策略是在生成可能的反轉數列時，只生成有機會轉換為目標數列的數列。這方法若能有效降低 $n$ 值，便能容忍更長的反轉距離以及比較數列的長度。

然而，這想法雖然理想，但它的困難在於不容易找到正確的篩選規則來判斷一個數列是否有可能轉換為目標數列。即使找到一項不錯的規則，但也很難保證這規則不會遺漏某西可能性，導致它雖然能在有限時間內回應，卻未必是最優解。

最後一種手段則相對值觀，就是分別從數列的前後端開始搜索，這樣可以將檢查的深度減半。如題目要求，對於長度為 10 的數列，即使窮舉，最多經過 9 次反轉便能轉會為任意數列。

如前所述，反轉距離大於 4 - 5 次就達到當前計算能力的極限，從兩端同時搜索，正好能將檢查深度控制在極限以內。

雙向排查的程式碼如下所示。為了降低主程式的複雜度，我新增了副程式 get_next_steps，用於生成數列集合的每個元素在經過一次反轉後可能產生的所有數列。
```python
def get_next_steps(curr, visited):
    """Generates next steps by swapping elements in `curr`, excluding those in `visited`.

    Args:
        curr (set): Current states.
        visited (set): States to exclude.

    Returns:
        set: Next possible states.
    """
    next_steps = set()
    for e in curr:
        next_steps.update(swap(e).difference(visited))
    return next_steps
```

其餘部分則與只從一端開始檢查的方式相似，只是要有兩組 curr 和 visited 分別存儲從前後端檢查數列的資訊。
```python
def rev_dist(s1, s2):
    f_curr, f_visited, b_curr, b_visited = {s1}, {s1}, {s2}, {s2}
    dist = 0
    while f_curr and b_curr:
        if f_curr.intersection(b_curr):
            return dist
        else:
            dist = dist + 1
            f_curr = get_next_steps(f_curr, f_visited)
            f_visited.update(f_curr)
        if b_curr.intersection(f_curr):
            return dist
        else:
            dist = dist + 1
            b_curr = get_next_steps(b_curr, b_visited)
            b_visited.update(b_curr)
    return -1
```

# 其它解法
說點題外話，這題算得上 Rosalind 平台的東巴了，我想半天解不出來，看了討論區才知道原來有那麼多人也覺得出題方在整人。儘管題目提示 "Don't be afraid to try an ugly solution."，但真想用暴力法解題，是絕對沒辦法在規定時間內通過測試的。

以下是我找到的另一種基於雙向搜索的解法，雖然它沒有處理起始數列與目標數列無法互相轉換的狀況，但整體想法卻清晰易懂。這方法首先將輸入的數列轉換為 set 以利比較，接著判斷兩者是否有交集。如果有，則直接回傳當前的反轉距離；反之，則為每個數列集合中的元素生成所有可能的數列，再判斷新生成的數列集合彼此是否交集。如果仍沒有交集，就遞迴呼叫這個副程式，持續搜索下一層，直到找到交集為止。

```python
def rev_dist(s1, s2, dist=0):
   s1 = {s1} if isinstance(s, tuple) else s1
   s2 = {s2} if isinstance(t, tuple) else s2
   if s1.intersection(s2):
        return dist
   else:
        next_s1 = { swap_e for e in s1 for swap_e in swap(s1) }
        next_s2 = { swap_e for e in s2 for swap_e in swap(s2) }
        if s1.intersection(next_s1) or s2.intersection(next_s1):
            return dist + 1
        elif next_s1.intersection(next_s2):
            return dist + 2
        else:
            return rev_dist(next_s1, next_s2, dist + 2)
```

