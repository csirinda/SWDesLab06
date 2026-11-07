# Lab06 Factory Pattern

เมื่อผู้ใช้ทำการจองนัดหมายฉีดวัคซีน (`addAppointment`) สำเร็จ จะมี**"การสร้างบริการส่งการแจ้งเตือน (Notification Service)"**

เนื่องจากการแจ้งเตือนอาจมีหลายช่องทาง (เช่น **Email**, **SMS**, **LINE**) การใช้ Factory Pattern จะช่วยรวบรวมตรรกะการสร้าง Object ของการแจ้งเตือนไว้ที่จุดเดียว ทำให้ Controller ไม่ต้องสนใจว่าแต่ละช่องทางมีวิธีสร้างและส่งข้อมูลอย่างไร

## Lab Preparation

1. ไปที่ GitHub classroom link
   <<GitHub classroom link>>

2. accept the invitation & click ที่ assignment link ที่ github classroom สร้างให้

3. ไปที่ Code >> GitHub CLI >> copy the instruction เช่น gh repo clone XXX

4. เปิด project folder ของคุณใน your local computer ด้วย VS Code

5. paste คำสั่ง 'gh repo clone XXX' ใน VS Code Terminal

6. cd เข้าไปใน folder ที่ clone มา

7. รันคำสั่งต่อไปนี้ใน VS Code Terminal เพื่อติดตั้ง package ที่เกี่ยวข้อง และรัน project VacQ

```bash
npm i
npm run dev
```

เมื่อต้องการหยุดการทำงานของ server ให้กดปุ่ม CTRL+C

Scope การทำงานของ project VacQ สามารถศึกษาได้จากคลิป SW Dev Practice: Chapter 01: SRS n UI

## Project Testing

ทดสอบการทำงานของโค้ดด้วยการยิง request ด้วย Postman

1. เปิดโปรแกรม Postman
2. File >> Import >> เลือกไฟล์ \_\_tests\_\_/PM.json
3. Collection "SWDesignLab06" แสดงในแถบด้านซ้ายมือ
4. เลือก Collection "SWDesignLab06" กดปุ่ม Run ที่มุมบนขวา เพื่อยิง request ทั้งหมดใน Collection
5. กดปุ่ม Start Run
6. request ที่มีเลข 1 สีเขียวด้านขวามือ คือคะแนนที่ได้รับ (คะแนนเต็ม 5)

## การส่งงาน

รันคำสั่งต่อไปนี้ใน VS Code Terminal

```bash
git add .
git commit -m 'Lab06'
git push -u origin main
```

\*นิสิตสามารถส่งกี่ครั้งก็ได้ จนกว่าจะ deadline

## การดูคะแนน

1. ไปที่เว็บ GitHub classroom ของนักเรียน
2. ไปที่แท็บ Actions
3. รอจน Actions ทำงานเสร็จ (icon หยุดหมุน และแสดงเป็น icon สีเขียวหรือสีแดง)
4. ไปที่แท็บ Code
5. เปลี่ยนจาก branch "main" เป็น "grades"
6. click ที่ไฟล์ grade.md เพื่อดูคะแนน (คะแนนเต็ม 5)

## Lab Part I: Instructions

### สถานการณ์ (Business Scenario)

เมื่อผู้ใช้จองนัดหมายวัคซีนผ่าน `POST /api/v1/hospitals/:hospitalId/appointments` ผู้ใช้อาจระบุช่องทางการรับการแจ้งเตือนมาใน Body เช่น `notificationType: "email"` หรือ `"sms"` หรือ `"line"`

---

### Step 1: สร้าง Notification Classes (Concrete Products)

สร้างไฟล์ `services/notifications/NotificationServices.js` เพื่อเก็บคลาสการแจ้งเตือนของแต่ละช่องทาง:

```javascript
// Interface / Base Class การแจ้งเตือน
class BaseNotification {
  async send(user, appointment) {
    throw new Error("Method 'send()' must be implemented.");
  }
}

// 1. ช่องทาง Email
class EmailNotification extends BaseNotification {
  async send(user, appointment) {
    console.log(
      `📧 [EMAIL] Sending email to ${user.email} for appointment on ${appointment.apptDate}`,
    );
    // โค้ดส่ง Email จริง เช่น Nodemailer
  }
}

// 2. ช่องทาง SMS
class SmsNotification extends BaseNotification {
  async send(user, appointment) {
    console.log(
      `📱 [SMS] Sending SMS to user ID ${user._id} for appointment on ${appointment.apptDate}`,
    );
    // โค้ดส่ง SMS จริง เช่น Twilio
  }
}

// 3. ช่องทาง LINE Notify
class LineNotification extends BaseNotification {
  async send(user, appointment) {
    console.log(
      `💬 [LINE] Sending LINE message to user ID ${user._id} for appointment on ${appointment.apptDate}`,
    );
    // โค้ดส่ง LINE Notify API
  }
}

module.exports = {
  EmailNotification,
  SmsNotification,
  LineNotification,
};
```

---

### Step 2: สร้าง Factory Class (The Factory)

สร้างไฟล์ `services/notifications/NotificationFactory.js` เพื่อทำหน้าที่ตัดสินใจและสร้าง Object ของการแจ้งเตือนตามประเภทที่ส่งเข้ามา:

```javascript
const {
  EmailNotification,
  SmsNotification,
  LineNotification,
} = require('./NotificationServices');

class NotificationFactory {
  /**
   * สร้าง Notification Object ตามประเภทที่ระบุ
   * @param {string} type - 'email' | 'sms' | 'line'
   */
  static createNotification(type) {
    switch (type ? type.toLowerCase() : 'email') {
      case 'sms':
        return new SmsNotification();
      case 'line':
        return new LineNotification();
      case 'email':
      default:
        return new EmailNotification(); // Default fallback
    }
  }
}

module.exports = NotificationFactory;
```

---

### Step 3: เรียกใช้งานใน `controllers/appointments.js`

นำ `NotificationFactory` ไปใช้ในฟังก์ชัน `addAppointment` เมื่อจองวัคซีนสำเร็จ:

```javascript
const Appointment = require('../models/Appointment');
const NotificationFactory = require('../services/notifications/NotificationFactory');

//@desc     Add appointment
//@route    POST /api/v1/hospitals/:hospitalId/appointments/
//@access   Private
exports.addAppointment = async (req, res, next) => {
  try {
    req.body.hospital = req.params.hospitalId;
    req.body.user = req.user.id;
    //Code ส่วนตรวจสอบจำนวน appointment ยังคงไว้ทั้งหมด

    // 1. บันทึกข้อมูลนัดหมายลง Database
    const appointment = await Appointment.create(req.body);

    // 2. ใช้ Factory สร้าง Notification Object ตามที่ผู้ใช้ระบุมาใน req.body.notificationType
    const notificationType = req.body.notificationType; // เช่น 'email', 'sms', 'line'
    const notificationService =
      NotificationFactory.createNotification(notificationType);

    // 3. เรียกใช้เมธอด send() โดยไม่ต้องรู้ว่าเป็น Email, SMS หรือ LINE
    await notificationService.send(req.user, appointment);

    res.status(200).json({
      success: true,
      data: appointment,
    });
  } catch (error) {
    console.log(error);
    return res.status(500).json({
      success: false,
      message: 'Cannot create Appointment',
    });
  }
};
```

---

### 💡 ประโยชน์ของการปรับโค้ดด้วย Factory Pattern

1. **ลดความซับซ้อนใน Controller:** Controller ไม่ต้องมีคำสั่ง `if (type === 'email') { ... } else if (type === 'sms') { ... }` ซ้อนกันหลายชั้น
2. **ขยายระบบง่าย (Open/Closed Principle):** หากอนาคตต้องการเพิ่มช่องทางใหม่ เช่น **"Push Notification"**

- เพียงแค่สร้างคลาส `PushNotification`
- เพิ่ม `case 'push':` ใน `NotificationFactory`
- โดย**ไม่ต้องแตะต้องโค้ดใน Controller เลย**

3. **Encapsulation:** แยกตรรกะการสร้างวัตถุ (Object Creation) ออกจากตรรกะการทำงานทางธุรกิจ (Business Logic) อย่างชัดเจน

## Lab Part II: Class Diagram for Factory Pattern

1. วาด Class Diagram สำหรับส่วน Factory Pattern ด้วย PlantUML ซึ่งประกอบด้วยคลาสต่อไปนี้
   -controllers/appointments.js
   -NotificationFactory
   -BaseNotification
   -EmailNotification
   -SmsNotification
   -LineNotification
   โดยระบุด้วยว่าคลาสใดเป็น concrete/Interface/abstract class แล้วใส่ความสัมพันธ์ให้ถูกต้อง
2. ส่งในส่วน attachment Files ใน Assignment Lab06 นี้
