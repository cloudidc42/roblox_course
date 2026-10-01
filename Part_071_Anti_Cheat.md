# Part 71: Basic Anti-Cheat Systems

## บทนำ

ระบบ Anti-Cheat เป็นสิ่งจำเป็นสำหรับเกม Roblox ทุกเกม เนื่องจากมีผู้เล่นที่พยายามโกงอยู่เสมอ ในบทนี้เราจะเรียนรู้วิธีป้องกันการโกงในรูปแบบต่างๆ

> **หมายเหตุสำคัญ**: Anti-Cheat ที่ดีที่สุดคือ Server-Side Validation ทุกอย่างที่ Client ส่งมาต้องได้รับการตรวจสอบบน Server

---

## 71.1 หลักการ Anti-Cheat

### กฎทอง: "Never Trust The Client"

```
❌ อย่าเชื่อ Client:
- ตำแหน่งที่ Client บอก
- ค่า stats ที่ Client ส่งมา
- ผลการยิงที่ Client รายงาน

✓ ทำบน Server เสมอ:
- คำนวณ damage
- ตรวจสอบตำแหน่ง
- อัพเดทข้อมูล
```

---

## 71.2 Speed Hack Detection

```lua
-- ServerScriptService/AntiCheat/SpeedHackDetector.lua
-- ตรวจจับการแก้ไขความเร็ว

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local SpeedHackDetector = {}

-- เก็บประวัติ position
local playerHistory = {}
local violations = {}
local VIOLATION_THRESHOLD = 3  -- โดน 3 ครั้ง = โกง
local MAX_SPEED = 80           -- ความเร็วสูงสุดที่ยอมรับ (studs/s)

-- เริ่มติดตาม
local function startTracking(player)
    playerHistory[player.UserId] = {
        positions = {},
        lastPos = nil,
        lastTime = 0
    }
    violations[player.UserId] = 0
end

-- คำนวณความเร็ว
local function calculateSpeed(pos1, pos2, deltaTime)
    if deltaTime <= 0 then return 0 end
    return (pos2 - pos1).Magnitude / deltaTime
end

-- ตรวจสอบทุก heartbeat
local function checkPlayer(player, dt)
    local character = player.Character
    if not character or not character:FindFirstChild("HumanoidRootPart") then return end
    
    local humanoid = character:FindFirstChildWhichIsA("Humanoid")
    if not humanoid then return end
    
    local history = playerHistory[player.UserId]
    if not history then return end
    
    local currentPos = character.HumanoidRootPart.Position
    local currentTime = tick()
    
    if history.lastPos and history.lastTime > 0 then
        local deltaTime = currentTime - history.lastTime
        local speed = calculateSpeed(history.lastPos, currentPos, deltaTime)
        
        -- ตรวจสอบความเร็ว
        local expectedMaxSpeed = humanoid.WalkSpeed * 1.5  -- เผื่อ buffer 50%
        local absoluteMax = math.max(MAX_SPEED, expectedMaxSpeed)
        
        if speed > absoluteMax then
            violations[player.UserId] = violations[player.UserId] + 1
            
            warn(string.format("[Anti-Cheat] Speed Violation: %s - Speed: %.1f studs/s (Max: %.1f)",
                player.Name, speed, absoluteMax))
            
            -- ส่งกลับไปตำแหน่งเดิม (rubber-band)
            if violations[player.UserId] >= VIOLATION_THRESHOLD then
                character.HumanoidRootPart.CFrame = CFrame.new(history.lastPos)
                warn("[Anti-Cheat] Teleported " .. player.Name .. " back - Speed Hack detected!")
                
                -- Log ลง DataStore
                logViolation(player, "speed_hack", {
                    speed = speed,
                    maxAllowed = absoluteMax,
                    position = currentPos
                })
            end
        else
            -- ลด violation count เมื่อเล่นปกติ
            if violations[player.UserId] > 0 then
                violations[player.UserId] = math.max(0, violations[player.UserId] - 0.1)
            end
        end
    end
    
    -- บันทึก position
    history.lastPos = currentPos
    history.lastTime = currentTime
    
    table.insert(history.positions, {pos = currentPos, time = currentTime})
    if #history.positions > 60 then
        table.remove(history.positions, 1)
    end
end

-- RunService Loop
RunService.Heartbeat:Connect(function(dt)
    for _, player in ipairs(Players:GetPlayers()) do
        if playerHistory[player.UserId] then
            checkPlayer(player, dt)
        end
    end
end)

Players.PlayerAdded:Connect(startTracking)
Players.PlayerRemoving:Connect(function(player)
    playerHistory[player.UserId] = nil
    violations[player.UserId] = nil
end)

return SpeedHackDetector
```

---

## 71.3 Teleport Hack Detection

```lua
-- ServerScriptService/AntiCheat/TeleportDetector.lua
-- ตรวจจับการ teleport ผิดกฎ

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local MAX_TELEPORT_DISTANCE = 30  -- studs ต่อ tick ที่ยอมรับ

local lastPositions = {}

RunService.Heartbeat:Connect(function()
    for _, player in ipairs(Players:GetPlayers()) do
        local character = player.Character
        if not character or not character:FindFirstChild("HumanoidRootPart") then continue end
        
        local currentPos = character.HumanoidRootPart.Position
        local lastPos = lastPositions[player.UserId]
        
        if lastPos then
            local distance = (currentPos - lastPos).Magnitude
            
            if distance > MAX_TELEPORT_DISTANCE then
                -- อาจเป็น teleport hack
                -- แต่ต้องตรวจสอบก่อนว่าไม่ใช่ legitimate teleport
                local isLegitTeleport = player:GetAttribute("LegitTeleport") or false
                
                if not isLegitTeleport then
                    warn("[Anti-Cheat] Teleport detected: " .. player.Name ..
                        " - Distance: " .. math.floor(distance) .. " studs")
                    
                    -- ส่งกลับ
                    character.HumanoidRootPart.CFrame = CFrame.new(lastPos)
                end
            end
        end
        
        lastPositions[player.UserId] = currentPos
        player:SetAttribute("LegitTeleport", false)  -- reset
    end
end)

-- ฟังก์ชันสำหรับทำ legitimate teleport
local function legitimateTeleport(player, position)
    player:SetAttribute("LegitTeleport", true)
    
    local character = player.Character
    if character and character:FindFirstChild("HumanoidRootPart") then
        character.HumanoidRootPart.CFrame = CFrame.new(position)
        lastPositions[player.UserId] = position
    end
    
    task.wait(0.1)
    player:SetAttribute("LegitTeleport", false)
end

return {legitimateTeleport = legitimateTeleport}
```

---

## 71.4 Remote Event Rate Limiting

```lua
-- ServerScriptService/AntiCheat/RateLimiter.lua
-- จำกัดความถี่ในการส่ง Remote Events

local Players = game:GetService("Players")

local RateLimiter = {}

local limits = {}      -- เก็บ timestamps ของแต่ละ event
local violations = {}

-- กำหนด Rate Limits
local rateLimits = {
    FireWeapon = {maxPerSecond = 20, cooldown = 0.05},
    Mine = {maxPerSecond = 10, cooldown = 0.1},
    PlaceTower = {maxPerSecond = 2, cooldown = 0.5},
    BuyUpgrade = {maxPerSecond = 5, cooldown = 0.2},
    Chat = {maxPerSecond = 3, cooldown = 0.3},
    CustomChat = {maxPerSecond = 2, cooldown = 0.5},
}

-- ตรวจสอบ rate limit
function RateLimiter.check(player, eventName)
    local limit = rateLimits[eventName]
    if not limit then return true end  -- ถ้าไม่มี limit ยอมรับ
    
    local userId = player.UserId
    
    if not limits[userId] then
        limits[userId] = {}
    end
    
    if not limits[userId][eventName] then
        limits[userId][eventName] = {timestamps = {}, lastViolation = 0}
    end
    
    local eventData = limits[userId][eventName]
    local now = tick()
    
    -- ตรวจสอบ cooldown
    local lastTimestamp = eventData.timestamps[#eventData.timestamps]
    if lastTimestamp and (now - lastTimestamp) < limit.cooldown then
        -- ละเมิด cooldown
        eventData.lastViolation = now
        return false, "cooldown"
    end
    
    -- ลบ timestamps เก่ากว่า 1 วินาที
    local validTimestamps = {}
    for _, ts in ipairs(eventData.timestamps) do
        if now - ts < 1 then
            table.insert(validTimestamps, ts)
        end
    end
    eventData.timestamps = validTimestamps
    
    -- ตรวจสอบจำนวนต่อวินาที
    if #eventData.timestamps >= limit.maxPerSecond then
        -- ละเมิด rate limit
        if not violations[userId] then violations[userId] = {} end
        if not violations[userId][eventName] then violations[userId][eventName] = 0 end
        violations[userId][eventName] = violations[userId][eventName] + 1
        
        warn(string.format("[RateLimit] %s violated %s limit (%d violations)",
            player.Name, eventName, violations[userId][eventName]))
        
        -- ถ้าละเมิดมาก อาจ kick
        if violations[userId][eventName] >= 50 then
            warn("[Anti-Cheat] Auto-kick: " .. player.Name .. " - Rate limit abuse on " .. eventName)
            -- player:Kick("กรุณาไม่โกง")
        end
        
        return false, "rate_limit"
    end
    
    -- อนุญาต
    table.insert(eventData.timestamps, now)
    return true
end

-- Clean up เมื่อผู้เล่นออก
Players.PlayerRemoving:Connect(function(player)
    limits[player.UserId] = nil
    violations[player.UserId] = nil
end)

return RateLimiter
```

---

## 71.5 Stat Validation

```lua
-- ServerScriptService/AntiCheat/StatValidator.lua
-- ตรวจสอบค่า stats ที่ผิดปกติ

local Players = game:GetService("Players")

local StatValidator = {}

-- กำหนดขอบเขตที่ยอมรับ
local statBounds = {
    WalkSpeed = {min = 0, max = 50},       -- ความเร็วปกติ
    JumpPower = {min = 0, max = 100},       -- Jump power ปกติ
    Health = {min = 0, max = 10000},        -- Health สูงสุด
    Gold = {min = 0, max = 1e12},           -- Gold สูงสุด
}

-- ตรวจสอบค่า stat
function StatValidator.validateStat(statName, value)
    local bounds = statBounds[statName]
    if not bounds then return true end  -- ไม่มีข้อกำหนด
    
    if type(value) ~= "number" then return false end
    if value ~= value then return false end  -- NaN check
    if value == math.huge or value == -math.huge then return false end  -- Infinity check
    
    return value >= bounds.min and value <= bounds.max
end

-- ตรวจสอบ Humanoid stats
local function checkHumanoidStats(player)
    local character = player.Character
    if not character then return end
    
    local humanoid = character:FindFirstChildWhichIsA("Humanoid")
    if not humanoid then return end
    
    -- ตรวจสอบ WalkSpeed
    if not StatValidator.validateStat("WalkSpeed", humanoid.WalkSpeed) then
        warn("[Anti-Cheat] Invalid WalkSpeed for " .. player.Name .. ": " .. humanoid.WalkSpeed)
        humanoid.WalkSpeed = 16  -- reset ค่า default
    end
    
    -- ตรวจสอบ JumpPower
    if not StatValidator.validateStat("JumpPower", humanoid.JumpPower) then
        warn("[Anti-Cheat] Invalid JumpPower for " .. player.Name .. ": " .. humanoid.JumpPower)
        humanoid.JumpPower = 50  -- reset ค่า default
    end
    
    -- ตรวจสอบ Health
    if not StatValidator.validateStat("Health", humanoid.Health) then
        warn("[Anti-Cheat] Invalid Health for " .. player.Name .. ": " .. humanoid.Health)
        humanoid.Health = 0  -- ทำให้ตาย
    end
    
    -- MaxHealth ต้องตรงกับที่ Server กำหนด
    local expectedMaxHealth = player:GetAttribute("MaxHealth") or 100
    if math.abs(humanoid.MaxHealth - expectedMaxHealth) > 0.1 then
        warn("[Anti-Cheat] MaxHealth mismatch for " .. player.Name ..
            ": " .. humanoid.MaxHealth .. " vs expected " .. expectedMaxHealth)
        humanoid.MaxHealth = expectedMaxHealth
        humanoid.Health = math.min(humanoid.Health, expectedMaxHealth)
    end
end

-- ตรวจสอบทุก 2 วินาที
task.spawn(function()
    while true do
        task.wait(2)
        for _, player in ipairs(Players:GetPlayers()) do
            checkHumanoidStats(player)
        end
    end
end)

return StatValidator
```

---

## 71.6 Exploit Detection

```lua
-- ServerScriptService/AntiCheat/ExploitDetector.lua
-- ตรวจจับ Exploit ทั่วไป

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local ExploitDetector = {}

-- ตรวจสอบ LocalScript ที่ผิดปกติ (ไม่ตรงกับที่คาดหวัง)
-- หมายเหตุ: ใน Roblox เราไม่สามารถ "scan" scripts ของ Client ได้โดยตรง
-- แต่เราสามารถ detect พฤติกรรมผิดปกติได้

-- Heartbeat Checker: ตรวจสอบว่า Client ยังส่ง heartbeat ปกติ
local heartbeatData = {}
local HEARTBEAT_INTERVAL = 5  -- วินาที
local HEARTBEAT_TIMEOUT = 30  -- วินาที

local heartbeatRemote = Instance.new("RemoteEvent")
heartbeatRemote.Name = "ClientHeartbeat"
heartbeatRemote.Parent = ReplicatedStorage.Remotes

-- รับ heartbeat จาก Client
heartbeatRemote.OnServerEvent:Connect(function(player, clientData)
    local now = tick()
    
    if not heartbeatData[player.UserId] then
        heartbeatData[player.UserId] = {}
    end
    
    local data = heartbeatData[player.UserId]
    
    -- ตรวจสอบ interval ของ heartbeat
    if data.lastHeartbeat then
        local interval = now - data.lastHeartbeat
        
        -- ถ้า interval ผิดปกติมาก
        if interval < HEARTBEAT_INTERVAL * 0.5 then
            -- Heartbeat เร็วเกินไป อาจเป็นการส่ง flood
            data.floodCount = (data.floodCount or 0) + 1
            if data.floodCount >= 10 then
                warn("[Anti-Cheat] Heartbeat flood from " .. player.Name)
            end
        end
    end
    
    data.lastHeartbeat = now
    data.clientData = clientData
    
    -- ตรวจสอบ client data
    if clientData then
        -- ตรวจสอบ FPS ที่ไม่ปกติ (อาจเป็น speed hack)
        if clientData.fps and clientData.fps > 1000 then
            warn("[Anti-Cheat] Suspicious FPS from " .. player.Name .. ": " .. clientData.fps)
        end
    end
end)

-- ตรวจสอบ Client ที่ไม่ส่ง heartbeat
task.spawn(function()
    while true do
        task.wait(HEARTBEAT_INTERVAL)
        
        local now = tick()
        for _, player in ipairs(Players:GetPlayers()) do
            local data = heartbeatData[player.UserId]
            
            if data and data.lastHeartbeat then
                local timeSince = now - data.lastHeartbeat
                
                if timeSince > HEARTBEAT_TIMEOUT then
                    warn("[Anti-Cheat] " .. player.Name .. " missed heartbeat for " ..
                        math.floor(timeSince) .. "s - possible exploit")
                end
            end
        end
    end
end)

-- ตรวจสอบ Noclip
local function checkNoclip(player)
    local character = player.Character
    if not character or not character:FindFirstChild("HumanoidRootPart") then return end
    
    -- ยิง ray จาก position ปัจจุบันลงพื้น
    local rootPos = character.HumanoidRootPart.Position
    local ray = workspace:Raycast(rootPos, Vector3.new(0, -10, 0))
    
    -- ถ้าไม่มีพื้นใต้เท้าในระยะ 10 studs และไม่ได้กระโดด
    local humanoid = character:FindFirstChildWhichIsA("Humanoid")
    if humanoid and humanoid.FloorMaterial == Enum.Material.Air then
        -- อาจเป็น noclip - แต่ต้องตรวจสอบด้วยว่ากำลัง jump หรือ fall อยู่หรือเปล่า
        local velocity = character.HumanoidRootPart.Velocity
        if math.abs(velocity.Y) < 1 and not ray then
            -- ลอยอยู่กลางอากาศ
            warn("[Anti-Cheat] Possible noclip/fly: " .. player.Name)
        end
    end
end

return ExploitDetector
```

---

## 71.7 Logging System

```lua
-- ServerScriptService/AntiCheat/ViolationLogger.lua
-- ระบบ Log การโกง

local DataStoreService = game:GetService("DataStoreService")
local Players = game:GetService("Players")

local violationStore = DataStoreService:GetDataStore("ViolationLog_v1")

local ViolationLogger = {}

-- Log การโกง
function ViolationLogger.log(player, violationType, details)
    local logEntry = {
        timestamp = os.time(),
        playerName = player.Name,
        userId = player.UserId,
        violationType = violationType,
        details = details,
        serverId = game.JobId
    }
    
    -- Print ใน console
    warn(string.format("[VIOLATION] %s | %s | %s | %s",
        os.date("%Y-%m-%d %H:%M:%S", logEntry.timestamp),
        player.Name,
        violationType,
        game:GetService("HttpService"):JSONEncode(details or {})
    ))
    
    -- บันทึกลง DataStore
    local success, err = pcall(function()
        local key = "violations_" .. player.UserId
        
        -- อ่านข้อมูลเดิม
        local existing = violationStore:GetAsync(key) or {violations = {}, totalCount = 0}
        
        table.insert(existing.violations, logEntry)
        existing.totalCount = existing.totalCount + 1
        
        -- เก็บแค่ 100 violations ล่าสุด
        if #existing.violations > 100 then
            table.remove(existing.violations, 1)
        end
        
        violationStore:SetAsync(key, existing)
    end)
    
    if not success then
        warn("ไม่สามารถ log violation: " .. err)
    end
    
    return logEntry
end

-- ดูประวัติการโกง
function ViolationLogger.getHistory(userId)
    local success, data = pcall(function()
        return violationStore:GetAsync("violations_" .. userId)
    end)
    
    if success and data then
        return data
    end
    return {violations = {}, totalCount = 0}
end

-- ตรวจสอบว่าผู้เล่นเคยโกงหรือไม่
function ViolationLogger.isHighRisk(userId)
    local history = ViolationLogger.getHistory(userId)
    return history.totalCount >= 10  -- 10+ violations = high risk
end

-- Action จาก violation count
function ViolationLogger.takeAction(player, violationType)
    local history = ViolationLogger.getHistory(player.UserId)
    local count = history.totalCount
    
    if count >= 50 then
        -- Ban
        warn("[Anti-Cheat] AUTO-BAN: " .. player.Name .. " - " .. count .. " violations")
        player:Kick("คุณถูก Ban เนื่องจากโกงเกม")
    elseif count >= 20 then
        -- Warn + Kick
        warn("[Anti-Cheat] KICK: " .. player.Name)
        player:Kick("ตรวจพบการโกง - ครั้งที่ " .. count)
    elseif count >= 10 then
        -- เตือน
        local notifyRemote = game.ReplicatedStorage.Remotes:FindFirstChild("Notification")
        if notifyRemote then
            notifyRemote:FireClient(player, "⚠️ ระบบตรวจพบพฤติกรรมผิดปกติ", "warning")
        end
    end
end

-- Log ทั่วไปสำหรับ Debug
function ViolationLogger.info(message)
    print("[Anti-Cheat] " .. message)
end

return ViolationLogger
```

---

## 71.8 Main Anti-Cheat Module

```lua
-- ServerScriptService/AntiCheat.lua
-- รวมทุก Anti-Cheat ระบบเข้าด้วยกัน

local AntiCheat = {}

-- โหลด modules
local SpeedDetector = require(script.SpeedHackDetector)
local TeleportDetector = require(script.TeleportDetector)
local RateLimiter = require(script.RateLimiter)
local StatValidator = require(script.StatValidator)
local ViolationLogger = require(script.ViolationLogger)

-- Wrapper สำหรับตรวจสอบ Remote Event
function AntiCheat.validateRemote(player, eventName, ...)
    -- ตรวจสอบ Rate Limit
    local allowed, reason = RateLimiter.check(player, eventName)
    
    if not allowed then
        ViolationLogger.log(player, "rate_limit_" .. eventName, {reason = reason})
        
        if reason == "rate_limit" then
            ViolationLogger.takeAction(player, "rate_limit")
        end
        
        return false
    end
    
    return true
end

-- ฟังก์ชันตรวจสอบ Position
function AntiCheat.validatePosition(player, claimedPosition, maxDistance)
    local character = player.Character
    if not character or not character:FindFirstChild("HumanoidRootPart") then
        return false
    end
    
    local actualPos = character.HumanoidRootPart.Position
    local distance = (actualPos - claimedPosition).Magnitude
    
    maxDistance = maxDistance or 15  -- default 15 studs
    
    if distance > maxDistance then
        ViolationLogger.log(player, "invalid_position", {
            claimed = claimedPosition,
            actual = actualPos,
            distance = distance
        })
        return false
    end
    
    return true
end

-- ตรวจสอบ Stat เมื่อ Client ส่งมา
function AntiCheat.validateClientStat(player, statName, value)
    if not StatValidator.validateStat(statName, value) then
        ViolationLogger.log(player, "invalid_stat_" .. statName, {value = value})
        return false
    end
    return true
end

print("[Anti-Cheat] ระบบป้องกันการโกงเริ่มทำงานแล้ว")
return AntiCheat
```

---

## 71.9 การใช้งาน Anti-Cheat ใน Remote Events

```lua
-- ตัวอย่างการใช้ Anti-Cheat ใน Weapon System
local AntiCheat = require(ServerScriptService.AntiCheat)

fireRemote.OnServerEvent:Connect(function(player, weaponId, startPos, directions)
    -- 1. ตรวจสอบ Rate Limit
    if not AntiCheat.validateRemote(player, "FireWeapon") then
        return  -- ปฏิเสธ
    end
    
    -- 2. ตรวจสอบ Position
    if not AntiCheat.validatePosition(player, startPos, 10) then
        return  -- ปฏิเสธ
    end
    
    -- 3. ตรวจสอบ Weapon ที่ถือ
    local character = player.Character
    local hasWeapon = false
    if character then
        for _, tool in ipairs(character:GetChildren()) do
            if tool:IsA("Tool") and tool:GetAttribute("WeaponId") == weaponId then
                hasWeapon = true
                break
            end
        end
    end
    
    if not hasWeapon then
        warn("[Anti-Cheat] " .. player.Name .. " ยิงอาวุธที่ไม่ได้ถือ: " .. weaponId)
        return
    end
    
    -- 4. ตรวจสอบ Direction validity
    for _, dir in ipairs(directions) do
        if type(dir) ~= "Vector3" or dir.Magnitude == 0 then
            warn("[Anti-Cheat] " .. player.Name .. " ส่ง direction ผิดรูปแบบ")
            return
        end
    end
    
    -- ผ่านการตรวจสอบทั้งหมด - ดำเนินการต่อ
    processBullet(player, weaponId, startPos, directions)
end)
```

---

## 71.10 ข้อผิดพลาดที่พบบ่อย

```lua
-- ❌ ผิด: Kick ผู้เล่นทันทีเมื่อตรวจพบ
if speedDetected then
    player:Kick("Speed Hack")  -- อาจเกิด false positive!
end

-- ✓ ถูก: สะสม violations ก่อน Action
local KICK_THRESHOLD = 5
if violations[player.UserId] >= KICK_THRESHOLD then
    -- Log และ Kick เฉพาะเมื่อแน่ใจ
    ViolationLogger.log(player, "auto_kick", violations[player.UserId])
    player:Kick("ตรวจพบการโกง")
end
```

---

## 71.11 แบบฝึกหัด

### แบบฝึกหัดที่ 1: Fly Hack Detection
สร้างระบบตรวจจับ Fly Hack:
- ตรวจสอบว่า Y velocity ผิดปกติหรือไม่
- ผู้เล่นอยู่กลางอากาศนานเกินไปหรือเปล่า

### แบบฝึกหัดที่ 2: God Mode Detection
ตรวจสอบว่าผู้เล่นรับ damage แล้ว HP ไม่ลดหรือเปล่า

### แบบฝึกหัดที่ 3: Item Duplication Detection
ตรวจสอบว่า inventory มี item มากกว่าที่ควรหรือไม่

---

## สรุป

ในบทนี้เราได้เรียนรู้:
- หลักการ "Never Trust The Client"
- Speed Hack Detection
- Teleport Detection
- Rate Limiting
- Stat Validation
- Violation Logging

ในบทถัดไปเราจะเรียนรู้ Performance Optimization!
