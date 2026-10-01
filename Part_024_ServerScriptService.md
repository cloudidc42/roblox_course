# ตอนที่ 24: ServerScriptService ใน Roblox

## บทนำ

ServerScriptService เป็นสถานที่สำหรับเก็บ Scripts ที่ทำงานบน Server เท่านั้น Scripts ใน ServerScriptService จะไม่ถูกส่งไปยัง Client ทำให้ปลอดภัยสำหรับเก็บ logic ที่สำคัญเช่น การจัดการข้อมูล การตรวจสอบความถูกต้อง และ game logic หลัก

---

## 24.1 ทำไมต้องใช้ ServerScriptService?

```
เปรียบเทียบ Locations สำหรับ Server Scripts:

ServerScriptService (แนะนำ)
✓ Scripts ไม่ถูกส่งไปยัง Client
✓ ปลอดภัย ไม่ถูก exploit
✓ เหมาะสำหรับ game logic, data management
✓ Scripts ทำงานอัตโนมัติ

Workspace (ไม่แนะนำ)
✗ Scripts อาจถูกดู (เปิดเผย logic)
✗ ไม่เหมาะสำหรับ sensitive code
✓ ใช้สำหรับ scripts ที่ต้องอยู่ใกล้ object
```

---

## 24.2 โครงสร้างที่แนะนำ

```
ServerScriptService/
├── GameManager.lua          (Main game loop)
├── PlayerManager.lua        (จัดการผู้เล่น)
├── DataManager.lua          (DataStore)
├── CombatSystem.lua         (ระบบต่อสู้)
├── EnemySpawner.lua         (spawn ศัตรู)
└── Modules/
    ├── Logger.lua           (logging)
    └── Validator.lua        (validation)
```

---

## 24.3 Server-Side Validation

สิ่งสำคัญที่สุดใน ServerScriptService คือการ **validate** ทุก request จาก Client:

```lua
-- Script: InputValidator.lua

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local Validator = {}

-- ตรวจสอบว่า player มีอยู่จริง
function Validator.isValidPlayer(player)
    if not player then return false, "ไม่มีผู้เล่น" end
    if not player.Parent then return false, "ผู้เล่นออกจากเกมแล้ว" end
    return true, nil
end

-- ตรวจสอบว่า character มีชีวิต
function Validator.isAlive(player)
    local ok, err = Validator.isValidPlayer(player)
    if not ok then return false, err end
    
    local character = player.Character
    if not character then return false, "ไม่มี Character" end
    
    local humanoid = character:FindFirstChild("Humanoid")
    if not humanoid then return false, "ไม่มี Humanoid" end
    
    if humanoid.Health <= 0 then return false, "ตายแล้ว" end
    
    return true, nil
end

-- ตรวจสอบตัวเลข
function Validator.isValidNumber(value, min, max)
    if type(value) ~= "number" then
        return false, "ค่าต้องเป็นตัวเลข"
    end
    if value ~= value then  -- NaN check
        return false, "ค่า NaN ไม่ถูกต้อง"
    end
    if math.abs(value) == math.huge then
        return false, "ค่าต้องไม่เป็น infinity"
    end
    if min and value < min then
        return false, "ค่าต้องมากกว่า " .. min
    end
    if max and value > max then
        return false, "ค่าต้องน้อยกว่า " .. max
    end
    return true, nil
end

-- ตรวจสอบ string
function Validator.isValidString(str, minLen, maxLen)
    if type(str) ~= "string" then
        return false, "ค่าต้องเป็น string"
    end
    local len = utf8.len(str) or #str  -- นับ UTF-8 characters
    if minLen and len < minLen then
        return false, "ความยาวน้อยกว่า " .. minLen
    end
    if maxLen and len > maxLen then
        return false, "ความยาวมากกว่า " .. maxLen
    end
    return true, nil
end

-- ตรวจสอบระยะห่าง (ป้องกัน teleport hack)
function Validator.isInRange(player, targetPosition, maxDistance)
    local character = player.Character
    if not character then return false, "ไม่มี Character" end
    
    local hrp = character:FindFirstChild("HumanoidRootPart")
    if not hrp then return false, "ไม่มี HumanoidRootPart" end
    
    local distance = (hrp.Position - targetPosition).Magnitude
    if distance > maxDistance then
        return false, "ระยะไกลเกินไป (" .. math.floor(distance) .. ")"
    end
    
    return true, nil
end

return Validator
```

---

## 24.4 ระบบ Anti-Cheat พื้นฐาน

```lua
-- Script: AntiCheat.lua

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local AntiCheat = {}

-- ติดตาม speed ของผู้เล่น
local playerTracker = {}

local MAX_SPEED = 100  -- studs/second สูงสุดที่ยอมรับ
local CHECK_INTERVAL = 1  -- ตรวจสอบทุก 1 วินาที
local MAX_VIOLATIONS = 3  -- violations ก่อน kick

Players.PlayerAdded:Connect(function(player)
    playerTracker[player.UserId] = {
        lastPosition = nil,
        violations = 0
    }
    
    player.CharacterAdded:Connect(function(character)
        local hrp = character:WaitForChild("HumanoidRootPart")
        playerTracker[player.UserId].lastPosition = hrp.Position
    end)
end)

Players.PlayerRemoving:Connect(function(player)
    playerTracker[player.UserId] = nil
end)

-- ตรวจสอบทุก interval
task.spawn(function()
    while true do
        task.wait(CHECK_INTERVAL)
        
        for _, player in ipairs(Players:GetPlayers()) do
            local tracker = playerTracker[player.UserId]
            if not tracker then continue end
            
            local character = player.Character
            if not character then continue end
            
            local hrp = character:FindFirstChild("HumanoidRootPart")
            if not hrp then continue end
            
            local currentPos = hrp.Position
            
            if tracker.lastPosition then
                local distance = (currentPos - tracker.lastPosition).Magnitude
                local speed = distance / CHECK_INTERVAL
                
                if speed > MAX_SPEED then
                    tracker.violations = tracker.violations + 1
                    warn(player.Name .. " เคลื่อนที่เร็วผิดปกติ: " .. math.floor(speed) .. " studs/s (violations: " .. tracker.violations .. ")")
                    
                    if tracker.violations >= MAX_VIOLATIONS then
                        player:Kick("ตรวจพบการโกง: เคลื่อนที่เร็วผิดปกติ")
                    end
                else
                    tracker.violations = math.max(0, tracker.violations - 1)  -- ลด violation เมื่อปกติ
                end
            end
            
            tracker.lastPosition = currentPos
        end
    end
end)

return AntiCheat
```

---

## 24.5 DataStore Manager

```lua
-- Script: DataManager.lua
-- จัดการ DataStore อย่างปลอดภัย

local DataStoreService = game:GetService("DataStoreService")
local Players = game:GetService("Players")

local DataManager = {}

local playerDataStore = DataStoreService:GetDataStore("PlayerData_v2")

-- Default data structure
local function getDefaultData()
    return {
        version = 2,
        
        -- Stats
        level = 1,
        experience = 0,
        gold = 500,
        
        -- Inventory
        inventory = {},
        
        -- Settings
        settings = {
            musicVolume = 0.5,
            sfxVolume = 1.0
        },
        
        -- Statistics
        totalKills = 0,
        totalDeaths = 0,
        totalPlaytime = 0,
        firstJoined = os.time()
    }
end

-- Player data cache
local dataCache = {}
local saveQueue = {}
local SAVE_INTERVAL = 60  -- save ทุก 60 วินาที

-- Load data ด้วย retry
local function loadWithRetry(key, maxRetries)
    maxRetries = maxRetries or 3
    
    for attempt = 1, maxRetries do
        local success, data = pcall(function()
            return playerDataStore:GetAsync(key)
        end)
        
        if success then
            return true, data
        else
            warn("Load failed (attempt " .. attempt .. "): " .. tostring(data))
            if attempt < maxRetries then
                task.wait(2 ^ attempt)  -- exponential backoff
            end
        end
    end
    
    return false, nil
end

-- Save data ด้วย retry
local function saveWithRetry(key, data, maxRetries)
    maxRetries = maxRetries or 3
    
    for attempt = 1, maxRetries do
        local success, err = pcall(function()
            playerDataStore:SetAsync(key, data)
        end)
        
        if success then
            return true
        else
            warn("Save failed (attempt " .. attempt .. "): " .. tostring(err))
            if attempt < maxRetries then
                task.wait(2 ^ attempt)
            end
        end
    end
    
    return false
end

-- โหลดข้อมูลผู้เล่น
function DataManager.loadPlayerData(player)
    local key = "player_" .. player.UserId
    
    local success, data = loadWithRetry(key)
    
    if success and data then
        -- Migration: อัปเดต data เป็น version ใหม่
        if not data.version or data.version < 2 then
            -- migrate old data
            data.version = 2
            if not data.settings then
                data.settings = getDefaultData().settings
            end
        end
        
        dataCache[player.UserId] = data
        print("โหลดข้อมูล " .. player.Name .. " สำเร็จ")
        return data
    else
        -- ใช้ default data
        local defaultData = getDefaultData()
        dataCache[player.UserId] = defaultData
        print("ใช้ข้อมูลเริ่มต้นสำหรับ " .. player.Name)
        return defaultData
    end
end

-- บันทึกข้อมูล
function DataManager.savePlayerData(player)
    local data = dataCache[player.UserId]
    if not data then return false end
    
    local key = "player_" .. player.UserId
    
    -- อัปเดต playtime
    data.lastSaved = os.time()
    
    local success = saveWithRetry(key, data)
    
    if success then
        print("บันทึกข้อมูล " .. player.Name .. " สำเร็จ")
    else
        warn("ไม่สามารถบันทึกข้อมูล " .. player.Name)
    end
    
    return success
end

-- อัปเดตค่าใน data
function DataManager.updateData(player, path, value)
    local data = dataCache[player.UserId]
    if not data then return false end
    
    -- path support: "stats.health" -> data.stats.health
    local parts = {}
    for part in path:gmatch("[^%.]+") do
        table.insert(parts, part)
    end
    
    local current = data
    for i = 1, #parts - 1 do
        current = current[parts[i]]
        if not current then return false end
    end
    
    current[parts[#parts]] = value
    return true
end

-- ดึงค่าจาก data
function DataManager.getData(player, path)
    local data = dataCache[player.UserId]
    if not data then return nil end
    
    if not path then return data end
    
    local parts = {}
    for part in path:gmatch("[^%.]+") do
        table.insert(parts, part)
    end
    
    local current = data
    for _, part in ipairs(parts) do
        current = current[part]
        if current == nil then return nil end
    end
    
    return current
end

-- Events
Players.PlayerAdded:Connect(function(player)
    DataManager.loadPlayerData(player)
end)

Players.PlayerRemoving:Connect(function(player)
    DataManager.savePlayerData(player)
    dataCache[player.UserId] = nil
end)

-- Auto-save
task.spawn(function()
    while true do
        task.wait(SAVE_INTERVAL)
        
        for _, player in ipairs(Players:GetPlayers()) do
            DataManager.savePlayerData(player)
        end
    end
end)

-- Save เมื่อ server ปิด
game:BindToClose(function()
    print("Server กำลังปิด... บันทึกข้อมูลทั้งหมด")
    
    for _, player in ipairs(Players:GetPlayers()) do
        DataManager.savePlayerData(player)
    end
    
    print("บันทึกข้อมูลทั้งหมดเสร็จแล้ว")
end)

return DataManager
```

---

## 24.6 Main Game Script

```lua
-- Script: GameMain.lua
-- Script หลักที่ควบคุมทั้งเกม

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local ServerScriptService = game:GetService("ServerScriptService")

-- Load modules
local DataManager = require(ServerScriptService.DataManager)

-- Game configuration
local CONFIG = {
    MIN_PLAYERS = 2,
    MAX_PLAYERS = 10,
    ROUND_TIME = 300,
    LOBBY_TIME = 30,
    COUNTDOWN_TIME = 10
}

-- Game state
local GameState = {
    LOBBY = "lobby",
    COUNTDOWN = "countdown",
    PLAYING = "playing",
    ENDING = "ending"
}

local currentState = GameState.LOBBY
local roundNumber = 0

-- Notify all players
local function notifyPlayers(message, style)
    local event = ReplicatedStorage:FindFirstChild("NotificationEvent")
    if event then
        event:FireAllClients({
            message = message,
            style = style or "info"
        })
    end
end

-- Check if game can start
local function canStartGame()
    return #Players:GetPlayers() >= CONFIG.MIN_PLAYERS
end

-- Start countdown
local function startCountdown()
    currentState = GameState.COUNTDOWN
    
    for i = CONFIG.COUNTDOWN_TIME, 1, -1 do
        notifyPlayers("เกมจะเริ่มใน " .. i .. " วินาที", "countdown")
        task.wait(1)
    end
end

-- Start round
local function startRound()
    currentState = GameState.PLAYING
    roundNumber = roundNumber + 1
    
    print("=== Round " .. roundNumber .. " เริ่ม ===")
    notifyPlayers("Round " .. roundNumber .. " เริ่มแล้ว!", "success")
    
    -- Spawn players
    for _, player in ipairs(Players:GetPlayers()) do
        local spawnPoint = workspace:FindFirstChild("SpawnLocation")
        if spawnPoint then
            player.RespawnLocation = spawnPoint
        end
        player:LoadCharacter()
    end
    
    -- Timer
    local timeLeft = CONFIG.ROUND_TIME
    
    while timeLeft > 0 and currentState == GameState.PLAYING do
        task.wait(1)
        timeLeft = timeLeft - 1
        
        -- แจ้งเตือนเวลา
        if timeLeft == 60 then
            notifyPlayers("เวลาเหลือ 1 นาที!", "warning")
        elseif timeLeft == 30 then
            notifyPlayers("เวลาเหลือ 30 วินาที!", "warning")
        elseif timeLeft <= 10 then
            notifyPlayers("เวลาเหลือ " .. timeLeft .. " วินาที!", "danger")
        end
    end
    
    endRound()
end

-- End round
local function endRound()
    currentState = GameState.ENDING
    
    print("=== Round " .. roundNumber .. " จบ ===")
    notifyPlayers("Round จบแล้ว!", "info")
    
    -- Show results
    task.wait(10)
    
    -- Reset to lobby
    currentState = GameState.LOBBY
    print("กลับสู่ Lobby")
end

-- Main game loop
task.spawn(function()
    while true do
        if currentState == GameState.LOBBY then
            if canStartGame() then
                task.wait(CONFIG.LOBBY_TIME)
                if canStartGame() then
                    startCountdown()
                    startRound()
                end
            else
                task.wait(5)
            end
        else
            task.wait(1)
        end
    end
end)

-- Player events
Players.PlayerAdded:Connect(function(player)
    notifyPlayers(player.Name .. " เข้าร่วม!", "info")
    
    if currentState == GameState.LOBBY and #Players:GetPlayers() >= CONFIG.MIN_PLAYERS then
        notifyPlayers("มีผู้เล่นเพียงพอแล้ว! เกมจะเริ่มเร็วๆ นี้", "success")
    end
end)

Players.PlayerRemoving:Connect(function(player)
    print(player.Name .. " ออกจากเกม")
    
    if currentState == GameState.PLAYING and #Players:GetPlayers() < 1 then
        print("ไม่มีผู้เล่นเหลือ หยุดเกม")
        currentState = GameState.ENDING
    end
end)

print("Game Server เริ่มทำงานแล้ว")
```

---

## 24.7 แบบฝึกหัด

### แบบฝึกหัดที่ 1: Kill Tracking

```lua
-- สร้างระบบติดตาม kills บน server:
-- - รับ event จาก client เมื่อโจมตี
-- - validate ว่าการโจมตีถูกต้อง (ระยะ, ผู้เล่นมีชีวิต)
-- - apply damage
-- - นับ kill/death
-- - แจ้งเตือน

-- เติมโค้ด
```

### แบบฝึกหัดที่ 2: Shop System

```lua
-- สร้างระบบ shop บน server:
-- - รับ request ซื้อสินค้าจาก client
-- - ตรวจสอบเงิน
-- - ตรวจสอบ inventory space
-- - ทำ transaction อย่างปลอดภัย

-- เติมโค้ด
```

---

## สรุป

| หลักการ | รายละเอียด |
|--------|-----------|
| Server-only | Code ใน SSS ไม่ถูกส่งไป Client |
| Validate ทุกอย่าง | อย่าเชื่อ Client blindly |
| DataStore | บันทึกข้อมูลด้วย retry |
| Game Loop | จัดการ round/state |
| Anti-Cheat | ตรวจสอบความผิดปกติ |

### บทถัดไป

ในบทที่ 25 เราจะเรียนเรื่อง **LocalScripts vs Scripts** - ความแตกต่างและการใช้งาน
