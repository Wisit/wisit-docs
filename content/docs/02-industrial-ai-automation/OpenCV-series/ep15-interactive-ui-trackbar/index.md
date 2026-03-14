---
title: "ตอนที่ 15: เสก UI ให้มีชีวิต! ปรับค่า Real-time ด้วย Trackbar"
weight: 715
date: 2026-03-14
author: วิสิทธิ์ แผ้วกระโทก
tags: ["OpenCV", "Computer Vision", "C++", "HighGUI", "Trackbar", "GUI"]
---
{{< figure src="ep15-interactive-ui-trackbar-cover.jpg" alt="รูปปกบทความ" >}}

## 1. 🎯 ตอนที่ 15: เสก UI ให้มีชีวิต! ปรับค่า Real-time ด้วย Trackbar

## 2. 📖 เปิดฉาก (The Hook)
น้องๆ เคยเจอปัญหาแบบนี้ไหมครับ? เขียนโค้ดทำ Thresholding เพื่อแยกวัตถุออกจากพื้นหลัง สมมติว่าตั้งค่า Threshold ไว้ที่ 100 พอกดรันปุ๊บ... อ้าว มืดไป! กลับมาแก้โค้ดเป็น 120 กดคอมไพล์ใหม่ รันใหม่... อ้าว คราวนี้สว่างไป! กว่าจะหาค่าที่พอดีกับแสงหน้างานโรงงานได้ ต้องสลับหน้าจอแก้โค้ดแล้วคอมไพล์ใหม่เป็นสิบๆ รอบ เสียเวลาจิบกาแฟหมดเลย!

ในโลกของ Computer Vision แสงและสภาพแวดล้อมมีการเปลี่ยนแปลงตลอดเวลา การฮาร์ดโค้ด (Hardcode) ตัวเลขลงไปตรงๆ จึงไม่ใช่ทางออกที่ดีครับ วันนี้พี่จะพาน้องๆ ไปรู้จักกับอาวุธลับในโมดูล **HighGUI** ที่ชื่อว่า **Trackbar** (แถบเลื่อน) ซึ่งจะเปลี่ยนหน้าต่างแสดงผลทื่อๆ ของเรา ให้กลายเป็นแผงควบคุมสไตล์ DJ ที่สามารถปรับค่าพารามิเตอร์ต่างๆ (เช่น ความสว่าง, ค่า Threshold) ได้แบบ Real-time โดยไม่ต้องปิดโปรแกรมเลยครับ!

## 3. 🧠 แก่นวิชา (Core Concepts)
ใน OpenCV โมดูล HighGUI ไม่ได้มีแค่ฟังก์ชัน `imshow` สำหรับโชว์ภาพเท่านั้น แต่มันยังมีเครื่องมือสร้าง UI พื้นฐานให้เราด้วย เหตุผลที่ OpenCV เรียกแถบเลื่อนนี้ว่า **Trackbar** ก็เพราะในยุคแรกๆ มันถูกสร้างมาเพื่อใช้ "เลื่อนดูเฟรมวิดีโอ" (Tracking video frames) นั่นเองครับ แต่ปัจจุบันเราเอามันมาประยุกต์ใช้ปรับค่าอะไรก็ได้ 

เพื่อให้หน้าต่างของเรามี Trackbar เราต้องรู้จักฟังก์ชันและกลไกเหล่านี้ครับ:

*   **1. การสร้างแถบเลื่อน (`createTrackbar`):**
    ฟังก์ชันนี้ใช้สำหรับสร้างแถบเลื่อนไปแปะไว้บนหน้าต่างที่เราสร้างขึ้น โดยมีพารามิเตอร์สำคัญคือ:
    *   `trackbarname`: ชื่อของแถบเลื่อน (Label ที่จะโชว์บนหน้าจอ)
    *   `winname`: ชื่อหน้าต่าง (Window) ที่เราต้องการเอาแถบเลื่อนไปแปะ
    *   `value`: พอยน์เตอร์ (Pointer) ชี้ไปยังตัวแปร Integer ที่จะเก็บค่าของแถบเลื่อน
    *   `count`: ค่าสูงสุดที่เลื่อนได้ (Max value) *หมายเหตุ: ค่าต่ำสุดของ Trackbar ใน OpenCV จะเริ่มต้นที่ 0 เสมอ*
    *   `onChange`: ฟังก์ชัน Callback ที่จะถูกเรียกอัตโนมัติเมื่อมีการขยับแถบเลื่อน 
    *   `userdata`: พอยน์เตอร์สำหรับส่งข้อมูล (เช่น Image Matrix) เข้าไปใน Callback แบบเนียนๆ โดยไม่ต้องใช้ Global Variable
*   **2. กลไก Callback (Event-Driven):**
    เหมือนกับตอนที่เราดักจับเมาส์เลยครับ เราจะฝากฟังก์ชัน (Callback) ไว้กับ OpenCV เมื่อผู้ใช้เอานิ้วไปลากแถบเลื่อน OpenCV จะวิ่งมาเรียกฟังก์ชันนี้ทันที พร้อมส่ง "ค่าตัวเลข ณ ตำแหน่งนั้น" มาให้เราเอาไปคำนวณต่อ
*   **3. ทริคการทำปุ่มสวิตช์ (The Switch Trick):**
    รู้ไหมครับว่า HighGUI แบบดั้งเดิมไม่มีปุ่ม (Button) ให้ใช้! วิศวกรสาย Vision รุ่นเก๋าจึงมักใช้ Trackbar ที่มีค่าสูงสุดเป็น 1 (คือเลื่อนได้แค่ 0 กับ 1) มาทำหน้าที่แทนสวิตช์ เปิด-ปิด (On/Off Switch) ครับ ถือเป็นทริคที่คลาสสิกมาก

{{< figure src="ep15-interactive-ui-trackbar-diagram.jpg" alt="รูปประกอบ" >}}

## 4. 💻 ร่ายมนต์คำสั่ง (Show me the Code)
เรามาดูโค้ด C++ ในการสร้างแอปพลิเคชันปรับค่า Threshold แบบ Real-time กันครับ โค้ดนี้เมื่อเราเลื่อน Trackbar ภาพขาวดำจะเปลี่ยนไปตามค่าที่เราเลื่อนทันที!

```cpp
#include <opencv2/opencv.hpp>
#include <iostream>

using namespace cv;
using namespace std;

// 1. สร้างโครงสร้างสำหรับส่งข้อมูลเข้าไปใน Callback (หลีกเลี่ยง Global Variable)
struct TrackbarData {
    Mat imageGray;
    String windowName;
};

// 2. สร้าง Callback Function ที่จะถูกเรียกเมื่อแถบเลื่อนขยับ
void onThresholdSlide(int pos, void* userdata) {
    // แปลง void* กลับเป็นโครงสร้างข้อมูลของเรา
    TrackbarData* data = (TrackbarData*)userdata;
    
    Mat result;
    // นำค่า pos (ตำแหน่งของ Trackbar) มาใช้เป็นค่า Threshold
    // พิกเซลไหนสว่างกว่า pos จะกลายเป็นสีขาว (255) ต่ำกว่าจะกลายเป็นสีดำ (0)
    threshold(data->imageGray, result, pos, 255, THRESH_BINARY);
    
    // อัปเดตภาพบนหน้าต่าง
    imshow(data->windowName, result);
}

int main() {
    // โหลดภาพและแปลงเป็น Grayscale รอไว้เลย
    Mat image = imread("factory_part.jpg", IMREAD_COLOR);
    if(image.empty()) return -1;
    
    Mat gray;
    cvtColor(image, gray, COLOR_BGR2GRAY);

    String winName = "Real-time Thresholding";
    namedWindow(winName, WINDOW_AUTOSIZE);

    // 3. เตรียมข้อมูลที่จะส่งให้ Callback
    TrackbarData myData;
    myData.imageGray = gray;
    myData.windowName = winName;

    // 4. สร้างตัวแปรเก็บค่าเริ่มต้นของ Trackbar
    int thresholdValue = 100; // ค่าเริ่มต้น
    int maxThreshold = 255;   // ค่าสูงสุด (เพราะพิกเซลเก็บค่าได้ถึง 255)

    // 5. สร้าง Trackbar แปะลงบนหน้าต่าง
    createTrackbar("Thresh Val", winName, &thresholdValue, maxThreshold, onThresholdSlide, &myData);

    // 6. เรียก Callback ด้วยตัวเอง 1 ครั้ง เพื่อวาดภาพตั้งต้นก่อนผู้ใช้จะขยับ Trackbar
    onThresholdSlide(thresholdValue, &myData);

    // วนลูปโชว์ UI ไปเรื่อยๆ จนกว่าจะกด ESC
    while(true) {
        if(waitKey(30) == 27) break; 
    }

    destroyAllWindows();
    return 0;
}
```

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips)
*   **ค่าติดลบทำยังไง? (The Negative Value Workaround):** อย่างที่บอกไปครับว่า Trackbar ใน OpenCV บังคับให้ค่าต่ำสุดเริ่มต้นที่ 0 เสมอ สมมติว่าน้องๆ อยากทำ Trackbar ปรับเพิ่ม/ลด ความสว่างภาพ (ช่วง -100 ถึง +100) วิธีแก้คือ ให้ตั้งค่า Max เป็น 200 (ช่วง 0-200) แล้วพอรับค่า `pos` เข้ามาใน Callback ให้เอาไปลบออกด้วย 100 เสมอ (`real_value = pos - 100;`) แค่นี้เราก็จะได้ค่าติดลบมาใช้งานแล้วครับ!
*   **ไม่ง้อ Callback ก็ได้นะ (`getTrackbarPos`):** ถ้าน้องๆ เขียนโค้ดรันลูปวิดีโอจากกล้อง (VideoCapture) อยู่แล้ว บางทีการใช้ Callback อาจจะทำให้โครงสร้างโค้ดซับซ้อน เราสามารถไม่ต้องใส่ฟังก์ชัน Callback ก็ได้ (ใส่ `NULL`) แล้วใช้คำสั่ง `int val = getTrackbarPos("TrackbarName", "WindowName");` ดึงค่าตัวเลขจากแถบเลื่อนมาใช้ดื้อๆ ในลูป `while(true)` ของวิดีโอได้เลยครับ สะดวกและอ่านโค้ดง่ายกว่าเวลาทำกับกล้อง Real-time
*   **ระวังคอขวดใน Callback:** ฟังก์ชัน Callback ควรทำงานให้เร็วที่สุด (ระดับมิลลิวินาที) ถ้าน้องๆ เอาโมเดล Deep Learning หนักๆ ไปรันทุกครั้งที่ Trackbar ขยับ 1 สเกล หน้าต่าง UI ของเราจะค้าง (Freeze) และกระตุกเป็นเจ้าเข้าเลยครับ! ควรใช้ Trackbar กับ Image Processing พื้นฐานที่เร็วๆ เท่านั้น

## 6. 🏁 บทสรุป (To be continued...)
ยอดเยี่ยมมากครับ! ตอนนี้วิชา HighGUI ของเราแข็งแกร่งครบถ้วนแล้ว ไม่ว่าจะเป็นการโชว์ภาพ, จับการคลิกเมาส์, และการสร้างแถบควบคุมแบบ Trackbar ทำให้เราสามารถพัฒนาเครื่องมือทดสอบ AI (Prototyping Tool) ไปใช้หน้างานจริงได้อย่างมืออาชีพ และจูนค่าต่างๆ ได้โดยไม่ต้องปวดหัวกับการคอมไพล์โค้ดใหม่

เมื่อ UI พร้อม เครื่องมือพร้อม ในตอนต่อไป พี่จะพาน้องๆ ดำดิ่งสู่โลกของการ "กรองภาพ" (Image Filtering) อย่างจริงจัง เราจะมาดูกันว่าการใช้เวทมนตร์อย่าง Convolution เพื่อเบลอภาพ ลดสัญญาณรบกวน (Noise) บนสายพานการผลิตนั้นทำงานอย่างไร เตรียมใจรับความสนุกทางคณิตศาสตร์ได้เลยครับ!

---
**ต้องการที่ปรึกษาด้านการพัฒนาระบบ AI Camera หรือ Machine Vision ให้กับโรงงานของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและติดตั้งระบบแบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p