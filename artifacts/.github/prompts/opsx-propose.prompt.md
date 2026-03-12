---
description: 提案一個新變更 - 在一個步驟中建立並產生所有構件 (artifacts)
---

提案一個新變更 - 在一個步驟中建立變更並產生所有構件 (artifacts)。

我將建立一個包含以下構件的變更：
- proposal.md (做什麼 & 為什麼)
- design.md (如何做)
- tasks.md (實作步驟)

準備好實作時，執行 /opsx:apply

---

**輸入**：`/opsx:propose` 後方的參數是變更名稱 (kebab-case)，或者是使用者想要建置內容的描述。

**步驟**

1. **如果未提供輸入，詢問他們想要建置什麼**

   使用 **AskUserQuestion 工具** (開放式，無預設選項) 詢問：
   > "您想處理什麼變更？描述您想要建置或修正的內容。"

   從他們的描述中，推導出一個 kebab-case 名稱 (例如："新增使用者驗證" → `add-user-auth`)。

   **重要 (IMPORTANT)**：在不了解使用者想要建置什麼的情況下，請勿繼續。

2. **建立變更目錄**
   ```bash
   openspec new change "<name>"
   ```
   這會在 `openspec/changes/<name>/` 建立一個帶有 `.openspec.yaml` 的腳手架變更。

3. **取得構件建構順序**
   ```bash
   openspec status --change "<name>" --json
   ```
   解析 JSON 以取得：
   - `applyRequires`：實作前所需的構件 ID 陣列 (例如：`["tasks"]`)
   - `artifacts`：所有構件及其狀態與相依性的清單

4. **依序建立構件直到可套用 (apply-ready)**

   使用 **TodoWrite 工具** 追蹤構件的處理進度。

   依相依性順序循環處理構件 (首先處理無待處理相依性的構件)：

   a. **針對每個處於 `ready` 狀態 (相依性已滿足) 的構件**：
      - 取得指令：
        ```bash
        openspec instructions <artifact-id> --change "<name>" --json
        ```
      - 指令 JSON 包含：
        - `context`：專案背景 (對您的限制 - 請勿包含在輸出中)
        - `rules`：構件專屬規則 (對您的限制 - 請勿包含在輸出中)
        - `template`：用於輸出檔案的結構
        - `instruction`：此構件類型的架構專屬指引
        - `outputPath`：寫入構件的位置
        - `dependencies`：需讀取以了解背景的已完成構件
      - 讀取任何已完成的相依性檔案以了解背景
      - 使用 `template` 作為結構建立構件檔案
      - 將 `context` 和 `rules` 作為限制套用 - 但請勿將其複製到檔案中
      - 顯示簡短進度："已建立 <artifact-id>"

   b. **繼續直到所有 `applyRequires` 構件皆已完成**
      - 建立每個構件後，重新執行 `openspec status --change "<name>" --json`
      - 檢查 `applyRequires` 中的每個構件 ID 在構件陣列中是否具有 `status: "done"`
      - 當所有 `applyRequires` 構件都完成時停止

   c. **如果構件需要使用者輸入** (背景不明確)：
      - 使用 **AskUserQuestion 工具** 進行釐清
      - 然後繼續建立

5. **顯示最終狀態**
   ```bash
   openspec status --change "<name>"
   ```

**輸出**

完成所有構件後，進行摘要：
- 變更名稱與位置
- 已建立的構件清單及簡短描述
- 準備就緒情況："所有構件已建立！準備好進行實作。"
- 提示："執行 `/opsx:apply` 開始實作。"

**構件建立指引**

- 遵循每個構件類型的 `openspec instructions` 中的 `instruction` 欄位
- 架構定義了每個構件應包含的內容 - 請遵循它
- 在建立新構件前，讀取相依構件以了解背景
- 使用 `template` 作為輸出檔案的結構 - 填寫其章節
- **重要 (IMPORTANT)**：`context` 和 `rules` 是對您的限制，而非檔案內容
  - 請勿將 `<context>`, `<rules>`, `<project_context>` 區塊複製到構件中
  - 這些用於引導您的撰寫，但絕不應出現在輸出中

**護欄 (Guardrails)**
- 建立實作所需的所有構件 (由架構的 `apply.requires` 定義)
- 建立新構件前務必讀取相依構件
- 如果背景極度不明確，請詢問使用者 - 但偏好做出合理的決定以保持動力
- 如果已存在同名變更，詢問使用者是否要繼續或建立新變更
- 在繼續下一步前，確認寫入後每個構件檔案皆已存在
