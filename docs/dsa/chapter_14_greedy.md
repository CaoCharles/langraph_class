---
title: 第 14 章：Greedy Algorithm（貪婪演算法）
description: 掌握貪婪策略的選擇與正確性證明，解決經典最佳化問題
---

# 第 14 章：Greedy Algorithm（貪婪演算法）

!!! note "🎯 本章學習目標"
    完成本章後，你將能夠：
    
    - [ ] 理解貪婪演算法的核心思想
    - [ ] 判斷問題是否適合使用貪婪策略
    - [ ] 掌握經典貪婪問題的解題模式
    - [ ] 理解貪婪策略的正確性證明方法
    - [ ] 區分貪婪與動態規劃的適用場景
    
    **預估學習時間**：2.5 小時  
    **前置知識**：第 10 章 排序演算法

---

## 1. 貪婪演算法核心思想

### 什麼是貪婪演算法？

**貪婪演算法**在每一步都選擇**當前看起來最優的選擇**，希望透過局部最優達到全局最優。

```mermaid
graph LR
    A["問題"] --> B["局部選擇 1"]
    B --> C["局部選擇 2"]
    C --> D["局部選擇 3"]
    D --> E["全局最優?"]
    
    style B fill:#51cf66
    style C fill:#51cf66
    style D fill:#51cf66
```

### 貪婪 vs 動態規劃

| 特性 | 貪婪 | 動態規劃 |
|------|------|---------|
| 決策方式 | 只看當前最優 | 考慮所有子問題 |
| 是否回溯 | 不回溯 | 比較所有選擇 |
| 正確性 | 需要證明 | 窮舉保證正確 |
| 效率 | 通常更快 | 可能較慢 |
| 適用條件 | 貪婪選擇性質 | 重疊子問題 |

### 貪婪適用條件

1. **貪婪選擇性質**：局部最優能導向全局最優
2. **最優子結構**：問題的最優解包含子問題的最優解

---

## 2. 區間問題

### Meeting Rooms II（會議室數量）

```python
import heapq

def min_meeting_rooms(intervals: list[list[int]]) -> int:
    """
    計算最少需要幾間會議室
    
    貪婪策略：按開始時間排序，用最小堆追蹤結束時間
    
    Time: O(n log n)
    Space: O(n)
    """
    if not intervals:
        return 0
    
    # 按開始時間排序
    intervals.sort(key=lambda x: x[0])
    
    # 最小堆存結束時間
    heap = [intervals[0][1]]
    
    for start, end in intervals[1:]:
        # 如果當前會議開始時，最早結束的會議已結束
        if start >= heap[0]:
            heapq.heappop(heap)
        
        heapq.heappush(heap, end)
    
    return len(heap)
```

### Non-overlapping Intervals（最少移除區間數）

```python
def erase_overlap_intervals(intervals: list[list[int]]) -> int:
    """
    移除最少的區間使剩餘區間不重疊
    
    貪婪策略：按結束時間排序，保留結束早的區間（給後面更多空間）
    
    Time: O(n log n)
    Space: O(1)
    """
    if not intervals:
        return 0
    
    # 按結束時間排序
    intervals.sort(key=lambda x: x[1])
    
    count = 0
    end = intervals[0][1]
    
    for i in range(1, len(intervals)):
        if intervals[i][0] < end:
            # 重疊，需要移除
            count += 1
        else:
            # 不重疊，更新結束時間
            end = intervals[i][1]
    
    return count
```

### Merge Intervals（合併區間）

```python
def merge(intervals: list[list[int]]) -> list[list[int]]:
    """
    合併所有重疊區間
    
    Time: O(n log n)
    Space: O(n)
    """
    if not intervals:
        return []
    
    intervals.sort(key=lambda x: x[0])
    result = [intervals[0]]
    
    for start, end in intervals[1:]:
        if start <= result[-1][1]:
            result[-1][1] = max(result[-1][1], end)
        else:
            result.append([start, end])
    
    return result
```

---

## 3. 任務排程問題

### Task Scheduler

```python
from collections import Counter

def least_interval(tasks: list[str], n: int) -> int:
    """
    CPU 任務排程：相同任務間隔至少 n 個時間單位
    
    貪婪策略：優先執行剩餘次數最多的任務
    
    公式解法：
    - 找出出現次數最多的任務 max_count
    - 計算需要的時間框架
    
    Time: O(tasks)
    Space: O(1)
    """
    count = Counter(tasks)
    max_count = max(count.values())
    
    # 有多少種任務出現了 max_count 次
    num_max = sum(1 for c in count.values() if c == max_count)
    
    # 公式：(max_count - 1) * (n + 1) + num_max
    # 但不能少於任務總數
    return max(len(tasks), (max_count - 1) * (n + 1) + num_max)
```

### Job Scheduling（最大利潤）

```python
import heapq

def job_scheduling(start: list[int], end: list[int], profit: list[int]) -> int:
    """
    選擇不重疊的工作以獲得最大利潤
    
    注意：這題需要 DP，純貪婪不行！
    """
    from bisect import bisect_right
    
    n = len(start)
    jobs = sorted(zip(end, start, profit))
    
    # dp[i] = 考慮前 i 個工作的最大利潤
    dp = [0] * (n + 1)
    ends = [job[0] for job in jobs]
    
    for i in range(1, n + 1):
        e, s, p = jobs[i - 1]
        # 找最後一個不衝突的工作
        j = bisect_right(ends, s, 0, i)
        dp[i] = max(dp[i - 1], dp[j] + p)
    
    return dp[n]
```

---

## 4. 股票問題

### Best Time to Buy and Sell Stock II

```python
def max_profit(prices: list[int]) -> int:
    """
    可以無限次買賣，求最大利潤
    
    貪婪策略：收集所有上漲區間的利潤
    
    Time: O(n)
    Space: O(1)
    """
    profit = 0
    
    for i in range(1, len(prices)):
        if prices[i] > prices[i - 1]:
            profit += prices[i] - prices[i - 1]
    
    return profit
```

---

## 5. 跳躍遊戲

### Jump Game

```python
def can_jump(nums: list[int]) -> bool:
    """
    判斷是否能跳到最後
    
    貪婪策略：維護能到達的最遠位置
    
    Time: O(n)
    Space: O(1)
    """
    max_reach = 0
    
    for i, jump in enumerate(nums):
        if i > max_reach:
            return False
        max_reach = max(max_reach, i + jump)
    
    return True
```

### Jump Game II（最少跳躍次數）

```python
def jump(nums: list[int]) -> int:
    """
    最少需要幾次跳躍到達終點
    
    貪婪策略：在當前範圍內選擇能跳最遠的位置
    
    Time: O(n)
    Space: O(1)
    """
    n = len(nums)
    if n <= 1:
        return 0
    
    jumps = 0
    curr_end = 0  # 當前跳躍範圍的終點
    max_reach = 0  # 下一跳能到達的最遠位置
    
    for i in range(n - 1):
        max_reach = max(max_reach, i + nums[i])
        
        if i == curr_end:
            jumps += 1
            curr_end = max_reach
            
            if curr_end >= n - 1:
                break
    
    return jumps
```

---

## 6. 字串問題

### Remove K Digits

```python
def remove_k_digits(num: str, k: int) -> str:
    """
    移除 k 位數字使剩餘數字最小
    
    貪婪策略：用單調棧維護遞增序列
    
    Time: O(n)
    Space: O(n)
    """
    stack = []
    
    for digit in num:
        while k > 0 and stack and stack[-1] > digit:
            stack.pop()
            k -= 1
        stack.append(digit)
    
    # 如果還沒移除夠，從末尾移除
    stack = stack[:-k] if k else stack
    
    # 移除前導零
    return ''.join(stack).lstrip('0') or '0'
```

### Partition Labels

```python
def partition_labels(s: str) -> list[int]:
    """
    分割字串使每個字母只出現在一個部分
    
    貪婪策略：每個部分的結束位置是其中字母的最遠出現位置
    
    Time: O(n)
    Space: O(1)
    """
    # 記錄每個字母最後出現的位置
    last = {c: i for i, c in enumerate(s)}
    
    result = []
    start = end = 0
    
    for i, c in enumerate(s):
        end = max(end, last[c])
        
        if i == end:
            result.append(end - start + 1)
            start = i + 1
    
    return result
```

---

## 7. 經典貪婪問題

### Candy（分糖果）

```python
def candy(ratings: list[int]) -> int:
    """
    每個孩子至少一顆糖，評分高的比鄰居多
    求最少糖果數
    
    貪婪策略：兩次遍歷
    
    Time: O(n)
    Space: O(n)
    """
    n = len(ratings)
    candies = [1] * n
    
    # 左到右：比左邊評分高的多一顆
    for i in range(1, n):
        if ratings[i] > ratings[i - 1]:
            candies[i] = candies[i - 1] + 1
    
    # 右到左：比右邊評分高的也要多
    for i in range(n - 2, -1, -1):
        if ratings[i] > ratings[i + 1]:
            candies[i] = max(candies[i], candies[i + 1] + 1)
    
    return sum(candies)
```

### Gas Station

```python
def can_complete_circuit(gas: list[int], cost: list[int]) -> int:
    """
    環形加油站，求起始位置
    
    貪婪策略：
    1. 總油量 >= 總消耗才有解
    2. 如果從 i 無法到達 j，那從 i~j 之間任何點出發也無法到達 j
    
    Time: O(n)
    Space: O(1)
    """
    total = 0
    current = 0
    start = 0
    
    for i in range(len(gas)):
        diff = gas[i] - cost[i]
        total += diff
        current += diff
        
        if current < 0:
            start = i + 1
            current = 0
    
    return start if total >= 0 else -1
```

---

## 8. 正確性證明方法

### 常見證明技巧

| 方法 | 說明 |
|------|------|
| **反證法** | 假設貪婪解不是最優，推導出矛盾 |
| **交換論證** | 證明將最優解改成貪婪選擇不會變差 |
| **歸納法** | 證明每一步的貪婪選擇都保持在最優解中 |

### 範例：區間排程問題

**目標**：選最多不重疊區間

**貪婪策略**：每次選結束時間最早的

**證明（交換論證）**：
1. 設最優解選了區間 A，貪婪解選了區間 B（結束更早）
2. 將 A 換成 B，因為 B 結束更早，不會影響後續區間的選擇
3. 所以貪婪解至少和最優解一樣好

---

## 9. 面試重點

!!! warning "⚠️ 常見陷阱"
    1. **錯誤地使用貪婪**
       - 不是所有最佳化問題都能用貪婪
       - 例如：0/1 背包只能用 DP
    
    2. **忘記排序**
       - 大多數區間問題需要先排序
       - 排序的依據（開始/結束）很重要
    
    3. **邊界條件**
       - 空陣列、單元素
       - 所有區間都重疊

!!! tip "💡 面試技巧"
    - **嘗試多種貪婪策略**：選最大？最小？最早？
    - **舉反例驗證**：能否找到貪婪策略失敗的例子？
    - **能證明最好**：展示正確性證明會加分
    - **對比 DP**：解釋為什麼選擇貪婪而非 DP

---

## 10. LeetCode 對應題目

| 難度 | 題號 | 題目名稱 | 考點 |
|------|------|---------|------|
| 🟢 Easy | #122 | Best Time to Buy and Sell Stock II | 貪婪入門 |
| 🟡 Medium | #55 | Jump Game | 可達性 |
| 🟡 Medium | #45 | Jump Game II | 最少跳躍 |
| 🟡 Medium | #435 | Non-overlapping Intervals | 區間排程 |
| 🟡 Medium | #452 | Minimum Number of Arrows | 區間問題 |
| 🟡 Medium | #763 | Partition Labels | 字串分割 |
| 🟡 Medium | #621 | Task Scheduler | 任務排程 |
| 🔴 Hard | #135 | Candy | 兩次遍歷 |
| 🔴 Hard | #134 | Gas Station | 環形問題 |

---

## 11. 練習題

!!! question "練習 1：Assign Cookies（Easy）"
    **題目描述：**
    
    給定孩子的胃口陣列 g 和餅乾大小陣列 s，每個孩子最多分一塊餅乾，求最多能滿足幾個孩子。
    
    **範例：**
    ```
    輸入：g = [1,2,3], s = [1,1]
    輸出：1
    ```
    
    **提示：**
    <details>
    <summary>點擊查看提示</summary>
    排序後，用最小的餅乾滿足最小胃口的孩子。
    </details>

!!! question "練習 2：Queue Reconstruction by Height（Medium）"
    **題目描述：**
    
    給定 `[[h, k], ...]`，h 是身高，k 是前面有多少人 >= 該身高。重建隊列。
    
    **範例：**
    ```
    輸入：people = [[7,0],[4,4],[7,1],[5,0],[6,1],[5,2]]
    輸出：[[5,0],[7,0],[5,2],[6,1],[4,4],[7,1]]
    ```
    
    **提示：**
    <details>
    <summary>點擊查看提示</summary>
    按身高降序排序，然後依 k 值插入。
    </details>

!!! question "練習 3：Minimum Number of Arrows to Burst Balloons（Medium）"
    **題目描述：**
    
    氣球用區間 [start, end] 表示，一支箭可以射穿 x 位置上的所有氣球。求最少需要幾支箭。
    
    **範例：**
    ```
    輸入：points = [[10,16],[2,8],[1,6],[7,12]]
    輸出：2
    ```
    
    **提示：**
    <details>
    <summary>點擊查看提示</summary>
    按結束位置排序，在最早結束的位置射一箭。
    </details>

---

## 12. 解答與詳解

??? success "練習 1 解答"
    ```python
    def find_content_children(g: list[int], s: list[int]) -> int:
        """
        Time: O(n log n + m log m)
        Space: O(1)
        """
        g.sort()
        s.sort()
        
        child = cookie = 0
        
        while child < len(g) and cookie < len(s):
            if s[cookie] >= g[child]:
                child += 1
            cookie += 1
        
        return child
    ```

??? success "練習 2 解答"
    ```python
    def reconstruct_queue(people: list[list[int]]) -> list[list[int]]:
        """
        貪婪策略：高個子先排，矮個子不影響高個子的 k 值
        
        Time: O(n² )
        Space: O(n)
        """
        # 按身高降序，k 升序
        people.sort(key=lambda x: (-x[0], x[1]))
        
        result = []
        for person in people:
            result.insert(person[1], person)
        
        return result
    ```

??? success "練習 3 解答"
    ```python
    def find_min_arrow_shots(points: list[list[int]]) -> int:
        """
        Time: O(n log n)
        Space: O(1)
        """
        if not points:
            return 0
        
        # 按結束位置排序
        points.sort(key=lambda x: x[1])
        
        arrows = 1
        end = points[0][1]
        
        for start, curr_end in points[1:]:
            if start > end:
                arrows += 1
                end = curr_end
        
        return arrows
    ```

---

## 13. 延伸學習

### 🚀 進階主題
- **Huffman Coding**：貪婪建構最優前綴碼
- **Minimum Spanning Tree**：Prim、Kruskal 演算法
- **Dijkstra 演算法**：貪婪求最短路徑
- **活動選擇問題的變體**：加權區間排程

### 🎉 課程總結

恭喜你完成了整個資料結構與演算法課程！你現在已經掌握了：

- **基礎資料結構**：Array、String、Linked List、Stack、Queue
- **進階資料結構**：Hash Table、Tree、BST、Heap、Graph
- **核心演算法**：排序、Binary Search、Backtracking、DP、Greedy

**下一步建議**：

1. 在 LeetCode 上練習每章的推薦題目
2. 參加模擬面試，練習手寫白板程式碼
3. 複習時間與空間複雜度，確保能清楚說明
4. 準備 System Design（系統設計）面試

祝你面試順利，拿到心儀的 offer！🚀
