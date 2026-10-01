# Part 81: Advanced DataStore - ระบบจัดเก็บข้อมูลขั้นสูง

## บทนำ (Introduction)

ในบทนี้เราจะเรียนรู้เกี่ยวกับระบบ DataStore ขั้นสูงในการพัฒนาเกม Roblox โดยเฉพาะการใช้ ProfileService ซึ่งเป็น library ยอดนิยมที่ช่วยจัดการข้อมูลผู้เล่นอย่างมีประสิทธิภาพและปลอดภัย

## ทำไมต้องใช้ Advanced DataStore?

DataStore พื้นฐานของ Roblox นั้นทำงานได้ แต่มีข้อจำกัดหลายอย่าง:

1. **Race Conditions** - เมื่อผู้เล่นหลายคนเซฟข้อมูลพร้อมกัน อาจเกิดการทับกัน
2. **Data Loss** - หากเซิร์ฟเวอร์ปิดกะทันหัน ข้อมูลอาจสูญหาย
3. **Session Locking** - ป้องกันการเล่นในหลายเซิร์ฟเวอร์พร้อมกัน
4. **Retry Logic** - การลองใหม่เมื่อ API ล้มเหลว

---

## ส่วนที่ 1: DataStore พื้นฐานที่ปรับปรุงแล้ว

### 1.1 การตั้งค่า DataStore Service

```lua
-- Script: DataStoreManager (ModuleScript ใน ServerStorage)
-- ระบบจัดการ DataStore แบบปรับปรุง

local DataStoreService = game:GetService("DataStoreService")
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

-- กำหนดค่าคงที่
local DATA_STORE_NAME = "PlayerData_v1"
local SAVE_INTERVAL = 60 -- บันทึกทุก 60 วินาที
local MAX_RETRIES = 3
local RETRY_DELAY = 2

-- สร้าง DataStore หลัก
local playerDataStore = DataStoreService:GetDataStore(DATA_STORE_NAME)

-- ข้อมูลเริ่มต้นของผู้เล่น
local DEFAULT_DATA = {
    -- ข้อมูลทั่วไป
    coins = 0,
    gems = 0,
    level = 1,
    experience = 0,
    
    -- สถิติ
    totalPlayTime = 0,
    totalDeaths = 0,
    totalKills = 0,
    highScore = 0,
    
    -- ไอเทมและอุปกรณ์
    inventory = {},
    equippedItems = {},
    
    -- การตั้งค่า
    settings = {
        musicVolume = 0.5,
        sfxVolume = 0.5,
        showHints = true
    },
    
    -- ความสำเร็จ
    achievements = {},
    
    -- เวลา
    lastLogin = 0,
    firstLogin = 0,
    
    -- เวอร์ชันข้อมูล (สำหรับ migration)
    dataVersion = 1
}

-- เก็บข้อมูลในหน่วยความจำ
local playerDataCache = {}
local savingStatus = {}

-- =====================================
-- ฟังก์ชันหลัก
-- =====================================

-- ฟังก์ชัน deep copy เพื่อป้องกันการ reference
local function deepCopy(original)
    local copy = {}
    for key, value in pairs(original) do
        if type(value) == "table" then
            copy[key] = deepCopy(value)
        else
            copy[key] = value
        end
    end
    return copy
end

-- ฟังก์ชัน merge ข้อมูลที่ขาดหายไป
local function mergeWithDefaults(data, defaults)
    for key, defaultValue in pairs(defaults) do
        if data[key] == nil then
            if type(defaultValue) == "table" then
                data[key] = deepCopy(defaultValue)
            else
                data[key] = defaultValue
            end
        elseif type(defaultValue) == "table" and type(data[key]) == "table" then
            mergeWithDefaults(data[key], defaultValue)
        end
    end
    return data
end

-- ฟังก์ชัน retry สำหรับ API calls
local function retryOperation(operation, maxRetries, delay)
    local lastError
    for attempt = 1, maxRetries do
        local success, result = pcall(operation)
        if success then
            return true, result
        else
            lastError = result
            if attempt < maxRetries then
                task.wait(delay * attempt) -- exponential backoff
            end
        end
    end
    return false, lastError
end

-- โหลดข้อมูลผู้เล่น
local function loadPlayerData(player)
    local userId = player.UserId
    local key = "Player_" .. userId
    
    local success, data = retryOperation(function()
        return playerDataStore:GetAsync(key)
    end, MAX_RETRIES, RETRY_DELAY)
    
    if success then
        if data then
            -- Merge กับค่าเริ่มต้นเพื่อเพิ่มข้อมูลใหม่
            data = mergeWithDefaults(data, deepCopy(DEFAULT_DATA))
        else
            -- ผู้เล่นใหม่ - ใช้ค่าเริ่มต้น
            data = deepCopy(DEFAULT_DATA)
            data.firstLogin = os.time()
        end
        
        -- อัพเดทเวลาเข้าสู่ระบบ
        data.lastLogin = os.time()
        
        -- เก็บในแคช
        playerDataCache[userId] = data
        
        print(string.format("[DataStore] โหลดข้อมูลสำเร็จ: %s", player.Name))
        return data
    else
        warn(string.format("[DataStore] ไม่สามารถโหลดข้อมูล %s: %s", player.Name, tostring(data)))
        -- ใช้ข้อมูลเริ่มต้นในกรณีฉุกเฉิน
        local defaultData = deepCopy(DEFAULT_DATA)
        defaultData.firstLogin = os.time()
        defaultData.lastLogin = os.time()
        playerDataCache[userId] = defaultData
        return defaultData
    end
end

-- บันทึกข้อมูลผู้เล่น
local function savePlayerData(player)
    local userId = player.UserId
    local data = playerDataCache[userId]
    
    if not data then
        warn(string.format("[DataStore] ไม่พบข้อมูลในแคช: %s", player.Name))
        return false
    end
    
    -- ป้องกันการบันทึกซ้ำซ้อน
    if savingStatus[userId] then
        return false
    end
    
    savingStatus[userId] = true
    
    local key = "Player_" .. userId
    local success, result = retryOperation(function()
        playerDataStore:SetAsync(key, data)
    end, MAX_RETRIES, RETRY_DELAY)
    
    savingStatus[userId] = nil
    
    if success then
        print(string.format("[DataStore] บันทึกข้อมูลสำเร็จ: %s", player.Name))
        return true
    else
        warn(string.format("[DataStore] ไม่สามารถบันทึกข้อมูล %s: %s", player.Name, tostring(result)))
        return false
    end
end

-- =====================================
-- API สาธารณะ
-- =====================================

local DataStoreManager = {}

-- ดึงข้อมูลจากแคช
function DataStoreManager:GetData(player)
    return playerDataCache[player.UserId]
end

-- อัพเดทข้อมูลเฉพาะส่วน
function DataStoreManager:UpdateData(player, updates)
    local data = playerDataCache[player.UserId]
    if not data then return false end
    
    for key, value in pairs(updates) do
        data[key] = value
    end
    return true
end

-- เพิ่มเหรียญ
function DataStoreManager:AddCoins(player, amount)
    local data = playerDataCache[player.UserId]
    if not data then return false end
    
    data.coins = math.max(0, data.coins + amount)
    return true
end

-- ลดเหรียญ (คืนค่า false ถ้าไม่พอ)
function DataStoreManager:SpendCoins(player, amount)
    local data = playerDataCache[player.UserId]
    if not data then return false end
    
    if data.coins < amount then
        return false, "ไม่มีเหรียญเพียงพอ"
    end
    
    data.coins = data.coins - amount
    return true
end

-- เพิ่ม Experience และตรวจสอบ Level Up
function DataStoreManager:AddExperience(player, amount)
    local data = playerDataCache[player.UserId]
    if not data then return false end
    
    data.experience = data.experience + amount
    
    -- คำนวณ XP ที่ต้องการสำหรับ level ถัดไป
    local function getRequiredXP(level)
        return math.floor(100 * (level ^ 1.5))
    end
    
    local leveled = false
    while data.experience >= getRequiredXP(data.level) do
        data.experience = data.experience - getRequiredXP(data.level)
        data.level = data.level + 1
        leveled = true
    end
    
    return leveled, data.level
end

-- บันทึกทันที
function DataStoreManager:ForceSave(player)
    return savePlayerData(player)
end

-- =====================================
-- Event Handlers
-- =====================================

Players.PlayerAdded:Connect(function(player)
    local data = loadPlayerData(player)
    
    -- ส่ง event แจ้งว่าข้อมูลพร้อมใช้งาน
    -- (ใช้ RemoteEvent หรือ BindableEvent ตามต้องการ)
    print(string.format("[DataStore] ผู้เล่น %s เข้าสู่เกม Level %d", player.Name, data.level))
end)

Players.PlayerRemoving:Connect(function(player)
    savePlayerData(player)
    -- ลบแคชเมื่อออกจากเกม
    task.delay(5, function()
        playerDataCache[player.UserId] = nil
        savingStatus[player.UserId] = nil
    end)
end)

-- Auto-save ทุก interval
task.spawn(function()
    while true do
        task.wait(SAVE_INTERVAL)
        for _, player in ipairs(Players:GetPlayers()) do
            savePlayerData(player)
        end
    end
end)

-- บันทึกเมื่อเซิร์ฟเวอร์ปิด
game:BindToClose(function()
    print("[DataStore] กำลังบันทึกข้อมูลก่อนปิดเซิร์ฟเวอร์...")
    
    local saveTasks = {}
    for _, player in ipairs(Players:GetPlayers()) do
        table.insert(saveTasks, task.spawn(function()
            savePlayerData(player)
        end))
    end
    
    -- รอให้บันทึกเสร็จทั้งหมด (สูงสุด 30 วินาที)
    local startTime = os.clock()
    while #saveTasks > 0 and os.clock() - startTime < 30 do
        task.wait(0.1)
    end
    
    print("[DataStore] บันทึกข้อมูลเสร็จสิ้น")
end)

return DataStoreManager
```

---

## ส่วนที่ 2: ProfileService - ระบบจัดการโปรไฟล์

ProfileService เป็น open-source library ที่แก้ปัญหาหลักของ DataStore

### 2.1 การติดตั้ง ProfileService

```lua
-- ดาวน์โหลด ProfileService จาก:
-- https://github.com/MadStudioRoblox/ProfileService

-- วางใน ServerScriptService หรือ ServerStorage
-- เป็น ModuleScript ชื่อ "ProfileService"
```

### 2.2 การตั้งค่าพื้นฐาน

```lua
-- Script: ProfileManager (ModuleScript ใน ServerStorage)
-- ระบบจัดการโปรไฟล์ด้วย ProfileService

local ProfileService = require(game.ServerStorage.ProfileService)
local Players = game:GetService("Players")

-- กำหนด template ข้อมูลผู้เล่น
local PROFILE_TEMPLATE = {
    -- สกุลเงิน
    Coins = 0,
    Gems = 0,
    
    -- ความคืบหน้า
    Level = 1,
    Experience = 0,
    
    -- สถิติ
    Stats = {
        TotalPlayTime = 0,
        TotalDeaths = 0,
        TotalKills = 0,
        HighScore = 0,
        GamesPlayed = 0,
    },
    
    -- ไอเทม
    Inventory = {},
    EquippedItems = {
        Weapon = nil,
        Armor = nil,
        Accessory = nil,
    },
    
    -- การตั้งค่า
    Settings = {
        MusicVolume = 0.5,
        SFXVolume = 0.5,
        ShowHints = true,
        AutoSprint = false,
    },
    
    -- ความสำเร็จ
    Achievements = {},
    CompletedQuests = {},
    
    -- ข้อมูลเวลา
    FirstJoin = 0,
    LastJoin = 0,
    
    -- Metadata
    Version = 1,
}

-- สร้าง ProfileStore
local ProfileStore = ProfileService.GetProfileStore(
    "PlayerData",
    PROFILE_TEMPLATE
)

-- เก็บ profiles ที่ active
local Profiles = {}

-- =====================================
-- การจัดการโปรไฟล์
-- =====================================

local function PlayerAdded(player)
    -- โหลดโปรไฟล์
    local profile = ProfileStore:LoadProfileAsync(
        "Player_" .. player.UserId,
        "ForceLoad" -- บังคับโหลด (ป้องกัน session ซ้ำ)
    )
    
    if profile ~= nil then
        -- ตรวจสอบว่าผู้เล่นยังอยู่
        profile:AddUserId(player.UserId) -- GDPR compliance
        profile:Reconcile() -- เพิ่มข้อมูลที่ขาดหายจาก template
        
        -- จัดการเมื่อโปรไฟล์ถูก release
        profile:ListenToRelease(function()
            Profiles[player] = nil
            player:Kick("ข้อมูลของคุณถูกโหลดจากเซิร์ฟเวอร์อื่น กรุณาเข้าสู่ระบบใหม่")
        end)
        
        -- ตรวจสอบอีกครั้งว่าผู้เล่นยังอยู่
        if player:IsDescendantOf(Players) then
            Profiles[player] = profile
            
            -- อัพเดทเวลา login
            if profile.Data.FirstJoin == 0 then
                profile.Data.FirstJoin = os.time()
            end
            profile.Data.LastJoin = os.time()
            
            print(string.format("[ProfileManager] โหลดโปรไฟล์สำเร็จ: %s", player.Name))
            
            -- เริ่มนับเวลาเล่น
            task.spawn(function()
                while Profiles[player] ~= nil do
                    task.wait(1)
                    if Profiles[player] then
                        Profiles[player].Data.Stats.TotalPlayTime += 1
                    end
                end
            end)
        else
            -- ผู้เล่นออกไปแล้วก่อนที่โปรไฟล์จะโหลด
            profile:Release()
        end
    else
        -- ไม่สามารถโหลดโปรไฟล์
        player:Kick("ไม่สามารถโหลดข้อมูลของคุณได้ กรุณาลองใหม่")
    end
end

-- ดำเนินการสำหรับผู้เล่นที่อยู่แล้ว
for _, player in ipairs(Players:GetPlayers()) do
    task.spawn(PlayerAdded, player)
end

Players.PlayerAdded:Connect(PlayerAdded)

Players.PlayerRemoving:Connect(function(player)
    local profile = Profiles[player]
    if profile ~= nil then
        profile:Release()
    end
end)

-- =====================================
-- API สาธารณะ
-- =====================================

local ProfileManager = {}

-- ดึงโปรไฟล์ของผู้เล่น
function ProfileManager:GetProfile(player)
    return Profiles[player]
end

-- ดึงข้อมูลของผู้เล่น
function ProfileManager:GetData(player)
    local profile = Profiles[player]
    if profile then
        return profile.Data
    end
    return nil
end

-- รอจนกว่าโปรไฟล์จะโหลด
function ProfileManager:WaitForProfile(player)
    local startTime = os.clock()
    while Profiles[player] == nil and player:IsDescendantOf(Players) do
        task.wait(0.1)
        if os.clock() - startTime > 10 then
            warn("[ProfileManager] รอโปรไฟล์นานเกินไป: " .. player.Name)
            return nil
        end
    end
    return Profiles[player]
end

-- เพิ่มเหรียญ
function ProfileManager:AddCoins(player, amount)
    local data = self:GetData(player)
    if data then
        data.Coins = math.max(0, data.Coins + amount)
        return true
    end
    return false
end

-- ใช้เหรียญ
function ProfileManager:SpendCoins(player, amount)
    local data = self:GetData(player)
    if not data then return false, "ไม่พบข้อมูล" end
    if data.Coins < amount then return false, "เหรียญไม่เพียงพอ" end
    
    data.Coins = data.Coins - amount
    return true
end

-- เพิ่มไอเทม
function ProfileManager:AddItem(player, itemId, quantity)
    local data = self:GetData(player)
    if not data then return false end
    
    quantity = quantity or 1
    
    if not data.Inventory[itemId] then
        data.Inventory[itemId] = 0
    end
    data.Inventory[itemId] = data.Inventory[itemId] + quantity
    return true
end

-- ตรวจสอบ Achievement
function ProfileManager:UnlockAchievement(player, achievementId)
    local data = self:GetData(player)
    if not data then return false end
    
    if not table.find(data.Achievements, achievementId) then
        table.insert(data.Achievements, achievementId)
        return true -- unlocked ใหม่
    end
    return false -- มีอยู่แล้ว
end

-- เพิ่ม XP และ Level Up
function ProfileManager:AddExperience(player, amount)
    local data = self:GetData(player)
    if not data then return false end
    
    data.Experience = data.Experience + amount
    
    local leveled = false
    local function xpForLevel(level)
        return math.floor(100 * (level ^ 1.5))
    end
    
    while data.Experience >= xpForLevel(data.Level) do
        data.Experience = data.Experience - xpForLevel(data.Level)
        data.Level = data.Level + 1
        leveled = true
    end
    
    return leveled
end

return ProfileManager
```

---

## ส่วนที่ 3: OrderedDataStore - ระบบ Leaderboard

```lua
-- Script: LeaderboardSystem (Script ใน ServerScriptService)
-- ระบบกระดานอันดับ

local DataStoreService = game:GetService("DataStoreService")
local Players = game:GetService("Players")

-- DataStores สำหรับ leaderboards
local coinsLeaderboard = DataStoreService:GetOrderedDataStore("Leaderboard_Coins")
local levelLeaderboard = DataStoreService:GetOrderedDataStore("Leaderboard_Level")
local killsLeaderboard = DataStoreService:GetOrderedDataStore("Leaderboard_Kills")

-- อัพเดท leaderboard สำหรับผู้เล่น
local function updateLeaderboard(player, coins, level, kills)
    local userId = player.UserId
    
    -- อัพเดทแต่ละ leaderboard
    local function safeSet(store, value)
        pcall(function()
            store:SetAsync(userId, math.floor(value))
        end)
    end
    
    safeSet(coinsLeaderboard, coins)
    safeSet(levelLeaderboard, level)
    safeSet(killsLeaderboard, kills)
end

-- ดึงข้อมูล top players
local function getTopPlayers(store, topCount)
    local success, pages = pcall(function()
        return store:GetSortedAsync(false, topCount)
    end)
    
    if not success then
        return {}
    end
    
    local results = {}
    local currentPage = pages:GetCurrentPage()
    
    for rank, entry in ipairs(currentPage) do
        local userId = entry.key
        local value = entry.value
        
        -- ดึงชื่อผู้เล่น
        local success, name = pcall(function()
            return Players:GetNameFromUserIdAsync(tonumber(userId))
        end)
        
        if success then
            table.insert(results, {
                rank = rank,
                name = name,
                userId = userId,
                value = value
            })
        end
    end
    
    return results
end

-- ตัวอย่างการดึงข้อมูล leaderboard
local function displayLeaderboard()
    print("=== Top 10 Coins ===")
    local topCoins = getTopPlayers(coinsLeaderboard, 10)
    for _, entry in ipairs(topCoins) do
        print(string.format("#%d %s: %d เหรียญ", entry.rank, entry.name, entry.value))
    end
end

-- อัพเดท leaderboard ทุก 5 นาที
task.spawn(function()
    while true do
        task.wait(300)
        for _, player in ipairs(Players:GetPlayers()) do
            -- ดึงข้อมูลจาก ProfileManager
            -- และอัพเดท leaderboard
        end
    end
end)
```

---

## ส่วนที่ 4: DataStore Versioning และ Migration

```lua
-- Script: DataMigration (ModuleScript)
-- ระบบ migration ข้อมูลเมื่อมีการอัพเดท

local DataMigration = {}

-- กำหนด migrations
local migrations = {
    -- version 1 -> 2: เพิ่ม field ใหม่
    [1] = function(data)
        if not data.Settings then
            data.Settings = {
                MusicVolume = 0.5,
                SFXVolume = 0.5,
            }
        end
        data.Version = 2
        return data
    end,
    
    -- version 2 -> 3: เปลี่ยนโครงสร้าง inventory
    [2] = function(data)
        -- แปลง array inventory เป็น dictionary
        if type(data.Inventory) == "table" then
            local oldInventory = data.Inventory
            data.Inventory = {}
            for _, itemId in ipairs(oldInventory) do
                data.Inventory[itemId] = (data.Inventory[itemId] or 0) + 1
            end
        end
        data.Version = 3
        return data
    end,
}

local CURRENT_VERSION = 3

-- ดำเนิน migration
function DataMigration:Migrate(data)
    local version = data.Version or 1
    
    while version < CURRENT_VERSION do
        local migration = migrations[version]
        if migration then
            local success, result = pcall(migration, data)
            if success then
                data = result
                version = data.Version
                print(string.format("[Migration] อัพเกรดจาก v%d เป็น v%d", version - 1, version))
            else
                warn(string.format("[Migration] ล้มเหลวที่ version %d: %s", version, tostring(result)))
                break
            end
        else
            break
        end
    end
    
    return data
end

return DataMigration
```

---

## ส่วนที่ 5: DataStore Backup System

```lua
-- Script: BackupSystem (ModuleScript)
-- ระบบสำรองข้อมูล

local DataStoreService = game:GetService("DataStoreService")

local BackupSystem = {}

-- สร้าง DataStore สำหรับ backup
local backupStore = DataStoreService:GetDataStore("PlayerBackups")

-- บันทึก backup
function BackupSystem:CreateBackup(player, data)
    local userId = player.UserId
    local timestamp = os.time()
    local backupKey = string.format("Backup_%d_%d", userId, timestamp)
    
    -- เก็บ backup 3 ชุดล่าสุด
    local metaKey = "BackupMeta_" .. userId
    
    local success, meta = pcall(function()
        return backupStore:GetAsync(metaKey) or {backups = {}}
    end)
    
    if not success then
        meta = {backups = {}}
    end
    
    -- บันทึก backup ใหม่
    pcall(function()
        backupStore:SetAsync(backupKey, {
            data = data,
            timestamp = timestamp,
            playerId = userId
        })
    end)
    
    -- อัพเดท metadata
    table.insert(meta.backups, 1, {
        key = backupKey,
        timestamp = timestamp
    })
    
    -- เก็บแค่ 3 ชุดล่าสุด
    while #meta.backups > 3 do
        local old = table.remove(meta.backups)
        -- ลบ backup เก่า
        pcall(function()
            backupStore:RemoveAsync(old.key)
        end)
    end
    
    pcall(function()
        backupStore:SetAsync(metaKey, meta)
    end)
    
    print(string.format("[Backup] สร้าง backup สำเร็จ: %s", player.Name))
end

-- กู้คืนข้อมูล backup ล่าสุด
function BackupSystem:RestoreLatestBackup(player)
    local userId = player.UserId
    local metaKey = "BackupMeta_" .. userId
    
    local success, meta = pcall(function()
        return backupStore:GetAsync(metaKey)
    end)
    
    if not success or not meta or #meta.backups == 0 then
        return nil, "ไม่พบ backup"
    end
    
    local latestBackup = meta.backups[1]
    local backupSuccess, backupData = pcall(function()
        return backupStore:GetAsync(latestBackup.key)
    end)
    
    if backupSuccess and backupData then
        return backupData.data, nil
    end
    
    return nil, "ไม่สามารถโหลด backup ได้"
end

return BackupSystem
```

---

## ส่วนที่ 6: Client-Side Data Handling

```lua
-- LocalScript: PlayerDataClient (StarterPlayerScripts)
-- จัดการข้อมูลฝั่ง Client

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local player = Players.LocalPlayer

-- RemoteEvents สำหรับการสื่อสาร
local remotes = ReplicatedStorage:WaitForChild("Remotes")
local getDataEvent = remotes:WaitForChild("GetData")
local updateDataEvent = remotes:WaitForChild("UpdateData")

-- เก็บข้อมูลในฝั่ง Client
local localData = {}

-- โหลดข้อมูลเมื่อเริ่มต้น
local function loadData()
    local success, data = pcall(function()
        return getDataEvent:InvokeServer()
    end)
    
    if success and data then
        localData = data
        print("[Client] โหลดข้อมูลสำเร็จ")
        print(string.format("  Level: %d | Coins: %d | Gems: %d",
            data.Level or 1,
            data.Coins or 0,
            data.Gems or 0
        ))
    else
        warn("[Client] ไม่สามารถโหลดข้อมูลได้")
    end
end

-- ฟังการอัพเดทจาก Server
updateDataEvent.OnClientEvent:Connect(function(updates)
    for key, value in pairs(updates) do
        localData[key] = value
    end
    -- อัพเดท UI
end)

-- เริ่มทำงาน
loadData()
```

---

## ส่วนที่ 7: ระบบ Transaction (การทำธุรกรรม)

```lua
-- ModuleScript: TransactionSystem
-- ระบบธุรกรรมที่ปลอดภัย

local DataStoreService = game:GetService("DataStoreService")
local transactionStore = DataStoreService:GetDataStore("Transactions")

local TransactionSystem = {}

-- สร้างการทำธุรกรรม
function TransactionSystem:CreateTransaction(playerId, transactionType, amount, description)
    local transactionId = string.format("TXN_%d_%d_%s",
        playerId,
        os.time(),
        math.random(1000, 9999)
    )
    
    local transaction = {
        id = transactionId,
        playerId = playerId,
        type = transactionType,
        amount = amount,
        description = description,
        timestamp = os.time(),
        status = "pending"
    }
    
    -- บันทึกการทำธุรกรรม
    local success = pcall(function()
        transactionStore:SetAsync(transactionId, transaction)
    end)
    
    if success then
        return transactionId
    end
    return nil
end

-- ยืนยันการทำธุรกรรม
function TransactionSystem:CompleteTransaction(transactionId)
    pcall(function()
        transactionStore:UpdateAsync(transactionId, function(data)
            if data then
                data.status = "completed"
                data.completedAt = os.time()
            end
            return data
        end)
    end)
end

-- ยกเลิกการทำธุรกรรม
function TransactionSystem:CancelTransaction(transactionId, reason)
    pcall(function()
        transactionStore:UpdateAsync(transactionId, function(data)
            if data then
                data.status = "cancelled"
                data.cancelReason = reason
                data.cancelledAt = os.time()
            end
            return data
        end)
    end)
end

return TransactionSystem
```

---

## ส่วนที่ 8: แบบฝึกหัดและโปรเจกต์

### แบบฝึกหัดที่ 1: สร้างระบบ Inventory

สร้างระบบ inventory ที่:
1. บันทึกไอเทมของผู้เล่น
2. มี stack limit (เช่น สูงสุด 99 ชิ้นต่อประเภท)
3. รองรับการแลกเปลี่ยนระหว่างผู้เล่น
4. มี transaction log

### แบบฝึกหัดที่ 2: Leaderboard ขั้นสูง

สร้าง leaderboard ที่:
1. แสดงผู้เล่น top 100
2. แสดงอันดับของผู้เล่นปัจจุบัน
3. มี category หลายอย่าง (coins, level, kills)
4. อัพเดทแบบ real-time

### แบบฝึกหัดที่ 3: Data Backup และ Restore

สร้างระบบที่:
1. สำรองข้อมูลอัตโนมัติทุกวัน
2. Admin สามารถ restore ข้อมูลผู้เล่นได้
3. มีประวัติ backup 7 วัน
4. แจ้งเตือนเมื่อ restore สำเร็จ

---

## สรุปบทที่ 81

ในบทนี้เราได้เรียนรู้:

1. **DataStore ขั้นสูง** - การจัดการ error, retry logic, และ session management
2. **ProfileService** - library ที่แก้ปัญหา DataStore ได้อย่างมีประสิทธิภาพ
3. **OrderedDataStore** - สำหรับระบบ leaderboard
4. **Data Versioning** - การ migrate ข้อมูลเมื่ออัพเดท schema
5. **Backup System** - การสำรองข้อมูลสำคัญ
6. **Transaction System** - การทำธุรกรรมที่ปลอดภัย

### คำแนะนำสำหรับ Production

- ใช้ ProfileService สำหรับเกมจริงเสมอ
- ทดสอบ edge cases (ออกกลางคัน, เน็ตหลุด)
- มีระบบ monitoring สำหรับ DataStore errors
- อย่าเก็บข้อมูลที่ sensitive (รหัสผ่าน, ข้อมูลส่วนตัว)
- ปฏิบัติตาม GDPR/COPPA guidelines

---

*บทถัดไป: Part 82 - Analytics System (ระบบวิเคราะห์เกม)*
