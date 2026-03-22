---
title: "ทฤษฎีของ Task Schedulers และ Custom Scheduling: ผู้จัดการคิวงานระดับเซียน"
weight: 1422
date: 2026-03-22
author: วิสิทธิ์ แผ้วกระโทก
tags: ["C#", "Concurrency", "TaskScheduler", "TPL", "Multithreading", "Scheduling"]
---
{{< figure src="task-schedulers-custom-scheduling-cover.jpg" alt="รูปปกบทความ" >}}

## 1. 🎯 ตอนที่ 22: ทฤษฎีของ Task Schedulers และ Custom Scheduling

## 2. 📖 เปิดฉาก (The Hook)
สวัสดีครับผู้อ่านทุกท่าน! กลับมาพบกันอีกครั้งในซีรีส์ **เจาะลึก C# Concurrency & Multithreading** 

ตลอดซีรีส์ที่ผ่านมา เวลาที่เราต้องการโยนงานไปทำแบบขนาน เรามักจะใช้ท่าไม้ตายอย่าง `Task.Run()` หรือ `Task.Factory.StartNew()` ใช่ไหมครับ? ทันทีที่เราเรียกคำสั่งเหล่านี้ เรากำลังโยนงานลงไปใน "หลุมดำ" ที่เรียกว่า .NET Thread Pool และปล่อยให้ระบบจัดการทุกอย่างให้อัตโนมัติ 

แต่ในโลกของสถาปัตยกรรมซอฟต์แวร์ระดับองค์กร บางครั้ง "ค่าเริ่มต้น" ก็ไม่ตอบโจทย์เสมอไปครับ ลองนึกภาพว่าคุณมีงานที่ต้องโยนเข้าคิว 100,000 งาน แต่คุณอยากจำกัดให้ทำงานพร้อมกันได้สูงสุดแค่ 4 งาน (Maximum Concurrency) เพื่อไม่ให้เซิร์ฟเวอร์ Database ปลายทางพัง หรือคุณมี "งาน VIP" ที่อยากให้ลัดคิวงานปกติที่รออยู่เป็นหมื่นๆ ชิ้น 

ในสถานการณ์แบบนี้ พระเอกที่จะมากอบกู้ระบบของเราคือ **`TaskScheduler`** ครับ! วันนี้ผมในฐานะ Senior Dev จะพาคุณไปผ่าตัดดูสมองกลที่คอยแจกจ่ายงานให้ Thread และเรียนรู้วิธีสร้าง "Custom Scheduler" เพื่อควบคุมทั้งระดับ Concurrency และ Priority ของ Task กันครับ!

## 3. 🧠 แก่นวิชา (Core Concepts)
เพื่อให้เห็นภาพว่า `Task` ไปถึง `Thread` ได้อย่างไร เราต้องมาทำความรู้จักกับตัวกลางที่เรียกว่า **Task Scheduler** ก่อนครับ:

*   **`TaskScheduler` คืออะไร?**
    เปรียบเสมือน "ผู้จัดการร้านอาหาร" ที่คอยรับออเดอร์ (Task) จากลูกค้า แล้วตัดสินใจว่าจะส่งออเดอร์นี้ไปให้พ่อครัว (Thread) คนไหนทำ และทำเมื่อไหร่ มันคือคลาส Abstract ที่รับหน้าที่จัดคิวงานให้กับ Thread
*   **The Default Task Scheduler (ผู้จัดการสายฟรีสไตล์):**
    หากเราไม่ระบุอะไร .NET จะใช้ `TaskScheduler.Default` ซึ่งทำงานร่วมกับ CLR Thread Pool อย่างแนบแน่น ผู้จัดการคนนี้มีกลยุทธ์สุดล้ำที่เรียกว่า **Work-Stealing** คือแบ่งคิวงานส่วนตัวให้พ่อครัวแต่ละคน (Local Queue) ถ้าพ่อครัวคนไหนทำออเดอร์ตัวเองหมดแล้ว ก็สามารถแอบไป "ขโมย" (Steal) งานจากคิวของพ่อครัวคนอื่นมาทำได้ เพื่อให้ประสิทธิภาพโดยรวม (Throughput) สูงสุด
*   **The Synchronization Context Scheduler (ผู้จัดการสาย UI):**
    ในแอปฯ ที่มีหน้าจออย่าง WPF หรือ Windows Forms จะมี Scheduler อีกตัวที่สร้างจาก `TaskScheduler.FromCurrentSynchronizationContext()` หน้าที่ของมันคือบังคับให้งานทั้งหมดต้องไปรันบน UI Thread เท่านั้น เพื่อป้องกันปัญหา Cross-thread exception
*   **ทำไมต้อง Custom Scheduler?**
    ผู้จัดการ Default เก่งเรื่อง "ความเร็วรวม" แต่แย่เรื่อง "ความเป็นระเบียบ" มันไม่สนเรื่องลำดับคิว (บางทีเอาของใหม่ทำก่อน) ไม่สนเรื่อง VIP และอาจจะส่งพ่อครัวรุมทำงานพร้อมกันเป็นร้อยคนจนทรัพยากรล้น (Oversubscription) หากคุณต้องการจัดระเบียบ เช่น จำกัดจำนวนพ่อครัว หรือจัดลำดับความสำคัญ คุณต้องเปลี่ยนผู้จัดการร้านใหม่ครับ!

{{< figure src="task-schedulers-custom-scheduling-diagram.jpg" alt="รูปประกอบ Architecture Diagram การทำงานของ TaskScheduler" >}}

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code)
การเขียน Custom `TaskScheduler` จากศูนย์ (สืบทอดคลาส Abstract) ต้องเขียนเมธอดจัดการคิวเองค่อนข้างยาวและซับซ้อน แต่โชคดีที่ไมโครซอฟท์มีเครื่องมือลับและคลาสสำเร็จรูปมาให้เราใช้งานได้ทันทีครับ!

**1. การจำกัดระดับ Concurrency ด้วย `ConcurrentExclusiveSchedulerPair`**
ใน .NET 4.5 เป็นต้นมา ไมโครซอฟท์ได้เพิ่มคลาสพิเศษที่ช่วยให้เราจำกัดจำนวน Task ที่ทำงานพร้อมกันได้อย่างง่ายดาย โดยไม่ต้องพึ่ง `SemaphoreSlim` ครับ

```csharp
using System;
using System.Threading;
using System.Threading.Tasks;

public class ThrottledSchedulerDemo
{
    public static void RunDemo()
    {
        // 1. สร้าง Scheduler Pair โดยครอบทับ Default Scheduler 
        // และจำกัดให้ทำงานพร้อมกันได้ "สูงสุดแค่ 4 งาน"
        var schedulerPair = new ConcurrentExclusiveSchedulerPair(
            TaskScheduler.Default, maxConcurrencyLevel: 4);

        // 2. ดึง Scheduler ตัวที่จำกัดโควต้าออกมาใช้งาน
        TaskScheduler customScheduler = schedulerPair.ConcurrentScheduler;

        // 3. สร้าง TaskFactory ที่ใช้ Custom Scheduler นี้
        var factory = new TaskFactory(customScheduler);

        Console.WriteLine("เริ่มโยนงาน 20 งานเข้าคิว (แต่จะรันพร้อมกันแค่ 4)...");

        for (int i = 1; i <= 20; i++)
        {
            int taskId = i;
            // 4. สั่งรันงานผ่าน Factory
            factory.StartNew(() => 
            {
                Console.WriteLine($"[Thread {Thread.CurrentThread.ManagedThreadId}] กำลังรันงานที่ {taskId}...");
                Thread.Sleep(1000); // จำลองงานหนัก 1 วินาที
            });
        }
        
        Console.ReadLine();
    }
}
```

**2. การควบคุม Priority ด้วย `PrioritizingTaskScheduler`**
หากเราต้องการลอจิกคิวแบบ VIP (ลัดคิวได้) เราต้องอาศัยคลาสจากไลบรารี **Parallel Extensions Extras** ของไมโครซอฟท์ (หรือเขียนสืบทอด `TaskScheduler` เองโดยใช้ `PriorityQueue`)

```csharp
// ตัวอย่างแนวคิดเมื่อมี PrioritizingTaskScheduler
// var priorityScheduler = new PrioritizingTaskScheduler();
// TaskScheduler vipScheduler = priorityScheduler.ComputePriorityScheduler(priority: 0); // สูงสุด
// TaskScheduler normalScheduler = priorityScheduler.ComputePriorityScheduler(priority: 1); // ธรรมดา

// var vipFactory = new TaskFactory(vipScheduler);
// var normalFactory = new TaskFactory(normalScheduler);

// // งานปกติถูกโยนเข้าคิวก่อน
// normalFactory.StartNew(() => DoNormalWork());

// // แต่งาน VIP จะถูกลัดคิวให้รันก่อนงานปกติที่ยังไม่ได้รัน!
// vipFactory.StartNew(() => DoVipWork()); 
```

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips)
ในระดับสถาปัตยกรรม นี่คือเคล็ดลับและ "จุดตาย" ที่คุณต้องระวังเมื่อยุ่งกับ Task Schedulers ครับ:

*   **หลีกเลี่ยงการบล็อก Thread ใน Custom Scheduler:** 
    จุดประสงค์ของ Scheduler คือการ "จัดคิว" หากโค้ดภายใน Task ของคุณมีการรันคำสั่งที่บล็อกเธรด (Blocking I/O) การจำกัด Concurrency อาจทำให้ Thread Pool หมดโควตา (Starvation) ทางแก้คือใช้ `async/await` ร่วมกับ `SemaphoreSlim` แทนหากเป็นงาน I/O-bound แต่ถ้าเป็นงานคำนวณหนักๆ (CPU-bound) การใช้ Custom Scheduler แบบจำกัดโควตาคือทางเลือกที่ถูกต้องที่สุดครับ
*   **ความซับซ้อนของ Work-Stealing:** 
    Default Task Scheduler ของ .NET ใช้ระบบ Local Queue แบบ LIFO (Last-In-First-Out) เพื่อให้แคชของ CPU ทำงานได้ดีที่สุด (Cache Locality) แลกมาด้วยการที่ "งานมักจะไม่ได้ทำงานตามลำดับที่ส่งเข้าไป" (Out of order) ดังนั้น **ห้ามคาดหวังเรื่องลำดับการทำงาน (Fairness)** จากคลาส `Task` ปกติเด็ดขาด หากคุณต้องการให้งานเรียงคิวเป๊ะๆ แบบ FIFO คุณต้องกำหนด Flag `TaskCreationOptions.PreferFairness` เข้าไปตอนสร้าง Task ครับ
*   **ระวัง Concurrent vs Exclusive:**
    คลาส `ConcurrentExclusiveSchedulerPair` มี Scheduler 2 ตัวอยู่ข้างใน ตัว `ConcurrentScheduler` อนุญาตให้ทำงานพร้อมกันตามโควตา แต่ตัว `ExclusiveScheduler` จะทำงานได้ "ทีละ 1 งานเท่านั้น" และจะ "บล็อกไม่ให้ Concurrent รันเลย" ระหว่างที่มันทำงานอยู่ เหมาะมากสำหรับใช้เป็น Reader/Writer lock ชั้นสูงโดยไม่ต้องเขียน `lock` ภายใน Task!

## 6. 🏁 บทสรุป (To be continued...)
**TaskScheduler** คือสมองกลเบื้องหลังที่กำหนดชะตากรรมของ Task ทุกตัวใน .NET การที่เรายอมดึงตัวเองออกจากความสะดวกสบายของ `Task.Run()` แล้วหันมากำหนด Scheduler ผ่าน `TaskFactory` จะช่วยให้คุณออกแบบระบบที่สามารถทนทานต่อโหลด (Scalable) และจัดลำดับความสำคัญของงาน (Priority) ได้อย่างเหนือชั้นครับ

ในตอนหน้า เราจะมาเจาะลึกโครงสร้างการทำงานแบบลึกสุดใจของ .NET Memory กันบ้าง เมื่อเราต้องการเขียนโค้ดที่รีดประสิทธิภาพสูงสุด โดยไม่สร้างขยะให้ Garbage Collector ต้องเหนื่อย เราจะใช้ฟีเจอร์ระดับเทพอย่าง **Span&lt;T&gt; และ Memory&lt;T&gt;** ได้อย่างไร? รอติดตามความเร็วแสงนี้กันได้เลยครับ!

---
**ต้องการที่ปรึกษาด้านการออกแบบสถาปัตยกรรมซอฟต์แวร์และการจัดการระบบ Concurrency ประสิทธิภาพสูงให้กับองค์กรของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและพัฒนาซอฟต์แวร์แบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p