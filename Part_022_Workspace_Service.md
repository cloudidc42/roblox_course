# ตอนที่ 22: Workspace Service ใน Roblox

## บทนำ

Workspace เป็น service ที่เก็บทุกสิ่งที่อยู่ใน "โลก" ของเกม ทั้ง Parts, Models, Characters, Camera รวมถึง scripts ที่รันใน workspace เอง ทุกอย่างที่ผู้เล่นมองเห็นและโต้ตอบในเกมจะอยู่ใน Workspace

---

## 22.1 การเข้าถึง Workspace

```lua
-- สามวิธีที่เทียบเท่ากัน
local workspace1 = game.Workspace
local workspace2 = game:GetService("Workspace")
local workspace3 = workspace  -- shorthand ใน Scripts

print(workspace1 == workspace2)  -- true
print(workspace2 == workspace3)  -- true
```

---

## 22.2 Properties สำคัญของ Workspace

```lua
local workspace = game.Workspace

-- ค่าแรงโน้มถ่วง (default: 196.2)
print(workspace.Gravity)
workspace.Gravity = 100  -- ลดแรงโน้มถ่วง (เหมือนบนดวงจันทร์)
workspace.Gravity = 300  -- เพิ่มแรงโน้มถ่วง

-- Camera
local camera = workspace.CurrentCamera
print(camera.FieldOfView)

-- Terrain
local terrain = workspace.Terrain
print(terrain.ClassName)  -- Terrain

-- Physics quality
workspace.StreamingEnabled = false  -- เปิด/ปิด streaming
workspace.StreamingMinRadius = 64
workspace.StreamingTargetRadius = 1024
```

---

## 22.3 Camera Control

```lua
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

-- กำหนดรูปแบบกล้อง
camera.CameraType = Enum.CameraType.Scriptable  -- ควบคุมเอง
-- camera.CameraType = Enum.CameraType.Follow   -- ติดตาม character
-- camera.CameraType = Enum.CameraType.Fixed    -- ตำแหน่งคงที่
-- camera.CameraType = Enum.CameraType.Attach   -- ติดกับ object

-- กำหนด CFrame ของกล้อง
camera.CFrame = CFrame.new(0, 50, 100) * CFrame.Angles(-0.5, 0, 0)

-- ズoom
camera.FieldOfView = 70  -- default 70

-- Smooth camera movement ด้วย Tween
local function tweenCamera(targetCFrame, duration)
    local tweenInfo = TweenInfo.new(duration, Enum.EasingStyle.Sine)
    local tween = TweenService:Create(camera, tweenInfo, {
        CFrame = targetCFrame
    })
    tween:Play()
    return tween
end

-- Cinematic camera
local function playCinematic()
    camera.CameraType = Enum.CameraType.Scriptable
    
    -- Keyframes
    local keyframes = {
        {CFrame.new(0, 20, 50) * CFrame.Angles(-0.3, 0, 0), 3},
        {CFrame.new(30, 10, 30) * CFrame.Angles(-0.2, -0.5, 0), 4},
        {CFrame.new(-10, 5, 20) * CFrame.Angles(-0.1, 0.3, 0), 3},
    }
    
    for _, keyframe in ipairs(keyframes) do
        local tween = tweenCamera(keyframe[1], keyframe[2])
        tween.Completed:Wait()
    end
    
    camera.CameraType = Enum.CameraType.Custom  -- กลับกล้องปกติ
end
```

---

## 22.4 Raycast

Raycast ใช้สำหรับตรวจสอบ collision บนเส้นตรง:

```lua
local workspace = game.Workspace

-- Raycast พื้นฐาน
local origin = Vector3.new(0, 100, 0)
local direction = Vector3.new(0, -200, 0)  -- ลงมา

local raycastParams = RaycastParams.new()
raycastParams.FilterType = Enum.RaycastFilterType.Exclude
raycastParams.FilterDescendantsInstances = {}  -- ไม่ exclude อะไร

local result = workspace:Raycast(origin, direction, raycastParams)

if result then
    print("ชน: " .. result.Instance.Name)
    print("ตำแหน่ง: " .. tostring(result.Position))
    print("Normal: " .. tostring(result.Normal))
    print("Material: " .. tostring(result.Material))
else
    print("ไม่ชนอะไร")
end

-- Raycast ที่กรอง objects บางอย่างออก
local function raycastIgnoreCharacter(origin, direction, character)
    local params = RaycastParams.new()
    params.FilterType = Enum.RaycastFilterType.Exclude
    params.FilterDescendantsInstances = {character}  -- ไม่นับ character เอง
    
    return workspace:Raycast(origin, direction, params)
end

-- Raycast สำหรับการยิง
local function fireBullet(origin, direction, player)
    local params = RaycastParams.new()
    params.FilterType = Enum.RaycastFilterType.Exclude
    params.FilterDescendantsInstances = {player.Character}
    
    local result = workspace:Raycast(origin, direction * 500, params)
    
    if result then
        -- ตรวจสอบว่าชน player หรือไม่
        local hitCharacter = result.Instance.Parent
        local hitPlayer = game.Players:GetPlayerFromCharacter(hitCharacter)
        
        if hitPlayer then
            print("ยิงโดน " .. hitPlayer.Name)
            -- apply damage
        else
            print("ยิงชน " .. result.Instance.Name)
            -- create bullet hole effect
        end
    end
end
```

---

## 22.5 Terrain Manipulation

```lua
local terrain = workspace.Terrain

-- เติม terrain ด้วย material
terrain:FillBlock(
    CFrame.new(0, 0, 0),    -- CFrame
    Vector3.new(100, 20, 100),  -- Size
    Enum.Material.Grass      -- Material
)

-- เติมทรงกลม
terrain:FillBall(
    Vector3.new(0, 50, 0),  -- Position
    30,                      -- Radius
    Enum.Material.Rock
)

-- เติมทรงกระบอก
terrain:FillCylinder(
    CFrame.new(0, 0, 0),
    50,          -- Height
    10,          -- Radius
    Enum.Material.Water
)

-- ลบ terrain
terrain:FillBlock(
    CFrame.new(0, 0, 0),
    Vector3.new(50, 50, 50),
    Enum.Material.Air  -- Air = ว่างเปล่า
)

-- อ่านค่า terrain
local material, occupancy = terrain:GetMaterialColor(
    Vector3.new(0, 0, 0)
)
print("Material ที่ (0,0,0): " .. tostring(material))

-- Procedural terrain generation
local function generateTerrain(size, height)
    local BLOCK_SIZE = 4
    
    for x = -size/2, size/2, BLOCK_SIZE do
        for z = -size/2, size/2, BLOCK_SIZE do
            -- Simple noise-like height
            local h = math.sin(x * 0.1) * math.cos(z * 0.1) * height
            h = math.floor(h)
            
            local material = Enum.Material.Grass
            if h < -2 then
                material = Enum.Material.Sand
            elseif h > 8 then
                material = Enum.Material.Rock
            end
            
            terrain:FillBlock(
                CFrame.new(x, h/2, z),
                Vector3.new(BLOCK_SIZE, math.abs(h) + 1, BLOCK_SIZE),
                material
            )
        end
    end
end
```

---

## 22.6 FindPartOnRay (เก่า) vs Raycast (ใหม่)

```lua
-- แบบเก่า (deprecated)
-- local ray = Ray.new(origin, direction * 500)
-- local part, position = workspace:FindPartOnRay(ray)

-- แบบใหม่ (แนะนำ)
local result = workspace:Raycast(origin, direction * 500)

-- Spherecast (Roblox ใหม่)
-- คล้าย Raycast แต่มีความหนา (sphere)
local spherecastResult = workspace:Spherecast(
    origin,       -- origin
    5,            -- radius
    direction * 100  -- direction (ไม่ต้อง * distance แยก)
)

-- Blockcast
local blockcastResult = workspace:Blockcast(
    CFrame.new(origin),
    Vector3.new(2, 2, 2),  -- size
    direction * 100
)
```

---

## 22.7 WorldToViewportPoint / ViewportToWorld

```lua
-- แปลงตำแหน่ง 3D เป็น 2D screen coordinates
local camera = workspace.CurrentCamera

local worldPosition = Vector3.new(0, 5, 0)
local screenPoint, onScreen = camera:WorldToViewportPoint(worldPosition)

if onScreen then
    print("อยู่บนหน้าจอที่: " .. screenPoint.X .. ", " .. screenPoint.Y)
    -- สามารถใช้สำหรับวาง UI เหนือ object ใน 3D world
end

-- แปลง 2D เป็น 3D ray
local screenX, screenY = 400, 300  -- ตรงกลางหน้าจอ
local unitRay = camera:ViewportPointToRay(screenX, screenY)

local result = workspace:Raycast(
    unitRay.Origin, 
    unitRay.Direction * 1000
)
```

---

## 22.8 ตัวอย่างโปรเจกต์: Shooting System

```lua
-- LocalScript: ShootingSystem.lua
-- ระบบยิงปืนที่ใช้ Raycast

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

-- Remote event สำหรับส่งผลการยิง
local shootEvent = ReplicatedStorage:WaitForChild("ShootEvent")

local canShoot = true
local SHOOT_COOLDOWN = 0.5
local BULLET_RANGE = 500

local function createBulletTracer(startPos, endPos)
    local distance = (endPos - startPos).Magnitude
    
    local tracer = Instance.new("Part")
    tracer.Size = Vector3.new(0.05, 0.05, distance)
    tracer.CFrame = CFrame.new(startPos, endPos) * CFrame.new(0, 0, -distance/2)
    tracer.Anchored = true
    tracer.CanCollide = false
    tracer.Material = Enum.Material.Neon
    tracer.BrickColor = BrickColor.new("Bright yellow")
    tracer.CastShadow = false
    tracer.Parent = workspace
    
    -- ลบหลัง 0.1 วินาที
    game.Debris:AddItem(tracer, 0.1)
end

local function createHitEffect(position, normal)
    -- Spark effect
    local hitEffect = Instance.new("Part")
    hitEffect.Size = Vector3.new(0.1, 0.1, 0.1)
    hitEffect.Position = position
    hitEffect.Anchored = true
    hitEffect.CanCollide = false
    hitEffect.Material = Enum.Material.Neon
    hitEffect.BrickColor = BrickColor.new("Bright yellow")
    hitEffect.Parent = workspace
    
    local sparks = Instance.new("SparkleEmission") -- deprecated แต่ใช้ได้
    -- ใช้ ParticleEmitter แทน
    
    game.Debris:AddItem(hitEffect, 0.5)
end

local function shoot()
    if not canShoot then return end
    if not player.Character then return end
    
    canShoot = false
    
    -- หา direction จาก camera
    local screenCenter = Vector2.new(
        camera.ViewportSize.X / 2,
        camera.ViewportSize.Y / 2
    )
    
    local ray = camera:ViewportPointToRay(screenCenter.X, screenCenter.Y)
    
    -- Raycast
    local params = RaycastParams.new()
    params.FilterType = Enum.RaycastFilterType.Exclude
    params.FilterDescendantsInstances = {player.Character}
    
    local result = workspace:Raycast(ray.Origin, ray.Direction * BULLET_RANGE, params)
    
    local hitPosition = result and result.Position or (ray.Origin + ray.Direction * BULLET_RANGE)
    
    -- แสดง bullet tracer
    local gunBarrel = player.Character:FindFirstChild("Tool")
    local startPos = gunBarrel and gunBarrel.Handle.Position or ray.Origin
    createBulletTracer(startPos, hitPosition)
    
    if result then
        createHitEffect(result.Position, result.Normal)
        
        -- ส่งข้อมูลไป Server
        local hitPlayer = Players:GetPlayerFromCharacter(result.Instance.Parent)
        shootEvent:FireServer(hitPosition, result.Instance, hitPlayer)
    end
    
    -- Cooldown
    task.delay(SHOOT_COOLDOWN, function()
        canShoot = true
    end)
end

-- Input
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        shoot()
    end
end)
```

---

## 22.9 แบบฝึกหัด

### แบบฝึกหัดที่ 1: Ground Detection

```lua
-- สร้างฟังก์ชันตรวจสอบว่า character กำลังยืนบนพื้นหรือไม่
-- ใช้ Raycast ยิงลงจาก HumanoidRootPart

local function isGrounded(character)
    -- เติมโค้ด
end
```

### แบบฝึกหัดที่ 2: Object Placement

```lua
-- สร้างระบบวาง object:
-- - Raycast จาก mouse position
-- - Preview object ที่จะวาง (transparent)
-- - วางจริงเมื่อคลิก

-- เติมโค้ด
```

---

## สรุป

| Feature | การใช้งาน |
|---------|----------|
| `workspace.Gravity` | ควบคุมแรงโน้มถ่วง |
| `workspace.CurrentCamera` | กล้องหลัก |
| `workspace.Terrain` | Terrain object |
| `workspace:Raycast(o, d, p)` | ยิง ray |
| `workspace:Spherecast(o, r, d)` | ยิง sphere |
| `camera.CameraType` | รูปแบบกล้อง |
| `camera.CFrame` | ตำแหน่งกล้อง |
| `camera:WorldToViewportPoint` | 3D -> 2D |
| `camera:ViewportPointToRay` | 2D -> Ray |

### บทถัดไป

ในบทที่ 23 เราจะเรียนเรื่อง **ReplicatedStorage** - การแชร์ข้อมูลระหว่าง Server และ Client
