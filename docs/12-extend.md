# 第 12 章　擴充 Claude Code：Skills、Subagents、Hooks、MCP、Plugins、Mods

> 先搞清楚「我想解決什麼問題」，再選對工具。

| 你的困擾 | 用這個 | 一句話 |
| --- | --- | --- |
| 每次都要重講專案規矩 | CLAUDE.md（第 11 章） | 每次自動讀的說明書 |
| 某種任務有固定步驟，常常要貼同一段說明 | **Skill** | 需要時才載入的流程說明書，可用 `/名稱` 呼叫 |
| 想派專人處理特定工作，不要弄亂主對話 | **Subagent** | 有自己 context 的分身 |
| 某件事「每次都一定要發生」 | **Hook** | 事件觸發的自動腳本 |
| 要連 GitHub、資料庫、Notion 等外部系統 | **MCP** | 外部工具的連接器 |
| 想把上面這些打包分享給團隊 | **Plugin** | 擴充套件包 |
| 想改 Claude Code 的介面、加面板或即時指令 | **Mod** | 在 Claude Code 內部執行的 JS 程式 |
| 想改變回覆的語氣與格式 | **Output style** | 整個 session 的回覆風格 |

## 12.1 Skills（技能）

Skill 是一個資料夾，裡面有 `SKILL.md`。平時只有名稱和描述佔空間，Claude 判斷相關或你輸入 `/名稱` 時才載入完整內容。

**放在哪裡**：

| 範圍 | 路徑 |
| --- | --- |
| 個人（所有專案） | `~/.claude/skills/名稱/SKILL.md` |
| 專案（commit 分享） | `.claude/skills/名稱/SKILL.md` |
| Plugin 內 | `plugin/skills/名稱/SKILL.md`（呼叫：`/plugin名:skill名`） |

![/skills 清單（尚未建立任何 skill 時）](../images/30-skills.png)

**範例**：`.claude/skills/release-notes/SKILL.md`

```markdown
---
name: release-notes
description: 依 git 紀錄撰寫版本說明。使用者要求寫 changelog 或 release notes 時使用。
argument-hint: [版本號]
allowed-tools: Bash(git log *), Bash(git describe *)
disable-model-invocation: true
---

## 上個版本以來的提交
!`git log $(git describe --tags --abbrev=0)..HEAD --oneline`

## 請你
為版本 $ARGUMENTS 撰寫 release notes：
1. 分成「新功能／修正／破壞性變更」
2. 每條一句話，用使用者看得懂的語言
3. 輸出 Markdown
```

使用：`/release-notes 2.4.0`。

| 常用欄位 | 作用 |
| --- | --- |
| `description` | **最重要**，Claude 靠它判斷何時使用 |
| `disable-model-invocation: true` | 只能由你手動呼叫（適合部署等有副作用的流程） |
| `user-invocable: false` | 不出現在 / 選單，只給 Claude 當背景知識 |
| `allowed-tools` | 執行時預先核准的工具 |
| `argument-hint`、`$ARGUMENTS`、`$0`、`$1` | 參數提示與取用 |
| `` !`指令` `` | 送給 Claude 前先執行並嵌入結果 |
| `context: fork` + `agent: Explore` | 交給子代理在背景執行 |
| `model`、`effort` | 執行時改用的模型與強度 |
| `paths` | 只在處理符合的檔案時自動觸發 |

> 舊的 `.claude/commands/名稱.md` 自訂指令仍可用，但新的建議都寫成 skill。最簡單的建立方法：做完一件事後說「把剛剛的流程做成一個 skill」。

管理：`/skills`（<kbd>t</kbd> 依大小排序、<kbd>Space</kbd> 切換可見）、`/skill-doctor`（找出吃 context 又少用的 skill）、設定 `skillOverrides` 隱藏特定 skill。

## 12.2 Subagents（子代理）

子代理在**獨立的 context** 工作，只把摘要交回，主對話保持乾淨。

**內建子代理**：

| 名稱 | 用途 |
| --- | --- |
| Explore | 快速唯讀搜尋程式碼 |
| Plan | Plan 模式中的研究 |
| general-purpose | 需要查也需要動手的多步驟任務 |
| claude-code-guide | 回答 Claude Code 用法問題 |
| statusline-setup | `/statusline` 時使用 |

**自訂子代理**：放在 `.claude/agents/`（專案）或 `~/.claude/agents/`（個人）。最簡單是直接說「幫我建立一個程式碼審查的 subagent」。

```markdown
---
name: code-reviewer
description: 程式碼審查專家。程式改動後主動使用。
tools: Read, Glob, Grep, Bash(git diff *)
model: sonnet
effort: high
---

你是資深審查員。檢查正確性、安全性、可讀性，
每個問題列出 檔案:行號、原因與修正建議。
```

| 常用欄位 | 作用 |
| --- | --- |
| `name`、`description` | 必填；description 決定 Claude 何時派它 |
| `tools`／`disallowedTools` | 可用／禁用的工具 |
| `model` | sonnet、opus、haiku、fable、inherit |
| `permissionMode`、`effort`、`maxTurns` | 權限、強度、回合上限 |
| `skills`、`mcpServers`、`hooks` | 預載的 skill、專屬 MCP 與 hooks |
| `memory` | 讓它有自己的長期記憶（user／project／local） |
| `isolation: worktree` | 在獨立 git worktree 中工作 |
| `background: true` | 永遠在背景執行 |

**呼叫方式**：「用 code-reviewer 檢查我的改動」、`@"code-reviewer (agent)"`、或 `claude --agent code-reviewer` 讓整個 session 都用它。

**背景與 Fork**：

- 主對話派出的子代理**預設在背景**跑，你可以繼續做別的；它要權限時會在主畫面詢問。
- **Fork 子代理**會繼承整段對話（不用重新解釋背景），用 `/subtask 任務` 啟動，結果回到本對話。
- 分不清楚三個指令？`/subtask`＝派分身做完回報；`/fork`＝複製成另一個獨立背景 session；`/branch`＝分岔並切換過去。

## 12.3 Hooks（掛鉤）

Hook 讓某件事**每次都一定發生**，不靠 Claude 記得。例如：改完檔自動格式化、阻擋改 `.env`、需要你時跳通知。

![/hooks 檢視（唯讀，要新增請改 settings.json 或請 Claude 幫忙）](../images/27-hooks.png)

**常用事件**：

| 事件 | 何時觸發 | 常見用途 |
| --- | --- | --- |
| `PreToolUse` | 工具執行前（可阻擋） | 擋危險指令、保護檔案 |
| `PostToolUse` | 工具成功後 | 自動格式化、跑 lint |
| `UserPromptSubmit` | 你送出訊息時 | 加入額外背景、檢查敏感字 |
| `Notification` | Claude 需要你時 | 桌面通知 |
| `Stop` | Claude 回答完 | 檢查任務是否真的完成 |
| `SessionStart` | session 開始（含壓縮後） | 載入環境資訊 |
| `PreCompact`／`PostCompact` | 壓縮前後 | 保存重要資訊 |
| `SubagentStart`／`SubagentStop` | 子代理開始／結束 | 記錄 |
| `FileChanged`、`CwdChanged` | 檔案變動、切換目錄 | 自動載入 .env |
| 其他 | `PermissionRequest`、`PermissionDenied`、`PostToolBatch`、`SessionEnd`、`WorktreeCreate`、`PreModelSwitch`… | 進階用途 |

**範例 1：改完檔自動跑 Prettier**（`.claude/settings.json`）

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          { "type": "command", "command": "jq -r '.tool_input.file_path' | xargs npx prettier --write" }
        ]
      }
    ]
  }
}
```

**範例 2：禁止修改 .env**——腳本 `.claude/hooks/protect.sh`（記得 `chmod +x`）：

```bash
#!/bin/bash
FILE=$(jq -r '.tool_input.file_path // empty')
if [[ "$FILE" == *".env"* ]]; then
  echo "不可以修改 $FILE" >&2
  exit 2   # exit 2 = 阻擋，stderr 會告訴 Claude 原因
fi
exit 0     # exit 0 = 放行
```

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          { "type": "command", "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/protect.sh", "onFailure": "block" }
        ]
      }
    ]
  }
}
```

`"onFailure": "block"`：腳本本身出錯或逾時也當作阻擋，安全用途建議加上。

**範例 3：需要你時跳 Windows 通知**（PowerShell）

```json
{
  "hooks": {
    "Notification": [
      { "hooks": [ { "type": "command", "command": "powershell -c \"[console]::beep(800,300)\"" } ] }
    ]
  }
}
```

Hook 類型：`command`（執行指令）、`http`（POST 到網址）、`mcp_tool`、`prompt`（讓模型判斷 ok/不 ok）、`agent`（實驗性）。

> ⚠️ Hook 以你的權限執行。審查別人 commit 的 hooks；全部停用用 `"disableAllHooks": true`。

## 12.4 MCP：連接外部工具

```bash
# 遠端 HTTP 伺服器（最常見）
claude mcp add --transport http notion https://mcp.notion.com/mcp

# 本機程式（-- 後面是啟動指令）
claude mcp add --transport stdio airtable --env AIRTABLE_API_KEY=你的金鑰 -- npx -y airtable-mcp-server

# 管理
claude mcp list
claude mcp remove notion
claude mcp login sentry    # 完成 OAuth 登入
```

session 中輸入 `/mcp` 查看狀態、登入、啟用／停用。

| 範圍（`--scope`） | 誰能用 | 存在哪 |
| --- | --- | --- |
| local（預設） | 只有你、這個專案 | `~/.claude.json` |
| project | 團隊（commit） | 專案根目錄 `.mcp.json` |
| user | 你的所有專案 | `~/.claude.json` |

你在 claude.ai 連好的 connectors 也能在 Claude Code 使用。MCP 工具太多會吃 context，用不到的請停用。

## 12.5 Plugins（擴充套件包）

Plugin 把 skills、子代理、hooks、MCP、output styles、workflows、mods 打包，透過「市集（marketplace）」分享。

```
/plugin                                         打開選單（Discover／Installed／Marketplaces）
/plugin install code-review@claude-plugins-official
claude plugin marketplace add 公司/claude-plugins   加入公司的市集
/reload-plugins                                 不重開就套用
```

![/plugin 市集（官方市集有數百個 plugin）](../images/29-plugin.png)

團隊共用：在 `.claude/settings.json` 寫 `extraKnownMarketplaces` 與 `enabledPlugins`，隊友信任資料夾後自動取得。

> ⚠️ Plugin 可能含有會執行的程式碼，只安裝信任來源。

## 12.6 Mods（2026/10 新功能）

Mod 是一種用 JavaScript／TypeScript 寫、**在 Claude Code 內部執行**的 plugin，可以：

- 畫出面板、按鈕、提示框上方的資訊列
- 改造內建介面（例如 spinner 顯示工具次數）
- 攔截工具呼叫（例如危險指令先顯示影響範圍再讓你決定）
- 新增立即執行的 `/指令`

做法：直接請 Claude「做一個 mod，在提示框上方顯示 context 使用量」。內建的 `/diff` 面板、AGENTS.md 讀取其實就是 mods。關閉全部：`--safe-mode`（一次）或 `"disableAllHooks": true`。

> ⚠️ Mod 不在沙盒中，能讀寫你的檔案與秘密。只裝信任的；安裝前用 `claude plugin validate 路徑` 看它會做什麼。

## 12.7 Output styles（回覆風格）

![Output style 選單](../images/15-outputstyle.png)

`/output-style concise` 或 `"outputStyle": "Explanatory"`。自訂風格：在 `~/.claude/output-styles/繁中導師.md` 寫：

```markdown
---
name: 繁中導師
description: 用繁體中文、先給圖再解釋
keep-coding-instructions: true
---
所有回覆使用繁體中文（台灣用語）。解釋架構時先畫 Mermaid 圖，再用條列說明。
```

## 12.8 狀態列（Status line）

```
/statusline 顯示目前模型、git 分支、context 使用百分比
```

Claude 會幫你寫好腳本並設定。手動設定：`"statusLine": { "type": "command", "command": "~/.claude/statusline.sh" }`，腳本從 stdin 收到 JSON（模型、目錄、花費、context…），輸出的文字就是狀態列。
