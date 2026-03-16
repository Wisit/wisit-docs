---
title: "ตอนที่ 15: Launch Files ศิลปะการเปิดร้อย Nodes ในคลิกเดียว"
weight: 1215
date: 2026-03-16
author: วิสิทธิ์ แผ้วกระโทก
tags: ["ROS 1", "ROS 2", "Launch Files", "Python", "Parameters", "Automation"]
---
{{< figure src="launch-files-cover.jpg" alt="รูปปกบทความ" >}}

## 1. 🎯 ตอนที่ 15: Launch Files ศิลปะการเปิดร้อย Nodes ในคลิกเดียว

สวัสดีครับน้องๆ วิศวกรหุ่นยนต์ทุกคน! เดินทางมาถึงตอนที่ 15 กันแล้ว ตอนนี้น้องๆ คงสามารถเขียน Node ทั้ง Publisher, Subscriber, Service ไปจนถึง Action และสร้าง Custom Message กันได้คล่องมือแล้วใช่ไหมครับ 

แต่ในโลกความเป็นจริงของหุ่นยนต์ระดับอุตสาหกรรม หุ่นยนต์หนึ่งตัวไม่ได้มีแค่ 2-3 Nodes นะครับ! ลองนึกถึงหุ่นยนต์ขับเคลื่อนอัตโนมัติ (AMR) ที่มีทั้งโหนดกล้อง, LIDAR, มอเตอร์ล้อซ้าย-ขวา, ระบบ Path Planning, ระบบ Localization... รวมๆ แล้วอาจจะมีมากกว่า 20 Nodes! ถ้าน้องต้องมานั่งเปิด Terminal 20 หน้าต่าง แล้วพิมพ์ `ros2 run` ทีละบรรทัด พร้อมกับพิมพ์กำหนด Parameter ให้แต่ละตัว... พี่รับรองว่าน้องจะหมดไฟในการทำหุ่นยนต์ไปซะก่อน! 

วันนี้พี่จะพาไปรู้จักกับ "เวทมนตร์แห่งการ Automation" ที่วิศวกร ROS ทุกคนต้องใช้ นั่นคือ **Launch Files** ครับ แค่พิมพ์คำสั่งเดียว หุ่นยนต์ทั้งระบบก็จะลุกขึ้นมามีชีวิตทันที!

## 2. 📖 เปิดฉาก (The Hook)

สมัยที่พี่ทำโปรเจกต์หุ่นยนต์ส่งของตัวแรก พี่พยายามเขียน Bash Script ยัดคำสั่ง `rosrun` (ในยุค ROS 1) ลงไปต่อๆ กันยาวเหยียด ผลคือพอมี Node นึงพัง พี่ก็ไม่รู้ว่าตัวไหนพัง แถมการจะส่งค่า Parameter ต่างๆ เข้าไปในแต่ละ Node ก็วุ่นวายจนโค้ดอ่านแทบไม่รู้เรื่อง

ปัญหานี้แก้ไขได้ด้วยสถาปัตยกรรมของ ROS ที่เรียกว่า **Launch Files** ครับ มันเปรียบเสมือน "ผู้จัดการวงออร์เคสตรา (Orchestration)" ที่คอยสั่งการว่า นักดนตรี (Node) คนไหนต้องเริ่มเล่นตอนไหน ใครต้องใช้โน้ตเพลง (Parameters) อะไร และถ้าต้องการเปลี่ยนคีย์เพลงกะทันหัน (Launch Arguments) ก็สามารถสั่งการจากส่วนกลางได้เลยโดยไม่ต้องไปไล่บอกทีละคน นี่แหละครับคือศิลปะของการจัดการระบบที่ซับซ้อน!

## 3. 🧠 แก่นวิชา (Core Concepts)

ก่อนที่เราจะไปลุยโค้ด มาทำความเข้าใจวิวัฒนาการและโครงสร้างของมันกันก่อนครับ:

*   **ยุค ROS 1 (XML ล้วน):** ในอดีต Launch file จะถูกเขียนด้วยโครงสร้าง XML (`.launch`) ซึ่งอ่านง่าย แต่มีข้อจำกัดตรงที่มันขาดความยืดหยุ่นในการเขียนลอจิก เช่น การทำเงื่อนไข If-Else หรือ Loop
*   **ยุค ROS 2 (พลังแห่ง Python):** แม้ ROS 2 จะยังรองรับ XML และ YAML อยู่ แต่ภาษาสายหลักที่ถูกดันขึ้นมาเป็นพระเอกคือ **Python (`.launch.py`)** ครับ! การใช้ Python ทำให้เราสามารถเขียนลอจิกควบคุมการเปิดปิด Node ได้อย่างอิสระ ตรวจสอบตัวแปรระบบ (Environment Variables) หรือแม้แต่การเช็คสเปคฮาร์ดแวร์ก่อนรัน Node ก็ทำได้หมด!

โครงสร้างหลักของ Python Launch File จะประกอบด้วย 3 ส่วนสำคัญ (The Holy Trinity):
1.  **`LaunchDescription`:** คือกล่องเปล่าๆ ที่ทำหน้าที่เป็น "รายการสิ่งที่ต้องทำ (To-Do List)" เราจะเอาทุกอย่างมายัดใส่กล่องนี้เพื่อส่งกลับไปให้ระบบประมวลผล
2.  **`Node`:** คำสั่งในการเรียกใช้โปรแกรมย่อย เราสามารถระบุชื่อ Package, ชื่อ Executable, เปลี่ยนชื่อ Node กะทันหัน (Name) และยัดค่า **Parameters** เข้าไปในนี้ได้เลย
3.  **`DeclareLaunchArgument` และ `LaunchConfiguration`:** นี่คือการรับค่า **Arguments** จากผู้ใช้งานผ่าน Terminal (เช่น ให้ผู้ใช้พิมพ์สั่งว่าอยากรันในโหมด Simulator หรือหุ่นยนต์จริง) แล้วส่งค่านั้นไปเปลี่ยนพฤติกรรมของ Node 

{{< figure src="launch-files-diagram.jpg" alt="รูปประกอบโครงสร้าง ROS 2 Launch File" >}}

## 4. 💻 ร่ายมนต์โค้ดและคำสั่ง (Show me the Code)

เรามาดูหน้าตาของ Python Launch File ฉบับมาตรฐานกันครับ โค้ดนี้จะสาธิตการรับ Argument จาก Terminal และการตั้งค่า Parameters ให้กับ Node ไปพร้อมๆ กัน!

**1. โค้ด Python Launch File (`my_robot.launch.py`):**
```python
from launch import LaunchDescription
from launch_ros.actions import Node
from launch.actions import DeclareLaunchArgument
from launch.substitutions import LaunchConfiguration

def generate_launch_description():
    # 1. สร้างตัวแปรรับค่าจาก Terminal (Launch Arguments)
    # สมมติว่าให้ผู้ใช้เลือกว่าจะใช้ Simulation Mode หรือไม่ (ค่าเริ่มต้นคือ false)
    use_sim_time_arg = DeclareLaunchArgument(
        'use_sim_time',
        default_value='false',
        description='Enable simulation mode for testing'
    )

    # ดึงค่าที่ผู้ใช้พิมพ์มาเก็บไว้ใน LaunchConfiguration
    sim_time_config = LaunchConfiguration('use_sim_time')

    # 2. นิยาม Node ที่ต้องการรัน พร้อมตั้งค่า Parameters
    my_custom_node = Node(
        package='my_robot_package',      # ชื่อแพ็กเกจ
        executable='robot_controller',   # ชื่อโปรแกรมที่คอมไพล์แล้ว
        name='main_controller',          # จับเปลี่ยนชื่อ Node เสียใหม่ (ถ้าต้องการ)
        output='screen',                 # ให้ปริ้นต์ Log ออกมาที่หน้าจอ Terminal ด้วย
        parameters=[                     # ยัด Parameters แบบ Dictionary เข้าไป
            {'max_speed': 2.5},
            {'use_sim_time': sim_time_config} # ใช้ค่า Argument ที่รับมาเมื่อกี้!
        ],
        remappings=[                     # เปลี่ยนชื่อ Topic (Remap) กลางอากาศได้ด้วย
            ('/cmd_vel', '/robot/cmd_vel')
        ]
    )

    # 3. เอาทุกอย่างยัดใส่กล่อง LaunchDescription แล้วส่งคืนให้ระบบรัน!
    return LaunchDescription([
        use_sim_time_arg,
        my_custom_node
    ])
```

**2. วิธีการรันผ่าน Terminal:**
เมื่อเราเขียนเสร็จและคอมไพล์แล้ว (อย่าลืมคอมไพล์นะครับ!) เราจะสั่งรันผ่าน Terminal แบบนี้ครับ:

```bash
# รัน Launch File ด้วยค่า Default (use_sim_time จะเป็น false)
ros2 launch my_robot_package my_robot.launch.py

# รันโดยส่งค่า Argument แทรกเข้าไป (use_sim_time จะกลายเป็น true)
ros2 launch my_robot_package my_robot.launch.py use_sim_time:=true
```

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips)

ในฐานะที่พี่คลุกคลีกับการทำหุ่นยนต์มา นี่คือจุดที่วิศวกรมือใหม่มักจะ "ตกม้าตาย" เรื่อง Launch Files ครับ:

*   **หลุมพราง `setup.py` (File Not Found!):**
    ถ้าน้องเขียน Node ด้วย Python (ใช้ `setup.py`) แล้วสั่ง `ros2 launch` ระบบฟ้องว่าหาไฟล์ไม่เจอ! สาเหตุคือ **น้องลืมไปอัปเดตไฟล์ `setup.py` ครับ!** น้องต้องไปเพิ่มบรรทัดนี้ในหมวด `data_files` เสมอ เพื่อบอกให้ `colcon build` ก๊อปปี้ไฟล์ Launch ไปไว้ในโฟลเดอร์ `install`:
    ```python
    ('share/' + package_name + '/launch', ['launch/my_robot.launch.py']),
    ```
*   **วิชาแยกร่าง (IncludeLaunchDescription):**
    ถ้าหุ่นยนต์มี 30 Nodes น้องอย่าเขียนทุกอย่างไว้ในไฟล์เดียวนะครับ! โค้ดจะยาวเป็นพันบรรทัด ให้ใช้เทคนิค "ไฟล์เรียกไฟล์" โดยแยกเป็นไฟล์ย่อย (เช่น `sensors.launch.py`, `navigation.launch.py`) แล้วใช้คำสั่ง `IncludeLaunchDescription(PythonLaunchDescriptionSource(...))` ในไฟล์หลักเพื่อดึงไฟล์ย่อยเหล่านั้นมาทำงานร่วมกัน นี่คือ Best Practice ของการทำ Modular Design ครับ!
*   **การโยนไฟล์ YAML เข้าไปใน Launch:**
    ในตัวอย่างพี่ใส่ Parameter เป็นดิกชันนารี `{'max_speed': 2.5}` ซึ่งเหมาะกับพารามิเตอร์น้อยๆ แต่ถ้า Node นั้นมีพารามิเตอร์สัก 50 ตัว ให้สร้างไฟล์ `config.yaml` ขึ้นมา แล้วใช้คำสั่ง `os.path.join(get_package_share_directory('ชื่อแพ็ก'), 'config', 'config.yaml')` เพื่อโหลดไฟล์นั้นไปโยนใส่บรรทัด `parameters=[...]` ของ Node ได้เลยครับ สะอาดและเป็นระเบียบกว่าเยอะ!

## 6. 🏁 บทสรุป (To be continued...)

**Launch Files** คือเครื่องมือทรงพลังที่เปลี่ยนงาน "กรรมกร" ให้กลายเป็นระบบ "อัตโนมัติ" การเข้าใจวิธีใช้ `LaunchDescription`, การตั้งค่า `Node` และการโยน `LaunchArgument` คือทักษะที่จะแยกความเป็นมือสมัครเล่นออกจากมืออาชีพครับ!

ตอนนี้น้องๆ มีอาวุธในการเปิดระบบขนาดใหญ่พร้อมกันแล้ว! ในตอนต่อไป พี่จะพาน้องๆ ไปลุยกับระบบสถาปัตยกรรมระดับปรมาจารย์ของ ROS นั่นคือ **TF (Transform Framework)** ที่ใช้ในการอ้างอิงพิกัดสามมิติระหว่างชิ้นส่วนต่างๆ ของหุ่นยนต์กันครับ เตรียมตัวยืดเส้นยืดสายรอไว้ได้เลย!

---
**ต้องการที่ปรึกษาด้านการออกแบบสถาปัตยกรรมหุ่นยนต์ (Robotics) และระบบ Automation ให้กับองค์กรของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและพัฒนาซอฟต์แวร์แบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p