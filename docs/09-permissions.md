# 第 9 章　權限、Auto mode 與沙盒

> Claude Code 能改檔、跑指令，所以「它可以做什麼、什麼要先問、什麼絕對不行」是最重要的安全設定。

## 9.1 六種權限模式

| 模式（設定值） | 畫面顯示 | 不用問就能做 | 什麼時候用 |
| --- | --- | --- | --- |
| Manual（`default`／`manual`） | `⏸ manual mode on` | 只有讀取 | 第一次用、重要專案、想逐步審核 |
| Accept edits（`acceptEdits`） | `⏵⏵ accept edits on` | 讀取、改檔、工作目錄內常見檔案指令（mkdir、mv、cp、rm…） | 信任方向，想加快但仍想審核指令 |
| Plan（`plan`） | `⏸ plan mode on` | 讀取與安全的查詢指令；**核准計畫前不能改檔** | 大改動前先討論方案 |
| Auto（`auto`） | `⏵⏵ auto mode on` | 全部，但每個動作先經分類器審查 | 互動模式的預設；日常使用 |
| Don't ask（`dontAsk`） | `⏵⏵ don't ask on` | 只有讀取與事先核准的工具，其他直接拒絕 | 自動化腳本、CI |
| Bypass（`bypassPermissions`） | `⏵⏵ bypass permissions on` | 幾乎全部，完全不問 | **只在隔離的容器／虛擬機中使用** |

切換方式：

- 互動中按 <kbd>Shift+Tab</kbd>（桌面版用模式選擇器或 Ctrl/Cmd+Shift+M）。
- 啟動時指定：`claude --permission-mode plan`。
- 設為預設：`"permissions": { "defaultMode": "acceptEdits" }`（`auto` 與 `bypassPermissions` 不能寫在專案設定檔，避免別人替你開大門）。

![Manual](../images/06-mode-manual.png)
![Accept edits](../images/06-mode-acceptedits.png)
![Plan](../images/06-mode-plan.png)
![Auto](../images/06-mode-auto.png)

> 🔒 **兩條永遠成立的規則**：① deny（禁止）規則在任何模式都有效，包括 bypass。② 寫入 `.git`、`.claude`、`.bashrc` 等受保護路徑，除 bypass 外都不會自動核准。

## 9.2 權限詢問框怎麼回答

修改檔案時：

![改檔詢問](../images/43-edit-permission.png)

執行指令時：

![指令詢問](../images/45-bash-permission.png)

| 選項 | 效果 |
| --- | --- |
| Yes | 只允許這次 |
| Yes, and don't ask again for: `npm test *` | 記住規則（寫入 `.claude/settings.local.json`），以後同類指令不再問 |
| Yes, and switch to accept edits／auto mode | 本 session 切換到較寬鬆的模式 |
| No | 拒絕；接著告訴 Claude 你希望怎麼做 |
| <kbd>Tab</kbd> | 回答的同時附上補充說明（例：「可以，但只改這個函式」） |

## 9.3 Auto mode 深入了解

Auto mode 用一個**獨立的安全分類器模型**，在每個動作執行前審查。它只看你的要求與 Claude 打算做的動作，看不到檔案或網頁內容，因此惡意網頁中的「指令」騙不了它。

| 預設會擋下 | 預設允許 |
| --- | --- |
| `curl … \| bash` 下載後執行 | 工作目錄內的檔案操作 |
| 把敏感資料送到外部 | 依 lock 檔安裝相依套件 |
| 正式環境部署、資料庫 migration | 讀 `.env` 並送給對應的 API |
| `git push --force`、`git reset --hard` | 唯讀的網路請求 |
| `terraform destroy` 之類的破壞操作 | 推送到目前 repo 的一般分支 |
| 合併未核准的 PR、提交秘密資訊 | |

- 連續被擋 3 次或累計 20 次，會暫停 Auto mode 改回詢問。
- 被擋的動作會出現在 `/permissions` 的 **Recently denied**，可以手動重試。
- 你在對話中說「不要 push」，分類器也會當成限制。
- 用 `/auto-mode-setup` 讓它認識你的環境（例如哪台是測試機）。
- `claude auto-mode defaults` 可看內建規則。

> ⚠️ Auto mode 能大幅減少打擾，但不保證 100% 安全。敏感操作請搭配 deny 規則、hooks、沙盒。

## 9.4 自訂規則：allow／ask／deny

用 `/permissions` 管理，或寫進設定檔：

![/permissions](../images/24-permissions.png)

```json
{
  "permissions": {
    "allow": [
      "Bash(npm run lint)",
      "Bash(npm run test *)",
      "Bash(git status)",
      "Bash(git diff *)",
      "WebFetch(domain:github.com)"
    ],
    "ask": ["Bash(git push *)"],
    "deny": [
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)",
      "Bash(rm -rf *)"
    ]
  }
}
```

| 規則寫法 | 意思 | 範例 |
| --- | --- | --- |
| `Bash(指令)` | 完全符合的指令 | `Bash(npm run build)` |
| `Bash(開頭 *)` | 以此開頭的指令 | `Bash(git log *)` |
| `Read(路徑)`／`Edit(路徑)` | 讀取／修改檔案，支援 `**` | `Edit(src/**)` |
| `WebFetch(domain:網域)` | 可抓取的網站 | `WebFetch(domain:docs.python.org)` |
| `mcp__伺服器` 或 `mcp__伺服器__工具` | MCP 工具 | `mcp__github__create_issue` |
| `Agent(名稱)` | 某個子代理 | `Agent(Explore)` |
| `工具(參數:值)` | 依參數比對 | `Agent(model:opus)` |

路徑寫法：`./` 相對專案、`~/` 家目錄、`//` 絕對路徑。判斷順序：**deny 優先於 ask，ask 優先於 allow**。

> 💡 懶得自己寫？`/fewer-permission-prompts` 會分析你最近的紀錄，自動建議白名單。

## 9.5 沙盒（Sandbox）

沙盒由作業系統強制限制 Claude 執行的指令能碰到哪些檔案與網路。macOS 用 Seatbelt，Linux 與 WSL2 用 bubblewrap；原生 Windows 不支援。

開啟：輸入 `/sandbox`，或設定 `"sandbox": { "enabled": true }`。`/sandbox` 會先檢查相依套件：

![/sandbox 相依檢查](../images/35-sandbox.png)

（Linux 需要 `bubblewrap` 與 `socat`，畫面會告訴你怎麼裝。）

| 面向 | 預設 | 調整用的設定 |
| --- | --- | --- |
| 寫入 | 只能寫工作目錄與暫存資料夾 | `filesystem.allowWrite`／`denyWrite` |
| 讀取 | 大部分可讀（**包括 ~/.ssh**，建議封鎖） | `filesystem.denyRead`／`allowRead` |
| 網路 | 經本機 proxy，允許清單預設是空的 | `network.allowedDomains`／`deniedDomains` |

```json
{
  "sandbox": {
    "enabled": true,
    "filesystem": { "denyRead": ["~/.ssh", "~/.aws"] },
    "network": { "allowedDomains": ["github.com", "*.npmjs.org", "pypi.org"] },
    "excludedCommands": ["docker compose *"]
  }
}
```

沙盒內執行的指令會自動核准（不再一直問），因為作業系統已經擋住了危險範圍。注意沙盒只管 shell 指令，Claude 的讀寫檔工具、MCP、hooks 不在沙盒內。

## 9.6 推薦的安全組合

| 使用情境 | 建議 |
| --- | --- |
| 新手、重要專案 | Manual 或 Auto + deny 秘密檔案 |
| 日常開發 | Auto + 常用指令 allow + deny `.env` |
| 想幾乎零打擾 | 沙盒（auto-allow）+ 精確 allow 規則 |
| CI／腳本 | `dontAsk` + `--allowedTools` 白名單 |
| 完全放手 | 只在 Docker／VM 中用 `--dangerously-skip-permissions` |
