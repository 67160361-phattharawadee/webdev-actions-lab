# LAB: GitHub Actions + GitHub Pages

ชื่อ: Phattharawadee Chokwisitkul | รหัสนิสิต: 67160361

- (ก) GitHub Repository: https://github.com/67160361-phattharawadee/webdev-actions-lab
- (ข) GitHub Pages: https://67160361-phattharawadee.github.io/webdev-actions-lab/

## Exercise 4: Workflow เริ่มทำงานเมื่อไร และ Job deploy ทำอะไร

Workflow `Deploy Static Website` จะทำงานใน 2 กรณี คือ

- มีการ `git push` commit ใหม่ขึ้น branch `main` (`on: push: branches: [main]`)
- กดปุ่ม **Run workflow** เองในแท็บ Actions (`workflow_dispatch`)

เมื่อเริ่มแล้ว GitHub จะเปิดเครื่อง Runner `ubuntu-latest` ขึ้นมาให้หนึ่งเครื่อง แล้ว Job `deploy` จะทำทีละ step ดังนี้

1. **Checkout source code** ดึงไฟล์ในรีโปลงมาบนเครื่อง Runner
2. **Check index.html exists** รัน `test -f index.html` ถ้าไม่มีไฟล์หน้าแรก step นี้จะล้มเหลวและหยุดก่อน deploy (ข้อ Challenge)
3. **Setup Pages** ตรวจสอบว่ารีโปเปิด GitHub Pages แบบ Source = GitHub Actions แล้ว
4. **Upload static website** แพ็กไฟล์ทั้งโฟลเดอร์ (`path: '.'`) เป็น Artifact
5. **Deploy to GitHub Pages** นำ Artifact ขึ้นเว็บจริง และส่ง URL ของเว็บกลับมาแสดงใน environment `github-pages`

`permissions` ให้สิทธิ์เท่าที่จำเป็น (อ่านโค้ด, เขียน Pages, ขอ id-token) และ `concurrency` ทำให้ถ้า push ติดกันหลายครั้ง จะยกเลิกรอบเก่าแล้ว deploy เฉพาะรอบล่าสุด

## คำถามท้ายแลป

**1) GitHub Actions แตกต่างจาก GitHub Pages อย่างไร?**

GitHub Actions คือ "คนทำงาน" เป็นระบบอัตโนมัติที่รันคำสั่งตามที่เขียนไว้ในไฟล์ YAML เมื่อมีเหตุการณ์เกิดขึ้น เช่น push ส่วน GitHub Pages คือ "ที่วางเว็บ" เป็น hosting สำหรับเว็บ static ที่ให้คนอื่นเปิดดูผ่าน URL ได้ ในแลปนี้ Actions เป็นตัวนำไฟล์ไปวางบน Pages ให้อัตโนมัติ

**2) เหตุใดจึงเลือก trigger ด้วย push ที่ branch main?**

เพราะ `main` เป็น branch หลักที่เก็บโค้ดเวอร์ชันที่พร้อมใช้งาน ถ้าทดลองหรือพัฒนาฟีเจอร์ใน branch อื่น เว็บจริงจะยังไม่เปลี่ยน จะ deploy ก็ต่อเมื่อโค้ดถูกรวมเข้า `main` แล้วเท่านั้น และไม่ต้องสั่ง deploy เองทุกครั้ง

**3) ถ้าแก้ไฟล์ในเครื่องแต่ไม่ได้ push เว็บไซต์จะเปลี่ยนหรือไม่ เพราะเหตุใด?**

ไม่เปลี่ยน เพราะการแก้ไฟล์หรือแม้แต่ `git commit` เกิดขึ้นแค่ในเครื่องของเรา GitHub ยังไม่รู้ว่ามีการเปลี่ยนแปลง Workflow จึงไม่ถูก trigger ต้อง `git push` ขึ้น GitHub ก่อน Actions ถึงจะเริ่มทำงานและ deploy เวอร์ชันใหม่

**4) เหตุใดโปรเจกต์ PHP + MySQL จึงไม่สามารถนำไปรันบน GitHub Pages ได้โดยตรง?**

GitHub Pages ส่งไฟล์ให้เบราว์เซอร์ตามที่มีอยู่เท่านั้น (static) ไม่มีตัวแปลภาษา PHP ฝั่ง server และไม่มีฐานข้อมูล MySQL ให้ใช้ ไฟล์ `.php` จึงไม่ถูกประมวลผล ถ้าจะใช้ต้องย้ายส่วน backend ไปไว้บน hosting ที่รองรับ PHP/MySQL แล้วให้หน้าเว็บบน Pages เรียกใช้ผ่าน API แทน

**5) อธิบายขั้นตอนการค้นหาสาเหตุ เมื่อ Workflow มีสถานะ Failed**

1. เปิดแท็บ **Actions** แล้วคลิก run ที่มีเครื่องหมาย ❌
2. คลิกเข้า Job `deploy` ดูว่า step ไหนเป็นสีแดง (step หลังจากนั้นจะเป็น skipped)
3. กางดู log ของ step นั้นเพื่ออ่านข้อความ error
4. นำ error ไปเทียบกับตาราง Troubleshooting แล้วแก้ เช่น ตั้งค่า Settings, แก้ YAML หรือแก้โค้ด
5. แก้แล้ว push ใหม่ หรือกด **Re-run jobs** แล้วตรวจว่าผ่านเป็นสีเขียว

ตัวอย่างจริงจากแลปนี้: รอบแรกที่ push ขึ้นไป run ล้มเหลวที่ step **Setup Pages** โดย log แจ้งว่า
`Get Pages site failed. Please verify that the repository has Pages enabled and configured to build using GitHub Actions`
สาเหตุคือยังไม่ได้เปิด GitHub Pages จึงแก้โดยไปที่ Settings → Pages → Source เลือก **GitHub Actions** แล้วกด Re-run ผลคือ deploy สำเร็จ
