# Part 68: Horror Game Atmosphere and Mechanics

## บทนำ

Horror Game ต้องอาศัยการสร้างบรรยากาศที่น่ากลัว ทั้งแสง เสียง และกลไกการเล่น ในบทนี้เราจะเรียนรู้การสร้าง Horror Game ที่มีบรรยากาศน่าตื่นเต้น พร้อมระบบ Jump Scare, Sanity, และ Monster AI

---

## 68.1 การสร้างบรรยากาศ Horror

### 68.1.1 Lighting Setup

```lua
-- ServerScriptService/HorrorLighting.lua
-- ตั้งค่าแสงสำหรับ Horror Game

local Lighting = game:GetService("Lighting")
local TweenService = game:GetService("TweenService")

-- ตั้งค่า Horror Lighting
local function setupHorrorLighting()
    Lighting.Brightness = 0
    Lighting.ClockTime = 0  -- กลางคืน
    Lighting.Ambient = Color3.fromRGB(5, 5, 15)
    Lighting.OutdoorAmbient = Color3.fromRGB(10, 10, 30)
    Lighting.FogColor = Color3.fromRGB(0, 0, 15)
    Lighting.FogEnd = 80
    Lighting.FogStart = 20
    Lighting.GlobalShadows = true
    Lighting.ShadowSoftness = 0.9
    
    -- เพิ่ม effects
    local bloom = Instance.new("BloomEffect")
    bloom.Intensity = 0.3
    bloom.Size = 24
    bloom.Threshold = 0.8
    bloom.Parent = Lighting
    
    local colorCorrection = Instance.new("ColorCorrectionEffect")
    colorCorrection.Brightness = -0.1
    colorCorrection.Contrast = 0.3
    colorCorrection.Saturation = -0.5  -- ลด saturation ให้ดูซีด
    colorCorrection.TintColor = Color3.fromRGB(200, 200, 230)  -- สีม่วงน้ำเงิน
    colorCorrection.Parent = Lighting
    
    local depthOfField = Instance.new("DepthOfFieldEffect")
    depthOfField.FarIntensity = 0.3
    depthOfField.FocusDistance = 20
    depthOfField.InFocusRadius = 10
    depthOfField.NearIntensity = 0.5
    depthOfField.Parent = Lighting
    
    print("ตั้งค่า Horror Lighting เสร็จแล้ว")
end

-- Lightning Flash (ฟ้าแลบ)
local function lightningFlash()
    TweenService:Create(Lighting, TweenInfo.new(0.05), {
        Brightness = 5
    }):Play()
    
    task.wait(0.05)
    
    TweenService:Create(Lighting, TweenInfo.new(0.1), {
        Brightness = 0
    }):Play()
    
    task.wait(0.1)
    
    -- Flash อีกครั้ง
    TweenService:Create(Lighting, TweenInfo.new(0.02), {
        Brightness = 3
    }):Play()
    
    task.wait(0.02)
    
    TweenService:Create(Lighting, TweenInfo.new(0.15), {
        Brightness = 0
    }):Play()
    
    -- เสียงฟ้าร้อง (delay)
    task.delay(0.3, function()
        local thunder = Instance.new("Sound")
        thunder.SoundId = "rbxassetid://3259536998"
        thunder.Volume = 0.8
        thunder.Parent = workspace
        thunder:Play()
        game:GetService("Debris"):AddItem(thunder, 5)
    end)
end

-- สุ่มฟ้าแลบ
local function startWeatherSystem()
    while true do
        local nextFlash = math.random(10, 60)  -- ทุก 10-60 วินาที
        task.wait(nextFlash)
        lightningFlash()
    end
end

setupHorrorLighting()
task.spawn(startWeatherSystem)
```

### 68.1.2 Ambient Sound System

```lua
-- ServerScriptService/HorrorSound.lua
-- ระบบเสียงบรรยากาศ Horror

local SoundService = game:GetService("SoundService")
local Players = game:GetService("Players")

-- เสียง Horror
local horrorSounds = {
    ambient = {
        "rbxassetid://9120268931",  -- เสียงลม
        "rbxassetid://9120268932",  -- เสียงนกฮูก
        "rbxassetid://9120268933",  -- เสียงสาดน้ำ
    },
    stinger = {  -- เสียงกระตุก (jump scare)
        "rbxassetid://9120268934",
        "rbxassetid://9120268935",
        "rbxassetid://9120268936",
    },
    monster = {
        "rbxassetid://9120268937",  -- เสียงครางของ monster
        "rbxassetid://9120268938",  -- เสียงเท้า
    },
    environment = {
        creaking = "rbxassetid://9120268939",  -- เสียงพื้นดังครืด
        dripping = "rbxassetid://9120268940",  -- เสียงน้ำหยด
        heartbeat = "rbxassetid://9120268941", -- เสียงหัวใจเต้น
    }
}

-- สร้าง Ambient Sound ถาวร
local function setupAmbientSounds()
    local soundPart = workspace:FindFirstChild("AmbientSounds") or Instance.new("Part")
    soundPart.Name = "AmbientSounds"
    soundPart.Anchored = true
    soundPart.CanCollide = false
    soundPart.Transparency = 1
    soundPart.Position = Vector3.new(0, 0, 0)
    soundPart.Parent = workspace
    
    -- เสียงลมพัด
    local windSound = Instance.new("Sound")
    windSound.Name = "Wind"
    windSound.SoundId = "rbxassetid://9120268931"
    windSound.Looped = true
    windSound.Volume = 0.3
    windSound.RollOffMaxDistance = 10000
    windSound.Parent = soundPart
    windSound:Play()
    
    -- เสียงน้ำหยด
    local drippingSound = Instance.new("Sound")
    drippingSound.Name = "Dripping"
    drippingSound.SoundId = horrorSounds.environment.dripping
    drippingSound.Looped = true
    drippingSound.Volume = 0.2
    drippingSound.Parent = soundPart
    drippingSound:Play()
    
    print("ตั้งค่า Ambient Sounds เสร็จแล้ว")
end

-- Random Stinger (เสียงกระตุก)
local function playRandomStinger()
    while true do
        task.wait(math.random(30, 120))  -- สุ่มทุก 30-120 วินาที
        
        local stingerId = horrorSounds.stinger[math.random(#horrorSounds.stinger)]
        local stinger = Instance.new("Sound")
        stinger.SoundId = stingerId
        stinger.Volume = 0.5
        stinger.Parent = workspace
        stinger:Play()
        game:GetService("Debris"):AddItem(stinger, 5)
    end
end

-- เสียงพื้นดังเมื่อผู้เล่นเดิน
local function setupCreakSounds()
    local creakPositions = {
        Vector3.new(10, 0, 10),
        Vector3.new(-15, 0, 20),
        Vector3.new(5, 0, -30),
    }
    
    for _, pos in ipairs(creakPositions) do
        local trigger = Instance.new("Part")
        trigger.Size = Vector3.new(5, 1, 5)
        trigger.Position = pos
        trigger.Anchored = true
        trigger.CanCollide = false
        trigger.Transparency = 1
        trigger.Parent = workspace
        
        local sound = Instance.new("Sound")
        sound.SoundId = horrorSounds.environment.creaking
        sound.Volume = 0.6
        sound.Parent = trigger
        
        local triggered = false
        trigger.Touched:Connect(function(hit)
            local character = hit.Parent
            if character:FindFirstChildWhichIsA("Humanoid") and not triggered then
                triggered = true
                sound:Play()
                task.delay(10, function() triggered = false end)
            end
        end)
    end
end

setupAmbientSounds()
task.spawn(playRandomStinger)
setupCreakSounds()
```

---

## 68.2 ระบบ Sanity

```lua
-- ServerScriptService/SanitySystem.lua
-- ระบบความสติ (Sanity)

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")

local Remotes = ReplicatedStorage:WaitForChild("Remotes")
local sanityUpdateEvent = Instance.new("RemoteEvent")
sanityUpdateEvent.Name = "SanityUpdate"
sanityUpdateEvent.Parent = Remotes

-- ข้อมูล Sanity
local playerSanity = {}
local MAX_SANITY = 100
local MIN_SANITY = 0

-- ตั้งค่าเริ่มต้น
Players.PlayerAdded:Connect(function(player)
    playerSanity[player.UserId] = MAX_SANITY
end)

Players.PlayerRemoving:Connect(function(player)
    playerSanity[player.UserId] = nil
end)

-- ลด Sanity
local function decreaseSanity(player, amount)
    if not playerSanity[player.UserId] then return end
    
    playerSanity[player.UserId] = math.max(MIN_SANITY, playerSanity[player.UserId] - amount)
    sanityUpdateEvent:FireClient(player, playerSanity[player.UserId])
end

-- เพิ่ม Sanity (เมื่ออยู่ในแสง)
local function increaseSanity(player, amount)
    if not playerSanity[player.UserId] then return end
    
    playerSanity[player.UserId] = math.min(MAX_SANITY, playerSanity[player.UserId] + amount)
    sanityUpdateEvent:FireClient(player, playerSanity[player.UserId])
end

-- ตรวจสอบแสงรอบตัวผู้เล่น
local function checkPlayerLight(player)
    local character = player.Character
    if not character or not character:FindFirstChild("HumanoidRootPart") then return 0 end
    
    local rootPos = character.HumanoidRootPart.Position
    local totalLight = 0
    
    -- ตรวจสอบ PointLights ในระยะ 20 studs
    for _, light in ipairs(workspace:GetDescendants()) do
        if light:IsA("PointLight") or light:IsA("SpotLight") then
            local lightParent = light.Parent
            if lightParent:IsA("BasePart") then
                local dist = (lightParent.Position - rootPos).Magnitude
                if dist < 20 then
                    local brightness = light.Brightness * (1 - dist/20)
                    totalLight = totalLight + brightness
                end
            end
        end
    end
    
    return totalLight
end

-- ลด Sanity เมื่ออยู่ในความมืด
RunService.Heartbeat:Connect(function()
    for _, player in ipairs(Players:GetPlayers()) do
        local lightLevel = checkPlayerLight(player)
        
        if lightLevel < 1 then
            decreaseSanity(player, 0.01)  -- ลดช้าๆ
        elseif lightLevel > 3 then
            increaseSanity(player, 0.02)  -- ฟื้นฟูในแสง
        end
        
        -- ถ้า Sanity ต่ำมาก
        if playerSanity[player.UserId] and playerSanity[player.UserId] < 20 then
            -- เพิ่ม hallucination effect
            local hallEvent = Remotes:FindFirstChild("Hallucination")
            if hallEvent then
                hallEvent:FireClient(player, playerSanity[player.UserId])
            end
        end
    end
end)

-- Monster ลด Sanity เมื่อเห็น
local function onPlayerSeeMonster(player, monster)
    local distance = 0
    local character = player.Character
    if character and character:FindFirstChild("HumanoidRootPart") and monster:FindFirstChild("HumanoidRootPart") then
        distance = (character.HumanoidRootPart.Position - monster.HumanoidRootPart.Position).Magnitude
    end
    
    -- ยิ่งใกล้ยิ่งลด Sanity มาก
    local sanityDamage = math.max(1, 20 - distance/5)
    decreaseSanity(player, sanityDamage)
end

return {
    decreaseSanity = decreaseSanity,
    increaseSanity = increaseSanity,
    getSanity = function(player) return playerSanity[player.UserId] end,
    onPlayerSeeMonster = onPlayerSeeMonster
}
```

---

## 68.3 Sanity Effects (Client)

```lua
-- StarterPlayerScripts/SanityEffects.lua
-- เอฟเฟกต์ Sanity ฝั่ง Client

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local Lighting = game:GetService("Lighting")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

-- สร้าง Sanity UI
local playerGui = player.PlayerGui
local sanityGui = Instance.new("ScreenGui")
sanityGui.Name = "SanityGui"
sanityGui.ResetOnSpawn = false
sanityGui.Parent = playerGui

-- Vignette (ขอบมืด)
local vignette = Instance.new("ImageLabel")
vignette.Name = "Vignette"
vignette.Size = UDim2.new(1, 0, 1, 0)
vignette.BackgroundTransparency = 1
vignette.Image = "rbxassetid://1079185975"  -- Vignette texture
vignette.ImageColor3 = Color3.new(0, 0, 0)
vignette.ImageTransparency = 0.5
vignette.ZIndex = 5
vignette.Parent = sanityGui

-- Sanity Bar
local sanityFrame = Instance.new("Frame")
sanityFrame.Name = "SanityFrame"
sanityFrame.Size = UDim2.new(0, 200, 0, 20)
sanityFrame.Position = UDim2.new(0, 10, 1, -30)
sanityFrame.BackgroundColor3 = Color3.fromRGB(50, 0, 50)
sanityFrame.BorderSizePixel = 0
sanityFrame.Parent = sanityGui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 5)
corner.Parent = sanityFrame

local sanityBar = Instance.new("Frame")
sanityBar.Name = "Bar"
sanityBar.Size = UDim2.new(1, 0, 1, 0)
sanityBar.BackgroundColor3 = Color3.fromRGB(150, 0, 200)
sanityBar.BorderSizePixel = 0
sanityBar.Parent = sanityFrame

local corner2 = Instance.new("UICorner")
corner2.CornerRadius = UDim.new(0, 5)
corner2.Parent = sanityBar

local sanityLabel = Instance.new("TextLabel")
sanityLabel.Size = UDim2.new(1, 0, 1, 0)
sanityLabel.BackgroundTransparency = 1
sanityLabel.Text = "สติ: 100%"
sanityLabel.TextColor3 = Color3.new(1, 1, 1)
sanityLabel.TextScaled = true
sanityLabel.Font = Enum.Font.Gotham
sanityLabel.Parent = sanityFrame

-- Overlay สำหรับ Hallucination
local overlay = Instance.new("Frame")
overlay.Name = "SanityOverlay"
overlay.Size = UDim2.new(1, 0, 1, 0)
overlay.BackgroundColor3 = Color3.fromRGB(50, 0, 80)
overlay.BackgroundTransparency = 1
overlay.ZIndex = 10
overlay.Parent = sanityGui

-- Heart Effect
local heartbeatGui = Instance.new("TextLabel")
heartbeatGui.Size = UDim2.new(1, 0, 1, 0)
heartbeatGui.BackgroundTransparency = 1
heartbeatGui.Text = ""
heartbeatGui.ZIndex = 15
heartbeatGui.Parent = sanityGui

-- รับอัพเดท Sanity
local currentSanity = 100
local Remotes = ReplicatedStorage:WaitForChild("Remotes")
local sanityEvent = Remotes:WaitForChild("SanityUpdate")

sanityEvent.OnClientEvent:Connect(function(sanityValue)
    currentSanity = sanityValue
    
    -- อัพเดท Bar
    local percent = sanityValue / 100
    TweenService:Create(sanityBar, TweenInfo.new(0.5), {
        Size = UDim2.new(percent, 0, 1, 0)
    }):Play()
    
    sanityLabel.Text = "สติ: " .. math.floor(sanityValue) .. "%"
    
    -- เปลี่ยนสี bar ตาม sanity
    local r = 1 - percent
    local g = 0
    local b = percent
    sanityBar.BackgroundColor3 = Color3.new(r, g, b)
    
    -- Vignette effect
    vignette.ImageTransparency = 0.3 + percent * 0.5
    
    -- เอฟเฟกต์เมื่อ Sanity ต่ำ
    if sanityValue < 30 then
        -- Color Correction หนักขึ้น
        local cc = Lighting:FindFirstChildWhichIsA("ColorCorrectionEffect")
        if cc then
            TweenService:Create(cc, TweenInfo.new(1), {
                Saturation = -0.9,
                Contrast = 0.5
            }):Play()
        end
    end
end)

-- Hallucination Effects
local hallEvent = Remotes:WaitForChild("Hallucination")

local hallucinationActive = false
local hallucinationConnection

hallEvent.OnClientEvent:Connect(function(sanityValue)
    if hallucinationActive then return end
    hallucinationActive = true
    
    -- Screen shake
    local originalCFrame = camera.CFrame
    local shakeIntensity = (100 - sanityValue) / 100
    
    hallucinationConnection = RunService.RenderStepped:Connect(function()
        local shakeX = (math.random() - 0.5) * shakeIntensity * 2
        local shakeY = (math.random() - 0.5) * shakeIntensity * 2
        
        camera.CFrame = camera.CFrame * CFrame.Angles(
            math.rad(shakeY * 0.5),
            math.rad(shakeX * 0.5),
            0
        )
    end)
    
    -- Overlay flash
    TweenService:Create(overlay, TweenInfo.new(0.3), {
        BackgroundTransparency = 0.7
    }):Play()
    
    task.wait(0.5)
    
    TweenService:Create(overlay, TweenInfo.new(0.5), {
        BackgroundTransparency = 1
    }):Play()
    
    task.wait(0.5)
    
    if hallucinationConnection then
        hallucinationConnection:Disconnect()
        hallucinationConnection = nil
    end
    
    hallucinationActive = false
end)

-- Heartbeat เมื่อ Sanity ต่ำ
RunService.RenderStepped:Connect(function()
    if currentSanity < 50 then
        -- เสียงหัวใจเต้นเร็วขึ้น
        local heartRate = math.floor((100 - currentSanity) / 10)  -- เต้น heartRate ครั้งต่อนาที offset
    end
end)
```

---

## 68.4 Monster AI

```lua
-- ServerScriptService/MonsterAI.lua
-- ระบบ AI สำหรับ Monster

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local PathfindingService = game:GetService("PathfindingService")

-- สร้าง Monster
local function createMonster(spawnPosition)
    local monster = Instance.new("Model")
    monster.Name = "Monster"
    
    -- Body
    local root = Instance.new("Part")
    root.Name = "HumanoidRootPart"
    root.Size = Vector3.new(2, 2, 1)
    root.Position = spawnPosition
    root.Anchored = false
    root.CanCollide = true
    root.Transparency = 1
    root.Parent = monster
    monster.PrimaryPart = root
    
    local torso = Instance.new("Part")
    torso.Name = "Torso"
    torso.Size = Vector3.new(3, 4, 2)
    torso.Position = spawnPosition
    torso.BrickColor = BrickColor.new("Black")
    torso.Material = Enum.Material.Neon
    torso.Parent = monster
    
    local head = Instance.new("Part")
    head.Name = "Head"
    head.Size = Vector3.new(3, 3, 3)
    head.Position = spawnPosition + Vector3.new(0, 3, 0)
    head.BrickColor = BrickColor.new("Really black")
    
    -- เพิ่มดวงตาเรืองแสง
    local leftEye = Instance.new("Part")
    leftEye.Size = Vector3.new(0.5, 0.5, 0.5)
    leftEye.Position = spawnPosition + Vector3.new(-0.8, 3.2, 1.2)
    leftEye.BrickColor = BrickColor.new("Bright red")
    leftEye.Material = Enum.Material.Neon
    leftEye.Parent = monster
    
    local rightEye = Instance.new("Part")
    rightEye.Size = Vector3.new(0.5, 0.5, 0.5)
    rightEye.Position = spawnPosition + Vector3.new(0.8, 3.2, 1.2)
    rightEye.BrickColor = BrickColor.new("Bright red")
    rightEye.Material = Enum.Material.Neon
    rightEye.Parent = monster
    
    -- แสงจากดวงตา
    local eyeLight = Instance.new("PointLight")
    eyeLight.Color = Color3.fromRGB(255, 0, 0)
    eyeLight.Brightness = 3
    eyeLight.Range = 10
    eyeLight.Parent = rightEye
    
    head.Parent = monster
    
    -- Humanoid
    local humanoid = Instance.new("Humanoid")
    humanoid.MaxHealth = 300
    humanoid.Health = 300
    humanoid.WalkSpeed = 12  -- เดินช้ากว่าผู้เล่นเล็กน้อย
    humanoid.DisplayDistanceType = Enum.HumanoidDisplayDistanceType.None
    humanoid.HealthDisplayDistanceType = Enum.HumanoidHealthDisplayDistanceType.None
    humanoid.Parent = monster
    
    -- เชื่อม parts
    local rootWeld = Instance.new("WeldConstraint")
    rootWeld.Part0 = root
    rootWeld.Part1 = torso
    rootWeld.Parent = monster
    
    local headWeld = Instance.new("WeldConstraint")
    headWeld.Part0 = torso
    headWeld.Part1 = head
    headWeld.Parent = monster
    
    monster.Parent = workspace
    
    -- เพิ่ม Attributes
    monster:SetAttribute("State", "patrol")  -- patrol, chase, attack
    monster:SetAttribute("AlertLevel", 0)    -- 0-100
    
    return monster
end

-- AI ของ Monster
local function startMonsterAI(monster)
    local humanoid = monster:FindFirstChildWhichIsA("Humanoid")
    if not humanoid then return end
    
    local state = "patrol"
    local target = nil
    local patrolPoints = {}
    local patrolIndex = 1
    
    -- สร้าง patrol points รอบ spawn
    local spawnPos = monster.PrimaryPart.Position
    for i = 1, 4 do
        local angle = (i / 4) * math.pi * 2
        local radius = 20
        table.insert(patrolPoints, Vector3.new(
            spawnPos.X + math.cos(angle) * radius,
            spawnPos.Y,
            spawnPos.Z + math.sin(angle) * radius
        ))
    end
    
    -- ตรวจสอบผู้เล่นในระยะ
    local function detectPlayers()
        local nearestPlayer = nil
        local nearestDist = 40  -- ระยะมองเห็น
        
        for _, player in ipairs(Players:GetPlayers()) do
            if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                local dist = (player.Character.HumanoidRootPart.Position - 
                    monster.PrimaryPart.Position).Magnitude
                
                if dist < nearestDist then
                    nearestDist = dist
                    nearestPlayer = player
                end
            end
        end
        
        return nearestPlayer, nearestDist
    end
    
    -- เดิน
    local function moveTo(position)
        if not monster.PrimaryPart then return end
        
        -- Pathfinding
        local path = PathfindingService:CreatePath({
            AgentRadius = 2,
            AgentHeight = 5,
            AgentCanJump = true,
        })
        
        local success = pcall(function()
            path:ComputeAsync(monster.PrimaryPart.Position, position)
        end)
        
        if success and path.Status == Enum.PathStatus.Success then
            local waypoints = path:GetWaypoints()
            
            for _, wp in ipairs(waypoints) do
                if wp.Action == Enum.PathWaypointAction.Jump then
                    humanoid.Jump = true
                end
                
                humanoid:MoveTo(wp.Position)
                humanoid.MoveToFinished:Wait()
                
                -- ตรวจสอบ state ทุก waypoint
                if state ~= monster:GetAttribute("State") then
                    break
                end
            end
        else
            -- fallback: ย้ายตรงๆ
            humanoid:MoveTo(position)
        end
    end
    
    -- เสียง Monster
    local function playMonsterSound(soundType)
        local sounds = {
            growl = "rbxassetid://9120268937",
            scream = "rbxassetid://9120268942",
            footstep = "rbxassetid://9120268938",
        }
        
        local sound = Instance.new("Sound")
        sound.SoundId = sounds[soundType] or sounds.growl
        sound.Volume = 0.8
        sound.RollOffMaxDistance = 60
        sound.Parent = monster.PrimaryPart
        sound:Play()
        game:GetService("Debris"):AddItem(sound, 5)
    end
    
    -- โจมตีผู้เล่น
    local lastAttack = 0
    local function attackPlayer(player)
        if tick() - lastAttack < 1.5 then return end  -- cooldown 1.5s
        
        if not player.Character then return end
        local dist = (player.Character.HumanoidRootPart.Position - 
            monster.PrimaryPart.Position).Magnitude
        
        if dist < 8 then
            lastAttack = tick()
            
            local humanoidTarget = player.Character:FindFirstChildWhichIsA("Humanoid")
            if humanoidTarget then
                humanoidTarget:TakeDamage(50)
                playMonsterSound("scream")
                
                -- Sanity damage
                local SanitySystem = require(script.Parent.SanitySystem)
                SanitySystem.decreaseSanity(player, 30)
                
                print("Monster โจมตี " .. player.Name .. "!")
            end
        end
    end
    
    -- Main AI Loop
    task.spawn(function()
        while monster.Parent do
            local player, dist = detectPlayers()
            
            if player then
                -- ตรวจสอบ Line of Sight
                local monsterPos = monster.PrimaryPart.Position
                local playerPos = player.Character.HumanoidRootPart.Position
                
                local rayResult = workspace:Raycast(
                    monsterPos,
                    (playerPos - monsterPos),
                    RaycastParams.new()
                )
                
                local hasLOS = rayResult and rayResult.Instance and
                    rayResult.Instance:FindFirstAncestorWhichIsA("Model") == player.Character
                
                if hasLOS then
                    if state ~= "chase" then
                        state = "chase"
                        monster:SetAttribute("State", "chase")
                        playMonsterSound("growl")
                    end
                    
                    target = player
                    
                    -- วิ่งเร็วขึ้น
                    humanoid.WalkSpeed = 18
                    
                    -- ย้ายไปหาผู้เล่น
                    task.spawn(function()
                        moveTo(playerPos)
                    end)
                    
                    -- โจมตี
                    attackPlayer(player)
                    
                    -- Sanity
                    local SanitySystem = require(script.Parent.SanitySystem)
                    SanitySystem.onPlayerSeeMonster(player, monster)
                    
                elseif state == "chase" and dist > 60 then
                    -- หยุดไล่ถ้าผู้เล่นหนีไปไกล
                    state = "patrol"
                    monster:SetAttribute("State", "patrol")
                    humanoid.WalkSpeed = 12
                    target = nil
                end
            else
                if state ~= "patrol" then
                    state = "patrol"
                    monster:SetAttribute("State", "patrol")
                    humanoid.WalkSpeed = 12
                end
                
                -- Patrol
                local nextPoint = patrolPoints[patrolIndex]
                task.spawn(function()
                    moveTo(nextPoint)
                end)
                
                task.wait(3)
                patrolIndex = (patrolIndex % #patrolPoints) + 1
            end
            
            task.wait(0.5)  -- Update rate
        end
    end)
end

-- สร้าง Monster
local monsterSpawns = {
    Vector3.new(50, 5, 50),
    Vector3.new(-50, 5, 50),
    Vector3.new(0, 5, -80),
}

for _, spawnPos in ipairs(monsterSpawns) do
    local monster = createMonster(spawnPos)
    startMonsterAI(monster)
    print("สร้าง Monster ที่ " .. tostring(spawnPos))
end
```

---

## 68.5 Jump Scare System

```lua
-- ServerScriptService/JumpScare.lua
-- ระบบ Jump Scare

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local Remotes = ReplicatedStorage:WaitForChild("Remotes")
local jumpScareEvent = Instance.new("RemoteEvent")
jumpScareEvent.Name = "JumpScare"
jumpScareEvent.Parent = Remotes

-- ตำแหน่ง Jump Scare Triggers
local jumpScareTriggers = {
    {
        position = Vector3.new(10, 5, 30),
        size = Vector3.new(5, 5, 5),
        imageId = "rbxassetid://9120268943",
        soundId = "rbxassetid://9120268944",
        cooldown = 300  -- ทุก 5 นาที
    },
    {
        position = Vector3.new(-20, 5, -10),
        size = Vector3.new(4, 4, 4),
        imageId = "rbxassetid://9120268945",
        soundId = "rbxassetid://9120268946",
        cooldown = 600
    }
}

local scareTimers = {}

for i, scareTrigger in ipairs(jumpScareTriggers) do
    local trigger = Instance.new("Part")
    trigger.Size = scareTrigger.size
    trigger.Position = scareTrigger.position
    trigger.Anchored = true
    trigger.CanCollide = false
    trigger.Transparency = 1
    trigger.Parent = workspace
    
    local lastScareTime = {}
    
    trigger.Touched:Connect(function(hit)
        local character = hit.Parent
        local player = Players:GetPlayerFromCharacter(character)
        if not player then return end
        
        local now = tick()
        local lastScare = lastScareTime[player.UserId] or 0
        
        if now - lastScare < scareTrigger.cooldown then return end
        
        lastScareTime[player.UserId] = now
        
        -- ส่ง Jump Scare ไปที่ Client
        jumpScareEvent:FireClient(player, {
            imageId = scareTrigger.imageId,
            soundId = scareTrigger.soundId,
            duration = 1.5
        })
        
        -- ลด Sanity
        local SanitySystem = require(script.Parent.SanitySystem)
        SanitySystem.decreaseSanity(player, 25)
        
        print("Jump Scare! -> " .. player.Name)
    end)
end

-- Client: รับ Jump Scare
-- StarterPlayerScripts/JumpScareClient.lua
```

```lua
-- StarterPlayerScripts/JumpScareClient.lua
-- Client handler สำหรับ Jump Scare

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local playerGui = player.PlayerGui

local Remotes = ReplicatedStorage:WaitForChild("Remotes")
local jumpScareEvent = Remotes:WaitForChild("JumpScare")

jumpScareEvent.OnClientEvent:Connect(function(data)
    -- สร้าง Full Screen Image
    local scareGui = Instance.new("ScreenGui")
    scareGui.Name = "JumpScare"
    scareGui.IgnoreGuiInset = true
    scareGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    scareGui.Parent = playerGui
    
    local scareImage = Instance.new("ImageLabel")
    scareImage.Size = UDim2.new(1, 0, 1, 0)
    scareImage.BackgroundColor3 = Color3.new(0, 0, 0)
    scareImage.Image = data.imageId or ""
    scareImage.ScaleType = Enum.ScaleType.Fit
    scareImage.ZIndex = 100
    scareImage.Parent = scareGui
    
    -- เสียง
    local scareSound = Instance.new("Sound")
    scareSound.SoundId = data.soundId or ""
    scareSound.Volume = 1
    scareSound.Parent = scareGui
    scareSound:Play()
    
    -- เขย่ากล้อง
    local camera = workspace.CurrentCamera
    for i = 1, 10 do
        camera.CFrame = camera.CFrame * CFrame.Angles(
            math.rad((math.random() - 0.5) * 5),
            math.rad((math.random() - 0.5) * 5),
            0
        )
        task.wait(0.03)
    end
    
    -- Fade out
    task.wait(data.duration or 1.5)
    
    TweenService:Create(scareImage, TweenInfo.new(0.5), {
        ImageTransparency = 1,
        BackgroundTransparency = 1
    }):Play()
    
    task.wait(0.5)
    scareGui:Destroy()
end)
```

---

## 68.6 ข้อผิดพลาดที่พบบ่อย

```lua
-- ❌ ผิด: Jump Scare บ่อยเกินไป ทำให้ผู้เล่นชิน
-- Cooldown สั้นเกินไป
trigger.Touched:Connect(function()
    playJumpScare()  -- ทุกครั้งที่แตะ
end)

-- ✓ ถูก: ใช้ Cooldown และ randomize
local lastScare = {}
trigger.Touched:Connect(function(hit)
    local player = Players:GetPlayerFromCharacter(hit.Parent)
    if not player then return end
    
    local cooldown = 300 + math.random(0, 300)  -- 5-10 นาที
    if tick() - (lastScare[player.UserId] or 0) < cooldown then return end
    
    -- Jump Scare มีโอกาส 30% เท่านั้น
    if math.random(100) > 30 then return end
    
    lastScare[player.UserId] = tick()
    playJumpScare(player)
end)
```

---

## 68.7 แบบฝึกหัด

### แบบฝึกหัดที่ 1: สร้างห้องปริศนา Horror
ห้องที่ต้อง:
- หาไฟฉาย
- แก้ปริศนา (หาตัวเลข 3 ตัว)
- ออกจากห้องก่อน Monster มาถึง

### แบบฝึกหัดที่ 2: เพิ่มระบบ Hiding
ให้ผู้เล่นซ่อนในตู้/ใต้เตียง:
- Monster ไม่เห็น
- Sanity ฟื้นฟูช้าๆ ขณะซ่อน

### แบบฝึกหัดที่ 3: สร้าง Multiple Monsters
Monster แต่ละตัวมีพฤติกรรมต่างกัน:
- Blind Monster: ตาบอด ใช้เสียงตาม
- Fast Monster: วิ่งเร็ว แต่ range มองเห็นสั้น

---

## สรุป

ในบทนี้เราได้สร้าง Horror Game ที่มี:
- Lighting และ Atmosphere
- ระบบ Sanity
- Monster AI ด้วย Pathfinding
- Jump Scare System
- เอฟเฟกต์ Hallucination

ในบทถัดไปเราจะสร้าง Social Game!
