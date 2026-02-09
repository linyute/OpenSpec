---
name: openspec-apply-change
description: 實作 OpenSpec 變更中的任務。當使用者想要開始實作、繼續實作或處理任務時使用。
license: MIT
compatibility: 需要 openspec CLI。
metadata:
  author: openspec
  version: "1.0"
  generatedBy: "1.1.1"
---

實作 OpenSpec 變更中的任務。

**輸入**：可在指令後選配指定變更名稱。如果省略，請檢查是否可以從對話內容中推斷。如果模糊不清或不明確，您必須提示使用者選擇可用的變更。

**步驟**

1. **選擇變更**

   如果提供了名稱，則使用它。否則：
   - 如果使用者提到了某個變更，從對話內容中推斷
   - 如果僅存在一個作用中的變更，則自動選擇
   - 如果模糊不清，執行 `openspec list --json` 以取得可用的變更，並使用 **AskUserQuestion 工具** 讓使用者選擇

   務必宣布：「使用變更：<名稱>」以及如何覆寫（例如 `/opsx:apply <其他名稱>`）。

2. **檢查狀態以瞭解 schema**
   ```bash
   openspec status --change "<name>" --json
   ```
   解析 JSON 以瞭解：
   - `schemaName`：正在使用的工作流（例如 "spec-driven"）
   - 哪個 artifact 包含任務（對於 spec-driven 通常是 "tasks"，其他請檢查狀態）

3. **取得實作 (apply) 指令**

   ```bash
   openspec instructions apply --change "<name>" --json
   ```

   這會回傳：
   - 內容檔案路徑（因 schema 而異 - 可能是 proposal/specs/design/tasks 或 spec/tests/implementation/docs）
   - 進度（總計、已完成、剩餘）
   - 附帶狀態的任務列表
   - 根據目前狀態提供的動態指令

   **處理狀態：**
   - 如果 `state: "blocked"`（缺失 artifacts）：顯示訊息，建議使用 `openspec-continue-change`
   - 如果 `state: "all_done"`：表示祝賀，建議進行封存
   - 否則：繼續進行實作

4. **讀取背景檔案**

   讀取實作指令輸出中 `contextFiles` 列表內的檔案。
   檔案取決於所使用的 schema：
   - **spec-driven**：proposal, specs, design, tasks
   - 其他 schemas：遵循 CLI 輸出的 `contextFiles`

5. **顯示目前進度**

   顯示：
   - 正在使用的 schema
   - 進度：「N/M 個任務已完成」
   - 剩餘任務概覽
   - 來自 CLI 的動態指令

6. **實作任務（循環執行直到完成或被阻擋）**

   針對每個待處理任務：
   - 顯示正在處理哪個任務
   - 執行所需的程式碼變更
   - 保持變更精簡且專注
   - 在任務檔案中將任務標記為完成：`- [ ]` → `- [x]`
   - 繼續執行下一個任務

   **在以下情況暫停：**
   - 任務不明確 → 要求澄清
   - 實作過程中發現設計問題 → 建議更新 artifacts
   - 遇到錯誤或阻礙 → 回報並等待引導
   - 使用者中斷

7. **完成或暫停時，顯示狀態**

   顯示：
   - 本次階段完成的任務
   - 整體進度：「N/M 個任務已完成」
   - 如果全部完成：建議進行封存
   - 如果暫停：解釋原因並等待引導

**實作過程中的輸出**

```
## 正在實作：<change-name> (schema: <schema-name>)

正在處理任務 3/7：<任務描述>
[...實作正在進行中...]
✓ 任務已完成

正在處理任務 4/7：<任務描述>
[...實作正在進行中...]
✓ 任務已完成
```

**完成時的輸出**

```
## 實作完成

**變更：** <change-name>
**Schema：** <schema-name>
**進度：** 7/7 個任務已完成 ✓

### 本次階段已完成
- [x] 任務 1
- [x] 任務 2
...

所有任務皆已完成！已準備好封存此變更。
```

**暫停時的輸出（遇到問題）**

```
## 實作暫停

**變更：** <change-name>
**Schema：** <schema-name>
**進度：** 4/7 個任務已完成

### 遇到的問題
<問題的描述>

**選項：**
1. <選項 1>
2. <選項 2>
3. 其他方法

您想要怎麼做？
```

**Guardrails**
- 持續處理任務直到完成或被阻擋
- 開始前務必讀取背景檔案（來自實作指令輸出）
- 如果任務模糊不清，請在實作前暫停並詢問
- 如果實作揭示了問題，請暫停並建議更新 artifacts
- 保持程式碼變更精簡，且範圍限縮於每個任務
- 完成每個任務後立即更新任務核取方塊
- 遇到錯誤、阻礙或不明確的需求時請暫停——不要猜測
- 使用來自 CLI 輸出的 `contextFiles`，不要假設特定的檔案名稱

**流暢的工作流整合**

此技能支援「對變更採取行動」模式：

- **可隨時呼叫**：在所有 artifacts 完成前（如果已存在任務）、部分實作後、與其他操作交替執行
- **允許更新 artifact**：如果實作揭示了設計問題，建議更新 artifacts - 不受階段鎖定，靈活工作
