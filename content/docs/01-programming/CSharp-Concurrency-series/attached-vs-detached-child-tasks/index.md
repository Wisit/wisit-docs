---
title: "Attached vs Detached Child Tasks: สายใยแห่ง Task แม่และลูก"
weight: 1423
date: 2026-03-23
author: วิสิทธิ์ แผ้วกระโทก
tags: ["C#", "Concurrency", "Task", "AttachedToParent", "Exception Handling"]
---
{{< figure src="attached-vs-detached-child-tasks-cover.jpg" alt="รูปปกบทความ" >}}

## 1. 🎯 ตอนที่ 23: Attached vs Detached Child Tasks (สายใยแห่ง Task แม่และลูก)

## 2. 📖 เปิดฉาก (The Hook)
สวัสดีครับผู้อ่านทุกท่าน! กลับมาพบกันอีกครั้งในซีรีส์ **เจาะลึก C# Concurrency & Multithreading** สไตล์ Software Architect ครับ

ในการเขียนโปรแกรมแบบ Concurrency สมัยใหม่ เรามักจะแบ่งงานชิ้นใหญ่ออกเป็นงานชิ้นเล็กๆ แล้วโยนเข้าไปใน Thread Pool ให้ทำงานพร้อมกัน บางครั้ง Task หลัก (Parent Task) จำเป็นต้องสร้าง Task ย่อย (Child Task) ขึ้นมาทำงานเฉพาะทางบางอย่างอยู่ภายในตัวมันเอง 

ถ้าเปรียบเทียบ Task เป็นเหมือน "ผู้จัดการ" และ Child Task เป็น "ลูกน้อง" คำถามคือ... ผู้จัดการคนนี้มีนิสัยแบบไหน? เป็นผู้จัดการแบบที่สั่งงานเสร็จแล้วก็เดินกลับบ้านไปเลย ไม่สนว่าลูกน้องจะทำเสร็จเมื่อไหร่ (Detached)? หรือเป็นผู้จัดการแบบที่ต้องนั่งเฝ้าจนกว่าลูกน้องทุกคนในทีมจะทำงานเสร็จครบ ถึงจะยอมปิดออฟฟิศกลับบ้าน (Attached)? 

ใน .NET การสร้าง Task ซ้อน Task มีกลไกที่ลึกล้ำซ่อนอยู่ครับ หากคุณไม่เข้าใจความแตกต่างระหว่างการทำงานแบบอิสระ (Detached) และการผูกมัด (Attached) คุณอาจจะต้องเจอกับบั๊กประหลาดที่โปรแกรมจบการทำงานไปก่อนที่ข้อมูลจะเซฟเสร็จ หรือ Exception ที่โผล่มาแบบจับมือใครดมไม่ได้! วันนี้เราจะมาเจาะลึกพฤติกรรมของมันกันครับ!

## 3. 🧠 แก่นวิชา (Core Concepts)
เมื่อ Task ใดๆ ทำการสร้าง Task ย่อยขึ้นมาภายใน Delegate ของตัวเอง Task ย่อยนั้นจะถูกแบ่งออกเป็น 2 ประเภทหลักๆ ดังนี้ครับ:

*   **Detached Child Tasks (ทำงานอิสระ / ลูกน้องนอกไส้):**
    *   โดยค่าเริ่มต้น (Default) เมื่อคุณสร้าง Task ย่อยขึ้นมาภายใน Task แม่ มันจะถือว่าเป็น **Detached Task** เสมอ,.
    *   มันจะทำงานอย่างเป็นอิสระจาก Task แม่ (Execute independently).
    *   Task แม่ **จะไม่รอ** ให้ Task ลูกประเภทนี้ทำงานเสร็จ,. พอ Task แม่ทำโค้ดในบล็อกของตัวเองเสร็จ มันก็ขึ้นสถานะ Completed เลยแม้ว่าลูกจะยังวิ่งอยู่ก็ตาม!
    *   Exception ของลูกประเภทนี้จะไม่ถูกส่งกลับ (Propagate) ไปยังแม่แบบอัตโนมัติ หากเกิด Error ขึ้น คุณต้องจัดการด้วยตัวเองในระดับลูก.
*   **Attached Child Tasks (ผูกติดชะตากรรม / ลูกน้องในไส้):**
    *   คุณจะสร้าง Task ประเภทนี้ได้ก็ต่อเมื่อระบุ Flag พิเศษที่ชื่อว่า `TaskCreationOptions.AttachedToParent` ลงไปตอนสร้าง Task ลูกเท่านั้น,.
    *   Task แม่จะเกิดการซิงโครไนซ์อย่างใกล้ชิด (Closely synchronized) กับลูกประเภทนี้. 
    *   Task แม่จะ **รออย่างลับๆ (Implicitly waits)** ให้ลูกที่ Attach ไว้ทั้งหมดทำงานให้เสร็จสิ้นก่อนที่ตัวมันเองจะถือว่าทำงานเสร็จสมบูรณ์.
    *   **สถานะ `WaitingForChildrenToComplete`:** เป็นสถานะพิเศษของ Task แม่! เมื่อ Task แม่รันโค้ดตัวเองเสร็จแล้ว แต่ยังมี Attached Child วิ่งอยู่ สถานะของ Task แม่จะเปลี่ยนเป็น `WaitingForChildrenToComplete`,. มันจะยังไม่เข้าสู่สถานะ `RanToCompletion`, `Faulted` หรือ `Canceled` จนกว่าลูกทุกคนจะทำงานจบ.
    *   **Exception Bubbling:** นี่คือจุดที่ทรงพลังมากครับ! หาก Attached Child โยน Exception ออกมา Exception นั้นจะทะลุ (Propagate) กลับไปหา Task แม่โดยอัตโนมัติ และจะถูกรวมไปแสดงผลตอนที่คุณเรียก `Task.Wait()` หรือ `.Result` ที่ตัวแม่,,,.

{{< figure src="attached-vs-detached-child-tasks-diagram.jpg" alt="รูปประกอบ Architecture Diagram การทำงานระหว่าง Attached และ Detached Tasks" >}}

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code)
ลองมาดูตัวอย่างการสร้าง Attached Child Task ที่ Task แม่ยอมอดทนรอจนลูกทำงานเสร็จ และยังคอยรับ Exception จากลูกมาจัดการให้ด้วยครับ:

```csharp
using System;
using System.Threading;
using System.Threading.Tasks;

public class TaskHierarchyDemo
{
    public static void RunDemo()
    {
        Console.WriteLine("=> เริ่มต้น Main Thread");

        // สร้าง Task แม่ โดยใช้ TaskFactory.StartNew
        Task parentTask = Task.Factory.StartNew(() =>
        {
            Console.WriteLine("[Parent] กำลังทำงาน...");

            // 1. สร้าง Task ลูกแบบ "ผูกติด" (AttachedToParent)
            // สังเกตการใช้ TaskCreationOptions.AttachedToParent
            Task childTask = Task.Factory.StartNew(() =>
            {
                Console.WriteLine("    [Child] เริ่มทำงาน...");
                Thread.Sleep(2000); // จำลองงานที่ใช้เวลา 2 วินาที
                
                // จำลองการเกิด Exception ใน Task ลูก
                throw new InvalidOperationException("เกิดข้อผิดพลาดรุนแรงใน Database!");
                
            }, TaskCreationOptions.AttachedToParent); 

            Console.WriteLine("[Parent] สั่งงานลูกเสร็จแล้ว กำลังจะออกจาก Delegate ของแม่...");
            // สังเกตว่า Task แม่ไม่ได้เขียนโค้ด childTask.Wait() เลยแม้แต่น้อย!
        });

        try
        {
            // 2. Main Thread รอให้ Task แม่ทำงานเสร็จ
            // เนื่องจากเราใช้ AttachedToParent การ Wait ที่แม่ จะรอให้ลูกเสร็จด้วยอัตโนมัติ!,
            parentTask.Wait(); 
        }
        catch (AggregateException ae)
        {
            // 3. Exception จากลูก จะถูกห่อ (Wrap) หายเข้าไปในแม่และทะลุออกมาที่นี่,,
            Console.WriteLine($"\n[Main] ตรวจพบ AggregateException!");
            foreach (var ex in ae.Flatten().InnerExceptions)
            {
                Console.WriteLine($"[Main] Exception ที่เจอ: {ex.Message}");
            }
        }

        Console.WriteLine($"สถานะของ Task แม่ตอนจบ: {parentTask.Status}"); 
        // จะเป็น Faulted เพราะลูกพัง แม่เลยพังตาม!,
    }
}
```

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips)
ในระดับโครงสร้างสถาปัตยกรรม นี่คือเคล็ดลับวิชาจากคัมภีร์ของชาว .NET ที่มักทำให้หลายคนตกม้าตายครับ:

*   **ระวัง `Task.Run()` ให้ดี มันคือผู้จัดการที่ปฏิเสธลูกน้อง! (`DenyChildAttach`):** 
    คุณรู้หรือไม่ว่า ถ้าคุณสร้าง Task แม่ด้วยคำสั่งยอดฮิตอย่าง `Task.Run()` ระบบจะทำการฝัง Flag `TaskCreationOptions.DenyChildAttach` ให้ตัวแม่โดยอัตโนมัติ. ความหมายของมันคือ "ฉันไม่อนุญาตให้ใครมาเกาะฉันเด็ดขาด" ดังนั้น ถ้าคุณพยายามสร้าง Task ลูกข้างในแล้วใส่ `AttachedToParent` เข้าไป... ลูกตัวนั้นจะถูกถีบออกไปกลายเป็น Detached Task ทันที (เสมือนว่าไม่ได้ใส่ Option เลย),.
*   **ทำไม Microsoft ถึงต้องทำ `DenyChildAttach`?**
    เหตุผลทางวิศวกรรมคือ เพื่อป้องกันโปรแกรมพังจาก Third-party ครับ! สมมติคุณเรียกใช้ไลบรารีของคนอื่นผ่าน `Task` ของคุณ แล้วไลบรารีนั้นดันทะลึ่งสร้าง Task ยาวๆ แล้วแปะ `AttachedToParent` มาที่ `Task` ของคุณ ถ้าไม่มีระบบป้องกันนี้ `Task` ของคุณจะไม่มีวันทำงานจบ (ค้าง) และทำให้ Performance ของระบบแย่ลงอย่างหนัก,,.
*   **มหกรรมห่อ Exception ซ้อน Exception (Nested AggregateExceptions):**
    หากคุณมี Task แม่ -> Task ลูก -> Task หลาน ที่แปะ `AttachedToParent` ต่อๆ กันเป็นทอดๆ แล้ว "หลาน" โยน Exception ออกมา... Exception นั้นจะถูกห่อด้วย `AggregateException` ตอนส่งให้ "ลูก" และจะถูกห่อด้วย `AggregateException` อีกชั้นตอนส่งให้ "แม่" นำไปสู่ฝันร้ายของคนอ่าน Log!, วิธีแก้คือให้คุณใช้เมธอด `.Flatten()` เสมอตอนดักจับ Error เพื่อบดขยี้กล่องที่ห่อซ้อนกันออกให้เหลือแค่ Exception เนื้อๆ ตัวจริงครับ,,,.
*   **การยกเลิก (Cancellation) ไม่ได้ผูกติดกัน:** 
    ถึงลูกจะ Attached กับแม่ แต่ถ้าคุณสั่ง Cancel Task แม่... Task ลูกจะไม่ถูก Cancel ตามไปด้วยโดยอัตโนมัตินะครับ! Task Cancellation เป็นระบบ "Cooperative" คุณต้องโยน `CancellationToken` ตัวเดียวกัน ส่งลงไปให้ Task ลูกตรวจสอบด้วยตัวเองเสมอ.

## 6. 🏁 บทสรุป (To be continued...)
การใช้ **Attached Child Tasks** ถือเป็นเทคนิคที่ทรงพลังมากเมื่อเราต้องการสร้าง "กราฟการทำงาน" (Graphs of asynchronous operations) ที่ประสานงานกันอย่างแน่นแฟ้น (Tightly synchronized). มันช่วยให้คุณสามารถดักจับ Exception ของงานย่อยนับร้อยตัวได้เบ็ดเสร็จในจุดเดียวที่ Task แม่! แต่จงจำไว้ว่าหากคุณต้องการให้ลูกเกาะแม่ได้ คุณต้องใช้ `Task.Factory.StartNew()` ในการสร้าง Task แม่แทนที่จะเป็น `Task.Run()` ครับ!

ในตอนหน้า เราจะขยับเข้าสู่ปัญหาที่โปรแกรมเมอร์ทุกคนต้องเจอเมื่อทำงานกับ Multithreading นั่นคือ "จะทำอย่างไรให้ Thread หลายตัวอ่านและเขียน Collection เดียวกันอย่างปลอดภัย?" เตรียมพบกับอาวุธสำคัญอย่าง **Concurrent Collections** (เช่น `ConcurrentDictionary`, `ConcurrentBag`) ที่เกิดมาเพื่อจัดการเรื่องนี้โดยเฉพาะ รอติดตามกันได้เลยครับ!

---
**ต้องการที่ปรึกษาด้านการออกแบบสถาปัตยกรรมซอฟต์แวร์และการจัดการระบบ Concurrency ประสิทธิภาพสูงให้กับองค์กรของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและพัฒนาซอฟต์แวร์แบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p