---
name: openspec-sync-specs
description: 將變更中的 delta 規格同步到主規格。當使用者想要使用 delta 規格中的變更來更新主規格，而不封存該變更時使用。
license: MIT
compatibility: 需要 openspec CLI。
metadata:
  author: openspec
  version: "1.0"
  generatedBy: "1.3.0"
---

將變更中的 delta 規格同步到主規格。

這是一個由 **代理程式驅動** 的操作 — 您將讀取 delta 規格並直接編輯主規格以套用變更。這允許智慧合併（例如：新增情境而不必複製整個需求）。

**輸入**：可選擇指定變更名稱。如果省略，請檢查是否可以從對話內容中推斷。如果模糊或不明確，您必須提示可用的變更。

**步驟**

1. **如果未提供變更名稱，請提示進行選擇**

   執行 `openspec list --json` 以獲取可用的變更。使用 **AskUserQuestion 工具** 讓使用者選擇。

   顯示具有 delta 規格的變更（在 `specs/` 目錄下）。

   **重要**：請勿猜測或自動選擇變更。務必讓使用者選擇。

2. **尋找 delta 規格**

   在 `openspec/changes/<name>/specs/*/spec.md` 中尋找 delta 規格檔案。

   每個 delta 規格檔案都包含如下區段：
   - `## ADDED Requirements` - 要新增的新需求
   - `## MODIFIED Requirements` - 對現有需求的變更
   - `## REMOVED Requirements` - 要移除的需求
   - `## RENAMED Requirements` - 要重新命名的需求（FROM:/TO: 格式）

   如果未找到 delta 規格，請通知使用者並停止。

3. **對於每個 delta 規格，將變更套用到主規格**

   對於在 `openspec/changes/<name>/specs/<capability>/spec.md` 具有 delta 規格的每個功能性 (capability)：

   a. **讀取 delta 規格** 以瞭解預期的變更

   b. **讀取位於 `openspec/specs/<capability>/spec.md` 的主規格**（可能尚不存在）

   c. **智慧套用變更**：

      **ADDED Requirements (新增需求)：**
      - 如果需求在主規格中不存在 → 建立它
      - 如果需求已存在 → 更新它以符合（視為隱含的 MODIFIED）

      **MODIFIED Requirements (修改需求)：**
      - 在主規格中尋找需求
      - 套用變更 — 這可以是：
        - 新增情境（不需要複製現有的情境）
        - 修改現有情境
        - 變更需求說明
      - 保留 delta 中未提及的情境/內容

      **REMOVED Requirements (移除需求)：**
      - 從主規格中移除整個需求區塊

      **RENAMED Requirements (重新命名需求)：**
      - 尋找 FROM 需求，重新命名為 TO

   d. **建立新的主規格**（如果功能性尚不存在）：
      - 建立 `openspec/specs/<capability>/spec.md`
      - 新增目的 (Purpose) 區段（可以很簡短，標記為待定 TBD）
      - 新增需求 (Requirements) 區段，並包含 ADDED 需求

4. **顯示摘要**

   套用所有變更後，摘要：
   - 更新了哪些功能性
   - 進行了哪些變更（需求的 新增/修改/移除/重新命名）

**Delta 規格格式參考範例**

```markdown
## ADDED Requirements

### Requirement: New Feature
The system SHALL do something new.

#### Scenario: Basic case
- **WHEN** user does X
- **THEN** system does Y

## MODIFIED Requirements

### Requirement: Existing Feature
#### Scenario: New scenario to add
- **WHEN** user does A
- **THEN** system does B

## REMOVED Requirements

### Requirement: Deprecated Feature

## RENAMED Requirements

- FROM: `### Requirement: Old Name`
- TO: `### Requirement: New Name`
```

**核心原則：智慧合併**

與程式化合併不同，您可以套用 **部分更新**：
- 要新增情境時，只需在 MODIFIED 下包含該情境即可 — 不要複製現有的情境
- Delta 代表的是 *意圖*，而不是全面的替換
- 請運用您的判斷力來明智地合併變更

**成功時的輸出**

```
## 規格已同步：<change-name>

已更新主規格：

**<capability-1>**:
- 已新增需求：「新功能」
- 已修改需求：「現有功能」（新增了 1 個情境）

**<capability-2>**:
- 已建立新的規格檔案
- 已新增需求：「另一個功能」

主規格現在已更新。變更仍處於啟動狀態 — 在實作完成後進行封存。
```

**防護準則**
- 在進行變更之前，先讀取 delta 和主規格
- 保留 delta 中未提及的現有內容
- 如果有不明確之處，請要求澄清
- 在執行過程中顯示您正在變更的內容
- 該操作應具備冪等性 — 執行兩次應得到相同的結果
