# 第 8 章　settings.json：所有設定項目逐一說明

> `/config` 面板只列出常用選項。**所有**設定都能寫在 `settings.json` 檔案裡。本章先教你設定檔的觀念，再把 v2.1.312 的全部設定 key（共 245 項）逐一說明並附範例。

## 8.1 設定檔在哪裡？

| 範圍 | 檔案位置 | 影響誰 | 適合放 |
| --- | --- | --- | --- |
| 使用者 | `~/.claude/settings.json`（Windows：`C:\Users\你\.claude\settings.json`） | 你在這台電腦的所有專案 | 個人偏好：主題、語言、預設模型 |
| 專案（共享） | 專案內 `.claude/settings.json` | 所有用這個專案的人（commit 進 git） | 團隊共用的權限、hooks、plugins |
| 專案（本機） | 專案內 `.claude/settings.local.json` | 只有你、只在這個專案（自動排除在 git 外） | 個人例外設定；權限提示選「不再詢問」時寫在這裡 |
| 組織管理 | IT 部署的 `managed-settings.json`、MDM 或後台下發 | 公司所有人 | 資安政策 |
| 全域 | `~/.claude.json` | 你 | Claude Code 自己維護（帳號、MCP、少數介面選項），一般不用手改 |

剛安裝時這些檔案都不存在，第一次在 `/config` 改設定或回答「不再詢問」時才會自動建立。也可以自己用編輯器建立。

## 8.2 優先順序：誰說了算？

同一個設定寫在多個檔案時，**上面的蓋過下面的**：

```
① 組織管理（最高，無法被覆蓋）
② 命令列 --settings（只影響這次）
③ 專案本機 .claude/settings.local.json
④ 專案共享 .claude/settings.json
⑤ 使用者 ~/.claude/settings.json（最低）
```

例外：清單類設定（例如 `permissions.allow`）會**合併**而不是覆蓋。

> 🔍 設定沒生效？在 session 中輸入 `/status`，看「Setting sources」列出實際載入了哪些檔案；在終端機執行 `claude doctor` 可列出寫錯而被忽略的設定。

## 8.3 設定檔的寫法

設定檔是**嚴格的 JSON**：用大括號包起來、key 用雙引號、項目之間用逗號，**最後一項後面不能有逗號，也不能寫註解**。

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "language": "繁體中文",
  "model": "opus",
  "outputStyle": "Explanatory",
  "permissions": {
    "allow": ["Bash(npm run test *)", "Bash(git diff *)"],
    "deny": ["Read(./.env)"]
  }
}
```

- 第一行 `$schema` 讓 VS Code 等編輯器提供自動完成與錯誤提示，強烈建議加上。
- 本章表格中的「範例」是**一個片段**，貼進大括號內即可；有多個項目時記得用逗號分隔。
- 存檔後大部分設定會**立即生效**（Claude Code 會監看設定檔）；`model` 與 effort 要在 session 中用 `/model`、`/effort` 才會即時切換。
- 懶得手改？在 session 中輸入 `/update-config 你想要的設定`，Claude 會幫你改對的檔案。

## 8.4 只想試一次？

```bash
claude --settings '{"model": "sonnet", "outputStyle": "Concise"}'
```

只影響這次啟動，不寫入任何檔案。

## 8.5 「範圍」欄位說明

下方表格的「範圍」表示這個 key 能寫在哪裡才有效：

| 標示 | 意思 |
| --- | --- |
| 任何設定檔 | 使用者、專案、本機、組織都可以 |
| 使用者／組織 | 只在 `~/.claude/settings.json`、`--settings` 或組織設定有效（寫在專案檔會被忽略，避免別人 commit 進來的設定影響你） |
| 使用者／本機／組織 | 同上，另外也接受 `.claude/settings.local.json` |
| 僅組織管理 | 只有 IT 部署的管理設定有效，一般使用者寫了也沒用 |
| ~/.claude.json | 全域設定檔中的選項（通常透過 `/config` 修改） |

標有「格式示意」的範例，表示該設定是較複雜的物件，範例展示用途與大致結構；正式使用前請對照[官方 All settings 文件](https://code.claude.com/docs/en/settings-reference)確認完整欄位。

## 8.6 全部設定項目


快速跳轉：[🧠 模型與推理](#cat-model)｜[🛡️ 權限](#cat-perm)｜[📦 沙盒（Sandbox）](#cat-sandbox)｜[📚 Context、壓縮與紀錄](#cat-context)｜[📝 記憶、指示與 Git](#cat-memory)｜[🧩 Hooks、Skills 與 Plugins](#cat-ext)｜[🔌 MCP 外部工具](#cat-mcp)｜[🚀 平行代理、工作流程與 Worktree](#cat-parallel)｜[🎨 介面與操作體驗](#cat-ui)｜[🔔 通知、遠端與桌面整合](#cat-remote)｜[🔑 環境、驗證與更新](#cat-env)｜[🏢 組織管理專用](#cat-managed)｜[🗄️ 已棄用（看到舊教學時參考）](#cat-deprecated)


<a id="cat-model"></a>

### 🧠 模型與推理

決定 Claude 用哪個模型、想得多深、回覆什麼語言。

| 設定 key | 範圍 | 說明 | 範例 |
| --- | --- | --- | --- |
| `model` | 任何設定檔 | 新 session 一開始使用的模型。可填別名（opus、sonnet、haiku、fable、best、opusplan）或完整模型 ID。 | `"model": "opus"` |
| `fallbackModel` | 任何設定檔 | 主要模型過載或無法使用時，依序改用的備援模型（最多三個）。 | `"fallbackModel": ["sonnet", "haiku"]` |
| `language` | 任何設定檔 | Claude 回覆與語音聽寫使用的語言，原樣傳給 Claude。想要繁中回覆就設這個。 | `"language": "繁體中文（台灣）"` |
| `effortLevel` | 任何設定檔 | 預設推理強度（還沒為某模型另存等級時使用）。可選 low、medium、high、xhigh。 | `"effortLevel": "high"` |
| `modelSettings` | 任何設定檔 | 依模型分別記住 effort 與自動壓縮門檻；/model、/effort 選擇時會自動寫入。 | `"modelSettings": { "claude-opus-5-5": { "effortLevel": "high" } }` |
| `maxEffortLevel` | 任何設定檔 | 推理強度上限，任何更高的選擇都會被壓到這裡（可全域或依模型）。適合控制成本。 | `"maxEffortLevel": "high"` |
| `alwaysThinkingEnabled` | 任何設定檔 | 設為 false 關閉延伸思考（Opus/Sonnet/Haiku 5.5 與 Fable 永遠開啟，設了也無效）。 | `"alwaysThinkingEnabled": false` |
| `showThinkingSummaries` | 任何設定檔 | 在對話中顯示 Claude 的思考摘要，而不是折疊起來。 | `"showThinkingSummaries": true` |
| `fastMode` | 任何設定檔 | 開啟 Fast mode（Opus 速度約 2.5 倍、單價較高，需 usage credits）。 | `"fastMode": true` |
| `fastModePerSessionOptIn` | 任何設定檔 | 每個新 session 都從 Fast mode 關閉開始，需要時再 /fast 開啟，避免意外花費。 | `"fastModePerSessionOptIn": true` |
| `advisorModel` | 任何設定檔 | 啟用「顧問工具」：關鍵時刻請另一個模型給建議，這裡填顧問模型。 | `"advisorModel": "opus"` |
| `ultracode` | 任何設定檔 | 讓 Claude 對每個實質任務都規劃多代理「動態工作流程」。很耗 token。 | `"ultracode": true` |
| `switchModelsOnFlag` | 任何設定檔 | 訊息被安全機制標記時：true 自動換模型繼續；false 暫停 session 讓你決定。 | `"switchModelsOnFlag": false` |
| `availableModels` | 任何設定檔 | 限制可選的模型清單（常由公司設定）。未設定代表全部可選。 | `"availableModels": ["sonnet", "haiku"]` |
| `availableModelsMatch` | 僅組織管理 | availableModels 的比對方式：prefix（預設，涵蓋衍生版本）或 exact。 | `"availableModelsMatch": "exact"` |
| `deniedModels` | 僅組織管理 | 封鎖特定模型，即使 availableModels 允許也不能選。 | `"deniedModels": ["claude-opus-5-5"]` |
| `enforceAvailableModels` | 任何設定檔 | 讓「Default」選項也必須落在 availableModels 之內。 | `"enforceAvailableModels": true` |
| `modelPicker` | 使用者／組織 | 自訂 /model 選單要列出哪些模型列、標籤與說明。 | `"modelPicker": { "options": [ { "model": "sonnet", "label": "日常用" } ] }` |
| `modelOverrides` | 任何設定檔 | 把 Anthropic 模型 ID 對應到雲端供應商的 ID（例如 Bedrock ARN）。 | `"modelOverrides": { "claude-opus-5-5": "arn:aws:bedrock:us-east-1:123456789012:inference-profile/example" }` |
| `modelPricing` | 僅組織管理 | 以公司合約價回報花費（給管理員）。 | `"modelPricing": { "claude-opus-5-5": { "input": 3.2, "output": 16 } }`<br><sub>格式示意</sub> |

<a id="cat-perm"></a>

### 🛡️ 權限

決定 Claude 哪些動作可以直接做、哪些要先問你、哪些永遠禁止。

| 設定 key | 範圍 | 說明 | 範例 |
| --- | --- | --- | --- |
| `permissions` | 任何設定檔 | 權限設定的總容器，包含下面各子項。 | `"permissions": { "allow": [], "ask": [], "deny": [] }` |
| `permissions.allow` | 任何設定檔 | 不用問就允許的工具使用規則。 | `"permissions": { "allow": ["Bash(npm run test *)", "Bash(git diff *)"] }` |
| `permissions.ask` | 任何設定檔 | 每次都要先問你的規則（即使其他地方允許）。 | `"permissions": { "ask": ["Bash(git push *)"] }` |
| `permissions.deny` | 任何設定檔 | 永遠禁止的規則，任何模式都有效，例如禁止讀取秘密檔案。 | `"permissions": { "deny": ["Read(./.env)", "Read(./secrets/**)"] }` |
| `permissions.defaultMode` | 任何設定檔 | 新 session 的起始權限模式：default(Manual)、acceptEdits、plan、auto、dontAsk、bypassPermissions。auto 與 bypass 不能從專案檔設定。 | `"permissions": { "defaultMode": "acceptEdits" }` |
| `permissions.additionalDirectories` | 任何設定檔 | 讓 Claude 也能存取目前資料夾以外的目錄。 | `"permissions": { "additionalDirectories": ["../shared-lib"] }` |
| `permissions.blockReadsOutsideWorkingDirectories` | 任何設定檔 | 在所有模式中都禁止讀取工作目錄以外的檔案。 | `"permissions": { "blockReadsOutsideWorkingDirectories": true }` |
| `permissions.disableBypassPermissionsMode` | 任何設定檔 | 禁止進入 bypassPermissions（完全不問）模式。 | `"permissions": { "disableBypassPermissionsMode": "disable" }` |
| `disableAutoMode` | 任何設定檔 | 把 Auto mode 從 Shift+Tab 循環中移除。 | `"disableAutoMode": "disable"` |
| `autoMode` | 使用者／組織 | 為 Auto mode 分類器加入你自己的允許／封鎖規則與環境描述（可用 /auto-mode-setup 產生）。 | `"autoMode": { "environment": ["$defaults", "staging.example.com 是測試機，可以部署"] }` |
| `autoMode.classifyAllShell` | 使用者／組織 | 所有 shell 指令都交給分類器審查（包含原本白名單內的）。 | `"autoMode": { "classifyAllShell": true }` |
| `useAutoModeDuringPlan` | 使用者／本機／組織 | Plan 模式下的 shell 指令也讓分類器自動審查；false 則改為詢問你。 | `"useAutoModeDuringPlan": false` |
| `skipAutoPermissionPrompt` | 使用者／組織 | 跳過第一次啟用 Auto mode 時的說明提示。 | `"skipAutoPermissionPrompt": true` |
| `skipDangerousModePermissionPrompt` | 使用者／本機／組織 | 跳過進入 bypassPermissions 時的確認對話框（不建議）。 | `"skipDangerousModePermissionPrompt": true` |
| `allowManagedPermissionRulesOnly` | 僅組織管理 | 只採用組織管理設定中的權限規則，忽略個人與專案規則。 | `"allowManagedPermissionRulesOnly": true` |

<a id="cat-sandbox"></a>

### 📦 沙盒（Sandbox）

用作業系統層級限制 Claude 執行的 shell 指令能碰到的檔案與網路。

| 設定 key | 範圍 | 說明 | 範例 |
| --- | --- | --- | --- |
| `sandbox` | 任何設定檔 | 沙盒設定的總容器（macOS、Linux、WSL2）。 | `"sandbox": { "enabled": true }` |
| `sandbox.enabled` | 任何設定檔 | 開啟 Bash 沙盒。也可在 session 中用 /sandbox 切換。 | `"sandbox": { "enabled": true }` |
| `sandbox.autoAllowBashIfSandboxed` | 任何設定檔 | 在沙盒中執行的指令自動核准、不跳提示。 | `"sandbox": { "autoAllowBashIfSandboxed": true }` |
| `sandbox.allowUnsandboxedCommands` | 任何設定檔 | 被沙盒擋下時，是否允許 Claude 改在沙盒外重試（仍需你核准）。 | `"sandbox": { "allowUnsandboxedCommands": false }` |
| `sandbox.excludedCommands` | 任何設定檔 | 這些指令一律在沙盒外執行（走一般權限流程）。 | `"sandbox": { "excludedCommands": ["docker compose *"] }` |
| `sandbox.failIfUnavailable` | 任何設定檔 | 沙盒無法啟動時直接拒絕開始，而不是不受保護地執行。 | `"sandbox": { "failIfUnavailable": true }` |
| `sandbox.filesystem` | 任何設定檔 | 檔案系統規則的容器。 | `"sandbox": { "filesystem": { "allowWrite": ["/tmp/build"] } }` |
| `sandbox.filesystem.allowWrite` | 任何設定檔 | 額外允許寫入的路徑。 | `"sandbox": { "filesystem": { "allowWrite": ["~/.kube", "/tmp/build"] } }` |
| `sandbox.filesystem.denyWrite` | 任何設定檔 | 禁止寫入的路徑。 | `"sandbox": { "filesystem": { "denyWrite": ["./config/prod"] } }` |
| `sandbox.filesystem.denyRead` | 任何設定檔 | 禁止讀取的路徑（預設沙盒可讀大部分檔案，包括 ~/.ssh）。 | `"sandbox": { "filesystem": { "denyRead": ["~/.ssh", "~/.aws"] } }` |
| `sandbox.filesystem.allowRead` | 任何設定檔 | 在 denyRead 範圍內重新開放可讀的子路徑。 | `"sandbox": { "filesystem": { "denyRead": ["~/"], "allowRead": ["."] } }` |
| `sandbox.filesystem.disabled` | 使用者／組織 | 關閉檔案隔離、只保留網路隔離。 | `"sandbox": { "filesystem": { "disabled": true } }` |
| `sandbox.filesystem.allowManagedReadPathsOnly` | 僅組織管理 | 禁止開發者自行重新開放被組織封鎖的讀取路徑。 | `"sandbox": { "filesystem": { "allowManagedReadPathsOnly": true } }` |
| `sandbox.network` | 任何設定檔 | 網路規則的容器。 | `"sandbox": { "network": { "allowedDomains": ["github.com"] } }` |
| `sandbox.network.allowedDomains` | 任何設定檔 | 預先允許、不用詢問就能連線的網域（支援 *.）。 | `"sandbox": { "network": { "allowedDomains": ["github.com", "*.npmjs.org"] } }` |
| `sandbox.network.deniedDomains` | 任何設定檔 | 封鎖的網域（即使符合允許的萬用字元也擋）。 | `"sandbox": { "network": { "deniedDomains": ["evil.example.com"] } }` |
| `sandbox.network.strictAllowlist` | 使用者／組織 | 不在允許清單的網域直接拒絕，而不是詢問你。 | `"sandbox": { "network": { "strictAllowlist": true } }` |
| `sandbox.network.allowLocalBinding` | 任何設定檔 | 允許沙盒內指令在 macOS 上監聽本機埠（例如啟動 dev server）。 | `"sandbox": { "network": { "allowLocalBinding": true } }` |
| `sandbox.network.allowUnixSockets` | 任何設定檔 | macOS 上允許連線的 Unix socket 路徑。 | `"sandbox": { "network": { "allowUnixSockets": ["/var/run/docker.sock"] } }` |
| `sandbox.network.allowAllUnixSockets` | 任何設定檔 | 允許連線所有 Unix socket。 | `"sandbox": { "network": { "allowAllUnixSockets": true } }` |
| `sandbox.network.allowMachLookup` | 任何設定檔 | 允許 macOS 工具連到 XPC 服務。 | `"sandbox": { "network": { "allowMachLookup": ["com.apple.coresimulator.*"] } }`<br><sub>格式示意</sub> |
| `sandbox.network.httpProxyPort` | 任何設定檔 | 改用你自己的 HTTP proxy。 | `"sandbox": { "network": { "httpProxyPort": 8080 } }` |
| `sandbox.network.socksProxyPort` | 任何設定檔 | 改用你自己的 SOCKS proxy。 | `"sandbox": { "network": { "socksProxyPort": 1080 } }` |
| `sandbox.network.tlsTerminate` | 使用者／組織 | 讓沙盒 proxy 解開 TLS 以檢查 HTTPS 請求內容。 | `"sandbox": { "network": { "tlsTerminate": true } }` |
| `sandbox.network.allowManagedDomainsOnly` | 僅組織管理 | 網路允許清單只能由組織設定。 | `"sandbox": { "network": { "allowManagedDomainsOnly": true } }` |
| `sandbox.credentials` | 任何設定檔 | 在沙盒中隱藏或遮罩憑證（環境變數、檔案）。 | `"sandbox": { "credentials": { "envVars": { "GITHUB_TOKEN": "mask" } } }`<br><sub>格式示意</sub> |
| `sandbox.credentials.envVars` | 任何設定檔 | 沙盒中要移除或遮罩的環境變數。 | `"sandbox": { "credentials": { "envVars": { "AWS_SECRET_ACCESS_KEY": "unset" } } }`<br><sub>格式示意</sub> |
| `sandbox.credentials.files` | 任何設定檔 | 封鎖或遮罩憑證檔的讀取。 | `"sandbox": { "credentials": { "files": { "~/.npmrc": "block" } } }`<br><sub>格式示意</sub> |
| `sandbox.credentials.allowPlaintextInject` | 使用者／組織 | 在信任的測試網路上，允許遮罩的憑證送到純 HTTP。 | `"sandbox": { "credentials": { "allowPlaintextInject": true } }` |
| `sandbox.credentials.awsPairs` | 使用者／組織 | 把 AWS 金鑰變數配對，讓 proxy 能重新簽章。 | `"sandbox": { "credentials": { "awsPairs": [["AWS_ACCESS_KEY_ID", "AWS_SECRET_ACCESS_KEY"]] } }`<br><sub>格式示意</sub> |
| `sandbox.credentials.sigv4` | 使用者／組織 | 遇到串流、預簽章或 SigV4A 請求時是失敗或放行。 | `"sandbox": { "credentials": { "sigv4": "fail" } }`<br><sub>格式示意</sub> |
| `sandbox.allowAppleEvents` | 使用者／組織 | 允許沙盒指令在 macOS 送出 Apple Events（控制其他 App）。 | `"sandbox": { "allowAppleEvents": true }` |
| `sandbox.enableWeakerNestedSandbox` | 任何設定檔 | 在無特權容器內執行較弱的 Linux 沙盒。 | `"sandbox": { "enableWeakerNestedSandbox": true }` |
| `sandbox.enableWeakerNetworkIsolation` | 任何設定檔 | macOS 上讓 gh、gcloud、terraform 在 MITM proxy 後仍能驗證 TLS。 | `"sandbox": { "enableWeakerNetworkIsolation": true }` |
| `sandbox.allowNestedUserNamespaces` | 任何設定檔 | 允許沙盒內建立 user namespace（Chromium、rootless podman 需要）；可能削弱檔案保護。 | `"sandbox": { "allowNestedUserNamespaces": true }` |
| `sandbox.ignoreViolations` | 任何設定檔 | 對預期中的探測不顯示違規報告。 | `"sandbox": { "ignoreViolations": { "*": ["/usr/local/share"] } }`<br><sub>格式示意</sub> |
| `sandbox.ripgrep` | 使用者／組織 | 在沙盒中使用你自己的 ripgrep 執行檔。 | `"sandbox": { "ripgrep": "/opt/bin/rg" }`<br><sub>格式示意</sub> |
| `sandbox.bwrapPath` | 僅組織管理 | bubblewrap 不在 PATH 時指定路徑。 | `"sandbox": { "bwrapPath": "/opt/bin/bwrap" }` |
| `sandbox.socatPath` | 僅組織管理 | socat 不在 PATH 時指定路徑。 | `"sandbox": { "socatPath": "/opt/bin/socat" }` |

<a id="cat-context"></a>

### 📚 Context、壓縮與紀錄

控制對話記憶空間怎麼用、何時摘要、保存多久。

| 設定 key | 範圍 | 說明 | 範例 |
| --- | --- | --- | --- |
| `autoCompactEnabled` | 任何設定檔 | context 快滿時自動摘要壓縮對話（建議保持開啟）。 | `"autoCompactEnabled": true` |
| `autoCompactWindow` | 任何設定檔 | context 用到多少 token 時觸發自動壓縮（10 萬～100 萬）。 | `"autoCompactWindow": 500000` |
| `idleCompaction` | 任何設定檔 | 設為 false 時，閒置的長對話不會在快取過期前先壓縮。 | `"idleCompaction": false` |
| `promptCacheTtl` | 任何設定檔 | 主對話的提示快取保存時間：5m 或 1h（訂閱預設 1h）。 | `"promptCacheTtl": "1h"` |
| `subagentPromptCacheTtl` | 任何設定檔 | subagent 與工作流程代理的快取保存時間。 | `"subagentPromptCacheTtl": "1h"` |
| `bashOutputMaxChars` | 任何設定檔 | 指令成功輸出時，直接給 Claude 看的最大字元數（預設 30000），超過的存成檔案。 | `"bashOutputMaxChars": 60000` |
| `skillListingBudgetFraction` | 任何設定檔 | 保留多少比例的 context 放 skills 清單（預設 0.01 = 1%）。 | `"skillListingBudgetFraction": 0.02` |
| `skillListingMaxDescChars` | 任何設定檔 | 每個 skill 描述在清單中的最大字元數（預設 1536）。 | `"skillListingMaxDescChars": 800` |
| `fileCheckpointingEnabled` | 任何設定檔 | 編輯前先快照檔案，讓 /rewind 能還原程式碼。 | `"fileCheckpointingEnabled": true` |
| `bashEditDiffEnabled` | 使用者／組織 | 記錄 Bash 指令改動了哪些檔案並顯示差異。 | `"bashEditDiffEnabled": false` |
| `cleanupPeriodDays` | 任何設定檔 | 對話紀錄保存天數（預設 30，最少 1）。 | `"cleanupPeriodDays": 90` |
| `desktopSessionCleanupPeriodDays` | 使用者／組織 | 桌面版產生的對話紀錄保存上限天數（0 = 不限）。 | `"desktopSessionCleanupPeriodDays": 60` |
| `plansDirectory` | 任何設定檔 | Plan 模式寫出計畫檔的位置（相對專案根目錄，預設 ~/.claude/plans/）。 | `"plansDirectory": "docs/plans"` |

<a id="cat-memory"></a>

### 📝 記憶、指示與 Git

控制 Claude 會讀到哪些指示、自動記住什麼、commit 怎麼署名。

| 設定 key | 範圍 | 說明 | 範例 |
| --- | --- | --- | --- |
| `autoMemoryEnabled` | 任何設定檔 | 開關自動記憶（Claude 自己記下你的偏好與專案脈絡）。 | `"autoMemoryEnabled": false` |
| `autoMemoryDirectory` | 任何設定檔 | 自動記憶的存放資料夾（需絕對路徑或 ~/ 開頭）。 | `"autoMemoryDirectory": "~/my-claude-memory"` |
| `claudeMdExcludes` | 任何設定檔 | 不要載入的 CLAUDE.md（路徑或萬用字元），適合大型 monorepo。 | `"claudeMdExcludes": ["**/legacy/**/CLAUDE.md"]` |
| `claudeMd` | 僅組織管理 | 組織統一注入的 CLAUDE.md 內容。 | `"claudeMd": "所有程式碼註解使用英文。"` |
| `includeGitInstructions` | 任何設定檔 | 是否在系統提示中包含內建的 commit／PR 流程指示。 | `"includeGitInstructions": false` |
| `attribution` | 任何設定檔 | 自訂 commit 與 PR 的署名文字（設為空字串可隱藏）。 | `"attribution": { "commit": "", "pr": "" }` |
| `attribution.commit` | 任何設定檔 | commit 結尾的署名（trailer）。 | `"attribution": { "commit": "Assisted-by: Claude" }` |
| `attribution.pr` | 任何設定檔 | PR 描述中的署名文字。 | `"attribution": { "pr": "🤖 由 Claude Code 協助產生" }` |
| `attribution.sessionUrl` | 任何設定檔 | 雲端／遠端 session 的 commit 不附 session 連結。 | `"attribution": { "sessionUrl": false }` |
| `outputStyle` | 任何設定檔 | 回覆風格：Default、Proactive、Concise、Explanatory、Learning 或自訂（大小寫要對）。 | `"outputStyle": "Explanatory"` |
| `agent` | 任何設定檔 | 每個 session 都以某個 subagent 身分當主代理啟動。 | `"agent": "code-reviewer"` |

<a id="cat-ext"></a>

### 🧩 Hooks、Skills 與 Plugins

擴充功能的開關與管控。

| 設定 key | 範圍 | 說明 | 範例 |
| --- | --- | --- | --- |
| `hooks` | 任何設定檔 | 在生命週期事件（工具前後、送出提示、結束…）自動執行指令。 | `"hooks": { "PostToolUse": [ { "matcher": "Edit\|Write", "hooks": [ { "type": "command", "command": "npx prettier --write ." } ] } ] }` |
| `disableAllHooks` | 任何設定檔 | 停用所有 hooks、狀態列指令與 @ 檔案建議指令（含使用者安裝的 mods）。 | `"disableAllHooks": true` |
| `allowManagedHooksOnly` | 僅組織管理 | 只執行組織部署的 hooks。 | `"allowManagedHooksOnly": true` |
| `allowedHttpHookUrls` | 任何設定檔 | HTTP hooks 只能送到這些網址（支援 *）。 | `"allowedHttpHookUrls": ["https://hooks.example.com/*"]` |
| `httpHookAllowedEnvVars` | 任何設定檔 | HTTP hooks 可放進 header 的環境變數白名單。 | `"httpHookAllowedEnvVars": ["HOOK_TOKEN"]` |
| `statusLine` | 任何設定檔 | 在提示框下方顯示自訂狀態列（執行你的指令，stdin 收到 session JSON）。 | `"statusLine": { "type": "command", "command": "~/.claude/statusline.sh" }` |
| `subagentStatusLine` | 任何設定檔 | 自訂 subagent 面板每一列的顯示。 | `"subagentStatusLine": { "type": "command", "command": "~/.claude/agent-row.sh" }`<br><sub>格式示意</sub> |
| `fileSuggestion` | 任何設定檔 | 用你自己的指令提供 @ 檔案自動完成清單。 | `"fileSuggestion": { "type": "command", "command": "~/.claude/list-files.sh" }` |
| `respectGitignore` | 任何設定檔 | @ 檔案選單是否排除 .gitignore 中的檔案（預設 true）。 | `"respectGitignore": false` |
| `disableSkillShellExecution` | 任何設定檔 | 禁止 skills／自訂指令中的 !`指令` 內嵌執行。 | `"disableSkillShellExecution": true` |
| `disableBundledSkills` | 任何設定檔 | 停用內建 skills 與 workflows（/code-review、/batch…）。 | `"disableBundledSkills": true` |
| `skillOverrides` | 任何設定檔 | 不改 SKILL.md 就隱藏或縮減某個 skill：on、name-only、user-invocable-only、off。 | `"skillOverrides": { "dataviz": "off" }` |
| `syncClaudeAiSkills` | 使用者／本機／組織 | 設 false 停止載入你在 claude.ai 開啟的 skills。 | `"syncClaudeAiSkills": false` |
| `syncClaudeAiPlugins` | 使用者／本機／組織 | 設 false 停止載入你在 claude.ai 開啟的 plugins。 | `"syncClaudeAiPlugins": false` |
| `enabledPlugins` | 任何設定檔 | 依範圍開關個別 plugin（格式：名稱@市集）。 | `"enabledPlugins": { "code-review@claude-plugins-official": true }` |
| `extraKnownMarketplaces` | 任何設定檔 | 為此 repo 或組織註冊額外的 plugin 市集。 | `"extraKnownMarketplaces": { "acme": { "source": { "source": "github", "repo": "acme/claude-plugins" } } }`<br><sub>格式示意</sub> |
| `pluginConfigs` | 使用者／組織 | plugin 設定對話框的已存答案（也用來設定內建 AGENTS.md 讀取方式）。 | `"pluginConfigs": { "cc-plugin-agents-md@builtin": { "options": { "instructionFiles": "claude-md-and-agents-md" } } }` |
| `prependPlugins` | 使用者／組織 | 組織的 mods 在每個使用者安裝的 mod 之前執行。 | `"prependPlugins": ["guard@acme"]`<br><sub>格式示意</sub> |
| `appendPlugins` | 使用者／組織 | 組織的 mods 在每個使用者安裝的 mod 之後執行。 | `"appendPlugins": ["audit@acme"]`<br><sub>格式示意</sub> |
| `strictKnownMarketplaces` | 僅組織管理 | 只允許這些市集來源。 | `"strictKnownMarketplaces": [ { "source": "github", "repo": "acme/claude-plugins" } ]`<br><sub>格式示意</sub> |
| `blockedMarketplaces` | 僅組織管理 | 封鎖特定市集來源。 | `"blockedMarketplaces": [ { "source": "github", "repo": "someone/untrusted" } ]`<br><sub>格式示意</sub> |
| `disableCommandPluginSources` | 僅組織管理 | 封鎖「執行指令來安裝」的 plugin 來源。 | `"disableCommandPluginSources": true` |
| `pluginSuggestionMarketplaces` | 僅組織管理 | 哪些市集的 plugin 可以出現在情境推薦中。 | `"pluginSuggestionMarketplaces": ["acme"]` |
| `pluginTrustMessage` | 僅組織管理 | 在 plugin 信任警告中附加公司自訂說明。 | `"pluginTrustMessage": "請只安裝 IT 審核過的 plugin。"` |
| `strictPluginOnlyCustomization` | 僅組織管理 | skills、agents、hooks、MCP 只能來自 plugin 與組織，擋掉個人／專案來源。 | `"strictPluginOnlyCustomization": true` |
| `strictPluginOnlyCustomization.agents` | 僅組織管理 | 只鎖 subagents。 | `"strictPluginOnlyCustomization": { "agents": true }` |
| `strictPluginOnlyCustomization.hooks` | 僅組織管理 | 只鎖 hooks。 | `"strictPluginOnlyCustomization": { "hooks": true }` |
| `strictPluginOnlyCustomization.mcp` | 僅組織管理 | 只鎖 MCP 伺服器。 | `"strictPluginOnlyCustomization": { "mcp": true }` |
| `strictPluginOnlyCustomization.skills` | 僅組織管理 | 只鎖 skills。 | `"strictPluginOnlyCustomization": { "skills": true }` |
| `disableSideloadFlags` | 僅組織管理 | 拒絕 --plugin-dir、--agents、--mcp-config 等側載旗標。 | `"disableSideloadFlags": true` |

<a id="cat-mcp"></a>

### 🔌 MCP 外部工具

管理 Claude 能連到哪些外部工具伺服器。

| 設定 key | 範圍 | 說明 | 範例 |
| --- | --- | --- | --- |
| `enableAllProjectMcpServers` | 任何設定檔 | 自動核准專案 .mcp.json 中所有伺服器，不再逐一詢問。 | `"enableAllProjectMcpServers": true` |
| `enabledMcpjsonServers` | 任何設定檔 | 核准專案 .mcp.json 中指定的伺服器。 | `"enabledMcpjsonServers": ["github", "sentry"]` |
| `disabledMcpjsonServers` | 任何設定檔 | 拒絕專案 .mcp.json 中指定的伺服器。 | `"disabledMcpjsonServers": ["experimental-db"]` |
| `allowedMcpServers` | 任何設定檔 | 允許使用者加入的 MCP 伺服器白名單。 | `"allowedMcpServers": [ { "serverName": "github" } ]`<br><sub>格式示意</sub> |
| `deniedMcpServers` | 任何設定檔 | 依名稱、網址或指令封鎖 MCP 伺服器（優先於白名單）。 | `"deniedMcpServers": [ { "serverName": "filesystem" } ]`<br><sub>格式示意</sub> |
| `allowManagedMcpServersOnly` | 僅組織管理 | 白名單只採用組織設定。 | `"allowManagedMcpServersOnly": true` |
| `managedMcpServers` | 僅組織管理 | 組織提供給每位使用者的遠端 MCP 伺服器。 | `"managedMcpServers": { "jira": { "type": "http", "url": "https://mcp.acme.com/jira" } }`<br><sub>格式示意</sub> |
| `allowAllClaudeAiMcps` | 僅組織管理 | 在 managed-mcp.json 鎖定下仍載入 claude.ai connectors。 | `"allowAllClaudeAiMcps": true` |
| `disableClaudeAiConnectors` | 任何設定檔 | 停用 claude.ai 上連好的 connectors。 | `"disableClaudeAiConnectors": true` |
| `allowClaudeInChromeWithManagedMcp` | 僅組織管理 | 在 managed-mcp.json 鎖定下仍允許 Claude in Chrome。 | `"allowClaudeInChromeWithManagedMcp": true` |
| `channelsEnabled` | 僅組織管理 | 允許組織使用 channels（MCP 推送事件進 session）。 | `"channelsEnabled": true` |
| `allowedChannelPlugins` | 僅組織管理 | 取代預設的 channel plugin 白名單。 | `"allowedChannelPlugins": ["ci-notifier@acme"]`<br><sub>格式示意</sub> |

<a id="cat-parallel"></a>

### 🚀 平行代理、工作流程與 Worktree

多個 Claude 同時工作時的行為。

| 設定 key | 範圍 | 說明 | 範例 |
| --- | --- | --- | --- |
| `disableAgentView` | 任何設定檔 | 關閉背景代理與 Agent view（claude agents、--bg、/background）。 | `"disableAgentView": true` |
| `defaultToAgentsView` | ~/.claude.json | 只打 claude 不帶參數時，直接開啟 Agent view。 | `"defaultToAgentsView": true` |
| `leftArrowOpensAgents` | ~/.claude.json | 關閉「空白輸入按 ← 開啟 Agent view」的快捷鍵。 | `"leftArrowOpensAgents": false` |
| `crossSessionInbound` | 任何設定檔 | 其他 session 傳來的訊息：accept 直接送達、hold 暫存待你審核、refuse 拒收。 | `"crossSessionInbound": "hold"` |
| `isolatePeerMachines` | 任何設定檔 | 要傳訊息給其他電腦上的 session 前先問你。 | `"isolatePeerMachines": true` |
| `teammateMode` | 任何設定檔 | Agent team 隊友的顯示方式：auto、tmux、iterm2、in-process。 | `"teammateMode": "tmux"` |
| `enableWorkflows` | 任何設定檔 | 為自己開關動態工作流程（Pro 方案需開啟）。 | `"enableWorkflows": true` |
| `disableWorkflows` | 任何設定檔 | 對所有人關閉動態工作流程。 | `"disableWorkflows": true` |
| `workflowKeywordTriggerEnabled` | 任何設定檔 | 設 false 讓提示中的 ultracode 字樣不再觸發工作流程。 | `"workflowKeywordTriggerEnabled": false` |
| `workflowSizeGuideline` | 任何設定檔 | 工作流程代理數建議：small(<5)、medium(<10)、large(<50)、unrestricted。 | `"workflowSizeGuideline": "small"` |
| `worktree` | 任何設定檔 | git worktree 的建立方式（容器）。 | `"worktree": { "baseRef": "head" }` |
| `worktree.baseRef` | 任何設定檔 | 新 worktree 從遠端預設分支（fresh）或本地 HEAD（head）分出。 | `"worktree": { "baseRef": "head" }` |
| `worktree.bgIsolation` | 任何設定檔 | 設 none 讓背景 session 直接改工作目錄，不建 worktree。 | `"worktree": { "bgIsolation": "none" }` |
| `worktree.sparsePaths` | 任何設定檔 | 每個 worktree 只 checkout 需要的目錄（大型 repo 加速）。 | `"worktree": { "sparsePaths": ["packages/web", "packages/shared"] }` |
| `worktree.symlinkDirectories` | 任何設定檔 | 把大型目錄用 symlink 共用到每個 worktree，省空間。 | `"worktree": { "symlinkDirectories": ["node_modules", ".cache"] }` |
| `processWrapper` | 使用者／組織 | （macOS/Linux）背景程序透過公司指定的啟動器執行。 | `"processWrapper": "/usr/local/bin/corp-launch"` |

<a id="cat-ui"></a>

### 🎨 介面與操作體驗

外觀、輸入方式、顯示內容。

| 設定 key | 範圍 | 說明 | 範例 |
| --- | --- | --- | --- |
| `theme` | 任何設定檔 | 介面配色：dark、light、auto、色盲友善、ANSI 等。 | `"theme": "dark"` |
| `tui` | 任何設定檔 | 終端機渲染器：default（傳統）或 fullscreen（無閃爍全螢幕）。 | `"tui": "fullscreen"` |
| `viewMode` | 任何設定檔 | 啟動時的對話檢視：default、verbose（完整工具輸出）、focus（只看重點）。 | `"viewMode": "focus"` |
| `verbose` | 任何設定檔 | 顯示完整工具輸出（viewMode 同時設定時以 viewMode 為準）。 | `"verbose": true` |
| `editorMode` | 任何設定檔 | 輸入框按鍵模式：normal 或 vim。 | `"editorMode": "vim"` |
| `vimInsertModeRemaps` | 使用者／組織 | Vim 插入模式中把兩個按鍵序列對應成 Esc。 | `"vimInsertModeRemaps": { "jj": "<Esc>" }` |
| `defaultShell` | 任何設定檔 | 輸入框 ! 指令使用 bash 或 powershell。 | `"defaultShell": "powershell"` |
| `respondToBashCommands` | 任何設定檔 | 設 false 時，! 指令的輸出只加入對話、Claude 不自動回應。 | `"respondToBashCommands": false` |
| `promptSuggestionEnabled` | 任何設定檔 | 設 false 隱藏灰色的「下一句建議」。 | `"promptSuggestionEnabled": false` |
| `emojiCompletionEnabled` | 任何設定檔 | 設 false 關閉 :shortcode: emoji 建議。 | `"emojiCompletionEnabled": false` |
| `spellcheck` | 使用者／組織 | 用已安裝的拼字檢查器在輸入框標示拼錯的字。 | `"spellcheck": { "enabled": true }`<br><sub>格式示意</sub> |
| `spinnerTipsEnabled` | 任何設定檔 | 工作中 spinner 下方是否顯示小提示。 | `"spinnerTipsEnabled": false` |
| `spinnerTipsOverride` | 任何設定檔 | 加入或取代 spinner 提示文字。 | `"spinnerTipsOverride": { "excludeDefault": true, "tips": ["記得先跑測試再 commit"] }` |
| `spinnerVerbs` | 任何設定檔 | 加入或取代工作中顯示的動詞（例如 Thinking、Baking）。 | `"spinnerVerbs": { "mode": "append", "verbs": ["思考中", "敲鍵盤中"] }` |
| `showTurnDuration` | 任何設定檔 | 回合結束後顯示「Cooked for 1m 20s」耗時。 | `"showTurnDuration": false` |
| `terminalProgressBarEnabled` | 任何設定檔 | 在支援的終端機分頁顯示進度條。 | `"terminalProgressBarEnabled": false` |
| `terminalTitleFromRename` | 任何設定檔 | 設 false 讓 /rename 不改終端機分頁標題。 | `"terminalTitleFromRename": false` |
| `timeFormat` | 任何設定檔 | 時間顯示：auto、12-hour、24-hour、24-hour-utc 或 strftime 樣式。 | `"timeFormat": "24-hour"` |
| `timeZone` | 任何設定檔 | 介面顯示時間用的時區（IANA 名稱）。 | `"timeZone": "Asia/Taipei"` |
| `prefersReducedMotion` | 任何設定檔 | 減少或關閉動畫（無障礙）。 | `"prefersReducedMotion": true` |
| `syntaxHighlightingDisabled` | 任何設定檔 | 關閉 diff 與程式碼區塊的語法上色。 | `"syntaxHighlightingDisabled": true` |
| `maxProseWidth` | 任何設定檔 | 寬終端機中，說明文字最多幾欄寬（表格與程式碼不受限）。 | `"maxProseWidth": 100` |
| `autoScrollEnabled` | 任何設定檔 | 全螢幕模式下自動捲到最新輸出。 | `"autoScrollEnabled": false` |
| `wheelScrollAccelerationEnabled` | 任何設定檔 | 全螢幕模式下滑鼠滾輪加速。 | `"wheelScrollAccelerationEnabled": false` |
| `awaySummaryEnabled` | 任何設定檔 | 離開一陣子回來時是否顯示「期間發生了什麼」摘要。 | `"awaySummaryEnabled": false` |
| `showClearContextOnPlanAccept` | 任何設定檔 | 核准計畫時多出「清空 context 再執行」選項。 | `"showClearContextOnPlanAccept": true` |
| `axScreenReader` | 任何設定檔 | 螢幕閱讀器友善輸出（無邊框、無動畫的線性文字）。 | `"axScreenReader": true` |
| `voice` | 任何設定檔 | 語音聽寫：開啟並選擇 hold（按住說話）或 tap（點一下開始/結束）。 | `"voice": { "enabled": true, "mode": "tap" }` |
| `voiceEnabled` | 任何設定檔 | 舊版單一開關的語音聽寫設定。 | `"voiceEnabled": true` |
| `copyFullResponse` | ~/.claude.json | /copy 直接複製整段回覆，不跳選擇器。 | `"copyFullResponse": true` |
| `copyOnSelect` | ~/.claude.json | 設 false 關閉「滑鼠選取即複製」。 | `"copyOnSelect": false` |
| `externalEditorContext` | ~/.claude.json | Ctrl+G 開外部編輯器時，附上上一則回覆（以註解形式）。 | `"externalEditorContext": true` |
| `diffTool` | ~/.claude.json | 檔案差異在哪裡顯示：auto（有連 IDE 時用 IDE）或 terminal（一律終端機）。 | `"diffTool": "terminal"` |
| `prStatusFooterEnabled` | ~/.claude.json | 設 false 隱藏底部的 PR 審查狀態徽章。 | `"prStatusFooterEnabled": false` |
| `prUrlTemplate` | 任何設定檔 | PR 連結改指向內部審查工具（可用 {owner}{repo}{number} 等變數）。 | `"prUrlTemplate": "https://reviews.example.com/{owner}/{repo}/pull/{number}"` |
| `footerLinksRegexes` | 使用者／組織 | 輸出符合正規表示式時，在輸入框下方顯示可點的連結徽章（例如 issue 編號）。 | `"footerLinksRegexes": [ { "regex": "JIRA-(\\d+)", "url": "https://jira.example.com/browse/JIRA-$1" } ]`<br><sub>格式示意</sub> |
| `companyAnnouncements` | 任何設定檔 | 啟動時顯示公司公告（多則時隨機一則）。 | `"companyAnnouncements": ["本週五 18:00 停機維護"]` |
| `feedbackSurveyRate` | 任何設定檔 | 品質問卷出現機率（0～1）。 | `"feedbackSurveyRate": 0` |
| `feedbackDrafts` | 使用者／組織 | Claude 代擬回饋報告的通知方式：notify、quiet、off。 | `"feedbackDrafts": "off"` |

<a id="cat-remote"></a>

### 🔔 通知、遠端與桌面整合

通知、手機、遠端控制、IDE 與瀏覽器整合。

| 設定 key | 範圍 | 說明 | 範例 |
| --- | --- | --- | --- |
| `preferredNotifChannel` | 任何設定檔 | 任務完成時的通知方式：auto、terminal_bell、iterm2、kitty、ghostty、notifications_disabled 等。 | `"preferredNotifChannel": "terminal_bell"` |
| `agentPushNotifEnabled` | 任何設定檔 | 允許 Claude 主動推播通知到手機。 | `"agentPushNotifEnabled": true` |
| `inputNeededNotifEnabled` | 任何設定檔 | Claude 在等你（權限或問題）時推播到手機。 | `"inputNeededNotifEnabled": true` |
| `askUserQuestionTimeout` | 使用者／組織 | Claude 問你問題時，閒置多久自動用已選答案繼續：60s、5m、10m、never。 | `"askUserQuestionTimeout": "5m"` |
| `dialogExpiry` | 使用者／組織 | 轉發到遠端裝置的對話框等多久沒回應就取消：60s、5m、10m、never。 | `"dialogExpiry": "10m"` |
| `autoContinueAtUsageLimit` | 使用者／組織 | 碰到 claude.ai 額度上限時，等重置後自動繼續任務。 | `"autoContinueAtUsageLimit": true` |
| `remoteControlAtStartup` | 任何設定檔 | 每個 session 啟動就自動開啟 Remote Control。 | `"remoteControlAtStartup": true` |
| `disableRemoteControl` | 任何設定檔 | 全面停用 Remote Control。 | `"disableRemoteControl": true` |
| `remote.defaultEnvironmentId` | 任何設定檔 | claude --cloud 預設使用的雲端環境。 | `"remote": { "defaultEnvironmentId": "env_abc123" }` |
| `enableArtifact` | 任何設定檔 | 設 false 關閉 Artifact（可分享網頁成品）工具。 | `"enableArtifact": false` |
| `disableDeepLinkRegistration` | 任何設定檔 | 不註冊 claude-cli:// 連結處理程式。 | `"disableDeepLinkRegistration": true` |
| `claudeInChromeDefaultEnabled` | ~/.claude.json | 每個 session 預設啟用 Chrome 整合。 | `"claudeInChromeDefaultEnabled": true` |
| `autoConnectIde` | ~/.claude.json | 在外部終端機啟動時自動連到執行中的 VS Code / JetBrains。 | `"autoConnectIde": true` |
| `autoInstallIdeExtension` | ~/.claude.json | 設 false 停止從 VS Code 終端機自動安裝擴充。 | `"autoInstallIdeExtension": false` |
| `sshConfigs` | 使用者／組織 | 在桌面版環境下拉選單中加入 SSH 連線。 | `"sshConfigs": [ { "id": "dev", "name": "開發機", "sshHost": "me@dev.example.com" } ]`<br><sub>格式示意</sub> |
| `sshHostAllowlist` | 僅組織管理 | 限制桌面版 SSH session 可連的主機。 | `"sshHostAllowlist": ["*.corp.example.com"]`<br><sub>格式示意</sub> |
| `disableDesktopLocalSessions` | 僅組織管理 | 關閉桌面版的本機 Code session。 | `"disableDesktopLocalSessions": true` |
| `disableMobileSimulatorTools` | 僅組織管理 | 禁止 Claude 操作 iOS 模擬器面板。 | `"disableMobileSimulatorTools": true` |
| `browserExternalPageTools` | 僅組織管理 | 桌面內建瀏覽器中，不讓 Claude 的工具作用在外部網頁。 | `"browserExternalPageTools": false` |
| `disableBrowserExternalNavigation` | 僅組織管理 | 桌面內建瀏覽器只能開 localhost。 | `"disableBrowserExternalNavigation": true` |

<a id="cat-env"></a>

### 🔑 環境、驗證與更新

環境變數、登入方式、憑證腳本、版本控管。

| 設定 key | 範圍 | 說明 | 範例 |
| --- | --- | --- | --- |
| `env` | 任何設定檔 | 每個 session 與子程序都會帶上的環境變數。 | `"env": { "CLAUDE_CODE_SUBAGENT_MODEL": "sonnet", "NODE_ENV": "development" }` |
| `apiKeyHelper` | 任何設定檔 | 執行腳本動態產生 API 憑證（例如從密碼管理器取）。 | `"apiKeyHelper": "~/.claude/get-key.sh"` |
| `awsAuthRefresh` | 任何設定檔 | Bedrock 憑證過期時執行的刷新指令。 | `"awsAuthRefresh": "aws sso login --profile dev"` |
| `awsCredentialExport` | 任何設定檔 | 以 JSON 輸出 Bedrock 憑證的指令。 | `"awsCredentialExport": "~/.claude/aws-creds.sh"` |
| `gcpAuthRefresh` | 任何設定檔 | 刷新 Google Cloud 憑證的指令。 | `"gcpAuthRefresh": "gcloud auth application-default login"` |
| `otelHeadersHelper` | 任何設定檔 | 產生輪替 OpenTelemetry header 的指令。 | `"otelHeadersHelper": "~/.claude/otel-headers.sh"` |
| `forceLoginMethod` | 任何設定檔 | 限制登入方式：claudeai、console、gateway。 | `"forceLoginMethod": "claudeai"` |
| `forceLoginOrgUUID` | 任何設定檔 | 只允許登入指定組織的帳號。 | `"forceLoginOrgUUID": "123e4567-e89b-12d3-a456-426614174000"` |
| `forceLoginGatewayUrl` | 僅組織管理 | 登入畫面要連的公司閘道網址。 | `"forceLoginGatewayUrl": "https://claude-gw.acme.com"` |
| `gatewayInternalNetworks` | 僅組織管理 | 讓 /login 能連到位於公司內網公有 IP 段的閘道。 | `"gatewayInternalNetworks": ["203.0.113.0/24"]` |
| `allowedProviders` | 僅組織管理 | 限制這台機器可用的 API 供應商。 | `"allowedProviders": ["anthropic", "bedrock"]` |
| `skipWebFetchPreflight` | 任何設定檔 | 連不到 Anthropic 時跳過 WebFetch 的網域預檢。 | `"skipWebFetchPreflight": true` |
| `autoUpdatesChannel` | 任何設定檔 | 自動更新頻道：latest（最新）或 stable（約晚一週、較穩）。 | `"autoUpdatesChannel": "stable"` |
| `minimumVersion` | 任何設定檔 | 自動更新不會裝到比這更舊的版本。 | `"minimumVersion": "2.1.300"` |
| `requiredMinimumVersion` | 僅組織管理 | 低於此版本就拒絕啟動。 | `"requiredMinimumVersion": "2.1.300"` |
| `requiredMaximumVersion` | 僅組織管理 | 高於此版本就拒絕啟動。 | `"requiredMaximumVersion": "2.1.312"` |

<a id="cat-managed"></a>

### 🏢 組織管理專用

給 IT／管理員部署政策用，一般使用者不需要設定。

| 設定 key | 範圍 | 說明 | 範例 |
| --- | --- | --- | --- |
| `managedSourcesBehavior` | 僅組織管理 | 多個管理來源時：first-wins（只用最高優先的）或 merge（合併）。 | `"managedSourcesBehavior": "merge"` |
| `parentSettingsBehavior` | 僅組織管理 | SDK 或 IDE 宿主傳入的限制要合併還是捨棄。 | `"parentSettingsBehavior": "merge"` |
| `policyHelper` | 僅組織管理 | 啟動時執行程式來計算管理設定。 | `"policyHelper": { "path": "/usr/local/bin/claude-policy", "timeoutMs": 5000 }`<br><sub>格式示意</sub> |
| `policyHelper.path` | 僅組織管理 | policy helper 執行檔路徑。 | `"policyHelper": { "path": "/usr/local/bin/claude-policy" }` |
| `policyHelper.refreshIntervalMs` | 僅組織管理 | 背景重新執行 helper 的間隔（毫秒）。 | `"policyHelper": { "refreshIntervalMs": 3600000 }` |
| `policyHelper.timeoutMs` | 僅組織管理 | 等待 helper 的上限（毫秒）。 | `"policyHelper": { "timeoutMs": 5000 }` |
| `forceRemoteSettingsRefresh` | 僅組織管理 | 啟動前必須成功取得伺服器下發的設定，否則不啟動。 | `"forceRemoteSettingsRefresh": true` |
| `wslInheritsWindowsSettings` | 僅組織管理 | 讓 WSL 讀取 Windows 端的管理政策。 | `"wslInheritsWindowsSettings": true` |

<a id="cat-deprecated"></a>

### 🗄️ 已棄用（看到舊教學時參考）

這些 key 已不建議或不再有效。

| 設定 key | 範圍 | 說明 | 範例 |
| --- | --- | --- | --- |
| `includeCoAuthoredBy` | 任何設定檔 | 已棄用，改用 attribution。 | `"attribution": { "commit": "" }` |
| `disableArtifact` | 任何設定檔 | 已棄用，改用 enableArtifact: false。 | `"enableArtifact": false` |
| `keybindingFlavor` | 任何設定檔 | 已無作用（文字編輯鍵一律 readline 風格）。 | （不需設定） |
| `permissionExplainerEnabled` | ~/.claude.json | v2.1.257 移除。 | （不需設定） |
| `taskOutputMaxChars` | 任何設定檔 | v2.1.277 移除。 | （不需設定） |
| `teammateDefaultModel` | ~/.claude.json | v2.1.234 移除。 | （不需設定） |


## 8.7 一份完整的範例設定檔

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "model": "opus",
  "effortLevel": "high",
  "language": "繁體中文",
  "outputStyle": "Explanatory",
  "theme": "dark",
  "timeFormat": "24-hour",
  "autoUpdatesChannel": "stable",
  "permissions": {
    "defaultMode": "acceptEdits",
    "allow": ["Bash(npm run test *)", "Bash(git diff *)", "Bash(git status)"],
    "ask": ["Bash(git push *)"],
    "deny": ["Read(./.env)", "Read(./.env.*)"]
  },
  "sandbox": {
    "enabled": true,
    "filesystem": { "denyRead": ["~/.ssh"] },
    "network": { "allowedDomains": ["github.com", "*.npmjs.org"] }
  },
  "statusLine": { "type": "command", "command": "~/.claude/statusline.sh" },
  "env": { "CLAUDE_CODE_SUBAGENT_MODEL": "sonnet" },
  "cleanupPeriodDays": 60
}
```
