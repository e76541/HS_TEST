# Plan · Workflow · Subagent

Grok 裡這三層**不一樣**，但常一起用。

```text
PLAN  →  WORKFLOW  →  SUBAGENT(s)
```

| 層級 | 一句話 | 像什麼 |
|------|--------|--------|
| **Plan** | 要做什麼、步驟與驗收 | 藍圖 / 設計稿 |
| **Workflow** | 怎麼編排多個 agent 去執行 | 自動化流水線 / SOP 腳本 |
| **Subagent** | 實際被派出來做事的單位 | 工人 / 子 session |

**Checklist** 不是獨立一層產品功能，而是 Plan 或 Workflow 裡的檢查項目。

---

## 1. Plan（規劃）

**是什麼：** 先想清楚目標、步驟、風險、完成條件。  
**不是：** 不會自動開一群 agent 跑完整 pipeline（那是 Workflow）。

**在 Grok 裡常見形態：**

- Plan mode（先規劃再動手）
- `plan` 類型子代理（探索 codebase、產出實作計畫，通常不改檔）
- 設計文件 / 實作計畫文字

**產出範例：** 要改哪些模組、順序、驗收標準、風險。

---

## 2. Workflow（流程編排）

**是什麼：** 可執行的多 agent 編排腳本（`.rhai`），真正調度子代理。  
**不是：** 靜態規劃表或待辦清單本身。

**會做的事：**

- `agent()` — 開一個子代理
- `parallel()` — 同時開多個
- `phase()` — 分階段（例如 Review → Verify）
- `complete()` — 結束並回傳結果

**存放位置：**

| 範圍 | 路徑 |
|------|------|
| 專案 | `<repo>/.grok/workflows/<name>.rhai` |
| 使用者 | `~/.grok/workflows/<name>.rhai` |

**常用指令：**

```text
/workflow <name> {...args...}
/workflow pause <display-name>
/workflow resume <display-name>
/workflow stop <display-name>
/workflows
/deep-research <query>
```

- `/workflows` = **執行中的 run 儀表板**（不是已存 workflow 目錄清單）
- 內建範例：`/deep-research` 會在背景跑研究 workflow

**預算（簡記）：** 預設每個 run 約 128 次 logical agent 呼叫；同時約最多 16 個子 agent。

---

## 3. Subagent（子代理）

**是什麼：** 主 agent（或 Workflow）派出去的**獨立子 session**。  
**特點：** 自己的 context；做完把摘要/結果交回。

**內建類型：**

| Type | 用途 |
|------|------|
| `general-purpose` | 通用，可做各種任務 |
| `explore` | 搜尋 / 讀檔 / 調查，通常不改檔 |
| `plan` | 產出實作計畫，通常不改檔 |

主 agent 也可直接 `spawn_subagent`，**不必**一定先有 Workflow。

**開關：** 預設開啟。可關：

```toml
# ~/.grok/config.toml
[subagents]
enabled = false
```

或環境變數 `GROK_SUBAGENTS=0`。

---

## 4. 三者關係（重點）

```text
┌─────────────┐
│    PLAN     │  藍圖：要做什麼
└──────┬──────┘
       │ 決定步驟與驗收
       ▼
┌─────────────┐
│  WORKFLOW   │  流水線：怎麼派工、並行、驗證
└──────┬──────┘
       │ agent() / parallel()
       ▼
┌─────────────┐
│  SUBAGENT   │  工人：各自執行一個工作單元
│  SUBAGENT   │
│  SUBAGENT   │
└─────────────┘
```

| 說法 | 對不對 |
|------|--------|
| Plan = Workflow | 錯。Plan 是規劃；Workflow 是會跑的編排。 |
| Workflow = Subagent | 錯。Workflow 會**用**很多 Subagent。 |
| 一定要有 Workflow 才能用 Subagent | 錯。主 agent 可直接派 Subagent。 |
| Checklist = Grok 的一層系統 | 錯。只是清單概念，掛在 Plan/Workflow 步驟裡。 |

---

## 5. 類比（好記）

| Grok | 現實 |
|------|------|
| Plan | 建築藍圖 |
| Workflow | 工地流水線 / 施工 SOP（會真的開工） |
| Subagent | 水電工、泥作、品管各自開工 |
| Checklist | 驗收勾選表（掛在藍圖或 SOP 裡） |

---

## 6. 一句話總結

> **Plan 想清楚 → Workflow 自動編排 → Subagent 實際執行。**

- **Plan** = 藍圖  
- **Workflow** = 可執行的多 agent 流水線  
- **Subagent** = 被派出來做事的子 session  

---

## 相關 Grok 說明

- Slash：`/workflow`、`/workflows`、`/deep-research`、`/plan`
- 文件：`~/.grok/docs/user-guide/04-slash-commands.md`、`16-subagents.md`
- 建立 workflow skill：`create-workflow`（`/create-workflow`）
