# LangGraph 核心概念知識庫

## 什麼是 LangGraph？

LangGraph 是一個用於構建有狀態、多角色應用程式的框架，專為 LLM（大型語言模型）設計。它擴展了 LangChain，提供了循環計算和狀態管理的能力。

### 核心特色

1. **循環支持**：與 DAG（有向無環圖）不同，LangGraph 支持循環，這對於 Agent 迭代非常重要。
2. **狀態管理**：內建狀態管理機制，自動處理狀態的讀取、更新和持久化。
3. **人機協作**：支持在執行過程中暫停，等待人工輸入或批准。

## StateGraph 基礎

StateGraph 是 LangGraph 的核心類別。使用方式如下：

```python
from langgraph.graph import StateGraph, START, END

class MyState(TypedDict):
    messages: list
    count: int

graph = StateGraph(MyState)
graph.add_node("my_node", my_function)
graph.add_edge(START, "my_node")
graph.add_edge("my_node", END)

app = graph.compile()
```

### 節點 (Node)

節點是圖中的處理單元。每個節點：
- 接收完整的 State 作為輸入
- 返回要更新的欄位（dict）
- 不應該直接修改 state

### 邊 (Edge)

邊定義節點之間的連接。有兩種類型：
1. **普通邊**：`add_edge(from, to)` - 固定路徑
2. **條件邊**：`add_conditional_edges(from, router, path_map)` - 動態路徑

## Reducer 機制

Reducer 決定如何合併多個節點對同一欄位的更新。

### add_messages

專門用於對話歷史的 Reducer：

```python
from langgraph.graph.message import add_messages

class State(TypedDict):
    messages: Annotated[list, add_messages]
```

### 自訂 Reducer

```python
def my_reducer(current, update):
    return current + update

class State(TypedDict):
    items: Annotated[list, my_reducer]
```

## Checkpointer 記憶體

Checkpointer 用於保存和恢復狀態。

### MemorySaver

```python
from langgraph.checkpoint.memory import MemorySaver

memory = MemorySaver()
app = graph.compile(checkpointer=memory)

# 使用 thread_id 區分不同對話
config = {"configurable": {"thread_id": "user-1"}}
result = app.invoke(input, config)
```

### 狀態管理 API

- `app.get_state(config)` - 獲取當前狀態
- `app.update_state(config, values)` - 更新狀態
- `app.get_state_history(config)` - 獲取歷史

## ReAct Agent 模式

ReAct (Reasoning + Acting) 是一種讓 Agent 交替思考和行動的模式。

### 流程

```
思考 → 行動 → 觀察 → 思考 → 行動 → ... → 完成
```

### 架構

```python
graph.add_node("agent", agent_node)
graph.add_node("tools", tool_node)

graph.add_conditional_edges("agent", should_continue, {
    "tools": "tools",
    "end": END
})
graph.add_edge("tools", "agent")
```

## 條件路由

使用條件路由實現分支邏輯：

```python
def router(state):
    if state["type"] == "a":
        return "node_a"
    return "node_b"

graph.add_conditional_edges("classifier", router, {
    "node_a": "node_a",
    "node_b": "node_b"
})
```

## 人機協作 (Human-in-the-Loop)

在關鍵節點暫停等待人工輸入：

```python
app = graph.compile(
    checkpointer=memory,
    interrupt_before=["execute"]  # 在這個節點前暫停
)

# 執行到中斷點
result = app.invoke(input, config)

# 人工審核後繼續
app.update_state(config, {"approved": True})
result = app.invoke(None, config)  # None 表示繼續
```

## 多 Agent 系統

### Supervisor 模式

一個主管 Agent 分配任務給工作 Agent：

```python
graph.add_node("supervisor", supervisor_node)
graph.add_node("researcher", researcher_node)
graph.add_node("writer", writer_node)

graph.add_conditional_edges("supervisor", route_to_worker)
graph.add_edge("researcher", "supervisor")
graph.add_edge("writer", "supervisor")
```

### 專家團隊模式

多個專家依序給出意見：

```python
graph.add_edge("tech_expert", "business_expert")
graph.add_edge("business_expert", "legal_expert")
graph.add_edge("legal_expert", "synthesize")
```

## 錯誤處理

### 重試機制

```python
def should_retry(state):
    if state["success"]:
        return "done"
    if state["attempts"] >= state["max_attempts"]:
        return "give_up"
    return "retry"

graph.add_conditional_edges("process", should_retry, {
    "retry": "process",  # 回到自己
    "done": END,
    "give_up": "error_handler"
})
```

### Fallback 機制

```python
def check_fallback(state):
    return "fallback" if state["error"] else "done"

graph.add_conditional_edges("primary", check_fallback, {
    "done": END,
    "fallback": "backup_service"
})
```

## 串流輸出

### stream() 方法

```python
for chunk in app.stream(input):
    for node_name, values in chunk.items():
        print(f"Node: {node_name}, Update: {values}")
```

### stream_mode

- `"updates"`: 只返回更新的欄位
- `"values"`: 返回完整狀態快照

## 子圖

將複雜邏輯封裝為子圖：

```python
# 建立子圖
subgraph = StateGraph(SubState)
subgraph.add_node(...)
sub_app = subgraph.compile()

# 嵌入主圖
main_graph.add_node("process", sub_app)
```

## 最佳實踐

1. **狀態設計**
   - 使用 TypedDict 定義類型
   - 明確指定 Reducer
   - 可選欄位用 `| None`

2. **節點設計**
   - 單一職責
   - 包含錯誤處理
   - 加入日誌輸出

3. **測試**
   - 節點可獨立測試
   - 使用 MemorySaver 測試狀態

4. **生產**
   - 使用 SqliteSaver 或 PostgresSaver
   - 設定重試和超時
   - 加入監控

## 常見問題

### Q: 如何防止無限循環？
A: 在狀態中加入計數器，在路由函數中檢查。

### Q: 如何共享資料給所有節點？
A: 放在 State 中，所有節點都能存取。

### Q: 如何除錯？
A: 使用 stream() 觀察每個節點的輸出，或使用 LangSmith 追蹤。

### Q: 記憶體會無限增長嗎？
A: 可以實作記憶體裁剪，只保留最近 N 條訊息。
