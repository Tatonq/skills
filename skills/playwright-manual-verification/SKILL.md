---
name: playwright-manual-verification
description: เขียน raw Playwright script ใช้ครั้งเดียวทิ้ง (.mjs ชั่วคราวใน project root) เพื่อยืนยันว่า fix ที่เพิ่งแก้ทำงานจริงบน browser ก่อน commit — ไม่ใช่ E2E test suite ถาวรที่ต้อง maintain (นั่นใช้ skill playwright-e2e / e2e-testing แทน). Use เมื่อแก้บั๊ก/feature ใน web app เสร็จแล้วต้องพิสูจน์บนหน้าจอจริงก่อน commit, หรือต้องหา test data จริงจาก API มาเทสเงื่อนไขเฉพาะ.
---

# Manual Post-fix Verification ด้วย Raw Playwright Script

สคริปต์ throwaway ยืนยันว่า fix ทำงานจริงบน browser ก่อน commit — **คนละเรื่องกับการเขียน E2E test suite ถาวร** ใช้แล้วลบทิ้ง ไม่ maintain ต่อ (พิสูจน์จริงกับ core-ui หลายรอบ 2026-07-11 แต่เทคนิคทั่วไปใช้ได้ทุกโปรเจกต์)

## ตำแหน่งไฟล์ + Node ESM resolution

เขียน `.mjs` ที่ `import { chromium } from "playwright"` แล้ววางไว้นอก project (เช่น scratchpad) จะ error `ERR_MODULE_NOT_FOUND` — Node ESM resolve `node_modules` จาก**ตำแหน่งไฟล์ที่ import ไม่ใช่ cwd** และ **`NODE_PATH` ใช้ไม่ได้กับ ESM สมัยใหม่** (ได้แค่ CJS)

วิธีแก้ที่ใช้จริง: copy สคริปต์เข้า project root ชั่วคราว ตั้งชื่อขึ้นต้นด้วย `.` และลงท้าย `-tmp` เช่น `.qa-creditfix-tmp.mjs` ให้เห็นชัดว่าชั่วคราว รันจากตรงนั้น **แล้วลบทิ้งทันทีหลังใช้เสร็จ ห้าม commit เด็ดขาด** (เช็ค `git status` ก่อน commit เสมอ)

## ดึง auth token แบบไม่ hardcode ซ้ำ

อ่านจาก e2e fixture file ที่มีอยู่แล้วในโปรเจกต์ด้วย `fs.readFileSync` + regex — **ห้าม copy ค่า JWT จริงมาแปะในสคริปต์ใหม่** เพราะ auto-mode classifier ของ Claude Code จะ flag การ duplicate credential ในไฟล์ใหม่เป็น "credential leakage" (token เดิมอยู่ในไฟล์ที่ commit แล้ว การอ้างอิงไม่สร้าง exposure ใหม่)

## Bootstrap มาตรฐาน

- `page.addInitScript` ตั้ง `localStorage.access_token` + localStorage key ที่ปิด onboarding/intro dialog ของแอป — pattern ทั่วไปคือ **"ปิด first-visit modal ที่ไม่เกี่ยวกับสิ่งที่จะเทสไปเลยตั้งแต่ต้น"**
- ถ้าแอปมี dialog บังคับเลือก context (สาขา/organization/tenant) ก่อนใช้งาน → เขียน helper แยก เลือกตัวเลือกแรกเสมอ + **retry pattern** (บาง dialog re-mount ระหว่าง route transition)

## Stuck overlay 2 ชั้น (gotcha ที่เจอซ้ำหลายรอบ)

1. **ชั้นแรก**: overlay div ที่ `aria-hidden=true` แต่ `data-state=open` ค้าง → fix: set `pointerEvents=none` เฉพาะ elements ที่เข้าเงื่อนไขนี้
2. **ชั้นที่สอง (ลึกกว่า พบทีหลัง)**: Radix remove-scroll ทิ้ง `document.body.style.pointerEvents = "none"` ค้างไว้แม้ dialog ปิดไปแล้วจริง — สังเกตจาก Playwright error ที่รายงานว่า **`<html>` เองเป็นตัว intercept pointer events** ทั้งที่ไม่มี dialog เปิดอยู่เลย → fix: เช็คก่อนว่าไม่มี dialog overlay ที่ visible จริง แล้วค่อย clear `body.style.pointerEvents` + neutralize `[role="dialog"][data-state="open"]` ที่เป็น zombie

## หา test data จริงแทนสร้างใหม่

ยิง `curl` ตรงไปที่ REST list endpoint (ดึง token วิธีเดียวกับข้างบน) กรองหา record จริงที่ตรงเงื่อนไขที่จะเทส เช่น "record ที่ status=locked" หรือ "type=cash ที่ไม่ล็อก" — เร็วกว่าสร้างข้อมูลใหม่ และตรงกับ data policy "ใช้ข้อมูล dev ที่มีอยู่ ห้าม seed ใหม่" ที่หลายโปรเจกต์ยึดถือ

## เทส conditional logic ทั้ง 2 ทิศทางเสมอ

ถ้า fix มีเงื่อนไข (เช่น "ทำ X เฉพาะตอน toggle = true") ต้องเทส**ทั้งสองทิศทาง** ไม่ใช่แค่ happy path — รวมถึงยืนยันฝั่ง negative ว่า toggle กลับ (false) แล้ว **"ไม่เกิดอะไรขึ้น" ตรงตามสเปกจริง** ๆ ด้วย
