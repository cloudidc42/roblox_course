# ตอนที่ 20: ภาพรวม Roblox Services

## บทนำ

Services ใน Roblox คือ objects พิเศษที่ให้ functionality ต่างๆ สำหรับการพัฒนาเกม แต่ละ service มีหน้าที่เฉพาะ เช่น จัดการผู้เล่น จัดการ network, ควบคุม physics, เล่นเสียง และอื่นๆ อีกมากมาย

---

## 20.1 การเข้าถึง Services

```lua
-- วิธีที่ถูกต้อง: game:GetService()
local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local ServerStorage = game:GetService("ServerStorage")
local ServerScriptService = game:GetService("ServerScriptService")
local StarterGui = game:GetService("StarterGui")
local StarterPack = game:GetService("StarterPack")
local StarterPlayer = game:GetService("StarterPlayer")
local Teams = game:GetService("Teams")
local SoundService = game:GetService("SoundService")
local Lighting = game:GetService("Lighting")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local HttpService = game:GetService("HttpService")
local DataStoreService = game:GetService("DataStoreService")
local UserInputService = game:GetService("UserInputService")
local ContextActionService = game:GetService("ContextActionService")
local CollectionService = game:GetService("CollectionService")
local MarketplaceService = game:GetService("MarketplaceService")
local BadgeService = game:GetService("BadgeService")
local GroupService = game:GetService("GroupService")
local GuiService = game:GetService("GuiService")
local InsertService = game:GetService("InsertService")
local PathfindingService = game:GetService("PathfindingService")
local PhysicsService = game:GetService("PhysicsService")
local ChatService = game:GetService("Chat")
local AnalyticsService = game:GetService("AnalyticsService")
```

---

## 20.2 Services ที่สำคัญที่สุด

### Players Service

```lua
local Players = game:GetService("Players")

-- ข้อมูลพื้นฐาน
print(Players.MaxPlayers)     -- จำนวนผู้เล่นสูงสุด
print(Players.NumPlayers)     -- จำนวนผู้เล่นปัจจุบัน (deprecated, ใช้ #GetPlayers())
print(Players.LocalPlayer)    -- ผู้เล่นปัจจุบัน (LocalScript only)

-- Methods
local allPlayers = Players:GetPlayers()          -- return table ของผู้เล่น
local player = Players:GetPlayerByUserId(123456) -- หาด้วย UserId
local player2 = Players:FindFirstChild("PlayerName") -- หาด้วยชื่อ

-- Events
Players.PlayerAdded:Connect(function(player) end)
Players.PlayerRemoving:Connect(function(player) end)
```

### TweenService

```lua
local TweenService = game:GetService("TweenService")

-- สร้าง Tween animation
local part = workspace.MyPart
local tweenInfo = TweenInfo.new(
    2,                       -- duration (วินาที)
    Enum.EasingStyle.Quad,   -- easing style
    Enum.EasingDirection.Out, -- easing direction
    0,                       -- จำนวนครั้งที่ทำซ้ำ (0 = ไม่ทำซ้ำ)
    false,                   -- reverse?
    0                        -- delay ก่อนเริ่ม
)

local tween = TweenService:Create(part, tweenInfo, {
    Position = Vector3.new(0, 20, 0),  -- เป้าหมาย
    Transparency = 0.5,
    Size = Vector3.new(8, 2, 8)
})

tween:Play()

-- Events
tween.Completed:Connect(function(playbackState)
    if playbackState == Enum.PlaybackState.Completed then
        print("Tween เสร็จแล้ว!")
    end
end)

-- ตัวอย่าง: Easing Styles ต่างๆ
local styles = {
    Enum.EasingStyle.Linear,
    Enum.EasingStyle.Sine,
    Enum.EasingStyle.Quad,
    Enum.EasingStyle.Cubic,
    Enum.EasingStyle.Quart,
    Enum.EasingStyle.Quint,
    Enum.EasingStyle.Exponential,
    Enum.EasingStyle.Circular,
    Enum.EasingStyle.Back,
    Enum.EasingStyle.Bounce,
    Enum.EasingStyle.Elastic,
}
```

### RunService

```lua
local RunService = game:GetService("RunService")

-- ตรวจสอบ environment
print(RunService:IsServer())  -- true บน server
print(RunService:IsClient())  -- true บน client
print(RunService:IsStudio())  -- true ใน Studio

-- Game loop events
RunService.Heartbeat:Connect(function(dt)
    -- ทำงานทุก frame บน server และ client
end)

RunService.Stepped:Connect(function(time, dt)
    -- ก่อน physics step
end)

RunService.RenderStepped:Connect(function(dt)
    -- ก่อน render (client only)
end)

-- ใช้ Heartbeat สำหรับ game update
local lastUpdate = os.clock()
RunService.Heartbeat:Connect(function()
    local now = os.clock()
    local dt = now - lastUpdate
    lastUpdate = now
    
    -- updateGame(dt)
end)
```

### HttpService

```lua
local HttpService = game:GetService("HttpService")

-- Encode/Decode JSON
local data = {name = "สมชาย", score = 100, items = {"ดาบ", "โล่"}}
local json = HttpService:JSONEncode(data)
print(json)  -- {"name":"สมชาย","score":100,"items":["ดาบ","โล่"]}

local decoded = HttpService:JSONDecode(json)
print(decoded.name)    -- สมชาย
print(decoded.score)   -- 100

-- Generate GUID
local guid = HttpService:GenerateGUID(false)
print(guid)  -- เช่น "a1b2c3d4-e5f6-7890-abcd-ef1234567890"

-- HTTP Request (ต้อง enable HTTP requests ก่อน)
local success, response = pcall(function()
    return HttpService:GetAsync("https://api.example.com/data")
end)

if success then
    local data = HttpService:JSONDecode(response)
    print(data.someField)
else
    print("Error: " .. response)
end
```

### DataStoreService

```lua
local DataStoreService = game:GetService("DataStoreService")

-- สร้าง DataStore
local playerDataStore = DataStoreService:GetDataStore("PlayerData")
local globalStore = DataStoreService:GetDataStore("GlobalData")
local orderedStore = DataStoreService:GetOrderedDataStore("Leaderboard")

-- บันทึกข้อมูล
local function savePlayerData(player, data)
    local key = "player_" .. player.UserId
    
    local success, err = pcall(function()
        playerDataStore:SetAsync(key, data)
    end)
    
    if success then
        print("บันทึกข้อมูล " .. player.Name .. " สำเร็จ")
    else
        warn("บันทึกข้อมูลล้มเหลว: " .. err)
    end
end

-- โหลดข้อมูล
local function loadPlayerData(player)
    local key = "player_" .. player.UserId
    
    local success, data = pcall(function()
        return playerDataStore:GetAsync(key)
    end)
    
    if success then
        return data or createDefaultData()
    else
        warn("โหลดข้อมูลล้มเหลว: " .. data)
        return createDefaultData()
    end
end

-- UpdateAsync (safe update)
local function addScore(player, points)
    local key = "player_" .. player.UserId
    
    local success, err = pcall(function()
        playerDataStore:UpdateAsync(key, function(oldData)
            oldData = oldData or {}
            oldData.score = (oldData.score or 0) + points
            return oldData
        end)
    end)
    
    return success
end
```

### Lighting Service

```lua
local Lighting = game:GetService("Lighting")

-- ตั้งค่า global lighting
Lighting.Brightness = 2
Lighting.Ambient = Color3.fromRGB(70, 70, 70)
Lighting.OutdoorAmbient = Color3.fromRGB(100, 100, 100)
Lighting.GlobalShadows = true

-- เวลาของวัน
Lighting.ClockTime = 14  -- 14:00 = บ่าย 2 โมง
Lighting.GeographicLatitude = 41.7

-- ฤดูกาล effects
local atmosphere = Lighting:FindFirstChildOfClass("Atmosphere")
if atmosphere then
    atmosphere.Density = 0.3
    atmosphere.Offset = 0
    atmosphere.Color = Color3.fromRGB(199, 199, 199)
    atmosphere.Decay = Color3.fromRGB(106, 127, 189)
    atmosphere.Glare = 0
    atmosphere.Haze = 0
end

-- Day/Night cycle
local function setDayNightCycle(speed)
    local RunService = game:GetService("RunService")
    
    RunService.Heartbeat:Connect(function(dt)
        Lighting.ClockTime = Lighting.ClockTime + dt * speed
        if Lighting.ClockTime >= 24 then
            Lighting.ClockTime = Lighting.ClockTime - 24
        end
    end)
end

setDayNightCycle(0.1)  -- 10% ความเร็วปกติ
```

---

## 20.3 Sound Service

```lua
local SoundService = game:GetService("SoundService")

-- ตั้งค่า global volume
SoundService.MusicVolume = 0.5
SoundService.AmbientReverb = Enum.ReverbType.NoReverb

-- เล่นเสียงใน workspace
local function playSound(soundId, parent, volume, pitch)
    local sound = Instance.new("Sound")
    sound.SoundId = "rbxassetid://" .. soundId
    sound.Volume = volume or 1
    sound.PlaybackSpeed = pitch or 1
    sound.Parent = parent
    sound:Play()
    
    -- ลบหลังเล่นเสร็จ
    sound.Ended:Connect(function()
        sound:Destroy()
    end)
    
    return sound
end

-- เล่นเพลง background
local function playBackgroundMusic(musicId)
    local music = Instance.new("Sound")
    music.SoundId = "rbxassetid://" .. musicId
    music.Volume = 0.3
    music.Looped = true
    music.Parent = SoundService
    music:Play()
    return music
end

-- เล่น sound effect ที่ตำแหน่ง
local function playSpatialSound(soundId, position)
    local part = Instance.new("Part")
    part.Position = position
    part.Anchored = true
    part.Transparency = 1
    part.CanCollide = false
    part.Parent = workspace
    
    local sound = Instance.new("Sound")
    sound.SoundId = "rbxassetid://" .. soundId
    sound.RollOffMaxDistance = 50
    sound.Parent = part
    sound:Play()
    
    -- ลบเมื่อเสร็จ
    game.Debris:AddItem(part, sound.TimeLength + 1)
end
```

---

## 20.4 PathfindingService

```lua
local PathfindingService = game:GetService("PathfindingService")

local function moveNPCToPosition(npc, destination)
    local humanoid = npc:FindFirstChild("Humanoid")
    local rootPart = npc:FindFirstChild("HumanoidRootPart")
    
    if not humanoid or not rootPart then return end
    
    -- สร้าง path
    local path = PathfindingService:CreatePath({
        AgentHeight = 5,
        AgentRadius = 2,
        AgentCanJump = true,
        AgentCanClimb = false,
        WaypointSpacing = 4
    })
    
    -- คำนวณ path
    local success, err = pcall(function()
        path:ComputeAsync(rootPart.Position, destination)
    end)
    
    if not success then
        print("ไม่สามารถหาเส้นทาง: " .. err)
        return
    end
    
    if path.Status == Enum.PathStatus.NoPath then
        print("ไม่มีเส้นทาง")
        return
    end
    
    -- เดินตาม waypoints
    local waypoints = path:GetWaypoints()
    
    for i, waypoint in ipairs(waypoints) do
        if waypoint.Action == Enum.PathWaypointAction.Jump then
            humanoid.Jump = true
        end
        
        humanoid:MoveTo(waypoint.Position)
        humanoid.MoveToFinished:Wait()
    end
    
    print("ถึงเป้าหมายแล้ว!")
end
```

---

## 20.5 PhysicsService

```lua
local PhysicsService = game:GetService("PhysicsService")

-- สร้าง Collision Groups
PhysicsService:RegisterCollisionGroup("Players")
PhysicsService:RegisterCollisionGroup("Enemies")
PhysicsService:RegisterCollisionGroup("Bullets")

-- กำหนดว่า groups ไหนชนกัน
PhysicsService:CollisionGroupSetCollidable("Players", "Bullets", false)
PhysicsService:CollisionGroupSetCollidable("Enemies", "Bullets", true)

-- กำหนด collision group ให้ part
local function setCollisionGroup(part, groupName)
    part.CollisionGroupId = PhysicsService:GetCollisionGroupId(groupName)
end

-- ใช้งาน: ลูกกระสุนไม่โดน character เอง
local Players = game:GetService("Players")

Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function(character)
        for _, part in ipairs(character:GetDescendants()) do
            if part:IsA("BasePart") then
                setCollisionGroup(part, "Players")
            end
        end
    end)
end)
```

---

## 20.6 CollectionService

```lua
local CollectionService = game:GetService("CollectionService")

-- Tag system: ติด tag ให้ instances
CollectionService:AddTag(workspace.MyPart, "Dangerous")
CollectionService:AddTag(workspace.MyPart, "Breakable")

-- ดู tags
local tags = CollectionService:GetTags(workspace.MyPart)
for _, tag in ipairs(tags) do
    print(tag)
end

-- หา instances ที่มี tag
local dangerous = CollectionService:GetTagged("Dangerous")
for _, part in ipairs(dangerous) do
    print("Dangerous part: " .. part.Name)
end

-- ตรวจสอบว่ามี tag หรือไม่
local hasDangerous = CollectionService:HasTag(workspace.MyPart, "Dangerous")
print("มี Dangerous tag: " .. tostring(hasDangerous))

-- ลบ tag
CollectionService:RemoveTag(workspace.MyPart, "Dangerous")

-- ตรวจสอบ instances ที่ถูก tag ใน future
CollectionService:GetInstanceAddedSignal("Coin"):Connect(function(instance)
    print("Coin ใหม่: " .. instance.Name)
    -- Setup coin behavior
end)

CollectionService:GetInstanceRemovedSignal("Coin"):Connect(function(instance)
    print("Coin ถูกลบ: " .. instance.Name)
end)
```

---

## 20.7 MarketplaceService

```lua
local MarketplaceService = game:GetService("MarketplaceService")
local Players = game:GetService("Players")

-- ตรวจสอบว่าผู้เล่นมี GamePass หรือไม่
local VIP_PASS_ID = 123456789

local function checkVIP(player)
    local success, hasPass = pcall(function()
        return MarketplaceService:UserOwnsGamePassAsync(player.UserId, VIP_PASS_ID)
    end)
    
    if success and hasPass then
        return true
    end
    return false
end

-- เปิดหน้าร้านค้า
local function promptPurchase(player, passId)
    MarketplaceService:PromptGamePassPurchase(player, passId)
end

-- ตรวจสอบเมื่อซื้อ
MarketplaceService.PromptGamePassPurchaseFinished:Connect(function(player, passId, wasPurchased)
    if wasPurchased then
        if passId == VIP_PASS_ID then
            print(player.Name .. " ซื้อ VIP Pass แล้ว!")
            -- ให้ VIP rewards
        end
    end
end)

-- Developer Products
local DOUBLE_XP_PRODUCT = 987654321

MarketplaceService.PromptProductPurchaseFinished:Connect(function(userId, productId, isPurchased)
    if isPurchased then
        local player = Players:GetPlayerByUserId(userId)
        if player then
            if productId == DOUBLE_XP_PRODUCT then
                print(player.Name .. " ซื้อ Double XP!")
                -- ให้ effect 30 นาที
            end
        end
    end
end)
```

---

## 20.8 BadgeService

```lua
local BadgeService = game:GetService("BadgeService")

-- Badge IDs
local FIRST_JOIN_BADGE = 111111111
local FIRST_KILL_BADGE = 222222222
local LEVEL_10_BADGE = 333333333

-- มอบ badge
local function awardBadge(player, badgeId)
    -- ตรวจสอบก่อนว่ามีแล้วหรือยัง
    local success, hasBadge = pcall(function()
        return BadgeService:UserHasBadgeAsync(player.UserId, badgeId)
    end)
    
    if not success or hasBadge then
        return false  -- มีแล้วหรือ error
    end
    
    -- มอบ badge
    local awardSuccess = pcall(function()
        BadgeService:AwardBadge(player.UserId, badgeId)
    end)
    
    if awardSuccess then
        print(player.Name .. " ได้รับ badge " .. badgeId)
        return true
    end
    
    return false
end

-- ใช้งาน
local Players = game:GetService("Players")

Players.PlayerAdded:Connect(function(player)
    -- First join badge
    awardBadge(player, FIRST_JOIN_BADGE)
end)
```

---

## 20.9 UserInputService

```lua
local UserInputService = game:GetService("UserInputService")

-- ตรวจสอบ input device
print("Touch: " .. tostring(UserInputService.TouchEnabled))
print("Keyboard: " .. tostring(UserInputService.KeyboardEnabled))
print("Gamepad: " .. tostring(UserInputService.GamepadEnabled))

-- ตรวจสอบ key ที่กด
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end  -- ถ้า Roblox ใช้ input นี้แล้ว
    
    if input.KeyCode == Enum.KeyCode.E then
        print("กดปุ่ม E")
    elseif input.KeyCode == Enum.KeyCode.Space then
        print("กดปุ่ม Space")
    end
    
    -- Mouse input
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        local mousePos = input.Position
        print("คลิกซ้ายที่: " .. mousePos.X .. ", " .. mousePos.Y)
    end
end)

-- Key ที่ถูกปล่อย
UserInputService.InputEnded:Connect(function(input, gameProcessed)
    if input.KeyCode == Enum.KeyCode.E then
        print("ปล่อยปุ่ม E")
    end
end)

-- ตรวจสอบว่า key กำลังถูกกดอยู่หรือไม่
local function isKeyDown(keyCode)
    return UserInputService:IsKeyDown(keyCode)
end

-- Mouse position
local function getMousePosition()
    return UserInputService:GetMouseLocation()
end

-- ซ่อน/แสดง cursor
UserInputService.MouseIconEnabled = false  -- ซ่อน cursor
UserInputService.MouseBehavior = Enum.MouseBehavior.LockCenter  -- ล็อกตรงกลาง
```

---

## 20.10 ตัวอย่างโปรเจกต์: Game Manager

```lua
-- Script: GameManager.lua
-- จัดการ services ต่างๆ ในเกม

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local GameManager = {}
GameManager.__index = GameManager

-- Game states
local GameState = {
    LOBBY = "lobby",
    STARTING = "starting",
    PLAYING = "playing",
    ENDING = "ending"
}

function GameManager.new()
    local self = setmetatable({}, GameManager)
    
    self.state = GameState.LOBBY
    self.players = {}
    self.round = 0
    self.gameTime = 300  -- 5 นาที
    self.timeLeft = 0
    self.connections = {}
    
    self:setupEvents()
    return self
end

function GameManager:setupEvents()
    -- Player join/leave
    table.insert(self.connections, 
        Players.PlayerAdded:Connect(function(player)
            self:onPlayerJoin(player)
        end)
    )
    
    table.insert(self.connections,
        Players.PlayerRemoving:Connect(function(player)
            self:onPlayerLeave(player)
        end)
    )
end

function GameManager:onPlayerJoin(player)
    self.players[player.UserId] = {
        player = player,
        kills = 0,
        deaths = 0
    }
    
    print(player.Name .. " เข้าร่วม (" .. #Players:GetPlayers() .. " คน)")
    
    if self.state == GameState.LOBBY and #Players:GetPlayers() >= 2 then
        self:startGame()
    end
end

function GameManager:onPlayerLeave(player)
    self.players[player.UserId] = nil
    print(player.Name .. " ออกจากเกม")
end

function GameManager:startGame()
    if self.state ~= GameState.LOBBY then return end
    
    self.state = GameState.STARTING
    self.round = self.round + 1
    
    print("เริ่ม Round " .. self.round)
    
    -- Countdown
    for i = 5, 1, -1 do
        -- แจ้งผู้เล่น
        local event = ReplicatedStorage:FindFirstChild("GameEvent")
        if event then
            event:FireAllClients("countdown", i)
        end
        task.wait(1)
    end
    
    self.state = GameState.PLAYING
    self.timeLeft = self.gameTime
    
    self:runGameLoop()
end

function GameManager:runGameLoop()
    local heartbeat
    heartbeat = RunService.Heartbeat:Connect(function(dt)
        if self.state ~= GameState.PLAYING then
            heartbeat:Disconnect()
            return
        end
        
        self.timeLeft = self.timeLeft - dt
        
        if self.timeLeft <= 0 then
            heartbeat:Disconnect()
            self:endGame()
        end
    end)
    
    table.insert(self.connections, heartbeat)
end

function GameManager:endGame()
    self.state = GameState.ENDING
    
    print("เกมจบ! Round " .. self.round)
    
    -- แสดงผล
    task.wait(5)
    
    -- Reset
    self.state = GameState.LOBBY
    
    print("กลับสู่ Lobby")
end

function GameManager:destroy()
    for _, conn in ipairs(self.connections) do
        conn:Disconnect()
    end
    self.connections = {}
end

-- สร้างและ start
local manager = GameManager.new()
print("Game Manager พร้อมใช้งาน")
```

---

## 20.11 แบบฝึกหัด

```lua
-- ใช้ Services ต่างๆ เพื่อ:
-- 1. สร้าง Day/Night cycle ที่ 24 วินาทีต่อวัน
-- 2. เพิ่มเสียง ambient ตามเวลา (กลางวัน/กลางคืน)
-- 3. เปลี่ยนสี ambient light ตามเวลา

local Lighting = game:GetService("Lighting")
local SoundService = game:GetService("SoundService")
local RunService = game:GetService("RunService")

-- เติมโค้ดที่นี่
```

---

## สรุป

| Service | หน้าที่ |
|---------|--------|
| Players | จัดการผู้เล่น |
| TweenService | สร้าง animations |
| RunService | Game loop, environment check |
| HttpService | JSON, HTTP requests |
| DataStoreService | บันทึกข้อมูลถาวร |
| Lighting | จัดการแสง/เวลา |
| SoundService | จัดการเสียง |
| PathfindingService | หาเส้นทาง AI |
| PhysicsService | Collision groups |
| CollectionService | Tag system |
| MarketplaceService | ร้านค้า |
| BadgeService | Badges |
| UserInputService | Input handling |

### บทถัดไป

ในบทที่ 21 เราจะเรียนเรื่อง **Players Service** อย่างละเอียด
