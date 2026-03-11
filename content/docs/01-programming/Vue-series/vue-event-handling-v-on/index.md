---
title: "Event Handling: รับมือกับทุก Click และ Keypress ด้วย v-on"
weight: 140
date: 2026-03-11
author: วิสิทธิ์ แผ้วกระโทก
tags: ["Vue3", "Events", "v-on", "Modifiers", "Frontend"]
---
{{< figure src="vue-event-handling-v-on-cover.jpg" alt="Aesthetic study notes about Event Handling and v-on directive in Vue.js on an iPad screen" >}}

## 1. 🎯 ตอนที่ 14: จับทุกความเคลื่อนไหว สยบทุกการคลิกด้วยเวทมนตร์แห่ง v-on

## 2. 📖 เปิดฉาก (The Hook)
น้องๆ เคยเขียน JavaScript แบบดั้งเดิมเพื่อจัดการปุ่มคลิกไหมครับ? เราต้องเริ่มจาก `document.getElementById('btn')` ตามด้วย `.addEventListener('click', function)` แล้วถ้าในหน้าเว็บมีปุ่มสัก 20 ปุ่มล่ะ? โค้ดของเราคงยาวเหยียดและเต็มไปด้วยการจับโยง DOM ที่น่าปวดหัวแน่ๆ แถมบางทีลืมลบ Event ทิ้งตอนเปลี่ยนหน้าเว็บ ก็ทำเอา Memory Leak กินแรมเครื่องไปอีก!

ในโลกของ Vue.js การรับมือกับการกระทำของผู้ใช้ (User Interaction) หรือที่เราเรียกว่า **Event Handling** นั้นถูกออกแบบมาให้ "งดงามและสั้นกระชับ" ชนิดที่ว่าอ่านปุ๊บรู้ปั๊บว่าปุ่มนี้ทำหน้าที่อะไร วันนี้พี่จะพามาจิบกาแฟ แล้วเจาะลึกคัมภีร์การใช้คำสั่ง `v-on` พร้อมกับอาวุธลับที่เรียกว่า "Event Modifiers" ที่จะช่วยให้เราไม่ต้องเขียนโค้ดซ้ำซากจำเจอีกต่อไปครับ

## 3. 🧠 แก่นวิชา (Core Concepts)

### 🎧 1. `v-on` (เครื่องดักฟังเสียงกระซิบจาก User)
เราใช้ Directive ที่ชื่อว่า `v-on` เพื่อคอยดักจับ DOM Events (เช่น การคลิก, การพิมพ์, การเลื่อนเมาส์) และสั่งให้มันไปเรียกใช้งานฟังก์ชัน (Methods) ที่เราเตรียมไว้
*   **ไวยากรณ์เต็ม:** `v-on:click="doSomething"`
*   **ท่าไม้ตาย (Shorthand):** เพราะเราใช้มันบ่อยมาก Vue จึงให้เราย่อเหลือแค่สัญลักษณ์ **`@`** ครับ เช่น `@click="doSomething"` (โปรระดับ Senior ใช้ท่านี้กันทุกคน!)

### 📥 2. การส่งพารามิเตอร์ (Passing Parameters)
บางครั้งปุ่มของเราก็ต้องการส่ง "ของติดไม้ติดมือ" ไปให้ฟังก์ชันด้วย เช่น คลิกปุ่มแล้วส่ง ID ของสินค้าไปลบ เราสามารถใส่วงเล็บและส่งค่าไปได้เลย เช่น `@click="deleteItem(123)"`

### 🛡️ 3. Event Modifiers (ฟิลเตอร์กรองเหตุการณ์)
ใน JavaScript ปกติ เวลาเราเจอ Event ที่ไม่อยากให้มันทำงานตามพฤติกรรมเดิม เรามักจะต้องเขียน `event.preventDefault()` (เช่น ห้ามฟอร์มรีเฟรชหน้าเว็บ) หรือ `event.stopPropagation()` (ห้ามการส่งต่อ Event ไปหา Element แม่) ไว้บรรทัดแรกของฟังก์ชันเสมอ
Vue เห็นว่ามันน่าเบื่อ เลยสร้าง **Modifiers** เป็นคำสั่งต่อท้ายด้วยจุด (`.`) ให้เราใช้ที่ Template ได้เลย:
*   **`.prevent`**: สั่งห้ามพฤติกรรมเริ่มต้น (เทียบเท่า `preventDefault()`) เหมาะสุดๆ กับการใส่ในแท็ก `<form @submit.prevent="...">`
*   **`.stop`**: สั่งหยุดการกระจายของ Event (Event Bubbling) ไม่ให้ทะลุขึ้นไปหา Element พ่อแม่ (เทียบเท่า `stopPropagation()`)
*   **`.once`**: ทำงานแค่ครั้งเดียวเท่านั้น คลิกครั้งที่สองจะนิ่งเงียบ

### ⌨️ 4. Key Modifiers (ดักจับคีย์บอร์ดแบบมือสไนเปอร์)
เวลาทำช่องค้นหา เรามักจะรอให้ผู้ใช้กดปุ่ม "Enter" ก่อนค่อยค้นหาข้อมูล Vue อนุญาตให้เราต่อท้ายชื่อปุ่มไปได้เลย เช่น `@keyup.enter="search"` (โคตรเท่และอ่านง่าย!)

{{< figure src="vue-event-handling-v-on-diagram.jpg" alt="Diagram showing the flow of DOM events intercepted by Vue v-on directive and modified by event modifiers" >}}

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code)

เรามาดูการรวมมิตรท่าต่างๆ ของ `v-on` ในรูปแบบ Single-File Component (SFC) ด้วย `<script setup>` กันครับ:

```vue
<template>
  <div class="checkout-card" @click="logBubble">
    <h2>ตะกร้าสินค้าของนักเวท</h2>

    <!-- 1. Shorthand พื้นฐาน (ไม่ส่งพารามิเตอร์) -->
    <button @click="sayHi">ทักทายแม่ค้า</button>

    <!-- 2. การส่งพารามิเตอร์ (Parameter Passing) -->
    <button @click="addPotion(5)">เพิ่มยาฟื้นพลัง 5 ขวด</button>
    <p>จำนวนยาในกระเป๋า: {{ potionCount }}</p>

    <!-- 3. Event Modifier (.prevent) สยบการรีเฟรชหน้าจอของ Form -->
    <!-- พอกด Submit ปุ๊บ หน้าเว็บจะไม่กระตุกรีเฟรชเลย -->
    <form @submit.prevent="checkout">
      <!-- 4. Key Modifier (.enter) ดักการกด Enter -->
      <input 
        type="text" 
        v-model="discountCode" 
        @keyup.enter="applyDiscount" 
        placeholder="ใส่โค้ดแล้วกด Enter" 
      />
      <button type="submit">ชำระเงิน</button>
    </form>

    <!-- 5. Event Modifier (.stop) หยุดการ Bubbling -->
    <!-- ถ้าไม่ใส่ .stop เวลากดปุ่มนี้ มันจะทะลุไปเรียก logBubble ของ div ตัวนอกด้วย! -->
    <button @click.stop="deleteCart">เทตะกร้าทิ้ง (ห้ามบอกใคร)</button>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const potionCount = ref(0)
const discountCode = ref('')

// ฟังก์ชันแบบไม่รับพารามิเตอร์
const sayHi = () => {
  alert('สวัสดีจ้า รับอะไรดีคะ?')
}

// ฟังก์ชันแบบรับพารามิเตอร์ (amount)
const addPotion = (amount) => {
  potionCount.value += amount
}

const applyDiscount = () => {
  console.log(`ตรวจสอบโค้ดส่วนลด: ${discountCode.value}`)
}

// เราไม่ต้องมานั่งเขียน event.preventDefault() ในนี้แล้ว! เพราะ Template จัดการให้
const checkout = () => {
  console.log('กำลังพาไปหน้าจ่ายเงิน...')
}

const deleteCart = () => {
  potionCount.value = 0
  console.log('ลบของเกลี้ยงตะกร้าแล้ว')
}

const logBubble = () => {
  console.log('🔴 เกิดการคลิกที่พื้นที่พื้นหลังของ Card!')
}
</script>

<style scoped>
.checkout-card { padding: 20px; background: #f0f4f8; border-radius: 10px; }
button, input { margin: 5px 0; display: block; }
</style>
```

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips)

**✨ ความลับของตัวแปรพิเศษ `$event`**
บางครั้งเราต้องการส่งพารามิเตอร์ของเราเองเข้าไปใน Method **พร้อมกับ** อยากได้ข้อมูลของ Native DOM Event (เช่น อยากรู้ว่าพิกัดเมาส์ที่คลิกอยู่ X,Y เท่าไหร่) ถ้าเราเขียนแค่ `@click="doAction(10)"` ตัว Event เดิมจะหายไปครับ 

ทางแก้ระดับโปรคือการใช้ตัวแปรพิเศษที่ชื่อว่า **`$event`** ยัดเข้าไปด้วยแบบนี้ครับ:
```html
<button @click="doAction(10, $event)">คลิกฉันสิ</button>
```
```javascript
const doAction = (score, event) => {
  console.log(`ได้คะแนน ${score}`)
  console.log(`พิกัดที่คลิกคือ X: ${event.clientX}`) // เราได้ Native Event คืนมาแล้ว!
}
```

**⚠️ ข้อควรระวัง: การเรียงลำดับ Modifiers (Chaining order matters!)**
เราสามารถต่อร้อย (Chain) modifiers เข้าด้วยกันได้ครับ เช่น `@click.stop.prevent` แต่จำไว้ว่า **"ลำดับมีผลเสมอ"** 
*   `@click.prevent.stop` จะป้องกันการทำงานเริ่มต้น (Prevent) ก่อน แล้วค่อยหยุดการทะลุขึ้น (Stop)
*   มันจะอ่านจากซ้ายไปขวา ดังนั้นจัดเรียงตามลำดับความสำคัญของ Business Logic ที่น้องๆ ต้องการเสมอนะครับ!

## 6. 🏁 บทสรุป (To be continued...)
คำสั่ง `v-on` (หรือ `@`) และผองเพื่อน Modifiers คือเครื่องมือที่ทรงพลังมากที่ช่วยให้เราแยก "ตรรกะของข้อมูล (Data Logic)" ออกจาก "รายละเอียดของ DOM (DOM Event details)" ได้อย่างเด็ดขาด เราไม่ต้องมานั่งเขียนโค้ดกั้นฟอร์มรีเฟรชใน Script อีกต่อไป ปล่อยให้ Template ของ Vue รับจบไปเลยครับ โค้ดเราจะสะอาดขึ้นเป็นกอง!

ตอนนี้เราจัดการกับทั้งการแสดงผล (v-bind) และรับการกระทำจากผู้ใช้ (v-on) ได้แล้ว ในตอนถัดไปเราจะขยับสเกลขึ้นไปอีกขั้น กับการทำ **Component Communication** หรือการส่งข้อมูลคุยกันระหว่าง Component แม่และลูก เตรียมตัวรับมือกับ `Props` และ `Emits` กันให้ดีครับ!

---

**ต้องการที่ปรึกษาและพัฒนาระบบ Web Application หรือ Frontend Architecture ให้กับธุรกิจของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและพัฒนาซอฟต์แวร์แบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p