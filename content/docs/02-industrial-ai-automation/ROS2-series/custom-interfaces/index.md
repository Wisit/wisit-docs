---
title: "ตอนที่ 14: ประกอบร่างข้อมูลหุ่นยนต์! การสร้าง Custom Messages และ Services ของตัวเอง"
weight: 1214
date: 2026-03-16
author: วิสิทธิ์ แผ้วกระโทก
tags: ["ROS 2", "Robotics", "Custom Messages", "Custom Services", "CMakeLists", "package.xml"]
---
{{< figure src="custom-interfaces-cover.jpg" alt="รูปปกบทความ" >}}

## 1. 🎯 ตอนที่ 14: ประกอบร่างข้อมูลหุ่นยนต์! การสร้าง Custom Messages และ Services ของตัวเอง

สวัสดีครับน้องๆ วิศวกรหุ่นยนต์ทุกคน! ผ่านมาหลายตอนแล้ว เราได้รู้จักการใช้ Topic และ Service เพื่อให้มดงาน (Node) ของเราสื่อสารกันได้ แต่ที่ผ่านมาเราแอบยืม "กล่องพัสดุมาตรฐาน" (Standard Messages) อย่าง `std_msgs/String` หรือ `geometry_msgs/Twist` มาใช้ตลอดเลยใช่ไหมครับ?

แล้วถ้าวันหนึ่ง น้องๆ สร้างหุ่นยนต์ที่มีเซ็นเซอร์แปลกๆ ล่ะ? เช่น อยากส่งข้อมูล "อุณหภูมิมอเตอร์, เปอร์เซ็นต์แบตเตอรี่, และรหัส Error" ไปพร้อมๆ กันในการประกาศ Topic ครั้งเดียว... ถ้าน้องต้องแยกส่ง 3 Topics คงวุ่นวายตายเลย! วันนี้พี่จะพามาแก้ปัญหานี้ด้วยการสร้าง **"Custom Interfaces (.msg และ .srv)"** หรือกล่องพัสดุสั่งตัดพิเศษฉบับของเราเองกันครับ!

## 2. 📖 เปิดฉาก (The Hook)

สมมติว่าน้องกำลังทำโปรเจกต์หุ่นยนต์โกดังสินค้า แล้วอยากสร้าง Service สั่งให้หุ่นยนต์ "เปิดไฟสัญญาณเตือนพร้อมเสียง" น้องลองไปค้นหาใน `std_srvs` หรือ `example_interfaces` ก็พบว่า... มันไม่มี Service ตัวไหนเลยที่รับค่า Boolean (เปิด/ปิด) และรับค่า Integer (รหัสเสียง) ได้ในกล่องเดียวกัน!

ปัญหาคลาสสิกของคนทำหุ่นยนต์คือ การพยายามเอาข้อมูลที่ซับซ้อนไปยัดลงใน Message พื้นฐานที่ไม่ตอบโจทย์ (เช่น แอบเอาข้อมูลใส่ลงใน String แล้วค่อยไปเขียนโค้ดแยก String ทีหลัง) ซึ่งเป็นวิธีการที่ **"ผิดมหันต์และไม่เสถียร"** ครับ! 

ในโลกของ ROS 2 ถ้าเราหา Interface ที่ตรงสเปกเป๊ะๆ ไม่เจอ กฎเหล็กคือ **"จงสร้างมันขึ้นมาใหม่ (Custom Interfaces)!"** แล้วจับมันแพ็กใส่ Package แยกไว้ เพื่อให้ Node ทุกตัวในระบบเรียกใช้งานได้อย่างเป็นระเบียบครับ

## 3. 🧠 แก่นวิชา (Core Concepts)

ก่อนที่เราจะไปเขียนโค้ด เราต้องเข้าใจหลักการสำคัญ 3 ข้อของการสร้าง Custom Interfaces ใน ROS 2 ก่อนครับ:

*   **กฎข้อที่ 1: แยก Interface Package ออกมาต่างหาก (Best Practice):** 
    เราจะไม่สร้างไฟล์ `.msg` หรือ `.srv` ปะปนไว้ใน Package ที่มีซอร์สโค้ด Node (C++/Python) ครับ แต่เราจะสร้าง Package ใหม่ขึ้นมา (เช่น `my_robot_interfaces`) เพื่อเก็บไฟล์ Interface เหล่านี้โดยเฉพาะ! วิธีนี้จะช่วยป้องกันปัญหา Dependency พันกันยุ่งเหยิง (Dependency Mess) ในอนาคต
*   **ไฟล์ `.msg` (สำหรับ Topic):** 
    เป็นเพียงไฟล์ Text ธรรมดาที่ประกาศชนิดของตัวแปรและชื่อตัวแปรเรียงกันลงมา (เช่น `float64 temperature`) เมื่อคอมไพล์เสร็จ ROS 2 จะแปลงไฟล์นี้เป็น Class ให้เราเรียกใช้ใน C++ หรือ Python อัตโนมัติ
*   **ไฟล์ `.srv` (สำหรับ Service):** 
    โครงสร้างจะคล้ายกับ `.msg` แต่ความพิเศษคือมันจะถูกแบ่งออกเป็น 2 ส่วนด้วยเครื่องหมายขีดกลางสามเส้น `---` โดยด้านบนคือฝั่ง **Request (คำขอ)** และด้านล่างคือฝั่ง **Response (คำตอบ)** ครับ

{{< figure src="custom-interfaces-diagram.jpg" alt="รูปประกอบโครงสร้างไฟล์ Interface Package" >}}

## 4. 💻 ร่ายมนต์โค้ดและคำสั่ง (Show me the Code)

มาลงมือทำกันเลยครับ! เราจะสร้าง Package ชื่อ `my_robot_interfaces` ขึ้นมาเพื่อเก็บ Custom Message และ Service ของเรา

**1. สร้าง Package สำหรับ Interfaces โดยเฉพาะ**
เปิด Terminal ขึ้นมาแล้วรันคำสั่งต่อไปนี้:
```bash
cd ~/ros2_ws/src/
# สร้าง Package เปล่าๆ แบบ ament_cmake (Interface ต้องเป็น ament_cmake เสมอ!)
ros2 pkg create my_robot_interfaces

# เข้าไปลบโฟลเดอร์ที่ไม่จำเป็นทิ้ง แล้วสร้างโฟลเดอร์ msg และ srv ขึ้นมาแทน
cd my_robot_interfaces
rm -r src/ include/
mkdir msg srv
```

**2. สร้าง Custom Message (`.msg`)**
สมมติว่าเราอยากส่งสถานะฮาร์ดแวร์หุ่นยนต์ ให้สร้างไฟล์ชื่อ `HardwareStatus.msg` ในโฟลเดอร์ `msg/`:
*(ข้อควรระวัง: ชื่อไฟล์ต้องเป็น PascalCase เช่น HardwareStatus เสมอ)*
```text
# ไฟล์: my_robot_interfaces/msg/HardwareStatus.msg
int64 version
float64 temperature
bool are_motors_ready
string debug_message
```

**3. สร้าง Custom Service (`.srv`)**
สมมติว่าเราอยากได้ Service สำหรับสั่งรีเซ็ตค่าต่างๆ ให้สร้างไฟล์ชื่อ `ResetCounter.srv` ในโฟลเดอร์ `srv/`:
```text
# ไฟล์: my_robot_interfaces/srv/ResetCounter.srv
int64 reset_value
---
bool success
string message
```

**4. ปลดล็อกพลังด้วย `package.xml`**
เปิดไฟล์ `package.xml` ของ `my_robot_interfaces` แล้วเพิ่มแท็กศักดิ์สิทธิ์ 3 บรรทัดนี้ต่อจาก `<buildtool_depend>ament_cmake</buildtool_depend>`:
```xml
  <build_depend>rosidl_default_generators</build_depend>
  <exec_depend>rosidl_default_runtime</exec_depend>
  <member_of_group>rosidl_interface_packages</member_of_group>
```

**5. สอนระบบให้คอมไพล์ใน `CMakeLists.txt`**
เปิดไฟล์ `CMakeLists.txt` แล้วเพิ่มโค้ดด้านล่างนี้ คั่นระหว่าง `find_package(ament_cmake REQUIRED)` กับ `ament_package()`:
```cmake
# ตามหาเครื่องมือสร้าง Interface
find_package(rosidl_default_generators REQUIRED)

# ระบุรายชื่อไฟล์ .msg และ .srv ของเรา
rosidl_generate_interfaces(${PROJECT_NAME}
  "msg/HardwareStatus.msg"
  "srv/ResetCounter.srv"
)

# ส่งออก Dependency ให้ Package อื่นใช้ได้
ament_export_dependencies(rosidl_default_runtime)
```

**6. ประกอบร่าง! (Build & Source)**
กลับมาที่ Workspace หลัก แล้วสั่งบิลด์เฉพาะ Package นี้:
```bash
cd ~/ros2_ws/
colcon build --packages-select my_robot_interfaces
source install/setup.bash

# ตรวจสอบความสำเร็จ! ถ้าขึ้นรายละเอียดตัวแปรแสดงว่าคอมไพล์ผ่านแล้ว
ros2 interface show my_robot_interfaces/msg/HardwareStatus
```

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips)

*   **วิชาตกม้าตาย: ลืมเครื่องหมาย `---` ในไฟล์ `.srv`:**
    พี่เจอน้องๆ บ่นว่าบิลด์ `.srv` ไม่ผ่านบ่อยมาก! สาเหตุหลักคือลืมใส่ `---` ครับ กฎเหล็กคือ **ต่อให้ Request หรือ Response ของน้องจะว่างเปล่า (ไม่มีตัวแปรเลยสักตัว) น้องก็ต้องใส่เครื่องหมาย `---` คั่นไว้เสมอ** ครับ! ไม่งั้นคอมไพเลอร์ของ ROS 2 จะงงว่านี่มันคือ `.msg` หรือ `.srv` กันแน่
*   **อย่าใช้ `std_msgs` ซ้อนเข้าไปใน Interface ถ้าไม่จำเป็น:**
    เวลาประกาศตัวแปรในไฟล์ `.msg` ให้ใช้ Primitive types (เช่น `int64`, `float32`, `string`) ตรงๆ ไปเลยครับ หลีกเลี่ยงการใช้ `std_msgs/Int64` มาซ้อนเป็นฟิลด์ย่อย เพราะมันจะทำให้โครงสร้างข้อมูลบวมโดยใช่เหตุ และต้องมานั่งอ้างอิงฟิลด์ `.data` ซ้อนไปซ้อนมาเวลาเขียนโค้ดครับ
*   **การนำไปใช้ใน Package อื่น:**
    หลังจากบิลด์เสร็จ ถ้าน้องจะเขียน Node (C++ หรือ Python) ใน Package อื่นเพื่อเรียกใช้กล่องพัสดุนี้ น้องต้องไปเพิ่ม `<depend>my_robot_interfaces</depend>` ใน `package.xml` ของ Package ปลายทางด้วยนะครับ! จากนั้นใน Python ก็แค่ร่ายมนต์ `from my_robot_interfaces.msg import HardwareStatus` ก็ใช้งานได้เลย!

## 6. 🏁 บทสรุป (To be continued...)

เห็นไหมครับว่าการสร้าง Custom Message และ Service ใน ROS 2 นั้นไม่ได้น่ากลัวอย่างที่คิด ขอแค่เรารู้จักการจัดการ "Interface Package" ให้เป็นระเบียบ และการตั้งค่าไฟล์ `CMakeLists.txt` กับ `package.xml` ให้ถูกต้อง เราก็สามารถออกแบบโครงสร้างข้อมูลให้เข้ากับหุ่นยนต์ของเราได้ 100% แล้วครับ!

ในตอนต่อไป พี่จะพาน้องๆ มารู้จักกับกระดูกสันหลังของหุ่นยนต์ นั่นก็คือ **TF (Transform Framework)** เราจะมาดูกันว่า หุ่นยนต์มันรู้ได้อย่างไรว่า "กล้อง" ติดอยู่ห่างจาก "ล้อ" กี่เซนติเมตร และมันช่วยในการคำนวณระยะทางได้อย่างไร เตรียมตัวรับมือกับคณิตศาสตร์และเวกเตอร์กันให้พร้อมนะครับ!

---
**ต้องการที่ปรึกษาด้านการออกแบบสถาปัตยกรรมหุ่นยนต์ (Robotics) และระบบ Automation ให้กับองค์กรของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและพัฒนาซอฟต์แวร์แบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p