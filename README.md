# Special Product EXP (MP) — 2026 Sales Dashboard

แดชบอร์ดสรุปยอดขายสินค้าพิเศษ (ชิ้นส่วนไก่ส่งออก) ประจำปี 2026 — เปรียบเทียบ **แผน (Plan)** กับ **ยอดขายจริง (Actual)** และ **โอกาสการขาย (Prospect)** รายเดือน หน่วยเป็นเมตริกตัน (MT)

หน้าเว็บ **ดึงข้อมูลสดจาก Google Sheet โดยอัตโนมัติทุกครั้งที่เปิด** ไม่ต้องอัปโหลดไฟล์ CSV ใดๆ

---

## ✨ ฟีเจอร์

- **ดึงข้อมูลสดอัตโนมัติ** จาก Google Sheet ทันทีที่เปิดหน้า + รีเฟรชเองทุก 5 นาที
- **ปุ่ม 🔄 Refresh** สำหรับดึงข้อมูลล่าสุดด้วยตนเอง
- **3 มุมมอง (แท็บ):**
  - 🏠 **Overview** — KPI รวมทั้งปี (Plan / Actual / Achievement % / Gap / Prospect) + ตารางสรุปรายสินค้า 22 ประเภท
  - 📅 **Monthly** — กราฟแท่งเปรียบเทียบ Plan vs Actual vs Prospect รายเดือนของแต่ละสินค้า
  - 🏢 **By Customer** — รายละเอียดยอดขายจริงและ pipeline แยกรายลูกค้า/รายสินค้า รายเดือน
- ทำงานแบบ **ไฟล์ HTML ไฟล์เดียว** ไม่ต้องติดตั้งอะไร ไม่ต้องมี backend
- ฟอนต์ **Sarabun** โทนเอกสารทางการ + พื้นหลังไล่เฉดสีฟ้าอ่อน

---

## 🚀 วิธีใช้งาน

### เปิดดูบนเครื่อง
ดับเบิลคลิกไฟล์ `special_product_exp_mp_dashboard.html` เปิดในเบราว์เซอร์ (แนะนำ Google Chrome) ได้ทันที

> ⚠️ บางเบราว์เซอร์อาจบล็อกการดึงข้อมูลข้ามโดเมนเมื่อเปิดไฟล์ตรงๆ ผ่าน `file://`
> ถ้าข้อมูลไม่ขึ้น ให้เปิดผ่าน local server เช่น:
> ```bash
> python3 -m http.server 8777
> # แล้วเปิด http://localhost:8777/special_product_exp_mp_dashboard.html
> ```

### เปิดเป็นลิงก์ให้คนอื่น (แนะนำ) — GitHub Pages
1. push ไฟล์นี้ขึ้น GitHub repository
2. ไปที่ **Settings → Pages** เลือก branch (เช่น `main`) แล้ว Save
3. รอสักครู่จะได้ลิงก์ `https://<username>.github.io/<repo>/special_product_exp_mp_dashboard.html`
4. ใครเปิดลิงก์ก็เห็นข้อมูลสดอัตโนมัติ

---

## 🔗 แหล่งข้อมูล (Data Source)

ข้อมูลมาจาก Google Sheet — แท็บที่ 2 ชื่อ **"Special Product EXP (MP)"**

หน้าเว็บเรียกข้อมูลผ่าน URL export CSV ในตัวของ Google Sheets:

```
https://docs.google.com/spreadsheets/d/<SHEET_ID>/export?format=csv&gid=<GID>
```

ค่าที่ใช้อยู่ปัจจุบัน (กำหนดในไฟล์ HTML ส่วน `SHEET_ID` / `SHEET_GID`):

| ตัวแปร | ค่า |
|---|---|
| `SHEET_ID` | `1BBYjNZCDXacTJEy7X4JFatt1V13gqaVm3qpj__5wxd8` |
| `SHEET_GID` | `169945695` (แท็บ Special Product EXP (MP)) |

### ⚙️ เงื่อนไขสำคัญ
- Google Sheet ต้องตั้งการแชร์เป็น **"ใครก็ตามที่มีลิงก์สามารถดูได้ (Anyone with the link – Viewer)"** หน้าเว็บจึงดึงข้อมูลได้
- หากเปลี่ยนเป็น private จะดึงข้อมูลไม่ได้ (ต้องใช้ระบบ authentication เพิ่ม)

### โครงสร้างชีทที่หน้าเว็บอ่าน
- คอลัมน์เดือนเริ่มที่ **Jan-26** (คอลัมน์ลำดับที่ 21) ถึง **Dec-26**
- คอลัมน์ประเภทแถว (ลำดับที่ 8): `All PLAN`, `All ACTUAL`, `All PROSPECT`, `ACTUAL`, `PROSPECT`
- คอลัมน์ข้อมูล: RM (ประเภทสินค้า), Customer, Code, Product name, Note

---

## 🛠️ เปลี่ยนไปใช้ Sheet อื่น

แก้ค่าในไฟล์ `special_product_exp_mp_dashboard.html` ส่วนนี้:

```js
const SHEET_ID='<ใส่ ID ของไฟล์ Google Sheet>';
const SHEET_GID='<ใส่ gid ของแท็บที่ต้องการ>';
```

> `SHEET_ID` คือส่วนใน URL: `/spreadsheets/d/<SHEET_ID>/edit`
> `SHEET_GID` คือเลขหลัง `#gid=` ใน URL ของแท็บนั้น

โครงสร้างคอลัมน์ของชีทใหม่ต้องตรงกับรูปแบบเดิม (ดูหัวข้อ "โครงสร้างชีทที่หน้าเว็บอ่าน")

---

## 📂 ไฟล์ในโปรเจกต์

| ไฟล์ | คำอธิบาย |
|---|---|
| `special_product_exp_mp_dashboard.html` | ไฟล์แดชบอร์ดทั้งหมด (HTML + CSS + JS ในไฟล์เดียว) |
| `README.md` | เอกสารนี้ |

---

## 💻 เทคโนโลยี

- HTML / CSS / JavaScript ล้วน (Vanilla, ไม่มี framework, ไม่มี dependency)
- Google Sheets CSV export API สำหรับดึงข้อมูล
- Google Fonts: **Sarabun** (เนื้อหา) + **IBM Plex Mono** (ตัวเลข)

---

## 📝 หมายเหตุ

- ข้อมูลที่ฝังไว้ในไฟล์ (hardcoded) ใช้เป็น **ค่าสำรอง (fallback)** กรณีดึงจาก Google Sheet ไม่สำเร็จ (เช่น ออฟไลน์)
- หน่วยข้อมูลทั้งหมดเป็น **เมตริกตัน (MT)**
