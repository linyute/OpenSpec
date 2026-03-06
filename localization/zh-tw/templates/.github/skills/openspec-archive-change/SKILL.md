---
name: openspec-archive-change
description: 在實驗性工作流中封存已完成的變更。當使用者想要在實作完成後敲定並封存變更時使用。
license: MIT
compatibility: 需要 openspec CLI。
metadata:
  author: openspec
  version: "1.0"
  generatedBy: "1.2.0"
---

在實驗性工作流中封存已完成的變更。

**輸入**：選配指定變更名稱。如果省略，檢查是否可從對話背景中推論。如果模糊或不明確，您必需 (MUST) 提示以取得可用的變更。

**步驟**

1. **如果未提供變更名稱，提示進行選擇**

   執行 `openspec list --json` 以取得可用的變更。使用 **AskUserQuestion 工具** 讓使用者進行選擇。

   僅顯示作用中的變更 (尚未封存的變更)。
   如果可用，包含每個變更所使用的架構 (schema)。

   **重要 (IMPORTANT)**：請勿猜測或自動選擇變更。務必讓使用者自行選擇。

2. **檢查構件完成狀態**

   執行 `openspec status --change "<name>" --json` 以檢查構件完成情況。

   解析 JSON 以了解：
   - `schemaName`：正在使用的工作流
   - `artifacts`：構件清單及其狀態 (`done` 或其他)

   **如果有任何構件未完成 (`done`)**：
   - 顯示列出未完成構件的警告
   - 使用 **AskUserQuestion 工具** 確認使用者是否要繼續
   - 如果使用者確認則繼續進行

3. **檢查工作 (task) 完成狀態**

   讀取工作檔案 (通常為 `tasks.md`) 以檢查是否有未完成的工作。

   計算標記為 `- [ ]` (未完成) 與 `- [x]` (已完成) 的工作數量。

   **如果發現未完成的工作**：
   - 顯示顯示未完成工作數量的警告
   - 使用 **AskUserQuestion 工具** 確認使用者是否要繼續
   - 如果使用者確認則繼續進行

   **如果工作檔案不存在**：直接進行，不顯示工作相關警告。

4. **評估 delta 規格 (spec) 同步狀態**

   檢查 `openspec/changes/<name>/specs/` 是否存在 delta 規格。如果不存在，直接進行，不顯示同步提示。

   **如果存在 delta 規格**：
   - 將每個 delta 規格與其在 `openspec/specs/<capability>/spec.md` 中對應的主要規格進行比較
   - 確定將套用哪些變更 (新增、修改、移除、重新命名)
   - 在提示前顯示合併摘要

   **提示選項**：
   - 如果需要變更："立即同步 (建議)"、"不進行同步直接封存"
   - 如果已同步："立即封存"、"仍然同步"、"取消"

   如果使用者選擇同步，使用 Task 工具 (subagent_type: "general-purpose", prompt: "使用 Skill 工具為變更 '<name>' 呼叫 openspec-sync-specs。Delta 規格分析：<包含分析後的 delta 規格摘要>")。無論選擇為何，皆繼續進行封存。

5. **執行封存**

   如果封存目錄不存在，則建立它：
   ```bash
   mkdir -p openspec/changes/archive
   ```

   使用目前日期產生目標名稱：`YYYY-MM-DD-<change-name>`

   **檢查目標是否已存在**：
   - 如果是：失敗並報錯，建議重新命名現有封存或使用不同的日期
   - 如果否：將變更目錄移動到封存目錄

   ```bash
   mv openspec/changes/<name> openspec/changes/archive/YYYY-MM-DD-<name>
   ```

6. **顯示摘要**

   顯示封存完成摘要，包括：
   - 變更名稱
   - 所使用的架構
   - 封存位置
   - 規格是否已同步 (如果適用)
   - 關於任何警告的說明 (未完成的構件/工作)

**成功時的輸出**

```
## 封存完成

**變更：** <change-name>
**架構：** <schema-name>
**封存至：** openspec/changes/archive/YYYY-MM-DD-<name>/
**規格：** ✓ 已同步至主要規格 (或 "無 delta 規格" 或 "已跳過同步")

所有構件已完成。所有工作已完成。
```

**護欄 (Guardrails)**
- 如果未提供變更，務必提示以進行選擇
- 使用構件圖 (openspec status --json) 進行完成度檢查
- 不要因為警告而阻擋封存 - 僅通知並確認
- 移動到封存時保留 .openspec.yaml (它會隨目錄移動)
- 顯示發生情況的清晰摘要
- 如果請求同步，使用 openspec-sync-specs 方法 (代理程式驅動)
- 如果存在 delta 規格，務必在提示前執行同步評估並顯示合併摘要
