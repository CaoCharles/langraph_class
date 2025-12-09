# 🚀 LangGraph 學習指南

一個使用 MkDocs + Jupyter Notebook 建構的互動式 LangGraph 學習平台。

## 📋 專案內容

- 📖 完整的 LangGraph 學習指南（13 個章節）
- 💻 5 個互動式 Jupyter Notebook 實作練習
- 🎨 使用 Material 主題的美觀文件網站

## 🛠️ 環境需求

- Python 3.12+
- [uv](https://docs.astral.sh/uv/) 套件管理器

## 🚀 快速開始

### 1. 設定環境變數

```bash
# 複製範例檔案
cp .env.example .env

# 編輯 .env 檔案，填入您的 API Key
```

### 2. 啟動 Jupyter Lab

```bash
uv run jupyter lab
```

### 3. 啟動 MkDocs 開發伺服器

```bash
uv run mkdocs serve
```

開啟瀏覽器訪問 http://127.0.0.1:8000

### 4. 建置靜態網站

```bash
uv run mkdocs build
```

## 📚 課程大綱

| 章節 | 主題 |
|------|------|
| 1-3 | 基礎概念與第一個應用 |
| 4-6 | 狀態管理與條件路由 |
| 7-8 | Tool 整合與 ReAct Agent |
| 9-11 | 多 Agent 與人機協作 |
| 12-13 | 最佳實踐與進階主題 |

## 📁 專案結構

```
langraph_class/
├── docs/                    # MkDocs 文件目錄
│   ├── index.md            # 首頁
│   ├── LangGraph_學習指南.md # 完整學習指南
│   └── notebooks/          # Jupyter Notebook 實作
│       ├── 01_setup.ipynb
│       ├── 02_first_graph.ipynb
│       ├── 03_state_management.ipynb
│       ├── 04_react_agent.ipynb
│       └── 05_tools.ipynb
├── mkdocs.yml              # MkDocs 設定
├── pyproject.toml          # 專案設定
├── .env.example            # 環境變數範例
└── README.md               # 本檔案
```

## 📖 學習資源

- [LangGraph 官方文件](https://langchain-ai.github.io/langgraph/)
- [LangChain 官方文件](https://python.langchain.com/)
- [LangSmith](https://smith.langchain.com/)

## 📝 License

MIT License
