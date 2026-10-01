# Part 44: RemoteFunctions - การเรียกฟังก์ชันข้ามฝั่ง

## บทนำ

RemoteFunction ต่างจาก RemoteEvent ตรงที่รองรับการตอบกลับ (return value) เหมาะสำหรับกรณีที่ต้องการถามข้อมูลจาก Server หรือ Client และรอคำตอบ เช่น "มีของชิ้นนี้ในคลังไหม?" หรือ "ราคาของชิ้นนี้คือเท่าไหร่?"

## RemoteEvent vs RemoteFunction

| คุณสมบัติ | RemoteEvent | RemoteFunction |
|-----------|-------------|----------------|
| การตอบกลับ | ไม่มี | มี (return value) |
| การทำงาน | Async (ไม่รอ) | Sync (รอคำตอบ) |
| ความเสี่ยง | ต่ำ | สูงกว่า (อาจค้าง) |
| การใช้งาน | แจ้งเหตุการณ์ | ถามข้อมูล |

## การสร้างและใช้งาน RemoteFunction

### การสร้าง

```lua
-- ServerScriptService/SetupFunctions.lua
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- สร้าง folder
local functionsFolder = Instance.new("Folder")
functionsFolder.Name = "Functions"
functionsFolder.Parent = ReplicatedStorage

-- สร้าง RemoteFunctions
local functionNames = {
    "GetPlayerData",
    "GetShopItems",
    "GetInventory",
    "GetLeaderboard",
    "GetQuestStatus",
    "CanPurchaseItem",
    "GetServerInfo"
}

for _, name in ipairs(functionNames) do
    local rf = Instance.new("RemoteFunction")
    rf.Name = name
    rf.Parent = functionsFolder
end

print("RemoteFunctions พร้อมใช้งาน!")
```

### Server กำหนด OnServerInvoke

```lua
-- Script ใน ServerScriptService
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")

local getShopItems = ReplicatedStorage:WaitForChild("Functions"):WaitForChild("GetShopItems")

-- ฐานข้อมูลสินค้า
local shopInventory = {
    {
        id = "sword_basic",
        name = "ดาบธรรมดา",
        description = "ดาบเหล็กธรรมดา ใช้งานได้ดี",
        price = 100,
        damage = 15,
        category = "weapon",
        icon = "rbxassetid://12345"
    },
    {
        id = "shield_wood",
        name = "โล่ไม้",
        description = "โล่ทำจากไม้โอ๊ค",
        price = 50,
        defense = 10,
        category = "armor",
        icon = "rbxassetid://12346"
    },
    {
        id = "potion_health",
        name = "ยาแดง",
        description = "ฟื้นฟู HP 50 หน่วย",
        price = 25,
        healAmount = 50,
        category = "consumable",
        icon = "rbxassetid://12347"
    }
}

-- ===== Server ตอบกลับ client =====
getShopItems.OnServerInvoke = function(player, category)
    print(string.format("[Shop] %s ขอข้อมูลสินค้า (category: %s)", 
        player.Name, category or "all"))
    
    -- กรองตาม category ถ้ามี
    if category and category ~= "all" then
        local filtered = {}
        for _, item in ipairs(shopInventory) do
            if item.category == category then
                table.insert(filtered, item)
            end
        end
        return filtered
    end
    
    return shopInventory
end
```

### Client เรียก InvokeServer

```lua
-- LocalScript ใน StarterGui
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")

local getShopItems = ReplicatedStorage:WaitForChild("Functions"):WaitForChild("GetShopItems")

-- ===== Client เรียก Server และรอคำตอบ =====
local function loadShopUI(category)
    -- InvokeServer จะ BLOCK จนกว่า server จะตอบกลับ
    local success, items = pcall(function()
        return getShopItems:InvokeServer(category or "all")
    end)
    
    if success and items then
        print("ได้รับสินค้า " .. #items .. " รายการ")
        
        -- แสดงใน UI
        for _, item in ipairs(items) do
            print(string.format("  - %s: %d เหรียญ", item.name, item.price))
        end
        
        return items
    else
        warn("โหลดสินค้าล้มเหลว: " .. tostring(items))
        return {}
    end
end

-- เรียกใช้
task.spawn(function()
    local allItems = loadShopUI("all")
    local weapons = loadShopUI("weapon")
end)
```

## ตัวอย่างการใช้งานจริง: ระบบร้านค้าสมบูรณ์

### Server-side

```lua
-- ServerScriptService/ShopServer.lua
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")

-- สมมติว่ามี DataManager จาก Part 42
-- local DataManager = require(ServerStorage.DataManager)

local Functions = ReplicatedStorage:WaitForChild("Functions")
local Events = ReplicatedStorage:WaitForChild("Events")

-- RemoteFunctions
local getInventoryRF = Functions:WaitForChild("GetInventory")
local canPurchaseRF = Functions:WaitForChild("CanPurchaseItem")
local getPlayerDataRF = Functions:WaitForChild("GetPlayerData")

-- RemoteEvents
local purchaseRE = Events:WaitForChild("Economy_Purchase")
local notifyRE = Events:WaitForChild("UI_Notification")

-- Item Database (ในระบบจริงควรใช้ ModuleScript)
local ITEMS = {
    sword_basic = { name = "ดาบธรรมดา", price = 100, type = "weapon", maxStack = 1 },
    sword_iron = { name = "ดาบเหล็ก", price = 500, type = "weapon", maxStack = 1 },
    shield_wood = { name = "โล่ไม้", price = 50, type = "armor", maxStack = 1 },
    potion_hp = { name = "ยาแดง", price = 25, type = "consumable", maxStack = 99 },
    potion_mp = { name = "ยาน้ำเงิน", price = 30, type = "consumable", maxStack = 99 }
}

-- ข้อมูลผู้เล่น (ในระบบจริงใช้ DataManager)
local playerData = {}

Players.PlayerAdded:Connect(function(player)
    playerData[player.UserId] = {
        coins = 500,
        inventory = {
            { id = "potion_hp", quantity = 3 }
        }
    }
end)

Players.PlayerRemoving:Connect(function(player)
    playerData[player.UserId] = nil
end)

-- ===== Get Player Data (Client asks Server) =====
getPlayerDataRF.OnServerInvoke = function(player)
    local data = playerData[player.UserId]
    if not data then return nil end
    
    -- คืนเฉพาะข้อมูลที่ client ควรรู้
    return {
        coins = data.coins,
        inventoryCount = #data.inventory
    }
end

-- ===== Get Inventory =====
getInventoryRF.OnServerInvoke = function(player)
    local data = playerData[player.UserId]
    if not data then return {} end
    
    -- เพิ่มข้อมูล item จาก database
    local enrichedInventory = {}
    for _, slot in ipairs(data.inventory) do
        local itemInfo = ITEMS[slot.id]
        if itemInfo then
            table.insert(enrichedInventory, {
                id = slot.id,
                name = itemInfo.name,
                quantity = slot.quantity,
                type = itemInfo.type,
                maxStack = itemInfo.maxStack
            })
        end
    end
    
    return enrichedInventory
end

-- ===== Can Purchase Check =====
canPurchaseRF.OnServerInvoke = function(player, itemId, quantity)
    quantity = quantity or 1
    
    -- ตรวจสอบ item มีอยู่จริง
    local item = ITEMS[itemId]
    if not item then
        return false, "ไม่พบสินค้านี้"
    end
    
    -- ตรวจสอบเหรียญ
    local data = playerData[player.UserId]
    if not data then
        return false, "ไม่พบข้อมูลผู้เล่น"
    end
    
    local totalCost = item.price * quantity
    if data.coins < totalCost then
        return false, string.format("เหรียญไม่พอ (ต้องการ %d, มี %d)", totalCost, data.coins)
    end
    
    -- ตรวจสอบ inventory ไม่เต็ม
    if #data.inventory >= 50 then
        return false, "คลังสินค้าเต็ม!"
    end
    
    return true, "ซื้อได้"
end

-- ===== Purchase Event =====
purchaseRE.OnServerEvent:Connect(function(player, itemId, quantity)
    quantity = math.max(1, math.min(quantity or 1, 99))
    
    -- ตรวจสอบอีกครั้งบน server (ไม่เชื่อ client)
    local canBuy, reason = canPurchaseRF.OnServerInvoke(player, itemId, quantity)
    
    if not canBuy then
        notifyRE:FireClient(player, "❌ " .. reason, "error")
        return
    end
    
    -- ดำเนินการซื้อ
    local data = playerData[player.UserId]
    local item = ITEMS[itemId]
    local totalCost = item.price * quantity
    
    -- หักเหรียญ
    data.coins = data.coins - totalCost
    
    -- เพิ่มของในคลัง
    local found = false
    for _, slot in ipairs(data.inventory) do
        if slot.id == itemId then
            slot.quantity = slot.quantity + quantity
            found = true
            break
        end
    end
    
    if not found then
        table.insert(data.inventory, { id = itemId, quantity = quantity })
    end
    
    -- แจ้งสำเร็จ
    notifyRE:FireClient(player, 
        string.format("✅ ซื้อ %s x%d สำเร็จ! (-💰%d)", item.name, quantity, totalCost),
        "success"
    )
    
    print(string.format("[Shop] %s ซื้อ %s x%d (-%d coins)", 
        player.Name, item.name, quantity, totalCost))
end)
```

### Client-side

```lua
-- LocalScript ใน StarterGui
-- ShopClient.lua

local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")

local player = Players.LocalPlayer

local Functions = ReplicatedStorage:WaitForChild("Functions")
local Events = ReplicatedStorage:WaitForChild("Events")

local getInventoryRF = Functions:WaitForChild("GetInventory")
local canPurchaseRF = Functions:WaitForChild("CanPurchaseItem")
local getPlayerDataRF = Functions:WaitForChild("GetPlayerData")
local purchaseRE = Events:WaitForChild("Economy_Purchase")
local notifyRE = Events:WaitForChild("UI_Notification")

-- ===== Functions =====

local function getPlayerInfo()
    local success, data = pcall(function()
        return getPlayerDataRF:InvokeServer()
    end)
    
    if success then
        return data
    else
        warn("getPlayerInfo ล้มเหลว: " .. tostring(data))
        return nil
    end
end

local function getInventory()
    local success, inventory = pcall(function()
        return getInventoryRF:InvokeServer()
    end)
    
    if success then
        return inventory or {}
    else
        return {}
    end
end

local function checkCanBuy(itemId, quantity)
    local success, canBuy, reason = pcall(function()
        return canPurchaseRF:InvokeServer(itemId, quantity)
    end)
    
    if success then
        return canBuy, reason
    else
        return false, "ตรวจสอบล้มเหลว"
    end
end

local function purchaseItem(itemId, quantity)
    -- ตรวจสอบก่อน (client-side check สำหรับ UX)
    local canBuy, reason = checkCanBuy(itemId, quantity)
    
    if not canBuy then
        print("❌ ซื้อไม่ได้: " .. reason)
        return false
    end
    
    -- ส่งคำขอซื้อ
    purchaseRE:FireServer(itemId, quantity)
    return true
end

-- ===== Shop UI =====

local shopGui = Instance.new("ScreenGui")
shopGui.Name = "ShopGui"
shopGui.Parent = player.PlayerGui

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 400, 0, 500)
mainFrame.Position = UDim2.new(0.5, -200, 0.5, -250)
mainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
mainFrame.Visible = false
mainFrame.Parent = shopGui

-- Title
local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, 0, 0, 50)
titleLabel.Text = "🏪 ร้านค้า"
titleLabel.TextSize = 24
titleLabel.TextColor3 = Color3.new(1, 1, 1)
titleLabel.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
titleLabel.Parent = mainFrame

-- ปุ่มเปิดร้าน
local openButton = Instance.new("TextButton")
openButton.Size = UDim2.new(0, 100, 0, 40)
openButton.Position = UDim2.new(0, 10, 0, 10)
openButton.Text = "🏪 ร้านค้า"
openButton.BackgroundColor3 = Color3.fromRGB(255, 165, 0)
openButton.Parent = shopGui

local isOpen = false
openButton.MouseButton1Click:Connect(function()
    isOpen = not isOpen
    mainFrame.Visible = isOpen
    
    if isOpen then
        -- โหลดข้อมูลเมื่อเปิด
        task.spawn(function()
            local playerInfo = getPlayerInfo()
            if playerInfo then
                print("💰 เหรียญ:", playerInfo.coins)
                print("🎒 ของในคลัง:", playerInfo.inventoryCount)
            end
        end)
    end
end)

-- ===== Receive Notifications =====
notifyRE.OnClientEvent:Connect(function(message, notifType)
    -- แสดง notification
    print("[Notification] " .. message)
    
    -- ในระบบจริงจะสร้าง UI notification ที่สวยงาม
    local notifLabel = Instance.new("TextLabel")
    notifLabel.Size = UDim2.new(0, 300, 0, 40)
    notifLabel.Position = UDim2.new(0.5, -150, 0, 60)
    notifLabel.Text = message
    notifLabel.TextSize = 16
    
    if notifType == "success" then
        notifLabel.BackgroundColor3 = Color3.fromRGB(0, 150, 0)
    elseif notifType == "error" then
        notifLabel.BackgroundColor3 = Color3.fromRGB(200, 0, 0)
    else
        notifLabel.BackgroundColor3 = Color3.fromRGB(50, 50, 100)
    end
    
    notifLabel.TextColor3 = Color3.new(1, 1, 1)
    notifLabel.Parent = player.PlayerGui
    
    -- ลบออกหลัง 3 วินาที
    game:GetService("Debris"):AddItem(notifLabel, 3)
end)

-- ทดสอบ
task.spawn(function()
    task.wait(2)
    print("=== ทดสอบร้านค้า ===")
    
    -- ดูข้อมูล
    local info = getPlayerInfo()
    if info then
        print("เหรียญ:", info.coins)
    end
    
    -- ตรวจสอบก่อนซื้อ
    local canBuy, reason = checkCanBuy("potion_hp", 5)
    print("ซื้อ potion_hp x5:", canBuy, reason)
    
    -- ซื้อ
    purchaseItem("potion_hp", 3)
end)
```

## ข้อควรระวัง: RemoteFunction อาจค้าง!

```lua
-- ⚠️ ปัญหาสำคัญ: ถ้า InvokeServer ไม่ได้รับการตอบกลับ จะ yield ตลอด!

-- ❌ อันตราย - ถ้า server timeout script จะค้าง
local result = myFunction:InvokeServer()  -- อาจค้างตลอดไป!

-- ✅ ปลอดภัยกว่า - ใช้ coroutine หรือ timeout
local function invokeWithTimeout(remoteFunc, timeout, ...)
    local result = nil
    local completed = false
    
    -- เรียกใน coroutine แยก
    task.spawn(function()
        local success, value = pcall(function()
            return remoteFunc:InvokeServer(...)
        end)
        if success then
            result = value
        end
        completed = true
    end)
    
    -- รอผลลัพธ์ หรือ timeout
    local startTime = os.clock()
    while not completed and os.clock() - startTime < timeout do
        task.wait(0.1)
    end
    
    if not completed then
        warn("RemoteFunction timeout!")
        return nil
    end
    
    return result
end

-- ใช้งาน
local data = invokeWithTimeout(getShopItems, 5, "weapon")  -- timeout 5 วินาที
```

## Client → Client ผ่าน Server

```lua
-- ถ้าต้องการให้ client คุยกับ client โดยตรง
-- ต้องผ่าน server เสมอ!

-- Server
local relayEvent = ReplicatedStorage:WaitForChild("Events"):WaitForChild("Relay")

relayEvent.OnServerEvent:Connect(function(sender, targetPlayerName, data)
    -- หา player target
    local target = nil
    for _, p in ipairs(Players:GetPlayers()) do
        if p.Name == targetPlayerName then
            target = p
            break
        end
    end
    
    if target then
        -- ส่งต่อไปยัง target
        relayEvent:FireClient(target, sender.Name, data)
    end
end)

-- Client A ส่งถึง Client B
relayEvent:FireServer("PlayerB", { type = "tradeRequest", item = "sword" })

-- Client B รับ
relayEvent.OnClientEvent:Connect(function(senderName, data)
    print(senderName .. " ส่งข้อความ:", data.type)
end)
```

## Server Invoke Client (ใช้ระวัง)

```lua
-- RemoteFunction สามารถให้ Server invoke Client ได้
-- แต่อันตรายมาก! ถ้า client disconnect ระหว่าง invoke server จะค้าง

-- ❌ ระวัง - อาจค้างได้
local confirmRF = ReplicatedStorage:WaitForChild("Functions"):WaitForChild("ConfirmAction")

-- Client กำหนด callback
confirmRF.OnClientInvoke = function(question)
    -- แสดง dialog และรอ user response
    -- ปัญหา: ถ้า user ปิดเกมระหว่างรอ server จะค้างตลอดไป!
    return true  -- user กด yes
end

-- Server invoke client (อันตราย!)
-- local response = confirmRF:InvokeClient(player, "คุณต้องการซื้อหรือไม่?")

-- ✅ ทางเลือกที่ดีกว่า: ใช้ RemoteEvent แทน
-- ส่ง request ไป รอ response กลับ
local confirmRequestRE = Events:WaitForChild("ConfirmRequest")
local confirmResponseRE = Events:WaitForChild("ConfirmResponse")

-- Server
local pendingConfirms = {}

local function askPlayerConfirm(player, question, callback)
    local requestId = tostring(os.time()) .. tostring(math.random(1000, 9999))
    pendingConfirms[requestId] = callback
    
    confirmRequestRE:FireClient(player, requestId, question)
    
    -- timeout 30 วินาที
    task.delay(30, function()
        if pendingConfirms[requestId] then
            pendingConfirms[requestId] = nil
            callback(false)  -- timeout = ปฏิเสธ
        end
    end)
end

confirmResponseRE.OnServerEvent:Connect(function(player, requestId, response)
    if pendingConfirms[requestId] then
        local callback = pendingConfirms[requestId]
        pendingConfirms[requestId] = nil
        callback(response)
    end
end)

-- Client
confirmRequestRE.OnClientEvent:Connect(function(requestId, question)
    -- แสดง dialog
    print("Server ถาม: " .. question)
    -- สมมติ user กด yes
    confirmResponseRE:FireServer(requestId, true)
end)
```

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: ระบบตรวจสอบ Player Stats
สร้าง RemoteFunction ที่ client สามารถถามสถิติของผู้เล่นคนอื่น

### แบบฝึกหัดที่ 2: ระบบ Quest Info
สร้าง RemoteFunction ที่ client ขอข้อมูล quest ที่กำลังทำ

### แบบฝึกหัดที่ 3: Server Configuration
สร้าง RemoteFunction ที่ client ขอ configuration ของ server เช่น เวลาเล่น, โหมดเกม

## สรุป

RemoteFunction เหมาะสำหรับ:
- **ถาม-ตอบข้อมูล** ที่ต้องการ return value
- **ตรวจสอบสิ่งต่างๆ** ก่อนดำเนินการ
- **โหลดข้อมูล** เมื่อ UI เปิด

ควรหลีกเลี่ยง:
- **Server InvokeClient** - อาจทำให้ server ค้าง
- **ใช้ทำงานที่ใช้เวลานาน** - ผู้เล่นจะรู้สึกเกมค้าง

ใช้ RemoteEvent แทนเมื่อไม่จำเป็นต้องรอการตอบกลับ
