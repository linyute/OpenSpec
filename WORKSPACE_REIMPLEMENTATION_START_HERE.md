# 工作區重新實作從這裡開始

這是為在工作區重新實作上工作的代理程式所準備的 grep 友善進入點。

實用的搜尋詞彙：

```text
workspace reimplementation
workspace poc
workspace-poc
workspace reference guide
workspace roadmap
fresh agent
start here
```

## 從這裡開始

依序閱讀這些檔案：

1. `WORKSPACE_REIMPLEMENTATION_DIRECTION.md`
2. `openspec/changes/workspace-reimplementation-roadmap/README.md`
3. `openspec/changes/workspace-reimplementation-roadmap/POC_REFERENCE_GUIDE.md`
4. 下一個實作切片的提案

POC 參考提交（commit）為：

```text
workspace-poc @ 79a45ac043f414e63d13e08b9da83b135cb20a39
```

將 POC 作為研究材料使用。請勿將其合併到實作分支中。除非切片提案或設計明確決定這樣做，否則請勿保留其架構。

## 實作順序

依序實作這些扁平的 OpenSpec 變更：

1. `workspace-foundation`
2. `workspace-create-and-register-repos`
3. `workspace-open-agent-context`
4. `workspace-change-planning`
5. `workspace-apply-repo-slice`
6. `workspace-verify-and-archive`

`workspace-reimplementation-roadmap` 是計劃的連續性和參考容器。

## 編輯之前

對於您即將實作的切片，請使用 `POC_REFERENCE_GUIDE.md` 檢查釘選的 POC 提交，然後寫下：

```text
POC findings for <slice>:

User behavior to preserve:
- ...

Tests or examples worth translating:
- ...

Implementation shortcuts to avoid:
- ...

Open design questions:
- ...
```

將持久的研究發現擷取到相關的 OpenSpec 成品中，以便未來的作業階段不依賴於對話歷史記錄。
