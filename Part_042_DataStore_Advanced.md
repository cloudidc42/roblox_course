# Part 42: DataStore ขั้นสูง - รูปแบบและการจัดการข้อผิดพลาด

## บทนำ

หลังจากเรียนรู้พื้นฐาน DataStore แล้ว บทนี้จะลงลึกถึงเทคนิคขั้นสูง รวมถึง session locking, data migration, การใช้ ProfileService และ pattern ที่ใช้ในเกมระดับมืออาชีพ

## Session Locking - ป้องกันข้อมูลเสียหาย

### ปัญหาที่เกิดขึ้น

เมื่อผู้เล่นเซิร์ฟเวอร์หมดความจุ Roblox อาจย้ายผู้เล่นไปยังเซิร์ฟเวอร์ใหม่ ถ้าทั้งสองเซิร์ฟเวอร์บันทึกข้อมูลพร้อมกัน ข้อมูลอาจเสียหายได้!

```
Server A กำลังบันทึก: coins = 100
Server B กำลังบันทึก: coins = 50
ผลลัพธ์อาจผิดพลาด!
```

### Session Lock Pattern

```lua
-- ServerScriptService/SessionLockManager.lua
-- ระบบป้องกันการบันทึกซ้อน

local DataStoreService = game:GetService("DataStoreService")
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local DATA_STORE_NAME = "PlayerData_v2"
local SESSION_STORE_NAME = "SessionLocks"
local SESSION_UPDATE_INTERVAL = 30  -- อัพเดท lock ทุก 30 วินาที
local SESSION_TIMEOUT = 120  -- ถ้า lock เก่ากว่า 120 วินาที ถือว่า expired

local dataStore = DataStoreService:GetDataStore(DATA_STORE_NAME)
local sessionStore = DataStoreService:GetDataStore(SESSION_STORE_NAME)

-- เก็บ lock ของเซิร์ฟเวอร์นี้
local activeLocks = {}

-- ID ของเซิร์ฟเวอร์นี้ (ไม่ซ้ำกัน)
local serverJobId = game.JobId

local function acquireSessionLock(player)
    local key = "Lock_" .. player.UserId
    local acquired = false
    local lockData = nil
    
    -- ใช้ UpdateAsync เพื่อป้องกัน race condition
    local success, result = pcall(function()
        return sessionStore:UpdateAsync(key, function(currentLock)
            -- ตรวจสอบ lock ปัจจุบัน
            if currentLock then
                local age = os.time() - (currentLock.timestamp or 0)
                
                if age < SESSION_TIMEOUT and currentLock.serverId ~= serverJobId then
                    -- lock ยังใช้งานอยู่โดย server อื่น
                    lockData = currentLock
                    return nil  -- ไม่เปลี่ยนแปลง
                end
            end
            
            -- สร้าง lock ใหม่
            local newLock = {
                serverId = serverJobId,
                timestamp = os.time(),
                playerName = player.Name
            }
            
            acquired = true
            return newLock
        end)
    end)
    
    if success and acquired then
        activeLocks[player.UserId] = true
        print("✓ Session lock acquired: " .. player.Name)
        return true
    else
        if lockData then
            warn("✗ Session lock denied: " .. player.Name .. 
                 " (locked by server " .. lockData.serverId .. ")")
        end
        return false
    end
end

local function releaseSessionLock(player)
    local key = "Lock_" .. player.UserId
    
    pcall(function()
        sessionStore:UpdateAsync(key, function(currentLock)
            if currentLock and currentLock.serverId == serverJobId then
                return nil  -- ลบ lock
            end
            return currentLock  -- ไม่ใช่ lock ของเรา ไม่ลบ
        end)
    end)
    
    activeLocks[player.UserId] = nil
    print("Lock released: " .. player.Name)
end

-- อัพเดท lock เป็นระยะเพื่อไม่ให้ expired
local function keepLockAlive(player)
    task.spawn(function()
        while player.Parent and activeLocks[player.UserId] do
            task.wait(SESSION_UPDATE_INTERVAL)
            
            if not player.Parent then break end
            
            local key = "Lock_" .. player.UserId
            pcall(function()
                sessionStore:UpdateAsync(key, function(currentLock)
                    if currentLock and currentLock.serverId == serverJobId then
                        currentLock.timestamp = os.time()
                        return currentLock
                    end
                    return nil  -- lock หายไป อย่าสร้างใหม่
                end)
            end)
        end
    end)
end

Players.PlayerAdded:Connect(function(player)
    -- รอ 2 วินาทีเผื่อ server เก่ายัง release lock ไม่ทัน
    task.wait(2)
    
    local lockAcquired = acquireSessionLock(player)
    
    if not lockAcquired then
        -- ไม่ได้ lock - kick ผู้เล่นออกชั่วคราว
        player:Kick("กำลังโหลดข้อมูล กรุณารอสักครู่แล้วเข้าใหม่")
        return
    end
    
    keepLockAlive(player)
    
    -- โหลดและเก็บข้อมูล...
end)

Players.PlayerRemoving:Connect(function(player)
    -- บันทึกข้อมูลก่อน
    -- saveData(player)
    
    -- Release lock
    releaseSessionLock(player)
end)
```

## Data Migration - การอัพเดทโครงสร้างข้อมูล

```lua
-- เมื่อต้องการเพิ่ม/ลบ field หรือเปลี่ยนโครงสร้างข้อมูล
-- ต้องทำ migration เพื่อไม่ให้ผู้เล่นเก่าสูญเสียข้อมูล

local CURRENT_VERSION = 3  -- เวอร์ชันปัจจุบัน

-- ฟังก์ชัน migration สำหรับแต่ละเวอร์ชัน
local migrations = {}

-- Migration v1 -> v2: เพิ่ม gems และ inventory
migrations[1] = function(data)
    data.gems = 0          -- เพิ่ม field ใหม่
    data.inventory = {}    -- เพิ่ม field ใหม่
    data.version = 2
    print("Migration v1 -> v2 สำเร็จ")
    return data
end

-- Migration v2 -> v3: เปลี่ยน playTime เป็น seconds เป็น minutes
migrations[2] = function(data)
    if data.totalPlayTime then
        -- เดิมเก็บเป็นวินาที เปลี่ยนเป็นนาที
        data.totalPlayTime = math.floor(data.totalPlayTime / 60)
    end
    -- เพิ่ม achievement system
    data.achievements = {}
    data.version = 3
    print("Migration v2 -> v3 สำเร็จ")
    return data
end

-- ฟังก์ชัน migrate ข้อมูล
local function migrateData(data)
    local currentVersion = data.version or 1
    
    if currentVersion == CURRENT_VERSION then
        return data  -- ไม่ต้อง migrate
    end
    
    print(string.format("กำลัง migrate ข้อมูล v%d -> v%d", currentVersion, CURRENT_VERSION))
    
    -- ทำ migration ทีละขั้น
    for version = currentVersion, CURRENT_VERSION - 1 do
        if migrations[version] then
            local success, result = pcall(migrations[version], data)
            if success then
                data = result
            else
                warn("Migration v" .. version .. " ล้มเหลว: " .. result)
                break
            end
        end
    end
    
    return data
end

-- ตัวอย่างการใช้ใน loadData
local function loadDataWithMigration(player)
    local DataStoreService = game:GetService("DataStoreService")
    local dataStore = DataStoreService:GetDataStore("PlayerData_v2")
    
    local key = "Player_" .. player.UserId
    local success, data = pcall(function()
        return dataStore:GetAsync(key)
    end)
    
    if success and data then
        -- ทำ migration ถ้าจำเป็น
        data = migrateData(data)
        return data
    else
        -- ผู้เล่นใหม่ - สร้างข้อมูล version ปัจจุบัน
        return {
            version = CURRENT_VERSION,
            coins = 0,
            gems = 0,
            level = 1,
            experience = 0,
            totalPlayTime = 0,
            inventory = {},
            achievements = {}
        }
    end
end
```

## DataStore Budget System

```lua
-- Roblox จำกัดจำนวน request ต่อนาที
-- เราสามารถเช็คได้ว่าเหลือ budget เท่าไหร่

local DataStoreService = game:GetService("DataStoreService")

-- ประเภทของ budget
-- Enum.DataStoreRequestType:
-- GetAsync, SetIncrementAsync, UpdateAsync, 
-- GetSortedAsync, SetIncrementSortedAsync, OnUpdate

local function checkBudget()
    local types = {
        { name = "GetAsync", type = Enum.DataStoreRequestType.GetAsync },
        { name = "SetAsync/IncrementAsync", type = Enum.DataStoreRequestType.SetIncrementAsync },
        { name = "UpdateAsync", type = Enum.DataStoreRequestType.UpdateAsync },
    }
    
    print("=== DataStore Budget ===")
    for _, requestType in ipairs(types) do
        local budget = DataStoreService:GetRequestBudgetForRequestType(requestType.type)
        print(string.format("%s: %d requests เหลือ", requestType.name, budget))
    end
end

-- เรียกดู budget ทุก 10 วินาที
task.spawn(function()
    while true do
        checkBudget()
        task.wait(10)
    end
end)

-- ระบบ queue เพื่อจัดการ request
local requestQueue = {}
local isProcessing = false

local function queueRequest(func, priority)
    priority = priority or 1  -- 1 = ปกติ, 2 = สำคัญ
    
    table.insert(requestQueue, {
        func = func,
        priority = priority,
        timestamp = os.time()
    })
    
    -- เรียงตาม priority
    table.sort(requestQueue, function(a, b)
        if a.priority ~= b.priority then
            return a.priority > b.priority  -- priority สูงก่อน
        end
        return a.timestamp < b.timestamp  -- ถ้า priority เท่ากัน เข้าก่อนออกก่อน
    end)
end

local function processQueue()
    if isProcessing then return end
    
    isProcessing = true
    task.spawn(function()
        while #requestQueue > 0 do
            -- ตรวจสอบ budget
            local budget = DataStoreService:GetRequestBudgetForRequestType(
                Enum.DataStoreRequestType.SetIncrementAsync
            )
            
            if budget <= 5 then
                warn("Budget ต่ำมาก! รอก่อน...")
                task.wait(5)
            end
            
            local request = table.remove(requestQueue, 1)
            
            local success, err = pcall(request.func)
            if not success then
                warn("Request ล้มเหลว: " .. err)
            end
            
            task.wait(0.1)  -- หน่วงเล็กน้อยระหว่าง request
        end
        isProcessing = false
    end)
end
```

## ระบบ DataStore ที่สมบูรณ์สำหรับเกม

```lua
-- ServerScriptService/DataManager.lua
-- ระบบจัดการข้อมูลที่สมบูรณ์และพร้อมใช้งานจริง

local DataStoreService = game:GetService("DataStoreService")
local Players = game:GetService("Players")
local HttpService = game:GetService("HttpService")

-- ===== การตั้งค่า =====
local CONFIG = {
    dataStoreName = "GameData_v3",
    maxRetries = 5,
    retryDelay = 2,
    autoSaveInterval = 120,  -- 2 นาที
    sessionLockTimeout = 180,
    currentDataVersion = 3
}

-- ===== DataStore =====
local gameDataStore = DataStoreService:GetDataStore(CONFIG.dataStoreName)

-- ===== State =====
local playerSessions = {}  -- { [userId] = { data, isLoaded, isSaving } }

-- ===== Default Data =====
local function createDefaultData()
    return {
        version = CONFIG.currentDataVersion,
        
        -- Currency
        coins = 100,
        gems = 5,
        
        -- Progress
        level = 1,
        experience = 0,
        prestigeLevel = 0,
        
        -- Stats
        totalPlayTime = 0,
        totalKills = 0,
        totalDeaths = 0,
        totalWins = 0,
        highestScore = 0,
        
        -- Inventory
        items = {},
        equippedItems = {},
        
        -- Settings
        settings = {
            musicVolume = 0.7,
            sfxVolume = 1.0,
            showDamageNumbers = true,
            autoCollect = false
        },
        
        -- Timestamps
        createdAt = os.time(),
        lastLogin = os.time()
    }
end

-- ===== Utility Functions =====

local function log(level, message, ...)
    local prefix = {
        INFO = "ℹ️",
        WARN = "⚠️",
        ERROR = "❌",
        SUCCESS = "✅"
    }
    print(string.format("%s [DataManager] %s", prefix[level] or "•", string.format(message, ...)))
end

local function deepCopy(obj)
    if type(obj) ~= "table" then return obj end
    local copy = {}
    for k, v in pairs(obj) do
        copy[k] = deepCopy(v)
    end
    return copy
end

-- ===== Core Functions =====

local function loadPlayerData(player)
    local userId = player.UserId
    local key = "P_" .. userId
    
    log("INFO", "กำลังโหลด %s (ID: %d)", player.Name, userId)
    
    local data = nil
    
    for attempt = 1, CONFIG.maxRetries do
        local ok, result = pcall(function()
            return gameDataStore:GetAsync(key)
        end)
        
        if ok then
            data = result
            break
        else
            log("WARN", "โหลดครั้งที่ %d/%d ล้มเหลว: %s", attempt, CONFIG.maxRetries, result)
            
            if attempt < CONFIG.maxRetries then
                task.wait(CONFIG.retryDelay * attempt)  -- backoff delay
            end
        end
    end
    
    -- ประมวลผลข้อมูล
    if data then
        -- ตรวจสอบและ migrate ถ้าจำเป็น
        if (data.version or 0) < CONFIG.currentDataVersion then
            data = migrateData(data)
        end
        log("SUCCESS", "โหลด %s สำเร็จ (Level %d, Coins: %d)", 
            player.Name, data.level, data.coins)
    else
        data = createDefaultData()
        log("INFO", "สร้างข้อมูลใหม่สำหรับ %s", player.Name)
    end
    
    data.lastLogin = os.time()
    return data
end

local function savePlayerData(player, silent)
    local userId = player.UserId
    local session = playerSessions[userId]
    
    if not session or not session.data then
        log("WARN", "ไม่พบ session ของ %s", player.Name)
        return false
    end
    
    if session.isSaving then
        log("INFO", "กำลังบันทึก %s อยู่แล้ว ข้าม...", player.Name)
        return true  -- ถือว่า success
    end
    
    session.isSaving = true
    
    local key = "P_" .. userId
    local dataToSave = deepCopy(session.data)
    
    -- อัพเดทเวลาเล่นรวม
    if session.sessionStartTime then
        local sessionTime = os.time() - session.sessionStartTime
        dataToSave.totalPlayTime = dataToSave.totalPlayTime + sessionTime
        session.sessionStartTime = os.time()  -- reset timer
    end
    
    local success = false
    
    for attempt = 1, CONFIG.maxRetries do
        local ok, err = pcall(function()
            gameDataStore:SetAsync(key, dataToSave)
        end)
        
        if ok then
            success = true
            break
        else
            log("WARN", "บันทึกครั้งที่ %d/%d ล้มเหลว: %s", attempt, CONFIG.maxRetries, err)
            if attempt < CONFIG.maxRetries then
                task.wait(CONFIG.retryDelay)
            end
        end
    end
    
    session.isSaving = false
    
    if not silent then
        if success then
            log("SUCCESS", "บันทึก %s สำเร็จ", player.Name)
        else
            log("ERROR", "บันทึก %s ล้มเหลว!", player.Name)
        end
    end
    
    return success
end

-- ===== Migration =====

function migrateData(data)
    local version = data.version or 0
    
    -- v0 -> v1: เพิ่ม gems
    if version < 1 then
        data.gems = 0
        data.version = 1
    end
    
    -- v1 -> v2: เพิ่ม prestige
    if version < 2 then
        data.prestigeLevel = 0
        data.highestScore = 0
        data.version = 2
    end
    
    -- v2 -> v3: เพิ่ม settings
    if version < 3 then
        data.settings = {
            musicVolume = 0.7,
            sfxVolume = 1.0,
            showDamageNumbers = true,
            autoCollect = false
        }
        data.version = 3
    end
    
    log("INFO", "Migration เสร็จสิ้น -> v%d", CONFIG.currentDataVersion)
    return data
end

-- ===== Event Handlers =====

Players.PlayerAdded:Connect(function(player)
    playerSessions[player.UserId] = {
        data = nil,
        isLoaded = false,
        isSaving = false,
        sessionStartTime = nil
    }
    
    -- โหลดข้อมูลใน background
    task.spawn(function()
        local data = loadPlayerData(player)
        
        -- ตรวจสอบว่าผู้เล่นยังอยู่
        if not player.Parent then
            log("WARN", "%s ออกก่อนโหลดข้อมูลเสร็จ", player.Name)
            return
        end
        
        local session = playerSessions[player.UserId]
        if session then
            session.data = data
            session.isLoaded = true
            session.sessionStartTime = os.time()
        end
        
        -- ตั้งค่า leaderstats
        setupLeaderstats(player, data)
        
        -- แจ้งว่าข้อมูลพร้อม
        -- (ใน system จริงจะใช้ RemoteEvent แจ้ง client)
    end)
end)

Players.PlayerRemoving:Connect(function(player)
    -- บันทึกก่อนออก (blocking)
    savePlayerData(player, false)
    
    -- ทำความสะอาด
    playerSessions[player.UserId] = nil
end)

function setupLeaderstats(player, data)
    local leaderstats = Instance.new("Folder")
    leaderstats.Name = "leaderstats"
    leaderstats.Parent = player
    
    local function makeValue(name, value, valueType)
        local v = Instance.new(valueType or "IntValue")
        v.Name = name
        v.Value = value
        v.Parent = leaderstats
        return v
    end
    
    local coinsVal = makeValue("💰 Coins", data.coins)
    local gemsVal = makeValue("💎 Gems", data.gems)
    local levelVal = makeValue("⭐ Level", data.level)
end

-- ===== Auto Save =====
task.spawn(function()
    while true do
        task.wait(CONFIG.autoSaveInterval)
        
        local count = 0
        for _, player in ipairs(Players:GetPlayers()) do
            local session = playerSessions[player.UserId]
            if session and session.isLoaded then
                task.spawn(function()
                    savePlayerData(player, true)
                end)
                count = count + 1
            end
        end
        
        if count > 0 then
            log("INFO", "Auto-save: %d ผู้เล่น", count)
        end
    end
end)

-- ===== Server Close =====
game:BindToClose(function()
    log("INFO", "Server กำลังปิด - บันทึกทั้งหมด...")
    
    local tasks = {}
    for _, player in ipairs(Players:GetPlayers()) do
        table.insert(tasks, task.spawn(function()
            savePlayerData(player, false)
        end))
    end
    
    task.wait(5)  -- รอสูงสุด 5 วินาที
    log("INFO", "บันทึกเสร็จสิ้น")
end)

-- ===== Public API =====

local DataManager = {}

function DataManager.waitForData(player, timeout)
    timeout = timeout or 10
    local start = os.clock()
    
    while os.clock() - start < timeout do
        local session = playerSessions[player.UserId]
        if session and session.isLoaded then
            return session.data
        end
        task.wait(0.1)
    end
    
    log("ERROR", "Timeout รอข้อมูล %s", player.Name)
    return nil
end

function DataManager.getData(player)
    local session = playerSessions[player.UserId]
    return session and session.data
end

function DataManager.setValue(player, path, value)
    local session = playerSessions[player.UserId]
    if not session or not session.data then return false end
    
    -- รองรับ path แบบ "settings.musicVolume"
    local keys = string.split(path, ".")
    local current = session.data
    
    for i = 1, #keys - 1 do
        current = current[keys[i]]
        if type(current) ~= "table" then return false end
    end
    
    current[keys[#keys]] = value
    return true
end

function DataManager.getValue(player, path)
    local session = playerSessions[player.UserId]
    if not session or not session.data then return nil end
    
    local keys = string.split(path, ".")
    local current = session.data
    
    for _, key in ipairs(keys) do
        if type(current) ~= "table" then return nil end
        current = current[key]
    end
    
    return current
end

function DataManager.addCoins(player, amount)
    local data = DataManager.getData(player)
    if not data then return false end
    
    if amount < 0 and data.coins < math.abs(amount) then
        return false  -- ไม่มีเหรียญพอ
    end
    
    data.coins = math.max(0, data.coins + amount)
    
    -- อัพเดท leaderstats
    local leaderstats = player:FindFirstChild("leaderstats")
    if leaderstats then
        local coinsVal = leaderstats:FindFirstChild("💰 Coins")
        if coinsVal then
            coinsVal.Value = data.coins
        end
    end
    
    return true
end

function DataManager.addExperience(player, amount)
    local data = DataManager.getData(player)
    if not data then return end
    
    data.experience = data.experience + amount
    
    local leveledUp = false
    local levelsGained = 0
    
    while true do
        local expRequired = math.floor(100 * (1.5 ^ (data.level - 1)))
        
        if data.experience >= expRequired then
            data.experience = data.experience - expRequired
            data.level = data.level + 1
            levelsGained = levelsGained + 1
            leveledUp = true
        else
            break
        end
    end
    
    if leveledUp then
        -- อัพเดท leaderstats
        local leaderstats = player:FindFirstChild("leaderstats")
        if leaderstats then
            local levelVal = leaderstats:FindFirstChild("⭐ Level")
            if levelVal then
                levelVal.Value = data.level
            end
        end
        
        log("SUCCESS", "%s เลื่อนระดับ +%d ครั้ง -> Level %d!", 
            player.Name, levelsGained, data.level)
    end
    
    return leveledUp, levelsGained
end

return DataManager
```

## Global DataStore - ข้อมูลร่วมกัน

```lua
-- ข้อมูลที่ใช้ร่วมกันทุกเซิร์ฟเวอร์ เช่น Event, Global Challenge

local DataStoreService = game:GetService("DataStoreService")
local globalStore = DataStoreService:GetDataStore("GlobalData")

-- Global event state
local function getGlobalEvent()
    local success, data = pcall(function()
        return globalStore:GetAsync("CurrentEvent")
    end)
    
    if success and data then
        return data
    end
    
    return nil
end

-- อัพเดท global counter (เช่น จำนวนศัตรูที่ทุกเซิร์ฟเวอร์ฆ่า)
local function addGlobalKills(amount)
    local success, newTotal = pcall(function()
        return DataStoreService:GetDataStore("GlobalStats"):IncrementAsync(
            "TotalEnemiesKilled", 
            amount
        )
    end)
    
    if success then
        print("รวมฆ่าศัตรูทั่วโลก: " .. newTotal)
        return newTotal
    end
    
    return nil
end

-- ตรวจสอบ global milestone
local function checkGlobalMilestones()
    task.spawn(function()
        while true do
            task.wait(300)  -- ทุก 5 นาที
            
            local success, totalKills = pcall(function()
                return DataStoreService:GetDataStore("GlobalStats"):GetAsync("TotalEnemiesKilled")
            end)
            
            if success and totalKills then
                -- ตรวจสอบ milestone
                local milestones = {1000, 10000, 100000, 1000000}
                
                for _, milestone in ipairs(milestones) do
                    if totalKills >= milestone then
                        -- ให้รางวัลพิเศษ...
                        print("🎉 Global Milestone: " .. milestone .. " kills!")
                    end
                end
            end
        end
    end)
end
```

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: ระบบ Backup อัตโนมัติ

```lua
-- สร้างระบบที่บันทึกข้อมูลสำรองทุกวัน
-- ใช้ key เป็น "Backup_{userId}_{date}"

local DataStoreService = game:GetService("DataStoreService")
local backupStore = DataStoreService:GetDataStore("PlayerBackups")

local function createDailyBackup(player, data)
    local today = os.date("%Y-%m-%d")  -- เช่น "2024-01-15"
    local key = string.format("Backup_%d_%s", player.UserId, today)
    
    -- ตรวจสอบว่าวันนี้มี backup แล้วหรือยัง
    local exists = false
    pcall(function()
        local existing = backupStore:GetAsync(key)
        exists = existing ~= nil
    end)
    
    if not exists then
        pcall(function()
            backupStore:SetAsync(key, {
                data = data,
                timestamp = os.time(),
                playerName = player.Name
            })
        end)
        print("💾 สร้าง backup สำหรับ " .. player.Name)
    end
end

-- TODO: เพิ่มฟังก์ชัน restoreFromBackup
```

### แบบฝึกหัดที่ 2: ระบบ Transaction
สร้างระบบซื้อขายที่ปลอดภัย ที่จะ rollback ถ้าเกิด error ระหว่างการโอน

### แบบฝึกหัดที่ 3: Data Compression
เรียนรู้วิธีบีบอัดข้อมูลเพื่อใช้ space ใน DataStore ให้น้อยลง

```lua
-- ตัวอย่าง: แทนที่จะเก็บ item names เก็บ item IDs แทน
local itemDatabase = {
    [1] = "Sword",
    [2] = "Shield", 
    [3] = "Bow",
    [4] = "Staff"
}

-- ❌ ใช้ space เยอะ
local badInventory = {
    "Sword", "Shield", "Bow", "Sword", "Staff"
}

-- ✅ ใช้ IDs ประหยัด space
local goodInventory = {1, 2, 3, 1, 4}
```

## เคล็ดลับจากมืออาชีพ

1. **ใช้ ProfileService** - library ที่ทำ session locking ให้อัตโนมัติ
2. **ทดสอบบน Server เสมอ** - DataStore ไม่ทำงานบน Client
3. **Log ทุกการบันทึก** - เพื่อ debug ได้ง่าย
4. **ใช้ Version Number** - เพื่อทำ migration ได้
5. **อย่าเก็บข้อมูลใหญ่** - เช่น screenshots หรือไฟล์ใหญ่
6. **ทดสอบ edge case** - เช่น ผู้เล่นออกกะทันหัน, server crash

## สรุป

ในบทนี้เราเรียนรู้:
- **Session Locking** - ป้องกันข้อมูลเสียหายจาก server หลายตัว
- **Data Migration** - อัพเดทโครงสร้างข้อมูลโดยไม่ทำลายข้อมูลเก่า
- **Budget Management** - จัดการ request limit ของ DataStore
- **Global DataStore** - ข้อมูลร่วมกันทุก server
- **Complete DataManager** - ระบบจัดการข้อมูลที่พร้อมใช้งานจริง

ระบบ DataStore ที่ดีคือหัวใจสำคัญของเกมที่ยั่งยืน ผู้เล่นจะรู้สึกมั่นใจว่าข้อมูลของตนปลอดภัย
