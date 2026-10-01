# Part 70: Advanced Multiplayer Systems

## บทนำ

ระบบ Multiplayer ขั้นสูงเป็นหัวใจสำคัญของเกม Roblox ที่ดี ในบทนี้เราจะเรียนรู้เทคนิคขั้นสูง เช่น State Synchronization, Server-Authoritative Architecture, และการจัดการ Latency

---

## 70.1 Server-Authoritative Architecture

### 70.1.1 หลักการ

```
Client (ผู้เล่น)          Server (เซิร์ฟเวอร์)
      |                         |
      |-- Input (WASD) -------> |
      |                         |-- ประมวลผล
      |                         |-- ตรวจสอบ validity
      |<-- State Update --------|
      |                         |
```

```lua
-- ServerScriptService/GameStateManager.lua
-- ระบบจัดการ State หลักของเกม

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")

-- Game State ที่ Server เป็นเจ้าของ
local GameState = {
    phase = "lobby",    -- lobby, countdown, playing, ended
    players = {},       -- ข้อมูลผู้เล่นทั้งหมด
    startTime = nil,
    endTime = nil,
    winner = nil,
}

-- Remotes
local Remotes = ReplicatedStorage:WaitForChild("Remotes")

local stateUpdateEvent = Instance.new("RemoteEvent")
stateUpdateEvent.Name = "StateUpdate"
stateUpdateEvent.Parent = Remotes

-- ส่ง State ให้ผู้เล่นใหม่
local function sendFullState(player)
    stateUpdateEvent:FireClient(player, {
        type = "full_state",
        state = GameState
    })
end

-- อัพเดท State และแจ้งทุกคน
local function broadcastStateUpdate(updateData)
    stateUpdateEvent:FireAllClients({
        type = "delta_update",
        data = updateData,
        serverTime = tick()
    })
end

-- เปลี่ยน Phase
local function changePhase(newPhase)
    GameState.phase = newPhase
    broadcastStateUpdate({phase = newPhase})
    print("Game Phase: " .. newPhase)
end

-- จัดการผู้เล่นใหม่
Players.PlayerAdded:Connect(function(player)
    GameState.players[player.UserId] = {
        name = player.Name,
        joinTime = tick(),
        ready = false,
        score = 0,
    }
    
    sendFullState(player)
    broadcastStateUpdate({
        type = "player_joined",
        playerId = player.UserId,
        playerName = player.Name,
        playerCount = #Players:GetPlayers()
    })
end)

Players.PlayerRemoving:Connect(function(player)
    GameState.players[player.UserId] = nil
    broadcastStateUpdate({
        type = "player_left",
        playerId = player.UserId,
        playerCount = #Players:GetPlayers() - 1
    })
end)

return {
    getState = function() return GameState end,
    changePhase = changePhase,
    broadcastUpdate = broadcastStateUpdate,
    sendFullState = sendFullState
}
```

---

## 70.2 Client-Side Prediction

```lua
-- StarterPlayerScripts/ClientPrediction.lua
-- Prediction เพื่อลด Latency Effect

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local player = Players.LocalPlayer
local Remotes = ReplicatedStorage:WaitForChild("Remotes")

-- Input Buffer
local inputBuffer = {}
local inputSequence = 0
local pendingInputs = {}

-- สถานะที่ทำนายไว้ (predicted state)
local predictedPosition = Vector3.new(0, 0, 0)
local confirmedPosition = Vector3.new(0, 0, 0)

-- ส่ง Input ไป Server พร้อม sequence number
local playerInputRemote = Remotes:WaitForChild("PlayerInput")

local function sendInput(inputData)
    inputSequence = inputSequence + 1
    
    local input = {
        sequence = inputSequence,
        timestamp = tick(),
        moveDirection = inputData.moveDirection,
        jump = inputData.jump,
        lookDirection = inputData.lookDirection,
    }
    
    -- เก็บใน pending list
    pendingInputs[inputSequence] = {
        input = input,
        predictedPos = predictedPosition
    }
    
    playerInputRemote:FireServer(input)
end

-- ประมวลผล Input ฝั่ง Client (Prediction)
local function processInputLocally(input)
    local character = player.Character
    if not character or not character:FindFirstChild("HumanoidRootPart") then return end
    
    local humanoid = character:FindFirstChildWhichIsA("Humanoid")
    if not humanoid then return end
    
    -- คำนวณ movement prediction
    local moveDir = input.moveDirection
    if moveDir.Magnitude > 0 then
        humanoid:Move(moveDir, true)
    end
    
    if input.jump and humanoid.FloorMaterial ~= Enum.Material.Air then
        humanoid.Jump = true
    end
    
    -- อัพเดท predicted position
    predictedPosition = character.HumanoidRootPart.Position
end

-- รับ Server Confirmation
local serverConfirmRemote = Remotes:WaitForChild("ServerConfirm")

serverConfirmRemote.OnClientEvent:Connect(function(confirmedData)
    -- ลบ pending inputs ที่ถูก confirm แล้ว
    for seq = 1, confirmedData.lastProcessedInput do
        pendingInputs[seq] = nil
    end
    
    confirmedPosition = confirmedData.position
    
    -- Reconcile: ถ้า predicted position ผิดเกิน threshold
    local diff = (confirmedPosition - predictedPosition).Magnitude
    
    if diff > 5 then  -- มากกว่า 5 studs = ผิดพลาดมาก
        -- Correction: เลื่อน character ไปตำแหน่งจริง
        local character = player.Character
        if character and character:FindFirstChild("HumanoidRootPart") then
            character.HumanoidRootPart.CFrame = CFrame.new(confirmedPosition)
        end
        
        -- Re-apply pending inputs
        for _, pending in pairs(pendingInputs) do
            processInputLocally(pending.input)
        end
    end
end)

-- Input Loop
RunService.RenderStepped:Connect(function(dt)
    local character = player.Character
    if not character then return end
    
    -- เก็บ Input
    local moveDir = Vector3.new(0, 0, 0)
    
    if UserInputService:IsKeyDown(Enum.KeyCode.W) then
        moveDir = moveDir + Vector3.new(0, 0, -1)
    end
    if UserInputService:IsKeyDown(Enum.KeyCode.S) then
        moveDir = moveDir + Vector3.new(0, 0, 1)
    end
    if UserInputService:IsKeyDown(Enum.KeyCode.A) then
        moveDir = moveDir + Vector3.new(-1, 0, 0)
    end
    if UserInputService:IsKeyDown(Enum.KeyCode.D) then
        moveDir = moveDir + Vector3.new(1, 0, 0)
    end
    
    if moveDir.Magnitude > 0 then
        moveDir = moveDir.Unit
    end
    
    local input = {
        moveDirection = moveDir,
        jump = UserInputService:IsKeyDown(Enum.KeyCode.Space),
        lookDirection = workspace.CurrentCamera.CFrame.LookVector
    }
    
    -- Predict locally
    processInputLocally(input)
    
    -- Send to server (throttled)
    sendInput(input)
end)
```

---

## 70.3 Interpolation สำหรับ Remote Players

```lua
-- StarterPlayerScripts/RemotePlayerInterpolation.lua
-- ทำให้ผู้เล่นคนอื่นเคลื่อนที่ smooth

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")

local Remotes = ReplicatedStorage:WaitForChild("Remotes")
local positionUpdateEvent = Remotes:WaitForChild("PositionUpdate")

-- เก็บประวัติ position ของผู้เล่นแต่ละคน
local playerPositionHistory = {}
local INTERPOLATION_DELAY = 0.1  -- 100ms delay สำหรับ smooth interpolation

-- รับ position update
positionUpdateEvent.OnClientEvent:Connect(function(updates)
    local now = tick()
    
    for userId, data in pairs(updates) do
        if not playerPositionHistory[userId] then
            playerPositionHistory[userId] = {}
        end
        
        table.insert(playerPositionHistory[userId], {
            timestamp = now,
            position = data.position,
            rotation = data.rotation
        })
        
        -- เก็บประวัติแค่ 30 frames
        local history = playerPositionHistory[userId]
        while #history > 30 do
            table.remove(history, 1)
        end
    end
end)

-- Interpolate positions
RunService.RenderStepped:Connect(function()
    local renderTime = tick() - INTERPOLATION_DELAY
    
    for _, otherPlayer in ipairs(Players:GetPlayers()) do
        if otherPlayer == Players.LocalPlayer then continue end
        
        local history = playerPositionHistory[otherPlayer.UserId]
        if not history or #history < 2 then continue end
        
        local character = otherPlayer.Character
        if not character or not character:FindFirstChild("HumanoidRootPart") then continue end
        
        -- หา 2 frames ที่ต้อง interpolate ระหว่าง
        local before, after
        for i = 1, #history - 1 do
            if history[i].timestamp <= renderTime and history[i+1].timestamp >= renderTime then
                before = history[i]
                after = history[i+1]
                break
            end
        end
        
        if before and after then
            local alpha = (renderTime - before.timestamp) / (after.timestamp - before.timestamp)
            alpha = math.clamp(alpha, 0, 1)
            
            -- Lerp position
            local interpolatedPos = before.position:Lerp(after.position, alpha)
            
            -- Slerp rotation
            local beforeCF = CFrame.new(before.position) * before.rotation
            local afterCF = CFrame.new(after.position) * after.rotation
            local interpolatedCF = beforeCF:Lerp(afterCF, alpha)
            
            -- Apply (เบาๆ เพื่อไม่ให้กระตุก)
            character.HumanoidRootPart.CFrame = CFrame.new(
                character.HumanoidRootPart.Position:Lerp(interpolatedPos, 0.3),
                character.HumanoidRootPart.Position:Lerp(interpolatedPos, 0.3) + interpolatedCF.LookVector
            )
        end
    end
end)
```

---

## 70.4 ระบบ Room/Lobby

```lua
-- ServerScriptService/RoomSystem.lua
-- ระบบ Room สำหรับหลายเกม

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TeleportService = game:GetService("TeleportService")

local Remotes = ReplicatedStorage:WaitForChild("Remotes")
local roomRemote = Instance.new("RemoteEvent")
roomRemote.Name = "RoomSystem"
roomRemote.Parent = Remotes

-- Room Configuration
local ROOM_CONFIGS = {
    deathmatch = {
        name = "Deathmatch",
        minPlayers = 2,
        maxPlayers = 10,
        placeId = 1234567890,  -- Game Place ID
    },
    racing = {
        name = "Racing",
        minPlayers = 2,
        maxPlayers = 8,
        placeId = 9876543210,
    },
    cooperative = {
        name = "Co-op",
        minPlayers = 2,
        maxPlayers = 4,
        placeId = 1111111111,
    }
}

-- เก็บ Rooms
local activeRooms = {}
local playerRoom = {}  -- player -> roomId

-- สร้าง Room ใหม่
local function createRoom(gameType, creator)
    local config = ROOM_CONFIGS[gameType]
    if not config then
        warn("ไม่พบ game type: " .. gameType)
        return nil
    end
    
    local roomId = gameType .. "_" .. math.random(10000, 99999)
    
    activeRooms[roomId] = {
        id = roomId,
        gameType = gameType,
        config = config,
        creator = creator.UserId,
        players = {creator.UserId},
        status = "waiting",
        createdAt = tick()
    }
    
    playerRoom[creator.UserId] = roomId
    
    print(creator.Name .. " สร้าง Room " .. roomId)
    
    -- แจ้งผู้เล่นใน lobby
    broadcastRoomList()
    
    return roomId
end

-- เข้าร่วม Room
local function joinRoom(player, roomId)
    local room = activeRooms[roomId]
    if not room then
        roomRemote:FireClient(player, {type = "error", message = "ไม่พบ Room"})
        return false
    end
    
    if #room.players >= room.config.maxPlayers then
        roomRemote:FireClient(player, {type = "error", message = "Room เต็มแล้ว"})
        return false
    end
    
    if room.status ~= "waiting" then
        roomRemote:FireClient(player, {type = "error", message = "Room เริ่มเกมแล้ว"})
        return false
    end
    
    -- ออก Room เก่าก่อน
    if playerRoom[player.UserId] then
        leaveRoom(player)
    end
    
    table.insert(room.players, player.UserId)
    playerRoom[player.UserId] = roomId
    
    -- แจ้ง players ใน Room
    for _, playerId in ipairs(room.players) do
        local p = Players:GetPlayerByUserId(playerId)
        if p then
            roomRemote:FireClient(p, {
                type = "player_joined",
                room = room,
                playerName = player.Name
            })
        end
    end
    
    print(player.Name .. " เข้าร่วม Room " .. roomId)
    broadcastRoomList()
    return true
end

-- ออก Room
function leaveRoom(player)
    local roomId = playerRoom[player.UserId]
    if not roomId then return end
    
    local room = activeRooms[roomId]
    if room then
        -- ลบออกจาก players list
        for i, playerId in ipairs(room.players) do
            if playerId == player.UserId then
                table.remove(room.players, i)
                break
            end
        end
        
        -- ถ้าไม่มีผู้เล่นเหลือ ลบ Room
        if #room.players == 0 then
            activeRooms[roomId] = nil
        else
            -- ถ้า creator ออก เปลี่ยน creator
            if room.creator == player.UserId then
                room.creator = room.players[1]
            end
            
            -- แจ้ง players ที่เหลือ
            for _, playerId in ipairs(room.players) do
                local p = Players:GetPlayerByUserId(playerId)
                if p then
                    roomRemote:FireClient(p, {
                        type = "player_left",
                        room = room,
                        playerName = player.Name
                    })
                end
            end
        end
    end
    
    playerRoom[player.UserId] = nil
    broadcastRoomList()
    print(player.Name .. " ออก Room " .. roomId)
end

-- เริ่มเกม
local function startGame(player, roomId)
    local room = activeRooms[roomId]
    if not room then return end
    
    -- ตรวจสอบว่าเป็น creator
    if room.creator ~= player.UserId then
        roomRemote:FireClient(player, {type = "error", message = "เฉพาะเจ้าของ Room เท่านั้น"})
        return
    end
    
    if #room.players < room.config.minPlayers then
        roomRemote:FireClient(player, {
            type = "error",
            message = "ต้องการผู้เล่นอย่างน้อย " .. room.config.minPlayers .. " คน"
        })
        return
    end
    
    room.status = "starting"
    
    -- Teleport ผู้เล่นทุกคนไปเกม
    local playerList = {}
    for _, playerId in ipairs(room.players) do
        local p = Players:GetPlayerByUserId(playerId)
        if p then
            table.insert(playerList, p)
        end
    end
    
    -- ส่ง message แจ้งก่อน teleport
    for _, p in ipairs(playerList) do
        roomRemote:FireClient(p, {
            type = "game_starting",
            countdown = 5
        })
    end
    
    task.wait(5)
    
    -- Teleport
    local teleportData = TeleportService:ReserveServer(room.config.placeId)
    TeleportService:TeleportToPrivateServer(
        room.config.placeId,
        teleportData,
        playerList
    )
    
    print("Teleport " .. #playerList .. " players ไปเกม " .. room.config.name)
end

-- ส่งรายการ Rooms
function broadcastRoomList()
    local roomList = {}
    for roomId, room in pairs(activeRooms) do
        if room.status == "waiting" then
            table.insert(roomList, {
                id = roomId,
                name = room.config.name,
                gameType = room.gameType,
                players = #room.players,
                maxPlayers = room.config.maxPlayers,
                creatorName = Players:GetPlayerByUserId(room.creator) and 
                    Players:GetPlayerByUserId(room.creator).Name or "Unknown"
            })
        end
    end
    
    roomRemote:FireAllClients({
        type = "room_list",
        rooms = roomList
    })
end

-- Remote handlers
roomRemote.OnServerEvent:Connect(function(player, action, data)
    if action == "create" then
        createRoom(data.gameType, player)
    elseif action == "join" then
        joinRoom(player, data.roomId)
    elseif action == "leave" then
        leaveRoom(player)
    elseif action == "start" then
        startGame(player, playerRoom[player.UserId])
    elseif action == "get_list" then
        broadcastRoomList()
    end
end)

Players.PlayerRemoving:Connect(leaveRoom)
```

---

## 70.5 Synchronized Events

```lua
-- ServerScriptService/SyncedEvents.lua
-- ระบบ Synced Events สำหรับ Effect ที่ทุกคนต้องเห็นพร้อมกัน

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local Remotes = ReplicatedStorage:WaitForChild("Remotes")

-- สร้าง SyncedEvent Remote
local syncedEventRemote = Instance.new("RemoteEvent")
syncedEventRemote.Name = "SyncedEvent"
syncedEventRemote.Parent = Remotes

-- ส่ง Synced Event ไปทุกคน (เช่น explosion, ability effect)
local function fireSyncedEvent(eventType, data, targetPlayers)
    local eventData = {
        type = eventType,
        timestamp = tick(),  -- ใช้ tick() เป็น timestamp
        data = data
    }
    
    if targetPlayers then
        for _, player in ipairs(targetPlayers) do
            syncedEventRemote:FireClient(player, eventData)
        end
    else
        syncedEventRemote:FireAllClients(eventData)
    end
end

-- ตัวอย่าง: Explosion ที่ทุกคนเห็นพร้อมกัน
local function createSyncedExplosion(position, radius, damage)
    -- ทำ damage บน Server ก่อน
    local enemies = workspace:FindFirstChild("Enemies")
    if enemies then
        for _, enemy in ipairs(enemies:GetChildren()) do
            if enemy:FindFirstChild("HumanoidRootPart") then
                local dist = (enemy.HumanoidRootPart.Position - position).Magnitude
                if dist <= radius then
                    local humanoid = enemy:FindFirstChildWhichIsA("Humanoid")
                    if humanoid then
                        local falloff = 1 - (dist / radius)
                        humanoid:TakeDamage(damage * falloff)
                    end
                end
            end
        end
    end
    
    -- แจ้ง Client ให้แสดง visual effect
    fireSyncedEvent("explosion", {
        position = position,
        radius = radius,
        intensity = 1.0
    })
end

return {
    fireEvent = fireSyncedEvent,
    createExplosion = createSyncedExplosion
}
```

---

## 70.6 Anti-Lag Measures

```lua
-- ServerScriptService/AntiLag.lua
-- ระบบจัดการ Lag และ Latency

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

-- ติดตาม Ping ของผู้เล่น
local playerPings = {}

local function measurePing(player)
    -- Roblox ไม่มี API ตรงๆ สำหรับ ping
    -- แต่เราสามารถประมาณได้จาก network stats
    local stats = game:GetService("Stats")
    return stats.Network.ServerStatsItem["Data Ping"].Value
end

-- ปรับ Hitbox ตาม Lag (Server-side Lag Compensation)
local function getCompensatedPosition(player, timestamp)
    -- หา position ของ player ณ เวลาที่ยิง
    local latency = tick() - timestamp
    
    if latency > 0.5 then  -- lag มากกว่า 500ms ไม่ compensate
        return nil
    end
    
    -- ดึงจาก position history
    local history = getPlayerPositionHistory(player)
    if not history then return nil end
    
    -- หา position ที่ใกล้เคียง timestamp ที่สุด
    local targetTime = timestamp
    local closest = nil
    local closestDiff = math.huge
    
    for _, record in ipairs(history) do
        local diff = math.abs(record.time - targetTime)
        if diff < closestDiff then
            closestDiff = diff
            closest = record
        end
    end
    
    return closest and closest.position or nil
end

-- Position History
local positionHistories = {}

RunService.Heartbeat:Connect(function()
    for _, player in ipairs(Players:GetPlayers()) do
        if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            if not positionHistories[player.UserId] then
                positionHistories[player.UserId] = {}
            end
            
            local history = positionHistories[player.UserId]
            table.insert(history, {
                time = tick(),
                position = player.Character.HumanoidRootPart.Position,
                cframe = player.Character.HumanoidRootPart.CFrame
            })
            
            -- เก็บแค่ 1 วินาที (60 frames)
            while #history > 60 do
                table.remove(history, 1)
            end
        end
    end
end)

function getPlayerPositionHistory(player)
    return positionHistories[player.UserId]
end

return {
    getCompensatedPosition = getCompensatedPosition,
    getHistory = getPlayerPositionHistory
}
```

---

## 70.7 ข้อผิดพลาดที่พบบ่อย

```lua
-- ❌ ผิด: ส่งข้อมูลมากเกินไปทาง RemoteEvent
-- ทุกๆ frame:
RunService.Heartbeat:Connect(function()
    playerInputRemote:FireServer(allPlayerData)  -- ข้อมูลเยอะมาก!
end)

-- ✓ ถูก: ส่งเฉพาะข้อมูลที่เปลี่ยนแปลง (Delta)
local lastSentData = {}
RunService.Heartbeat:Connect(function()
    local currentData = getCurrentInput()
    
    -- เปรียบเทียบกับที่ส่งครั้งล่าสุด
    if hasChanged(currentData, lastSentData) then
        playerInputRemote:FireServer(currentData)
        lastSentData = currentData
    end
end)
```

---

## 70.8 แบบฝึกหัด

### แบบฝึกหัดที่ 1: สร้างระบบ Party
ระบบ Party ที่:
- สร้าง/เข้าร่วม Party กับเพื่อน
- Teleport ทั้ง Party ไปด้วยกัน
- แชร์ Rewards ในกลุ่ม

### แบบฝึกหัดที่ 2: Spectator Mode
โหมดดูเกม:
- ดูผู้เล่นคนอื่นเล่น
- เปลี่ยนเป้าหมายที่ดูได้
- First-person หรือ Free camera

### แบบฝึกหัดที่ 3: Cross-Server Chat
Chat ข้ามเซิร์ฟเวอร์:
- ใช้ MessagingService
- แสดงข้อความจากทุกเซิร์ฟเวอร์

---

## สรุป

ในบทนี้เราได้เรียนรู้:
- Server-Authoritative Architecture
- Client-Side Prediction
- Interpolation
- Room/Lobby System
- Synced Events
- Anti-Lag Measures

ในบทถัดไปเราจะเรียนรู้ Anti-Cheat Systems!
