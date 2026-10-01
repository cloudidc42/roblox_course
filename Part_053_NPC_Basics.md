# Part 53: NPC พื้นฐาน - สร้าง AI เบื้องต้น

## บทนำ

NPC (Non-Player Character) คือตัวละครที่ควบคุมโดย AI ในเกม ตั้งแต่ผู้ขายของในร้านไปจนถึงศัตรูที่ต้องสู้ ในบทนี้เราจะเรียนรู้การสร้าง NPC พื้นฐานด้วย AI ง่ายๆ ที่ทำให้เกมมีชีวิตชีวา

## ประเภทของ NPC

1. **Friendly NPC** - ผู้ช่วย, ผู้ขายของ, Quest givers
2. **Enemy NPC** - ศัตรูที่โจมตีผู้เล่น
3. **Neutral NPC** - ไม่โจมตีก่อนแต่ตอบโต้เมื่อโดนโจมตี
4. **Ambient NPC** - ตัวละครประกอบฉาก (ชาวบ้าน)

## การสร้าง NPC Model

```
NPC Model Structure:
Model (NPC_Slime)
├── HumanoidRootPart (ส่วนหลัก)
├── Head
├── Torso / UpperTorso
├── LeftArm / RightArm
├── LeftLeg / RightLeg
├── Humanoid (ควบคุม animation, health)
│   └── Animator
└── Script (NPC AI - Script ใน Model)
```

## สร้าง NPC พื้นฐาน

### Script ใน NPC Model

```lua
-- Script ใน NPC Model
-- NPC_BasicAI.lua

local NPC = script.Parent  -- Model ของ NPC
local humanoid = NPC:WaitForChild("Humanoid")
local rootPart = NPC:WaitForChild("HumanoidRootPart")

-- ===== Configuration =====
local CONFIG = {
    -- Combat
    health = 50,
    maxHealth = 50,
    damage = 10,
    attackRange = 5,        -- ระยะโจมตี
    detectionRange = 30,    -- ระยะตรวจจับผู้เล่น
    attackCooldown = 2,     -- วินาที
    
    -- Movement
    walkSpeed = 12,
    runSpeed = 18,
    
    -- Behavior
    aggroType = "hostile",  -- hostile, neutral, friendly
    respawnTime = 10,       -- วินาที
    
    -- Loot
    expReward = 25,
    coinRange = { min = 5, max = 20 }
}

-- ===== State Machine =====
local State = {
    IDLE = "Idle",
    PATROL = "Patrol",
    CHASE = "Chase",
    ATTACK = "Attack",
    FLEE = "Flee",
    DEAD = "Dead"
}

local currentState = State.IDLE
local target = nil
local lastAttackTime = 0
local spawnPosition = rootPart.Position

-- ===== Services =====
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

-- ===== Initialize =====
humanoid.MaxHealth = CONFIG.maxHealth
humanoid.Health = CONFIG.health
humanoid.WalkSpeed = CONFIG.walkSpeed

-- ===== Helper Functions =====

local function getDistance(pos1, pos2)
    return (pos1 - pos2).Magnitude
end

local function findNearestPlayer()
    local nearest = nil
    local nearestDist = CONFIG.detectionRange
    
    for _, player in ipairs(Players:GetPlayers()) do
        local character = player.Character
        if not character then continue end
        
        local playerRoot = character:FindFirstChild("HumanoidRootPart")
        local playerHumanoid = character:FindFirstChild("Humanoid")
        
        if not playerRoot or not playerHumanoid then continue end
        if playerHumanoid.Health <= 0 then continue end
        
        local dist = getDistance(rootPart.Position, playerRoot.Position)
        
        if dist < nearestDist then
            nearest = player
            nearestDist = dist
        end
    end
    
    return nearest, nearestDist
end

-- ===== State Functions =====

local function doIdle()
    humanoid.WalkSpeed = CONFIG.walkSpeed
    
    -- ยืนนิ่งสักพักแล้วค่อย patrol
    task.wait(math.random(2, 5))
    
    if currentState == State.IDLE then
        currentState = State.PATROL
    end
end

-- Patrol points (สร้างรอบจุด spawn)
local patrolPoints = {}
local currentPatrolPoint = 1

local function generatePatrolPoints()
    local radius = 15
    local numPoints = 4
    
    for i = 1, numPoints do
        local angle = (i - 1) * (2 * math.pi / numPoints)
        local x = spawnPosition.X + radius * math.cos(angle)
        local z = spawnPosition.Z + radius * math.sin(angle)
        
        -- Raycast ลงพื้น
        local rayOrigin = Vector3.new(x, spawnPosition.Y + 10, z)
        local rayDir = Vector3.new(0, -20, 0)
        local result = workspace:Raycast(rayOrigin, rayDir)
        
        local y = result and result.Position.Y + 2 or spawnPosition.Y
        table.insert(patrolPoints, Vector3.new(x, y, z))
    end
end

generatePatrolPoints()

local function doPatrol()
    if #patrolPoints == 0 then
        currentState = State.IDLE
        return
    end
    
    local targetPoint = patrolPoints[currentPatrolPoint]
    humanoid:MoveTo(targetPoint)
    
    -- รอให้ถึงจุด หรือ timeout
    local moveTimeout = 5
    local elapsed = 0
    
    while elapsed < moveTimeout do
        task.wait(0.1)
        elapsed = elapsed + 0.1
        
        if currentState ~= State.PATROL then return end
        
        if getDistance(rootPart.Position, targetPoint) < 3 then
            break
        end
    end
    
    -- ไปจุดถัดไป
    currentPatrolPoint = (currentPatrolPoint % #patrolPoints) + 1
    task.wait(1)  -- หยุดนิดนึงก่อนไปต่อ
end

local function doChase()
    if not target or not target.Character then
        currentState = State.IDLE
        target = nil
        return
    end
    
    local targetRoot = target.Character:FindFirstChild("HumanoidRootPart")
    local targetHumanoid = target.Character:FindFirstChild("Humanoid")
    
    if not targetRoot or not targetHumanoid or targetHumanoid.Health <= 0 then
        currentState = State.IDLE
        target = nil
        return
    end
    
    humanoid.WalkSpeed = CONFIG.runSpeed
    humanoid:MoveTo(targetRoot.Position)
    
    local dist = getDistance(rootPart.Position, targetRoot.Position)
    
    -- ถ้าใกล้พอโจมตี
    if dist <= CONFIG.attackRange then
        currentState = State.ATTACK
        return
    end
    
    -- ถ้าไกลเกินไป หยุดไล่
    if dist > CONFIG.detectionRange * 1.5 then
        currentState = State.IDLE
        target = nil
        humanoid.WalkSpeed = CONFIG.walkSpeed
        return
    end
    
    task.wait(0.1)
end

local function doAttack()
    if not target or not target.Character then
        currentState = State.IDLE
        return
    end
    
    local targetRoot = target.Character:FindFirstChild("HumanoidRootPart")
    local targetHumanoid = target.Character:FindFirstChild("Humanoid")
    
    if not targetRoot or not targetHumanoid or targetHumanoid.Health <= 0 then
        currentState = State.IDLE
        target = nil
        return
    end
    
    local dist = getDistance(rootPart.Position, targetRoot.Position)
    
    -- ถ้าไกลเกินโจมตีได้ กลับไป chase
    if dist > CONFIG.attackRange * 1.5 then
        currentState = State.CHASE
        return
    end
    
    -- หันหน้าหา target
    local lookAt = CFrame.lookAt(rootPart.Position, targetRoot.Position)
    rootPart.CFrame = lookAt
    
    -- โจมตีถ้า cooldown ผ่านแล้ว
    local now = os.clock()
    if now - lastAttackTime >= CONFIG.attackCooldown then
        lastAttackTime = now
        
        -- Deal damage
        targetHumanoid:TakeDamage(CONFIG.damage)
        
        -- เอฟเฟกต์เสียง (ถ้ามี)
        -- local sound = NPC:FindFirstChild("AttackSound")
        -- if sound then sound:Play() end
        
        print(NPC.Name .. " โจมตี " .. target.Name .. " (" .. CONFIG.damage .. " damage)")
        
        -- Animation โจมตี (ถ้ามี)
        -- playAttackAnimation()
    end
    
    task.wait(0.1)
end

-- ===== Death and Respawn =====

local isDead = false

local function onDeath()
    if isDead then return end
    isDead = true
    currentState = State.DEAD
    
    print(NPC.Name .. " ตายแล้ว!")
    
    -- Drop loot
    local coinAmount = math.random(CONFIG.coinRange.min, CONFIG.coinRange.max)
    dropLoot(rootPart.Position, coinAmount)
    
    -- Notify เพื่อให้ระบบ EXP ทำงาน
    -- (ในระบบจริงจะใช้ RemoteEvent หรือ BindableEvent)
    if target then
        local ReplicatedStorage = game:GetService("ReplicatedStorage")
        local killEvent = ReplicatedStorage:FindFirstChild("Events") and
                         ReplicatedStorage.Events:FindFirstChild("EnemyKilled")
        
        if killEvent then
            killEvent:FireAllClients({
                npcName = NPC.Name,
                killer = target,
                expReward = CONFIG.expReward,
                coinReward = coinAmount
            })
        end
    end
    
    -- ซ่อน NPC
    NPC.Parent = workspace
    for _, part in ipairs(NPC:GetDescendants()) do
        if part:IsA("BasePart") then
            part.Transparency = 0.8
            part.CanCollide = false
        end
    end
    
    -- Respawn
    task.delay(CONFIG.respawnTime, function()
        respawnNPC()
    end)
end

function dropLoot(position, coinAmount)
    -- สร้าง coin pickup
    local coinPart = Instance.new("Part")
    coinPart.Name = "CoinDrop"
    coinPart.Size = Vector3.new(1, 0.3, 1)
    coinPart.Position = position + Vector3.new(
        math.random(-2, 2), 2, math.random(-2, 2)
    )
    coinPart.BrickColor = BrickColor.new("Bright yellow")
    coinPart.Material = Enum.Material.SmoothPlastic
    coinPart.Parent = workspace
    
    -- Auto-collect ด้วย proximity
    local prompt = Instance.new("ProximityPrompt")
    prompt.ActionText = "เก็บ " .. coinAmount .. " 💰"
    prompt.MaxActivationDistance = 5
    prompt.Parent = coinPart
    
    prompt.Triggered:Connect(function(player)
        -- ให้เหรียญ
        local leaderstats = player:FindFirstChild("leaderstats")
        if leaderstats then
            local coins = leaderstats:FindFirstChild("💰 เหรียญ")
            if coins then
                coins.Value = coins.Value + coinAmount
            end
        end
        coinPart:Destroy()
    end)
    
    -- ลบหลัง 30 วินาที
    game:GetService("Debris"):AddItem(coinPart, 30)
end

function respawnNPC()
    isDead = false
    
    -- รีเซ็ต
    humanoid.Health = CONFIG.maxHealth
    humanoid.WalkSpeed = CONFIG.walkSpeed
    
    -- คืนตำแหน่ง
    rootPart.CFrame = CFrame.new(spawnPosition)
    
    -- แสดงอีกครั้ง
    for _, part in ipairs(NPC:GetDescendants()) do
        if part:IsA("BasePart") then
            part.Transparency = 0
            part.CanCollide = true
        end
    end
    
    target = nil
    currentState = State.IDLE
    
    print(NPC.Name .. " respawn แล้ว!")
end

-- ===== Health Bar =====
local function createHealthBar()
    local billboardGui = Instance.new("BillboardGui")
    billboardGui.Size = UDim2.new(0, 100, 0, 20)
    billboardGui.StudsOffset = Vector3.new(0, 3, 0)
    billboardGui.AlwaysOnTop = false
    billboardGui.Parent = rootPart
    
    local background = Instance.new("Frame")
    background.Size = UDim2.new(1, 0, 1, 0)
    background.BackgroundColor3 = Color3.fromRGB(50, 0, 0)
    background.Parent = billboardGui
    
    local healthBar = Instance.new("Frame")
    healthBar.Name = "HealthBar"
    healthBar.Size = UDim2.new(1, 0, 1, 0)
    healthBar.BackgroundColor3 = Color3.fromRGB(0, 200, 0)
    healthBar.Parent = background
    
    local nameLabel = Instance.new("TextLabel")
    nameLabel.Size = UDim2.new(1, 0, 0, 12)
    nameLabel.Position = UDim2.new(0, 0, -0.8, 0)
    nameLabel.Text = NPC.Name
    nameLabel.TextSize = 10
    nameLabel.TextColor3 = Color3.new(1,1,1)
    nameLabel.BackgroundTransparency = 1
    nameLabel.Parent = background
    
    -- อัพเดท health bar
    humanoid:GetPropertyChangedSignal("Health"):Connect(function()
        local pct = humanoid.Health / humanoid.MaxHealth
        healthBar.Size = UDim2.new(math.max(0, pct), 0, 1, 0)
        
        -- เปลี่ยนสีตาม HP
        if pct > 0.5 then
            healthBar.BackgroundColor3 = Color3.fromRGB(0, 200, 0)
        elseif pct > 0.25 then
            healthBar.BackgroundColor3 = Color3.fromRGB(255, 165, 0)
        else
            healthBar.BackgroundColor3 = Color3.fromRGB(200, 0, 0)
        end
    end)
end

createHealthBar()

-- ===== Main Loop =====
humanoid.Died:Connect(onDeath)

-- อัพเดท state
task.spawn(function()
    while NPC.Parent do
        if isDead then
            task.wait(0.5)
            continue
        end
        
        -- ตรวจจับผู้เล่น
        if CONFIG.aggroType == "hostile" then
            local nearestPlayer, dist = findNearestPlayer()
            
            if nearestPlayer and currentState ~= State.ATTACK and 
               currentState ~= State.CHASE then
                target = nearestPlayer
                currentState = State.CHASE
            end
        end
        
        -- ทำงานตาม state
        if currentState == State.IDLE then
            doIdle()
        elseif currentState == State.PATROL then
            doPatrol()
        elseif currentState == State.CHASE then
            doChase()
        elseif currentState == State.ATTACK then
            doAttack()
        end
        
        task.wait(0.1)
    end
end)

print(NPC.Name .. " AI เริ่มทำงาน!")
```

## NPC ผู้ขายของ (Friendly NPC)

```lua
-- Script ใน ShopNPC Model

local NPC = script.Parent
local rootPart = NPC:WaitForChild("HumanoidRootPart")

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- สร้าง ProximityPrompt
local prompt = Instance.new("ProximityPrompt")
prompt.ActionText = "🏪 เปิดร้าน"
prompt.ObjectText = "พ่อค้า"
prompt.MaxActivationDistance = 8
prompt.Parent = rootPart

-- แสดงชื่อ
local billboardGui = Instance.new("BillboardGui")
billboardGui.Size = UDim2.new(0, 150, 0, 50)
billboardGui.StudsOffset = Vector3.new(0, 4, 0)
billboardGui.Parent = rootPart

local nameLabel = Instance.new("TextLabel")
nameLabel.Size = UDim2.new(1, 0, 0.5, 0)
nameLabel.Text = "🧙 พ่อค้าผู้รอบรู้"
nameLabel.TextSize = 13
nameLabel.Font = Enum.Font.GothamBold
nameLabel.TextColor3 = Color3.fromRGB(255, 215, 0)
nameLabel.BackgroundTransparency = 1
nameLabel.Parent = billboardGui

local roleLabel = Instance.new("TextLabel")
roleLabel.Size = UDim2.new(1, 0, 0.5, 0)
roleLabel.Position = UDim2.new(0, 0, 0.5, 0)
roleLabel.Text = "คลิกเพื่อซื้อขาย"
roleLabel.TextSize = 10
roleLabel.TextColor3 = Color3.new(0.8, 0.8, 0.8)
roleLabel.BackgroundTransparency = 1
roleLabel.Parent = billboardGui

-- เมื่อผู้เล่นเข้ามาคุย
prompt.Triggered:Connect(function(player)
    print(player.Name .. " เปิดร้านค้ากับ " .. NPC.Name)
    
    -- แจ้ง client เปิด shop UI
    local openShopRE = ReplicatedStorage:FindFirstChild("Events") and
                       ReplicatedStorage.Events:FindFirstChild("OpenShop")
    
    if openShopRE then
        openShopRE:FireClient(player, {
            shopId = "main_shop",
            npcName = NPC.Name
        })
    end
end)

-- หันหน้าหาผู้เล่นที่เข้าใกล้
task.spawn(function()
    while NPC.Parent do
        task.wait(0.5)
        
        local nearest = nil
        local nearestDist = 15
        
        for _, player in ipairs(Players:GetPlayers()) do
            local character = player.Character
            if not character then continue end
            
            local playerRoot = character:FindFirstChild("HumanoidRootPart")
            if not playerRoot then continue end
            
            local dist = (rootPart.Position - playerRoot.Position).Magnitude
            if dist < nearestDist then
                nearest = player
                nearestDist = dist
            end
        end
        
        if nearest and nearest.Character then
            local playerRoot = nearest.Character:FindFirstChild("HumanoidRootPart")
            if playerRoot then
                -- หันหน้า (แกน Y เท่านั้น)
                local lookDir = playerRoot.Position - rootPart.Position
                lookDir = Vector3.new(lookDir.X, 0, lookDir.Z)
                
                if lookDir.Magnitude > 0.1 then
                    rootPart.CFrame = CFrame.new(rootPart.Position, 
                        rootPart.Position + lookDir)
                end
            end
        end
    end
end)
```

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: สร้าง NPC แบบ Neutral
NPC ที่ปกติเดินลาดตระเวน แต่โจมตีเมื่อถูกโจมตีก่อน

### แบบฝึกหัดที่ 2: NPC Dialog
สร้าง NPC ที่คุยได้ มี dialog tree ให้เลือก

### แบบฝึกหัดที่ 3: Boss NPC
สร้าง Boss ที่มี HP สูง มี phase และ special attacks

## สรุป

NPC AI พื้นฐานประกอบด้วย:
- **State Machine** - Idle, Patrol, Chase, Attack, Dead
- **Detection** - ตรวจจับผู้เล่น
- **Movement** - เดิน ไล่ หนี
- **Combat** - โจมตี cooldown
- **Respawn** - ฟื้นคืนชีพ
- **Loot** - drop ของ

ในบทถัดไปเราจะเรียนรู้ Pathfinding ที่ซับซ้อนขึ้น!
