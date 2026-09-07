# CodeGraph 安裝與使用指南（供 AI Agent 閱讀）

> 本文件的目的是讓任何 AI 程式碼助手（opencode、Claude Code、Cursor 等）閱讀後，能直接理解 CodeGraph 是什麼、如何安裝、以及如何在任務中運用它。

---

## 一、CodeGraph 是什麼？

CodeGraph 是一個「程式碼知識圖譜（Code Knowledge Graph）」工具：

- 用 Rust 撰寫的核心引擎，掃描整個專案後建立「符號（symbol）、呼叫關係（call edges）、依賴（dependencies）」的圖譜，存入本機 SQLite。
- AI agent 不必逐檔 grep / glob / Read，只要一次查詢就能拿到：相關程式碼、呼叫路徑、影響範圍（blast radius）。
- 內建檔案監看器，程式碼一改，圖譜自動同步（預設 debounce 2 秒），索引永遠不會過期。
- 100% 本地執行，不需 API key，資料不外洩。
- 官方基準測試（7 個真實開源專案）：**少 88% 工具呼叫、快 53%、少 62% tokens、省 44% 成本**。

## 二、安裝前檢查

1. 確認作業系統（Windows / macOS / Linux）。
2. 檢查是否已安裝 CodeGraph CLI：
   ```powershell
   codegraph version
   ```
   有輸出代表已安裝（可跳過第三步，直接到第四步）。

## 三、安裝 CodeGraph CLI

### Windows（PowerShell）

```powershell
irm https://raw.githubusercontent.com/colbymchenry/codegraph/main/install.ps1 | iex
```

### macOS / Linux

```bash
curl -fsSL https://raw.githubusercontent.com/colbymchenry/codegraph/main/install.sh | sh
```

### 或使用 npm（任何系統）

```bash
npm i -g @colbymchenry/codegraph
```

> **注意**：安裝完後必須**開啟新的終端視窗**（或重啟 shell），`codegraph` 指令才會出現在 PATH。

### 升級

```powershell
codegraph upgrade
```

## 四、把 CodeGraph 接進 AI Agent

在**新的終端視窗**執行：

```powershell
codegraph install
```

- 會自動偵測已安裝的 agent（opencode、Claude Code、Cursor、Codex、Gemini CLI 等）。
- 把 CodeGraph 的 MCP server 設定寫進該 agent 的設定檔（opencode 會寫入 `opencode.json`）。
- 這一步**只負責接線，不會建立索引**；每個專案要另外執行 `codegraph init`。

執行完後，**重啟你的 AI agent**（opencode 需重啟才能載入 MCP server）。

> 若在無法互動的環境（CI / 自動化），可用非互動模式：
> ```powershell
> codegraph install --yes
> ```

## 五、為專案建立索引（每個專案一次）

```powershell
cd <專案資料夾>
codegraph init
```

- 會在專案根目錄建立 `.codegraph/` 資料夾並建立完整圖譜（大型專案可能花幾分鐘）。
- 之後**不需再手動同步**：檔案 watcher 自動更新圖譜。
- 想確認狀態：`codegraph status`；移除索引：`codegraph uninit`。

## 六、AI Agent 如何使用 CodeGraph

### 6.1 透過 MCP 工具（agent 內建）

當 `.codegraph/` 存在且 MCP server 已載入時，agent 會自動多出 `codegraph_explore` 工具：

| 工具 | 用途 |
|---|---|
| `codegraph_explore` | 一次回答幾乎所有結構性問題：「X 怎麼運作的？」「X 如何呼叫到 Y？」「這個區域的架構是什麼？」回傳相關符號的原始碼（依檔案分組）、呼叫路徑、影響範圍摘要。 |

**使用原則（重要）：**

- 回答結構性問題時**直接用 `codegraph_explore`**，不要先 grep/Read 一輪——那是重複工作。
- 查詢中可以指名檔案或符號，會回傳該檔案目前的行號原始碼（等同 Read 工具）。
- **信任結果，不要再用 grep 重新驗證。**
- 若 MCP 回應帶有 `⚠️` 過期旗標（表示某檔案剛被編輯、圖譜尚未同步），請直接 Read 該檔案拿最新內容。
- 每個專案要各自 `codegraph init` 後才有索引；無索引的資料夾會得到提示改用內建工具。

### 6.2 透過 CLI（無 MCP 環境，例如 subagent）

| 指令 | 用途 |
|---|---|
| `codegraph explore <查詢>` | 等同 `codegraph_explore`：一次取得相關原始碼＋呼叫路徑＋影響範圍 |
| `codegraph node <符號或檔案>` | 單一符號的原始碼與呼叫者；或讀取檔案（含行號） |
| `codegraph search <關鍵字>` | 全文搜尋符號（FTS5） |
| `codegraph callers <符號>` | 誰呼叫了這個函式 |
| `codegraph callees <符號>` | 這個函式呼叫了誰 |
| `codegraph impact <符號>` | 修改某符號會影響哪些程式碼 |
| `codegraph files` | 顯示檔案結構 |
| `codegraph affected <檔案...>` | 找出受影響的測試檔（可接 `git diff`） |
| `codegraph status` | 索引狀態、是否有待同步檔案 |

## 七、常見問題（Troubleshooting）

| 問題 | 解決方式 |
|---|---|
| `codegraph` 不是可辨識的指令 | 安裝後開新的終端視窗；或確認 PATH |
| agent 沒有 `codegraph_explore` 工具 | 確認已執行 `codegraph install` 並**重啟 agent** |
| 查詢說「此專案無索引」 | 在該專案執行 `codegraph init` |
| 索引卡住（stale lock） | `codegraph unlock` |
| 想停用遙測 | `codegraph telemetry off` |
| 移除所有設定 | `codegraph uninstall`（保留 CLI：`--keep-cli`） |

## 八、給 AI Agent 的建議工作流程

1. 接到任務後，若工作目錄（或相關專案）存在 `.codegraph/`，優先使用 `codegraph_explore` 收集上下文。
2. 若不存在：先問使用者是否要執行 `codegraph init` 建立索引（大型專案建立後整體效率顯著提升）。
3. 改動程式前，用 `codegraph impact` 評估影響範圍；改完後等 watcher 自動同步（約 2 秒）。
4. 需要找測試時，用 `codegraph affected` 從 git 變更推導受影響的測試檔。

---

*本文檔由 opencode 依 CodeGraph 官方 README（github.com/colbymchenry/codegraph）整理產生。*
