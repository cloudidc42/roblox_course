# Part 41: DataStore พื้นฐาน - การบันทึกข้อมูลผู้เล่น

## บทนำ

DataStore เป็นระบบที่ Roblox ให้บริการสำหรับการบันทึกและโหลดข้อมูลผู้เล่นอย่างถาวร ข้อมูลที่บันทึกจะยังคงอยู่แม้ผู้เล่นออกจากเกมและกลับมาใหม่ นี่คือระบบที่สำคัญอย่างมากสำหรับเกมที่ต้องการระบบความก้าวหน้า เช่น เงิน, ระดับ, ไอเทม และสถิติต่างๆ

## ทำความเข้าใจ DataStore

### DataStore คืออะไร?

```
DataStoreService
    └── DataStore (ชื่อ)
            └── Key (รหัส เช่น userId)
                    └── Value (ค่า เช่น { coins = 100, level = 5 })
```

DataStore ทำงานเหมือนฐานข้อมูล key-value โดย:
- **DataStoreService** คือ service หลัก
- **DataStore** คือตารางข้อมูล (เราตั้งชื่อเอง)
- **Key** คือรหัสสำหรับแต่ละผู้เล่น (มักใช้ UserId)
- **Value** คือข้อมูลที่เราบันทึก (string, number, table)

### ข้อจำกัดสำคัญ

| ข้อจำกัด | ค่า |
|----------|-----|
| ขนาดข้อมูลสูงสุดต่อ key | 4 MB |
| จำนวน request ต่อนาที | 60 + 10 × จำนวนผู้เล่น |
| ขนาด key สูงสุด | 50 ตัวอักษร |
| ขนาด value สูงสุด | 4 MB |

## การเปิดใช้งาน DataStore

ก่อนใช้งาน DataStore ต้องเปิดใช้งานใน Studio:
1. ไปที่ **Home** > **Game Settings**
2. คลิกที่แท็บ **Security**
3. เปิด **Enable Studio Access to API Services**

## การเขียนโค้ด DataStore พื้นฐาน

### โครงสร้างพื้นฐาน

```lua
-- Script นี้ต้องอยู่ใน ServerScriptService
-- DataStore ทำงานได้เฉพาะฝั่ง Server เท่านั้น!

local DataStoreService = game:GetService("DataStoreService")
local Players = game:GetService("Players")

-- สร้าง DataStore ชื่อ "PlayerData"
-- ชื่อนี้จะเป็นตัวระบุในระบบของ Roblox
local playerDataStore = DataStoreService:GetDataStore("PlayerData")

print("DataStore พร้อมใช้งาน!")
```

### การบันทึกข้อมูล (SetAsync)

```lua
local DataStoreService = game:GetService("DataStoreService")
local playerDataStore = DataStoreService:GetDataStore("PlayerData")

-- ฟังก์ชันบันทึกข้อมูลผู้เล่น
local function savePlayerData(player)
    -- สร้าง key จาก UserId (ไม่ใช้ชื่อ เพราะชื่อเปลี่ยนได้)
    local key = "Player_" .. player.UserId
    
    -- ข้อมูลที่ต้องการบันทึก
    local data = {
        coins = 100,        -- จำนวนเหรียญ
        level = 1,          -- ระดับ
        experience = 0,     -- ประสบการณ์
        playTime = 0,       -- เวลาเล่น (วินาที)
        joinDate = os.time() -- วันที่เข้าร่วม
    }
    
    -- บันทึกด้วย pcall เพื่อป้องกัน error
    local success, errorMessage = pcall(function()
        playerDataStore:SetAsync(key, data)
    end)
    
    if success then
        print("บันทึกข้อมูลของ " .. player.Name .. " สำเร็จ!")
    else
        warn("บันทึกข้อมูลล้มเหลว: " .. errorMessage)
    end
end

-- ทดสอบการบันทึก
local Players = game:GetService("Players")
Players.PlayerAdded:Connect(function(player)
    savePlayerData(player)
end)
```

### การโหลดข้อมูล (GetAsync)

```lua
local DataStoreService = game:GetService("DataStoreService")
local Players = game:GetService("Players")
local playerDataStore = DataStoreService:GetDataStore("PlayerData")

-- ฟังก์ชันโหลดข้อมูลผู้เล่น
local function loadPlayerData(player)
    local key = "Player_" .. player.UserId
    
    local success, data = pcall(function()
        return playerDataStore:GetAsync(key)
    end)
    
    if success then
        if data then
            -- ผู้เล่นเคยเล่นมาก่อน - โหลดข้อมูลเก่า
            print("โหลดข้อมูลของ " .. player.Name .. " สำเร็จ!")
            print("  เหรียญ: " .. data.coins)
            print("  ระดับ: " .. data.level)
            return data
        else
            -- ผู้เล่นใหม่ - สร้างข้อมูลเริ่มต้น
            print(player.Name .. " เป็นผู้เล่นใหม่!")
            local defaultData = {
                coins = 0,
                level = 1,
                experience = 0,
                playTime = 0,
                joinDate = os.time()
            }
            return defaultData
        end
    else
        -- เกิด error ในการโหลด
        warn("โหลดข้อมูลล้มเหลว: " .. data)
        -- คืนค่าเริ่มต้นเพื่อไม่ให้เกมค้าง
        return {
            coins = 0,
            level = 1,
            experience = 0,
            playTime = 0,
            joinDate = os.time()
        }
    end
end

Players.PlayerAdded:Connect(function(player)
    local playerData = loadPlayerData(player)
    -- เก็บข้อมูลไว้ใช้งาน...
end)
```

## ระบบ DataStore แบบสมบูรณ์

### การสร้างระบบจัดการข้อมูลผู้เล่น

```lua
-- ServerScriptService/PlayerDataManager.lua
-- ระบบจัดการข้อมูลผู้เล่นแบบสมบูรณ์

local DataStoreService = game:GetService("DataStoreService")
local Players = game:GetService("Players")

-- ===== การตั้งค่า =====
local DATA_STORE_NAME = "PlayerData_v1"  -- เพิ่ม version เพื่อ reset ได้ง่าย
local SAVE_INTERVAL = 60  -- บันทึกทุก 60 วินาที
local MAX_RETRY = 3       -- จำนวนครั้งที่ลองใหม่เมื่อ error

-- ===== DataStore =====
local playerDataStore = DataStoreService:GetDataStore(DATA_STORE_NAME)

-- ===== เก็บข้อมูลในหน่วยความจำ =====
-- ใช้ table นี้เก็บข้อมูลระหว่างที่เล่น (ไม่ต้องโหลดจาก DataStore ทุกครั้ง)
local playerDataCache = {}

-- ===== ค่าเริ่มต้นสำหรับผู้เล่นใหม่ =====
local DEFAULT_DATA = {
    -- เศรษฐกิจ
    coins = 0,
    gems = 0,
    
    -- ความก้าวหน้า
    level = 1,
    experience = 0,
    
    -- สถิติ
    totalPlayTime = 0,
    totalKills = 0,
    totalDeaths = 0,
    
    -- การตั้งค่า
    musicVolume = 0.5,
    sfxVolume = 0.8,
    
    -- ข้อมูลระบบ
    firstJoin = os.time(),
    lastLogin = os.time(),
    version = 1  -- สำหรับ migration ข้อมูลในอนาคต
}

-- ===== ฟังก์ชันช่วยเหลือ =====

-- Deep copy table (คัดลอก table แบบลึก)
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

-- Merge ข้อมูลเก่ากับ default (สำหรับ field ที่เพิ่งเพิ่ม)
local function mergeWithDefaults(data)
    local merged = deepCopy(DEFAULT_DATA)
    for key, value in pairs(data) do
        merged[key] = value
    end
    return merged
end

-- ===== ฟังก์ชันหลัก =====

-- โหลดข้อมูลผู้เล่น (พร้อม retry)
local function loadData(player)
    local key = "Player_" .. player.UserId
    local data = nil
    local lastError = nil
    
    -- ลองโหลดหลายครั้ง
    for attempt = 1, MAX_RETRY do
        local success, result = pcall(function()
            return playerDataStore:GetAsync(key)
        end)
        
        if success then
            data = result
            break  -- โหลดสำเร็จ หยุดลอง
        else
            lastError = result
            warn(string.format(
                "ลองโหลดครั้งที่ %d/%d ล้มเหลว: %s",
                attempt, MAX_RETRY, result
            ))
            
            if attempt < MAX_RETRY then
                task.wait(2)  -- รอก่อนลองใหม่
            end
        end
    end
    
    -- ตรวจสอบผลลัพธ์
    if data then
        -- ผู้เล่นเก่า - merge กับ default เพื่อเพิ่ม field ใหม่
        data = mergeWithDefaults(data)
        data.lastLogin = os.time()  -- อัพเดทเวลา login
        print("✓ โหลดข้อมูล " .. player.Name .. " สำเร็จ (Level " .. data.level .. ")")
    else
        if lastError then
            warn("✗ โหลดข้อมูลล้มเหลวทั้งหมด " .. MAX_RETRY .. " ครั้ง")
        end
        -- สร้างข้อมูลใหม่
        data = deepCopy(DEFAULT_DATA)
        print("✓ สร้างข้อมูลใหม่สำหรับ " .. player.Name)
    end
    
    return data
end

-- บันทึกข้อมูลผู้เล่น (พร้อม retry)
local function saveData(player, showMessage)
    local key = "Player_" .. player.UserId
    local data = playerDataCache[player.UserId]
    
    if not data then
        warn("ไม่พบข้อมูลใน cache สำหรับ " .. player.Name)
        return false
    end
    
    -- อัพเดทเวลาเล่น
    data.totalPlayTime = data.totalPlayTime + (os.time() - (data._sessionStart or os.time()))
    data._sessionStart = os.time()
    
    local success = false
    
    for attempt = 1, MAX_RETRY do
        local ok, err = pcall(function()
            playerDataStore:SetAsync(key, data)
        end)
        
        if ok then
            success = true
            break
        else
            warn(string.format("บันทึกครั้งที่ %d/%d ล้มเหลว: %s", attempt, MAX_RETRY, err))
            if attempt < MAX_RETRY then
                task.wait(2)
            end
        end
    end
    
    if showMessage then
        if success then
            print("✓ บันทึกข้อมูล " .. player.Name .. " สำเร็จ")
        else
            warn("✗ บันทึกข้อมูล " .. player.Name .. " ล้มเหลว!")
        end
    end
    
    return success
end

-- ===== เหตุการณ์ผู้เล่น =====

Players.PlayerAdded:Connect(function(player)
    -- โหลดข้อมูล
    local data = loadData(player)
    data._sessionStart = os.time()  -- บันทึกเวลาเริ่ม session
    
    -- เก็บใน cache
    playerDataCache[player.UserId] = data
    
    -- สร้าง leaderstats (แสดงในตาราง)
    local leaderstats = Instance.new("Folder")
    leaderstats.Name = "leaderstats"
    leaderstats.Parent = player
    
    local coinsValue = Instance.new("IntValue")
    coinsValue.Name = "Coins"
    coinsValue.Value = data.coins
    coinsValue.Parent = leaderstats
    
    local levelValue = Instance.new("IntValue")
    levelValue.Name = "Level"
    levelValue.Value = data.level
    levelValue.Parent = leaderstats
    
    -- อัพเดท leaderstats เมื่อข้อมูลเปลี่ยน
    -- (ในระบบจริงควรใช้ function แยก)
end)

Players.PlayerRemoving:Connect(function(player)
    -- บันทึกข้อมูลเมื่อผู้เล่นออก
    saveData(player, true)
    
    -- ลบออกจาก cache
    playerDataCache[player.UserId] = nil
end)

-- ===== บันทึกอัตโนมัติ =====
task.spawn(function()
    while true do
        task.wait(SAVE_INTERVAL)
        
        local playerCount = 0
        for _, player in ipairs(Players:GetPlayers()) do
            if playerDataCache[player.UserId] then
                saveData(player, false)
                playerCount = playerCount + 1
            end
        end
        
        if playerCount > 0 then
            print(string.format("🔄 บันทึกข้อมูลอัตโนมัติ: %d ผู้เล่น", playerCount))
        end
    end
end)

-- ===== บันทึกก่อนปิดเซิร์ฟเวอร์ =====
game:BindToClose(function()
    print("กำลังปิดเซิร์ฟเวอร์ - บันทึกข้อมูลทั้งหมด...")
    
    local saveCoroutines = {}
    
    for _, player in ipairs(Players:GetPlayers()) do
        -- บันทึกแบบ parallel
        table.insert(saveCoroutines, task.spawn(function()
            saveData(player, true)
        end))
    end
    
    -- รอให้ทุก coroutine เสร็จ
    for _, co in ipairs(saveCoroutines) do
        task.wait()
    end
    
    print("✓ บันทึกข้อมูลทั้งหมดเสร็จสิ้น")
    task.wait(2)  -- รอให้ข้อมูลบันทึกสมบูรณ์
end)

-- ===== API สำหรับ Script อื่น =====
-- ใช้ ModuleScript แทนในระบบจริง แต่นี่แสดงหลักการ

local PlayerDataManager = {}

function PlayerDataManager.getData(player)
    return playerDataCache[player.UserId]
end

function PlayerDataManager.addCoins(player, amount)
    local data = playerDataCache[player.UserId]
    if data then
        data.coins = data.coins + amount
        -- อัพเดท leaderstats
        local leaderstats = player:FindFirstChild("leaderstats")
        if leaderstats then
            local coinsValue = leaderstats:FindFirstChild("Coins")
            if coinsValue then
                coinsValue.Value = data.coins
            end
        end
        return data.coins
    end
    return nil
end

function PlayerDataManager.addExperience(player, amount)
    local data = playerDataCache[player.UserId]
    if not data then return end
    
    data.experience = data.experience + amount
    
    -- ตรวจสอบ level up
    local expNeeded = data.level * 100  -- ต้องการ EXP = level × 100
    
    while data.experience >= expNeeded do
        data.experience = data.experience - expNeeded
        data.level = data.level + 1
        expNeeded = data.level * 100
        
        -- แจ้งผู้เล่น
        print(player.Name .. " เลื่อนระดับเป็น Level " .. data.level .. "!")
        
        -- อัพเดท leaderstats
        local leaderstats = player:FindFirstChild("leaderstats")
        if leaderstats then
            local levelValue = leaderstats:FindFirstChild("Level")
            if levelValue then
                levelValue.Value = data.level
            end
        end
    end
end

return PlayerDataManager
```

## UpdateAsync - การอัพเดทข้อมูลอย่างปลอดภัย

```lua
-- UpdateAsync ดีกว่า SetAsync เมื่อหลาย server อาจเขียนข้อมูลพร้อมกัน
-- (สำคัญมากสำหรับเกมที่มีหลาย server)

local DataStoreService = game:GetService("DataStoreService")
local dataStore = DataStoreService:GetDataStore("PlayerCoins")

local function addCoinsToPlayer(userId, amount)
    local key = "Player_" .. userId
    
    local success, result = pcall(function()
        return dataStore:UpdateAsync(key, function(oldData)
            -- oldData คือข้อมูลปัจจุบันจาก DataStore
            -- ถ้าไม่มีข้อมูลเก่า ใช้ค่าเริ่มต้น
            oldData = oldData or { coins = 0 }
            
            -- ตรวจสอบความถูกต้อง
            if amount < 0 and oldData.coins < math.abs(amount) then
                -- ไม่มีเหรียญพอ - ยกเลิกการอัพเดทโดย return nil
                return nil
            end
            
            -- อัพเดทข้อมูล
            oldData.coins = oldData.coins + amount
            oldData.lastUpdated = os.time()
            
            -- คืนค่าข้อมูลที่อัพเดทแล้ว
            return oldData
        end)
    end)
    
    if success then
        if result then
            print("อัพเดทเหรียญสำเร็จ: " .. result.coins .. " เหรียญ")
            return result.coins
        else
            print("ยกเลิกการอัพเดท (เหรียญไม่พอ)")
            return nil
        end
    else
        warn("UpdateAsync ล้มเหลว: " .. result)
        return nil
    end
end

-- ตัวอย่างการใช้
-- addCoinsToPlayer(12345678, 50)   -- เพิ่ม 50 เหรียญ
-- addCoinsToPlayer(12345678, -30)  -- ลด 30 เหรียญ
```

## IncrementAsync - เพิ่มค่าตัวเลข

```lua
-- IncrementAsync ใช้สำหรับเพิ่มตัวเลขแบบ atomic
-- เหมาะสำหรับ: จำนวนครั้งที่เล่น, คะแนนรวม, สถิติ

local DataStoreService = game:GetService("DataStoreService")
local statsStore = DataStoreService:GetDataStore("GlobalStats")

local function incrementPlayCount(userId)
    local key = "PlayCount_" .. userId
    
    local success, newValue = pcall(function()
        return statsStore:IncrementAsync(key, 1)  -- เพิ่มทีละ 1
    end)
    
    if success then
        print("เล่นไปทั้งหมด " .. newValue .. " ครั้ง")
        return newValue
    else
        warn("IncrementAsync ล้มเหลว: " .. newValue)
        return nil
    end
end

-- สำหรับ Global Leaderboard
local function addToGlobalScore(userId, score)
    local key = "TotalScore_" .. userId
    
    local success, newScore = pcall(function()
        return statsStore:IncrementAsync(key, score)
    end)
    
    if success then
        return newScore
    else
        return nil
    end
end
```

## OrderedDataStore - ตารางคะแนน

```lua
-- OrderedDataStore ใช้สำหรับการจัดอันดับ (Leaderboard)
-- รองรับเฉพาะตัวเลขจำนวนเต็ม (integer)

local DataStoreService = game:GetService("DataStoreService")
local Players = game:GetService("Players")

local coinsLeaderboard = DataStoreService:GetOrderedDataStore("CoinsLeaderboard")

-- บันทึกคะแนนลง Leaderboard
local function updateLeaderboard(player, coins)
    local key = "Player_" .. player.UserId
    
    local success, err = pcall(function()
        coinsLeaderboard:SetAsync(key, coins)
    end)
    
    if not success then
        warn("อัพเดท Leaderboard ล้มเหลว: " .. err)
    end
end

-- ดึงอันดับสูงสุด 10 อันดับ
local function getTopPlayers(count)
    count = count or 10
    
    local success, pages = pcall(function()
        -- true = เรียงจากมากไปน้อย
        return coinsLeaderboard:GetSortedAsync(true, count)
    end)
    
    if not success then
        warn("ดึง Leaderboard ล้มเหลว: " .. pages)
        return {}
    end
    
    local topPlayers = {}
    local currentPage = pages:GetCurrentPage()
    
    for rank, entry in ipairs(currentPage) do
        table.insert(topPlayers, {
            rank = rank,
            userId = tonumber(entry.key:match("Player_(%d+)")),
            score = entry.value
        })
    end
    
    return topPlayers
end

-- แสดงตาราง Leaderboard
local function displayLeaderboard()
    print("=== TOP 10 COINS ===")
    local topPlayers = getTopPlayers(10)
    
    for _, entry in ipairs(topPlayers) do
        -- ดึงชื่อจาก UserId (ต้องใช้ async)
        local success, username = pcall(function()
            return Players:GetNameFromUserIdAsync(entry.userId)
        end)
        
        local name = success and username or "Unknown"
        print(string.format("#%d %s - %d เหรียญ", entry.rank, name, entry.score))
    end
end

-- ทดสอบ
task.spawn(function()
    task.wait(3)
    displayLeaderboard()
end)
```

## การจัดการข้อผิดพลาดขั้นสูง

```lua
-- ระบบ retry ที่ชาญฉลาด
local function retryAsync(func, maxRetries, retryDelay)
    maxRetries = maxRetries or 3
    retryDelay = retryDelay or 1
    
    local lastError
    
    for attempt = 1, maxRetries do
        local success, result = pcall(func)
        
        if success then
            return true, result
        else
            lastError = result
            
            -- ตรวจสอบประเภท error
            if result:find("429") then
                -- Too Many Requests - รอนานขึ้น
                warn("Rate limited! รอ " .. (retryDelay * 2) .. " วินาที...")
                task.wait(retryDelay * 2)
            elseif result:find("503") then
                -- Service Unavailable
                warn("DataStore ไม่พร้อม รอ " .. retryDelay .. " วินาที...")
                task.wait(retryDelay)
            else
                -- Error อื่น
                warn("Error: " .. result)
                task.wait(retryDelay)
            end
        end
    end
    
    return false, lastError
end

-- ใช้งาน
local DataStoreService = game:GetService("DataStoreService")
local myStore = DataStoreService:GetDataStore("MyStore")

local function safeGet(key)
    local success, result = retryAsync(function()
        return myStore:GetAsync(key)
    end, 3, 1)
    
    if success then
        return result
    else
        warn("ไม่สามารถโหลดข้อมูลได้: " .. (result or "Unknown error"))
        return nil
    end
end

local function safeSet(key, value)
    local success, result = retryAsync(function()
        myStore:SetAsync(key, value)
    end, 3, 1)
    
    if not success then
        warn("ไม่สามารถบันทึกข้อมูลได้: " .. (result or "Unknown error"))
    end
    
    return success
end
```

## การทดสอบใน Studio

```lua
-- Script สำหรับทดสอบ DataStore
-- วางใน Command Bar หรือ Script ชั่วคราว

local DataStoreService = game:GetService("DataStoreService")
local testStore = DataStoreService:GetDataStore("TestStore")

-- ทดสอบ 1: Set และ Get
print("=== ทดสอบ DataStore ===")

-- Set ข้อมูล
local setSuccess, setErr = pcall(function()
    testStore:SetAsync("TestKey", { value = 42, name = "Test" })
end)

if setSuccess then
    print("✓ SetAsync สำเร็จ")
else
    print("✗ SetAsync ล้มเหลว: " .. setErr)
end

-- Get ข้อมูล
local getSuccess, getData = pcall(function()
    return testStore:GetAsync("TestKey")
end)

if getSuccess and getData then
    print("✓ GetAsync สำเร็จ: value=" .. getData.value)
else
    print("✗ GetAsync ล้มเหลว")
end

-- ทดสอบ 2: UpdateAsync
local updateSuccess, _ = pcall(function()
    testStore:UpdateAsync("TestKey", function(old)
        old = old or { value = 0 }
        old.value = old.value + 1
        return old
    end)
end)

print(updateSuccess and "✓ UpdateAsync สำเร็จ" or "✗ UpdateAsync ล้มเหลว")

-- ลบข้อมูลทดสอบ
local removeSuccess, _ = pcall(function()
    testStore:RemoveAsync("TestKey")
end)

print(removeSuccess and "✓ RemoveAsync สำเร็จ" or "✗ RemoveAsync ล้มเหลว")

print("=== การทดสอบเสร็จสิ้น ===")
```

## ข้อผิดพลาดที่พบบ่อยและวิธีแก้ไข

### 1. ไม่ได้ใช้ pcall

```lua
-- ❌ อันตราย - ถ้า error จะทำให้ script หยุดทำงาน
local data = dataStore:GetAsync(key)

-- ✅ ถูกต้อง - ใช้ pcall เสมอ
local success, data = pcall(function()
    return dataStore:GetAsync(key)
end)
```

### 2. บันทึกบ่อยเกินไป

```lua
-- ❌ อันตราย - อาจถูก rate limit
Players.PlayerAdded:Connect(function(player)
    -- ทุกวินาที = 60 ครั้ง/นาที (เกินลิมิต!)
    while true do
        task.wait(1)
        saveData(player)
    end
end)

-- ✅ ถูกต้อง - บันทึกทุก 60 วินาที
local function startAutoSave(player)
    task.spawn(function()
        while player.Parent do
            task.wait(60)
            if player.Parent then  -- ตรวจสอบอีกครั้งหลัง wait
                saveData(player)
            end
        end
    end)
end
```

### 3. ลืมจัดการกับ nil

```lua
-- ❌ อันตราย - ถ้า data เป็น nil จะ error
local data = dataStore:GetAsync(key)
print(data.coins)  -- ERROR ถ้า data เป็น nil

-- ✅ ถูกต้อง - ตรวจสอบก่อนใช้
local success, data = pcall(function()
    return dataStore:GetAsync(key)
end)

if success and data then
    print(data.coins)
else
    print("ไม่มีข้อมูล หรือโหลดล้มเหลว")
end
```

### 4. ใช้ชื่อผู้เล่นเป็น Key

```lua
-- ❌ อันตราย - ชื่อผู้เล่นเปลี่ยนได้
local key = player.Name  -- "PlayerName" เปลี่ยนได้!

-- ✅ ถูกต้อง - ใช้ UserId ซึ่งไม่เปลี่ยน
local key = "Player_" .. player.UserId  -- 123456789 ไม่เปลี่ยน
```

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: ระบบบันทึกคะแนนง่ายๆ
สร้าง script ที่:
1. บันทึกคะแนนสูงสุดของผู้เล่น
2. โหลดคะแนนเมื่อผู้เล่นเข้ามา
3. อัพเดทถ้าคะแนนใหม่สูงกว่าเก่า
4. แสดงใน leaderstats

```lua
-- เฉลย: ระบบบันทึกคะแนนสูงสุด
local DataStoreService = game:GetService("DataStoreService")
local Players = game:GetService("Players")
local highScoreStore = DataStoreService:GetDataStore("HighScores")

local function loadHighScore(player)
    local key = "HS_" .. player.UserId
    local success, score = pcall(function()
        return highScoreStore:GetAsync(key)
    end)
    return (success and score) or 0
end

local function updateHighScore(player, newScore)
    local key = "HS_" .. player.UserId
    local success, finalScore = pcall(function()
        return highScoreStore:UpdateAsync(key, function(oldScore)
            oldScore = oldScore or 0
            if newScore > oldScore then
                return newScore  -- อัพเดทถ้าสูงกว่า
            end
            return nil  -- ไม่อัพเดทถ้าต่ำกว่า
        end)
    end)
    
    if success then
        print(player.Name .. " คะแนนสูงสุด: " .. (finalScore or 0))
    end
end

Players.PlayerAdded:Connect(function(player)
    local highScore = loadHighScore(player)
    
    local leaderstats = Instance.new("Folder")
    leaderstats.Name = "leaderstats"
    leaderstats.Parent = player
    
    local hsValue = Instance.new("IntValue")
    hsValue.Name = "HighScore"
    hsValue.Value = highScore
    hsValue.Parent = leaderstats
end)
```

### แบบฝึกหัดที่ 2: ระบบนับวันติดกัน
สร้างระบบที่นับว่าผู้เล่นเล่นติดกันกี่วัน (daily streak)

### แบบฝึกหัดที่ 3: ระบบ Backup
สร้างระบบที่บันทึกข้อมูลสำรอง (ใช้ DataStore ชื่อต่างกัน) ทุกวัน

## สรุป

DataStore เป็นระบบสำคัญที่ทุกเกม Roblox ต้องใช้ หลักการสำคัญที่ต้องจำ:

1. **ใช้ pcall เสมอ** - DataStore อาจ fail ได้ทุกเมื่อ
2. **ใช้ UserId เป็น key** - ไม่ใช้ชื่อผู้เล่น
3. **บันทึกเมื่อผู้เล่นออก** - ใช้ PlayerRemoving
4. **บันทึกก่อนปิด server** - ใช้ BindToClose
5. **อย่าบันทึกบ่อยเกิน** - มี rate limit
6. **ใช้ UpdateAsync** สำหรับการอัพเดทที่ต้องการความถูกต้อง

ในบทถัดไปเราจะเรียนรู้ DataStore ขั้นสูง รวมถึง pattern ที่ใช้ในการพัฒนาเกมจริง
