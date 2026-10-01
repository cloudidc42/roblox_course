# ตอนที่ 10: ตัวแปรและชนิดข้อมูลใน Lua

## บทนำ

ในการเขียนโปรแกรมทุกภาษา ตัวแปร (Variables) คือพื้นฐานที่สำคัญที่สุด ตัวแปรคือ "กล่อง" ที่เราใช้เก็บข้อมูลต่างๆ ไว้ในหน่วยความจำของคอมพิวเตอร์ ใน Lua ซึ่งเป็นภาษาที่ Roblox ใช้ การทำความเข้าใจตัวแปรและชนิดข้อมูลจะช่วยให้คุณสร้างเกมได้อย่างมีประสิทธิภาพ

---

## 10.1 ตัวแปรคืออะไร?

ตัวแปรคือชื่อที่เราตั้งให้กับพื้นที่ในหน่วยความจำ เพื่อเก็บข้อมูลที่ต้องการใช้ในโปรแกรม

### การประกาศตัวแปร

ใน Lua มีสองแบบหลักในการประกาศตัวแปร:

**1. ตัวแปรโลคอล (Local Variables)** - ใช้คำว่า `local`
```lua
local playerName = "สมชาย"
local playerScore = 100
local isAlive = true
```

**2. ตัวแปรกลอบอล (Global Variables)** - ไม่ใช้คำว่า `local`
```lua
playerName = "สมชาย"
playerScore = 100
isAlive = true
```

### ความแตกต่างระหว่าง Local และ Global

```lua
-- ตัวอย่างการใช้งานตัวแปร local
local function testLocal()
    local x = 10  -- ตัวแปรนี้ใช้ได้แค่ในฟังก์ชันนี้
    print(x)      -- แสดง: 10
end

testLocal()
-- print(x)  -- จะเกิดข้อผิดพลาด เพราะ x ไม่มีในขอบเขตนี้

-- ตัวอย่างการใช้งานตัวแปร global
function testGlobal()
    globalVar = 20  -- ตัวแปรนี้ใช้ได้ทุกที่
    print(globalVar)
end

testGlobal()
print(globalVar)  -- แสดง: 20 (ยังใช้ได้นอกฟังก์ชัน)
```

### ทำไมต้องใช้ Local?

ใน Roblox การใช้ `local` เป็นสิ่งที่แนะนำเสมอเพราะ:
1. **ประสิทธิภาพดีกว่า** - Lua เข้าถึงตัวแปร local ได้เร็วกว่า global
2. **ป้องกันบั๊ก** - ตัวแปรไม่ไปทับกันโดยไม่ตั้งใจ
3. **โค้ดสะอาดกว่า** - รู้ขอบเขตการใช้งานชัดเจน

```lua
-- แนวทางที่ดี: ใช้ local เสมอ
local GameSettings = {
    maxPlayers = 10,
    gameTime = 300,
    difficulty = "ปานกลาง"
}

local function getPlayerCount()
    local count = 0
    for _, player in pairs(game.Players:GetPlayers()) do
        count = count + 1
    end
    return count
end
```

---

## 10.2 ชนิดข้อมูลใน Lua

Lua มีชนิดข้อมูลพื้นฐาน 8 ชนิด:

### 1. nil

`nil` แทนค่าว่างหรือการไม่มีค่า

```lua
local myVar = nil
print(myVar)        -- แสดง: nil
print(type(myVar))  -- แสดง: nil

-- ตัวแปรที่ยังไม่ได้กำหนดค่าจะเป็น nil อัตโนมัติ
local undeclared
print(undeclared)   -- แสดง: nil

-- ใช้ nil เพื่อล้างค่าตัวแปร
local playerData = "ข้อมูลผู้เล่น"
playerData = nil  -- ล้างค่า
print(playerData)   -- แสดง: nil
```

**การใช้งานใน Roblox:**
```lua
-- ตรวจสอบว่า object มีอยู่หรือไม่
local sword = player.Character:FindFirstChild("Sword")
if sword ~= nil then
    print("ผู้เล่นมีดาบ")
else
    print("ผู้เล่นไม่มีดาบ")
end

-- เขียนสั้นกว่าได้
if sword then
    print("ผู้เล่นมีดาบ")
end
```

### 2. boolean

ค่าความจริง มีแค่ `true` หรือ `false`

```lua
local isGameRunning = true
local isPlayerDead = false

print(isGameRunning)        -- แสดง: true
print(type(isGameRunning))  -- แสดง: boolean

-- การดำเนินการกับ boolean
print(true and false)   -- แสดง: false
print(true or false)    -- แสดง: true
print(not true)         -- แสดง: false
print(not false)        -- แสดง: true
```

**การใช้งานใน Roblox:**
```lua
local Players = game:GetService("Players")

local function checkPlayerStatus(player)
    local character = player.Character
    local humanoid = character and character:FindFirstChild("Humanoid")
    
    local isAlive = humanoid and humanoid.Health > 0 or false
    local hasWeapon = character and character:FindFirstChild("Tool") ~= nil or false
    
    print("ผู้เล่น " .. player.Name .. " มีชีวิต: " .. tostring(isAlive))
    print("ผู้เล่นมีอาวุธ: " .. tostring(hasWeapon))
    
    return isAlive
end
```

### 3. number

ตัวเลข ทั้งจำนวนเต็มและทศนิยม

```lua
-- จำนวนเต็ม
local playerCount = 10
local score = 9999
local level = 1

-- ทศนิยม
local health = 100.5
local speed = 16.5
local pi = 3.14159

-- จำนวนลบ
local debt = -100
local temperature = -5.5

-- สัญกรณ์ทางวิทยาศาสตร์
local bigNumber = 1e10      -- 10,000,000,000
local smallNumber = 1.5e-3  -- 0.0015

print(type(playerCount))  -- แสดง: number
print(type(health))       -- แสดง: number (Lua ไม่แยก int กับ float)
```

**การดำเนินการทางคณิตศาสตร์:**
```lua
local a = 10
local b = 3

print(a + b)   -- บวก: 13
print(a - b)   -- ลบ: 7
print(a * b)   -- คูณ: 30
print(a / b)   -- หาร: 3.3333...
print(a // b)  -- หารเอาส่วนจำนวนเต็ม: 3
print(a % b)   -- เศษจากการหาร: 1
print(a ^ b)   -- ยกกำลัง: 1000
print(-a)      -- ค่าลบ: -10
```

**การใช้งานใน Roblox:**
```lua
local Players = game:GetService("Players")

-- ระบบคะแนน
local function addScore(player, points)
    local leaderstats = player:FindFirstChild("leaderstats")
    if leaderstats then
        local score = leaderstats:FindFirstChild("Score")
        if score then
            score.Value = score.Value + points
            print(player.Name .. " ได้รับ " .. points .. " คะแนน")
            print("คะแนนรวม: " .. score.Value)
        end
    end
end

-- ระบบ Health
local function calculateHealthPercentage(currentHealth, maxHealth)
    if maxHealth <= 0 then return 0 end
    return (currentHealth / maxHealth) * 100
end

local healthPercent = calculateHealthPercentage(75, 100)
print("เลือดเหลือ: " .. healthPercent .. "%")  -- แสดง: เลือดเหลือ: 75%
```

### 4. string

ข้อความ (ตัวอักษร)

```lua
-- วิธีการสร้าง string
local name1 = "สวัสดีโลก"     -- ใช้ double quotes
local name2 = 'สวัสดี Roblox'  -- ใช้ single quotes
local longText = [[
    นี่คือข้อความ
    หลายบรรทัด
    ใน Lua
]]

print(type(name1))  -- แสดง: string

-- การรวม string (concatenation)
local firstName = "สมชาย"
local lastName = "ใจดี"
local fullName = firstName .. " " .. lastName
print(fullName)  -- แสดง: สมชาย ใจดี

-- ความยาวของ string
local text = "Hello"
print(#text)  -- แสดง: 5

-- การแปลงตัวเลขเป็น string
local score = 100
local scoreText = "คะแนน: " .. score  -- Lua แปลงอัตโนมัติ
print(scoreText)  -- แสดง: คะแนน: 100

-- ฟังก์ชัน string ที่ใช้บ่อย
local message = "Hello, World!"
print(string.upper(message))   -- HELLO, WORLD!
print(string.lower(message))   -- hello, world!
print(string.len(message))     -- 13
print(string.reverse(message)) -- !dlroW ,olleH
print(string.sub(message, 1, 5)) -- Hello
```

**การใช้งานใน Roblox:**
```lua
local Players = game:GetService("Players")

-- แสดงข้อความต้อนรับผู้เล่น
local function welcomePlayer(player)
    local welcomeMsg = "ยินดีต้อนรับ " .. player.Name .. " สู่เกมของเรา!"
    
    -- แสดงใน Output
    print(welcomeMsg)
    
    -- ส่งข้อความไปยังผู้เล่น
    local gui = player.PlayerGui
    -- (โค้ดสร้าง GUI จะอยู่ในบทถัดไป)
end

-- ตรวจสอบชื่อผู้เล่น
local function isValidName(name)
    -- ตรวจสอบว่าชื่อมีความยาวเหมาะสม
    local nameLength = string.len(name)
    if nameLength < 3 or nameLength > 20 then
        return false, "ชื่อต้องมีความยาว 3-20 ตัวอักษร"
    end
    return true, "ชื่อถูกต้อง"
end

local valid, message = isValidName("สมชาย")
print(message)  -- ชื่อถูกต้อง
```

### 5. table

ชนิดข้อมูลที่ซับซ้อนที่สุดใน Lua ใช้เก็บข้อมูลหลายอย่าง (จะอธิบายละเอียดในบทที่ 14)

```lua
-- ตัวอย่างเบื้องต้น
local playerData = {
    name = "สมชาย",
    score = 100,
    level = 5,
    isAlive = true
}

-- อ่านค่า
print(playerData.name)   -- สมชาย
print(playerData.score)  -- 100

-- เปลี่ยนค่า
playerData.score = 200
print(playerData.score)  -- 200

-- Array (ตาราง indexed)
local weapons = {"ดาบ", "โล่", "ธนู", "ไม้กายสิทธิ์"}
print(weapons[1])  -- ดาบ (Lua เริ่มที่ 1 ไม่ใช่ 0)
print(weapons[2])  -- โล่
print(#weapons)    -- 4 (จำนวนสมาชิก)
```

### 6. function

ฟังก์ชันก็เป็น type หนึ่งใน Lua (จะอธิบายละเอียดในบทที่ 13)

```lua
-- ฟังก์ชันเป็น first-class value
local greet = function(name)
    return "สวัสดี " .. name
end

print(type(greet))    -- แสดง: function
print(greet("โลก"))   -- แสดง: สวัสดี โลก

-- ฟังก์ชันสามารถเก็บในตาราง
local Utils = {
    add = function(a, b) return a + b end,
    sub = function(a, b) return a - b end,
    mul = function(a, b) return a * b end
}

print(Utils.add(10, 5))  -- 15
print(Utils.sub(10, 5))  -- 5
print(Utils.mul(10, 5))  -- 50
```

### 7. userdata

ข้อมูลจากภาษา C ซึ่งใน Roblox คือ object ต่างๆ เช่น Instance, Vector3, CFrame

```lua
-- ตัวอย่าง userdata ใน Roblox
local part = Instance.new("Part")
print(type(part))  -- แสดง: userdata

local position = Vector3.new(0, 10, 0)
print(type(position))  -- แสดง: userdata

local cf = CFrame.new(0, 5, 0)
print(type(cf))  -- แสดง: userdata
```

### 8. thread

สำหรับ coroutines (การทำงานพร้อมกัน)

```lua
-- ตัวอย่าง thread
local co = coroutine.create(function()
    print("เริ่ม coroutine")
    coroutine.yield()
    print("ดำเนินต่อ")
end)

print(type(co))      -- แสดง: thread
coroutine.resume(co)  -- เริ่ม coroutine
coroutine.resume(co)  -- ดำเนินต่อ
```

---

## 10.3 การแปลงชนิดข้อมูล (Type Conversion)

### การแปลงโดยอัตโนมัติ (Implicit Conversion)

```lua
-- Lua แปลง string เป็น number ในการคำนวณ
local result = "10" + 5
print(result)  -- แสดง: 15 (string "10" ถูกแปลงเป็น number)

local result2 = "3.14" * 2
print(result2)  -- แสดง: 6.28

-- แต่จะเกิด error ถ้าแปลงไม่ได้
-- local error = "hello" + 5  -- ERROR!
```

### การแปลงด้วย Functions

```lua
-- string -> number
local numStr = "42"
local num = tonumber(numStr)
print(num)         -- แสดง: 42
print(type(num))   -- แสดง: number

-- number -> string
local score = 100
local scoreStr = tostring(score)
print(scoreStr)        -- แสดง: 100
print(type(scoreStr))  -- แสดง: string

-- ตรวจสอบการแปลง
local invalid = tonumber("ไม่ใช่ตัวเลข")
print(invalid)  -- แสดง: nil (แปลงไม่ได้)

-- การแปลง boolean
print(tostring(true))   -- แสดง: true
print(tostring(false))  -- แสดง: false
print(tostring(nil))    -- แสดง: nil
```

**การใช้งานใน Roblox:**
```lua
-- รับค่าจาก TextBox และแปลงเป็นตัวเลข
local function processInput(inputText)
    local value = tonumber(inputText)
    if value == nil then
        print("กรุณาใส่ตัวเลข")
        return false
    end
    
    if value < 0 or value > 100 then
        print("กรุณาใส่ตัวเลขระหว่าง 0-100")
        return false
    end
    
    print("ค่าที่ได้รับ: " .. value)
    return true
end

processInput("75")    -- ค่าที่ได้รับ: 75
processInput("abc")   -- กรุณาใส่ตัวเลข
processInput("150")   -- กรุณาใส่ตัวเลขระหว่าง 0-100
```

---

## 10.4 การตรวจสอบชนิดข้อมูล

### ฟังก์ชัน type()

```lua
-- ตรวจสอบชนิดข้อมูล
print(type(nil))           -- nil
print(type(true))          -- boolean
print(type(false))         -- boolean
print(type(42))            -- number
print(type(3.14))          -- number
print(type("สวัสดี"))       -- string
print(type({}))            -- table
print(type(print))         -- function

-- ใน Roblox
local part = Instance.new("Part")
print(type(part))          -- userdata
print(type(Vector3.new())) -- userdata
```

### การตรวจสอบใน Roblox

```lua
-- ตรวจสอบว่า object เป็น Instance หรือไม่
local function isInstance(obj)
    return typeof(obj) == "Instance"
end

-- typeof() ให้ข้อมูลละเอียดกว่า type()
local part = Instance.new("Part")
print(typeof(part))          -- Instance
print(typeof(Vector3.new())) -- Vector3
print(typeof(42))            -- number
print(typeof("text"))        -- string

-- ตรวจสอบ class ของ Instance
local function getInstanceInfo(obj)
    if typeof(obj) == "Instance" then
        print("ชื่อ: " .. obj.Name)
        print("ClassName: " .. obj.ClassName)
        print("เป็น BasePart: " .. tostring(obj:IsA("BasePart")))
    end
end

local workspace = game:GetService("Workspace")
local myPart = Instance.new("Part")
myPart.Parent = workspace
getInstanceInfo(myPart)
```

---

## 10.5 ขอบเขตของตัวแปร (Scope)

### Local Scope

```lua
-- ตัวอย่างขอบเขต
local x = 1  -- ขอบเขตภายนอก

do
    local x = 2  -- ตัวแปร x ใหม่ในขอบเขตนี้
    print(x)  -- แสดง: 2
end

print(x)  -- แสดง: 1 (กลับมาใช้ x ของขอบเขตภายนอก)

-- ขอบเขตในฟังก์ชัน
local function outer()
    local outerVar = "ข้างนอก"
    
    local function inner()
        local innerVar = "ข้างใน"
        print(outerVar)  -- เข้าถึงได้! (closure)
        print(innerVar)  -- เข้าถึงได้
    end
    
    inner()
    -- print(innerVar)  -- ERROR! ไม่สามารถเข้าถึงตัวแปรของ inner ได้
end

outer()
```

### Closure

```lua
-- ตัวอย่าง closure ใน Roblox
local function createCounter(startValue)
    local count = startValue or 0  -- เก็บค่าใน closure
    
    return {
        increment = function()
            count = count + 1
            return count
        end,
        decrement = function()
            count = count - 1
            return count
        end,
        getCount = function()
            return count
        end,
        reset = function()
            count = startValue or 0
        end
    }
end

local scoreCounter = createCounter(0)
print(scoreCounter.increment())  -- 1
print(scoreCounter.increment())  -- 2
print(scoreCounter.increment())  -- 3
print(scoreCounter.decrement())  -- 2
print(scoreCounter.getCount())   -- 2
scoreCounter.reset()
print(scoreCounter.getCount())   -- 0
```

---

## 10.6 ตัวแปรพิเศษใน Roblox

### Roblox Data Types

Roblox มีชนิดข้อมูลพิเศษที่ใช้บ่อย:

```lua
-- Vector3: ตำแหน่งใน 3D space
local position = Vector3.new(10, 5, 0)
print(position.X)  -- 10
print(position.Y)  -- 5
print(position.Z)  -- 0

-- CFrame: ตำแหน่งและการหมุน
local cf = CFrame.new(0, 10, 0)  -- ตำแหน่งที่ (0, 10, 0)
local rotatedCF = CFrame.Angles(0, math.rad(45), 0)  -- หมุน 45 องศา

-- Color3: สี
local red = Color3.new(1, 0, 0)         -- แดง
local green = Color3.new(0, 1, 0)       -- เขียว
local blue = Color3.new(0, 0, 1)        -- น้ำเงิน
local white = Color3.new(1, 1, 1)       -- ขาว
local customColor = Color3.fromRGB(255, 128, 0)  -- ส้ม

-- BrickColor: สีแบบ Roblox เก่า
local brickRed = BrickColor.new("Bright red")
local brickBlue = BrickColor.new("Bright blue")

-- UDim2: ขนาดและตำแหน่งสำหรับ GUI
local guiSize = UDim2.new(0.5, 0, 0.1, 0)   -- 50% ของหน้าจอ
local guiPos = UDim2.new(0.25, 0, 0.45, 0)  -- ตำแหน่ง 25%, 45%

-- NumberRange: ช่วงตัวเลข
local speedRange = NumberRange.new(10, 50)

-- NumberSequence: ลำดับตัวเลข
local alphaSequence = NumberSequence.new({
    NumberSequenceKeypoint.new(0, 1),    -- เริ่มต้น opacity = 1
    NumberSequenceKeypoint.new(0.5, 0.5), -- กลาง opacity = 0.5
    NumberSequenceKeypoint.new(1, 0)     -- สิ้นสุด opacity = 0
})
```

### ตัวอย่างการใช้งานจริง

```lua
-- สร้าง Part พร้อมกำหนดค่าต่างๆ
local function createColoredPart(name, position, color, size)
    local part = Instance.new("Part")
    part.Name = name
    part.Position = position   -- Vector3
    part.BrickColor = BrickColor.new(color)  -- BrickColor
    part.Size = size           -- Vector3
    part.Anchored = true       -- boolean
    part.Parent = game.Workspace
    
    return part
end

-- สร้างแพลตฟอร์มสีแดงที่ตำแหน่ง (0, 5, 0)
local platform = createColoredPart(
    "แพลตฟอร์มสีแดง",
    Vector3.new(0, 5, 0),
    "Bright red",
    Vector3.new(10, 1, 10)
)

print("สร้าง: " .. platform.Name)
print("ตำแหน่ง: " .. tostring(platform.Position))
```

---

## 10.7 การตั้งชื่อตัวแปร (Naming Conventions)

### กฎเกณฑ์การตั้งชื่อ

```lua
-- ถูกต้อง:
local playerName = "สมชาย"
local _privateVar = 10
local camelCase = true
local PascalCase = "สำหรับ classes"
local CONSTANT_VALUE = 100  -- สำหรับค่าคงที่

-- ผิด:
-- local 1stPlayer = "ไม่ได้! ห้ามขึ้นต้นด้วยตัวเลข"
-- local my-var = "ไม่ได้! ห้ามใช้เครื่องหมายลบ"
-- local if = "ไม่ได้! เป็น keyword"
```

### Keywords ที่ห้ามใช้เป็นชื่อตัวแปร

```lua
-- Keywords ใน Lua (ห้ามใช้เป็นชื่อตัวแปร):
-- and, break, do, else, elseif, end
-- false, for, function, goto, if, in
-- local, nil, not, or, repeat, return
-- then, true, until, while
```

### Convention ที่ใช้ใน Roblox

```lua
-- camelCase สำหรับตัวแปรทั่วไปและฟังก์ชัน
local playerScore = 0
local gameRunning = false

local function getPlayerData() end
local function updateScore() end

-- PascalCase สำหรับ Classes, Modules, Services
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local PlayerModule = require(ReplicatedStorage.PlayerModule)

-- UPPER_CASE สำหรับค่าคงที่
local MAX_PLAYERS = 10
local GRAVITY = 196.2
local GAME_VERSION = "1.0.0"

-- _ นำหน้าสำหรับ private หรือ unused
local _internalCache = {}
local function _helperFunction() end

-- ใช้ชื่อที่สื่อความหมาย
local n = 5                           -- ไม่ดี
local numberOfActivePlayers = 5       -- ดี!
local hp = 100                        -- ไม่ดี (แต่ยอมรับได้ในบางบริบท)
local playerHealthPoints = 100        -- ดีที่สุด
```

---

## 10.8 Multiple Assignment

Lua รองรับการกำหนดค่าหลายตัวแปรพร้อมกัน

```lua
-- กำหนดหลายค่าพร้อมกัน
local x, y, z = 1, 2, 3
print(x, y, z)  -- แสดง: 1  2  3

-- สลับค่าตัวแปร (Swap)
local a, b = 10, 20
print(a, b)  -- 10  20

a, b = b, a  -- สลับค่า
print(a, b)  -- 20  10

-- รับค่าจากฟังก์ชันที่ return หลายค่า
local function getMinMax(numbers)
    local min = numbers[1]
    local max = numbers[1]
    
    for _, num in ipairs(numbers) do
        if num < min then min = num end
        if num > max then max = num end
    end
    
    return min, max
end

local scores = {45, 78, 23, 91, 67, 34}
local minScore, maxScore = getMinMax(scores)
print("คะแนนต่ำสุด: " .. minScore)   -- 23
print("คะแนนสูงสุด: " .. maxScore)   -- 91

-- ถ้า return น้อยกว่า variable ที่รับ
local p, q, r = 1, 2
print(p, q, r)  -- 1  2  nil

-- ถ้า return มากกว่า variable ที่รับ
local m, n = 1, 2, 3  -- 3 ถูกทิ้ง
print(m, n)  -- 1  2
```

---

## 10.9 ค่าคงที่ (Constants)

Lua ไม่มี keyword `const` แต่เราสามารถใช้ convention เพื่อระบุค่าคงที่:

```lua
-- ค่าคงที่ด้วย UPPER_CASE convention
local MAX_HEALTH = 100
local MIN_SPEED = 0
local SPAWN_HEIGHT = 10
local GAME_TITLE = "My Awesome Roblox Game"

-- หรือเก็บในตาราง
local Constants = {
    MAX_PLAYERS = 20,
    MIN_LEVEL = 1,
    MAX_LEVEL = 100,
    BASE_DAMAGE = 10,
    RESPAWN_TIME = 5,
    
    -- สี
    TEAM_RED = Color3.fromRGB(255, 50, 50),
    TEAM_BLUE = Color3.fromRGB(50, 50, 255),
    
    -- ข้อความ
    GAME_OVER_MESSAGE = "เกมจบแล้ว!",
    WINNER_MESSAGE = "ผู้ชนะคือ: "
}

-- การใช้งาน
print(Constants.MAX_PLAYERS)  -- 20
print(Constants.GAME_OVER_MESSAGE)  -- เกมจบแล้ว!
```

---

## 10.10 ตัวอย่างโปรเจกต์: ระบบตัวละคร

นำทุกอย่างที่เรียนมารวมกัน:

```lua
-- Script: CharacterSystem.lua
-- ระบบจัดการข้อมูลตัวละครผู้เล่น

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- ค่าคงที่
local DEFAULT_HEALTH = 100
local DEFAULT_SPEED = 16
local DEFAULT_JUMP_POWER = 50
local RESPAWN_TIME = 5

-- ชนิดข้อมูลสำหรับ CharacterData
local CharacterData = {}

-- สร้างข้อมูลตัวละครเริ่มต้น
local function createDefaultCharacterData(playerName)
    return {
        -- ข้อมูลพื้นฐาน
        name = playerName,          -- string
        level = 1,                  -- number
        experience = 0,             -- number
        
        -- สถิติ
        health = DEFAULT_HEALTH,    -- number
        maxHealth = DEFAULT_HEALTH, -- number
        speed = DEFAULT_SPEED,      -- number
        jumpPower = DEFAULT_JUMP_POWER, -- number
        
        -- สถานะ
        isAlive = true,             -- boolean
        isInCombat = false,         -- boolean
        currentWeapon = nil,        -- nil (ยังไม่มีอาวุธ)
        
        -- สิ่งของ
        inventory = {},             -- table (ว่างเปล่า)
        equippedItems = {
            head = nil,
            body = nil,
            weapon = nil
        },
        
        -- สถิติการเล่น
        kills = 0,                  -- number
        deaths = 0,                 -- number
        playTime = 0                -- number (วินาที)
    }
end

-- อัปเดตข้อมูลตัวละคร
local function updateCharacterStat(charData, stat, value)
    if charData[stat] == nil then
        print("Warning: ไม่พบ stat: " .. tostring(stat))
        return false
    end
    
    local oldValue = charData[stat]
    charData[stat] = value
    
    -- ตรวจสอบค่า Health
    if stat == "health" then
        charData.health = math.max(0, math.min(value, charData.maxHealth))
        charData.isAlive = charData.health > 0
    end
    
    print(stat .. ": " .. tostring(oldValue) .. " -> " .. tostring(charData[stat]))
    return true
end

-- แสดงข้อมูลตัวละคร
local function displayCharacterInfo(charData)
    print("=== ข้อมูลตัวละคร ===")
    print("ชื่อ: " .. charData.name)
    print("เลเวล: " .. charData.level)
    print("EXP: " .. charData.experience)
    print("เลือด: " .. charData.health .. "/" .. charData.maxHealth)
    print("ความเร็ว: " .. charData.speed)
    print("มีชีวิต: " .. tostring(charData.isAlive))
    print("Kills/Deaths: " .. charData.kills .. "/" .. charData.deaths)
    print("====================")
end

-- ทดสอบระบบ
local testCharacter = createDefaultCharacterData("สมชาย")
displayCharacterInfo(testCharacter)

-- ทดสอบการอัปเดตค่า
updateCharacterStat(testCharacter, "health", 75)
updateCharacterStat(testCharacter, "level", 5)
updateCharacterStat(testCharacter, "kills", testCharacter.kills + 1)

displayCharacterInfo(testCharacter)
```

---

## 10.11 แบบฝึกหัด

### แบบฝึกหัดที่ 1: ตรวจสอบชนิดข้อมูล

เขียนโค้ดที่รับค่าต่างๆ และแสดงชนิดข้อมูล:

```lua
-- ฝึกทำ:
local values = {42, "สวัสดี", true, nil, {1,2,3}, print, 3.14}

for i, v in ipairs(values) do
    -- ให้แสดง: "ค่าที่ i: v มีชนิดข้อมูล: type"
    -- เติมโค้ดที่นี่
end
```

### แบบฝึกหัดที่ 2: ระบบผู้เล่น

สร้างตัวแปรสำหรับเก็บข้อมูลผู้เล่น:
- ชื่อ (string)
- คะแนน (number)  
- เลเวล (number)
- สถานะ online (boolean)
- รายการไอเทม (table)

```lua
-- เติมโค้ดที่นี่
local player = {
    -- เติมข้อมูล
}

-- แสดงข้อมูลทั้งหมด
```

### แบบฝึกหัดที่ 3: การแปลงชนิดข้อมูล

```lua
-- 1. แปลง string เป็น number และคำนวณ
local strNum1 = "100"
local strNum2 = "50"
-- คำนวณผลรวมและแสดงผล

-- 2. แปลง boolean เป็น string
local gameStatus = true
-- แสดง: "สถานะเกม: กำลังเล่น" หรือ "สถานะเกม: หยุด"

-- 3. ตรวจสอบว่าสามารถแปลงได้หรือไม่
local inputs = {"123", "4.56", "hello", "true", "0"}
-- ตรวจสอบแต่ละค่าว่าแปลงเป็น number ได้หรือไม่
```

---

## สรุป

ในบทนี้เราได้เรียนรู้:

| หัวข้อ | สิ่งที่เรียนรู้ |
|--------|--------------|
| ตัวแปร | local vs global, การประกาศและกำหนดค่า |
| ชนิดข้อมูล | nil, boolean, number, string, table, function, userdata, thread |
| การแปลงชนิด | tonumber(), tostring(), การแปลงอัตโนมัติ |
| การตรวจสอบ | type(), typeof() |
| ขอบเขต | local scope, global scope, closure |
| Convention | camelCase, PascalCase, UPPER_CASE |

### สิ่งสำคัญที่ต้องจำ

1. **ใช้ `local` เสมอ** เพื่อประสิทธิภาพและป้องกันบั๊ก
2. **ตรวจสอบ nil** ก่อนใช้งานตัวแปรเสมอ
3. **ตั้งชื่อให้สื่อความหมาย** เพื่อให้โค้ดอ่านง่าย
4. **ใช้ `type()` หรือ `typeof()`** เมื่อต้องตรวจสอบชนิดข้อมูล
5. **Roblox types** เช่น Vector3, CFrame, Color3 เป็น userdata

### บทถัดไป

ในบทที่ 11 เราจะเรียนเรื่อง **Operators and Expressions** - ตัวดำเนินการและนิพจน์ต่างๆ ที่ใช้ในการคำนวณและเปรียบเทียบค่า
