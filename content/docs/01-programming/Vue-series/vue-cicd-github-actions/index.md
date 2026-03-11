---
title: "CI/CD สำหรับ Vue.js: ส่ง Code ขึ้น Production แบบอัตโนมัติด้วย GitHub Actions"
weight: 410
date: 2026-03-11
author: วิสิทธิ์ แผ้วกระโทก
tags: ["Vue3", "CI/CD", "GitHub Actions", "Netlify", "Vercel", "DevOps"]
---
{{< figure src="vue-cicd-github-actions-cover.jpg" alt="Aesthetic study notes about CI/CD pipeline and GitHub Actions for Vue.js on an iPad screen" >}}

## 1. 🎯 ตอนที่ 41: ปลดแอกภาระคนงานเหมือง ด้วยระบบ Deploy อัตโนมัติ (CI/CD)

## 2. 📖 เปิดฉาก (The Hook)
น้องๆ เคยเจอวงจรชีวิตแบบนี้ไหมครับ? ตี 2 คืนวันศุกร์ นั่งแก้โค้ดเสร็จปุ๊บ สั่งรัน `npm run build` รอให้มันสร้างโฟลเดอร์ `dist` จากนั้นก็ต้องเปิดหน้าเว็บ Hosting (เช่น Netlify หรือ FTP แบบโบราณ) แล้วลากไฟล์ไปวาง (Drag and Drop) เพื่ออัปเดตหน้าเว็บ 

พอตื่นเช้ามา ปรากฏว่าลูกค้าบอก *"น้อง พิมพ์คำว่า 'สวัสดี' ผิดน่ะ แก้ให้หน่อย"*... เราก็ต้องมานั่งแก้โค้ด บิลด์ใหม่ ลากไปวางใหม่ วนลูปไปเรื่อยๆ จนกว่าจะหมดแรง! การทำงานแบบ Manual (Manual Deployment) นอกจากจะเสียเวลาแล้ว ยังเสี่ยงต่อความผิดพลาดของมนุษย์ (Human Error) สูงมากครับ

ในฐานะ Senior Developer เราจะไม่ทนทำงานซ้ำซากครับ! วันนี้พี่จะพามาจิบกาแฟทำความรู้จักกับสถาปัตยกรรมที่เรียกว่า **CI/CD (Continuous Integration / Continuous Deployment)** และใช้อาวุธคู่กายอย่าง **GitHub Actions** เพื่อสร้าง "หุ่นยนต์พนักงาน" ที่จะคอยรับโค้ดของเราไป Build และ Deploy ขึ้นสู่ Cloud Platform อย่าง Netlify หรือ Vercel แบบอัตโนมัติทุกครั้งที่เรากด `git push`!

## 3. 🧠 แก่นวิชา (Core Concepts)

ก่อนจะไปเขียนสคริปต์สั่งหุ่นยนต์ เรามาทำความเข้าใจแนวคิดของ Automation Build & Deploy กันก่อนครับ:

### ⚙️ 1. CI/CD คืออะไร?
*   **CI (Continuous Integration):** คือการนำโค้ดที่นักพัฒนาเขียน ไปรวมไว้ใน Repository ส่วนกลาง (เช่น GitHub) บ่อยๆ เมื่อมีการ Push โค้ดเข้าไป ระบบจะทำหน้าที่ "ตรวจสอบ" เบื้องต้น เช่น รันคำสั่ง Lint หรือ Automated Tests เพื่อให้มั่นใจว่าโค้ดใหม่จะไม่ไปทำลายระบบเก่า
*   **CD (Continuous Delivery/Deployment):** เมื่อโค้ดผ่านการทดสอบ (CI) แล้ว CD จะทำหน้าที่รับช่วงต่อ นำโค้ดนั้นไป Build (เช่น `npm run build`) และนำไปติดตั้ง (Deploy) บน Production Server อัตโนมัติโดยที่มนุษย์ไม่ต้องทำอะไรเลย

### 🏭 2. The Deployment Pipeline (สายพานการผลิต)
การทำงานของ CI/CD จะถูกแบ่งเป็นช่วงๆ เหมือนสายพานในโรงงานครับ ประกอบด้วย:
1.  **Source Stage:** ตรวจจับเมื่อมีการ `git push`
2.  **Build Stage:** ติดตั้ง Dependencies (`npm install`) และมัดรวมไฟล์ (`npm run build`)
3.  **Test Stage:** (ถ้ามี) รัน Unit Test หรือ E2E Test
4.  **Deploy Stage:** นำผลลัพธ์ไปส่งที่ Server ปลายทาง

### 🤖 3. GitHub Actions (วาทยกรผู้คุมสายพาน)
เป็นแพลตฟอร์ม CI/CD ที่ฝังมากับ GitHub เลยครับ เราสามารถสั่งงานมันได้ผ่านการเขียนไฟล์ตั้งค่าแบบ **YAML** (`.yml`) หากมีคนทำกิจกรรม (Events) ตามที่เรากำหนด (เช่น Push โค้ดเข้าสาขา `main`) หุ่นยนต์ของ GitHub ก็จะไปเปิดเครื่อง Server จำลอง (Runners เช่น Ubuntu) แล้วทำตามขั้นตอน (Steps) ที่เราเขียนไว้ทีละบรรทัด

{{< figure src="vue-cicd-github-actions-diagram.jpg" alt="Diagram showing the CI/CD deployment pipeline for a Vue.js application with GitHub Actions" >}}

## 4. 💻 ร่ายมนต์โค้ด (Show me the Code)

เรามาลงมือสร้าง Workflow ในการ Deploy โปรเจกต์ Vue.js ของเราขึ้นสู่ **Netlify** ผ่าน GitHub Actions กันครับ (วิธีนี้ใช้สำหรับ Netlify แบบผ่าน Action)

### Step 1: เตรียม Secrets (กุญแจความลับ)
เราไม่ควรเอา Token หรือรหัสผ่านไปแปะไว้ในโค้ดตรงๆ ครับ เราต้องไปสร้างตัวแปรลับ (Secrets) ใน GitHub 
1. ไปที่บัญชี Netlify ของน้องๆ สร้าง **Netlify Auth Token** (อยู่ใน User Settings > Applications) และหา **Site ID** ของโปรเจกต์
2. กลับมาที่ GitHub Repository ของเรา ไปที่ **Settings > Secrets and variables > Actions**
3. เพิ่ม Secret 2 ตัวชื่อ: `NETLIFY_AUTH_TOKEN` และ `NETLIFY_SITE_ID` แล้วใส่ค่าที่เราได้มาลงไป

### Step 2: สร้างไฟล์ GitHub Actions Workflow
ในโปรเจกต์ Vue ของเรา ให้สร้างโฟลเดอร์ `.github/workflows/` และสร้างไฟล์ชื่อ `deploy.yml` ครับ

**ไฟล์: `.github/workflows/deploy.yml`**
```yaml
# ตั้งชื่องานให้เข้าใจง่าย
name: Deploy Vue App to Netlify

# 1. Source Stage: กำหนดเงื่อนไขว่าหุ่นยนต์จะเริ่มทำงานตอนไหน
on:
  push:
    branches:
      - main # ทำงานเมื่อมีการ Push โค้ดเข้าสาขา main

# 2. กำหนดสายพานการทำงาน (Jobs)
jobs:
  build_and_deploy:
    # เลือกเครื่อง Server จำลองของ GitHub ให้เป็น Ubuntu
    runs-on: ubuntu-latest

    # 3. กำหนดขั้นตอนการทำงาน (Steps)
    steps:
      # Step 3.1: ดึง Source code ของเรามาไว้ที่เครื่องจำลอง (Checkout)
      - name: Checkout Repository
        uses: actions/checkout@v3

      # Step 3.2: ติดตั้ง Node.js
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'

      # Step 3.3: Build Stage - ติดตั้งแพ็กเกจและบิลด์โปรเจกต์
      - name: Install Dependencies and Build
        run: |
          npm ci
          npm run build

      # Step 3.4: Deploy Stage - ส่งโฟลเดอร์ dist ไปที่ Netlify
      - name: Deploy to Netlify
        uses: nwtgck/actions-netlify@v2.0 # เรียกใช้ Action สำเร็จรูปของ Netlify
        with:
          publish-dir: './dist' # โฟลเดอร์ที่บิลด์เสร็จแล้วของ Vue 3
          production-branch: main
          github-token: ${{ secrets.GITHUB_TOKEN }} # ตัวแปรที่ GitHub มีให้เอง
        env:
          # ดึงกุญแจลับที่เราซ่อนไว้ใน GitHub Secrets มาใช้งาน
          NETLIFY_AUTH_TOKEN: ${{ secrets.NETLIFY_AUTH_TOKEN }}
          NETLIFY_SITE_ID: ${{ secrets.NETLIFY_SITE_ID }}
```
*เพียงเท่านี้นะครับ! ครั้งต่อไปที่น้องๆ พิมพ์คำสั่ง `git commit` และ `git push origin main` ตัว GitHub Actions จะจับสัญญาณได้ และเริ่มกระบวนการ Build และโยนไฟล์ขึ้น Netlify ให้เราทันที เราแค่รอนั่งจิบกาแฟสวยๆ รอลิงก์เว็บอัปเดตได้เลย!*

## 5. 🛡️ เคล็ดลับจากคัมภีร์ลับ (Under the Hood / Pro-Tips)

**🚀 1. ท่าไม้ตายแบบไร้โค้ดด้วย Vercel / Netlify Git Integration**
รู้หรือไม่ครับ? ความจริงแล้วแพลตฟอร์มอย่าง **Vercel** หรือ **Netlify** ในยุคปัจจุบัน เขาฉลาดล้ำหน้าไปมาก เขามีฟีเจอร์ "Git Integration" ที่เราแทบไม่ต้องมานั่งเขียนไฟล์ `.yml` เลยด้วยซ้ำ! 
เราเพียงแค่เข้าไปที่หน้าเว็บ Vercel (หรือ Netlify) กดปุ่ม **"Import Project from Git"** แล้วผูกบัญชี GitHub เข้าด้วยกัน จากนั้นตัว Vercel จะตรวจจับอัตโนมัติว่าโปรเจกต์เราเป็น Vue.js ตั้งค่าคำสั่ง `npm run build` และ Directory `dist` ให้อัตโนมัติ เมื่อตั้งค่าเสร็จ ทุกครั้งที่ Push โค้ด Vercel ก็จะไปดึงโค้ดเรามา Deploy ให้เองโดยตรงเลยครับ ถือเป็นวิธีที่ง่ายที่สุด (Zero-Config) สำหรับโปรเจกต์ที่ไม่ต้องการปรับแต่ง Pipeline อะไรซับซ้อนนัก

**🧪 2. แทรกการทำ Automated Testing เพื่อความปลอดภัยสูงสุด**
การทำ Continuous Deployment (CD) ที่ดี ต้องมีความมั่นใจระดับ 100% ว่าโค้ดใหม่จะไม่ทำระบบพัง! พี่แนะนำให้เพิ่ม Step การรัน Unit Test (`npm run test:unit`) หรือ E2E Test (เช่น Cypress) คั่นไว้ก่อน Step การ Deploy ในไฟล์ `.yml` ครับ หากเทสต์ไม่ผ่าน (Fail) ตัว GitHub Actions จะหยุดการทำงานทันที (Halt) ทำให้โค้ดที่พังๆ ไม่มีทางหลุดขึ้น Production เด็ดขาด! นี่แหละคือเกราะป้องกันระดับ Enterprise

**🔒 3. การจัดการ Environment Variables**
เวลาเราทำแอป Vue ที่ต้องต่อ API យើងมักจะมีไฟล์ `.env` อยู่ในเครื่อง (เช่น URL ของ Backend) ซึ่งไฟล์นี้เรา **ห้าม!** อัปโหลดขึ้น GitHub เด็ดขาดเพราะเสี่ยงต่อความปลอดภัย 
เวลา Deploy ผ่าน GitHub Actions เราจะต้องเอาค่าใน `.env` พวกนี้ ไปใส่ไว้ใน **GitHub Secrets** ด้วยครับ แล้วให้ Workflow อ่านค่ามาฝังลงไปตอนสั่ง `npm run build` เพื่อให้เว็บที่ถูกสร้างขึ้นมาใหม่รู้จัก URL ของ Backend ครับ

## 6. 🏁 บทสรุป (To be continued...)
การนำสถาปัตยกรรม **CI/CD** เข้ามาใช้เปรียบเสมือนการว่าจ้างหุ่นยนต์มาทำงานแทนเราครับ มันเปลี่ยนกระบวนการที่เคยน่าเบื่อ กินเวลา และเสี่ยงต่อความผิดพลาด ให้กลายเป็นกระบวนการที่รวดเร็ว มั่นคง และเป็นอัตโนมัติ 

ไม่ว่าน้องๆ จะเลือกเขียน Workflow เองอย่างละเอียดด้วย **GitHub Actions** หรือใช้ทางลัดสุดล้ำด้วย **Git Integration ของ Vercel/Netlify** สิ่งที่ได้กลับมาคือ "เวลา" ที่เราสามารถเอาไปโฟกัสกับการเขียนฟีเจอร์เจ๋งๆ ให้กับ Vue.js ของเราได้อย่างเต็มที่นั่นเองครับ 

และนี่ก็คือวิชาทั้งหมดในการสร้างและส่งมอบแอปพลิเคชัน Vue.js ระดับ Enterprise ที่เราเดินทางกันมาอย่างยาวนาน! หวังว่าน้องๆ จะได้รับประโยชน์และนำไปปรับใช้ในหน้างานจริงได้อย่างมืออาชีพนะครับ ลุยเลย!

---

**ต้องการที่ปรึกษาและพัฒนาระบบ Web Application หรือ Frontend Architecture ให้กับธุรกิจของคุณ?**
ทีมงาน WP Solution พร้อมให้บริการออกแบบและพัฒนาซอฟต์แวร์แบบครบวงจร 
ดูรายละเอียดบริการของเราได้ที่: [www.wpsolution2017.com](https://www.wpsolution2017.com)
หรือพูดคุยปรึกษาเบื้องต้นได้ที่ Line: wisit.p