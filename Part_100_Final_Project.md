# Part 100: Final Project - สร้างเกม RPG สมบูรณ์แบบ

## บทนำ (Introduction)

🎊 ยินดีด้วย! คุณมาถึงบทสุดท้ายของ course นี้แล้ว!
ในบทนี้เราจะนำทุกอย่างที่เรียนมาตลอด 99 บทมารวมกัน
โดยสร้างเกม **"Legends of Roblox"** — เกม RPG สมบูรณ์แบบตั้งแต่เริ่มต้น

### เป้าหมายของ Final Project:
- สร้างเกม RPG ที่มีระบบครบถ้วนและ playable จริงๆ
- รวมทุกทักษะที่เรียนมา: DataStore, Combat, AI, UI, Monetization
- เขียน code ที่ production-ready และ scalable
- เตรียม game สำหรับ launch จริง

### โครงสร้างเกม "Legends of Roblox":
```
เกม RPG Multiplayer ที่มี:
- Character system พร้อม stats และ leveling
- Combat system แบบ real-time
- Dungeon system ที่ procedurally generated
- Quest system ด้วย NPC dialog
- Inventory และ Equipment system
- Party system สำหรับ multiplayer
- Monetization ครบถ้วน (GamePass + Products)
- DataStore สำหรับ save progress
- Leaderboard
```

---

## Phase 1: Game Architecture

### 1.1 โครงสร้างโฟลเดอร์

```
game structure:
📁 ServerScriptService/
  📁 Modules/
    PlayerManager.lua
    DataManager.lua
    CombatManager.lua
    QuestManager.lua
    DungeonManager.lua
    LootManager.lua
    PartyManager.lua
    NPCManager.lua
    MonetizationManager.lua
    AntiCheatManager.lua
  📁 Services/
    GameService.lua
    PlayerService.lua
    CombatService.lua
  Main.server.lua

📁 ReplicatedStorage/
  📁 Modules/
    GameConfig.lua
    Types.lua
    Utils.lua
    Constants.lua
  📁 Events/
    CombatEvents.lua
    UIEvents.lua
    GameEvents.lua
  📁 Assets/
    📁 Animations/
    📁 VFX/
    📁 Items/

📁 StarterPlayer/
  📁 StarterPlayerScripts/
    ClientMain.client.lua
    📁 Controllers/
      CombatController.lua
      UIController.lua
      InputController.lua
    📁 UI/
      HUDManager.lua
      InventoryUI.lua
      ShopUI.lua
      QuestUI.lua

📁 StarterGui/
  MainHUD.rbxmx
  InventoryGui.rbxmx
  QuestGui.rbxmx
  ShopGui.rbxmx
```

### 1.2 GameConfig Module

```lua
-- ModuleScript: GameConfig
-- การตั้งค่าหลักของเกมทั้งหมด
-- All main game configurations

local GameConfig = {}

-- ==================== GAME INFO ====================
GameConfig.GAME_NAME = "Legends of Roblox"
GameConfig.VERSION = "1.0.0"
GameConfig.MAX_PLAYERS = 20

-- ==================== PLAYER STATS ====================
GameConfig.DEFAULT_PLAYER_DATA = {
    -- Character Stats
    Level = 1,
    Experience = 0,
    Gold = 100,
    Gems = 0,
    
    -- Combat Stats
    MaxHealth = 100,
    MaxMana = 50,
    Attack = 10,
    Defense = 5,
    Speed = 16,
    
    -- Progress
    QuestsCompleted = {},
    AchievementsUnlocked = {},
    DungeonsCleared = {},
    
    -- Inventory
    Inventory = {},
    Equipment = {
        Weapon = nil,
        Armor = nil,
        Helm = nil,
        Accessory = nil,
    },
    
    -- Social
    PartyInvites = {},
    FriendList = {},
    
    -- Settings
    MusicEnabled = true,
    SFXEnabled = true,
    QualityLevel = "Medium",
    
    -- Timestamps
    FirstJoin = 0,
    LastLogin = 0,
    TotalPlayTime = 0,
}

-- ==================== LEVEL SYSTEM ====================
GameConfig.MAX_LEVEL = 100

-- XP required per level (ใช้ formula แทน table ใหญ่)
function GameConfig.GetXPForLevel(level)
    return math.floor(100 * (level ^ 1.5))
end

-- Stat growth per level
GameConfig.STAT_GROWTH = {
    MaxHealth = 15,    -- +15 HP per level
    MaxMana = 8,       -- +8 MP per level
    Attack = 2,        -- +2 ATK per level
    Defense = 1,       -- +1 DEF per level
}

-- ==================== CLASSES ====================
GameConfig.CLASSES = {
    Warrior = {
        Name = "Warrior",
        Description = "นักรบผู้แข็งแกร่ง ทนทาน และพลังโจมตีสูง",
        StatMultipliers = {
            MaxHealth = 1.3,
            MaxMana = 0.7,
            Attack = 1.2,
            Defense = 1.3,
        },
        StartingWeapon = "Iron_Sword",
        Color = Color3.fromRGB(200, 100, 50),
        Icon = "rbxassetid://0",
        Abilities = {"PowerStrike", "Shield", "WarCry"},
    },
    
    Mage = {
        Name = "Mage",
        Description = "นักเวทย์ผู้เชี่ยวชาญด้านเวทมนตร์ พลังโจมตีสูงมาก",
        StatMultipliers = {
            MaxHealth = 0.7,
            MaxMana = 1.5,
            Attack = 1.5,
            Defense = 0.7,
        },
        StartingWeapon = "Oak_Staff",
        Color = Color3.fromRGB(80, 120, 220),
        Icon = "rbxassetid://0",
        Abilities = {"Fireball", "IceSpike", "Teleport"},
    },
    
    Archer = {
        Name = "Archer",
        Description = "นักธนูที่เร็วและคล่องแคล่ว โจมตีจากระยะไกล",
        StatMultipliers = {
            MaxHealth = 0.9,
            MaxMana = 1.0,
            Attack = 1.1,
            Defense = 0.9,
            Speed = 1.2,
        },
        StartingWeapon = "Wooden_Bow",
        Color = Color3.fromRGB(80, 180, 80),
        Icon = "rbxassetid://0",
        Abilities = {"Arrow_Rain", "Dodge", "Eagle_Eye"},
    },
    
    Healer = {
        Name = "Healer",
        Description = "นักรักษาผู้ช่วยเหลือเพื่อนร่วมทีม",
        StatMultipliers = {
            MaxHealth = 1.0,
            MaxMana = 1.3,
            Attack = 0.7,
            Defense = 1.0,
        },
        StartingWeapon = "Holy_Rod",
        Color = Color3.fromRGB(220, 200, 80),
        Icon = "rbxassetid://0",
        Abilities = {"Heal", "Mass_Heal", "Resurrection"},
    },
}

-- ==================== ABILITIES ====================
GameConfig.ABILITIES = {
    PowerStrike = {
        Name = "Power Strike",
        Description = "โจมตีหนักด้วยพลัง 200% ของ ATK ปกติ",
        ManaCost = 15,
        Cooldown = 5,
        Range = 8,
        DamageMultiplier = 2.0,
        Type = "Melee",
        VFX = "PowerStrike_VFX",
        Animation = "rbxassetid://0",
    },
    
    Shield = {
        Name = "Shield",
        Description = "สร้างกำแพงป้องกัน ลด damage 50% เป็นเวลา 5 วินาที",
        ManaCost = 20,
        Cooldown = 15,
        Duration = 5,
        DefenseBonus = 0.5,
        Type = "Buff",
        VFX = "Shield_VFX",
    },
    
    WarCry = {
        Name = "War Cry",
        Description = "เพิ่ม ATK ของตัวเองและพันธมิตรในรัศมี 15 studs 30% เป็นเวลา 10 วินาที",
        ManaCost = 25,
        Cooldown = 20,
        Radius = 15,
        AttackBonus = 0.3,
        Duration = 10,
        Type = "AOE_Buff",
    },
    
    Fireball = {
        Name = "Fireball",
        Description = "ยิงลูกไฟทำ damage กับพื้นที่",
        ManaCost = 30,
        Cooldown = 3,
        Range = 50,
        AOERadius = 8,
        DamageMultiplier = 2.5,
        Type = "AOE",
        VFX = "Fireball_VFX",
        ProjectileSpeed = 60,
    },
    
    Heal = {
        Name = "Heal",
        Description = "รักษา HP ของตัวเองหรือพันธมิตร 50 HP",
        ManaCost = 20,
        Cooldown = 3,
        HealAmount = 50,
        Range = 20,
        Type = "Heal",
        VFX = "Heal_VFX",
    },
}

-- ==================== ITEMS ====================
GameConfig.ITEM_QUALITY = {
    Common = {color = Color3.fromRGB(200, 200, 200), multiplier = 1.0},
    Uncommon = {color = Color3.fromRGB(80, 200, 80), multiplier = 1.2},
    Rare = {color = Color3.fromRGB(80, 100, 220), multiplier = 1.5},
    Epic = {color = Color3.fromRGB(160, 60, 220), multiplier = 2.0},
    Legendary = {color = Color3.fromRGB(255, 165, 0), multiplier = 3.0},
}

GameConfig.ITEMS = {
    -- Weapons
    Iron_Sword = {
        Name = "Iron Sword",
        Description = "ดาบเหล็กธรรมดา",
        Type = "Weapon",
        Slot = "Weapon",
        Quality = "Common",
        Stats = {Attack = 8},
        Value = 50,
        Icon = "rbxassetid://0",
    },
    
    Oak_Staff = {
        Name = "Oak Staff",
        Description = "ไม้เท้าไม้โอ๊ค เพิ่มพลังเวทย์",
        Type = "Weapon",
        Slot = "Weapon",
        Quality = "Common",
        Stats = {Attack = 12, MaxMana = 20},
        Value = 80,
        Icon = "rbxassetid://0",
    },
    
    Wooden_Bow = {
        Name = "Wooden Bow",
        Description = "ธนูไม้สำหรับมือใหม่",
        Type = "Weapon",
        Slot = "Weapon",
        Quality = "Common",
        Stats = {Attack = 10, Speed = 2},
        Value = 60,
        Icon = "rbxassetid://0",
    },
    
    -- Armor
    Leather_Armor = {
        Name = "Leather Armor",
        Description = "เกราะหนังเบาๆ",
        Type = "Armor",
        Slot = "Armor",
        Quality = "Common",
        Stats = {Defense = 5, MaxHealth = 20},
        Value = 70,
        Icon = "rbxassetid://0",
    },
    
    -- Consumables
    Health_Potion = {
        Name = "Health Potion",
        Description = "ยาฟื้นฟู HP 50 จุด",
        Type = "Consumable",
        Quality = "Common",
        HealAmount = 50,
        Stackable = true,
        MaxStack = 99,
        Value = 25,
        Icon = "rbxassetid://0",
    },
    
    Mana_Potion = {
        Name = "Mana Potion",
        Description = "ยาฟื้นฟู MP 30 จุด",
        Type = "Consumable",
        Quality = "Common",
        ManaAmount = 30,
        Stackable = true,
        MaxStack = 99,
        Value = 20,
        Icon = "rbxassetid://0",
    },
}

-- ==================== DUNGEONS ====================
GameConfig.DUNGEONS = {
    {
        id = "forest_dungeon",
        Name = "Enchanted Forest",
        Description = "ป่าที่เต็มไปด้วยอสูรต้นไม้",
        MinLevel = 1,
        MaxLevel = 10,
        Floors = 3,
        Difficulty = "Easy",
        Rewards = {
            Gold = {min = 100, max = 300},
            Exp = {min = 200, max = 500},
            LootTable = "forest",
        },
        BossName = "Ancient Tree Spirit",
        Icon = "rbxassetid://0",
    },
    {
        id = "cave_dungeon",
        Name = "Crystal Cave",
        Description = "ถ้ำที่เต็มไปด้วยคริสตัลและสัตว์ประหลาด",
        MinLevel = 10,
        MaxLevel = 25,
        Floors = 5,
        Difficulty = "Medium",
        Rewards = {
            Gold = {min = 300, max = 800},
            Exp = {min = 500, max = 1200},
            LootTable = "cave",
        },
        BossName = "Crystal Golem",
        Icon = "rbxassetid://0",
    },
    {
        id = "volcano_dungeon",
        Name = "Volcano of Doom",
        Description = "ภูเขาไฟที่มีปีศาจไฟอาศัยอยู่",
        MinLevel = 25,
        MaxLevel = 50,
        Floors = 7,
        Difficulty = "Hard",
        Rewards = {
            Gold = {min = 800, max = 2000},
            Exp = {min = 1500, max = 3500},
            LootTable = "volcano",
        },
        BossName = "Infernal Drake",
        Icon = "rbxassetid://0",
    },
}

-- ==================== QUESTS ====================
GameConfig.QUESTS = {
    {
        id = "q001",
        Name = "First Steps",
        Description = "เอาชนะ Slime 5 ตัวเพื่อพิสูจน์ความสามารถ",
        Type = "Kill",
        Objective = {
            Type = "Kill",
            Target = "Slime",
            Required = 5,
        },
        Rewards = {
            Gold = 50,
            Experience = 100,
            Items = {"Health_Potion"},
        },
        Level = 1,
        NPCGiver = "Elder_Marcus",
    },
    {
        id = "q002",
        Name = "Heal the Wounded",
        Description = "เก็บ Healing Herbs 10 ชิ้น เพื่อช่วยเหลือชาวบ้าน",
        Type = "Collect",
        Objective = {
            Type = "Collect",
            Target = "Healing_Herb",
            Required = 10,
        },
        Rewards = {
            Gold = 75,
            Experience = 150,
            Items = {"Mana_Potion", "Mana_Potion"},
        },
        Level = 1,
        NPCGiver = "Healer_Maria",
    },
}

-- ==================== MONETIZATION ====================
GameConfig.GAMEPASSES = {
    VIP = {
        id = 123456,  -- Replace with real ID
        Name = "VIP Pass",
        Price = 199,  -- Robux
        Benefits = {
            "Gold boost 2x",
            "XP boost 1.5x",
            "Exclusive VIP title",
            "VIP only area access",
            "Daily free gems",
        },
    },
    ExtraSlots = {
        id = 234567,
        Name = "Extra Inventory",
        Price = 99,
        Benefits = {
            "50 additional inventory slots",
        },
    },
    DoubleXP = {
        id = 345678,
        Name = "Double XP",
        Price = 149,
        Benefits = {
            "2x experience from all sources",
        },
    },
}

GameConfig.DEV_PRODUCTS = {
    Gold_1000 = {
        id = 456789,
        Name = "1,000 Gold",
        Price = 25,
        Gold = 1000,
    },
    Gold_5000 = {
        id = 567890,
        Name = "5,000 Gold",
        Price = 100,
        Gold = 5500,  -- bonus 500
    },
    Gems_100 = {
        id = 678901,
        Name = "100 Gems",
        Price = 49,
        Gems = 100,
    },
}

return GameConfig
```

---

## Phase 2: Core Systems

### 2.1 PlayerManager

```lua
-- ModuleScript: PlayerManager
-- จัดการข้อมูลและสถานะของ player ทุกคน
-- Manages all player data and state

local PlayerManager = {}
PlayerManager.__index = PlayerManager

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")

local GameConfig = require(ReplicatedStorage.Modules.GameConfig)
local DataManager = require(script.Parent.DataManager)

local activePlayers = {}  -- {userId -> playerObject}

-- สร้าง player object (Create player object)
local function createPlayerObject(player, data)
    local obj = {
        Player = player,
        UserId = player.UserId,
        Name = player.Name,
        Data = data,
        Character = player.Character,
        
        -- Runtime state (ไม่ได้ save)
        CurrentHealth = data.MaxHealth,
        CurrentMana = data.MaxMana,
        IsInCombat = false,
        IsInDungeon = false,
        DungeonId = nil,
        PartyId = nil,
        QuestProgress = {},  -- {questId -> progress}
        ActiveBuffs = {},
        
        -- Connections
        _connections = {},
    }
    
    return obj
end

-- เมื่อ player เข้าเกม (When player joins)
function PlayerManager.OnPlayerAdded(player)
    -- โหลดข้อมูล
    local data = DataManager:LoadPlayer(player)
    
    if not data then
        -- ข้อมูลใหม่ (New player)
        data = table.clone(GameConfig.DEFAULT_PLAYER_DATA)
        data.FirstJoin = os.time()
        data.Level = 1
    end
    
    data.LastLogin = os.time()
    
    -- สร้าง player object
    local playerObj = createPlayerObject(player, data)
    activePlayers[player.UserId] = playerObj
    
    -- ตั้งค่า Character เมื่อ spawn
    player.CharacterAdded:Connect(function(character)
        playerObj.Character = character
        PlayerManager.OnCharacterSpawned(player, character, playerObj)
    end)
    
    if player.Character then
        PlayerManager.OnCharacterSpawned(player, player.Character, playerObj)
    end
    
    -- Auto-save ทุก 60 วินาที
    local autoSaveConnection = RunService.Heartbeat:Connect(function()
        -- จะทำ timer check ที่นี่
    end)
    
    table.insert(playerObj._connections, autoSaveConnection)
    
    print(player.Name, "joined the game - Level", data.Level)
    
    return playerObj
end

-- เมื่อ Character spawn (When character spawns)
function PlayerManager.OnCharacterSpawned(player, character, playerObj)
    local humanoid = character:WaitForChild("Humanoid")
    
    -- Apply player stats
    PlayerManager.ApplyStats(playerObj)
    
    -- Reset health/mana ตอน respawn
    local data = playerObj.Data
    playerObj.CurrentHealth = data.MaxHealth
    playerObj.CurrentMana = data.MaxMana
    
    -- Update UI
    local updateEvent = ReplicatedStorage.Events:FindFirstChild("UpdateHUD")
    if updateEvent then
        updateEvent:FireClient(player, {
            health = playerObj.CurrentHealth,
            maxHealth = data.MaxHealth,
            mana = playerObj.CurrentMana,
            maxMana = data.MaxMana,
            level = data.Level,
            gold = data.Gold,
        })
    end
    
    -- Handle death
    humanoid.Died:Connect(function()
        PlayerManager.OnPlayerDied(playerObj)
    end)
end

-- Apply stats ไปยัง Humanoid (Apply stats to Humanoid)
function PlayerManager.ApplyStats(playerObj)
    local character = playerObj.Character
    if not character then return end
    
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end
    
    local data = playerObj.Data
    
    -- ตรวจสอบ Equipment bonus
    local totalMaxHealth = data.MaxHealth
    local totalAttack = data.Attack
    local totalDefense = data.Defense
    local totalSpeed = data.Speed
    
    for slot, itemId in pairs(data.Equipment) do
        if itemId then
            local item = GameConfig.ITEMS[itemId]
            if item and item.Stats then
                totalMaxHealth = totalMaxHealth + (item.Stats.MaxHealth or 0)
                totalAttack = totalAttack + (item.Stats.Attack or 0)
                totalDefense = totalDefense + (item.Stats.Defense or 0)
                totalSpeed = totalSpeed + (item.Stats.Speed or 0)
            end
        end
    end
    
    -- ตรวจสอบ Buff bonus
    for _, buff in ipairs(playerObj.ActiveBuffs) do
        if buff.AttackBonus then
            totalAttack = totalAttack * (1 + buff.AttackBonus)
        end
        if buff.DefenseBonus then
            totalDefense = totalDefense * (1 + buff.DefenseBonus)
        end
    end
    
    -- Apply ไปยัง Humanoid
    humanoid.MaxHealth = totalMaxHealth
    humanoid.WalkSpeed = totalSpeed
    
    -- บันทึก runtime stats
    playerObj.TotalAttack = totalAttack
    playerObj.TotalDefense = totalDefense
end

-- เมื่อ player ตาย (When player dies)
function PlayerManager.OnPlayerDied(playerObj)
    playerObj.IsInCombat = false
    playerObj.ActiveBuffs = {}
    
    -- ลด gold 10%
    local goldLoss = math.floor(playerObj.Data.Gold * 0.1)
    playerObj.Data.Gold = math.max(0, playerObj.Data.Gold - goldLoss)
    
    -- Respawn หลัง 5 วินาที
    local player = playerObj.Player
    task.delay(5, function()
        if player and player.Parent then
            player:LoadCharacter()
        end
    end)
    
    print(playerObj.Name, "died. Lost", goldLoss, "gold")
end

-- Level Up (Level up player)
function PlayerManager.CheckLevelUp(playerObj)
    local data = playerObj.Data
    local requiredXP = GameConfig.GetXPForLevel(data.Level)
    
    while data.Experience >= requiredXP and data.Level < GameConfig.MAX_LEVEL do
        data.Experience = data.Experience - requiredXP
        data.Level = data.Level + 1
        
        -- เพิ่ม stats ตาม growth rate
        for stat, growth in pairs(GameConfig.STAT_GROWTH) do
            data[stat] = (data[stat] or 0) + growth
        end
        
        -- รักษา HP/MP เต็ม
        playerObj.CurrentHealth = data.MaxHealth
        playerObj.CurrentMana = data.MaxMana
        
        -- Apply stats ใหม่
        PlayerManager.ApplyStats(playerObj)
        
        -- แจ้ง client
        local levelUpEvent = ReplicatedStorage.Events:FindFirstChild("LevelUp")
        if levelUpEvent then
            levelUpEvent:FireClient(playerObj.Player, {
                newLevel = data.Level,
                stats = {
                    maxHealth = data.MaxHealth,
                    maxMana = data.MaxMana,
                    attack = data.Attack,
                    defense = data.Defense,
                },
            })
        end
        
        print(playerObj.Name, "leveled up to", data.Level)
        
        requiredXP = GameConfig.GetXPForLevel(data.Level)
    end
end

-- เพิ่ม Experience (Add experience)
function PlayerManager.AddExperience(playerObj, amount)
    -- VIP bonus
    local bonusMultiplier = 1.0
    -- (check GamePass ที่นี่)
    
    playerObj.Data.Experience = playerObj.Data.Experience + math.floor(amount * bonusMultiplier)
    PlayerManager.CheckLevelUp(playerObj)
    
    -- Update UI
    local updateEvent = ReplicatedStorage.Events:FindFirstChild("UpdateXP")
    if updateEvent then
        updateEvent:FireClient(playerObj.Player, {
            exp = playerObj.Data.Experience,
            requiredExp = GameConfig.GetXPForLevel(playerObj.Data.Level),
        })
    end
end

-- เพิ่ม Gold (Add gold)
function PlayerManager.AddGold(playerObj, amount)
    -- VIP bonus
    local bonusMultiplier = 1.0
    
    playerObj.Data.Gold = playerObj.Data.Gold + math.floor(amount * bonusMultiplier)
    
    -- Update UI
    local goldEvent = ReplicatedStorage.Events:FindFirstChild("UpdateGold")
    if goldEvent then
        goldEvent:FireClient(playerObj.Player, {
            gold = playerObj.Data.Gold,
        })
    end
end

-- หา player object (Get player object)
function PlayerManager.Get(player)
    return activePlayers[player.UserId]
end

-- เมื่อ player ออกเกม (When player leaves)
function PlayerManager.OnPlayerRemoving(player)
    local playerObj = activePlayers[player.UserId]
    if not playerObj then return end
    
    -- อัพเดท play time
    playerObj.Data.TotalPlayTime = playerObj.Data.TotalPlayTime + 
        (os.time() - playerObj.Data.LastLogin)
    
    -- Save data
    DataManager:SavePlayer(player, playerObj.Data)
    
    -- ล้าง connections
    for _, conn in ipairs(playerObj._connections) do
        conn:Disconnect()
    end
    
    activePlayers[player.UserId] = nil
    print(player.Name, "left the game - data saved")
end

-- ดึง players ทั้งหมด
function PlayerManager.GetAll()
    return activePlayers
end

return PlayerManager
```

### 2.2 CombatManager

```lua
-- ModuleScript: CombatManager
-- ระบบต่อสู้หลักของเกม
-- Main combat system of the game

local CombatManager = {}
CombatManager.__index = CombatManager

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")

local GameConfig = require(ReplicatedStorage.Modules.GameConfig)
local PlayerManager = require(script.Parent.PlayerManager)

-- ==================== ดำเนินการโจมตี ====================

-- คำนวณ damage (Calculate damage)
local function calculateDamage(attacker, attackerStats, targetDefense, ability)
    local baseDamage = attackerStats.TotalAttack or attackerStats.Attack or 10
    
    if ability then
        baseDamage = baseDamage * (ability.DamageMultiplier or 1.0)
    end
    
    -- ลด damage ด้วย defense
    local mitigated = baseDamage * (100 / (100 + targetDefense))
    
    -- Random variance ±10%
    local variance = mitigated * 0.1
    local finalDamage = mitigated + math.random(-variance, variance)
    
    -- Critical hit (10% chance)
    local isCritical = math.random(1, 100) <= 10
    if isCritical then
        finalDamage = finalDamage * 1.5
    end
    
    return math.max(1, math.round(finalDamage)), isCritical
end

-- โจมตีพื้นฐาน (Basic attack)
function CombatManager.BasicAttack(attackerPlayer, targetCharacter)
    local attackerObj = PlayerManager.Get(attackerPlayer)
    if not attackerObj then return end
    
    -- Cooldown check
    local now = tick()
    if attackerObj._lastAttackTime and now - attackerObj._lastAttackTime < 0.5 then
        return false, "Attack too fast"
    end
    attackerObj._lastAttackTime = now
    
    -- Range check
    local attackerRoot = attackerObj.Character and attackerObj.Character:FindFirstChild("HumanoidRootPart")
    local targetRoot = targetCharacter:FindFirstChild("HumanoidRootPart")
    
    if not attackerRoot or not targetRoot then return false end
    
    local distance = (attackerRoot.Position - targetRoot.Position).Magnitude
    if distance > 10 then return false, "Out of range" end
    
    -- Calculate damage
    local targetHumanoid = targetCharacter:FindFirstChildOfClass("Humanoid")
    if not targetHumanoid or targetHumanoid.Health <= 0 then return false end
    
    -- Get target defense
    local targetPlayer = Players:GetPlayerFromCharacter(targetCharacter)
    local targetDefense = 5  -- default
    
    if targetPlayer then
        local targetObj = PlayerManager.Get(targetPlayer)
        if targetObj then
            targetDefense = targetObj.TotalDefense or targetObj.Data.Defense
        end
    end
    
    local damage, isCritical = calculateDamage(attackerObj, attackerObj, targetDefense)
    
    -- Apply damage
    targetHumanoid:TakeDamage(damage)
    
    -- Show damage number
    CombatManager.ShowDamageNumber(targetRoot.Position, damage, isCritical)
    
    -- Update attacker combat state
    attackerObj.IsInCombat = true
    
    return true, damage
end

-- ใช้ Ability (Use ability)
function CombatManager.UseAbility(attackerPlayer, abilityName, targetPosition, targetCharacter)
    local attackerObj = PlayerManager.Get(attackerPlayer)
    if not attackerObj then return false end
    
    local abilityConfig = GameConfig.ABILITIES[abilityName]
    if not abilityConfig then return false, "Unknown ability" end
    
    -- Check cooldown
    local now = tick()
    local cooldownKey = "cd_" .. abilityName
    local lastUsed = attackerObj[cooldownKey] or 0
    
    if now - lastUsed < abilityConfig.Cooldown then
        local remaining = math.ceil(abilityConfig.Cooldown - (now - lastUsed))
        return false, "Cooldown: " .. remaining .. "s"
    end
    
    -- Check mana
    if attackerObj.CurrentMana < abilityConfig.ManaCost then
        return false, "Not enough mana"
    end
    
    -- Check range
    local attackerRoot = attackerObj.Character and attackerObj.Character:FindFirstChild("HumanoidRootPart")
    if not attackerRoot then return false end
    
    if targetCharacter then
        local targetRoot = targetCharacter:FindFirstChild("HumanoidRootPart")
        if targetRoot then
            local dist = (attackerRoot.Position - targetRoot.Position).Magnitude
            if dist > abilityConfig.Range then
                return false, "Target out of range"
            end
        end
    end
    
    -- Deduct mana
    attackerObj.CurrentMana = attackerObj.CurrentMana - abilityConfig.ManaCost
    
    -- Set cooldown
    attackerObj[cooldownKey] = now
    
    -- Execute ability based on type
    if abilityConfig.Type == "Melee" then
        if targetCharacter then
            local targetHumanoid = targetCharacter:FindFirstChildOfClass("Humanoid")
            local targetDef = 5
            
            local targetPlayer = Players:GetPlayerFromCharacter(targetCharacter)
            if targetPlayer then
                local tObj = PlayerManager.Get(targetPlayer)
                if tObj then targetDef = tObj.TotalDefense end
            end
            
            local damage, isCrit = calculateDamage(attackerObj, attackerObj, targetDef, abilityConfig)
            targetHumanoid:TakeDamage(damage)
            CombatManager.ShowDamageNumber(targetCharacter.HumanoidRootPart.Position, damage, isCrit)
        end
        
    elseif abilityConfig.Type == "AOE" then
        -- หา targets ในรัศมี (Find targets in radius)
        for _, player in ipairs(Players:GetPlayers()) do
            if player ~= attackerPlayer and player.Character then
                local tRoot = player.Character:FindFirstChild("HumanoidRootPart")
                if tRoot then
                    local dist = (targetPosition - tRoot.Position).Magnitude
                    if dist <= abilityConfig.AOERadius then
                        local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
                        if humanoid then
                            local tObj = PlayerManager.Get(player)
                            local tDef = tObj and tObj.TotalDefense or 5
                            local damage, isCrit = calculateDamage(attackerObj, attackerObj, tDef, abilityConfig)
                            humanoid:TakeDamage(damage)
                            CombatManager.ShowDamageNumber(tRoot.Position, damage, isCrit)
                        end
                    end
                end
            end
        end
        
    elseif abilityConfig.Type == "Heal" then
        local targetObj = targetCharacter and 
            PlayerManager.Get(Players:GetPlayerFromCharacter(targetCharacter))
        
        local healTarget = targetCharacter or attackerObj.Character
        local healObj = targetObj or attackerObj
        
        if healTarget then
            local humanoid = healTarget:FindFirstChildOfClass("Humanoid")
            if humanoid then
                local healAmount = abilityConfig.HealAmount
                local newHealth = math.min(
                    humanoid.Health + healAmount,
                    humanoid.MaxHealth
                )
                humanoid.Health = newHealth
                
                if healObj then
                    healObj.CurrentHealth = newHealth
                end
                
                -- Show heal number (green)
                CombatManager.ShowHealNumber(healTarget.HumanoidRootPart.Position, healAmount)
            end
        end
        
    elseif abilityConfig.Type == "Buff" then
        -- Apply buff
        table.insert(attackerObj.ActiveBuffs, {
            name = abilityName,
            startTime = now,
            duration = abilityConfig.Duration,
            DefenseBonus = abilityConfig.DefenseBonus,
            AttackBonus = abilityConfig.AttackBonus,
        })
        
        PlayerManager.ApplyStats(attackerObj)
        
        -- ลบ buff หลัง duration
        task.delay(abilityConfig.Duration, function()
            for i, buff in ipairs(attackerObj.ActiveBuffs) do
                if buff.name == abilityName then
                    table.remove(attackerObj.ActiveBuffs, i)
                    PlayerManager.ApplyStats(attackerObj)
                    break
                end
            end
        end)
        
    elseif abilityConfig.Type == "AOE_Buff" then
        -- Apply buff to self and nearby allies
        local range = abilityConfig.Radius or 15
        
        for _, player in ipairs(Players:GetPlayers()) do
            local pObj = PlayerManager.Get(player)
            if pObj and pObj.Character then
                local pRoot = pObj.Character:FindFirstChild("HumanoidRootPart")
                if pRoot then
                    local dist = (attackerRoot.Position - pRoot.Position).Magnitude
                    if dist <= range then
                        table.insert(pObj.ActiveBuffs, {
                            name = abilityName,
                            startTime = now,
                            duration = abilityConfig.Duration,
                            AttackBonus = abilityConfig.AttackBonus,
                        })
                        PlayerManager.ApplyStats(pObj)
                        
                        task.delay(abilityConfig.Duration, function()
                            for i, buff in ipairs(pObj.ActiveBuffs) do
                                if buff.name == abilityName then
                                    table.remove(pObj.ActiveBuffs, i)
                                    PlayerManager.ApplyStats(pObj)
                                    break
                                end
                            end
                        end)
                    end
                end
            end
        end
    end
    
    -- Update mana UI
    local updateManaEvent = ReplicatedStorage.Events:FindFirstChild("UpdateMana")
    if updateManaEvent then
        updateManaEvent:FireClient(attackerPlayer, {
            mana = attackerObj.CurrentMana,
            maxMana = attackerObj.Data.MaxMana,
            cooldown = {
                ability = abilityName,
                remaining = abilityConfig.Cooldown,
            }
        })
    end
    
    return true
end

-- แสดงตัวเลข damage (Show damage number)
function CombatManager.ShowDamageNumber(position, damage, isCritical)
    local ReplicatedStorage = game:GetService("ReplicatedStorage")
    local showEvent = ReplicatedStorage.Events:FindFirstChild("ShowDamageNumber")
    
    if showEvent then
        showEvent:FireAllClients({
            position = position,
            damage = damage,
            isCritical = isCritical,
            type = "damage",
        })
    end
end

-- แสดงตัวเลข heal (Show heal number)
function CombatManager.ShowHealNumber(position, amount)
    local ReplicatedStorage = game:GetService("ReplicatedStorage")
    local showEvent = ReplicatedStorage.Events:FindFirstChild("ShowDamageNumber")
    
    if showEvent then
        showEvent:FireAllClients({
            position = position,
            damage = amount,
            type = "heal",
        })
    end
end

-- Mana regeneration
function CombatManager.StartManaRegen()
    local RunService = game:GetService("RunService")
    
    RunService.Heartbeat:Connect(function()
        local now = tick()
        
        for _, playerObj in pairs(PlayerManager.GetAll()) do
            if not playerObj.IsInCombat then
                -- regen 2% mana per second out of combat
                local regenRate = playerObj.Data.MaxMana * 0.02
                playerObj.CurrentMana = math.min(
                    playerObj.CurrentMana + regenRate / 60,
                    playerObj.Data.MaxMana
                )
            end
        end
    end)
end

return CombatManager
```

---

## Phase 3: Game Systems

### 3.1 QuestManager

```lua
-- ModuleScript: QuestManager
-- ระบบ Quest ของเกม
-- Game quest system

local QuestManager = {}

local ReplicatedStorage = game:GetService("ReplicatedStorage")
local GameConfig = require(ReplicatedStorage.Modules.GameConfig)
local PlayerManager = require(script.Parent.PlayerManager)

-- เริ่ม Quest (Start quest)
function QuestManager.AcceptQuest(player, questId)
    local playerObj = PlayerManager.Get(player)
    if not playerObj then return false end
    
    local quest = nil
    for _, q in ipairs(GameConfig.QUESTS) do
        if q.id == questId then quest = q break end
    end
    
    if not quest then return false, "Quest not found" end
    
    -- ตรวจสอบ level requirement
    if playerObj.Data.Level < quest.Level then
        return false, "Level too low"
    end
    
    -- ตรวจสอบว่าทำไปแล้วหรือยัง
    if playerObj.Data.QuestsCompleted[questId] then
        return false, "Quest already completed"
    end
    
    -- ตรวจสอบว่ากำลังทำอยู่หรือไม่
    if playerObj.QuestProgress[questId] then
        return false, "Quest already active"
    end
    
    -- เริ่ม quest progress tracking
    playerObj.QuestProgress[questId] = {
        questId = questId,
        progress = 0,
        required = quest.Objective.Required,
        startTime = os.time(),
    }
    
    -- แจ้ง client
    local questUpdateEvent = ReplicatedStorage.Events:FindFirstChild("QuestUpdate")
    if questUpdateEvent then
        questUpdateEvent:FireClient(player, {
            type = "accepted",
            quest = quest,
            progress = playerObj.QuestProgress[questId],
        })
    end
    
    return true
end

-- อัพเดท quest progress (Update quest progress)
function QuestManager.UpdateProgress(player, actionType, target, amount)
    local playerObj = PlayerManager.Get(player)
    if not playerObj then return end
    
    for questId, progress in pairs(playerObj.QuestProgress) do
        local quest = nil
        for _, q in ipairs(GameConfig.QUESTS) do
            if q.id == questId then quest = q break end
        end
        
        if not quest then continue end
        
        -- ตรวจสอบว่า action ตรงกับ objective
        local obj = quest.Objective
        if obj.Type == actionType and obj.Target == target then
            progress.progress = progress.progress + (amount or 1)
            
            -- ตรวจสอบว่า complete หรือยัง
            if progress.progress >= progress.required then
                QuestManager.CompleteQuest(player, questId)
            else
                -- แจ้ง progress update
                local questUpdateEvent = ReplicatedStorage.Events:FindFirstChild("QuestUpdate")
                if questUpdateEvent then
                    questUpdateEvent:FireClient(player, {
                        type = "progress",
                        questId = questId,
                        progress = progress,
                    })
                end
            end
        end
    end
end

-- Complete Quest (Complete quest and give rewards)
function QuestManager.CompleteQuest(player, questId)
    local playerObj = PlayerManager.Get(player)
    if not playerObj then return end
    
    local quest = nil
    for _, q in ipairs(GameConfig.QUESTS) do
        if q.id == questId then quest = q break end
    end
    
    if not quest then return end
    
    -- ลบ quest progress
    playerObj.QuestProgress[questId] = nil
    
    -- Mark as completed
    playerObj.Data.QuestsCompleted[questId] = os.time()
    
    -- ให้ rewards
    PlayerManager.AddGold(playerObj, quest.Rewards.Gold)
    PlayerManager.AddExperience(playerObj, quest.Rewards.Experience)
    
    if quest.Rewards.Items then
        for _, itemId in ipairs(quest.Rewards.Items) do
            -- Add item to inventory
            table.insert(playerObj.Data.Inventory, {
                id = itemId,
                quantity = 1,
            })
        end
    end
    
    -- แจ้ง completion
    local questUpdateEvent = ReplicatedStorage.Events:FindFirstChild("QuestUpdate")
    if questUpdateEvent then
        questUpdateEvent:FireClient(player, {
            type = "completed",
            questId = questId,
            questName = quest.Name,
            rewards = quest.Rewards,
        })
    end
    
    print(player.Name, "completed quest:", quest.Name)
end

return QuestManager
```

---

## Phase 4: UI System

### 4.1 HUD Manager (Client)

```lua
-- LocalScript: HUDManager
-- จัดการ HUD ของผู้เล่น
-- Manages player HUD

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local player = Players.LocalPlayer
local playerGui = player.PlayerGui

-- รอ HUD GUI
local hudGui = playerGui:WaitForChild("MainHUD")
local hudFrame = hudGui:WaitForChild("HUDFrame")

-- Elements
local healthBar = hudFrame:WaitForChild("HealthBar"):WaitForChild("Fill")
local healthLabel = hudFrame:WaitForChild("HealthBar"):WaitForChild("Label")
local manaBar = hudFrame:WaitForChild("ManaBar"):WaitForChild("Fill")
local manaLabel = hudFrame:WaitForChild("ManaBar"):WaitForChild("Label")
local levelLabel = hudFrame:WaitForChild("LevelLabel")
local goldLabel = hudFrame:WaitForChild("GoldLabel")
local xpBar = hudFrame:WaitForChild("XPBar"):WaitForChild("Fill")

-- อัพเดท health bar (Update health bar)
local function updateHealthBar(current, max)
    local percent = math.clamp(current / max, 0, 1)
    
    TweenService:Create(
        healthBar,
        TweenInfo.new(0.3, Enum.EasingStyle.Quad),
        {Size = UDim2.new(percent, 0, 1, 0)}
    ):Play()
    
    healthLabel.Text = current .. "/" .. max
    
    -- เปลี่ยนสีตาม HP
    if percent > 0.6 then
        healthBar.BackgroundColor3 = Color3.fromRGB(80, 200, 80)
    elseif percent > 0.3 then
        healthBar.BackgroundColor3 = Color3.fromRGB(220, 180, 50)
    else
        healthBar.BackgroundColor3 = Color3.fromRGB(220, 60, 60)
        -- Flash effect เมื่อ HP ต่ำ
    end
end

-- อัพเดท mana bar (Update mana bar)
local function updateManaBar(current, max)
    local percent = math.clamp(current / max, 0, 1)
    
    TweenService:Create(
        manaBar,
        TweenInfo.new(0.3),
        {Size = UDim2.new(percent, 0, 1, 0)}
    ):Play()
    
    manaLabel.Text = current .. "/" .. max
end

-- ==================== Remote Event Handlers ====================
local events = ReplicatedStorage:WaitForChild("Events")

-- อัพเดท HUD ทั้งหมด
events:WaitForChild("UpdateHUD").OnClientEvent:Connect(function(data)
    updateHealthBar(data.health, data.maxHealth)
    updateManaBar(data.mana, data.maxMana)
    levelLabel.Text = "Lv." .. data.level
    goldLabel.Text = "💰 " .. tostring(data.gold)
end)

-- อัพเดท XP
events:WaitForChild("UpdateXP").OnClientEvent:Connect(function(data)
    local percent = data.exp / data.requiredExp
    TweenService:Create(
        xpBar,
        TweenInfo.new(0.5, Enum.EasingStyle.Quad),
        {Size = UDim2.new(percent, 0, 1, 0)}
    ):Play()
end)

-- Level Up notification
events:WaitForChild("LevelUp").OnClientEvent:Connect(function(data)
    -- แสดง Level Up animation
    local notification = Instance.new("Frame")
    notification.Size = UDim2.new(0, 300, 0, 100)
    notification.Position = UDim2.new(0.5, -150, 0.5, -50)
    notification.BackgroundColor3 = Color3.fromRGB(255, 215, 0)
    notification.BackgroundTransparency = 0.2
    notification.Parent = hudGui
    
    local label = Instance.new("TextLabel", notification)
    label.Size = UDim2.new(1, 0, 1, 0)
    label.BackgroundTransparency = 1
    label.Text = "⭐ LEVEL UP! ⭐\nLevel " .. data.newLevel
    label.TextColor3 = Color3.fromRGB(50, 30, 0)
    label.TextSize = 24
    label.Font = Enum.Font.GothamBold
    
    -- Animate and remove
    TweenService:Create(
        notification,
        TweenInfo.new(0.5, Enum.EasingStyle.Back, Enum.EasingDirection.Out),
        {Size = UDim2.new(0, 350, 0, 120)}
    ):Play()
    
    task.delay(2, function()
        TweenService:Create(
            notification,
            TweenInfo.new(0.5),
            {Position = UDim2.new(0.5, -175, 0.3, 0), BackgroundTransparency = 1}
        ):Play()
        
        task.delay(0.5, function()
            notification:Destroy()
        end)
    end)
    
    levelLabel.Text = "Lv." .. data.newLevel
end)

-- Damage numbers
events:WaitForChild("ShowDamageNumber").OnClientEvent:Connect(function(data)
    local camera = workspace.CurrentCamera
    
    -- Convert 3D position to 2D screen position
    local screenPos, onScreen = camera:WorldToScreenPoint(data.position + Vector3.new(0, 3, 0))
    
    if not onScreen then return end
    
    local label = Instance.new("TextLabel")
    label.Position = UDim2.new(0, screenPos.X - 30, 0, screenPos.Y - 15)
    label.Size = UDim2.new(0, 60, 0, 30)
    label.BackgroundTransparency = 1
    label.TextSize = data.isCritical and 24 or 18
    label.Font = Enum.Font.GothamBold
    label.ZIndex = 10
    
    if data.type == "heal" then
        label.Text = "+" .. data.damage
        label.TextColor3 = Color3.fromRGB(100, 255, 100)
    elseif data.isCritical then
        label.Text = "💥 " .. data.damage
        label.TextColor3 = Color3.fromRGB(255, 200, 50)
    else
        label.Text = tostring(data.damage)
        label.TextColor3 = Color3.fromRGB(255, 80, 80)
    end
    
    label.Parent = hudGui
    
    -- Animate upward and fade
    TweenService:Create(
        label,
        TweenInfo.new(1.5, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
        {
            Position = UDim2.new(0, screenPos.X - 30, 0, screenPos.Y - 60),
            TextTransparency = 1,
        }
    ):Play()
    
    game:GetService("Debris"):AddItem(label, 1.5)
end)

-- Quest update notifications
events:WaitForChild("QuestUpdate").OnClientEvent:Connect(function(data)
    if data.type == "completed" then
        -- Quest complete notification
        local notification = Instance.new("Frame")
        notification.Size = UDim2.new(0, 350, 0, 80)
        notification.Position = UDim2.new(1, 10, 0.7, 0)
        notification.BackgroundColor3 = Color3.fromRGB(50, 150, 50)
        notification.BackgroundTransparency = 0.1
        notification.Parent = hudGui
        
        local notifLabel = Instance.new("TextLabel", notification)
        notifLabel.Size = UDim2.new(1, -10, 1, 0)
        notifLabel.Position = UDim2.new(0, 5, 0, 0)
        notifLabel.BackgroundTransparency = 1
        notifLabel.Text = "✅ Quest Complete!\n" .. (data.questName or "") ..
            "\n+" .. (data.rewards and data.rewards.Gold or 0) .. " Gold"
        notifLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
        notifLabel.TextSize = 14
        notifLabel.Font = Enum.Font.GothamBold
        
        -- Slide in
        TweenService:Create(
            notification,
            TweenInfo.new(0.5, Enum.EasingStyle.Back),
            {Position = UDim2.new(1, -360, 0.7, 0)}
        ):Play()
        
        -- Slide out and remove
        task.delay(3, function()
            TweenService:Create(
                notification,
                TweenInfo.new(0.5),
                {Position = UDim2.new(1, 10, 0.7, 0)}
            ):Play()
            task.delay(0.6, function()
                notification:Destroy()
            end)
        end)
    end
end)
```

---

## Phase 5: Main Scripts

### 5.1 Server Main

```lua
-- Script: Main.server
-- Entry point สำหรับ server-side logic
-- Server-side entry point

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- โหลด managers
local PlayerManager = require(game.ServerScriptService.Modules.PlayerManager)
local CombatManager = require(game.ServerScriptService.Modules.CombatManager)
local QuestManager = require(game.ServerScriptService.Modules.QuestManager)

print("=== Legends of Roblox Server Starting ===")

-- ==================== Player Events ====================
Players.PlayerAdded:Connect(function(player)
    PlayerManager.OnPlayerAdded(player)
end)

Players.PlayerRemoving:Connect(function(player)
    PlayerManager.OnPlayerRemoving(player)
end)

-- Handle players who joined before script loaded
for _, player in ipairs(Players:GetPlayers()) do
    PlayerManager.OnPlayerAdded(player)
end

-- ==================== Remote Events ====================
local remotes = ReplicatedStorage.Events

-- Combat: Basic Attack
remotes.BasicAttack.OnServerEvent:Connect(function(player, targetCharacter)
    local success, result = CombatManager.BasicAttack(player, targetCharacter)
    if not success then
        -- ไม่ต้องแจ้ง error เล็กๆ
    end
end)

-- Combat: Use Ability
remotes.UseAbility.OnServerEvent:Connect(function(player, abilityName, targetPosition, targetCharacter)
    local success, err = CombatManager.UseAbility(player, abilityName, targetPosition, targetCharacter)
    if not success and err then
        remotes.AbilityFailed:FireClient(player, err)
    end
end)

-- Quest: Accept
remotes.AcceptQuest.OnServerEvent:Connect(function(player, questId)
    QuestManager.AcceptQuest(player, questId)
end)

-- ==================== Start Systems ====================
CombatManager.StartManaRegen()

print("=== Server Ready! ===")
```

### 5.2 Client Main

```lua
-- LocalScript: ClientMain
-- Entry point สำหรับ client-side logic
-- Client-side entry point

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()

print("Client started for:", player.Name)

-- ==================== Input Handling ====================
local remotes = ReplicatedStorage:WaitForChild("Events")

-- Mouse click = attack
local mouse = player:GetMouse()
local isAttacking = false

mouse.Button1Down:Connect(function()
    if isAttacking then return end
    
    local target = mouse.Target
    if not target then return end
    
    local targetCharacter = target:FindFirstAncestorOfClass("Model")
    if not targetCharacter then return end
    
    -- Check ว่าเป็น NPC หรือ player
    local targetHumanoid = targetCharacter:FindFirstChildOfClass("Humanoid")
    if not targetHumanoid or targetHumanoid.Health <= 0 then return end
    
    -- ไม่โจมตีตัวเอง
    if targetCharacter == character then return end
    
    isAttacking = true
    remotes.BasicAttack:FireServer(targetCharacter)
    
    task.wait(0.5)  -- attack animation time
    isAttacking = false
end)

-- Keyboard shortcuts สำหรับ abilities
local abilityKeys = {
    [Enum.KeyCode.Q] = "Ability1",
    [Enum.KeyCode.E] = "Ability2",
    [Enum.KeyCode.R] = "Ability3",
    [Enum.KeyCode.F] = "Ability4",
}

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    local abilitySlot = abilityKeys[input.KeyCode]
    if not abilitySlot then return end
    
    -- TODO: Map slot to ability based on equipped abilities
    -- ตัวอย่าง: ถ้าเป็น Warrior class ใช้ Q = PowerStrike
    local abilityName = "PowerStrike"  -- placeholder
    
    local mouseTarget = mouse.Target
    local targetCharacter = mouseTarget and mouseTarget:FindFirstAncestorOfClass("Model")
    local targetPos = mouse.Hit.Position
    
    remotes.UseAbility:FireServer(abilityName, targetPos, targetCharacter)
end)

-- Inventory toggle
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    if input.KeyCode == Enum.KeyCode.I or input.KeyCode == Enum.KeyCode.Tab then
        -- Toggle inventory
        local inventoryGui = player.PlayerGui:FindFirstChild("InventoryGui")
        if inventoryGui then
            inventoryGui.Enabled = not inventoryGui.Enabled
        end
    end
end)

print("Client ready!")
```

---

## Phase 6: Launch Checklist

### 6.1 Pre-Launch Checklist สมบูรณ์

```
PRE-LAUNCH CHECKLIST สำหรับ "Legends of Roblox":

TECHNICAL:
[ ] DataStore save/load ทำงานถูกต้อง
[ ] ไม่มี race conditions ใน DataStore
[ ] Remote Events มี validation ครบ
[ ] Rate limiting ทำงานถูกต้อง
[ ] Anti-cheat ผ่าน testing
[ ] Performance: server < 70% capacity ด้วย 20 players
[ ] Memory leaks: ไม่มี connections ค้างใน console
[ ] No script errors ใน output
[ ] Cross-server features ทำงาน (ถ้ามี)

GAMEPLAY:
[ ] Tutorial สมบูรณ์ (player รู้วิธีเล่นทันที)
[ ] Tutorial ไม่สามารถ skip ได้โดยไม่เจตนา
[ ] Death/Respawn ทำงานถูกต้อง
[ ] Quest ทุกอันสามารถ complete ได้
[ ] Boss encounters สามารถ defeat ได้
[ ] ไม่มี soft-lock situations
[ ] Difficulty balance: เกมไม่ยากหรือง่ายเกินไป
[ ] End-game content มี (สำหรับ level max)

MONETIZATION:
[ ] GamePass ทุกอันให้ benefits จริง
[ ] Developer Products ให้ items จริง
[ ] ราคาสมเหตุสมผล
[ ] Tooltip บอก benefits ชัดเจน
[ ] ทดสอบ purchase flow ครบ (sandbox mode)
[ ] ไม่มี Pay-to-Win เกินไป (ยังสนุกได้แบบฟรี)

CONTENT:
[ ] ชื่อและ description เกม SEO-friendly
[ ] Thumbnail และ icon คุณภาพดี
[ ] Social media ready (screenshots, trailer)
[ ] Game description ภาษาอังกฤษ ชัดเจน
[ ] Age rating ถูกต้อง

LEGAL/COMPLIANCE:
[ ] ไม่มีเนื้อหาละเมิดลิขสิทธิ์ (เพลง, assets)
[ ] Terms & Conditions สำหรับ purchase
[ ] Privacy policy (ถ้าเก็บ user data)
[ ] COPPA compliance (สำหรับผู้ใช้อายุต่ำกว่า 13)

COMMUNITY:
[ ] Discord server ตั้งค่าเสร็จแล้ว
[ ] Feedback/bug report channel
[ ] Admin system ทำงาน
[ ] Moderation team พร้อม
```

---

## สรุปสุดท้าย (Final Summary)

### สิ่งที่เรียนได้จาก Course นี้ทั้งหมด 100 บท:

```
Parts 1-20: พื้นฐาน Lua และ Roblox Studio
  - Lua syntax, variables, functions, OOP
  - Roblox Studio interface, Parts, Models
  - Scripts, LocalScripts, ModuleScripts
  - Events, RemoteEvents, RemoteFunctions
  - Basic UI creation

Parts 21-40: Game Mechanics พื้นฐาน
  - Player movement, combat basics
  - Inventory systems
  - NPC basics
  - DataStore introduction
  - Leaderboards

Parts 41-60: Intermediate Systems
  - Advanced OOP patterns
  - Custom physics
  - Particle effects
  - Sound design
  - Advanced UI animations

Parts 61-80: Advanced Systems
  - Performance optimization
  - Advanced networking
  - Custom game frameworks
  - Mobile/touch support
  - Accessibility

Parts 81-100: Professional/Expert Level
  - ProfileService và DataStore ขั้นสูง (81)
  - Analytics system (82)
  - Monetization strategies (83)
  - Developer Products (84)
  - GamePass implementation (85)
  - VIP system (86)
  - In-game advertising (87)
  - Testing & debugging (88)
  - Git & version control (89)
  - Team development (90)
  - Publishing & marketing (91)
  - Community management (92)
  - Update strategy (93)
  - Advanced animations (94)
  - Procedural generation (95)
  - Advanced AI & behavior trees (96)
  - Advanced networking & lag compensation (97)
  - Plugin development (98)
  - Portfolio & career (99)
  - Final project: Complete RPG (100)
```

### ข้อความส่งท้าย:

```
🎊 ยินดีด้วยที่จบ Course "Roblox Game Development" ครบ 100 บท! 🎊

คุณได้เรียนรู้สิ่งที่ developer มืออาชีพใช้จริงในอุตสาหกรรม
ตอนนี้คุณมีทักษะที่ครบครันเพื่อ:

✅ สร้างเกมคุณภาพสูงที่มีผู้เล่นนับล้าน
✅ ทำงานในทีม Roblox studio ระดับโลก
✅ เริ่มต้น studio ของตัวเอง
✅ สร้างรายได้จาก Roblox อย่างยั่งยืน

สิ่งสำคัญที่สุดที่ต้องจำ:
1. Code ที่ดีคือ code ที่คนอื่นอ่านแล้วเข้าใจ
2. Test เสมอก่อน launch
3. Community feedback ล้ำค่ากว่า assumption
4. Never stop learning — Roblox อัพเดทตลอด
5. Share knowledge กับ community — มันจะกลับมาหาคุณ

ขอให้โชคดีในการเดินทางสู่ความสำเร็จ!
Happy developing! 🚀
```

---

*จบ Course: Roblox Game Development - 100 บท*
*คุณพร้อมแล้วสำหรับการเดินทางในฐานะ Professional Roblox Developer!*
