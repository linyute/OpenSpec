---
description: 建立變更並一次產生實作所需的所有產出物
---

快速通關 (Fast-forward) 產出物建立流程 —— 產生開始實作所需的所有內容。

**輸入**：`/opsx:ff` 之後的引數為變更名稱（kebab-case 格式），或是使用者想要建構內容的描述。

**步驟**

1. **如果未提供輸入，則詢問使用者想要建構什麼**

   使用 **AskUserQuestion 工具**（開放式問題，無預設選項）詢問：
   > 「您想要處理什麼變更？請描述您想要建構或修正的內容。」

   根據使用者的描述，衍生出一個 kebab-case 名稱（例如：「add user authentication」→ `add-user-auth`）。

   **重要事項**：在不瞭解使用者想要建構什麼的情況下，請勿繼續。

2. **建立變更目錄**
   ```bash
   openspec new change "<name>"
   ```
   這會在 `openspec/changes/<name>/` 建立一個支架式的變更。

3. **取得產出物建構順序**
   ```bash
   openspec status --change "<name>" --json
   ```
   解析 JSON 以取得：
   - `applyRequires`：實作前所需的產出物 ID 陣列（例如：`["tasks"]`）
   - `artifacts`：所有產出物的清單及其狀態和相依性

4. **依序建立產出物，直到可以開始實作**

   使用 **TodoWrite 工具** 追蹤產出物的處理進度。

   按相依順序循環處理產出物（優先處理無擱置相依性的產出物）：

   a. **對於每個「就緒 (ready)」（相依性已滿足）的產出物**：
      - 取得指示：
        ```bash
        openspec instructions <artifact-id> --change "<name>" --json
        ```
      - 指示 JSON 包含：
        - `context`：專案背景（對您的約束 —— 請勿包含在輸出中）
        - `rules`：產出物特定規則（對您的約束 —— 請勿包含在輸出中）
        - `template`：輸出檔案要使用的結構
        - `instruction`：此產出物類期的結構定義 (schema) 特定指引
        - `outputPath`：寫入產出物的位置
        - `dependencies`：供參考內容的已完成產出物
      - 讀取任何已完成的相依檔案以取得內容
      - 使用 `template` 作為結構來建立產出物檔案
      - 將 `context` 和 `rules` 作為約束條件套用 —— 但請勿將它們複製到檔案中
      - 顯示簡短進度：「✓ 已建立 <artifact-id>」

   b. **持續執行，直到所有 `applyRequires` 產出物皆已完成**
      - 建立每個產出物後，重新執行 `openspec status --change "<name>" --json`
      - 檢查 `applyRequires` 中的每個產出物 ID 在 artifacts 陣列中的狀態是否皆為 `status: "done"`
      - 當所有 `applyRequires` 產出物皆完成時停止

   c. **如果產出物需要使用者輸入**（語境不明確）：
      - 使用 **AskUserQuestion 工具** 進行澄清
      - 然後繼續建立流程

5. **顯示最終狀態**
   ```bash
   openspec status --change "<name>"
   ```

**輸出**

完成所有產出物後，進行摘要：
- 變更名稱與位置
- 已建立的產出物清單及其簡短描述
- 就緒狀態：「所有產出物均已建立！準備好進行實作。」
- 提示：「執行 `/opsx:apply` 開始實作。」

**產出物建立指引**

- 針對每種產出物類型，遵循來自 `openspec instructions` 的 `instruction` 欄位
- 結構定義 (schema) 定義了每個產出物應包含的內容 —— 請遵循它
- 在建立新產出物之前，請讀取相依產出物以取得內容
- 使用 `template` 作為輸出檔案的結構 —— 填寫其中的章節
- **重要事項**：`context` 和 `rules` 是對您的約束，而非檔案內容
  - 請勿將 `<context>`、`<rules>`、`<project_context>` 區塊複製到產出物中
  - 這些是用來指引您的寫作，絕對不應出現在輸出中

**防護機制**
- 建立實作所需的所有產出物（如結構定義的 `apply.requires` 所定義）
- 在建立新產出物之前，務必讀取相依產出物
- 如果語境嚴重不明確，請詢問使用者 —— 但為了保持進度，優先做出合理的決定
- 如果已存在同名變更，請詢問使用者是要繼續該變更還是建立新變更
- 在繼續下一步之前，驗證寫入後的每個產出物檔案確實存在
