---
name: openspec-archive-change
description: 在實驗性工作流中封存已完成的變更。在實作完成後，當使用者想要完成並封存變更時使用。
license: MIT
compatibility: 需要 openspec CLI。
metadata:
  author: openspec
  version: "1.0"
  generatedBy: "1.1.1"
---

在實驗性工作流中封存已完成的變更。

**輸入**：可在指令後選配指定變更名稱。如果省略，請檢查是否可以從對話內容中推斷。如果模糊不清或不明確，您必須提示使用者選擇可用的變更。

**步驟**

1. **如果未提供變更名稱，提示進行選擇**

   執行 `openspec list --json` 以取得可用的變更。使用 **AskUserQuestion 工具** 讓使用者選擇。

   僅顯示作用中（尚未封存）的變更。
   如果可用，請包含每個變更所使用的 schema。

   **重要**：請勿猜測或自動選擇變更。務必讓使用者選擇。

2. **檢查 artifact 完成狀態**

   執行 `openspec status --change "<name>" --json` 以檢查 artifact 完成情況。

   解析 JSON 以瞭解：
   - `schemaName`：正在使用的工作流
   - `artifacts`：artifacts 列表及其狀態（`done` 或其他）

   **如果有任何 artifacts 未完成 (`done`)：**
   - 顯示警告，列出未完成的 artifacts
   - 使用 **AskUserQuestion 工具** 確認使用者是否想要繼續
   - 如果使用者確認則繼續執行

3. **檢查任務完成狀態**

   讀取任務檔案（通常為 `tasks.md`）以檢查未完成的任務。

   計算標記為 `- [ ]`（未完成）與 `- [x]`（已完成）的任務。

   **如果發現未完成的任務：**
   - 顯示警告，呈現未完成任務的數量
   - 使用 **AskUserQuestion 工具** 確認使用者是否想要繼續
   - 如果使用者確認則繼續執行

   **如果不存在任務檔案：** 直接繼續，不顯示任務相關警告。

4. **評估 delta spec 同步狀態**

   檢查 `openspec/changes/<name>/specs/` 是否存在 delta specs。如果不存在，則繼續執行且不提示同步。

   **如果存在 delta specs：**
   - 將每個 delta spec 與其在 `openspec/specs/<capability>/spec.md` 中對應的主 spec 進行比較
   - 確定將套用哪些變更（新增、修改、移除、更名）
   - 在提示前顯示整合摘要

   **提示選項：**
   - 如果需要變更：「立即同步（建議）」、「直接封存且不進行同步」
   - 如果已同步：「立即封存」、「仍然同步」、「取消」

   如果使用者選擇同步，執行 `/opsx:sync` 邏輯（使用 `openspec-sync-specs` 技能）。無論選擇為何，之後皆繼續進行封存。

5. **執行封存**

   如果封存目錄不存在，請建立它：
   ```bash
   mkdir -p openspec/changes/archive
   ```

   使用目前日期產生目標名稱：`YYYY-MM-DD-<change-name>`

   **檢查目標是否已存在：**
   - 如果是：失敗並顯示錯誤，建議重新命名現有封存或使用不同的日期
   - 如果否：將變更目錄移動至封存

   ```bash
   mv openspec/changes/<name> openspec/changes/archive/YYYY-MM-DD-<name>
   ```

6. **顯示摘要**

   顯示封存完成摘要，包含：
   - 變更名稱
   - 所使用的 schema
   - 封存位置
   - 是否已同步 specs（如果適用）
   - 關於任何警告（未完成的 artifacts/任務）的註記

**成功輸出**

```
## 封存完成

**變更：** <change-name>
**Schema：** <schema-name>
**封存至：** openspec/changes/archive/YYYY-MM-DD-<name>/
**Specs：** ✓ 已同步至主 specs (或 "無 delta specs" 或 "已跳過同步")

所有 artifacts 已完成。所有任務已完成。
```

**Guardrails**
- 如果未提供，務必提示選擇變更
- 使用 artifact 圖表 (`openspec status --json`) 進行完成情況檢查
- 不要因為警告而阻擋封存——僅告知並確認
- 移動至封存時保留 `.openspec.yaml`（它會隨目錄移動）
- 顯示發生的情況之清晰摘要
- 如果要求同步，使用 `openspec-sync-specs` 方法（代理驅動）
- 如果存在 delta specs，務必在提示前執行同步評估並顯示整合摘要
