# Part 84: Developer Products - ระบบสินค้าที่ซื้อได้หลายครั้ง

## บทนำ

Developer Products คือสินค้าที่ผู้เล่นสามารถซื้อได้หลายครั้ง เหมาะสำหรับ consumables เช่น เหรียญ, ยา, บูสต์ชั่วคราว ต่างจาก GamePasses ที่ซื้อครั้งเดียว

## ความแตกต่างระหว่าง Developer Products และ GamePasses

| คุณสมบัติ | Developer Products | GamePasses |
|-----------|-------------------|------------|
| ซื้อซ้ำได้ | ✅ ได้ | ❌ ไม่ได้ |
| ใช้แล้วหมด | ✅ ใช่ | ❌ ถาวร |
| เหมาะสำหรับ | Consumables | Permanent buffs |
| ตรวจสอบ | ไม่ต้องตรวจสอบ ownership | ต้องตรวจสอบ |

---

## ส่วนที่ 1: การสร้าง Developer Product

### 1.1 สร้างใน Creator Dashboard

1. ไปที่ create.roblox.com
2. เลือกเกมของคุณ
3. ไปที่ Monetization > Developer Products
4. คลิก "Create a Developer Product"
5. ตั้งชื่อ, ราคา, รูปภาพ
6. จด Product ID ไว้ใช้ในโค้ด

### 1.2 โครงสร้างพื้นฐาน

```lua
-- Script: DevProductHandler (Script ใน ServerScriptService)
-- ระบบจัดการ Developer Products

local MarketplaceService = game:GetService("MarketplaceService")
local Players = game:GetService("Players")

-- รายการ Product IDs
-- (แทนที่ด้วย ID จริงจาก Creator Dashboard)
local PRODUCT_IDS = {
    -- เหรียญ
    COINS_100 = 1234567801,
    COINS_500 = 1234567802,
    COINS_1000 = 1234567803,
    COINS_5000 = 1234567804,
    
    -- Gems
    GEMS_50 = 1234567811,
    GEMS_200 = 1234567812,
    GEMS_500 = 1234567813,
    
    -- Boosts
    BOOST_EXP_2X_30MIN = 1234567821,
    BOOST_EXP_3X_1HR = 1234567822,
    BOOST_COINS_2X_30MIN = 1234567823,
    
    -- Utilities
    REVIVE_TOKEN = 1234567831,
    EXTRA_LIFE = 1234567832,
    INVENTORY_EXPAND = 1234567833,
    STAT_RESET = 1234567834,
}

-- รายการข้อมูลสินค้า
local PRODUCTS = {
    [PRODUCT_IDS.COINS_100] = {
        name = "100 เหรียญ",
        type = "currency",
        currencyType = "coins",
        amount = 100,
        description = "เหรียญ 100 ชิ้น",
    },
    
    [PRODUCT_IDS.COINS_500] = {
        name = "500 เหรียญ",
        type = "currency",
        currencyType = "coins",
        amount = 500,
        bonus = 50, -- โบนัส 10%
        description = "เหรียญ 500 + โบนัส 50",
    },
    
    [PRODUCT_IDS.COINS_1000] = {
        name = "1,000 เหรียญ",
        type = "currency",
        currencyType = "coins",
        amount = 1000,
        bonus = 150,
        description = "เหรียญ 1,000 + โบนัส 150",
    },
    
    [PRODUCT_IDS.GEMS_50] = {
        name = "50 Gems",
        type = "currency",
        currencyType = "gems",
        amount = 50,
        description = "Gems 50 ชิ้น",
    },
    
    [PRODUCT_IDS.BOOST_EXP_2X_30MIN] = {
        name = "EXP x2 (30 นาที)",
        type = "boost",
        boostType = "exp",
        multiplier = 2,
        duration = 1800,
        description = "เพิ่ม EXP เป็น 2 เท่าเป็นเวลา 30 นาที",
    },
    
    [PRODUCT_IDS.BOOST_EXP_3X_1HR] = {
        name = "EXP x3 (1 ชั่วโมง)",
        type = "boost",
        boostType = "exp",
        multiplier = 3,
        duration = 3600,
        description = "เพิ่ม EXP เป็น 3 เท่าเป็นเวลา 1 ชั่วโมง",
    },
    
    [PRODUCT_IDS.REVIVE_TOKEN] = {
        name = "โทเค็นฟื้นคืนชีพ",
        type = "item",
        itemId = "REVIVE_TOKEN",
        amount = 1,
        description = "ฟื้นคืนชีพในสนามรบ 1 ครั้ง",
    },
    
    [PRODUCT_IDS.INVENTORY_EXPAND] = {
        name = "ขยาย Inventory +10",
        type = "upgrade",
        upgradeType = "inventory",
        amount = 10,
        description = "เพิ่มช่อง inventory อีก 10 ช่อง",
    },
}
```

---

## ส่วนที่ 2: ProcessReceipt Handler

```lua
-- ProcessReceipt คือฟังก์ชันหลักที่ Roblox เรียกเมื่อมีการซื้อ
-- ต้องส่งคืน Enum.ProductPurchaseDecision.PurchaseGranted หรือ NotProcessedYet

local function processReceiptHandler(receiptInfo)
    -- receiptInfo มีข้อมูล:
    -- .PlayerId - UserId ของผู้ซื้อ
    -- .ProductId - ID ของสินค้า
    -- .PurchaseId - ID ของการซื้อ (unique)
    -- .CurrencySpent - Robux ที่ใช้
    
    local player = Players:GetPlayerByUserId(receiptInfo.PlayerId)
    
    -- ถ้าผู้เล่นออกจากเกมแล้ว
    if not player then
        -- ส่งคืน NotProcessedYet เพื่อให้ Roblox ลองใหม่ในครั้งถัดไป
        return Enum.ProductPurchaseDecision.NotProcessedYet
    end
    
    -- ดึงข้อมูลสินค้า
    local product = PRODUCTS[receiptInfo.ProductId]
    
    if not product then
        warn(string.format("[DevProduct] ไม่พบสินค้า ID: %d", receiptInfo.ProductId))
        -- ถ้าไม่รู้จักสินค้า ให้ grant เพื่อไม่ให้ซื้อซ้ำ
        return Enum.ProductPurchaseDecision.PurchaseGranted
    end
    
    -- ดำเนินการตามประเภทสินค้า
    local success = false
    
    if product.type == "currency" then
        success = grantCurrency(player, product)
    elseif product.type == "boost" then
        success = grantBoost(player, product)
    elseif product.type == "item" then
        success = grantItem(player, product)
    elseif product.type == "upgrade" then
        success = grantUpgrade(player, product)
    end
    
    if success then
        -- บันทึกการซื้อ
        logPurchase(player, receiptInfo, product)
        
        -- แจ้งผู้เล่น
        notifyPlayer(player, product)
        
        print(string.format("[DevProduct] ประมวลผลการซื้อสำเร็จ: %s ซื้อ %s",
            player.Name, product.name
        ))
        
        return Enum.ProductPurchaseDecision.PurchaseGranted
    else
        -- ถ้าล้มเหลว รอลองใหม่
        warn(string.format("[DevProduct] ไม่สามารถประมวลผลการซื้อ: %s", product.name))
        return Enum.ProductPurchaseDecision.NotProcessedYet
    end
end

-- ตั้งค่า handler
MarketplaceService.ProcessReceipt = processReceiptHandler
```

---

## ส่วนที่ 3: ฟังก์ชันให้รางวัล

```lua
-- ให้สกุลเงิน
local function grantCurrency(player, product)
    local ProfileManager = require(game.ServerStorage.ProfileManager)
    local data = ProfileManager:GetData(player)
    
    if not data then return false end
    
    local totalAmount = product.amount + (product.bonus or 0)
    
    if product.currencyType == "coins" then
        data.Coins = (data.Coins or 0) + totalAmount
    elseif product.currencyType == "gems" then
        data.Gems = (data.Gems or 0) + totalAmount
    else
        return false
    end
    
    return true
end

-- ให้ boost
local BoostSystem = require(game.ServerStorage.BoostSystem)

local function grantBoost(player, product)
    local ProfileManager = require(game.ServerStorage.ProfileManager)
    local data = ProfileManager:GetData(player)
    
    if not data then return false end
    
    -- ตรวจสอบว่ามี boost อยู่แล้วหรือไม่ (stack หรือรีเซ็ต timer)
    if not data.ActiveBoosts then
        data.ActiveBoosts = {}
    end
    
    local boostKey = product.boostType
    local existingBoost = data.ActiveBoosts[boostKey]
    
    if existingBoost and existingBoost.endTime > os.time() then
        -- มี boost อยู่แล้ว - เพิ่มเวลา
        existingBoost.endTime = existingBoost.endTime + product.duration
        existingBoost.multiplier = math.max(existingBoost.multiplier, product.multiplier)
    else
        -- Boost ใหม่
        data.ActiveBoosts[boostKey] = {
            type = product.boostType,
            multiplier = product.multiplier,
            endTime = os.time() + product.duration,
            productId = product.productId,
        }
    end
    
    -- อัพเดท boost ใน active systems
    BoostSystem:RefreshPlayer(player)
    
    return true
end

-- ให้ไอเทม
local function grantItem(player, product)
    local ProfileManager = require(game.ServerStorage.ProfileManager)
    local data = ProfileManager:GetData(player)
    
    if not data then return false end
    
    if not data.Inventory then
        data.Inventory = {}
    end
    
    local itemId = product.itemId
    local amount = product.amount or 1
    
    data.Inventory[itemId] = (data.Inventory[itemId] or 0) + amount
    
    return true
end

-- ให้การอัพเกรด
local function grantUpgrade(player, product)
    local ProfileManager = require(game.ServerStorage.ProfileManager)
    local data = ProfileManager:GetData(player)
    
    if not data then return false end
    
    if product.upgradeType == "inventory" then
        data.MaxInventorySlots = (data.MaxInventorySlots or 50) + product.amount
    elseif product.upgradeType == "stamina" then
        data.MaxStamina = (data.MaxStamina or 100) + product.amount
    end
    
    return true
end

-- แจ้งผู้เล่น
local function notifyPlayer(player, product)
    local ReplicatedStorage = game:GetService("ReplicatedStorage")
    local remotes = ReplicatedStorage:FindFirstChild("Remotes")
    
    if remotes then
        local purchaseEvent = remotes:FindFirstChild("PurchaseSuccess")
        if purchaseEvent then
            purchaseEvent:FireClient(player, {
                productName = product.name,
                description = product.description,
                type = product.type,
            })
        end
    end
end

-- บันทึกการซื้อ
local function logPurchase(player, receiptInfo, product)
    local DataStoreService = game:GetService("DataStoreService")
    local purchaseLog = DataStoreService:GetDataStore("PurchaseLog")
    
    pcall(function()
        local key = string.format("Purchase_%d_%s",
            receiptInfo.PlayerId,
            receiptInfo.PurchaseId
        )
        
        purchaseLog:SetAsync(key, {
            userId = receiptInfo.PlayerId,
            playerName = player.Name,
            productId = receiptInfo.ProductId,
            productName = product.name,
            purchaseId = receiptInfo.PurchaseId,
            robuxSpent = receiptInfo.CurrencySpent,
            timestamp = os.time(),
        })
    end)
end
```

---

## ส่วนที่ 4: Client-Side Purchase UI

```lua
-- LocalScript: ShopUI (StarterGui)
-- UI ร้านค้า Developer Products

local Players = game:GetService("Players")
local MarketplaceService = game:GetService("MarketplaceService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local player = Players.LocalPlayer
local gui = script.Parent -- หรือ PlayerGui

-- ==============================
-- ข้อมูลสินค้าสำหรับแสดง UI
-- ==============================
local DISPLAY_PRODUCTS = {
    {
        category = "เหรียญ",
        items = {
            {productId = 1234567801, name = "100 เหรียญ", price = 49, icon = "rbxassetid://coin_icon"},
            {productId = 1234567802, name = "550 เหรียญ", price = 199, icon = "rbxassetid://coin_icon", badge = "ยอดนิยม!"},
            {productId = 1234567803, name = "1,150 เหรียญ", price = 349, icon = "rbxassetid://coin_icon"},
            {productId = 1234567804, name = "5,500 เหรียญ", price = 1499, icon = "rbxassetid://coin_icon", badge = "คุ้มที่สุด!"},
        }
    },
    {
        category = "บูสต์",
        items = {
            {productId = 1234567821, name = "EXP x2 30 นาที", price = 50, icon = "rbxassetid://exp_icon"},
            {productId = 1234567822, name = "EXP x3 1 ชั่วโมง", price = 75, icon = "rbxassetid://exp_icon"},
            {productId = 1234567823, name = "Coins x2 30 นาที", price = 50, icon = "rbxassetid://coin_icon"},
        }
    },
    {
        category = "ยูทิลิตี้",
        items = {
            {productId = 1234567831, name = "โทเค็นฟื้นคืนชีพ", price = 25, icon = "rbxassetid://revive_icon"},
            {productId = 1234567833, name = "Inventory +10 ช่อง", price = 99, icon = "rbxassetid://bag_icon"},
        }
    }
}

-- ==============================
-- ฟังก์ชัน UI
-- ==============================

local function createShopItem(itemData, parent)
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(0, 150, 0, 200)
    frame.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
    frame.BorderSizePixel = 0
    frame.Parent = parent
    
    -- Corner radius
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 10)
    corner.Parent = frame
    
    -- ไอคอน
    local icon = Instance.new("ImageLabel")
    icon.Size = UDim2.new(0.7, 0, 0.4, 0)
    icon.Position = UDim2.new(0.15, 0, 0.05, 0)
    icon.BackgroundTransparency = 1
    icon.Image = itemData.icon or ""
    icon.Parent = frame
    
    -- ชื่อสินค้า
    local nameLabel = Instance.new("TextLabel")
    nameLabel.Size = UDim2.new(1, -10, 0, 40)
    nameLabel.Position = UDim2.new(0, 5, 0.48, 0)
    nameLabel.BackgroundTransparency = 1
    nameLabel.Text = itemData.name
    nameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    nameLabel.TextSize = 14
    nameLabel.Font = Enum.Font.GothamBold
    nameLabel.TextWrapped = true
    nameLabel.Parent = frame
    
    -- ราคา
    local priceLabel = Instance.new("TextLabel")
    priceLabel.Size = UDim2.new(1, -10, 0, 25)
    priceLabel.Position = UDim2.new(0, 5, 0.68, 0)
    priceLabel.BackgroundTransparency = 1
    priceLabel.Text = "🟢 " .. itemData.price .. " Robux"
    priceLabel.TextColor3 = Color3.fromRGB(0, 200, 100)
    priceLabel.TextSize = 13
    priceLabel.Font = Enum.Font.Gotham
    priceLabel.Parent = frame
    
    -- ปุ่มซื้อ
    local buyButton = Instance.new("TextButton")
    buyButton.Size = UDim2.new(0.85, 0, 0, 35)
    buyButton.Position = UDim2.new(0.075, 0, 0.82, 0)
    buyButton.BackgroundColor3 = Color3.fromRGB(0, 180, 80)
    buyButton.Text = "ซื้อ"
    buyButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    buyButton.TextSize = 15
    buyButton.Font = Enum.Font.GothamBold
    buyButton.Parent = frame
    
    local buyCorner = Instance.new("UICorner")
    buyCorner.CornerRadius = UDim.new(0, 8)
    buyCorner.Parent = buyButton
    
    -- Badge (ถ้ามี)
    if itemData.badge then
        local badge = Instance.new("TextLabel")
        badge.Size = UDim2.new(0, 80, 0, 22)
        badge.Position = UDim2.new(1, -85, 0, 5)
        badge.BackgroundColor3 = Color3.fromRGB(255, 100, 0)
        badge.Text = itemData.badge
        badge.TextColor3 = Color3.fromRGB(255, 255, 255)
        badge.TextSize = 10
        badge.Font = Enum.Font.GothamBold
        badge.Parent = frame
        
        local badgeCorner = Instance.new("UICorner")
        badgeCorner.CornerRadius = UDim.new(0, 5)
        badgeCorner.Parent = badge
    end
    
    -- Event: ซื้อ
    buyButton.MouseButton1Click:Connect(function()
        -- ยืนยันการซื้อ
        MarketplaceService:PromptProductPurchase(player, itemData.productId)
    end)
    
    -- Animation hover
    buyButton.MouseEnter:Connect(function()
        buyButton.BackgroundColor3 = Color3.fromRGB(0, 220, 100)
    end)
    
    buyButton.MouseLeave:Connect(function()
        buyButton.BackgroundColor3 = Color3.fromRGB(0, 180, 80)
    end)
    
    return frame
end

-- ==============================
-- สร้างหน้าร้านค้า
-- ==============================

local function buildShopUI()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "ShopGui"
    screenGui.Parent = player.PlayerGui
    
    -- พื้นหลัง
    local background = Instance.new("Frame")
    background.Size = UDim2.new(0, 700, 0, 500)
    background.Position = UDim2.new(0.5, -350, 0.5, -250)
    background.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
    background.Parent = screenGui
    
    -- Title
    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, 0, 0, 50)
    title.BackgroundColor3 = Color3.fromRGB(30, 30, 50)
    title.Text = "🛒 ร้านค้า"
    title.TextColor3 = Color3.fromRGB(255, 215, 0)
    title.TextSize = 24
    title.Font = Enum.Font.GothamBold
    title.Parent = background
    
    -- Tab buttons
    local tabContainer = Instance.new("Frame")
    tabContainer.Size = UDim2.new(1, 0, 0, 40)
    tabContainer.Position = UDim2.new(0, 0, 0.1, 0)
    tabContainer.BackgroundTransparency = 1
    tabContainer.Parent = background
    
    -- Content area
    local content = Instance.new("ScrollingFrame")
    content.Size = UDim2.new(1, -20, 0.8, 0)
    content.Position = UDim2.new(0, 10, 0.18, 0)
    content.BackgroundTransparency = 1
    content.Parent = background
    
    -- Grid layout
    local grid = Instance.new("UIGridLayout")
    grid.CellSize = UDim2.new(0, 155, 0, 205)
    grid.CellPadding = UDim2.new(0, 10, 0, 10)
    grid.Parent = content
    
    -- สร้างสินค้า
    for _, category in ipairs(DISPLAY_PRODUCTS) do
        for _, item in ipairs(category.items) do
            createShopItem(item, content)
        end
    end
    
    return screenGui
end

-- รอรับ event จาก server
local remotes = ReplicatedStorage:WaitForChild("Remotes")
local purchaseSuccess = remotes:WaitForChild("PurchaseSuccess")

purchaseSuccess.OnClientEvent:Connect(function(data)
    -- แสดง notification การซื้อสำเร็จ
    local notification = Instance.new("ScreenGui")
    notification.Name = "PurchaseNotification"
    notification.Parent = player.PlayerGui
    
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(0, 300, 0, 80)
    frame.Position = UDim2.new(0.5, -150, 0, 20)
    frame.BackgroundColor3 = Color3.fromRGB(0, 150, 60)
    frame.Parent = notification
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 10)
    corner.Parent = frame
    
    local text = Instance.new("TextLabel")
    text.Size = UDim2.new(1, -20, 1, 0)
    text.Position = UDim2.new(0, 10, 0, 0)
    text.BackgroundTransparency = 1
    text.Text = "✅ ซื้อ " .. data.productName .. " สำเร็จ!"
    text.TextColor3 = Color3.fromRGB(255, 255, 255)
    text.TextSize = 16
    text.Font = Enum.Font.GothamBold
    text.Parent = frame
    
    -- ลบหลัง 3 วินาที
    task.delay(3, function()
        notification:Destroy()
    end)
end)
```

---

## ส่วนที่ 5: Boost System ที่สมบูรณ์

```lua
-- ModuleScript: BoostSystem (ServerStorage)
-- ระบบบูสต์แบบสมบูรณ์

local Players = game:GetService("Players")
local BoostSystem = {}

-- ตรวจสอบว่ามี boost active หรือไม่
function BoostSystem:GetActiveBoost(player, boostType)
    local ProfileManager = require(game.ServerStorage.ProfileManager)
    local data = ProfileManager:GetData(player)
    
    if not data or not data.ActiveBoosts then return nil end
    
    local boost = data.ActiveBoosts[boostType]
    if not boost then return nil end
    
    -- ตรวจสอบว่า boost หมดอายุหรือยัง
    if os.time() > boost.endTime then
        data.ActiveBoosts[boostType] = nil
        return nil
    end
    
    return boost
end

-- คำนวณ multiplier รวม (รวม boost ทั้งหมด)
function BoostSystem:GetMultiplier(player, boostType)
    local boost = self:GetActiveBoost(player, boostType)
    
    -- ตรวจสอบ subscription bonus
    local ProfileManager = require(game.ServerStorage.ProfileManager)
    local data = ProfileManager:GetData(player)
    
    local baseMultiplier = 1
    local boostMultiplier = boost and boost.multiplier or 1
    
    -- เพิ่ม subscription bonus
    if data and data.SubscriptionPerks then
        local perks = data.SubscriptionPerks
        if boostType == "exp" and perks.expMultiplier then
            baseMultiplier = perks.expMultiplier
        elseif boostType == "coins" and perks.coinMultiplier then
            baseMultiplier = perks.coinMultiplier
        end
    end
    
    -- ใช้ค่าสูงสุดระหว่าง base กับ boost
    return math.max(baseMultiplier, boostMultiplier)
end

-- รีเฟรช boost ของผู้เล่น (เรียกหลังซื้อ)
function BoostSystem:RefreshPlayer(player)
    -- ส่ง event ให้ client อัพเดท UI
    local ReplicatedStorage = game:GetService("ReplicatedStorage")
    local remotes = ReplicatedStorage:FindFirstChild("Remotes")
    if remotes then
        local boostEvent = remotes:FindFirstChild("BoostUpdate")
        if boostEvent then
            local ProfileManager = require(game.ServerStorage.ProfileManager)
            local data = ProfileManager:GetData(player)
            if data then
                boostEvent:FireClient(player, data.ActiveBoosts or {})
            end
        end
    end
end

-- ล้าง boost ที่หมดอายุ
function BoostSystem:CleanupExpiredBoosts(player)
    local ProfileManager = require(game.ServerStorage.ProfileManager)
    local data = ProfileManager:GetData(player)
    
    if not data or not data.ActiveBoosts then return end
    
    for boostType, boost in pairs(data.ActiveBoosts) do
        if os.time() > boost.endTime then
            data.ActiveBoosts[boostType] = nil
        end
    end
end

-- ตรวจสอบ boost ทุก 1 นาที
task.spawn(function()
    while true do
        task.wait(60)
        for _, player in ipairs(Players:GetPlayers()) do
            BoostSystem:CleanupExpiredBoosts(player)
        end
    end
end)

return BoostSystem
```

---

## ส่วนที่ 6: Receipt Processing ที่แข็งแกร่ง

```lua
-- การป้องกัน duplicate purchases

local DataStoreService = game:GetService("DataStoreService")
local processedReceipts = DataStoreService:GetDataStore("ProcessedReceipts")

-- ตรวจสอบว่าซื้อซ้ำหรือไม่
local function isReceiptProcessed(purchaseId)
    local success, result = pcall(function()
        return processedReceipts:GetAsync(purchaseId)
    end)
    return success and result ~= nil
end

-- บันทึกว่าประมวลผลแล้ว
local function markReceiptProcessed(purchaseId, userId)
    pcall(function()
        processedReceipts:SetAsync(purchaseId, {
            userId = userId,
            processedAt = os.time()
        })
    end)
end

-- Handler ที่ป้องกัน duplicates
local safeProcessReceipt = function(receiptInfo)
    -- ตรวจสอบ duplicate
    if isReceiptProcessed(receiptInfo.PurchaseId) then
        -- เคยประมวลผลแล้ว - grant ทันที
        return Enum.ProductPurchaseDecision.PurchaseGranted
    end
    
    local player = Players:GetPlayerByUserId(receiptInfo.PlayerId)
    if not player then
        return Enum.ProductPurchaseDecision.NotProcessedYet
    end
    
    -- ประมวลผลปกติ
    local product = PRODUCTS[receiptInfo.ProductId]
    if not product then
        markReceiptProcessed(receiptInfo.PurchaseId, receiptInfo.PlayerId)
        return Enum.ProductPurchaseDecision.PurchaseGranted
    end
    
    local success = false
    if product.type == "currency" then
        success = grantCurrency(player, product)
    elseif product.type == "boost" then
        success = grantBoost(player, product)
    elseif product.type == "item" then
        success = grantItem(player, product)
    elseif product.type == "upgrade" then
        success = grantUpgrade(player, product)
    end
    
    if success then
        markReceiptProcessed(receiptInfo.PurchaseId, receiptInfo.PlayerId)
        logPurchase(player, receiptInfo, product)
        notifyPlayer(player, product)
        return Enum.ProductPurchaseDecision.PurchaseGranted
    end
    
    return Enum.ProductPurchaseDecision.NotProcessedYet
end

MarketplaceService.ProcessReceipt = safeProcessReceipt
```

---

## ส่วนที่ 7: แบบฝึกหัด

### แบบฝึกหัดที่ 1: Coin Pack System
สร้างระบบขายเหรียญที่มี:
1. หลายขนาด (50, 250, 1000, 5000)
2. โบนัสเพิ่มตามปริมาณ
3. Double bonus เมื่อซื้อครั้งแรก
4. UI ที่สวยงาม

### แบบฝึกหัดที่ 2: Boost Shop
สร้าง shop บูสต์ที่:
1. แสดงเวลาที่เหลือของ boost ที่ active
2. ซื้อ stack ได้ (เพิ่มเวลา)
3. มี VIP boost (ถ้ามี VIP pass)
4. แสดงสถิติบูสต์ที่ใช้

### แบบฝึกหัดที่ 3: Transaction History
สร้าง UI แสดงประวัติการซื้อ:
1. รายการซื้อ 30 วันล่าสุด
2. สรุปยอดใช้จ่าย
3. Filter ตามประเภท
4. Export เป็น text

---

## สรุปบทที่ 84

Developer Products เป็นเครื่องมือหลักในการสร้างรายได้แบบ recurring ระบบที่ดีต้องมี:

1. **ProcessReceipt ที่แข็งแกร่ง** - ป้องกัน duplicate, handle errors
2. **UX ที่ดี** - UI ชัดเจน, confirm ก่อนซื้อ
3. **Value ที่ชัดเจน** - ผู้เล่นรู้ว่าได้อะไร
4. **Logging ครบถ้วน** - ติดตามทุก transaction

*บทถัดไป: Part 85 - GamePass System Implementation*
