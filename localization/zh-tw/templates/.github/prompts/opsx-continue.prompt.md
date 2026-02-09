---
description: 繼續處理變更 - 建立下一個 artifact（實驗性）
---

透過建立下一個 artifact 來繼續處理變更。

**輸入**：可在 `/opsx:continue` 後選配指定變更名稱（例如 `/opsx:continue add-auth`）。如果省略，請檢查是否可以從對話內容中推斷。如果模糊不清或不明確，您必須提示使用者選擇可用的變更。

**步驟**

1. **如果未提供變更名稱，提示進行選擇**

   執行 `openspec list --json` 以取得可用的變更，並依最近修改時間排序。然後使用 **AskUserQuestion 工具** 讓使用者選擇要處理的變更。

   呈現前 3-4 個最近修改的變更作為選項，顯示：
   - 變更名稱
   - Schema（來自 `schema` 欄位，如果存在；否則為 "spec-driven"）
   - 狀態（例如 "0/5 個任務"、"已完成"、"無任務"）
   - 最近修改時間（來自 `lastModified` 欄位）

   將最近修改的變更標記為「（建議）」，因為這很可能是使用者想要繼續的變更。

   **重要**：請勿猜測或自動選擇變更。務必讓使用者選擇。

2. **檢查目前狀態**
   ```bash
   openspec status --change "<name>" --json
   ```
   解析 JSON 以瞭解目前狀態。回應包含：
   - `schemaName`：正在使用的工作流 schema（例如 "spec-driven"）
   - `artifacts`：artifacts 陣列及其狀態（"done"、"ready"、"blocked"）
   - `isComplete`：布林值，指示所有 artifacts 是否已完成

3. **根據狀態採取行動**：

   ---

   **如果所有 artifacts 皆已完成 (`isComplete: true`)**：
   - 向使用者表示祝賀
   - 顯示最終狀態，包括所使用的 schema
   - 建議：「所有 artifacts 已建立！您現在可以使用 `/opsx:apply` 實作此變更，或使用 `/opsx:archive` 進行封存。」
   - 停止

   ---

   **如果已準備好建立 artifacts**（狀態顯示 artifacts 的 `status` 為 `"ready"`）：
   - 從狀態輸出中挑選第一個 `status` 為 `"ready"` 的 artifact
   - 取得其指令：
     ```bash
     openspec instructions <artifact-id> --change "<name>" --json
     ```
   - 解析 JSON。關鍵欄位為：
     - `context`：專案背景（對您的約束 - **不要**包含在輸出中）
     - `rules`：Artifact 專屬規則（對您的約束 - **不要**包含在輸出中）
     - `template`：用於輸出檔案的結構
     - `instruction`：Schema 專屬引導
     - `outputPath`：寫入 artifact 的位置
     - `dependencies`：需讀取以獲取背景資訊的已完成 artifacts
   - **建立 artifact 檔案**：
     - 讀取任何已完成的相依檔案以獲取背景資訊
     - 使用 `template` 作為結構 - 填寫其章節
     - 寫入時將 `context` 與 `rules` 作為約束條件套用 - 但**不要**將它們複製到檔案中
     - 寫入指令中指定的輸出路徑
   - 顯示建立了什麼以及現在解鎖了什麼
   - 建立一個 artifact 後停止

   ---

   **如果沒有已就緒的 artifacts（全部被阻擋）**：
   - 在有效的 schema 中不應發生這種情況
   - 顯示狀態並建議檢查問題

4. **建立 artifact 後，顯示進度**
   ```bash
   openspec status --change "<name>"
   ```

**輸出**

每次呼叫後，顯示：
- 建立了哪個 artifact
- 正在使用的 schema 工作流
- 目前進度（N/M 已完成）
- 現在解鎖了哪些 artifacts
- 提示：「執行 `/opsx:continue` 以建立下一個 artifact」

**Artifact 建立指南**

Artifact 類型及其用途取決於 schema。使用指令輸出中的 `instruction` 欄位來瞭解要建立什麼。

常見的 artifact 模式：

**spec-driven schema** (proposal → specs → design → tasks)：
- **proposal.md**：如果不明確，請詢問使用者關於變更的資訊。填寫 Why、What Changes、Capabilities、Impact。
  - Capabilities 章節至關重要 - 列表中列出的每個功能都需要一個 spec 檔案。
- **specs/<capability>/spec.md**：為 proposal 的 Capabilities 章節中列出的每個功能建立一個 spec（使用功能名稱，而非變更名稱）。
- **design.md**：記錄技術決策、架構與實作方法。
- **tasks.md**：將實作分解為附帶核取方塊的任務。

針對其他 schemas，遵循 CLI 輸出中的 `instruction` 欄位。

**Guardrails**
- 每次呼叫僅建立一個 artifact
- 在建立新 artifact 之前，務必讀取相依的 artifacts
- 絕不跳過 artifacts 或不按順序建立
- 如果背景資訊不明確，請在建立前詢問使用者
- 寫入後驗證 artifact 檔案是否存在，然後再標記進度
- 使用 schema 的 artifact 序列，不要假設特定的 artifact 名稱
- **重要**：`context` 與 `rules` 是對您的約束，而非檔案內容
  - **不要**將 `<context>`、`<rules>`、`<project_context>` 區塊複製到 artifact 中
  - 這些用於引導您編寫內容，但絕不應出現在輸出中
