# Part 66: Racing Game with Vehicles

## บทนำ

Racing Game เป็นประเภทเกมที่ผู้เล่นแข่งขันรถกัน Roblox มี VehicleSeat และ BodyVelocity/VectorForce ที่ช่วยให้สร้างรถได้อย่างง่ายดาย ในบทนี้เราจะสร้าง Racing Game ที่มีระบบรถ, การจัดอันดับ, Lap System, และ Drifting

---

## 66.1 โครงสร้างเกม

```
Workspace
├── Track (Model)
│   ├── Road Parts
│   ├── Checkpoints (Folder)
│   └── StartFinishLine
├── Cars (Folder)
│   └── Car_Template (Model)
├── Barriers
└── Decorations

ServerScriptService
├── RaceManager (Script)
├── LapTracker (Script)
└── VehicleManager (Script)

StarterGui
└── RaceHUD (ScreenGui)
```

---

## 66.2 การสร้างรถ

### 66.2.1 Car Model Setup

```lua
-- Script สำหรับสร้างรถ (ใน ServerScriptService/VehicleManager)
-- โครงสร้างรถ:
-- Car (Model)
--   ├── Body (Part) [PrimaryPart]
--   ├── Seat (VehicleSeat)
--   ├── FL_Wheel (Part)  -- Front Left
--   ├── FR_Wheel (Part)  -- Front Right  
--   ├── RL_Wheel (Part)  -- Rear Left
--   ├── RR_Wheel (Part)  -- Rear Right
--   └── CarScript (Script)

local function buildCar(carData, spawnPosition)
    local car = Instance.new("Model")
    car.Name = carData.name
    
    -- Body (ตัวรถหลัก)
    local body = Instance.new("Part")
    body.Name = "Body"
    body.Size = carData.size or Vector3.new(8, 2, 4)
    body.Position = spawnPosition
    body.BrickColor = carData.color or BrickColor.new("Bright red")
    body.Material = Enum.Material.SmoothPlastic
    body.CustomPhysicalProperties = PhysicalProperties.new(
        2,    -- density (หนัก)
        0.3,  -- friction
        0.1,  -- elasticity
        1, 0.5
    )
    car.PrimaryPart = body
    body.Parent = car
    
    -- VehicleSeat
    local seat = Instance.new("VehicleSeat")
    seat.Name = "Seat"
    seat.Size = Vector3.new(4, 0.5, 2)
    seat.Position = spawnPosition + Vector3.new(0, 1, 0)
    seat.CFrame = CFrame.new(spawnPosition + Vector3.new(0, 1.5, 0))
    seat.Anchored = false
    seat.CanCollide = false
    
    -- ตั้งค่าการขับ
    seat.MaxSpeed = carData.maxSpeed or 100
    seat.Torque = carData.torque or 150
    seat.TurnSpeed = carData.turnSpeed or 1.5
    seat.HeadsUpDisplay = false  -- ซ่อน default HUD
    
    seat.Parent = car
    
    -- เชื่อม Seat กับ Body
    local seatWeld = Instance.new("WeldConstraint")
    seatWeld.Part0 = body
    seatWeld.Part1 = seat
    seatWeld.Parent = car
    
    -- สร้างล้อ
    local wheelPositions = {
        FL = Vector3.new(3, -0.5, -2.5),    -- Front Left
        FR = Vector3.new(3, -0.5, 2.5),     -- Front Right
        RL = Vector3.new(-3, -0.5, -2.5),   -- Rear Left
        RR = Vector3.new(-3, -0.5, 2.5),    -- Rear Right
    }
    
    for wheelName, offset in pairs(wheelPositions) do
        local wheel = Instance.new("Part")
        wheel.Name = wheelName .. "_Wheel"
        wheel.Size = Vector3.new(0.5, 2, 2)
        wheel.Position = spawnPosition + offset
        wheel.BrickColor = BrickColor.new("Really black")
        wheel.Material = Enum.Material.SmoothPlastic
        wheel.CustomPhysicalProperties = PhysicalProperties.new(
            0.5, 0.8, 0.1, 1, 0.5
        )
        wheel.Parent = car
        
        -- เชื่อมล้อกับ Body ด้วย CylindricalConstraint (ให้ล้อหมุนได้)
        local attachment0 = Instance.new("Attachment")
        attachment0.Position = Vector3.new(0, 0, 0)
        attachment0.Parent = body
        
        local attachment1 = Instance.new("Attachment")
        attachment1.Position = Vector3.new(0, 0, 0)
        attachment1.Parent = wheel
        
        -- HingeConstraint (ล้อหมุนรอบแกน)
        local hinge = Instance.new("HingeConstraint")
        hinge.Attachment0 = attachment0
        hinge.Attachment1 = attachment1
        hinge.ActuatorType = Enum.ActuatorType.None
        hinge.LimitsEnabled = false
        hinge.Parent = car
        
        -- WeldConstraint สำหรับล้อหน้า (สามารถหันได้)
        if wheelName:sub(1, 1) == "F" then
            -- ล้อหน้าใช้ HingeConstraint
        else
            -- ล้อหลังใช้ WeldConstraint
            local weld = Instance.new("WeldConstraint")
            weld.Part0 = body
            weld.Part1 = wheel
            weld.Parent = car
        end
    end
    
    -- Lights
    local headlight1 = Instance.new("Part")
    headlight1.Name = "Headlight_L"
    headlight1.Size = Vector3.new(0.3, 0.5, 0.5)
    headlight1.Position = spawnPosition + Vector3.new(4, 0.5, -1.5)
    headlight1.BrickColor = BrickColor.new("White")
    headlight1.Material = Enum.Material.Neon
    headlight1.Parent = car
    
    local light1 = Instance.new("SpotLight")
    light1.Range = 30
    light1.Angle = 45
    light1.Brightness = 5
    light1.Face = Enum.NormalId.Front
    light1.Parent = headlight1
    
    local headlightWeld = Instance.new("WeldConstraint")
    headlightWeld.Part0 = body
    headlightWeld.Part1 = headlight1
    headlightWeld.Parent = car
    
    -- Exhaust Particles
    local exhaustPart = Instance.new("Part")
    exhaustPart.Name = "Exhaust"
    exhaustPart.Size = Vector3.new(0.5, 0.5, 0.5)
    exhaustPart.Position = spawnPosition + Vector3.new(-4, 0, 0)
    exhaustPart.Transparency = 1
    exhaustPart.CanCollide = false
    exhaustPart.Parent = car
    
    local exhaustParticle = Instance.new("ParticleEmitter")
    exhaustParticle.Color = ColorSequence.new(Color3.fromRGB(180, 180, 180))
    exhaustParticle.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0),
        NumberSequenceKeypoint.new(1, 1)
    })
    exhaustParticle.LightEmission = 0
    exhaustParticle.Size = NumberSequence.new(0.3)
    exhaustParticle.Rate = 20
    exhaustParticle.Speed = NumberRange.new(5, 10)
    exhaustParticle.Parent = exhaustPart
    
    local exhaustWeld = Instance.new("WeldConstraint")
    exhaustWeld.Part0 = body
    exhaustWeld.Part1 = exhaustPart
    exhaustWeld.Parent = car
    
    -- เพิ่ม Attributes
    car:SetAttribute("CarType", carData.id)
    car:SetAttribute("MaxSpeed", carData.maxSpeed or 100)
    car:SetAttribute("Acceleration", carData.acceleration or 50)
    
    car.Parent = workspace.Cars or workspace
    print("สร้างรถ " .. carData.name .. " เสร็จแล้ว")
    return car
end

-- ข้อมูลรถ
local carTypes = {
    {
        id = "speedster",
        name = "Speedster",
        maxSpeed = 150,
        torque = 200,
        turnSpeed = 1.2,
        acceleration = 80,
        color = BrickColor.new("Bright red"),
        size = Vector3.new(8, 2, 4)
    },
    {
        id = "muscle",
        name = "Muscle Car",
        maxSpeed = 120,
        torque = 300,
        turnSpeed = 0.8,
        acceleration = 100,
        color = BrickColor.new("Bright blue"),
        size = Vector3.new(9, 2.5, 4.5)
    },
    {
        id = "formula",
        name = "Formula 1",
        maxSpeed = 200,
        torque = 250,
        turnSpeed = 2.0,
        acceleration = 120,
        color = BrickColor.new("Bright yellow"),
        size = Vector3.new(10, 1.5, 3.5)
    }
}
```

### 66.2.2 CarController LocalScript

```lua
-- StarterPlayerScripts/CarController.lua
-- Client-side car controller

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

-- ตรวจสอบว่าผู้เล่นอยู่ในรถ
local currentSeat = nil
local currentCar = nil

-- Camera settings
local cameraOffset = Vector3.new(0, 8, 20)  -- กล้องอยู่ด้านหลัง
local cameraLerpSpeed = 0.1

-- ตรวจจับการขึ้นรถ
local function onSeated(isSeated, seat)
    if isSeated and seat:IsA("VehicleSeat") then
        currentSeat = seat
        currentCar = seat.Parent
        print("เข้านั่งรถ: " .. currentCar.Name)
        
        -- เปลี่ยนกล้องเป็น car follow
        camera.CameraType = Enum.CameraType.Scriptable
        
    elseif not isSeated then
        currentSeat = nil
        currentCar = nil
        camera.CameraType = Enum.CameraType.Custom
    end
end

-- รอ Character
player.CharacterAdded:Connect(function(character)
    local humanoid = character:WaitForChild("Humanoid")
    humanoid.Seated:Connect(onSeated)
end)

if player.Character then
    local humanoid = player.Character:FindFirstChild("Humanoid")
    if humanoid then
        humanoid.Seated:Connect(onSeated)
    end
end

-- อัพเดทกล้อง
local cameraTargetPos = Vector3.new(0, 0, 0)
local cameraTargetFocus = Vector3.new(0, 0, 0)

RunService.RenderStepped:Connect(function(dt)
    if not currentCar or not currentCar:FindFirstChild("Body") then return end
    
    local carBody = currentCar.Body
    local carCFrame = carBody.CFrame
    
    -- คำนวณตำแหน่งกล้องด้านหลังรถ
    local targetPos = carCFrame * CFrame.new(-cameraOffset.Z, cameraOffset.Y, 0)
    local targetFocus = carCFrame.Position + carCFrame.LookVector * 20
    
    -- Smooth camera movement
    cameraTargetPos = cameraTargetPos:Lerp(targetPos.Position, cameraLerpSpeed)
    cameraTargetFocus = cameraTargetFocus:Lerp(targetFocus, cameraLerpSpeed)
    
    camera.CFrame = CFrame.new(cameraTargetPos, cameraTargetFocus)
    
    -- แสดงความเร็ว (Speedometer)
    local velocity = carBody.Velocity
    local speed = math.floor(velocity.Magnitude * 3.6)  -- แปลงเป็น km/h
    
    -- อัพเดท HUD speedometer
    local playerGui = player.PlayerGui
    local raceHUD = playerGui:FindFirstChild("RaceHUD")
    if raceHUD then
        local speedLabel = raceHUD:FindFirstChild("SpeedLabel", true)
        if speedLabel then
            speedLabel.Text = speed .. " km/h"
        end
    end
end)

-- Drift System
local isDrifting = false
local driftStartTime = 0
local driftPoints = 0

-- ตรวจจับ drift (กดเบรก + หัน)
local function updateDrift()
    if not currentCar then return end
    
    local carBody = currentCar:FindFirstChild("Body")
    if not carBody then return end
    
    local velocity = carBody.Velocity
    local speed = velocity.Magnitude
    
    if speed < 20 then 
        if isDrifting then
            -- หยุด drift
            isDrifting = false
            -- บวก drift points
            local raceHUD = player.PlayerGui:FindFirstChild("RaceHUD")
            if raceHUD then
                local driftLabel = raceHUD:FindFirstChild("DriftLabel", true)
                if driftLabel then
                    driftLabel.Text = ""
                end
            end
        end
        return 
    end
    
    -- ตรวจสอบว่ากำลัง drift
    local forwardVelocity = carBody.CFrame.LookVector:Dot(velocity.Unit)
    local sideVelocity = carBody.CFrame.RightVector:Dot(velocity)
    
    if math.abs(sideVelocity) > 5 and UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then
        if not isDrifting then
            isDrifting = true
            driftStartTime = tick()
            driftPoints = 0
        end
        
        -- สะสม drift points
        local driftDuration = tick() - driftStartTime
        driftPoints = math.floor(driftDuration * 100 * (math.abs(sideVelocity) / 10))
        
        -- แสดง drift meter
        local raceHUD = player.PlayerGui:FindFirstChild("RaceHUD")
        if raceHUD then
            local driftLabel = raceHUD:FindFirstChild("DriftLabel", true)
            if driftLabel then
                driftLabel.Text = "DRIFT! +" .. driftPoints
                driftLabel.TextColor3 = Color3.fromHSV(driftDuration / 3, 1, 1)
            end
        end
    end
end

RunService.RenderStepped:Connect(updateDrift)
```

---

## 66.3 ระบบ Lap Tracking

```lua
-- ServerScriptService/LapTracker.lua
-- ระบบนับรอบ

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local Remotes = ReplicatedStorage:WaitForChild("Remotes")
local lapUpdateEvent = Instance.new("RemoteEvent")
lapUpdateEvent.Name = "LapUpdate"
lapUpdateEvent.Parent = Remotes

-- ข้อมูลการแข่ง
local raceData = {}
local TOTAL_LAPS = 3
local CHECKPOINT_COUNT = 5  -- จำนวน checkpoint ในแต่ละรอบ

-- เริ่มเก็บข้อมูลผู้เล่น
local function initPlayerRace(player)
    raceData[player.UserId] = {
        currentLap = 1,
        checkpointsHit = {},  -- รายการ checkpoint ที่ผ่านในรอบนี้
        lapTimes = {},
        lapStartTime = tick(),
        totalTime = 0,
        finished = false,
        position = 0  -- อันดับ
    }
end

-- ตรวจสอบ checkpoint
local function hitCheckpoint(player, checkpointNumber)
    local data = raceData[player.UserId]
    if not data or data.finished then return end
    
    -- ตรวจสอบว่าผ่าน checkpoint ตามลำดับ
    local expectedCheckpoint = #data.checkpointsHit + 1
    
    if checkpointNumber == expectedCheckpoint then
        table.insert(data.checkpointsHit, checkpointNumber)
        
        print(player.Name .. " ผ่าน Checkpoint " .. checkpointNumber)
        
        -- ตรวจสอบครบรอบ
        if #data.checkpointsHit >= CHECKPOINT_COUNT then
            completeLap(player)
        end
    end
end

-- จบรอบ
local function completeLap(player)
    local data = raceData[player.UserId]
    if not data then return end
    
    local lapTime = tick() - data.lapStartTime
    table.insert(data.lapTimes, lapTime)
    
    print(player.Name .. " จบ Lap " .. data.currentLap .. " เวลา: " .. string.format("%.2f", lapTime) .. "s")
    
    -- รีเซ็ต checkpoints
    data.checkpointsHit = {}
    data.lapStartTime = tick()
    
    if data.currentLap >= TOTAL_LAPS then
        -- จบการแข่ง!
        finishRace(player)
    else
        data.currentLap = data.currentLap + 1
        
        -- แจ้ง Client
        lapUpdateEvent:FireClient(player, {
            type = "lapComplete",
            lap = data.currentLap,
            lapTime = lapTime,
            bestLap = math.min(table.unpack(data.lapTimes))
        })
    end
end

-- จบการแข่ง
local function finishRace(player)
    local data = raceData[player.UserId]
    if not data then return end
    
    data.finished = true
    data.totalTime = tick() - data.lapStartTime  -- เวลารวม
    
    -- คำนวณอันดับ
    local finishedPlayers = {}
    for userId, playerData in pairs(raceData) do
        if playerData.finished then
            table.insert(finishedPlayers, {userId = userId, time = playerData.totalTime})
        end
    end
    
    table.sort(finishedPlayers, function(a, b) return a.time < b.time end)
    
    for position, entry in ipairs(finishedPlayers) do
        local p = Players:GetPlayerByUserId(entry.userId)
        if p then
            raceData[entry.userId].position = position
            
            lapUpdateEvent:FireClient(p, {
                type = "raceComplete",
                position = position,
                totalTime = entry.time,
                lapTimes = raceData[entry.userId].lapTimes
            })
        end
    end
    
    print(player.Name .. " จบการแข่ง! อันดับ " .. data.position)
end

-- ตั้ง Checkpoints ใน Workspace
local function setupCheckpoints()
    local checkpointFolder = workspace:FindFirstChild("Checkpoints") or Instance.new("Folder")
    checkpointFolder.Name = "Checkpoints"
    checkpointFolder.Parent = workspace
    
    -- (checkpoint จริงๆ ควรวางใน workspace editor)
    -- นี่เป็นตัวอย่างการ detect
    
    for _, checkpoint in ipairs(checkpointFolder:GetChildren()) do
        local cpNumber = tonumber(checkpoint.Name:match("CP(%d+)"))
        if cpNumber then
            checkpoint.Touched:Connect(function(hit)
                -- ตรวจสอบว่าเป็นรถ
                local car = hit:FindFirstAncestorWhichIsA("Model")
                if not car then return end
                
                -- หาผู้เล่นที่อยู่ในรถ
                local seat = car:FindFirstChildWhichIsA("VehicleSeat")
                if seat and seat.Occupant then
                    local character = seat.Occupant.Parent
                    local player = Players:GetPlayerFromCharacter(character)
                    if player then
                        hitCheckpoint(player, cpNumber)
                    end
                end
            end)
        end
    end
end

-- เริ่มต้น
Players.PlayerAdded:Connect(initPlayerRace)
setupCheckpoints()
```

---

## 66.4 Race Manager

```lua
-- ServerScriptService/RaceManager.lua
-- จัดการการแข่งทั้งหมด

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local Remotes = ReplicatedStorage:WaitForChild("Remotes")
local raceStateEvent = Instance.new("RemoteEvent")
raceStateEvent.Name = "RaceState"
raceStateEvent.Parent = Remotes

-- States: waiting, countdown, racing, finished
local gameState = "waiting"
local MIN_PLAYERS = 2
local COUNTDOWN_TIME = 10
local RACE_TIMEOUT = 300  -- 5 นาที

-- ตำแหน่ง Spawn ของรถ
local spawnPositions = {
    Vector3.new(0, 5, -20),
    Vector3.new(5, 5, -20),
    Vector3.new(-5, 5, -20),
    Vector3.new(10, 5, -20),
    Vector3.new(-10, 5, -20),
    Vector3.new(15, 5, -20),
    Vector3.new(-15, 5, -20),
    Vector3.new(20, 5, -20),
}

-- Car templates
local carTemplates = {
    "speedster", "muscle", "formula"
}

-- แจก car ให้ผู้เล่น
local playerCars = {}

local function giveCar(player, spawnIndex)
    local spawnPos = spawnPositions[spawnIndex] or Vector3.new(0, 5, 0)
    
    -- เลือก car แบบ random
    local carTypeId = carTemplates[math.random(#carTemplates)]
    local carTypeData = {
        id = carTypeId,
        name = carTypeId .. "_" .. player.Name,
        maxSpeed = 120,
        torque = 200,
        turnSpeed = 1.5,
        color = BrickColor.Random()
    }
    
    -- สร้างรถ
    local carFolder = workspace:FindFirstChild("Cars") or Instance.new("Folder")
    carFolder.Name = "Cars"
    carFolder.Parent = workspace
    
    -- Clone car template
    local carTemplate = game.ReplicatedStorage:FindFirstChild("Car_Template")
    if carTemplate then
        local car = carTemplate:Clone()
        car.Name = "Car_" .. player.UserId
        car:SetAttribute("OwnerId", player.UserId)
        
        if car.PrimaryPart then
            car:SetPrimaryPartCFrame(CFrame.new(spawnPos))
        end
        
        -- เปลี่ยนสีรถ
        for _, part in ipairs(car:GetDescendants()) do
            if part:IsA("BasePart") and part.Name == "Body" then
                part.BrickColor = BrickColor.Random()
            end
        end
        
        car.Parent = carFolder
        playerCars[player.UserId] = car
        
        -- Teleport ผู้เล่นไปที่รถ
        local seat = car:FindFirstChildWhichIsA("VehicleSeat")
        if seat and player.Character then
            local rootPart = player.Character:FindFirstChild("HumanoidRootPart")
            if rootPart then
                task.wait(0.1)
                seat:Sit(player.Character:FindFirstChildWhichIsA("Humanoid"))
            end
        end
        
        return car
    end
    
    return nil
end

-- เริ่มเกม
local function startRace()
    if gameState ~= "waiting" then return end
    
    local players = Players:GetPlayers()
    if #players < MIN_PLAYERS then
        print("รอผู้เล่นเพิ่ม... (" .. #players .. "/" .. MIN_PLAYERS .. ")")
        return
    end
    
    -- Countdown
    gameState = "countdown"
    raceStateEvent:FireAllClients({state = "countdown", time = COUNTDOWN_TIME})
    
    for i = COUNTDOWN_TIME, 1, -1 do
        raceStateEvent:FireAllClients({state = "countdown_tick", remaining = i})
        task.wait(1)
    end
    
    -- เริ่มแข่ง
    gameState = "racing"
    
    -- แจกรถ
    for i, player in ipairs(Players:GetPlayers()) do
        giveCar(player, i)
    end
    
    raceStateEvent:FireAllClients({state = "race_start"})
    print("การแข่งขันเริ่มแล้ว!")
    
    -- Timeout
    task.delay(RACE_TIMEOUT, function()
        if gameState == "racing" then
            endRace()
        end
    end)
end

-- จบเกม
local function endRace()
    gameState = "finished"
    raceStateEvent:FireAllClients({state = "race_end"})
    
    -- ลบรถ
    for userId, car in pairs(playerCars) do
        if car.Parent then
            car:Destroy()
        end
    end
    playerCars = {}
    
    -- รอ 10 วินาทีแล้วเริ่มใหม่
    task.wait(10)
    gameState = "waiting"
    
    -- ตรวจสอบผู้เล่นและเริ่มใหม่
    if #Players:GetPlayers() >= MIN_PLAYERS then
        startRace()
    end
end

-- เริ่มต้น
task.spawn(function()
    while true do
        task.wait(5)
        if gameState == "waiting" and #Players:GetPlayers() >= MIN_PLAYERS then
            startRace()
        end
    end
end)

Players.PlayerRemoving:Connect(function(player)
    if playerCars[player.UserId] then
        playerCars[player.UserId]:Destroy()
        playerCars[player.UserId] = nil
    end
end)
```

---

## 66.5 Race HUD

```lua
-- StarterGui/RaceHUD/LocalScript

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local raceHUD = script.Parent

-- สร้าง Speedometer
local speedFrame = Instance.new("Frame")
speedFrame.Name = "SpeedFrame"
speedFrame.Size = UDim2.new(0, 200, 0, 80)
speedFrame.Position = UDim2.new(1, -210, 1, -100)
speedFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
speedFrame.BackgroundTransparency = 0.5
speedFrame.BorderSizePixel = 0
speedFrame.Parent = raceHUD

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 10)
corner.Parent = speedFrame

local speedLabel = Instance.new("TextLabel")
speedLabel.Name = "SpeedLabel"
speedLabel.Size = UDim2.new(1, 0, 0.6, 0)
speedLabel.BackgroundTransparency = 1
speedLabel.Text = "0 km/h"
speedLabel.TextColor3 = Color3.new(1, 1, 1)
speedLabel.TextScaled = true
speedLabel.Font = Enum.Font.GothamBold
speedLabel.Parent = speedFrame

-- Lap Counter
local lapFrame = Instance.new("Frame")
lapFrame.Name = "LapFrame"
lapFrame.Size = UDim2.new(0, 200, 0, 60)
lapFrame.Position = UDim2.new(0.5, -100, 0, 10)
lapFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
lapFrame.BackgroundTransparency = 0.4
lapFrame.BorderSizePixel = 0
lapFrame.Parent = raceHUD

local corner2 = Instance.new("UICorner")
corner2.CornerRadius = UDim.new(0, 10)
corner2.Parent = lapFrame

local lapLabel = Instance.new("TextLabel")
lapLabel.Name = "LapLabel"
lapLabel.Size = UDim2.new(1, 0, 1, 0)
lapLabel.BackgroundTransparency = 1
lapLabel.Text = "LAP 1/3"
lapLabel.TextColor3 = Color3.new(1, 1, 1)
lapLabel.TextScaled = true
lapLabel.Font = Enum.Font.GothamBold
lapLabel.Parent = lapFrame

-- Position Display
local posFrame = Instance.new("Frame")
posFrame.Name = "PositionFrame"
posFrame.Size = UDim2.new(0, 150, 0, 60)
posFrame.Position = UDim2.new(1, -160, 0, 10)
posFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 80)
posFrame.BackgroundTransparency = 0.3
posFrame.BorderSizePixel = 0
posFrame.Parent = raceHUD

local corner3 = Instance.new("UICorner")
corner3.CornerRadius = UDim.new(0, 10)
corner3.Parent = posFrame

local posLabel = Instance.new("TextLabel")
posLabel.Name = "PositionLabel"
posLabel.Size = UDim2.new(1, 0, 1, 0)
posLabel.BackgroundTransparency = 1
posLabel.Text = "P1"
posLabel.TextColor3 = Color3.fromRGB(255, 215, 0)
posLabel.TextScaled = true
posLabel.Font = Enum.Font.GothamBold
posLabel.Parent = posFrame

-- Drift Indicator
local driftLabel = Instance.new("TextLabel")
driftLabel.Name = "DriftLabel"
driftLabel.Size = UDim2.new(0, 300, 0, 60)
driftLabel.Position = UDim2.new(0.5, -150, 0.7, 0)
driftLabel.BackgroundTransparency = 1
driftLabel.Text = ""
driftLabel.TextColor3 = Color3.fromRGB(255, 150, 0)
driftLabel.TextScaled = true
driftLabel.Font = Enum.Font.GothamBold
driftLabel.ZIndex = 5
driftLabel.Parent = raceHUD

-- Countdown Display
local countdownLabel = Instance.new("TextLabel")
countdownLabel.Name = "CountdownLabel"
countdownLabel.Size = UDim2.new(0, 200, 0, 100)
countdownLabel.Position = UDim2.new(0.5, -100, 0.4, 0)
countdownLabel.BackgroundTransparency = 1
countdownLabel.Text = ""
countdownLabel.TextColor3 = Color3.new(1, 1, 0)
countdownLabel.TextScaled = true
countdownLabel.Font = Enum.Font.GothamBold
countdownLabel.ZIndex = 10
countdownLabel.Visible = false
countdownLabel.Parent = raceHUD

-- รับ Race State Updates
local Remotes = ReplicatedStorage:WaitForChild("Remotes")
local raceStateEvent = Remotes:WaitForChild("RaceState")
local lapUpdateEvent = Remotes:WaitForChild("LapUpdate")

raceStateEvent.OnClientEvent:Connect(function(data)
    if data.state == "countdown" then
        countdownLabel.Visible = true
        countdownLabel.Text = "เตรียมพร้อม!"
        
    elseif data.state == "countdown_tick" then
        countdownLabel.Text = tostring(data.remaining)
        -- Animate
        countdownLabel.TextColor3 = data.remaining <= 3 
            and Color3.fromRGB(255, 0, 0) 
            or Color3.fromRGB(255, 255, 0)
        
        local tween = TweenService:Create(countdownLabel, TweenInfo.new(0.5), {
            TextTransparency = 0
        })
        tween:Play()
        
    elseif data.state == "race_start" then
        countdownLabel.Text = "GO!"
        countdownLabel.TextColor3 = Color3.fromRGB(0, 255, 0)
        task.wait(1)
        countdownLabel.Visible = false
        
    elseif data.state == "race_end" then
        -- แสดงผลสิ้นสุด
    end
end)

lapUpdateEvent.OnClientEvent:Connect(function(data)
    if data.type == "lapComplete" then
        lapLabel.Text = "LAP " .. data.lap .. "/3"
        
        -- แสดงเวลารอบ
        countdownLabel.Visible = true
        countdownLabel.Text = string.format("รอบเสร็จ! %.2fs", data.lapTime)
        task.wait(2)
        countdownLabel.Visible = false
        
    elseif data.type == "raceComplete" then
        local posText = {"🥇 1st!", "🥈 2nd!", "🥉 3rd!"}
        countdownLabel.Visible = true
        countdownLabel.Text = posText[data.position] or "#" .. data.position
        countdownLabel.TextColor3 = data.position == 1 
            and Color3.fromRGB(255, 215, 0)
            or Color3.new(1, 1, 1)
    end
end)
```

---

## 66.6 ข้อผิดพลาดที่พบบ่อย

```lua
-- ❌ ผิด: ใช้ BodyVelocity กับรถ (ล้าสมัย)
local bv = Instance.new("BodyVelocity")
bv.Velocity = Vector3.new(0, 0, -50)

-- ✓ ถูก: ใช้ VehicleSeat ที่ built-in หรือ LinearVelocity (ใหม่กว่า)
-- VehicleSeat.MaxSpeed กำหนดความเร็วสูงสุด
-- VehicleSeat.Torque กำหนดแรงบิด
```

---

## 66.7 แบบฝึกหัด

### แบบฝึกหัดที่ 1: เพิ่มระบบ Power-up
วาง Power-up บนสนามแข่ง:
- Speed Boost: เพิ่มความเร็ว 10 วินาที
- Shield: ป้องกันการชน 1 ครั้ง
- EMP: ทำให้รถคันอื่นหยุดชั่วคราว

### แบบฝึกหัดที่ 2: สร้าง Track ใหม่
ออกแบบ Track ที่มี:
- ส่วนตรง (Speed sections)
- โค้งแหลม (Tight corners)
- Jump ramp
- Tunnel

### แบบฝึกหัดที่ 3: เพิ่มระบบ Garage
ให้ผู้เล่นสามารถ:
- เลือกสีรถ
- เลือกประเภทรถ
- Upgrade Performance (Speed, Handling)

---

## สรุป

ในบทนี้เราได้สร้าง Racing Game ที่มี:
- ระบบรถพร้อม VehicleSeat
- Camera follow
- Drift system
- Lap tracking
- Race HUD

ในบทถัดไปเราจะสร้าง Shooter Game!
