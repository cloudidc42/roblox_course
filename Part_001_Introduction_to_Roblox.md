# ตอนที่ 1: แนะนำ Roblox Platform
## Part 1: Introduction to Roblox Platform

---

**ระดับ:** พื้นฐาน (Beginner)  
**เวลาเรียน:** 60-90 นาที  
**ข้อกำหนดเบื้องต้น:** ไม่มี

---

## 🎯 วัตถุประสงค์การเรียนรู้

เมื่อเรียนจบตอนนี้ คุณจะสามารถ:
1. อธิบายว่า Roblox คืออะไรและทำงานอย่างไร
2. เข้าใจโครงสร้างของ Roblox Platform
3. รู้ว่า Roblox Studio คืออะไรและใช้ทำอะไร
4. เข้าใจ Business Model ของ Roblox
5. รู้ถึงโอกาสในการสร้างรายได้จาก Roblox

---

## 1. Roblox คืออะไร?

### 1.1 ภาพรวม

**Roblox** คือ Platform เกมออนไลน์และ Game Engine ที่ช่วยให้ผู้ใช้สามารถ:
- **เล่นเกม** ที่สร้างโดยนักพัฒนาอื่นๆ
- **สร้างเกม** ด้วยตัวเองโดยใช้ Roblox Studio
- **แบ่งปัน** ผลงานกับผู้เล่นทั่วโลก
- **สร้างรายได้** จากเกมที่สร้าง

Roblox ก่อตั้งขึ้นในปี **2004** โดย **David Baszucki** และ **Erik Cassel** และเปิดตัวสู่สาธารณะในปี **2006** ปัจจุบัน (2024) มีผู้ใช้งานมากกว่า **200 ล้านบัญชี** และมีเกมมากกว่า **40 ล้านเกม** ในระบบ

### 1.2 สิ่งที่ทำให้ Roblox แตกต่าง

```
Roblox ≠ เกมทั่วไป
Roblox = Platform สำหรับสร้างและเล่นเกม
```

เปรียบได้กับ:
- **YouTube** สำหรับวิดีโอ → **Roblox** สำหรับเกม
- ทุกคนสามารถเป็นทั้ง "ผู้ชม" และ "ผู้สร้างคอนเทนต์"

### 1.3 ตัวเลขสำคัญของ Roblox (2024)

| สถิติ | ตัวเลข |
|-------|--------|
| ผู้เล่นรายวัน (DAU) | ~80 ล้านคน |
| จำนวนเกมทั้งหมด | 40+ ล้านเกม |
| อายุผู้เล่นส่วนใหญ่ | 9-17 ปี |
| เงินที่นักพัฒนาได้รับต่อปี | $700M+ |
| Robux ที่หมุนเวียนต่อปี | มากกว่า 4 พันล้าน |

---

## 2. โครงสร้างของ Roblox Platform

### 2.1 ส่วนประกอบหลัก

```
Roblox Platform
├── Roblox Client (ตัวเกม)
│   ├── Windows
│   ├── macOS
│   ├── iOS
│   ├── Android
│   └── Xbox One
├── Roblox Studio (เครื่องมือสร้างเกม)
└── Roblox Website (roblox.com)
    ├── Game Catalog
    ├── Avatar Editor
    ├── Developer Exchange (DevEx)
    └── Marketplace
```

### 2.2 Roblox Client

**Roblox Client** คือแอปพลิเคชันที่ผู้เล่นใช้เล่นเกม มีให้ดาวน์โหลดฟรีในทุก Platform

คุณสมบัติหลัก:
- เล่นเกมออนไลน์กับผู้เล่นทั่วโลก
- จัดการ Avatar (ตัวละครของคุณ)
- ซื้อขาย Items ในตลาด
- ติดตามเพื่อนและ Groups

### 2.3 Roblox Studio

**Roblox Studio** คือเครื่องมือสร้างเกมของ Roblox ซึ่งเป็น:
- **ฟรี** สำหรับทุกคน
- รองรับ **Windows** และ **macOS**
- มี **Scripting** ด้วยภาษา **Lua**
- มี **3D Modeling** tools พื้นฐาน
- มีระบบ **Testing** ในตัว

---

## 3. Roblox Studio คืออะไร

### 3.1 ภาพรวม Roblox Studio

Roblox Studio เป็น Integrated Development Environment (IDE) ที่:

```
Roblox Studio = 3D Editor + Code Editor + Game Engine
```

ประกอบด้วย:
1. **3D Viewport** - พื้นที่สร้างและแก้ไข 3D Objects
2. **Script Editor** - เขียนโค้ด Lua
3. **Explorer Panel** - ดูโครงสร้าง Objects ทั้งหมด
4. **Properties Panel** - แก้ไข Properties ของ Objects
5. **Toolbox** - Library ของ Assets สำเร็จรูป

### 3.2 สิ่งที่สร้างได้ด้วย Roblox Studio

**ประเภทเกมที่นิยม:**

| ประเภท | ตัวอย่าง | ความยาก |
|--------|---------|---------|
| Obby (Obstacle Course) | Tower of Hell | ⭐ ง่าย |
| Simulator | Pet Simulator | ⭐⭐ ปานกลาง |
| RPG | Anime Adventures | ⭐⭐⭐ ยาก |
| FPS (First Person Shooter) | Arsenal | ⭐⭐⭐ ยาก |
| Tycoon | Lumber Tycoon | ⭐⭐ ปานกลาง |
| Racing | Vehicle Simulator | ⭐⭐ ปานกลาง |
| Horror | Piggy | ⭐⭐⭐ ยาก |

---

## 4. ภาษา Lua ใน Roblox

### 4.1 Lua คืออะไร?

**Lua** (อ่านว่า "ลัว") คือภาษาโปรแกรมที่:
- สร้างขึ้นในปี 1993 ที่ประเทศบราซิล
- ออกแบบมาให้ **เรียนง่าย** และ **ทำงานเร็ว**
- ใช้ใน Roblox, World of Warcraft, LÖVE 2D
- เป็น **Scripting Language** ที่ฝังอยู่ใน Applications

### 4.2 ตัวอย่าง Lua เบื้องต้น

```lua
-- นี่คือ Comment ในภาษา Lua
-- This is a comment in Lua

-- แสดงข้อความ (Print to console)
print("สวัสดีโลก!") -- Hello, World!

-- ตัวแปรพื้นฐาน (Basic variables)
local playerName = "ผู้เล่น"  -- String
local playerLevel = 1          -- Number
local isAlive = true           -- Boolean

-- ฟังก์ชัน (Function)
local function greetPlayer(name)
    print("สวัสดี " .. name .. "! ยินดีต้อนรับ")
end

greetPlayer(playerName)
```

### 4.3 Luau - Lua สำหรับ Roblox

Roblox ใช้ **Luau** ซึ่งเป็น Lua ที่ดัดแปลงพิเศษ:
- มี **Type Checking** เพิ่มเติม
- มี **Performance Improvements**
- มี **Roblox-specific APIs**
- Compatible กับ Lua 5.1 พื้นฐาน

---

## 5. Robux และระบบเศรษฐกิจ

### 5.1 Robux คืออะไร?

**Robux** (R$) คือสกุลเงินเสมือนของ Roblox ที่ใช้:
- ซื้อ Avatar items
- ซื้อ Game Passes
- ซื้อ Developer Products ในเกม
- จ่ายค่า Premium Features

### 5.2 อัตราแลกเปลี่ยน Robux

| แพ็คเกจ | ราคา (USD) | Robux ที่ได้ |
|--------|-----------|------------|
| 400 R$ | $4.99 | 400 |
| 800 R$ | $9.99 | 800 |
| 1,700 R$ | $19.99 | 1,700 |
| 4,500 R$ | $49.99 | 4,500 |
| 10,000 R$ | $99.99 | 10,000 |

### 5.3 การสร้างรายได้สำหรับ Developers

นักพัฒนาสามารถรับ Robux จาก:

```
รายได้จาก:
1. Game Passes - ขายสิทธิ์พิเศษในเกม
2. Developer Products - ขาย Items ในเกม  
3. Private Servers - ขาย Server ส่วนตัว
4. Premium Payouts - Roblox จ่ายตาม Engagement
```

**Developer Exchange (DevEx)**:
- แปลง Robux เป็นเงินจริงได้
- อัตราปัจจุบัน: **350 Robux = $1 USD**
- ต้องมีอย่างน้อย 100,000 Robux จึงจะแลกได้
- ต้องมี Roblox Premium

---

## 6. ตัวอย่างความสำเร็จของนักพัฒนา Roblox

### 6.1 นักพัฒนาที่ประสบความสำเร็จ

**Adopt Me!** โดยทีม Uplift Games:
- มียอดเล่นมากกว่า 30 พันล้านครั้ง
- รายได้หลายร้อยล้านดอลลาร์
- ทีมงานเริ่มจากนักพัฒนา 2 คน

**Tower of Hell** โดย YXCeptional Studios:
- เกม Obby ที่ได้รับความนิยมสูงสุด
- ยอดผู้เล่นพร้อมกันมากกว่า 200,000 คน

### 6.2 นักพัฒนาอายุน้อย

Roblox มีนักพัฒนาอายุ **12-17 ปี** ที่สร้างรายได้:
- มากกว่า $50,000 ต่อปี
- บางคนสร้างรายได้มากกว่า $1 ล้านต่อปี

---

## 7. Roblox Community

### 7.1 Developer Forum (DevForum)

**devforum.roblox.com** เป็นชุมชนของนักพัฒนา Roblox:
- ถามคำถามและรับความช่วยเหลือ
- แบ่งปันผลงาน
- อ่านประกาศจาก Roblox
- หา Collaborators

### 7.2 Discord Servers

มี Discord Servers หลายแห่งสำหรับนักพัฒนา:
- Roblox Official Discord
- Roblox Developer Community
- Lua Programming

### 7.3 Creator Hub

**create.roblox.com** คือ:
- พอร์ทัลสำหรับนักพัฒนา
- Documentation ครบถ้วน
- Analytics ของเกม
- ระบบ Monetization

---

## 8. ทำไมต้องเรียน Roblox Development?

### 8.1 ประโยชน์ที่ได้รับ

**ทักษะที่ได้เรียน:**
1. **Programming** - Lua scripting ที่ถ่ายทอดไปใช้กับภาษาอื่นได้
2. **Game Design** - การออกแบบเกมที่สนุกและน่าเล่น
3. **3D Modeling** - การสร้างสภาพแวดล้อม 3D
4. **Problem Solving** - การแก้ปัญหาเชิงตรรกะ
5. **Business Skills** - การ Monetize ผลิตภัณฑ์

### 8.2 Roblox vs Unity vs Unreal

| Feature | Roblox | Unity | Unreal |
|---------|--------|-------|--------|
| ราคา | ฟรี | ฟรี/จ่าย | ฟรี/จ่าย |
| ความยาก | ง่าย | ปานกลาง | ยาก |
| ภาษา | Lua | C# | C++/Blueprint |
| Target | เกม Online | ทุกประเภท | AAA Games |
| Multiplayer | Built-in | ต้องตั้งเอง | ต้องตั้งเอง |
| Community | 200M+ users | ใหญ่มาก | ใหญ่มาก |

### 8.3 เส้นทางอาชีพ

การเรียน Roblox Development นำไปสู่:
- **Junior Game Developer** → **Senior Developer**
- **Freelance Roblox Developer** (รายได้ $20-100/ชั่วโมง)
- **Game Designer** สำหรับบริษัทเกม
- **Full-stack Developer** (ต่อยอดเรียน Unity/Unreal)

---

## 9. โครงสร้างของเกม Roblox

### 9.1 Place คืออะไร

ใน Roblox แต่ละ "เกม" เรียกว่า **Experience** และประกอบด้วย **Places** หนึ่งหรือหลาย Place:

```
Experience (เกม)
├── Starting Place (Place หลัก)
├── Lobby Place
├── Game Place 1
├── Game Place 2
└── ...
```

### 9.2 โครงสร้างภายใน Place

```
Place
├── Workspace          -- สภาพแวดล้อม 3D
├── ReplicatedStorage  -- ข้อมูลที่แชร์ระหว่าง Server-Client
├── ServerStorage      -- ข้อมูลเฉพาะ Server
├── ServerScriptService -- Scripts ที่รันบน Server
├── StarterGui         -- GUI ที่ผู้เล่นเห็น
├── StarterPack        -- Tools ที่ผู้เล่นได้รับ
├── StarterPlayer      -- Scripts ในตัวผู้เล่น
├── Lighting           -- ระบบแสง
├── SoundService       -- ระบบเสียง
└── Teams              -- ทีมในเกม
```

---

## 10. ขั้นตอนแรกสู่ Roblox Development

### 10.1 สิ่งที่ต้องทำทันที

**ขั้นตอนที่ 1:** สร้างบัญชี Roblox
- ไปที่ www.roblox.com
- กดปุ่ม "Sign Up"
- กรอกข้อมูล Username, Password, วันเกิด
- ยืนยัน Email

**ขั้นตอนที่ 2:** ดาวน์โหลด Roblox Studio
- ไปที่ www.roblox.com/create
- กดปุ่ม "Start Creating"
- ดาวน์โหลดและติดตั้ง Studio

**ขั้นตอนที่ 3:** เข้าใช้งาน Studio
- เปิด Roblox Studio
- Login ด้วยบัญชี Roblox ของคุณ
- เลือก Template เริ่มต้น

### 10.2 Templates ที่มีให้เลือก

เมื่อเปิด Studio ครั้งแรก คุณจะเห็น Templates:

| Template | เหมาะสำหรับ |
|---------|------------|
| Baseplate | เริ่มต้นจากศูนย์ |
| Flat Terrain | เกมที่ใช้ Terrain |
| Classic Baseplate | สไตล์เก่า |
| Racing | เกมแข่งรถ |
| Obby | เกม Obstacle Course |
| Team Vs Team | เกมแบบทีม |

---

## 11. แนวคิดพื้นฐานที่ต้องเข้าใจ

### 11.1 Server vs Client

ใน Roblox มีแนวคิดสำคัญเรื่อง **Server** และ **Client**:

```
Server = คอมพิวเตอร์ที่รันเกม (Roblox Servers)
Client = คอมพิวเตอร์ของผู้เล่น (เครื่องคุณ)
```

**สิ่งที่รันบน Server:**
- ตรวจสอบความถูกต้องของเกม
- จัดการข้อมูลผู้เล่น
- ตรวจสอบการ Cheat

**สิ่งที่รันบน Client:**
- แสดงผลกราฟิก
- รับ Input จากผู้เล่น
- UI animations

### 11.2 Script Types

```lua
-- Server Script (ใน ServerScriptService)
-- รันบน Server เท่านั้น
-- มี access ถึง game:GetService("Players")

-- Local Script (ใน StarterPlayerScripts)
-- รันบน Client ของผู้เล่นแต่ละคน
-- มี access ถึง LocalPlayer

-- Module Script
-- Library ของ Functions ที่แชร์ได้
-- ต้อง require() ก่อนใช้
```

### 11.3 Instances และ Objects

ทุกอย่างใน Roblox เป็น **Instance** (object):

```lua
-- ตัวอย่างการสร้าง Part ด้วยโค้ด
local newPart = Instance.new("Part")  -- สร้าง Part ใหม่
newPart.Name = "MyPart"               -- ตั้งชื่อ
newPart.Size = Vector3.new(4, 1, 4)  -- ตั้งขนาด
newPart.Position = Vector3.new(0, 5, 0)  -- ตั้งตำแหน่ง
newPart.BrickColor = BrickColor.new("Bright red")  -- ตั้งสี
newPart.Parent = workspace            -- วางใน Workspace
```

---

## 12. ตัวอย่างโค้ดแรกของคุณ

มาลองเขียนโค้ดแรกด้วยกัน! โค้ดนี้จะสร้างส่วนประกอบ 3D ง่ายๆ

### 12.1 Hello World ใน Roblox

```lua
-- Script: HelloWorld
-- ใส่ใน: ServerScriptService

-- แสดงข้อความใน Output
print("สวัสดีโลก Roblox!")
print("Hello, Roblox World!")

-- แสดงข้อมูล Game
local Players = game:GetService("Players")

-- รอให้ผู้เล่นเข้าเกม
Players.PlayerAdded:Connect(function(player)
    print("ผู้เล่น " .. player.Name .. " เข้าเกมแล้ว!")
    print("Player " .. player.Name .. " has joined the game!")
end)
```

### 12.2 สร้าง Part แรก

```lua
-- Script: CreateFirstPart
-- ใส่ใน: ServerScriptService

-- สร้าง Part ใหม่
local part = Instance.new("Part")

-- ตั้งค่า Properties
part.Name = "MyFirstPart"                    -- ชื่อของ Part
part.Size = Vector3.new(10, 1, 10)          -- ขนาด (กว้าง, สูง, ลึก)
part.Position = Vector3.new(0, 5, 0)        -- ตำแหน่ง (X, Y, Z)
part.BrickColor = BrickColor.new("Bright blue")  -- สีน้ำเงิน
part.Material = Enum.Material.SmoothPlastic  -- วัสดุ
part.Anchored = true                         -- ยึดติดกับที่

-- วางใน Workspace
part.Parent = workspace

print("สร้าง Part เรียบร้อยแล้ว!")
```

---

## 13. ระบบ Coordinate ใน Roblox

### 13.1 Vector3

Roblox ใช้ระบบ 3 มิติ (X, Y, Z):

```
X = แกนซ้าย-ขวา (Left-Right)
Y = แกนบน-ล่าง (Up-Down)  
Z = แกนหน้า-หลัง (Forward-Back)
```

```lua
-- การใช้ Vector3
local position = Vector3.new(0, 10, 0)    -- อยู่กลาง สูง 10 หน่วย
local size = Vector3.new(4, 4, 4)          -- ขนาด 4x4x4

-- การบวก Vector
local pos1 = Vector3.new(1, 0, 0)
local pos2 = Vector3.new(0, 0, 1)
local combined = pos1 + pos2              -- Vector3.new(1, 0, 1)

-- ระยะห่างระหว่างสองจุด
local distance = (pos1 - pos2).Magnitude
print("ระยะห่าง:", distance)
```

### 13.2 หน่วยวัดใน Roblox

```
1 Stud = หน่วยวัดพื้นฐานของ Roblox
1 Stud ≈ 28 cm ในโลกจริง (โดยประมาณ)

ขนาดตัวละคร R6:
- สูง: 5 Studs
- กว้าง: 2 Studs

ขนาดตัวละคร R15:
- สูง: 4.5 Studs
- กว้าง: 1.5 Studs
```

---

## 14. สรุปบทเรียน

### สิ่งที่เรียนในตอนนี้:

✅ Roblox คือ Platform สำหรับสร้างและเล่นเกมออนไลน์  
✅ Roblox Studio คือเครื่องมือสร้างเกม (ฟรี)  
✅ ภาษาโปรแกรมที่ใช้คือ Lua (Luau)  
✅ Robux คือสกุลเงินเสมือน และสามารถแปลงเป็นเงินจริงได้  
✅ โครงสร้างพื้นฐานของ Place ใน Roblox  
✅ แนวคิด Server vs Client  
✅ ระบบ Coordinate 3D  

---

## 📚 แบบฝึกหัดตอนที่ 1

### แบบฝึกหัดที่ 1: ทำความรู้จัก Roblox
1. สร้างบัญชี Roblox (ถ้ายังไม่มี)
2. เล่นเกม 3-5 เกมบน Roblox ต่างประเภทกัน
3. สังเกตว่าเกมแต่ละประเภทมีความแตกต่างกันอย่างไร

### แบบฝึกหัดที่ 2: สำรวจ Roblox Studio
1. ดาวน์โหลดและติดตั้ง Roblox Studio
2. เปิด Studio และสำรวจ Interface
3. ลองกดปุ่มต่างๆ และดูว่าทำอะไร

### แบบฝึกหัดที่ 3: โค้ดแรก
1. เปิด Roblox Studio
2. สร้าง Project ใหม่ (Baseplate)
3. ใน ServerScriptService สร้าง Script ใหม่
4. เขียนโค้ด `print("สวัสดี Roblox!")` แล้วกด Play
5. ดูผลลัพธ์ใน Output panel

### แบบฝึกหัดที่ 4: ทดลองสร้าง Part
1. ใน Script เขียนโค้ดสร้าง Part ตามตัวอย่างในตอนที่ 12.2
2. กด Play และดูว่า Part ปรากฏใน Workspace หรือไม่
3. ลองเปลี่ยนสี ขนาด และตำแหน่งของ Part

### แบบฝึกหัดที่ 5: คำถามท้ายบท
1. Robux 1,000 เท่ากับกี่ดอลลาร์ (ถ้าใช้ DevEx)?
2. Script ที่รันบน Server คือ Script ประเภทใด?
3. Vector3.new(1, 2, 3) หมายถึงตำแหน่งอะไร?
4. ความแตกต่างระหว่าง Server และ Client คืออะไร?

---

## 💡 เคล็ดลับและข้อควรระวัง

### เคล็ดลับ (Tips):
1. **เรียนรู้จากการทำ** - ลองทำทุกตัวอย่างด้วยตัวเอง
2. **อ่าน Documentation** - Roblox มี Docs ที่ครบถ้วน
3. **เข้าร่วม Community** - DevForum และ Discord มีความช่วยเหลือมากมาย
4. **สร้างโปรเจกต์เล็กๆ** - เริ่มจากเกมง่ายๆ ก่อน
5. **ดู Source Code** - ศึกษาจากเกมที่คนอื่นสร้าง

### ข้อควรระวัง (Warning):
1. **อย่าตั้ง Username ที่มีข้อมูลส่วนตัว** - ชื่อจริง, อายุ, ที่อยู่
2. **อย่าแชร์ Password** กับใคร
3. **อย่า Exploit** หรือ Cheat ในเกมของคนอื่น
4. **อ่าน Terms of Service** ของ Roblox

---

## 🔗 แหล่งข้อมูลเพิ่มเติม

- **Roblox Creator Documentation**: https://create.roblox.com/docs
- **Roblox Developer Forum**: https://devforum.roblox.com
- **Roblox Studio Download**: https://www.roblox.com/create
- **Luau Documentation**: https://luau-lang.org

---

## ⏭️ ตอนถัดไป

ในตอนที่ 2 เราจะเรียนรู้:
- การดาวน์โหลดและติดตั้ง Roblox Studio อย่างละเอียด
- การตั้งค่า Studio ให้เหมาะกับการพัฒนา
- การ Login และสร้าง Project แรก

---

*ตอนที่ 1/100 | ระดับ: พื้นฐาน | เวลา: 60-90 นาที*
