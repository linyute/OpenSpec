# 工作流 (Workflows)

本指南涵蓋了 OpenSpec 的常見工作流模式以及何時使用各個模式。有關基本設定，請參閱 [入門指南 (Getting Started)](getting-started.md)。有關命令參考，請參閱 [命令 (Commands)](commands.md)。

## 理念：行動而非階段

傳統的工作流強制您經過多個階段：規劃、實作，然後完成。但實際工作並非總是能整齊地歸類。

OPSX 採取了不同的方法：

```text
傳統工作流 (階段鎖定)：

  規劃 ────────► 實作 ────────► 完成
    │             │
    │  「無法回頭」 │
    └─────────────┘

OPSX (流動行動)：

  提案 (proposal) ──→ 規格 (specs) ──→ 設計 (design) ──→ 任務 (tasks) ──→ 實作 (implement)
```

**關鍵原則：**

- **行動而非階段** - 命令是您可以隨時執行的動作，而非您被困住的階段。
- **依賴關係是促成因素** - 它們顯示了目前可以建立哪些內容，而非下一步「必須」建立什麼。

> **自訂：** OPSX 工作流由定義成品順序的結構描述 (schemas) 驅動。詳情請參閱 [自訂 (Customization)](customization.md) 以了解如何建立自訂結構描述。

## 兩種模式

### 預設快速路徑 (`core` 設定檔)

全新安裝預設使用 `core`，提供以下命令：
- `/opsx:propose`
- `/opsx:explore`
- `/opsx:apply`
- `/opsx:archive`

典型的流程：

```text
/opsx:propose ──► /opsx:apply ──► /opsx:archive
```

### 擴展/完整工作流 (自訂選擇)

如果您想要明確的「搭建鷹架並建構」命令 (`/opsx:new`, `/opsx:continue`, `/opsx:ff`, `/opsx:verify`, `/opsx:sync`, `/opsx:bulk-archive`, `/opsx:onboard`)，請透過以下方式啟用：

```bash
openspec config profile
openspec update
```

## 工作流模式 (擴展模式)

### 快速開發功能

當您知道要建構什麼，只需要執行時：

```text
/opsx:new ──► /opsx:ff ──► /opsx:apply ──► /opsx:verify ──► /opsx:archive
```

**範例對話：**

```text
您：/opsx:new add-logout-button

AI：已建立 openspec/changes/add-logout-button/
     準備好建立：proposal

您：/opsx:ff

AI：正在建立所有規劃成品...
     ✓ proposal.md
     ✓ specs/
     ✓ design.md
     ✓ tasks.md
     準備好進行實作！

您：/opsx:apply

AI：正在處理任務...
     ✓ 已在頁首新增登出按鈕
     ✓ 已實作清除工作階段
     ✓ 已新增確認對話框
     所有任務已完成！

您：/opsx:archive

AI：✓ 已合併規格
    ✓ 已封存變更
```

**最適合：** 中小型功能、錯誤修復、直接明確的變更。

### 探索性開發

當需求不明確，或您需要先進行調查時：

```text
/opsx:explore ──► /opsx:new ──► /opsx:continue ──► ... ──► /opsx:apply
```

**範例對話：**

```text
您：/opsx:explore

AI：您想探索什麼？

您：我想改進頁面載入效能，但不確定瓶頸在哪裡。

AI：讓我協助調查...
     [分析組合包大小、識別緩慢查詢、檢視元件渲染模式]

     我發現了三個主要瓶頸：
     1. 未經優化的大型圖片
     2. ProductList 中的同步資料獲取
     3. 來自 Context 變更的重新渲染

     您想先處理哪一個？

您：我們先處理資料獲取的問題。

您：/opsx:new optimize-product-list-fetching

AI：已建立變更。準備好建立：proposal

您：/opsx:continue

AI：正在根據我們的探索內容建立提案...
     [將調查見解納入提案撰寫]

     準備好建立：specs, design
     您接下來想建立哪一個？

您：/opsx:continue

AI：正在建立規格...
```

**最適合：** 效能優化、除錯、架構決策、不明確的需求。

### 平行變更

同時處理多個變更：

```text
變更 A：/opsx:new ──► /opsx:ff ──► /opsx:apply (進行中)
                                         │
                                    上下文切換
                                         │
變更 B：/opsx:new ──► /opsx:ff ──────► /opsx:apply
```

**範例對話：**

```text
您：我正在處理 add-dark-mode，但被拉去修復一個錯誤

您：/opsx:new fix-login-redirect

AI：已建立 openspec/changes/fix-login-redirect/

您：/opsx:ff

AI：✓ 已建立規劃成品

您：/opsx:apply

AI：正在處理 fix-login-redirect...
     [實作修復]
     所有任務已完成！

您：/opsx:archive

AI：✓ 已封存 fix-login-redirect

您：我們回到深色模式吧

您：/opsx:apply add-dark-mode

AI：恢復 add-dark-mode...
     從任務 2.3 開始：更新頁首 (Header)...
```

**最適合：** 平行工作流、緊急中斷、團隊協作。

當您有多個已完成的變更時，請使用 `/opsx:bulk-archive`：

```text
您：/opsx:bulk-archive

AI：發現 3 個已完成的變更：
     - add-dark-mode (所有任務已完成)
     - fix-login-redirect (所有任務已完成)
     - update-footer (所有任務已完成)

     正在檢查規格衝突...
     ⚠ add-dark-mode 和 update-footer 皆觸及了 specs/ui/

     正在調查程式碼庫以解決衝突...
     兩項變更皆已實作。將按時間順序套用。

     要封存所有 3 個變更嗎？

您：是

AI：✓ 已封存 add-dark-mode
     ✓ 已封存 fix-login-redirect
     ✓ 已封存 update-footer
     規格合併順序：add-dark-mode → update-footer
```

批量封存 (Bulk archive) 會偵測多個變更是否觸及相同的規格，並藉由檢查實際實作的內容來解決衝突。

### 完成變更

建議的完成流程：

```text
/opsx:apply ──► /opsx:verify ──► /opsx:archive
                    │                 │
              驗證實作內容       若需要則提示進行
                                 規格同步
```

#### 驗證：檢查您的工作

`/opsx:verify` 從三個維度驗證實作是否符合您的成品內容：

```text
您：/opsx:verify

AI：正在驗證 add-auth...

     完整性 (COMPLETENESS)
     ✓ tasks.md 中的所有 12 個任務皆已勾選
     ✓ 規格中的所有需求皆有對應的程式碼
     ⚠ 情境「閒置後工作階段逾時」未經測試

     正確性 (CORRECTNESS)
     ✓ 實作符合規格意圖
     ✓ 已處理情境中的邊緣案例
     ✓ 錯誤狀態符合規格定義

     連貫性 (COHERENCE)
     ✓ 技術決策反映在程式碼結構中
     ✓ 命名慣例與 design.md 一致
     ⚠ 技術方案 (Design) 提到「事件驅動」，但實作使用的是輪詢 (polling)

     摘要
     ─────────────────────────────
     嚴重問題：0
     警告：2
     準備好封存：是 (帶有警告)

     建議：
     1. 為工作階段逾時情境新增測試
     2. 考慮按設計重構為事件驅動，或更新 design.md
```

**驗證 (verify) 的檢查項目：**

| 維度   | 驗證內容                                   |
| ------ | ------------------------------------------ |
| 完整性 | 所有任務已完成、所有需求已實作、情境已涵蓋 |
| 正確性 | 實作符合規格意圖、邊緣案例已處理           |
| 連貫性 | 技術決策反映在程式碼中、模式保持一致       |

驗證不會阻擋封存，但它會顯現您可能想要先處理的問題。

#### 封存：正式結束變更

`/opsx:archive` 完成變更並將其移至封存：

```text
您：/opsx:archive

AI：正在封存 add-auth...

     成品狀態：
     ✓ proposal.md 存在
     ✓ specs/ 存在
     ✓ design.md 存在
     ✓ tasks.md 存在 (12/12 個任務已完成)

     增量規格：尚未同步至主規格
     → 您現在要進行同步嗎？

您：是

AI：✓ 已將規格同步至 openspec/specs/auth/spec.md
     ✓ 已移動至 openspec/changes/archive/2025-01-24-add-auth/

     變更封存成功。
```

若規格尚未同步，封存命令會發出提示。它不會因為任務未完成而阻擋，但會給予警告。

## 何時該使用什麼

### `/opsx:ff` vs `/opsx:continue`

| 情況                     | 使用命令         |
| ------------------------ | ---------------- |
| 需求明確，準備好開始建構 | `/opsx:ff`       |
| 正在探索，想檢閱每個步驟 | `/opsx:continue` |
| 想在建立規格前先疊代提案 | `/opsx:continue` |
| 時間緊迫，需要快速行動   | `/opsx:ff`       |
| 變更複雜，想要掌控細節   | `/opsx:continue` |

**經驗法則：** 如果您能預先描述完整的範圍，請使用 `/opsx:ff`。如果您是邊做邊摸索，請使用 `/opsx:continue`。

### 何時更新 vs 重新開始

一個常見的問題：何時更新現有變更是可以接受的，以及何時該開始一個新變更？

**在以下情況更新現有變更：**

- 相同的意圖，精煉的執行方式
- 範圍縮小 (先推出 MVP，其餘隨後)
- 學習驅動的修正 (程式碼庫與您預期的不同)
- 根據實作發現進行設計微調

**在以下情況開始新變更：**

- 意圖發生根本性變化
- 範圍擴大到完全不同的工作
- 原始變更可以作為獨立項目標記為「完成」
- 修補 (patching) 會造成混淆而非釐清

```text
                     ┌─────────────────────────────────────┐
                     │     這是否為相同的工作？               │
                     └──────────────┬──────────────────────┘
                                    │
                 ┌──────────────────┼──────────────────┐
                 │                  │                  │
                 ▼                  ▼                  ▼
          意圖相同？         重疊 >50%？        原始變更是否可以在
          問題相同？         範圍相同？         沒有這些變更的情況下
                 │                  │           標記為「完成」？
                 │                  │                  │
       ┌────────┴────────┐  ┌──────┴──────┐   ┌───────┴───────┐
       │                 │  │             │   │               │
      是                否  是             否  否              是
       │                 │  │             │   │               │
       ▼                 ▼  ▼             ▼   ▼               ▼
      更新              新建 更新          新建 更新            新建
```

**範例：「新增深色模式」**

- 「還需要支援自訂主題」→ 新變更 (範圍擴大)
- 「系統偏好偵測比預期困難」→ 更新 (意圖相同)
- 「我們先發佈切換功能，稍後再加入偏好設定」→ 更新後封存，然後建立新變更

## 最佳實踐

### 保持變更焦點明確

每個變更應為一個邏輯工作單元。如果您正在「新增功能 X 且同時重構 Y」，請考慮將其拆分為兩個獨立的變更。

**為什麼這很重要：**
- 更易於檢閱和理解
- 更整潔的封存歷史記錄
- 可以獨立發佈
- 若有需要，回滾更簡單

### 對不明確的需求使用 `/opsx:explore`

在致力於一項變更之前，先探索問題領域：

```text
您：/opsx:explore

AI：您想探索什麼？

您：我們應該如何處理 API 的速率限制？

AI：讓我調查您目前的設定與選項...
     [研究程式碼庫，考慮各種模式]

     我看到三種我們可以採取的方法：
     1. 基於中間件 (簡單，針對每個端點)
     2. 以 Redis 為後盾 (具擴展性，共享狀態)
     3. 每位使用者一個令牌桶 (公平，複雜)

     您目前的技術棧建議使用第 1 或第 2 種。您的規模為何？
```

探索能在您建立成品之前協助釐清思緒。

### 封存前進行驗證

使用 `/opsx:verify` 檢查實作是否符合成品內容：

```text
您：/opsx:verify

AI：正在驗證 add-dark-mode...

     ✓ tasks.md 中的所有任務皆已勾選
     ✓ 規格中的需求皆有對應程式碼
     ✓ 技術決策已反映在實作中

     準備好封存！
```

在您關閉變更之前發現任何不符之處。

### 清楚地命名變更

良好的名稱能讓 `openspec list` 更有用：

```text
推薦：                          避免：
add-dark-mode                  feature-1
fix-login-redirect             update
optimize-product-query         changes
implement-2fa                  wip
```

## 命令快速參考

有關完整的命令詳細資訊和選項，請參閱 [命令 (Commands)](commands.md)。

| 命令                 | 用途                | 何時使用                     |
| -------------------- | ------------------- | ---------------------------- |
| `/opsx:propose`      | 建立變更 + 規劃成品 | 快速預設路徑 (`core` 設定檔) |
| `/opsx:explore`      | 思考想法            | 需求不明確、進行調查時       |
| `/opsx:new`          | 開始建立變更鷹架    | 擴展模式，需明確成品控制時   |
| `/opsx:continue`     | 建立下一個成品      | 擴展模式，逐步建立成品時     |
| `/opsx:ff`           | 建立所有規劃成品    | 擴展模式，範圍明確時         |
| `/opsx:apply`        | 實作任務            | 準備好撰寫程式碼時           |
| `/opsx:verify`       | 驗證實作內容        | 擴展模式，封存之前           |
| `/opsx:sync`         | 合併增量規格        | 擴展模式，選用               |
| `/opsx:archive`      | 完成變更            | 所有工作皆已完成時           |
| `/opsx:bulk-archive` | 封存多個變更        | 擴展模式，處理平行工作時     |

## 後續步驟

- [命令 (Commands)](commands.md) - 帶有選項的完整命令參考
- [概念 (Concepts)](concepts.md) - 深入探討規格、成品和結構描述
- [自訂 (Customization)](customization.md) - 建立自訂工作流
