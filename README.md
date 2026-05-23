# tick-v14-memory

Claude Code auto-memory snapshot for V14 (Taiwan stock overnight trading ML system).

跨設備轉移用。源頭機器：`tick/ver12/ML_auto_processing_v4/`。

## 內容（5 個 memory 檔）

| 檔案 | 內容 |
|---|---|
| `project_multislot_value_evidence.md` | Multi-slot dataset 動機（entry × exit 2D grid 發現 09:00 進場 counterfactual 達 5x PnL） |
| `project_v14_n3_consensus.md` | V14 N=3 連續 confirm 機制 + N=2/3/4 sweep（5y +3,568K, win 65%, PnL/\|DD\|=35.6） |
| `project_v14_agent_spec.md` | V14 N=3 Agent 操作手冊（用戶說「啟動」時的 SOP，含 2026-05-22 重構） |
| `project_v14_paper_trade_log.md` | 5/21 +$7,300 勝率 2/2、5/22 5287 open 等實戰 log |
| `project_v14_t2_findings.md` | V14_T2 + dual-horizon 13 策略分析（V14_T2 vs V3 apples −35.4%、S9 split_lots +5.52M） |

## 在新設備使用

1. clone repo：
   ```bash
   git clone https://github.com/yaufan/tick-v14-memory.git
   ```

2. 把 5 個 `.md` 複製到新機 Claude Code memory 目錄：
   ```
   C:\Users\<user>\.claude\projects\<project-id>\memory\
   ```
   `<project-id>` 是依工作目錄路徑 hash 算的；用同樣的工作目錄路徑（如 `C:\Work\Claude Project\tick\ver12\ML_auto_processing_v4`）會自動對齊。

3. 編輯目標機的 `MEMORY.md`（**不要整檔覆蓋**，會把那邊既有 V13/V3 索引弄丟），新增以下 5 行：

   ```markdown
   - [Multi-slot Entry 的價值證據](project_multislot_value_evidence.md) — 09:00 entry counterfactual 達 5x PnL; 證明 multi-slot dataset 是該系列最大改進方向
   - [V14 N=3 Consensus 機制](project_v14_n3_consensus.md) — 6 slots 連續 3 確認進場 + 4th 加碼; 5y +3,568K, 勝率 65%, PnL/|DD|=35.6 全 V14 最高
   - [V14 N=3 Agent 操作手冊](project_v14_agent_spec.md) — 用戶說「啟動」時, 新 session 跑 v14_agent_consec3_v1/live_v14_consec3.py (★ 5/22 重構 + Task Scheduler 全自動)
   - [V14 Paper Trade 實戰 log](project_v14_paper_trade_log.md) — 每日進出場 + 模型 attribution (5/21: +$7,300 勝率 2/2; 5/22 5287 open)
   - [V14_T2 (T+2 horizon) findings](project_v14_t2_findings.md) — V14_T2 12:30 +3.30M 但 apples vs V3 −35.4%; S4 mean-pred ensemble +5.25M best single PnL; S2 inter +2.45M best risk-adj (2026-05-22)
   ```

4. 重開 Claude Code session — 自動 load。
