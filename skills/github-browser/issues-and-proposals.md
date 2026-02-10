# GitHub Browser Skill — Issues & Improvement Proposals

> 來自實際使用經驗的改進提案。記錄在此供未來迭代參考。

---

## Issue #1: `--json` field name 在不同 API 間不一致 ✅ DONE

**問題：** `gh search repos --json` 的 field 是 `stargazersCount`（有 s），但 `gh repo view --json` 的 field 是 `stargazerCount`（沒有 s）。直接導致命令失敗浪費一個 round。

**影響：** 任何需要跨 search API 和 repo view API 的操作都會踩到。

**建議：** 在 `search.md` 加一個 field name 對照表：

| 用途 | search API field | repo view field |
|------|-----------------|-----------------|
| Star 數 | `stargazersCount` | `stargazerCount` |

應實際測試更多 field 差異後補完此表。

---

## Issue #2: GitHub search 不支援布林 OR ✅ DONE

**問題：** `gh search repos "langchain OR langgraph"` 不會做布林搜尋，而是把 "OR" 當成 literal text 搜尋。結果全是無關 repo。

**影響：** 任何需要搜尋多個相關 keyword 的場景。

**建議：** 在 `search.md` 明確記錄：
- GitHub search **不支援** `OR` / `AND` 布林語法（repo search 中）
- 正確做法是**分開執行多次搜尋**，再合併結果
- 注意 code search 的 syntax 跟 repo search 不完全一樣

---

## Issue #3: 缺少 org 級別瀏覽

**問題：** Skill 只教了 repo 級別的瀏覽，沒有 org 級別的瀏覽方式。很多時候想看一個組織（如 `langchain-ai`）底下的所有 repo。

**建議：** 在 `repo-exploration.md` 加入：

```bash
# 列出 org 所有 repo，按 stars 排序
gh api orgs/{org}/repos?sort=stars&per_page=30 --jq '.[].full_name'

# 含 star 數和描述
gh api orgs/{org}/repos?sort=stars&per_page=30 \
  --jq '.[] | "\(.stargazers_count) | \(.full_name) | \(.description)"'
```

---

## Issue #4: 缺少 README 摘要的最佳實踐

**問題：** 調查 repo 時經常需要快速抓 README 摘要，但目前的方法各有缺點：

| 方法 | 優點 | 缺點 |
|------|------|------|
| `gh repo view {owner}/{repo}` | 乾淨渲染、包含 metadata | 無法平行、一次只能看一個 |
| `gh api .../contents/README.md` + base64 -d | 可平行呼叫 | 原始 markdown，前 30 行常是 badges/HTML |

**建議：** 記錄一個「批量 repo 調查」workflow：
1. 用 `gh search repos` 粗篩候選
2. 用平行 `gh api` 讀 README（搭配 `head -50` 取更多行）
3. 對特別感興趣的 repo 用 `gh repo view` 取完整資訊

---

## Proposal #1: 加入「Research Workflow」到決策樹 🔄 DONE — 待測試

**背景：** 目前 SKILL.md 的決策樹有三類：理解 repo / 讀 issue-PR / 搜尋。但最常見的使用場景之一是「調查某個主題/生態的熱門 repos」，這不屬於以上任何一類。

**建議：** 在決策樹加入第四類：

```
### "I want to research a topic/ecosystem"
1. Identify search dimensions (see Proposal #2)
2. Multi-channel search: repo search + code search + topic + org browsing
3. Deduplicate and rank results
4. Deep dive into top candidates (README, structure, activity)
→ See references/research-workflow.md
```

新增 `references/research-workflow.md` 記錄完整的調查 playbook。

---

## Proposal #2: 加入「搜尋策略思維框架」 🔄 DONE — 待測試

**背景：** Skill 目前只教命令用法，不教搜尋思維。AI 收到「找 X 相關的 repos」時，會直接把 X 當 keyword 去 `gh search repos`，但這是最表層的搜尋。

**問題本質：** 「相關」有多個維度，不同維度需要不同搜尋方式：

| 「相關」的意思 | 適合的搜尋方式 |
|---------------|---------------|
| Repo 名稱/描述提到 X | `gh search repos "X"` |
| Code 裡 import / 使用了 X | `gh search code "from X" --language python` |
| 依賴了 X（在 requirements.txt 等） | `gh search code "X" --filename requirements.txt` |
| GitHub topics 標了 X | `gh search repos --topic X` |
| 屬於 X 的官方組織 | `gh api orgs/{X-org}/repos` |
| 社群討論/推薦的 X 生態專案 | 超出 gh CLI 範圍，需 web search |

**建議：** 在 `search.md` 或新增 `references/search-strategy.md` 加入：

### 搜尋策略決策流程

```
收到搜尋請求時：
1. 解析意圖 — 「相關」是指什麼？名稱包含？程式碼使用？同生態？
2. 選擇搜尋管道 — 至少用 2-3 種不同管道
   a. repo search（名稱/描述）
   b. code search（實際使用）
   c. topic search（社群標籤）
   d. org browsing（官方生態）
3. 交叉比對 — 在多個管道出現的 repo 更值得注意
4. 從結果發現新線索 — 例如反覆出現的 org、相關 topic、類似專案的 "awesome-X" 列表
5. 迭代深入 — 根據新線索調整搜尋
```

### Code search 找依賴的實用命令

```bash
# 找 Python 專案中 import langchain 的 repo
gh search code "from langchain" --language python --sort indexed

# 找在 requirements.txt 中依賴 langgraph 的 repo
gh search code "langgraph" --filename requirements.txt

# 找在 pyproject.toml 中依賴 langchain 的 repo
gh search code "langchain" --filename pyproject.toml

# 找特定 topic 的 repo
gh search repos --topic langchain --sort stars
gh search repos --topic ai-agent --sort stars
```

---

## Proposal #3: README 前段 noise 問題

**問題：** 用 `gh api` + base64 decode 讀 README 時，前 30 行通常充斥 badges、logo HTML、空行。真正的 project description 可能在第 30-50 行甚至更後面。

**可能的解法：**
1. 增加 `head` 行數到 50-60
2. 用 `--jq` 或 `sed` 過濾 HTML 標籤和 badge 行
3. 改用 `gh repo view` 的 `--json description` 取乾淨的一行描述（但資訊量少）
4. 混合策略：先用 description 做初篩，對感興趣的再讀完整 README

**目前沒有完美解法，** 記錄在此供未來迭代時考慮。

---

## 優先級與狀態

| 項目 | 影響 | 優先 | 狀態 | 修改檔案 |
|------|------|------|------|---------|
| Issue #1: field name 差異 | 高 | P0 | ✅ Done | `search.md`, `search-strategy.md` |
| Issue #2: 布林 OR 不支援 | 中 | P0 | ✅ Done | `search.md`, `search-strategy.md` |
| Issue #3: org 級瀏覽 | 中 | P1 | 📋 Open | — |
| Issue #4: README 摘要 | 低 | P1 | 📋 Open | — |
| Proposal #1: Research Workflow | 高 | P1 | 🔄 Done, 待測試 | `SKILL.md` 決策樹 |
| Proposal #2: 搜尋策略框架 | 高 | P1 | 🔄 Done, 待測試 | `SKILL.md`, 新增 `references/search-strategy.md` |
| Proposal #3: README noise | 低 | P2 | 📋 Open | — |
