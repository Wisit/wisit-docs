---
title: "Unit Testing กับ Vue: มั่นใจทุกครั้งที่อัปเดตด้วย Vitest และ Vue Test Utils"
weight: 370
date: 2026-03-11
author: วิสิทธิ์ แผ้วกระโทก
tags: ["Vue3", "Unit Test", "Vitest", "Vue Test Utils", "Best Practices", "Testing"]
---
{{< figure src="vue-unit-testing-vitest-cover.jpg" alt="Aesthetic study notes about Unit Testing Vue components on an iPad screen" >}}

## 1. 🎯 ตอนที่ 37: กางตาข่ายนิรภัยให้โค้ดด้วย Unit Testing

## 2. 📖 เปิดฉาก (The Hook)
น้องๆ เคยเจอ "ฝันร้ายบ่ายวันศุกร์" แบบนี้ไหมครับ? เราเพิ่งแก้โค้ดปุ่ม `Button` ไปนิดเดียวเพื่อให้สีมันสวยขึ้น แล้วก็กด Deploy ขึ้น Production ด้วยความมั่นใจ... แต่ปรากฏว่าหน้าเว็บพัง ลูกค้ากดสั่งซื้อของไม่ได้! สาเหตุเพราะโค้ดที่เราแก้ ดันไปกระทบกับฟังก์ชันคำนวณราคาซะงั้น 

การนั่งกดเทสต์เว็บด้วยมือ (Manual Testing) ซ้ำๆ ทุกครั้งที่แก้โค้ด นอกจากจะน่าเบื่อและกินเวลาแล้ว ยังเสี่ยงต่อการหลุดรอดของสายตาเรามากๆ ครับ ในโลกของวิศวกรรมซอฟต์แวร์ระดับ Senior เราจึงมี "ตาข่ายนิรภัย" ที่เรียกว่า **Automated Testing** โดยเฉพาะ **Unit Testing** ซึ่งเปรียบเสมือนการจับชิ้นส่วน (Component) แต่ละชิ้นมาเข้าห้องแล็บ แล้วทดสอบแบบแยกส่วน (Isolation) เพื่อดูว่ามันทำงานได้ถูกต้องตามสเปกหรือไม่

วันนี้พี่จะพามาจิบกาแฟ และเรียนรู้วิธีการเขียน Unit Test สำหรับ Vue Component ด้วยเครื่องมือระดับพระกาฬอย่าง **Vitest** และ **Vue Test Utils** กันครับ รับรองว่าเขียนโค้ดเสร็จแล้วจะนอนหลับสบาย ไม่ต้องกลัวบั๊กตามหลอกหลอน!

## 3. 🧠 แก่นวิชา (Core Concepts)

ก่อนจะไปดูโค้ด เราต้องรู้จักอาวุธและกระบวนท่ากันก่อนครับ:

### 🛠️ อาวุธคู่กาย (The Tools)
1. **Vitest:** เป็น Test Framework และ Test Runner ตัวใหม่ที่เกิดมาเพื่อใช้งานร่วมกับ Vite โดยเฉพาะ มันทำงานได้เร็วกว่า Jest มาก และมี API ที่เข้ากันได้กับ Jest แทบจะ 100% หน้าที่ของมันคือการสั่งรันโค้ดทดสอบและสรุปผลว่า "ผ่าน (Pass)" หรือ "ตก (Fail)"
2. **Vue Test Utils:** เป็นไลบรารีอย่างเป็นทางการจากทีม Vue มีหน้าที่สร้างสภาพแวดล้อมจำลอง (Simulated DOM) เพื่อ "เมานต์ (Mount)" Component ขึ้นมาในหน่วยความจำ ทำให้เราสามารถเข้าไปคลิก พิมพ์ หรือดึงค่า HTML มาตรวจสอบได้

### 🥋 กระบวนท่า AAA (Arrange, Act, Assert)
การเขียนเทสต์เคสที่ดีระดับมือโปร จะยึดหลักการ 3 ขั้นตอนเสมอ (AAA Protocol):
*   **Arrange (เตรียมการจัดฉาก):** เตรียม Component, ข้อมูล (Props), หรือ Mock ฟังก์ชันต่างๆ ให้พร้อมก่อนเริ่มทดสอบ
*   **Act (ลงมือทำ):** จำลองพฤติกรรมของผู้ใช้ เช่น การคลิกปุ่ม (Click) หรือการกรอกฟอร์ม
*   **Assert (ตรวจสอบ):** ตรวจสอบว่าผลลัพธ์ที่ได้ (เช่น ข้อความบนจอเปลี่ยนไป หรือ Event ถูกส่งออกไป) ตรงกับสิ่งที่เราคาดหวังไว้หรือไม่

{{< figure src="vue-unit-testing-vitest-diagram.jpg" alt="Diagram showing the Unit Testing flow using the AAA pattern with Vitest and Vue Test Utils" >}}

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code)

เรามาดูการเขียน Unit Test ผ่านตัวอย่าง Component ง่ายๆ อย่าง `LikeButton.vue` กันครับ

**1. ชิ้นส่วนที่เราจะทดสอบ (`src/components/LikeButton.vue`)**
```vue
<template>
  <div class="like-container">
    <!-- ใส่ data-testid เพื่อให้เทสต์หา Element เจอได้ง่ายขึ้น -->
    <h3 data-testid="title">{{ title }}</h3>
    <button data-testid="like-btn" @click="handleLike">
      Likes: {{ count }}
    </button>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const props = defineProps({
  title: { type: String, required: true }
})

const emit = defineEmits(['liked'])
const count = ref(0)

const handleLike = () => {
  count.value++
  emit('liked', count.value)
}
</script>
```

**2. การเขียนชุดทดสอบ (`src/components/__tests__/LikeButton.spec.js`)**
เราจะใช้ Vitest และ Vue Test Utils ในการเขียนทดสอบทีละสเต็ปตามหลัก AAA ครับ:

```javascript
import { describe, it, expect } from 'vitest'
import { mount } from '@vue/test-utils'
import LikeButton from '../LikeButton.vue'

// 1. ใช้ describe เพื่อจัดกลุ่มชุดการทดสอบ
describe('LikeButton.vue', () => {

  // 🧪 Test Case 1: ตรวจสอบการรับ Props (Input -> DOM Output)
  it('renders props.title correctly', () => {
    // 💥 Arrange: สร้าง Component และส่ง Props เข้าไป
    const wrapper = mount(LikeButton, {
      props: { title: 'บทความสอน Vue' }
    })

    // 💥 Assert: ค้นหา Element และตรวจสอบข้อความที่แสดงผล
    const titleElement = wrapper.find('[data-testid="title"]')
    expect(titleElement.text()).toBe('บทความสอน Vue') 
  })

  // 🧪 Test Case 2: ตรวจสอบพฤติกรรมการคลิกปุ่ม (User Interaction)
  it('increments count and emits event when clicked', async () => {
    // 💥 Arrange: เตรียม Component ให้พร้อม
    const wrapper = mount(LikeButton, {
      props: { title: 'ทดสอบปุ่ม Like' }
    })
    const button = wrapper.find('[data-testid="like-btn"]')

    // 💥 Act: จำลองการคลิกปุ่ม (ต้องใช้ await เพราะการขยับ DOM ใน Vue ใช้เวลาทำงาน)
    await button.trigger('click')

    // 💥 Assert 1: ตัวเลขบนปุ่มต้องเปลี่ยนเป็น 1
    expect(button.text()).toContain('Likes: 1')

    // 💥 Assert 2: มีการยิง Event 'liked' ออกมา (Emitted)
    expect(wrapper.emitted()).toHaveProperty('liked')
    
    // 💥 Assert 3: Payload ที่ส่งมาพร้อม Event ต้องเป็นค่า 1
    // .emitted('liked') คืนค่าเป็น Array ของเหตุการณ์ที่เกิดขึ้น เช่น [ ]
    expect(wrapper.emitted('liked')).toEqual()
  })
})
```

เมื่อน้องๆ รันคำสั่ง `npm run test:unit` หรือ `vitest` ตัวโปรแกรมจะประมวลผลและขึ้นเครื่องหมาย ✅ สีเขียว บ่งบอกว่า Component ของเราทำงานสมบูรณ์แล้วครับ!

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Testing Best Practices)

การเขียนเทสต์ให้ "รันผ่าน" ไม่ยากเท่าการเขียนเทสต์ให้ "แข็งแกร่ง (Resilient)" ครับ นี่คือเทคนิคระดับ Senior ที่ช่วยให้เทสต์ไม่พังง่ายๆ เวลาเรา Refactor โค้ด:

**🎯 1. ทดสอบพฤติกรรม (Behavior) อย่าทดสอบไส้ใน (Implementation Details)**
กฎเหล็กเลยครับ! เราควรทดสอบ "Input" (เช่น Props, การคลิกของผู้ใช้) และ "Output" (เช่น สิ่งที่เรนเดอร์ออกหน้าจอ, Event ที่ถูกยิงออกไป)
**❌ สิ่งที่ไม่ควรทำ:** อย่าพยายามไปดึงค่าตัวแปร (Local state) ใน `<script>` มาทดสอบตรงๆ เพราะถ้าวันหนึ่งเราเปลี่ยนชื่อตัวแปรหน้าบ้าน เทสต์จะพังทันที ทั้งๆ ที่หน้าเว็บยังคลิกทำงานได้ปกติ!

**🚨 2. `mount` vs `shallowMount` เลือกใช้อะไรดี?**
ใน Vue Test Utils เรามีคำสั่งจำลอง Component 2 แบบ:
*   **`mount` (เรนเดอร์จัดเต็ม):** วาด Component ที่ทดสอบ พร้อมกับวาด Component ลูกหลาน (Child components) ทั้งหมดลงไปด้วย เหมาะสำหรับ Component เล็กๆ หรือการทำ Integration Test
*   **`shallowMount` (เรนเดอร์แค่เปลือก):** วาดแค่ Component ปัจจุบันเพียงตัวเดียว หากมี Component ลูก มันจะถูกจำลองเป็นกล่องเปล่าๆ (Stubbed) ทิ้งไป พี่แนะนำให้ใช้ท่านี้เป็นหลักสำหรับ Unit Test ครับ เพราะมันทำให้เทสต์วิ่งไวปานจรวด และเป็นการตีกรอบให้เราโฟกัสเฉพาะลอจิกของไฟล์นั้นจริงๆ

**🔍 3. ใช้ `data-testid` เพื่อผูกจุดทดสอบ**
สังเกตไหมครับว่าพี่ไม่ใช้การค้นหาด้วยชื่อ Class (เช่น `wrapper.find('.btn-primary')`) เพราะชื่อ Class มักจะถูกทีมดีไซเนอร์เปลี่ยนได้เสมอ หากเปลี่ยนปุ๊บ เทสต์เราพังปั๊บ (Brittle Tests) การใช้ `data-testid="like-btn"` จะเป็นการประกาศให้รู้กันในทีมว่า "นี่คือท่อต่อสำหรับรันเทสต์ ห้ามใครมาลบหรือแก้ชื่อเด็ดขาด!" ทำให้เทสต์ของเราทนทานต่อการเปลี่ยนแปลงดีไซน์ครับ

## 6. 🏁 บทสรุป (To be continued...)
การเขียน **Unit Test** อาจจะดูเหมือนเป็นการเพิ่มงาน (Overhead) ในช่วงแรกของการตั้งไข่โปรเจกต์ แต่ในระยะยาว (Long-term) มันคือ "กรมธรรม์ประกันภัย" ที่คุ้มค่าที่สุดครับ! มันทำให้น้องกล้าที่จะรื้อโค้ดเก่าๆ กล้าที่จะเพิ่มฟีเจอร์ใหม่ๆ (Refactor) โดยไม่ต้องกลัวว่าของเดิมจะพัง และยังช่วยลดภาระของทีม QA ลงไปได้มหาศาล

เมื่อพื้นฐาน Unit Test ของเราแข็งแกร่งแล้ว เราก็พร้อมที่จะขยับขึ้นไปทดสอบการทำงานของระบบแบบภาพรวมเสมือนมีผู้ใช้มากดจริงๆ หรือที่เรียกว่า **End-to-End (E2E) Testing** ด้วยเครื่องมืออย่าง Cypress กันในบทความต่อไป เตรียมตัวก้าวขึ้นสู่ระดับโปรได้เลยครับ!

---

**ต้องการที่ปรึกษาและพัฒนาระบบ Web Application หรือ Frontend Architecture ให้กับธุรกิจของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและพัฒนาซอฟต์แวร์แบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p