# LAB_RULES.md — กฎบังคับของ FT420 Quant Research Lab

กฎเหล่านี้ผูกพันทุก skill, agent, และโค้ดใน `ft420_lab`. ฝ่าฝืน = experiment ไม่ valid

## 1. Reproducibility (ห้ามละเมิด)

ทุก experiment ต้องบันทึกครบ: strategy_id, strategy_version, dataset_version, symbol, timeframe,
ช่วงข้อมูล, train/validation/test period, feature set, parameters, random seed, spread/commission/slippage
assumption, code hash (ถ้ามี git), metrics, artifacts, เหตุผล reject/promote

**Config เดิม + data เดิม + seed เดิม ต้องรันซ้ำได้ผลเดิมเป๊ะ** — ห้ามมี non-determinism ที่ไม่ seed
(เช่น `random`, `numpy.random`, model init ต้องรับ seed explicit เสมอ)

## 2. Anti-Overfit (ห้ามละเมิด)

- ห้าม rank strategy จากกำไร in-sample อย่างเดียว
- Time-series split เท่านั้น ห้าม shuffle train/test
- ห้าม future leakage: indicator ที่ index T ห้ามใช้ข้อมูลแท่ง T ที่ยังไม่ปิด (`shift(1)` เสมอ)
- เข้าไม้ที่ปิดแท่ง j → เริ่มเช็ค SL/TP ที่แท่ง j+1 เท่านั้น (ห้าม fill ในแท่งเดียวกับสัญญาณ)
- Spread/commission สมจริง, slippage ปรับค่าได้ (เป็นสัดส่วนของ spread ไม่ใช่ค่าคงที่ — เพื่อไม่พังเวลาข้ามตลาด)
- Walk-forward บังคับ, ต้องมี minimum trade sample ก่อนเชื่อผล
- Parameter sensitivity, regime breakdown, ผลรายปี/รายช่วง ต้องรายงานเสมอ
- Ablation บังคับสำหรับ ML/features หลายตัว
- **Final test set ต้อง lock — ห้าม optimize จาก final test set**

## 3. ตัวเลขที่ต้องรู้คำตอบล่วงหน้า (ใช้จับบั๊กในโค้ดเอง)

- Barrier สมมาตร (±1 ATR) → base rate ต้อง ~50%
- ซื้อมั่ว/ขายมั่ว → ต้องติดลบประมาณค่าธรรมเนียม (ต้นทุน)
- สุ่มทิศ → ต้องได้ประมาณ -cost/R
- ถ้าตัวเลขเหล่านี้เพี้ยน = มีบั๊กในโค้ด **ห้ามอ่านผลลัพธ์ต่อจนกว่าจะแก้**

## 4. กลุ่มควบคุม (Control Group) — บังคับตั้งแต่ต้น

ทุก experiment ต้องมีกลุ่มควบคุมคู่กัน: กลับข้างสัญญาณ, สุ่มทิศ (≥2 seed), ซื้อล้วน, ขายล้วน
กลุ่มควบคุมที่ใช้โค้ด engine ชุดเดียวกันจับบั๊กใน engine เองไม่ได้ — ต้องระวังเวลาตีความ
เวลากลับข้าง/สุ่มทิศ SL ต้องอยู่ฝั่งขาดทุนของทิศใหม่เสมอ (ไม่ใช่ทิศเดิม)

## 5. สถิติ

- ไม้ต้องไม่ทับกัน (overlapping trades ทำให้ n พองและนัยสำคัญพองตาม)
- Bootstrap CI (90% เป็นค่าเริ่มต้น), ตัดไม้กำไรสูงสุด 5% แล้วดูว่ายังบวกไหม
- แบ่งครึ่งเวลา (first half / second half) ต้องบวกทั้งสองครึ่ง (หรืออธิบายได้ว่าทำไมไม่)
- นับจำนวนสมมติฐานที่ทดสอบ แก้ Bonferroni หรือรายงาน multiple-comparison แบบตรงไปตรงมา
- ใช้หน่วย R เวลาเทียบข้ามตลาด/สินทรัพย์ (หน่วย $ ไม่ยุติธรรม)

## 6. LLM/AI Boundary

- LLM/AI ห้ามเป็นตัวตัดสิน BUY/SELL ที่ runtime
- LLM ใช้ได้เฉพาะ: นักวิจัย, ผู้ช่วยพัฒนา, ผู้ช่วยวิเคราะห์ผล, ผู้ช่วยสร้าง hypothesis
- การตัดสินเทรดจริงต้องมาจาก deterministic logic หรือ trained model ที่ reproduce ได้เท่านั้น

## 7. External Repo Study

ห้าม clone แล้ว copy โค้ดตรง ๆ ทุก repo อ้างอิงต้องผ่าน: อ่าน architecture, แยก reuse ได้, หา assumption,
ตรวจ license, ตรวจ look-ahead/leakage, ตรวจ train/live gap, ดึงเฉพาะ concept, adapt เข้า FT420 contract
บันทึกเป็น `repo_reports/<repo>.md`

## 8. Promotion Discipline

- Candidate ที่ fail ห้าม promote ไม่ว่ากรณีใด
- Feature ที่ไม่มี contribution (ablation แสดงว่าลบออกไม่กระทบ) → เอาออก
- ML ที่ไม่ชนะ baseline → อย่าใช้ ML
- Forecast model (Chronos/Darts) ที่ไม่ช่วย OOS อย่างเสถียร → disable
- RL ที่ไม่ stable (multi-seed OOS) → ห้ามเปิด production

## 9. Safety Boundary เฉพาะเครื่องนี้ (เพิ่มจาก audit)

- ห้ามเรียก `MetaTrader5.initialize()` / login บัญชีจริงโดยไม่ confirm กับผู้ใช้ก่อนทุกครั้ง — เครื่องนี้มี
  MT5 terminal เดียว การสลับบัญชีเสี่ยงกระทบ EA/บัญชีจริงที่รันอยู่
- ห้ามแก้ไฟล์ใน `../web/` (FT420 Scanner bundle) โดยตรง — ส่งออกเฉพาะผ่าน FT420 Integration Contract
- ห้าม import หรือพึ่งพาโค้ด/ข้อมูลจาก `../../quant_system/` (ตัดสินใจแยกอิสระ 2026-09-13)

## 10. Failure Policy

- Test fail → แก้ก่อน ไปต่อไม่ได้
- Experiment fail → บันทึก failure ตรงไปตรงมา ห้ามพยายามทำให้ผลดูดี
- Strategy ไม่มี edge → REJECT พร้อมเหตุผล ห้ามปั้นเลข
