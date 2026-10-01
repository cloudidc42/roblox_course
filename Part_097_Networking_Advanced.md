# Part 97: Advanced Networking และ Lag Compensation

## บทนำ (Introduction)

ระบบ Networking ที่ดีคือหัวใจของเกม Multiplayer ที่น่าเล่น ในบทนี้เราจะเรียนรู้เทคนิคขั้นสูง
ในการจัดการ Network latency, Client-side prediction, Server reconciliation,
และ Lag compensation เพื่อให้เกมรู้สึก responsive แม้ network ช้า

### สิ่งที่จะได้เรียนรู้:
- Remote Events/Functions ที่มีประสิทธิภาพ
- Client-side prediction (ทำ client ตอบสนองทันที)
- Server-side reconciliation (ตรวจสอบความถูกต้องจาก server)
- Lag compensation สำหรับ hit detection
- Network compression และ optimization
- Anti-cheat ระดับ network
- Rate limiting และ flood protection

---

## ส่วนที่ 1: Network Architecture

### 1.1 RemoteEvent Manager

```lua
-- ModuleScript: NetworkManager (Shared)
-- จัดการ Remote Events และ Functions อย่างเป็นระบบ
-- Manages Remote Events and Functions systematically

local NetworkManager = {}
NetworkManager.__index = NetworkManager

local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")

local IS_SERVER = RunService:IsServer()
local IS_CLIENT = RunService:IsClient()

-- โฟลเดอร์สำหรับ Remotes (Folder for remotes)
local REMOTES_FOLDER_NAME = "Remotes"

-- Event definitions (กำหนด events ทั้งหมดที่ใช้ในเกม)
local EVENT_DEFINITIONS = {
    -- Player actions
    PlayerAction = {type = "RemoteEvent"},
    RequestData = {type = "RemoteFunction"},
    
    -- Combat
    DealDamage = {type = "RemoteEvent"},
    AbilityActivated = {type = "RemoteEvent"},
    
    -- UI
    ShowNotification = {type = "RemoteEvent"},
    OpenShop = {type = "RemoteEvent"},
    
    -- Game state
    GameStateChanged = {type = "RemoteEvent"},
    SyncTime = {type = "RemoteFunction"},
    
    -- Movement (for prediction)
    MovePlayer = {type = "RemoteEvent"},
    PositionCorrection = {type = "RemoteEvent"},
}

-- Rate limiting (จำกัดความถี่ในการส่ง events)
local RATE_LIMITS = {
    PlayerAction = 20,      -- 20 ครั้งต่อวินาที
    DealDamage = 10,        -- 10 ครั้งต่อวินาที
    AbilityActivated = 5,   -- 5 ครั้งต่อวินาที
    MovePlayer = 60,        -- 60 ครั้งต่อวินาที (60fps)
}

-- สร้าง NetworkManager (Create NetworkManager)
function NetworkManager.new()
    local self = setmetatable({}, NetworkManager)
    
    self.Remotes = {}
    self.Handlers = {}
    self.RateLimitTracker = {}  -- {player -> {event -> {count, resetTime}}}
    
    -- สร้างหรือหา Remotes folder
    local remotesFolder
    if IS_SERVER then
        remotesFolder = ReplicatedStorage:FindFirstChild(REMOTES_FOLDER_NAME)
        if not remotesFolder then
            remotesFolder = Instance.new("Folder")
            remotesFolder.Name = REMOTES_FOLDER_NAME
            remotesFolder.Parent = ReplicatedStorage
        end
        
        -- สร้าง Remote instances
        for name, def in pairs(EVENT_DEFINITIONS) do
            local existing = remotesFolder:FindFirstChild(name)
            if not existing then
                local remote = Instance.new(def.type)
                remote.Name = name
                remote.Parent = remotesFolder
                self.Remotes[name] = remote
            else
                self.Remotes[name] = existing
            end
        end
    else
        -- Client: รอจนกว่า remotes จะพร้อม
        remotesFolder = ReplicatedStorage:WaitForChild(REMOTES_FOLDER_NAME, 10)
        if remotesFolder then
            for name in pairs(EVENT_DEFINITIONS) do
                self.Remotes[name] = remotesFolder:WaitForChild(name, 10)
            end
        end
    end
    
    return self
end

-- ลงทะเบียน Handler (Register event handler)
function NetworkManager:On(eventName, handler)
    local remote = self.Remotes[eventName]
    if not remote then
        warn("NetworkManager: Unknown event:", eventName)
        return
    end
    
    if remote:IsA("RemoteEvent") then
        if IS_SERVER then
            remote.OnServerEvent:Connect(function(player, ...)
                -- ตรวจสอบ rate limit
                if self:CheckRateLimit(player, eventName) then
                    handler(player, ...)
                end
            end)
        else
            remote.OnClientEvent:Connect(function(...)
                handler(...)
            end)
        end
    elseif remote:IsA("RemoteFunction") then
        if IS_SERVER then
            remote.OnServerInvoke = function(player, ...)
                if self:CheckRateLimit(player, eventName) then
                    return handler(player, ...)
                end
                return nil, "Rate limited"
            end
        else
            remote.OnClientInvoke = function(...)
                return handler(...)
            end
        end
    end
end

-- ส่ง event (Fire event)
function NetworkManager:Fire(eventName, target, ...)
    local remote = self.Remotes[eventName]
    if not remote then
        warn("NetworkManager: Unknown event:", eventName)
        return
    end
    
    if IS_SERVER then
        if remote:IsA("RemoteEvent") then
            if target == "AllClients" then
                remote:FireAllClients(...)
            elseif target then
                remote:FireClient(target, ...)
            end
        end
    else
        if remote:IsA("RemoteEvent") then
            remote:FireServer(...)
        elseif remote:IsA("RemoteFunction") then
            return remote:InvokeServer(...)
        end
    end
end

-- ตรวจสอบ Rate Limit (Check rate limit)
function NetworkManager:CheckRateLimit(player, eventName)
    local limit = RATE_LIMITS[eventName]
    if not limit then return true end  -- ไม่มี limit
    
    local now = tick()
    
    if not self.RateLimitTracker[player] then
        self.RateLimitTracker[player] = {}
    end
    
    local playerLimits = self.RateLimitTracker[player]
    
    if not playerLimits[eventName] then
        playerLimits[eventName] = {count = 0, resetTime = now + 1}
    end
    
    local eventLimit = playerLimits[eventName]
    
    -- Reset ถ้าหมดเวลา (Reset if time expired)
    if now >= eventLimit.resetTime then
        eventLimit.count = 0
        eventLimit.resetTime = now + 1
    end
    
    eventLimit.count = eventLimit.count + 1
    
    if eventLimit.count > limit then
        -- Rate limited! อาจเป็น cheat
        warn("Rate limit exceeded:", player.Name, eventName, eventLimit.count)
        return false
    end
    
    return true
end

-- ล้างข้อมูล player ที่ออกไป (Clean up when player leaves)
function NetworkManager:CleanupPlayer(player)
    self.RateLimitTracker[player] = nil
end

return NetworkManager
```

### 1.2 ระบบ Time Synchronization

```lua
-- Script: TimeSync (Server)
-- ซิงค์เวลาระหว่าง server และ client
-- Synchronize time between server and client

local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")

-- สร้าง RemoteFunction สำหรับ time sync
local timeSyncRemote = ReplicatedStorage:WaitForChild("Remotes"):WaitForChild("SyncTime")

-- Server time ที่แม่นยำ (Accurate server time)
local serverStartTime = os.clock()

timeSyncRemote.OnServerInvoke = function(player)
    return os.clock()  -- ส่งเวลา server กลับไป
end

-- ==================== Client-side TimeSyncModule ====================
-- LocalScript หรือ ModuleScript ฝั่ง client

local TimeSync = {}

local offset = 0          -- ผลต่างระหว่าง client และ server
local rtt = 0             -- Round-trip time (ping)
local synced = false

-- ทำการ sync (Perform time synchronization using NTP-like algorithm)
function TimeSync.Sync()
    local remotes = game.ReplicatedStorage:WaitForChild("Remotes")
    local syncRemote = remotes:WaitForChild("SyncTime")
    
    local samples = {}
    
    -- เก็บตัวอย่าง 5 ครั้ง (Collect 5 samples)
    for i = 1, 5 do
        local sendTime = os.clock()
        local serverTime = syncRemote:InvokeServer()
        local receiveTime = os.clock()
        
        local roundTrip = receiveTime - sendTime
        local estimatedServerTime = serverTime + roundTrip / 2
        local sampleOffset = estimatedServerTime - receiveTime
        
        table.insert(samples, {
            offset = sampleOffset,
            rtt = roundTrip,
        })
        
        task.wait(0.1)
    end
    
    -- เรียงตาม RTT และใช้ค่ากลาง (Sort by RTT and use median)
    table.sort(samples, function(a, b) return a.rtt < b.rtt end)
    
    local medianIndex = math.floor(#samples / 2) + 1
    offset = samples[medianIndex].offset
    rtt = samples[medianIndex].rtt
    synced = true
    
    print(string.format("Time sync complete. Offset: %.3fms, RTT: %.1fms", 
        offset * 1000, rtt * 1000))
    
    return offset, rtt
end

-- หาเวลา server ปัจจุบัน (Get current server time estimate)
function TimeSync.GetServerTime()
    if not synced then return os.clock() end
    return os.clock() + offset
end

-- หา ping (Get ping in ms)
function TimeSync.GetPing()
    return rtt * 1000
end

-- re-sync เป็นระยะๆ (Periodic re-sync)
function TimeSync.StartAutoSync(interval)
    interval = interval or 30  -- re-sync ทุก 30 วินาที
    
    task.spawn(function()
        while true do
            task.wait(interval)
            TimeSync.Sync()
        end
    end)
end

return TimeSync
```

---

## ส่วนที่ 2: Client-Side Prediction

### 2.1 Movement Prediction System

```lua
-- LocalScript: MovementPrediction
-- ระบบ Client-Side Prediction สำหรับการเคลื่อนที่
-- Client-Side Prediction for player movement

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local rootPart = character:WaitForChild("HumanoidRootPart")

-- Constants
local SEND_RATE = 1/20  -- ส่งข้อมูลการเคลื่อนที่ 20 ครั้งต่อวินาที
local MAX_POSITION_ERROR = 3  -- ผิดพลาดได้ไม่เกิน 3 studs ก่อน correction

-- State
local inputBuffer = {}      -- คิว inputs ที่รอส่ง
local pendingMoves = {}     -- moves ที่รอ server confirm
local sequenceNumber = 0    -- เลขลำดับ input
local lastSendTime = 0

-- Remote events
local remotes = ReplicatedStorage:WaitForChild("Remotes")
local moveEvent = remotes:WaitForChild("MovePlayer")
local correctionEvent = remotes:WaitForChild("PositionCorrection")

-- บันทึก snapshot ของ state (Record state snapshot)
local function captureState()
    return {
        position = rootPart.CFrame.Position,
        velocity = rootPart.AssemblyLinearVelocity,
        time = tick(),
    }
end

-- อ่าน input ปัจจุบัน (Read current input)
local function readInput()
    local moveDirection = Vector3.zero
    
    if UserInputService:IsKeyDown(Enum.KeyCode.W) or UserInputService:IsKeyDown(Enum.KeyCode.Up) then
        moveDirection = moveDirection + Vector3.new(0, 0, -1)
    end
    if UserInputService:IsKeyDown(Enum.KeyCode.S) or UserInputService:IsKeyDown(Enum.KeyCode.Down) then
        moveDirection = moveDirection + Vector3.new(0, 0, 1)
    end
    if UserInputService:IsKeyDown(Enum.KeyCode.A) or UserInputService:IsKeyDown(Enum.KeyCode.Left) then
        moveDirection = moveDirection + Vector3.new(-1, 0, 0)
    end
    if UserInputService:IsKeyDown(Enum.KeyCode.D) or UserInputService:IsKeyDown(Enum.KeyCode.Right) then
        moveDirection = moveDirection + Vector3.new(1, 0, 0)
    end
    
    if moveDirection.Magnitude > 0 then
        moveDirection = moveDirection.Unit
    end
    
    return {
        direction = moveDirection,
        jump = UserInputService:IsKeyDown(Enum.KeyCode.Space),
        sprint = UserInputService:IsKeyDown(Enum.KeyCode.LeftShift),
    }
end

-- ใช้ input บน client (Apply input on client for prediction)
local function applyInputLocally(input, dt)
    local speed = input.sprint and 24 or 16
    
    if input.direction.Magnitude > 0 then
        -- หมุนตามทิศที่กด (Rotate toward move direction)
        local camera = workspace.CurrentCamera
        local camCF = camera.CFrame
        local forward = Vector3.new(camCF.LookVector.X, 0, camCF.LookVector.Z).Unit
        local right = Vector3.new(camCF.RightVector.X, 0, camCF.RightVector.Z).Unit
        
        local worldDirection = forward * -input.direction.Z + right * input.direction.X
        
        humanoid:Move(worldDirection, false)
        humanoid.WalkSpeed = speed
    else
        humanoid:Move(Vector3.zero)
    end
    
    if input.jump then
        humanoid.Jump = true
    end
end

-- รับ correction จาก server (Receive position correction from server)
correctionEvent.OnClientEvent:Connect(function(correctedPosition, correctedVelocity, confirmedSequence)
    -- หาว่า correction error มากแค่ไหน
    local positionError = (rootPart.Position - correctedPosition).Magnitude
    
    if positionError > MAX_POSITION_ERROR then
        -- Error มากเกิน: teleport ไปตำแหน่งที่ถูก (Significant error: teleport to correct position)
        rootPart.CFrame = CFrame.new(correctedPosition)
        
        if correctedVelocity then
            rootPart.AssemblyLinearVelocity = correctedVelocity
        end
        
        -- ล้าง pending moves ที่ server ยืนยันแล้ว
        for i = #pendingMoves, 1, -1 do
            if pendingMoves[i].sequence <= confirmedSequence then
                table.remove(pendingMoves, i)
            end
        end
        
        -- Re-apply pending moves ที่ยังไม่ได้รับ confirm
        for _, move in ipairs(pendingMoves) do
            applyInputLocally(move.input, move.dt)
            task.wait()  -- รอ 1 frame
        end
    end
end)

-- Game loop
RunService.Heartbeat:Connect(function(dt)
    -- อ่านและ apply input บน client ทันที (Apply input immediately on client)
    local input = readInput()
    applyInputLocally(input, dt)
    
    -- บันทึก pending move
    sequenceNumber = sequenceNumber + 1
    table.insert(pendingMoves, {
        sequence = sequenceNumber,
        input = input,
        dt = dt,
        time = tick(),
        position = rootPart.CFrame.Position,
    })
    
    -- จำกัด buffer size (Limit buffer size)
    while #pendingMoves > 100 do
        table.remove(pendingMoves, 1)
    end
    
    -- ส่ง input ไปยัง server ตาม send rate (Send input to server at send rate)
    local now = tick()
    if now - lastSendTime >= SEND_RATE then
        lastSendTime = now
        
        -- Batch inputs (ส่งหลาย input พร้อมกัน)
        if #inputBuffer > 0 then
            moveEvent:FireServer(inputBuffer, sequenceNumber)
            inputBuffer = {}
        end
    end
    
    -- เพิ่ม input เข้า buffer
    table.insert(inputBuffer, {
        sequence = sequenceNumber,
        input = input,
        dt = dt,
    })
end)
```

### 2.2 Server Reconciliation

```lua
-- Script: MovementServer
-- Server ตรวจสอบและ reconcile การเคลื่อนที่ของ client
-- Server validates and reconciles client movement

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local remotes = ReplicatedStorage:WaitForChild("Remotes")
local moveEvent = remotes:WaitForChild("MovePlayer")
local correctionEvent = remotes:WaitForChild("PositionCorrection")

-- ข้อมูล player (Player data)
local playerData = {}

-- Constants สำหรับตรวจสอบ
local MAX_SPEED = 28          -- ความเร็วสูงสุดที่อนุญาต
local MAX_POSITION_JUMP = 20  -- teleport ได้ไม่เกิน 20 studs ต่อวินาที
local CORRECTION_THRESHOLD = 2 -- ส่ง correction ถ้าผิด > 2 studs

Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function(character)
        local rootPart = character:WaitForChild("HumanoidRootPart")
        
        playerData[player] = {
            lastPosition = rootPart.Position,
            lastTime = tick(),
            confirmedSequence = 0,
        }
    end)
end)

Players.PlayerRemoving:Connect(function(player)
    playerData[player] = nil
end)

-- รับ movement input จาก client (Receive movement input from client)
moveEvent.OnServerEvent:Connect(function(player, inputBatch, latestSequence)
    local character = player.Character
    if not character then return end
    
    local rootPart = character:FindFirstChild("HumanoidRootPart")
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if not rootPart or not humanoid then return end
    
    local data = playerData[player]
    if not data then return end
    
    local now = tick()
    local deltaTime = now - data.lastTime
    data.lastTime = now
    
    -- ตรวจสอบ position เพื่อหา teleport/speedhack (Check for teleport/speedhack)
    local currentPos = rootPart.Position
    local positionDelta = (currentPos - data.lastPosition).Magnitude
    local maxExpectedDelta = MAX_SPEED * deltaTime * 1.2  -- 20% tolerance
    
    if positionDelta > maxExpectedDelta and deltaTime < 1 then
        -- อาจเป็น speedhack หรือ teleport hack
        warn(player.Name, "possible speedhack detected:", 
            positionDelta, "studs in", deltaTime, "seconds")
        
        -- ส่ง correction กลับไป
        correctionEvent:FireClient(player, data.lastPosition, Vector3.zero, latestSequence)
        
        -- Teleport กลับ
        rootPart.CFrame = CFrame.new(data.lastPosition)
        return
    end
    
    -- อัพเดทข้อมูล (Update data)
    data.lastPosition = currentPos
    data.confirmedSequence = latestSequence
    
    -- Server เปรียบเทียบตำแหน่งที่คาดหวัง (Server compares expected vs actual position)
    -- ในเกมจริงอาจต้องทำ server-side physics simulation
    -- สำหรับตัวอย่างนี้ เราแค่ validate speed
    
    -- ส่ง correction ถ้าจำเป็น (Send correction if needed)
    -- (ในกรณีนี้เราไม่ส่ง correction เพราะ position ถูกต้อง)
end)

-- Periodic position check (ตรวจสอบตำแหน่งเป็นระยะๆ)
RunService.Heartbeat:Connect(function()
    for player, data in pairs(playerData) do
        local character = player.Character
        if character then
            local rootPart = character:FindFirstChild("HumanoidRootPart")
            if rootPart then
                data.lastPosition = rootPart.Position
            end
        end
    end
end)
```

---

## ส่วนที่ 3: Lag Compensation สำหรับ Hit Detection

### 3.1 Lag Compensation System

```lua
-- ModuleScript: LagCompensation
-- ระบบชดเชย lag สำหรับการยิง/โจมตี
-- Lag compensation system for shooting/attacking

local LagCompensation = {}
LagCompensation.__index = LagCompensation

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

-- เก็บประวัติ position ของผู้เล่น
local HISTORY_DURATION = 1.0   -- เก็บย้อนหลัง 1 วินาที
local HISTORY_RATE = 0.05      -- บันทึกทุก 0.05 วินาที (20fps)
local MAX_HISTORY_ENTRIES = math.ceil(HISTORY_DURATION / HISTORY_RATE)

-- สร้าง LagCompensation (Create lag compensation manager)
function LagCompensation.new()
    local self = setmetatable({}, LagCompensation)
    
    self.PlayerHistories = {}  -- {player -> [{time, positions}]}
    self._lastHistoryTime = 0
    
    -- เริ่มบันทึก history
    self._recordConnection = RunService.Heartbeat:Connect(function()
        local now = tick()
        if now - self._lastHistoryTime >= HISTORY_RATE then
            self._lastHistoryTime = now
            self:RecordPositions(now)
        end
    end)
    
    -- ลบ history เมื่อ player ออก
    Players.PlayerRemoving:Connect(function(player)
        self.PlayerHistories[player] = nil
    end)
    
    return self
end

-- บันทึก positions (Record player positions)
function LagCompensation:RecordPositions(timestamp)
    for _, player in ipairs(Players:GetPlayers()) do
        local character = player.Character
        if not character then continue end
        
        if not self.PlayerHistories[player] then
            self.PlayerHistories[player] = {}
        end
        
        local history = self.PlayerHistories[player]
        
        -- บันทึก hitboxes (Record hitbox positions)
        local snapshot = {
            time = timestamp,
            parts = {},
        }
        
        for _, part in ipairs(character:GetDescendants()) do
            if part:IsA("BasePart") then
                snapshot.parts[part.Name] = {
                    cframe = part.CFrame,
                    size = part.Size,
                }
            end
        end
        
        table.insert(history, snapshot)
        
        -- ลบ entries เก่า (Remove old entries)
        while #history > MAX_HISTORY_ENTRIES do
            table.remove(history, 1)
        end
    end
end

-- หา snapshot ที่ใกล้เคียงกับเวลาที่ต้องการ (Find snapshot near target time)
function LagCompensation:GetSnapshot(player, targetTime)
    local history = self.PlayerHistories[player]
    if not history or #history == 0 then return nil end
    
    -- หา snapshot ที่ใกล้ที่สุด (Binary search for closest snapshot)
    local low = 1
    local high = #history
    
    while low < high do
        local mid = math.floor((low + high) / 2)
        if history[mid].time < targetTime then
            low = mid + 1
        else
            high = mid
        end
    end
    
    -- Interpolate ระหว่าง 2 snapshots ที่อยู่ก่อนและหลัง
    if low > 1 and low <= #history then
        local before = history[low - 1]
        local after = history[low]
        
        local t = (targetTime - before.time) / (after.time - before.time)
        t = math.clamp(t, 0, 1)
        
        -- Interpolated snapshot
        local interpolated = {
            time = targetTime,
            parts = {},
        }
        
        for partName, beforeData in pairs(before.parts) do
            local afterData = after.parts[partName]
            if afterData then
                -- Lerp CFrame
                interpolated.parts[partName] = {
                    cframe = beforeData.cframe:Lerp(afterData.cframe, t),
                    size = beforeData.size,
                }
            else
                interpolated.parts[partName] = beforeData
            end
        end
        
        return interpolated
    end
    
    return history[low]
end

-- ตรวจสอบ hit ด้วย lag compensation (Check hit with lag compensation)
function LagCompensation:CheckHit(shooterPlayer, targetPlayer, rayOrigin, rayDirection, shootTime)
    -- คำนวณ lag ของ shooter (Calculate shooter's lag)
    local shooterPing = shooterPlayer:GetNetworkPing()
    local compensatedTime = shootTime - shooterPing
    
    -- จำกัด compensation ไม่เกิน 300ms
    compensatedTime = math.max(compensatedTime, tick() - 0.3)
    
    -- หา snapshot ของ target ณ เวลาที่ยิง
    local snapshot = self:GetSnapshot(targetPlayer, compensatedTime)
    if not snapshot then return false, nil end
    
    -- เช็ค ray กับ hitboxes ของ target (Check ray against target hitboxes)
    local hitPart = nil
    local hitDistance = math.huge
    
    for partName, partData in pairs(snapshot.parts) do
        -- สร้าง virtual hitbox (Create virtual hitbox at compensated position)
        local partCFrame = partData.cframe
        local partSize = partData.size
        
        -- Ray vs OBB (Oriented Bounding Box) test
        local localRayOrigin = partCFrame:PointToObjectSpace(rayOrigin)
        local localRayDirection = partCFrame:VectorToObjectSpace(rayDirection)
        
        local halfSize = partSize / 2
        
        -- AABB test ใน local space
        local tMin = -math.huge
        local tMax = math.huge
        
        for axis = 1, 3 do
            local dirs = {localRayDirection.X, localRayDirection.Y, localRayDirection.Z}
            local origins = {localRayOrigin.X, localRayOrigin.Y, localRayOrigin.Z}
            local sizes = {halfSize.X, halfSize.Y, halfSize.Z}
            
            if math.abs(dirs[axis]) < 1e-8 then
                if math.abs(origins[axis]) > sizes[axis] then
                    tMin = math.huge  -- ไม่ชน
                    break
                end
            else
                local t1 = (-sizes[axis] - origins[axis]) / dirs[axis]
                local t2 = (sizes[axis] - origins[axis]) / dirs[axis]
                
                if t1 > t2 then t1, t2 = t2, t1 end
                
                tMin = math.max(tMin, t1)
                tMax = math.min(tMax, t2)
                
                if tMin > tMax then break end
            end
        end
        
        if tMin <= tMax and tMin > 0 and tMin < hitDistance then
            hitDistance = tMin
            hitPart = partName
        end
    end
    
    if hitPart then
        local hitPosition = rayOrigin + rayDirection * hitDistance
        return true, {
            part = hitPart,
            position = hitPosition,
            distance = hitDistance,
        }
    end
    
    return false, nil
end

-- หยุดระบบ (Stop system)
function LagCompensation:Destroy()
    if self._recordConnection then
        self._recordConnection:Disconnect()
    end
end

return LagCompensation
```

---

## ส่วนที่ 4: Network Optimization

### 4.1 Data Compression

```lua
-- ModuleScript: NetworkCompressor
-- บีบอัดข้อมูลก่อนส่งผ่าน network
-- Compress data before sending over network

local NetworkCompressor = {}

-- บีบอัด Vector3 (Compress Vector3 to integers)
-- ช่วยลดขนาดข้อมูลที่ส่ง
function NetworkCompressor.CompressPosition(position, precision)
    precision = precision or 10  -- เก็บ 1 decimal place
    
    return {
        math.round(position.X * precision),
        math.round(position.Y * precision),
        math.round(position.Z * precision),
        precision,
    }
end

function NetworkCompressor.DecompressPosition(data)
    local precision = data[4] or 10
    return Vector3.new(
        data[1] / precision,
        data[2] / precision,
        data[3] / precision
    )
end

-- บีบอัด CFrame (Compress CFrame)
function NetworkCompressor.CompressCFrame(cf, precision)
    precision = precision or 100
    
    local pos = cf.Position
    local rx, ry, rz = cf:ToEulerAnglesXYZ()
    
    return {
        math.round(pos.X * precision),
        math.round(pos.Y * precision),
        math.round(pos.Z * precision),
        math.round(rx * 1000),
        math.round(ry * 1000),
        math.round(rz * 1000),
        precision,
    }
end

function NetworkCompressor.DecompressCFrame(data)
    local precision = data[7] or 100
    
    local pos = Vector3.new(
        data[1] / precision,
        data[2] / precision,
        data[3] / precision
    )
    
    local rx = data[4] / 1000
    local ry = data[5] / 1000
    local rz = data[6] / 1000
    
    return CFrame.new(pos) * CFrame.fromEulerAnglesXYZ(rx, ry, rz)
end

-- บีบอัด Color3 (Compress Color3 to single integer)
function NetworkCompressor.CompressColor(color)
    local r = math.round(color.R * 255)
    local g = math.round(color.G * 255)
    local b = math.round(color.B * 255)
    
    -- Pack into single integer (24-bit color)
    return r * 65536 + g * 256 + b
end

function NetworkCompressor.DecompressColor(packed)
    local r = math.floor(packed / 65536)
    local g = math.floor((packed % 65536) / 256)
    local b = packed % 256
    
    return Color3.fromRGB(r, g, b)
end

-- บีบอัด array ของ positions (Compress array of positions using delta encoding)
function NetworkCompressor.CompressPositionArray(positions, precision)
    precision = precision or 10
    
    if #positions == 0 then return {} end
    
    local compressed = {}
    
    -- First position เก็บ absolute
    local first = positions[1]
    compressed[1] = {
        math.round(first.X * precision),
        math.round(first.Y * precision),
        math.round(first.Z * precision),
    }
    
    -- ตำแหน่งถัดไปเก็บเป็น delta (Subsequent positions stored as deltas)
    local prevX = compressed[1][1]
    local prevY = compressed[1][2]
    local prevZ = compressed[1][3]
    
    for i = 2, #positions do
        local pos = positions[i]
        local cx = math.round(pos.X * precision)
        local cy = math.round(pos.Y * precision)
        local cz = math.round(pos.Z * precision)
        
        compressed[i] = {
            cx - prevX,  -- delta X
            cy - prevY,  -- delta Y
            cz - prevZ,  -- delta Z
        }
        
        prevX, prevY, prevZ = cx, cy, cz
    end
    
    return compressed, precision
end

function NetworkCompressor.DecompressPositionArray(compressed, precision)
    if #compressed == 0 then return {} end
    
    precision = precision or 10
    local positions = {}
    
    local x = compressed[1][1]
    local y = compressed[1][2]
    local z = compressed[1][3]
    
    positions[1] = Vector3.new(x / precision, y / precision, z / precision)
    
    for i = 2, #compressed do
        x = x + compressed[i][1]
        y = y + compressed[i][2]
        z = z + compressed[i][3]
        
        positions[i] = Vector3.new(x / precision, y / precision, z / precision)
    end
    
    return positions
end

return NetworkCompressor
```

### 4.2 Event Batching System

```lua
-- ModuleScript: EventBatcher
-- รวม events หลายตัวและส่งพร้อมกัน
-- Batch multiple events and send together

local EventBatcher = {}
EventBatcher.__index = EventBatcher

local RunService = game:GetService("RunService")

-- สร้าง EventBatcher (Create EventBatcher)
function EventBatcher.new(remote, sendRate)
    local self = setmetatable({}, EventBatcher)
    
    self.Remote = remote
    self.SendRate = sendRate or (1/20)  -- 20 Hz default
    self.Buffer = {}
    self.LastSendTime = 0
    
    -- Auto-flush timer
    self._connection = RunService.Heartbeat:Connect(function()
        local now = tick()
        if now - self.LastSendTime >= self.SendRate then
            self:Flush()
            self.LastSendTime = now
        end
    end)
    
    return self
end

-- เพิ่ม event เข้า buffer (Add event to buffer)
function EventBatcher:Add(eventType, data)
    table.insert(self.Buffer, {type = eventType, data = data, time = tick()})
end

-- ส่ง buffer (Flush buffer)
function EventBatcher:Flush()
    if #self.Buffer == 0 then return end
    
    local toSend = self.Buffer
    self.Buffer = {}
    
    -- ส่งแบบ batch
    self.Remote:FireServer(toSend)
end

-- หยุดระบบ (Stop system)
function EventBatcher:Destroy()
    if self._connection then
        self._connection:Disconnect()
    end
    self:Flush()  -- ส่ง events ที่เหลือ
end

return EventBatcher
```

---

## ส่วนที่ 5: Anti-Cheat ระดับ Network

### 5.1 Server-Side Validation

```lua
-- ModuleScript: ServerValidator
-- ตรวจสอบความถูกต้องของ actions จาก client ฝั่ง server
-- Server-side validation of client actions

local ServerValidator = {}
ServerValidator.__index = ServerValidator

local Players = game:GetService("Players")

-- สร้าง validator (Create validator)
function ServerValidator.new()
    local self = setmetatable({}, ServerValidator)
    
    self.PlayerStats = {}   -- เก็บสถิติ actions
    self.SuspiciousFlags = {}
    
    Players.PlayerAdded:Connect(function(player)
        self.PlayerStats[player] = {
            actions = {},
            flagCount = 0,
        }
    end)
    
    Players.PlayerRemoving:Connect(function(player)
        self.PlayerStats[player] = nil
        self.SuspiciousFlags[player] = nil
    end)
    
    return self
end

-- ตรวจสอบ action (Validate action)
function ServerValidator:ValidateAction(player, actionType, actionData)
    local stats = self.PlayerStats[player]
    if not stats then return false, "No player data" end
    
    local now = tick()
    
    -- ตรวจสอบตาม action type
    if actionType == "Attack" then
        return self:ValidateAttack(player, actionData, now)
    elseif actionType == "Move" then
        return self:ValidateMove(player, actionData, now)
    elseif actionType == "UseAbility" then
        return self:ValidateAbility(player, actionData, now)
    end
    
    return true, nil
end

-- ตรวจสอบการโจมตี (Validate attack)
function ServerValidator:ValidateAttack(player, data, now)
    local character = player.Character
    if not character then return false, "No character" end
    
    local stats = self.PlayerStats[player]
    
    -- ตรวจสอบ attack speed (Check attack speed)
    local lastAttack = stats.lastAttackTime or 0
    local minAttackInterval = 0.3  -- โจมตีได้ทุก 300ms อย่างน้อย
    
    if now - lastAttack < minAttackInterval then
        self:FlagPlayer(player, "AttackSpeed")
        return false, "Attack too fast"
    end
    
    -- ตรวจสอบระยะโจมตี (Check attack range)
    local targetPlayer = Players:GetPlayerByUserId(data.targetUserId)
    if targetPlayer and targetPlayer.Character then
        local attackerRoot = character:FindFirstChild("HumanoidRootPart")
        local targetRoot = targetPlayer.Character:FindFirstChild("HumanoidRootPart")
        
        if attackerRoot and targetRoot then
            local distance = (attackerRoot.Position - targetRoot.Position).Magnitude
            local maxRange = data.weaponRange or 8  -- ระยะสูงสุดตาม weapon
            
            if distance > maxRange * 1.3 then  -- 30% tolerance
                self:FlagPlayer(player, "AttackRange")
                return false, "Target out of range"
            end
        end
    end
    
    -- ตรวจสอบ damage amount (Check damage amount)
    local maxDamage = data.weaponDamage or 50
    if data.damage and data.damage > maxDamage * 1.1 then
        self:FlagPlayer(player, "DamageHack")
        return false, "Invalid damage amount"
    end
    
    stats.lastAttackTime = now
    return true, nil
end

-- ตรวจสอบการเคลื่อนที่ (Validate movement)
function ServerValidator:ValidateMove(player, data, now)
    local character = player.Character
    if not character then return false, "No character" end
    
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if not humanoid then return false, "No humanoid" end
    
    -- ตรวจสอบว่า player ไม่ได้ fly หรือ noclip
    local rootPart = character:FindFirstChild("HumanoidRootPart")
    if rootPart then
        local position = rootPart.Position
        
        -- ตรวจสอบว่าอยู่บนพื้นหรือในอากาศที่สมเหตุสมผล
        local raycastParams = RaycastParams.new()
        raycastParams.FilterDescendantsInstances = {character}
        
        local groundCheck = workspace:Raycast(position, Vector3.new(0, -10, 0), raycastParams)
        
        if not groundCheck then
            -- อยู่สูงมากเกิน อาจเป็น fly hack
            if position.Y > 500 then
                self:FlagPlayer(player, "FlyHack")
                return false, "Suspicious position"
            end
        end
    end
    
    return true, nil
end

-- ตรวจสอบการใช้ ability (Validate ability usage)
function ServerValidator:ValidateAbility(player, data, now)
    local stats = self.PlayerStats[player]
    
    -- ตรวจสอบว่า ability นั้น unlock แล้วหรือยัง
    -- (อ้างอิงจาก player data store)
    
    -- ตรวจสอบ cooldown
    local abilityId = data.abilityId
    local cooldownKey = "ability_" .. tostring(abilityId)
    local lastUsed = stats.actions[cooldownKey] or 0
    local cooldown = data.cooldown or 5
    
    if now - lastUsed < cooldown then
        self:FlagPlayer(player, "AbilityCooldown")
        return false, "Ability on cooldown"
    end
    
    stats.actions[cooldownKey] = now
    return true, nil
end

-- Flag suspicious player (Flag suspicious activity)
function ServerValidator:FlagPlayer(player, reason)
    local stats = self.PlayerStats[player]
    if not stats then return end
    
    stats.flagCount = stats.flagCount + 1
    
    if not self.SuspiciousFlags[player] then
        self.SuspiciousFlags[player] = {}
    end
    
    table.insert(self.SuspiciousFlags[player], {
        reason = reason,
        time = tick(),
    })
    
    warn("Suspicious activity:", player.Name, reason, "- Flag count:", stats.flagCount)
    
    -- Auto-kick ถ้า flags มากเกินไป
    if stats.flagCount >= 10 then
        player:Kick("Kicked for suspicious activity")
    end
end

-- รายงานสถิติ (Report statistics)
function ServerValidator:GetPlayerReport(player)
    return {
        flags = self.SuspiciousFlags[player] or {},
        flagCount = self.PlayerStats[player] and self.PlayerStats[player].flagCount or 0,
    }
end

return ServerValidator
```

---

## ส่วนที่ 6: ตัวอย่าง Combat System แบบ Complete

### 6.1 Combat ที่ใช้ Lag Compensation และ Server Validation

```lua
-- Script: CombatSystem (Server)
-- ระบบต่อสู้ที่รวม lag compensation และ server validation
-- Combat system with lag compensation and server validation

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local LagCompensation = require(game.ServerScriptService.LagCompensation)
local ServerValidator = require(game.ServerScriptService.ServerValidator)

local lagComp = LagCompensation.new()
local validator = ServerValidator.new()

local remotes = ReplicatedStorage:WaitForChild("Remotes")
local dealDamageRemote = remotes:WaitForChild("DealDamage")

-- รับ damage request จาก client (Receive damage request from client)
dealDamageRemote.OnServerEvent:Connect(function(player, attackData)
    -- Validate action
    local valid, reason = validator:ValidateAction(player, "Attack", attackData)
    if not valid then
        warn("Invalid attack from", player.Name, ":", reason)
        return
    end
    
    -- หา target player (Find target player)
    local targetPlayer = Players:GetPlayerByUserId(attackData.targetUserId)
    if not targetPlayer or not targetPlayer.Character then return end
    
    -- ใช้ lag compensation ตรวจสอบ hit (Use lag compensation to verify hit)
    local character = player.Character
    if not character then return end
    
    local rootPart = character:FindFirstChild("HumanoidRootPart")
    if not rootPart then return end
    
    -- คำนวณ ray จาก attacker ไปยัง target (Calculate ray from attacker to target)
    local targetCharRoot = targetPlayer.Character:FindFirstChild("HumanoidRootPart")
    if not targetCharRoot then return end
    
    local rayOrigin = rootPart.Position + Vector3.new(0, 1.5, 0)
    local rayDirection = (targetCharRoot.Position - rayOrigin).Unit * attackData.weaponRange
    
    -- ตรวจสอบ hit ด้วย lag compensation
    local shootTime = attackData.timestamp or tick()
    local hitConfirmed, hitInfo = lagComp:CheckHit(
        player, 
        targetPlayer, 
        rayOrigin, 
        rayDirection, 
        shootTime
    )
    
    if hitConfirmed then
        -- คำนวณ damage (Calculate damage)
        local baseDamage = attackData.damage or 10
        
        -- Headshot bonus
        local damageMultiplier = 1.0
        if hitInfo.part == "Head" then
            damageMultiplier = 2.0  -- 2x damage for headshot
        elseif hitInfo.part and hitInfo.part:find("Leg") then
            damageMultiplier = 0.7  -- 0.7x for leg shots
        end
        
        local finalDamage = math.round(baseDamage * damageMultiplier)
        
        -- ใช้ damage กับ target (Apply damage to target)
        local targetHumanoid = targetPlayer.Character:FindFirstChildOfClass("Humanoid")
        if targetHumanoid then
            targetHumanoid:TakeDamage(finalDamage)
            
            -- แจ้งผล hit ให้ attacker (Notify attacker of hit)
            game.ReplicatedStorage.Events.HitConfirmed:FireClient(player, {
                damage = finalDamage,
                hitPart = hitInfo.part,
                targetName = targetPlayer.Name,
            })
        end
    end
end)
```

---

## แบบฝึกหัด (Practice Exercises)

### แบบฝึกหัดที่ 1: Ping Display
```lua
-- สร้าง HUD ที่แสดง ping ของผู้เล่นแบบ real-time
-- สี: เขียว (<50ms), เหลือง (50-150ms), แดง (>150ms)
-- อัพเดททุก 2 วินาที
```

### แบบฝึกหัดที่ 2: Movement Interpolation
```lua
-- สร้างระบบ interpolation สำหรับตัวละครของผู้เล่นคนอื่น
-- เพื่อให้การเคลื่อนที่ดูลื่นไหลแม้ network lag
-- ใช้ lerp() กับ position history
```

### แบบฝึกหัดที่ 3: Network Debug Panel
```lua
-- สร้าง debug panel ที่แสดง:
-- - Ping
-- - Packets sent/received per second
-- - Network budget used (bytes)
-- - Rate limit status ของแต่ละ event
```

---

## สรุป (Summary)

ในบทนี้เราได้เรียนรู้:

1. **NetworkManager** - จัดการ Remote Events/Functions อย่างเป็นระบบพร้อม rate limiting
2. **Time Synchronization** - NTP-like algorithm สำหรับ sync เวลาระหว่าง server และ client
3. **Client-Side Prediction** - ทำให้การเคลื่อนที่ responsive โดยไม่รอ server response
4. **Server Reconciliation** - Server ตรวจสอบและแก้ไข client position ที่ผิด
5. **Lag Compensation** - ระบบตรวจสอบ hit โดยคำนึงถึง latency ของผู้เล่น
6. **Network Compression** - ลดขนาดข้อมูลที่ส่งผ่าน network
7. **Event Batching** - รวม events หลายตัวส่งพร้อมกันเพื่อประหยัด bandwidth
8. **Server Validation** - ตรวจสอบ actions จาก client ฝั่ง server เพื่อป้องกัน cheat

### หลักการสำคัญ:
- **Never trust the client** - ตรวจสอบทุกอย่างบน server เสมอ
- **Client prediction** ทำให้เกมรู้สึก responsive แม้ ping สูง
- **Lag compensation** ช่วยให้ hit detection ยุติธรรมสำหรับผู้เล่นทุกคน
- **Rate limiting** ป้องกัน DDoS และ exploit ผ่าน remotes

*บทถัดไป: Part 98 - Roblox Studio Plugin Development*
