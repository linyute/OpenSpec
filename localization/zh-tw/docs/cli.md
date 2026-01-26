# CLI 參考

OpenSpec CLI (`openspec`) 提供用於專案設定、驗證、狀態檢查和管理的終端機指令。這些指令與 [指令](commands.md) 中記錄的 AI 斜線指令（如 `/opsx:new`）互補。

## 摘要

| 類別         | 指令                                                            | 用途                              |
| ------------ | --------------------------------------------------------------- | --------------------------------- |
| **設定**     | `init`, `update`                                                | 在您的專案中初始化和更新 OpenSpec |
| **瀏覽**     | `list`, `view`, `show`                                          | 探索變更和規格                    |
| **驗證**     | `validate`                                                      | 檢查變更和規格是否存在問題        |
| **生命週期** | `archive`                                                       | 完成已結束的變更                  |
| **工作流程** | `status`, `instructions`, `templates`, `schemas`                | 基於成品驅動的工作流程支援        |
| **Schema**   | `schema init`, `schema fork`, `schema validate`, `schema which` | 建立和管理自訂工作流程            |
| **設定**     | `config`                                                        | 檢視和修改設定                    |
| **公用程式** | `feedback`, `completion`                                        | 回饋和 shell 整合                 |

---

## 真人 vs 代理指令

大多數 CLI 指令是為在終端機中由 **真人使用** 而設計的。某些指令也透過 JSON 輸出支援 **代理/指令碼使用**。

### 僅限真人指令

這些指令是互動式的，專為終端機使用而設計：

| 指令                          | 用途                     |
| ----------------------------- | ------------------------ |
| `openspec init`               | 初始化專案（互動式提示） |
| `openspec view`               | 互動式儀表板             |
| `openspec config edit`        | 在編輯器中開啟設定       |
| `openspec feedback`           | 透過 GitHub 提交回饋     |
| `openspec completion install` | 安裝 shell 補全          |

### 代理相容指令

這些指令支援 `--json` 輸出，供 AI 代理和指令碼程式化使用：

| 指令                    | 真人使用          | 代理使用                    |
| ----------------------- | ----------------- | --------------------------- |
| `openspec list`         | 瀏覽變更/規格     | `--json` 用於結構化資料     |
| `openspec show <item>`  | 讀取內容          | `--json` 用於剖析           |
| `openspec validate`     | 檢查問題          | `--all --json` 用於批次驗證 |
| `openspec status`       | 檢視成品進度      | `--json` 用於結構化狀態     |
| `openspec instructions` | 獲取後續步驟      | `--json` 用於代理指示       |
| `openspec templates`    | 尋找範本路徑      | `--json` 用於路徑解析       |
| `openspec schemas`      | 列出可用的 schema | `--json` 用於 schema 探索   |

---

## 全域選項

這些選項適用於所有指令：

| 選項              | 說明         |
| ----------------- | ------------ |
| `--version`, `-V` | 顯示版本號   |
| `--no-color`      | 停用彩色輸出 |
| `--help`, `-h`    | 顯示指令說明 |

---

## 設定指令

### `openspec init`

在您的專案中初始化 OpenSpec。建立資料夾結構並設定 AI 工具整合。

```
openspec init [path] [options]
```

**引數：**

| 引數   | 必填 | 說明                       |
| ------ | ---- | -------------------------- |
| `path` | 否   | 目標目錄（預設：目前目錄） |

**選項：**

| 選項             | 說明                                                      |
| ---------------- | --------------------------------------------------------- |
| `--tools <list>` | 非互動式地設定 AI 工具。使用 `all`、`none` 或逗號分隔列表 |
| `--force`        | 自動清理舊版檔案而不提示                                  |

**支援的工具：** `amazon-q`, `antigravity`, `auggie`, `claude`, `cline`, `codex`, `codebuddy`, `continue`, `costrict`, `crush`, `cursor`, `factory`, `gemini`, `github-copilot`, `iflow`, `kilocode`, `opencode`, `qoder`, `qwen`, `roocode`, `windsurf`

**範例：**

```bash
# 互動式初始化
openspec init

# 在特定目錄中初始化
openspec init ./my-project

# 非互動式：為 Claude 和 Cursor 進行設定
openspec init --tools claude,cursor

# 為所有支援的工具進行設定
openspec init --tools all

# 跳過提示並自動清理舊版檔案
openspec init --force
```

**建立的內容：**

```
openspec/
├── specs/              # 您的規格（單一事實來源）
├── changes/            # 提議的變更
└── config.yaml         # 專案設定

.claude/skills/         # Claude Code 技能檔案（如果選擇了 claude）
.cursor/rules/          # Cursor 規則（如果選擇了 cursor）
... (其他工具設定)
```

---

### `openspec update`

在升級 CLI 後更新 OpenSpec 指示檔案。重新產生 AI 工具設定檔案。

```
openspec update [path] [options]
```

**引數：**

| 引數   | 必填 | 說明                       |
| ------ | ---- | -------------------------- |
| `path` | 否   | 目標目錄（預設：目前目錄） |

**選項：**

| 選項      | 說明                       |
| --------- | -------------------------- |
| `--force` | 即使檔案已是最新也強制更新 |

**範例：**

```bash
# 在 npm 升級後更新指示檔案
npm update @fission-ai/openspec
openspec update
```

---

## 瀏覽指令

### `openspec list`

列出專案中的變更或規格。

```
openspec list [options]
```

**選項：**

| 選項             | 說明                              |
| ---------------- | --------------------------------- |
| `--specs`        | 列出規格而非變更                  |
| `--changes`      | 列出變更（預設）                  |
| `--sort <order>` | 按 `recent`（預設）或 `name` 排序 |
| `--json`         | 以 JSON 格式輸出                  |

**範例：**

```bash
# 列出所有作用中的變更
openspec list

# 列出所有規格
openspec list --specs

# 指令碼使用的 JSON 輸出
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

開啟一個終端機介面，用於導覽專案的規格和變更。

---

### `openspec show`

顯示變更或規格的詳細資訊。

```
openspec show [item-name] [options]
```

**引數：**

| 引數        | 必填 | 說明                               |
| ----------- | ---- | ---------------------------------- |
| `item-name` | 否   | 變更或規格名稱（如果省略則會提示） |

**選項：**

| 選項               | 說明                                                 |
| ------------------ | ---------------------------------------------------- |
| `--type <type>`    | 指定類型：`change` 或 `spec`（如果不明確則自動偵測） |
| `--json`           | 以 JSON 格式輸出                                     |
| `--no-interactive` | 停用提示                                             |

**變更特定選項：**

| 選項            | 說明                       |
| --------------- | -------------------------- |
| `--deltas-only` | 僅顯示差異規格 (JSON 模式) |

**規格特定選項：**

| 選項                     | 說明                                      |
| ------------------------ | ----------------------------------------- |
| `--requirements`         | 僅顯示需求，排除情境 (JSON 模式)          |
| `--no-scenarios`         | 排除情境內容 (JSON 模式)                  |
| `-r, --requirement <id>` | 按從 1 開始的索引顯示特定需求 (JSON 模式) |

**範例：**

```bash
# 互動式選擇
openspec show

# 顯示特定變更
openspec show add-dark-mode

# 顯示特定規格
openspec show auth --type spec

# 用於剖析的 JSON 輸出
openspec show add-dark-mode --json
```

---

## 驗證指令

### `openspec validate`

驗證變更和規格是否存在結構性問題。

```
openspec validate [item-name] [options]
```

**引數：**

| 引數        | 必填 | 說明                                 |
| ----------- | ---- | ------------------------------------ |
| `item-name` | 否   | 要驗證的特定項目（如果省略則會提示） |

**選項：**

| 選項                | 說明                                                          |
| ------------------- | ------------------------------------------------------------- |
| `--all`             | 驗證所有變更和規格                                            |
| `--changes`         | 驗證所有變更                                                  |
| `--specs`           | 驗證所有規格                                                  |
| `--type <type>`     | 當名稱不明確時指定類型：`change` 或 `spec`                    |
| `--strict`          | 啟用嚴格驗證模式                                              |
| `--json`            | 以 JSON 格式輸出                                              |
| `--concurrency <n>` | 最大平行驗證數（預設：6，或 `OPENSPEC_CONCURRENCY` 環境變數） |
| `--no-interactive`  | 停用提示                                                      |

**範例：**

```bash
# 互動式驗證
openspec validate

# 驗證特定變更
openspec validate add-dark-mode

# 驗證所有變更
openspec validate --changes

# 帶 JSON 輸出的完整驗證（用於 CI/指令碼）
openspec validate --all --json

# 具有更高平行度的嚴格驗證
openspec validate --all --strict --concurrency 12
```

**輸出 (文字)：**

```
正在驗證 add-dark-mode...
  ✓ proposal.md 有效
  ✓ specs/ui/spec.md 有效
  ⚠ design.md: 缺少 "Technical Approach" 章節

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

## 生命週期指令

### `openspec archive`

封存已完成的變更並將差異規格合併到主規格中。

```
openspec archive [change-name] [options]
```

**引數：**

| 引數          | 必填 | 說明                             |
| ------------- | ---- | -------------------------------- |
| `change-name` | 否   | 要封存的變更（如果省略則會提示） |

**選項：**

| 選項            | 說明                                             |
| --------------- | ------------------------------------------------ |
| `-y, --yes`     | 跳過確認提示                                     |
| `--skip-specs`  | 跳過規格更新（用於僅限基礎設施/工具/文件的變更） |
| `--no-validate` | 跳過驗證（需要確認）                             |

**範例：**

```bash
# 互動式封存
openspec archive

# 封存特定變更
openspec archive add-dark-mode

# 無提示封存（CI/指令碼）
openspec archive add-dark-mode --yes

# 封存不影響規格的工具變更
openspec archive update-ci-config --skip-specs
```

**它的作用：**

1. 驗證變更（除非使用 `--no-validate`）
2. 提示確認（除非使用 `--yes`）
3. 將差異規格合併到 `openspec/specs/` 中
4. 將變更資料夾移動到 `openspec/changes/archive/YYYY-MM-DD-<name>/`

---

## 工作流程指令

這些指令支援成品驅動的 OPSX 工作流程。它們對於檢查進度的真人和決定後續步驟的代理都很有用。

### `openspec status`

顯示變更的成品完成狀態。

```
openspec status [options]
```

**選項：**

| 選項              | 說明                                  |
| ----------------- | ------------------------------------- |
| `--change <id>`   | 變更名稱（如果省略則會提示）          |
| `--schema <name>` | Schema 覆寫（從變更的設定中自動偵測） |
| `--json`          | 以 JSON 格式輸出                      |

**範例：**

```bash
# 互動式狀態檢查
openspec status

# 特定變更的狀態
openspec status --change add-dark-mode

# 代理使用的 JSON
openspec status --change add-dark-mode --json
```

**輸出 (文字)：**

```
變更：add-dark-mode
Schema：spec-driven

成品：
  ✓ proposal     proposal.md 已存在
  ✓ specs        specs/ 已存在
  ◆ design       就緒 (需要：specs)
  ○ tasks        已封鎖 (需要：design)

下一步：使用 /opsx:continue 建立 design
```

**輸出 (JSON)：**

```json
{
  "change": "add-dark-mode",
  "schema": "spec-driven",
  "artifacts": [
    {"id": "proposal", "status": "complete", "path": "proposal.md"},
    {"id": "specs", "status": "complete", "path": "specs/"},
    {"id": "design", "status": "ready", "requires": ["specs"]},
    {"id": "tasks", "status": "blocked", "requires": ["design"]}
  ],
  "next": "design"
}
```

---

### `openspec instructions`

獲取用於建立成品或套用任務的豐富指示。由 AI 代理使用以瞭解接下來要建立什麼。

```
openspec instructions [artifact] [options]
```

**引數：**

| 引數       | 必填 | 說明                                                       |
| ---------- | ---- | ---------------------------------------------------------- |
| `artifact` | 否   | 成品 ID：`proposal`、`specs`、`design`、`tasks` 或 `apply` |

**選項：**

| 選項              | 說明                           |
| ----------------- | ------------------------------ |
| `--change <id>`   | 變更名稱（在非互動模式下必填） |
| `--schema <name>` | Schema 覆寫                    |
| `--json`          | 以 JSON 格式輸出               |

**特殊情況：** 使用 `apply` 作為成品以獲取任務實作指示。

**範例：**

```bash
# 獲取下一個成品的指示
openspec instructions --change add-dark-mode

# 獲取特定成品的指示
openspec instructions design --change add-dark-mode

# 獲取套用/實作指示
openspec instructions apply --change add-dark-mode

# 供代理使用的 JSON
openspec instructions design --change add-dark-mode --json
```

**輸出包含：**

- 成品的範本內容
- 來自設定的專案內容
- 來自相依成品的內容
- 來自設定的每個成品規則

---

### `openspec templates`

顯示 schema 中所有成品的解析範本路徑。

```
openspec templates [options]
```

**選項：**

| 選項              | 說明                                   |
| ----------------- | -------------------------------------- |
| `--schema <name>` | 要檢查的 schema（預設：`spec-driven`） |
| `--json`          | 以 JSON 格式輸出                       |

**範例：**

```bash
# 顯示預設 schema 的範本路徑
openspec templates

# 顯示自訂 schema 的範本路徑
openspec templates --schema tdd

# 用於程式化使用的 JSON
openspec templates --json
```

**輸出 (文字)：**

```
Schema：spec-driven

範本：
  proposal  → ~/.openspec/schemas/spec-driven/templates/proposal.md
  specs     → ~/.openspec/schemas/spec-driven/templates/specs.md
  design    → ~/.openspec/schemas/spec-driven/templates/design.md
  tasks     → ~/.openspec/schemas/spec-driven/templates/tasks.md
```

---

### `openspec schemas`

列出可用的工作流程 schema 及其說明和成品流程。

```
openspec schemas [options]
```

**選項：**

| 選項     | 說明             |
| -------- | ---------------- |
| `--json` | 以 JSON 格式輸出 |

**範例：**

```bash
openspec schemas
```

**輸出：**

```
可用的 schema：

  spec-driven (套件)
    預設的規格驅動開發工作流程
    流程：proposal → specs → design → tasks

  tdd (套件)
    測試驅動開發工作流程
    流程：spec → tests → implementation → docs

  my-custom (專案)
    此專案的自訂工作流程
    流程：research → proposal → tasks
```

---

## Schema 指令

用於建立和管理自訂工作流程 schema 的指令。

### `openspec schema init`

建立一個新的專案本地 schema。

```
openspec schema init <name> [options]
```

**引數：**

| 引數   | 必填 | 說明                     |
| ------ | ---- | ------------------------ |
| `name` | 是   | Schema 名稱 (kebab-case) |

**選項：**

| 選項                   | 說明                                                     |
| ---------------------- | -------------------------------------------------------- |
| `--description <text>` | Schema 說明                                              |
| `--artifacts <list>`   | 逗號分隔的成品 ID（預設：`proposal,specs,design,tasks`） |
| `--default`            | 設定為專案預設 schema                                    |
| `--no-default`         | 不提示設定為預設值                                       |
| `--force`              | 覆寫現有 schema                                          |
| `--json`               | 以 JSON 格式輸出                                         |

**範例：**

```bash
# 互動式建立 schema
openspec schema init research-first

# 帶有特定成品的非互動式建立
openspec schema init tdd-lite \
  --description "Lightweight TDD workflow" \
  --artifacts "spec,tests,implementation" \
  --default
```

**建立的內容：**

```
openspec/schemas/<name>/
├── schema.yaml           # Schema 定義
└── templates/
    ├── proposal.md       # 每個成品的範本
    ├── specs.md
    ├── design.md
    └── tasks.md
```

---

### `openspec schema fork`

複製現有的 schema 到您的專案進行自訂。

```
openspec schema fork <source> [name] [options]
```

**引數：**

| 引數     | 必填 | 說明                                      |
| -------- | ---- | ----------------------------------------- |
| `source` | 是   | 要複製的 schema                           |
| `name`   | 否   | 新 schema 名稱（預設：`<source>-custom`） |

**選項：**

| 選項      | 說明             |
| --------- | ---------------- |
| `--force` | 覆寫現有目標     |
| `--json`  | 以 JSON 格式輸出 |

**範例：**

```bash
# 分支內建的 spec-driven schema
openspec schema fork spec-driven my-workflow
```

---

### `openspec schema validate`

驗證 schema 的結構和範本。

```
openspec schema validate [name] [options]
```

**引數：**

| 引數   | 必填 | 說明                                  |
| ------ | ---- | ------------------------------------- |
| `name` | 否   | 要驗證的 schema（如果省略則驗證所有） |

**選項：**

| 選項        | 說明               |
| ----------- | ------------------ |
| `--verbose` | 顯示詳細的驗證步驟 |
| `--json`    | 以 JSON 格式輸出   |

**範例：**

```bash
# 驗證特定 schema
openspec schema validate my-workflow

# 驗證所有 schema
openspec schema validate
```

---

### `openspec schema which`

顯示 schema 的解析來源（對於偵錯優先順序很有用）。

```
openspec schema which [name] [options]
```

**引數：**

| 引數   | 必填 | 說明        |
| ------ | ---- | ----------- |
| `name` | 否   | Schema 名稱 |

**選項：**

| 選項     | 說明                     |
| -------- | ------------------------ |
| `--all`  | 列出所有 schema 及其來源 |
| `--json` | 以 JSON 格式輸出         |

**範例：**

```bash
# 檢查 schema 來自哪裡
openspec schema which spec-driven
```

**輸出：**

```
spec-driven 解析自：套件
  來源：/usr/local/lib/node_modules/@fission-ai/openspec/schemas/spec-driven
```

**Schema 優先順序：**

1. 專案：`openspec/schemas/<name>/`
2. 使用者：`~/.local/share/openspec/schemas/<name>/`
3. 套件：內建 schema

---

## 設定指令

### `openspec config`

檢視和修改全域 OpenSpec 設定。

```
openspec config <subcommand> [options]
```

**子指令：**

| 子指令              | 說明                |
| ------------------- | ------------------- |
| `path`              | 顯示設定檔案路徑    |
| `list`              | 顯示所有目前設定    |
| `get <key>`         | 獲取特定值          |
| `set <key> <value>` | 設定值              |
| `unset <key>`       | 移除鍵              |
| `reset`             | 重設為預設值        |
| `edit`              | 在 `$EDITOR` 中開啟 |

**範例：**

```bash
# 顯示設定檔案路徑
openspec config path

# 列出所有設定
openspec config list

# 獲取特定值
openspec config get telemetry.enabled

# 設定值
openspec config set telemetry.enabled false

# 明確設定字串值
openspec config set user.name "My Name" --string

# 移除自訂設定
openspec config unset user.name

# 重設所有設定
openspec config reset --all --yes

# 在編輯器中編輯設定
openspec config edit
```

---

## 公用程式指令

### `openspec feedback`

提交有關 OpenSpec 的回饋。建立一個 GitHub issue。

```
openspec feedback <message> [options]
```

**引數：**

| 引數      | 必填 | 說明     |
| --------- | ---- | -------- |
| `message` | 是   | 回饋訊息 |

**選項：**

| 選項            | 說明     |
| --------------- | -------- |
| `--body <text>` | 詳細說明 |

**要求：** 必須安裝 GitHub CLI (`gh`) 並進行驗證。

**範例：**

```bash
openspec feedback "Add support for custom artifact types" \
  --body "I'd like to define my own artifact types beyond the built-in ones."
```

---

### `openspec completion`

管理 OpenSpec CLI 的 shell 補全。

```
openspec completion <subcommand> [shell]
```

**子指令：**

| 子指令              | 說明                      |
| ------------------- | ------------------------- |
| `generate [shell]`  | 將補全指令碼輸出到 stdout |
| `install [shell]`   | 為您的 shell 安裝補全     |
| `uninstall [shell]` | 移除已安裝的補全          |

**支援的 shell：** `bash`, `zsh`, `fish`, `powershell`

**範例：**

```bash
# 安裝補全（自動偵測 shell）
openspec completion install

# 為特定 shell 安裝
openspec completion install zsh

# 產生用於手動安裝的指令碼
openspec completion generate bash > ~/.bash_completion.d/openspec

# 解除安裝
openspec completion uninstall
```

---

## 離開代碼

| 代碼 | 意義                         |
| ---- | ---------------------------- |
| `0`  | 成功                         |
| `1`  | 錯誤（驗證失敗、檔案缺失等） |

---

## 環境變數

| 變數                   | 說明                            |
| ---------------------- | ------------------------------- |
| `OPENSPEC_CONCURRENCY` | 批次驗證的預設平行度（預設：6） |
| `EDITOR` 或 `VISUAL`   | `openspec config edit` 的編輯器 |
| `NO_COLOR`             | 設定後停用彩色輸出              |

---

## 相關文件

- [指令](commands.md) - AI 斜線指令 (`/opsx:new`, `/opsx:apply` 等)
- [工作流程](workflows.md) - 常見模式以及何時使用各個指令
- [自訂](customization.md) - 建立自訂 schema 和範本
- [入門指南](getting-started.md) - 首次設定指南
