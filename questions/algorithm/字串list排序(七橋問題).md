# 字串 list 組排序(七橋問題)

## 問題

給定字串 list，要求進行判斷是否存在以下序列

前一個字串末尾字元等於後一個字串首字元。

例如：

```python
{ "ab" , "de" , "bc" , "cd" } #此字串list滿足條件
{ "ab" , "bc" , "cd" , "fg" } #此字串list不滿足條件
```

問題出自 [segmentfault](https://segmentfault.com/q/1010000005152540/a-1020000005595508), by [nbandroid](https://segmentfault.com/u/nbandroid)

## 回答

[ted](https://segmentfault.com/u/ted) 大的算法非常完整，就算法的部份可以參考他的說法

>其實就是七橋問題，把一個字符串視作從首字母到尾字母的有向鏈接，求特定字符串集合能不能一次不重複走完。從圖論的角度來看，由於可能存在不連通的子圖，所以不能簡單地用各個節點的度來判斷，但節點的度可以用來快速排除不符合的集合。
>所以很簡單，如果集合中存在這樣的序列，必然是以下情況之一：
>
>循環。所有節點的出度等於入度；
>
>不循環。除了某兩個節點外，其他所有節點的出度等於入度。
>
>如果符合上面的條件，那麼還要跑一次 dfs 看能不能單路徑遍歷所有邊。
>如果要排序，那就是 dfs 出來的那條路徑。
> by ted

以下是我用 Python 實現的一個範例(`algo.py`):

首先是一個基礎的 `dfs` function，不需要做任何事前確認就能夠正確地辨識出 string list 是否符合條件:

```python
def dfs(prestr, str_lst):
    for idx, string in enumerate(str_lst):
        if not prestr or string[0] == prestr[-1]:
            if len(str_lst)==1:
                return [string]
            # print(string, str_lst[:idx] + str_lst[idx+1:])  # 反註解掉此行用以觀察搜尋的過程
            result = dfs(string, str_lst[:idx] + str_lst[idx+1:])
            if result:
                return [string] + result
    return None
```

但是有了事前檢查可以減少不必要的 `dfs`。

用來作提前檢查的 `pre_check`:

```python
import collections

def pre_check(str_lst):

    heads = (string[0] for string in str_lst)
    tails = (string[-1] for string in str_lst)
    first_head = None
    last_tail = None

    head_counter = collections.Counter(heads)
    tail_counter = collections.Counter(tails)

    sub_counter = head_counter-tail_counter
    if len(sub_counter) > 1:
        return False, None, None
    elif len(sub_counter)==1:
        first_head = list(sub_counter)

    sub_counter = tail_counter-head_counter
    if len(sub_counter) > 1:
        return False
    elif len(sub_counter)==1:
        last_tail = list(sub_counter)

    return True, first_head, last_tail
```

有作 `pre_check` 的 完整版 `check`:

```python
def check(str_lst):
    result, first_head, last_tail = pre_check(str_lst)
    if result:
        if (first_head is None and last_tail is None) or (first_head and last_tail):
            return dfs(None, str_lst)
        else:
            return None
    else:
        return None
```

測試結果:

```python
>>> from algo import dfs, check
>>> s1 = ["ab", "de", "bc", "cd"]
>>> dfs(None, s1)
['ab', 'bc', 'cd', 'de']
>>> check(s1)
['ab', 'bc', 'cd', 'de']
>>> s2 = ["ab","bc","cd","fg"]
>>> dfs(None, s2)  # 回傳 None 代表 s2 並不符合條件
>>> check(s2)  # 回傳 None 代表 s2 並不符合條件
>>> s3 = ["gh","ab","ef","hi","bc","cd","fg","de"]  # 無循環
>>> dfs(None, s3)
['ab', 'bc', 'cd', 'de', 'ef', 'fg', 'gh', 'hi']
>>> check(s3)
['ab', 'bc', 'cd', 'de', 'ef', 'fg', 'gh', 'hi']
>>> s4 = ["gh","ab","ef","ha","bc","cd","fg","de"]  # 循環
>>> dfs(None, s4)
['gh', 'ha', 'ab', 'bc', 'cd', 'de', 'ef', 'fg']
>>> check(s4)
['gh', 'ha', 'ab', 'bc', 'cd', 'de', 'ef', 'fg']
```
