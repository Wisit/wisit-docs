---
title: "ตอนที่ 13: Workshop 1 - สร้างโปรแกรม Text Editor (Part 3: การจัดการ Font และ Color)"
weight: 1613
date: 2026-03-20
author: วิสิทธิ์ แผ้วกระโทก
tags: ["C++", "Qt5", "Qt6", "QFontDialog", "QColorDialog", "Clipboard"]
---
{{< figure src="qt-workshop-text-editor-part3-cover.jpg" alt="รูปปกบทความ Workshop 1 สร้างโปรแกรม Text Editor (Part 3)" >}}

## 1. 🎯 ตอนที่ 13: Workshop 1 - สร้างโปรแกรม Text Editor (Part 3: การจัดการ Font และ Color)

## 2. 📖 เปิดฉาก (The Hook)
สวัสดีครับน้องๆ สถาปนิกซอฟต์แวร์ทุกคน! ยินดีต้อนรับเข้าสู่โค้งสุดท้ายของมหากาพย์ **Workshop สร้างโปรแกรม Text Editor ด้วย C++ และ Qt** ครับ!

จากสองตอนที่ผ่านมา เราได้สร้างโครงสร้าง UI ที่สวยงาม และทำให้โปรแกรมของเรามี "สมอง" ในการบันทึกและเปิดไฟล์ได้แล้ว แต่ Text Editor ที่ดีจะขาดการปรับแต่งหน้าตาไปไม่ได้เลยครับ ลองนึกภาพว่าถ้าเราต้องเขียนโค้ดเพื่อสร้าง "หน้าต่างเลือกสี (Color Picker)" หรือ "หน้าต่างเลือกฟอนต์ (Font Selector)" ขึ้นมาเองตั้งแต่ศูนย์ เราคงต้องเสียเวลาเขียน C++ เป็นพันๆ บรรทัดแน่ๆ 

แต่ในโลกของ Qt เรามีทางลัดที่เรียกว่า **Standard Dialogs** ซึ่งเป็นบะหมี่กึ่งสำเร็จรูปที่ Qt เตรียมไว้ให้ วันนี้พี่จะพาไปร่ายมนต์เรียกใช้ `QFontDialog` และ `QColorDialog` เพื่อให้ผู้ใช้เปลี่ยนฟอนต์และสีได้ดั่งใจ พร้อมกับปิดท้ายด้วยกลไกการจัดการคลิปบอร์ด (Clipboard) เพื่อทำระบบ Copy/Paste ให้สมบูรณ์แบบครับ เตรียม IDE ให้พร้อมแล้วลุยกันเลย!

## 3. 🧠 แก่นวิชา (Core Concepts)
ในการปรับแต่งการแสดงผลและจัดการข้อมูลชั่วคราว เราจะพึ่งพา 3 คลาสระดับพระเอกของ Qt ดังนี้ครับ:

*   **`QFontDialog` (หน้าต่างเลือกฟอนต์):** เป็น Standard Dialog ที่ให้ผู้ใช้เลือกชื่อฟอนต์, ขนาด (Size), ความหนา (Bold), และความเอียง (Italic) โดยเราจะใช้ฟังก์ชันแบบ Static ที่ชื่อว่า `getFont()` ซึ่งมันจะคืนค่ากลับมาเป็นออบเจกต์ `QFont` พร้อมกับรับค่าพารามิเตอร์อ้างอิง `bool *ok` เพื่อเช็กว่าผู้ใช้กด OK หรือ Cancel
*   **`QColorDialog` (หน้าต่างเลือกสี):** ใช้สำหรับดึงหน้าต่างเลือกสีของระบบปฏิบัติการขึ้นมา โดยใช้ฟังก์ชัน `getColor()` ค่าที่คืนกลับมาจะเป็นออบเจกต์ `QColor` ซึ่งเราต้องใช้ฟังก์ชัน `isValid()` เพื่อตรวจสอบความถูกต้องก่อนนำไปใช้งาน
*   **`QClipboard` และระบบ Copy/Paste:** ตามหลักการแล้ว การเข้าถึงคลิปบอร์ดของระบบปฏิบัติการจะต้องเรียกผ่าน `QApplication::clipboard()` แต่สำหรับ Widget พื้นฐานอย่าง `QPlainTextEdit` นั้น สถาปนิกของ Qt ได้เตรียม Slot สำเร็จรูปอย่าง `copy()`, `cut()`, และ `paste()` เอาไว้ให้เราหมดแล้ว! เราจึงแทบไม่ต้องเขียนลอจิกระดับล่างเองเลย

{{< figure src="qt-workshop-text-editor-part3-diagram.jpg" alt="แผนภาพแสดงการทำงานของ QFontDialog, QColorDialog และ Clipboard" >}}

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code)
เรามาดูวิธีการนำ 3 คลาสนี้ไปผูกกับเมนูหรือปุ่ม (QAction) ที่เราออกแบบไว้ใน Text Editor กันครับ (เขียนใน `mainwindow.cpp`)

**1. การเปลี่ยนฟอนต์ด้วย QFontDialog**
```cpp
#include <QFontDialog>
#include <QColorDialog>
#include <QPalette>

void MainWindow::on_actionFont_triggered()
{
    // 1. ดึงฟอนต์ปัจจุบันที่ใช้อยู่ใน textEdit ออกมาเป็นค่าเริ่มต้น
    QFont iniFont = ui->plainTextEdit->font();
    
    bool ok = false; // ตัวแปรสำหรับรับสถานะการกดปุ่ม
    
    // 2. เรียกหน้าต่างเลือกฟอนต์ โดยส่ง pointer &ok เข้าไป
    QFont font = QFontDialog::getFont(&ok, iniFont, this);
    
    // 3. ตรวจสอบว่าผู้ใช้กด OK (ok == true) ไม่ใช่กด Cancel
    if (ok) {
        // นำฟอนต์ใหม่ไปประยุกต์ใช้กับ QPlainTextEdit
        ui->plainTextEdit->setFont(font);
    }
}
```

**2. การเปลี่ยนสีข้อความด้วย QColorDialog และ QPalette**
```cpp
void MainWindow::on_actionColor_triggered()
{
    // 1. ดึง Palette (จานสี) ปัจจุบันของ plainTextEdit ออกมา
    QPalette pal = ui->plainTextEdit->palette();
    
    // 2. ดึงสีของข้อความ (Text) ปัจจุบันมาเป็นค่าเริ่มต้นให้ Dialog
    QColor iniColor = pal.color(QPalette::Text);
    
    // 3. เรียกหน้าต่างเลือกสี
    QColor color = QColorDialog::getColor(iniColor, this, "เลือกสีข้อความ");
    
    // 4. ตรวจสอบว่าสีที่ได้มาถูกต้อง (ผู้ใช้กด OK)
    if (color.isValid()) {
        // เปลี่ยนสีในช่อง QPalette::Text ของจานสี
        pal.setColor(QPalette::Text, color);
        // นำจานสีใหม่ไปติดตั้งกลับให้ QPlainTextEdit
        ui->plainTextEdit->setPalette(pal);
    }
}
```

**3. การจัดการคลิปบอร์ด (Copy / Cut / Paste)**
```cpp
// ลอจิกการทำงานนั้นง่ายแสนง่าย เพราะ QPlainTextEdit จัดการให้หมดแล้ว!
void MainWindow::on_actionCopy_triggered()
{
    ui->plainTextEdit->copy();
}

void MainWindow::on_actionCut_triggered()
{
    ui->plainTextEdit->cut();
}

void MainWindow::on_actionPaste_triggered()
{
    ui->plainTextEdit->paste();
}
```

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips)
ในการใช้งาน Dialogs เหล่านี้ พี่มีข้อควรระวัง 2 จุดที่ Senior มักจะใช้ตรวจสอบโค้ดของ Junior เสมอครับ:

1.  **ต้องส่งค่าเริ่มต้น (Initial Value) ให้ Dialog เสมอ:** 
    สังเกตในโค้ดพี่จะใช้ `ui->plainTextEdit->font()` หรือ `pal.color()` ดึงค่าปัจจุบันส่งไปให้ Dialog ก่อนเสมอ ถ้าน้องไม่ส่งไป เวลาผู้ใช้กดเปลี่ยนสี หน้าต่างเลือกสีมันจะเด้งไปที่สีขาวหรือดำโดยอัตโนมัติ ซึ่งจะทำให้ User Experience (UX) ดูแย่มากครับ
2.  **ความแตกต่างของระบบ Check Validity:** 
    น้องๆ อาจจะสงสัยว่าทำไม `QFontDialog` ถึงใช้ตัวแปร `bool *ok` แต่ `QColorDialog` กลับใช้ `color.isValid()`? 
    คำตอบคือ สถาปัตยกรรมของ `QFont` ไม่มีสถานะ "ฟอนต์ว่างเปล่า" (Invalid Font) อยู่ในตัวเองครับ มันต้องมีฟอนต์เสมอ Qt เลยต้องใช้ตัวแปรภายนอก (`ok`) มาช่วยเช็กว่าเกิดจากการกดปุ่ม Cancel หรือไม่ ในขณะที่คลาส `QColor` ถูกออกแบบมาให้มีสถานะ Invalid อยู่ในตัวเอง (เช็กผ่าน `isValid()`) จึงไม่ต้องใช้ตัวแปร `ok` มาช่วยครับ นี่คือเกร็ดความรู้ระดับโครงสร้างคลาสเลยทีเดียว!

## 6. 🏁 บทสรุป (To be continued...)
จบลงไปอย่างสมบูรณ์แบบครับสำหรับ **Workshop 1: สร้างโปรแกรม Text Editor**! ตอนนี้โปรแกรมของเราไม่เพียงแค่จัดการไฟล์ได้ แต่ยังรองรับการปรับแต่ง UI อย่างการเปลี่ยนฟอนต์และสี รวมถึงมีระบบ Copy/Paste ที่สมบูรณ์พร้อมใช้งานจริงแล้วครับ 

ความรู้ตลอด 13 ตอนที่ผ่านมาถือเป็น "แก่น" ของการพัฒนา Cross-Platform GUI ด้วย C++ หวังว่าน้องๆ จะเห็นภาพความทรงพลังของสถาปัตยกรรม Qt มากขึ้นนะครับ ในบทต่อไปเราจะเริ่มขยับสเกลไปสู่การทำ **กราฟิก 2 มิติ (2D Graphics และ QPainter)** ใครที่อยากทำโปรแกรมวาดรูป หรือทำ Dashboard สวยๆ ห้ามพลาดเด็ดขาด แล้วพบกันครับ!

---
**ต้องการที่ปรึกษาด้านการออกแบบสถาปัตยกรรมซอฟต์แวร์ C++ หรือระบบ Cross-Platform GUI ให้กับองค์กรของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและพัฒนาซอฟต์แวร์แบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p