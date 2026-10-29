# 第 10 章　模型、推理強度、Fast mode 與 Context 管理

## 10.1 選哪個模型？

![/model 選單](../images/17-model.png)

| 選項 | 模型（2026/10） | 特色 | 價格（每百萬 token 輸入／輸出） |
| --- | --- | --- | --- |
| Default（推薦） | 目前為 Opus 5.5 | 帳號預設 | — |
| Opus | Opus 5.5 | 日常與複雜任務的最佳平衡 | $4／$20 |
| Fable | Fable 5.1 | 最強，適合最難、最長的任務（需有權限） | $10／$50 |
| Sonnet | Sonnet 5.5 | 例行任務、快、便宜 | $2／$10 |
| Haiku | Haiku 5.5 | 最快最便宜，適合簡單問答 | $0.10／$0.50（超過 10 萬 token 的提示較貴） |

> 訂閱方案使用者不是按 token 付費，而是消耗方案額度；越強的模型消耗越快。

**切換方式**（優先順序由高到低）：

1. session 中 `/model sonnet`（選單中按 <kbd>s</kbd> 只用於本次）
2. 啟動時 `claude --model sonnet`
3. 環境變數 `ANTHROPIC_MODEL=sonnet`
4. 設定檔 `"model": "sonnet"`

**特殊別名**：`opusplan` 在 Plan 模式用 Opus 想、執行時用 Sonnet 做，省額度；`best` 有 Fable 就用 Fable，否則用 Opus。

## 10.2 推理強度（Effort）

![/effort 滑桿](../images/23-effort.png)

| 等級 | 適合 |
| --- | --- |
| low | 改錯字、格式化、簡單問題（最快、最省） |
| medium | 一般開發（Opus／Sonnet／Haiku 5.5 的預設） |
| high | 較難的除錯、跨多檔修改 |
| xhigh | 架構設計、困難演算法 |
| max | 最難的問題（只對本次 session 有效） |

```
/effort high          直接設定
/effort               打開滑桿
/effort auto          回到模型預設
claude --effort xhigh 啟動時指定
```

**ultrathink**：在提示中任何地方寫 `ultrathink`，**只有這一回合**會用 xhigh 深入思考，不改變整體設定。例如：「ultrathink 為什麼這個 race condition 只在正式環境出現？」

**ultracode**：不是強度等級，而是讓 Claude 對大任務自動派出多個子代理（動態工作流程）。單次使用就在提示中寫 `ultracode:`；整個 session 開啟用 `/effort ultracode`。很耗額度，見第 13 章。

> ⚠️ 從舊模型換到 Opus 5.5 時，預設會從 medium 起跳。若你習慣 high，請在 5.5 上執行一次 `/effort high`（會記住）。

## 10.3 Fast mode

- 同一個 Opus 模型，速度最高約 **2.5 倍**，品質不變，單價較高（Opus 5.5 為 $8／$40）。
- 開關：`/fast` 或 <kbd>Alt+O</kbd>（Mac：<kbd>Option+O</kbd>）；開啟時輸入框旁出現 `↯`。
- 訂閱方案使用 Fast mode 會從 **usage credits（額外儲值）** 扣款，不含在月費額度裡；需要先開啟 usage credits。Team／Enterprise 需管理員開放。
- 小技巧：**在對話一開始就開**，中途開會用高價重算整段 context 一次。

## 10.4 Context 管理

![/context](../images/32-context.png)

上圖解讀：

| 項目 | 意思 |
| --- | --- |
| `32.7k/1m tokens (3%)` | 已用 3.27 萬，上限 100 萬 |
| System prompt／System tools | Claude Code 本身的說明與工具定義（固定成本） |
| Skills | 可用 skills 的清單 |
| Messages | 你們的對話 |
| Free space | 剩餘空間 |
| Autocompact buffer | 預留給自動壓縮的空間 |

**保持 context 健康的習慣**：

1. **一個任務一個對話**：換題目就 `/clear`。
2. **主動壓縮**：`/compact 只保留資料庫結構的決定`。
3. **把吵雜工作交給子代理**：大量搜尋、log 分析讓 subagent 做，只回傳摘要。
4. **精簡 CLAUDE.md**：長流程改寫成 skill（需要時才載入）。
5. **關掉用不到的 MCP**：每個 MCP 工具定義都佔空間。

相關設定：`/autocompact 500k`（自動壓縮門檻）、`"idleCompaction": false`（關閉閒置壓縮）、`CLAUDE_CODE_DISABLE_1M_CONTEXT=1`（強制 20 萬視窗）。

## 10.5 省額度的實用技巧

- 例行工作用 medium 甚至 low；需要時才 `ultrathink`。
- 子代理用 Sonnet：設定 `"env": { "CLAUDE_CODE_SUBAGENT_MODEL": "sonnet" }`。
- 經常 `/usage` 看哪個 skill、MCP 最耗。
- 切換模型、effort、output style 會讓快取失效，下一次請求較貴，避免頻繁切換。
- 很久沒動的大對話，重新開始可能比 `/resume` 更省。
