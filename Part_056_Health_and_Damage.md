# Part 56: ระบบ Health, Damage และ Respawn

## บทนำ

ระบบ Health เป็นพื้นฐานของทุกเกมที่มีการต่อสู้ ในบทนี้เราจะสร้างระบบ Health ที่สมบูรณ์ ครอบคลุมทั้ง HP bar, การตาย, การ respawn และระบบฟื้นฟูอัตโนมัติ

## ระบบ Health พื้นฐาน

```lua
-- ServerScriptService/HealthSystem.lua

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- ===== Configuration =====
local HEALTH_CONFIG = {
    -- HP Regeneration
    regenEnabled = true,
    regenDelay = 5,       -- รอกี่วินาทีหลัง damage จึงเริ่ม regen
    regenRate = 2,        -- HP ต่อวินาที
    regenPercent = false, -- regen เป็น % หรือ flat
    
    -- Respawn
    respawnTime = 5,
    respawnInvincibilityTime = 3,
    respawnWithFullHP = true,
    
    -- Death
    deathScreenDuration = 3,
    keepToolsOnDeath = false,
    
    -- Shield
    shieldEnabled = true,
    shieldCapacity = 50,
    shieldRegenDelay = 8,
    shieldRegenRate = 5
}

-- ===== Player Health Data =====
local healthData = {}

local function initHealthData(player)
    healthData[player.UserId] = {
        lastDamageTime = 0,
        isRegenerating = false,
        shield = 0,
        maxShield = HEALTH_CONFIG.shieldCapacity,
        lastShieldDamageTime = 0
    }
end

-- ===== Setup Character =====
local function setupCharacter(player)
    local character = player.Character
    if not character then return end
    
    local humanoid = character:WaitForChild("Humanoid")
    
    -- อย่าให้ HP regen อัตโนมัติ (เราจัดการเอง)
    humanoid.HealthDisplayDistance = 0  -- ซ่อน default health bar
    humanoid.NameDisplayDistance = 0    -- ซ่อน default name display
    
    -- รับ damage event
    humanoid.HealthChanged:Connect(function(newHealth)
        local data = healthData[player.UserId]
        if not data then return end
        
        local change = newHealth - humanoid.Health
        if change < 0 then
            -- ได้รับ damage
            data.lastDamageTime = os.clock()
            data.lastShieldDamageTime = os.clock()
            data.isRegenerating = false
            
            -- แจ้ง client
            local Events = ReplicatedStorage:FindFirstChild("Events")
            if Events then
                local updateHPEvent = Events:FindFirstChild("UI_UpdateHP")
                if updateHPEvent then
                    updateHPEvent:FireClient(player, {
                        health = newHealth,
                        maxHealth = humanoid.MaxHealth,
                        change = change
                    })
                end
            end
        end
    end)
    
    -- ตายแล้ว
    humanoid.Died:Connect(function()
        onPlayerDied(player)
    end)
    
    -- Invincibility เมื่อ spawn
    if HEALTH_CONFIG.respawnWithFullHP then
        humanoid.Health = humanoid.MaxHealth
    end
    
    -- Spawn invincibility
    task.spawn(function()
        -- ทำให้ไม่ได้รับ damage ชั่วคราว
        local invincibleTag = Instance.new("BoolValue")
        invincibleTag.Name = "Invincible"
        invincibleTag.Value = true
        invincibleTag.Parent = character
        
        task.delay(HEALTH_CONFIG.respawnInvincibilityTime, function()
            if invincibleTag and invincibleTag.Parent then
                invincibleTag:Destroy()
            end
        end)
    end)
    
    -- เริ่ม health regen
    startHealthRegen(player)
end

-- ===== Health Regeneration =====
local function startHealthRegen(player)
    task.spawn(function()
        while player.Parent do
            task.wait(1)
            
            local character = player.Character
            if not character then continue end
            
            local humanoid = character:FindFirstChild("Humanoid")
            if not humanoid or humanoid.Health <= 0 then continue end
            
            local data = healthData[player.UserId]
            if not data then continue end
            
            local now = os.clock()
            
            -- HP Regen
            if HEALTH_CONFIG.regenEnabled then
                local timeSinceDamage = now - data.lastDamageTime
                
                if timeSinceDamage >= HEALTH_CONFIG.regenDelay and 
                   humanoid.Health < humanoid.MaxHealth then
                    
                    local regenAmount
                    if HEALTH_CONFIG.regenPercent then
                        regenAmount = humanoid.MaxHealth * (HEALTH_CONFIG.regenRate / 100)
                    else
                        regenAmount = HEALTH_CONFIG.regenRate
                    end
                    
                    humanoid.Health = math.min(
                        humanoid.MaxHealth,
                        humanoid.Health + regenAmount
                    )
                end
            end
            
            -- Shield Regen
            if HEALTH_CONFIG.shieldEnabled then
                local timeSinceShieldDamage = now - (data.lastShieldDamageTime or 0)
                
                if timeSinceShieldDamage >= HEALTH_CONFIG.shieldRegenDelay and
                   data.shield < data.maxShield then
                    
                    data.shield = math.min(
                        data.maxShield,
                        data.shield + HEALTH_CONFIG.shieldRegenRate
                    )
                end
            end
            
            -- Update UI
            local Events = ReplicatedStorage:FindFirstChild("Events")
            if Events then
                local updateEvent = Events:FindFirstChild("UI_UpdateHP")
                if updateEvent then
                    updateEvent:FireClient(player, {
                        health = humanoid.Health,
                        maxHealth = humanoid.MaxHealth,
                        shield = data.shield,
                        maxShield = data.maxShield
                    })
                end
            end
        end
    end)
end

-- ===== Death Handler =====
local function onPlayerDied(player)
    print("💀 " .. player.Name .. " ตาย!")
    
    -- แจ้ง client
    local Events = ReplicatedStorage:FindFirstChild("Events")
    if Events then
        local diedEvent = Events:FindFirstChild("PlayerDied")
        if diedEvent then
            diedEvent:FireClient(player)
        end
        
        -- แจ้งทุกคน
        local announceEvent = Events:FindFirstChild("AnnounceKill")
        if announceEvent then
            -- หา killer
            announceEvent:FireAllClients({
                player = player.Name,
                killer = "Unknown"  -- ในระบบจริงจะ track
            })
        end
    end
    
    -- Respawn
    task.delay(HEALTH_CONFIG.respawnTime, function()
        if player.Parent then
            player:LoadCharacter()
        end
    end)
end

-- ===== Damage with Shield =====
function applyDamageWithShield(player, damage)
    local data = healthData[player.UserId]
    if not data then return damage end
    
    if HEALTH_CONFIG.shieldEnabled and data.shield > 0 then
        -- ลด damage จาก shield ก่อน
        local shieldAbsorb = math.min(data.shield, damage)
        data.shield = data.shield - shieldAbsorb
        damage = damage - shieldAbsorb
        
        data.lastShieldDamageTime = os.clock()
        
        if shieldAbsorb > 0 then
            print(string.format("🛡️ Shield absorb %d damage (%d remaining)", 
                shieldAbsorb, data.shield))
        end
    end
    
    return damage  -- damage ที่เหลือหลังจาก shield
end

-- ===== Events =====
Players.PlayerAdded:Connect(function(player)
    initHealthData(player)
    
    player.CharacterAdded:Connect(function()
        task.wait(0.1)  -- รอ character โหลด
        setupCharacter(player)
    end)
    
    -- ถ้า character มีอยู่แล้ว
    if player.Character then
        setupCharacter(player)
    end
end)

Players.PlayerRemoving:Connect(function(player)
    healthData[player.UserId] = nil
end)
```

## Health Bar UI

```lua
-- LocalScript ใน StarterGui

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer

-- ===== สร้าง Health Bar UI =====
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "HealthGui"
screenGui.ResetOnSpawn = false
screenGui.Parent = player.PlayerGui

-- Main Health Bar Frame
local healthFrame = Instance.new("Frame")
healthFrame.Size = UDim2.new(0, 300, 0, 20)
healthFrame.Position = UDim2.new(0.5, -150, 1, -80)
healthFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
healthFrame.Parent = screenGui
Instance.new("UICorner", healthFrame).CornerRadius = UDim.new(1, 0)

-- HP Background
local hpBg = Instance.new("Frame")
hpBg.Size = UDim2.new(1, 0, 0.6, 0)
hpBg.Position = UDim2.new(0, 0, 0, 0)
hpBg.BackgroundColor3 = Color3.fromRGB(50, 0, 0)
hpBg.Parent = healthFrame
Instance.new("UICorner", hpBg).CornerRadius = UDim.new(1, 0)

-- HP Bar
local hpBar = Instance.new("Frame")
hpBar.Name = "HPBar"
hpBar.Size = UDim2.new(1, 0, 1, 0)
hpBar.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
hpBar.Parent = hpBg
Instance.new("UICorner", hpBar).CornerRadius = UDim.new(1, 0)

-- HP Text
local hpText = Instance.new("TextLabel")
hpText.Name = "HPText"
hpText.Size = UDim2.new(1, 0, 0.6, 0)
hpText.Text = "100/100"
hpText.TextSize = 12
hpText.Font = Enum.Font.GothamBold
hpText.TextColor3 = Color3.new(1,1,1)
hpText.BackgroundTransparency = 1
hpText.ZIndex = 2
hpText.Parent = healthFrame

-- Shield Bar
local shieldBg = Instance.new("Frame")
shieldBg.Size = UDim2.new(1, 0, 0.35, 0)
shieldBg.Position = UDim2.new(0, 0, 0.65, 0)
shieldBg.BackgroundColor3 = Color3.fromRGB(0, 30, 60)
shieldBg.Parent = healthFrame
Instance.new("UICorner", shieldBg).CornerRadius = UDim.new(1, 0)

local shieldBar = Instance.new("Frame")
shieldBar.Name = "ShieldBar"
shieldBar.Size = UDim2.new(0, 0, 1, 0)  -- เริ่มที่ 0
shieldBar.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
shieldBar.Parent = shieldBg
Instance.new("UICorner", shieldBar).CornerRadius = UDim.new(1, 0)

-- ===== Death Screen =====
local deathScreen = Instance.new("Frame")
deathScreen.Size = UDim2.new(1, 0, 1, 0)
deathScreen.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
deathScreen.BackgroundTransparency = 1
deathScreen.Visible = false
deathScreen.ZIndex = 100
deathScreen.Parent = screenGui

local deathLabel = Instance.new("TextLabel")
deathLabel.Size = UDim2.new(0, 400, 0, 100)
deathLabel.Position = UDim2.new(0.5, -200, 0.4, -50)
deathLabel.Text = "💀 คุณตายแล้ว"
deathLabel.TextSize = 48
deathLabel.Font = Enum.Font.GothamBold
deathLabel.TextColor3 = Color3.fromRGB(200, 0, 0)
deathLabel.BackgroundTransparency = 1
deathLabel.ZIndex = 101
deathLabel.Parent = deathScreen

local respawnLabel = Instance.new("TextLabel")
respawnLabel.Size = UDim2.new(0, 300, 0, 40)
respawnLabel.Position = UDim2.new(0.5, -150, 0.55, 0)
respawnLabel.Text = "กำลัง respawn ใน 5 วินาที..."
respawnLabel.TextSize = 18
respawnLabel.TextColor3 = Color3.new(0.8, 0.8, 0.8)
respawnLabel.BackgroundTransparency = 1
respawnLabel.ZIndex = 101
respawnLabel.Parent = deathScreen

-- ===== Update HP UI =====
local function updateHealthUI(data)
    local hpPercent = data.health / data.maxHealth
    
    -- Animate HP bar
    TweenService:Create(hpBar, TweenInfo.new(0.2), {
        Size = UDim2.new(math.max(0, hpPercent), 0, 1, 0)
    }):Play()
    
    -- Tween HP bar color
    local targetColor
    if hpPercent > 0.6 then
        targetColor = Color3.fromRGB(0, 200, 0)
    elseif hpPercent > 0.3 then
        targetColor = Color3.fromRGB(255, 165, 0)
    else
        targetColor = Color3.fromRGB(200, 0, 0)
    end
    
    TweenService:Create(hpBar, TweenInfo.new(0.3), {
        BackgroundColor3 = targetColor
    }):Play()
    
    -- Update text
    hpText.Text = string.format("%d/%d", math.floor(data.health), data.maxHealth)
    
    -- Update shield
    if data.shield ~= nil then
        local shieldPercent = data.shield / (data.maxShield or 50)
        TweenService:Create(shieldBar, TweenInfo.new(0.3), {
            Size = UDim2.new(math.max(0, shieldPercent), 0, 1, 0)
        }):Play()
    end
end

-- ===== Events =====
local Events = ReplicatedStorage:WaitForChild("Events")

local updateHPEvent = Events:WaitForChild("UI_UpdateHP")
updateHPEvent.OnClientEvent:Connect(updateHealthUI)

local diedEvent = Events:WaitForChild("PlayerDied")
diedEvent.OnClientEvent:Connect(function()
    -- แสดง death screen
    deathScreen.Visible = true
    TweenService:Create(deathScreen, TweenInfo.new(0.5), {
        BackgroundTransparency = 0.4
    }):Play()
    
    -- Countdown
    for i = 5, 1, -1 do
        respawnLabel.Text = string.format("กำลัง respawn ใน %d วินาที...", i)
        task.wait(1)
    end
    
    TweenService:Create(deathScreen, TweenInfo.new(0.5), {
        BackgroundTransparency = 1
    }):Play()
    
    task.wait(0.5)
    deathScreen.Visible = false
end)

-- อัพเดทจาก Humanoid โดยตรง (ตรงกว่ารอ event จาก server)
player.CharacterAdded:Connect(function(character)
    local humanoid = character:WaitForChild("Humanoid")
    
    humanoid.HealthChanged:Connect(function()
        updateHealthUI({
            health = humanoid.Health,
            maxHealth = humanoid.MaxHealth
        })
    end)
    
    -- เริ่มต้น
    updateHealthUI({
        health = humanoid.Health,
        maxHealth = humanoid.MaxHealth,
        shield = 0,
        maxShield = 50
    })
end)
```

## Damage Numbers บนหัวศัตรู

```lua
-- ส่วนต่างๆ ของ combat effects
local function createFloatingDamage(position, damage, damageType)
    local part = Instance.new("Part")
    part.Size = Vector3.new(0.1, 0.1, 0.1)
    part.Position = position
    part.Anchored = true
    part.CanCollide = false
    part.Transparency = 1
    part.Parent = workspace
    
    local billboardGui = Instance.new("BillboardGui")
    billboardGui.Size = UDim2.new(0, 100, 0, 50)
    billboardGui.AlwaysOnTop = true
    billboardGui.Parent = part
    
    local damageLabel = Instance.new("TextLabel")
    damageLabel.Size = UDim2.new(1, 0, 1, 0)
    damageLabel.BackgroundTransparency = 1
    damageLabel.Text = tostring(damage)
    
    -- เลือกสีตาม damage type
    local colors = {
        physical = Color3.new(1, 1, 1),
        fire = Color3.fromRGB(255, 100, 0),
        ice = Color3.fromRGB(100, 200, 255),
        heal = Color3.fromRGB(100, 255, 100),
        crit = Color3.fromRGB(255, 215, 0)
    }
    
    damageLabel.TextColor3 = colors[damageType] or colors.physical
    damageLabel.TextSize = damageType == "crit" and 24 or 18
    damageLabel.Font = Enum.Font.GothamBold
    damageLabel.TextStrokeColor3 = Color3.new(0, 0, 0)
    damageLabel.TextStrokeTransparency = 0.5
    damageLabel.Parent = billboardGui
    
    -- Animate
    local TweenService = game:GetService("TweenService")
    
    TweenService:Create(part, TweenInfo.new(1.5), {
        Position = position + Vector3.new(math.random(-2, 2), 5, 0)
    }):Play()
    
    TweenService:Create(damageLabel, TweenInfo.new(1.5, Enum.EasingStyle.Quad, Enum.EasingDirection.In, 0, false, 0.5), {
        TextTransparency = 1,
        TextStrokeTransparency = 1
    }):Play()
    
    game:GetService("Debris"):AddItem(part, 2)
end
```

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Last Stand
ระบบ "Last Stand" - เมื่อ HP ถึง 0 ให้มีโอกาส 20% ที่จะรอดตายด้วย 1 HP

### แบบฝึกหัดที่ 2: Healing Zone
สร้างพื้นที่ที่เมื่อยืนอยู่จะ regen HP เร็วขึ้น

### แบบฝึกหัดที่ 3: Lives System
ระบบ "ชีวิต" - ผู้เล่นมี 3 ชีวิต เมื่อหมดต้องออก

## สรุป

ระบบ Health ที่สมบูรณ์ต้องมี:
- **HP Bar** ที่ animate สวยงาม
- **Shield System** - ชั้นป้องกันเพิ่มเติม
- **Health Regen** - ฟื้นฟูอัตโนมัติ
- **Death Screen** - แสดงเมื่อตาย
- **Respawn** - กลับมาอีกครั้ง
- **Invincibility Frame** - ช่วงเวลาไม่โดน damage หลัง spawn
