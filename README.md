# skills

ชุด Claude Code **skills + agents** สาย E2E testing / QA — สกัดจากการใช้งานจริงกับระบบ SOP (sys9.co)

## สารบัญ

- [โครงสร้าง](#โครงสร้าง)
- [Skills (10)](#skills-10)
  - [e2e-testing](skills/e2e-testing/SKILL.md)
  - [playwright-e2e](skills/playwright-e2e/SKILL.md)
  - [playwright-manual-verification](skills/playwright-manual-verification/SKILL.md)
  - [webwright](skills/webwright/SKILL.md)
  - [browser-qa](skills/browser-qa/SKILL.md)
  - [click-path-audit](skills/click-path-audit/SKILL.md)
  - [functional-qa-personas](skills/functional-qa-personas/SKILL.md)
  - [uat-signoff-personas](skills/uat-signoff-personas/SKILL.md)
  - [ui-usability-loop](skills/ui-usability-loop/SKILL.md)
  - [qa-debate-fix-pipeline](skills/qa-debate-fix-pipeline/SKILL.md)
- [Agents (2)](#agents-2)
  - [test-engineer](agents/test-engineer.md)
  - [webwright-qa-expert](agents/webwright-qa-expert.md)
- [ติดตั้ง](#ติดตั้ง)

## โครงสร้าง

```
skills/   → วางที่ ~/.claude/skills/<name>   (หรือ symlink)
agents/   → วางที่ ~/.claude/agents/<name>.md
```

## Skills (10)

| skill | ใช้เมื่อ |
|---|---|
| [`e2e-testing`](skills/e2e-testing/SKILL.md) | Playwright E2E patterns, POM, config, CI, flaky strategies |
| [`playwright-e2e`](skills/playwright-e2e/SKILL.md) | เขียน Playwright suite แบบมี AuthTokenManager / parallel / API testing |
| [`playwright-manual-verification`](skills/playwright-manual-verification/SKILL.md) | สคริปต์ Playwright ใช้ครั้งเดียวทิ้ง พิสูจน์ fix บน browser จริงก่อน commit |
| [`webwright`](skills/webwright/SKILL.md) | ขับ browser ทีละคำสั่งแบบ code-as-action พร้อม screenshot + action log |
| [`browser-qa`](skills/browser-qa/SKILL.md) | visual testing / UI interaction verification หลัง deploy |
| [`click-path-audit`](skills/click-path-audit/SKILL.md) | ไล่ทุกปุ่มผ่าน state change เต็ม sequence หาบั๊กที่ฟังก์ชันเดี่ยวผ่านแต่รวมกันพัง |
| [`functional-qa-personas`](skills/functional-qa-personas/SKILL.md) | Haiku personas หลายกลุ่มขับ Playwright ทำธุรกรรมจริงจนจบ business flow |
| [`uat-signoff-personas`](skills/uat-signoff-personas/SKILL.md) | เลเยอร์ UAT sign-off ครอบ functional-qa-personas (acceptance criteria + โหวต) |
| [`ui-usability-loop`](skills/ui-usability-loop/SKILL.md) | จำลองผู้ใช้ 10 คนขับเว็บจริง → usability report (workflow มาตรฐานงาน UI) |
| [`qa-debate-fix-pipeline`](skills/qa-debate-fix-pipeline/SKILL.md) | pipeline ใหญ่: multi-persona QA → debate 3 โมเดล → verify → phased fix |

## Agents (2)

| agent | บทบาท |
|---|---|
| [`test-engineer`](agents/test-engineer.md) | เขียน/ปรับปรุง test ทุกชั้น (Go table-driven, pytest, Playwright E2E) |
| [`webwright-qa-expert`](agents/webwright-qa-expert.md) | ออกแบบ/รีวิว E2E ด้วย Webwright, บังคับ selector แบบ id/name, accessibility |

## ติดตั้ง

```bash
git clone git@github.com:Tatonq/skills.git
cd skills
for s in skills/*; do ln -sfn "$PWD/$s" ~/.claude/skills/$(basename "$s"); done
for a in agents/*.md; do ln -sfn "$PWD/$a" ~/.claude/agents/$(basename "$a"); done
```

> หมายเหตุ: บาง skill อ้าง path/URL ภายในทีม (ตัวอย่างรายงาน ฯลฯ) — ใช้เป็น reference pattern ได้ แต่ต้องปรับให้เข้ากับโปรเจกต์ของคุณ
