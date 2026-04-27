---
name: openspec-continue-change
description: 透過建立下一個成品來繼續進行 OpenSpec 變更。當使用者想要推進其變更、建立下一個成品或繼續其工作流程時使用。
license: MIT
compatibility: 需要 openspec CLI。
metadata:
  author: openspec
  version: "1.0"
  generatedBy: "1.3.1"
---

透過建立下一個成品來繼續進行變更。

**輸入**：可選擇指定變更名稱。如果省略，請檢查是否可以從對話內容中推斷。如果模糊或不明確，你必須提示可用的變更。

**步驟**

1. **如果未提供變更名稱，提示進行選擇**

   執行 `openspec list --json` 以取得按最近修改排序的可用變更。然後使用 **AskUserQuestion 工具** 讓使用者選擇要處理的變更。

   將最近修改的前 3-4 個變更作為選項呈現，顯示：
   - 變更名稱
   - Schema（如果存在 `schema` 欄位，否則為「spec-driven」）
   - 狀態（例如：「0/5 個任務」、「完成」、「無任務」）
   - 最近修改的時間（來自 `lastModified` 欄位）

   將最近修改的變更標記為「(建議)」，因為這很可能是使用者想要繼續的部分。

   **重要**：不要猜測或自動選擇變更。務必讓使用者選擇。

2. **檢查目前狀態**
   ```bash
   openspec status --change "<name>" --json
   ```
   解析 JSON 以了解目前狀態。回應包含：
   - `schemaName`：正在使用的工作流程 Schema（例如：「spec-driven」）
   - `artifacts`：包含其狀態（「done」、「ready」、「blocked」）的成品陣列
   - `isComplete`：表示所有成品是否已完成的布林值

3. **根據狀態採取行動**：

   ---

   **如果所有成品均已完成 (`isComplete: true`)**：
   - 恭喜使用者
   - 顯示最終狀態，包括使用的 Schema
   - 建議：「所有成品已建立！你現在可以實作此變更或將其封存。」
   - 停止

   ---

   **如果有成品準備好建立**（狀態顯示成品的 `status: "ready"`）：
   - 從狀態輸出中挑選第一個 `status: "ready"` 的成品
   - 取得其指令：
     ```bash
     openspec instructions <artifact-id> --change "<name>" --json
     ```
   - 解析 JSON。關鍵欄位為：
     - `context`：專案背景（對你的約束 - 請勿包含在輸出中）
     - `rules`：特定成品的規則（對你的約束 - 請勿包含在輸出中）
     - `template`：用於輸出檔案的結構
     - `instruction`：特定 Schema 的指引
     - `outputPath`：寫入成品的位置
     - `dependencies`：需讀取以了解上下文的已完成成品
   - **建立成品檔案**：
     - 讀取任何已完成的相依檔案以了解上下文
     - 使用 `template` 作為結構 - 填寫其各個區段
     - 在寫入時將 `context` 和 `rules` 作為約束應用 - 但不要將它們複製到檔案中
     - 寫入指令中指定的輸出路徑
   - 顯示已建立的內容以及現在已解除鎖定的內容
   - 建立一個成品後停止

   ---

   **如果沒有成品準備好（全部被阻擋）**：
   - 在有效的 Schema 中不應發生這種情況
   - 顯示狀態並建議檢查問題

4. **建立成品後，顯示進度**
   ```bash
   openspec status --change "<name>"
   ```

**輸出**

每次呼叫後，顯示：
- 建立了哪個成品
- 正在使用的 Schema 工作流程
- 目前進度（N/M 已完成）
- 現在有哪些成品已解除鎖定
- 提示：「想要繼續嗎？只需叫我繼續或告訴我接下來要做什麼。」

**成品建立指南**

成品類型及其目的取決於 Schema。請使用指令輸出中的 `instruction` 欄位來了解要建立什麼。

常見的成品模式：

**spec-driven schema** (proposal → specs → design → tasks)：
- **proposal.md**：如果不清楚，請詢問使用者關於變更的資訊。填寫「原因 (Why)」、「變更內容 (What Changes)」、「功能 (Capabilities)」、「影響 (Impact)」。
  - 「功能 (Capabilities)」區段至關重要 - 列出的每個功能都需要一個 spec 檔案。
- **specs/<capability>/spec.md**：為 proposal 的「功能 (Capabilities)」區段中列出的每個功能建立一個 spec（使用功能名稱，而非變更名稱）。
- **design.md**：文件化技術決策、架構和實作方法。
- **tasks.md**：將實作拆分為帶有核取方塊的任務。

對於其他 Schema，請遵循 CLI 輸出中的 `instruction` 欄位。

**防護欄 (Guardrails)**
- 每次呼叫建立一個成品
- 在建立新成品之前，務必讀取相依成品
- 絕不跳過成品或不按順序建立
- 如果上下文不明確，在建立前詢問使用者
- 在標記進度之前，驗證寫入後的成品檔案是否存在
- 使用 Schema 的成品序列，不要假設特定的成品名稱
- **重要**：`context` 和 `rules` 是對你的約束，而不是檔案的內容
  - 不要將 `<context>`、`<rules>`、`<project_context>` 區塊複製到成品中
  - 這些指引你編寫內容，但不應出現在輸出中
