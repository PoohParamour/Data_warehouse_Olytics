# Marketing Funnel by Olist - Knowledge Base

## Overview
ชุดข้อมูล **Marketing Funnel by Olist** เป็นชุดข้อมูลเสริม (Supplementary Dataset) ที่ทาง Olist ปล่อยออกมาคู่กับชุดข้อมูล E-Commerce หลัก (Brazilian E-Commerce Public Dataset) 

ชุดข้อมูลนี้ทำหน้าที่เก็บประวัติ **"ก่อนการขาย" (Pre-sales / Seller Acquisition)** โดยบันทึกวงจรการทำการตลาดแบบ B2B เพื่อดึงดูดและหา "ผู้ขาย (Sellers)" เข้ามาเปิดร้านบนแพลตฟอร์ม Olist ตั้งแต่เป็นคนที่สนใจ (Lead) ไปจนถึงคนที่เซ็นสัญญาเปิดร้านได้สำเร็จ (Closed Deal)

เราสามารถนำชุดข้อมูลนี้ไป Join กับชุดข้อมูล E-Commerce หลักได้ผ่านคอลัมน์ `seller_id` เพื่อวิเคราะห์ภาพรวมการทำงานข้ามระบบ ตั้งแต่ต้นทุนการหาลูกค้า (Sellers) ไปจนถึงผลงานยอดขายที่พวกเขาทำได้

## Data Schema & Relationships

ชุดข้อมูลนี้ประกอบด้วย 2 ตารางหลัก ที่มีความสัมพันธ์กันผ่านคอลัมน์ `mql_id`:

### 1. `olist_marketing_qualified_leads_dataset.csv` (MQL - ว่าที่ผู้ขาย)
เก็บข้อมูลของคนที่ติดต่อเข้ามา หรือถูกคัดกรองเบื้องต้นว่าเป็นผู้ที่มีแนวโน้มจะมาเปิดร้าน (Marketing Qualified Lead)
- **Columns:**
  - `mql_id`: (Primary Key) รหัสเฉพาะสำหรับ Lead แต่ละคน
  - `first_contact_date`: วันที่ Lead ติดต่อหรือลงทะเบียนเข้ามาเป็นครั้งแรก
  - `landing_page_id`: รหัสของ Landing Page (หน้าเว็บ) ที่ Lead คนนั้นเข้ามากดสมัคร (ช่วยให้รู้ว่าแคมเปญไหนทำงานได้ดี)
  - `origin`: ช่องทางที่ดึงดูด Lead เข้ามา เช่น `organic_search`, `paid_search`, `social`, `email` เป็นต้น

### 2. `olist_closed_deals_dataset.csv` (ร้านค้าที่ปิดการขายได้สำเร็จ)
เก็บข้อมูลของ Lead ที่ตกลงเซ็นสัญญากับ Olist และกลายมาเป็นผู้ขายอย่างเป็นทางการ (Closed Deals)
- **Columns:**
  - `mql_id`: (Primary Key) รหัส Lead เชื่อมโยงกับตาราง MQL
  - `seller_id`: (Foreign Key) รหัสผู้ขาย **(คอลัมน์นี้สำคัญมาก ใช้สำหรับนำไป Join กับ `olist_sellers_dataset` ในชุดข้อมูล E-Commerce หลัก)**
  - `sdr_id`: รหัสพนักงานขายส่วนหน้า (Sales Development Representative) ที่เป็นคนติดต่อคัดกรองเบื้องต้น
  - `sr_id`: รหัสพนักงานขายส่วนปิดการขาย (Sales Representative) ที่เป็นคนนำเสนอและเซ็นสัญญา
  - `won_date`: วันและเวลาที่ปิดการขายได้สำเร็จ (เซ็นสัญญา)
  - `business_segment`: หมวดหมู่ของธุรกิจที่ผู้ขายทำ เช่น `pet`, `car_accessories`, `health_beauty`
  - `lead_type`: ประเภทของธุรกิจ (เช่น ออนไลน์, ออฟไลน์, หรือทั้งคู่)
  - `lead_behaviour_profile`: โปรไฟล์พฤติกรรมของผู้ขาย (การประเมินจากทีม Sales)
  - `has_company`: มีการจดทะเบียนนิติบุคคลหรือไม่
  - `has_gtin`: สินค้ามีบาร์โค้ดสากล (GTIN) หรือไม่
  - `average_stock`: จำนวนสต็อกเฉลี่ยที่ร้านมี
  - `business_type`: โมเดลธุรกิจ เช่น `reseller` (ผู้ซื้อมาขายไป), `manufacturer` (ผู้ผลิต)
  - `declared_product_catalog_size`: จำนวนแคตตาล็อกสินค้าที่คาดหวังว่าจะนำมาลงขาย
  - `declared_monthly_revenue`: รายได้ต่อเดือนที่ประเมินไว้ตอนสมัคร (0.0 อาจหมายถึงไม่ได้ระบุ)

## Best Practices for Usage
- **Sales Conversion Rate:** สามารถนำ `mql_id` จากทั้ง 2 ตารางมานับเพื่อหา Conversion Rate ได้ (MQL ทั้งหมด เทียบกับ MQL ที่อยู่ใน Closed Deals)
- **Sales Cycle (Lead Time to Close):** คำนวณระยะเวลาตั้งแต่ดึงดูดผู้ขายได้จนถึงปิดการขาย โดยหาความต่างระหว่าง `won_date` (ตาราง Closed Deals) กับ `first_contact_date` (ตาราง MQL)
- **End-to-End Analysis (การเชื่อมกับ E-Commerce Data):**
  - นำ `seller_id` จากตาราง Closed Deals ไป Join กับตารางในคลังข้อมูล E-Commerce
  - **ตัวอย่าง 1:** วิเคราะห์หาว่า ผู้ขายที่มาจากช่องทาง (Origin) แบบ `paid_search` สามารถทำยอดขาย (Sales Revenue) บนแพลตฟอร์ม Olist ได้มากกว่าคนที่มาจาก `social` หรือไม่?
  - **ตัวอย่าง 2:** ร้านค้าประเภท `manufacturer` (ผู้ผลิต) เทียบกับ `reseller` (คนรับมาขาย) ใครได้รับคะแนนรีวิวจากลูกค้า (Review Score) ดีกว่ากัน?
