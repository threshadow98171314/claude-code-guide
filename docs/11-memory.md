# 第 11 章　記憶：CLAUDE.md、Rules、AGENTS.md 與自動記憶

> 每個新 session，Claude 都像第一天上班的同事。「記憶」功能讓它每次都知道你的專案規矩和你的習慣。

## 11.1 兩種記憶

| | CLAUDE.md | 自動記憶（Auto memory） |
| --- | --- | --- |
| 誰寫 | 你 | Claude 自己 |
| 內容 | 指示與規則 | 學到的偏好、你糾正過的事、專案脈絡 |
| 範圍 | 專案、個人或組織 | 每個 repo 一份 |
| 載入 | 每個 session 自動讀 | 每個 session 自動讀（前 200 行或 25KB） |

![/memory 對話框](../images/26-memory.png)

> 記憶是「提醒」不是「強制」。要**保證**某事絕不發生，請用 deny 規則或 hook。

## 11.2 CLAUDE.md 放哪裡

| 類型 | 位置 | 誰看得到 | 放什麼 |
| --- | --- | --- | --- |
| 個人 | `~/.claude/CLAUDE.md` | 只有你，所有專案 | 「回覆用繁體中文」「我習慣用 pnpm」 |
| 專案 | `./CLAUDE.md` 或 `./.claude/CLAUDE.md` | 團隊（commit 進 git） | 架構、指令、規範 |
| 專案本機 | `./CLAUDE.local.md` | 只有你（加入 .gitignore） | 你的測試帳號、本機網址 |
| 組織 | IT 部署 | 全公司 | 公司規範 |

從工作目錄往上每一層的 CLAUDE.md 都會載入並**合併**（越靠近工作目錄的越後讀）；子資料夾的 CLAUDE.md 在 Claude 碰到那個資料夾的檔案時才載入。

## 11.3 怎麼寫一份好的 CLAUDE.md

先 `/init` 產生初稿，再修改。範例：

```markdown
# 專案：Acme 網站

## 常用指令
- 安裝：pnpm install
- 開發：pnpm dev（http://localhost:3000）
- 測試：pnpm test；單一檔案：pnpm test -- path/to/file
- 提交前必須通過：pnpm typecheck && pnpm lint

## 程式規範
- TypeScript strict，禁止使用 any
- API 路由一律用 withAuth() 包起來
- 新元件放 src/components/，並附 *.test.tsx

## 不要做的事
- 不要修改 src/generated/（自動產生）
- 資料庫 migration 要另開 PR

@docs/architecture.md
```

原則：**具體、簡短、可驗證**。「程式要寫好」沒用；「函式超過 50 行要拆分」有用。用 `/doctor prompt-audit` 檢查寫得好不好。

**匯入其他檔案**：寫 `@路徑` 即可，例如 `@docs/architecture.md`、`@~/.claude/my-rules.md`。

## 11.4 `.claude/rules/`：依檔案類型套用的規則

```markdown
---
paths:
  - "src/components/**/*.tsx"
---
- 元件一律用 function component 與 hooks
- props 型別命名為 XxxProps
```

有 `paths` 的規則只在 Claude 碰到符合的檔案時載入，比全部塞進 CLAUDE.md 更省 context。個人通用規則放 `~/.claude/rules/`。

## 11.5 AGENTS.md（與其他 AI 工具共用）

很多 AI 編碼工具使用 `AGENTS.md`。Claude Code 的規則：

| 你的專案有 | Claude 讀 |
| --- | --- |
| 只有 AGENTS.md | AGENTS.md |
| AGENTS.md 和 CLAUDE.md | 只讀 CLAUDE.md |
| CLAUDE.md 裡寫了 `@AGENTS.md` | 兩者（不重複） |

可在 `/config` → Project instructions 改成兩者都讀：

![Project instructions](../images/22-projinstr.png)

> 💡 最穩的做法：共用內容寫在 AGENTS.md，CLAUDE.md 第一行寫 `@AGENTS.md`，下面再補 Claude 專屬的說明。

## 11.6 自動記憶

Claude 會在工作中自己記下四類筆記：

| 類型 | 例子 |
| --- | --- |
| user（關於你） | 「使用者是後端工程師，熟 Go，不熟前端」 |
| feedback（你的糾正） | 「使用者要求測試用 vitest 不要用 jest」 |
| project（專案脈絡） | 「v2 API 預計 11 月上線，舊端點暫不刪除」 |
| reference（資訊在哪） | 「bug 追蹤在 Linear 的 WEB 專案」 |

- 存在 `~/.claude/projects/<專案>/memory/`，`MEMORY.md` 是索引，都是普通 Markdown，可以直接打開修改或刪除。
- 開關：`/memory` 的 Auto-memory、設定 `"autoMemoryEnabled": false`，或環境變數 `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1`。
- 想讓它記住某件事，直接說「記住：這個專案的測試要用 Docker 跑」。

## 11.7 大型 monorepo 小技巧

- 用 `"claudeMdExcludes": ["**/other-team/**/CLAUDE.md"]` 跳過別組的指示。
- 各子專案放自己的 CLAUDE.md，只在進入時載入。
- 共用規則用 `.claude/rules/` 搭配 `paths`。
