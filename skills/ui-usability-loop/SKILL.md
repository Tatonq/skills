---
name: ui-usability-loop
description: Workflow มาตรฐานสำหรับงาน UI ทุกชิ้น — ออกแบบด้วย frontend-ux-specialist ก่อนโค้ด, โค้ด+e2e, จำลองผู้ใช้ 10 คนด้วย Haiku ขับเว็บจริงผ่าน Playwright, แล้วบันทึกผลเป็น usability report. Use เมื่อเริ่มงาน UI ใหม่ (หน้าใหม่, feature ที่มีหน้าจอ, redesign) หรือเมื่อต้องทดสอบ usability ของหน้าที่มีอยู่. พิสูจน์จริงกับโปรเจกต์ DO Review T5 (2026-07-11).
---

# UI Usability Loop

Workflow บังคับ 4 ขั้นสำหรับงาน UI ทุกชิ้น: **ออกแบบ → โค้ด+e2e → จำลองผู้ใช้ 10 คน → บันทึกผล** — ไม่ผ่านเกณฑ์ = วนแก้แล้วทดสอบใหม่

> Skill นี้วัด **usability** (หาปุ่มเจอไหม เข้าใจง่ายไหม) — ถ้าโจทย์คือทดสอบว่า **business flow ทำงานถูกจริง end-to-end** (ทำธุรกรรมจริงจนจบ เช่น ขอส่วนลด/อนุมัติ/credit holding) ให้ใช้ skill `functional-qa-personas` แทน

## ขั้น 1 — ออกแบบก่อนโค้ด (frontend-ux-specialist)

Spawn sub-agent `frontend-ux-specialist` ให้ผลิต design spec ก่อนเขียนโค้ดเสมอ เป้าหมายเดียว: **ใช้งานง่าย เข้าใจโดยไม่ต้องอธิบาย** (self-explanatory)

Brief ต้องสั่งให้ spec ครอบ:

1. Layout + component tree
2. ข้อความ label ภาษาไทย**ทุกจุด** (ไม่ปล่อยให้ dev คิดเอง)
3. ระบบ chip/สี พร้อม**เหตุผล** และ accessibility (contrast, colorblind)
4. Interaction + states ครบ: loading / empty / error
5. จุดเผื่อ feature ถัดไป (extension points)
6. **โจทย์ usability task 5-8 ข้อ** — ผลิตจาก spec เลย จะถูกใช้ตรง ๆ ในขั้น 3

## ขั้น 2 — โค้ด + e2e

- **TDD ฝั่ง API ก่อน** (httptest) → แล้วค่อยทำ UI ตาม spec
- Playwright e2e เฉพาะ **flow หลัก** — ไม่ unit-test component
- ก่อนถือว่าเสร็จ: **ตรวจสายตาด้วย screenshot ผ่าน Playwright** เสมอ (โค้ดผ่าน test ไม่ได้แปลว่าหน้าตาถูก)

Gotcha e2e: หน้า list ที่มี skeleton loading — selector ต้อง `tr:not(.skeleton-row)` กันคลิกโดนแถว skeleton

## ขั้น 3 — Haiku จำลองผู้ใช้ 10 คน

Spawn Agent 10 ตัว `model: haiku` persona ต่างกัน (รันคู่ขนานได้) ชุดที่ใช้จริง:

| # | Persona |
|---|---------|
| 1 | พนักงานคลังไม่ถนัดคอม อายุ 52 |
| 2 | เด็กฝึกงาน 24 |
| 3 | หัวหน้ารีบ ๆ 45 |
| 4 | บัญชีละเอียด 38 |
| 5 | เซลส์สายพิมพ์ค้น 29 |
| 6 | ผู้สูงวัยสายตาไม่ดี 58 |
| 7 | คนขับรถ ไม่รู้ศัพท์ ERP |
| 8 | IT support ไม่รู้ business |
| 9 | ผู้จัดการสาย Excel |
| 10 | พนักงานใหม่วันแรก |

กติกาต่อ agent:

- ได้ **URL + โจทย์ task** (จากขั้น 1) — **ไม่มีคำอธิบายวิธีใช้** ใด ๆ
- ขับเว็บจริงผ่าน Playwright: เขียน script `.mjs` แล้วรันจาก dir ที่มี `@playwright/test` ติดตั้งอยู่
- ตัดสินใจจาก **screenshot เท่านั้น** — **ห้าม**อ่าน source code, **ห้าม**ยิง API ตรง
- **กำชับ path ของ script ใน prompt ให้ชัด** — Haiku บางตัวเขียน script ทิ้งนอก dir ที่สั่ง (เช่นลงใน `web/` ของโปรเจกต์) → ตรวจ `git status` กวาดไฟล์แปลกก่อน commit เสมอ
- **script อยู่นอก dir ที่มี `node_modules` (เช่น scratchpad) จะ `import '@playwright/test'` ไม่เจอ** — ESM resolve จากตำแหน่งไฟล์ ไม่ใช่ cwd. ทางแก้: สั่งให้เขียนเป็นไฟล์ `.spec.js` แบบ Playwright test จริง แล้วรันด้วย `npx playwright test <absolute-path-นอก-testDir> --config=<path>/playwright.config.js` จาก dir ของโปรเจกต์ — Playwright ยอมรับ path ไฟล์ทดสอบนอก `testDir` ได้ปกติ ไม่ต้อง copy เข้า repo
- **สั่งให้ persona จับ selector เจาะจง class/testid ไม่ใช่ text กว้าง ๆ** — selector แบบ `a,button,div[class*=card]` filter ข้อความทั่วไป (เช่น "SO"/"ใบ") มีโอกาสไปโดน nav link บนสุดที่บังเอิญมีคำเดียวกัน แทนปุ่มที่ต้องการในการ์ด (พบจริง T7: 2/10 persona รายงาน "navigation พัง" เพราะเหตุนี้ ไม่ใช่บั๊กแอป) — ยืนยันด้วย screenshot ก่อนเชื่อว่า "กดปุ่มถูกต้องแล้ว"
- **ห้ามตั้งค่า fixture (ทะเบียนรถ/รหัสที่ค้นหาได้) ให้ซ้ำกับข้อมูลจริงในฐาน** — ถ้าซ้ำ persona ที่ค้นด้วยฟิลด์นั้นมีโอกาสเปิดข้อมูลจริงแทน fixture โดยไม่รู้ตัว (พบจริง T7: ตั้งทะเบียนปลอมชนทะเบียนจริงที่มี 971 รอบ ทำให้หลาย persona ทดสอบข้อมูลผิดชุด) — ตั้ง key ที่ค้นหาได้ (docno, รหัส, ทะเบียน) ให้เป็นค่าที่ไม่มีทางชนของจริงเสมอ
- **หลัง client-side navigate (คลิกปุ่มที่เปลี่ยนหน้าโดยไม่ reload เต็ม) ต้องรอ signal ที่มั่นคงก่อน interact ต่อ ไม่ใช่แค่ URL เปลี่ยน** — ถ้าหน้าใหม่ fetch ข้อมูล async แล้ว unmount/remount component ระหว่างรอ (เช่น `{doc && <Form/>}`) การ `.fill()`/คลิกที่แข่งจังหวะนี้จะเจอ element เก่าที่กำลังถูกแทนที่ ค่าที่กรอกหายไปเงียบๆ — สั่งให้ persona (และ e2e ของตัวเอง) รอ element ที่ยืนยันว่าข้อมูลใหม่โหลดเสร็จจริง (เช่น title/เลขที่ตรงกับที่คาด) ก่อนกดต่อเสมอ (พบจริง T8: ทั้ง e2e ของตัวเองและ Haiku บาง persona พลาดแบบเดียวกัน)

**เกณฑ์ผ่าน: ≥8/10 คนทำ task หลักสำเร็จ (≥5/6 ข้อ)** — ไม่ผ่าน = แก้ UI แล้ววนทดสอบใหม่ทั้งรอบ

ต้นทุนจริงต่อรอบ: Haiku ≈ 60-110K tokens/ตัว, ~6-17 นาที (คู่ขนาน)

### สัญญาณที่อ่านได้จากผู้ใช้จำลอง

- **Haiku อ่านข้อความไทยจาก screenshot เพี้ยนเมื่อฟอนต์เล็ก** (เช่น "ไม่มีขนาด" → "ไม่มีนาค") — ใช้เป็น proxy signal เรื่อง readability ได้จริง; รอบ T5 นำไปสู่การขยาย base font 14.5→15.5px, chip ≥13px
- **Haiku อ่านเครื่องหมาย "~" (ตัวหนอน = ประมาณ) เป็น "-" (ลบ) ผิดซ้ำหลายครั้ง** (พบ T7: 3/10 persona รายงานว่าน้ำหนัก "ติดลบ") — เป็นข้อจำกัดการอ่านภาพ ไม่ใช่บั๊ก ต้องเปิด screenshot จริงเช็คก่อนเชื่อทุกครั้งที่มีการรายงานค่าติดลบ/ผิดปกติผิดวิสัย
- **Element ที่ตัวอย่างข้อมูลเจอยาก จะหาไม่เจอ** (เช่น badge ⚠ ที่โผล่แค่บางบรรทัด) — ถ้า task ต้องการให้เห็น ให้เพิ่ม legend/คำอธิบายใน UI แทนหวังให้บังเอิญเจอ
- บั๊กที่จับได้จริง: ปุ่ม render แบบ disabled เมื่อ state เก่า → กติกา **"ซ่อน ไม่ใช่โชว์ปุ่มกดไม่ได้"**

### ข้อจำกัดของวิธี

LLM จำลอง motor skill/ความเคยชินจริงไม่ได้ — ถือเป็น **UX smoke test** จับปัญหา findability / labeling / flow ไม่ใช่ replacement ของ user testing จริง

## ขั้น 4 — บันทึกผล

เขียน `docs/usability-test-<ticket>.md` ใน repo ของโปรเจกต์ ครอบ:

- ตารางผลรายคน (persona × ผ่าน/ตก)
- ผลราย task
- Fix ที่ทำทันทีในรอบนั้น
- Backlog (สิ่งที่เจอแต่ยังไม่แก้)
- ข้อจำกัดของวิธีทดสอบ

ตัวอย่างรายงานจริง: `~/work/gitlab/sys9.co/do-review/docs/usability-test-t5.md`

## หลัก design ที่ตกผลึก (ใช้กับ spec ทุกงาน)

- **"แดง = ข้อมูลหายระดับบรรทัดเท่านั้น"** — aggregate ที่ขาดเป็นภาวะปกติ ห้ามใช้แดง (กัน alarm fatigue)
- ทุกสัญลักษณ์ต้องมี**ข้อความไทยกำกับ** ไม่พึ่งสีอย่างเดียว (colorblind + WCAG 1.4.1)
- chip ต้อง **≥13px**
- unmatched/ผิดปกติที่ไม่ใช่ error ใช้ **amber** ไม่ใช่แดง
- **query string เป็น source of truth ของ filter** — back/share ทำงานถูกเอง
