# ตอนที่ 7: การสร้าง Baseplate และ Environment แรก
## Part 7: Creating Your First Baseplate and Environment

---

**ระดับ:** พื้นฐาน (Beginner)  
**เวลาเรียน:** 90-120 นาที  
**ข้อกำหนดเบื้องต้น:** ตอนที่ 1-6

---

## 🎯 วัตถุประสงค์การเรียนรู้

เมื่อเรียนจบตอนนี้ คุณจะสามารถ:
1. สร้าง Environment พื้นฐานสำหรับเกม
2. ออกแบบ Level Layout ง่ายๆ
3. ใช้ Parts สร้างโครงสร้างต่างๆ
4. เพิ่ม Details และ Decorations
5. ตั้งค่า Spawn Points และ Checkpoints

---

## 1. Baseplate คืออะไร?

### 1.1 ความหมาย

**Baseplate** คือพื้นที่หลักที่ผู้เล่น Spawn และเดินบน:
- เป็น Part ขนาดใหญ่ที่เป็นพื้น
- Anchored (ยึดติด ไม่ตก)
- มี CanCollide = true
- โดยทั่วไปมีขนาด 512 x 20 x 512 Studs

### 1.2 โครงสร้าง Default

```
Default Baseplate Template:
├── Workspace
│   ├── Camera
│   ├── Terrain
│   ├── Baseplate (512 x 20 x 512)
│   └── SpawnLocation
```

---

## 2. ปรับแต่ง Baseplate

### 2.1 เปลี่ยนขนาดและรูปลักษณ์

```lua
-- Script: CustomizeBaseplate
-- วางใน: ServerScriptService

local baseplate = workspace:FindFirstChild("Baseplate")

if baseplate then
    -- ขนาด
    baseplate.Size = Vector3.new(200, 5, 200)
    
    -- ตำแหน่ง (กลาง, ต่ำลงมา)
    baseplate.Position = Vector3.new(0, -2.5, 0)
    
    -- สี
    baseplate.BrickColor = BrickColor.new("Medium stone grey")
    baseplate.Material = Enum.Material.Grass
    
    -- คุณสมบัติ
    baseplate.Anchored = true
    baseplate.CanCollide = true
    baseplate.CastShadow = false  -- ประหยัด Performance
    
    print("Baseplate ปรับแต่งเรียบร้อย!")
end
```

### 2.2 สร้าง Custom Floor แทน Baseplate

```lua
-- Script: CreateCustomFloor
-- วางใน: ServerScriptService

-- ลบ Baseplate เดิม
local oldBaseplate = workspace:FindFirstChild("Baseplate")
if oldBaseplate then
    oldBaseplate:Destroy()
end

-- สร้าง Floor ใหม่แบบ Tile
local floorFolder = Instance.new("Folder")
floorFolder.Name = "Floor"
floorFolder.Parent = workspace

local tileSize = 10   -- ขนาดของแต่ละ Tile
local mapWidth = 20   -- จำนวน Tiles กว้าง
local mapDepth = 20   -- จำนวน Tiles ลึก

-- Colors สำหรับ Pattern
local colors = {
    BrickColor.new("Medium stone grey"),
    BrickColor.new("Dark stone grey"),
}

for x = 0, mapWidth - 1 do
    for z = 0, mapDepth - 1 do
        local tile = Instance.new("Part")
        tile.Name = "Tile_" .. x .. "_" .. z
        tile.Size = Vector3.new(tileSize, 1, tileSize)
        tile.Position = Vector3.new(
            (x - mapWidth/2) * tileSize + tileSize/2,
            0,
            (z - mapDepth/2) * tileSize + tileSize/2
        )
        
        -- สร้าง Checkerboard Pattern
        local colorIndex = ((x + z) % 2) + 1
        tile.BrickColor = colors[colorIndex]
        tile.Material = Enum.Material.SmoothPlastic
        tile.Anchored = true
        tile.CastShadow = false
        tile.Parent = floorFolder
    end
end

print("สร้าง Custom Floor เรียบร้อย!")
print("Tiles:", mapWidth * mapDepth)
```

---

## 3. SpawnLocation

### 3.1 SpawnLocation คืออะไร?

**SpawnLocation** คือจุดที่ผู้เล่น Respawn หลังจากตาย:

```lua
-- ดู SpawnLocation เริ่มต้น
local spawn = workspace:FindFirstChild("SpawnLocation")
if spawn then
    print("Spawn at:", spawn.Position)
    print("Spawn size:", spawn.Size)
    print("Spawn Team:", spawn.TeamColor)
end
```

### 3.2 ปรับแต่ง SpawnLocation

```lua
-- Script: ConfigureSpawn
-- วางใน: ServerScriptService

local spawn = workspace:FindFirstChild("SpawnLocation")
if spawn then
    -- ตำแหน่ง
    spawn.Position = Vector3.new(0, 1.5, 0)
    
    -- ขนาด
    spawn.Size = Vector3.new(6, 1, 6)
    
    -- สี
    spawn.BrickColor = BrickColor.new("Bright green")
    
    -- Respawn Cooldown
    spawn.Duration = 0  -- 0 = Respawn ทันที
    
    -- Team
    spawn.Neutral = true  -- true = ทุก Team ใช้ได้
end
```

### 3.3 สร้าง SpawnLocations หลายจุด

```lua
-- Script: MultipleSpawns
-- วางใน: ServerScriptService

local function createSpawn(name, position, color)
    -- ลบ SpawnLocation เดิม
    local existing = workspace:FindFirstChild(name)
    if existing then existing:Destroy() end
    
    local spawn = Instance.new("SpawnLocation")
    spawn.Name = name
    spawn.Size = Vector3.new(6, 1, 6)
    spawn.Position = position
    spawn.BrickColor = color
    spawn.Neutral = true
    spawn.Duration = 0
    spawn.Anchored = true
    spawn.Material = Enum.Material.SmoothPlastic
    spawn.Parent = workspace
    
    return spawn
end

-- สร้าง Spawn หลายจุด
createSpawn("Spawn1", Vector3.new(0, 1.5, 0), BrickColor.new("Bright green"))
createSpawn("Spawn2", Vector3.new(50, 1.5, 0), BrickColor.new("Bright blue"))
createSpawn("Spawn3", Vector3.new(-50, 1.5, 0), BrickColor.new("Bright yellow"))
createSpawn("Spawn4", Vector3.new(0, 1.5, 50), BrickColor.new("Bright orange"))

print("สร้าง SpawnLocations เรียบร้อย!")
```

---

## 4. สร้าง Environment พื้นฐาน

### 4.1 Sky Box

```lua
-- เพิ่ม Sky ให้กับ Lighting
local Lighting = game:GetService("Lighting")

local sky = Instance.new("Sky")
sky.SkyboxBk = "rbxassetid://159454291"  -- Back
sky.SkyboxDn = "rbxassetid://159454296"  -- Down  
sky.SkyboxFt = "rbxassetid://159454293"  -- Front
sky.SkyboxLf = "rbxassetid://159454297"  -- Left
sky.SkyboxRt = "rbxassetid://159454295"  -- Right
sky.SkyboxUp = "rbxassetid://159454292"  -- Up
sky.Parent = Lighting
```

### 4.2 Atmosphere

```lua
-- เพิ่ม Atmosphere Effects
local Lighting = game:GetService("Lighting")

local atmosphere = Instance.new("Atmosphere")
atmosphere.Density = 0.3          -- ความหนาแน่น Haze
atmosphere.Offset = 0.25          -- Offset ของ Haze
atmosphere.Color = Color3.fromRGB(199, 170, 107)  -- สี Haze
atmosphere.Decay = Color3.fromRGB(106, 112, 125)  -- สีที่ลางลง
atmosphere.Glare = 0              -- ความสว่างรอบดวงอาทิตย์
atmosphere.Haze = 0               -- ความหมอก
atmosphere.Parent = Lighting
```

### 4.3 Bloom Effect

```lua
-- เพิ่ม Bloom Post Processing
local Lighting = game:GetService("Lighting")

local bloom = Instance.new("BloomEffect")
bloom.Intensity = 0.5
bloom.Size = 24
bloom.Threshold = 0.95
bloom.Parent = Lighting

-- Color Correction
local colorCorrection = Instance.new("ColorCorrectionEffect")
colorCorrection.Brightness = 0
colorCorrection.Contrast = 0
colorCorrection.Saturation = 0
colorCorrection.TintColor = Color3.fromRGB(255, 255, 255)
colorCorrection.Parent = Lighting
```

---

## 5. สร้าง Walls และ Boundaries

### 5.1 กำแพงล้อมรอบ Map

```lua
-- Script: CreateMapBoundaries
-- วางใน: ServerScriptService

local function createWall(name, size, position, color)
    local wall = Instance.new("Part")
    wall.Name = name
    wall.Size = size
    wall.Position = position
    wall.BrickColor = color or BrickColor.new("Medium stone grey")
    wall.Material = Enum.Material.SmoothPlastic
    wall.Anchored = true
    wall.CanCollide = true
    wall.Parent = workspace
    return wall
end

-- Map ขนาด 200 x 200
local mapSize = 200
local wallHeight = 20
local wallThickness = 4

-- กำแพง 4 ด้าน
-- ด้านหน้า (Front)
createWall("Wall_Front", 
    Vector3.new(mapSize + wallThickness * 2, wallHeight, wallThickness),
    Vector3.new(0, wallHeight/2, -mapSize/2 - wallThickness/2)
)

-- ด้านหลัง (Back)
createWall("Wall_Back",
    Vector3.new(mapSize + wallThickness * 2, wallHeight, wallThickness),
    Vector3.new(0, wallHeight/2, mapSize/2 + wallThickness/2)
)

-- ด้านซ้าย (Left)
createWall("Wall_Left",
    Vector3.new(wallThickness, wallHeight, mapSize),
    Vector3.new(-mapSize/2 - wallThickness/2, wallHeight/2, 0)
)

-- ด้านขวา (Right)
createWall("Wall_Right",
    Vector3.new(wallThickness, wallHeight, mapSize),
    Vector3.new(mapSize/2 + wallThickness/2, wallHeight/2, 0)
)

print("สร้างกำแพงเรียบร้อย!")
```

---

## 6. สร้าง Obstacles

### 6.1 Platform Jump Sequence

```lua
-- Script: CreatePlatforms
-- วางใน: ServerScriptService

local function createPlatform(pos, size, color)
    local platform = Instance.new("Part")
    platform.Size = size or Vector3.new(6, 1, 6)
    platform.Position = pos
    platform.BrickColor = color or BrickColor.new("Bright blue")
    platform.Material = Enum.Material.SmoothPlastic
    platform.Anchored = true
    platform.Parent = workspace
    return platform
end

-- สร้าง Platforms แบบ Staircase
local platforms = {
    {Vector3.new(0, 2, 10), Vector3.new(6, 1, 6)},
    {Vector3.new(8, 4, 18), Vector3.new(5, 1, 5)},
    {Vector3.new(0, 6, 26), Vector3.new(6, 1, 6)},
    {Vector3.new(-8, 8, 34), Vector3.new(5, 1, 5)},
    {Vector3.new(0, 10, 42), Vector3.new(8, 1, 8)},  -- Final Platform
}

local colors = {
    BrickColor.new("Bright blue"),
    BrickColor.new("Bright green"),
    BrickColor.new("Bright yellow"),
    BrickColor.new("Bright orange"),
    BrickColor.new("Bright red"),
}

for i, data in ipairs(platforms) do
    local p = createPlatform(data[1], data[2], colors[i])
    p.Name = "Platform_" .. i
end

print("สร้าง Platforms เรียบร้อย!")
```

### 6.2 สร้าง Obstacles หมุน

```lua
-- Script: RotatingObstacle
-- วางใน: ServerScriptService

local RunService = game:GetService("RunService")

-- สร้าง Spinning Obstacle
local obstacle = Instance.new("Part")
obstacle.Name = "SpinningObstacle"
obstacle.Size = Vector3.new(20, 1, 2)
obstacle.Position = Vector3.new(0, 3, 20)
obstacle.BrickColor = BrickColor.new("Bright red")
obstacle.Material = Enum.Material.Neon
obstacle.Anchored = true
obstacle.Parent = workspace

-- หมุนต่อเนื่อง
local angle = 0
local spinSpeed = 60  -- องศาต่อวินาที

RunService.Heartbeat:Connect(function(dt)
    angle = angle + spinSpeed * dt
    obstacle.CFrame = CFrame.new(0, 3, 20) * CFrame.Angles(0, math.rad(angle), 0)
end)

print("สร้าง Rotating Obstacle เรียบร้อย!")
```

### 6.3 Moving Platform

```lua
-- Script: MovingPlatform
-- วางใน: ServerScriptService

local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")

-- สร้าง Moving Platform
local platform = Instance.new("Part")
platform.Name = "MovingPlatform"
platform.Size = Vector3.new(8, 1, 8)
platform.Position = Vector3.new(-20, 3, 0)
platform.BrickColor = BrickColor.new("Bright green")
platform.Material = Enum.Material.SmoothPlastic
platform.Anchored = true
platform.Parent = workspace

-- Animation ไปกลับ
local startPos = Vector3.new(-20, 3, 0)
local endPos = Vector3.new(20, 3, 0)
local moveDuration = 3

local tweenInfo = TweenInfo.new(
    moveDuration,
    Enum.EasingStyle.Sine,
    Enum.EasingDirection.InOut,
    -1,       -- -1 = Loop ไม่มีที่สิ้นสุด
    true,     -- true = Reverses กลับ
    0         -- Delay
)

local tween = TweenService:Create(platform, tweenInfo, {
    Position = endPos
})

tween:Play()
print("Moving Platform กำลังเคลื่อนที่!")
```

---

## 7. สร้าง Decorations

### 7.1 ต้นไม้

```lua
-- Script: CreateTree
-- วางใน: ServerScriptService

local function createTree(position, trunkColor, leavesColor)
    local tree = Instance.new("Model")
    tree.Name = "Tree"
    tree.Parent = workspace
    
    -- ลำต้น (Trunk)
    local trunk = Instance.new("Part")
    trunk.Name = "Trunk"
    trunk.Shape = Enum.PartType.Cylinder
    trunk.Size = Vector3.new(6, 1.5, 1.5)
    trunk.CFrame = CFrame.new(position + Vector3.new(0, 3, 0)) * CFrame.Angles(0, 0, math.pi/2)
    trunk.BrickColor = trunkColor or BrickColor.new("Reddish brown")
    trunk.Material = Enum.Material.Wood
    trunk.Anchored = true
    trunk.Parent = tree
    
    -- ใบ (Leaves) - หลายชั้น
    local leafData = {
        {size = Vector3.new(7, 5, 7), yOffset = 7},
        {size = Vector3.new(6, 4, 6), yOffset = 10},
        {size = Vector3.new(4, 3, 4), yOffset = 13},
    }
    
    for i, data in ipairs(leafData) do
        local leaves = Instance.new("Part")
        leaves.Name = "Leaves_" .. i
        leaves.Shape = Enum.PartType.Ball
        leaves.Size = data.size
        leaves.Position = position + Vector3.new(0, data.yOffset, 0)
        leaves.BrickColor = leavesColor or BrickColor.new("Bright green")
        leaves.Material = Enum.Material.Grass
        leaves.Anchored = true
        leaves.CastShadow = false
        leaves.Parent = tree
    end
    
    tree.PrimaryPart = trunk
    return tree
end

-- วางต้นไม้รอบๆ
local treePositions = {
    Vector3.new(30, 0, 30),
    Vector3.new(-30, 0, 30),
    Vector3.new(30, 0, -30),
    Vector3.new(-30, 0, -30),
    Vector3.new(50, 0, 0),
    Vector3.new(-50, 0, 0),
}

for _, pos in ipairs(treePositions) do
    createTree(pos)
end

print("สร้างต้นไม้เรียบร้อย!")
```

### 7.2 หิน (Rocks)

```lua
-- Script: CreateRocks
-- วางใน: ServerScriptService

local function createRock(position, size, rotation)
    local rock = Instance.new("Part")
    rock.Name = "Rock"
    rock.Shape = Enum.PartType.Ball
    rock.Size = size or Vector3.new(
        math.random(2, 5),
        math.random(1, 3),
        math.random(2, 5)
    )
    rock.CFrame = CFrame.new(position) * CFrame.Angles(
        math.random() * math.pi,
        math.random() * math.pi,
        math.random() * math.pi
    )
    rock.BrickColor = BrickColor.new("Dark stone grey")
    rock.Material = Enum.Material.Rock
    rock.Anchored = true
    rock.Parent = workspace
    return rock
end

-- วางหินแบบ Random
math.randomseed(os.time())
for i = 1, 15 do
    local x = math.random(-90, 90)
    local z = math.random(-90, 90)
    
    -- หลีกเลี่ยงตรงกลาง
    if math.abs(x) > 15 or math.abs(z) > 15 then
        createRock(Vector3.new(x, 1, z))
    end
end

print("สร้างหินเรียบร้อย!")
```

### 7.3 หลอดไฟ (Lamp Posts)

```lua
-- Script: CreateLampPosts
-- วางใน: ServerScriptService

local function createLampPost(position)
    local lampModel = Instance.new("Model")
    lampModel.Name = "LampPost"
    lampModel.Parent = workspace
    
    -- เสา
    local pole = Instance.new("Part")
    pole.Name = "Pole"
    pole.Shape = Enum.PartType.Cylinder
    pole.Size = Vector3.new(8, 0.4, 0.4)
    pole.CFrame = CFrame.new(position + Vector3.new(0, 4, 0)) * CFrame.Angles(0, 0, math.pi/2)
    pole.BrickColor = BrickColor.new("Dark stone grey")
    pole.Material = Enum.Material.Metal
    pole.Anchored = true
    pole.Parent = lampModel
    
    -- หัวโคม
    local lampHead = Instance.new("Part")
    lampHead.Name = "LampHead"
    lampHead.Size = Vector3.new(1.5, 1, 1.5)
    lampHead.Position = position + Vector3.new(0, 8.5, 0)
    lampHead.BrickColor = BrickColor.new("Dark stone grey")
    lampHead.Material = Enum.Material.Metal
    lampHead.Anchored = true
    lampHead.Parent = lampModel
    
    -- แสง
    local light = Instance.new("PointLight")
    light.Brightness = 5
    light.Color = Color3.fromRGB(255, 220, 150)  -- สีเหลืองอุ่น
    light.Range = 20
    light.Parent = lampHead
    
    -- หลอดไฟ (Neon Part)
    local bulb = Instance.new("Part")
    bulb.Name = "Bulb"
    bulb.Shape = Enum.PartType.Ball
    bulb.Size = Vector3.new(0.8, 0.8, 0.8)
    bulb.Position = position + Vector3.new(0, 8.2, 0)
    bulb.BrickColor = BrickColor.new("Bright yellow")
    bulb.Material = Enum.Material.Neon
    bulb.Anchored = true
    bulb.CastShadow = false
    bulb.Parent = lampModel
    
    lampModel.PrimaryPart = pole
    return lampModel
end

-- วาง Lamp Posts รอบถนน
local lampPositions = {
    Vector3.new(15, 0, 0),
    Vector3.new(-15, 0, 0),
    Vector3.new(15, 0, 30),
    Vector3.new(-15, 0, 30),
    Vector3.new(15, 0, -30),
    Vector3.new(-15, 0, -30),
}

for _, pos in ipairs(lampPositions) do
    createLampPost(pos)
end

print("สร้าง Lamp Posts เรียบร้อย!")
```

---

## 8. ถนนและเส้นทาง

### 8.1 สร้างถนน

```lua
-- Script: CreateRoad
-- วางใน: ServerScriptService

local function createRoad(startPos, endPos, width)
    width = width or 8
    
    -- คำนวณขนาดและทิศทาง
    local direction = (endPos - startPos)
    local length = direction.Magnitude
    local center = (startPos + endPos) / 2
    
    -- สร้าง Road Part
    local road = Instance.new("Part")
    road.Name = "Road"
    road.Size = Vector3.new(width, 0.5, length)
    road.CFrame = CFrame.lookAt(center, center + direction) * CFrame.Angles(0, math.pi/2, 0)
    road.BrickColor = BrickColor.new("Dark stone grey")
    road.Material = Enum.Material.SmoothPlastic
    road.Anchored = true
    road.Parent = workspace
    
    -- เส้นกลางถนน
    local centerLine = Instance.new("Part")
    centerLine.Size = Vector3.new(0.3, 0.6, length)
    centerLine.CFrame = CFrame.lookAt(center + Vector3.new(0, 0.3, 0), center + direction + Vector3.new(0, 0.3, 0)) * CFrame.Angles(0, math.pi/2, 0)
    centerLine.BrickColor = BrickColor.new("Bright yellow")
    centerLine.Material = Enum.Material.SmoothPlastic
    centerLine.Anchored = true
    centerLine.CastShadow = false
    centerLine.Parent = workspace
    
    return road
end

-- สร้างถนนหลาย Segment
createRoad(Vector3.new(-50, 0.25, 0), Vector3.new(50, 0.25, 0))    -- ซ้ายไปขวา
createRoad(Vector3.new(0, 0.25, -50), Vector3.new(0, 0.25, 50))    -- หน้าไปหลัง
createRoad(Vector3.new(-50, 0.25, 30), Vector3.new(50, 0.25, 30))  -- ถนนขนาน
```

---

## 9. Checkpoint System

### 9.1 สร้าง Checkpoints

```lua
-- Script: CheckpointSystem
-- วางใน: ServerScriptService

local Players = game:GetService("Players")
local checkpointData = {}  -- เก็บ Checkpoint ของแต่ละ Player

-- สร้าง Checkpoint
local function createCheckpoint(id, position)
    local checkpoint = Instance.new("Part")
    checkpoint.Name = "Checkpoint_" .. id
    checkpoint.Size = Vector3.new(6, 8, 1)
    checkpoint.Position = position
    checkpoint.BrickColor = BrickColor.new("Bright blue")
    checkpoint.Material = Enum.Material.Neon
    checkpoint.Transparency = 0.5
    checkpoint.Anchored = true
    checkpoint.CanCollide = false
    checkpoint.Parent = workspace
    
    -- เพิ่ม BillboardGui แสดงหมายเลข
    local billboard = Instance.new("BillboardGui")
    billboard.Size = UDim2.new(0, 100, 0, 50)
    billboard.StudsOffset = Vector3.new(0, 5, 0)
    billboard.Parent = checkpoint
    
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, 0, 1, 0)
    label.Text = "Checkpoint " .. id
    label.TextColor3 = Color3.fromRGB(255, 255, 255)
    label.BackgroundTransparency = 1
    label.TextScaled = true
    label.Parent = billboard
    
    -- ตรวจจับ Touch
    checkpoint.Touched:Connect(function(hit)
        local character = hit.Parent
        local player = Players:GetPlayerFromCharacter(character)
        
        if player then
            -- ตรวจสอบว่าเป็น Checkpoint ใหม่ (หมายเลขสูงกว่า)
            local currentCheckpoint = checkpointData[player.UserId] or 0
            if id > currentCheckpoint then
                checkpointData[player.UserId] = id
                print(player.Name .. " ถึง Checkpoint " .. id)
                
                -- เปลี่ยนสีเป็นเขียว
                checkpoint.BrickColor = BrickColor.new("Bright green")
                
                -- บันทึกตำแหน่ง Spawn ใหม่
                -- (ต้องใช้ร่วมกับระบบ Respawn)
            end
        end
    end)
    
    return checkpoint
end

-- สร้าง Checkpoints
createCheckpoint(1, Vector3.new(0, 4, 20))
createCheckpoint(2, Vector3.new(0, 4, 50))
createCheckpoint(3, Vector3.new(0, 4, 80))
createCheckpoint(4, Vector3.new(0, 4, 110))

print("สร้าง Checkpoints เรียบร้อย!")
```

---

## 10. Kill Zones

### 10.1 สร้าง Kill Zone

```lua
-- Script: KillZone
-- วางใน: ServerScriptService

local Players = game:GetService("Players")

local function createKillZone(name, size, position, color)
    local zone = Instance.new("Part")
    zone.Name = name
    zone.Size = size
    zone.Position = position
    zone.BrickColor = color or BrickColor.new("Bright red")
    zone.Material = Enum.Material.Neon
    zone.Transparency = 0.6
    zone.Anchored = true
    zone.CanCollide = false
    zone.Parent = workspace
    
    -- ตรวจจับ Touch
    zone.Touched:Connect(function(hit)
        local character = hit.Parent
        local humanoid = character:FindFirstChildOfClass("Humanoid")
        
        if humanoid and humanoid.Health > 0 then
            humanoid.Health = 0  -- ฆ่าตัวละคร
        end
    end)
    
    return zone
end

-- สร้าง Lava Floor
createKillZone(
    "LavaFloor",
    Vector3.new(200, 5, 200),
    Vector3.new(0, -10, 0),
    BrickColor.new("Bright orange")
)

-- สร้าง Void Zone (ตกน้ำ)
createKillZone(
    "VoidZone",
    Vector3.new(1000, 5, 1000),
    Vector3.new(0, -200, 0),
    BrickColor.new("Navy blue")
)

print("สร้าง Kill Zones เรียบร้อย!")
```

---

## 11. โปรเจกต์: สร้าง Complete Level

```lua
-- Script: BuildCompleteLevel
-- วางใน: ServerScriptService

-- ===== COMPLETE LEVEL BUILDER =====
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local Players = game:GetService("Players")

print("กำลังสร้าง Level...")

-- ===== 1. สร้างพื้น (Floor) =====
local floor = Instance.new("Part")
floor.Name = "Floor"
floor.Size = Vector3.new(100, 1, 100)
floor.Position = Vector3.new(0, 0, 0)
floor.BrickColor = BrickColor.new("Medium stone grey")
floor.Material = Enum.Material.SmoothPlastic
floor.Anchored = true
floor.Parent = workspace

-- ===== 2. สร้าง Walls =====
local wallsFolder = Instance.new("Folder")
wallsFolder.Name = "Walls"
wallsFolder.Parent = workspace

local wallConfigs = {
    {size = Vector3.new(100, 20, 2), pos = Vector3.new(0, 10, 51)},   -- หน้า
    {size = Vector3.new(100, 20, 2), pos = Vector3.new(0, 10, -51)},  -- หลัง
    {size = Vector3.new(2, 20, 100), pos = Vector3.new(51, 10, 0)},   -- ขวา
    {size = Vector3.new(2, 20, 100), pos = Vector3.new(-51, 10, 0)},  -- ซ้าย
}

for i, config in ipairs(wallConfigs) do
    local wall = Instance.new("Part")
    wall.Name = "Wall_" .. i
    wall.Size = config.size
    wall.Position = config.pos
    wall.BrickColor = BrickColor.new("White")
    wall.Material = Enum.Material.SmoothPlastic
    wall.Anchored = true
    wall.Parent = wallsFolder
end

-- ===== 3. สร้าง Platforms =====
local platformsFolder = Instance.new("Folder")
platformsFolder.Name = "Platforms"
platformsFolder.Parent = workspace

local platformData = {
    {pos = Vector3.new(0, 2, 20), size = Vector3.new(8, 1, 8), color = "Bright blue"},
    {pos = Vector3.new(15, 4, 30), size = Vector3.new(6, 1, 6), color = "Bright green"},
    {pos = Vector3.new(0, 6, 40), size = Vector3.new(8, 1, 8), color = "Bright yellow"},
    {pos = Vector3.new(-15, 8, 30), size = Vector3.new(6, 1, 6), color = "Bright orange"},
    {pos = Vector3.new(0, 10, 20), size = Vector3.new(10, 1, 10), color = "Bright red"},
}

for i, data in ipairs(platformData) do
    local p = Instance.new("Part")
    p.Name = "Platform_" .. i
    p.Size = data.size
    p.Position = data.pos
    p.BrickColor = BrickColor.new(data.color)
    p.Material = Enum.Material.SmoothPlastic
    p.Anchored = true
    p.Parent = platformsFolder
end

-- ===== 4. Moving Platform =====
local movingPlatform = Instance.new("Part")
movingPlatform.Name = "MovingPlatform"
movingPlatform.Size = Vector3.new(8, 1, 8)
movingPlatform.Position = Vector3.new(-20, 5, 0)
movingPlatform.BrickColor = BrickColor.new("Cyan")
movingPlatform.Material = Enum.Material.Neon
movingPlatform.Anchored = true
movingPlatform.Parent = workspace

-- Tween Moving Platform
local tweenInfo = TweenInfo.new(3, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, true)
TweenService:Create(movingPlatform, tweenInfo, {Position = Vector3.new(20, 5, 0)}):Play()

-- ===== 5. Kill Zone =====
local lava = Instance.new("Part")
lava.Name = "Lava"
lava.Size = Vector3.new(200, 2, 200)
lava.Position = Vector3.new(0, -5, 0)
lava.BrickColor = BrickColor.new("Neon orange")
lava.Material = Enum.Material.Neon
lava.Transparency = 0.3
lava.Anchored = true
lava.CanCollide = false
lava.Parent = workspace

lava.Touched:Connect(function(hit)
    local humanoid = hit.Parent:FindFirstChildOfClass("Humanoid")
    if humanoid then humanoid.Health = 0 end
end)

-- ===== 6. Finish Line =====
local finish = Instance.new("Part")
finish.Name = "FinishLine"
finish.Size = Vector3.new(12, 8, 1)
finish.Position = Vector3.new(0, 14.5, 20)
finish.BrickColor = BrickColor.new("Bright yellow")
finish.Material = Enum.Material.Neon
finish.Transparency = 0.5
finish.Anchored = true
finish.CanCollide = false
finish.Parent = workspace

finish.Touched:Connect(function(hit)
    local player = Players:GetPlayerFromCharacter(hit.Parent)
    if player then
        print(player.Name .. " ชนะ! เข้าเส้นชัยแล้ว!")
    end
end)

print("Level สร้างเรียบร้อย!")
print("เริ่มเล่นโดยกด Play (F5)")
```

---

## 📚 แบบฝึกหัดตอนที่ 7

### แบบฝึกหัดที่ 1: Baseplate ปรับแต่ง
1. ปรับขนาด Baseplate ตามที่คุณต้องการ
2. เปลี่ยน Material เป็น Grass
3. เปลี่ยนสีและดูผลลัพธ์

### แบบฝึกหัดที่ 2: SpawnLocation
1. ลบ SpawnLocation เดิม
2. สร้าง SpawnLocation ใหม่ที่ตำแหน่งต่างๆ
3. ทดสอบโดยกด Play

### แบบฝึกหัดที่ 3: Decorations
1. สร้างต้นไม้ 5 ต้น
2. สร้างหิน 3 ก้อน
3. สร้าง Lamp Post 2 ต้น

### แบบฝึกหัดที่ 4: Obstacles
1. สร้าง Platforms แบบ Staircase
2. สร้าง Moving Platform
3. สร้าง Rotating Obstacle

### แบบฝึกหัดที่ 5: Complete Level
1. คัดลอกโค้ด Complete Level
2. รันและทดสอบ
3. ปรับแต่งตาม Style ที่ชอบ

---

## 💡 เคล็ดลับ

1. **Plan ก่อนสร้าง** - วาง Layout บนกระดาษก่อน
2. **ใช้ Grid** - ทำให้ Parts Align กันสวยงาม
3. **Group เป็น Models** - จัดระเบียบง่าย
4. **ทดสอบบ่อยๆ** - กด Play เพื่อทดสอบ
5. **คิดถึง Player Experience** - เกมต้องสนุกและยุติธรรม

---

## ⏭️ ตอนถัดไป

ในตอนที่ 8 เราจะเรียนรู้:
- การสร้าง Terrain เบื้องต้น
- Terrain Tools ต่างๆ
- การสร้างภูมิประเทศ

---

*ตอนที่ 7/100 | ระดับ: พื้นฐาน | เวลา: 90-120 นาที*
