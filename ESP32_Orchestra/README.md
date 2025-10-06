# ESP32 Orchestra Conductor & Musicians
## วงดนตรีประสานเสียงด้วย ESP32, Buzzer และ ESP-NOW

โปรเจคนี้จะสร้างวงดนตรีที่ประกอบด้วย:
- **1 Conductor (ผู้บังคับการ)** - ควบคุมการเล่นเพลงและส่งคำสั่งผ่าน ESP-NOW
- **หลาย Musicians (นักดนตรี)** - รับคำสั่งและเล่นเสียงประสานกัน

## 🎵 วิธีการทำงาน

### Architecture
```
    Conductor ESP32 (Master)
    ├── MIDI Parser
    ├── ESP-NOW Broadcaster  
    └── เพลงแบ่งเป็น Parts
            |
    ┌───────┼───────┐
    │       │       │
Musician1 Musician2 Musician3
(Part A)  (Part B)  (Part C)
```

### Communication Protocol
1. **Conductor** แปลงไฟล์ MIDI เป็น parts แยกกัน
2. ส่งคำสั่ง `START_SONG`, `PLAY_NOTE`, `STOP_NOTE`, `END_SONG` ผ่าน ESP-NOW
3. **Musicians** รับคำสั่งและเล่นเสียงตาม part ของตัวเอง

## 🎼 เพลงที่รองรับ

### 1. Twinkle Twinkle Little Star (4 Parts)
- **Part A (Melody)**: โน๊ตหลัก
- **Part B (Harmony)**: เสียงประสาน
- **Part C (Bass)**: เสียงเบส
- **Part D (Rhythm)**: จังหวะ

### 2. Happy Birthday (3 Parts)  
- **Part A (Melody)**: เสียงนำ
- **Part B (Harmony)**: เสียงประสาน
- **Part C (Bass)**: เสียงเบส

### 3. Canon in D (4 Parts)
- **Part A**: Voice 1
- **Part B**: Voice 2  
- **Part C**: Voice 3
- **Part D**: Voice 4

## 🔧 ฮาร์ดแวร์

### Conductor (1 ตัว)
- ESP32 Development Board
- LED แสดงสถานะ (GPIO 2)
- Push Button สำหรับเลือกเพลง (GPIO 0)

### Musicians (3-4 ตัว)
- ESP32 Development Board แต่ละตัว
- Buzzer หรือ Small Speaker (GPIO 18)
- LED แสดงสถานะ (GPIO 2)
- Resistor 220Ω (ถ้าใช้ Speaker)

## 🔗 การเชื่อมต่อ

### Conductor
```
ESP32 Conductor    Component
GPIO 2      ----> LED (Status)
GPIO 0      ----> Push Button (Song Select)
GND         ----> Common Ground
```

### Musicians
```
ESP32 Musician     Component
GPIO 18     ----> Buzzer/Speaker (+)
GPIO 2      ----> LED (Status)
GND         ----> Buzzer/Speaker (-) & LED (-)
```

## 📡 ESP-NOW Communication

### Message Types
```cpp
typedef enum {
    MSG_SONG_START = 1,     // เริ่มเพลง
    MSG_PLAY_NOTE = 2,      // เล่นโน๊ต
    MSG_STOP_NOTE = 3,      // หยุดโน๊ต  
    MSG_SONG_END = 4,       // จบเพลง
    MSG_SYNC_TIME = 5       // ซิงค์เวลา
} message_type_t;

typedef struct {
    message_type_t type;
    uint8_t song_id;        // รหัสเพลง
    uint8_t part_id;        // Part A, B, C, D (0-3)
    uint8_t note;           // MIDI Note (0-127)
    uint8_t velocity;       // ความแรง (0-127)
    uint32_t timestamp;     // เวลาในการเล่น
    uint16_t duration_ms;   // ความยาวโน๊ต
} orchestra_message_t;
```

### Broadcasting Strategy
- ใช้ **Broadcast Address** `FF:FF:FF:FF:FF:FF`
- Musicians กรองข้อความตาม `part_id` ของตัวเอง
- Timestamp synchronization เพื่อเล่นพร้อมกัน

## 🚀 วิธีการใช้งาน

### 1. Setup Musicians
1. Flash โค้ด `musician.ino` ลงใน ESP32 แต่ละตัว
2. กำหนด `MUSICIAN_ID` (0, 1, 2, 3) ให้ต่างกันในแต่ละตัว
3. เชื่อมต่อ Buzzer กับ GPIO 18

### 2. Setup Conductor  
1. Flash โค้ด `conductor.ino` ลงใน ESP32
2. เชื่อมต่อ Push Button กับ GPIO 0
3. เชื่อมต่อ LED กับ GPIO 2

### 3. การเล่น
1. เปิด Musicians ทุกตัวก่อน (LED จะกระพริบแสดงว่าพร้อม)
2. เปิด Conductor 
3. กด Button บน Conductor เพื่อเลือกเพลง
4. Conductor จะส่งสัญญาณเริ่มเพลง
5. Musicians จะเล่นเสียงประสานกัน

## 📁 โครงสร้างไฟล์

```
ESP32_Orchestra/
├── README.md
├── conductor/                 # โค้ดสำหรับ Conductor
│   ├── conductor.ino
│   ├── midi_songs.h          # เพลง MIDI ที่แปลงแล้ว
│   ├── espnow_conductor.h    # ESP-NOW Communication
│   └── orchestra_common.h    # Common definitions
├── musician/                 # โค้ดสำหรับ Musicians  
│   ├── musician.ino
│   ├── sound_player.h        # Sound generation
│   ├── espnow_musician.h     # ESP-NOW Communication
│   └── orchestra_common.h    # Common definitions
├── songs/                    # เพลงต้นฉบับ
│   ├── twinkle_star_4parts.mid
│   ├── happy_birthday_3parts.mid
│   └── canon_in_d_4parts.mid
└── tools/
    └── midi_to_orchestra.py  # แปลง MIDI เป็น Orchestra format
```

## 🎯 การเรียนรู้

นักเรียนจะได้เรียนรู้:
1. **MIDI Processing** - การแปลงไฟล์เสียงเป็นโน๊ต
2. **ESP-NOW Broadcasting** - การสื่อสารแบบ One-to-Many
3. **Time Synchronization** - การเล่นเพลงให้ประสานกัน
4. **Embedded Systems** - การทำงานร่วมกันของหลาย MCU
5. **Digital Music** - หลักการเสียงดนตรีดิจิทัล

## 🔧 Troubleshooting

### ปัญหาที่พบบ่อย
1. **Musicians ไม่รับสัญญาณ** - ตรวจสอบ ESP-NOW setup
2. **เสียงไม่ประสาน** - ปรับ timestamp synchronization  
3. **Buzzer เสียงเบา** - เช็คการเชื่อมต่อและ PWM settings
4. **การตอบสนองล่าช้า** - ลด delay ใน main loop

### Debug Tips
- ใช้ Serial Monitor ดู ESP-NOW messages
- LED กระพริบแสดงสถานะการรับส่งข้อมูล
- ทดสอบทีละ Musician ก่อนเล่นแบบ full orchestra

---

## 🎵 "ให้เสียงดนตรีเกิดจากความร่วมมือ ไม่ใช่แค่ ESP32 ตัวเดียว!"