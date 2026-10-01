# ตอนที่ 15: การจัดการ String ใน Lua

## บทนำ

String manipulation เป็นส่วนสำคัญในการพัฒนาเกม ไม่ว่าจะเป็นการแสดงข้อความให้ผู้เล่น การจัดการชื่อ การ format ตัวเลข หรือการ parse ข้อมูล Lua มี string library ที่ครบครันสำหรับงานเหล่านี้

---

## 15.1 String Methods พื้นฐาน

```lua
local str = "Hello, Roblox World!"

-- ความยาว
print(#str)                     -- 20
print(string.len(str))          -- 20

-- แปลงตัวพิมพ์
print(string.upper(str))        -- HELLO, ROBLOX WORLD!
print(string.lower(str))        -- hello, roblox world!

-- ตัดส่วน (substring)
print(string.sub(str, 1, 5))    -- Hello
print(string.sub(str, 8, 13))   -- Roblox
print(string.sub(str, -6))      -- World! (จากท้าย)
print(string.sub(str, -6, -2))  -- World

-- กลับหัว
print(string.reverse(str))      -- !dlroW xolboR ,olleH

-- ทำซ้ำ
print(string.rep("ab", 3))      -- ababab
print(string.rep("ab", 3, "-")) -- ab-ab-ab (มี separator)

-- หา byte value
print(string.byte("A"))         -- 65
print(string.byte("Hello", 1))  -- 72 (H)
print(string.byte("Hello", 1, 3))  -- 72 101 108

-- แปลง byte เป็น char
print(string.char(72, 101, 108, 108, 111))  -- Hello
```

### Method Syntax

```lua
-- ใช้ได้สองแบบ (equivalent)
local text = "hello world"

-- แบบที่ 1: string.function(str, ...)
print(string.upper(text))

-- แบบที่ 2: str:function(...) (method syntax)
print(text:upper())

-- ทั้งสองแบบให้ผลเหมือนกัน
print(string.sub(text, 1, 5))
print(text:sub(1, 5))
```

---

## 15.2 String.find และ String.match

### string.find

```lua
local str = "Hello Roblox World"

-- หาตำแหน่ง substring
local start, finish = string.find(str, "Roblox")
print(start, finish)  -- 7  12

-- หาจากตำแหน่งที่กำหนด
local s2, e2 = string.find(str, "o", 5)  -- หา "o" จาก index 5
print(s2, e2)  -- 8  8

-- ไม่พบ
local s3, e3 = string.find(str, "Python")
print(s3, e3)  -- nil  nil

-- ตรวจสอบว่ามี substring
local function contains(str, sub)
    return string.find(str, sub, 1, true) ~= nil
end

print(contains("Hello World", "World"))   -- true
print(contains("Hello World", "Python"))  -- false
```

### string.match

```lua
-- หาและ return ส่วนที่ match
local str = "Today is 2024-01-15"

-- Pattern พื้นฐาน
local year = string.match(str, "%d+")  -- หาตัวเลขแรก
print(year)  -- 2024

-- Capture groups
local y, m, d = string.match(str, "(%d+)-(%d+)-(%d+)")
print(y, m, d)  -- 2024  01  15

-- Pattern ต่างๆ
local email = "user@example.com"
local user, domain = string.match(email, "(.+)@(.+)")
print(user, domain)  -- user  example.com
```

---

## 15.3 Pattern Matching

Lua ใช้ pattern matching แทน Regular Expression:

```lua
-- Character classes
-- %d  ตัวเลข (0-9)
-- %a  ตัวอักษร (a-z, A-Z)
-- %l  lowercase
-- %u  uppercase
-- %s  whitespace
-- %p  punctuation
-- %w  alphanumeric
-- %c  control characters
-- .   ทุกอย่างยกเว้น newline

-- Quantifiers
-- *   0 ครั้งขึ้นไป
-- +   1 ครั้งขึ้นไป
-- ?   0 หรือ 1 ครั้ง
-- -   0 ครั้งขึ้นไป (non-greedy)

-- ตัวอย่าง
local function extractNumbers(str)
    local numbers = {}
    for num in string.gmatch(str, "%d+") do
        table.insert(numbers, tonumber(num))
    end
    return numbers
end

local nums = extractNumbers("ฉันมี 5 ลูกแอปเปิ้ล 3 กล้วย และ 10 ส้ม")
for _, n in ipairs(nums) do io.write(n .. " ") end
-- 5 3 10

-- ตรวจสอบ format
local function isValidEmail(email)
    return string.match(email, "^[%w%.]+@[%w%.]+%.[%a]+$") ~= nil
end

print(isValidEmail("user@example.com"))      -- true
print(isValidEmail("invalid.email"))         -- false
print(isValidEmail("test@test.co.th"))       -- true

-- ตรวจสอบตัวเลข
local function isNumber(str)
    return string.match(str, "^%-?%d+%.?%d*$") ~= nil
end

print(isNumber("123"))    -- true
print(isNumber("-45.6"))  -- true
print(isNumber("12.3.4")) -- false
print(isNumber("abc"))    -- false
```

---

## 15.4 string.gmatch

วน loop ผ่านทุก match:

```lua
-- แยกคำ
local sentence = "สวัสดี โลก ยินดีต้อนรับ สู่ Roblox"
print("คำในประโยค:")
for word in string.gmatch(sentence, "%S+") do
    print("  - " .. word)
end

-- แยก CSV
local csv = "สมชาย,25,กรุงเทพ,นักพัฒนา"
local fields = {}
for field in string.gmatch(csv, "[^,]+") do
    table.insert(fields, field)
end
print("ชื่อ: " .. fields[1])
print("อายุ: " .. fields[2])
print("เมือง: " .. fields[3])

-- หา key=value pairs
local config = "name=Roblox version=2024 mode=debug"
local settings = {}
for key, value in string.gmatch(config, "(%w+)=(%w+)") do
    settings[key] = value
end

for k, v in pairs(settings) do
    print(k .. " = " .. v)
end
```

---

## 15.5 string.gsub

แทนที่ pattern ด้วยค่าอื่น:

```lua
-- แทนที่อย่างง่าย
local str = "Hello World Hello"
print(string.gsub(str, "Hello", "Goodbye"))  -- Goodbye World Goodbye  2

-- จำกัดจำนวนการแทนที่
print(string.gsub(str, "Hello", "Hi", 1))    -- Hi World Hello  1

-- ใช้กับ pattern
local text = "ราคา 100 บาท และ 200 บาท"
-- เพิ่ม 10% ให้ทุกราคา
local result = string.gsub(text, "%d+", function(n)
    return tostring(math.floor(tonumber(n) * 1.1))
end)
print(result)  -- ราคา 110 บาท และ 220 บาท

-- ลบ whitespace
local function trim(str)
    return string.gsub(str, "^%s*(.-)%s*$", "%1")
end

print(trim("  hello world  "))  -- "hello world"
print(trim("\t\n  text  \n"))   -- "text"

-- Sanitize HTML
local function escapeHTML(str)
    str = string.gsub(str, "&", "&amp;")
    str = string.gsub(str, "<", "&lt;")
    str = string.gsub(str, ">", "&gt;")
    str = string.gsub(str, '"', "&quot;")
    return str
end

print(escapeHTML("<script>alert('xss')</script>"))
-- &lt;script&gt;alert(&#39;xss&#39;)&lt;/script&gt;
```

---

## 15.6 string.format

การ format string ที่มีประสิทธิภาพ:

```lua
-- Format specifiers
-- %s  string
-- %d  integer
-- %f  float
-- %e  scientific notation
-- %g  shorter of %e or %f
-- %i  integer
-- %u  unsigned integer
-- %x  hex lowercase
-- %X  hex uppercase
-- %o  octal
-- %q  quoted string
-- %%  literal %

-- ตัวอย่างพื้นฐาน
print(string.format("ชื่อ: %s", "สมชาย"))
print(string.format("เลเวล: %d", 15))
print(string.format("ตัวเลข: %.2f", 3.14159))
print(string.format("Hex: %X", 255))       -- FF
print(string.format("Percent: %.1f%%", 75.5))  -- Percent: 75.5%

-- Padding
print(string.format("%10s", "right"))  -- "     right"
print(string.format("%-10s", "left"))  -- "left      "
print(string.format("%05d", 42))       -- "00042"

-- หลายค่า
print(string.format("%s (%d) - %.0f%%", "ผู้เล่น", 5, 87.3))
-- ผู้เล่น (5) - 87%

-- Format เวลา
local function formatTime(seconds)
    local hours = math.floor(seconds / 3600)
    local mins = math.floor((seconds % 3600) / 60)
    local secs = seconds % 60
    
    if hours > 0 then
        return string.format("%02d:%02d:%02d", hours, mins, secs)
    else
        return string.format("%02d:%02d", mins, secs)
    end
end

print(formatTime(90))    -- 01:30
print(formatTime(3661))  -- 01:01:01
print(formatTime(45))    -- 00:45
```

---

## 15.7 String Utilities สำหรับ Roblox

```lua
-- ===== String Utilities =====

local StringUtils = {}

-- Trim whitespace
function StringUtils.trim(str)
    return str:match("^%s*(.-)%s*$")
end

-- Split string
function StringUtils.split(str, sep)
    sep = sep or "%s"
    local parts = {}
    for part in str:gmatch("[^" .. sep .. "]+") do
        table.insert(parts, part)
    end
    return parts
end

-- Starts with
function StringUtils.startsWith(str, prefix)
    return str:sub(1, #prefix) == prefix
end

-- Ends with
function StringUtils.endsWith(str, suffix)
    return str:sub(-#suffix) == suffix
end

-- Pad string
function StringUtils.padLeft(str, length, char)
    char = char or " "
    while #str < length do
        str = char .. str
    end
    return str
end

function StringUtils.padRight(str, length, char)
    char = char or " "
    while #str < length do
        str = str .. char
    end
    return str
end

-- Count occurrences
function StringUtils.count(str, pattern)
    local count = 0
    for _ in str:gmatch(pattern) do
        count = count + 1
    end
    return count
end

-- Replace all
function StringUtils.replace(str, find, replace)
    -- escape pattern characters
    find = find:gsub("([%(%)%.%%%+%-%*%?%[%^%$])", "%%%1")
    return str:gsub(find, replace)
end

-- Format number with commas
function StringUtils.formatNumber(n)
    local s = tostring(math.floor(n))
    local result = ""
    local len = #s
    
    for i = 1, len do
        if i > 1 and (len - i + 1) % 3 == 0 then
            result = result .. ","
        end
        result = result .. s:sub(i, i)
    end
    
    return result
end

-- Truncate
function StringUtils.truncate(str, maxLen, suffix)
    suffix = suffix or "..."
    if #str <= maxLen then
        return str
    end
    return str:sub(1, maxLen - #suffix) .. suffix
end

-- Capitalize
function StringUtils.capitalize(str)
    return str:sub(1, 1):upper() .. str:sub(2):lower()
end

-- Title case
function StringUtils.titleCase(str)
    return str:gsub("(%a)([%w_']*)", function(first, rest)
        return first:upper() .. rest:lower()
    end)
end

-- ทดสอบ
print(StringUtils.trim("  hello world  "))       -- "hello world"
print(StringUtils.split("a,b,c,d", ",")[2])      -- b
print(StringUtils.startsWith("Hello", "He"))      -- true
print(StringUtils.endsWith("World", "ld"))         -- true
print(StringUtils.padLeft("42", 5, "0"))           -- 00042
print(StringUtils.count("hello world hello", "hello"))  -- 2
print(StringUtils.formatNumber(1234567))          -- 1,234,567
print(StringUtils.truncate("Long text here", 10)) -- Long text...
print(StringUtils.titleCase("hello world"))        -- Hello World
```

---

## 15.8 การใช้งานจริงใน Roblox

### Chat Filter System

```lua
-- Script: ChatFilter.lua

local bannedWords = {"คำหยาบ1", "คำหยาบ2", "spam"}
local bannedPatterns = {
    "http[s]?://",     -- URLs
    "discord%.gg",     -- Discord invites
    "www%.",           -- websites
}

local function containsBannedWord(message)
    local lowerMsg = message:lower()
    
    -- ตรวจสอบคำต้องห้าม
    for _, word in ipairs(bannedWords) do
        if lowerMsg:find(word, 1, true) then
            return true, "คำต้องห้าม: " .. word
        end
    end
    
    -- ตรวจสอบ patterns
    for _, pattern in ipairs(bannedPatterns) do
        if lowerMsg:find(pattern) then
            return true, "ลิงก์/URL ไม่อนุญาต"
        end
    end
    
    return false, nil
end

local function filterMessage(message, playerName)
    -- ตรวจสอบความยาว
    if #message > 200 then
        return false, "ข้อความยาวเกินไป (สูงสุด 200 ตัวอักษร)"
    end
    
    if #message == 0 then
        return false, "ข้อความว่างเปล่า"
    end
    
    -- ตรวจสอบ banned content
    local isBanned, reason = containsBannedWord(message)
    if isBanned then
        print("[ChatFilter] บล็อก " .. playerName .. ": " .. reason)
        return false, "ข้อความของคุณถูกบล็อก"
    end
    
    return true, message
end

-- ทดสอบ
local testMessages = {
    "สวัสดีทุกคน!",
    "ตรวจสอบ http://example.com ด้วย",
    "สนุกมากเลย",
    string.rep("a", 250),  -- ยาวเกินไป
}

for _, msg in ipairs(testMessages) do
    local ok, result = filterMessage(msg, "TestPlayer")
    if ok then
        print("✓ ส่งได้: " .. result:sub(1, 30))
    else
        print("✗ บล็อก: " .. result)
    end
end
```

### Player Name Formatter

```lua
-- Script: NameFormatter.lua

-- ฟอร์แมตชื่อผู้เล่นให้สวยงาม
local function formatPlayerTag(player, data)
    local title = data.title or ""
    local name = player.Name
    local level = data.level or 1
    local team = data.teamName or ""
    
    -- สร้าง tag
    local tag = ""
    
    if team ~= "" then
        tag = tag .. "[" .. team .. "] "
    end
    
    if title ~= "" then
        tag = tag .. title .. " "
    end
    
    tag = tag .. name
    tag = tag .. " Lv." .. level
    
    return tag
end

-- Format scoreboard
local function formatScoreRow(rank, name, score, kills, deaths)
    return string.format(
        "%-4s %-20s %-8s %-8s %-8s",
        rank .. ".",
        name:sub(1, 20),  -- ตัดถ้ายาวเกิน
        score,
        kills,
        deaths
    )
end

-- แสดง scoreboard
local function displayScoreboard(players)
    local header = string.format(
        "%-4s %-20s %-8s %-8s %-8s",
        "#", "ชื่อ", "คะแนน", "Kill", "Death"
    )
    
    print(header)
    print(string.rep("-", 52))
    
    for i, p in ipairs(players) do
        local kd = p.deaths > 0 
            and string.format("%.1f", p.kills / p.deaths)
            or p.kills .. ".0"
        
        print(formatScoreRow(i, p.name, p.score, p.kills, p.deaths))
    end
end

-- ทดสอบ
local players = {
    {name = "สมชาย", score = 1250, kills = 15, deaths = 5},
    {name = "LongNamePlayer", score = 980, kills = 12, deaths = 8},
    {name = "Pro_Gamer_Thai", score = 750, kills = 8, deaths = 10},
    {name = "น้องใหม่", score = 200, kills = 3, deaths = 15},
}

table.sort(players, function(a, b) return a.score > b.score end)
displayScoreboard(players)
```

---

## 15.9 Unicode และภาษาไทย

```lua
-- ภาษาไทยใช้ UTF-8 ซึ่งแต่ละตัวอักษรอาจใช้หลาย bytes
local thai = "สวัสดี"
print(#thai)        -- ตัวเลขที่มากกว่าจำนวนตัวอักษรที่เห็น (ประมาณ 18 bytes)

-- ตรวจสอบจำนวนตัวอักษรจริงๆ (UTF-8)
local function utf8Length(str)
    local len = 0
    local i = 1
    while i <= #str do
        local byte = str:byte(i)
        if byte < 0x80 then
            i = i + 1
        elseif byte < 0xE0 then
            i = i + 2
        elseif byte < 0xF0 then
            i = i + 3
        else
            i = i + 4
        end
        len = len + 1
    end
    return len
end

print(utf8Length("Hello"))   -- 5
print(utf8Length("สวัสดี"))   -- 6

-- การทำงานกับข้อความภาษาไทยใน Roblox ปกติทำได้ดี
-- แต่ต้องระวังเรื่อง string operations บางอย่างที่นับ bytes
local playerInput = "สมชาย"
local isValidLength = utf8Length(playerInput) >= 2 and utf8Length(playerInput) <= 20
print("ชื่อถูกต้อง: " .. tostring(isValidLength))
```

---

## 15.10 แบบฝึกหัด

### แบบฝึกหัดที่ 1: Text Parser

```lua
-- Parse text command จาก chat
-- Format: "/command arg1 arg2 arg3"
-- ตัวอย่าง: "/give สมชาย ดาบ 1"

local function parseCommand(message)
    -- ตรวจสอบว่าเป็น command (ขึ้นต้นด้วย /)
    -- แยก command name และ arguments
    -- Return {command, args} หรือ nil ถ้าไม่ใช่ command
end

local commands = {
    "/give สมชาย ดาบ 1",
    "/kick BadPlayer",
    "/heal all",
    "ข้อความปกติ",
    "/ban Cheater 1d สาเหตุ"
}

for _, cmd in ipairs(commands) do
    local result = parseCommand(cmd)
    if result then
        print("Command: " .. result.command)
        print("Args: " .. table.concat(result.args, ", "))
    else
        print("ไม่ใช่ command: " .. cmd)
    end
    print()
end
```

### แบบฝึกหัดที่ 2: Template Engine

```lua
-- สร้าง simple template engine
-- ใช้ {{variable}} เป็น placeholder

local function render(template, data)
    -- แทนที่ {{key}} ด้วยค่าจาก data
    -- ถ้าไม่พบ key ให้ใช้ "" แทน
end

local template = "สวัสดี {{name}}! คุณอยู่ Level {{level}} มีคะแนน {{score}} คะแนน"
local data = {name = "สมชาย", level = 15, score = 1250}

print(render(template, data))
-- สวัสดี สมชาย! คุณอยู่ Level 15 มีคะแนน 1250 คะแนน
```

### แบบฝึกหัดที่ 3: String Analyzer

```lua
-- วิเคราะห์ข้อความและบอก:
-- - จำนวนตัวอักษร (ไม่รวม space)
-- - จำนวนคำ
-- - จำนวนตัวเลข
-- - คำที่ยาวที่สุด

local function analyzeText(text)
    -- เติมโค้ด
end

local text = "ฉันมี 5 แมว และ 3 สุนัข อาศัยอยู่ที่กรุงเทพมหานคร"
local analysis = analyzeText(text)
-- แสดงผลการวิเคราะห์
```

---

## สรุป

| Function | การใช้งาน |
|----------|----------|
| `string.upper/lower` | แปลง case |
| `string.sub(s, i, j)` | ตัด substring |
| `string.len(s)` หรือ `#s` | ความยาว |
| `string.find(s, p)` | หาตำแหน่ง |
| `string.match(s, p)` | หา match แรก |
| `string.gmatch(s, p)` | วน loop matches |
| `string.gsub(s, p, r)` | แทนที่ |
| `string.format(fmt, ...)` | format string |
| `string.rep(s, n)` | ทำซ้ำ |
| `string.reverse(s)` | กลับหัว |
| `string.byte/char` | แปลง char/byte |

### บทถัดไป

ในบทที่ 16 เราจะเรียนเรื่อง **Math Library** - ฟังก์ชันคณิตศาสตร์ใน Lua
