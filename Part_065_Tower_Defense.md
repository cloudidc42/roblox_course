# Part 65: Tower Defense Game Mechanics

## บทนำ

Tower Defense เป็นประเภทเกมที่ผู้เล่นต้องวางหอคอยเพื่อป้องกันศัตรูที่เดินตามเส้นทาง ในบทนี้เราจะสร้าง Tower Defense Game ที่มีระบบครบครัน ตั้งแต่ระบบ Wave, Tower, Enemy ไปจนถึง Economy

---

## 65.1 โครงสร้างเกม

```
ServerScriptService
├── TDCore (Script)           -- ระบบหลัก
├── TowerSystem (Script)      -- ระบบหอคอย
├── EnemySystem (Script)      -- ระบบศัตรู
├── WaveManager (Script)      -- จัดการ Wave
└── EconomySystem (Script)    -- ระบบเงิน

ReplicatedStorage
├── Remotes
│   ├── PlaceTower (RemoteEvent)
│   ├── UpgradeTower (RemoteEvent)
│   ├── SellTower (RemoteEvent)
│   └── StartWave (RemoteEvent)
└── Modules
    ├── TowerData (ModuleScript)
    ├── EnemyData (ModuleScript)
    └── WaveData (ModuleScript)
```

---

## 65.2 ระบบ Tower

### 65.2.1 TowerData Module

```lua
-- ReplicatedStorage/Modules/TowerData.lua
-- ข้อมูล Tower ทั้งหมด

local TowerData = {}

local towers = {
    ["basic_tower"] = {
        name = "หอคอยธรรมดา",
        description = "หอคอยพื้นฐาน ราคาถูก แต่ไม่แข็งแกร่ง",
        cost = 100,
        range = 15,
        damage = 10,
        fireRate = 1,        -- ยิงกี่ครั้งต่อวินาที
        projectileSpeed = 30,
        projectileColor = Color3.fromRGB(255, 255, 0),
        targetType = "first",  -- "first", "last", "strongest", "weakest"
        sellValue = 50,
        size = Vector3.new(4, 6, 4),
        color = BrickColor.new("Medium stone grey"),
        
        -- Upgrade path (3 levels)
        upgrades = {
            {cost = 150, damage = 20, range = 18, name = "Level 2"},
            {cost = 300, damage = 35, range = 20, name = "Level 3"},
        }
    },
    
    ["sniper_tower"] = {
        name = "หอคอยสไนเปอร์",
        description = "ระยะไกลมาก ดาเมจสูง แต่ยิงช้า",
        cost = 300,
        range = 40,
        damage = 80,
        fireRate = 0.3,
        projectileSpeed = 100,
        projectileColor = Color3.fromRGB(255, 0, 0),
        targetType = "strongest",
        sellValue = 150,
        size = Vector3.new(4, 10, 4),
        color = BrickColor.new("Dark grey"),
        
        upgrades = {
            {cost = 400, damage = 150, range = 50, name = "Level 2"},
            {cost = 800, damage = 280, range = 60, name = "Level 3"},
        }
    },
    
    ["cannon_tower"] = {
        name = "ปืนใหญ่",
        description = "ยิงระเบิด ทำ AOE damage",
        cost = 500,
        range = 20,
        damage = 40,
        fireRate = 0.5,
        projectileSpeed = 25,
        projectileColor = Color3.fromRGB(50, 50, 50),
        targetType = "first",
        splashRadius = 8,    -- AOE radius
        sellValue = 250,
        size = Vector3.new(6, 6, 6),
        color = BrickColor.new("Reddish brown"),
        
        upgrades = {
            {cost = 600, damage = 70, splashRadius = 10, name = "Level 2"},
            {cost = 1200, damage = 120, splashRadius = 14, name = "Level 3"},
        }
    },
    
    ["freeze_tower"] = {
        name = "หอคอยน้ำแข็ง",
        description = "ทำให้ศัตรูช้าลง",
        cost = 400,
        range = 18,
        damage = 5,
        fireRate = 0.8,
        projectileSpeed = 30,
        projectileColor = Color3.fromRGB(100, 200, 255),
        targetType = "first",
        slowEffect = 0.5,    -- ลดความเร็ว 50%
        slowDuration = 2,    -- นาน 2 วินาที
        sellValue = 200,
        size = Vector3.new(4, 8, 4),
        color = BrickColor.new("Cyan"),
        
        upgrades = {
            {cost = 500, damage = 8, slowEffect = 0.7, slowDuration = 3, name = "Level 2"},
            {cost = 1000, damage = 15, slowEffect = 0.9, slowDuration = 4, name = "Level 3"},
        }
    },
    
    ["laser_tower"] = {
        name = "เลเซอร์",
        description = "ยิงต่อเนื่องไม่หยุด",
        cost = 800,
        range = 25,
        damage = 15,
        fireRate = 5,  -- ยิงเร็วมาก
        projectileSpeed = 200,
        projectileColor = Color3.fromRGB(0, 255, 0),
        targetType = "first",
        sellValue = 400,
        size = Vector3.new(4, 8, 4),
        color = BrickColor.new("Lime green"),
        
        upgrades = {
            {cost = 1000, damage = 25, range = 28, name = "Level 2"},
            {cost = 2000, damage = 45, range = 32, name = "Level 3"},
        }
    },
}

function TowerData.getTower(towerId)
    return towers[towerId]
end

function TowerData.getAllTowers()
    return towers
end

function TowerData.getUpgradeCost(towerId, currentLevel)
    local tower = towers[towerId]
    if not tower or not tower.upgrades then return nil end
    if currentLevel > #tower.upgrades then return nil end
    return tower.upgrades[currentLevel]
end

return TowerData
```

### 65.2.2 Tower System

```lua
-- ServerScriptService/TowerSystem.lua
-- ระบบวาง/อัพเกรด/ขาย Tower

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")

local TowerData = require(ReplicatedStorage.Modules.TowerData)

local Remotes = ReplicatedStorage:WaitForChild("Remotes")

-- เก็บข้อมูล Tower ที่วางแล้ว
local placedTowers = {}  -- [towerPart] = {towerId, level, owner, nextFireTime, ...}
local playerTowerCounts = {}  -- [userId] = count

-- จำนวน Tower สูงสุดต่อผู้เล่น
local MAX_TOWERS_PER_PLAYER = 20

-- สร้าง Tower Model
local function createTowerModel(towerData, position, playerId)
    local model = Instance.new("Model")
    model.Name = towerData.name
    
    -- ฐาน
    local base = Instance.new("Part")
    base.Name = "Base"
    base.Size = Vector3.new(towerData.size.X, 1, towerData.size.Z)
    base.Position = position
    base.Anchored = true
    base.BrickColor = BrickColor.new("Dark stone grey")
    base.Parent = model
    model.PrimaryPart = base
    
    -- ตัว Tower
    local body = Instance.new("Part")
    body.Name = "Body"
    body.Size = towerData.size
    body.Position = position + Vector3.new(0, towerData.size.Y/2, 0)
    body.Anchored = true
    body.BrickColor = towerData.color
    body.Parent = model
    
    -- ปืน/barrel
    local barrel = Instance.new("Part")
    barrel.Name = "Barrel"
    barrel.Size = Vector3.new(0.5, 0.5, 3)
    barrel.Position = position + Vector3.new(0, towerData.size.Y, 0)
    barrel.Anchored = true
    barrel.BrickColor = BrickColor.new("Black")
    barrel.Parent = model
    
    -- Range indicator (จะซ่อนตอนเล่นจริง)
    local rangeIndicator = Instance.new("Part")
    rangeIndicator.Name = "RangeIndicator"
    rangeIndicator.Size = Vector3.new(towerData.range * 2, 0.1, towerData.range * 2)
    rangeIndicator.Position = position
    rangeIndicator.Anchored = true
    rangeIndicator.CanCollide = false
    rangeIndicator.Transparency = 0.9
    rangeIndicator.BrickColor = BrickColor.new("Bright blue")
    rangeIndicator.Shape = Enum.PartType.Cylinder
    rangeIndicator.Parent = model
    
    -- เพิ่ม Billboard แสดงชื่อ
    local billboard = Instance.new("BillboardGui")
    billboard.Size = UDim2.new(0, 150, 0, 40)
    billboard.StudsOffset = Vector3.new(0, towerData.size.Y + 2, 0)
    billboard.Parent = body
    
    local nameLabel = Instance.new("TextLabel")
    nameLabel.Size = UDim2.new(1, 0, 1, 0)
    nameLabel.BackgroundTransparency = 1
    nameLabel.Text = towerData.name
    nameLabel.TextColor3 = Color3.new(1, 1, 1)
    nameLabel.TextScaled = true
    nameLabel.Font = Enum.Font.GothamBold
    nameLabel.Parent = billboard
    
    -- เพิ่ม Attributes
    model:SetAttribute("TowerId", "")  -- จะ set ทีหลัง
    model:SetAttribute("Level", 1)
    model:SetAttribute("OwnerId", playerId)
    
    model.Parent = workspace.Towers or workspace
    return model
end

-- วาง Tower
local function placeTower(player, towerId, position)
    local towerConfig = TowerData.getTower(towerId)
    if not towerConfig then return false, "ไม่พบ Tower" end
    
    -- ตรวจสอบจำนวน Tower
    local count = playerTowerCounts[player.UserId] or 0
    if count >= MAX_TOWERS_PER_PLAYER then
        return false, "วาง Tower ได้สูงสุด " .. MAX_TOWERS_PER_PLAYER .. " อัน"
    end
    
    -- ตรวจสอบเงิน
    local EconomySystem = require(script.Parent.EconomySystem)
    if not EconomySystem.hasCoins(player, towerConfig.cost) then
        return false, "เงินไม่พอ! ต้องการ " .. towerConfig.cost .. " Gold"
    end
    
    -- ตรวจสอบว่าวางในพื้นที่ที่อนุญาต
    local allowedZone = workspace:FindFirstChild("TowerZone")
    if allowedZone then
        local zonePos = allowedZone.Position
        local zoneSize = allowedZone.Size
        
        if math.abs(position.X - zonePos.X) > zoneSize.X/2 or
           math.abs(position.Z - zonePos.Z) > zoneSize.Z/2 then
            return false, "วาง Tower ได้เฉพาะในพื้นที่สีเขียว!"
        end
    end
    
    -- ตรวจสอบว่าซ้อนกับ Tower อื่นหรือไม่
    for _, existingTower in ipairs(workspace.Towers:GetChildren()) do
        if existingTower.PrimaryPart then
            local dist = (existingTower.PrimaryPart.Position - position).Magnitude
            if dist < 5 then
                return false, "ตำแหน่งนี้มี Tower อยู่แล้ว!"
            end
        end
    end
    
    -- หักเงิน
    EconomySystem.removeCoins(player, towerConfig.cost)
    
    -- สร้าง Tower
    local towerModel = createTowerModel(towerConfig, position, player.UserId)
    towerModel:SetAttribute("TowerId", towerId)
    
    -- เพิ่มในตาราง
    placedTowers[towerModel] = {
        towerId = towerId,
        level = 1,
        owner = player,
        nextFireTime = 0,
        currentConfig = towerConfig
    }
    
    playerTowerCounts[player.UserId] = count + 1
    
    print(player.Name .. " วาง " .. towerConfig.name .. " ที่ " .. tostring(position))
    return true, towerModel
end

-- อัพเกรด Tower
local function upgradeTower(player, towerModel)
    local towerInfo = placedTowers[towerModel]
    if not towerInfo then return false, "ไม่พบ Tower" end
    
    -- ตรวจสอบว่าเป็นเจ้าของ
    if towerInfo.owner ~= player then
        return false, "นี่ไม่ใช่ Tower ของคุณ"
    end
    
    local upgradeData = TowerData.getUpgradeCost(towerInfo.towerId, towerInfo.level)
    if not upgradeData then
        return false, "Tower นี้อัพเกรดสูงสุดแล้ว"
    end
    
    -- ตรวจสอบเงิน
    local EconomySystem = require(script.Parent.EconomySystem)
    if not EconomySystem.hasCoins(player, upgradeData.cost) then
        return false, "เงินไม่พอ! ต้องการ " .. upgradeData.cost
    end
    
    EconomySystem.removeCoins(player, upgradeData.cost)
    
    -- อัพเดท Tower
    towerInfo.level = towerInfo.level + 1
    towerModel:SetAttribute("Level", towerInfo.level)
    
    -- อัพเดท stats
    local baseConfig = TowerData.getTower(towerInfo.towerId)
    local newConfig = {}
    for k, v in pairs(baseConfig) do
        newConfig[k] = v
    end
    for k, v in pairs(upgradeData) do
        newConfig[k] = v
    end
    towerInfo.currentConfig = newConfig
    
    -- เปลี่ยนสี Tower ตาม level
    local levelColors = {
        BrickColor.new("Medium stone grey"),  -- Level 1
        BrickColor.new("Bright blue"),        -- Level 2
        BrickColor.new("Bright orange"),      -- Level 3
        BrickColor.new("Bright red"),         -- Level 4
    }
    
    local body = towerModel:FindFirstChild("Body")
    if body then
        body.BrickColor = levelColors[math.min(towerInfo.level, #levelColors)]
    end
    
    -- อัพเดท Range Indicator
    local rangeIndicator = towerModel:FindFirstChild("RangeIndicator")
    if rangeIndicator and newConfig.range then
        rangeIndicator.Size = Vector3.new(newConfig.range * 2, 0.1, newConfig.range * 2)
    end
    
    print(player.Name .. " อัพเกรด " .. baseConfig.name .. " เป็น " .. upgradeData.name)
    return true
end

-- ขาย Tower
local function sellTower(player, towerModel)
    local towerInfo = placedTowers[towerModel]
    if not towerInfo then return false end
    
    if towerInfo.owner ~= player then return false end
    
    local towerConfig = TowerData.getTower(towerInfo.towerId)
    local sellValue = towerConfig.sellValue
    
    -- คำนวณราคาขาย (เพิ่มตาม upgrade)
    for i = 2, towerInfo.level do
        local upgrade = TowerData.getUpgradeCost(towerInfo.towerId, i - 1)
        if upgrade then
            sellValue = sellValue + math.floor(upgrade.cost * 0.5)
        end
    end
    
    local EconomySystem = require(script.Parent.EconomySystem)
    EconomySystem.addCoins(player, sellValue)
    
    -- ลบ Tower
    placedTowers[towerModel] = nil
    playerTowerCounts[player.UserId] = (playerTowerCounts[player.UserId] or 1) - 1
    towerModel:Destroy()
    
    print(player.Name .. " ขาย Tower ได้ " .. sellValue .. " Gold")
    return true, sellValue
end

-- ระบบยิง Tower (รันทุก tick)
RunService.Heartbeat:Connect(function()
    local now = tick()
    
    for towerModel, towerInfo in pairs(placedTowers) do
        if not towerModel.Parent then
            placedTowers[towerModel] = nil
            continue
        end
        
        if now < towerInfo.nextFireTime then continue end
        
        local config = towerInfo.currentConfig
        local barrel = towerModel:FindFirstChild("Barrel")
        if not barrel then continue end
        
        -- หา target ในระยะ
        local target = findTarget(barrel.Position, config.range, config.targetType)
        if not target then continue end
        
        -- หมุน barrel ไปหา target
        local direction = (target.HumanoidRootPart.Position - barrel.Position).Unit
        barrel.CFrame = CFrame.lookAt(barrel.Position, target.HumanoidRootPart.Position)
        
        -- ยิง Projectile
        fireProjectile(barrel.Position, target, config)
        
        towerInfo.nextFireTime = now + (1 / config.fireRate)
    end
end)

-- หา target ในระยะ
function findTarget(position, range, targetType)
    local enemies = workspace:FindFirstChild("Enemies")
    if not enemies then return nil end
    
    local validTargets = {}
    
    for _, enemy in ipairs(enemies:GetChildren()) do
        if enemy:FindFirstChild("HumanoidRootPart") then
            local dist = (enemy.HumanoidRootPart.Position - position).Magnitude
            if dist <= range then
                local humanoid = enemy:FindFirstChildWhichIsA("Humanoid")
                if humanoid and humanoid.Health > 0 then
                    table.insert(validTargets, {
                        model = enemy,
                        distance = dist,
                        health = humanoid.Health,
                        progress = enemy:GetAttribute("PathProgress") or 0
                    })
                end
            end
        end
    end
    
    if #validTargets == 0 then return nil end
    
    -- เลือก target ตาม strategy
    if targetType == "first" then
        -- เลือกตัวที่ไปไกลที่สุดในเส้นทาง
        table.sort(validTargets, function(a, b) return a.progress > b.progress end)
    elseif targetType == "last" then
        table.sort(validTargets, function(a, b) return a.progress < b.progress end)
    elseif targetType == "strongest" then
        table.sort(validTargets, function(a, b) return a.health > b.health end)
    elseif targetType == "weakest" then
        table.sort(validTargets, function(a, b) return a.health < b.health end)
    elseif targetType == "nearest" then
        table.sort(validTargets, function(a, b) return a.distance < b.distance end)
    end
    
    return validTargets[1].model
end

-- ยิง Projectile
function fireProjectile(startPos, target, config)
    local projectile = Instance.new("Part")
    projectile.Name = "Projectile"
    projectile.Size = Vector3.new(0.5, 0.5, 1)
    projectile.Position = startPos
    projectile.Anchored = true
    projectile.CanCollide = false
    projectile.BrickColor = BrickColor.fromColor3(config.projectileColor)
    projectile.Material = Enum.Material.Neon
    projectile.CastShadow = false
    projectile.Parent = workspace
    
    -- ลบอัตโนมัติ
    game:GetService("Debris"):AddItem(projectile, 5)
    
    local targetHRP = target:FindFirstChild("HumanoidRootPart")
    if not targetHRP then
        projectile:Destroy()
        return
    end
    
    -- เคลื่อนที่ไปยัง target
    local RunService = game:GetService("RunService")
    local connection
    
    connection = RunService.Heartbeat:Connect(function(dt)
        if not projectile.Parent or not targetHRP.Parent then
            connection:Disconnect()
            projectile:Destroy()
            return
        end
        
        local direction = (targetHRP.Position - projectile.Position)
        local dist = direction.Magnitude
        
        if dist < 2 then
            -- กระแทก!
            connection:Disconnect()
            
            -- ทำ damage
            local humanoid = target:FindFirstChildWhichIsA("Humanoid")
            if humanoid and humanoid.Health > 0 then
                humanoid:TakeDamage(config.damage)
                
                -- Slow effect
                if config.slowEffect then
                    applySlowEffect(target, config.slowEffect, config.slowDuration or 2)
                end
                
                -- AOE (Cannon)
                if config.splashRadius then
                    applySplashDamage(projectile.Position, config.splashRadius, config.damage * 0.5)
                end
            end
            
            projectile:Destroy()
        else
            -- เคลื่อนที่
            local step = direction.Unit * config.projectileSpeed * dt
            projectile.CFrame = CFrame.new(projectile.Position + step,
                projectile.Position + step + direction.Unit)
        end
    end)
end

-- Slow Effect
function applySlowEffect(enemy, slowAmount, duration)
    local humanoid = enemy:FindFirstChildWhichIsA("Humanoid")
    if not humanoid then return end
    
    local originalSpeed = enemy:GetAttribute("BaseWalkSpeed") or humanoid.WalkSpeed
    enemy:SetAttribute("BaseWalkSpeed", originalSpeed)
    
    humanoid.WalkSpeed = originalSpeed * (1 - slowAmount)
    
    -- เปลี่ยนสีตัวละครเป็นน้ำแข็ง
    for _, part in ipairs(enemy:GetDescendants()) do
        if part:IsA("BasePart") then
            part.BrickColor = BrickColor.new("Cyan")
        end
    end
    
    task.delay(duration, function()
        if enemy.Parent then
            humanoid.WalkSpeed = originalSpeed
            -- คืนสีเดิม
        end
    end)
end

-- AOE Damage
function applySplashDamage(position, radius, damage)
    local enemies = workspace:FindFirstChild("Enemies")
    if not enemies then return end
    
    for _, enemy in ipairs(enemies:GetChildren()) do
        if enemy:FindFirstChild("HumanoidRootPart") then
            local dist = (enemy.HumanoidRootPart.Position - position).Magnitude
            if dist <= radius then
                local humanoid = enemy:FindFirstChildWhichIsA("Humanoid")
                if humanoid then
                    humanoid:TakeDamage(damage)
                end
            end
        end
    end
end

-- Remote handlers
local placeTowerRemote = Remotes:WaitForChild("PlaceTower")
local upgradeTowerRemote = Remotes:WaitForChild("UpgradeTower")
local sellTowerRemote = Remotes:WaitForChild("SellTower")

placeTowerRemote.OnServerEvent:Connect(function(player, towerId, position)
    local success, result = placeTower(player, towerId, position)
    if not success then
        local notifyEvent = Remotes:FindFirstChild("Notification")
        if notifyEvent then
            notifyEvent:FireClient(player, result, "error")
        end
    end
end)

upgradeTowerRemote.OnServerEvent:Connect(function(player, towerModel)
    local success, message = upgradeTower(player, towerModel)
    if not success then
        local notifyEvent = Remotes:FindFirstChild("Notification")
        if notifyEvent then
            notifyEvent:FireClient(player, message, "error")
        end
    end
end)

sellTowerRemote.OnServerEvent:Connect(function(player, towerModel)
    sellTower(player, towerModel)
end)
```

---

## 65.3 ระบบ Enemy

```lua
-- ServerScriptService/EnemySystem.lua
-- ระบบศัตรู

local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")

local EnemyData = require(ReplicatedStorage.Modules.EnemyData)

-- Path waypoints (กำหนดใน workspace)
local pathWaypoints = {}

-- โหลด waypoints จาก workspace
local function loadWaypoints()
    local waypointFolder = workspace:FindFirstChild("EnemyPath")
    if not waypointFolder then
        warn("ไม่พบ EnemyPath folder!")
        return
    end
    
    -- เรียง waypoints ตามหมายเลข
    local waypointList = {}
    for _, wp in ipairs(waypointFolder:GetChildren()) do
        local num = tonumber(wp.Name:match("WP(%d+)")) or 0
        table.insert(waypointList, {num = num, pos = wp.Position})
    end
    table.sort(waypointList, function(a, b) return a.num < b.num end)
    
    for _, wp in ipairs(waypointList) do
        table.insert(pathWaypoints, wp.pos)
    end
    
    print("โหลด " .. #pathWaypoints .. " waypoints")
end

-- สร้าง Enemy
local function spawnEnemy(enemyType, spawnPos)
    local enemyConfig = EnemyData.getEnemy(enemyType)
    if not enemyConfig then return end
    
    local model = Instance.new("Model")
    model.Name = enemyType
    
    -- Root
    local root = Instance.new("Part")
    root.Name = "HumanoidRootPart"
    root.Size = Vector3.new(2, 2, 1)
    root.Position = spawnPos
    root.Anchored = false
    root.CanCollide = false
    root.Transparency = 1
    root.Parent = model
    model.PrimaryPart = root
    
    -- Body
    local body = Instance.new("Part")
    body.Name = "Torso"
    body.Size = enemyConfig.size or Vector3.new(3, 3, 3)
    body.Position = spawnPos
    body.BrickColor = enemyConfig.color
    body.Material = enemyConfig.material or Enum.Material.SmoothPlastic
    body.CanCollide = false
    body.Parent = model
    
    -- Head
    local head = Instance.new("Part")
    head.Name = "Head"
    head.Size = Vector3.new(2, 2, 2)
    head.Position = spawnPos + Vector3.new(0, 2.5, 0)
    head.BrickColor = enemyConfig.headColor or enemyConfig.color
    head.CanCollide = false
    head.Parent = model
    
    -- Humanoid
    local humanoid = Instance.new("Humanoid")
    humanoid.MaxHealth = enemyConfig.health
    humanoid.Health = enemyConfig.health
    humanoid.WalkSpeed = enemyConfig.speed
    humanoid.DisplayDistanceType = Enum.HumanoidDisplayDistanceType.Always
    humanoid.HealthDisplayDistanceType = Enum.HumanoidHealthDisplayDistanceType.Always
    humanoid.NameDisplayDistance = 50
    humanoid.Parent = model
    
    -- Health Bar custom
    local healthBar = Instance.new("BillboardGui")
    healthBar.Size = UDim2.new(0, 100, 0, 20)
    healthBar.StudsOffset = Vector3.new(0, 4, 0)
    healthBar.Parent = body
    
    local bg = Instance.new("Frame")
    bg.Size = UDim2.new(1, 0, 1, 0)
    bg.BackgroundColor3 = Color3.new(0.3, 0, 0)
    bg.BorderSizePixel = 0
    bg.Parent = healthBar
    
    local bar = Instance.new("Frame")
    bar.Name = "Bar"
    bar.Size = UDim2.new(1, 0, 1, 0)
    bar.BackgroundColor3 = Color3.new(0, 1, 0)
    bar.BorderSizePixel = 0
    bar.Parent = bg
    
    -- Attributes
    model:SetAttribute("EnemyType", enemyType)
    model:SetAttribute("PathProgress", 0)
    model:SetAttribute("Reward", enemyConfig.reward)
    model:SetAttribute("Damage", enemyConfig.damage)
    model:SetAttribute("BaseWalkSpeed", enemyConfig.speed)
    
    model.Parent = workspace.Enemies or workspace
    
    -- ติดตาม path
    local waypointIndex = 1
    
    humanoid.HealthChanged:Connect(function(hp)
        -- อัพเดท health bar
        local maxHP = humanoid.MaxHealth
        local percent = hp / maxHP
        local healthBar = body:FindFirstChild("BillboardGui")
        if healthBar then
            local bar = healthBar:FindFirstChild("Frame"):FindFirstChild("Bar")
            if bar then
                bar.Size = UDim2.new(percent, 0, 1, 0)
                bar.BackgroundColor3 = Color3.new(1 - percent, percent, 0)
            end
        end
    end)
    
    -- เดินตาม path
    local connection
    connection = RunService.Heartbeat:Connect(function(dt)
        if not model.Parent then
            connection:Disconnect()
            return
        end
        
        if waypointIndex > #pathWaypoints then
            -- ถึงปลายทาง! ทำ damage ให้เซิร์ฟเวอร์
            connection:Disconnect()
            
            local TDCore = require(script.Parent.TDCore)
            TDCore.enemyReachedEnd(enemyConfig.damage)
            
            model:Destroy()
            return
        end
        
        local targetWP = pathWaypoints[waypointIndex]
        local currentPos = root.Position
        local direction = (targetWP - currentPos)
        local dist = direction.Magnitude
        
        if dist < 2 then
            waypointIndex = waypointIndex + 1
            model:SetAttribute("PathProgress", waypointIndex)
        else
            -- เคลื่อนที่
            local moveDir = direction.Unit
            local speed = humanoid.WalkSpeed * dt
            local newPos = currentPos + moveDir * speed
            
            root.CFrame = CFrame.new(newPos, newPos + moveDir)
            body.CFrame = CFrame.new(newPos, newPos + moveDir)
            head.CFrame = CFrame.new(newPos + Vector3.new(0, 2.5, 0), newPos + Vector3.new(0, 2.5, 0) + moveDir)
        end
        
        -- ตาย
        if humanoid.Health <= 0 then
            connection:Disconnect()
            
            local TDCore = require(script.Parent.TDCore)
            TDCore.enemyKilled(enemyConfig.reward)
            
            -- Death effect
            for _, part in ipairs(model:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.Anchored = true
                    game:GetService("TweenService"):Create(part,
                        TweenInfo.new(0.5),
                        {Transparency = 1}
                    ):Play()
                end
            end
            
            game:GetService("Debris"):AddItem(model, 1)
        end
    end)
    
    return model
end

loadWaypoints()

return {spawnEnemy = spawnEnemy}
```

---

## 65.4 ระบบ Wave Manager

```lua
-- ServerScriptService/WaveManager.lua
-- จัดการ Waves

local ReplicatedStorage = game:GetService("ReplicatedStorage")
local WaveData = require(ReplicatedStorage.Modules.WaveData)

local EnemySystem = require(script.Parent.EnemySystem)

local Remotes = ReplicatedStorage:WaitForChild("Remotes")
local waveUpdateEvent = Instance.new("RemoteEvent")
waveUpdateEvent.Name = "WaveUpdate"
waveUpdateEvent.Parent = Remotes

local currentWave = 0
local isWaveActive = false
local enemiesRemaining = 0

-- เริ่ม Wave
local function startWave()
    if isWaveActive then return end
    
    currentWave = currentWave + 1
    local waveConfig = WaveData.getWave(currentWave)
    
    if not waveConfig then
        -- ชนะเกม!
        print("ผู้เล่นชนะ! ผ่านทุก wave แล้ว")
        return
    end
    
    isWaveActive = true
    enemiesRemaining = 0
    
    -- นับ enemies ทั้งหมด
    for _, group in ipairs(waveConfig.enemyGroups) do
        enemiesRemaining = enemiesRemaining + group.count
    end
    
    -- แจ้ง Client
    waveUpdateEvent:FireAllClients({
        wave = currentWave,
        totalEnemies = enemiesRemaining,
        status = "started"
    })
    
    print("Wave " .. currentWave .. " เริ่มแล้ว! " .. enemiesRemaining .. " enemies")
    
    -- Spawn enemies
    for _, group in ipairs(waveConfig.enemyGroups) do
        task.spawn(function()
            task.wait(group.delay or 0)
            
            for i = 1, group.count do
                local spawnPos = pathWaypoints[1] + Vector3.new(
                    math.random(-3, 3), 0, math.random(-3, 3)
                )
                
                EnemySystem.spawnEnemy(group.enemyType, spawnPos)
                task.wait(group.spawnInterval or 1)
            end
        end)
    end
end

-- Enemy ถึงปลายทาง
local function enemyReachedEnd(damage)
    enemiesRemaining = enemiesRemaining - 1
    
    local TDCore = require(script.Parent.TDCore)
    TDCore.takeLivesLoss(damage)
    
    checkWaveComplete()
end

-- Enemy ถูกฆ่า
local function enemyKilled(reward)
    enemiesRemaining = enemiesRemaining - 1
    
    local TDCore = require(script.Parent.TDCore)
    TDCore.addGold(reward)
    
    checkWaveComplete()
end

function checkWaveComplete()
    if enemiesRemaining <= 0 and isWaveActive then
        isWaveActive = false
        
        -- Bonus gold
        local TDCore = require(script.Parent.TDCore)
        TDCore.addGold(50 * currentWave)
        
        waveUpdateEvent:FireAllClients({
            wave = currentWave,
            status = "complete"
        })
        
        print("Wave " .. currentWave .. " เสร็จสิ้น!")
    end
end

return {
    startWave = startWave,
    enemyReachedEnd = enemyReachedEnd,
    enemyKilled = enemyKilled
}
```

---

## 65.5 Wave Data

```lua
-- ReplicatedStorage/Modules/WaveData.lua

local WaveData = {}

local waves = {
    -- Wave 1: ง่ายมาก
    {
        enemyGroups = {
            {enemyType = "basic_enemy", count = 5, spawnInterval = 2, delay = 0}
        }
    },
    -- Wave 2
    {
        enemyGroups = {
            {enemyType = "basic_enemy", count = 8, spawnInterval = 1.5, delay = 0},
            {enemyType = "fast_enemy", count = 3, spawnInterval = 1, delay = 5}
        }
    },
    -- Wave 3: เริ่มยาก
    {
        enemyGroups = {
            {enemyType = "basic_enemy", count = 10, spawnInterval = 1, delay = 0},
            {enemyType = "tank_enemy", count = 2, spawnInterval = 3, delay = 3}
        }
    },
    -- Wave 5: Boss Wave
    {
        enemyGroups = {
            {enemyType = "basic_enemy", count = 15, spawnInterval = 0.8, delay = 0},
            {enemyType = "boss_enemy", count = 1, spawnInterval = 1, delay = 10}
        }
    },
}

function WaveData.getWave(waveNumber)
    -- ถ้าเกิน waves ที่กำหนด ให้ generate แบบ dynamic
    if waveNumber <= #waves then
        return waves[waveNumber]
    end
    
    -- Dynamic wave generation
    local difficulty = math.floor(waveNumber / 5) + 1
    return {
        enemyGroups = {
            {
                enemyType = "basic_enemy",
                count = 10 + waveNumber * 2,
                spawnInterval = math.max(0.3, 1 - waveNumber * 0.05),
                delay = 0
            }
        }
    }
end

return WaveData
```

---

## 65.6 ข้อผิดพลาดที่พบบ่อย

```lua
-- ❌ ผิด: ใช้ Humanoid:MoveTo() เพื่อให้ศัตรูเดิน
-- ทำให้ศัตรูวิ่งผ่านกำแพงและ path ไม่แม่นยำ

-- ✓ ถูก: เคลื่อนที่ด้วยการ set CFrame โดยตรงตาม waypoints
-- ดังที่แสดงในโค้ด EnemySystem ด้านบน
```

---

## 65.7 แบบฝึกหัด

### แบบฝึกหัดที่ 1: เพิ่ม Tower ใหม่
สร้าง "Poison Tower" ที่:
- ทำ damage ช้าๆ ต่อเนื่อง (DoT)
- ตัวกระสุนสีเขียว
- ราคา 600 Gold

### แบบฝึกหัดที่ 2: เพิ่ม Enemy พิเศษ
สร้าง "Invisible Enemy" ที่:
- มองไม่เห็นด้วยตาเปล่า
- ต้องใช้ Radar Tower ถึงจะยิงได้

### แบบฝึกหัดที่ 3: Map หลายเส้นทาง
สร้าง Map ที่มี 2 เส้นทาง:
- เส้นทางบน: ศัตรูเร็ว แต่ผ่านน้อย
- เส้นทางล่าง: ศัตรูช้า แต่ผ่านมาก

---

## สรุป

ในบทนี้เราได้สร้าง Tower Defense ที่มี:
- Tower หลายประเภทพร้อม upgrade path
- Enemy system ที่เดินตาม waypoints
- Wave manager
- AOE, Slow, และ Projectile systems

ในบทถัดไปเราจะสร้าง Racing Game!
