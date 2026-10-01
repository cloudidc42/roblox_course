# Part 67: First/Third Person Shooter Mechanics

## บทนำ

Shooter Game เป็นประเภทเกมที่ต้องการความแม่นยำในการออกแบบระบบ ตั้งแต่กลไกการยิง ระบบอาวุธ ไปจนถึงระบบ Respawn ในบทนี้เราจะสร้าง Shooter Game ที่สมบูรณ์แบบ

---

## 67.1 โครงสร้างเกม

```
ServerScriptService
├── ShooterCore (Script)
├── WeaponSystem (Script)
├── HitDetection (Script)
└── RespawnSystem (Script)

StarterPack
└── WeaponTool (Tool)
    ├── Handle (Part)
    └── WeaponScript (LocalScript)

StarterPlayerScripts
├── ShooterController (LocalScript)
└── CameraController (LocalScript)

ReplicatedStorage
├── Remotes
│   ├── FireWeapon (RemoteEvent)
│   ├── ReloadWeapon (RemoteEvent)
│   └── HitConfirm (RemoteEvent)
└── Modules
    ├── WeaponData (ModuleScript)
    └── BulletData (ModuleScript)
```

---

## 67.2 ระบบอาวุธ

### 67.2.1 WeaponData Module

```lua
-- ReplicatedStorage/Modules/WeaponData.lua

local WeaponData = {}

local weapons = {
    ["pistol"] = {
        name = "ปืนพก",
        damage = 25,
        headshotMultiplier = 2.0,
        range = 100,
        fireRate = 3,           -- ยิงต่อวินาที
        magazineSize = 12,
        totalAmmo = 48,
        reloadTime = 1.5,
        bulletSpread = 2,       -- ความแม่นยำ (องศา)
        bulletSpeed = 200,
        automatic = false,
        recoil = {
            vertical = 1,
            horizontal = 0.5
        },
        sounds = {
            fire = "rbxassetid://9120268922",
            reload = "rbxassetid://9120268923",
            empty = "rbxassetid://9120268924"
        }
    },
    
    ["assault_rifle"] = {
        name = "ปืนไรเฟิล",
        damage = 30,
        headshotMultiplier = 1.5,
        range = 200,
        fireRate = 8,
        magazineSize = 30,
        totalAmmo = 120,
        reloadTime = 2.5,
        bulletSpread = 3,
        bulletSpeed = 400,
        automatic = true,
        recoil = {
            vertical = 1.5,
            horizontal = 0.8
        },
        sounds = {
            fire = "rbxassetid://9120268925",
            reload = "rbxassetid://9120268926",
            empty = "rbxassetid://9120268924"
        }
    },
    
    ["sniper"] = {
        name = "สไนเปอร์",
        damage = 150,
        headshotMultiplier = 3.0,
        range = 1000,
        fireRate = 0.5,
        magazineSize = 5,
        totalAmmo = 20,
        reloadTime = 4.0,
        bulletSpread = 0.1,     -- แม่นมาก
        bulletSpeed = 1000,
        automatic = false,
        recoil = {
            vertical = 5,
            horizontal = 1
        },
        zoomFOV = 20,           -- Scope zoom
        sounds = {
            fire = "rbxassetid://9120268927",
            reload = "rbxassetid://9120268928",
            empty = "rbxassetid://9120268924"
        }
    },
    
    ["shotgun"] = {
        name = "ช็อตกัน",
        damage = 20,            -- ต่อ pellet
        pelletCount = 8,        -- จำนวน pellets ต่อนัด
        headshotMultiplier = 1.5,
        range = 50,
        fireRate = 1.2,
        magazineSize = 6,
        totalAmmo = 24,
        reloadTime = 3.5,
        bulletSpread = 10,      -- กระจายมาก
        bulletSpeed = 150,
        automatic = false,
        recoil = {
            vertical = 4,
            horizontal = 1.5
        },
        sounds = {
            fire = "rbxassetid://9120268929",
            reload = "rbxassetid://9120268930",
            empty = "rbxassetid://9120268924"
        }
    },
}

function WeaponData.getWeapon(weaponId)
    return weapons[weaponId]
end

function WeaponData.getAllWeapons()
    return weapons
end

return WeaponData
```

### 67.2.2 Weapon Tool LocalScript

```lua
-- StarterPack/WeaponTool/WeaponScript (LocalScript)
-- ควบคุมอาวุธฝั่ง Client

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera
local mouse = player:GetMouse()

-- โหลด module
local WeaponData = require(ReplicatedStorage.Modules.WeaponData)

-- Tool reference
local tool = script.Parent
local handle = tool:FindFirstChild("Handle")

-- อ่านข้อมูลอาวุธ
local weaponId = tool:GetAttribute("WeaponId") or "pistol"
local weaponConfig = WeaponData.getWeapon(weaponId)

-- Remotes
local Remotes = ReplicatedStorage:WaitForChild("Remotes")
local fireRemote = Remotes:WaitForChild("FireWeapon")
local reloadRemote = Remotes:WaitForChild("ReloadWeapon")
local hitConfirmEvent = Remotes:WaitForChild("HitConfirm")

-- State
local state = {
    currentAmmo = weaponConfig.magazineSize,
    totalAmmo = weaponConfig.totalAmmo,
    isReloading = false,
    isFiring = false,
    lastFireTime = 0,
    isAiming = false,
}

-- UI
local playerGui = player.PlayerGui
local shooterHUD = playerGui:FindFirstChild("ShooterHUD")

-- สร้าง HUD ถ้ายังไม่มี
if not shooterHUD then
    shooterHUD = Instance.new("ScreenGui")
    shooterHUD.Name = "ShooterHUD"
    shooterHUD.ResetOnSpawn = false
    shooterHUD.Parent = playerGui
    
    -- Crosshair
    local crosshairFrame = Instance.new("Frame")
    crosshairFrame.Name = "Crosshair"
    crosshairFrame.Size = UDim2.new(0, 20, 0, 20)
    crosshairFrame.Position = UDim2.new(0.5, -10, 0.5, -10)
    crosshairFrame.BackgroundTransparency = 1
    crosshairFrame.Parent = shooterHUD
    
    -- สร้างเส้น crosshair
    local lines = {
        {size = UDim2.new(0, 2, 0, 8), pos = UDim2.new(0.5, -1, 0, 0)},    -- Top
        {size = UDim2.new(0, 2, 0, 8), pos = UDim2.new(0.5, -1, 1, -8)},   -- Bottom
        {size = UDim2.new(0, 8, 0, 2), pos = UDim2.new(0, 0, 0.5, -1)},    -- Left
        {size = UDim2.new(0, 8, 0, 2), pos = UDim2.new(1, -8, 0.5, -1)},   -- Right
    }
    
    for _, lineData in ipairs(lines) do
        local line = Instance.new("Frame")
        line.Size = lineData.size
        line.Position = lineData.pos
        line.BackgroundColor3 = Color3.new(1, 1, 1)
        line.BorderSizePixel = 0
        line.ZIndex = 10
        line.Parent = crosshairFrame
    end
    
    -- Ammo Display
    local ammoFrame = Instance.new("Frame")
    ammoFrame.Name = "AmmoFrame"
    ammoFrame.Size = UDim2.new(0, 150, 0, 60)
    ammoFrame.Position = UDim2.new(1, -160, 1, -80)
    ammoFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    ammoFrame.BackgroundTransparency = 0.5
    ammoFrame.BorderSizePixel = 0
    ammoFrame.Parent = shooterHUD
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 8)
    corner.Parent = ammoFrame
    
    local ammoLabel = Instance.new("TextLabel")
    ammoLabel.Name = "AmmoLabel"
    ammoLabel.Size = UDim2.new(1, 0, 0.6, 0)
    ammoLabel.BackgroundTransparency = 1
    ammoLabel.Text = "12 / 48"
    ammoLabel.TextColor3 = Color3.new(1, 1, 1)
    ammoLabel.TextScaled = true
    ammoLabel.Font = Enum.Font.GothamBold
    ammoLabel.Parent = ammoFrame
    
    local weaponNameLabel = Instance.new("TextLabel")
    weaponNameLabel.Name = "WeaponName"
    weaponNameLabel.Size = UDim2.new(1, 0, 0.4, 0)
    weaponNameLabel.Position = UDim2.new(0, 0, 0.6, 0)
    weaponNameLabel.BackgroundTransparency = 1
    weaponNameLabel.Text = weaponConfig.name
    weaponNameLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
    weaponNameLabel.TextScaled = true
    weaponNameLabel.Font = Enum.Font.Gotham
    weaponNameLabel.Parent = ammoFrame
end

-- อัพเดท Ammo HUD
local function updateAmmoHUD()
    local ammoLabel = shooterHUD:FindFirstChild("AmmoLabel", true)
    if ammoLabel then
        ammoLabel.Text = state.currentAmmo .. " / " .. state.totalAmmo
        ammoLabel.TextColor3 = state.currentAmmo == 0 
            and Color3.fromRGB(255, 0, 0)
            or Color3.new(1, 1, 1)
    end
end

-- Recoil
local recoilOffset = Vector2.new(0, 0)
local function applyRecoil()
    local config = weaponConfig.recoil
    recoilOffset = recoilOffset + Vector2.new(
        (math.random() - 0.5) * config.horizontal,
        -config.vertical
    )
end

local function resetRecoil()
    recoilOffset = recoilOffset:Lerp(Vector2.new(0, 0), 0.1)
    camera.CFrame = camera.CFrame * CFrame.Angles(
        math.rad(recoilOffset.Y * 0.1),
        math.rad(recoilOffset.X * 0.1),
        0
    )
end

-- Muzzle Flash
local function showMuzzleFlash()
    if not handle then return end
    
    local flash = Instance.new("PointLight")
    flash.Brightness = 20
    flash.Range = 20
    flash.Color = Color3.fromRGB(255, 200, 100)
    flash.Parent = handle
    
    game:GetService("Debris"):AddItem(flash, 0.05)
end

-- Spread คำนวณ
local function calculateSpread()
    local spread = weaponConfig.bulletSpread
    
    -- เพิ่ม spread ถ้าวิ่ง
    local character = player.Character
    if character then
        local humanoid = character:FindFirstChildWhichIsA("Humanoid")
        if humanoid and humanoid.MoveDirection.Magnitude > 0 then
            spread = spread * 2
        end
    end
    
    -- ลด spread ถ้า ADS
    if state.isAiming then
        spread = spread * 0.3
    end
    
    return spread
end

-- ยิงกระสุน
local function fire()
    local now = tick()
    local fireCooldown = 1 / weaponConfig.fireRate
    
    if now - state.lastFireTime < fireCooldown then return end
    if state.isReloading then return end
    if state.currentAmmo <= 0 then
        -- เสียงปืนหมดกระสุน
        local sound = Instance.new("Sound")
        sound.SoundId = weaponConfig.sounds.empty
        sound.Parent = handle or workspace
        sound:Play()
        game:GetService("Debris"):AddItem(sound, 2)
        return
    end
    
    state.lastFireTime = now
    state.currentAmmo = state.currentAmmo - 1
    updateAmmoHUD()
    
    -- คำนวณทิศทางกระสุน
    local spread = calculateSpread()
    local spreadRad = math.rad(spread)
    
    local direction = (mouse.Hit.Position - camera.CFrame.Position).Unit
    
    -- เพิ่ม spread แบบ random
    local spreadX = (math.random() - 0.5) * 2 * spreadRad
    local spreadY = (math.random() - 0.5) * 2 * spreadRad
    direction = (CFrame.new(Vector3.new(), direction) * 
        CFrame.Angles(spreadY, spreadX, 0)).LookVector
    
    -- สำหรับ Shotgun: ยิงหลาย pellets
    local pelletCount = weaponConfig.pelletCount or 1
    local directions = {}
    
    for i = 1, pelletCount do
        if i == 1 then
            table.insert(directions, direction)
        else
            local pelletSpread = math.rad(weaponConfig.bulletSpread)
            local px = (math.random() - 0.5) * 2 * pelletSpread
            local py = (math.random() - 0.5) * 2 * pelletSpread
            local pd = (CFrame.new(Vector3.new(), direction) * 
                CFrame.Angles(py, px, 0)).LookVector
            table.insert(directions, pd)
        end
    end
    
    -- ส่งไป Server
    local startPos = camera.CFrame.Position
    if handle then
        startPos = handle.Position
    end
    
    fireRemote:FireServer(weaponId, startPos, directions)
    
    -- Visual effects (Client-side)
    showMuzzleFlash()
    applyRecoil()
    
    -- เสียง
    local sound = Instance.new("Sound")
    sound.SoundId = weaponConfig.sounds.fire
    sound.Volume = 0.8
    sound.Parent = handle or workspace
    sound:Play()
    game:GetService("Debris"):AddItem(sound, 3)
    
    -- Eject casing
    if handle then
        local casing = Instance.new("Part")
        casing.Name = "Casing"
        casing.Size = Vector3.new(0.1, 0.1, 0.3)
        casing.Position = handle.Position
        casing.BrickColor = BrickColor.new("Bright yellow")
        casing.Material = Enum.Material.SmoothPlastic
        casing.Parent = workspace
        
        local v = Instance.new("BodyVelocity")
        v.Velocity = handle.CFrame.RightVector * 10 + Vector3.new(0, 5, 0)
        v.MaxForce = Vector3.new(1e5, 1e5, 1e5)
        v.Parent = casing
        
        game:GetService("Debris"):AddItem(casing, 3)
    end
    
    -- Auto reload ถ้าหมด magazine
    if state.currentAmmo == 0 and state.totalAmmo > 0 then
        task.delay(0.5, reload)
    end
end

-- Reload
local function reload()
    if state.isReloading then return end
    if state.currentAmmo == weaponConfig.magazineSize then return end
    if state.totalAmmo == 0 then return end
    
    state.isReloading = true
    
    -- Reload animation / sound
    local sound = Instance.new("Sound")
    sound.SoundId = weaponConfig.sounds.reload
    sound.Parent = handle or workspace
    sound:Play()
    
    -- แสดง "กำลัง Reload"
    local ammoLabel = shooterHUD:FindFirstChild("AmmoLabel", true)
    if ammoLabel then
        ammoLabel.Text = "RELOADING..."
        ammoLabel.TextColor3 = Color3.fromRGB(255, 165, 0)
    end
    
    task.wait(weaponConfig.reloadTime)
    
    -- คำนวณ ammo หลัง reload
    local needed = weaponConfig.magazineSize - state.currentAmmo
    local available = math.min(needed, state.totalAmmo)
    
    state.currentAmmo = state.currentAmmo + available
    state.totalAmmo = state.totalAmmo - available
    state.isReloading = false
    
    reloadRemote:FireServer(weaponId, state.currentAmmo, state.totalAmmo)
    updateAmmoHUD()
    
    print("Reload เสร็จ: " .. state.currentAmmo .. " กระสุน")
end

-- ADS (Aim Down Sight)
local function startADS()
    state.isAiming = true
    
    -- เปลี่ยน FOV
    local targetFOV = weaponConfig.zoomFOV or 45
    TweenService:Create(camera, TweenInfo.new(0.2), {FieldOfView = targetFOV}):Play()
end

local function stopADS()
    state.isAiming = false
    TweenService:Create(camera, TweenInfo.new(0.2), {FieldOfView = 70}):Play()
end

-- Input handling
tool.Equipped:Connect(function()
    -- Show HUD
    shooterHUD.Enabled = true
    updateAmmoHUD()
    
    -- ซ่อน default character cursor
    UserInputService.MouseIconEnabled = false
    
    -- คลิกขวา = ADS
    mouse.Button2Down:Connect(startADS)
    mouse.Button2Up:Connect(stopADS)
end)

tool.Unequipped:Connect(function()
    shooterHUD.Enabled = false
    UserInputService.MouseIconEnabled = true
    state.isAiming = false
    camera.FieldOfView = 70
end)

-- ยิง
tool.Activated:Connect(fire)

-- Auto fire
local fireConnection
UserInputService.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        if weaponConfig.automatic then
            fireConnection = RunService.Heartbeat:Connect(function()
                if UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton1) then
                    fire()
                end
            end)
        end
    end
    
    if input.KeyCode == Enum.KeyCode.R then
        reload()
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        if fireConnection then
            fireConnection:Disconnect()
            fireConnection = nil
        end
    end
end)

-- Update loop
RunService.RenderStepped:Connect(function()
    resetRecoil()
end)

-- Hitmarker
hitConfirmEvent.OnClientEvent:Connect(function(isHeadshot)
    -- แสดง hitmarker
    local crosshair = shooterHUD:FindFirstChild("Crosshair", true)
    if crosshair then
        for _, line in ipairs(crosshair:GetChildren()) do
            if line:IsA("Frame") then
                local origColor = line.BackgroundColor3
                line.BackgroundColor3 = isHeadshot 
                    and Color3.fromRGB(255, 100, 0)
                    or Color3.fromRGB(255, 0, 0)
                
                task.delay(0.1, function()
                    line.BackgroundColor3 = origColor
                end)
            end
        end
    end
end)
```

---

## 67.3 Server-Side Hit Detection

```lua
-- ServerScriptService/HitDetection.lua
-- ตรวจสอบการโจมตีฝั่ง Server

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local WeaponData = require(ReplicatedStorage.Modules.WeaponData)

local Remotes = ReplicatedStorage:WaitForChild("Remotes")
local fireRemote = Remotes:WaitForChild("FireWeapon")
local hitConfirmEvent = Remotes:WaitForChild("HitConfirm")

-- Config สำหรับป้องกันการโกง
local MAX_BULLETS_PER_SECOND = 20
local MAX_BULLET_DISTANCE = 500  -- studs

-- เก็บข้อมูล bullet timestamps
local bulletTimestamps = {}

-- ระยะทางกระสุนสูงสุด
local function validateShot(player, startPos, directions, weaponConfig)
    -- ตรวจสอบ rate of fire
    local userId = player.UserId
    local now = tick()
    
    if not bulletTimestamps[userId] then
        bulletTimestamps[userId] = {}
    end
    
    -- ลบ timestamps เก่า
    local timestamps = bulletTimestamps[userId]
    local validTimestamps = {}
    for _, ts in ipairs(timestamps) do
        if now - ts < 1 then
            table.insert(validTimestamps, ts)
        end
    end
    bulletTimestamps[userId] = validTimestamps
    
    -- ตรวจสอบจำนวนกระสุน
    if #validTimestamps >= MAX_BULLETS_PER_SECOND then
        warn(player.Name .. " ยิงเร็วเกินไป! อาจโกง")
        return false
    end
    
    table.insert(bulletTimestamps[userId], now)
    
    -- ตรวจสอบตำแหน่ง
    local character = player.Character
    if not character or not character:FindFirstChild("HumanoidRootPart") then
        return false
    end
    
    local playerPos = character.HumanoidRootPart.Position
    local distance = (startPos - playerPos).Magnitude
    
    if distance > 15 then  -- ไกลเกินไปจากตัวผู้เล่น
        warn(player.Name .. " ยิงจากตำแหน่งผิดปกติ (" .. math.floor(distance) .. " studs)")
        return false
    end
    
    return true
end

-- Raycast สำหรับกระสุน
local function performRaycast(startPos, direction, maxDistance, excludeList)
    local rayParams = RaycastParams.new()
    rayParams.FilterDescendantsInstances = excludeList or {}
    rayParams.FilterType = Enum.RaycastFilterType.Exclude
    
    return workspace:Raycast(startPos, direction * maxDistance, rayParams)
end

-- ประมวลผลกระสุน
local function processBullet(player, weaponId, startPos, direction)
    local weaponConfig = WeaponData.getWeapon(weaponId)
    if not weaponConfig then return end
    
    local character = player.Character
    local excludeList = character and {character} or {}
    
    -- สร้าง Bullet Trail (Visual)
    local trailEnd = startPos + direction * weaponConfig.range
    
    -- Raycast
    local result = performRaycast(startPos, direction, weaponConfig.range, excludeList)
    
    if result then
        local hitInstance = result.Instance
        local hitPos = result.Position
        
        -- ตรวจสอบว่าโดน Character
        local hitCharacter = hitInstance:FindFirstAncestorWhichIsA("Model")
        if hitCharacter then
            local humanoid = hitCharacter:FindFirstChildWhichIsA("Humanoid")
            local hitPlayer = Players:GetPlayerFromCharacter(hitCharacter)
            
            if humanoid and humanoid.Health > 0 then
                -- คำนวณ damage
                local damage = weaponConfig.damage
                local isHeadshot = hitInstance.Name == "Head"
                
                if isHeadshot then
                    damage = damage * weaponConfig.headshotMultiplier
                end
                
                -- ลด damage ตามระยะทาง
                local dist = (hitPos - startPos).Magnitude
                local falloff = math.max(0.5, 1 - dist / weaponConfig.range)
                damage = math.floor(damage * falloff)
                
                -- ใส่ damage
                humanoid:TakeDamage(damage)
                
                -- Hit confirm
                hitConfirmEvent:FireClient(player, isHeadshot)
                
                -- แสดงตัวเลข damage
                local dmgEvent = Remotes:FindFirstChild("DamageNumber")
                if dmgEvent then
                    dmgEvent:FireAllClients(hitPos, damage, isHeadshot)
                end
                
                print(player.Name .. " ยิงโดน " .. (hitPlayer and hitPlayer.Name or hitCharacter.Name) .. 
                    " - " .. damage .. " damage" .. (isHeadshot and " (HEADSHOT!)" or ""))
            end
        else
            -- โดน wall/environment
            createBulletHole(hitPos, result.Normal)
        end
    end
end

-- สร้างรอยกระสุน
function createBulletHole(position, normal)
    local hole = Instance.new("Part")
    hole.Name = "BulletHole"
    hole.Size = Vector3.new(0.3, 0.3, 0.1)
    hole.CFrame = CFrame.new(position, position + normal) * CFrame.Angles(0, 0, math.random(0, 628)/100)
    hole.Anchored = true
    hole.CanCollide = false
    hole.BrickColor = BrickColor.new("Black")
    hole.Material = Enum.Material.SmoothPlastic
    hole.Parent = workspace
    
    game:GetService("Debris"):AddItem(hole, 10)  -- ลบหลัง 10 วินาที
end

-- Remote handler
fireRemote.OnServerEvent:Connect(function(player, weaponId, startPos, directions)
    local weaponConfig = WeaponData.getWeapon(weaponId)
    if not weaponConfig then return end
    
    -- Validate
    if not validateShot(player, startPos, directions, weaponConfig) then return end
    
    -- Process แต่ละ bullet/pellet
    for _, direction in ipairs(directions) do
        processBullet(player, weaponId, startPos, direction)
    end
end)
```

---

## 67.4 Respawn System

```lua
-- ServerScriptService/RespawnSystem.lua
-- ระบบ Respawn

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local RESPAWN_TIME = 5  -- วินาที
local playerStats = {}  -- เก็บ kills/deaths

-- เริ่มต้นข้อมูลผู้เล่น
Players.PlayerAdded:Connect(function(player)
    playerStats[player.UserId] = {kills = 0, deaths = 0}
    
    -- Leaderstats
    local leaderstats = Instance.new("Folder")
    leaderstats.Name = "leaderstats"
    leaderstats.Parent = player
    
    local killsVal = Instance.new("IntValue")
    killsVal.Name = "Kills"
    killsVal.Value = 0
    killsVal.Parent = leaderstats
    
    local deathsVal = Instance.new("IntValue")
    deathsVal.Name = "Deaths"
    deathsVal.Value = 0
    deathsVal.Parent = leaderstats
    
    -- ตรวจสอบตาย
    player.CharacterAdded:Connect(function(character)
        local humanoid = character:WaitForChild("Humanoid")
        
        humanoid.Died:Connect(function()
            local data = playerStats[player.UserId]
            if data then
                data.deaths = data.deaths + 1
                local leaderstats = player:FindFirstChild("leaderstats")
                if leaderstats and leaderstats:FindFirstChild("Deaths") then
                    leaderstats.Deaths.Value = data.deaths
                end
            end
            
            -- แจ้งระบบ kill credit
            local killCredit = humanoid:GetAttribute("KilledBy")
            if killCredit then
                local killer = Players:FindFirstChild(killCredit)
                if killer then
                    local killerData = playerStats[killer.UserId]
                    if killerData then
                        killerData.kills = killerData.kills + 1
                        local killerLeader = killer:FindFirstChild("leaderstats")
                        if killerLeader and killerLeader:FindFirstChild("Kills") then
                            killerLeader.Kills.Value = killerData.kills
                        end
                    end
                end
            end
            
            -- Respawn
            task.delay(RESPAWN_TIME, function()
                if player.Parent then
                    player:LoadCharacter()
                end
            end)
        end)
        
        -- ตั้ง Kill Credit เมื่อถูกโจมตี
        humanoid.HealthChanged:Connect(function(hp)
            -- ระบบ damage tagging
        end)
    end)
end)

Players.PlayerRemoving:Connect(function(player)
    playerStats[player.UserId] = nil
end)
```

---

## 67.5 Map Design สำหรับ Shooter

```lua
-- Script สร้าง Cover System
-- ServerScriptService/MapBuilder

local function createCoverObject(position, size, rotation)
    local cover = Instance.new("Part")
    cover.Name = "Cover"
    cover.Size = size
    cover.CFrame = CFrame.new(position) * CFrame.Angles(0, math.rad(rotation), 0)
    cover.Anchored = true
    cover.BrickColor = BrickColor.new("Medium stone grey")
    cover.Material = Enum.Material.SmoothPlastic
    cover.Parent = workspace.Map or workspace
    return cover
end

-- สร้าง Cover ในแผนที่
local coverLayout = {
    -- Low covers (ซ่อนได้ แต่ยังยิงข้ามได้)
    {pos = Vector3.new(20, 1.5, 0), size = Vector3.new(6, 3, 1), rot = 0},
    {pos = Vector3.new(-20, 1.5, 0), size = Vector3.new(6, 3, 1), rot = 0},
    {pos = Vector3.new(0, 1.5, 20), size = Vector3.new(1, 3, 6), rot = 0},
    
    -- Full covers (ซ่อนได้สนิท)
    {pos = Vector3.new(30, 3, 30), size = Vector3.new(5, 6, 2), rot = 45},
    {pos = Vector3.new(-30, 3, 30), size = Vector3.new(5, 6, 2), rot = -45},
    
    -- Boxes
    {pos = Vector3.new(10, 2, 15), size = Vector3.new(4, 4, 4), rot = 30},
    {pos = Vector3.new(-10, 2, 15), size = Vector3.new(4, 4, 4), rot = -20},
}

local function buildMap()
    local mapFolder = workspace:FindFirstChild("Map") or Instance.new("Folder")
    mapFolder.Name = "Map"
    mapFolder.Parent = workspace
    
    for _, cover in ipairs(coverLayout) do
        createCoverObject(cover.pos, cover.size, cover.rot)
    end
    
    print("สร้าง Map เสร็จแล้ว - " .. #coverLayout .. " cover objects")
end

buildMap()
```

---

## 67.6 ข้อผิดพลาดที่พบบ่อย

```lua
-- ❌ ผิด: ทำ damage บน Client
-- LocalScript:
humanoid:TakeDamage(damage)  -- โกงได้ง่ายมาก!

-- ✓ ถูก: ส่ง hit detection ไป Server และทำ damage บน Server
-- LocalScript:
fireRemote:FireServer(weaponId, startPos, direction)

-- ServerScript:
fireRemote.OnServerEvent:Connect(function(player, weaponId, startPos, direction)
    -- ตรวจสอบ validity
    -- แล้วทำ damage บน Server เท่านั้น
    humanoid:TakeDamage(damage)
end)
```

---

## 67.7 แบบฝึกหัด

### แบบฝึกหัดที่ 1: เพิ่มระบบ Grenade
สร้าง Grenade Tool:
- โยนด้วยคลิกซ้าย
- ระเบิดหลัง 3 วินาที
- AOE damage

### แบบฝึกหัดที่ 2: สร้าง Team System
ระบบทีม:
- แบ่ง 2 ทีม (Red vs Blue)
- ไม่สามารถยิงเพื่อนร่วมทีม
- ชนะเมื่อ kills ถึง 25

### แบบฝึกหัดที่ 3: เพิ่ม Kill Streak
ระบบ Kill Streak:
- 3 kills: Speed Boost
- 5 kills: Airstrike
- 10 kills: Nuke (ทุกคนตาย)

---

## สรุป

ในบทนี้เราได้สร้าง Shooter Game ที่มี:
- Weapon system พร้อม recoil, spread, reload
- Server-side hit detection (Anti-cheat)
- Respawn system
- Hitmarker และ HUD

ในบทถัดไปเราจะสร้าง Horror Game!
