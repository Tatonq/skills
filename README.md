# qa-e2e-skills

ชุด Claude Code **skills + agents** สาย E2E testing / QA — สกัดจากการใช้งานจริงกับระบบ SOP (sys9.co)

## โครงสร้าง

```
skills/   → วางที่ ~/.claude/skills/<name>   (หรือ symlink)
agents/   → วางที่ ~/.claude/agents/<name>.md
```

## Skills (10)

| skill | ใช้เมื่อ |
|---|---|
| `e2e-testing` | Playwright E2E patterns, POM, config, CI, flaky strategies |
| `playwright-e2e` | เขียน Playwright suite แบบมี AuthTokenManager / parallel / API testing |
| `playwright-manual-verification` | สคริปต์ Playwright ใช้ครั้งเดียวทิ้ง พิสูจน์ fix บน browser จริงก่อน commit |
| `webwright` | ขับ browser ทีละคำสั่งแบบ code-as-action พร้อม screenshot + action log |
| `browser-qa` | visual testing / UI interaction verification หลัง deploy |
| `click-path-audit` | ไล่ทุกปุ่มผ่าน state change เต็ม sequence หาบั๊กที่ฟังก์ชันเดี่ยวผ่านแต่รวมกันพัง |
| `functional-qa-personas` | Haiku personas หลายกลุ่มขับ Playwright ทำธุรกรรมจริงจนจบ business flow |
| `uat-signoff-personas` | เลเยอร์ UAT sign-off ครอบ functional-qa-personas (acceptance criteria + โหวต) |
| `ui-usability-loop` | จำลองผู้ใช้ 10 คนขับเว็บจริง → usability report (workflow มาตรฐานงาน UI) |
| `qa-debate-fix-pipeline` | pipeline ใหญ่: multi-persona QA → debate 3 โมเดล → verify → phased fix |

## Agents (2)

| agent | บทบาท |
|---|---|
| `test-engineer` | เขียน/ปรับปรุง test ทุกชั้น (Go table-driven, pytest, Playwright E2E) |
| `webwright-qa-expert` | ออกแบบ/รีวิว E2E ด้วย Webwright, บังคับ selector แบบ id/name, accessibility |

## ติดตั้ง

```bash
git clone git@github.com:Tatonq/qa-e2e-skills.git
cd qa-e2e-skills
for s in skills/*; do ln -sfn "$PWD/$s" ~/.claude/skills/$(basename "$s"); done
for a in agents/*.md; do ln -sfn "$PWD/$a" ~/.claude/agents/$(basename "$a"); done
```

> หมายเหตุ: บาง skill อ้าง path/URL ภายในทีม (ตัวอย่างรายงาน ฯลฯ) — ใช้เป็น reference pattern ได้ แต่ต้องปรับให้เข้ากับโปรเจกต์ของคุณ
