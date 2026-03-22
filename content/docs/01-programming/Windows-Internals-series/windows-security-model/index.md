---
title: "ชำแหละสถาปัตยกรรม: ตอนที่ 7 สถาปัตยกรรมความปลอดภัย (Security Model) เบื้องต้น"
weight: 407
date: 2026-03-21
author: วิสิทธิ์ แผ้วกระโทก
tags: ["Windows Kernel", "Security", "Access Token", "SID", "ACL"]
---
{{< figure src="windows-security-model-cover.jpg" alt="รูปปกบทความ" >}}

## 1. 🎯 ตอนที่ 7: สถาปัตยกรรมความปลอดภัย (Security Model) เบื้องต้น
เจาะลึกระบบรักษาความปลอดภัยระดับแก่นของ Windows ใครคือผู้กุมชะตาว่าคุณมีสิทธิ์เปิดไฟล์หรือรันโปรแกรม?

## 2. 📖 เปิดฉาก (The Hook)
จิบกาแฟกันครับน้องๆ... พี่เชื่อว่าทุกคนน่าจะเคยเจอหน้าต่างแจ้งเตือนสุดคลาสสิกอย่าง **"Access Denied"** เวลาพยายามจะลบไฟล์บางไฟล์ หรือเวลาที่เราคลิกขวาที่ไฟล์ เลือก Properties ไปที่แท็บ Security แล้วเห็นตารางที่ติ๊กเครื่องหมายถูกในช่อง Allow หรือ Deny ใช่ไหมครับ?

มองเผินๆ มันเหมือนจะเป็นแค่การตั้งค่าธรรมดาๆ แต่รู้หรือไม่ครับว่า ภายใต้หน้าต่าง UI ที่ดูเรียบง่ายนั้น มีจักรกลความปลอดภัยระดับ Kernel ที่ซับซ้อนและทรงพลังมากซ่อนอยู่ ซึ่งทำงานอยู่เบื้องหลังทุกๆ การกระทำของเราบน Windows ไม่ว่าจะเป็นการเปิดไฟล์ สร้าง Process หรือแม้แต่การอ่าน Registry วันนี้เราจะมาชำแหละกลไกเหล่านี้กันครับว่า ระบบมันรู้ได้อย่างไรว่า "เราคือใคร" และ "เรามีสิทธิ์ทำอะไรได้บ้าง"

## 3. 🧠 แก่นวิชา (Core Concepts)
ในโลกของ Windows Security การตัดสินสิทธิ์ทั้งหมดจะถูกดูแลโดยกรรมกรระดับสูงใน Kernel Mode ที่ชื่อว่า **Security Reference Monitor (SRM)** กลไกการทำงานของมันเปรียบเสมือนสมการง่ายๆ คือ: **Identity (ตัวตน) + Desired Access (สิทธิ์ที่ต้องการ) + Security Settings (การตั้งค่าของวัตถุ) = Yes หรือ No** โดยอาศัย 3 องค์ประกอบหลักดังนี้ครับ:

*   **1. SID (Security Identifier) - รหัสบัตรประชาชนที่แท้จริง**
    Windows ไม่ได้จดจำเราจากชื่อ Username อย่าง "Admin" หรือ "John" นะครับ แต่มันจดจำเราด้วยชุดตัวเลขที่เรียกว่า SID ซึ่งเป็นโครงสร้างข้อมูลที่มีความยาวไม่แน่นอน ประกอบด้วย Authority และ Relative Identifier (RID) 
    *ตัวอย่างเช่น:* `S-1-1-0` คือ SID ของกลุ่ม Everyone (ทุกคน) หรือ `S-1-5-32-544` คือ SID ของกลุ่ม Administrators การใช้ SID ทำให้มั่นใจได้ว่าตัวตนของเราจะไม่ซ้ำกันอย่างแน่นอนทั่วโลก

*   **2. Access Token - บัตรประจำตัวพนักงาน (Security Context)**
    เมื่อเราใส่รหัสผ่านและเข้าสู่ระบบสำเร็จ (Logon) กระบวนการ LSASS (Local Security Authority Subsystem Service) จะสร้างวัตถุที่เรียกว่า **Access Token** ขึ้นมาและผูกติดไว้กับ Process แรกของเรา (เช่น Explorer.exe) หลังจากนั้นทุกๆ โปรแกรมที่เราเปิดขึ้นมาก็จะ "สืบทอด" (Inherit) บัตรพนักงานใบนี้ไปใช้ด้วย
    สิ่งที่บรรจุอยู่ใน Access Token ประกอบด้วย:
    *   User SID (บอกว่าเราคือใคร)
    *   Group SIDs (บอกว่าเราอยู่แผนกไหนหรือกลุ่มไหนบ้าง)
    *   Privileges (สิทธิพิเศษระดับระบบ เช่น สิทธิ์ในการปิดเครื่อง SeShutdownPrivilege หรือสิทธิ์ในการดีบักโปรแกรม SeDebugPrivilege)

*   **3. Security Descriptor และ ACLs - แม่กุญแจและสมุดจดชื่อ**
    ทุกๆ วัตถุ (Objects) ใน Windows ไม่ว่าจะเป็นไฟล์ โฟลเดอร์ หรือ Process จะมี **Security Descriptor** แปะอยู่เสมอ มันเปรียบเสมือนแม่กุญแจที่ประกอบด้วย:
    *   Owner SID (ใครคือเจ้าของ)
    *   **DACL (Discretionary Access Control List):** นี่คือหัวใจสำคัญ! มันคือรายการที่บรรจุ **ACEs (Access Control Entries)** ที่บอกว่า SID ไหน (ใคร) มีสิทธิ์ Allow (อนุญาต) หรือ Deny (ปฏิเสธ) การเข้าถึงแบบไหน (เช่น Read, Write)
    *   **SACL (System Access Control List):** ใช้สำหรับตั้งค่าการเก็บ Log (Auditing) ว่าถ้าใครมาแอบเปิดไฟล์นี้ ให้ระบบบันทึกเหตุการณ์ลง Event Log

{{< figure src="windows-security-model-diagram.jpg" alt="รูปประกอบการทำ Access Check ระหว่าง Access Token และ Security Descriptor" >}}

## 4. 💻 ร่ายมนต์โค้ดและคำสั่ง (Show me the Code/Commands)
เราสามารถใช้คำสั่งในเครื่องมือ **WinDbg** เพื่อส่องดูไส้ในของ Access Token ในระดับ Kernel Mode ได้ครับ ว่าบัตรพนักงานใบนี้มีหน้าตาเป็นอย่างไรเมื่อระบบปฏิบัติการมองเห็น

```text
// ใช้คำสั่ง !process เพื่อหา Token ของโปรแกรม (เช่น cmd.exe)
lkd> !process 0 1 cmd.exe
...
    Token                             afd48470     <-- ได้ที่อยู่ (Address) ของ Token มาแล้ว
...

// ใช้คำสั่ง !token เพื่อแกะโครงสร้างของบัตรพนักงานใบนี้
lkd> !token afd48470
_TOKEN afd48470
TS Session ID: 0x1
User: S-1-5-21-2778343003-3541292008-524615573-500 (User: ALEX-LAPTOP\Administrator)  <-- User SID
Groups: 
 00 S-1-5-32-544 (Alias: BUILTIN\Administrators) Attributes - Mandatory Default Enabled Owner <-- Group SIDs
 01 S-1-1-0 (Well Known Group: localhost\Everyone) Attributes - Mandatory Default Enabled
 ...
Privs: 
 19 0x000000013 SeShutdownPrivilege               Attributes - 
 20 0x000000014 SeDebugPrivilege                  Attributes -   <-- สิทธิพิเศษ (Privileges) ที่มี
```
*คอมเมนต์สไตล์คุยกัน: น้องๆ จะเห็นว่าใน Token หนึ่งใบ มันเก็บข้อมูลครอบจักรวาลเลยครับ! เวลาโปรแกรมเราสั่งเปิดไฟล์ `CreateFile()` ระบบ SRM ก็จะเอา "User SID" และ "Group SIDs" จาก Token ใบนี้ ไปไล่เทียบกับ "DACL" ของไฟล์นั้นทีละบรรทัดว่ามีสิทธิ์ตรงกันไหม ถ้าไม่ตรงก็เตรียมเจอ Access Denied ได้เลยครับ*

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips)
ข้อควรระวังระดับ Pro-Tip สำหรับ System Admin และสาย Security เลยนะครับ คือเรื่อง **"กฎการจัดเรียงลำดับ ACE (Ordering the ACEs)"** 

เวลาที่เราใช้หน้าต่าง GUI (Properties -> Security) ของ Windows ไปแก้ไขสิทธิ์ หน้าต่างนั้นจะแสดงรายชื่อกลุ่มและผู้ใช้ "เรียงตามตัวอักษร (Alphabetical ordering)" ซึ่งนี่เป็น **ภาพลวงตา** ที่ทำให้หลายคนสับสนมานักต่อนัก!

ในความเป็นจริงระดับ Kernel นั้น กฎเหล็กของ SRM คือ **"กฎข้อห้าม (Deny) จะต้องถูกประมวลผลก่อนกฎอนุญาต (Allow) เสมอ"** (All Denied-type ACEs must come before Allowed types) กระบวนการ (Access Check Algorithm) จะวิ่งอ่าน ACE ใน DACL ทีละรายการจากบนลงล่าง หากมันเจอกฎ `Deny` ที่ตรงกับ SID ของเราเมื่อไหร่ มันจะเตะเราออก (Access Denied) ทันที! แม้ว่าด้านล่างจะมีกฎ `Allow` แบบ Full Control รออยู่ก็ตาม ดังนั้น อย่าเชื่อในสิ่งที่ GUI เรียงให้ดู ให้จำไว้เสมอว่า "Deny overrides Allow" ครับ!

**Trick สืบสวน:** ระบบ Windows อนุญาตให้ Thread ย่อยๆ ภายใน Process สามารถขอยืมบัตรพนักงานของคนอื่นมาสวมรอยชั่วคราวได้ด้วย! กลไกนี้เรียกว่า **Impersonation** ซึ่งมีประโยชน์มากในการทำ Client/Server Model เช่น เวลาเราโหลดหน้าเว็บผ่าน IIS ตัว Server Thread จะทำการ Impersonate เป็น User ของเราเพื่อไปอ่านไฟล์ ทำให้ปลอดภัยกว่าการให้ Server รันด้วยสิทธิ์สูงสุดตลอดเวลาครับ

## 6. 🏁 บทสรุป (To be continued...)
สถาปัตยกรรมความปลอดภัยของ Windows ถูกออกแบบมาอย่างแข็งแกร่งมากโดยอิงตามมาตรฐานระดับองค์กร (เช่น Common Criteria) การทำความเข้าใจว่าตัวตนถูกเก็บด้วย **SID** ไว้ใน **Access Token** และถูกนำไปตรวจสอบกับ **Security Descriptor (DACL)** ของวัตถุ จะช่วยให้เราสามารถวิเคราะห์ปัญหาการเข้าถึงไฟล์ หรือตามรอยการทำงานของมัลแวร์ได้อย่างทะลุปรุโปร่ง 

ในตอนต่อไป เราจะมาเจาะลึกกันว่า ถ้าเป็นเช่นนั้นแล้ว ทำไมมัลแวร์บางตัวถึงยังทะลุระบบความปลอดภัยไปได้? และเทคโนโลยีสมัยใหม่อย่าง Virtualization-based security (VBS) เข้ามาช่วยอุดช่องโหว่ได้อย่างไร ห้ามพลาดนะครับ!

---
**ต้องการที่ปรึกษาและพัฒนาระบบ Automation หรือ Enterprise Infrastructure ให้กับธุรกิจของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและติดตั้งระบบแบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p