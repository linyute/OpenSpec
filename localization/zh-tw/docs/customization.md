# 自訂

OpenSpec 提供兩個層級的自訂：

1. **專案設定 (Project Config)** (`openspec/config.yaml`) - 輕量級的每個專案自訂，用於預設 schema、共用背景資訊和每個成品的規則。
2. **Schema 覆寫 (Schema Overrides)** - 透過 XDG 目錄進行完整的 schema 和範本自訂。

---

## 專案設定

`openspec/config.yaml` 檔案提供了一個輕量級的自訂層，讓團隊可以：

- **設定預設 schema** - 新的變更會自動使用此 schema，而不需要每次都指定 `--schema`。
- **注入專案背景資訊** - 在建立任何成品時顯示給 AI 的共用背景資訊（技術堆疊、慣例）。
- **新增每個成品的規則** - 僅套用於特定成品（例如：提案、規格）的自訂規則。

### 建立設定

您可以透過互動式設定來建立設定：

```bash
openspec init
```

或者手動建立：

```yaml
# openspec/config.yaml
schema: spec-driven

context: |
  技術堆疊：TypeScript, React, Node.js, PostgreSQL
  API 風格：RESTful，記錄在 docs/api.md 中
  測試：Jest + React Testing Library
  我們重視所有公共 API 的向後相容性

rules:
  proposal:
    - 包含回滾計畫
    - 識別受影響的團隊
  specs:
    - 使用 Given/When/Then 格式
    - 在發明新模式之前參考現有模式
```

### 設定如何影響工作流程

**預設 schema 選擇：**

```bash
# 無設定：必須指定 schema
openspec new change my-feature --schema spec-driven

# 有設定：schema 是自動的
openspec new change my-feature
```

**背景資訊和規則注入：**

產生成品的指示時，會注入背景資訊和規則：

```xml
<context>
技術堆疊：TypeScript, React, Node.js, PostgreSQL
API 風格：RESTful，記錄在 docs/api.md 中
...
</context>

<rules>
- 包含回滾計畫
- 識別受影響的團隊
</rules>

<template>
[Schema 內建的範本]
</template>
```

- **背景資訊 (context)** 出現在所有成品中。
- **規則 (rules)** 僅出現在相符的成品中。

### Schema 解析優先順序

1. CLI 旗標獲勝：`openspec new change feature --schema tdd`
2. 變更 Metadata（如果 `.openspec.yaml` 指定了 schema）
3. 專案設定 (`openspec/config.yaml`)
4. 預設 schema (`spec-driven`)

### 錯誤處理

設定提供了適當的錯誤處理：

```bash
# schema 名稱拼字錯誤 - 顯示建議
# Schema 'spec-drivne' not found
# Did you mean: spec-driven (built-in)

# 規則中不明的成品 ID - 警告但繼續執行
# ⚠️ Unknown artifact ID in rules: "testplan". Valid IDs for schema "spec-driven": ...
```

---

## Schema 覆寫

對於更深層次的自訂，您可以覆寫整個 schema 或範本。

### Schema 解析如何運作

OpenSpec 使用遵循 XDG 基本目錄規格的 2 層級 schema 解析系統：

1. **使用者覆寫**：`${XDG_DATA_HOME}/openspec/schemas/<name>/`
2. **套件內建**：`<npm-package>/schemas/<name>/`

請求 schema 時，解析器會先檢查使用者目錄。如果找到，則使用該整個 schema 目錄。否則，它將回退到套件的內建 schema。

### 覆寫目錄

| 平台 | 路徑 |
|----------|------|
| macOS/Linux | `~/.local/share/openspec/schemas/` |
| Windows | `%LOCALAPPDATA%\[openspec](https://github.com/openspec/openspec)\schemas\` |
| 全部 (如果已設定) | `$XDG_DATA_HOME/openspec/schemas/` |

### 手動 Schema 覆寫

要覆寫預設的 `spec-driven` schema：

**1. 建立目錄結構：**

```bash
# macOS/Linux
mkdir -p ~/.local/share/openspec/schemas/spec-driven/templates
```

**2. 尋找並複製預設 schema 檔案：**

```bash
# 尋找套件位置
npm list -g openspec --parseable

# 從套件的 schemas/ 目錄複製檔案
cp <package-path>/schemas/spec-driven/schema.yaml ~/.local/share/openspec/schemas/spec-driven/
cp <package-path>/schemas/spec-driven/templates/*.md ~/.local/share/openspec/schemas/spec-driven/templates/
```

**3. 修改複製的檔案：**

編輯 `schema.yaml` 以更改工作流程結構：

```yaml
name: spec-driven
version: 1
description: 我的自訂工作流程
artifacts:
  - id: proposal
    generates: proposal.md
    description: 初始提案
    template: proposal.md
    requires: []
  # 新增、移除或修改成品...
```

編輯 `templates/` 中的範本以自訂內容引導。

### 目前的限制

| 問題 | 影響 |
|-------|--------|
| **路徑發現** | 使用者必須瞭解 XDG 慣例和平台特定路徑 |
| **套件位置** | 尋找 npm 套件路徑因安裝方法而異 |
| **無腳手架 (scaffolding)** | 使用者必須手動建立目錄並複製檔案 |
| **無驗證** | 無法確認實際解析的是哪個 schema |
| **需要完整複製** | 即使只更改一個範本也必須複製整個 schema |

### 相關檔案

| 檔案 | 用途 |
|------|---------|
| `src/core/artifact-graph/resolver.ts` | Schema 解析邏輯 |
| `src/core/artifact-graph/instruction-loader.ts` | 範本載入 |
| `src/core/global-config.ts` | XDG 路徑輔助程式 |
| `schemas/spec-driven/` | 預設 schema 和範本 |
