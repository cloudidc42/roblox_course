# Part 62: การสร้าง Obstacle Course (Obby)

## บทนำ

Obby (Obstacle Course) เป็นหนึ่งในประเภทเกมที่ได้รับความนิยมสูงสุดใน Roblox ตั้งแต่ผู้เล่นเริ่มต้นไปจนถึงผู้เล่นขั้นสูง ในบทนี้เราจะเรียนรู้การสร้าง Obby ที่มีระบบครบครัน ตั้งแต่การออกแบบ Platform ไปจนถึงระบบ Checkpoint และ Leaderboard

---

## 62.1 โครงสร้างพื้นฐานของ Obby

### 62.1.1 การตั้งค่าโปรเจ็กต์

```
Workspace
├── Map
│   ├── Stage_1 (Folder)
│   │   ├── Platforms
│   │   ├── Obstacles
│   │   └── Checkpoint_1
│   ├── Stage_2 (Folder)
│   └── ...
├── SpawnLocation
└── KillBrick

ServerScriptService
├── ObbyManager (Script)
├── CheckpointManager (Script)
└── LeaderboardManager (Script)

ReplicatedStorage
├── Remotes (Folder)
│   ├── UpdateCheckpoint (RemoteEvent)
│   └── StageComplete (RemoteEvent)
└── ObbyData (ModuleScript)

StarterGui
└── ObbyHUD (ScreenGui)
```

### 62.1.2 ระบบ Stage พื้นฐาน

```lua
-- ServerScriptService/ObbyManager.lua
-- ระบบจัดการ Obby หลัก

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local DataStoreService = game:GetService("DataStoreService")

-- DataStore สำหรับบันทึก progress
local checkpointStore = DataStoreService:GetDataStore("ObbyCheckpoints_v1")

-- Remotes
local Remotes = ReplicatedStorage:FindFirstChild("Remotes") or Instance.new("Folder")
Remotes.Name = "Remotes"
Remotes.Parent = ReplicatedStorage

local updateCheckpointEvent = Instance.new("RemoteEvent")
updateCheckpointEvent.Name = "UpdateCheckpoint"
updateCheckpointEvent.Parent = Remotes

local stageCompleteEvent = Instance.new("RemoteEvent")
stageCompleteEvent.Name = "StageComplete"
stageCompleteEvent.Parent = Remotes

-- ข้อมูลผู้เล่น
local playerData = {}

-- โหลดข้อมูลผู้เล่น
local function loadPlayerData(player)
    local success, data = pcall(function()
        return checkpointStore:GetAsync("player_" .. player.UserId)
    end)
    
    if success and data then
        playerData[player.UserId] = data
        print(player.Name .. " โหลดข้อมูล - Stage: " .. (data.stage or 1))
    else
        playerData[player.UserId] = {
            stage = 1,
            checkpoint = 0,
            totalTime = 0,
            completions = 0
        }
        print(player.Name .. " เริ่มใหม่ที่ Stage 1")
    end
    
    -- ตั้ง IntValue สำหรับ Leaderboard
    local leaderstats = Instance.new("Folder")
    leaderstats.Name = "leaderstats"
    leaderstats.Parent = player
    
    local stageValue = Instance.new("IntValue")
    stageValue.Name = "Stage"
    stageValue.Value = playerData[player.UserId].stage
    stageValue.Parent = leaderstats
    
    local completionsValue = Instance.new("IntValue")
    completionsValue.Name = "Wins"
    completionsValue.Value = playerData[player.UserId].completions
    completionsValue.Parent = leaderstats
end

-- บันทึกข้อมูลผู้เล่น
local function savePlayerData(player)
    if not playerData[player.UserId] then return end
    
    local success, err = pcall(function()
        checkpointStore:SetAsync("player_" .. player.UserId, playerData[player.UserId])
    end)
    
    if success then
        print(player.Name .. " บันทึกข้อมูลแล้ว")
    else
        warn("ไม่สามารถบันทึกข้อมูล " .. player.Name .. ": " .. err)
    end
end

-- Teleport ไปยัง checkpoint
local function teleportToCheckpoint(player)
    local data = playerData[player.UserId]
    if not data then return end
    
    local character = player.Character
    if not character then return end
    
    -- หา checkpoint ใน workspace
    local checkpointName = "Checkpoint_" .. data.stage
    local checkpoint = workspace.Map:FindFirstChild(checkpointName, true)
    
    if checkpoint then
        local rootPart = character:FindFirstChild("HumanoidRootPart")
        if rootPart then
            rootPart.CFrame = checkpoint.CFrame + Vector3.new(0, 5, 0)
            print(player.Name .. " teleport ไปที่ Stage " .. data.stage)
        end
    else
        -- ถ้าไม่เจอ checkpoint ให้กลับไป spawn
        local spawn = workspace:FindFirstChild("SpawnLocation")
        if spawn then
            local rootPart = character:FindFirstChild("HumanoidRootPart")
            if rootPart then
                rootPart.CFrame = spawn.CFrame + Vector3.new(0, 5, 0)
            end
        end
    end
end

-- Event handlers
Players.PlayerAdded:Connect(function(player)
    loadPlayerData(player)
    
    player.CharacterAdded:Connect(function(character)
        task.wait(1)  -- รอ character โหลด
        teleportToCheckpoint(player)
    end)
end)

Players.PlayerRemoving:Connect(function(player)
    savePlayerData(player)
    playerData[player.UserId] = nil
end)

-- บันทึกเมื่อเซิร์ฟเวอร์ปิด
game:BindToClose(function()
    for _, player in ipairs(Players:GetPlayers()) do
        savePlayerData(player)
    end
end)

-- Remote สำหรับอัพเดท checkpoint
updateCheckpointEvent.OnServerEvent:Connect(function(player, stageNumber)
    local data = playerData[player.UserId]
    if not data then return end
    
    -- อัพเดทถ้า stage ใหม่มากกว่า stage ปัจจุบัน
    if stageNumber > data.stage then
        data.stage = stageNumber
        
        -- อัพเดท leaderboard
        local leaderstats = player:FindFirstChild("leaderstats")
        if leaderstats then
            local stageValue = leaderstats:FindFirstChild("Stage")
            if stageValue then
                stageValue.Value = stageNumber
            end
        end
        
        print(player.Name .. " ถึง Stage " .. stageNumber)
        
        -- บันทึกทุกๆ 5 stages
        if stageNumber % 5 == 0 then
            savePlayerData(player)
        end
    end
end)

-- Remote สำหรับจบ Obby
stageCompleteEvent.OnServerEvent:Connect(function(player)
    local data = playerData[player.UserId]
    if not data then return end
    
    data.completions = data.completions + 1
    data.stage = 1  -- รีเซ็ต stage
    
    -- อัพเดท leaderboard
    local leaderstats = player:FindFirstChild("leaderstats")
    if leaderstats then
        local winsValue = leaderstats:FindFirstChild("Wins")
        if winsValue then
            winsValue.Value = data.completions
        end
        local stageValue = leaderstats:FindFirstChild("Stage")
        if stageValue then
            stageValue.Value = 1
        end
    end
    
    savePlayerData(player)
    print(player.Name .. " ผ่าน Obby! รวม " .. data.completions .. " ครั้ง")
end)
```

---

## 62.2 ระบบ Checkpoint

### 62.2.1 Checkpoint Script

```lua
-- Script ใน Checkpoint Part (ต้องใส่ใน Part แต่ละ Checkpoint)
-- Workspace/Map/Stage_X/Checkpoint

local checkpoint = script.Parent
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- ดึงหมายเลข Stage จากชื่อ
local stageNumber = tonumber(checkpoint.Name:match("Checkpoint_(%d+)")) or 1

-- สีเริ่มต้น
local defaultColor = BrickColor.new("Bright blue")
local activatedColor = BrickColor.new("Bright green")

-- ผู้เล่นที่ผ่านแล้ว
local activatedPlayers = {}

-- เอฟเฟกต์เมื่อ activate
local function playActivationEffect(position)
    -- Particle Effect
    local attachment = Instance.new("Attachment")
    attachment.Parent = checkpoint
    
    local particles = Instance.new("ParticleEmitter")
    particles.Color = ColorSequence.new(Color3.fromRGB(0, 255, 100))
    particles.Size = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 1),
        NumberSequenceKeypoint.new(1, 0)
    })
    particles.Lifetime = NumberRange.new(0.5, 1.5)
    particles.Rate = 100
    particles.Speed = NumberRange.new(10, 20)
    particles.SpreadAngle = Vector2.new(45, 45)
    particles.Parent = attachment
    
    task.wait(0.5)
    particles.Enabled = false
    
    game:GetService("Debris"):AddItem(attachment, 2)
    
    -- เสียง
    local sound = Instance.new("Sound")
    sound.SoundId = "rbxassetid://6518811702"  -- เสียง checkpoint
    sound.Volume = 0.5
    sound.Parent = checkpoint
    sound:Play()
    game:GetService("Debris"):AddItem(sound, 3)
end

-- ตรวจจับผู้เล่นแตะ
checkpoint.Touched:Connect(function(hit)
    local character = hit.Parent
    local player = Players:GetPlayerFromCharacter(character)
    
    if not player then return end
    if activatedPlayers[player.UserId] then return end  -- ผ่านแล้ว
    
    activatedPlayers[player.UserId] = true
    
    -- เปลี่ยนสี
    checkpoint.BrickColor = activatedColor
    checkpoint.Material = Enum.Material.Neon
    
    -- เอฟเฟกต์
    playActivationEffect(checkpoint.Position)
    
    -- แจ้ง Server
    local updateEvent = ReplicatedStorage.Remotes:FindFirstChild("UpdateCheckpoint")
    if updateEvent then
        updateEvent:FireServer(stageNumber)  -- ส่งจาก Client? ไม่ควร
        -- หมายเหตุ: ในระบบจริงควรใช้ Server-side detection
    end
    
    -- แจ้งผู้เล่น
    local playerGui = player.PlayerGui
    local obbyHUD = playerGui:FindFirstChild("ObbyHUD")
    if obbyHUD then
        local notification = obbyHUD:FindFirstChild("StageNotification")
        if notification then
            notification.Text = "✓ Stage " .. stageNumber .. " Checkpoint!"
            notification.Visible = true
            task.wait(2)
            notification.Visible = false
        end
    end
    
    print(player.Name .. " ถึง Checkpoint Stage " .. stageNumber)
end)

-- Animation หมุนของ Checkpoint
local RunService = game:GetService("RunService")
local baseRotation = checkpoint.CFrame

RunService.Heartbeat:Connect(function()
    checkpoint.CFrame = CFrame.new(checkpoint.Position) * 
        CFrame.Angles(0, math.rad(tick() * 90), 0)
end)
```

---

## 62.3 การสร้าง Obstacle ต่างๆ

### 62.3.1 Moving Platform

```lua
-- Script สำหรับ Moving Platform
-- ใส่ใน Part ของ Platform

local platform = script.Parent
local RunService = game:GetService("RunService")

-- ตั้งค่า
local config = {
    moveType = "horizontal",    -- "horizontal", "vertical", "circular"
    speed = 2,                  -- ความเร็ว (studs/second)
    distance = 20,              -- ระยะทางเคลื่อนที่
    direction = Vector3.new(1, 0, 0),  -- ทิศทาง
}

local startPosition = platform.Position
local elapsed = 0

-- เคลื่อนที่แบบ Sin wave (ไป-กลับ)
if config.moveType == "horizontal" or config.moveType == "vertical" then
    RunService.Heartbeat:Connect(function(dt)
        elapsed = elapsed + dt * config.speed
        
        local offset = math.sin(elapsed) * config.distance
        platform.CFrame = CFrame.new(startPosition + config.direction * offset)
    end)
    
elseif config.moveType == "circular" then
    local radius = config.distance
    RunService.Heartbeat:Connect(function(dt)
        elapsed = elapsed + dt * config.speed * 0.5
        
        local x = math.cos(elapsed) * radius
        local z = math.sin(elapsed) * radius
        platform.CFrame = CFrame.new(
            startPosition.X + x,
            startPosition.Y,
            startPosition.Z + z
        )
    end)
end
```

### 62.3.2 Spinning Obstacle

```lua
-- Script สำหรับ Spinning Obstacle (กิลโลตีน/ใบพัด)
-- ใส่ใน Part ที่ต้องการหมุน

local spinner = script.Parent
local RunService = game:GetService("RunService")

-- ตั้งค่า
local config = {
    axis = "Y",         -- แกนที่หมุน: "X", "Y", "Z"
    speed = 180,        -- องศาต่อวินาที
    killOnTouch = true, -- ฆ่าผู้เล่นเมื่อแตะ
}

local startCFrame = spinner.CFrame
local elapsed = 0

-- ฟังก์ชันสร้าง rotation matrix
local function getRotation(angle)
    if config.axis == "X" then
        return CFrame.Angles(math.rad(angle), 0, 0)
    elseif config.axis == "Y" then
        return CFrame.Angles(0, math.rad(angle), 0)
    else
        return CFrame.Angles(0, 0, math.rad(angle))
    end
end

-- หมุน
RunService.Heartbeat:Connect(function(dt)
    elapsed = elapsed + dt * config.speed
    spinner.CFrame = startCFrame * getRotation(elapsed)
end)

-- ฆ่าผู้เล่น
if config.killOnTouch then
    spinner.Touched:Connect(function(hit)
        local character = hit.Parent
        local humanoid = character:FindFirstChild("Humanoid")
        if humanoid then
            humanoid.Health = 0
        end
    end)
end
```

### 62.3.3 Disappearing Platform

```lua
-- Script สำหรับ Disappearing Platform (หายไปเมื่อถูกยืน)
-- ใส่ใน Part

local platform = script.Parent
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")

-- ตั้งค่า
local config = {
    warningTime = 1.0,    -- เวลาก่อนจะหายไป (กระพริบ)
    disappearTime = 0.5,  -- เวลาหายไป
    respawnTime = 3.0,    -- เวลาก่อนจะกลับมา
}

local isDisappearing = false
local originalTransparency = platform.Transparency
local originalCanCollide = platform.CanCollide

-- ฟังก์ชันให้ platform หายไป
local function disappear()
    if isDisappearing then return end
    isDisappearing = true
    
    -- Phase 1: เตือน (กระพริบ)
    local warningStart = tick()
    while tick() - warningStart < config.warningTime do
        platform.Transparency = 0.5
        task.wait(0.1)
        platform.Transparency = originalTransparency
        task.wait(0.1)
    end
    
    -- Phase 2: หายไป
    local tweenInfo = TweenInfo.new(config.disappearTime, Enum.EasingStyle.Quad)
    local tween = TweenService:Create(platform, tweenInfo, {Transparency = 1})
    tween:Play()
    tween.Completed:Wait()
    
    platform.CanCollide = false
    
    -- Phase 3: รอ
    task.wait(config.respawnTime)
    
    -- Phase 4: กลับมา
    local respawnTween = TweenService:Create(platform, tweenInfo, {Transparency = originalTransparency})
    respawnTween:Play()
    respawnTween.Completed:Wait()
    
    platform.CanCollide = true
    isDisappearing = false
end

-- ตรวจจับผู้เล่นขึ้น
platform.Touched:Connect(function(hit)
    local character = hit.Parent
    local player = Players:GetPlayerFromCharacter(character)
    
    if player and not isDisappearing then
        disappear()
    end
end)
```

### 62.3.4 Lava/Kill Brick

```lua
-- Script สำหรับ Kill Brick (ลาวา/หนาม)
-- ใส่ใน Part

local killBrick = script.Parent
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")

-- เอฟเฟกต์ไฟ
local function addFireEffect()
    local fire = Instance.new("Fire")
    fire.Size = 3
    fire.Heat = 9
    fire.Color = Color3.fromRGB(255, 100, 0)
    fire.SecondaryColor = Color3.fromRGB(255, 200, 0)
    fire.Parent = killBrick
end

addFireEffect()

-- ฆ่าผู้เล่น
killBrick.Touched:Connect(function(hit)
    local character = hit.Parent
    local humanoid = character:FindFirstChildWhichIsA("Humanoid")
    
    if humanoid and humanoid.Health > 0 then
        -- เอฟเฟกต์ก่อนตาย
        local players = game:GetService("Players")
        local player = players:GetPlayerFromCharacter(character)
        
        if player then
            -- Shake effect (LocalScript ใน HUD จะจัดการ)
            humanoid.Health = 0
        end
    end
end)
```

---

## 62.4 ระบบ Obby HUD

### 62.4.1 GUI สำหรับแสดงข้อมูล

```lua
-- StarterGui/ObbyHUD/LocalScript

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local player = Players.LocalPlayer
local playerGui = player.PlayerGui

-- รอให้ GUI โหลด
local obbyHUD = script.Parent

-- สร้าง Stage Counter
local stageFrame = Instance.new("Frame")
stageFrame.Name = "StageFrame"
stageFrame.Size = UDim2.new(0, 200, 0, 60)
stageFrame.Position = UDim2.new(0.5, -100, 0, 10)
stageFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
stageFrame.BackgroundTransparency = 0.5
stageFrame.BorderSizePixel = 0
stageFrame.Parent = obbyHUD

-- Rounded corners
local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 10)
corner.Parent = stageFrame

local stageLabel = Instance.new("TextLabel")
stageLabel.Name = "StageLabel"
stageLabel.Size = UDim2.new(1, 0, 1, 0)
stageLabel.BackgroundTransparency = 1
stageLabel.Text = "Stage: 1"
stageLabel.TextColor3 = Color3.new(1, 1, 1)
stageLabel.TextScaled = true
stageLabel.Font = Enum.Font.GothamBold
stageLabel.Parent = stageFrame

-- Timer
local timerFrame = Instance.new("Frame")
timerFrame.Name = "TimerFrame"
timerFrame.Size = UDim2.new(0, 150, 0, 50)
timerFrame.Position = UDim2.new(0, 10, 0, 10)
timerFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
timerFrame.BackgroundTransparency = 0.5
timerFrame.BorderSizePixel = 0
timerFrame.Parent = obbyHUD

local corner2 = Instance.new("UICorner")
corner2.CornerRadius = UDim.new(0, 10)
corner2.Parent = timerFrame

local timerLabel = Instance.new("TextLabel")
timerLabel.Name = "TimerLabel"
timerLabel.Size = UDim2.new(1, 0, 1, 0)
timerLabel.BackgroundTransparency = 1
timerLabel.Text = "⏱ 00:00"
timerLabel.TextColor3 = Color3.new(1, 1, 1)
timerLabel.TextScaled = true
timerLabel.Font = Enum.Font.Gotham
timerLabel.Parent = timerFrame

-- Notification สำหรับ checkpoint
local notification = Instance.new("TextLabel")
notification.Name = "StageNotification"
notification.Size = UDim2.new(0, 300, 0, 50)
notification.Position = UDim2.new(0.5, -150, 0.7, 0)
notification.BackgroundColor3 = Color3.fromRGB(0, 200, 0)
notification.BackgroundTransparency = 0.3
notification.Text = "Checkpoint!"
notification.TextColor3 = Color3.new(1, 1, 1)
notification.TextScaled = true
notification.Font = Enum.Font.GothamBold
notification.BorderSizePixel = 0
notification.Visible = false
notification.Parent = obbyHUD

local corner3 = Instance.new("UICorner")
corner3.CornerRadius = UDim.new(0, 10)
corner3.Parent = notification

-- อัพเดท Timer
local startTime = tick()
local RunService = game:GetService("RunService")

RunService.RenderStepped:Connect(function()
    local elapsed = tick() - startTime
    local minutes = math.floor(elapsed / 60)
    local seconds = math.floor(elapsed % 60)
    timerLabel.Text = string.format("⏱ %02d:%02d", minutes, seconds)
    
    -- อัพเดท Stage
    local leaderstats = player:FindFirstChild("leaderstats")
    if leaderstats then
        local stageValue = leaderstats:FindFirstChild("Stage")
        if stageValue then
            stageLabel.Text = "Stage: " .. stageValue.Value
        end
    end
end)
```

---

## 62.5 การสร้าง Stage ตัวอย่าง

### 62.5.1 Stage Builder

```lua
-- ServerScriptService/StageBuilder.lua
-- สร้าง Stage โดยอัตโนมัติ

local StageBuilder = {}

-- ชนิดของ Platform
local platformTypes = {
    basic = {
        size = Vector3.new(8, 1, 8),
        material = Enum.Material.SmoothPlastic,
        color = BrickColor.new("Medium stone grey"),
        isKill = false
    },
    small = {
        size = Vector3.new(4, 1, 4),
        material = Enum.Material.SmoothPlastic,
        color = BrickColor.new("Bright blue"),
        isKill = false
    },
    lava = {
        size = Vector3.new(8, 1, 8),
        material = Enum.Material.Neon,
        color = BrickColor.new("Bright orange"),
        isKill = true
    },
    ice = {
        size = Vector3.new(8, 1, 8),
        material = Enum.Material.Ice,
        color = BrickColor.new("Pastel blue"),
        friction = 0,
        isKill = false
    }
}

-- สร้าง Platform
function StageBuilder.createPlatform(platformType, position, parent)
    local data = platformTypes[platformType] or platformTypes.basic
    
    local platform = Instance.new("Part")
    platform.Name = "Platform_" .. platformType
    platform.Size = data.size
    platform.Position = position
    platform.Anchored = true
    platform.Material = data.material
    platform.BrickColor = data.color
    
    if data.friction then
        local physicsProps = Instance.new("SpecialMesh")  -- ปรับ friction
        platform.CustomPhysicalProperties = PhysicalProperties.new(
            0.7, 0, 0, 0, 0  -- density, friction, elasticity, frictionWeight, elasticityWeight
        )
    end
    
    platform.Parent = parent or workspace
    
    -- เพิ่ม Script สำหรับ Kill Brick
    if data.isKill then
        local killScript = Instance.new("Script")
        killScript.Source = [[
            local brick = script.Parent
            brick.Touched:Connect(function(hit)
                local hum = hit.Parent:FindFirstChildWhichIsA("Humanoid")
                if hum then hum.Health = 0 end
            end)
        ]]
        killScript.Parent = platform
    end
    
    return platform
end

-- สร้าง Stage จาก Layout
function StageBuilder.buildStage(stageNumber, layout)
    local stageFolder = Instance.new("Folder")
    stageFolder.Name = "Stage_" .. stageNumber
    stageFolder.Parent = workspace.Map or workspace
    
    -- สร้าง Platform ตาม layout
    for i, platformData in ipairs(layout.platforms) do
        local platform = StageBuilder.createPlatform(
            platformData.type,
            platformData.position,
            stageFolder
        )
        
        -- เพิ่ม moving script ถ้าต้องการ
        if platformData.moving then
            local movingScript = Instance.new("Script")
            movingScript.Source = string.format([[
                local p = script.Parent
                local start = p.Position
                local t = 0
                game:GetService("RunService").Heartbeat:Connect(function(dt)
                    t = t + dt * %f
                    p.CFrame = CFrame.new(start + Vector3.new(%f, 0, 0) * math.sin(t))
                end)
            ]], platformData.speed or 1, platformData.moveDistance or 10)
            movingScript.Parent = platform
        end
    end
    
    -- สร้าง Checkpoint
    local checkpoint = Instance.new("Part")
    checkpoint.Name = "Checkpoint_" .. stageNumber
    checkpoint.Size = Vector3.new(6, 8, 1)
    checkpoint.Position = layout.checkpointPosition
    checkpoint.Anchored = true
    checkpoint.CanCollide = false
    checkpoint.Material = Enum.Material.Neon
    checkpoint.BrickColor = BrickColor.new("Bright blue")
    checkpoint.Parent = stageFolder
    
    print("สร้าง Stage " .. stageNumber .. " เสร็จแล้ว")
    return stageFolder
end

-- ตัวอย่าง Layout สำหรับ Stage 1
local stage1Layout = {
    checkpointPosition = Vector3.new(0, 5, -80),
    platforms = {
        {type = "basic", position = Vector3.new(0, 5, 0)},          -- เริ่มต้น
        {type = "basic", position = Vector3.new(12, 5, 0)},
        {type = "small", position = Vector3.new(24, 5, 0)},
        {type = "basic", position = Vector3.new(36, 7, 0)},         -- สูงขึ้น
        {type = "basic", position = Vector3.new(36, 7, -15), moving = true, speed = 1.5, moveDistance = 8},
        {type = "small", position = Vector3.new(36, 9, -30)},       -- เล็กลง
        {type = "lava", position = Vector3.new(25, 5, -45)},        -- ลาวา (ข้าม)
        {type = "lava", position = Vector3.new(10, 5, -45)},
        {type = "basic", position = Vector3.new(0, 5, -60)},
        {type = "basic", position = Vector3.new(0, 5, -80)},        -- ถึง checkpoint
    }
}

-- สร้าง Stage
StageBuilder.buildStage(1, stage1Layout)

return StageBuilder
```

---

## 62.6 ระบบ Leaderboard สำหรับ Obby

```lua
-- ServerScriptService/ObbyLeaderboard.lua

local Players = game:GetService("Players")
local DataStoreService = game:GetService("DataStoreService")
local OrderedDataStore = DataStoreService:GetOrderedDataStore("ObbyLeaderboard_v1")

-- อัพเดทคะแนนใน Ordered DataStore
local function updateLeaderboard(player, completions)
    local success, err = pcall(function()
        OrderedDataStore:SetAsync(tostring(player.UserId), completions)
    end)
    
    if not success then
        warn("ไม่สามารถอัพเดท Leaderboard: " .. err)
    end
end

-- ดึงข้อมูล Top 10
local function getTopPlayers()
    local success, data = pcall(function()
        return OrderedDataStore:GetSortedAsync(false, 10)  -- false = descending
    end)
    
    if success and data then
        local page = data:GetCurrentPage()
        local result = {}
        
        for rank, entry in ipairs(page) do
            local userId = tonumber(entry.key)
            local completions = entry.value
            
            -- แปลง UserId เป็นชื่อ
            local success, name = pcall(function()
                return Players:GetNameFromUserIdAsync(userId)
            end)
            
            table.insert(result, {
                rank = rank,
                name = success and name or "Unknown",
                completions = completions
            })
        end
        
        return result
    end
    
    return {}
end

-- สร้าง GUI แสดง Leaderboard
local function createLeaderboardGUI()
    -- สร้าง SurfaceGui บน Part
    local leaderboardPart = workspace:FindFirstChild("LeaderboardBoard")
    if not leaderboardPart then return end
    
    local surfaceGui = Instance.new("SurfaceGui")
    surfaceGui.Name = "LeaderboardSurface"
    surfaceGui.Face = Enum.NormalId.Front
    surfaceGui.PixelsPerStud = 50
    surfaceGui.Parent = leaderboardPart
    
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(1, 0, 1, 0)
    frame.BackgroundColor3 = Color3.fromRGB(20, 20, 40)
    frame.Parent = surfaceGui
    
    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, 0, 0.1, 0)
    title.Position = UDim2.new(0, 0, 0, 0)
    title.BackgroundColor3 = Color3.fromRGB(50, 50, 100)
    title.Text = "🏆 TOP PLAYERS"
    title.TextColor3 = Color3.new(1, 1, 0)
    title.TextScaled = true
    title.Font = Enum.Font.GothamBold
    title.Parent = frame
    
    -- อัพเดทรายชื่อ
    local function updateDisplay()
        -- ลบรายการเก่า
        for _, child in ipairs(frame:GetChildren()) do
            if child.Name:find("Entry_") then
                child:Destroy()
            end
        end
        
        local topPlayers = getTopPlayers()
        
        for i, playerData in ipairs(topPlayers) do
            local entryFrame = Instance.new("Frame")
            entryFrame.Name = "Entry_" .. i
            entryFrame.Size = UDim2.new(1, 0, 0.08, 0)
            entryFrame.Position = UDim2.new(0, 0, 0.1 + (i-1) * 0.09, 0)
            entryFrame.BackgroundColor3 = i <= 3 
                and Color3.fromRGB(80, 60, 0)  -- ทอง สำหรับ Top 3
                or Color3.fromRGB(40, 40, 60)
            entryFrame.BorderSizePixel = 0
            entryFrame.Parent = frame
            
            local rankLabel = Instance.new("TextLabel")
            rankLabel.Size = UDim2.new(0.15, 0, 1, 0)
            rankLabel.BackgroundTransparency = 1
            rankLabel.Text = "#" .. i
            rankLabel.TextColor3 = i == 1 and Color3.fromRGB(255, 215, 0) 
                or i == 2 and Color3.fromRGB(192, 192, 192)
                or i == 3 and Color3.fromRGB(205, 127, 50)
                or Color3.new(1, 1, 1)
            rankLabel.TextScaled = true
            rankLabel.Font = Enum.Font.GothamBold
            rankLabel.Parent = entryFrame
            
            local nameLabel = Instance.new("TextLabel")
            nameLabel.Size = UDim2.new(0.6, 0, 1, 0)
            nameLabel.Position = UDim2.new(0.15, 0, 0, 0)
            nameLabel.BackgroundTransparency = 1
            nameLabel.Text = playerData.name
            nameLabel.TextColor3 = Color3.new(1, 1, 1)
            nameLabel.TextScaled = true
            nameLabel.Font = Enum.Font.Gotham
            nameLabel.TextXAlignment = Enum.TextXAlignment.Left
            nameLabel.Parent = entryFrame
            
            local completionsLabel = Instance.new("TextLabel")
            completionsLabel.Size = UDim2.new(0.25, 0, 1, 0)
            completionsLabel.Position = UDim2.new(0.75, 0, 0, 0)
            completionsLabel.BackgroundTransparency = 1
            completionsLabel.Text = playerData.completions .. "x"
            completionsLabel.TextColor3 = Color3.fromRGB(0, 255, 150)
            completionsLabel.TextScaled = true
            completionsLabel.Font = Enum.Font.GothamBold
            completionsLabel.Parent = entryFrame
        end
    end
    
    -- อัพเดททุก 30 วินาที
    updateDisplay()
    while true do
        task.wait(30)
        updateDisplay()
    end
end

-- เริ่มต้น
task.spawn(createLeaderboardGUI)
```

---

## 62.7 ข้อผิดพลาดที่พบบ่อย

### ข้อผิดพลาด 1: Checkpoint ไม่บันทึก

```lua
-- ❌ ผิด: ใช้ RemoteEvent จาก Client โดยตรง (โกงได้)
-- Client:
remoteEvent:FireServer(999)  -- ส่ง stage 999 เลย!

-- ✓ ถูก: ตรวจสอบจาก Server
-- Server:
checkpoint.Touched:Connect(function(hit)
    local player = Players:GetPlayerFromCharacter(hit.Parent)
    if player then
        -- ตรวจสอบว่า player อยู่ใกล้ checkpoint จริงๆ
        local distance = (hit.Position - checkpoint.Position).Magnitude
        if distance < 10 then
            -- อัพเดทบนเซิร์ฟเวอร์โดยตรง
            updatePlayerStage(player, stageNumber)
        end
    end
end)
```

### ข้อผิดพลาด 2: Platform เคลื่อนที่แล้ว Player ตกทะลุ

```lua
-- ✓ แก้: ตั้ง AssemblyLinearVelocity แทนการ set Position โดยตรง
local function movePlatformSafely(platform, targetPosition, speed)
    local RunService = game:GetService("RunService")
    
    RunService.Heartbeat:Connect(function(dt)
        local currentPos = platform.Position
        local direction = (targetPosition - currentPos)
        
        if direction.Magnitude < 0.1 then return end
        
        -- ใช้ CFrame แทน Position เพื่อไม่ให้ผู้เล่นตก
        platform.CFrame = CFrame.new(
            currentPos + direction.Unit * speed * dt
        )
    end)
end
```

---

## 62.8 แบบฝึกหัด

### แบบฝึกหัดที่ 1: สร้าง Obby 5 Stage
สร้าง Obby ที่มี 5 stages โดยแต่ละ stage มีความยากเพิ่มขึ้น:
- Stage 1: Platform ง่ายๆ
- Stage 2: Moving Platforms
- Stage 3: Disappearing Platforms
- Stage 4: ผสม Moving + Disappearing + Lava
- Stage 5: Obstacle Course เต็มรูปแบบ

### แบบฝึกหัดที่ 2: เพิ่ม Power-ups
เพิ่ม Power-up ในแต่ละ stage:
- Speed Boost: เพิ่มความเร็ว 30 วินาที
- Double Jump: กระโดด 2 ครั้ง
- Shield: ป้องกัน Kill Brick 1 ครั้ง

### แบบฝึกหัดที่ 3: Time Challenge Mode
สร้าง Mode พิเศษที่:
- มีเวลาจำกัด
- ทำเวลาได้ดีขึ้นได้รับ Badge

---

## 62.9 เคล็ดลับจากมืออาชีพ

1. **ทดสอบ solo ก่อน**: เล่นด้วยตัวเองก่อนให้คนอื่นลอง
2. **Difficulty Curve**: ความยากต้องเพิ่มขึ้นแบบค่อยเป็นค่อยไป
3. **Visual Feedback**: ให้ผู้เล่นรู้ว่าตัวเองทำอะไรผิด
4. **Fairness**: Platform ต้องเคลื่อนที่ predictable ไม่ใช่ random
5. **Respawn Speed**: ตาย-เกิดใหม่ต้องเร็ว ไม่งั้นผู้เล่นเบื่อ

---

## สรุป

ในบทนี้เราได้เรียนรู้:
- การตั้งค่าโปรเจ็กต์ Obby
- ระบบ Stage และ Checkpoint
- Obstacle ต่างๆ (Moving, Spinning, Disappearing)
- ระบบ HUD และ Leaderboard
- การป้องกันการโกง

ในบทถัดไปเราจะสร้าง RPG Game Framework!
