---
title: "สถาปัตยกรรมระดับแกนกลางของ WPF: ผ่าโครงสร้างเพื่อเข้าใจความทรงพลัง"
weight: 40
date: 2026-03-09
author: วิสิทธิ์ แผ้วกระโทก
tags: ["WPF", "Architecture", "DirectX", "milcore", "Retained Mode"] 
---

{{< figure src="wpf-core-architecture-cover.jpg" alt="ภาพหน้าปกบทความ WPF Core Architecture" >}}

## 1. 🎯 ชื่อบทความ (Title): สถาปัตยกรรมระดับแกนกลางของ WPF: ผ่าโครงสร้างเพื่อปลดล็อกพลังแห่ง UI

## 2. 👋 เกริ่นนำ (Introduction)
สวัสดีครับน้องๆ และเพื่อนนักพัฒนาทุกคน! วันนี้พี่จะพามาเจาะลึกในระดับ "แกนกลาง (Core Concepts)" ของ **Windows Presentation Foundation (WPF)** กันครับ 

เวลาที่เราสร้างแอปพลิเคชันขึ้นมาสักตัว ลองเปรียบเทียบว่าแอปของเราคือ "โรงงานอัจฉริยะ" ครับ ในอดีต (ยุค Windows Forms หรือ GDI/GDI+) ผู้จัดการโรงงาน (โปรแกรมเมอร์) ต้องลงไปสั่งงานและประกอบชิ้นส่วนเองทุกจุดทีละพิกเซล (Immediate Mode) ซึ่งทำให้เสียเวลาและเหนื่อยมาก 

แต่พอมาเป็นยุคของ WPF สถาปัตยกรรมถูกออกแบบใหม่หมดให้เป็นแบบแบ่งชั้น (Multilayered Architecture) เปรียบเสมือนเรามี "ทีมผู้บริหาร" (Managed Code) ที่คอยรับคำสั่งและวางแผนงาน จากนั้นส่งต่อให้ "หัวหน้าคนงานเครื่องจักรกลหนัก" (Unmanaged Code) ที่ไปสั่งการ "เครื่องจักรประสิทธิภาพสูง" (DirectX/GPU) ให้ทำงานแทนแบบอัตโนมัติ การเข้าใจโครงสร้างนี้จะเปี่ยมไปด้วยมนต์ขลังที่ช่วยให้น้องๆ รีดประสิทธิภาพของ WPF ออกมาได้สูงสุดโดยไม่สะดุดเลยครับ!

## 3. 📖 เนื้อหาหลัก (Core Concept)
จากเอกสารและคลังความรู้เชิงลึก สถาปัตยกรรมของ WPF ถูกแบ่งออกเป็นชั้นต่างๆ เพื่อให้การทำงานมีประสิทธิภาพ โดยแยกความรับผิดชอบกันอย่างชัดเจนดังนี้ครับ:

*   **The Managed WPF API (ทีมผู้บริหารชั้นบน):** เขียนด้วยภาษา C# ซึ่งทำงานบน .NET CLR ทำให้เราเขียนโค้ดได้อย่างปลอดภัยและเป็นระเบียบ ชั้นนี้ประกอบด้วย 3 Assembly หลัก:
    *   **PresentationFramework.dll:** เป็นชั้นบนสุดที่เราเรียกใช้บ่อยที่สุด เก็บพวกคอนโทรลต่างๆ (Button, Label, Panels), ระบบ Data Binding, และ Styles
    *   **PresentationCore.dll:** ชั้นที่เก็บคลาสพื้นฐานระดับล่าง เช่น `UIElement` และ `Visual` ซึ่งเป็นรากฐานของการวาดและระบบ Event ในหน้าจอ
    *   **WindowsBase.dll:** เก็บส่วนประกอบที่ลึกที่สุด เช่น `DispatcherObject` (จัดการ Threading) และ `DependencyObject` (จัดการระบบ Property อัจฉริยะของ WPF)
*   **Media Integration Layer (MIL) / milcore.dll (หัวหน้าคนงานชั้นกลาง):** ชิ้นส่วนนี้ถูกเขียนด้วย **Unmanaged Code** (C/C++) เพื่อให้สามารถเชื่อมต่อกับ DirectX ได้อย่างแนบแน่นและดึงประสิทธิภาพมาได้สูงสุด `milcore` จะรับเอาคำสั่งโครงสร้างต้นไม้ภาพ (Visual Tree) จากฝั่ง Managed มาประมวลผลเพื่อส่งต่อ
*   **DirectX (เครื่องจักรกลหนักชั้นล่าง):** ทุกๆ อย่างใน WPF ไม่ว่าจะเป็น 2D, 3D, ข้อความธรรมดา หรือปุ่ม จะถูกแปลงเป็นรูปสามเหลี่ยมเรขาคณิต (Triangles) และพื้นผิว (Textures) แล้วถูกโยนให้ DirectX ประมวลผลบนการ์ดจอ (GPU) ล้วนๆ ครับ
*   **Retained Mode Graphics:** WPF ทิ้งระบบ Immediate Mode ที่แอปต้องวาดจอเองเวลาถูกบัง (เช่น `WM_PAINT`) ไปใช้แบบ **Retained Mode** แทน นั่นคือเรามีหน้าที่แค่ "อธิบาย" ว่าอยากได้หน้าจอแบบไหน ส่วนการสั่งวาด การจำสถานะกราฟิก และการ Refresh หน้าจอ ปล่อยให้ระบบจัดการเองทั้งหมดครับ

{{< figure src="wpf-core-architecture-diagram.jpg" alt="ภาพ System Flow การทำงานของสถาปัตยกรรม WPF แบ่งเป็น Managed, Unmanaged และ DirectX" >}}

## 4. 💻 ตัวอย่างโค้ด (Code Example)
เพื่อให้เห็นภาพว่าสถาปัตยกรรมเหล่านี้ทำงานร่วมกันอย่างไร พี่จะขอยกตัวอย่างการข้ามผ่าน `PresentationFramework` (Controls ทั่วไป) แล้วมุดลงไปคุยกับ `PresentationCore` โดยตรงผ่านคลาส `DrawingVisual` ครับ วิธีนี้เหมาะมากเวลาที่เราต้องการวาดกราฟิกจำนวนเป็นหมื่นๆ ชิ้นโดยไม่กิน Memory ของระบบ (เช่น กราฟ หรือ แผนที่) เพราะเราตัด Overhead ของ Managed Control ออกไป:

```csharp
using System.Windows;
using System.Windows.Media;

namespace WpfCoreArchitecture
{
    // สร้างคลาสของเราเองที่สืบทอดจาก FrameworkElement เพื่อให้ไปแปะบนหน้าต่างได้
    public class HighPerformanceCanvas : FrameworkElement
    {
        private VisualCollection _visuals;

        public HighPerformanceCanvas()
        {
            // สร้าง Collection สำหรับเก็บ Visual (คลาสจาก PresentationCore.dll)
            _visuals = new VisualCollection(this);
            
            // เรียกฟังก์ชันเพื่อสั่งวาดทันที
            CreateVisuals();
        }

        private void CreateVisuals()
        {
            // สร้าง DrawingVisual ซึ่งเป็น Object ระดับล่าง กินทรัพยากรน้อยมาก
            DrawingVisual visual = new DrawingVisual();
            
            // เปิด DrawingContext เพื่อเริ่มบันทึกคำสั่งวาด (Retained Mode)
            using (DrawingContext dc = visual.RenderOpen())
            {
                // สั่งวาดสี่เหลี่ยม การทำงานนี้จะถูกส่งไปให้ milcore.dll และ DirectX ต่อไป
                dc.DrawRectangle(Brushes.LightBlue, new Pen(Brushes.Blue, 2), 
                                 new Rect(10, 10, 100, 100));
            } // คำสั่งวาดจะถูกจดจำไว้ที่นี่ (Retain) ทันทีที่ using block สิ้นสุด

            // เพิ่มเข้าสู่ Visual Tree
            _visuals.Add(visual);
        }

        // ต้อง Override 2 ตัวนี้เสมอเพื่อให้ระบบ WPF Element Tree มองเห็น Visual ของเรา
        protected override int VisualChildrenCount => _visuals.Count;

        protected override Visual GetVisualChild(int index)
        {
            return _visuals[index];
        }
    }
}
```
*(โค้ดนี้สาธิตให้เห็นว่าการเข้าใจสถาปัตยกรรมระดับล่างอย่าง Visual System สามารถช่วยให้เราควบคุมการทำงานของแอปพลิเคชันได้อย่างมีประสิทธิภาพ)*

## 5. 🛡️ ข้อควรระวัง / Best Practices
จากสถาปัตยกรรมดังกล่าว มีข้อควรระวังสำคัญเพื่อป้องกันไม่ให้เกิดคอขวดหรือแอปแครช (Crash) ครับ:

*   **ระวังเรื่อง Thread Affinity เสมอ:** วัตถุใน WPF เกือบทั้งหมดสืบทอดมาจาก `DispatcherObject` (อยู่ใน `WindowsBase.dll`) ซึ่งมีกฎเหล็กว่า "Thread ไหนสร้าง Thread นั้นเท่านั้นที่มีสิทธิ์แก้ไข" หากเรามีการดึงข้อมูลจาก Database ด้วย Background Thread ห้ามเอาไปยัดใส่ UI ตรงๆ เด็ดขาด ต้องใช้ `Dispatcher.BeginInvoke()` เพื่อส่งงานข้าม Thread เสมอครับ
*   **เลือก Visual ให้เหมาะสมกับงาน:** หากต้องการทำ Control ที่มีการรับค่าผู้ใช้ทั่วไป ให้ใช้คลาสจาก `PresentationFramework` (เช่น `Button`, `UserControl`) แต่หากต้องทำ Data Visualization ที่วาดจุดเป็นหมื่นๆ จุดบนกราฟ ควรลงไปใช้ `DrawingVisual` (ใน `PresentationCore`) เพื่อหลีกเลี่ยง Overhead ทำให้แอปพลิเคชันของเราเร็วขึ้นมหาศาลครับ
*   **ตรวจสอบ RenderCapability.Tier:** แม้ WPF จะพึ่งพา DirectX แต่บนเครื่องรุ่นเก่ามากๆ ที่การ์ดจอไม่แรง WPF จะสลับไปใช้ Software Rendering (CPU) ซึ่งอาจทำให้กระตุกได้ เราสามารถตรวจสอบสเปคได้ผ่าน `RenderCapability.Tier` เพื่อสั่งปิด Effect ซับซ้อนบางอย่างได้ครับ

## 6. 🏁 สรุป (Conclusion & CTA)
เมื่อเราซูมออกมาดูในบริบทที่กว้างขึ้น สถาปัตยกรรมของ WPF เป็นการผสมผสานอย่างชาญฉลาดระหว่างความสะดวกสบายของการเขียนโค้ด .NET (Managed) และความทรงพลังของการ์ดจอ (Unmanaged/DirectX) โครงสร้างแบบ Retained Mode ช่วยปลดล็อกขีดจำกัดเดิมๆ ให้เราสร้าง UI ที่มีทั้งความงาม ลื่นไหล และปรับขนาดได้ทุกมิติครับ! 

หวังว่าทุกคนจะเห็นภาพเบื้องหลังการทำงานของ WPF กันชัดเจนขึ้นนะครับ ในบทความหน้าเราจะเจาะลึกเข้าไปในระบบ Dependency Property กันต่อ เตรียมตัวให้พร้อมครับ!

---
**ต้องการที่ปรึกษาและพัฒนาระบบ Automation ให้กับโรงงานของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและติดตั้งระบบแบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p