# Part 60: Currency System - ระบบสกุลเงินในเกม

## บทนำ

Currency System เป็นระบบพื้นฐานที่ขับเคลื่อนเศรษฐกิจในเกม ระบบที่ดีต้องป้องกัน exploit ป้องกันการ duplication และทำงานได้อย่างน่าเชื่อถือกับ DataStore ในบทนี้เราจะสร้างระบบสกุลเงินหลายประเภทพร้อมระบบ Transaction ที่ปลอดภัย

## สกุลเงินในเกม RPG ทั่วไป

```
Currency Types:
├── Coins (เหรียญ) - สกุลเงินหลัก หาได้จากการเล่น
├── Gems (อัญมณี) - สกุลเงินพรีเมียม
├── Tokens (โทเค็น) - สำหรับ events/seasons
└── Honor Points - สำหรับ PvP/achievements
```

## CurrencyConfig Module

```lua
-- ReplicatedStorage/Shared/CurrencyConfig.lua

local CurrencyConfig = {}

-- ===== Currency Definitions =====
CurrencyConfig.Currencies = {
    ["coins"] = {
        id = "coins",
        name = "Coins",
        nameThai = "เหรียญ",
        icon = "rbxassetid://coin_icon",
        color = Color3.fromRGB(255, 200, 50),
        
        -- Limits
        maxAmount = 999999999,  -- 999 ล้าน
        startAmount = 100,      -- เริ่มต้น
        
        -- ซื้อได้ด้วยเงินจริงไหม
        isPremium = false,
        
        -- DataStore key
        datastoreKey = "coins"
    },
    
    ["gems"] = {
        id = "gems",
        name = "Gems",
        nameThai = "อัญมณี",
        icon = "rbxassetid://gem_icon",
        color = Color3.fromRGB(100, 200, 255),
        
        maxAmount = 99999,
        startAmount = 10,
        
        isPremium = true,       -- ซื้อด้วยเงินจริงได้
        
        datastoreKey = "gems"
    },
    
    ["tokens"] = {
        id = "tokens",
        name = "Event Tokens",
        nameThai = "โทเค็น Event",
        icon = "rbxassetid://token_icon",
        color = Color3.fromRGB(255, 150, 50),
        
        maxAmount = 9999,
        startAmount = 0,
        
        isPremium = false,
        
        -- หมดอายุเมื่อจบ event
        expires = false,
        
        datastoreKey = "tokens"
    },
    
    ["honor"] = {
        id = "honor",
        name = "Honor Points",
        nameThai = "คะแนนเกียรติยศ",
        icon = "rbxassetid://honor_icon",
        color = Color3.fromRGB(200, 100, 255),
        
        maxAmount = 999999,
        startAmount = 0,
        
        isPremium = false,
        
        datastoreKey = "honor"
    }
}

-- ===== Exchange Rates =====
-- ใช้สำหรับ cross-currency trades
CurrencyConfig.ExchangeRates = {
    -- gems -> coins (1 gem = 100 coins)
    ["gems_to_coins"] = {
        from = "gems",
        to = "coins",
        rate = 100,
        minAmount = 1
    }
}

-- ===== Earning Sources =====
CurrencyConfig.EarningSources = {
    -- Coins earning
    kill_common   = {currency = "coins", amount = 5, variance = 3},
    kill_uncommon = {currency = "coins", amount = 15, variance = 5},
    kill_rare     = {currency = "coins", amount = 30, variance = 10},
    kill_boss     = {currency = "coins", amount = 200, variance = 50},
    quest_reward  = {currency = "coins", amount = 500},
    daily_login   = {currency = "coins", amount = 100},
    
    -- Gems earning
    achievement   = {currency = "gems", amount = 5},
    weekly_quest  = {currency = "gems", amount = 10},
    season_pass   = {currency = "gems", amount = 50},
    
    -- Tokens earning
    event_kill    = {currency = "tokens", amount = 1},
    event_quest   = {currency = "tokens", amount = 20}
}

-- ===== Helper =====
function CurrencyConfig.getCurrency(currencyId)
    return CurrencyConfig.Currencies[currencyId]
end

function CurrencyConfig.isValid(currencyId)
    return CurrencyConfig.Currencies[currencyId] ~= nil
end

return CurrencyConfig
```

## CurrencyManager - Server Script

```lua
-- ServerScriptService/CurrencyManager.lua

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local DataStoreService = game:GetService("DataStoreService")

local CurrencyConfig = require(ReplicatedStorage.Shared.CurrencyConfig)

-- ===== Data Store =====
local currencyStore = DataStoreService:GetDataStore("PlayerCurrencyData_v2")

-- ===== Remotes =====
local remotes = ReplicatedStorage:WaitForChild("Remotes")
local currencyUpdateEvent = remotes:WaitForChild("CurrencyUpdate")
local currencyTransactionEvent = remotes:WaitForChild("CurrencyTransaction")
local requestBalanceFunc = remotes:WaitForChild("RequestBalance")

-- ===== Player Data =====
local playerCurrencies = {}
local transactionLocks = {}  -- ป้องกัน concurrent transactions

-- ===== Default Data =====
local function getDefaultCurrencies()
    local defaults = {}
    for currencyId, config in pairs(CurrencyConfig.Currencies) do
        defaults[currencyId] = config.startAmount
    end
    return defaults
end

-- ===== Load/Save =====
local function loadCurrencies(player)
    local success, data = pcall(function()
        return currencyStore:GetAsync("currency_" .. player.UserId)
    end)
    
    if success and data then
        -- Merge กับ defaults (รองรับ currencies ใหม่)
        local defaults = getDefaultCurrencies()
        for currencyId, defaultAmount in pairs(defaults) do
            if data[currencyId] == nil then
                data[currencyId] = defaultAmount
            end
        end
        return data
    else
        return getDefaultCurrencies()
    end
end

local function saveCurrencies(player)
    local data = playerCurrencies[player]
    if not data then return end
    
    local success, err = pcall(function()
        -- ใช้ UpdateAsync เพื่อความปลอดภัย
        currencyStore:UpdateAsync("currency_" .. player.UserId, function(oldData)
            -- รวม old data กับ new data
            -- ถ้าไม่มี old data ใช้ current data
            return data
        end)
    end)
    
    if not success then
        warn("[Currency] Failed to save for " .. player.Name .. ": " .. tostring(err))
    end
end

-- ===== Setup Leaderstats =====
local function updateLeaderstats(player, currencies)
    local leaderstats = player:FindFirstChild("leaderstats")
    if not leaderstats then
        leaderstats = Instance.new("Folder")
        leaderstats.Name = "leaderstats"
        leaderstats.Parent = player
    end
    
    -- Coins ใน leaderstats
    local coinsValue = leaderstats:FindFirstChild("Coins") or Instance.new("NumberValue")
    coinsValue.Name = "Coins"
    coinsValue.Value = currencies.coins or 0
    coinsValue.Parent = leaderstats
    
    -- Gems ใน leaderstats
    local gemsValue = leaderstats:FindFirstChild("Gems") or Instance.new("NumberValue")
    gemsValue.Name = "Gems"
    gemsValue.Value = currencies.gems or 0
    gemsValue.Parent = leaderstats
end

-- ===== Get Balance =====
local function getBalance(player, currencyId)
    local currencies = playerCurrencies[player]
    if not currencies then return 0 end
    return currencies[currencyId] or 0
end

-- ===== Add Currency =====
local function addCurrency(player, currencyId, amount, source)
    -- Validate inputs
    if not player or not player.Parent then return false, "INVALID_PLAYER" end
    if not CurrencyConfig.isValid(currencyId) then return false, "INVALID_CURRENCY" end
    if type(amount) ~= "number" then return false, "INVALID_AMOUNT" end
    if amount <= 0 then return false, "NEGATIVE_AMOUNT" end
    
    local currencies = playerCurrencies[player]
    if not currencies then return false, "PLAYER_NOT_LOADED" end
    
    local config = CurrencyConfig.getCurrency(currencyId)
    local current = currencies[currencyId] or 0
    
    -- ตรวจสอบ max
    local newAmount = math.min(current + amount, config.maxAmount)
    local actualAdded = newAmount - current
    
    currencies[currencyId] = newAmount
    
    -- อัพเดท leaderstats
    updateLeaderstats(player, currencies)
    
    -- แจ้ง client
    currencyUpdateEvent:FireClient(player, {
        type = "Add",
        currencyId = currencyId,
        amount = actualAdded,
        newBalance = newAmount,
        source = source or "Unknown"
    })
    
    print(string.format("[Currency] +%d %s for %s from %s (balance: %d)",
        actualAdded, currencyId, player.Name, source or "Unknown", newAmount))
    
    return true, actualAdded
end

-- ===== Remove Currency (Spend) =====
local function removeCurrency(player, currencyId, amount, reason)
    -- Validate
    if not player or not player.Parent then return false, "INVALID_PLAYER" end
    if not CurrencyConfig.isValid(currencyId) then return false, "INVALID_CURRENCY" end
    if type(amount) ~= "number" or amount <= 0 then return false, "INVALID_AMOUNT" end
    
    local currencies = playerCurrencies[player]
    if not currencies then return false, "PLAYER_NOT_LOADED" end
    
    local current = currencies[currencyId] or 0
    
    -- ตรวจสอบว่ามีพอไหม
    if current < amount then
        currencyUpdateEvent:FireClient(player, {
            type = "InsufficientFunds",
            currencyId = currencyId,
            required = amount,
            balance = current
        })
        return false, "INSUFFICIENT_FUNDS"
    end
    
    -- หัก currency
    local newAmount = current - amount
    currencies[currencyId] = newAmount
    
    -- อัพเดท leaderstats
    updateLeaderstats(player, currencies)
    
    -- แจ้ง client
    currencyUpdateEvent:FireClient(player, {
        type = "Remove",
        currencyId = currencyId,
        amount = amount,
        newBalance = newAmount,
        reason = reason or "Unknown"
    })
    
    print(string.format("[Currency] -%d %s for %s (reason: %s, balance: %d)",
        amount, currencyId, player.Name, reason or "Unknown", newAmount))
    
    return true
end

-- ===== Transfer Currency (ส่งให้ผู้เล่นอื่น) =====
local TRANSFER_COOLDOWN = 5  -- วินาที
local transferCooldowns = {}

local function transferCurrency(fromPlayer, toPlayer, currencyId, amount)
    -- ตรวจสอบ rate limiting
    local now = os.clock()
    local lastTransfer = transferCooldowns[fromPlayer]
    if lastTransfer and now - lastTransfer < TRANSFER_COOLDOWN then
        return false, "TRANSFER_COOLDOWN"
    end
    
    -- ตรวจสอบ currency
    local config = CurrencyConfig.getCurrency(currencyId)
    if not config then return false, "INVALID_CURRENCY" end
    
    -- ห้ามส่ง premium currency
    if config.isPremium then
        return false, "PREMIUM_NOT_TRANSFERABLE"
    end
    
    -- ตรวจสอบจำนวน
    if amount < 1 or amount > 100000 then
        return false, "INVALID_AMOUNT"
    end
    
    -- ตรวจสอบว่าผู้รับ online
    if not toPlayer.Parent then
        return false, "PLAYER_OFFLINE"
    end
    
    -- ดำเนินการ transfer
    local success, err = removeCurrency(fromPlayer, currencyId, amount, "Transfer to " .. toPlayer.Name)
    if not success then return false, err end
    
    addCurrency(toPlayer, currencyId, amount, "Transfer from " .. fromPlayer.Name)
    
    transferCooldowns[fromPlayer] = now
    
    print(string.format("[Currency] %s transferred %d %s to %s",
        fromPlayer.Name, amount, currencyId, toPlayer.Name))
    
    return true
end

-- ===== Transaction Lock (ป้องกัน race condition) =====
local function withLock(player, callback)
    local userId = player.UserId
    
    -- รอ lock ว่าง
    while transactionLocks[userId] do
        task.wait(0.05)
    end
    
    transactionLocks[userId] = true
    local success, result = pcall(callback)
    transactionLocks[userId] = nil
    
    if not success then
        warn("[Currency] Transaction error: " .. tostring(result))
        return false
    end
    
    return result
end

-- ===== Currency Exchange =====
local function exchangeCurrency(player, fromId, toId, amount)
    return withLock(player, function()
        -- หา exchange rate
        local rateKey = fromId .. "_to_" .. toId
        local rate = CurrencyConfig.ExchangeRates[rateKey]
        
        if not rate then
            return false, "NO_EXCHANGE_RATE"
        end
        
        if amount < rate.minAmount then
            return false, "BELOW_MINIMUM"
        end
        
        -- คำนวณ
        local receiveAmount = amount * rate.rate
        
        -- ทำ exchange
        local success, err = removeCurrency(player, fromId, amount, "Exchange to " .. toId)
        if not success then return false, err end
        
        addCurrency(player, toId, receiveAmount, "Exchange from " .. fromId)
        
        return true, receiveAmount
    end)
end

-- ===== Daily Login Reward =====
local loginStore = DataStoreService:GetDataStore("DailyLoginData")

local function checkDailyLogin(player)
    local userId = player.UserId
    
    local success, lastLogin = pcall(function()
        return loginStore:GetAsync("login_" .. userId)
    end)
    
    if not success then return end
    
    local today = os.date("%Y-%m-%d")
    
    if lastLogin ~= today then
        -- ให้ daily reward
        addCurrency(player, "coins", CurrencyConfig.EarningSources.daily_login.amount, "Daily Login")
        
        -- บันทึก
        pcall(function()
            loginStore:SetAsync("login_" .. userId, today)
        end)
        
        -- แจ้ง client
        currencyUpdateEvent:FireClient(player, {
            type = "DailyReward",
            message = "Daily Login Reward! +" .. CurrencyConfig.EarningSources.daily_login.amount .. " Coins"
        })
        
        print("[Currency] Daily reward for " .. player.Name)
    end
end

-- ===== Drop Coins from Enemy =====
local function dropCoinsAtPosition(position, amount, source)
    -- สร้าง coin pickup ที่ตำแหน่งนั้น
    local coin = Instance.new("Part")
    coin.Name = "CoinDrop"
    coin.Size = Vector3.new(1, 0.3, 1)
    coin.Position = position + Vector3.new(
        math.random(-2, 2),
        1,
        math.random(-2, 2)
    )
    coin.Anchored = false
    coin.CanCollide = true
    coin.Material = Enum.Material.SmoothPlastic
    coin.BrickColor = BrickColor.new("Bright yellow")
    coin.Shape = Enum.PartType.Cylinder
    coin.Parent = workspace
    
    -- เก็บข้อมูลจำนวน coins
    local coinTag = Instance.new("NumberValue")
    coinTag.Name = "CoinAmount"
    coinTag.Value = amount
    coinTag.Parent = coin
    
    -- ProximityPrompt สำหรับเก็บ
    local prompt = Instance.new("ProximityPrompt")
    prompt.ActionText = "เก็บ " .. amount .. " Coins"
    prompt.ObjectText = "Coins"
    prompt.HoldDuration = 0
    prompt.MaxActivationDistance = 8
    prompt.Parent = coin
    
    prompt.Triggered:Connect(function(player)
        if not coin.Parent then return end
        
        local coinAmount = coinTag.Value
        
        -- ลบ coin ก่อน (ป้องกัน double pickup)
        coin:Destroy()
        
        -- ให้ coins
        addCurrency(player, "coins", coinAmount, source or "CoinDrop")
    end)
    
    -- Auto destroy หลัง 30 วินาที
    game:GetService("Debris"):AddItem(coin, 30)
    
    return coin
end

-- ===== Event Handlers =====

-- Request Balance
requestBalanceFunc.OnServerInvoke = function(player, currencyId)
    if currencyId then
        return getBalance(player, currencyId)
    else
        -- ส่งทุก balances
        return playerCurrencies[player] or {}
    end
end

-- Currency Transaction (ซื้อของ)
currencyTransactionEvent.OnServerEvent:Connect(function(player, transactionData)
    if not transactionData then return end
    
    -- Rate limiting
    -- (ใช้ lock เพื่อป้องกัน concurrent)
    withLock(player, function()
        if transactionData.type == "purchase" then
            -- ซื้อของ (validate ใน ShopManager)
            -- ShopManager จะเรียก removeCurrency โดยตรง
            
        elseif transactionData.type == "exchange" then
            local success, result = exchangeCurrency(
                player,
                transactionData.from,
                transactionData.to,
                transactionData.amount
            )
            
            currencyUpdateEvent:FireClient(player, {
                type = "ExchangeResult",
                success = success,
                result = result
            })
        end
    end)
end)

-- ===== Player Lifecycle =====
Players.PlayerAdded:Connect(function(player)
    local currencies = loadCurrencies(player)
    playerCurrencies[player] = currencies
    
    -- Setup leaderstats
    updateLeaderstats(player, currencies)
    
    -- ส่งข้อมูลเริ่มต้นไปยัง client
    player.CharacterAdded:Connect(function()
        task.wait(0.5)
        currencyUpdateEvent:FireClient(player, {
            type = "Initialize",
            currencies = currencies
        })
        
        -- ตรวจสอบ daily login
        checkDailyLogin(player)
    end)
end)

Players.PlayerRemoving:Connect(function(player)
    saveCurrencies(player)
    playerCurrencies[player] = nil
    transferCooldowns[player] = nil
end)

-- Auto-save ทุก 2 นาที
task.spawn(function()
    while true do
        task.wait(120)
        for _, player in ipairs(Players:GetPlayers()) do
            saveCurrencies(player)
        end
        print("[Currency] Auto-saved all players")
    end
end)

-- ===== Public API =====
local CurrencyManager = {}

function CurrencyManager.getBalance(player, currencyId)
    return getBalance(player, currencyId)
end

function CurrencyManager.addCurrency(player, currencyId, amount, source)
    return addCurrency(player, currencyId, amount, source)
end

function CurrencyManager.removeCurrency(player, currencyId, amount, reason)
    return removeCurrency(player, currencyId, amount, reason)
end

function CurrencyManager.canAfford(player, currencyId, amount)
    return getBalance(player, currencyId) >= amount
end

function CurrencyManager.dropCoins(position, amount, source)
    return dropCoinsAtPosition(position, amount, source)
end

function CurrencyManager.giveKillReward(player, enemyRarity)
    local source = CurrencyConfig.EarningSources["kill_" .. (enemyRarity or "common")]
    if source then
        local variance = source.variance or 0
        local amount = source.amount + math.random(-variance, variance)
        return addCurrency(player, source.currency, math.max(1, amount), "KillReward")
    end
end

return CurrencyManager
```

## CurrencyUI - Client Script

```lua
-- StarterPlayerScripts/CurrencyUI.lua

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local CurrencyConfig = require(ReplicatedStorage.Shared.CurrencyConfig)

-- ===== Remotes =====
local remotes = ReplicatedStorage:WaitForChild("Remotes")
local currencyUpdateEvent = remotes:WaitForChild("CurrencyUpdate")

-- ===== Local Currency Cache =====
local localCurrencies = {}

-- ===== Create Currency Display =====
local function createCurrencyDisplay()
    local gui = Instance.new("ScreenGui")
    gui.Name = "CurrencyUI"
    gui.ResetOnSpawn = false
    gui.Parent = player.PlayerGui
    
    -- Currency display (บนขวา)
    local frame = Instance.new("Frame")
    frame.Name = "CurrencyDisplay"
    frame.Size = UDim2.new(0, 180, 0, 120)
    frame.Position = UDim2.new(1, -196, 0, 16)
    frame.BackgroundColor3 = Color3.fromRGB(15, 15, 25)
    frame.BackgroundTransparency = 0.2
    frame.BorderSizePixel = 0
    frame.Parent = gui
    
    Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 10)
    
    -- Layout
    local layout = Instance.new("UIListLayout")
    layout.FillDirection = Enum.FillDirection.Vertical
    layout.Padding = UDim.new(0, 4)
    layout.Parent = frame
    
    local padding = Instance.new("UIPadding")
    padding.PaddingAll = UDim.new(0, 8)
    padding.Parent = frame
    
    -- สร้าง row สำหรับแต่ละ currency
    local displayOrder = {"coins", "gems", "tokens"}
    
    for _, currencyId in ipairs(displayOrder) do
        local config = CurrencyConfig.getCurrency(currencyId)
        if not config then continue end
        
        local row = Instance.new("Frame")
        row.Name = currencyId .. "Row"
        row.Size = UDim2.new(1, 0, 0, 28)
        row.BackgroundTransparency = 1
        row.Parent = frame
        
        -- Icon placeholder
        local iconFrame = Instance.new("Frame")
        iconFrame.Size = UDim2.new(0, 22, 0, 22)
        iconFrame.Position = UDim2.new(0, 0, 0.5, -11)
        iconFrame.BackgroundColor3 = config.color
        iconFrame.BorderSizePixel = 0
        iconFrame.Parent = row
        
        Instance.new("UICorner", iconFrame).CornerRadius = UDim.new(0, 5)
        
        local iconText = Instance.new("TextLabel")
        iconText.Size = UDim2.new(1, 0, 1, 0)
        iconText.BackgroundTransparency = 1
        iconText.Text = currencyId == "coins" and "G" or currencyId == "gems" and "💎" or "T"
        iconText.Font = Enum.Font.GothamBold
        iconText.TextSize = 12
        iconText.TextColor3 = Color3.new(0, 0, 0)
        iconText.Parent = iconFrame
        
        -- Amount label
        local amountLabel = Instance.new("TextLabel")
        amountLabel.Name = "Amount"
        amountLabel.Size = UDim2.new(1, -30, 1, 0)
        amountLabel.Position = UDim2.new(0, 28, 0, 0)
        amountLabel.BackgroundTransparency = 1
        amountLabel.Text = "0"
        amountLabel.Font = Enum.Font.GothamBold
        amountLabel.TextSize = 16
        amountLabel.TextColor3 = config.color
        amountLabel.TextXAlignment = Enum.TextXAlignment.Left
        amountLabel.Parent = row
    end
    
    return gui
end

-- ===== Format Number =====
local function formatNumber(n)
    if n >= 1000000 then
        return string.format("%.1fM", n / 1000000)
    elseif n >= 1000 then
        return string.format("%.1fK", n / 1000)
    else
        return tostring(math.floor(n))
    end
end

-- ===== Update Currency Display =====
local currencyGui = createCurrencyDisplay()

local function updateCurrencyDisplay(currencyId, newBalance)
    local display = currencyGui:FindFirstChild("CurrencyDisplay")
    if not display then return end
    
    local row = display:FindFirstChild(currencyId .. "Row")
    if not row then return end
    
    local amountLabel = row:FindFirstChild("Amount")
    if not amountLabel then return end
    
    local config = CurrencyConfig.getCurrency(currencyId)
    if not config then return end
    
    -- Animate number change
    local oldText = amountLabel.Text
    amountLabel.Text = formatNumber(newBalance)
    
    -- Flash effect
    TweenService:Create(amountLabel, TweenInfo.new(0.1), {
        TextColor3 = Color3.new(1, 1, 1)
    }):Play()
    
    task.delay(0.2, function()
        TweenService:Create(amountLabel, TweenInfo.new(0.3), {
            TextColor3 = config.color
        }):Play()
    end)
end

-- ===== Currency Gain Notification =====
local function showCurrencyGain(currencyId, amount, source)
    local config = CurrencyConfig.getCurrency(currencyId)
    if not config then return end
    
    local display = currencyGui:FindFirstChild("CurrencyDisplay")
    if not display then return end
    
    -- Floating notification
    local notif = Instance.new("TextLabel")
    notif.Size = UDim2.new(0, 150, 0, 30)
    notif.AnchorPoint = Vector2.new(1, 0)
    notif.Position = UDim2.new(1, -16, 0, 140)
    notif.BackgroundTransparency = 1
    notif.Text = "+" .. formatNumber(amount) .. " " .. (config.nameThai or config.name)
    notif.Font = Enum.Font.GothamBold
    notif.TextSize = 16
    notif.TextColor3 = config.color
    notif.TextStrokeTransparency = 0
    notif.TextStrokeColor3 = Color3.new(0, 0, 0)
    notif.TextXAlignment = Enum.TextXAlignment.Right
    notif.ZIndex = 10
    notif.Parent = currencyGui
    
    -- Animate
    TweenService:Create(notif, TweenInfo.new(1.5, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
        Position = UDim2.new(1, -16, 0, 110),
        TextTransparency = 1,
        TextStrokeTransparency = 1
    }):Play()
    
    game:GetService("Debris"):AddItem(notif, 1.5)
end

-- ===== Insufficient Funds Notification =====
local function showInsufficientFunds(currencyId, required, balance)
    local config = CurrencyConfig.getCurrency(currencyId)
    local name = config and (config.nameThai or config.name) or currencyId
    
    local notif = Instance.new("TextLabel")
    notif.Size = UDim2.new(0, 250, 0, 35)
    notif.AnchorPoint = Vector2.new(0.5, 0)
    notif.Position = UDim2.new(0.5, 0, 0.4, 0)
    notif.BackgroundColor3 = Color3.fromRGB(180, 30, 30)
    notif.BackgroundTransparency = 0.2
    notif.BorderSizePixel = 0
    notif.Text = name .. " ไม่พอ! ต้องการ " .. formatNumber(required)
    notif.Font = Enum.Font.GothamBold
    notif.TextSize = 14
    notif.TextColor3 = Color3.new(1, 1, 1)
    notif.ZIndex = 20
    notif.Parent = currencyGui
    
    Instance.new("UICorner", notif).CornerRadius = UDim.new(0, 8)
    
    -- Shake animation
    task.spawn(function()
        for i = 1, 4 do
            TweenService:Create(notif, TweenInfo.new(0.05), {
                Position = UDim2.new(0.5, 5, 0.4, 0)
            }):Play()
            task.wait(0.05)
            TweenService:Create(notif, TweenInfo.new(0.05), {
                Position = UDim2.new(0.5, -5, 0.4, 0)
            }):Play()
            task.wait(0.05)
        end
        TweenService:Create(notif, TweenInfo.new(0.05), {
            Position = UDim2.new(0.5, 0, 0.4, 0)
        }):Play()
    end)
    
    -- Fade out
    task.delay(2, function()
        TweenService:Create(notif, TweenInfo.new(0.5), {
            BackgroundTransparency = 1,
            TextTransparency = 1
        }):Play()
        game:GetService("Debris"):AddItem(notif, 0.5)
    end)
end

-- ===== Daily Reward Popup =====
local function showDailyReward(message)
    local popup = Instance.new("Frame")
    popup.Size = UDim2.new(0, 300, 0, 150)
    popup.AnchorPoint = Vector2.new(0.5, 0.5)
    popup.Position = UDim2.new(0.5, 0, 0.5, 0)
    popup.BackgroundColor3 = Color3.fromRGB(20, 20, 40)
    popup.BackgroundTransparency = 0
    popup.BorderSizePixel = 0
    popup.ZIndex = 30
    popup.Parent = currencyGui
    
    Instance.new("UICorner", popup).CornerRadius = UDim.new(0, 15)
    
    -- Border glow
    local border = Instance.new("UIStroke")
    border.Color = Color3.fromRGB(255, 200, 0)
    border.Thickness = 2
    border.Parent = popup
    
    -- Icon
    local icon = Instance.new("TextLabel")
    icon.Size = UDim2.new(0, 60, 0, 60)
    icon.Position = UDim2.new(0.5, -30, 0, 15)
    icon.BackgroundColor3 = Color3.fromRGB(255, 180, 0)
    icon.BorderSizePixel = 0
    icon.Text = "🎁"
    icon.Font = Enum.Font.GothamBold
    icon.TextSize = 30
    icon.TextColor3 = Color3.new(1, 1, 1)
    icon.ZIndex = 31
    icon.Parent = popup
    
    Instance.new("UICorner", icon).CornerRadius = UDim.new(0.5, 0)
    
    local titleLabel = Instance.new("TextLabel")
    titleLabel.Size = UDim2.new(1, -20, 0, 25)
    titleLabel.Position = UDim2.new(0, 10, 0, 80)
    titleLabel.BackgroundTransparency = 1
    titleLabel.Text = "Daily Login Reward!"
    titleLabel.Font = Enum.Font.GothamBold
    titleLabel.TextSize = 18
    titleLabel.TextColor3 = Color3.fromRGB(255, 215, 0)
    titleLabel.ZIndex = 31
    titleLabel.Parent = popup
    
    local msgLabel = Instance.new("TextLabel")
    msgLabel.Size = UDim2.new(1, -20, 0, 25)
    msgLabel.Position = UDim2.new(0, 10, 0, 108)
    msgLabel.BackgroundTransparency = 1
    msgLabel.Text = message
    msgLabel.Font = Enum.Font.Gotham
    msgLabel.TextSize = 13
    msgLabel.TextColor3 = Color3.new(1, 1, 1)
    msgLabel.ZIndex = 31
    msgLabel.Parent = popup
    
    -- Entrance animation
    popup.Position = UDim2.new(0.5, 0, -0.2, 0)
    TweenService:Create(popup, TweenInfo.new(0.6, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
        Position = UDim2.new(0.5, 0, 0.5, 0)
    }):Play()
    
    -- Auto close
    task.delay(3, function()
        TweenService:Create(popup, TweenInfo.new(0.4), {
            Position = UDim2.new(0.5, 0, 1.2, 0),
            BackgroundTransparency = 1
        }):Play()
        game:GetService("Debris"):AddItem(popup, 0.4)
    end)
end

-- ===== Event Handler =====
currencyUpdateEvent.OnClientEvent:Connect(function(data)
    if data.type == "Initialize" then
        -- ตั้งค่าเริ่มต้น
        for currencyId, balance in pairs(data.currencies) do
            localCurrencies[currencyId] = balance
            updateCurrencyDisplay(currencyId, balance)
        end
        
    elseif data.type == "Add" then
        localCurrencies[data.currencyId] = data.newBalance
        updateCurrencyDisplay(data.currencyId, data.newBalance)
        showCurrencyGain(data.currencyId, data.amount, data.source)
        
    elseif data.type == "Remove" then
        localCurrencies[data.currencyId] = data.newBalance
        updateCurrencyDisplay(data.currencyId, data.newBalance)
        
    elseif data.type == "InsufficientFunds" then
        showInsufficientFunds(data.currencyId, data.required, data.balance)
        
    elseif data.type == "DailyReward" then
        showDailyReward(data.message)
        
    elseif data.type == "ExchangeResult" then
        if data.success then
            print("[Currency] Exchange successful: received " .. tostring(data.result))
        else
            print("[Currency] Exchange failed: " .. tostring(data.result))
        end
    end
end)

-- ===== Public Functions =====
local CurrencyUI = {}

function CurrencyUI.getBalance(currencyId)
    return localCurrencies[currencyId] or 0
end

function CurrencyUI.canAfford(currencyId, amount)
    return (localCurrencies[currencyId] or 0) >= amount
end

return CurrencyUI
```

## Anti-Exploit Patterns

```lua
-- ตัวอย่างการป้องกัน Currency Exploit

-- ===== 1. Server-Side Validation =====
-- ห้ามให้ client คำนวณ currency เอง

-- BAD (อันตราย):
-- Client: "ฉันได้ 1000 coins จากการ kill"
-- Server: ตกลง เพิ่ม 1000 coins

-- GOOD (ปลอดภัย):
-- Client: "ฉัน kill enemy ID 12345"
-- Server: ตรวจสอบ kill จริง คำนวณ reward เอง แล้วให้

-- ===== 2. Rate Limiting =====
local currencyRateLimit = {}

local function canReceiveCurrency(player, source)
    local userId = player.UserId
    local now = os.clock()
    
    if not currencyRateLimit[userId] then
        currencyRateLimit[userId] = {}
    end
    
    local limits = currencyRateLimit[userId]
    local sourceLimit = limits[source]
    
    -- ตัวอย่าง: ได้ coins จาก kill ได้สูงสุด 10 ครั้ง/วินาที
    local LIMITS = {
        kill = {maxPerSecond = 10, window = 1},
        daily_login = {maxPerDay = 1, window = 86400}
    }
    
    local limit = LIMITS[source]
    if not limit then return true end
    
    if not sourceLimit then
        limits[source] = {count = 1, windowStart = now}
        return true
    end
    
    -- Reset window ถ้าหมดเวลา
    if now - sourceLimit.windowStart > (limit.window or 1) then
        limits[source] = {count = 1, windowStart = now}
        return true
    end
    
    -- ตรวจสอบ limit
    local maxCount = limit.maxPerSecond or limit.maxPerDay or 999
    if sourceLimit.count >= maxCount then
        warn("[AntiExploit] Rate limit exceeded for " .. player.Name .. " source: " .. source)
        return false
    end
    
    sourceLimit.count = sourceLimit.count + 1
    return true
end

-- ===== 3. Sanity Checks =====
local function validateCurrencyAmount(amount, source)
    -- ตรวจสอบว่า amount สมเหตุสมผล
    local MAX_AMOUNTS = {
        kill = 1000,          -- kill reward ไม่เกิน 1000
        quest = 10000,        -- quest reward ไม่เกิน 10000
        default = 100000      -- default max
    }
    
    local max = MAX_AMOUNTS[source] or MAX_AMOUNTS.default
    
    if amount <= 0 then return false, "ZERO_OR_NEGATIVE" end
    if amount > max then return false, "EXCEEDS_MAX" end
    if amount ~= math.floor(amount) then return false, "NOT_INTEGER" end
    
    return true
end

-- ===== 4. Transaction Logging =====
local function logTransaction(player, currencyId, amount, reason, success)
    -- บันทึกทุก transaction เพื่อ audit
    local log = {
        userId = player.UserId,
        username = player.Name,
        currency = currencyId,
        amount = amount,
        reason = reason,
        success = success,
        timestamp = os.time(),
        balance = getBalance(player, currencyId)
    }
    
    -- ในระบบจริงควร save ลง DataStore หรือ external logging service
    print(string.format("[TX] User:%s Currency:%s Amount:%d Reason:%s Success:%s",
        player.Name, currencyId, amount, reason, tostring(success)))
end
```

## Currency Economy Design

```lua
-- ===== ตัวอย่างการ balance Economy =====

--[[
ECONOMY DESIGN PRINCIPLES:

1. CURRENCY SINKS (ทำให้ currency มีค่า):
   - ซื้อ items
   - Upgrade weapons
   - Fast travel
   - Crafting materials
   - Respawn costs (optional)

2. CURRENCY SOURCES (หาได้จากที่ไหน):
   - Kill enemies
   - Complete quests
   - Daily login
   - Selling items
   - Achievements

3. BALANCE FORMULA:
   - Average earn per hour ≈ ราคาไอเทมกลาง
   - ไม่ให้ผู้เล่นรวยเกินไปเร็วเกินไป
   - ไม่ให้ผู้เล่นรู้สึกว่าหาไม่พอ

4. INFLATION PREVENTION:
   - มี currency sinks เพียงพอ
   - จำกัด max currency ที่เก็บได้
   - ทำให้ items มีความต้องการ

]]

-- ===== Currency Balance Table (ตัวอย่าง) =====
local ECONOMY_BALANCE = {
    -- ราคา items
    prices = {
        common_sword = 100,        -- หาได้ใน 5-10 นาที
        rare_sword = 1000,         -- หาได้ใน 50-100 นาที
        legendary_armor = 50000,   -- หาได้ใน สัปดาห์
    },
    
    -- Earn rates (coins/hour)
    earn_rates = {
        level_1_to_10 = 200,       -- 200 coins/hour
        level_10_to_30 = 500,
        level_30_plus = 1200
    },
    
    -- Time to afford items (hours)
    time_to_afford = {
        common_sword = 0.5,        -- 30 นาที
        rare_sword = 2,            -- 2 ชั่วโมง
        legendary_armor = 40       -- 40 ชั่วโมง
    }
}
```

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Auction House
สร้างระบบ Auction House ที่ผู้เล่นสามารถวาง items ขายให้ผู้เล่นอื่นได้ พร้อมระบบ bidding

### แบบฝึกหัดที่ 2: Season Pass
สร้าง Season Pass ที่ผู้เล่นซื้อด้วย Gems แล้วได้รับ rewards ตาม season level ที่สะสมด้วย tokens

### แบบฝึกหัดที่ 3: Gambling System
สร้างระบบ "Fortune Wheel" ที่ใช้ Coins หมุน และได้รับ random rewards

### แบบฝึกหัดที่ 4: Currency Conversion UI
สร้าง UI สวยงามสำหรับ exchange Gems เป็น Coins พร้อมแสดง live preview จำนวนที่จะได้รับ

## สรุป

ระบบสกุลเงินที่ปลอดภัยและสมบูรณ์:
- **CurrencyConfig** - นิยามสกุลเงิน, exchange rates, earning sources
- **CurrencyManager** - API ปลอดภัยสำหรับ add/remove/transfer currency
- **Transaction Lock** - ป้องกัน race condition
- **Daily Login** - ให้ rewards ผู้เล่นที่ login ทุกวัน
- **Coin Drop** - drop coins ในโลกให้เก็บ
- **Anti-Exploit** - rate limiting, sanity checks, logging
- **CurrencyUI** - แสดงยอดเงิน, notifications, daily reward popup
- **Economy Design** - balance ระหว่าง earning และ spending
