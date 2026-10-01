# Part 40: Touched Events (เหตุการณ์การสัมผัส)

## บทนำ

`Touched` และ `TouchEnded` เป็น Events พื้นฐานที่สุดใน Roblox สำหรับตรวจจับการสัมผัสระหว่าง Parts เหมาะสำหรับ Traps, Collectibles, Teleporters, Buttons, และอื่นๆ อีกมากมาย

---

## 40.1 พื้นฐาน Touched Event

### Touched

```lua
-- Touched: ทำงานทุกครั้งที่ Part อื่นสัมผัส Part นี้
local part = workspace.MyPart

part.Touched:Connect(function(hit)
    -- hit: Part ที่มาสัมผัส
    print("สัมผัสโดย:", hit.Name)
    print("Parent:", hit.Parent.Name)
end)
```

### TouchEnded

```lua
-- TouchEnded: ทำงานเมื่อ Part หยุดสัมผัส
part.TouchEnded:Connect(function(hit)
    print("หยุดสัมผัส:", hit.Name)
end)
```

### ข้อควรระวัง

```lua
-- Touched เรียกหลายครั้งมาก (ทุก Frame ที่สัมผัส)
-- ต้องใช้ Debounce เสมอ!

local part = workspace.TrapPart
local debounce = false

part.Touched:Connect(function(hit)
    if debounce then return end
    debounce = true
    
    -- ทำอะไรบางอย่าง
    print("Trap activated!")
    
    task.wait(2)  -- รอ 2 วินาทีก่อน activate ได้อีก
    debounce = false
end)
```

---

## 40.2 ตรวจจับผู้เล่น

```lua
local part = workspace.InteractivePart
local Players = game:GetService("Players")

part.Touched:Connect(function(hit)
    -- วิธีที่ 1: ตรวจจาก Parent
    local character = hit.Parent
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    
    if humanoid then
        -- มี Humanoid = น่าจะเป็นผู้เล่นหรือ NPC
        local player = Players:GetPlayerFromCharacter(character)
        if player then
            print("ผู้เล่นสัมผัส:", player.Name)
        end
    end
end)

-- วิธีที่ 2: ตรวจหลายชั้น
local function getPlayerFromHit(hit)
    -- ตรวจ character โดยตรง
    local character = hit.Parent
    local player = Players:GetPlayerFromCharacter(character)
    
    -- ตรวจ character ที่ลึกกว่า (เช่น accessory)
    if not player then
        character = hit.Parent.Parent
        player = Players:GetPlayerFromCharacter(character)
    end
    
    return player
end
```

---

## 40.3 Debounce Patterns

### Per-Instance Debounce (ป้องกันการ trigger ซ้ำ)

```lua
-- ใช้ table เก็บ debounce แต่ละ instance
local part = workspace.CollectibleCoin
local touchedBy = {}  -- เก็บว่า character ไหนเก็บแล้ว

part.Touched:Connect(function(hit)
    local character = hit.Parent
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    
    if not humanoid then return end
    if touchedBy[character] then return end  -- เก็บแล้ว
    
    touchedBy[character] = true
    
    -- ให้รางวัล
    local player = game:GetService("Players"):GetPlayerFromCharacter(character)
    if player then
        print(player.Name .. " เก็บเหรียญ!")
        -- ให้เหรียญ...
        part:Destroy()
    end
end)
```

### Cooldown Debounce (ป้องกัน spam)

```lua
local part = workspace.DamagePart
local cooldowns = {}  -- cooldown แยกต่างหากสำหรับแต่ละ character

local function getDamage(character, damage)
    if cooldowns[character] then return end
    
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end
    
    cooldowns[character] = true
    humanoid:TakeDamage(damage)
    
    -- รอก่อน damage ได้อีก
    task.delay(1, function()
        cooldowns[character] = nil
    end)
end

part.Touched:Connect(function(hit)
    getDamage(hit.Parent, 10)
end)

-- ลบ cooldown เมื่อ character หาย
game:GetService("Players").PlayerRemoving:Connect(function(player)
    if player.Character then
        cooldowns[player.Character] = nil
    end
end)
```

---

## 40.4 Trap และ Hazard

### Spike Trap

```lua
-- Script ใน SpikeTrap
local trap = script.Parent
local spikes = trap:FindFirstChild("Spikes")

local damage = 50
local cooldowns = {}
local activated = false

-- Animation: spikes ขึ้นลง
local function activateSpikes()
    if activated then return end
    activated = true
    
    -- Tween spikes ขึ้น
    local TweenService = game:GetService("TweenService")
    local originalPos = spikes.Position
    local upPos = originalPos + Vector3.new(0, 3, 0)
    
    local tweenUp = TweenService:Create(
        spikes, 
        TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
        {Position = upPos}
    )
    
    local tweenDown = TweenService:Create(
        spikes,
        TweenInfo.new(0.5, Enum.EasingStyle.Quad, Enum.EasingDirection.In),
        {Position = originalPos}
    )
    
    tweenUp:Play()
    tweenUp.Completed:Connect(function()
        task.wait(0.3)
        tweenDown:Play()
        tweenDown.Completed:Connect(function()
            activated = false
        end)
    end)
end

-- Pressure Plate
local plate = trap:FindFirstChild("Plate")
if plate then
    plate.Touched:Connect(function(hit)
        local character = hit.Parent
        if not character:FindFirstChildOfClass("Humanoid") then return end
        
        activateSpikes()
    end)
end

-- Damage เมื่อโดน spike
if spikes then
    spikes.Touched:Connect(function(hit)
        local character = hit.Parent
        if cooldowns[character] then return end
        
        local humanoid = character:FindFirstChildOfClass("Humanoid")
        if not humanoid then return end
        
        cooldowns[character] = true
        humanoid:TakeDamage(damage)
        
        task.delay(0.5, function()
            cooldowns[character] = nil
        end)
    end)
end
```

### Lava Floor

```lua
-- Script ใน LavaFloor
local lava = script.Parent
local damage = 5
local damageInterval = 0.5

-- เก็บผู้เล่นที่อยู่บน lava
local onLava = {}

lava.Touched:Connect(function(hit)
    local character = hit.Parent
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    
    if not humanoid or onLava[character] then return end
    
    onLava[character] = true
    
    -- Damage loop
    task.spawn(function()
        while onLava[character] do
            if humanoid.Health > 0 then
                humanoid:TakeDamage(damage)
            end
            task.wait(damageInterval)
        end
    end)
end)

lava.TouchEnded:Connect(function(hit)
    local character = hit.Parent
    onLava[character] = nil
end)
```

### Moving Platform

```lua
-- Script ใน MovingPlatform
local platform = script.Parent
local TweenService = game:GetService("TweenService")

local point1 = platform.Position
local point2 = platform.Position + Vector3.new(0, 0, 20)
local speed = 3

local passengersAttachments = {}  -- เก็บ Attachment ของผู้โดยสาร

-- ผูก character กับ platform
local function addPassenger(character)
    if passengersAttachments[character] then return end
    
    local hrp = character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    
    -- สร้าง AlignPosition เพื่อผูก
    local attachment0 = Instance.new("Attachment", platform)
    local attachment1 = Instance.new("Attachment", hrp)
    
    local alignPos = Instance.new("AlignPosition")
    alignPos.Attachment0 = attachment1
    alignPos.Attachment1 = attachment0
    alignPos.RigidityEnabled = false
    alignPos.Responsiveness = 50
    alignPos.MaxForce = 5000
    
    -- ค่า offset
    local offset = hrp.Position - platform.Position
    attachment0.Position = offset  -- relative offset
    
    alignPos.Parent = hrp
    
    passengersAttachments[character] = {
        att0 = attachment0,
        att1 = attachment1,
        align = alignPos
    }
end

local function removePassenger(character)
    local data = passengersAttachments[character]
    if not data then return end
    
    if data.att0 and data.att0.Parent then data.att0:Destroy() end
    if data.att1 and data.att1.Parent then data.att1:Destroy() end
    if data.align and data.align.Parent then data.align:Destroy() end
    
    passengersAttachments[character] = nil
end

platform.Touched:Connect(function(hit)
    local character = hit.Parent
    if not character:FindFirstChildOfClass("Humanoid") then return end
    addPassenger(character)
end)

platform.TouchEnded:Connect(function(hit)
    local character = hit.Parent
    removePassenger(character)
end)

-- Platform movement
local function movePlatform()
    while true do
        TweenService:Create(
            platform,
            TweenInfo.new((point2 - point1).Magnitude / speed),
            {Position = point2}
        ):Play()
        task.wait((point2 - point1).Magnitude / speed + 1)
        
        TweenService:Create(
            platform,
            TweenInfo.new((point2 - point1).Magnitude / speed),
            {Position = point1}
        ):Play()
        task.wait((point2 - point1).Magnitude / speed + 1)
    end
end

task.spawn(movePlatform)
```

---

## 40.5 Collectibles System

### Coin System

```lua
-- Script ใน CoinFolder (Server)
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")

local coinEvent = ReplicatedStorage:WaitForChild("CoinEvent")

-- ค่า coin แต่ละชนิด
local coinValues = {
    Bronze = 1,
    Silver = 5,
    Gold = 10,
    Diamond = 50,
}

local function setupCoin(coin)
    local coinType = coin:GetAttribute("CoinType") or "Bronze"
    local value = coinValues[coinType] or 1
    local collected = false
    
    -- Magnetic effect - ดูดเข้าหาผู้เล่นที่ใกล้
    local RunService = game:GetService("RunService")
    local MAGNET_RANGE = 8
    
    local magnetConn = RunService.Heartbeat:Connect(function()
        if collected then return end
        
        local nearest = nil
        local nearestDist = MAGNET_RANGE
        
        for _, player in pairs(Players:GetPlayers()) do
            local char = player.Character
            if char then
                local hrp = char:FindFirstChild("HumanoidRootPart")
                if hrp then
                    local dist = (hrp.Position - coin.Position).Magnitude
                    if dist < nearestDist then
                        nearest = hrp
                        nearestDist = dist
                    end
                end
            end
        end
        
        if nearest then
            -- ดูดเข้าหา HumanoidRootPart
            local direction = (nearest.Position - coin.Position).Unit
            coin.CFrame = coin.CFrame + direction * 0.5
        end
    end)
    
    coin.Touched:Connect(function(hit)
        if collected then return end
        
        local character = hit.Parent
        local humanoid = character:FindFirstChildOfClass("Humanoid")
        if not humanoid then return end
        
        local player = Players:GetPlayerFromCharacter(character)
        if not player then return end
        
        collected = true
        magnetConn:Disconnect()
        
        -- ให้เหรียญ
        coinEvent:FireClient(player, "Collected", value, coinType, coin.Position)
        coin:Destroy()
    end)
end

-- Setup ทุก coin ใน folder
local coinsFolder = workspace:FindFirstChild("Coins")
if coinsFolder then
    for _, coin in pairs(coinsFolder:GetChildren()) do
        if coin:IsA("BasePart") then
            setupCoin(coin)
        end
    end
    
    -- Setup coins ใหม่
    coinsFolder.ChildAdded:Connect(function(coin)
        if coin:IsA("BasePart") then
            task.wait()  -- รอ properties load
            setupCoin(coin)
        end
    end)
end
```

```lua
-- LocalScript ใน StarterPlayerScripts (Client effects)
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local coinEvent = ReplicatedStorage:WaitForChild("CoinEvent")

-- แสดง +N text
local function showCoinText(value, coinType, position)
    local colors = {
        Bronze = Color3.fromRGB(180, 100, 30),
        Silver = Color3.fromRGB(200, 200, 200),
        Gold = Color3.fromRGB(255, 215, 0),
        Diamond = Color3.fromRGB(100, 200, 255),
    }
    
    local billboard = Instance.new("BillboardGui")
    billboard.Size = UDim2.new(0, 100, 0, 40)
    billboard.StudsOffset = Vector3.new(0, 2, 0)
    billboard.AlwaysOnTop = true
    
    -- ใส่ใน workspace ที่ position
    local part = Instance.new("Part")
    part.Anchored = true
    part.CanCollide = false
    part.Transparency = 1
    part.Size = Vector3.new(0.1, 0.1, 0.1)
    part.Position = position
    part.Parent = workspace
    billboard.Parent = part
    
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, 0, 1, 0)
    label.BackgroundTransparency = 1
    label.Text = "+" .. value
    label.TextColor3 = colors[coinType] or Color3.fromRGB(255, 215, 0)
    label.Font = Enum.Font.GothamBold
    label.TextSize = 24
    label.TextStrokeTransparency = 0
    label.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    label.Parent = billboard
    
    -- ลอยขึ้นแล้วหาย
    TweenService:Create(part, TweenInfo.new(1.5, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
        Position = position + Vector3.new(0, 5, 0)
    }):Play()
    
    TweenService:Create(label, TweenInfo.new(1.5), {
        TextTransparency = 1,
        TextStrokeTransparency = 1,
    }):Play()
    
    game:GetService("Debris"):AddItem(part, 1.5)
end

-- เสียงเก็บเหรียญ
local collectSound = Instance.new("Sound")
collectSound.SoundId = "rbxassetid://4590662766"
collectSound.Volume = 0.7
collectSound.Parent = player.PlayerGui

coinEvent.OnClientEvent:Connect(function(event, value, coinType, position)
    if event == "Collected" then
        collectSound:Play()
        showCoinText(value, coinType, position)
    end
end)
```

---

## 40.6 Teleporter

```lua
-- Script ใน Teleporter Pad (Server)
local teleportPad = script.Parent
local Players = game:GetService("Players")

-- Properties จาก Attributes
local targetPosition = teleportPad:GetAttribute("TargetPosition") or Vector3.new(0, 50, 0)
local cooldown = teleportPad:GetAttribute("Cooldown") or 3

local debounce = {}

teleportPad.Touched:Connect(function(hit)
    local character = hit.Parent
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    
    if not humanoid or humanoid.Health <= 0 then return end
    
    -- Debounce per character
    if debounce[character] then return end
    debounce[character] = true
    
    local hrp = character:FindFirstChild("HumanoidRootPart")
    if not hrp then
        debounce[character] = nil
        return
    end
    
    -- Teleport
    hrp.CFrame = CFrame.new(targetPosition + Vector3.new(0, 3, 0))
    
    -- เอฟเฟกต์ (บอก Client)
    local player = Players:GetPlayerFromCharacter(character)
    if player then
        local remoteEvent = game:GetService("ReplicatedStorage"):FindFirstChild("TeleportEffect")
        if remoteEvent then
            remoteEvent:FireClient(player)
        end
    end
    
    task.delay(cooldown, function()
        debounce[character] = nil
    end)
end)
```

### Teleport ระหว่าง Pads

```lua
-- Script ใน TeleportManager (Server)
local teleportPads = workspace.TeleportPads:GetChildren()
local Players = game:GetService("Players")
local cooldowns = {}

-- จับคู่ pads
for i, pad in ipairs(teleportPads) do
    local nextPad = teleportPads[i % #teleportPads + 1]
    
    pad.Touched:Connect(function(hit)
        local character = hit.Parent
        if not character:FindFirstChildOfClass("Humanoid") then return end
        if cooldowns[character] then return end
        
        local hrp = character:FindFirstChild("HumanoidRootPart")
        if not hrp then return end
        
        cooldowns[character] = true
        
        -- Teleport ไปยัง pad ถัดไป
        hrp.CFrame = CFrame.new(
            nextPad.Position + Vector3.new(0, 3, 0)
        )
        
        task.delay(1, function()
            cooldowns[character] = nil
        end)
    end)
end
```

---

## 40.7 Checkpoint System

```lua
-- Script: CheckpointSystem (Server)
local checkpoints = workspace.Checkpoints:GetChildren()
local Players = game:GetService("Players")

-- เรียงลำดับ checkpoints
table.sort(checkpoints, function(a, b)
    local numA = tonumber(a.Name:match("%d+")) or 0
    local numB = tonumber(b.Name:match("%d+")) or 0
    return numA < numB
end)

-- เก็บ checkpoint ของแต่ละผู้เล่น
local playerCheckpoints = {}

local function getCheckpointIndex(checkpoint)
    for i, cp in ipairs(checkpoints) do
        if cp == checkpoint then return i end
    end
    return 0
end

-- Setup แต่ละ checkpoint
for i, checkpoint in ipairs(checkpoints) do
    -- Visual indicator
    local indicator = Instance.new("PointLight")
    indicator.Color = Color3.fromRGB(255, 200, 50)
    indicator.Brightness = 3
    indicator.Range = 20
    indicator.Parent = checkpoint
    
    checkpoint.Touched:Connect(function(hit)
        local character = hit.Parent
        local humanoid = character:FindFirstChildOfClass("Humanoid")
        if not humanoid or humanoid.Health <= 0 then return end
        
        local player = Players:GetPlayerFromCharacter(character)
        if not player then return end
        
        local currentIndex = playerCheckpoints[player.UserId] or 0
        
        -- ต้องผ่านตามลำดับ
        if i > currentIndex then
            playerCheckpoints[player.UserId] = i
            
            -- เปลี่ยนสี indicator
            indicator.Color = Color3.fromRGB(0, 255, 0)
            
            print(player.Name .. " ผ่าน Checkpoint " .. i)
            
            -- บันทึกใน DataStore (ถ้ามี)
        end
    end)
end

-- Respawn ที่ checkpoint ล่าสุด
Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function(character)
        local hrp = character:WaitForChild("HumanoidRootPart")
        
        local checkpointIndex = playerCheckpoints[player.UserId] or 0
        
        if checkpointIndex > 0 and checkpoints[checkpointIndex] then
            task.wait(0.1)  -- รอ character load
            hrp.CFrame = CFrame.new(
                checkpoints[checkpointIndex].Position + Vector3.new(0, 3, 0)
            )
        end
    end)
end)

Players.PlayerRemoving:Connect(function(player)
    -- บันทึกก่อน remove (ถ้าใช้ DataStore)
    -- playerCheckpoints[player.UserId] = nil  -- ไม่ลบถ้าต้องการ persist
end)
```

---

## 40.8 Kill Brick และ Reset

```lua
-- Script ใน KillBrick
local killBrick = script.Parent
local cooldowns = {}

killBrick.Touched:Connect(function(hit)
    local character = hit.Parent
    if cooldowns[character] then return end
    
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if not humanoid or humanoid.Health <= 0 then return end
    
    cooldowns[character] = true
    
    -- Kill
    humanoid.Health = 0
    
    task.delay(3, function()
        cooldowns[character] = nil
    end)
end)
```

### Shrinking Kill Zone

```lua
-- Script: ใน ShrinkingZone
-- Circle zone ที่หดตัวเรื่อยๆ ผู้เล่นนอก zone โดน damage

local zone = script.Parent  -- Part วงกลม
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")

local shrinkRate = 0.1  -- หดต่อวินาที
local damage = 5
local damageInterval = 1

-- เริ่ม shrink
local shrinkTween = TweenService:Create(
    zone,
    TweenInfo.new(120, Enum.EasingStyle.Linear),  -- ใช้เวลา 120 วินาที
    {Size = Vector3.new(0.1, zone.Size.Y, 0.1)}
)
shrinkTween:Play()

-- Damage ผู้เล่นนอก zone
local lastDamageTime = {}

RunService.Heartbeat:Connect(function()
    local now = tick()
    
    for _, player in pairs(Players:GetPlayers()) do
        local char = player.Character
        if not char then continue end
        
        local hrp = char:FindFirstChild("HumanoidRootPart")
        if not hrp then continue end
        
        -- ตรวจว่าอยู่นอก zone หรือไม่
        local zoneCenter = Vector3.new(zone.Position.X, 0, zone.Position.Z)
        local playerFlat = Vector3.new(hrp.Position.X, 0, hrp.Position.Z)
        local distance = (playerFlat - zoneCenter).Magnitude
        local zoneRadius = zone.Size.X / 2
        
        if distance > zoneRadius then
            -- นอก zone - damage
            if not lastDamageTime[player] or (now - lastDamageTime[player]) >= damageInterval then
                lastDamageTime[player] = now
                local humanoid = char:FindFirstChildOfClass("Humanoid")
                if humanoid and humanoid.Health > 0 then
                    humanoid:TakeDamage(damage)
                end
            end
        end
    end
end)
```

---

## 40.9 Interactive Objects

### Door ที่เปิดเมื่อผ่านใกล้

```lua
-- Script ใน AutomaticDoor
local door = script.Parent
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")

local closedPos = door.Position
local openPos = closedPos + Vector3.new(0, door.Size.Y + 0.5, 0)

local playersNear = {}
local isOpen = false

local function openDoor()
    if isOpen then return end
    isOpen = true
    
    TweenService:Create(
        door,
        TweenInfo.new(0.5, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
        {Position = openPos}
    ):Play()
end

local function closeDoor()
    if not isOpen then return end
    isOpen = false
    
    TweenService:Create(
        door,
        TweenInfo.new(0.5, Enum.EasingStyle.Quad, Enum.EasingDirection.In),
        {Position = closedPos}
    ):Play()
end

-- Sensor parts (invisible triggers)
local sensorFront = door.Parent:FindFirstChild("SensorFront")
local sensorBack = door.Parent:FindFirstChild("SensorBack")

local function setupSensor(sensor)
    sensor.CanCollide = false
    
    sensor.Touched:Connect(function(hit)
        local character = hit.Parent
        if not character:FindFirstChildOfClass("Humanoid") then return end
        
        playersNear[character] = true
        openDoor()
    end)
    
    sensor.TouchEnded:Connect(function(hit)
        local character = hit.Parent
        playersNear[character] = nil
        
        -- ปิดถ้าไม่มีใครอยู่ใกล้
        if next(playersNear) == nil then
            task.delay(1, function()
                if next(playersNear) == nil then
                    closeDoor()
                end
            end)
        end
    end)
end

if sensorFront then setupSensor(sensorFront) end
if sensorBack then setupSensor(sensorBack) end
```

### Button ที่กดแล้วเปิดประตู

```lua
-- Script ใน ButtonPuzzle
local button = workspace.PuzzleButton
local door = workspace.PuzzleDoor
local TweenService = game:GetService("TweenService")

local buttonPressCount = 0
local requiredPresses = 3  -- กด 3 ครั้งถึงเปิด
local doorOpen = false
local buttonCooldown = false

button.Touched:Connect(function(hit)
    if buttonCooldown then return end
    if not hit.Parent:FindFirstChildOfClass("Humanoid") then return end
    
    buttonCooldown = true
    buttonPressCount = buttonPressCount + 1
    
    -- กด button ลง
    TweenService:Create(
        button,
        TweenInfo.new(0.1),
        {Position = button.Position - Vector3.new(0, 0.2, 0)}
    ):Play()
    
    task.wait(0.3)
    
    -- คืนสภาพ
    TweenService:Create(
        button,
        TweenInfo.new(0.1),
        {Position = button.Position + Vector3.new(0, 0.2, 0)}
    ):Play()
    
    print("กด:", buttonPressCount .. "/" .. requiredPresses)
    
    if buttonPressCount >= requiredPresses and not doorOpen then
        doorOpen = true
        
        -- เปิดประตู
        TweenService:Create(
            door,
            TweenInfo.new(1, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
            {Position = door.Position + Vector3.new(0, 10, 0)}
        ):Play()
        
        print("ประตูเปิดแล้ว!")
    end
    
    task.delay(0.5, function()
        buttonCooldown = false
    end)
end)
```

---

## 40.10 ระบบ CanTouch Property

```lua
-- CanTouch ควบคุมว่า Touched event จะ fire หรือไม่
local part = workspace.MyPart

-- ปิด touch detection ชั่วคราว
part.CanTouch = false

-- เปิดอีกครั้ง
part.CanTouch = true

-- ประโยชน์: ปิด touch สำหรับ Parts ที่ไม่ต้องการ interaction
-- เพื่อประหยัด performance
```

---

## 40.11 Power-Up System

```lua
-- Script ใน PowerUpManager (Server)
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")

-- ข้อมูล power-ups
local powerUpTypes = {
    Speed = {
        duration = 10,
        color = Color3.fromRGB(0, 200, 255),
        icon = "🏃",
        effect = function(humanoid)
            humanoid.WalkSpeed = humanoid.WalkSpeed * 2
            return function()
                humanoid.WalkSpeed = humanoid.WalkSpeed / 2
            end
        end
    },
    Jump = {
        duration = 15,
        color = Color3.fromRGB(100, 255, 100),
        icon = "⬆️",
        effect = function(humanoid)
            humanoid.JumpHeight = humanoid.JumpHeight * 2
            return function()
                humanoid.JumpHeight = humanoid.JumpHeight / 2
            end
        end
    },
    Invincible = {
        duration = 5,
        color = Color3.fromRGB(255, 215, 0),
        icon = "⭐",
        effect = function(humanoid)
            -- ทำให้ invincible
            local original = {}
            
            -- Override TakeDamage
            local old = humanoid.TakeDamage
            local connection = game:GetService("RunService").Heartbeat:Connect(function()
                humanoid.Health = humanoid.MaxHealth
            end)
            
            return function()
                connection:Disconnect()
            end
        end
    },
}

-- Active effects ของแต่ละผู้เล่น
local activeEffects = {}

local function applyPowerUp(player, powerUpName)
    local data = powerUpTypes[powerUpName]
    if not data then return end
    
    local character = player.Character
    if not character then return end
    
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end
    
    -- ยกเลิก effect เดิม (ถ้ามี)
    if activeEffects[player.UserId] and activeEffects[player.UserId][powerUpName] then
        local oldCleanup = activeEffects[player.UserId][powerUpName]
        oldCleanup()
    end
    
    if not activeEffects[player.UserId] then
        activeEffects[player.UserId] = {}
    end
    
    -- Apply effect
    local cleanup = data.effect(humanoid)
    activeEffects[player.UserId][powerUpName] = cleanup
    
    -- บอก client
    local powerUpEvent = ReplicatedStorage:FindFirstChild("PowerUpEvent")
    if powerUpEvent then
        powerUpEvent:FireClient(player, "Start", powerUpName, data.duration, data.color)
    end
    
    -- หมดเวลา
    task.delay(data.duration, function()
        if activeEffects[player.UserId] and 
           activeEffects[player.UserId][powerUpName] == cleanup then
            cleanup()
            activeEffects[player.UserId][powerUpName] = nil
            
            if powerUpEvent then
                powerUpEvent:FireClient(player, "End", powerUpName)
            end
        end
    end)
end

-- Setup power-up pickups
for _, powerUp in pairs(workspace.PowerUps:GetChildren()) do
    local powerUpName = powerUp:GetAttribute("PowerUpType") or "Speed"
    local respawnTime = powerUp:GetAttribute("RespawnTime") or 30
    
    powerUp.Touched:Connect(function(hit)
        local character = hit.Parent
        if not character:FindFirstChildOfClass("Humanoid") then return end
        
        local player = Players:GetPlayerFromCharacter(character)
        if not player then return end
        
        -- เก็บ power-up
        applyPowerUp(player, powerUpName)
        
        -- ซ่อนและ respawn
        powerUp.Transparency = 1
        powerUp.CanTouch = false
        
        task.delay(respawnTime, function()
            powerUp.Transparency = 0
            powerUp.CanTouch = true
        end)
    end)
end

Players.PlayerRemoving:Connect(function(player)
    -- ทำความสะอาด
    if activeEffects[player.UserId] then
        for _, cleanup in pairs(activeEffects[player.UserId]) do
            if type(cleanup) == "function" then
                cleanup()
            end
        end
        activeEffects[player.UserId] = nil
    end
end)
```

---

## 40.12 ระบบ Anti-Cheat สำหรับ Touched

```lua
-- Validate ว่า touch เกิดขึ้นจริงๆ (server-side)

local function isValidTouch(hit, target, maxDistance)
    -- ตรวจว่า hit Part อยู่ใกล้ target จริงๆ
    if not hit or not hit.Parent then return false end
    if not target then return false end
    
    local maxDist = maxDistance or 10
    local distance = (hit.Position - target.Position).Magnitude
    
    return distance <= maxDist
end

-- ตัวอย่างใช้กับ Collectible
local coin = workspace.Coin

coin.Touched:Connect(function(hit)
    -- Validate ระยะ
    if not isValidTouch(hit, coin, 5) then
        warn("Suspicious touch from:", hit.Name, "at distance:", 
             (hit.Position - coin.Position).Magnitude)
        return
    end
    
    -- ดำเนินการปกติ
    local character = hit.Parent
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if humanoid then
        print("Valid touch!")
    end
end)
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Obstacle Course
สร้าง Obby ที่มี:
- Moving Platforms (เคลื่อนที่หลายทิศ)
- Spike Traps ที่กระดกขึ้นลง
- Kill Bricks และ Reset
- Checkpoint System

### แบบฝึกหัดที่ 2: Puzzle Room
สร้าง Puzzle Room ที่:
- กดปุ่มหลายอันในลำดับที่ถูกต้อง
- ประตูเปิดเมื่อทำครบ
- มี Timer (ทำไม่ทันก็ reset)
- Multiple rooms ที่ยากขึ้นเรื่อยๆ

### แบบฝึกหัดที่ 3: Collectible Game
สร้างเกมเก็บของที่:
- Coins หลายสี ราคาต่างกัน
- Power-ups ที่เพิ่ม Speed/Score
- Timer นับถอยหลัง
- Leaderboard แสดงคะแนน

### แบบฝึกหัดที่ 4: Battle Royale Zone
สร้าง Safe Zone ที่:
- หดตัวเรื่อยๆ ตลอดเกม
- แสดง visual indicator บน map
- Damage นอก zone เพิ่มขึ้นตามเวลา
- Alert เมื่อ zone กำลังหด

---

## สรุป

ในบทนี้เราได้เรียนรู้:
- **Touched/TouchEnded**: Events พื้นฐาน
- **Debounce Patterns**: Per-instance, Cooldown
- **Hazards**: Spike Trap, Lava Floor, Kill Brick
- **Collectibles**: Coin System พร้อม Magnet Effect
- **Teleporter**: ย้ายระหว่างจุดต่างๆ
- **Checkpoint**: บันทึกความก้าวหน้า
- **Interactive Objects**: Door, Button Puzzle
- **Power-Up System**: Speed, Jump, Invincible
- **CanTouch Property**: ควบคุม performance

---

## อ้างอิง
- [BasePart.Touched](https://create.roblox.com/docs/reference/engine/classes/BasePart#Touched)
- [BasePart.TouchEnded](https://create.roblox.com/docs/reference/engine/classes/BasePart#TouchEnded)
- [BasePart.CanTouch](https://create.roblox.com/docs/reference/engine/classes/BasePart#CanTouch)
- [BasePart.CanCollide](https://create.roblox.com/docs/reference/engine/classes/BasePart#CanCollide)
- [Humanoid.TakeDamage](https://create.roblox.com/docs/reference/engine/classes/Humanoid#TakeDamage)
