# Handoff — Special Product EXP (MP) Dashboard

เอกสารส่งต่องาน สำหรับผู้ที่จะดูแล/พัฒนา dashboard นี้ต่อ
อัปเดตล่าสุด: 2026-06-02

---

## 1. ภาพรวม

Dashboard หน้าเดียว (single-file HTML) สรุปยอดขายสินค้าพิเศษส่งออกปี 2026 — เปรียบเทียบ **Plan vs Actual vs Prospect** รายเดือน หน่วยเมตริกตัน (MT)

- ดึงข้อมูล **สดจาก Google Sheet อัตโนมัติ** ทุกครั้งที่เปิดหน้า (ไม่มีการอัปโหลด CSV แล้ว)
- ไม่มี backend / ไม่มี build step — เปิดไฟล์ในเบราว์เซอร์ได้เลย
- ทุกอย่าง (HTML + CSS + JS + ข้อมูลสำรอง) อยู่ในไฟล์เดียว

---

## 2. ไฟล์ในโปรเจกต์

| ไฟล์ | บทบาท |
|---|---|
| `special_product_exp_mp_dashboard.html` | **ไฟล์หลักทั้งหมด** (HTML/CSS/JS รวมในไฟล์เดียว) |
| `README.md` | คู่มือผู้ใช้ / วิธี deploy |
| `handoff.md` | เอกสารนี้ |
| `.claude/launch.json` | config สำหรับรัน local preview server (ไม่ต้อง deploy) |

> ตอนอัปขึ้น GitHub: ไฟล์ที่จำเป็นจริงคือ `.html` + `README.md` (+ `handoff.md`) ส่วน `.claude/` เป็นแค่ตัวช่วย preview

---

## 3. แหล่งข้อมูล (Google Sheet)

| รายการ | ค่า |
|---|---|
| Sheet ID | `1BBYjNZCDXacTJEy7X4JFatt1V13gqaVm3qpj__5wxd8` |
| GID (แท็บ 2) | `169945695` — ชื่อแท็บ "Special  Product EXP (MP)" |
| URL ที่หน้าเว็บเรียก | `https://docs.google.com/spreadsheets/d/<ID>/export?format=csv&gid=<GID>` |

กำหนดไว้ในโค้ดที่ตัวแปร `SHEET_ID` / `SHEET_GID` (ฟังก์ชัน Google Sheet live source)

**เงื่อนไขสำคัญ:** ชีทต้องตั้งแชร์เป็น *"Anyone with the link – Viewer"* มิฉะนั้นจะดึงไม่ได้

### โครงสร้างคอลัมน์ที่ parser อ่าน (0-based index)
- col 2 = RM (ชื่อประเภทสินค้า)
- col 3 = Customer, col 4 = Code, col 5 = Product name, col 6 = Note
- col 8 = **Type** ของแถว: `All PLAN` / `All ACTUAL` / `All PROSPECT` / `ACTUAL` / `PROSPECT` (และ `%` ที่ข้าม)
- col 21–32 = เดือน **Jan-26 … Dec-26** (parser หา `Jan-26` แบบ dynamic ด้วย findIndex อยู่แล้ว เผื่อ layout ชีทขยับ)

### ชื่อ RM ที่ไม่ตรงระหว่างชีทกับ key ภายใน → จัดการด้วย `RM_MAP`
`04 Shoulder & Ribs`→`04 SHOULDER & RIBS`, `08 BL Scrap`→`08 BL SCRAP`, `13 Back Rib`→`13 BACK RIB`, `05 SBB TRIMMING (เอ็นกลาง)`→`05 SBB TRIMMING (CENTER TENDON)`
(ตรวจแล้ว ทั้ง 22 RM map ได้ครบ ไม่มีตกหล่น)

---

## 4. สถาปัตยกรรมโค้ด (ใน `<script>`)

**ข้อมูลสำรอง (hardcoded fallback) — ใช้เมื่อดึงสดไม่สำเร็จ**
- `D{}` = plan/actual รายเดือนต่อ RM
- `PR{}` = prospect รายเดือนต่อ RM
- `CP[]` = รายละเอียดลูกค้า (actual + prospect)
- ⚠️ ค่าพวกนี้เป็น **snapshot เก่า** (รวม Plan ≈ 12,686) ส่วนข้อมูลสดล่าสุด ≈ 15,176 / Actual ≈ 5,792 / 38% — อย่าสับสนว่าเลขสำรองคือเลขจริง

**Flow การทำงาน**
1. `loadFromSheet()` ทำงานตอนโหลดหน้า → `fetchSheetCSV(3)` (retry 3 ครั้ง backoff 600/1200ms)
2. ได้ CSV → `applyCSV(text)` parse แล้วอัปเดต `D`/`PR` + customer data → เรียก render ทั้งหมด
3. อัปเดตสถานะมุมขวาบน (`#ust` + จุดสี `#syncdot`: load/ok/err)
4. `setInterval(loadFromSheet, 5 นาที)` — refresh เองเรื่อยๆ + ปุ่ม Refresh (`#rf`)

**ฟังก์ชัน render หลัก**
- `renderKPI()` — แถบสรุป 5 ตัว (Total plan / Actual / Achievement / Gap / Prospect)
- `renderTable()` — ตาราง Overview รายสินค้า (progress bar + status badge)
- `renderCharts()` — แท็บ Monthly: bar chart 12 เดือน × 3 แท่ง/เดือน + hover tooltip + grow animation + **ตารางตัวเลขรายเดือนใต้กราฟ** (Target/Actual/Prospect × Jan–Dec + Total, class `.mt`); layout 2 คอลัมน์ (`@media min-width:980`)
- `renderCustomers(custActual, custProspect)` — แท็บ By customer: panel ต่อ RM, **แถวสรุป Target + Actual รายเดือน (RM-level) อยู่บนสุด** (class `.sumrow`) แล้วตามด้วยแถว Actual คู่ Prospect แยกรายลูกค้า
- helper: `clr(pct)` (สีตาม %), `st(pct)` (badge label+class), `F()` (format ตัวเลข), `S()` (sum)

---

## 5. ระบบดีไซน์ (Theme: "Slate & Indigo")

- **สี:** กำหนดเป็น CSS custom properties ใน `:root` ด้วย **OKLCH** — เปลี่ยนธีมทั้งหน้าได้จากจุดเดียว
  - neutrals = cool slate (`--g0`…`--g9`)
  - `--chrome` = สลีตเข้ม (header/footer)
  - data semantic: `--bl` actual/on-track (อินดิโก = accent), `--gn` over plan (เขียว), `--pu` prospect (ทอง = warm accent เดียวที่ตัดกับโทนเย็น), `--or` below target (ส้มไหม้), `--rd` attention (แดง) + แต่ละตัวมีเวอร์ชัน light `--*l` สำหรับพื้น badge/pill
  - หมายเหตุ: เคยลองธีม "Emerald & Sand" (chrome เขียว) แต่สีเขียวของ chrome ไปแข่งกับสีเขียวของข้อมูล เลยเปลี่ยนเป็น chrome กลาง (slate) ให้สีข้อมูลเด่น
- **ฟอนต์:** `Sarabun` (เนื้อหา ไทย+อังกฤษ) + `IBM Plex Mono` (ตัวเลข, tabular-nums)
- **หลักการ:** ไม่มี emoji-as-icon, ไม่มี glassmorphism, ไม่มี gradient ลูกกวาด — เน้น restrained/พรีเมียมแบบ Linear/Stripe
- มี `@media (prefers-reduced-motion: reduce)` ปิดอนิเมชันให้ผู้ใช้ที่ตั้งค่าไว้
- responsive: `@media(max-width:860px)` (summary เรียงแนวตั้ง ฯลฯ)

---

## 6. งานที่ทำไปแล้ว (timeline ย่อ)

1. เชื่อม Google Sheet (เดิม upload CSV → เปลี่ยนเป็น fetch อัตโนมัติ)
2. ลบฟีเจอร์ Upload CSV ออก
3. Auto-refresh ทุก 5 นาที + ปุ่ม Refresh
4. Redesign ธีมเป็น **Emerald & Sand** (OKLCH tokens, header เขียวมรกต)
5. ถอด emoji ทั้งหมดในส่วน customer + รวมสี prospect เป็นทองทั้งระบบ
6. แก้กราฟ Monthly **หลุดกรอบ**: ถอดตัวเลขบนแท่งออก → ใส่ **hover tooltip** (Plan/Actual/Prospect) + **grow animation** (stagger) + hover highlight
7. แก้ตาราง Overview **เหลือพื้นที่ขวา**: ให้ Product type + Progress เป็นคอลัมน์ยืด, ล็อก Status = 160px
8. เพิ่ม **retry 3 ครั้ง** ตอนดึงล้มเหลว + ข้อความสถานะชัดขึ้น (`last update kept` / `sample data`) + ใบ้กรณีเปิดผ่าน `file://`
9. เปลี่ยนธีมเป็น **Slate & Indigo** (chrome กลางให้สีข้อมูลเด่น)
10. แท็บ Monthly: เปลี่ยน grid เป็น **2 คอลัมน์** + เพิ่ม **ตารางตัวเลขรายเดือน** Target/Actual/Prospect ใต้กราฟทุกตัว

---

## 7. งานค้าง / ที่ควรพิจารณาต่อ

- [x] ~~Layout กราฟ Monthly 2 คอลัมน์~~ — ทำแล้ว (`.cgd` = 2 คอลัมน์ที่ ≥980px)
- [ ] **ข้อมูลสำรอง (`D`/`PR`/`CP`) เก่า** — ถ้าต้องการให้ตอน offline แสดงเลขใกล้เคียงของจริง ควรอัปเดต snapshot เป็นระยะ
- [ ] **เลข Week 20 / 2026** ที่ header เป็น hardcoded — ยังไม่ได้ดึงจากชีท
- [ ] **`file://` caveat** — เปิดไฟล์ตรงๆ บางเบราว์เซอร์บล็อก fetch ข้ามโดเมน แนะนำ deploy เป็น URL (ดู README)

---

## 8. วิธีรัน / ทดสอบ

```bash
cd "Nicha_stock no order"
python3 -m http.server 8777
# เปิด http://localhost:8777/special_product_exp_mp_dashboard.html
```
ตรวจว่ามุมขวาบนขึ้นจุด **เขียว "Synced HH:MM"** และ Total plan ≈ 15,176 = ดึงสดสำเร็จ
ถ้าขึ้น **แดง "Sync failed"** = ดึงไม่ติด (เช็คเน็ต / สิทธิ์แชร์ชีท / กด Refresh)

**Deploy เป็นลิงก์:** push ขึ้น GitHub แล้วเปิด GitHub Pages → ได้ URL ที่เปิดที่ไหนก็ดึงข้อมูลสดอัตโนมัติ (รายละเอียดใน README)

---

## 9. วิธีเปลี่ยนไปใช้ Sheet อื่น

แก้ `SHEET_ID` / `SHEET_GID` ในไฟล์ HTML (ส่วน Google Sheet live source)
โครงสร้างคอลัมน์ของชีทใหม่ต้องตรงรูปแบบในข้อ 3 (หรือปรับ index ใน `applyCSV` + `RM_MAP` ตามจริง)
