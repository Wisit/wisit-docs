---
title: "เพิ่มสปีดเว็บด้วย Performance Optimization: Lazy Loading และ Suspense"
weight: 350
date: 2026-03-11
author: วิสิทธิ์ แผ้วกระโทก
tags: ["Vue3", "Performance", "Lazy Loading", "Suspense", "Frontend Architecture"]
---
{{< figure src="vue-performance-lazy-load-suspense-cover.jpg" alt="Aesthetic study notes about Web Performance Optimization Lazy Loading and Suspense on an iPad screen" >}}

## 1. 🎯 ตอนที่ 35: สลัดความอืดอาด รีดรีดสปีดเว็บให้พุ่งปรี๊ดด้วย Lazy Loading และ Suspense

## 2. 📖 เปิดฉาก (The Hook)
น้องๆ ลองจินตนาการว่าเรากำลังจะจัดกระเป๋าไปเที่ยวเชียงใหม่ 3 วัน... เราจำเป็นต้องเอาตู้เสื้อผ้าทั้งตู้ ทีวี และตู้เย็น ยัดใส่กระเป๋าเดินทางลากไปพร้อมกันตั้งแต่ก้าวแรกที่ออกจากบ้านไหมครับ? คงไม่ใช่มั้ยครับ เราก็แค่พกสิ่งจำเป็นไป ส่วนอะไรที่อยากได้เพิ่ม ค่อยไปหาซื้อเอาดาบหน้า 

การทำ Single-Page Application (SPA) ก็เหมือนกันครับ! โดยค่าเริ่มต้น (Default) เครื่องมืออย่าง Webpack หรือ Vite จะทำการมัดรวมโค้ด (Bundle) ของ Component ทั้งหมดร้อยกว่าหน้าในโปรเจกต์ของเรา ให้กลายเป็นไฟล์ JavaScript ก้อนยักษ์ก้อนเดียว (Main Bundle) ผลก็คือ... เมื่อ User พิมพ์ URL เข้าเว็บเราครั้งแรก Browser จะต้องดาวน์โหลดไฟล์ยักษ์นี้ให้เสร็จก่อน หน้าเว็บถึงจะแสดงผล (หน้าจอจะขาวโพลนไปหลายวินาที) ซึ่งทำให้ User หงุดหงิดและหนีออกจากเว็บเราไปในที่สุด

ในฐานะ Senior Developer วันนี้พี่จะพามาแก้ปัญหานี้ด้วยเทคนิคการทำ **Code Splitting** หั่นโค้ดก้อนยักษ์ให้เป็นชิ้นเล็กๆ โหลดเฉพาะหน้าเว็บที่ User กำลังจะดู (Lazy Loading) และใช้อาวุธใหม่ของ Vue 3 อย่าง **`<Suspense>`** มาเป็นวาทยกรคอยคุมจังหวะการโหลดให้เนียนตาที่สุด จิบกาแฟให้พร้อม แล้วมาเพิ่มสปีดเว็บกันครับ!

## 3. 🧠 แก่นวิชา (Core Concepts)

### 📦 1. Code Splitting & Route Lazy Loading (โหลดเมื่อจำเป็น)
แทนที่เราจะ `import` Component ทุกหน้ามารอไว้ (Eager Loading) เราจะใช้ฟังก์ชัน `import()` แบบ Dynamic แทนครับ ซึ่งตัว Bundler (Vite/Webpack) จะฉลาดพอที่จะหั่น Component นั้นพร้อมกับไลบรารีที่มันใช้ ออกไปเป็นไฟล์ `.js` ก้อนเล็กๆ (Chunk) แยกต่างหาก และจะโหลดไฟล์นี้ก็ต่อเมื่อ User คลิกเปลี่ยนหน้าไปที่ Route นั้นจริงๆ 

### 🧩 2. Async Components (หั่น Component ย่อยภายในหน้า)
ไม่ใช่แค่หน้าเว็บ (Route) นะครับที่เราหั่นได้! บางครั้งในหน้าเว็บหน้าเดียว เราอาจจะมี Component ที่หนักมากๆ ซ่อนอยู่ เช่น "กราฟ Dashboard แสนซับซ้อน" หรือ "หน้าต่าง Modal ที่มี Text Editor ตัวใหญ่ๆ" เราสามารถใช้ `defineAsyncComponent` เพื่อบอกให้ Vue โหลด Component เหล่านี้ก็ต่อเมื่อมันต้องถูกวาดลงจอ (Render) เท่านั้น

### ⏳ 3. `<Suspense>` (ผู้กำกับการรอคอย)
เมื่อเราหั่น Component ออกไปโหลดทีหลัง สิ่งที่หลีกเลี่ยงไม่ได้คือ "ช่วงเวลาแห่งการรอคอย (Loading Time)" ในอดีตเราต้องมานั่งสร้างตัวแปร `isLoading` เปิดๆ ปิดๆ เองทุก Component แต่ใน Vue 3 เรามี Component พิเศษที่ชื่อว่า `<Suspense>` มันจะทำหน้าที่ครอบ Component ลูกๆ เอาไว้ ถ้าลูกคนไหนกำลัง "โหลด (Async)" อยู่ มันจะแสดง `#fallback` (เช่น รูป Spinner) ออกมาแทนอัตโนมัติ จนกว่าลูกจะพร้อม ถึงจะสลับกลับไปแสดง `#default`

{{< figure src="vue-performance-lazy-load-suspense-diagram.jpg" alt="Diagram showing Vue.js Performance Optimization with Route Lazy Loading and Suspense" >}}

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code)

เรามาดูการเปลี่ยนแอปธรรมดา ให้กลายเป็นแอปที่โหลดไวปานสายฟ้าแลบกันครับ!

### Step 1: หั่น Route ด้วย Lazy Loading (`src/router/index.js`)
เปลี่ยนการ `import` หน้าต่างๆ ให้เป็นแบบ Dynamic ฟังก์ชันครับ
```javascript
import { createRouter, createWebHistory } from 'vue-router'
// ❌ แบบเก่า (Eager Loading): โหลดรวมเป็นก้อนเดียวตั้งแต่แรก
// import HomeView from '../views/HomeView.vue'
// import DashboardView from '../views/DashboardView.vue'

const routes = [
  {
    path: '/',
    name: 'Home',
    // หน้าแรกยังไง User ก็ต้องเห็น อนุโลมให้ใช้วิธีโหลดปกติได้ครับ
    component: () => import('../views/HomeView.vue') 
  },
  {
    path: '/dashboard',
    name: 'Dashboard',
    // ✅ แบบใหม่ (Lazy Loading): โหลดเมื่อ User คลิกเข้า URL /dashboard เท่านั้น
    // 🌟 Pro-Tip: การใส่ Magic Comment แบบด้านล่างนี้ จะทำให้ไฟล์ที่ถูกหั่นออกมา 
    // มีชื่อสวยๆ ว่า "admin-dashboard.js" แทนที่จะเป็นตัวเลขสุ่มๆ ช่วยให้ Debug ง่ายขึ้น
    component: () => import(
      /* webpackChunkName: "admin-dashboard" */ 
      '../views/DashboardView.vue'
    )
  }
]
// ...
```

### Step 2: หั่น Component ย่อยด้วย `defineAsyncComponent`
สมมติว่าในหน้า Dashboard เรามีกราฟที่ใช้ Library หนักๆ เราจะแยกมันออกไปครับ
```vue
<!-- src/views/DashboardView.vue -->
<script setup>
import { defineAsyncComponent } from 'vue'

// 1. สร้าง Component ที่จะถูกโหลดก็ต่อเมื่อถูกเรียกใช้
const HeavyChart = defineAsyncComponent(() => 
  import('../components/HeavyChart.vue')
)
</script>
```

### Step 3: กางอาณาเขต `<Suspense>` ครอบ Component ไว้!
คราวนี้เราไม่ต้องมานั่งเขียน `v-if="isLoading"` อีกต่อไป เราจะโยนหน้าที่นี้ให้ Suspense ครับ
```vue
<template>
  <div class="dashboard-page">
    <h1>สรุปยอดขายประจำปี 📊</h1>
    
    <!-- 2. ใช้ Suspense ครอบ Component ที่เป็น Async ไว้ -->
    <Suspense>
      <!-- 🌟 Slot #default: สิ่งที่จะแสดงเมื่อ Component โหลดเสร็จแล้ว -->
      <template #default>
        <HeavyChart />
      </template>

      <!-- 🌟 Slot #fallback: สิ่งที่จะแสดงคั่นเวลาระหว่างรอ (Loading State) -->
      <template #fallback>
        <div class="loading-spinner">
          กำลังเตรียมข้อมูลกราฟฟิก... ⏳
        </div>
      </template>
    </Suspense>

  </div>
</template>
```

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips)

**🚨 1. ระวังปัญหา "ไฟกะพริบ (Flickering Loader)"**
ถ้าน้องๆ ใช้ `defineAsyncComponent` แล้วปรากฏว่าเน็ตของ User แรงมาก โหลด Component เสร็จภายใน 0.05 วินาที สิ่งที่ User เห็นคือตัว Spinner จะกระพริบแวบเดียวแล้วหายไป ซึ่งมันดูไม่สวยงามครับ (Janky UI) 
วิธีแก้คือเราสามารถตั้งค่า `delay` ใน `defineAsyncComponent` แบบนี้ได้ครับ:
```javascript
const HeavyChart = defineAsyncComponent({
  loader: () => import('./HeavyChart.vue'),
  delay: 200 // ถ้าโหลดเสร็จก่อน 200ms ไม่ต้องโชว์ Loading เลย ให้ข้ามไปโชว์ของจริงเลย! เนียนสุดๆ
})
```

**🗂️ 2. จัดกลุ่ม Chunk ด้วย Magic Comments**
บางครั้งหน้าเว็บที่เกี่ยวข้องกัน (เช่น `/settings/profile`, `/settings/security`) เวลา User เข้ามาถึงโซน Settings แล้ว เขามักจะกดดูหน้าอื่นๆ ในโซนนี้ด้วย เราสามารถสั่งให้ Webpack มัดรวม Component พวกนี้ไว้ในไฟล์ (Chunk) เดียวกันได้ โดยการตั้งชื่อ `webpackChunkName` ให้เหมือนกันครับ
```javascript
const SettingsProfile = () => import(/* webpackChunkName: "settings-group" */ './Profile.vue')
const SettingsSecurity = () => import(/* webpackChunkName: "settings-group" */ './Security.vue')
```
*ผลลัพธ์คือ: พอ User เข้าหน้า Profile ไฟล์ `settings-group.js` จะถูกโหลดมาทั้งก้อน พอ User กดสลับไปหน้า Security มันก็จะโหลดขึ้นมาแสดงได้ทันทีโดยไม่ต้องรอโหลดไฟล์ใหม่ครับ!*

**⚠️ 3. `<Suspense>` ยังเป็น Experimental Feature**
แม้ว่า `<Suspense>` จะทรงพลังมาก แต่ในเอกสารของ Vue 3 ยังระบุว่ามันเป็น "ฟีเจอร์ทดลอง (Experimental)" อยู่ ดังนั้น API อาจจะมีการเปลี่ยนแปลงได้เล็กน้อยในอนาคตครับ แต่ในระดับใช้งานจริง หลายๆ โปรเจกต์ Enterprise ก็เริ่มนำมาใช้งานร่วมกับ Vue Router (ครอบ RouterView) กันอย่างแพร่หลายแล้วครับ

## 6. 🏁 บทสรุป (To be continued...)
การทำ **Performance Optimization** ผ่าน **Lazy Loading** ถือเป็นไม้ตายที่แยกนักพัฒนาธรรมดาออกจากนักพัฒนาระดับ Senior ครับ การลดขนาด Initial Bundle Size จะช่วยให้คะแนน Web Vitals (เช่น LCP, Time to Interactive) ของเราดีขึ้น พุ่งทะยานติดหน้าแรก Google ได้ง่ายขึ้น และด้วยพระเอกอย่าง **`<Suspense>`** การจัดการ Loading State ของเราก็กลายเป็นเรื่องกล้วยๆ โค้ดสะอาดตาและดูแพงขึ้นเป็นกอง

ตอนนี้หน้าเว็บของเราทั้งโครงสร้างดี ดึงข้อมูลเก่ง และโหลดไวปานสายฟ้าแล้ว ในบทความต่อไป เราจะมาก้าวเข้าสู่ศาสตร์แห่งความสวยงามกับ **Transitions และ Animations** ใน Vue 3 เปลี่ยนเว็บนิ่งๆ ให้ดูลื่นไหลราวกับแอปพลิเคชันบนมือถือระดับพรีเมียม เตรียมตัววาดลวดลายกันได้เลยครับ!

---

**ต้องการที่ปรึกษาและพัฒนาระบบ Web Application หรือ Frontend Architecture ให้กับธุรกิจของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและพัฒนาซอฟต์แวร์แบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p