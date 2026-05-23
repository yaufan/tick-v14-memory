---
name: V14 Multi-slot + N=3 Consensus 機制 (2026-05-21 發現)
description: V14_T1 multi-slot 跨 slots N=3 連續確認 + addon, 比 V14 single-slot baseline risk-adjusted 高 41%, mean win% 65%
type: project
originSessionId: 98433687-e388-4a6e-b3c9-c5bc3fe2102f
---
# V14 N=3 Consensus 機制

## TL;DR

V14 multi-slot 模型 (6 個 slots: 10:00/10:30/11:00/11:30/12:00/12:30) 各自取 top 2%, 用以下規則:
- **3 個連續 slot 都被推薦** → 在第 3 slot's close 進場 1 lot
- **第 4 個 slot 也被推薦** → addon 加碼 1 lot (max 2 lots/stock)
- Exit: T+1 09:00
- Budget cap: 1M/day

## 為什麼 N=3 是 sweet spot

跨 N=2/3/4 sweep (`_v14_consecutive_n_sweep.py`):

| N | 5y PnL | trades | win% | mean DD | sharpe | **PnL/\|DD\|** |
|---|---:|---:|---:|---:|---:|---:|
| 2 (寬鬆) | +3,779K | 2,413 | 59.8% | -161K | +0.268 | 23.5 |
| **3 ⭐** | +3,568K | 1,401 | **65.0%** | **-100K** | +0.342 | **35.6** |
| 4 (嚴格) | +2,735K | 931 | 67.7% | -83K | +0.357 | 33.1 |

N=3: **risk-adjusted 最佳**, 比 N=2 高 51%.

## Vs V14_T1 single-slot baseline

| 指標 | V14_T1 12:30 | **V14 N=3** |
|---|---:|---:|
| 5y PnL | +4,030K | +3,568K (-12%) |
| Mean DD | -160K | **-100K (-38%)** |
| Win rate | ~58% | **~65%** |
| Sharpe | +0.277 | +0.349 |
| PnL/\|DD\| | 25.2 | **35.6 (+41%)** |

絕對 PnL 略低, 但風險調整後**顯著更好** + 勝率高 7 pp + DD 少 38%.

## 4 個觸發窗口 (每天最多 4 個 entry chance)

| 連續 3 slots | Entry @ | Addon @ (4th slot) |
|---|---|---|
| 10:00, 10:30, 11:00 | 11:00 close | 11:30 close |
| 10:30, 11:00, 11:30 | 11:30 close | 12:00 close |
| 11:00, 11:30, 12:00 | 12:00 close | 12:30 close |
| 11:30, 12:00, 12:30 | 12:30 close | (無, 12:30 是最後) |

## Addon 觸發頻率

5 年中, **60% trades 有 addon** (842/1,401). 連續 4 都確認的股票很常見, 信號穩定.

## 詳細結果見

- 本機 verdict: `_V14_CONSECUTIVE_N_SWEEP_VERDICT.md`
- 腳本: `_v14_consecutive_n_sweep.py`
- 比較對象 V14_T1 audit: `_V14_T1_AUDIT_VERDICT.md`
- V14_T1 vs V13: `_V14_T1_APPLES_TO_APPLES_VERDICT.md` (V14 真正 alpha +16.8%, 其餘 universe 擴張)

## 使用情境

**保守型 + 高勝率**: N=3 適合.
**追絕對 PnL**: V14_T1 single-slot 12:30 較高.

Live paper trade 推薦先用 **V14 N=3** (audit pass + 高勝率 + 低 DD).
