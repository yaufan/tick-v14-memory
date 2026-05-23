---
name: V14_T2 (T+2 horizon) findings
description: V14 T+2 horizon model audit + apples-to-apples vs V3 + dual-horizon strategy comparison (2026-05-22)
type: project
originSessionId: 98433687-e388-4a6e-b3c9-c5bc3fe2102f
---
V14_T2 = V12Inception × 5 seeds, walk-forward W1-W5, label_t2 (T+2 exit), dataset_v14_multislot.

## Headline numbers (12:30 slot, full universe)

- V14_T2 12:30 5y cum: **+3,303K**
- V14_T1 12:30 5y cum: +4,030K (T+1 horizon stronger)
- Best slot: 12:30 across all 6 slots

## Apples-to-apples vs V3 (Attn-29 ×10s + T+2)

5y on (sid,date) intersection (28-38K keys/year):
- V3 on intersection: **+1,985K**
- V14_T2 on intersection: **+1,282K**
- Δ: **−703K (−35.4%)** — V14_T2 UNDERPERFORMS V3 on same universe

**Why:** V14_T2's +3.30M headline comes mostly from broader candidate universe in dataset_v14_multislot, not from genuine alpha over V3's Attn-29 ensemble.

Compare with V14_T1: +16.8% over V13 baseline (positive but small alpha).
V14_T2 has negative alpha vs V3 — confirms feedback memory "Attention 架構合 T+2/T+3, CNN 合 T+1": at T+2, Inception is worse than Attention.

## 4 audits — all PASS

- #1 Shuffle: 5/5 (delta +119K to +1,323K)
- #2 Per-stock signal: mean corr +0.016, median +0.013, %>0 = 59.4% (much weaker than T1: +0.092, 87.2%)
- #3 Top-N sensitivity: top 1%/3% = 4/5
- #4 V14_T2 vs V14_T1 picks overlap: 21-48% (substantially different picks)

## Dual-horizon (T1+T2) strategies on 12:30

| Strategy | 5y cum | PnL/\|DD\| |
|---|---:|---:|
| S0 T1_only | +4,030K | 25.12 |
| S1 T2_only | +3,303K | 10.12 |
| S2 inter_T1exit (top 2% both, T+1) | +2,446K | **36.07** (best risk-adj) |
| S3 inter_T2exit (top 2% both, T+2) | +2,571K | 12.82 |
| **S4 mean_pred_T2** (avg z, T+2) | **+5,248K** | 14.20 (best single-port PnL) |
| S5 union_2x_T2 | +4,053K | 10.52 |
| S6 T1+T2_sum (independent) | +7,333K¹ | —¹ |

¹ S6 DD not joined daily — theoretical upper bound only.

## Verdicts
- **Best single PnL**: S4 mean_pred_T2 (+30.3% over T1-only). Worth audit + apples-vs-V3 if pursuing.
- **Best risk-adj**: S2 inter_T1exit (DD only −68K).
- **Production**: do NOT replace V3 with V14_T2 alone — apples test fails.

## Files
- `ML_auto_processing_v4/_V14_T2_FINAL_VERDICT.md` — aggregated verdict (★ start here)
- `_V14_T2_BACKTEST_VERDICT.md`, `_V14_T2_AUDIT_VERDICT.md`, `_V14_T2_APPLES_VS_V3_VERDICT.md`, `_V14_DUAL_HORIZON_VERDICT.md`
