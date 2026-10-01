# ตอนที่ 25: LocalScripts vs Scripts ใน Roblox

## บทนำ

หนึ่งในสิ่งที่สำคัญที่สุดในการพัฒนาเกม Roblox คือการเข้าใจความแตกต่างระหว่าง **Scripts** (Server Scripts) และ **LocalScripts** (Client Scripts) การเลือกใช้ประเภทที่ถูกต้องจะทำให้เกมทำงานถูกต้อง ปลอดภัย และมีประสิทธิภาพ

---

## 25.1 ภาพรวม

```
┌─────────────────────────────────────────────────────────┐
│                    Roblox Architecture                    │
├───────────────────────┬─────────────────────────────────┤
│       SERVER          │           CLIENT(s)              │
│                       │                                   │
│  Script (Server)      │  LocalScript (Client)            │
│  - ทำงานบน Server     │  - ทำงานบน Client ของผู้เล่น     │
│  - มองเห็นจาก Client  │  - Client แต่ละคนรัน copy ของตน  │
│  - จัดการ game logic  │  - จัดการ UI, input, effects      │
│  - DataStore          │  - ไม่สามารถเข้า DataStore ได้    │
│  - Trustedข้อมูล      │  - ไม่ Trusted (ถูก exploit ได้)  │
└───────────────────────┴─────────────────────────────────┘
```

---

## 25.2 Scripts (Server Scripts)

### ที่อยู่ที่วางได้
- ServerScriptService (แนะนำ)
- Workspace (ไม่แนะนำแต่ใช้ได้)

### ลักษณะเด่น

```lua
-- Script ทำงานบน Server เท่านั้น
-- ทดสอบว่าเราอยู่ที่ไหน
local RunService = game:GetService("RunService")
print(RunService:IsServer())  -- true
print(RunService:IsClient())  -- false

-- Server มี authority เหนือ:
-- 1. Players object
-- 2. Character behavior
-- 3. DataStore
-- 4. ทุก Instance ใน game

-- ตัวอย่าง Server Script
local Players = game:GetService("Players")
local DataStoreService = game:GetService("DataStoreService")

-- สามารถเข้าถึง DataStore ได้
local store = DataStoreService:GetDataStore("PlayerData")

Players.PlayerAdded:Connect(function(player)
    -- สร้าง leaderstats (มองเห็นได้จาก client)
    local leaderstats = Instance.new("Folder")
    leaderstats.Name = "leaderstats"
    leaderstats.Parent = player
    
    local gold = Instance.new("IntValue")
    gold.Name = "Gold"
    gold.Value = 0
    gold.Parent = leaderstats
    
    -- โหลดข้อมูลจาก DataStore
    local key = "player_" .. player.UserId
    local success, data = pcall(function()
        return store:GetAsync(key)
    end)
    
    if success and data then
        gold.Value = data.gold or 0
    end
end)
```

---

## 25.3 LocalScripts (Client Scripts)

### ที่อยู่ที่วางได้
- StarterPlayerScripts (ใน StarterPlayer) - แนะนำ
- StarterCharacterScripts - รันทุกครั้งที่ character spawn
- StarterGui - ใช้กับ ScreenGui
- PlayerGui - GUI ของผู้เล่น

### ลักษณะเด่น

```lua
-- LocalScript ทำงานบน Client เท่านั้น
local RunService = game:GetService("RunService")
print(RunService:IsServer())  -- false
print(RunService:IsClient())  -- true

-- มีสิทธิ์เข้าถึง LocalPlayer
local Players = game:GetService("Players")
local player = Players.LocalPlayer  -- ใช้ได้เฉพาะ LocalScript

-- UserInputService
local UserInputService = game:GetService("UserInputService")

-- Camera
local camera = workspace.CurrentCamera

-- ตัวอย่าง LocalScript - ระบบ Input
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    if input.KeyCode == Enum.KeyCode.E then
        print("ผู้เล่นกด E")
        -- ส่งไป Server
        local event = game.ReplicatedStorage.ActionEvent
        event:FireServer("interact")
    end
end)
```

---

## 25.4 ตารางเปรียบเทียบ

| Feature | Script (Server) | LocalScript (Client) |
|---------|----------------|---------------------|
| ทำงานที่ | Server | Client |
| Players.LocalPlayer | nil | มีค่า |
| DataStoreService | ใช้ได้ | ใช้ไม่ได้ |
| UserInputService | ใช้ได้บางส่วน | ใช้ได้เต็มที่ |
| Camera control | จำกัด | เต็มที่ |
| RemoteEvent:Fire | FireClient/All | FireServer |
| Workspace access | เต็มที่ | เต็มที่ |
| Script security | สูง (ไม่เปิดเผย) | ต่ำ (ถูก exploit ได้) |
| Game logic | ควรอยู่ที่นี่ | อย่าใส่ logic สำคัญ |
| จำนวน instance | 1 | เท่าจำนวนผู้เล่น |

---

## 25.5 ModuleScript

ModuleScript ไม่ได้ทำงานเองแต่ถูก require() โดย Scripts อื่น:

```lua
-- ModuleScript: MathUtils
local MathUtils = {}

function MathUtils.clamp(value, min, max)
    return math.max(min, math.min(max, value))
end

function MathUtils.lerp(a, b, t)
    return a + (b - a) * t
end

return MathUtils

-- ใช้จาก Script:
local MathUtils = require(game.ReplicatedStorage.MathUtils)
print(MathUtils.clamp(5, 0, 10))  -- 5
```

---

## 25.6 Client-Server Communication Pattern

```lua
-- ════════════════════════════════
-- Pattern 1: Client -> Server (Action)
-- ════════════════════════════════

-- SERVER (Script ใน ServerScriptService)
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local attackEvent = Instance.new("RemoteEvent")
attackEvent.Name = "AttackEvent"
attackEvent.Parent = ReplicatedStorage

attackEvent.OnServerEvent:Connect(function(player, targetId, weaponId)
    -- VALIDATE ทุกอย่างเสมอ!
    if not player.Character then return end
    
    local target = game.Players:GetPlayerByUserId(targetId)
    if not target then return end
    if not target.Character then return end
    
    -- ตรวจสอบระยะ
    local attPos = player.Character.HumanoidRootPart.Position
    local defPos = target.Character.HumanoidRootPart.Position
    local distance = (attPos - defPos).Magnitude
    
    if distance > 20 then  -- max range
        print("Cheat detected: " .. player.Name .. " attacked from distance " .. distance)
        return
    end
    
    -- Apply damage
    local humanoid = target.Character:FindFirstChild("Humanoid")
    if humanoid then
        humanoid:TakeDamage(15)
    end
end)

-- ════════════════════════════════
-- CLIENT (LocalScript ใน StarterPlayerScripts)
-- ════════════════════════════════

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local player = Players.LocalPlayer
local attackEvent = ReplicatedStorage:WaitForChild("AttackEvent")

-- ตรวจสอบ target ด้วย Raycast
local function getAttackTarget()
    local camera = workspace.CurrentCamera
    local unitRay = camera:ViewportPointToRay(
        camera.ViewportSize.X / 2,
        camera.ViewportSize.Y / 2
    )
    
    local params = RaycastParams.new()
    params.FilterType = Enum.RaycastFilterType.Exclude
    params.FilterDescendantsInstances = {player.Character}
    
    local result = workspace:Raycast(unitRay.Origin, unitRay.Direction * 20, params)
    
    if result then
        local hitCharacter = result.Instance.Parent
        return game.Players:GetPlayerFromCharacter(hitCharacter)
    end
    
    return nil
end

-- ส่งการโจมตีไป Server
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        local target = getAttackTarget()
        if target then
            attackEvent:FireServer(target.UserId, "sword")
        end
    end
end)
```

---

## 25.7 GUI System (LocalScript)

```lua
-- LocalScript: UIManager.lua (ใน StarterGui หรือ StarterPlayerScripts)

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local playerGui = player.PlayerGui

-- Health Bar
local function createHealthBar()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "HUD"
    screenGui.ResetOnSpawn = false
    
    local healthFrame = Instance.new("Frame")
    healthFrame.Name = "HealthBar"
    healthFrame.Size = UDim2.new(0, 200, 0, 25)
    healthFrame.Position = UDim2.new(0, 10, 1, -40)
    healthFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    healthFrame.BorderSizePixel = 0
    healthFrame.Parent = screenGui
    
    local UICorner = Instance.new("UICorner")
    UICorner.CornerRadius = UDim.new(0, 4)
    UICorner.Parent = healthFrame
    
    local healthFill = Instance.new("Frame")
    healthFill.Name = "Fill"
    healthFill.Size = UDim2.new(1, 0, 1, 0)
    healthFill.BackgroundColor3 = Color3.fromRGB(50, 200, 50)
    healthFill.BorderSizePixel = 0
    healthFill.Parent = healthFrame
    
    local UICorner2 = Instance.new("UICorner")
    UICorner2.CornerRadius = UDim.new(0, 4)
    UICorner2.Parent = healthFill
    
    local healthText = Instance.new("TextLabel")
    healthText.Name = "Text"
    healthText.Size = UDim2.new(1, 0, 1, 0)
    healthText.BackgroundTransparency = 1
    healthText.TextColor3 = Color3.new(1, 1, 1)
    healthText.Text = "100/100"
    healthText.Font = Enum.Font.GothamBold
    healthText.TextSize = 14
    healthText.Parent = healthFrame
    
    screenGui.Parent = playerGui
    
    return {
        frame = healthFill,
        text = healthText
    }
end

-- อัปเดต UI ตาม character
local healthBar = createHealthBar()

local function updateHealthUI(current, max)
    local percentage = current / max
    
    TweenService:Create(healthBar.frame, TweenInfo.new(0.2), {
        Size = UDim2.new(percentage, 0, 1, 0)
    }):Play()
    
    healthBar.text.Text = math.floor(current) .. "/" .. max
    
    -- เปลี่ยนสีตามเลือด
    local color
    if percentage > 0.6 then
        color = Color3.fromRGB(50, 200, 50)   -- เขียว
    elseif percentage > 0.3 then
        color = Color3.fromRGB(255, 200, 0)   -- เหลือง
    else
        color = Color3.fromRGB(200, 50, 50)   -- แดง
    end
    
    TweenService:Create(healthBar.frame, TweenInfo.new(0.2), {
        BackgroundColor3 = color
    }):Play()
end

-- ติดตาม Health ของ character
player.CharacterAdded:Connect(function(character)
    local humanoid = character:WaitForChild("Humanoid")
    
    -- อัปเดต UI ทันที
    updateHealthUI(humanoid.Health, humanoid.MaxHealth)
    
    -- ติดตาม health changes
    humanoid.HealthChanged:Connect(function(health)
        updateHealthUI(health, humanoid.MaxHealth)
    end)
end)

-- ถ้ามี character แล้ว
if player.Character then
    local humanoid = player.Character:FindFirstChild("Humanoid")
    if humanoid then
        updateHealthUI(humanoid.Health, humanoid.MaxHealth)
    end
end
```

---

## 25.8 ตัวอย่างโปรเจกต์: Chat System

```lua
-- ════════════════════════════
-- SERVER (Script)
-- ════════════════════════════
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- สร้าง Remote Events
local eventsFolder = Instance.new("Folder")
eventsFolder.Name = "ChatEvents"
eventsFolder.Parent = ReplicatedStorage

local sendMessageEvent = Instance.new("RemoteEvent")
sendMessageEvent.Name = "SendMessage"
sendMessageEvent.Parent = eventsFolder

local receiveMessageEvent = Instance.new("RemoteEvent")
receiveMessageEvent.Name = "ReceiveMessage"
receiveMessageEvent.Parent = eventsFolder

-- Message history
local chatHistory = {}
local MAX_HISTORY = 50

-- กรองข้อความ (ง่ายๆ)
local function filterMessage(message)
    local banned = {"badword1", "badword2"}
    local filtered = message
    for _, word in ipairs(banned) do
        filtered = filtered:gsub(word, string.rep("*", #word))
    end
    return filtered
end

-- รับข้อความจาก Client
sendMessageEvent.OnServerEvent:Connect(function(player, message)
    -- Validate
    if type(message) ~= "string" then return end
    if #message == 0 or #message > 200 then return end
    
    -- Filter
    local filtered = filterMessage(message)
    
    -- สร้าง message data
    local msgData = {
        sender = player.Name,
        message = filtered,
        timestamp = os.time(),
        team = player.Team and player.Team.Name or nil
    }
    
    -- เพิ่มใน history
    table.insert(chatHistory, msgData)
    if #chatHistory > MAX_HISTORY then
        table.remove(chatHistory, 1)
    end
    
    -- ส่งไปยัง clients ทั้งหมด
    receiveMessageEvent:FireAllClients(msgData)
end)

-- ════════════════════════════
-- CLIENT (LocalScript)
-- ════════════════════════════
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local playerGui = player.PlayerGui

-- รอ Events
local chatEvents = ReplicatedStorage:WaitForChild("ChatEvents")
local sendEvent = chatEvents:WaitForChild("SendMessage")
local receiveEvent = chatEvents:WaitForChild("ReceiveMessage")

-- สร้าง Chat UI
local function createChatUI()
    local gui = Instance.new("ScreenGui")
    gui.Name = "ChatGui"
    gui.ResetOnSpawn = false
    
    -- Chat container
    local container = Instance.new("Frame")
    container.Name = "Container"
    container.Size = UDim2.new(0, 400, 0, 300)
    container.Position = UDim2.new(0, 10, 1, -350)
    container.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    container.BackgroundTransparency = 0.5
    container.BorderSizePixel = 0
    container.Parent = gui
    
    -- Messages scroll frame
    local scrollFrame = Instance.new("ScrollingFrame")
    scrollFrame.Name = "Messages"
    scrollFrame.Size = UDim2.new(1, 0, 1, -40)
    scrollFrame.BackgroundTransparency = 1
    scrollFrame.BorderSizePixel = 0
    scrollFrame.ScrollBarThickness = 4
    scrollFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
    scrollFrame.AutomaticCanvasSize = Enum.AutomaticSize.Y
    scrollFrame.Parent = container
    
    local listLayout = Instance.new("UIListLayout")
    listLayout.FillDirection = Enum.FillDirection.Vertical
    listLayout.SortOrder = Enum.SortOrder.LayoutOrder
    listLayout.Parent = scrollFrame
    
    -- Input box
    local inputBox = Instance.new("TextBox")
    inputBox.Name = "Input"
    inputBox.Size = UDim2.new(1, -60, 0, 35)
    inputBox.Position = UDim2.new(0, 0, 1, -35)
    inputBox.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    inputBox.BorderSizePixel = 0
    inputBox.TextColor3 = Color3.new(1, 1, 1)
    inputBox.PlaceholderText = "พิมพ์ข้อความ..."
    inputBox.Font = Enum.Font.Gotham
    inputBox.TextSize = 14
    inputBox.ClearTextOnFocus = false
    inputBox.Parent = container
    
    -- Send button
    local sendButton = Instance.new("TextButton")
    sendButton.Name = "Send"
    sendButton.Size = UDim2.new(0, 55, 0, 35)
    sendButton.Position = UDim2.new(1, -55, 1, -35)
    sendButton.BackgroundColor3 = Color3.fromRGB(0, 120, 200)
    sendButton.BorderSizePixel = 0
    sendButton.TextColor3 = Color3.new(1, 1, 1)
    sendButton.Text = "ส่ง"
    sendButton.Font = Enum.Font.GothamBold
    sendButton.TextSize = 14
    sendButton.Parent = container
    
    gui.Parent = playerGui
    
    return {
        scrollFrame = scrollFrame,
        inputBox = inputBox,
        sendButton = sendButton
    }
end

local chatUI = createChatUI()

-- เพิ่มข้อความใน UI
local messageCount = 0
local function addMessage(sender, message, teamName)
    messageCount = messageCount + 1
    
    local label = Instance.new("TextLabel")
    label.Name = "Msg" .. messageCount
    label.Size = UDim2.new(1, -10, 0, 0)
    label.AutomaticSize = Enum.AutomaticSize.Y
    label.BackgroundTransparency = 1
    label.TextColor3 = Color3.new(1, 1, 1)
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Font = Enum.Font.Gotham
    label.TextSize = 13
    label.TextWrapped = true
    label.LayoutOrder = messageCount
    
    local teamPrefix = teamName and "[" .. teamName .. "] " or ""
    label.Text = teamPrefix .. sender .. ": " .. message
    label.Parent = chatUI.scrollFrame
    
    -- Scroll ลงไปล่างสุด
    task.wait()  -- รอให้ layout อัปเดต
    chatUI.scrollFrame.CanvasPosition = Vector2.new(
        0, 
        chatUI.scrollFrame.AbsoluteCanvasSize.Y
    )
end

-- ส่งข้อความ
local function sendMessage()
    local message = chatUI.inputBox.Text
    if message == "" then return end
    
    sendEvent:FireServer(message)
    chatUI.inputBox.Text = ""
end

-- Events
chatUI.sendButton.MouseButton1Click:Connect(sendMessage)

chatUI.inputBox.FocusLost:Connect(function(enterPressed)
    if enterPressed then
        sendMessage()
    end
end)

-- รับข้อความจาก Server
receiveEvent.OnClientEvent:Connect(function(msgData)
    addMessage(msgData.sender, msgData.message, msgData.team)
end)
```

---

## 25.9 ข้อผิดพลาดที่พบบ่อย

### 1. ใส่ LocalScript ผิดที่

```lua
-- ผิด: LocalScript ใน ServerScriptService
-- LocalScript จะไม่ทำงาน!

-- ถูก: LocalScript ใน StarterPlayerScripts
-- หรือ StarterGui
```

### 2. เรียก Players.LocalPlayer จาก Server

```lua
-- ผิด: จาก Server Script
local Players = game:GetService("Players")
local player = Players.LocalPlayer  -- nil! ใช้ไม่ได้บน Server

-- ถูก: จาก LocalScript
local player = Players.LocalPlayer  -- ใช้ได้

-- หรือบน Server ให้รับจาก parameter
Players.PlayerAdded:Connect(function(player)
    -- player คือ Player object ที่เข้าร่วม
end)
```

### 3. ทำ Game Logic บน Client

```lua
-- ผิด: ตัดสินใจบน Client (ถูก exploit ได้)
-- LocalScript:
if isPlayerDead() then
    respawnPlayer()  -- Client ตัดสินใจเองว่าจะ respawn
end

-- ถูก: Server ตัดสินใจ
-- Server Script:
humanoid.Died:Connect(function()
    task.wait(5)
    player:LoadCharacter()  -- Server สั่ง respawn
end)
```

### 4. ลืม WaitForChild ใน Client

```lua
-- ผิด: Client อาจโหลดก่อน Server สร้าง object
local event = game.ReplicatedStorage.MyEvent  -- ERROR! อาจยังไม่มี

-- ถูก: รอจนกว่าจะมี
local event = game.ReplicatedStorage:WaitForChild("MyEvent")
```

---

## 25.10 Summary: เลือกใช้อะไรที่ไหน?

```
ต้องการทำอะไร?
│
├── จัดการผู้เล่น, DataStore, Game Logic
│   └── Script ใน ServerScriptService
│
├── UI, Input, Camera, Effects
│   └── LocalScript ใน StarterPlayerScripts หรือ StarterGui
│
├── Character-specific logic (respawn ทุกครั้ง)
│   └── LocalScript ใน StarterCharacterScripts
│
├── โค้ดที่ใช้ร่วมกัน
│   └── ModuleScript ใน ReplicatedStorage
│
├── โค้ด Server-only
│   └── ModuleScript ใน ServerScriptService
│
└── ส่งข้อมูล Server <-> Client
    └── RemoteEvent/RemoteFunction ใน ReplicatedStorage
```

---

## 25.11 แบบฝึกหัด

### แบบฝึกหัดที่ 1: Complete Feature

```lua
-- สร้าง feature ครบวงจร: ระบบ Shop
-- 1. GUI แสดงสินค้า (LocalScript)
-- 2. ปุ่มซื้อ ส่ง request ไป Server (LocalScript)
-- 3. Server validate และทำ transaction (Script)
-- 4. Server แจ้งผล (Script -> RemoteEvent -> LocalScript)
-- 5. LocalScript อัปเดต UI แสดงผล

-- เติมโค้ดทั้ง 3 ส่วน
```

### แบบฝึกหัดที่ 2: Debug Practice

```lua
-- หาข้อผิดพลาดในโค้ดต่อไปนี้:

-- Script ที่ 1 (อ้างว่าเป็น Server Script):
local player = game.Players.LocalPlayer  -- บรรทัด 1
local char = player.Character             -- บรรทัด 2
char.Humanoid.Health = 0                  -- บรรทัด 3

-- Script ที่ 2 (LocalScript ใน StarterGui):
local DataStore = game:GetService("DataStoreService")  -- บรรทัด 1
local store = DataStore:GetDataStore("Data")           -- บรรทัด 2
local data = store:GetAsync("key")                      -- บรรทัด 3
```

---

## สรุปบทเรียน

| ประเภท | ที่วาง | ใช้สำหรับ |
|--------|--------|----------|
| Script | ServerScriptService | Game logic, DataStore, validation |
| LocalScript | StarterPlayerScripts, StarterGui | UI, Input, Camera, Effects |
| ModuleScript | ReplicatedStorage (shared), ServerScriptService (server-only) | Reusable code |

### หลักการสำคัญ 5 ข้อ

1. **Never trust the client** - validate ทุก request บน Server เสมอ
2. **Server = Authority** - Game logic สำคัญต้องอยู่บน Server
3. **Client = Presentation** - UI, Input, Effects อยู่บน Client
4. **RemoteEvents สำหรับสื่อสาร** - ใช้ RemoteEvent/Function ผ่าน ReplicatedStorage
5. **WaitForChild เสมอ** - Client ต้องรอ Server สร้าง objects ก่อน

### จบซีรีส์บทที่ 10-25

ยินดีด้วย! คุณได้เรียนรู้พื้นฐาน Lua programming และ Roblox development ครบถ้วนแล้ว ตั้งแต่ตัวแปร, ตัวดำเนินการ, โครงสร้างควบคุม, ฟังก์ชัน, tables, strings, math, events, instances, hierarchy จนถึง services ต่างๆ

ในบทถัดไปเราจะเริ่มสร้างเกมจริงๆ โดยนำความรู้ทั้งหมดมาประยุกต์ใช้!
