# Part 80: Badge System (ระบบ Badge และ Achievement)

## บทนำ

Badge และ Achievement System ช่วยให้ผู้เล่นมีเป้าหมายในการเล่นระยะยาว ในบทนี้เราจะสร้างระบบ Badge ครอบคลุม รวมถึง Badge Roblox จริง, Achievement ในเกม, Progress Tracking และ Reward System

---

## 1. Badge Service Integration

### 1.1 Roblox Badge System

```lua
-- BadgeManager.lua (Server Script)
-- ระบบ Badge เชื่อมกับ Roblox Badge Service

local BadgeService = game:GetService("BadgeService")
local Players = game:GetService("Players")
local DataStoreService = game:GetService("DataStoreService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- Badge IDs จาก Roblox (ต้องสร้างใน Creator Dashboard ก่อน)
local BADGE_IDS = {
    -- First Time Badges
    FIRST_JOIN = 000000001,          -- เข้าเกมครั้งแรก
    FIRST_KILL = 000000002,          -- Kill แรก
    FIRST_LEVEL_10 = 000000003,      -- ถึง Level 10
    
    -- Exploration Badges
    EXPLORE_ALL_AREAS = 000000010,   -- สำรวจทุกพื้นที่
    FIND_SECRET_ROOM = 000000011,    -- พบห้องลับ
    
    -- Combat Badges
    KILL_100_ENEMIES = 000000020,    -- ฆ่า Enemy 100 ตัว
    KILL_BOSS = 000000021,           -- ฆ่า Boss ครั้งแรก
    KILL_10_BOSSES = 000000022,      -- ฆ่า Boss 10 ครั้ง
    
    -- Social Badges
    PLAY_WITH_FRIEND = 000000030,    -- เล่นกับเพื่อน
    GUILD_MEMBER = 000000031,        -- เข้า Guild
    
    -- Achievement Badges
    REACH_MAX_LEVEL = 000000040,     -- Max Level
    COLLECT_ALL_PETS = 000000041,    -- รวบรวม Pet ทุกตัว
    SPEEDRUN_COMPLETE = 000000042,   -- Speedrun ผ่าน
}

-- ============================================================
-- Badge Award Functions
-- ============================================================

-- ให้ Badge
local function awardBadge(player, badgeId, badgeName)
    -- ตรวจสอบว่ามีแล้วหรือยัง
    local success, hasBadge = pcall(function()
        return BadgeService:UserHasBadgeAsync(player.UserId, badgeId)
    end)
    
    if not success then
        warn("ตรวจสอบ Badge ล้มเหลว:", hasBadge)
        return false
    end
    
    if hasBadge then
        return false  -- มีแล้ว
    end
    
    -- ให้ Badge
    local awardSuccess, err = pcall(function()
        BadgeService:AwardBadge(player.UserId, badgeId)
    end)
    
    if awardSuccess then
        print(string.format("🏅 %s ได้รับ Badge: %s", player.Name, badgeName or tostring(badgeId)))
        
        -- แจ้ง Client เพื่อแสดง Notification
        local badgeEvent = ReplicatedStorage:FindFirstChild("BadgeEvents")
        if badgeEvent then
            local notify = badgeEvent:FindFirstChild("BadgeAwarded")
            if notify then
                notify:FireClient(player, badgeId, badgeName)
            end
        end
        
        return true
    else
        warn("ให้ Badge ล้มเหลว:", err)
        return false
    end
end

-- ตรวจสอบ Badge
local function checkBadge(player, badgeId)
    local success, result = pcall(function()
        return BadgeService:UserHasBadgeAsync(player.UserId, badgeId)
    end)
    return success and result or false
end

-- ============================================================
-- Badge Triggers (เงื่อนไขที่ได้ Badge)
-- ============================================================

local BadgeTriggers = {}

-- First Join
function BadgeTriggers.onPlayerJoin(player)
    task.spawn(function()
        awardBadge(player, BADGE_IDS.FIRST_JOIN, "การเข้าเล่นครั้งแรก")
    end)
end

-- Level Up
function BadgeTriggers.onLevelUp(player, newLevel)
    if newLevel >= 10 then
        awardBadge(player, BADGE_IDS.FIRST_LEVEL_10, "ถึง Level 10!")
    end
    if newLevel >= 50 then  -- Max Level ตัวอย่าง
        awardBadge(player, BADGE_IDS.REACH_MAX_LEVEL, "Max Level!")
    end
end

-- Kill Enemy
function BadgeTriggers.onEnemyKill(player, totalKills)
    if totalKills == 1 then
        awardBadge(player, BADGE_IDS.FIRST_KILL, "Kill แรก!")
    end
    if totalKills >= 100 then
        awardBadge(player, BADGE_IDS.KILL_100_ENEMIES, "100 Kills!")
    end
end

-- Kill Boss
function BadgeTriggers.onBossKill(player, totalBossKills)
    if totalBossKills == 1 then
        awardBadge(player, BADGE_IDS.KILL_BOSS, "ฆ่า Boss ครั้งแรก!")
    end
    if totalBossKills >= 10 then
        awardBadge(player, BADGE_IDS.KILL_10_BOSSES, "Boss Slayer!")
    end
end

return {
    BADGE_IDS = BADGE_IDS,
    awardBadge = awardBadge,
    checkBadge = checkBadge,
    BadgeTriggers = BadgeTriggers
}
```

---

## 2. Achievement System

### 2.1 Achievement Database

```lua
-- AchievementDatabase.lua (Module Script ใน ReplicatedStorage)
-- ฐานข้อมูล Achievement ทั้งหมด

local AchievementDatabase = {}

-- ประเภทของ Achievement
local ACHIEVEMENT_TYPES = {
    COUNTER = "counter",    -- นับจำนวน (Kill X enemies)
    MILESTONE = "milestone", -- ถึงจุด (Reach level X)
    DISCOVERY = "discovery", -- ค้นพบ (Find secret room)
    COLLECTION = "collection", -- สะสม (Collect X items)
    TIME = "time",          -- เวลา (Play for X hours)
    SOCIAL = "social",      -- สังคม (Play with friends)
}

-- Achievement List
local ACHIEVEMENTS = {
    -- === COMBAT ===
    first_blood = {
        id = "first_blood",
        name = "First Blood",
        nameTh = "เลือดแรก",
        description = "ฆ่า Enemy ตัวแรก",
        icon = "⚔️",
        type = ACHIEVEMENT_TYPES.MILESTONE,
        target = 1,
        stat = "totalKills",
        reward = {coins = 50, xp = 100},
        rarity = "COMMON",
        badgeId = nil,  -- เชื่อม Badge Roblox ถ้ามี
    },
    
    warrior = {
        id = "warrior",
        name = "Warrior",
        nameTh = "นักรบ",
        description = "ฆ่า Enemy 100 ตัว",
        icon = "⚔️",
        type = ACHIEVEMENT_TYPES.COUNTER,
        target = 100,
        stat = "totalKills",
        reward = {coins = 500, xp = 1000},
        rarity = "COMMON",
    },
    
    slayer = {
        id = "slayer",
        name = "Slayer",
        nameTh = "นักล่า",
        description = "ฆ่า Enemy 1,000 ตัว",
        icon = "🗡️",
        type = ACHIEVEMENT_TYPES.COUNTER,
        target = 1000,
        stat = "totalKills",
        reward = {coins = 5000, xp = 10000, item = "sword_upgrade"},
        rarity = "RARE",
    },
    
    -- === EXPLORATION ===
    explorer = {
        id = "explorer",
        name = "Explorer",
        nameTh = "นักสำรวจ",
        description = "เยี่ยมชม 5 พื้นที่",
        icon = "🗺️",
        type = ACHIEVEMENT_TYPES.COUNTER,
        target = 5,
        stat = "areasVisited",
        reward = {coins = 300, xp = 500},
        rarity = "COMMON",
    },
    
    adventurer = {
        id = "adventurer",
        name = "Adventurer",
        nameTh = "นักผจญภัย",
        description = "สำรวจพื้นที่ทั้งหมด 20 แห่ง",
        icon = "🧭",
        type = ACHIEVEMENT_TYPES.COUNTER,
        target = 20,
        stat = "areasVisited",
        reward = {coins = 2000, xp = 5000},
        rarity = "RARE",
    },
    
    -- === PROGRESSION ===
    level_10 = {
        id = "level_10",
        name = "Rising Star",
        nameTh = "ดาวรุ่ง",
        description = "ถึง Level 10",
        icon = "⭐",
        type = ACHIEVEMENT_TYPES.MILESTONE,
        target = 10,
        stat = "level",
        reward = {coins = 1000, xp = 0, item = "rare_chest"},
        rarity = "COMMON",
    },
    
    level_50 = {
        id = "level_50",
        name = "Legend",
        nameTh = "ตำนาน",
        description = "ถึง Level 50",
        icon = "👑",
        type = ACHIEVEMENT_TYPES.MILESTONE,
        target = 50,
        stat = "level",
        reward = {coins = 10000, xp = 0, item = "legendary_chest"},
        rarity = "EPIC",
    },
    
    -- === COLLECTION ===
    collector = {
        id = "collector",
        name = "Collector",
        nameTh = "นักสะสม",
        description = "เก็บ Item ครบ 50 ชิ้น",
        icon = "📦",
        type = ACHIEVEMENT_TYPES.COLLECTION,
        target = 50,
        stat = "itemsCollected",
        reward = {coins = 2000, xp = 3000},
        rarity = "UNCOMMON",
    },
    
    -- === TIME ===
    dedicated = {
        id = "dedicated",
        name = "Dedicated Player",
        nameTh = "ผู้เล่นขยัน",
        description = "เล่นรวม 10 ชั่วโมง",
        icon = "⏰",
        type = ACHIEVEMENT_TYPES.TIME,
        target = 36000,  -- วินาที (10 ชั่วโมง)
        stat = "totalPlayTime",
        reward = {coins = 5000, xp = 0, title = "Veteran"},
        rarity = "UNCOMMON",
    },
    
    -- === SOCIAL ===
    team_player = {
        id = "team_player",
        name = "Team Player",
        nameTh = "ผู้เล่นทีม",
        description = "เล่นกับเพื่อน 10 ครั้ง",
        icon = "👥",
        type = ACHIEVEMENT_TYPES.SOCIAL,
        target = 10,
        stat = "sessionWithFriends",
        reward = {coins = 1000, xp = 2000},
        rarity = "COMMON",
    },
}

-- ดู Achievement
function AchievementDatabase.getAchievement(id)
    return ACHIEVEMENTS[id]
end

-- ดูทั้งหมด
function AchievementDatabase.getAllAchievements()
    return ACHIEVEMENTS
end

-- ดู Achievement ตาม Stat
function AchievementDatabase.getAchievementsByStat(stat)
    local results = {}
    for id, achievement in pairs(ACHIEVEMENTS) do
        if achievement.stat == stat then
            table.insert(results, achievement)
        end
    end
    return results
end

return AchievementDatabase
```

---

## 3. Achievement Tracker (Server)

### 3.1 Progress Tracking

```lua
-- AchievementTracker.lua (Server Script)
-- ติดตามและให้ Achievement

local Players = game:GetService("Players")
local DataStoreService = game:GetService("DataStoreService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local AchievementDatabase = require(ReplicatedStorage:WaitForChild("AchievementDatabase"))

local achievementStore = DataStoreService:GetDataStore("Achievements_v1")

-- ============================================================
-- SETUP EVENTS
-- ============================================================

local achievementFolder = Instance.new("Folder")
achievementFolder.Name = "AchievementEvents"
achievementFolder.Parent = ReplicatedStorage

local updateAchievements = Instance.new("RemoteEvent")
updateAchievements.Name = "UpdateAchievements"
updateAchievements.Parent = achievementFolder

local achievementUnlocked = Instance.new("RemoteEvent")
achievementUnlocked.Name = "AchievementUnlocked"
achievementUnlocked.Parent = achievementFolder

-- ============================================================
-- PLAYER ACHIEVEMENT DATA
-- ============================================================

local playerAchievements = {}
-- Format: {
--   completed = {[id] = true},       -- Achievement ที่ได้แล้ว
--   progress = {[id] = number},      -- Progress ปัจจุบัน
--   stats = {kills = 0, level = 1, ...}  -- Stats ทั้งหมด
-- }

local function getDefaultData()
    return {
        completed = {},
        progress = {},
        stats = {
            totalKills = 0,
            areasVisited = 0,
            level = 1,
            itemsCollected = 0,
            totalPlayTime = 0,
            sessionWithFriends = 0,
        },
        joinTime = tick(),  -- ใช้คำนวณ Play Time
    }
end

local function loadAchievements(player)
    local success, data = pcall(function()
        return achievementStore:GetAsync("achievements_" .. player.UserId)
    end)
    
    if success and data then
        -- Merge กับ Default
        local defaults = getDefaultData()
        for key, value in pairs(defaults) do
            if data[key] == nil then
                data[key] = value
            end
        end
        -- Merge stats
        for stat, defaultVal in pairs(defaults.stats) do
            if data.stats[stat] == nil then
                data.stats[stat] = defaultVal
            end
        end
        playerAchievements[player.UserId] = data
    else
        playerAchievements[player.UserId] = getDefaultData()
    end
    
    return playerAchievements[player.UserId]
end

local function saveAchievements(player)
    local data = playerAchievements[player.UserId]
    if not data then return end
    
    -- บันทึก Play Time
    data.stats.totalPlayTime = data.stats.totalPlayTime + 
        (tick() - data.joinTime)
    data.joinTime = tick()
    
    task.spawn(function()
        pcall(function()
            achievementStore:SetAsync("achievements_" .. player.UserId, data)
        end)
    end)
end

-- ============================================================
-- STAT UPDATING AND ACHIEVEMENT CHECKING
-- ============================================================

-- อัพเดท Stat และตรวจสอบ Achievement
local function updateStat(player, statName, value, mode)
    local data = playerAchievements[player.UserId]
    if not data then return end
    
    mode = mode or "add"  -- "add", "set", "max"
    
    local oldValue = data.stats[statName] or 0
    
    if mode == "add" then
        data.stats[statName] = oldValue + value
    elseif mode == "set" then
        data.stats[statName] = value
    elseif mode == "max" then
        data.stats[statName] = math.max(oldValue, value)
    end
    
    local newValue = data.stats[statName]
    
    -- ตรวจสอบ Achievements ที่เชื่อมกับ Stat นี้
    local relevantAchievements = AchievementDatabase.getAchievementsByStat(statName)
    
    for _, achievement in ipairs(relevantAchievements) do
        if not data.completed[achievement.id] then
            -- อัพเดท Progress
            data.progress[achievement.id] = newValue
            
            -- ตรวจสอบว่าผ่านเกณฑ์แล้วหรือยัง
            if newValue >= achievement.target then
                unlockAchievement(player, achievement)
            end
        end
    end
    
    -- ส่ง Update ไปยัง Client
    updateAchievements:FireClient(player, data)
end

-- Unlock Achievement
function unlockAchievement(player, achievement)
    local data = playerAchievements[player.UserId]
    if not data then return end
    
    if data.completed[achievement.id] then return end
    
    -- Mark as completed
    data.completed[achievement.id] = true
    
    -- ให้ Reward
    local reward = achievement.reward
    if reward then
        -- ส่งผ่าน Event ไปยัง Player Data System
        -- (ใน Production จะเชื่อมกับ PlayerDataManager)
        print(string.format(
            "[ACHIEVEMENT] %s ได้รับ: %s\nReward: %s Coins, %s XP",
            player.Name,
            achievement.nameTh,
            tostring(reward.coins or 0),
            tostring(reward.xp or 0)
        ))
    end
    
    -- แจ้ง Client
    achievementUnlocked:FireClient(player, achievement)
    
    -- บันทึก
    saveAchievements(player)
    
    print(string.format("🏆 %s unlocked: %s", player.Name, achievement.nameTh))
end

-- ============================================================
-- PUBLIC API สำหรับ Systems อื่นๆ เรียกใช้
-- ============================================================

local AchievementTracker = {}

-- เพิ่ม Kill Count
function AchievementTracker.recordKill(player)
    updateStat(player, "totalKills", 1, "add")
end

-- อัพเดท Level
function AchievementTracker.recordLevelUp(player, newLevel)
    updateStat(player, "level", newLevel, "max")
end

-- บันทึกการเยี่ยม Area ใหม่
function AchievementTracker.recordAreaVisit(player)
    updateStat(player, "areasVisited", 1, "add")
end

-- บันทึกการเก็บ Item
function AchievementTracker.recordItemCollect(player)
    updateStat(player, "itemsCollected", 1, "add")
end

-- ดูข้อมูลของ Player
function AchievementTracker.getData(player)
    return playerAchievements[player.UserId]
end

-- ============================================================
-- PLAYER LIFECYCLE
-- ============================================================

Players.PlayerAdded:Connect(function(player)
    local data = loadAchievements(player)
    
    -- ส่งข้อมูลให้ Client
    player.CharacterAdded:Connect(function()
        task.wait(1)
        updateAchievements:FireClient(player, data)
    end)
    
    updateAchievements:FireClient(player, data)
    
    -- บันทึกทุก 5 นาที
    task.spawn(function()
        while player.Parent do
            task.wait(300)
            if player.Parent then
                saveAchievements(player)
            end
        end
    end)
end)

Players.PlayerRemoving:Connect(function(player)
    saveAchievements(player)
    playerAchievements[player.UserId] = nil
end)

return AchievementTracker
```

---

## 4. Achievement UI (Client)

### 4.1 Notification System

```lua
-- AchievementNotification.lua (Local Script)
-- แสดง Notification เมื่อได้ Achievement

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- สร้าง Notification Container
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "AchievementNotifications"
screenGui.ResetOnSpawn = false
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.Parent = playerGui

local notifContainer = Instance.new("Frame")
notifContainer.Name = "NotifContainer"
notifContainer.Size = UDim2.new(0, 350, 1, 0)
notifContainer.Position = UDim2.new(1, -360, 0, 0)
notifContainer.BackgroundTransparency = 1
notifContainer.Parent = screenGui

local listLayout = Instance.new("UIListLayout")
listLayout.SortOrder = Enum.SortOrder.LayoutOrder
listLayout.VerticalAlignment = Enum.VerticalAlignment.Bottom
listLayout.Padding = UDim.new(0, 5)
listLayout.Parent = notifContainer

local containerPadding = Instance.new("UIPadding")
containerPadding.PaddingBottom = UDim.new(0, 10)
containerPadding.Parent = notifContainer

-- Notification Queue
local notifQueue = {}
local isShowingNotif = false

-- แสดง Notification
local function showNotification(achievement)
    local notif = Instance.new("Frame")
    notif.Name = "Notification"
    notif.Size = UDim2.new(1, 0, 0, 80)
    notif.BackgroundColor3 = Color3.fromRGB(20, 20, 35)
    notif.BorderSizePixel = 0
    notif.Position = UDim2.new(1, 0, 0, 0)  -- เริ่มนอกหน้าจอ
    notif.Parent = notifContainer
    
    local notifCorner = Instance.new("UICorner")
    notifCorner.CornerRadius = UDim.new(0, 10)
    notifCorner.Parent = notif
    
    -- Accent Bar
    local accentBar = Instance.new("Frame")
    accentBar.Size = UDim2.new(0, 4, 1, 0)
    accentBar.BackgroundColor3 = Color3.fromRGB(255, 200, 0)
    accentBar.BorderSizePixel = 0
    accentBar.Parent = notif
    
    local accentCorner = Instance.new("UICorner")
    accentCorner.CornerRadius = UDim.new(0, 10)
    accentCorner.Parent = accentBar
    
    -- Icon
    local iconLabel = Instance.new("TextLabel")
    iconLabel.Size = UDim2.new(0, 50, 1, 0)
    iconLabel.Position = UDim2.new(0, 10, 0, 0)
    iconLabel.BackgroundTransparency = 1
    iconLabel.Text = achievement.icon or "🏆"
    iconLabel.TextSize = 32
    iconLabel.Font = Enum.Font.GothamBold
    iconLabel.Parent = notif
    
    -- Header
    local headerLabel = Instance.new("TextLabel")
    headerLabel.Size = UDim2.new(1, -70, 0, 20)
    headerLabel.Position = UDim2.new(0, 65, 0, 12)
    headerLabel.BackgroundTransparency = 1
    headerLabel.Text = "🏆 Achievement Unlocked!"
    headerLabel.TextColor3 = Color3.fromRGB(255, 200, 0)
    headerLabel.TextSize = 12
    headerLabel.Font = Enum.Font.GothamBold
    headerLabel.TextXAlignment = Enum.TextXAlignment.Left
    headerLabel.Parent = notif
    
    -- Achievement Name
    local nameLabel = Instance.new("TextLabel")
    nameLabel.Size = UDim2.new(1, -70, 0, 22)
    nameLabel.Position = UDim2.new(0, 65, 0, 30)
    nameLabel.BackgroundTransparency = 1
    nameLabel.Text = achievement.nameTh or achievement.name
    nameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    nameLabel.TextSize = 16
    nameLabel.Font = Enum.Font.GothamBold
    nameLabel.TextXAlignment = Enum.TextXAlignment.Left
    nameLabel.Parent = notif
    
    -- Description
    local descLabel = Instance.new("TextLabel")
    descLabel.Size = UDim2.new(1, -70, 0, 18)
    descLabel.Position = UDim2.new(0, 65, 0, 52)
    descLabel.BackgroundTransparency = 1
    descLabel.Text = achievement.description
    descLabel.TextColor3 = Color3.fromRGB(180, 180, 180)
    descLabel.TextSize = 12
    descLabel.Font = Enum.Font.Gotham
    descLabel.TextXAlignment = Enum.TextXAlignment.Left
    descLabel.Parent = notif
    
    -- Slide In
    notif.Position = UDim2.new(1, 10, 0, 0)
    
    TweenService:Create(notif, TweenInfo.new(0.4, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
        Position = UDim2.new(0, 0, 0, 0)
    }):Play()
    
    -- รอ แล้ว Slide Out
    task.delay(4, function()
        if not notif.Parent then return end
        
        local slideOut = TweenService:Create(notif, 
            TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.In),
            {Position = UDim2.new(1, 10, 0, 0)}
        )
        slideOut:Play()
        slideOut.Completed:Wait()
        
        if notif.Parent then
            notif:Destroy()
        end
    end)
end

-- Queue System
local function processQueue()
    if isShowingNotif or #notifQueue == 0 then return end
    
    isShowingNotif = true
    local achievement = table.remove(notifQueue, 1)
    showNotification(achievement)
    
    task.delay(1.5, function()
        isShowingNotif = false
        processQueue()
    end)
end

-- รับ Achievement Unlocked Event
local achievementFolder = ReplicatedStorage:WaitForChild("AchievementEvents")
local achievementUnlocked = achievementFolder:WaitForChild("AchievementUnlocked")

achievementUnlocked.OnClientEvent:Connect(function(achievement)
    table.insert(notifQueue, achievement)
    processQueue()
    
    -- เล่นเสียง
    local sound = Instance.new("Sound")
    sound.SoundId = "rbxassetid://0"  -- Achievement Sound
    sound.Volume = 0.5
    sound.Parent = workspace
    sound:Play()
    task.delay(3, function() sound:Destroy() end)
end)
```

---

## 5. Achievement Display Panel

### 5.1 Achievement Browser

```lua
-- AchievementPanel.lua (Local Script)
-- หน้าแสดง Achievements ทั้งหมด

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local AchievementDatabase = require(ReplicatedStorage:WaitForChild("AchievementDatabase"))

local achievementFolder = ReplicatedStorage:WaitForChild("AchievementEvents")
local updateAchievements = achievementFolder:WaitForChild("UpdateAchievements")

-- State
local playerData = nil
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "AchievementPanel"
screenGui.ResetOnSpawn = false
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.Enabled = false
screenGui.Parent = playerGui

-- Main Frame
local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 700, 0, 500)
mainFrame.Position = UDim2.new(0.5, -350, 0.5, -250)
mainFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 25)
mainFrame.BorderSizePixel = 0
mainFrame.Parent = screenGui

local mainCorner = Instance.new("UICorner")
mainCorner.CornerRadius = UDim.new(0, 12)
mainCorner.Parent = mainFrame

-- Title Bar
local titleBar = Instance.new("Frame")
titleBar.Name = "TitleBar"
titleBar.Size = UDim2.new(1, 0, 0, 50)
titleBar.BackgroundColor3 = Color3.fromRGB(25, 25, 40)
titleBar.BorderSizePixel = 0
titleBar.Parent = mainFrame

local titleBarCorner = Instance.new("UICorner")
titleBarCorner.CornerRadius = UDim.new(0, 12)
titleBarCorner.Parent = titleBar

local titleFix = Instance.new("Frame")
titleFix.Size = UDim2.new(1, 0, 0.5, 0)
titleFix.Position = UDim2.new(0, 0, 0.5, 0)
titleFix.BackgroundColor3 = Color3.fromRGB(25, 25, 40)
titleFix.BorderSizePixel = 0
titleFix.Parent = titleBar

local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, -100, 1, 0)
titleLabel.Position = UDim2.new(0, 20, 0, 0)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "🏆 Achievements"
titleLabel.TextColor3 = Color3.fromRGB(255, 200, 0)
titleLabel.TextSize = 22
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextXAlignment = Enum.TextXAlignment.Left
titleLabel.Parent = titleBar

-- Stats Row
local statsRow = Instance.new("Frame")
statsRow.Name = "StatsRow"
statsRow.Size = UDim2.new(1, 0, 0, 40)
statsRow.Position = UDim2.new(0, 0, 0, 50)
statsRow.BackgroundColor3 = Color3.fromRGB(20, 20, 35)
statsRow.BorderSizePixel = 0
statsRow.Parent = mainFrame

local completedLabel = Instance.new("TextLabel")
completedLabel.Name = "Completed"
completedLabel.Size = UDim2.new(0.5, 0, 1, 0)
completedLabel.BackgroundTransparency = 1
completedLabel.Text = "ทำสำเร็จ: 0/0"
completedLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
completedLabel.TextSize = 14
completedLabel.Font = Enum.Font.Gotham
completedLabel.Parent = statsRow

-- Achievement Scroll
local achievementScroll = Instance.new("ScrollingFrame")
achievementScroll.Name = "AchievementScroll"
achievementScroll.Size = UDim2.new(1, 0, 1, -95)
achievementScroll.Position = UDim2.new(0, 0, 0, 95)
achievementScroll.BackgroundTransparency = 1
achievementScroll.BorderSizePixel = 0
achievementScroll.ScrollBarThickness = 4
achievementScroll.ScrollBarImageColor3 = Color3.fromRGB(100, 100, 150)
achievementScroll.CanvasSize = UDim2.new(0, 0, 0, 0)
achievementScroll.AutomaticCanvasSize = Enum.AutomaticSize.Y
achievementScroll.Parent = mainFrame

local scrollPadding = Instance.new("UIPadding")
scrollPadding.PaddingAll = UDim.new(0, 10)
scrollPadding.Parent = achievementScroll

local scrollLayout = Instance.new("UIListLayout")
scrollLayout.SortOrder = Enum.SortOrder.LayoutOrder
scrollLayout.Padding = UDim.new(0, 5)
scrollLayout.Parent = achievementScroll

-- สร้าง Achievement Card
local function createAchievementCard(achievement, data)
    local isCompleted = data and data.completed and data.completed[achievement.id]
    local progress = (data and data.progress and data.progress[achievement.id]) or 
                     (data and data.stats and data.stats[achievement.stat]) or 0
    
    local card = Instance.new("Frame")
    card.Name = "Achievement_" .. achievement.id
    card.Size = UDim2.new(1, 0, 0, 70)
    card.BackgroundColor3 = isCompleted and 
        Color3.fromRGB(30, 45, 30) or 
        Color3.fromRGB(25, 25, 40)
    card.BorderSizePixel = 0
    card.Parent = achievementScroll
    
    local cardCorner = Instance.new("UICorner")
    cardCorner.CornerRadius = UDim.new(0, 8)
    cardCorner.Parent = card
    
    -- Completed Overlay
    if isCompleted then
        local completedOverlay = Instance.new("Frame")
        completedOverlay.Size = UDim2.new(0, 4, 1, 0)
        completedOverlay.BackgroundColor3 = Color3.fromRGB(60, 200, 80)
        completedOverlay.BorderSizePixel = 0
        completedOverlay.Parent = card
        
        local overlayCorner = Instance.new("UICorner")
        overlayCorner.CornerRadius = UDim.new(0, 8)
        overlayCorner.Parent = completedOverlay
    end
    
    -- Icon
    local iconLabel = Instance.new("TextLabel")
    iconLabel.Size = UDim2.new(0, 55, 1, 0)
    iconLabel.BackgroundTransparency = 1
    iconLabel.Text = achievement.icon or "🏆"
    iconLabel.TextSize = 30
    iconLabel.TextTransparency = isCompleted and 0 or 0.3
    iconLabel.Font = Enum.Font.GothamBold
    iconLabel.Parent = card
    
    -- Name
    local nameLabel = Instance.new("TextLabel")
    nameLabel.Size = UDim2.new(0.5, 0, 0, 22)
    nameLabel.Position = UDim2.new(0, 60, 0, 10)
    nameLabel.BackgroundTransparency = 1
    nameLabel.Text = achievement.nameTh or achievement.name
    nameLabel.TextColor3 = isCompleted and 
        Color3.fromRGB(100, 220, 100) or 
        Color3.fromRGB(200, 200, 200)
    nameLabel.TextSize = 15
    nameLabel.Font = Enum.Font.GothamBold
    nameLabel.TextXAlignment = Enum.TextXAlignment.Left
    nameLabel.Parent = card
    
    -- Description
    local descLabel = Instance.new("TextLabel")
    descLabel.Size = UDim2.new(0.7, 0, 0, 16)
    descLabel.Position = UDim2.new(0, 60, 0, 33)
    descLabel.BackgroundTransparency = 1
    descLabel.Text = achievement.description
    descLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
    descLabel.TextSize = 12
    descLabel.Font = Enum.Font.Gotham
    descLabel.TextXAlignment = Enum.TextXAlignment.Left
    descLabel.Parent = card
    
    -- Progress Bar
    if not isCompleted then
        local progressBg = Instance.new("Frame")
        progressBg.Size = UDim2.new(0.65, 0, 0, 6)
        progressBg.Position = UDim2.new(0, 60, 0, 53)
        progressBg.BackgroundColor3 = Color3.fromRGB(40, 40, 60)
        progressBg.BorderSizePixel = 0
        progressBg.Parent = card
        
        local progressCorner = Instance.new("UICorner")
        progressCorner.CornerRadius = UDim.new(1, 0)
        progressCorner.Parent = progressBg
        
        local progressRatio = math.min(progress / achievement.target, 1)
        
        local progressFill = Instance.new("Frame")
        progressFill.Size = UDim2.new(progressRatio, 0, 1, 0)
        progressFill.BackgroundColor3 = Color3.fromRGB(100, 150, 255)
        progressFill.BorderSizePixel = 0
        progressFill.Parent = progressBg
        
        local fillCorner = Instance.new("UICorner")
        fillCorner.CornerRadius = UDim.new(1, 0)
        fillCorner.Parent = progressFill
        
        local progressText = Instance.new("TextLabel")
        progressText.Size = UDim2.new(0.3, 0, 0, 18)
        progressText.Position = UDim2.new(0.7, 0, 0, 47)
        progressText.BackgroundTransparency = 1
        progressText.Text = string.format("%d/%d", 
            math.min(progress, achievement.target), achievement.target)
        progressText.TextColor3 = Color3.fromRGB(150, 150, 150)
        progressText.TextSize = 11
        progressText.Font = Enum.Font.Gotham
        progressText.TextXAlignment = Enum.TextXAlignment.Right
        progressText.Parent = card
    else
        -- Completed checkmark
        local checkLabel = Instance.new("TextLabel")
        checkLabel.Size = UDim2.new(0, 30, 0, 30)
        checkLabel.Position = UDim2.new(1, -40, 0.5, -15)
        checkLabel.BackgroundTransparency = 1
        checkLabel.Text = "✅"
        checkLabel.TextSize = 20
        checkLabel.Font = Enum.Font.GothamBold
        checkLabel.Parent = card
    end
    
    -- Reward Info
    if achievement.reward then
        local rewardText = ""
        if achievement.reward.coins then
            rewardText = rewardText .. string.format("💰%d ", achievement.reward.coins)
        end
        if achievement.reward.xp then
            rewardText = rewardText .. string.format("⭐%d ", achievement.reward.xp)
        end
        
        if rewardText ~= "" then
            local rewardLabel = Instance.new("TextLabel")
            rewardLabel.Size = UDim2.new(0.25, 0, 0, 18)
            rewardLabel.Position = UDim2.new(0.75, -5, 0, 10)
            rewardLabel.BackgroundTransparency = 1
            rewardLabel.Text = rewardText
            rewardLabel.TextColor3 = Color3.fromRGB(200, 180, 100)
            rewardLabel.TextSize = 11
            rewardLabel.Font = Enum.Font.Gotham
            rewardLabel.TextXAlignment = Enum.TextXAlignment.Right
            rewardLabel.Parent = card
        end
    end
    
    return card
end

-- อัพเดท Panel
local function updatePanel(data)
    playerData = data
    
    -- ล้าง Cards เก่า
    for _, child in ipairs(achievementScroll:GetChildren()) do
        if child:IsA("Frame") then
            child:Destroy()
        end
    end
    
    local allAchievements = AchievementDatabase.getAllAchievements()
    local completedCount = 0
    local totalCount = 0
    
    for id, achievement in pairs(allAchievements) do
        totalCount = totalCount + 1
        if data and data.completed and data.completed[id] then
            completedCount = completedCount + 1
        end
        createAchievementCard(achievement, data)
    end
    
    completedLabel.Text = string.format(
        "ทำสำเร็จ: %d/%d (%.0f%%)",
        completedCount, totalCount,
        totalCount > 0 and (completedCount / totalCount * 100) or 0
    )
end

-- รับ Update จาก Server
updateAchievements.OnClientEvent:Connect(updatePanel)

-- Open/Close ด้วย Y
UserInputService.InputBegan:Connect(function(input, processed)
    if processed then return end
    if input.KeyCode == Enum.KeyCode.Y then
        screenGui.Enabled = not screenGui.Enabled
    end
end)

-- Close Button
local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 30, 0, 30)
closeBtn.Position = UDim2.new(1, -40, 0, 10)
closeBtn.BackgroundColor3 = Color3.fromRGB(200, 60, 60)
closeBtn.Text = "✕"
closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
closeBtn.TextSize = 14
closeBtn.Font = Enum.Font.GothamBold
closeBtn.BorderSizePixel = 0
closeBtn.Parent = titleBar

local closeBtnCorner = Instance.new("UICorner")
closeBtnCorner.CornerRadius = UDim.new(0, 6)
closeBtnCorner.Parent = closeBtn

closeBtn.Activated:Connect(function()
    screenGui.Enabled = false
end)
```

---

## 6. ข้อผิดพลาดที่พบบ่อย

### ❌ Badge Award ล้มเหลวเมื่อ Game ไม่ใช่ Published

```lua
-- ❌ แบบผิด: ไม่จัดการ Error เมื่อทดสอบใน Studio
BadgeService:AwardBadge(player.UserId, badgeId)
-- Error ใน Studio: "Badges can only be awarded in Published games"

-- ✅ แบบถูก: ใช้ pcall และจัดการ Studio Mode
local function awardBadge(player, badgeId)
    if game:GetService("RunService"):IsStudio() then
        print("[STUDIO] จำลองการให้ Badge:", badgeId)
        return true  -- จำลองว่าสำเร็จ
    end
    
    local success, err = pcall(function()
        BadgeService:AwardBadge(player.UserId, badgeId)
    end)
    
    if not success then
        warn("Badge Award Error:", err)
        return false
    end
    return true
end
```

---

## 7. แบบฝึกหัด

### แบบฝึกหัดที่ 1: Achievement Integration
เชื่อม Achievement System กับ Game Systems อื่น:
- ทุกครั้งที่ Kill Enemy เรียก `AchievementTracker.recordKill()`
- ทุกครั้งที่ Level Up เรียก `AchievementTracker.recordLevelUp()`
- ทดสอบว่า Notification ปรากฏ

### แบบฝึกหัดที่ 2: Daily Achievements
สร้าง Daily Achievement System:
- Reset ทุก 24 ชั่วโมง
- Achievements ที่ทำต่างกันทุกวัน
- Reward พิเศษเมื่อทำครบทุกวัน 7 วัน

### แบบฝึกหัดที่ 3: Achievement Leaderboard
สร้าง Achievement Point Leaderboard:
- แต่ละ Achievement มี Point ตาม Rarity
- แสดง Top 10 Players ที่มีคะแนนสูงสุด
- ใช้ OrderedDataStore

---

## สรุป (จบ Part 80)

Badge และ Achievement System ที่ดีต้องมี:
1. **Persistent Tracking**: บันทึก Progress ด้วย DataStore
2. **Satisfying Notifications**: Animation และเสียงที่น่าพอใจ
3. **Clear Progress**: แสดง Progress Bar ชัดเจน
4. **Meaningful Rewards**: Reward ที่คุ้มค่ากับความพยายาม
5. **Variety**: Achievement หลากหลาย ทั้งง่ายและยาก

---

## สรุปภาพรวม Parts 61-80

เราได้เรียนรู้ครบทั้ง 20 หัวข้อสำคัญ:
- **61-66**: Map Design, Obby, RPG, Simulator, Tower Defense, Racing
- **67-71**: Shooter, Horror, Social, Multiplayer, Anti-Cheat  
- **72-75**: Performance Optimization, Memory Management, Loading Screen, Settings
- **76-80**: Cutscene, Day/Night, Weather, Pet System, Badge System

ในส่วนถัดไป (Parts 81-100) เราจะเรียนรู้ระดับ Professional ครอบคลุม:
- Monetization, Analytics, Live Operations
- Advanced Scripting Patterns
- การ Publish และ Marketing เกม Roblox
