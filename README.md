# ConsignTrack — ระบบตรวจสอบการใช้เวชภัณฑ์กลุ่ม Consignment

ระบบหน้าเดียวสำหรับอัปโหลดรายงานการใช้เวชภัณฑ์ประจำวัน (Daily Sale Report) ค้นหาตาม HN
เลือกช่วงวันที่ กรองตามกลุ่มเวชภัณฑ์ (Categoryname) ดูรายละเอียดรายการ พิมพ์ใบงาน/บันทึก PDF
และเก็บฐานข้อมูลบน **Cloud Firestore (Firebase)**

---

## 1. ไฟล์ในชุดนี้

| ไฟล์ | หน้าที่ |
|---|---|
| `index.html` | ตัวเว็บทั้งหมดในไฟล์เดียว (รวมโค้ด ฟอนต์ และข้อมูลตัวอย่าง) — อัปขึ้น GitHub ได้เลย |

> ถ้าต้องการแก้ไขตัวระบบ ให้แก้ที่ไฟล์ต้นฉบับ `.dc.html` แล้ว build ใหม่ — อย่าแก้ `index.html` โดยตรง

---

## 2. ตั้งค่า Firebase (Cloud Firestore)

### 2.1 สร้างโปรเจกต์
1. เข้า https://console.firebase.google.com → **Add project** ตั้งชื่อ เช่น `consign-track`
2. เมนูซ้าย **Build → Firestore Database → Create database**
   - เลือก **Start in production mode** (หรือ test mode ระหว่างทดสอบ)
   - เลือก location เช่น `asia-southeast1 (Singapore)`

### 2.2 เอา config
1. ไอคอน ⚙ ข้างชื่อโปรเจกต์ → **Project settings**
2. เลื่อนลงหัวข้อ **Your apps** → กดไอคอน `</>` (Web) → ตั้งชื่อ app → **Register app**
3. คัดลอกอ็อบเจกต์ `firebaseConfig` ที่ขึ้นมา หน้าตาประมาณนี้:

```js
const firebaseConfig = {
  apiKey: "AIzaSy....",
  authDomain: "consign-track.firebaseapp.com",
  projectId: "consign-track",
  storageBucket: "consign-track.appspot.com",
  messagingSenderId: "1234567890",
  appId: "1:1234567890:web:abcdef"
};
```

### 2.3 ใส่ config เข้าระบบ (ไม่ต้องแก้โค้ด)
1. เปิดเว็บ → กดไอคอน **⚙ ที่มุมซ้ายล่าง** (ข้างสถานะการเชื่อมต่อ)
2. วางทั้งก้อน `{ ... }` ลงในช่อง → **บันทึกและเชื่อมต่อ**
3. จุดสถานะจะเปลี่ยนเป็นเขียว “เชื่อมต่อ Firebase แล้ว”
   ระบบจำค่าไว้ในเครื่อง (localStorage) ของเบราว์เซอร์นั้น

### 2.4 กฎความปลอดภัย (Rules)
ระหว่างทดสอบใช้ได้ตามนี้ (อนุญาตอ่าน/เขียนทั้งหมด — **ควรจำกัดก่อนใช้งานจริง**):

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /usage/{doc} { allow read, write: if true; }
  }
}
```

### โครงสร้างข้อมูล
- ระบบเขียนลง collection ชื่อ **`usage`** — 1 แถวของ Excel = 1 document
- แต่ละ document มีฟิลด์ตามคอลัมน์ Excel + `dateKey` (YYYYMMDD) + `uploadedAt`
- เวลาอัปโหลดไฟล์วันเดิมซ้ำ ระบบจะ **ลบของวันนั้นทิ้งแล้วเขียนใหม่** (กันข้อมูลซ้ำ)

---

## 3. นำขึ้น GitHub Pages (ฟรี)

### วิธี A — ผ่านหน้าเว็บ GitHub
1. สร้าง repository ใหม่ เช่น `consign-track` (Public)
2. กด **Add file → Upload files** → ลาก `index.html` เข้าไป → **Commit**
3. ไป **Settings → Pages**
4. หัวข้อ *Build and deployment* → Source: **Deploy from a branch**
   - Branch: `main` / folder: `/ (root)` → **Save**
5. รอ ~1 นาที จะได้ลิงก์ `https://<username>.github.io/consign-track/`

### วิธี B — ผ่าน Git (command line)
```bash
git init
git add index.html
git commit -m "ConsignTrack"
git branch -M main
git remote add origin https://github.com/<username>/consign-track.git
git push -u origin main
# จากนั้นเปิด Settings → Pages แล้วตั้ง Branch = main /(root)
```

> หลัง deploy แล้ว แต่ละเครื่องที่เปิดเว็บต้องวาง Firebase config ในหน้า ⚙ ครั้งแรกครั้งเดียว

---

## 4. การใช้งานประจำวัน
1. **อัปโหลด Excel** → เลือกไฟล์ `dailysalereport_export.xlsx` → ตรวจพรีวิว → **ยืนยันนำเข้าระบบ**
   (ถ้าเชื่อม Firebase แล้ว ข้อมูลจะถูกบันทึกขึ้น Cloud ให้ทุกเครื่องเห็นตรงกัน)
2. **ค้นหา HN** → พิมพ์ HN เลือกช่วงวันที่ และกดกลุ่มเวชภัณฑ์ที่สนใจ (มีปุ่ม ★ เฉพาะ Consignment)
3. คลิกแถวเพื่อดู **รายละเอียด** / กด **พิมพ์ใบงาน / PDF** เพื่อพิมพ์หรือ Save as PDF
4. **รายงานสรุป** → ดูยอดตามกลุ่ม/ผู้ป่วย/แพทย์ และส่งออก CSV
