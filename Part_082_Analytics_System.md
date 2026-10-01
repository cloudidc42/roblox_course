# Part 82: Analytics System - ระบบวิเคราะห์และติดตามเกม

## บทนำ

Analytics หรือระบบวิเคราะห์ข้อมูลเป็นสิ่งสำคัญมากในการพัฒนาเกมให้ประสบความสำเร็จ ช่วยให้เราเข้าใจพฤติกรรมผู้เล่น ค้นพบปัญหา และปรับปรุงเกมได้อย่างมีข้อมูลรองรับ

## ทำไม Analytics ถึงสำคัญ?

- **ค้นหาจุดตาย (Death Spots)** - รู้ว่าผู้เล่นตายที่ไหนมากที่สุด
- **Funnel Analysis** - ผู้เล่นออกจากเกมตอนไหน
- **Monetization Insights** - ผู้เล่นซื้ออะไร เมื่อไหร่
- **Performance Monitoring** - ตรวจสอบ lag, crashes
- **A/B Testing** - ทดสอบฟีเจอร์ใหม่

---

## ส่วนที่ 1: ระบบ Analytics พื้นฐาน

### 1.1 Event Tracking System

```lua
-- ModuleScript: AnalyticsManager (ServerStorage)
-- ระบบติดตาม events ในเกม

local DataStoreService = game:GetService("DataStoreService")
local Players = game:GetService("Players")
local HttpService = game:GetService("HttpService")

-- DataStore สำหรับเก็บ analytics
local analyticsStore = DataStoreService:GetDataStore("Analytics_v1")
local sessionStore = DataStoreService:GetDataStore("Sessions_v1")

-- ตัวแปรระดับเซิร์ฟเวอร์
local serverSessionId = HttpService:GenerateGUID(false)
local serverStartTime = os.time()
local eventBuffer = {} -- buffer ก่อน flush
local BUFFER_SIZE = 50 -- flush เมื่อมี 50 events
local FLUSH_INTERVAL = 30 -- หรือทุก 30 วินาที

-- =====================================
-- Event Types
-- =====================================

local EventType = {
    -- Gameplay Events
    PLAYER_JOIN = "player_join",
    PLAYER_LEAVE = "player_leave",
    PLAYER_DEATH = "player_death",
    PLAYER_LEVELUP = "player_levelup",
    PLAYER_PURCHASE = "player_purchase",
    
    -- UI Events  
    BUTTON_CLICK = "button_click",
    MENU_OPEN = "menu_open",
    MENU_CLOSE = "menu_close",
    
    -- Game Events
    QUEST_START = "quest_start",
    QUEST_COMPLETE = "quest_complete",
    ACHIEVEMENT_UNLOCK = "achievement_unlock",
    BOSS_KILL = "boss_kill",
    
    -- Economy Events
    ITEM_BUY = "item_buy",
    ITEM_SELL = "item_sell",
    COINS_EARN = "coins_earn",
    COINS_SPEND = "coins_spend",
    
    -- Technical Events
    ERROR = "error",
    PERFORMANCE_ISSUE = "performance_issue",
}

-- =====================================
-- Analytics Manager
-- =====================================

local AnalyticsManager = {}
AnalyticsManager.EventType = EventType

-- สร้าง event
function AnalyticsManager:Track(player, eventType, properties)
    properties = properties or {}
    
    local event = {
        -- Identifiers
        eventId = HttpService:GenerateGUID(false),
        sessionId = serverSessionId,
        
        -- Player info
        playerId = player and player.UserId or 0,
        playerName = player and player.Name or "Unknown",
        
        -- Event data
        eventType = eventType,
        properties = properties,
        
        -- Metadata
        timestamp = os.time(),
        serverTime = tick(),
        gameId = game.GameId,
        placeId = game.PlaceId,
        placeVersion = game.PlaceVersion,
    }
    
    -- เพิ่มเข้า buffer
    table.insert(eventBuffer, event)
    
    -- Flush ถ้า buffer เต็ม
    if #eventBuffer >= BUFFER_SIZE then
        self:FlushBuffer()
    end
    
    return event.eventId
end

-- บันทึก events ใน buffer ไปยัง DataStore
function AnalyticsManager:FlushBuffer()
    if #eventBuffer == 0 then return end
    
    local eventsToSave = eventBuffer
    eventBuffer = {}
    
    -- บันทึกเป็น batch
    local batchKey = string.format("Events_%d_%s",
        os.time(),
        serverSessionId:sub(1, 8)
    )
    
    pcall(function()
        analyticsStore:SetAsync(batchKey, {
            events = eventsToSave,
            savedAt = os.time(),
            serverId = serverSessionId,
        })
    end)
    
    print(string.format("[Analytics] บันทึก %d events", #eventsToSave))
end

-- Flush อัตโนมัติ
task.spawn(function()
    while true do
        task.wait(FLUSH_INTERVAL)
        AnalyticsManager:FlushBuffer()
    end
end)

-- Flush เมื่อ server ปิด
game:BindToClose(function()
    AnalyticsManager:FlushBuffer()
end)

return AnalyticsManager
```

### 1.2 Session Tracking

```lua
-- ModuleScript: SessionTracker
-- ติดตาม session ของผู้เล่น

local Players = game:GetService("Players")
local DataStoreService = game:GetService("DataStoreService")

local sessionStore = DataStoreService:GetDataStore("PlayerSessions")

local SessionTracker = {}
local activeSessions = {}

-- เริ่ม session ใหม่
function SessionTracker:StartSession(player)
    local userId = player.UserId
    local sessionId = string.format("SES_%d_%d", userId, os.time())
    
    local session = {
        sessionId = sessionId,
        userId = userId,
        playerName = player.Name,
        startTime = os.time(),
        endTime = nil,
        duration = nil,
        
        -- สถิติระหว่าง session
        events = {},
        deathCount = 0,
        killCount = 0,
        coinsEarned = 0,
        coinsSpent = 0,
        levelsGained = 0,
        
        -- Device info (จาก client)
        platform = nil,
        locale = nil,
    }
    
    activeSessions[userId] = session
    
    -- บันทึกเริ่ม session
    pcall(function()
        sessionStore:SetAsync(sessionId, session)
    end)
    
    return sessionId
end

-- จบ session
function SessionTracker:EndSession(player)
    local userId = player.UserId
    local session = activeSessions[userId]
    
    if not session then return end
    
    session.endTime = os.time()
    session.duration = session.endTime - session.startTime
    
    -- บันทึก session สุดท้าย
    pcall(function()
        sessionStore:SetAsync(session.sessionId, session)
    end)
    
    -- บันทึกสถิติรวม
    local statsKey = "Stats_" .. userId
    pcall(function()
        sessionStore:UpdateAsync(statsKey, function(data)
            data = data or {
                totalSessions = 0,
                totalPlayTime = 0,
                averageSessionLength = 0,
            }
            
            data.totalSessions = data.totalSessions + 1
            data.totalPlayTime = data.totalPlayTime + session.duration
            data.averageSessionLength = data.totalPlayTime / data.totalSessions
            data.lastSession = os.time()
            
            return data
        end)
    end)
    
    activeSessions[userId] = nil
    
    print(string.format("[Session] ผู้เล่น %s เล่นนาน %d นาที",
        player.Name,
        math.floor(session.duration / 60)
    ))
end

-- อัพเดทข้อมูล session
function SessionTracker:UpdateSession(player, updates)
    local session = activeSessions[player.UserId]
    if not session then return end
    
    for key, value in pairs(updates) do
        if type(value) == "number" and type(session[key]) == "number" then
            session[key] = session[key] + value
        else
            session[key] = value
        end
    end
end

Players.PlayerAdded:Connect(function(player)
    SessionTracker:StartSession(player)
end)

Players.PlayerRemoving:Connect(function(player)
    SessionTracker:EndSession(player)
end)

return SessionTracker
```

---

## ส่วนที่ 2: Funnel Analysis

```lua
-- ModuleScript: FunnelAnalytics
-- ติดตาม funnel การเล่นเกม

local DataStoreService = game:GetService("DataStoreService")
local funnelStore = DataStoreService:GetDataStore("FunnelData")

local FunnelAnalytics = {}

-- กำหนด funnels
local FUNNELS = {
    -- Onboarding funnel
    onboarding = {
        "game_start",
        "tutorial_step1",
        "tutorial_step2", 
        "tutorial_complete",
        "first_quest_start",
        "first_quest_complete",
        "first_purchase_prompt",
        "first_purchase_complete",
    },
    
    -- Combat funnel
    combat = {
        "enter_combat_zone",
        "first_attack",
        "first_kill",
        "boss_encounter",
        "boss_damage",
        "boss_killed",
    },
    
    -- Shop funnel
    shop = {
        "shop_open",
        "item_viewed",
        "item_add_to_cart",
        "checkout_start",
        "purchase_complete",
    },
}

-- บันทึกขั้นตอน funnel
function FunnelAnalytics:TrackStep(player, funnelName, step)
    local userId = player.UserId
    local key = string.format("Funnel_%s_%d", funnelName, userId)
    
    pcall(function()
        funnelStore:UpdateAsync(key, function(data)
            data = data or {
                funnelName = funnelName,
                userId = userId,
                steps = {},
                startTime = os.time(),
            }
            
            -- บันทึกขั้นตอนนี้ถ้ายังไม่เคย
            if not data.steps[step] then
                data.steps[step] = {
                    completedAt = os.time(),
                    timeFromStart = os.time() - (data.startTime or os.time()),
                }
            end
            
            return data
        end)
    end)
end

-- คำนวณ conversion rate
function FunnelAnalytics:GetConversionRate(funnelName)
    -- ดึงข้อมูลทั้งหมดสำหรับ funnel นี้
    -- (ในความเป็นจริงต้องใช้ analytics service ภายนอก)
    local funnel = FUNNELS[funnelName]
    if not funnel then return nil end
    
    local stats = {
        funnelName = funnelName,
        steps = {},
    }
    
    -- คำนวณ conversion ของแต่ละขั้นตอน
    -- นี่เป็นตัวอย่าง - ในความเป็นจริงต้องดึงข้อมูลจาก DataStore
    for i, step in ipairs(funnel) do
        stats.steps[step] = {
            stepNumber = i,
            users = 0, -- จำนวนผู้ใช้ที่ผ่านขั้นนี้
            conversionRate = 0,
        }
    end
    
    return stats
end

return FunnelAnalytics
```

---

## ส่วนที่ 3: Heatmap System

```lua
-- ModuleScript: HeatmapSystem
-- ระบบ heatmap ตำแหน่งในเกม

local DataStoreService = game:GetService("DataStoreService")
local heatmapStore = DataStoreService:GetDataStore("Heatmap_v1")

local HeatmapSystem = {}

-- ความละเอียดของ grid (สตั้ดต่อช่อง)
local GRID_SIZE = 10

-- แปลง position เป็น grid coordinates
local function positionToGrid(position)
    return {
        x = math.floor(position.X / GRID_SIZE),
        y = math.floor(position.Y / GRID_SIZE),
        z = math.floor(position.Z / GRID_SIZE),
    }
end

-- บันทึกตำแหน่งผู้เล่น
function HeatmapSystem:RecordPosition(player, eventType)
    local character = player.Character
    if not character then return end
    
    local hrp = character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    
    local grid = positionToGrid(hrp.Position)
    local gridKey = string.format("%d_%d_%d", grid.x, grid.y, grid.z)
    
    -- บันทึกใน heatmap
    local storeKey = string.format("Heatmap_%s_%d", eventType, game.PlaceId)
    
    pcall(function()
        heatmapStore:IncrementAsync(
            storeKey .. "_" .. gridKey,
            1
        )
    end)
end

-- ตัวอย่างการใช้งาน: บันทึกตำแหน่งที่ตาย
local Players = game:GetService("Players")

Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function(character)
        local humanoid = character:WaitForChild("Humanoid")
        humanoid.Died:Connect(function()
            HeatmapSystem:RecordPosition(player, "death")
        end)
    end)
end)

-- บันทึกตำแหน่งปัจจุบันทุก 10 วินาที (player activity)
task.spawn(function()
    while true do
        task.wait(10)
        for _, player in ipairs(Players:GetPlayers()) do
            HeatmapSystem:RecordPosition(player, "activity")
        end
    end
end)

return HeatmapSystem
```

---

## ส่วนที่ 4: Performance Analytics

```lua
-- Script: PerformanceMonitor (Script ใน ServerScriptService)
-- ติดตาม performance ของเซิร์ฟเวอร์

local RunService = game:GetService("RunService")
local Players = game:GetService("Players")

local PerformanceMonitor = {}

local metrics = {
    fps = {},
    ping = {},
    memory = {},
    playerCount = {},
}

local function getServerStats()
    return {
        playerCount = #Players:GetPlayers(),
        serverTime = os.time(),
        memory = collectgarbage("count"), -- KB
        -- Roblox specific stats
        dataReceiveKbps = game:GetService("Stats").DataReceiveKbps,
        dataSendKbps = game:GetService("Stats").DataSendKbps,
        heartbeatTimeMs = game:GetService("Stats").HeartbeatTimeMs,
        instanceCount = game:GetService("Stats").InstanceCount,
    }
end

-- เก็บ metrics ทุก 5 นาที
task.spawn(function()
    while true do
        task.wait(300)
        
        local stats = getServerStats()
        
        -- เก็บข้อมูลไว้ใน buffer
        table.insert(metrics.playerCount, {
            time = stats.serverTime,
            value = stats.playerCount
        })
        
        -- จำกัดขนาด buffer
        if #metrics.playerCount > 100 then
            table.remove(metrics.playerCount, 1)
        end
        
        -- แจ้งเตือนถ้า performance ต่ำ
        if stats.heartbeatTimeMs > 100 then
            warn(string.format("[Performance] Heartbeat สูง: %.1fms", stats.heartbeatTimeMs))
        end
    end
end)

-- ฟังก์ชันดึงสถิติ
function PerformanceMonitor:GetCurrentStats()
    return getServerStats()
end

function PerformanceMonitor:GetHistoricalData(metricName)
    return metrics[metricName] or {}
end

return PerformanceMonitor
```

---

## ส่วนที่ 5: A/B Testing Framework

```lua
-- ModuleScript: ABTesting
-- ระบบทดสอบ A/B

local DataStoreService = game:GetService("DataStoreService")
local abStore = DataStoreService:GetDataStore("ABTests")

local ABTesting = {}

-- กำหนด experiments
local EXPERIMENTS = {
    -- ทดสอบการแสดงราคา
    shopPriceDisplay = {
        variants = {"original", "discounted", "bundled"},
        weights = {50, 30, 20}, -- เปอร์เซ็นต์
    },
    
    -- ทดสอบ tutorial ยาวหรือสั้น
    tutorialLength = {
        variants = {"short", "long"},
        weights = {50, 50},
    },
    
    -- ทดสอบ spawn location
    spawnLocation = {
        variants = {"center", "edge", "custom"},
        weights = {33, 33, 34},
    },
}

-- กำหนด variant ให้ผู้เล่น
function ABTesting:GetVariant(player, experimentName)
    local experiment = EXPERIMENTS[experimentName]
    if not experiment then return nil end
    
    local userId = player.UserId
    local key = string.format("AB_%s_%d", experimentName, userId)
    
    -- ตรวจสอบว่าเคยกำหนดแล้วหรือยัง
    local success, storedVariant = pcall(function()
        return abStore:GetAsync(key)
    end)
    
    if success and storedVariant then
        return storedVariant
    end
    
    -- กำหนด variant ใหม่
    local random = math.random(100)
    local cumulative = 0
    local assignedVariant = experiment.variants[1]
    
    for i, weight in ipairs(experiment.weights) do
        cumulative = cumulative + weight
        if random <= cumulative then
            assignedVariant = experiment.variants[i]
            break
        end
    end
    
    -- บันทึก
    pcall(function()
        abStore:SetAsync(key, assignedVariant)
    end)
    
    return assignedVariant
end

-- บันทึก conversion สำหรับ A/B test
function ABTesting:TrackConversion(player, experimentName, conversionType)
    local variant = self:GetVariant(player, experimentName)
    if not variant then return end
    
    local key = string.format("ABResult_%s_%s", experimentName, variant)
    
    pcall(function()
        abStore:IncrementAsync(key .. "_conversions", 1)
    end)
    
    print(string.format("[A/B] %s - %s: %s conversion",
        experimentName, variant, conversionType
    ))
end

return ABTesting
```

---

## ส่วนที่ 6: Analytics Dashboard (Server-Side)

```lua
-- Script: AnalyticsDashboard
-- Dashboard สำหรับดูสถิติเกม

local DataStoreService = game:GetService("DataStoreService")
local Players = game:GetService("Players")

local dashboardData = {
    realtime = {
        onlinePlayers = 0,
        serverCount = 0,
        totalEventsToday = 0,
    },
    daily = {
        dau = 0, -- Daily Active Users
        newPlayers = 0,
        revenue = 0,
        totalPlayTime = 0,
    },
    gameplay = {
        averageSessionLength = 0,
        topDeathLocations = {},
        popularItems = {},
        completionRates = {},
    }
}

-- อัพเดทข้อมูล realtime
local function updateRealtime()
    dashboardData.realtime.onlinePlayers = #Players:GetPlayers()
end

task.spawn(function()
    while true do
        task.wait(60)
        updateRealtime()
    end
end)

Players.PlayerAdded:Connect(updateRealtime)
Players.PlayerRemoving:Connect(function()
    task.wait(1)
    updateRealtime()
end)

-- ฟังก์ชันรายงานสถิติ (สำหรับ admin)
local function printDashboard()
    print("=== Analytics Dashboard ===")
    print(string.format("Online Players: %d", dashboardData.realtime.onlinePlayers))
    print(string.format("DAU: %d", dashboardData.daily.dau))
    print(string.format("Avg Session: %.1f min", dashboardData.gameplay.averageSessionLength / 60))
    print("==========================")
end
```

---

## ส่วนที่ 7: External Analytics Integration

```lua
-- ModuleScript: ExternalAnalytics
-- ส่งข้อมูลไปยัง analytics service ภายนอก (ถ้าต้องการ)

local HttpService = game:GetService("HttpService")

local ExternalAnalytics = {}

-- URL ของ analytics endpoint
-- (ต้องเปิดใช้งาน HTTP requests ใน game settings)
local ANALYTICS_URL = "https://your-analytics-service.com/api/events"
local API_KEY = "your_api_key_here"

-- ส่ง event ไปยัง external service
function ExternalAnalytics:SendEvent(eventData)
    -- ตรวจสอบว่าเปิดใช้ HTTP service
    local success, result = pcall(function()
        local response = HttpService:PostAsync(
            ANALYTICS_URL,
            HttpService:JSONEncode(eventData),
            Enum.HttpContentType.ApplicationJson,
            false, -- compress
            {
                ["Authorization"] = "Bearer " .. API_KEY,
                ["Content-Type"] = "application/json",
            }
        )
        return response
    end)
    
    if not success then
        warn("[External Analytics] ส่งข้อมูลล้มเหลว: " .. tostring(result))
    end
end

-- Batch send
local batchQueue = {}
local BATCH_SIZE = 20
local BATCH_INTERVAL = 60

function ExternalAnalytics:QueueEvent(event)
    table.insert(batchQueue, event)
    
    if #batchQueue >= BATCH_SIZE then
        self:SendBatch()
    end
end

function ExternalAnalytics:SendBatch()
    if #batchQueue == 0 then return end
    
    local batch = batchQueue
    batchQueue = {}
    
    self:SendEvent({
        type = "batch",
        events = batch,
        timestamp = os.time(),
    })
end

task.spawn(function()
    while true do
        task.wait(BATCH_INTERVAL)
        ExternalAnalytics:SendBatch()
    end
end)

return ExternalAnalytics
```

---

## ส่วนที่ 8: Analytics Reports

### วิธีอ่านและตีความข้อมูล Analytics

**1. Retention Rate**
- Day 1 Retention: กี่เปอร์เซ็นต์กลับมาเล่นวันถัดไป
- Day 7 Retention: กลับมาหลัง 1 สัปดาห์
- Day 30 Retention: กลับมาหลัง 1 เดือน

**2. Session Analysis**
- Average Session Length: ความยาวเซสชันเฉลี่ย
- Session Frequency: เล่นกี่ครั้งต่อวัน/สัปดาห์
- Peak Hours: ช่วงเวลาที่มีผู้เล่นมากที่สุด

**3. Monetization Metrics**
- ARPU (Average Revenue Per User): รายได้เฉลี่ยต่อผู้ใช้
- ARPPU (Average Revenue Per Paying User): รายได้เฉลี่ยต่อผู้ซื้อ
- Conversion Rate: เปอร์เซ็นต์ผู้เล่นที่ซื้อของ

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Death Heatmap
สร้างระบบที่:
1. บันทึกตำแหน่งที่ผู้เล่นตาย
2. แสดงเป็น visual heatmap ใน Studio
3. ระบุจุดอันตรายสูงสุด 5 อันดับ

### แบบฝึกหัดที่ 2: Player Journey
ติดตาม:
1. เวลาที่ใช้ในแต่ละ zone
2. ลำดับการทำ quest
3. จุดที่ผู้เล่นออกจากเกม

### แบบฝึกหัดที่ 3: Economy Analysis
วิเคราะห์:
1. ไอเทมยอดนิยม
2. ราคาที่เหมาะสม
3. Inflation ในเกม

---

## สรุปบทที่ 82

Analytics เป็นเครื่องมือสำคัญที่ช่วยให้นักพัฒนาตัดสินใจบนพื้นฐานของข้อมูล แทนที่จะเดาสุ่ม การมีระบบ analytics ที่ดีจะช่วย:

1. เพิ่ม retention rate ด้วยการแก้ปัญหาที่ผู้เล่นพบ
2. เพิ่มรายได้ด้วยการปรับปรุง monetization
3. ลดต้นทุนการพัฒนาด้วยการโฟกัสที่สิ่งที่สำคัญจริงๆ

*บทถัดไป: Part 83 - Monetization (กลยุทธ์การสร้างรายได้)*
