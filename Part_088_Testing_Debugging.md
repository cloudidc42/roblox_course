# Part 88: Testing and Debugging - การทดสอบและแก้ไขบัค

## บทนำ

การทดสอบและ debugging เป็นทักษะสำคัญที่ทำให้เกมมีคุณภาพสูง นักพัฒนาที่ดีใช้เวลากับการทดสอบไม่น้อยกว่าการเขียนโค้ด ในบทนี้เราจะเรียนรู้วิธีทดสอบอย่างมีระบบ

---

## ส่วนที่ 1: Unit Testing

### 1.1 หลักการ Unit Testing

Unit Testing คือการทดสอบฟังก์ชันหรือ module แต่ละชิ้นแบบแยกส่วน

```lua
-- ModuleScript: TestFramework
-- Framework สำหรับการทดสอบ

local TestFramework = {}

local results = {
    passed = 0,
    failed = 0,
    errors = 0,
    tests = {},
}

-- ฟังก์ชัน assert พื้นฐาน
function TestFramework.assertEqual(actual, expected, message)
    if actual ~= expected then
        error(string.format("❌ %s\n  Expected: %s\n  Got: %s",
            message or "Assertion failed",
            tostring(expected),
            tostring(actual)
        ))
    end
end

function TestFramework.assertNotEqual(actual, unexpected, message)
    if actual == unexpected then
        error(string.format("❌ %s\n  Should not equal: %s",
            message or "Assertion failed",
            tostring(unexpected)
        ))
    end
end

function TestFramework.assertTrue(condition, message)
    if not condition then
        error(string.format("❌ %s\n  Expected true, got false",
            message or "Assertion failed"
        ))
    end
end

function TestFramework.assertFalse(condition, message)
    if condition then
        error(string.format("❌ %s\n  Expected false, got true",
            message or "Assertion failed"
        ))
    end
end

function TestFramework.assertNil(value, message)
    if value ~= nil then
        error(string.format("❌ %s\n  Expected nil, got: %s",
            message or "Assertion failed",
            tostring(value)
        ))
    end
end

function TestFramework.assertNotNil(value, message)
    if value == nil then
        error(string.format("❌ %s\n  Expected non-nil value",
            message or "Assertion failed"
        ))
    end
end

function TestFramework.assertTableEqual(actual, expected, message)
    for key, value in pairs(expected) do
        if actual[key] ~= value then
            error(string.format("❌ %s\n  Key '%s': Expected %s, Got %s",
                message or "Table assertion failed",
                tostring(key),
                tostring(value),
                tostring(actual[key])
            ))
        end
    end
end

-- รันการทดสอบ
function TestFramework.test(name, testFn)
    local success, err = pcall(testFn)
    
    local result = {
        name = name,
        passed = success,
        error = err,
    }
    
    table.insert(results.tests, result)
    
    if success then
        results.passed += 1
        print(string.format("✅ %s", name))
    else
        results.failed += 1
        print(string.format("❌ %s\n   %s", name, tostring(err)))
    end
end

-- จัดกลุ่มการทดสอบ
function TestFramework.describe(groupName, testsFn)
    print(string.format("\n📋 %s", groupName))
    testsFn()
end

-- แสดงสรุปผล
function TestFramework.summary()
    print(string.format("\n=============================="))
    print(string.format("📊 ผลการทดสอบ:"))
    print(string.format("  ✅ ผ่าน: %d", results.passed))
    print(string.format("  ❌ ล้มเหลว: %d", results.failed))
    print(string.format("  รวมทั้งหมด: %d", results.passed + results.failed))
    
    if results.failed == 0 then
        print("🎉 ผ่านทุกการทดสอบ!")
    else
        print(string.format("⚠️  มี %d การทดสอบที่ล้มเหลว", results.failed))
    end
    print("==============================\n")
    
    return results
end

function TestFramework.reset()
    results = {passed = 0, failed = 0, errors = 0, tests = {}}
end

return TestFramework
```

### 1.2 ตัวอย่าง Unit Tests

```lua
-- Script: RunTests (Script ใน ServerScriptService)
-- รันการทดสอบทั้งหมด

local Test = require(game.ServerStorage.TestFramework)

-- ========================================
-- ทดสอบ Math Utility
-- ========================================
local MathUtil = require(game.ServerStorage.MathUtil)

Test.describe("MathUtil Tests", function()
    Test.test("clamp ค่าให้อยู่ในช่วง", function()
        Test.assertEqual(MathUtil.clamp(5, 0, 10), 5)
        Test.assertEqual(MathUtil.clamp(-5, 0, 10), 0)
        Test.assertEqual(MathUtil.clamp(15, 0, 10), 10)
    end)
    
    Test.test("lerp interpolate ค่าระหว่างสอง", function()
        Test.assertEqual(MathUtil.lerp(0, 10, 0), 0)
        Test.assertEqual(MathUtil.lerp(0, 10, 1), 10)
        Test.assertEqual(MathUtil.lerp(0, 10, 0.5), 5)
    end)
    
    Test.test("round ปัดเศษ", function()
        Test.assertEqual(MathUtil.round(1.4), 1)
        Test.assertEqual(MathUtil.round(1.5), 2)
        Test.assertEqual(MathUtil.round(-1.5), -1)
    end)
end)

-- ========================================
-- ทดสอบ Inventory System
-- ========================================
local InventorySystem = require(game.ServerStorage.InventorySystem)

Test.describe("Inventory System Tests", function()
    -- สร้าง mock data
    local mockData = {
        Inventory = {},
        MaxInventorySlots = 50,
    }
    
    Test.test("เพิ่มไอเทมได้", function()
        local success = InventorySystem.addItem(mockData, "sword", 1)
        Test.assertTrue(success, "ควรเพิ่มไอเทมได้")
        Test.assertEqual(mockData.Inventory["sword"], 1, "ควรมี 1 sword")
    end)
    
    Test.test("เพิ่มไอเทมซ้ำได้", function()
        InventorySystem.addItem(mockData, "sword", 2)
        Test.assertEqual(mockData.Inventory["sword"], 3, "ควรมี 3 sword")
    end)
    
    Test.test("ลบไอเทมได้", function()
        local success = InventorySystem.removeItem(mockData, "sword", 2)
        Test.assertTrue(success, "ควรลบไอเทมได้")
        Test.assertEqual(mockData.Inventory["sword"], 1, "ควรเหลือ 1 sword")
    end)
    
    Test.test("ลบไอเทมที่ไม่มีล้มเหลว", function()
        local success = InventorySystem.removeItem(mockData, "magic_staff", 1)
        Test.assertFalse(success, "ควรล้มเหลวเมื่อไม่มีไอเทม")
    end)
    
    Test.test("ตรวจสอบ full inventory", function()
        mockData.MaxInventorySlots = 1
        mockData.Inventory = {}
        
        InventorySystem.addItem(mockData, "item1", 1)
        local canAdd = InventorySystem.canAddItem(mockData, "item2")
        Test.assertFalse(canAdd, "ไม่ควรเพิ่มไอเทมเมื่อเต็ม")
        
        mockData.MaxInventorySlots = 50 -- reset
    end)
end)

-- ========================================
-- ทดสอบ Economy System
-- ========================================
Test.describe("Economy System Tests", function()
    local mockData = {
        Coins = 1000,
        Gems = 50,
    }
    
    Test.test("เพิ่มเหรียญ", function()
        local success = EconomySystem.addCoins(mockData, 500)
        Test.assertTrue(success)
        Test.assertEqual(mockData.Coins, 1500)
    end)
    
    Test.test("ใช้เหรียญเพียงพอ", function()
        local success = EconomySystem.spendCoins(mockData, 200)
        Test.assertTrue(success)
        Test.assertEqual(mockData.Coins, 1300)
    end)
    
    Test.test("ใช้เหรียญไม่พอ", function()
        local success, err = EconomySystem.spendCoins(mockData, 9999)
        Test.assertFalse(success)
        Test.assertNotNil(err)
    end)
    
    Test.test("เหรียญไม่ติดลบ", function()
        mockData.Coins = 100
        EconomySystem.spendCoins(mockData, 100)
        Test.assertEqual(mockData.Coins, 0)
        
        local success = EconomySystem.spendCoins(mockData, 1)
        Test.assertFalse(success)
    end)
end)

-- ========================================
-- แสดงผล
-- ========================================
Test.summary()
```

---

## ส่วนที่ 2: Integration Testing

```lua
-- Script: IntegrationTests
-- ทดสอบการทำงานร่วมกันของระบบต่างๆ

local Test = require(game.ServerStorage.TestFramework)

-- Mock Player object
local function createMockPlayer(userId, name)
    local mockPlayer = {
        UserId = userId,
        Name = name,
        IsDescendantOf = function(self, parent)
            return true -- เสมือนว่าอยู่ใน Players
        end,
        Character = nil,
    }
    return mockPlayer
end

Test.describe("DataStore Integration", function()
    Test.test("บันทึกและโหลดข้อมูลสอดคล้องกัน", function()
        local testPlayer = createMockPlayer(12345, "TestPlayer")
        
        -- จำลองการ save/load
        local savedData = {
            Coins = 500,
            Level = 5,
            Inventory = {["Sword"] = 2}
        }
        
        -- ทดสอบ deep copy
        local loadedData = deepCopy(savedData)
        Test.assertEqual(loadedData.Coins, 500)
        Test.assertEqual(loadedData.Level, 5)
        Test.assertEqual(loadedData.Inventory["Sword"], 2)
        
        -- ทดสอบว่า modification ไม่กระทบ original
        loadedData.Coins = 1000
        Test.assertEqual(savedData.Coins, 500, "Original ไม่ควรเปลี่ยน")
    end)
    
    Test.test("Merge กับ default data ถูกต้อง", function()
        local partialData = {Coins = 100}
        local defaults = {Coins = 0, Gems = 0, Level = 1}
        
        local merged = mergeWithDefaults(partialData, defaults)
        
        Test.assertEqual(merged.Coins, 100, "Coins ควรคงค่าเดิม")
        Test.assertEqual(merged.Gems, 0, "Gems ควรใช้ default")
        Test.assertEqual(merged.Level, 1, "Level ควรใช้ default")
    end)
end)

Test.describe("GamePass Integration", function()
    Test.test("ตรวจสอบ pass ownership ถูกต้อง", function()
        -- ในการทดสอบ production ต้องใช้ mock
        -- สำหรับบทนี้เป็นตัวอย่าง structure
        
        local mockPassOwnership = {
            [12345] = {[111111101] = true, [111111102] = false}
        }
        
        local function mockHasPass(userId, passId)
            return mockPassOwnership[userId] and 
                   mockPassOwnership[userId][passId] == true
        end
        
        Test.assertTrue(mockHasPass(12345, 111111101), "ควรมี pass 101")
        Test.assertFalse(mockHasPass(12345, 111111102), "ไม่ควรมี pass 102")
        Test.assertFalse(mockHasPass(99999, 111111101), "ผู้เล่นอื่นไม่ควรมี pass")
    end)
end)

Test.summary()
```

---

## ส่วนที่ 3: Performance Testing

```lua
-- Script: PerformanceTests
-- ทดสอบ performance ของโค้ด

local function benchmark(name, iterations, testFn)
    local startTime = os.clock()
    
    for i = 1, iterations do
        testFn(i)
    end
    
    local endTime = os.clock()
    local totalTime = endTime - startTime
    local avgTime = totalTime / iterations
    
    print(string.format(
        "⏱️  %s\n   รวม: %.4f วินาที | เฉลี่ย: %.6f วินาที | %d iterations",
        name, totalTime, avgTime, iterations
    ))
    
    return totalTime, avgTime
end

-- ทดสอบ string concatenation vs table.concat
benchmark("String concatenation", 10000, function(i)
    local s = ""
    for j = 1, 100 do
        s = s .. "x"
    end
end)

benchmark("table.concat", 10000, function(i)
    local parts = {}
    for j = 1, 100 do
        table.insert(parts, "x")
    end
    local s = table.concat(parts)
end)

-- ทดสอบ ipairs vs numeric for
local testTable = {}
for i = 1, 1000 do
    testTable[i] = i
end

benchmark("ipairs", 100000, function()
    local sum = 0
    for _, v in ipairs(testTable) do
        sum = sum + v
    end
end)

benchmark("numeric for", 100000, function()
    local sum = 0
    for i = 1, #testTable do
        sum = sum + testTable[i]
    end
end)

-- ทดสอบ table lookup vs if-else chain
local lookup = {A = 1, B = 2, C = 3, D = 4, E = 5}
local keys = {"A", "B", "C", "D", "E", "A", "C"}

benchmark("table lookup", 100000, function(i)
    local key = keys[(i % #keys) + 1]
    local val = lookup[key]
end)

benchmark("if-else chain", 100000, function(i)
    local key = keys[(i % #keys) + 1]
    local val
    if key == "A" then val = 1
    elseif key == "B" then val = 2
    elseif key == "C" then val = 3
    elseif key == "D" then val = 4
    elseif key == "E" then val = 5
    end
end)
```

---

## ส่วนที่ 4: Debug Utilities

```lua
-- ModuleScript: DebugUtil
-- เครื่องมือ debug ที่มีประโยชน์

local DebugUtil = {}

-- เปิด/ปิด debug mode
DebugUtil.DEBUG_MODE = true

-- Log ระดับต่างๆ
local LOG_LEVELS = {
    DEBUG = {level = 0, prefix = "🔍 DEBUG"},
    INFO = {level = 1, prefix = "ℹ️  INFO"},
    WARN = {level = 2, prefix = "⚠️  WARN"},
    ERROR = {level = 3, prefix = "❌ ERROR"},
}

local currentLogLevel = LOG_LEVELS.DEBUG

function DebugUtil.setLogLevel(levelName)
    currentLogLevel = LOG_LEVELS[levelName] or LOG_LEVELS.DEBUG
end

local function log(level, ...)
    if not DebugUtil.DEBUG_MODE then return end
    if level.level < currentLogLevel.level then return end
    
    local args = {...}
    local parts = {}
    for _, arg in ipairs(args) do
        if type(arg) == "table" then
            table.insert(parts, DebugUtil.tableToString(arg))
        else
            table.insert(parts, tostring(arg))
        end
    end
    
    local timestamp = string.format("[%.2f]", os.clock())
    print(string.format("%s %s %s", timestamp, level.prefix, table.concat(parts, " ")))
end

function DebugUtil.debug(...) log(LOG_LEVELS.DEBUG, ...) end
function DebugUtil.info(...) log(LOG_LEVELS.INFO, ...) end
function DebugUtil.warn(...) log(LOG_LEVELS.WARN, ...) end
function DebugUtil.error(...) log(LOG_LEVELS.ERROR, ...) end

-- แปลง table เป็น string (สำหรับ debug)
function DebugUtil.tableToString(t, indent)
    indent = indent or 0
    
    if type(t) ~= "table" then
        return tostring(t)
    end
    
    local spaces = string.rep("  ", indent)
    local parts = {"{"}
    
    for key, value in pairs(t) do
        local keyStr = type(key) == "string" and key or "[" .. tostring(key) .. "]"
        local valueStr
        
        if type(value) == "table" then
            valueStr = DebugUtil.tableToString(value, indent + 1)
        else
            valueStr = tostring(value)
        end
        
        table.insert(parts, string.format("%s  %s = %s,", spaces, keyStr, valueStr))
    end
    
    table.insert(parts, spaces .. "}")
    return table.concat(parts, "\n")
end

-- Profile function execution time
function DebugUtil.profile(name, fn, ...)
    local startTime = os.clock()
    local results = {pcall(fn, ...)}
    local endTime = os.clock()
    
    local success = table.remove(results, 1)
    
    DebugUtil.debug(string.format("⏱️  %s ใช้เวลา: %.4fms", 
        name, (endTime - startTime) * 1000
    ))
    
    if not success then
        DebugUtil.error("Function failed:", results[1])
        return nil
    end
    
    return table.unpack(results)
end

-- ตรวจสอบ memory usage
function DebugUtil.checkMemory(label)
    local memory = collectgarbage("count")
    DebugUtil.info(string.format("💾 %s Memory: %.2f KB", label or "", memory))
    return memory
end

-- Stack trace
function DebugUtil.trace(message)
    if not DebugUtil.DEBUG_MODE then return end
    
    local traceback = debug.traceback(message or "Stack trace:", 2)
    print(traceback)
end

-- Watch variable changes
local watchers = {}

function DebugUtil.watch(obj, property, callback)
    -- ใช้ proxy pattern เพื่อ watch property changes
    -- (Roblox มี __newindex หากเป็น table)
    local key = tostring(obj) .. "_" .. property
    watchers[key] = {
        obj = obj,
        property = property,
        lastValue = obj[property],
        callback = callback or function(old, new)
            DebugUtil.debug(string.format("Property '%s' เปลี่ยน: %s -> %s",
                property, tostring(old), tostring(new)
            ))
        end,
    }
end

function DebugUtil.checkWatchers()
    for key, watcher in pairs(watchers) do
        local current = watcher.obj[watcher.property]
        if current ~= watcher.lastValue then
            watcher.callback(watcher.lastValue, current)
            watcher.lastValue = current
        end
    end
end

return DebugUtil
```

---

## ส่วนที่ 5: Visual Debugging Tools

```lua
-- LocalScript: VisualDebugger
-- เครื่องมือ debug แบบ visual

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local DEBUG_MODE = true -- เปิด/ปิดด้วย shift+F1

-- ==============================
-- Debug Overlay
-- ==============================
local function createDebugOverlay()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "DebugOverlay"
    screenGui.ResetOnSpawn = false
    screenGui.Parent = player.PlayerGui
    
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(0, 250, 0, 200)
    frame.Position = UDim2.new(0, 5, 0, 5)
    frame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    frame.BackgroundTransparency = 0.5
    frame.Visible = DEBUG_MODE
    frame.Parent = screenGui
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 8)
    corner.Parent = frame
    
    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, 0, 0, 25)
    title.BackgroundColor3 = Color3.fromRGB(0, 50, 100)
    title.Text = "🔍 Debug Info"
    title.TextColor3 = Color3.new(1, 1, 1)
    title.TextSize = 14
    title.Font = Enum.Font.GothamBold
    title.Parent = frame
    
    -- Debug lines
    local lines = {}
    local lineHeight = 18
    
    local function createLine(index, label)
        local line = Instance.new("TextLabel")
        line.Size = UDim2.new(1, -10, 0, lineHeight)
        line.Position = UDim2.new(0, 5, 0, 25 + (index - 1) * lineHeight)
        line.BackgroundTransparency = 1
        line.TextColor3 = Color3.fromRGB(200, 255, 200)
        line.TextSize = 11
        line.Font = Enum.Font.RobotoMono
        line.TextXAlignment = Enum.TextXAlignment.Left
        line.Parent = frame
        lines[label] = line
        return line
    end
    
    createLine(1, "fps")
    createLine(2, "pos")
    createLine(3, "speed")
    createLine(4, "ping")
    createLine(5, "memory")
    createLine(6, "players")
    createLine(7, "tick")
    
    -- อัพเดท debug info
    local frameCount = 0
    local lastTime = os.clock()
    local fps = 60
    
    RunService.Heartbeat:Connect(function()
        frameCount += 1
        local now = os.clock()
        
        if now - lastTime >= 0.5 then
            fps = frameCount / (now - lastTime)
            frameCount = 0
            lastTime = now
        end
        
        local character = player.Character
        local hrp = character and character:FindFirstChild("HumanoidRootPart")
        local pos = hrp and hrp.Position or Vector3.zero
        local humanoid = character and character:FindFirstChildOfClass("Humanoid")
        
        if lines.fps then
            local color = fps >= 55 and Color3.fromRGB(100, 255, 100)
                or fps >= 30 and Color3.fromRGB(255, 255, 0)
                or Color3.fromRGB(255, 100, 100)
            lines.fps.Text = string.format("FPS: %.0f", fps)
            lines.fps.TextColor3 = color
        end
        
        if lines.pos then
            lines.pos.Text = string.format("Pos: %.1f, %.1f, %.1f", pos.X, pos.Y, pos.Z)
        end
        
        if lines.speed then
            local speed = humanoid and humanoid.WalkSpeed or 0
            lines.speed.Text = string.format("Speed: %.1f", speed)
        end
        
        if lines.memory then
            lines.memory.Text = string.format("Memory: %.1fMB", collectgarbage("count") / 1024)
        end
        
        if lines.players then
            lines.players.Text = string.format("Players: %d", #Players:GetPlayers())
        end
        
        if lines.tick then
            lines.tick.Text = string.format("Tick: %.2f", tick())
        end
    end)
    
    -- Toggle debug overlay
    UserInputService.InputBegan:Connect(function(input, gameProcessed)
        if gameProcessed then return end
        if input.KeyCode == Enum.KeyCode.F1 
            and UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then
            frame.Visible = not frame.Visible
        end
    end)
    
    return frame
end

createDebugOverlay()
```

---

## ส่วนที่ 6: Bug Tracking System

```lua
-- ModuleScript: BugTracker
-- ระบบติดตามบัค

local DataStoreService = game:GetService("DataStoreService")
local bugStore = DataStoreService:GetDataStore("BugReports")

local BugTracker = {}

-- บันทึก bug โดยอัตโนมัติ
function BugTracker:AutoCapture(errorMessage, source, level)
    local bugReport = {
        id = string.format("BUG_%d_%d", os.time(), math.random(1000)),
        message = errorMessage,
        source = source or "Unknown",
        level = level or 0,
        timestamp = os.time(),
        placeId = game.PlaceId,
        placeVersion = game.PlaceVersion,
    }
    
    pcall(function()
        bugStore:SetAsync(bugReport.id, bugReport)
    end)
    
    warn(string.format("[BugTracker] จับบัค: %s\n  ที่: %s", 
        errorMessage, source or "Unknown"
    ))
end

-- ผู้เล่นรายงานบัค
function BugTracker:PlayerReport(player, description, category)
    local report = {
        id = string.format("REPORT_%d_%d", player.UserId, os.time()),
        reporterId = player.UserId,
        reporterName = player.Name,
        description = description,
        category = category or "general",
        timestamp = os.time(),
        placeId = game.PlaceId,
    }
    
    pcall(function()
        bugStore:SetAsync(report.id, report)
    end)
    
    print(string.format("[BugTracker] รายงานจาก %s: %s", player.Name, description))
    return report.id
end

-- จัดการ script errors
local function onError(message, trace)
    BugTracker:AutoCapture(message, trace)
end

-- เชื่อมกับ error events (ถ้า Roblox รองรับ)
-- game:GetService("ScriptContext").Error:Connect(onError)

return BugTracker
```

---

## ส่วนที่ 7: Testing Best Practices

### Checklist ก่อน Release

```
✅ Unit tests ผ่านทั้งหมด
✅ Integration tests ผ่านทั้งหมด  
✅ ทดสอบกับผู้เล่น 1 คน
✅ ทดสอบกับผู้เล่น 10 คน
✅ ทดสอบกับผู้เล่น 50+ คน
✅ ทดสอบ edge cases (เข้า-ออก เร็ว)
✅ ทดสอบ network lag (ใช้ Emulation)
✅ ทดสอบบน Mobile
✅ ทดสอบบน Xbox
✅ ทดสอบ DataStore failures
✅ ทดสอบ การซื้อ GamePass/DevProduct
✅ ตรวจสอบ memory leaks
✅ ตรวจสอบ performance (FPS)
```

### Common Bugs และวิธีแก้

```lua
-- Bug 1: Memory Leak จาก event connections
-- ❌ ผิด
local connection = player.CharacterAdded:Connect(function(char)
    -- ไม่ disconnect เมื่อ player ออก
end)

-- ✅ ถูก
local connections = {}
table.insert(connections, player.CharacterAdded:Connect(function(char)
    -- จัดการ
end))

Players.PlayerRemoving:Connect(function(leavingPlayer)
    if leavingPlayer == player then
        for _, conn in ipairs(connections) do
            conn:Disconnect()
        end
    end
end)

-- Bug 2: Race Condition ใน async operations
-- ❌ ผิด
local data = getDataAsync()
player.Character.Humanoid.MaxHealth = data.maxHealth -- อาจ error ถ้า character ตายก่อน

-- ✅ ถูก
local data = getDataAsync()
if player.Character and player.Character:FindFirstChildOfClass("Humanoid") then
    player.Character.Humanoid.MaxHealth = data.maxHealth
end

-- Bug 3: NaN และ Infinity
-- ❌ ผิด
local multiplier = someValue / 0 -- Infinity
humanoid.MaxHealth = humanoid.MaxHealth * multiplier

-- ✅ ถูก
local function safeMultiply(value, multiplier)
    if multiplier ~= multiplier or math.abs(multiplier) == math.huge then
        return value -- return unchanged
    end
    return value * multiplier
end
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: เขียน Unit Tests
เขียน tests สำหรับ:
1. Inventory system ทั้งหมด
2. Economy calculations
3. Level progression

### แบบฝึกหัดที่ 2: Performance Profiling
ทำ benchmark:
1. เปรียบเทียบ array vs dictionary lookup
2. วัด overhead ของ RemoteEvents
3. หา bottleneck ใน game loop

### แบบฝึกหัดที่ 3: Bug Report UI
สร้าง UI สำหรับผู้เล่นรายงานบัค:
1. เลือก category
2. พิมพ์คำอธิบาย
3. แนบ screenshot
4. ส่งไปยัง Discord webhook

---

## สรุปบทที่ 88

การทดสอบที่ดีประกอบด้วย:

1. **Unit Tests** - ทดสอบทุก function อย่างแยกส่วน
2. **Integration Tests** - ทดสอบการทำงานร่วมกัน
3. **Performance Tests** - ตรวจสอบ FPS, memory
4. **User Testing** - ทดสอบกับผู้เล่นจริง
5. **Bug Tracking** - ติดตามและแก้ไขอย่างเป็นระบบ

*บทถัดไป: Part 89 - Git and Version Control*
