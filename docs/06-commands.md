# 第 6 章　斜線指令大全

> 在輸入框打 `/` 就會跳出指令選單，繼續打字可以篩選。下面依「什麼時候用」分類。標示 🧩 的是內建 skill（交給 Claude 執行的提示），🔀 是內建工作流程。可用的指令會依方案、平台略有不同。

![斜線指令選單](../images/05-slash.png)

## 6.1 新手最常用的 12 個

| 指令 | 做什麼 | 例子 |
| --- | --- | --- |
| `/help` | 說明與快捷鍵 | `/help` |
| `/init` | 自動產生 CLAUDE.md | `/init` |
| `/clear` | 開新對話（舊的可找回） | `/clear` |
| `/resume` | 回到以前的對話 | `/resume` |
| `/model` | 換模型 | `/model sonnet` |
| `/config` | 設定面板 | `/config` |
| `/plan` | 進入規劃模式 | `/plan 幫登入加上兩步驟驗證` |
| `/diff` | 看改了什麼 | `/diff` |
| `/rewind` | 回到之前的狀態 | `/rewind` |
| `/compact` | 壓縮對話騰出空間 | `/compact 保留 API 設計的決定` |
| `/usage` | 看花費與額度 | `/usage` |
| `/doctor` 🧩 | 健檢並修正設定問題 | `/doctor` |

## 6.2 專案設定與健檢

| 指令 | 用途 |
| --- | --- |
| `/init` | 分析專案、產生 CLAUDE.md。設環境變數 `CLAUDE_CODE_NEW_INIT=1` 會用互動流程連 skills、hooks 一起設定 |
| `/memory` | 編輯 CLAUDE.md、開關自動記憶 |
| `/config [key=value]`（別名 `/settings`） | 設定面板；也能 `/config theme=dark` 直接改 |
| `/update-config 描述` 🧩 | 用一句話請 Claude 改 settings.json，例：`/update-config 允許 npm test 不用問` |
| `/permissions` | 管理允許／詢問／禁止規則 |
| `/fewer-permission-prompts` 🧩 | 分析你的使用紀錄，自動把常用唯讀指令加入白名單 |
| `/auto-mode-setup` | 讓 Auto mode 認識你的環境（哪些是測試機、哪些分支能推） |
| `/doctor`（別名 `/checkup`）🧩 | 全面健檢；`/doctor prompt-audit` 檢查你的指示檔寫得好不好 |
| `/status` | 版本、模型、帳號、載入的設定檔 |
| `/hooks`、`/mcp`、`/plugin`、`/skills`、`/agents` | 檢視／管理各類擴充 |
| `/skill-doctor` | 看每個 skill 吃多少 context、多常被用 |
| `/reload-skills`、`/reload-plugins` | 不重開就重新載入 |
| `/terminal-setup` | 安裝 Shift+Enter 換行等終端機設定 |
| `/statusline 描述` | 自訂狀態列，例：`/statusline 顯示模型與 git 分支` |
| `/theme`、`/output-style`、`/color`、`/tui`、`/keybindings` | 外觀與按鍵 |
| `/import codex|gemini|cursor` | 從其他 AI 工具匯入設定 |
| `/powerup` | 互動式小課程，學新功能 |

## 6.3 工作進行中

| 指令 | 用途 |
| --- | --- |
| `/plan [描述]` | 進入規劃模式 |
| `/model [模型]` | 換模型（<kbd>s</kbd> 只用於本次） |
| `/effort [low…max｜auto｜ultracode]` | 調整推理強度 |
| `/fast [on｜off]` | 開關 Fast mode（更快、較貴） |
| `/advisor [模型｜off]` | 關鍵時刻諮詢另一個更強的模型 |
| `/context` | 看 context 用量 |
| `/compact [重點]` | 摘要壓縮對話 |
| `/autocompact [auto｜500k]` | 設定何時自動壓縮 |
| `/btw 問題` | 旁支問題，不進入對話歷史 |
| `/goal 條件` | 設定目標，Claude 會持續工作直到達成，例：`/goal 所有測試通過` |
| `/loop [間隔] [提示]` 🧩 | 定時重複執行，例：`/loop 5m 檢查部署是否完成` |
| `/add-dir 路徑` | 加入額外可存取的資料夾 |
| `/cd 路徑` | 把 session 移到另一個資料夾（保留對話） |
| `/tasks` | 看背景指令與子代理 |
| `/subtask 任務` | 派一個繼承完整對話的子代理在背景做，結果回到這裡 |
| `/batch 指示` 🧩 | 大改動拆成 5–30 份平行處理，例：`/batch 把 src/ 從 JS 轉成 TS` |
| `/deep-research 問題` 🔀 | 多來源網路研究並交叉驗證 |
| `/workflows` | 查看／暫停／存檔動態工作流程 |
| `/chrome`、`/claude-in-chrome 任務` 🧩 | 讓 Claude 操作你的 Chrome 測網頁 |
| `/run`、`/verify` 🧩 | 實際啟動 app 驗證改動 |
| `/debug [描述]` 🧩 | 開除錯日誌並分析 |
| `/claude-api` 🧩 | 開發 Claude API 應用的輔助 |
| `/design`、`/slides`、`/dataviz` 🧩 | 設計畫板、簡報、圖表設計 |

## 6.4 收尾與交付

| 指令 | 用途 |
| --- | --- |
| `/diff` | 檢視所有改動 |
| `/code-review [等級] [--fix] [--comment] [PR編號]`（別名 `/review`）🧩 | 找 bug；`--fix` 直接修；`--comment` 貼到 PR；`ultra` 走雲端深度審查 |
| `/ultrareview` | 雲端多代理深度審查 |
| `/security-review` | 資安審查 |
| `/simplify` 🧩 | 找可簡化、可重用的地方並修改 |
| `/autofix-pr` | 開雲端 session 盯著 PR，CI 失敗就自動修 |
| `/install-github-app`、`/install-slack-app` | 安裝 GitHub／Slack 整合 |
| `/copy [N]` | 複製回應或其中的程式碼區塊 |
| `/export [檔名]` | 匯出對話成文字檔 |
| `/artifacts` | 列出你的 artifacts |

## 6.5 Session 管理

| 指令 | 用途 |
| --- | --- |
| `/clear [名稱]`（別名 `/new`、`/reset`） | 開新對話 |
| `/resume [名稱或 ID]` | 回到舊對話 |
| `/rename [名稱]` | 命名 session |
| `/branch` | 從這點分岔並切換過去（原對話保留） |
| `/fork [提示]` | 複製成新的背景 session，你繼續在這裡 |
| `/background`（`/bg`） | 把整個 session 丟到背景 |
| `/rewind`（`/undo`、`/checkpoint`） | 回溯 |
| `/recap` | 一行摘要這個 session |
| `/exit`（`/quit`） | 離開 |
| `/teleport`、`/remote-control`（`/rc`）、`/desktop` | 雲端 session 拉回本機、讓手機遙控、轉到桌面版 |
| `/schedule`（`/routines`） | 建立雲端排程任務 |
| `/list-agents`（`/peers`） | 列出可傳訊息的其他 session |

## 6.6 帳號與其他

| 指令 | 用途 |
| --- | --- |
| `/usage`（`/cost`、`/stats`） | 花費、額度、各功能用量 |
| `/insights` | 產生你的使用習慣分析報告 |
| `/rate-limit-options`、`/usage-credits`、`/upgrade` | 額度用完時的選項 |
| `/login`、`/logout` | 切換帳號 |
| `/team-onboarding` | 依你的使用紀錄產生給隊友的上手指南 |
| `/release-notes` | 看版本更新內容 |
| `/bug`、`/feedback` | 回報問題 |
| `/mobile` | 手機 App 下載 QR code |
| `/voice` | 語音輸入 |

> ⚠️ 已移除的舊指令：`/vim`（改用 `/config` → Editor mode）、`/pr-comments`（直接請 Claude 看 PR 留言）、`/ultraplan`（改用 plan 模式）。

## 6.7 自己做指令

任何 skill 都會變成 `/名稱` 指令。在 `.claude/skills/發版/SKILL.md` 寫好流程，就能輸入 `/發版` 使用。詳見第 12 章。
