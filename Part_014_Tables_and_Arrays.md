# ตอนที่ 14: Tables และ Arrays ใน Lua

## บทนำ

Table คือโครงสร้างข้อมูลที่สำคัญที่สุดใน Lua เป็นโครงสร้างข้อมูลประเภทเดียวที่ Lua มีให้ แต่มีความยืดหยุ่นมากเพราะสามารถทำหน้าที่ได้ทั้ง Array, Dictionary, Set, Object, และอื่นๆ การเข้าใจ Table อย่างลึกซึ้งจะทำให้คุณเขียนโค้ด Roblox ได้อย่างมีประสิทธิภาพ

---

## 14.1 พื้นฐาน Table

### การสร้าง Table

```lua
-- Table ว่างเปล่า
local emptyTable = {}

-- Array style (sequential)
local fruits = {"แอปเปิ้ล", "กล้วย", "ส้ม", "มะม่วง"}

-- Dictionary style (key-value)
local player = {
    name = "สมชาย",
    level = 10,
    health = 100
}

-- Mixed (ทั้ง array และ dictionary)
local mixed = {
    "ค่าที่ 1",    -- index 1
    "ค่าที่ 2",    -- index 2
    name = "Table",
    count = 5
}
```

### การเข้าถึงข้อมูล

```lua
-- Array access ด้วย index (เริ่มที่ 1)
local fruits = {"แอปเปิ้ล", "กล้วย", "ส้ม"}
print(fruits[1])  -- แอปเปิ้ล
print(fruits[2])  -- กล้วย
print(fruits[3])  -- ส้ม
print(fruits[4])  -- nil (ไม่มีข้อมูล)

-- Dictionary access
local player = {name = "สมชาย", level = 10}
print(player.name)      -- สมชาย (dot notation)
print(player["level"])  -- 10 (bracket notation)

-- ความแตกต่าง: bracket notation ใช้กับ dynamic key ได้
local key = "name"
print(player[key])  -- สมชาย
```

### การแก้ไขข้อมูล

```lua
local player = {
    name = "สมชาย",
    level = 1,
    health = 100
}

-- แก้ไขค่า
player.level = 5
player.health = 80
player["name"] = "สมชาย ใจดี"

-- เพิ่มค่าใหม่
player.mana = 50
player.isAlive = true

-- ลบค่า
player.isAlive = nil  -- ลบโดยกำหนดเป็น nil

print(player.level)   -- 5
print(player.mana)    -- 50
print(player.isAlive) -- nil
```

---

## 14.2 Array Operations

### table.insert และ table.remove

```lua
local items = {"ดาบ", "โล่", "ธนู"}

-- เพิ่มท้าย
table.insert(items, "ไม้กายสิทธิ์")
-- items = {"ดาบ", "โล่", "ธนู", "ไม้กายสิทธิ์"}

-- เพิ่มที่ตำแหน่งที่กำหนด
table.insert(items, 2, "หมวก")
-- items = {"ดาบ", "หมวก", "โล่", "ธนู", "ไม้กายสิทธิ์"}

-- ลบท้าย (return ค่าที่ลบ)
local removed = table.remove(items)
print(removed)  -- ไม้กายสิทธิ์

-- ลบที่ตำแหน่งที่กำหนด
local removed2 = table.remove(items, 2)
print(removed2)  -- หมวก
-- items = {"ดาบ", "โล่", "ธนู"}

-- ดูจำนวนสมาชิก
print(#items)  -- 3
```

### table.sort

```lua
-- เรียงลำดับ array
local numbers = {5, 2, 8, 1, 9, 3, 7, 4, 6}
table.sort(numbers)  -- ascending
for _, v in ipairs(numbers) do io.write(v .. " ") end
-- 1 2 3 4 5 6 7 8 9

-- เรียงลำดับ descending
table.sort(numbers, function(a, b) return a > b end)
for _, v in ipairs(numbers) do io.write(v .. " ") end
-- 9 8 7 6 5 4 3 2 1

-- เรียงลำดับ objects
local players = {
    {name = "สมชาย", score = 350},
    {name = "สมหญิง", score = 520},
    {name = "สมศักดิ์", score = 280},
    {name = "สมใจ", score = 450}
}

-- เรียงตามคะแนน (มากไปน้อย)
table.sort(players, function(a, b)
    return a.score > b.score
end)

print("อันดับ:")
for i, p in ipairs(players) do
    print(i .. ". " .. p.name .. " - " .. p.score)
end

-- เรียงตามชื่อ (a-z)
table.sort(players, function(a, b)
    return a.name < b.name
end)
```

### table.concat

```lua
-- รวม array เป็น string
local words = {"สวัสดี", "โลก", "ยินดีต้อนรับ"}
local sentence = table.concat(words, " ")
print(sentence)  -- สวัสดี โลก ยินดีต้อนรับ

local csv = table.concat(words, ",")
print(csv)  -- สวัสดี,โลก,ยินดีต้อนรับ

-- รวมเฉพาะบางส่วน
local nums = {1, 2, 3, 4, 5}
print(table.concat(nums, "-", 2, 4))  -- 2-3-4
```

---

## 14.3 Nested Tables

```lua
-- Table ซ้อน Table
local gameData = {
    players = {
        {name = "Player1", score = 100},
        {name = "Player2", score = 200},
        {name = "Player3", score = 150}
    },
    settings = {
        maxPlayers = 10,
        gameTime = 300,
        map = "Forest"
    },
    stats = {
        totalKills = 0,
        totalDeaths = 0,
        gamesPlayed = 0
    }
}

-- การเข้าถึง nested values
print(gameData.players[1].name)   -- Player1
print(gameData.settings.maxPlayers) -- 10
print(gameData.stats.totalKills)   -- 0

-- การแก้ไข
gameData.players[1].score = 150
gameData.settings.map = "Desert"
gameData.stats.totalKills = gameData.stats.totalKills + 5

-- เพิ่มผู้เล่นใหม่
table.insert(gameData.players, {name = "Player4", score = 75})
print("จำนวนผู้เล่น: " .. #gameData.players)  -- 4
```

---

## 14.4 Table ในรูปแบบ Dictionary

```lua
-- Dictionary สำหรับเก็บข้อมูล
local inventory = {}

-- เพิ่ม items
inventory["ดาบ"] = {count = 1, damage = 15}
inventory["ยาแดง"] = {count = 5, heal = 50}
inventory["โล่"] = {count = 1, defense = 10}

-- ตรวจสอบและใช้งาน
if inventory["ยาแดง"] then
    local potion = inventory["ยาแดง"]
    if potion.count > 0 then
        potion.count = potion.count - 1
        print("ใช้ยาแดง! เหลือ: " .. potion.count)
    end
end

-- วน loop ผ่าน dictionary
print("\nรายการสิ่งของ:")
for itemName, itemData in pairs(inventory) do
    print("- " .. itemName .. " x" .. itemData.count)
end

-- ลบ item
inventory["โล่"] = nil

-- นับจำนวน items ใน dictionary
local function countDict(dict)
    local count = 0
    for _ in pairs(dict) do
        count = count + 1
    end
    return count
end

print("\nจำนวนชนิด item: " .. countDict(inventory))
```

---

## 14.5 Table เป็น Set

```lua
-- Set: เก็บค่าที่ไม่ซ้ำกัน
local function createSet(list)
    local set = {}
    for _, v in ipairs(list) do
        set[v] = true
    end
    return set
end

local function setContains(set, value)
    return set[value] == true
end

local function setAdd(set, value)
    set[value] = true
end

local function setRemove(set, value)
    set[value] = nil
end

local function setToArray(set)
    local arr = {}
    for k in pairs(set) do
        table.insert(arr, k)
    end
    return arr
end

-- ใช้งาน
local bannedPlayers = createSet({"cheater1", "spammer", "hacker99"})

print(setContains(bannedPlayers, "cheater1"))   -- true
print(setContains(bannedPlayers, "goodplayer")) -- false

setAdd(bannedPlayers, "newcheater")
setRemove(bannedPlayers, "spammer")

local bannedList = setToArray(bannedPlayers)
print("ผู้ถูก ban:")
for _, name in ipairs(bannedList) do
    print("- " .. name)
end

-- Set Operations
local function setUnion(set1, set2)
    local result = {}
    for k in pairs(set1) do result[k] = true end
    for k in pairs(set2) do result[k] = true end
    return result
end

local function setIntersection(set1, set2)
    local result = {}
    for k in pairs(set1) do
        if set2[k] then result[k] = true end
    end
    return result
end

local admins = createSet({"admin1", "admin2", "mod1"})
local online = createSet({"admin1", "player1", "player2", "mod1"})

local onlineAdmins = setIntersection(admins, online)
print("\nAdmin ที่ online:")
for name in pairs(onlineAdmins) do
    print("- " .. name)
end
```

---

## 14.6 Metatables

Metatables ช่วยให้เราปรับแต่งพฤติกรรมของ table

```lua
-- __index: กำหนด default values
local defaults = {
    health = 100,
    mana = 50,
    speed = 16,
    level = 1
}

local player = setmetatable({
    name = "สมชาย",
    health = 75  -- override default
}, {
    __index = defaults  -- ถ้าหาค่าไม่เจอใน player ให้ไปหาใน defaults
})

print(player.name)    -- สมชาย (อยู่ใน player)
print(player.health)  -- 75 (override default)
print(player.mana)    -- 50 (จาก defaults)
print(player.level)   -- 1 (จาก defaults)

-- __newindex: intercept การ assign
local readOnly = setmetatable({}, {
    __newindex = function(table, key, value)
        error("ไม่สามารถแก้ไขค่าได้! Key: " .. tostring(key))
    end,
    __index = {
        x = 10,
        y = 20
    }
})

print(readOnly.x)  -- 10
-- readOnly.x = 20  -- ERROR!

-- __tostring: กำหนดวิธีแปลงเป็น string
local Point = {}
Point.__index = Point
Point.__tostring = function(self)
    return string.format("Point(%d, %d)", self.x, self.y)
end

function Point.new(x, y)
    return setmetatable({x = x, y = y}, Point)
end

local p = Point.new(3, 4)
print(tostring(p))  -- Point(3, 4)

-- __add: operator overloading
Point.__add = function(a, b)
    return Point.new(a.x + b.x, a.y + b.y)
end

Point.__eq = function(a, b)
    return a.x == b.x and a.y == b.y
end

local p1 = Point.new(1, 2)
local p2 = Point.new(3, 4)
local p3 = p1 + p2
print(tostring(p3))  -- Point(4, 6)
print(p1 == Point.new(1, 2))  -- true
```

---

## 14.7 Object-Oriented Programming ด้วย Tables

```lua
-- Class ด้วย Table และ Metatable
local Character = {}
Character.__index = Character

-- Constructor
function Character.new(name, className)
    local self = setmetatable({}, Character)
    self.name = name
    self.className = className or "นักรบ"
    self.level = 1
    self.experience = 0
    self.health = 100
    self.maxHealth = 100
    self.mana = 50
    self.maxMana = 50
    self.attackPower = 10
    self.defense = 5
    self.skills = {}
    return self
end

-- Methods
function Character:gainExp(amount)
    self.experience = self.experience + amount
    print(self.name .. " ได้รับ " .. amount .. " EXP")
    
    -- ตรวจสอบ level up
    local expNeeded = self.level * 100
    while self.experience >= expNeeded do
        self.experience = self.experience - expNeeded
        self:levelUp()
        expNeeded = self.level * 100
    end
end

function Character:levelUp()
    self.level = self.level + 1
    
    -- เพิ่มสถิติตาม class
    if self.className == "นักรบ" then
        self.maxHealth = self.maxHealth + 20
        self.attackPower = self.attackPower + 3
        self.defense = self.defense + 2
    elseif self.className == "นักเวทย์" then
        self.maxMana = self.maxMana + 20
        self.attackPower = self.attackPower + 5
        self.maxHealth = self.maxHealth + 5
    elseif self.className == "นักธนู" then
        self.attackPower = self.attackPower + 4
        self.maxHealth = self.maxHealth + 10
    end
    
    -- ฟื้นฟูสมบูรณ์เมื่อ level up
    self.health = self.maxHealth
    self.mana = self.maxMana
    
    print("🎉 " .. self.name .. " เลเวลอัพ! เลเวล " .. self.level)
end

function Character:attack(target)
    local damage = self.attackPower - (target.defense * 0.3)
    damage = math.max(1, math.floor(damage))
    
    target:takeDamage(damage)
    return damage
end

function Character:takeDamage(damage)
    self.health = math.max(0, self.health - damage)
    print(self.name .. " รับความเสียหาย " .. damage .. " เลือดเหลือ: " .. self.health .. "/" .. self.maxHealth)
    
    if self.health <= 0 then
        self:onDeath()
    end
end

function Character:heal(amount)
    local oldHealth = self.health
    self.health = math.min(self.maxHealth, self.health + amount)
    local healed = self.health - oldHealth
    print(self.name .. " ฟื้นฟู " .. healed .. " เลือดปัจจุบัน: " .. self.health)
end

function Character:onDeath()
    print(self.name .. " ถูกสังหาร!")
end

function Character:learnSkill(skillName)
    table.insert(self.skills, skillName)
    print(self.name .. " เรียนรู้สกิล: " .. skillName)
end

function Character:getStatus()
    return string.format(
        "[%s - %s] Lv.%d HP:%d/%d MP:%d/%d ATK:%d DEF:%d",
        self.name, self.className, self.level,
        self.health, self.maxHealth,
        self.mana, self.maxMana,
        self.attackPower, self.defense
    )
end

-- Inheritance: Warrior extends Character
local Warrior = setmetatable({}, {__index = Character})
Warrior.__index = Warrior

function Warrior.new(name)
    local self = Character.new(name, "นักรบ")
    setmetatable(self, Warrior)
    self.rage = 0  -- attribute พิเศษของ Warrior
    self.maxRage = 100
    return self
end

function Warrior:berserkerRage()
    if self.rage >= 50 then
        self.rage = self.rage - 50
        local tempAttack = self.attackPower * 2
        print(self.name .. " ใช้ Berserker Rage! ATK: " .. self.attackPower .. " -> " .. tempAttack)
        return tempAttack
    else
        print("Rage ไม่พอ! (" .. self.rage .. "/50)")
        return self.attackPower
    end
end

function Warrior:attack(target)
    -- Override: เพิ่ม rage เมื่อโจมตี
    local damage = Character.attack(self, target)  -- เรียก parent method
    self.rage = math.min(self.maxRage, self.rage + 10)
    print(self.name .. " Rage: " .. self.rage .. "/" .. self.maxRage)
    return damage
end

-- ทดสอบ
local hero = Warrior.new("สมชาย")
local enemy = Character.new("Goblin", "ศัตรู")
enemy.maxHealth = 50
enemy.health = 50
enemy.defense = 2

print(hero:getStatus())
print(enemy:getStatus())
print()

hero:attack(enemy)
hero:attack(enemy)
hero:attack(enemy)
hero:berserkerRage()
hero:gainExp(150)
print(hero:getStatus())
```

---

## 14.8 การทำงานกับ Tables ใน Roblox

### การเก็บข้อมูลผู้เล่น

```lua
-- Script: PlayerDataManager.lua

local PlayerDataManager = {}

-- เก็บข้อมูลผู้เล่นทั้งหมด
local playerData = {}

-- สร้างข้อมูลเริ่มต้น
local function createDefaultData(player)
    return {
        userId = player.UserId,
        name = player.Name,
        
        -- Stats
        level = 1,
        experience = 0,
        gold = 100,
        
        -- Character
        health = 100,
        maxHealth = 100,
        
        -- Inventory
        inventory = {},
        equippedWeapon = nil,
        
        -- Statistics
        kills = 0,
        deaths = 0,
        playtime = 0,
        
        -- Settings
        settings = {
            musicVolume = 0.5,
            sfxVolume = 1.0,
            showDamageNumbers = true
        },
        
        -- Timestamps
        firstJoined = os.time(),
        lastLogin = os.time()
    }
end

function PlayerDataManager.initPlayer(player)
    playerData[player.UserId] = createDefaultData(player)
    print("สร้างข้อมูลสำหรับ " .. player.Name)
    return playerData[player.UserId]
end

function PlayerDataManager.getPlayerData(player)
    return playerData[player.UserId]
end

function PlayerDataManager.removePlayer(player)
    playerData[player.UserId] = nil
end

function PlayerDataManager.addItem(player, itemName, quantity)
    local data = playerData[player.UserId]
    if not data then return false end
    
    quantity = quantity or 1
    
    if data.inventory[itemName] then
        data.inventory[itemName] = data.inventory[itemName] + quantity
    else
        data.inventory[itemName] = quantity
    end
    
    print(player.Name .. " ได้รับ " .. itemName .. " x" .. quantity)
    return true
end

function PlayerDataManager.removeItem(player, itemName, quantity)
    local data = playerData[player.UserId]
    if not data then return false end
    
    quantity = quantity or 1
    
    if not data.inventory[itemName] or data.inventory[itemName] < quantity then
        print("ไม่มี " .. itemName .. " เพียงพอ")
        return false
    end
    
    data.inventory[itemName] = data.inventory[itemName] - quantity
    
    if data.inventory[itemName] <= 0 then
        data.inventory[itemName] = nil  -- ลบออกจาก inventory
    end
    
    return true
end

function PlayerDataManager.getTopPlayers(count)
    local sorted = {}
    
    for _, data in pairs(playerData) do
        table.insert(sorted, data)
    end
    
    table.sort(sorted, function(a, b)
        return a.level > b.level or 
               (a.level == b.level and a.experience > b.experience)
    end)
    
    local result = {}
    for i = 1, math.min(count, #sorted) do
        table.insert(result, sorted[i])
    end
    
    return result
end

-- ทดสอบ
local mockPlayer = {
    UserId = 12345,
    Name = "TestPlayer"
}

local data = PlayerDataManager.initPlayer(mockPlayer)
PlayerDataManager.addItem(mockPlayer, "ยาแดง", 5)
PlayerDataManager.addItem(mockPlayer, "ดาบ", 1)
PlayerDataManager.addItem(mockPlayer, "ยาแดง", 3)

print("\nInventory:")
for item, count in pairs(data.inventory) do
    print("- " .. item .. " x" .. count)
end

PlayerDataManager.removeItem(mockPlayer, "ยาแดง", 2)
print("\nหลังใช้ยา 2 ขวด:")
for item, count in pairs(data.inventory) do
    print("- " .. item .. " x" .. count)
end
```

---

## 14.9 Pattern สำคัญสำหรับ Tables

### Observer Pattern

```lua
-- Observer Pattern ด้วย Tables
local EventEmitter = {}
EventEmitter.__index = EventEmitter

function EventEmitter.new()
    return setmetatable({events = {}}, EventEmitter)
end

function EventEmitter:on(event, callback)
    if not self.events[event] then
        self.events[event] = {}
    end
    
    local id = #self.events[event] + 1
    self.events[event][id] = callback
    
    -- Return disconnect function
    return function()
        if self.events[event] then
            self.events[event][id] = nil
        end
    end
end

function EventEmitter:emit(event, ...)
    if not self.events[event] then return end
    
    for _, callback in pairs(self.events[event]) do
        if callback then
            callback(...)
        end
    end
end

-- ใช้งาน
local gameEvents = EventEmitter.new()

-- Subscribe
local disconnect1 = gameEvents:on("roundStart", function()
    print("Round เริ่มแล้ว! เตรียมตัว!")
end)

gameEvents:on("roundStart", function()
    print("Timer เริ่มนับ...")
end)

gameEvents:on("playerKill", function(killer, victim)
    print(killer .. " สังหาร " .. victim)
end)

-- Emit
gameEvents:emit("roundStart")
gameEvents:emit("playerKill", "สมชาย", "ศัตรู1")

disconnect1()  -- ยกเลิก subscription แรก
gameEvents:emit("roundStart")  -- แสดงเฉพาะ Timer
```

### Cache Pattern

```lua
-- Caching ผลลัพธ์การคำนวณด้วย Table
local cache = {}

local function expensiveOperation(key)
    if cache[key] then
        print("Cache hit: " .. key)
        return cache[key]
    end
    
    print("Cache miss: " .. key .. " (กำลังคำนวณ...)")
    -- จำลองการคำนวณที่ใช้เวลา
    local result = 0
    for i = 1, 1000000 do
        result = result + i
    end
    
    cache[key] = result
    return result
end

print(expensiveOperation("calc1"))
print(expensiveOperation("calc1"))  -- จาก cache
print(expensiveOperation("calc2"))
```

---

## 14.10 แบบฝึกหัด

### แบบฝึกหัดที่ 1: Inventory System

```lua
-- สร้าง Inventory System ที่มี:
-- - เพิ่ม/ลบ items
-- - ตรวจสอบจำนวน
-- - แสดงรายการทั้งหมด
-- - คำนวณมูลค่ารวม

local itemValues = {
    ["ดาบ"] = 100,
    ["โล่"] = 80,
    ["ยาแดง"] = 30,
    ["เกราะ"] = 150,
    ["ธนู"] = 120
}

local Inventory = {}
Inventory.__index = Inventory

function Inventory.new()
    return setmetatable({items = {}, capacity = 20}, Inventory)
end

function Inventory:addItem(name, count)
    -- เติมโค้ด
end

function Inventory:removeItem(name, count)
    -- เติมโค้ด
end

function Inventory:getCount(name)
    -- เติมโค้ด
end

function Inventory:display()
    -- เติมโค้ด
end

function Inventory:getTotalValue()
    -- เติมโค้ด
end

-- ทดสอบ
local inv = Inventory.new()
inv:addItem("ดาบ", 1)
inv:addItem("ยาแดง", 5)
inv:addItem("โล่", 1)
inv:display()
print("มูลค่ารวม: " .. inv:getTotalValue())
```

### แบบฝึกหัดที่ 2: Leaderboard

```lua
-- สร้าง Leaderboard ที่:
-- - เพิ่มผู้เล่นพร้อมคะแนน
-- - อัปเดตคะแนน
-- - แสดง Top N ผู้เล่น
-- - หาอันดับของผู้เล่น

local Leaderboard = {}

-- เติมโค้ด

-- ทดสอบ
-- Leaderboard.addPlayer("สมชาย", 350)
-- Leaderboard.addPlayer("สมหญิง", 520)
-- Leaderboard.updateScore("สมชาย", 400)
-- Leaderboard.displayTop(5)
-- print("อันดับสมชาย: " .. Leaderboard.getRank("สมชาย"))
```

### แบบฝึกหัดที่ 3: Quest Log

```lua
-- สร้าง Quest Log ที่:
-- - เพิ่ม quest
-- - อัปเดต progress
-- - ตรวจสอบว่าเสร็จหรือยัง
-- - แสดงรายการ quest ทั้งหมด

-- เติมโค้ด
```

---

## สรุป

| การใช้งาน | รูปแบบ |
|----------|--------|
| Array | `{v1, v2, v3}` เข้าถึงด้วย `t[1]` |
| Dictionary | `{k1=v1, k2=v2}` เข้าถึงด้วย `t.k1` |
| เพิ่ม element | `table.insert(t, v)` หรือ `t[k] = v` |
| ลบ element | `table.remove(t, i)` หรือ `t[k] = nil` |
| เรียงลำดับ | `table.sort(t, compareFn)` |
| รวม string | `table.concat(t, sep)` |
| วนซ้ำ array | `for i, v in ipairs(t)` |
| วนซ้ำ dict | `for k, v in pairs(t)` |
| จำนวน | `#t` (เฉพาะ sequential) |

### บทถัดไป

ในบทที่ 15 เราจะเรียนเรื่อง **String Manipulation** - การจัดการ string อย่างละเอียด
