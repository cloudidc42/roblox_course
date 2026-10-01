# Part 51: ระบบ Inventory สมบูรณ์

## บทนำ

Inventory System หรือระบบกระเป๋าเป็นฟีเจอร์พื้นฐานในเกม RPG ที่ช่วยให้ผู้เล่นเก็บและจัดการไอเทมต่างๆ ในบทนี้เราจะสร้างระบบ Inventory ที่สมบูรณ์ รองรับทั้งไอเทม stackable และ non-stackable, drag-and-drop และ equipment slots

## โครงสร้าง Inventory

```lua
-- โครงสร้างข้อมูล Inventory
playerData.inventory = {
    slots = {
        [1] = { itemId = "sword_basic", quantity = 1, slotType = "any" },
        [2] = { itemId = "potion_hp", quantity = 5, slotType = "any" },
        [3] = nil,  -- ว่าง
        ...
    },
    maxSlots = 24,
    
    equipment = {
        weapon = { itemId = "sword_basic", quantity = 1 },
        armor = nil,
        helmet = nil,
        boots = nil,
        accessory1 = nil,
        accessory2 = nil
    }
}
```

## Inventory Manager (Server)

```lua
-- ServerStorage/Inventory/InventoryManager.lua (ModuleScript)

local InventoryManager = {}

-- ===== Configuration =====
local CONFIG = {
    defaultSlots = 24,
    maxSlots = 100,
    
    -- ประเภท slot
    slotTypes = {
        any = "any",
        weapon = "weapon",
        armor = "armor",
        helmet = "helmet",
        boots = "boots",
        accessory = "accessory"
    }
}

-- Item Database reference (จาก ShopData หรือ ItemDatabase module)
local ITEMS = {
    sword_basic = { 
        name = "ดาบธรรมดา", 
        type = "weapon", 
        stackable = false, 
        maxStack = 1,
        stats = { damage = 15 }
    },
    shield_wood = { 
        name = "โล่ไม้", 
        type = "armor", 
        stackable = false,
        maxStack = 1,
        stats = { defense = 10 }
    },
    potion_hp = { 
        name = "ยาแดง", 
        type = "consumable", 
        stackable = true, 
        maxStack = 99,
        stats = { heal = 50 }
    },
    potion_mp = { 
        name = "ยาน้ำเงิน", 
        type = "consumable", 
        stackable = true, 
        maxStack = 99,
        stats = { mana = 50 }
    },
    gem_red = { 
        name = "พลอยแดง", 
        type = "material", 
        stackable = true, 
        maxStack = 999 
    }
}

-- ===== Initialization =====

function InventoryManager.createInventory(maxSlots)
    maxSlots = maxSlots or CONFIG.defaultSlots
    
    local inventory = {
        slots = {},
        maxSlots = maxSlots,
        equipment = {
            weapon = nil,
            armor = nil,
            helmet = nil,
            boots = nil,
            accessory1 = nil,
            accessory2 = nil
        }
    }
    
    -- สร้าง slots ว่าง
    for i = 1, maxSlots do
        inventory.slots[i] = nil
    end
    
    return inventory
end

-- ===== Core Functions =====

-- หา slot ว่าง
function InventoryManager.findEmptySlot(inventory)
    for i = 1, inventory.maxSlots do
        if not inventory.slots[i] then
            return i
        end
    end
    return nil
end

-- หา slot ที่มี item นี้อยู่แล้ว (สำหรับ stackable)
function InventoryManager.findStackableSlot(inventory, itemId)
    local item = ITEMS[itemId]
    if not item or not item.stackable then return nil end
    
    for i = 1, inventory.maxSlots do
        local slot = inventory.slots[i]
        if slot and slot.itemId == itemId and 
           slot.quantity < (item.maxStack or 99) then
            return i
        end
    end
    return nil
end

-- เพิ่มไอเทม
function InventoryManager.addItem(inventory, itemId, quantity)
    quantity = quantity or 1
    
    local item = ITEMS[itemId]
    if not item then
        return false, "ไม่พบไอเทม: " .. itemId
    end
    
    local remaining = quantity
    
    while remaining > 0 do
        -- หา stack slot ก่อน
        local stackSlot = InventoryManager.findStackableSlot(inventory, itemId)
        
        if stackSlot then
            local slot = inventory.slots[stackSlot]
            local canAdd = (item.maxStack or 99) - slot.quantity
            local adding = math.min(canAdd, remaining)
            
            slot.quantity = slot.quantity + adding
            remaining = remaining - adding
        else
            -- หา slot ว่าง
            local emptySlot = InventoryManager.findEmptySlot(inventory)
            
            if not emptySlot then
                return false, "กระเป๋าเต็ม! (เพิ่มได้เพียง " .. (quantity - remaining) .. " ชิ้น)"
            end
            
            local adding = math.min(item.maxStack or 1, remaining)
            
            inventory.slots[emptySlot] = {
                itemId = itemId,
                quantity = adding,
                addedAt = os.time()
            }
            
            remaining = remaining - adding
        end
    end
    
    return true, string.format("เพิ่ม %s x%d เข้าคลัง", item.name, quantity)
end

-- ลบไอเทม
function InventoryManager.removeItem(inventory, itemId, quantity)
    quantity = quantity or 1
    
    -- ตรวจสอบว่ามีพอไหม
    local total = InventoryManager.countItem(inventory, itemId)
    if total < quantity then
        return false, string.format("ของไม่พอ (มี %d ต้องการลบ %d)", total, quantity)
    end
    
    local remaining = quantity
    
    for i = 1, inventory.maxSlots do
        if remaining <= 0 then break end
        
        local slot = inventory.slots[i]
        if slot and slot.itemId == itemId then
            if slot.quantity <= remaining then
                remaining = remaining - slot.quantity
                inventory.slots[i] = nil
            else
                slot.quantity = slot.quantity - remaining
                remaining = 0
            end
        end
    end
    
    return true, "ลบสำเร็จ"
end

-- นับจำนวนไอเทม
function InventoryManager.countItem(inventory, itemId)
    local total = 0
    
    for _, slot in pairs(inventory.slots) do
        if slot and slot.itemId == itemId then
            total = total + (slot.quantity or 1)
        end
    end
    
    -- ตรวจสอบใน equipment ด้วย
    for _, equip in pairs(inventory.equipment) do
        if equip and equip.itemId == itemId then
            total = total + 1
        end
    end
    
    return total
end

-- ย้าย item จาก slot นึงไปอีก slot
function InventoryManager.moveItem(inventory, fromSlot, toSlot)
    if fromSlot == toSlot then return true end
    
    local fromItem = inventory.slots[fromSlot]
    if not fromItem then return false, "ไม่มีของใน slot นั้น" end
    
    local toItem = inventory.slots[toSlot]
    
    if toItem then
        -- ตรวจสอบว่าเป็น item เดียวกันและ stackable
        if toItem.itemId == fromItem.itemId then
            local item = ITEMS[fromItem.itemId]
            if item and item.stackable then
                -- Stack กัน
                local canAdd = (item.maxStack or 99) - toItem.quantity
                local transfer = math.min(canAdd, fromItem.quantity)
                
                toItem.quantity = toItem.quantity + transfer
                fromItem.quantity = fromItem.quantity - transfer
                
                if fromItem.quantity <= 0 then
                    inventory.slots[fromSlot] = nil
                end
                
                return true
            end
        end
        
        -- Swap
        inventory.slots[fromSlot] = toItem
        inventory.slots[toSlot] = fromItem
    else
        -- ย้ายไป slot ว่าง
        inventory.slots[toSlot] = fromItem
        inventory.slots[fromSlot] = nil
    end
    
    return true
end

-- ===== Equipment =====

function InventoryManager.equip(inventory, slotIndex)
    local slot = inventory.slots[slotIndex]
    if not slot then return false, "ไม่มีของใน slot นั้น" end
    
    local item = ITEMS[slot.itemId]
    if not item then return false, "ไม่พบข้อมูลไอเทม" end
    
    -- ตรวจสอบว่า equip ได้ไหม
    local equipSlot = item.type
    if not inventory.equipment[equipSlot] and equipSlot ~= "weapon" and equipSlot ~= "armor" then
        return false, "ไอเทมนี้ไม่สามารถสวมใส่ได้"
    end
    
    -- ถ้ามีของ equip อยู่แล้ว ถอดออกก่อน
    if inventory.equipment[equipSlot] then
        local unequipped = inventory.equipment[equipSlot]
        
        -- ใส่ของเก่ากลับ inventory
        local emptySlot = InventoryManager.findEmptySlot(inventory)
        if emptySlot then
            inventory.slots[emptySlot] = unequipped
        end
    end
    
    -- Equip ของใหม่
    inventory.equipment[equipSlot] = slot
    inventory.slots[slotIndex] = nil
    
    return true, "สวม " .. item.name .. " แล้ว"
end

function InventoryManager.unequip(inventory, equipSlot)
    local equipped = inventory.equipment[equipSlot]
    if not equipped then return false, "ไม่มีของใน slot นั้น" end
    
    local emptySlot = InventoryManager.findEmptySlot(inventory)
    if not emptySlot then return false, "กระเป๋าเต็ม!" end
    
    inventory.slots[emptySlot] = equipped
    inventory.equipment[equipSlot] = nil
    
    local item = ITEMS[equipped.itemId]
    return true, "ถอด " .. (item and item.name or equipped.itemId) .. " แล้ว"
end

-- ===== Sorting =====

function InventoryManager.sort(inventory, sortType)
    sortType = sortType or "type"
    
    -- เก็บ items ทั้งหมด
    local items = {}
    for i = 1, inventory.maxSlots do
        if inventory.slots[i] then
            table.insert(items, inventory.slots[i])
            inventory.slots[i] = nil
        end
    end
    
    -- เรียงลำดับ
    if sortType == "type" then
        table.sort(items, function(a, b)
            local itemA = ITEMS[a.itemId]
            local itemB = ITEMS[b.itemId]
            local typeA = itemA and itemA.type or "z"
            local typeB = itemB and itemB.type or "z"
            
            if typeA ~= typeB then
                return typeA < typeB
            end
            return a.itemId < b.itemId
        end)
    elseif sortType == "name" then
        table.sort(items, function(a, b)
            local nameA = (ITEMS[a.itemId] and ITEMS[a.itemId].name) or a.itemId
            local nameB = (ITEMS[b.itemId] and ITEMS[b.itemId].name) or b.itemId
            return nameA < nameB
        end)
    elseif sortType == "quantity" then
        table.sort(items, function(a, b)
            return (a.quantity or 1) > (b.quantity or 1)
        end)
    end
    
    -- ใส่กลับ
    for i, item in ipairs(items) do
        inventory.slots[i] = item
    end
end

-- ===== Stats Calculation =====

function InventoryManager.getEquipmentStats(inventory)
    local stats = {
        damage = 0,
        defense = 0,
        speed = 0,
        hp = 0,
        mp = 0
    }
    
    for _, equip in pairs(inventory.equipment) do
        if equip then
            local item = ITEMS[equip.itemId]
            if item and item.stats then
                for stat, value in pairs(item.stats) do
                    stats[stat] = (stats[stat] or 0) + value
                end
            end
        end
    end
    
    return stats
end

-- ===== Information =====

function InventoryManager.getInfo(inventory)
    local usedSlots = 0
    local totalItems = 0
    
    for _, slot in pairs(inventory.slots) do
        if slot then
            usedSlots = usedSlots + 1
            totalItems = totalItems + (slot.quantity or 1)
        end
    end
    
    return {
        used = usedSlots,
        max = inventory.maxSlots,
        totalItems = totalItems,
        freeSlots = inventory.maxSlots - usedSlots
    }
end

return InventoryManager
```

## Inventory UI (Client)

```lua
-- LocalScript ใน StarterGui/InventoryGui

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local playerGui = player.PlayerGui

-- ===== UI Setup =====
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "InventoryGui"
screenGui.ResetOnSpawn = false
screenGui.Parent = playerGui

-- Main Frame
local invFrame = Instance.new("Frame")
invFrame.Name = "InventoryFrame"
invFrame.Size = UDim2.new(0, 500, 0, 420)
invFrame.Position = UDim2.new(0.5, -250, 0.5, -210)
invFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 35)
invFrame.Visible = false
invFrame.Parent = screenGui
Instance.new("UICorner", invFrame).CornerRadius = UDim.new(0, 16)

-- Title
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 45)
title.Text = "🎒 กระเป๋า"
title.TextSize = 20
title.Font = Enum.Font.GothamBold
title.TextColor3 = Color3.new(1,1,1)
title.BackgroundColor3 = Color3.fromRGB(35, 100, 200)
Instance.new("UICorner", title).CornerRadius = UDim.new(0, 16)
title.BackgroundTransparency = 0
title.Parent = invFrame

-- Equipment Panel (ซ้าย)
local equipPanel = Instance.new("Frame")
equipPanel.Size = UDim2.new(0, 140, 1, -55)
equipPanel.Position = UDim2.new(0, 0, 0, 50)
equipPanel.BackgroundColor3 = Color3.fromRGB(15, 15, 25)
Instance.new("UICorner", equipPanel).CornerRadius = UDim.new(0, 10)
equipPanel.Parent = invFrame

local equipTitle = Instance.new("TextLabel")
equipTitle.Size = UDim2.new(1, 0, 0, 30)
equipTitle.Text = "⚙️ สวมใส่"
equipTitle.TextSize = 13
equipTitle.TextColor3 = Color3.new(0.8,0.8,0.8)
equipTitle.BackgroundTransparency = 1
equipTitle.Parent = equipPanel

-- Equipment Slots
local equipSlots = {
    { name = "weapon", label = "⚔️ อาวุธ", pos = UDim2.new(0.5, -30, 0, 35) },
    { name = "armor", label = "🛡️ เกราะ", pos = UDim2.new(0.5, -30, 0, 90) },
    { name = "helmet", label = "⛑️ หมวก", pos = UDim2.new(0.5, -30, 0, 145) },
    { name = "boots", label = "👢 รองเท้า", pos = UDim2.new(0.5, -30, 0, 200) },
    { name = "accessory1", label = "💍 แหวน1", pos = UDim2.new(0.5, -30, 0, 255) },
    { name = "accessory2", label = "💍 แหวน2", pos = UDim2.new(0.5, -30, 0, 310) }
}

local equipSlotButtons = {}

for _, slot in ipairs(equipSlots) do
    local slotFrame = Instance.new("Frame")
    slotFrame.Size = UDim2.new(0, 60, 0, 50)
    slotFrame.Position = slot.pos
    slotFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 50)
    Instance.new("UICorner", slotFrame).CornerRadius = UDim.new(0, 8)
    slotFrame.Parent = equipPanel
    
    local slotLabel = Instance.new("TextLabel")
    slotLabel.Size = UDim2.new(1, 0, 0.5, 0)
    slotLabel.Text = slot.label
    slotLabel.TextSize = 10
    slotLabel.TextColor3 = Color3.new(0.6, 0.6, 0.6)
    slotLabel.BackgroundTransparency = 1
    slotLabel.TextWrapped = true
    slotLabel.Parent = slotFrame
    
    local itemDisplay = Instance.new("TextLabel")
    itemDisplay.Name = "ItemDisplay"
    itemDisplay.Size = UDim2.new(1, 0, 0.5, 0)
    itemDisplay.Position = UDim2.new(0, 0, 0.5, 0)
    itemDisplay.Text = "ว่าง"
    itemDisplay.TextSize = 10
    itemDisplay.TextColor3 = Color3.new(0.4, 0.4, 0.4)
    itemDisplay.BackgroundTransparency = 1
    itemDisplay.TextWrapped = true
    itemDisplay.Parent = slotFrame
    
    equipSlotButtons[slot.name] = {
        frame = slotFrame,
        label = itemDisplay
    }
end

-- Inventory Grid (ขวา)
local gridFrame = Instance.new("ScrollingFrame")
gridFrame.Size = UDim2.new(1, -150, 1, -55)
gridFrame.Position = UDim2.new(0, 145, 0, 50)
gridFrame.BackgroundTransparency = 1
gridFrame.ScrollBarThickness = 4
gridFrame.Parent = invFrame

local gridLayout = Instance.new("UIGridLayout")
gridLayout.CellSize = UDim2.new(0, 60, 0, 60)
gridLayout.CellPadding = UDim2.new(0, 5, 0, 5)
gridLayout.Parent = gridFrame

Instance.new("UIPadding", gridFrame).PaddingLeft = UDim.new(0, 5)

-- ===== Slot Buttons =====
local MAX_SLOTS = 24
local slotButtons = {}
local selectedSlot = nil

local function createSlot(slotIndex)
    local slotBtn = Instance.new("TextButton")
    slotBtn.Name = "Slot_" .. slotIndex
    slotBtn.Size = UDim2.new(0, 60, 0, 60)
    slotBtn.BackgroundColor3 = Color3.fromRGB(25, 25, 45)
    slotBtn.Text = ""
    Instance.new("UICorner", slotBtn).CornerRadius = UDim.new(0, 8)
    slotBtn.Parent = gridFrame
    
    -- Slot number
    local numLabel = Instance.new("TextLabel")
    numLabel.Size = UDim2.new(0, 15, 0, 15)
    numLabel.Position = UDim2.new(0, 2, 0, 2)
    numLabel.Text = tostring(slotIndex)
    numLabel.TextSize = 8
    numLabel.TextColor3 = Color3.new(0.4, 0.4, 0.4)
    numLabel.BackgroundTransparency = 1
    numLabel.Parent = slotBtn
    
    -- Item icon
    local icon = Instance.new("TextLabel")
    icon.Name = "Icon"
    icon.Size = UDim2.new(1, -4, 0.6, 0)
    icon.Position = UDim2.new(0, 2, 0, 5)
    icon.Text = ""
    icon.TextSize = 26
    icon.BackgroundTransparency = 1
    icon.Parent = slotBtn
    
    -- Quantity
    local quantity = Instance.new("TextLabel")
    quantity.Name = "Quantity"
    quantity.Size = UDim2.new(1, -4, 0, 18)
    quantity.Position = UDim2.new(0, 2, 1, -20)
    quantity.Text = ""
    quantity.TextSize = 11
    quantity.Font = Enum.Font.GothamBold
    quantity.TextColor3 = Color3.new(1,1,1)
    quantity.BackgroundTransparency = 1
    quantity.TextXAlignment = Enum.TextXAlignment.Right
    quantity.Parent = slotBtn
    
    slotButtons[slotIndex] = slotBtn
    
    -- Click handler
    slotBtn.MouseButton1Click:Connect(function()
        onSlotClicked(slotIndex)
    end)
    
    return slotBtn
end

-- สร้าง slots ทั้งหมด
for i = 1, MAX_SLOTS do
    createSlot(i)
end

gridFrame.CanvasSize = UDim2.new(0, 0, 0, gridLayout.AbsoluteContentSize.Y + 10)

-- ===== Slot Icons =====
local itemIcons = {
    sword_basic = "🗡️",
    shield_wood = "🛡️",
    potion_hp = "🔴",
    potion_mp = "🔵",
    gem_red = "💎",
    default = "📦"
}

-- ===== Update UI =====
local inventoryData = nil

local function updateSlotUI(slotIndex, slotData)
    local btn = slotButtons[slotIndex]
    if not btn then return end
    
    local icon = btn:FindFirstChild("Icon")
    local quantity = btn:FindFirstChild("Quantity")
    
    if slotData then
        -- แสดงไอเทม
        btn.BackgroundColor3 = Color3.fromRGB(35, 35, 60)
        
        if icon then
            icon.Text = itemIcons[slotData.itemId] or itemIcons.default
        end
        
        if quantity then
            if (slotData.quantity or 1) > 1 then
                quantity.Text = "x" .. slotData.quantity
            else
                quantity.Text = ""
            end
        end
    else
        -- ว่าง
        btn.BackgroundColor3 = Color3.fromRGB(25, 25, 45)
        if icon then icon.Text = "" end
        if quantity then quantity.Text = "" end
    end
    
    -- Highlight ถ้า selected
    if selectedSlot == slotIndex then
        btn.BackgroundColor3 = Color3.fromRGB(60, 60, 100)
    end
end

function updateInventoryUI(data)
    inventoryData = data
    
    if not data then return end
    
    -- อัพเดท inventory slots
    for i = 1, MAX_SLOTS do
        updateSlotUI(i, data.slots[i])
    end
    
    -- อัพเดท equipment slots
    for slotName, slotUI in pairs(equipSlotButtons) do
        local equipped = data.equipment[slotName]
        if equipped then
            local icon = itemIcons[equipped.itemId] or itemIcons.default
            slotUI.label.Text = icon
            slotUI.label.TextColor3 = Color3.fromRGB(255, 215, 0)
        else
            slotUI.label.Text = "ว่าง"
            slotUI.label.TextColor3 = Color3.new(0.4, 0.4, 0.4)
        end
    end
    
    -- อัพเดท canvas size
    gridFrame.CanvasSize = UDim2.new(0, 0, 0, gridLayout.AbsoluteContentSize.Y + 10)
end

-- ===== Actions =====

local actionMenu = nil

function onSlotClicked(slotIndex)
    if not inventoryData then return end
    
    local slot = inventoryData.slots[slotIndex]
    
    -- ลบ menu เก่า
    if actionMenu then
        actionMenu:Destroy()
        actionMenu = nil
    end
    
    -- ถ้าคลิก slot เดิม deselect
    if selectedSlot == slotIndex then
        selectedSlot = nil
        updateSlotUI(slotIndex, slot)
        return
    end
    
    -- ถ้ามี selected slot แล้ว ย้ายไอเทม
    if selectedSlot and inventoryData.slots[selectedSlot] then
        -- ส่ง move request
        local moveEvent = ReplicatedStorage.Events:FindFirstChild("Inventory_MoveItem")
        if moveEvent then
            moveEvent:FireServer(selectedSlot, slotIndex)
        end
        
        local prevSlot = selectedSlot
        selectedSlot = nil
        updateSlotUI(prevSlot, inventoryData.slots[prevSlot])
        return
    end
    
    if slot then
        -- Select slot และแสดง action menu
        selectedSlot = slotIndex
        updateSlotUI(slotIndex, slot)
        showActionMenu(slotIndex, slot)
    end
end

function showActionMenu(slotIndex, slotData)
    if actionMenu then actionMenu:Destroy() end
    
    local btn = slotButtons[slotIndex]
    local itemInfo = {
        sword_basic = { name = "ดาบธรรมดา", type = "weapon" },
        potion_hp = { name = "ยาแดง", type = "consumable" },
        -- ...
    }[slotData.itemId] or { name = slotData.itemId, type = "misc" }
    
    actionMenu = Instance.new("Frame")
    actionMenu.Size = UDim2.new(0, 120, 0, 0)  -- จะคำนวณ height ทีหลัง
    actionMenu.Position = UDim2.new(0, 
        btn.AbsolutePosition.X - gridFrame.AbsolutePosition.X + 65,
        0, 
        btn.AbsolutePosition.Y - gridFrame.AbsolutePosition.Y)
    actionMenu.BackgroundColor3 = Color3.fromRGB(20, 20, 35)
    Instance.new("UICorner", actionMenu).CornerRadius = UDim.new(0, 8)
    local menuStroke = Instance.new("UIStroke")
    menuStroke.Color = Color3.fromRGB(100, 100, 150)
    menuStroke.Parent = actionMenu
    actionMenu.ZIndex = 10
    actionMenu.Parent = gridFrame
    
    local actions = {}
    
    -- Use / Equip
    if itemInfo.type == "consumable" then
        table.insert(actions, { text = "🧪 ใช้", action = function()
            local useEvent = ReplicatedStorage.Events:FindFirstChild("Inventory_UseItem")
            if useEvent then useEvent:FireServer(slotIndex) end
        end })
    elseif itemInfo.type == "weapon" or itemInfo.type == "armor" then
        table.insert(actions, { text = "⚙️ สวมใส่", action = function()
            local equipEvent = ReplicatedStorage.Events:FindFirstChild("Inventory_Equip")
            if equipEvent then equipEvent:FireServer(slotIndex) end
        end })
    end
    
    -- Drop
    table.insert(actions, { text = "🗑️ ทิ้ง", action = function()
        local dropEvent = ReplicatedStorage.Events:FindFirstChild("Inventory_DropItem")
        if dropEvent then dropEvent:FireServer(slotIndex, 1) end
    end })
    
    -- Cancel
    table.insert(actions, { text = "✕ ยกเลิก", action = function()
        selectedSlot = nil
        if actionMenu then actionMenu:Destroy() actionMenu = nil end
        updateSlotUI(slotIndex, slotData)
    end })
    
    -- สร้างปุ่ม actions
    local totalHeight = #actions * 30 + 5
    actionMenu.Size = UDim2.new(0, 120, 0, totalHeight)
    
    for i, action in ipairs(actions) do
        local actionBtn = Instance.new("TextButton")
        actionBtn.Size = UDim2.new(1, -4, 0, 25)
        actionBtn.Position = UDim2.new(0, 2, 0, (i-1) * 28 + 3)
        actionBtn.Text = action.text
        actionBtn.TextSize = 12
        actionBtn.BackgroundColor3 = Color3.fromRGB(35, 35, 55)
        actionBtn.TextColor3 = Color3.new(1,1,1)
        Instance.new("UICorner", actionBtn).CornerRadius = UDim.new(0, 6)
        actionBtn.ZIndex = 11
        actionBtn.Parent = actionMenu
        
        actionBtn.MouseButton1Click:Connect(function()
            action.action()
            if actionMenu then
                actionMenu:Destroy()
                actionMenu = nil
            end
            selectedSlot = nil
        end)
    end
end

-- ===== Keyboard Shortcut =====
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    if input.KeyCode == Enum.KeyCode.I or input.KeyCode == Enum.KeyCode.Tab then
        invFrame.Visible = not invFrame.Visible
        
        if invFrame.Visible then
            -- โหลดข้อมูล inventory
            local getInventoryRF = ReplicatedStorage.Functions:FindFirstChild("GetInventory")
            if getInventoryRF then
                task.spawn(function()
                    local ok, data = pcall(function()
                        return getInventoryRF:InvokeServer()
                    end)
                    if ok then
                        updateInventoryUI(data)
                    end
                end)
            end
        end
    end
end)

-- ===== Toggle Button =====
local toggleBtn = Instance.new("TextButton")
toggleBtn.Size = UDim2.new(0, 50, 0, 50)
toggleBtn.Position = UDim2.new(0, 10, 0.5, -25)
toggleBtn.Text = "🎒"
toggleBtn.TextSize = 28
toggleBtn.BackgroundColor3 = Color3.fromRGB(35, 100, 200)
Instance.new("UICorner", toggleBtn).CornerRadius = UDim.new(1,0)
toggleBtn.Parent = screenGui

toggleBtn.MouseButton1Click:Connect(function()
    invFrame.Visible = not invFrame.Visible
end)
```

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Drag and Drop
เพิ่มระบบ drag-and-drop สำหรับย้ายไอเทม

### แบบฝึกหัดที่ 2: Item Tooltip
แสดงรายละเอียดไอเทมเมื่อ hover เมาส์

### แบบฝึกหัดที่ 3: Quick Access Bar
สร้าง hotbar ที่ผู้เล่นกด 1-9 เพื่อใช้ไอเทมได้เร็ว

## สรุป

ระบบ Inventory ที่ดีต้องมี:
- **โครงสร้างข้อมูลที่ชัดเจน** - slots, equipment
- **Functions ที่ครบ** - add, remove, move, equip
- **UI ที่ใช้งานง่าย** - คลิก, drag-drop
- **Validation** - ตรวจสอบทุก action บน server
- **Feedback ที่ดี** - แสดงชื่อ, จำนวน, icon
