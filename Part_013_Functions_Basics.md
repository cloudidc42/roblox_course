# ตอนที่ 13: ฟังก์ชัน (Functions) ใน Lua

## บทนำ

ฟังก์ชัน (Functions) คือการรวมกลุ่มโค้ดที่ทำงานอย่างเดียวกันไว้ด้วยกัน เพื่อให้สามารถเรียกใช้ซ้ำได้ ฟังก์ชันเป็นหัวใจสำคัญของการเขียนโปรแกรมที่ดี ช่วยให้โค้ดสะอาด อ่านง่าย และบำรุงรักษาได้ง่าย

---

## 13.1 การสร้างฟังก์ชัน

### รูปแบบพื้นฐาน

```lua
-- วิธีที่ 1: Function statement
function greet()
    print("สวัสดี!")
end

-- วิธีที่ 2: Function expression (แนะนำ)
local greet = function()
    print("สวัสดี!")
end

-- เรียกใช้
greet()  -- สวัสดี!
```

### ฟังก์ชันพร้อม Parameters

```lua
-- พารามิเตอร์เดียว
local function sayHello(name)
    print("สวัสดี " .. name .. "!")
end

sayHello("สมชาย")  -- สวัสดี สมชาย!
sayHello("World")   -- สวัสดี World!

-- หลายพารามิเตอร์
local function createPlayer(name, level, health)
    print("สร้างผู้เล่น: " .. name)
    print("เลเวล: " .. level)
    print("เลือด: " .. health)
end

createPlayer("สมชาย", 5, 100)
```

### ฟังก์ชันที่ Return ค่า

```lua
-- Return ค่าเดียว
local function add(a, b)
    return a + b
end

local sum = add(10, 5)
print(sum)  -- 15

-- Return หลายค่า
local function divmod(a, b)
    local quotient = math.floor(a / b)
    local remainder = a % b
    return quotient, remainder
end

local q, r = divmod(17, 5)
print("ผลหาร: " .. q .. " เศษ: " .. r)  -- ผลหาร: 3 เศษ: 2

-- Return ก่อนเวลา (early return)
local function safeDivide(a, b)
    if b == 0 then
        print("Error: หารด้วย 0 ไม่ได้!")
        return nil
    end
    return a / b
end

print(safeDivide(10, 2))  -- 5
print(safeDivide(10, 0))  -- nil (พร้อมข้อความ error)
```

---

## 13.2 Local Functions

```lua
-- ฟังก์ชัน local ที่แนะนำให้ใช้เสมอ
local function calculateArea(width, height)
    return width * height
end

-- ระวัง! ฟังก์ชัน local ที่ recursive
-- แบบนี้ผิด:
-- local function factorial(n)
--     if n <= 1 then return 1 end
--     return n * factorial(n - 1)  -- factorial ยังไม่ถูก declare!
-- end

-- แบบนี้ถูก:
local factorial  -- declare ก่อน
factorial = function(n)
    if n <= 1 then return 1 end
    return n * factorial(n - 1)
end

print(factorial(5))  -- 120

-- หรือใช้ local function syntax (Lua handle recursive ได้)
local function factorial2(n)
    if n <= 1 then return 1 end
    return n * factorial2(n - 1)  -- ใช้ได้กับ local function syntax
end

print(factorial2(5))  -- 120
```

---

## 13.3 Varargs (...)

ฟังก์ชันที่รับพารามิเตอร์ไม่จำกัดจำนวน

```lua
-- Varargs ด้วย ...
local function sum(...)
    local args = {...}  -- รวบรวม arguments เป็น table
    local total = 0
    
    for _, v in ipairs(args) do
        total = total + v
    end
    
    return total
end

print(sum(1, 2, 3))          -- 6
print(sum(10, 20, 30, 40))   -- 100
print(sum(1, 2, 3, 4, 5))    -- 15

-- ดูจำนวน arguments
local function countArgs(...)
    return select('#', ...)  -- นับจำนวน
end

print(countArgs(1, 2, 3))    -- 3
print(countArgs("a", "b"))   -- 2

-- ผสม fixed parameters กับ varargs
local function log(level, ...)
    local args = {...}
    local message = table.concat(args, " ")  -- รวม string ด้วย space
    print("[" .. level .. "] " .. message)
end

log("INFO", "เกมเริ่ม", "ผู้เล่น:", 10)
-- [INFO] เกมเริ่ม ผู้เล่น: 10

log("ERROR", "ไม่สามารถโหลด", "ข้อมูลได้")
-- [ERROR] ไม่สามารถโหลด ข้อมูลได้
```

---

## 13.4 Default Parameters

Lua ไม่มี default parameters แต่จำลองได้:

```lua
-- วิธีที่ 1: or operator
local function createPart(name, size, color)
    name = name or "Part"
    size = size or Vector3.new(4, 1, 2)
    color = color or BrickColor.new("Medium stone grey")
    
    local part = Instance.new("Part")
    part.Name = name
    part.Size = size
    part.BrickColor = color
    return part
end

local defaultPart = createPart()         -- ใช้ค่า default ทั้งหมด
local customPart = createPart("MyPart", Vector3.new(10, 2, 10))

-- วิธีที่ 2: Options table (pattern ที่ดีกว่า)
local function spawnEnemy(options)
    options = options or {}
    
    local config = {
        name = options.name or "ศัตรู",
        health = options.health or 100,
        damage = options.damage or 10,
        speed = options.speed or 16,
        level = options.level or 1,
        position = options.position or Vector3.new(0, 0, 0),
        isBoss = options.isBoss or false
    }
    
    -- สร้างศัตรูด้วย config
    print("Spawn " .. config.name .. " (Level " .. config.level .. ")")
    print("เลือด: " .. config.health .. " ความเร็ว: " .. config.speed)
    
    return config
end

-- ใช้งาน
local normalEnemy = spawnEnemy()
local bossEnemy = spawnEnemy({
    name = "บอสยักษ์",
    health = 5000,
    damage = 100,
    level = 50,
    isBoss = true
})
```

---

## 13.5 Higher-Order Functions

ฟังก์ชันที่รับหรือ return ฟังก์ชันอื่น

```lua
-- รับฟังก์ชันเป็น parameter
local function applyToAll(list, func)
    local result = {}
    for i, v in ipairs(list) do
        result[i] = func(v)
    end
    return result
end

local numbers = {1, 2, 3, 4, 5}

local doubled = applyToAll(numbers, function(x) return x * 2 end)
for _, v in ipairs(doubled) do io.write(v .. " ") end
-- 2 4 6 8 10

local squared = applyToAll(numbers, function(x) return x ^ 2 end)
for _, v in ipairs(squared) do io.write(v .. " ") end
-- 1 4 9 16 25

-- filter function
local function filter(list, predicate)
    local result = {}
    for _, v in ipairs(list) do
        if predicate(v) then
            table.insert(result, v)
        end
    end
    return result
end

local evens = filter(numbers, function(x) return x % 2 == 0 end)
-- {2, 4}

-- reduce function
local function reduce(list, func, initial)
    local acc = initial
    for _, v in ipairs(list) do
        acc = func(acc, v)
    end
    return acc
end

local total = reduce(numbers, function(acc, x) return acc + x end, 0)
print("รวม: " .. total)  -- รวม: 15

local product = reduce(numbers, function(acc, x) return acc * x end, 1)
print("คูณ: " .. product)  -- คูณ: 120
```

### Return Functions

```lua
-- Factory functions
local function createMultiplier(factor)
    return function(x)
        return x * factor
    end
end

local double = createMultiplier(2)
local triple = createMultiplier(3)
local tenTimes = createMultiplier(10)

print(double(5))   -- 10
print(triple(5))   -- 15
print(tenTimes(5)) -- 50

-- Closure-based counter
local function makeCounter(start, step)
    start = start or 0
    step = step or 1
    local count = start
    
    return {
        next = function()
            count = count + step
            return count
        end,
        reset = function()
            count = start
        end,
        current = function()
            return count
        end
    }
end

local waveCounter = makeCounter(0)
print("Wave: " .. waveCounter.next())  -- Wave: 1
print("Wave: " .. waveCounter.next())  -- Wave: 2
print("Wave: " .. waveCounter.next())  -- Wave: 3
waveCounter.reset()
print("Wave: " .. waveCounter.current())  -- Wave: 0
```

---

## 13.6 Closures

```lua
-- Closure ช่วยให้ฟังก์ชัน "จำ" ค่าจาก scope ที่อยู่ข้างนอก
local function createTimer()
    local startTime = os.time()
    
    return {
        elapsed = function()
            return os.time() - startTime
        end,
        reset = function()
            startTime = os.time()
        end
    }
end

local timer = createTimer()
-- ... (ผ่านเวลาไป)
print("เวลาที่ผ่านไป: " .. timer.elapsed() .. " วินาที")

-- Memoization ด้วย closure
local function memoize(func)
    local cache = {}
    
    return function(...)
        local key = table.concat({...}, ",")
        
        if cache[key] == nil then
            cache[key] = func(...)
            print("คำนวณใหม่สำหรับ: " .. key)
        else
            print("ใช้ cache สำหรับ: " .. key)
        end
        
        return cache[key]
    end
end

local function expensiveCalc(n)
    -- จำลองการคำนวณที่ใช้เวลานาน
    local result = 0
    for i = 1, n do
        result = result + i
    end
    return result
end

local memoCalc = memoize(expensiveCalc)
print(memoCalc(100))  -- คำนวณใหม่สำหรับ: 100, ผล: 5050
print(memoCalc(100))  -- ใช้ cache สำหรับ: 100, ผล: 5050
print(memoCalc(50))   -- คำนวณใหม่สำหรับ: 50, ผล: 1275
```

---

## 13.7 Methods (ฟังก์ชันใน Table)

```lua
-- Object-Oriented style ด้วย table
local Player = {}
Player.__index = Player  -- สำหรับ inheritance

-- Constructor
function Player.new(name, level)
    local self = setmetatable({}, Player)
    self.name = name
    self.level = level or 1
    self.health = 100
    self.maxHealth = 100
    self.isAlive = true
    return self
end

-- Methods
function Player:takeDamage(damage)
    if not self.isAlive then return end
    
    self.health = math.max(0, self.health - damage)
    
    if self.health <= 0 then
        self.isAlive = false
        self:onDeath()
    end
    
    print(self.name .. " รับความเสียหาย " .. damage .. " เลือดเหลือ: " .. self.health)
end

function Player:heal(amount)
    if not self.isAlive then return end
    
    self.health = math.min(self.maxHealth, self.health + amount)
    print(self.name .. " ฟื้นฟู " .. amount .. " เลือดปัจจุบัน: " .. self.health)
end

function Player:onDeath()
    print(self.name .. " ตายแล้ว!")
    -- trigger respawn, etc.
end

function Player:getInfo()
    return string.format(
        "[%s] Level %d - HP: %d/%d - %s",
        self.name,
        self.level,
        self.health,
        self.maxHealth,
        self.isAlive and "มีชีวิต" or "ตาย"
    )
end

-- ทดสอบ
local player1 = Player.new("สมชาย", 10)
print(player1:getInfo())
player1:takeDamage(30)
player1:heal(20)
player1:takeDamage(100)  -- ตาย
print(player1:getInfo())
```

---

## 13.8 ฟังก์ชัน Utility สำหรับ Roblox

```lua
-- ===== Utility Functions สำหรับ Roblox =====

-- รอจนกว่า condition จะเป็น true
local function waitUntil(condition, timeout, interval)
    interval = interval or 0.1
    timeout = timeout or 10
    
    local elapsed = 0
    while not condition() do
        task.wait(interval)
        elapsed = elapsed + interval
        
        if elapsed >= timeout then
            return false  -- timeout
        end
    end
    
    return true  -- สำเร็จ
end

-- ใช้งาน
local Players = game:GetService("Players")
local function waitForPlayer(playerName)
    local success = waitUntil(function()
        return Players:FindFirstChild(playerName) ~= nil
    end, 30)  -- รอสูงสุด 30 วินาที
    
    if success then
        return Players:FindFirstChild(playerName)
    else
        print("Timeout รอ " .. playerName)
        return nil
    end
end

-- Deep copy table
local function deepCopy(original)
    local copy = {}
    for k, v in pairs(original) do
        if type(v) == "table" then
            copy[k] = deepCopy(v)  -- recursive copy
        else
            copy[k] = v
        end
    end
    return copy
end

-- ทดสอบ deep copy
local original = {a = 1, b = {c = 2, d = 3}}
local copy = deepCopy(original)
copy.b.c = 999  -- ไม่กระทบ original

print(original.b.c)  -- 2 (ไม่เปลี่ยน)
print(copy.b.c)      -- 999

-- Debounce: ป้องกันการ call ถี่เกินไป
local function debounce(func, cooldown)
    local lastCall = 0
    
    return function(...)
        local now = os.clock()
        if now - lastCall >= cooldown then
            lastCall = now
            return func(...)
        end
    end
end

-- ใช้กับ Touched event
local function onTouched(hit)
    print("Touched! " .. hit.Name)
end

local debouncedTouched = debounce(onTouched, 0.5)  -- cooldown 0.5 วินาที
-- script.Parent.Touched:Connect(debouncedTouched)

-- Format number ให้อ่านง่าย
local function formatNumber(n)
    if n >= 1000000 then
        return string.format("%.1fM", n / 1000000)
    elseif n >= 1000 then
        return string.format("%.1fK", n / 1000)
    else
        return tostring(n)
    end
end

print(formatNumber(1500))       -- 1.5K
print(formatNumber(2500000))    -- 2.5M
print(formatNumber(999))        -- 999

-- Interpolate ค่าระหว่าง a และ b
local function lerp(a, b, t)
    return a + (b - a) * t
end

-- ใช้กับ animation
for t = 0, 1, 0.1 do
    local value = lerp(0, 100, t)
    print(string.format("t=%.1f: %.1f", t, value))
end
```

---

## 13.9 ฟังก์ชัน Pattern ใน Roblox

### Event Callback Pattern

```lua
local Players = game:GetService("Players")

-- Pattern: สร้าง callback system
local EventSystem = {}
EventSystem.__index = EventSystem

function EventSystem.new()
    local self = setmetatable({}, EventSystem)
    self.listeners = {}
    return self
end

function EventSystem:on(event, callback)
    if not self.listeners[event] then
        self.listeners[event] = {}
    end
    table.insert(self.listeners[event], callback)
    
    -- Return unsubscribe function
    return function()
        for i, cb in ipairs(self.listeners[event]) do
            if cb == callback then
                table.remove(self.listeners[event], i)
                break
            end
        end
    end
end

function EventSystem:emit(event, ...)
    if self.listeners[event] then
        for _, callback in ipairs(self.listeners[event]) do
            callback(...)
        end
    end
end

-- ใช้งาน
local gameEvents = EventSystem.new()

local unsubscribe = gameEvents:on("PlayerDied", function(playerName)
    print(playerName .. " ตายแล้ว!")
end)

gameEvents:on("PlayerDied", function(playerName)
    print("บันทึก: " .. playerName .. " died")
end)

gameEvents:emit("PlayerDied", "สมชาย")
-- สมชาย ตายแล้ว!
-- บันทึก: สมชาย died

unsubscribe()  -- ยกเลิก listener แรก
gameEvents:emit("PlayerDied", "สมหญิง")
-- บันทึก: สมหญิง died (เฉพาะ listener ที่สอง)
```

### Async Pattern ด้วย Coroutines

```lua
-- ฟังก์ชัน async แบบง่าย
local function async(func)
    return function(...)
        local args = {...}
        coroutine.wrap(function()
            func(table.unpack(args))
        end)()
    end
end

-- ใช้งาน
local processPlayer = async(function(player)
    print("เริ่มประมวลผล " .. player.Name)
    
    task.wait(1)  -- รอข้อมูล
    print("โหลดข้อมูล " .. player.Name .. " เสร็จ")
    
    task.wait(0.5)
    print("setup " .. player.Name .. " เสร็จ")
end)

-- เรียกแบบ non-blocking
-- processPlayer(somePlayer)
```

---

## 13.10 ตัวอย่างโปรเจกต์: ระบบ Quest

```lua
-- Script: QuestSystem.lua

local QuestSystem = {}
QuestSystem.__index = QuestSystem

-- สร้าง Quest
function QuestSystem.createQuest(id, title, description, objectives, rewards)
    return {
        id = id,
        title = title,
        description = description,
        objectives = objectives,
        rewards = rewards,
        isCompleted = false,
        progress = {}
    }
end

-- ตรวจสอบความคืบหน้า
function QuestSystem.updateProgress(quest, objectiveId, amount)
    if quest.isCompleted then
        return false, "Quest เสร็จแล้ว"
    end
    
    local objective = nil
    for _, obj in ipairs(quest.objectives) do
        if obj.id == objectiveId then
            objective = obj
            break
        end
    end
    
    if not objective then
        return false, "ไม่พบ objective"
    end
    
    -- อัปเดต progress
    if not quest.progress[objectiveId] then
        quest.progress[objectiveId] = 0
    end
    
    quest.progress[objectiveId] = math.min(
        quest.progress[objectiveId] + amount,
        objective.required
    )
    
    print(string.format("[Quest: %s] %s: %d/%d", 
        quest.title, 
        objective.description,
        quest.progress[objectiveId],
        objective.required
    ))
    
    -- ตรวจสอบว่าทุก objective เสร็จหรือยัง
    local allCompleted = true
    for _, obj in ipairs(quest.objectives) do
        local progress = quest.progress[obj.id] or 0
        if progress < obj.required then
            allCompleted = false
            break
        end
    end
    
    if allCompleted then
        quest.isCompleted = true
        return true, "Quest เสร็จแล้ว!"
    end
    
    return true, "อัปเดต progress สำเร็จ"
end

-- แสดงข้อมูล Quest
function QuestSystem.displayQuest(quest)
    print("=== " .. quest.title .. " ===")
    print(quest.description)
    print("\nภารกิจ:")
    
    for _, obj in ipairs(quest.objectives) do
        local progress = quest.progress[obj.id] or 0
        local status = progress >= obj.required and "[เสร็จ]" or "[กำลังทำ]"
        print(string.format("  %s %s: %d/%d", 
            status, obj.description, progress, obj.required))
    end
    
    if quest.isCompleted then
        print("\n[Quest เสร็จแล้ว!]")
        print("รางวัล:")
        for rewardType, value in pairs(quest.rewards) do
            print("  " .. rewardType .. ": " .. tostring(value))
        end
    end
    
    print("=" .. string.rep("=", #quest.title + 2))
end

-- ทดสอบ
local mainQuest = QuestSystem.createQuest(
    "q001",
    "นักรบมือใหม่",
    "พิสูจน์ตัวเองด้วยการฆ่าศัตรู",
    {
        {id = "kill_goblin", description = "ฆ่า Goblin", required = 10},
        {id = "kill_orc", description = "ฆ่า Orc", required = 5},
        {id = "collect_potion", description = "เก็บ Potion", required = 3}
    },
    {
        exp = 500,
        gold = 100,
        item = "ดาบนักรบ"
    }
)

-- จำลองการเล่น
QuestSystem.updateProgress(mainQuest, "kill_goblin", 5)
QuestSystem.updateProgress(mainQuest, "kill_goblin", 5)  -- ครบ 10
QuestSystem.updateProgress(mainQuest, "kill_orc", 3)
QuestSystem.updateProgress(mainQuest, "collect_potion", 3)
QuestSystem.updateProgress(mainQuest, "kill_orc", 2)  -- ครบ 5

QuestSystem.displayQuest(mainQuest)
```

---

## 13.11 แบบฝึกหัด

### แบบฝึกหัดที่ 1: ฟังก์ชัน Utility

```lua
-- สร้างฟังก์ชันต่อไปนี้:

-- 1. clamp(value, min, max): จำกัดค่าให้อยู่ในช่วง [min, max]
local function clamp(value, min, max)
    -- เติมโค้ด
end

-- 2. roundTo(value, decimals): ปัดเศษทศนิยมตามที่กำหนด
local function roundTo(value, decimals)
    -- เติมโค้ด
end

-- 3. isInArray(arr, value): ตรวจสอบว่าค่าอยู่ใน array หรือไม่
local function isInArray(arr, value)
    -- เติมโค้ด
end

-- ทดสอบ
print(clamp(15, 0, 10))   -- 10
print(clamp(-5, 0, 10))   -- 0
print(clamp(5, 0, 10))    -- 5

print(roundTo(3.14159, 2))  -- 3.14
print(roundTo(2.567, 1))    -- 2.6

print(isInArray({1, 2, 3, 4, 5}, 3))   -- true
print(isInArray({1, 2, 3, 4, 5}, 6))   -- false
```

### แบบฝึกหัดที่ 2: ระบบ Shop

```lua
-- สร้างระบบร้านค้าอย่างง่าย:
-- - รายการสินค้าพร้อมราคา
-- - ฟังก์ชัน buy(playerMoney, itemName)
-- - ฟังก์ชัน displayShop()
-- - ฟังก์ชัน getDiscount(level) คืนค่า discount ตามเลเวล

local Shop = {}

local items = {
    {name = "ดาบ", price = 100, damage = 15},
    {name = "โล่", price = 80, defense = 10},
    {name = "ยาแดง", price = 30, heal = 50},
    {name = "ธนู", price = 120, damage = 12, range = 50}
}

-- เติมโค้ด functions ที่นี่

-- ทดสอบ
-- displayShop()
-- local result = buy(150, "ดาบ")
-- print(result)
```

### แบบฝึกหัดที่ 3: Higher-Order Functions

```lua
-- ใช้ higher-order functions เพื่อ:
-- 1. หาผลรวมของเลขคู่ในรายการ
-- 2. แปลงชื่อทั้งหมดเป็น uppercase
-- 3. กรองผู้เล่นที่เลเวล >= 10

local numbers = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
local names = {"สมชาย", "สมหญิง", "สมศักดิ์", "สมใจ"}
local players = {
    {name = "Player1", level = 5},
    {name = "Player2", level = 15},
    {name = "Player3", level = 10},
    {name = "Player4", level = 3}
}

-- เติมโค้ดที่นี่
```

---

## สรุป

| แนวคิด | รายละเอียด |
|--------|-----------|
| Function Declaration | `local function name(params) ... end` |
| Function Expression | `local name = function(params) ... end` |
| Return | Return ค่าเดียวหรือหลายค่า |
| Varargs | `...` สำหรับ parameters ไม่จำกัด |
| Default Params | `param = param or default` |
| Higher-Order | รับหรือ return ฟังก์ชัน |
| Closure | ฟังก์ชันที่ "จำ" scope ข้างนอก |
| Method | ฟังก์ชันใน table ด้วย `:` |

### หลักการสำคัญ

1. **ฟังก์ชันควรทำสิ่งเดียว** (Single Responsibility)
2. **ตั้งชื่อให้สื่อความหมาย** เช่น `calculateDamage()` ไม่ใช่ `calc()`
3. **ใช้ local** เสมอ
4. **Early Return** เพื่อลด nesting
5. **Document functions** ด้วย comments

### บทถัดไป

ในบทที่ 14 เราจะเรียนเรื่อง **Tables and Arrays** - การใช้งาน table ซึ่งเป็นโครงสร้างข้อมูลหลักใน Lua
