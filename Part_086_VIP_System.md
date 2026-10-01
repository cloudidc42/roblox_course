# Part 86: VIP System - ระบบสมาชิก VIP และสิทธิพิเศษ

## บทนำ

ระบบ VIP เป็นหนึ่งในวิธีที่ดีที่สุดในการสร้างรายได้อย่างยั่งยืน โดยให้สิทธิพิเศษมากมายที่ทำให้ประสบการณ์การเล่นดีขึ้น ในบทนี้เราจะสร้างระบบ VIP ที่สมบูรณ์และหลากหลาย

---

## ส่วนที่ 1: VIP Tier System

```lua
-- ModuleScript: VIPConfig (ReplicatedStorage)
-- กำหนด VIP Tiers และสิทธิพิเศษ

local VIPConfig = {}

-- VIP Tier IDs (GamePass IDs จาก Creator Dashboard)
VIPConfig.TierIDs = {
    BRONZE = 111111101,  -- 99 Robux
    SILVER = 111111102,  -- 249 Robux
    GOLD = 111111103,    -- 499 Robux
    DIAMOND = 111111104, -- 999 Robux
    LEGEND = 111111105,  -- 1999 Robux
}

-- ข้อมูลแต่ละ tier
VIPConfig.Tiers = {
    BRONZE = {
        id = VIPConfig.TierIDs.BRONZE,
        name = "Bronze VIP",
        color = Color3.fromRGB(180, 120, 80),
        badge = "🥉",
        price = 99,
        
        perks = {
            -- Multipliers
            expMultiplier = 1.25,
            coinMultiplier = 1.25,
            
            -- Bonuses
            extraLives = 1,
            inventorySlots = 10,
            dailyCoins = 500,
            
            -- Features
            privateArea = false,
            customName = false,
            vipChat = true,
            
            -- Cosmetics
            nameColor = Color3.fromRGB(180, 120, 80),
            chatColor = Color3.fromRGB(180, 120, 80),
        },
    },
    
    SILVER = {
        id = VIPConfig.TierIDs.SILVER,
        name = "Silver VIP",
        color = Color3.fromRGB(200, 200, 210),
        badge = "🥈",
        price = 249,
        
        perks = {
            expMultiplier = 1.5,
            coinMultiplier = 1.5,
            extraLives = 2,
            inventorySlots = 25,
            dailyCoins = 1000,
            dailyGems = 5,
            privateArea = false,
            customName = false,
            vipChat = true,
            skipQueue = true,
            nameColor = Color3.fromRGB(200, 200, 210),
        },
    },
    
    GOLD = {
        id = VIPConfig.TierIDs.GOLD,
        name = "Gold VIP",
        color = Color3.fromRGB(255, 215, 0),
        badge = "👑",
        price = 499,
        
        perks = {
            expMultiplier = 2.0,
            coinMultiplier = 2.0,
            extraLives = 3,
            inventorySlots = 50,
            dailyCoins = 2500,
            dailyGems = 15,
            privateArea = true,
            customName = false,
            vipChat = true,
            skipQueue = true,
            exclusivePets = {"GoldenDragon"},
            nameColor = Color3.fromRGB(255, 215, 0),
        },
    },
    
    DIAMOND = {
        id = VIPConfig.TierIDs.DIAMOND,
        name = "Diamond VIP",
        color = Color3.fromRGB(100, 200, 255),
        badge = "💎",
        price = 999,
        
        perks = {
            expMultiplier = 2.5,
            coinMultiplier = 2.5,
            extraLives = 5,
            inventorySlots = 100,
            dailyCoins = 5000,
            dailyGems = 30,
            privateArea = true,
            customName = true,
            vipChat = true,
            skipQueue = true,
            exclusivePets = {"DiamondPhoenix"},
            exclusiveZone = true,
            nameColor = Color3.fromRGB(100, 200, 255),
        },
    },
    
    LEGEND = {
        id = VIPConfig.TierIDs.LEGEND,
        name = "Legend VIP",
        color = Color3.fromRGB(255, 100, 255),
        badge = "⭐",
        price = 1999,
        
        perks = {
            expMultiplier = 3.0,
            coinMultiplier = 3.0,
            extraLives = 10,
            inventorySlots = 200,
            dailyCoins = 10000,
            dailyGems = 75,
            privateArea = true,
            customName = true,
            vipChat = true,
            skipQueue = true,
            exclusivePets = {"LegendaryUnicorn", "CosmicDragon"},
            exclusiveZone = true,
            developerContact = true, -- สามารถติดต่อนักพัฒนาได้
            nameColor = Color3.fromRGB(255, 100, 255),
            nameGlow = true,
        },
    },
}

-- ลำดับ tier (สูงกว่าได้ประโยชน์มากกว่า)
VIPConfig.TierOrder = {"BRONZE", "SILVER", "GOLD", "DIAMOND", "LEGEND"}

return VIPConfig
```

---

## ส่วนที่ 2: VIP Manager (Server)

```lua
-- ModuleScript: VIPManager (ServerStorage)
-- ระบบจัดการ VIP

local MarketplaceService = game:GetService("MarketplaceService")
local Players = game:GetService("Players")
local VIPConfig = require(game.ReplicatedStorage.VIPConfig)

local VIPManager = {}
local vipCache = {} -- cache VIP tier ของผู้เล่น

-- ดึง VIP Tier ของผู้เล่น (คืนค่า tier สูงสุด)
function VIPManager:GetTier(player)
    local userId = player.UserId
    
    -- ตรวจสอบ cache
    if vipCache[userId] then
        return vipCache[userId]
    end
    
    local highestTier = nil
    
    -- ตรวจสอบแต่ละ tier จากสูงสุดไปต่ำสุด
    for i = #VIPConfig.TierOrder, 1, -1 do
        local tierName = VIPConfig.TierOrder[i]
        local tier = VIPConfig.Tiers[tierName]
        
        local success, owns = pcall(function()
            return MarketplaceService:UserOwnsGamePassAsync(userId, tier.id)
        end)
        
        if success and owns then
            highestTier = tierName
            break
        end
        
        task.wait(0.05) -- ป้องกัน throttle
    end
    
    -- บันทึก cache
    vipCache[userId] = highestTier
    return highestTier
end

-- ดึง perks ของผู้เล่น
function VIPManager:GetPerks(player)
    local tier = self:GetTier(player)
    if not tier then return nil end
    
    return VIPConfig.Tiers[tier].perks
end

-- ดึงข้อมูล tier
function VIPManager:GetTierInfo(player)
    local tier = self:GetTier(player)
    if not tier then return nil end
    
    return VIPConfig.Tiers[tier], tier
end

-- ตรวจสอบสิทธิ์เฉพาะ
function VIPManager:HasPerk(player, perkName)
    local perks = self:GetPerks(player)
    if not perks then return false end
    
    return perks[perkName] == true or (type(perks[perkName]) == "number" and perks[perkName] > 0)
end

-- ใช้ perks กับผู้เล่น
function VIPManager:ApplyPerks(player, character)
    local tier = self:GetTier(player)
    if not tier then return end
    
    local perks = VIPConfig.Tiers[tier].perks
    local tierInfo = VIPConfig.Tiers[tier]
    
    -- Speed (ถ้ามี speed perk)
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if humanoid then
        -- เก็บ speed เพิ่มเติม
        humanoid.WalkSpeed = 16 -- base speed
    end
    
    -- บันทึก VIP data ใน profile
    local ProfileManager = require(game.ServerStorage.ProfileManager)
    local data = ProfileManager:GetData(player)
    
    if data then
        data.VIPTier = tier
        data.VIPPerks = perks
        data.VIPBadge = tierInfo.badge
        data.VIPColor = {
            R = tierInfo.color.R,
            G = tierInfo.color.G,
            B = tierInfo.color.B,
        }
    end
    
    -- แจ้ง client
    local remotes = game.ReplicatedStorage:FindFirstChild("Remotes")
    if remotes then
        local vipEvent = remotes:FindFirstChild("VIPUpdate")
        if vipEvent then
            vipEvent:FireClient(player, {
                tier = tier,
                tierName = tierInfo.name,
                badge = tierInfo.badge,
                color = tierInfo.color,
                perks = perks,
            })
        end
    end
    
    print(string.format("[VIP] ใช้ perks %s สำหรับ %s", tier, player.Name))
end

-- ให้รางวัล daily VIP
function VIPManager:ClaimDailyRewards(player)
    local perks = self:GetPerks(player)
    if not perks then return false, "ไม่มี VIP" end
    
    local ProfileManager = require(game.ServerStorage.ProfileManager)
    local data = ProfileManager:GetData(player)
    if not data then return false, "ไม่พบข้อมูล" end
    
    -- ตรวจสอบว่ารับรางวัลวันนี้แล้วหรือยัง
    local today = os.date("%Y-%m-%d")
    if data.LastVIPDailyClaim == today then
        return false, "รับรางวัลวันนี้แล้ว"
    end
    
    -- ให้รางวัล
    if perks.dailyCoins then
        data.Coins = (data.Coins or 0) + perks.dailyCoins
    end
    if perks.dailyGems then
        data.Gems = (data.Gems or 0) + perks.dailyGems
    end
    
    data.LastVIPDailyClaim = today
    
    return true, {
        coins = perks.dailyCoins or 0,
        gems = perks.dailyGems or 0,
    }
end

-- ล้าง cache เมื่อออกจากเกม
Players.PlayerRemoving:Connect(function(player)
    task.delay(5, function()
        vipCache[player.UserId] = nil
    end)
end)

-- อัพเดท cache เมื่อซื้อ GamePass
MarketplaceService.PromptGamePassPurchaseFinished:Connect(function(player, passId, purchased)
    if purchased then
        vipCache[player.UserId] = nil -- ล้าง cache
        task.delay(1, function()
            if player:IsDescendantOf(Players) then
                local character = player.Character
                if character then
                    VIPManager:ApplyPerks(player, character)
                end
            end
        end)
    end
end)

return VIPManager
```

---

## ส่วนที่ 3: VIP Visual Effects

```lua
-- LocalScript: VIPVisuals (StarterCharacterScripts)
-- เอฟเฟกต์ภาพสำหรับ VIP

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()

local vipData = nil

-- รับข้อมูล VIP
local remotes = ReplicatedStorage:WaitForChild("Remotes")
local vipEvent = remotes:WaitForChild("VIPUpdate")

vipEvent.OnClientEvent:Connect(function(data)
    vipData = data
    applyVIPEffects()
end)

-- ใช้เอฟเฟกต์ VIP
local function applyVIPEffects()
    if not vipData then return end
    
    -- ชื่อเรืองแสง (Name Glow)
    local nameTag = character:FindFirstChild("NameTag")
    if not nameTag then
        nameTag = Instance.new("BillboardGui")
        nameTag.Name = "NameTag"
        nameTag.Size = UDim2.new(0, 200, 0, 50)
        nameTag.StudsOffset = Vector3.new(0, 3, 0)
        nameTag.AlwaysOnTop = false
        nameTag.Parent = character:WaitForChild("HumanoidRootPart")
    end
    
    -- แสดงชื่อ VIP
    local nameLabel = nameTag:FindFirstChild("NameLabel") or Instance.new("TextLabel")
    nameLabel.Name = "NameLabel"
    nameLabel.Size = UDim2.new(1, 0, 1, 0)
    nameLabel.BackgroundTransparency = 1
    nameLabel.Text = string.format("%s %s", vipData.badge or "", player.Name)
    nameLabel.TextColor3 = vipData.color or Color3.new(1, 1, 1)
    nameLabel.TextSize = 16
    nameLabel.Font = Enum.Font.GothamBold
    nameLabel.TextStrokeTransparency = 0.5
    nameLabel.Parent = nameTag
    
    -- Glow effect สำหรับ Legend tier
    if vipData.perks and vipData.perks.nameGlow then
        local tween = TweenService:Create(
            nameLabel,
            TweenInfo.new(1, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, true),
            {TextColor3 = Color3.fromRGB(255, 200, 255)}
        )
        tween:Play()
    end
end

-- VIP Trail effect
local function createVIPTrail()
    if not vipData then return end
    
    local hrp = character:WaitForChild("HumanoidRootPart")
    
    -- สร้าง attachment points
    local attachment0 = Instance.new("Attachment")
    attachment0.Name = "VIPTrailAttachment0"
    attachment0.Position = Vector3.new(0, 1, 0)
    attachment0.Parent = hrp
    
    local attachment1 = Instance.new("Attachment")
    attachment1.Name = "VIPTrailAttachment1"
    attachment1.Position = Vector3.new(0, -1, 0)
    attachment1.Parent = hrp
    
    -- สร้าง Trail
    local trail = Instance.new("Trail")
    trail.Attachment0 = attachment0
    trail.Attachment1 = attachment1
    trail.Lifetime = 0.5
    trail.MinLength = 0.1
    
    -- สีตาม tier
    local tierColor = vipData.color or Color3.new(1, 1, 1)
    trail.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, tierColor),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(255, 255, 255)),
    })
    
    trail.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0),
        NumberSequenceKeypoint.new(1, 1),
    })
    
    trail.Parent = hrp
end

-- VIP Particle effect
local function createVIPParticles()
    if not vipData then return end
    if not vipData.perks or not vipData.perks.nameGlow then return end
    
    local hrp = character:WaitForChild("HumanoidRootPart")
    
    local particles = Instance.new("ParticleEmitter")
    particles.Rate = 5
    particles.Speed = NumberRange.new(2, 5)
    particles.Lifetime = NumberRange.new(0.5, 1)
    particles.Size = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.2),
        NumberSequenceKeypoint.new(0.5, 0.3),
        NumberSequenceKeypoint.new(1, 0),
    })
    particles.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0),
        NumberSequenceKeypoint.new(1, 1),
    })
    particles.Color = ColorSequence.new(vipData.color or Color3.new(1, 1, 1))
    particles.Parent = hrp
end

-- เริ่มเอฟเฟกต์
applyVIPEffects()
createVIPTrail()
createVIPParticles()
```

---

## ส่วนที่ 4: VIP Exclusive Area

```lua
-- Script: VIPExclusiveArea (Script)
-- พื้นที่สำหรับ VIP โดยเฉพาะ

local Players = game:GetService("Players")
local VIPManager = require(game.ServerStorage.VIPManager)
local TweenService = game:GetService("TweenService")

-- สร้างพื้นที่ VIP
local function setupVIPZone()
    -- หา VIP Zone ในแผนที่
    local vipZone = workspace:FindFirstChild("VIPZone")
    if not vipZone then
        -- สร้างพื้นที่ใหม่ถ้าไม่มี
        vipZone = Instance.new("Model")
        vipZone.Name = "VIPZone"
        vipZone.Parent = workspace
        
        -- พื้น VIP
        local floor = Instance.new("Part")
        floor.Name = "Floor"
        floor.Size = Vector3.new(50, 1, 50)
        floor.Position = Vector3.new(0, 0.5, 100)
        floor.BrickColor = BrickColor.new("Bright yellow")
        floor.Material = Enum.Material.SmoothPlastic
        floor.Anchored = true
        floor.Parent = vipZone
        
        -- Trigger zone
        local trigger = Instance.new("Part")
        trigger.Name = "Trigger"
        trigger.Size = Vector3.new(52, 20, 52)
        trigger.Position = Vector3.new(0, 10, 100)
        trigger.Transparency = 0.9
        trigger.CanCollide = false
        trigger.BrickColor = BrickColor.new("Bright yellow")
        trigger.Anchored = true
        trigger.Parent = vipZone
        
        return vipZone, trigger
    end
    
    return vipZone, vipZone:FindFirstChild("Trigger")
end

local vipZone, trigger = setupVIPZone()

if trigger then
    -- ตรวจสอบเมื่อมีคนเข้า VIP Zone
    trigger.Touched:Connect(function(hit)
        local character = hit.Parent
        local player = Players:GetPlayerFromCharacter(character)
        if not player then return end
        
        -- ตรวจสอบ VIP tier
        local tier = VIPManager:GetTier(player)
        local tierInfo = tier and VIPConfig.Tiers[tier]
        
        if not tier or not tierInfo.perks.privateArea then
            -- ไม่มีสิทธิ์เข้า
            local hrp = character:FindFirstChild("HumanoidRootPart")
            if hrp then
                -- ผลัก
                hrp.Velocity = (hrp.Position - Vector3.new(0, 0, 100)).Unit * 50
            end
            
            -- แจ้ง
            local remotes = game.ReplicatedStorage:FindFirstChild("Remotes")
            if remotes then
                local notifyEvent = remotes:FindFirstChild("Notify")
                if notifyEvent then
                    notifyEvent:FireClient(player, {
                        message = "🔒 พื้นที่นี้สำหรับ Gold VIP ขึ้นไปเท่านั้น",
                        type = "locked",
                    })
                end
            end
        else
            -- มีสิทธิ์เข้า - ให้ bonus
            local ProfileManager = require(game.ServerStorage.ProfileManager)
            local data = ProfileManager:GetData(player)
            if data then
                -- ให้ bonus เหรียญในพื้นที่ VIP
                data.VIPZoneBonus = true
            end
        end
    end)
    
    trigger.TouchEnded:Connect(function(hit)
        local character = hit.Parent
        local player = Players:GetPlayerFromCharacter(character)
        if not player then return end
        
        local ProfileManager = require(game.ServerStorage.ProfileManager)
        local data = ProfileManager:GetData(player)
        if data then
            data.VIPZoneBonus = false
        end
    end)
end
```

---

## ส่วนที่ 5: VIP Chat System

```lua
-- Script: VIPChatSystem (Script ใน ServerScriptService)
-- ระบบแชท VIP พิเศษ

local Players = game:GetService("Players")
local VIPManager = require(game.ServerStorage.VIPManager)
local TextService = game:GetService("TextService")

-- ช่อง VIP Chat
local VIP_CHANNEL = "VIP"
local MINIMUM_TIER_FOR_VIP_CHAT = "SILVER"

-- ตรวจสอบว่าสามารถใช้ VIP chat ได้
local function canUseVIPChat(player)
    local tier = VIPManager:GetTier(player)
    if not tier then return false end
    
    local tierIndex = table.find(VIPConfig.TierOrder, tier)
    local minIndex = table.find(VIPConfig.TierOrder, MINIMUM_TIER_FOR_VIP_CHAT)
    
    return tierIndex and minIndex and tierIndex >= minIndex
end

-- ส่งข้อความ VIP
local function sendVIPMessage(player, message)
    if not canUseVIPChat(player) then
        return false, "ต้องมี Silver VIP ขึ้นไปเพื่อใช้ VIP Chat"
    end
    
    -- Filter ข้อความ
    local filteredMessage
    local success, result = pcall(function()
        return TextService:FilterStringAsync(
            message,
            player.UserId,
            Enum.TextFilterContext.PublicChat
        )
    end)
    
    if not success then
        return false, "ไม่สามารถกรองข้อความได้"
    end
    filteredMessage = result
    
    -- ส่งไปยัง VIP players ทั้งหมด
    local tierInfo = VIPManager:GetTierInfo(player)
    local badge = tierInfo and tierInfo.badge or "💎"
    
    for _, targetPlayer in ipairs(Players:GetPlayers()) do
        if canUseVIPChat(targetPlayer) then
            -- ส่ง event ไปยัง VIP players
            local remotes = game.ReplicatedStorage:FindFirstChild("Remotes")
            if remotes then
                local chatEvent = remotes:FindFirstChild("VIPChat")
                if chatEvent then
                    local displayMessage
                    local filterSuccess, filterResult = pcall(function()
                        return filteredMessage:GetNonChatStringForUserAsync(targetPlayer.UserId)
                    end)
                    
                    if filterSuccess then
                        displayMessage = filterResult
                    else
                        displayMessage = "..."
                    end
                    
                    chatEvent:FireClient(targetPlayer, {
                        sender = player.Name,
                        badge = badge,
                        message = displayMessage,
                        color = tierInfo and tierInfo.color,
                    })
                end
            end
        end
    end
    
    return true
end

-- RemoteFunction สำหรับ VIP chat
local remotes = game.ReplicatedStorage:FindFirstChild("Remotes") or Instance.new("Folder")
remotes.Name = "Remotes"
remotes.Parent = game.ReplicatedStorage

local vipChatRemote = Instance.new("RemoteFunction")
vipChatRemote.Name = "SendVIPMessage"
vipChatRemote.Parent = remotes

vipChatRemote.OnServerInvoke = function(player, message)
    return sendVIPMessage(player, message)
end
```

---

## ส่วนที่ 6: VIP Dashboard UI

```lua
-- LocalScript: VIPDashboard
-- หน้าต่าง Dashboard สำหรับ VIP

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local player = Players.LocalPlayer
local vipData = nil

-- รับข้อมูล VIP จาก server
local remotes = ReplicatedStorage:WaitForChild("Remotes")
local vipEvent = remotes:WaitForChild("VIPUpdate")

vipEvent.OnClientEvent:Connect(function(data)
    vipData = data
    updateDashboard()
end)

-- สร้าง VIP Dashboard
local function createVIPDashboard()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "VIPDashboard"
    screenGui.Parent = player.PlayerGui
    
    -- VIP Status Bar (ด้านบนขวา)
    local statusBar = Instance.new("Frame")
    statusBar.Name = "StatusBar"
    statusBar.Size = UDim2.new(0, 250, 0, 50)
    statusBar.Position = UDim2.new(1, -260, 0, 10)
    statusBar.BackgroundColor3 = Color3.fromRGB(20, 20, 35)
    statusBar.Visible = false -- ซ่อนถ้าไม่มี VIP
    statusBar.Parent = screenGui
    
    local statusCorner = Instance.new("UICorner")
    statusCorner.CornerRadius = UDim.new(0, 10)
    statusCorner.Parent = statusBar
    
    -- VIP badge
    local badgeLabel = Instance.new("TextLabel")
    badgeLabel.Name = "Badge"
    badgeLabel.Size = UDim2.new(0, 50, 1, 0)
    badgeLabel.BackgroundTransparency = 1
    badgeLabel.Text = "👑"
    badgeLabel.TextSize = 24
    badgeLabel.Font = Enum.Font.GothamBold
    badgeLabel.Parent = statusBar
    
    -- VIP tier name
    local tierLabel = Instance.new("TextLabel")
    tierLabel.Name = "TierName"
    tierLabel.Size = UDim2.new(0.6, 0, 0.5, 0)
    tierLabel.Position = UDim2.new(0.2, 0, 0, 0)
    tierLabel.BackgroundTransparency = 1
    tierLabel.Text = "Gold VIP"
    tierLabel.TextColor3 = Color3.fromRGB(255, 215, 0)
    tierLabel.TextSize = 14
    tierLabel.Font = Enum.Font.GothamBold
    tierLabel.Parent = statusBar
    
    -- Daily reward button
    local dailyBtn = Instance.new("TextButton")
    dailyBtn.Name = "DailyBtn"
    dailyBtn.Size = UDim2.new(0.75, 0, 0.5, 0)
    dailyBtn.Position = UDim2.new(0.2, 0, 0.5, 0)
    dailyBtn.BackgroundColor3 = Color3.fromRGB(50, 150, 50)
    dailyBtn.Text = "🎁 Daily Reward"
    dailyBtn.TextColor3 = Color3.new(1, 1, 1)
    dailyBtn.TextSize = 11
    dailyBtn.Font = Enum.Font.Gotham
    dailyBtn.Parent = statusBar
    
    local dailyBtnCorner = Instance.new("UICorner")
    dailyBtnCorner.CornerRadius = UDim.new(0, 5)
    dailyBtnCorner.Parent = dailyBtn
    
    dailyBtn.MouseButton1Click:Connect(function()
        local claimDaily = remotes:WaitForChild("ClaimVIPDaily")
        local success, result = claimDaily:InvokeServer()
        
        if success then
            -- แสดง reward notification
            print("รับรางวัล VIP ประจำวัน: " .. (result.coins or 0) .. " coins")
        end
    end)
    
    return screenGui, statusBar
end

-- อัพเดท dashboard
local function updateDashboard()
    if not vipData then return end
    
    local gui = player.PlayerGui:FindFirstChild("VIPDashboard")
    if not gui then return end
    
    local statusBar = gui:FindFirstChild("StatusBar")
    if statusBar then
        statusBar.Visible = true
        
        local badge = statusBar:FindFirstChild("Badge")
        if badge then badge.Text = vipData.badge or "💎" end
        
        local tierName = statusBar:FindFirstChild("TierName")
        if tierName then
            tierName.Text = vipData.tierName or "VIP"
            tierName.TextColor3 = vipData.color or Color3.new(1, 1, 1)
        end
    end
end

-- สร้าง UI
createVIPDashboard()
```

---

## ส่วนที่ 7: VIP Exclusive Content

```lua
-- Script: VIPExclusiveContent (Script)
-- เนื้อหาพิเศษสำหรับ VIP

local Players = game:GetService("Players")
local VIPManager = require(game.ServerStorage.VIPManager)

-- ให้สัตว์เลี้ยงพิเศษตาม tier
local function grantExclusivePets(player)
    local tierInfo, tier = VIPManager:GetTierInfo(player)
    if not tierInfo or not tierInfo.perks.exclusivePets then return end
    
    local ProfileManager = require(game.ServerStorage.ProfileManager)
    local data = ProfileManager:GetData(player)
    if not data then return end
    
    for _, petId in ipairs(tierInfo.perks.exclusivePets) do
        if not data.Inventory then data.Inventory = {} end
        if not data.Inventory[petId] then
            data.Inventory[petId] = 1
            print(string.format("[VIP] ให้ %s แก่ %s", petId, player.Name))
        end
    end
end

-- Custom name สำหรับ Diamond+ VIP
local function applyCustomName(player, customName)
    local tierInfo, tier = VIPManager:GetTierInfo(player)
    if not tierInfo or not tierInfo.perks.customName then
        return false, "ต้องมี Diamond VIP ขึ้นไป"
    end
    
    local ProfileManager = require(game.ServerStorage.ProfileManager)
    local data = ProfileManager:GetData(player)
    if data then
        data.CustomName = customName
    end
    
    return true
end

-- ใช้งานเมื่อผู้เล่นเข้าสู่เกม
Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function(character)
        task.delay(3, function() -- รอโหลด passes
            grantExclusivePets(player)
            VIPManager:ApplyPerks(player, character)
        end)
    end)
end)
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: VIP Comparison Table
สร้าง UI ที่แสดงตาราง:
1. เปรียบเทียบ Free vs Bronze vs Silver vs Gold
2. Highlight tier ปัจจุบัน
3. ปุ่ม upgrade ถ้า tier ต่ำกว่า

### แบบฝึกหัดที่ 2: VIP Leaderboard
สร้าง leaderboard พิเศษสำหรับ VIP:
1. แสดงแค่ผู้เล่น VIP
2. จัดอันดับตาม tier
3. แสดง badge ของแต่ละคน

### แบบฝึกหัดที่ 3: VIP Event System
สร้าง event พิเศษสำหรับ VIP:
1. เปิด dungeon พิเศษทุกคืน
2. เฉพาะ Gold+ เท่านั้น
3. ให้ legendary loot

---

## สรุปบทที่ 86

VIP System ที่ดีสร้าง value ที่ชัดเจนให้กับผู้เล่น:

1. **หลาย Tier** - ให้ตัวเลือกตามงบประมาณ
2. **Visual Recognition** - ให้ VIP รู้สึกพิเศษ
3. **Exclusive Content** - มีของที่ซื้อไม่ได้จากที่อื่น
4. **Daily Rewards** - สร้าง engagement ต่อเนื่อง
5. **Community Perks** - VIP Chat สร้าง sense of community

*บทถัดไป: Part 87 - In-game Advertising and Sponsorships*
