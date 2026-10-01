# Part 43: RemoteEvents - การสื่อสารระหว่าง Client และ Server

## บทนำ

RemoteEvents คือช่องทางการสื่อสารทางเดียวระหว่าง Client (ผู้เล่น) และ Server ในเกม Roblox เนื่องจาก Roblox ใช้สถาปัตยกรรม Client-Server การทำงานบางอย่างต้องส่งข้อมูลข้ามฝั่ง RemoteEvents ทำให้สิ่งนี้เป็นไปได้

## ทำความเข้าใจ Client-Server Architecture

```
[Client] ←──RemoteEvent──→ [Server]
  (ผู้เล่น)                  (เซิร์ฟเวอร์)
  
Client รู้:          Server รู้:
- อินพุตของตัวเอง    - ข้อมูลทุกผู้เล่น
- UI/GUI             - ตรรกะเกม
- Camera             - DataStore
- Effects            - Anti-cheat
```

### ทิศทางการส่งข้อมูล

```
Client -> Server: FireServer()
Server -> Client: FireClient() หรือ FireAllClients()
```

## การสร้าง RemoteEvent

### วิธีที่ 1: สร้างใน ReplicatedStorage (แนะนำ)

```lua
-- Studio: สร้าง Folder ใน ReplicatedStorage ชื่อ "Events"
-- แล้วสร้าง RemoteEvent ข้างในชื่อ "DamageEvent"

-- หรือสร้างด้วยโค้ดใน ServerScript:
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- สร้าง folder สำหรับเก็บ Events
local eventsFolder = Instance.new("Folder")
eventsFolder.Name = "Events"
eventsFolder.Parent = ReplicatedStorage

-- สร้าง RemoteEvent ต่างๆ
local remoteEvents = {
    "DamagePlayer",
    "HealPlayer", 
    "GiveCoins",
    "ShowNotification",
    "UpdateLeaderboard",
    "PlayerDied",
    "GameStarted",
    "GameEnded",
    "PurchaseItem",
    "UseSkill"
}

for _, eventName in ipairs(remoteEvents) do
    local event = Instance.new("RemoteEvent")
    event.Name = eventName
    event.Parent = eventsFolder
end

print("สร้าง RemoteEvents เสร็จสิ้น!")
```

### วิธีที่ 2: สร้างใน Script (ModuleScript)

```lua
-- ReplicatedStorage/NetworkEvents.lua (ModuleScript)
-- เก็บ reference ของ RemoteEvents ทั้งหมด

local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- รอให้ folder พร้อม
local function waitForEvent(path)
    return ReplicatedStorage:WaitForChild(path, 10)
end

local NetworkEvents = {
    -- Combat
    DamagePlayer = waitForEvent("Events/DamagePlayer"),
    HealPlayer = waitForEvent("Events/HealPlayer"),
    PlayerDied = waitForEvent("Events/PlayerDied"),
    
    -- Economy  
    GiveCoins = waitForEvent("Events/GiveCoins"),
    PurchaseItem = waitForEvent("Events/PurchaseItem"),
    
    -- UI
    ShowNotification = waitForEvent("Events/ShowNotification"),
    UpdateLeaderboard = waitForEvent("Events/UpdateLeaderboard"),
    
    -- Game
    GameStarted = waitForEvent("Events/GameStarted"),
    GameEnded = waitForEvent("Events/GameEnded"),
    
    -- Skills
    UseSkill = waitForEvent("Events/UseSkill")
}

return NetworkEvents
```

## Client → Server: FireServer()

### ตัวอย่าง: ผู้เล่นกดปุ่มซื้อของ

```lua
-- LocalScript (ใน StarterGui หรือ StarterPlayerScripts)
-- ผู้เล่นกดปุ่ม แจ้ง server ว่าต้องการซื้อ

local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")

local player = Players.LocalPlayer

-- รอ RemoteEvent
local purchaseEvent = ReplicatedStorage:WaitForChild("Events"):WaitForChild("PurchaseItem")

-- ฟังก์ชันกดซื้อ
local function buyItem(itemId, quantity)
    -- ส่งข้อมูลไปยัง server
    -- FireServer() จะส่ง player โดยอัตโนมัติ (ไม่ต้องส่งเอง)
    purchaseEvent:FireServer(itemId, quantity)
    
    print("ส่งคำขอซื้อ: " .. itemId .. " x" .. quantity)
end

-- เชื่อมกับปุ่มใน GUI
local shopGui = player.PlayerGui:WaitForChild("ShopGui")
local buyButton = shopGui:WaitForChild("BuyButton")

buyButton.MouseButton1Click:Connect(function()
    buyItem("Sword_001", 1)
end)
```

### ตัวอย่าง: ผู้เล่นกดยิงปืน

```lua
-- LocalScript - จัดการ input ของผู้เล่น
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

local shootEvent = ReplicatedStorage:WaitForChild("Events"):WaitForChild("ShootWeapon")

-- Cooldown ป้องกันยิงเร็วเกิน
local lastShot = 0
local SHOOT_COOLDOWN = 0.2  -- 200ms

local function shoot()
    local now = tick()
    if now - lastShot < SHOOT_COOLDOWN then return end
    lastShot = now
    
    -- คำนวณทิศทางยิง (จาก camera)
    local ray = camera:ViewportPointToRay(
        camera.ViewportSize.X / 2,
        camera.ViewportSize.Y / 2
    )
    
    local origin = ray.Origin
    local direction = ray.Direction * 300  -- ระยะ 300 studs
    
    -- ส่งไปยัง server
    shootEvent:FireServer(origin, direction)
end

-- กดคลิกซ้ายเพื่อยิง
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end  -- ไม่ยิงถ้า UI กำลังใช้งาน
    
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        shoot()
    end
end)
```

## Server รับข้อมูลจาก Client

```lua
-- Script ใน ServerScriptService
-- รับและประมวลผลการซื้อของ

local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")

local purchaseEvent = ReplicatedStorage:WaitForChild("Events"):WaitForChild("PurchaseItem")

-- ฐานข้อมูลสินค้า
local itemDatabase = {
    Sword_001 = { name = "ดาบธรรมดา", price = 100, damage = 10 },
    Sword_002 = { name = "ดาบเหล็ก", price = 500, damage = 25 },
    Shield_001 = { name = "โล่ไม้", price = 50, defense = 5 }
}

-- ===== Server รับ FireServer =====
purchaseEvent.OnServerEvent:Connect(function(player, itemId, quantity)
    -- player จะถูกส่งมาโดยอัตโนมัติเป็น parameter แรก
    
    print(string.format("[Server] %s ต้องการซื้อ %s x%d", 
        player.Name, itemId, quantity))
    
    -- ===== VALIDATION (สำคัญมาก!) =====
    -- เสมอตรวจสอบข้อมูลบน server ไม่เชื่อ client
    
    -- 1. ตรวจสอบ itemId
    local item = itemDatabase[itemId]
    if not item then
        warn("❌ itemId ไม่ถูกต้อง: " .. tostring(itemId))
        return  -- หยุดทำงาน
    end
    
    -- 2. ตรวจสอบ quantity
    if type(quantity) ~= "number" or quantity <= 0 or quantity > 100 then
        warn("❌ quantity ไม่ถูกต้อง: " .. tostring(quantity))
        return
    end
    
    -- 3. ตรวจสอบเหรียญ
    local totalCost = item.price * quantity
    local playerCoins = getPlayerCoins(player)  -- ดึงจาก DataStore/cache
    
    if playerCoins < totalCost then
        -- ส่งแจ้งเตือนกลับไปยัง client
        local notifyEvent = ReplicatedStorage:WaitForChild("Events"):WaitForChild("ShowNotification")
        notifyEvent:FireClient(player, "❌ เหรียญไม่พอ!", "error")
        return
    end
    
    -- 4. ดำเนินการซื้อ
    deductCoins(player, totalCost)
    giveItem(player, itemId, quantity)
    
    -- ส่งผลสำเร็จกลับ client
    local notifyEvent = ReplicatedStorage:WaitForChild("Events"):WaitForChild("ShowNotification")
    notifyEvent:FireClient(player, 
        string.format("✅ ซื้อ %s สำเร็จ!", item.name), 
        "success"
    )
    
    print(string.format("[Server] ✅ %s ซื้อ %s สำเร็จ", player.Name, item.name))
end)

-- ฟังก์ชันช่วย (placeholder)
function getPlayerCoins(player)
    return 1000  -- ดึงจาก DataStore จริงๆ
end

function deductCoins(player, amount)
    print("หักเหรียญ " .. amount .. " จาก " .. player.Name)
end

function giveItem(player, itemId, quantity)
    print("ให้ " .. itemId .. " x" .. quantity .. " กับ " .. player.Name)
end
```

## Server → Client: FireClient() และ FireAllClients()

```lua
-- Script ใน ServerScriptService
-- ส่งข้อมูลไปยัง client

local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")

local notifyEvent = ReplicatedStorage:WaitForChild("Events"):WaitForChild("ShowNotification")
local gameStartEvent = ReplicatedStorage:WaitForChild("Events"):WaitForChild("GameStarted")

-- ===== FireClient: ส่งถึงผู้เล่นคนเดียว =====
local function notifyPlayer(player, message, notifType)
    notifyEvent:FireClient(player, message, notifType or "info")
end

-- ===== FireAllClients: ส่งถึงทุกคน =====
local function announceToAll(message)
    gameStartEvent:FireAllClients(message)
end

-- ===== FireAllClients ยกเว้นคนหนึ่ง =====
local function announceExcept(excludedPlayer, message)
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= excludedPlayer then
            notifyEvent:FireClient(player, message, "announcement")
        end
    end
end

-- ตัวอย่าง: เริ่มเกม
local function startGame()
    print("[Server] เริ่มเกม!")
    
    -- แจ้งทุกคน
    gameStartEvent:FireAllClients({
        timeLimit = 300,  -- 5 นาที
        mode = "BattleRoyale",
        maxPlayers = Players.MaxPlayers
    })
    
    announceToAll("🎮 เกมเริ่มแล้ว! กู้ชีวิตให้นานที่สุด!")
end

-- เรียกใช้หลัง 5 วินาที
task.delay(5, startGame)
```

### Client รับจาก Server

```lua
-- LocalScript
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")

local player = Players.LocalPlayer

-- รับ notification
local notifyEvent = ReplicatedStorage:WaitForChild("Events"):WaitForChild("ShowNotification")

notifyEvent.OnClientEvent:Connect(function(message, notifType)
    print("[Client] Notification:", message, notifType)
    
    -- แสดง UI notification
    showNotificationUI(message, notifType)
end)

-- รับ game started
local gameStartEvent = ReplicatedStorage:WaitForChild("Events"):WaitForChild("GameStarted")

gameStartEvent.OnClientEvent:Connect(function(gameData)
    print("[Client] เกมเริ่ม!")
    print("  เวลา:", gameData.timeLimit .. "วินาที")
    print("  โหมด:", gameData.mode)
    
    -- เริ่ม UI countdown
    startCountdown(gameData.timeLimit)
end)

-- ฟังก์ชัน UI (placeholder)
function showNotificationUI(message, notifType)
    -- สร้าง notification บน screen
    print("📢 " .. message)
end

function startCountdown(seconds)
    -- แสดง countdown บน screen
    print("⏱️ เหลือ " .. seconds .. " วินาที")
end
```

## ระบบ RemoteEvent ที่สมบูรณ์

### การจัดระเบียบ Events

```lua
-- ServerScriptService/EventManager.lua
-- ระบบจัดการ RemoteEvents ที่เป็นระเบียบ

local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")

-- สร้าง event structure
local EventManager = {}

-- สร้าง events ทั้งหมดตอน server start
local eventDefinitions = {
    -- Format: { name, description }
    
    -- Combat Events
    { "Combat_DealDamage", "ผู้เล่นโจมตี" },
    { "Combat_HealTarget", "รักษา target" },
    { "Combat_PlayerDied", "ผู้เล่นตาย" },
    { "Combat_UseSkill", "ใช้ skill" },
    
    -- Economy Events
    { "Economy_Purchase", "ซื้อสินค้า" },
    { "Economy_Trade", "แลกเปลี่ยน" },
    { "Economy_DropCoin", "วาง coin" },
    
    -- UI Events
    { "UI_Notification", "แสดง notification" },
    { "UI_UpdateHUD", "อัพเดท HUD" },
    { "UI_OpenMenu", "เปิดเมนู" },
    
    -- World Events
    { "World_PlaceObject", "วางวัตถุ" },
    { "World_InteractNPC", "คุยกับ NPC" },
    { "World_CollectItem", "เก็บของ" }
}

-- สร้าง folders
local eventsFolder = ReplicatedStorage:FindFirstChild("Events")
if not eventsFolder then
    eventsFolder = Instance.new("Folder")
    eventsFolder.Name = "Events"
    eventsFolder.Parent = ReplicatedStorage
end

-- สร้าง RemoteEvents
for _, def in ipairs(eventDefinitions) do
    local name, description = def[1], def[2]
    
    if not eventsFolder:FindFirstChild(name) then
        local event = Instance.new("RemoteEvent")
        event.Name = name
        event.Parent = eventsFolder
    end
end

-- Getter function
function EventManager.getEvent(name)
    return eventsFolder:WaitForChild(name, 5)
end

-- Helper: fire to player with validation
function EventManager.fireClient(eventName, player, ...)
    local event = EventManager.getEvent(eventName)
    if event and player and player.Parent then
        event:FireClient(player, ...)
    end
end

-- Helper: fire to all players
function EventManager.fireAll(eventName, ...)
    local event = EventManager.getEvent(eventName)
    if event then
        event:FireAllClients(...)
    end
end

-- Helper: connect server handler
function EventManager.onServerEvent(eventName, handler)
    local event = EventManager.getEvent(eventName)
    if event then
        event.OnServerEvent:Connect(function(player, ...)
            -- Log ทุก event
            -- print(string.format("[Event] %s from %s", eventName, player.Name))
            handler(player, ...)
        end)
    end
end

-- ===== Register Handlers =====

-- Combat: Deal Damage
EventManager.onServerEvent("Combat_DealDamage", function(player, targetId, damage, weaponId)
    -- Validation
    if type(damage) ~= "number" or damage <= 0 or damage > 9999 then
        warn("Invalid damage: " .. tostring(damage))
        return
    end
    
    -- หา target
    local target = Players:GetPlayerByUserId(targetId)
    if not target then
        -- อาจเป็น NPC
        -- handleNPCDamage(targetId, damage, player)
        return
    end
    
    -- ใช้ damage บน target
    local humanoid = target.Character and target.Character:FindFirstChild("Humanoid")
    if humanoid then
        humanoid:TakeDamage(damage)
        print(string.format("[Combat] %s โจมตี %s: %d damage", 
            player.Name, target.Name, damage))
    end
end)

-- UI: Notification (ไม่จำเป็น เพราะ server ส่งให้ client ไม่ใช่ client ส่งตัวเอง)
-- แต่ถ้า client ต้องการแสดง UI ที่ไม่เกี่ยวกับ server state ก็ทำได้

-- Economy: Purchase
EventManager.onServerEvent("Economy_Purchase", function(player, itemId, quantity)
    print(string.format("[Economy] %s ซื้อ %s x%d", player.Name, itemId, quantity))
    -- ประมวลผลการซื้อ...
end)

return EventManager
```

## Anti-Cheat ด้วย RemoteEvent

```lua
-- Server-side validation สำหรับ RemoteEvents
-- สำคัญมาก! อย่าเชื่อ client โดยไม่ตรวจสอบ

local ServerStorage = game:GetService("ServerStorage")
local Players = game:GetService("Players")

-- ติดตาม rate ของแต่ละผู้เล่น
local playerRates = {}

local function checkRateLimit(player, eventName, maxPerSecond)
    local userId = player.UserId
    
    if not playerRates[userId] then
        playerRates[userId] = {}
    end
    
    if not playerRates[userId][eventName] then
        playerRates[userId][eventName] = {
            count = 0,
            resetTime = os.time() + 1
        }
    end
    
    local rateData = playerRates[userId][eventName]
    
    -- Reset counter ทุกวินาที
    if os.time() >= rateData.resetTime then
        rateData.count = 0
        rateData.resetTime = os.time() + 1
    end
    
    rateData.count = rateData.count + 1
    
    if rateData.count > maxPerSecond then
        warn(string.format("⚠️ Rate limit exceeded: %s sent %s %d times/sec!", 
            player.Name, eventName, rateData.count))
        return false
    end
    
    return true
end

-- ล้างข้อมูลเมื่อผู้เล่นออก
Players.PlayerRemoving:Connect(function(player)
    playerRates[player.UserId] = nil
end)

-- ตัวอย่างการใช้ใน event handler
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local shootEvent = ReplicatedStorage:WaitForChild("Events"):WaitForChild("Combat_Shoot")

local MAX_SHOOT_PER_SECOND = 10  -- ยิงได้สูงสุด 10 ครั้ง/วินาที

shootEvent.OnServerEvent:Connect(function(player, origin, direction)
    -- Rate limit check
    if not checkRateLimit(player, "Shoot", MAX_SHOOT_PER_SECOND) then
        return  -- Ignore - อาจเป็น cheat
    end
    
    -- Type checking
    if typeof(origin) ~= "Vector3" or typeof(direction) ~= "Vector3" then
        warn("Invalid types from " .. player.Name)
        return
    end
    
    -- Distance check (ป้องกันยิงจากไกลเกิน)
    local character = player.Character
    if not character then return end
    
    local rootPart = character:FindFirstChild("HumanoidRootPart")
    if not rootPart then return end
    
    local distance = (origin - rootPart.Position).Magnitude
    if distance > 20 then  -- ถ้าจุดยิงไกลจากตัวผู้เล่นเกิน 20 studs
        warn(string.format("⚠️ Suspicious shot from %s (distance: %.1f)", 
            player.Name, distance))
        return
    end
    
    -- Direction check (ตรวจว่าทิศทางสมเหตุสมผล)
    if direction.Magnitude > 1000 then
        warn("Invalid direction magnitude: " .. direction.Magnitude)
        return
    end
    
    -- ประมวลผลการยิง...
    processBullet(player, origin, direction)
end)

function processBullet(player, origin, direction)
    -- Raycast เพื่อหาสิ่งที่ถูกยิง
    local raycastParams = RaycastParams.new()
    raycastParams.FilterDescendantsInstances = {player.Character}
    raycastParams.FilterType = Enum.RaycastFilterType.Exclude
    
    local result = workspace:Raycast(origin, direction, raycastParams)
    
    if result then
        local hit = result.Instance
        local humanoid = hit.Parent:FindFirstChild("Humanoid") or 
                         hit.Parent.Parent:FindFirstChild("Humanoid")
        
        if humanoid then
            -- ตรวจสอบว่าอยู่ในระยะ
            local hitDistance = (result.Position - origin).Magnitude
            if hitDistance <= 350 then  -- ระยะยิงสูงสุด
                humanoid:TakeDamage(20)
                print(player.Name .. " โจมตี " .. hit.Parent.Name)
            end
        end
    end
end
```

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: ระบบ Chat สี
สร้างระบบ chat ที่ผู้เล่น VIP สามารถส่งข้อความสีทองได้

```lua
-- LocalScript
local chatEvent = ReplicatedStorage:WaitForChild("Events"):WaitForChild("SendChatMessage")

-- ส่งข้อความ
chatEvent:FireServer("สวัสดีทุกคน!", "gold")

-- ServerScript
chatEvent.OnServerEvent:Connect(function(player, message, color)
    -- ตรวจสอบว่าเป็น VIP
    local isVIP = checkVIPStatus(player)
    
    if color == "gold" and not isVIP then
        color = "white"  -- ไม่อนุญาต non-VIP ใช้สีทอง
    end
    
    -- ส่งไปทุกคน
    local broadcastEvent = ReplicatedStorage:WaitForChild("Events"):WaitForChild("ReceiveChatMessage")
    broadcastEvent:FireAllClients(player.Name, message, color)
end)
```

### แบบฝึกหัดที่ 2: ระบบ Vote
สร้างระบบลงคะแนนเสียงสำหรับ map ถัดไป

### แบบฝึกหัดที่ 3: ระบบ Trade
สร้างระบบแลกเปลี่ยนของระหว่างผู้เล่น

## เคล็ดลับจากมืออาชีพ

```lua
-- 1. ใช้ table เป็น parameter แทนหลาย parameter แยก
-- ❌ ยาก
event:FireServer(1, "sword", 3, true, 100)

-- ✅ อ่านง่าย
event:FireServer({
    itemType = 1,
    itemName = "sword",
    quantity = 3,
    isEquipped = true,
    price = 100
})

-- 2. เสมอ debounce events จาก client
local debounces = {}
event.OnServerEvent:Connect(function(player, ...)
    if debounces[player.UserId] then return end
    debounces[player.UserId] = true
    task.delay(0.5, function()
        debounces[player.UserId] = nil
    end)
    
    -- ประมวลผล...
end)

-- 3. ใช้ pcall ใน event handler
event.OnServerEvent:Connect(function(player, ...)
    local ok, err = pcall(function()
        -- โค้ดที่อาจ error
    end)
    if not ok then
        warn("Error in event handler: " .. err)
    end
end)
```

## สรุป

RemoteEvents เป็นหัวใจของการสื่อสาร Client-Server ใน Roblox:

- **Client → Server**: ใช้ `FireServer()` รับด้วย `OnServerEvent`
- **Server → Client**: ใช้ `FireClient()` หรือ `FireAllClients()` รับด้วย `OnClientEvent`
- **ตรวจสอบทุกอย่างบน Server** - อย่าเชื่อ Client
- **Rate Limiting** - ป้องกัน spam
- **Type Validation** - ป้องกัน exploit
- **จัดระเบียบ Events** - ทำให้โค้ดดูแลง่าย

ในบทถัดไปเราจะเรียนรู้ RemoteFunctions ที่ตอบกลับได้!
