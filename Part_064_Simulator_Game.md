# Part 64: การสร้าง Simulator Game

## บทนำ

Simulator Game เป็นประเภทเกมที่นิยมมากใน Roblox ตัวอย่างเช่น Pet Simulator, Mining Simulator, Clicking Simulator ผู้เล่นจะเก็บสะสมค่าต่างๆ และอัพเกรดตัวเองเรื่อยๆ ในบทนี้เราจะสร้าง Mining Simulator พร้อมระบบครบครัน

---

## 64.1 โครงสร้าง Simulator Game

```
ServerScriptService
├── SimulatorCore (Script)
├── MiningSystem (Script)
├── UpgradeSystem (Script)
└── RebornSystem (Script)

ReplicatedStorage
├── Remotes
│   ├── Mine (RemoteEvent)
│   ├── BuyUpgrade (RemoteEvent)
│   ├── Reborn (RemoteEvent)
│   └── GetData (RemoteFunction)
└── Modules
    ├── SimData (ModuleScript)
    ├── UpgradeData (ModuleScript)
    └── OreData (ModuleScript)

StarterGui
├── SimHUD (ScreenGui)
└── ShopGUI (ScreenGui)
```

---

## 64.2 ระบบหลัก (Core System)

### 64.2.1 SimData Module

```lua
-- ReplicatedStorage/Modules/SimData.lua
-- ข้อมูลพื้นฐานของ Simulator

local SimData = {}

function SimData.getDefault()
    return {
        -- ทรัพยากร
        coins = 0,
        gems = 0,
        ores = 0,
        
        -- Stats
        miningSpeed = 1,          -- ขุดกี่ครั้งต่อวินาที
        miningStrength = 1,       -- ขุดได้กี่ layer ต่อครั้ง
        oreMultiplier = 1,        -- คูณ ore ที่ได้
        coinMultiplier = 1,       -- คูณ coin ที่ได้
        luckMultiplier = 1,       -- โอกาสได้ rare ore
        
        -- Upgrades ที่ซื้อแล้ว
        upgrades = {},
        
        -- Reborn
        rebornCount = 0,
        rebornMultiplier = 1,     -- bonus จาก reborn
        
        -- Area ที่ปลดล็อก
        unlockedAreas = {"starter_mine"},
        currentArea = "starter_mine",
        
        -- Statistics
        totalMines = 0,
        totalCoinsEarned = 0,
        
        -- Timestamp
        lastSave = 0,
        firstJoin = os.time(),
    }
end

return SimData
```

### 64.2.2 SimulatorCore Script

```lua
-- ServerScriptService/SimulatorCore.lua
-- ระบบหลัก Simulator

local Players = game:GetService("Players")
local DataStoreService = game:GetService("DataStoreService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local SimData = require(ReplicatedStorage.Modules.SimData)

local dataStore = DataStoreService:GetDataStore("SimulatorV3")
local playerCache = {}

-- โหลดข้อมูล
local function loadData(player)
    local key = "sim_" .. player.UserId
    local success, data = pcall(function()
        return dataStore:GetAsync(key)
    end)
    
    local finalData
    if success and data then
        finalData = data
        -- เพิ่ม fields ที่หายไป (เมื่ออัพเดทเกม)
        local defaults = SimData.getDefault()
        for k, v in pairs(defaults) do
            if finalData[k] == nil then
                finalData[k] = v
            end
        end
    else
        finalData = SimData.getDefault()
    end
    
    playerCache[player.UserId] = finalData
    setupPlayerValues(player, finalData)
    
    print(player.Name .. " โหลดข้อมูล: " .. finalData.coins .. " coins")
end

-- ตั้งค่า Values
function setupPlayerValues(player, data)
    -- Leaderstats
    local leaderstats = Instance.new("Folder")
    leaderstats.Name = "leaderstats"
    leaderstats.Parent = player
    
    local coinsVal = Instance.new("NumberValue")
    coinsVal.Name = "Coins"
    coinsVal.Value = data.coins
    coinsVal.Parent = leaderstats
    
    local gemsVal = Instance.new("NumberValue")
    gemsVal.Name = "Gems"
    gemsVal.Value = data.gems
    gemsVal.Parent = leaderstats
    
    local rebornVal = Instance.new("IntValue")
    rebornVal.Name = "Reborn"
    rebornVal.Value = data.rebornCount
    rebornVal.Parent = leaderstats
    
    -- SimStats folder
    local simStats = Instance.new("Folder")
    simStats.Name = "SimStats"
    simStats.Parent = player
    
    local miningSpeedVal = Instance.new("NumberValue")
    miningSpeedVal.Name = "MiningSpeed"
    miningSpeedVal.Value = data.miningSpeed
    miningSpeedVal.Parent = simStats
    
    local oreMultVal = Instance.new("NumberValue")
    oreMultVal.Name = "OreMultiplier"
    oreMultVal.Value = data.oreMultiplier
    oreMultVal.Parent = simStats
end

-- บันทึกข้อมูล
local function saveData(player)
    local data = playerCache[player.UserId]
    if not data then return end
    
    data.lastSave = os.time()
    
    -- อัพเดทจาก Values
    local leaderstats = player:FindFirstChild("leaderstats")
    if leaderstats then
        if leaderstats:FindFirstChild("Coins") then
            data.coins = leaderstats.Coins.Value
        end
        if leaderstats:FindFirstChild("Gems") then
            data.gems = leaderstats.Gems.Value
        end
    end
    
    local success, err = pcall(function()
        dataStore:SetAsync("sim_" .. player.UserId, data)
    end)
    
    if not success then
        warn("บันทึกล้มเหลว " .. player.Name .. ": " .. err)
    end
end

-- อัพเดท Coins
local function addCoins(player, amount)
    local data = playerCache[player.UserId]
    if not data then return end
    
    local finalAmount = math.floor(amount * data.coinMultiplier * data.rebornMultiplier)
    data.coins = data.coins + finalAmount
    data.totalCoinsEarned = data.totalCoinsEarned + finalAmount
    
    local leaderstats = player:FindFirstChild("leaderstats")
    if leaderstats and leaderstats:FindFirstChild("Coins") then
        leaderstats.Coins.Value = data.coins
    end
    
    return finalAmount
end

-- Auto-save
task.spawn(function()
    while true do
        task.wait(60)
        for _, player in ipairs(Players:GetPlayers()) do
            saveData(player)
        end
        print("Auto-saved all players")
    end
end)

Players.PlayerAdded:Connect(function(player)
    loadData(player)
    
    -- Offline earnings (passive income)
    local data = playerCache[player.UserId]
    if data and data.lastSave > 0 then
        local offlineTime = os.time() - data.lastSave
        if offlineTime > 60 then
            local passiveIncome = calculatePassiveIncome(player, offlineTime)
            if passiveIncome > 0 then
                addCoins(player, passiveIncome)
                print(player.Name .. " รับ offline income: " .. passiveIncome .. " coins (" .. math.floor(offlineTime/60) .. " นาที)")
            end
        end
    end
end)

Players.PlayerRemoving:Connect(function(player)
    saveData(player)
    playerCache[player.UserId] = nil
end)

-- คำนวณ passive income
function calculatePassiveIncome(player, seconds)
    local data = playerCache[player.UserId]
    if not data then return 0 end
    
    -- ตรวจสอบว่ามี auto-miner upgrades หรือไม่
    local autoMinerLevel = data.upgrades["auto_miner"] or 0
    if autoMinerLevel == 0 then return 0 end
    
    local coinsPerSecond = autoMinerLevel * 10
    return math.floor(coinsPerSecond * math.min(seconds, 8 * 3600))  -- max 8 ชั่วโมง
end

game:BindToClose(function()
    for _, player in ipairs(Players:GetPlayers()) do
        saveData(player)
    end
end)

-- Public API
local SimCore = {
    addCoins = addCoins,
    getData = function(player) return playerCache[player.UserId] end,
}

return SimCore
```

---

## 64.3 ระบบการขุด (Mining System)

```lua
-- ServerScriptService/MiningSystem.lua
-- ระบบขุด Ore

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")

local OreData = require(ReplicatedStorage.Modules.OreData)

-- Remotes
local Remotes = ReplicatedStorage:WaitForChild("Remotes")
local mineRemote = Instance.new("RemoteEvent")
mineRemote.Name = "Mine"
mineRemote.Parent = Remotes

-- ข้อมูล cooldown
local mineCooldowns = {}
local miningTargets = {}  -- player -> ore

-- สร้าง Ore ใน Map
local function spawnOre(oreType, position, parent)
    local oreConfig = OreData.getOre(oreType)
    if not oreConfig then return end
    
    local ore = Instance.new("Part")
    ore.Name = oreType
    ore.Size = Vector3.new(4, 4, 4)
    ore.Position = position
    ore.Anchored = true
    ore.Material = oreConfig.material or Enum.Material.Rock
    ore.BrickColor = oreConfig.color
    ore:SetAttribute("OreType", oreType)
    ore:SetAttribute("MaxHealth", oreConfig.health)
    ore:SetAttribute("CurrentHealth", oreConfig.health)
    
    -- เพิ่ม texture
    if oreConfig.texture then
        local texture = Instance.new("SpecialMesh")
        texture.MeshType = Enum.MeshType.Sphere
        texture.Parent = ore
    end
    
    ore.Parent = parent or workspace
    
    -- เพิ่มแสง (สำหรับ rare ores)
    if oreConfig.glows then
        local light = Instance.new("PointLight")
        light.Color = oreConfig.lightColor or Color3.new(1, 1, 1)
        light.Brightness = 3
        light.Range = 10
        light.Parent = ore
    end
    
    return ore
end

-- ฟังก์ชันขุด
local function mineOre(player, ore)
    local data = require(script.Parent.SimulatorCore).getData(player)
    if not data then return end
    
    -- คำนวณ damage
    local damage = data.miningStrength * (1 + (data.upgrades["pick_strength"] or 0) * 0.5)
    
    local currentHealth = ore:GetAttribute("CurrentHealth") or 0
    local newHealth = currentHealth - damage
    
    if newHealth <= 0 then
        -- แตก!
        local oreType = ore:GetAttribute("OreType")
        local oreConfig = OreData.getOre(oreType)
        
        if oreConfig then
            -- คำนวณของที่ได้
            local baseCoins = oreConfig.baseCoins
            local finalCoins = require(script.Parent.SimulatorCore).addCoins(player, baseCoins)
            
            -- โอกาสได้ Gems
            if oreConfig.gemChance and math.random() < oreConfig.gemChance * data.luckMultiplier then
                local gemAmount = math.random(1, oreConfig.maxGems or 1)
                data.gems = data.gems + gemAmount
                local leaderstats = player:FindFirstChild("leaderstats")
                if leaderstats and leaderstats:FindFirstChild("Gems") then
                    leaderstats.Gems.Value = data.gems
                end
                print(player.Name .. " ได้ " .. gemAmount .. " Gems!")
            end
            
            data.totalMines = data.totalMines + 1
            
            -- Floating text notification
            local notifyEvent = Remotes:FindFirstChild("FloatingText")
            if notifyEvent then
                notifyEvent:FireClient(player, ore.Position, "+" .. finalCoins .. " 💰")
            end
            
            -- เอฟเฟกต์แตก
            local breakEffect = Instance.new("Part")
            breakEffect.Size = ore.Size
            breakEffect.CFrame = ore.CFrame
            breakEffect.Anchored = true
            breakEffect.CanCollide = false
            breakEffect.BrickColor = ore.BrickColor
            breakEffect.Material = ore.Material
            breakEffect.Parent = workspace
            
            local tween = TweenService:Create(breakEffect, TweenInfo.new(0.3), {
                Size = ore.Size * 2,
                Transparency = 1
            })
            tween:Play()
            game:GetService("Debris"):AddItem(breakEffect, 0.5)
            
            -- ลบ ore และ respawn
            local orePosition = ore.Position
            local oreParent = ore.Parent
            ore:Destroy()
            
            task.delay(oreConfig.respawnTime or 5, function()
                spawnOre(oreType, orePosition, oreParent)
            end)
        end
    else
        ore:SetAttribute("CurrentHealth", newHealth)
        
        -- แสดง health bar
        local healthPercent = newHealth / (ore:GetAttribute("MaxHealth") or 1)
        ore.BrickColor = BrickColor.new(
            healthPercent > 0.5 and ore:GetAttribute("OreType") and 
            OreData.getOre(ore:GetAttribute("OreType")).color or
            BrickColor.new("Bright red")
        )
    end
end

-- Remote handler
mineRemote.OnServerEvent:Connect(function(player, oreRef)
    -- ตรวจสอบ cooldown
    local now = tick()
    local data = require(script.Parent.SimulatorCore).getData(player)
    if not data then return end
    
    local cooldown = 1 / data.miningSpeed
    if mineCooldowns[player.UserId] and now < mineCooldowns[player.UserId] then
        return
    end
    mineCooldowns[player.UserId] = now + cooldown
    
    -- ตรวจสอบ ore
    if not oreRef or not oreRef.Parent then return end
    
    -- ตรวจสอบระยะ
    local character = player.Character
    if not character or not character:FindFirstChild("HumanoidRootPart") then return end
    
    local dist = (character.HumanoidRootPart.Position - oreRef.Position).Magnitude
    if dist > 20 then
        warn(player.Name .. " พยายามขุด ore ที่ไกลเกินไป (" .. math.floor(dist) .. " studs)")
        return
    end
    
    mineOre(player, oreRef)
end)

-- สร้าง Ores ใน Starter Mine
local function setupStarterMine()
    local mineFolder = workspace:FindFirstChild("Mines") or Instance.new("Folder")
    mineFolder.Name = "Mines"
    mineFolder.Parent = workspace
    
    local starterMine = Instance.new("Folder")
    starterMine.Name = "starter_mine"
    starterMine.Parent = mineFolder
    
    local orePositions = {}
    
    -- สุ่มตำแหน่ง Ore
    for i = 1, 30 do
        local x = math.random(-50, 50)
        local z = math.random(-50, 50)
        
        -- เลือกประเภท ore ตาม weight
        local rand = math.random(100)
        local oreType
        if rand <= 60 then
            oreType = "coal"
        elseif rand <= 80 then
            oreType = "iron"
        elseif rand <= 93 then
            oreType = "gold"
        elseif rand <= 99 then
            oreType = "diamond"
        else
            oreType = "crystal"
        end
        
        spawnOre(oreType, Vector3.new(x, 5, z), starterMine)
    end
    
    print("สร้าง Starter Mine เสร็จแล้ว - " .. #starterMine:GetChildren() .. " ores")
end

setupStarterMine()
```

---

## 64.4 ระบบ Upgrade

```lua
-- ReplicatedStorage/Modules/UpgradeData.lua
-- ข้อมูล Upgrades ทั้งหมด

local UpgradeData = {}

local upgrades = {
    -- Pickaxe Upgrades
    ["pick_speed"] = {
        name = "ความเร็วขุด",
        description = "เพิ่มความเร็วขุด {value}%",
        icon = "⛏️",
        maxLevel = 50,
        
        -- ราคาแต่ละ level
        baseCost = 100,
        costMultiplier = 1.5,  -- cost * 1.5^level
        
        -- ผลต่อ stats
        effect = function(level) 
            return {stat = "miningSpeed", value = 1 + level * 0.1}  -- +10% ต่อ level
        end,
        
        currency = "coins"  -- ซื้อด้วย coins
    },
    
    ["pick_strength"] = {
        name = "พลังขุด",
        description = "เพิ่มพลังทำลาย ore {value}%",
        icon = "💪",
        maxLevel = 50,
        baseCost = 150,
        costMultiplier = 1.6,
        
        effect = function(level)
            return {stat = "miningStrength", value = 1 + level * 0.2}
        end,
        
        currency = "coins"
    },
    
    ["coin_boost"] = {
        name = "เพิ่ม Coins",
        description = "ได้รับ Coins มากขึ้น {value}%",
        icon = "💰",
        maxLevel = 100,
        baseCost = 500,
        costMultiplier = 1.4,
        
        effect = function(level)
            return {stat = "coinMultiplier", value = 1 + level * 0.15}
        end,
        
        currency = "coins"
    },
    
    ["luck_boost"] = {
        name = "โชคลาภ",
        description = "เพิ่มโอกาสได้ rare items",
        icon = "🍀",
        maxLevel = 20,
        baseCost = 50,
        costMultiplier = 2.0,
        
        effect = function(level)
            return {stat = "luckMultiplier", value = 1 + level * 0.25}
        end,
        
        currency = "gems"  -- ซื้อด้วย gems
    },
    
    ["auto_miner"] = {
        name = "หุ่น Auto Miner",
        description = "ขุดอัตโนมัติเมื่อออฟไลน์",
        icon = "🤖",
        maxLevel = 10,
        baseCost = 200,
        costMultiplier = 3.0,
        
        effect = function(level)
            return {stat = "autoMine", value = level * 10}  -- 10 coins/sec ต่อ level
        end,
        
        currency = "gems"
    },
}

function UpgradeData.getUpgrade(upgradeId)
    return upgrades[upgradeId]
end

function UpgradeData.calculateCost(upgradeId, currentLevel)
    local upgrade = upgrades[upgradeId]
    if not upgrade then return 0 end
    
    return math.floor(upgrade.baseCost * (upgrade.costMultiplier ^ currentLevel))
end

function UpgradeData.getAllUpgrades()
    return upgrades
end

return UpgradeData
```

### 64.4.1 Upgrade System Script

```lua
-- ServerScriptService/UpgradeSystem.lua
-- ระบบซื้อ Upgrade

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local UpgradeData = require(ReplicatedStorage.Modules.UpgradeData)

local Remotes = ReplicatedStorage:WaitForChild("Remotes")
local buyUpgradeRemote = Instance.new("RemoteEvent")
buyUpgradeRemote.Name = "BuyUpgrade"
buyUpgradeRemote.Parent = Remotes

local upgradeResultEvent = Instance.new("RemoteEvent")
upgradeResultEvent.Name = "UpgradeResult"
upgradeResultEvent.Parent = Remotes

-- ซื้อ Upgrade
local function buyUpgrade(player, upgradeId)
    local SimCore = require(script.Parent.SimulatorCore)
    local data = SimCore.getData(player)
    if not data then return end
    
    local upgrade = UpgradeData.getUpgrade(upgradeId)
    if not upgrade then
        warn("ไม่พบ upgrade: " .. upgradeId)
        return
    end
    
    local currentLevel = data.upgrades[upgradeId] or 0
    
    -- ตรวจสอบ max level
    if currentLevel >= upgrade.maxLevel then
        upgradeResultEvent:FireClient(player, false, "อัพเกรดถึง max level แล้ว!")
        return
    end
    
    -- คำนวณราคา
    local cost = UpgradeData.calculateCost(upgradeId, currentLevel)
    
    -- ตรวจสอบว่ามีเงินพอ
    local currency = upgrade.currency
    local leaderstats = player:FindFirstChild("leaderstats")
    
    if currency == "coins" then
        if data.coins < cost then
            upgradeResultEvent:FireClient(player, false, "Coins ไม่พอ! ต้องการ " .. cost)
            return
        end
        data.coins = data.coins - cost
        if leaderstats and leaderstats:FindFirstChild("Coins") then
            leaderstats.Coins.Value = data.coins
        end
    elseif currency == "gems" then
        if data.gems < cost then
            upgradeResultEvent:FireClient(player, false, "Gems ไม่พอ! ต้องการ " .. cost)
            return
        end
        data.gems = data.gems - cost
        if leaderstats and leaderstats:FindFirstChild("Gems") then
            leaderstats.Gems.Value = data.gems
        end
    end
    
    -- อัพเกรด
    data.upgrades[upgradeId] = currentLevel + 1
    local newLevel = data.upgrades[upgradeId]
    
    -- ใช้ effect
    local effect = upgrade.effect(newLevel)
    if effect then
        data[effect.stat] = effect.value
        
        -- อัพเดท SimStats
        local simStats = player:FindFirstChild("SimStats")
        if simStats and simStats:FindFirstChild(effect.stat) then
            simStats[effect.stat].Value = effect.value
        end
    end
    
    -- แจ้งผล
    upgradeResultEvent:FireClient(player, true, upgrade.name .. " Level " .. newLevel, data.upgrades, data)
    print(player.Name .. " อัพเกรด " .. upgrade.name .. " เป็น Level " .. newLevel)
end

buyUpgradeRemote.OnServerEvent:Connect(buyUpgrade)
```

---

## 64.5 ระบบ Reborn

```lua
-- ServerScriptService/RebornSystem.lua
-- ระบบ Reborn (Prestige)

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local Remotes = ReplicatedStorage:WaitForChild("Remotes")
local rebornRemote = Instance.new("RemoteEvent")
rebornRemote.Name = "Reborn"
rebornRemote.Parent = Remotes

-- Reborn Requirements
local REBORN_REQUIREMENTS = {
    {coins = 1000000, reward = "x1.5 Multiplier"},
    {coins = 10000000, reward = "x2.0 Multiplier"},
    {coins = 100000000, reward = "x3.0 Multiplier"},
}

-- คำนวณ Reborn Multiplier
local function getRebornMultiplier(rebornCount)
    local multipliers = {1, 1.5, 2.0, 3.0, 5.0, 8.0, 12.0, 20.0}
    return multipliers[math.min(rebornCount + 1, #multipliers)] or 20.0
end

-- Reborn
local function doReborn(player)
    local SimCore = require(script.Parent.SimulatorCore)
    local data = SimCore.getData(player)
    if not data then return end
    
    -- ตรวจสอบว่าถึง requirement
    local nextReborn = REBORN_REQUIREMENTS[data.rebornCount + 1]
    if not nextReborn then
        local notifyRemote = Remotes:FindFirstChild("Notification")
        if notifyRemote then
            notifyRemote:FireClient(player, "คุณ Reborn สูงสุดแล้ว!", "info")
        end
        return
    end
    
    if data.coins < nextReborn.coins then
        local notifyRemote = Remotes:FindFirstChild("Notification")
        if notifyRemote then
            notifyRemote:FireClient(player, "ต้องการ " .. nextReborn.coins .. " Coins!", "error")
        end
        return
    end
    
    -- ยืนยัน Reborn
    -- (ใน production ควรมี confirmation dialog ก่อน)
    
    -- รีเซ็ต
    data.coins = 0
    data.gems = 0
    data.upgrades = {}
    data.rebornCount = data.rebornCount + 1
    
    -- คำนวณ multiplier ใหม่
    data.rebornMultiplier = getRebornMultiplier(data.rebornCount)
    
    -- รีเซ็ต stats กลับไปพื้นฐาน
    data.miningSpeed = 1
    data.miningStrength = 1
    data.oreMultiplier = 1
    data.coinMultiplier = 1
    data.luckMultiplier = 1
    
    -- อัพเดท Values
    local leaderstats = player:FindFirstChild("leaderstats")
    if leaderstats then
        if leaderstats:FindFirstChild("Coins") then leaderstats.Coins.Value = 0 end
        if leaderstats:FindFirstChild("Gems") then leaderstats.Gems.Value = 0 end
        if leaderstats:FindFirstChild("Reborn") then leaderstats.Reborn.Value = data.rebornCount end
    end
    
    print(player.Name .. " Reborn! #" .. data.rebornCount .. " Multiplier: x" .. data.rebornMultiplier)
    
    -- แจ้ง Client
    local rebornEvent = Remotes:FindFirstChild("RebornSuccess")
    if rebornEvent then
        rebornEvent:FireClient(player, data.rebornCount, data.rebornMultiplier)
    end
end

rebornRemote.OnServerEvent:Connect(doReborn)
```

---

## 64.6 Ore Database

```lua
-- ReplicatedStorage/Modules/OreData.lua
-- ข้อมูล Ore ทั้งหมด

local OreData = {}

local ores = {
    coal = {
        name = "ถ่านหิน",
        color = BrickColor.new("Black"),
        material = Enum.Material.Rock,
        health = 3,
        baseCoins = 5,
        gemChance = 0,
        respawnTime = 3,
        weight = 60,  -- ความน่าจะเป็นที่จะ spawn
        glows = false
    },
    iron = {
        name = "เหล็ก",
        color = BrickColor.new("Medium stone grey"),
        material = Enum.Material.SmoothPlastic,
        health = 5,
        baseCoins = 15,
        gemChance = 0.02,
        maxGems = 1,
        respawnTime = 5,
        weight = 25,
        glows = false
    },
    gold = {
        name = "ทอง",
        color = BrickColor.new("Bright yellow"),
        material = Enum.Material.Neon,
        health = 8,
        baseCoins = 50,
        gemChance = 0.05,
        maxGems = 2,
        respawnTime = 10,
        weight = 10,
        glows = true,
        lightColor = Color3.fromRGB(255, 215, 0)
    },
    diamond = {
        name = "เพชร",
        color = BrickColor.new("Cyan"),
        material = Enum.Material.Ice,
        health = 15,
        baseCoins = 200,
        gemChance = 0.1,
        maxGems = 5,
        respawnTime = 30,
        weight = 4,
        glows = true,
        lightColor = Color3.fromRGB(100, 200, 255)
    },
    crystal = {
        name = "คริสตัล",
        color = BrickColor.new("Hot pink"),
        material = Enum.Material.Neon,
        health = 30,
        baseCoins = 1000,
        gemChance = 0.3,
        maxGems = 20,
        respawnTime = 120,
        weight = 1,
        glows = true,
        lightColor = Color3.fromRGB(255, 100, 200)
    },
}

function OreData.getOre(oreType)
    return ores[oreType]
end

return OreData
```

---

## 64.7 GUI สำหรับ Simulator

```lua
-- StarterGui/SimHUD/LocalScript

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local playerGui = player.PlayerGui
local simHUD = script.Parent

-- สร้าง Stats Display
local statsFrame = Instance.new("Frame")
statsFrame.Name = "StatsFrame"
statsFrame.Size = UDim2.new(0, 250, 0, 120)
statsFrame.Position = UDim2.new(0, 10, 0.5, -60)
statsFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
statsFrame.BackgroundTransparency = 0.3
statsFrame.BorderSizePixel = 0
statsFrame.Parent = simHUD

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 12)
corner.Parent = statsFrame

-- Stats Labels
local statsData = {
    {name = "coinsLabel", text = "💰 Coins: 0", yPos = 0},
    {name = "gemsLabel", text = "💎 Gems: 0", yPos = 0.25},
    {name = "speedLabel", text = "⛏️ Speed: 1x", yPos = 0.5},
    {name = "rebornLabel", text = "🔄 Reborn: 0", yPos = 0.75},
}

local labels = {}
for _, data in ipairs(statsData) do
    local label = Instance.new("TextLabel")
    label.Name = data.name
    label.Size = UDim2.new(1, -10, 0.25, 0)
    label.Position = UDim2.new(0, 5, data.yPos, 0)
    label.BackgroundTransparency = 1
    label.Text = data.text
    label.TextColor3 = Color3.new(1, 1, 1)
    label.TextScaled = true
    label.Font = Enum.Font.Gotham
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = statsFrame
    labels[data.name] = label
end

-- Floating Text Function
local function createFloatingText(position, text)
    local screenPos, onScreen = workspace.CurrentCamera:WorldToScreenPoint(position)
    if not onScreen then return end
    
    local floatLabel = Instance.new("TextLabel")
    floatLabel.Size = UDim2.new(0, 100, 0, 30)
    floatLabel.Position = UDim2.new(0, screenPos.X - 50, 0, screenPos.Y - 15)
    floatLabel.BackgroundTransparency = 1
    floatLabel.Text = text
    floatLabel.TextColor3 = Color3.fromRGB(255, 215, 0)
    floatLabel.TextScaled = true
    floatLabel.Font = Enum.Font.GothamBold
    floatLabel.ZIndex = 10
    floatLabel.Parent = simHUD
    
    -- Animate up and fade
    local tween = TweenService:Create(floatLabel, TweenInfo.new(1.5, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
        Position = UDim2.new(0, screenPos.X - 50, 0, screenPos.Y - 80),
        TextTransparency = 1,
    })
    tween:Play()
    tween.Completed:Connect(function()
        floatLabel:Destroy()
    end)
end

-- อัพเดท Stats
game:GetService("RunService").RenderStepped:Connect(function()
    local leaderstats = player:FindFirstChild("leaderstats")
    local simStats = player:FindFirstChild("SimStats")
    
    if leaderstats then
        if labels.coinsLabel and leaderstats:FindFirstChild("Coins") then
            local coins = leaderstats.Coins.Value
            -- แสดงตัวเลขแบบสั้น
            local displayCoins
            if coins >= 1e9 then
                displayCoins = string.format("%.1fB", coins/1e9)
            elseif coins >= 1e6 then
                displayCoins = string.format("%.1fM", coins/1e6)
            elseif coins >= 1e3 then
                displayCoins = string.format("%.1fK", coins/1e3)
            else
                displayCoins = tostring(math.floor(coins))
            end
            labels.coinsLabel.Text = "💰 " .. displayCoins
        end
        
        if labels.gemsLabel and leaderstats:FindFirstChild("Gems") then
            labels.gemsLabel.Text = "💎 " .. math.floor(leaderstats.Gems.Value)
        end
        
        if labels.rebornLabel and leaderstats:FindFirstChild("Reborn") then
            labels.rebornLabel.Text = "🔄 Reborn: " .. leaderstats.Reborn.Value
        end
    end
    
    if simStats and labels.speedLabel then
        if simStats:FindFirstChild("MiningSpeed") then
            labels.speedLabel.Text = "⛏️ " .. string.format("%.1f", simStats.MiningSpeed.Value) .. "x"
        end
    end
end)

-- รับ Floating Text Events
local Remotes = ReplicatedStorage:WaitForChild("Remotes")
local floatTextEvent = Remotes:WaitForChild("FloatingText")

floatTextEvent.OnClientEvent:Connect(function(position, text)
    createFloatingText(position, text)
end)
```

---

## 64.8 ข้อผิดพลาดที่พบบ่อย

### ข้อผิดพลาด 1: Number Overflow

```lua
-- ❌ ผิด: ใช้ int ธรรมดา ทำให้ overflow ที่ตัวเลขใหญ่
local coins = 9999999999999  -- อาจ overflow!

-- ✓ ถูก: ใช้ NumberValue และตรวจสอบขอบเขต
local MAX_COINS = 1e15  -- 1 quadrillion

local function safeAddCoins(current, amount)
    local newAmount = current + amount
    return math.min(newAmount, MAX_COINS)
end
```

### ข้อผิดพลาด 2: Cooldown Bypass

```lua
-- ✓ ถูก: ตรวจสอบ cooldown บน Server เสมอ
mineRemote.OnServerEvent:Connect(function(player, oreRef)
    local now = tick()
    local lastMine = mineCooldowns[player.UserId] or 0
    local cooldown = 1 / (data.miningSpeed or 1)
    
    if now - lastMine < cooldown then
        -- ผู้เล่นพยายามโกง ไม่ตอบสนอง
        return
    end
    
    mineCooldowns[player.UserId] = now
    -- ดำเนินการขุดต่อ...
end)
```

---

## 64.9 แบบฝึกหัด

### แบบฝึกหัดที่ 1: เพิ่ม Area ใหม่
สร้าง Area ที่ 2 "Deep Mine" ที่:
- ต้องการ 100,000 Coins เพื่อปลดล็อก
- มี Ore ที่ให้ผลตอบแทนสูงกว่า
- มี Ore พิเศษ "Emerald"

### แบบฝึกหัดที่ 2: สร้าง Daily Challenges
ระบบ Challenge รายวัน:
- ขุด 1000 Ores ในวันนี้
- รับ 10,000 Coins เป็นรางวัล
- Reset ทุกเที่ยงคืน

### แบบฝึกหัดที่ 3: สร้าง Hatch System
ระบบ Egg Hatch:
- ใช้ Gems ซื้อไข่
- ฟักไข่รับ Pet/Equipment แบบสุ่ม

---

## สรุป

ในบทนี้เราได้สร้าง Simulator Game ที่มี:
- ระบบ Mining
- ระบบ Upgrade
- ระบบ Reborn/Prestige
- Floating Text และ HUD
- DataStore สำหรับบันทึกข้อมูล

ในบทถัดไปเราจะสร้าง Tower Defense Game!
