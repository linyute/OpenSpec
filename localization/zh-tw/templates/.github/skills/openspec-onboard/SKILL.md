---
name: openspec-onboard
description: OpenSpec 引導式新手上路 - 透過旁白引導與實際程式碼庫工作完成一個完整的工作流循環。
license: MIT
compatibility: 需要 openspec CLI。
metadata:
  author: openspec
  version: "1.0"
  generatedBy: "1.1.1"
---

引導使用者完成他們的第一個完整的 OpenSpec 工作流循環。這是一個教學體驗——您將在他們的程式碼庫中執行實際工作，同時解釋每個步驟。

---

## Preflight

在開始之前，檢查 OpenSpec 是否已初始化：

```bash
openspec status --json 2>&1 || echo "NOT_INITIALIZED"
```

**如果未初始化：**
> 此專案尚未設定 OpenSpec。請先執行 `openspec init`，然後再回到 `/opsx:onboard`。

如果未初始化則在此停止。

---

## Phase 1: Welcome

顯示：

```
## 歡迎使用 OpenSpec！

我將引導您完成一個完整的變更循環——從構想到實作——使用您程式碼庫中的實際任務。在此過程中，您將透過實作來學習工作流。

**我們將要做的事：**
1. 在您的程式碼庫中挑選一個小型的實際任務
2. 簡要探索問題
3. 建立一個變更（我們工作的容器）
4. 建構 artifacts：proposal → specs → design → tasks
5. 實作任務
6. 封存已完成的變更

**時間：** ~15-20 分鐘

讓我們從尋找要處理的任務開始。
```

---

## Phase 2: Task Selection

### Codebase Analysis

掃描程式碼庫以尋找小型改進機會。尋找：

1. **TODO/FIXME 註解** - 在程式碼檔案中搜尋 `TODO`、`FIXME`、`HACK`、`XXX`
2. **缺失的錯誤處理** - 吞掉錯誤的 `catch` 區塊，沒有 try-catch 的風險操作
3. **沒有測試的函式** - 將 `src/` 與測試目錄進行交叉引用
4. **型別問題** - TypeScript 檔案中的 `any` 型別（`: any`、`as any`）
5. **偵錯成品** - 非偵錯程式碼中的 `console.log`、`console.debug`、`debugger` 陳述句
6. **缺失的驗證** - 沒有驗證的使用者輸入處理常式

同時檢查最近的 git 活動：
```bash
git log --oneline -10 2>/dev/null || echo "No git history"
```

### Present Suggestions

根據您的分析，提出 3-4 個具體建議：

```
## 任務建議

根據掃描您的程式碼庫，這裡有一些適合入門的任務：

**1. [最有希望的任務]**
   位置：`src/path/to/file.ts:42`
   範圍：~1-2 個檔案，~20-30 行
   為什麼它很好：[簡要原因]

**2. [第二個任務]**
   位置：`src/another/file.ts`
   範圍：~1 個檔案，~15 行
   為什麼它很好：[簡要原因]

**3. [第三個任務]**
   位置：[位置]
   範圍：[估計]
   為什麼它很好：[簡要原因]

**4. 其他事？**
   告訴我您想處理什麼。

您對哪個任務感興趣？（挑選一個數字或描述您自己的任務）
```

**If nothing found:** 退而求其次，詢問使用者想要建構什麼：
> 我在您的程式碼庫中沒有發現明顯的快速獲勝點。有什麼您一直想新增或修正的小任務嗎？

### Scope Guardrail

如果使用者挑選或描述了太大的內容（重大功能、需耗時多日的工作）：

```
那是一個很有價值的任務，但對於您第一次的 OpenSpec 演練來說，它可能超出了理想範圍。

為了學習工作流，越小越好——這可以讓您在不陷入實作細節的情況下看到完整的循環。

**選項：**
1. **切分得更小** - [他們的任務] 中最小且有用的部分是什麼？也許只是 [特定的切片]？
2. **挑選其他任務** - 其他建議之一，或其他小型任務？
3. **照做不誤** - 如果您真的想挑戰這個任務，我們可以。只需知道這會花費更長的時間。

您偏好哪一個？
```

如果使用者堅持，請讓他們覆寫——這是一個軟性的護欄。

---

## Phase 3: Explore Demo

一旦選定任務，簡要示範 explore 模式：

```
Before we create a change, let me quickly show you **explore mode**—it's how you think through problems before committing to a direction.
```

花 1-2 分鐘調查相關程式碼：
- 閱讀涉及的檔案
- 如果有助於釐清，畫一個簡單的 ASCII 圖表
- 記錄任何考量因素

```
## Quick Exploration

[您的簡要分析——您發現了什麼，任何考量因素]

┌─────────────────────────────────────────┐
│   [選配：如果有助於釐清，加入 ASCII 圖表]   │
└─────────────────────────────────────────┘

Explore 模式 (`/opsx:explore`) 用於這類思考——在實作前進行調查。您可以在需要思考問題時隨時使用它。

Now let's create a change to hold our work.
```

**PAUSE** - 在繼續之前等待使用者確認。

---

## Phase 4: Create the Change

**EXPLAIN：**
```
## Creating a Change

OpenSpec 中的「變更」是一個容器，用於存放圍繞一項工作的所有思考與規劃。它位於 `openspec/changes/<name>/` 並持有您的 artifacts——proposal、specs、design、tasks。

讓我為我們的任務建立一個。
```

**DO：** 使用衍生的 kebab-case 名稱建立變更：
```bash
openspec new change "<derived-name>"
```

**SHOW：**
```
Created: `openspec/changes/<name>/`

資料夾結構：
```
openspec/changes/<name>/
├── proposal.md    ← 我們為什麼要做這件事（空白，我們將填寫它）
├── design.md      ← 我們將如何建構它（空白）
├── specs/         ← 詳細需求（空白）
└── tasks.md       ← 實作檢查清單（空白）
```

Now let's fill in the first artifact—the proposal.
```

---

## Phase 5: Proposal

**EXPLAIN：**
```
## The Proposal

Proposal 記錄了我們**為什麼 (Why)** 要做這項變更，以及它在高階層面上**涉及什麼 (What)**。它是這項工作的「電梯簡報」。

我將根據我們的任務草擬一個。
```

**DO：** 草擬 proposal 內容（先不儲存）：

```
Here's a draft proposal:

---

## Why

[1-2 句話說明問題/機會]

## What Changes

[將會有所不同的項目列表]

## Capabilities

### New Capabilities
- `<capability-name>`: [簡要描述]

### Modified Capabilities
<!-- 如果修改現有行為 -->

## Impact

- `src/path/to/file.ts`: [變更內容]
- [其他檔案，如果適用]

---

Does this capture the intent? I can adjust before we save it.
```

**PAUSE** - 等待使用者核准/回饋。

核准後，儲存 proposal：
```bash
openspec instructions proposal --change "<name>" --json
```
然後將內容寫入 `openspec/changes/<name>/proposal.md`。

```
Proposal saved. This is your "why" document—you can always come back and refine it as understanding evolves.

Next up: specs.
```

---

## Phase 6: Specs

**EXPLAIN：**
```
## Specs

Specs 以精確、可測試的術語定義我們正在建構**什麼 (What)**。它們使用需求/情境格式，使預期行為變得清晰明瞭。

對於像這樣的小任務，我們可能只需要一個 spec 檔案。
```

**DO：** 建立 spec 檔案：
```bash
mkdir -p openspec/changes/<name>/specs/<capability-name>
```

草擬 spec 內容：

```
Here's the spec:

---

## ADDED Requirements

### Requirement: <名稱>

<系統應該做什麼的描述>

#### Scenario: <情境名稱>

- **WHEN** <觸發條件>
- **THEN** <預期結果>
- **AND** <額外的結果，如果需要>

---

This format—WHEN/THEN/AND—makes requirements testable. You can literally read them as test cases.
```

Save to `openspec/changes/<name>/specs/<capability>/spec.md`.

---

## Phase 7: Design

**EXPLAIN：**
```
## Design

Design 記錄了我們將**如何 (How)** 建構它——技術決策、權衡、方法。

對於小型變更，這可能很簡短。這沒關係——並非每個變更都需要深度的設計討論。
```

**DO：** 草擬 design.md：

```
Here's the design:

---

## Context

[關於目前狀態的簡要背景]

## Goals / Non-Goals

**目標：**
- [我們試圖實現的目標]

**非目標：**
- [明確排除在範圍之外的內容]

## Decisions

### Decision 1: [關鍵決策]

[對方法與理由的解釋]

---

For a small task, this captures the key decisions without over-engineering.
```

Save to `openspec/changes/<name>/design.md`.

---

## Phase 8: Tasks

**EXPLAIN：**
```
## Tasks

最後，我們將工作分解為實作任務——驅動實作階段的檢查清單。

這些應該要小巧、清晰，且具備邏輯順序。
```

**DO：** 根據 specs 與 design 產生任務：

```
Here are the implementation tasks:

---

## 1. [類別或檔案]

- [ ] 1.1 [具體任務]
- [ ] 1.2 [具體任務]

## 2. 驗證

- [ ] 2.1 [驗證步驟]

---

Each checkbox becomes a unit of work in the apply phase. Ready to implement?
```

**PAUSE** - 等待使用者確認他們已準備好進行實作。

Save to `openspec/changes/<name>/tasks.md`.

---

## Phase 9: Apply (實作)

**EXPLAIN：**
```
## Implementation

現在我們實作每個任務，並在完成後勾選它們。我會宣布每一項，並偶爾註記 specs/design 如何影響了實作方法。
```

**DO：** 對於每個任務：

1. Announce: "Working on task N: [description]"
2. 在程式碼庫中實作變更
3. 自然地引用 specs/design：「Spec 提到 X，所以我正在執行 Y」
4. 在 tasks.md 中標記完成：`- [ ]` → `- [x]`
5. 簡要狀態：「✓ 任務 N 已完成」

保持輕簡的旁白——不要過度解釋每一行程式碼。

在所有任務完成後：

```
## Implementation Complete

All tasks done:
- [x] Task 1
- [x] Task 2
- [x] ...

The change is implemented! One more step—let's archive it.
```

---

## Phase 10: Archive

**EXPLAIN：**
```
## Archiving

當變更完成後，我們將其封存。這會將它從 `openspec/changes/` 移動到 `openspec/changes/archive/YYYY-MM-DD-<name>/`。

封存的變更將成為您專案的決策歷史——您隨時可以找回它們，以了解為什麼某個功能是以特定方式建構的。
```

**DO：**
```bash
openspec archive "<name>"
```

**SHOW：**
```
Archived to: `openspec/changes/archive/YYYY-MM-DD-<name>/`

The change is now part of your project's history. The code is in your codebase, the decision record is preserved.
```

---

## Phase 11: Recap & Next Steps

```
## Congratulations!

您剛剛完成了一個完整的 OpenSpec 循環：

1. **Explore** - 深入思考問題
2. **New** - 建立變更容器
3. **Proposal** - 捕捉為什麼 (WHY)
4. **Specs** - 詳細定義什麼 (WHAT)
5. **Design** - 決定如何 (HOW)
6. **Tasks** - 將其分解為步驟
7. **Apply** - 實作工作
8. **Archive** - 保存記錄

同樣的節奏適用於任何規模的變更——無論是小修正還是重大功能。

---

## 指令參考

| 指令 | 用途 |
|---------|--------------|
| `/opsx:explore` | 在工作前/工作中思考問題 |
| `/opsx:new` | 開始新變更，引導建立 artifacts |
| `/opsx:ff` | 快速通關：一次建立所有 artifacts |
| `/opsx:continue` | 繼續處理現有的變更 |
| `/opsx:apply` | 實作變更中的任務 |
| `/opsx:verify` | 驗證實作是否符合 artifacts |
| `/opsx:archive` | 封存已完成的變更 |

---

## 接下來呢？

嘗試對您真正想建構的功能使用 `/opsx:new` 或 `/opsx:ff`。您現在已經掌握了這個節奏！
```

---

## Graceful Exit Handling

### 使用者想中途停止

如果使用者表示需要停止、想暫停，或看起來不感興趣：

```
No problem! Your change is saved at `openspec/changes/<name>/`.

稍後想從上次停下的地方繼續：
- `/opsx:continue <name>` - 恢復 artifact 建立
- `/opsx:apply <name>` - 跳轉至實作（如果已存在任務）

工作不會遺失。準備好後隨時回來。
```

優雅退出，不給予壓力。

### 使用者只想看指令參考

如果使用者表示他們只想看指令或跳過教學：

```
## OpenSpec Quick Reference

| 指令 | 用途 |
|---------|--------------|
| `/opsx:explore` | 思考問題（不變更程式碼） |
| `/opsx:new <name>` | 開始新變更，按部就班執行 |
| `/opsx:ff <name>` | 快速通關：一次完成所有 artifacts |
| `/opsx:continue <name>` | 繼續現有的變更 |
| `/opsx:apply <name>` | 實作任務 |
| `/opsx:verify <name>` | 驗證實作 |
| `/opsx:archive <name>` | 完成後封存 |

嘗試使用 `/opsx:new` 開始您的第一個變更，或者如果您想加快速度，請使用 `/opsx:ff`。
```

優雅退出。

---

## Guardrails

- 在關鍵轉換點（探索後、proposal 草擬後、任務完成後、封存後）**遵循 EXPLAIN → DO → SHOW → PAUSE 模式**
- 在實作過程中**保持輕簡的旁白**——教學但不說教
- 即使變更很小，也**不要跳過階段**——目標是教授工作流
- 在標記點**暫停以獲得確認**，但不要過度暫停
- **優雅處理退出**——絕不強迫使用者繼續
- **使用真實的程式碼庫任務**—— don't simulate or use fake examples
- **溫和地調整範圍**——引導至較小的任務，但尊重使用者的選擇
