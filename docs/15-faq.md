# 第 15 章　疑難排解、最佳實踐與新功能時間軸

## 15.1 常見問題

| 狀況 | 怎麼辦 |
| --- | --- |
| 安裝後找不到 `claude` | 重開終端機；仍不行參考官方 Troubleshoot installation 修正 PATH |
| 不知道哪裡壞了 | session 中 `/doctor`（會提議修正）；終端機 `claude doctor`（只診斷） |
| 懷疑是自己的設定造成 | `claude --safe-mode` 停用所有自訂後比較 |
| 設定沒生效 | `/status` 看 Setting sources；檢查 JSON 格式（不能有註解、結尾逗號）；是否被更高優先的檔案覆蓋；專案設定是否還沒信任資料夾 |
| Claude 不照 CLAUDE.md 做 | 寫得更具體簡短；`/doctor prompt-audit`；必須遵守的改用 hook 或 deny 規則 |
| 一直跳權限詢問 | `/fewer-permission-prompts`；回答時選「don't ask again」；考慮 Auto mode 或沙盒 |
| Auto mode 一直擋 | `/permissions` → Recently denied 手動重試；`/auto-mode-setup`；在對話中明確授權 |
| 回答品質變差、忘東忘西 | context 太滿：`/clear` 開新對話或 `/compact` |
| 額度用太快 | `/usage` 看誰最耗；降低 effort；子代理用 Sonnet；少用 workflows |
| Hook 沒觸發 | `/hooks` 確認；腳本 `chmod +x`；`claude --debug` 看執行紀錄 |
| MCP 連不上 | `/mcp` 看狀態、重新登入；`claude --debug='mcp'` |
| 畫面亂掉 | <kbd>Ctrl+L</kbd>；試 `/tui fullscreen` |
| 想回報 bug | `/bug`（可選擇附多少對話，送出前有確認） |

## 15.2 給新手的十條最佳實踐

1. **專案一定用 git**，改壞了能救回來。
2. **先問再改**：先讓 Claude 讀懂、解釋，再要求修改。
3. **大改動先 Plan**：<kbd>Shift+Tab</kbd> 到 Plan 模式，看過計畫再執行。
4. **給驗證方法**：告訴它「完成的標準」（哪個測試要過、畫面要長怎樣）。
5. **方向不對立刻 <kbd>Esc</kbd>**，不要等它做完。
6. **一個任務一個對話**，換題目就 `/clear`。
7. **重複講的事寫進 CLAUDE.md**，重複的流程做成 skill。
8. **仔細看權限詢問框**，尤其是刪除、推送、部署。
9. **秘密檔案設 deny**：`Read(./.env)`。
10. **每次看 `/diff` 再 commit**。

## 15.3 推薦設定（複製即用）

**新手安心版**（`~/.claude/settings.json`）：

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "language": "繁體中文",
  "outputStyle": "Explanatory",
  "permissions": {
    "defaultMode": "default",
    "deny": ["Read(./.env)", "Read(./.env.*)", "Read(./secrets/**)"]
  }
}
```

**日常效率版**：

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "language": "繁體中文",
  "outputStyle": "Concise",
  "effortLevel": "high",
  "permissions": {
    "allow": ["Bash(git status)", "Bash(git diff *)", "Bash(git log *)", "Bash(npm run test *)", "Bash(npm run lint)"],
    "deny": ["Read(./.env)", "Read(./.env.*)"]
  },
  "env": { "CLAUDE_CODE_SUBAGENT_MODEL": "sonnet" }
}
```

**團隊專案**（`.claude/settings.json`，commit 進 repo）：

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "permissions": {
    "allow": ["Bash(pnpm test *)", "Bash(pnpm lint)", "Bash(pnpm typecheck)"],
    "ask": ["Bash(git push *)"],
    "deny": ["Read(./.env)", "Edit(./src/generated/**)"]
  },
  "hooks": {
    "PostToolUse": [
      { "matcher": "Edit|Write", "hooks": [ { "type": "command", "command": "pnpm prettier --write ." } ] }
    ]
  }
}
```

## 15.4 2026 年 8–10 月重要更新

| 日期（版本） | 更新 |
| --- | --- |
| 8 月上旬 | Session 之間可互傳訊息；自架雲端環境 beta；Auto mode 成為 Pro／Max／Team 新 session 預設 |
| 8 月中 | 桌面版額度重置後自動續跑；fork 子代理預設開啟；`@` 可提及其他 session |
| 8 月下旬 | `/design` 預覽；Concise 回覆風格；桌面版可 `/resume` 終端機 session |
| 9 月初 | Claude Fable 5.1 上線；`/skill-doctor`；全螢幕 `/diff` 即時面板 |
| 9/18（2.1.277） | 沒有 CLAUDE.md 時自動讀 AGENTS.md |
| 9/22（2.1.280） | **Claude Opus 5.5** 成為預設模型 |
| 9/28（2.1.284） | Sonnet 5.5；互動模式全面預設 Auto mode |
| 10/1（2.1.287） | **Mods** 正式上線 |
| 10/7（2.1.293） | Haiku 5.5 成為預設 Haiku |
| 10/8（2.1.295） | Hooks 支援 `onFailure: "block"` |
| 10/14（2.1.299） | **閒置壓縮（Idle compaction）** |
| 10/16（2.1.302） | fork 子代理在 `-p`、SDK、桌面版、VS Code 預設開啟 |
| 10/19（2.1.304） | `/auto-mode-setup` 支援更多供應商；Code Review 可設 Quick／Standard／Deep |
| 10/21（2.1.307） | `@claude review stop`；安全事件遙測 |
| 10/23（2.1.310） | 沙盒強化；`claude ultrareview` 需信任工作區 |
| 10/26（2.1.311） | `ultrathink` 在 5.5 系列以 xhigh 執行該回合 |
| 10/27（2.1.312） | 大量修正（例如大型 CLAUDE.md 不再被覆寫） |

最新內容請在 Claude Code 中輸入 `/release-notes`。

## 15.5 參考資料

- [Claude Code 官方文件](https://code.claude.com/docs/en/overview)（有繁體中文版）
- [Changelog（版本更新紀錄）](https://code.claude.com/docs/en/changelog)
- [What's new（每週重點）](https://code.claude.com/docs/en/whats-new/index)
- [Settings](https://code.claude.com/docs/en/settings)、[All settings](https://code.claude.com/docs/en/settings-reference)
- [Commands](https://code.claude.com/docs/en/commands)、[CLI reference](https://code.claude.com/docs/en/cli-reference)、[Interactive mode](https://code.claude.com/docs/en/interactive-mode)
- [Permission modes](https://code.claude.com/docs/en/permission-modes)、[Sandboxing](https://code.claude.com/docs/en/sandboxing)
- [Model configuration](https://code.claude.com/docs/en/model-config)、[Fast mode](https://code.claude.com/docs/en/fast-mode)
- [Memory](https://code.claude.com/docs/en/memory)、[Skills](https://code.claude.com/docs/en/skills)、[Subagents](https://code.claude.com/docs/en/sub-agents)、[Hooks](https://code.claude.com/docs/en/hooks-guide)、[MCP](https://code.claude.com/docs/en/mcp)
- [Plugins](https://code.claude.com/docs/en/plugins/overview)、[Mods](https://code.claude.com/docs/en/plugins/mods/overview)、[Workflows](https://code.claude.com/docs/en/workflows)、[Agent view](https://code.claude.com/docs/en/agent-view)
- [Desktop app](https://code.claude.com/docs/en/desktop)、[Output styles](https://code.claude.com/docs/en/output-styles)
- 免費課程：[Claude Code 101（Claude Academy）](https://academy.claude.com/courses/claude-code-101)
