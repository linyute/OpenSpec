---
name: openspec-new-change
description: 使用實驗性 artifact 工作流開始一個新的 OpenSpec 變更。當使用者想要透過結構化的逐步方法來建立新功能、修正或修改時使用。
license: MIT
compatibility: 需要 openspec CLI。
metadata:
  author: openspec
  version: "1.0"
  generatedBy: "1.1.1"
---

使用實驗性 artifact 驅動方法開始新變更。

**輸入**：使用者的請求應包含一個變更名稱（kebab-case）或者是對其想要建構內容的描述。

**步驟**

1. **如果未提供清晰的輸入，詢問他們想要建構什麼**

   使用 **AskUserQuestion 工具**（開放式，無預設選項）詢問：
   > 「您想要處理什麼變更？請描述您想要建構或修正的內容。」

   根據他們的描述，推導出 kebab-case 名稱（例如，「新增使用者驗證」→ `add-user-auth`）。

   **重要**：在未瞭解使用者想要建構什麼之前，請勿繼續。

2. **確定工作流 schema**

   除非使用者明確要求不同的工作流，否則請使用預設 schema（省略 `--schema`）。

   **僅在使用者提到以下內容時使用不同的 schema：**
   - 特定的 schema 名稱 → 使用 `--schema <name>`
   - 「顯示工作流 (show workflows)」或「有哪些工作流 (what workflows)」 → 執行 `openspec schemas --json` 並讓他們選擇

   **否則**：省略 `--schema` 以使用預設值。

3. **建立變更目錄**
   ```bash
   openspec new change "<name>"
   ```
   僅在使用者要求特定工作流時才加入 `--schema <name>`。
   這會使用選定的 schema 在 `openspec/changes/<name>/` 建立 scaffolded 變更。

4. **顯示 artifact 狀態**
   ```bash
   openspec status --change "<name>"
   ```
   這會顯示需要建立哪些 artifacts，以及哪些已就緒（相依性已滿足）。

5. **取得第一個 artifact 的指令**
   第一個 artifact 取決於 schema（例如對於 spec-driven 為 `proposal`）。
   檢查狀態輸出以找到第一個狀態為 "ready" 的 artifact。
   ```bash
   openspec instructions <first-artifact-id> --change "<name>"
   ```
   這會輸出建立第一個 artifact 的模板與背景資訊。

6. **停止並等待使用者指示**

**輸出**

完成步驟後，進行摘要：
- 變更名稱與位置
- 正在使用的 Schema/工作流及其 artifact 序列
- 目前狀態（0/N 個 artifacts 已完成）
- 第一個 artifact 的模板
- 提示：「準備好建立第一個 artifact 了嗎？直接描述此變更的內容，我會為您草擬，或者要求我繼續。」

**Guardrails**
- 尚未建立任何 artifacts - 僅顯示指令
- 除了顯示第一個 artifact 模板外，不要再進一步操作
- 如果名稱無效（不是 kebab-case），要求提供有效名稱
- 如果已存在同名變更，建議改為繼續該變更
- 如果使用非預設工作流，請傳遞 `--schema`
