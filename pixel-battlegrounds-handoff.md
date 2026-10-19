# Pixel Battlegrounds — Handoff สำหรับแชตใหม่

## 👉 วิธีใช้ไฟล์นี้
1. เปิดแชตใหม่ในบัญชีใหม่
2. แนบไฟล์ 2 ไฟล์: `pixel-battlegrounds-handoff.md` (ไฟล์นี้) และ `index.html` (ตัวเกม)
3. คัดลอกข้อความในกล่องด้านล่างไปวางเป็นข้อความแรก

```
นี่คือโปรเจกต์เกม Pixel Battlegrounds ที่ทำต่อจากแชตเดิม
ไฟล์ handoff.md สรุปทุกอย่าง (สถาปัตยกรรม, การตัดสินใจ, ผลทดสอบ, ข้อจำกัด)
ไฟล์ index.html คือ source code เวอร์ชันล่าสุดที่ใช้งานจริง
ช่วยอ่านทั้งสองไฟล์ก่อน แล้วทำงานต่อจากหัวข้อ "Milestone ถัดไป"
ตอบเป็นภาษาไทย ใช้อังกฤษเฉพาะคำเทคนิค
```

---

## 1. ภาพรวม
- **เกม:** auto-battler 3v3 สไตล์ pixel art ใน arena มุมมองเฉียง (pseudo-3D) มี HP bar ของทั้งสองทีมอยู่ด้านบน
- **ตัวละคร:** สัตว์ประหลาด **original ทั้งหมด 27 ตัว** — ตั้งใจไม่ใช้โปเกมอน (ชื่อ/รูป/สัญลักษณ์) เพราะติดลิขสิทธิ์ ให้คงแนวทางนี้ต่อ
- **รูปแบบไฟล์:** HTML ไฟล์เดียว (~83 KB) ไม่มี asset ภายนอก ยกเว้นฟอนต์ Silkscreen จาก Google Fonts (มี fallback)
- **การใช้งานจริง:** โฮสต์บน GitHub Pages (repo `pixel-battlegrounds`, branch `main`, root) — อัปเดตโดยอัปโหลด `index.html` ทับ

## 2. สถาปัตยกรรม (ใน `index.html`)
| ส่วน | ที่อยู่ | หน้าที่ |
|---|---|---|
| Core | `<script id="core">` | data (TYPES, CHART, MOVES, REGIONS, ROSTER_DEF), procedural sprite generator `genSprite()`, rng `mulberry32`, class `Battle` — **ไม่มี DOM** ทดสอบใน Node ได้ |
| UI | `<script>` ตัวที่สอง (IIFE) | bitmap font 5x7, prerender arena/crowd, particles pool, HUD, log, state machine, team select DOM |

- **Resolution ภายใน:** canvas 640x360 + DOM overlay ขนาดเดียวกัน ทั้ง stage scale ด้วย CSS transform (`image-rendering: pixelated`)
- **Game states:** `MENU` (มี demo battle วิ่งด้านหลัง) → `SELECT` → `PREP` (VS intro 2.2s) → `BATTLE` ⇄ `PAUSED` → `VICTORY` / `DEFEAT` / `DRAW`
- **Simulation:** fixed timestep `STEP = 1/60`, seeded rng, event queue (`send/use/launch/hit/miss/faint/fizzle/end`) — renderer อ่าน event แล้วเคลียร์ทุกเฟรม (ไม่อ่านระหว่าง PREP) ทำให้ action ไม่ซ้ำตามเฟรมเรต
- **Projection:** `ZF=2.2, FY0=120, FY1=350`; โลกกว้าง 10 x ลึก 8 หน่วย; `HOME = (2.8, 2.7)` / `(7.2, 2.7)`; `ARENA y ∈ [1.2, 4.4]`
- **Sprite:** grid 36x36 วาดที่ 2x (72px); สร้างจาก `body` (quad, biped, bird, serpent, blob, bug, fish, golem) + `feat` list; มี 2 เฟรม + flash + grey + icon 14x14

## 3. กลไกต่อสู้
- **Stats (เลเวล 50):** HP = base + 60, อื่นๆ = base + 5
- **Damage:** `((22 * power * A / D) / 50 + 2) * STAB(1.5) * typeEff * crit(1.5, โอกาส 1/14) * rand(0.85–1.0) * DMG_SCALE(0.72)` ขั้นต่ำ 1
- **Physical** = พุ่งเข้าไปตี (ระยะ 1.35) · **Special** = ชาร์จ 0.32s แล้วยิง projectile (homing, หาย ถ้าเป้าหมาย faint)
- **Cooldown:** `clamp(2.6 - (spe - 45) / 60, 1.0, 2.4) * rand(0.85–1.15)`; หลังโจมตี physical คูณ 0.8 (ชดเชยเวลาวิ่ง)
- **AI เลือกท่า:** สุ่มถ่วงน้ำหนักด้วย (expected damage)²
- **Faint:** 1.3s → ส่งตัวถัดไปหลัง 0.6s; ทีมหมด = แพ้; หมดพร้อมกัน = DRAW; เกิน 180s ตัดสินด้วย %HP รวม
- **10 ธาตุ:** Fire, Water, Leaf, Volt, Stone, Frost, Mind, Shadow, Metal, Wind (+ Basic สำหรับท่ากลาง)

## 4. Roster (27 ตัว, 9 Regions x 3)
| Region | ตัวละคร |
|---|---|
| I Emberreach | Embrux, Cindrill, Magmaw |
| II Tidewell | Ripplet, Mirelle, Kelpion |
| III Mossvale | Mossling, Thornet, Bloomsprite |
| IV Stormspire | Voltik, Sparkhorn, Galewing |
| V Frostmere | Frostail, Tundrox, Glacirex |
| VI Duskhollow | Duskeel, Umbrabat, Wraithorn |
| VII Ironridge | Ferrock, Cogsprite, Steelwyrm |
| VIII Skyreach | Runeowl, Nimbus, Talondra |
| IX Prism Isles | Voltaur, Prismane, Shoalback |

**เพิ่มตัวใหม่:** เพิ่ม 1 บรรทัดใน `ROSTER_DEF` รูปแบบ
`[id, name, region, 'TYPE/TYPE', [hp,atk,def,spa,spd,spe], 'move1 move2 move3', body, scale, '#main #sec #accent', 'feat1 feat2']`
ตัวที่ข้อมูลไม่ครบ (ท่า/ธาตุ/body/feat ไม่ถูกต้อง) ระบบจะซ่อนเอง

## 5. การทดสอบที่ทำแล้ว
- **Core (Node):** ดึงโค้ดจาก `<script id="core">` รันใน `vm` → จำลอง 3,000 แมตช์ ผ่านทั้งหมด: HP อยู่ในช่วง, ตัวที่ faint ไม่โจมตี/ไม่ถูกตี, `end` เกิดครั้งเดียว, ฝั่งแพ้ถูก wipe จริง, seed เดิมผลเดิม · ความยาวแมตช์ median ~34s (p5 23s, p95 47s) · ไม่มี timeout
- **E2E (Playwright + Chromium):** กดทุกปุ่มครบลูป (search, filter region/ธาตุ, เลือกเกิน 3, ลบ slot, random, clear, confirm, pause, speed, rematch, change team, main menu, seed) + เล่น 6 แมตช์ติด → ไม่มี error, ไม่มี projectile ค้าง
- **Balance ล่าสุด:** win rate 37–61% (แรงสุด Voltaur, Cogsprite, Sparkhorn · อ่อนสุด Frostail, Mossling, Ripplet)

## 6. การตัดสินใจ / สิ่งที่แก้ไปแล้ว
- ออกแบบ Cindrill ใหม่ (สีม่วง, เขา + เปลวไฟบนหัว, ไม่มีหาง) เพื่อไม่ให้คล้ายตัวละครที่มีลิขสิทธิ์
- ปีก Umbrabat ย่อลงไม่ให้บังตัว
- ย้ายเส้นต่อสู้ลงมากลางพื้น (wy 2.7) ให้ arena ไม่โล่งครึ่งล่าง
- เอา timer บนจอ jumbotron ออก (ซ้ำกับ HUD)
- buff Mossling, Thornet, Frostail, Umbrabat, Galewing · nerf Voltaur, Prismane

## 7. ข้อจำกัดที่รู้อยู่
- Arena เดียว ไม่มีเสียง ไม่มี status effect / ability / evolution
- Sprite 2 เฟรมต่อตัว ขนาดคงที่ (ไม่ย่อตามความลึก)
- สมดุลยังไม่เป๊ะ (ธาตุ Volt แรง, Leaf/Frost อ่อน)
- มือถือเล่นได้ แต่แนวนอนดีที่สุด
- ทีมล่าสุดจำไว้ใน localStorage ของแต่ละเบราว์เซอร์

## 8. Milestone ถัดไป (แนะนำตามลำดับ)
1. **Balance pass:** ลด Volt / เพิ่ม Leaf, Frost แล้วรัน 3,000 แมตช์เทียบ
2. **เสียง:** chiptune BGM + SFX ตามธาตุ (Web Audio, ไม่ใช้ไฟล์ภายนอก)
3. **กลยุทธ์ผู้เล่น:** จัดลำดับลงสนาม + เลือกสไตล์ AI (บุก / ระวัง)
4. **เนื้อหา:** arena ที่ 2, status effect พื้นฐาน (burn, freeze, stun)
