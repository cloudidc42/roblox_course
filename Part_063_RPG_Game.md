# Part 63: การสร้าง RPG Game Framework

## บทนำ

RPG (Role-Playing Game) เป็นหนึ่งในประเภทเกมที่ซับซ้อนที่สุดใน Roblox แต่ก็เป็นที่นิยมมากที่สุดด้วย ในบทนี้เราจะสร้าง Framework พื้นฐานที่ครอบคลุมระบบ Stats, Combat, Inventory, Quest, และ Level Up

---

## 63.1 โครงสร้าง RPG Framework

```
ServerScriptService
├── RPGCore (Script)               -- ระบบหลัก
├── CombatSystem (Script)          -- ระบบต่อสู้
├── LevelSystem (Script)           -- ระบบเลเวล
├── InventorySystem (Script)       -- ระบบกระเป๋า
├── QuestSystem (Script)           -- ระบบเควส
└── NPCManager (Script)            -- จัดการ NPC

ReplicatedStorage
├── Remotes (Folder)
│   ├── AttackEnemy (RemoteEvent)
│   ├── EquipItem (RemoteEvent)
│   ├── AcceptQuest (RemoteEvent)
│   └── GetPlayerStats (RemoteFunction)
├── Modules (Folder)
│   ├── PlayerData (ModuleScript)
│   ├── ItemDatabase (ModuleScript)
│   ├── EnemyDatabase (ModuleScript)
│   └── QuestDatabase (ModuleScript)
└── Assets (Folder)

StarterPlayerScripts
└── RPGClient (LocalScript)

StarterGui
├── StatsHUD (ScreenGui)
├── InventoryGUI (ScreenGui)
└── QuestLog (ScreenGui)
```

---

## 63.2 ระบบ Player Stats

### 63.2.1 PlayerData Module

```lua
-- ReplicatedStorage/Modules/PlayerData.lua
-- Module สำหรับจัดการข้อมูลผู้เล่น

local PlayerData = {}

-- ค่า default สำหรับผู้เล่นใหม่
function PlayerData.getDefault()
    return {
        -- ข้อมูลพื้นฐาน
        level = 1,
        exp = 0,
        expToNext = 100,
        gold = 50,
        
        -- Stats พื้นฐาน
        stats = {
            maxHP = 100,
            currentHP = 100,
            maxMP = 50,
            currentMP = 50,
            attack = 10,
            defense = 5,
            speed = 16,
            luck = 5
        },
        
        -- Stat Points ที่รอการแจก
        statPoints = 0,
        
        -- Class
        class = "Warrior",  -- "Warrior", "Mage", "Archer", "Rogue"
        
        -- Inventory
        inventory = {},
        equipped = {
            weapon = nil,
            armor = nil,
            helmet = nil,
            boots = nil,
            accessory = nil
        },
        
        -- Quests
        activeQuests = {},
        completedQuests = {},
        
        -- Skills
        skills = {},
        skillPoints = 0,
        
        -- Achievements
        kills = 0,
        deaths = 0,
        questsCompleted = 0,
        
        -- Settings
        settings = {
            autoLoot = true,
            showDamage = true
        }
    }
end

-- คำนวณ EXP ที่ต้องการสำหรับเลเวลถัดไป
function PlayerData.calculateExpNeeded(level)
    -- สูตร: 100 * (level ^ 1.5)
    return math.floor(100 * (level ^ 1.5))
end

-- คำนวณ Stats จาก level และ class
function PlayerData.calculateStats(level, class)
    local baseStats = {
        Warrior = {maxHP = 150, attack = 15, defense = 10, maxMP = 30, speed = 14},
        Mage    = {maxHP = 80,  attack = 20, defense = 5,  maxMP = 100, speed = 15},
        Archer  = {maxHP = 100, attack = 18, defense = 7,  maxMP = 50,  speed = 18},
        Rogue   = {maxHP = 90,  attack = 22, defense = 6,  maxMP = 40,  speed = 20},
    }
    
    local base = baseStats[class] or baseStats.Warrior
    local growth = level - 1
    
    return {
        maxHP = base.maxHP + growth * 10,
        currentHP = base.maxHP + growth * 10,
        maxMP = base.maxMP + growth * 5,
        currentMP = base.maxMP + growth * 5,
        attack = base.attack + growth * 2,
        defense = base.defense + growth * 1,
        speed = base.speed,
        luck = 5 + growth
    }
end

return PlayerData
```

### 63.2.2 RPGCore Script

```lua
-- ServerScriptService/RPGCore.lua
-- ระบบหลักของ RPG

local Players = game:GetService("Players")
local DataStoreService = game:GetService("DataStoreService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- Modules
local PlayerDataModule = require(ReplicatedStorage.Modules.PlayerData)

-- DataStore
local rpgDataStore = DataStoreService:GetDataStore("RPGData_v2")

-- ข้อมูลผู้เล่นในหน่วยความจำ
local playerDataCache = {}

-- โหลดข้อมูลผู้เล่น
local function loadData(player)
    local success, data = pcall(function()
        return rpgDataStore:GetAsync("rpg_" .. player.UserId)
    end)
    
    if success and data then
        -- ตรวจสอบและเติม fields ที่หายไป
        local defaultData = PlayerDataModule.getDefault()
        for key, value in pairs(defaultData) do
            if data[key] == nil then
                data[key] = value
            end
        end
        playerDataCache[player.UserId] = data
    else
        playerDataCache[player.UserId] = PlayerDataModule.getDefault()
    end
    
    setupPlayerValues(player)
    print(player.Name .. " โหลดข้อมูล RPG แล้ว - Level " .. playerDataCache[player.UserId].level)
end

-- ตั้งค่า Values สำหรับ Player
function setupPlayerValues(player)
    local data = playerDataCache[player.UserId]
    if not data then return end
    
    -- Leaderstats
    local leaderstats = player:FindFirstChild("leaderstats") or Instance.new("Folder")
    leaderstats.Name = "leaderstats"
    leaderstats.Parent = player
    
    local function createOrUpdateValue(parent, name, valueType, value)
        local v = parent:FindFirstChild(name)
        if not v then
            v = Instance.new(valueType)
            v.Name = name
            v.Parent = parent
        end
        v.Value = value
        return v
    end
    
    createOrUpdateValue(leaderstats, "Level", "IntValue", data.level)
    createOrUpdateValue(leaderstats, "Gold", "IntValue", data.gold)
    createOrUpdateValue(leaderstats, "Class", "StringValue", data.class)
    
    -- PlayerStats (สำหรับ Local access)
    local playerStats = player:FindFirstChild("PlayerStats") or Instance.new("Folder")
    playerStats.Name = "PlayerStats"
    playerStats.Parent = player
    
    createOrUpdateValue(playerStats, "HP", "IntValue", data.stats.currentHP)
    createOrUpdateValue(playerStats, "MaxHP", "IntValue", data.stats.maxHP)
    createOrUpdateValue(playerStats, "MP", "IntValue", data.stats.currentMP)
    createOrUpdateValue(playerStats, "MaxMP", "IntValue", data.stats.maxMP)
    createOrUpdateValue(playerStats, "Attack", "IntValue", data.stats.attack)
    createOrUpdateValue(playerStats, "Defense", "IntValue", data.stats.defense)
    createOrUpdateValue(playerStats, "EXP", "IntValue", data.exp)
    createOrUpdateValue(playerStats, "ExpToNext", "IntValue", data.expToNext)
end

-- บันทึกข้อมูล
local function saveData(player)
    local data = playerDataCache[player.UserId]
    if not data then return end
    
    -- อัพเดทค่าจาก Values
    local playerStats = player:FindFirstChild("PlayerStats")
    if playerStats then
        data.stats.currentHP = playerStats:FindFirstChild("HP") and playerStats.HP.Value or data.stats.currentHP
        data.stats.currentMP = playerStats:FindFirstChild("MP") and playerStats.MP.Value or data.stats.currentMP
    end
    
    local success, err = pcall(function()
        rpgDataStore:SetAsync("rpg_" .. player.UserId, data)
    end)
    
    if success then
        print(player.Name .. " บันทึกข้อมูลแล้ว")
    else
        warn("บันทึกล้มเหลว: " .. err)
    end
end

-- ระบบ Level Up
local function addEXP(player, amount)
    local data = playerDataCache[player.UserId]
    if not data then return end
    
    data.exp = data.exp + amount
    
    -- ตรวจสอบ Level Up
    local leveled = false
    while data.exp >= data.expToNext do
        data.exp = data.exp - data.expToNext
        data.level = data.level + 1
        data.expToNext = PlayerDataModule.calculateExpNeeded(data.level)
        data.statPoints = data.statPoints + 5  -- ได้ 5 Stat Points ต่อ level
        
        -- คำนวณ Stats ใหม่
        local newStats = PlayerDataModule.calculateStats(data.level, data.class)
        data.stats.maxHP = newStats.maxHP
        data.stats.currentHP = newStats.currentHP  -- ฟื้นฟู HP เต็ม
        data.stats.maxMP = newStats.maxMP
        data.stats.currentMP = newStats.currentMP
        data.stats.attack = newStats.attack + (data.equippedBonus and data.equippedBonus.attack or 0)
        data.stats.defense = newStats.defense + (data.equippedBonus and data.equippedBonus.defense or 0)
        
        leveled = true
        print(player.Name .. " Level Up! ขึ้นเป็น Level " .. data.level)
    end
    
    -- อัพเดท Values
    local playerStats = player:FindFirstChild("PlayerStats")
    if playerStats then
        if playerStats:FindFirstChild("EXP") then
            playerStats.EXP.Value = data.exp
        end
        if playerStats:FindFirstChild("ExpToNext") then
            playerStats.ExpToNext.Value = data.expToNext
        end
        if playerStats:FindFirstChild("HP") then
            playerStats.HP.Value = data.stats.currentHP
        end
        if playerStats:FindFirstChild("MaxHP") then
            playerStats.MaxHP.Value = data.stats.maxHP
        end
    end
    
    local leaderstats = player:FindFirstChild("leaderstats")
    if leaderstats and leaderstats:FindFirstChild("Level") then
        leaderstats.Level.Value = data.level
    end
    
    -- แจ้ง Client ถ้า Level Up
    if leveled then
        local levelUpEvent = ReplicatedStorage.Remotes:FindFirstChild("LevelUp")
        if levelUpEvent then
            levelUpEvent:FireClient(player, data.level, data.statPoints)
        end
    end
end

-- ระบบ Damage
local function takeDamage(player, amount)
    local data = playerDataCache[player.UserId]
    if not data then return end
    
    -- คำนวณ damage หลัง defense
    local actualDamage = math.max(1, amount - data.stats.defense)
    data.stats.currentHP = math.max(0, data.stats.currentHP - actualDamage)
    
    -- อัพเดท Value
    local playerStats = player:FindFirstChild("PlayerStats")
    if playerStats and playerStats:FindFirstChild("HP") then
        playerStats.HP.Value = data.stats.currentHP
    end
    
    -- ตรวจสอบตาย
    if data.stats.currentHP <= 0 then
        local character = player.Character
        if character then
            local humanoid = character:FindFirstChildWhichIsA("Humanoid")
            if humanoid then
                humanoid.Health = 0
            end
        end
        data.deaths = data.deaths + 1
    end
    
    return actualDamage
end

-- Event Handlers
Players.PlayerAdded:Connect(function(player)
    loadData(player)
    
    player.CharacterAdded:Connect(function(character)
        local humanoid = character:WaitForChild("Humanoid")
        
        -- ตั้ง MaxHealth จาก Stats
        local data = playerDataCache[player.UserId]
        if data then
            humanoid.MaxHealth = data.stats.maxHP
            humanoid.Health = data.stats.currentHP
            humanoid.WalkSpeed = data.stats.speed
        end
        
        -- ตรวจสอบเมื่อ HP เปลี่ยน
        humanoid.HealthChanged:Connect(function(hp)
            local d = playerDataCache[player.UserId]
            if d then
                d.stats.currentHP = math.floor(hp)
                local playerStats = player:FindFirstChild("PlayerStats")
                if playerStats and playerStats:FindFirstChild("HP") then
                    playerStats.HP.Value = math.floor(hp)
                end
            end
        end)
    end)
end)

Players.PlayerRemoving:Connect(function(player)
    saveData(player)
    playerDataCache[player.UserId] = nil
end)

game:BindToClose(function()
    for _, player in ipairs(Players:GetPlayers()) do
        saveData(player)
    end
end)

-- Public API
local RPGCore = {
    addEXP = addEXP,
    takeDamage = takeDamage,
    getData = function(player) return playerDataCache[player.UserId] end,
}

return RPGCore
```

---

## 63.3 ระบบ Combat

```lua
-- ServerScriptService/CombatSystem.lua
-- ระบบต่อสู้ของ RPG

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")

-- สร้าง Remotes
local Remotes = ReplicatedStorage:WaitForChild("Remotes")

local attackRemote = Instance.new("RemoteEvent")
attackRemote.Name = "PlayerAttack"
attackRemote.Parent = Remotes

local damageEvent = Instance.new("RemoteEvent")
damageEvent.Name = "ShowDamage"
damageEvent.Parent = Remotes

-- ข้อมูล Cooldown
local attackCooldowns = {}

-- แสดง Damage Number
local function showDamageNumber(position, damage, isPlayer)
    for _, player in ipairs(Players:GetPlayers()) do
        local dist = (player.Character and player.Character:FindFirstChild("HumanoidRootPart"))
            and (player.Character.HumanoidRootPart.Position - position).Magnitude or 999
        
        if dist < 100 then  -- แสดงในระยะ 100 studs
            damageEvent:FireClient(player, position, damage, isPlayer)
        end
    end
end

-- ระบบโจมตี
attackRemote.OnServerEvent:Connect(function(player, targetId)
    -- ตรวจสอบ Cooldown
    local now = tick()
    if attackCooldowns[player.UserId] and now < attackCooldowns[player.UserId] then
        return  -- ยังอยู่ใน cooldown
    end
    
    -- ดึงข้อมูลผู้เล่น
    local RPGCore = require(script.Parent.RPGCore)
    local playerData = RPGCore.getData(player)
    if not playerData then return end
    
    -- หา target
    local target
    if targetId then
        target = workspace:FindFirstChild("Enemies"):FindFirstChild(tostring(targetId))
    end
    
    if not target then
        -- หา target ที่ใกล้ที่สุด
        local character = player.Character
        if not character or not character:FindFirstChild("HumanoidRootPart") then return end
        
        local rootPos = character.HumanoidRootPart.Position
        local nearestDist = 15  -- ระยะโจมตี
        
        for _, enemy in ipairs(workspace.Enemies:GetChildren()) do
            if enemy:FindFirstChild("HumanoidRootPart") then
                local dist = (enemy.HumanoidRootPart.Position - rootPos).Magnitude
                if dist < nearestDist then
                    nearestDist = dist
                    target = enemy
                end
            end
        end
    end
    
    if not target then return end
    
    -- คำนวณ Damage
    local baseDamage = playerData.stats.attack
    local critChance = playerData.stats.luck / 100
    local isCrit = math.random() < critChance
    
    local damage = baseDamage
    if isCrit then
        damage = damage * 2
    end
    
    -- เพิ่ม Random variance ±20%
    damage = math.floor(damage * (0.8 + math.random() * 0.4))
    
    -- ตั้ง Cooldown (0.5 วินาที)
    attackCooldowns[player.UserId] = now + 0.5
    
    -- ใส่ damage ให้ enemy
    local enemyHumanoid = target:FindFirstChildWhichIsA("Humanoid")
    if enemyHumanoid then
        enemyHumanoid:TakeDamage(damage)
        showDamageNumber(target.HumanoidRootPart.Position, damage, false)
        
        -- ตรวจสอบตาย
        if enemyHumanoid.Health <= 0 then
            local enemyData = target:FindFirstChild("EnemyData")
            if enemyData then
                local expReward = enemyData:GetAttribute("EXP") or 10
                local goldReward = enemyData:GetAttribute("Gold") or 5
                
                RPGCore.addEXP(player, expReward)
                
                local data = RPGCore.getData(player)
                if data then
                    data.gold = data.gold + goldReward
                    data.kills = data.kills + 1
                    
                    local leaderstats = player:FindFirstChild("leaderstats")
                    if leaderstats and leaderstats:FindFirstChild("Gold") then
                        leaderstats.Gold.Value = data.gold
                    end
                end
                
                print(player.Name .. " สังหาร " .. target.Name .. " ได้ " .. expReward .. " EXP, " .. goldReward .. " Gold")
            end
            
            -- ลบ enemy
            target:Destroy()
        end
    end
end)
```

---

## 63.4 ระบบ Inventory

```lua
-- ServerScriptService/InventorySystem.lua
-- ระบบกระเป๋าไอเท็ม

local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")

-- โหลด Item Database
local ItemDatabase = require(ReplicatedStorage.Modules.ItemDatabase)

local Remotes = ReplicatedStorage:WaitForChild("Remotes")

-- สร้าง Remotes
local equipItemRemote = Instance.new("RemoteEvent")
equipItemRemote.Name = "EquipItem"
equipItemRemote.Parent = Remotes

local unequipItemRemote = Instance.new("RemoteEvent")
unequipItemRemote.Name = "UnequipItem"
unequipItemRemote.Parent = Remotes

local updateInventoryEvent = Instance.new("RemoteEvent")
updateInventoryEvent.Name = "UpdateInventory"
updateInventoryEvent.Parent = Remotes

-- เพิ่มไอเท็มในกระเป๋า
local function addItem(player, itemId, quantity)
    local RPGCore = require(script.Parent.RPGCore)
    local data = RPGCore.getData(player)
    if not data then return false end
    
    quantity = quantity or 1
    local itemData = ItemDatabase.getItem(itemId)
    if not itemData then
        warn("ไม่พบไอเท็ม: " .. itemId)
        return false
    end
    
    -- ตรวจสอบว่ามีในกระเป๋าแล้ว
    for _, slot in ipairs(data.inventory) do
        if slot.itemId == itemId and itemData.stackable then
            slot.quantity = slot.quantity + quantity
            updateInventoryEvent:FireClient(player, data.inventory, data.equipped)
            return true
        end
    end
    
    -- ตรวจสอบพื้นที่
    if #data.inventory >= 30 then  -- max 30 slots
        local notifyEvent = Remotes:FindFirstChild("Notification")
        if notifyEvent then
            notifyEvent:FireClient(player, "กระเป๋าเต็ม!", "error")
        end
        return false
    end
    
    -- เพิ่ม slot ใหม่
    table.insert(data.inventory, {
        itemId = itemId,
        quantity = quantity,
        equipped = false
    })
    
    updateInventoryEvent:FireClient(player, data.inventory, data.equipped)
    print(player.Name .. " ได้รับ " .. itemData.name .. " x" .. quantity)
    return true
end

-- สวมไอเท็ม
local function equipItem(player, itemId)
    local RPGCore = require(script.Parent.RPGCore)
    local data = RPGCore.getData(player)
    if not data then return end
    
    local itemData = ItemDatabase.getItem(itemId)
    if not itemData then return end
    
    -- ตรวจสอบว่ามีในกระเป๋า
    local foundItem = false
    for _, slot in ipairs(data.inventory) do
        if slot.itemId == itemId then
            foundItem = true
            break
        end
    end
    
    if not foundItem then
        warn(player.Name .. " ไม่มีไอเท็ม " .. itemId .. " ในกระเป๋า")
        return
    end
    
    -- ตรวจสอบ Level requirement
    if itemData.levelReq and data.level < itemData.levelReq then
        local notifyEvent = Remotes:FindFirstChild("Notification")
        if notifyEvent then
            notifyEvent:FireClient(player, "ต้องการ Level " .. itemData.levelReq, "error")
        end
        return
    end
    
    -- ถอดไอเท็มเก่า (ถ้ามี)
    local slot = itemData.slot  -- "weapon", "armor", etc.
    local oldItemId = data.equipped[slot]
    if oldItemId then
        -- ลบ bonus จากไอเท็มเก่า
        local oldItem = ItemDatabase.getItem(oldItemId)
        if oldItem and oldItem.bonus then
            for stat, value in pairs(oldItem.bonus) do
                data.stats[stat] = data.stats[stat] - value
            end
        end
    end
    
    -- สวมไอเท็มใหม่
    data.equipped[slot] = itemId
    
    -- เพิ่ม bonus
    if itemData.bonus then
        for stat, value in pairs(itemData.bonus) do
            data.stats[stat] = data.stats[stat] + value
        end
    end
    
    -- อัพเดท Character
    local character = player.Character
    if character then
        local humanoid = character:FindFirstChildWhichIsA("Humanoid")
        if humanoid then
            humanoid.MaxHealth = data.stats.maxHP
            humanoid.WalkSpeed = data.stats.speed
        end
    end
    
    updateInventoryEvent:FireClient(player, data.inventory, data.equipped)
    print(player.Name .. " สวม " .. itemData.name)
end

-- Remote handlers
equipItemRemote.OnServerEvent:Connect(equipItem)

unequipItemRemote.OnServerEvent:Connect(function(player, slot)
    local RPGCore = require(script.Parent.RPGCore)
    local data = RPGCore.getData(player)
    if not data then return end
    
    local itemId = data.equipped[slot]
    if not itemId then return end
    
    -- ลบ bonus
    local ItemDatabase = require(ReplicatedStorage.Modules.ItemDatabase)
    local itemData = ItemDatabase.getItem(itemId)
    if itemData and itemData.bonus then
        for stat, value in pairs(itemData.bonus) do
            data.stats[stat] = data.stats[stat] - value
        end
    end
    
    data.equipped[slot] = nil
    updateInventoryEvent:FireClient(player, data.inventory, data.equipped)
end)

-- Public API
return {
    addItem = addItem,
    equipItem = equipItem
}
```

---

## 63.5 Item Database

```lua
-- ReplicatedStorage/Modules/ItemDatabase.lua
-- ฐานข้อมูลไอเท็มทั้งหมด

local ItemDatabase = {}

local items = {
    -- ดาบ (Weapons)
    ["iron_sword"] = {
        id = "iron_sword",
        name = "ดาบเหล็ก",
        description = "ดาบธรรมดาทำจากเหล็ก",
        type = "weapon",
        slot = "weapon",
        icon = "rbxassetid://6031280882",
        value = 100,
        levelReq = 1,
        stackable = false,
        bonus = {attack = 10},
        rarity = "Common"
    },
    ["steel_sword"] = {
        id = "steel_sword",
        name = "ดาบเหล็กกล้า",
        description = "ดาบที่แข็งแกร่งกว่าดาบธรรมดา",
        type = "weapon",
        slot = "weapon",
        icon = "rbxassetid://6031280882",
        value = 500,
        levelReq = 5,
        stackable = false,
        bonus = {attack = 25},
        rarity = "Uncommon"
    },
    ["fire_sword"] = {
        id = "fire_sword",
        name = "ดาบไฟ",
        description = "ดาบที่มีเปลวไฟลุกอยู่ตลอดเวลา",
        type = "weapon",
        slot = "weapon",
        icon = "rbxassetid://6031280882",
        value = 2000,
        levelReq = 10,
        stackable = false,
        bonus = {attack = 50},
        special = "burn",  -- ทำ burn damage
        rarity = "Rare"
    },
    
    -- เกราะ (Armor)
    ["leather_armor"] = {
        id = "leather_armor",
        name = "เกราะหนัง",
        description = "เกราะพื้นฐานทำจากหนัง",
        type = "armor",
        slot = "armor",
        icon = "rbxassetid://6031280882",
        value = 80,
        levelReq = 1,
        stackable = false,
        bonus = {defense = 5, maxHP = 20},
        rarity = "Common"
    },
    ["iron_armor"] = {
        id = "iron_armor",
        name = "เกราะเหล็ก",
        description = "เกราะเหล็กป้องกันได้ดี",
        type = "armor",
        slot = "armor",
        icon = "rbxassetid://6031280882",
        value = 400,
        levelReq = 5,
        stackable = false,
        bonus = {defense = 15, maxHP = 50},
        rarity = "Uncommon"
    },
    
    -- ยา (Consumables)
    ["health_potion"] = {
        id = "health_potion",
        name = "ยาฟื้นฟู HP",
        description = "ฟื้นฟู HP 50 หน่วย",
        type = "consumable",
        slot = nil,
        icon = "rbxassetid://6031280882",
        value = 25,
        levelReq = 1,
        stackable = true,
        effect = {type = "heal", amount = 50},
        rarity = "Common"
    },
    ["mana_potion"] = {
        id = "mana_potion",
        name = "ยาฟื้นฟู MP",
        description = "ฟื้นฟู MP 30 หน่วย",
        type = "consumable",
        slot = nil,
        icon = "rbxassetid://6031280882",
        value = 30,
        levelReq = 1,
        stackable = true,
        effect = {type = "mana", amount = 30},
        rarity = "Common"
    },
}

function ItemDatabase.getItem(itemId)
    return items[itemId]
end

function ItemDatabase.getAllItems()
    return items
end

function ItemDatabase.getItemsByType(itemType)
    local result = {}
    for _, item in pairs(items) do
        if item.type == itemType then
            table.insert(result, item)
        end
    end
    return result
end

return ItemDatabase
```

---

## 63.6 ระบบ Quest

```lua
-- ReplicatedStorage/Modules/QuestDatabase.lua
-- ฐานข้อมูลเควส

local QuestDatabase = {}

local quests = {
    ["quest_001"] = {
        id = "quest_001",
        name = "ผู้เริ่มต้น",
        description = "กำจัดสัตว์ประหลาดสีเขียว 5 ตัวในป่าใกล้บ้าน",
        type = "kill",
        
        objectives = {
            {
                description = "สังหาร Slime",
                target = "Slime",
                required = 5,
                current = 0
            }
        },
        
        rewards = {
            exp = 100,
            gold = 50,
            items = {
                {itemId = "health_potion", quantity = 3}
            }
        },
        
        levelReq = 1,
        prereqs = {},  -- เควสที่ต้องทำก่อน
        
        startNPC = "Elder_Harold",
        turnInNPC = "Elder_Harold",
        
        dialogue = {
            start = "ขอบคุณที่มาช่วย ช่วยฉันกำจัด Slime ในป่าด้วยนะ",
            complete = "ขอบคุณมากเลย! นี่คือรางวัลของเจ้า"
        }
    },
    
    ["quest_002"] = {
        id = "quest_002",
        name = "นักสำรวจ",
        description = "ค้นหาหมู่บ้านแห่งใหม่ทางทิศตะวันออก",
        type = "explore",
        
        objectives = {
            {
                description = "ค้นพบหมู่บ้าน Thornwood",
                target = "Village_Thornwood",
                required = 1,
                current = 0
            }
        },
        
        rewards = {
            exp = 200,
            gold = 100,
            items = {}
        },
        
        levelReq = 3,
        prereqs = {"quest_001"},
        
        startNPC = "Elder_Harold",
        turnInNPC = "Village_Elder",
    },
}

function QuestDatabase.getQuest(questId)
    return quests[questId]
end

function QuestDatabase.getAvailableQuests(playerLevel, completedQuests)
    local available = {}
    
    for questId, quest in pairs(quests) do
        -- ตรวจสอบ level
        if playerLevel >= quest.levelReq then
            -- ตรวจสอบ prerequisites
            local prereqsMet = true
            for _, prereqId in ipairs(quest.prereqs) do
                if not completedQuests[prereqId] then
                    prereqsMet = false
                    break
                end
            end
            
            if prereqsMet and not completedQuests[questId] then
                table.insert(available, quest)
            end
        end
    end
    
    return available
end

return QuestDatabase
```

---

## 63.7 ระบบ NPC

```lua
-- ServerScriptService/NPCManager.lua
-- ระบบ NPC ใน RPG

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")

local QuestDatabase = require(ReplicatedStorage.Modules.QuestDatabase)

-- สร้าง NPC
local function createNPC(config)
    local npcModel = Instance.new("Model")
    npcModel.Name = config.name
    
    -- สร้างร่างกาย (simplified)
    local rootPart = Instance.new("Part")
    rootPart.Name = "HumanoidRootPart"
    rootPart.Size = Vector3.new(2, 2, 1)
    rootPart.Position = config.position
    rootPart.Anchored = true
    rootPart.Transparency = 1
    rootPart.Parent = npcModel
    npcModel.PrimaryPart = rootPart
    
    local torso = Instance.new("Part")
    torso.Name = "Torso"
    torso.Size = Vector3.new(2, 2, 1)
    torso.Position = config.position + Vector3.new(0, 0, 0)
    torso.Anchored = true
    torso.BrickColor = config.color or BrickColor.new("Medium stone grey")
    torso.Parent = npcModel
    
    local head = Instance.new("Part")
    head.Name = "Head"
    head.Size = Vector3.new(2, 2, 2)
    head.Position = config.position + Vector3.new(0, 2, 0)
    head.Anchored = true
    head.BrickColor = config.skinColor or BrickColor.new("Bright yellow")
    head.Parent = npcModel
    
    -- Humanoid
    local humanoid = Instance.new("Humanoid")
    humanoid.MaxHealth = 0
    humanoid.DisplayDistanceType = Enum.HumanoidDisplayDistanceType.Subject
    humanoid.HealthDisplayDistanceType = Enum.HumanoidHealthDisplayDistanceType.None
    humanoid.Parent = npcModel
    
    -- ชื่อ NPC
    local billboard = Instance.new("BillboardGui")
    billboard.Size = UDim2.new(0, 200, 0, 60)
    billboard.StudsOffset = Vector3.new(0, 3, 0)
    billboard.Parent = head
    
    local nameLabel = Instance.new("TextLabel")
    nameLabel.Size = UDim2.new(1, 0, 0.6, 0)
    nameLabel.BackgroundTransparency = 1
    nameLabel.Text = config.displayName or config.name
    nameLabel.TextColor3 = Color3.fromRGB(255, 255, 100)
    nameLabel.TextScaled = true
    nameLabel.Font = Enum.Font.GothamBold
    nameLabel.Parent = billboard
    
    local roleLabel = Instance.new("TextLabel")
    roleLabel.Size = UDim2.new(1, 0, 0.4, 0)
    roleLabel.Position = UDim2.new(0, 0, 0.6, 0)
    roleLabel.BackgroundTransparency = 1
    roleLabel.Text = config.role or "NPC"
    roleLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
    roleLabel.TextScaled = true
    roleLabel.Font = Enum.Font.Gotham
    roleLabel.Parent = billboard
    
    -- เพิ่ม Attributes
    npcModel:SetAttribute("NPCType", config.type or "questgiver")
    npcModel:SetAttribute("Quests", table.concat(config.quests or {}, ","))
    npcModel:SetAttribute("ShopId", config.shopId or "")
    npcModel:SetAttribute("Dialogue", config.dialogue or "สวัสดี นักเดินทาง!")
    
    npcModel.Parent = workspace.NPCs or workspace
    
    -- หมุน NPC ไปหาผู้เล่นที่ใกล้ที่สุด
    RunService.Heartbeat:Connect(function()
        local nearestPlayer = nil
        local nearestDist = 20
        
        for _, player in ipairs(Players:GetPlayers()) do
            if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                local dist = (player.Character.HumanoidRootPart.Position - rootPart.Position).Magnitude
                if dist < nearestDist then
                    nearestDist = dist
                    nearestPlayer = player
                end
            end
        end
        
        if nearestPlayer and nearestPlayer.Character then
            local lookDir = (nearestPlayer.Character.HumanoidRootPart.Position - rootPart.Position)
            lookDir = Vector3.new(lookDir.X, 0, lookDir.Z).Unit
            
            local newCFrame = CFrame.new(rootPart.Position, rootPart.Position + lookDir)
            rootPart.CFrame = newCFrame
            torso.CFrame = newCFrame
            head.CFrame = newCFrame + Vector3.new(0, 2, 0)
        end
    end)
    
    print("สร้าง NPC " .. config.name .. " แล้ว")
    return npcModel
end

-- สร้าง NPCs ในเกม
local function setupNPCs()
    local npcFolder = Instance.new("Folder")
    npcFolder.Name = "NPCs"
    npcFolder.Parent = workspace
    
    createNPC({
        name = "Elder_Harold",
        displayName = "Elder Harold",
        role = "Quest Giver",
        position = Vector3.new(0, 5, 10),
        type = "questgiver",
        quests = {"quest_001", "quest_002"},
        dialogue = "สวัสดี นักผจญภัยหนุ่ม! ฉันมีงานให้เจ้าทำ",
        color = BrickColor.new("Reddish brown"),
        skinColor = BrickColor.new("Bright yellow")
    })
    
    createNPC({
        name = "Merchant_Tom",
        displayName = "Tom the Merchant",
        role = "Shop",
        position = Vector3.new(20, 5, 0),
        type = "merchant",
        shopId = "general_store",
        dialogue = "มาดูสินค้าของฉันสิ! ราคาพิเศษ!",
        color = BrickColor.new("Medium blue"),
    })
end

setupNPCs()
```

---

## 63.8 ข้อผิดพลาดที่พบบ่อย

### ข้อผิดพลาด 1: ข้อมูลหาย

```lua
-- ❌ ผิด: บันทึกข้อมูลทันทีทุกครั้งที่มีการเปลี่ยนแปลง
-- จะทำให้ถึง Rate Limit ของ DataStore

-- ✓ ถูก: ใช้ระบบ Queue บันทึกข้อมูล
local saveQueue = {}
local SAVE_INTERVAL = 60  -- บันทึกทุก 60 วินาที

local function queueSave(userId)
    saveQueue[userId] = tick()  -- mark ว่าต้องบันทึก
end

-- Background save loop
task.spawn(function()
    while true do
        task.wait(SAVE_INTERVAL)
        for userId, _ in pairs(saveQueue) do
            local player = Players:GetPlayerByUserId(userId)
            if player then
                saveData(player)
            end
            saveQueue[userId] = nil
        end
    end
end)
```

---

## 63.9 แบบฝึกหัด

### แบบฝึกหัดที่ 1: สร้าง Class ใหม่
เพิ่ม Class "Healer" ที่มี stats:
- HP ต่ำกว่า Warrior
- MP สูงมาก
- มี Skill ฟื้นฟู HP

### แบบฝึกหัดที่ 2: สร้าง Skill System
สร้างระบบ Skill ที่:
- แต่ละ Skill ใช้ MP
- มี Cooldown
- มีผลต่างกัน (โจมตี, บาดเจ็บ, ฟื้นฟู)

### แบบฝึกหัดที่ 3: สร้าง World Boss
สร้าง Boss ที่:
- HP สูงมาก (10,000+)
- มีท่าโจมตีหลายแบบ
- ต้องการผู้เล่นหลายคนช่วยกันฆ่า
- ให้รางวัลพิเศษ

---

## สรุป

ในบทนี้เราได้สร้าง RPG Framework ที่ครอบคลุม:
- ระบบ Stats และ Level Up
- ระบบ Combat
- ระบบ Inventory
- ระบบ Quest
- ระบบ NPC

ในบทถัดไปเราจะสร้าง Simulator Game!
