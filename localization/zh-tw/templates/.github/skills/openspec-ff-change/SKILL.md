---
name: openspec-ff-change
description: 快速通關 (Fast-forward) OpenSpec artifact 建立過程。當使用者想要快速建立實作所需的所有 artifacts，而不必逐一執行每個步驟時使用。
license: MIT
compatibility: 需要 openspec CLI。
metadata:
  author: openspec
  version: "1.0"
  generatedBy: "1.1.1"
---

快速通關 (Fast-forward) artifact 建立過程 - 一次產生開始實作所需的所有內容。

**輸入**：使用者的請求應包含一個變更名稱（kebab-case）或者是對其想要建構內容的描述。

**步驟**

1. **如果未提供清晰的輸入，詢問他們想要建構什麼**

   使用 **AskUserQuestion 工具**（開放式，無預設選項）詢問：
   > 「您想要處理什麼變更？請描述您想要建構或修正的內容。」

   根據他們的描述，推導出 kebab-case 名稱（例如，「新增使用者驗證」→ `add-user-auth`）。

   **重要**：在未瞭解使用者想要建構什麼之前，請勿繼續。

2. **建立變更目錄**
   ```bash
   openspec new change "<name>"
   ```
   這會在 `openspec/changes/<name>/` 建立 scaffolded 變更。

3. **取得 artifact 建立順序**
   ```bash
   openspec status --change "<name>" --json
   ```
   解析 JSON 以取得：
   - `applyRequires`：實作前所需的 artifact IDs 陣列（例如 `["tasks"]`）
   - `artifacts`：所有 artifacts 的列表及其狀態與相依性

4. **依序建立 artifacts 直到可供實作 (apply-ready)**

   使用 **TodoWrite 工具** 追蹤 artifacts 的進度。

   依相依性順序循環處理 artifacts（優先處理無待處理相依性的 artifacts）：

   a. **針對每個狀態為 `ready`（相依性已滿足）的 artifact**：
      - 取得指令：
        ```bash
        openspec instructions <artifact-id> --change "<name>" --json
        ```
      - 指令 JSON 包含：
        - `context`：專案背景（對您的約束 - **不要**包含在輸出中）
        - `rules`：Artifact 專屬規則（對您的約束 - **不要**包含在輸出中）
        - `template`：用於輸出檔案的結構
        - `instruction`：針對此 artifact 類型的 schema 專屬引導
        - `outputPath`：寫入 artifact 的位置
        - `dependencies`：需讀取以獲取背景資訊的已完成 artifacts
      - 讀取任何已完成的相依檔案以獲取背景資訊
      - 使用 `template` 作為結構建立 artifact 檔案
      - 將 `context` 與 `rules` 作為約束條件套用 - 但**不要**將它們複製到檔案中
      - 顯示簡要進度：「✓ 已建立 <artifact-id>」

   b. **持續執行直到所有 `applyRequires` 中的 artifacts 皆已完成**
      - 建立每個 artifact 後，重新執行 `openspec status --change "<name>" --json`
      - 檢查 `applyRequires` 中的每個 artifact ID 在 artifacts 陣列中的 `status` 是否皆為 `"done"`
      - 當所有 `applyRequires` 中的 artifacts 皆完成時停止

   c. **如果 artifact 需要使用者輸入**（背景資訊不明確）：
      - 使用 **AskUserQuestion 工具** 進行釐清
      - 然後繼續建立

5. **顯示最終狀態**
   ```bash
   openspec status --change "<name>"
   ```

**輸出**

完成所有 artifacts 後，進行摘要：
- 變更名稱與位置
- 已建立的 artifacts 列表及其簡要描述
- 就緒狀態：「所有 artifacts 已建立！已準備好進行實作。」
- 提示：「執行 `/opsx:apply` 或要求我開始實作以處理任務。」

**Artifact 建立指南**

- 針對每種 artifact 類型，遵循 `openspec instructions` 中的 `instruction` 欄位
- Schema 定義了每個 artifact 應包含的內容 - 請遵循它
- 在建立新的 artifact 之前，讀取相依的 artifacts 以獲取背景資訊
- 使用 `template` 作為輸出檔案的結構 - 填寫其章節
- **重要**：`context` 與 `rules` 是對您的約束，而非檔案內容
  - **不要**將 `<context>`、`<rules>`、`<project_context>` 區塊複製到 artifact 中
  - 這些用於引導您編寫內容，但絕不應出現在輸出中

**Guardrails**
- 建立實作所需的所有 artifacts（由 schema 的 `apply.requires` 定義）
- 在建立新的 artifact 之前，務必讀取相依的 artifacts
- 如果背景資訊極度不明確，請詢問使用者 - 但優先做出合理的決定以保持動力
- 如果已存在同名變更，建議改為繼續該變更
- 在繼續下一步之前，驗證寫入後的每個 artifact 檔案是否存在
