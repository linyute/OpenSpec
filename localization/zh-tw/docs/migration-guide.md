# 遷移到 OPSX

本指南協助您從舊版 OpenSpec 工作流程轉換到 OPSX。遷移過程設計得非常流暢 — 您現有的工作將被保留，且新系統提供了更大的靈活性。

## 有什麼變更？

OPSX 取代了舊有的階段鎖定工作流程，改用流動的、基於動作的方法。以下是關鍵的轉變：

| 面向         | 舊版                                                         | OPSX                                            |
| ------------ | ------------------------------------------------------------ | ----------------------------------------------- |
| **指令**     | `/openspec:proposal`, `/openspec:apply`, `/openspec:archive` | `/opsx:new`, `/opsx:continue`, `/opsx:apply` 等 |
| **工作流程** | 一次建立所有成品                                             | 增量建立或一次建立 — 由您選擇                   |
| **回退**     | 尷尬的階段門檻                                               | 自然 — 隨時更新任何成品                         |
| **自訂**     | 固定結構                                                     | Schema 驅動，完全可修改                         |
| **設定**     | `CLAUDE.md` 帶有標記 + `project.md`                          | `openspec/config.yaml` 中乾淨的設定             |

**理念的轉變：** 工作不是線性的。OPSX 不再假裝它是線性的。

---

## 在您開始之前

### 您現有的工作是安全的

遷移過程在設計時就考慮到了保留：

- **`openspec/changes/` 中的作用中變更** — 完全保留。您可以使用 OPSX 指令繼續這些變更。
- **已封存的變更** — 不受影響。您的歷史記錄保持完好。
- **`openspec/specs/` 中的主規格** — 不受影響。這些是您的單一事實來源。
- **您在 CLAUDE.md、AGENTS.md 等檔案中的內容** — 已保留。僅移除 OpenSpec 標記區塊；您撰寫的所有內容都會保留。

### 哪些會被移除

僅會移除由 OpenSpec 管理且正在被替換的檔案：

| 項目                                              | 原因                         |
| ------------------------------------------------- | ---------------------------- |
| 舊版斜線指令目錄/檔案                             | 由新的技能 (skills) 系統取代 |
| `openspec/AGENTS.md`                              | 已過時的工作流程觸發器       |
| `CLAUDE.md`、`AGENTS.md` 等檔案中的 OpenSpec 標記 | 不再需要                     |

**按工具分類的舊版指令位置**（範例 — 您的工具可能有所不同）：

- Claude Code：`.claude/commands/openspec/`
- Cursor：`.cursor/commands/openspec-*.md`
- Windsurf：`.windsurf/workflows/openspec-*.md`
- Cline：`.clinerules/workflows/openspec-*.md`
- Roo：`.roo/commands/openspec-*.md`
 - GitHub Copilot：`.github/prompts/openspec-*.prompt.md`（僅限 IDE 擴充功能；不支援 Copilot CLI）
- 以及其他 (Augment, Continue, Amazon Q 等)

遷移程式會偵測您已設定的工具並清理其舊版檔案。

移除列表看起來可能很長，但這些都是 OpenSpec 最初建立的檔案。您自己的內容絕不會被刪除。

### 哪些需要您的注意

有一個檔案需要手動遷移：

**`openspec/project.md`** — 此檔案不會自動刪除，因為它可能包含您撰寫的專案背景資訊。您需要：

1. 檢視其內容
2. 將有用的背景資訊移至 `openspec/config.yaml`（請參閱下方的指導）
3. 準備就緒後刪除該檔案

**為什麼我們做出這項變更：**

舊的 `project.md` 是被動的 — 代理可能會讀它，可能不會，也可能會忘記讀過的內容。我們發現其可靠性不一致。

新的 `config.yaml` 背景資訊會 **主動注入到每個 OpenSpec 規劃請求中**。這意味著當 AI 建立成品時，您的專案慣例、技術堆疊和規則始終存在。可靠性更高。

**折衷方案：**

因為背景資訊會注入到每個請求中，所以您需要保持簡潔。專注於真正重要的事情：
- 技術堆疊和關鍵慣例
- AI 需要知道的非顯而易見的約束
- 以前經常被忽略的規則

不要擔心一次就做到完美。我們仍在學習這裡什麼最有效，並且隨著實驗的進行，我們將改進背景資訊注入的工作方式。

---

## 執行遷移

`openspec init` 和 `openspec update` 都會偵測舊版檔案並引導您完成相同的清理程序。請使用適合您情況的指令：

### 使用 `openspec init`

如果您想新增工具或重新設定已設定的工具，請執行此指令：

```bash
openspec init
```

init 指令會偵測舊版檔案並引導您完成清理：

```
正在升級到新的 OpenSpec

OpenSpec 現在使用代理技能 (agent skills)，這是編碼代理中新興的標準。
這簡化了您的設定，同時保持一切像以前一樣正常運作。

要移除的檔案
無須保留的使用者內容：
  • .claude/commands/openspec/
  • openspec/AGENTS.md

要更新的檔案
OpenSpec 標記將被移除，您的內容將被保留：
  • CLAUDE.md
  • AGENTS.md

需要您的注意
  • openspec/project.md
    我們不會刪除此檔案。它可能包含有用的專案背景資訊。

    新的 openspec/config.yaml 有一個用於規劃背景資訊的 "context:" 章節。
    這包含在每個 OpenSpec 請求中，並且比舊的 project.md 方法更可靠。

    檢視 project.md，將任何有用的內容移至 config.yaml 的 context 章節，
    然後在準備就緒時刪除該檔案。

? 升級並清理舊版檔案？ (Y/n)
```

**當您選擇「是 (yes)」時會發生什麼：**

1. 移除舊版斜線指令目錄
2. 從 `CLAUDE.md`、`AGENTS.md` 等檔案中剝離 OpenSpec 標記（您的內容保留）
3. 刪除 `openspec/AGENTS.md`
4. 在 `.claude/skills/` 中安裝新技能
5. 建立具有預設 schema 的 `openspec/config.yaml`

### 使用 `openspec update`

如果您只想遷移並將現有工具重新整理到最新版本，請執行此指令：

```bash
openspec update
```

update 指令也會偵測並清理舊版成品，然後將您的技能重新整理到最新版本。

### 非互動式 / CI 環境

對於腳本化遷移：

```bash
openspec init --force --tools claude
```

`--force` 旗標會跳過提示並自動接受清理。

---

## 將 project.md 遷移到 config.yaml

舊的 `openspec/project.md` 是一個用於專案背景資訊的自由格式 Markdown 檔案。新的 `openspec/config.yaml` 是結構化的，並且關鍵在於，它會 **注入到每個規劃請求中**，因此當 AI 工作時，您的慣例始終存在。

### 之前 (project.md)

```markdown
# 專案背景資訊

這是一個使用 React 和 Node.js 的 TypeScript monorepo。
我們使用 Jest 進行測試並遵循嚴格的 ESLint 規則。
我們的 API 是 RESTful，記錄在 docs/api.md 中。

## 慣例

- 所有公共 API 必須保持向後相容性
- 新功能應包含測試
- 規格使用 Given/When/Then 格式
```

### 之後 (config.yaml)

```yaml
schema: spec-driven

context: |
  技術堆疊：TypeScript, React, Node.js
  測試：使用 React Testing Library 的 Jest
  API：RESTful，記錄在 docs/api.md 中
  我們為所有公共 API 保持向後相容性

rules:
  proposal:
    - 為高風險變更包含回滾計畫
  specs:
    - 情境使用 Given/When/Then 格式
    - 在發明新模式之前參考現有模式
  design:
    - 為複雜流程包含循序圖
```

### 關鍵差異

| project.md        | config.yaml                                        |
| ----------------- | -------------------------------------------------- |
| 自由格式 Markdown | 結構化 YAML                                        |
| 一大塊文字        | 分離背景資訊和每個成品的規則                       |
| 不清楚何時使用    | 背景資訊出現在所有成品中；規則僅出現在相符的成品中 |
| 無 schema 選擇    | 明確的 `schema:` 欄位設定預設工作流程              |

### 該保留什麼，該捨棄什麼

遷移時，請有所選擇。問問自己：「AI 在 *每個* 規劃請求中都需要這個嗎？」

**適合放進 `context:` 的候選內容：**
- 技術堆疊（語言、框架、資料庫）
- 關鍵架構模式（monorepo、微服務等）
- 非顯而易見的約束（「我們不能使用程式庫 X，因為...」）
- 經常被忽略的關鍵慣例

**改為移至 `rules:`：**
- 特定成品的格式（「在規格中使用 Given/When/Then」）
- 檢閱準則（「提案必須包含回滾計畫」）
- 這些僅針對相符的成品出現，保持其他請求更輕量

**完全忽略：**
- AI 已經知道的一般最佳實踐
- 可以總結的冗長說明
- 不影響目前工作的歷史背景

### 遷移步驟

1. **建立 config.yaml**（如果 init 尚未建立）：
   ```yaml
   schema: spec-driven
   ```

2. **新增您的背景資訊**（保持簡潔 — 這會進入每個請求）：
   ```yaml
   context: |
     在此處輸入您的專案背景。
     專注於 AI 真正需要知道的內容。
   ```

3. **新增每個成品的規則**（選用）：
   ```yaml
   rules:
     proposal:
       - 您的提案特定指南
     specs:
       - 您的規格撰寫規則
   ```

4. 搬移所有有用的內容後，**刪除 project.md**。

**不要過度思考。** 從精華開始並不斷疊代。如果您發現 AI 遺漏了重要的內容，請將其新增。如果背景資訊感覺過於臃腫，請將其修剪。這是一個動態的文件。

### 需要協助嗎？使用此提示

如果您不確定如何精煉您的 project.md，請詢問您的 AI 助理：

```
我正在從 OpenSpec 的舊版 project.md 遷移到新的 config.yaml 格式。

這是我目前的 project.md：
[在此貼上您的 project.md 內容]

請協助我建立一個具有以下內容的 config.yaml：
1. 簡潔的 `context:` 章節（這會注入到每個規劃請求中，因此請保持精簡 — 專注於技術堆疊、關鍵約束和經常被忽略的慣例）
2. 針對特定成品的 `rules:`（如果任何內容是針對特定成品的，例如 "使用 Given/When/Then" 屬於規格規則，而非全域背景資訊）

捨棄 AI 模型已經知道的任何通用內容。對簡潔性要冷酷無情。
```

AI 會協助您識別哪些是精華，哪些可以修剪。

---

## 新指令

遷移後，您將擁有 9 個 OPSX 指令，而不是 3 個：

| 指令                 | 用途                                      |
| -------------------- | ----------------------------------------- |
| `/opsx:explore`      | 在無結構的情況下思考想法                  |
| `/opsx:new`          | 開始一個新的變更                          |
| `/opsx:continue`     | 建立下一個成品（一次一個）                |
| `/opsx:ff`           | 快進 — 一次建立所有規劃成品               |
| `/opsx:apply`        | 實作來自 tasks.md 的任務                  |
| `/opsx:verify`       | 驗證實作是否符合規格                      |
| `/opsx:sync`         | 預覽規格合併（選用 — 封存會在需要時提示） |
| `/opsx:archive`      | 完成並封存變更                            |
| `/opsx:bulk-archive` | 一次封存多個變更                          |

### 舊版指令映射

| 舊版                 | OPSX 對等指令               |
| -------------------- | --------------------------- |
| `/openspec:proposal` | `/opsx:new` 然後 `/opsx:ff` |
| `/openspec:apply`    | `/opsx:apply`               |
| `/openspec:archive`  | `/opsx:archive`             |

### 新功能

**細粒度成品建立：**
```
/opsx:continue
```
根據相依性一次建立一個成品。當您想檢閱每個步驟時，請使用此功能。

**探索模式：**
```
/opsx:explore
```
在投入變更之前與夥伴一起思考想法。

---

## 瞭解新架構

### 從階段鎖定到流動

舊版工作流程強制執行線性進展：

```
┌──────────────┐      ┌──────────────┐      ┌──────────────┐
│   規劃階段    │ ───► │   實作階段    │ ───► │   封存階段    │
└──────────────┘      └──────────────┘      └──────────────┘

如果您正在實作並意識到設計錯了？
太遺憾了。階段門檻不允許您輕易回頭。
```

OPSX 使用動作，而非階段：

```
         ┌────────────────────────────────────────┐
         │           動作 (而非階段)                │
         │                                        │
         │     new ◄──► continue ◄──► apply ◄──► archive │
         │      │          │           │           │   │
         │      └──────────┴───────────┴───────────┘   │
         │              任何順序                        │
         └────────────────────────────────────────┘
```

### 相依圖

成品形成一個有向圖。相依性是推動因素，而非門檻：

```
                        proposal
                       (根節點)
                            │
              ┌─────────────┴─────────────┐
              │                           │
              ▼                           ▼
           specs                       design
        (需要：                     (需要：
         proposal)                   proposal)
              │                           │
              └─────────────┬─────────────┘
                            │
                            ▼
                         tasks
                     (需要：
                     specs, design)
```

當您執行 `/opsx:continue` 時，它會檢查哪些已就緒並提供下一個成品。您也可以按任何順序建立多個已就緒的成品。

### 技能 (Skills) vs 指令

舊版系統使用工具特定的指令檔案：

```
.claude/commands/openspec/
├── proposal.md
├── apply.md
└── archive.md
```

OPSX 使用新興的 **技能 (skills)** 標準：

```
.claude/skills/
├── openspec-explore/SKILL.md
├── openspec-new-change/SKILL.md
├── openspec-continue-change/SKILL.md
├── openspec-apply-change/SKILL.md
└── ...
```

技能可跨多個 AI 編碼工具被辨識，並提供更豐富的 Metadata。

---

## 繼續現有變更

您正在進行的變更可與 OPSX 指令無縫運作。

**有來自舊版工作流程的作用中變更嗎？**

```
/opsx:apply add-my-feature
```

OPSX 會讀取現有成品並從您停下的地方繼續。

**想向現有變更新增更多成品嗎？**

```
/opsx:continue add-my-feature
```

根據已存在的內容顯示準備好建立的成品。

**需要查看狀態嗎？**

```bash
openspec status --change add-my-feature
```

---

## 新的設定系統

### config.yaml 結構

```yaml
# 必填：新變更的預設 schema
schema: spec-driven

# 選用：專案背景資訊 (最大 50KB)
# 注入到所有成品指示中
context: |
  您的專案背景、技術堆疊、
  慣例和約束。

# 選用：針對每個成品的規則
# 僅注入到相符的成品中
rules:
  proposal:
    - 包含回滾計畫
  specs:
    - 使用 Given/When/Then 格式
  design:
    - 記錄備援策略
  tasks:
    - 分解為最多 2 小時的區塊
```

### Schema 解析

在確定使用哪個 schema 時，OPSX 會依序檢查：

1. **CLI 旗標**：`--schema <name>`（最高優先順序）
2. **變更 Metadata**：變更目錄中的 `.openspec.yaml`
3. **專案設定**：`openspec/config.yaml`
4. **預設值**：`spec-driven`

### 可用的 Schema

| Schema        | 成品                                 | 最適合       |
| ------------- | ------------------------------------ | ------------ |
| `spec-driven` | proposal → specs → design → tasks    | 大多數專案   |

列出所有可用的 schema：

```bash
openspec schemas
```

### 自訂 Schema

建立您自己的工作流程：

```bash
openspec schema init my-workflow
```

或分支現有的 schema：

```bash
openspec schema fork spec-driven my-workflow
```

詳情請參閱 [自訂](customization.md)。

---

## 疑難排解

### 「在非互動模式下偵測到舊版檔案」

您正在 CI 或非互動環境中執行。請使用：

```bash
openspec init --force
```

### 遷移後指令未出現

重新啟動您的 IDE。技能會在啟動時被偵測。

### 「規則中不明的成品 ID」

檢查您的 `rules:` 鍵是否符合您的 schema 的成品 ID：

- **spec-driven**: `proposal`, `specs`, `design`, `tasks`

執行此指令以查看有效的成品 ID：

```bash
openspec schemas --json
```

### 設定未生效

1. 確保檔案位於 `openspec/config.yaml`（不是 `.yml`）
2. 驗證 YAML 語法
3. 設定變更會立即生效 — 無需重新啟動

### project.md 未遷移

系統刻意保留 `project.md`，因為它可能包含您的自訂內容。請手動檢閱，將有用的部分移至 `config.yaml`，然後刪除它。

### 想要查看哪些內容會被清理？

執行 init 並拒絕清理提示 — 您將看到完整的偵測摘要，而不會進行任何變更。

---

## 快速參考

### 遷移後的檔案

```
project/
├── openspec/
│   ├── specs/                    # 未變更
│   ├── changes/                  # 未變更
│   │   └── archive/              # 未變更
│   └── config.yaml               # 新增：專案設定
├── .claude/
│   └── skills/                   # 新增：OPSX 技能
│       ├── openspec-explore/
│       ├── openspec-new-change/
│       └── ...
├── CLAUDE.md                     # 已移除 OpenSpec 標記，您的內容已保留
└── AGENTS.md                     # 已移除 OpenSpec 標記，您的內容已保留
```

### 哪些消失了

- `.claude/commands/openspec/` — 由 `.claude/skills/` 取代
- `openspec/AGENTS.md` — 已過時
- `openspec/project.md` — 遷移到 `config.yaml` 後刪除
- `CLAUDE.md`、`AGENTS.md` 等檔案中的 OpenSpec 標記區塊

### 指令速查表

```
/opsx:new          開始一個變更
/opsx:continue     建立下一個成品
/opsx:ff           建立所有規劃成品
/opsx:apply        實作任務
/opsx:archive      完成並封存
```

---

## 獲取協助

- **Discord**: [discord.gg/YctCnvvshC](https://discord.gg/YctCnvvshC)
- **GitHub Issues**: [github.com/Fission-AI/OpenSpec/issues](https://github.com/Fission-AI/OpenSpec/issues)
- **文件**: [docs/opsx.md](opsx.md) 獲取完整的 OPSX 參考資訊
