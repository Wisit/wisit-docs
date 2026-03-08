---
title: "เจาะลึก Selection (if, elif, else): ทางแยกและการตัดสินใจในโลกของ Control Statements"
weight: 70
date: 2026-03-08
author: วิสิทธิ์ แผ้วกระโทก
tags: ["Python 3", "Control Flow", "If-Else", "Selection", "Best Practices"]
---

{{< figure src="python-selection-control-flow-cover.jpg" alt="ภาพปกบทความ เจาะลึก Selection (if, elif, else) ใน Python" >}}

## 1. 🎯 ชื่อบทความ (Title): เจาะลึก Selection (if, elif, else): ทางแยกและการตัดสินใจในโลกของ Control Statements

## 2. 👋 เกริ่นนำ (Introduction)
สวัสดีครับเพื่อนๆ นักพัฒนาและวิศวกรทุกคน! ลองจินตนาการดูว่า ถ้าโปรแกรมคอมพิวเตอร์ทำงานแบบเส้นตรง (Sequential) คืออ่านคำสั่งจากบรรทัดบนลงล่างไปเรื่อยๆ อย่างเดียวโดยไม่สามารถ "คิด" หรือ "ตัดสินใจ" อะไรได้เลย ระบบ Automation ของเราคงจะทื่อหน้าน่าดูใช่ไหมครับ? เช่น หุ่นยนต์ AGV เจอกำแพงขวางหน้าแต่มันก็ยังเดินชนต่อไปเพราะโปรแกรมไม่มีเงื่อนไขให้มันหยุด

นี่แหละครับคือจุดที่ **Control Statements (คำสั่งควบคุมทิศทาง)** เข้ามามีบทบาทสำคัญ ในบริบทที่กว้างขึ้นของวิทยาการคอมพิวเตอร์ โครงสร้างการควบคุมจะแบ่งเป็น 3 กลุ่มหลัก คือ การทำงานตามลำดับ (Sequence), การทำซ้ำ (Iteration/Loops) และสิ่งที่เราจะพูดถึงกันในวันนี้คือ **การเลือกทำ (Selection)** หรือการสร้างเงื่อนไข `if`, `elif`, `else` นั่นเองครับ วันนี้พี่วิสิทธิ์จะพาน้องๆ ไปดูว่า Python ออกแบบระบบ Selection มาอย่างไร ทำไมมันถึงเรียบง่าย อ่านง่าย และป้องกันบั๊กระดับโลกที่ภาษาอื่นมักจะเจอกันได้แบบอยู่หมัด!

## 3. 📖 เนื้อหาหลัก (Core Concept)
ในภาพรวมของ Control Statements การทำ Selection คือการสร้าง "สาขา (Branches)" หรือทางแยกให้กับโปรแกรม โดยใช้ตรรกะแบบ Boolean (True/False) เป็นตัวตัดสินใจว่าโปรแกรมควรจะเดินไปทางไหน ซึ่งใน Python มีปรัชญาการออกแบบที่น่าสนใจดังนี้ครับ:

*   **The if Statement (ผู้เฝ้าประตู):**
    คำสั่ง `if` คือจุดเริ่มต้นของการตัดสินใจ ถ้านิพจน์ (Expression) ที่เราทดสอบมีค่าความจริงเป็น `True` โค้ดที่อยู่ในบล็อกนั้นก็จะถูกทำงาน (Execute) แต่ถ้าเป็น `False` ก็จะถูกข้ามไป
*   **The if-else Statement (ทางแยกสองแพร่ง):**
    ใช้จัดการเมื่อเรามี 2 ทางเลือกที่ชัดเจน "ถ้าเงื่อนไขจริงทำแบบนี้ ถ้าไม่ใช่ (else) ให้ไปทำอีกแบบ" 
*   **The elif Statement (ทางเลือกหลายเงื่อนไข):**
    `elif` ย่อมาจาก "else if" ใช้เมื่อเรามีตัวเลือกมากกว่า 2 ทาง (Multiway branching) โปรแกรมจะไล่เช็คเงื่อนไขไปทีละตัวจากบนลงล่าง เมื่อเจอเงื่อนไขไหนที่เป็น `True` อันแรก มันจะทำงานในบล็อกนั้นแล้ว **กระโดดออกจากโครงสร้างนี้ทันที (Short-circuiting)** โดยไม่สนใจเงื่อนไขด้านล่างอีกเลย 
*   **ความมหัศจรรย์ของ Indentation (การเว้นวรรค):**
    รู้หรือไม่ครับว่าในภาษาอย่าง C หรือ Java เราต้องใช้วงเล็บปีกกา `{}` เพื่อครอบบล็อกของเงื่อนไข ซึ่งมักก่อให้เกิดบั๊กคลาสสิกที่เรียกว่า **"Dangling else"** (การจับคู่ else ผิดตัว เพราะปีกกาหรือการจัดหน้าบรรทัดไม่ชัดเจน) แต่ใน Python บังคับให้เราใช้ **การย่อหน้า (Indentation)** เพื่อกำหนดขอบเขตบล็อก (Block of code) โค้ดที่คุณเห็น (WYSIWYG - What You See Is What You Get) คือลอจิกที่คอมพิวเตอร์เห็นจริงๆ สิ่งนี้บังคับให้ Python Code ทั่วโลกมีโครงสร้างที่สะอาดตาและอ่านง่ายเหมือนภาษาอังกฤษครับ!
*   **ทางเลือกแทน Switch-Case:**
    ในอดีต (ก่อน Python 3.10) Python ตั้งใจที่จะ **ไม่มี** คำสั่ง `switch/case` เหมือนภาษาอื่น โดยมองว่าการใช้ `if-elif-else` ต่อกัน หรือการใช้ Dictionary Mapping นั้นให้ความยืดหยุ่นและเป็น Pythonic way มากกว่าครับ (แม้ปัจจุบันจะมี `match-case` แล้วก็ตาม แต่พื้นฐาน `elif` ก็ยังเป็นหัวใจหลัก)

{{< figure src="python-selection-control-flow-diagram.jpg" alt="แผนภาพ Flowchart แสดงการทำงานของ if, elif และ else ในระดับ Control Flow" >}}

## 4. 💻 ตัวอย่างโค้ด (Code Example)
ลองมาดูตัวอย่างการเขียนสคริปต์สกัดข้อมูลอุณหภูมิและประเมินสถานะของเครื่องจักรในโรงงาน สังเกตการใช้ Indentation ที่ชัดเจนครับ:

```python
def check_machine_status(temperature):
    """
    ฟังก์ชันประเมินสถานะเครื่องจักรจากอุณหภูมิ
    สาธิตการใช้ Multiway Branching (if-elif-else)
    """
    print(f"Reading Temperature: {temperature}°C")
    
    # เริ่มต้น Control Statement สำหรับ Selection
    if temperature > 85.0:
        # ถ้ามากกว่า 85 ให้หยุดเครื่องทันที
        status = "CRITICAL"
        action = "EMERGENCY_STOP"
        
    elif temperature >= 70.0:
        # ถ้าอยู่ระหว่าง 70.0 - 85.0 (elif จะทำเมื่อ if ด้านบนเป็น False)
        status = "WARNING"
        action = "TURN_ON_COOLING_FAN"
        
    elif temperature < 10.0:
        # กรณีอุณหภูมิต่ำเกินไป
        status = "COLD_STATE"
        action = "WARM_UP_REQUIRED"
        
    else:
        # หากไม่ตรงกับเงื่อนไขใดเลย (อุณหภูมิปกติ)
        status = "NORMAL"
        action = "CONTINUE_OPERATION"
        
    # คืนค่ากลับไปเป็น Tuple
    return status, action

# ทดสอบการเรียกใช้งาน
current_temp = 75.5
machine_status, required_action = check_machine_status(current_temp)

print(f"Status: {machine_status} -> Action: {required_action}")
```

## 5. 🛡️ ข้อควรระวัง / Best Practices
ในการเขียน Selection Statements ให้เป็นระดับ Senior Dev (Clean Code) มีจุดที่ต้องระวังดังนี้ครับ:

*   **หลีกเลี่ยงการทำ Nested If ที่ลึกเกินไป (Deep Nesting):** การเขียน `if` ซ้อน `if` ซ้อน `if` ลึกลงไปเรื่อยๆ (บางคนเรียกว่า Arrow Code เพราะรูปทรงเหมือนหัวลูกศร) จะทำให้โค้ดอ่านยากและดูแลรักษายากมาก ควรใช้ Logical Operators อย่าง `and` / `or` เข้ามาช่วยรวบเงื่อนไข หรือใช้เทคนิค "Return Early" เพื่อตัดเคสที่ไม่ใช่ออกไปก่อน
*   **ใช้ Ternary Operator สำหรับเงื่อนไขสั้นๆ:** หากลอจิกของคุณมีแค่การกำหนดค่าตัวแปร เช่น 
    `if is_online: status = "ON" else: status = "OFF"` 
    คุณสามารถยุบให้เป็นบรรทัดเดียวได้แบบ Pythonic style คือ: 
    `status = "ON" if is_online else "OFF"` จะทำให้โค้ดกระชับขึ้นมากครับ
*   **อย่าเปรียบเทียบกับ True/False ตรงๆ:** กฎเหล็กของ PEP 8 คือห้ามเขียน `if flag == True:` แต่ให้เขียน `if flag:` แทน เพราะ Python มีแนวคิดเรื่อง Truthiness อยู่แล้ว (อ่านเพิ่มเติมได้ในบทความเรื่อง Boolean ของเราครับ)

## 6. 🏁 สรุป (Conclusion & CTA)
ในบริบทของ Control Statements แล้ว Selection (`if`, `elif`, `else`) เปรียบเสมือน "สมอง" ที่คอยคัดกรองและตัดสินใจให้ระบบของเราเดินไปในทิศทางที่ถูกต้องครับ เสน่ห์ของ Python คือการใช้ Indentation มาบังคับสร้างบล็อกของตรรกะ ทำให้ปัญหาที่ซ่อนเร้นอย่าง Dangling else หมดไป แม้ Python จะทำงานผ่าน Interpreter ซึ่งอาจจะไม่เร็วเท่า C/C++ แต่ด้วยไวยากรณ์ที่ออกแบบมาอย่างพิถีพิถันนี้ ช่วยประหยัดเวลาของโปรแกรมเมอร์ในการ Debug ไปได้มหาศาลเลยทีเดียว!

---

**ต้องการที่ปรึกษาและพัฒนาระบบ Automation ให้กับโรงงานของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและติดตั้งระบบแบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p