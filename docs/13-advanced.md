# 第 13 章　進階：平行工作、自動化、雲端與程式碼審查

## 13.1 同時跑多個 Claude

| 方式 | 誰規劃 | 規模 | 適合 |
| --- | --- | --- | --- |
| 子代理／`/subtask` | Claude 逐回合決定 | 每次幾個 | 搜尋、審查、小支線 |
| 背景 session（Agent view） | 你 | 你照顧得來的數量 | 同時推進幾個獨立任務 |
| `/batch` | Claude 拆解 | 5–30 份 | 大規模遷移、重構 |
| Agent teams | 隊長代理帶隊友 | 少數長期隊友 | 需要互相討論的長任務 |
| 動態工作流程（Workflows） | Claude 寫的腳本 | 數十～上百個代理 | 全庫稽核、大量檔案轉換、交叉驗證研究 |

### Agent view 與背景 session

![Agent view](../images/51-agentview.png)

```bash
claude agents                      # 打開 Agent view
claude --bg "調查那個不穩定的測試"   # 直接開一個背景 session
/bg                                # 把目前 session 丟到背景
/fork 改用另一種做法試試             # 複製對話成新背景 session
claude attach 名稱 / claude logs 名稱 / claude stop 名稱
```

| Agent view 按鍵 | 功能 |
| --- | --- |
| 輸入任務 + <kbd>Enter</kbd> | 派出新 session |
| <kbd>Space</kbd> | 預覽並直接回覆 |
| <kbd>Enter</kbd> 或 <kbd>→</kbd> | 進入完整對話；空白輸入按 <kbd>←</kbd> 返回 |
| <kbd>Ctrl+T</kbd> | 釘選（保持執行） |
| <kbd>Ctrl+R</kbd> | 重新命名 |
| <kbd>Ctrl+X</kbd> | 停止（2 秒內再按刪除） |
| <kbd>Ctrl+F</kbd> | 依名稱搜尋 |

- 背景 session 由背景服務執行，**關掉終端機也會繼續**；閒置約 1 小時會暫停，回覆就恢復。
- 從 Agent view 派出的 session 會自動在 `.claude/worktrees/` 下建立自己的 **git worktree**，平行修改不會互相覆蓋。

### Git worktree

手動建立：`claude -w feature-auth`；從 PR 建立：`claude -w '#1234'`。相關設定：`worktree.baseRef`、`worktree.symlinkDirectories`（共用 node_modules 省空間）。

### 動態工作流程（Workflows）與 ultracode

```
/deep-research Node.js 20 到 22 的權限模型改了什麼？     內建研究工作流程
ultracode: 稽核 src/routes/ 下所有 API 的授權檢查          單次觸發
/workflows                                              查看進度（p 暫停、x 停止、s 存成指令）
```

Claude 會寫一個 JavaScript 腳本，同時指揮多個子代理，還能讓代理互相檢查結果。好用的工作流程可以存成 `/指令` 重複使用（存到 `.claude/workflows/`）。

> ⚠️ 工作流程非常耗額度。先用小範圍試，在 `/workflows` 看每個代理的 token 消耗；`/config` → Dynamic workflow size 可限制規模。

## 13.2 自動化與排程

| | `/loop` | 桌面版排程 | 雲端排程任務 |
| --- | --- | --- | --- |
| 在哪跑 | 目前開著的 session | 你的電腦 | Anthropic 雲端 |
| 關電腦 | 停止 | 停止 | 照跑 |
| 觸發 | 固定間隔 | 排程 | 排程、API、GitHub 事件 |
| 建立 | `/loop 10m 檢查 CI` | 桌面 App | `/schedule` 或 claude.ai/code/routines |

- `/goal 所有測試通過且沒有型別錯誤`：Claude 會持續工作直到條件成立。
- `autoContinueAtUsageLimit`：額度用完時，重置後自動繼續。

### 非互動模式（腳本、CI）

```bash
claude -p "列出所有 TODO" --output-format json
claude -p "跑測試並修正失敗" --permission-mode dontAsk --allowedTools "Bash(npm test)" "Read" "Edit"
claude -p "重構 utils" --max-budget-usd 5 --max-turns 30
cat error.log | claude -p "分析這個錯誤"
```

### GitHub 整合

- session 中執行 `/install-github-app`：安裝 Claude GitHub App 並設定 Actions。之後在 issue／PR 留言 `@claude 幫忙修這個 bug`。
- **Code Review**：PR 留言 `@claude review`（單次）、`@claude review always`（每次更新都審）、`@claude review stop`（停止）。
- `/autofix-pr`：開雲端 session 盯著 PR，CI 失敗或有審查意見就自動修。

## 13.3 程式碼審查工具

| 工具 | 說明 |
| --- | --- |
| `/code-review` | 本機找 bug；`--fix` 直接修；`high`、`xhigh` 更仔細；`/code-review 1234` 審 PR |
| `/code-review ultra`（`/ultrareview`） | 雲端多代理深度審查，結果回到你的終端機 |
| `/security-review` | 資安審查（注入、授權、資料外洩） |
| `/simplify` | 找可重用、可簡化的地方並修改 |
| `/verify`、`/run` | 實際啟動 app 驗證功能 |

## 13.4 雲端、手機與遠端

- **雲端 session**：`claude --cloud "修登入 bug"` 或到 claude.ai/code；在 Anthropic 雲端跑，關電腦也繼續。`/teleport` 把雲端 session 拉回本機。
- **Remote Control**：`/rc` 或 `claude --rc` 讓目前 session 能從手機或網頁遙控（程式仍在你電腦上跑），任務完成或需要你時會推播。
- **手機 App**：`/mobile` 顯示下載 QR code。
- **Chrome 整合**：`claude --chrome` 讓 Claude 操作瀏覽器測試網頁。

## 13.5 Agent SDK

想用程式打造自己的 AI 代理？Agent SDK（TypeScript／Python）提供和 Claude Code 相同的代理迴圈、工具、權限、hooks、子代理與 MCP。文件：code.claude.com/docs/en/agent-sdk/overview。
