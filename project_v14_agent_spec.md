---
name: V14 N=3 Agent 操作手冊 (新 session 用)
description: 用戶在另一 session 開 agent, 輸入「啟動」時跑 V14 N=3 推論並輸出推薦
type: project
originSessionId: 98433687-e388-4a6e-b3c9-c5bc3fe2102f
---
# V14 N=3 Agent — 操作手冊

## 用戶 workflow

```
[新 Claude Code session in tick/]
用戶: 「啟動」
↓
Agent (新 session) 跑:
  cd C:\Work\Claude Project\tick\v14_agent_consec3_v1
  python live_v14_consec3.py
↓
Agent 輸出推薦清單 (entry / addon, per consecutive trigger)
↓
用戶手動下單 → add_position.py 記錄
↓
T+1 09:00 賣出 → close_position.py
```

## Agent 預備工作 (已完成)

| 檔案 | 用途 |
|---|---|
| `C:\Work\Claude Project\tick\v14_agent_consec3_v1\` | Agent folder |
| `inception_seed{1..5}.pth` | V14_T1 W5 weights (5 seeds, V12Inception, d_dim=29) |
| `live_v14_consec3.py` | 主腳本 (用戶輸入「啟動」時跑這個) |
| `add_position.py` | 進場後記錄 |
| `close_position.py` | T+1 09:00 出場後關倉 |
| `stock_info.csv` | sid→股名 |
| `positions.json` | 持倉 |

## live_v14_consec3.py 邏輯

### Input
- 從 v3/2026 讀本地 daily/k1m/inst/margin/macro
- 從 FinMind snapshot 拉現價 (12:30 之前用 K1m bar by minute)

### 流程
```
1. 判斷現在 slot (10:00 / 10:30 / 11:00 / 11:30 / 12:00 / 12:30)
2. 對所有完成的 slots 跑 V14 inference (V12Inception, time_to_close 變數)
3. 算每個 slot 的 top 2% sids
4. 找 N=3 consecutive triggers (在已完成 slots 中):
   - 10:00 ∩ 10:30 ∩ 11:00 → entry @ 11:00 (若 11:00 已過)
   - 10:30 ∩ 11:00 ∩ 11:30 → entry @ 11:30 (若 11:30 已過)
   - 11:00 ∩ 11:30 ∩ 12:00 → entry @ 12:00
   - 11:30 ∩ 12:00 ∩ 12:30 → entry @ 12:30
5. 找 addon triggers (entry 已過後的第 4 slot 也 picks):
   - 例: 10:00-10:30-11:00 entry@11:00, 若 11:30 也 picks → addon @11:30
6. 顯示分類:
   - 🆕 NEW ENTRIES (本 slot 新觸發的 3-consec, 該進場)
   - ➕ ADDONS (已 entry 的, 該加碼)
   - 👀 WATCHING (1-2 個 slot 已 confirm, 等下個確認)
```

### 輸出格式
```
=========================================
V14 N=3 Agent — 5/21 11:32  (current slot: 11:30 ready)
=========================================

🆕 NEW ENTRIES (3-consec triggered, 該進場 1 lot @ 該 slot close):

| sid | 名稱 | 確認 slots | entry slot | entry $ | pred_T1 | atr% |
|-----|-----|-----------|-----------|--------|--------|-----|
| 2330 | 台積電 | 10:00-10:30-11:00 | 11:00 | 1250 | +1.5% | 1.2% |

➕ ADDONS (已 entry 的, 4th slot 確認, 該加碼 1 lot):

| sid | 名稱 | 已 entry | addon slot | addon $ |
|-----|-----|---------|-----------|--------|
| 2454 | 聯發科 | 11:00 (yest cmd) | 11:30 | 1185 |

👀 WATCHING (2 個 slot 已確認, 等 11:30 或之後)

| sid | 名稱 | 已 confirm | 等 |
|-----|-----|-----------|-----|
| 6505 | 台塑化 | 10:30-11:00 | 等 11:30 |

EXIT 提醒: T+1 09:00 開盤賣 (= 明天 5/22 09:00)
持倉: ...
```

## 重要規則

1. **每個 slot 最多 1 個 trigger event** — agent 應該記錄已 trigger 的, 不重複推薦
2. **持倉滿 2 lots 後不再加碼**
3. **每日 budget 1M** — agent 應該算累計已下單金額
4. **執行視窗**: 用戶每 30 min 跑一次 (11:00, 11:30, 12:00, 12:30) — 每次都會出新 trigger

## V14 模型架構 (對齊訓練)

`V12Inception` (multi-kernel parallel CNN):
- InceptionBlock: kernel 1/3/5/7 並聯 → concat → BN → GELU → Dropout
- d_dim=29 (production 26 + macro3 TAIEX/IXIC/DJI)
- k_dim=5 (OHLCV)
- 3 個 InceptionBlock × 兩塔 (daily + k1m)
- Head: Linear 256→128→32→1

## V14 features 與 V13 不同點

| 維度 | V13 | V14 |
|---|---|---|
| time_to_close | 60 (常數) | **60-210 (依 slot)** ⭐ |
| k_seq window | 11:30-12:30 (固定) | **slot-60min ~ slot (動態)** |
| 訓練 samples 包含 slots | 12:30 only | **10:00 ~ 12:30 都有** |

## 用戶輸入「啟動」時 agent 該做的

```bash
cd "C:/Work/Claude Project/tick/v14_agent_consec3_v1"
python live_v14_consec3.py
```

並把輸出 (推薦清單) 給用戶看, 由用戶決定下單.

## 已知限制 / TODO

- **今日 K1m**: ✅ 已解(2026-05-21 重構) → V14 改讀 `snapshot_stream/<date>/*.parquet` 用 `aggregate_snapshot_stream.aggregate_today(hybrid_fill=False)`(pure today, no T-1 drift, no yf)
- **Collector 啟動時機**: ✅ 已自動化(2026-05-22 註冊 Task Scheduler `TickV14_Collector` 週一-五 08:30 起跑 → 13:30 停)
- **V14 早跑 schedule**: ✅ 自動 — `TickV14_Slot1000/1030/1100/1130/1200/1230` 各 slot 提前 5 min 跑(tol=5 min freshness 通過)
- **歷史 d_seq features**: 用 local v3/{year}/

## 2026-05-22 重大重構(改進 spec)

1. **Pure collector K1m source** (no yf/no T-1):`get_k1_window_at_slot` 改成從 `aggregate_snapshot_stream` 拿;slot 之前的所有 minute bars 來自 collector(每 10s poll FinMind tick_snapshot)
2. **Pre-filter universe**: `filter_universe()` 函式 close≥50 + 20-day turnover≥10M (~742 sids) — 加速 inference 3-5 倍
3. **Picks cache**: `recommendations/<today>_slots.json` 載入時跳過已 infer 的 slot,只算新 slot → 第二次 run < 1 min
4. **處置股 filter**: `fetch_disposition_set()` (port from V13) 自動排除全額交割股
5. **Freshness tolerance**: `SLOT_FRESHNESS_TOL_MIN=5` 允許提前 5 min 啟動;`completed_slots` 含 upcoming if within tol
6. **Enrich tradable section**: `_enrich_all_slots.py` 加「可下單 picks (filter 漲停後)」section,用台股 tick rule 算正確漲停價;處置股額外標記 🚫
7. **Extended picks (top-30)**: `_extended_picks.py` re-runs inference 不只 top 2%,用於過熱市況找 top 2% 以外的 actionable
8. **(5/22 PM)Smart picks 整合**: V14 主程式 `run_slot_inference` 改成 一次 inference 同時 keep top 30 (非只 top 2%):
   - `n_primary = max(1, int(len(cands) * TOP_PCT))` 標 `is_primary=True`
   - `n_extended = max(30, n_primary*4)` 全存,標 rank 1-30
   - JSON 每 slot 存 top 30 picks (有 rank + is_primary 欄位)
   - **N=3 consensus 只算 primary** (legacy 兼容:無欄位視為 primary)
   - Enrich 「可下單 picks」自動把 primary + extended 合進 tradable list,⭐=primary 🟢=extended fallback
   - **效益**: 過熱市況 primary 全漲停時,extended fallback 自動補進可下單清單;backtest 邏輯(N=3) 不受影響

## 跟 V14 訓練分布的關鍵 trade-off

- V14 backtest 假設 fill at slot close — 漲停股 backtest 表現好但 live 買不到
- 過熱市況下,V14 picks 可能 80%+ 都漲停 → 強訊號但無法執行
- Workaround: 用 enrich tradable section 看「未漲停 picks」(訊號弱但能進場)
- 已下單例子:5/21 5243+3213 各 +3%, 5/22 5287 數字(N=1, ATR 1.04% 全最低)

## Audit 狀態

V14_T1 12:30 audit pass 4/4 (見 `_V14_T1_AUDIT_VERDICT.md`).
N=3 consensus rule 是 V14_T1 picks 的 post-process, 不需另跑 audit.

## 相關 memory

- `project_v14_n3_consensus.md` ← 核心研究發現 + 詳細數據
- `project_v13_v3_paper_trade_plan.md` ← 既有 paper trade 框架
- `paper_trading_workflow.md` ← 一般 workflow
