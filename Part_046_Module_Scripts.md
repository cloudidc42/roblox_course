# Part 46: ModuleScripts - การจัดระเบียบโค้ด

## บทนำ

ModuleScript คือ script พิเศษใน Roblox ที่ช่วยให้เราแบ่งโค้ดออกเป็นส่วนๆ แล้ว require มาใช้จาก script อื่น คล้ายกับ "library" หรือ "module" ในภาษาโปรแกรมอื่น เป็นเครื่องมือที่สำคัญที่สุดอย่างหนึ่งในการพัฒนาเกมที่ซับซ้อน

## ทำไมต้องใช้ ModuleScript?

### ปัญหาที่พบโดยไม่ใช้ ModuleScript

```lua
-- ❌ โค้ดทุกอย่างใน Script เดียว (ยาว ซับซ้อน ดูแลยาก)

-- ServerScript.lua (5000 บรรทัด!)
-- player management code...
-- combat code...
-- economy code...
-- quest code...
-- inventory code...
-- NPC code...
-- UI code...
```

### วิธีที่ดีกว่าด้วย ModuleScripts

```
ServerScriptService/
├── GameManager.lua        (Script หลัก - เล็ก)
└── Modules/
    ├── PlayerModule.lua   (ModuleScript)
    ├── CombatModule.lua   (ModuleScript)
    ├── EconomyModule.lua  (ModuleScript)
    ├── QuestModule.lua    (ModuleScript)
    └── InventoryModule.lua (ModuleScript)
```

## การสร้าง ModuleScript พื้นฐาน

```lua
-- ModuleScript ชื่อ "MathUtils"
-- สร้างใน ReplicatedStorage หรือ ServerStorage

-- ทุก ModuleScript ต้องคืน table (หรือ function)
local MathUtils = {}

-- ฟังก์ชันต่างๆ ที่ต้องการ export
function MathUtils.clamp(value, min, max)
    return math.max(min, math.min(max, value))
end

function MathUtils.lerp(a, b, t)
    return a + (b - a) * t
end

function MathUtils.round(value, decimals)
    decimals = decimals or 0
    local factor = 10 ^ decimals
    return math.floor(value * factor + 0.5) / factor
end

function MathUtils.randomRange(min, max)
    return min + math.random() * (max - min)
end

function MathUtils.distance(a, b)
    return (b - a).Magnitude
end

function MathUtils.percentOf(part, whole)
    if whole == 0 then return 0 end
    return (part / whole) * 100
end

-- ต้อง return module เสมอ!
return MathUtils
```

### การใช้งาน

```lua
-- Script อื่น
local MathUtils = require(game.ReplicatedStorage.Modules.MathUtils)

-- ใช้ฟังก์ชัน
local health = 75
local maxHealth = 100
local percent = MathUtils.percentOf(health, maxHealth)
print(string.format("HP: %d%%", percent))  -- HP: 75%

local clampedValue = MathUtils.clamp(150, 0, 100)
print(clampedValue)  -- 100

local distance = MathUtils.distance(Vector3.new(0,0,0), Vector3.new(3,4,0))
print(MathUtils.round(distance, 2))  -- 5
```

## ModuleScript Pattern ต่างๆ

### Pattern 1: Singleton (มีเพียงตัวเดียว)

```lua
-- ReplicatedStorage/Modules/Config.lua (ModuleScript)
-- ใช้เก็บ configuration ของเกม

local Config = {
    -- Game Settings
    game = {
        maxPlayers = 20,
        roundDuration = 300,    -- 5 นาที
        lobbyWaitTime = 30,
        minPlayersToStart = 2
    },
    
    -- Economy
    economy = {
        startingCoins = 100,
        coinValueMultiplier = 1.0,
        shopTaxRate = 0.05        -- 5% ภาษี
    },
    
    -- Combat
    combat = {
        defaultDamage = 10,
        criticalChance = 0.15,
        criticalMultiplier = 2.0,
        respawnTime = 5,
        invincibilityTime = 3
    },
    
    -- Leveling
    leveling = {
        baseExpRequired = 100,
        expMultiplier = 1.5,      -- ต้องการ exp เพิ่มขึ้น 50% ต่อ level
        maxLevel = 100
    },
    
    -- ฟังก์ชันช่วยเหลือ
    getExpRequired = function(level)
        return math.floor(100 * (1.5 ^ (level - 1)))
    end,
    
    isMaxLevel = function(level)
        return level >= 100
    end
}

-- ป้องกันการแก้ไขจากภายนอก
return table.freeze(Config)  -- Lua 5.4+ / Roblox supports this
```

### Pattern 2: Factory

```lua
-- ReplicatedStorage/Modules/ItemFactory.lua (ModuleScript)
-- สร้าง item objects

local ItemFactory = {}

-- Base item class
local function createItem(itemData)
    local item = {
        id = itemData.id,
        name = itemData.name,
        description = itemData.description or "",
        icon = itemData.icon or "rbxassetid://0",
        rarity = itemData.rarity or "common",
        stackable = itemData.stackable or false,
        maxStack = itemData.maxStack or 1,
        value = itemData.value or 0,
        
        -- Methods
        getDisplayName = function(self)
            local rarityColors = {
                common = "⬜",
                uncommon = "🟩",
                rare = "🟦",
                epic = "🟪",
                legendary = "🟨"
            }
            local prefix = rarityColors[self.rarity] or ""
            return prefix .. " " .. self.name
        end,
        
        getSellPrice = function(self)
            local rarityMultiplier = {
                common = 1,
                uncommon = 2,
                rare = 5,
                epic = 10,
                legendary = 25
            }
            return math.floor(self.value * (rarityMultiplier[self.rarity] or 1))
        end
    }
    return item
end

-- Weapon factory
function ItemFactory.createWeapon(data)
    local weapon = createItem(data)
    weapon.type = "weapon"
    weapon.damage = data.damage or 10
    weapon.attackSpeed = data.attackSpeed or 1.0
    weapon.range = data.range or 5
    weapon.durability = data.durability or 100
    weapon.maxDurability = data.maxDurability or 100
    
    -- Weapon methods
    weapon.getDPS = function(self)
        return self.damage * self.attackSpeed
    end
    
    weapon.isDamaged = function(self)
        return self.durability < self.maxDurability * 0.25
    end
    
    weapon.repair = function(self, amount)
        self.durability = math.min(self.maxDurability, self.durability + amount)
    end
    
    return weapon
end

-- Armor factory
function ItemFactory.createArmor(data)
    local armor = createItem(data)
    armor.type = "armor"
    armor.defense = data.defense or 5
    armor.armorSlot = data.armorSlot or "body"  -- head, body, legs, feet
    armor.durability = data.durability or 100
    
    return armor
end

-- Consumable factory
function ItemFactory.createConsumable(data)
    local consumable = createItem(data)
    consumable.type = "consumable"
    consumable.stackable = true
    consumable.maxStack = data.maxStack or 99
    consumable.effect = data.effect  -- function ที่ใช้เมื่อ consume
    consumable.cooldown = data.cooldown or 0
    
    consumable.use = function(self, player)
        if self.effect then
            self.effect(player)
            return true
        end
        return false
    end
    
    return consumable
end

-- Item Database
local itemDatabase = {}

function ItemFactory.registerItem(itemData)
    local item
    
    if itemData.type == "weapon" then
        item = ItemFactory.createWeapon(itemData)
    elseif itemData.type == "armor" then
        item = ItemFactory.createArmor(itemData)
    elseif itemData.type == "consumable" then
        item = ItemFactory.createConsumable(itemData)
    else
        item = createItem(itemData)
    end
    
    itemDatabase[itemData.id] = item
    return item
end

function ItemFactory.getItem(id)
    return itemDatabase[id]
end

function ItemFactory.getAllItems()
    return itemDatabase
end

-- ลงทะเบียน items เริ่มต้น
ItemFactory.registerItem({
    id = "sword_basic",
    name = "ดาบธรรมดา",
    description = "ดาบสำหรับมือใหม่",
    type = "weapon",
    damage = 15,
    attackSpeed = 1.2,
    rarity = "common",
    value = 100
})

ItemFactory.registerItem({
    id = "sword_iron",
    name = "ดาบเหล็ก",
    type = "weapon",
    damage = 30,
    attackSpeed = 1.0,
    rarity = "uncommon",
    value = 500
})

ItemFactory.registerItem({
    id = "potion_hp_small",
    name = "ยาแดงเล็ก",
    type = "consumable",
    maxStack = 99,
    rarity = "common",
    value = 25,
    effect = function(player)
        local humanoid = player.Character and 
                         player.Character:FindFirstChild("Humanoid")
        if humanoid then
            humanoid.Health = math.min(humanoid.MaxHealth, humanoid.Health + 25)
        end
    end
})

return ItemFactory
```

### Pattern 3: Service (Service Provider Pattern)

```lua
-- ServerScriptService/Services/NotificationService.lua (ModuleScript)
-- Service ที่ให้บริการแสดง notification

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local NotificationService = {}

-- Private state
local notifyEvent = nil
local initialized = false

-- เริ่มต้น service
function NotificationService.init()
    if initialized then return end
    
    -- สร้าง RemoteEvent ถ้ายังไม่มี
    local events = ReplicatedStorage:FindFirstChild("Events")
    if not events then
        events = Instance.new("Folder")
        events.Name = "Events"
        events.Parent = ReplicatedStorage
    end
    
    notifyEvent = events:FindFirstChild("ShowNotification")
    if not notifyEvent then
        notifyEvent = Instance.new("RemoteEvent")
        notifyEvent.Name = "ShowNotification"
        notifyEvent.Parent = events
    end
    
    initialized = true
    print("[NotificationService] Initialized")
end

-- ส่ง notification ให้ผู้เล่นคนเดียว
function NotificationService.notify(player, message, notifType, duration)
    if not initialized then
        warn("[NotificationService] ยังไม่ได้ init!")
        return
    end
    
    notifyEvent:FireClient(player, {
        message = message,
        type = notifType or "info",
        duration = duration or 3,
        timestamp = os.time()
    })
end

-- ส่งให้ทุกคน
function NotificationService.notifyAll(message, notifType, duration)
    if not initialized then return end
    
    notifyEvent:FireAllClients({
        message = message,
        type = notifType or "announcement",
        duration = duration or 5,
        timestamp = os.time()
    })
end

-- Types ของ notification
NotificationService.Types = {
    INFO = "info",
    SUCCESS = "success",
    WARNING = "warning",
    ERROR = "error",
    ANNOUNCEMENT = "announcement"
}

-- เริ่มต้น service ทันที
NotificationService.init()

return NotificationService
```

## ModuleScript แบบ Shared (Client และ Server ใช้ร่วมกัน)

```lua
-- ReplicatedStorage/Shared/Constants.lua (ModuleScript)
-- ค่าคงที่ที่ทั้ง Client และ Server ใช้

local Constants = {}

-- Game States
Constants.GameState = {
    LOBBY = "lobby",
    COUNTDOWN = "countdown",
    PLAYING = "playing",
    ENDED = "ended"
}

-- Item Rarities
Constants.Rarity = {
    COMMON = "common",
    UNCOMMON = "uncommon",
    RARE = "rare",
    EPIC = "epic",
    LEGENDARY = "legendary"
}

Constants.RarityColors = {
    common = Color3.fromRGB(200, 200, 200),
    uncommon = Color3.fromRGB(30, 200, 30),
    rare = Color3.fromRGB(30, 100, 255),
    epic = Color3.fromRGB(160, 30, 255),
    legendary = Color3.fromRGB(255, 165, 0)
}

-- Notification Types
Constants.NotifType = {
    INFO = "info",
    SUCCESS = "success",
    WARNING = "warning",
    ERROR = "error"
}

-- Damage Types
Constants.DamageType = {
    PHYSICAL = "physical",
    MAGIC = "magic",
    FIRE = "fire",
    ICE = "ice",
    POISON = "poison",
    TRUE = "true"  -- ทะลุ defense
}

-- Max values
Constants.MaxHealth = 100
Constants.MaxLevel = 100
Constants.MaxInventorySlots = 50

return Constants
```

## การจัดระเบียบโครงสร้าง ModuleScript

```
ReplicatedStorage/
├── Shared/              <- ทั้ง Client และ Server ใช้ได้
│   ├── Constants.lua
│   ├── Types.lua
│   ├── Utils/
│   │   ├── MathUtils.lua
│   │   ├── StringUtils.lua
│   │   └── TableUtils.lua
│   └── Data/
│       ├── ItemData.lua
│       └── QuestData.lua
│
ServerStorage/
└── Server/              <- เฉพาะ Server เท่านั้น
    ├── Services/
    │   ├── DataService.lua
    │   ├── CombatService.lua
    │   └── EconomyService.lua
    └── Systems/
        ├── QuestSystem.lua
        └── AchievementSystem.lua
```

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: StringUtils
สร้าง ModuleScript สำหรับจัดการ strings

```lua
-- StringUtils.lua
local StringUtils = {}

function StringUtils.trim(s)
    return s:match("^%s*(.-)%s*$")
end

function StringUtils.split(s, delimiter)
    delimiter = delimiter or ","
    local result = {}
    for part in s:gmatch("[^" .. delimiter .. "]+") do
        table.insert(result, StringUtils.trim(part))
    end
    return result
end

function StringUtils.capitalize(s)
    return s:sub(1,1):upper() .. s:sub(2):lower()
end

function StringUtils.formatNumber(n)
    -- 1000000 -> "1,000,000"
    local s = tostring(math.floor(n))
    local result = ""
    local count = 0
    
    for i = #s, 1, -1 do
        if count > 0 and count % 3 == 0 then
            result = "," .. result
        end
        result = s:sub(i,i) .. result
        count = count + 1
    end
    
    return result
end

function StringUtils.abbreviateNumber(n)
    if n >= 1000000 then
        return string.format("%.1fM", n/1000000)
    elseif n >= 1000 then
        return string.format("%.1fK", n/1000)
    else
        return tostring(n)
    end
end

return StringUtils
```

### แบบฝึกหัดที่ 2: TableUtils
สร้าง ModuleScript สำหรับจัดการ tables

### แบบฝึกหัดที่ 3: ItemDatabase
สร้าง ModuleScript ที่เก็บข้อมูล items ทั้งหมดของเกม

## เคล็ดลับ

```lua
-- 1. ใช้ table.freeze ป้องกันการแก้ไขโดยไม่ตั้งใจ
local ReadOnlyConfig = table.freeze({
    maxPlayers = 20,
    roundTime = 300
})

-- 2. Lazy loading - โหลดเฉพาะเมื่อจำเป็น
local _expensiveModule = nil
local function getExpensiveModule()
    if not _expensiveModule then
        _expensiveModule = require(script.Parent.ExpensiveModule)
    end
    return _expensiveModule
end

-- 3. Memoization - cache ผลลัพธ์ที่คำนวณแล้ว
local expCache = {}
local function getExpRequired(level)
    if not expCache[level] then
        expCache[level] = math.floor(100 * (1.5 ^ (level-1)))
    end
    return expCache[level]
end
```

## สรุป

ModuleScript คือรากฐานของโค้ดที่ดีใน Roblox:
- **Code Reuse** - เขียนครั้งเดียว ใช้หลายที่
- **Separation of Concerns** - แยกความรับผิดชอบ
- **Maintainability** - ดูแลง่าย แก้ไขง่าย
- **Testability** - ทดสอบง่าย

หลักการสำคัญ: **"Every script should do one thing well"**
