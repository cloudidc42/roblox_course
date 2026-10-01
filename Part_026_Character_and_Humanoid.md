# ตอนที่ 26: Character และ Humanoid Properties

## บทนำ

ในการพัฒนาเกม Roblox สิ่งที่สำคัญที่สุดคือการทำความเข้าใจระบบ Character และ Humanoid ของผู้เล่น Character คือโมเดล 3D ที่ผู้เล่นควบคุม ส่วน Humanoid คือ object ที่จัดการพฤติกรรมและคุณสมบัติของตัวละคร เช่น ชีวิต ความเร็ว และอื่นๆ

การเข้าใจระบบนี้จะช่วยให้คุณสามารถ:
- ปรับแต่งตัวละครผู้เล่นได้
- จัดการระบบชีวิต (Health) ได้
- ควบคุมการเคลื่อนที่ของตัวละครได้
- สร้างเอฟเฟกต์พิเศษให้กับตัวละครได้

---

## 26.1 โครงสร้างของ Character

เมื่อผู้เล่น Join เกม Roblox จะสร้าง Character โดยอัตโนมัติ Character มีโครงสร้างดังนี้:

```
Character (Model)
├── HumanoidRootPart (BasePart) - ส่วนกลางของตัวละคร
├── Head (BasePart) - หัว
├── UpperTorso (BasePart) - ลำตัวส่วนบน
├── LowerTorso (BasePart) - ลำตัวส่วนล่าง
├── LeftUpperArm (BasePart) - แขนบนซ้าย
├── LeftLowerArm (BasePart) - แขนล่างซ้าย
├── LeftHand (BasePart) - มือซ้าย
├── RightUpperArm (BasePart) - แขนบนขวา
├── RightLowerArm (BasePart) - แขนล่างขวา
├── RightHand (BasePart) - มือขวา
├── LeftUpperLeg (BasePart) - ขาบนซ้าย
├── LeftLowerLeg (BasePart) - ขาล่างซ้าย
├── LeftFoot (BasePart) - เท้าซ้าย
├── RightUpperLeg (BasePart) - ขาบนขวา
├── RightLowerLeg (BasePart) - ขาล่างขวา
├── RightFoot (BasePart) - เท้าขวา
├── Humanoid (Humanoid) - ควบคุมพฤติกรรมตัวละคร
├── Animate (LocalScript) - จัดการ animation
└── HumanoidDescription (HumanoidDescription) - รูปลักษณ์ตัวละคร
```

### การเข้าถึง Character

```lua
-- วิธีที่ 1: ผ่าน Players Service (ฝั่ง Server)
local Players = game:GetService("Players")

Players.PlayerAdded:Connect(function(player)
    -- รอจนกว่า Character จะโหลด
    player.CharacterAdded:Connect(function(character)
        print("Character ของ " .. player.Name .. " โหลดแล้ว!")
        
        -- เข้าถึง Humanoid
        local humanoid = character:WaitForChild("Humanoid")
        print("Health:", humanoid.Health)
        print("MaxHealth:", humanoid.MaxHealth)
        print("WalkSpeed:", humanoid.WalkSpeed)
    end)
end)
```

```lua
-- วิธีที่ 2: ผ่าน LocalScript (ฝั่ง Client)
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

-- รอ Character
local character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()

-- เข้าถึง Humanoid
local humanoid = character:WaitForChild("Humanoid")
print("เราเล่นเป็น:", LocalPlayer.Name)
print("ชีวิตปัจจุบัน:", humanoid.Health)
```

---

## 26.2 Humanoid Properties

Humanoid มี Properties ที่สำคัญมากมาย มาดูทีละตัว:

### 26.2.1 Health และ MaxHealth

```lua
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")

-- อ่านค่าชีวิต
print("ชีวิตปัจจุบัน:", humanoid.Health)      -- ค่าชีวิตตอนนี้ (default: 100)
print("ชีวิตสูงสุด:", humanoid.MaxHealth)     -- ชีวิตสูงสุด (default: 100)

-- ตั้งค่าชีวิตสูงสุด
humanoid.MaxHealth = 200
humanoid.Health = 200

-- เติมชีวิต
humanoid.Health = humanoid.MaxHealth

-- ลดชีวิต
humanoid:TakeDamage(50)  -- ลดชีวิต 50 (ปลอดภัยกว่าการ set ตรง)

-- ตรวจสอบเมื่อชีวิตเปลี่ยน
humanoid.HealthChanged:Connect(function(health)
    print("ชีวิตเปลี่ยนเป็น:", health)
    
    if health <= 0 then
        print("ตัวละครตายแล้ว!")
    elseif health < 30 then
        print("ชีวิตน้อย! ระวัง!")
    end
end)
```

### 26.2.2 WalkSpeed และ JumpHeight

```lua
local humanoid = character:WaitForChild("Humanoid")

-- ความเร็วเดิน
print("ความเร็วปกติ:", humanoid.WalkSpeed)  -- default: 16

-- เปลี่ยนความเร็ว
humanoid.WalkSpeed = 32     -- วิ่งเร็วขึ้น 2 เท่า
humanoid.WalkSpeed = 8      -- เดินช้า

-- ความสูงการกระโดด
print("ความสูงกระโดด:", humanoid.JumpHeight)  -- default: 7.2

-- เปลี่ยนความสูงกระโดด
humanoid.JumpHeight = 15    -- กระโดดสูงขึ้น
humanoid.JumpHeight = 0     -- กระโดดไม่ได้

-- ทำให้กระโดด
humanoid.Jump = true

-- JumpPower (วิธีเก่า)
humanoid.UseJumpPower = true    -- ใช้ JumpPower แทน JumpHeight
humanoid.JumpPower = 50         -- แรงกระโดด (default: 50)
```

### 26.2.3 State ของ Humanoid

```lua
local humanoid = character:WaitForChild("Humanoid")
local HumanoidStateType = Enum.HumanoidStateType

-- ดู State ปัจจุบัน
local currentState = humanoid:GetState()
print("State ปัจจุบัน:", currentState)

-- State ต่างๆ
--[[
Enum.HumanoidStateType.Idle        - หยุดนิ่ง
Enum.HumanoidStateType.Running     - วิ่ง
Enum.HumanoidStateType.Jumping     - กระโดด
Enum.HumanoidStateType.Freefall    - ตกลงมา
Enum.HumanoidStateType.Landed      - ลงจอด
Enum.HumanoidStateType.Climbing    - ปีน
Enum.HumanoidStateType.Swimming    - ว่ายน้ำ
Enum.HumanoidStateType.Dead        - ตาย
Enum.HumanoidStateType.Seated      - นั่ง
]]

-- ตรวจสอบการเปลี่ยน State
humanoid.StateChanged:Connect(function(oldState, newState)
    print("State เปลี่ยนจาก", oldState, "เป็น", newState)
    
    if newState == HumanoidStateType.Jumping then
        print("กระโดดแล้ว!")
    elseif newState == HumanoidStateType.Freefall then
        print("กำลังตก...")
    elseif newState == HumanoidStateType.Landed then
        print("ลงจอดแล้ว!")
    end
end)

-- บังคับเปลี่ยน State
humanoid:ChangeState(HumanoidStateType.Jumping)
```

### 26.2.4 MoveDirection

```lua
local humanoid = character:WaitForChild("Humanoid")
local RunService = game:GetService("RunService")

-- ดูทิศทางการเคลื่อนที่
RunService.Heartbeat:Connect(function()
    local moveDir = humanoid.MoveDirection
    
    if moveDir.Magnitude > 0 then
        -- กำลังเดิน
        print("กำลังเดินไปทาง:", moveDir)
    end
end)

-- สั่งให้เดินไปยังตำแหน่ง
local targetPosition = Vector3.new(10, 0, 10)
humanoid:MoveTo(targetPosition)

-- รอจนกว่าจะถึงจุดหมาย
humanoid.MoveToFinished:Connect(function(reached)
    if reached then
        print("ถึงจุดหมายแล้ว!")
    else
        print("ไปไม่ถึงจุดหมาย (timeout)")
    end
end)
```

---

## 26.3 การจัดการ Character

### 26.3.1 Respawn ตัวละคร

```lua
-- Server Script
local Players = game:GetService("Players")

local function respawnPlayer(player)
    player:LoadCharacter()  -- โหลด Character ใหม่
end

-- ตัวอย่าง: Respawn เมื่อกด Button
local button = workspace.RespawnButton

button.Touched:Connect(function(hit)
    local character = hit.Parent
    local player = Players:GetPlayerFromCharacter(character)
    
    if player then
        respawnPlayer(player)
    end
end)
```

### 26.3.2 เปลี่ยนรูปลักษณ์ Character

```lua
-- Server Script
local Players = game:GetService("Players")

Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function(character)
        local humanoid = character:WaitForChild("Humanoid")
        
        -- เปลี่ยนสีผิว
        local description = humanoid:GetAppliedDescription()
        description.SkinColor = Color3.fromRGB(255, 200, 150)
        humanoid:ApplyDescription(description)
    end)
end)
```

### 26.3.3 การ Anchor Character

```lua
-- ทำให้ตัวละครอยู่กับที่ (ไม่สามารถเคลื่อนที่ได้)
local function freezeCharacter(character)
    local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
    humanoidRootPart.Anchored = true
end

local function unfreezeCharacter(character)
    local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
    humanoidRootPart.Anchored = false
end

-- ตัวอย่างการใช้งาน
local Players = game:GetService("Players")
Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function(character)
        -- Freeze ตอนเริ่ม
        freezeCharacter(character)
        
        -- Unfreeze หลัง 3 วินาที
        wait(3)
        unfreezeCharacter(character)
        print("เกมเริ่มแล้ว!")
    end)
end)
```

---

## 26.4 HumanoidDescription

HumanoidDescription เป็น object ที่ควบคุมรูปลักษณ์ของตัวละคร

```lua
-- Server Script
local Players = game:GetService("Players")

Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function(character)
        local humanoid = character:WaitForChild("Humanoid")
        
        -- สร้าง HumanoidDescription ใหม่
        local description = Instance.new("HumanoidDescription")
        
        -- ตั้งค่า Body Type
        description.BodyTypeScale = 0.5      -- ขนาดร่างกาย (0-1)
        description.HeadScale = 1.0          -- ขนาดหัว
        description.DepthScale = 1.0         -- ความลึก
        description.WidthScale = 1.0         -- ความกว้าง
        description.ProportionScale = 0.0    -- สัดส่วน (0=Classic, 1=Proportions)
        
        -- ตั้งค่าสีผิว
        description.SkinColor = Color3.fromRGB(255, 200, 150)
        
        -- ใส่ Accessories (ต้องใช้ Asset ID จาก Roblox Catalog)
        -- description.Hat1 = 123456789  -- ID ของหมวก
        
        -- Apply description ให้ Humanoid
        humanoid:ApplyDescription(description)
    end)
end)
```

---

## 26.5 การเข้าถึง Parts ของ Character

```lua
-- LocalScript
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()

-- เข้าถึง Parts ต่างๆ
local head = character:WaitForChild("Head")
local upperTorso = character:WaitForChild("UpperTorso")
local lowerTorso = character:WaitForChild("LowerTorso")
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")

-- ดูตำแหน่งของตัวละคร
print("ตำแหน่ง:", humanoidRootPart.Position)

-- ดูว่าตัวละครหันไปทิศไหน
print("ทิศทาง:", humanoidRootPart.CFrame.LookVector)

-- เปลี่ยนสีของ Parts
head.BrickColor = BrickColor.new("Bright red")
upperTorso.BrickColor = BrickColor.new("Bright blue")
```

---

## 26.6 Events ของ Humanoid

```lua
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")

-- เมื่อชีวิตเปลี่ยน
humanoid.HealthChanged:Connect(function(newHealth)
    print("ชีวิตใหม่:", newHealth)
end)

-- เมื่อตาย
humanoid.Died:Connect(function()
    print("ตัวละครตายแล้ว!")
    -- แสดง death screen
end)

-- เมื่อกระโดด
humanoid.Jumping:Connect(function(isActive)
    if isActive then
        print("กระโดดขึ้น!")
    end
end)

-- เมื่อ State เปลี่ยน
humanoid.StateChanged:Connect(function(old, new)
    print("State:", old, "->", new)
end)

-- เมื่อถึงจุดหมาย MoveTo
humanoid.MoveToFinished:Connect(function(reached)
    print("MoveTo เสร็จ, ถึงหรือเปล่า:", reached)
end)

-- เมื่อปีน
humanoid.Climbing:Connect(function(speed)
    print("กำลังปีน, ความเร็ว:", speed)
end)

-- เมื่อสัมผัส
humanoid.Touched:Connect(function(part, limbPart)
    print("แขนขา", limbPart.Name, "สัมผัส", part.Name)
end)
```

---

## 26.7 ระบบ Damage และ Healing

```lua
-- ModuleScript: HealthSystem
local HealthSystem = {}

-- ดึง Humanoid จาก Character
function HealthSystem.getHumanoid(character)
    return character:FindFirstChildOfClass("Humanoid")
end

-- ทำดาเมจ
function HealthSystem.dealDamage(character, damage)
    local humanoid = HealthSystem.getHumanoid(character)
    if humanoid and humanoid.Health > 0 then
        humanoid:TakeDamage(damage)
        return true
    end
    return false
end

-- รักษาชีวิต
function HealthSystem.heal(character, amount)
    local humanoid = HealthSystem.getHumanoid(character)
    if humanoid and humanoid.Health > 0 then
        humanoid.Health = math.min(
            humanoid.Health + amount,
            humanoid.MaxHealth
        )
        return true
    end
    return false
end

-- เติมชีวิตเต็ม
function HealthSystem.fullHeal(character)
    local humanoid = HealthSystem.getHumanoid(character)
    if humanoid then
        humanoid.Health = humanoid.MaxHealth
        return true
    end
    return false
end

-- ตรวจสอบว่าตายหรือไม่
function HealthSystem.isDead(character)
    local humanoid = HealthSystem.getHumanoid(character)
    if humanoid then
        return humanoid.Health <= 0
    end
    return true
end

-- เปลี่ยนชีวิตสูงสุด
function HealthSystem.setMaxHealth(character, maxHealth)
    local humanoid = HealthSystem.getHumanoid(character)
    if humanoid then
        humanoid.MaxHealth = maxHealth
        humanoid.Health = maxHealth  -- เติมชีวิตเต็มด้วย
    end
end

return HealthSystem
```

### การใช้งาน HealthSystem

```lua
-- Server Script
local Players = game:GetService("Players")
local HealthSystem = require(game.ServerScriptService.HealthSystem)

Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function(character)
        -- เพิ่มชีวิตสูงสุดให้ผู้เล่น
        HealthSystem.setMaxHealth(character, 200)
        
        -- รอ 5 วินาทีแล้วทำดาเมจ
        wait(5)
        HealthSystem.dealDamage(character, 50)
        print("ชีวิตหลังถูกโจมตี:", character.Humanoid.Health)
        
        -- รักษา
        wait(2)
        HealthSystem.heal(character, 30)
        print("ชีวิตหลังรักษา:", character.Humanoid.Health)
    end)
end)
```

---

## 26.8 Humanoid Animations

```lua
-- LocalScript: จัดการ Animations
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local animator = humanoid:WaitForChild("Animator")

-- โหลด Animation
local function loadAnimation(animationId)
    local animation = Instance.new("Animation")
    animation.AnimationId = "rbxassetid://" .. animationId
    return animator:LoadAnimation(animation)
end

-- เล่น Animation
local myAnimation = loadAnimation(123456789)  -- ใส่ Animation ID ของคุณ

-- เล่น animation
myAnimation:Play()

-- หยุด animation
myAnimation:Stop()

-- Fade In/Out
myAnimation:Play(0.1)   -- Fade in ใน 0.1 วินาที
myAnimation:Stop(0.5)   -- Fade out ใน 0.5 วินาที

-- ปรับความเร็ว animation
myAnimation:AdjustSpeed(2)  -- เล่นเร็วขึ้น 2 เท่า
myAnimation:AdjustSpeed(0.5)  -- เล่นช้าลงครึ่งหนึ่ง

-- ตรวจสอบ animation ที่กำลังเล่น
local playingAnimations = animator:GetPlayingAnimationTracks()
for _, track in ipairs(playingAnimations) do
    print("กำลังเล่น:", track.Animation.AnimationId)
end
```

---

## 26.9 การทำ Ragdoll

```lua
-- Script: RagdollSystem (Server)
local function enableRagdoll(character)
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end
    
    -- Disable Humanoid
    humanoid:ChangeState(Enum.HumanoidStateType.Physics)
    
    -- ทำให้ Joints กลายเป็น BallSocketConstraints
    for _, joint in ipairs(character:GetDescendants()) do
        if joint:IsA("Motor6D") then
            -- เปลี่ยน Motor6D เป็น BallSocketConstraint
            local attachment0 = Instance.new("Attachment")
            local attachment1 = Instance.new("Attachment")
            
            attachment0.CFrame = joint.C0
            attachment1.CFrame = joint.C1
            
            attachment0.Parent = joint.Part0
            attachment1.Parent = joint.Part1
            
            local ballSocket = Instance.new("BallSocketConstraint")
            ballSocket.Attachment0 = attachment0
            ballSocket.Attachment1 = attachment1
            ballSocket.LimitsEnabled = true
            ballSocket.TwistLimitsEnabled = true
            ballSocket.Parent = joint.Parent
            
            joint.Enabled = false
        end
    end
end

local function disableRagdoll(character)
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end
    
    -- ลบ Constraints ที่สร้างขึ้น
    for _, obj in ipairs(character:GetDescendants()) do
        if obj:IsA("BallSocketConstraint") or 
           (obj:IsA("Attachment") and obj.Name == "") then
            obj:Destroy()
        end
    end
    
    -- Enable Joints กลับ
    for _, joint in ipairs(character:GetDescendants()) do
        if joint:IsA("Motor6D") then
            joint.Enabled = true
        end
    end
    
    humanoid:ChangeState(Enum.HumanoidStateType.GettingUp)
end
```

---

## 26.10 Custom Spawn System

```lua
-- Server Script: Custom Spawn
local Players = game:GetService("Players")
local SpawnPoints = workspace:WaitForChild("SpawnPoints")

-- เก็บ spawn points ทั้งหมด
local spawnList = SpawnPoints:GetChildren()
local usedSpawns = {}

local function getRandomSpawn()
    -- หา spawn ที่ว่าง
    local available = {}
    for _, spawn in ipairs(spawnList) do
        if not usedSpawns[spawn] then
            table.insert(available, spawn)
        end
    end
    
    if #available == 0 then
        -- ถ้าไม่มี spawn ว่าง ใช้สุ่ม
        return spawnList[math.random(1, #spawnList)]
    end
    
    return available[math.random(1, #available)]
end

Players.PlayerAdded:Connect(function(player)
    -- ปิด Auto Respawn
    player.RespawnLocation = nil
    
    player.CharacterAdded:Connect(function(character)
        local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
        
        -- หา spawn point
        local spawn = getRandomSpawn()
        usedSpawns[spawn] = player
        
        -- ย้ายไปยัง spawn point
        humanoidRootPart.CFrame = spawn.CFrame + Vector3.new(0, 3, 0)
        
        -- เมื่อตาย ลบออกจาก usedSpawns
        local humanoid = character:WaitForChild("Humanoid")
        humanoid.Died:Connect(function()
            usedSpawns[spawn] = nil
        end)
    end)
end)

Players.PlayerRemoving:Connect(function(player)
    -- ลบ spawn ที่ใช้เมื่อออกจากเกม
    for spawn, p in pairs(usedSpawns) do
        if p == player then
            usedSpawns[spawn] = nil
        end
    end
end)
```

---

## 26.11 ระบบ Team และ Character Color

```lua
-- Server Script: Team System
local Players = game:GetService("Players")
local Teams = game:GetService("Teams")

-- สร้าง Teams
local redTeam = Instance.new("Team")
redTeam.Name = "ทีมแดง"
redTeam.TeamColor = BrickColor.new("Bright red")
redTeam.AutoAssignable = false
redTeam.Parent = Teams

local blueTeam = Instance.new("Team")
blueTeam.Name = "ทีมน้ำเงิน"
blueTeam.TeamColor = BrickColor.new("Bright blue")
blueTeam.AutoAssignable = false
blueTeam.Parent = Teams

local teamCount = {red = 0, blue = 0}

Players.PlayerAdded:Connect(function(player)
    -- Assign team ให้สมดุล
    if teamCount.red <= teamCount.blue then
        player.Team = redTeam
        player.TeamColor = redTeam.TeamColor
        teamCount.red = teamCount.red + 1
    else
        player.Team = blueTeam
        player.TeamColor = blueTeam.TeamColor
        teamCount.blue = teamCount.blue + 1
    end
    
    print(player.Name, "อยู่ทีม", player.Team.Name)
    
    -- เปลี่ยนสีตัวละครตาม Team
    player.CharacterAdded:Connect(function(character)
        local teamColor = player.TeamColor
        
        -- เปลี่ยนสีทุก Part ของ Character
        for _, part in ipairs(character:GetChildren()) do
            if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then
                part.BrickColor = teamColor
            end
        end
    end)
end)

Players.PlayerRemoving:Connect(function(player)
    if player.Team == redTeam then
        teamCount.red = math.max(0, teamCount.red - 1)
    elseif player.Team == blueTeam then
        teamCount.blue = math.max(0, teamCount.blue - 1)
    end
end)
```

---

## 26.12 Character Customization System

```lua
-- ModuleScript: CharacterCustomizer
local CharacterCustomizer = {}

-- เปลี่ยนขนาดตัวละคร
function CharacterCustomizer.setSize(character, scale)
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end
    
    local description = humanoid:GetAppliedDescription()
    description.HeadScale = scale
    description.BodyTypeScale = scale * 0.5
    description.WidthScale = scale
    description.DepthScale = scale
    description.ProportionScale = scale * 0.5
    humanoid:ApplyDescription(description)
end

-- เปลี่ยนสีผิว
function CharacterCustomizer.setSkinColor(character, color)
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end
    
    local description = humanoid:GetAppliedDescription()
    description.SkinColor = color
    humanoid:ApplyDescription(description)
end

-- ทำให้ล่องหน
function CharacterCustomizer.setTransparency(character, transparency)
    for _, part in ipairs(character:GetDescendants()) do
        if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then
            part.Transparency = transparency
        end
    end
end

-- เรืองแสง
function CharacterCustomizer.makeGlow(character, color, brightness)
    for _, part in ipairs(character:GetDescendants()) do
        if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then
            -- ลบ PointLight เก่า
            local oldLight = part:FindFirstChildOfClass("PointLight")
            if oldLight then oldLight:Destroy() end
            
            -- เพิ่ม PointLight ใหม่
            local light = Instance.new("PointLight")
            light.Color = color
            light.Brightness = brightness or 5
            light.Range = 20
            light.Parent = part
        end
    end
end

return CharacterCustomizer
```

---

## 26.13 แบบฝึกหัด

### แบบฝึกหัดที่ 1: Health Bar System
สร้างระบบแถบชีวิตที่แสดงเหนือหัวตัวละคร

```lua
-- LocalScript: HealthBar
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local head = character:WaitForChild("Head")

-- สร้าง BillboardGui สำหรับ Health Bar
local billboard = Instance.new("BillboardGui")
billboard.Size = UDim2.new(0, 100, 0, 10)
billboard.StudsOffset = Vector3.new(0, 3, 0)
billboard.AlwaysOnTop = false
billboard.Parent = head

-- สร้าง Background
local background = Instance.new("Frame")
background.Size = UDim2.new(1, 0, 1, 0)
background.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
background.BorderSizePixel = 0
background.Parent = billboard

-- สร้าง Health Bar
local healthBar = Instance.new("Frame")
healthBar.Size = UDim2.new(1, 0, 1, 0)
healthBar.BackgroundColor3 = Color3.fromRGB(0, 200, 0)
healthBar.BorderSizePixel = 0
healthBar.Parent = background

-- อัพเดต Health Bar
humanoid.HealthChanged:Connect(function(health)
    local percentage = health / humanoid.MaxHealth
    healthBar.Size = UDim2.new(percentage, 0, 1, 0)
    
    -- เปลี่ยนสีตามชีวิต
    if percentage > 0.6 then
        healthBar.BackgroundColor3 = Color3.fromRGB(0, 200, 0)  -- เขียว
    elseif percentage > 0.3 then
        healthBar.BackgroundColor3 = Color3.fromRGB(255, 200, 0)  -- เหลือง
    else
        healthBar.BackgroundColor3 = Color3.fromRGB(200, 0, 0)  -- แดง
    end
end)
```

### แบบฝึกหัดที่ 2: Speed Boost Zone
สร้างโซนที่ทำให้ผู้เล่นวิ่งเร็วขึ้น

```lua
-- Script: SpeedBoostZone
local speedZone = workspace.SpeedBoostZone  -- Part ในชื่อ SpeedBoostZone
local Players = game:GetService("Players")

local boostSpeed = 50
local normalSpeed = 16
local playersInZone = {}

speedZone.Touched:Connect(function(hit)
    local character = hit.Parent
    local player = Players:GetPlayerFromCharacter(character)
    
    if player and not playersInZone[player] then
        local humanoid = character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            playersInZone[player] = true
            humanoid.WalkSpeed = boostSpeed
            print(player.Name, "ได้รับ Speed Boost!")
        end
    end
end)

speedZone.TouchEnded:Connect(function(hit)
    local character = hit.Parent
    local player = Players:GetPlayerFromCharacter(character)
    
    if player and playersInZone[player] then
        local humanoid = character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            playersInZone[player] = nil
            humanoid.WalkSpeed = normalSpeed
            print(player.Name, "ออกจาก Speed Zone")
        end
    end
end)
```

---

## 26.14 สรุป

ในบทนี้เราได้เรียนรู้:

1. **โครงสร้างของ Character** - Parts ต่างๆ ที่ประกอบขึ้น
2. **Humanoid Properties** - Health, WalkSpeed, JumpHeight, State
3. **การจัดการ Character** - Respawn, Anchor, เปลี่ยนรูปลักษณ์
4. **HumanoidDescription** - การปรับแต่งรูปร่างและสีผิว
5. **Events ของ Humanoid** - HealthChanged, Died, StateChanged
6. **ระบบ Damage และ Healing** - TakeDamage, การรักษา
7. **Animations** - LoadAnimation, Play, Stop
8. **Team System** - การแบ่งทีมและเปลี่ยนสีตัวละคร

ในบทต่อไป เราจะเรียนรู้เกี่ยวกับการควบคุมการเคลื่อนที่ของผู้เล่นอย่างละเอียด

---

## แหล่งอ้างอิง

- [Roblox Developer Hub - Humanoid](https://developer.roblox.com/en-us/api-reference/class/Humanoid)
- [Roblox Developer Hub - Character](https://developer.roblox.com/en-us/api-reference/class/Model)
- [Roblox Developer Hub - HumanoidDescription](https://developer.roblox.com/en-us/api-reference/class/HumanoidDescription)
