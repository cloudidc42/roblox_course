# ตอนที่ 23: ReplicatedStorage ใน Roblox

## บทนำ

ReplicatedStorage เป็น container พิเศษที่ข้อมูลถูก "replicate" (ส่งสำเนา) จาก Server ไปยัง Client ทุกคนโดยอัตโนมัติ ทำให้ทั้ง Server Scripts และ LocalScripts สามารถเข้าถึงข้อมูลใน ReplicatedStorage ได้

---

## 23.1 ทำไมต้องใช้ ReplicatedStorage?

```
Server (Script)            Client (LocalScript)
     |                           |
     | --- ReplicatedStorage --- |
     |   ข้อมูลถูก sync         |
     | RemoteEvents              |
     | RemoteFunctions           |
     | ModuleScripts             |
     | Templates (Models, Tools) |
```

ReplicatedStorage ใช้สำหรับ:
1. **RemoteEvents/RemoteFunctions** - สื่อสาร Server <-> Client
2. **ModuleScripts** - โค้ดที่ใช้ร่วมกัน
3. **Templates** - ต้นแบบสำหรับ clone
4. **Shared Data** - ข้อมูลที่ทุกคนต้องรู้

---

## 23.2 การใช้งานพื้นฐาน

```lua
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- เพิ่ม object ใน ReplicatedStorage (Server)
local function setupReplicatedStorage()
    -- สร้าง folder จัดระเบียบ
    local events = Instance.new("Folder")
    events.Name = "Events"
    events.Parent = ReplicatedStorage
    
    local modules = Instance.new("Folder")
    modules.Name = "Modules"
    modules.Parent = ReplicatedStorage
    
    local templates = Instance.new("Folder")
    templates.Name = "Templates"
    templates.Parent = ReplicatedStorage
    
    -- สร้าง RemoteEvents
    local combatEvent = Instance.new("RemoteEvent")
    combatEvent.Name = "CombatEvent"
    combatEvent.Parent = events
    
    local shopEvent = Instance.new("RemoteEvent")
    shopEvent.Name = "ShopEvent"
    shopEvent.Parent = events
    
    print("Setup ReplicatedStorage เสร็จแล้ว")
end

-- เข้าถึงจาก Client
local events = ReplicatedStorage:WaitForChild("Events")
local combatEvent = events:WaitForChild("CombatEvent")
```

---

## 23.3 ModuleScripts ใน ReplicatedStorage

ModuleScripts เป็นวิธีที่ดีในการแชร์โค้ดระหว่าง Server และ Client:

```lua
-- ModuleScript: GameConfig (ใน ReplicatedStorage/Modules)
local GameConfig = {}

GameConfig.MAX_PLAYERS = 10
GameConfig.GAME_TIME = 300        -- วินาที
GameConfig.RESPAWN_TIME = 5       -- วินาที
GameConfig.MAX_LEVEL = 100
GameConfig.BASE_HEALTH = 100
GameConfig.BASE_SPEED = 16
GameConfig.BASE_JUMP_POWER = 50

GameConfig.ITEM_COSTS = {
    ["ดาบ"] = 100,
    ["โล่"] = 80,
    ["ยาแดง"] = 30,
    ["เกราะ"] = 150,
    ["ธนู"] = 120
}

GameConfig.LEVEL_EXP = function(level)
    return math.floor(100 * (level ^ 1.5))
end

GameConfig.TEAM_COLORS = {
    ["ทีมแดง"] = Color3.fromRGB(255, 50, 50),
    ["ทีมน้ำเงิน"] = Color3.fromRGB(50, 50, 255),
    ["ทีมเขียว"] = Color3.fromRGB(50, 200, 50)
}

return GameConfig
```

```lua
-- ใช้งาน ModuleScript จาก Server
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local GameConfig = require(ReplicatedStorage.Modules.GameConfig)

print("จำนวนผู้เล่นสูงสุด: " .. GameConfig.MAX_PLAYERS)
print("EXP สำหรับ Level 10: " .. GameConfig.LEVEL_EXP(10))

-- ใช้งานจาก Client (LocalScript) - เหมือนกัน
local GameConfig = require(game.ReplicatedStorage.Modules.GameConfig)
print("Max Players: " .. GameConfig.MAX_PLAYERS)
```

---

## 23.4 Shared Utility Modules

```lua
-- ModuleScript: Utils (ใน ReplicatedStorage/Modules)
local Utils = {}

-- String utilities
function Utils.trim(str)
    return str:match("^%s*(.-)%s*$")
end

function Utils.split(str, sep)
    local parts = {}
    for part in str:gmatch("[^" .. (sep or "%s") .. "]+") do
        table.insert(parts, part)
    end
    return parts
end

-- Math utilities
function Utils.clamp(value, min, max)
    return math.max(min, math.min(max, value))
end

function Utils.lerp(a, b, t)
    return a + (b - a) * t
end

function Utils.round(n, decimals)
    local factor = 10 ^ (decimals or 0)
    return math.floor(n * factor + 0.5) / factor
end

function Utils.formatNumber(n)
    if n >= 1e9 then
        return string.format("%.1fB", n / 1e9)
    elseif n >= 1e6 then
        return string.format("%.1fM", n / 1e6)
    elseif n >= 1e3 then
        return string.format("%.1fK", n / 1e3)
    end
    return tostring(n)
end

-- Table utilities
function Utils.deepCopy(t)
    if type(t) ~= "table" then return t end
    local copy = {}
    for k, v in pairs(t) do
        copy[k] = Utils.deepCopy(v)
    end
    return setmetatable(copy, getmetatable(t))
end

function Utils.contains(arr, value)
    for _, v in ipairs(arr) do
        if v == value then return true end
    end
    return false
end

function Utils.filter(arr, predicate)
    local result = {}
    for _, v in ipairs(arr) do
        if predicate(v) then
            table.insert(result, v)
        end
    end
    return result
end

function Utils.map(arr, transform)
    local result = {}
    for i, v in ipairs(arr) do
        result[i] = transform(v)
    end
    return result
end

function Utils.reduce(arr, func, initial)
    local acc = initial
    for _, v in ipairs(arr) do
        acc = func(acc, v)
    end
    return acc
end

-- Time utilities
function Utils.formatTime(seconds)
    local h = math.floor(seconds / 3600)
    local m = math.floor((seconds % 3600) / 60)
    local s = seconds % 60
    
    if h > 0 then
        return string.format("%d:%02d:%02d", h, m, s)
    else
        return string.format("%02d:%02d", m, s)
    end
end

return Utils
```

---

## 23.5 Template System

```lua
-- Server Script: TemplateManager.lua
-- สร้างและจัดการ templates ใน ReplicatedStorage

local ReplicatedStorage = game:GetService("ReplicatedStorage")

local function setupTemplates()
    local templates = ReplicatedStorage:FindFirstChild("Templates")
    if not templates then
        templates = Instance.new("Folder")
        templates.Name = "Templates"
        templates.Parent = ReplicatedStorage
    end
    
    -- Enemy template
    local enemyTemplate = Instance.new("Model")
    enemyTemplate.Name = "Enemy"
    
    local body = Instance.new("Part")
    body.Name = "HumanoidRootPart"
    body.Size = Vector3.new(2, 2, 1)
    body.Anchored = false
    body.Parent = enemyTemplate
    
    local humanoid = Instance.new("Humanoid")
    humanoid.MaxHealth = 100
    humanoid.Health = 100
    humanoid.Parent = enemyTemplate
    
    enemyTemplate.PrimaryPart = body
    enemyTemplate.Parent = templates
    
    return templates
end

-- Clone template สำหรับใช้งาน
local function spawnFromTemplate(templateName, position)
    local templates = ReplicatedStorage:FindFirstChild("Templates")
    if not templates then return nil end
    
    local template = templates:FindFirstChild(templateName)
    if not template then
        warn("ไม่พบ template: " .. templateName)
        return nil
    end
    
    local clone = template:Clone()
    
    if clone:IsA("Model") and clone.PrimaryPart then
        clone:SetPrimaryPartCFrame(CFrame.new(position))
    end
    
    clone.Parent = workspace
    return clone
end
```

---

## 23.6 ระบบ Event Management

```lua
-- Server Script: EventManager.lua
-- จัดการ Remote Events อย่างเป็นระบบ

local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")

local EventManager = {}
local events = {}

-- สร้าง events folder
local eventsFolder = ReplicatedStorage:FindFirstChild("Events")
if not eventsFolder then
    eventsFolder = Instance.new("Folder")
    eventsFolder.Name = "Events"
    eventsFolder.Parent = ReplicatedStorage
end

-- สร้างหรือดึง RemoteEvent
local function getOrCreateEvent(name)
    local event = eventsFolder:FindFirstChild(name)
    if not event then
        event = Instance.new("RemoteEvent")
        event.Name = name
        event.Parent = eventsFolder
    end
    return event
end

-- Register events
local eventNames = {
    "CombatEvent",
    "ShopEvent",
    "QuestEvent",
    "ChatEvent",
    "UIEvent",
    "PlayerDataEvent",
    "NotificationEvent"
}

for _, name in ipairs(eventNames) do
    events[name] = getOrCreateEvent(name)
end

-- Helper functions
function EventManager.fireToPlayer(eventName, player, ...)
    local event = events[eventName]
    if event then
        event:FireClient(player, ...)
    end
end

function EventManager.fireToAll(eventName, ...)
    local event = events[eventName]
    if event then
        event:FireAllClients(...)
    end
end

function EventManager.fireToTeam(eventName, team, ...)
    for _, player in ipairs(Players:GetPlayers()) do
        if player.Team == team then
            EventManager.fireToPlayer(eventName, player, ...)
        end
    end
end

function EventManager.onServer(eventName, callback)
    local event = events[eventName]
    if event then
        event.OnServerEvent:Connect(callback)
    end
end

-- Notification system
function EventManager.notify(player, message, notifType, duration)
    EventManager.fireToPlayer("NotificationEvent", player, {
        message = message,
        type = notifType or "info",
        duration = duration or 3
    })
end

function EventManager.notifyAll(message, notifType, duration)
    EventManager.fireToAll("NotificationEvent", {
        message = message,
        type = notifType or "info",
        duration = duration or 3
    })
end

-- ตัวอย่างการใช้งาน
EventManager.onServer("CombatEvent", function(player, action, targetId, data)
    print(player.Name .. " ทำ: " .. action)
    
    if action == "attack" then
        local target = Players:GetPlayerByUserId(targetId)
        if target then
            -- คำนวณและ apply damage
            local damage = data.damage or 10
            EventManager.notify(target, "คุณถูกโจมตีจาก " .. player.Name .. " -" .. damage .. " HP", "warning")
        end
    end
end)

return EventManager
```

---

## 23.7 Shared Game Data

```lua
-- ModuleScript: SharedGameData
-- ข้อมูลที่ใช้ร่วมกันระหว่าง Server และ Client

local SharedGameData = {}

-- Item Database
SharedGameData.Items = {
    sword = {
        id = "sword",
        name = "ดาบเหล็ก",
        type = "weapon",
        damage = 15,
        speed = -2,     -- ลดความเร็ว
        price = 100,
        description = "ดาบมาตรฐานของนักรบ",
        icon = "rbxassetid://0",
        stackable = false,
        maxStack = 1
    },
    shield = {
        id = "shield",
        name = "โล่ไม้",
        type = "armor",
        defense = 10,
        price = 80,
        description = "โล่ที่ทำจากไม้แข็ง",
        icon = "rbxassetid://0",
        stackable = false,
        maxStack = 1
    },
    health_potion = {
        id = "health_potion",
        name = "ยาพื้นฐาน",
        type = "consumable",
        healAmount = 50,
        price = 30,
        description = "ยาฟื้นฟูเลือดขั้นพื้นฐาน",
        icon = "rbxassetid://0",
        stackable = true,
        maxStack = 99
    },
    bow = {
        id = "bow",
        name = "ธนูไม้",
        type = "rangedWeapon",
        damage = 12,
        range = 50,
        price = 120,
        description = "ธนูสำหรับโจมตีระยะไกล",
        icon = "rbxassetid://0",
        stackable = false,
        maxStack = 1
    }
}

-- Enemy Database
SharedGameData.Enemies = {
    goblin = {
        id = "goblin",
        name = "ก็อบลิน",
        health = 50,
        damage = 8,
        speed = 14,
        exp = 20,
        gold = math.random(5, 15),
        drops = {
            {item = "health_potion", chance = 0.3, count = 1}
        }
    },
    orc = {
        id = "orc",
        name = "ออร์ค",
        health = 150,
        damage = 20,
        speed = 10,
        exp = 50,
        gold = math.random(20, 40),
        drops = {
            {item = "sword", chance = 0.1, count = 1},
            {item = "health_potion", chance = 0.5, count = 2}
        }
    },
    dragon = {
        id = "dragon",
        name = "มังกรไฟ (Boss)",
        health = 5000,
        damage = 100,
        speed = 12,
        exp = 1000,
        gold = math.random(200, 500),
        drops = {
            {item = "sword", chance = 0.8, count = 1},
            {item = "shield", chance = 0.6, count = 1},
            {item = "health_potion", chance = 1.0, count = 5}
        }
    }
}

-- Skill Database
SharedGameData.Skills = {
    fireball = {
        id = "fireball",
        name = "ลูกไฟ",
        damage = 80,
        manaCost = 30,
        cooldown = 5,
        range = 30,
        aoeRadius = 5,
        description = "ยิงลูกไฟไปยังเป้าหมาย"
    },
    heal = {
        id = "heal",
        name = "รักษา",
        healAmount = 100,
        manaCost = 25,
        cooldown = 8,
        description = "ฟื้นฟูเลือดของตัวเอง"
    }
}

-- Helper functions
function SharedGameData.getItem(id)
    return SharedGameData.Items[id]
end

function SharedGameData.getEnemy(id)
    return SharedGameData.Enemies[id]
end

function SharedGameData.getSkill(id)
    return SharedGameData.Skills[id]
end

function SharedGameData.getItemsByType(itemType)
    local result = {}
    for id, item in pairs(SharedGameData.Items) do
        if item.type == itemType then
            table.insert(result, item)
        end
    end
    return result
end

return SharedGameData
```

---

## 23.8 แบบฝึกหัด

### แบบฝึกหัดที่ 1: Module System

```lua
-- สร้าง ModuleScript ชื่อ "PlayerUtils" ที่มีฟังก์ชัน:
-- - getPlayerLevel(player): return level จาก leaderstats
-- - getPlayerScore(player): return score จาก leaderstats
-- - isPlayerAlive(player): ตรวจสอบว่ามีชีวิต
-- - getPlayerTeamName(player): return ชื่อทีม

-- ใน ReplicatedStorage/Modules/PlayerUtils
local PlayerUtils = {}

function PlayerUtils.getPlayerLevel(player)
    -- เติมโค้ด
end

function PlayerUtils.isPlayerAlive(player)
    -- เติมโค้ด
end

return PlayerUtils
```

### แบบฝึกหัดที่ 2: Item System

```lua
-- ใช้ SharedGameData Module สร้างระบบ shop:
-- - แสดงรายการสินค้า
-- - ซื้อสินค้า (ตรวจสอบเงิน)
-- - เพิ่มใน inventory

local ReplicatedStorage = game:GetService("ReplicatedStorage")
local SharedGameData = require(ReplicatedStorage.Modules.SharedGameData)

-- เติมโค้ดระบบ shop
```

---

## สรุป

| ใช้สำหรับ | ประเภท |
|----------|--------|
| Client-Server communication | RemoteEvent, RemoteFunction |
| Shared code | ModuleScript |
| Templates | Models, Tools, Parts |
| Shared config | ModuleScript with data |

### ข้อควรจำ

1. ReplicatedStorage ถูก replicate ไปยัง Client ทุกคน
2. อย่าเก็บข้อมูลที่ควรเป็น Server-only ใน ReplicatedStorage
3. ใช้ `WaitForChild()` ใน Client เพื่อรอข้อมูล
4. ModuleScripts ใน ReplicatedStorage ใช้ได้ทั้ง Server และ Client

### บทถัดไป

ในบทที่ 24 เราจะเรียนเรื่อง **ServerScriptService** - การใช้งานและจัดการ server scripts
