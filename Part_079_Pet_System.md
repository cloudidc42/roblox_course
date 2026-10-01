# Part 79: Pet System (ระบบเลี้ยงสัตว์)

## บทนำ

Pet System เป็นระบบที่ได้รับความนิยมมากในเกม Roblox โดยเฉพาะ Simulator Games ในบทนี้เราจะสร้างระบบ Pet ที่สมบูรณ์ รวมถึงการ Hatch, Equip, Stats Bonus, Evolution และ AI ที่ตามผู้เล่น

---

## 1. Pet Data System

### 1.1 PetDatabase Module

```lua
-- PetDatabase.lua (Module Script ใน ReplicatedStorage)
-- ฐานข้อมูล Pet ทั้งหมด

local PetDatabase = {}

-- ระดับความหายาก
local RARITY = {
    COMMON = {name = "Common", color = Color3.fromRGB(150, 150, 150), multiplier = 1},
    RARE = {name = "Rare", color = Color3.fromRGB(50, 100, 255), multiplier = 2},
    EPIC = {name = "Epic", color = Color3.fromRGB(150, 50, 255), multiplier = 4},
    LEGENDARY = {name = "Legendary", color = Color3.fromRGB(255, 200, 0), multiplier = 10},
    MYTHIC = {name = "Mythic", color = Color3.fromRGB(255, 50, 50), multiplier = 25},
}

-- ข้อมูล Pet
local PETS = {
    -- Common Pets
    cat = {
        id = "cat",
        name = "แมว",
        nameEn = "Cat",
        rarity = "COMMON",
        modelId = "rbxassetid://0",  -- ใส่ Model ID จริง
        icon = "rbxassetid://0",
        stats = {
            coinMultiplier = 1.1,   -- เพิ่มเหรียญ 10%
            luck = 1,
            speed = 0,
        },
        description = "แมวน้อยน่ารัก เพิ่มเหรียญ 10%",
        evolutionRequirement = 100,  -- Feed ครั้ง
        evolvesTo = "mega_cat",
    },
    
    dog = {
        id = "dog",
        name = "สุนัข",
        nameEn = "Dog",
        rarity = "COMMON",
        modelId = "rbxassetid://0",
        icon = "rbxassetid://0",
        stats = {
            coinMultiplier = 1.0,
            luck = 2,              -- เพิ่มโชค 2%
            speed = 5,             -- วิ่งเร็วขึ้น 5%
        },
        description = "สุนัขซื่อสัตย์ เพิ่มความเร็ว 5%",
        evolutionRequirement = 100,
        evolvesTo = "mega_dog",
    },
    
    -- Rare Pets
    dragon_egg = {
        id = "dragon_egg",
        name = "ไข่มังกร",
        nameEn = "Dragon Egg",
        rarity = "RARE",
        modelId = "rbxassetid://0",
        icon = "rbxassetid://0",
        stats = {
            coinMultiplier = 1.5,
            luck = 5,
            speed = 0,
            fireDamage = 10,       -- ดีลไฟ
        },
        description = "ไข่มังกรลึกลับ เพิ่มเหรียญ 50%",
        evolutionRequirement = 200,
        evolvesTo = "baby_dragon",
    },
    
    -- Epic Pets
    phoenix = {
        id = "phoenix",
        name = "ฟีนิกซ์",
        nameEn = "Phoenix",
        rarity = "EPIC",
        modelId = "rbxassetid://0",
        icon = "rbxassetid://0",
        stats = {
            coinMultiplier = 3.0,
            luck = 15,
            speed = 10,
            fireResistance = 50,   -- ต้านไฟ
        },
        description = "นกฟีนิกซ์ผู้ฟื้นคืนชีพ เพิ่มเหรียญ 3 เท่า",
        evolutionRequirement = 500,
        evolvesTo = "titan_phoenix",
    },
    
    -- Legendary Pets
    cosmic_cat = {
        id = "cosmic_cat",
        name = "แมวจักรวาล",
        nameEn = "Cosmic Cat",
        rarity = "LEGENDARY",
        modelId = "rbxassetid://0",
        icon = "rbxassetid://0",
        stats = {
            coinMultiplier = 8.0,
            luck = 30,
            speed = 20,
            allStats = 2.0,        -- เพิ่มทุก Stat 2 เท่า
        },
        description = "แมวจากอีกมิติ เพิ่มทุกอย่าง 8 เท่า",
        evolutionRequirement = 1000,
        evolvesTo = nil,           -- ไม่ Evolve อีกแล้ว
    },
    
    -- Evolved Versions
    mega_cat = {
        id = "mega_cat",
        name = "เมก้าแมว",
        nameEn = "Mega Cat",
        rarity = "RARE",
        modelId = "rbxassetid://0",
        icon = "rbxassetid://0",
        isEvolved = true,
        stats = {
            coinMultiplier = 1.5,
            luck = 3,
            speed = 5,
        },
        description = "แมวที่ Evolve แล้ว แรงขึ้น 50%",
    },
}

-- Egg Types (ฟักได้ Pet อะไร)
local EGGS = {
    basic_egg = {
        id = "basic_egg",
        name = "ไข่พื้นฐาน",
        icon = "rbxassetid://0",
        price = 100,
        currency = "coins",
        hatchTime = 10,  -- วินาที
        pets = {
            {id = "cat", weight = 70},
            {id = "dog", weight = 30},
        }
    },
    
    rare_egg = {
        id = "rare_egg",
        name = "ไข่หายาก",
        icon = "rbxassetid://0",
        price = 1000,
        currency = "gems",
        hatchTime = 30,
        pets = {
            {id = "dragon_egg", weight = 70},
            {id = "phoenix", weight = 30},
        }
    },
    
    legendary_egg = {
        id = "legendary_egg",
        name = "ไข่ตำนาน",
        icon = "rbxassetid://0",
        price = 5000,
        currency = "gems",
        hatchTime = 60,
        pets = {
            {id = "phoenix", weight = 70},
            {id = "cosmic_cat", weight = 30},
        }
    },
}

-- ดู Pet Data
function PetDatabase.getPet(petId)
    return PETS[petId]
end

-- ดู Egg Data
function PetDatabase.getEgg(eggId)
    return EGGS[eggId]
end

-- สุ่ม Pet จาก Egg
function PetDatabase.rollPet(eggId)
    local egg = EGGS[eggId]
    if not egg then return nil end
    
    local totalWeight = 0
    for _, entry in ipairs(egg.pets) do
        totalWeight = totalWeight + entry.weight
    end
    
    local roll = math.random() * totalWeight
    local cumulative = 0
    
    for _, entry in ipairs(egg.pets) do
        cumulative = cumulative + entry.weight
        if roll <= cumulative then
            return PETS[entry.id]
        end
    end
    
    return PETS[egg.pets[1].id]  -- Fallback
end

-- คำนวณ Stats รวมของ Pets ที่ Equip
function PetDatabase.calculateBonusStats(equippedPets)
    local totalStats = {
        coinMultiplier = 1.0,
        luck = 0,
        speed = 0,
        fireDamage = 0,
    }
    
    for _, petData in ipairs(equippedPets) do
        local pet = PETS[petData.id]
        if pet and pet.stats then
            for stat, value in pairs(pet.stats) do
                if totalStats[stat] ~= nil then
                    if stat == "coinMultiplier" then
                        totalStats[stat] = totalStats[stat] * value
                    else
                        totalStats[stat] = totalStats[stat] + value
                    end
                end
            end
        end
    end
    
    return totalStats
end

-- ดูทุก Pets
function PetDatabase.getAllPets()
    return PETS
end

-- ดูทุก Eggs
function PetDatabase.getAllEggs()
    return EGGS
end

-- ดู Rarity Info
function PetDatabase.getRarity(rarityKey)
    return RARITY[rarityKey]
end

return PetDatabase
```

---

## 2. Pet Manager (Server)

### 2.1 Core Pet Logic

```lua
-- PetManager.lua (Server Script)
-- จัดการ Pet ทั้งหมดบน Server

local Players = game:GetService("Players")
local DataStoreService = game:GetService("DataStoreService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local PetDatabase = require(ReplicatedStorage:WaitForChild("PetDatabase"))

local petStore = DataStoreService:GetDataStore("PetData_v2")

-- ============================================================
-- REMOTE EVENTS/FUNCTIONS
-- ============================================================

local petsFolder = Instance.new("Folder")
petsFolder.Name = "PetEvents"
petsFolder.Parent = ReplicatedStorage

local events = {
    hatchEgg = Instance.new("RemoteFunction"),
    equipPet = Instance.new("RemoteEvent"),
    unequipPet = Instance.new("RemoteEvent"),
    feedPet = Instance.new("RemoteEvent"),
    releasePet = Instance.new("RemoteEvent"),
    updatePets = Instance.new("RemoteEvent"),
}

for name, event in pairs(events) do
    event.Name = name
    event.Parent = petsFolder
end

-- ============================================================
-- PLAYER PET DATA
-- ============================================================

local playerPets = {}   -- {[userId] = {inventory = {}, equipped = {}}}

-- โหลด Data
local function loadPetData(player)
    local success, data = pcall(function()
        return petStore:GetAsync("pets_" .. player.UserId)
    end)
    
    local defaultData = {
        inventory = {},   -- Pet ทั้งหมดที่มี
        equipped = {},    -- Pet ที่ Equip อยู่ (สูงสุด 3 ตัว)
        maxSlots = 3,     -- จำนวน Slot
    }
    
    if success and data then
        -- Merge กับ Default เพื่อป้องกัน nil fields
        for key, value in pairs(defaultData) do
            if data[key] == nil then
                data[key] = value
            end
        end
        playerPets[player.UserId] = data
    else
        playerPets[player.UserId] = defaultData
    end
    
    return playerPets[player.UserId]
end

-- บันทึก Data
local function savePetData(player)
    local data = playerPets[player.UserId]
    if not data then return end
    
    task.spawn(function()
        pcall(function()
            petStore:SetAsync("pets_" .. player.UserId, data)
        end)
    end)
end

-- ============================================================
-- EGG HATCHING
-- ============================================================

-- Hatch Egg (RemoteFunction)
events.hatchEgg.OnServerInvoke = function(player, eggId)
    local eggData = PetDatabase.getEgg(eggId)
    if not eggData then
        return {success = false, error = "Egg ไม่ถูกต้อง"}
    end
    
    local data = playerPets[player.UserId]
    if not data then
        return {success = false, error = "ไม่พบข้อมูล"}
    end
    
    -- ตรวจสอบว่ามีพอ
    -- (ตรวจสอบ Coins/Gems จาก PlayerData module อื่น)
    
    -- สุ่ม Pet
    local rolledPet = PetDatabase.rollPet(eggId)
    if not rolledPet then
        return {success = false, error = "ไม่สามารถสุ่ม Pet ได้"}
    end
    
    -- สร้าง Pet Instance
    local petInstance = {
        id = rolledPet.id,
        uid = player.UserId .. "_" .. tostring(tick()),  -- Unique ID
        level = 1,
        feedCount = 0,
        nickname = nil,
    }
    
    -- เพิ่มใน Inventory
    table.insert(data.inventory, petInstance)
    
    -- บันทึก
    savePetData(player)
    
    -- ส่งข้อมูลกลับ
    events.updatePets:FireClient(player, data)
    
    return {
        success = true,
        pet = petInstance,
        petData = rolledPet,
    }
end

-- ============================================================
-- EQUIP/UNEQUIP
-- ============================================================

events.equipPet.OnServerEvent:Connect(function(player, petUid)
    local data = playerPets[player.UserId]
    if not data then return end
    
    -- หา Pet ใน Inventory
    local petInstance
    for _, pet in ipairs(data.inventory) do
        if pet.uid == petUid then
            petInstance = pet
            break
        end
    end
    
    if not petInstance then
        warn("Pet ไม่พบใน Inventory:", petUid)
        return
    end
    
    -- ตรวจสอบว่า Equip แล้วหรือยัง
    for _, equipped in ipairs(data.equipped) do
        if equipped.uid == petUid then
            return  -- Equip แล้ว
        end
    end
    
    -- ตรวจสอบ Slot
    if #data.equipped >= data.maxSlots then
        -- Auto-unequip ตัวแรก
        table.remove(data.equipped, 1)
    end
    
    table.insert(data.equipped, petInstance)
    
    -- Update Pet Models
    updatePetModels(player)
    savePetData(player)
    events.updatePets:FireClient(player, data)
end)

events.unequipPet.OnServerEvent:Connect(function(player, petUid)
    local data = playerPets[player.UserId]
    if not data then return end
    
    for i, equipped in ipairs(data.equipped) do
        if equipped.uid == petUid then
            table.remove(data.equipped, i)
            break
        end
    end
    
    updatePetModels(player)
    savePetData(player)
    events.updatePets:FireClient(player, data)
end)

-- ============================================================
-- PET MODELS
-- ============================================================

local petModels = {}  -- {[userId] = {models...}}

function updatePetModels(player)
    local character = player.Character
    if not character then return end
    
    local hrp = character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    
    -- ลบ Models เก่า
    if petModels[player.UserId] then
        for _, model in ipairs(petModels[player.UserId]) do
            if model.Parent then model:Destroy() end
        end
    end
    petModels[player.UserId] = {}
    
    local data = playerPets[player.UserId]
    if not data then return end
    
    -- สร้าง Models สำหรับ Pet ที่ Equip
    for i, equippedPet in ipairs(data.equipped) do
        local petData = PetDatabase.getPet(equippedPet.id)
        if not petData then continue end
        
        -- สร้าง Simple Placeholder Model
        -- (ใน Production จะ load จาก AssetService)
        local petModel = Instance.new("Model")
        petModel.Name = "Pet_" .. equippedPet.id
        
        local petPart = Instance.new("Part")
        petPart.Name = "HumanoidRootPart"
        petPart.Size = Vector3.new(2, 2, 2)
        petPart.CanCollide = false
        petPart.Anchored = true
        petPart.BrickColor = BrickColor.new("Bright blue")  -- Placeholder
        petPart.Parent = petModel
        
        petModel.PrimaryPart = petPart
        petModel.Parent = workspace.Pets or workspace
        
        -- เพิ่ม BillboardGui แสดงชื่อ
        local billboard = Instance.new("BillboardGui")
        billboard.Size = UDim2.new(0, 100, 0, 30)
        billboard.StudsOffset = Vector3.new(0, 2, 0)
        billboard.Parent = petPart
        
        local nameLabel = Instance.new("TextLabel")
        nameLabel.Size = UDim2.new(1, 0, 1, 0)
        nameLabel.BackgroundTransparency = 1
        nameLabel.Text = petData.name
        nameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
        nameLabel.TextSize = 12
        nameLabel.Font = Enum.Font.GothamBold
        nameLabel.TextStrokeTransparency = 0
        nameLabel.Parent = billboard
        
        table.insert(petModels[player.UserId], petModel)
    end
    
    -- เริ่ม AI สำหรับ Pets
    startPetAI(player)
end

-- ============================================================
-- PET AI (ติดตาม Player)
-- ============================================================

local petAIConnections = {}  -- {[userId] = connection}

function startPetAI(player)
    -- ยกเลิก AI เก่า
    if petAIConnections[player.UserId] then
        petAIConnections[player.UserId]:Disconnect()
    end
    
    local RunService = game:GetService("RunService")
    
    local connection = RunService.Heartbeat:Connect(function(dt)
        local character = player.Character
        if not character then return end
        
        local hrp = character:FindFirstChild("HumanoidRootPart")
        if not hrp then return end
        
        local models = petModels[player.UserId]
        if not models then return end
        
        local numPets = #models
        
        -- วาง Pets รอบๆ ผู้เล่นเป็นวงกลม
        for i, petModel in ipairs(models) do
            if not petModel.Parent then continue end
            
            local petPart = petModel.PrimaryPart
            if not petPart then continue end
            
            -- คำนวณ Target Position
            local angle = (i - 1) * (math.pi * 2 / numPets) + tick() * 0.5
            local radius = 5  -- ระยะห่างจาก Player
            
            local targetPos = hrp.Position + Vector3.new(
                math.cos(angle) * radius,
                0,
                math.sin(angle) * radius
            )
            
            -- Smooth ตาม Player
            local currentPos = petPart.Position
            local newPos = currentPos:Lerp(targetPos, dt * 5)
            
            -- Bounce Animation
            local bounceY = math.sin(tick() * 3 + i) * 0.3
            
            petPart.CFrame = CFrame.new(newPos + Vector3.new(0, bounceY + 1, 0))
        end
    end)
    
    petAIConnections[player.UserId] = connection
end

-- ============================================================
-- PLAYER EVENTS
-- ============================================================

Players.PlayerAdded:Connect(function(player)
    loadPetData(player)
    
    player.CharacterAdded:Connect(function(character)
        task.wait(1)  -- รอ Character โหลด
        updatePetModels(player)
    end)
    
    -- ส่งข้อมูลไปยัง Client
    events.updatePets:FireClient(player, playerPets[player.UserId])
end)

Players.PlayerRemoving:Connect(function(player)
    -- ลบ AI
    if petAIConnections[player.UserId] then
        petAIConnections[player.UserId]:Disconnect()
        petAIConnections[player.UserId] = nil
    end
    
    -- ลบ Models
    if petModels[player.UserId] then
        for _, model in ipairs(petModels[player.UserId]) do
            if model.Parent then model:Destroy() end
        end
        petModels[player.UserId] = nil
    end
    
    savePetData(player)
    playerPets[player.UserId] = nil
end)

-- ============================================================
-- FEED PET
-- ============================================================

events.feedPet.OnServerEvent:Connect(function(player, petUid)
    local data = playerPets[player.UserId]
    if not data then return end
    
    -- หา Pet
    local petInstance
    for _, pet in ipairs(data.inventory) do
        if pet.uid == petUid then
            petInstance = pet
            break
        end
    end
    
    if not petInstance then return end
    
    -- เพิ่ม Feed Count
    petInstance.feedCount = (petInstance.feedCount or 0) + 1
    
    -- ตรวจสอบ Evolution
    local petData = PetDatabase.getPet(petInstance.id)
    if petData and petData.evolutionRequirement and 
       petData.evolvesTo and
       petInstance.feedCount >= petData.evolutionRequirement then
        
        -- EVOLVE!
        petInstance.id = petData.evolvesTo
        petInstance.feedCount = 0
        petInstance.level = (petInstance.level or 1) + 1
        
        -- แจ้ง Client
        local newPetData = PetDatabase.getPet(petData.evolvesTo)
        events.updatePets:FireClient(player, data)
        
        print(string.format("%s's pet evolved to %s!", player.Name, petData.evolvesTo))
    end
    
    savePetData(player)
    events.updatePets:FireClient(player, data)
end)
```

---

## 3. Pet UI (Client)

### 3.1 Pet Inventory GUI

```lua
-- PetUI.lua (Local Script)
-- หน้า Inventory ของ Pet

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local PetDatabase = require(ReplicatedStorage:WaitForChild("PetDatabase"))
local petsFolder = ReplicatedStorage:WaitForChild("PetEvents")

local updatePets = petsFolder:WaitForChild("updatePets")
local equipPet = petsFolder:WaitForChild("equipPet")
local unequipPet = petsFolder:WaitForChild("unequipPet")
local feedPet = petsFolder:WaitForChild("feedPet")

-- State
local petData = nil
local selectedPet = nil

-- สร้าง UI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "PetUI"
screenGui.ResetOnSpawn = false
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.Enabled = false
screenGui.Parent = playerGui

local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 600, 0, 450)
mainFrame.Position = UDim2.new(0.5, -300, 0.5, -225)
mainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
mainFrame.BorderSizePixel = 0
mainFrame.Parent = screenGui

local frameCorner = Instance.new("UICorner")
frameCorner.CornerRadius = UDim.new(0, 12)
frameCorner.Parent = mainFrame

-- Title
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 45)
title.BackgroundColor3 = Color3.fromRGB(30, 30, 50)
title.Text = "🐾 Pets ของฉัน"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.TextSize = 20
title.Font = Enum.Font.GothamBold
title.BorderSizePixel = 0
title.Parent = mainFrame

-- Close Button
local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 30, 0, 30)
closeBtn.Position = UDim2.new(1, -40, 0.5, -15)
closeBtn.AnchorPoint = Vector2.new(0, 0)
closeBtn.BackgroundColor3 = Color3.fromRGB(200, 60, 60)
closeBtn.Text = "✕"
closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
closeBtn.TextSize = 14
closeBtn.Font = Enum.Font.GothamBold
closeBtn.BorderSizePixel = 0
closeBtn.Parent = title

local closeBtnCorner = Instance.new("UICorner")
closeBtnCorner.CornerRadius = UDim.new(0, 6)
closeBtnCorner.Parent = closeBtn

closeBtn.Activated:Connect(function()
    screenGui.Enabled = false
end)

-- Pet Grid
local petGrid = Instance.new("ScrollingFrame")
petGrid.Name = "PetGrid"
petGrid.Size = UDim2.new(0.6, -10, 1, -55)
petGrid.Position = UDim2.new(0, 5, 0, 50)
petGrid.BackgroundColor3 = Color3.fromRGB(15, 15, 25)
petGrid.BorderSizePixel = 0
petGrid.ScrollBarThickness = 4
petGrid.ScrollBarImageColor3 = Color3.fromRGB(100, 100, 150)
petGrid.CanvasSize = UDim2.new(0, 0, 0, 0)
petGrid.AutomaticCanvasSize = Enum.AutomaticSize.Y
petGrid.Parent = mainFrame

local petGridCorner = Instance.new("UICorner")
petGridCorner.CornerRadius = UDim.new(0, 8)
petGridCorner.Parent = petGrid

local gridLayout = Instance.new("UIGridLayout")
gridLayout.CellSize = UDim2.new(0, 80, 0, 90)
gridLayout.CellPadding = UDim2.new(0, 5, 0, 5)
gridLayout.SortOrder = Enum.SortOrder.LayoutOrder
gridLayout.Parent = petGrid

local gridPadding = Instance.new("UIPadding")
gridPadding.PaddingAll = UDim.new(0, 8)
gridPadding.Parent = petGrid

-- Pet Detail Panel
local detailPanel = Instance.new("Frame")
detailPanel.Name = "DetailPanel"
detailPanel.Size = UDim2.new(0.4, -10, 1, -55)
detailPanel.Position = UDim2.new(0.6, 5, 0, 50)
detailPanel.BackgroundColor3 = Color3.fromRGB(15, 15, 25)
detailPanel.BorderSizePixel = 0
detailPanel.Parent = mainFrame

local detailCorner = Instance.new("UICorner")
detailCorner.CornerRadius = UDim.new(0, 8)
detailCorner.Parent = detailPanel

-- Detail Labels
local detailIcon = Instance.new("TextLabel")
detailIcon.Size = UDim2.new(1, 0, 0, 70)
detailIcon.Position = UDim2.new(0, 0, 0, 10)
detailIcon.BackgroundTransparency = 1
detailIcon.TextSize = 50
detailIcon.Parent = detailPanel

local detailName = Instance.new("TextLabel")
detailName.Size = UDim2.new(1, -10, 0, 25)
detailName.Position = UDim2.new(0, 5, 0, 85)
detailName.BackgroundTransparency = 1
detailName.Text = "เลือก Pet"
detailName.TextColor3 = Color3.fromRGB(255, 255, 255)
detailName.TextSize = 16
detailName.Font = Enum.Font.GothamBold
detailName.Parent = detailPanel

local detailRarity = Instance.new("TextLabel")
detailRarity.Size = UDim2.new(1, -10, 0, 20)
detailRarity.Position = UDim2.new(0, 5, 0, 112)
detailRarity.BackgroundTransparency = 1
detailRarity.Text = ""
detailRarity.TextSize = 13
detailRarity.Font = Enum.Font.Gotham
detailRarity.Parent = detailPanel

local detailStats = Instance.new("TextLabel")
detailStats.Size = UDim2.new(1, -10, 0, 80)
detailStats.Position = UDim2.new(0, 5, 0, 135)
detailStats.BackgroundTransparency = 1
detailStats.Text = ""
detailStats.TextColor3 = Color3.fromRGB(180, 200, 180)
detailStats.TextSize = 12
detailStats.Font = Enum.Font.Gotham
detailStats.TextXAlignment = Enum.TextXAlignment.Left
detailStats.TextYAlignment = Enum.TextYAlignment.Top
detailStats.TextWrapped = true
detailStats.Parent = detailPanel

-- Equip Button
local equipBtn = Instance.new("TextButton")
equipBtn.Size = UDim2.new(1, -20, 0, 40)
equipBtn.Position = UDim2.new(0, 10, 1, -100)
equipBtn.BackgroundColor3 = Color3.fromRGB(60, 180, 100)
equipBtn.Text = "Equip"
equipBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
equipBtn.TextSize = 14
equipBtn.Font = Enum.Font.GothamBold
equipBtn.BorderSizePixel = 0
equipBtn.Visible = false
equipBtn.Parent = detailPanel

local equipBtnCorner = Instance.new("UICorner")
equipBtnCorner.CornerRadius = UDim.new(0, 8)
equipBtnCorner.Parent = equipBtn

-- Feed Button
local feedBtn = Instance.new("TextButton")
feedBtn.Size = UDim2.new(1, -20, 0, 35)
feedBtn.Position = UDim2.new(0, 10, 1, -55)
feedBtn.BackgroundColor3 = Color3.fromRGB(180, 120, 40)
feedBtn.Text = "🍖 Feed"
feedBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
feedBtn.TextSize = 13
feedBtn.Font = Enum.Font.GothamBold
feedBtn.BorderSizePixel = 0
feedBtn.Visible = false
feedBtn.Parent = detailPanel

local feedBtnCorner = Instance.new("UICorner")
feedBtnCorner.CornerRadius = UDim.new(0, 8)
feedBtnCorner.Parent = feedBtn

-- ฟังก์ชันสร้าง Pet Card
local function createPetCard(petInstance)
    local petInfo = PetDatabase.getPet(petInstance.id)
    if not petInfo then return end
    
    local card = Instance.new("TextButton")
    card.Name = "PetCard_" .. petInstance.uid
    card.BackgroundColor3 = Color3.fromRGB(30, 30, 45)
    card.BorderSizePixel = 0
    card.Text = ""
    card.Parent = petGrid
    
    local cardCorner = Instance.new("UICorner")
    cardCorner.CornerRadius = UDim.new(0, 6)
    cardCorner.Parent = card
    
    -- Pet Icon
    local icon = Instance.new("TextLabel")
    icon.Size = UDim2.new(1, 0, 0.6, 0)
    icon.BackgroundTransparency = 1
    icon.TextSize = 30
    icon.Text = petInfo.icon and "🐱" or "❓"  -- Placeholder
    icon.Parent = card
    
    -- Pet Name
    local nameLabel = Instance.new("TextLabel")
    nameLabel.Size = UDim2.new(1, -4, 0.25, 0)
    nameLabel.Position = UDim2.new(0, 2, 0.6, 0)
    nameLabel.BackgroundTransparency = 1
    nameLabel.Text = petInfo.name
    nameLabel.TextColor3 = Color3.fromRGB(220, 220, 220)
    nameLabel.TextSize = 10
    nameLabel.Font = Enum.Font.GothamBold
    nameLabel.TextTruncate = Enum.TextTruncate.AtEnd
    nameLabel.Parent = card
    
    -- Rarity Color Bar
    local rarityBar = Instance.new("Frame")
    rarityBar.Size = UDim2.new(1, 0, 0.08, 0)
    rarityBar.Position = UDim2.new(0, 0, 0.92, 0)
    rarityBar.BorderSizePixel = 0
    
    local rarityInfo = PetDatabase.getRarity(petInfo.rarity)
    if rarityInfo then
        rarityBar.BackgroundColor3 = rarityInfo.color
    else
        rarityBar.BackgroundColor3 = Color3.fromRGB(150, 150, 150)
    end
    rarityBar.Parent = card
    
    -- Equipped Indicator
    local equippedIndicator = Instance.new("Frame")
    equippedIndicator.Name = "EquippedIndicator"
    equippedIndicator.Size = UDim2.new(0, 18, 0, 18)
    equippedIndicator.Position = UDim2.new(1, -20, 0, 2)
    equippedIndicator.BackgroundColor3 = Color3.fromRGB(255, 200, 50)
    equippedIndicator.BorderSizePixel = 0
    equippedIndicator.Visible = false
    equippedIndicator.Parent = card
    
    local indCorner = Instance.new("UICorner")
    indCorner.CornerRadius = UDim.new(1, 0)
    indCorner.Parent = equippedIndicator
    
    local indLabel = Instance.new("TextLabel")
    indLabel.Size = UDim2.new(1, 0, 1, 0)
    indLabel.BackgroundTransparency = 1
    indLabel.Text = "E"
    indLabel.TextColor3 = Color3.fromRGB(0, 0, 0)
    indLabel.TextSize = 10
    indLabel.Font = Enum.Font.GothamBold
    indLabel.Parent = equippedIndicator
    
    -- Click Handler
    card.Activated:Connect(function()
        selectedPet = petInstance
        showPetDetail(petInstance)
    end)
    
    return card
end

-- แสดงรายละเอียด Pet
function showPetDetail(petInstance)
    local petInfo = PetDatabase.getPet(petInstance.id)
    if not petInfo then return end
    
    detailIcon.Text = "🐱"  -- ใส่ Icon จริง
    detailName.Text = petInfo.name
    
    local rarityInfo = PetDatabase.getRarity(petInfo.rarity)
    if rarityInfo then
        detailRarity.Text = rarityInfo.name
        detailRarity.TextColor3 = rarityInfo.color
    end
    
    -- แสดง Stats
    local statsText = ""
    if petInfo.stats then
        for stat, value in pairs(petInfo.stats) do
            if stat == "coinMultiplier" then
                statsText = statsText .. string.format("💰 Coin: x%.1f\n", value)
            elseif stat == "luck" then
                statsText = statsText .. string.format("🍀 Luck: +%d%%\n", value)
            elseif stat == "speed" then
                statsText = statsText .. string.format("⚡ Speed: +%d%%\n", value)
            end
        end
        
        -- Evolution Progress
        if petInfo.evolutionRequirement and petInfo.evolvesTo then
            statsText = statsText .. string.format(
                "\n🥚 Evolution: %d/%d",
                petInstance.feedCount or 0,
                petInfo.evolutionRequirement
            )
        end
    end
    
    detailStats.Text = statsText
    
    -- ตรวจว่า Equip อยู่หรือเปล่า
    local isEquipped = false
    if petData and petData.equipped then
        for _, eq in ipairs(petData.equipped) do
            if eq.uid == petInstance.uid then
                isEquipped = true
                break
            end
        end
    end
    
    equipBtn.Visible = true
    feedBtn.Visible = true
    
    if isEquipped then
        equipBtn.Text = "Unequip"
        equipBtn.BackgroundColor3 = Color3.fromRGB(180, 60, 60)
    else
        equipBtn.Text = "⚔️ Equip"
        equipBtn.BackgroundColor3 = Color3.fromRGB(60, 180, 100)
    end
    
    -- Equip/Unequip Handler
    equipBtn.Activated:Connect(function()
        if isEquipped then
            unequipPet:FireServer(petInstance.uid)
        else
            equipPet:FireServer(petInstance.uid)
        end
    end)
    
    -- Feed Handler
    feedBtn.Activated:Connect(function()
        feedPet:FireServer(petInstance.uid)
    end)
end

-- อัพเดท Grid
local function updatePetGrid(data)
    petData = data
    
    -- ล้าง Grid เก่า
    for _, child in ipairs(petGrid:GetChildren()) do
        if child:IsA("TextButton") then
            child:Destroy()
        end
    end
    
    -- สร้าง Cards ใหม่
    if data and data.inventory then
        for _, petInstance in ipairs(data.inventory) do
            createPetCard(petInstance)
        end
    end
end

-- รับ Update จาก Server
updatePets.OnClientEvent:Connect(function(data)
    updatePetGrid(data)
end)

-- Open/Close ด้วย P
local UserInputService = game:GetService("UserInputService")
UserInputService.InputBegan:Connect(function(input, processed)
    if processed then return end
    if input.KeyCode == Enum.KeyCode.P then
        screenGui.Enabled = not screenGui.Enabled
    end
end)
```

---

## 4. ข้อผิดพลาดที่พบบ่อย

### ❌ ข้อผิดพลาดที่ 1: ไม่ Validate Pet UID บน Server

```lua
-- ❌ แบบผิด
events.equipPet.OnServerEvent:Connect(function(player, petUid)
    -- เพิ่มโดยตรงโดยไม่ตรวจสอบว่า Pet เป็นของ Player จริงๆ!
    table.insert(data.equipped, {uid = petUid})
end)

-- ✅ แบบถูก: ตรวจสอบว่า Pet นั้นมีอยู่ใน Inventory ของ Player
events.equipPet.OnServerEvent:Connect(function(player, petUid)
    local data = playerPets[player.UserId]
    if not data then return end
    
    -- ค้นหาใน Inventory ก่อน
    local petInstance = nil
    for _, pet in ipairs(data.inventory) do
        if pet.uid == petUid then
            petInstance = pet
            break
        end
    end
    
    if not petInstance then
        warn("Pet UID ไม่พบ:", petUid, "Player:", player.Name)
        return  -- ไม่ดำเนินการต่อ
    end
    
    -- ตอนนี้ปลอดภัยแล้ว
    table.insert(data.equipped, petInstance)
end)
```

### ❌ ข้อผิดพลาดที่ 2: Pet AI ไม่ Cleanup เมื่อ Character Reset

```lua
-- ❌ แบบผิด: Connection ยังอยู่แม้ Character Reset
RunService.Heartbeat:Connect(function()
    local char = player.Character
    -- ถ้า char เป็น nil จะ Error!
    local hrp = char.HumanoidRootPart
end)

-- ✅ แบบถูก: ตรวจสอบทุกครั้ง
RunService.Heartbeat:Connect(function()
    local char = player.Character
    if not char then return end
    
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    -- ปลอดภัย
end)
```

---

## 5. แบบฝึกหัด

### แบบฝึกหัดที่ 1: Pet Trading System
สร้างระบบแลกเปลี่ยน Pet:
- ผู้เล่นสองคนเลือก Pet ที่ต้องการแลก
- ยืนยันทั้งสองฝ่าย
- Exchange อย่างปลอดภัย (Atomic Transaction)

### แบบฝึกหัดที่ 2: Pet Battle
สร้างระบบต่อสู้ Pet:
- เลือก Pet สู้กัน
- Stats ของ Pet มีผลต่อผลการต่อสู้
- EXP จากการต่อสู้ช่วย Evolution

### แบบฝึกหัดที่ 3: Pet Leaderboard
สร้าง Leaderboard ของ Pet:
- Rank ตาม Rarity สูงสุด
- Rank ตาม Total Pet Power
- แสดงใน Server Leaderboard

---

## สรุป

Pet System ที่ดีต้องมี:
1. **Server Authority**: Validate ทุกอย่างบน Server
2. **Persistent Data**: บันทึก Pet Data ด้วย DataStore
3. **Smooth AI**: Pet ตามผู้เล่นอย่าง Smooth
4. **Clear UI**: Inventory ดูง่าย Drag & Drop
5. **Progression**: Evolution ทำให้ Player อยากเล่นต่อ

ในส่วนถัดไป (Part 80) เราจะสร้าง Badge System ที่สมบูรณ์ พร้อมระบบ Achievement ที่น่าติดตาม
