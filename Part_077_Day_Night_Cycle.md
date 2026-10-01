# Part 77: Day/Night Cycle (ระบบกลางวัน/กลางคืน)

## บทนำ

Day/Night Cycle ทำให้เกมมีชีวิตชีวาและสมจริงมากขึ้น ในบทนี้เราจะสร้างระบบ Cycle ที่สมบูรณ์ รวมถึงการเปลี่ยน Lighting, Sky, Atmosphere และ Gameplay Effects ที่เชื่อมกับเวลาของวัน

---

## 1. โครงสร้างระบบ Day/Night

### 1.1 TimeManager

```lua
-- TimeManager.lua (Server Script)
-- ระบบจัดการเวลาในเกม

local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")

local TimeManager = {}
TimeManager.__index = TimeManager

-- ค่าคงที่ของเวลา
local TIME_CONFIG = {
    cycleLength = 600,    -- ความยาว 1 วัน (วินาที Real-time) = 10 นาที
    startTime = 6,        -- เวลาเริ่มต้น (6 = 6:00 เช้า)
    timeScale = 1,        -- ความเร็วของเวลา (1 = ปกติ)
}

-- ช่วงเวลา (0-24)
local TIME_PERIODS = {
    {name = "midnight", start = 0, finish = 4},
    {name = "dawn", start = 4, finish = 6},
    {name = "morning", start = 6, finish = 10},
    {name = "midday", start = 10, finish = 14},
    {name = "afternoon", start = 14, finish = 17},
    {name = "evening", start = 17, finish = 20},
    {name = "night", start = 20, finish = 24},
}

function TimeManager.new()
    local self = setmetatable({}, TimeManager)
    
    self.currentTime = TIME_CONFIG.startTime  -- ชั่วโมง (0-24)
    self.timeScale = TIME_CONFIG.timeScale
    self.cycleLength = TIME_CONFIG.cycleLength
    self.paused = false
    
    self.periodCallbacks = {}  -- Callbacks เมื่อเข้าช่วงเวลาใหม่
    self.lastPeriod = nil
    
    -- สร้าง RemoteEvent สำหรับ Sync
    local eventsFolder = Instance.new("Folder")
    eventsFolder.Name = "TimeEvents"
    eventsFolder.Parent = ReplicatedStorage
    
    local timeUpdate = Instance.new("RemoteEvent")
    timeUpdate.Name = "TimeUpdate"
    timeUpdate.Parent = eventsFolder
    
    self.timeUpdateEvent = timeUpdate
    
    return self
end

-- แปลงเวลา Game เป็น Real Time Progress
function TimeManager:getTimeProgress()
    return (self.currentTime % 24) / 24
end

-- เพิ่มเวลา
function TimeManager:advance(deltaTime)
    if self.paused then return end
    
    -- แปลง delta time เป็นชั่วโมง Game
    -- ใน 10 นาที Real = 24 ชั่วโมง Game
    local hoursPerSecond = 24 / self.cycleLength
    self.currentTime = (self.currentTime + deltaTime * hoursPerSecond * self.timeScale) % 24
    
    -- ตรวจสอบ Period Change
    local currentPeriod = self:getCurrentPeriod()
    if currentPeriod and currentPeriod.name ~= self.lastPeriod then
        self.lastPeriod = currentPeriod.name
        self:_notifyPeriodChange(currentPeriod)
    end
end

-- หาช่วงเวลาปัจจุบัน
function TimeManager:getCurrentPeriod()
    for _, period in ipairs(TIME_PERIODS) do
        if self.currentTime >= period.start and self.currentTime < period.finish then
            return period
        end
    end
    return TIME_PERIODS[1]  -- midnight
end

-- ลงทะเบียน Callback เมื่อเข้าช่วงเวลา
function TimeManager:onPeriodChange(periodName, callback)
    if not self.periodCallbacks[periodName] then
        self.periodCallbacks[periodName] = {}
    end
    table.insert(self.periodCallbacks[periodName], callback)
end

-- แจ้ง Period Change
function TimeManager:_notifyPeriodChange(period)
    print(string.format("[TIME] เข้าช่วงเวลา: %s (%.1f)", period.name, self.currentTime))
    
    if self.periodCallbacks[period.name] then
        for _, callback in ipairs(self.periodCallbacks[period.name]) do
            task.spawn(callback, period)
        end
    end
    
    if self.periodCallbacks["*"] then
        for _, callback in ipairs(self.periodCallbacks["*"]) do
            task.spawn(callback, period)
        end
    end
end

-- แปลงชั่วโมงเป็น String แสดงผล
function TimeManager:getFormattedTime()
    local hours = math.floor(self.currentTime)
    local minutes = math.floor((self.currentTime - hours) * 60)
    local ampm = hours < 12 and "AM" or "PM"
    local displayHours = hours % 12
    if displayHours == 0 then displayHours = 12 end
    return string.format("%d:%02d %s", displayHours, minutes, ampm)
end

-- เริ่มระบบ
function TimeManager:start()
    local lastTick = tick()
    
    RunService.Heartbeat:Connect(function()
        local now = tick()
        local dt = now - lastTick
        lastTick = now
        
        self:advance(dt)
        
        -- Sync ไปยัง Client ทุก 1 วินาที
        if math.floor(now) ~= math.floor(now - dt) then
            self.timeUpdateEvent:FireAllClients(self.currentTime)
        end
    end)
    
    print("Time Manager started")
end

-- ตั้งเวลาโดยตรง
function TimeManager:setTime(hour)
    self.currentTime = math.clamp(hour, 0, 24)
    -- Sync ทันที
    self.timeUpdateEvent:FireAllClients(self.currentTime)
end

return TimeManager
```

---

## 2. Lighting System

### 2.1 Dynamic Lighting

```lua
-- LightingController.lua (Local Script)
-- ควบคุม Lighting ตามเวลา

local Lighting = game:GetService("Lighting")
local TweenService = game:GetService("TweenService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- Lighting Presets ตามเวลา
local LIGHTING_PRESETS = {
    midnight = {
        Ambient = Color3.fromRGB(10, 10, 20),
        OutdoorAmbient = Color3.fromRGB(15, 15, 30),
        Brightness = 0.3,
        ClockTime = 0,
        FogEnd = 800,
        FogColor = Color3.fromRGB(10, 10, 20),
        ShadowSoftness = 0.1,
    },
    
    dawn = {
        Ambient = Color3.fromRGB(60, 40, 50),
        OutdoorAmbient = Color3.fromRGB(100, 80, 60),
        Brightness = 1.5,
        ClockTime = 5,
        FogEnd = 2000,
        FogColor = Color3.fromRGB(200, 150, 100),
        ShadowSoftness = 0.2,
    },
    
    morning = {
        Ambient = Color3.fromRGB(90, 90, 110),
        OutdoorAmbient = Color3.fromRGB(150, 150, 170),
        Brightness = 3,
        ClockTime = 8,
        FogEnd = 5000,
        FogColor = Color3.fromRGB(200, 210, 230),
        ShadowSoftness = 0.5,
    },
    
    midday = {
        Ambient = Color3.fromRGB(120, 120, 130),
        OutdoorAmbient = Color3.fromRGB(200, 200, 200),
        Brightness = 5,
        ClockTime = 12,
        FogEnd = 10000,
        FogColor = Color3.fromRGB(230, 240, 255),
        ShadowSoftness = 0.8,
    },
    
    afternoon = {
        Ambient = Color3.fromRGB(110, 100, 90),
        OutdoorAmbient = Color3.fromRGB(180, 170, 140),
        Brightness = 4,
        ClockTime = 15,
        FogEnd = 6000,
        FogColor = Color3.fromRGB(220, 210, 190),
        ShadowSoftness = 0.6,
    },
    
    evening = {
        Ambient = Color3.fromRGB(80, 50, 40),
        OutdoorAmbient = Color3.fromRGB(140, 90, 60),
        Brightness = 2,
        ClockTime = 18,
        FogEnd = 3000,
        FogColor = Color3.fromRGB(200, 120, 80),
        ShadowSoftness = 0.3,
    },
    
    night = {
        Ambient = Color3.fromRGB(20, 20, 40),
        OutdoorAmbient = Color3.fromRGB(30, 30, 60),
        Brightness = 0.5,
        ClockTime = 22,
        FogEnd = 1000,
        FogColor = Color3.fromRGB(20, 20, 40),
        ShadowSoftness = 0.1,
    },
}

-- Atmosphere Presets
local ATMOSPHERE_PRESETS = {
    midday = {
        Density = 0.3,
        Offset = 0.25,
        Color = Color3.fromRGB(199, 199, 199),
        Decay = Color3.fromRGB(106, 112, 125),
        Glare = 0,
        Haze = 0,
    },
    
    dawn = {
        Density = 0.45,
        Offset = 0.1,
        Color = Color3.fromRGB(255, 160, 100),
        Decay = Color3.fromRGB(200, 100, 50),
        Glare = 0.3,
        Haze = 2,
    },
    
    night = {
        Density = 0.8,
        Offset = 0,
        Color = Color3.fromRGB(20, 30, 60),
        Decay = Color3.fromRGB(10, 15, 40),
        Glare = 0,
        Haze = 0,
    },
}

-- หา Atmosphere
local atmosphere = Lighting:FindFirstChildOfClass("Atmosphere")
if not atmosphere then
    atmosphere = Instance.new("Atmosphere")
    atmosphere.Parent = Lighting
end

-- Sky
local sky = Lighting:FindFirstChildOfClass("Sky")

-- ฟังก์ชัน Interpolate Color
local function lerpColor(c1, c2, t)
    return Color3.new(
        c1.R + (c2.R - c1.R) * t,
        c1.G + (c2.G - c1.G) * t,
        c1.B + (c2.B - c1.B) * t
    )
end

-- ฟังก์ชัน Apply Lighting Preset พร้อม Transition
local currentPresetName = "morning"
local isTransitioning = false

local function applyPreset(presetName, transitionTime)
    local preset = LIGHTING_PRESETS[presetName]
    if not preset then return end
    
    transitionTime = transitionTime or 2
    
    local tweenInfo = TweenInfo.new(transitionTime, Enum.EasingStyle.Sine)
    
    -- Tween Lighting Properties
    local lightingTween = TweenService:Create(Lighting, tweenInfo, {
        Ambient = preset.Ambient,
        OutdoorAmbient = preset.OutdoorAmbient,
        Brightness = preset.Brightness,
        ClockTime = preset.ClockTime,
        FogEnd = preset.FogEnd,
        FogColor = preset.FogColor,
    })
    
    lightingTween:Play()
    
    -- Tween Atmosphere
    local atmoPreset = ATMOSPHERE_PRESETS[presetName] or ATMOSPHERE_PRESETS.midday
    
    local atmosphereTween = TweenService:Create(atmosphere, tweenInfo, {
        Density = atmoPreset.Density,
        Offset = atmoPreset.Offset,
        Color = atmoPreset.Color,
        Decay = atmoPreset.Decay,
        Glare = atmoPreset.Glare,
        Haze = atmoPreset.Haze,
    })
    
    atmosphereTween:Play()
    
    currentPresetName = presetName
end

-- อัพเดท Lighting ตาม Time ของ Server
local timeEvents = ReplicatedStorage:WaitForChild("TimeEvents")
local timeUpdate = timeEvents:WaitForChild("TimeUpdate")

-- Map Time Period ไปยัง Preset
local PERIOD_TO_PRESET = {
    midnight = "midnight",
    dawn = "dawn",
    morning = "morning",
    midday = "midday",
    afternoon = "afternoon",
    evening = "evening",
    night = "night",
}

-- Smooth Clock Time Update
timeUpdate.OnClientEvent:Connect(function(gameTime)
    -- อัพเดท Clock Time แบบ Smooth
    Lighting.ClockTime = gameTime
    
    -- หาช่วงเวลา
    local periodName
    if gameTime >= 0 and gameTime < 4 then
        periodName = "midnight"
    elseif gameTime >= 4 and gameTime < 6 then
        periodName = "dawn"
    elseif gameTime >= 6 and gameTime < 10 then
        periodName = "morning"
    elseif gameTime >= 10 and gameTime < 14 then
        periodName = "midday"
    elseif gameTime >= 14 and gameTime < 17 then
        periodName = "afternoon"
    elseif gameTime >= 17 and gameTime < 20 then
        periodName = "evening"
    else
        periodName = "night"
    end
    
    -- เปลี่ยน Preset ถ้าเข้าช่วงเวลาใหม่
    if periodName ~= currentPresetName then
        applyPreset(periodName, 5)  -- 5 วินาที transition
    end
end)

-- ใช้ Preset เริ่มต้น
applyPreset("morning", 0)
```

---

## 3. Sky System

### 3.1 Dynamic Sky

```lua
-- SkyController.lua (Local Script)
-- เปลี่ยน Sky ตามเวลา

local Lighting = game:GetService("Lighting")
local TweenService = game:GetService("TweenService")

-- Sky Asset IDs (เปลี่ยนเป็น Asset จริง)
local SKY_ASSETS = {
    day = {
        SkyboxBk = "rbxassetid://159454288",
        SkyboxDn = "rbxassetid://159454296",
        SkyboxFt = "rbxassetid://159454279",
        SkyboxLf = "rbxassetid://159454283",
        SkyboxRt = "rbxassetid://159454291",
        SkyboxUp = "rbxassetid://159454300",
    },
    night = {
        SkyboxBk = "rbxassetid://159454288",  -- ใช้ Night Sky
        SkyboxDn = "rbxassetid://159454296",
        SkyboxFt = "rbxassetid://159454279",
        SkyboxLf = "rbxassetid://159454283",
        SkyboxRt = "rbxassetid://159454291",
        SkyboxUp = "rbxassetid://159454300",
    }
}

-- Star Visibility ตามเวลา
local function updateStars(gameTime)
    local sky = Lighting:FindFirstChildOfClass("Sky")
    if not sky then return end
    
    -- ดาวเห็นได้ตอนกลางคืน
    if gameTime >= 20 or gameTime < 4 then
        sky.StarCount = 3000
    elseif gameTime >= 4 and gameTime < 6 then
        -- ค่อยๆ หายไปตอนรุ่งเช้า
        local t = (gameTime - 4) / 2
        sky.StarCount = math.floor(3000 * (1 - t))
    elseif gameTime >= 17 and gameTime < 20 then
        -- ค่อยๆ ปรากฏตอนพลบค่ำ
        local t = (gameTime - 17) / 3
        sky.StarCount = math.floor(3000 * t)
    else
        sky.StarCount = 0
    end
end

-- Sun/Moon Position
local function updateCelestialBodies(gameTime)
    -- Roblox Lighting.ClockTime จัดการ Sun/Moon โดยอัตโนมัติ
    -- แต่เราสามารถ adjust ได้ผ่าน Lighting.GeographicLatitude
    
    -- ปรับ Latitude ตามฤดูกาล (ถ้าต้องการ)
    -- Lighting.GeographicLatitude = 41.7  -- New York latitude
end

-- เรียกทุกครั้งที่เวลาอัพเดท
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local timeEvents = ReplicatedStorage:WaitForChild("TimeEvents")
local timeUpdate = timeEvents:WaitForChild("TimeUpdate")

timeUpdate.OnClientEvent:Connect(function(gameTime)
    updateStars(gameTime)
    updateCelestialBodies(gameTime)
end)
```

---

## 4. Gameplay Effects ตามเวลา

### 4.1 Time-Based Gameplay

```lua
-- TimeGameplay.lua (Server Script)
-- ผลกระทบของเวลาต่อ Gameplay

local Players = game:GetService("Players")

local TimeGameplay = {}
TimeGameplay.__index = TimeGameplay

-- ผล Buff/Debuff ตามเวลา
local TIME_EFFECTS = {
    day = {
        walkSpeedMultiplier = 1.0,
        damageMultiplier = 1.0,
        visibilityRange = 100,
        moodBonus = 10,
    },
    night = {
        walkSpeedMultiplier = 0.9,    -- เดินช้าลงนิดหน่อย
        damageMultiplier = 1.2,       -- ศัตรูแรงขึ้น
        visibilityRange = 30,          -- มองไม่เห็นไกล
        moodBonus = -5,
    },
    dawn = {
        walkSpeedMultiplier = 1.05,
        damageMultiplier = 0.9,
        visibilityRange = 60,
        moodBonus = 20,  -- รุ่งเช้า Mood ดี
    },
}

-- สร้าง Lantern/Torch สำหรับ Night
local function createPlayerLight(character)
    local hrp = character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    
    -- ลบ Light เก่า
    local oldLight = hrp:FindFirstChild("PlayerLight")
    if oldLight then oldLight:Destroy() end
    
    local light = Instance.new("PointLight")
    light.Name = "PlayerLight"
    light.Brightness = 3
    light.Range = 20
    light.Color = Color3.fromRGB(255, 200, 100)
    light.Parent = hrp
    
    return light
end

-- ลบ Light ออกตอนกลางวัน
local function removePlayerLight(character)
    local hrp = character:FindFirstChild("HumanoidRootPart")
    if hrp then
        local light = hrp:FindFirstChild("PlayerLight")
        if light then light:Destroy() end
    end
end

-- ใช้ Effects กับ Player ทุกคน
function TimeGameplay:applyTimeEffects(periodName)
    local effects = TIME_EFFECTS[periodName] or TIME_EFFECTS.day
    local isNight = periodName == "night" or periodName == "midnight"
    
    for _, player in ipairs(Players:GetPlayers()) do
        local char = player.Character
        if not char then continue end
        
        local humanoid = char:FindFirstChild("Humanoid")
        if humanoid then
            -- ใช้ Speed Multiplier
            humanoid.WalkSpeed = 16 * effects.walkSpeedMultiplier
        end
        
        -- จัดการ Player Light
        if isNight then
            createPlayerLight(char)
        else
            removePlayerLight(char)
        end
    end
    
    print(string.format("[GAMEPLAY] เปลี่ยนเป็น: %s", periodName))
end

return TimeGameplay
```

---

## 5. Time UI

### 5.1 Clock Display

```lua
-- ClockUI.lua (Local Script)
-- แสดงนาฬิกาและเวลา

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- สร้าง Clock UI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "ClockUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = playerGui

local clockFrame = Instance.new("Frame")
clockFrame.Name = "Clock"
clockFrame.Size = UDim2.new(0, 150, 0, 60)
clockFrame.Position = UDim2.new(1, -160, 0, 10)
clockFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
clockFrame.BackgroundTransparency = 0.4
clockFrame.BorderSizePixel = 0
clockFrame.Parent = screenGui

local clockCorner = Instance.new("UICorner")
clockCorner.CornerRadius = UDim.new(0, 8)
clockCorner.Parent = clockFrame

-- ไอคอนเวลา
local timeIcon = Instance.new("TextLabel")
timeIcon.Name = "Icon"
timeIcon.Size = UDim2.new(0, 30, 1, 0)
timeIcon.BackgroundTransparency = 1
timeIcon.TextSize = 24
timeIcon.Font = Enum.Font.GothamBold
timeIcon.Parent = clockFrame

-- ข้อความเวลา
local timeLabel = Instance.new("TextLabel")
timeLabel.Name = "TimeLabel"
timeLabel.Size = UDim2.new(1, -35, 0.6, 0)
timeLabel.Position = UDim2.new(0, 30, 0, 5)
timeLabel.BackgroundTransparency = 1
timeLabel.Text = "12:00 PM"
timeLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
timeLabel.TextSize = 18
timeLabel.Font = Enum.Font.GothamBold
timeLabel.TextXAlignment = Enum.TextXAlignment.Left
timeLabel.Parent = clockFrame

-- ชื่อช่วงเวลา
local periodLabel = Instance.new("TextLabel")
periodLabel.Name = "Period"
periodLabel.Size = UDim2.new(1, -35, 0.35, 0)
periodLabel.Position = UDim2.new(0, 30, 0.62, 0)
periodLabel.BackgroundTransparency = 1
periodLabel.Text = "Morning"
periodLabel.TextColor3 = Color3.fromRGB(200, 200, 150)
periodLabel.TextSize = 12
periodLabel.Font = Enum.Font.Gotham
periodLabel.TextXAlignment = Enum.TextXAlignment.Left
periodLabel.Parent = clockFrame

-- Progress Bar (แสดงความคืบหน้าของวัน)
local dayProgress = Instance.new("Frame")
dayProgress.Name = "DayProgress"
dayProgress.Size = UDim2.new(1, -10, 0, 3)
dayProgress.Position = UDim2.new(0, 5, 1, -8)
dayProgress.BackgroundColor3 = Color3.fromRGB(50, 50, 70)
dayProgress.BorderSizePixel = 0
dayProgress.Parent = clockFrame

local progressFill = Instance.new("Frame")
progressFill.Name = "Fill"
progressFill.Size = UDim2.new(0.5, 0, 1, 0)
progressFill.BackgroundColor3 = Color3.fromRGB(255, 200, 50)
progressFill.BorderSizePixel = 0
progressFill.Parent = dayProgress

-- อัพเดท UI
local PERIOD_ICONS = {
    midnight = "🌑",
    dawn = "🌅",
    morning = "🌤️",
    midday = "☀️",
    afternoon = "⛅",
    evening = "🌆",
    night = "🌙",
}

local PERIOD_NAMES_TH = {
    midnight = "เที่ยงคืน",
    dawn = "รุ่งเช้า",
    morning = "เช้า",
    midday = "เที่ยงวัน",
    afternoon = "บ่าย",
    evening = "เย็น",
    night = "กลางคืน",
}

local function updateClock(gameTime)
    -- แปลงเวลา
    local hours = math.floor(gameTime)
    local minutes = math.floor((gameTime - hours) * 60)
    local ampm = hours < 12 and "AM" or "PM"
    local displayHours = hours % 12
    if displayHours == 0 then displayHours = 12 end
    
    timeLabel.Text = string.format("%d:%02d %s", displayHours, minutes, ampm)
    
    -- หาช่วงเวลา
    local periodName
    if gameTime >= 0 and gameTime < 4 then periodName = "midnight"
    elseif gameTime >= 4 and gameTime < 6 then periodName = "dawn"
    elseif gameTime >= 6 and gameTime < 10 then periodName = "morning"
    elseif gameTime >= 10 and gameTime < 14 then periodName = "midday"
    elseif gameTime >= 14 and gameTime < 17 then periodName = "afternoon"
    elseif gameTime >= 17 and gameTime < 20 then periodName = "evening"
    else periodName = "night"
    end
    
    timeIcon.Text = PERIOD_ICONS[periodName] or "🕐"
    periodLabel.Text = PERIOD_NAMES_TH[periodName] or ""
    
    -- อัพเดท Progress Bar
    local progress = gameTime / 24
    TweenService:Create(progressFill, TweenInfo.new(0.5), {
        Size = UDim2.new(progress, 0, 1, 0)
    }):Play()
    
    -- เปลี่ยนสี Progress ตามเวลา
    local progressColor
    if gameTime >= 6 and gameTime < 18 then
        progressColor = Color3.fromRGB(255, 200, 50)  -- เหลือง: กลางวัน
    else
        progressColor = Color3.fromRGB(100, 100, 200) -- น้ำเงิน: กลางคืน
    end
    
    TweenService:Create(progressFill, TweenInfo.new(0.5), {
        BackgroundColor3 = progressColor
    }):Play()
end

-- รับ Time Update จาก Server
local timeEvents = ReplicatedStorage:WaitForChild("TimeEvents")
local timeUpdate = timeEvents:WaitForChild("TimeUpdate")

timeUpdate.OnClientEvent:Connect(updateClock)
```

---

## 6. ข้อผิดพลาดที่พบบ่อย

### ❌ ข้อผิดพลาดที่ 1: ใช้ wait() แทน Heartbeat สำหรับ Time

```lua
-- ❌ แบบผิด: ใช้ task.wait - ไม่ smooth
task.spawn(function()
    while true do
        currentTime = currentTime + 0.01
        updateLighting()
        task.wait(0.1)  -- กระตุก
    end
end)

-- ✅ แบบถูก: ใช้ Heartbeat
RunService.Heartbeat:Connect(function(dt)
    currentTime = (currentTime + dt * hoursPerSecond) % 24
    -- Smooth มากกว่า
end)
```

### ❌ ข้อผิดพลาดที่ 2: ทำ Lighting Changes บน Client ทุกคนแยกกัน

```lua
-- ❌ แบบผิด: Server ทำ Lighting โดยตรง (ไม่ sync)
Lighting.ClockTime = currentTime  -- เปลี่ยนบน Server ไม่ส่งผล Client

-- ✅ แบบถูก: ส่ง Event ให้ Client ทำ
timeUpdateEvent:FireAllClients(currentTime)
-- Client จะอัพเดท Lighting ตัวเอง
```

### ❌ ข้อผิดพลาดที่ 3: Tween เร็วเกินไปทำให้กระตุก

```lua
-- ❌ แบบผิด: Tween ทุกครั้งที่ Time Update (บ่อยเกินไป)
timeUpdate.OnClientEvent:Connect(function(time)
    TweenService:Create(Lighting, TweenInfo.new(0.1), {...}):Play()  -- กระตุก!
end)

-- ✅ แบบถูก: Tween เฉพาะเมื่อเปลี่ยน Period
local lastPeriod = ""
timeUpdate.OnClientEvent:Connect(function(time)
    local newPeriod = getPeriod(time)
    if newPeriod ~= lastPeriod then
        lastPeriod = newPeriod
        TweenService:Create(Lighting, TweenInfo.new(5), {...}):Play()  -- Smooth transition
    end
    Lighting.ClockTime = time  -- อัพเดทตรงๆ โดยไม่ Tween
end)
```

---

## 7. แบบฝึกหัด

### แบบฝึกหัดที่ 1: Weather Integration
เชื่อม Day/Night กับ Weather:
- ฝนตกบ่อยช่วงเย็น
- หมอกหนาตอนรุ่งเช้า
- อากาศดีช่วงเที่ยง

### แบบฝึกหัดที่ 2: NPC Schedule
สร้าง NPC ที่มีตาราง:
- เช้า: ทำงาน
- เที่ยง: พัก/ขาย
- ค่ำ: กลับบ้าน
- คืน: นอนหลับ

### แบบฝึกหัดที่ 3: Day/Night Events
สร้าง Events พิเศษตามเวลา:
- กลางดึก: Mob พิเศษ Spawn
- รุ่งเช้า: Bonus EXP
- เที่ยง: Double Drops

---

## สรุป

Day/Night Cycle ที่ดีต้องมี:
1. **Server Authority**: Server ควบคุมเวลา Client แค่แสดงผล
2. **Smooth Transitions**: Tween แทน Instant Change
3. **Gameplay Integration**: เวลามีผลต่อการเล่นจริง
4. **Visual Polish**: Sky, Lighting, Atmosphere เปลี่ยนพร้อมกัน
5. **Sync**: ทุก Client เห็นเวลาเดียวกัน

ในส่วนถัดไป (Part 78) เราจะสร้าง Weather System ที่สมจริง ตั้งแต่ฝน หิมะ ไปจนถึงพายุ
