# 第 2 章　Chat 與 Code 差在哪？Claude 有哪些用法？

> 很多人第一次打開 Claude 桌面版會看到「Chat」和「Code」兩個分頁，不知道該用哪個。這章幫你一次弄清楚。

## 2.1 一張表看懂

| | **Chat（聊天）** | **Code（Claude Code）** |
| --- | --- | --- |
| 本質 | 和 Claude 對話、一起思考與寫作 | 讓 Claude 在你的**程式專案資料夾**裡直接動手 |
| 能碰到你的檔案嗎 | 只能看你**上傳**的檔案 | 可以讀寫整個專案資料夾 |
| 能執行指令嗎 | 不能在你電腦上執行 | 可以（安裝套件、跑測試、git…） |
| 成果在哪 | 對話中的回答、文件、artifact 網頁 | 直接改好的程式碼、commit、PR |
| 適合 | 問問題、寫文章／信、分析資料、研究、腦力激盪、學概念 | 寫程式、修 bug、重構、寫測試、自動化、整理專案 |
| 用到的設定 | claude.ai 設定（個人偏好、記憶、connectors） | `~/.claude/settings.json`、CLAUDE.md 等（本教學主要內容） |

> 💡 **簡單判斷法**：「我要的東西最後是**一段文字／一份文件**」→ Chat；「我要的東西最後是**改好的程式碼／一個能跑的專案**」→ Code。

同一個訂閱方案（Pro、Max、Team、Enterprise）的用量額度由 Chat 與 Claude Code 共用。

## 2.2 Chat 能做的事

- **一般對話**：問答、解釋觀念、翻譯、寫作、改稿。
- **上傳檔案分析**：PDF、Word、Excel、圖片、程式碼。
- **Artifacts**：讓 Claude 產生可分享的網頁、文件、簡報、試算表。
- **Projects（專案）**：把相關資料與指示集中，所有對話共用。
- **研究（Research）與網路搜尋**：查最新資訊並附來源。
- **Connectors**：連 Google Drive、Gmail、Slack、Notion 等，讓 Claude 讀你自己的資料。
- **記憶**：記住你的偏好與背景，下次對話自動帶入。

適合「還沒進入寫程式階段」的事，例如：先在 Chat 討論架構與需求，確定後再到 Code 實作。

## 2.3 Code 的各種入口

Claude Code 不只一種用法，以下都是同一套 Claude Code，**共用相同的設定檔**（雲端 session 例外）：

| 入口 | 長什麼樣子 | 適合誰 |
| --- | --- | --- |
| **終端機（CLI）** | 在終端機輸入 `claude` | 功能最完整，本教學截圖都來自這裡 |
| **桌面版 Code 分頁** | Claude 桌面 App 中的 Code 分頁，有圖形介面 | 不熟終端機的新手、喜歡視覺化 diff 的人 |
| **VS Code / Cursor 擴充** | 編輯器側邊的聊天面板 | 平常就用 VS Code 的人 |
| **JetBrains 外掛** | IntelliJ、PyCharm 等 | JetBrains 使用者 |
| **網頁版（claude.ai/code）** | 在 Anthropic 雲端執行，關掉電腦也繼續跑 | 長任務、手機上派工 |
| **手機 App** | 遠端遙控電腦上的 session，或開雲端 session | 外出時查看進度 |
| **Slack** | 在頻道 @Claude 派工 | 團隊協作 |
| **GitHub Actions** | 在 PR／issue 中 `@claude` | 自動化審查、修 bug |

## 2.4 桌面版 Code 分頁介面導覽

桌面版把終端機的功能做成按鈕與面板，主要元素：

| 介面元素 | 位置 | 功能 | 對應 CLI 操作 |
| --- | --- | --- | --- |
| **+ New**（或 Ctrl+N／Cmd+N） | 左側欄上方 | 開新 session | 執行 `claude` |
| **Session 清單** | 左側欄 | 所有對話，可同時跑多個 | `/resume`、Agent view |
| **環境選擇器** | 輸入框下方第一個 | 在哪裡執行：**Local**（本機）、**Cloud**（雲端）、**SSH**（遠端主機）、**WSL**（Windows） | `--cloud` 等 |
| **資料夾選擇** | 輸入框下方 | 選要工作的專案資料夾 | `cd 專案` |
| **權限模式選擇器** | 輸入框下方 | Manual／Accept edits／Plan／Auto／Bypass | <kbd>Shift+Tab</kbd>（桌面版用 Ctrl/Cmd+Shift+M） |
| **模型選擇** | 輸入框下方 | 切換模型 | `/model` |
| **Worktree 按鈕** | 開始 session 前 | 在獨立 git worktree 中工作 | `claude -w` |
| **Changes（差異）面板** | 標題列／Ctrl/Cmd+Shift+D | 檢視改動、在行上留言請 Claude 修 | `/diff` |
| **Browser 面板** | Ctrl/Cmd+Shift+B | 內建瀏覽器，Claude 可開 dev server 驗證畫面 | `--chrome` |
| **Terminal 面板** | Ctrl+\` | 與 session 共用環境的終端機 | — |
| **Files、Plan、Background tasks 面板** | 標題列 ⋮ | 檔案編輯、計畫、背景工作 | `/tasks` |
| **iOS Simulator 面板** | （macOS） | 讓 Claude 操作模擬器測試 App | — |
| **Settings → Code（或 Claude Code）** | 設定頁 | 提示建議、Worktree 位置與分支前綴、合併後自動封存、Remote Control、電腦操作等 | `/config` |

桌面版與 CLI 的差異：

- 桌面版沒有 `dontAsk` 模式；`/config` 在桌面版會開啟設定頁而不是終端機選單。
- 桌面版會自行設定部分環境變數（例如本機 session 的 API 逾時），所以在設定檔改這些變數可能無效。
- 雲端 session 只支援 Accept edits、Plan、Auto 三種模式。
- 桌面版的 `.claude/launch.json` 可設定 dev server，讓 Claude 自動開瀏覽器驗證改動。

> 📷 **關於截圖**：桌面 App 無法被自動化工具控制，本教學的截圖皆擷取自同版本（v2.1.312）的終端機 CLI。桌面版的對話內容、權限詢問、差異顯示與 CLI 是同一套引擎，看懂 CLI 畫面，桌面版自然看得懂。

## 2.5 Claude Code 能幫你做什麼？（實際例子）

| 類型 | 你可以這樣說 |
| --- | --- |
| 理解專案 | 「這個專案在做什麼？入口在哪？畫出資料流」 |
| 修 bug | 「使用者輸入錯誤密碼後畫面空白，幫我找原因並修好」 |
| 新功能 | 「在註冊表單加上 email 格式驗證，並寫測試」 |
| 重構 | 「把 auth 模組從 callback 改成 async/await」 |
| 測試 | 「幫 utils/ 底下沒有測試的函式補上單元測試」 |
| Git | 「幫我寫 commit 訊息」「解這個 merge conflict」「開 PR」 |
| 文件 | 「依目前程式更新 README 的安裝說明」 |
| 自動化 | 「每天早上檢查 CI 狀態並整理失敗原因」 |
| 非程式工作 | 整理資料夾、批次改檔名、處理 CSV、寫腳本 |

## 2.6 其他你該知道的基礎知識

- **終端機（Terminal）**：輸入文字指令操作電腦的視窗。macOS 是「終端機」App；Windows 是 PowerShell 或 Windows Terminal。第 3 章會教最基本的操作。
- **專案資料夾**：Claude Code 以「你啟動它的資料夾」為工作範圍。請先 `cd` 到你的專案再執行 `claude`。
- **Git**：版本控制工具，能記錄每次修改、隨時還原。強烈建議專案都用 git 管理，Claude 改壞了可以輕鬆回復。
- **訂閱 vs API**：用 claude.ai 訂閱登入，費用包含在月費內（有額度上限）；用 Claude Console API key，則依 token 用量計費。
- **安全意識**：Claude Code 能執行指令，請只在你信任的資料夾使用；不要把密碼、金鑰寫在對話裡；重要操作仔細看確認框。
