---
title: "การจัดการ Error และ Loading State: มอบประสบการณ์ที่ดีที่สุดให้ User"
weight: 310
date: 2026-03-11
author: วิสิทธิ์ แผ้วกระโทก
tags: ["Vue3", "UX", "Error Handling", "Loading State", "ErrorBoundary"]
---
{{< figure src="vue-error-loading-state-cover.jpg" alt="Aesthetic study notes about Handling Error and Loading State in Vue.js on an iPad screen" >}}

## 1. 🎯 ตอนที่ 31: ศิลปะแห่งการรอคอยและรับมือกับความผิดหวัง (Error & Loading States)

## 2. 📖 เปิดฉาก (The Hook)
น้องๆ ครับ ลองจินตนาการว่าเราเดินเข้าไปสั่งอาหารในร้านอาหารหรูแห่งหนึ่ง พนักงานรับออเดอร์เสร็จแล้วก็เดินหายไป... ผ่านไป 15 นาที โต๊ะเราก็ยังว่างเปล่า ไม่มีน้ำ ไม่มีพนักงานมาบอกว่าเกิดอะไรขึ้น เราคงรู้สึกหงุดหงิดและอยากลุกออกจากร้านใช่ไหมครับ? 

แต่ถ้าเปลี่ยนใหม่ พนักงานเดินมาเสิร์ฟน้ำเก๊กฮวยเย็นๆ พร้อมบอกว่า *"รอสักครู่นะคะ ตอนนี้เชฟกำลังย่างสเต็กให้ คิวค่อนข้างเยอะค่ะ"* (Loading State) หรือเดินมาบอกขอโทษว่า *"ขออภัยจริงๆ ค่ะ วัตถุดิบเมนูนี้หมด รับเป็นเมนูอื่นแทนไหมคะ?"* (Error State) ความรู้สึกของเราจะต่างกันลิบลับเลย!

การเขียน Web Application ก็เหมือนการเปิดร้านอาหารครับ! การยิง API ไปหา Server ต้องใช้เวลา (Network Latency) และบางครั้ง Server ก็อาจจะล่ม (Server Crash) ถ้าน้องๆ ปล่อยให้หน้าเว็บ "ค้าง" หรือ "ขาวโพลน" โดยไม่บอกอะไร User เลย นั่นคือฝันร้ายของ UX (User Experience) ครับ วันนี้พี่จะพามาดูเทคนิคการทำ Loading Spinners และการสร้าง "กำแพงกันบั๊ก (Error Boundaries)" เพื่อมอบประสบการณ์ระดับพรีเมียมให้กับผู้ใช้งานกันครับ! จิบกาแฟแล้วมาลุยกันเลย

## 3. 🧠 แก่นวิชา (Core Concepts)

การจัดการสถานะ (State) ของหน้าจอเวลาเชื่อมต่อกับ API โดยพื้นฐานแล้วเราจะแบ่งออกเป็น 3 สถานะหลักๆ ครับ:

### ⏳ 1. Loading State (กำลังโหลด)
ช่วงเวลาที่ Request ถูกส่งออกไปแต่ยังไม่ได้คำตอบกลับมา เราจะใช้เทคนิค **Conditional Rendering** (`v-if`, `v-else`) คู่กับตัวแปร State (เช่น `isLoading`) เพื่อสลับหน้าจอไปแสดงตัวหมุนๆ (Spinner) หรือ Skeleton Loading

### ❌ 2. Error State (เกิดข้อผิดพลาด)
เมื่อ API ตอบกลับมาว่าพัง (เช่น 404 Not Found หรือ 500 Server Error) เราต้องจับข้อผิดพลาดนั้นด้วย `try...catch` แล้วเก็บข้อความเตือนไว้ในตัวแปร State (เช่น `error`) เพื่อนำไปแสดงผลให้ User ทราบ พร้อมปุ่ม "ลองใหม่อีกครั้ง (Retry)"

### 🛡️ 3. Error Boundaries (กำแพงกักกันความเสียหาย)
ในโปรเจกต์สเกลใหญ่ระดับ Enterprise บางครั้ง Component ลูกๆ ที่อยู่ลึกๆ อาจจะพังโดยที่เราไม่คาดคิด (เช่น ตัวแปรพัง, เรนเดอร์ไม่ได้) ถ้าเราไม่ดักไว้ หน้าเว็บจะขาว (Crash) ไปทั้งหน้า! Vue 3 มี Hook ทรงพลังที่ชื่อว่า **`onErrorCaptured`** ที่ทำหน้าที่เป็น "ตาข่ายนิรภัย" คอยดักจับ Error จาก Component ลูกๆ ไม่ให้ลามไปทำพังทั้งแอปพลิเคชัน

{{< figure src="vue-error-loading-state-diagram.jpg" alt="Diagram showing Vue.js Error Boundary architecture and Conditional Rendering flow" >}}

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code)

เรามาดู 2 กระบวนท่าหลักที่เราจะได้ใช้ทำมาหากินกันบ่อยๆ ครับ

### ท่าที่ 1: การจัดการสถานะแบบพื้นฐาน (Basic Loading & Error)
ท่านี้เราจะใช้ตัวแปร `ref` 3 ตัว เพื่อคุมสถานะหน้าจอครับ

```vue
<template>
  <div class="user-profile-card">
    
    <!-- 1. ถ้ากำลังโหลด ให้โชว์ Spinner -->
    <div v-if="isLoading" class="loading-state">
      <div class="spinner"></div>
      <p>กำลังค้นหาข้อมูลผู้ใช้งาน... ⏳</p>
    </div>

    <!-- 2. ถ้าโหลดพัง ให้โชว์ Error พร้อมปุ่มกดลองใหม่ -->
    <div v-else-if="error" class="error-state">
      <h3>🚨 อุ๊ปส์! เกิดข้อผิดพลาด</h3>
      <p>{{ error }}</p>
      <button @click="fetchUserProfile">ลองใหม่อีกครั้ง</button>
    </div>

    <!-- 3. ถ้าสำเร็จและมีข้อมูล ให้โชว์ของจริง -->
    <div v-else class="success-state">
      <h2>{{ user.name }}</h2>
      <p>Email: {{ user.email }}</p>
    </div>

  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const user = ref(null)
const isLoading = ref(false)
const error = ref(null)

const fetchUserProfile = async () => {
  // เริ่มต้นด้วยการเปิด Loading และเคลียร์ Error เก่าทิ้ง
  isLoading.value = true
  error.value = null

  try {
    const response = await fetch('https://jsonplaceholder.typicode.com/users/1')
    if (!response.ok) throw new Error('เซิร์ฟเวอร์ไม่ตอบสนอง กรุณาลองใหม่')
    
    user.value = await response.json()
  } catch (err) {
    // ถ้าพัง โยนข้อความเข้าไปเก็บในตัวแปร error
    error.value = err.message 
  } finally {
    // ไม่ว่าจะพังหรือสำเร็จ ก็ต้องปิด Loading เสมอ!
    isLoading.value = false 
  }
}

// โหลดข้อมูลทันทีที่เปิดหน้าเว็บ
onMounted(() => {
  fetchUserProfile()
})
</script>

<style scoped>
.spinner { /* ใส่ CSS ทำอนิเมชันหมุนๆ ตรงนี้ */ }
.error-state { color: red; background: #fee; padding: 1rem; border-radius: 8px; }
</style>
```

---

### ท่าที่ 2: กางอาณาเขตกำแพงกันบั๊ก (Error Boundary Component)
เราจะสร้าง Component ห่อหุ้ม (Wrapper) เอาไว้ครอบ Component อื่นๆ ที่เสี่ยงจะพังครับ

**ไฟล์ `src/components/ErrorBoundary.vue`**
```vue
<template>
  <div>
    <!-- ถ้ามี Error เกิดขึ้นใน Component ลูก ให้แสดงหน้าต่างนี้แทน -->
    <div v-if="error" class="error-boundary-alert">
      <h3>⚠️ ระบบส่วนนี้ขัดข้องชั่วคราว</h3>
      <p>{{ error.message }}</p>
      <button @click="resetError">โหลดส่วนนี้ใหม่</button>
    </div>

    <!-- ถ้าไม่มี Error ก็ปล่อยให้ Component ลูก (Slot) แสดงผลไปตามปกติ -->
    <slot v-else></slot>
  </div>
</template>

<script setup>
import { ref, onErrorCaptured } from 'vue'

const error = ref(null)

// ฟังก์ชันเคลียร์ Error เพื่อให้ User ลองกดโหลดใหม่
const resetError = () => {
  error.value = null
}

// 🪤 ดักจับ Error จาก Component ลูกที่อยู่ใน <slot>
onErrorCaptured((err, instance, info) => {
  console.error('จับบั๊กได้แล้ว! ต้นตอมาจาก:', instance.$.type.name)
  error.value = err
  
  // Return false เพื่อบอก Vue ว่า "ฉันจัดการ Error นี้แล้ว ไม่ต้องส่งฟ้องไปถึงตัว App หลัก (Global)"
  return false 
})
</script>
```

**วิธีนำไปใช้ในไฟล์แม่ (เช่น `Dashboard.vue`):**
```vue
<template>
  <div class="dashboard">
    <h1>ระบบจัดการข้อมูล</h1>
    
    <!-- ถ้า UserProfile พัง หน้า Dashboard ก็ยังอยู่! -->
    <ErrorBoundary>
      <UserProfile />
    </ErrorBoundary>

    <ErrorBoundary>
      <ActivityFeed />
    </ErrorBoundary>
  </div>
</template>
```
*เห็นความเจ๋งไหมครับ! ถ้า `ActivityFeed` มีบั๊กและ Render ไม่ขึ้น มันก็จะโชว์แค่คำว่า "ระบบส่วนนี้ขัดข้องชั่วคราว" ในกรอบของมัน โดยที่ `UserProfile` และส่วนอื่นๆ ของหน้าเว็บยังทำงานต่อไปได้ตามปกติ หน้าเว็บไม่ขาวโพลน!*

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips)

มีเคล็ดวิชา 2 ข้อที่ Senior Dev มักจะจัดการเพื่อรีดเร้น UX ให้ออกมาดีที่สุดครับ:

**✨ 1. ระวังปัญหา "Loading กะพริบตาเดียว" (The Flickering Loader)**
ลองคิดดูนะครับ ถ้าน้องโชคดีมาก อินเทอร์เน็ตแรงสุดๆ API ตอบกลับมาภายในเวลา 0.1 วินาที สิ่งที่ User จะเห็นคือ "Spinner โผล่มาแวบเดียวแล้วหายไป" การกระพริบแบบนี้ทำให้แสบตาและดูไม่แพงครับ! 
*💡 Pro-Tip:* เรามักจะเขียนโค้ดหน่วงเวลา (Delay) เล็กน้อย เช่น "ถ้าโหลดนานเกิน 300ms ค่อยแสดง Spinner แต่ถ้าโหลดเสร็จก่อนหน้านั้น ไม่ต้องโชว์อะไรเลย" เทคนิคนี้จะทำให้เว็บดูตอบสนองได้ลื่นไหลเนียนตาขึ้นมากครับ

**☠️ 2. กฎของการ Unmount (อย่าทำร้าย Component ที่ตายไปแล้ว)**
ในจังหวะที่ยิง API รอผลอยู่ (สมมติว่าใช้เวลา 3 วินาที) User ดันใจร้อน กดปุ่ม "Back" เปลี่ยนหน้าหนีไปซะก่อน แปลว่า Component นั้นถูกทำลาย (Unmounted) ไปแล้ว! แต่พอวินาทีที่ 3 API ส่งข้อมูลกลับมา โค้ดในคำสั่ง `try` หรือ `finally` พยายามจะเอาค่าไปยัดใส่ตัวแปร `isLoading.value = false` ของ Component ที่ไม่มีอยู่จริงแล้ว! (ซึ่งมักจะทำให้เกิด Error บน Console)
*💡 Pro-Tip:* ในระบบใหญ่ๆ เราควรเช็กเสมอว่า "Component ยังอยู่ไหม (isMounted)?" หรือใช้เทคนิค "Cancel Token" ของ Axios/Fetch เพื่อยกเลิก Request นั้นทิ้งไปเลยเมื่อผู้ใช้เปลี่ยนหน้าเว็บครับ

## 6. 🏁 บทสรุป (To be continued...)
การเขียนโปรแกรมที่ดี ไม่ใช่แค่การทำให้แอปพลิเคชัน "ทำงานได้ตามปกติ (Happy Path)" เท่านั้นครับ แต่มันคือศิลปะของการคาดเดาและรับมือกับ "ความล้มเหลว (Unhappy Path)" ต่างหาก 

การจัดการ **Error และ Loading States** อย่างเป็นระบบ ไม่ว่าจะเป็นการโชว์ Spinner สวยๆ, การบอกข้อผิดพลาดที่อ่านรู้เรื่อง หรือการใช้ **Error Boundaries** กั้นความเสียหาย สิ่งเหล่านี้แหละครับคือเส้นแบ่งที่ชัดเจนระหว่าง "มือใหม่" และ "Senior Developer" ที่ใส่ใจใน User Experience อย่างแท้จริง

ในบทความถัดไป เราจะขยับสเกลขึ้นไปอีกขั้น กับการจัดการ Application ที่มีความซับซ้อนสูงด้วยเทคนิคการทำ **Code Splitting และ Lazy Loading** เพื่อให้แอปพลิเคชันของเราโหลดไวขึ้นเป็นจรวด! เตรียมตัวให้พร้อมครับ!

---

**ต้องการที่ปรึกษาและพัฒนาระบบ Web Application หรือ Frontend Architecture ให้กับธุรกิจของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและพัฒนาซอฟต์แวร์แบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p