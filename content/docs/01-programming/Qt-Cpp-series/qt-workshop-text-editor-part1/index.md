---
title: "ตอนที่ 11: Workshop 1 - สร้างโปรแกรม Text Editor (Part 1: ออกแบบ UI)"
weight: 1611
date: 2026-03-20
author: วิสิทธิ์ แผ้วกระโทก
tags: ["C++", "Qt5", "Qt6", "Workshop", "QPlainTextEdit", "QMainWindow"]
---
{{< figure src="qt-workshop-text-editor-part1-cover.jpg" alt="รูปปกบทความ Workshop 1 สร้างโปรแกรม Text Editor" >}}

## 1. 🎯 ตอนที่ 11: Workshop 1 - สร้างโปรแกรม Text Editor (Part 1: ออกแบบ UI)

## 2. 📖 เปิดฉาก (The Hook)
สวัสดีครับน้องๆ สถาปนิกซอฟต์แวร์ทุกคน! เดินทางกันมาถึงตอนที่ 11 แล้วนะครับในซีรีส์ **ลุยโปรเจกต์ Cross-Platform GUI ด้วย C++ และ Qt** 

ตลอด 10 ตอนที่ผ่านมา เราได้เรียนรู้วรยุทธ์กระบวนท่าต่างๆ ไปมากมาย ไม่ว่าจะเป็นโครงสร้างของ `QMainWindow`, ระบบการสื่อสารอัจฉริยะอย่าง Signals & Slots, การจัดการ Layout, เวทมนตร์ของ `QAction`, และการจัดการ Resource System... ถ้าน้องๆ แค่อ่านทฤษฎีอย่างเดียว วิชาคงไม่ซึมเข้ากระดูกแน่ๆ ครับ!

วันนี้พี่เลยจะพามาทำ **Workshop** ของจริง! เราจะนำชิ้นส่วนความรู้ทั้งหมดที่กระจัดกระจายอยู่ มาประกอบร่างสร้างโปรแกรม **Text Editor** (โปรแกรมแก้ไขข้อความสไตล์ Notepad) ที่สามารถรันได้ทั้งบน Windows, Mac และ Linux! โดยใน Part 1 นี้ เราจะมาโฟกัสที่การขึ้นโครงสถาปัตยกรรมหน้าจอ (UI Design) กันก่อน เตรียม IDE ให้พร้อม แล้วมาลุยโค้ดกันเลยครับ!

## 3. 🧠 แก่นวิชา (Core Concepts)
การจะสร้างโปรแกรมที่มีหน้าตาระดับมืออาชีพ เราจะใช้โครงสร้างสถาปัตยกรรมของ `QMainWindow` เป็นแกนหลักครับ โดยมีพระเอกและนางเอกในงานนี้คือ:

*   **`QMainWindow` (โครงสร้างคฤหาสน์):** เป็นกรอบนอกสุดที่จะช่วยจัดการพื้นที่ให้เราโดยอัตโนมัติ เราจะใช้มันเพื่อวาง Menu Bar (แถบเมนู), Tool Bar (แถบเครื่องมือ) และ Status Bar (แถบสถานะ)
*   **`QPlainTextEdit` (ห้องนั่งเล่นหลัก):** นี่คือ Widget หัวใจสำคัญของ Text Editor ครับ ถ้าน้องๆ จำได้ Qt มี Widget สำหรับจัดการข้อความหลายตัว เช่น `QLineEdit` (บรรทัดเดียว) หรือ `QTextEdit` (หลายบรรทัดแบบ Rich Text ที่รองรับ HTML) แต่สำหรับโปรแกรมที่เน้นพิมพ์โค้ดหรือข้อความธรรมดา (Plain text) อย่าง Notepad สถาปนิกของ Qt แนะนำให้ใช้ **`QPlainTextEdit`** ครับ เพราะมันถูกออกแบบมาให้ประมวลผลข้อความยาวๆ เป็นย่อหน้า (Paragraph / Block) ได้เร็วกว่า และใช้ Memory น้อยกว่ามาก!
*   **การตั้งเป็น Central Widget:** กฎเหล็กของ `QMainWindow` คือมันต้องมี Central Widget เสมอ ในที่นี้เราจะเสก `QPlainTextEdit` ให้เป็นศูนย์กลางของหน้าต่าง โดยมันจะขยายตัวเต็มพื้นที่ว่างที่เหลือโดยอัตโนมัติ

{{< figure src="qt-workshop-text-editor-part1-diagram.jpg" alt="แผนภาพโครงสร้าง UI ของโปรแกรม Text Editor" >}}

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code)
เรามาดูวิธีการเขียน C++ เพื่อขึ้นโครง UI ของ Text Editor กันครับ โค้ดส่วนนี้จะอยู่ในไฟล์ `mainwindow.cpp` ภายใน Constructor ของคลาส `MainWindow` ครับ

```cpp
#include "mainwindow.h"
#include <QPlainTextEdit>
#include <QMenu>
#include <QMenuBar>
#include <QToolBar>
#include <QAction>
#include <QIcon>
#include <QStatusBar>

MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
{
    // 1. กำหนดค่าเริ่มต้นให้หน้าต่าง
    setWindowTitle(tr("WP Text Editor"));
    resize(800, 600); // ตั้งขนาดเริ่มต้น

    // 2. สร้าง QPlainTextEdit และตั้งเป็น Central Widget (ศูนย์กลางของหน้าต่าง)
    QPlainTextEdit *textEdit = new QPlainTextEdit(this);
    setCentralWidget(textEdit); // คำสั่งนี้สำคัญมาก! ถ้าลืม UI จะพังทันที

    // 3. สร้างกลุ่ม QAction สำหรับคำสั่งพื้นฐาน (เปรียบเสมือนรีโมทคอนโทรล)
    QAction *newAct = new QAction(QIcon(":/images/new.png"), tr("สร้างใหม่ (&N)"), this);
    newAct->setShortcut(QKeySequence::New); // ผูกคีย์ลัด Ctrl+N
    newAct->setStatusTip(tr("สร้างไฟล์เอกสารใหม่"));

    QAction *openAct = new QAction(QIcon(":/images/open.png"), tr("เปิดไฟล์ (&O)"), this);
    openAct->setShortcut(QKeySequence::Open);

    QAction *saveAct = new QAction(QIcon(":/images/save.png"), tr("บันทึก (&S)"), this);
    saveAct->setShortcut(QKeySequence::Save);

    QAction *exitAct = new QAction(tr("ออกจากโปรแกรม (&Q)"), this);
    exitAct->setShortcut(tr("Ctrl+Q"));

    // 4. สร้าง Menu Bar ด้านบน และนำ Action ใส่เข้าไป
    QMenu *fileMenu = menuBar()->addMenu(tr("ไฟล์ (&F)"));
    fileMenu->addAction(newAct);
    fileMenu->addAction(openAct);
    fileMenu->addAction(saveAct);
    fileMenu->addSeparator(); // ขีดเส้นคั่นให้สวยงาม
    fileMenu->addAction(exitAct);

    // 5. สร้าง Tool Bar (แถบเครื่องมือ) และนำ Action "ตัวเดิม" มาใส่
    QToolBar *fileToolBar = addToolBar(tr("File Toolbar"));
    fileToolBar->addAction(newAct);
    fileToolBar->addAction(openAct);
    fileToolBar->addAction(saveAct);

    // 6. ตั้งค่า Status Bar ด้านล่างสุด
    statusBar()->showMessage(tr("พร้อมใช้งาน"), 3000);

    // ปล. อย่าลืมเชื่อม Signals & Slots ไว้ใช้งานในตอนหน้านะครับ!
    // connect(exitAct, SIGNAL(triggered()), this, SLOT(close()));
}

MainWindow::~MainWindow()
{
}
```
*(อ้างอิงวิธีการจัดวาง Central Widget, การสร้าง Action และเมนู จากโครงสร้างของ QMainWindow)*

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips)
ในการออกแบบ UI ของ Text Editor ให้ดูเป็นมือโปรระดับ Senior พี่มี 2 เคล็ดลับมาฝากครับ:

1.  **การดักจับสถานะ "ไฟล์ถูกแก้ไข" (Document Modified):** 
    เวลาเราพิมพ์งานใน Notepad ถ้างยังไม่ได้เซฟ จะมีเครื่องหมายดอกจัน `*` โผล่ที่หัวหน้าต่างใช่ไหมครับ? ใน Qt เราทำแบบนั้นได้ง่ายมากๆ! คลาส `QPlainTextEdit` จะมี Signal ที่ชื่อว่า `modificationChanged(bool)` ซึ่งจะถูกเปล่งออกมาเมื่อข้อความมีการเปลี่ยนแปลง น้องๆ สามารถนำ Signal นี้ไป `connect()` เข้ากับ Slot `setWindowModified(bool)` ของหน้าต่าง `QWidget` หลักได้เลย จากนั้นแค่ตั้งชื่อหน้าต่างโดยมี `[*]` ต่อท้าย เช่น `setWindowTitle("Untitled[*]");` ระบบของ Qt จะจัดการซ่อน/แสดงเครื่องหมาย `*` ให้เราแบบอัตโนมัติ! โคตรเจ๋งเลยครับ!
2.  **ทางลัดในการใช้ฟังก์ชัน Edit มาตรฐาน:**
    น้องๆ ไม่ต้องมานั่งเขียนโค้ดลอจิกในการ Copy, Cut, Paste, Undo หรือ Redo เองนะครับ! คลาส `QPlainTextEdit` มีฟังก์ชันเหล่านี้เตรียมไว้ให้หมดแล้วเป็น Public Slots (`copy()`, `cut()`, `paste()`, `undo()`, `redo()`) น้องๆ แค่สร้าง `QAction` บนหน้าจอ แล้วใช้คำสั่ง `connect()` ลากสายไปผูกกับ Slot เหล่านี้ได้เลยทันที งานเสร็จไวแบบไม่ต้องออกแรง!

## 6. 🏁 บทสรุป (To be continued...)
เป็นยังไงกันบ้างครับ? แค่ใช้เวลาเขียนโค้ดไม่กี่บรรทัด เราก็ได้หน้าตาโปรแกรม Text Editor ที่มีทั้ง Menu Bar, Tool Bar และพื้นที่พิมพ์งานที่ขยายสัดส่วนตามหน้าต่างได้อย่างสมบูรณ์แบบ นี่แหละครับคือพลังแห่งสถาปัตยกรรม `QMainWindow` ของ Qt!

แต่โปรแกรมของเราตอนนี้ยังเป็นแค่ "โครงเปล่า" ครับ กดเปิดไฟล์ หรือเซฟไฟล์จริงๆ ยังไม่ได้ ในตอนหน้า (Part 2) พี่จะพาไปเขียนลอจิกด้านหลัง (Business Logic) เราจะนำ **QFileDialog** มาใช้เลือกไฟล์บนระบบปฏิบัติการ และใช้ **QFile** ร่วมกับ **QTextStream** เพื่ออ่านและเขียนไฟล์ข้อความลงฮาร์ดดิสก์จริงๆ กันครับ เตรียมตัวเป็นนักพัฒนาแบบ Full-stack GUI กันได้เลย แล้วพบกันครับ!

---
**ต้องการที่ปรึกษาด้านการออกแบบสถาปัตยกรรมซอฟต์แวร์ C++ หรือระบบ Cross-Platform GUI ให้กับองค์กรของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและพัฒนาซอฟต์แวร์แบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p