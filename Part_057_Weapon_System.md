# Part 57: Weapon System - ระบบอาวุธและกระสุน

## บทนำ

ระบบอาวุธเป็นหัวใจสำคัญของเกม Action RPG ทุกประเภท ในบทนี้เราจะสร้างระบบอาวุธที่ครบครัน ตั้งแต่ Melee Weapons (อาวุธประชิดตัว) ไปจนถึง Ranged Weapons (อาวุธระยะไกล) พร้อม Projectile System และ Special Effects

## สถาปัตยกรรมระบบอาวุธ

```
WeaponSystem/
├── WeaponData (Module) - ฐานข้อมูลอาวุธ
├── WeaponManager (Server) - จัดการ logic อาวุธ
├── WeaponController (Client) - รับ input และ animation
├── ProjectileManager (Server) - จัดการกระสุน
└── WeaponEffects (Client) - Visual effects
```

## WeaponData Module - ฐานข้อมูลอาวุธ

```lua
-- ReplicatedStorage/Shared/WeaponData.lua

local WeaponData = {}

-- ===== Weapon Types =====
WeaponData.Types = {
    SWORD = "Sword",
    AXE = "Axe",
    STAFF = "Staff",
    BOW = "Bow",
    GUN = "Gun",
    DAGGER = "Dagger",
    HAMMER = "Hammer",
    SPEAR = "Spear"
}

-- ===== Damage Types =====
WeaponData.DamageTypes = {
    PHYSICAL = "Physical",
    MAGIC = "Magic",
    FIRE = "Fire",
    ICE = "Ice",
    LIGHTNING = "Lightning",
    POISON = "Poison"
}

-- ===== Rarity Levels =====
WeaponData.Rarity = {
    COMMON = {name = "Common", color = Color3.fromRGB(200, 200, 200), multiplier = 1.0},
    UNCOMMON = {name = "Uncommon", color = Color3.fromRGB(30, 200, 30), multiplier = 1.2},
    RARE = {name = "Rare", color = Color3.fromRGB(30, 100, 255), multiplier = 1.5},
    EPIC = {name = "Epic", color = Color3.fromRGB(160, 30, 240), multiplier = 2.0},
    LEGENDARY = {name = "Legendary", color = Color3.fromRGB(255, 165, 0), multiplier = 3.0}
}

-- ===== Weapon Database =====
WeaponData.Weapons = {
    -- ===== SWORDS =====
    ["iron_sword"] = {
        id = "iron_sword",
        name = "Iron Sword",
        namethai = "ดาบเหล็ก",
        type = "Sword",
        damageType = "Physical",
        rarity = "COMMON",
        
        -- Stats
        baseDamage = 25,
        damageRange = {min = 20, max = 30},  -- ความเสียหายสุ่มในช่วงนี้
        attackSpeed = 1.0,       -- attacks per second
        range = 6,               -- melee range
        critChance = 0.05,       -- 5% crit chance
        critMultiplier = 1.5,    -- 1.5x damage on crit
        
        -- Requirements
        requireLevel = 1,
        requireStrength = 0,
        
        -- Combat properties
        knockback = 10,
        comboCount = 3,          -- จำนวน combo hits
        
        -- Animation
        animationId = "rbxassetid://123456789",
        
        -- Model
        modelId = "rbxassetid://987654321",
        
        -- Special properties
        special = nil,
        
        -- Description
        description = "ดาบเหล็กธรรมดา เหมาะสำหรับผู้เริ่มต้น"
    },
    
    ["fire_sword"] = {
        id = "fire_sword",
        name = "Fire Sword",
        namehai = "ดาบไฟ",
        type = "Sword",
        damageType = "Physical",
        rarity = "RARE",
        
        baseDamage = 45,
        damageRange = {min = 38, max = 52},
        attackSpeed = 0.9,
        range = 6,
        critChance = 0.1,
        critMultiplier = 2.0,
        
        requireLevel = 15,
        requireStrength = 20,
        
        knockback = 15,
        comboCount = 3,
        
        -- Special: เพิ่ม fire damage
        special = {
            type = "OnHit",
            effect = "BurnTarget",
            chance = 0.3,    -- 30% chance to burn
            data = {
                damage = 5,
                duration = 3,
                interval = 1
            }
        },
        
        description = "ดาบที่ชุบด้วยไฟ มีโอกาสติดไฟเป้าหมาย"
    },
    
    ["legendary_blade"] = {
        id = "legendary_blade",
        name = "Excalibur",
        nameThai = "เอ็กซ์คาลิเบอร์",
        type = "Sword",
        damageType = "Physical",
        rarity = "LEGENDARY",
        
        baseDamage = 120,
        damageRange = {min = 100, max = 140},
        attackSpeed = 1.2,
        range = 7,
        critChance = 0.25,
        critMultiplier = 3.0,
        
        requireLevel = 50,
        requireStrength = 80,
        
        knockback = 30,
        comboCount = 5,
        
        special = {
            type = "OnCrit",
            effect = "HolyExplosion",
            data = {
                damage = 50,
                radius = 10,
                stunDuration = 1
            }
        },
        
        description = "ดาบในตำนาน มีพลังศักดิ์สิทธิ์ที่ไม่มีใครเทียบได้"
    },
    
    -- ===== BOWS =====
    ["wooden_bow"] = {
        id = "wooden_bow",
        name = "Wooden Bow",
        nameThai = "ธนูไม้",
        type = "Bow",
        damageType = "Physical",
        rarity = "COMMON",
        
        baseDamage = 20,
        damageRange = {min = 15, max = 25},
        attackSpeed = 1.2,
        range = 60,          -- ระยะยิงสูงสุด
        critChance = 0.08,
        critMultiplier = 2.0,
        
        requireLevel = 1,
        
        -- Projectile settings
        projectile = {
            speed = 100,
            gravity = 1,         -- กระทบ gravity หรือเปล่า
            piercing = false,    -- ทะลุผ่านหลายเป้าหมาย
            bounces = 0,
            maxRange = 80,
            hitRadius = 1,       -- radius ของ hitbox กระสุน
            modelId = "rbxassetid://arrow_model"
        },
        
        -- Ammo system
        ammo = {
            type = "Arrow",
            infinite = true      -- ถ้า false ต้องมีลูกศรใน inventory
        },
        
        description = "ธนูไม้ธรรมดา เหมาะสำหรับผู้เริ่มต้น"
    },
    
    ["magic_bow"] = {
        id = "magic_bow",
        name = "Magic Bow",
        nameThai = "ธนูเวทย์",
        type = "Bow",
        damageType = "Magic",
        rarity = "EPIC",
        
        baseDamage = 70,
        damageRange = {min = 60, max = 80},
        attackSpeed = 1.0,
        range = 80,
        critChance = 0.15,
        critMultiplier = 2.5,
        
        requireLevel = 30,
        
        projectile = {
            speed = 150,
            gravity = 0,         -- ลูกศรเวทย์ไม่ตกลงพื้น
            piercing = true,     -- ทะลุผ่านได้ 3 เป้าหมาย
            piercingCount = 3,
            bounces = 0,
            maxRange = 100,
            hitRadius = 1.5,
            modelId = "rbxassetid://magic_arrow_model",
            trailEffect = "MagicTrail"
        },
        
        ammo = {
            type = "MagicArrow",
            infinite = true
        },
        
        special = {
            type = "OnHit",
            effect = "MagicExplosion",
            chance = 0.2,
            data = {damage = 30, radius = 5}
        },
        
        description = "ธนูเวทย์ที่ยิงลูกศรวิเศษ ทะลุผ่านศัตรูได้"
    },
    
    -- ===== STAFFS =====
    ["fire_staff"] = {
        id = "fire_staff",
        name = "Fire Staff",
        nameThai = "ไม้เท้าไฟ",
        type = "Staff",
        damageType = "Fire",
        rarity = "RARE",
        
        baseDamage = 55,
        damageRange = {min = 45, max = 65},
        attackSpeed = 0.7,
        range = 50,
        critChance = 0.12,
        critMultiplier = 2.0,
        
        requireLevel = 20,
        requireIntelligence = 30,
        
        projectile = {
            speed = 80,
            gravity = 0,
            piercing = false,
            bounces = 0,
            maxRange = 60,
            hitRadius = 2,
            modelId = "rbxassetid://fireball_model",
            trailEffect = "FireTrail",
            explodeOnHit = true,
            explosionRadius = 8
        },
        
        special = {
            type = "OnHit",
            effect = "BurnArea",
            data = {
                damage = 8,
                duration = 4,
                interval = 1,
                radius = 5
            }
        },
        
        description = "ไม้เท้าที่ยิงลูกไฟ ระเบิดเมื่อกระทบ"
    },
    
    -- ===== DAGGERS =====
    ["poison_dagger"] = {
        id = "poison_dagger",
        name = "Poison Dagger",
        nameThai = "มีดพิษ",
        type = "Dagger",
        damageType = "Physical",
        rarity = "UNCOMMON",
        
        baseDamage = 18,
        damageRange = {min = 14, max = 22},
        attackSpeed = 2.0,       -- โจมตีเร็ว
        range = 4,
        critChance = 0.2,        -- Crit สูง
        critMultiplier = 2.5,
        
        requireLevel = 8,
        
        knockback = 5,
        comboCount = 4,
        
        special = {
            type = "OnHit",
            effect = "PoisonTarget",
            chance = 0.4,
            data = {
                damage = 3,
                duration = 6,
                interval = 1
            }
        },
        
        description = "มีดเร็วที่มีพิษ โจมตีได้บ่อยครั้งมาก"
    }
}

-- ===== Helper Functions =====

function WeaponData.getWeapon(weaponId)
    return WeaponData.Weapons[weaponId]
end

function WeaponData.isRanged(weaponData)
    return weaponData.type == "Bow" or weaponData.type == "Gun" or weaponData.type == "Staff"
end

function WeaponData.isMelee(weaponData)
    return not WeaponData.isRanged(weaponData)
end

function WeaponData.calculateDamage(weaponData, level, strength)
    local base = weaponData.baseDamage
    local min = weaponData.damageRange.min
    local max = weaponData.damageRange.max
    
    -- สุ่ม damage ในช่วง min-max
    local damage = math.random(min, max)
    
    -- Bonus จาก level
    local levelBonus = level * 0.5
    
    -- Bonus จาก strength (สำหรับ physical)
    local statBonus = strength * 0.3
    
    -- Rarity multiplier
    local rarity = WeaponData.Rarity[weaponData.rarity]
    local rarityMult = rarity and rarity.multiplier or 1.0
    
    return math.floor((damage + levelBonus + statBonus) * rarityMult)
end

function WeaponData.isCritical(weaponData)
    return math.random() < weaponData.critChance
end

return WeaponData
```

## WeaponManager - Server Script

```lua
-- ServerScriptService/WeaponManager.lua

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")

local WeaponData = require(ReplicatedStorage.Shared.WeaponData)

-- ===== Events =====
local remotes = ReplicatedStorage:WaitForChild("Remotes")
local equipWeaponEvent = remotes:WaitForChild("EquipWeapon")
local unequipWeaponEvent = remotes:WaitForChild("UnequipWeapon")
local meleeAttackEvent = remotes:WaitForChild("MeleeAttack")
local shootProjectileEvent = remotes:WaitForChild("ShootProjectile")
local hitConfirmEvent = remotes:WaitForChild("HitConfirm")
local weaponFeedbackEvent = remotes:WaitForChild("WeaponFeedback")

-- ===== Player Data =====
local playerWeapons = {}  -- [player] = {equippedWeapon, cooldowns}

-- ===== Setup =====
local function setupPlayer(player)
    playerWeapons[player] = {
        equippedWeapon = nil,
        lastAttackTime = 0,
        comboCount = 0,
        lastComboTime = 0,
        isAttacking = false
    }
end

local function cleanupPlayer(player)
    playerWeapons[player] = nil
end

-- ===== Weapon Equip =====
local function equipWeapon(player, weaponId)
    local data = playerWeapons[player]
    if not data then return false end
    
    local weaponInfo = WeaponData.getWeapon(weaponId)
    if not weaponInfo then
        warn("Weapon not found: " .. weaponId)
        return false
    end
    
    -- ตรวจสอบ requirements
    local char = player.Character
    if not char then return false end
    
    local hum = char:FindFirstChild("Humanoid")
    if not hum or hum.Health <= 0 then return false end
    
    -- ตรวจสอบ level requirement
    local stats = player:FindFirstChild("Stats")
    if stats then
        local level = stats:FindFirstChild("Level")
        if level and level.Value < weaponInfo.requireLevel then
            -- ไม่ถึง level ที่ต้องการ
            return false, "LEVEL_REQUIREMENT"
        end
    end
    
    -- เก็บข้อมูลอาวุธ
    data.equippedWeapon = weaponId
    data.lastAttackTime = 0
    data.comboCount = 0
    
    print(player.Name .. " equipped " .. weaponInfo.name)
    return true
end

-- ===== Melee Attack Processing =====
local function processMeleeAttack(player, hitTargets, attackData)
    local data = playerWeapons[player]
    if not data or not data.equippedWeapon then return end
    
    local weaponInfo = WeaponData.getWeapon(data.equippedWeapon)
    if not weaponInfo then return end
    
    -- ตรวจสอบ attack cooldown
    local now = os.clock()
    local cooldown = 1 / weaponInfo.attackSpeed
    
    if now - data.lastAttackTime < cooldown then
        return  -- ยังไม่หายคูลดาวน์
    end
    
    -- ตรวจสอบ combo
    if now - data.lastComboTime > 2 then
        data.comboCount = 0  -- รีเซ็ต combo ถ้านานเกินไป
    end
    
    data.lastAttackTime = now
    data.lastComboTime = now
    data.comboCount = (data.comboCount % weaponInfo.comboCount) + 1
    
    local char = player.Character
    if not char then return end
    
    local rootPart = char:FindFirstChild("HumanoidRootPart")
    if not rootPart then return end
    
    -- Combo damage multiplier
    local comboMultipliers = {1.0, 1.0, 1.5}  -- Combo 3 ทำ 1.5x damage
    local comboMult = comboMultipliers[data.comboCount] or 1.0
    
    -- ดึง stats ของ player
    local stats = player:FindFirstChild("Stats")
    local level = stats and stats:FindFirstChild("Level") and stats.Level.Value or 1
    local strength = stats and stats:FindFirstChild("Strength") and stats.Strength.Value or 0
    
    -- Validate hit targets (Anti-cheat: ตรวจสอบว่า target อยู่ใกล้จริง)
    local validTargets = {}
    for _, targetInfo in ipairs(hitTargets) do
        local targetChar = workspace:FindFirstChild(targetInfo.name)
        if not targetChar then continue end
        
        local targetRoot = targetChar:FindFirstChild("HumanoidRootPart")
        if not targetRoot then continue end
        
        -- ตรวจสอบระยะห่าง
        local dist = (rootPart.Position - targetRoot.Position).Magnitude
        if dist > weaponInfo.range + 3 then  -- +3 tolerance
            continue  -- เป้าหมายไกลเกินไป
        end
        
        -- ตรวจสอบ line of sight (optional)
        local rayResult = workspace:Raycast(
            rootPart.Position,
            targetRoot.Position - rootPart.Position,
            RaycastParams.new()
        )
        
        table.insert(validTargets, targetChar)
    end
    
    -- ลดความเสียหายให้แต่ละเป้าหมาย
    for _, targetChar in ipairs(validTargets) do
        local targetHum = targetChar:FindFirstChild("Humanoid")
        if not targetHum or targetHum.Health <= 0 then continue end
        
        -- คำนวณ damage
        local damage = WeaponData.calculateDamage(weaponInfo, level, strength)
        damage = math.floor(damage * comboMult)
        
        -- ตรวจสอบ Critical Hit
        local isCrit = WeaponData.isCritical(weaponInfo)
        if isCrit then
            damage = math.floor(damage * weaponInfo.critMultiplier)
        end
        
        -- Apply damage
        targetHum:TakeDamage(damage)
        
        -- Special effects
        if weaponInfo.special then
            local special = weaponInfo.special
            if special.type == "OnHit" then
                if math.random() < (special.chance or 1) then
                    applySpecialEffect(targetChar, special.effect, special.data)
                end
            elseif special.type == "OnCrit" and isCrit then
                applySpecialEffect(targetChar, special.effect, special.data)
            end
        end
        
        -- Knockback
        applyKnockback(targetChar, rootPart.Position, weaponInfo.knockback)
        
        -- แจ้ง client แสดง damage number
        hitConfirmEvent:FireClient(player, {
            target = targetChar.Name,
            damage = damage,
            isCrit = isCrit,
            position = targetChar.HumanoidRootPart.Position
        })
        
        print(string.format("[Weapon] %s hit %s for %d damage (crit: %s, combo: %d)",
            player.Name, targetChar.Name, damage, tostring(isCrit), data.comboCount))
    end
end

-- ===== Special Effect Application =====
function applySpecialEffect(targetChar, effectType, data)
    local targetHum = targetChar:FindFirstChild("Humanoid")
    if not targetHum then return end
    
    if effectType == "BurnTarget" then
        -- สร้าง burn effect
        local burnTag = targetChar:FindFirstChild("BurnEffect")
        if burnTag then burnTag:Destroy() end
        
        burnTag = Instance.new("BoolValue")
        burnTag.Name = "BurnEffect"
        burnTag.Parent = targetChar
        
        -- Tick damage
        task.spawn(function()
            local endTime = os.clock() + data.duration
            while os.clock() < endTime and burnTag.Parent do
                task.wait(data.interval)
                if targetHum.Health > 0 then
                    targetHum:TakeDamage(data.damage)
                end
            end
            if burnTag.Parent then
                burnTag:Destroy()
            end
        end)
        
    elseif effectType == "PoisonTarget" then
        local poisonTag = targetChar:FindFirstChild("PoisonEffect")
        if poisonTag then poisonTag:Destroy() end
        
        poisonTag = Instance.new("BoolValue")
        poisonTag.Name = "PoisonEffect"
        poisonTag.Parent = targetChar
        
        task.spawn(function()
            local endTime = os.clock() + data.duration
            while os.clock() < endTime and poisonTag.Parent do
                task.wait(data.interval)
                if targetHum.Health > 0 then
                    targetHum:TakeDamage(data.damage)
                end
            end
            if poisonTag.Parent then
                poisonTag:Destroy()
            end
        end)
        
    elseif effectType == "HolyExplosion" then
        -- AOE damage รอบ target
        local targetRoot = targetChar:FindFirstChild("HumanoidRootPart")
        if not targetRoot then return end
        
        local params = OverlapParams.new()
        local hits = workspace:GetPartBoundsInRadius(targetRoot.Position, data.radius, params)
        
        local hitCharacters = {}
        for _, part in ipairs(hits) do
            local char = part:FindFirstAncestorOfClass("Model")
            if char and not hitCharacters[char] then
                local hum = char:FindFirstChild("Humanoid")
                if hum and hum.Health > 0 then
                    hitCharacters[char] = true
                    hum:TakeDamage(data.damage)
                    
                    -- Stun
                    if data.stunDuration then
                        local stunTag = Instance.new("BoolValue")
                        stunTag.Name = "Stunned"
                        stunTag.Parent = char
                        hum.WalkSpeed = 0
                        
                        task.delay(data.stunDuration, function()
                            if stunTag.Parent then
                                stunTag:Destroy()
                                hum.WalkSpeed = 16
                            end
                        end)
                    end
                end
            end
        end
    end
end

-- ===== Knockback =====
function applyKnockback(targetChar, sourcePosition, force)
    local targetRoot = targetChar:FindFirstChild("HumanoidRootPart")
    if not targetRoot then return end
    
    local direction = (targetRoot.Position - sourcePosition).Unit
    direction = Vector3.new(direction.X, 0.3, direction.Z).Unit
    
    local velocity = Instance.new("LinearVelocity")
    velocity.VectorVelocity = direction * force
    velocity.Attachment0 = targetRoot:FindFirstChild("RootAttachment") or Instance.new("Attachment", targetRoot)
    velocity.MaxForce = math.huge
    velocity.Parent = targetRoot
    
    game:GetService("Debris"):AddItem(velocity, 0.2)
end

-- ===== Projectile System =====
local ProjectileManager = {}
local activeProjectiles = {}

function ProjectileManager.shoot(player, weaponInfo, origin, direction)
    local projectileData = weaponInfo.projectile
    if not projectileData then return end
    
    local stats = player:FindFirstChild("Stats")
    local level = stats and stats:FindFirstChild("Level") and stats.Level.Value or 1
    local intelligence = stats and stats:FindFirstChild("Intelligence") and stats.Intelligence.Value or 0
    
    -- สร้าง projectile object (invisible server-side)
    -- Client จะแสดง visual ของตัวเอง
    local projectile = {
        id = game:GetService("HttpService"):GenerateGUID(false),
        owner = player,
        weaponInfo = weaponInfo,
        origin = origin,
        direction = direction,
        speed = projectileData.speed,
        position = origin,
        startTime = os.clock(),
        maxRange = projectileData.maxRange or weaponInfo.range,
        hitCount = 0,
        maxHits = projectileData.piercing and (projectileData.piercingCount or 999) or 1,
        hitTargets = {},
        
        -- Damage info
        baseDamage = WeaponData.calculateDamage(weaponInfo, level, intelligence),
        isCrit = WeaponData.isCritical(weaponInfo)
    }
    
    activeProjectiles[projectile.id] = projectile
    
    -- แจ้ง ALL clients ให้แสดง visual
    local shootEvent = remotes:WaitForChild("ProjectileShot")
    shootEvent:FireAllClients({
        id = projectile.id,
        shooterId = player.UserId,
        origin = origin,
        direction = direction,
        speed = projectileData.speed,
        weaponId = weaponInfo.id,
        projectileData = projectileData
    })
    
    return projectile.id
end

-- ===== Projectile Update Loop =====
RunService.Heartbeat:Connect(function(dt)
    local now = os.clock()
    local toRemove = {}
    
    for id, proj in pairs(activeProjectiles) do
        -- อัพเดทตำแหน่ง
        local elapsed = now - proj.startTime
        local newPos = proj.origin + proj.direction * (proj.speed * elapsed)
        
        -- Apply gravity
        local projData = proj.weaponInfo.projectile
        if projData and projData.gravity and projData.gravity > 0 then
            local gravityEffect = Vector3.new(0, -projData.gravity * elapsed * elapsed * 20, 0)
            newPos = newPos + gravityEffect
        end
        
        -- ตรวจสอบ range
        local distTraveled = (newPos - proj.origin).Magnitude
        if distTraveled > proj.maxRange then
            table.insert(toRemove, id)
            continue
        end
        
        -- ตรวจสอบการชนพื้น (Raycast)
        local rayResult = workspace:Raycast(
            proj.position,
            newPos - proj.position,
            RaycastParams.new()
        )
        
        if rayResult then
            -- ชนอะไรบางอย่าง
            local hitPart = rayResult.Instance
            local hitChar = hitPart:FindFirstAncestorOfClass("Model")
            
            if hitChar then
                local hitHum = hitChar:FindFirstChild("Humanoid")
                if hitHum and hitHum.Health > 0 and not proj.hitTargets[hitChar] then
                    -- ตรวจสอบว่าไม่ใช่ owner
                    local hitPlayer = Players:GetPlayerFromCharacter(hitChar)
                    if hitPlayer ~= proj.owner then
                        proj.hitTargets[hitChar] = true
                        proj.hitCount = proj.hitCount + 1
                        
                        -- คำนวณ damage
                        local damage = proj.baseDamage
                        if proj.isCrit then
                            damage = math.floor(damage * proj.weaponInfo.critMultiplier)
                        end
                        
                        hitHum:TakeDamage(damage)
                        
                        -- Special effect
                        if proj.weaponInfo.special then
                            local special = proj.weaponInfo.special
                            if special.type == "OnHit" and math.random() < (special.chance or 1) then
                                applySpecialEffect(hitChar, special.effect, special.data)
                            end
                        end
                        
                        -- Explosion on hit
                        if projData.explodeOnHit then
                            createExplosion(newPos, projData.explosionRadius, proj.owner, proj.baseDamage * 0.5)
                        end
                        
                        -- Hit confirm
                        hitConfirmEvent:FireClient(proj.owner, {
                            target = hitChar.Name,
                            damage = damage,
                            isCrit = proj.isCrit,
                            position = rayResult.Position
                        })
                        
                        -- หยุดถ้าไม่ pierce
                        if proj.hitCount >= proj.maxHits then
                            table.insert(toRemove, id)
                        end
                    end
                end
            else
                -- ชนพื้น/กำแพง
                if projData and projData.explodeOnHit then
                    createExplosion(rayResult.Position, projData.explosionRadius, proj.owner, proj.baseDamage)
                end
                table.insert(toRemove, id)
            end
        end
        
        proj.position = newPos
    end
    
    -- ลบ projectile ที่หมดอายุ
    for _, id in ipairs(toRemove) do
        activeProjectiles[id] = nil
        -- แจ้ง clients ให้ลบ visual
        local removeEvent = remotes:WaitForChild("ProjectileRemoved")
        removeEvent:FireAllClients(id)
    end
end)

-- ===== Explosion =====
function createExplosion(position, radius, owner, damage)
    local params = OverlapParams.new()
    local hits = workspace:GetPartBoundsInRadius(position, radius, params)
    
    local hitChars = {}
    for _, part in ipairs(hits) do
        local char = part:FindFirstAncestorOfClass("Model")
        if char and not hitChars[char] then
            local hum = char:FindFirstChild("Humanoid")
            if hum and hum.Health > 0 then
                -- ตรวจสอบว่าไม่ใช่ owner
                local player = Players:GetPlayerFromCharacter(char)
                if player ~= owner then
                    hitChars[char] = true
                    
                    -- damage ลดลงตามระยะ
                    local charRoot = char:FindFirstChild("HumanoidRootPart")
                    if charRoot then
                        local dist = (charRoot.Position - position).Magnitude
                        local falloff = 1 - (dist / radius)
                        local finalDamage = math.floor(damage * falloff)
                        
                        if finalDamage > 0 then
                            hum:TakeDamage(finalDamage)
                        end
                    end
                end
            end
        end
    end
    
    -- Visual explosion effect (แจ้ง clients)
    local explosionEvent = remotes:WaitForChild("ShowExplosion")
    explosionEvent:FireAllClients({
        position = position,
        radius = radius
    })
end

-- ===== Event Handlers =====
equipWeaponEvent.OnServerEvent:Connect(function(player, weaponId)
    equipWeapon(player, weaponId)
end)

meleeAttackEvent.OnServerEvent:Connect(function(player, hitTargets, attackData)
    -- Rate limiting
    local data = playerWeapons[player]
    if not data then return end
    
    if not data.equippedWeapon then return end
    local weaponInfo = WeaponData.getWeapon(data.equippedWeapon)
    if not weaponInfo then return end
    
    local now = os.clock()
    local cooldown = 1 / weaponInfo.attackSpeed
    
    if now - data.lastAttackTime < cooldown * 0.8 then  -- 20% tolerance
        return
    end
    
    processMeleeAttack(player, hitTargets or {}, attackData or {})
end)

shootProjectileEvent.OnServerEvent:Connect(function(player, origin, direction, weaponId)
    -- Validate
    local data = playerWeapons[player]
    if not data then return end
    
    local char = player.Character
    if not char then return end
    
    local rootPart = char:FindFirstChild("HumanoidRootPart")
    if not rootPart then return end
    
    -- ตรวจสอบว่า origin ไม่ห่างจาก rootPart มากเกินไป
    local dist = (origin - rootPart.Position).Magnitude
    if dist > 10 then return end  -- Anti-cheat
    
    -- ตรวจสอบ direction ว่าเป็น unit vector
    if direction.Magnitude < 0.9 or direction.Magnitude > 1.1 then return end
    
    local weaponInfo = WeaponData.getWeapon(data.equippedWeapon)
    if not weaponInfo or not weaponInfo.projectile then return end
    
    -- Cooldown check
    local now = os.clock()
    local cooldown = 1 / weaponInfo.attackSpeed
    if now - data.lastAttackTime < cooldown * 0.8 then return end
    
    data.lastAttackTime = now
    
    ProjectileManager.shoot(player, weaponInfo, origin, direction.Unit)
end)

-- ===== Player Events =====
Players.PlayerAdded:Connect(setupPlayer)
Players.PlayerRemoving:Connect(cleanupPlayer)

for _, player in ipairs(Players:GetPlayers()) do
    setupPlayer(player)
end
```

## WeaponController - Client Script

```lua
-- StarterPlayerScripts/WeaponController.lua

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

-- ===== Remotes =====
local remotes = ReplicatedStorage:WaitForChild("Remotes")
local equipWeaponEvent = remotes:WaitForChild("EquipWeapon")
local meleeAttackEvent = remotes:WaitForChild("MeleeAttack")
local shootProjectileEvent = remotes:WaitForChild("ShootProjectile")
local hitConfirmEvent = remotes:WaitForChild("HitConfirm")
local projectileShotEvent = remotes:WaitForChild("ProjectileShot")
local projectileRemovedEvent = remotes:WaitForChild("ProjectileRemoved")
local showExplosionEvent = remotes:WaitForChild("ShowExplosion")

local WeaponData = require(ReplicatedStorage.Shared.WeaponData)

-- ===== State =====
local equippedWeaponId = nil
local isAttacking = false
local lastClickTime = 0
local attackCooldown = 0

-- ===== Visual Projectiles =====
local visualProjectiles = {}

-- ===== Crosshair / Aim =====
local function getAimDirection()
    -- Raycast จากกล้องไปหน้าจอ center
    local char = player.Character
    if not char then return Vector3.new(0, 0, -1) end
    
    local rootPart = char:FindFirstChild("HumanoidRootPart")
    if not rootPart then return Vector3.new(0, 0, -1) end
    
    local screenCenter = Vector2.new(
        camera.ViewportSize.X / 2,
        camera.ViewportSize.Y / 2
    )
    
    local ray = camera:ScreenPointToRay(screenCenter.X, screenCenter.Y)
    
    -- Raycast ไปข้างหน้า
    local params = RaycastParams.new()
    params.FilterDescendantsInstances = {char}
    params.FilterType = Enum.RaycastFilterType.Exclude
    
    local result = workspace:Raycast(ray.Origin, ray.Direction * 200, params)
    
    local aimPoint
    if result then
        aimPoint = result.Position
    else
        aimPoint = ray.Origin + ray.Direction * 200
    end
    
    -- Direction จาก weapon/hand ไปยัง aimPoint
    local weaponPos = rootPart.Position + Vector3.new(0, 1, 0)
    return (aimPoint - weaponPos).Unit
end

-- ===== Melee Attack =====
local function doMeleeAttack()
    local char = player.Character
    if not char then return end
    
    local rootPart = char:FindFirstChild("HumanoidRootPart")
    if not rootPart then return end
    
    local weaponInfo = WeaponData.getWeapon(equippedWeaponId)
    if not weaponInfo then return end
    
    isAttacking = true
    
    -- ตรวจหา targets ใน range
    local attackPos = rootPart.Position + rootPart.CFrame.LookVector * (weaponInfo.range / 2)
    local attackSize = Vector3.new(weaponInfo.range, 4, weaponInfo.range)
    
    local params = OverlapParams.new()
    params.FilterDescendantsInstances = {char}
    params.FilterType = Enum.RaycastFilterType.Exclude
    
    local hits = workspace:GetPartBoundsInBox(
        CFrame.new(attackPos),
        attackSize,
        params
    )
    
    local hitTargets = {}
    local seenChars = {}
    
    for _, part in ipairs(hits) do
        local hitChar = part:FindFirstAncestorOfClass("Model")
        if hitChar and hitChar ~= char and not seenChars[hitChar] then
            local hitHum = hitChar:FindFirstChild("Humanoid")
            if hitHum and hitHum.Health > 0 then
                seenChars[hitChar] = true
                table.insert(hitTargets, {name = hitChar.Name})
            end
        end
    end
    
    -- ส่งข้อมูลไปยัง Server
    meleeAttackEvent:FireServer(hitTargets, {
        position = rootPart.Position,
        lookVector = rootPart.CFrame.LookVector
    })
    
    -- Play animation
    local animator = char:FindFirstChild("Humanoid") and char.Humanoid:FindFirstChild("Animator")
    if animator and weaponInfo.animationId then
        local anim = Instance.new("Animation")
        anim.AnimationId = weaponInfo.animationId
        local track = animator:LoadAnimation(anim)
        track:Play()
    end
    
    -- Swing effect
    playSwingEffect(rootPart.Position, rootPart.CFrame.LookVector, weaponInfo)
    
    -- Reset
    task.delay(1 / weaponInfo.attackSpeed, function()
        isAttacking = false
    end)
end

-- ===== Ranged Attack =====
local function doRangedAttack()
    local char = player.Character
    if not char then return end
    
    local rootPart = char:FindFirstChild("HumanoidRootPart")
    if not rootPart then return end
    
    local weaponInfo = WeaponData.getWeapon(equippedWeaponId)
    if not weaponInfo then return end
    
    isAttacking = true
    
    local origin = rootPart.Position + Vector3.new(0, 1, 0)
    local direction = getAimDirection()
    
    -- ส่งไปยัง Server
    shootProjectileEvent:FireServer(origin, direction, equippedWeaponId)
    
    -- แสดง visual ทันที (client-side prediction)
    createVisualProjectile({
        id = "local_" .. os.clock(),
        origin = origin,
        direction = direction,
        speed = weaponInfo.projectile.speed,
        weaponId = equippedWeaponId,
        projectileData = weaponInfo.projectile
    })
    
    task.delay(1 / weaponInfo.attackSpeed, function()
        isAttacking = false
    end)
end

-- ===== Visual Effects =====
function playSwingEffect(position, direction, weaponInfo)
    -- สร้าง swing trail effect
    local trail = Instance.new("Part")
    trail.Size = Vector3.new(0.2, 0.2, weaponInfo.range)
    trail.Position = position + direction * (weaponInfo.range / 2)
    trail.CFrame = CFrame.lookAt(position, position + direction)
    trail.Anchored = true
    trail.CanCollide = false
    trail.Material = Enum.Material.Neon
    trail.BrickColor = BrickColor.new("Bright yellow")
    trail.Transparency = 0.3
    trail.Parent = workspace
    
    -- Fade out
    local tween = TweenService:Create(
        trail,
        TweenInfo.new(0.2),
        {Transparency = 1, Size = Vector3.new(0.05, 0.05, weaponInfo.range)}
    )
    tween:Play()
    tween.Completed:Connect(function()
        trail:Destroy()
    end)
end

function createVisualProjectile(data)
    local projInfo = data.projectileData
    
    -- สร้าง visual part
    local proj = Instance.new("Part")
    proj.Name = "Projectile_" .. data.id
    proj.Size = Vector3.new(0.5, 0.5, 2)
    proj.Position = data.origin
    proj.Anchored = true
    proj.CanCollide = false
    proj.Material = Enum.Material.Neon
    
    -- สี ตามประเภทอาวุธ
    local weaponInfo = WeaponData.getWeapon(data.weaponId)
    if weaponInfo then
        if weaponInfo.damageType == "Fire" then
            proj.BrickColor = BrickColor.new("Bright orange")
        elseif weaponInfo.damageType == "Magic" then
            proj.BrickColor = BrickColor.new("Bright violet")
        elseif weaponInfo.damageType == "Ice" then
            proj.BrickColor = BrickColor.new("Bright blue")
        else
            proj.BrickColor = BrickColor.new("Bright yellow")
        end
    end
    
    proj.Parent = workspace
    
    -- Trail
    if projInfo and projInfo.trailEffect then
        local attachment0 = Instance.new("Attachment", proj)
        local attachment1 = Instance.new("Attachment", proj)
        attachment0.Position = Vector3.new(0, 0, -1)
        attachment1.Position = Vector3.new(0, 0, 1)
        
        local trail = Instance.new("Trail")
        trail.Attachment0 = attachment0
        trail.Attachment1 = attachment1
        trail.Lifetime = 0.3
        trail.MinLength = 0.1
        trail.Parent = proj
    end
    
    visualProjectiles[data.id] = {
        part = proj,
        origin = data.origin,
        direction = data.direction,
        speed = data.speed,
        startTime = os.clock(),
        maxRange = (projInfo and projInfo.maxRange) or 100,
        gravity = (projInfo and projInfo.gravity) or 0
    }
end

-- ===== Update Visual Projectiles =====
RunService.RenderStepped:Connect(function(dt)
    local now = os.clock()
    local toRemove = {}
    
    for id, proj in pairs(visualProjectiles) do
        local elapsed = now - proj.startTime
        
        -- อัพเดทตำแหน่ง
        local newPos = proj.origin + proj.direction * (proj.speed * elapsed)
        
        -- Gravity
        if proj.gravity and proj.gravity > 0 then
            local gravDrop = Vector3.new(0, -proj.gravity * elapsed * elapsed * 20, 0)
            newPos = newPos + gravDrop
        end
        
        -- ตรวจสอบ range
        local dist = (newPos - proj.origin).Magnitude
        if dist > proj.maxRange then
            table.insert(toRemove, id)
            continue
        end
        
        -- ตรวจสอบชนพื้น
        local params = RaycastParams.new()
        local result = workspace:Raycast(
            proj.part.Position,
            newPos - proj.part.Position,
            params
        )
        
        if result then
            table.insert(toRemove, id)
            -- แสดง impact effect
            showImpactEffect(result.Position)
        else
            -- อัพเดทตำแหน่ง
            proj.part.CFrame = CFrame.lookAt(newPos, newPos + proj.direction)
        end
    end
    
    for _, id in ipairs(toRemove) do
        if visualProjectiles[id] then
            visualProjectiles[id].part:Destroy()
            visualProjectiles[id] = nil
        end
    end
end)

function showImpactEffect(position)
    -- Impact particle effect
    local impact = Instance.new("Part")
    impact.Size = Vector3.new(1, 1, 1)
    impact.Position = position
    impact.Anchored = true
    impact.CanCollide = false
    impact.Material = Enum.Material.Neon
    impact.BrickColor = BrickColor.new("Bright yellow")
    impact.Shape = Enum.PartType.Ball
    impact.Parent = workspace
    
    local tween = TweenService:Create(
        impact,
        TweenInfo.new(0.3),
        {Size = Vector3.new(3, 3, 3), Transparency = 1}
    )
    tween:Play()
    tween.Completed:Connect(function()
        impact:Destroy()
    end)
end

-- ===== Damage Numbers UI =====
hitConfirmEvent.OnClientEvent:Connect(function(data)
    local char = player.Character
    if not char then return end
    
    -- สร้าง damage number
    local screenGui = player.PlayerGui:FindFirstChild("DamageGui")
    if not screenGui then
        screenGui = Instance.new("ScreenGui")
        screenGui.Name = "DamageGui"
        screenGui.ResetOnSpawn = false
        screenGui.Parent = player.PlayerGui
    end
    
    -- Convert 3D position to screen position
    local worldPos = data.position + Vector3.new(0, 3, 0)
    local screenPos, onScreen = camera:WorldToScreenPoint(worldPos)
    
    if not onScreen then return end
    
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(0, 100, 0, 40)
    label.Position = UDim2.new(0, screenPos.X - 50, 0, screenPos.Y - 20)
    label.BackgroundTransparency = 1
    label.Text = tostring(data.damage)
    label.Font = Enum.Font.GothamBold
    label.TextSize = data.isCrit and 28 or 20
    label.TextColor3 = data.isCrit and Color3.fromRGB(255, 200, 0) or Color3.fromRGB(255, 50, 50)
    label.TextStrokeTransparency = 0
    label.TextStrokeColor3 = Color3.new(0, 0, 0)
    label.ZIndex = 10
    label.Parent = screenGui
    
    if data.isCrit then
        label.Text = "CRIT! " .. data.damage
    end
    
    -- Animate: float up and fade
    local targetPos = UDim2.new(0, screenPos.X - 50, 0, screenPos.Y - 70)
    
    TweenService:Create(label, TweenInfo.new(1, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
        Position = targetPos,
        TextTransparency = 1,
        TextStrokeTransparency = 1
    }):Play()
    
    game:GetService("Debris"):AddItem(label, 1)
end)

-- ===== Server Projectile Events =====
projectileShotEvent.OnClientEvent:Connect(function(data)
    -- ถ้าเป็นของ player ตัวเอง ไม่ต้องสร้างซ้ำ
    if data.shooterId == player.UserId then return end
    
    createVisualProjectile(data)
end)

projectileRemovedEvent.OnClientEvent:Connect(function(id)
    if visualProjectiles[id] then
        visualProjectiles[id].part:Destroy()
        visualProjectiles[id] = nil
    end
end)

showExplosionEvent.OnClientEvent:Connect(function(data)
    -- แสดง explosion effect
    local explosion = Instance.new("Explosion")
    explosion.Position = data.position
    explosion.BlastRadius = data.radius
    explosion.BlastPressure = 0  -- ไม่ต้องการแรงดัน physics
    explosion.DestroyJointRadiusPercent = 0  -- ไม่ทำลายชิ้นส่วน
    explosion.Parent = workspace
end)

-- ===== Input Handling =====
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        -- Left click = attack
        if not equippedWeaponId then return end
        if isAttacking then return end
        
        local weaponInfo = WeaponData.getWeapon(equippedWeaponId)
        if not weaponInfo then return end
        
        local now = os.clock()
        if now - lastClickTime < attackCooldown then return end
        
        lastClickTime = now
        attackCooldown = 1 / weaponInfo.attackSpeed
        
        if WeaponData.isRanged(weaponInfo) then
            doRangedAttack()
        else
            doMeleeAttack()
        end
    end
end)

-- ===== Test Equip (สำหรับทดสอบ) =====
-- จะถูกแทนที่ด้วย Inventory system จริง
task.wait(2)
equippedWeaponId = "iron_sword"
equipWeaponEvent:FireServer("iron_sword")
print("[WeaponController] Equipped iron_sword for testing")
```

## ระบบ Weapon Upgrade

```lua
-- ServerScriptService/WeaponUpgrade.lua

local ReplicatedStorage = game:GetService("ReplicatedStorage")
local WeaponData = require(ReplicatedStorage.Shared.WeaponData)

local WeaponUpgradeSystem = {}

-- ===== Upgrade Materials =====
local UPGRADE_MATERIALS = {
    [1] = {material = "Iron Ore", amount = 3, coins = 100},
    [2] = {material = "Iron Ore", amount = 5, coins = 200},
    [3] = {material = "Steel Bar", amount = 3, coins = 400},
    [4] = {material = "Steel Bar", amount = 5, coins = 600},
    [5] = {material = "Mythril", amount = 2, coins = 1000}
}

-- Max upgrade level ตาม rarity
local MAX_UPGRADE_LEVEL = {
    COMMON = 3,
    UNCOMMON = 5,
    RARE = 7,
    EPIC = 9,
    LEGENDARY = 10
}

-- ===== Upgrade Multipliers =====
-- แต่ละ level เพิ่ม stats ตามนี้
local function getUpgradeMultiplier(currentLevel)
    return 1 + (currentLevel * 0.1)  -- +10% ต่อ level
end

-- ===== Upgrade Weapon =====
function WeaponUpgradeSystem.upgrade(player, weaponInstance)
    -- weaponInstance = object ใน inventory ที่มี {id, upgradeLevel}
    
    local weaponInfo = WeaponData.getWeapon(weaponInstance.id)
    if not weaponInfo then return false, "INVALID_WEAPON" end
    
    local currentLevel = weaponInstance.upgradeLevel or 0
    local maxLevel = MAX_UPGRADE_LEVEL[weaponInfo.rarity] or 5
    
    if currentLevel >= maxLevel then
        return false, "MAX_LEVEL"
    end
    
    -- ตรวจสอบ materials
    local reqMat = UPGRADE_MATERIALS[currentLevel + 1]
    if not reqMat then return false, "NO_REQUIREMENT" end
    
    -- ตรวจสอบ inventory ของ player (simplified)
    -- ในระบบจริง ต้องตรวจสอบ inventory system
    
    -- อัพเกรด
    weaponInstance.upgradeLevel = currentLevel + 1
    
    local newStats = WeaponUpgradeSystem.getUpgradedStats(weaponInfo, weaponInstance.upgradeLevel)
    
    print(string.format("[Upgrade] %s upgraded %s to +%d",
        player.Name, weaponInfo.name, weaponInstance.upgradeLevel))
    
    return true, newStats
end

function WeaponUpgradeSystem.getUpgradedStats(weaponInfo, upgradeLevel)
    if upgradeLevel == 0 then
        return {
            baseDamage = weaponInfo.baseDamage,
            critChance = weaponInfo.critChance,
            attackSpeed = weaponInfo.attackSpeed
        }
    end
    
    local mult = getUpgradeMultiplier(upgradeLevel)
    
    return {
        baseDamage = math.floor(weaponInfo.baseDamage * mult),
        critChance = math.min(weaponInfo.critChance + upgradeLevel * 0.01, 0.5),  -- max 50%
        attackSpeed = weaponInfo.attackSpeed + upgradeLevel * 0.05
    }
end

return WeaponUpgradeSystem
```

## Weapon Visual Setup

```lua
-- Script ใน ServerScriptService/WeaponVisuals.lua

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- ===== Tool-based Weapon System =====
-- วิธีที่ง่ายที่สุดสำหรับ Roblox คือใช้ Tool

local function createWeaponTool(weaponId)
    local WeaponData = require(ReplicatedStorage.Shared.WeaponData)
    local weaponInfo = WeaponData.getWeapon(weaponId)
    
    if not weaponInfo then return nil end
    
    -- สร้าง Tool
    local tool = Instance.new("Tool")
    tool.Name = weaponInfo.name
    tool.ToolTip = weaponInfo.description or weaponInfo.name
    tool.RequiresHandle = true
    tool.CanBeDropped = false
    
    -- Handle (ส่วนที่จับ)
    local handle = Instance.new("Part")
    handle.Name = "Handle"
    handle.Size = Vector3.new(0.5, 0.5, 3)
    handle.BrickColor = BrickColor.new("Medium stone grey")
    handle.Material = Enum.Material.SmoothPlastic
    handle.Parent = tool
    
    -- ตั้งค่า grip
    tool.GripPos = Vector3.new(0, 0, -1)
    tool.GripForward = Vector3.new(0, 0, -1)
    
    -- Tag เก็บ weapon ID
    local weaponTag = Instance.new("StringValue")
    weaponTag.Name = "WeaponId"
    weaponTag.Value = weaponId
    weaponTag.Parent = tool
    
    -- Script ใน Tool
    local localScript = Instance.new("LocalScript")
    localScript.Source = [[
        local tool = script.Parent
        local handle = tool:WaitForChild("Handle")
        local weaponIdTag = tool:WaitForChild("WeaponId")
        
        local equipped = false
        
        tool.Equipped:Connect(function()
            equipped = true
            -- แจ้ง WeaponController
            local event = game.ReplicatedStorage.Remotes.EquipWeapon
            event:FireServer(weaponIdTag.Value)
        end)
        
        tool.Unequipped:Connect(function()
            equipped = false
            local event = game.ReplicatedStorage.Remotes.UnequipWeapon
            event:FireServer()
        end)
        
        tool.Activated:Connect(function()
            if not equipped then return end
            -- WeaponController จะจัดการ
        end)
    ]]
    localScript.Parent = tool
    
    return tool
end

-- ตัวอย่างการให้อาวุธกับผู้เล่น
Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function(char)
        task.wait(1)
        
        -- ให้ iron_sword สำหรับทดสอบ
        local tool = createWeaponTool("iron_sword")
        if tool then
            tool.Parent = player.Backpack
        end
    end)
end)
```

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Dual Wield System
สร้างระบบที่ผู้เล่นสามารถถือ 2 มีดพร้อมกัน โดยแต่ละมือโจมตีสลับกัน

### แบบฝึกหัดที่ 2: Weapon Enchantment
สร้างระบบ Enchantment ที่เพิ่ม elemental damage (fire/ice/lightning) ให้กับอาวุธ

### แบบฝึกหัดที่ 3: Throwing Weapons
สร้างอาวุธขว้าง (throwing knife, shuriken) ที่หลังจากขว้างแล้วสามารถเรียกคืนมือได้

### แบบฝึกหัดที่ 4: Weapon Durability
เพิ่มระบบ Durability ที่อาวุธสึกหรอเมื่อใช้ และต้องซ่อมแซม

## สรุป

ระบบอาวุธที่ครบครันประกอบด้วย:
- **WeaponData** - ฐานข้อมูลอาวุธครบถ้วน
- **Melee System** - ตรวจ hitbox, combo, knockback
- **Projectile System** - กระสุนที่คำนวณ trajectory บน server
- **Special Effects** - burn, poison, explosion
- **Visual Feedback** - swing trails, impact effects, damage numbers
- **Anti-cheat** - validation ระยะห่าง, rate limiting
- **Upgrade System** - เพิ่ม stats ตาม upgrade level
