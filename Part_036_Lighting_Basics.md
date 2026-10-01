# ตอนที่ 36: Lighting Basics - การตั้งค่าแสงสว่าง

## บทนำ

Lighting ใน Roblox มีผลต่อบรรยากาศและความสวยงามของเกมอย่างมาก เราสามารถปรับแสงอาทิตย์, เวลาของวัน, สีบรรยากาศ, ฝน หิมะ, และเอฟเฟกต์แสงต่างๆ ได้ ในบทนี้เราจะเรียนรู้ทุกอย่างเกี่ยวกับระบบ Lighting

---

## 36.1 Lighting Service Properties

```lua
local Lighting = game:GetService("Lighting")

-- เวลาของวัน (0-24 ชั่วโมง)
Lighting.ClockTime = 14.5    -- บ่าย 2:30
Lighting.ClockTime = 0       -- เที่ยงคืน
Lighting.ClockTime = 6       -- 6 โมงเช้า

-- หรือใช้ TimeOfDay (format "HH:MM:SS")
Lighting.TimeOfDay = "14:30:00"  -- บ่าย 2:30

-- สีบรรยากาศ
Lighting.Ambient = Color3.fromRGB(100, 100, 100)  -- สีแสงทั่วไป
Lighting.OutdoorAmbient = Color3.fromRGB(150, 150, 180)  -- สีแสงนอกบ้าน

-- สีของแสงอาทิตย์
Lighting.ColorShift_Bottom = Color3.fromRGB(0, 0, 0)  -- สีพื้น
Lighting.ColorShift_Top = Color3.fromRGB(0, 0, 0)     -- สีฟ้า

-- ความสว่าง
Lighting.Brightness = 2  -- ความสว่างของแสงอาทิตย์ (default: 2)

-- Environment Diffuse (แสงกระจาย)
Lighting.EnvironmentDiffuseScale = 1.0
-- Environment Specular (แสงสะท้อน)
Lighting.EnvironmentSpecularScale = 1.0

-- Global Shadows
Lighting.GlobalShadows = true  -- เปิด Shadow ทั้งหมด

-- ShadowSoftness
Lighting.ShadowSoftness = 0.25  -- ความนุ่มของเงา (0-1)

-- Fog
Lighting.FogColor = Color3.fromRGB(192, 192, 192)  -- สีหมอก
Lighting.FogStart = 0     -- ระยะเริ่มหมอก
Lighting.FogEnd = 100000  -- ระยะหมอกหนาสุด (default)
-- ลดเพื่อเพิ่มหมอก:
Lighting.FogEnd = 200     -- หมอกหนา

-- ExposureCompensation
Lighting.ExposureCompensation = 0  -- -5 ถึง 5 (default: 0)

-- Geographic Coordinates (ตำแหน่งบนโลก สำหรับทิศทางแสง)
Lighting.GeographicLatitude = 41.733  -- ละติจูด
```

---

## 36.2 Light Objects

### 36.2.1 PointLight

```lua
-- PointLight ส่งแสงออกไปรอบทิศทาง
local part = workspace.Lamp

local pointLight = Instance.new("PointLight")
pointLight.Brightness = 5        -- ความสว่าง
pointLight.Color = Color3.fromRGB(255, 220, 100)  -- สีส้มอุ่น (เหมือนโคมไฟ)
pointLight.Range = 20             -- รัศมีส่องแสง (studs)
pointLight.Shadows = true         -- ทอดเงา
pointLight.Enabled = true
pointLight.Parent = part
```

### 36.2.2 SpotLight

```lua
-- SpotLight ส่งแสงไปในทิศทางเดียว (เหมือนไฟฉาย)
local lamp = workspace.Flashlight

local spotLight = Instance.new("SpotLight")
spotLight.Brightness = 10
spotLight.Color = Color3.new(1, 1, 1)  -- ขาว
spotLight.Range = 40
spotLight.Angle = 45        -- มุมกรวยแสง (0-180 องศา)
spotLight.Face = Enum.NormalId.Front  -- ด้านที่ส่องแสง
spotLight.Shadows = true
spotLight.Parent = lamp
```

### 36.2.3 SurfaceLight

```lua
-- SurfaceLight ส่องแสงจากผิว Part
local screen = workspace.TVScreen

local surfLight = Instance.new("SurfaceLight")
surfLight.Brightness = 3
surfLight.Color = Color3.fromRGB(100, 200, 255)  -- น้ำเงินอมเขียว (เหมือนหน้าจอ)
surfLight.Range = 15
surfLight.Angle = 90
surfLight.Face = Enum.NormalId.Front  -- ด้านหน้าของ Part
surfLight.Shadows = false
surfLight.Parent = screen
```

---

## 36.3 Atmosphere Effects

```lua
local Lighting = game:GetService("Lighting")

-- Atmosphere Object (ต้องใส่ใน Lighting)
local atmosphere = Instance.new("Atmosphere")

-- Density - ความหนาแน่นของบรรยากาศ (หมอก)
atmosphere.Density = 0.3        -- 0 = ใส, 1 = หนามาก

-- Offset - ระยะห่างก่อนเห็นหมอก
atmosphere.Offset = 0.25

-- Color - สีของหมอก/บรรยากาศ
atmosphere.Color = Color3.fromRGB(199, 170, 107)  -- สีทราย

-- Decay - การลดสีตามระยะ
atmosphere.Decay = Color3.fromRGB(100, 80, 60)

-- Glare - แสงจ้าบนขอบฟ้า
atmosphere.Glare = 0            -- 0-1

-- Haze - ฝ้าบนขอบฟ้า
atmosphere.Haze = 2.8           -- 0-10

atmosphere.Parent = Lighting

-- ตัวอย่าง: Atmosphere แบบต่างๆ

-- Clear Day
local function setAtmosphereClearDay()
    atmosphere.Density = 0.1
    atmosphere.Color = Color3.fromRGB(199, 199, 199)
    atmosphere.Haze = 0
    atmosphere.Glare = 0.1
end

-- Foggy Morning
local function setAtmosphereFoggy()
    atmosphere.Density = 0.7
    atmosphere.Color = Color3.fromRGB(200, 210, 220)
    atmosphere.Haze = 5
    atmosphere.Glare = 0
end

-- Sunset
local function setAtmosphereSunset()
    atmosphere.Density = 0.4
    atmosphere.Color = Color3.fromRGB(255, 180, 100)
    atmosphere.Haze = 3
    atmosphere.Glare = 1
end

-- Night
local function setAtmosphereNight()
    atmosphere.Density = 0.2
    atmosphere.Color = Color3.fromRGB(40, 60, 100)
    atmosphere.Haze = 0
    atmosphere.Glare = 0
end
```

---

## 36.4 Sky Box

```lua
local Lighting = game:GetService("Lighting")

-- สร้าง Sky
local sky = Instance.new("Sky")

-- Skybox Images (ต้องมีทั้ง 6 ด้าน)
sky.SkyboxBk = "rbxassetid://SKY_BACK"    -- ด้านหลัง
sky.SkyboxDn = "rbxassetid://SKY_DOWN"    -- ด้านล่าง
sky.SkyboxFt = "rbxassetid://SKY_FRONT"   -- ด้านหน้า
sky.SkyboxLf = "rbxassetid://SKY_LEFT"    -- ด้านซ้าย
sky.SkyboxRt = "rbxassetid://SKY_RIGHT"   -- ด้านขวา
sky.SkyboxUp = "rbxassetid://SKY_TOP"     -- ด้านบน

-- ดาว
sky.StarCount = 3000       -- จำนวนดาว
sky.CelestialBodiesShown = true  -- แสดงดวงอาทิตย์และดวงจันทร์

-- ดวงอาทิตย์
sky.SunAngularSize = 21    -- ขนาดดวงอาทิตย์
sky.SunDecorationCount = 2 -- จำนวนแสงรอบดวงอาทิตย์
sky.SunTextureId = "rbxassetid://SUN_TEX"

-- ดวงจันทร์
sky.MoonAngularSize = 11
sky.MoonTextureId = "rbxassetid://MOON_TEX"

sky.Parent = Lighting
```

---

## 36.5 Post Processing Effects

```lua
local Lighting = game:GetService("Lighting")

-- 1. BloomEffect - แสงเรืองรอง
local bloom = Instance.new("BloomEffect")
bloom.Intensity = 0.8    -- ความแรง (0-1)
bloom.Size = 24          -- ขนาด Bloom (1-56)
bloom.Threshold = 2      -- ค่าความสว่างขั้นต่ำที่จะเกิด Bloom (0-10)
bloom.Parent = Lighting

-- 2. BlurEffect - เบลอ
local blur = Instance.new("BlurEffect")
blur.Size = 5            -- ขนาดการ Blur (0-56)
blur.Parent = Lighting

-- 3. ColorCorrectionEffect - แก้ไขสี
local colorCorrect = Instance.new("ColorCorrectionEffect")
colorCorrect.Brightness = 0      -- -1 ถึง 1 (ความสว่าง)
colorCorrect.Contrast = 0        -- -1 ถึง 1 (คอนทราสต์)
colorCorrect.Saturation = 0      -- -1 ถึง 1 (ความสดใส)
colorCorrect.TintColor = Color3.new(1, 1, 1)  -- Tint
colorCorrect.Parent = Lighting

-- 4. DepthOfFieldEffect - โฟกัสระยะ
local dof = Instance.new("DepthOfFieldEffect")
dof.FarIntensity = 1.0   -- ความเบลอระยะไกล
dof.FocusDistance = 20   -- ระยะที่โฟกัส
dof.InFocusRadius = 5    -- รัศมีที่ชัด
dof.NearIntensity = 0    -- ความเบลอระยะใกล้
dof.Parent = Lighting

-- 5. SunRaysEffect - แสงอาทิตย์
local sunRays = Instance.new("SunRaysEffect")
sunRays.Intensity = 0.25  -- ความแรงของแสง (0-1)
sunRays.Spread = 0.5      -- การกระจาย (0-1)
sunRays.Parent = Lighting
```

---

## 36.6 Day/Night Cycle

```lua
-- Script: Day/Night Cycle
local Lighting = game:GetService("Lighting")
local TweenService = game:GetService("TweenService")

-- ระยะเวลา 1 วันเกม (วินาที)
local DAY_LENGTH = 600  -- 10 นาที = 1 วัน

local function updateLighting()
    local currentTime = Lighting.ClockTime
    
    -- ปรับแสงตามเวลา
    if currentTime >= 6 and currentTime < 8 then
        -- รุ่งเช้า
        Lighting.Ambient = Color3.fromRGB(80, 70, 60)
        Lighting.OutdoorAmbient = Color3.fromRGB(150, 130, 110)
        Lighting.Brightness = 1.5
    elseif currentTime >= 8 and currentTime < 17 then
        -- กลางวัน
        Lighting.Ambient = Color3.fromRGB(120, 120, 120)
        Lighting.OutdoorAmbient = Color3.fromRGB(180, 180, 200)
        Lighting.Brightness = 2
    elseif currentTime >= 17 and currentTime < 19 then
        -- เย็น
        Lighting.Ambient = Color3.fromRGB(100, 70, 50)
        Lighting.OutdoorAmbient = Color3.fromRGB(200, 140, 80)
        Lighting.Brightness = 1.8
    elseif currentTime >= 19 and currentTime < 21 then
        -- ค่ำ
        Lighting.Ambient = Color3.fromRGB(50, 40, 60)
        Lighting.OutdoorAmbient = Color3.fromRGB(70, 60, 90)
        Lighting.Brightness = 0.8
    else
        -- กลางคืน
        Lighting.Ambient = Color3.fromRGB(20, 20, 40)
        Lighting.OutdoorAmbient = Color3.fromRGB(30, 30, 60)
        Lighting.Brightness = 0.3
    end
end

-- Run Cycle
local RunService = game:GetService("RunService")
local startTime = tick()
local initialClockTime = 8  -- เริ่มที่ 8 โมงเช้า

RunService.Heartbeat:Connect(function(dt)
    local elapsed = tick() - startTime
    local dayProgress = (elapsed / DAY_LENGTH) % 1
    local gameTime = (initialClockTime + dayProgress * 24) % 24
    
    Lighting.ClockTime = gameTime
    updateLighting()
end)
```

---

## 36.7 Weather Effects

```lua
-- LocalScript / Script: Weather System
local Lighting = game:GetService("Lighting")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")

local WeatherSystem = {}

-- Atmosphere สำหรับ Weather
local atmosphere = Lighting:FindFirstChildOfClass("Atmosphere") or 
    Instance.new("Atmosphere", Lighting)

-- Rain Part
local rainFolder = Instance.new("Folder")
rainFolder.Name = "Rain"
rainFolder.Parent = workspace

-- ฝน
function WeatherSystem.setRain(intensity)
    intensity = math.clamp(intensity, 0, 1)
    
    -- ปรับ Lighting
    TweenService:Create(Lighting, TweenInfo.new(3), {
        Brightness = 1.5 - intensity * 0.8,
        Ambient = Color3.new(0.4 - intensity * 0.2, 0.4 - intensity * 0.2, 0.5 - intensity * 0.1)
    }):Play()
    
    -- ปรับ Atmosphere
    TweenService:Create(atmosphere, TweenInfo.new(3), {
        Density = 0.1 + intensity * 0.4,
        Color = Color3.fromRGB(150 + intensity * 30, 160 + intensity * 20, 180 + intensity * 10),
        Haze = intensity * 3
    }):Play()
    
    -- เปิด/ปิด Rain Particles (ต้องมี Part ชื่อ RainEmitter ใน workspace)
    local emitter = workspace:FindFirstChild("RainEmitter")
    if emitter then
        local particleEmitter = emitter:FindFirstChildOfClass("ParticleEmitter")
        if particleEmitter then
            particleEmitter.Rate = math.floor(intensity * 500)
        end
    end
end

-- หิมะ
function WeatherSystem.setSnow(intensity)
    intensity = math.clamp(intensity, 0, 1)
    
    TweenService:Create(Lighting, TweenInfo.new(3), {
        Brightness = 1.8 + intensity * 0.3,
        Ambient = Color3.new(0.7 + intensity * 0.2, 0.7 + intensity * 0.2, 0.8 + intensity * 0.1)
    }):Play()
    
    TweenService:Create(atmosphere, TweenInfo.new(3), {
        Density = intensity * 0.5,
        Color = Color3.fromRGB(200, 210, 230),
        Haze = intensity * 2
    }):Play()
end

-- หมอก
function WeatherSystem.setFog(intensity)
    intensity = math.clamp(intensity, 0, 1)
    
    TweenService:Create(atmosphere, TweenInfo.new(5), {
        Density = intensity * 0.9,
        Color = Color3.fromRGB(180, 190, 200),
        Haze = intensity * 8
    }):Play()
end

-- พายุทราย
function WeatherSystem.setSandstorm(intensity)
    intensity = math.clamp(intensity, 0, 1)
    
    TweenService:Create(Lighting, TweenInfo.new(3), {
        Brightness = 1.5 - intensity * 0.7,
        Ambient = Color3.new(0.6 + intensity * 0.2, 0.5 + intensity * 0.1, 0.3)
    }):Play()
    
    TweenService:Create(atmosphere, TweenInfo.new(3), {
        Density = 0.3 + intensity * 0.6,
        Color = Color3.fromRGB(220, 180, 120),
        Haze = intensity * 6,
        Glare = intensity
    }):Play()
end

-- Clear Weather
function WeatherSystem.setClear()
    TweenService:Create(Lighting, TweenInfo.new(3), {
        Brightness = 2,
        Ambient = Color3.fromRGB(120, 120, 120)
    }):Play()
    
    TweenService:Create(atmosphere, TweenInfo.new(3), {
        Density = 0.1,
        Color = Color3.fromRGB(199, 199, 199),
        Haze = 0,
        Glare = 0
    }):Play()
end

return WeatherSystem
```

---

## 36.8 Interior Lighting

```lua
-- Script: Interior Lighting
-- สำหรับการตั้งค่าแสงภายในอาคาร

local function setupInteriorLighting()
    local Lighting = game:GetService("Lighting")
    
    -- เมื่ออยู่ในอาคาร ลดแสงนอก
    Lighting.OutdoorAmbient = Color3.fromRGB(50, 50, 50)
    Lighting.Brightness = 0.5
end

-- ตัวอย่าง: ไฟในบ้าน
local house = workspace.House
local lights = {}

-- สร้างไฟในทุกห้อง
for _, room in ipairs(house:GetChildren()) do
    if room.Name:find("Room") then
        -- หาจุดกลางห้อง
        local center = room:FindFirstChild("LightPoint")
        if center then
            local light = Instance.new("PointLight")
            light.Brightness = 3
            light.Color = Color3.fromRGB(255, 240, 200)  -- สีเหลืองอุ่น
            light.Range = 20
            light.Shadows = true
            light.Parent = center
            table.insert(lights, light)
        end
    end
end

-- Toggle ไฟ
local lightsOn = true

local function toggleLights()
    lightsOn = not lightsOn
    for _, light in ipairs(lights) do
        light.Enabled = lightsOn
    end
end
```

---

## 36.9 Dynamic Lighting (Script Controlled)

```lua
-- Script: Dynamic Lighting Effects

local TweenService = game:GetService("TweenService")

-- ไฟกระพริบ (Flickering Light)
local function createFlickerLight(part)
    local light = Instance.new("PointLight")
    light.Brightness = 5
    light.Color = Color3.fromRGB(255, 200, 100)
    light.Range = 15
    light.Parent = part
    
    local RunService = game:GetService("RunService")
    RunService.Heartbeat:Connect(function()
        if math.random() < 0.05 then  -- 5% chance
            light.Brightness = math.random(2, 7)
        end
    end)
    
    return light
end

-- ไฟฟ้าแลบ (Lightning)
local function createLightning()
    local Lighting = game:GetService("Lighting")
    
    local function flash()
        local originalBrightness = Lighting.Brightness
        
        -- Flash!
        Lighting.Brightness = 6
        task.wait(0.05)
        Lighting.Brightness = 3
        task.wait(0.05)
        Lighting.Brightness = 6
        task.wait(0.05)
        Lighting.Brightness = originalBrightness
        
        -- เสียงฟ้าร้อง
        local thunder = Instance.new("Sound")
        thunder.SoundId = "rbxassetid://THUNDER_SOUND_ID"
        thunder.Volume = 1
        thunder.Parent = game.SoundService
        thunder:Play()
        thunder.Ended:Connect(function() thunder:Destroy() end)
    end
    
    -- Random ฟ้าแลบ
    task.spawn(function()
        while true do
            local nextFlash = math.random(5, 30)  -- 5-30 วินาที
            task.wait(nextFlash)
            flash()
        end
    end)
end

-- ไฟดิสโก้
local function createDiscoLight(part)
    local light = Instance.new("PointLight")
    light.Brightness = 8
    light.Range = 30
    light.Parent = part
    
    local RunService = game:GetService("RunService")
    local hue = 0
    
    RunService.Heartbeat:Connect(function(dt)
        hue = (hue + dt) % 1
        light.Color = Color3.fromHSV(hue, 1, 1)
    end)
    
    -- หมุน SpotLight
    local spotLight = Instance.new("SpotLight")
    spotLight.Brightness = 10
    spotLight.Range = 40
    spotLight.Angle = 30
    spotLight.Parent = part
    
    local angle = 0
    RunService.Heartbeat:Connect(function(dt)
        angle = angle + dt * 60
        part.CFrame = CFrame.new(part.Position) * CFrame.Angles(0, math.rad(angle), 0)
    end)
    
    return {light, spotLight}
end
```

---

## 36.10 แบบฝึกหัด

### แบบฝึกหัดที่ 1: Time-based Preset System

```lua
-- Script: Lighting Presets
local Lighting = game:GetService("Lighting")
local TweenService = game:GetService("TweenService")

local LightingPresets = {}

-- Presets
local presets = {
    dawn = {
        ClockTime = 6,
        Brightness = 1.5,
        Ambient = Color3.fromRGB(80, 70, 70),
        OutdoorAmbient = Color3.fromRGB(150, 130, 110),
        ColorShift_Top = Color3.fromRGB(255, 150, 50),
        FogEnd = 500,
        FogColor = Color3.fromRGB(255, 200, 150),
    },
    
    midday = {
        ClockTime = 12,
        Brightness = 2.5,
        Ambient = Color3.fromRGB(140, 140, 140),
        OutdoorAmbient = Color3.fromRGB(200, 200, 210),
        ColorShift_Top = Color3.new(0, 0, 0),
        FogEnd = 100000,
        FogColor = Color3.fromRGB(192, 192, 192),
    },
    
    sunset = {
        ClockTime = 18.5,
        Brightness = 1.8,
        Ambient = Color3.fromRGB(100, 70, 50),
        OutdoorAmbient = Color3.fromRGB(200, 140, 80),
        ColorShift_Top = Color3.fromRGB(200, 100, 50),
        FogEnd = 300,
        FogColor = Color3.fromRGB(220, 150, 100),
    },
    
    night = {
        ClockTime = 22,
        Brightness = 0.3,
        Ambient = Color3.fromRGB(20, 20, 40),
        OutdoorAmbient = Color3.fromRGB(30, 30, 60),
        ColorShift_Top = Color3.new(0, 0, 0),
        FogEnd = 150,
        FogColor = Color3.fromRGB(20, 30, 50),
    },
    
    apocalypse = {
        ClockTime = 14,
        Brightness = 0.8,
        Ambient = Color3.fromRGB(80, 40, 30),
        OutdoorAmbient = Color3.fromRGB(150, 80, 50),
        ColorShift_Top = Color3.fromRGB(150, 50, 0),
        FogEnd = 100,
        FogColor = Color3.fromRGB(200, 80, 50),
    },
}

function LightingPresets.apply(presetName, duration)
    local preset = presets[presetName]
    if not preset then
        warn("ไม่พบ Preset:", presetName)
        return
    end
    
    duration = duration or 3
    
    -- Tween ทุก Properties
    local tweenGoals = {}
    for prop, value in pairs(preset) do
        if prop ~= "ClockTime" then
            tweenGoals[prop] = value
        end
    end
    
    TweenService:Create(
        Lighting,
        TweenInfo.new(duration, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut),
        tweenGoals
    ):Play()
    
    -- ClockTime แยก
    TweenService:Create(
        Lighting,
        TweenInfo.new(duration, Enum.EasingStyle.Linear),
        {ClockTime = preset.ClockTime}
    ):Play()
    
    print("เปลี่ยนเป็น Preset:", presetName)
end

-- ตัวอย่าง: เปลี่ยน Preset ทุก 10 วินาที
local presetNames = {"dawn", "midday", "sunset", "night"}
local presetIndex = 1

task.spawn(function()
    while true do
        LightingPresets.apply(presetNames[presetIndex], 3)
        presetIndex = (presetIndex % #presetNames) + 1
        task.wait(10)
    end
end)
```

---

## 36.11 Neon Lighting

```lua
-- Script: Neon City Lighting
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")

-- ตั้งค่า Lighting แบบ Cyberpunk
local Lighting = game:GetService("Lighting")
Lighting.ClockTime = 23
Lighting.Brightness = 0.2
Lighting.Ambient = Color3.fromRGB(10, 5, 20)
Lighting.OutdoorAmbient = Color3.fromRGB(20, 10, 40)
Lighting.FogEnd = 200
Lighting.FogColor = Color3.fromRGB(20, 10, 50)

-- Bloom สำหรับ Neon
local bloom = Lighting:FindFirstChildOfClass("BloomEffect") or 
    Instance.new("BloomEffect", Lighting)
bloom.Intensity = 1.5
bloom.Size = 40
bloom.Threshold = 0.5

-- สร้าง Neon Signs
local neonColors = {
    Color3.fromRGB(255, 0, 100),    -- ชมพู
    Color3.fromRGB(0, 200, 255),    -- ฟ้า Cyan
    Color3.fromRGB(150, 0, 255),    -- ม่วง
    Color3.fromRGB(255, 100, 0),    -- ส้ม
    Color3.fromRGB(0, 255, 100),    -- เขียว Neon
}

local function createNeonLight(parent, color)
    -- Neon Part
    local neonPart = Instance.new("Part")
    neonPart.BrickColor = BrickColor.new("Neon orange")  -- ต้องเป็น Neon material
    neonPart.Material = Enum.Material.Neon
    neonPart.Color = color
    neonPart.Size = Vector3.new(0.5, 0.5, 0.5)
    neonPart.Anchored = true
    neonPart.Parent = parent
    
    -- Point Light
    local light = Instance.new("PointLight")
    light.Color = color
    light.Brightness = 5
    light.Range = 15
    light.Parent = neonPart
    
    -- Flicker
    task.spawn(function()
        while neonPart.Parent do
            if math.random() < 0.02 then
                neonPart.Transparency = 0.8
                light.Enabled = false
                task.wait(0.05)
                neonPart.Transparency = 0
                light.Enabled = true
            end
            task.wait(0.1)
        end
    end)
    
    return neonPart
end
```

---

## 36.12 สรุป

ในบทนี้เราได้เรียนรู้:

1. **Lighting Properties** - ClockTime, Ambient, Brightness, Fog
2. **Light Objects** - PointLight, SpotLight, SurfaceLight
3. **Atmosphere** - Density, Color, Haze, Glare
4. **Sky Box** - Custom Skybox
5. **Post-Processing Effects** - Bloom, Blur, ColorCorrection, DepthOfField, SunRays
6. **Day/Night Cycle** - ระบบวนรอบวัน/คืน
7. **Weather System** - ฝน, หิมะ, หมอก, พายุทราย
8. **Interior Lighting** - แสงภายในอาคาร
9. **Dynamic Lighting** - ไฟกระพริบ, ฟ้าแลบ, Disco
10. **Lighting Presets** - ชุดตั้งค่าแสงสำเร็จรูป
11. **Neon Lighting** - แสง Neon สไตล์ Cyberpunk

ในบทต่อไป เราจะเรียนรู้เกี่ยวกับ Particle Effects

---

## แหล่งอ้างอิง

- [Roblox Developer Hub - Lighting](https://developer.roblox.com/en-us/api-reference/class/Lighting)
- [Roblox Developer Hub - Atmosphere](https://developer.roblox.com/en-us/api-reference/class/Atmosphere)
- [Roblox Developer Hub - PointLight](https://developer.roblox.com/en-us/api-reference/class/PointLight)
