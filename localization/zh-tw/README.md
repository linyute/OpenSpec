<p align="center">
  <a href="https://github.com/Fission-AI/OpenSpec">
    <picture>
      <source srcset="assets/openspec_bg.png">
      <img src="assets/openspec_bg.png" alt="OpenSpec logo">
    </picture>
  </a>
</p>

<p align="center">
  <a href="https://github.com/Fission-AI/OpenSpec/actions/workflows/ci.yml"><img alt="CI" src="https://github.com/Fission-AI/OpenSpec/actions/workflows/ci.yml/badge.svg" /></a>
  <a href="https://www.npmjs.com/package/@fission-ai/openspec"><img alt="npm version" src="https://img.shields.io/npm/v/@fission-ai/openspec?style=flat-square" /></a>
  <a href="./LICENSE"><img alt="License: MIT" src="https://img.shields.io/badge/License-MIT-blue.svg?style=flat-square" /></a>
  <a href="https://discord.gg/YctCnvvshC"><img alt="Discord" src="https://img.shields.io/discord/1411657095639601154?style=flat-square&logo=discord&logoColor=white&label=Discord&suffix=%20online" /></a>
</p>

<details>
<summary><strong>最受喜愛的規格框架。</strong></summary>

[![Stars](https://img.shields.io/github/stars/Fission-AI/OpenSpec?style=flat-square&label=Stars)](https://github.com/Fission-AI/OpenSpec/stargazers)
[![Downloads](https://img.shields.io/npm/dm/@fission-ai/openspec?style=flat-square&label=Downloads/mo)](https://www.npmjs.com/package/@fission-ai/openspec)
[![Contributors](https://img.shields.io/github/contributors/Fission-AI/OpenSpec?style=flat-square&label=Contributors)](https://github.com/Fission-AI/OpenSpec/graphs/contributors)

</details>
<p></p>
我們的理念：

```text
→ 流動而非僵化
→ 疊代而非瀑布
→ 簡單而非複雜
→ 專為既有專案 (brownfield) 打造，而不僅僅是新專案 (greenfield)
→ 從個人專案到企業規模皆可擴展
```

> [!TIP]
> **新工作流程現已推出！** 我們使用新的成品引導式工作流程重建了 OpenSpec。
>
> 執行 `/opsx:onboard` 以開始使用。 → [在此瞭解更多資訊](docs/opsx.md)

<p align="center">
  在 X 上關注 <a href="https://x.com/0xTab">@0xTab</a> 以獲取更新 · 加入 <a href="https://discord.gg/YctCnvvshC">OpenSpec Discord</a> 進行諮詢與提問。
</p>

### 團隊

在團隊中使用 OpenSpec？[發送電子郵件至此](mailto:teams@openspec.dev) 以獲取我們 Slack 頻道的存取權限。

<!-- TODO: Add GIF demo of /opsx:new → /opsx:archive workflow -->

## 看看它的運作方式

```text
您：/opsx:new add-dark-mode
AI： 已建立 openspec/changes/add-dark-mode/
     準備好建立：proposal

您：/opsx:ff   # "快進 (fast-forward)" - 產生所有規劃文件
AI： ✓ proposal.md — 我們為什麼要這樣做，正在變更什麼
     ✓ specs/       — 需求與情境
     ✓ design.md    — 技術方法
     ✓ tasks.md     — 實作檢查清單
     準備好進行實作！

您：/opsx:apply
AI： 正在實作任務...
     ✓ 1.1 新增佈景主題背景提供者 (theme context provider)
     ✓ 1.2 建立切換元件 (toggle component)
     ✓ 2.1 新增 CSS 變數
     ✓ 2.2 串接 localStorage
     所有任務已完成！

您：/opsx:archive
AI： 已封存至 openspec/changes/archive/2025-01-23-add-dark-mode/
     規格已更新。準備好進行下一個功能。
```

<details>
<summary><strong>OpenSpec 儀表板</strong></summary>

<p align="center">
  <img src="assets/openspec_dashboard.png" alt="OpenSpec dashboard preview" width="90%">
</p>

</details>

## 快速開始

**需要 Node.js 20.19.0 或更高版本。**

全域安裝 OpenSpec：

```bash
npm install -g @fission-ai/openspec@latest
```

接著導覽至您的專案目錄並初始化：

```bash
cd your-project
openspec init
```

現在告訴您的 AI：`/opsx:new <您想要建構的內容>`

> [!NOTE]
> 不確定您的工具是否受支援？[檢視完整清單](docs/supported-tools.md) – 我們支援 20 多種工具且持續增加中。
>
> 同時支援 pnpm、yarn、bun 和 nix。[參閱安裝選項](docs/installation.md)。

## 文件

→ **[入門指南](docs/getting-started.md)**: 第一步<br>
→ **[工作流程](docs/workflows.md)**: 組合與模式<br>
→ **[指令](docs/commands.md)**: 斜線指令與技能<br>
→ **[CLI](docs/cli.md)**: 終端機參考<br>
→ **[受支援的工具](docs/supported-tools.md)**: 工具整合與安裝路徑<br>
→ **[核心概念](docs/concepts.md)**: 各部分如何整合<br>
→ **[多語言](docs/multi-language.md)**: 多語言支援<br>
→ **[自訂](docs/customization.md)**: 打造專屬您的 OpenSpec


## 為什麼選擇 OpenSpec？

AI 程式碼助理雖然強大，但當需求僅存在於對話歷史中時，其表現是不可預測的。OpenSpec 增加了一個輕量級的規格層，讓您在撰寫任何程式碼之前，先就「要建構什麼」達成共識。

- **先共識再開發** — 真人與 AI 在撰寫程式碼前就規格達成一致。
- **保持井然有序** — 每次變更都有自己的資料夾，包含提案、規格、設計和任務。
- **流暢地工作** — 隨時更新任何成品，沒有僵化的階段門檻。
- **使用您現有的工具** — 透過斜線指令與 20 多種 AI 助理搭配使用。

### 比較

**vs. [Spec Kit](https://github.com/github/spec-kit)** (GitHub) — 詳盡但沉重。僵化的階段門檻、大量的 Markdown、需要 Python 設定。OpenSpec 較輕量且讓您能自由疊代。

**vs. [Kiro](https://kiro.dev)** (AWS) — 強大但您會被鎖定在其 IDE 中且僅限於 Claude 模型。OpenSpec 支援您已在使用的工具。

**vs. 什麼都不用** — 沒有規格的 AI 編碼意味著模糊的提示和不可預測的結果。OpenSpec 在不增加繁瑣流程的前提下帶來了可預測性。

## 更新 OpenSpec

**升級套件**

```bash
npm install -g @fission-ai/openspec@latest
```

**重新整理代理指示**

在每個專案內執行此指令，以重新產生 AI 指引並確保最新的斜線指令已生效：

```bash
openspec update
```

## 使用說明

**模型選擇**：OpenSpec 在具備高推理能力的模型上表現最佳。我們建議在規劃和實作階段都使用 Opus 4.5 和 GPT 5.2。

**背景資訊衛生 (Context hygiene)**：OpenSpec 受益於乾淨的背景資訊視窗。在開始實作前，請清除您的對話背景資訊，並在整個工作階段期間保持良好的背景資訊衛生。

## 貢獻

**小修補** — 錯誤修復、錯字更正和微小改進可以直接以 PR (Pull Request) 形式提交。

**較大的變更** — 對於新功能、重大的重構或架構變更，請先提交 OpenSpec 變更提案 (change proposal)，以便我們在實作開始前就意圖和目標達成一致。

撰寫提案時，請謹記 OpenSpec 的理念：我們服務於跨不同編碼代理、模型和使用場景的廣大使用者。變更應該對每個人都運作良好。

**歡迎 AI 產生的程式碼** — 只要它經過測試和驗證。包含 AI 產生程式碼的 PR 應註明所使用的編碼代理和模型（例如：「使用 Claude Code 和 claude-opus-4-5-20251101 產生」）。

### 開發

- 安裝相依項：`pnpm install`
- 建構：`pnpm run build`
- 測試：`pnpm test`
- 本地開發 CLI：`pnpm run dev` 或 `pnpm run dev:cli`
- 慣例提交 (Conventional commits)（單行）：`type(scope): subject`

## 其他

<details>
<summary><strong>遙測 (Telemetry)</strong></summary>

OpenSpec 收集匿名的使用統計資料。

我們僅收集指令名稱和版本，以瞭解使用模式。不收集參數、路徑、內容或個人識別資訊 (PII)。在 CI 環境中會自動停用。

**選擇退出 (Opt-out)：** `export OPENSPEC_TELEMETRY=0` 或 `export DO_NOT_TRACK=1`

</details>

<details>
<summary><strong>維護者與顧問</strong></summary>

參閱 [MAINTAINERS.md](MAINTAINERS.md) 以獲取核心維護者和顧問清單。

</details>

## 授權

MIT
