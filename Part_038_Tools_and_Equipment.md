# Part 38: Tools and Equipment (เครื่องมือและอุปกรณ์)

## บทนำ

`Tool` ใน Roblox คือ Instance ที่ผู้เล่นสามารถถือในมือได้ เช่น ดาบ, ปืน, ไม้พาย หรืออุปกรณ์ต่างๆ เมื่อ activate Tool จะเรียก Script ที่อยู่ภายใน เป็นระบบพื้นฐานสำหรับ gameplay ทุกรูปแบบ

---

## 38.1 โครงสร้าง Tool

```
Tool
├── Handle (BasePart - จำเป็น)
│   ├── SpecialMesh (ถ้าต้องการรูปทรงพิเศษ)
│   ├── Sound (เสียงเมื่อถือ)
│   └── ParticleEmitter (เอฟเฟกต์)
├── Script (Server-side logic)
├── LocalScript (Client-side logic)
└── RemoteEvent/Function (การสื่อสาร)
```

### Properties ของ Tool

```lua
-- Properties หลักของ Tool
local tool = Instance.new("Tool")

tool.Name = "Sword"                         -- ชื่อ Tool
tool.RequiresHandle = true                  -- ต้องมี Handle
tool.CanBeDropped = true                    -- วางได้หรือไม่
tool.Enabled = true                         -- เปิดใช้งาน
tool.GripForward = Vector3.new(0, 0, -1)    -- ทิศทางการถือ
tool.GripPos = Vector3.new(0, 0, 0)        -- ตำแหน่งในมือ
tool.GripRight = Vector3.new(1, 0, 0)       -- แกนขวา
tool.GripUp = Vector3.new(0, 1, 0)         -- แกนบน
tool.ManualActivationOnly = false           -- ต้องกด Activate เอง

tool.Parent = game:GetService("StarterPack")  -- ใส่ใน StarterPack
```

### Tool Events

```lua
-- Events ของ Tool
tool.Equipped:Connect(function()
    -- เมื่อผู้เล่นหยิบ Tool
    print("Tool equipped!")
end)

tool.Unequipped:Connect(function()
    -- เมื่อผู้เล่นวาง Tool
    print("Tool unequipped!")
end)

tool.Activated:Connect(function()
    -- เมื่อคลิกซ้าย (PC) หรือแตะ (Mobile)
    print("Tool activated!")
end)

tool.Deactivated:Connect(function()
    -- เมื่อปล่อยคลิก
    print("Tool deactivated!")
end)
```

---

## 38.2 สร้าง Tool ด้วย Script

### Tool สร้างในโค้ด

```lua
-- Script ใน ServerScriptService
local function createSwordTool()
    local tool = Instance.new("Tool")
    tool.Name = "IronSword"
    tool.RequiresHandle = true
    tool.CanBeDropped = false
    
    -- Handle (ด้ามจับ)
    local handle = Instance.new("Part")
    handle.Name = "Handle"
    handle.Size = Vector3.new(0.3, 3, 0.3)
    handle.BrickColor = BrickColor.new("Medium stone grey")
    handle.Material = Enum.Material.SmoothPlastic
    handle.Parent = tool
    
    -- ใบดาบ
    local blade = Instance.new("Part")
    blade.Name = "Blade"
    blade.Size = Vector3.new(0.2, 2, 0.1)
    blade.BrickColor = BrickColor.new("Bright blue")
    blade.Material = Enum.Material.SmoothPlastic
    blade.Parent = tool
    
    -- เชื่อม blade กับ handle
    local weld = Instance.new("WeldConstraint")
    weld.Part0 = handle
    weld.Part1 = blade
    weld.Parent = handle
    blade.CFrame = handle.CFrame * CFrame.new(0, 2.5, 0)
    
    -- GripPos ปรับตำแหน่งในมือ
    tool.GripPos = Vector3.new(0, -1, 0)
    tool.GripForward = Vector3.new(0, -1, 0)
    
    return tool
end

-- ให้ผู้เล่นเมื่อ join
local Players = game:GetService("Players")
Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function(character)
        local sword = createSwordTool()
        sword.Parent = player.Backpack
    end)
end)
```

---

## 38.3 ดาบ (Sword Tool)

### Script ฝั่ง Server

```lua
-- Script ใน Tool (SwordTool/Script)
local tool = script.Parent
local handle = tool:WaitForChild("Handle")

local damage = 25
local cooldown = 0.5
local canAttack = true

local Players = game:GetService("Players")

-- เล่น animation เมื่อ activate
local swingAnimation = Instance.new("Animation")
swingAnimation.AnimationId = "rbxassetid://507766388"  -- Sword swing animation
swingAnimation.Parent = tool

-- Hitbox สำหรับตรวจจับการชน
local function createHitbox(character, position)
    -- หา Humanoids ใกล้เคียง
    local hitParts = workspace:GetPartBoundsInBox(
        CFrame.new(position),
        Vector3.new(5, 5, 5)
    )
    
    local hit = {}
    
    for _, part in pairs(hitParts) do
        local model = part:FindFirstAncestorOfClass("Model")
        if model and model ~= character then
            local humanoid = model:FindFirstChildOfClass("Humanoid")
            if humanoid and humanoid.Health > 0 and not hit[model] then
                hit[model] = true
                humanoid:TakeDamage(damage)
                
                -- เอฟเฟกต์ hit
                local hitEffect = Instance.new("PointLight")
                hitEffect.Color = Color3.fromRGB(255, 50, 0)
                hitEffect.Brightness = 3
                hitEffect.Range = 10
                hitEffect.Parent = part
                game:GetService("Debris"):AddItem(hitEffect, 0.2)
            end
        end
    end
end

tool.Activated:Connect(function()
    if not canAttack then return end
    canAttack = false
    
    local character = tool.Parent
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    local hrp = character:FindFirstChild("HumanoidRootPart")
    
    if not humanoid or not hrp then return end
    
    -- เล่น animation
    local animator = humanoid:FindFirstChildOfClass("Animator")
    if animator then
        local track = animator:LoadAnimation(swingAnimation)
        track:Play()
        
        -- ตรวจ hit เมื่อ animation กลางทาง
        task.delay(0.2, function()
            local hitPos = hrp.Position + hrp.CFrame.LookVector * 3
            createHitbox(character, hitPos)
        end)
    end
    
    -- Cooldown
    task.wait(cooldown)
    canAttack = true
end)
```

### LocalScript สำหรับ Effects

```lua
-- LocalScript ใน Tool
local tool = script.Parent
local handle = tool:WaitForChild("Handle")
local Players = game:GetService("Players")
local player = Players.LocalPlayer

-- เสียงดาบ
local swingSound = Instance.new("Sound")
swingSound.SoundId = "rbxassetid://12221976"
swingSound.Volume = 0.5
swingSound.Parent = handle

local hitSound = Instance.new("Sound")
hitSound.SoundId = "rbxassetid://12222030"
hitSound.Volume = 0.7
hitSound.Parent = handle

-- Trail สำหรับดาบ
local trailAttachment0 = Instance.new("Attachment")
trailAttachment0.Position = Vector3.new(0, 1, 0)
trailAttachment0.Parent = handle

local trailAttachment1 = Instance.new("Attachment")
trailAttachment1.Position = Vector3.new(0, -1, 0)
trailAttachment1.Parent = handle

local swordTrail = Instance.new("Trail")
swordTrail.Attachment0 = trailAttachment0
swordTrail.Attachment1 = trailAttachment1
swordTrail.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(100, 150, 255)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(50, 50, 200)),
})
swordTrail.Transparency = NumberSequence.new({
    NumberSequenceKeypoint.new(0, 0),
    NumberSequenceKeypoint.new(1, 1),
})
swordTrail.Lifetime = 0.2
swordTrail.Enabled = false
swordTrail.LightEmission = 0.5
swordTrail.Parent = handle

tool.Activated:Connect(function()
    swingSound:Play()
    
    -- เปิด Trail ชั่วคราว
    swordTrail.Enabled = true
    task.delay(0.4, function()
        swordTrail.Enabled = false
    end)
end)
```

---

## 38.4 ปืน (Gun Tool)

### Raycast Gun

```lua
-- Script ใน GunTool (Server)
local tool = script.Parent
local handle = tool:WaitForChild("Handle")
local remoteEvent = tool:WaitForChild("GunEvent")

local damage = 30
local range = 500
local fireRate = 0.1  -- วินาทีระหว่างยิง
local magazineSize = 30
local reloadTime = 2

local canFire = true
local ammo = magazineSize
local isReloading = false

-- ยิง
remoteEvent.OnServerEvent:Connect(function(player, direction, origin)
    if not canFire or isReloading or ammo <= 0 then return end
    canFire = false
    ammo = ammo - 1
    
    -- สร้าง RaycastParams
    local raycastParams = RaycastParams.new()
    raycastParams.FilterDescendantsInstances = {player.Character}
    raycastParams.FilterType = Enum.RaycastFilterType.Exclude
    
    -- ยิง Ray
    local result = workspace:Raycast(origin, direction * range, raycastParams)
    
    if result then
        local hit = result.Instance
        local model = hit:FindFirstAncestorOfClass("Model")
        
        if model then
            local humanoid = model:FindFirstChildOfClass("Humanoid")
            if humanoid and humanoid.Health > 0 then
                humanoid:TakeDamage(damage)
                
                -- บอก clients ว่าโดน
                remoteEvent:FireAllClients("Hit", result.Position)
            end
        end
        
        -- บอก clients ว่ากระสุนไปชน
        remoteEvent:FireAllClients("Impact", result.Position, result.Normal)
    end
    
    -- บอก client เรื่อง ammo
    remoteEvent:FireClient(player, "AmmoUpdate", ammo)
    
    task.wait(fireRate)
    canFire = true
    
    -- Auto reload
    if ammo <= 0 then
        isReloading = true
        remoteEvent:FireClient(player, "Reload")
        task.wait(reloadTime)
        ammo = magazineSize
        isReloading = false
        remoteEvent:FireClient(player, "ReloadComplete", ammo)
    end
end)
```

```lua
-- LocalScript ใน GunTool (Client)
local tool = script.Parent
local handle = tool:WaitForChild("Handle")
local remoteEvent = tool:WaitForChild("GunEvent")

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera
local mouse = player:GetMouse()

local isFiring = false
local character = nil
local ammoDisplay = nil

-- เสียง
local fireSound = Instance.new("Sound")
fireSound.SoundId = "rbxassetid://132373574"
fireSound.Volume = 0.8
fireSound.Parent = handle

local reloadSound = Instance.new("Sound")
reloadSound.SoundId = "rbxassetid://130781269"
reloadSound.Volume = 0.5
reloadSound.Parent = handle

-- GUI แสดง Ammo
local function createAmmoDisplay()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "AmmoDisplay"
    screenGui.ResetOnSpawn = false
    
    local label = Instance.new("TextLabel")
    label.Name = "AmmoLabel"
    label.Size = UDim2.new(0, 150, 0, 50)
    label.Position = UDim2.new(1, -160, 1, -60)
    label.BackgroundTransparency = 1
    label.TextColor3 = Color3.fromRGB(255, 255, 255)
    label.TextSize = 24
    label.Font = Enum.Font.GothamBold
    label.TextXAlignment = Enum.TextXAlignment.Right
    label.Parent = screenGui
    
    screenGui.Parent = player.PlayerGui
    return label
end

-- เอฟเฟกต์กระสุนชน
local function createImpactEffect(position, normal)
    local attachment = Instance.new("Attachment")
    attachment.WorldPosition = position
    attachment.CFrame = CFrame.new(position, position + normal)
    attachment.Parent = workspace.Terrain
    
    local spark = Instance.new("ParticleEmitter")
    spark.Color = ColorSequence.new(Color3.fromRGB(255, 200, 50))
    spark.Size = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.1),
        NumberSequenceKeypoint.new(1, 0),
    })
    spark.Speed = NumberRange.new(5, 15)
    spark.SpreadAngle = Vector2.new(45, 45)
    spark.Lifetime = NumberRange.new(0.3, 0.6)
    spark.LightEmission = 0.8
    spark.Parent = attachment
    spark:Emit(10)
    
    game:GetService("Debris"):AddItem(attachment, 1)
end

-- ยิง
local function fire()
    character = player.Character
    if not character then return end
    
    local hrp = character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    
    -- คำนวณทิศทาง
    local unitRay = camera:ScreenPointToRay(
        mouse.X, mouse.Y
    )
    local direction = unitRay.Direction
    local origin = camera.CFrame.Position
    
    -- ส่งไป Server
    remoteEvent:FireServer(direction, origin)
    
    -- เอฟเฟกต์ client
    fireSound:Play()
    
    -- Muzzle flash
    local muzzleLight = Instance.new("PointLight")
    muzzleLight.Color = Color3.fromRGB(255, 200, 100)
    muzzleLight.Brightness = 10
    muzzleLight.Range = 20
    muzzleLight.Parent = handle
    game:GetService("Debris"):AddItem(muzzleLight, 0.05)
end

-- รับ events จาก server
remoteEvent.OnClientEvent:Connect(function(eventType, ...)
    local args = {...}
    
    if eventType == "Impact" then
        createImpactEffect(args[1], args[2])
    elseif eventType == "AmmoUpdate" then
        if ammoDisplay then
            ammoDisplay.Text = args[1] .. " / 30"
        end
    elseif eventType == "Reload" then
        reloadSound:Play()
        if ammoDisplay then
            ammoDisplay.Text = "RELOADING..."
        end
    elseif eventType == "ReloadComplete" then
        if ammoDisplay then
            ammoDisplay.Text = args[1] .. " / 30"
        end
    end
end)

-- เมื่อ equip
tool.Equipped:Connect(function()
    ammoDisplay = createAmmoDisplay()
    ammoDisplay.Text = "30 / 30"
    isFiring = false
end)

-- เมื่อ unequip
tool.Unequipped:Connect(function()
    isFiring = false
    if player.PlayerGui:FindFirstChild("AmmoDisplay") then
        player.PlayerGui.AmmoDisplay:Destroy()
    end
end)

-- Click เพื่อยิง
tool.Activated:Connect(function()
    isFiring = true
end)

tool.Deactivated:Connect(function()
    isFiring = false
end)

-- Auto fire loop
RunService.Heartbeat:Connect(function()
    if isFiring and tool.Parent == player.Character then
        fire()
    end
end)

-- Reload ด้วย R
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.R and tool.Parent == player.Character then
        remoteEvent:FireServer("RequestReload")
    end
end)
```

---

## 38.5 ไม้กายสิทธิ์ (Magic Wand)

```lua
-- Script ใน MagicWand (Server)
local tool = script.Parent
local handle = tool:WaitForChild("Handle")
local spellEvent = tool:WaitForChild("SpellEvent")

local spellDamage = 40
local spellSpeed = 60
local cooldown = 1
local canCast = true

spellEvent.OnServerEvent:Connect(function(player, targetPosition)
    if not canCast then return end
    canCast = false
    
    local character = player.Character
    if not character then
        canCast = true
        return
    end
    
    local hrp = character:FindFirstChild("HumanoidRootPart")
    if not hrp then
        canCast = true
        return
    end
    
    -- สร้างกระสุนเวทมนตร์
    local spellBall = Instance.new("Part")
    spellBall.Name = "MagicSpell"
    spellBall.Size = Vector3.new(1, 1, 1)
    spellBall.Shape = Enum.PartType.Ball
    spellBall.BrickColor = BrickColor.new("Bright violet")
    spellBall.Material = Enum.Material.Neon
    spellBall.CanCollide = false
    spellBall.CFrame = CFrame.new(handle.Position)
    spellBall.Parent = workspace
    
    -- แสง
    local light = Instance.new("PointLight")
    light.Color = Color3.fromRGB(150, 0, 255)
    light.Brightness = 5
    light.Range = 20
    light.Parent = spellBall
    
    -- Particle
    local particle = Instance.new("ParticleEmitter")
    particle.Color = ColorSequence.new(Color3.fromRGB(150, 0, 255))
    particle.Size = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.3),
        NumberSequenceKeypoint.new(1, 0),
    })
    particle.Rate = 50
    particle.Speed = NumberRange.new(0, 2)
    particle.SpreadAngle = Vector2.new(180, 180)
    particle.Lifetime = NumberRange.new(0.3, 0.5)
    particle.LightEmission = 1
    particle.Parent = spellBall
    
    -- ทิศทาง
    local direction = (targetPosition - handle.Position).Unit
    
    -- BodyVelocity
    local bv = Instance.new("BodyVelocity")
    bv.Velocity = direction * spellSpeed
    bv.MaxForce = Vector3.new(1e5, 1e5, 1e5)
    bv.Parent = spellBall
    
    -- ตรวจสอบการชน
    local hitDebounce = {}
    spellBall.Touched:Connect(function(hit)
        if hitDebounce[hit] then return end
        
        local model = hit:FindFirstAncestorOfClass("Model")
        if model and model ~= character then
            local humanoid = model:FindFirstChildOfClass("Humanoid")
            if humanoid and humanoid.Health > 0 then
                hitDebounce[hit] = true
                humanoid:TakeDamage(spellDamage)
                
                -- เอฟเฟกต์ระเบิด
                spellEvent:FireAllClients("Explode", spellBall.Position)
                spellBall:Destroy()
            end
        end
        
        -- ชน terrain/parts
        if not model or not model:FindFirstChildOfClass("Humanoid") then
            spellEvent:FireAllClients("Explode", spellBall.Position)
            spellBall:Destroy()
        end
    end)
    
    -- ลบหลัง 5 วินาที
    game:GetService("Debris"):AddItem(spellBall, 5)
    
    task.wait(cooldown)
    canCast = true
end)
```

---

## 38.6 Equipment System

### Inventory System

```lua
-- ModuleScript: EquipmentManager
local EquipmentManager = {}
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- ข้อมูลอาวุธ
local weaponData = {
    Sword = {
        damage = 25,
        attackSpeed = 1.5,
        range = 5,
        tool = ReplicatedStorage.Tools.Sword
    },
    Bow = {
        damage = 35,
        attackSpeed = 1.0,
        range = 100,
        tool = ReplicatedStorage.Tools.Bow
    },
    Staff = {
        damage = 50,
        attackSpeed = 0.8,
        range = 80,
        spellEffect = "Fire",
        tool = ReplicatedStorage.Tools.Staff
    }
}

-- Equipment ปัจจุบันของผู้เล่น
local playerEquipment = {}

-- ให้อาวุธ
function EquipmentManager.giveWeapon(player, weaponName)
    local data = weaponData[weaponName]
    if not data then
        warn("EquipmentManager: '" .. weaponName .. "' not found")
        return false
    end
    
    -- ตรวจสอบว่ามีแล้วหรือเปล่า
    if player.Backpack:FindFirstChild(weaponName) or 
       (player.Character and player.Character:FindFirstChild(weaponName)) then
        return false  -- มีแล้ว
    end
    
    -- Clone tool
    local tool = data.tool:Clone()
    
    -- ตั้งค่า damage จาก data
    local config = tool:FindFirstChild("Config")
    if not config then
        config = Instance.new("Configuration")
        config.Parent = tool
    end
    
    local dmgValue = Instance.new("NumberValue")
    dmgValue.Name = "Damage"
    dmgValue.Value = data.damage
    dmgValue.Parent = config
    
    -- ให้ผู้เล่น
    tool.Parent = player.Backpack
    
    -- บันทึก
    if not playerEquipment[player.UserId] then
        playerEquipment[player.UserId] = {}
    end
    playerEquipment[player.UserId][weaponName] = true
    
    return true
end

-- เอาอาวุธออก
function EquipmentManager.removeWeapon(player, weaponName)
    -- ลบจาก Backpack
    local inBackpack = player.Backpack:FindFirstChild(weaponName)
    if inBackpack then
        inBackpack:Destroy()
    end
    
    -- ลบจาก Character (ถ้ากำลังถือ)
    if player.Character then
        local equipped = player.Character:FindFirstChild(weaponName)
        if equipped then
            equipped.Parent = player.Backpack
            equipped:Destroy()
        end
    end
    
    -- ลบจาก record
    if playerEquipment[player.UserId] then
        playerEquipment[player.UserId][weaponName] = nil
    end
end

-- ตรวจสอบว่ามีอาวุธ
function EquipmentManager.hasWeapon(player, weaponName)
    if playerEquipment[player.UserId] then
        return playerEquipment[player.UserId][weaponName] == true
    end
    return false
end

-- ลบข้อมูลเมื่อ leave
game:GetService("Players").PlayerRemoving:Connect(function(player)
    playerEquipment[player.UserId] = nil
end)

return EquipmentManager
```

---

## 38.7 Tool ที่ใช้สองมือ (Two-Handed Tool)

```lua
-- LocalScript ใน TwoHandedSword
local tool = script.Parent
local handle = tool:WaitForChild("Handle")
local Players = game:GetService("Players")
local player = Players.LocalPlayer

-- Animations
local animations = {
    idle = "rbxassetid://507766388",
    swing = "rbxassetid://522635514",
    block = "rbxassetid://522635514",
}

local animTracks = {}
local character = nil
local humanoid = nil
local animator = nil

-- โหลด animations
local function loadAnimations()
    character = player.Character
    humanoid = character:FindFirstChildOfClass("Humanoid")
    animator = humanoid:FindFirstChildOfClass("Animator")
    
    if not animator then
        animator = Instance.new("Animator")
        animator.Parent = humanoid
    end
    
    for name, id in pairs(animations) do
        local anim = Instance.new("Animation")
        anim.AnimationId = id
        animTracks[name] = animator:LoadAnimation(anim)
    end
end

tool.Equipped:Connect(function()
    loadAnimations()
    if animTracks.idle then
        animTracks.idle:Play()
    end
end)

tool.Unequipped:Connect(function()
    for _, track in pairs(animTracks) do
        track:Stop()
    end
end)

tool.Activated:Connect(function()
    if animTracks.idle then animTracks.idle:Stop() end
    if animTracks.swing then
        animTracks.swing:Play()
        animTracks.swing.Stopped:Wait()
        if animTracks.idle then animTracks.idle:Play() end
    end
end)
```

---

## 38.8 Consumable Items (ของที่ใช้แล้วหมด)

### Health Potion

```lua
-- Script ใน HealthPotion (Server)
local tool = script.Parent
local handle = tool:WaitForChild("Handle")
local potionEvent = tool:WaitForChild("PotionEvent")

local healAmount = 50
local cooldown = 30  -- วินาที
local isOnCooldown = {}

tool.Activated:Connect(function()
    local character = tool.Parent
    if not character then return end
    
    local player = game:GetService("Players"):GetPlayerFromCharacter(character)
    if not player then return end
    
    -- ตรวจ cooldown
    if isOnCooldown[player.UserId] then return end
    
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if not humanoid or humanoid.Health >= humanoid.MaxHealth then return end
    
    -- Heal
    humanoid.Health = math.min(humanoid.MaxHealth, humanoid.Health + healAmount)
    
    -- Cooldown
    isOnCooldown[player.UserId] = true
    potionEvent:FireClient(player, "StartCooldown", cooldown)
    
    task.delay(cooldown, function()
        isOnCooldown[player.UserId] = false
        potionEvent:FireClient(player, "EndCooldown")
    end)
end)

game:GetService("Players").PlayerRemoving:Connect(function(player)
    isOnCooldown[player.UserId] = nil
end)
```

```lua
-- LocalScript ใน HealthPotion (Client)
local tool = script.Parent
local potionEvent = tool:WaitForChild("PotionEvent")
local Players = game:GetService("Players")
local player = Players.LocalPlayer

-- Particle effect เมื่อ heal
local function showHealEffect()
    local character = player.Character
    if not character then return end
    
    local hrp = character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    
    local attachment = Instance.new("Attachment", hrp)
    
    local healParticle = Instance.new("ParticleEmitter")
    healParticle.Color = ColorSequence.new(Color3.fromRGB(100, 255, 100))
    healParticle.Size = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0),
        NumberSequenceKeypoint.new(0.3, 0.5),
        NumberSequenceKeypoint.new(1, 0),
    })
    healParticle.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.5),
        NumberSequenceKeypoint.new(1, 1),
    })
    healParticle.Speed = NumberRange.new(3, 8)
    healParticle.SpreadAngle = Vector2.new(180, 180)
    healParticle.EmissionDirection = Enum.NormalId.Top
    healParticle.Lifetime = NumberRange.new(1, 2)
    healParticle.LightEmission = 0.5
    healParticle.Parent = attachment
    healParticle:Emit(30)
    
    game:GetService("Debris"):AddItem(attachment, 2.5)
end

-- Cooldown UI
local cooldownGui = nil

local function showCooldown(duration)
    -- ลบ GUI เก่า
    if cooldownGui then cooldownGui:Destroy() end
    
    cooldownGui = Instance.new("ScreenGui")
    cooldownGui.Name = "PotionCooldown"
    
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(0, 60, 0, 60)
    frame.Position = UDim2.new(0.5, -30, 1, -80)
    frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    frame.BorderSizePixel = 0
    frame.Parent = cooldownGui
    
    local fill = Instance.new("Frame")
    fill.Name = "Fill"
    fill.Size = UDim2.new(1, 0, 1, 0)
    fill.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
    fill.BorderSizePixel = 0
    fill.Parent = frame
    
    local timer = Instance.new("TextLabel")
    timer.Size = UDim2.new(1, 0, 1, 0)
    timer.BackgroundTransparency = 1
    timer.TextColor3 = Color3.fromRGB(255, 255, 255)
    timer.Font = Enum.Font.GothamBold
    timer.TextSize = 14
    timer.Parent = frame
    
    cooldownGui.Parent = player.PlayerGui
    
    -- อัพเดท timer
    local startTime = tick()
    local connection
    connection = game:GetService("RunService").Heartbeat:Connect(function()
        local elapsed = tick() - startTime
        local remaining = math.max(0, duration - elapsed)
        local progress = remaining / duration
        
        fill.Size = UDim2.new(progress, 0, 1, 0)
        timer.Text = string.format("%.1f", remaining)
        
        if remaining <= 0 then
            connection:Disconnect()
        end
    end)
end

tool.Activated:Connect(function()
    showHealEffect()
end)

potionEvent.OnClientEvent:Connect(function(event, ...)
    local args = {...}
    if event == "StartCooldown" then
        showCooldown(args[1])
    elseif event == "EndCooldown" then
        if cooldownGui then
            cooldownGui:Destroy()
            cooldownGui = nil
        end
    end
end)
```

---

## 38.9 Tool Customization System

```lua
-- ModuleScript: ToolCustomizer
-- ระบบ upgrade อาวุธ

local ToolCustomizer = {}

-- ระดับ upgrade
local upgradeData = {
    Sword = {
        maxLevel = 5,
        levels = {
            [1] = {damage = 25, speed = 1.5, name = "Iron Sword"},
            [2] = {damage = 35, speed = 1.6, name = "Steel Sword"},
            [3] = {damage = 50, speed = 1.7, name = "Silver Sword"},
            [4] = {damage = 70, speed = 1.9, name = "Gold Sword"},
            [5] = {damage = 100, speed = 2.0, name = "Dragon Sword"},
        }
    }
}

-- เก็บระดับของผู้เล่น
local playerLevels = {}  -- [userId][toolName] = level

function ToolCustomizer.getLevel(player, toolName)
    if playerLevels[player.UserId] then
        return playerLevels[player.UserId][toolName] or 1
    end
    return 1
end

function ToolCustomizer.upgrade(player, toolName)
    local data = upgradeData[toolName]
    if not data then return false, "Tool not found" end
    
    local currentLevel = ToolCustomizer.getLevel(player, toolName)
    if currentLevel >= data.maxLevel then
        return false, "Max level reached"
    end
    
    -- เพิ่มระดับ
    if not playerLevels[player.UserId] then
        playerLevels[player.UserId] = {}
    end
    
    local newLevel = currentLevel + 1
    playerLevels[player.UserId][toolName] = newLevel
    
    -- อัพเดท tool ถ้ากำลังถืออยู่
    local character = player.Character
    if character then
        local tool = character:FindFirstChild(toolName) or
                     player.Backpack:FindFirstChild(toolName)
        
        if tool then
            local config = tool:FindFirstChild("Config")
            if config then
                local dmg = config:FindFirstChild("Damage")
                if dmg then
                    dmg.Value = data.levels[newLevel].damage
                end
            end
            
            -- เปลี่ยนชื่อ
            tool.Name = data.levels[newLevel].name
        end
    end
    
    return true, newLevel
end

function ToolCustomizer.getStats(toolName, level)
    local data = upgradeData[toolName]
    if not data or not data.levels[level] then return nil end
    return data.levels[level]
end

return ToolCustomizer
```

---

## 38.10 Special Tools

### Grappling Hook

```lua
-- LocalScript ใน GrapplingHook
local tool = script.Parent
local handle = tool:WaitForChild("Handle")
local hookEvent = tool:WaitForChild("HookEvent")

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera
local mouse = player:GetMouse()

local isHooked = false
local hookBeam = nil
local hookPoint = nil
local ropeConstraint = nil
local hookPart = nil

local function createHook()
    local character = player.Character
    if not character then return end
    
    local hrp = character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    
    -- ตัดเชือกเก่า
    if isHooked then
        detachHook()
        return
    end
    
    -- Raycast ไปยังที่มองอยู่
    local unitRay = camera:ScreenPointToRay(mouse.X, mouse.Y)
    local params = RaycastParams.new()
    params.FilterDescendantsInstances = {character}
    params.FilterType = Enum.RaycastFilterType.Exclude
    
    local result = workspace:Raycast(
        camera.CFrame.Position,
        unitRay.Direction * 200,
        params
    )
    
    if not result then return end
    
    -- สร้าง hook part ที่จุดชน
    hookPart = Instance.new("Part")
    hookPart.Name = "HookAnchor"
    hookPart.Size = Vector3.new(0.3, 0.3, 0.3)
    hookPart.Anchored = true
    hookPart.CanCollide = false
    hookPart.CFrame = CFrame.new(result.Position)
    hookPart.Transparency = 0.5
    hookPart.BrickColor = BrickColor.new("Bright yellow")
    hookPart.Parent = workspace
    
    -- RopeConstraint
    local attachment0 = Instance.new("Attachment", hookPart)
    local attachment1 = Instance.new("Attachment", hrp)
    
    ropeConstraint = Instance.new("RopeConstraint")
    ropeConstraint.Attachment0 = attachment0
    ropeConstraint.Attachment1 = attachment1
    ropeConstraint.Length = (result.Position - hrp.Position).Magnitude
    ropeConstraint.Restitution = 0.1
    ropeConstraint.Visible = true
    ropeConstraint.Color = BrickColor.new("Dark orange")
    ropeConstraint.Thickness = 0.1
    ropeConstraint.Parent = hookPart
    
    isHooked = true
    
    -- Swing!
    local bodyVelocity = Instance.new("BodyVelocity", hrp)
    bodyVelocity.Velocity = Vector3.new(0, 10, 0)  -- กระโดดขึ้นเล็กน้อย
    bodyVelocity.MaxForce = Vector3.new(0, 5000, 0)
    
    task.delay(0.1, function()
        if bodyVelocity and bodyVelocity.Parent then
            bodyVelocity:Destroy()
        end
    end)
end

local function detachHook()
    if hookPart then
        hookPart:Destroy()
        hookPart = nil
    end
    isHooked = false
end

tool.Activated:Connect(createHook)

tool.Unequipped:Connect(function()
    if isHooked then
        detachHook()
    end
end)
```

### Building Tool (Place Parts)

```lua
-- LocalScript ใน BuildTool
local tool = script.Parent
local buildEvent = tool:WaitForChild("BuildEvent")

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera
local mouse = player:GetMouse()

-- ตัวอย่างที่จะวาง
local previewPart = nil
local selectedBlockType = "Block"  -- ชนิด block ที่เลือก

local blockTemplates = {
    Block = {Size = Vector3.new(4, 4, 4), Material = Enum.Material.SmoothPlastic},
    Platform = {Size = Vector3.new(8, 1, 8), Material = Enum.Material.SmoothPlastic},
    Wall = {Size = Vector3.new(0.5, 8, 8), Material = Enum.Material.Concrete},
    Ramp = {Size = Vector3.new(4, 4, 4), Material = Enum.Material.SmoothPlastic},
}

-- สร้าง preview
local function createPreview(blockType)
    if previewPart then previewPart:Destroy() end
    
    local template = blockTemplates[blockType]
    if not template then return end
    
    previewPart = Instance.new("Part")
    previewPart.Size = template.Size
    previewPart.Material = template.Material
    previewPart.BrickColor = BrickColor.new("Bright blue")
    previewPart.Transparency = 0.5
    previewPart.CanCollide = false
    previewPart.Anchored = true
    previewPart.Parent = workspace
end

-- อัพเดทตำแหน่ง preview ตาม mouse
RunService.RenderStepped:Connect(function()
    if not tool.Parent or not previewPart then return end
    
    local character = player.Character
    if not character then return end
    
    local unitRay = camera:ScreenPointToRay(mouse.X, mouse.Y)
    local params = RaycastParams.new()
    params.FilterDescendantsInstances = {character, previewPart}
    params.FilterType = Enum.RaycastFilterType.Exclude
    
    local result = workspace:Raycast(
        camera.CFrame.Position,
        unitRay.Direction * 100,
        params
    )
    
    if result then
        -- Snap ไปที่ grid
        local pos = result.Position + result.Normal * (previewPart.Size.Y / 2)
        local snappedPos = Vector3.new(
            math.round(pos.X / 4) * 4,
            math.round(pos.Y / 4) * 4,
            math.round(pos.Z / 4) * 4
        )
        previewPart.CFrame = CFrame.new(snappedPos)
    end
end)

-- วาง block
tool.Activated:Connect(function()
    if not previewPart then return end
    
    buildEvent:FireServer(
        selectedBlockType,
        previewPart.CFrame,
        previewPart.Size
    )
end)

tool.Equipped:Connect(function()
    createPreview(selectedBlockType)
end)

tool.Unequipped:Connect(function()
    if previewPart then
        previewPart:Destroy()
        previewPart = nil
    end
end)
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Staff ไฟ
สร้าง Fire Staff ที่:
- ยิงลูกไฟออกมา
- ลูกไฟทิ้ง Trail ไว้
- เมื่อชนมีระเบิดไฟ
- มีเอฟเฟกต์ aura รอบไม้

### แบบฝึกหัดที่ 2: Bomb Launcher
สร้าง Bomb Launcher ที่:
- ยิงระเบิดด้วย arc (เส้นโค้ง)
- Preview เส้นทางการยิง (Trajectory Line)
- ระเบิดมีรัศมีทำลาย
- มีจำนวนระเบิดจำกัด

### แบบฝึกหัดที่ 3: Fishing Rod
สร้าง Fishing Rod ที่:
- โยนสายตกปลา
- ปลาขึ้นมาแบบสุ่ม
- UI แสดงสถิติการตกปลา
- ปลาชนิดต่างๆ มีค่าต่างกัน

### แบบฝึกหัดที่ 4: Builder Kit
สร้าง Builder Kit ที่:
- วางบล็อกหลายชนิด
- ลบบล็อกได้
- เปลี่ยนสีบล็อกได้
- บันทึกและโหลดโครงสร้างได้

---

## สรุป

ในบทนี้เราได้เรียนรู้:
- **Tool Structure**: Handle, Events, Properties
- **Sword**: Server/Client split, Animation, Hitbox
- **Gun**: Raycast, Muzzle flash, Ammo system
- **Magic Wand**: Projectile, Spell effects
- **Equipment System**: Inventory, Upgrade system
- **Special Tools**: Grappling Hook, Building Tool
- **Consumables**: Potion ที่มี cooldown UI

---

## อ้างอิง
- [Tool](https://create.roblox.com/docs/reference/engine/classes/Tool)
- [Humanoid](https://create.roblox.com/docs/reference/engine/classes/Humanoid)
- [RaycastParams](https://create.roblox.com/docs/reference/engine/datatypes/RaycastParams)
- [RopeConstraint](https://create.roblox.com/docs/reference/engine/classes/RopeConstraint)
