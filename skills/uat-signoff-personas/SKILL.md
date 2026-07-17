---
name: uat-signoff-personas
description: เลเยอร์ UAT sign-off ครอบ functional-qa-personas — สัมภาษณ์ทิศทางทีละคำถาม เก็บ acceptance criteria จาก user โดยตรง, generate role-scoped personas + จับคู่ criteria↔role, ส่งให้ functional-qa-personas เป็น execution engine, แล้วสังเคราะห์โหวต Pass/Fail/Conditional ต่อ criteria เป็นรายงาน sign-off ทางการ. Use เมื่อ user ต้องการ UAT sign-off / user acceptance test แบบมี acceptance criteria เป็นข้อ ๆ ที่ต้องโหวตผ่าน-ตก — ถ้าโจทย์แค่หา bug เชิง functional โดยไม่ต้องการ sign-off ทางการ ให้ใช้ functional-qa-personas ตรง ๆ แทน.
---

# UAT Sign-off ด้วย Role-scoped Personas

เลเยอร์ **UAT sign-off** ที่ครอบ skill `functional-qa-personas` — ใช้ตัวนั้นเป็น **execution engine เท่านั้น** (ขั้นรัน persona ขับ Playwright + gotchas ทั้งหมดของมัน) แต่**ไม่ใช้ persona-generation ของมัน** เพราะ UAT ต้องการ persona ที่ผูกกับ role จริงใน scope และผูกกับ acceptance criteria ที่ user กำหนด ไม่ใช่กลุ่ม scenario ที่ skill นั้นคิดเอง

ผลลัพธ์สุดท้าย: รายงาน sign-off ที่ทุก acceptance criteria มีโหวต Pass / Fail / Conditional จาก persona ใน role ที่เกี่ยวข้อง พร้อมเหตุผล

## ขั้น 1 — สัมภาษณ์ทิศทาง (ทีละคำถาม รอ feedback)

เขียน interview flow เองในขั้นนี้ — ถามทีละคำถาม พร้อมคำตอบแนะนำ (suggested answers) ให้เลือก แล้ว**รอ feedback ก่อนถามข้อถัดไป** (แนวคิด branch-by-branch ยืมมาจาก skill `grilling` แต่**ไม่ chain ไปเรียก skill นั้นจริง** — ทำ flow เองในตัว)

สิ่งที่ต้องเก็บให้ครบก่อนจบขั้น 1:

1. **Scope / flow ที่จะทดสอบ** — ระบบไหน หน้าไหน ธุรกรรมอะไรบ้าง จุดเริ่ม-จุดจบของ flow
2. **Acceptance criteria เป็นข้อ ๆ** — ให้ user **พิมพ์เองตรง ๆ** เท่านั้น ห้ามพยายามดึงอัตโนมัติจาก Plane / ticket / เครื่องมือใด ๆ (ถ้า user อยาก copy มาจากที่อื่นก็ให้ user เป็นคน paste เอง)
3. **จำนวนคนรวม** (persona ทั้งหมดที่จะรัน)
4. **Path เก็บรายงาน sign-off** — ไฟล์ปลายทางที่ user ระบุ

### เช็คพอยต์เตือนล่วงหน้า 2 จุด (บังคับ — ต้องเตือนตั้งแต่ขั้นสัมภาษณ์ ไม่ใช่ disclaimer ท้ายงาน)

- **(ก) ถ้า scope มีการอนุมัติข้าม role** (requester ≠ approver เช่น เซลส์ขอส่วนลด → ผู้จัดการอนุมัติ) — เตือนข้อจำกัด single-account ของ `functional-qa-personas` ทันที: **ทุก persona login ด้วย JWT บัญชีเดียวกันจริง** การแบ่ง role เป็นแค่ roleplay เชิง UI/พฤติกรรม **ไม่ใช่การทดสอบสิทธิ์ (permission) จริง** — criteria ที่ต้องพิสูจน์ว่า "role อื่นทำไม่ได้" จะพิสูจน์ไม่ได้ด้วยวิธีนี้ ให้ user รับทราบและตัดสินใจว่าจะ (1) รับข้อจำกัดแล้วไปต่อ (2) ตัด criteria นั้นออก หรือ (3) จัดบัญชีจริงหลายบัญชีมาก่อน
- **(ข) ถ้าจำนวนคนรวมน้อยกว่าจำนวน role ที่จำเป็น** — เตือนแล้วให้ user เลือก: เพิ่มจำนวนคน หรือตัด role ที่สำคัญน้อยสุดออก (บอกให้ชัดว่า role ไหนจะไม่มีคนโหวต criteria ไหนบ้าง)

## ขั้น 2 — วิเคราะห์ scope → generate persona + จับคู่ criteria → ยืนยันก่อนรัน

ทำเองทั้งหมดในขั้นนี้ (นี่คือส่วนที่**แทนที่** persona-generation ของ functional-qa-personas):

1. **วิเคราะห์ scope/flow** จากขั้น 1 → ระบุ role ที่มีอยู่จริงใน flow (เช่น เซลส์, ผู้จัดการฝ่ายขาย, บัญชี, คลัง)
2. **Generate role-scoped personas** — แต่ละ persona ผูกกับ role เดียว มีบุคลิก/ความชำนาญต่างกันได้ แต่ mission ต้องสมเหตุสมผลกับสิ่งที่ role นั้นทำจริงใน flow (ไม่ใช่สุ่ม persona ทั่วไป)
3. **จับคู่ criteria ↔ role** — แต่ละ acceptance criteria กำหนดว่า role ไหน**เกี่ยวข้อง**บ้าง เฉพาะ persona ใน role ที่เกี่ยวข้องเท่านั้นที่จะโหวต criteria นั้น (role ที่ไม่เกี่ยวไม่ต้องโหวต — กันโหวตมั่วจากคนที่ไม่ได้ใช้ feature นั้น)
4. **แจกจ่ายคนลง role** — กติกา: กระจายคนละ 1 role ให้**ครบทุก role ก่อน** แล้วค่อยเพิ่มคนซ้ำใน role ที่**สำคัญสุดต่อ criteria** (role ที่ถูกจับคู่กับ criteria มากสุด/วิกฤตสุด)
5. **โชว์ตารางสรุปให้ user ยืนยันก่อนรัน** — ตารางต้องเห็น: persona × role × mission ย่อ × criteria ที่ตัวเองจะโหวต — **ห้ามเริ่ม execution จนกว่า user จะยืนยัน**

## ขั้น 3 — Execution ผ่าน functional-qa-personas

ส่ง **persona list ที่ generate แล้วจากขั้น 2** ให้ `functional-qa-personas` ทำหน้าที่ execution เท่านั้น:

- ใช้กลไกของมันทั้งหมด: `00-mission-control.md` (login bootstrap, ตาราง path ที่ verify แล้ว, กติกาโดเมน, E2E gotchas), การ spawn Haiku ขับ Playwright, รูปแบบรายงานบังคับ
- **ห้ามให้มัน generate persona ซ้ำ** — persona/กลุ่ม มาจากขั้น 2 เท่านั้น
- **เพิ่มในรูปแบบรายงานต่อ persona**: นอกจากรายงานมาตรฐานของ functional-qa-personas แล้ว ต้องมี **self-vote ต่อ criteria** — เฉพาะ criteria ที่ role ตัวเองถูกจับคู่ไว้ โหวต `Pass` / `Fail` / `Conditional` พร้อม**เหตุผลจากสิ่งที่ทำ/เห็นจริง** (Conditional = ผ่านแบบมีเงื่อนไข ต้องระบุเงื่อนไขชัด)

## ขั้น 4 — สังเคราะห์เป็นรายงาน sign-off

1. **รวมโหวตเป็นตารางต่อ criteria** — criteria × (โหวตจากแต่ละ persona ใน role ที่เกี่ยวข้อง) × ผลรวม (Pass ทั้งหมด / มี Fail / มี Conditional) พร้อมสรุปเหตุผลหลัก
2. **Cross-reference ข้ามกลุ่ม/ข้าม role ก่อนสรุป** — ใช้เกณฑ์เดียวกับขั้นสังเคราะห์ของ functional-qa-personas: แยกทุกปัญหา/Fail เป็น (ก) **บั๊กจริงน่าเชื่อถือสูง** (หลาย persona อิสระเจอตรงกัน) (ข) **น่าสงสัยต้องตรวจเพิ่ม** (อาจเป็น selector ผิดหรือ shared-account artifact) (ค) **known test-harness artifact** ไม่ใช่บั๊กแอป — Fail ที่เป็นกลุ่ม (ค) ไม่ควรทำให้ criteria ตก แต่ต้อง note ไว้
3. **เขียนไฟล์รายงานไปที่ path ที่ user ระบุ**ในขั้น 1 ครอบ: ตารางโหวตต่อ criteria, รายละเอียด Fail/Conditional พร้อมการจำแนก (ก)/(ข)/(ค), ข้อจำกัดของรอบทดสอบ (รวม single-account), และข้อสรุป sign-off รวม

## หมายเหตุ

- **ข้อจำกัด single-account ต้องเตือนล่วงหน้าเสมอ** (เช็คพอยต์ ก ในขั้น 1) — ไม่ใช่เพิ่งมาเขียนเป็น disclaimer ตอนท้ายรายงาน user ต้องรู้ก่อนตัดสินใจรัน
- **เป็นเครื่องมือกลาง ๆ** — ไม่ผูกกับ Plane หรือเครื่องมือ tracking เฉพาะใด ทุก input (scope, criteria, จำนวนคน, path รายงาน) มาจาก user พิมพ์ตรง ๆ ในขั้นสัมภาษณ์เท่านั้น
