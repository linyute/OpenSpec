---
name: openspec-bulk-archive-change
description: 一次封存多個已完成的變更。在封存多個平行變更時使用。
license: MIT
compatibility: 需要 openspec CLI。
metadata:
  author: openspec
  version: "1.0"
  generatedBy: "1.1.1"
---

在單一操作中封存多個已完成的變更。

此技能允許您批次封存變更，透過檢查程式碼庫以確定實際實作的內容，智慧地處理 spec 衝突。

**輸入**：不需輸入（會提示進行選擇）

**步驟**

1. **取得作用中的變更**

   執行 `openspec list --json` 以取得所有作用中的變更。

   如果不存在作用中的變更，請告知使用者並停止。

2. **提示選擇變更**

   使用 **AskUserQuestion 工具** 的多選功能讓使用者選擇變更：
   - 顯示每個變更及其 schema
   - 包含「所有變更」的選項
   - 允許選擇任何數量（1 個以上即可，2 個以上是典型使用案例）

   **重要**：請勿自動選擇。務必讓使用者選擇。

3. **批次驗證 - 收集所有選定變更的狀態**

   針對每個選定的變更，收集：

   a. **Artifact 狀態** - 執行 `openspec status --change "<name>" --json`
      - 解析 `schemaName` 與 `artifacts` 列表
      - 註記哪些 artifacts 已完成 (`done`) 或處於其他狀態

   b. **任務完成情況** - 讀取 `openspec/changes/<name>/tasks.md`
      - 計算 `- [ ]`（未完成）與 `- [x]`（已完成）
      - 如果不存在任務檔案，註記為「無任務」

   c. **Delta specs** - 檢查 `openspec/changes/<name>/specs/` 目錄
      - 列出存在哪些功能 (capability) 的 specs
      - 針對每一項，提取需求名稱（符合 `### Requirement: <name>` 的行）

4. **偵測 spec 衝突**

   建立 `功能 (capability) -> [涉及該功能的變更]` 的映射表：

   ```
   auth -> [change-a, change-b]  <- 衝突（2 個以上變更）
   api  -> [change-c]            <- 正常（僅 1 個變更）
   ```

   當 2 個以上選定的變更針對相同的功能具有 delta specs 時，即存在衝突。

5. **以代理方式解決衝突**

   **針對每個衝突**，調查程式碼庫：

   a. **讀取 delta specs**，從每個衝突的變更中瞭解其聲稱新增/修改的內容

   b. **搜尋程式碼庫**以尋找實作證據：
      - 尋找實作每個 delta spec 中需求的程式碼
      - 檢查相關檔案、函式或測試

   c. **決定解決方案**：
      - 如果實際上僅實作了一個變更 -> 僅同步該變更的 specs
      - 如果兩個都已實作 -> 按時間順序套用（舊的優先，新的覆蓋）
      - 如果兩者皆未實作 -> 跳過 spec 同步，警告使用者

   d. **記錄解決方案**，針對每個衝突：
      - 套用哪個變更的 specs
      - 套用順序（如果兩個都要套用）
      - 理由（在程式碼庫中發現了什麼）

6. **顯示整合狀態表**

   顯示摘要所有變更的表格：

   ```
   | 變更              | Artifacts | 任務 | Specs   | 衝突     | 狀態  |
   | ----------------- | --------- | ---- | ------- | -------- | ----- |
   | schema-management | 已完成    | 5/5  | 2 delta | 無       | 就緒  |
   | project-config    | 已完成    | 3/3  | 1 delta | 無       | 就緒  |
   | add-oauth         | 已完成    | 4/4  | 1 delta | auth (!) | 就緒* |
   | add-verify-skill  | 剩 1 項   | 2/5  | 無      | 無       | 警告  |
   ```

   針對衝突，顯示解決方案：
   ```
   * 衝突解決方案：
     - auth spec：將先套用 add-oauth 再套用 add-jwt（皆已實作，按時間順序）
   ```

   針對未完成的變更，顯示警告：
   ```
   警告：
   - add-verify-skill：1 個未完成的 artifact，3 個未完成的任務
   ```

7. **確認批次操作**

   使用 **AskUserQuestion 工具** 進行單次確認：

   - 「是否封存 N 個變更？」並根據狀態提供選項
   - 選項可能包括：
     - 「封存所有 N 個變更」
     - 「僅封存 N 個就緒的變更（跳過未完成項）」
     - 「取消」

   如果存在未完成的變更，請明確告知它們將帶著警告被封存。

8. **執行每個已確認變更的封存**

   按決定的順序處理變更（遵循衝突解決方案）：

   a. 如果存在 delta specs，則**同步 specs**：
      - 使用 openspec-sync-specs 方法（代理驅動的智慧合併）
      - 針對衝突，依解決順序套用
      - 追蹤是否已執行同步

   b. **執行封存**：
      ```bash
      mkdir -p openspec/changes/archive
      mv openspec/changes/<name> openspec/changes/archive/YYYY-MM-DD-<name>
      ```

   c. **追蹤結果**，針對每個變更：
      - 成功：已成功封存
      - 失敗：封存過程中發生錯誤（記錄錯誤）
      - 已跳過：使用者選擇不封存（如果適用）

9. **顯示摘要**

   顯示最終結果：

   ```
   ## 批次封存完成

   已封存 3 個變更：
   - schema-management-cli -> archive/2026-01-19-schema-management-cli/
   - project-config -> archive/2026-01-19-project-config/
   - add-oauth -> archive/2026-01-19-add-oauth/

   已跳過 1 個變更：
   - add-verify-skill（使用者選擇不封存未完成項）

   Spec 同步摘要：
   - 4 個 delta specs 已同步至主 specs
   - 解決了 1 個衝突（auth：按時間順序套用了兩者）
   ```

   如果發生任何失敗：
   ```
   失敗 1 個變更：
   - some-change：目標封存目錄已存在
   ```

**衝突解決範例**

範例 1：僅實作一項
```
衝突：specs/auth/spec.md 被 [add-oauth, add-jwt] 修改

檢查 add-oauth：
- Delta 新增了 "OAuth Provider Integration" 需求
- 搜尋程式碼庫... 在 src/auth/oauth.ts 中找到了實作 OAuth 流程的內容

檢查 add-jwt：
- Delta 新增了 "JWT Token Handling" 需求
- 搜尋程式碼庫... 未發現 JWT 實作

解決方案：僅實作了 add-oauth。將僅同步 add-oauth 的 specs。
```

範例 2：兩者皆已實作
```
衝突：specs/api/spec.md 被 [add-rest-api, add-graphql] 修改

檢查 add-rest-api（建立於 2026-01-10）：
- Delta 新增了 "REST Endpoints" 需求
- 搜尋程式碼庫... 發現 src/api/rest.ts

檢查 add-graphql（建立於 2026-01-15）：
- Delta 新增了 "GraphQL Schema" 需求
- 搜尋程式碼庫... 發現 src/api/graphql.ts

解決方案：兩者皆已實作。將先套用 add-rest-api 的 specs，
再套用 add-graphql 的 specs（時間順序，較新的優先）。
```

**成功輸出**

```
## 批次封存完成

已封存 N 個變更：
- <change-1> -> archive/YYYY-MM-DD-<change-1>/
- <change-2> -> archive/YYYY-MM-DD-<change-2>/

Spec 同步摘要：
- N 個 delta specs 已同步至主 specs
- 無衝突（或：已解決 M 個衝突）
```

**部分成功輸出**

```
## 批次封存完成（部分成功）

已封存 N 個變更：
- <change-1> -> archive/YYYY-MM-DD-<change-1>/

已跳過 M 個變更：
- <change-2>（使用者選擇不封存未完成項）

失敗 K 個變更：
- <change-3>：目標封存目錄已存在
```

**無變更時的輸出**

```
## 無變更可供封存

找不到作用中的變更。使用 `/opsx:new` 建立新變更。
```

**Guardrails**
- 允許任何數量的變更（1 個以上即可，2 個以上是典型使用案例）
- 務必提示進行選擇，絕不自動選擇
- 提早偵測 spec 衝突，並透過檢查程式碼庫解決
- 當兩個變更皆已實作時，按時間順序套用 specs
- 僅在缺失實作時跳過 spec 同步（並警告使用者）
- 在確認前顯示每個變更的清晰狀態
- 針對整個批次進行單次確認
- 追蹤並回報所有結果（成功/跳過/失敗）
- 移動至封存時保留 `.openspec.yaml`
- 封存目錄目標使用目前日期：`YYYY-MM-DD-<name>`
- 如果目標封存目錄已存在，則該變更失敗，但繼續處理其他變更
