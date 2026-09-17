# FT420 Quant Research Lab (`ft420_lab`)

ระบบวิจัยกลยุทธ์เทรด Forex/XAUUSD แบบ Quant + ML (ไม่ใช้ LLM ตัดสิน BUY/SELL ที่ runtime)
ดู `LAB_RULES.md` (กฎบังคับ) และ `AGENTS.md` (ทีม agent 8 ตำแหน่ง) ก่อนเริ่มงานทุกครั้ง

สถานะโปรเจกต์: **แยกอิสระจาก `quant_system/` เดิมเต็มรูปแบบ** (ดู `PROJECT_AUDIT.md`)

## Pipeline

```
Market Data -> Data Validation -> Feature Library -> Market Regime -> Strategy Candidate Generator
  -> Experiment Runner -> Backtest -> Walk Forward -> Cost/Spread/Slippage Stress Test -> Ablation
  -> Robustness Ranking -> Reject/Incubate/Promote -> Strategy Registry -> FT420 Scanner/Forward Report
```

## Quick Start

```bash
python -m cli.lab doctor
python -m cli.research baseline --symbol XAUUSD --timeframe H1
python -m cli.compare --experiment exp_001 exp_002
python -m cli.promote --candidate candidate_001
```

## โครงสร้าง

ดูสารบัญเต็มใน `PROJECT_AUDIT.md` ส่วน "Execution / Data Flow" — สรุปสั้น:

- `contracts/` — dataclass/schema กลางที่ทุกโมดูลต้องใช้ร่วมกัน
- `data/` — adapter (CSV/MT5/Synthetic), data quality gate, dataset catalog
- `features/` — feature library แยกตามหมวด (technical/structure/volatility/session/price_action/ml)
- `regimes/` — market regime classifier (deterministic ก่อน, HMM ทีหลัง)
- `strategies/` — baseline (deterministic, ไม่มี ML) และ candidate (config-driven)
- `models/` — ML models (xgboost/lightgbm) + optional (qlib/darts/chronos)
- `engine/` — backtest simulator, cost model, position, metrics
- `validation/` — temporal split, walk-forward, stress test, ablation, sensitivity, leakage check
- `promotion/` — scorer, gates, registry writer
- `experiments/` — experiment definitions/runs/artifacts/leaderboard (generated, git-ignore ได้)
- `skills/` — SKILL.md แบบ reusable instruction ต่อ capability
- `agents/` — นิยามบทบาท agent 8 ตัว
- `cli/` — command line entrypoints

## Definition of Done

ดู `reports/prompt1/FINAL_REPORT.md` (จะสร้างเมื่อ checklist ครบ) — สถานะปัจจุบันของแต่ละ phase อยู่ใน `ประวัติการทำงาน.md`
