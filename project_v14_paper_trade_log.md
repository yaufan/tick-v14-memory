---
name: V14 Paper Trade 實戰交易 log
description: V14 N=3 Consensus paper trade 每日進場+出場記錄,含 pred vs 實際差、模型 attribution、過熱市況問題
type: project
originSessionId: 5262005c-fb3d-4d94-a14d-cf31d8a64b57
---
# V14 Paper Trade Log

每日進場/出場 + 模型表現追蹤。Position 細節在 `tick/v14_agent_consec3_v1/positions.json`。

## 2026-05-21 (V14 上線第一日)

**進場**(各 1 張,$230.7K total):

| sid | 名稱 | slot | buy | pred T+1 | 進場 rationale |
|---|---|---|---|---|---|
| 5243 | 乙盛-KY | 10:00 (N=1) | 112.6 | +2.44% | 唯一未漲停 + ATR 6.7% + 量比 0.78x |
| 3213 | 茂訊 | 12:30 (N=1) | 118.1 | +2.59% | ATR 2.64% 低波動 + 未漲停 + 量比 2.42x |

**5/22 09:00 出場(T+1 open)**:

| sid | sell | PnL | % | vs pred |
|---|---|---|---|---|
| 5243 | 116.0 | +$3,400 | +3.02% | 略超預期(+0.58pp) |
| 3213 | 122.0 | +$3,900 | +3.30% | 超預期(+0.71pp) |

**勝率 2/2,Net +$7,300 (+3.16%)**。模型平均 +2.5% 預測 → 實際 +3.16%,low ATR picks 表現超預期。

## 2026-05-22 (V14 第二日 — 過熱市況遇到結構性問題)

**問題**:V14 picks 95%+ 已漲停 → 實務無法 fill。
- Slot 11:00 觸發 N=3 entry 6 檔(6727, 4760, 4722, 6419, 6271, 3581)全部漲停
- Slot 12:00/12:30 加碼 trigger 5 檔全漲停
- 唯一可下單 = 4722 國精化(剛打開 1 tick)+ 9942 茂順 + 2745 五福 + 4771 望隼 + 5287 數字(後 4 個是 extended top-30 或 N=1 picks)

**進場**(1 張,$153.5K):

| sid | 名稱 | slot | buy | pred T+1 | 進場 rationale |
|---|---|---|---|---|---|
| 5287 | 數字 | 12:30 (N=1) | 153.5 | +3.26% | **ATR 1.04% 全 picks 最低**、量比 5.7x、未漲停離 12.5、賣盤 1 張 |

**5/25 (週一) 09:00 待出場**。

## 觀察 / 待驗證

1. **「ATR 最低」是當日最 robust 進場準則**:5/21 兩檔贏家都低 ATR (6.49% 6.68%),5/22 5287 也選最低
2. **V14 在過熱市況勝率下降**:模型訓練資料可能 over-represent 漲停動能,實戰買不到 → 結構性 gap
3. **N=3 連續 confirm 不一定可進場**:5/22 N=3 6 檔全漲停 → 模型強訊號 ≠ 實務 actionable
4. **未漲停 picks 雖 N=1 但可能反而是真機會**:5/21 兩贏家都 N=1
5. **跨日延續性高**:5/22 picks (6271, 4760, 6727) 是 5/21 picks 延續 → 模型一致性 OK

## 操作 SOP(2026-05-22 標準化後)

- 週一-五 Task Scheduler 自動跑:08:30 collector + 09:55-12:25 V14 每 30 min
- 用戶開 Claude → `cd tick/v14_agent_consec3_v1 && python _enrich_all_slots.py`
- 看「可下單 picks (filter 漲停後)」section,優先 ATR < 3% + N≥2 連續
- 若可下單 = 0,看 `python _extended_picks.py <slot> 30` 找 top-30 中的 actionable