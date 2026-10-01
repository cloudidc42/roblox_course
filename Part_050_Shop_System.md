# Part 50: ระบบร้านค้า (Shop System) สมบูรณ์

## บทนำ

ระบบร้านค้าเป็นหัวใจของเกมแนว RPG และ Simulator ช่วยให้ผู้เล่นใช้เหรียญที่หามาซื้อไอเทม อาวุธ หรือ upgrade ต่างๆ ในบทนี้เราจะสร้างระบบร้านค้าที่สมบูรณ์แบบ ทั้ง UI สวยงาม และระบบหลังบ้านที่ปลอดภัย

## โครงสร้างระบบร้านค้า

```
ShopSystem/
├── Server/
│   ├── ShopData.lua        - ข้อมูลสินค้า
│   ├── ShopManager.lua     - logic ซื้อขาย
│   └── ShopServer.lua      - RemoteEvent handlers
├── Client/
│   ├── ShopClient.lua      - ส่ง request
│   └── ShopUI.lua          - UI ร้านค้า
└── Shared/
    └── ShopConfig.lua      - การตั้งค่าร่วม
```

## ข้อมูลสินค้า (ShopData)

```lua
-- ServerStorage/Shop/ShopData.lua (ModuleScript)
-- ข้อมูลสินค้าทั้งหมด

local ShopData = {}

-- Categories
ShopData.categories = {
    { id = "weapons", name = "⚔️ อาวุธ", icon = "rbxassetid://111" },
    { id = "armor", name = "🛡️ เกราะ", icon = "rbxassetid://222" },
    { id = "potions", name = "🧪 ยา", icon = "rbxassetid://333" },
    { id = "tools", name = "🔧 เครื่องมือ", icon = "rbxassetid://444" },
    { id = "special", name = "✨ พิเศษ", icon = "rbxassetid://555" }
}

-- Items
ShopData.items = {
    -- ===== อาวุธ =====
    {
        id = "sword_basic",
        name = "ดาบธรรมดา",
        description = "ดาบเหล็กสำหรับมือใหม่\n+15 พลังโจมตี",
        category = "weapons",
        price = 100,
        currency = "coins",
        rarity = "common",
        maxOwn = 1,
        stats = { damage = 15, speed = -0 },
        icon = "rbxassetid://1001",
        previewModel = "SwordBasic"
    },
    {
        id = "sword_iron",
        name = "ดาบเหล็ก",
        description = "ดาบที่ทำจากเหล็กแท้\n+30 พลังโจมตี",
        category = "weapons",
        price = 500,
        currency = "coins",
        rarity = "uncommon",
        requireLevel = 5,
        maxOwn = 1,
        stats = { damage = 30 },
        icon = "rbxassetid://1002"
    },
    {
        id = "sword_diamond",
        name = "ดาบเพชร",
        description = "ดาบที่คมกว่าโกนผม\n+75 พลังโจมตี\n🌟 มีแสงพิเศษ",
        category = "weapons",
        price = 50,
        currency = "gems",  -- ใช้ gems ไม่ใช่ coins
        rarity = "rare",
        requireLevel = 20,
        maxOwn = 1,
        stats = { damage = 75, critChance = 0.1 },
        icon = "rbxassetid://1003"
    },
    
    -- ===== เกราะ =====
    {
        id = "armor_leather",
        name = "เกราะหนัง",
        description = "เกราะทำจากหนังสัตว์\n+10 ป้องกัน",
        category = "armor",
        price = 80,
        currency = "coins",
        rarity = "common",
        maxOwn = 1,
        stats = { defense = 10 },
        icon = "rbxassetid://2001"
    },
    {
        id = "armor_iron",
        name = "เกราะเหล็ก",
        description = "เกราะโลหะมาตรฐาน\n+25 ป้องกัน",
        category = "armor",
        price = 400,
        currency = "coins",
        rarity = "uncommon",
        requireLevel = 8,
        maxOwn = 1,
        stats = { defense = 25 },
        icon = "rbxassetid://2002"
    },
    
    -- ===== ยา =====
    {
        id = "potion_hp_small",
        name = "ยาฟื้นฟูเล็ก",
        description = "ฟื้นฟู HP 25 หน่วย",
        category = "potions",
        price = 20,
        currency = "coins",
        rarity = "common",
        stackable = true,
        maxStack = 99,
        stats = { healAmount = 25 },
        icon = "rbxassetid://3001"
    },
    {
        id = "potion_hp_medium",
        name = "ยาฟื้นฟูกลาง",
        description = "ฟื้นฟู HP 75 หน่วย",
        category = "potions",
        price = 50,
        currency = "coins",
        rarity = "uncommon",
        stackable = true,
        maxStack = 99,
        stats = { healAmount = 75 },
        icon = "rbxassetid://3002"
    },
    {
        id = "elixir_exp",
        name = "ยาเพิ่ม EXP",
        description = "เพิ่ม EXP 2 เท่า นาน 30 นาที\n⏱️ 30 นาที",
        category = "potions",
        price = 10,
        currency = "gems",
        rarity = "rare",
        stackable = true,
        maxStack = 10,
        stats = { expMultiplier = 2, duration = 1800 },
        icon = "rbxassetid://3003"
    },
    
    -- ===== พิเศษ =====
    {
        id = "vip_pass_day",
        name = "VIP 1 วัน",
        description = "สิทธิพิเศษ VIP 1 วัน\n- เหรียญ x2\n- EXP x1.5\n- บินได้!",
        category = "special",
        price = 100,
        currency = "gems",
        rarity = "legendary",
        stackable = false,
        maxOwn = 1,
        icon = "rbxassetid://5001"
    }
}

-- สร้าง lookup table
local itemLookup = {}
for _, item in ipairs(ShopData.items) do
    itemLookup[item.id] = item
end

function ShopData.getItem(id)
    return itemLookup[id]
end

function ShopData.getItemsByCategory(category)
    local results = {}
    for _, item in ipairs(ShopData.items) do
        if item.category == category then
            table.insert(results, item)
        end
    end
    return results
end

function ShopData.getAllItems()
    return ShopData.items
end

return ShopData
```

## Shop Manager (Server Logic)

```lua
-- ServerStorage/Shop/ShopManager.lua (ModuleScript)
local ShopData = require(script.Parent.ShopData)

local ShopManager = {}

-- สมมติว่ามี PlayerDataManager ที่จัดการ data
-- local DataManager = require(...)

-- ตรวจสอบว่าซื้อได้ไหม
function ShopManager.canPurchase(playerData, itemId, quantity)
    quantity = quantity or 1
    
    -- ตรวจสอบ item มีอยู่
    local item = ShopData.getItem(itemId)
    if not item then
        return false, "ไม่พบสินค้า"
    end
    
    -- ตรวจสอบ level requirement
    if item.requireLevel and playerData.level < item.requireLevel then
        return false, string.format("ต้องการ Level %d (คุณ Level %d)", 
            item.requireLevel, playerData.level)
    end
    
    -- ตรวจสอบเหรียญ/gems
    local currency = item.currency or "coins"
    local totalCost = item.price * quantity
    
    if currency == "coins" then
        if playerData.coins < totalCost then
            return false, string.format("เหรียญไม่พอ (ต้องการ %d, มี %d)", 
                totalCost, playerData.coins)
        end
    elseif currency == "gems" then
        if (playerData.gems or 0) < totalCost then
            return false, string.format("Gems ไม่พอ (ต้องการ %d, มี %d)", 
                totalCost, playerData.gems or 0)
        end
    end
    
    -- ตรวจสอบ maxOwn
    if item.maxOwn then
        local owned = ShopManager.countItem(playerData, itemId)
        if owned + quantity > item.maxOwn then
            return false, string.format("สามารถมีได้สูงสุด %d ชิ้น", item.maxOwn)
        end
    end
    
    -- ตรวจสอบ stack size
    if item.stackable then
        local owned = ShopManager.countItem(playerData, itemId)
        local maxStack = item.maxStack or 99
        if owned + quantity > maxStack then
            return false, string.format("ถือได้สูงสุด %d ชิ้น", maxStack)
        end
    end
    
    return true, "ซื้อได้"
end

-- นับจำนวน item ในคลัง
function ShopManager.countItem(playerData, itemId)
    local inventory = playerData.inventory or {}
    
    for _, slot in ipairs(inventory) do
        if slot.id == itemId then
            return slot.quantity or 1
        end
    end
    
    return 0
end

-- ดำเนินการซื้อ
function ShopManager.purchase(playerData, itemId, quantity)
    quantity = quantity or 1
    
    -- ตรวจสอบอีกรอบ
    local canBuy, reason = ShopManager.canPurchase(playerData, itemId, quantity)
    if not canBuy then
        return false, reason
    end
    
    local item = ShopData.getItem(itemId)
    local currency = item.currency or "coins"
    local totalCost = item.price * quantity
    
    -- หักเงิน
    if currency == "coins" then
        playerData.coins = playerData.coins - totalCost
    elseif currency == "gems" then
        playerData.gems = (playerData.gems or 0) - totalCost
    end
    
    -- เพิ่มไอเทม
    ShopManager.addItem(playerData, itemId, quantity)
    
    return true, string.format("ซื้อ %s x%d สำเร็จ!", item.name, quantity)
end

-- เพิ่มไอเทมในคลัง
function ShopManager.addItem(playerData, itemId, quantity)
    if not playerData.inventory then
        playerData.inventory = {}
    end
    
    local item = ShopData.getItem(itemId)
    
    if item and item.stackable then
        -- หา slot ที่มีอยู่แล้ว
        for _, slot in ipairs(playerData.inventory) do
            if slot.id == itemId then
                slot.quantity = (slot.quantity or 0) + quantity
                return
            end
        end
    end
    
    -- สร้าง slot ใหม่
    table.insert(playerData.inventory, {
        id = itemId,
        quantity = quantity,
        acquiredAt = os.time()
    })
end

-- ขายไอเทม
function ShopManager.sell(playerData, itemId, quantity)
    quantity = quantity or 1
    
    local item = ShopData.getItem(itemId)
    if not item then
        return false, "ไม่พบสินค้า"
    end
    
    -- ตรวจสอบว่ามีของพอขาย
    local owned = ShopManager.countItem(playerData, itemId)
    if owned < quantity then
        return false, string.format("ของไม่พอ (มี %d, ต้องการขาย %d)", owned, quantity)
    end
    
    -- ราคาขาย (50% ของราคาซื้อ)
    local sellPrice = math.floor(item.price * 0.5 * quantity)
    
    -- ลบของออกจากคลัง
    ShopManager.removeItem(playerData, itemId, quantity)
    
    -- เพิ่มเหรียญ
    playerData.coins = playerData.coins + sellPrice
    
    return true, string.format("ขาย %s x%d ได้ %d เหรียญ!", item.name, quantity, sellPrice)
end

-- ลบไอเทมจากคลัง
function ShopManager.removeItem(playerData, itemId, quantity)
    for i = #playerData.inventory, 1, -1 do
        local slot = playerData.inventory[i]
        if slot.id == itemId then
            if (slot.quantity or 1) <= quantity then
                table.remove(playerData.inventory, i)
                quantity = quantity - (slot.quantity or 1)
            else
                slot.quantity = slot.quantity - quantity
                quantity = 0
            end
            
            if quantity <= 0 then break end
        end
    end
end

return ShopManager
```

## Shop Server (RemoteEvent Handlers)

```lua
-- ServerScriptService/ShopServer.lua (Script)
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local ServerStorage = game:GetService("ServerStorage")
local Players = game:GetService("Players")

local ShopData = require(ServerStorage.Shop.ShopData)
local ShopManager = require(ServerStorage.Shop.ShopManager)

-- Events and Functions
local Events = ReplicatedStorage:WaitForChild("Events")
local Functions = ReplicatedStorage:WaitForChild("Functions")

local purchaseRE = Events:WaitForChild("Shop_Purchase")
local sellRE = Events:WaitForChild("Shop_Sell")
local notifyRE = Events:WaitForChild("UI_Notification")
local getShopRF = Functions:WaitForChild("GetShopItems")
local checkPurchaseRF = Functions:WaitForChild("CanPurchase")

-- Player data (ในระบบจริงใช้ DataManager)
local playerDatas = {}

Players.PlayerAdded:Connect(function(player)
    playerDatas[player.UserId] = {
        coins = 1000,
        gems = 20,
        level = 10,
        inventory = {}
    }
end)

Players.PlayerRemoving:Connect(function(player)
    playerDatas[player.UserId] = nil
end)

-- ===== Get Shop Items =====
getShopRF.OnServerInvoke = function(player, category)
    local items
    if category and category ~= "all" then
        items = ShopData.getItemsByCategory(category)
    else
        items = ShopData.getAllItems()
    end
    
    -- เพิ่มข้อมูล "owned" สำหรับแต่ละ item
    local playerData = playerDatas[player.UserId]
    local enriched = {}
    
    for _, item in ipairs(items) do
        local owned = playerData and ShopManager.countItem(playerData, item.id) or 0
        
        local enrichedItem = {}
        for k, v in pairs(item) do
            enrichedItem[k] = v
        end
        enrichedItem.owned = owned
        enrichedItem.canAfford = true  -- ตรวจสอบ
        
        if playerData then
            local currency = item.currency or "coins"
            if currency == "coins" then
                enrichedItem.canAfford = playerData.coins >= item.price
            elseif currency == "gems" then
                enrichedItem.canAfford = (playerData.gems or 0) >= item.price
            end
        end
        
        table.insert(enriched, enrichedItem)
    end
    
    return enriched
end

-- ===== Can Purchase Check =====
checkPurchaseRF.OnServerInvoke = function(player, itemId, quantity)
    local playerData = playerDatas[player.UserId]
    if not playerData then
        return false, "ไม่พบข้อมูลผู้เล่น"
    end
    
    return ShopManager.canPurchase(playerData, itemId, quantity or 1)
end

-- ===== Purchase =====
purchaseRE.OnServerEvent:Connect(function(player, itemId, quantity)
    quantity = math.max(1, math.min(quantity or 1, 99))
    
    local playerData = playerDatas[player.UserId]
    if not playerData then return end
    
    local success, message = ShopManager.purchase(playerData, itemId, quantity)
    
    notifyRE:FireClient(player, message, success and "success" or "error")
    
    if success then
        -- อัพเดท leaderstats
        local leaderstats = player:FindFirstChild("leaderstats")
        if leaderstats then
            local coinsVal = leaderstats:FindFirstChild("💰 เหรียญ")
            if coinsVal then
                coinsVal.Value = playerData.coins
            end
        end
        
        print(string.format("[Shop] %s ซื้อ %s x%d", player.Name, itemId, quantity))
    end
end)

-- ===== Sell =====
sellRE.OnServerEvent:Connect(function(player, itemId, quantity)
    quantity = math.max(1, math.min(quantity or 1, 99))
    
    local playerData = playerDatas[player.UserId]
    if not playerData then return end
    
    local success, message = ShopManager.sell(playerData, itemId, quantity)
    
    notifyRE:FireClient(player, message, success and "success" or "error")
    
    if success then
        local leaderstats = player:FindFirstChild("leaderstats")
        if leaderstats then
            local coinsVal = leaderstats:FindFirstChild("💰 เหรียญ")
            if coinsVal then
                coinsVal.Value = playerData.coins
            end
        end
    end
end)
```

## Shop UI (Client)

```lua
-- LocalScript ใน StarterGui/ShopGui
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local playerGui = player.PlayerGui

local Functions = ReplicatedStorage:WaitForChild("Functions")
local Events = ReplicatedStorage:WaitForChild("Events")

local getShopRF = Functions:WaitForChild("GetShopItems")
local canPurchaseRF = Functions:WaitForChild("CanPurchase")
local purchaseRE = Events:WaitForChild("Shop_Purchase")
local sellRE = Events:WaitForChild("Shop_Sell")

-- ===== Shop UI Structure =====
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "ShopGui"
screenGui.ResetOnSpawn = false
screenGui.Parent = playerGui

-- Main Shop Window
local shopWindow = Instance.new("Frame")
shopWindow.Name = "ShopWindow"
shopWindow.Size = UDim2.new(0, 700, 0, 500)
shopWindow.Position = UDim2.new(0.5, -350, 0.5, -250)
shopWindow.BackgroundColor3 = Color3.fromRGB(18, 18, 28)
shopWindow.Visible = false
shopWindow.Parent = screenGui

Instance.new("UICorner", shopWindow).CornerRadius = UDim.new(0, 16)

-- Title
local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, 55)
titleBar.BackgroundColor3 = Color3.fromRGB(255, 140, 0)
Instance.new("UICorner", titleBar).CornerRadius = UDim.new(0, 16)
titleBar.Parent = shopWindow

local shopTitle = Instance.new("TextLabel")
shopTitle.Size = UDim2.new(1, -60, 1, 0)
shopTitle.Position = UDim2.new(0, 20, 0, 0)
shopTitle.Text = "🏪 ร้านค้า"
shopTitle.TextSize = 22
shopTitle.Font = Enum.Font.GothamBold
shopTitle.TextColor3 = Color3.fromRGB(18, 18, 28)
shopTitle.TextXAlignment = Enum.TextXAlignment.Left
shopTitle.BackgroundTransparency = 1
shopTitle.Parent = titleBar

-- Close button
local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 35, 0, 35)
closeBtn.Position = UDim2.new(1, -45, 0, 10)
closeBtn.Text = "✕"
closeBtn.TextSize = 18
closeBtn.Font = Enum.Font.GothamBold
closeBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
closeBtn.TextColor3 = Color3.new(1,1,1)
Instance.new("UICorner", closeBtn).CornerRadius = UDim.new(1,0)
closeBtn.Parent = titleBar

-- Category sidebar
local sidebar = Instance.new("Frame")
sidebar.Size = UDim2.new(0, 150, 1, -65)
sidebar.Position = UDim2.new(0, 0, 0, 60)
sidebar.BackgroundColor3 = Color3.fromRGB(12, 12, 20)
Instance.new("UICorner", sidebar).CornerRadius = UDim.new(0, 12)
sidebar.Parent = shopWindow

local sidebarLayout = Instance.new("UIListLayout")
sidebarLayout.Padding = UDim.new(0, 5)
sidebarLayout.Parent = sidebar

local sidePadding = Instance.new("UIPadding")
sidePadding.PaddingTop = UDim.new(0, 10)
sidePadding.PaddingLeft = UDim.new(0, 8)
sidePadding.PaddingRight = UDim.new(0, 8)
sidePadding.Parent = sidebar

-- Items area
local itemsArea = Instance.new("ScrollingFrame")
itemsArea.Size = UDim2.new(1, -160, 1, -65)
itemsArea.Position = UDim2.new(0, 155, 0, 60)
itemsArea.BackgroundTransparency = 1
itemsArea.ScrollBarThickness = 4
itemsArea.ScrollBarImageColor3 = Color3.fromRGB(255, 140, 0)
itemsArea.Parent = shopWindow

local itemsLayout = Instance.new("UIGridLayout")
itemsLayout.CellSize = UDim2.new(0, 155, 0, 180)
itemsLayout.CellPadding = UDim2.new(0, 8, 0, 8)
itemsLayout.Parent = itemsArea

local itemsPadding = Instance.new("UIPadding")
itemsPadding.PaddingTop = UDim.new(0, 8)
itemsPadding.PaddingLeft = UDim.new(0, 8)
itemsPadding.Parent = itemsArea

-- ===== Functions =====
local selectedCategory = "weapons"
local categoryButtons = {}

local function createCategoryButton(category)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, 0, 0, 40)
    btn.Text = category.name
    btn.TextSize = 12
    btn.Font = Enum.Font.Gotham
    btn.BackgroundColor3 = Color3.fromRGB(25, 25, 40)
    btn.TextColor3 = Color3.new(0.8, 0.8, 0.8)
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 8)
    btn.Parent = sidebar
    
    categoryButtons[category.id] = btn
    
    btn.MouseButton1Click:Connect(function()
        selectedCategory = category.id
        
        -- อัพเดท visual
        for catId, button in pairs(categoryButtons) do
            if catId == selectedCategory then
                button.BackgroundColor3 = Color3.fromRGB(255, 140, 0)
                button.TextColor3 = Color3.fromRGB(18, 18, 28)
            else
                button.BackgroundColor3 = Color3.fromRGB(25, 25, 40)
                button.TextColor3 = Color3.new(0.8, 0.8, 0.8)
            end
        end
        
        loadItems(selectedCategory)
    end)
    
    return btn
end

-- สร้าง category buttons
local categories = {
    { id = "weapons", name = "⚔️ อาวุธ" },
    { id = "armor", name = "🛡️ เกราะ" },
    { id = "potions", name = "🧪 ยา" },
    { id = "special", name = "✨ พิเศษ" }
}

for _, cat in ipairs(categories) do
    createCategoryButton(cat)
end

local rarityColors = {
    common = Color3.fromRGB(150, 150, 150),
    uncommon = Color3.fromRGB(30, 200, 30),
    rare = Color3.fromRGB(30, 100, 255),
    epic = Color3.fromRGB(160, 30, 255),
    legendary = Color3.fromRGB(255, 165, 0)
}

local function createItemCard(item)
    local card = Instance.new("Frame")
    card.BackgroundColor3 = Color3.fromRGB(25, 25, 40)
    Instance.new("UICorner", card).CornerRadius = UDim.new(0, 10)
    
    local rarityColor = rarityColors[item.rarity] or rarityColors.common
    local stroke = Instance.new("UIStroke")
    stroke.Color = rarityColor
    stroke.Thickness = 2
    stroke.Parent = card
    
    -- Icon
    local iconFrame = Instance.new("Frame")
    iconFrame.Size = UDim2.new(1, -16, 0, 80)
    iconFrame.Position = UDim2.new(0, 8, 0, 8)
    iconFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 25)
    Instance.new("UICorner", iconFrame).CornerRadius = UDim.new(0, 8)
    iconFrame.Parent = card
    
    local iconLabel = Instance.new("TextLabel")
    iconLabel.Size = UDim2.new(1, 0, 1, 0)
    iconLabel.Text = "🗡️"  -- placeholder
    iconLabel.TextSize = 40
    iconLabel.BackgroundTransparency = 1
    iconLabel.Parent = iconFrame
    
    -- Item name
    local nameLabel = Instance.new("TextLabel")
    nameLabel.Size = UDim2.new(1, -10, 0, 30)
    nameLabel.Position = UDim2.new(0, 5, 0, 92)
    nameLabel.Text = item.name
    nameLabel.TextSize = 12
    nameLabel.Font = Enum.Font.GothamBold
    nameLabel.TextColor3 = rarityColor
    nameLabel.TextWrapped = true
    nameLabel.BackgroundTransparency = 1
    nameLabel.Parent = card
    
    -- Price
    local priceLabel = Instance.new("TextLabel")
    priceLabel.Size = UDim2.new(1, -10, 0, 20)
    priceLabel.Position = UDim2.new(0, 5, 0, 122)
    priceLabel.Text = (item.currency == "gems" and "💎 " or "💰 ") .. item.price
    priceLabel.TextSize = 13
    priceLabel.Font = Enum.Font.GothamBold
    priceLabel.TextColor3 = item.canAfford and Color3.fromRGB(255, 215, 0) or Color3.fromRGB(200, 50, 50)
    priceLabel.BackgroundTransparency = 1
    priceLabel.Parent = card
    
    -- Buy button
    local buyBtn = Instance.new("TextButton")
    buyBtn.Size = UDim2.new(1, -16, 0, 28)
    buyBtn.Position = UDim2.new(0, 8, 1, -36)
    buyBtn.Text = item.owned and item.owned > 0 and "✅ มีแล้ว" or "ซื้อ"
    buyBtn.TextSize = 13
    buyBtn.Font = Enum.Font.GothamBold
    buyBtn.BackgroundColor3 = item.canAfford and Color3.fromRGB(0, 180, 0) or Color3.fromRGB(80, 80, 80)
    buyBtn.TextColor3 = Color3.new(1,1,1)
    Instance.new("UICorner", buyBtn).CornerRadius = UDim.new(0, 6)
    buyBtn.Parent = card
    
    buyBtn.MouseButton1Click:Connect(function()
        -- ตรวจสอบก่อนซื้อ
        local canBuy, reason = pcall(function()
            return canPurchaseRF:InvokeServer(item.id, 1)
        end)
        
        if canBuy then
            purchaseRE:FireServer(item.id, 1)
            -- รีโหลด items หลังซื้อ
            task.delay(0.5, function()
                loadItems(selectedCategory)
            end)
        else
            print("ซื้อไม่ได้: " .. tostring(reason))
        end
    end)
    
    return card
end

function loadItems(category)
    -- Clear
    for _, child in ipairs(itemsArea:GetChildren()) do
        if child:IsA("Frame") then child:Destroy() end
    end
    
    task.spawn(function()
        local success, items = pcall(function()
            return getShopRF:InvokeServer(category)
        end)
        
        if success and items then
            for _, item in ipairs(items) do
                local card = createItemCard(item)
                card.Parent = itemsArea
            end
            
            -- อัพเดท canvas
            itemsArea.CanvasSize = UDim2.new(0, 0, 0, 
                itemsLayout.AbsoluteContentSize.Y + 20)
        end
    end)
end

-- ===== Open/Close =====
local isOpen = false

local function toggleShop()
    isOpen = not isOpen
    shopWindow.Visible = isOpen
    
    if isOpen then
        loadItems(selectedCategory)
    end
end

-- ปุ่มเปิดร้านค้า (ใน toolbar)
local shopButton = Instance.new("TextButton")
shopButton.Size = UDim2.new(0, 50, 0, 50)
shopButton.Position = UDim2.new(0.5, -25, 1, -60)
shopButton.Text = "🏪"
shopButton.TextSize = 28
shopButton.BackgroundColor3 = Color3.fromRGB(255, 140, 0)
Instance.new("UICorner", shopButton).CornerRadius = UDim.new(1,0)
shopButton.Parent = screenGui

shopButton.MouseButton1Click:Connect(toggleShop)
closeBtn.MouseButton1Click:Connect(toggleShop)
```

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Bundle Deals
เพิ่มระบบ "ซื้อเป็นชุด" ที่ถูกกว่าซื้อทีละชิ้น

### แบบฝึกหัดที่ 2: Limited Time Sales
สร้างระบบ sale/discount ที่มีระยะเวลา

### แบบฝึกหัดที่ 3: Shop Search
เพิ่มช่องค้นหาสินค้าในร้าน

## สรุป

ระบบร้านค้าที่ดีต้องมี:
- **ข้อมูลสินค้าที่ชัดเจน** - ชื่อ, ราคา, คำอธิบาย
- **Validation ที่ปลอดภัย** - ตรวจสอบบน Server เสมอ
- **UI ที่ใช้ง่าย** - หา filter, ราคาชัดเจน
- **Feedback ที่ดี** - แจ้งสำเร็จ/ล้มเหลว
- **ประสบการณ์ที่สนุก** - ทำให้อยากซื้อ!
