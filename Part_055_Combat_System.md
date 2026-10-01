# Part 55: ระบบต่อสู้ (Combat System) สมบูรณ์

## บทนำ

ระบบต่อสู้เป็นหัวใจของเกมแนว Action และ RPG ในบทนี้เราจะสร้างระบบต่อสู้ที่สมบูรณ์ ครอบคลุมทั้งการโจมตี, การป้องกัน, เอฟเฟกต์ต่างๆ และการประมวลผลบน Server

## โครงสร้างระบบต่อสู้

```
CombatSystem/
├── Server/
│   ├── CombatManager.lua   - ประมวลผลการต่อสู้
│   ├── DamageHandler.lua   - จัดการ damage
│   └── HitDetection.lua    - ตรวจจับการโจมตี
├── Client/
│   ├── CombatClient.lua    - input และ animation
│   └── CombatEffects.lua   - visual effects
└── Shared/
    └── CombatConfig.lua    - การตั้งค่า
```

## Combat Configuration

```lua
-- ReplicatedStorage/Shared/CombatConfig.lua (ModuleScript)

local CombatConfig = {}

-- ===== Damage Types =====
CombatConfig.DamageTypes = {
    PHYSICAL = "physical",
    MAGIC = "magic",
    FIRE = "fire",
    ICE = "ice",
    LIGHTNING = "lightning",
    POISON = "poison",
    TRUE = "true"  -- ทะลุ defense ทุกชนิด
}

-- ===== Status Effects =====
CombatConfig.StatusEffects = {
    BURN = {
        id = "burn",
        name = "🔥 ไฟไหม้",
        tickDamage = 5,
        tickInterval = 1,
        color = Color3.fromRGB(255, 100, 0)
    },
    POISON = {
        id = "poison",
        name = "☠️ พิษ",
        tickDamage = 3,
        tickInterval = 0.5,
        color = Color3.fromRGB(100, 200, 0)
    },
    FREEZE = {
        id = "freeze",
        name = "❄️ แช่แข็ง",
        speedMultiplier = 0,
        color = Color3.fromRGB(100, 200, 255)
    },
    SLOW = {
        id = "slow",
        name = "🐢 ช้า",
        speedMultiplier = 0.5,
        color = Color3.fromRGB(150, 100, 200)
    },
    STUN = {
        id = "stun",
        name = "⚡ งง",
        speedMultiplier = 0,
        canAttack = false,
        color = Color3.fromRGB(255, 255, 0)
    }
}

-- ===== Hit Types =====
CombatConfig.HitTypes = {
    NORMAL = { name = "ปกติ", multiplier = 1.0 },
    CRITICAL = { name = "Critical!", multiplier = 2.0 },
    GLANCING = { name = "Glancing", multiplier = 0.5 },
    BLOCK = { name = "Block", multiplier = 0.1 },
    MISS = { name = "Miss", multiplier = 0 }
}

-- ===== Defense Formula =====
-- finalDamage = baseDamage * (100 / (100 + defense))
-- 0 defense = 100% damage
-- 100 defense = 50% damage
-- 200 defense = 33% damage

function CombatConfig.calculateMitigatedDamage(damage, defense)
    return damage * (100 / (100 + math.max(0, defense)))
end

-- ===== Critical Hit =====
function CombatConfig.rollCrit(critChance, critMultiplier)
    if math.random() < critChance then
        return true, critMultiplier or 2.0
    end
    return false, 1.0
end

return CombatConfig
```

## Combat Manager (Server)

```lua
-- ServerScriptService/Combat/CombatManager.lua (Script/ModuleScript)

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")

local CombatConfig = require(ReplicatedStorage.Shared.CombatConfig)

local CombatManager = {}

-- ===== State =====
local combatantData = {}  -- ข้อมูลสำหรับแต่ละตัวที่เข้าร่วมต่อสู้

-- ===== ลงทะเบียน combatant =====
function CombatManager.registerCombatant(entity, stats)
    local id = tostring(entity) -- ใช้ entity เป็น key
    
    combatantData[id] = {
        entity = entity,
        stats = {
            health = stats.health or 100,
            maxHealth = stats.maxHealth or 100,
            mana = stats.mana or 50,
            maxMana = stats.maxMana or 50,
            attack = stats.attack or 10,
            defense = stats.defense or 0,
            magicPower = stats.magicPower or 0,
            magicDefense = stats.magicDefense or 0,
            critChance = stats.critChance or 0.05,
            critMultiplier = stats.critMultiplier or 2.0,
            speed = stats.speed or 16,
            dodgeChance = stats.dodgeChance or 0.02
        },
        statusEffects = {},
        lastHitTime = {},  -- สำหรับ cooldown ต่อเป้าหมาย
        invincible = false,
        invincibleUntil = 0
    }
    
    return combatantData[id]
end

-- ===== Main Damage Function =====
function CombatManager.dealDamage(attacker, target, damageInfo)
    --[[
        damageInfo = {
            baseDamage = number,
            damageType = string,    -- physical, magic, fire, etc.
            source = string,        -- weapon id, spell id
            knockback = Vector3?,
            statusEffect = table?   -- { type, duration }
        }
    ]]
    
    -- ตรวจสอบ target
    local targetHumanoid = target:FindFirstChild("Humanoid") or
                           (target.Character and target.Character:FindFirstChild("Humanoid"))
    
    if not targetHumanoid or targetHumanoid.Health <= 0 then
        return { success = false, reason = "invalid_target" }
    end
    
    -- ตรวจสอบ invincibility
    if os.clock() < (combatantData[tostring(target)] and 
                     combatantData[tostring(target)].invincibleUntil or 0) then
        return { 
            success = true, 
            damage = 0, 
            hitType = "invincible",
            blocked = true 
        }
    end
    
    -- ดึง stats
    local attackerData = combatantData[tostring(attacker)]
    local targetData = combatantData[tostring(target)]
    
    local baseDamage = damageInfo.baseDamage
    if not baseDamage and attackerData then
        baseDamage = attackerData.stats.attack
    end
    baseDamage = baseDamage or 10
    
    local targetDefense = targetData and targetData.stats.defense or 0
    local targetMagDef = targetData and targetData.stats.magicDefense or 0
    
    -- Dodge check
    local dodgeChance = targetData and targetData.stats.dodgeChance or 0
    if math.random() < dodgeChance then
        -- Show miss
        showDamageNumber(target, 0, "miss")
        return { success = true, damage = 0, hitType = "miss" }
    end
    
    -- Critical hit check
    local critChance = attackerData and attackerData.stats.critChance or 0.05
    local critMult = attackerData and attackerData.stats.critMultiplier or 2.0
    local isCrit, critMultiplier = CombatConfig.rollCrit(critChance, critMult)
    
    -- คำนวณ final damage
    local finalDamage = baseDamage
    
    -- Apply defense
    local damageType = damageInfo.damageType or CombatConfig.DamageTypes.PHYSICAL
    
    if damageType == CombatConfig.DamageTypes.PHYSICAL then
        finalDamage = CombatConfig.calculateMitigatedDamage(finalDamage, targetDefense)
    elseif damageType == CombatConfig.DamageTypes.MAGIC or
           damageType == CombatConfig.DamageTypes.FIRE or
           damageType == CombatConfig.DamageTypes.ICE then
        finalDamage = CombatConfig.calculateMitigatedDamage(finalDamage, targetMagDef)
    end
    -- TRUE damage ไม่ลด defense
    
    -- Apply crit
    if isCrit then
        finalDamage = finalDamage * critMultiplier
    end
    
    -- Final floor
    finalDamage = math.max(1, math.floor(finalDamage))
    
    -- Apply damage
    targetHumanoid:TakeDamage(finalDamage)
    
    -- Knockback
    if damageInfo.knockback then
        local targetRoot = target:FindFirstChild("HumanoidRootPart") or
                          (target.Character and target.Character:FindFirstChild("HumanoidRootPart"))
        
        if targetRoot then
            local velocity = Instance.new("LinearVelocity")
            velocity.VectorVelocity = damageInfo.knockback
            velocity.MaxForce = math.huge
            velocity.Parent = targetRoot
            
            game:GetService("Debris"):AddItem(velocity, 0.15)
        end
    end
    
    -- Show damage number
    local hitType = isCrit and "crit" or "normal"
    showDamageNumber(target, finalDamage, hitType, damageType)
    
    -- Apply status effect
    if damageInfo.statusEffect and targetData then
        CombatManager.applyStatusEffect(target, damageInfo.statusEffect)
    end
    
    -- ยิง events
    local Events = ReplicatedStorage:FindFirstChild("Events")
    if Events then
        local dmgEvent = Events:FindFirstChild("Combat_DamageDealt")
        if dmgEvent then
            dmgEvent:FireAllClients({
                target = target,
                damage = finalDamage,
                isCrit = isCrit,
                damageType = damageType,
                hitType = hitType
            })
        end
    end
    
    return {
        success = true,
        damage = finalDamage,
        isCrit = isCrit,
        hitType = hitType,
        damageType = damageType
    }
end

-- ===== Damage Numbers =====
function showDamageNumber(target, damage, hitType, damageType)
    local targetRoot = (target.Character and target.Character:FindFirstChild("HumanoidRootPart")) or
                       target:FindFirstChild("HumanoidRootPart")
    
    if not targetRoot then return end
    
    local Events = ReplicatedStorage:FindFirstChild("Events")
    if Events then
        local showDmgEvent = Events:FindFirstChild("UI_ShowDamageNumber")
        if showDmgEvent then
            showDmgEvent:FireAllClients({
                position = targetRoot.Position,
                damage = damage,
                hitType = hitType,
                damageType = damageType
            })
        end
    end
end

-- ===== Status Effects =====
function CombatManager.applyStatusEffect(target, effectData)
    local targetData = combatantData[tostring(target)]
    if not targetData then return end
    
    local effectConfig = CombatConfig.StatusEffects[effectData.type:upper()]
    if not effectConfig then return end
    
    -- Remove existing effect of same type
    for i, existing in ipairs(targetData.statusEffects) do
        if existing.type == effectData.type then
            table.remove(targetData.statusEffects, i)
            break
        end
    end
    
    local effect = {
        type = effectData.type,
        config = effectConfig,
        duration = effectData.duration or 5,
        startTime = os.time(),
        lastTick = os.clock()
    }
    
    table.insert(targetData.statusEffects, effect)
    
    -- Apply speed effect
    local targetHumanoid = (target.Character and target.Character:FindFirstChild("Humanoid")) or
                           target:FindFirstChild("Humanoid")
    
    if targetHumanoid then
        if effectConfig.speedMultiplier ~= nil then
            local originalSpeed = targetData.stats.speed
            targetHumanoid.WalkSpeed = originalSpeed * effectConfig.speedMultiplier
            
            -- รีเซ็ต speed เมื่อหมด
            task.delay(effectData.duration, function()
                if targetHumanoid and targetHumanoid.Parent then
                    targetHumanoid.WalkSpeed = originalSpeed
                end
            end)
        end
    end
    
    print(string.format("Applied %s to %s for %ds", effectData.type, tostring(target), effectData.duration))
end

-- ===== Update Status Effects (tick damage) =====
task.spawn(function()
    while true do
        task.wait(0.5)
        
        for id, data in pairs(combatantData) do
            local target = data.entity
            
            for i = #data.statusEffects, 1, -1 do
                local effect = data.statusEffects[i]
                local elapsed = os.time() - effect.startTime
                
                -- หมดเวลา
                if elapsed >= effect.duration then
                    table.remove(data.statusEffects, i)
                    continue
                end
                
                -- Tick damage
                if effect.config.tickDamage then
                    local now = os.clock()
                    if now - effect.lastTick >= effect.config.tickInterval then
                        effect.lastTick = now
                        
                        local targetHumanoid = (target.Character and 
                                               target.Character:FindFirstChild("Humanoid")) or
                                               target:FindFirstChild("Humanoid")
                        
                        if targetHumanoid and targetHumanoid.Health > 0 then
                            targetHumanoid:TakeDamage(effect.config.tickDamage)
                        end
                    end
                end
            end
        end
    end
end)

-- ===== Hitbox Detection =====
function CombatManager.createHitbox(origin, size, excludeModel, callback)
    local overlapParams = OverlapParams.new()
    overlapParams.FilterDescendantsInstances = { excludeModel }
    overlapParams.FilterType = Enum.RaycastFilterType.Exclude
    
    local parts = workspace:GetPartBoundsInBox(
        CFrame.new(origin),
        size,
        overlapParams
    )
    
    local hitTargets = {}
    
    for _, part in ipairs(parts) do
        local model = part:FindFirstAncestorOfClass("Model")
        if model then
            local humanoid = model:FindFirstChild("Humanoid")
            if humanoid and humanoid.Health > 0 then
                -- หลีกเลี่ยง hit ซ้ำ
                local alreadyHit = false
                for _, hit in ipairs(hitTargets) do
                    if hit == model then
                        alreadyHit = true
                        break
                    end
                end
                
                if not alreadyHit then
                    table.insert(hitTargets, model)
                    if callback then
                        callback(model, humanoid)
                    end
                end
            end
        end
    end
    
    return hitTargets
end

return CombatManager
```

## Combat Client (Input Handling)

```lua
-- LocalScript ใน StarterPlayerScripts

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local rootPart = character:WaitForChild("HumanoidRootPart")

local Events = ReplicatedStorage:WaitForChild("Events")
local attackRE = Events:WaitForChild("Combat_Attack")

-- ===== Combat State =====
local isAttacking = false
local comboCount = 0
local lastAttackTime = 0
local comboResetTime = 2.5  -- วินาที

-- ===== Combo System =====
local combos = {
    [1] = { name = "Strike1", damage = 15, range = 5, knockback = Vector3.new(0, 10, -15) },
    [2] = { name = "Strike2", damage = 12, range = 5, knockback = Vector3.new(0, 5, -10) },
    [3] = { name = "Strike3", damage = 20, range = 7, knockback = Vector3.new(0, 20, -25) }
}

local function performAttack()
    if isAttacking then return end
    if humanoid.Health <= 0 then return end
    
    isAttacking = true
    
    -- Combo
    local now = os.clock()
    if now - lastAttackTime > comboResetTime then
        comboCount = 1
    else
        comboCount = (comboCount % #combos) + 1
    end
    lastAttackTime = now
    
    local combo = combos[comboCount]
    
    -- ส่ง attack event ไป Server
    attackRE:FireServer({
        combo = comboCount,
        direction = rootPart.CFrame.LookVector,
        position = rootPart.Position
    })
    
    print(string.format("⚔️ Combo %d: %s", comboCount, combo.name))
    
    -- Cooldown
    task.delay(0.4, function()
        isAttacking = false
    end)
end

-- ===== Input =====
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    -- Left Click = Attack
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        performAttack()
    end
    
    -- Q = Skill 1
    if input.KeyCode == Enum.KeyCode.Q then
        local skillRE = Events:FindFirstChild("Combat_UseSkill")
        if skillRE then
            skillRE:FireServer({ skillId = "skill_1", target = nil })
        end
    end
    
    -- E = Skill 2
    if input.KeyCode == Enum.KeyCode.E then
        local skillRE = Events:FindFirstChild("Combat_UseSkill")
        if skillRE then
            skillRE:FireServer({ skillId = "skill_2", target = nil })
        end
    end
end)
```

## Damage Number UI

```lua
-- LocalScript ใน StarterGui

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

local showDmgEvent = ReplicatedStorage.Events:WaitForChild("UI_ShowDamageNumber")

local damageColors = {
    normal = Color3.fromRGB(255, 255, 255),
    crit = Color3.fromRGB(255, 200, 0),
    miss = Color3.fromRGB(150, 150, 150),
    fire = Color3.fromRGB(255, 100, 0),
    ice = Color3.fromRGB(100, 200, 255),
    poison = Color3.fromRGB(100, 220, 50),
    heal = Color3.fromRGB(100, 255, 100)
}

showDmgEvent.OnClientEvent:Connect(function(data)
    -- Convert world position to screen position
    local screenPos, onScreen = camera:WorldToScreenPoint(data.position + Vector3.new(0, 3, 0))
    if not onScreen then return end
    
    -- สร้าง damage label
    local label = Instance.new("TextLabel")
    label.Text = data.hitType == "miss" and "Miss" or tostring(data.damage)
    
    if data.hitType == "crit" then
        label.Text = "⚡ " .. data.damage .. "!"
    end
    
    label.TextSize = data.hitType == "crit" and 24 or 18
    label.Font = Enum.Font.GothamBold
    label.TextColor3 = damageColors[data.hitType] or damageColors[data.damageType] or damageColors.normal
    label.BackgroundTransparency = 1
    label.Size = UDim2.new(0, 100, 0, 40)
    label.Position = UDim2.fromOffset(screenPos.X - 50, screenPos.Y - 20)
    label.ZIndex = 10
    label.TextStrokeColor3 = Color3.new(0,0,0)
    label.TextStrokeTransparency = 0.5
    label.Parent = player.PlayerGui
    
    -- Animate up and fade
    TweenService:Create(label, TweenInfo.new(1.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
        Position = UDim2.fromOffset(screenPos.X - 50 + math.random(-20, 20), screenPos.Y - 80),
        TextTransparency = 1,
        TextStrokeTransparency = 1
    }):Play()
    
    game:GetService("Debris"):AddItem(label, 1.3)
end)
```

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Block System
เพิ่มระบบป้องกัน (Block) ด้วยการกด Space ลด damage 80%

### แบบฝึกหัดที่ 2: Elemental Weakness
สร้างระบบ weakness/resistance (ไฟ ทำ damage 2x กับ น้ำแข็ง)

### แบบฝึกหัดที่ 3: Combo Finisher
เพิ่ม combo finisher ที่ทำ damage 3x หลัง combo 3 ท่า

## สรุป

ระบบต่อสู้ที่ดีต้องมี:
- **Damage Formula** ที่ balance
- **Hit Detection** ที่แม่นยำ  
- **Status Effects** ที่หลากหลาย
- **Visual Feedback** - damage numbers, effects
- **Server Validation** - ตรวจสอบทุกอย่างบน server
- **Combo System** - ทำให้การต่อสู้น่าสนุก
