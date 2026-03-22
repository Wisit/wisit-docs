---
title: "Async Streams: รีดพลัง await foreach และ IAsyncEnumerable เพื่อจัดการข้อมูลมหาศาล"
weight: 1421
date: 2026-03-22
author: วิสิทธิ์ แผ้วกระโทก
tags: ["C#", "Concurrency", "IAsyncEnumerable", "Async Streams", "await foreach", "C# 8.0"]
---
{{< figure src="async-streams-iasyncenumerable-cover.jpg" alt="รูปปกบทความ" >}}

## 1. 🎯 ตอนที่ 21: Async Streams: การใช้ await foreach (IAsyncEnumerable)

## 2. 📖 เปิดฉาก (The Hook)
สวัสดีครับผู้อ่านทุกท่าน! กลับมาพบกันอีกครั้งในซีรีส์ **เจาะลึก C# Concurrency & Multithreading** 

หากเราย้อนกลับไปในยุคก่อน C# 8.0 โปรแกรมเมอร์สาย .NET มักจะเผชิญกับภาวะกลืนไม่เข้าคายไม่ออกครับ ในฝั่งหนึ่ง เรามีคีย์เวิร์ด `yield return` ที่ช่วยสร้าง Iterator สำหรับดึงข้อมูลทีละชิ้น (Lazy Evaluation) ช่วยประหยัด Memory ได้มหาศาล แต่อีกฝั่งหนึ่ง เรามี `async/await` ที่ช่วยจัดการ I/O โดยไม่บล็อก Thread ทว่า... สองสิ่งนี้กลับ **"อยู่ร่วมโลกกันไม่ได้!"**,

ถ้าคุณต้องดึงข้อมูล 1 ล้านเรกคอร์ดจาก Database แบบ Asynchronous คุณต้องเขียนเมธอดให้คืนค่าเป็น `Task<IEnumerable<T>>`, ผลลัพธ์คือคุณต้องสร้าง `List<T>` ขนาดมหึมาเพื่อรอรับข้อมูลให้ "ครบทั้ง 1 ล้านเรกคอร์ด" เสียก่อน ถึงจะโยน (Return) ก้อนข้อมูลนั้นกลับไปให้คนเรียกใช้งานได้ (Eager Evaluation) สิ่งที่ตามมาคือการใช้ Memory ที่พุ่งปรี๊ดและโปรแกรมที่ตอบสนองช้าลงอย่างเห็นได้ชัด

เพื่อทลายกำแพงนี้ วิศวกรของไมโครซอฟท์จึงได้เปิดตัวสถาปัตยกรรมใหม่ใน C# 8.0 และ .NET Core 3.0 ที่ชื่อว่า **"Async Streams"** วันนี้ในฐานะ Software Architect ผมจะพาคุณไปสัมผัสเวทมนตร์ของการรวมร่างระหว่าง `yield return` และ `await` ผ่านอินเทอร์เฟซทรงพลัง `IAsyncEnumerable<T>` ที่จะเปลี่ยนวิธีที่คุณจัดการกับกระแสข้อมูลไปตลอดกาลครับ!

## 3. 🧠 แก่นวิชา (Core Concepts)
กลไกของ Async Streams ถูกสร้างขึ้นบนรากฐานของ Interface ชุดใหม่ที่เข้ามาปฏิวัติวงการ Collection ใน .NET ครับ:

*   **กำเนิด `IAsyncEnumerable<T>`:** 
    มันคือตัวแทนของ Collection ที่ข้อมูลแต่ละชิ้นอาจจะ "ไม่ได้มาพร้อมกันทันที" (Piecemeal fashion) เช่น ข้อมูลจาก Video Stream หรือการเรียก API แบบแบ่งหน้า (Paging), แทนที่จะคืนค่าแบบ `Task<IEnumerable<T>>` ที่ต้องรอของครบถึงจะมาเป็นก้อนใหญ่ มันอนุญาตให้คุณเสิร์ฟข้อมูล "ทีละชิ้น" ให้กับผู้เรียกได้เลย
*   **เจาะลึก `IAsyncEnumerator<T>`:** 
    เบื้องหลังของมันคืออินเทอร์เฟซ `IAsyncEnumerator<T>` ที่เปลี่ยนสถาปัตยกรรมของเมธอด `MoveNext()` แบบเดิมๆ ให้กลายเป็น `MoveNextAsync()`, และคืนค่าเป็น `ValueTask<bool>`, ซึ่งหมายความว่า ในขณะที่ระบบกำลังเอื้อมมือไปหยิบข้อมูลชิ้นถัดไปจาก Network หรือ Database มันสามารถปล่อย Thread ให้ว่าง (Asynchronous yield) ได้ทันที!
*   **`await foreach` (The Consumer):**
    เมื่อมีผู้สร้างข้อมูลแบบขนาน ย่อมต้องมีผู้เสพข้อมูล C# จึงเพิ่มคีย์เวิร์ด `await foreach` เข้ามาเพื่อดึงข้อมูลจาก `IAsyncEnumerable<T>` โดยมันจะรับหน้าที่จัดการจังหวะการรอ (Await) ชิ้นข้อมูลถัดไป และทำการทำลายล้างทรัพยากร (Dispose) ผ่าน `IAsyncDisposable` ให้อัตโนมัติเมื่อจบการทำงาน,
*   **เปรียบเทียบให้เห็นภาพ:**
    `Task<IEnumerable<T>>` เปรียบเสมือนการสั่ง **"อาหารโต๊ะจีน"** ที่เชฟต้องทำอาหารให้ครบทั้ง 10 อย่างแล้วยกมาเสิร์ฟพร้อมกัน (กิน Memory เยอะ รอนาน) ในขณะที่ `IAsyncEnumerable<T>` เปรียบเสมือนการทานซูชิแบบ **"โอมากาเสะ (Omakase)"** ที่เชฟปั้นเสร็จ 1 คำ (yield return) คุณก็หยิบเข้าปาก (await foreach) ได้ทันที! (ใช้ Memory น้อย ข้อมูลไหลลื่น)

{{< figure src="async-streams-iasyncenumerable-diagram.jpg" alt="รูปประกอบ Architecture Diagram เปรียบเทียบ Task<IEnumerable<T>> กับ IAsyncEnumerable<T>" >}}

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code)
ลองมาดูการเขียน Iterator ที่สามารถ `await` ได้ภายในตัว และวิธีการเรียกใช้งานด้วย `await foreach` กันครับ,,,

```csharp
using System;
using System.Collections.Generic;
using System.Threading.Tasks;

public class AsyncStreamsDemo
{
    // 1. ฝั่ง Producer (ผู้ผลิตข้อมูล)
    // สังเกตว่าเราสามารถใช้คำว่า async กับ IAsyncEnumerable<int> ได้เลย
    public static async IAsyncEnumerable<int> GenerateNumbersAsync(int start, int count, int delayMs)
    {
        for (int i = start; i < start + count; i++)
        {
            // สามารถ await งาน I/O หรือหน่วงเวลาได้สบายๆ 
            // Thread จะไม่ถูกบล็อกระหว่างรอ
            await Task.Delay(delayMs); 
            
            // เมื่อข้อมูลพร้อม ก็เสิร์ฟ "ทีละคำ" ให้ผู้เรียกด้วย yield return
            yield return i; 
        }
    }

    // 2. ฝั่ง Consumer (ผู้เสพข้อมูล)
    public static async Task ProcessStreamAsync()
    {
        Console.WriteLine("เริ่มดึงข้อมูลจาก Async Stream...");

        // ใช้ await foreach เพื่อดึงข้อมูลทีละชิ้น 
        // สังเกตว่าตัวเลขจะถูกปรินต์ออกมาทีละตัวตาม Delay ไม่ต้องรอของมาครบทั้งหมด!
        await foreach (var number in GenerateNumbersAsync(0, 10, 500))
        {
            Console.WriteLine($"ได้รับข้อมูล: {number} เมื่อ {DateTime.Now:T}");
        }

        Console.WriteLine("ดึงข้อมูลเสร็จสิ้น!");
    }
}
```

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips)
ในฐานะสถาปนิกซอฟต์แวร์ นี่คือจุดตายและเคล็ดวิชาขั้นสูงที่คุณต้องรู้เมื่อนำ Async Streams ไปใช้บนโปรเจกต์จริงครับ:

*   **วิชาลับ Cancellation Token (`[EnumeratorCancellation]`):** 
    เวลาที่เราดึงข้อมูลผ่าน Stream ยาวๆ ผู้ใช้อาจจะกดยกเลิกกลางคันได้ กฎเหล็กคือคุณต้องส่ง `CancellationToken` เข้าไปให้ Stream ด้วย แต่ในโครงสร้างของ `IAsyncEnumerable` คุณต้องใช้ Attribute `[EnumeratorCancellation]` แปะไว้หน้าพารามิเตอร์ เพื่อให้คอมไพเลอร์รู้ว่าจะต้องจับ Token นี้ฉีดเข้าไปใน `GetAsyncEnumerator()` อย่างถูกต้อง,
    ```csharp
    public async IAsyncEnumerable<int> GetDataAsync([EnumeratorCancellation] CancellationToken token = default) 
    {
        // ใช้ token ภายในลูป 
        await Task.Delay(1000, token);
        yield return 1;
    }
    ```
    เวลาเรียกใช้ให้ใช้ Extension Method `.WithCancellation()` แบบนี้ครับ:
    `await foreach (var item in GetDataAsync().WithCancellation(myToken))`
*   **`ConfigureAwait(false)` ในลูป `await foreach`:**
    ในบริบทของไลบรารีหรือแบ็กเอนด์ที่ไม่ได้สนใจ UI Thread คุณไม่ควรให้ `await foreach` กระโดดกลับมาที่ `SynchronizationContext` เดิมในทุกๆ รอบของลูป (ซึ่งกิน Overhead สูงมาก) คุณสามารถปิดมันได้โดยการเติม `.ConfigureAwait(false)` ตามหลัง Stream ดังนี้:
    `await foreach (var item in GetDataAsync().ConfigureAwait(false))`
*   **Pull-based (IAsyncEnumerable) ปะทะ Push-based (Rx / IObservable):**
    อย่าจำสับสนกับ Reactive Extensions (Rx) นะครับ! `IAsyncEnumerable<T>` เป็นสถาปัตยกรรมแบบ **Pull (ดึง)** ซึ่งหมายความว่า Consumer เป็นผู้กุมบังเหียน สั่งขอข้อมูลชิ้นต่อไป (ผ่าน MoveNextAsync) ดังนั้น Producer จะไม่ผลิตข้อมูลจนกว่าคุณจะร้องขอ ส่วน Rx `IObservable<T>` เป็นแบบ **Push (ผลัก)** ที่ Producer จะยิงเหตุการณ์ (Events) ใส่คุณรัวๆ ไม่ว่าคุณจะพร้อมรับหรือไม่ก็ตาม จงเลือกใช้ให้ถูกกับสถานการณ์ (Use case) ครับ,

## 6. 🏁 บทสรุป (To be continued...)
**Async Streams (`IAsyncEnumerable<T>`)** เป็นฟีเจอร์ระดับมาสเตอร์พีซที่เข้ามาเติมเต็มระบบ Asynchronous ของ C# ให้สมบูรณ์แบบ มันช่วยให้เราสามารถประมวลผลข้อมูลขนาดมหาศาลแบบ Real-time ได้โดยไม่ใช้ Memory จนหมดเกลี้ยง (OutOfMemoryException) เป็นอาวุธที่ Web API หรือ Microservices สมัยใหม่ควรหยิบมาใช้เพื่อยกระดับ Performance ให้กับระบบอย่างแท้จริงครับ

ในตอนหน้า เราจะมาเจาะลึกฟีเจอร์แห่งโลกคอนเคอเรนซีที่ลงลึกไปถึงระดับหน่วยความจำ (Memory Management) อย่าง **Span&lt;T&gt; และ Memory&lt;T&gt;** เครื่องมือที่จะช่วยให้เราหั่นและคัดลอกข้อมูลโดยไม่ต้องสร้างภาระให้กับ Garbage Collector (Zero Allocation) รอติดตามความเร็วแสงกันได้เลยครับ!

---
**ต้องการที่ปรึกษาด้านการออกแบบสถาปัตยกรรมซอฟต์แวร์และการจัดการระบบ Concurrency ประสิทธิภาพสูงให้กับองค์กรของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและพัฒนาซอฟต์แวร์แบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p