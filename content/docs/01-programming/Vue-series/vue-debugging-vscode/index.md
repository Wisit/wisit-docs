---
title: "เทคนิคการ Debugging Vue.js ใน VS Code อย่างมืออาชีพ"
weight: 280
date: 2026-03-11
author: วิสิทธิ์ แผ้วกระโทก
tags: ["Vue3", "Debugging", "VS Code", "Vue DevTools", "Frontend Architecture"]
---
{{< figure src="vue-debugging-vscode-cover.jpg" alt="Aesthetic study notes about Vue.js Debugging in VS Code on an iPad screen" >}}

## 1. 🎯 ตอนที่ 28: บอกลา console.log() สู่เทคนิค Debugging Vue.js อย่างมือโปร

## 2. 📖 เปิดฉาก (The Hook)
น้องๆ เคยตกอยู่ในวงจรนี้ไหมครับ? เวลาโค้ดพัง หรือข้อมูลไม่ยอมแสดงผลบนหน้าจอ สิ่งแรกที่เราทำคือการรัวพิมพ์ `console.log('test 1')`, `console.log('data:', data.value)` ลงไปแทบจะทุกบรรทัด พอดูผลเสร็จก็ต้องมานั่งไล่ลบ `console.log` ทิ้ง เผลอลืมลบติดไปบน Production ก็มีให้เห็นบ่อยๆ การทำแบบนี้เปรียบเสมือน "การคลำหาของในห้องมืดด้วยไฟฉายกระบอกเล็กๆ" ครับ

แต่ในฐานะ Senior Developer เรามีวิธีที่สง่างามกว่านั้น! วันนี้พี่จะพามาเปิดสวิตช์ไฟให้สว่างจ้าไปทั้งห้อง ด้วยเทคนิคการ **Debugging** ผ่าน VS Code โดยตรง เราจะสามารถหยุดเวลา (Pause Execution) ของแอปพลิเคชัน เพื่อแอบดูตัวแปรทุกตัวในระบบ พร้อมกับใช้อาวุธลับอย่าง **Vue DevTools** เพื่อเจาะดูไส้ในของ Component จิบกาแฟให้พร้อม แล้วมาปราบ Bug กันครับ!

## 3. 🧠 แก่นวิชา (Core Concepts)

### 🗺️ 1. Source Maps (ลายแทงขุมทรัพย์)
เวลาที่เราเขียน Vue (โดยเฉพาะ Composition API หรือ TypeScript) โค้ดของเราจะถูก Compile และบีบอัดจนอ่านแทบไม่รู้เรื่องก่อนนำไปรันบน Browser ครับ การที่เราจะให้ VS Code รู้ว่าโค้ดที่รันอยู่ ตรงกับไฟล์ `.vue` บรรทัดไหนของเรา เราต้องเปิดใช้งาน **Source Map** ก่อน มันคือ "ลายแทง" ที่เชื่อมโค้ดหน้างานกลับมาหาโค้ดต้นฉบับครับ

### 🛑 2. Breakpoints & Debug Console (จุดหยุดเวลา)
*   **Breakpoints (จุดแดงตัวหยุดเวลา):** คือการที่เราคลิกที่หน้าตัวเลขบรรทัดใน VS Code (Gutter) เพื่อสร้างจุดสีแดง เป็นการสั่งให้ระบบ "หยุดการทำงานทันที" เมื่อโค้ดวิ่งมาถึงบรรทัดนี้
*   **Debug Console:** เมื่อโค้ดหยุด เราสามารถใช้หน้าต่าง Debug Console ใน VS Code เพื่อพิมพ์ชื่อตัวแปร หรือรันคำสั่ง JavaScript เพื่อทดสอบค่าต่างๆ ได้แบบ Real-time (ไม่ต้องพึ่ง `console.log` อีกต่อไป!)

### 🥽 3. Vue DevTools (แว่นตา X-Ray เจาะลึก Component)
เป็น Browser Extension (Chrome/Firefox) ที่คนทำ Vue ขาดไม่ได้! มันช่วยให้เรามองเห็น "Component Tree" (แผนผังครอบครัว) ของหน้าเว็บ พร้อมทั้งดูและแก้ไขค่า State (Data, Props) ได้สดๆ บน Browser เลยครับ

{{< figure src="vue-debugging-vscode-diagram.jpg" alt="Diagram showing the Vue.js Debugging workflow connecting VS Code and Vue DevTools" >}}

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code)

เรามาดูขั้นตอนการเซ็ตอัป VS Code ให้พร้อมสำหรับการ Debugging กันครับ

### Step 1: เปิดใช้งาน Source Map ในโปรเจกต์
หากน้องๆ ใช้ **Vue CLI (Webpack)** ต้องเข้าไปเพิ่มโค้ดในไฟล์ `vue.config.js` เพื่อบอกให้มันสร้าง Source Map ตอน Development ครับ:

```javascript
// vue.config.js
module.exports = {
  // เปิดใช้งาน Source Map เพื่อง่ายต่อการ Debug ใน VS Code
  configureWebpack: {
    devtool: 'source-map' 
  }
}
```
*(หมายเหตุ: หากน้องๆ ใช้ Vite ตามมาตรฐานใหม่ Source Map มักจะถูกเปิดไว้ให้อยู่แล้วในโหมด dev ครับ)*

### Step 2: สร้างไฟล์ตั้งค่าการรันใน VS Code (`launch.json`)
ให้ไปที่เมนู **Run and Debug (Ctrl+Shift+D)** ใน VS Code กดคลิก "create a launch.json file" เลือกเป็น Web App (Chrome/Edge) แล้วปรับแก้ไฟล์ให้หน้าตาประมาณนี้ครับ:

```json
// .vscode/launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "chrome", // หรือ "edge"
      "request": "launch",
      "name": "Launch Vue App against localhost",
      "url": "http://localhost:8080", // เปลี่ยนเป็น Port ที่โปรเจกต์ของน้องรันอยู่ (เช่น 5173 สำหรับ Vite)
      "webRoot": "${workspaceFolder}/src",
      "sourceMapPathOverrides": {
        "webpack:///src/*": "${webRoot}/*"
      }
    }
  ]
}
```

### Step 3: วิธีการลงสนามจริง (Set Breakpoints)
สมมติเรามี Component สำหรับดึงข้อมูลผู้ใช้งาน เราสามารถไปคลิกตรงหน้าตัวเลขบรรทัดเพื่อวาง "จุดสีแดง (Breakpoint)" ได้เลย:

```vue
<script setup>
import { ref } from 'vue'

const user = ref(null)
const isLoading = ref(false)

const fetchUser = async (id) => {
  isLoading.value = true
  
  // 🔴 เรากดสร้าง Breakpoint สีแดงไว้ที่บรรทัดนี้ใน VS Code
  const response = await fetch(`https://api.example.com/users/${id}`) 
  const data = await response.json()
  
  // 🔴 หรือสร้าง Breakpoint ไว้ตรงนี้ เพื่อดูค่า data ก่อนโยนใส่ user.value
  user.value = data 
  isLoading.value = false
}
</script>
```
**สิ่งที่เกิดขึ้น:**
1. กด F5 ใน VS Code เพื่อเปิด Browser แบบ Debug Mode
2. พอกดปุ่มเรียกฟังก์ชัน `fetchUser` ในหน้าเว็บ... หน้าเว็บจะค้าง (Pause) ทันที!
3. เด้งกลับมาดูใน VS Code น้องๆ จะเห็นโค้ดหยุดทำงานที่บรรทัดนั้น สามารถเอาเมาส์ไปชี้ที่ตัวแปร `data` เพื่อดูค่าข้างในได้เลย หรือพิมพ์ `data` ในช่อง **Debug Console** ด้านล่างเพื่อตรวจสอบแบบละเอียดครับ!

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips)

**🚨 1. หยุดตกหลุมพราง `console.log(refValue)`**
ใน Vue 3 Composition API ตัวแปรที่เราสร้างด้วย `ref()` หรือ `reactive()` มันถูกห่อหุ้มด้วย **Proxy** (ตัวแทน) ครับ ถ้าน้องๆ ใช้ `console.log()` พิมพ์มันออกมาดูบนเบราว์เซอร์ตรงๆ น้องมักจะเจอ Object หน้าตาประหลาดๆ ที่เต็มไปด้วย `[[Handler]]` และ `[[Target]]` จนหาข้อมูลจริงไม่เจอ! 
*ทางแก้:* ให้ใช้ **Vue DevTools** ใน Browser เสมอครับ! เมื่อเปิดแท็บ Vue ใน F12 Developer Tools น้องจะเห็นข้อมูลที่ถูกแกะกล่อง (Unwrap) ออกมาให้ดูอย่างสวยงาม อ่านง่ายสบายตา

**🔍 2. พลังแห่งการ "แก้ไขค่าสดๆ" ผ่าน Vue DevTools**
รู้หรือไม่ครับว่าใน Vue DevTools น้องๆ สามารถคลิกที่ตัวแปร (เช่น `isLoading: true`) แล้วพิมพ์แก้ค่ามันเป็น `false` ได้เลยทันที! หน้าเว็บจะอัปเดต (Reactive) ตามค่าที่น้องแอบแก้ทันที เทคนิคนี้มีประโยชน์มากเวลาที่เราอยากทดสอบ UI ในสถานะต่างๆ โดยไม่ต้องไปนั่งแก้โค้ดและรอให้แอปรันใหม่ครับ

**⏳ 3. Time-Travel Debugging (ไทม์แมชชีนแห่ง Vuex/Pinia)**
หากโปรเจกต์ของน้องใช้ระบบ State Management อย่าง **Pinia** หรือ **Vuex** ตัว Vue DevTools จะมีแท็บแยกมาให้เลย มันจะคอยบันทึกทุกๆ การเปลี่ยนแปลงข้อมูล (Mutations) เป็นลำดับขั้น น้องสามารถกดปุ่ม "ย้อนกลับ (Revert)" เพื่อถอยหลังสถานะของแอปพลิเคชันกลับไปเหมือนตอนก่อนที่ผู้ใช้จะกดปุ่มได้! นี่คือฟีเจอร์ระดับเทพที่ช่วยหาบั๊กประหลาดๆ ได้ดีเยี่ยมครับ

## 6. 🏁 บทสรุป (To be continued...)
การขยับจากการใช้ `console.log` มาใช้ระบบ **Breakpoints ใน VS Code** และ **Vue DevTools** คือจุดเปลี่ยนผ่านสำคัญที่แยกมือใหม่กับมืออาชีพออกจากกันครับ มันช่วยประหยัดเวลาในการเดาสุ่ม และทำให้เราเข้าใจวงจรชีวิตการไหลของข้อมูล (Data Flow) ภายใน Component ได้อย่างทะลุปรุโปร่ง

ตอนนี้น้องๆ ก็มีอาวุธสำหรับจัดการบั๊กครบมือแล้ว! ในบทความหน้า เราจะมาดูวิชาขั้นสุดยอดของการนำ Vue.js ไปผสานพลังกับเทคโนโลยีฝั่ง Backend เพื่อสร้างแطปพลิเคชันที่สมบูรณ์แบบ (Full-Stack) เตรียมตัวให้พร้อมครับ!

---

**ต้องการที่ปรึกษาและพัฒนาระบบ Web Application หรือ Frontend Architecture ให้กับธุรกิจของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและพัฒนาซอฟต์แวร์แบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p