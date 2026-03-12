# Microservice Task Management System (Secure & Scalable)

## 1. ชื่อโครงงาน
Microservice Task Management System with Centralized Logging and JWT Authentication

## 2. สมาชิกกลุ่ม
1. [นายสรภูริ์  ทองจันทร์] [67543206078-7]
2. [นายเกียรติศักดิ์   อุปพรม] [67543206002-7]

## 3. ภาพรวมระบบ
ระบบจัดการ Task ที่ออกแบบมาในรูปแบบ Microservices โดยเน้นความปลอดภัย (Security) และการตรวจสอบย้อนกลับ (Observability) ผ่าน Centralized Logging ระบบประกอบด้วย Services หลัก ได้แก่ Auth, Task, Log และมี Nginx เป็น Gateway ในการจัดการ HTTPS และ Rate Limiting

## 4. Architecture Overview
ระบบประกอบด้วย 5 ส่วนหลัก:
1. **Nginx Cloud Gateway**: ทำหน้าที่เป็น Reverse Proxy, HTTPS Termination และ Load Balancer/Rate Limiter
2. **Auth Service (Node.js)**: จัดการการเข้าสู่ระบบและออก JWT Token
3. **Task Service (Node.js)**: จัดการ CRUD ของ Task รายบุคคล
4. **Log Service (Node.js)**: รับ Log จาก Service ต่างๆ และบันทึกลงฐานข้อมูลเพื่อการตรวจสอบ
5. **Database (PostgreSQL)**: แยกฐานข้อมูลตาม Service เพื่อความเป็นอิสระ (Set 2 architecture)

## 5. Auth Service
- **หน้าที่**: ตรวจสอบสิทธิ์ผู้ใช้ (Email/Password) และสร้าง Access Token (JWT)
- **Port**: 3001
- **Endpoints หลัก**: 
  - `POST /api/auth/login`: ตรวจสอบสิทธิ์และคืนค่า JWT
  - `GET /api/auth/verify`: ตรวจสอบความถูกต้องของ Token
  - `GET /api/auth/me`: คืนค่าข้อมูลผู้ใช้ปัจจุบัน

## 6. Task Service
- **หน้าที่**: จัดการรายการงาน (Create, Read, Update, Delete)
- **Port**: 3002
- **Security**: ต้องมี JWT Token ใน Header `Authorization: Bearer <token>` ทุกครั้ง
- **Endpoints หลัก**:
  - `GET /api/tasks`: ดึงรายการงานทั้งหมด
  - `POST /api/tasks`: สร้างงานใหม่
  - `PUT /api/tasks/:id`: แก้ไขงาน
  - `DELETE /api/tasks/:id`: ลบงาน

## 7. Database Design (Set 2: 3 DB)
ระบบใช้สถาปัตยกรรมแบบ Database-per-service:
- `auth-db`: เก็บตาราง `users` และสิทธิ์การใช้งาน
- `task-db`: เก็บตาราง `tasks` ผูกกับ `user_id`
- `user-db`: เก็บตาราง `profiles` (ข้อมูลส่วนตัวเพิ่มเติม)

## 8. Nginx HTTPS Gateway / Gateway Strategy
ใช้ Nginx เป็น API Gateway จัดการความปลอดภัยดังนี้:
- **HTTPS Only**: บังคับใช้ TLS 1.2/1.3 บน Port 443
- **Rate Limiting**: 
  - Login: จำกัด 5 requests ต่อนาที (ป้องกัน Brute Force)
  - API ทั่วไป: จำกัด 30 requests ต่อนาที
- **Gateway Strategy**: รวม API ทุก Service ไว้ภายใต้ Domain เดียวกัน แยกตาม Path (`/api/auth`, `/api/tasks`, `/api/logs`)

## 9. JWT Authentication Flow
1. User ส่ง Credentials ไปที่ `/api/auth/login`
2. Auth Service ออก JWT Token (บรรจุ `user_id`, `role`, `expires`)
3. Frontend เก็บ Token ใน LocalStorage หรือ Cookie
4. เมื่อเรียกใช้ Page/API อื่น จะส่ง Token ไปกับ Header `Authorization: Bearer <token>`
5. Service เป้าหมายตรวจสอบความถูกต้องของ Token ก่อนประมวลผล

## 10. Basic Activity Log Integration
- **Log Service**: ทำหน้าที่เป็นศูนย์กลางรับ Log ผ่าน REST API
- **Log Types**:
  - `INFO`: การทำงานปกติ (Login success, Task created)
  - `WARN`: สิ่งที่ควรระวัง (Login failed)
  - `ERROR`: ข้อผิดพลาดของระบบ
- **การเชื่อมต่อ**: ทุก Service จะส่ง Log ไปยัง `log-service` เมื่อเกิดเหตุการณ์สำคัญ

## 11. Frontend Main App
- เขียนด้วย Vanilla JS (HTML/CSS)
- หน้า Dashboard สำหรับจัดการ Task (Add/Edit/Delete/Filter)
- ระบบจัดการ State สำหรับแสดงผล Task และการกรองข้อมูล

## 12. Frontend Log Viewer
- หน้าต่างพิเศษสำหรับ Admin ในการดู Activity Logs ย้อนหลัง
- แสดงผลในรูปแบบตาราง พร้อมความสามารถในการ Filter ตาม Service หรือ Log Level

## 13. วิธี run ระบบ
1. เตรียมไฟล์ `.env` ตามตัวอย่าง
2. ตรวจสอบว่า Docker และ Docker Compose ติดตั้งเรียบร้อยแล้ว
3. รันคำสั่ง:
   ```bash
   docker-compose up --build -d
   ```
4. เข้าไปที่ `https://localhost` (กดยอมรับ Self-signed cert)

## 14. Environment Variables
```env
POSTGRES_DB=taskdb
POSTGRES_USER=admin
POSTGRES_PASSWORD=secret
JWT_SECRET=your_jwt_secret_key
JWT_EXPIRES=1h
```

## 15. Sample Endpoints
- Login: `POST https://localhost/api/auth/login`
- Get Tasks: `GET https://localhost/api/tasks` (Require JWT)
- Get Logs: `GET https://localhost/api/logs` (Require JWT & Admin Role)

## 16. วิธีทดสอบ
1. ใช้ Postman หรือ cURL ในการทดสอบ API ผ่าน Gateway
2. ทดสอบ Rate Limit โดยการ Login รัวๆ จะได้รับ HTTP 429
3. ตรวจสอบ Log ในหน้า Log Viewer หลังจากทำรายการต่างๆ

## 17. หลักฐานใน docs/
- Screenshots การรันระบบผ่าน HTTPS
- Screenshots ผลลัพธ์การเรียก API สำเร็จ
- Screenshots หน้า Log Viewer

## 18. ปัญหาที่พบและวิธีแก้
- **Self-signed Cert**: Browser แจ้งเตือนความปลอดภัย -> วิธีแก้: กดยอมรับ (Advanced -> Proceed)
- **CORS**: ปัญหาการเรียกข้าม Service -> วิธีแก้: ใช้ Nginx Gateway รวมทุก Service ไว้ภายใต้ Host/Port เดียวกัน

## 19. สิ่งที่ยังไม่สมบูรณ์
- ระบบ User Profile (User Service) อยู่ในระหว่างการพัฒนา
- ระบบการ Reset Password