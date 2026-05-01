# 工作區重新實作方向

日期：2026-04-30

Fresh-agent 進入點：請先閱讀 `WORKSPACE_REIMPLEMENTATION_START_HERE.md`，然後返回此文件以獲取完整的產品方向。

本文件記錄了根據我們從工作區 POC 中學到的經驗，從頭開始重新實作 OpenSpec 工作區支援的預期方向。

重新實作應圍繞著真實使用者使用 OpenSpec 的路徑進行排序：

```text
建立工作區
  -> 新增存放庫
  -> 開啟工作區
  -> 跨存放庫探索
  -> 建立提案
  -> 套用一個存放庫切片
  -> 驗證
  -> 封存
```

目標不是重建每個 POC 機制。目標是每次完成一個面向使用者的功能，順序與使用者自然建立、實作、驗證和封存變更的順序相同。

## 北極星（North Star）

使用者應該這樣想：

```text
我有一個多存放庫的產品目標。
我建立了一個 OpenSpec 工作區。
我用我的代理程式開啟它。
代理程式可以看到已註冊的存放庫。
我們進行探索，直到範圍明確。
然後我們建立一個提案。
然後我們一次實作一個存放庫切片。
```

他們不應該這樣想：

```text
我需要建立一個變更，以便讓存放庫變得可見。
我需要產生存放庫本地的成品（artifacts）。
我需要理解工作區疊加層（overlays）。
我需要與提案檔案分開管理目標 Metadata。
```

核心產品規則是：

```text
存放庫的可見性並非變更承諾。
```

已註冊的存放庫是工作區的工作集。建立變演是計劃承諾。套用變演是實作工作流程。

## 建構順序

### 1. 工作區建立

首先讓工作區建立變得乏味且穩固。

使用者目標：

```text
建立一個跨存放庫計劃存放的地方。
```

預期表面：

```bash
openspec workspace create my-workspace
openspec workspace add-repo openspec /path/to/openspec
openspec workspace add-repo landing /path/to/openspec-landing
```

預期結果：

```text
workspace/
  AGENTS.md
  changes/
  .openspec-workspace/
```

產品決策：

- 使用 `.openspec-workspace/` 而非 `.openspec/` 作為工作區 Metadata。
- 保持 `changes/` 在工作區根目錄可見。
- 將已註冊的存放庫視為工作區的工作集。
- 讓 `doctor` 顯示人類可讀的存放庫名稱和解析後的路徑。

延後處理：

- 分支。
- 工作樹（Worktrees）。
- 套用。
- 封存。
- 複雜的目標生命週期。

當使用者可以建立工作區、註冊存放庫並執行 `doctor` 以精確檢視 OpenSpec 所知道的資訊時，即告完成。

### 2. 工作區開啟

接下來讓工作區可以按照使用者期望的方式開啟。

使用者目標：

```text
使用我的開發代理程式開啟這個多存放庫的工作集。
```

預期表面：

```bash
openspec workspace open
openspec workspace open --agent codex
openspec workspace open --agent github-copilot
```

產品行為：

- `workspace open` 會開啟協調工作區以及已註冊的存放庫。
- 存放庫可見性是預設的。
- 變更選擇是選用的焦點，而非存取存放庫的機制。
- `--agent` 預設應該是單次作業階段的覆蓋。持久儲存偏好的代理程式應需要明確的偏好設定操作。

對於 GitHub Copilot，產生或開啟一個包含以下內容的 `.code-workspace` 檔案：

```text
工作區根目錄
已註冊的存放庫 A
已註冊的存放庫 B
```

對於 Claude 和 Codex，透過代理程式支援的機制掛載已註冊的存放庫目錄。

延後處理：

- `workspace open --change`。
- 作業階段內的升級流程。
- 針對個別變更的掛載限制。

當開啟工作區能讓代理程式檢視到協調根目錄和所有已註冊的存放庫時，即告完成。

### 3. 代理程式指引與探索

接著讓探索功能運作。

使用者目標：

```text
告訴代理程式一個粗略的產品目標，並讓它在建立提案之前檢查存放庫。
```

預期使用者提示：

```text
探索我們應該如何讓 OpenSpec 文件在登陸頁面（landing page）上可用。
查看已註冊的存放庫，但先不要實作。
```

代理程式行為：

- 理解其處於工作區模式。
- 檢查已註冊的存放庫。
- 解釋可能受影響的存放庫。
- 僅在需要時要求澄清。
- 在探索期間避免進行實作編輯。

建構：

- 工作區層級的 `AGENTS.md` 指引。
- 工作區作業階段中的常規 OpenSpec 技能和指令。
- 疊加在常規 `/explore` 之上的工作區特定指引，而非取代它。

延後處理：

- 提案成品產生。
- 目標確認指令。
- 套用內容（apply context）提供者。

當使用者可以開啟工作區並執行有用的跨存放庫探索，而無需建立虛擬變更時，即告完成。

### 4. 提案建立

只有在探索功能運作後，才建構提案建立。

使用者目標：

```text
現在我們已經瞭解了範圍，請擷取計劃。
```

預期使用者提示：

```text
為此變更建立一個提案。
針對實際受影響的存放庫。
```

偏好的成品形狀：

```text
changes/integrate-docs/
  proposal.md
  design.md
  tasks.md
  specs/
    openspec/
      docs-conventions/spec.md
    landing/
      docs-routing/spec.md
```

關鍵工作流程規則：

```text
/explore 可能會讓目標保持未知。
/propose 可能會發現目標。
/propose 在表示準備好套用之前，必須確認目標。
```

目標應盡可能由提案成品本身表示。如果有 `specs/landing/...`，那麼 `landing` 就在範圍內。避免使用單獨要求的 `targets: [...]` Metadata 清單作為作用中的事實來源。

延後處理：

- 存放庫本地實體化。
- 工作樹選擇。
- 多存放庫實作。
- 封存。

當使用者可以探索，然後建立一個帶有存放庫範圍規格（specs）和工作的工作區提案時，即告完成。

### 5. 狀態（Status）

在實作之前，讓狀態功能變得卓越。

使用者目標：

```text
我們在哪裡，涉及哪些存放庫，以及是否準備好實作？
```

預期表面：

```bash
openspec status
openspec status --change integrate-docs
```

人類可讀的輸出應回答：

```text
變更：integrate-docs
範圍：openspec, landing
提案：存在
設計：存在
工作：存在
是否準備好套用：是/否
```

狀態還應該捕捉結構性錯誤：

- `specs/` 下未知的存放庫資料夾。
- 缺失工作。
- 沒有確認的受影響存放庫。
- 已註冊的存放庫路徑缺失。

當代理程式和使用者可以在套用前信任狀態時，即告完成。

### 6. 套用一個存放庫切片

現在才建構 `/apply`。

使用者目標：

```text
為一個存放庫實作計劃中的切片。
```

預期使用者提示：

```text
/apply integrate-docs for landing
```

產品合約：

```text
/apply 意味著實作。
```

它並不意味著：

```text
複製計劃檔案
實體化存放庫本地的 OpenSpec 狀態
首次建立提案檔案
```

代理程式行為：

1. 向 OpenSpec 要求套用內容（apply context）。
2. 閱讀提案、設計、工作和相關規格。
3. 確認目標存放庫的檢出（checkout）。
4. 僅編輯該存放庫。
5. 更新工作區工作。
6. 執行相關檢查。

這在內部可能需要一個標準化的內容指令，但那是支援性機制：

```json
{
  "mode": "workspace",
  "change": "integrate-docs",
  "target": "landing",
  "implementationRoot": "/repos/openspec-landing",
  "contextFiles": [
    "changes/integrate-docs/proposal.md",
    "changes/integrate-docs/design.md",
    "changes/integrate-docs/tasks.md",
    "changes/integrate-docs/specs/landing/docs-routing/spec.md"
  ],
  "allowedEditRoots": [
    "/repos/openspec-landing"
  ],
  "tasksFile": "changes/integrate-docs/tasks.md"
}
```

延後處理：

- 一次套用多個存放庫。
- 自動建立分支。
- 工作樹管理。
- 存放庫本地 OpenSpec 鏡像。

當一個存放庫切片可以從中央工作區計劃中實作時，即告完成。

### 7. 驗證

然後建構驗證功能。

使用者目標：

```text
檢查已實作的存放庫切片是否符合計劃。
```

預期提示：

```text
/verify integrate-docs for landing
```

行為：

- 讀取與 `/apply` 相同的標準化內容。
- 檢查實作的檢出。
- 檢查該存放庫的工作和規格。
- 執行存放庫驗證。
- 清晰回報差距。

預設行為應驗證一個存放庫切片。整個工作區的驗證可以稍後再進行。

當使用者可以針對中央工作區計劃驗證一個已實作的存放庫切片時，即告完成。

### 8. 封存

封存在第一個完整迴圈中放在最後。

使用者目標：

```text
變更已完成。將其移出作用中的計劃。
```

預期提示：

```text
/archive integrate-docs
```

行為：

- 要求所有目標存放庫切片都已完成或明確接受。
- 封存工作區變更。
- 除非 OpenSpec 稍後決定存放庫本地封存很重要，否則不要求存放庫本地計劃副本。

當使用者可以完成整個生命週期時，即告完成：

```text
工作區建立
  -> 開啟
  -> 探索
  -> 提案
  -> 套用存放庫 A
  -> 套用存放庫 B
  -> 驗證
  -> 封存
```

## 實作紀律

僅建構下一個使用者可見的步驟。

順序應始終基於這些問題：

```text
1. 我可以建立工作區嗎？
2. 我可以看到我的存放庫嗎？
3. 我的代理程式可以探索它們嗎？
4. 我們可以擷取提案嗎？
5. 狀態可以告訴我們它是否準備好了嗎？
6. 代理程式可以實作一個存放庫切片嗎？
7. 我們可以驗證它嗎？
8. 我們可以封存它嗎？
```

除非下一個使用者可見的功能需要內部抽象，否則請避免從內部抽象開始。

不要從以下內容開始：

- 目標 Metadata 機制。
- 實體化。
- 配接器（Adapter）抽象。
- 分支協調。
- 工作樹協調。
- 多存放庫套用。

這些在以後可能很重要，但它們不應定義第一個重新實作路徑。

## 產品形狀

工作區應該感覺像是跨越多個存放庫的 OpenSpec 常規工作流程，而不是具有其自身生命週期的第二個產品。

持久的產品模型是：

```text
工作區 = 中央計劃事實來源
已註冊的存放庫 = 可見的工作集
提案 = 範圍明確的計劃承諾
存放庫目標 = 計劃中一個受影響的存放庫
分支/工作樹 = 實作檢出
/apply = 實作一個選定的存放庫切片
```

保持使用者旅程簡單：

```text
開啟工作區。
要求代理程式進行探索。
當範圍明確時建立提案。
一次實作一個存放庫切片。
驗證。
封存。
```
