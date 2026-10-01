# ตอนที่ 8: การสร้าง Terrain เบื้องต้น
## Part 8: Basic Terrain Creation

---

**ระดับ:** พื้นฐาน (Beginner)  
**เวลาเรียน:** 90-120 นาที  
**ข้อกำหนดเบื้องต้น:** ตอนที่ 1-7

---

## 🎯 วัตถุประสงค์การเรียนรู้

เมื่อเรียนจบตอนนี้ คุณจะสามารถ:
1. ใช้งาน Terrain Editor ใน Roblox Studio
2. สร้างภูมิประเทศพื้นฐาน (หุบเขา, ภูเขา, แม่น้ำ)
3. ใช้ Terrain Tools ต่างๆ (Add, Subtract, Paint, etc.)
4. สร้าง Terrain ด้วยโค้ด
5. เข้าใจ Terrain Materials และ Biomes

---

## 1. Terrain ใน Roblox คืออะไร?

### 1.1 ภาพรวม

**Terrain** คือระบบ Voxel-based ที่ช่วยสร้างภูมิประเทศแบบ Smooth:
- ไม่ใช่ Parts ทั่วไป
- ใช้ระบบ Voxel (ก้อนขนาด 4x4x4 Studs)
- รองรับวัสดุหลายแบบ (Grass, Water, Rock, etc.)
- Smooth กว่า Parts ปกติ
- มี Physics ในตัว

```lua
-- Terrain อยู่ใน Workspace
local terrain = workspace.Terrain
print(terrain.ClassName)  -- "Terrain"

-- Properties หลัก
print(terrain.WaterWaveSize)    -- ขนาดคลื่นน้ำ
print(terrain.WaterWaveSpeed)   -- ความเร็วคลื่น
print(terrain.WaterColor)       -- สีน้ำ
print(terrain.WaterTransparency)-- ความใสน้ำ
```

### 1.2 ความแตกต่างระหว่าง Terrain vs Parts

| คุณสมบัติ | Terrain | Parts |
|---------|---------|-------|
| รูปร่าง | Smooth, Voxel | Fixed shapes |
| Performance | ดีกว่าสำหรับพื้นดิน | ดีสำหรับ Objects เล็ก |
| ขนาด | ใหญ่มากได้ | จำกัด |
| Script | ใช้ Terrain API | ปรับแต่งได้หลากหลาย |
| วัสดุ | Terrain Materials | Part Materials |

---

## 2. Terrain Editor

### 2.1 เปิด Terrain Editor

```
Home > Terrain Editor
หรือ View > Terrain Editor
```

### 2.2 Terrain Editor Tools

```
Terrain Editor
├── Create
│   ├── Generate           -- สร้าง Terrain อัตโนมัติ
│   ├── Import             -- Import Heightmap
│   └── Clear              -- ลบ Terrain ทั้งหมด
├── Edit
│   ├── Add                -- เพิ่ม Terrain
│   ├── Subtract           -- ลบ Terrain
│   ├── Paint              -- ทาสีวัสดุ
│   ├── Smooth             -- ทำให้เรียบ
│   ├── Flatten            -- ทำให้แบน
│   ├── Erode              -- กัดกร่อน
│   └── Sea Level          -- ตั้งระดับน้ำทะเล
└── Region
    ├── Select             -- เลือกพื้นที่
    ├── Copy               -- Copy Terrain
    ├── Paste              -- Paste Terrain
    ├── Move               -- ย้าย Terrain
    ├── Rotate             -- หมุน Terrain
    ├── Fill               -- เติมพื้นที่
    └── Replace            -- แทนที่วัสดุ
```

---

## 3. Create Tools

### 3.1 Generate (สร้างอัตโนมัติ)

```
1. เปิด Terrain Editor
2. คลิก "Generate"
3. ตั้งค่า:
   - Biome Type: Temperate, Artic, Tropics, etc.
   - Position & Size: พื้นที่ที่จะ Generate
   - Seed: ตัวเลข Random (เปลี่ยนเพื่อได้ผลต่างกัน)
   - Caves: เพิ่มถ้ำ
   - Lakes: เพิ่มทะเลสาบ
4. กด "Generate"
```

**การตั้งค่า Generate:**

| ตัวเลือก | ความหมาย |
|---------|---------|
| Biome | ประเภทภูมิอากาศ |
| Position | จุดศูนย์กลาง |
| Size | ขนาดที่จะสร้าง |
| Seed | ค่า Random |
| Caves | มีถ้ำหรือไม่ |
| Lakes | มีทะเลสาบหรือไม่ |

### 3.2 Import Heightmap

```
Import รูปภาพ Grayscale เพื่อสร้าง Terrain:
- สีขาว = ความสูงมาก
- สีดำ = ความสูงน้อย
- รองรับ: PNG, BMP, TGA
```

```
1. คลิก "Import"
2. เลือก Heightmap image
3. เลือก Colormap image (optional)
4. กำหนดขนาดและตำแหน่ง
5. กด "Import"
```

---

## 4. Edit Tools

### 4.1 Add Tool (เพิ่ม Terrain)

```
1. เลือก "Add" ใน Edit Tools
2. เลือก Material (Grass, Rock, etc.)
3. ตั้ง Brush Size (ขนาดแปรง)
4. ตั้ง Brush Strength (ความแรง)
5. คลิกหรือลากใน Viewport เพื่อวาด
```

**เคล็ดลับ Add Tool:**
- Hold Shift เพื่อลบแทนเพิ่ม
- ปรับ Brush Shape (Sphere, Cube, Cylinder)
- ใช้ Plane Lock เพื่อวาดในระนาบเดียว

### 4.2 Subtract Tool (ลบ Terrain)

```
1. เลือก "Subtract"
2. ตั้ง Brush Settings
3. คลิกเพื่อลบ Terrain
```

**การสร้างถ้ำ:**
```
1. สร้าง Terrain ก้อนใหญ่ด้วย Add
2. ใช้ Subtract เพื่อเจาะรูภายใน
3. ปรับ Brush Size ให้เล็กลงสำหรับรายละเอียด
```

### 4.3 Paint Tool (ทาสีวัสดุ)

```
1. เลือก "Paint"
2. เลือก Material ที่ต้องการ
3. ทาลงบน Terrain
```

**Materials ที่มีให้เลือก:**
```
Nature:
- Grass         -- หญ้า
- Ground        -- ดิน
- Sand          -- ทราย
- Rock          -- หิน
- Sandstone     -- หินทราย
- Mud           -- โคลน
- Snow          -- หิมะ
- Ice           -- น้ำแข็ง
- LeafyGrass    -- หญ้าใบ
- SaltFlats     -- ทะเลเกลือ
- Limestone     -- หินปูน
- Pavement      -- ทางเดิน
- Basalt        -- หินบะซอลต์
- CrackedLava   -- ลาวาแตก
- Cobblestone   -- หินกรวด
- Asphalt       -- ยางมะตอย

Water:
- Water         -- น้ำ
- WoodPlanks    -- กระดานไม้
```

### 4.4 Smooth Tool (ทำเรียบ)

```
1. เลือก "Smooth"
2. ลากบนพื้นที่ที่ต้องการ
3. ลาก = ทำให้เรียบขึ้น
```

### 4.5 Flatten Tool (ทำแบน)

```
1. เลือก "Flatten"
2. คลิกที่ความสูงที่ต้องการ
3. ลากบนพื้นที่อื่น = ทำให้ได้ความสูงเดียวกัน
```

---

## 5. Terrain API ด้วยโค้ด

### 5.1 FillBlock

```lua
-- Script: TerrainAPI
-- วางใน: ServerScriptService

local terrain = workspace.Terrain

-- FillBlock: เติม Terrain เป็น Box
-- พารามิเตอร์: CFrame, Size, Material
terrain:FillBlock(
    CFrame.new(0, 0, 0),           -- ตำแหน่งและการหมุน
    Vector3.new(50, 20, 50),       -- ขนาด
    Enum.Material.Grass            -- วัสดุ
)

print("สร้าง Terrain Block เรียบร้อย!")
```

### 5.2 FillBall

```lua
-- FillBall: เติม Terrain เป็น Sphere
-- พารามิเตอร์: Center, Radius, Material
terrain:FillBall(
    Vector3.new(0, 10, 0),         -- ตำแหน่งศูนย์กลาง
    15,                             -- รัศมี
    Enum.Material.Rock             -- วัสดุ
)

-- เจาะรู (Subtract) โดยใช้ Air
terrain:FillBall(
    Vector3.new(0, 10, 0),
    10,
    Enum.Material.Air              -- Air = ลบ Terrain
)
```

### 5.3 FillCylinder

```lua
-- FillCylinder: เติม Terrain เป็น Cylinder
-- พารามิเตอร์: CFrame, Height, Radius, Material
terrain:FillCylinder(
    CFrame.new(0, 0, 0),
    30,                             -- ความสูง
    20,                             -- รัศมี
    Enum.Material.Ground
)
```

### 5.4 FillWedge

```lua
-- FillWedge: เติม Terrain เป็น Wedge
terrain:FillWedge(
    CFrame.new(0, 0, 50),
    Vector3.new(20, 15, 20),
    Enum.Material.Rock
)
```

### 5.5 ReplaceMaterial

```lua
-- แทนที่วัสดุทั้งหมดในพื้นที่
-- Region3: กำหนดพื้นที่ (min, max)
local region = Region3.new(
    Vector3.new(-50, -50, -50),    -- มุมล่างซ้าย
    Vector3.new(50, 50, 50)        -- มุมบนขวา
)

terrain:ReplaceMaterial(
    region,                         -- พื้นที่
    4,                              -- Resolution (1, 2, 4)
    Enum.Material.Grass,           -- วัสดุเดิม
    Enum.Material.Snow             -- วัสดุใหม่
)
```

### 5.6 ReadVoxels และ WriteVoxels

```lua
-- อ่านข้อมูล Voxel
local region = Region3.new(
    Vector3.new(-20, -10, -20),
    Vector3.new(20, 10, 20)
)

local resolution = 4  -- ขนาด Voxel (4 Studs)

local materials, occupancies = terrain:ReadVoxels(region, resolution)

print("Voxel data size:", #materials, #materials[1], #materials[1][1])

-- เขียนข้อมูล Voxel
-- (ต้องสร้าง materials และ occupancies tables ก่อน)
```

---

## 6. สร้าง Terrain ด้วยโค้ด (Procedural)

### 6.1 สร้าง Island

```lua
-- Script: GenerateIsland
-- วางใน: ServerScriptService

local terrain = workspace.Terrain

print("กำลังสร้าง Island...")

-- ล้าง Terrain เดิม
terrain:Clear()

-- ===== สร้าง Island =====
local islandRadius = 80
local islandHeight = 15

-- สร้างพื้นน้ำก่อน
terrain:FillBlock(
    CFrame.new(0, -15, 0),
    Vector3.new(500, 30, 500),
    Enum.Material.Water
)

-- สร้าง Island หลัก (ทรงกลม Flattened)
for angle = 0, 360, 5 do
    local rad = math.rad(angle)
    local x = math.cos(rad)
    local z = math.sin(rad)
    
    -- วงรอบนอกสุด
    for r = 0, islandRadius, 4 do
        local heightFactor = 1 - (r / islandRadius)  -- ยิ่งออกนอก ยิ่งต่ำ
        local height = islandHeight * heightFactor * heightFactor
        
        local pos = Vector3.new(x * r, height / 2, z * r)
        terrain:FillBall(pos, 8, Enum.Material.Grass)
    end
end

-- เพิ่มหาดทราย (ขอบ Island)
terrain:FillCylinder(
    CFrame.new(0, -1, 0),
    4,
    islandRadius + 5,
    Enum.Material.Sand
)

-- เพิ่มหิน (Rocks)
local rockPositions = {
    Vector3.new(30, 8, 20),
    Vector3.new(-25, 6, 35),
    Vector3.new(15, 5, -40),
    Vector3.new(-40, 7, -10),
}

for _, pos in ipairs(rockPositions) do
    terrain:FillBall(pos, math.random(5, 10), Enum.Material.Rock)
end

-- ภูเขาตรงกลาง
terrain:FillBall(Vector3.new(0, 20, 0), 25, Enum.Material.Rock)
terrain:FillBall(Vector3.new(0, 30, 0), 15, Enum.Material.Snow)

print("Island สร้างเรียบร้อย!")
```

### 6.2 สร้าง Valley (หุบเขา)

```lua
-- Script: GenerateValley
-- วางใน: ServerScriptService

local terrain = workspace.Terrain
terrain:Clear()

print("กำลังสร้าง Valley...")

-- ===== พื้นหุบเขา =====
terrain:FillBlock(
    CFrame.new(0, -5, 0),
    Vector3.new(200, 10, 200),
    Enum.Material.Grass
)

-- ===== ภูเขาด้านข้าง =====
local function createMountainRange(side)
    for z = -80, 80, 10 do
        local height = math.random(30, 60)
        local width = math.random(20, 35)
        
        local xPos = side * (50 + math.random(0, 20))
        
        terrain:FillBall(
            Vector3.new(xPos, height/2, z),
            width,
            Enum.Material.Rock
        )
        
        -- หิมะบนยอด
        if height > 45 then
            terrain:FillBall(
                Vector3.new(xPos, height - 5, z),
                width * 0.4,
                Enum.Material.Snow
            )
        end
    end
end

createMountainRange(1)   -- ขวา
createMountainRange(-1)  -- ซ้าย

-- ===== แม่น้ำกลาง =====
for z = -100, 100, 4 do
    local xOffset = math.sin(z * 0.05) * 10  -- คดเคี้ยว
    terrain:FillCylinder(
        CFrame.new(xOffset, 0, z),
        8, 8,
        Enum.Material.Water
    )
end

-- ===== ต้นไม้ (เป็น Cylinder+Ball) =====
for i = 1, 20 do
    local x = math.random(-40, 40)
    local z = math.random(-80, 80)
    
    -- ลำต้น
    terrain:FillCylinder(
        CFrame.new(x, 4, z),
        8, 1.5,
        Enum.Material.Ground
    )
    
    -- ยอด
    terrain:FillBall(Vector3.new(x, 10, z), 5, Enum.Material.Grass)
end

print("Valley สร้างเรียบร้อย!")
```

### 6.3 สร้าง Dungeon

```lua
-- Script: GenerateDungeon
-- วางใน: ServerScriptService

local terrain = workspace.Terrain
terrain:Clear()

print("กำลังสร้าง Dungeon...")

-- ===== สร้าง Dungeon ด้วย Rock =====
local dungeonSize = 100
local dungeonHeight = 20

-- เติม Solid Rock ก่อน
terrain:FillBlock(
    CFrame.new(0, dungeonHeight/2, 0),
    Vector3.new(dungeonSize, dungeonHeight, dungeonSize),
    Enum.Material.Rock
)

-- ===== เจาะ Corridors =====
local function createCorridor(start, endPos, width, height)
    local direction = (endPos - start).Unit
    local length = (endPos - start).Magnitude
    
    for t = 0, length, 4 do
        local pos = start + direction * t
        terrain:FillBall(
            pos + Vector3.new(0, height/2, 0),
            width,
            Enum.Material.Air  -- Air = เจาะ
        )
    end
end

-- Corridors หลัก
createCorridor(Vector3.new(-45, 5, 0), Vector3.new(45, 5, 0), 5, 8)   -- แนวนอน
createCorridor(Vector3.new(0, 5, -45), Vector3.new(0, 5, 45), 5, 8)   -- แนวตั้ง

-- ห้อง
local rooms = {
    {Vector3.new(0, 5, 0), 15},     -- ห้องกลาง
    {Vector3.new(40, 5, 0), 10},    -- ห้องขวา
    {Vector3.new(-40, 5, 0), 10},   -- ห้องซ้าย
    {Vector3.new(0, 5, 40), 10},    -- ห้องหน้า
    {Vector3.new(0, 5, -40), 10},   -- ห้องหลัง
}

for _, room in ipairs(rooms) do
    terrain:FillBall(room[1], room[2], Enum.Material.Air)
    -- เพิ่มพื้น
    terrain:FillBall(room[1] - Vector3.new(0, room[2] - 2, 0), room[2] * 0.8, Enum.Material.Cobblestone)
end

print("Dungeon สร้างเรียบร้อย!")
```

---

## 7. Water System

### 7.1 สร้าง Ocean

```lua
-- Script: CreateOcean
-- วางใน: ServerScriptService

local terrain = workspace.Terrain

-- ตั้งค่า Water
terrain.WaterWaveSize = 0.15        -- ขนาดคลื่น
terrain.WaterWaveSpeed = 10         -- ความเร็วคลื่น
terrain.WaterColor = Color3.fromRGB(20, 100, 170)  -- สีน้ำ
terrain.WaterTransparency = 0.3    -- ความใส

-- สร้าง Ocean (ใต้ระดับ Baseplate)
terrain:FillBlock(
    CFrame.new(0, -25, 0),
    Vector3.new(1000, 50, 1000),
    Enum.Material.Water
)

print("Ocean สร้างเรียบร้อย!")
```

### 7.2 สร้าง River

```lua
-- Script: CreateRiver
-- วางใน: ServerScriptService

local terrain = workspace.Terrain

-- สร้างแม่น้ำคดเคี้ยว
local riverLength = 200
local riverWidth = 8
local riverDepth = 6

for t = 0, riverLength, 4 do
    local xOffset = math.sin(t * 0.03) * 20  -- คดเคี้ยวใน X
    local zPos = t - riverLength/2
    
    -- ขุดร่อง
    terrain:FillBall(
        Vector3.new(xOffset, -riverDepth/2, zPos),
        riverWidth,
        Enum.Material.Air
    )
    
    -- เติมน้ำ
    terrain:FillBall(
        Vector3.new(xOffset, 0, zPos),
        riverWidth * 0.8,
        Enum.Material.Water
    )
end

print("River สร้างเรียบร้อย!")
```

---

## 8. Terrain Materials ทั้งหมด

### 8.1 รายการ Materials

```lua
-- Materials ทั้งหมดของ Terrain
local terrainMaterials = {
    -- Ground
    Enum.Material.Grass,
    Enum.Material.Ground,
    Enum.Material.Sand,
    Enum.Material.Sandstone,
    Enum.Material.Rock,
    Enum.Material.Mud,
    Enum.Material.Snow,
    Enum.Material.Ice,
    Enum.Material.Glacier,
    Enum.Material.LeafyGrass,
    Enum.Material.SaltFlats,
    Enum.Material.Limestone,
    Enum.Material.Pavement,
    Enum.Material.Asphalt,
    Enum.Material.CrackedLava,
    Enum.Material.Cobblestone,
    Enum.Material.Basalt,
    Enum.Material.Concrete,
    Enum.Material.Gravel,
    Enum.Material.Woodplanks,
    
    -- Liquid
    Enum.Material.Water,
    
    -- Special
    Enum.Material.Air,  -- ลบ Terrain
}

-- ตัวอย่างการแสดงทุก Material
for _, mat in ipairs(terrainMaterials) do
    print(mat)
end
```

### 8.2 Material Combinations

```lua
-- ตัวอย่างการใช้ Materials ร่วมกัน
local terrain = workspace.Terrain

-- สนามหญ้ากับทางเดิน
terrain:FillBlock(CFrame.new(0, 0, 0), Vector3.new(100, 2, 100), Enum.Material.Grass)
terrain:FillBlock(CFrame.new(0, 0.5, 0), Vector3.new(5, 2, 100), Enum.Material.Pavement)  -- ทางเดินกลาง

-- ภูเขาหินพร้อมหิมะ
terrain:FillBall(Vector3.new(50, 0, 50), 30, Enum.Material.Rock)
terrain:FillBall(Vector3.new(50, 20, 50), 15, Enum.Material.Snow)

-- ชายหาด
terrain:FillBlock(CFrame.new(0, -5, 0), Vector3.new(100, 10, 100), Enum.Material.Sand)
terrain:FillBlock(CFrame.new(0, -5, 70), Vector3.new(100, 10, 60), Enum.Material.Water)
```

---

## 9. Terrain Performance

### 9.1 Terrain LOD

Terrain มี Level of Detail (LOD) อัตโนมัติ:
- ใกล้ = คุณภาพสูง
- ไกล = คุณภาพต่ำ (ประหยัด Performance)

### 9.2 Region3 สำหรับ Operations

```lua
-- ใช้ Region3 กำหนดพื้นที่ Terrain
local region = Region3.new(
    Vector3.new(-100, -50, -100),  -- min corner
    Vector3.new(100, 50, 100)      -- max corner
)

-- ขนาด Region
local size = region.Size
print("Region size:", size)

-- ศูนย์กลาง
local center = region.CFrame
print("Region center:", center)
```

### 9.3 Terrain Streaming

```
workspace.StreamingEnabled = true
-- ทำให้โหลด Terrain และ Parts เฉพาะส่วนที่ผู้เล่นใกล้
-- ลด Memory และเพิ่ม Performance สำหรับ Map ขนาดใหญ่
```

---

## 10. ตัวอย่างโปรเจกต์สมบูรณ์: Forest Map

```lua
-- Script: GenerateForestMap
-- วางใน: ServerScriptService

local terrain = workspace.Terrain
local RunService = game:GetService("RunService")

print("กำลังสร้าง Forest Map...")
print("กรุณารอสักครู่...")

-- ล้าง Terrain เดิม
terrain:Clear()

-- ===== ตั้งค่าเบื้องต้น =====
local MAP_SIZE = 300        -- ขนาด Map
local MAX_HEIGHT = 40       -- ความสูงสูงสุด
local TREE_COUNT = 50       -- จำนวนต้นไม้

-- ===== 1. สร้างพื้นดินหลัก =====
terrain:FillBlock(
    CFrame.new(0, -10, 0),
    Vector3.new(MAP_SIZE, 20, MAP_SIZE),
    Enum.Material.Ground
)

-- ===== 2. สร้างพื้นที่ราบ =====
terrain:FillBlock(
    CFrame.new(0, 0, 0),
    Vector3.new(MAP_SIZE * 0.6, 4, MAP_SIZE * 0.6),
    Enum.Material.Grass
)

-- ===== 3. สร้างเนินเขารอบๆ =====
math.randomseed(12345)  -- Seed สำหรับ Reproducible

for i = 1, 30 do
    local angle = math.random() * math.pi * 2
    local distance = math.random(60, 130)
    local x = math.cos(angle) * distance
    local z = math.sin(angle) * distance
    local height = math.random(20, MAX_HEIGHT)
    local radius = math.random(20, 40)
    
    -- เนิน
    terrain:FillBall(
        Vector3.new(x, height * 0.3, z),
        radius,
        Enum.Material.Grass
    )
    
    -- ยอดเขา
    terrain:FillBall(
        Vector3.new(x, height * 0.7, z),
        radius * 0.5,
        Enum.Material.Rock
    )
    
    -- หิมะบนยอด (ถ้าสูงพอ)
    if height > 35 then
        terrain:FillBall(
            Vector3.new(x, height, z),
            radius * 0.25,
            Enum.Material.Snow
        )
    end
end

-- ===== 4. สร้างแม่น้ำ =====
for t = -150, 150, 4 do
    local xOffset = math.sin(t * 0.02) * 30 + math.cos(t * 0.01) * 10
    
    -- ขุดร่อง
    terrain:FillBall(
        Vector3.new(xOffset, -2, t),
        6,
        Enum.Material.Air
    )
    
    -- เติมน้ำ
    terrain:FillBall(
        Vector3.new(xOffset, 0, t),
        5,
        Enum.Material.Water
    )
end

-- ===== 5. สร้างทางเดิน (Path) =====
for t = -100, 100, 4 do
    local xOffset = math.sin(t * 0.03) * 5  -- คดเล็กน้อย
    terrain:FillCylinder(
        CFrame.new(xOffset, 1, t),
        5, 2.5,
        Enum.Material.Pavement
    )
end

-- ===== 6. เพิ่มหิน =====
for i = 1, 25 do
    local x = math.random(-80, 80)
    local z = math.random(-80, 80)
    
    -- ไม่ให้ทับทางเดิน
    if math.abs(x) > 8 then
        local size = math.random(3, 8)
        terrain:FillBall(
            Vector3.new(x, size * 0.5, z),
            size,
            Enum.Material.Rock
        )
    end
end

-- ===== 7. ตั้งค่า Water =====
terrain.WaterWaveSize = 0.2
terrain.WaterWaveSpeed = 15
terrain.WaterColor = Color3.fromRGB(40, 120, 180)
terrain.WaterTransparency = 0.2

-- ===== 8. ตั้งค่า Lighting =====
local Lighting = game:GetService("Lighting")
Lighting.Brightness = 1.5
Lighting.ClockTime = 10  -- เช้า
Lighting.GlobalShadows = true

-- Atmosphere
local atm = Lighting:FindFirstChildOfClass("Atmosphere")
if not atm then atm = Instance.new("Atmosphere", Lighting) end
atm.Density = 0.2
atm.Haze = 0.1
atm.Color = Color3.fromRGB(180, 200, 150)

print("Forest Map สร้างเรียบร้อย!")
print("Map Size:", MAP_SIZE .. "x" .. MAP_SIZE .. " Studs")
```

---

## 📚 แบบฝึกหัดตอนที่ 8

### แบบฝึกหัดที่ 1: Terrain Editor
1. เปิด Terrain Editor
2. ใช้ Generate สร้าง Terrain อัตโนมัติ
3. ลอง Biome ต่างๆ

### แบบฝึกหัดที่ 2: Edit Tools
1. ทดลองใช้ Add, Subtract, Paint, Smooth, Flatten
2. สร้างเนินเขาเล็กๆ
3. เพิ่มแม่น้ำด้วย Water material

### แบบฝึกหัดที่ 3: Terrain API
1. สร้าง Script ที่ใช้ FillBlock
2. ลอง FillBall และ FillCylinder
3. ลอง FillBall ด้วย Air เพื่อเจาะรู

### แบบฝึกหัดที่ 4: Island
1. คัดลอกโค้ด GenerateIsland
2. รันและดูผลลัพธ์
3. แก้ไขขนาดและรูปร่าง

### แบบฝึกหัดที่ 5: Forest Map
1. คัดลอกโค้ด GenerateForestMap
2. รันและสำรวจ Map ที่สร้าง
3. ปรับแต่ง Parameters ต่างๆ

---

## 💡 เคล็ดลับ

1. **ใช้ Smooth หลัง Add** - ทำให้ภูมิประเทศดูเป็นธรรมชาติ
2. **Layer หลายๆ Material** - เพิ่มความสมจริง
3. **Reference ภาพจริง** - ดูภาพภูมิประเทศจริงเพื่อแรงบันดาล
4. **Test Performance** - Terrain ขนาดใหญ่อาจช้าได้

---

## ⏭️ ตอนถัดไป

ในตอนที่ 9 เราจะเรียนรู้:
- Lua Programming เบื้องต้น
- Variables, Data Types
- Syntax ของ Lua
- โค้ดแรกของคุณ

---

*ตอนที่ 8/100 | ระดับ: พื้นฐาน | เวลา: 90-120 นาที*
