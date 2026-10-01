# Part 73: Memory Management (การจัดการหน่วยความจำ)

## บทนำ

Memory Management ใน Roblox เป็นสิ่งสำคัญมาก เพราะถ้าเกมใช้ Memory มากเกินไปจะทำให้ Client Crash และ Server ทำงานช้า บทนี้จะสอนวิธีตรวจหา Memory Leak วิธีจัดการ Asset Loading และ Garbage Collection

---

## 1. ทำความเข้าใจ Memory ใน Roblox

### 1.1 ประเภทของ Memory

```lua
-- MemoryTypes.lua - แสดงประเภท Memory ที่ Roblox ใช้
local Stats = game:GetService("Stats")

-- ดู Memory แยกตาม Category
local function showMemoryBreakdown()
    print("=== Memory Breakdown ===")
    print(string.format("Total Memory: %.2f MB", Stats:GetTotalMemoryUsageMb()))
    
    -- Lua Heap: Memory ที่ Lua Scripts ใช้
    -- GraphicsMesh: Memory สำหรับ Meshes
    -- GraphicsTexture: Memory สำหรับ Textures  
    -- Sounds: Memory สำหรับ Audio
    -- Instances: Memory สำหรับ Roblox Instances
    
    -- หมวดหมู่สำคัญ:
    local categories = {
        Enum.DeveloperMemoryTag.Script,
        Enum.DeveloperMemoryTag.Instances,
        Enum.DeveloperMemoryTag.Gui,
        Enum.DeveloperMemoryTag.Animation,
        Enum.DeveloperMemoryTag.Sounds,
        Enum.DeveloperMemoryTag.ModelLODs,
        Enum.DeveloperMemoryTag.PhysicsCollision,
    }
    
    print("\nMemory ตามหมวด:")
    for _, tag in ipairs(categories) do
        local usage = Stats:GetMemoryUsageMbForTag(tag)
        if usage > 0.1 then  -- แสดงเฉพาะที่ > 0.1 MB
            print(string.format("  %-30s: %.2f MB", tostring(tag), usage))
        end
    end
end

showMemoryBreakdown()
```

### 1.2 Lua Garbage Collector

```lua
-- GCExplainer.lua - อธิบายการทำงานของ Garbage Collector

--[[
Lua ใช้ Incremental Garbage Collector (GC)
- GC จะเก็บ Objects ที่ไม่มี Reference อีกต่อไป
- แต่ GC ไม่ได้ทำงานทันที - มีความล่าช้า
- Memory จะค่อยๆ เพิ่มขึ้นระหว่าง GC Cycles

สาเหตุที่ Memory ไม่ลด:
1. ยังมี Reference อยู่ (Memory Leak)
2. GC ยังไม่ทำงาน (ปกติ)
3. Roblox Instances ต้องใช้ :Destroy() ก่อน
]]

-- ตัวอย่าง: Objects ที่ GC เก็บไม่ได้
local leakedTable = {}

local function createObject()
    local obj = {data = string.rep("X", 1000)}  -- Object ใหญ่
    
    -- ❌ Memory Leak: ใส่ใน table global แล้วไม่เคยลบ
    table.insert(leakedTable, obj)
    
    return obj
end

-- ✅ Pattern ที่ถูกต้อง: ใช้ Weak References
local weakCache = setmetatable({}, {__mode = "v"})  -- Weak Values

local function createCachedObject(key)
    if weakCache[key] then
        return weakCache[key]  -- ใช้ cached version
    end
    
    local obj = {data = string.rep("X", 1000), key = key}
    weakCache[key] = obj  -- GC สามารถเก็บได้เมื่อไม่มี Reference อื่น
    return obj
end

-- Force GC (ไม่แนะนำในโค้ดจริง - แค่สำหรับทดสอบ)
local function forceGC()
    collectgarbage("collect")
    print(string.format("GC Count: %d KB", collectgarbage("count")))
end
```

---

## 2. Memory Leak Detection

### 2.1 ระบบตรวจจับ Memory Leak

```lua
-- MemoryLeakDetector.lua (Module Script)
-- ระบบตรวจหา Memory Leak

local Stats = game:GetService("Stats")
local RunService = game:GetService("RunService")

local MemoryLeakDetector = {}
MemoryLeakDetector.__index = MemoryLeakDetector

function MemoryLeakDetector.new(config)
    local self = setmetatable({}, MemoryLeakDetector)
    
    config = config or {}
    self.checkInterval = config.checkInterval or 10   -- วินาที
    self.leakThreshold = config.leakThreshold or 50   -- MB ต่อนาที
    self.alertThreshold = config.alertThreshold or 1500 -- MB
    self.history = {}
    self.maxHistory = 30  -- เก็บ 5 นาที (30 * 10s)
    self.running = false
    
    return self
end

-- เริ่มตรวจสอบ
function MemoryLeakDetector:start()
    if self.running then return end
    self.running = true
    
    task.spawn(function()
        while self.running do
            self:checkMemory()
            task.wait(self.checkInterval)
        end
    end)
    
    print("Memory Leak Detector started")
end

-- หยุดตรวจสอบ
function MemoryLeakDetector:stop()
    self.running = false
end

-- ตรวจสอบ Memory
function MemoryLeakDetector:checkMemory()
    local currentMB = Stats:GetTotalMemoryUsageMb()
    local timestamp = tick()
    
    -- บันทึกประวัติ
    table.insert(self.history, {
        memory = currentMB,
        timestamp = timestamp
    })
    
    -- ลบข้อมูลเก่า
    if #self.history > self.maxHistory then
        table.remove(self.history, 1)
    end
    
    -- ตรวจสอบ Memory สูงเกินไป
    if currentMB > self.alertThreshold then
        warn(string.format(
            "[MEMORY ALERT] Memory สูงถึง %.1f MB! (เกณฑ์: %d MB)",
            currentMB, self.alertThreshold
        ))
    end
    
    -- คำนวณ Memory Leak Rate
    if #self.history >= 6 then  -- ต้องมีข้อมูลอย่างน้อย 1 นาที
        local oldRecord = self.history[#self.history - 5]
        local newRecord = self.history[#self.history]
        
        local timeDiff = newRecord.timestamp - oldRecord.timestamp
        local memDiff = newRecord.memory - oldRecord.memory
        
        -- แปลงเป็น MB ต่อนาที
        local leakRate = (memDiff / timeDiff) * 60
        
        if leakRate > self.leakThreshold then
            warn(string.format(
                "[MEMORY LEAK SUSPECTED] Memory เพิ่มขึ้น %.1f MB/นาที!",
                leakRate
            ))
            self:analyzeLeak()
        end
    end
end

-- วิเคราะห์หาสาเหตุ
function MemoryLeakDetector:analyzeLeak()
    print("\n=== Memory Leak Analysis ===")
    
    -- ตรวจสอบ Instance Count
    local instanceCount = 0
    local guiCount = 0
    local scriptCount = 0
    
    for _, inst in ipairs(game:GetDescendants()) do
        instanceCount = instanceCount + 1
        if inst:IsA("GuiObject") then guiCount = guiCount + 1 end
        if inst:IsA("LuaSourceContainer") then scriptCount = scriptCount + 1 end
    end
    
    print(string.format("  Total Instances: %d", instanceCount))
    print(string.format("  GUI Objects: %d", guiCount))
    print(string.format("  Scripts: %d", scriptCount))
    
    -- ตรวจสอบ Memory ตาม Category
    local scriptMem = Stats:GetMemoryUsageMbForTag(Enum.DeveloperMemoryTag.Script)
    local guiMem = Stats:GetMemoryUsageMbForTag(Enum.DeveloperMemoryTag.Gui)
    local instMem = Stats:GetMemoryUsageMbForTag(Enum.DeveloperMemoryTag.Instances)
    
    print(string.format("  Script Memory: %.2f MB", scriptMem))
    print(string.format("  GUI Memory: %.2f MB", guiMem))
    print(string.format("  Instance Memory: %.2f MB", instMem))
    
    -- แนะนำสาเหตุ
    print("\nสาเหตุที่เป็นไปได้:")
    if scriptMem > 50 then
        print("  ⚠️ Script Memory สูง - ตรวจสอบ Global Variables และ Closures")
    end
    if guiMem > 100 then
        print("  ⚠️ GUI Memory สูง - ตรวจสอบ GUI ที่สร้างแต่ไม่ได้ Destroy")
    end
    if instMem > 200 then
        print("  ⚠️ Instance Memory สูง - ตรวจสอบ Instances ที่ไม่ได้ Destroy")
    end
end

-- ดูกราฟ Memory ใน Console
function MemoryLeakDetector:printGraph()
    if #self.history == 0 then
        print("ไม่มีข้อมูล")
        return
    end
    
    local minMem = math.huge
    local maxMem = 0
    
    for _, record in ipairs(self.history) do
        minMem = math.min(minMem, record.memory)
        maxMem = math.max(maxMem, record.memory)
    end
    
    local range = maxMem - minMem
    if range < 1 then range = 1 end
    
    print(string.format("\nMemory Graph (%.1f - %.1f MB):", minMem, maxMem))
    
    local GRAPH_HEIGHT = 5
    local GRAPH_WIDTH = math.min(#self.history, 40)
    
    for row = GRAPH_HEIGHT, 1, -1 do
        local line = ""
        local threshold = minMem + (range * row / GRAPH_HEIGHT)
        
        for col = 1, GRAPH_WIDTH do
            local idx = #self.history - GRAPH_WIDTH + col
            if idx > 0 and self.history[idx].memory >= threshold then
                line = line .. "█"
            else
                line = line .. " "
            end
        end
        
        print(string.format("  %.0fMB |%s|", threshold, line))
    end
    
    print(string.format("       └%s┘", string.rep("─", GRAPH_WIDTH)))
end

return MemoryLeakDetector
```

---

## 3. Asset Loading และ Caching

### 3.1 Smart Asset Manager

```lua
-- AssetManager.lua (Module Script)
-- ระบบโหลดและ Cache Assets อย่างมีประสิทธิภาพ

local ContentProvider = game:GetService("ContentProvider")
local AssetService = game:GetService("AssetService")

local AssetManager = {}
AssetManager.__index = AssetManager

function AssetManager.new()
    local self = setmetatable({}, AssetManager)
    
    -- Cache สำหรับ Assets ที่โหลดแล้ว
    -- ใช้ Weak Keys เพื่อให้ GC เก็บได้เมื่อไม่ใช้แล้ว
    self.imageCache = setmetatable({}, {__mode = "v"})
    self.modelCache = setmetatable({}, {__mode = "v"})
    
    self.loadingQueue = {}   -- Queue สำหรับโหลด Assets
    self.isLoading = false   -- กำลังโหลดอยู่หรือไม่
    self.stats = {
        cacheHits = 0,
        cacheMisses = 0,
        totalLoaded = 0,
        totalFailed = 0
    }
    
    return self
end

-- โหลด Asset แบบ Async
function AssetManager:loadAsync(assetId, assetType)
    assetType = assetType or "Image"
    
    local cache = assetType == "Model" and self.modelCache or self.imageCache
    
    -- ตรวจสอบ Cache
    if cache[assetId] then
        self.stats.cacheHits = self.stats.cacheHits + 1
        return cache[assetId]
    end
    
    self.stats.cacheMisses = self.stats.cacheMisses + 1
    
    -- โหลด Asset
    local success, result = pcall(function()
        if assetType == "Model" then
            return AssetService:CreatePlaceAsync and 
                   game:GetObjects("rbxassetid://" .. assetId)[1]
        else
            -- สำหรับ Image - เพียงแค่ Preload
            ContentProvider:PreloadAsync({"rbxassetid://" .. assetId})
            return "rbxassetid://" .. assetId
        end
    end)
    
    if success then
        cache[assetId] = result
        self.stats.totalLoaded = self.stats.totalLoaded + 1
        return result
    else
        self.stats.totalFailed = self.stats.totalFailed + 1
        warn(string.format("โหลด Asset %s ล้มเหลว: %s", assetId, tostring(result)))
        return nil
    end
end

-- Preload รายการ Assets (สำหรับ Loading Screen)
function AssetManager:preloadList(assetList, progressCallback)
    local total = #assetList
    local loaded = 0
    local failed = 0
    
    return task.spawn(function()
        -- โหลดเป็น batches
        local BATCH_SIZE = 20
        
        for i = 1, total, BATCH_SIZE do
            local batch = {}
            
            for j = i, math.min(i + BATCH_SIZE - 1, total) do
                table.insert(batch, "rbxassetid://" .. tostring(assetList[j]))
            end
            
            local success, err = pcall(function()
                ContentProvider:PreloadAsync(batch, function(assetId, status)
                    if status == Enum.AssetFetchStatus.Success then
                        loaded = loaded + 1
                    else
                        failed = failed + 1
                        warn("โหลดไม่สำเร็จ:", assetId)
                    end
                    
                    -- เรียก Callback
                    if progressCallback then
                        progressCallback(loaded + failed, total, loaded, failed)
                    end
                end)
            end)
            
            if not success then
                warn("Batch loading error:", err)
            end
            
            task.wait()  -- yield ระหว่าง batches
        end
        
        print(string.format("Preload เสร็จ: %d/%d สำเร็จ, %d ล้มเหลว", 
            loaded, total, failed))
    end)
end

-- ล้าง Cache ที่ไม่ใช้
function AssetManager:cleanCache()
    -- Weak tables จะถูกล้างอัตโนมัติ แต่เราสามารถ force ได้
    local clearedImages = 0
    local clearedModels = 0
    
    for key, value in pairs(self.imageCache) do
        if value == nil then  -- GC เก็บไปแล้ว
            self.imageCache[key] = nil
            clearedImages = clearedImages + 1
        end
    end
    
    print(string.format("Cache cleaned: %d images, %d models", 
        clearedImages, clearedModels))
end

-- ดูสถิติ
function AssetManager:getStats()
    local hitRate = 0
    local total = self.stats.cacheHits + self.stats.cacheMisses
    if total > 0 then
        hitRate = self.stats.cacheHits / total * 100
    end
    
    return {
        cacheHits = self.stats.cacheHits,
        cacheMisses = self.stats.cacheMisses,
        hitRate = string.format("%.1f%%", hitRate),
        totalLoaded = self.stats.totalLoaded,
        totalFailed = self.stats.totalFailed
    }
end

return AssetManager
```

---

## 4. Instance Lifecycle Management

### 4.1 Instance Tracker

```lua
-- InstanceTracker.lua (Module Script)
-- ติดตาม Lifecycle ของ Instances

local InstanceTracker = {}
InstanceTracker.__index = InstanceTracker

function InstanceTracker.new(name)
    local self = setmetatable({}, InstanceTracker)
    self.name = name or "Tracker"
    self.created = 0
    self.destroyed = 0
    self.tracked = setmetatable({}, {__mode = "k"})  -- Weak Keys
    return self
end

-- Track Instance ใหม่
function InstanceTracker:track(instance, metadata)
    self.created = self.created + 1
    
    self.tracked[instance] = {
        createdAt = tick(),
        metadata = metadata or {},
        stack = debug and debug.traceback() or "N/A"
    }
    
    -- Auto-track destruction
    instance.Destroying:Connect(function()
        self.destroyed = self.destroyed + 1
        self.tracked[instance] = nil
    end)
    
    return instance
end

-- นับ Instances ที่ยังมีอยู่
function InstanceTracker:getLiveCount()
    local count = 0
    for _ in pairs(self.tracked) do
        count = count + 1
    end
    return count
end

-- หา Instances ที่มีชีวิตนานผิดปกติ
function InstanceTracker:findLongLived(maxAge)
    maxAge = maxAge or 300  -- 5 นาที
    local now = tick()
    local longLived = {}
    
    for instance, data in pairs(self.tracked) do
        local age = now - data.createdAt
        if age > maxAge then
            table.insert(longLived, {
                instance = instance,
                age = age,
                metadata = data.metadata,
                createdAt = data.createdAt
            })
        end
    end
    
    table.sort(longLived, function(a, b) return a.age > b.age end)
    return longLived
end

-- รายงานสถานะ
function InstanceTracker:report()
    local liveCount = self:getLiveCount()
    
    print(string.format("\n=== %s Report ===", self.name))
    print(string.format("  Created: %d", self.created))
    print(string.format("  Destroyed: %d", self.destroyed))
    print(string.format("  Live: %d", liveCount))
    print(string.format("  Leak Suspected: %s", 
        liveCount > self.created * 0.5 and "YES ⚠️" or "No"))
    
    local longLived = self:findLongLived()
    if #longLived > 0 then
        print(string.format("  Long-lived Instances: %d", #longLived))
        for i = 1, math.min(3, #longLived) do
            local data = longLived[i]
            print(string.format("    - Age: %.0fs, Metadata: %s", 
                data.age, tostring(data.metadata.name or "unknown")))
        end
    end
end

return InstanceTracker
```

---

## 5. GUI Memory Management

### 5.1 GUI Pool สำหรับ Repeated Elements

```lua
-- GUIPool.lua (Local Module Script)
-- Pool สำหรับ GUI Elements ที่ใช้ซ้ำ

local GUIPool = {}
GUIPool.__index = GUIPool

function GUIPool.new(template, parent, initialSize)
    local self = setmetatable({}, GUIPool)
    self.template = template
    self.parent = parent
    self.available = {}
    self.inUse = {}
    
    -- Pre-create
    for i = 1, (initialSize or 5) do
        local gui = template:Clone()
        gui.Visible = false
        gui.Parent = parent
        table.insert(self.available, gui)
    end
    
    return self
end

-- ขอ GUI Element
function GUIPool:get()
    local gui
    
    if #self.available > 0 then
        gui = table.remove(self.available)
    else
        gui = self.template:Clone()
        gui.Parent = self.parent
    end
    
    gui.Visible = true
    self.inUse[gui] = true
    return gui
end

-- คืน GUI Element
function GUIPool:release(gui)
    if not self.inUse[gui] then return end
    
    gui.Visible = false
    self:resetGui(gui)
    
    self.inUse[gui] = nil
    table.insert(self.available, gui)
end

-- Reset GUI สู่สถานะเริ่มต้น
function GUIPool:resetGui(gui)
    -- Override ตาม GUI type
    if gui:IsA("Frame") then
        -- Reset position, size, etc.
    end
end

-- ตัวอย่างการใช้งาน:
--[[
-- สร้าง Damage Number Pool
local damageTemplate = script.DamageNumberTemplate
local damagePool = GUIPool.new(damageTemplate, playerGui.MainGui, 20)

local function showDamage(position, amount)
    local label = damagePool:get()
    label.Text = tostring(amount)
    
    -- Animate
    local tween = TweenService:Create(label, TweenInfo.new(1), {
        Position = label.Position + UDim2.new(0, 0, -0.1, 0),
        TextTransparency = 1
    })
    
    tween:Play()
    tween.Completed:Connect(function()
        damagePool:release(label)
    end)
end
]]

return GUIPool
```

---

## 6. Data Structure Optimization

### 6.1 ใช้ Data Structures อย่างถูกต้อง

```lua
-- DataStructures.lua - ตัวอย่าง Data Structures ที่ efficient

--[[ 
1. LRU Cache (Least Recently Used)
   - เก็บ N items ล่าสุด
   - เหมาะสำหรับ Cache ขนาดจำกัด
]]

local LRUCache = {}
LRUCache.__index = LRUCache

function LRUCache.new(maxSize)
    return setmetatable({
        maxSize = maxSize,
        cache = {},
        order = {},  -- LRU Order
        size = 0
    }, LRUCache)
end

function LRUCache:get(key)
    if not self.cache[key] then return nil end
    
    -- Move to front (most recently used)
    self:moveToFront(key)
    return self.cache[key]
end

function LRUCache:set(key, value)
    if self.cache[key] then
        -- Update existing
        self.cache[key] = value
        self:moveToFront(key)
        return
    end
    
    -- Evict if full
    if self.size >= self.maxSize then
        local lruKey = table.remove(self.order)  -- Remove last (LRU)
        self.cache[lruKey] = nil
        self.size = self.size - 1
    end
    
    -- Add new
    self.cache[key] = value
    table.insert(self.order, 1, key)  -- Add to front
    self.size = self.size + 1
end

function LRUCache:moveToFront(key)
    for i, k in ipairs(self.order) do
        if k == key then
            table.remove(self.order, i)
            table.insert(self.order, 1, key)
            break
        end
    end
end

function LRUCache:clear()
    self.cache = {}
    self.order = {}
    self.size = 0
end

--[[
2. Spatial Hash - แทน GetDescendants สำหรับ Proximity Check
   - ค้นหา Objects ในพื้นที่เร็วมาก O(1) แทน O(n)
]]

local SpatialHash = {}
SpatialHash.__index = SpatialHash

function SpatialHash.new(cellSize)
    return setmetatable({
        cellSize = cellSize or 20,
        cells = {}
    }, SpatialHash)
end

function SpatialHash:_getCell(position)
    local x = math.floor(position.X / self.cellSize)
    local y = math.floor(position.Y / self.cellSize)
    local z = math.floor(position.Z / self.cellSize)
    return string.format("%d,%d,%d", x, y, z)
end

function SpatialHash:insert(object, position)
    local cell = self:_getCell(position)
    if not self.cells[cell] then
        self.cells[cell] = {}
    end
    self.cells[cell][object] = position
end

function SpatialHash:remove(object, position)
    local cell = self:_getCell(position)
    if self.cells[cell] then
        self.cells[cell][object] = nil
    end
end

function SpatialHash:query(position, radius)
    local results = {}
    local cellRadius = math.ceil(radius / self.cellSize)
    local centerX = math.floor(position.X / self.cellSize)
    local centerY = math.floor(position.Y / self.cellSize)
    local centerZ = math.floor(position.Z / self.cellSize)
    
    for dx = -cellRadius, cellRadius do
        for dy = -cellRadius, cellRadius do
            for dz = -cellRadius, cellRadius do
                local cell = string.format("%d,%d,%d", 
                    centerX + dx, centerY + dy, centerZ + dz)
                
                if self.cells[cell] then
                    for obj, objPos in pairs(self.cells[cell]) do
                        local dist = (objPos - position).Magnitude
                        if dist <= radius then
                            table.insert(results, {object = obj, distance = dist})
                        end
                    end
                end
            end
        end
    end
    
    return results
end

return {LRUCache = LRUCache, SpatialHash = SpatialHash}
```

---

## 7. Server-Side Memory Management

### 7.1 Player Data Cleanup

```lua
-- PlayerMemoryManager.lua (Server Script)
-- จัดการ Memory สำหรับ Player Data

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local PlayerMemoryManager = {}
PlayerMemoryManager.__index = PlayerMemoryManager

function PlayerMemoryManager.new()
    local self = setmetatable({}, PlayerMemoryManager)
    self.playerData = {}      -- ข้อมูล Player
    self.playerObjects = {}   -- Objects ที่สร้างให้ Player
    self.cleanupCallbacks = {} -- Callbacks เมื่อ Player ออก
    return self
end

-- ลงทะเบียน Cleanup Callback
function PlayerMemoryManager:onPlayerLeave(callback)
    table.insert(self.cleanupCallbacks, callback)
end

-- เก็บ Object ที่เชื่อมกับ Player
function PlayerMemoryManager:trackObject(player, object)
    if not self.playerObjects[player] then
        self.playerObjects[player] = {}
    end
    table.insert(self.playerObjects[player], object)
end

-- Setup auto-cleanup เมื่อ Player ออก
function PlayerMemoryManager:setup()
    Players.PlayerAdded:Connect(function(player)
        self.playerData[player] = {}
        self.playerObjects[player] = {}
        
        print(string.format("[MEMORY] Player joined: %s", player.Name))
    end)
    
    Players.PlayerRemoving:Connect(function(player)
        -- เรียก Cleanup Callbacks
        for _, callback in ipairs(self.cleanupCallbacks) do
            local success, err = pcall(callback, player)
            if not success then
                warn(string.format("Cleanup callback failed: %s", err))
            end
        end
        
        -- ลบ Objects ที่สร้างให้ Player
        if self.playerObjects[player] then
            for _, obj in ipairs(self.playerObjects[player]) do
                if typeof(obj) == "Instance" and obj.Parent then
                    obj:Destroy()
                end
            end
            self.playerObjects[player] = nil
        end
        
        -- ล้าง Player Data
        self.playerData[player] = nil
        
        print(string.format("[MEMORY] Cleaned up data for: %s", player.Name))
    end)
    
    -- Monitor Memory ทุก 60 วินาที
    task.spawn(function()
        while true do
            task.wait(60)
            self:reportMemoryUsage()
        end
    end)
end

-- รายงาน Memory
function PlayerMemoryManager:reportMemoryUsage()
    local Stats = game:GetService("Stats")
    local totalMB = Stats:GetTotalMemoryUsageMb()
    local playerCount = #Players:GetPlayers()
    
    print(string.format(
        "[MEMORY REPORT] Total: %.1fMB | Players: %d | Per Player: %.1fMB",
        totalMB, playerCount,
        playerCount > 0 and totalMB / playerCount or 0
    ))
end

return PlayerMemoryManager
```

---

## 8. Common Memory Patterns

### 8.1 Dispose Pattern

```lua
-- DisposableClass.lua - Pattern สำหรับจัดการ Lifecycle

local Disposable = {}
Disposable.__index = Disposable

function Disposable.new()
    local self = setmetatable({}, Disposable)
    self._disposed = false
    self._connections = {}
    self._instances = {}
    self._timers = {}
    return self
end

-- เพิ่ม Connection ให้ auto-disconnect เมื่อ Dispose
function Disposable:addConnection(connection)
    if self._disposed then
        connection:Disconnect()
        return
    end
    table.insert(self._connections, connection)
end

-- เพิ่ม Instance ให้ auto-destroy เมื่อ Dispose
function Disposable:addInstance(instance)
    if self._disposed then
        instance:Destroy()
        return
    end
    table.insert(self._instances, instance)
end

-- เพิ่ม Timer ให้ auto-cancel เมื่อ Dispose
function Disposable:addTimer(thread)
    if self._disposed then
        task.cancel(thread)
        return
    end
    table.insert(self._timers, thread)
end

-- ทำลายทุกอย่าง
function Disposable:dispose()
    if self._disposed then return end
    self._disposed = true
    
    -- Disconnect connections
    for _, conn in ipairs(self._connections) do
        if conn.Connected then
            conn:Disconnect()
        end
    end
    self._connections = {}
    
    -- Destroy instances
    for _, inst in ipairs(self._instances) do
        if typeof(inst) == "Instance" and inst.Parent then
            inst:Destroy()
        end
    end
    self._instances = {}
    
    -- Cancel timers
    for _, thread in ipairs(self._timers) do
        pcall(task.cancel, thread)
    end
    self._timers = {}
    
    print("Disposed!")
end

-- ตรวจสอบว่า Disposed แล้ว
function Disposable:isDisposed()
    return self._disposed
end

return Disposable

--[[
ตัวอย่างการใช้งาน:

local MySystem = setmetatable({}, {__index = Disposable})

function MySystem.new()
    local self = Disposable.new()
    setmetatable(self, {__index = MySystem})
    
    -- Auto-manage connections
    self:addConnection(RunService.Heartbeat:Connect(function()
        -- logic
    end))
    
    -- Auto-manage instances
    local gui = Instance.new("ScreenGui")
    self:addInstance(gui)
    
    -- Auto-manage timers
    local timer = task.delay(5, function()
        -- delayed work
    end)
    self:addTimer(timer)
    
    return self
end

-- เมื่อต้องการลบ:
mySystem:dispose()  -- ล้างทุกอย่างอัตโนมัติ
]]
```

---

## 9. ข้อผิดพลาดที่พบบ่อย

### ❌ Memory Leak ที่พบบ่อยที่สุด

```lua
-- ❌ แบบผิด 1: Closure ที่ถือ Reference ใหญ่
local function badClosure()
    local hugeTable = {}
    for i = 1, 100000 do
        hugeTable[i] = string.rep("data", 100)
    end
    
    -- Closure นี้ถือ hugeTable ไว้ตลอด!
    return function()
        return #hugeTable  -- ใช้แค่ length แต่ถือทั้ง Table
    end
end

-- ✅ แบบถูก: เก็บเฉพาะสิ่งที่ต้องการ
local function goodClosure()
    local hugeTable = {}
    for i = 1, 100000 do
        hugeTable[i] = string.rep("data", 100)
    end
    
    local tableSize = #hugeTable  -- เก็บแค่ Size
    hugeTable = nil  -- ปล่อย Reference
    
    return function()
        return tableSize  -- ใช้แค่ number เล็กๆ
    end
end

-- ❌ แบบผิด 2: Event Connection วนซ้ำ
local function badEventLoop()
    -- สร้าง GUI ใหม่และ connect ทุกครั้งที่เรียก
    local gui = Instance.new("Frame")
    
    -- เมื่อเรียกซ้ำ - สะสม Connections ที่ยังมีอยู่!
    gui.MouseButton1Click:Connect(function()
        print("clicked")
    end)
    
    return gui
end

-- ✅ แบบถูก: Track และ Clean connections
local function goodEventLoop()
    local gui = Instance.new("Frame")
    local connections = {}
    
    table.insert(connections, gui.MouseButton1Click:Connect(function()
        print("clicked")
    end))
    
    -- Clean up เมื่อ GUI ถูกลบ
    gui.Destroying:Connect(function()
        for _, conn in ipairs(connections) do
            conn:Disconnect()
        end
    end)
    
    return gui
end

-- ❌ แบบผิด 3: ลืม Destroy Instance
local function badParticles()
    -- สร้าง ParticleEmitter ทุกครั้งที่มี Event
    workspace.SomeEvent.OnServerEvent:Connect(function(player, position)
        local emitter = Instance.new("ParticleEmitter")
        emitter.Parent = workspace
        -- ลืม Destroy! สะสมเรื่อยๆ
    end)
end

-- ✅ แบบถูก: Destroy หลังใช้
local function goodParticles()
    workspace.SomeEvent.OnServerEvent:Connect(function(player, position)
        local part = Instance.new("Part")
        part.Position = position
        part.Anchored = true
        part.Parent = workspace
        
        local emitter = Instance.new("ParticleEmitter")
        emitter.Parent = part
        
        -- Destroy หลัง effect หมด
        task.delay(3, function()
            part:Destroy()  -- Destroy Parent จะลบ Children ด้วย
        end)
    end)
end
```

---

## 10. Memory Profiling Workflow

```lua
-- MemoryProfilingWorkflow.lua
-- ขั้นตอนการ Profile Memory

--[[
ขั้นตอนที่ 1: Baseline
- วัด Memory ตอนเริ่มเกม (idle)
- ไม่มี Players, ไม่มี Actions

ขั้นตอนที่ 2: Stress Test
- เพิ่ม Players จำลอง
- ทำ Actions ต่างๆ ซ้ำหลายครั้ง
- ดู Memory เพิ่มขึ้นเท่าไหร่

ขั้นตอนที่ 3: Cleanup Test
- ลบ Players ออก
- Memory ควรกลับใกล้ Baseline
- ถ้าไม่กลับ = Memory Leak

ขั้นตอนที่ 4: Long-run Test
- ให้เกมทำงาน 30-60 นาที
- ดูว่า Memory เพิ่มขึ้นเรื่อยๆ หรือเปล่า
]]

local function performMemoryTest()
    local Stats = game:GetService("Stats")
    
    -- Step 1: Baseline
    local baseline = Stats:GetTotalMemoryUsageMb()
    print(string.format("Baseline Memory: %.1f MB", baseline))
    
    -- Step 2: ทำ Actions
    local testObjects = {}
    for i = 1, 100 do
        local part = Instance.new("Part")
        part.Parent = workspace
        table.insert(testObjects, part)
    end
    
    task.wait(1)
    local afterCreate = Stats:GetTotalMemoryUsageMb()
    print(string.format("After Create: %.1f MB (+%.1f MB)", 
        afterCreate, afterCreate - baseline))
    
    -- Step 3: Cleanup
    for _, part in ipairs(testObjects) do
        part:Destroy()
    end
    testObjects = {}
    
    -- รอ GC
    task.wait(2)
    collectgarbage("collect")
    task.wait(1)
    
    local afterCleanup = Stats:GetTotalMemoryUsageMb()
    print(string.format("After Cleanup: %.1f MB (+%.1f MB from baseline)", 
        afterCleanup, afterCleanup - baseline))
    
    if afterCleanup - baseline > 5 then
        warn("อาจมี Memory Leak! Memory ไม่กลับสู่ Baseline")
    else
        print("Memory Management ปกติ ✅")
    end
end
```

---

## 11. แบบฝึกหัด

### แบบฝึกหัดที่ 1: Memory Leak Hunt
สร้าง Script ที่จงใจมี Memory Leak แล้ว:
- ตรวจหา Leak ด้วย MemoryLeakDetector
- แก้ไข Leak
- วัด Memory ก่อนและหลังแก้ไข

### แบบฝึกหัดที่ 2: GUI Performance Test
สร้าง Inventory GUI ที่รองรับ 1000 Items:
- ใช้ GUIPool สำหรับ Item Slots
- วัด Memory ตอน Open/Close
- ตรวจสอบว่า Memory กลับคืนหลัง Close

### แบบฝึกหัดที่ 3: LRU Cache Implementation
สร้าง LRU Cache สำหรับ Player Statistics:
- Cache ข้อมูล 50 Players ล่าสุด
- ทดสอบ Cache Hit Rate
- วัด Performance เทียบกับ ไม่ใช้ Cache

---

## สรุป

Memory Management ที่ดีต้องการความใส่ใจในทุกขั้นตอน:

1. **Track Memory**: ใช้ Stats Service ตรวจสอบสม่ำเสมอ
2. **Detect Leaks**: ตรวจหา Memory ที่เพิ่มขึ้นต่อเนื่อง
3. **Manage Assets**: Cache อย่างชาญฉลาด ใช้ Weak References
4. **Clean Connections**: Disconnect เสมอเมื่อไม่ใช้
5. **Dispose Pattern**: จัดการ Lifecycle อย่างเป็นระบบ
6. **Pool Objects**: Reuse แทนสร้างและลบซ้ำๆ

ในส่วนถัดไป (Part 74) เราจะสร้าง Loading Screen ที่สวยงามและ functional พร้อมระบบ Progress ที่แสดงสถานะการโหลด Assets อย่างถูกต้อง
