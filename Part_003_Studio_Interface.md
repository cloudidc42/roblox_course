# ตอนที่ 3: ทำความเข้าใจ Studio Interface
## Part 3: Understanding the Studio Interface

---

**ระดับ:** พื้นฐาน (Beginner)  
**เวลาเรียน:** 60-90 นาที  
**ข้อกำหนดเบื้องต้น:** ตอนที่ 1-2

---

## 🎯 วัตถุประสงค์การเรียนรู้

เมื่อเรียนจบตอนนี้ คุณจะสามารถ:
1. บอกชื่อและหน้าที่ของทุกส่วนใน Studio Interface
2. ใช้งาน Explorer Panel ได้อย่างคล่องแคล่ว
3. ใช้งาน Properties Panel ได้
4. ใช้งาน Output Panel สำหรับ Debug
5. Customize Layout ให้เหมาะกับตัวเอง

---

## 1. ภาพรวม Studio Interface

### 1.1 ส่วนประกอบหลัก

```
+================================================+
|  File | Edit | View | Insert | Model | ...    |  ← Menu Bar
+================================================+
|  [Play] [Stop] | [Select][Move][Scale][Rotate] | ← Main Toolbar
+================================================+
|        |                              |        |
| Tools  |                              |Explorer|
| Panel  |     3D Viewport              +--------+
|        |   (Workspace View)           |Propert-|
+--------+                              |ies     |
| Output |                              |        |
| Panel  |                              |        |
+--------+------------------------------+--------+
```

### 1.2 รายการ Panels ทั้งหมด

| Panel | ตำแหน่งเริ่มต้น | หน้าที่ |
|-------|---------------|---------|
| Explorer | ขวาบน | โครงสร้าง Objects |
| Properties | ขวาล่าง | Properties ของ Object |
| Output | ล่าง | Error/Print messages |
| Command Bar | ล่าง | รัน Lua แบบ inline |
| Toolbox | ซ้าย | Assets Library |
| Script Editor | กลาง (Tab) | เขียนโค้ด |

---

## 2. Menu Bar (แถบเมนู)

### 2.1 File Menu

```
File
├── New                     (Ctrl+N)
├── Open Recent
├── Open from Roblox...
├── Close
├── Save                    (Ctrl+S)
├── Save As...
├── Save to Roblox As...    (Ctrl+Shift+S)
├── Publish to Roblox       
├── Publish to Roblox As...
├── Studio Settings...      (Alt+S)
└── Exit                    (Alt+F4)
```

### 2.2 Edit Menu

```
Edit
├── Undo                    (Ctrl+Z)
├── Redo                    (Ctrl+Y)
├── Cut                     (Ctrl+X)
├── Copy                    (Ctrl+C)
├── Paste                   (Ctrl+V)
├── Duplicate               (Ctrl+D)
├── Delete                  (Delete)
├── Select All              (Ctrl+A)
├── Find...                 (Ctrl+F) - ค้นหา Script
└── Replace...              (Ctrl+H) - แทนที่ใน Script
```

### 2.3 View Menu

```
View
├── Explorer                (Alt+X)
├── Properties
├── Output                  (Alt+F2)
├── Diagnostics
├── Script Editor
├── Script Analysis
├── Toolbox                 (Ctrl+Alt+X)
├── Command Bar
├── Asset Manager
└── Game Settings
```

### 2.4 Insert Menu

```
Insert
├── Part
│   ├── Block Part
│   ├── Sphere Part
│   ├── Cylinder Part
│   ├── Wedge Part
│   └── Corner Wedge Part
├── Special Mesh
├── Model
├── Script
├── LocalScript
├── ModuleScript
└── ...
```

### 2.5 Model Menu

```
Model
├── Move                    (G)
├── Scale                   (R)
├── Rotate                  (R)
├── Transform
├── Anchor
├── Weld Together
├── Collisions ON/OFF
├── Snap To Grid
└── ...
```

### 2.6 Test Menu

```
Test
├── Play                    (F5)
├── Run                     (F8)
├── Play Here               (Shift+F5)
├── Pause                   (F6)
├── Resume                  (F6)
├── Stop                    (Shift+F5)
└── Team Test Settings
```

---

## 3. Main Toolbar (แถบเครื่องมือหลัก)

### 3.1 Play Controls

```
[▶ Play]  [⏸ Pause]  [⏹ Stop]
   F5        F6        F5

[▶▶ Play Here]  [▶ Run]
   Shift+F5      F8
```

### 3.2 Transform Tools

```
[↖ Select]  [↕ Move]  [⤡ Scale]  [↺ Rotate]
    V          G          R          R

หรือใช้ Keyboard:
V = Select Mode
G = Move/Grab
R = Rotate/Resize (สลับระหว่าง Scale และ Rotate)
```

### 3.3 Other Tools

```
[Anchor]  [Collisions]  [Join Surfaces]  [Snap to Grid]
```

---

## 4. Explorer Panel (อย่างละเอียด)

### 4.1 ภาพรวม Explorer

Explorer แสดง **Tree Structure** ของทุก Object ในเกม:

```
Explorer
├── 🎮 Workspace
│   ├── 📷 Camera
│   ├── 🌍 Terrain
│   ├── 🟦 Baseplate
│   └── 🟩 SpawnLocation
├── 🔄 ReplicatedFirst
├── 🔄 ReplicatedStorage
├── 💾 ServerScriptService
│   └── 📜 Script
├── 💽 ServerStorage
├── 🎯 StarterGui
├── 🎒 StarterPack
├── 👤 StarterPlayer
│   ├── 📜 StarterPlayerScripts
│   └── 📜 StarterCharacterScripts
├── 💡 Lighting
├── 🔊 SoundService
└── 👥 Teams
```

### 4.2 การใช้งาน Explorer

**การ Select Object:**
```
คลิกซ้ายครั้งเดียว = Select Object เดียว
Ctrl + คลิก = Select หลาย Object
Shift + คลิก = Select ช่วง Objects
```

**การ Expand/Collapse:**
```
คลิกที่ ▶ เพื่อ Expand
คลิกที่ ▼ เพื่อ Collapse
Alt + คลิก ▶ เพื่อ Expand ทั้งหมด
```

**การ Search:**
```
พิมพ์ในช่อง Search ด้านบน Explorer
รองรับ Partial Match (ค้นหาแบบไม่สมบูรณ์)
```

**คลิกขวาที่ Object:**
```
Context Menu:
├── Cut
├── Copy
├── Paste
├── Duplicate          (Ctrl+D)
├── Delete             (Delete)
├── Rename             (F2)
├── Select Children
├── Select Descendants
├── Add Instance...
└── Open Script (สำหรับ Scripts)
```

### 4.3 Icons ใน Explorer

| Icon | ความหมาย |
|------|---------|
| 🟦 | Part/BasePart |
| 📜 | Script |
| 📄 | LocalScript |
| 📦 | ModuleScript |
| 🖼️ | Model |
| 📷 | Camera |
| 💡 | Light |
| 🔊 | Sound |
| 🎨 | Decal/Texture |

---

## 5. Properties Panel (อย่างละเอียด)

### 5.1 ภาพรวม

Properties Panel แสดงและแก้ไข Properties ของ Object ที่เลือก:

```
Properties - Part "Baseplate"
+---------------------------+
| Appearance               ▼|
|   BrickColor:  [████] Mid Gray |
|   CastShadow:  ✓         |
|   Color:       [███████]  |
|   Material:    SmoothPlastic |
|   Transparency: 0         |
+---------------------------+
| Behavior                 ▼|
|   Anchored:    ✓         |
|   CanCollide:  ✓         |
|   CanQuery:    ✓         |
|   CanTouch:    ✓         |
+---------------------------+
| Data                     ▼|
|   Name:    Baseplate      |
|   Parent:  Workspace      |
+---------------------------+
| Part                     ▼|
|   CFrame:  0, -10, 0      |
|   Position: 0, -10, 0     |
|   Rotation: 0, 0, 0       |
|   Size:    512, 20, 512   |
+---------------------------+
```

### 5.2 Categories ของ Properties

**Appearance (ลักษณะ):**
```
BrickColor  - สีแบบ Roblox Classic
Color       - สีแบบ RGB
Material    - วัสดุ (Plastic, Metal, Wood, etc.)
Transparency - ความโปร่งใส (0=ทึบ, 1=ใส)
Reflectance - การสะท้อนแสง
CastShadow  - ทำเงาหรือไม่
```

**Behavior (พฤติกรรม):**
```
Anchored    - ยึดติดไม่เคลื่อนไหว
CanCollide  - ชนกับ Object อื่นหรือไม่
CanQuery    - ถูก Raycast ได้หรือไม่
CanTouch    - Trigger Touch Events หรือไม่
Massless    - ไม่มีมวล (สำหรับ Accessories)
```

**Data (ข้อมูล):**
```
Name    - ชื่อของ Object
Parent  - Parent ของ Object
```

**Part/Geometry (รูปร่าง):**
```
CFrame    - Position + Rotation รวมกัน
Position  - ตำแหน่ง (X, Y, Z)
Rotation  - การหมุน (X, Y, Z) ในองศา
Size      - ขนาด (กว้าง, สูง, ลึก)
```

### 5.3 การแก้ไข Properties

```
วิธีที่ 1: คลิกที่ค่าแล้วพิมพ์ใหม่
วิธีที่ 2: Drag Slider (สำหรับตัวเลข)
วิธีที่ 3: คลิก Checkbox (สำหรับ Boolean)
วิธีที่ 4: เลือกจาก Dropdown (สำหรับ Enum)
วิธีที่ 5: เปิด Color Picker (สำหรับ Color)
```

---

## 6. Output Panel (อย่างละเอียด)

### 6.1 ภาพรวม Output

Output Panel แสดง:
- ผลลัพธ์จาก `print()`
- Warnings (สีเหลือง)
- Errors (สีแดง)
- Stack Traces เมื่อเกิด Error

### 6.2 ตัวอย่าง Output

```lua
-- โค้ดนี้จะสร้าง Output ต่างๆ
print("ข้อความปกติ")          -- ขาว
warn("ข้อความเตือน")           -- เหลือง
error("ข้อความผิดพลาด")        -- แดง

-- Output ที่เห็น:
-- ข้อความปกติ
-- ⚠ ข้อความเตือน - Script:2
-- ✗ ข้อความผิดพลาด - Script:3 stack end
```

### 6.3 การ Filter Output

```
Output มีปุ่ม Filter ด้านบน:
[All] [Errors] [Warnings] [Messages] [Information]
```

### 6.4 การ Clear Output

```
คลิกขวาใน Output > Clear Output
หรือ กด Ctrl+L
```

### 6.5 การใช้ Output สำหรับ Debug

```lua
-- ตัวอย่างการใช้ print ช่วย Debug
local function calculateDamage(baseDamage, multiplier)
    print("=== calculateDamage called ===")
    print("baseDamage:", baseDamage)
    print("multiplier:", multiplier)
    
    local result = baseDamage * multiplier
    
    print("result:", result)
    print("==============================")
    
    return result
end

local damage = calculateDamage(10, 1.5)
-- Output จะแสดง:
-- === calculateDamage called ===
-- baseDamage: 10
-- multiplier: 1.5
-- result: 15
-- ==============================
```

---

## 7. Script Editor (อย่างละเอียด)

### 7.1 การเปิด Script Editor

```
วิธีที่ 1: ดับเบิลคลิกที่ Script ใน Explorer
วิธีที่ 2: คลิกขวาที่ Script > Open Script
วิธีที่ 3: View > Script Editor
```

### 7.2 ส่วนประกอบของ Script Editor

```
+------------------------------------------+
| Tab: Script1 | Tab: Script2 | + New      | ← Tabs
+------------------------------------------+
|  1  | -- สวัสดีโลก                       | ← Line Numbers
|  2  | print("Hello World!")               |
|  3  |                                     |
|  4  | local x = 10                        |
|  5  | local y = 20                        |
|  6  | print(x + y)                        |
+------------------------------------------+
| Line: 6  Col: 14  |  Lua  | UTF-8        | ← Status Bar
+------------------------------------------+
```

### 7.3 Keyboard Shortcuts ใน Script Editor

```
การแก้ไข:
Ctrl+Z          - Undo
Ctrl+Y          - Redo  
Ctrl+A          - Select All
Ctrl+C          - Copy
Ctrl+X          - Cut
Ctrl+V          - Paste
Ctrl+D          - Duplicate Line (ถ้าไม่มี Selection)

การนำทาง:
Ctrl+G          - Go to Line
Ctrl+F          - Find
Ctrl+H          - Find & Replace
Ctrl+Home       - ไปบรรทัดแรก
Ctrl+End        - ไปบรรทัดสุดท้าย

Format:
Ctrl+/          - Toggle Comment
Tab             - Indent (เพิ่มการเยื้อง)
Shift+Tab       - Unindent (ลดการเยื้อง)

Script Actions:
Ctrl+S          - Save Script
F5              - Play/Test
```

### 7.4 Auto-complete

```lua
-- พิมพ์บางส่วนแล้วกด Tab หรือ Enter เพื่อ Complete
game:Ge  →  game:GetService()
workspace.Ba  →  workspace.Baseplate
print(  →  print()
```

### 7.5 Syntax Highlighting

| สี | ความหมาย |
|----|---------|
| น้ำเงิน | Keywords (local, function, if, etc.) |
| เขียว | Comments (--) |
| ส้ม | Strings ("text") |
| ม่วง | Numbers |
| ขาว | Variables และ Functions |

---

## 8. Command Bar

### 8.1 ภาพรวม

Command Bar เป็น Input เล็กๆ ที่ด้านล่าง ใช้รัน Lua code แบบ Immediate:

```
View > Command Bar เพื่อเปิด
```

### 8.2 การใช้ Command Bar

```lua
-- พิมพ์โค้ดแล้วกด Enter เพื่อรัน

-- ตัวอย่างการใช้งาน:
print("Hello!")

-- เลือก Parts ทั้งหมด
game.Selection:Set(workspace:GetDescendants())

-- เพิ่ม Part ด้วย Command
Instance.new("Part", workspace)

-- ดู Properties ของ Object
print(workspace.Baseplate.Size)
```

### 8.3 ประโยชน์ของ Command Bar

```
1. ทดสอบโค้ดเร็วๆ โดยไม่ต้องสร้าง Script
2. รัน Commands แบบ One-time
3. Debug ค่า Properties
4. Batch Operations บน Objects
```

---

## 9. Toolbox Panel

### 9.1 ภาพรวม Toolbox

Toolbox มี Assets สำเร็จรูปที่ใช้ใน Project:

```
Toolbox
├── Inventory (Assets ของคุณ)
├── Marketplace  
│   ├── Models
│   ├── Meshes
│   ├── Images
│   ├── Decals
│   ├── Audio
│   ├── Videos
│   └── Plugins
└── Creator Store
```

### 9.2 การค้นหาใน Toolbox

```
1. เปิด Toolbox (Ctrl+Alt+X)
2. พิมพ์ชื่อ Asset ที่ต้องการ
3. กด Enter หรือคลิก Search
4. ดับเบิลคลิกหรือลาก Asset ไปใน Viewport
```

### 9.3 ข้อควรระวังเมื่อใช้ Toolbox

```
⚠️ ระวัง: Free Models อาจมี Malicious Code!
ตรวจสอบ Scripts ทุกตัวก่อนใช้
ดู Reviews และ Ratings ก่อนเลือก
```

---

## 10. Asset Manager

### 10.1 ภาพรวม

Asset Manager ใช้จัดการ Assets ในโปรเจกต์:

```
View > Asset Manager
```

```
Asset Manager
├── Images
├── Meshes
├── Audio
├── Video
├── Animations
└── Models
```

### 10.2 การ Import Assets

```
1. เปิด Asset Manager
2. คลิกปุ่ม Import (ลูกศรขึ้น)
3. เลือกไฟล์จากคอมพิวเตอร์
4. รอ Upload
```

---

## 11. การ Customize Layout

### 11.1 การ Drag Panels

```
คลิกที่ Tab ของ Panel แล้วลากไปยังตำแหน่งที่ต้องการ
```

### 11.2 Layout ที่แนะนำ

**สำหรับผู้เริ่มต้น:**
```
+------------------------+----------+
|                        | Explorer |
|     3D Viewport        +----------+
|                        |Properties|
+------------------------+----------+
| Output                            |
+-----------------------------------+
```

**สำหรับ Scripter:**
```
+------------------+----+-----------+
|                  |Scri|           |
|  3D Viewport     |pt  | Explorer  |
|                  |Edi-+           |
|                  |tor | Properties|
+------------------+----+           |
| Output                |           |
+-----------------------+-----------+
```

### 11.3 การ Reset Layout

```
View > Reset View Layout
```

---

## 12. Game Settings

### 12.1 การเข้าถึง

```
Home > Game Settings
หรือ File > Game Settings
```

### 12.2 หมวดหมู่ใน Game Settings

```
Game Settings
├── Basic Info
│   ├── Name
│   ├── Description
│   └── Genre
├── Access
│   ├── Public/Private
│   └── Playable Age
├── Security
│   ├── Allow Copying
│   └── HTTP Requests
├── Places
│   └── Configure Places
├── Monetization
│   ├── VIP Servers
│   └── Paid Access
├── Avatar
│   ├── Avatar Type (R6/R15)
│   └── Avatar Restrictions
└── World
    ├── Gravity
    ├── Jump Power
    └── Walk Speed
```

---

## 13. โค้ดตัวอย่างสำหรับตอนนี้

### 13.1 ทดสอบ Explorer Structure

```lua
-- Script: ExploreWorkspace
-- วางใน: ServerScriptService

-- พิมพ์โครงสร้าง Workspace
local function printHierarchy(instance, indent)
    indent = indent or 0
    local prefix = string.rep("  ", indent)
    print(prefix .. instance.Name .. " (" .. instance.ClassName .. ")")
    
    for _, child in pairs(instance:GetChildren()) do
        printHierarchy(child, indent + 1)
    end
end

print("=== Workspace Structure ===")
printHierarchy(workspace)
print("===========================")
```

### 13.2 ทดสอบ Properties

```lua
-- Script: TestProperties
-- วางใน: ServerScriptService

-- ค้นหา Baseplate
local baseplate = workspace:FindFirstChild("Baseplate")

if baseplate then
    print("=== Baseplate Properties ===")
    print("Name:", baseplate.Name)
    print("ClassName:", baseplate.ClassName)
    print("Size:", baseplate.Size)
    print("Position:", baseplate.Position)
    print("Color:", baseplate.Color)
    print("Material:", baseplate.Material)
    print("Anchored:", baseplate.Anchored)
    print("CanCollide:", baseplate.CanCollide)
    print("Transparency:", baseplate.Transparency)
    print("============================")
else
    warn("ไม่พบ Baseplate!")
end
```

### 13.3 สร้าง Objects ด้วยโค้ด

```lua
-- Script: CreateObjects
-- วางใน: ServerScriptService

-- สร้าง Folder สำหรับจัดระเบียบ
local folder = Instance.new("Folder")
folder.Name = "MyObjects"
folder.Parent = workspace

-- สร้าง Parts หลายๆ ชิ้น
local colors = {
    BrickColor.new("Bright red"),
    BrickColor.new("Bright blue"),
    BrickColor.new("Bright green"),
    BrickColor.new("Bright yellow"),
    BrickColor.new("Bright orange")
}

for i, color in ipairs(colors) do
    local part = Instance.new("Part")
    part.Name = "ColorPart_" .. i
    part.Size = Vector3.new(3, 3, 3)
    part.Position = Vector3.new((i - 3) * 5, 3, 0)
    part.BrickColor = color
    part.Anchored = true
    part.Parent = folder
end

print("สร้าง Parts เรียบร้อย!")
print("ดูใน Explorer > Workspace > MyObjects")
```

### 13.4 ใช้ Output สำหรับ Debug

```lua
-- Script: DebugExample
-- วางใน: ServerScriptService

-- ฟังก์ชันช่วย Debug
local function debugLog(category, message, value)
    local log = "[" .. category .. "] " .. message
    if value ~= nil then
        log = log .. ": " .. tostring(value)
    end
    print(log)
end

-- ตัวอย่างการใช้งาน
debugLog("INIT", "Script started")
debugLog("INFO", "Workspace children count", #workspace:GetChildren())
debugLog("TEST", "Math check", 2 + 2)

-- ทดสอบ warn และ error
warn("นี่คือ Warning ทดสอบ")

-- error ด้วย pcall (ไม่หยุด Script)
local success, err = pcall(function()
    error("นี่คือ Error ทดสอบ")
end)

if not success then
    print("Caught error:", err)
end

debugLog("DONE", "Script finished")
```

---

## 14. Keyboard Shortcuts สรุป

### 14.1 Studio Shortcuts

```
เกี่ยวกับ Files:
Ctrl+N      - New Place
Ctrl+O      - Open Place
Ctrl+S      - Save
Ctrl+Shift+S - Save As

เกี่ยวกับ Edit:
Ctrl+Z      - Undo
Ctrl+Y      - Redo
Ctrl+D      - Duplicate
Delete      - Delete
Ctrl+A      - Select All
F2          - Rename

เกี่ยวกับ View:
F4          - Properties
Alt+X       - Explorer
Alt+F2      - Output
Ctrl+Alt+X  - Toolbox

เกี่ยวกับ Transform:
V           - Select
G           - Move
R           - Rotate
Ctrl+L      - Toggle Local/World Space
,           - Decrease Snap
.           - Increase Snap

เกี่ยวกับ Play:
F5          - Play
F8          - Run
Shift+F5    - Play Here
F6          - Pause/Resume
```

---

## 📚 แบบฝึกหัดตอนที่ 3

### แบบฝึกหัดที่ 1: สำรวจ Interface
1. เปิด Roblox Studio
2. ระบุทุก Panel ที่อธิบายในบทนี้
3. เปิดและปิด Panel แต่ละอัน

### แบบฝึกหัดที่ 2: Explorer
1. Expand ทุก Node ใน Explorer
2. ค้นหา "Baseplate" ด้วย Search
3. Select Multiple Objects พร้อมกัน

### แบบฝึกหัดที่ 3: Properties
1. Select Baseplate
2. ดู Properties ทั้งหมด
3. เปลี่ยน Color ของ Baseplate
4. เปลี่ยน Size เป็น Vector3.new(100, 20, 100)

### แบบฝึกหัดที่ 4: Output
1. สร้าง Script ใน ServerScriptService
2. เขียนโค้ด print, warn, error
3. กด Play และดู Output ที่ต่างกัน

### แบบฝึกหัดที่ 5: Command Bar
1. เปิด Command Bar
2. พิมพ์: `print(workspace.Baseplate.Size)`
3. พิมพ์: `Instance.new("Part", workspace)`
4. ดูผลลัพธ์

### แบบฝึกหัดที่ 6: Script Editor
1. สร้าง Script ใหม่
2. ทดสอบ Keyboard Shortcuts ต่างๆ
3. ลอง Auto-complete
4. ทดสอบ Find & Replace

---

## 💡 เคล็ดลับ

1. **จำ Keyboard Shortcuts** - ช่วยให้ทำงานเร็วขึ้นมาก
2. **ใช้ Search ใน Explorer** - เมื่อ Project ใหญ่
3. **อ่าน Output ทุกครั้ง** - บอก Error ที่เกิดขึ้น
4. **ใช้ Command Bar ทดสอบ** - เร็วกว่า Play
5. **Customize Layout** - ให้เหมาะกับ Workflow ของคุณ

---

## ⏭️ ตอนถัดไป

ในตอนที่ 4 เราจะเรียนรู้:
- Basic Parts และ Models
- การสร้างและจัดการ Parts
- Part Types ต่างๆ
- การสร้าง Models

---

*ตอนที่ 3/100 | ระดับ: พื้นฐาน | เวลา: 60-90 นาที*
