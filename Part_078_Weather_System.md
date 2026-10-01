# Part 78: Weather System (ระบบสภาพอากาศ)

## บทนำ

Weather System สมจริงช่วยสร้าง Immersion และอาจมีผลต่อ Gameplay ในบทนี้เราจะสร้างระบบสภาพอากาศที่ครอบคลุม รวมถึงฝน หิมะ พายุ หมอก และเชื่อมต่อกับ Day/Night Cycle

---

## 1. Weather Manager

### 1.1 Core Weather System

```lua
-- WeatherManager.lua (Server Script)
-- ระบบจัดการสภาพอากาศหลัก

local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local WeatherManager = {}
WeatherManager.__index = WeatherManager

-- ประเภทสภาพอากาศ
local WEATHER_TYPES = {
    CLEAR = "clear",
    CLOUDY = "cloudy",
    RAIN = "rain",
    HEAVY_RAIN = "heavy_rain",
    THUNDER = "thunder",
    SNOW = "snow",
    BLIZZARD = "blizzard",
    FOG = "fog",
    SANDSTORM = "sandstorm",
}

-- การตั้งค่าของแต่ละสภาพอากาศ
local WEATHER_CONFIG = {
    clear = {
        displayName = "แดดออก",
        icon = "☀️",
        fogDensity = 0,
        windSpeed = 5,
        temperature = 28,
        gameplayEffects = {
            visibility = 1.0,
            moveSpeed = 1.0,
        }
    },
    
    cloudy = {
        displayName = "มีเมฆ",
        icon = "☁️",
        fogDensity = 0.1,
        windSpeed = 10,
        temperature = 22,
        gameplayEffects = {
            visibility = 0.8,
            moveSpeed = 1.0,
        }
    },
    
    rain = {
        displayName = "ฝนตก",
        icon = "🌧️",
        fogDensity = 0.3,
        windSpeed = 15,
        temperature = 18,
        gameplayEffects = {
            visibility = 0.6,
            moveSpeed = 0.9,     -- เดินช้าลงบนพื้นเปียก
            fireDamageReduction = 0.5,  -- ไฟติดยาก
        }
    },
    
    heavy_rain = {
        displayName = "ฝนหนัก",
        icon = "⛈️",
        fogDensity = 0.5,
        windSpeed = 25,
        temperature = 15,
        gameplayEffects = {
            visibility = 0.4,
            moveSpeed = 0.8,
            fireDamageReduction = 0.8,
        }
    },
    
    thunder = {
        displayName = "พายุฝนฟ้าคะนอง",
        icon = "⛈️",
        fogDensity = 0.6,
        windSpeed = 35,
        temperature = 12,
        lightningFrequency = 5,  -- ฟ้าผ่าทุก N วินาที (โดยเฉลี่ย)
        gameplayEffects = {
            visibility = 0.3,
            moveSpeed = 0.75,
        }
    },
    
    snow = {
        displayName = "หิมะตก",
        icon = "🌨️",
        fogDensity = 0.2,
        windSpeed = 8,
        temperature = -2,
        gameplayEffects = {
            visibility = 0.7,
            moveSpeed = 0.85,    -- เดินลำบากในหิมะ
        }
    },
    
    blizzard = {
        displayName = "พายุหิมะ",
        icon = "❄️",
        fogDensity = 0.8,
        windSpeed = 50,
        temperature = -15,
        gameplayEffects = {
            visibility = 0.2,
            moveSpeed = 0.6,
        }
    },
    
    fog = {
        displayName = "หมอกหนา",
        icon = "🌫️",
        fogDensity = 0.9,
        windSpeed = 2,
        temperature = 10,
        gameplayEffects = {
            visibility = 0.15,
            moveSpeed = 0.95,
        }
    },
}

-- ลำดับของ Weather Transitions (สมเหตุสมผล)
local WEATHER_TRANSITIONS = {
    clear = {"cloudy", "clear", "clear"},          -- ส่วนมากยังแดดอยู่
    cloudy = {"clear", "rain", "cloudy", "fog"},
    rain = {"cloudy", "heavy_rain", "rain"},
    heavy_rain = {"rain", "thunder", "rain"},
    thunder = {"heavy_rain", "rain"},
    snow = {"cloudy", "blizzard", "snow"},
    blizzard = {"snow", "snow"},
    fog = {"clear", "cloudy", "fog"},
}

function WeatherManager.new()
    local self = setmetatable({}, WeatherManager)
    
    self.currentWeather = WEATHER_TYPES.CLEAR
    self.nextWeather = nil
    self.transitionProgress = 0
    self.weatherDuration = 0        -- วินาทีที่เหลือในสภาพอากาศนี้
    self.autoChange = true          -- เปลี่ยนอัตโนมัติ
    self.changeInterval = {min = 120, max = 300}  -- 2-5 นาที
    
    self.callbacks = {}
    
    -- สร้าง Events
    self:_createEvents()
    
    return self
end

-- สร้าง RemoteEvents
function WeatherManager:_createEvents()
    local weatherFolder = Instance.new("Folder")
    weatherFolder.Name = "WeatherEvents"
    weatherFolder.Parent = ReplicatedStorage
    
    local weatherChange = Instance.new("RemoteEvent")
    weatherChange.Name = "WeatherChange"
    weatherChange.Parent = weatherFolder
    
    local weatherUpdate = Instance.new("RemoteEvent")
    weatherUpdate.Name = "WeatherUpdate"
    weatherUpdate.Parent = weatherFolder
    
    self.weatherChangeEvent = weatherChange
    self.weatherUpdateEvent = weatherUpdate
end

-- เปลี่ยนสภาพอากาศ
function WeatherManager:setWeather(weatherType, transitionTime)
    transitionTime = transitionTime or 10  -- วินาที
    
    local config = WEATHER_CONFIG[weatherType]
    if not config then
        warn("Unknown weather type:", weatherType)
        return
    end
    
    local oldWeather = self.currentWeather
    self.currentWeather = weatherType
    
    -- แจ้ง Client
    self.weatherChangeEvent:FireAllClients(weatherType, oldWeather, transitionTime)
    
    -- เรียก Callbacks
    if self.callbacks[weatherType] then
        for _, cb in ipairs(self.callbacks[weatherType]) do
            task.spawn(cb, weatherType, oldWeather)
        end
    end
    
    if self.callbacks["*"] then
        for _, cb in ipairs(self.callbacks["*"]) do
            task.spawn(cb, weatherType, oldWeather)
        end
    end
    
    -- ใช้ Gameplay Effects
    self:_applyGameplayEffects(config.gameplayEffects)
    
    print(string.format("[WEATHER] เปลี่ยนเป็น: %s %s", 
        config.icon, config.displayName))
end

-- ใช้ Gameplay Effects
function WeatherManager:_applyGameplayEffects(effects)
    if not effects then return end
    
    for _, player in ipairs(Players:GetPlayers()) do
        local char = player.Character
        if not char then continue end
        
        local humanoid = char:FindFirstChild("Humanoid")
        if humanoid and effects.moveSpeed then
            humanoid.WalkSpeed = 16 * effects.moveSpeed
        end
    end
end

-- เลือกสภาพอากาศถัดไปอัตโนมัติ
function WeatherManager:_selectNextWeather()
    local transitions = WEATHER_TRANSITIONS[self.currentWeather]
    if not transitions or #transitions == 0 then
        return WEATHER_TYPES.CLEAR
    end
    
    return transitions[math.random(#transitions)]
end

-- เริ่ม Auto Change
function WeatherManager:startAutoChange()
    task.spawn(function()
        while true do
            -- รอก่อนเปลี่ยน
            local waitTime = math.random(
                self.changeInterval.min,
                self.changeInterval.max
            )
            task.wait(waitTime)
            
            if self.autoChange then
                local nextWeather = self:_selectNextWeather()
                self:setWeather(nextWeather)
            end
        end
    end)
    
    print("Weather Auto-Change started")
end

-- ลงทะเบียน Callback
function WeatherManager:onWeatherChange(weatherType, callback)
    if not self.callbacks[weatherType] then
        self.callbacks[weatherType] = {}
    end
    table.insert(self.callbacks[weatherType], callback)
end

return WeatherManager
```

---

## 2. Weather Visual Effects

### 2.1 Rain System

```lua
-- RainSystem.lua (Local Script)
-- ระบบฝนพร้อม Particle Effects

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local Lighting = game:GetService("Lighting")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- Weather Effects State
local activeEffects = {
    rain = nil,
    snow = nil,
    fog = nil,
    lightning = nil,
}

-- ===========================
-- RAIN EFFECT
-- ===========================

local function createRainEffect(intensity)
    -- intensity: 0-1
    intensity = intensity or 0.5
    
    local camera = workspace.CurrentCamera
    
    -- สร้าง Rain Part ที่ติดกับ Camera
    local rainPart = Instance.new("Part")
    rainPart.Name = "RainEmitter"
    rainPart.Size = Vector3.new(1, 1, 1)
    rainPart.Anchored = true
    rainPart.CanCollide = false
    rainPart.Transparency = 1
    rainPart.Parent = workspace
    
    -- Rain Emitter
    local emitter = Instance.new("ParticleEmitter")
    emitter.Name = "RainParticles"
    emitter.Rate = math.floor(500 * intensity)      -- จำนวนฝน
    emitter.Lifetime = NumberRange.new(1, 1.5)
    emitter.Speed = NumberRange.new(50, 60)          -- ความเร็วหยดน้ำ
    emitter.SpreadAngle = Vector2.new(5, 5)          -- กระจายนิดหน่อย
    emitter.Rotation = NumberRange.new(0, 0)
    emitter.RotSpeed = NumberRange.new(0, 0)
    emitter.Size = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.05),
        NumberSequenceKeypoint.new(1, 0.02),
    })
    emitter.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.3),
        NumberSequenceKeypoint.new(0.8, 0.3),
        NumberSequenceKeypoint.new(1, 1),
    })
    emitter.LightInfluence = 0.5
    emitter.Color = ColorSequence.new(Color3.fromRGB(150, 180, 220))
    emitter.EmissionDirection = Enum.NormalId.Bottom
    emitter.VelocityInheritance = 0
    emitter.Parent = rainPart
    
    -- Ripple Effect ที่พื้น
    local rippleEmitter = Instance.new("ParticleEmitter")
    rippleEmitter.Name = "RippleParticles"
    -- (ใส่ texture ของ ripple)
    rippleEmitter.Rate = math.floor(100 * intensity)
    rippleEmitter.Lifetime = NumberRange.new(0.5, 0.8)
    rippleEmitter.Speed = NumberRange.new(0, 1)
    rippleEmitter.Size = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.2),
        NumberSequenceKeypoint.new(0.5, 1.0),
        NumberSequenceKeypoint.new(1, 1.5),
    })
    rippleEmitter.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0),
        NumberSequenceKeypoint.new(0.7, 0.5),
        NumberSequenceKeypoint.new(1, 1),
    })
    rippleEmitter.Rotation = NumberRange.new(0, 360)
    rippleEmitter.EmissionDirection = Enum.NormalId.Top
    rippleEmitter.Parent = rainPart
    
    -- ติดตาม Camera
    local followConnection = RunService.RenderStepped:Connect(function()
        local camCFrame = camera.CFrame
        -- วาง emitter เหนือ Camera
        rainPart.CFrame = CFrame.new(camCFrame.Position + Vector3.new(0, 30, 0))
    end)
    
    return {
        part = rainPart,
        emitter = emitter,
        connection = followConnection,
        
        stop = function()
            followConnection:Disconnect()
            emitter.Enabled = false
            rippleEmitter.Enabled = false
            task.delay(2, function()
                rainPart:Destroy()
            end)
        end,
        
        setIntensity = function(newIntensity)
            emitter.Rate = math.floor(500 * newIntensity)
            rippleEmitter.Rate = math.floor(100 * newIntensity)
        end
    }
end

-- ===========================
-- SNOW EFFECT
-- ===========================

local function createSnowEffect(intensity)
    intensity = intensity or 0.5
    
    local camera = workspace.CurrentCamera
    
    local snowPart = Instance.new("Part")
    snowPart.Name = "SnowEmitter"
    snowPart.Size = Vector3.new(1, 1, 1)
    snowPart.Anchored = true
    snowPart.CanCollide = false
    snowPart.Transparency = 1
    snowPart.Parent = workspace
    
    local snowEmitter = Instance.new("ParticleEmitter")
    snowEmitter.Rate = math.floor(200 * intensity)
    snowEmitter.Lifetime = NumberRange.new(3, 5)
    snowEmitter.Speed = NumberRange.new(5, 15)
    snowEmitter.SpreadAngle = Vector2.new(30, 30)    -- กระจายมาก
    snowEmitter.Rotation = NumberRange.new(0, 360)
    snowEmitter.RotSpeed = NumberRange.new(-30, 30)
    snowEmitter.Size = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.1),
        NumberSequenceKeypoint.new(1, 0.15),
    })
    snowEmitter.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.2),
        NumberSequenceKeypoint.new(0.9, 0.2),
        NumberSequenceKeypoint.new(1, 1),
    })
    snowEmitter.LightInfluence = 0.7
    snowEmitter.Color = ColorSequence.new(Color3.fromRGB(240, 248, 255))
    snowEmitter.EmissionDirection = Enum.NormalId.Bottom
    snowEmitter.Parent = snowPart
    
    local connection = RunService.RenderStepped:Connect(function()
        local camCFrame = camera.CFrame
        snowPart.CFrame = CFrame.new(camCFrame.Position + Vector3.new(0, 50, 0))
    end)
    
    return {
        part = snowPart,
        emitter = snowEmitter,
        connection = connection,
        
        stop = function()
            connection:Disconnect()
            snowEmitter.Enabled = false
            task.delay(5, function()
                snowPart:Destroy()
            end)
        end
    }
end

-- ===========================
-- LIGHTNING EFFECT
-- ===========================

local function createLightningEffect()
    local Lighting = game:GetService("Lighting")
    
    local isActive = true
    
    local function flashLightning()
        while isActive do
            -- รอ Random Time
            task.wait(math.random(3, 8))
            
            if not isActive then break end
            
            -- Flash ฟ้าแลบ
            local originalBrightness = Lighting.Brightness
            
            -- Flash 1
            Lighting.Brightness = 10
            task.wait(0.05)
            Lighting.Brightness = originalBrightness
            task.wait(0.1)
            
            -- Flash 2
            Lighting.Brightness = 8
            task.wait(0.03)
            Lighting.Brightness = originalBrightness
            
            -- เสียงฟ้าร้อง (Simulate ด้วย delay)
            task.wait(math.random(5, 20) / 10)  -- 0.5-2 วินาที
            -- ใส่ Sound Effect ที่นี่
        end
    end
    
    task.spawn(flashLightning)
    
    return {
        stop = function()
            isActive = false
        end
    }
end

-- ===========================
-- FOG EFFECT
-- ===========================

local function applyFogEffect(density, color)
    local Lighting = game:GetService("Lighting")
    
    -- ใช้ Atmosphere สำหรับ Fog ที่สมจริง
    local atmosphere = Lighting:FindFirstChildOfClass("Atmosphere")
    if not atmosphere then
        atmosphere = Instance.new("Atmosphere")
        atmosphere.Parent = Lighting
    end
    
    TweenService:Create(atmosphere, TweenInfo.new(5), {
        Density = density,
        Color = color or Color3.fromRGB(180, 180, 190),
        Decay = color or Color3.fromRGB(120, 120, 130),
        Haze = density * 3,
    }):Play()
    
    -- ลด Fog End Distance
    TweenService:Create(Lighting, TweenInfo.new(5), {
        FogEnd = math.max(200, 2000 * (1 - density)),
        FogColor = color or Color3.fromRGB(180, 180, 190),
    }):Play()
end

-- ===========================
-- WEATHER CHANGE HANDLER
-- ===========================

local weatherFolder = ReplicatedStorage:WaitForChild("WeatherEvents")
local weatherChange = weatherFolder:WaitForChild("WeatherChange")

weatherChange.OnClientEvent:Connect(function(newWeather, oldWeather, transitionTime)
    print(string.format("[CLIENT WEATHER] %s -> %s", oldWeather or "?", newWeather))
    
    -- หยุด Effects เก่า
    for effectName, effect in pairs(activeEffects) do
        if effect then
            effect.stop()
            activeEffects[effectName] = nil
        end
    end
    
    -- เริ่ม Effects ใหม่
    if newWeather == "rain" then
        activeEffects.rain = createRainEffect(0.5)
        applyFogEffect(0.3)
        
    elseif newWeather == "heavy_rain" then
        activeEffects.rain = createRainEffect(1.0)
        applyFogEffect(0.5)
        
    elseif newWeather == "thunder" then
        activeEffects.rain = createRainEffect(1.0)
        activeEffects.lightning = createLightningEffect()
        applyFogEffect(0.6)
        
    elseif newWeather == "snow" then
        activeEffects.snow = createSnowEffect(0.6)
        applyFogEffect(0.2, Color3.fromRGB(220, 230, 240))
        
    elseif newWeather == "blizzard" then
        activeEffects.snow = createSnowEffect(1.0)
        applyFogEffect(0.8, Color3.fromRGB(200, 215, 230))
        
    elseif newWeather == "fog" then
        applyFogEffect(0.9)
        
    elseif newWeather == "clear" or newWeather == "cloudy" then
        -- ลด Fog
        applyFogEffect(0, Color3.fromRGB(200, 210, 220))
    end
end)
```

---

## 3. Sound Effects สำหรับ Weather

```lua
-- WeatherSounds.lua (Local Script)
-- เสียงประกอบสภาพอากาศ

local SoundService = game:GetService("SoundService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")

-- สร้าง Sound Container
local soundsFolder = Instance.new("Folder")
soundsFolder.Name = "WeatherSounds"
soundsFolder.Parent = workspace

-- Sound Definitions (ใส่ SoundId จริง)
local WEATHER_SOUNDS = {
    rain = {
        id = "rbxassetid://131961136",   -- Rain sound
        volume = 0.5,
        looped = true,
    },
    heavy_rain = {
        id = "rbxassetid://131961136",
        volume = 1.0,
        looped = true,
    },
    thunder = {
        id = "rbxassetid://131961136",   -- Thunder sound
        volume = 1.0,
        looped = true,
    },
    thunder_clap = {
        id = "rbxassetid://2869009983",  -- Thunder clap
        volume = 0.8,
        looped = false,
    },
    wind = {
        id = "rbxassetid://131961136",   -- Wind sound
        volume = 0.3,
        looped = true,
    },
    blizzard = {
        id = "rbxassetid://131961136",   -- Blizzard sound
        volume = 0.8,
        looped = true,
    },
}

local activeSounds = {}

-- สร้างและเล่น Sound
local function playWeatherSound(soundName)
    local config = WEATHER_SOUNDS[soundName]
    if not config then return end
    
    -- หยุด Sound เก่า
    if activeSounds[soundName] then
        TweenService:Create(activeSounds[soundName], TweenInfo.new(1), {Volume = 0}):Play()
        task.delay(1, function()
            if activeSounds[soundName] then
                activeSounds[soundName]:Stop()
                activeSounds[soundName]:Destroy()
                activeSounds[soundName] = nil
            end
        end)
    end
    
    local sound = Instance.new("Sound")
    sound.Name = soundName
    sound.SoundId = config.id
    sound.Volume = 0
    sound.Looped = config.looped
    sound.Parent = soundsFolder
    
    sound:Play()
    
    -- Fade In
    TweenService:Create(sound, TweenInfo.new(2), {Volume = config.volume}):Play()
    
    activeSounds[soundName] = sound
    return sound
end

-- หยุด Sound
local function stopWeatherSound(soundName)
    if activeSounds[soundName] then
        local sound = activeSounds[soundName]
        TweenService:Create(sound, TweenInfo.new(2), {Volume = 0}):Play()
        task.delay(2, function()
            if sound.Parent then
                sound:Stop()
                sound:Destroy()
            end
        end)
        activeSounds[soundName] = nil
    end
end

-- หยุดทุก Weather Sounds
local function stopAllWeatherSounds()
    for name, _ in pairs(activeSounds) do
        stopWeatherSound(name)
    end
end

-- รับ Weather Change Events
local weatherFolder = ReplicatedStorage:WaitForChild("WeatherEvents")
local weatherChange = weatherFolder:WaitForChild("WeatherChange")

weatherChange.OnClientEvent:Connect(function(newWeather)
    stopAllWeatherSounds()
    
    if newWeather == "rain" then
        playWeatherSound("rain")
        playWeatherSound("wind")
    elseif newWeather == "heavy_rain" then
        playWeatherSound("heavy_rain")
    elseif newWeather == "thunder" then
        playWeatherSound("heavy_rain")
        playWeatherSound("thunder")
    elseif newWeather == "blizzard" then
        playWeatherSound("blizzard")
    end
end)
```

---

## 4. Weather UI

### 4.1 Weather Display

```lua
-- WeatherUI.lua (Local Script)
-- แสดงสภาพอากาศปัจจุบัน

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- สร้าง Weather UI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "WeatherUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = playerGui

-- Weather Frame
local weatherFrame = Instance.new("Frame")
weatherFrame.Name = "WeatherFrame"
weatherFrame.Size = UDim2.new(0, 180, 0, 55)
weatherFrame.Position = UDim2.new(1, -190, 0, 75)  -- ใต้ Clock
weatherFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
weatherFrame.BackgroundTransparency = 0.4
weatherFrame.BorderSizePixel = 0
weatherFrame.Parent = screenGui

local frameCorner = Instance.new("UICorner")
frameCorner.CornerRadius = UDim.new(0, 8)
frameCorner.Parent = weatherFrame

-- Weather Icon
local weatherIcon = Instance.new("TextLabel")
weatherIcon.Size = UDim2.new(0, 40, 1, 0)
weatherIcon.BackgroundTransparency = 1
weatherIcon.TextSize = 28
weatherIcon.Font = Enum.Font.GothamBold
weatherIcon.Parent = weatherFrame

-- Weather Name
local weatherName = Instance.new("TextLabel")
weatherName.Size = UDim2.new(1, -45, 0.55, 0)
weatherName.Position = UDim2.new(0, 42, 0, 5)
weatherName.BackgroundTransparency = 1
weatherName.Text = "แดดออก"
weatherName.TextColor3 = Color3.fromRGB(255, 255, 255)
weatherName.TextSize = 14
weatherName.Font = Enum.Font.GothamBold
weatherName.TextXAlignment = Enum.TextXAlignment.Left
weatherName.Parent = weatherFrame

-- Temperature
local tempLabel = Instance.new("TextLabel")
tempLabel.Size = UDim2.new(1, -45, 0.4, 0)
tempLabel.Position = UDim2.new(0, 42, 0.58, 0)
tempLabel.BackgroundTransparency = 1
tempLabel.Text = "28°C"
tempLabel.TextColor3 = Color3.fromRGB(200, 200, 150)
tempLabel.TextSize = 12
tempLabel.Font = Enum.Font.Gotham
tempLabel.TextXAlignment = Enum.TextXAlignment.Left
tempLabel.Parent = weatherFrame

local WEATHER_INFO = {
    clear = {name = "แดดออก", icon = "☀️", temp = 28, color = Color3.fromRGB(255, 230, 100)},
    cloudy = {name = "มีเมฆ", icon = "☁️", temp = 22, color = Color3.fromRGB(180, 180, 200)},
    rain = {name = "ฝนตก", icon = "🌧️", temp = 18, color = Color3.fromRGB(100, 150, 220)},
    heavy_rain = {name = "ฝนหนัก", icon = "⛈️", temp = 15, color = Color3.fromRGB(60, 100, 180)},
    thunder = {name = "พายุฝน", icon = "⛈️", temp = 12, color = Color3.fromRGB(80, 80, 150)},
    snow = {name = "หิมะตก", icon = "🌨️", temp = -2, color = Color3.fromRGB(180, 210, 240)},
    blizzard = {name = "พายุหิมะ", icon = "❄️", temp = -15, color = Color3.fromRGB(200, 230, 255)},
    fog = {name = "หมอกหนา", icon = "🌫️", temp = 10, color = Color3.fromRGB(150, 160, 170)},
}

-- อัพเดท Weather UI
local function updateWeatherUI(newWeather, transitionTime)
    local info = WEATHER_INFO[newWeather] or WEATHER_INFO.clear
    
    -- Fade Out
    TweenService:Create(weatherFrame, TweenInfo.new(0.3), {
        BackgroundTransparency = 1
    }):Play()
    
    task.wait(0.4)
    
    if not weatherFrame.Parent then return end
    
    -- อัพเดทข้อมูล
    weatherIcon.Text = info.icon
    weatherName.Text = info.name
    tempLabel.Text = string.format("%d°C", info.temp)
    
    -- Fade In
    TweenService:Create(weatherFrame, TweenInfo.new(0.5), {
        BackgroundTransparency = 0.4
    }):Play()
    
    -- แสดง Notification ชั่วคราว
    -- (ถ้าต้องการ)
end

local weatherFolder = ReplicatedStorage:WaitForChild("WeatherEvents")
local weatherChange = weatherFolder:WaitForChild("WeatherChange")

weatherChange.OnClientEvent:Connect(function(newWeather, oldWeather, transitionTime)
    updateWeatherUI(newWeather, transitionTime)
end)
```

---

## 5. ข้อผิดพลาดที่พบบ่อย

### ❌ ข้อผิดพลาดที่ 1: Particle Effect ไม่ Destroy เมื่อเลิกใช้

```lua
-- ❌ แบบผิด
local function startRain()
    local part = Instance.new("Part")
    local emitter = Instance.new("ParticleEmitter")
    emitter.Parent = part
    part.Parent = workspace
    -- ไม่มีทางลบ!
end

-- ✅ แบบถูก: ส่งคืน Stop Function
local function startRain()
    local part = Instance.new("Part")
    -- ...
    return function()
        emitter.Enabled = false
        task.delay(emitter.Lifetime.Max, function()
            part:Destroy()
        end)
    end
end

local stopRain = startRain()
-- เมื่อต้องการหยุด:
stopRain()
```

### ❌ ข้อผิดพลาดที่ 2: ทำ Particle Effect บน Server

```lua
-- ❌ แบบผิด: ทำ Rain ที่ Server - ทุก Client เห็น Part แต่ไม่ smooth
local rainPart = Instance.new("Part")
rainPart.Parent = workspace  -- อยู่บน Server

-- ✅ แบบถูก: ทำบน Client แต่ละคน
-- Server ส่ง Event ให้ Client ทำ Effect เอง
weatherChangeEvent:FireAllClients(newWeather)
-- Client จะสร้าง Particle Effect ของตัวเอง
```

---

## 6. แบบฝึกหัด

### แบบฝึกหัดที่ 1: Seasonal Weather
สร้างระบบฤดูกาล:
- ฤดูหนาว: หิมะ บ่อยขึ้น
- ฤดูร้อน: แดดจัด ร้อน
- ฤดูฝน: ฝนบ่อย

### แบบฝึกหัดที่ 2: Weather Gameplay Effects
สร้าง Effects ที่มีผลต่อ Gameplay:
- ฝน: ปลดล็อค Water Powers
- หิมะ: ทำให้ศัตรู Slow
- พายุ: Visibility ลด + เพิ่ม Electric Damage

### แบบฝึกหัดที่ 3: Weather Forecast
สร้างระบบพยากรณ์อากาศ:
- แสดงสภาพอากาศ 3 ชั่วโมงข้างหน้า
- แจ้งเตือนเมื่อจะมีพายุ
- แสดงใน Mini-map หรือ Phone UI

---

## สรุป

Weather System ที่ดีต้องมี:
1. **Visual Effects**: Particles ที่สมจริง
2. **Sound Effects**: เสียงประกอบครบถ้วน
3. **Lighting Changes**: เปลี่ยน Atmosphere ตามสภาพอากาศ
4. **Gameplay Integration**: สภาพอากาศมีผลต่อการเล่น
5. **Performance**: Particle Effect ที่ไม่หนักเกินไป

ในส่วนถัดไป (Part 79) เราจะสร้าง Pet System ที่สมบูรณ์ พร้อมระบบ Evolution, Stats และ AI
