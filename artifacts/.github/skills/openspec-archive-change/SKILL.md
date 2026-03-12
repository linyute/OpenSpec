---
name: openspec-archive-change
description: 在實驗性工作流中封存已完成的變更。當使用者在實作完成後想要結束並封存變更時使用。
license: MIT
compatibility: 需要 openspec CLI。
metadata:
  author: openspec
  version: "1.0"
  generatedBy: "1.2.0"
---

在實驗性工作流中封存已完成的變更。

**輸入**：可選擇性地指定變更名稱。如果省略，請檢查是否可以從對話上下文中推斷。如果模糊或不明確，您「必須」提示可用的變更。

**步驟**

1. **如果未提供變更名稱，請提示進行選擇**

   執行 `openspec list --json` 以取得可用的變更。使用 **AskUserQuestion 工具** 讓使用者選擇。

   僅顯示作用中的變更（尚未封存）。
   如果可用，請包含每個變更所使用的結構描述（schema）。

   **重要**：請勿猜測或自動選擇變更。務必讓使用者自行選擇。

2. **檢查成品（artifact）完成狀態**

   執行 `openspec status --change "<name>" --json` 以檢查成品完成情況。

   解析 JSON 以了解：
   - `schemaName`: 正在使用的工作流
   - `artifacts`: 成品列表及其狀態（`done` 或其他）

   **如果有任何成品尚未為 `done`：**
   - 顯示列出未完成成品的警告
   - 使用 **AskUserQuestion 工具** 確認使用者是否要繼續
   - 如果使用者確認，則繼續

3. **檢查任務（task）完成狀態**

   讀取任務檔案（通常是 `tasks.md`）以檢查未完成的任務。

   計算標記為 `- [ ]`（未完成）與 `- [x]`（已完成）的任務數量。

   **如果發現未完成的任務：**
   - 顯示未完成任務數量的警告
   - 使用 **AskUserQuestion 工具** 確認使用者是否要繼續
   - 如果使用者確認，則繼續

   **如果任務檔案不存在**：繼續執行，不顯示任務相關警告。

4. **評估差異規格（delta spec）同步狀態**

   檢查 `openspec/changes/<name>/specs/` 中的差異規格。如果不存在，則繼續執行而不提示同步。

   **如果存在差異規格：**
   - 將每個差異規格與其在 `openspec/specs/<capability>/spec.md` 中對應的主要規格進行比較
   - 確定將套用的變更（新增、修改、移除、重命名）
   - 在提示前顯示綜合摘要

   **提示選項：**
   - 如果需要變更：「立即同步（建議）」、「不進行同步直接封存」
   - 如果已同步：「立即封存」、「仍然同步」、「取消」

   如果使用者選擇同步，請使用 Task 工具（subagent_type: "general-purpose", prompt: "使用 Skill 工具為變更 '<name>' 呼叫 openspec-sync-specs。差異規格分析：<包含分析後的差異規格摘要>"）。無論選擇如何，都繼續進行封存。

5. **執行封存**

   如果封存目錄不存在，請建立它：
   ```bash
   mkdir -p openspec/changes/archive
   ```

   使用目前日期產生目標名稱：`YYYY-MM-DD-<change-name>`

   **檢查目標是否已存在：**
   - 如果是：則回報錯誤失敗，建議重新命名現有封存或使用不同的日期
   - 如果否：將變更目錄移至封存

   ```bash
   mv openspec/changes/<name> openspec/changes/archive/YYYY-MM-DD-<name>
   ```

6. **顯示摘要**

   顯示封存完成摘要，包括：
   - 變更名稱
   - 所使用的結構描述
   - 封存位置
   - 規格是否已同步（如果適用）
   - 關於任何警告的附註（未完成的成品/任務）

**成功時的輸出**

```
## 封存完成

**變更：** <change-name>
**結構描述：** <schema-name>
**已封存至：** openspec/changes/archive/YYYY-MM-DD-<name>/
**規格：** ✓ 已同步至主要規格 (或 "無差異規格" 或 "已跳過同步")

所有成品已完成。所有任務已完成。
```

**護欄**
- 如果未提供變更名稱，務必提示進行選擇
- 使用成品圖（openspec status --json）進行完成檢查
- 不要因警告而阻礙封存 - 僅通知並確認
- 移動到封存時保留 .openspec.yaml（它隨目錄一起移動）
- 顯示所發生情況的清晰摘要
- 如果請求同步，請使用 openspec-sync-specs 方法（代理程式驅動）
- 如果存在差異規格，務必在提示前執行同步評估並顯示綜合摘要
