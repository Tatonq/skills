---
name: qa-debate-fix-pipeline
description: Pipeline ครบวงจรสำหรับ QA + brainstorm + implement รอบใหญ่บนระบบทั้งก้อน — multi-persona functional QA → synthesis เป็น workorder → 3-model debate ring → verify-from-source (Phase 0) → phased implementation → manual QA จริง → ship. Use เมื่อโจทย์คือ "ทดสอบ/ยกเครื่องระบบ X ทั้งระบบแล้วแก้ให้จบ" ที่ต้องทั้งหา bug, จัดลำดับ, และลงมือแก้ในรอบเดียว — ไม่คุ้มกับ feature เดี่ยวหรือ bug เดี่ยว (ใช้ skill ย่อยตรง ๆ แทน). พิสูจน์จริงกับระบบ SOP 2026-07-11 (Phase 1 ship เข้า dev @ac5e08c2)
---

# QA → Debate → Fix Pipeline

Meta-pipeline ประกอบจาก skill ย่อยที่มีอยู่แล้ว — ตัวนี้กำหนด **ลำดับ, จุดส่งต่อ, และกติกาที่ทำให้ไม่หลงทาง**. เหมาะกับงานสเกล "ทั้งระบบ" ที่จบใน 1-2 วัน

## ลำดับ 6 ขั้น

1. **Multi-persona functional QA** — ใช้ skill `functional-qa-personas` (หลายกลุ่ม persona ขับ Playwright ทำธุรกรรมจริงจนจบ business flow บน dev server) → ได้ raw reports ต่อกลุ่ม
2. **Synthesis → workorder** — agent สังเคราะห์ (Opus ขึ้นไป) รวม raw reports เป็น `*-WORKORDER.md`: ปัญหา P1-Pn จัดกลุ่ม พร้อม**กรองแยก** สิ่งที่ไม่ใช่บั๊กจริง (permission artifact ของ test user, E2E-gotcha, สภาพแวดล้อม) ออกจากบั๊ก product
3. **3-model debate ring** — ใช้ skill `multi-model-debate` (Sonnet/Opus/Fable เขียนแผนอิสระ → วิจารณ์วงแหวน → revise → synthesize) บน workorder → `plan-final.md`
4. **Phase 0: verify-from-source** — ก่อน implement ตอบทุก unknown ในแผนด้วยการ**อ่านซอร์สจริง ไม่เดา** (รอบ SOP: 4 unknowns ใน 45 นาที) — กติกา "investigate-then-rank": ห้ามจัด priority สุดท้ายก่อน verify สมมติฐาน (pattern นี้โผล่อิสระ 2 ครั้งใน debate — เป็น insight ที่แรงที่สุดของวิธีนี้)
5. **Phased implementation** — team lead แจกงานให้ expert agents ขนานกัน ทีละ phase (อย่ารวม P ทั้งหมดเป็นก้อนเดียว — ship Phase 1 ที่ชัวร์ก่อน)
6. **Manual QA จริงก่อน commit** — เปิด browser จริงยืนยันทุกจุดแก้ (skill `playwright-manual-verification`) แล้วค่อย commit+push

## กติกาส่งต่อระหว่างขั้น

- ขั้น 1→2: raw reports ต้องแนบ evidence (screenshot/URL/doc no.) — ไม่งั้น synthesis แยกบั๊กจริงจาก artifact ไม่ได้
- ขั้น 2→3: debate รับ **workorder ที่กรองแล้ว** ไม่ใช่ raw reports (ไม่งั้นเถียงกันเรื่อง noise)
- ขั้น 4→5: unknown ที่ verify ไม่ได้ = ตัดออกจาก phase ปัจจุบัน ไม่ใช่เดาต่อ
- งาน queue: โฟลเดอร์ debate อยู่ `~/.agent-monitor/knowledge/debates/<job-id>/` แยกจาก `queue/` (ดู skill `agent-monitor-queue`)

## Gotchas จากรอบจริง (2026-07-11)

1. **Agent auto-commit เองโดยไม่มีคำสั่ง** — เจอแล้วให้ supersede ด้วย commit ใหม่ ห้าม amend/rebase ทับ
2. **Background agent ที่รายงาน "completed" อาจยังรันต่อหลายชั่วโมง** แล้วเขียนทับไฟล์ทีหลัง — ก่อนใช้ไฟล์ผลลัพธ์ เช็ค mtime ว่านิ่งแล้ว; งานสำคัญให้ pin เนื้อหาที่จะใช้ออกมาก่อน
3. **Synthesis agent ค้างไม่มี notification ได้** — kill แล้วใช้ draft ที่มันทิ้งไว้ (มักเกือบสมบูรณ์) แทนการรันใหม่ทั้งรอบ
4. รายงาน "เสร็จ/merge แล้ว" จาก agent เชิงกลไก ต้อง verify กับ git history เองเสมอ

## เมื่อไหร่ไม่ควรใช้

- bug เดี่ยว/feature เดี่ยว → `debug-mantra` หรือทำตรง ๆ
- ต้องการแค่หา bug ไม่ต้องแก้ → `functional-qa-personas` เดี่ยว ๆ
- ต้องการ UAT sign-off ทางการมี acceptance criteria → `uat-signoff-personas`
- คำถาม architecture ที่ไม่ต้องมี QA นำ → `multi-model-debate` เดี่ยว ๆ
