# ตอนที่ 5: Properties Panel
## Part 5: Using the Properties Panel

---

**ระดับ:** พื้นฐาน (Beginner)  
**เวลาเรียน:** 60-90 นาที  
**ข้อกำหนดเบื้องต้น:** ตอนที่ 1-4

---

## 🎯 วัตถุประสงค์การเรียนรู้

เมื่อเรียนจบตอนนี้ คุณจะสามารถ:
1. ใช้งาน Properties Panel ได้อย่างครบถ้วน
2. เข้าใจ Property Categories ต่างๆ
3. แก้ไข Properties ผ่าน Panel และผ่านโค้ด
4. รู้จัก Properties สำคัญของ Instance ต่างๆ
5. ใช้ Script เพื่ออ่านและเขียน Properties

---

## 1. ภาพรวม Properties Panel

### 1.1 การเปิด Properties Panel

```
View > Properties
หรือ กด F4
หรือ คลิกที่ Object ใน Explorer (จะเปิดอัตโนมัติ)
```

### 1.2 โครงสร้างของ Properties Panel

```
Properties - Part "MyPart"
+--------------------------------------+
| 🔍 [Search Properties...]            |
+--------------------------------------+
| Appearance                         ▼ |
|   BrickColor:      [████] Bright red |
|   CastShadow:      ✓                 |
|   Color:           [█████████]       |
|   Material:        SmoothPlastic ▼   |
|   Reflectance:     [====] 0          |
|   Transparency:    [====] 0          |
+--------------------------------------+
| Behavior                           ▼ |
|   Anchored:        ✓                 |
|   CanCollide:      ✓                 |
|   CanQuery:        ✓                 |
|   CanTouch:        ✓                 |
|   CollisionGroup:  Default ▼         |
|   Locked:          □                 |
|   Massless:        □                 |
+--------------------------------------+
| Data                               ▼ |
|   ClassName:       Part              |
|   Name:            MyPart            |
|   Parent:          Workspace         |
+--------------------------------------+
| Part                               ▼ |
|   BackSurface:     Smooth ▼          |
|   BottomSurface:   Inlet ▼           |
|   CFrame:          ...               |
|   FrontSurface:    Smooth ▼          |
|   LeftSurface:     Smooth ▼          |
|   Position:        0, 5, 0           |
|   RightSurface:    Smooth ▼          |
|   Rotation:        0, 0, 0           |
|   Size:            4, 4, 4           |
|   TopSurface:      Smooth ▼          |
+--------------------------------------+
```

---

## 2. Property Types

### 2.1 Boolean Properties

```
Properties ที่มีค่า true/false:
- Anchored: ✓ หรือ □
- CanCollide: ✓ หรือ □
- Visible: ✓ หรือ □

วิธีแก้ไข: คลิกที่ Checkbox
```

```lua
-- ใน Script
part.Anchored = true
part.CanCollide = false
print(part.Anchored)  -- แสดง: true
```

### 2.2 Number Properties

```
Properties ที่เป็นตัวเลข:
- Transparency: 0-1
- Reflectance: 0-1

วิธีแก้ไข: 
- คลิกและพิมพ์ตัวเลข
- ลาก Slider
```

```lua
-- ใน Script
part.Transparency = 0.5
part.Reflectance = 0.3
```

### 2.3 String Properties

```
Properties ที่เป็นข้อความ:
- Name: ชื่อของ Object

วิธีแก้ไข: คลิกและพิมพ์
```

```lua
-- ใน Script
part.Name = "NewName"
print(part.Name)  -- แสดง: NewName
```

### 2.4 Enum Properties

```
Properties ที่มีตัวเลือกจำกัด:
- Material: SmoothPlastic, Metal, Wood...
- Shape: Block, Ball, Cylinder...

วิธีแก้ไข: คลิก Dropdown และเลือก
```

```lua
-- ใน Script
part.Material = Enum.Material.Metal
part.Shape = Enum.PartType.Ball
```

### 2.5 Vector3 Properties

```
Properties ที่เป็น 3D Vector:
- Size: (X, Y, Z)
- Position: (X, Y, Z)
- Rotation: (X, Y, Z)

วิธีแก้ไข: คลิก Expand (▶) แล้วพิมพ์แต่ละค่า
```

```lua
-- ใน Script
part.Size = Vector3.new(4, 4, 4)
part.Position = Vector3.new(0, 5, 0)
print(part.Size.X, part.Size.Y, part.Size.Z)  -- แสดง: 4 4 4
```

### 2.6 Color Properties

```
Properties ที่เป็นสี:
- Color: Color3 (RGB)
- BrickColor: Classic Roblox color

วิธีแก้ไข: คลิกที่กล่องสีเพื่อเปิด Color Picker
```

```lua
-- ใน Script
part.Color = Color3.fromRGB(255, 0, 0)     -- แดง
part.BrickColor = BrickColor.new("Bright blue")
```

### 2.7 CFrame Properties

```
Properties ที่เป็น CFrame:
- CFrame: ทั้ง Position + Rotation

วิธีแก้ไข: ใส่ค่าใน Sub-fields
```

```lua
-- ใน Script
part.CFrame = CFrame.new(0, 5, 0)
part.CFrame = CFrame.new(0, 5, 0) * CFrame.Angles(0, math.rad(45), 0)
```

---

## 3. Part Properties อย่างละเอียด

### 3.1 Appearance Properties

```lua
-- ===== APPEARANCE PROPERTIES =====

local part = Instance.new("Part")
part.Anchored = true
part.Parent = workspace

-- Material - วัสดุของ Part
part.Material = Enum.Material.SmoothPlastic  -- พลาสติกเรียบ
part.Material = Enum.Material.Metal          -- โลหะ
part.Material = Enum.Material.Wood           -- ไม้
part.Material = Enum.Material.Neon           -- เรืองแสง
-- ดูรายการทั้งหมด: Enum.Material

-- Color - สีแบบ RGB
part.Color = Color3.fromRGB(255, 100, 50)    -- ส้มแดง

-- BrickColor - สี Classic
part.BrickColor = BrickColor.new("Bright red")

-- Transparency - ความโปร่งใส
part.Transparency = 0      -- ทึบสนิท
part.Transparency = 0.5    -- ครึ่งใส
part.Transparency = 1      -- ใสสนิท

-- Reflectance - การสะท้อนแสง
part.Reflectance = 0       -- ไม่สะท้อน
part.Reflectance = 1       -- สะท้อนมาก

-- CastShadow - ทำเงาหรือไม่
part.CastShadow = true
part.CastShadow = false    -- ดีสำหรับ Performance
```

### 3.2 Behavior Properties

```lua
-- ===== BEHAVIOR PROPERTIES =====

-- Anchored - ยึดติด ไม่เคลื่อนไหว
part.Anchored = true

-- CanCollide - ชนกับ Objects อื่น
part.CanCollide = true
part.CanCollide = false  -- ทะลุได้

-- CanQuery - ถูก Raycast ตรวจจับ
part.CanQuery = true
part.CanQuery = false  -- Raycast ไม่เจอ

-- CanTouch - Trigger Touch Events
part.CanTouch = true
part.CanTouch = false  -- ไม่ trigger Touched event

-- CollisionGroup - กลุ่ม Collision
part.CollisionGroup = "Default"
-- กำหนดด้วย PhysicsService

-- Locked - ล็อคใน Studio (ป้องกันการเลือก)
part.Locked = false
part.Locked = true   -- เลือก ลบ แก้ไขใน Studio ไม่ได้

-- Massless - ไม่มีมวลสำหรับ Assembly
part.Massless = false
```

### 3.3 Data Properties (Read-only)

```lua
-- ===== DATA PROPERTIES =====
-- Properties เหล่านี้อ่านได้แต่เปลี่ยนไม่ได้ทั้งหมด

-- ClassName - ประเภทของ Instance
print(part.ClassName)  -- "Part"

-- Name - ชื่อของ Instance (เปลี่ยนได้)
part.Name = "MySpecialPart"
print(part.Name)

-- Parent - Parent ของ Instance (เปลี่ยนได้)
part.Parent = workspace
print(part.Parent.Name)  -- "Workspace"
```

### 3.4 Part Geometry Properties

```lua
-- ===== PART GEOMETRY PROPERTIES =====

-- Position - ตำแหน่ง Center ของ Part
part.Position = Vector3.new(0, 5, 0)

-- Rotation - การหมุนเป็นองศา
part.Rotation = Vector3.new(0, 45, 0)  -- หมุน 45° รอบ Y

-- Size - ขนาด
part.Size = Vector3.new(4, 2, 6)  -- กว้าง 4, สูง 2, ลึก 6

-- CFrame - ทั้ง Position + Rotation
part.CFrame = CFrame.new(0, 5, 0) * CFrame.Angles(0, math.rad(45), 0)

-- Shape - รูปทรง (เฉพาะ Part พื้นฐาน)
part.Shape = Enum.PartType.Block
part.Shape = Enum.PartType.Ball
part.Shape = Enum.PartType.Cylinder
```

### 3.5 Surface Properties

```lua
-- Surface Types สำหรับแต่ละ Face
part.TopSurface = Enum.SurfaceType.Smooth
part.BottomSurface = Enum.SurfaceType.Inlet  -- เชื่อมกับ Stud
part.FrontSurface = Enum.SurfaceType.Weld
part.BackSurface = Enum.SurfaceType.Smooth
part.LeftSurface = Enum.SurfaceType.Smooth
part.RightSurface = Enum.SurfaceType.Smooth

-- Surface Types:
-- Smooth    = เรียบ
-- Inlet     = เว้า (รับ Stud)
-- Studs     = มีปุ่ม Lego
-- Glue      = เชื่อม
-- Weld      = เชื่อม
-- Universal = รอบทิศทาง
```

---

## 4. Humanoid Properties

### 4.1 ภาพรวม Humanoid

Humanoid คือ Component ที่ควบคุม Character:

```lua
-- ดู Humanoid Properties
local player = game.Players.LocalPlayer
local character = player.Character

if character then
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    
    if humanoid then
        -- Health - พลังชีวิต
        print("Health:", humanoid.Health)
        print("MaxHealth:", humanoid.MaxHealth)
        
        -- Movement
        print("WalkSpeed:", humanoid.WalkSpeed)
        print("JumpPower:", humanoid.JumpPower)
        
        -- State
        print("HipHeight:", humanoid.HipHeight)
        print("AutoJumpEnabled:", humanoid.AutoJumpEnabled)
    end
end
```

### 4.2 การเปลี่ยน Humanoid Properties

```lua
-- Script ใน StarterCharacterScripts หรือ StarterPlayerScripts

local Players = game:GetService("Players")
local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")

-- เพิ่มความเร็ว
humanoid.WalkSpeed = 20  -- ค่าเริ่มต้น 16
humanoid.JumpPower = 70  -- ค่าเริ่มต้น 50

-- เพิ่ม Health สูงสุด
humanoid.MaxHealth = 200
humanoid.Health = humanoid.MaxHealth

-- ปิด Auto-jump
humanoid.AutoJumpEnabled = false
```

---

## 5. Script Properties

### 5.1 Script Properties

```lua
-- Script Properties
local script = script  -- อ้างถึง Script นี้เอง

-- Enabled - เปิด/ปิด Script
-- (ดูใน Properties Panel)

-- ClassName
print(script.ClassName)  -- "Script", "LocalScript", หรือ "ModuleScript"

-- Name
print(script.Name)  -- ชื่อของ Script

-- Parent
print(script.Parent.Name)  -- Parent ของ Script
```

### 5.2 Script Disabled Property

```lua
-- ใน Script อื่นที่ต้องการ Disable Script นี้
local targetScript = workspace:FindFirstChild("MyScript")
if targetScript and targetScript:IsA("Script") then
    targetScript.Disabled = true   -- ปิด Script
    -- targetScript.Disabled = false  -- เปิด Script
end
```

---

## 6. GUI Properties

### 6.1 ScreenGui Properties

```lua
-- Properties ของ ScreenGui
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "MyGui"
screenGui.Parent = game.Players.LocalPlayer.PlayerGui

-- ResetOnSpawn - Reset GUI เมื่อ Respawn
screenGui.ResetOnSpawn = false

-- ZIndexBehavior
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

-- Enabled
screenGui.Enabled = true
```

### 6.2 Frame Properties

```lua
local frame = Instance.new("Frame")
frame.Parent = screenGui

-- Size - ขนาดใน UDim2
frame.Size = UDim2.new(0.5, 0, 0.5, 0)
-- (0.5 = 50% ของ Screen, 0 = ไม่มี Offset)

-- Position - ตำแหน่งใน UDim2
frame.Position = UDim2.new(0.25, 0, 0.25, 0)
-- (25% จากซ้าย, 25% จากบน)

-- BackgroundColor3
frame.BackgroundColor3 = Color3.fromRGB(50, 50, 50)

-- BackgroundTransparency
frame.BackgroundTransparency = 0.5

-- BorderSizePixel
frame.BorderSizePixel = 2

-- ZIndex - ลำดับการแสดงผล (สูงกว่า = ด้านหน้า)
frame.ZIndex = 1
```

---

## 7. การค้นหา Properties ด้วยโค้ด

### 7.1 อ่าน Property ด้วยโค้ด

```lua
-- การอ่าน Properties
local part = workspace.Baseplate

-- ตรงๆ
print(part.Size)        -- Vector3
print(part.Position)    -- Vector3
print(part.Color)       -- Color3
print(part.Material)    -- Enum
print(part.Anchored)    -- Boolean
print(part.Name)        -- String

-- เก็บใน Variable
local partSize = part.Size
local partX = partSize.X
local partY = partSize.Y
local partZ = partSize.Z
print(partX, partY, partZ)
```

### 7.2 เขียน Property ด้วยโค้ด

```lua
-- การเขียน Properties
local part = workspace.Baseplate

part.BrickColor = BrickColor.new("Bright blue")
part.Material = Enum.Material.Neon
part.Transparency = 0.3
part.Anchored = true
part.Name = "ColorfulBase"

-- หลายค่าพร้อมกัน
local properties = {
    BrickColor = BrickColor.new("Bright red"),
    Material = Enum.Material.Metal,
    Transparency = 0,
    Reflectance = 0.5,
    CastShadow = true,
}

for propName, value in pairs(properties) do
    part[propName] = value
end
```

### 7.3 การใช้ pcall สำหรับ Property ที่ไม่แน่ใจ

```lua
-- บาง Property อาจไม่มีใน Instance นั้น
local instance = workspace.Baseplate

local success, value = pcall(function()
    return instance.SomePossibleProperty
end)

if success then
    print("ค่า Property:", value)
else
    print("ไม่มี Property นี้:", value)  -- value เป็น error message
end
```

---

## 8. Instance Methods ที่เกี่ยวกับ Properties

### 8.1 GetPropertyChangedSignal

```lua
-- ดักจับการเปลี่ยน Property
local part = workspace.Baseplate

part:GetPropertyChangedSignal("Color"):Connect(function()
    print("สีเปลี่ยน! สีใหม่:", part.Color)
end)

-- ทดสอบ
wait(2)
part.Color = Color3.fromRGB(255, 0, 0)
-- Output: สีเปลี่ยน! สีใหม่: 1, 0, 0
```

### 8.2 Changed Event

```lua
-- Changed event จะ Fire เมื่อ Property ใดก็ได้เปลี่ยน
local part = workspace.Baseplate

part.Changed:Connect(function(property)
    print("Property ที่เปลี่ยน:", property)
    print("ค่าใหม่:", part[property])
end)
```

### 8.3 IsA() และ ClassName

```lua
-- ตรวจสอบประเภทของ Instance
local instance = workspace.Baseplate

print(instance:IsA("Part"))       -- true
print(instance:IsA("BasePart"))   -- true (Part เป็น subclass)
print(instance:IsA("Model"))      -- false

print(instance.ClassName)         -- "Part"

-- ตรวจสอบก่อนใช้ Properties เฉพาะ
if instance:IsA("BasePart") then
    print("Size:", instance.Size)
    print("Position:", instance.Position)
end
```

---

## 9. ตัวอย่างโปรแกรม: Property Inspector

```lua
-- Script: PropertyInspector
-- วางใน: ServerScriptService

-- Function แสดง Properties ของ Instance
local function inspectInstance(instance)
    print("========================================")
    print("INSPECTING:", instance.Name)
    print("CLASS:", instance.ClassName)
    print("PATH:", instance:GetFullName())
    print("========================================")
    
    -- ตรวจสอบ Properties ที่ทุก Instance มี
    print("\n[Common Properties]")
    print("  Name:", instance.Name)
    print("  Parent:", instance.Parent and instance.Parent.Name or "nil")
    print("  Children count:", #instance:GetChildren())
    
    -- ถ้าเป็น BasePart
    if instance:IsA("BasePart") then
        print("\n[BasePart Properties]")
        print("  Size:", instance.Size)
        print("  Position:", instance.Position)
        print("  Rotation:", instance.Rotation)
        print("  Color:", instance.Color)
        print("  Material:", instance.Material)
        print("  Transparency:", instance.Transparency)
        print("  Anchored:", instance.Anchored)
        print("  CanCollide:", instance.CanCollide)
        print("  CastShadow:", instance.CastShadow)
        print("  Reflectance:", instance.Reflectance)
    end
    
    -- ถ้าเป็น Model
    if instance:IsA("Model") then
        print("\n[Model Properties]")
        print("  PrimaryPart:", instance.PrimaryPart and instance.PrimaryPart.Name or "nil")
        local extents = instance:GetExtentsSize()
        print("  Extents Size:", extents)
    end
    
    -- ถ้าเป็น Humanoid
    if instance:IsA("Humanoid") then
        print("\n[Humanoid Properties]")
        print("  Health:", instance.Health)
        print("  MaxHealth:", instance.MaxHealth)
        print("  WalkSpeed:", instance.WalkSpeed)
        print("  JumpPower:", instance.JumpPower)
    end
    
    print("========================================\n")
end

-- ตรวจสอบ Baseplate
wait(1)  -- รอให้โหลด

local baseplate = workspace:FindFirstChild("Baseplate")
if baseplate then
    inspectInstance(baseplate)
end

-- ตรวจสอบ Workspace
inspectInstance(workspace)
```

---

## 10. ตัวอย่างการใช้ Properties สำหรับ Game Effects

### 10.1 Flashing Effect (กระพริบ)

```lua
-- Script: FlashingPart
-- วางใน: ServerScriptService

local part = Instance.new("Part")
part.Name = "FlashingLight"
part.Size = Vector3.new(2, 2, 2)
part.Position = Vector3.new(0, 5, 0)
part.Anchored = true
part.Material = Enum.Material.Neon
part.BrickColor = BrickColor.new("Bright red")
part.Parent = workspace

-- กระพริบ
while true do
    part.Transparency = 0      -- ทึบ
    wait(0.5)
    part.Transparency = 1      -- ใส
    wait(0.5)
end
```

### 10.2 Color Cycling Effect (เปลี่ยนสี)

```lua
-- Script: ColorCycling
-- วางใน: ServerScriptService

local part = Instance.new("Part")
part.Name = "RainbowPart"
part.Size = Vector3.new(4, 4, 4)
part.Position = Vector3.new(0, 5, 0)
part.Anchored = true
part.Material = Enum.Material.Neon
part.Parent = workspace

-- สีรุ้ง
local hue = 0

while true do
    -- Color3.fromHSV(Hue, Saturation, Value)
    part.Color = Color3.fromHSV(hue, 1, 1)
    hue = (hue + 0.01) % 1  -- วนรอบ 0-1
    wait(0.05)
end
```

### 10.3 Growing and Shrinking (ขยาย-หด)

```lua
-- Script: PulsePart
-- วางใน: ServerScriptService

local part = Instance.new("Part")
part.Name = "PulsingPart"
part.Shape = Enum.PartType.Ball
part.Size = Vector3.new(3, 3, 3)
part.Position = Vector3.new(0, 5, 0)
part.Anchored = true
part.BrickColor = BrickColor.new("Bright blue")
part.Material = Enum.Material.Neon
part.Parent = workspace

local baseSize = 3
local amplitude = 1
local speed = 2
local time = 0

-- Loop ขยาย-หด
game:GetService("RunService").Heartbeat:Connect(function(dt)
    time = time + dt * speed
    local scale = baseSize + amplitude * math.sin(time)
    part.Size = Vector3.new(scale, scale, scale)
end)
```

---

## 11. Script Properties Access Patterns

### 11.1 Property Chains

```lua
-- การเข้าถึง Properties แบบลูกโซ่
local player = game.Players.LocalPlayer
local character = player.Character
local humanoid = character and character:FindFirstChild("Humanoid")
local health = humanoid and humanoid.Health

if health then
    print("Health:", health)
end

-- หรือแบบสั้นกว่า (ต้องระวัง nil!)
local hp = game.Players.LocalPlayer.Character
             and game.Players.LocalPlayer.Character:FindFirstChild("Humanoid")
             and game.Players.LocalPlayer.Character.Humanoid.Health
```

### 11.2 Default Values

```lua
-- ค่าเริ่มต้นของ Part Properties
local part = Instance.new("Part")
print("Default Material:", part.Material)          -- SmoothPlastic
print("Default Color:", part.Color)                -- (0.6, 0.522, 0.435)
print("Default Size:", part.Size)                  -- 4, 1.2, 2
print("Default Position:", part.Position)          -- 0, 0, 0
print("Default Anchored:", part.Anchored)          -- false
print("Default CanCollide:", part.CanCollide)      -- true
print("Default Transparency:", part.Transparency)  -- 0
```

---

## 📚 แบบฝึกหัดตอนที่ 5

### แบบฝึกหัดที่ 1: สำรวจ Properties
1. Select Baseplate ใน Explorer
2. ดู Properties ทั้งหมดใน Properties Panel
3. สำรวจ Property Categories แต่ละอัน

### แบบฝึกหัดที่ 2: แก้ไขผ่าน Panel
1. เปลี่ยน Color ของ Baseplate
2. เปลี่ยน Material เป็น Grass
3. ลอง Transparency ที่ 0.5
4. เปิด/ปิด CastShadow

### แบบฝึกหัดที่ 3: แก้ไขผ่านโค้ด
1. สร้าง Script ใน ServerScriptService
2. เขียนโค้ดเพื่อเปลี่ยน Properties ของ Baseplate
3. รันและดูผลลัพธ์

### แบบฝึกหัดที่ 4: Property Inspector
1. คัดลอกโค้ด PropertyInspector
2. รันและดู Output
3. แก้ไขให้ Inspect Objects อื่นๆ ด้วย

### แบบฝึกหัดที่ 5: Effects
1. สร้าง FlashingPart Effect
2. สร้าง ColorCycling Effect
3. ทดสอบและปรับแต่ง Timing

### แบบฝึกหัดที่ 6: GetPropertyChangedSignal
1. สร้าง Part
2. ใช้ GetPropertyChangedSignal ดักการเปลี่ยนสี
3. ลองเปลี่ยนสีใน Properties Panel ขณะ Play

---

## 💡 เคล็ดลับ

1. **ค้นหาใน Properties** - พิมพ์ชื่อ Property ในช่อง Search
2. **Copy Properties** - คลิกขวาที่ Object > Copy และ Paste ค่า Properties
3. **Multiple Selection** - เลือกหลาย Objects พร้อมกันเพื่อแก้ Property ทีเดียว
4. **Reset to Default** - คลิกขวาที่ Property > Reset To Default

---

## ⏭️ ตอนถัดไป

ในตอนที่ 6 เราจะเรียนรู้:
- Workspace และ Explorer Panel อย่างละเอียด
- Service Hierarchy
- Instance Tree Management

---

*ตอนที่ 5/100 | ระดับ: พื้นฐาน | เวลา: 60-90 นาที*
