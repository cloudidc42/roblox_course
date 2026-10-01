# Part 58: Skill System - ระบบสกิลและความสามารถพิเศษ

## บทนำ

Skill System เป็นระบบที่ทำให้ตัวละครมีความสามารถพิเศษ (Abilities) ที่ใช้พลังงาน MP/Mana เพื่อสร้างผลกระทบต่างๆ ในเกม ตั้งแต่การโจมตี AOE ไปจนถึงการรักษา การ Buff และ การ Debuff

## สถาปัตยกรรมระบบสกิล

```
SkillSystem/
├── SkillData (Module) - ฐานข้อมูลสกิล
├── SkillManager (Server) - execute สกิล
├── SkillController (Client) - รับ input + แสดง UI
├── SkillTreeManager (Server/Client) - unlock สกิล
└── SkillEffects (Client) - Visual effects
```

## SkillData Module

```lua
-- ReplicatedStorage/Shared/SkillData.lua

local SkillData = {}

-- ===== Skill Types =====
SkillData.Types = {
    ACTIVE = "Active",       -- ใช้งานเอง
    PASSIVE = "Passive",     -- ทำงานอัตโนมัติ
    ULTIMATE = "Ultimate",   -- พลังพิเศษใช้ชาร์จ
    TOGGLE = "Toggle"        -- เปิด/ปิดได้
}

-- ===== Target Types =====
SkillData.Targets = {
    SELF = "Self",
    SINGLE = "Single",
    AOE = "AOE",
    LINE = "Line",
    DIRECTION = "Direction",
    ALL_ALLIES = "AllAllies",
    ALL_ENEMIES = "AllEnemies"
}

-- ===== Effect Types =====
SkillData.Effects = {
    DAMAGE = "Damage",
    HEAL = "Heal",
    BUFF = "Buff",
    DEBUFF = "Debuff",
    TELEPORT = "Teleport",
    SUMMON = "Summon",
    SHIELD = "Shield"
}

-- ===== Skill Database =====
SkillData.Skills = {
    
    -- ===== WARRIOR SKILLS =====
    ["slash"] = {
        id = "slash",
        name = "Slash",
        nameThai = "ฟันดาบ",
        type = "Active",
        class = "Warrior",
        icon = "rbxassetid://slash_icon",
        
        -- Resource cost
        mpCost = 15,
        cooldown = 3,           -- วินาที
        castTime = 0,           -- instant cast
        channelTime = 0,        -- ไม่ต้อง channel
        
        -- Target
        targetType = "AOE",
        range = 8,
        aoeRadius = 5,
        
        -- Damage
        baseDamage = 40,
        damageMultiplier = 1.5, -- 1.5x ของ attack damage
        damageType = "Physical",
        
        -- Level scaling (ต่อ skill level)
        levelScaling = {
            damage = 5,          -- +5 damage ต่อ level
            mpCostReduction = 1  -- -1 mp ต่อ level
        },
        
        -- Effects
        effects = {
            {type = "Damage", value = 40},
            {type = "Knockback", value = 15}
        },
        
        -- Unlock
        requireClassLevel = 1,
        
        description = "ฟันดาบพื้นที่รอบตัว ทำ 1.5x damage ศัตรูหลายตัว"
    },
    
    ["battle_cry"] = {
        id = "battle_cry",
        name = "Battle Cry",
        nameThai = "เสียงร้องนักรบ",
        type = "Active",
        class = "Warrior",
        icon = "rbxassetid://battle_cry_icon",
        
        mpCost = 25,
        cooldown = 30,
        castTime = 0.5,
        
        targetType = "Self",
        range = 20,              -- radius ของ buff
        
        baseDamage = 0,
        
        effects = {
            {type = "Buff", stat = "Attack", value = 30, duration = 15},
            {type = "Buff", stat = "Speed", value = 5, duration = 15},
            {type = "AoeBuff", radius = 20, stat = "Attack", value = 15, duration = 15}
        },
        
        requireClassLevel = 5,
        
        description = "เพิ่ม Attack ตัวเองและพันธมิตรรอบข้าง 15 วินาที"
    },
    
    ["shield_bash"] = {
        id = "shield_bash",
        name = "Shield Bash",
        nameThai = "ฟาดโล่",
        type = "Active",
        class = "Warrior",
        
        mpCost = 20,
        cooldown = 8,
        castTime = 0,
        
        targetType = "Single",
        range = 5,
        
        baseDamage = 30,
        damageMultiplier = 1.0,
        
        effects = {
            {type = "Damage", value = 30},
            {type = "Stun", duration = 2},
            {type = "Knockback", value = 20}
        },
        
        requireClassLevel = 10,
        description = "ฟาดด้วยโล่ stun ศัตรู 2 วินาที"
    },
    
    ["whirlwind"] = {
        id = "whirlwind",
        name = "Whirlwind",
        nameThai = "พายุหมุน",
        type = "Active",
        class = "Warrior",
        
        mpCost = 40,
        cooldown = 20,
        castTime = 0,
        channelTime = 2,  -- หมุนอยู่ 2 วินาที
        
        targetType = "AOE",
        aoeRadius = 7,
        
        baseDamage = 25,
        
        effects = {
            {type = "ChannelDamage", value = 25, tickRate = 0.3}
        },
        
        requireClassLevel = 20,
        description = "หมุนตัวโจมตีพื้นที่ต่อเนื่อง 2 วินาที"
    },
    
    -- ===== MAGE SKILLS =====
    ["fireball"] = {
        id = "fireball",
        name = "Fireball",
        nameThai = "ลูกไฟ",
        type = "Active",
        class = "Mage",
        icon = "rbxassetid://fireball_icon",
        
        mpCost = 25,
        cooldown = 5,
        castTime = 1.0,
        
        targetType = "Direction",
        range = 60,
        
        baseDamage = 70,
        damageMultiplier = 2.0,
        damageType = "Fire",
        
        projectile = {
            speed = 60,
            gravity = 0,
            explodeOnHit = true,
            explosionRadius = 8,
            modelId = "rbxassetid://fireball_model"
        },
        
        effects = {
            {type = "Damage", value = 70},
            {type = "AoeDamage", radius = 8, value = 40},
            {type = "Burn", damage = 5, duration = 4, chance = 0.5}
        },
        
        requireClassLevel = 1,
        description = "ยิงลูกไฟระเบิดเมื่อกระทบ ทำ AOE damage"
    },
    
    ["ice_nova"] = {
        id = "ice_nova",
        name = "Ice Nova",
        nameThai = "โนวาน้ำแข็ง",
        type = "Active",
        class = "Mage",
        
        mpCost = 35,
        cooldown = 12,
        castTime = 0.5,
        
        targetType = "AOE",
        range = 12,
        
        baseDamage = 50,
        damageType = "Ice",
        
        effects = {
            {type = "AoeDamage", radius = 12, value = 50},
            {type = "Freeze", duration = 2, chance = 0.6}
        },
        
        requireClassLevel = 10,
        description = "ระเบิดน้ำแข็งรอบตัว freeze ศัตรู"
    },
    
    ["meteor"] = {
        id = "meteor",
        name = "Meteor",
        nameThai = "อุกกาบาต",
        type = "Ultimate",
        class = "Mage",
        
        mpCost = 100,
        cooldown = 60,
        castTime = 2.0,   -- cast นาน
        
        targetType = "AOE",
        range = 50,
        aoeRadius = 15,
        
        baseDamage = 300,
        damageType = "Fire",
        
        effects = {
            {type = "Damage", value = 300, radius = 15},
            {type = "Burn", damage = 20, duration = 6},
            {type = "Stun", duration = 3}
        },
        
        requireClassLevel = 30,
        description = "เรียกอุกกาบาตตกลงมา ทำ massive AOE damage"
    },
    
    -- ===== HEALER SKILLS =====
    ["heal"] = {
        id = "heal",
        name = "Heal",
        nameThai = "รักษา",
        type = "Active",
        class = "Healer",
        
        mpCost = 30,
        cooldown = 4,
        castTime = 1.0,
        
        targetType = "Single",
        range = 20,
        
        baseHeal = 80,
        healMultiplier = 1.0,
        
        effects = {
            {type = "Heal", value = 80}
        },
        
        requireClassLevel = 1,
        description = "รักษา HP เป้าหมาย"
    },
    
    ["mass_heal"] = {
        id = "mass_heal",
        name = "Mass Heal",
        nameThai = "รักษาหมู่",
        type = "Active",
        class = "Healer",
        
        mpCost = 60,
        cooldown = 15,
        castTime = 1.5,
        
        targetType = "AOE",
        range = 20,
        
        baseHeal = 50,
        
        effects = {
            {type = "AoeHeal", radius = 20, value = 50}
        },
        
        requireClassLevel = 15,
        description = "รักษาพันธมิตรทุกคนรอบข้าง"
    },
    
    ["resurrection"] = {
        id = "resurrection",
        name = "Resurrection",
        nameThai = "ฟื้นคืนชีพ",
        type = "Ultimate",
        class = "Healer",
        
        mpCost = 150,
        cooldown = 120,
        castTime = 3.0,
        
        targetType = "Single",
        range = 10,
        
        effects = {
            {type = "Resurrect", hpPercent = 0.5}  -- ฟื้น 50% HP
        },
        
        requireClassLevel = 40,
        description = "ฟื้นคืนชีพพันธมิตรที่เสียชีวิต"
    },
    
    -- ===== PASSIVE SKILLS =====
    ["iron_skin"] = {
        id = "iron_skin",
        name = "Iron Skin",
        nameThai = "หนังเหล็ก",
        type = "Passive",
        class = "Warrior",
        
        effects = {
            {type = "StatBonus", stat = "Defense", value = 20}
        },
        
        requireClassLevel = 3,
        description = "เพิ่ม Defense ถาวร +20"
    },
    
    ["mana_efficiency"] = {
        id = "mana_efficiency",
        name = "Mana Efficiency",
        nameThai = "ประหยัด MP",
        type = "Passive",
        class = "Mage",
        
        effects = {
            {type = "ReduceMpCost", percent = 0.15}  -- ลด MP cost 15%
        },
        
        requireClassLevel = 5,
        description = "ลด MP cost ของสกิลทุกอย่าง 15%"
    }
}

-- ===== Skill Tree =====
-- กำหนดว่าสกิลไหนต้อง unlock สกิลไหนก่อน
SkillData.SkillTree = {
    Warrior = {
        {id = "slash", requires = {}},
        {id = "iron_skin", requires = {}},
        {id = "shield_bash", requires = {"slash"}},
        {id = "battle_cry", requires = {"slash", "iron_skin"}},
        {id = "whirlwind", requires = {"shield_bash", "battle_cry"}}
    },
    Mage = {
        {id = "fireball", requires = {}},
        {id = "mana_efficiency", requires = {}},
        {id = "ice_nova", requires = {"fireball"}},
        {id = "meteor", requires = {"fireball", "ice_nova"}}
    },
    Healer = {
        {id = "heal", requires = {}},
        {id = "mass_heal", requires = {"heal"}},
        {id = "resurrection", requires = {"mass_heal"}}
    }
}

-- ===== Skill Points Cost =====
SkillData.SkillPointCost = {
    Active = 1,
    Passive = 1,
    Ultimate = 3
}

-- ===== Helper Functions =====
function SkillData.getSkill(skillId)
    return SkillData.Skills[skillId]
end

function SkillData.getClassSkills(className)
    local skills = {}
    for id, skill in pairs(SkillData.Skills) do
        if skill.class == className then
            table.insert(skills, skill)
        end
    end
    return skills
end

function SkillData.canUnlock(skillId, unlockedSkills)
    -- หาใน tree
    local skill = SkillData.Skills[skillId]
    if not skill then return false end
    
    local tree = SkillData.SkillTree[skill.class]
    if not tree then return true end
    
    for _, entry in ipairs(tree) do
        if entry.id == skillId then
            -- ตรวจสอบ prerequisites
            for _, reqId in ipairs(entry.requires) do
                if not unlockedSkills[reqId] then
                    return false
                end
            end
            return true
        end
    end
    
    return true
end

return SkillData
```

## SkillManager - Server

```lua
-- ServerScriptService/SkillManager.lua

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local SkillData = require(ReplicatedStorage.Shared.SkillData)

-- ===== Remotes =====
local remotes = ReplicatedStorage:WaitForChild("Remotes")
local useSkillEvent = remotes:WaitForChild("UseSkill")
local unlockSkillEvent = remotes:WaitForChild("UnlockSkill")
local skillResultEvent = remotes:WaitForChild("SkillResult")
local skillFeedbackEvent = remotes:WaitForChild("SkillFeedback")

-- ===== Player Skill Data =====
local playerSkills = {}

local function getPlayerSkills(player)
    if not playerSkills[player] then
        playerSkills[player] = {
            unlockedSkills = {},    -- [skillId] = skillLevel
            cooldowns = {},         -- [skillId] = endTime
            activeEffects = {},     -- active buffs/debuffs
            skillPoints = 0,
            isChanneling = false,
            channelingSkill = nil
        }
    end
    return playerSkills[player]
end

-- ===== Cooldown Management =====
local function isOnCooldown(player, skillId)
    local skills = getPlayerSkills(player)
    local endTime = skills.cooldowns[skillId]
    if not endTime then return false end
    return os.clock() < endTime
end

local function setCooldown(player, skillId, duration)
    local skills = getPlayerSkills(player)
    skills.cooldowns[skillId] = os.clock() + duration
    
    -- แจ้ง client
    skillFeedbackEvent:FireClient(player, {
        type = "CooldownStart",
        skillId = skillId,
        duration = duration
    })
end

local function getRemainingCooldown(player, skillId)
    local skills = getPlayerSkills(player)
    local endTime = skills.cooldowns[skillId]
    if not endTime then return 0 end
    return math.max(0, endTime - os.clock())
end

-- ===== MP Management =====
local function getPlayerMP(player)
    local stats = player:FindFirstChild("Stats")
    if not stats then return 0, 0 end
    
    local currentMP = stats:FindFirstChild("MP")
    local maxMP = stats:FindFirstChild("MaxMP")
    
    return (currentMP and currentMP.Value or 0), (maxMP and maxMP.Value or 100)
end

local function consumeMP(player, amount)
    local stats = player:FindFirstChild("Stats")
    if not stats then return false end
    
    local currentMP = stats:FindFirstChild("MP")
    if not currentMP then return false end
    
    if currentMP.Value < amount then
        return false
    end
    
    currentMP.Value = currentMP.Value - amount
    return true
end

-- ===== Skill Level Scaling =====
local function getScaledValue(skill, skillLevel, baseValue, scalingKey)
    if not skill.levelScaling or not skill.levelScaling[scalingKey] then
        return baseValue
    end
    return baseValue + (skill.levelScaling[scalingKey] * (skillLevel - 1))
end

-- ===== Effect Application =====
local function applyEffect(player, target, effect, skill, skillLevel)
    local targetChar = target
    local targetHum = targetChar and targetChar:FindFirstChild("Humanoid")
    
    if effect.type == "Damage" then
        if not targetHum or targetHum.Health <= 0 then return end
        local damage = getScaledValue(skill, skillLevel, effect.value, "damage")
        targetHum:TakeDamage(damage)
        
    elseif effect.type == "AoeDamage" then
        -- AOE damage
        local centerChar = player.Character
        if not centerChar then return end
        
        local centerRoot = centerChar:FindFirstChild("HumanoidRootPart")
        if not centerRoot then return end
        
        -- ถ้ามี target ใช้ target เป็น center
        local center = centerRoot.Position
        if target then
            local targetRoot = target:FindFirstChild("HumanoidRootPart")
            if targetRoot then
                center = targetRoot.Position
            end
        end
        
        local params = OverlapParams.new()
        local hits = workspace:GetPartBoundsInRadius(center, effect.radius, params)
        
        local hitChars = {}
        for _, part in ipairs(hits) do
            local char = part:FindFirstAncestorOfClass("Model")
            if char and not hitChars[char] and char ~= player.Character then
                local hum = char:FindFirstChild("Humanoid")
                if hum and hum.Health > 0 then
                    hitChars[char] = true
                    local damage = getScaledValue(skill, skillLevel, effect.value, "damage")
                    
                    -- Falloff
                    local charRoot = char:FindFirstChild("HumanoidRootPart")
                    if charRoot then
                        local dist = (charRoot.Position - center).Magnitude
                        local falloff = 1 - (dist / effect.radius) * 0.5
                        damage = math.floor(damage * falloff)
                    end
                    
                    hum:TakeDamage(damage)
                end
            end
        end
        
    elseif effect.type == "Heal" then
        if not targetHum then return end
        local healAmount = getScaledValue(skill, skillLevel, effect.value, "heal")
        local newHP = math.min(targetHum.MaxHealth, targetHum.Health + healAmount)
        targetHum.Health = newHP
        
    elseif effect.type == "AoeHeal" then
        local playerChar = player.Character
        if not playerChar then return end
        
        local root = playerChar:FindFirstChild("HumanoidRootPart")
        if not root then return end
        
        local params = OverlapParams.new()
        local hits = workspace:GetPartBoundsInRadius(root.Position, effect.radius, params)
        
        local healedChars = {}
        for _, part in ipairs(hits) do
            local char = part:FindFirstAncestorOfClass("Model")
            if char and not healedChars[char] then
                local hum = char:FindFirstChild("Humanoid")
                if hum and hum.Health > 0 and hum.Health < hum.MaxHealth then
                    healedChars[char] = true
                    local healAmount = getScaledValue(skill, skillLevel, effect.value, "heal")
                    hum.Health = math.min(hum.MaxHealth, hum.Health + healAmount)
                end
            end
        end
        
    elseif effect.type == "Buff" then
        local playerSkillsData = getPlayerSkills(player)
        
        -- Apply buff
        local buffId = skill.id .. "_" .. effect.stat
        
        -- Remove existing buff of same type
        local existing = playerSkillsData.activeEffects[buffId]
        if existing and existing.cleanup then
            existing.cleanup()
        end
        
        -- Apply stat change
        local stats = player:FindFirstChild("Stats")
        if stats then
            local stat = stats:FindFirstChild(effect.stat)
            if stat then
                stat.Value = stat.Value + effect.value
                
                playerSkillsData.activeEffects[buffId] = {
                    endTime = os.clock() + effect.duration,
                    cleanup = function()
                        stat.Value = stat.Value - effect.value
                        playerSkillsData.activeEffects[buffId] = nil
                    end
                }
                
                -- Auto remove after duration
                task.delay(effect.duration, function()
                    if playerSkillsData.activeEffects[buffId] then
                        playerSkillsData.activeEffects[buffId].cleanup()
                    end
                end)
            end
        end
        
    elseif effect.type == "Stun" then
        if not targetHum then return end
        
        local prevSpeed = targetHum.WalkSpeed
        local prevJump = targetHum.JumpPower
        targetHum.WalkSpeed = 0
        targetHum.JumpPower = 0
        
        -- Stun tag
        local stunTag = Instance.new("BoolValue")
        stunTag.Name = "Stunned"
        stunTag.Parent = targetChar
        
        task.delay(effect.duration, function()
            if stunTag.Parent then
                stunTag:Destroy()
                targetHum.WalkSpeed = prevSpeed
                targetHum.JumpPower = prevJump
            end
        end)
        
    elseif effect.type == "Burn" then
        if not targetHum then return end
        if math.random() > (effect.chance or 1) then return end
        
        -- Remove existing burn
        local existingBurn = targetChar:FindFirstChild("BurnEffect")
        if existingBurn then existingBurn:Destroy() end
        
        local burnTag = Instance.new("BoolValue")
        burnTag.Name = "BurnEffect"
        burnTag.Parent = targetChar
        
        task.spawn(function()
            local endTime = os.clock() + effect.duration
            while os.clock() < endTime and burnTag.Parent do
                task.wait(1)
                if targetHum.Health > 0 then
                    targetHum:TakeDamage(effect.damage)
                end
            end
            if burnTag.Parent then burnTag:Destroy() end
        end)
        
    elseif effect.type == "Freeze" then
        if not targetHum then return end
        if math.random() > (effect.chance or 1) then return end
        
        local prevSpeed = targetHum.WalkSpeed
        targetHum.WalkSpeed = 0
        
        local freezeTag = Instance.new("BoolValue")
        freezeTag.Name = "Frozen"
        freezeTag.Parent = targetChar
        
        task.delay(effect.duration, function()
            if freezeTag.Parent then
                freezeTag:Destroy()
                targetHum.WalkSpeed = prevSpeed
            end
        end)
        
    elseif effect.type == "Resurrect" then
        -- หาผู้เล่นที่ตาย
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= player and p.Character then
                local targetChar = p.Character
                local targetRoot = targetChar:FindFirstChild("HumanoidRootPart")
                if not targetRoot then continue end
                
                local dist = (player.Character.HumanoidRootPart.Position - targetRoot.Position).Magnitude
                if dist <= skill.range then
                    local hum = targetChar:FindFirstChild("Humanoid")
                    if hum and hum.Health <= 0 then
                        -- Resurrect!
                        p:LoadCharacter()
                        task.wait(1)
                        if p.Character then
                            local newHum = p.Character:FindFirstChild("Humanoid")
                            if newHum then
                                newHum.Health = newHum.MaxHealth * (effect.hpPercent or 0.5)
                            end
                        end
                        break
                    end
                end
            end
        end
        
    elseif effect.type == "StatBonus" then
        -- Passive stat bonus (ใช้ตอน equip skill เท่านั้น)
        local stats = player:FindFirstChild("Stats")
        if stats then
            local stat = stats:FindFirstChild(effect.stat)
            if stat then
                stat.Value = stat.Value + effect.value
            end
        end
        
    elseif effect.type == "ReduceMpCost" then
        -- Passive MP cost reduction (จัดการตอนคำนวณ cost)
        -- เก็บไว้ใน passive effects
        local skills = getPlayerSkills(player)
        skills.mpCostReduction = (skills.mpCostReduction or 0) + effect.percent
        
    elseif effect.type == "ChannelDamage" then
        -- Channel damage (ทำงานตลอด channelTime)
        -- จัดการโดย channel system ด้านล่าง
        
    elseif effect.type == "Knockback" then
        if not targetChar then return end
        local targetRoot = targetChar:FindFirstChild("HumanoidRootPart")
        if not targetRoot then return end
        
        local playerRoot = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
        if not playerRoot then return end
        
        local direction = (targetRoot.Position - playerRoot.Position).Unit
        direction = Vector3.new(direction.X, 0.3, direction.Z).Unit
        
        local velocity = Instance.new("LinearVelocity")
        velocity.VectorVelocity = direction * effect.value
        local att = targetRoot:FindFirstChild("RootAttachment") or Instance.new("Attachment", targetRoot)
        velocity.Attachment0 = att
        velocity.MaxForce = math.huge
        velocity.Parent = targetRoot
        
        game:GetService("Debris"):AddItem(velocity, 0.2)
    end
end

-- ===== Channel System =====
local function channelSkill(player, skill, skillLevel, targetPos)
    local skills = getPlayerSkills(player)
    skills.isChanneling = true
    skills.channelingSkill = skill.id
    
    local startTime = os.clock()
    local endTime = startTime + skill.channelTime
    
    -- แจ้ง client เริ่ม channel
    skillFeedbackEvent:FireClient(player, {
        type = "ChannelStart",
        skillId = skill.id,
        duration = skill.channelTime
    })
    
    -- Tick damage loop
    for _, effect in ipairs(skill.effects) do
        if effect.type == "ChannelDamage" then
            task.spawn(function()
                while os.clock() < endTime and skills.isChanneling do
                    task.wait(effect.tickRate)
                    
                    if not skills.isChanneling then break end
                    
                    -- AOE damage ณ ตำแหน่ง player
                    local playerChar = player.Character
                    if not playerChar then break end
                    
                    local root = playerChar:FindFirstChild("HumanoidRootPart")
                    if not root then break end
                    
                    local params = OverlapParams.new()
                    local hits = workspace:GetPartBoundsInRadius(root.Position, skill.aoeRadius or 7, params)
                    
                    local hitChars = {}
                    for _, part in ipairs(hits) do
                        local char = part:FindFirstAncestorOfClass("Model")
                        if char and not hitChars[char] and char ~= playerChar then
                            local hum = char:FindFirstChild("Humanoid")
                            if hum and hum.Health > 0 then
                                hitChars[char] = true
                                hum:TakeDamage(effect.value)
                            end
                        end
                    end
                end
            end)
        end
    end
    
    -- รอจบ channel
    task.delay(skill.channelTime, function()
        if skills.channelingSkill == skill.id then
            skills.isChanneling = false
            skills.channelingSkill = nil
            
            skillFeedbackEvent:FireClient(player, {
                type = "ChannelEnd",
                skillId = skill.id
            })
        end
    end)
end

-- ===== Main Skill Use =====
local function useSkill(player, skillId, targetInfo)
    local skillsData = getPlayerSkills(player)
    
    -- ตรวจสอบว่าปลดล็อคสกิลหรือยัง
    if not skillsData.unlockedSkills[skillId] then
        skillFeedbackEvent:FireClient(player, {
            type = "Error",
            message = "สกิลยังไม่ได้ปลดล็อค"
        })
        return false
    end
    
    local skill = SkillData.getSkill(skillId)
    if not skill then return false end
    
    local skillLevel = skillsData.unlockedSkills[skillId] or 1
    
    -- ตรวจสอบ cooldown
    if isOnCooldown(player, skillId) then
        local remaining = getRemainingCooldown(player, skillId)
        skillFeedbackEvent:FireClient(player, {
            type = "OnCooldown",
            skillId = skillId,
            remaining = remaining
        })
        return false
    end
    
    -- ตรวจสอบ channeling
    if skillsData.isChanneling then
        return false
    end
    
    -- คำนวณ MP cost (รวม passive reductions)
    local mpCost = skill.mpCost
    if skill.type == "Active" then
        mpCost = math.floor(mpCost * (1 - (skillsData.mpCostReduction or 0)))
    end
    
    -- ตรวจสอบ MP
    if skill.mpCost > 0 then
        if not consumeMP(player, mpCost) then
            skillFeedbackEvent:FireClient(player, {
                type = "InsufficientMP",
                required = mpCost
            })
            return false
        end
    end
    
    -- Cast time (ถ้ามี)
    if skill.castTime > 0 then
        skillFeedbackEvent:FireClient(player, {
            type = "CastStart",
            skillId = skillId,
            duration = skill.castTime
        })
        task.wait(skill.castTime)
        
        -- ตรวจสอบว่ายัง alive อยู่
        local char = player.Character
        if not char then return false end
        local hum = char:FindFirstChild("Humanoid")
        if not hum or hum.Health <= 0 then return false end
    end
    
    -- Set cooldown
    setCooldown(player, skillId, skill.cooldown)
    
    -- หา target
    local target = nil
    if targetInfo and targetInfo.targetName then
        target = workspace:FindFirstChild(targetInfo.targetName)
    end
    
    -- Channel skill
    if skill.channelTime and skill.channelTime > 0 then
        channelSkill(player, skill, skillLevel, targetInfo and targetInfo.position)
    else
        -- Apply effects ทันที
        for _, effect in ipairs(skill.effects) do
            applyEffect(player, target, effect, skill, skillLevel)
        end
    end
    
    -- แจ้ง all clients แสดง effect
    local skillResultData = {
        skillId = skillId,
        casterId = player.UserId,
        targetName = targetInfo and targetInfo.targetName,
        position = targetInfo and targetInfo.position,
        direction = targetInfo and targetInfo.direction
    }
    
    -- Fire เฉพาะบาง skills ให้ทุกคนเห็น
    if skill.targetType == "AOE" or skill.targetType == "Direction" then
        skillResultEvent:FireAllClients(skillResultData)
    else
        skillResultEvent:FireClient(player, skillResultData)
    end
    
    print(string.format("[Skill] %s used %s (level %d)", player.Name, skill.name, skillLevel))
    return true
end

-- ===== Unlock Skill =====
local function unlockSkill(player, skillId)
    local skillsData = getPlayerSkills(player)
    
    if skillsData.unlockedSkills[skillId] then
        -- Upgrade existing skill
        local currentLevel = skillsData.unlockedSkills[skillId]
        local skill = SkillData.getSkill(skillId)
        
        if not skill then return false end
        
        -- ตรวจสอบ skill points
        local cost = SkillData.SkillPointCost[skill.type] or 1
        if skillsData.skillPoints < cost then
            return false, "INSUFFICIENT_SKILL_POINTS"
        end
        
        skillsData.skillPoints = skillsData.skillPoints - cost
        skillsData.unlockedSkills[skillId] = currentLevel + 1
        
        print(player.Name .. " upgraded " .. skillId .. " to level " .. (currentLevel + 1))
        return true
    else
        -- Unlock new skill
        local skill = SkillData.getSkill(skillId)
        if not skill then return false end
        
        -- ตรวจสอบ prerequisites
        if not SkillData.canUnlock(skillId, skillsData.unlockedSkills) then
            return false, "PREREQUISITES_NOT_MET"
        end
        
        -- ตรวจสอบ skill points
        local cost = SkillData.SkillPointCost[skill.type] or 1
        if skillsData.skillPoints < cost then
            return false, "INSUFFICIENT_SKILL_POINTS"
        end
        
        -- ตรวจสอบ class level
        local stats = player:FindFirstChild("Stats")
        local classLevel = stats and stats:FindFirstChild("ClassLevel") and stats.ClassLevel.Value or 0
        if classLevel < (skill.requireClassLevel or 0) then
            return false, "CLASS_LEVEL_REQUIREMENT"
        end
        
        skillsData.skillPoints = skillsData.skillPoints - cost
        skillsData.unlockedSkills[skillId] = 1
        
        -- Apply passive immediately
        if skill.type == "Passive" then
            for _, effect in ipairs(skill.effects) do
                applyEffect(player, nil, effect, skill, 1)
            end
        end
        
        print(player.Name .. " unlocked skill: " .. skillId)
        return true
    end
end

-- ===== Event Handlers =====
useSkillEvent.OnServerEvent:Connect(function(player, skillId, targetInfo)
    useSkill(player, skillId, targetInfo)
end)

unlockSkillEvent.OnServerEvent:Connect(function(player, skillId)
    local success, reason = unlockSkill(player, skillId)
    skillFeedbackEvent:FireClient(player, {
        type = success and "SkillUnlocked" or "UnlockFailed",
        skillId = skillId,
        reason = reason
    })
end)

-- ===== Test Setup =====
Players.PlayerAdded:Connect(function(player)
    -- ให้ skill points เพื่อทดสอบ
    local skills = getPlayerSkills(player)
    skills.skillPoints = 10
    
    -- Unlock สกิลเริ่มต้น
    task.wait(2)
    unlockSkill(player, "slash")
    unlockSkill(player, "fireball")
end)

Players.PlayerRemoving:Connect(function(player)
    playerSkills[player] = nil
end)
```

## SkillController - Client

```lua
-- StarterPlayerScripts/SkillController.lua

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

local SkillData = require(ReplicatedStorage.Shared.SkillData)

-- ===== Remotes =====
local remotes = ReplicatedStorage:WaitForChild("Remotes")
local useSkillEvent = remotes:WaitForChild("UseSkill")
local skillFeedbackEvent = remotes:WaitForChild("SkillFeedback")
local skillResultEvent = remotes:WaitForChild("SkillResult")

-- ===== Skill Hotkeys =====
local SKILL_HOTKEYS = {
    [Enum.KeyCode.Q] = 1,  -- Skill slot 1
    [Enum.KeyCode.E] = 2,  -- Skill slot 2
    [Enum.KeyCode.R] = 3,  -- Skill slot 3
    [Enum.KeyCode.F] = 4   -- Skill slot 4
}

-- ===== Player Skills =====
local equippedSkills = {"slash", "fireball", nil, nil}  -- สกิลที่ติด
local skillCooldowns = {}  -- [skillId] = {startTime, duration}

-- ===== Skill Bar UI =====
local function createSkillBar()
    local gui = Instance.new("ScreenGui")
    gui.Name = "SkillBar"
    gui.ResetOnSpawn = false
    gui.Parent = player.PlayerGui
    
    local frame = Instance.new("Frame")
    frame.Name = "SkillBarFrame"
    frame.Size = UDim2.new(0, 260, 0, 70)
    frame.Position = UDim2.new(0.5, -130, 1, -90)
    frame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    frame.BackgroundTransparency = 0.3
    frame.BorderSizePixel = 0
    frame.Parent = gui
    
    Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 8)
    
    local layout = Instance.new("UIListLayout")
    layout.FillDirection = Enum.FillDirection.Horizontal
    layout.Padding = UDim.new(0, 5)
    layout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    layout.VerticalAlignment = Enum.VerticalAlignment.Center
    layout.Parent = frame
    
    -- สร้าง 4 skill slots
    for i = 1, 4 do
        local slot = Instance.new("Frame")
        slot.Name = "Slot" .. i
        slot.Size = UDim2.new(0, 55, 0, 55)
        slot.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        slot.BorderSizePixel = 0
        slot.Parent = frame
        
        Instance.new("UICorner", slot).CornerRadius = UDim.new(0, 6)
        
        -- Icon
        local icon = Instance.new("ImageLabel")
        icon.Name = "Icon"
        icon.Size = UDim2.new(1, 0, 1, 0)
        icon.BackgroundTransparency = 1
        icon.Image = ""
        icon.Parent = slot
        
        -- Cooldown overlay
        local cdOverlay = Instance.new("Frame")
        cdOverlay.Name = "CooldownOverlay"
        cdOverlay.Size = UDim2.new(1, 0, 1, 0)
        cdOverlay.BackgroundColor3 = Color3.new(0, 0, 0)
        cdOverlay.BackgroundTransparency = 0.5
        cdOverlay.Visible = false
        cdOverlay.ZIndex = 2
        cdOverlay.Parent = slot
        
        -- Cooldown text
        local cdText = Instance.new("TextLabel")
        cdText.Name = "CooldownText"
        cdText.Size = UDim2.new(1, 0, 1, 0)
        cdText.BackgroundTransparency = 1
        cdText.Text = ""
        cdText.Font = Enum.Font.GothamBold
        cdText.TextSize = 18
        cdText.TextColor3 = Color3.new(1, 1, 1)
        cdText.ZIndex = 3
        cdText.Parent = slot
        
        -- Key binding label
        local keyLabel = Instance.new("TextLabel")
        keyLabel.Name = "KeyLabel"
        keyLabel.Size = UDim2.new(0, 18, 0, 18)
        keyLabel.Position = UDim2.new(0, 2, 0, 2)
        keyLabel.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
        keyLabel.BackgroundTransparency = 0.5
        keyLabel.Text = i == 1 and "Q" or i == 2 and "E" or i == 3 and "R" or "F"
        keyLabel.Font = Enum.Font.Gotham
        keyLabel.TextSize = 10
        keyLabel.TextColor3 = Color3.new(1, 1, 1)
        keyLabel.ZIndex = 4
        keyLabel.Parent = slot
        
        Instance.new("UICorner", keyLabel).CornerRadius = UDim.new(0, 3)
        
        -- MP Cost label
        local mpLabel = Instance.new("TextLabel")
        mpLabel.Name = "MPLabel"
        mpLabel.Size = UDim2.new(1, 0, 0, 14)
        mpLabel.Position = UDim2.new(0, 0, 1, -14)
        mpLabel.BackgroundTransparency = 1
        mpLabel.Text = ""
        mpLabel.Font = Enum.Font.Gotham
        mpLabel.TextSize = 10
        mpLabel.TextColor3 = Color3.fromRGB(100, 150, 255)
        mpLabel.ZIndex = 4
        mpLabel.Parent = slot
        
        -- ตั้งค่า icon ตามสกิลที่ติด
        local skillId = equippedSkills[i]
        if skillId then
            local skillInfo = SkillData.getSkill(skillId)
            if skillInfo then
                icon.Image = skillInfo.icon or ""
                mpLabel.Text = skillInfo.mpCost .. " MP"
            end
        end
    end
    
    return gui
end

local skillBarGui = createSkillBar()

-- ===== Update Cooldown Display =====
local function updateCooldownDisplay(skillId, duration)
    skillCooldowns[skillId] = {
        startTime = os.clock(),
        duration = duration
    }
end

-- ===== Use Skill =====
local function useSkill(slotIndex)
    local skillId = equippedSkills[slotIndex]
    if not skillId then return end
    
    local skillInfo = SkillData.getSkill(skillId)
    if not skillInfo then return end
    
    -- ตรวจสอบ cooldown ฝั่ง client
    local cdData = skillCooldowns[skillId]
    if cdData then
        local elapsed = os.clock() - cdData.startTime
        if elapsed < cdData.duration then return end
    end
    
    -- หา target/position
    local char = player.Character
    if not char then return end
    
    local targetInfo = {}
    
    if skillInfo.targetType == "Direction" or skillInfo.targetType == "AOE" then
        -- ใช้ aim direction
        local screenCenter = Vector2.new(camera.ViewportSize.X/2, camera.ViewportSize.Y/2)
        local ray = camera:ScreenPointToRay(screenCenter.X, screenCenter.Y)
        
        local params = RaycastParams.new()
        params.FilterDescendantsInstances = {char}
        local result = workspace:Raycast(ray.Origin, ray.Direction * 100, params)
        
        targetInfo.position = result and result.Position or (ray.Origin + ray.Direction * 50)
        targetInfo.direction = ray.Direction
        
    elseif skillInfo.targetType == "Single" then
        -- หา nearest enemy
        local root = char:FindFirstChild("HumanoidRootPart")
        if root then
            local nearest = nil
            local nearestDist = skillInfo.range
            
            for _, obj in ipairs(workspace:GetChildren()) do
                if obj ~= char and obj:IsA("Model") then
                    local hum = obj:FindFirstChild("Humanoid")
                    local objRoot = obj:FindFirstChild("HumanoidRootPart")
                    if hum and hum.Health > 0 and objRoot then
                        local dist = (root.Position - objRoot.Position).Magnitude
                        if dist < nearestDist then
                            nearest = obj
                            nearestDist = dist
                        end
                    end
                end
            end
            
            if nearest then
                targetInfo.targetName = nearest.Name
                targetInfo.position = nearest.HumanoidRootPart.Position
            end
        end
    end
    
    -- ส่งไปยัง Server
    useSkillEvent:FireServer(skillId, targetInfo)
    
    -- Optimistic CD update (รอ feedback จาก server)
    updateCooldownDisplay(skillId, skillInfo.cooldown)
    
    print("[SkillController] Using skill: " .. skillId)
end

-- ===== Feedback Handler =====
skillFeedbackEvent.OnClientEvent:Connect(function(data)
    if data.type == "CooldownStart" then
        updateCooldownDisplay(data.skillId, data.duration)
        
    elseif data.type == "CastStart" then
        -- แสดง cast bar
        showCastBar(data.duration)
        
    elseif data.type == "InsufficientMP" then
        -- แสดง notification
        showNotification("MP ไม่เพียงพอ! ต้องการ " .. data.required .. " MP", Color3.fromRGB(100, 150, 255))
        
    elseif data.type == "OnCooldown" then
        showNotification(string.format("อีก %.1f วินาที", data.remaining), Color3.fromRGB(200, 200, 200))
        
    elseif data.type == "SkillUnlocked" then
        showNotification("ปลดล็อคสกิล: " .. (data.skillId or ""), Color3.fromRGB(255, 215, 0))
    end
end)

-- ===== Skill Visual Effects =====
skillResultEvent.OnClientEvent:Connect(function(data)
    local skillInfo = SkillData.getSkill(data.skillId)
    if not skillInfo then return end
    
    -- แสดง effect ตาม skill
    if data.skillId == "fireball" then
        showFireballEffect(data.position, data.direction)
    elseif data.skillId == "ice_nova" then
        showIceNovaEffect(data.position)
    elseif data.skillId == "heal" then
        showHealEffect(data.position)
    elseif data.skillId == "whirlwind" then
        showWhirlwindEffect(data.position)
    end
end)

function showFireballEffect(position, direction)
    if not position then return end
    
    -- Fireball particle effect
    local fireball = Instance.new("Part")
    fireball.Size = Vector3.new(1, 1, 1)
    fireball.Shape = Enum.PartType.Ball
    fireball.Position = position
    fireball.Anchored = true
    fireball.CanCollide = false
    fireball.Material = Enum.Material.Neon
    fireball.BrickColor = BrickColor.new("Bright orange")
    fireball.Parent = workspace
    
    TweenService:Create(fireball, TweenInfo.new(0.5), {
        Size = Vector3.new(5, 5, 5),
        Transparency = 1
    }):Play()
    
    game:GetService("Debris"):AddItem(fireball, 0.5)
end

function showIceNovaEffect(position)
    if not position then return end
    
    -- Ice nova ring effect
    for i = 1, 8 do
        local shard = Instance.new("Part")
        shard.Size = Vector3.new(0.5, 2, 0.5)
        
        local angle = (i / 8) * math.pi * 2
        local offset = Vector3.new(math.cos(angle) * 6, 0, math.sin(angle) * 6)
        shard.Position = position + offset
        shard.Anchored = true
        shard.CanCollide = false
        shard.Material = Enum.Material.Neon
        shard.BrickColor = BrickColor.new("Bright blue")
        shard.Parent = workspace
        
        TweenService:Create(shard, TweenInfo.new(0.8), {
            Position = shard.Position + Vector3.new(0, 3, 0),
            Transparency = 1
        }):Play()
        
        game:GetService("Debris"):AddItem(shard, 0.8)
    end
end

function showHealEffect(position)
    if not position then return end
    
    -- Heal sparkles
    for i = 1, 5 do
        local sparkle = Instance.new("Part")
        sparkle.Size = Vector3.new(0.3, 0.3, 0.3)
        sparkle.Shape = Enum.PartType.Ball
        sparkle.Position = position + Vector3.new(
            math.random(-2, 2),
            math.random(0, 3),
            math.random(-2, 2)
        )
        sparkle.Anchored = true
        sparkle.CanCollide = false
        sparkle.Material = Enum.Material.Neon
        sparkle.BrickColor = BrickColor.new("Bright green")
        sparkle.Parent = workspace
        
        TweenService:Create(sparkle, TweenInfo.new(1), {
            Position = sparkle.Position + Vector3.new(0, 3, 0),
            Transparency = 1
        }):Play()
        
        game:GetService("Debris"):AddItem(sparkle, 1)
    end
end

function showWhirlwindEffect(position)
    if not position then return end
    -- Whirlwind spinning particles
    task.spawn(function()
        for frame = 1, 20 do
            local part = Instance.new("Part")
            local angle = (frame / 20) * math.pi * 4
            part.Size = Vector3.new(0.5, 0.5, 0.5)
            part.Position = position + Vector3.new(math.cos(angle) * 4, 1, math.sin(angle) * 4)
            part.Anchored = true
            part.CanCollide = false
            part.Material = Enum.Material.Neon
            part.BrickColor = BrickColor.new("Bright yellow")
            part.Parent = workspace
            
            TweenService:Create(part, TweenInfo.new(0.3), {Transparency = 1}):Play()
            game:GetService("Debris"):AddItem(part, 0.3)
            
            task.wait(0.05)
        end
    end)
end

-- ===== Cast Bar UI =====
function showCastBar(duration)
    local gui = player.PlayerGui:FindFirstChild("SkillBar")
    if not gui then return end
    
    local existing = gui:FindFirstChild("CastBar")
    if existing then existing:Destroy() end
    
    local castFrame = Instance.new("Frame")
    castFrame.Name = "CastBar"
    castFrame.Size = UDim2.new(0, 200, 0, 20)
    castFrame.Position = UDim2.new(0.5, -100, 0.5, 30)
    castFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    castFrame.BorderSizePixel = 0
    castFrame.Parent = gui
    
    Instance.new("UICorner", castFrame).CornerRadius = UDim.new(0, 10)
    
    local fill = Instance.new("Frame")
    fill.Name = "Fill"
    fill.Size = UDim2.new(0, 0, 1, 0)
    fill.BackgroundColor3 = Color3.fromRGB(255, 200, 0)
    fill.BorderSizePixel = 0
    fill.Parent = castFrame
    
    Instance.new("UICorner", fill).CornerRadius = UDim.new(0, 10)
    
    TweenService:Create(fill, TweenInfo.new(duration, Enum.EasingStyle.Linear), {
        Size = UDim2.new(1, 0, 1, 0)
    }):Play()
    
    game:GetService("Debris"):AddItem(castFrame, duration + 0.1)
end

-- ===== Notification =====
function showNotification(message, color)
    local gui = player.PlayerGui:FindFirstChild("SkillBar")
    if not gui then return end
    
    local notif = Instance.new("TextLabel")
    notif.Size = UDim2.new(0, 200, 0, 30)
    notif.Position = UDim2.new(0.5, -100, 0.5, -60)
    notif.BackgroundTransparency = 1
    notif.Text = message
    notif.Font = Enum.Font.GothamBold
    notif.TextSize = 16
    notif.TextColor3 = color or Color3.new(1, 1, 1)
    notif.TextStrokeTransparency = 0
    notif.ZIndex = 10
    notif.Parent = gui
    
    TweenService:Create(notif, TweenInfo.new(1.5), {
        Position = UDim2.new(0.5, -100, 0.5, -100),
        TextTransparency = 1,
        TextStrokeTransparency = 1
    }):Play()
    
    game:GetService("Debris"):AddItem(notif, 1.5)
end

-- ===== Cooldown Update Loop =====
game:GetService("RunService").RenderStepped:Connect(function()
    local skillBarFrame = skillBarGui and skillBarGui:FindFirstChild("SkillBarFrame")
    if not skillBarFrame then return end
    
    for i = 1, 4 do
        local skillId = equippedSkills[i]
        local slot = skillBarFrame:FindFirstChild("Slot" .. i)
        if not slot then continue end
        
        local overlay = slot:FindFirstChild("CooldownOverlay")
        local cdText = slot:FindFirstChild("CooldownText")
        
        if skillId and skillCooldowns[skillId] then
            local cdData = skillCooldowns[skillId]
            local elapsed = os.clock() - cdData.startTime
            local remaining = cdData.duration - elapsed
            
            if remaining > 0 then
                if overlay then overlay.Visible = true end
                if cdText then cdText.Text = string.format("%.1f", remaining) end
            else
                if overlay then overlay.Visible = false end
                if cdText then cdText.Text = "" end
                skillCooldowns[skillId] = nil
            end
        else
            if overlay then overlay.Visible = false end
            if cdText then cdText.Text = "" end
        end
    end
end)

-- ===== Input =====
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    local slotIndex = SKILL_HOTKEYS[input.KeyCode]
    if slotIndex then
        useSkill(slotIndex)
    end
end)
```

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Skill Combo System
สร้างระบบที่การใช้สกิลบางคู่ติดกันจะ trigger "Combo Effect" พิเศษ เช่น Fireball + Ice Nova = Steam Explosion

### แบบฝึกหัดที่ 2: Skill Synergy
สร้างระบบ Synergy ที่ party ที่มีทั้ง Warrior + Mage + Healer ได้รับ bonus stats

### แบบฝึกหัดที่ 3: Conditional Skills
สร้างสกิลที่ทำงานต่างกันตามสถานการณ์ เช่น "Vengeance" ทำ damage ตาม HP ที่ขาดหาย

## สรุป

ระบบสกิลที่สมบูรณ์ประกอบด้วย:
- **SkillData** - ฐานข้อมูลสกิลทุกประเภท
- **SkillManager** - execute logic และ apply effects บน server
- **Cooldown System** - จัดการ cooldown ทั้ง client และ server
- **MP System** - ตรวจสอบและใช้ MP
- **Channel System** - สกิลที่ต้อง channel ต่อเนื่อง
- **Visual Effects** - แสดงผลสวยงามฝั่ง client
- **Skill Tree** - ระบบ unlock สกิลตามลำดับ
