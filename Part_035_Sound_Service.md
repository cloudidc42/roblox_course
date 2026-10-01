# ตอนที่ 35: SoundService - เพิ่มเสียงในเกม

## บทนำ

เสียงเป็นส่วนสำคัญที่ทำให้เกมมีชีวิตชีวา ใน Roblox เราสามารถเพิ่มเพลงประกอบ, เสียงเอฟเฟกต์, เสียงสิ่งแวดล้อม และอื่นๆ ได้ผ่าน SoundService และ Sound Objects ในบทนี้เราจะเรียนรู้ทุกอย่างเกี่ยวกับระบบเสียงใน Roblox

---

## 35.1 พื้นฐาน Sound

```lua
-- Script / LocalScript
local SoundService = game:GetService("SoundService")

-- สร้าง Sound
local sound = Instance.new("Sound")
sound.SoundId = "rbxassetid://123456789"  -- Asset ID ของไฟล์เสียง
sound.Volume = 0.5           -- ระดับเสียง (0-1)
sound.Pitch = 1.0            -- ระดับเสียงสูงต่ำ (0.5 = ครึ่งหนึ่ง, 2 = สองเท่า)
sound.RollOffMaxDistance = 60 -- ระยะได้ยินสูงสุด (สำหรับ 3D Sound)
sound.Looped = false         -- วนซ้ำ
sound.PlaybackSpeed = 1.0    -- ความเร็วเล่น

-- ที่วาง Sound มีผลต่อการทำงาน:
-- ใน SoundService - เสียง 2D (ได้ยินทั่วทั้งเกม)
-- ใน Part - เสียง 3D (ดังตามระยะทาง)
-- ใน Workspace - เสียง 2D Global

-- Sound ใน SoundService (Background Music)
sound.Parent = SoundService

-- Sound ใน Part (3D Positional)
-- sound.Parent = workspace.SomePart

-- เล่นเสียง
sound:Play()

-- หยุดเสียง
sound:Stop()

-- Pause
sound:Pause()

-- Resume หลัง Pause
sound:Resume()

-- Events
sound.Played:Connect(function()
    print("เสียงเริ่มเล่น!")
end)

sound.Ended:Connect(function()
    print("เสียงหยุด!")
end)

sound.Paused:Connect(function()
    print("เสียง Pause!")
end)

sound.Resumed:Connect(function()
    print("เสียง Resume!")
end)

sound.DidLoop:Connect(function()
    print("เสียงวนซ้ำ!")
end)
```

---

## 35.2 SoundService Properties

```lua
local SoundService = game:GetService("SoundService")

-- ระดับเสียงทั้งหมด (Master Volume)
SoundService.MasterVolume = 0.8

-- เปิด/ปิดเสียงทั้งหมด
SoundService.MuteAudio = false

-- Ambient Reverb (เสียงก้อง)
SoundService.AmbientReverb = Enum.ReverbType.NoReverb
-- NoReverb, GenericReverb, PaddedCell, Room, Bathroom,
-- StoneRoom, Auditorium, ConcertHall, Cave, Arena,
-- Hangar, CarpettedHallway, Hallway, StoneCorridor,
-- Alley, Forest, City, Mountains, Quarry, Plain,
-- ParkingLot, SewerPipe, Underwater, SmallRoom, 
-- MediumRoom, LargeRoom, MediumHall, LargeHall, Plate

-- Doppler Effect
SoundService.DopplerScale = 1.0  -- 0 = ปิด, >1 = เพิ่ม effect
```

---

## 35.3 การจัดการเสียงหลายชนิด

```lua
-- ModuleScript: SoundManager
local SoundService = game:GetService("SoundService")

local SoundManager = {}

-- Categories ของเสียง
local categories = {
    music = Instance.new("SoundGroup"),
    sfx = Instance.new("SoundGroup"),
    ambient = Instance.new("SoundGroup"),
    ui = Instance.new("SoundGroup"),
}

-- ตั้งค่า SoundGroups
categories.music.Name = "Music"
categories.music.Volume = 0.7
categories.music.Parent = SoundService

categories.sfx.Name = "SFX"
categories.sfx.Volume = 1.0
categories.sfx.Parent = SoundService

categories.ambient.Name = "Ambient"
categories.ambient.Volume = 0.5
categories.ambient.Parent = SoundService

categories.ui.Name = "UI"
categories.ui.Volume = 0.8
categories.ui.Parent = SoundService

-- Sound Registry
local sounds = {}

-- ลงทะเบียนเสียง
function SoundManager.register(id, soundId, category, config)
    local sound = Instance.new("Sound")
    sound.Name = id
    sound.SoundId = "rbxassetid://" .. soundId
    
    -- Apply config
    if config then
        sound.Volume = config.volume or 1.0
        sound.Pitch = config.pitch or 1.0
        sound.Looped = config.looped or false
        sound.PlaybackSpeed = config.speed or 1.0
        sound.RollOffMaxDistance = config.maxDist or 60
        sound.RollOffMinDistance = config.minDist or 10
    end
    
    -- กำหนด Category (SoundGroup)
    if categories[category] then
        sound.SoundGroup = categories[category]
    end
    
    sound.Parent = SoundService
    sounds[id] = sound
    return sound
end

-- เล่นเสียง
function SoundManager.play(id, config)
    local sound = sounds[id]
    if not sound then
        warn("ไม่พบเสียง:", id)
        return
    end
    
    -- Apply runtime config
    if config then
        if config.volume then sound.Volume = config.volume end
        if config.pitch then sound.Pitch = config.pitch end
    end
    
    -- ถ้าเล่นอยู่แล้ว ให้ Clone
    if sound.IsPlaying then
        local clone = sound:Clone()
        clone.Parent = SoundService
        clone:Play()
        clone.Ended:Connect(function()
            clone:Destroy()
        end)
        return clone
    end
    
    sound:Play()
    return sound
end

-- หยุดเสียง
function SoundManager.stop(id)
    local sound = sounds[id]
    if sound then
        sound:Stop()
    end
end

-- Fade Out
function SoundManager.fadeOut(id, duration)
    local sound = sounds[id]
    if not sound then return end
    
    local TweenService = game:GetService("TweenService")
    local tween = TweenService:Create(
        sound,
        TweenInfo.new(duration or 1),
        {Volume = 0}
    )
    tween:Play()
    tween.Completed:Connect(function()
        sound:Stop()
        sound.Volume = sound.Volume  -- Reset
    end)
end

-- Fade In
function SoundManager.fadeIn(id, targetVolume, duration)
    local sound = sounds[id]
    if not sound then return end
    
    sound.Volume = 0
    sound:Play()
    
    local TweenService = game:GetService("TweenService")
    TweenService:Create(
        sound,
        TweenInfo.new(duration or 1),
        {Volume = targetVolume or 1}
    ):Play()
end

-- ปรับระดับเสียงทุก Category
function SoundManager.setCategoryVolume(category, volume)
    if categories[category] then
        categories[category].Volume = volume
    end
end

-- Mute/Unmute Category
function SoundManager.muteCategory(category, muted)
    if categories[category] then
        categories[category].Volume = muted and 0 or 1
    end
end

return SoundManager
```

---

## 35.4 Background Music System

```lua
-- Script: Background Music System
local SoundService = game:GetService("SoundService")
local TweenService = game:GetService("TweenService")

local MusicSystem = {}

-- Playlist
local playlist = {
    {id = "rbxassetid://MUSIC_1", name = "Main Theme", duration = 180},
    {id = "rbxassetid://MUSIC_2", name = "Battle Theme", duration = 120},
    {id = "rbxassetid://MUSIC_3", name = "Peaceful Theme", duration = 200},
}

local currentIndex = 0
local currentMusic = nil
local isPlaying = false
local isShuffle = false

-- สร้าง Music Player
local musicSound = Instance.new("Sound")
musicSound.Volume = 0.5
musicSound.Looped = false
musicSound.Parent = SoundService

-- เล่นเพลง
local function playTrack(index)
    local track = playlist[index]
    if not track then return end
    
    currentIndex = index
    
    -- Fade Out เพลงเก่า
    if currentMusic and musicSound.IsPlaying then
        TweenService:Create(musicSound, TweenInfo.new(1), {Volume = 0}):Play()
        task.wait(1)
        musicSound:Stop()
    end
    
    -- เล่นเพลงใหม่
    musicSound.SoundId = track.id
    musicSound.Volume = 0
    musicSound:Play()
    
    -- Fade In
    TweenService:Create(musicSound, TweenInfo.new(1), {Volume = 0.5}):Play()
    
    currentMusic = track
    print("กำลังเล่น:", track.name)
end

-- เล่นต่อไป
local function playNext()
    local nextIndex
    
    if isShuffle then
        nextIndex = math.random(1, #playlist)
    else
        nextIndex = (currentIndex % #playlist) + 1
    end
    
    playTrack(nextIndex)
end

-- เล่น
function MusicSystem.play(startIndex)
    isPlaying = true
    playTrack(startIndex or 1)
end

-- หยุด
function MusicSystem.stop()
    isPlaying = false
    TweenService:Create(musicSound, TweenInfo.new(1), {Volume = 0}):Play()
    task.delay(1, function()
        musicSound:Stop()
    end)
end

-- Pause
function MusicSystem.pause()
    musicSound:Pause()
end

-- Resume
function MusicSystem.resume()
    musicSound:Resume()
end

-- เปลี่ยนเพลง
function MusicSystem.next()
    playNext()
end

function MusicSystem.previous()
    local prevIndex = ((currentIndex - 2) % #playlist) + 1
    playTrack(prevIndex)
end

-- ระดับเสียง
function MusicSystem.setVolume(volume)
    musicSound.Volume = volume
end

-- Shuffle
function MusicSystem.setShuffle(enabled)
    isShuffle = enabled
end

-- เมื่อเพลงจบ
musicSound.Ended:Connect(function()
    if isPlaying then
        task.wait(1)  -- รอก่อนเล่นเพลงถัดไป
        playNext()
    end
end)

return MusicSystem
```

---

## 35.5 Sound Effects System

```lua
-- Script: Sound Effects
local SoundService = game:GetService("SoundService")

local SFX = {}

-- Pool ของ Sound สำหรับ Pooling
local soundPool = {}
local MAX_POOL_SIZE = 20

local function getFromPool(soundId)
    if not soundPool[soundId] then
        soundPool[soundId] = {}
    end
    
    -- หา Sound ที่ว่าง
    for _, sound in ipairs(soundPool[soundId]) do
        if not sound.IsPlaying then
            return sound
        end
    end
    
    -- สร้างใหม่ถ้ายังไม่เต็ม Pool
    if #soundPool[soundId] < MAX_POOL_SIZE then
        local sound = Instance.new("Sound")
        sound.SoundId = "rbxassetid://" .. soundId
        sound.Parent = SoundService
        table.insert(soundPool[soundId], sound)
        return sound
    end
    
    return nil
end

-- เล่น SFX
function SFX.play(soundId, config)
    local sound = getFromPool(soundId)
    if not sound then return end
    
    -- Apply config
    if config then
        sound.Volume = config.volume or 1.0
        sound.Pitch = config.pitch or 1.0
        
        -- Random Pitch (ทำให้เสียงไม่ซ้ำกัน)
        if config.pitchVariation then
            local variation = config.pitchVariation
            sound.Pitch = 1 + (math.random() * 2 - 1) * variation
        end
    end
    
    sound:Play()
    return sound
end

-- เล่น SFX ที่ Position (3D)
function SFX.playAt(soundId, position, config)
    local part = Instance.new("Part")
    part.Size = Vector3.new(1, 1, 1)
    part.Position = position
    part.Anchored = true
    part.CanCollide = false
    part.Transparency = 1
    part.Parent = workspace
    
    local sound = Instance.new("Sound")
    sound.SoundId = "rbxassetid://" .. soundId
    sound.Volume = config and config.volume or 1.0
    sound.RollOffMaxDistance = config and config.maxDist or 40
    sound.Parent = part
    
    sound:Play()
    
    sound.Ended:Connect(function()
        part:Destroy()
    end)
    
    return sound
end

-- ตัวอย่าง SFX IDs
local sfxIds = {
    jump = "12345678",
    land = "23456789",
    hit = "34567890",
    collect = "45678901",
    button_click = "56789012",
    explosion = "67890123",
    footstep_grass = "78901234",
    footstep_wood = "89012345",
}

-- Register ทั้งหมด
for name, id in pairs(sfxIds) do
    getFromPool(id)  -- Pre-warm pool
end

return SFX
```

---

## 35.6 Footstep Sound System

```lua
-- LocalScript: Footsteps
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local SoundService = game:GetService("SoundService")

local LocalPlayer = Players.LocalPlayer

-- Sound IDs สำหรับ Material ต่างๆ
local footstepSounds = {
    [Enum.Material.Grass] = {
        "GRASS_1_ID", "GRASS_2_ID", "GRASS_3_ID"
    },
    [Enum.Material.Wood] = {
        "WOOD_1_ID", "WOOD_2_ID"
    },
    [Enum.Material.Concrete] = {
        "CONCRETE_1_ID", "CONCRETE_2_ID", "CONCRETE_3_ID"
    },
    [Enum.Material.Sand] = {
        "SAND_1_ID", "SAND_2_ID"
    },
    [Enum.Material.Water] = {
        "WATER_1_ID", "WATER_2_ID"
    },
    default = {
        "DEFAULT_1_ID", "DEFAULT_2_ID"
    }
}

-- สร้าง Sound สำหรับ Footstep
local footstepSound = Instance.new("Sound")
footstepSound.Volume = 0.5
footstepSound.Parent = SoundService

local stepInterval = 0.4  -- วินาทีระหว่างก้าว
local lastStepTime = 0
local lastStepFoot = false  -- สลับเท้าซ้าย-ขวา

-- ตรวจสอบพื้นผิวที่ยืนอยู่
local function getGroundMaterial(character)
    local hrp = character:FindFirstChild("HumanoidRootPart")
    if not hrp then return Enum.Material.Grass end
    
    local ray = workspace:Raycast(
        hrp.Position,
        Vector3.new(0, -4, 0),
        RaycastParams.new()
    )
    
    if ray then
        return ray.Material
    end
    
    return Enum.Material.Grass
end

-- เล่นเสียงก้าว
local function playFootstep(material)
    local sounds = footstepSounds[material] or footstepSounds.default
    local soundId = sounds[math.random(1, #sounds)]
    
    footstepSound.SoundId = "rbxassetid://" .. soundId
    footstepSound.Pitch = 0.9 + math.random() * 0.2  -- Random pitch
    footstepSound:Play()
end

-- อัพเดต Footsteps
RunService.Heartbeat:Connect(function()
    local character = LocalPlayer.Character
    if not character then return end
    
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end
    
    -- ตรวจสอบว่ากำลังเดินหรือวิ่ง
    local isMoving = humanoid.MoveDirection.Magnitude > 0.1
    local isOnGround = humanoid:GetState() == Enum.HumanoidStateType.Running or
                       humanoid:GetState() == Enum.HumanoidStateType.Idle
    
    if isMoving and isOnGround then
        local now = tick()
        local interval = stepInterval
        
        -- ถ้าวิ่ง เพิ่มความถี่เสียง
        if humanoid.WalkSpeed > 20 then
            interval = interval * 0.6
        end
        
        if now - lastStepTime >= interval then
            lastStepTime = now
            local material = getGroundMaterial(character)
            playFootstep(material)
        end
    end
end)
```

---

## 35.7 3D Spatial Sound

```lua
-- Script: 3D Spatial Sound
-- เสียงที่ดังจากตำแหน่งใน 3D Space

-- เสียงไฟ (Crackling Fire)
local function createFireSound(part)
    local fireSound = Instance.new("Sound")
    fireSound.SoundId = "rbxassetid://FIRE_SOUND_ID"
    fireSound.Volume = 0.8
    fireSound.Looped = true
    fireSound.RollOffMode = Enum.RollOffMode.Inverse
    fireSound.RollOffMinDistance = 5    -- ระยะที่ได้ยินเต็มที่
    fireSound.RollOffMaxDistance = 40   -- ระยะสูงสุด
    fireSound.Parent = part
    fireSound:Play()
    return fireSound
end

-- เสียงน้ำตก
local function createWaterfallSound(part)
    local waterSound = Instance.new("Sound")
    waterSound.SoundId = "rbxassetid://WATERFALL_SOUND_ID"
    waterSound.Volume = 1.0
    waterSound.Looped = true
    waterSound.RollOffMaxDistance = 80
    waterSound.Parent = part
    waterSound:Play()
    return waterSound
end

-- เสียงลม (Ambient)
local windSound = Instance.new("Sound")
windSound.SoundId = "rbxassetid://WIND_SOUND_ID"
windSound.Volume = 0.3
windSound.Looped = true
windSound.Parent = SoundService  -- 2D Global
windSound:Play()

-- ระบบ Sound Zones
local SoundZone = {}
SoundZone.__index = SoundZone

function SoundZone.new(zonePart, soundId, volume)
    local self = setmetatable({
        zone = zonePart,
        sound = Instance.new("Sound"),
        playersInZone = {},
    }, SoundZone)
    
    self.sound.SoundId = "rbxassetid://" .. soundId
    self.sound.Volume = 0
    self.sound.Looped = true
    self.sound.Parent = SoundService
    
    local targetVolume = volume or 0.5
    
    -- ตรวจสอบผู้เล่นในโซน
    local Players = game:GetService("Players")
    local RunService = game:GetService("RunService")
    local TweenService = game:GetService("TweenService")
    
    RunService.Heartbeat:Connect(function()
        for _, player in ipairs(Players:GetPlayers()) do
            local character = player.Character
            if not character then continue end
            
            local hrp = character:FindFirstChild("HumanoidRootPart")
            if not hrp then continue end
            
            -- ตรวจสอบว่าอยู่ในโซนหรือไม่
            local isInZone = self:isInZone(hrp.Position)
            local wasInZone = self.playersInZone[player]
            
            if isInZone and not wasInZone then
                -- เข้าโซน
                self.playersInZone[player] = true
                TweenService:Create(self.sound, TweenInfo.new(1), {Volume = targetVolume}):Play()
                self.sound:Play()
            elseif not isInZone and wasInZone then
                -- ออกจากโซน
                self.playersInZone[player] = nil
                if not next(self.playersInZone) then
                    TweenService:Create(self.sound, TweenInfo.new(1), {Volume = 0}):Play()
                    task.delay(1, function() self.sound:Stop() end)
                end
            end
        end
    end)
    
    return self
end

function SoundZone:isInZone(position)
    local zone = self.zone
    local zonePos = zone.Position
    local zoneSize = zone.Size / 2
    
    return math.abs(position.X - zonePos.X) <= zoneSize.X and
           math.abs(position.Y - zonePos.Y) <= zoneSize.Y and
           math.abs(position.Z - zonePos.Z) <= zoneSize.Z
end
```

---

## 35.8 UI Sound Effects

```lua
-- LocalScript: UI Sounds
local SoundService = game:GetService("SoundService")

local UISounds = {}

-- Sound IDs (ใส่ IDs จริง)
local uiSoundIds = {
    click = "rbxassetid://CLICK_ID",
    hover = "rbxassetid://HOVER_ID",
    open = "rbxassetid://OPEN_ID",
    close = "rbxassetid://CLOSE_ID",
    success = "rbxassetid://SUCCESS_ID",
    error = "rbxassetid://ERROR_ID",
    notification = "rbxassetid://NOTIF_ID",
    levelUp = "rbxassetid://LEVELUP_ID",
    purchase = "rbxassetid://PURCHASE_ID",
    equip = "rbxassetid://EQUIP_ID",
}

-- สร้าง Sounds
local sounds = {}
for name, id in pairs(uiSoundIds) do
    local s = Instance.new("Sound")
    s.SoundId = id
    s.Volume = 0.7
    s.Parent = SoundService
    sounds[name] = s
end

function UISounds.play(name, volume, pitch)
    local s = sounds[name]
    if not s then return end
    
    -- Clone เพื่อเล่นซ้อนกันได้
    if s.IsPlaying then
        local clone = s:Clone()
        clone.Parent = SoundService
        if volume then clone.Volume = volume end
        if pitch then clone.Pitch = pitch end
        clone:Play()
        clone.Ended:Connect(function() clone:Destroy() end)
    else
        if volume then s.Volume = volume end
        if pitch then s.Pitch = pitch end
        s:Play()
    end
end

-- Auto-attach ให้กับ GUI
function UISounds.attachToButton(button)
    button.MouseEnter:Connect(function()
        UISounds.play("hover", 0.3)
    end)
    
    button.MouseButton1Click:Connect(function()
        UISounds.play("click")
    end)
end

-- ตัวอย่าง
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local screenGui = Instance.new("ScreenGui")
screenGui.Parent = LocalPlayer.PlayerGui

local btn = Instance.new("TextButton")
btn.Size = UDim2.new(0, 200, 0, 50)
btn.AnchorPoint = Vector2.new(0.5, 0.5)
btn.Position = UDim2.new(0.5, 0, 0.5, 0)
btn.Text = "กด (พร้อมเสียง)"
btn.Parent = screenGui

UISounds.attachToButton(btn)

return UISounds
```

---

## 35.9 Dynamic Music System

```lua
-- LocalScript: Dynamic Music ตามสถานการณ์
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local SoundService = game:GetService("SoundService")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer

-- Music Layers
local musicLayers = {
    base = Instance.new("Sound"),       -- เพลงพื้นฐาน
    combat = Instance.new("Sound"),     -- เพลงสู้รบ
    danger = Instance.new("Sound"),     -- เพลงอันตราย
    victory = Instance.new("Sound"),    -- เพลงชนะ
}

-- ตั้งค่า
musicLayers.base.SoundId = "rbxassetid://BASE_MUSIC"
musicLayers.base.Looped = true
musicLayers.base.Volume = 0.5

musicLayers.combat.SoundId = "rbxassetid://COMBAT_MUSIC"
musicLayers.combat.Looped = true
musicLayers.combat.Volume = 0

musicLayers.danger.SoundId = "rbxassetid://DANGER_MUSIC"
musicLayers.danger.Looped = true
musicLayers.danger.Volume = 0

musicLayers.victory.SoundId = "rbxassetid://VICTORY_MUSIC"
musicLayers.victory.Looped = false
musicLayers.victory.Volume = 0

for _, sound in pairs(musicLayers) do
    sound.Parent = SoundService
    sound:Play()
end

-- สถานะปัจจุบัน
local currentState = "peaceful"

local function setMusicState(state)
    if state == currentState then return end
    currentState = state
    
    -- ตั้งค่า Target Volumes
    local targets = {
        peaceful = {base = 0.5, combat = 0, danger = 0},
        combat = {base = 0.2, combat = 0.7, danger = 0},
        danger = {base = 0.1, combat = 0.3, danger = 0.7},
        victory = {base = 0, combat = 0, danger = 0},
    }
    
    local target = targets[state] or targets.peaceful
    
    for layerName, volume in pairs(target) do
        TweenService:Create(
            musicLayers[layerName],
            TweenInfo.new(2, Enum.EasingStyle.Sine),
            {Volume = volume}
        ):Play()
    end
    
    if state == "victory" then
        musicLayers.victory.Volume = 0.8
        musicLayers.victory:Play()
    end
end

-- ตรวจสอบ HP สำหรับ Danger Music
RunService.Heartbeat:Connect(function()
    local character = LocalPlayer.Character
    if not character then return end
    
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end
    
    local healthPct = humanoid.Health / humanoid.MaxHealth
    
    if healthPct <= 0.2 then
        setMusicState("danger")
    end
end)

-- API
return {
    setState = setMusicState,
    getCurrentState = function() return currentState end
}
```

---

## 35.10 แบบฝึกหัด

### แบบฝึกหัดที่ 1: สร้าง Music Player UI

```lua
-- LocalScript: Music Player UI
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local SoundService = game:GetService("SoundService")

local LocalPlayer = Players.LocalPlayer

local screenGui = Instance.new("ScreenGui")
screenGui.Parent = LocalPlayer.PlayerGui

-- Playlist
local tracks = {
    {title = "เพลงที่ 1", artist = "Artist A", id = "rbxassetid://111"},
    {title = "เพลงที่ 2", artist = "Artist B", id = "rbxassetid://222"},
    {title = "เพลงที่ 3", artist = "Artist C", id = "rbxassetid://333"},
}

local currentTrack = 1
local isPlaying = false

-- Sound
local player = Instance.new("Sound")
player.Parent = SoundService

-- Player UI
local playerFrame = Instance.new("Frame")
playerFrame.Size = UDim2.new(0, 300, 0, 120)
playerFrame.Position = UDim2.new(0.5, -150, 1, -130)
playerFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
playerFrame.BorderSizePixel = 0
playerFrame.Parent = screenGui

local pCorner = Instance.new("UICorner")
pCorner.CornerRadius = UDim.new(0, 12)
pCorner.Parent = playerFrame

-- Track Info
local trackTitle = Instance.new("TextLabel")
trackTitle.Size = UDim2.new(1, -20, 0, 25)
trackTitle.Position = UDim2.new(0, 10, 0, 10)
trackTitle.BackgroundTransparency = 1
trackTitle.Text = tracks[currentTrack].title
trackTitle.TextColor3 = Color3.new(1, 1, 1)
trackTitle.Font = Enum.Font.GothamBold
trackTitle.TextSize = 16
trackTitle.TextXAlignment = Enum.TextXAlignment.Left
trackTitle.Parent = playerFrame

local artistName = Instance.new("TextLabel")
artistName.Size = UDim2.new(1, -20, 0, 20)
artistName.Position = UDim2.new(0, 10, 0, 35)
artistName.BackgroundTransparency = 1
artistName.Text = tracks[currentTrack].artist
artistName.TextColor3 = Color3.fromRGB(150, 150, 180)
artistName.Font = Enum.Font.Gotham
artistName.TextSize = 13
artistName.TextXAlignment = Enum.TextXAlignment.Left
artistName.Parent = playerFrame

-- Progress
local progressBg = Instance.new("Frame")
progressBg.Size = UDim2.new(1, -20, 0, 4)
progressBg.Position = UDim2.new(0, 10, 0, 62)
progressBg.BackgroundColor3 = Color3.fromRGB(50, 50, 70)
progressBg.BorderSizePixel = 0
progressBg.Parent = playerFrame

local progressBar = Instance.new("Frame")
progressBar.Size = UDim2.new(0, 0, 1, 0)
progressBar.BackgroundColor3 = Color3.fromRGB(100, 150, 255)
progressBar.BorderSizePixel = 0
progressBar.Parent = progressBg

-- Controls
local controlsFrame = Instance.new("Frame")
controlsFrame.Size = UDim2.new(1, 0, 0, 40)
controlsFrame.Position = UDim2.new(0, 0, 1, -45)
controlsFrame.BackgroundTransparency = 1
controlsFrame.Parent = playerFrame

local btnLayout = Instance.new("UIListLayout")
btnLayout.FillDirection = Enum.FillDirection.Horizontal
btnLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
btnLayout.Padding = UDim.new(0, 15)
btnLayout.VerticalAlignment = Enum.VerticalAlignment.Center
btnLayout.Parent = controlsFrame

local function makeCtrlBtn(text)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 36, 0, 36)
    btn.BackgroundTransparency = 1
    btn.Text = text
    btn.TextSize = 22
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.Parent = controlsFrame
    return btn
end

local prevBtn = makeCtrlBtn("⏮")
local playBtn = makeCtrlBtn("▶")
local nextBtn = makeCtrlBtn("⏭")

-- ฟังก์ชัน
local function updateTrackInfo()
    trackTitle.Text = tracks[currentTrack].title
    artistName.Text = tracks[currentTrack].artist
end

local function playTrack()
    player.SoundId = tracks[currentTrack].id
    player:Play()
    isPlaying = true
    playBtn.Text = "⏸"
end

local function pauseTrack()
    player:Pause()
    isPlaying = false
    playBtn.Text = "▶"
end

playBtn.MouseButton1Click:Connect(function()
    if isPlaying then
        pauseTrack()
    else
        if player.IsPaused then
            player:Resume()
            isPlaying = true
            playBtn.Text = "⏸"
        else
            playTrack()
        end
    end
end)

nextBtn.MouseButton1Click:Connect(function()
    currentTrack = (currentTrack % #tracks) + 1
    updateTrackInfo()
    playTrack()
end)

prevBtn.MouseButton1Click:Connect(function()
    currentTrack = ((currentTrack - 2) % #tracks) + 1
    updateTrackInfo()
    playTrack()
end)

-- อัพเดต Progress Bar
local RunService = game:GetService("RunService")
RunService.Heartbeat:Connect(function()
    if player.IsPlaying and player.TimeLength > 0 then
        local pct = player.TimePosition / player.TimeLength
        TweenService:Create(progressBar, TweenInfo.new(0.1), {
            Size = UDim2.new(pct, 0, 1, 0)
        }):Play()
    end
end)

-- เล่นเพลงแรก
playTrack()
```

---

## 35.11 สรุป

ในบทนี้เราได้เรียนรู้:

1. **Sound Basics** - Play, Stop, Pause, Resume
2. **SoundService Properties** - MasterVolume, AmbientReverb
3. **SoundGroup** - จัดการเสียงเป็น Category
4. **Sound Manager** - ระบบจัดการเสียงส่วนกลาง
5. **Background Music** - ระบบเพลงประกอบพร้อม Fade
6. **Sound Effects Pool** - Pooling สำหรับ SFX
7. **Footstep System** - เสียงก้าวเดินตามพื้นผิว
8. **3D Spatial Sound** - เสียงตำแหน่งใน 3D
9. **UI Sounds** - เสียงสำหรับ UI
10. **Dynamic Music** - เปลี่ยนเพลงตามสถานการณ์

ในบทต่อไป เราจะเรียนรู้เกี่ยวกับ Lighting Basics

---

## แหล่งอ้างอิง

- [Roblox Developer Hub - Sound](https://developer.roblox.com/en-us/api-reference/class/Sound)
- [Roblox Developer Hub - SoundService](https://developer.roblox.com/en-us/api-reference/class/SoundService)
- [Roblox Developer Hub - SoundGroup](https://developer.roblox.com/en-us/api-reference/class/SoundGroup)
