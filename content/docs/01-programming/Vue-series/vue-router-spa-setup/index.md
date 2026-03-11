---
title: "สร้าง Single Page Application (SPA) ด้วย Vue Router 4"
weight: 240
date: 2026-03-11
author: วิสิทธิ์ แผ้วกระโทก
tags: ["Vue3", "Vue Router", "SPA", "Dynamic Routing", "Frontend"]
---
{{< figure src="vue-router-spa-setup-cover.jpg" alt="Aesthetic study notes about Single Page Application and Vue Router 4 on an iPad screen" >}}

## 1. 🎯 ตอนที่ 24: สร้าง Single-Page Application (SPA) ให้ลื่นไหลปานสายน้ำด้วย Vue Router 4

## 2. 📖 เปิดฉาก (The Hook)
น้องๆ ครับ ในยุคก่อนเวลาเราทำเว็บไซต์ที่มีหลายๆ หน้า (Multi-Page Application) ทุกครั้งที่ผู้ใช้คลิกเปลี่ยนหน้า เบราว์เซอร์จะต้องส่งคำขอไปที่ Server แล้วทำการ "โหลดหน้าเว็บใหม่ทั้งหมด (Full page reload)" หน้าจอจะขาววูบไปแวบหนึ่ง ซึ่งประสบการณ์แบบนี้มันดูเก่าและไม่ทันใจวัยรุ่นเอาซะเลย

ลองจินตนาการถึง "โรงละคร" ดูนะครับ แบบดั้งเดิมคือพอจบฉากทีนึง ก็ต้องปิดม่าน รื้อเวทีทิ้ง แล้วสร้างเวทีใหม่หมด... แต่ในโลกของ **Single-Page Application (SPA)** เวทีของเราจะถูกสร้างขึ้นมาแค่ครั้งเดียว (หน้า `index.html`) จากนั้นพอเปลี่ยนฉาก เราก็แค่เรียก "นักแสดง (Component) ชุดใหม่" ขึ้นมาสลับสับเปลี่ยนบนเวทีเดิม!

และผู้กำกับเวทีที่คอยสั่งการว่า URL นี้ต้องเอานักแสดงคนไหนขึ้นเวที ก็คือพระเอกของเราในวันนี้... **Vue Router 4** นั่นเองครับ! จิบกาแฟให้พร้อม แล้วเรามาเปลี่ยนเว็บธรรมดาให้กลายเป็น SPA ที่เปลี่ยนหน้าไวปานสายฟ้าแลบกันครับ

## 3. 🧠 แก่นวิชา (Core Concepts)

**Vue Router** คือ Official Routing Library ที่ออกแบบมาเพื่อทำงานร่วมกับ Vue.js โดยเฉพาะ ในเวอร์ชัน 4 นี้ถูกเขียนขึ้นมาใหม่เพื่อรองรับ Vue 3 Composition API อย่างสมบูรณ์แบบครับ โดยมีหัวใจหลักอยู่ 3 ส่วน:

### 🗺️ 1. The Route Configuration (แผนที่นำทาง)
เราต้องสร้างอาร์เรย์ (Array) เพื่อจับคู่ (Mapping) ว่า URL path ไหน จะต้องแสดง Component อะไร เช่น ถ้า URL คือ `/about` ให้แสดง Component `AboutView` เป็นต้น

### 🎭 2. `<router-view>` (เวทีการแสดง)
นี่คือ Component พิเศษที่ Vue Router เตรียมไว้ให้ มันทำหน้าที่เป็น "พื้นที่ว่าง (Placeholder)" เมื่อ URL เปลี่ยนแปลง Vue Router จะเอา Component ที่ตรงกับเงื่อนไขมาเสียบลงในช่องว่างนี้โดยอัตโนมัติ

### 🚪 3. `<router-link>` (ประตูมิติ)
ถ้าเราใช้แท็ก `<a>` (Anchor tag) แบบปกติ เบราว์เซอร์จะทำการรีเฟรชหน้าเว็บเสมอ เราจึงต้องเปลี่ยนมาใช้ `<router-link>` แทน ซึ่งเบื้องหลังมันก็คือแท็ก `<a>` นั่นแหละครับ แต่มันถูกสกัดเวทมนตร์เอาไว้ไม่ให้เบราว์เซอร์รีเฟรชหน้าเว็บ (Prevent default) และจัดการเปลี่ยน URL ให้เราแบบเนียนๆ

{{< figure src="vue-router-spa-setup-diagram.jpg" alt="Diagram showing Vue Router architecture mapping URL paths to specific Vue components" >}}

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code)

มาดูขั้นตอนการประกอบร่างตั้งแต่ติดตั้ง ยันสร้างหน้าแสดงรายละเอียดสินค้า (Dynamic Routing) กันเลยครับ!

**Step 1: ติดตั้ง Vue Router** (เปิด Terminal แล้วพิมพ์เลย)
```bash
npm install vue-router@4
```

**Step 2: สร้างไฟล์กำหนดเส้นทาง (`src/router/index.js`)**
เราจะสร้างไฟล์แยกออกมาเพื่อจัดการ Route ทั้งหมดให้เป็นระเบียบครับ
```javascript
import { createRouter, createWebHistory } from 'vue-router'
import HomeView from '../views/HomeView.vue'

const routes = [
  {
    path: '/',
    name: 'Home',
    component: HomeView
  },
  {
    path: '/about',
    name: 'About',
    // 🌟 Pro-Tip: Lazy Loading (โหลดโค้ดเฉพาะตอนคลิกเข้าหน้านี้ ช่วยให้เว็บหน้าแรกเบาขึ้น)
    component: () => import('../views/AboutView.vue')
  },
  {
    // 🌟 Dynamic Route: ใช้เครื่องหมาย colon (:) เพื่อกำหนดตัวแปรรับค่า ID ของสินค้า
    path: '/product/:id',
    name: 'ProductDetail',
    component: () => import('../views/ProductDetail.vue')
  }
]

// สร้าง Instance ของ Router
const router = createRouter({
  // ใช้ Web History API เพื่อให้ URL สวยงาม ไม่มีเครื่องหมาย # (Hash) มากวนใจ
  history: createWebHistory(),
  routes
})

export default router
```

**Step 3: ลงทะเบียน Router ใน `src/main.js`**
```javascript
import { createApp } from 'vue'
import App from './App.vue'
import router from './router' // นำเข้า Router ที่เราสร้างไว้

const app = createApp(App)

app.use(router) // ติดตั้งพลังการนำทางให้กับแอปของเรา
app.mount('#app')
```

**Step 4: วางเวทีใน `src/App.vue`**
```vue
<template>
  <div class="app-layout">
    <!-- แถบนำทาง (Navigation Bar) -->
    <nav class="navbar">
      <!-- ใช้ router-link แทนแท็ก a -->
      <router-link to="/">หน้าแรก</router-link> |
      <router-link to="/about">เกี่ยวกับเรา</router-link> |
      
      <!-- ตัวอย่างการส่งลิงก์แบบระบุพารามิเตอร์ (ไปหาสินค้า ID 123) -->
      <router-link :to="{ name: 'ProductDetail', params: { id: '123' } }">
        ดูสินค้าเด่น
      </router-link>
    </nav>

    <!-- พื้นที่แสดงผล: Component จะถูกนำมาสวมแทนที่ตรงนี้! -->
    <main class="main-content">
      <router-view></router-view>
    </main>
  </div>
</template>
```

**Step 5: สร้างหน้า Dynamic Route (`src/views/ProductDetail.vue`)**
เมื่อเราเข้ามาที่ `/product/123` เราจะดึงเลข `123` ออกมาใช้งานได้อย่างไร? คำตอบคือใช้ Composable ที่ชื่อว่า `useRoute` ครับ!
```vue
<template>
  <div class="product-detail">
    <h2>รายละเอียดสินค้า</h2>
    <!-- ดึงค่าพารามิเตอร์จาก URL มาแสดงผล -->
    <p>กำลังค้นหาข้อมูลของรหัสสินค้า: <strong>{{ productId }}</strong></p>
  </div>
</template>

<script setup>
import { computed } from 'vue'
import { useRoute } from 'vue-router'

// อัญเชิญ useRoute() เพื่อดึงข้อมูลของ "เส้นทางปัจจุบัน"
const route = useRoute()

// ใช้ computed เพื่อให้มันอัปเดตอัตโนมัติ หากผู้ใช้เปลี่ยน URL จาก /product/123 เป็น /product/456
const productId = computed(() => route.params.id)

// ในการทำงานจริง เรามักจะเอา route.params.id ก้อนนี้ ไปยิง API ดึงข้อมูลสินค้าต่อครับ!
</script>
```

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips)

มีเคล็ดลับ 2 อย่างที่ Senior Dev มักจะจัดการให้เรียบร้อยตั้งแต่เริ่มโปรเจกต์ครับ:

**🚨 1. การจัดการหน้า 404 (Not Found)**
ถ้าผู้ใช้พิมพ์ URL มั่วๆ เช่น `/blablabla` ระบบจะหา Component ไม่เจอแล้วหน้าเว็บจะขาวโพลน วิธีแก้คือให้เราเพิ่ม Route ดักจับ (Catch-all) ไว้ที่ **"ล่างสุด"** ของอาร์เรย์ `routes` เสมอครับ:
```javascript
{
  // ใช้ Regular Expression ดักจับทุก URL ที่ไม่ตรงกับด้านบนเลย
  path: '/:pathMatch(.*)*', 
  name: 'NotFound',
  component: () => import('../views/NotFound.vue')
}
```

**🚥 2. ความต่างของ `useRoute` vs `useRouter`**
เวลาใช้ Composition API มือใหม่มักจะสับสน 2 ตัวนี้ พี่ขอสรุปให้จำง่ายๆ ครับ:
*   `useRoute()` (ไม่มี r): เอาไว้ **"อ่าน"** ข้อมูลของ URL ปัจจุบัน (เช่น อ่าน params, อ่าน query)
*   `useRouter()` (มี r): เอาไว้ **"สั่งการ"** เปลี่ยนหน้าผ่านโค้ด JavaScript (Programmatic Navigation) เช่น สั่ง `router.push('/login')` หลังจากกดปุ่ม Submit ฟอร์มเสร็จ

## 6. 🏁 บทสรุป (To be continued...)
ด้วยการประสานพลังของ **Vue Router** ตอนนี้แอปพลิเคชันของเราก็สามารถทำงานแบบ Single-Page Application ได้อย่างสมบูรณ์แบบแล้วครับ การสลับหน้าไปมาจะลื่นไหล ไม่มีอาการกระตุกจากการโหลดหน้าเว็บใหม่ และผู้ใช้ก็ยังสามารถก็อปปี้ URL (เช่น หน้าสินค้ารหัส 123) ส่งไปให้เพื่อนดูได้เหมือนเว็บปกติเลย

แต่คำถามถัดมาคือ... ถ้าเรามีหน้า "Dashboard สำหรับ Admin" ที่บังคับว่า **"ต้องล็อกอินก่อนเท่านั้นถึงจะเข้าได้"** ล่ะ? เราจะเอา Logic ตรงนี้ไปดักไว้ที่ไหนดี?

ในบทต่อไป เราจะมากางคัมภีร์วิชาขั้นสูงที่เรียกว่า **Navigation Guards (ยามเฝ้าประตูเส้นทาง)** เพื่อป้องกันไม่ให้คนนอกทะลุทะลวงเข้ามาในพื้นที่หวงห้ามของเรากันครับ เตรียมตัวให้พร้อม!

---

**ต้องการที่ปรึกษาและพัฒนาระบบ Web Application หรือ Frontend Architecture ให้กับธุรกิจของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและพัฒนาซอฟต์แวร์แบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p