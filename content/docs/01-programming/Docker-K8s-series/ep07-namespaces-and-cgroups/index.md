---
title: "โลกของ Linux Namespaces และ cgroups: เวทมนตร์เบื้องหลัง Container ที่มือใหม่ต้องรู้"
weight: 607
date: 2026-03-22
author: วิสิทธิ์ แผ้วกระโทก
tags: ["Docker", "Linux", "Namespaces", "cgroups", "DevOps"]
---
{{< figure src="ep07-namespaces-and-cgroups-cover.jpg" alt="รูปปกบทความ โลกของ Linux Namespaces และ cgroups" >}}

## 1. 🎯 ตอนที่ 7: โลกของ Linux Namespaces และ cgroups (เวทมนตร์เบื้องหลัง Container)
สวัสดีครับน้องๆ! วันนี้พี่จะขอพาวางเมาส์ พักการพิมพ์คำสั่งรัวๆ แล้วมานั่งจิบกาแฟคุยเรื่อง "ทฤษฎีเบื้องหลัง" กันสักนิดครับ หลายคนที่ใช้งาน Docker มักจะคิดว่ามันคือกล่องเวทมนตร์วิเศษที่เสกแอปพลิเคชันขึ้นมาได้ในพริบตา แต่ความจริงแล้วมันมีกลไกของระบบปฏิบัติการ (OS) ซ่อนอยู่เบื้องหลังครับ วันนี้เราจะมาเจาะลึก 2 ฮีโร่ตัวจริงที่ชื่อว่า **Linux Namespaces** และ **cgroups** กัน รับรองว่าอธิบายภาษาคน เข้าใจง่าย ไม่ปวดหัวแน่นอน!

## 2. 📖 เปิดฉาก (The Hook)
เวลาที่เราพูดถึง Virtual Machine (VM) เรามักจะนึกถึงการจำลอง Hardware ขึ้นมาใหม่ทั้งชุด ต้องลง OS ใหม่ กินแรม กิน CPU มหาศาล แต่พอมาเป็น Docker ทำไมมันถึงเบาหวิว สตาร์ทเสร็จในเสี้ยววินาที? 

คำตอบคือ Docker ไม่ได้สร้าง OS ใหม่ครับ แต่มันเป็นเพียง "กระบวนการ (Process)" ธรรมดาๆ ที่วิ่งอยู่บน Host OS ของเรานี่แหละ! เพียงแต่มันใช้ฟีเจอร์ลับของ Linux Kernel มา "หลอก" ให้ Process นั้นคิดว่าตัวมันเองเป็นเจ้าของเครื่องแต่เพียงผู้เดียว ราวกับว่าเราจับแอปพลิเคชันไปขังไว้ในห้องกระจกที่มองไม่เห็นคนอื่น และจำกัดไม่ให้กินข้าวเกินโควตา... และนั่นคือที่มาของเทคโนโลยีที่เรียกว่า **Namespaces** และ **cgroups** ครับ

## 3. 🧠 แก่นวิชา (Core Concepts)
เพื่อให้เห็นภาพชัดเจนที่สุด พี่ขอเปรียบเทียบ Container เป็น **"ห้องพักในโรงแรม"** ครับ โดยโรงแรมนี้คือ Linux Kernel (Host OS) ของเรา

**1. Linux Namespaces (การแยกส่วน / Isolation)**
Namespaces คือเทคโนโลยีที่ใช้สร้างกำแพงห้องพัก ทำให้แขกในห้อง (แอปพลิเคชัน) มองไม่เห็นว่ามีแขกคนอื่นอยู่ในโรงแรมนี้ด้วย มันหลอกแอปพลิเคชันว่า "นายคือเจ้านายคนเดียวของเครื่องนี้นะ!" Docker ใช้ Namespaces หลักๆ ดังนี้ครับ:
*   **PID (Process ID):** ทำให้ Container มี Process Tree เป็นของตัวเอง แอปพลิเคชันใน Container จะคิดว่าตัวเองเป็น PID 1 (Process แรกของเครื่อง) เสมอ และไม่สามารถมองเห็น Process ที่รันอยู่บน Host ได้เลย,
*   **NET (Network):** ให้ Container มี Network Stack ของตัวเอง เช่น มี IP Address, มี Port และ Routing Table ส่วนตัว (เหมือนแต่ละห้องพักมีเบอร์โทรศัพท์สายตรงและเร้าเตอร์ของตัวเอง)
*   **MNT (Mount):** ทำให้ Container มี Root Filesystem (`/`) เป็นของตัวเอง แยกขาดจาก Host (เหมือนแต่ละห้องมีตู้เสื้อผ้าส่วนตัว)
*   **อื่นๆ:** ยังมี IPC (แยกการสื่อสารระหว่าง Process), UTS (ให้มี Hostname ของตัวเอง), และ USER (แยกสิทธิ์ User/Group)

**2. Control Groups หรือ cgroups (การจำกัดทรัพยากร / Resource Limitation)**
ถ้า Namespaces คือกำแพงห้อง cgroups ก็คือ **"มิเตอร์น้ำและมิเตอร์ไฟ"** ครับ 
ในโรงแรมที่ทุกคนแชร์แท็งก์น้ำและหม้อแปลงไฟเดียวกัน (แชร์ CPU และ RAM ของ Host) ถ้ามีแขกห้องหนึ่งเปิดน้ำทิ้งไว้จนหมดแท็งก์ ห้องอื่นก็จะเดือดร้อน (ปัญหา Noisy Neighbor) cgroups ซึ่งถูกคิดค้นโดยวิศวกรของ Google ตั้งแต่ปี 2006 จะเข้ามาทำหน้าที่:
*   **จำกัด (Limit):** กำหนดว่า Container นี้ห้ามใช้ RAM เกิน 512MB หรือห้ามใช้ CPU เกิน 1.5 Cores
*   **จัดลำดับความสำคัญ (Prioritize):** กำหนดว่า Container ไหนควรได้ CPU ก่อนเมื่อระบบมีการใช้งานหนักๆ
*   **ตรวจสอบ (Accounting):** คอยนับว่าแต่ละ Container ใช้ทรัพยากรไปเท่าไหร่แล้ว (นี่คือที่มาของคำสั่ง `docker stats`)

{{< figure src="ep07-namespaces-and-cgroups-diagram.jpg" alt="รูปประกอบการทำงานของ Namespaces และ cgroups" >}}

## 4. 💻 ร่ายมนต์คำสั่ง (Show me the Code/Commands)
เรามาดูการพิสูจน์เวทมนตร์นี้ผ่านคำสั่ง Docker CLI กันครับ!

```bash
# 1. พิสูจน์ Namespaces (PID Isolation)
# รัน Ubuntu แล้วสั่งดู Process (ps) เราจะเห็นว่า bash ได้ PID 1
docker run --rm -it ubuntu bash
root@container_id:/# ps -ef
# Output จะโชว์ว่า PID 1 คือ /bin/bash (โดน Namespaces หลอกเรียบร้อย!)

# 2. พิสูจน์ cgroups (Memory & CPU Limitation)
# สั่งรัน NGINX โดยจำกัด RAM ไว้ที่ไม่เกิน 512 Megabytes
docker run -d --name web-limited --memory="512m" nginx:latest

# สั่งรัน Database โดยจำกัด CPU ให้ใช้ได้แค่ 1.5 Cores
docker run -d --name db-limited --cpus="1.5" mysql:latest

# ดูผลลัพธ์การทำงานของ cgroups ด้วยคำสั่ง stats
docker stats
# เราจะเห็นคอลัมน์ MEM USAGE / LIMIT โชว์ว่าลิมิตไว้ที่ 512MiB พอดีเป๊ะ!
```
*อธิบายคอมเมนต์สไตล์พี่สอนน้อง:*
เห็นไหมครับว่าคำสั่งที่เราพิมพ์เข้าไปอย่าง `--memory` หรือ `--cpus` เบื้องหลังแล้ว Docker Client จะวิ่งไปบอก Docker Daemon ให้ไปสั่ง Linux Kernel ว่า "ช่วยสร้าง cgroups ลิมิต RAM/CPU ให้ Process นี้หน่อยนะ!" 

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips)
ในฐานะ Cloud Architect พี่มีเรื่องต้องเตือนเกี่ยวกับกลไกเหล่านี้ครับ:

*   **ระวัง OOM Killer (Out Of Memory):** เนื่องจาก cgroups เป็นตัวจำกัด Hard Limit ของ RAM ถ้าแอปพลิเคชันของเราใน Container ดันกิน RAM เกินค่า `--memory` ที่ตั้งไว้ ตัว Linux Kernel จะเรียกมือสังหารที่ชื่อว่า **OOM (Out Of Memory) Killer** มา "ฆ่า" Process นั้นทิ้งทันที! ทำให้ Container ดับไปเลย ดังนั้นต้องจูนค่านี้ให้พอดีกับที่แอปพลิเคชันต้องการใช้จริงๆ นะครับ
*   **CPU Shares ไม่ใช่ Hard Limit:** ถ้าเราใช้ `--cpu-shares` (ค่าเริ่มต้นคือ 1024) มันเป็นแค่ค่าสัมพัทธ์ (Relative weight) คล้ายๆ การจัดลำดับความสำคัญ ถ้า Host ว่าง Container ก็จะกิน CPU 100% ได้ แต่ถ้า Host ยุ่ง มันถึงจะนำค่า share นี้มาหารแบ่งกันครับ,
*   **กับดัก `--privileged` Mode:** การรัน Container ด้วยธง `--privileged` คือการบอกให้ Docker ทลายกำแพง Namespaces บางส่วนทิ้งและให้สิทธิ์ระดับ Root เต็มขั้นกับ Container นั้น, ซึ่งอันตรายมากๆ ในระดับ Production เพราะแฮกเกอร์อาจทะลุจาก Container ออกมาคุม Host ได้เลย ควรหลีกเลี่ยงนะครับ!

## 6. 🏁 บทสรุป (To be continued...)
สรุปง่ายๆ ให้จำขึ้นใจเลยนะครับ: **"Namespaces ใช้จำกัดการมองเห็น (Isolation) ส่วน cgroups ใช้จำกัดการกินทรัพยากร (Resource Limitation)"**, เมื่อนำเทคโนโลยี 2 ตัวนี้ของ Linux มารวมกับความง่ายในการจัดการ Image ของ Docker เราก็จะได้สิ่งที่เรียกว่า "Container" ที่ทั้งเบา เร็ว และปลอดภัยครับ

ในตอนต่อไป เราจะมาเจาะลึกเรื่องการทำ **Networking** ของ Docker กันแบบเต็มๆ มาดูกันว่า Container ที่ถูกจับแยกห้อง (Namespaces) ไปแล้ว มันจะคุยหากันเองหรือคุยกับโลกอินเทอร์เน็ตได้อย่างไร สนุกแน่นอนครับ ติดตามกันไว้ได้เลย!

---
**ต้องการที่ปรึกษาด้านการวางระบบ Infrastructure, DevOps และ CI/CD ให้กับองค์กรของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและวางระบบ Server/Cloud แบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p