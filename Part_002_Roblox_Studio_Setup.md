# ตอนที่ 2: การติดตั้งและตั้งค่า Roblox Studio
## Part 2: Setting Up Roblox Studio

---

**ระดับ:** พื้นฐาน (Beginner)  
**เวลาเรียน:** 45-60 นาที  
**ข้อกำหนดเบื้องต้น:** บัญชี Roblox (ตอนที่ 1)

---

## 🎯 วัตถุประสงค์การเรียนรู้

เมื่อเรียนจบตอนนี้ คุณจะสามารถ:
1. ดาวน์โหลดและติดตั้ง Roblox Studio สำเร็จ
2. ตั้งค่า Studio ให้เหมาะกับการพัฒนา
3. สร้าง Project แรกของคุณ
4. เข้าใจความต้องการของระบบ (System Requirements)
5. แก้ปัญหาการติดตั้งเบื้องต้นได้

---

## 1. ความต้องการของระบบ (System Requirements)

### 1.1 Windows

| ส่วนประกอบ | ขั้นต่ำ | แนะนำ |
|-----------|--------|-------|
| OS | Windows 7 SP1 | Windows 10/11 |
| CPU | Intel Core 2 Duo | Intel Core i5+ |
| RAM | 4 GB | 8 GB+ |
| GPU | DirectX 9 | DirectX 11+ |
| พื้นที่ดิสก์ | 1 GB | 5 GB+ |
| Internet | 4 Mbps | 10+ Mbps |

### 1.2 macOS

| ส่วนประกอบ | ขั้นต่ำ | แนะนำ |
|-----------|--------|-------|
| OS | macOS 10.11 | macOS 12+ |
| CPU | Intel Core 2 Duo | Apple M1+ |
| RAM | 4 GB | 8 GB+ |
| GPU | Metal Compatible | Metal 2+ |
| พื้นที่ดิสก์ | 1 GB | 5 GB+ |
| Internet | 4 Mbps | 10+ Mbps |

### 1.3 หมายเหตุสำคัญ

```
⚠️ Roblox Studio รองรับเฉพาะ Windows และ macOS
❌ ไม่รองรับ Linux, iOS, Android, Chromebook

ถ้าใช้ Linux: ใช้ Wine หรือ Virtual Machine (ไม่แนะนำ)
```

---

## 2. ขั้นตอนการดาวน์โหลดและติดตั้ง

### 2.1 สำหรับ Windows

**ขั้นตอนที่ 1: เข้าเว็บ Roblox**
```
1. เปิดเบราว์เซอร์ (Chrome, Firefox, Edge)
2. ไปที่: www.roblox.com
3. Login ด้วยบัญชีของคุณ
```

**ขั้นตอนที่ 2: ไปที่หน้า Create**
```
1. คลิกที่ปุ่ม "Create" ในเมนูด้านบน
   หรือไปที่: www.roblox.com/create
2. คลิก "Start Creating"
```

**ขั้นตอนที่ 3: ดาวน์โหลด Studio**
```
1. คลิกปุ่ม "Download Studio" (สีเขียว)
2. ไฟล์ RobloxStudioLauncherBeta.exe จะดาวน์โหลด
3. รอให้ดาวน์โหลดเสร็จ
```

**ขั้นตอนที่ 4: ติดตั้ง**
```
1. ดับเบิลคลิกไฟล์ที่ดาวน์โหลด
2. กด "Yes" ถ้ามี UAC prompt
3. รอให้ติดตั้งเสร็จ (ประมาณ 2-5 นาที)
4. Studio จะเปิดอัตโนมัติ
```

### 2.2 สำหรับ macOS

**ขั้นตอนที่ 1-2:** เหมือนกับ Windows

**ขั้นตอนที่ 3: ดาวน์โหลด**
```
1. คลิก "Download Studio"
2. ไฟล์ RobloxStudio.dmg จะดาวน์โหลด
```

**ขั้นตอนที่ 4: ติดตั้ง**
```
1. เปิดไฟล์ .dmg
2. ลาก Roblox Studio ไปไว้ใน Applications folder
3. เปิด Roblox Studio จาก Applications
4. ถ้ามี Security warning กด "Open"
```

---

## 3. การ Login ครั้งแรก

### 3.1 หน้า Login

เมื่อเปิด Studio ครั้งแรก:

```
1. กรอก Username และ Password ของ Roblox
2. คลิก "Log In"
3. ถ้ามี 2-Factor Authentication ให้กรอกรหัสด้วย
```

### 3.2 หน้า Landing Page

หลัง Login จะเห็น Landing Page:

```
New Tab (หน้าใหม่)
├── Recent (โปรเจกต์ล่าสุด)
├── My Games (เกมของคุณ)
├── Templates (แม่แบบ)
├── Tutorials (บทเรียน)
└── New
    ├── Baseplate
    ├── Flat Terrain
    ├── Classic Baseplate
    └── อื่นๆ
```

---

## 4. สร้าง Project แรก

### 4.1 เลือก Template

1. บนหน้า Landing Page เลือก **"New"**
2. คลิก **"Baseplate"**
3. รอให้ Studio โหลด Project

### 4.2 ส่วนประกอบของ Baseplate

เมื่อ Project โหลดเสร็จ คุณจะเห็น:

```
Workspace
├── Camera (กล้อง)
├── Terrain (พื้นดิน)
├── Baseplate (แผ่นพื้นสีเทา)
└── SpawnLocation (จุด Spawn)
```

---

## 5. การตั้งค่า Studio

### 5.1 File > Studio Settings

เข้าถึงได้โดย: **File > Studio Settings** หรือ **Alt+S**

#### General Settings

```
Display Name: [ชื่อที่แสดง]
Theme: Dark (แนะนำ) / Light

Font:
- Editor Font: Consolas (แนะนำ)
- Font Size: 14 (แนะนำ)
```

#### Scripting Settings

```
Auto-complete: ON (แนะนำ)
Script Editor Color Theme: Dark (แนะนำ)
Tab Width: 4 (แนะนำ)
Indent Using Spaces: ON

Line Numbers: ON
Syntax Highlighting: ON
Bracket Matching: ON
```

#### Camera Settings

```
Camera Speed: 0.5 (เริ่มต้น)
Camera Shift Speed: 0.2
Camera Zoom Speed: 10
Mouse Sensitivity: 1.0
```

### 5.2 ตั้งค่าแนะนำสำหรับผู้เริ่มต้น

```
Theme: Dark         -- ถนอมสายตา
Font Size: 14-16    -- อ่านง่าย
Auto-complete: ON   -- ช่วยเขียนโค้ด
Line Numbers: ON    -- ดูตำแหน่งโค้ดง่าย
```

---

## 6. ทำความรู้จัก Layout หลัก

### 6.1 ภาพรวม Layout

```
+------------------------------------------+
|  Menu Bar (File, Edit, View, etc.)        |
+------------------------------------------+
|  Toolbar (Play, Stop, Tools, etc.)        |
+--+-----------------------------------+---+
|  |                                   |   |
|  |        3D Viewport                | E |
|  |                                   | x |
|  |                                   | p |
|E |                                   | l |
|x |                                   | o |
|p |                                   | r |
|l |                                   | e |
|o |                                   | r |
|r +-----------------------------------+   |
|e |  Output                           |   |
|r |                                   +---+
|  |                                   |Pro|
+--+-----------------------------------+per|
                                       |tie|
                                       |s  |
                                       +---+
```

### 6.2 ส่วนประกอบหลัก

1. **Menu Bar** - เมนูหลักทั้งหมด
2. **Toolbar** - ปุ่มที่ใช้บ่อย
3. **3D Viewport** - พื้นที่ทำงาน 3D
4. **Explorer** - โครงสร้าง Object ทั้งหมด
5. **Properties** - Properties ของ Object ที่เลือก
6. **Output** - แสดงผลการ print และ errors

---

## 7. การนำทางใน 3D Viewport

### 7.1 Mouse Controls

```
การมองรอบๆ (Rotate View):
- กด Right Click แล้วลาก

การเลื่อน View (Pan):
- กด Middle Click แล้วลาก
- หรือ Hold Shift + Right Click แล้วลาก

การ Zoom:
- Scroll Wheel
- หรือ Right Click + Scroll

การบิน (Fly):
- Hold Right Click
- กด W/A/S/D เพื่อเลื่อน
- กด E เพื่อขึ้น, Q เพื่อลง
- Shift เพื่อเพิ่มความเร็ว
```

### 7.2 Keyboard Shortcuts พื้นฐาน

```
F - Focus ไปที่ Object ที่เลือก
Ctrl+Z - Undo
Ctrl+Y - Redo
Ctrl+S - Save
Ctrl+D - Duplicate Object
Delete - ลบ Object ที่เลือก

Play:
F5 - Play (Test ใน Studio)
F8 - Stop
Shift+F5 - Play Here (Test ณ ตำแหน่งปัจจุบัน)
```

### 7.3 Transform Tools

```
Select Tool (V): เลือก Object
Move Tool (M): ย้าย Object (กด G)
Scale Tool (S): ปรับขนาด (กด R - Resize)  
Rotate Tool (R): หมุน Object (กด R - Rotate)
```

**หมายเหตุ:** ใน Studio ใหม่ปุ่มอยู่ใน Toolbar

---

## 8. การ Save Project

### 8.1 Save ลง Cloud (แนะนำ)

```
File > Save to Roblox
หรือ Ctrl+S
```

ข้อดี:
- บันทึกใน Cloud อัตโนมัติ
- เข้าถึงได้จากทุกที่
- ป้องกันการสูญหาย

### 8.2 Save เป็น Local File

```
File > Save As...
เลือก Location บนคอมพิวเตอร์
```

ไฟล์จะบันทึกเป็น `.rbxl` หรือ `.rbxlx`

### 8.3 Auto-Save Settings

```
Studio Settings > General
Enable Auto-save: ON
Auto-save Interval: 5 minutes (แนะนำ)
```

---

## 9. Plugin ที่แนะนำสำหรับผู้เริ่มต้น

### 9.1 เข้าถึง Plugin Store

```
Plugins > Manage Plugins
หรือ ไปที่ create.roblox.com/store/plugins
```

### 9.2 Plugin พื้นฐานที่ควรติดตั้ง

| Plugin | ประโยชน์ | ฟรี/จ่าย |
|--------|---------|---------|
| Part To Terrain | แปลง Part เป็น Terrain | ฟรี |
| Stravant - Model Reflect | สะท้อน Models | ฟรี |
| Rojo | Sync โค้ดกับ VS Code | ฟรี |
| Studio+ | เพิ่มฟีเจอร์ | ฟรี |
| GapFill | เติมช่องว่างระหว่าง Parts | ฟรี |

### 9.3 การติดตั้ง Plugin

```lua
-- Plugin ไม่ใช่โค้ด แต่เป็น Tool ใน Studio
-- ติดตั้งผ่าน Plugin Manager เท่านั้น
```

```
1. ไปที่ Plugins > Plugin Manager
2. คลิก "Install Plugins"
3. ค้นหา Plugin ที่ต้องการ
4. กด "Install"
5. Restart Studio
```

---

## 10. การตั้งค่า Script Editor

### 10.1 เปิด Script Editor

```
ดับเบิลคลิกที่ Script ใน Explorer
หรือ คลิกขวาที่ Script > Open Script
```

### 10.2 Themes สำหรับ Script Editor

ไปที่: **File > Studio Settings > Scripting > Script Editor Color Theme**

```
แนะนำสำหรับผู้เริ่มต้น:
- Dark (ค่าเริ่มต้น สีเข้ม)
- One Dark (สีเข้ม คล้าย VS Code)
- Solarized Dark (สีเข้มสบายตา)
```

### 10.3 การ Toggle Line Numbers

```
View > Line Numbers: ON/OFF
หรือ Studio Settings > Scripting > Show Line Numbers: ON
```

---

## 11. การทดสอบเกม (Play Mode)

### 11.1 Play Modes

```
Play (F5):
- จำลองเป็น Server + 1 Client
- เห็นทั้ง Server และ Client
- เหมาะสำหรับทดสอบทั่วไป

Play Here (Shift+F5):
- เหมือน Play แต่ Spawn ณ ตำแหน่งปัจจุบัน
- เหมาะทดสอบพื้นที่เฉพาะ

Run (F8):
- รัน Server เท่านั้น ไม่มี Client
- เหมาะทดสอบ Server Scripts
```

### 11.2 Team Test

```
Home > Test > เพิ่ม Players เพิ่ม
- จำลองผู้เล่นหลายคน
- ทดสอบ Multiplayer features
```

### 11.3 การหยุด Play Mode

```
กด Stop (F5 อีกครั้ง) หรือ กดปุ่ม Stop (สีแดง)

⚠️ ข้อควรระวัง:
การแก้ไขใน Play Mode จะไม่บันทึก!
เสมอ Stop ก่อนแล้วค่อยแก้ไข
```

---

## 12. การแก้ปัญหาการติดตั้ง

### 12.1 ปัญหาที่พบบ่อย

**ปัญหา: Studio ไม่เปิด / Crash ทันที**
```
แนวทางแก้ไข:
1. ตรวจสอบ System Requirements
2. อัปเดต GPU Driver
3. Reinstall Roblox Studio
4. ลองเปิดด้วย Administrator
5. ตรวจสอบ Antivirus (อาจบล็อค)
```

**ปัญหา: Login ไม่ได้**
```
แนวทางแก้ไข:
1. ตรวจสอบ Username และ Password
2. Reset Password ถ้าลืม
3. ตรวจสอบการเชื่อมต่อ Internet
4. ลอง Login บนเว็บ roblox.com ก่อน
```

**ปัญหา: Studio ทำงานช้า**
```
แนวทางแก้ไข:
1. ปิด Applications อื่นที่ไม่จำเป็น
2. ลด Graphics Quality ใน Studio Settings
3. เพิ่ม RAM (ถ้าเป็นไปได้)
4. ใช้ SSD แทน HDD
```

**ปัญหา: Script Editor ไม่เปิด**
```
แนวทางแก้ไข:
1. ตรวจสอบว่า Script ไม่ได้ Disabled
2. Restart Studio
3. ตรวจสอบ Plugin ที่อาจขัดแย้ง
```

### 12.2 Log Files

```
Windows Log Location:
%localappdata%\Roblox\logs\

macOS Log Location:
~/Library/Logs/Roblox/
```

---

## 13. Roblox Studio Updates

### 13.1 การอัปเดตอัตโนมัติ

Roblox Studio อัปเดตอัตโนมัติทุกครั้งที่เปิด:
```
1. Studio ตรวจสอบ Update เมื่อเปิด
2. Download และติดตั้ง Update อัตโนมัติ
3. Restart ถ้าจำเป็น
```

### 13.2 เวอร์ชัน Beta

Roblox มี Studio Beta ที่ทดสอบฟีเจอร์ใหม่:
```
File > Studio Settings > Studio > Enable Studio Beta Features: ON
```

⚠️ Beta Features อาจมี Bug ไม่เสถียร

---

## 14. การ Backup Project

### 14.1 Local Backup

```lua
-- ไม่ใช่โค้ด แต่เป็น Settings
File > Studio Settings
Auto-save interval: 5 นาที
Number of autosaves: 10
```

### 14.2 Version History บน Cloud

```
1. ไปที่ roblox.com/develop
2. เลือก Game ของคุณ
3. กด Configure > Version History
4. เลือก Version ที่ต้องการ Revert
```

### 14.3 แนวปฏิบัติที่ดี

```
💡 Tips สำหรับ Backup:
1. Save บ่อยๆ (Ctrl+S)
2. ใช้ Git สำหรับโปรเจกต์ใหญ่
3. เก็บ Backup ไว้หลาย Version
4. ตั้งชื่อ File ให้ชัดเจน (v1.0, v1.1)
```

---

## 15. การตั้งค่าขั้นสูงสำหรับ Developers

### 15.1 Rojo - ทำงานร่วมกับ VS Code

**Rojo** คือ Plugin ที่ช่วย Sync ไฟล์ระหว่าง VS Code และ Roblox Studio:

```
ข้อดี:
✅ ใช้ VS Code ที่คุ้นเคย
✅ Version control ด้วย Git
✅ IntelliSense ที่ดีกว่า
✅ Multi-file organization

ข้อเสีย:
❌ Setup ซับซ้อนกว่า
❌ ต้องเรียนรู้ Rojo workflow
```

### 15.2 VS Code Extension สำหรับ Roblox

```
Extensions ที่แนะนำ:
1. Roblox LSP (Language Support)
2. Luau Linting
3. Selene (Linter สำหรับ Lua)
```

---

## 16. ตัวอย่างโค้ดสำหรับตอนนี้

### 16.1 ทดสอบว่า Studio ทำงานถูกต้อง

```lua
-- Script: TestSetup
-- วางใน: ServerScriptService

-- ทดสอบ Basic Output
print("=== Roblox Studio Setup Test ===")
print("Script is running correctly!")
print("=================================")

-- แสดงข้อมูลเกม
print("Game Name:", game.Name)
print("Place ID:", game.PlaceId)

-- แสดง Roblox Version
print("Roblox Version:", version())

-- นับจำนวน Objects ใน Workspace
local objectCount = 0
for _, child in pairs(workspace:GetChildren()) do
    objectCount = objectCount + 1
end
print("Objects in Workspace:", objectCount)
```

### 16.2 ตรวจสอบ Services

```lua
-- Script: CheckServices  
-- วางใน: ServerScriptService

-- ตรวจสอบว่า Services ทำงาน
local services = {
    "Players",
    "TweenService", 
    "RunService",
    "UserInputService",
    "SoundService",
    "Lighting"
}

for _, serviceName in pairs(services) do
    local success, service = pcall(function()
        return game:GetService(serviceName)
    end)
    
    if success then
        print("✓ " .. serviceName .. " - OK")
    else
        print("✗ " .. serviceName .. " - ERROR!")
    end
end
```

### 16.3 Hello World ที่สมบูรณ์

```lua
-- Script: HelloWorld
-- วางใน: ServerScriptService

local Players = game:GetService("Players")

-- Function สำหรับทักทายผู้เล่น
local function onPlayerAdded(player)
    print("========================================")
    print("ยินดีต้อนรับ! / Welcome!")
    print("ผู้เล่น: " .. player.Name)
    print("User ID: " .. player.UserId)
    print("Account Age: " .. player.AccountAge .. " วัน/days")
    print("========================================")
    
    -- รอให้ Character โหลด
    player.CharacterAdded:Wait()
    print(player.Name .. " กำลังเล่นอยู่! / is now playing!")
end

-- เชื่อม Event
Players.PlayerAdded:Connect(onPlayerAdded)

print("Server Script: HelloWorld - Started!")
```

---

## 📚 แบบฝึกหัดตอนที่ 2

### แบบฝึกหัดที่ 1: การติดตั้ง
1. ดาวน์โหลดและติดตั้ง Roblox Studio ให้สำเร็จ
2. Login ด้วยบัญชี Roblox ของคุณ
3. สร้าง Project ใหม่จาก Baseplate template

### แบบฝึกหัดที่ 2: การตั้งค่า
1. เปิด Studio Settings
2. เปลี่ยน Theme เป็น Dark
3. ตั้ง Font Size เป็น 14
4. เปิดใช้ Auto-save

### แบบฝึกหัดที่ 3: ทดสอบ
1. สร้าง Script ใน ServerScriptService
2. เขียนโค้ดจากตัวอย่าง TestSetup
3. กด Play (F5) และดู Output

### แบบฝึกหัดที่ 4: การนำทาง
1. ทดลองใช้ Mouse Controls ทั้งหมด
2. บินรอบๆ Workspace ด้วย WASD
3. Focus ไปที่ Baseplate ด้วย F

### แบบฝึกหัดที่ 5: การ Save
1. Save Project ลง Cloud (Ctrl+S)
2. ตั้งชื่อ Project ว่า "Roblox Course Project"
3. ปิด Studio แล้วเปิดใหม่ ตรวจสอบว่า Project ยังอยู่

---

## 💡 เคล็ดลับและข้อควรระวัง

### เคล็ดลับ:
1. **ตั้ง Auto-save** - ป้องกันงานหาย
2. **ใช้ Keyboard Shortcuts** - เร็วกว่า Mouse มาก
3. **อย่าแก้ไขใน Play Mode** - งานจะหาย
4. **Save บ่อยๆ** - ก่อนทดลองอะไรที่เสี่ยง

### ข้อควรระวัง:
1. **อย่าลบ Baseplate** - จนกว่าจะรู้ว่าทำไม
2. **อย่าลบ SpawnLocation** - ผู้เล่นจะ Spawn ไม่ได้
3. **ระวัง Delete ผิด** - ใช้ Ctrl+Z เพื่อ Undo

---

## ⏭️ ตอนถัดไป

ในตอนที่ 3 เราจะเรียนรู้:
- ทุกส่วนของ Studio Interface อย่างละเอียด
- การใช้งาน Explorer, Properties, Output
- Toolbar และ Tools ต่างๆ
- Customizing the Layout

---

*ตอนที่ 2/100 | ระดับ: พื้นฐาน | เวลา: 45-60 นาที*
