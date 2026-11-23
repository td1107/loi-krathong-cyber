# MOON//FLOAT — ลอยกระทงออนไลน์

เว็บแอป HTML/CSS/JavaScript หน้าเดียว ธีม Halftone Dark Cyber x Y2K ใช้ Cloud Firestore สำหรับซิงก์กระทงแบบ real-time และเก็บภาพที่ถูกบีบอัดจากฝั่ง browser ลงในเอกสาร Firestore โดยตรง

## ทดลองทันที

เปิด `index.html` ผ่าน local web server ได้เลยโดยยังไม่ต้องตั้งค่า Firebase ระบบจะทำงานเป็น **Demo Mode** ใน browser เครื่องนี้ (ไม่ได้แชร์กับเพื่อนจริง)

ตัวอย่างคำสั่งจากโฟลเดอร์โปรเจกต์:

```bash
python -m http.server 8000
```

แล้วเปิด `http://localhost:8000` ใน browser

## เปิด Real-time ด้วย Firebase

1. ไปที่ [Firebase Console](https://console.firebase.google.com/) แล้วสร้าง Project
2. เพิ่ม Web App ใน Project และคัดลอก `firebaseConfig` ที่ Firebase ให้มา
3. เปิด **Build → Authentication → Sign-in method → Anonymous** แล้ว Enable
4. เปิด **Build → Firestore Database** แล้วสร้างฐานข้อมูล
5. เข้า Firestore → Rules แล้วแทนที่ด้วยเนื้อหาในไฟล์ `firestore.rules` จากนั้นกด Publish
6. เปิด `index.html` แล้วนำค่า `apiKey`, `authDomain`, `projectId`, `appId` ไปแทนค่าตัวอย่างใน `firebaseConfig` ซึ่งอยู่ช่วงต้นของ JavaScript module ในไฟล์
7. Deploy เว็บขึ้น static hosting เช่น Firebase Hosting, Netlify หรือ GitHub Pages (สำหรับพัฒนาในเครื่องใช้ local web server ตามด้านบน)
8. ส่ง URL พร้อมรหัสห้องเดียวกันให้เพื่อน เช่น `?room=moon-crew10` ทุกคนที่ใช้รหัสเดียวกันจะเห็นกระทงชุดเดียวกัน

> Firebase Web config เป็นค่าระบุโปรเจกต์ ไม่ใช่ server secret แต่ความปลอดภัยจริงต้องอาศัย Authentication และ Firestore Security Rules เสมอ

## ข้อมูลที่จัดเก็บ

เอกสารอยู่ที่ `rooms/{roomId}/boats/{boatId}` มีชื่อเล่น, คำอธิษฐาน, รูป JPEG data URL ที่บีบอัดบน client, รูปทรงกรอบ, สไตล์กระทง, สติ๊กเกอร์, ค่าปรับแต่งภาพ, UID แบบ Anonymous และ timestamp

- รูปที่เลือก (JPG/PNG/WebP) จะถูกแปลงเป็น JPEG และย่อด้านยาวไม่เกิน 350 px
- ระบบพยายามบีบอัดภาพให้ไม่เกิน 140 KB ก่อนบันทึก
- Query เรียงเวลาล่าสุดและอ่านสูงสุด 30 เอกสารต่อห้อง
- ผู้ใช้ที่ส่งกระทงเองสามารถบันทึกภาพ PNG จากหน้าต่างรายละเอียดได้
- หากไม่ได้ใส่ Firebase config เว็บจะแสดงตัวอย่างใน local Demo Mode เท่านั้น

## ข้อควรทราบก่อนใช้งานจริง

- Security Rules ที่ให้มาอนุญาตให้อ่าน/เพิ่มกระทงได้กับผู้ใช้ที่ Anonymous sign-in สำเร็จภายในห้องที่รู้ room ID; คนที่ทราบรหัสห้องสามารถเข้าห้องได้ จึงควรใช้รหัสที่เดายากขึ้นเมื่อเป็นกลุ่มปิด
- Rules ป้องกันการแก้ไขและลบเอกสารจาก client แต่ยังไม่ได้ทำระบบผู้ดูแล/ลบเนื้อหา, rate limit หรือระบบยืนยันตัวตนด้วยบัญชีจริง
- การเก็บภาพใน Firestore เหมาะกับกลุ่มเล็กและภาพย่อขนาด ไม่เหมาะกับไฟล์ภาพใหญ่หรือผู้ใช้จำนวนมาก หากขยายระบบควรย้ายไฟล์ไป Firebase Storage แล้วเก็บ URL ใน Firestore
- ค่า `createdAt` ใช้ `serverTimestamp()`; การเรียงแบบ real-time ใช้ Firestore query `orderBy(createdAt, desc)` และ `limit(30)`
