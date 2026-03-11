---
title: "Props: การส่งผ่านข้อมูลจากพ่อสู่ลูก (Parent to Child)"
weight: 160
date: 2026-03-11
author: วิสิทธิ์ แผ้วกระโทก
tags: ["Vue3", "Props", "Component", "One-Way Data Flow", "Frontend"]
---
{{< figure src="vue-props-parent-to-child-cover.jpg" alt="Aesthetic study notes about Vue.js Props and parent-to-child data flow on an iPad screen" >}}

## 1. 🎯 ตอนที่ 16: Props เคล็ดวิชาส่งผ่านข้อมูลจากพ่อสู่ลูก (Parent to Child)

## 2. 📖 เปิดฉาก (The Hook)
ในบทที่แล้วพี่ได้พาน้องๆ ไปดูวิธีการหั่นหน้าเว็บออกเป็นชิ้นส่วนเล็กๆ อย่าง Component กันไปแล้วใช่ไหมครับ? แต่ปัญหาที่ตามมาติดๆ ก็คือ... "แล้วชิ้นส่วนพวกนี้มันจะคุยกันยังไงล่ะ?" 

ลองจินตนาการว่าเราสร้าง Component ชื่อ `<UserProfile />` ขึ้นมาเป็นตัวต่อ Lego หน้าตาหล่อเท่ แต่ถ้าเราไม่สามารถบอกมันได้ว่า "เฮ้! ครั้งนี้จงแสดงชื่อ John นะ ส่วนอีกที่นึงจงแสดงชื่อ Jane นะ" Component ของเราก็จะกลายเป็นรูปปั้นหินที่แข็งทื่อและนำไปใช้ซ้ำ (Reuse) ไม่ได้เลย

นี่แหละครับคือจุดที่ **Props (Properties)** เข้ามาเป็นพระเอกขี่ม้าขาว! Props เปรียบเสมือน "ช่องทางส่งจดหมาย" ที่ Parent Component (ตัวแม่) ใช้สำหรับโยนข้อมูลลงไปให้ Child Component (ตัวลูก) นำไปแสดงผล วันนี้เราจะมาเจาะลึกวิธีการตั้งค่าช่องทางนี้ให้ปลอดภัย รัดกุม และถูกหลักสถาปัตยกรรมกันครับ จิบกาแฟให้พร้อมแล้วลุยกันเลย!

## 3. 🧠 แก่นวิชา (Core Concepts)

### 📥 1. Props คืออะไร?
**Props** (ย่อมาจาก Properties) คือ Custom Attributes ที่เราลงทะเบียนไว้ใน Component เพื่อรอรับข้อมูลที่ถูกส่งมาจาก Parent Component ถ้าให้เปรียบเทียบง่ายๆ การสร้าง Component ก็เหมือนการสร้าง "ฟังก์ชัน" และ Props ก็คือ "พารามิเตอร์ (Parameters)" ที่เรารับเข้ามาในฟังก์ชันนั้นนั่นเอง

### 🚦 2. กฎเหล็ก One-Way Data Flow (การไหลของข้อมูลทางเดียว)
ในโลกของ Vue.js ข้อมูลจาก Props จะไหลแบบ **"บนลงล่าง (One-Way-Down Binding)"** เสมอ หมายความว่า:
*   เมื่อข้อมูลฝั่ง Parent เปลี่ยนแปลง... ข้อมูลฝั่ง Child จะถูกอัปเดตตามโดยอัตโนมัติ
*   แต่ **Child ไม่มีสิทธิ์แอบไปแก้ไขค่า Props โดยตรงเด็ดขาด!**

เหตุผลที่ Vue บังคับกฎนี้ก็เพื่อป้องกันไม่ให้แอปพลิเคชันของเรามีข้อมูลไหลย้อนศรจนพันกันยุ่งเหยิง ถ้า Child แต่ละตัวแอบดัดแปลงข้อมูลของ Parent ได้ตามใจชอบ เวลาเกิดบั๊กขึ้นมา เราจะตามหาต้นตอไม่ได้เลยครับว่าใครเป็นคนแก้ข้อมูลนี้ (ทำให้ Data Flow เข้าใจยาก)

### 🛡️ 3. Props Validation (ด่านตรวจคนเข้าเมือง)
ในเมื่อ Parent เป็นคนโยนข้อมูลมาให้ เราจะไว้ใจ Parent ได้ 100% ไหม? คำตอบคือ "ไม่!" ครับ บ่อยครั้งที่เราคาดหวังจะได้ตัวเลข (Number) แต่ Parent ดันส่งข้อความ (String) มาให้ซะงั้น จนทำให้ระบบพัง 
Vue จึงมีระบบ **Props Validation** ให้เราสามารถกำหนด "สเปก" ของข้อมูลได้ เช่น บังคับชนิดข้อมูล (Type), กำหนดว่าต้องส่งมาเสมอ (Required), หรือถ้าไม่ส่งมาจะใช้ค่าเริ่มต้นอะไร (Default) ซึ่งจะช่วยให้การทำงานร่วมกันเป็นทีมปลอดภัยขึ้นมหาศาลครับ

{{< figure src="vue-props-parent-to-child-diagram.jpg" alt="Diagram showing Vue One-Way Data Flow from Parent Component passing Props down to Child Component" >}}

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code)

เรามาดูวิธีประกาศรับ Props ในรูปแบบ `<script setup>` สมัยใหม่ ควบคู่กับการทำ Validation ไปเลยครับ

**ชิ้นส่วนลูก: `src/components/UserProfile.vue`**
```vue
<template>
  <div class="profile-card">
    <!-- นำข้อมูลจาก Props มาแสดงผลได้ทันที -->
    <h3>{{ name }}</h3>
    <p>อายุ: {{ age }} ปี</p>
    <p>สถานะ: {{ isActive ? 'ออนไลน์' : 'ออฟไลน์' }}</p>
  </div>
</template>

<script setup>
// ประกาศรับ Props ด้วย defineProps แบบ Object Syntax เพื่อทำ Validation
const props = defineProps({
  // 1. ตรวจสอบ Type พื้นฐาน และบังคับว่าต้องส่งมา (required)
  name: {
    type: String,
    required: true 
  },
  // 2. กำหนด Type และใส่ค่าเริ่มต้น (default) หากไม่ได้ส่งมา
  age: {
    type: Number,
    default: 18 
  },
  // 3. กำหนดให้เป็น Boolean
  isActive: {
    type: Boolean,
    default: false
  },
  // 4. การใช้ Custom Validator ตรวจสอบความถูกต้องลึกซึ้ง
  role: {
    type: String,
    validator: (value) => {
      // ค่าที่ส่งมาต้องเป็นหนึ่งในนี้เท่านั้น ถ้าไม่ใช่จะแจ้งเตือนใน Console
      return ['admin', 'user', 'guest'].includes(value)
    }
  }
})

// สามารถดึงค่า props มาใช้งานใน script ได้ด้วย props.name, props.age
console.log('User name is: ', props.name)
</script>
```

**ชิ้นส่วนแม่: `src/App.vue` (ผู้เรียกใช้)**
```vue
<template>
  <div class="app-container">
    <h2>จัดการผู้ใช้งาน</h2>
    
    <!-- 1. ส่งข้อมูลแบบ Static (เป็น String เสมอ) -->
    <UserProfile name="สมชาย" role="admin" />

    <!-- 2. ส่งข้อมูลแบบ Dynamic ด้วย v-bind (:) -->
    <!-- :age="25" คือการบอก Vue ว่านี่คือ Number ไม่ใช่ String "25" -->
    <UserProfile 
      :name="currentUser.name" 
      :age="currentUser.age" 
      :is-active="true"
      role="user"
    />
  </div>
</template>

<script setup>
import { ref } from 'vue'
import UserProfile from './components/UserProfile.vue'

const currentUser = ref({
  name: 'สมหญิง',
  age: 25
})
</script>
```

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips)

**🚨 หลุมพรางคลาสสิก: "แอบแก้ Props ใน Component ลูก"**
เมื่อมือใหม่รับ Props เข้ามา มักจะเผลอเขียนโค้ดเพื่อแก้ไขค่ามันโดยตรงแบบนี้:
```javascript
const props = defineProps(['count'])

const increment = () => {
  // ❌ โค้ดนี้จะพัง! และ Vue จะด่าใน Console ว่า "Avoid mutating a prop directly"
  props.count++ 
}
```
**ทำไมถึงพัง?** เพราะทุกครั้งที่ Parent Component ทำการ Re-render มันจะดันข้อมูลก้อนใหม่ลงมาทับค่าที่เราแอบแก้ไว้จนหายเกลี้ยงครับ!

**✅ วิธีแก้ที่ถูกต้อง (The Right Way):**
ถ้าเราต้องการเอาข้อมูลจาก Props มาใช้เป็น "ค่าเริ่มต้น" และอยากจะแก้ไขมันต่อใน Component ลูก ให้เรานำ Props ไปโยนใส่ `ref` เพื่อสร้าง **Local State (ตัวแปรส่วนตัว)** แทนครับ:

```javascript
import { ref } from 'vue'

const props = defineProps(['initialCount'])

// ✅ ก๊อปปี้ค่าเริ่มต้นมาสร้างเป็น State ส่วนตัว ทีนี้ก็แก้ไขได้อย่างอิสระแล้ว!
const localCount = ref(props.initialCount) 

const increment = () => {
  localCount.value++ // ทำงานได้อย่างสมบูรณ์ ไม่กระทบ Parent
}
```

*⚠️ คำเตือนระดับ Senior:* ใน JavaScript พวก Object หรือ Array มันถูกส่งค่าเป็น **"Reference (การอ้างอิง)"** นั่นหมายความว่า ถ้า Props ที่ส่งมาเป็น Object แล้วน้องๆ แอบไปแก้ `props.user.name = 'Mew'` แม้ Vue จะไม่ด่าตรงๆ แต่ค่าใน Parent จะถูกเปลี่ยนตามไปด้วย! ซึ่งนี่คือ "บั๊ก" ที่หาตัวจับยากที่สุด (Side Effect) ท่องไว้เลยครับว่า **"อย่าดัดแปลงข้อมูล Props เด็ดขาด"**

## 6. 🏁 บทสรุป (To be continued...)
**Props** คือกุญแจสำคัญที่ทำให้ Component ของเรามีความยืดหยุ่น (Reusable) และสื่อสารกับ Parent ได้อย่างมีระเบียบ การใช้ Props ควบคู่กับการทำ Validation จะทำให้โครงสร้างโค้ดของคุณแข็งแกร่ง และทำงานร่วมกับเพื่อนในทีมได้อย่างสบายใจ 

แต่เดี๋ยวก่อน... ในเมื่อ Component ลูกไม่ได้รับอนุญาตให้แก้ข้อมูลของแม่ แล้วถ้าเรา "จำเป็นต้องอัปเดตข้อมูล" กลับไปล่ะ จะต้องทำยังไง?

คำตอบรอเราอยู่ในตอนหน้าครับ! เราจะมากางคัมภีร์เรียนรู้วิธีการส่งสัญญาณเตือนจากลูกขึ้นไปหาแม่ด้วยการใช้ **Custom Events (Emits)** เตรียมตัวให้พร้อม แล้วพบกันครับ!

---

**ต้องการที่ปรึกษาและพัฒนาระบบ Web Application หรือ Frontend Architecture ให้กับธุรกิจของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและพัฒนาซอฟต์แวร์แบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p