---
title: "เจาะลึก Visual Throughput (FPS): วัดชีพจรความลื่นไหลของ UI ผ่าน Application Timeline"
weight: 62
date: 2026-03-09
author: วิสิทธิ์ แผ้วกระโทก
tags: ["WPF", "Performance", "Application Timeline", "FPS", "Composition Thread"]
---

{{< figure src="visual-throughput-fps-timeline-cover.jpg" alt="ภาพหน้าปกบทความ Visual Throughput (FPS) ใน Application Timeline" >}}

## 1. 🎯 ชื่อบทความ (Title): เจาะลึก Visual Throughput (FPS): วัดชีพจรความลื่นไหลของ UI ผ่าน Application Timeline

## 2. 👋 เกริ่นนำ (Introduction)
สวัสดีครับน้องๆ และเพื่อนนักพัฒนาทุกคน! กลับมาพบกับพี่วิสิทธิ์อีกครั้งในซีรีส์การรีดประสิทธิภาพ (Performance Tuning) ของแอปพลิเคชัน WPF นะครับ 

เวลาที่เราเล่นเกมหรือดูหนัง สิ่งที่ทำให้เรารู้สึกว่าภาพมัน "ลื่นไหล" หรือ "กระตุก" ก็คือค่า **FPS (Frames Per Second)** หรืออัตราเฟรมต่อวินาทีนั่นเองครับ ในการพัฒนา Desktop Application ด้วย WPF ก็เช่นกัน แม้แอปของเราจะมีฟีเจอร์ Data-binding ที่ทรงพลังหรือมีแอนิเมชันที่สวยงามอลังการแค่ไหน แต่ถ้าผู้ใช้เลื่อนหน้าจอแล้วกระตุก ประสบการณ์การใช้งาน (UX) ก็จะติดลบทันที

เพื่อที่จะหาว่าความกระตุกนั้นมาจากไหน ในเครื่องมือ **Application Timeline** ของ Visual Studio จึงมีส่วนกราฟที่เรียกว่า **Visual Throughput (FPS)** ซึ่งเปรียบเสมือน "เครื่องวัดชีพจร" ที่คอยบอกเราว่า หน้าจอของเรากำลังเต้นถูกจังหวะอยู่หรือไม่ วันนี้พี่จะพามาเจาะลึกกันครับว่ากราฟนี้ทำงานอย่างไรในสถาปัตยกรรมของ WPF!

## 3. 📖 เนื้อหาหลัก (Core Concept)
ในบริบทที่กว้างขึ้นของการวิเคราะห์แอปพลิเคชันผ่าน **Application Timeline** ส่วนของ **Visual Throughput (FPS)** คือกราฟที่แสดงจำนวนเฟรมต่อวินาทีที่ระบบสามารถวาด (Render) ออกมาได้ตลอดช่วงอายุการทำงานของแอปพลิเคชัน 

ความมหัศจรรย์ของเครื่องมือนี้คือ มันไม่ได้แสดงแค่ค่า FPS รวมๆ แบบผิวเผิน แต่มันแยกชีพจรการทำงานออกเป็น **2 เธรดหลัก (Threads)** ที่ทำงานประสานกันอยู่เบื้องหลัง ซึ่งถ้าน้องๆ เอาเมาส์ไปชี้ที่กราฟ มันจะแสดง Tooltip บอกค่า FPS ของทั้งสองส่วนนี้ ณ เวลานั้นๆ อย่างชัดเจนครับ:

*   **UI Thread (เธรดส่วนติดต่อผู้ใช้):** เธรดนี้เปรียบเสมือน "พ่อครัวใหญ่" มีหน้าที่จัดการลอจิกโค้ด C# ของเรา, การคำนวณ Data-binding, และการจัดวางโครงสร้างหน้าจอ (Layout) หากพ่อครัวทำอาหารช้า (รันลอจิกหนัก) ค่า FPS ของ UI Thread ก็จะตก 
*   **Composition Thread (เธรดการประกอบภาพ):** เธรดนี้คือ "เด็กเสิร์ฟสุดเหาะ" ซึ่งเป็นเธรดผู้ช่วยที่ WPF สร้างขึ้นมาจัดการงานเบื้องหลัง มีหน้าที่นำพื้นผิว (Graphic Textures) ต่างๆ มาประกอบรวมกันแล้วส่งพัสดุนี้ไปให้การ์ดจอ (GPU) วาดลงหน้าจอ การมีเธรดนี้ทำให้แอนิเมชันบางอย่างยังคงลื่นไหลได้ แม้ UI Thread จะกำลังยุ่งอยู่ก็ตาม

โดยปกติแล้ว WPF จะพยายามรักษาเฟรมเรตเป้าหมายไว้ที่ **60 FPS** เสมอเพื่อให้ภาพดูลื่นไหลที่สุด แต่ถ้าน้องๆ สั่งให้แอปทำแอนิเมชันซับซ้อนหลายๆ ตัวพร้อมกันจน CPU หรือ GPU รับไม่ไหว ค่า FPS ในกราฟ Visual Throughput จะแสดงอาการ "ร่วง (Drop)" ทันที ซึ่งเป็นสัญญาณเตือนให้เราต้องเข้าไปจัดการครับ

{{< figure src="visual-throughput-fps-timeline-diagram.jpg" alt="System Flow เปรียบเทียบการทำงานระหว่าง UI Thread และ Composition Thread ที่ส่งผลต่อ Visual Throughput" >}}

## 4. 💻 ตัวอย่างโค้ด (Code Example)
เมื่อเราดูกราฟ Visual Throughput แล้วพบว่าแอปกระตุกเพราะเฟรมเรตตก (สมมติว่าตกลงมาเหลือ 20-30 FPS) วิธีแก้ปัญหาแบบ Clean Code อย่างหนึ่งคือ **"การลดภาระของระบบ"** ด้วยการสั่งจำกัดเฟรมเรต (Desired Frame Rate) ในจุดที่ไม่จำเป็นต้องลื่นไหลถึง 60 FPS ครับ

นี่คือตัวอย่างการใช้ `Timeline.DesiredFrameRate` เพื่อปรับลดเฟรมเรตของแอนิเมชันให้เหลือเพียง 30 FPS เพื่อประหยัดทรัพยากร CPU/GPU:

**ฝั่ง XAML:**
```xml
<Window x:Class="WpfFpsDemo.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Visual Throughput Optimization" Height="300" Width="400">
    <Canvas ClipToBounds="True">
        <Ellipse Name="myEllipse" Fill="LimeGreen" Width="50" Height="50" Canvas.Left="0" Canvas.Top="100"/>
        
        <Canvas.Triggers>
            <EventTrigger RoutedEvent="Window.Loaded">
                <BeginStoryboard>
                    <!-- เราปรับลด Frame Rate ของ Storyboard นี้เหลือ 30 FPS 
                         เพื่อลดภาระของ Composition Thread และ GPU ในการเรนเดอร์ -->
                    <Storyboard Timeline.DesiredFrameRate="30">
                        <!-- แอนิเมชันเคลื่อนที่วงกลมจากซ้ายไปขวา -->
                        <DoubleAnimation 
                            Storyboard.TargetName="myEllipse" 
                            Storyboard.TargetProperty="(Canvas.Left)" 
                            From="0" To="350" Duration="0:0:5" 
                            RepeatBehavior="Forever" AutoReverse="True"/>
                    </Storyboard>
                </BeginStoryboard>
            </EventTrigger>
        </Canvas.Triggers>
    </Canvas>
</Window>
```
*(เมื่อรันโค้ดนี้และวัดผลด้วย Application Timeline น้องๆ จะเห็นกราฟ Visual Throughput วิ่งเกาะอยู่ที่เส้น 30 FPS อย่างสม่ำเสมอ ช่วยคืนทรัพยากรให้ระบบเอาไปทำอย่างอื่นได้ครับ)*

## 5. 🛡️ ข้อควรระวัง / Best Practices
ในการวิเคราะห์กราฟ Visual Throughput พี่มีข้อแนะนำระดับ Senior Dev ดังนี้ครับ:

*   **วิเคราะห์แบบ Release Mode เสมอ:** การใช้ Application Timeline (และเครื่องมือ Diagnostic อื่นๆ) เพื่อดู FPS ควรตั้งค่าโปรเจกต์เป็น `Release` ไม่ใช่ `Debug` เพื่อไม่ให้ตัว Debugger ของ Visual Studio มาดึงประสิทธิภาพจนเฟรมเรตตกเกินจริง
*   **แยกแยะให้ออกว่าใครคือตัวการ:** 
    *   ถ้า **UI Thread FPS** ตก แต่ **Composition Thread FPS** ยังสูง: แสดงว่าน้องๆ อาจจะเขียนโค้ด C# คำนวณหนักเกินไปบนหน้าจอ หรือโหลด Control จำนวนมากพร้อมกัน (Layout bottleneck)
    *   ถ้า **Composition Thread FPS** ตก: แสดงว่า GPU ทำงานหนักเกินไป อาจจะมาจาก Effect (เช่น DropShadow), 3D Model ที่ซับซ้อน, หรือมีพื้นที่โปร่งใสซ้อนทับกันมากเกินไปครับ
*   **ใช้ Hardware Acceleration อย่างชาญฉลาด:** แม้ WPF จะผลักภาระไปให้ Composition Thread ทำงานคู่กับ GPU เพื่อเพิ่มความลื่นไหล แต่ฟีเจอร์บางอย่างก็ยังบังคับรันบนซอฟต์แวร์ (Software Rendering) ซึ่งกินแรง CPU ควรอ่านกราฟควบคู่ไปกับ "UI Thread Utilization" เสมอเพื่อหาต้นตอครับ

## 6. 🏁 สรุป (Conclusion & CTA)
กราฟ **Visual Throughput (FPS)** ในเครื่องมือ Application Timeline คือกุญแจสำคัญที่ทำให้นักพัฒนามองเห็น "ความสมูท" ของแอปพลิเคชันได้อย่างเป็นรูปธรรม การที่ WPF ออกแบบสถาปัตยกรรมให้แยก **UI Thread** ออกจาก **Composition Thread** ถือเป็นมนต์ขลังที่ทำให้ Desktop Application ทันสมัยและตอบสนองได้ดีเยี่ยม เมื่อเรารู้จุดที่เฟรมเรตตก เราก็สามารถเข้าไปจูนโค้ด ลดการใช้เอฟเฟกต์ หรือกำหนด `DesiredFrameRate` ให้เหมาะสมได้ครับ!

ในบทความถัดไป เราจะมาเจาะลึกถึงส่วน "Timeline Details" ที่อยู่ด้านล่างสุดของกราฟ ว่ามันช่วยเราชี้เป้า Control ที่ทำให้แอปช้าได้อย่างไร รอติดตามนะครับ!

---

**ต้องการที่ปรึกษาและพัฒนาระบบ Automation ให้กับโรงงานของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและติดตั้งระบบแบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p