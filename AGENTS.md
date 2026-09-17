# AGENTS.md — ทีม Agent ของ FT420 Quant Research Lab

นิยามเต็มของแต่ละ agent อยู่ใน `agents/<name>.md` ไฟล์นี้เป็น index + กติกาการส่งงานต่อกัน

| # | Agent | ไฟล์ | หน้าที่หลัก | มีสิทธิ์ veto? |
|---|---|---|---|---|
| 1 | Research Director | `agents/research_director.md` | คุม research workflow, แจกงาน, กัน leakage/ซ้ำ | ปฏิเสธ experiment ผิดกติกา |
| 2 | Repo Analyst | `agents/repo_analyst.md` | ศึกษา external repo → `repo_reports/<repo>.md` | ไม่ |
| 3 | Data Engineer | `agents/data_engineer.md` | data import/validate/version → DataQualityReport | บล็อก experiment ถ้า data fail |
| 4 | Quant Researcher | `agents/quant_researcher.md` | สร้าง deterministic baseline 6 ตระกูล | ไม่ |
| 5 | ML Researcher | `agents/ml_researcher.md` | feature matrix, model, temporal CV, ablation | ไม่ |
| 6 | Validation Agent | `agents/validation_agent.md` | walk-forward, stress test, sensitivity | **ใช่ (veto promotion)** |
| 7 | Forensic Agent | `agents/forensic_agent.md` | วิเคราะห์ว่าทำไมชนะ/แพ้ → hypothesis ใหม่ | ไม่ (ห้ามแก้ strategy ตรง) |
| 8 | Release Agent | `agents/release_agent.md` | freeze + export candidate ที่ผ่านแล้ว | ไม่ (รับเฉพาะที่ผ่าน validation) |

## ลำดับการส่งงาน (handoff order)

```
Research Director -> เลือกโจทย์ -> Data Engineer (ตรวจ data ก่อนเสมอ)
  -> Quant Researcher (baseline) หรือ ML Researcher (ML candidate)
  -> Validation Agent (walk-forward/stress/ablation, มีสิทธิ์ veto)
  -> ถ้า fail -> Forensic Agent (หา hypothesis ใหม่) -> กลับ Research Director
  -> ถ้า pass -> Release Agent (freeze + export) -> Strategy Registry
```

## กติการ่วม

- ทุก agent ต้องอ้างอิง `LAB_RULES.md` เสมอ ห้ามเลี่ยงกฎ anti-overfit/reproducibility
- ห้าม agent ใดปรับ metric ให้ดูดีขึ้นหลังเห็นผล — ต้องรายงานตามจริง
- Research Director และ Validation Agent ต้องเป็นอิสระจาก Quant/ML Researcher (คนละมุมมอง แม้จะรันโดย agent เดียวกันในทางเทคนิค ก็ต้องแยก step และ log แยก)
