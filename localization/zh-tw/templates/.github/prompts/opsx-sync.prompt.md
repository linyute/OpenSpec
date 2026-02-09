---
description: 將變更中的 delta specs 同步至主 specs
---

將變更中的 delta specs 同步至主 specs。

這是一個**代理驅動 (agent-driven)** 的操作 - 您將讀取 delta specs 並直接編輯主 specs 以套用變更。這允許智慧合併（例如：新增情境而不必複製整個需求）。

**輸入**：可在 `/opsx:sync` 後選配指定變更名稱（例如 `/opsx:sync add-auth`）。如果省略，請檢查是否可以從對話內容中推斷。如果模糊不清或不明確，您必須提示使用者選擇可用的變更。

**步驟**

1. **如果未提供變更名稱，提示進行選擇**

   執行 `openspec list --json` 以取得可用的變更。使用 **AskUserQuestion 工具** 讓使用者選擇。

   顯示具有 delta specs（在 `specs/` 目錄下）的變更。

   **重要**：請勿猜測或自動選擇變更。務必讓使用者選擇。

2. **尋找 delta specs**

   在 `openspec/changes/<name>/specs/*/spec.md` 中尋找 delta spec 檔案。

   每個 delta spec 檔案包含如下章節：
   - `## ADDED Requirements` - 要新增的新需求
   - `## MODIFIED Requirements` - 對現有需求的變更
   - `## REMOVED Requirements` - 要移除的需求
   - `## RENAMED Requirements` - 要更名的需求（使用 FROM:/TO: 格式）

   如果未找到 delta specs，請告知使用者並停止。

3. **針對每個 delta spec，將變更套用至主 specs**

   針對在 `openspec/changes/<name>/specs/<capability>/spec.md` 具有 delta spec 的每個功能 (capability)：

   a. **讀取 delta spec** 以瞭解預期的變更

   b. **讀取主 spec**，位於 `openspec/specs/<capability>/spec.md`（可能尚不存在）

   c. **智慧套用變更**：

      **ADDED Requirements：**
      - 如果主 spec 中不存在該需求 → 新增它
      - 如果需求已存在 → 更新它以使其符合（視為隱含的 MODIFIED）

      **MODIFIED Requirements：**
      - 在主 spec 中尋找該需求
      - 套用變更 - 內容可能包括：
        - 新增情境（不需複製現有的情境）
        - 修改現有的情境
        - 變更需求描述
      - 保留 delta 中未提及的情境/內容

      **REMOVED Requirements：**
      - 從主 spec 中移除整個需求區塊

      **RENAMED Requirements：**
      - 尋找 FROM 需求，將其更名為 TO

   d. **建立新的主 spec**（如果該功能尚不存在）：
      - 建立 `openspec/specs/<capability>/spec.md`
      - 新增 Purpose 章節（可以很簡短，標記為 TBD）
      - 新增包含 ADDED 需求的 Requirements 章節

4. **顯示摘要**

   套用所有變更後，摘要如下：
   - 更新了哪些功能 (capabilities)
   - 做了哪些變更（新增/修改/移除/更名需求）

**Delta Spec 格式參考**

```markdown
## ADDED Requirements

### Requirement: 新功能
系統必須執行某些新功能。

#### Scenario: 基本案例
- **WHEN** 使用者執行 X
- **THEN** 系統執行 Y

## MODIFIED Requirements

### Requirement: 現有功能
#### Scenario: 要新增的新情境
- **WHEN** 使用者執行 A
- **THEN** 系統執行 B

## REMOVED Requirements

### Requirement: 已棄用的功能

## RENAMED Requirements

- FROM: `### Requirement: 舊名稱`
- TO: `### Requirement: 新名稱`
```

**關鍵原則：智慧合併**

不同於程式化的合併，您可以套用**部分更新**：
- 若要新增情境，只需在 MODIFIED 下包含該情境 - 不要複製現有的情境
- Delta 代表的是*意圖*，而非整體的替換
- 請運用您的判斷力來明智地合併變更

**成功輸出**

```
## Specs 已同步：<change-name>

已更新主 specs：

**<capability-1>**：
- 已新增需求："新功能"
- 已修改需求："現有功能"（新增了 1 個情境）

**<capability-2>**：
- 已建立新的 spec 檔案
- 已新增需求："另一個功能"

主 specs 現已更新。變更仍保持作用中 - 待實作完成後再進行封存。
```

**Guardrails**
- 在進行變更前讀取 delta 與主 specs
- 保留 delta 中未提及的現有內容
- 如果有不明確之處，請要求澄清
- 隨時顯示您正在變更的內容
- 操作應具備冪等性 (idempotent) - 執行兩次的結果應相同
