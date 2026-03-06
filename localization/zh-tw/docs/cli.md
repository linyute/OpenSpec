# CLI 參考

OpenSpec CLI (`openspec`) 提供用於專案設定、驗證、狀態檢查和管理的終端機命令。這些命令與 [命令 (Commands)](commands.md) 中記載的 AI 斜線命令 (如 `/opsx:propose`) 互補。

## 摘要

| 類別                   | 命令                                                            | 用途                                  |
| ---------------------- | --------------------------------------------------------------- | ------------------------------------- |
| **設定**               | `init`, `update`                                                | 在您的專案中初始化並更新 OpenSpec     |
| **瀏覽**               | `list`, `view`, `show`                                          | 探索變更和規格 (specs)                |
| **驗證**               | `validate`                                                      | 檢查變更和規格是否存在問題            |
| **生命週期**           | `archive`                                                       | 完成已結束的變更                      |
| **工作流**             | `status`, `instructions`, `templates`, `schemas`                | 成品引導 (artifact-driven) 工作流支援 |
| **結構描述 (Schemas)** | `schema init`, `schema fork`, `schema validate`, `schema which` | 建立並管理自訂工作流                  |
| **設定 (Config)**      | `config`                                                        | 檢視並修改設定                        |
| **工具**               | `feedback`, `completion`                                        | 回饋與 shell 整合                     |

---

## 人類 vs 代理程式命令

大多數 CLI 命令是為終端機中的**人類使用**而設計的。某些命令也透過 JSON 輸出支援**代理程式/指令碼使用**。

### 僅限人類使用的命令

這些命令具備互動性，專為終端機使用而設計：

| 命令                          | 用途                         |
| ----------------------------- | ---------------------------- |
| `openspec init`               | 初始化專案 (互動式提示)      |
| `openspec view`               | 互動式儀表板                 |
| `openspec config edit`        | 在編輯器中開啟設定           |
| `openspec feedback`           | 透過 GitHub 提交回饋         |
| `openspec completion install` | 安裝 shell 補全 (completion) |

### 代理程式相容命令

這些命令支援 `--json` 輸出，供 AI 代理程式和指令碼進行程式化使用：

| 命令                    | 人類使用           | 代理程式使用                |
| ----------------------- | ------------------ | --------------------------- |
| `openspec list`         | 瀏覽變更/規格      | `--json` 用於結構化資料     |
| `openspec show <item>`  | 讀取內容           | `--json` 用於解析           |
| `openspec validate`     | 檢查問題           | `--all --json` 用於批量驗證 |
| `openspec status`       | 檢視成品進度       | `--json` 用於結構化狀態     |
| `openspec instructions` | 獲取後續步驟       | `--json` 用於代理程式指令   |
| `openspec templates`    | 尋找模板路徑       | `--json` 用於路徑解析       |
| `openspec schemas`      | 列出可用的結構描述 | `--json` 用於結構描述探索   |

---

## 全域選項

這些選項適用於所有命令：

| 選項              | 描述         |
| ----------------- | ------------ |
| `--version`, `-V` | 顯示版本號   |
| `--no-color`      | 停用彩色輸出 |
| `--help`, `-h`    | 顯示命令幫助 |

---

## 設定命令

### `openspec init`

在您的專案中初始化 OpenSpec。建立資料夾結構並設定 AI 工具整合。

預設行為使用全域設定預設值：設定檔 `core`、遞送 `both`、工作流 `propose, explore, apply, archive`。

```
openspec init [路徑] [選項]
```

**引數 (Arguments)：**

| 引數   | 必要 | 描述                      |
| ------ | ---- | ------------------------- |
| `path` | 否   | 目標目錄 (預設：目前目錄) |

**選項：**

| 選項                 | 描述                                                          |
| -------------------- | ------------------------------------------------------------- |
| `--tools <清單>`     | 以非互動方式設定 AI 工具。使用 `all`、`none` 或逗號分隔的清單 |
| `--force`            | 自動清理舊版檔案，無需提示                                    |
| `--profile <設定檔>` | 針對此次 init 執行覆寫全域設定檔 (`core` 或 `custom`)         |

`--profile custom` 使用全域設定 (`openspec config profile`) 中目前選擇的任何工作流。

**支援的工具 ID (`--tools`)：** `amazon-q`, `antigravity`, `auggie`, `claude`, `cline`, `codex`, `codebuddy`, `continue`, `costrict`, `crush`, `cursor`, `factory`, `gemini`, `github-copilot`, `iflow`, `kilocode`, `kiro`, `opencode`, `pi`, `qoder`, `qwen`, `roocode`, `trae`, `windsurf`

**範例：**

```bash
# 互動式初始化
openspec init

# 在特定目錄中初始化
openspec init ./my-project

# 非互動式：為 Claude 和 Cursor 進行設定
openspec init --tools claude,cursor

# 為所有受支援的工具進行設定
openspec init --tools all

# 針對此次執行覆寫設定檔
openspec init --profile core

# 跳過提示並自動清理舊版檔案
openspec init --force
```

**建立的內容：**

```
openspec/
├── specs/              # 您的規格 (specs) (真相來源)
├── changes/            # 提案變更
└── config.yaml         # 專案設定

.claude/skills/         # Claude Code 技能 (若選擇 claude)
.cursor/skills/         # Cursor 技能 (若選擇 cursor)
.cursor/commands/       # Cursor OPSX 命令 (若遞送包含命令)
... (其他工具設定)
```

---

### `openspec update`

升級 CLI 後更新 OpenSpec 指令檔案。使用您目前的全域設定檔、選擇的工作流和遞送模式重新生成 AI 工具設定檔。

```
openspec update [路徑] [選項]
```

**引數：**

| 引數   | 必要 | 描述                      |
| ------ | ---- | ------------------------- |
| `path` | 否   | 目標目錄 (預設：目前目錄) |

**選項：**

| 選項      | 描述                         |
| --------- | ---------------------------- |
| `--force` | 即便檔案已是最新，也強制更新 |

**範例：**

```bash
# 在 npm 升級後更新指令檔案
npm update @fission-ai/openspec
openspec update
```

---

## 瀏覽命令

### `openspec list`

列出專案中的變更或規格 (specs)。

```
openspec list [選項]
```

**選項：**

| 選項            | 描述                                           |
| --------------- | ---------------------------------------------- |
| `--specs`       | 列出規格而非變更                               |
| `--changes`     | 列出變更 (預設)                                |
| `--sort <順序>` | 按 `recent` (最近，預設) 或 `name` (名稱) 排序 |
| `--json`        | 以 JSON 輸出                                   |

**範例：**

```bash
# 列出所有作用中的變更
openspec list

# 列出所有規格
openspec list --specs

# 指令碼適用的 JSON 輸出
openspec list --json
```

**輸出 (文字)：**

```
作用中的變更：
  add-dark-mode     UI 主題切換支援
  fix-login-bug     工作階段逾時處理
```

---

### `openspec view`

顯示用於探索規格和變更的互動式儀表板。

```
openspec view
```

開啟終端機型介面，用於導覽專案的規格和變更。

---

### `openspec show`

顯示變更或規格的詳細資訊。

```
openspec show [項目名稱] [選項]
```

**引數：**

| 引數        | 必要 | 描述                            |
| ----------- | ---- | ------------------------------- |
| `item-name` | 否   | 變更或規格的名稱 (若省略則提示) |

**選項：**

| 選項               | 描述                                                            |
| ------------------ | --------------------------------------------------------------- |
| `--type <類型>`    | 指定類型：`change` (變更) 或 `spec` (規格) (若無歧義則自動偵測) |
| `--json`           | 以 JSON 輸出                                                    |
| `--no-interactive` | 停用提示                                                        |

**變更專用選項：**

| 選項            | 描述                                     |
| --------------- | ---------------------------------------- |
| `--deltas-only` | 僅顯示增量規格 (delta specs) (JSON 模式) |

**規格專用選項：**

| 選項                     | 描述                                        |
| ------------------------ | ------------------------------------------- |
| `--requirements`         | 僅顯示需求，排除情境 (JSON 模式)            |
| `--no-scenarios`         | 排除情境內容 (JSON 模式)                    |
| `-r, --requirement <id>` | 依據從 1 開始的索引顯示特定需求 (JSON 模式) |

**範例：**

```bash
# 互動式選擇
openspec show

# 顯示特定變更
openspec show add-dark-mode

# 顯示特定規格
openspec show auth --type spec

# 解析適用的 JSON 輸出
openspec show add-dark-mode --json
```

---

## 驗證命令

### `openspec validate`

驗證變更和規格是否存在結構性問題。

```
openspec validate [項目名稱] [選項]
```

**引數：**

| 引數        | 必要 | 描述                            |
| ----------- | ---- | ------------------------------- |
| `item-name` | 否   | 要驗證的特定項目 (若省略則提示) |

**選項：**

| 選項                | 描述                                                             |
| ------------------- | ---------------------------------------------------------------- |
| `--all`             | 驗證所有變更和規格                                               |
| `--changes`         | 驗證所有變更                                                     |
| `--specs`           | 驗證所有規格                                                     |
| `--type <類型>`     | 名稱有歧義時指定類型：`change` 或 `spec`                         |
| `--strict`          | 啟用嚴格驗證模式                                                 |
| `--json`            | 以 JSON 輸出                                                     |
| `--concurrency <n>` | 最大平行驗證數 (預設：6，或使用 `OPENSPEC_CONCURRENCY` 環境變數) |
| `--no-interactive`  | 停用提示                                                         |

**範例：**

```bash
# 互動式驗證
openspec validate

# 驗證特定變更
openspec validate add-dark-mode

# 驗證所有變更
openspec validate --changes

# 以 JSON 輸出驗證所有內容 (適用於 CI/指令碼)
openspec validate --all --json

# 增加平行度進行嚴格驗證
openspec validate --all --strict --concurrency 12
```

**輸出 (文字)：**

```
正在驗證 add-dark-mode...
  ✓ proposal.md 有效
  ✓ specs/ui/spec.md 有效
  ⚠ design.md：缺少「技術方案 (Technical Approach)」章節

發現 1 個警告
```

**輸出 (JSON)：**

```json
{
  "version": "1.0.0",
  "results": {
    "changes": [
      {
        "name": "add-dark-mode",
        "valid": true,
        "warnings": ["design.md: missing 'Technical Approach' section"]
      }
    ]
  },
  "summary": {
    "total": 1,
    "valid": 1,
    "invalid": 0
  }
}
```

---

## 生命週期命令

### `openspec archive`

封存已完成的變更，並將增量規格合併至主規格中。

```
openspec archive [變更名稱] [選項]
```

**引數：**

| 引數          | 必要 | 描述                        |
| ------------- | ---- | --------------------------- |
| `change-name` | 否   | 要封存的變更 (若省略則提示) |

**選項：**

| 選項            | 描述                                            |
| --------------- | ----------------------------------------------- |
| `-y, --yes`     | 跳過確認提示                                    |
| `--skip-specs`  | 跳過規格更新 (適用於僅基礎設施/工具/文件的變更) |
| `--no-validate` | 跳過驗證 (需要確認)                             |

**範例：**

```bash
# 互動式封存
openspec archive

# 封存特定變更
openspec archive add-dark-mode

# 無提示封存 (適用於 CI/指令碼)
openspec archive add-dark-mode --yes

# 封存不影響規格的工具變更
openspec archive update-ci-config --skip-specs
```

**執行內容：**

1. 驗證變更 (除非使用 `--no-validate`)
2. 提示進行確認 (除非使用 `--yes`)
3. 將增量規格合併至 `openspec/specs/`
4. 將變更資料夾移動至 `openspec/changes/archive/YYYY-MM-DD-<名稱>/`

---

## 工作流命令

這些命令支援成品引導的 OPSX 工作流。它們對於人類檢查進度以及代理程式決定後續步驟都很有用。

### `openspec status`

顯示變更的成品完成狀態。

```
openspec status [選項]
```

**選項：**

| 選項              | 描述                                  |
| ----------------- | ------------------------------------- |
| `--change <id>`   | 變更名稱 (若省略則提示)               |
| `--schema <名稱>` | 結構描述覆寫 (從變更的設定中自動偵測) |
| `--json`          | 以 JSON 輸出                          |

**範例：**

```bash
# 互動式狀態檢查
openspec status

# 特定變更的狀態
openspec status --change add-dark-mode

# 代理程式使用的 JSON
openspec status --change add-dark-mode --json
```

**輸出 (文字)：**

```
變更：add-dark-mode
結構描述：spec-driven
進度：2/4 個成品已完成

[x] proposal
[ ] design
[x] specs
[-] tasks (受阻於：design)
```

**輸出 (JSON)：**

```json
{
  "changeName": "add-dark-mode",
  "schemaName": "spec-driven",
  "isComplete": false,
  "applyRequires": ["tasks"],
  "artifacts": [
    {"id": "proposal", "outputPath": "proposal.md", "status": "done"},
    {"id": "design", "outputPath": "design.md", "status": "ready"},
    {"id": "specs", "outputPath": "specs/**/*.md", "status": "done"},
    {"id": "tasks", "outputPath": "tasks.md", "status": "blocked", "missingDeps": ["design"]}
  ]
}
```

---

### `openspec instructions`

獲取用於建立成品或執行任務的豐富指令。AI 代理程式使用此命令來了解下一步要建立什麼。

```
openspec instructions [成品] [選項]
```

**引數：**

| 引數       | 必要 | 描述                                                       |
| ---------- | ---- | ---------------------------------------------------------- |
| `artifact` | 否   | 成品 ID：`proposal`, `specs`, `design`, `tasks` 或 `apply` |

**選項：**

| 選項              | 描述                            |
| ----------------- | ------------------------------- |
| `--change <id>`   | 變更名稱 (在非互動模式下為必要) |
| `--schema <名稱>` | 結構描述覆寫                    |
| `--json`          | 以 JSON 輸出                    |

**特殊情況：** 使用 `apply` 作為成品以獲取任務實作指令。

**範例：**

```bash
# 獲取下一個成品的指令
openspec instructions --change add-dark-mode

# 獲取特定成品的指令
openspec instructions design --change add-dark-mode

# 獲取執行 (apply)/實作指令
openspec instructions apply --change add-dark-mode

# 代理程式使用的 JSON
openspec instructions design --change add-dark-mode --json
```

**輸出內容包括：**

- 成品的模板內容
- 來自設定的專案內容 (context)
- 來自依賴成品的內容
- 來自設定的個別成品規則

---

### `openspec templates`

顯示結構描述中所有成品的已解析模板路徑。

```
openspec templates [選項]
```

**選項：**

| 選項              | 描述                                   |
| ----------------- | -------------------------------------- |
| `--schema <名稱>` | 要檢查的結構描述 (預設：`spec-driven`) |
| `--json`          | 以 JSON 輸出                           |

**範例：**

```bash
# 顯示預設結構描述的模板路徑
openspec templates

# 顯示自訂結構描述的模板
openspec templates --schema my-workflow

# 程式化使用的 JSON
openspec templates --json
```

**輸出 (文字)：**

```
結構描述：spec-driven

模板：
  proposal  → ~/.openspec/schemas/spec-driven/templates/proposal.md
  specs     → ~/.openspec/schemas/spec-driven/templates/specs.md
  design    → ~/.openspec/schemas/spec-driven/templates/design.md
  tasks     → ~/.openspec/schemas/spec-driven/templates/tasks.md
```

---

### `openspec schemas`

列出可用的工作流結構描述及其描述與成品流。

```
openspec schemas [選項]
```

**選項：**

| 選項     | 描述         |
| -------- | ------------ |
| `--json` | 以 JSON 輸出 |

**範例：**

```bash
openspec schemas
```

**輸出：**

```
可用的結構描述：

  spec-driven (套件)
    預設的規格驅動開發工作流
    流程：proposal → specs → design → tasks

  my-custom (專案)
    此專案的自訂工作流
    流程：research → proposal → tasks
```

---

## 結構描述命令

用於建立和管理自訂工作流結構描述的命令。

### `openspec schema init`

建立一個新的專案本地結構描述。

```
openspec schema init <名稱> [選項]
```

**引數：**

| 引數   | 必要 | 描述                      |
| ------ | ---- | ------------------------- |
| `name` | 是   | 結構描述名稱 (kebab-case) |

**選項：**

| 選項                   | 描述                                                      |
| ---------------------- | --------------------------------------------------------- |
| `--description <文字>` | 結構描述描述                                              |
| `--artifacts <清單>`   | 以逗號分隔的成品 ID (預設：`proposal,specs,design,tasks`) |
| `--default`            | 設定為專案預設結構描述                                    |
| `--no-default`         | 不要提示設定為預設                                        |
| `--force`              | 覆寫現有結構描述                                          |
| `--json`               | 以 JSON 輸出                                              |

**範例：**

```bash
# 互動式結構描述建立
openspec schema init research-first

# 帶有特定成品的非互動式建立
openspec schema init rapid \
  --description "快速疊代工作流" \
  --artifacts "proposal,tasks" \
  --default
```

**建立的內容：**

```
openspec/schemas/<名稱>/
├── schema.yaml           # 結構描述定義
└── templates/
    ├── proposal.md       # 每個成品的模板
    ├── specs.md
    ├── design.md
    └── tasks.md
```

---

### `openspec schema fork`

將現有結構描述複製到您的專案中以進行自訂。

```
openspec schema fork <來源> [名稱] [選項]
```

**引數：**

| 引數     | 必要 | 描述                                   |
| -------- | ---- | -------------------------------------- |
| `source` | 是   | 要複製的結構描述                       |
| `name`   | 否   | 新結構描述名稱 (預設：`<來源>-custom`) |

**選項：**

| 選項      | 描述         |
| --------- | ------------ |
| `--force` | 覆寫現有目標 |
| `--json`  | 以 JSON 輸出 |

**範例：**

```bash
# 衍生 (fork) 內建的 spec-driven 結構描述
openspec schema fork spec-driven my-workflow
```

---

### `openspec schema validate`

驗證結構描述的結構與模板。

```
openspec schema validate [名稱] [選項]
```

**引數：**

| 引數   | 必要 | 描述                                |
| ------ | ---- | ----------------------------------- |
| `name` | 否   | 要驗證的結構描述 (若省略則驗證所有) |

**選項：**

| 選項        | 描述             |
| ----------- | ---------------- |
| `--verbose` | 顯示詳細驗證步驟 |
| `--json`    | 以 JSON 輸出     |

**範例：**

```bash
# 驗證特定結構描述
openspec schema validate my-workflow

# 驗證所有結構描述
openspec schema validate
```

---

### `openspec schema which`

顯示結構描述從何處解析 (對於除錯優先順序很有用)。

```
openspec schema which [名稱] [選項]
```

**引數：**

| 引數   | 必要 | 描述         |
| ------ | ---- | ------------ |
| `name` | 否   | 結構描述名稱 |

**選項：**

| 選項     | 描述                     |
| -------- | ------------------------ |
| `--all`  | 列出所有結構描述及其來源 |
| `--json` | 以 JSON 輸出             |

**範例：**

```bash
# 檢查結構描述來源
openspec schema which spec-driven
```

**輸出：**

```
spec-driven 解析來源：套件 (package)
  來源：/usr/local/lib/node_modules/@fission-ai/openspec/schemas/spec-driven
```

**結構描述優先順序：**

1. 專案 (Project)：`openspec/schemas/<名稱>/`
2. 使用者 (User)：`~/.local/share/openspec/schemas/<名稱>/`
3. 套件 (Package)：內建結構描述

---

## 設定命令

### `openspec config`

檢視並修改全域 OpenSpec 設定。

```
openspec config <子命令> [選項]
```

**子命令：**

| 子命令             | 描述                                           |
| ------------------ | ---------------------------------------------- |
| `path`             | 顯示設定檔位置                                 |
| `list`             | 顯示目前所有設定                               |
| `get <鍵>`         | 獲取特定值                                     |
| `set <鍵> <值>`    | 設定一個值                                     |
| `unset <鍵>`       | 移除一個鍵                                     |
| `reset`            | 重設為預設值                                   |
| `edit`             | 在 `$EDITOR` 中開啟                            |
| `profile [預設集]` | 透過互動方式或預設集 (preset) 設定工作流設定檔 |

**範例：**

```bash
# 顯示設定檔路徑
openspec config path

# 列出所有設定
openspec config list

# 獲取特定值
openspec config get telemetry.enabled

# 設定一個值
openspec config set telemetry.enabled false

# 明確地設定字串值
openspec config set user.name "My Name" --string

# 移除自訂設定
openspec config unset user.name

# 重設所有設定
openspec config reset --all --yes

# 在編輯器中編輯設定
openspec config edit

# 透過以行動為基礎的精靈設定設定檔
openspec config profile

# 快速預設集：將工作流切換為 core (保留遞送模式)
openspec config profile core
```

`openspec config profile` 從目前狀態摘要開始，然後讓您選擇：
- 變更遞送 + 工作流
- 僅變更遞送
- 僅變更工作流
- 保留目前設定 (結束)

如果您保留目前設定，則不會寫入任何變更，也不會顯示更新提示。
若無設定變更，但目前專案檔案與您的全域設定檔/遞送不同步，OpenSpec 將顯示警告並建議執行 `openspec update`。
按下 `Ctrl+C` 也會乾淨地取消流程 (無堆疊追蹤) 並以代碼 `130` 結束。
在工作流核取清單中，`[x]` 表示該工作流已在全域設定中選取。若要將這些選擇套用至專案檔案，請執行 `openspec update` (或在專案內收到提示時選擇 `Apply changes to this project now?`)。

**互動式範例：**

```bash
# 僅更新遞送
openspec config profile
# 選擇：Change delivery only
# 選擇遞送：Skills only

# 僅更新工作流
openspec config profile
# 選擇：Change workflows only
# 在核取清單中切換工作流，然後確認
```

---

## 實用命令

### `openspec feedback`

提交有關 OpenSpec 的回饋。建立 GitHub issue。

```
openspec feedback <訊息> [選項]
```

**引數：**

| 引數      | 必要 | 描述     |
| --------- | ---- | -------- |
| `message` | 是   | 回饋訊息 |

**選項：**

| 選項            | 描述     |
| --------------- | -------- |
| `--body <文字>` | 詳細描述 |

**需求：** 必須安裝 GitHub CLI (`gh`) 並已通過驗證。

**範例：**

```bash
openspec feedback "新增對自訂成品類型的支援" \
  --body "我想在內建類型之外定義自己的成品類型。"
```

---

### `openspec completion`

管理 OpenSpec CLI 的 shell 補全 (completion)。

```
openspec completion <子命令> [shell]
```

**子命令：**

| 子命令              | 描述                      |
| ------------------- | ------------------------- |
| `generate [shell]`  | 將補全指令碼輸出至 stdout |
| `install [shell]`   | 為您的 shell 安裝補全     |
| `uninstall [shell]` | 移除已安裝的補全          |

**支援的 shell：** `bash`, `zsh`, `fish`, `powershell`

**範例：**

```bash
# 安裝補全 (自動偵測 shell)
openspec completion install

# 為特定 shell 安裝
openspec completion install zsh

# 生成用於手動安裝的指令碼
openspec completion generate bash > ~/.bash_completion.d/openspec

# 解除安裝
openspec completion uninstall
```

---

## 結束代碼 (Exit Codes)

| 代碼 | 意義                        |
| ---- | --------------------------- |
| `0`  | 成功                        |
| `1`  | 錯誤 (驗證失敗、檔案缺失等) |

---

## 環境變數

| 變數                   | 描述                                 |
| ---------------------- | ------------------------------------ |
| `OPENSPEC_CONCURRENCY` | 批量驗證的預設平行度 (預設：6)       |
| `EDITOR` 或 `VISUAL`   | 用於 `openspec config edit` 的編輯器 |
| `NO_COLOR`             | 設定後停用彩色輸出                   |

---

## 相關文件

- [命令 (Commands)](commands.md) - AI 斜線命令 (`/opsx:propose`, `/opsx:apply` 等)
- [工作流 (Workflows)](workflows.md) - 常見模式以及何時使用各個命令
- [自訂 (Customization)](customization.md) - 建立自訂結構描述與模板
- [入門指南 (Getting Started)](getting-started.md) - 首次設定指南
