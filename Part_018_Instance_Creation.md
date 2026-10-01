# ตอนที่ 18: การสร้าง Instance ใน Scripts

## บทนำ

Instance ใน Roblox คือ object ทุกชนิดที่มีอยู่ในเกม ตั้งแต่ Parts, Models, Scripts ไปจนถึง GUI elements การสร้าง Instance ด้วย Script ช่วยให้เราสร้างและจัดการ objects แบบ dynamic ได้

---

## 18.1 Instance.new()

ฟังก์ชันพื้นฐานในการสร้าง Instance:

```lua
-- สร้าง Part พื้นฐาน
local part = Instance.new("Part")
part.Name = "MyPart"
part.Size = Vector3.new(4, 1, 4)
part.Position = Vector3.new(0, 5, 0)
part.Anchored = true
part.BrickColor = BrickColor.new("Bright red")
part.Parent = game.Workspace

-- สร้างพร้อม Parent (deprecated แต่ยังใช้ได้)
-- local part = Instance.new("Part", game.Workspace)  -- ไม่แนะนำ
```

### Instance Types ที่ใช้บ่อย

```lua
-- Parts
local part = Instance.new("Part")
local wedge = Instance.new("WedgePart")
local sphere = Instance.new("SpherePart")
local cylinder = Instance.new("CylinderMesh")  -- Mesh
local union = Instance.new("UnionOperation")

-- Models
local model = Instance.new("Model")

-- Scripts
local script = Instance.new("Script")
local localScript = Instance.new("LocalScript")
local moduleScript = Instance.new("ModuleScript")

-- GUI
local screenGui = Instance.new("ScreenGui")
local frame = Instance.new("Frame")
local textLabel = Instance.new("TextLabel")
local textButton = Instance.new("TextButton")
local imageLabel = Instance.new("ImageLabel")
local textBox = Instance.new("TextBox")

-- Effects
local particles = Instance.new("ParticleEmitter")
local light = Instance.new("PointLight")
local sound = Instance.new("Sound")
local fire = Instance.new("Fire")
local smoke = Instance.new("Smoke")
local sparkles = Instance.new("Sparkles")

-- Physics
local bodyVelocity = Instance.new("BodyVelocity")
local bodyGyro = Instance.new("BodyGyro")
local bodyPosition = Instance.new("BodyPosition")

-- Constraints
local weld = Instance.new("WeldConstraint")
local hinge = Instance.new("HingeConstraint")
local spring = Instance.new("SpringConstraint")

-- Values
local intValue = Instance.new("IntValue")
local numberValue = Instance.new("NumberValue")
local stringValue = Instance.new("StringValue")
local boolValue = Instance.new("BoolValue")
local objectValue = Instance.new("ObjectValue")
local vector3Value = Instance.new("Vector3Value")
local cframeValue = Instance.new("CFrameValue")
local colorValue = Instance.new("Color3Value")
```

---

## 18.2 การตั้งค่า Properties

```lua
-- สร้างและตั้งค่า Part อย่างสมบูรณ์
local function createDetailedPart()
    local part = Instance.new("Part")
    
    -- ชื่อ
    part.Name = "DetailedPart"
    
    -- ขนาดและตำแหน่ง
    part.Size = Vector3.new(4, 1, 4)
    part.Position = Vector3.new(0, 10, 0)
    part.Rotation = Vector3.new(0, 45, 0)
    -- หรือใช้ CFrame:
    -- part.CFrame = CFrame.new(0, 10, 0) * CFrame.Angles(0, math.rad(45), 0)
    
    -- รูปร่าง
    part.Shape = Enum.PartType.Block  -- Block, Cylinder, Ball
    
    -- วัสดุ
    part.Material = Enum.Material.SmoothPlastic
    
    -- สี
    part.BrickColor = BrickColor.new("Bright red")
    -- หรือ:
    -- part.Color = Color3.fromRGB(255, 50, 50)
    
    -- Transparency (0 = ทึบ, 1 = โปร่งใส)
    part.Transparency = 0.3
    
    -- Reflectance (0-1)
    part.Reflectance = 0.5
    
    -- Physics
    part.Anchored = true           -- ไม่ขยับด้วย physics
    part.CanCollide = true         -- ชนกับ objects อื่น
    part.Massless = false          -- มีมวล
    part.CustomPhysicalProperties = PhysicalProperties.new(
        1,     -- density
        0.3,   -- friction
        0.5,   -- elasticity
        0,     -- frictionWeight
        0      -- elasticityWeight
    )
    
    -- Surface types
    part.TopSurface = Enum.SurfaceType.Smooth
    part.BottomSurface = Enum.SurfaceType.Smooth
    
    part.Parent = game.Workspace
    return part
end
```

---

## 18.3 สร้าง Complex Objects

### สร้าง Model

```lua
local function createTree(position)
    local model = Instance.new("Model")
    model.Name = "Tree"
    
    -- ลำต้น
    local trunk = Instance.new("Part")
    trunk.Name = "Trunk"
    trunk.Size = Vector3.new(1, 5, 1)
    trunk.Position = position + Vector3.new(0, 2.5, 0)
    trunk.Anchored = true
    trunk.BrickColor = BrickColor.new("Reddish brown")
    trunk.Material = Enum.Material.Wood
    trunk.Parent = model
    
    -- ใบ (ทรงกลม)
    local leaves = Instance.new("Part")
    leaves.Name = "Leaves"
    leaves.Shape = Enum.PartType.Ball
    leaves.Size = Vector3.new(6, 6, 6)
    leaves.Position = position + Vector3.new(0, 8, 0)
    leaves.Anchored = true
    leaves.BrickColor = BrickColor.new("Bright green")
    leaves.Material = Enum.Material.Grass
    leaves.Parent = model
    
    -- กำหนด PrimaryPart
    model.PrimaryPart = trunk
    model.Parent = game.Workspace
    
    return model
end

-- สร้างป่า
for i = 1, 10 do
    local x = math.random(-50, 50)
    local z = math.random(-50, 50)
    createTree(Vector3.new(x, 0, z))
end
```

### สร้าง GUI

```lua
-- สร้าง Health Bar
local function createHealthBar(player)
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "HealthGui"
    screenGui.ResetOnSpawn = false
    
    -- Container
    local frame = Instance.new("Frame")
    frame.Name = "HealthFrame"
    frame.Size = UDim2.new(0, 200, 0, 30)
    frame.Position = UDim2.new(0, 10, 1, -40)  -- มุมซ้ายล่าง
    frame.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    frame.BorderSizePixel = 2
    frame.Parent = screenGui
    
    -- Background (ส่วนที่เป็นสีแดง)
    local bgBar = Instance.new("Frame")
    bgBar.Name = "BgBar"
    bgBar.Size = UDim2.new(1, 0, 1, 0)
    bgBar.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
    bgBar.BorderSizePixel = 0
    bgBar.Parent = frame
    
    -- Health bar (ส่วนที่เป็นสีเขียว)
    local healthBar = Instance.new("Frame")
    healthBar.Name = "HealthBar"
    healthBar.Size = UDim2.new(1, 0, 1, 0)  -- เริ่มต้น 100%
    healthBar.BackgroundColor3 = Color3.fromRGB(0, 200, 0)
    healthBar.BorderSizePixel = 0
    healthBar.Parent = frame
    
    -- Text
    local healthText = Instance.new("TextLabel")
    healthText.Name = "HealthText"
    healthText.Size = UDim2.new(1, 0, 1, 0)
    healthText.BackgroundTransparency = 1
    healthText.TextColor3 = Color3.new(1, 1, 1)
    healthText.TextStrokeTransparency = 0.5
    healthText.Text = "100/100"
    healthText.Font = Enum.Font.GothamBold
    healthText.TextSize = 14
    healthText.Parent = frame
    
    screenGui.Parent = player.PlayerGui
    
    -- ฟังก์ชันอัปเดต health
    local function updateHealth(current, max)
        local percentage = current / max
        healthBar.Size = UDim2.new(percentage, 0, 1, 0)
        healthText.Text = math.floor(current) .. "/" .. max
        
        -- เปลี่ยนสีตามเลือด
        if percentage > 0.6 then
            healthBar.BackgroundColor3 = Color3.fromRGB(0, 200, 0)
        elseif percentage > 0.3 then
            healthBar.BackgroundColor3 = Color3.fromRGB(255, 200, 0)
        else
            healthBar.BackgroundColor3 = Color3.fromRGB(200, 0, 0)
        end
    end
    
    return screenGui, updateHealth
end
```

---

## 18.4 Cloning Instances

```lua
-- Clone: ทำสำเนา instance
local original = game.Workspace.Tree

-- Clone และวางใหม่
local clone1 = original:Clone()
clone1.Position = Vector3.new(20, 0, 0)  -- ถ้า clone เป็น Part
clone1.Parent = game.Workspace

-- Clone จาก ReplicatedStorage (pattern ที่แนะนำ)
local enemyTemplate = game.ReplicatedStorage:FindFirstChild("EnemyTemplate")

local function spawnEnemy(position)
    if not enemyTemplate then
        print("ไม่พบ template!")
        return nil
    end
    
    local enemy = enemyTemplate:Clone()
    
    -- ตั้งค่าหลัง clone
    enemy.Name = "Enemy_" .. tostring(os.clock())
    
    -- ย้ายไปตำแหน่งที่ต้องการ
    if enemy:IsA("Model") and enemy.PrimaryPart then
        enemy:SetPrimaryPartCFrame(CFrame.new(position))
    elseif enemy:IsA("BasePart") then
        enemy.Position = position
    end
    
    enemy.Parent = game.Workspace
    return enemy
end
```

---

## 18.5 การลบ Instance

```lua
-- Destroy: ลบ instance ถาวร
local part = game.Workspace.MyPart

-- วิธีที่ 1: Destroy ทันที
part:Destroy()

-- วิธีที่ 2: ลบหลังเวลาที่กำหนด
game.Debris:AddItem(part, 5)  -- ลบหลัง 5 วินาที

-- วิธีที่ 3: ลบด้วย task.delay
task.delay(5, function()
    if part and part.Parent then
        part:Destroy()
    end
end)

-- ตัวอย่างจริง: ลูกกระสุน
local function createBullet(startPos, direction, speed)
    local bullet = Instance.new("Part")
    bullet.Name = "Bullet"
    bullet.Size = Vector3.new(0.2, 0.2, 1)
    bullet.CFrame = CFrame.new(startPos, startPos + direction)
    bullet.BrickColor = BrickColor.new("Bright yellow")
    bullet.Material = Enum.Material.Neon
    bullet.CanCollide = false
    bullet.Parent = game.Workspace
    
    -- เพิ่มความเร็ว
    local velocity = Instance.new("LinearVelocity")
    velocity.MaxForce = math.huge
    velocity.VectorVelocity = direction * speed
    velocity.Parent = bullet
    
    -- ลบหลัง 3 วินาที
    game.Debris:AddItem(bullet, 3)
    
    -- หรือลบเมื่อกระทบ
    bullet.Touched:Connect(function(hit)
        if hit ~= bullet then
            bullet:Destroy()
        end
    end)
    
    return bullet
end
```

---

## 18.6 WeldConstraint

```lua
-- เชื่อม Parts เข้าด้วยกัน
local function weldParts(part0, part1)
    local weld = Instance.new("WeldConstraint")
    weld.Part0 = part0
    weld.Part1 = part1
    weld.Parent = part0
    return weld
end

-- สร้าง Sword Model ที่ weld กัน
local function createSword()
    local model = Instance.new("Model")
    model.Name = "Sword"
    
    -- ด้ามจับ
    local handle = Instance.new("Part")
    handle.Name = "Handle"
    handle.Size = Vector3.new(0.3, 0.3, 1.5)
    handle.BrickColor = BrickColor.new("Dark orange")
    handle.Anchored = false
    handle.Parent = model
    
    -- ใบมีด
    local blade = Instance.new("Part")
    blade.Name = "Blade"
    blade.Size = Vector3.new(0.1, 0.1, 3)
    blade.BrickColor = BrickColor.new("Light stone grey")
    blade.Material = Enum.Material.SmoothPlastic
    blade.Anchored = false
    
    -- วางใบมีดต่อจากด้ามจับ
    blade.CFrame = handle.CFrame * CFrame.new(0, 0, -2.25)
    blade.Parent = model
    
    -- Weld
    weldParts(handle, blade)
    
    model.PrimaryPart = handle
    return model
end
```

---

## 18.7 ตัวอย่างโปรเจกต์: Obstacle Course Generator

```lua
-- Script: ObstacleCourseGenerator.lua
-- สร้าง obstacle course แบบ procedural

local function createPlatform(pos, size, color)
    local part = Instance.new("Part")
    part.Size = size
    part.Position = pos
    part.Anchored = true
    part.BrickColor = BrickColor.new(color)
    part.Material = Enum.Material.SmoothPlastic
    part.TopSurface = Enum.SurfaceType.Smooth
    part.Parent = game.Workspace
    return part
end

local function createKillBrick(pos, size)
    local part = createPlatform(pos, size, "Bright red")
    part.Name = "KillBrick"
    
    -- Kill player on touch
    part.Touched:Connect(function(hit)
        local humanoid = hit.Parent:FindFirstChild("Humanoid")
        if humanoid then
            humanoid.Health = 0
        end
    end)
    
    return part
end

local function createJumpPad(pos)
    local pad = createPlatform(pos, Vector3.new(4, 0.5, 4), "Bright yellow")
    pad.Name = "JumpPad"
    
    local debounce = false
    pad.Touched:Connect(function(hit)
        if debounce then return end
        
        local humanoid = hit.Parent:FindFirstChild("Humanoid")
        if humanoid then
            debounce = true
            local rootPart = hit.Parent:FindFirstChild("HumanoidRootPart")
            if rootPart then
                local bv = Instance.new("BodyVelocity")
                bv.MaxForce = Vector3.new(0, math.huge, 0)
                bv.Velocity = Vector3.new(0, 80, 0)
                bv.Parent = rootPart
                
                game.Debris:AddItem(bv, 0.2)
            end
            
            task.wait(1)
            debounce = false
        end
    end)
    
    return pad
end

local function generateObstacleCourse(length)
    local startPos = Vector3.new(0, 0, 0)
    local currentPos = startPos
    
    -- Start Platform
    createPlatform(currentPos, Vector3.new(10, 1, 10), "Bright green")
    currentPos = currentPos + Vector3.new(0, 0, -12)
    
    for i = 1, length do
        -- สุ่ม obstacle type
        local obstacleType = math.random(1, 5)
        
        if obstacleType == 1 then
            -- Platform ปกติ
            local height = math.random(-2, 5)
            createPlatform(
                currentPos + Vector3.new(0, height, 0),
                Vector3.new(math.random(3, 8), 1, math.random(3, 8)),
                "Medium stone grey"
            )
            currentPos = currentPos + Vector3.new(0, height, -math.random(8, 12))
            
        elseif obstacleType == 2 then
            -- Kill brick ขวางทาง
            createKillBrick(
                currentPos + Vector3.new(0, 1, -5),
                Vector3.new(8, 3, 1)
            )
            -- Platform ที่ต้องกระโดดข้าม
            createPlatform(currentPos, Vector3.new(4, 1, 4), "Bright blue")
            currentPos = currentPos + Vector3.new(0, 0, -14)
            createPlatform(currentPos, Vector3.new(4, 1, 4), "Bright blue")
            currentPos = currentPos + Vector3.new(0, 0, -6)
            
        elseif obstacleType == 3 then
            -- Jump pad
            createJumpPad(currentPos)
            currentPos = currentPos + Vector3.new(0, 15, -15)
            createPlatform(currentPos, Vector3.new(6, 1, 6), "Bright orange")
            currentPos = currentPos + Vector3.new(0, 0, -8)
            
        elseif obstacleType == 4 then
            -- Narrow path
            local pathLength = math.random(15, 30)
            for j = 0, pathLength, 4 do
                createPlatform(
                    currentPos + Vector3.new(0, 0, -j),
                    Vector3.new(2, 1, 4),
                    "Bright yellow"
                )
            end
            currentPos = currentPos + Vector3.new(0, 0, -(pathLength + 6))
            
        else
            -- Staircase
            for step = 1, 5 do
                createPlatform(
                    currentPos + Vector3.new(0, step * 2, -step * 3),
                    Vector3.new(6, 1, 3),
                    "Light blue"
                )
            end
            currentPos = currentPos + Vector3.new(0, 10, -18)
        end
    end
    
    -- End Platform (ชัยชนะ)
    local endPlatform = createPlatform(
        currentPos, 
        Vector3.new(15, 1, 15), 
        "Bright yellow"
    )
    endPlatform.Name = "EndPlatform"
    
    -- เพิ่ม finish detection
    endPlatform.Touched:Connect(function(hit)
        local player = game.Players:GetPlayerFromCharacter(hit.Parent)
        if player then
            print(player.Name .. " เดินทางถึงเส้นชัย!")
        end
    end)
    
    print("สร้าง Obstacle Course เสร็จแล้ว! (" .. length .. " sections)")
end

-- สร้าง course
generateObstacleCourse(10)
```

---

## 18.8 แบบฝึกหัด

### แบบฝึกหัดที่ 1: สร้าง Coin

```lua
-- สร้าง Coin ที่:
-- - หมุนรอบตัวเอง
-- - เมื่อผู้เล่นเก็บได้ ให้คะแนน +10
-- - หายไปและปรากฏใหม่หลัง 5 วินาที

local function createCoin(position, value)
    -- เติมโค้ด
end

-- สร้าง coins หลายๆ อัน
for i = 1, 5 do
    createCoin(Vector3.new(i * 5, 2, 0), 10)
end
```

### แบบฝึกหัดที่ 2: สร้าง Moving Platform

```lua
-- สร้าง Platform ที่เคลื่อนที่ระหว่าง 2 จุด:
-- - เคลื่อนที่แบบ smooth (tween)
-- - รอที่จุดหมายก่อนกลับ

local function createMovingPlatform(startPos, endPos, speed, waitTime)
    -- เติมโค้ด
end

createMovingPlatform(
    Vector3.new(0, 5, 0),
    Vector3.new(20, 5, 0),
    5,   -- speed
    2    -- wait time
)
```

---

## สรุป

| Action | Code |
|--------|------|
| สร้าง | `Instance.new("Type")` |
| Clone | `instance:Clone()` |
| ลบ | `instance:Destroy()` |
| ลบหลังเวลา | `game.Debris:AddItem(inst, time)` |
| Weld | `WeldConstraint` |
| Set Parent | `inst.Parent = parent` |

### บทถัดไป

ในบทที่ 19 เราจะเรียนเรื่อง **Parent-Child Hierarchy** - การจัดการโครงสร้างของ instances
