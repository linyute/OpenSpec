# 支援的工具

OpenSpec 可與 20 多種 AI 程式碼助理搭配使用。當您執行 `openspec init` 時，系統會提示您選擇所使用的工具，OpenSpec 將會設定相應的整合。

## 它如何運作

針對您選擇的每個工具，OpenSpec 會安裝：

1. **技能 (Skills)** — 可重複使用的指示檔案，驅動 `/opsx:*` 工作流程指令。
2. **指令 (Commands)** — 工具特定的斜線指令繫結。

## 工具目錄參考

| 工具                 | 技能位置             | 指令位置                            |
| -------------------- | -------------------- | ----------------------------------- |
| Amazon Q Developer   | `.amazonq/skills/`   | `.amazonq/prompts/`                 |
| Antigravity          | `.agent/skills/`     | `.agent/workflows/`                 |
| Auggie (Augment CLI) | `.augment/skills/`   | `.augment/commands/`                |
| Claude Code          | `.claude/skills/`    | `.claude/commands/opsx/`            |
| Cline                | `.cline/skills/`     | `.clinerules/workflows/`            |
| CodeBuddy            | `.codebuddy/skills/` | `.codebuddy/commands/opsx/`         |
| Codex                | `.codex/skills/`     | `~/.codex/prompts/`*                |
| Continue             | `.continue/skills/`  | `.continue/prompts/`                |
| CoStrict             | `.cospec/skills/`    | `.cospec/openspec/commands/`        |
| Crush                | `.crush/skills/`     | `.crush/commands/opsx/`             |
| Cursor               | `.cursor/skills/`    | `.cursor/commands/`                 |
| Factory Droid        | `.factory/skills/`   | `.factory/commands/`                |
| Gemini CLI           | `.gemini/skills/`    | `.gemini/commands/opsx/`            |
| GitHub Copilot       | `.github/skills/`    | `.github/prompts/`                  |
| iFlow                | `.iflow/skills/`     | `.iflow/commands/`                  |
| Kilo Code            | `.kilocode/skills/`  | `.kilocode/workflows/`              |
| OpenCode             | `.opencode/skills/`  | `.opencode/command/`                |
| Qoder                | `.qoder/skills/`     | `.qoder/commands/opsx/`             |
| Qwen Code            | `.qwen/skills/`      | `.qwen/commands/`                   |
| RooCode              | `.roo/skills/`       | `.roo/commands/`                    |
| Trae                 | `.trae/skills/`      | `.trae/skills/` (via `/openspec-*`) |
| Windsurf             | `.windsurf/skills/`  | `.windsurf/workflows/`              |

\* Codex 指令會安裝在全域家目錄 (`~/.codex/prompts/` 或 `$CODEX_HOME/prompts/`)，而不是專案目錄。

## 非互動式設定

對於 CI/CD 或腳本化設定，請使用 `--tools` 旗標：

```bash
# 設定特定工具
openspec init --tools claude,cursor

# 設定所有支援的工具
openspec init --tools all

# 跳過工具設定
openspec init --tools none
```

**可用的工具 ID：** `amazon-q`, `antigravity`, `auggie`, `claude`, `cline`, `codebuddy`, `codex`, `continue`, `costrict`, `crush`, `cursor`, `factory`, `gemini`, `github-copilot`, `iflow`, `kilocode`, `opencode`, `qoder`, `qwen`, `roocode`, `trae`, `windsurf`

## 安裝的內容

針對每個工具，OpenSpec 會產生 10 個技能檔案來驅動 OPSX 工作流程：

| 技能                           | 用途                                                |
| ------------------------------ | --------------------------------------------------- |
| `openspec-explore`             | 探索想法的思考夥伴                                  |
| `openspec-new-change`          | 開始一個新的變更                                    |
| `openspec-continue-change`     | 建立下一個成品                                      |
| `openspec-ff-change`           | 快進完成所有規劃成品                                |
| `openspec-apply-change`        | 實作任務                                            |
| `openspec-verify-change`       | 驗證實作完整性                                      |
| `openspec-sync-specs`          | 將差異規格同步到主規格（選用 — 封存會在需要時提示） |
| `openspec-archive-change`      | 封存一個已完成的變更                                |
| `openspec-bulk-archive-change` | 一次封存多個變更                                    |
| `openspec-onboard`             | 引導式導覽，完成一個完整的工作流程週期              |

這些技能透過 `/opsx:new`、`/opsx:apply` 等斜線指令呼叫。請參閱 [指令](commands.md) 獲取完整列表。

## 新增工具

想要新增對另一個 AI 編碼助理的支援嗎？請查看 [指令配接器模式 (command adapter pattern)](../CONTRIBUTING.md) 或在 GitHub 上提交 issue。

---

## 相關內容

- [CLI 參考](cli.md) — 終端機指令
- [指令](commands.md) — 斜線指令與技能
- [入門指南](getting-started.md) — 首次設定
