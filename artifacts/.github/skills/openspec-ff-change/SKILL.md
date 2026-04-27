---
name: openspec-ff-change
description: 快速進行 OpenSpec 成品建立。當使用者想要快速建立實作所需的所有成品，而不想逐一執行每個步驟時使用。
license: MIT
compatibility: 需要 openspec CLI。
metadata:
  author: openspec
  version: "1.0"
  generatedBy: "1.3.1"
---

快速進行成品建立 - 一次產生開始實作所需的所有內容。

**輸入**：使用者的請求應包含變更名稱 (kebab-case) 或他們想要建置內容的說明。

**步驟**

1. **如果未提供明確輸入，請詢問他們想要建置什麼**

   使用 **AskUserQuestion 工具**（開放式，無預設選項）詢問：
   > "您想要處理哪個變更？請說明您想要建置或修正的內容。"

   根據其說明，衍生出一個 kebab-case 名稱（例如："add user authentication" → `add-user-auth`）。

   **重要**：在不瞭解使用者想要建置什麼的情況下，請勿繼續。

2. **建立變更目錄**
   ```bash
   openspec new change "<name>"
   ```
   這會在 `openspec/changes/<name>/` 建立一個腳手架變更。

3. **獲取成品建構順序**
   ```bash
   openspec status --change "<name>" --json
   ```
   解析 JSON 以獲取：
   - `applyRequires`：實作前所需的成品 ID 陣列（例如：`["tasks"]`）
   - `artifacts`：所有成品的列表及其狀態與相依性

4. **依序建立成品，直到準備好套用**

   使用 **TodoWrite 工具** 追蹤成品的進度。

   按相依性順序迴圈處理成品（無待處理相依性的成品優先）：

   a. **對於每個處於 `ready` 狀態（相依性已滿足）的成品**：
      - 獲取指示：
        ```bash
        openspec instructions <artifact-id> --change "<name>" --json
        ```
      - 指示 JSON 包含：
        - `context`：專案背景（對您的約束 - 請勿包含在輸出中）
        - `rules`：成品特定規則（對您的約束 - 請勿包含在輸出中）
        - `template`：用於輸出檔案的結構
        - `instruction`：Schema 特定的成品類型指引
        - `outputPath`：寫入成品的位置
        - `dependencies`：要讀取以獲取背景資訊的已完成成品
      - 讀取任何已完成的相依性檔案以獲取背景資訊
      - 使用 `template` 作為結構建立成品檔案
      - 將 `context` 和 `rules` 作為約束套用 - 但請勿將其複製到檔案中
      - 顯示簡短進度："✓ 已建立 <artifact-id>"

   b. **繼續直到所有 `applyRequires` 成品皆已完成**
      - 建立每個成品後，重新執行 `openspec status --change "<name>" --json`
      - 檢查 `applyRequires` 中的每個成品 ID 在 artifacts 陣列中的 `status` 是否為 `"done"`
      - 當所有 `applyRequires` 成品皆完成時停止

   c. **如果成品需要使用者輸入**（背景資訊不明確）：
      - 使用 **AskUserQuestion 工具** 進行釐清
      - 然後繼續建立

5. **顯示最終狀態**
   ```bash
   openspec status --change "<name>"
   ```

**輸出**

完成所有成品後，進行總結：
- 變更名稱與位置
- 已建立的成品列表及簡短說明
- 準備就緒內容："所有成品已建立！準備好進行實作。"
- 提示："執行 `/opsx:apply` 或要求我實作以開始處理任務。"

**成品建立指南**

- 遵循每個成品類型的 `openspec instructions` 中的 `instruction` 欄位
- Schema 定義了每個成品應包含的內容 - 請遵循它
- 在建立新成品之前，讀取相依集成品以獲取背景資訊
- 使用 `template` 作為輸出檔案的結構 - 填寫其區段
- **重要**：`context` 和 `rules` 是對您的約束，而不是檔案的內容
  - 請勿將 `<context>`、`<rules>`、`<project_context>` 區塊複製到成品中
  - 這些會引導您編寫內容，但不應出現在輸出中

**防護欄**
- 建立實作所需的所有成品（如 Schema 的 `apply.requires` 所定義）
- 在建立新成品之前，務必讀取相依集成品
- 如果背景資訊極度不明確，請詢問使用者 - 但優先做出合理的決定以保持進度
- 如果已存在同名變更，請建議繼續該變更
- 寫入後，在繼續下一步之前，驗證每個成品檔案是否存在
