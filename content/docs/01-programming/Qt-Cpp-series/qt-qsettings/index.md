---
title: "ตอนที่ 15: QSettings การบันทึกการตั้งค่าแอปพลิเคชัน"
weight: 1615
date: 2026-03-20
author: วิสิทธิ์ แผ้วกระโทก
tags: ["C++", "Qt5", "Qt6", "QSettings", "GUI", "Configuration"]
---
{{< figure src="qt-qsettings-cover.jpg" alt="รูปปกบทความ QSettings การบันทึกการตั้งค่าแอปพลิเคชัน" >}}

## 1. 🎯 ตอนที่ 15: QSettings การบันทึกการตั้งค่าแอปพลิเคชัน

## 2. 📖 เปิดฉาก (The Hook)
สวัสดีครับน้องๆ สถาปนิกซอฟต์แวร์ทุกคน! กลับมาลุยกันต่อในซีรีส์ **ลุยโปรเจกต์ Cross-Platform GUI ด้วย C++ และ Qt** ครับ

ถ้าน้องๆ เคยโหลดแอปพลิเคชันมาใช้งาน แล้วต้องมานั่งจัดหน้าต่าง ลาก Tool Bar ไปไว้ด้านข้าง เลือกธีมสีเป็น Dark Mode... แต่พอเผลอปิดโปรแกรมแล้วเปิดใหม่ ทุกอย่างกลับกลายเป็นค่าเริ่มต้น (Default) หมดเลย! ความรู้สึกตอนนั้นคือหงุดหงิดใช่ไหมครับ? นั่นแหละครับคือหายนะทาง User Experience (UX) ที่โปรแกรมเมอร์อย่างเราต้องหลีกเลี่ยงให้ได้

ในอดีต ถ้าเราจะเขียน C++ เพื่อจำค่าเหล่านี้ เราต้องมานั่งปวดหัวกับการเขียนโค้ดเปิด/ปิดไฟล์ Text, หรือถ้าทำบน Windows ก็ต้องไปเรียกใช้ Win32 API เพื่อเจาะเข้า Registry อันแสนซับซ้อน แต่ไม่ต้องห่วงครับ! สถาปนิกของ Qt ได้สร้าง "ตู้เซฟแห่งความทรงจำ" ที่ชื่อว่า **`QSettings`** เอาไว้ให้เราแล้ว มันสามารถจัดการบันทึกค่าทุกอย่างลงในระบบปฏิบัติการได้อย่างแนบเนียนและเป็น Cross-platform อย่างแท้จริง วันนี้พี่จะพาไปเจาะลึกวิชานี้กันครับ!

## 3. 🧠 แก่นวิชา (Core Concepts)
**`QSettings`** เป็นคลาสที่เกิดมาเพื่อจัดการกับข้อมูลการตั้งค่า (Settings/Preferences) โดยมันมีกลไกอัจฉริยะที่ทำงานแตกต่างกันไปตาม OS ที่มันไปรันอยู่ (Platform-dependent backend) ดังนี้ครับ:

*   **บน Windows:** ข้อมูลจะถูกบันทึกลงใน **System Registry** (เช่น `HKEY_CURRENT_USER\Software\...`)
*   **บน macOS:** จะบันทึกผ่าน API ที่ชื่อว่า **Core Foundation Preferences** (ไฟล์ `.plist`)
*   **บน UNIX/Linux:** จะบันทึกเป็นไฟล์ข้อความธรรมดา (Text files) เช่น `.conf` หรือ `.ini`

**โครงสร้างการจัดเก็บแบบ Key-Value Pair**
การทำงานของ `QSettings` คล้ายกับการใช้งานระบบ Dictionary (หรือ `QMap`) คือเราจะต้องกำหนด **"คีย์ (Key)"** (เปรียบเสมือนชื่อโฟลเดอร์หรือชื่อตัวแปร) คู่กับ **"ค่า (Value)"** โดยค่าที่บันทึกนั้นจะถูกห่อหุ้มด้วยคลาส **`QVariant`** ซึ่งหมายความว่ามันสามารถเก็บได้ตั้งแต่ตัวเลข `int`, `bool`, ข้อความ `QString`, ไปจนถึงพิกัดหน้าจออย่าง `QRect` หรือรูปภาพเลยทีเดียว!

นอกจากนี้ เรายังสามารถจัดระเบียบ Key ให้เป็นหมวดหมู่ (Hierarchy) ได้เหมือนการสร้างโฟลเดอร์ย่อย โดยใช้คำสั่ง `beginGroup()` และ `endGroup()` ครับ

{{< figure src="qt-qsettings-diagram.jpg" alt="แผนภาพการทำงานแบบ Cross-platform ของ QSettings" >}}

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code)
ในการใช้งานจริง เรามักจะเขียนฟังก์ชัน `writeSettings()` เอาไว้เรียกตอนที่โปรแกรมกำลังจะปิด (ใน `closeEvent()`) และเขียนฟังก์ชัน `readSettings()` เอาไว้เรียกตอนเปิดโปรแกรม (ใน Constructor) ครับ มาดูตัวอย่างกัน:

```cpp
#include <QApplication>
#include <QMainWindow>
#include <QSettings>
#include <QCloseEvent>
#include <QDebug>

class MainWindow : public QMainWindow {
    // ...
protected:
    void closeEvent(QCloseEvent *event) override {
        writeSettings(); // บันทึกค่าก่อนหน้าต่างจะถูกปิด
        event->accept();
    }

private:
    void writeSettings() {
        // 1. ระบุชื่อบริษัท (Organization) และชื่อแอปพลิเคชัน (Application)
        QSettings settings("WP_Solution", "MyAwesomeApp");

        // 2. บันทึกข้อมูลตำแหน่งและขนาดของหน้าต่าง (Geometry)
        settings.setValue("geometry", saveGeometry());
        
        // 3. สร้างหมวดหมู่ (Group) สำหรับการตั้งค่าทั่วไป
        settings.beginGroup("Preferences");
        settings.setValue("theme", "Dark");             // บันทึก String
        settings.setValue("showGrid", true);            // บันทึก Boolean
        settings.endGroup(); // ปิดหมวดหมู่
    }

    void readSettings() {
        QSettings settings("WP_Solution", "MyAwesomeApp");

        // 1. คืนค่าตำแหน่งและขนาดของหน้าต่าง 
        // สังเกตการแปลงค่า QVariant กลับมาเป็น QByteArray
        QByteArray geometry = settings.value("geometry").toByteArray();
        restoreGeometry(geometry);

        // 2. อ่านค่าการตั้งค่าทั่วไป (ถ้าไม่มีค่าที่เคยเซฟไว้ จะใช้ Default Value ในพารามิเตอร์ที่ 2)
        settings.beginGroup("Preferences");
        QString theme = settings.value("theme", "Light").toString(); // ค่าเริ่มต้นคือ Light
        bool isGridVisible = settings.value("showGrid", false).toBool();
        settings.endGroup();

        qDebug() << "Theme loaded:" << theme;
    }
};
```
*(อ้างอิงวิธีการจากโครงสร้างคลาส QSettings ในหนังสือ C++ GUI Qt4 编程)*

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips)
ในฐานะ Senior พี่มี 3 เคล็ดวิชาในการใช้งาน `QSettings` ให้โปรที่สุดมาฝากครับ:

1.  **ใช้ Default Value เป็นตาข่ายรองรับเสมอ:** 
    ในการเรียกคำสั่ง `settings.value("key", defaultValue)` น้องๆ **"ต้อง"** ใส่ `defaultValue` (พารามิเตอร์ตัวที่ 2) ไว้เสมอครับ เพราะในครั้งแรกที่ผู้ใช้รันโปรแกรม มันจะยังไม่มี Key นี้อยู่ใน Registry การใส่ค่าเริ่มต้น (เช่น กำหนดให้เป็น `false` หรือ `"Light"`) จะช่วยให้โปรแกรมไม่พังและทำงานด้วยค่ามาตรฐานได้อย่างปลอดภัย
2.  **เวทมนตร์แห่ง `saveGeometry()` และ `restoreGeometry()`:**
    สมัยก่อนการจำพิกัดหน้าจอ เราต้องมานั่งเซฟค่า แกน X, แกน Y, ความกว้าง, ความสูง เอาเอง แต่ใน Qt เรามีฟังก์ชัน `saveGeometry()` ที่เหนือชั้นกว่านั้นครับ! มันฉลาดพอที่จะจัดการกับปัญหาระบบ "หลายจอภาพ (Multi-monitor)" ได้ด้วย ถ้ายูสเซอร์เคยลากหน้าต่างไปไว้จอที่ 2 แล้วเซฟ พอเปิดมาใหม่มันก็จะเด้งไปที่จอ 2 อย่างแม่นยำ!
3.  **ตั้งค่า Organization แบบ Global (Global Configuration):**
    ถ้าน้องๆ มีหน้าต่างหรือคลาสหลายตัวที่ต้องเรียกใช้ `QSettings` การมานั่งพิมพ์ `QSettings("WP_Solution", "MyApp")` ทุกครั้งคงน่ารำคาญ พี่แนะนำให้ไปตั้งค่าที่ไฟล์ `main.cpp` เลยครับ:
    ```cpp
    QApplication::setOrganizationName("WP_Solution");
    QApplication::setApplicationName("MyAwesomeApp");
    ```
    หลังจากนั้น ไม่ว่าน้องๆ จะอยู่ที่ไหนในโค้ด C++ แค่พิมพ์ `QSettings settings;` โดดๆ แบบไม่มีพารามิเตอร์ มันจะรู้ทันทีว่าต้องดึงค่าจาก Registry โฟลเดอร์ไหน! สะอาดและโคตรคลีนครับ!

## 6. 🏁 บทสรุป (To be continued...)
ด้วยพลังของ **`QSettings`** ตอนนี้โปรแกรม C++ GUI ของน้องๆ ก็มีความทรงจำแล้วครับ! ไม่ว่าผู้ใช้จะปรับแต่ง UI ไว้อย่างไร โปรแกรมก็จะสามารถฟื้นคืนสภาพเดิมได้อย่างสมบูรณ์แบบ ช่วยยกระดับความรู้สึกเป็นมืออาชีพ (Professional Feel) ให้กับซอฟต์แวร์ของเราไปอีกขั้น

ในตอนต่อไป เราจะมาเจาะลึกระบบการจัดการข้อมูลที่ใหญ่ขึ้น อย่างการคุยกับ **Database (SQL)** ผ่านคลาส `QSqlDatabase` กันครับ เตรียมตัวรับมือกับข้อมูลจำนวนมหาศาลกันได้เลย แล้วพบกันครับ!

---
**ต้องการที่ปรึกษาด้านการออกแบบสถาปัตยกรรมซอฟต์แวร์ C++ หรือระบบ Cross-Platform GUI ให้กับองค์กรของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและพัฒนาซอฟต์แวร์แบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p