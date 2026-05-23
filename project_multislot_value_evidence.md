---
name: Multi-slot Entry Dataset 的價值證據 (2026-05-18 backtest)
description: Entry × Exit 2D 網格 backtest 發現 — V13/V3 picks 在 09:00 進場可達 5x PnL (counterfactual), 證明 multi-slot dataset 值得做
type: project
originSessionId: 98433687-e388-4a6e-b3c9-c5bc3fe2102f
---
# Multi-slot Entry Dataset 的價值證據

## TL;DR

2026-05-18 跑「進場時間 × 出場時間」2D grid (`_entry_exit_grid_v2.py`), 對 V13 + V3 既有 picks 套用不同進場時點 (09:00 / 10:00 / 11:00 / 11:30 / 12:00 / 12:30 / 13:30). 發現:

**進場越早, PnL 越高 — 線性遞減** (相對 12:30 baseline):

| Entry 時點 | V13 PnL × baseline | V3 PnL × baseline |
|---|---:|---:|
| 09:00 open | **4.6x** (+5,069K vs +1,109K) | **2.9x** (+5,848K vs +1,985K) |
| 10:00 K1m | 2.4x | 1.5x |
| 11:00 K1m | 1.3x | 1.0x |
| 11:30 K1m | 0.8x | 0.6x |
| 12:00 K1m | 0.1x | 0.3x |
| 12:30 (current) | 1.0x (baseline) | 1.0x (baseline) |

## ⚠️ 重要 caveat: 這**不是可執行**策略

V13/V3 模型用的 features 包含 **T 11:30-12:30 K1m (k_seq)**. 09:00 時 features 不存在 → 「09:00 進場用 12:30 signal」= **time travel / lookahead bias**.

實際 live trading 必須在 12:30 才能跑 inference, 不能往前.

## 但這 counterfactual 告訴我們三件大事

### 1. V13/V3 picks alpha **真實且強**
- 從 09:00 → T+1 09:00 picks 平均漲 ~1.5%
- 從 12:30 → T+1 09:00 picks 平均漲 ~0.3% (V13 抓到的部分)
- **70-80% 的 alpha 在 09:00→12:30 早盤已 price in**, 12:30 進場只抓 20-30%

### 2. Alpha 在早盤的衰減模式幾乎**線性**
從 09:00 → 12:00 PnL 從 5M 線性減到 0.15M. 表示 picks 的 directional alpha 是**整個早盤一路 realize** 的, 不是某個瞬間突發.

### 3. **Multi-slot dataset 的理論上限是 2-5x PnL**
若能訓練「10:00 entry 專屬模型」(features 只用 09:00-10:00 K1m + 前日法人/融資), 預期可實現 counterfactual gain 的 70-80%, 即:
- V13 10:00 模型: 5y +1,109K → ~+2,000K (估)
- V3 10:00 模型: 5y +1,985K → ~+3,000K (估)
- V13 09:30 模型 (盡可能早): 上限 ~+4,000K

## 怎麼做 Multi-slot Dataset (簡述)

### 改 build script
1. `_rebuild_v3_macro3_dataset.py` 改成:
   - 多次走訪每個 T 日, 每個 entry slot (e.g. 10:00, 11:00, 12:30) 各生成一筆 (sid, date, slot) 樣本
   - 每筆樣本的 features 重新計算:
     - `k_seq` 範圍從 [11:30-12:30] 改成 [slot-60min, slot]
     - `intraday_*` features 用該 slot 為止的資料
     - `time_to_close` 變成 **真正有變異的 feature** (slot=10:00 → 210 min; slot=12:30 → 60 min)
   - 'buy_price' = T slot K1m close
   - label = (next_open - buy_price) / buy_price (T+1 09:00 不變)

### 訓練
- 同一個模型 (V13 Inception / V3 Attn-29) 訓練, 加 `time_to_close` 作為 input (現在是真有變異)
- 預期模型自動學會「slot 越早, 信號越保守 (features 少); slot 越晚, 信號越強」

### 部署
- Live: 多個 slot 各跑 inference (10:00 / 11:00 / 12:30), 看哪個 slot 的 top picks 最強
- 或: 每個 slot 各下一些單, 分散時間風險

## 樣本量考量
原 dataset 251K rows. Multi-slot (e.g. 5 slots) × ~50K T-days = ~250K rows per slot. **訓練樣本同等規模**, OK.

K1m 全部已有(09:00-13:30 全天), 不需新抓資料.

## 開發成本估
- Build multi-slot dataset: ~1 天
- 訓練每個 slot model: ~3 hr per model
- Live infra: ~半天

## 為什麼之前沒做

之前一直在改善 12:30 模型 (V1 → V5 → V13 → V3). 並不知道「早盤進場的上限這麼高」. 今天 entry × exit grid 才量化出來.

## 何時做?

**新 session 評估**:
- V13/V3 paper trade 30 筆對齊度 OK → 確認模型在 live 有效
- 若 live PnL 跟 backtest 對齊 → multi-slot dataset 是下一個大方向
- 預期 ROI 是該系列**最大**改進機會 (其他 incremental 都 < 50%)

## 參考檔案
- 本機 verdict: `_ENTRY_EXIT_GRID_V2_VERDICT.md`
- 腳本: `_entry_exit_grid_v2.py`
- 既有 TODO: `project_v12_todo_features.md`
