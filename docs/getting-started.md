# 入門指南 (Getting Started)

本指南說明在您安裝並初始化 OpenSpec 後，它是如何運作的。有關安裝指令，請參閱 [主 README 檔案](../README.md#快速入門)。

## 運作方式

OpenSpec 協助您與您的 AI 編碼助理在撰寫任何程式碼之前，先就建構內容達成共識。

**預設快速路徑 (核心設定檔)：**

```text
/opsx:propose ──► /opsx:apply ──► /opsx:archive
```

**擴展路徑 (自訂工作流選擇)：**

```text
/opsx:new ──► /opsx:ff 或 /opsx:continue ──► /opsx:apply ──► /opsx:verify ──► /opsx:archive
```

預設的全域設定檔為 `core`，包含 `propose`, `explore`, `apply` 以及 `archive`。您可以使用 `openspec config profile` 啟用擴展工作流命令，然後在專案中執行 `openspec update`。

## OpenSpec 建立的內容

執行 `openspec init` 後，您的專案將具備以下結構：

```
openspec/
├── specs/              # 真相來源 (您系統的行為)
│   └── <domain>/
│       └── spec.md
├── changes/            # 提案更新 (每個變更一個資料夾)
│   └── <變更名稱>/
│       ├── proposal.md
│       ├── design.md
│       ├── tasks.md
│       └── specs/      # 增量規格 (正在變更的內容)
│           └── <domain>/
│               └── spec.md
└── config.yaml         # 專案設定 (選用)
```

**兩個關鍵目錄：**

- **`specs/`** - 真相來源。這些規格描述了您的系統目前的行為。按網域 (domain) 組織 (例如：`specs/auth/`, `specs/payments/`)。

- **`changes/`** - 提案修改。每個變更都有其專屬資料夾，包含所有相關成品。當變更完成時，其規格會合併到主 `specs/` 目錄中。

## 理解成品 (Artifacts)

每個變更資料夾都包含指引工作的成品：

| 成品          | 用途                                                           |
| ------------- | -------------------------------------------------------------- |
| `proposal.md` | 「為什麼」以及「做什麼」 - 捕捉意圖、範圍和方法 (approach)     |
| `specs/`      | 顯示新增 (ADDED)/修改 (MODIFIED)/移除 (REMOVED) 需求的增量規格 |
| `design.md`   | 「如何做」 - 技術方案 (technical approach) 和架構決策          |
| `tasks.md`    | 帶有核取方塊的實作檢查清單                                     |

**成品相互建構：**

```
提案 (proposal) ──► 規格 (specs) ──► 設計 (design) ──► 任務 (tasks) ──► 實作 (implement)
      ▲               ▲              ▲                               │
      └───────────────┴──────────────┴───────────────────────────────┘
                             隨做隨學，隨時更新
```

在實作過程中學到更多資訊時，您隨時可以回頭精煉先前的成品。

## 增量規格 (Delta Specs) 如何運作

增量規格是 OpenSpec 中的核心概念。它們顯示了相對於目前規格正在變更的內容。

### 格式

增量規格使用不同區段來表示變更類型：

```markdown
# 驗證增量 (Delta for Auth)

## 新增需求 (ADDED Requirements)

### 需求：雙重驗證 (Two-Factor Authentication)
系統必須 (MUST) 在登入期間要求第二個驗證因素。

#### 情境：需要 OTP
- 假設 (GIVEN) 使用者已啟用 2FA
- 當 (WHEN) 使用者提交有效的憑證
- 那麼 (THEN) 出現 OTP 挑戰

## 修改需求 (MODIFIED Requirements)

### 需求：工作階段逾時 (Session Timeout)
系統應 (SHALL) 在閒置 30 分鐘後讓工作階段過期。
(先前為：60 分鐘)

#### 情境：閒置逾時
- 假設 (GIVEN) 一個已通過驗證的工作階段
- 當 (WHEN) 經過 30 分鐘沒有活動
- 那麼 (THEN) 工作階段失效

## 移除需求 (REMOVED Requirements)

### 需求：記住我 (Remember Me)
(已棄用，改用 2FA)
```

### 封存時會發生什麼

當您封存變更時：

1. **新增 (ADDED)** 的需求會附加到主規格中
2. **修改 (MODIFIED)** 的需求會取代現有的版本
3. **移除 (REMOVED)** 的需求會從主規格中刪除

變更資料夾會移動到 `openspec/changes/archive/` 以供稽核歷程記錄使用。

## 範例：您的第一個變更

讓我們逐步完成為應用程式新增深色模式的過程。

### 1. 開始變更 (預設方式)

```text
您：/opsx:propose add-dark-mode

AI：已建立 openspec/changes/add-dark-mode/
     ✓ proposal.md — 為什麼我們要這樣做，哪些部分正在變更
     ✓ specs/       — 需求和情境
     ✓ design.md    — 技術方案 (technical approach)
     ✓ tasks.md     — 實作檢查清單
     準備好進行實作！
```

如果您啟用了擴展工作流設定檔，您也可以分兩步執行此操作：先執行 `/opsx:new` 然後執行 `/opsx:ff` (或使用 `/opsx:continue` 逐步執行)。

### 2. 建立了什麼內容

**proposal.md** - 捕捉意圖：

```markdown
# 提案：新增深色模式

## 意圖
使用者要求新增深色模式選項，以減輕夜間使用的眼部疲勞。

## 範圍
- 在設定中新增主題切換功能
- 支援系統偏好偵測
- 將偏好持久化於 localStorage 中

## 方法
使用 CSS 自訂屬性進行佈景，並搭配 React context 進行狀態管理。
```

**specs/ui/spec.md** - 顯示新需求的增量規格：

```markdown
# UI 增量

## 新增需求 (ADDED Requirements)

### 需求：主題選擇
系統應 (SHALL) 允許使用者在亮色和深色主題之間切換。

#### 情境：手動切換
- 假設 (GIVEN) 使用者在任何頁面上
- 當 (WHEN) 使用者點擊主題切換按鈕
- 那麼 (THEN) 主題立即切換
- 並且 (AND) 該偏好在不同的工作階段中皆能維持

#### 情境：系統偏好
- 假設 (GIVEN) 使用者沒有儲存的偏好
- 當 (WHEN) 應用程式載入時
- 那麼 (THEN) 使用系統偏好的配色方案
```

**tasks.md** - 實作檢查清單：

```markdown
# 任務

## 1. 主題基礎設施
- [ ] 1.1 建立具有亮色/深色狀態的 ThemeContext
- [ ] 1.2 新增顏色的 CSS 自訂屬性
- [ ] 1.3 實作 localStorage 持久化

## 2. UI 元件
- [ ] 2.1 建立 ThemeToggle 元件
- [ ] 2.2 在設定頁面新增切換功能
- [ ] 2.3 更新 Header 以包含快速切換功能

## 3. 樣式
- [ ] 3.1 定義深色主題調色盤
- [ ] 3.2 更新元件以使用 CSS 變數
```

### 3. 實作 (Implement)

```
您：/opsx:apply

AI：正在瀏覽任務...
     ✓ 1.1 已建立具有亮色/深色狀態的 ThemeContext
     ✓ 1.2 已在 globals.css 中新增 CSS 自訂屬性
     ✓ 1.3 已實作 localStorage 持久化
     ✓ 2.1 已建立 ThemeToggle 元件
     ...
     所有任務已完成！
```

在實作過程中，如果您發現技術方案 (design) 需要調整，只需更新該成品並繼續即可。

### 4. 封存 (Archive)

```
您：/opsx:archive

AI：正在封存 add-dark-mode...
     ✓ 已將規格合併至 openspec/specs/ui/spec.md
     ✓ 已移動至 openspec/changes/archive/2025-01-24-add-dark-mode/
     完成！準備好進行下一個功能。
```

您的增量規格現在已成為主規格的一部分，記錄了系統的運作方式。

## 驗證與檢閱

使用 CLI 檢查您的變更：

```bash
# 列出作用中的變更
openspec list

# 檢視變更詳細資訊
openspec show add-dark-mode

# 驗證規格格式
openspec validate add-dark-mode

# 互動式儀表板
openspec view
```

## 後續步驟

- [工作流 (Workflows)](workflows.md) - 常見模式及何時使用各個命令
- [命令 (Commands)](commands.md) - 所有斜線命令的完整參考
- [概念 (Concepts)](concepts.md) - 對規格、變更和結構描述的深入理解
- [自訂 (Customization)](customization.md) - 讓 OpenSpec 以您想要的方式運作
