# Part 85: GamePass System - ระบบ GamePass แบบสมบูรณ์

## บทนำ

GamePasses คือสินค้าที่ผู้เล่นซื้อครั้งเดียวแล้วได้สิทธิ์ตลอดชีพ เหมาะสำหรับสิทธิ์พิเศษถาวร เช่น VIP access, permanent boosts, special abilities

---

## ส่วนที่ 1: การสร้าง GamePass

### 1.1 สร้างใน Creator Dashboard

1. ไปที่ create.roblox.com
2. เลือกเกมของคุณ
3. ไปที่ Monetization > Passes
4. คลิก "Create a Pass"
5. อัพโหลดรูปภาพ 150x150 px
6. ตั้งชื่อ, คำอธิบาย, ราคา
7. จด Pass ID

### 1.2 กำหนด GamePasses

```lua
-- ModuleScript: GamePassConfig (ReplicatedStorage)
-- กำหนดค่า GamePasses ทั้งหมด

local GamePassConfig = {}

-- GamePass IDs (แทนที่ด้วย ID จริง)
GamePassConfig.IDs = {
    VIP = 123456781,
    DOUBLE_EXP = 123456782,
    DOUBLE_COINS = 123456783,
    EXTRA_SLOTS = 123456784,
    FLYING = 123456785,
    SPEED_BOOST = 123456786,
    PREMIUM_PETS = 123456787,
    RAINBOW_TRAIL = 123456788,
    PREMIUM_CHAT = 123456789,
    DEVELOPER = 123456790,
}

-- ข้อมูลของแต่ละ GamePass
GamePassConfig.Passes = {
    [GamePassConfig.IDs.VIP] = {
        id = GamePassConfig.IDs.VIP,
        name = "VIP Pass",
        description = "รับสิทธิ์ VIP ทั้งหมด!\n• EXP x1.5\n• Coins x1.5\n• VIP Area\n• VIP Chat Badge\n• +20 Inventory Slots\n• Daily Gem Bonus",
        icon = "rbxassetid://vip_icon",
        price = 499,
        category = "premium",
        perks = {
            expMultiplier = 1.5,
            coinMultiplier = 1.5,
            inventoryBonus = 20,
            vipArea = true,
            chatBadge = "👑 VIP",
            dailyGems = 10,
        },
    },
    
    [GamePassConfig.IDs.DOUBLE_EXP] = {
        id = GamePassConfig.IDs.DOUBLE_EXP,
        name = "Double EXP",
        description = "รับ EXP สองเท่าตลอดเวลา!",
        icon = "rbxassetid://exp_icon",
        price = 249,
        category = "progression",
        perks = {
            expMultiplier = 2.0,
        },
    },
    
    [GamePassConfig.IDs.DOUBLE_COINS] = {
        id = GamePassConfig.IDs.DOUBLE_COINS,
        name = "Double Coins",
        description = "รับเหรียญสองเท่าตลอดเวลา!",
        icon = "rbxassetid://coin_icon",
        price = 249,
        category = "progression",
        perks = {
            coinMultiplier = 2.0,
        },
    },
    
    [GamePassConfig.IDs.FLYING] = {
        id = GamePassConfig.IDs.FLYING,
        name = "Flying",
        description = "บินได้! กด F เพื่อสลับโหมดบิน",
        icon = "rbxassetid://wing_icon",
        price = 399,
        category = "ability",
        perks = {
            canFly = true,
        },
    },
    
    [GamePassConfig.IDs.SPEED_BOOST] = {
        id = GamePassConfig.IDs.SPEED_BOOST,
        name = "Speed Boost",
        description = "เร็วขึ้น 50%!",
        icon = "rbxassetid://speed_icon",
        price = 199,
        category = "ability",
        perks = {
            speedMultiplier = 1.5,
        },
    },
    
    [GamePassConfig.IDs.RAINBOW_TRAIL] = {
        id = GamePassConfig.IDs.RAINBOW_TRAIL,
        name = "Rainbow Trail",
        description = "ทิ้งสายรุ้งสวยงามไว้เบื้องหลัง",
        icon = "rbxassetid://rainbow_icon",
        price = 149,
        category = "cosmetic",
        perks = {
            trailType = "rainbow",
        },
    },
}

-- หมวดหมู่
GamePassConfig.Categories = {
    premium = {name = "พรีเมียม", icon = "👑"},
    progression = {name = "ความก้าวหน้า", icon = "⬆️"},
    ability = {name = "ความสามารถ", icon = "⚡"},
    cosmetic = {name = "ตกแต่ง", icon = "✨"},
}

return GamePassConfig
```

---

## ส่วนที่ 2: Server-Side GamePass Handler

```lua
-- ModuleScript: GamePassHandler (ServerStorage)
-- ระบบจัดการ GamePass ฝั่งเซิร์ฟเวอร์

local MarketplaceService = game:GetService("MarketplaceService")
local Players = game:GetService("Players")
local GamePassConfig = require(game.ReplicatedStorage.GamePassConfig)

local GamePassHandler = {}

-- Cache ว่าผู้เล่นมี pass อะไรบ้าง
local playerPassCache = {}

-- ตรวจสอบว่าผู้เล่นมี GamePass หรือไม่
function GamePassHandler:HasPass(player, passId)
    local userId = player.UserId
    
    -- ตรวจสอบจาก cache ก่อน
    if playerPassCache[userId] and playerPassCache[userId][passId] ~= nil then
        return playerPassCache[userId][passId]
    end
    
    -- ถ้าไม่มีใน cache ให้ตรวจสอบจาก Roblox
    local success, result = pcall(function()
        return MarketplaceService:UserOwnsGamePassAsync(userId, passId)
    end)
    
    if success then
        -- บันทึกลง cache
        if not playerPassCache[userId] then
            playerPassCache[userId] = {}
        end
        playerPassCache[userId][passId] = result
        return result
    else
        warn(string.format("[GamePass] ไม่สามารถตรวจสอบ pass %d สำหรับ %s", passId, player.Name))
        return false
    end
end

-- โหลด passes ทั้งหมดของผู้เล่น
function GamePassHandler:LoadAllPasses(player)
    local userId = player.UserId
    playerPassCache[userId] = {}
    
    for passName, passId in pairs(GamePassConfig.IDs) do
        local success, owns = pcall(function()
            return MarketplaceService:UserOwnsGamePassAsync(userId, passId)
        end)
        
        if success then
            playerPassCache[userId][passId] = owns
        end
        
        -- หน่วงเวลาเล็กน้อยเพื่อไม่ให้ API throttle
        task.wait(0.1)
    end
    
    return playerPassCache[userId]
end

-- ดึง perks ทั้งหมดของผู้เล่น
function GamePassHandler:GetAllPerks(player)
    local perks = {
        expMultiplier = 1,
        coinMultiplier = 1,
        speedMultiplier = 1,
        inventoryBonus = 0,
        vipArea = false,
        canFly = false,
        chatBadge = nil,
        trailType = nil,
        dailyGems = 0,
    }
    
    for _, passConfig in pairs(GamePassConfig.Passes) do
        if self:HasPass(player, passConfig.id) then
            local passPerks = passConfig.perks
            
            -- รวม multipliers (ใช้สูงสุด ไม่ใช่คูณกัน)
            if passPerks.expMultiplier then
                perks.expMultiplier = math.max(perks.expMultiplier, passPerks.expMultiplier)
            end
            if passPerks.coinMultiplier then
                perks.coinMultiplier = math.max(perks.coinMultiplier, passPerks.coinMultiplier)
            end
            if passPerks.speedMultiplier then
                perks.speedMultiplier = math.max(perks.speedMultiplier, passPerks.speedMultiplier)
            end
            
            -- รวม bonuses (บวกกัน)
            if passPerks.inventoryBonus then
                perks.inventoryBonus = perks.inventoryBonus + passPerks.inventoryBonus
            end
            if passPerks.dailyGems then
                perks.dailyGems = perks.dailyGems + passPerks.dailyGems
            end
            
            -- Boolean flags
            if passPerks.vipArea then perks.vipArea = true end
            if passPerks.canFly then perks.canFly = true end
            
            -- String overrides
            if passPerks.chatBadge then perks.chatBadge = passPerks.chatBadge end
            if passPerks.trailType then perks.trailType = passPerks.trailType end
        end
    end
    
    return perks
end

-- ใช้ perks กับผู้เล่น
function GamePassHandler:ApplyPerks(player, character)
    local perks = self:GetAllPerks(player)
    
    -- Speed
    if perks.speedMultiplier ~= 1 then
        local humanoid = character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            humanoid.WalkSpeed = 16 * perks.speedMultiplier
        end
    end
    
    -- Store perks ใน player
    local ProfileManager = require(game.ServerStorage.ProfileManager)
    local data = ProfileManager:GetData(player)
    if data then
        data.ActivePerks = perks
    end
    
    -- แจ้ง client
    local ReplicatedStorage = game:GetService("ReplicatedStorage")
    local remotes = ReplicatedStorage:FindFirstChild("Remotes")
    if remotes then
        local perksEvent = remotes:FindFirstChild("UpdatePerks")
        if perksEvent then
            perksEvent:FireClient(player, perks)
        end
    end
end

-- ล้าง cache เมื่อผู้เล่นออก
Players.PlayerRemoving:Connect(function(player)
    task.delay(5, function()
        playerPassCache[player.UserId] = nil
    end)
end)

-- จัดการเมื่อผู้เล่นซื้อ GamePass ขณะอยู่ในเกม
MarketplaceService.PromptGamePassPurchaseFinished:Connect(function(player, passId, purchased)
    if purchased then
        -- อัพเดท cache
        if playerPassCache[player.UserId] then
            playerPassCache[player.UserId][passId] = true
        end
        
        -- ใช้ perks ใหม่
        local character = player.Character
        if character then
            GamePassHandler:ApplyPerks(player, character)
        end
        
        print(string.format("[GamePass] %s ซื้อ pass %d", player.Name, passId))
        
        -- แจ้ง client
        local ReplicatedStorage = game:GetService("ReplicatedStorage")
        local remotes = ReplicatedStorage:FindFirstChild("Remotes")
        if remotes then
            local purchaseEvent = remotes:FindFirstChild("GamePassPurchased")
            if purchaseEvent then
                local passConfig = GamePassConfig.Passes[passId]
                if passConfig then
                    purchaseEvent:FireClient(player, passConfig)
                end
            end
        end
    end
end)

return GamePassHandler
```

---

## ส่วนที่ 3: Flying System (GamePass Ability)

```lua
-- LocalScript: FlyingSystem (StarterCharacterScripts)
-- ระบบบิน (ต้องมี Flying GamePass)

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local hrp = character:WaitForChild("HumanoidRootPart")

local canFly = false
local isFlying = false
local flySpeed = 50
local flyHeight = 0
local bodyVelocity = nil
local bodyGyro = nil

-- ตรวจสอบสิทธิ์บิน
local remotes = ReplicatedStorage:WaitForChild("Remotes")
local getPerksEvent = remotes:WaitForChild("GetPerks")

local perks = getPerksEvent:InvokeServer()
if perks and perks.canFly then
    canFly = true
end

-- อัพเดท perks เมื่อซื้อ pass
local updatePerksEvent = remotes:WaitForChild("UpdatePerks")
updatePerksEvent.OnClientEvent:Connect(function(newPerks)
    if newPerks.canFly then
        canFly = true
    end
end)

-- เริ่มบิน
local function startFlying()
    if not canFly or isFlying then return end
    isFlying = true
    
    humanoid.PlatformStand = true
    
    -- สร้าง BodyVelocity
    bodyVelocity = Instance.new("BodyVelocity")
    bodyVelocity.Velocity = Vector3.zero
    bodyVelocity.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
    bodyVelocity.Parent = hrp
    
    -- สร้าง BodyGyro
    bodyGyro = Instance.new("BodyGyro")
    bodyGyro.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
    bodyGyro.D = 100
    bodyGyro.CFrame = hrp.CFrame
    bodyGyro.Parent = hrp
    
    print("[Fly] เริ่มบิน")
end

-- หยุดบิน
local function stopFlying()
    if not isFlying then return end
    isFlying = false
    
    humanoid.PlatformStand = false
    
    if bodyVelocity then
        bodyVelocity:Destroy()
        bodyVelocity = nil
    end
    
    if bodyGyro then
        bodyGyro:Destroy()
        bodyGyro = nil
    end
    
    print("[Fly] หยุดบิน")
end

-- Toggle บิน
local function toggleFly()
    if isFlying then
        stopFlying()
    else
        startFlying()
    end
end

-- Input สำหรับบิน
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if not canFly then return end
    
    if input.KeyCode == Enum.KeyCode.F then
        toggleFly()
    end
end)

-- อัพเดทการบิน
RunService.Heartbeat:Connect(function()
    if not isFlying or not bodyVelocity then return end
    
    local camera = workspace.CurrentCamera
    local moveDirection = humanoid.MoveDirection
    
    -- ทิศทางการบิน
    local velocity = Vector3.zero
    
    if moveDirection.Magnitude > 0 then
        velocity = moveDirection * flySpeed
    end
    
    -- ขึ้น/ลง
    if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
        velocity = velocity + Vector3.new(0, flySpeed * 0.5, 0)
    elseif UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then
        velocity = velocity + Vector3.new(0, -flySpeed * 0.5, 0)
    end
    
    bodyVelocity.Velocity = velocity
    
    -- หมุนตามทิศทางกล้อง
    if velocity.Magnitude > 0 then
        local lookAt = Vector3.new(velocity.X, 0, velocity.Z)
        if lookAt.Magnitude > 0 then
            bodyGyro.CFrame = CFrame.lookAt(hrp.Position, hrp.Position + lookAt)
        end
    end
end)

-- หยุดบินเมื่อ character respawn
humanoid.Died:Connect(stopFlying)
```

---

## ส่วนที่ 4: VIP Area System

```lua
-- Script: VIPAreaSystem (Script)
-- ระบบพื้นที่ VIP

local GamePassHandler = require(game.ServerStorage.GamePassHandler)
local Players = game:GetService("Players")

-- สร้าง VIP Area
local function createVIPArea()
    local vipArea = workspace:FindFirstChild("VIPArea")
    if not vipArea then return end
    
    -- หาประตู/กำแพง VIP
    local vipDoor = vipArea:FindFirstChild("VIPDoor")
    
    -- Touch detector
    local detector = vipArea:FindFirstChild("Detector") or Instance.new("Part")
    detector.Name = "Detector"
    detector.Size = Vector3.new(10, 20, 2)
    detector.Transparency = 1
    detector.CanCollide = false
    detector.Parent = vipArea
    
    detector.Touched:Connect(function(hit)
        local character = hit.Parent
        local player = Players:GetPlayerFromCharacter(character)
        if not player then return end
        
        -- ตรวจสอบ VIP
        local hasVIP = GamePassHandler:HasPass(player, GamePassConfig.IDs.VIP)
        
        if not hasVIP then
            -- ผลัก/บล็อก
            local hrp = character:FindFirstChild("HumanoidRootPart")
            if hrp then
                hrp.Velocity = hrp.CFrame.LookVector * -30
            end
            
            -- แจ้งเตือน
            local remotes = game.ReplicatedStorage:FindFirstChild("Remotes")
            if remotes then
                local notifyEvent = remotes:FindFirstChild("Notify")
                if notifyEvent then
                    notifyEvent:FireClient(player, {
                        message = "⚠️ ต้องมี VIP Pass เพื่อเข้าพื้นที่นี้",
                        type = "warning",
                        action = {
                            text = "ซื้อ VIP",
                            passId = GamePassConfig.IDs.VIP,
                        }
                    })
                end
            end
        end
    end)
end

-- สร้าง VIP Area เมื่อเกมเริ่ม
task.spawn(createVIPArea)
```

---

## ส่วนที่ 5: GamePass UI

```lua
-- LocalScript: GamePassUI
-- UI แสดงและซื้อ GamePasses

local Players = game:GetService("Players")
local MarketplaceService = game:GetService("MarketplaceService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local player = Players.LocalPlayer

-- สร้าง GamePass Shop UI
local function createGamePassShop()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "GamePassShop"
    screenGui.Parent = player.PlayerGui
    
    local mainFrame = Instance.new("Frame")
    mainFrame.Size = UDim2.new(0, 800, 0, 550)
    mainFrame.Position = UDim2.new(0.5, -400, 0.5, -275)
    mainFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 25)
    mainFrame.Visible = false
    mainFrame.Parent = screenGui
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 15)
    corner.Parent = mainFrame
    
    -- Title
    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, 0, 0, 55)
    title.BackgroundColor3 = Color3.fromRGB(20, 20, 35)
    title.Text = "✨ GamePasses"
    title.TextColor3 = Color3.fromRGB(255, 215, 0)
    title.TextSize = 26
    title.Font = Enum.Font.GothamBold
    title.Parent = mainFrame
    
    local titleCorner = Instance.new("UICorner")
    titleCorner.CornerRadius = UDim.new(0, 15)
    titleCorner.Parent = title
    
    -- Close button
    local closeBtn = Instance.new("TextButton")
    closeBtn.Size = UDim2.new(0, 40, 0, 40)
    closeBtn.Position = UDim2.new(1, -45, 0, 7)
    closeBtn.BackgroundColor3 = Color3.fromRGB(180, 50, 50)
    closeBtn.Text = "✕"
    closeBtn.TextColor3 = Color3.new(1, 1, 1)
    closeBtn.TextSize = 20
    closeBtn.Font = Enum.Font.GothamBold
    closeBtn.Parent = mainFrame
    
    local closeBtnCorner = Instance.new("UICorner")
    closeBtnCorner.CornerRadius = UDim.new(0, 8)
    closeBtnCorner.Parent = closeBtn
    
    closeBtn.MouseButton1Click:Connect(function()
        mainFrame.Visible = false
    end)
    
    -- Scroll area
    local scrollFrame = Instance.new("ScrollingFrame")
    scrollFrame.Size = UDim2.new(1, -20, 0.87, 0)
    scrollFrame.Position = UDim2.new(0, 10, 0.12, 0)
    scrollFrame.BackgroundTransparency = 1
    scrollFrame.ScrollBarThickness = 6
    scrollFrame.Parent = mainFrame
    
    -- Grid layout
    local grid = Instance.new("UIGridLayout")
    grid.CellSize = UDim2.new(0, 230, 0, 280)
    grid.CellPadding = UDim2.new(0, 10, 0, 10)
    grid.Parent = scrollFrame
    
    local GamePassConfig = require(ReplicatedStorage:WaitForChild("GamePassConfig"))
    
    -- สร้างการ์ดสำหรับแต่ละ GamePass
    for passId, passConfig in pairs(GamePassConfig.Passes) do
        local card = createPassCard(passConfig, scrollFrame)
    end
    
    -- อัพเดทขนาด scroll
    local function updateScrollSize()
        local count = #scrollFrame:GetChildren() - 1 -- -1 for UIGridLayout
        local rows = math.ceil(count / 3)
        scrollFrame.CanvasSize = UDim2.new(0, 0, 0, rows * 290 + 10)
    end
    
    updateScrollSize()
    
    return mainFrame
end

-- สร้างการ์ด GamePass
local function createPassCard(passConfig, parent)
    local card = Instance.new("Frame")
    card.Size = UDim2.new(0, 230, 0, 280)
    card.BackgroundColor3 = Color3.fromRGB(25, 25, 40)
    card.Parent = parent
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 12)
    corner.Parent = card
    
    -- Gradient background
    local gradient = Instance.new("UIGradient")
    gradient.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(30, 30, 50)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(20, 20, 35)),
    })
    gradient.Rotation = 135
    gradient.Parent = card
    
    -- Icon
    local iconFrame = Instance.new("Frame")
    iconFrame.Size = UDim2.new(0, 80, 0, 80)
    iconFrame.Position = UDim2.new(0.5, -40, 0, 15)
    iconFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 65)
    iconFrame.Parent = card
    
    local iconCorner = Instance.new("UICorner")
    iconCorner.CornerRadius = UDim.new(0, 12)
    iconCorner.Parent = iconFrame
    
    local icon = Instance.new("ImageLabel")
    icon.Size = UDim2.new(0.8, 0, 0.8, 0)
    icon.Position = UDim2.new(0.1, 0, 0.1, 0)
    icon.BackgroundTransparency = 1
    icon.Image = passConfig.icon or ""
    icon.Parent = iconFrame
    
    -- Name
    local nameLabel = Instance.new("TextLabel")
    nameLabel.Size = UDim2.new(1, -10, 0, 35)
    nameLabel.Position = UDim2.new(0, 5, 0, 100)
    nameLabel.BackgroundTransparency = 1
    nameLabel.Text = passConfig.name
    nameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    nameLabel.TextSize = 16
    nameLabel.Font = Enum.Font.GothamBold
    nameLabel.TextWrapped = true
    nameLabel.Parent = card
    
    -- Description
    local descLabel = Instance.new("TextLabel")
    descLabel.Size = UDim2.new(1, -10, 0, 80)
    descLabel.Position = UDim2.new(0, 5, 0, 138)
    descLabel.BackgroundTransparency = 1
    descLabel.Text = passConfig.description
    descLabel.TextColor3 = Color3.fromRGB(180, 180, 200)
    descLabel.TextSize = 11
    descLabel.Font = Enum.Font.Gotham
    descLabel.TextWrapped = true
    descLabel.TextYAlignment = Enum.TextYAlignment.Top
    descLabel.Parent = card
    
    -- Price
    local priceLabel = Instance.new("TextLabel")
    priceLabel.Size = UDim2.new(1, -10, 0, 22)
    priceLabel.Position = UDim2.new(0, 5, 0, 220)
    priceLabel.BackgroundTransparency = 1
    priceLabel.Text = "💎 " .. passConfig.price .. " Robux"
    priceLabel.TextColor3 = Color3.fromRGB(100, 200, 255)
    priceLabel.TextSize = 13
    priceLabel.Font = Enum.Font.GothamBold
    priceLabel.Parent = card
    
    -- Buy button
    local buyBtn = Instance.new("TextButton")
    buyBtn.Size = UDim2.new(0.88, 0, 0, 35)
    buyBtn.Position = UDim2.new(0.06, 0, 0, 240)
    buyBtn.BackgroundColor3 = Color3.fromRGB(80, 120, 220)
    buyBtn.Text = "ซื้อ GamePass"
    buyBtn.TextColor3 = Color3.new(1, 1, 1)
    buyBtn.TextSize = 14
    buyBtn.Font = Enum.Font.GothamBold
    buyBtn.Parent = card
    
    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 8)
    btnCorner.Parent = buyBtn
    
    buyBtn.MouseButton1Click:Connect(function()
        MarketplaceService:PromptGamePassPurchase(Players.LocalPlayer, passConfig.id)
    end)
    
    return card
end
```

---

## ส่วนที่ 6: Chat Badge System (VIP Badge)

```lua
-- Script: ChatBadgeSystem (Script ใน ServerScriptService)
-- ระบบแสดงป้ายในแชท

local Players = game:GetService("Players")
local GamePassHandler = require(game.ServerStorage.GamePassHandler)
local GamePassConfig = require(game.ReplicatedStorage.GamePassConfig)

-- ฟังก์ชันใส่ badge ในชื่อ
local function addChatBadge(player)
    local perks = GamePassHandler:GetAllPerks(player)
    
    if perks.chatBadge then
        -- ใช้ Chat service เพื่อแสดง badge
        -- (ต้องใช้ ChatService หรือ custom chat)
        local chatTag = {
            TagText = perks.chatBadge,
            TagColor = BrickColor.new("Bright yellow"),
        }
        
        -- สำหรับ Roblox default chat:
        local Chat = game:GetService("Chat")
        -- Chat อาจต้องการ module พิเศษ
        
        print(string.format("[ChatBadge] %s ได้รับ badge: %s", player.Name, perks.chatBadge))
    end
end

Players.PlayerAdded:Connect(function(player)
    -- รอโหลด passes
    task.delay(2, function()
        addChatBadge(player)
    end)
end)
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: GamePass Benefits UI
สร้าง UI แสดงประโยชน์ GamePass ที่:
1. เปรียบเทียบ free vs premium
2. แสดง perks ที่ active
3. แนะนำ pass ที่คุ้มค่าที่สุด

### แบบฝึกหัดที่ 2: Pass Combo Bonuses
สร้างระบบที่ให้ bonus พิเศษเมื่อมี pass หลายอัน:
1. ซื้อ VIP + Double EXP = EXP x3
2. ซื้อ Speed + Flying = Super Speed
3. แสดง combo ที่เป็นไปได้

### แบบฝึกหัดที่ 3: Pass Preview
สร้างระบบ preview pass ที่:
1. แสดงตัวอย่าง skin/effect
2. Demo flying ชั่วคราว 10 วินาที
3. Compare before/after speed

---

## สรุปบทที่ 85

GamePass System ที่สมบูรณ์ประกอบด้วย:

1. **Configuration** - จัดการ pass IDs และ perks อย่างเป็นระบบ
2. **Server Validation** - ตรวจสอบ ownership ที่ server เสมอ
3. **Perk Application** - ใช้ perks กับ gameplay อย่างถูกต้อง
4. **UI/UX** - Shop ที่น่าใช้และแสดงข้อมูลครบถ้วน
5. **Cache System** - ลด API calls ด้วย caching

*บทถัดไป: Part 86 - VIP and Premium Features*
