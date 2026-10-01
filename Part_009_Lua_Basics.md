# ตอนที่ 9: Lua Programming เบื้องต้น
## Part 9: Introduction to Lua Programming

---

**ระดับ:** พื้นฐาน (Beginner)  
**เวลาเรียน:** 120-180 นาที  
**ข้อกำหนดเบื้องต้น:** ตอนที่ 1-8

---

## 🎯 วัตถุประสงค์การเรียนรู้

เมื่อเรียนจบตอนนี้ คุณจะสามารถ:
1. เข้าใจพื้นฐานของภาษา Lua
2. เขียน Comments และ Print statements
3. ใช้ Variables และ Data Types
4. เขียน Functions พื้นฐาน
5. ทำความเข้าใจ Scope ของตัวแปร
6. ใช้ String Operations พื้นฐาน

---

## 1. ทำไมต้องเรียน Lua?

### 1.1 Lua ในบริบท Roblox

Roblox ใช้ **Luau** (Lua 5.1 + Extensions):
- ภาษาเดียวสำหรับทุกส่วนของเกม
- เรียนรู้ง่าย ทำงานได้เร็ว
- ต่อยอดไปยังภาษาอื่นได้ (Python, JavaScript, C#)

### 1.2 Lua vs Python (เปรียบเทียบ Syntax)

```lua
-- Lua
local message = "สวัสดี"
print(message)

if true then
    print("เป็นจริง")
end
```

```python
# Python
message = "สวัสดี"
print(message)

if True:
    print("เป็นจริง")
```

Lua คล้าย Python แต่:
- ใช้ `local` ประกาศตัวแปร
- ใช้ `then` และ `end` แทน indentation
- Index เริ่มต้นที่ 1 (ไม่ใช่ 0!)
- ใช้ `~=` แทน `!=` สำหรับ Not Equal

---

## 2. Comments (ความคิดเห็น)

### 2.1 Single-line Comment

```lua
-- นี่คือ Comment บรรทัดเดียว
-- This is a single-line comment

print("Hello!")  -- Comment ท้ายบรรทัดก็ได้

-- ===== SECTION TITLE =====
-- การใช้ Comment เพื่อจัดระเบียบโค้ด

-- TODO: เพิ่ม Feature นี้ในอนาคต
-- FIXME: แก้ Bug ในฟังก์ชันนี้
-- NOTE: ระวัง Bug ตรงนี้
-- HACK: แก้ปัญหาชั่วคราว ต้องแก้ให้ถูกต้อง
```

### 2.2 Multi-line Comment

```lua
--[[
    นี่คือ Comment หลายบรรทัด
    This is a multi-line comment
    
    ใช้สำหรับ:
    - อธิบายโค้ดยาวๆ
    - ปิดการทำงาน (Comment out) หลายบรรทัด
    - Documentation
]]

--[[
    ตัวอย่าง Documentation Comment:
    
    Function: calculateDamage
    Parameters:
        baseDamage (number) - ความเสียหายพื้นฐาน
        multiplier (number) - ตัวคูณ
    Returns:
        number - ความเสียหายสุดท้าย
]]
```

### 2.3 แนวปฏิบัติที่ดีสำหรับ Comments

```lua
-- ❌ ไม่ดี: Comment ที่ชัดเจนอยู่แล้ว
local x = 5  -- กำหนด x เป็น 5

-- ✅ ดี: Comment อธิบาย "ทำไม" ไม่ใช่ "อะไร"
local maxHealth = 100  -- ตามมาตรฐาน RPG ทั่วไป

-- ✅ ดี: Comment ที่ซับซ้อน
-- คำนวณ Golden Ratio สำหรับ UI Layout
local goldenRatio = (1 + math.sqrt(5)) / 2  -- ≈ 1.618
```

---

## 3. print() และ Output

### 3.1 print() พื้นฐาน

```lua
-- print() แสดงข้อความใน Output
print("Hello, World!")
print("สวัสดีโลก!")

-- print() หลายค่า (คั่นด้วย Tab)
print("ชื่อ:", "สมชาย", "อายุ:", 25)
-- Output: ชื่อ:	สมชาย	อายุ:	25

-- print() ตัวเลข
print(42)
print(3.14)
print(-100)
```

### 3.2 warn() และ error()

```lua
-- warn() แสดงข้อความสีเหลือง (Warning)
warn("ระวัง! ค่านี้อาจไม่ถูกต้อง")

-- error() หยุดการทำงานและแสดงสีแดง
error("เกิดข้อผิดพลาด! หยุดทำงาน")

-- error() พร้อม Level (ไม่แสดง Stack Trace เพิ่มเติม)
error("ข้อผิดพลาด", 0)  -- ไม่แสดง line number
error("ข้อผิดพลาด", 1)  -- แสดงบรรทัดที่ error (ค่าเริ่มต้น)
error("ข้อผิดพลาด", 2)  -- แสดงบรรทัดที่เรียกฟังก์ชัน
```

### 3.3 tostring() และ String Formatting

```lua
-- แปลงค่าเป็น String
local number = 42
local text = tostring(number)  -- "42"
print(text)

-- String Concatenation (ต่อ String)
local firstName = "สม"
local lastName = "ชาย"
local fullName = firstName .. lastName  -- "สมชาย"
print(fullName)

-- ต่อกับตัวเลข (ต้อง tostring ก่อน)
local score = 100
print("คะแนน: " .. tostring(score))
-- หรือใช้ String.format
print(string.format("คะแนน: %d", score))

-- string.format เหมือน printf ใน C
print(string.format("ชื่อ: %s, อายุ: %d", "สมชาย", 25))
print(string.format("ทศนิยม: %.2f", 3.14159))  -- 3.14
print(string.format("Hex: 0x%X", 255))           -- 0xFF
```

---

## 4. Variables (ตัวแปร)

### 4.1 การประกาศ Variable

```lua
-- ===== LOCAL VARIABLES (แนะนำเสมอ) =====
local myVariable = "ค่าของตัวแปร"
local playerHealth = 100
local isAlive = true
local position = Vector3.new(0, 5, 0)

-- ===== GLOBAL VARIABLES (หลีกเลี่ยง!) =====
globalVariable = "ทั่วทั้ง Script"  -- ไม่มี local
-- ⚠️ Global Variables ช้ากว่า Local มาก!
-- ⚠️ อาจทำให้เกิด Bug ที่หาได้ยาก!
```

### 4.2 Naming Conventions (รูปแบบการตั้งชื่อ)

```lua
-- ✅ camelCase (แนะนำสำหรับ Variables)
local playerName = "สมชาย"
local maxHealth = 100
local isGameOver = false

-- ✅ PascalCase (แนะนำสำหรับ Modules/Classes)
local PlayerModule = require(game.ReplicatedStorage.PlayerModule)

-- ✅ SCREAMING_CASE (แนะนำสำหรับ Constants)
local MAX_PLAYERS = 20
local DEFAULT_SPEED = 16
local GAME_VERSION = "1.0.0"

-- ❌ ไม่แนะนำ
local my_variable = "underscore"  -- ไม่ใช่ Style ของ Lua
local MyVariable = "PascalCase สำหรับ Variable"  -- ทำให้สับสน
```

### 4.3 Multiple Assignment

```lua
-- กำหนดหลาย Variable พร้อมกัน
local x, y, z = 1, 2, 3
print(x, y, z)  -- 1 2 3

-- Swap Values
local a, b = 10, 20
a, b = b, a  -- Swap!
print(a, b)  -- 20 10

-- Function ที่ Return หลายค่า
local function getCoords()
    return 5, 10, 15
end

local px, py, pz = getCoords()
print(px, py, pz)  -- 5 10 15
```

---

## 5. Data Types (ประเภทข้อมูล)

### 5.1 nil

```lua
-- nil = ไม่มีค่า / ว่างเปล่า
local empty = nil
print(empty)          -- nil
print(type(empty))    -- nil

-- ตรวจสอบ nil
if empty == nil then
    print("ว่างเปล่า!")
end

-- ใช้ ~= nil เพื่อตรวจสอบว่ามีค่า
if empty ~= nil then
    print("มีค่า!")
else
    print("ไม่มีค่า!")
end
```

### 5.2 boolean

```lua
-- boolean = true หรือ false เท่านั้น
local isPlaying = true
local isGameOver = false

print(type(isPlaying))   -- boolean
print(isPlaying)         -- true
print(not isPlaying)     -- false (ตรงข้าม)

-- Truthy และ Falsy ใน Lua
-- ❗ ใน Lua: false และ nil = falsy
-- ❗ ทุกค่าอื่น (รวมถึง 0 และ "") = truthy
-- นี่ต่างจาก JavaScript ที่ 0 และ "" เป็น falsy!

if 0 then print("0 เป็น truthy!")  end  -- พิมพ์ออกมา!
if "" then print("'' เป็น truthy!") end  -- พิมพ์ออกมา!
if false then print("false") else print("false คือ falsy") end
if nil then print("nil") else print("nil คือ falsy") end
```

### 5.3 number

```lua
-- number = ตัวเลขทุกประเภท (Integer และ Float รวมกัน)
local integer = 42
local float = 3.14
local negative = -100
local scientific = 1e6  -- 1,000,000

print(type(integer))    -- number (ไม่ใช่ int!)

-- การดำเนินการทางคณิตศาสตร์
print(10 + 3)   -- 13 (บวก)
print(10 - 3)   -- 7  (ลบ)
print(10 * 3)   -- 30 (คูณ)
print(10 / 3)   -- 3.3333... (หาร)
print(10 // 3)  -- 3  (หารปัดลง - Floor Division)
print(10 % 3)   -- 1  (เศษ - Modulo)
print(2 ^ 10)   -- 1024 (ยกกำลัง)

-- Math Library
print(math.floor(3.7))   -- 3  (ปัดลง)
print(math.ceil(3.2))    -- 4  (ปัดขึ้น)
print(math.abs(-5))      -- 5  (ค่าสัมบูรณ์)
print(math.sqrt(16))     -- 4  (รากที่สอง)
print(math.max(1, 5, 3)) -- 5  (ค่าสูงสุด)
print(math.min(1, 5, 3)) -- 1  (ค่าต่ำสุด)
print(math.pi)           -- 3.14159...
print(math.random())     -- 0-1 แบบสุ่ม
print(math.random(1, 6)) -- 1-6 แบบสุ่ม (เหมือน ลูกเต๋า)
```

### 5.4 string

```lua
-- string = ข้อความ
local text1 = "ใช้ Double Quotes"
local text2 = 'ใช้ Single Quotes'
local text3 = [[
    ใช้ Double Brackets
    รองรับหลายบรรทัด
    และ Special Characters ทั้งหมด
]]

print(type(text1))   -- string

-- String Length
print(#"Hello")      -- 5 (ความยาว)
print(#"สวัสดี")    -- 18 (ไทย = 3 bytes/ตัว)

-- String Methods
local str = "Hello, World!"
print(string.upper(str))          -- HELLO, WORLD!
print(string.lower(str))          -- hello, world!
print(string.len(str))            -- 13 (ความยาว)
print(string.sub(str, 1, 5))      -- Hello (ตัดจาก 1 ถึง 5)
print(string.rep("Ha", 3))        -- HaHaHa (ซ้ำ 3 ครั้ง)
print(string.reverse("abc"))      -- cba (กลับหลัง)

-- ค้นหา
print(string.find("Hello World", "World"))  -- 7  11
print(string.find("Hello World", "Roblox")) -- nil

-- แทนที่
print(string.gsub("Hello World", "World", "Roblox"))  -- Hello Roblox  1

-- String Method Syntax (OOP Style)
local greeting = "สวัสดี"
print(greeting:upper())   -- เหมือน string.upper(greeting)
print(greeting:len())     -- เหมือน string.len(greeting)
print(greeting:sub(1, 3)) -- ตัด 3 อักขระแรก
```

### 5.5 table

```lua
-- table = Array หรือ Dictionary
-- (จะเรียนรายละเอียดในตอนที่ 15)

-- Array (ลำดับ)
local fruits = {"แอปเปิ้ล", "กล้วย", "ส้ม"}
print(fruits[1])  -- แอปเปิ้ล (เริ่มต้นที่ 1 !)
print(#fruits)    -- 3 (ความยาว)

-- Dictionary (key-value)
local player = {
    name = "สมชาย",
    health = 100,
    level = 1
}
print(player.name)    -- สมชาย
print(player["health"]) -- 100
```

### 5.6 function

```lua
-- function = ฟังก์ชัน (เป็น First-class value!)
local greet = function(name)
    print("สวัสดี " .. name)
end

greet("สมชาย")  -- สวัสดี สมชาย

-- ส่ง Function เป็น Argument ได้!
local function callFunction(fn, value)
    fn(value)
end

callFunction(greet, "สมหญิง")  -- สวัสดี สมหญิง

-- ประเภทของ type()
print(type(nil))        -- nil
print(type(true))       -- boolean
print(type(42))         -- number
print(type("text"))     -- string
print(type({}))         -- table
print(type(print))      -- function
```

---

## 6. Functions (ฟังก์ชัน)

### 6.1 การสร้าง Function

```lua
-- วิธีที่ 1: Function Statement (ทั่วไป)
local function greet(name)
    print("สวัสดี " .. name)
end

-- วิธีที่ 2: Function Expression
local greet2 = function(name)
    print("สวัสดี " .. name)
end

-- วิธีที่ 3: Anonymous Function (ไม่มีชื่อ)
-- ใช้เป็น Callback
local result = (function(x) return x * 2 end)(5)
print(result)  -- 10

-- เรียกใช้
greet("สมชาย")
greet2("สมชาย")
```

### 6.2 Parameters และ Arguments

```lua
-- Parameters = ตัวแปรใน Function Definition
-- Arguments = ค่าที่ส่งเมื่อเรียก Function

local function addNumbers(a, b)  -- a, b คือ Parameters
    return a + b
end

local sum = addNumbers(3, 4)  -- 3, 4 คือ Arguments
print(sum)  -- 7

-- Default Parameters (ทำเองใน Lua)
local function greetWithTitle(name, title)
    title = title or "คุณ"  -- ถ้าไม่ส่ง title ใช้ "คุณ"
    print(title .. name .. " สวัสดี!")
end

greetWithTitle("สมชาย")          -- คุณสมชาย สวัสดี!
greetWithTitle("สมหญิง", "นาง")  -- นางสมหญิง สวัสดี!
```

### 6.3 Return Values

```lua
-- Return ค่าเดียว
local function double(x)
    return x * 2
end

print(double(5))  -- 10

-- Return หลายค่า
local function getPlayerInfo(player)
    return player.Name, player.UserId, player.AccountAge
end

-- local name, id, age = getPlayerInfo(somePlayer)

-- Return ไม่มีค่า
local function printMessage(msg)
    print(msg)
    -- return  -- ไม่จำเป็นต้องเขียน
end

-- ตรวจสอบ Return Value
local function safeDivide(a, b)
    if b == 0 then
        return nil, "หารด้วย 0 ไม่ได้!"
    end
    return a / b, nil
end

local result, errorMsg = safeDivide(10, 0)
if errorMsg then
    warn(errorMsg)
else
    print("ผลลัพธ์:", result)
end
```

### 6.4 Variadic Functions (รับ Arguments ไม่จำกัด)

```lua
-- ... = Varargs (หลาย Arguments)
local function sum(...)
    local args = {...}  -- เก็บทุก argument ในตาราง
    local total = 0
    for _, value in ipairs(args) do
        total = total + value
    end
    return total
end

print(sum(1, 2, 3))        -- 6
print(sum(1, 2, 3, 4, 5)) -- 15
print(sum(10))             -- 10

-- ใช้ select() กับ Varargs
local function countArgs(...)
    return select("#", ...)  -- นับจำนวน Arguments
end

print(countArgs(1, 2, 3))  -- 3
```

---

## 7. Scope (ขอบเขตตัวแปร)

### 7.1 Local vs Global

```lua
-- Global Scope
globalVar = "Global"  -- ⚠️ ไม่แนะนำ

-- Local Scope
local function test()
    local localVar = "Local"
    print(globalVar)  -- เข้าถึงได้
    print(localVar)   -- เข้าถึงได้
end

test()
print(globalVar)  -- เข้าถึงได้
-- print(localVar)  -- ❌ Error! localVar ไม่มีตรงนี้
```

### 7.2 Block Scope

```lua
-- if block
if true then
    local insideIf = "อยู่ใน if"
    print(insideIf)  -- OK
end
-- print(insideIf)  -- ❌ Error! ออกนอก block แล้ว

-- for loop scope
for i = 1, 5 do
    local loopVar = i * 2
    -- loopVar ใช้ได้แค่ใน loop นี้
end
-- print(loopVar)  -- ❌ Error!

-- do...end block (สร้าง Scope ใหม่ได้)
do
    local tempVar = "ชั่วคราว"
    print(tempVar)  -- OK
end
-- print(tempVar)  -- ❌ Error!
```

### 7.3 Upvalues (Closure)

```lua
-- Function ที่ "จำ" ตัวแปรนอก Scope ได้
local function makeCounter()
    local count = 0  -- Upvalue
    
    return function()  -- Inner function
        count = count + 1  -- เข้าถึง count ได้!
        return count
    end
end

local counter = makeCounter()
print(counter())  -- 1
print(counter())  -- 2
print(counter())  -- 3

-- สร้าง Counter สองตัวแยกกัน
local counter1 = makeCounter()
local counter2 = makeCounter()
print(counter1())  -- 1
print(counter1())  -- 2
print(counter2())  -- 1 (แยกกัน!)
```

---

## 8. String Operations ขั้นสูง

### 8.1 Pattern Matching

```lua
-- Patterns เหมือน RegEx แต่ง่ายกว่า

local text = "สวัสดี 123 โลก"

-- หาตัวเลข
local nums = string.match(text, "%d+")
print(nums)  -- 123

-- หาทุกตัวเลข
for num in string.gmatch(text, "%d+") do
    print(num)
end

-- Patterns:
-- %d = ตัวเลข (digit)
-- %a = ตัวอักษร (alpha)
-- %w = ตัวอักษรหรือตัวเลข (word)
-- %s = Whitespace
-- %l = ตัวพิมพ์เล็ก (lowercase)
-- %u = ตัวพิมพ์ใหญ่ (uppercase)
-- %p = เครื่องหมายวรรคตอน
-- %+,%*,%.  = ตัวอักษรพิเศษ (ต้อง Escape)

-- . = ตัวอักษรใดก็ได้
-- + = 1 ตัวขึ้นไป
-- * = 0 ตัวขึ้นไป
-- ? = 0 หรือ 1 ตัว
-- ^ = ต้นของ string
-- $ = ท้ายของ string
```

### 8.2 String Manipulation

```lua
-- Split String (แยก string ด้วย separator)
local function split(str, sep)
    local result = {}
    local pattern = string.format("([^%s]+)", sep)
    for word in string.gmatch(str, pattern) do
        table.insert(result, word)
    end
    return result
end

local words = split("สวัสดี,โลก,Roblox", ",")
for i, word in ipairs(words) do
    print(i, word)
end
-- 1  สวัสดี
-- 2  โลก
-- 3  Roblox

-- Trim Whitespace
local function trim(str)
    return string.match(str, "^%s*(.-)%s*$")
end

print(trim("  สวัสดี  "))  -- สวัสดี (ไม่มี space)

-- Check if starts/ends with
local function startsWith(str, prefix)
    return string.sub(str, 1, #prefix) == prefix
end

local function endsWith(str, suffix)
    return string.sub(str, -#suffix) == suffix
end

print(startsWith("Hello World", "Hello"))  -- true
print(endsWith("Hello World", "World"))    -- true
```

---

## 9. Error Handling

### 9.1 pcall (Protected Call)

```lua
-- pcall รัน Function แบบ Protected (ไม่ Crash ถ้า Error)
local success, result = pcall(function()
    local x = 10 / 0  -- ไม่ Error ใน Lua (ได้ inf)
    return x
end)

print(success, result)  -- true  inf

-- ตัวอย่างที่ Error จริง
local success2, errorMsg = pcall(function()
    error("เกิดข้อผิดพลาด!")
end)

print(success2)    -- false
print(errorMsg)    -- Script:3: เกิดข้อผิดพลาด!

-- Pattern พื้นฐาน
local ok, err = pcall(function()
    -- โค้ดที่อาจ Error
    local part = workspace.NonExistentPart  -- nil
    part.Size = Vector3.new(1, 1, 1)  -- Error! indexing nil
end)

if not ok then
    warn("เกิด Error:", err)
end
```

### 9.2 xpcall (Extended Protected Call)

```lua
-- xpcall รับ Error Handler เพิ่มเติม
local function errorHandler(err)
    return "Error Handler: " .. tostring(err) .. "\n" .. debug.traceback()
end

local success, result = xpcall(function()
    error("ข้อผิดพลาดทดสอบ")
end, errorHandler)

if not success then
    print(result)  -- แสดง Stack Trace
end
```

---

## 10. ตัวอย่างโปรแกรมสมบูรณ์

### 10.1 Calculator

```lua
-- Script: Calculator
-- วางใน: ServerScriptService

-- ===== Simple Calculator =====

local Calculator = {}

function Calculator.add(a, b)
    return a + b
end

function Calculator.subtract(a, b)
    return a - b
end

function Calculator.multiply(a, b)
    return a * b
end

function Calculator.divide(a, b)
    if b == 0 then
        error("ไม่สามารถหารด้วย 0 ได้!", 2)
    end
    return a / b
end

function Calculator.power(base, exp)
    return base ^ exp
end

function Calculator.sqrt(x)
    if x < 0 then
        error("ไม่สามารถหาค่า sqrt ของจำนวนลบ!", 2)
    end
    return math.sqrt(x)
end

function Calculator.factorial(n)
    if n < 0 then
        error("Factorial ของจำนวนลบไม่มี!")
    end
    if n == 0 then return 1 end
    local result = 1
    for i = 1, n do
        result = result * i
    end
    return result
end

-- Test Calculator
print("=== Calculator Test ===")
print("10 + 5 =", Calculator.add(10, 5))
print("10 - 5 =", Calculator.subtract(10, 5))
print("10 × 5 =", Calculator.multiply(10, 5))
print("10 ÷ 5 =", Calculator.divide(10, 5))
print("2 ^ 10 =", Calculator.power(2, 10))
print("√16 =", Calculator.sqrt(16))
print("5! =", Calculator.factorial(5))

-- Error Handling
local ok, err = pcall(function()
    Calculator.divide(10, 0)
end)
if not ok then
    print("Error:", err)
end

print("======================")
```

### 10.2 Player Stats System

```lua
-- Script: PlayerStats
-- วางใน: ServerScriptService

local Players = game:GetService("Players")

-- ===== Player Stats Manager =====
local PlayerStats = {}
local statsData = {}  -- เก็บ Stats ของแต่ละ Player

-- สร้าง Stats สำหรับ Player
local function createStats(player)
    statsData[player.UserId] = {
        name = player.Name,
        health = 100,
        maxHealth = 100,
        speed = 16,
        level = 1,
        experience = 0,
        kills = 0,
        deaths = 0,
    }
    print("สร้าง Stats สำหรับ " .. player.Name)
end

-- ลบ Stats เมื่อ Player ออก
local function removeStats(player)
    statsData[player.UserId] = nil
    print("ลบ Stats ของ " .. player.Name)
end

-- ดู Stats
function PlayerStats.getStats(player)
    return statsData[player.UserId]
end

-- เพิ่ม Experience
function PlayerStats.addExperience(player, amount)
    local stats = statsData[player.UserId]
    if not stats then return end
    
    stats.experience = stats.experience + amount
    print(player.Name .. " ได้รับ " .. amount .. " EXP!")
    
    -- Check Level Up
    local expNeeded = stats.level * 100
    if stats.experience >= expNeeded then
        stats.experience = stats.experience - expNeeded
        stats.level = stats.level + 1
        stats.maxHealth = stats.maxHealth + 10
        stats.health = stats.maxHealth  -- Heal on level up
        print("🎉 " .. player.Name .. " Level UP! Level " .. stats.level)
    end
end

-- เพิ่ม Kill
function PlayerStats.addKill(player)
    local stats = statsData[player.UserId]
    if stats then
        stats.kills = stats.kills + 1
        PlayerStats.addExperience(player, 50)  -- EXP จาก Kill
    end
end

-- บันทึก Death
function PlayerStats.addDeath(player)
    local stats = statsData[player.UserId]
    if stats then
        stats.deaths = stats.deaths + 1
    end
end

-- แสดง Stats
function PlayerStats.printStats(player)
    local stats = statsData[player.UserId]
    if not stats then
        print("ไม่พบ Stats สำหรับ " .. player.Name)
        return
    end
    
    print("=== Stats: " .. stats.name .. " ===")
    print(string.format("  Level: %d | EXP: %d/%d", 
        stats.level, stats.experience, stats.level * 100))
    print(string.format("  HP: %d/%d | Speed: %d",
        stats.health, stats.maxHealth, stats.speed))
    print(string.format("  Kills: %d | Deaths: %d | K/D: %.1f",
        stats.kills, stats.deaths, 
        stats.deaths > 0 and stats.kills/stats.deaths or stats.kills))
    print("=====================================")
end

-- เชื่อม Events
Players.PlayerAdded:Connect(function(player)
    createStats(player)
    
    -- ทดสอบ Stats
    wait(2)
    PlayerStats.addKill(player)
    PlayerStats.addKill(player)
    PlayerStats.addExperience(player, 70)
    PlayerStats.printStats(player)
end)

Players.PlayerRemoving:Connect(removeStats)

-- สำหรับผู้เล่นที่อยู่แล้ว
for _, player in pairs(Players:GetPlayers()) do
    createStats(player)
end
```

### 10.3 String Utility Library

```lua
-- Script: StringUtils
-- วางใน: ServerScriptService

-- ===== String Utility Library =====
local StringUtils = {}

-- แปลงเป็น Title Case
function StringUtils.toTitleCase(str)
    return (str:gsub("(%a)([%w_']*)", function(first, rest)
        return first:upper() .. rest:lower()
    end))
end

-- นับจำนวน Occurrences
function StringUtils.count(str, pattern)
    local count = 0
    for _ in str:gmatch(pattern) do
        count = count + 1
    end
    return count
end

-- Pad String (เติมช่องว่าง)
function StringUtils.padLeft(str, length, char)
    char = char or " "
    str = tostring(str)
    while #str < length do
        str = char .. str
    end
    return str
end

function StringUtils.padRight(str, length, char)
    char = char or " "
    str = tostring(str)
    while #str < length do
        str = str .. char
    end
    return str
end

-- ตรวจสอบ Number
function StringUtils.isNumber(str)
    return tonumber(str) ~= nil
end

-- Test
print("=== String Utils Test ===")
print(StringUtils.toTitleCase("hello world"))       -- Hello World
print(StringUtils.count("banana", "a"))             -- 3
print(StringUtils.padLeft("42", 6, "0"))            -- 000042
print(StringUtils.padRight("hello", 10, "."))       -- hello.....
print(StringUtils.isNumber("123"))                  -- true
print(StringUtils.isNumber("abc"))                  -- false
print("=========================")
```

---

## 📚 แบบฝึกหัดตอนที่ 9

### แบบฝึกหัดที่ 1: Comments และ Print
1. เขียน Comment แบบ Single-line และ Multi-line
2. ทดสอบ print(), warn(), error()
3. ทดลอง string.format()

### แบบฝึกหัดที่ 2: Variables
1. สร้างตัวแปรทุกประเภท
2. ลอง type() กับทุก Type
3. ทดสอบ Multiple Assignment

### แบบฝึกหัดที่ 3: Numbers
1. ทดสอบ Math Operations ทั้งหมด
2. ใช้ math.random() สร้างเลข Random
3. ทดสอบ math.floor(), math.ceil(), etc.

### แบบฝึกหัดที่ 4: Strings
1. ทดสอบ String Methods ต่างๆ
2. ลอง String Concatenation
3. ทดสอบ String Pattern Matching

### แบบฝึกหัดที่ 5: Functions
1. สร้าง Function ที่รับ Parameter
2. สร้าง Function ที่ Return ค่า
3. ทดสอบ pcall กับ Function ที่ Error

### แบบฝึกหัดที่ 6: โปรเจกต์
สร้าง **Score Calculator** สำหรับเกม:
- Function คำนวณคะแนนจาก Kills, Assists, Deaths
- Function แปลงคะแนนเป็น Rank (Bronze, Silver, Gold, etc.)
- Function แสดงผลลัพธ์แบบ Formatted
- ทดสอบด้วย Players หลายคน

---

## 💡 เคล็ดลับ Lua

1. **ใช้ local เสมอ** - local variables เร็วกว่า global
2. **Truthy/Falsy ต่างจาก ภาษาอื่น** - จำไว้ว่า 0 และ "" เป็น truthy
3. **Index เริ่มที่ 1** - ไม่ใช่ 0 เหมือนภาษาอื่น
4. **String เป็น Immutable** - ทุก String Operation สร้าง String ใหม่
5. **pcall เป็น Best Practice** - ใช้เสมอเมื่อโค้ดอาจ Error

---

## ⏭️ ตอนถัดไป

ในตอนที่ 10 เราจะเรียนรู้:
- Variables อย่างละเอียด
- Data Types ทั้งหมดใน Luau
- Type Checking
- การแปลงค่าระหว่าง Types

---

*ตอนที่ 9/100 | ระดับ: พื้นฐาน | เวลา: 120-180 นาที*
