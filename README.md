# 🛰️ ESP32 IoT Project (Transmitter)

## 📌 Overview

โปรเจกต์นี้เป็นระบบ IoT ฝั่งส่งข้อมูล (Transmitter) ที่ใช้ ESP32 สำหรับอ่านค่าจากเซ็นเซอร์ **HTU21D** และ **DS3231** แล้วส่งข้อมูลไปยังตัวรับ/เกตเวย์ผ่าน **ESP-NOW**
เหมาะสำหรับระบบที่ต้องการเก็บข้อมูลอุณหภูมิและเวลา โดยประหยัดพลังงานด้วย Deep Sleep

---

## 🔧 Hardware

* **ESP32** – ตัวหลักสำหรับอ่าน sensor และส่งข้อมูล
* **HTU21D** – Sensor วัดอุณหภูมิและความชื้น
* **DS3231** – Real-Time Clock (ตรวจสอบเวลา)
* **ปุ่มกด** – ใช้เข้าสู่โหมด Config (Web Server AP Mode)

---

## 🧠 Features

* อ่านค่าจาก HTU21D (อุณหภูมิ / ความชื้น)
* อ่านเวลาแบบ Real-Time จาก DS3231
* จัดรูปแบบข้อมูล JSON ก่อนส่ง (Timestamp + Temperature)
* ส่งข้อมูลผ่าน ESP-NOW ไปยัง MAC ของ Receiver
* เข้าสู่ **Deep Sleep 60 นาที** หลังส่งข้อมูลสำเร็จ
* รองรับการตั้งค่าแรกเริ่มผ่าน **Web Server (AP Mode)**
* ตรวจสอบสถานะการส่งข้อมูลสำเร็จก่อนเข้าสู่ Deep Sleep

---

## 📂 Project Structure

```
ESP32_Transmitter/
├── main/
│   ├── main.c
├── components/
│   ├── htu21d/
│   └── ds3231/
├── docs/
│   └── diagram.png   # System Diagram
└── README.md
```

---

## 🔄 System Flow

1. **เช็ค NVS** ว่าอุปกรณ์ถูกตั้งค่าครั้งแรกหรือไม่ (`configured flag`)
2. **ถ้าไม่ถูกตั้งค่า** → เปิด AP + Web Server ให้ตั้งค่า MAC ของ Receiver และเวลา
3. **ถ้าตั้งค่าแล้ว** → สร้าง Tasks:

   * อ่าน TEMP จาก HTU21D
   * อ่าน TIME จาก DS3231
   * ส่งข้อมูลผ่าน ESP-NOW
   * ตรวจสอบปุ่มกดเพื่อเข้าสู่โหมด Config
4. รอให้ **TEMP_OK_BIT** และ **TIME_OK_BIT** ครบก่อนส่งข้อมูล
5. จัด JSON ข้อมูลที่ส่ง:

```json
{
  "ts": 1696990123000,
  "values": {
    "temperature": 26.50,
    "store": true
  }
}
```

* `ts` = UTC timestamp (ms)
* `temperature` = อุณหภูมิ (°C)
* `store` = true → บันทึกลง server/DB

6. ส่ง JSON ผ่าน **ESP-NOW** ไปยัง MAC ของ Receiver
7. รอยืนยันการส่งสำเร็จ (`SEND_SUCCESS_BIT`) → เข้าสู่ **Deep Sleep 60 นาที**
8. หลัง Deep Sleep ครบ 60 นาที → ตื่นขึ้นมาอ่านค่าใหม่แล้วส่งรอบถัดไป

---

## 🖼️ System Diagram

![System Diagram](docs/diagram.png)

**คำอธิบาย Diagram:**

* HTU21D → อ่าน Temp/Humidity
* DS3231 → อ่านเวลา
* ESP32 → จัด JSON → ส่งผ่าน ESP-NOW → Receiver
* Deep Sleep → รอรอบต่อไป

---

## ⚙️ Build & Flash

1. เปิด ESP-IDF Terminal หรือ VSCode + ESP-IDF Extension
2. ตั้ง target ESP32:

```bash
idf.py set-target esp32
```

3. Build โค้ด:

```bash
idf.py build
```

4. Flash ลง ESP32:

```bash
idf.py flash
```

5. ดู Serial Monitor:

```bash
idf.py monitor
```

---

## 📝 Configuration (ครั้งแรก)

* หลัง Flash ESP32 จะเข้าสู่ **Web Server AP Mode**
* **SSID:** `DS3231_Setup`
* **Password:** `12345678`
* เข้าเว็บ `http://192.168.4.1`
* ตั้งค่า:

  * MAC ของ Receiver
  * เวลา (วัน/เดือน/ปี, ชั่วโมง/นาที)
* บันทึก → ESP32 รีสตาร์ทและใช้งานปกติ

**เข้าสู่โหมด Config อีกครั้ง:**

* กดปุ่มบน ESP32 → เปิด AP + Web Server

---

## 📊 Example JSON from ESP32

```json
{
  "ts": 1696990123000,
  "values": {
    "temperature": 26.5,
    "store": true
  }
}
```

---

## 🧾 License

MIT
