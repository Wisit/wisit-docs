---
title: "ชำแหละสถาปัตยกรรม: ปฐมบทของ Process และ Thread ใครคือผู้ใช้แรงงานตัวจริง?"
weight: 404
date: 2026-03-13
author: วิสิทธิ์ แผ้วกระโทก
tags: ["Windows Kernel", "Process", "Thread", "System Architecture", "WinDbg"]
---
{{< figure src="process-and-thread-genesis-cover.jpg" alt="รูปปกบทความ" >}}

## 1. 🎯 ตอนที่ 4: ปฐมบทของ Process และ Thread
เจาะลึกความแตกต่างระหว่าง "ผู้จัดการ" และ "กรรมกร" ในโลกของระบบปฏิบัติการ ทำไม Windows ถึงต้องแบ่งแยกสองสิ่งนี้ออกจากกัน?

## 2. 📖 เปิดฉาก (The Hook)
หยิบกาแฟมาจิบกันสักนิดครับน้องๆ... พี่เชื่อว่าเวลาคอมพิวเตอร์อืดหรือช้า สิ่งแรกที่พวกเราทุกคนทำคือการกด `Ctrl+Shift+Esc` เพื่อเปิด Task Manager ขึ้นมาดู แล้วก็มักจะพูดกันว่า *"โห... Process ตัวนี้กิน CPU ไปตั้ง 100% แน่ะ!"* 

แต่ในฐานะที่พวกเรากำลังก้าวขึ้นเป็นวิศวกรระบบระดับเซียน พี่ต้องขอเบรกความเชื่อนี้ไว้ก่อนเลยครับ เพราะในมุมมองของ Windows Internals แล้ว **"Process ไม่เคยรันโค้ดและไม่เคยกิน CPU ครับ!"**, อ้าว... แล้วใครเป็นคนทำล่ะ? 

วันนี้เราจะมาชำแหละโครงสร้างของ 2 คำศัพท์คลาสสิกอย่าง **Process** และ **Thread** กัน ว่าเบื้องหลังที่แท้จริงแล้ว ใครคือตัวตน (Entity) ที่เข้าไปวิ่งอยู่บน CPU และทำไม Windows ยุคใหม่ถึงต้องออกแบบมาให้เป็นระบบปฏิบัติการแบบ Multithreading 

## 3. 🧠 แก่นวิชา (Core Concepts)
เพื่อให้เห็นภาพชัดเจนที่สุด ให้เปรียบเทียบ **Process** เป็น "โรงงานหรือตึกออฟฟิศ" และ **Thread** เป็น "พนักงานหรือกรรมกร" ที่อยู่ในตึกนั้นครับ

*   🏢 **Process (ผู้จัดการ / คอนเทนเนอร์):**
    ในทางเทคนิค Process คือ "ที่กักเก็บทรัพยากร (Container)" ที่ระบบปฏิบัติการสร้างขึ้นมาเพื่อรองรับการทำงานของโปรแกรม,, ตัวมันเองเป็นแค่โครงสร้างข้อมูลใน Kernel (ที่เรียกว่า `EPROCESS`), สิ่งที่ Process ครอบครองไว้ประกอบด้วย:
    *   **Virtual Address Space:** พื้นที่หน่วยความจำส่วนตัว (เปรียบเหมือนพื้นที่ตึกโรงงาน),
    *   **Executable Program:** โค้ดและข้อมูลที่ถูก Map เข้ามาในหน่วยความจำ,
    *   **Handle Table:** รายการของทรัพยากรที่เปิดใช้งานอยู่ เช่น ไฟล์, Network Sockets, Mutex (เปรียบเหมือนกุญแจไขห้องต่างๆ ในโรงงาน),
    *   **Security Context (Access Token):** สิทธิ์ความปลอดภัยที่ระบุว่าโปรแกรมนี้ทำอะไรได้บ้าง,
    *   **Process ID (PID):** รหัสประจำตัว,
    *   *ข้อสำคัญ:* Process ที่ไม่มี Thread จะไร้ประโยชน์และมักจะถูกทำลายทิ้งทันที,

*   👷‍♂️ **Thread (กรรมกรผู้ใช้แรงงาน):**
    Thread คือหน่วยย่อยที่สุดที่ Windows Kernel จัดคิวให้เข้าไปประมวลผล (Scheduled for execution) บน CPU,, ทุก Process จะต้องมีอย่างน้อย 1 Thread (Main Thread) สิ่งที่ Thread ครอบครองคือ:
    *   **CPU Registers (Context):** สถานะการทำงานของ CPU ว่ากำลังรันคำสั่งไหนอยู่ (Instruction Pointer),
    *   **Stacks:** มี 2 ตัวคือ User-mode Stack และ Kernel-mode Stack ไว้เก็บตัวแปรชั่วคราวและการเรียกฟังก์ชัน,
    *   **Thread Local Storage (TLS):** พื้นที่เก็บข้อมูลส่วนตัวของแต่ละ Thread,
    *   **Thread ID (TID):** รหัสประจำตัวของ Thread,

**ทำไม Windows ถึงต้องเป็น Multithreading OS?**
ย้อนกลับไปยุค Windows 16-bit ในอดีต OS เป็นแบบ Single-threaded หมายความว่าถ้ามีโปรแกรมไหนเกิดบั๊กแล้วเข้าสู่ Infinite Loop (ค้าง) มันจะดึงให้ทั้งระบบคอมพิวเตอร์แฮงก์ตามไปด้วยจนต้องกดปุ่ม Reset,, 

Windows ยุคใหม่ (ตั้งแต่ NT เป็นต้นมา) จึงถูกออกแบบให้เป็น **Preemptive Multithreading**, และรองรับ **Symmetric Multiprocessing (SMP)**, หมายความว่า OS สามารถกระจาย Thread จำนวนมากไปรันบน CPU หลายๆ คอร์ได้อย่างอิสระ, หาก Thread ของโปรแกรมหนึ่งค้าง OS ก็แค่สลับ (Context Switch) เอา Thread อื่นขึ้นมาทำงานแทน ทำให้ระบบมีความเสถียร (Robustness) และตอบสนองได้ดี (Responsiveness) นั่นเองครับ,,

{{< figure src="process-and-thread-genesis-diagram.jpg" alt="รูปประกอบความสัมพันธ์ของ Process และ Thread" >}}

## 4. 💻 ร่ายมนต์โค้ดและคำสั่ง (Show me the Code/Commands)
เวลาที่เราต้องการดูว่าโครงสร้างระดับ Kernel ของ Process และ Thread หน้าตาเป็นอย่างไร เราสามารถใช้เครื่องมือระดับเทพอย่าง **WinDbg** เข้าไปส่องดูโครงสร้างที่ชื่อว่า `_EPROCESS` (Executive Process) และ `_ETHREAD` ได้ครับ

```text
// ส่องดูโครงสร้างของ Process (EPROCESS) ด้วย WinDbg
lkd> dt nt!_eprocess
   +0x000 Pcb              : _KPROCESS          <-- โครงสร้างระดับ Kernel ลึกสุด (KPROCESS)
   ...
   +0x2e8 UniqueProcessId  : Ptr64 Void         <-- ตรงนี้คือ PID ของเรา
   +0x2f0 ActiveProcessLinks : _LIST_ENTRY      <-- ดับเบิ้ลลิงก์ลิสต์ที่ร้อยเรียง Process ทั้งระบบเข้าด้วยกัน
   +0x418 ObjectTable      : Ptr64 _HANDLE_TABLE<-- โต๊ะเก็บกุญแจ (Handle Table)
   ...

// ส่องดูโครงสร้างของ Thread (ETHREAD)
lkd> dt nt!_ethread
   +0x000 Tcb              : _KTHREAD           <-- โครงสร้าง KTHREAD ที่เก็บสถานะของ CPU
   ...
   +0x618 Cid              : _CLIENT_ID         <-- เก็บทั้ง PID ของ Process แม่ และ TID ของตัวเอง
   +0x660 ImpersonationInfo : Ptr64 _PS_IMPERSONATION_INFORMATION <-- ข้อมูลการสวมรอยสิทธิ์ (Security)
```
*คอมเมนต์สไตล์คุยกัน: จะเห็นว่าในมุมมองของ Windows แล้ว ทั้ง Process และ Thread ไม่ใช่เวทมนตร์วิเศษอะไรเลยครับ มันเป็นแค่ก้อน Data Structure (โครงสร้างข้อมูล) ขนาดใหญ่ที่ภาษา C/C++ จองพื้นที่ไว้ใน Kernel Mode Memory แล้ว OS ก็เอาไปใช้บริหารจัดการเท่านั้นเอง!*

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips)
ข้อควรระวังระดับ Pro-Tip สำหรับ Developer คือเรื่อง **"Thread Overhead" (ต้นทุนของการสร้าง Thread)** ครับ

Developer มือใหม่มักจะคิดว่า "ถ้างั้นแอปเราอยากให้เร็วๆ ก็สร้าง Thread ไปเลย 1,000 ตัวสิ!" แต่ในความเป็นจริง การสลับการทำงานระหว่าง Thread บน CPU (เรียกว่า **Context Switch**) เป็นกระบวนการที่ "แพงมาก" (Expensive operation), เพราะ CPU ต้องหยุดทำงานชั่วคราวเพื่อบันทึกค่า Register ของ Thread เก่า และโหลดค่า Register ของ Thread ใหม่ แถมยังทำให้ CPU Cache รวนอีกด้วย

**Best Practice:** การสร้าง Thread เยอะเกินจำนวน CPU Cores จะทำให้ประสิทธิภาพโดยรวม "ช้าลง" ไม่ใช่เร็วขึ้น!, หากต้องการเขียนโปรแกรมรับโหลดหนักๆ ควรหันไปใช้สถาปัตยกรรมแบบ **Thread Pool** ร่วมกับ **Asynchronous I/O (เช่น I/O Completion Ports)** แทนการสร้าง Thread ใหม่พร่ำเพรื่อครับ,,,

## 6. 🏁 บทสรุป (To be continued...)
สรุปสั้นๆ ให้ขึ้นใจเลยนะครับ: **"Process เป็นเพียงแค่กล่องเก็บของ (Container) ส่วน Thread คือกรรมกรที่ออกไปวิ่งบน CPU จริงๆ"** เมื่อเราเข้าใจจุดนี้ เวลาเจอปัญหาแอปพลิเคชันค้าง หรือต้องการทำ Performance Tuning เราจะสามารถโฟกัสไปที่ Thread และ Context Switch ได้อย่างถูกต้องและตรงจุด 

ในตอนต่อไป เราจะมาเจาะลึกกันในเรื่องที่ปราบเซียนที่สุดอีกเรื่องหนึ่ง นั่นคือ "Virtual Memory" (หน่วยความจำเสมือน) ว่า OS หลอกโปรแกรมของเราให้คิดว่ามีแรมใช้ไม่จำกัดได้อย่างไร รอติดตามกันนะครับ!

---
**ต้องการที่ปรึกษาและพัฒนาระบบ Automation หรือ Enterprise Infrastructure ให้กับธุรกิจของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและติดตั้งระบบแบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p