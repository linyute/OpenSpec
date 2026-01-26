# @fission-ai/openspec

## 1.0.0

### 重大變更

- [#578](https://github.com/Fission-AI/OpenSpec/pull/578) [`0cc9d90`](https://github.com/Fission-AI/OpenSpec/commit/0cc9d9025af367faa1688a7b2606a2549053cd3f) 感謝 [@TabishB](https://github.com/TabishB)! - ## OpenSpec 1.0 — OPSX 版本發佈

  工作流程已從頭開始重建。OPSX 取代了舊有的階段鎖定 `/openspec:*` 指令，改用基於動作的系統，AI 能夠理解存在哪些成品、哪些已準備好建立，以及每個動作會解鎖什麼。

  ### 破壞性變更

  - **移除舊指令** — `/openspec:proposal`、`/openspec:apply` 和 `/openspec:archive` 不再存在
  - **移除設定檔案** — 不再產生工具特定的指示檔案（`CLAUDE.md`、`.cursorrules`、`AGENTS.md`、`project.md`）
  - **遷移** — 執行 `openspec init` 進行升級。將偵測舊版成品並在確認後清理。

  ### 從靜態提示到動態指示

  **之前：** AI 每次收到的都是相同的靜態指示，無論專案狀態如何。

  **現在：** 指示由三個層級動態組合而成：

  1. **背景資訊 (Context)** — 來自 `config.yaml` 的專案背景（技術堆疊、慣例）
  2. **規則 (Rules)** — 成品特定的約束（例如：「為未知事項提議試驗任務」）
  3. **範本 (Template)** — 輸出檔案的實際結構

  AI 會向 CLI 查詢即時狀態：存在哪些成品、哪些已準備好建立、哪些相依性已滿足，以及每個動作會解鎖什麼。

  ### 從階段鎖定到基於動作

  **之前：** 線性工作流程 — 提案 → 套用 → 封存。無法輕易回頭或疊代。

  **現在：** 針對變更的靈活動作。隨時編輯任何成品。成品圖會自動追蹤狀態。

  | 指令                 | 它的作用                       |
  | -------------------- | ------------------------------ |
  | `/opsx:explore`      | 在投入變更之前思考想法         |
  | `/opsx:new`          | 開始一個新的變更               |
  | `/opsx:continue`     | 一次建立一個成品（逐步執行）   |
  | `/opsx:ff`           | 一次建立所有規劃成品（快進）   |
  | `/opsx:apply`        | 實作任務                       |
  | `/opsx:verify`       | 驗證實作是否符合成品           |
  | `/opsx:sync`         | 將差異規格同步到主規格         |
  | `/opsx:archive`      | 封存已完成的變更               |
  | `/opsx:bulk-archive` | 封存多個變更並進行衝突偵測     |
  | `/opsx:onboard`      | 完整工作流程的 15 分鐘引導導覽 |

  ### 從文字合併到語義規格同步

  **之前：** 規格更新需要手動合併或大規模取代檔案。

  **現在：** 差異規格使用 AI 能理解的語義標記：

  - `## ADDED Requirements` — 要新增的新需求
  - `## MODIFIED Requirements` — 部分更新（新增情境而不必複製現有情境）
  - `## REMOVED Requirements` — 帶有理由和遷移註記的刪除
  - `## RENAMED Requirements` — 保留內容的重新命名

  封存功能在需求層級解析這些內容，而非脆弱的標題匹配。

  ### 從分散檔案到代理技能

  **之前：** 專案根目錄有 8 個以上的設定檔案 + 散佈在 21 個工具特定位置、格式各異的斜線指令。

  **現在：** 單一的 `.claude/skills/` 目錄，包含帶有 YAML 前言 (frontmatter) 的 Markdown 檔案。Claude Code、Cursor、Windsurf 會自動偵測。跨編輯器相容。

  ### 新功能

  - **新手引導技能** — `/opsx:onboard` 引導新使用者完成他們的第一個完整變更，提供具備程式碼庫意識的任務建議和逐步敘述（11 個階段，約 15 分鐘）

  - **支援 21 種 AI 工具** — Claude Code、Cursor、Windsurf、Continue、Gemini CLI、GitHub Copilot、Amazon Q、Cline、RooCode、Kilo Code、Auggie、CodeBuddy、Qoder、Qwen、CoStrict、Crush、Factory、OpenCode、Antigravity、iFlow 和 Codex

  - **互動式設定** — `openspec init` 顯示動畫歡迎畫面和用於選擇工具的可搜尋複選列表。預先選擇已設定的工具以便輕鬆重新整理。

  - **可自訂的 Schema** — 在 `openspec/schemas/` 中定義自訂成品工作流程，無需觸碰套件程式碼。團隊可以透過版本控制共享工作流程。

  ### 錯誤修復

  - 修復了當指令名稱包含冒號時 Claude Code 的 YAML 剖析失敗問題
  - 修復了任務檔案剖析，以處理複選框行上的結尾空格
  - 修復了 JSON 指示輸出，將背景資訊/規則與範本分開 — AI 之前會將約束區塊複製到成品檔案中

  ### 文件

  - 新的入門指南、CLI 參考、核心概念文件
  - 移除了誤導性的「在執行中編輯並繼續」之說法，該功能尚未實作
  - 新增了從 OPSX 之前版本升級的遷移指南


## 0.23.0

### 次要變更

- [#540](https://github.com/Fission-AI/OpenSpec/pull/540) [`c4cfdc7`](https://github.com/Fission-AI/OpenSpec/commit/c4cfdc7c499daef30d8a218f5f59b8d9e5adb754) 感謝 [@TabishB](https://github.com/TabishB)! - ### 新功能

  - **批次封存技能** — 使用 `/opsx:bulk-archive` 在單次操作中封存多個已完成的變更。包含批次驗證、規格衝突偵測以及合併後的確認。

  ### 其他

  - **簡化設定** — 建立設定時現在改用合理的預設值並輔以有用的註解，而非互動式提示。

## 0.22.0

### 次要變更

- [#530](https://github.com/Fission-AI/OpenSpec/pull/530) [`33466b1`](https://github.com/Fission-AI/OpenSpec/commit/33466b1e2a6798bdd6d0e19149173585b0612e6f) 感謝 [@TabishB](https://github.com/TabishB)! - 新增專案層級設定、專案本地 Schema 以及 Schema 管理指令

  **新功能**

  - **專案層級設定** — 透過 `openspec/config.yaml` 針對每個專案設定 OpenSpec 行為，包含自訂規則注入、背景資訊檔案以及 Schema 解析設定。
  - **專案本地 Schema** — 在您專案的 `openspec/schemas/` 目錄中定義自訂成品 Schema，用於專案特定的工作流程。
  - **Schema 管理指令** — 新的 `openspec schema` 指令（`list`、`show`、`export`、`validate`），用於檢查和管理成品 Schema（實驗性）。

  **錯誤修復**

  - 修復了設定載入以處理專案設定中為空的 `rules` 欄位。

## 0.21.0

### 次要變更

- [#516](https://github.com/Fission-AI/OpenSpec/pull/516) [`b5a8847`](https://github.com/Fission-AI/OpenSpec/commit/b5a884748be6156a7bb140b4941cfec4f20a9fc8) 感謝 [@TabishB](https://github.com/TabishB)! - 新增回饋指令和 Nix flake 支援

  **新功能**

  - **回饋指令** — 直接從 CLI 使用 `openspec feedback` 提交回饋，這會建立 GitHub Issue 並自動包含 Metadata，且針對手動提交提供優雅的回退方案。
  - **Nix flake 支援** — 使用新的 `flake.nix` 安裝和開發 OpenSpec，包含自動化的 flake 維護和 CI 驗證。

  **錯誤修復**

  - **探索模式護欄** — 探索模式現在明確禁止實作，將焦點保持在思考和發現上，同時仍然允許建立成品。

  **其他**

  - 改進了 `opsx apply` 中的變更推斷 — 當不明確時，自動從對話背景資訊中偵測目標變更或進行提示。
  - 精簡了封存同步評估，提供更清晰的差異規格位置指引。

## 0.20.0

### 次要變更

- [#502](https://github.com/Fission-AI/OpenSpec/pull/502) [`9db74aa`](https://github.com/Fission-AI/OpenSpec/commit/9db74aa5ac6547efadaed795217cfa17444f2004) 感謝 [@TabishB](https://github.com/TabishB)! - 新增 `/opsx:verify` 指令並修復 vitest 處理程序風暴

  **新功能**

  - **`/opsx:verify` 指令** — 驗證變更實作是否符合其規格。

  **錯誤修復**

  - 透過限制工作平行度修復了 vitest 處理程序風暴。
  - 修復了代理工作流程，使其針對驗證指令使用非互動模式。
  - 修復了 PowerShell 補全產生器以移除結尾的逗號。

## 0.19.0

### 次要變更

- eb152eb: 新增 Continue IDE 支援、shell 補全以及 `/opsx:explore` 指令

  **新功能**

  - **Continue IDE 支援** – OpenSpec 現在可為 [Continue](https://continue.dev/) 產生斜線指令，擴展了除 Cursor、Windsurf、Claude Code 等工具之外的編輯器整合選項。
  - **Bash、Fish 和 PowerShell 的 shell 補全** – 執行 `openspec completion install` 以在您偏好的 shell 中設定 Tab 補全。
  - **`/opsx:explore` 指令** – 一種新的思考夥伴模式，用於在投入變更之前探索想法和調查問題。
  - **Codebuddy 斜線指令改進** – 更新了 frontmatter 格式以獲得更好的相容性。

  **錯誤修復**

  - 當指令具有子指令時，shell 補全現在可以正確提供父層級旗標（如 `--help`）。
  - 修復了測試中的 Windows 相容性問題。

  **其他**

  - 新增了選用的匿名使用統計資訊，以協助瞭解 OpenSpec 的使用情況。這在預設情況下是 **選擇退出 (opt-out)** 的 – 設定 `OPENSPEC_TELEMETRY=0` 或 `DO_NOT_TRACK=1` 即可停用。僅收集指令名稱和版本；不收集參數、檔案路徑或內容。在 CI 環境中會自動停用。

## 0.18.0

### 次要變更

- 8dfd824: 新增 OPSX 實驗性工作流程指令和增強的成品系統

  **新指令：**

  - `/opsx:ff` - 快進完成成品建立，一次產生所有需要的成品。
  - `/opsx:sync` - 將變更中的差異規格同步到主規格。
  - `/opsx:archive` - 封存已完成的變更，並進行智慧同步檢查。

  **成品工作流程增強：**

  - 具備 Schema 意識的套用指示，提供內嵌指引和 XML 輸出。
  - 實驗性成品工作流程的代理 Schema 選擇。
  - 透過 `.openspec.yaml` 檔案設定每個變更的 Schema Metadata。
  - 實驗性成品工作流程的代理技能 (Agent Skills)。
  - 用於範本載入和變更背景資訊的指示載入器。
  - 將 Schema 重構為包含範本的目錄。

  **改進：**

  - 增強了 list 指令，包含最後修改時間戳記和排序功能。
  - 用於更好支援工作流程的變更建立公用程式。

  **修復：**

  - 正規化路徑以實現跨平台 glob 相容性。
  - 建立新規格檔案時允許 REMOVED 需求。

## 0.17.2

### 修補程式變更

- 455c65f: 修復了 validate 指令中的 `--no-interactive` 旗標，以正確停用載入動畫 (spinner)，防止在 pre-commit 勾點和 CI 環境中掛起。

## 0.17.1

### 修補程式變更

- a2757e7: 透過對 @inquirer/prompts 使用動態匯入，修復了 config 指令中的 pre-commit 勾點掛起問題。

  由於在模組載入時註冊了 stdin 事件接聽器，config 指令曾導致 pre-commit 勾點無限期掛起。此修復將靜態匯入轉換為動態匯入，僅在實際互動式使用 `config reset` 指令時才載入 inquirer。

  同時新增了包含規則的 ESLint，以防止靜態匯入 @inquirer，避免未來發生退化 (regressions)。

## 0.17.0

### 次要變更

- 2e71835: 新增 `openspec config` 指令與 Oh-my-zsh 補全

  **新功能**

  - 新增 `openspec config` 指令，用於管理全域設定。
  - 實作了支援 XDG 基本目錄規格 (XDG Base Directory specification) 的全域設定目錄。
  - 新增 Oh-my-zsh shell 補全支援，提升 CLI 使用體驗。

  **錯誤修復**

  - 透過使用動態匯入修復了 pre-commit 勾點掛起的問題。
  - 在所有平台上皆尊重 XDG_CONFIG_HOME 環境變數。
  - 解決了 zsh-installer 測試中的 Windows 相容性問題。
  - 使 cli-completion 規格與實作保持一致。
  - 移除了斜線指令中寫死的 agent 欄位。

  **文件**

  - 將 README 中的 AI 工具清單按字母順序排列，並使其可摺疊。

## 0.16.0

### 次要變更

- c08fbc1: 新增 AI 工具整合與增強：

  - **feat(iflow-cli)**: 新增 iFlow-cli 整合，支援斜線指令與文件。
  - **feat(init)**: 在初始化後新增 IDE 重啟說明，告知使用者斜線指令已可用。
  - **feat(antigravity)**: 新增 Antigravity 斜線指令支援。
  - **fix**: 為 Qwen Code 產生 TOML 指令（修復 #293）。
  - 釐清了腳手架提案文件並增強了提案指南。
  - 更新了提案指南，強調在實作前採用設計優先 (design-first) 的方法。

## 未發佈

### 次要變更

- 新增 Continue 斜線指令支援，使 `openspec init` 能產生帶有 Markdown frontmatter 和 `$ARGUMENTS` 佔位符的 `.continue/prompts/openspec-*.prompt` 檔案，並在 `openspec update` 時重新整理。

- 新增 Antigravity 斜線指令支援，使 `openspec init` 能產生僅包含說明的 frontmatter 的 `.agent/workflows/openspec-*.md` 檔案，且 `openspec update` 會與 Windsurf 一起重新整理現有的工作流程。

## 0.15.0

### 次要變更

- 4758c5c: 新增對具有原生斜線指令整合的新 AI 工具之支援

  - **Gemini CLI**: 為 Gemini CLI 新增原生基於 TOML 的斜線指令支援，整合了 `.gemini/commands/openspec/`。
  - **RooCode**: 新增 RooCode 整合，包含設定器、斜線指令和範本。
  - **Cline**: 修復 Cline 以針對斜線指令使用工作流程而非規則（`.clinerules/workflows/` 路徑）。
  - **文件**: 更新文件以反映新的整合與工作流程變更。

## 0.14.0

### 次要變更

- 8386b91: 新增對新 AI 助理的支援與設定改進

  - feat: 新增 Qwen Code 支援與斜線指令整合。
  - feat: 為套用 (apply) 斜線指令新增 $ARGUMENTS 支援，用於動態變數傳遞。
  - feat: 為設定與文件新增 Qoder CLI 支援。
  - feat: 新增 CoStrict AI 助理支援。
  - fix: 在擴展模式下重新建立缺失的 OpenSpec 範本檔案。
  - fix: 防止對工具進行錯誤的「已設定」偵測。
  - fix: 使用 change-id 作為回退標題，而非 "Untitled Change"。
  - docs: 新增填寫專案層級背景資訊的指引。
  - docs: 在 README 的受支援 AI 工具中新增 Crush。

## 0.13.0

### 次要變更

- 668a125: 新增對多個 AI 助理的支援並改進驗證

  此版本新增了對數個新 AI 程式碼助理的支援：

  - CodeBuddy Code - AI 驅動的程式碼助理。
  - CodeRabbit - AI 程式碼檢閱助理。
  - Cline - 基於 Claude 的 CLI 助理。
  - Crush AI - AI 助理平台。
  - Auggie (Augment CLI) - 程式碼增強工具。

  新功能：

  - 封存 (Archive) 斜線指令現在支援參數，以實現更靈活的工作流程。

  錯誤修復：

  - 差異規格驗證現在可處理不區分大小寫的標題，並正確偵測空章節。
  - 封存驗證現在可正確遵守 --no-validate 旗標並忽略 Metadata。

  文件改進：

  - 新增了 VS Code dev container 設定，以便於開發環境設定。
  - 更新了 AGENTS.md，包含明確的 change-id 標記。
  - 增強了斜線指令文件，新增了重啟說明。

## 0.12.0

### 次要變更

- 082abb4: 新增斜線指令的工廠函式支援以及非互動式初始化選項

  此版本包含兩個新功能：

  - **斜線指令的工廠函式支援**: 現在可以將斜線指令定義為傳回指令物件的函式，實現動態指令設定。
  - **非互動式初始化選項**: 在 `openspec init` 中新增了 `--tools`、`--all-tools` 和 `--skip-tools` CLI 旗標，以便在 CI/CD 流水線中自動進行初始化，同時保持與互動模式的向下相容性。

## 0.11.0

### 次要變更

- 312e1d6: 新增 Amazon Q Developer CLI 整合。OpenSpec 現在支援 Amazon Q Developer，並在 `.amazonq/prompts/` 目錄中自動產生提示，允許您搭配 Amazon Q 的 @ 語法來使用 OpenSpec 斜線指令。

## 0.10.0

### 次要變更

- d7e0ce8: 改進了初始化精靈的 Enter 鍵行為，讓使用者能更自然地完成提示流程。

## 0.9.2

### 修補程式變更

- 2ae0484: 修復了跨平台路徑處理問題。此版本包含對 joinPath 行為和斜線指令路徑解析的修復，以確保 OpenSpec 在所有平台上都能正常運作。

## 0.9.1

### 修補程式變更

- 8210970: 修復了在選擇 Codex 整合時 OpenSpec 無法在 Windows 上運作的問題。此版本包含對跨平台路徑處理和正規化的修復，以確保 OpenSpec 在 Windows 系統上能正確運作。

## 0.9.0

### 次要變更

- efbbf3b: 新增對 Codex 和 GitHub Copilot 斜線指令的支援，包含 YAML frontmatter 和 $ARGUMENTS。

## 未發佈

### 次要變更

- 新增 GitHub Copilot 斜線指令支援。OpenSpec 現在會將提示寫入至 `.github/prompts/openspec-{proposal,apply,archive}.prompt.md`，包含 YAML frontmatter 和 `$ARGUMENTS` 佔位符，並在 `openspec update` 時進行重新整理。

## 0.8.1

### 修補程式變更

- d070d08: 修復了 CLI 版本不匹配的問題，並新增了發佈防護，透過 `openspec --version` 驗證封裝後的 tarball 印出的版本與 package.json 相同。

## 0.8.0

### 次要變更

- c29b06d: 新增 Windsurf 支援。
- 新增 Codex 斜線指令支援。OpenSpec 現在會直接將提示寫入至 Codex 的全域目錄（`~/.codex/prompts` 或 `$CODEX_HOME/prompts`），並在 `openspec update` 時進行重新整理。

## 0.7.0

### 次要變更

- 新增原生 Kilo Code 工作流程整合，使 `openspec init` 和 `openspec update` 能管理 `.kilocode/workflows/openspec-*.md` 檔案。
- 在初始化/更新期間，一律建立受管理的根目錄 `AGENTS.md` 交付存根 (hand-off stub)，並重新組合 AI 工具提示，以保持指示的一致性。

## 0.6.0

### 次要變更

- 將產生的根目錄代理指示精簡為受管理的交付存根 (hand-off stub)，並更新初始化/更新流程以安全地重新整理。

## 0.5.0

### 次要變更

- feat: 實作第一階段端對端 (E2E) 測試，具備跨平台 CI 矩陣。

  - 在 `test/helpers/run-cli.ts` 中新增了共用的 runCLI 輔助程式，用於 spawn 測試。
  - 建立了 `test/cli-e2e/basic.test.ts`，涵蓋 help、version、validate 流程。
  - 將現有的 CLI 執行測試遷移至使用 runCLI 輔助程式。
  - 將 CI 矩陣擴展至 bash (Linux/macOS) 和 pwsh (Windows)。
  - 拆分 PR 和主工作流程，以優化回饋。

### 修補程式變更

- 使套用 (apply) 指示更具體。

  改進代理範本和斜線指令範本，提供更具體且具可操作性的套用指示。

- docs: 改進文件與清理。

  - 在文件中記錄了封存 (archive) 指令的非互動旗標。
  - 取代了 README 中的 Discord 徽章。
  - 封存已完成的變更，以實現更好的組織。

## 0.4.0

### 次要變更

- 新增 OpenSpec 變更提案，用於 CLI 改進與增強使用者體驗。
- 新增 Opencode 斜線指令支援，用於 AI 驅動的開發工作流程。

### 修補程式變更

- 新增文件改進，包含針對封存指令範本的 --yes 旗標以及 Discord 徽章。
- 修復了 Markdown 剖析器中的換行符號正規化，以正確處理 CRLF 檔案。

## 0.3.0

### 次要變更

- 增強了 `openspec init`，包含擴展模式、多工具選擇以及互動式 `AGENTS.md` 設定器。

## 0.2.0

### 次要變更

- ce5cead: - 新增 `openspec view` 儀表板，可一目瞭然地彙總規格計數和變更進度。
  - 在重新命名的 `openspec/AGENTS.md` 指示檔案旁產生並更新 AI 斜線指令。
  - 移除了已棄用的 `openspec diff` 指令，並引導使用者使用 `openspec show`。

## 0.1.0

### 次要變更

- 24b4866: 最初版本發佈
