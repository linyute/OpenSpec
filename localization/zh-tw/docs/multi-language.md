# 多語言指南

設定 OpenSpec 以產生英文以外語言的成品。

## 快速設定

在您的 `openspec/config.yaml` 中新增語言指示：

```yaml
schema: spec-driven

context: |
  語言：繁體中文 (zh-TW)
  所有成品必須以繁體中文撰寫。

  # 您下方的其他專案背景資訊...
  技術堆疊：TypeScript, React, Node.js
```

就這樣。所有產生的成品現在都將是繁體中文。

## 語言範例

### 葡萄牙語（巴西）

```yaml
context: |
  Language: Portuguese (pt-BR)
  All artifacts must be written in Brazilian Portuguese.
```

### 西班牙語

```yaml
context: |
  Idioma: Español
  Todos los artefactos deben escribirse en español.
```

### 中文（簡體）

```yaml
context: |
  语言：中文（简体）
  所有产出物必须用简体中文撰写。
```

### 日語

```yaml
context: |
  言語：日本語
  すべての成果物は日本語で作成してください。
```

### 法語

```yaml
context: |
  Langue : Français
  Tous les artefacts doivent être rédigés en français.
```

### 德語

```yaml
context: |
  Sprache: Deutsch
  Alle Artefakte müssen auf Deutsch verfasst werden.
```

## 提示

### 處理技術術語

決定如何處理技術術語：

```yaml
context: |
  語言：日語
  用日語撰寫，但：
  - 將 "API"、"REST"、"GraphQL" 等技術術語保留為英文
  - 程式碼範例和檔案路徑保持為英文
```

### 與其他背景資訊結合

語言設定可與您的其他專案背景資訊共存：

```yaml
schema: spec-driven

context: |
  語言：葡萄牙語 (pt-BR)
  所有成品必須以巴西葡萄牙語撰寫。

  技術堆疊：TypeScript, React 18, Node.js 20
  資料庫：使用 Prisma ORM 的 PostgreSQL
```

## 驗證

驗證您的語言設定是否奏效：

```bash
# 檢查指示 - 應顯示您的語言背景資訊
openspec instructions proposal --change my-change

# 輸出將包含您的語言背景資訊
```

## 相關文件

- [自訂指南](./customization.md) — 專案設定選項
- [工作流程指南](./workflows.md) — 完整的工作流程文件
