# Part 59: Level System - ระบบเลเวลและประสบการณ์

## บทนำ

Level System คือหัวใจของ RPG ทุกเกม ทำให้ผู้เล่นรู้สึกถึงการเติบโตและความก้าวหน้า ในบทนี้เราจะสร้างระบบเลเวลแบบครบวงจร ตั้งแต่การรับ EXP ไปจนถึงการ Level Up พร้อม stat points และระบบ prestige

## สถาปัตยกรรมระบบเลเวล

```
LevelSystem/
├── LevelConfig (Module) - ตารางประสบการณ์และ rewards
├── LevelManager (Server) - จัดการ EXP และ level up
├── LevelUI (Client) - แสดง EXP bar และ level up animation
└── StatManager (Server) - จัดการ stat points
```

## LevelConfig Module

```lua
-- ReplicatedStorage/Shared/LevelConfig.lua

local LevelConfig = {}

-- ===== Max Level =====
LevelConfig.MAX_LEVEL = 100

-- ===== EXP Formula =====
-- EXP ที่ต้องการต่อ level ใช้สูตร: base * level^exponent
LevelConfig.EXP_BASE = 100
LevelConfig.EXP_EXPONENT = 1.5

-- ===== Prestige System =====
LevelConfig.PRESTIGE_UNLOCK_LEVEL = 100  -- ต้องถึง level นี้ก่อน prestige
LevelConfig.PRESTIGE_MAX = 10
LevelConfig.PRESTIGE_BONUS_PER = 0.05  -- +5% stats ต่อ prestige

-- ===== Stat Points =====
LevelConfig.STAT_POINTS_PER_LEVEL = 3   -- ได้ stat points ต่อ level
LevelConfig.SKILL_POINTS_PER_LEVEL = 1  -- ได้ skill points ต่อ level

-- ===== Stats =====
LevelConfig.BASE_STATS = {
    MaxHealth = 100,
    MaxMP = 50,
    Attack = 10,
    Defense = 5,
    Speed = 16,
    CritChance = 0.05,
    MagicPower = 10
}

-- ===== Stat Growth Per Level =====
LevelConfig.STAT_GROWTH = {
    MaxHealth = 10,   -- +10 HP per level
    MaxMP = 5,        -- +5 MP per level
    Attack = 1,       -- +1 Attack per level
    Defense = 0.5,    -- +0.5 Defense per level
}

-- ===== Level Up Rewards =====
-- รางวัลพิเศษเมื่อถึง level บางระดับ
LevelConfig.MILESTONE_REWARDS = {
    [5]  = {type = "coins", amount = 500, message = "เข้าสู่ระดับ 5! ได้รับ 500 เหรียญ"},
    [10] = {type = "item", itemId = "rare_sword", message = "เข้าสู่ระดับ 10! ได้รับ Rare Sword"},
    [20] = {type = "skill_point", amount = 3, message = "เข้าสู่ระดับ 20! ได้รับ Skill Points พิเศษ"},
    [30] = {type = "coins", amount = 2000, message = "เข้าสู่ระดับ 30! ได้รับ 2000 เหรียญ"},
    [50] = {type = "title", titleId = "champion", message = "เข้าสู่ระดับ 50! ได้รับตำแหน่ง Champion"},
    [100] = {type = "prestige_unlock", message = "ถึงระดับสูงสุด! Prestige พร้อมใช้งาน!"}
}

-- ===== EXP Sources =====
LevelConfig.EXP_SOURCES = {
    kill_common = 20,
    kill_uncommon = 50,
    kill_rare = 100,
    kill_boss = 500,
    quest_complete = 200,
    daily_quest = 100,
    explore_zone = 50,
    craft_item = 10,
    achievement = 300
}

-- ===== Calculate Functions =====

-- คำนวณ EXP ที่ต้องการสำหรับ level นั้น
function LevelConfig.expRequiredForLevel(level)
    if level <= 0 then return 0 end
    return math.floor(LevelConfig.EXP_BASE * (level ^ LevelConfig.EXP_EXPONENT))
end

-- คำนวณ EXP สะสมทั้งหมดถึง level นั้น
function LevelConfig.totalExpToLevel(level)
    local total = 0
    for i = 1, level - 1 do
        total = total + LevelConfig.expRequiredForLevel(i)
    end
    return total
end

-- คำนวณ level จาก total EXP
function LevelConfig.levelFromExp(totalExp)
    local level = 1
    local expSpent = 0
    
    while level < LevelConfig.MAX_LEVEL do
        local needed = LevelConfig.expRequiredForLevel(level)
        if expSpent + needed > totalExp then
            break
        end
        expSpent = expSpent + needed
        level = level + 1
    end
    
    return level, totalExp - expSpent
end

-- คำนวณ stats ตาม level
function LevelConfig.calculateStats(level, prestige)
    local stats = {}
    local prestigeBonus = 1 + (prestige or 0) * LevelConfig.PRESTIGE_BONUS_PER
    
    for stat, base in pairs(LevelConfig.BASE_STATS) do
        local growth = LevelConfig.STAT_GROWTH[stat] or 0
        stats[stat] = math.floor((base + growth * (level - 1)) * prestigeBonus)
    end
    
    return stats
end

-- คำนวณ bonus EXP จาก prestige
function LevelConfig.getExpBonus(prestige)
    return 1 + (prestige or 0) * 0.1  -- +10% EXP per prestige
end

return LevelConfig
```

## LevelManager - Server Script

```lua
-- ServerScriptService/LevelManager.lua

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local DataStoreService = game:GetService("DataStoreService")

local LevelConfig = require(ReplicatedStorage.Shared.LevelConfig)

-- ===== Data Store =====
local levelStore = DataStoreService:GetDataStore("PlayerLevelData_v2")

-- ===== Remotes =====
local remotes = ReplicatedStorage:WaitForChild("Remotes")
local levelUpdateEvent = remotes:WaitForChild("LevelUpdate")
local levelUpEvent = remotes:WaitForChild("LevelUp")
local expGainEvent = remotes:WaitForChild("ExpGain")
local spendStatPointEvent = remotes:WaitForChild("SpendStatPoint")
local prestigeEvent = remotes:WaitForChild("PrestigeRequest")

-- ===== Player Level Data =====
local playerData = {}

-- Default data structure
local function getDefaultData()
    return {
        level = 1,
        exp = 0,
        totalExp = 0,
        prestige = 0,
        statPoints = 0,
        skillPoints = 0,
        
        -- Allocated stats
        allocatedStats = {
            Strength = 0,
            Intelligence = 0,
            Vitality = 0,
            Agility = 0,
            Luck = 0
        },
        
        -- Achievements
        totalKills = 0,
        totalExpEarned = 0,
        
        version = 1
    }
end

-- ===== Load/Save =====
local function loadPlayerData(player)
    local success, data = pcall(function()
        return levelStore:GetAsync("level_" .. player.UserId)
    end)
    
    if success and data then
        -- Merge กับ default เพื่อรองรับ fields ใหม่
        local default = getDefaultData()
        for key, value in pairs(default) do
            if data[key] == nil then
                data[key] = value
            end
        end
        return data
    else
        return getDefaultData()
    end
end

local function savePlayerData(player)
    local data = playerData[player]
    if not data then return end
    
    pcall(function()
        levelStore:SetAsync("level_" .. player.UserId, data)
    end)
end

-- ===== Leaderstats Setup =====
local function setupLeaderstats(player, data)
    -- สร้าง Leaderstats folder
    local leaderstats = player:FindFirstChild("leaderstats") or Instance.new("Folder")
    leaderstats.Name = "leaderstats"
    leaderstats.Parent = player
    
    -- Level
    local levelValue = leaderstats:FindFirstChild("Level") or Instance.new("IntValue")
    levelValue.Name = "Level"
    levelValue.Value = data.level
    levelValue.Parent = leaderstats
    
    -- สร้าง Stats folder สำหรับ stats จริง
    local statsFolder = player:FindFirstChild("Stats") or Instance.new("Folder")
    statsFolder.Name = "Stats"
    statsFolder.Parent = player
    
    -- คำนวณ stats พื้นฐาน
    local baseStats = LevelConfig.calculateStats(data.level, data.prestige)
    
    -- Stats ที่ผู้เล่น allocate เอง
    local allocBonus = {
        MaxHealth = data.allocatedStats.Vitality * 20,
        Attack = data.allocatedStats.Strength * 2,
        MagicPower = data.allocatedStats.Intelligence * 2,
        Speed = data.allocatedStats.Agility * 0.5,
        CritChance = data.allocatedStats.Luck * 0.005
    }
    
    -- สร้าง stat values
    local statNames = {"Level", "MaxHealth", "HP", "MaxMP", "MP", "Attack", "Defense",
                       "MagicPower", "StatPoints", "SkillPoints", "Prestige",
                       "TotalExp", "Strength", "Intelligence", "Vitality", "Agility", "Luck"}
    
    for _, statName in ipairs(statNames) do
        if not statsFolder:FindFirstChild(statName) then
            local val = Instance.new("NumberValue")
            val.Name = statName
            val.Parent = statsFolder
        end
    end
    
    -- ตั้งค่า stats
    local stats = statsFolder
    stats.Level.Value = data.level
    stats.MaxHealth.Value = (baseStats.MaxHealth or 100) + (allocBonus.MaxHealth or 0)
    stats.HP.Value = stats.MaxHealth.Value
    stats.MaxMP.Value = (baseStats.MaxMP or 50) + 0
    stats.MP.Value = stats.MaxMP.Value
    stats.Attack.Value = (baseStats.Attack or 10) + (allocBonus.Attack or 0)
    stats.Defense.Value = baseStats.Defense or 5
    stats.MagicPower.Value = (baseStats.MagicPower or 10) + (allocBonus.MagicPower or 0)
    stats.StatPoints.Value = data.statPoints
    stats.SkillPoints.Value = data.skillPoints
    stats.Prestige.Value = data.prestige
    stats.TotalExp.Value = data.totalExp
    
    -- Allocated stats
    stats.Strength.Value = data.allocatedStats.Strength
    stats.Intelligence.Value = data.allocatedStats.Intelligence
    stats.Vitality.Value = data.allocatedStats.Vitality
    stats.Agility.Value = data.allocatedStats.Agility
    stats.Luck.Value = data.allocatedStats.Luck
    
    -- ตั้ง HP จริงใน Humanoid ด้วย
    local char = player.Character
    if char then
        local hum = char:FindFirstChild("Humanoid")
        if hum then
            hum.MaxHealth = stats.MaxHealth.Value
            hum.Health = stats.MaxHealth.Value
        end
    end
end

-- ===== Give EXP =====
local function giveExp(player, amount, source)
    local data = playerData[player]
    if not data then return end
    
    -- Prestige bonus
    local expBonus = LevelConfig.getExpBonus(data.prestige)
    local finalExp = math.floor(amount * expBonus)
    
    data.exp = data.exp + finalExp
    data.totalExp = data.totalExp + finalExp
    data.totalExpEarned = (data.totalExpEarned or 0) + finalExp
    
    -- Update stats
    local stats = player:FindFirstChild("Stats")
    if stats and stats:FindFirstChild("TotalExp") then
        stats.TotalExp.Value = data.totalExp
    end
    
    -- แจ้ง client
    expGainEvent:FireClient(player, {
        amount = finalExp,
        source = source or "Unknown",
        bonus = expBonus > 1 and (expBonus - 1) or nil
    })
    
    -- ตรวจสอบ level up
    checkLevelUp(player)
    
    print(string.format("[Level] %s gained %d EXP from %s (bonus: %.0f%%)",
        player.Name, finalExp, source or "Unknown", (expBonus - 1) * 100))
end

-- ===== Level Up Check =====
function checkLevelUp(player)
    local data = playerData[player]
    if not data then return end
    
    if data.level >= LevelConfig.MAX_LEVEL then return end
    
    local leveled = false
    
    -- ตรวจว่า level up หรือยัง
    while data.level < LevelConfig.MAX_LEVEL do
        local expNeeded = LevelConfig.expRequiredForLevel(data.level)
        
        if data.exp >= expNeeded then
            data.exp = data.exp - expNeeded
            data.level = data.level + 1
            leveled = true
            
            -- Stat points
            data.statPoints = data.statPoints + LevelConfig.STAT_POINTS_PER_LEVEL
            data.skillPoints = data.skillPoints + LevelConfig.SKILL_POINTS_PER_LEVEL
            
            -- อัพเดท stats
            updatePlayerStats(player)
            
            -- Milestone rewards
            local milestone = LevelConfig.MILESTONE_REWARDS[data.level]
            if milestone then
                grantMilestoneReward(player, milestone)
            end
            
            print(string.format("[Level] %s leveled up to %d!", player.Name, data.level))
        else
            break
        end
    end
    
    if leveled then
        -- อัพเดท leaderstats
        local leaderstats = player:FindFirstChild("leaderstats")
        if leaderstats then
            local levelValue = leaderstats:FindFirstChild("Level")
            if levelValue then levelValue.Value = data.level end
        end
        
        -- แจ้ง client แสดง level up animation
        levelUpEvent:FireClient(player, {
            newLevel = data.level,
            statPoints = data.statPoints,
            skillPoints = data.skillPoints
        })
        
        -- อัพเดท level bar
        sendLevelUpdate(player)
    end
end

-- ===== Update Stats After Level Up =====
function updatePlayerStats(player)
    local data = playerData[player]
    if not data then return end
    
    local stats = player:FindFirstChild("Stats")
    if not stats then return end
    
    -- คำนวณ stats ใหม่
    local baseStats = LevelConfig.calculateStats(data.level, data.prestige)
    
    -- Allocated stat bonuses
    local allocBonus = {
        MaxHealth = data.allocatedStats.Vitality * 20,
        Attack = data.allocatedStats.Strength * 2,
        MagicPower = data.allocatedStats.Intelligence * 2
    }
    
    -- อัพเดท stats
    if stats:FindFirstChild("Level") then stats.Level.Value = data.level end
    
    local newMaxHP = (baseStats.MaxHealth or 100) + (allocBonus.MaxHealth or 0)
    if stats:FindFirstChild("MaxHealth") then
        local oldMax = stats.MaxHealth.Value
        stats.MaxHealth.Value = newMaxHP
        
        -- เพิ่ม HP ตามสัดส่วน
        if stats:FindFirstChild("HP") then
            local hpPercent = stats.HP.Value / math.max(oldMax, 1)
            stats.HP.Value = math.floor(newMaxHP * hpPercent)
        end
    end
    
    if stats:FindFirstChild("Attack") then
        stats.Attack.Value = (baseStats.Attack or 10) + (allocBonus.Attack or 0)
    end
    if stats:FindFirstChild("Defense") then
        stats.Defense.Value = baseStats.Defense or 5
    end
    if stats:FindFirstChild("MagicPower") then
        stats.MagicPower.Value = (baseStats.MagicPower or 10) + (allocBonus.MagicPower or 0)
    end
    if stats:FindFirstChild("StatPoints") then
        stats.StatPoints.Value = data.statPoints
    end
    if stats:FindFirstChild("SkillPoints") then
        stats.SkillPoints.Value = data.skillPoints
    end
    
    -- อัพเดท Humanoid MaxHealth
    local char = player.Character
    if char then
        local hum = char:FindFirstChild("Humanoid")
        if hum then
            hum.MaxHealth = newMaxHP
        end
    end
end

-- ===== Milestone Reward =====
function grantMilestoneReward(player, milestone)
    print(string.format("[Level] Milestone reward for %s: %s", player.Name, milestone.message))
    
    -- แจ้ง client
    levelUpdateEvent:FireClient(player, {
        type = "MilestoneReward",
        message = milestone.message,
        rewardType = milestone.type
    })
    
    if milestone.type == "coins" then
        -- ใช้ CurrencyManager
        -- CurrencyManager.addCoins(player, milestone.amount)
        
    elseif milestone.type == "skill_point" then
        local data = playerData[player]
        if data then
            data.skillPoints = data.skillPoints + milestone.amount
            local stats = player:FindFirstChild("Stats")
            if stats and stats:FindFirstChild("SkillPoints") then
                stats.SkillPoints.Value = data.skillPoints
            end
        end
        
    elseif milestone.type == "prestige_unlock" then
        levelUpdateEvent:FireClient(player, {
            type = "PrestigeAvailable"
        })
    end
end

-- ===== Send Level Update =====
function sendLevelUpdate(player)
    local data = playerData[player]
    if not data then return end
    
    levelUpdateEvent:FireClient(player, {
        type = "Update",
        level = data.level,
        exp = data.exp,
        expRequired = LevelConfig.expRequiredForLevel(data.level),
        totalExp = data.totalExp,
        statPoints = data.statPoints,
        skillPoints = data.skillPoints,
        prestige = data.prestige
    })
end

-- ===== Spend Stat Point =====
local function spendStatPoint(player, statName)
    local data = playerData[player]
    if not data then return false end
    
    -- ตรวจสอบ stat points
    if data.statPoints <= 0 then
        return false, "NO_STAT_POINTS"
    end
    
    -- ตรวจสอบ valid stat
    local validStats = {"Strength", "Intelligence", "Vitality", "Agility", "Luck"}
    local isValid = false
    for _, s in ipairs(validStats) do
        if s == statName then
            isValid = true
            break
        end
    end
    
    if not isValid then return false, "INVALID_STAT" end
    
    -- ใช้ stat point
    data.statPoints = data.statPoints - 1
    data.allocatedStats[statName] = (data.allocatedStats[statName] or 0) + 1
    
    -- อัพเดท stats
    updatePlayerStats(player)
    
    -- อัพเดท allocated stat value
    local stats = player:FindFirstChild("Stats")
    if stats and stats:FindFirstChild(statName) then
        stats[statName].Value = data.allocatedStats[statName]
    end
    
    sendLevelUpdate(player)
    
    print(string.format("[Level] %s spent stat point on %s (total: %d)",
        player.Name, statName, data.allocatedStats[statName]))
    
    return true
end

-- ===== Prestige =====
local function prestige(player)
    local data = playerData[player]
    if not data then return false end
    
    -- ตรวจสอบ level requirement
    if data.level < LevelConfig.PRESTIGE_UNLOCK_LEVEL then
        return false, "LEVEL_REQUIREMENT"
    end
    
    -- ตรวจสอบ max prestige
    if data.prestige >= LevelConfig.PRESTIGE_MAX then
        return false, "MAX_PRESTIGE"
    end
    
    -- บันทึกก่อน prestige
    local oldPrestige = data.prestige
    
    -- Reset level
    data.level = 1
    data.exp = 0
    data.prestige = data.prestige + 1
    data.statPoints = 0
    data.skillPoints = 0
    
    -- Reset allocated stats (optional: บาง game เก็บไว้ให้)
    data.allocatedStats = {
        Strength = 0, Intelligence = 0,
        Vitality = 0, Agility = 0, Luck = 0
    }
    
    -- อัพเดท stats (จะมี prestige bonus)
    updatePlayerStats(player)
    
    -- อัพเดท leaderstats
    local leaderstats = player:FindFirstChild("leaderstats")
    if leaderstats and leaderstats:FindFirstChild("Level") then
        leaderstats.Level.Value = 1
    end
    
    -- แจ้ง client
    levelUpdateEvent:FireClient(player, {
        type = "Prestige",
        newPrestige = data.prestige,
        message = string.format("Prestige %d! +%.0f%% stats!", data.prestige, LevelConfig.PRESTIGE_BONUS_PER * 100)
    })
    
    sendLevelUpdate(player)
    
    print(string.format("[Level] %s prestiged to %d!", player.Name, data.prestige))
    return true
end

-- ===== Event Handlers =====
spendStatPointEvent.OnServerEvent:Connect(function(player, statName)
    local success, reason = spendStatPoint(player, statName)
    if not success then
        levelUpdateEvent:FireClient(player, {
            type = "Error",
            message = reason or "ไม่สามารถใช้ Stat Point ได้"
        })
    end
end)

prestigeEvent.OnServerEvent:Connect(function(player)
    local success, reason = prestige(player)
    if not success then
        levelUpdateEvent:FireClient(player, {
            type = "Error",
            message = reason or "ไม่สามารถ Prestige ได้"
        })
    end
end)

-- ===== Player Setup =====
Players.PlayerAdded:Connect(function(player)
    -- โหลดข้อมูล
    local data = loadPlayerData(player)
    playerData[player] = data
    
    -- Setup leaderstats เมื่อตัวละครโหลด
    player.CharacterAdded:Connect(function(char)
        task.wait(0.5)  -- รอให้ character โหลดเสร็จ
        setupLeaderstats(player, data)
        sendLevelUpdate(player)
    end)
    
    -- ถ้าตัวละครมีอยู่แล้ว
    if player.Character then
        setupLeaderstats(player, data)
        sendLevelUpdate(player)
    end
end)

Players.PlayerRemoving:Connect(function(player)
    savePlayerData(player)
    playerData[player] = nil
end)

-- Auto-save ทุก 2 นาที
task.spawn(function()
    while true do
        task.wait(120)
        for _, player in ipairs(Players:GetPlayers()) do
            savePlayerData(player)
        end
    end
end)

-- ===== Public API =====
local LevelManager = {}

function LevelManager.giveExp(player, amount, source)
    giveExp(player, amount, source)
end

function LevelManager.getLevel(player)
    local data = playerData[player]
    return data and data.level or 1
end

function LevelManager.getPrestige(player)
    local data = playerData[player]
    return data and data.prestige or 0
end

function LevelManager.getPlayerData(player)
    return playerData[player]
end

return LevelManager
```

## LevelUI - Client Script

```lua
-- StarterPlayerScripts/LevelUI.lua

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer

-- ===== Remotes =====
local remotes = ReplicatedStorage:WaitForChild("Remotes")
local levelUpdateEvent = remotes:WaitForChild("LevelUpdate")
local levelUpEvent = remotes:WaitForChild("LevelUp")
local expGainEvent = remotes:WaitForChild("ExpGain")
local spendStatPointEvent = remotes:WaitForChild("SpendStatPoint")

-- ===== UI Setup =====
local function createLevelUI()
    local gui = Instance.new("ScreenGui")
    gui.Name = "LevelUI"
    gui.ResetOnSpawn = false
    gui.Parent = player.PlayerGui
    
    -- ===== EXP Bar (บนซ้าย) =====
    local expFrame = Instance.new("Frame")
    expFrame.Name = "ExpBar"
    expFrame.Size = UDim2.new(0, 300, 0, 40)
    expFrame.Position = UDim2.new(0, 16, 0, 16)
    expFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
    expFrame.BorderSizePixel = 0
    expFrame.Parent = gui
    
    Instance.new("UICorner", expFrame).CornerRadius = UDim.new(0, 8)
    
    -- Level badge
    local levelBadge = Instance.new("Frame")
    levelBadge.Name = "LevelBadge"
    levelBadge.Size = UDim2.new(0, 40, 0, 40)
    levelBadge.Position = UDim2.new(0, 0, 0, 0)
    levelBadge.BackgroundColor3 = Color3.fromRGB(255, 180, 0)
    levelBadge.BorderSizePixel = 0
    levelBadge.Parent = expFrame
    
    Instance.new("UICorner", levelBadge).CornerRadius = UDim.new(0, 8)
    
    local levelText = Instance.new("TextLabel")
    levelText.Name = "LevelText"
    levelText.Size = UDim2.new(1, 0, 1, 0)
    levelText.BackgroundTransparency = 1
    levelText.Text = "1"
    levelText.Font = Enum.Font.GothamBold
    levelText.TextSize = 18
    levelText.TextColor3 = Color3.new(0, 0, 0)
    levelText.Parent = levelBadge
    
    -- Prestige stars (ถ้า prestige > 0 แสดง)
    local prestigeLabel = Instance.new("TextLabel")
    prestigeLabel.Name = "PrestigeLabel"
    prestigeLabel.Size = UDim2.new(0, 40, 0, 12)
    prestigeLabel.Position = UDim2.new(0, 0, 1, 2)
    prestigeLabel.BackgroundTransparency = 1
    prestigeLabel.Text = ""
    prestigeLabel.Font = Enum.Font.Gotham
    prestigeLabel.TextSize = 10
    prestigeLabel.TextColor3 = Color3.fromRGB(255, 215, 0)
    prestigeLabel.Parent = expFrame
    
    -- EXP bar background
    local expBg = Instance.new("Frame")
    expBg.Name = "ExpBackground"
    expBg.Size = UDim2.new(1, -50, 0, 12)
    expBg.Position = UDim2.new(0, 46, 0.5, 4)
    expBg.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
    expBg.BorderSizePixel = 0
    expBg.Parent = expFrame
    
    Instance.new("UICorner", expBg).CornerRadius = UDim.new(0, 6)
    
    -- EXP fill
    local expFill = Instance.new("Frame")
    expFill.Name = "ExpFill"
    expFill.Size = UDim2.new(0, 0, 1, 0)
    expFill.BackgroundColor3 = Color3.fromRGB(80, 200, 120)
    expFill.BorderSizePixel = 0
    expFill.Parent = expBg
    
    Instance.new("UICorner", expFill).CornerRadius = UDim.new(0, 6)
    
    -- EXP text
    local expText = Instance.new("TextLabel")
    expText.Name = "ExpText"
    expText.Size = UDim2.new(1, -50, 0, 14)
    expText.Position = UDim2.new(0, 46, 0, 4)
    expText.BackgroundTransparency = 1
    expText.Text = "0 / 100 EXP"
    expText.Font = Enum.Font.Gotham
    expText.TextSize = 11
    expText.TextColor3 = Color3.fromRGB(200, 200, 200)
    expText.TextXAlignment = Enum.TextXAlignment.Left
    expText.Parent = expFrame
    
    return gui
end

-- ===== Stat Points UI =====
local function createStatPointsUI()
    local gui = player.PlayerGui:FindFirstChild("LevelUI")
    if not gui then return end
    
    -- Stat Points Panel (กลาง-ขวา)
    local statFrame = Instance.new("Frame")
    statFrame.Name = "StatPointsFrame"
    statFrame.Size = UDim2.new(0, 220, 0, 250)
    statFrame.Position = UDim2.new(1, -236, 0.5, -125)
    statFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 25)
    statFrame.BackgroundTransparency = 0.2
    statFrame.BorderSizePixel = 0
    statFrame.Visible = false
    statFrame.Parent = gui
    
    Instance.new("UICorner", statFrame).CornerRadius = UDim.new(0, 10)
    
    -- Title
    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, -10, 0, 30)
    title.Position = UDim2.new(0, 5, 0, 5)
    title.BackgroundTransparency = 1
    title.Text = "Stat Points"
    title.Font = Enum.Font.GothamBold
    title.TextSize = 16
    title.TextColor3 = Color3.fromRGB(255, 215, 0)
    title.Parent = statFrame
    
    -- Points available
    local pointsLabel = Instance.new("TextLabel")
    pointsLabel.Name = "PointsLabel"
    pointsLabel.Size = UDim2.new(1, -10, 0, 20)
    pointsLabel.Position = UDim2.new(0, 5, 0, 38)
    pointsLabel.BackgroundTransparency = 1
    pointsLabel.Text = "Points: 0"
    pointsLabel.Font = Enum.Font.Gotham
    pointsLabel.TextSize = 13
    pointsLabel.TextColor3 = Color3.fromRGB(200, 255, 200)
    pointsLabel.TextXAlignment = Enum.TextXAlignment.Left
    pointsLabel.Parent = statFrame
    
    -- Stats list
    local stats = {"Strength", "Intelligence", "Vitality", "Agility", "Luck"}
    local statDescriptions = {
        Strength = "Attack +2",
        Intelligence = "Magic Power +2",
        Vitality = "Max HP +20",
        Agility = "Speed +0.5",
        Luck = "Crit +0.5%"
    }
    
    for i, statName in ipairs(stats) do
        local row = Instance.new("Frame")
        row.Name = statName .. "Row"
        row.Size = UDim2.new(1, -10, 0, 32)
        row.Position = UDim2.new(0, 5, 0, 65 + (i-1) * 36)
        row.BackgroundColor3 = Color3.fromRGB(30, 30, 45)
        row.BorderSizePixel = 0
        row.Parent = statFrame
        
        Instance.new("UICorner", row).CornerRadius = UDim.new(0, 6)
        
        local statLabel = Instance.new("TextLabel")
        statLabel.Size = UDim2.new(0, 90, 1, 0)
        statLabel.Position = UDim2.new(0, 8, 0, 0)
        statLabel.BackgroundTransparency = 1
        statLabel.Text = statName
        statLabel.Font = Enum.Font.Gotham
        statLabel.TextSize = 13
        statLabel.TextColor3 = Color3.new(1, 1, 1)
        statLabel.TextXAlignment = Enum.TextXAlignment.Left
        statLabel.Parent = row
        
        local descLabel = Instance.new("TextLabel")
        descLabel.Size = UDim2.new(0, 80, 1, 0)
        descLabel.Position = UDim2.new(0, 90, 0, 0)
        descLabel.BackgroundTransparency = 1
        descLabel.Text = statDescriptions[statName] or ""
        descLabel.Font = Enum.Font.Gotham
        descLabel.TextSize = 11
        descLabel.TextColor3 = Color3.fromRGB(150, 200, 150)
        descLabel.TextXAlignment = Enum.TextXAlignment.Left
        descLabel.Parent = row
        
        local valueLabel = Instance.new("TextLabel")
        valueLabel.Name = "Value"
        valueLabel.Size = UDim2.new(0, 30, 1, 0)
        valueLabel.Position = UDim2.new(1, -60, 0, 0)
        valueLabel.BackgroundTransparency = 1
        valueLabel.Text = "0"
        valueLabel.Font = Enum.Font.GothamBold
        valueLabel.TextSize = 14
        valueLabel.TextColor3 = Color3.fromRGB(255, 200, 100)
        valueLabel.Parent = row
        
        -- Plus button
        local plusBtn = Instance.new("TextButton")
        plusBtn.Name = "PlusButton"
        plusBtn.Size = UDim2.new(0, 24, 0, 24)
        plusBtn.Position = UDim2.new(1, -28, 0.5, -12)
        plusBtn.BackgroundColor3 = Color3.fromRGB(60, 150, 60)
        plusBtn.Text = "+"
        plusBtn.Font = Enum.Font.GothamBold
        plusBtn.TextSize = 14
        plusBtn.TextColor3 = Color3.new(1, 1, 1)
        plusBtn.BorderSizePixel = 0
        plusBtn.Parent = row
        
        Instance.new("UICorner", plusBtn).CornerRadius = UDim.new(0, 4)
        
        plusBtn.MouseButton1Click:Connect(function()
            spendStatPointEvent:FireServer(statName)
        end)
    end
    
    return statFrame
end

-- ===== Level Up Animation =====
local function showLevelUpAnimation(newLevel)
    local gui = player.PlayerGui:FindFirstChild("LevelUI")
    if not gui then return end
    
    -- Big level up text
    local levelUpFrame = Instance.new("Frame")
    levelUpFrame.Size = UDim2.new(0, 400, 0, 120)
    levelUpFrame.Position = UDim2.new(0.5, -200, 0.5, -60)
    levelUpFrame.BackgroundTransparency = 1
    levelUpFrame.ZIndex = 20
    levelUpFrame.Parent = gui
    
    -- Glow background
    local glow = Instance.new("Frame")
    glow.Size = UDim2.new(1, 0, 1, 0)
    glow.BackgroundColor3 = Color3.fromRGB(255, 215, 0)
    glow.BackgroundTransparency = 0.7
    glow.BorderSizePixel = 0
    glow.ZIndex = 19
    glow.Parent = levelUpFrame
    
    Instance.new("UICorner", glow).CornerRadius = UDim.new(0, 15)
    
    -- "LEVEL UP!" text
    local mainText = Instance.new("TextLabel")
    mainText.Size = UDim2.new(1, 0, 0, 60)
    mainText.Position = UDim2.new(0, 0, 0, 10)
    mainText.BackgroundTransparency = 1
    mainText.Text = "LEVEL UP!"
    mainText.Font = Enum.Font.GothamBlack
    mainText.TextSize = 48
    mainText.TextColor3 = Color3.fromRGB(255, 215, 0)
    mainText.TextStrokeTransparency = 0
    mainText.TextStrokeColor3 = Color3.fromRGB(100, 80, 0)
    mainText.ZIndex = 21
    mainText.Parent = levelUpFrame
    
    -- Level number
    local levelNumText = Instance.new("TextLabel")
    levelNumText.Size = UDim2.new(1, 0, 0, 40)
    levelNumText.Position = UDim2.new(0, 0, 0, 65)
    levelNumText.BackgroundTransparency = 1
    levelNumText.Text = "Level " .. newLevel
    levelNumText.Font = Enum.Font.GothamBold
    levelNumText.TextSize = 28
    levelNumText.TextColor3 = Color3.new(1, 1, 1)
    levelNumText.ZIndex = 21
    levelNumText.Parent = levelUpFrame
    
    -- Entrance animation
    levelUpFrame.Position = UDim2.new(0.5, -200, 0.3, 0)
    TweenService:Create(levelUpFrame, TweenInfo.new(0.5, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
        Position = UDim2.new(0.5, -200, 0.5, -60)
    }):Play()
    
    -- Scale effect on text
    TweenService:Create(mainText, TweenInfo.new(0.3, Enum.EasingStyle.Bounce), {
        TextSize = 52
    }):Play()
    
    -- Sparkles
    for i = 1, 20 do
        task.spawn(function()
            task.wait(math.random() * 0.5)
            local sparkle = Instance.new("Frame")
            sparkle.Size = UDim2.new(0, 6, 0, 6)
            sparkle.Position = UDim2.new(
                math.random() * 0.8 + 0.1,
                0,
                math.random() * 0.8 + 0.1,
                0
            )
            sparkle.BackgroundColor3 = Color3.fromRGB(255, 255, 100)
            sparkle.BackgroundTransparency = 0
            sparkle.BorderSizePixel = 0
            sparkle.ZIndex = 22
            sparkle.AnchorPoint = Vector2.new(0.5, 0.5)
            sparkle.Parent = levelUpFrame
            
            Instance.new("UICorner", sparkle).CornerRadius = UDim.new(1, 0)
            
            TweenService:Create(sparkle, TweenInfo.new(0.8), {
                Size = UDim2.new(0, 0, 0, 0),
                BackgroundTransparency = 1
            }):Play()
            
            task.delay(0.8, function() sparkle:Destroy() end)
        end)
    end
    
    -- Fade out after 2.5 seconds
    task.delay(2, function()
        TweenService:Create(levelUpFrame, TweenInfo.new(0.5), {
            Position = UDim2.new(0.5, -200, 0.3, 0),
            BackgroundTransparency = 1
        }):Play()
        
        task.delay(0.5, function()
            levelUpFrame:Destroy()
        end)
    end)
end

-- ===== EXP Gain Notification =====
local function showExpGain(amount, source, bonus)
    local gui = player.PlayerGui:FindFirstChild("LevelUI")
    if not gui then return end
    
    local expNotif = Instance.new("TextLabel")
    expNotif.Size = UDim2.new(0, 150, 0, 25)
    expNotif.Position = UDim2.new(0, 16, 0, 60)
    expNotif.BackgroundTransparency = 1
    expNotif.Text = "+" .. amount .. " EXP"
    if bonus then
        expNotif.Text = expNotif.Text .. " (+" .. math.floor(bonus * 100) .. "% bonus)"
    end
    expNotif.Font = Enum.Font.GothamBold
    expNotif.TextSize = 14
    expNotif.TextColor3 = Color3.fromRGB(100, 220, 150)
    expNotif.TextStrokeTransparency = 0
    expNotif.TextStrokeColor3 = Color3.new(0, 0, 0)
    expNotif.ZIndex = 10
    expNotif.TextXAlignment = Enum.TextXAlignment.Left
    expNotif.Parent = gui
    
    TweenService:Create(expNotif, TweenInfo.new(1.5), {
        Position = UDim2.new(0, 16, 0, 30),
        TextTransparency = 1,
        TextStrokeTransparency = 1
    }):Play()
    
    game:GetService("Debris"):AddItem(expNotif, 1.5)
end

-- ===== Update EXP Bar =====
local levelUI = createLevelUI()
local statPointsUI = createStatPointsUI()

local function updateExpBar(data)
    local expFrame = levelUI:FindFirstChild("ExpBar")
    if not expFrame then return end
    
    -- อัพเดท level badge
    local levelBadge = expFrame:FindFirstChild("LevelBadge")
    if levelBadge then
        local levelText = levelBadge:FindFirstChild("LevelText")
        if levelText then levelText.Text = tostring(data.level) end
    end
    
    -- อัพเดท prestige
    local prestigeLabel = expFrame:FindFirstChild("PrestigeLabel")
    if prestigeLabel then
        if data.prestige and data.prestige > 0 then
            prestigeLabel.Text = string.rep("★", data.prestige)
        else
            prestigeLabel.Text = ""
        end
    end
    
    -- อัพเดท EXP bar
    local expBg = expFrame:FindFirstChild("ExpBackground")
    if expBg then
        local expFill = expBg:FindFirstChild("ExpFill")
        if expFill and data.expRequired > 0 then
            local percent = data.exp / data.expRequired
            TweenService:Create(expFill, TweenInfo.new(0.5), {
                Size = UDim2.new(percent, 0, 1, 0)
            }):Play()
        end
    end
    
    -- อัพเดท EXP text
    local expText = expFrame:FindFirstChild("ExpText")
    if expText then
        expText.Text = string.format("%d / %d EXP", data.exp or 0, data.expRequired or 100)
    end
    
    -- อัพเดท stat points panel
    if statPointsUI then
        local pointsLabel = statPointsUI:FindFirstChild("PointsLabel")
        if pointsLabel then
            pointsLabel.Text = "Points: " .. (data.statPoints or 0)
        end
        
        -- Show panel ถ้ามี stat points
        statPointsUI.Visible = (data.statPoints or 0) > 0
    end
    
    -- อัพเดท allocated stats
    if data.allocatedStats then
        for statName, value in pairs(data.allocatedStats) do
            local row = statPointsUI and statPointsUI:FindFirstChild(statName .. "Row")
            if row then
                local valueLabel = row:FindFirstChild("Value")
                if valueLabel then valueLabel.Text = tostring(value) end
            end
        end
    end
end

-- ===== Event Handlers =====
levelUpdateEvent.OnClientEvent:Connect(function(data)
    if data.type == "Update" then
        updateExpBar(data)
    elseif data.type == "MilestoneReward" then
        -- แสดง notification
        print("[LevelUI] Milestone: " .. data.message)
    elseif data.type == "PrestigeAvailable" then
        print("[LevelUI] Prestige available!")
    elseif data.type == "Prestige" then
        print("[LevelUI] Prestiged! " .. data.message)
    end
end)

levelUpEvent.OnClientEvent:Connect(function(data)
    showLevelUpAnimation(data.newLevel)
    updateExpBar({
        level = data.newLevel,
        exp = 0,
        expRequired = 100,
        statPoints = data.statPoints
    })
end)

expGainEvent.OnClientEvent:Connect(function(data)
    showExpGain(data.amount, data.source, data.bonus)
end)
```

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: EXP Boost System
สร้างระบบ "EXP Boost" ที่ผู้เล่นสามารถซื้อ item เพื่อเพิ่ม EXP rate ชั่วคราว

### แบบฝึกหัดที่ 2: Class Level System
สร้างระบบ Class Level แยกออกมา เช่น Warrior Level, Mage Level ที่เพิ่มขึ้นเมื่อใช้ skill ของ class นั้น

### แบบฝึกหัดที่ 3: Achievement Levels
สร้างระบบ Achievement ที่ให้ EXP เมื่อทำสำเร็จ เช่น "Kill 100 enemies", "Explore 10 zones"

### แบบฝึกหัดที่ 4: Party EXP Share
สร้างระบบแบ่ง EXP ใน party โดย EXP จะถูกแบ่งตามจำนวนสมาชิก

## สรุป

ระบบเลเวลที่สมบูรณ์ประกอบด้วย:
- **LevelConfig** - สูตรคำนวณ EXP และตาราง rewards
- **LevelManager** - จัดการ EXP, level up, stat points
- **Prestige System** - ระบบ reset level พร้อม bonus
- **Stat Point Allocation** - ผู้เล่นเลือก stats เองได้
- **Milestone Rewards** - รางวัลพิเศษตาม level
- **Level UI** - EXP bar, level up animation, stat points panel
- **DataStore Integration** - บันทึกความก้าวหน้า
