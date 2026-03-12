# 受支援的工具 (Supported Tools)

OpenSpec 可與多種 AI 編碼助理搭配使用。當您執行 `openspec init` 時，OpenSpec 會根據您作用中的設定檔/工作流選擇以及遞送模式，來設定所選取的工具。

## 運作方式

對於每個選取的工具，OpenSpec 可以安裝：

1. **技能 (Skills)** (若遞送模式包含技能)：`.../skills/openspec-*/SKILL.md`
2. **命令 (Commands)** (若遞送模式包含命令)：特定工具的 `opsx-*` 命令檔案

根據預設，OpenSpec 使用 `core` 設定檔，其中包含：
- `propose`
- `explore`
- `apply`
- `archive`

您可以透過 `openspec config profile` 啟用擴展工作流 (`new`, `continue`, `ff`, `verify`, `sync`, `bulk-archive`, `onboard`)，然後執行 `openspec update`。

## 工具目錄參考

| 工具 (ID) | 技能路徑模式 | 命令路徑模式 |
|-----------|---------------------|----------------------|
| Amazon Q Developer (`amazon-q`) | `.amazonq/skills/openspec-*/SKILL.md` | `.amazonq/prompts/opsx-<id>.md` |
| Antigravity (`antigravity`) | `.agent/skills/openspec-*/SKILL.md` | `.agent/workflows/opsx-<id>.md` |
| Auggie (`auggie`) | `.augment/skills/openspec-*/SKILL.md` | `.augment/commands/opsx-<id>.md` |
| Claude Code (`claude`) | `.claude/skills/openspec-*/SKILL.md` | `.claude/commands/opsx/<id>.md` |
| Cline (`cline`) | `.cline/skills/openspec-*/SKILL.md` | `.clinerules/workflows/opsx-<id>.md` |
| CodeBuddy (`codebuddy`) | `.codebuddy/skills/openspec-*/SKILL.md` | `.codebuddy/commands/opsx/<id>.md` |
| Codex (`codex`) | `.codex/skills/openspec-*/SKILL.md` | `$CODEX_HOME/prompts/opsx-<id>.md`\* |
| Continue (`continue`) | `.continue/skills/openspec-*/SKILL.md` | `.continue/prompts/opsx-<id>.prompt` |
| CoStrict (`costrict`) | `.cospec/skills/openspec-*/SKILL.md` | `.cospec/openspec/commands/opsx-<id>.md` |
| Crush (`crush`) | `.crush/skills/openspec-*/SKILL.md` | `.crush/commands/opsx/<id>.md` |
| Cursor (`cursor`) | `.cursor/skills/openspec-*/SKILL.md` | `.cursor/commands/opsx-<id>.md` |
| Factory Droid (`factory`) | `.factory/skills/openspec-*/SKILL.md` | `.factory/commands/opsx-<id>.md` |
| Gemini CLI (`gemini`) | `.gemini/skills/openspec-*/SKILL.md` | `.gemini/commands/opsx/<id>.toml` |
| GitHub Copilot (`github-copilot`) | `.github/skills/openspec-*/SKILL.md` | `.github/prompts/opsx-<id>.prompt.md`\*\* |
| iFlow (`iflow`) | `.iflow/skills/openspec-*/SKILL.md` | `.iflow/commands/opsx-<id>.md` |
| Kilo Code (`kilocode`) | `.kilocode/skills/openspec-*/SKILL.md` | `.kilocode/workflows/opsx-<id>.md` |
| Kiro (`kiro`) | `.kiro/skills/openspec-*/SKILL.md` | `.kiro/prompts/opsx-<id>.prompt.md` |
| OpenCode (`opencode`) | `.opencode/skills/openspec-*/SKILL.md` | `.opencode/commands/opsx-<id>.md` |
| Pi (`pi`) | `.pi/skills/openspec-*/SKILL.md` | `.pi/prompts/opsx-<id>.md` |
| Qoder (`qoder`) | `.qoder/skills/openspec-*/SKILL.md` | `.qoder/commands/opsx/<id>.md` |
| Qwen Code (`qwen`) | `.qwen/skills/openspec-*/SKILL.md` | `.qwen/commands/opsx-<id>.toml` |
| RooCode (`roocode`) | `.roo/skills/openspec-*/SKILL.md` | `.roo/commands/opsx-<id>.md` |
| Trae (`trae`) | `.trae/skills/openspec-*/SKILL.md` | 不會生成 (無命令適配器；請使用以技能為基礎的 `/openspec-*` 呼叫) |
| Windsurf (`windsurf`) | `.windsurf/skills/openspec-*/SKILL.md` | `.windsurf/workflows/opsx-<id>.md` |

\* Codex 命令會安裝在全域的 Codex home 目錄中 (若有設定則為 `$CODEX_HOME/prompts/`，否則為 `~/.codex/prompts/`)，而非您的專案目錄。

\*\* GitHub Copilot 提示檔案在 IDE 擴充功能 (VS Code, JetBrains, Visual Studio) 中會被辨識為自訂的斜線命令。Copilot CLI 目前不會直接讀取 `.github/prompts/*.prompt.md`。

## 非互動式設定

對於 CI/CD 或指令碼式設定，請使用 `--tools` (以及選用的 `--profile`)：

```bash
# 設定特定工具
openspec init --tools claude,cursor

# 設定所有受支援的工具
openspec init --tools all

# 跳過工具設定
openspec init --tools none

# 針對此次執行覆寫設定檔
openspec init --profile core
```

**可用的工具 ID (`--tools`)：** `amazon-q`, `antigravity`, `auggie`, `claude`, `cline`, `codex`, `codebuddy`, `continue`, `costrict`, `crush`, `cursor`, `factory`, `gemini`, `github-copilot`, `iflow`, `kilocode`, `kiro`, `opencode`, `pi`, `qoder`, `qwen`, `roocode`, `trae`, `windsurf`

## 視工作流而定的安裝

OpenSpec 根據選取的工作流安裝對應的工作流成品：

- **核心設定檔 (預設)：** `propose`, `explore`, `apply`, `archive`
- **自訂選擇：** 所有工作流 ID 的任何子集：
  `propose`, `explore`, `new`, `continue`, `apply`, `ff`, `sync`, `archive`, `bulk-archive`, `verify`, `onboard`

換句話說，技能/命令的數量取決於設定檔和遞送模式，而非固定的。

## 生成的技能名稱

當透過設定檔/工作流設定選取時，OpenSpec 會生成以下技能：

- `openspec-propose`
- `openspec-explore`
- `openspec-new-change`
- `openspec-continue-change`
- `openspec-apply-change`
- `openspec-ff-change`
- `openspec-sync-specs`
- `openspec-archive-change`
- `openspec-bulk-archive-change`
- `openspec-verify-change`
- `openspec-onboard`

有關命令行為，請參閱 [命令 (Commands)](commands.md)；有關 `init`/`update` 選項，請參閱 [CLI](cli.md)。

## 相關內容

- [CLI 參考](cli.md) — 終端機命令
- [命令 (Commands)](commands.md) — 斜線命令與技能
- [入門指南 (Getting Started)](getting-started.md) — 首次設定
