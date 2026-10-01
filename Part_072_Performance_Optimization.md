# Part 72: Performance Optimization (การปรับแต่งประสิทธิภาพเกม)

## บทนำ

ประสิทธิภาพเกม (Performance) เป็นหัวใจสำคัญของการพัฒนาเกม Roblox ที่ดี เกมที่ทำงานช้าหรือกระตุก (lag) จะทำให้ผู้เล่นเลิกเล่น ในบทนี้เราจะเรียนรู้เทคนิคการปรับแต่งประสิทธิภาพแบบครอบคลุม ตั้งแต่การลด Part Count ไปจนถึงระบบ LOD และการใช้ Profiler

---

## 1. เข้าใจ Performance Metrics

### 1.1 ตัวชี้วัดที่สำคัญ

```lua
-- PerformanceMonitor.lua (Server Script)
-- ระบบตรวจสอบประสิทธิภาพแบบ Real-time

local RunService = game:GetService("RunService")
local Players = game:GetService("Players")
local Stats = game:GetService("Stats")

local PerformanceMonitor = {}
PerformanceMonitor.__index = PerformanceMonitor

-- ค่าเกณฑ์ที่ยอมรับได้ (Acceptable thresholds)
local THRESHOLDS = {
    fps = 30,           -- FPS ต่ำสุดที่ยอมรับได้
    ping = 200,         -- Ping สูงสุด (ms)
    partCount = 10000,  -- จำนวน Part สูงสุด
    scriptCount = 500,  -- จำนวน Script สูงสุด
    memory = 1500       -- Memory สูงสุด (MB)
}

function PerformanceMonitor.new()
    local self = setmetatable({}, PerformanceMonitor)
    self.metrics = {
        serverFPS = 60,
        heartbeatTime = 0,
        partCount = 0,
        scriptCount = 0,
        memory = 0,
        networkSend = 0,
        networkReceive = 0
    }
    self.history = {}
    self.alerts = {}
    return self
end

-- อัพเดท metrics ทุก 5 วินาที
function PerformanceMonitor:startMonitoring()
    RunService.Heartbeat:Connect(function(dt)
        -- วัด server FPS จาก delta time
        self.metrics.serverFPS = math.floor(1 / dt)
        self.metrics.heartbeatTime = dt * 1000 -- แปลงเป็น milliseconds
    end)
    
    -- เก็บ metrics ทุก 5 วินาที
    task.spawn(function()
        while true do
            self:collectMetrics()
            self:checkAlerts()
            task.wait(5)
        end
    end)
end

function PerformanceMonitor:collectMetrics()
    -- นับ Parts ในเกม
    local partCount = 0
    local scriptCount = 0
    
    -- วิธีที่เร็วกว่าการใช้ GetDescendants บน Workspace
    for _, instance in ipairs(workspace:GetDescendants()) do
        if instance:IsA("BasePart") then
            partCount = partCount + 1
        elseif instance:IsA("Script") or instance:IsA("LocalScript") or instance:IsA("ModuleScript") then
            scriptCount = scriptCount + 1
        end
    end
    
    self.metrics.partCount = partCount
    self.metrics.scriptCount = scriptCount
    
    -- Memory usage
    self.metrics.memory = Stats:GetTotalMemoryUsageMb()
    
    -- Network stats
    self.metrics.networkSend = Stats.DataSendKbps
    self.metrics.networkReceive = Stats.DataReceiveKbps
    
    -- บันทึกประวัติ (เก็บแค่ 60 records)
    table.insert(self.history, {
        timestamp = tick(),
        fps = self.metrics.serverFPS,
        memory = self.metrics.memory,
        partCount = partCount
    })
    
    if #self.history > 60 then
        table.remove(self.history, 1)
    end
    
    print(string.format(
        "[PERF] FPS: %d | Memory: %.1fMB | Parts: %d | Scripts: %d",
        self.metrics.serverFPS,
        self.metrics.memory,
        partCount,
        scriptCount
    ))
end

-- แจ้งเตือนเมื่อเกินค่าเกณฑ์
function PerformanceMonitor:checkAlerts()
    local alerts = {}
    
    if self.metrics.serverFPS < THRESHOLDS.fps then
        table.insert(alerts, string.format("⚠️ FPS ต่ำ: %d (ควร > %d)", 
            self.metrics.serverFPS, THRESHOLDS.fps))
    end
    
    if self.metrics.partCount > THRESHOLDS.partCount then
        table.insert(alerts, string.format("⚠️ Parts มากเกินไป: %d (ควร < %d)", 
            self.metrics.partCount, THRESHOLDS.partCount))
    end
    
    if self.metrics.memory > THRESHOLDS.memory then
        table.insert(alerts, string.format("⚠️ Memory สูง: %.1fMB (ควร < %dMB)", 
            self.metrics.memory, THRESHOLDS.memory))
    end
    
    for _, alert in ipairs(alerts) do
        warn(alert)
    end
    
    self.alerts = alerts
end

-- ส่งข้อมูลไปยัง Client (สำหรับ Debug HUD)
function PerformanceMonitor:getReport()
    return {
        metrics = self.metrics,
        alerts = self.alerts,
        status = #self.alerts == 0 and "GOOD" or "WARNING"
    }
end

return PerformanceMonitor
```

---

## 2. Part Count Optimization

### 2.1 ทำไม Part Count ถึงสำคัญ

Part Count มีผลโดยตรงต่อ:
- **Rendering Time**: GPU ต้องวาด Part ทุกชิ้น
- **Physics Simulation**: Engine ต้องคำนวณ Collision
- **Memory Usage**: แต่ละ Part ใช้ RAM
- **Network Traffic**: ข้อมูล Part ต้องส่งให้ Client

### 2.2 เทคนิคลด Part Count

```lua
-- PartOptimizer.lua (Module Script)
-- เครื่องมือวิเคราะห์และปรับแต่ง Parts

local PartOptimizer = {}

-- ค้นหา Parts ที่ไม่จำเป็น
function PartOptimizer.findRedundantParts()
    local redundant = {}
    
    for _, part in ipairs(workspace:GetDescendants()) do
        if part:IsA("BasePart") then
            local reasons = {}
            
            -- Part เล็กมาก (เล็กกว่า 0.1 studs)
            if part.Size.X < 0.1 or part.Size.Y < 0.1 or part.Size.Z < 0.1 then
                table.insert(reasons, "ขนาดเล็กเกินไป (<0.1 studs)")
            end
            
            -- Part อยู่ใต้พื้น (Buried)
            if part.Position.Y < -50 then
                table.insert(reasons, "อยู่ใต้ map")
            end
            
            -- Part ที่ไม่มี Children และไม่มี Scripts
            local hasChildren = #part:GetChildren() > 0
            if not hasChildren and part.Name == "Part" then
                table.insert(reasons, "Part ไม่มีชื่อและ Children")
            end
            
            -- Part ที่ซ้อนทับกันมาก
            -- (ตรวจสอบด้วย spatial query)
            
            if #reasons > 0 then
                table.insert(redundant, {
                    part = part,
                    path = part:GetFullName(),
                    reasons = reasons
                })
            end
        end
    end
    
    print(string.format("พบ Parts ที่น่าสงสัย: %d ชิ้น", #redundant))
    for i, data in ipairs(redundant) do
        print(string.format("  %d. %s: %s", i, data.path, table.concat(data.reasons, ", ")))
    end
    
    return redundant
end

-- รวม Parts ที่อยู่ใกล้กันเป็น Union
function PartOptimizer.suggestUnions()
    -- วิเคราะห์ Parts ที่อยู่ใน Model เดียวกัน
    local models = {}
    
    for _, model in ipairs(workspace:GetDescendants()) do
        if model:IsA("Model") then
            local parts = {}
            for _, child in ipairs(model:GetDescendants()) do
                if child:IsA("BasePart") and not child:IsA("MeshPart") then
                    table.insert(parts, child)
                end
            end
            
            if #parts > 5 then
                table.insert(models, {
                    model = model,
                    partCount = #parts,
                    name = model.Name
                })
            end
        end
    end
    
    -- เรียงตาม part count
    table.sort(models, function(a, b) return a.partCount > b.partCount end)
    
    print("\nModels ที่ควร Union (มี Parts มาก):")
    for i = 1, math.min(10, #models) do
        local data = models[i]
        print(string.format("  %d. %s: %d parts", i, data.name, data.partCount))
    end
    
    return models
end

-- เปลี่ยน CanCollide ของ Decorative Parts
function PartOptimizer.optimizeCollisions(folder)
    -- Parts ที่ไม่ต้องการ Collision (decorative)
    local decorativeNames = {
        "Leaf", "Flower", "Grass", "Decoration", "Detail",
        "Trim", "Border", "Ornament", "Sign"
    }
    
    local optimized = 0
    
    for _, part in ipairs(folder:GetDescendants()) do
        if part:IsA("BasePart") then
            for _, name in ipairs(decorativeNames) do
                if part.Name:find(name) then
                    part.CanCollide = false
                    part.CastShadow = false -- ลด Shadow Rendering ด้วย
                    optimized = optimized + 1
                    break
                end
            end
        end
    end
    
    print(string.format("ปิด Collision สำหรับ %d Decorative Parts", optimized))
    return optimized
end

-- ใช้ Anchored และปิด Physics สำหรับ Static Objects
function PartOptimizer.anchorStaticObjects(folder)
    local anchored = 0
    
    for _, part in ipairs(folder:GetDescendants()) do
        if part:IsA("BasePart") and not part.Anchored then
            -- ถ้าไม่มี Weld หรือ Motor - น่าจะเป็น static
            local hasJoint = false
            for _, child in ipairs(part:GetChildren()) do
                if child:IsA("JointInstance") or child:IsA("Motor6D") then
                    hasJoint = true
                    break
                end
            end
            
            if not hasJoint then
                part.Anchored = true
                anchored = anchored + 1
            end
        end
    end
    
    print(string.format("Anchor %d Static Parts", anchored))
    return anchored
end

return PartOptimizer
```

---

## 3. ระบบ LOD (Level of Detail)

LOD คือการแสดงผล Objects ที่อยู่ไกลด้วยคุณภาพต่ำกว่า เพื่อประหยัด GPU

```lua
-- LODSystem.lua (Local Script ใน StarterPlayerScripts)
-- ระบบ Level of Detail อัตโนมัติ

local RunService = game:GetService("RunService")
local Players = game:GetService("Players")

local LODSystem = {}
LODSystem.__index = LODSystem

-- ระดับ LOD และระยะทาง
local LOD_LEVELS = {
    {distance = 50,  quality = "HIGH"},    -- ใกล้ - คุณภาพสูง
    {distance = 100, quality = "MEDIUM"},  -- กลาง - คุณภาพปานกลาง
    {distance = 200, quality = "LOW"},     -- ไกล - คุณภาพต่ำ
    {distance = math.huge, quality = "NONE"} -- ไกลมาก - ซ่อน
}

-- การตั้งค่าสำหรับแต่ละ quality level
local QUALITY_SETTINGS = {
    HIGH = {
        castShadow = true,
        transparency = 0,
        detailLevel = 1.0
    },
    MEDIUM = {
        castShadow = false,  -- ปิด Shadow เพื่อประหยัด GPU
        transparency = 0,
        detailLevel = 0.5
    },
    LOW = {
        castShadow = false,
        transparency = 0,
        detailLevel = 0.25
    },
    NONE = {
        castShadow = false,
        transparency = 1,    -- ซ่อน Part โดยทำ transparent
        detailLevel = 0
    }
}

function LODSystem.new()
    local self = setmetatable({}, LODSystem)
    self.player = Players.LocalPlayer
    self.lodObjects = {}        -- Objects ที่ต้องการ LOD
    self.updateInterval = 0.5  -- อัพเดทบ่อยแค่ไหน (วินาที)
    self.lastUpdate = 0
    return self
end

-- ลงทะเบียน Object สำหรับ LOD
function LODSystem:registerObject(model, lodConfig)
    lodConfig = lodConfig or {}
    
    -- เก็บ Original settings
    local parts = {}
    for _, part in ipairs(model:GetDescendants()) do
        if part:IsA("BasePart") then
            parts[part] = {
                originalCastShadow = part.CastShadow,
                originalTransparency = part.Transparency
            }
        end
    end
    
    self.lodObjects[model] = {
        model = model,
        parts = parts,
        currentLOD = "HIGH",
        priority = lodConfig.priority or 1,
        maxDistance = lodConfig.maxDistance or 200
    }
end

-- อัพเดท LOD สำหรับทุก Objects
function LODSystem:update()
    local camera = workspace.CurrentCamera
    if not camera then return end
    
    local cameraPos = camera.CFrame.Position
    
    for model, data in pairs(self.lodObjects) do
        -- ตรวจสอบว่า Model ยังอยู่
        if not model.Parent then
            self.lodObjects[model] = nil
            continue
        end
        
        -- หาจุดกึ่งกลางของ Model
        local center = model:GetBoundingBox().Position
        local distance = (center - cameraPos).Magnitude
        
        -- หา LOD Level ที่เหมาะสม
        local newLOD = "NONE"
        for _, level in ipairs(LOD_LEVELS) do
            if distance <= level.distance then
                newLOD = level.quality
                break
            end
        end
        
        -- อัพเดทเฉพาะเมื่อ LOD เปลี่ยน
        if newLOD ~= data.currentLOD then
            self:applyLOD(data, newLOD)
            data.currentLOD = newLOD
        end
    end
end

-- ใช้ LOD Settings กับ Object
function LODSystem:applyLOD(data, quality)
    local settings = QUALITY_SETTINGS[quality]
    if not settings then return end
    
    for part, original in pairs(data.parts) do
        if part.Parent then -- ตรวจสอบว่า Part ยังอยู่
            -- ใช้ smooth transition สำหรับ transparency
            if quality == "NONE" then
                part.Transparency = 1
                part.CanQuery = false  -- ปิด Raycasting ด้วย
                part.CanCollide = false
            else
                part.Transparency = original.originalTransparency
                part.CastShadow = settings.castShadow
                part.CanQuery = true
                -- Collision: คืนค่าตาม original settings
            end
        end
    end
end

-- เริ่มระบบ LOD
function LODSystem:start()
    RunService.RenderStepped:Connect(function()
        local now = tick()
        if now - self.lastUpdate >= self.updateInterval then
            self.lastUpdate = now
            self:update()
        end
    end)
    
    print("LOD System เริ่มทำงาน")
end

-- ตัวอย่างการใช้งาน
local lodSystem = LODSystem.new()

-- ลงทะเบียน Models ที่ต้องการ LOD
workspace.DescendantAdded:Connect(function(instance)
    if instance:IsA("Model") and instance:HasTag("LOD") then
        lodSystem:registerObject(instance)
    end
end)

-- ลงทะเบียน Models ที่มีอยู่แล้ว
for _, model in ipairs(workspace:GetDescendants()) do
    if model:IsA("Model") and model:HasTag("LOD") then
        lodSystem:registerObject(model)
    end
end

lodSystem:start()
```

---

## 4. Instance Reuse และ Object Pooling

การสร้างและทำลาย Instances บ่อยๆ ทำให้ Garbage Collector ทำงานหนัก

```lua
-- ObjectPool.lua (Module Script)
-- ระบบ Pool สำหรับ Reuse Objects

local ObjectPool = {}
ObjectPool.__index = ObjectPool

function ObjectPool.new(template, initialSize)
    local self = setmetatable({}, ObjectPool)
    self.template = template
    self.available = {}    -- Objects ที่พร้อมใช้
    self.inUse = {}        -- Objects ที่กำลังใช้อยู่
    self.totalCreated = 0
    
    -- สร้าง Objects ล่วงหน้า (Pre-warm)
    self:prewarm(initialSize or 10)
    
    return self
end

-- สร้าง Objects ล่วงหน้า
function ObjectPool:prewarm(count)
    for i = 1, count do
        local obj = self:createNew()
        obj.Parent = nil  -- ซ่อนก่อน
        table.insert(self.available, obj)
    end
    print(string.format("Pool Pre-warmed: %d objects", count))
end

-- สร้าง Object ใหม่จาก Template
function ObjectPool:createNew()
    local obj = self.template:Clone()
    self.totalCreated = self.totalCreated + 1
    return obj
end

-- ขอ Object จาก Pool
function ObjectPool:get()
    local obj
    
    if #self.available > 0 then
        -- ดึงจาก Pool ที่มีอยู่
        obj = table.remove(self.available)
    else
        -- สร้างใหม่ถ้า Pool หมด
        obj = self:createNew()
        warn(string.format("Pool หมด! สร้าง Object ใหม่ (total: %d)", self.totalCreated))
    end
    
    self.inUse[obj] = true
    return obj
end

-- คืน Object กลับ Pool
function ObjectPool:release(obj)
    if not self.inUse[obj] then
        warn("พยายามคืน Object ที่ไม่ได้มาจาก Pool!")
        return
    end
    
    -- Reset Object กลับสู่สถานะเริ่มต้น
    self:resetObject(obj)
    
    self.inUse[obj] = nil
    obj.Parent = nil  -- ซ่อน
    table.insert(self.available, obj)
end

-- Reset Object (Override ตามความต้องการ)
function ObjectPool:resetObject(obj)
    -- ตัวอย่าง: Reset Position และ Velocity
    if obj:IsA("BasePart") then
        obj.Velocity = Vector3.zero
        obj.AssemblyLinearVelocity = Vector3.zero
        obj.AssemblyAngularVelocity = Vector3.zero
    end
    
    -- ลบ Connections ชั่วคราว
    if obj:FindFirstChild("_tempConnections") then
        for _, conn in ipairs(obj._tempConnections) do
            conn:Disconnect()
        end
    end
end

-- ดูสถานะ Pool
function ObjectPool:getStats()
    return {
        available = #self.available,
        inUse = 0,  -- นับจาก dictionary
        total = self.totalCreated
    }
end

return ObjectPool

--[[
ตัวอย่างการใช้งาน Bullet Pool:

local BulletPool = require(script.ObjectPool)

local bulletTemplate = workspace.BulletTemplate  -- Template Part
local bulletPool = BulletPool.new(bulletTemplate, 50)

-- เมื่อยิงกระสุน
local bullet = bulletPool:get()
bullet.CFrame = gun.CFrame
bullet.Parent = workspace.Bullets

-- เมื่อกระสุนหมดอายุ
task.delay(3, function()
    bulletPool:release(bullet)
end)
]]
```

---

## 5. Connection Management

Memory Leak ที่พบบ่อยที่สุดใน Roblox มาจาก Connections ที่ไม่ได้ Disconnect

```lua
-- ConnectionManager.lua (Module Script)
-- ระบบจัดการ Connections เพื่อป้องกัน Memory Leak

local ConnectionManager = {}
ConnectionManager.__index = ConnectionManager

function ConnectionManager.new()
    local self = setmetatable({}, ConnectionManager)
    self.connections = {}
    self.groups = {}  -- จัดกลุ่ม Connections
    return self
end

-- เพิ่ม Connection พร้อม Key
function ConnectionManager:add(key, connection)
    -- Disconnect connection เก่าถ้ามี
    if self.connections[key] then
        self.connections[key]:Disconnect()
    end
    self.connections[key] = connection
    return connection
end

-- เพิ่ม Connection ในกลุ่ม
function ConnectionManager:addToGroup(groupName, connection)
    if not self.groups[groupName] then
        self.groups[groupName] = {}
    end
    table.insert(self.groups[groupName], connection)
    return connection
end

-- Disconnect Connection เดียว
function ConnectionManager:remove(key)
    if self.connections[key] then
        self.connections[key]:Disconnect()
        self.connections[key] = nil
    end
end

-- Disconnect ทั้งกลุ่ม
function ConnectionManager:removeGroup(groupName)
    if self.groups[groupName] then
        for _, conn in ipairs(self.groups[groupName]) do
            if conn.Connected then
                conn:Disconnect()
            end
        end
        self.groups[groupName] = nil
    end
end

-- Disconnect ทั้งหมด
function ConnectionManager:disconnectAll()
    for key, conn in pairs(self.connections) do
        if conn.Connected then
            conn:Disconnect()
        end
    end
    self.connections = {}
    
    for groupName, conns in pairs(self.groups) do
        for _, conn in ipairs(conns) do
            if conn.Connected then
                conn:Disconnect()
            end
        end
    end
    self.groups = {}
    
    print("Disconnected all connections")
end

-- นับจำนวน Active Connections
function ConnectionManager:count()
    local total = 0
    for _ in pairs(self.connections) do total = total + 1 end
    for _, group in pairs(self.groups) do
        total = total + #group
    end
    return total
end

return ConnectionManager

--[[
ตัวอย่างการใช้งานใน NPC Script:

local CM = require(script.Parent.ConnectionManager)
local connections = CM.new()

-- เพิ่ม Connections
connections:add("heartbeat", RunService.Heartbeat:Connect(function(dt)
    -- NPC AI logic
end))

connections:add("playerAdded", Players.PlayerAdded:Connect(function(player)
    -- React to new player
end))

-- เมื่อ NPC ถูกลบ
npc.Destroying:Connect(function()
    connections:disconnectAll()
end)
]]
```

---

## 6. Script Optimization

### 6.1 หลีกเลี่ยงการทำงานซ้ำซ้อน

```lua
-- ScriptOptimizer.lua - ตัวอย่างการเขียน Script ที่มีประสิทธิภาพ

-- ❌ แบบผิด: ค้นหา Service ซ้ำทุกครั้ง
local function badExample()
    while true do
        local players = game:GetService("Players"):GetPlayers()  -- ค้นหาซ้ำทุก loop!
        for _, player in ipairs(players) do
            local character = player.Character  -- ดีแล้ว
            -- ...
        end
        task.wait(1)
    end
end

-- ✅ แบบถูก: Cache Services ไว้นอก Loop
local Players = game:GetService("Players")  -- Cache ครั้งเดียว

local function goodExample()
    while true do
        local players = Players:GetPlayers()  -- ใช้ cached service
        for _, player in ipairs(players) do
            local character = player.Character
            -- ...
        end
        task.wait(1)
    end
end

-- ❌ แบบผิด: ใช้ FindFirstChild ใน Tight Loop
local function badFindExample(character)
    RunService.Heartbeat:Connect(function()
        local humanoid = character:FindFirstChild("Humanoid")  -- ค้นหาทุก frame!
        if humanoid then
            -- ...
        end
    end)
end

-- ✅ แบบถูก: Cache Reference
local function goodFindExample(character)
    local humanoid = character:WaitForChild("Humanoid")  -- Cache ครั้งเดียว
    
    RunService.Heartbeat:Connect(function()
        if humanoid and humanoid.Parent then  -- ใช้ cached reference
            -- ...
        end
    end)
end

-- ❌ แบบผิด: String Concatenation ใน Loop
local function badStringExample()
    local result = ""
    for i = 1, 1000 do
        result = result .. tostring(i) .. ", "  -- สร้าง String ใหม่ทุกครั้ง!
    end
    return result
end

-- ✅ แบบถูก: ใช้ table.concat
local function goodStringExample()
    local parts = {}
    for i = 1, 1000 do
        parts[i] = tostring(i)
    end
    return table.concat(parts, ", ")  -- รวมครั้งเดียว
end

-- ❌ แบบผิด: Global Variables ใน Module
-- (Global ช้ากว่า Local ประมาณ 10%)
gCounter = 0  -- Global

-- ✅ แบบถูก: Local Variables
local counter = 0  -- Local (เร็วกว่า)

-- ❌ แบบผิด: Nested Function ใน Loop
local function badLoopExample()
    for i = 1, 100 do
        local function check()  -- สร้าง function ใหม่ทุก iteration!
            return i * 2
        end
        print(check())
    end
end

-- ✅ แบบถูก: Function นอก Loop
local function doubleValue(n)
    return n * 2
end

local function goodLoopExample()
    for i = 1, 100 do
        print(doubleValue(i))  -- ใช้ function เดิม
    end
end
```

### 6.2 ใช้ RunService อย่างถูกวิธี

```lua
-- RunServiceOptimizer.lua
-- วิธีใช้ RunService ที่ถูกต้อง

local RunService = game:GetService("RunService")

-- ❌ แบบผิด: ทำงานหนักทุก Frame
local heavyConnection = RunService.Heartbeat:Connect(function(dt)
    -- คำนวณ path finding ทุก frame - หนักมาก!
    local path = PathfindingService:CreatePath()
    path:ComputeAsync(start, finish)
end)

-- ✅ แบบถูก: แบ่งงานหนักออก และทำตาม Interval
local UPDATE_INTERVAL = 0.5  -- อัพเดททุก 0.5 วินาที
local lastUpdate = 0

local optimizedConnection = RunService.Heartbeat:Connect(function(dt)
    local now = tick()
    
    -- งานเบา: ทำทุก frame
    updateCharacterPosition(dt)
    
    -- งานหนัก: ทำตาม interval
    if now - lastUpdate >= UPDATE_INTERVAL then
        lastUpdate = now
        updatePathfinding()
    end
end)

-- ❌ แบบผิด: ใช้ RenderStepped สำหรับ Server Logic
-- RenderStepped ทำงานก่อน Render - ใช้สำหรับ Camera/Visual เท่านั้น!
local badServerLogic = RunService.RenderStepped:Connect(function()
    -- Server logic ใน RenderStepped - ผิดที่!
    Players:GetPlayers()  -- ไม่มีผลบน Server
end)

-- ✅ แบบถูก: เลือก Event ที่ถูกต้อง
-- Heartbeat: Server/Client Logic, Physics
-- RenderStepped: Camera Control, Visual Updates (Client เท่านั้น)
-- Stepped: Physics Calculations (ก่อน Physics Step)

-- Server:
local serverLogic = RunService.Heartbeat:Connect(function(dt)
    updateGameLogic(dt)
end)

-- Client Camera:
local cameraUpdate = RunService.RenderStepped:Connect(function()
    updateCameraFollow()
end)
```

---

## 7. Streaming Enabled

StreamingEnabled ทำให้ Server ส่ง Instances ไปยัง Client เฉพาะที่จำเป็น

```lua
-- StreamingConfig.lua (Server Script)
-- การตั้งค่า Streaming Enabled

local workspace = game:GetService("Workspace")

-- เปิด Streaming (ทำใน Properties ของ Workspace หรือผ่าน Script)
workspace.StreamingEnabled = true

-- ตั้งค่า Streaming Radius
workspace.StreamingMinRadius = 64    -- รัศมีขั้นต่ำที่ต้องโหลด (studs)
workspace.StreamingTargetRadius = 512 -- รัศมีเป้าหมาย

-- จัดการ Streaming ใน Client
-- LocalScript ใน StarterPlayerScripts:
--[[
local Players = game:GetService("Players")
local player = Players.LocalPlayer

-- รอให้ Character Stream เข้ามา
player.CharacterAdded:Connect(function(character)
    -- ใช้ WaitForChild แทน FindFirstChild
    local humanoid = character:WaitForChild("Humanoid")
    local hrp = character:WaitForChild("HumanoidRootPart")
    
    -- ตั้งค่า StreamingPriority สำหรับ Important Objects
    -- (ทำใน Workspace โดยตรง หรือผ่าน CollectionService)
end)
]]

-- ตั้งค่า StreamingPriority สำหรับ Important Parts
local function setHighPriority(instance)
    -- Parts ที่สำคัญ (Spawn, Checkpoints) ควรโหลดก่อน
    instance:SetAttribute("StreamingPriority", 1000)
end

-- Important Objects ควร SetAnchor และมี Priority สูง
local importantObjects = {
    workspace:FindFirstChild("Spawn"),
    workspace:FindFirstChild("Checkpoints")
}

for _, obj in ipairs(importantObjects) do
    if obj then
        setHighPriority(obj)
    end
end
```

---

## 8. Profiling Tools

### 8.1 ใช้ MicroProfiler

```lua
-- Profiler.lua (Module Script)
-- เครื่องมือ Profiling ด้วย debug.profilebegin/end

local Profiler = {}

-- เริ่ม Profiling Block
function Profiler.begin(label)
    debug.profilebegin(label)
end

-- จบ Profiling Block
function Profiler.finish()
    debug.profileend()
end

-- Wrap function ด้วย Profiling
function Profiler.wrap(label, func)
    return function(...)
        debug.profilebegin(label)
        local results = table.pack(func(...))
        debug.profileend()
        return table.unpack(results, 1, results.n)
    end
end

-- ตัวอย่างการใช้งาน:
--[[
local function heavyCalculation()
    debug.profilebegin("HeavyCalculation")
    -- คำนวณหนัก
    local sum = 0
    for i = 1, 10000 do
        sum = sum + math.sqrt(i)
    end
    debug.profileend()
    return sum
end

-- หรือใช้ wrap:
local profiledPathfind = Profiler.wrap("Pathfinding", function(start, finish)
    local path = PathfindingService:CreatePath()
    path:ComputeAsync(start, finish)
    return path
end)
]]

-- Benchmark function
function Profiler.benchmark(label, func, iterations)
    iterations = iterations or 100
    
    local startTime = tick()
    for i = 1, iterations do
        func()
    end
    local elapsed = tick() - startTime
    
    local avgMs = (elapsed / iterations) * 1000
    print(string.format("[BENCHMARK] %s: %.3fms avg (%d iterations)", 
        label, avgMs, iterations))
    
    return avgMs
end

return Profiler
```

### 8.2 Custom Performance Dashboard

```lua
-- PerformanceDashboard.lua (Local Script)
-- HUD แสดง Performance Stats

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Stats = game:GetService("Stats")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- สร้าง HUD
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "PerformanceDashboard"
screenGui.ResetOnSpawn = false
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.Parent = playerGui

-- Background Frame
local frame = Instance.new("Frame")
frame.Name = "Dashboard"
frame.Size = UDim2.new(0, 200, 0, 150)
frame.Position = UDim2.new(0, 10, 0, 10)
frame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
frame.BackgroundTransparency = 0.5
frame.BorderSizePixel = 0
frame.Parent = screenGui

-- Corner
local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 8)
corner.Parent = frame

-- Stats Labels
local function createLabel(name, y)
    local label = Instance.new("TextLabel")
    label.Name = name
    label.Size = UDim2.new(1, -10, 0, 20)
    label.Position = UDim2.new(0, 5, 0, y)
    label.BackgroundTransparency = 1
    label.TextColor3 = Color3.fromRGB(255, 255, 255)
    label.TextSize = 12
    label.Font = Enum.Font.Code
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = frame
    return label
end

local fpsLabel = createLabel("FPS", 5)
local pingLabel = createLabel("Ping", 25)
local memoryLabel = createLabel("Memory", 45)
local partsLabel = createLabel("Parts", 65)
local renderLabel = createLabel("Render", 85)
local networkLabel = createLabel("Network", 105)

-- เก็บ FPS History
local fpsHistory = {}
local frameCount = 0
local lastFpsUpdate = 0

-- อัพเดท Stats
local updateInterval = 0.5
local lastUpdate = 0

RunService.RenderStepped:Connect(function(dt)
    frameCount = frameCount + 1
    local now = tick()
    
    if now - lastUpdate < updateInterval then return end
    lastUpdate = now
    
    -- คำนวณ FPS จาก frame count
    local fps = math.floor(frameCount / updateInterval)
    frameCount = 0
    
    -- ตัดสีตาม FPS
    local fpsColor
    if fps >= 55 then
        fpsColor = Color3.fromRGB(0, 255, 0)     -- เขียว: ดี
    elseif fps >= 30 then
        fpsColor = Color3.fromRGB(255, 255, 0)   -- เหลือง: พอใช้
    else
        fpsColor = Color3.fromRGB(255, 0, 0)     -- แดง: แย่
    end
    
    fpsLabel.Text = string.format("FPS: %d", fps)
    fpsLabel.TextColor3 = fpsColor
    
    -- Ping
    local ping = player:GetNetworkPing() * 1000
    pingLabel.Text = string.format("Ping: %dms", math.floor(ping))
    pingLabel.TextColor3 = ping < 100 and Color3.fromRGB(0, 255, 0) 
        or ping < 200 and Color3.fromRGB(255, 255, 0)
        or Color3.fromRGB(255, 0, 0)
    
    -- Memory
    local memory = Stats:GetTotalMemoryUsageMb()
    memoryLabel.Text = string.format("Memory: %.1fMB", memory)
    memoryLabel.TextColor3 = memory < 500 and Color3.fromRGB(0, 255, 0)
        or memory < 1000 and Color3.fromRGB(255, 255, 0)
        or Color3.fromRGB(255, 0, 0)
    
    -- Parts (approximate from workspace)
    -- ใช้ workspace.PersistentLoaded หรือนับจาก Descendants (หนัก)
    partsLabel.Text = "Parts: (check Studio)"
    
    -- Render Distance
    renderLabel.Text = string.format("RenderDist: %d", 
        workspace.CurrentCamera.MaxAxisExtents or 0)
    
    -- Network
    networkLabel.Text = string.format("Net: ↑%.1f ↓%.1f KB/s",
        Stats.DataSendKbps, Stats.DataReceiveKbps)
end)

-- Toggle Dashboard ด้วย F3
local UserInputService = game:GetService("UserInputService")
UserInputService.InputBegan:Connect(function(input, processed)
    if not processed and input.KeyCode == Enum.KeyCode.F3 then
        frame.Visible = not frame.Visible
    end
end)
```

---

## 9. การลด Draw Calls

```lua
-- DrawCallOptimizer.lua (Module Script)
-- เทคนิคลด Draw Calls

local DrawCallOptimizer = {}

--[[
Draw Call คือคำสั่งที่ CPU ส่งให้ GPU วาด Object
ยิ่งน้อย Draw Calls ยิ่งเร็ว

วิธีลด Draw Calls:
1. ใช้ Texture Atlas (รวม Textures)
2. ใช้ SurfaceAppearance แทน multiple Decals  
3. ใช้ Union Parts
4. ใช้ MeshPart แทนการ Stack Parts
5. ใช้ Same Material/Color เพื่อ Batching
]]

-- ตรวจสอบ Parts ที่มี Material เหมือนกัน (สำหรับแนะนำการรวม)
function DrawCallOptimizer.analyzeMaterials(folder)
    local materialGroups = {}
    
    for _, part in ipairs(folder:GetDescendants()) do
        if part:IsA("BasePart") then
            local key = tostring(part.Material) .. "_" .. tostring(part.Color)
            
            if not materialGroups[key] then
                materialGroups[key] = {
                    material = part.Material,
                    color = part.Color,
                    parts = {}
                }
            end
            
            table.insert(materialGroups[key].parts, part)
        end
    end
    
    -- หา Groups ที่ใหญ่ที่สุด
    local sorted = {}
    for key, group in pairs(materialGroups) do
        table.insert(sorted, group)
    end
    table.sort(sorted, function(a, b) return #a.parts > #b.parts end)
    
    print("\nMaterial Groups (สำหรับ Batching):")
    for i = 1, math.min(5, #sorted) do
        local g = sorted[i]
        print(string.format("  %s / RGB(%d,%d,%d): %d parts",
            tostring(g.material),
            g.color.R * 255, g.color.G * 255, g.color.B * 255,
            #g.parts))
    end
    
    return sorted
end

-- แนะนำการใช้ Decal แทน Part Stack
function DrawCallOptimizer.findOverlappingDecals(folder)
    local results = {}
    
    for _, part in ipairs(folder:GetDescendants()) do
        if part:IsA("BasePart") then
            local decals = {}
            for _, child in ipairs(part:GetChildren()) do
                if child:IsA("Decal") or child:IsA("Texture") then
                    table.insert(decals, child)
                end
            end
            
            -- Parts ที่มี Decal หลายอัน
            if #decals > 2 then
                table.insert(results, {
                    part = part,
                    decalCount = #decals,
                    suggestion = "พิจารณาใช้ SurfaceAppearance แทน"
                })
            end
        end
    end
    
    return results
end

return DrawCallOptimizer
```

---

## 10. Memory Optimization Basics

```lua
-- MemoryOptimizer.lua (Module Script)
-- เทคนิคพื้นฐานการจัดการ Memory

local MemoryOptimizer = {}

-- 1. Weak Tables - อนุญาตให้ GC เก็บ Objects ที่ไม่ได้ใช้
function MemoryOptimizer.createWeakCache()
    -- Weak Table: keys จะถูก GC ถ้าไม่มี reference อื่น
    local cache = setmetatable({}, {__mode = "v"})  -- weak values
    
    return {
        set = function(key, value)
            cache[key] = value
        end,
        get = function(key)
            return cache[key]
        end
    }
end

-- 2. ทำลาย Unused Instances
function MemoryOptimizer.destroyUnused(parent)
    local destroyed = 0
    
    for _, instance in ipairs(parent:GetDescendants()) do
        -- ลบ Parts ที่ซ่อนและไม่ได้ใช้
        if instance:IsA("BasePart") and 
           instance.Transparency >= 1 and
           not instance:HasTag("KeepHidden") then
            instance:Destroy()
            destroyed = destroyed + 1
        end
        
        -- ลบ Scripts ที่ disabled
        if (instance:IsA("Script") or instance:IsA("LocalScript")) and
           not instance.Enabled then
            -- ระวัง: บาง Script disabled โดยตั้งใจ
            -- instance:Destroy()
        end
    end
    
    print(string.format("ลบ %d unused instances", destroyed))
    return destroyed
end

-- 3. Image Preloading อย่างมีประสิทธิภาพ
function MemoryOptimizer.preloadAssets(assetList, callback)
    local ContentProvider = game:GetService("ContentProvider")
    
    -- Preload เป็น batch เพื่อไม่ให้ใช้ Memory พร้อมกัน
    local BATCH_SIZE = 10
    
    task.spawn(function()
        for i = 1, #assetList, BATCH_SIZE do
            local batch = {}
            for j = i, math.min(i + BATCH_SIZE - 1, #assetList) do
                table.insert(batch, assetList[j])
            end
            
            ContentProvider:PreloadAsync(batch)
            
            -- รายงาน progress
            local progress = math.min(i + BATCH_SIZE - 1, #assetList) / #assetList
            if callback then
                callback(progress)
            end
            
            task.wait()  -- yield เพื่อไม่ block
        end
        
        print("Preload เสร็จสิ้น!")
    end)
end

-- 4. Clear Data เมื่อ Player ออก
function MemoryOptimizer.setupPlayerCleanup(playerData)
    local Players = game:GetService("Players")
    
    Players.PlayerRemoving:Connect(function(player)
        -- ลบข้อมูลของ Player ออกจาก Memory
        if playerData[player.UserId] then
            playerData[player.UserId] = nil
            print(string.format("Cleaned up data for %s", player.Name))
        end
    end)
end

return MemoryOptimizer
```

---

## 11. ข้อผิดพลาดที่พบบ่อย

### ❌ ข้อผิดพลาดที่ 1: ไม่ Disconnect Event Connections

```lua
-- ❌ แบบผิด: Connections สะสมจนทำให้ Memory Leak
local function setupNPC(npc)
    RunService.Heartbeat:Connect(function()
        -- ไม่มี Disconnect - Connections สะสมเรื่อยๆ!
        if not npc.Parent then return end
        updateNPC(npc)
    end)
end

-- ✅ แบบถูก: Disconnect เมื่อไม่ใช้แล้ว
local function setupNPCFixed(npc)
    local connection = RunService.Heartbeat:Connect(function()
        if not npc.Parent then return end
        updateNPC(npc)
    end)
    
    -- Disconnect เมื่อ NPC ถูกลบ
    npc.Destroying:Connect(function()
        connection:Disconnect()
    end)
end
```

### ❌ ข้อผิดพลาดที่ 2: สร้าง Instances ใน Loop โดยไม่จำเป็น

```lua
-- ❌ แบบผิด: สร้าง Part ใหม่ทุกครั้งที่ยิง
local function badShoot(position)
    local bullet = Instance.new("Part")  -- สร้างทุกครั้ง!
    bullet.Size = Vector3.new(0.2, 0.2, 2)
    bullet.Position = position
    bullet.Parent = workspace
    
    task.delay(3, function()
        bullet:Destroy()  -- Destroy ทุกครั้ง
    end)
end

-- ✅ แบบถูก: ใช้ Object Pool
local bulletPool = ObjectPool.new(bulletTemplate, 50)

local function goodShoot(position)
    local bullet = bulletPool:get()
    bullet.Position = position
    bullet.Parent = workspace
    
    task.delay(3, function()
        bulletPool:release(bullet)  -- คืน Pool
    end)
end
```

### ❌ ข้อผิดพลาดที่ 3: GetDescendants() ใน Tight Loop

```lua
-- ❌ แบบผิด: เรียก GetDescendants ทุก frame
RunService.Heartbeat:Connect(function()
    local allParts = workspace:GetDescendants()  -- ช้ามาก!
    for _, part in ipairs(allParts) do
        if part:IsA("BasePart") then
            -- process
        end
    end
end)

-- ✅ แบบถูก: Cache และ Track แทน
local trackedParts = {}

workspace.DescendantAdded:Connect(function(instance)
    if instance:IsA("BasePart") then
        table.insert(trackedParts, instance)
    end
end)

workspace.DescendantRemoving:Connect(function(instance)
    for i, part in ipairs(trackedParts) do
        if part == instance then
            table.remove(trackedParts, i)
            break
        end
    end
end)

RunService.Heartbeat:Connect(function()
    for _, part in ipairs(trackedParts) do  -- ใช้ cached list
        -- process
    end
end)
```

---

## 12. แบบฝึกหัด

### แบบฝึกหัดที่ 1: Performance Audit
สร้าง Script ที่วิเคราะห์ Game ของคุณ:
- นับ Parts ทั้งหมดและแยกตาม Type
- หา 10 Models ที่มี Parts มากที่สุด
- ตรวจสอบ Parts ที่ไม่ได้ Anchor (อาจเป็น Physics overhead)
- แสดงผลในรูปแบบ Report

### แบบฝึกหัดที่ 2: LOD Implementation
สร้างระบบ LOD สำหรับ Tree ใน Forest:
- ระยะ < 30: แสดงทั้ง Trunk และ Leaves
- ระยะ 30-100: แสดงเฉพาะ Low-poly Mesh
- ระยะ > 100: ซ่อน
- วัด FPS ก่อนและหลังใช้ LOD

### แบบฝึกหัดที่ 3: Connection Profiler
สร้างเครื่องมือ Track จำนวน Active Connections:
- นับ Connections ที่สร้างและ Disconnect
- แจ้งเตือนเมื่อมี Connections เกิน 100
- แสดง Top 5 Scripts ที่มี Connections มากที่สุด

---

## สรุป

การปรับแต่งประสิทธิภาพเป็นกระบวนการต่อเนื่อง ไม่ใช่ทำครั้งเดียวแล้วจบ หลักการสำคัญ:

1. **วัดก่อนปรับ**: ใช้ MicroProfiler หา Bottleneck จริงๆ
2. **Reduce Part Count**: Union, หลีกเลี่ยง Decorative Parts ที่มาก
3. **LOD System**: แสดงผลตามระยะทาง
4. **Object Pooling**: Reuse แทนสร้างใหม่
5. **Connection Cleanup**: Disconnect เสมอ
6. **Cache References**: หลีกเลี่ยง FindFirstChild ใน Loop
7. **Streaming Enabled**: ให้ Roblox จัดการ Stream ให้

ในส่วนถัดไป (Part 73) เราจะเจาะลึก Memory Management โดยเฉพาะ รวมถึงวิธีตรวจหา Memory Leak และการจัดการ Asset Loading อย่างมีประสิทธิภาพ
