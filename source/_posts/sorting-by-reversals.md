---
title: ROSALIND｜Sorting by Reversals (SORT)
date: 2024-06-07 17:53:35
tags: rosalind
categories:
---

給定一對數列，計算兩者的最小反轉距離，並依序列出反轉數列的步驟。

> A reversal of a permutation can be encoded by the two indices at the endpoints of the interval that it inverts; for example, the reversal that transforms (4,1,2,6,3,5) into (4,1,3,6,2,5) is encoded by [3,5] .
> A collection of reversals sorts π into γ if the collection contains drev(π,γ) reversals, which when successively applied to π yield γ .

> Given: Two permutations π and γ , each of length 10.
> Return: The reversal distance drev(π,γ) , followed by a collection of reversals sorting π into γ . If multiple collections of such reversals exist, you may return any one.

(https://rosalind.info/problems/sort/)

<!--more-->

這題的概念與 [Reversal Distance](https://5uperb0y.com/reversal-distance/) 相同，只是需要修改程式碼以便記錄和追蹤數列反轉的步驟。首先，在生成反轉數列的同時，我用 dictionary 記錄新生成數列的來源以及被反轉的子數列的首尾位置。這裡把首位座標值加 1 是因為題目的座標系統從 1 開始）。

雖然這些資訊也可以用 list 紀錄，但使用 dictionary 可以透過 key 將資訊分門別類，在取值時會比用 list 時以座標取值明確。

```python
def swap(t):
    return {
        t[:i] + t[i:j][::-1] + t[j:]: {"parent": t, "move": (i + 1, j)}
        for j in range(len(t) + 1)
        for i in range(j)
    }
```

以下這個副程式用於生成數列集合的每個元素在經過一次反轉後可能產生的所有數列。由於修改了 swap 的輸出格式，所以這副程式也要跟著調整，確保之後能取得新生成的數列、這些數列的來源以及反轉步驟等資訊。

雖然以 for 調用 dictionary 的時候，直接寫變項名稱（`var`）的效果與 `var.keys()` 一致，不過因為我常常忘記這特性，所以覺得還是清楚列出來比較好。

```python
def get_next_steps(current, visited):
    return {
        swap_e: details
        for e in current.keys()
        for swap_e, details in swap(e).items()
        if swap_e not in visited.keys()
    }
```

由於記錄了各數列的源頭與反轉方式，所以配合已檢查過的數列清單 visited，即可追蹤數列的反轉步驟。簡而言之，就是拿當前數列去查詢 visited，記錄反轉步驟，再查出該數列的源頭，查詢數列源頭的反轉步驟，直到追蹤到起始數列。

這裡我假設起始數列的資料格式為 `{seq: -1}`，所以只要檢查到 -1 就表示已經追蹤到起點了。

```python
def get_moves(meet_point, visited):
    moves = []
    current = visited[meet_point]
    while current != -1:
        moves.append(current["move"])
        current = visited[current["parent"]]
    return moves
```

除了變項名稱以外，我也對計算反轉距離的功能做了一些調整。首先，因應使用 dictionary 記錄數列，比較雙向搜索是否有交集時，得額外取出兩者的 keys 來比對。

其次，我轉而以交錯方式來推進搜索：利用 `check_forward` 當成開關，這變項決定這次檢查要從哪個方向開始，這樣能確保每次迴圈只會從數列前端或後端推進一步。這樣就不用像我之前寫得那樣，在每個迴圈為各方向的推進檢查兩次。

其餘的步驟與先前的寫法類似，一旦雙向搜索的進度交錯，就分別使用 get_moves 重建反轉步驟。

因為這個副程式預設用 append() 添加元素，所以對於從前端開始的檢查而言，在追蹤完所有反轉步驟後，要將輸出的 list 反轉才能得到正確的順序。結合前後兩端的反轉步驟，即得到兩數列互相轉換所需經過得最少反轉步驟。

```python
def rev_dist(s1, s2):
    cur_fwd, vis_fwd = {s1: -1}, {s1: -1}
    cur_bwd, vis_bwd = {s2: -1}, {s2: -1}
    dist = 0
    check_forward = True
    while cur_fwd and cur_bwd:
        intersection = cur_fwd.keys() & cur_bwd.keys()
        if intersection:
            meet_point = intersection.pop()
            return dist, get_moves(meet_point, vis_fwd)[::-1] + get_moves(meet_point, vis_bwd)
        else:
            if check_forward:
                cur_fwd = get_next_steps(cur_fwd, vis_fwd)
                vis_fwd.update(cur_fwd)
            else:
                cur_bwd = get_next_steps(cur_bwd, vis_bwd)
                vis_bwd.update(cur_bwd)
            dist = dist + 1
            check_forward = not check_forward
    return -1
```

# 討論

## and、& 和 intersection 的差異
intersection 和 & 是 set 專用的操作符，兩者的功能相同。
```python
> a = {1, 2, 3}
> b = {3, 4, 5}
> a & b
{3}
> a.intersection(b)
{3}
```
然而 & 只允許 set 之前的操作，intersection 則支援 set 和任何 iterable 的比較，所以有可讀性和泛用性的優勢。詳情可參考這則 [stackoverflow 的說明](https://stackoverflow.com/questions/63827915/and-intersection-difference) 和[官方文件](https://docs.python.org/3.8/library/stdtypes.html#frozenset.intersection)

```python
> a = {1, 2, 3}
> a.intersection([3, 4, 5])
{3}
> a.intersection({3:3, 4:4, 5:5})
{3}
```

and 則是邏輯判斷符，不能用來集合運算。
```python
# 這裡的邏輯是：a is True, then b，所以印出 b。
> a and b
{3, 4, 5}
# 用這案例會更加清楚：{} is False，所以不會印出 b，只有呈現 {}
> {} and b
{}
```

## 其他解題想法

我覺得這種需要追蹤路徑的問題，如果用 class 來設計結構與方法可能會讓程式碼變得好維護許多。另外，之前在 Reversal Distance 也有列出用遞迴的解題方式，或許有空都可以拿來改寫看看。