# 🚀 LangGraph Agent 學習指南

> **LangGraph** 是由 LangChain 團隊開發的框架，專門用於建構複雜、有狀態的 AI Agent 應用程式。本指南將帶領您從零開始，逐步掌握 LangGraph 的核心概念與程式撰寫技巧。

---

## 📋 目錄

1. [前置準備](#1-前置準備)
2. [LangGraph 核心概念](#2-langgraph-核心概念)
3. [第一個 LangGraph 應用](#3-第一個-langgraph-應用)
4. [狀態管理 (State Management)](#4-狀態管理-state-management)
5. [節點與邊 (Nodes & Edges)](#5-節點與邊-nodes--edges)
6. [條件路由與分支 (Conditional Routing)](#6-條件路由與分支-conditional-routing)
7. [Tool 整合與呼叫](#7-tool-整合與呼叫)
8. [ReAct Agent 模式](#8-react-agent-模式)
9. [多 Agent 系統](#9-多-agent-系統)
10. [記憶體與持久化](#10-記憶體與持久化)
11. [Human-in-the-Loop 人機協作](#11-human-in-the-loop-人機協作)
12. [錯誤處理與最佳實踐](#12-錯誤處理與最佳實踐)
13. [進階主題與學習資源](#13-進階主題與學習資源)

---

## 1. 前置準備

### 1.1 環境需求

```bash
# Python 版本要求
Python >= 3.9

# 建議使用虛擬環境
python -m venv langgraph_env
source langgraph_env/bin/activate  # macOS/Linux
```

### 1.2 安裝套件

```bash
# 核心套件
pip install langgraph langchain langchain-openai

# 可選：其他 LLM 提供者
pip install langchain-anthropic  # Claude
pip install langchain-google-genai  # Gemini
```

### 1.3 API 金鑰設定

```python
import os

# 設定 OpenAI API Key
os.environ["OPENAI_API_KEY"] = "your-api-key"

# 或使用 .env 檔案
from dotenv import load_dotenv
load_dotenv()
```

### 📌 學習重點
- 確保 Python 環境正確配置
- 理解 LangGraph 與 LangChain 的關係
- 熟悉 API 金鑰管理方式

---

## 2. LangGraph 核心概念

### 2.1 什麼是 LangGraph？

LangGraph 將 AI 工作流程建模為**有向圖 (Directed Graph)**，由三個核心元素組成：

```
┌─────────────────────────────────────────────────────────┐
│                    LangGraph 架構                        │
├─────────────────────────────────────────────────────────┤
│                                                         │
│   ┌─────────┐    Edge    ┌─────────┐    Edge           │
│   │  Node   │ ────────▶  │  Node   │ ────────▶  ...    │
│   │ (處理器) │            │ (處理器) │                   │
│   └─────────┘            └─────────┘                   │
│        │                      │                         │
│        ▼                      ▼                         │
│   ┌─────────────────────────────────────────┐          │
│   │              State (狀態)                │          │
│   │         共享的執行上下文                  │          │
│   └─────────────────────────────────────────┘          │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### 2.2 三大核心元素

| 元素 | 說明 | 類比 |
|------|------|------|
| **Nodes (節點)** | 執行特定任務的處理單元 | 工作站 |
| **Edges (邊)** | 定義節點間的連接與流向 | 傳送帶 |
| **State (狀態)** | 在節點間共享的資料結構 | 工作單 |

### 2.3 為什麼選擇 LangGraph？

| 場景 | LangChain | LangGraph |
|------|-----------|-----------|
| 線性流程 | ✅ 適合 | ⚠️ 過度設計 |
| 條件分支 | ⚠️ 較難 | ✅ 原生支援 |
| 循環迭代 | ❌ 不支援 | ✅ 原生支援 |
| 有狀態對話 | ⚠️ 需額外處理 | ✅ 內建支援 |
| 多 Agent | ❌ 複雜 | ✅ 原生支援 |

### 📌 學習重點
- 理解圖 (Graph) 的概念與優勢
- 區分 Node、Edge、State 的職責
- 了解何時應該使用 LangGraph

---

## 3. 第一個 LangGraph 應用

### 3.1 基本架構

```python
from typing import TypedDict, Annotated
from langgraph.graph import StateGraph, START, END
from langchain_openai import ChatOpenAI

# Step 1: 定義狀態結構
class State(TypedDict):
    messages: list[str]
    response: str

# Step 2: 定義節點函數
def chatbot_node(state: State) -> dict:
    """處理使用者訊息並生成回應"""
    llm = ChatOpenAI(model="gpt-4o-mini")
    user_message = state["messages"][-1]
    response = llm.invoke(user_message)
    return {"response": response.content}

# Step 3: 建構圖
workflow = StateGraph(State)

# Step 4: 新增節點
workflow.add_node("chatbot", chatbot_node)

# Step 5: 定義邊 (流程)
workflow.add_edge(START, "chatbot")
workflow.add_edge("chatbot", END)

# Step 6: 編譯並執行
app = workflow.compile()

# 執行
result = app.invoke({"messages": ["Hello, how are you?"], "response": ""})
print(result["response"])
```

### 3.2 執行流程圖解

```
START ──▶ chatbot ──▶ END
             │
             ▼
        生成回應
```

### 📌 學習重點
- 掌握 `StateGraph` 的建構流程
- 理解 `START` 和 `END` 特殊節點
- 學會使用 `compile()` 和 `invoke()`

---

## 4. 狀態管理 (State Management)

### 4.1 使用 TypedDict 定義狀態

```python
from typing import TypedDict, Annotated
from operator import add

class ConversationState(TypedDict):
    # 基本欄位
    user_input: str
    
    # 使用 Annotated 定義更新方式
    messages: Annotated[list[str], add]  # 累加模式
    
    # 可選欄位
    metadata: dict
```

### 4.2 狀態更新機制

```python
from langgraph.graph import add_messages

class ChatState(TypedDict):
    # 使用內建的 add_messages reducer
    messages: Annotated[list, add_messages]
    current_step: str

def node_a(state: ChatState) -> dict:
    # 只返回要更新的欄位
    return {
        "messages": [{"role": "assistant", "content": "Hello!"}],
        "current_step": "greeting_complete"
    }
```

### 4.3 Reducer 函數說明

| Reducer | 行為 | 使用場景 |
|---------|------|----------|
| `add` | 累加列表 | 訊息歷史 |
| `add_messages` | 智能合併訊息 | 對話管理 |
| 自訂函數 | 自訂邏輯 | 複雜更新 |

### 📌 學習重點
- 使用 `TypedDict` 確保型別安全
- 理解 `Annotated` 與 Reducer 的作用
- 掌握狀態的不可變更新原則

---

## 5. 節點與邊 (Nodes & Edges)

### 5.1 節點類型

```python
from langchain_openai import ChatOpenAI
from langgraph.graph import StateGraph

# 類型一：普通函數
def process_input(state):
    return {"processed": state["raw_input"].strip()}

# 類型二：LLM 呼叫
def llm_node(state):
    llm = ChatOpenAI()
    response = llm.invoke(state["messages"])
    return {"messages": [response]}

# 類型三：工具呼叫
def tool_node(state):
    # 執行外部 API 或工具
    result = external_api_call(state["query"])
    return {"tool_result": result}

# 新增到圖
graph = StateGraph(State)
graph.add_node("process", process_input)
graph.add_node("generate", llm_node)
graph.add_node("execute", tool_node)
```

### 5.2 邊的類型

```python
from langgraph.graph import StateGraph, START, END

graph = StateGraph(State)

# 1. 基本邊：無條件連接
graph.add_edge("node_a", "node_b")

# 2. 條件邊：根據狀態決定下一步
def router(state):
    if state["needs_tool"]:
        return "tool_node"
    return "end_node"

graph.add_conditional_edges(
    "decision_node",
    router,
    {
        "tool_node": "tool_node",
        "end_node": END
    }
)
```

### 5.3 流程圖示例

```
                    ┌──────────────┐
                    │    START     │
                    └──────┬───────┘
                           │
                    ┌──────▼───────┐
                    │   process    │
                    └──────┬───────┘
                           │
                    ┌──────▼───────┐
                    │   decision   │
                    └──────┬───────┘
                           │
             ┌─────────────┴─────────────┐
             │                           │
      ┌──────▼───────┐            ┌──────▼───────┐
      │  tool_node   │            │   generate   │
      └──────┬───────┘            └──────┬───────┘
             │                           │
             └─────────────┬─────────────┘
                           │
                    ┌──────▼───────┐
                    │     END      │
                    └──────────────┘
```

### 📌 學習重點
- 理解不同節點的設計模式
- 掌握條件邊的路由邏輯
- 學會設計複雜的工作流程圖

---

## 6. 條件路由與分支 (Conditional Routing)

### 6.1 基本條件路由

```python
from typing import Literal

def should_continue(state) -> Literal["continue", "end"]:
    """決定是否繼續執行"""
    messages = state["messages"]
    last_message = messages[-1]
    
    # 判斷邏輯
    if last_message.tool_calls:
        return "continue"
    return "end"

# 設定條件邊
graph.add_conditional_edges(
    "agent",
    should_continue,
    {
        "continue": "action",
        "end": END
    }
)
```

### 6.2 多分支路由

```python
def intent_router(state) -> Literal["search", "calculate", "chat", "end"]:
    """根據意圖分類"""
    intent = state.get("detected_intent", "chat")
    
    intent_map = {
        "question": "search",
        "math": "calculate",
        "conversation": "chat",
        "goodbye": "end"
    }
    
    return intent_map.get(intent, "chat")

graph.add_conditional_edges(
    "classifier",
    intent_router,
    {
        "search": "search_node",
        "calculate": "calculate_node",
        "chat": "chat_node",
        "end": END
    }
)
```

### 6.3 循環控制

```python
def check_iteration(state) -> Literal["continue", "stop"]:
    """防止無限循環"""
    iteration_count = state.get("iteration", 0)
    max_iterations = 5
    
    if iteration_count >= max_iterations:
        return "stop"
    if state.get("task_complete", False):
        return "stop"
    return "continue"
```

### 📌 學習重點
- 使用 `Literal` 型別確保路由安全
- 設計清晰的路由邏輯
- 實作循環計數器防止無限迴圈

---

## 7. Tool 整合與呼叫

### 7.1 定義工具

```python
from langchain_core.tools import tool

@tool
def search_web(query: str) -> str:
    """搜尋網路資訊
    
    Args:
        query: 搜尋關鍵字
    """
    # 實際實作搜尋邏輯
    return f"搜尋結果：關於 '{query}' 的資訊..."

@tool
def calculate(expression: str) -> str:
    """計算數學表達式
    
    Args:
        expression: 數學表達式
    """
    try:
        result = eval(expression)
        return f"計算結果：{result}"
    except Exception as e:
        return f"計算錯誤：{str(e)}"

# 工具列表
tools = [search_web, calculate]
```

### 7.2 綁定工具到 LLM

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4o-mini")
llm_with_tools = llm.bind_tools(tools)
```

### 7.3 使用 ToolNode

```python
from langgraph.prebuilt import ToolNode

# 建立工具節點
tool_node = ToolNode(tools)

# 在圖中使用
graph = StateGraph(MessagesState)
graph.add_node("tools", tool_node)
```

### 7.4 完整工具呼叫流程

```python
from typing import Annotated, TypedDict
from langgraph.graph import StateGraph, START, END
from langgraph.graph.message import add_messages
from langgraph.prebuilt import ToolNode
from langchain_openai import ChatOpenAI

class State(TypedDict):
    messages: Annotated[list, add_messages]

def agent(state: State):
    llm = ChatOpenAI(model="gpt-4o-mini").bind_tools(tools)
    response = llm.invoke(state["messages"])
    return {"messages": [response]}

def should_use_tools(state) -> str:
    last_message = state["messages"][-1]
    if last_message.tool_calls:
        return "tools"
    return END

# 建構圖
graph = StateGraph(State)
graph.add_node("agent", agent)
graph.add_node("tools", ToolNode(tools))

graph.add_edge(START, "agent")
graph.add_conditional_edges("agent", should_use_tools)
graph.add_edge("tools", "agent")  # 工具執行後回到 agent

app = graph.compile()
```

### 📌 學習重點
- 使用 `@tool` 裝飾器定義工具
- 理解 `bind_tools()` 的作用
- 掌握工具呼叫與結果處理的循環

---

## 8. ReAct Agent 模式

### 8.1 ReAct 概念

ReAct (Reasoning + Acting) 是一種讓 Agent 交替進行**推理**和**行動**的模式：

```
思考 (Thought) → 行動 (Action) → 觀察 (Observation) → 思考 → ...
```

### 8.2 使用預建 ReAct Agent

```python
from langgraph.prebuilt import create_react_agent
from langchain_openai import ChatOpenAI

# 定義工具
tools = [search_web, calculate]

# 建立 LLM
llm = ChatOpenAI(model="gpt-4o-mini")

# 一行建立 ReAct Agent
agent = create_react_agent(llm, tools)

# 執行
result = agent.invoke({
    "messages": [("user", "台北今天天氣如何？")]
})
```

### 8.3 自訂 ReAct Agent

```python
from langgraph.graph import StateGraph, START, END
from langgraph.prebuilt import ToolNode
from langchain_core.messages import SystemMessage

class AgentState(TypedDict):
    messages: Annotated[list, add_messages]
    iteration: int

def create_custom_react_agent(llm, tools, system_prompt=""):
    
    def agent_node(state: AgentState):
        messages = state["messages"]
        if system_prompt:
            messages = [SystemMessage(content=system_prompt)] + messages
        
        llm_with_tools = llm.bind_tools(tools)
        response = llm_with_tools.invoke(messages)
        
        return {
            "messages": [response],
            "iteration": state.get("iteration", 0) + 1
        }
    
    def should_continue(state) -> str:
        # 防止無限循環
        if state.get("iteration", 0) >= 10:
            return END
            
        last_message = state["messages"][-1]
        if last_message.tool_calls:
            return "tools"
        return END
    
    # 建構圖
    graph = StateGraph(AgentState)
    graph.add_node("agent", agent_node)
    graph.add_node("tools", ToolNode(tools))
    
    graph.add_edge(START, "agent")
    graph.add_conditional_edges("agent", should_continue)
    graph.add_edge("tools", "agent")
    
    return graph.compile()
```

### 📌 學習重點
- 理解 ReAct 的推理-行動循環
- 使用 `create_react_agent` 快速建立 Agent
- 學會自訂 Agent 行為與限制

---

## 9. 多 Agent 系統

### 9.1 Supervisor 模式

```python
from typing import Literal

class MultiAgentState(TypedDict):
    messages: Annotated[list, add_messages]
    next_agent: str

def supervisor(state) -> dict:
    """主管節點：決定下一個執行的 Agent"""
    llm = ChatOpenAI(model="gpt-4o-mini")
    
    system_prompt = """你是一個團隊主管。根據任務內容，決定應該交給哪個成員：
    - researcher: 需要搜尋資訊
    - coder: 需要寫程式
    - writer: 需要撰寫內容
    - FINISH: 任務完成
    """
    
    response = llm.invoke([
        {"role": "system", "content": system_prompt},
        *state["messages"]
    ])
    
    return {"next_agent": response.content}

def route_to_agent(state) -> Literal["researcher", "coder", "writer", "end"]:
    next_agent = state.get("next_agent", "").lower()
    if "finish" in next_agent:
        return "end"
    return next_agent

# 建構多 Agent 圖
graph = StateGraph(MultiAgentState)
graph.add_node("supervisor", supervisor)
graph.add_node("researcher", researcher_agent)
graph.add_node("coder", coder_agent)
graph.add_node("writer", writer_agent)

graph.add_edge(START, "supervisor")
graph.add_conditional_edges("supervisor", route_to_agent)

# 每個 agent 執行完回到 supervisor
for agent in ["researcher", "coder", "writer"]:
    graph.add_edge(agent, "supervisor")
```

### 9.2 協作模式

```
         ┌──────────────────┐
         │    Supervisor    │
         └────────┬─────────┘
                  │
    ┌─────────────┼─────────────┐
    │             │             │
    ▼             ▼             ▼
┌───────┐   ┌───────┐   ┌───────┐
│Researcher│ │ Coder │   │Writer│
└───────┘   └───────┘   └───────┘
```

### 📌 學習重點
- 設計 Agent 職責分工
- 實作 Supervisor 協調邏輯
- 處理 Agent 間的訊息傳遞

---

## 10. 記憶體與持久化

### 10.1 Checkpointer 基礎

```python
from langgraph.checkpoint.memory import MemorySaver

# 建立記憶體 checkpointer
memory = MemorySaver()

# 編譯時加入 checkpointer
app = graph.compile(checkpointer=memory)

# 使用 thread_id 區分對話
config = {"configurable": {"thread_id": "user-123"}}

# 第一次對話
result1 = app.invoke(
    {"messages": [("user", "我叫小明")]},
    config
)

# 第二次對話（保持上下文）
result2 = app.invoke(
    {"messages": [("user", "我叫什麼名字？")]},
    config
)
```

### 10.2 生產環境持久化

```python
# 使用 PostgreSQL 持久化
from langgraph.checkpoint.postgres import PostgresSaver

# 連接資料庫
connection_string = "postgresql://user:pass@localhost:5432/langgraph"
postgres_saver = PostgresSaver.from_conn_string(connection_string)

app = graph.compile(checkpointer=postgres_saver)
```

### 10.3 對話歷史管理

```python
def manage_memory(state):
    """管理對話記憶，避免 token 超限"""
    messages = state["messages"]
    
    # 策略一：保留最近 N 條
    if len(messages) > 20:
        messages = messages[-20:]
    
    # 策略二：摘要舊訊息
    # ...
    
    return {"messages": messages}
```

### 📌 學習重點
- 使用 Checkpointer 實現持久化
- 理解 `thread_id` 的作用
- 設計記憶體管理策略

---

## 11. Human-in-the-Loop 人機協作

### 11.1 中斷點設定

```python
from langgraph.graph import StateGraph

# 編譯時指定中斷節點
app = graph.compile(
    checkpointer=memory,
    interrupt_before=["sensitive_action"]  # 執行前中斷
    # 或 interrupt_after=["review_step"]   # 執行後中斷
)
```

### 11.2 處理中斷與恢復

```python
# 執行到中斷點
config = {"configurable": {"thread_id": "task-1"}}
result = app.invoke({"task": "刪除所有資料"}, config)

# 此時程式暫停，等待人工確認
print("待確認操作：", result["pending_action"])

# 人工審核後繼續執行
user_approval = input("是否確認？(y/n): ")

if user_approval == "y":
    # 繼續執行
    final_result = app.invoke(None, config)
else:
    # 取消操作
    app.update_state(config, {"status": "cancelled"})
```

### 11.3 動態修改狀態

```python
# 取得當前狀態
current_state = app.get_state(config)

# 修改狀態
app.update_state(
    config,
    {"messages": [("human", "我想修改之前的指令")]}
)

# 從新狀態繼續
result = app.invoke(None, config)
```

### 📌 學習重點
- 設計人機協作的中斷點
- 實作審核與確認流程
- 處理狀態的動態修改

---

## 12. 錯誤處理與最佳實踐

### 12.1 錯誤處理模式

```python
def robust_node(state):
    """帶有錯誤處理的節點"""
    try:
        result = risky_operation(state)
        return {"result": result, "error": None}
    except TimeoutError:
        return {"result": None, "error": "操作超時"}
    except ValueError as e:
        return {"result": None, "error": f"參數錯誤: {e}"}
    except Exception as e:
        return {"result": None, "error": f"未知錯誤: {e}"}

def error_router(state):
    """根據錯誤決定下一步"""
    if state.get("error"):
        return "fallback"
    return "continue"
```

### 12.2 最佳實踐清單

| 類別 | 建議 |
|------|------|
| **狀態設計** | 保持最小化、使用型別標註、避免暫存值 |
| **圖結構** | 先規劃後實作、避免過度複雜 |
| **錯誤處理** | 每個節點都要 try/except、設計 fallback 路徑 |
| **循環控制** | 設定最大迭代次數、明確退出條件 |
| **效能優化** | 使用非同步節點、快取結果、減少 I/O |
| **測試** | 測試整個 Graph，而非單一函數 |
| **可觀測性** | 整合 LangSmith 進行追蹤與除錯 |

### 12.3 程式碼範本

```python
from langgraph.graph import StateGraph, START, END
from typing import TypedDict, Annotated, Literal
from langgraph.graph.message import add_messages
from langgraph.prebuilt import ToolNode
from langgraph.checkpoint.memory import MemorySaver

# 1. 明確的狀態定義
class AppState(TypedDict):
    messages: Annotated[list, add_messages]
    current_step: str
    error: str | None
    iteration: int

# 2. 帶有錯誤處理的節點
def safe_node(state: AppState) -> dict:
    try:
        # 業務邏輯
        return {"current_step": "completed", "error": None}
    except Exception as e:
        return {"error": str(e)}

# 3. 明確的路由邏輯
def router(state: AppState) -> Literal["next", "error", "end"]:
    if state.get("error"):
        return "error"
    if state.get("iteration", 0) >= 5:
        return "end"
    return "next"

# 4. 建構圖
graph = StateGraph(AppState)
graph.add_node("main", safe_node)
graph.add_node("error_handler", lambda s: {"error": None})

graph.add_edge(START, "main")
graph.add_conditional_edges("main", router)
graph.add_edge("error_handler", "main")

# 5. 帶有記憶體的編譯
app = graph.compile(checkpointer=MemorySaver())
```

### 📌 學習重點
- 實作 fallback 機制
- 遵循最佳實踐規範
- 建立可重用的程式模板

---

## 13. 進階主題與學習資源

### 13.1 進階主題

- **Streaming 串流輸出**：即時回傳生成內容
- **Subgraphs 子圖**：模組化複雜工作流程
- **動態圖生成**：根據條件動態建構圖
- **分散式執行**：跨節點執行大型 Agent 系統

### 13.2 官方資源

| 資源 | 連結 |
|------|------|
| 官方文件 | [https://langchain-ai.github.io/langgraph/](https://langchain-ai.github.io/langgraph/) |
| GitHub | [https://github.com/langchain-ai/langgraph](https://github.com/langchain-ai/langgraph) |
| LangSmith | [https://smith.langchain.com/](https://smith.langchain.com/) |
| LangGraph Studio | [https://studio.langchain.com/](https://studio.langchain.com/) |

### 13.3 推薦學習路徑

```
Week 1: 基礎概念
├── 環境設定
├── 理解 Graph 結構
└── 完成第一個應用

Week 2: 核心功能
├── 狀態管理深入
├── 條件路由實作
└── Tool 整合練習

Week 3: Agent 開發
├── ReAct Agent
├── 自訂 Agent 邏輯
└── 錯誤處理

Week 4: 進階應用
├── 多 Agent 系統
├── 持久化與記憶
└── Human-in-the-Loop
```

### 13.4 實作練習建議

1. **聊天機器人**：建立有記憶的對話 Agent
2. **搜尋助手**：整合搜尋 API 的 ReAct Agent
3. **程式碼助手**：能寫程式並執行的 Agent
4. **多步驟任務**：規劃-執行-驗證的工作流程
5. **團隊協作**：多 Agent 分工的專案管理系統

---

## 📝 附錄：常用程式碼片段

### A. 最小化 Agent 範本

```python
from langgraph.prebuilt import create_react_agent
from langchain_openai import ChatOpenAI
from langchain_core.tools import tool

@tool
def my_tool(input: str) -> str:
    """工具描述"""
    return f"處理: {input}"

agent = create_react_agent(
    ChatOpenAI(model="gpt-4o-mini"),
    [my_tool]
)

result = agent.invoke({"messages": [("user", "請幫我...")]})
```

### B. 帶記憶的對話範本

```python
from langgraph.checkpoint.memory import MemorySaver

app = graph.compile(checkpointer=MemorySaver())

config = {"configurable": {"thread_id": "session-1"}}
response = app.invoke({"messages": [("user", "你好")]}, config)
```

### C. 除錯技巧

```python
# 視覺化圖結構
from IPython.display import Image, display
display(Image(app.get_graph().draw_mermaid_png()))

# 逐步執行
for step in app.stream({"messages": inputs}):
    print(step)
```

---

> 💡 **提示**：建議邊學邊做，從簡單的範例開始，逐步增加複雜度。有問題時善用 LangSmith 進行追蹤和除錯！

**祝學習愉快！🎉**
