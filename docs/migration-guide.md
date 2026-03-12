# 遷移至 OPSX (Migrating to OPSX)

本指南協助您從舊版的 OpenSpec 工作流過渡到 OPSX。遷移過程旨在保持順暢 — 您的現有工作將被保留，且新系統提供了更大的靈活性。

## 正在變更的內容

OPSX 以流動的、基於行動的方法取代了舊有的階段鎖定式工作流。以下是主要的轉變：

| 面向         | 舊版                                                         | OPSX                                                                         |
| ------------ | ------------------------------------------------------------ | ---------------------------------------------------------------------------- |
| **命令**     | `/openspec:proposal`, `/openspec:apply`, `/openspec:archive` | 預設：`/opsx:propose`, `/opsx:apply`, `/opsx:archive` (擴展工作流命令為選用) |
| **工作流**   | 一次建立所有成品                                             | 逐步建立或一次建立所有 — 由您決定                                            |
| **回頭修改** | 彆扭的階段閘門                                               | 自然 — 隨時更新任何成品                                                      |
| **自訂**     | 固定結構                                                     | 結構描述驅動，完全可自訂 (hackable)                                          |
| **設定**     | 帶有標記的 `CLAUDE.md` + `project.md`                        | `openspec/config.yaml` 中的整潔設定                                          |

**理念的轉變：** 工作並非線性的。OPSX 不再假裝它是。

---

## 在您開始之前

### 您的現有工作是安全的

遷移程序的設計考慮到了保留現有成果：

- **`openspec/changes/` 中的作用中變更** — 完全保留。您可以使用 OPSX 命令繼續進行。
- **已封存的變更** — 不受影響。您的歷史記錄將完整保留。
- **`openspec/specs/` 中的主規格** — 不受影響。這些是您的真相來源。
- **您在 CLAUDE.md, AGENTS.md 等檔案中的內容** — 保留。僅移除 OpenSpec 標記區塊；您撰寫的所有內容都會保留。

### 會被移除的內容

僅移除將被取代的、由 OpenSpec 管理的檔案：

| 內容                                              | 原因                         |
| ------------------------------------------------- | ---------------------------- |
| 舊版斜線命令目錄/檔案                             | 由新的技能 (skills) 系統取代 |
| `openspec/AGENTS.md`                              | 已過時的工作流觸發器         |
| `CLAUDE.md`, `AGENTS.md` 等檔案中的 OpenSpec 標記 | 不再需要                     |

**按工具分類的舊版命令位置** (範例 — 您的工具可能有所不同)：

- Claude Code: `.claude/commands/openspec/`
- Cursor: `.cursor/commands/openspec-*.md`
- Windsurf: `.windsurf/workflows/openspec-*.md`
- Cline: `.clinerules/workflows/openspec-*.md`
- Roo: `.roo/commands/openspec-*.md`
- GitHub Copilot: `.github/prompts/openspec-*.prompt.md` (僅限 IDE 擴充功能；Copilot CLI 未支援)
- 其他 (Augment, Continue, Amazon Q 等)

遷移程序會偵測您已設定的工具，並清理其舊版檔案。

移除清單可能看起來很長，但這些都是 OpenSpec 最初建立的檔案。您的個人內容絕不會被刪除。

### 需要您留意的事項

有一個檔案需要手動遷移：

**`openspec/project.md`** — 此檔案不會自動刪除，因為它可能包含您撰寫的專案內容 (context)。您需要執行以下操作：

1. 檢閱其內容
2. 將有用的內容移動至 `openspec/config.yaml` (請參閱下方的指引)
3. 準備就緒後刪除該檔案

**為什麼我們要做這項變更：**

舊有的 `project.md` 是被動的 — 代理程式可能會讀取，也可能不會，甚至讀過後會忘記。我們發現其可靠性並不一致。

新的 `config.yaml` 內容會**主動注入到每一個 OpenSpec 規劃請求中**。這意味著當 AI 正在建立成品時，您的專案慣例、技術棧和規則始終存在。可靠性更高。

**權衡 (The tradeoff)：**

因為內容會注入到每個請求中，所以您會希望內容保持簡潔。請專注於真正重要的事項：
- 技術棧和關鍵慣例
- AI 需要知道的非顯而易見的約束
- 以前經常被忽略的規則

不必擔心一開始就做到完美。我們仍在學習這裡最有效的做法，並將隨著實驗的進行改進內容注入的運作方式。

---

## 執行遷移

`openspec init` 和 `openspec update` 都會偵測舊版檔案，並引導您完成相同的清理程序。請根據您的情況選擇其一：

- 全新安裝預設使用 `core` 設定檔 (`propose`, `explore`, `apply`, `archive`)。
- 遷移後的安裝會透過在需要時寫入 `custom` 設定檔來保留您先前安裝的工作流。

### 使用 `openspec init`

如果您想要新增工具或重新設定已設定的工具，請執行此命令：

```bash
openspec init
```

init 命令會偵測舊版檔案並引導您完成清理：

```
正在升級至新版 OpenSpec

OpenSpec 現在使用代理程式技能 (agent skills)，這是各個編碼代理程式
之間新興的標準。這簡化了您的設定，同時讓一切保持如常運作。

要移除的檔案
無須保留的使用者內容：
  • .claude/commands/openspec/
  • openspec/AGENTS.md

要更新的檔案
OpenSpec 標記將被移除，您的內容將被保留：
  • CLAUDE.md
  • AGENTS.md

需要您留意
  • openspec/project.md
    我們不會刪除此檔案。它可能包含有用的專案內容。

    新的 openspec/config.yaml 有一個「context:」區段用於規劃內容。
    這會包含在每個 OpenSpec 請求中，比舊有的 project.md 方法更可靠。

    檢閱 project.md，將任何有用的內容移動到 config.yaml 的 context
    區段，然後在準備就緒後刪除該檔案。

? 升級並清理舊版檔案？ (Y/n)
```

**當您回答「是 (yes)」時會發生什麼：**

1. 舊版斜線命令目錄被移除
2. OpenSpec 標記從 `CLAUDE.md`, `AGENTS.md` 等檔案中被清除 (您的內容保留)
3. `openspec/AGENTS.md` 被刪除
4. 新技能安裝在 `.claude/skills/` 中
5. 使用預設結構描述建立 `openspec/config.yaml`

### 使用 `openspec update`

如果您只是想遷移並將現有工具重新整理為最新版本，請執行此命令：

```bash
openspec update
```

update 命令同樣會偵測並清理舊版成品，然後重新整理生成的技能/命令，以符合您目前的設定檔和遞送設定。

### 非互動式 / CI 環境

對於指令碼式的遷移：

```bash
openspec init --force --tools claude
```

`--force` 旗標會跳過提示並自動接受清理。

---

## 將 project.md 遷移至 config.yaml

舊有的 `openspec/project.md` 是用於專案內容 (context) 的自由格式 markdown 檔案。新的 `openspec/config.yaml` 是結構化的，並且關鍵在於它會**注入到每一個規劃請求中**，因此當 AI 工作時，您的慣例始終存在。

### 遷移前 (project.md)

```markdown
# 專案內容 (Project Context)

這是一個使用 React 和 Node.js 的 TypeScript monorepo。
我們使用 Jest 進行測試，並遵循嚴格的 ESLint 規則。
我們的 API 是 RESTful 的，並記載於 docs/api.md 中。

## 慣例 (Conventions)

- 所有公開 API 必須維持回溯相容性
- 新功能應包含測試
- 規格說明請使用 Given/When/Then 格式
```

### 遷移後 (config.yaml)

```yaml
schema: spec-driven

context: |
  技術棧：TypeScript, React, Node.js
  測試：Jest 搭配 React Testing Library
  API：RESTful，記載於 docs/api.md
  我們為所有公開 API 維持回溯相容性

rules:
  proposal:
    - 對於高風險變更需包含回滾計畫
  specs:
    - 情境請使用 Given/When/Then 格式
    - 在發明新模式之前請先參考現有模式
  design:
    - 對於複雜流程請包含序列圖
```

### 關鍵差異

| project.md        | config.yaml                                    |
| ----------------- | ---------------------------------------------- |
| 自由格式 markdown | 結構化的 YAML                                  |
| 一大塊文字        | 分離的內容 (context) 與個別成品規則 (rules)    |
| 不清楚何時被使用  | 內容出現在所有成品中；規則僅出現在符合的成品中 |
| 無法選擇結構描述  | 明確的 `schema:` 欄位設定預設工作流            |

### 保留什麼，捨棄什麼

在遷移時，請有所取捨。問問自己：「AI 在處理*每一個*規劃請求時都需要這個嗎？」

**適合放入 `context:` 的內容：**
- 技術棧 (語言、框架、資料庫)
- 關鍵架構模式 (monorepo, 微服務等)
- 非顯而易見的約束 (「我們不能使用 X 函式庫，因為……」)
- 經常被忽略的關鍵慣例

**改放入 `rules:` 的內容：**
- 特定成品的格式 (「在規格中使用 Given/When/Then」)
- 檢閱準則 (「提案必須包含回滾計畫」)
- 這些內容僅會在處理對應成品時出現，讓其他請求保持精簡

**完全不需要放入的內容：**
- AI 已經知道的一般最佳實踐
- 可以被簡化的冗長解釋
- 不影響目前工作的歷史背景

### 遷移步驟

1. **建立 config.yaml** (如果 init 尚未建立)：
   ```yaml
   schema: spec-driven
   ```

2. **新增您的內容** (請保持簡潔 — 這些內容會進入每個請求)：
   ```yaml
   context: |
     在此填入您的專案背景。
     專注於 AI 真正需要知道的內容。
   ```

3. **新增個別成品規則** (選用)：
   ```yaml
   rules:
     proposal:
       - 您的提案專用指引
     specs:
       - 您的規格撰寫規則
   ```

4. **刪除 project.md** (當您已移動所有有用內容後)。

**不要想太多。** 從核心內容開始，然後不斷疊代。如果您發現 AI 遺漏了某些重要內容，再將其補上。如果內容顯得臃腫，就進行修剪。這是一份動態的文件。

### 需要協助？使用此提示

如果您不確定如何精煉您的 project.md，請詢問您的 AI 助理：

```
我正在將 OpenSpec 舊有的 project.md 遷移到新的 config.yaml 格式。

這是我目前的 project.md：
[貼上您的 project.md 內容]

請協助我建立一個 config.yaml，包含：
1. 簡潔的 `context:` 區段 (這會注入到每個規劃請求中，因此請保持精簡 — 專注於技術棧、關鍵約束以及經常被忽略的慣例)
2. 針對特定成品的 `rules:` (如果有任何內容是特定於某個成品的，例如「使用 Given/When/Then」屬於規格規則，而非全域內容)

請剔除 AI 模型已經知道的任何通用內容。請務必保持精確簡練。
```

AI 會協助您識別哪些是核心內容，哪些可以被修剪。

---
## 新命令

命令的可用性取決於設定檔：

**預設 (`core` 設定檔)：**

| 命令            | 用途                               |
| --------------- | ---------------------------------- |
| `/opsx:propose` | 在一個步驟中建立變更並生成規劃成品 |
| `/opsx:explore` | 以無結構的方式思考想法             |
| `/opsx:apply`   | 實作 tasks.md 中的任務             |
| `/opsx:archive` | 完成並封存變更                     |

**擴展工作流 (自訂選擇)：**

| 命令                 | 用途                        |
| -------------------- | --------------------------- |
| `/opsx:new`          | 開始一個新的變更鷹架        |
| `/opsx:continue`     | 建立下一個成品 (一次一個)   |
| `/opsx:ff`           | 快進 — 一次建立所有規劃成品 |
| `/opsx:verify`       | 驗證實作是否與規格相符      |
| `/opsx:sync`         | 預覽/規格合併，而不封存     |
| `/opsx:bulk-archive` | 一次封存多個變更            |
| `/opsx:onboard`      | 引導式端到端上線工作流      |

使用 `openspec config profile` 啟用擴展命令，然後執行 `openspec update`。

### 來自舊版的命令對應

| 舊版                 | OPSX 等效命令                                              |
| -------------------- | ---------------------------------------------------------- |
| `/openspec:proposal` | `/opsx:propose` (預設) 或 `/opsx:new` 接 `/opsx:ff` (擴展) |
| `/openspec:apply`    | `/opsx:apply`                                              |
| `/openspec:archive`  | `/opsx:archive`                                            |

### 新功能

這些功能屬於擴展工作流命令集。

**細粒度的成品建立：**
```
/opsx:continue
```
根據依賴關係一次建立一個成品。當您想要檢閱每個步驟時，請使用此命令。

**探索模式：**
```
/opsx:explore
```
在致力於某項變更之前，與夥伴一起思考想法。

---

## 理解新架構

### 從階段鎖定到流動性

舊有的工作流強制執行線性進程：

```
┌──────────────┐      ┌──────────────┐      ┌──────────────┐
│   規劃階段    │ ───► │   實作階段    │ ───►  │   封存階段   │
└──────────────┘      └──────────────┘      └──────────────┘

如果您在實作時發現設計錯誤？
太糟了。階段閘門讓您很難回頭。
```

OPSX 使用行動 (actions)，而非階段：

```
         ┌───────────────────────────────────────────────┐
         │           行動 (非階段)                        │
         │                                               │
         │     new ◄──► continue ◄──► apply ◄──► archive │
         │      │          │           │             │   │
         │      └──────────┴───────────┴─────────────┘   │
         │                    任何順序                    │
         └───────────────────────────────────────────────┘
```

### 依賴圖 (Dependency Graph)

成品形成一個有向圖。依賴關係是促成因素 (enablers)，而非閘門：

```
                        提案 (proposal)
                       (根節點)
                            │
              ┌─────────────┴─────────────┐
              │                           │
              ▼                           ▼
           規格 (specs)                設計 (design)
          (需要：                      (需要：
           提案)                       提案)
              │                           │
              └─────────────┬─────────────┘
                            │
                            ▼
                         任務 (tasks)
                        (需要：
                         規格, 設計)
```

當您執行 `/opsx:continue` 時，它會檢查哪些內容已就緒並提供下一個成品。您也可以按照任何順序建立多個就緒的成品。

### 技能 (Skills) vs 命令 (Commands)

舊系統使用特定工具的命令檔案：

```
.claude/commands/openspec/
├── proposal.md
├── apply.md
└── archive.md
```

OPSX 使用新興的**技能 (skills)** 標準：

```
.claude/skills/
├── openspec-explore/SKILL.md
├── openspec-new-change/SKILL.md
├── openspec-continue-change/SKILL.md
├── openspec-apply-change/SKILL.md
└── ...
```

技能可在多個 AI 編碼工具中被辨識，並提供更豐富的元資料。

---

## 繼續現有的變更

您正在進行中的變更與 OPSX 命令無縫銜接。

**有來自舊版工作流的作用中變更嗎？**

```
/opsx:apply add-my-feature
```

OPSX 會讀取現有成品並從您上次離開的地方繼續。

**想為現有變更新增更多成品嗎？**

```
/opsx:continue add-my-feature
```

根據已存在的內容顯示準備好建立的項目。

**需要檢視狀態嗎？**

```bash
openspec status --change add-my-feature
```

---

## 新的設定系統

### config.yaml 結構

```yaml
# 必要：新變更的預設結構描述
schema: spec-driven

# 選用：專案內容 (最大 50KB)
# 注入到「所有」成品的指令中
context: |
  您的專案背景、技術棧、
  慣例和約束。

# 選用：個別成品規則
# 僅注入到符合的成品中
rules:
  proposal:
    - 包含回滾計畫
  specs:
    - 使用 Given/When/Then 格式
  design:
    - 記錄備援策略
  tasks:
    - 拆分為最多 2 小時的區塊
```

### 結構描述解析

在決定使用哪個結構描述時，OPSX 按順序檢查：

1. **CLI 旗標**：`--schema <名稱>` (最高優先順序)
2. **變更元資料**：變更目錄中的 `.openspec.yaml`
3. **專案設定**：`openspec/config.yaml`
4. **預設值**：`spec-driven`

### 可用的結構描述

| 結構描述      | 成品                              | 最適合     |
| ------------- | --------------------------------- | ---------- |
| `spec-driven` | proposal → specs → design → tasks | 大多數專案 |

列出所有可用的結構描述：

```bash
openspec schemas
```

### 自訂結構描述

建立您自己的工作流：

```bash
openspec schema init my-workflow
```

或衍生 (fork) 現有的結構描述：

```bash
openspec schema fork spec-driven my-workflow
```

詳情請參閱 [自訂 (Customization)](customization.md)。

---

## 疑難排解

### 「在非互動模式下偵測到舊版檔案」

您正在 CI 或非互動式環境中執行。請使用：

```bash
openspec init --force
```

### 遷移後命令未出現

請重啟您的 IDE。技能是在啟動時偵測的。

### 「rules 中有不明的成品 ID」

請檢查您的 `rules:` 鍵是否與您結構描述的成品 ID 相符：

- **spec-driven**: `proposal`, `specs`, `design`, `tasks`

執行此命令以查看有效的成品 ID：

```bash
openspec schemas --json
```

### 設定未套用

1. 確保檔案位於 `openspec/config.yaml` (不是 `.yml`)
2. 驗證 YAML 語法
3. 設定變更會立即生效 — 無需重啟

### project.md 未遷移

系統刻意保留了 `project.md`，因為它可能包含您的自訂內容。請手動檢閱它，將有用的部分移動至 `config.yaml`，然後將其刪除。

### 想看看會清理哪些內容？

執行 init 並拒絕清理提示 — 您將看到完整的偵測摘要，而不會進行任何更改。

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
│       ├── openspec-propose/     # 預設 core 設定檔
│       ├── openspec-explore/
│       ├── openspec-apply-change/
│       └── ...                   # 擴展設定檔會新增 new/continue/ff 等。
├── CLAUDE.md                     # OpenSpec 標記已移除，您的內容已保留
└── AGENTS.md                     # OpenSpec 標記已移除，您的內容已保留
```

### 已移除的內容

- `.claude/commands/openspec/` — 由 `.claude/skills/` 取代
- `openspec/AGENTS.md` — 已過時
- `openspec/project.md` — 遷移至 `config.yaml` 後刪除
- `CLAUDE.md`, `AGENTS.md` 等檔案中的 OpenSpec 標記區塊

### 命令速查表

```text
/opsx:propose      快速開始 (預設 core 設定檔)
/opsx:apply        實作任務
/opsx:archive      完成並封存

# 擴展工作流 (若已啟用)：
/opsx:new          搭建變更鷹架
/opsx:continue     建立下一個成品
/opsx:ff           建立規劃成品
```

---

## 獲取協助

- **Discord**: [discord.gg/YctCnvvshC](https://discord.gg/YctCnvvshC)
- **GitHub Issues**: [github.com/Fission-AI/OpenSpec/issues](https://github.com/Fission-AI/OpenSpec/issues)
- **說明文件**: [docs/opsx.md](opsx.md) 提供完整的 OPSX 參考資料
