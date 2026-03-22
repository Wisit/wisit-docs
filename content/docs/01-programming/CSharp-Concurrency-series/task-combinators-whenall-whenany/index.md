---
title: "Task Combinators: ควบคุมฝูง Task ด้วยเวทมนตร์ WhenAll และ WhenAny"
weight: 1420
date: 2026-03-22
author: วิสิทธิ์ แผ้วกระโทก
tags: ["C#", "Concurrency", "Task", "WhenAll", "WhenAny", "Async", "Await"]
---
{{< figure src="task-combinators-whenall-whenany-cover.jpg" alt="รูปปกบทความ" >}}

## 1. 🎯 ตอนที่ 20: Task Combinators: ควบคุมฝูง Task ด้วย WhenAll และ WhenAny

## 2. 📖 เปิดฉาก (The Hook)
สวัสดีครับผู้อ่านทุกท่าน! กลับมาพบกันอีกครั้งในซีรีส์ **เจาะลึก C# Concurrency & Multithreading** 

ลองจินตนาการดูนะครับว่า คุณกำลังเขียนระบบหน้าบ้าน (Front-end) หรือ API Gateway ที่ต้องดึงข้อมูลจาก 3 แหล่งพร้อมๆ กัน: ดึงโปรไฟล์ผู้ใช้จาก Database, ดึงยอดเงินจากระบบ Payment, และดึงประวัติการสั่งซื้อจากระบบ CRM ถ้าคุณใช้คีย์เวิร์ด `await` เรียก API ทีละตัวแบบเรียงลำดับ (Sequential) กว่าจะได้ข้อมูลครบ ผู้ใช้คงกดปิดหน้าเว็บหนีไปแล้ว!

หรือในอีกสถานการณ์หนึ่ง คุณต้องการดึงราคาหุ้นล่าสุด แต่คุณมี API ให้เรียกตั้ง 3 เจ้า คุณอยากจะ "ยิงคำขอไปพร้อมกันทั้ง 3 เจ้า แล้วเอาข้อมูลจากเจ้าที่ตอบกลับมาเร็วที่สุด" ส่วนที่เหลือก็ช่างมัน 

ในโลกของ C# สถาปนิกซอฟต์แวร์ไม่ต้องมานั่งเขียนลอจิกเช็คสถานะ Task ให้ปวดหัวครับ เพราะ Microsoft ได้เตรียมอาวุธหนักที่เรียกว่า **"Task Combinators"** มาให้เราแล้ว วันนี้ผมในฐานะ Senior Dev จะพาคุณไปเจาะลึกสุดยอดเมธอดอย่าง `Task.WhenAll` และ `Task.WhenAny` ที่จะเปลี่ยนโค้ดเรียงลำดับอันเชื่องช้า ให้กลายเป็นระบบ Asynchronous ประสิทธิภาพสูงกันครับ!

## 3. 🧠 แก่นวิชา (Core Concepts)
**Task Combinators** คือเมธอดที่ใช้สำหรับ "รวบรวม" (Combine) Task หลายๆ ตัวเข้าด้วยกัน แล้วสร้าง Task ตัวใหม่ขึ้นมาเพื่อเป็นตัวแทนของกลุ่มนั้น โดยที่เราไม่ต้องไปยุ่งกับ Thread หรือเขียน Signaling เองเลยครับ 

*   **`Task.WhenAll` (มาครบถึงจะจบงาน - The Fork/Join Pattern):**
    *   คืนค่าเป็น Task ตัวใหม่ที่จะ "เสร็จสมบูรณ์" (Completed) ก็ต่อเมื่อ **"Task ย่อยทุกตัวทำงานเสร็จแล้ว"**
    *   เปรียบเสมือนการสั่งอาหาร 3 อย่าง คุณจะเริ่มกินได้ก็ต่อเมื่ออาหารทั้ง 3 จานมาเสิร์ฟครบแล้วเท่านั้น
    *   หาก Task ย่อยเป็นการคืนค่า (Return value) เช่น `Task<int>` ตัว `WhenAll` จะฉลาดพอที่จะรวมผลลัพธ์ทั้งหมดกลับมาให้ในรูปแบบของ Array (`int[]`) อัตโนมัติครับ
*   **`Task.WhenAny` (ใครไวสุดคนนั้นชนะ - The Redundancy Pattern):**
    *   คืนค่าเป็น Task ตัวใหม่ที่จะ "เสร็จสมบูรณ์" ทันทีที่ **"มี Task ย่อยตัวใดตัวหนึ่งทำงานเสร็จ"** (ไม่ว่าจะสำเร็จ, พัง, หรือถูกยกเลิกก็ตาม)
    *   เปรียบเสมือนคุณเรียกแท็กซี่จาก 3 แอปพลิเคชันพร้อมกัน คันไหนขับมาถึงหน้าบ้านคุณก่อน คุณก็ขึ้นคันนั้นเลย!
    *   ค่าที่ได้กลับมาจาก `WhenAny` คือ "ตัว Task ที่ชนะ" (The winning task) ซึ่งคุณจะต้องไปดึงผลลัพธ์จากมันอีกที

{{< figure src="task-combinators-whenall-whenany-diagram.jpg" alt="รูปประกอบ Architecture Diagram เปรียบเทียบ WhenAll และ WhenAny" >}}

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code)
มาดูตัวอย่างโค้ดที่แสดงให้เห็นถึงพลังของ Combinators ทั้งสองตัวนี้กันครับ

```csharp
using System;
using System.Linq;
using System.Diagnostics;
using System.Net.Http;
using System.Threading.Tasks;

public class TaskCombinatorsDemo
{
    private static readonly HttpClient _httpClient = new HttpClient();

    // 1. ตัวอย่างการใช้ Task.WhenAll (รอจนกว่าจะโหลดเสร็จทุกเว็บ)
    public static async Task DownloadAllApisAsync()
    {
        string[] urls = { "https://api.example.com/users", "https://api.example.com/orders", "https://api.example.com/payments" };
        
        Console.WriteLine("=> เริ่มยิง API ทั้ง 3 ตัวพร้อมกันด้วย WhenAll...");
        var stopwatch = Stopwatch.StartNew();

        // สร้าง Query ของ Tasks (ยังไม่ได้ await นะครับ แค่สร้างไว้)
        var downloadTasks = urls.Select(url => _httpClient.GetStringAsync(url)).ToArray();

        // ใช้ WhenAll เพื่อรอให้ทุก Task โหลดเสร็จ
        // ผลลัพธ์ที่ได้จะถูกห่อรวมมาเป็น Array ของ string อัตโนมัติ!
        string[] results = await Task.WhenAll(downloadTasks);

        stopwatch.Stop();
        Console.WriteLine($"=> โหลดเสร็จครบ 3 ตัว! ใช้เวลาไป {stopwatch.ElapsedMilliseconds} ms");
        Console.WriteLine($"ขนาดข้อมูลที่ได้: {results.Length}, {results.Length}, {results.Length} bytes");
    }

    // 2. ตัวอย่างการใช้ Task.WhenAny (แข่งกันโหลด ใครเสร็จก่อนเอาคนนั้น)
    public static async Task GetFastestResponseAsync()
    {
        string[] urls = { "https://api.server1.com/quote", "https://api.server2.com/quote", "https://api.server3.com/quote" };
        
        Console.WriteLine("\n=> เริ่มแข่งยิง API 3 เซิร์ฟเวอร์ด้วย WhenAny...");
        
        var downloadTasks = urls.Select(url => _httpClient.GetStringAsync(url)).ToArray();

        // WhenAny จะคืนค่า "Task ที่เข้าเส้นชัยเป็นคนแรก" กลับมาให้เรา
        Task<string> winningTask = await Task.WhenAny(downloadTasks);

        // แกะผลลัพธ์ออกจาก Task ที่ชนะ (ต้อง await อีกรอบเพื่อดึงผลลัพธ์อย่างปลอดภัย)
        string fastestResult = await winningTask;

        Console.WriteLine($"=> ได้ข้อมูลจากเซิร์ฟเวอร์ที่เร็วที่สุดแล้ว! ขนาดข้อมูล: {fastestResult.Length} bytes");
        
        // หมายเหตุ: Task ที่แพ้อีก 2 ตัวจะยังคงรันอยู่หลังบ้านจนกว่าจะเสร็จ เราควรจัดการมันด้วย (ดูใน Pro-Tips)
    }
}
```

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips)
ในระดับโครงสร้างสถาปัตยกรรม (Architecture) มีจุดตายที่คุณต้องระวังเวลาใช้ Combinators เหล่านี้ครับ:

*   **กับดัก Exception ของ `WhenAll`:**
    หากคุณรัน 3 Task แล้วพังไป 2 ตัว, `Task.WhenAll` จะรอจนกว่าตัวที่ 3 จะรันเสร็จ (หรือพังตาม) แล้วจะมัดรวม Error ทั้งหมดใส่กล่องที่เรียกว่า `AggregateException` แต่ทว่า... เมื่อคุณใช้คีย์เวิร์ด `await Task.WhenAll(...)` มันจะแกะกล่องแล้ว **โยนเฉพาะ Exception ของ Task แรกสุดที่พังออกมาเท่านั้น!** หากคุณต้องการดู Error ทุกตัว คุณต้องเอาโค้ดไปครอบ `try/catch` แล้วไปเจาะดูที่ `taskArray.Where(t => t.IsFaulted)` แทนครับ
*   **วิญญาณเร่ร่อนของ `WhenAny` (Unobserved Tasks):**
    เมื่อ `WhenAny` ได้ผู้ชนะมาแล้ว คำถามคือ "แล้วผู้แพ้ล่ะ ไปไหน?" คำตอบคือมันยังคงทำงานอยู่เบื้องหลังครับ! หากผู้แพ้เหล่านั้นเกิด Exception ขึ้นมาในภายหลัง Exception นั้นจะกลายเป็น "Unobserved Task Exception" ซึ่งอาจจะโผล่มาหลอกหลอนตอน Garbage Collector ทำงาน ดังนั้นคุณควรเขียนโค้ดต่อท้ายเพื่อดักจับ (Swallow) หรือ Log Error ของผู้แพ้เสมอ เช่น `.ContinueWith(t => Log(t.Exception), TaskContinuationOptions.OnlyOnFaulted)`
*   **สร้างระบบ Timeout สุดเท่ด้วย `WhenAny`:**
    ใน Legacy Code เราอาจจะคุ้นเคยกับการส่ง Timeout ไปที่ฟังก์ชัน แต่ในโลกของ Async คุณสามารถสร้างระบบ Timeout ครอบ Task อะไรก็ได้ในโลก โดยการจับมันไปแข่งกับ `Task.Delay`!
    ```csharp
    Task myTask = DoHeavyWorkAsync();
    Task timeoutTask = Task.Delay(3000); // ตั้งเวลา 3 วินาที
    Task winner = await Task.WhenAny(myTask, timeoutTask);
    
    if (winner == timeoutTask) 
        throw new TimeoutException("ทำงานช้าเกินไป โดนตัดจบแล้ว!");
    ```

## 6. 🏁 บทสรุป (To be continued...)
**Task.WhenAll** และ **Task.WhenAny** คือสุดยอดอาวุธที่ทำให้ C# ก้าวขึ้นเป็นราชาแห่ง Asynchronous Programming มันช่วยให้เราสามารถสั่งรันงานแบบคู่ขนาน (Parallelism แบบ I/O-bound) และจัดการรวมผลลัพธ์หรือทำ Timeout ได้อย่างสวยงามด้วยโค้ดเพียงไม่กี่บรรทัดครับ

แต่ช้าก่อน... แม้ว่าเราจะควบคุมวงจรชีวิตของ Task ได้แล้ว แต่ถ้าเราต้องการให้ Task นับร้อยตัว ส่งผ่านข้อมูลหากันเหมือน "สายพานโรงงาน" (Pipeline) ล่ะ เราจะทำอย่างไร? ในตอนหน้า เราจะเปิดประตูเข้าสู่คลังอาวุธสุดล้ำที่ชื่อว่า **TPL Dataflow** เครื่องมือที่จะเปลี่ยนระบบ Concurrency ของคุณให้เป็นสายพานการผลิตอัตโนมัติ รอติดตามกันได้เลยครับ!

---
**ต้องการที่ปรึกษาด้านการออกแบบสถาปัตยกรรมซอฟต์แวร์และการจัดการระบบ Concurrency ประสิทธิภาพสูงให้กับองค์กรของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและพัฒนาซอฟต์แวร์แบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p