---
title: "ตอนที่ 21: ROS Controllers (ros_control) ระบบสมองกลสั่งการมอเตอร์"
weight: 1221
date: 2026-03-16
author: วิสิทธิ์ แผ้วกระโทก
tags: ["ROS 2", "Robotics", "ros2_control", "Controller Manager", "Hardware Interface", "PID"]
---
{{< figure src="ros2-control-system-cover.jpg" alt="รูปปกบทความ" >}}

## 1. 🎯 ตอนที่ 21: ROS Controllers (ros_control) ระบบสมองกลสั่งการมอเตอร์

สวัสดีครับน้องๆ วิศวกรหุ่นยนต์ทุกคน! ในตอนที่ผ่านๆ มา เราได้เรียนรู้วิธีการคำนวณสมการ Kinematics เพื่อแปลงคำสั่งความเร็ว (Twist) ให้เป็นความเร็วของล้อซ้าย-ขวากันไปแล้ว แต่ในโลกของความเป็นจริง การสั่งให้มอเตอร์หมุนด้วยความเร็ว 5 rad/s มันไม่ได้ทำได้ด้วยการสั่งแค่ครั้งเดียวแล้วจบนะครับ!

ลองนึกภาพหุ่นยนต์ของเราวิ่งขึ้นเนินดูสิครับ น้ำหนักและแรงโน้มถ่วงจะดึงให้มอเตอร์หมุนช้าลง ถ้าระบบของเราไม่รู้จัก "การปรับตัว" หุ่นยนต์ก็คงไหลถอยหลังตกเนินไปแล้ว! วันนี้พี่จะพาไปเจาะลึกสถาปัตยกรรมระดับปรมาจารย์ที่ชื่อว่า **`ros2_control`** (พัฒนาต่อยอดมาจาก `ros_control` ในยุค ROS 1) ซึ่งเปรียบเสมือน "ระบบประสาทอัตโนมัติ" ที่คอยคุมวงจร PID ของหุ่นยนต์ให้ทำงานได้อย่างแม่นยำและเสถียรที่สุดครับ!

## 2. 📖 เปิดฉาก (The Hook)

หน้าที่ของ Controller คือสิ่งที่อยู่ตรงกลางระหว่าง Sensor และ Actuator มันรับข้อมูลเข้ามา ประมวลผลผ่านอัลกอริทึม แล้วส่งคำสั่งไปควบคุมมอเตอร์ให้ขยับ

สมัยก่อน เวลาวิศวกรจะสร้างหุ่นยนต์ขึ้นมาสักตัว พวกเขาต้องเขียนลูปคำนวณวงจร **PID (Proportional-Integral-Derivative)** แยกกันสำหรับมอเตอร์ทุกตัว ทั้งเขียนโค้ดอ่านค่า Encoder (Sensor) ทั้งเขียนโค้ดสั่งงาน PWM (Actuator) พอต้องเปลี่ยนฮาร์ดแวร์ทีนึง ก็ต้องมารื้อโค้ดเขียนใหม่ทั้งหมด! มันคือการ "Reinvent the wheel" หรือการประดิษฐ์ล้อซ้ำแล้วซ้ำเล่าที่น่าเบื่อมาก

เพื่อแก้ปัญหานี้ ทีมพัฒนา ROS จึงได้สร้างเฟรมเวิร์ก **`ros2_control`** ขึ้นมา มันออกแบบมาให้เราแยก "สมการคณิตศาสตร์ (Controllers)" ออกจาก "การคุยกับฮาร์ดแวร์จริง (Hardware Interface)" อย่างเด็ดขาด! ทำให้เราสามารถนำโค้ด PID สุดเทพที่เขียนไว้ ไปใช้ได้กับทั้งหุ่นยนต์ในโลกจำลอง (Gazebo) และหุ่นยนต์ของจริงได้โดยไม่ต้องแก้โค้ดเลยแม้แต่บรรทัดเดียว!

## 3. 🧠 แก่นวิชา (Core Concepts)

สถาปัตยกรรมของ **`ros2_control`** มีความสง่างามและถูกแบ่งออกเป็นส่วนๆ อย่างชัดเจน ดังนี้ครับ:

*   **Controller Manager (ผู้จัดการใหญ่):** 
    เปรียบเสมือนผู้จัดการโรงงาน มีหน้าที่รับผิดชอบในการโหลด (Load), ตั้งค่า (Configure), และจัดการวงจรชีวิตของ Controllers ต่างๆ รวมถึงควบคุมการทำงานในระดับ Real-time loop โดยจะเรียกฟังก์ชัน `update()` ของแต่ละ Controller ที่กำลังทำงานอยู่ (Active) ตามลำดับ
*   **Controllers (วิศวกรผู้คำนวณ):** 
    คือปลั๊กอินหรือโมดูลที่รับผิดชอบในการคำนวณตรรกะการควบคุม เช่น การควบคุมตำแหน่ง (Position), ความเร็ว (Velocity), หรือแรงบิด (Effort/Torque) Controller จะนำค่าที่วัดได้จริง (Feedback) มาหักลบกับเป้าหมาย (Setpoint) เพื่อหาค่า Error และส่งต่อให้สมการอย่าง PID คำนวณหาค่าคำสั่ง (Command)
*   **Resource Manager (ผู้ดูแลคลังทรัพยากร):** 
    ทำหน้าที่จัดการทรัพยากรฮาร์ดแวร์อย่างมีประสิทธิภาพ เพื่อให้แน่ใจว่า Controller หลายๆ ตัวสามารถแชร์การเข้าถึง Hardware Interfaces ได้โดยไม่เกิดความขัดแย้ง (Conflicts)
*   **Hardware Interface (ล่ามแปลภาษา):**
    นี่คือส่วนที่ต้องไปคลุกคลีกับฮาร์ดแวร์ (หรือ Simulator) โดยตรง มันจะเตรียม State Interface (อ่านค่าจาก Sensor เช่น Encoder) และ Command Interface (เขียนค่าลง Actuator เช่น สั่ง Voltage หรือ PWM) เอาไว้ให้ Controller เรียกใช้งาน 

{{< figure src="ros2-control-system-diagram.jpg" alt="รูปประกอบสถาปัตยกรรม ros2_control" >}}

## 4. 💻 ร่ายมนต์โค้ดและคำสั่ง (Show me the Code)

เพื่อให้ระบบ `ros2_control` ทำงานได้ เราต้องนิยามมันลงไปในพิมพ์เขียว (URDF/Xacro) ของหุ่นยนต์เสียก่อนครับ ลองดูตัวอย่างการเซ็ตอัปกัน:

**1. การฝัง `ros2_control` ลงใน URDF/Xacro:**
เราจะใช้แท็ก `<ros2_control>` เพื่อบอกว่าข้อต่อ (Joint) นี้รองรับการควบคุมแบบไหน
```xml
<!-- 1. ประกาศ Plugin สำหรับคุยกับ Simulator (ในที่นี้คือ Ignition Gazebo) -->
<ros2_control name="IgnitionSystem" type="system">
  <hardware>
    <plugin>ign_ros2_control/IgnitionSystem</plugin>
  </hardware>
  
  <!-- 2. กำหนดชนิดของการควบคุมให้กับข้อต่อที่ชื่อ revolute_joint -->
  <joint name="revolute_joint">
    <!-- Command Interface (สิ่งที่เราสั่งมอเตอร์ได้) -->
    <command_interface name="position" />
    <command_interface name="velocity" />
    
    <!-- State Interface (สิ่งที่เราอ่านกลับมาจากเซ็นเซอร์ได้) -->
    <state_interface name="position"/>
    <state_interface name="velocity"/>
    <state_interface name="effort"/>
  </joint>
</ros2_control>
```

**2. การตั้งค่า Controller (YAML Configuration):**
ต่อมา เราต้องสร้างไฟล์ `controller.yaml` เพื่อบอก Controller Manager ว่าเราจะใช้ Controller ตัวไหนบ้าง
```yaml
controller_manager:
  ros__parameters:
    update_rate: 100 # ความถี่ของ Real-time loop (Hz)

    # ประกาศใช้งาน Joint State Broadcaster (ทำหน้าที่อ่านและพ่นค่า /joint_states)
    joint_state_broadcaster:
      type: joint_state_broadcaster/JointStateBroadcaster

    # ประกาศใช้งาน Velocity Controller
    velocity_control:
      type: forward_command_controller/ForwardCommandController

# กำหนดสเปกของ Velocity Controller
velocity_control:
  ros__parameters:
    joints:
      - revolute_joint
    interface_name: velocity # ควบคุมด้วยโหมดความเร็ว
```

**3. คำสั่ง Terminal สำหรับจัดการ Controllers:**
น้องๆ สามารถใช้คำสั่ง `ros2 control` เพื่อเช็คและสั่งงาน Controller Manager ได้แบบ Real-time เลยครับ!
```bash
# ขอดูรายชื่อ Controller ทั้งหมดที่โหลดอยู่ในระบบตอนนี้
ros2 control list_controllers

# โหลด Controller ใหม่ (แต่ยังไม่เริ่มทำงาน)
ros2 control load_controller velocity_control

# สั่งให้ Controller เริ่มทำงาน (หรือใช้บริการ switch_controller เพื่อสลับโหมดการทำงาน)
ros2 control set_controller_state velocity_control active
```

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips)

ในฐานะที่พี่ปรับจูน PID จนมอเตอร์ไหม้มาหลายตัว นี่คือสิ่งที่น้องต้องระวังครับ:

*   **อย่าสับสนระหว่าง "Load", "Start" และ "Stop":** 
    วงจรชีวิตของ Controller แบ่งเป็นสเตตัสชัดเจน เมื่อเรา "Load" (หรือ Initialized) Controller เข้ามา มันยังไม่ทำงาน จนกว่าเราจะสั่ง "Start" (Running) และเมื่อเราสั่ง "Stop" มันแค่หยุดการทำงาน (กลับไปสถานะ Initialized) แต่มันยังไม่ได้ถูก "Unload" ออกจากหน่วยความจำนะครับ!
*   **ศัตรูตัวฉกาจของ Controller คือ "ความหน่วง" (Delays):** 
    ลูปการทำงาน `update()` ของ Controller Manager (Real-time loop) จะต้องเสร็จสิ้นภายในเวลาที่กำหนด (เช่น 1ms สำหรับ 1000Hz) ถ้าน้องเขียนโค้ดที่ต้องรอข้อมูลนานๆ (Blocking) หรือใช้ Network ที่มี Delay สูงๆ ระบบ PID จะคำนวณพลาด หุ่นยนต์จะเกิดการสั่น (Oscillate) และไม่เสถียรในที่สุด!
*   **จาก Simulation สู่ฮาร์ดแวร์จริง:** 
    ความเจ๋งของเฟรมเวิร์กนี้คือ ตอนน้องทำ Simulation ใน Gazebo น้องใช้ `ign_ros2_control/IgnitionSystem` เป็นปลั๊กอินฮาร์ดแวร์ พอหุ่นยนต์ประกอบเสร็จ น้องแค่ไปเขียน C++ Hardware Interface สำหรับไดรเวอร์มอเตอร์ของน้อง แล้วเปลี่ยนแท็ก `<plugin>` ใน URDF... ปุ๊ง! โค้ด Navigation และ Controller ทั้งหมดของน้องจะทำงานกับหุ่นจริงได้ทันทีแบบไร้รอยต่อครับ!

## 6. 🏁 บทสรุป (To be continued...)

**`ros2_control`** คือระบบสมองกลหลังบ้านที่คอยซัพพอร์ตการเคลื่อนไหวของหุ่นยนต์ มันช่วยคัดกรองความวุ่นวายของการสื่อสารฮาร์ดแวร์ออกจากลอจิกการคำนวณ ทำให้โค้ดของเราเป็นระเบียบ (Modular) จัดการทรัพยากรได้อย่างชาญฉลาด และสามารถสลับโหมดควบคุมระหว่าง Position, Velocity, และ Effort ได้อย่างอิสระ

เมื่อหุ่นยนต์ของเรารู้จักตัวเองผ่าน TF และมีกล้ามเนื้อที่ถูกควบคุมอย่างแม่นยำด้วย `ros2_control` แล้ว มันก็พร้อมที่จะก้าวเดินออกไปสำรวจโลกกว้างครับ! ในตอนต่อไป พี่จะพาน้องๆ ไปพบกับความท้าทายระดับมหากาพย์ นั่นคือ **Navigation Stack (Nav2)** ที่จะทำให้หุ่นยนต์สามารถวางแผนเส้นทางหลบหลีกสิ่งกีดขวางได้เองอย่างอัจฉริยะ เตรียมตัวจับพวงมาลัยให้แน่น แล้วพบกันครับ!

---
**ต้องการที่ปรึกษาด้านการออกแบบสถาปัตยกรรมหุ่นยนต์ (Robotics) และระบบ Automation ให้กับองค์กรของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและพัฒนาซอฟต์แวร์แบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p