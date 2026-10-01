# Part 69: Social Features and Roleplay Mechanics

## บทนำ

Social Game และ Roleplay Game เป็นประเภทที่เน้นการมีปฏิสัมพันธ์ระหว่างผู้เล่น ตัวอย่างเช่น Brookhaven, Bloxburg, Welcome to Bloxburg ในบทนี้เราจะสร้าง Social Game ที่มีระบบครบครัน ตั้งแต่การซื้อบ้าน ทำงาน ไปจนถึงระบบความสัมพันธ์

---

## 69.1 โครงสร้าง Social Game

```
ServerScriptService
├── SocialCore (Script)
├── JobSystem (Script)
├── HouseSystem (Script)
├── RelationshipSystem (Script)
└── ChatSystem (Script)

ReplicatedStorage
├── Remotes
│   ├── GetJob (RemoteEvent)
│   ├── BuyHouse (RemoteEvent)
│   ├── SendFriendRequest (RemoteEvent)
│   └── EmotePlay (RemoteEvent)
└── Modules
    ├── JobData (ModuleScript)
    ├── HouseData (ModuleScript)
    └── EmoteData (ModuleScript)

StarterGui
├── SocialHUD (ScreenGui)
├── PhoneUI (ScreenGui)
└── InteractionPrompt (ScreenGui)
```

---

## 69.2 ระบบ Character Customization

```lua
-- ServerScriptService/CharacterCustomization.lua
-- ระบบแต่งตัว

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local Remotes = ReplicatedStorage:WaitForChild("Remotes")
local customizeRemote = Instance.new("RemoteEvent")
customizeRemote.Name = "CustomizeCharacter"
customizeRemote.Parent = Remotes

-- ข้อมูลของผู้เล่น
local playerCustomization = {}

-- ตั้งค่า default
Players.PlayerAdded:Connect(function(player)
    playerCustomization[player.UserId] = {
        bodyColors = {
            head = Color3.fromRGB(255, 220, 180),
            torso = Color3.fromRGB(200, 160, 100),
            leftArm = Color3.fromRGB(255, 220, 180),
            rightArm = Color3.fromRGB(255, 220, 180),
            leftLeg = Color3.fromRGB(200, 160, 100),
            rightLeg = Color3.fromRGB(200, 160, 100),
        },
        clothes = {
            shirt = nil,
            pants = nil,
        },
        accessories = {},
        walkStyle = "default",
        gender = "neutral"
    }
end)

-- Apply customization ให้ character
local function applyCustomization(player)
    local character = player.Character
    if not character then return end
    
    local data = playerCustomization[player.UserId]
    if not data then return end
    
    -- เปลี่ยนสีผิว
    local humanoid = character:FindFirstChildWhichIsA("Humanoid")
    if humanoid and humanoid.RigType == Enum.HumanoidRigType.R6 then
        local bodyColors = character:FindFirstChild("Body Colors")
        if bodyColors and data.bodyColors then
            bodyColors.HeadColor3 = data.bodyColors.head
            bodyColors.TorsoColor3 = data.bodyColors.torso
            bodyColors.LeftArmColor3 = data.bodyColors.leftArm
            bodyColors.RightArmColor3 = data.bodyColors.rightArm
            bodyColors.LeftLegColor3 = data.bodyColors.leftLeg
            bodyColors.RightLegColor3 = data.bodyColors.rightLeg
        end
    end
    
    -- ใส่เสื้อผ้า
    if data.clothes.shirt then
        local existingShirt = character:FindFirstChildWhichIsA("Shirt")
        if existingShirt then existingShirt:Destroy() end
        
        local shirt = Instance.new("Shirt")
        shirt.ShirtTemplate = data.clothes.shirt
        shirt.Parent = character
    end
    
    if data.clothes.pants then
        local existingPants = character:FindFirstChildWhichIsA("Pants")
        if existingPants then existingPants:Destroy() end
        
        local pants = Instance.new("Pants")
        pants.PantsTemplate = data.clothes.pants
        pants.Parent = character
    end
end

-- Remote: อัพเดท customization
customizeRemote.OnServerEvent:Connect(function(player, customData)
    local current = playerCustomization[player.UserId]
    if not current then return end
    
    -- อัพเดทข้อมูล
    if customData.bodyColor then
        for part, color in pairs(customData.bodyColor) do
            current.bodyColors[part] = color
        end
    end
    
    if customData.shirt then
        current.clothes.shirt = customData.shirt
    end
    
    if customData.pants then
        current.clothes.pants = customData.pants
    end
    
    applyCustomization(player)
    print(player.Name .. " อัพเดท customization")
end)

-- Apply เมื่อ character spawn
Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function()
        task.wait(0.5)
        applyCustomization(player)
    end)
end)
```

---

## 69.3 ระบบงาน (Job System)

```lua
-- ReplicatedStorage/Modules/JobData.lua
-- ข้อมูลอาชีพ

local JobData = {}

local jobs = {
    ["police"] = {
        name = "ตำรวจ",
        description = "รักษาความปลอดภัยในเมือง",
        uniform = {
            shirt = "rbxassetid://1234",
            pants = "rbxassetid://5678",
        },
        salary = 50,            -- ต่อนาที
        salaryInterval = 60,    -- วินาที
        tools = {"Handcuffs", "Flashlight", "Radio"},
        spawnPoint = Vector3.new(50, 5, 50),
        maxPlayers = 4,
        badge = "rbxassetid://1111",
    },
    ["firefighter"] = {
        name = "นักดับเพลิง",
        description = "ดับไฟและช่วยเหลือผู้คน",
        uniform = {
            shirt = "rbxassetid://2345",
            pants = "rbxassetid://6789",
        },
        salary = 45,
        salaryInterval = 60,
        tools = {"WaterHose", "Axe"},
        spawnPoint = Vector3.new(-50, 5, 50),
        maxPlayers = 4,
    },
    ["doctor"] = {
        name = "แพทย์",
        description = "ช่วยเหลือผู้บาดเจ็บ",
        uniform = {
            shirt = "rbxassetid://3456",
            pants = "rbxassetid://7890",
        },
        salary = 60,
        salaryInterval = 60,
        tools = {"MedKit", "Stethoscope"},
        spawnPoint = Vector3.new(0, 5, 80),
        maxPlayers = 3,
    },
    ["chef"] = {
        name = "เชฟ",
        description = "ทำอาหารในร้าน",
        uniform = {
            shirt = "rbxassetid://4567",
            pants = "rbxassetid://8901",
        },
        salary = 35,
        salaryInterval = 60,
        tools = {"CookingPan", "Spatula"},
        spawnPoint = Vector3.new(-80, 5, 0),
        maxPlayers = 3,
    },
    ["taxi_driver"] = {
        name = "คนขับแท็กซี่",
        description = "รับส่งผู้โดยสาร",
        uniform = {
            shirt = "rbxassetid://5678",
            pants = "rbxassetid://9012",
        },
        salary = 40,
        salaryInterval = 60,
        tools = {"CarKey"},
        spawnPoint = Vector3.new(80, 5, 0),
        maxPlayers = 5,
    },
    ["criminal"] = {
        name = "อาชญากร",
        description = "ทำกิจกรรมผิดกฎหมาย",
        uniform = {
            shirt = "rbxassetid://6789",
            pants = "rbxassetid://0123",
        },
        salary = 0,
        salaryInterval = 0,
        tools = {"Crowbar"},
        spawnPoint = Vector3.new(0, 5, -80),
        maxPlayers = 8,
    }
}

function JobData.getJob(jobId)
    return jobs[jobId]
end

function JobData.getAllJobs()
    return jobs
end

return JobData
```

### 69.3.1 Job System Script

```lua
-- ServerScriptService/JobSystem.lua
-- ระบบอาชีพ

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local DataStoreService = game:GetService("DataStoreService")

local JobData = require(ReplicatedStorage.Modules.JobData)

local jobStore = DataStoreService:GetDataStore("JobData_v1")
local Remotes = ReplicatedStorage:WaitForChild("Remotes")

-- สร้าง Remotes
local getJobRemote = Instance.new("RemoteEvent")
getJobRemote.Name = "GetJob"
getJobRemote.Parent = Remotes

local quitJobRemote = Instance.new("RemoteEvent")
quitJobRemote.Name = "QuitJob"
quitJobRemote.Parent = Remotes

local jobUpdateEvent = Instance.new("RemoteEvent")
jobUpdateEvent.Name = "JobUpdate"
jobUpdateEvent.Parent = Remotes

-- ข้อมูลผู้เล่นในงาน
local playerJobs = {}
local jobCounts = {}  -- จำนวนผู้เล่นในแต่ละอาชีพ

-- นับจำนวน players ในแต่ละ job
local function getJobCount(jobId)
    local count = 0
    for _, data in pairs(playerJobs) do
        if data.jobId == jobId then
            count = count + 1
        end
    end
    return count
end

-- รับงาน
local function getJob(player, jobId)
    local job = JobData.getJob(jobId)
    if not job then
        return false, "ไม่พบอาชีพ"
    end
    
    -- ตรวจสอบจำนวน
    if getJobCount(jobId) >= job.maxPlayers then
        return false, "อาชีพนี้เต็มแล้ว! (" .. job.maxPlayers .. "/" .. job.maxPlayers .. ")"
    end
    
    -- ออกงานเดิมก่อน
    if playerJobs[player.UserId] then
        quitJob(player)
    end
    
    -- รับงานใหม่
    playerJobs[player.UserId] = {
        jobId = jobId,
        startTime = tick(),
        totalEarned = 0,
        lastPay = tick()
    }
    
    -- Teleport ไป spawn point ของ job
    local character = player.Character
    if character and character:FindFirstChild("HumanoidRootPart") then
        character.HumanoidRootPart.CFrame = CFrame.new(job.spawnPoint)
    end
    
    -- ใส่ uniform
    if job.uniform then
        local shirt = character:FindFirstChildWhichIsA("Shirt") or Instance.new("Shirt")
        shirt.ShirtTemplate = job.uniform.shirt
        shirt.Parent = character
        
        local pants = character:FindFirstChildWhichIsA("Pants") or Instance.new("Pants")
        pants.PantsTemplate = job.uniform.pants
        pants.Parent = character
    end
    
    -- ให้ tools
    local backpack = player:FindFirstChild("Backpack")
    if backpack then
        for _, toolName in ipairs(job.tools or {}) do
            local toolTemplate = game.ReplicatedStorage:FindFirstChild("Tools"):FindFirstChild(toolName)
            if toolTemplate then
                toolTemplate:Clone().Parent = backpack
            end
        end
    end
    
    -- อัพเดท leaderboard
    local leaderstats = player:FindFirstChild("leaderstats")
    if leaderstats then
        local jobValue = leaderstats:FindFirstChild("Job") or Instance.new("StringValue")
        jobValue.Name = "Job"
        jobValue.Value = job.name
        jobValue.Parent = leaderstats
    end
    
    jobUpdateEvent:FireClient(player, {
        status = "got_job",
        jobId = jobId,
        jobName = job.name,
        salary = job.salary
    })
    
    print(player.Name .. " เริ่มทำงาน: " .. job.name)
    return true
end

-- ออกงาน
function quitJob(player)
    local data = playerJobs[player.UserId]
    if not data then return end
    
    playerJobs[player.UserId] = nil
    
    -- ลบ tools
    local backpack = player:FindFirstChild("Backpack")
    if backpack then
        backpack:ClearAllChildren()
    end
    
    -- อัพเดท leaderboard
    local leaderstats = player:FindFirstChild("leaderstats")
    if leaderstats then
        local jobValue = leaderstats:FindFirstChild("Job")
        if jobValue then jobValue.Value = "ว่างงาน" end
    end
    
    jobUpdateEvent:FireClient(player, {status = "quit_job"})
    print(player.Name .. " ออกจากงาน")
end

-- จ่ายเงินเดือน
task.spawn(function()
    while true do
        task.wait(10)  -- ตรวจสอบทุก 10 วินาที
        
        local now = tick()
        
        for userId, jobData in pairs(playerJobs) do
            local job = JobData.getJob(jobData.jobId)
            if not job or job.salaryInterval == 0 then continue end
            
            local timeSinceLastPay = now - jobData.lastPay
            
            if timeSinceLastPay >= job.salaryInterval then
                local player = Players:GetPlayerByUserId(userId)
                if player then
                    jobData.lastPay = now
                    jobData.totalEarned = jobData.totalEarned + job.salary
                    
                    -- เพิ่มเงิน
                    local leaderstats = player:FindFirstChild("leaderstats")
                    if leaderstats then
                        local cashValue = leaderstats:FindFirstChild("Cash")
                        if cashValue then
                            cashValue.Value = cashValue.Value + job.salary
                        end
                    end
                    
                    jobUpdateEvent:FireClient(player, {
                        status = "paid",
                        amount = job.salary,
                        total = jobData.totalEarned
                    })
                    
                    print(player.Name .. " รับเงินเดือน $" .. job.salary)
                end
            end
        end
    end
end)

-- Remote handlers
getJobRemote.OnServerEvent:Connect(function(player, jobId)
    local success, message = getJob(player, jobId)
    if not success then
        jobUpdateEvent:FireClient(player, {status = "error", message = message})
    end
end)

quitJobRemote.OnServerEvent:Connect(quitJob)
```

---

## 69.4 ระบบ Emotes

```lua
-- ReplicatedStorage/Modules/EmoteData.lua
-- ข้อมูล Emotes

local EmoteData = {}

local emotes = {
    ["dance"] = {
        name = "เต้น",
        animation = "rbxassetid://507771019",  -- Default Roblox dance
        icon = "rbxassetid://507779681",
        duration = -1,  -- -1 = loop
        category = "dance"
    },
    ["wave"] = {
        name = "โบกมือ",
        animation = "rbxassetid://507770239",
        icon = "rbxassetid://507770239",
        duration = 3,
        category = "gesture"
    },
    ["sit"] = {
        name = "นั่ง",
        animation = "rbxassetid://2506281",
        icon = "rbxassetid://507770239",
        duration = -1,
        category = "pose"
    },
    ["point"] = {
        name = "ชี้",
        animation = "rbxassetid://507770453",
        icon = "rbxassetid://507770453",
        duration = 3,
        category = "gesture"
    },
    ["laugh"] = {
        name = "หัวเราะ",
        animation = "rbxassetid://507770818",
        icon = "rbxassetid://507770818",
        duration = 3,
        category = "expression"
    },
    ["salute"] = {
        name = "ยืนตรง",
        animation = "rbxassetid://3348092915",
        icon = "rbxassetid://3348092915",
        duration = 3,
        category = "gesture"
    }
}

function EmoteData.getEmote(emoteId)
    return emotes[emoteId]
end

function EmoteData.getAllEmotes()
    return emotes
end

function EmoteData.getByCategory(category)
    local result = {}
    for id, emote in pairs(emotes) do
        if emote.category == category then
            result[id] = emote
        end
    end
    return result
end

return EmoteData
```

### 69.4.1 Emote System

```lua
-- StarterPlayerScripts/EmoteSystem.lua
-- LocalScript สำหรับ Emotes

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local EmoteData = require(ReplicatedStorage.Modules.EmoteData)

local Remotes = ReplicatedStorage:WaitForChild("Remotes")
local emoteRemote = Remotes:WaitForChild("EmotePlay")

-- สร้าง Emote Wheel GUI
local playerGui = player.PlayerGui
local emoteWheelGui = Instance.new("ScreenGui")
emoteWheelGui.Name = "EmoteWheel"
emoteWheelGui.ResetOnSpawn = false
emoteWheelGui.Enabled = false
emoteWheelGui.Parent = playerGui

local wheelFrame = Instance.new("Frame")
wheelFrame.Size = UDim2.new(0, 300, 0, 300)
wheelFrame.Position = UDim2.new(0.5, -150, 0.5, -150)
wheelFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
wheelFrame.BackgroundTransparency = 0.5
wheelFrame.BorderSizePixel = 0
wheelFrame.Parent = emoteWheelGui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0.5, 0)  -- วงกลม
corner.Parent = wheelFrame

-- เพิ่ม Emote Buttons รอบวงกลม
local allEmotes = EmoteData.getAllEmotes()
local emoteList = {}
for id, data in pairs(allEmotes) do
    table.insert(emoteList, {id = id, data = data})
end

for i, emote in ipairs(emoteList) do
    local angle = (i / #emoteList) * math.pi * 2 - math.pi/2
    local radius = 100
    
    local btn = Instance.new("TextButton")
    btn.Name = emote.id
    btn.Size = UDim2.new(0, 60, 0, 60)
    btn.Position = UDim2.new(
        0.5 + math.cos(angle) * radius / 300 - 0.1,
        -30,
        0.5 + math.sin(angle) * radius / 300 - 0.1,
        -30
    )
    btn.BackgroundColor3 = Color3.fromRGB(50, 50, 80)
    btn.BorderSizePixel = 0
    btn.Text = emote.data.name
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.TextScaled = true
    btn.Font = Enum.Font.Gotham
    btn.Parent = wheelFrame
    
    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0.5, 0)
    btnCorner.Parent = btn
    
    btn.MouseButton1Click:Connect(function()
        emoteRemote:FireServer(emote.id)
        emoteWheelGui.Enabled = false
    end)
end

-- เปิด/ปิด Emote Wheel ด้วยปุ่ม E
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    if input.KeyCode == Enum.KeyCode.E then
        emoteWheelGui.Enabled = not emoteWheelGui.Enabled
        
        -- หยุดตัวละครเมื่อเปิด wheel
        if emoteWheelGui.Enabled then
            UserInputService.MouseIconEnabled = true
        end
    end
end)
```

---

## 69.5 ระบบ Interaction

```lua
-- StarterPlayerScripts/InteractionSystem.lua
-- ระบบ Interaction กับ Objects

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

-- Proximity Prompt สำหรับ Interaction
local function createInteractionPrompt(part, label, action)
    local prompt = Instance.new("ProximityPrompt")
    prompt.ActionText = label
    prompt.ObjectText = part.Name
    prompt.HoldDuration = 0
    prompt.MaxActivationDistance = 10
    prompt.RequiresLineOfSight = false
    prompt.Parent = part
    
    prompt.Triggered:Connect(function(triggeringPlayer)
        if triggeringPlayer == player then
            action(triggeringPlayer)
        end
    end)
    
    return prompt
end

-- ตั้งค่า Interactions สำหรับ Objects ต่างๆ
local function setupInteractions()
    -- ร้านค้า
    local shops = workspace:FindFirstChild("Shops")
    if shops then
        for _, shop in ipairs(shops:GetChildren()) do
            local door = shop:FindFirstChild("Door")
            if door then
                createInteractionPrompt(door, "เข้าร้าน", function(p)
                    local Remotes = ReplicatedStorage:WaitForChild("Remotes")
                    local shopRemote = Remotes:FindFirstChild("OpenShop")
                    if shopRemote then
                        shopRemote:FireServer(shop.Name)
                    end
                end)
            end
        end
    end
    
    -- เก้าอี้นั่ง
    local chairs = workspace:FindFirstChild("Chairs")
    if chairs then
        for _, chair in ipairs(chairs:GetChildren()) do
            local seat = chair:FindFirstChildWhichIsA("Seat") or chair
            
            createInteractionPrompt(chair, "นั่ง", function(p)
                local character = p.Character
                if character then
                    local humanoid = character:FindFirstChildWhichIsA("Humanoid")
                    if humanoid and seat:IsA("Seat") then
                        seat:Sit(humanoid)
                    end
                end
            end)
        end
    end
    
    -- ATM
    local atms = workspace:FindFirstChild("ATMs")
    if atms then
        for _, atm in ipairs(atms:GetChildren()) do
            createInteractionPrompt(atm, "ถอนเงิน", function(p)
                local Remotes = ReplicatedStorage:WaitForChild("Remotes")
                local atmRemote = Remotes:FindFirstChild("UseATM")
                if atmRemote then
                    atmRemote:FireServer()
                end
            end)
        end
    end
end

setupInteractions()
```

---

## 69.6 ระบบ Chat Custom

```lua
-- ServerScriptService/ChatSystem.lua
-- ระบบ Chat แบบ Custom

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Chat = game:GetService("Chat")

local Remotes = ReplicatedStorage:WaitForChild("Remotes")
local chatRemote = Instance.new("RemoteEvent")
chatRemote.Name = "CustomChat"
chatRemote.Parent = Remotes

-- Filter คำแบบ bad words
local function filterMessage(message)
    -- Roblox มี built-in chat filter แต่ตัวอย่างนี้แสดงให้เห็น custom filtering
    local filtered = Chat:FilterStringAsync(message, 0, Enum.TextFilterContext.PublicChat)
    return filtered
end

-- ส่ง Chat Message
chatRemote.OnServerEvent:Connect(function(player, message, chatType)
    -- ตรวจสอบความยาว
    if #message > 200 then
        message = message:sub(1, 200)
    end
    
    -- กรอง message
    local success, filteredMessage = pcall(function()
        local result = Chat:FilterStringAsync(message, player.UserId, Enum.TextFilterContext.PublicChat)
        return result:GetNonChatStringForBroadcastAsync()
    end)
    
    if not success then
        filteredMessage = "..."  -- ถ้า filter ล้มเหลว
    end
    
    -- ส่งไปผู้เล่นทุกคน
    local chatData = {
        playerName = player.Name,
        displayName = player.DisplayName,
        message = filteredMessage,
        chatType = chatType or "global",  -- "global", "local", "team"
        position = player.Character and 
            player.Character:FindFirstChild("HumanoidRootPart") and
            player.Character.HumanoidRootPart.Position
    }
    
    if chatType == "local" then
        -- ส่งเฉพาะผู้เล่นที่อยู่ใกล้ (100 studs)
        for _, otherPlayer in ipairs(Players:GetPlayers()) do
            if otherPlayer.Character and otherPlayer.Character:FindFirstChild("HumanoidRootPart") then
                local dist = chatData.position and 
                    (otherPlayer.Character.HumanoidRootPart.Position - chatData.position).Magnitude or 0
                
                if dist < 100 or otherPlayer == player then
                    chatRemote:FireClient(otherPlayer, chatData)
                end
            end
        end
    else
        -- Global chat
        chatRemote:FireAllClients(chatData)
    end
end)
```

---

## 69.7 Phone UI

```lua
-- StarterGui/PhoneUI/LocalScript
-- UI โทรศัพท์สำหรับ Social Features

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local phoneGui = script.Parent

-- สร้างโทรศัพท์
local phoneFrame = Instance.new("Frame")
phoneFrame.Name = "Phone"
phoneFrame.Size = UDim2.new(0, 300, 0, 500)
phoneFrame.Position = UDim2.new(1, -320, 1, -520)
phoneFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
phoneFrame.BorderSizePixel = 0
phoneFrame.Visible = false
phoneFrame.Parent = phoneGui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 20)
corner.Parent = phoneFrame

-- Phone Header
local header = Instance.new("Frame")
header.Size = UDim2.new(1, 0, 0.1, 0)
header.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
header.BorderSizePixel = 0
header.Parent = phoneFrame

local corner2 = Instance.new("UICorner")
corner2.CornerRadius = UDim.new(0, 20)
corner2.Parent = header

local timeLabel = Instance.new("TextLabel")
timeLabel.Size = UDim2.new(0.5, 0, 1, 0)
timeLabel.BackgroundTransparency = 1
timeLabel.Text = "12:00"
timeLabel.TextColor3 = Color3.new(1, 1, 1)
timeLabel.TextScaled = true
timeLabel.Font = Enum.Font.GothamBold
timeLabel.Parent = header

local signalLabel = Instance.new("TextLabel")
signalLabel.Size = UDim2.new(0.5, 0, 1, 0)
signalLabel.Position = UDim2.new(0.5, 0, 0, 0)
signalLabel.BackgroundTransparency = 1
signalLabel.Text = "📶 🔋"
signalLabel.TextColor3 = Color3.new(1, 1, 1)
signalLabel.TextScaled = true
signalLabel.Font = Enum.Font.Gotham
signalLabel.TextXAlignment = Enum.TextXAlignment.Right
signalLabel.Parent = header

-- App Icons
local appsFrame = Instance.new("Frame")
appsFrame.Size = UDim2.new(1, -20, 0.8, -20)
appsFrame.Position = UDim2.new(0, 10, 0.12, 10)
appsFrame.BackgroundTransparency = 1
appsFrame.Parent = phoneFrame

local gridLayout = Instance.new("UIGridLayout")
gridLayout.CellSize = UDim2.new(0, 75, 0, 75)
gridLayout.CellPadding = UDim2.new(0, 10, 0, 10)
gridLayout.Parent = appsFrame

-- สร้าง App ต่างๆ
local apps = {
    {name = "Jobs", icon = "💼", color = Color3.fromRGB(50, 100, 200)},
    {name = "Map", icon = "🗺️", color = Color3.fromRGB(50, 200, 50)},
    {name = "Friends", icon = "👥", color = Color3.fromRGB(100, 50, 200)},
    {name = "Chat", icon = "💬", color = Color3.fromRGB(0, 150, 255)},
    {name = "Store", icon = "🛒", color = Color3.fromRGB(200, 100, 50)},
    {name = "Stats", icon = "📊", color = Color3.fromRGB(200, 50, 50)},
    {name = "Emotes", icon = "🎭", color = Color3.fromRGB(150, 0, 200)},
    {name = "Settings", icon = "⚙️", color = Color3.fromRGB(100, 100, 100)},
}

for _, appData in ipairs(apps) do
    local appBtn = Instance.new("ImageButton")
    appBtn.Size = UDim2.new(0, 75, 0, 75)
    appBtn.BackgroundColor3 = appData.color
    appBtn.BorderSizePixel = 0
    appBtn.Parent = appsFrame
    
    local appCorner = Instance.new("UICorner")
    appCorner.CornerRadius = UDim.new(0, 15)
    appCorner.Parent = appBtn
    
    local iconLabel = Instance.new("TextLabel")
    iconLabel.Size = UDim2.new(1, 0, 0.6, 0)
    iconLabel.BackgroundTransparency = 1
    iconLabel.Text = appData.icon
    iconLabel.TextScaled = true
    iconLabel.Parent = appBtn
    
    local nameLabel = Instance.new("TextLabel")
    nameLabel.Size = UDim2.new(1, 0, 0.4, 0)
    nameLabel.Position = UDim2.new(0, 0, 0.6, 0)
    nameLabel.BackgroundTransparency = 1
    nameLabel.Text = appData.name
    nameLabel.TextColor3 = Color3.new(1, 1, 1)
    nameLabel.TextScaled = true
    nameLabel.Font = Enum.Font.Gotham
    nameLabel.Parent = appBtn
    
    appBtn.MouseButton1Click:Connect(function()
        print("เปิด App: " .. appData.name)
        -- เปิด App ที่เลือก
    end)
end

-- Phone Toggle Button
local toggleBtn = Instance.new("ImageButton")
toggleBtn.Name = "PhoneToggle"
toggleBtn.Size = UDim2.new(0, 50, 0, 50)
toggleBtn.Position = UDim2.new(1, -60, 1, -60)
toggleBtn.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
toggleBtn.BorderSizePixel = 0
toggleBtn.Parent = phoneGui

local toggleCorner = Instance.new("UICorner")
toggleCorner.CornerRadius = UDim.new(0, 10)
toggleCorner.Parent = toggleBtn

local toggleIcon = Instance.new("TextLabel")
toggleIcon.Size = UDim2.new(1, 0, 1, 0)
toggleIcon.BackgroundTransparency = 1
toggleIcon.Text = "📱"
toggleIcon.TextScaled = true
toggleIcon.Parent = toggleBtn

-- Toggle Phone
local phoneOpen = false
toggleBtn.MouseButton1Click:Connect(function()
    phoneOpen = not phoneOpen
    
    if phoneOpen then
        phoneFrame.Visible = true
        TweenService:Create(phoneFrame, TweenInfo.new(0.3, Enum.EasingStyle.Bounce), {
            Position = UDim2.new(1, -320, 0.5, -250)
        }):Play()
    else
        TweenService:Create(phoneFrame, TweenInfo.new(0.2), {
            Position = UDim2.new(1, 50, 0.5, -250)
        }):Play()
        
        task.wait(0.2)
        phoneFrame.Visible = false
    end
end)

-- อัพเดทเวลา
game:GetService("RunService").RenderStepped:Connect(function()
    local h = math.floor(game:GetService("Lighting").ClockTime)
    local m = math.floor((game:GetService("Lighting").ClockTime - h) * 60)
    timeLabel.Text = string.format("%02d:%02d", h, m)
end)
```

---

## 69.8 ข้อผิดพลาดที่พบบ่อย

```lua
-- ❌ ผิด: ไม่ Filter Chat Message
chatRemote.OnServerEvent:Connect(function(player, message)
    chatRemote:FireAllClients(player.Name .. ": " .. message)
    -- อันตราย! ไม่มี filter
end)

-- ✓ ถูก: ใช้ Roblox Chat Filter
chatRemote.OnServerEvent:Connect(function(player, message)
    local success, filtered = pcall(function()
        local result = Chat:FilterStringAsync(message, player.UserId, Enum.TextFilterContext.PublicChat)
        return result:GetNonChatStringForBroadcastAsync()
    end)
    
    if success then
        chatRemote:FireAllClients(player.Name .. ": " .. filtered)
    end
end)
```

---

## 69.9 แบบฝึกหัด

### แบบฝึกหัดที่ 1: สร้างระบบบ้าน
- ผู้เล่นซื้อบ้านด้วยเงินในเกม
- ตกแต่งภายในบ้านได้
- เชิญเพื่อนเข้าบ้านได้

### แบบฝึกหัดที่ 2: สร้างระบบการแต่งงาน
- ขอแต่งงานผู้เล่นอื่น
- ใส่แหวนในเกม
- Status แสดงในโปรไฟล์

### แบบฝึกหัดที่ 3: สร้าง Mini-game
- เกมในเกม เช่น ตีปิงปอง, Tic-Tac-Toe
- เล่นกับเพื่อนแบบ 1v1

---

## สรุป

ในบทนี้เราได้สร้าง Social Game ที่มี:
- Character Customization
- Job System
- Emote Wheel
- Interaction System
- Custom Chat
- Phone UI

ในบทถัดไปเราจะเรียนรู้ Advanced Multiplayer Systems!
