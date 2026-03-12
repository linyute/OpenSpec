---
description: 將差異規格從變更同步至主規格
---

將差異規格從變更同步至主規格。

這是一個 **代理驅動 (agent-driven)** 的操作 —— 您將讀取差異規格，並直接編輯主規格以套用變更。這允許智慧合併（例如：新增案例而不必複製整個需求）。

**輸入**：在 `/opsx:sync` 之後可以選擇性指定一個變更名稱（例如：`/opsx:sync add-auth`）。如果省略，請檢查是否可以從對話語境中推論。如果語義模糊或不明確，您必須提示使用者選擇可用的變更。

**步驟**

1. **如果未提供變更名稱，則提示使用者選擇**

   執行 `openspec list --json` 以取得可用的變更。使用 **AskUserQuestion 工具** 讓使用者選擇。

   顯示具有差異規格（在 `specs/` 目錄下）的變更。

   **重要事項**：請勿猜測或自動選取變更。務必讓使用者自行選擇。

2. **尋找差異規格**

   在 `openspec/changes/<name>/specs/*/spec.md` 中尋找差異規格檔案。

   每個差異規格檔案包含如下章節：
   - `## ADDED Requirements` —— 欲新增的新需求
   - `## MODIFIED Requirements` —— 對現有需求的變更
   - `## REMOVED Requirements` —— 欲移除的需求
   - `## RENAMED Requirements` —— 欲重新命名的需求（格式為 FROM:/TO:）

   如果找不到差異規格，請通知使用者並停止。

3. **對於每個差異規格，將變更套用至主規格**

   對於在 `openspec/changes/<name>/specs/<capability>/spec.md` 具有差異規格的每個能力：

   a. **讀取差異規格**：以瞭解預期的變更。

   b. **讀取主規格**：位於 `openspec/specs/<capability>/spec.md`（可能尚不存在）。

   c. **智慧套用變更**：

      **新增的需求 (ADDED Requirements)：**
      - 如果主規格中不存在該需求 -> 新增它
      - 如果該需求已存在 -> 更新它以使其相符（視為隱性的 MODIFIED）

      **修改的需求 (MODIFIED Requirements)：**
      - 在主規格中找到該需求
      - 套用變更 —— 這可以是：
        - 新增新案例（不需複製現有案例）
        - 修改現有案例
        - 變更需求描述
      - 保留差異規格中未提及的案例/內容

      **移除的需求 (REMOVED Requirements)：**
      - 從主規格中移除整個需求區塊

      **重新命名的需求 (RENAMED Requirements)：**
      - 找到 FROM 的需求，將其重新命名為 TO

   d. **如果能力尚不存在，則建立新的主規格**：
      - 建立 `openspec/specs/<capability>/spec.md`
      - 新增「用途 (Purpose)」章節（可以簡短，並標記為 TBD）
      - 使用「新增的需求 (ADDED Requirements)」新增「需求 (Requirements)」章節

4. **顯示摘要**

   套用所有變更後，摘要說明：
   - 哪些能力已更新
   - 做了哪些變更（需求的 新增/修改/移除/重新命名）

**差異規格格式參考**

```markdown
## ADDED Requirements

### Requirement: 新功能 (New Feature)
系統「應 (SHALL)」執行某些新操作。

#### Scenario: 基本案例
- **WHEN** 使用者執行 X
- **THEN** 系統執行 Y

## MODIFIED Requirements

### Requirement: 現有功能 (Existing Feature)
#### Scenario: 欲新增的新案例
- **WHEN** 使用者執行 A
- **THEN** 系統執行 B

## REMOVED Requirements

### Requirement: 已廢棄的功能 (Deprecated Feature)

## RENAMED Requirements

- FROM: `### Requirement: 舊名稱`
- TO: `### Requirement: 新名稱`
```

**關鍵原則：智慧合併**

與程式化合併不同，您可以套用 **部分更新**：
- 要新增一個案例，只需將該案例包含在 MODIFIED 之下 —— 不要複製現有的案例
- 差異規格代表的是「意圖」，而非整體的替換
- 請發揮您的判斷力，合理地合併變更

**成功輸出**

```
## 規格已同步：<change-name>

已更新主規格：

**<capability-1>**：
- 已新增需求：「新功能」
- 已修改需求：「現有功能」（已新增 1 個案例）

**<capability-2>**：
- 已建立新的規格檔案
- 已新增需求：「另一個功能」

主規格現已更新。該變更仍保持作用中狀態 —— 請在實作完成後進行封存。
```

**防護機制**
- 在進行變更前，務必同時讀取差異規格和主規格
- 保留主規格中未在差異規格提及的現有內容
- 如果有不明確之處，請要求澄清
- 隨時顯示您正在變更的內容
- 操作應具備冪等性 (idempotent) —— 執行兩次應得到相同的結果
