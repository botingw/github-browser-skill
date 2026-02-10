# GitHub Browser Skill — 設計規格

> 此文件記錄 skill 的設計決策與原則，供未來迭代參考。不會被 Claude Code 自動載入。

## 概要

純 SKILL.md 指引的本地 skill，教 AI 用 `gh` CLI 高效瀏覽 GitHub。
第一版唯讀，架構預留寫入操作擴充空間。

## 檔案結構

```
~/.claude/skills/github-browser/
├── SKILL.md                          # 主指引（觸發條件 + 核心命令速查 + 決策樹）
├── DESIGN.md                         # 本文件（設計規格，不被自動載入）
└── references/
    ├── repo-exploration.md           # Repo 瀏覽（file tree、讀檔案、metadata）
    ├── issues-and-prs.md             # Issue/PR（內容、comments、diff、review）
    └── search.md                     # 搜尋（code/issue/repo search）
```

## 設計原則

1. **不寫腳本** — 所有指引都是教 AI 直接用 `gh` + `--jq` / `| base64 -d`，不產生 shell script
2. **SKILL.md 精簡** — 控制在 ~150 行，只放決策樹和速查；細節在 references/
3. **命令都經過實測** — 用實際 `gh` CLI 測試驗證過的命令模式
4. **預留擴充** — 決策樹和 references 可日後加入寫入操作（create issue、comment、merge 等）

## 架構決策

### 為什麼分 SKILL.md + references/?

- SKILL.md 是 Claude Code 自動載入的入口，必須精簡以控制 context 消耗
- 複雜場景（大檔案處理、PR review comments、搜尋 filter 組合）放在 references/
- AI 在需要時才讀 references，按需載入

### 為什麼用 `gh api` 而非 `gh` 子命令?

- `gh repo view`、`gh issue view` 等子命令適合快速概覽
- 但精確操作（讀特定檔案、取 PR inline comments、file tree）需要 `gh api` + `--jq`
- 決策樹根據場景導向最適合的方式

### URL 解析為什麼在 SKILL.md 裡?

- 使用者經常直接貼 GitHub URL，AI 需要快速解析出 owner/repo 和操作類型
- 放在主文件確保每次都能立即參考，不需額外讀檔

## SKILL.md 內容架構

1. **前提檢查** — `gh auth status`，未登入時提示用戶
2. **URL 解析表** — GitHub URL pattern → 操作類型的對應表
3. **決策樹** — 三大類任務的路由：理解 repo / 讀 issue-PR / 搜尋
4. **核心命令速查** — 最常用的 ~15 個命令模式
5. **指向 references/** — 複雜場景的連結

## References 設計摘要

### repo-exploration.md
- 遞迴 file tree（git/trees API）
- 讀檔案（contents API + base64 decode）
- 大檔案（>1MB，blob API fallback）
- 目錄瀏覽、repo metadata、branches/tags、releases、commits

### issues-and-prs.md
- Issue/PR 列表 + 各種 filter（state、label、assignee、milestone、author、base branch）
- Issue/PR 內容 + comments
- PR diff（`gh pr diff` + API-based `pulls/{n}/files`）
- PR inline review comments（`pulls/{n}/comments`）
- CI/checks 狀態
- Timeline 事件

### search.md
- Code search（language、filename filter、exact match）
- Issue/PR search（type、state、sort filter）
- Repo search（language、stars filter）
- JSON 輸出欄位
- Rate limit 注意事項（code: 10/min, issue/repo: 30/min）
- Code search 限制與 fallback 策略

## 驗證場景

安裝後應測試：
1. 給 GitHub repo URL → 自動讀取 README
2. 要求看特定 repo 的特定檔案
3. 要求看 PR 的 diff 和 review comments
4. 搜尋 repo 裡的特定程式碼
5. 列出 repo 的 open issues

## 迭代指引

### 加新功能時

- 先判斷：屬於「核心常用操作」還是「進階場景」？
  - 核心 → 加到 SKILL.md 的速查表（但注意 ~150 行上限）
  - 進階 → 加到對應的 reference 文件
- 如果是全新類別（例如 GitHub Actions、Discussions），考慮新增 reference 文件

### 加寫入操作時

- 在 SKILL.md 的決策樹加一個新區塊（例如「我要建立/修改」）
- 新增 `references/write-operations.md`
- 寫入操作要明確標注風險等級，教 AI 在執行前確認用戶意圖

### 修改命令時

- 先用 `gh` CLI 實際測試新命令
- 確認 `--jq` filter 的輸出格式正確
- 注意 GitHub API 的版本變更可能影響回應格式

### 不該做的事

- 不要讓 SKILL.md 膨脹超過 200 行 — 超過就拆到 references
- 不要加入需要額外工具安裝的命令 — 只依賴 `gh` CLI
- 不要寫 shell script — 保持「命令模板」的形式
- 不要在 references 之間建立循環引用
