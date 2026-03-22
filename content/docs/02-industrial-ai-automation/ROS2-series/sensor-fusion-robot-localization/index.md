---
title: "ตอนที่ 25: Sensor Fusion รวมพลังเซนเซอร์ด้วย Robot_Localization"
weight: 1225
date: 2026-03-21
author: วิสิทธิ์ แผ้วกระโทก
tags: ["ROS 2", "Robotics", "Sensor Fusion", "EKF", "robot_localization", "Odometry"]
---
{{< figure src="sensor-fusion-robot-localization-cover.jpg" alt="รูปปกบทความ" >}}

## 1. 🎯 ตอนที่ 25: Sensor Fusion รวมพลังเซนเซอร์ด้วย Robot_Localization

สวัสดีครับน้องๆ วิศวกรหุ่นยนต์ทุกคน! ในตอนที่แล้ว พี่ได้เล่าถึงฝันร้ายของการทำ Odometry ด้วย Wheel Encoders เพียงอย่างเดียวไปแล้วใช่ไหมครับ? ว่าพอล้อหุ่นยนต์เกิดการลื่นไถล (Slip) หรือวิ่งบนพื้นขรุขระ ค่าตำแหน่งมันจะเพี้ยนสะสม (Drift) จนหุ่นยนต์คิดว่าตัวเองทะลุกำแพงไปแล้ว!

"ถ้าอย่างนั้น เราก็ใช้ IMU หรือ GPS มาช่วยสิพี่?" ถูกต้องครับ! แต่ปัญหาคือ IMU ก็มี Noise (สัญญาณรบกวน) ส่วน GPS ก็อัปเดตช้าและใช้ในร่มไม่ได้... แล้วเราจะเชื่อใครดีล่ะ? 

คำตอบของวิศวกรระดับโลกก็คือ **"ไม่ต้องเชื่อใครแบบ 100% แต่จงเอาข้อดีของทุกคนมารวมกัน!"** และนั่นคือที่มาของศาสตร์ที่เรียกว่า **Sensor Fusion** ครับ วันนี้พี่จะพาไปเจาะลึกแพ็กเกจระดับเทพใน ROS 2 ที่ชื่อว่า `robot_localization` ซึ่งจะใช้คณิตศาสตร์ของ Extended Kalman Filter (EKF) มาปั่นรวมข้อมูลเซนเซอร์ให้หุ่นยนต์ของเรารู้ตำแหน่งที่แม่นยำที่สุดครับ!

## 2. 📖 เปิดฉาก (The Hook)

ลองจินตนาการว่าน้องถูกปิดตา แล้วต้องเดินไปหยิบของในห้องที่คุ้นเคยดูนะครับ 
*   **Wheel Encoder** คือ "การนับก้าว" น้องจำได้ว่าก้าวไป 5 ก้าว น่าจะถึงโต๊ะแล้ว (แต่จริงๆ อาจจะก้าวสั้นไป)
*   **IMU** คือ "สัมผัสในหูชั้นใน" ที่คอยบอกว่าน้องกำลังหันหน้าไปทางไหน หรือกำลังเอียงตัวอยู่หรือเปล่า
*   **GPS** คือ "เพื่อนที่ยืนอยู่มุมห้อง" คอยตะโกนบอกตำแหน่งน้องทุกๆ 1 วินาที (แต่อาจจะกะระยะคลาดเคลื่อนไปบ้าง)

ถ้าน้องฟังแค่เสียงก้าวเดินอย่างเดียว น้องอาจจะเดินชนกำแพง แต่ถ้าน้องเอาข้อมูลทั้งหมดนี้มา "ชั่งน้ำหนัก" ว่าจังหวะไหนควรเชื่อความรู้สึกตัวเอง จังหวะไหนควรเชื่อเพื่อนที่ตะโกนบอก น้องก็จะเดินถึงเป้าหมายได้อย่างแม่นยำ! ในโลกของ ROS เราเรียกกระบวนการชั่งน้ำหนักและรวมข้อมูลแบบนี้ว่า **Sensor Fusion** และอัลกอริทึมพระเอกที่ทำหน้าที่นี้ก็คือ **Kalman Filter** ครับ!,

## 3. 🧠 แก่นวิชา (Core Concepts)

แพ็กเกจมาตรฐานใน ROS 2 ที่วิศวกรทุกคนต้องใช้คือ **`robot_localization`** ครับ ภายในแพ็กเกจนี้จะมี Node ที่ทำงานด้วยสมการ **Extended Kalman Filter (EKF)** ซึ่งออกแบบมาเพื่อรับมือกับระบบที่ไม่มีความเป็นเชิงเส้น (Non-linear dynamics) อย่างหุ่นยนต์โดยเฉพาะ,

EKF ทำงานแบบวนลูป 2 ขั้นตอน (Predict & Update) คล้ายๆ กับการเก็งข้อสอบครับ:
*   **1. Prediction Step (การทำนาย):** EKF จะใช้แบบจำลองฟิสิกส์ (Kinematic Model) ทำนายว่า "จากความเร็วเดิมในวินาทีที่แล้ว ตอนนี้หุ่นยนต์น่าจะขยับไปอยู่ตรงไหนแล้ว?"
*   **2. Update Step (การอัปเดตและแก้ไข):** EKF จะกวาดข้อมูลจากเซนเซอร์ของจริง (Encoder, IMU, GPS) เข้ามาดู จากนั้นมันจะดูค่าความน่าเชื่อถือที่เรียกว่า **Covariance (เมทริกซ์ความแปรปรวน)** เซนเซอร์ตัวไหนมี Noise ต่ำ (Covariance ต่ำ) EKF จะ "ให้ค่าน้ำหนัก" เซนเซอร์ตัวนั้นสูง แล้วเอาไปดึงค่าที่ทำนายไว้ตอนแรกให้กลับมาอยู่ในร่องในรอยที่ถูกต้องที่สุดครับ!,,

ผลลัพธ์ที่ได้จาก `robot_localization` คือค่า Odometry ที่ถูกฟิลเตอร์จนเนียนกริบ (พ่นออกทาง Topic `/odometry/filtered`) และมันยังช่วยทำหน้าที่ประกาศค่าพิกัดความสัมพันธ์ (TF Tree) จากเฟรม `odom` ไปยัง `base_link` ให้อย่างเสถียรสุดๆ อีกด้วยครับ!

{{< figure src="sensor-fusion-robot-localization-diagram.jpg" alt="รูปประกอบสถาปัตยกรรม EKF ใน robot_localization" >}}

## 4. 💻 ร่ายมนต์โค้ดและคำสั่ง (Show me the Code)

มาลงมือทำกันเลยครับ! สมมติว่าเรามี Topic `/odom` (จาก Encoders) และ `/imu/data` (จากเซนเซอร์ IMU) เราจะจับมันมาฟิวชันกัน

**1. ติดตั้งแพ็กเกจ:**
```bash
sudo apt install ros-foxy-robot-localization
```

**2. สร้างไฟล์ Configuration (`ekf.yaml`):**
นี่คือหัวใจหลักครับ! เราต้องบอก EKF ว่าจะรับ Topic อะไรบ้าง และ "จะดึงตัวแปรไหนจากเซนเซอร์ตัวไหน",

```yaml
ekf_filter_node:
  ros__parameters:
    frequency: 30.0 # ความถี่ในการคำนวณ EKF loop (Hz)
    sensor_timeout: 0.1
    two_d_mode: true # เปิดโหมด 2D (หุ่นยนต์วิ่งบนพื้นเรียบ ไม่ได้บิน)
    
    # กำหนด Coordinate Frames
    map_frame: map
    odom_frame: odom
    base_link_frame: base_link
    world_frame: odom # ในระดับ Local Fusion ให้ world เป็น odom
    
    # เซนเซอร์ตัวที่ 1: Wheel Odometry
    odom0: /odom
    # เมทริกซ์ 15 ค่า: [X, Y, Z, Roll, Pitch, Yaw, Vx, Vy, Vz, Vroll, Vpitch, Vyaw, Ax, Ay, Az]
    odom0_config: [false, false, false, # ไม่เชื่อ Position X, Y, Z จากล้อโดยตรง (เพราะมัน Drift)
                   false, false, false, 
                   true,  true,  false, # เชื่อความเร็วเชิงเส้น (Vx, Vy) จากล้อ!
                   false, false, true,  # เชื่อความเร็วในการเลี้ยว (Vyaw) จากล้อ
                   false, false, false]
    
    # เซนเซอร์ตัวที่ 2: IMU
    imu0: /imu/data
    imu0_config: [false, false, false,
                  false, false, true,  # เชื่อทิศทาง (Yaw / Heading) จาก IMU แบบ 100%!
                  false, false, false,
                  false, false, true,  # เชื่อความเร็วในการหมุน (Vyaw) จาก IMU
                  true,  true,  false] # เชื่อความเร่ง (Ax, Ay) จาก IMU
```

**3. สร้าง Launch File (`localization.launch.py`):**
```python
from launch import LaunchDescription
from launch_ros.actions import Node
import os
from ament_index_python.packages import get_package_share_directory

def generate_launch_description():
    # ชี้เป้าไปที่ไฟล์ ekf.yaml ของเรา
    config_dir = os.path.join(get_package_share_directory('my_robot_pkg'), 'config', 'ekf.yaml')

    return LaunchDescription([
        Node(
            package='robot_localization',
            executable='ekf_node',
            name='ekf_filter_node',
            output='screen',
            parameters=[config_dir],
            # ให้ Node นี้พ่นข้อมูลออกไปที่ Topic ใหม่
            remappings=[('/odometry/filtered', '/odom_combined')] 
        )
    ])
```

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips)

ในฐานะที่พี่ปรับจูน EKF จนกราฟแกว่งเป็นรถไฟเหาะมาแล้ว นี่คือจุดตายที่น้องต้องระวังครับ:

*   **อย่าเชื่อสิ่งที่มีความหมายซ้ำซ้อนกัน (Redundant Absolute Variables):**
    ในเมทริกซ์ `config` (ค่า true/false) ถ้าน้องตั้งค่าให้รับ `Yaw` (ทิศทาง) ทั้งจาก Odometry และ IMU พร้อมกัน (ตั้งเป็น `true` ทั้งคู่) ถ้าระบบ 2 ตัวนี้เกิดเริ่มต้นที่มุม 0 องศาไม่เท่ากัน EKF จะตีกันเองและหมุนวนไปมาครับ! กฎเหล็กคือ สำหรับตัวแปรที่เป็นค่าสัมบูรณ์ (Absolute Pose เช่น X, Y, Yaw) ให้เลือกดึงจากเซนเซอร์ที่แม่นยำที่สุดเพียง "ตัวเดียว" เท่านั้น (เช่น เอา Yaw จาก IMU) ส่วนความเร็ว (Velocity) สามารถดึงมาฟิวชันกันได้ครับ
*   **Covariance คือพระเจ้า:**
    ต่อให้น้องเซ็ต `ekf.yaml` ถูกเป๊ะ แต่ถ้าไดรเวอร์ IMU ของน้องพ่นค่า Covariance Matrix ออกมาเป็น 0 (แปลว่าเซนเซอร์ตัวนี้บอกว่าตัวเองแม่นยำ 100% ไร้ความผิดพลาด) EKF จะเชื่อ IMU หัวปักหัวปำ และโยนข้อมูลจากล้อทิ้งทันที! น้องต้องไปแก้โค้ดฝั่งไดรเวอร์ฮาร์ดแวร์ให้พ่นค่า Covariance ที่สะท้อนถึง Noise ในโลกความเป็นจริงด้วยครับ,
*   **การเพิ่ม GPS (Global Fusion):**
    ถ้าหุ่นยนต์วิ่ง Outdoor และมี GPS เข้ามาช่วย ใน `robot_localization` เราจะไม่ได้ยัด GPS เข้า `ekf_node` ตัวเดิมตรงๆ นะครับ แต่เราจะเปิด EKF node อีกตัวหนึ่งควบคู่กับ `navsat_transform_node` โดยเปลี่ยน `world_frame` ให้เป็น `map` แทน เพื่อแยกระหว่างการคำนวณระยะสั้น (Odom) กับการคำนวณพิกัดโลก (Global Map) ออกจากกันอย่างชัดเจน

## 6. 🏁 บทสรุป (To be continued...)

**Sensor Fusion** ด้วย EKF คือกระบวนการที่แยก "ของเล่น" ออกจาก "หุ่นยนต์อุตสาหกรรม" อย่างแท้จริงครับ! การนำ Wheel Encoders มาอุดช่องโหว่ความไม่นิ่งของ IMU และนำ IMU มาแก้การลื่นไถลของล้อ ทำให้หุ่นยนต์ของเรามีประสาทสัมผัสที่เฉียบคม รับรู้การเคลื่อนที่ของตัวเองได้อย่างเสถียร ไร้รอยต่อ

แต่ถึงแม้จะฟิวชันจนได้ Odometry ที่ดีแค่ไหน เมื่อวิ่งไปสัก 1 ชั่วโมง มันก็ยังมี Drift เล็กๆ สะสมอยู่ดีครับ... วิธีเดียวที่จะล้างบาง Drift เหล่านี้ให้เป็น 0 คือการให้หุ่นยนต์ "มองเห็นและจดจำ" สภาพแวดล้อมรอบตัว! ในตอนต่อไป พี่จะพาน้องๆ ไปพบกับอัลกอริทึมที่ยิ่งใหญ่ที่สุดของวงการหุ่นยนต์ นั่นคือ **SLAM (Simultaneous Localization and Mapping)** เตรียมเลเซอร์ (LiDAR) ของน้องให้พร้อม แล้วมาสร้างแผนที่กันครับ!

---
**ต้องการที่ปรึกษาด้านการออกแบบสถาปัตยกรรมหุ่นยนต์ (Robotics) และระบบ Automation ให้กับองค์กรของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและพัฒนาซอฟต์แวร์แบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p