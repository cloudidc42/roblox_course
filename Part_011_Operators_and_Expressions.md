# ตอนที่ 11: ตัวดำเนินการและนิพจน์ใน Lua

## บทนำ

ตัวดำเนินการ (Operators) คือสัญลักษณ์ที่ใช้ในการคำนวณ เปรียบเทียบ หรือดำเนินการกับข้อมูล ใน Lua มีตัวดำเนินการหลายประเภทที่จำเป็นต้องรู้เพื่อสร้างเกมใน Roblox

---

## 11.1 ตัวดำเนินการทางคณิตศาสตร์ (Arithmetic Operators)

```lua
-- ตัวดำเนินการพื้นฐาน
local a = 10
local b = 3

-- บวก (+)
print(a + b)    -- 13

-- ลบ (-)
print(a - b)    -- 7

-- คูณ (*)
print(a * b)    -- 30

-- หาร (/)
print(a / b)    -- 3.3333333333333

-- หารเอาส่วนจำนวนเต็ม (//)
print(a // b)   -- 3

-- เศษจากการหาร/โมดูลัส (%)
print(a % b)    -- 1

-- ยกกำลัง (^)
print(a ^ b)    -- 1000.0

-- ค่าลบ (unary minus)
print(-a)       -- -10
print(-b)       -- -3
```

### การใช้งานจริงใน Roblox

```lua
-- ระบบ damage calculation
local function calculateDamage(baseDamage, attackPower, defense)
    local totalDamage = (baseDamage + attackPower) - defense
    return math.max(1, totalDamage)  -- ความเสียหายขั้นต่ำ 1
end

-- ระบบ experience
local function getExpForLevel(level)
    return math.floor(100 * (level ^ 1.5))
end

for level = 1, 10 do
    print("Level " .. level .. " ต้องการ " .. getExpForLevel(level) .. " EXP")
end

-- ระบบคำนวณเวลา
local function formatTime(seconds)
    local minutes = math.floor(seconds / 60)
    local remainingSeconds = seconds % 60
    return string.format("%02d:%02d", minutes, remainingSeconds)
end

print(formatTime(90))   -- 01:30
print(formatTime(125))  -- 02:05
print(formatTime(3600)) -- 60:00

-- ระบบ Grid
local GRID_SIZE = 4

local function snapToGrid(position)
    return Vector3.new(
        math.floor(position.X / GRID_SIZE) * GRID_SIZE,
        math.floor(position.Y / GRID_SIZE) * GRID_SIZE,
        math.floor(position.Z / GRID_SIZE) * GRID_SIZE
    )
end

local rawPos = Vector3.new(7.3, 5.1, 12.8)
local snappedPos = snapToGrid(rawPos)
print("ตำแหน่งที่ snap แล้ว: " .. tostring(snappedPos))
-- Vector3(4, 4, 12)
```

---

## 11.2 ตัวดำเนินการเปรียบเทียบ (Comparison Operators)

ผลลัพธ์จะเป็น boolean (true/false)

```lua
local x = 10
local y = 20

-- เท่ากับ (==)
print(x == y)     -- false
print(x == 10)    -- true
print("a" == "a") -- true
print("a" == "A") -- false (Lua case-sensitive)

-- ไม่เท่ากับ (~=)
print(x ~= y)     -- true
print(x ~= 10)    -- false

-- น้อยกว่า (<)
print(x < y)      -- true
print(y < x)      -- false

-- มากกว่า (>)
print(x > y)      -- false
print(y > x)      -- true

-- น้อยกว่าหรือเท่ากับ (<=)
print(x <= 10)    -- true
print(x <= 9)     -- false

-- มากกว่าหรือเท่ากับ (>=)
print(x >= 10)    -- true
print(x >= 11)    -- false
```

### ข้อควรระวังในการเปรียบเทียบ

```lua
-- เปรียบเทียบ nil
local var = nil
print(var == nil)   -- true
print(var ~= nil)   -- false

-- เปรียบเทียบต่างชนิด
print(1 == "1")     -- false (ไม่แปลงชนิดในการเปรียบเทียบ!)
print(0 == false)   -- false
print(nil == false) -- false

-- เปรียบเทียบ table (เปรียบเทียบ reference ไม่ใช่ค่า)
local t1 = {1, 2, 3}
local t2 = {1, 2, 3}
local t3 = t1

print(t1 == t2)  -- false (คนละ object)
print(t1 == t3)  -- true (object เดียวกัน)
```

### การใช้งานจริงใน Roblox

```lua
local Players = game:GetService("Players")

-- ระบบตรวจสอบเลเวล
local function checkLevelRequirement(playerLevel, requiredLevel)
    if playerLevel >= requiredLevel then
        print("ผ่านเงื่อนไขเลเวล!")
        return true
    else
        local deficit = requiredLevel - playerLevel
        print("ต้องการเลเวลอีก " .. deficit .. " เลเวล")
        return false
    end
end

checkLevelRequirement(15, 10)   -- ผ่านเงื่อนไขเลเวล!
checkLevelRequirement(5, 20)    -- ต้องการเลเวลอีก 15 เลเวล

-- ระบบ Team
local function areSameTeam(player1, player2)
    local team1 = player1.Team
    local team2 = player2.Team
    
    if team1 == nil or team2 == nil then
        return false
    end
    
    return team1 == team2
end

-- ระบบ Zone
local function isInSafeZone(position)
    local safeZoneCenter = Vector3.new(0, 0, 0)
    local safeZoneRadius = 50
    
    local distance = (position - safeZoneCenter).Magnitude
    return distance <= safeZoneRadius
end
```

---

## 11.3 ตัวดำเนินการตรรกะ (Logical Operators)

```lua
-- and: ทั้งสองต้องเป็น true
print(true and true)    -- true
print(true and false)   -- false
print(false and true)   -- false
print(false and false)  -- false

-- or: อย่างน้อยหนึ่งต้องเป็น true
print(true or true)     -- true
print(true or false)    -- true
print(false or true)    -- true
print(false or false)   -- false

-- not: กลับค่า
print(not true)   -- false
print(not false)  -- true
print(not nil)    -- true
```

### Truthiness ใน Lua

**สำคัญมาก:** ใน Lua มีแค่ `false` และ `nil` เท่านั้นที่เป็น "falsy"
ค่าอื่นๆ ทั้งหมดเป็น "truthy" รวมถึง 0 และ ""

```lua
-- สิ่งที่เป็น falsy
if false then print("false เป็น truthy") else print("false เป็น falsy") end
if nil then print("nil เป็น truthy") else print("nil เป็น falsy") end

-- สิ่งที่เป็น truthy (อาจแปลกใจ!)
if 0 then print("0 เป็น truthy") end          -- พิมพ์นี้!
if "" then print("string ว่างเป็น truthy") end -- พิมพ์นี้!
if {} then print("table ว่างเป็น truthy") end  -- พิมพ์นี้!
```

### Short-circuit Evaluation

```lua
-- and: ถ้าฝั่งซ้ายเป็น false/nil จะ return ฝั่งซ้ายทันที
print(false and "hello")  -- false
print(nil and "world")    -- nil
print(true and "hello")   -- hello (return ฝั่งขวา)
print(1 and 2)            -- 2

-- or: ถ้าฝั่งซ้ายเป็น true จะ return ฝั่งซ้ายทันที
print(false or "hello")   -- hello
print(nil or "world")     -- world
print(true or "hello")    -- true
print(1 or 2)             -- 1
```

### Ternary Pattern ใน Lua

```lua
-- Lua ไม่มี ternary operator (? :) แต่เราจำลองได้
local x = 10
local result = x > 5 and "มากกว่า 5" or "ไม่มากกว่า 5"
print(result)  -- มากกว่า 5

-- ใช้สำหรับ default values
local playerName = nil
local displayName = playerName or "ผู้เล่นไม่รู้จัก"
print(displayName)  -- ผู้เล่นไม่รู้จัก

-- แต่ระวัง! ถ้า value ที่ต้องการเป็น false จะมีปัญหา
local playerScore = 0
local score = playerScore or 100  -- ผิด! 0 เป็น truthy แล้ว
print(score)  -- 100 (ผิดพลาด!)

-- วิธีที่ถูกต้อง
local score2 = playerScore ~= nil and playerScore or 100
-- แต่ถ้า playerScore = false ก็ยังมีปัญหา ใช้ if/else ดีกว่า
local score3
if playerScore ~= nil then
    score3 = playerScore
else
    score3 = 100
end
```

### การใช้งานจริงใน Roblox

```lua
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

-- ตรวจสอบเงื่อนไขหลายอย่าง
local function canPlayerAttack(player)
    local character = player.Character
    if not character then return false end
    
    local humanoid = character:FindFirstChild("Humanoid")
    if not humanoid then return false end
    
    local isAlive = humanoid.Health > 0
    local hasCooldown = not player:FindFirstChild("AttackCooldown")
    local hasWeapon = character:FindFirstChild("Sword") ~= nil
    
    return isAlive and hasCooldown and hasWeapon
end

-- ค่า default สำหรับ config
local function createGameConfig(options)
    options = options or {}  -- ถ้าไม่ส่ง options ใช้ table ว่าง
    
    return {
        maxPlayers = options.maxPlayers or 10,
        gameTime = options.gameTime or 300,
        difficulty = options.difficulty or "ปานกลาง",
        enablePvP = options.enablePvP ~= nil and options.enablePvP or true
    }
end

local config1 = createGameConfig()
print(config1.maxPlayers)   -- 10 (ค่า default)

local config2 = createGameConfig({maxPlayers = 20, difficulty = "ยาก"})
print(config2.maxPlayers)   -- 20
print(config2.difficulty)   -- ยาก
print(config2.gameTime)     -- 300 (ค่า default)
```

---

## 11.4 ตัวดำเนินการ String (String Operators)

```lua
-- การต่อ string (..)
local firstName = "สมชาย"
local lastName = "รักสงบ"
local fullName = firstName .. " " .. lastName
print(fullName)  -- สมชาย รักสงบ

-- การต่อกับตัวเลข (Lua แปลงอัตโนมัติ)
local score = 100
local message = "คะแนน: " .. score
print(message)  -- คะแนน: 100

-- ความยาว string (#)
local text = "Hello, World!"
print(#text)    -- 13

local thai = "สวัสดี"
print(#thai)    -- 18 (ภาษาไทย UTF-8 ใช้หลาย bytes ต่อตัวอักษร)
```

### String Comparison

```lua
-- เปรียบเทียบตาม ASCII/Unicode value
print("abc" < "abd")    -- true (c < d)
print("abc" < "abcd")   -- true (สั้นกว่า)
print("B" < "a")        -- true (uppercase มาก่อน lowercase ใน ASCII)

-- เปรียบเทียบความเท่ากัน
print("hello" == "hello")  -- true
print("Hello" == "hello")  -- false (case-sensitive)
print("  hi  " == "hi")    -- false (มี space)
```

---

## 11.5 ตัวดำเนินการ Length (#)

```lua
-- ความยาว string
local str = "Hello"
print(#str)  -- 5

-- จำนวนสมาชิกใน array (sequential table)
local arr = {10, 20, 30, 40, 50}
print(#arr)  -- 5

-- ระวัง! # ไม่น่าเชื่อถือสำหรับ table ที่มี nil
local arr2 = {1, 2, nil, 4, 5}
print(#arr2)  -- ผลลัพธ์ไม่แน่นอน! (อาจเป็น 2 หรือ 5)

-- ระวัง! # ไม่นับ key-value pairs
local dict = {a = 1, b = 2, c = 3}
print(#dict)  -- 0 (ไม่ใช่ 3!)
```

---

## 11.6 ลำดับการดำเนินการ (Operator Precedence)

จากสูงสุดไปต่ำสุด:

```lua
-- 1. ^ (ยกกำลัง) - ขวาไปซ้าย
-- 2. unary operators: not, # , - (ค่าลบ)
-- 3. *, /, //, % (คูณ, หาร)
-- 4. +, - (บวก, ลบ)
-- 5. .. (ต่อ string) - ขวาไปซ้าย
-- 6. <, >, <=, >=, ~=, == (เปรียบเทียบ)
-- 7. and
-- 8. or

-- ตัวอย่าง
print(2 + 3 * 4)      -- 14 (คูณก่อน)
print((2 + 3) * 4)    -- 20 (วงเล็บก่อน)
print(2 ^ 3 ^ 2)      -- 512 (2^(3^2) = 2^9 = 512, ขวาไปซ้าย)
print(10 - 3 - 2)     -- 5 (ซ้ายไปขวา)
print(not 1 == 1)      -- false (not (1==1) ไม่ใช่ (not 1) == 1)
print(not (1 == 1))    -- false
print((not 1) == 1)    -- false (not 1 = false, false == 1 = false)

-- ตัวอย่างที่ซับซ้อน
local x = 5
local y = 3
local z = 2

local result = x + y * z ^ 2 - 1
-- = 5 + 3 * (2^2) - 1
-- = 5 + 3 * 4 - 1
-- = 5 + 12 - 1
-- = 16
print(result)  -- 16
```

### แนะนำให้ใช้วงเล็บเพื่อความชัดเจน

```lua
-- แบบยากอ่าน
local confusing = x + y * z ^ 2 and z < x or y == 3

-- แบบชัดเจน
local clear = ((x + (y * (z ^ 2))) and (z < x)) or (y == 3)

-- เขียนแยกบรรทัดให้อ่านง่าย
local powerCalc = z ^ 2           -- 4
local mulCalc = y * powerCalc     -- 12
local addCalc = x + mulCalc       -- 17
local cmpCalc = z < x             -- true
local andCalc = addCalc and cmpCalc -- 17 and true = true (17 เป็น truthy)
local orCalc = andCalc or (y == 3)  -- true or true = true
```

---

## 11.7 ตัวดำเนินการ Bitwise (Lua 5.3+)

Roblox ใช้ Lua 5.1 ดังนั้น bitwise operators ใช้ผ่าน bit32 library:

```lua
-- bit32 library ใน Roblox
-- AND
print(bit32.band(12, 10))   -- 8  (1100 & 1010 = 1000)

-- OR
print(bit32.bor(12, 10))    -- 14 (1100 | 1010 = 1110)

-- XOR
print(bit32.bxor(12, 10))   -- 6  (1100 ^ 1010 = 0110)

-- NOT
print(bit32.bnot(0))        -- 4294967295 (all 1s in 32 bits)

-- Shift left
print(bit32.lshift(1, 4))   -- 16 (1 << 4)

-- Shift right
print(bit32.rshift(16, 4))  -- 1  (16 >> 4)

-- การใช้งานจริง: Flags
local PERM_READ  = 1   -- 001
local PERM_WRITE = 2   -- 010
local PERM_EXEC  = 4   -- 100

local userPerms = bit32.bor(PERM_READ, PERM_WRITE)  -- 011 = 3

-- ตรวจสอบ permission
local canRead  = bit32.band(userPerms, PERM_READ) ~= 0   -- true
local canWrite = bit32.band(userPerms, PERM_WRITE) ~= 0  -- true
local canExec  = bit32.band(userPerms, PERM_EXEC) ~= 0   -- false

print("อ่าน: " .. tostring(canRead))    -- true
print("เขียน: " .. tostring(canWrite))  -- true
print("รัน: " .. tostring(canExec))     -- false
```

---

## 11.8 นิพจน์ (Expressions)

นิพจน์คือการรวมตัวดำเนินการกับค่าเพื่อได้ผลลัพธ์:

```lua
-- นิพจน์ทางคณิตศาสตร์
local area = 3.14 * 5 ^ 2        -- พื้นที่วงกลม radius = 5
local hypotenuse = math.sqrt(3^2 + 4^2)  -- ทฤษฎีปีทาโกรัส

-- นิพจน์ boolean
local isAdult = age >= 18 and hasId == true
local canPlay = (membershipLevel >= 1) or (dayTrialExpiry > os.time())

-- นิพจน์ string
local greeting = "สวัสดี " .. playerName .. "! คุณมี " .. lives .. " ชีวิต"

-- นิพจน์ซับซ้อน
local damage = math.max(1, math.floor((attack * 1.5) - (defense * 0.8)))
```

### Function Call เป็น Expression

```lua
-- function call สามารถเป็นส่วนหนึ่งของนิพจน์
local greeting = string.upper("hello") .. " " .. string.lower("WORLD")
print(greeting)  -- HELLO world

local total = math.abs(-5) + math.floor(3.7) + math.ceil(2.1)
print(total)  -- 5 + 3 + 3 = 11

-- Method chain (ผ่าน table)
local result = ("  hello world  "):match("(%S+)$")  -- หาคำสุดท้าย
print(result)  -- world
```

---

## 11.9 ตัวอย่างการใช้งานจริงใน Roblox

### ระบบ Combat

```lua
-- Script: CombatSystem.lua

local CRITICAL_CHANCE = 0.15   -- 15%
local CRITICAL_MULTIPLIER = 2.0
local DODGE_CHANCE = 0.1       -- 10%

-- คำนวณความเสียหาย
local function calculateDamage(attacker, defender)
    -- ตรวจสอบการ dodge
    local rollDodge = math.random()
    if rollDodge < DODGE_CHANCE then
        return 0, "dodge"
    end
    
    -- คำนวณ base damage
    local baseDamage = attacker.attackPower - (defender.defense * 0.5)
    baseDamage = math.max(1, math.floor(baseDamage))
    
    -- ตรวจสอบ critical hit
    local rollCrit = math.random()
    local isCritical = rollCrit < CRITICAL_CHANCE
    
    local finalDamage = isCritical 
        and math.floor(baseDamage * CRITICAL_MULTIPLIER) 
        or baseDamage
    
    local hitType = isCritical and "critical" or "normal"
    
    return finalDamage, hitType
end

-- ทดสอบ
local attacker = {attackPower = 50, defense = 20}
local defender = {attackPower = 30, defense = 15}

math.randomseed(os.time())
local damage, hitType = calculateDamage(attacker, defender)
print("ความเสียหาย: " .. damage .. " (" .. hitType .. ")")
```

### ระบบ Score & Ranking

```lua
-- Script: ScoreSystem.lua

local SCORE_KILL = 100
local SCORE_ASSIST = 30
local SCORE_OBJECTIVE = 250
local SCORE_WIN_BONUS = 500

-- คำนวณ score สุดท้าย
local function calculateFinalScore(stats)
    local baseScore = 
        (stats.kills * SCORE_KILL) +
        (stats.assists * SCORE_ASSIST) +
        (stats.objectives * SCORE_OBJECTIVE)
    
    -- Bonus สำหรับ win
    local winBonus = stats.isWinner and SCORE_WIN_BONUS or 0
    
    -- Streak bonus
    local streakBonus = 0
    if stats.killStreak >= 10 then
        streakBonus = math.floor(baseScore * 0.5)   -- 50% bonus
    elseif stats.killStreak >= 5 then
        streakBonus = math.floor(baseScore * 0.25)  -- 25% bonus
    elseif stats.killStreak >= 3 then
        streakBonus = math.floor(baseScore * 0.1)   -- 10% bonus
    end
    
    -- Accuracy bonus
    local accuracyBonus = 0
    if stats.totalShots > 0 then
        local accuracy = stats.hits / stats.totalShots
        if accuracy >= 0.8 then
            accuracyBonus = 200
        elseif accuracy >= 0.6 then
            accuracyBonus = 100
        end
    end
    
    local totalScore = baseScore + winBonus + streakBonus + accuracyBonus
    
    return {
        base = baseScore,
        winBonus = winBonus,
        streakBonus = streakBonus,
        accuracyBonus = accuracyBonus,
        total = totalScore
    }
end

-- ทดสอบ
local playerStats = {
    kills = 15,
    assists = 8,
    objectives = 3,
    isWinner = true,
    killStreak = 7,
    hits = 72,
    totalShots = 100
}

local scoreBreakdown = calculateFinalScore(playerStats)
print("=== สรุปคะแนน ===")
print("คะแนนพื้นฐาน: " .. scoreBreakdown.base)
print("โบนัสชนะ: " .. scoreBreakdown.winBonus)
print("โบนัส Streak: " .. scoreBreakdown.streakBonus)
print("โบนัส Accuracy: " .. scoreBreakdown.accuracyBonus)
print("คะแนนรวม: " .. scoreBreakdown.total)
```

### ระบบ Physics/Movement

```lua
-- Script: MovementSystem.lua

local GRAVITY = Vector3.new(0, -196.2, 0)
local AIR_RESISTANCE = 0.98

-- คำนวณ trajectory (เส้นทางการเคลื่อนที่)
local function calculateProjectilePosition(initialPos, initialVelocity, time)
    local x = initialPos.X + (initialVelocity.X * time)
    local y = initialPos.Y + (initialVelocity.Y * time) + (0.5 * GRAVITY.Y * time^2)
    local z = initialPos.Z + (initialVelocity.Z * time)
    
    return Vector3.new(x, y, z)
end

-- คำนวณว่าต้องยิงด้วยมุมเท่าไหร่เพื่อให้ถึงเป้าหมาย
local function calculateLaunchAngle(distance, speed)
    -- angle = 0.5 * arcsin(g * d / v^2)
    local g = math.abs(GRAVITY.Y)
    local ratio = g * distance / (speed ^ 2)
    
    if ratio > 1 then
        return nil  -- ไม่สามารถยิงถึงได้
    end
    
    local angle = 0.5 * math.asin(ratio)
    return math.deg(angle)
end

-- ตัวอย่างการใช้งาน
local startPos = Vector3.new(0, 5, 0)
local vel = Vector3.new(20, 30, 0)

print("เส้นทางการเคลื่อนที่:")
for t = 0, 3, 0.5 do
    local pos = calculateProjectilePosition(startPos, vel, t)
    print(string.format("t=%.1f: (%.1f, %.1f, %.1f)", t, pos.X, pos.Y, pos.Z))
end

local angle = calculateLaunchAngle(50, 30)
if angle then
    print("\nมุมยิงสำหรับระยะ 50 studs ด้วยความเร็ว 30: " .. string.format("%.1f", angle) .. " องศา")
else
    print("\nไม่สามารถยิงถึงเป้าหมายได้")
end
```

---

## 11.10 แบบฝึกหัด

### แบบฝึกหัดที่ 1: ตัวดำเนินการคณิตศาสตร์

```lua
-- คำนวณสิ่งต่อไปนี้:
-- 1. พื้นที่สามเหลี่ยมที่มี base = 8 และ height = 5
-- 2. เส้นรอบวงของวงกลมที่มี radius = 7
-- 3. จำนวนนาทีใน 3 ชั่วโมง 45 นาที
-- 4. เปอร์เซ็นต์ที่ 35 จาก 200

local base = 8
local height = 5
-- เติมโค้ดที่นี่
```

### แบบฝึกหัดที่ 2: ตัวดำเนินการเปรียบเทียบ

```lua
-- เขียนฟังก์ชันที่:
-- 1. รับตัวเลขสองตัวและบอกว่าตัวไหนมากกว่า
-- 2. ตรวจสอบว่าตัวเลขอยู่ในช่วง [min, max] หรือไม่
-- 3. ตรวจสอบว่า string เป็น palindrome หรือไม่

local function compare(a, b)
    -- เติมโค้ด
end

local function isInRange(value, min, max)
    -- เติมโค้ด
end

local function isPalindrome(str)
    -- เติมโค้ด
end

print(compare(10, 20))
print(isInRange(5, 1, 10))
print(isPalindrome("racecar"))
print(isPalindrome("hello"))
```

### แบบฝึกหัดที่ 3: ตัวดำเนินการตรรกะ

```lua
-- เขียนฟังก์ชันตรวจสอบเงื่อนไขการเข้าถึง:
-- ผู้ใช้สามารถเข้าถึงได้ถ้า:
-- - มีอายุ >= 13 ปี AND
-- - มีบัญชี (hasAccount = true) AND
-- - ไม่ถูก ban (isBanned = false) AND
-- - (มี premium OR เล่นน้อยกว่า 1 ชั่วโมงต่อวัน)

local function canAccess(age, hasAccount, isBanned, hasPremium, dailyHours)
    -- เติมโค้ด
end

-- ทดสอบ
print(canAccess(15, true, false, false, 0.5))   -- true
print(canAccess(12, true, false, true, 5))       -- false (อายุน้อย)
print(canAccess(16, true, true, true, 2))        -- false (ถูก ban)
print(canAccess(18, true, false, false, 3))      -- false (เล่นนาน ไม่มี premium)
```

---

## สรุป

| ประเภท | ตัวดำเนินการ | ตัวอย่าง |
|--------|------------|---------|
| คณิตศาสตร์ | +, -, *, /, //, %, ^ | 10 + 5, 10 % 3 |
| เปรียบเทียบ | ==, ~=, <, >, <=, >= | a == b, x > 0 |
| ตรรกะ | and, or, not | a and b, not x |
| String | .., # | "a" .. "b", #str |
| Length | # | #array, #string |

### สิ่งสำคัญที่ต้องจำ

1. **~=** ไม่ใช่ != สำหรับ "ไม่เท่ากับ"
2. **0 และ "" เป็น truthy** ต่างจากภาษาอื่น
3. **Short-circuit**: `and` และ `or` หยุดเมื่อรู้ผลลัพธ์แล้ว
4. **ใช้วงเล็บ** เมื่อไม่แน่ใจลำดับการดำเนินการ
5. **ระวังการเปรียบเทียบ string** - Lua case-sensitive

### บทถัดไป

ในบทที่ 12 เราจะเรียนเรื่อง **Control Structures** - if/else, for loop, while loop ที่ใช้ควบคุมการทำงานของโปรแกรม
