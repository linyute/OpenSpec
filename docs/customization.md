# 客製化

OpenSpec 提供三種層級的客製化：

| 層級                               | 功能                               | 適用對象           |
| ---------------------------------- | ---------------------------------- | ------------------ |
| **專案配置 (Project Config)**      | 設定預設值，注入上下文/規則        | 大多數團隊         |
| **自定義 Schema (Custom Schemas)** | 定義您自己的工作流成品 (artifacts) | 具有獨特流程的團隊 |
| **全域覆寫 (Global Overrides)**    | 在所有專案中共享 Schema            | 進階使用者         |

---

## 專案配置 (Project Configuration)

`openspec/config.yaml` 檔案是為您的團隊客製化 OpenSpec 最簡單的方式。它讓您可以：

- **設定預設 Schema** - 在每個指令中跳過 `--schema`
- **注入專案上下文 (context)** - 讓 AI 查看您的技術棧、慣例等。
- **添加單個成品的規則** - 為特定成品自定義規則

### 快速設定

```bash
openspec init
```

這將引導您以互動方式建立配置。或者也可以手動建立一個：

```yaml
# openspec/config.yaml
schema: spec-driven

context: |
  技術棧：TypeScript, React, Node.js, PostgreSQL
  API 風格：RESTful，記錄在 docs/api.md
  測試：Jest + React Testing Library
  我們重視所有公開 API 的回溯相容性 (backwards compatibility)

rules:
  proposal:
    - 包含回滾計畫 (rollback plan)
    - 識別受影響的團隊
  specs:
    - 使用 Given/When/Then 格式
    - 在發明新模式之前參考現有模式
```

### 運作原理

**預設 Schema：**

```bash
# 無配置時
openspec new change my-feature --schema spec-driven

# 有配置時 - Schema 會自動套用
openspec new change my-feature
```

**上下文與規則注入：**

在產生任何成品時，您的上下文與規則會被注入到 AI 提示詞 (prompt) 中：

```xml
<context>
技術棧：TypeScript, React, Node.js, PostgreSQL
...
</context>

<rules>
- 包含回滾計畫 (rollback plan)
- 識別受影響的團隊
</rules>

<template>
[Schema 內建範本]
</template>
```

- **上下文 (Context)** 會出現在「所有」成品中
- **規則 (Rules)** 「僅」會出現在相符的成品中

### Schema 解析順序

當 OpenSpec 需要一個 Schema 時，它會按照以下順序檢查：

1. CLI 旗標：`--schema <name>`
2. 變更 Metadata (`變更資料夾中的 .openspec.yaml`)
3. 專案配置 (`openspec/config.yaml`)
4. 預設值 (`spec-driven`)

---

## 自定義 Schema (Custom Schemas)

當專案配置不足以滿足需求時，您可以建立具有完全自定義工作流的專案 Schema。自定義 Schema 存放在專案的 `openspec/schemas/` 目錄中，並隨您的程式碼進行版本控制。

```text
your-project/
├── openspec/
│   ├── config.yaml        # 專案配置
│   ├── schemas/           # 自定義 Schema 存放於此
│   │   └── my-workflow/
│   │       ├── schema.yaml
│   │       └── templates/
│   └── changes/           # 您的變更
└── src/
```

### 複製現有 Schema

客製化最快的方法是複製 (fork) 一個內建的 Schema：

```bash
openspec schema fork spec-driven my-workflow
```

這會將整個 `spec-driven` Schema 複製到 `openspec/schemas/my-workflow/`，在那裡您可以自由編輯。

**您將獲得：**

```text
openspec/schemas/my-workflow/
├── schema.yaml           # 工作流定義
└── templates/
    ├── proposal.md       # 提案成品的範本
    ├── spec.md           # 規格說明的範本
    ├── design.md         # 設計文件的範本
    └── tasks.md          # 任務清單的範本
```

現在編輯 `schema.yaml` 以更改工作流，或編輯範本以更改 AI 產生的內容。

### 從頭開始建立 Schema

對於一個完全全新的工作流：

```bash
# 互動式
openspec schema init research-first

# 非互動式
openspec schema init rapid \
  --description "快速迭代工作流" \
  --artifacts "proposal,tasks" \
  --default
```

### Schema 結構

Schema 定義了工作流中的成品以及它們彼此之間的依賴關係：

```yaml
# openspec/schemas/my-workflow/schema.yaml
name: my-workflow
version: 1
description: 我團隊的自定義工作流

artifacts:
  - id: proposal
    generates: proposal.md
    description: 初始提案文件
    template: proposal.md
    instruction: |
      建立一個提案，解釋「為什麼」需要這項變更。
      重點在於問題，而非解決方案。
    requires: []

  - id: design
    generates: design.md
    description: 技術設計
    template: design.md
    instruction: |
      建立一份解釋「如何」實作的設計文件。
    requires:
      - proposal    # 在提案存在之前無法建立設計

  - id: tasks
    generates: tasks.md
    description: 實作檢核表
    template: tasks.md
    requires:
      - design

apply:
  requires: [tasks]
  tracks: tasks.md
```

**關鍵欄位：**

| 欄位          | 用途                                           |
| ------------- | ---------------------------------------------- |
| `id`          | 唯一識別碼，用於指令與規則中                   |
| `generates`   | 輸出檔案名稱 (支援 glob，例如 `specs/**/*.md`) |
| `template`    | `templates/` 目錄中的範本檔案                  |
| `instruction` | 建立此成品時的 AI 指示                         |
| `requires`    | 依賴關係 - 哪些成品必須先存在                  |

### 範本 (Templates)

範本是引導 AI 的 Markdown 檔案。在建立該成品時，它們會被注入到提示詞中。

```markdown
<!-- templates/proposal.md -->
## 為什麼 (Why)

<!-- 解釋此變更的動機。這解決了什麼問題？ -->

## 哪些變更 (What Changes)

<!-- 描述將會變更的內容。具體說明新的功能或修改。 -->

## 影響 (Impact)

<!-- 受影響的程式碼、API、依賴項、系統 -->
```

範本可以包含：
- AI 應填寫的章節標題
- 帶有 AI 引導資訊的 HTML 註解
- 顯示預期結構的範例格式

### 驗證您的 Schema

在使用自定義 Schema 之前，請先驗證它：

```bash
openspec schema validate my-workflow
```

這會檢查：
- `schema.yaml` 語法是否正確
- 所有引用的範本是否都存在
- 是否有無循環依賴
- 成品 ID 是否有效

### 使用您的自定義 Schema

建立完成後，透過以下方式使用您的 Schema：

```bash
# 在指令中指定
openspec new change feature --schema my-workflow

# 或在 config.yaml 中設定為預設值
schema: my-workflow
```

### 除錯 Schema 解析

不確定正在使用哪個 Schema？使用以下指令檢查：

```bash
# 查看特定 Schema 是從何處解析的
openspec schema which my-workflow

# 列出所有可用的 Schema
openspec schema which --all
```

輸出會顯示它是來自您的專案、使用者目錄還是套件：

```text
Schema: my-workflow
Source: project
Path: /path/to/project/openspec/schemas/my-workflow
```

---

> **注意：** OpenSpec 也支援位於 `~/.local/share/openspec/schemas/` 的使用者層級 Schema，以便在不同專案之間共享，但建議使用 `openspec/schemas/` 中的專案層級 Schema，因為它們會隨您的程式碼進行版本控制。

---

## 範例

### 快速迭代工作流

最小化的快速迭代工作流：

```yaml
# openspec/schemas/rapid/schema.yaml
name: rapid
version: 1
description: 快速迭代，最小化開銷

artifacts:
  - id: proposal
    generates: proposal.md
    description: 快速提案
    template: proposal.md
    instruction: |
      為此變更建立簡短提案。
      專注於做什麼和為什麼，跳過詳細規格。
    requires: []

  - id: tasks
    generates: tasks.md
    description: 實作檢核表
    template: tasks.md
    requires: [proposal]

apply:
  requires: [tasks]
  tracks: tasks.md
```

### 添加評論 (Review) 成品

複製預設 Schema 並添加評論步驟：

```bash
openspec schema fork spec-driven with-review
```

然後編輯 `schema.yaml` 以添加：

```yaml
  - id: review
    generates: review.md
    description: 實作前的評論檢核表
    template: review.md
    instruction: |
      根據設計建立評論檢核表。
      包含安全性、效能和測試考量。
    requires:
      - design

  - id: tasks
    # ... 現有的任務配置 ...
    requires:
      - specs
      - design
      - review    # 現在任務也需要評論
```

---

## 延伸閱讀

- [CLI 參考：Schema 指令](cli.md#schema-commands) - 完整指令文件
