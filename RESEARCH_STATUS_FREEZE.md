# RESEARCH_STATUS_FREEZE.md

วันที่ freeze: 2026-09-14 · ผู้ตัดสินใจ: ผู้ใช้ (หลังอ่าน Prompt 1 + Prompt 1.5 ผลครบ)

## การตัดสินใจ

หยุดไล่หา Research Family ใหม่ชั่วคราว — freeze ผลวิจัยทั้งหมดตามตารางด้านล่าง แล้วเปิดสอง track
คู่กัน: **(1) Lead-Lag Forward Incubation** (เบื้องหลัง, ไม่ต้อง active effort ต่อเนื่อง) และ
**(2) Platform Full Audit/Stabilization** (Codex รับผิดชอบ — runtime/production layer ของ FT420 ที่
ตอนนี้เป็นคอขวดจริงมากกว่า Research Engine)

---

## Frozen Research State

**PROMOTED: 0**

## Prompt 1.7 addendum (2026-09-15)

R7/R8/R9 were audited without changing any frozen Lead-Lag files. R9 clean-room
FVG/iFVG/ORB/AMD candidates are rejected on XAUUSD M15; FVG also rejects on
EURUSD/GBPUSD. R7 remains data-unavailable pending native M5. A separate R8
clean-room London breakout + deterministic regime gate is **INCUBATING** on
XAUUSD H1 only; it is not a modification of Lead-Lag and needs cross-market and
untouched confirmation before any promotion discussion.

**INCUBATING:**

1. `correlation_lead_lag_filter` (Part A candidate C)
   - median Δ expectancy = **+0.0195** (ดีที่สุดในทั้งโปรเจกต์ รวม Prompt 1)
   - gate = **7/8**
   - blocker = **pooled bootstrap 90% CI ยังแตะขอบศูนย์** ([-0.0107, +0.1667]) — ไม่ใช่ threshold
     instability (threshold stability **ผ่าน**จริง range=0.15 ≤ 0.20) ดูรายละเอียดที่
  `reports/prompt1_5/CORRELATION_REPORT.md` หัวข้อ 4
   - **สถานะ: `INCUBATING_FORWARD`** — freeze แล้ว, ดูหัวข้อ "Lead-Lag Forward Incubation" ด้านล่าง

2. `structure_breakout_vol_below` (Prompt 1)
   - median Δ expectancy = **+0.0043**
   - bootstrap CI crosses 0 ([-0.101, +0.125])
   - status = **WEAK EVIDENCE** (`INCUBATING_WEAK_EVIDENCE` ตามที่ตั้งไว้ใน Prompt 1) — ไม่มีแผน
     forward-incubate ตัวนี้เป็นพิเศษตอนนี้ (สัญญาณอ่อนกว่า Lead-Lag ชัดเจน)

**REJECTED:**
- directional baselines (6 ตระกูล, Prompt 1)
- ML filters (Prompt 1)
- directional forecasts (Prompt 1)
- correlation filters ส่วนใหญ่ (A, B ของ Part A — Prompt 1.5)
- Supply/Demand baselines ทั้งหมด (B1-B5 — Prompt 1.5)
- pairs/stat-arb variants ทั้งหมด (static/rolling/Kalman — Prompt 1.5)
- triangular arbitrage ทั้งหมด (3 triangle — Prompt 1.5)

---

## Lead-Lag Forward Incubation — วิธีทำงาน

**หลักการ:** ห้าม retune threshold จากข้อมูล OOS เดิมที่ใช้ตัดสินไปแล้ว (จะกลายเป็น tuning-on-OOS)
— freeze code ของ candidate ตรงจุดที่ validate แล้ว แล้วให้ **ข้อมูลอนาคตจริง** (bar ที่ยังไม่เกิด ณ
วันที่ freeze) เป็นตัวตัดสินแทน

```
Candidate v1 frozen (2026-09-14)
      ↓
ไม่มี retrain/tune จาก forward result  ← บังคับด้วย sha256 hash check, ไม่ใช่แค่สัญญา
      ↓
เก็บสัญญาณ forward (re-run เป็นระยะ เช่นทุกเดือน)
      ↓
100+ opportunities หรือช่วงเวลาที่กำหนด
      ↓
ตรวจ: expectancy / threshold stability / long-short balance / regime stability / cost robustness
      ↓
PROMOTE / REJECT (คนตัดสิน ไม่ auto-decide)
```

**Infrastructure ที่สร้างไว้:**

- `experiments/forward_tracking/lead_lag_v1/frozen_manifest.json` — sha256 hash ของไฟล์ทั้ง 11 ที่
  ผลของ candidate นี้ขึ้นกับ (`cli/research_correlation.py`, `research_families/correlation/
  features.py`, `models/common/{feature_walkforward,ml_walkforward}.py`,
  `contracts/cross_asset.py`, `engine/{simulator,costs,position,metrics}.py`,
  `strategies/baselines/structure_breakout.py`, `features/factory.py`)
- `cli/lead_lag_forward_track.py` — สคริปต์ที่ต้อง**รันซ้ำเป็นระยะ** (แนะนำทุกเดือน หรือถี่กว่านั้นถ้า
  สะดวก): (1) hash-verify ไฟล์ frozen ทั้งหมดก่อน — **ปฏิเสธรันทันทีถ้ามีไฟล์ไหนถูกแก้** (ยืนยันแล้ว
  ด้วย tamper-test จริงว่าปฏิเสธได้ถูกต้อง) (2) ดึงข้อมูล MT5 สดใหม่ (read-only) (3) รัน pipeline
  เดิมที่ freeze ไว้ซ้ำ (4) เก็บเฉพาะ fold ที่ test_window เริ่มตั้งแต่ 2026-09-14 เป็นต้นไปเข้า
  forward log (ไม่นับ fold เดิมที่ใช้ตัดสิน 7/8 ไปแล้วซ้ำ) (5) พิมพ์ judge report เทียบ 5 เกณฑ์
- `experiments/forward_tracking/lead_lag_v1/forward_log.json` — สะสมผล forward fold จริง
  (ปัจจุบันว่างเปล่า — ยังไม่มี bar ใหม่เกิดขึ้นหลัง freeze date เลย เพราะเพิ่ง freeze วันนี้)

**สถานะปัจจุบัน (2026-09-14, วันที่ freeze):** ยืนยันแล้วว่า mechanism ทำงานถูกต้องครบวงจร (init
freeze → tamper-test ปฏิเสธได้จริง → restore แล้วรันสำเร็จ → ตรวจพบว่ายังไม่มี forward data ใหม่
อย่างถูกต้อง) — 0/100 forward opportunities สะสม รอเวลาจริงผ่านไปก่อนจะมี fold ใหม่ให้เก็บ

**การตัดสินใจในอนาคต:** เมื่อสะสมได้ ≥100 forward opportunities (คาดว่าประมาณ 1-2 fold ใหม่ ~3-6
เดือนข้างหน้า จากอัตรา ~90-120 filtered trade ต่อ fold เดิม) ให้ตรวจ 5 เกณฑ์ตาม judge report แล้ว
คนตัดสิน PROMOTE/REJECT — สคริปต์ไม่ auto-decide

---

## Platform Full Audit / Stabilization

มอบหมายให้ **Codex** ทำงานคู่ขนาน (นอกขอบเขตของ session นี้) — ประเด็นที่ระบุไว้: Mobile/PC state ไม่
ตรงกัน, chart ไม่ realtime, signal วน, popup ไม่เด้ง — ครอบคลุม runtime/production layer ของ FT420
(อยู่ที่ `aINEWWORLD/web`, แยกจาก `ReLab` โดยสิ้นเชิง) ไม่ใช่ขอบเขตของ Research Lab นี้

---

## Roadmap

```
Prompt 1       ✅
Prompt 1.5     ✅
      ↓
Freeze research results   ✅ (เอกสารนี้)
      ↓
Lead-Lag Forward Incubation ───────────┐
                                       │ ทำเบื้องหลัง
Platform Full Audit / Realtime Fix  ←──┘  (Codex)
      ↓
Mobile/Desktop sync, Chart realtime, Signal state machine, Popup/alerts
      ↓
Prompt 2 — Skill / Agent System
```
