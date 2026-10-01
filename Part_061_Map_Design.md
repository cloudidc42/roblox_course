# Part 61: การออกแบบแผนที่และด่าน (Map and Level Design)

## บทนำ

การออกแบบแผนที่และด่านเป็นหนึ่งในทักษะที่สำคัญที่สุดในการพัฒนาเกม Roblox ที่ดี แผนที่ที่ออกแบบมาอย่างดีจะทำให้ผู้เล่นรู้สึกสนุกสนาน ท้าทาย และต้องการกลับมาเล่นอีก ในบทนี้เราจะเรียนรู้หลักการออกแบบแผนที่อย่างมืออาชีพ

---

## 61.1 หลักการพื้นฐานของการออกแบบแผนที่

### 61.1.1 Flow (การไหลของการเล่น)

Flow หมายถึงเส้นทางที่ผู้เล่นจะเดินผ่านแผนที่ของคุณ ควรออกแบบให้:

- **ชัดเจน**: ผู้เล่นรู้ว่าต้องไปทางไหน
- **ลื่นไหล**: ไม่มีจุดที่ทำให้ผู้เล่นติดหรือสับสน
- **หลากหลาย**: มีหลายเส้นทางให้เลือก

```lua
-- ตัวอย่าง: ระบบ Waypoint สำหรับนำทางผู้เล่น
-- Script นี้วางใน ServerScriptService

local waypoints = {
    {position = Vector3.new(0, 5, 0), label = "จุดเริ่มต้น"},
    {position = Vector3.new(50, 5, 0), label = "พื้นที่กลาง"},
    {position = Vector3.new(100, 5, 50), label = "บอสโซน"},
    {position = Vector3.new(150, 5, 50), label = "จุดสิ้นสุด"},
}

-- สร้าง visual waypoints
local function createWaypoints()
    local folder = Instance.new("Folder")
    folder.Name = "Waypoints"
    folder.Parent = workspace
    
    for i, waypoint in ipairs(waypoints) do
        local part = Instance.new("Part")
        part.Name = "Waypoint_" .. i
        part.Size = Vector3.new(2, 0.2, 2)
        part.Position = waypoint.position
        part.Anchored = true
        part.CanCollide = false
        part.BrickColor = BrickColor.new("Bright yellow")
        part.Material = Enum.Material.Neon
        part.Parent = folder
        
        -- เพิ่ม BillboardGui แสดงชื่อ
        local billboard = Instance.new("BillboardGui")
        billboard.Size = UDim2.new(0, 200, 0, 50)
        billboard.StudsOffset = Vector3.new(0, 3, 0)
        billboard.Parent = part
        
        local label = Instance.new("TextLabel")
        label.Size = UDim2.new(1, 0, 1, 0)
        label.BackgroundTransparency = 1
        label.Text = waypoint.label
        label.TextColor3 = Color3.new(1, 1, 0)
        label.TextScaled = true
        label.Font = Enum.Font.GothamBold
        label.Parent = billboard
    end
end

createWaypoints()
print("สร้าง Waypoints เสร็จแล้ว!")
```

### 61.1.2 Pacing (จังหวะการเล่น)

Pacing คือการควบคุมความตึงเครียดและการพักผ่อนของผู้เล่น:

```lua
-- ระบบ Zone ที่มี Pacing ต่างกัน
-- ServerScriptService/ZoneManager

local ZoneManager = {}

-- กำหนดโซนต่างๆ
local zones = {
    {
        name = "Safe Zone",        -- โซนปลอดภัย
        color = Color3.fromRGB(100, 200, 100),
        spawnRate = 0,             -- ไม่มีศัตรู
        musicId = "rbxassetid://1234567",
        ambience = "peaceful"
    },
    {
        name = "Combat Zone",      -- โซนต่อสู้
        color = Color3.fromRGB(200, 100, 100),
        spawnRate = 5,             -- ศัตรูออกทุก 5 วินาที
        musicId = "rbxassetid://2345678",
        ambience = "tense"
    },
    {
        name = "Puzzle Zone",      -- โซนปริศนา
        color = Color3.fromRGB(100, 100, 200),
        spawnRate = 0,
        musicId = "rbxassetid://3456789",
        ambience = "mysterious"
    },
    {
        name = "Boss Zone",        -- โซนบอส
        color = Color3.fromRGB(200, 50, 50),
        spawnRate = 30,            -- บอสออกทุก 30 วินาที
        musicId = "rbxassetid://4567890",
        ambience = "epic"
    }
}

-- ฟังก์ชันเปลี่ยนบรรยากาศตามโซน
function ZoneManager.enterZone(player, zoneName)
    for _, zone in ipairs(zones) do
        if zone.name == zoneName then
            -- เปลี่ยนไฟพื้นหลัง
            local lighting = game:GetService("Lighting")
            if zone.ambience == "peaceful" then
                lighting.Brightness = 2
                lighting.Ambient = Color3.fromRGB(150, 150, 200)
            elseif zone.ambience == "tense" then
                lighting.Brightness = 1
                lighting.Ambient = Color3.fromRGB(200, 100, 100)
            elseif zone.ambience == "mysterious" then
                lighting.Brightness = 0.5
                lighting.Ambient = Color3.fromRGB(50, 50, 100)
            elseif zone.ambience == "epic" then
                lighting.Brightness = 1.5
                lighting.Ambient = Color3.fromRGB(200, 50, 50)
            end
            
            print(player.Name .. " เข้าสู่ " .. zone.name)
            break
        end
    end
end

return ZoneManager
```

---

## 61.2 การออกแบบภูมิประเทศ (Terrain Design)

### 61.2.1 Terrain API

Roblox มี Terrain API ที่ทรงพลังสำหรับสร้างภูมิประเทศ:

```lua
-- Script สร้างภูมิประเทศแบบ Procedural
-- RunService ใน ServerScriptService

local terrain = workspace.Terrain

-- ฟังก์ชันสร้างเนินเขา
local function createHill(centerX, centerZ, radius, height)
    for x = centerX - radius, centerX + radius, 4 do
        for z = centerZ - radius, centerZ + radius, 4 do
            local distance = math.sqrt((x - centerX)^2 + (z - centerZ)^2)
            if distance <= radius then
                -- คำนวณความสูงแบบ Gaussian
                local hillHeight = height * math.exp(-(distance^2) / (2 * (radius/3)^2))
                
                -- เติมดิน
                local region = Region3.new(
                    Vector3.new(x - 2, 0, z - 2),
                    Vector3.new(x + 2, hillHeight, z + 2)
                )
                terrain:FillBlock(
                    CFrame.new(x, hillHeight/2, z),
                    Vector3.new(4, hillHeight, 4),
                    Enum.Material.Grass
                )
            end
        end
    end
end

-- ฟังก์ชันสร้างแม่น้ำ
local function createRiver(startX, startZ, endX, endZ, width, depth)
    local steps = 50
    for i = 0, steps do
        local t = i / steps
        local x = startX + (endX - startX) * t
        local z = startZ + (endZ - startZ) * t
        
        -- เพิ่มความคดเคี้ยวแบบ Sine
        local waveOffset = math.sin(t * math.pi * 4) * 10
        
        terrain:FillCylinder(
            CFrame.new(x + waveOffset, -depth/2, z),
            depth,
            width,
            Enum.Material.Water
        )
    end
end

-- สร้างแผนที่พื้นฐาน
local function generateMap()
    -- สร้างพื้นดินพื้นฐาน
    terrain:FillBlock(
        CFrame.new(0, -5, 0),
        Vector3.new(500, 10, 500),
        Enum.Material.Grass
    )
    
    -- สร้างเนินเขาหลายลูก
    local hills = {
        {x = 100, z = 100, r = 40, h = 30},
        {x = -80, z = 150, r = 50, h = 45},
        {x = 200, z = -50, r = 35, h = 25},
        {x = -150, z = -100, r = 60, h = 50},
    }
    
    for _, hill in ipairs(hills) do
        createHill(hill.x, hill.z, hill.r, hill.h)
        print("สร้างเนินเขาที่ " .. hill.x .. ", " .. hill.z)
    end
    
    -- สร้างแม่น้ำ
    createRiver(-200, 0, 200, 0, 15, 5)
    print("สร้างแม่น้ำเสร็จแล้ว")
    
    print("สร้างแผนที่เสร็จสมบูรณ์!")
end

generateMap()
```

### 61.2.2 การวางต้นไม้และสิ่งประดับตกแต่ง

```lua
-- ระบบวางต้นไม้แบบ Procedural
-- ServerScriptService/PropPlacer

local PropPlacer = {}

-- กำหนด Props ต่างๆ
local propTypes = {
    {
        name = "Tree_Oak",          -- ต้นโอ๊ก
        folder = "Trees",
        density = 0.3,              -- ความหนาแน่น 0-1
        minScale = 0.8,
        maxScale = 1.5,
        requiresMaterial = {Enum.Material.Grass, Enum.Material.Ground}
    },
    {
        name = "Rock_Large",        -- หินใหญ่
        folder = "Rocks",
        density = 0.1,
        minScale = 0.5,
        maxScale = 2.0,
        requiresMaterial = {Enum.Material.Rock, Enum.Material.Grass}
    },
    {
        name = "Flower_Red",        -- ดอกไม้แดง
        folder = "Plants",
        density = 0.5,
        minScale = 0.5,
        maxScale = 1.0,
        requiresMaterial = {Enum.Material.Grass}
    }
}

-- ฟังก์ชันวาง prop แบบ random
function PropPlacer.scatterProps(areaSize, propName)
    local folder = workspace:FindFirstChild("Props") or Instance.new("Folder")
    folder.Name = "Props"
    folder.Parent = workspace
    
    -- หา template
    local template = game.ReplicatedStorage:FindFirstChild(propName)
    if not template then
        warn("ไม่พบ template สำหรับ " .. propName)
        return
    end
    
    local count = 0
    local maxProps = 100
    
    while count < maxProps do
        -- สุ่มตำแหน่ง
        local x = math.random(-areaSize/2, areaSize/2)
        local z = math.random(-areaSize/2, areaSize/2)
        
        -- ตรวจสอบพื้นดิน
        local ray = workspace:Raycast(
            Vector3.new(x, 100, z),
            Vector3.new(0, -200, 0)
        )
        
        if ray and ray.Instance then
            local prop = template:Clone()
            local scale = math.random(80, 150) / 100  -- 0.8 - 1.5
            
            prop:SetPrimaryPartCFrame(
                CFrame.new(ray.Position) * 
                CFrame.Angles(0, math.random(0, 628)/100, 0) *
                CFrame.new(0, prop.PrimaryPart.Size.Y * scale / 2, 0)
            )
            
            -- ปรับขนาด
            for _, part in ipairs(prop:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.Size = part.Size * scale
                end
            end
            
            prop.Parent = folder
            count = count + 1
        end
    end
    
    print("วาง " .. count .. " " .. propName .. " เสร็จแล้ว")
end

return PropPlacer
```

---

## 61.3 การออกแบบด่าน (Level Design)

### 61.3.1 หลักการ 3C (Challenge, Choice, Consequence)

```lua
-- ระบบ Level ที่มี 3C
-- ServerScriptService/LevelSystem

local LevelSystem = {}
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- โครงสร้างด่าน
local levelData = {
    {
        id = 1,
        name = "ป่าเริ่มต้น",
        difficulty = "Easy",
        
        -- Challenge: ความท้าทาย
        challenges = {
            {type = "enemy", count = 3, enemyType = "Slime"},
            {type = "puzzle", id = "ColorPuzzle_01"},
            {type = "platforming", difficulty = 1}
        },
        
        -- Choice: ทางเลือก
        paths = {
            {
                name = "ทางสั้น",
                description = "เร็วกว่าแต่มีศัตรูมากกว่า",
                enemyMultiplier = 2.0,
                timeBonus = 30
            },
            {
                name = "ทางยาว",
                description = "ปลอดภัยกว่าแต่ใช้เวลามากกว่า",
                enemyMultiplier = 0.5,
                timeBonus = 0
            }
        },
        
        -- Consequence: ผลลัพธ์
        rewards = {
            completion = {gold = 100, exp = 50},
            fastClear = {gold = 200, exp = 100, bonus = "Speed Badge"},
            secretPath = {gold = 150, exp = 75, item = "Hidden Gem"}
        }
    }
}

-- ฟังก์ชันเริ่มด่าน
function LevelSystem.startLevel(player, levelId)
    local level = levelData[levelId]
    if not level then
        warn("ไม่พบด่าน " .. levelId)
        return
    end
    
    -- บันทึกเวลาเริ่ม
    local startTime = tick()
    
    -- แจ้งผู้เล่น
    local remoteEvent = ReplicatedStorage:FindFirstChild("LevelStarted")
    if remoteEvent then
        remoteEvent:FireClient(player, {
            levelName = level.name,
            difficulty = level.difficulty,
            paths = level.paths
        })
    end
    
    print(player.Name .. " เริ่มด่าน " .. level.name)
    return startTime
end

-- ฟังก์ชันจบด่าน
function LevelSystem.completeLevel(player, levelId, startTime, pathChosen)
    local level = levelData[levelId]
    if not level then return end
    
    local completionTime = tick() - startTime
    local rewards = level.rewards.completion
    
    -- ตรวจสอบ Fast Clear
    if completionTime < 120 then  -- ภายใน 2 นาที
        rewards = level.rewards.fastClear
        print(player.Name .. " ล้างด่านเร็ว! เวลา: " .. math.floor(completionTime) .. " วินาที")
    end
    
    -- ให้รางวัล
    local playerData = player:FindFirstChild("PlayerData")
    if playerData then
        local goldValue = playerData:FindFirstChild("Gold")
        local expValue = playerData:FindFirstChild("EXP")
        
        if goldValue then goldValue.Value = goldValue.Value + rewards.gold end
        if expValue then expValue.Value = expValue.Value + rewards.exp end
    end
    
    print(player.Name .. " ผ่านด่าน " .. level.name .. "! ได้รับ " .. rewards.gold .. " Gold")
end

return LevelSystem
```

### 61.3.2 การออกแบบด้วย Modularity

```lua
-- ระบบ Room-based Level Generation
-- ServerScriptService/RoomGenerator

local RoomGenerator = {}

-- Template ห้องต่างๆ
local roomTemplates = {
    entrance = {
        size = Vector3.new(40, 20, 40),
        doors = {"north", "east", "south", "west"},
        spawnPoint = Vector3.new(0, 5, -15),
        type = "start"
    },
    corridor = {
        size = Vector3.new(20, 15, 60),
        doors = {"north", "south"},
        enemyCount = 2,
        type = "combat"
    },
    largeChamber = {
        size = Vector3.new(80, 30, 80),
        doors = {"north", "east", "south", "west"},
        enemyCount = 8,
        hasChest = true,
        type = "combat"
    },
    puzzleRoom = {
        size = Vector3.new(50, 20, 50),
        doors = {"north", "south"},
        puzzleType = "pressure_plate",
        type = "puzzle"
    },
    bossRoom = {
        size = Vector3.new(100, 40, 100),
        doors = {"south"},
        bossType = "Dragon",
        type = "boss"
    }
}

-- สร้างห้องจาก template
function RoomGenerator.createRoom(templateName, position)
    local template = roomTemplates[templateName]
    if not template then
        warn("ไม่พบ template: " .. templateName)
        return nil
    end
    
    local roomFolder = Instance.new("Folder")
    roomFolder.Name = "Room_" .. templateName .. "_" .. math.random(1000, 9999)
    roomFolder.Parent = workspace
    
    -- สร้างพื้น
    local floor = Instance.new("Part")
    floor.Name = "Floor"
    floor.Size = Vector3.new(template.size.X, 1, template.size.Z)
    floor.Position = position
    floor.Anchored = true
    floor.Material = Enum.Material.SmoothPlastic
    floor.BrickColor = BrickColor.new("Medium stone grey")
    floor.Parent = roomFolder
    
    -- สร้างเพดาน
    local ceiling = Instance.new("Part")
    ceiling.Name = "Ceiling"
    ceiling.Size = Vector3.new(template.size.X, 1, template.size.Z)
    ceiling.Position = position + Vector3.new(0, template.size.Y, 0)
    ceiling.Anchored = true
    ceiling.Material = Enum.Material.SmoothPlastic
    ceiling.BrickColor = BrickColor.new("Dark stone grey")
    ceiling.Parent = roomFolder
    
    -- สร้างผนัง
    local wallThickness = 2
    local walls = {
        {pos = Vector3.new(template.size.X/2, template.size.Y/2, 0), size = Vector3.new(wallThickness, template.size.Y, template.size.Z)},
        {pos = Vector3.new(-template.size.X/2, template.size.Y/2, 0), size = Vector3.new(wallThickness, template.size.Y, template.size.Z)},
        {pos = Vector3.new(0, template.size.Y/2, template.size.Z/2), size = Vector3.new(template.size.X, template.size.Y, wallThickness)},
        {pos = Vector3.new(0, template.size.Y/2, -template.size.Z/2), size = Vector3.new(template.size.X, template.size.Y, wallThickness)},
    }
    
    for i, wallData in ipairs(walls) do
        local wall = Instance.new("Part")
        wall.Name = "Wall_" .. i
        wall.Size = wallData.size
        wall.Position = position + wallData.pos
        wall.Anchored = true
        wall.Material = Enum.Material.SmoothPlastic
        wall.BrickColor = BrickColor.new("Medium stone grey")
        wall.Parent = roomFolder
    end
    
    -- สร้างประตู
    for _, doorSide in ipairs(template.doors) do
        local doorPos
        if doorSide == "north" then
            doorPos = position + Vector3.new(0, 5, -template.size.Z/2)
        elseif doorSide == "south" then
            doorPos = position + Vector3.new(0, 5, template.size.Z/2)
        elseif doorSide == "east" then
            doorPos = position + Vector3.new(template.size.X/2, 5, 0)
        elseif doorSide == "west" then
            doorPos = position + Vector3.new(-template.size.X/2, 5, 0)
        end
        
        if doorPos then
            local door = Instance.new("Part")
            door.Name = "Door_" .. doorSide
            door.Size = Vector3.new(8, 10, 2)
            door.Position = doorPos
            door.Anchored = true
            door.CanCollide = false
            door.Transparency = 0.7
            door.BrickColor = BrickColor.new("Bright yellow")
            door.Material = Enum.Material.Neon
            door.Parent = roomFolder
        end
    end
    
    print("สร้างห้อง " .. templateName .. " ที่ตำแหน่ง " .. tostring(position))
    return roomFolder
end

-- สร้างดันเจี้ยนอัตโนมัติ
function RoomGenerator.generateDungeon(seed)
    math.randomseed(seed or tick())
    
    local rooms = {}
    local currentPos = Vector3.new(0, 0, 0)
    
    -- สร้างห้องทางเข้า
    local entranceRoom = RoomGenerator.createRoom("entrance", currentPos)
    table.insert(rooms, {room = entranceRoom, pos = currentPos, template = "entrance"})
    
    -- สร้างห้องกลาง
    local roomSequence = {"corridor", "largeChamber", "corridor", "puzzleRoom", "corridor", "bossRoom"}
    
    for _, roomType in ipairs(roomSequence) do
        currentPos = currentPos + Vector3.new(0, 0, -roomTemplates[roomType].size.Z - 5)
        local newRoom = RoomGenerator.createRoom(roomType, currentPos)
        table.insert(rooms, {room = newRoom, pos = currentPos, template = roomType})
    end
    
    print("สร้างดันเจี้ยน " .. #rooms .. " ห้องเสร็จแล้ว!")
    return rooms
end

return RoomGenerator
```

---

## 61.4 การออกแบบแสงและบรรยากาศ

### 61.4.1 การใช้ Lighting อย่างมีประสิทธิภาพ

```lua
-- ระบบจัดการแสงสำหรับแต่ละบริเวณ
-- ServerScriptService/LightingManager

local LightingManager = {}
local Lighting = game:GetService("Lighting")
local TweenService = game:GetService("TweenService")

-- ชุดแสงสำหรับบรรยากาศต่างๆ
local lightingPresets = {
    day = {
        Brightness = 2,
        ClockTime = 14,
        Ambient = Color3.fromRGB(150, 150, 200),
        OutdoorAmbient = Color3.fromRGB(200, 200, 255),
        ShadowSoftness = 0.2,
        FogColor = Color3.fromRGB(200, 220, 255),
        FogEnd = 1000,
        FogStart = 500
    },
    night = {
        Brightness = 0.2,
        ClockTime = 23,
        Ambient = Color3.fromRGB(20, 20, 60),
        OutdoorAmbient = Color3.fromRGB(30, 30, 80),
        ShadowSoftness = 0.5,
        FogColor = Color3.fromRGB(10, 10, 30),
        FogEnd = 200,
        FogStart = 50
    },
    dungeon = {
        Brightness = 0.1,
        ClockTime = 23,
        Ambient = Color3.fromRGB(10, 5, 20),
        OutdoorAmbient = Color3.fromRGB(5, 5, 15),
        ShadowSoftness = 0.8,
        FogColor = Color3.fromRGB(5, 5, 15),
        FogEnd = 100,
        FogStart = 20
    },
    sunset = {
        Brightness = 1,
        ClockTime = 18,
        Ambient = Color3.fromRGB(200, 100, 50),
        OutdoorAmbient = Color3.fromRGB(255, 150, 80),
        ShadowSoftness = 0.3,
        FogColor = Color3.fromRGB(255, 180, 120),
        FogEnd = 800,
        FogStart = 200
    }
}

-- เปลี่ยนบรรยากาศแบบ Smooth
function LightingManager.transition(presetName, duration)
    local preset = lightingPresets[presetName]
    if not preset then
        warn("ไม่พบ preset: " .. presetName)
        return
    end
    
    duration = duration or 3
    
    local tweenInfo = TweenInfo.new(
        duration,
        Enum.EasingStyle.Quad,
        Enum.EasingDirection.InOut
    )
    
    local tween = TweenService:Create(Lighting, tweenInfo, {
        Brightness = preset.Brightness,
        Ambient = preset.Ambient,
        OutdoorAmbient = preset.OutdoorAmbient,
        FogColor = preset.FogColor,
        FogEnd = preset.FogEnd,
        FogStart = preset.FogStart
    })
    
    tween:Play()
    
    -- เปลี่ยน ClockTime แบบ smooth
    local targetTime = preset.ClockTime
    local startTime = Lighting.ClockTime
    local elapsed = 0
    
    local connection
    connection = game:GetService("RunService").Heartbeat:Connect(function(dt)
        elapsed = elapsed + dt
        local alpha = math.min(elapsed / duration, 1)
        Lighting.ClockTime = startTime + (targetTime - startTime) * alpha
        
        if alpha >= 1 then
            connection:Disconnect()
        end
    end)
    
    print("เปลี่ยนบรรยากาศเป็น " .. presetName)
end

-- เพิ่มแสงจุดในห้อง
function LightingManager.addRoomLight(position, color, brightness, range)
    local light = Instance.new("PointLight")
    light.Color = color or Color3.fromRGB(255, 200, 100)
    light.Brightness = brightness or 5
    light.Range = range or 20
    
    local part = Instance.new("Part")
    part.Name = "LightSource"
    part.Size = Vector3.new(0.5, 0.5, 0.5)
    part.Position = position
    part.Anchored = true
    part.CanCollide = false
    part.Transparency = 1
    part.Parent = workspace
    
    light.Parent = part
    
    -- เพิ่มไฟกระพริบ
    local RunService = game:GetService("RunService")
    local baseRange = light.Range
    
    RunService.Heartbeat:Connect(function()
        light.Range = baseRange + math.sin(tick() * 3) * 2
    end)
    
    return part
end

return LightingManager
```

---

## 61.5 การออกแบบเพื่อ Player Experience

### 61.5.1 Visual Cues และ Signposting

```lua
-- ระบบ Visual Cues สำหรับนำทางผู้เล่น
-- StarterPlayerScripts/NavigationUI

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

-- สร้าง Arrow ที่ชี้ไปยังจุดหมาย
local function createDirectionArrow()
    local arrow = Instance.new("ScreenGui")
    arrow.Name = "DirectionArrow"
    arrow.ResetOnSpawn = false
    arrow.Parent = player.PlayerGui
    
    local arrowImage = Instance.new("ImageLabel")
    arrowImage.Name = "Arrow"
    arrowImage.Size = UDim2.new(0, 50, 0, 50)
    arrowImage.Position = UDim2.new(0.5, -25, 0.5, -25)
    arrowImage.BackgroundTransparency = 1
    arrowImage.Image = "rbxassetid://3926305545"  -- Arrow icon
    arrowImage.ImageColor3 = Color3.fromRGB(255, 255, 0)
    arrowImage.Visible = false
    arrowImage.Parent = arrow
    
    return arrowImage
end

local directionArrow = createDirectionArrow()
local currentTarget = nil

-- อัพเดทลูกศรชี้ทาง
local function updateNavigationArrow()
    if not currentTarget then
        directionArrow.Visible = false
        return
    end
    
    local character = player.Character
    if not character or not character:FindFirstChild("HumanoidRootPart") then
        directionArrow.Visible = false
        return
    end
    
    local rootPos = character.HumanoidRootPart.Position
    local targetPos = currentTarget
    
    -- ตรวจสอบว่า target อยู่บนจอหรือไม่
    local _, onScreen = camera:WorldToScreenPoint(targetPos)
    
    if onScreen then
        directionArrow.Visible = false  -- ไม่แสดงถ้าเห็น target
    else
        directionArrow.Visible = true
        
        -- คำนวณทิศทาง
        local direction = (targetPos - rootPos).Unit
        local cameraLook = camera.CFrame.LookVector
        local cameraRight = camera.CFrame.RightVector
        
        local dotRight = direction:Dot(cameraRight)
        local dotForward = direction:Dot(cameraLook)
        
        -- คำนวณมุม
        local angle = math.atan2(dotRight, dotForward)
        
        -- หมุนลูกศร
        directionArrow.Rotation = math.deg(angle)
        
        -- วาง Arrow ที่ขอบจอ
        local radius = 150
        local screenX = 0.5 + math.sin(angle) * radius / camera.ViewportSize.X
        local screenY = 0.5 - math.cos(angle) * radius / camera.ViewportSize.Y
        
        directionArrow.Position = UDim2.new(
            math.clamp(screenX, 0.05, 0.95),
            -25,
            math.clamp(screenY, 0.05, 0.95),
            -25
        )
    end
end

-- ตั้ง target ที่ต้องการไป
local function setNavigationTarget(targetPosition)
    currentTarget = targetPosition
    if targetPosition then
        directionArrow.Visible = true
        print("ตั้งจุดหมายที่ " .. tostring(targetPosition))
    else
        directionArrow.Visible = false
    end
end

RunService.RenderStepped:Connect(updateNavigationArrow)

-- ทดสอบ: ชี้ไปยัง spawn point
setNavigationTarget(Vector3.new(100, 5, 100))
```

---

## 61.6 การทดสอบและปรับแต่งแผนที่

### 61.6.1 Playtesting Checklist

```lua
-- ระบบ Debug สำหรับทดสอบแผนที่
-- ServerScriptService/MapDebugger

local MapDebugger = {}

-- รายการตรวจสอบ
local checks = {
    {name = "Spawn Points", check = function()
        local spawns = workspace:FindFirstChild("SpawnLocations")
        if spawns and #spawns:GetChildren() > 0 then
            return true, #spawns:GetChildren() .. " spawn points พบ"
        end
        return false, "ไม่พบ Spawn Points!"
    end},
    
    {name = "Checkpoints", check = function()
        local checkpoints = workspace:FindFirstChild("Checkpoints")
        if checkpoints and #checkpoints:GetChildren() > 0 then
            return true, #checkpoints:GetChildren() .. " checkpoints พบ"
        end
        return false, "ไม่พบ Checkpoints - ผู้เล่นจะต้องเริ่มใหม่จากต้น!"
    end},
    
    {name = "Kill Brick", check = function()
        local killBrick = workspace:FindFirstChild("KillBrick")
        if killBrick then
            return true, "Kill Brick มีอยู่"
        end
        return false, "ไม่มี Kill Brick - ผู้เล่นอาจตกออกนอกแผนที่!"
    end},
    
    {name = "Boundaries", check = function()
        -- ตรวจสอบว่าแผนที่มีขอบเขต
        local walls = {
            workspace:FindFirstChild("Wall_North"),
            workspace:FindFirstChild("Wall_South"),
            workspace:FindFirstChild("Wall_East"),
            workspace:FindFirstChild("Wall_West")
        }
        
        local count = 0
        for _, wall in ipairs(walls) do
            if wall then count = count + 1 end
        end
        
        if count == 4 then
            return true, "มีขอบเขตครบ 4 ด้าน"
        end
        return false, "มีขอบเขตแค่ " .. count .. "/4 ด้าน"
    end},
    
    {name = "Performance", check = function()
        local partCount = 0
        for _, v in ipairs(workspace:GetDescendants()) do
            if v:IsA("BasePart") then
                partCount = partCount + 1
            end
        end
        
        if partCount < 5000 then
            return true, "Parts: " .. partCount .. " (ดี)"
        elseif partCount < 10000 then
            return false, "Parts: " .. partCount .. " (มากเกินไป - อาจกระทบ performance)"
        else
            return false, "Parts: " .. partCount .. " (มากเกินไปมาก! ต้องปรับปรุง)"
        end
    end}
}

-- รันการตรวจสอบทั้งหมด
function MapDebugger.runChecks()
    print("=== เริ่มตรวจสอบแผนที่ ===")
    local passed = 0
    local failed = 0
    
    for _, checkData in ipairs(checks) do
        local success, message = checkData.check()
        
        if success then
            print("✓ " .. checkData.name .. ": " .. message)
            passed = passed + 1
        else
            warn("✗ " .. checkData.name .. ": " .. message)
            failed = failed + 1
        end
    end
    
    print("=== ผลการตรวจสอบ: " .. passed .. " ผ่าน, " .. failed .. " ล้มเหลว ===")
end

return MapDebugger
```

---

## 61.7 ข้อผิดพลาดที่พบบ่อยในการออกแบบแผนที่

### ข้อผิดพลาด 1: การสร้างแผนที่ที่ใหญ่เกินไป

```lua
-- ❌ ผิด: แผนที่ใหญ่เกินไป ทำให้ผู้เล่นรู้สึกว่างเปล่า
local WRONG_MAP_SIZE = Vector3.new(10000, 500, 10000)

-- ✓ ถูก: แผนที่ขนาดพอเหมาะ สมดุลกับจำนวนผู้เล่น
local CORRECT_MAP_SIZE = Vector3.new(500, 100, 500)
-- สำหรับ 10-20 ผู้เล่น แผนที่ 500x500 studs เหมาะสม
```

### ข้อผิดพลาด 2: ไม่มี Checkpoint

```lua
-- ✓ ถูก: เพิ่ม Checkpoint ในจุดสำคัญ
local function setupCheckpoints()
    local checkpoints = {
        {position = Vector3.new(0, 5, 0), id = 1},       -- จุดเริ่มต้น
        {position = Vector3.new(100, 5, 0), id = 2},     -- กลางทาง
        {position = Vector3.new(200, 5, 100), id = 3},   -- ก่อนบอส
    }
    
    for _, cp in ipairs(checkpoints) do
        local checkpoint = Instance.new("Part")
        checkpoint.Name = "Checkpoint_" .. cp.id
        checkpoint.Position = cp.position
        checkpoint.Size = Vector3.new(10, 1, 10)
        checkpoint.Anchored = true
        checkpoint.CanCollide = false
        checkpoint.Transparency = 0.5
        checkpoint.BrickColor = BrickColor.new("Bright green")
        
        -- เพิ่ม attribute สำหรับ ID
        checkpoint:SetAttribute("CheckpointID", cp.id)
        
        checkpoint.Parent = workspace:FindFirstChild("Checkpoints") or workspace
    end
end
```

### ข้อผิดพลาด 3: การใช้แสงไม่เหมาะสม

```lua
-- ❌ ผิด: ใช้แสงสว่างเกินไป ทำลายบรรยากาศ
local wrongLighting = {
    Brightness = 10,  -- สว่างเกินไป
    Ambient = Color3.fromRGB(255, 255, 255)  -- ขาวทั้งหมด
}

-- ✓ ถูก: ปรับแสงให้เหมาะกับบรรยากาศ
local correctLighting = {
    Brightness = 1.5,
    Ambient = Color3.fromRGB(130, 140, 170),
    OutdoorAmbient = Color3.fromRGB(180, 190, 220)
}
```

---

## 61.8 แบบฝึกหัด

### แบบฝึกหัดที่ 1: สร้างแผนที่ 3 โซน
สร้างแผนที่ที่มี 3 โซนที่แตกต่างกัน:
1. โซนเริ่มต้น (Safe Zone) - สีเขียว, สงบ
2. โซนกลาง (Challenge Zone) - สีส้ม, มีศัตรู
3. โซนสุดท้าย (Boss Zone) - สีแดง, มีบอส

### แบบฝึกหัดที่ 2: สร้าง Procedural Dungeon
ใช้ RoomGenerator ที่เรียนมาสร้างดันเจี้ยนที่มี:
- ห้องทางเข้า 1 ห้อง
- ห้องต่อสู้ 3-5 ห้อง
- ห้องปริศนา 1-2 ห้อง
- ห้องบอส 1 ห้อง

### แบบฝึกหัดที่ 3: ปรับแต่งระบบแสง
สร้างระบบที่เปลี่ยนแสงตามเวลาในเกม:
- เช้า: แสงสีส้มอบอุ่น
- กลางวัน: แสงสว่างจ้า
- เย็น: แสงสีทอง
- กลางคืน: แสงมืด มีแสงจันทร์สีน้ำเงิน

---

## 61.9 เคล็ดลับจากมืออาชีพ

1. **Rule of Three**: จัดวางสิ่งของเป็นชุดๆ ละ 3 ชิ้น จะดูสวยงามกว่า
2. **Silhouette Test**: แผนที่ที่ดีควรจำได้ง่ายแม้เห็นแค่ shadow
3. **Color Coding**: ใช้สีบอกประเภทของพื้นที่ (เช่น แดง=อันตราย, เขียว=ปลอดภัย)
4. **Sound Design**: เสียงสำคัญพอๆ กับภาพในการสร้างบรรยากาศ
5. **Player Testing**: ให้คนอื่นเล่นและดูว่าเขาไปทางไหนก่อน

---

## สรุป

ในบทนี้เราได้เรียนรู้:
- หลักการ Flow, Pacing, และ 3C
- การสร้างภูมิประเทศด้วย Terrain API
- การออกแบบด่านแบบ Modular
- การใช้แสงและบรรยากาศ
- Visual Cues สำหรับนำทางผู้เล่น
- การทดสอบและ Debug แผนที่

ในบทถัดไปเราจะนำความรู้เหล่านี้ไปสร้าง Obstacle Course (Obby) ที่สนุกและท้าทาย!
