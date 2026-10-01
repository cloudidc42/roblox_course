# ตอนที่ 4: Basic Parts และ Models
## Part 4: Working with Basic Parts and Models

---

**ระดับ:** พื้นฐาน (Beginner)  
**เวลาเรียน:** 90-120 นาที  
**ข้อกำหนดเบื้องต้น:** ตอนที่ 1-3

---

## 🎯 วัตถุประสงค์การเรียนรู้

เมื่อเรียนจบตอนนี้ คุณจะสามารถ:
1. สร้างและจัดการ Parts ประเภทต่างๆ
2. เข้าใจ CFrame, Position, Rotation และ Size
3. ใช้ Transform Tools (Move, Scale, Rotate)
4. จัดกลุ่ม Objects เป็น Models
5. ใช้ Union Operations
6. สร้างโครงสร้างง่ายๆ ด้วย Parts

---

## 1. Part คืออะไร?

### 1.1 ภาพรวม

**Part** คือ Building Block พื้นฐานของ Roblox - เป็น 3D Object ที่:
- มีรูปร่าง (Shape)
- มีขนาด (Size)
- มีตำแหน่ง (Position)
- มีการหมุน (Rotation)
- มีวัสดุ (Material)
- มีสี (Color)

```lua
-- ตัวอย่าง Part ง่ายๆ
local part = Instance.new("Part")
part.Size = Vector3.new(4, 4, 4)
part.Position = Vector3.new(0, 5, 0)
part.Parent = workspace
```

### 1.2 ประเภทของ Parts

Roblox มี Part หลายประเภท:

| ประเภท | Shape | Class Name |
|--------|-------|-----------|
| Block | สี่เหลี่ยม | Part |
| Sphere | ทรงกลม | Part (Shape = Ball) |
| Cylinder | ทรงกระบอก | Part (Shape = Cylinder) |
| Wedge | ลิ่ม | WedgePart |
| Corner Wedge | มุมลิ่ม | CornerWedgePart |
| Truss | โครงสร้างสะพาน | TrussPart |
| Mesh Part | รูปทรง Custom | MeshPart |

---

## 2. การสร้าง Parts

### 2.1 สร้างด้วย Menu

```
Insert > Part > Block Part  (หรือ Sphere, Cylinder, etc.)
```

### 2.2 สร้างด้วย Toolbar

```
Home > Insert > Part > เลือกประเภท
```

### 2.3 สร้างด้วยโค้ด

```lua
-- สร้าง Block Part
local block = Instance.new("Part")
block.Name = "MyBlock"
block.Size = Vector3.new(4, 4, 4)
block.Position = Vector3.new(0, 5, 0)
block.Anchored = true
block.Parent = workspace

-- สร้าง Sphere Part
local sphere = Instance.new("Part")
sphere.Name = "MySphere"
sphere.Shape = Enum.PartType.Ball  -- เปลี่ยนเป็น Sphere
sphere.Size = Vector3.new(4, 4, 4)
sphere.Position = Vector3.new(5, 5, 0)
sphere.BrickColor = BrickColor.new("Bright red")
sphere.Anchored = true
sphere.Parent = workspace

-- สร้าง Cylinder Part
local cylinder = Instance.new("Part")
cylinder.Name = "MyCylinder"
cylinder.Shape = Enum.PartType.Cylinder
cylinder.Size = Vector3.new(1, 6, 6)  -- Cylinder ด้านยาวเป็น X
cylinder.Position = Vector3.new(-5, 5, 0)
cylinder.BrickColor = BrickColor.new("Bright blue")
cylinder.Anchored = true
cylinder.Parent = workspace

-- สร้าง Wedge Part
local wedge = Instance.new("WedgePart")
wedge.Name = "MyWedge"
wedge.Size = Vector3.new(4, 4, 4)
wedge.Position = Vector3.new(0, 5, 5)
wedge.BrickColor = BrickColor.new("Bright green")
wedge.Anchored = true
wedge.Parent = workspace

print("สร้าง Parts ทั้งหมดเรียบร้อย!")
```

---

## 3. Transform Tools

### 3.1 Select Tool (V)

**Select Tool** ใช้เลือก Objects:
```
- คลิกซ้ายที่ Object เพื่อเลือก
- Ctrl+คลิก เพื่อเพิ่มใน Selection
- Drag เพื่อเลือกหลาย Objects พร้อมกัน
- Escape เพื่อยกเลิก Selection
```

### 3.2 Move Tool (G)

**Move Tool** ใช้ย้าย Objects:

```
- กด G หรือเลือกจาก Toolbar
- ลาก Arrows (สีแดง = X, เขียว = Y, น้ำเงิน = Z)
- Ctrl+คลิก เพื่อย้ายหลาย Objects
```

**Snap Settings (การ Snap):**
```
Snap เปิด: Objects จะ Snap ตาม Grid
Snap ปิด: เคลื่อนไหวอิสระ

กด , เพื่อลด Snap Size
กด . เพื่อเพิ่ม Snap Size
```

### 3.3 Scale Tool (R - Resize)

**Scale Tool** ใช้ปรับขนาด:
```
- ลาก Corner Handles เพื่อ Resize ทุกทิศ
- ลาก Face Handles เพื่อ Resize ทิศเดียว
- Alt+Drag เพื่อ Resize จากศูนย์กลาง
```

### 3.4 Rotate Tool (R - Rotate)

**Rotate Tool** ใช้หมุน:
```
- ลาก Arc (วงกลม) เพื่อหมุนรอบแกน
- สีแดง = หมุนรอบแกน X
- สีเขียว = หมุนรอบแกน Y
- สีน้ำเงิน = หมุนรอบแกน Z
```

---

## 4. Position และ CFrame

### 4.1 Position

```lua
-- Position คือตำแหน่งของ Part ในโลก 3D
-- Vector3(X, Y, Z)

local part = Instance.new("Part")

-- ตั้งตำแหน่งตรงกลาง ที่ความสูง 5
part.Position = Vector3.new(0, 5, 0)

-- ย้าย Part ไปทางขวา 10 หน่วย
part.Position = Vector3.new(10, 5, 0)

-- ย้าย Part ขึ้น 5 หน่วยจากตำแหน่งเดิม
part.Position = part.Position + Vector3.new(0, 5, 0)

print("Position:", part.Position)
```

### 4.2 CFrame

**CFrame** (Coordinate Frame) คือการรวมของ Position และ Rotation:

```lua
-- CFrame ง่ายๆ
local part = Instance.new("Part")
part.Anchored = true
part.Parent = workspace

-- ตั้ง CFrame ด้วย Position เท่านั้น
part.CFrame = CFrame.new(0, 5, 0)

-- ตั้ง CFrame ด้วย Position + Rotation (Euler angles)
part.CFrame = CFrame.new(0, 5, 0) * CFrame.Angles(0, math.pi/4, 0)
-- หมุน 45 องศา รอบแกน Y

-- CFrame.lookAt - หันหน้าไปทิศทางที่ต้องการ
local targetPos = Vector3.new(10, 0, 0)
local eyePos = Vector3.new(0, 5, 0)
part.CFrame = CFrame.lookAt(eyePos, targetPos)

-- การ Offset CFrame
part.CFrame = part.CFrame + Vector3.new(5, 0, 0)
-- ย้าย Part 5 หน่วยไปทางขวา (World Space)

-- Relative Offset
part.CFrame = part.CFrame * CFrame.new(5, 0, 0)
-- ย้าย Part 5 หน่วยไปข้างหน้า (Local Space)
```

### 4.3 ความแตกต่างระหว่าง Position และ CFrame

```lua
-- Position: เปลี่ยนตำแหน่งเท่านั้น ไม่กระทบ Rotation
part.Position = Vector3.new(10, 5, 0)

-- CFrame: เปลี่ยนทั้ง Position และ Rotation
part.CFrame = CFrame.new(10, 5, 0)

-- เมื่อ Part มี Rotation อยู่แล้ว:
-- ใช้ Position จะรักษา Rotation เดิม
-- ใช้ CFrame.new() จะ Reset Rotation

-- วิธีย้าย Part โดยคงการ Rotate:
local offset = Vector3.new(0, 5, 0)
part.CFrame = part.CFrame + offset
```

---

## 5. Size และ Scale

### 5.1 Size

```lua
-- Size คือขนาดของ Part ใน Studs
-- Vector3(Width, Height, Depth)

local part = Instance.new("Part")
part.Anchored = true

-- ขนาด 4x4x4
part.Size = Vector3.new(4, 4, 4)

-- ขนาดแบบ Rectangle
part.Size = Vector3.new(8, 2, 4)  -- กว้าง 8, สูง 2, ลึก 4

-- ขนาดของ Cylinder (x = ความยาว, y = radius, z = radius)
local cylinder = Instance.new("Part")
cylinder.Shape = Enum.PartType.Cylinder
cylinder.Size = Vector3.new(10, 2, 2)  -- ยาว 10, เส้นผ่านศูนย์กลาง 2

part.Parent = workspace
cylinder.Parent = workspace
```

### 5.2 Minimum Size

```lua
-- Parts ต้องมีขนาดขั้นต่ำ 0.05 studs ในแต่ละมิติ
local part = Instance.new("Part")
part.Size = Vector3.new(0.05, 0.05, 0.05)  -- ขนาดเล็กที่สุด

-- ถ้าตั้งน้อยกว่า 0.05 จะถูก Clamp อัตโนมัติ
```

---

## 6. Material และ Color

### 6.1 Materials

```lua
local part = Instance.new("Part")
part.Anchored = true
part.Size = Vector3.new(4, 4, 4)
part.Parent = workspace

-- Material Types
part.Material = Enum.Material.SmoothPlastic  -- พลาสติกเรียบ (ค่าเริ่มต้น)
part.Material = Enum.Material.Plastic        -- พลาสติก
part.Material = Enum.Material.Wood           -- ไม้
part.Material = Enum.Material.WoodPlanks     -- กระดานไม้
part.Material = Enum.Material.Metal          -- โลหะ
part.Material = Enum.Material.DiamondPlate  -- แผ่นเพชร
part.Material = Enum.Material.Cobblestone   -- หินกรวด
part.Material = Enum.Material.Brick         -- อิฐ
part.Material = Enum.Material.Concrete      -- คอนกรีต
part.Material = Enum.Material.Grass         -- หญ้า
part.Material = Enum.Material.Ground        -- ดิน
part.Material = Enum.Material.Sand          -- ทราย
part.Material = Enum.Material.Rock          -- หิน
part.Material = Enum.Material.Glass         -- กระจก
part.Material = Enum.Material.Ice           -- น้ำแข็ง
part.Material = Enum.Material.Neon          -- นีออน (เรืองแสง)
part.Material = Enum.Material.ForceField    -- Field ป้องกัน
```

### 6.2 Color

```lua
-- วิธีที่ 1: BrickColor (Classic colors)
part.BrickColor = BrickColor.new("Bright red")
part.BrickColor = BrickColor.new("Bright blue")
part.BrickColor = BrickColor.new("Bright green")
part.BrickColor = BrickColor.new("White")
part.BrickColor = BrickColor.new("Black")

-- วิธีที่ 2: Color3 (RGB)
part.Color = Color3.new(1, 0, 0)              -- แดง (RGB: 255, 0, 0)
part.Color = Color3.new(0, 1, 0)              -- เขียว
part.Color = Color3.new(0, 0, 1)              -- น้ำเงิน
part.Color = Color3.fromRGB(255, 165, 0)      -- ส้ม
part.Color = Color3.fromRGB(128, 0, 128)      -- ม่วง

-- วิธีที่ 3: Color3.fromHSV
part.Color = Color3.fromHSV(0.5, 1, 1)        -- ฟ้าสด

-- ตัวอย่างใช้งานจริง
part.BrickColor = BrickColor.new("Bright red")
part.Material = Enum.Material.Metal
```

### 6.3 BrickColor ที่นิยม

```lua
-- สีที่ใช้บ่อย
local colors = {
    "Bright red",
    "Bright blue", 
    "Bright green",
    "Bright yellow",
    "Bright orange",
    "Bright violet",
    "White",
    "Black",
    "Medium stone grey",
    "Dark stone grey",
    "Reddish brown",  -- น้ำตาล
    "Sand green",     -- เขียวทราย
    "Gold",           -- ทอง
    "Silver"          -- เงิน
}
```

---

## 7. Part Properties เพิ่มเติม

### 7.1 Physics Properties

```lua
local part = Instance.new("Part")
part.Anchored = true              -- ยึดติด ไม่ตกหรือเคลื่อน
part.CanCollide = true            -- ชนกับ Objects อื่นได้
part.CanTouch = true              -- Trigger Touch Events
part.CanQuery = true              -- Raycast ผ่านได้

-- Mass
part.CustomPhysicalProperties = PhysicalProperties.new(
    1,      -- Density (ความหนาแน่น)
    0.3,    -- Friction (แรงเสียดทาน)
    0.5,    -- Elasticity (ความยืดหยุ่น)
    0,      -- FrictionWeight
    0       -- ElasticityWeight
)

-- Part ที่ไม่ Anchored จะตกลงมาตาม Physics
local fallingPart = Instance.new("Part")
fallingPart.Anchored = false  -- จะตกลงมา!
fallingPart.Position = Vector3.new(0, 20, 0)
fallingPart.Parent = workspace
```

### 7.2 Visual Properties

```lua
-- Transparency (0 = ทึบ, 1 = ใส)
part.Transparency = 0          -- ทึบสนิท
part.Transparency = 0.5        -- ใสครึ่งนึง
part.Transparency = 1          -- ใสสนิท (มองไม่เห็น)

-- Reflectance (0 = ไม่สะท้อน, 1 = สะท้อนมาก)
part.Reflectance = 0           -- ไม่สะท้อน
part.Reflectance = 0.5         -- สะท้อนพอประมาณ
part.Reflectance = 1           -- สะท้อนสูงสุด

-- CastShadow
part.CastShadow = true         -- ทำเงา
part.CastShadow = false        -- ไม่ทำเงา (ดีสำหรับ Performance)
```

---

## 8. Models

### 8.1 Model คืออะไร?

**Model** คือ Container ที่รวม Parts และ Instances อื่นๆ เข้าด้วยกัน:

```
Model
├── Part 1
├── Part 2
├── Part 3
└── Script (optional)
```

### 8.2 การสร้าง Model

**ด้วย GUI:**
```
1. Select Parts ที่ต้องการ
2. คลิกขวา > Group (Ctrl+G)
หรือ Model > Group Objects (Ctrl+G)
```

**ด้วยโค้ด:**
```lua
-- สร้าง Model ด้วยโค้ด
local model = Instance.new("Model")
model.Name = "MyHouse"
model.Parent = workspace

-- สร้าง Parts ภายใน Model
local floor = Instance.new("Part")
floor.Name = "Floor"
floor.Size = Vector3.new(10, 1, 10)
floor.Position = Vector3.new(0, 0.5, 0)
floor.Anchored = true
floor.BrickColor = BrickColor.new("Reddish brown")
floor.Parent = model  -- ← วางใน Model

local wall1 = Instance.new("Part")
wall1.Name = "Wall_Left"
wall1.Size = Vector3.new(1, 6, 10)
wall1.Position = Vector3.new(-5.5, 3.5, 0)
wall1.Anchored = true
wall1.BrickColor = BrickColor.new("White")
wall1.Parent = model

local wall2 = Instance.new("Part")
wall2.Name = "Wall_Right"
wall2.Size = Vector3.new(1, 6, 10)
wall2.Position = Vector3.new(5.5, 3.5, 0)
wall2.Anchored = true
wall2.BrickColor = BrickColor.new("White")
wall2.Parent = model

print("สร้าง Model MyHouse เรียบร้อย!")
```

### 8.3 PrimaryPart

**PrimaryPart** คือ Reference Part หลักของ Model:

```lua
-- กำหนด PrimaryPart
local model = workspace.MyHouse
model.PrimaryPart = workspace.MyHouse.Floor

-- ใช้ SetPrimaryPartCFrame เพื่อย้าย Model ทั้งหมด
model:SetPrimaryPartCFrame(CFrame.new(10, 0, 0))
-- ย้าย Model ทั้งหมดไปที่ตำแหน่ง (10, 0, 0)

-- ย้าย Model ไปทิศทางที่ต้องการ
model:SetPrimaryPartCFrame(
    CFrame.new(10, 0, 10) * CFrame.Angles(0, math.pi/2, 0)
)
-- ย้ายและหมุน 90 องศา
```

### 8.4 Model:GetExtentsSize()

```lua
-- ขนาดรวมของ Model
local model = workspace.MyHouse
local extents = model:GetExtentsSize()
print("Model size:", extents)
-- แสดงขนาด Bounding Box ของ Model

-- ใช้หาตรงกลางของ Model
local center = model:GetPivot()
print("Model center:", center)
```

---

## 9. Union Operations

### 9.1 Union คืออะไร?

Union คือการรวม Parts เข้าด้วยกันเป็น Object เดียว:

```
Union = Part A + Part B = Object ใหม่
```

### 9.2 วิธีทำ Union

```
1. Select Parts ที่ต้องการ Union
2. Model > Union (Ctrl+Shift+G)
หรือ คลิกขวา > Union
```

### 9.3 Negate (เจาะรู)

```
1. Select Part ที่จะเป็น "Hole"
2. Model > Negate
3. Select Part ที่จะถูกเจาะ + Negated Part
4. Model > Union
```

ตัวอย่าง:
```
ตัวอย่าง: ทำหน้าต่างในกำแพง
1. สร้าง Wall (Part ใหญ่)
2. สร้าง Window Hole (Part เล็ก ทับกำแพง)
3. Negate Window Hole
4. Union Wall + Window Hole
5. ได้กำแพงที่มีรูหน้าต่าง
```

### 9.4 ข้อดี/ข้อเสียของ Union

```
ข้อดี:
✅ ลดจำนวน Parts (ดีสำหรับ Performance)
✅ สร้างรูปร่างซับซ้อนได้
✅ จัดการง่ายขึ้น

ข้อเสีย:
❌ ไม่สามารถแก้ไขทีหลังได้ง่าย
❌ ใช้ Compute มากกว่า Part เดี่ยว
❌ มีข้อจำกัดด้าน Collision
```

---

## 10. Anchoring และ Physics

### 10.1 Anchored Parts

```lua
-- Anchored = true: Part ไม่เคลื่อนไหว (ตามฟิสิกส์)
local wall = Instance.new("Part")
wall.Anchored = true  -- ยึดติด
wall.Size = Vector3.new(1, 10, 10)
wall.Position = Vector3.new(0, 5, 0)
wall.Parent = workspace

-- Anchored = false: Part ตกลงมาตามแรงโน้มถ่วง
local ball = Instance.new("Part")
ball.Anchored = false  -- ไม่ยึดติด จะตก!
ball.Shape = Enum.PartType.Ball
ball.Size = Vector3.new(2, 2, 2)
ball.Position = Vector3.new(0, 20, 0)
ball.Parent = workspace
```

### 10.2 CanCollide

```lua
-- CanCollide = true: ชนกับ Objects อื่น
-- CanCollide = false: ทะลุผ่านได้

local part = Instance.new("Part")
part.Anchored = true
part.CanCollide = false  -- ผู้เล่นเดินทะลุได้
part.Transparency = 0.5  -- มองเห็นแต่ทะลุได้
part.Parent = workspace

-- ใช้กับ Kill Zones หรือ Trigger Areas
local killZone = Instance.new("Part")
killZone.Anchored = true
killZone.CanCollide = false
killZone.CanTouch = true  -- แต่ Trigger Touch Event ได้
killZone.Transparency = 0.7
killZone.BrickColor = BrickColor.new("Bright red")
killZone.Parent = workspace
```

---

## 11. Decals และ Textures

### 11.1 Decal (ภาพบน Face ของ Part)

```lua
-- ใส่ภาพบน Part
local part = Instance.new("Part")
part.Size = Vector3.new(5, 5, 1)
part.Anchored = true
part.Parent = workspace

-- สร้าง Decal
local decal = Instance.new("Decal")
decal.Texture = "rbxassetid://1234567"  -- ID ของ Image
decal.Face = Enum.NormalId.Front        -- ใส่ที่ Face ด้านหน้า
decal.Parent = part

-- Faces ที่มีให้เลือก:
-- Enum.NormalId.Front
-- Enum.NormalId.Back
-- Enum.NormalId.Left
-- Enum.NormalId.Right
-- Enum.NormalId.Top
-- Enum.NormalId.Bottom
```

### 11.2 Texture (ลายที่ซ้ำ)

```lua
-- Texture จะ Tile ซ้ำบน Part
local texture = Instance.new("Texture")
texture.Texture = "rbxassetid://1234567"
texture.StudsPerTileU = 4  -- ความถี่แนวนอน
texture.StudsPerTileV = 4  -- ความถี่แนวตั้ง
texture.Face = Enum.NormalId.Top
texture.Parent = part
```

---

## 12. SpecialMesh

### 12.1 การใช้ SpecialMesh

```lua
-- SpecialMesh เปลี่ยนรูปทรงของ Part
local part = Instance.new("Part")
part.Anchored = true
part.Parent = workspace

local mesh = Instance.new("SpecialMesh")
mesh.MeshType = Enum.MeshType.Sphere      -- ทรงกลม
mesh.MeshType = Enum.MeshType.Cylinder    -- ทรงกระบอก  
mesh.MeshType = Enum.MeshType.Brick       -- เหมือน Block ปกติ
mesh.MeshType = Enum.MeshType.Wedge      -- ลิ่ม
mesh.MeshType = Enum.MeshType.Torso      -- รูปทรงตัว
mesh.MeshType = Enum.MeshType.Head       -- รูปทรงหัว
mesh.MeshType = Enum.MeshType.FileMesh   -- Custom Mesh (ต้องใส่ MeshId)

mesh.Scale = Vector3.new(2, 2, 2)  -- ขยาย Mesh 2 เท่า
mesh.Parent = part
```

---

## 13. ตัวอย่างโปรเจกต์: สร้างบ้านง่ายๆ

```lua
-- Script: BuildSimpleHouse
-- วางใน: ServerScriptService

-- ===== สร้างบ้านง่ายๆ =====

local function createHouse(position)
    local house = Instance.new("Model")
    house.Name = "SimpleHouse"
    house.Parent = workspace
    
    -- ===== พื้น (Floor) =====
    local floor = Instance.new("Part")
    floor.Name = "Floor"
    floor.Size = Vector3.new(14, 0.5, 12)
    floor.CFrame = CFrame.new(position + Vector3.new(0, 0.25, 0))
    floor.BrickColor = BrickColor.new("Reddish brown")
    floor.Material = Enum.Material.WoodPlanks
    floor.Anchored = true
    floor.Parent = house
    
    -- ===== กำแพง (Walls) =====
    local wallData = {
        -- {ชื่อ, ขนาด, ตำแหน่ง offset}
        {"Wall_Front", Vector3.new(14, 6, 0.5), Vector3.new(0, 3.25, -6)},
        {"Wall_Back",  Vector3.new(14, 6, 0.5), Vector3.new(0, 3.25, 6)},
        {"Wall_Left",  Vector3.new(0.5, 6, 12), Vector3.new(-7, 3.25, 0)},
        {"Wall_Right", Vector3.new(0.5, 6, 12), Vector3.new(7, 3.25, 0)},
    }
    
    for _, data in ipairs(wallData) do
        local wall = Instance.new("Part")
        wall.Name = data[1]
        wall.Size = data[2]
        wall.CFrame = CFrame.new(position + data[3])
        wall.BrickColor = BrickColor.new("White")
        wall.Material = Enum.Material.SmoothPlastic
        wall.Anchored = true
        wall.Parent = house
    end
    
    -- ===== หลังคา (Roof) =====
    local roof = Instance.new("WedgePart")
    roof.Name = "Roof"
    roof.Size = Vector3.new(15, 4, 7)
    roof.CFrame = CFrame.new(position + Vector3.new(0, 8.5, -3.5))
    roof.BrickColor = BrickColor.new("Bright red")
    roof.Material = Enum.Material.Brick
    roof.Anchored = true
    roof.Parent = house
    
    local roof2 = Instance.new("WedgePart")
    roof2.Name = "Roof2"
    roof2.Size = Vector3.new(15, 4, 7)
    -- หมุน 180 องศาสำหรับอีกด้าน
    roof2.CFrame = CFrame.new(position + Vector3.new(0, 8.5, 3.5)) * CFrame.Angles(0, math.pi, 0)
    roof2.BrickColor = BrickColor.new("Bright red")
    roof2.Material = Enum.Material.Brick
    roof2.Anchored = true
    roof2.Parent = house
    
    -- ===== ประตู (Door) =====
    local door = Instance.new("Part")
    door.Name = "Door"
    door.Size = Vector3.new(2, 4, 0.3)
    door.CFrame = CFrame.new(position + Vector3.new(0, 2.5, -6))
    door.BrickColor = BrickColor.new("Reddish brown")
    door.Material = Enum.Material.Wood
    door.Anchored = true
    door.Parent = house
    
    -- ===== หน้าต่าง (Windows) =====
    local windowPositions = {
        Vector3.new(-4, 3, -6),
        Vector3.new(4, 3, -6),
        Vector3.new(-4, 3, 6),
        Vector3.new(4, 3, 6),
    }
    
    for i, pos in ipairs(windowPositions) do
        local window = Instance.new("Part")
        window.Name = "Window_" .. i
        window.Size = Vector3.new(2, 2, 0.2)
        window.CFrame = CFrame.new(position + pos)
        window.Color = Color3.fromRGB(173, 216, 230)  -- สีฟ้าอ่อน
        window.Material = Enum.Material.Glass
        window.Transparency = 0.5
        window.Anchored = true
        window.Parent = house
    end
    
    -- ตั้ง PrimaryPart
    house.PrimaryPart = floor
    
    return house
end

-- สร้างบ้าน
local myHouse = createHouse(Vector3.new(0, 0, 0))
print("สร้างบ้านเรียบร้อย! ชื่อ:", myHouse.Name)
print("จำนวน Parts:", #myHouse:GetDescendants())
```

---

## 14. การจัดการ Parts หลายๆ ชิ้น

### 14.1 GetChildren() และ GetDescendants()

```lua
-- GetChildren() - ได้ลูกโดยตรงเท่านั้น
local model = workspace.SimpleHouse
local children = model:GetChildren()
print("Children:", #children)

for _, child in pairs(children) do
    print(" -", child.Name, ":", child.ClassName)
end

-- GetDescendants() - ได้ทุก Object ในลำดับชั้นทั้งหมด
local descendants = model:GetDescendants()
print("\nAll descendants:", #descendants)
```

### 14.2 FindFirstChild() และ FindFirstChildOfClass()

```lua
-- หา Object ตามชื่อ
local floor = model:FindFirstChild("Floor")
if floor then
    print("พบ Floor:", floor.Size)
end

-- หา Object ตาม Class
local firstPart = model:FindFirstChildOfClass("Part")
if firstPart then
    print("Part แรกใน Model:", firstPart.Name)
end

-- หาแบบ Recursive (ค้นใน ลูกของลูกด้วย)
local deepObject = model:FindFirstChild("SomeName", true)
```

### 14.3 WaitForChild()

```lua
-- ใน LocalScript ที่รันก่อน Object สร้าง
-- ต้องใช้ WaitForChild แทน FindFirstChild

local Players = game:GetService("Players")
local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()

-- รอให้ HumanoidRootPart โหลด
local rootPart = character:WaitForChild("HumanoidRootPart")
print("RootPart position:", rootPart.Position)
```

---

## 📚 แบบฝึกหัดตอนที่ 4

### แบบฝึกหัดที่ 1: สร้าง Parts ต่างๆ
1. สร้าง Block, Sphere, Cylinder, Wedge
2. ปรับขนาดและสีให้ต่างกัน
3. ลองใช้ Materials ต่างๆ

### แบบฝึกหัดที่ 2: Transform Tools
1. ย้าย Part ด้วย Move Tool
2. ปรับขนาดด้วย Scale Tool
3. หมุนด้วย Rotate Tool
4. ทำซ้ำด้วย Ctrl+D แล้วจัดวาง

### แบบฝึกหัดที่ 3: สร้าง Model
1. สร้าง Parts 3-5 ชิ้น
2. Group เป็น Model (Ctrl+G)
3. ตั้งชื่อ Model
4. ย้าย Model ทั้งหมด

### แบบฝึกหัดที่ 4: โค้ดสร้าง Parts
1. คัดลอกโค้ด BuildSimpleHouse
2. รันและดูผลลัพธ์
3. แก้ไขสีและขนาดตามที่ต้องการ

### แบบฝึกหัดที่ 5: Union
1. สร้าง Part 2 ชิ้นทับกัน
2. ลอง Negate หนึ่งชิ้น
3. Union ทั้งสอง
4. ดูผลลัพธ์

### แบบฝึกหัดที่ 6: โปรเจกต์
สร้างเมืองเล็กๆ ที่มี:
- บ้าน 3 หลัง
- ถนน
- ต้นไม้ (Part ทรงกระบอก + Part ทรงกลม)
- ป้าย

---

## 💡 เคล็ดลับ

1. **ใช้ Grid Snap** - ทำให้ Parts ต่อกันสนิท
2. **ตั้งชื่อ Parts ทุกชิ้น** - ง่ายต่อการหาทีหลัง
3. **Group Parts เป็น Models** - จัดระเบียบดี
4. **Anchor ทุกอย่างที่ไม่ควรเคลื่อน** - ป้องกัน Physics ทำงานผิด
5. **ใช้ Duplicate (Ctrl+D)** - เร็วกว่าสร้างใหม่ทุกครั้ง

---

## ⏭️ ตอนถัดไป

ในตอนที่ 5 เราจะเรียนรู้:
- Properties Panel อย่างละเอียด
- การตั้งค่า Properties ต่างๆ
- Instance Properties vs Service Properties

---

*ตอนที่ 4/100 | ระดับ: พื้นฐาน | เวลา: 90-120 นาที*
