---
name: openspec-new-change
description: 使用實驗性的 artifact 工作流開始一個新的 OpenSpec 變更。當使用者想要以結構化的逐步方法建立新功能、修復或修改時使用。
license: MIT
compatibility: 需要 openspec CLI。
metadata:
  author: openspec
  version: "1.0"
  generatedBy: "1.3.1"
---

使用實驗性的 artifact 驅動方法開始一個新的變更。

**輸入**：使用者的請求應包含變更名稱 (kebab-case) 或他們想要建構之內容的說明。

**步驟**

1. **若未提供明確輸入，詢問他們想要建構什麼**

   使用 **AskUserQuestion 工具**（開放式，無預設選項）來詢問：
   > 「您想要進行什麼變更？請說明您想要建構或修復的內容。」

   從他們的說明中，衍生出一個 kebab-case 名稱（例如：「add user authentication」→ `add-user-auth`）。

   **重要**：在不瞭解使用者想要建構什麼的情況下，請勿繼續。

2. **判斷工作流結構描述 (workflow schema)**

   使用預設結構描述（省略 `--schema`），除非使用者明確請求不同的工作流。

   **僅在使用者提到以下內容時使用不同的結構描述：**
   - 特定的結構描述名稱 → 使用 `--schema <name>`
   - 「顯示工作流」或「有哪些工作流」→ 執行 `openspec schemas --json` 並讓他們選擇

   **否則**：省略 `--schema` 以使用預設值。

3. **建立變更目錄**
   ```bash
   openspec new change "<name>"
   ```
   僅在使用者請求特定工作流時，才加入 `--schema <name>`。
   這將會在 `openspec/changes/<name>/` 以選定的結構描述建立一個鷹架變更。

4. **顯示 artifact 狀態**
   ```bash
   openspec status --change "<name>"
   ```
   這會顯示哪些 artifact 需要被建立，以及哪些已就緒（相依性已滿足）。

5. **取得第一個 artifact 的指示**
   第一個 artifact 取決於結構描述（例如：規格驅動的 `proposal`）。
   檢查狀態輸出以找到第一個狀態為「就緒 (ready)」的 artifact。
   ```bash
   openspec instructions <first-artifact-id> --change "<name>"
   ```
   這會輸出建立第一個 artifact 的範本和內容。

6. **停止並等待使用者指示**

**輸出**

完成步驟後，摘要如下：
- 變更名稱與位置
- 正在使用的結構描述/工作流及其 artifact 序列
- 目前狀態（0/N 個 artifact 已完成）
- 第一個 artifact 的範本
- 提示：「準備好建立第一個 artifact 了嗎？只要說明此變更的內容，我就會為您起草，或者要求我繼續執行。」

**防護準則 (Guardrails)**
- 尚不建立任何 artifact - 僅顯示指示
- 在顯示第一個 artifact 範本後請勿繼續進行
- 如果名稱無效（不是 kebab-case），請要求一個有效的名稱
- 如果已存在該名稱的變更，建議改為繼續該變更
- 如果使用非預設工作流，請傳遞 --schema
