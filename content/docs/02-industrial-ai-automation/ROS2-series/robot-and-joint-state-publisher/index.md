---
title: "ตอนที่ 19: Robot State Publisher และ Joint State Publisher ผู้ปลุกเสกโมเดลให้มีชีวิต"
weight: 1219
date: 2026-03-16
author: วิสิทธิ์ แผ้วกระโทก
tags: ["ROS 2", "Robotics", "URDF", "TF", "robot_state_publisher", "joint_state_publisher"]
---
{{< figure src="robot-and-joint-state-publisher-cover.jpg" alt="รูปปกบทความ" >}}

## 1. 🎯 ตอนที่ 19: Robot State Publisher และ Joint State Publisher ผู้ปลุกเสกโมเดลให้มีชีวิต

สวัสดีครับน้องๆ ว่าที่วิศวกรหุ่นยนต์ทุกคน! ในตอนที่ผ่านๆ มา เราได้เรียนรู้วิธีการเขียน "พิมพ์เขียว" ของหุ่นยนต์ด้วย URDF และ Xacro กันไปแล้ว แต่พิมพ์เขียวก็ยังเป็นแค่ไฟล์ข้อความ XML นิ่งๆ ใช่ไหมครับ? 

ถ้าน้องๆ เอา URDF ไปโหลดในระบบ ROS 2 ดื้อๆ หุ่นยนต์มันจะไม่รู้เลยว่าตอนนี้ล้อหมุนไปกี่องศาแล้ว หรือแขนกลยกขึ้นไปสูงแค่ไหน เพราะมันยังขาด "สมองส่วนที่ทำหน้าที่รับรู้โครงสร้างร่างกาย" วันนี้พี่จะพาไปรู้จักกับ Node พระเอก 2 ตัวที่จะมาประกอบร่างข้อมูล URDF เข้ากับองศาของมอเตอร์ แล้วแปลงออกมาเป็นระบบพิกัด TF Tree ให้ระบบทั้งระบบรับรู้ได้อย่างสมบูรณ์ครับ!

## 2. 📖 เปิดฉาก (The Hook)

น้องๆ ลองนึกภาพหุ่นยนต์เสิร์ฟอาหารที่มีแขนกลยืดหดได้ดูนะครับ 
สมมติว่าเซ็นเซอร์ LIDAR ของหุ่นยนต์มองเห็นกำแพงอยู่ห่างออกไป 1 เมตร การที่ระบบนำทางจะคำนวณหลบหลีกได้ มันต้องรู้ว่า LIDAR ติดอยู่ตรงไหนของตัวรถ และในขณะเดียวกัน ถ้าแขนกลกำลังยื่นออกไปหยิบแก้วน้ำ ระบบก็ต้องรู้ว่าปลายแขนกล (End-effector) ขยับออกห่างจากฐานรถไปกี่เซนติเมตรแล้ว 

การจะมานั่งเขียนโค้ดตรีโกณมิติคำนวณ Forward Kinematics เองทุกครั้งที่มอเตอร์ขยับ คงเป็นฝันร้ายของโปรแกรมเมอร์แน่ๆ! วิศวกร ROS จึงได้สร้างเครื่องมือสำเร็จรูปที่ชื่อว่า **`robot_state_publisher`** และ **`joint_state_publisher`** ขึ้นมา เพื่อทำหน้าที่รับข้อมูล "มุมมอเตอร์" มาคำนวณร่วมกับ "พิมพ์เขียว URDF" แล้วกระจายบอกทุกคนในระบบผ่านระบบ TF แบบอัตโนมัติ! นี่แหละครับคือเวทมนตร์ของการลดภาระงานในสไตล์ ROS!

## 3. 🧠 แก่นวิชา (Core Concepts)

เพื่อให้เห็นภาพการทำงานร่วมกัน พี่ขอแยกอธิบายหน้าที่ของ Node แต่ละตัวเปรียบเทียบให้ฟังง่ายๆ แบบนี้ครับ:

*   **`joint_state_publisher` (เส้นประสาทรับความรู้สึกจากกล้ามเนื้อ):**
    Node ตัวนี้มีหน้าที่อ่านไฟล์ URDF เพื่อค้นหาว่ามี "ข้อต่อ (Joints)" ตัวไหนบ้างที่ขยับได้ (Non-fixed joints) จากนั้นมันจะทำการส่งข้อมูลสถานะของข้อต่อเหล่านั้น (เช่น ตำแหน่งองศา, ความเร็ว, และแรงบิด) ออกมาในรูปแบบข้อความ `sensor_msgs/JointState` บน Topic ที่ชื่อว่า **`/joint_states`** 
    *(Tips: สำหรับช่วงทดสอบ เรามักจะใช้ `joint_state_publisher_gui` ซึ่งจะมีหน้าต่าง Pop-up พร้อมสไลเดอร์ (Sliders) โผล่ขึ้นมาให้เราเอามือเลื่อนค่าองศาจำลองไปมาได้เลย)*

*   **`robot_state_publisher` (สมองส่วนคำนวณตำแหน่งร่างกาย):**
    นี่คือ Node ตัวท็อปของระบบ! มันต้องการอินพุต 2 อย่างคือ:
    1.  **URDF (พิมพ์เขียว):** รับเข้ามาผ่านพารามิเตอร์ `robot_description`
    2.  **`/joint_states` (มุมมอเตอร์):** รับเข้ามาจากการ Subscribe Topic
    
    เมื่อได้ข้อมูลครบ มันจะใช้ไลบรารี Kinematic Dynamics Library (KDL) ภายในตัวมัน คำนวณสมการ **Forward Kinematics** ทันที เพื่อหาว่าจากมุมมอเตอร์ปัจจุบัน ชิ้นส่วนแต่ละชิ้น (Link) จะมี 3D Poses (ตำแหน่งและทิศทางแบบ Quaternion) อยู่ตรงไหนในสเปซ 3 มิติ จากนั้นมันจะพ่นผลลัพธ์ออกไปที่ Topic **`/tf`** (สำหรับชิ้นส่วนที่เคลื่อนไหว) และ **`/tf_static`** (สำหรับชิ้นส่วนที่ติดตายตัว)

สรุปผังการไหลของข้อมูล (Data Flow) ก็คือ: 
**Hardware (หรือ GUI)** -> พ่นลง `/joint_states` -> **`robot_state_publisher`** (อ่าน URDF) -> พ่นลง `/tf` และ `/tf_static` ให้ RViz หรือ Navigation เอาไปใช้ต่อครับ!

{{< figure src="robot-and-joint-state-publisher-diagram.jpg" alt="รูปประกอบ Architecture ของ Robot State Publisher" >}}

## 4. 💻 ร่ายมนต์โค้ดและคำสั่ง (Show me the Code)

เรามาดูวิธีร่ายมนต์ปลุก 2 Nodes นี้ผ่าน Terminal และ Launch File กันครับ!

**1. การรันผ่าน Terminal (เพื่อทดสอบด่วน):**
เปิด Terminal หน้าต่างแรก เพื่อรัน `robot_state_publisher` พร้อมโยนไฟล์ Xacro เข้าไป:
```bash
ros2 run robot_state_publisher robot_state_publisher --ros-args -p robot_description:="$(xacro /home/<user>/my_robot.urdf.xacro)"

# ถ้ารันสำเร็จ จะมี Log โชว์ขึ้นมาว่า:
# [robot_state_publisher]: got segment base_link
# [robot_state_publisher]: got segment right_wheel
```

เปิด Terminal หน้าต่างที่สอง เพื่อรัน `joint_state_publisher_gui` เสกสไลเดอร์ขึ้นมาควบคุม:
```bash
ros2 run joint_state_publisher_gui joint_state_publisher_gui
```

**2. การประกอบร่างลง Python Launch File แบบมือโปร:**
ในการใช้งานจริง เราจะจับทุกอย่างมัดรวมกันใน Launch File (`display.launch.py`) เพื่อให้เปิดระบบได้ในคำสั่งเดียวครับ:
```python
import os
from launch import LaunchDescription
from launch_ros.actions import Node
from launch_ros.parameter_descriptions import ParameterValue
from launch.substitutions import Command
from ament_index_python.packages import get_package_share_path

def generate_launch_description():
    # 1. หาพาทของไฟล์ URDF/Xacro ของเรา
    urdf_path = os.path.join(get_package_share_path('my_robot_description'), 'urdf', 'my_robot.urdf.xacro')
    
    # 2. แปลงไฟล์ Xacro เป็น String เก็บไว้ในตัวแปร Parameter
    robot_description = ParameterValue(Command(['xacro ', urdf_path]), value_type=str)

    # 3. กำหนด Node robot_state_publisher และยัดพารามิเตอร์เข้าไป
    robot_state_publisher_node = Node(
        package="robot_state_publisher",
        executable="robot_state_publisher",
        parameters=[{'robot_description': robot_description}]
    )
    
    # 4. กำหนด Node joint_state_publisher_gui (เพื่อใช้สไลเดอร์เทสโมเดล)
    joint_state_publisher_gui_node = Node(
        package="joint_state_publisher_gui",
        executable="joint_state_publisher_gui"
    )

    # โยน Nodes เข้าไปใน LaunchDescription
    return LaunchDescription([
        robot_state_publisher_node,
        joint_state_publisher_gui_node
    ])
```

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips)

ในฐานะที่พี่ดีบักหุ่นยนต์มาเยอะ นี่คือจุดที่วิศวกรมือใหม่มักจะหัวหมุนที่สุดครับ:

*   **ลาก่อน GUI เมื่อใช้หุ่นยนต์จริง:** 
    จำไว้นะครับว่า Node `joint_state_publisher_gui` มีไว้สำหรับ **จำลองและทดสอบ (Testing) เท่านั้น!** เมื่อน้องๆ นำโค้ดไปรันบนหุ่นยนต์จริง หรือรันใน Simulator อย่าง Gazebo น้องๆ **ห้าม** สตาร์ท Node นี้เด็ดขาด เพราะฮาร์ดแวร์ไดรเวอร์ (เช่น `ros2_control` หรือตัวอ่าน Encoder ของล้อ) จะเป็นคนรับหน้าที่พ่นข้อมูลลง Topic `/joint_states` แทน ถ้าน้องเปิด GUI ค้างไว้ ข้อมูลสไลเดอร์ของน้องจะไปตีกับข้อมูลฮาร์ดแวร์จริง จนล้อหุ่นยนต์ใน RViz หมุนกระตุกเป็นเจ้าเข้าเลยครับ!
*   **RViz ขาวโพลน หรือ Error แดงเถือก (Missing TFs):**
    ถ้าน้องรัน `robot_state_publisher` แล้ว แต่หุ่นยนต์ใน RViz แขนขาด ล้อหาย หรือมี Error ฟ้องว่า "No transform from [wheel_link] to [base_link]" สาเหตุยอดฮิต 99% คือ **ไม่มีใครกำลังพ่นข้อมูลลง Topic `/joint_states` ครับ!** เพราะถ้า `robot_state_publisher` ไม่ได้รับมุมมอเตอร์ มันก็จะไม่ยอมคำนวณ TF ของชิ้นส่วนที่ขยับได้ออกมาเลย วิธีเช็กง่ายๆ คือรัน `ros2 topic echo /joint_states` ดูครับว่ามีข้อมูลไหลมาไหม
*   **ความแตกต่างของ `/tf` และ `/tf_static`:**
    เพื่อประหยัดทรัพยากรคอมพิวเตอร์ `robot_state_publisher` จะแยกการส่งข้อมูล TF ออกเป็น 2 Topics ชิ้นส่วนที่ขยับไม่ได้ (Fixed joints) อย่างเซ็นเซอร์กล้อง จะถูกส่งผ่าน `/tf_static` แค่ครั้งเดียวตอนเริ่มระบบ ส่วนชิ้นส่วนที่ขยับได้ (Revolute/Continuous) จะถูกพ่นผ่าน `/tf` ด้วยความถี่สูงครับ

## 6. 🏁 บทสรุป (To be continued...)

การทำงานร่วมกันระหว่าง **`robot_state_publisher`** และ **`joint_state_publisher`** คือหัวใจหลักที่เปลี่ยนโมเดลกระดาษ (URDF) ให้กลายเป็นหุ่นยนต์ที่มีความรับรู้ใน 3D Space (TF Tree) มันเป็นกระบวนการมาตรฐานที่ระบบ ROS 2 ทุกระบบต้องใช้ ไม่ว่าจะเป็นหุ่นยนต์ดูดฝุ่นตัวเล็กๆ ไปจนถึงแขนกลอุตสาหกรรมในโรงงาน!

ตอนนี้เรามีโมเดล 3 มิติ มีระบบพิกัด TF และรู้วิธีขยับข้อต่อหุ่นยนต์ด้วยสไลเดอร์กันแล้ว! แต่เพื่อความสมจริง หุ่นยนต์ของเราควรจะต้องมีแรงโน้มถ่วง เดินชนกำแพงได้ และใช้เซ็นเซอร์อ่านค่าได้จริงๆ ในตอนต่อไป พี่จะพาน้องๆ โยนหุ่นยนต์ของเราเข้าสู่โลก **Gazebo Simulator** แบบเต็มตัว เตรียมบอกลา GUI Slider แล้วมาเขียนไดรเวอร์คุมมอเตอร์จำลองกันครับ!

---
**ต้องการที่ปรึกษาด้านการออกแบบสถาปัตยกรรมหุ่นยนต์ (Robotics) และระบบ Automation ให้กับองค์กรของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและพัฒนาซอฟต์แวร์แบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p