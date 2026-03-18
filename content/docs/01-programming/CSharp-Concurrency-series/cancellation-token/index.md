---
title: "ยกเลิก Task อย่างไรไม่ให้ระบบพัง: ศิลปะแห่ง Cooperative Cancellation"
weight: 1409
date: 2026-03-18
author: วิสิทธิ์ แผ้วกระโทก
tags: ["C#", "Concurrency", "Task", "CancellationToken", "Asynchronous"] 
---
{{< figure src="cancellation-token-cover.jpg" alt="รูปปกบทความ" >}}

## 1. 🎯 ตอนที่ 9: การยกเลิก Task อย่างถูกวิธีด้วย CancellationToken

## 2. 📖 เปิดฉาก (The Hook)
สวัสดีครับผู้อ่านทุกท่าน! กลับมาพบกันอีกครั้งในซีรีส์ **เจาะลึก C# Concurrency & Multithreading** 

ถ้าย้อนกลับไปในยุคมืดของการเขียนโปรแกรมแบบ Multithreading เมื่อเราต้องการหยุดการทำงานของ Thread ที่รันนานเกินไป (เช่น ติดลูปหรือรอโหลดข้อมูล) โปรแกรมเมอร์สายโหดมักจะใช้คำสั่ง `Thread.Abort()` ซึ่งเปรียบเสมือนการส่ง "มือสังหาร" ไปปลิดชีพพนักงาน (Thread) แบบไม่ให้ตั้งตัว! ผลที่ตามมาคือความหายนะครับ พนักงานตายคาที่โดยไม่ได้คืนทรัพยากร (Resource Leaks), ไฟล์ถูกล็อกค้างไว้, หรือข้อมูลใน Database อยู่ในสถานะครึ่งๆ กลางๆ

แต่ในยุคแสงสว่างของ **Task Parallel Library (TPL)** และ Modern C# สถาปัตยกรรมได้เปลี่ยนไปอย่างสิ้นเชิง Microsoft ได้แนะนำกลไกที่เรียกว่า **Cooperative Cancellation (การยกเลิกแบบร่วมมือกัน)** ซึ่งเปลี่ยนจากการ "สั่งฆ่า" เป็นการ "ส่งสัญญาณขอร้องอย่างสุภาพ" ให้พนักงานเก็บของและปิดงานด้วยตัวเอง วันนี้ในฐานะ Senior Architect รุ่นพี่ ผมจะพาคุณไปเจาะลึกการใช้เวทมนตร์ที่ชื่อว่า `CancellationToken` เพื่อยกเลิก Task อย่างสง่างามและถูกต้องตามหลักการกันครับ!

## 3. 🧠 แก่นวิชา (Core Concepts)
ระบบ Cooperative Cancellation ใน .NET 4.0 เป็นต้นมา ถูกออกแบบมาโดยแยกหน้าที่ของผู้สั่งการและผู้ปฏิบัติงานออกจากกันอย่างชัดเจน เพื่อความปลอดภัยของระบบ ลองมาดูองค์ประกอบหลักกันครับ:

*   **Cooperative Cancellation (การยกเลิกแบบร่วมมือ):** 
    กฎเหล็กคือ "การยกเลิกจะไม่ถูกบังคับ (Not forced)" ผู้สั่งการทำได้แค่ "ขอร้อง" ส่วน Task ปลายทางต้องเป็นคนคอยตรวจสอบสัญญาณ (Polling) และตัดสินใจหยุดการทำงานด้วยตัวเองในจุดที่ปลอดภัย
*   **`CancellationTokenSource` (ผู้ถือกดปุ่มยกเลิก):**
    เป็นออบเจกต์ (Class) ที่ฝั่งผู้สั่งการ (เช่น UI Thread) ถือเอาไว้ หน้าที่ของมันคือเป็นตัวสร้าง Token และมีเมธอด `Cancel()` หรือ `CancelAfter()` เพื่อปล่อยสัญญาณฉุกเฉิน
*   **`CancellationToken` (เพจเจอร์รับสัญญาณ):**
    เป็นโครงสร้างข้อมูลแบบ Value Type ขนาดเล็ก (Struct) ที่ถูกสร้างจาก `CancellationTokenSource.Token` และจะถูกโยนเข้าไปให้ Task ปลายทางถือไว้ หน้าที่ของมันคือให้ Task ใช้ตรวจสอบสถานะ `IsCancellationRequested` ว่ามีใครกดปุ่มยกเลิกมาหรือยัง
*   **การตอบสนอง (Responding):**
    เมื่อ Task ตรวจพบว่าสัญญาณถูกส่งมา มันไม่ควรแค่ `return` ออกไปเงียบๆ แต่ควรเรียกเมธอด `ThrowIfCancellationRequested()` เพื่อโยน `OperationCanceledException` ออกมา การทำเช่นนี้เป็นการประกาศให้ระบบ CLR และ Task รู้ว่า "ฉันไม่ได้ทำงานเสร็จนะ แต่ฉันจงใจหยุดเพราะถูกสั่งยกเลิก" ซึ่งจะทำให้สถานะของ Task เปลี่ยนเป็น `TaskStatus.Canceled` อย่างถูกต้อง (ไม่ใช่ `RanToCompletion` หรือ `Faulted`)

{{< figure src="cancellation-token-diagram.jpg" alt="รูปประกอบ Cooperative Cancellation Flow" >}}

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code)
ลองมาดูตัวอย่างการเขียนโค้ดเพื่อยกเลิกงานที่ใช้เวลานานๆ (เช่น ลูปประมวลผลข้อมูล) อย่างถูกวิธีกันครับ

```csharp
using System;
using System.Threading;
using System.Threading.Tasks;

public class CancellationDemo
{
    public static async Task Main()
    {
        Console.WriteLine("--- เริ่มต้นโปรแกรม ---");

        // 1. สร้าง "ผู้สั่งการ" (Source)
        // ข้อควรระวัง: CancellationTokenSource ต้องใช้ using เพื่อจัดการ unmanaged resources
        using (var cts = new CancellationTokenSource())
        {
            // 2. ดึง "เพจเจอร์รับสัญญาณ" (Token) ออกมา
            CancellationToken token = cts.Token;

            // 3. โยน Token เข้าไปใน Task
            // Senior Tip: สังเกตว่าเราส่ง token เข้าไป 2 ที่ 
            // - ที่แรก: ส่งให้ Task.Run() เพื่อให้ CLR รู้ว่าถ้ากดยกเลิกก่อนรัน ไม่ต้องสตาร์ท Task เลย
            // - ที่สอง: ส่งเข้าไปในเมธอด DoHeavyWork เพื่อใช้เช็คตอนที่ Task รันไปแล้ว
            Task myTask = Task.Run(() => DoHeavyWork(token), token); 

            // จำลองการทำงานอื่น แล้วกดยกเลิก
            Console.WriteLine("รอ 2 วินาที แล้วจะกดยกเลิกงาน...");
            await Task.Delay(2000);
            
            // 4. "ผู้สั่งการ" ปล่อยสัญญาณยกเลิก!
            cts.Cancel();
            Console.WriteLine("--- สัญญาณยกเลิกถูกส่งไปแล้ว! ---");

            try
            {
                // 5. รอให้ Task พัง (โยน OperationCanceledException ออกมา)
                await myTask;
            }
            catch (OperationCanceledException)
            {
                // 6. ดักจับ Error นี้เฉพาะเพื่อบอกผู้ใช้ ไม่ใช่บั๊กของระบบ
                Console.WriteLine($"[ดักจับ] Task ถูกยกเลิกเรียบร้อยแล้ว สถานะคือ: {myTask.Status}"); // Canceled
            }
            finally
            {
                Console.WriteLine("--- โปรแกรมจบการทำงาน ---");
            }
        }
    }

    // Task ปลายทาง: "ผู้รับเหมา" ที่ต้องคอยตรวจสอบสัญญาณระหว่างทำงาน
    static void DoHeavyWork(CancellationToken ct)
    {
        for (int i = 1; i <= 10; i++)
        {
            // 7. [Cooperative Check] วิธีที่ 1: ตรวจสอบสัญญาณทีละรอบแบบเร็วๆ
            // ถ้าใช้ `IsCancellationRequested` เฉยๆ เราสามารถ Cleanup ก่อนโยน Error ได้
            if (ct.IsCancellationRequested)
            {
                Console.WriteLine($"[Worker] ตรวจพบสัญญาณขอยกเลิกที่รอบ {i}... กำลัง Cleanup ทรัพยากร!");
                
                // 8. เมธอดนี้จะโยน OperationCanceledException ทันที
                // การโยน Error เป็นสิ่งสำคัญเพื่อให้ Task กลายเป็นสถานะ "Canceled"
                ct.ThrowIfCancellationRequested(); 
            }

            // จำลองงานหนัก (CPU-bound) หรืองาน I/O
            Console.WriteLine($"[Worker] กำลังทำงานรอบที่ {i}...");
            Thread.Sleep(500); 
            // Senior Tip: อย่าใช้ Thread.SpinWait() หรือ .Sleep() กับ Thread Pool ถ้าไม่จำเป็นนะครับ 
            // ในงานจริงควรใช้ await Task.Delay(500, ct) แล้วทำเมธอดนี้ให้เป็น async
        }
    }
}
```

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips)
ในฐานะสถาปนิกซอฟต์แวร์ นี่คือข้อควรระวังหรือ "จุดตาย" ที่ Developer หลายคนพลาดเมื่อทำงานกับ CancellationToken ครับ:

*   **อย่าแค่ `return` ออกจากลูปเด็ดขาด:** บางคนเช็ค `if (ct.IsCancellationRequested) return;` ซึ่งมันทำงานได้ครับ ลูปหยุดจริง แต่สิ่งที่ตามมาคือสถานะของ Task จะกลายเป็น `RanToCompletion` (รันจบสมบูรณ์) เสมือนว่างานทำเสร็จครบ 100% ซึ่งจะทำให้คนที่เรียก `await` คิดว่าคุณได้ผลลัพธ์มาครบถ้วน นำไปสู่บั๊กในระบบที่แก้ไขยากครับ คุณ **ต้องโยน `OperationCanceledException`** กลับออกมา เพื่อให้ TPL รู้ว่าคุณจงใจหยุดกลางคัน
*   **กฎของการส่ง Token ไป 2 ที่:** ทำไมเราต้องส่ง Token ไปทั้งใน `Task.Run(..., token)` และใน `DoHeavyWork(token)`? เหตุผลก็คือ ถ้าผู้สั่งการกดยกเลิก *ก่อนที่* Task จะมีคิวเข้าไปรันบน Thread Pool (สตาร์ทไม่ทัน), ตัว `Task.Run` จะเห็น Token และระงับการสร้าง Task ทันที (Transition เป็น Canceled ทันทีโดยไม่ต้องเรียก Delegate ขยะ) แต่ถ้า Task รันไปแล้ว (เปลี่ยนสถานะเป็น Running), CLR จะหมดหน้าที่ระงับ Task นั้น และส่งไม้ต่อให้พนักงานที่ชื่อ `DoHeavyWork` เป็นคนเอา Token ตัวเดียวกันไปเช็คเอง (Cooperative) ว่าต้องหยุดหรือไม่
*   **`CancellationTokenSource` ใช้แล้วทิ้ง (Single-use):** ออบเจกต์ตัวนี้เมื่อคุณเรียก `.Cancel()` ไปแล้ว มันจะไม่มีวันเปลี่ยนสถานะกลับเป็น `false` ได้อีกเลย และเนื่องจากมันมี unmanaged resources ภายใน (Wait Handles) คุณควรเรียกใช้บล็อก `using` เสมอ เพื่อให้มั่นใจว่าเมื่อจบงาน `.Dispose()` จะถูกเรียกเคลียร์ความจำ
*   **ตรวจสอบ (Polling) ถี่แค่ไหนดี?** การเรียก `ThrowIfCancellationRequested()` นั้นทำงานเร็วมาก (เป็นการอ่านตัวแปร volatile หนึ่งครั้ง) อย่างไรก็ตาม Microsoft แนะนำว่าคุณควรตรวจเช็คสัญญาณสัก 1 ครั้งต่อ 1-10 มิลลิวินาที ซึ่งเป็นจังหวะที่ระบบตอบสนองทันท่วงทีและไม่เกิดคอขวดครับ

## 6. 🏁 บทสรุป (To be continued...)
**Cooperative Cancellation** คือหัวใจหลักในการเขียนโปรแกรมแบบ Multithreading ยุคใหม่ ที่เน้น "ความร่วมมือกัน" ระหว่าง Thread ปลายทางและ Main Thread การเขียนลูปประมวลผลที่คอยฟัง (Poll) เสียงเรียกจาก CancellationToken ทำให้ระบบสามารถถอนตัวได้อย่างปลอดภัย ประหยัดทรัพยากร CPU และไม่ทิ้งปัญหาไฟล์ค้างไว้เป็นภาระของ OS ครับ!

ในตอนหน้า เราจะมาเจาะลึกสุดยอดของอาวุธระดับตำนานอย่าง `Parallel.ForEach` และพลพรรคของ `Task Parallel Library` (TPL) กันว่ามันจะช่วยรีดพลังจาก CPU ทุก Core บนเครื่องของคุณออกมาได้มหาศาลขนาดไหน รอติดตามกันได้เลยครับ!

---
**ต้องการที่ปรึกษาด้านการออกแบบสถาปัตยกรรมซอฟต์แวร์และการจัดการระบบ Concurrency ประสิทธิภาพสูงให้กับองค์กรของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและพัฒนาซอฟต์แวร์แบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p