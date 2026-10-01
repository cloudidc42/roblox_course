# ตอนที่ 28: Camera System และการควบคุม Camera

## บทนำ

ระบบ Camera ใน Roblox เป็นสิ่งที่กำหนด "มุมมอง" ที่ผู้เล่นมองเห็นเกม Roblox มี Camera System ในตัวที่ดีมาก แต่เราสามารถปรับแต่งหรือสร้างระบบ Camera แบบกำหนดเองได้ ในบทนี้เราจะเรียนรู้ทุกอย่างเกี่ยวกับ Camera ตั้งแต่พื้นฐานจนถึงขั้นสูง

---

## 28.1 Camera Basics

### 28.1.1 การเข้าถึง Camera

```lua
-- LocalScript เท่านั้น!
local camera = workspace.CurrentCamera

-- Type ของ Camera
print("Camera Type:", camera.CameraType)
-- Enum.CameraType.Custom (default - ติดตาม Character)
-- Enum.CameraType.Scriptable (ควบคุมเอง)
-- Enum.CameraType.Follow
-- Enum.CameraType.Fixed
-- Enum.CameraType.Attach
-- Enum.CameraType.Watch

-- ตำแหน่งและทิศทาง Camera
print("ตำแหน่ง Camera:", camera.CFrame.Position)
print("Camera มองไปทาง:", camera.CFrame.LookVector)

-- Field of View
print("FOV:", camera.FieldOfView)  -- default: 70
```

### 28.1.2 CameraType ต่างๆ

```lua
-- LocalScript
local camera = workspace.CurrentCamera

-- 1. Custom (ติดตาม Character)
camera.CameraType = Enum.CameraType.Custom

-- 2. Scriptable (ควบคุมเองทั้งหมด)
camera.CameraType = Enum.CameraType.Scriptable

-- 3. Follow (ติดตาม Subject)
camera.CameraType = Enum.CameraType.Follow
camera.CameraSubject = workspace.SomeObject  -- object ที่ติดตาม

-- 4. Fixed (อยู่กับที่)
camera.CameraType = Enum.CameraType.Fixed
camera.CFrame = CFrame.new(0, 50, 0) * CFrame.Angles(math.rad(-90), 0, 0)

-- 5. Attach (ติดกับ Part)
camera.CameraType = Enum.CameraType.Attach
camera.CameraSubject = workspace.SomePart

-- คืนค่าเป็น Default
camera.CameraType = Enum.CameraType.Custom
```

---

## 28.2 การเคลื่อนย้าย Camera

### 28.2.1 CFrame ของ Camera

```lua
-- LocalScript
local camera = workspace.CurrentCamera
local TweenService = game:GetService("TweenService")

-- ตั้งค่า Camera เป็น Scriptable
camera.CameraType = Enum.CameraType.Scriptable

-- ย้าย Camera ไปยังตำแหน่ง
camera.CFrame = CFrame.new(Vector3.new(0, 50, 0))

-- มองไปยังจุดหมาย
local targetPosition = Vector3.new(0, 0, 0)
camera.CFrame = CFrame.new(camera.CFrame.Position, targetPosition)

-- หมุน Camera
camera.CFrame = CFrame.new(0, 50, 0) * 
    CFrame.Angles(math.rad(-45), math.rad(45), 0)

-- Lerp ตำแหน่ง Camera (เคลื่อนที่แบบ smooth)
local startCFrame = camera.CFrame
local endCFrame = CFrame.new(100, 50, 100) * CFrame.Angles(math.rad(-30), 0, 0)
local duration = 2

local startTime = tick()
local RunService = game:GetService("RunService")

local connection
connection = RunService.RenderStepped:Connect(function()
    local elapsed = tick() - startTime
    local alpha = math.min(elapsed / duration, 1)
    
    -- Lerp
    camera.CFrame = startCFrame:Lerp(endCFrame, alpha)
    
    if alpha >= 1 then
        connection:Disconnect()
        print("Camera เคลื่อนที่เสร็จ!")
    end
end)
```

### 28.2.2 Camera Tween

```lua
-- LocalScript
local camera = workspace.CurrentCamera
local TweenService = game:GetService("TweenService")

camera.CameraType = Enum.CameraType.Scriptable

local function tweenCamera(targetCFrame, duration, style, direction)
    local tween = TweenService:Create(
        camera,
        TweenInfo.new(
            duration or 1,
            style or Enum.EasingStyle.Quad,
            direction or Enum.EasingDirection.Out
        ),
        {CFrame = targetCFrame}
    )
    tween:Play()
    return tween
end

-- ใช้งาน
local tween = tweenCamera(
    CFrame.new(50, 30, 50) * CFrame.Angles(math.rad(-30), math.rad(-45), 0),
    2,
    Enum.EasingStyle.Sine,
    Enum.EasingDirection.InOut
)

tween.Completed:Connect(function()
    print("Camera Tween เสร็จ!")
end)
```

---

## 28.3 Third Person Camera System

```lua
-- LocalScript: ThirdPersonCamera
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local LocalPlayer = Players.LocalPlayer
local camera = workspace.CurrentCamera

-- ค่า Camera
local cameraSettings = {
    distance = 15,        -- ระยะห่างจาก Character
    minDistance = 5,      -- ระยะใกล้สุด
    maxDistance = 50,     -- ระยะไกลสุด
    height = 5,           -- ความสูง
    smoothness = 0.1,     -- ความ smooth ในการเคลื่อนที่
    sensitivity = 0.3,    -- ความไวของ Mouse
    minVertAngle = -60,   -- มุมแนวตั้งต่ำสุด
    maxVertAngle = 80,    -- มุมแนวตั้งสูงสุด
}

local cameraX = 0  -- มุมแนวนอน
local cameraY = 20 -- มุมแนวตั้ง

-- Lock cursor
UserInputService.MouseBehavior = Enum.MouseBehavior.LockCenter

-- อัพเดต Camera
RunService.RenderStepped:Connect(function()
    local character = LocalPlayer.Character
    if not character then return end
    
    local hrp = character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    
    -- อ่าน Mouse Input
    local delta = UserInputService:GetMouseDelta()
    cameraX = cameraX - delta.X * cameraSettings.sensitivity
    cameraY = math.clamp(
        cameraY - delta.Y * cameraSettings.sensitivity,
        cameraSettings.minVertAngle,
        cameraSettings.maxVertAngle
    )
    
    -- คำนวณตำแหน่ง Camera
    local targetCFrame = CFrame.new(hrp.Position + Vector3.new(0, cameraSettings.height, 0)) *
        CFrame.Angles(0, math.rad(cameraX), 0) *
        CFrame.Angles(math.rad(cameraY), 0, 0)
    
    local offset = targetCFrame * CFrame.new(0, 0, cameraSettings.distance)
    
    -- ตรวจสอบสิ่งกีดขวาง (Raycast)
    local raycastParams = RaycastParams.new()
    raycastParams.FilterDescendantsInstances = {character}
    raycastParams.FilterType = Enum.RaycastFilterType.Exclude
    
    local ray = workspace:Raycast(
        hrp.Position + Vector3.new(0, cameraSettings.height, 0),
        offset.Position - (hrp.Position + Vector3.new(0, cameraSettings.height, 0)),
        raycastParams
    )
    
    local finalDistance = cameraSettings.distance
    if ray then
        finalDistance = math.max(1, ray.Distance - 0.5)
    end
    
    local finalOffset = targetCFrame * CFrame.new(0, 0, finalDistance)
    
    -- Smooth Camera
    camera.CFrame = camera.CFrame:Lerp(
        CFrame.new(finalOffset.Position, hrp.Position + Vector3.new(0, cameraSettings.height, 0)),
        cameraSettings.smoothness
    )
    
    -- หมุน Character ตาม Camera
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if humanoid and humanoid.MoveDirection.Magnitude > 0 then
        hrp.CFrame = CFrame.new(hrp.Position) * CFrame.Angles(0, math.rad(cameraX), 0)
    end
end)

-- Scroll Wheel สำหรับ Zoom
UserInputService.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseWheel then
        cameraSettings.distance = math.clamp(
            cameraSettings.distance - input.Position.Z * 2,
            cameraSettings.minDistance,
            cameraSettings.maxDistance
        )
    end
end)
```

---

## 28.4 First Person Camera

```lua
-- LocalScript: FirstPersonCamera
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local LocalPlayer = Players.LocalPlayer
local camera = workspace.CurrentCamera

local sensitivity = 0.3
local cameraX = 0
local cameraY = 0

-- ซ่อนตัวละคร (สำหรับ FPS)
local function setCharacterVisibility(character, visible)
    for _, part in ipairs(character:GetDescendants()) do
        if part:IsA("BasePart") or part:IsA("Decal") then
            if part.Name ~= "HumanoidRootPart" then
                part.LocalTransparencyModifier = visible and 0 or 1
            end
        end
    end
end

-- ตั้งค่า First Person
camera.CameraType = Enum.CameraType.Scriptable
UserInputService.MouseBehavior = Enum.MouseBehavior.LockCenter

RunService.RenderStepped:Connect(function()
    local character = LocalPlayer.Character
    if not character then return end
    
    local head = character:FindFirstChild("Head")
    local hrp = character:FindFirstChild("HumanoidRootPart")
    if not head or not hrp then return end
    
    -- ซ่อนตัวละคร
    setCharacterVisibility(character, false)
    
    -- อ่าน Mouse
    local delta = UserInputService:GetMouseDelta()
    cameraX = cameraX - delta.X * sensitivity
    cameraY = math.clamp(cameraY - delta.Y * sensitivity, -80, 80)
    
    -- ตั้งค่า Camera ที่ตำแหน่งหัว
    local headCFrame = head.CFrame
    local cameraCFrame = headCFrame * 
        CFrame.Angles(0, math.rad(cameraX - headCFrame:ToEulerAnglesYXZ() * (180/math.pi)), 0) *
        CFrame.Angles(math.rad(cameraY), 0, 0)
    
    -- คำนวณแบบง่ายกว่า
    camera.CFrame = CFrame.new(head.Position) *
        CFrame.Angles(0, math.rad(cameraX), 0) *
        CFrame.Angles(math.rad(cameraY), 0, 0)
    
    -- หมุน HRP ตาม Camera
    hrp.CFrame = CFrame.new(hrp.Position) * CFrame.Angles(0, math.rad(cameraX), 0)
end)
```

---

## 28.5 Cinematic Camera

```lua
-- LocalScript: CinematicCamera
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")

local camera = workspace.CurrentCamera

-- ระบบ Cinematic Camera
local CinematicCamera = {}

-- เส้นทาง Camera (Keyframes)
local keyframes = {
    {position = Vector3.new(0, 20, 50),  lookAt = Vector3.new(0, 0, 0),  duration = 3},
    {position = Vector3.new(50, 10, 0),  lookAt = Vector3.new(0, 5, 0),  duration = 4},
    {position = Vector3.new(0, 50, -30), lookAt = Vector3.new(0, 0, 0),  duration = 3},
    {position = Vector3.new(-50, 10, 0), lookAt = Vector3.new(0, 5, 0),  duration = 4},
}

function CinematicCamera.play(onFinished)
    camera.CameraType = Enum.CameraType.Scriptable
    
    -- ซ่อน GUI
    for _, gui in ipairs(game.Players.LocalPlayer.PlayerGui:GetChildren()) do
        if gui:IsA("ScreenGui") and gui.Name ~= "CinematicGui" then
            gui.Enabled = false
        end
    end
    
    local function playKeyframe(index)
        if index > #keyframes then
            -- จบแล้ว
            camera.CameraType = Enum.CameraType.Custom
            
            -- แสดง GUI กลับ
            for _, gui in ipairs(game.Players.LocalPlayer.PlayerGui:GetChildren()) do
                gui.Enabled = true
            end
            
            if onFinished then onFinished() end
            return
        end
        
        local kf = keyframes[index]
        local targetCFrame = CFrame.new(kf.position, kf.lookAt)
        
        local tween = TweenService:Create(
            camera,
            TweenInfo.new(kf.duration, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut),
            {CFrame = targetCFrame}
        )
        
        tween:Play()
        tween.Completed:Connect(function()
            playKeyframe(index + 1)
        end)
    end
    
    -- เริ่มต้นที่ keyframe แรก
    camera.CFrame = CFrame.new(keyframes[1].position, keyframes[1].lookAt)
    playKeyframe(2)
end

function CinematicCamera.stop()
    camera.CameraType = Enum.CameraType.Custom
end

-- เรียกใช้ Cinematic
CinematicCamera.play(function()
    print("Cinematic จบแล้ว!")
end)
```

---

## 28.6 Orbital Camera (สำหรับ Menu หรือ Showcase)

```lua
-- LocalScript: OrbitalCamera
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local camera = workspace.CurrentCamera
camera.CameraType = Enum.CameraType.Scriptable

local target = Vector3.new(0, 5, 0)  -- จุดที่ camera หมุนรอบ
local radius = 30       -- รัศมี
local height = 15       -- ความสูง
local speed = 20        -- ความเร็วหมุน (องศาต่อวินาที)
local angle = 0         -- มุมปัจจุบัน
local isAutoRotating = true

RunService.RenderStepped:Connect(function(dt)
    if isAutoRotating then
        angle = angle + speed * dt
    end
    
    -- คำนวณตำแหน่ง Camera
    local x = target.X + radius * math.cos(math.rad(angle))
    local z = target.Z + radius * math.sin(math.rad(angle))
    local y = target.Y + height
    
    camera.CFrame = CFrame.new(Vector3.new(x, y, z), target)
end)

-- ควบคุมด้วย Mouse Drag
local isDragging = false
local lastMouseX = 0

UserInputService.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        isDragging = true
        isAutoRotating = false
        lastMouseX = input.Position.X
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        isDragging = false
        isAutoRotating = true
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if isDragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position.X - lastMouseX
        angle = angle + delta * 0.5
        lastMouseX = input.Position.X
    end
    
    -- Zoom ด้วย Scroll Wheel
    if input.UserInputType == Enum.UserInputType.MouseWheel then
        radius = math.clamp(radius - input.Position.Z * 2, 10, 100)
    end
end)
```

---

## 28.7 Camera Shake Effect

```lua
-- LocalScript: CameraShake
local RunService = game:GetService("RunService")

local camera = workspace.CurrentCamera

local shakeSettings = {
    duration = 0,
    intensity = 0,
    frequency = 10,
}

local shakeTime = 0

local function startShake(duration, intensity, frequency)
    shakeSettings.duration = duration
    shakeSettings.intensity = intensity
    shakeSettings.frequency = frequency or 10
    shakeTime = tick()
end

-- อัพเดต Shake
local originalCFrame = camera.CFrame
RunService.RenderStepped:Connect(function()
    local elapsed = tick() - shakeTime
    
    if elapsed < shakeSettings.duration then
        local remaining = 1 - (elapsed / shakeSettings.duration)
        local currentIntensity = shakeSettings.intensity * remaining
        
        -- สร้าง Random shake
        local shakeX = (math.random() * 2 - 1) * currentIntensity
        local shakeY = (math.random() * 2 - 1) * currentIntensity
        local shakeZ = (math.random() * 2 - 1) * currentIntensity
        
        camera.CFrame = camera.CFrame * CFrame.new(shakeX, shakeY, shakeZ)
    end
end)

-- ตัวอย่าง: Shake เมื่อตก
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

LocalPlayer.CharacterAdded:Connect(function(character)
    local humanoid = character:WaitForChild("Humanoid")
    
    humanoid.StateChanged:Connect(function(_, newState)
        if newState == Enum.HumanoidStateType.Landed then
            -- Shake เมื่อลงจอด
            startShake(0.3, 0.5, 20)
        end
    end)
end)
```

---

## 28.8 Top-Down Camera

```lua
-- LocalScript: TopDownCamera
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local LocalPlayer = Players.LocalPlayer
local camera = workspace.CurrentCamera

-- ตั้งค่า Top Down
camera.CameraType = Enum.CameraType.Scriptable

local cameraHeight = 50
local cameraAngle = -70  -- มุมเงย
local cameraSmooth = 0.15

RunService.RenderStepped:Connect(function()
    local character = LocalPlayer.Character
    if not character then return end
    
    local hrp = character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    
    -- ตำแหน่ง Camera อยู่เหนือตัวละคร
    local targetPosition = hrp.Position + Vector3.new(0, cameraHeight, 0)
    local targetCFrame = CFrame.new(targetPosition) * 
        CFrame.Angles(math.rad(cameraAngle), 0, 0)
    
    -- Smooth Follow
    camera.CFrame = camera.CFrame:Lerp(targetCFrame, cameraSmooth)
end)

-- ปิด Default Camera Controls
local StarterPlayerScripts = game:GetService("StarterPlayer")
-- ใน Studio: StarterPlayer > DisableCharacterAutoLoads = false

-- ตรวจสอบ Scroll Wheel สำหรับ Zoom
UserInputService.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseWheel then
        cameraHeight = math.clamp(
            cameraHeight - input.Position.Z * 5,
            20, 150
        )
    end
end)
```

---

## 28.9 Split Screen Camera

```lua
-- LocalScript: SplitScreen (สำหรับ 2 ผู้เล่น Local)
-- หมายเหตุ: Roblox ไม่รองรับ Split Screen แบบ Native
-- แต่เราสามารถทำ Fake Split Screen ได้

local camera = workspace.CurrentCamera
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

-- สร้าง Viewport สำหรับ Player 2
local LocalPlayer = Players.LocalPlayer
local screenGui = Instance.new("ScreenGui")
screenGui.Parent = LocalPlayer.PlayerGui

-- Viewport Frame สำหรับแสดง View ของ Player 2
local viewport = Instance.new("ViewportFrame")
viewport.Size = UDim2.new(0.5, 0, 1, 0)
viewport.Position = UDim2.new(0.5, 0, 0, 0)
viewport.BackgroundColor3 = Color3.new(0, 0, 0)
viewport.Parent = screenGui

-- Camera ใน Viewport
local viewportCamera = Instance.new("Camera")
viewportCamera.Parent = viewport
viewport.CurrentCamera = viewportCamera

-- ตัวแบ่ง
local divider = Instance.new("Frame")
divider.Size = UDim2.new(0, 4, 1, 0)
divider.Position = UDim2.new(0.5, -2, 0, 0)
divider.BackgroundColor3 = Color3.new(1, 1, 1)
divider.BorderSizePixel = 0
divider.Parent = screenGui

-- อัพเดต Viewport Camera
RunService.RenderStepped:Connect(function()
    -- ตามผู้เล่นคนที่ 2 (ถ้ามี)
    local players = Players:GetPlayers()
    for _, player in ipairs(players) do
        if player ~= LocalPlayer and player.Character then
            local hrp = player.Character:FindFirstChild("HumanoidRootPart")
            if hrp then
                viewportCamera.CFrame = CFrame.new(
                    hrp.Position + Vector3.new(0, 10, 20),
                    hrp.Position
                )
            end
        end
    end
end)
```

---

## 28.10 Camera Field of View Effects

```lua
-- LocalScript: FOV Effects
local TweenService = game:GetService("TweenService")
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer
local camera = workspace.CurrentCamera

local normalFOV = 70
local sprintFOV = 90

-- FOV Tween เมื่อ Sprint
local function setFOV(targetFOV, duration)
    local tween = TweenService:Create(
        camera,
        TweenInfo.new(duration or 0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
        {FieldOfView = targetFOV}
    )
    tween:Play()
end

-- ตรวจสอบการ Sprint
LocalPlayer.CharacterAdded:Connect(function(character)
    local humanoid = character:WaitForChild("Humanoid")
    
    RunService.Heartbeat:Connect(function()
        if humanoid.WalkSpeed > 20 then
            -- กำลัง Sprint
            if math.abs(camera.FieldOfView - sprintFOV) > 1 then
                setFOV(sprintFOV, 0.3)
            end
        else
            -- ปกติ
            if math.abs(camera.FieldOfView - normalFOV) > 1 then
                setFOV(normalFOV, 0.5)
            end
        end
    end)
end)
```

---

## 28.11 Cutscene System

```lua
-- LocalScript: CutsceneSystem
local TweenService = game:GetService("TweenService")
local Players = game:GetService("Players")

local LocalPlayer = Players.LocalPlayer
local camera = workspace.CurrentCamera

local CutsceneSystem = {}

-- สร้าง Cutscene
function CutsceneSystem.create(shots)
    return {
        shots = shots,
        isPlaying = false
    }
end

-- เล่น Cutscene
function CutsceneSystem.play(cutscene, onFinished)
    if cutscene.isPlaying then return end
    cutscene.isPlaying = true
    
    -- เปลี่ยน Camera Mode
    camera.CameraType = Enum.CameraType.Scriptable
    
    -- ซ่อน Core GUI
    game:GetService("StarterGui"):SetCoreGuiEnabled(
        Enum.CoreGuiType.All, false
    )
    
    -- เล่นทีละ Shot
    local function playShot(index)
        if index > #cutscene.shots then
            -- จบ Cutscene
            cutscene.isPlaying = false
            camera.CameraType = Enum.CameraType.Custom
            
            -- แสดง GUI กลับ
            game:GetService("StarterGui"):SetCoreGuiEnabled(
                Enum.CoreGuiType.All, true
            )
            
            if onFinished then onFinished() end
            return
        end
        
        local shot = cutscene.shots[index]
        
        -- ย้าย Camera ไปตำแหน่งเริ่มต้น (ทันที)
        if shot.startCFrame then
            camera.CFrame = shot.startCFrame
        end
        
        -- Tween ไปยังตำแหน่งสิ้นสุด
        if shot.endCFrame then
            local tween = TweenService:Create(
                camera,
                TweenInfo.new(
                    shot.duration or 2,
                    shot.style or Enum.EasingStyle.Linear,
                    shot.direction or Enum.EasingDirection.InOut
                ),
                {CFrame = shot.endCFrame}
            )
            tween:Play()
            tween.Completed:Connect(function()
                wait(shot.holdTime or 0)
                playShot(index + 1)
            end)
        else
            wait(shot.duration or 2)
            playShot(index + 1)
        end
    end
    
    playShot(1)
end

-- ตัวอย่าง Cutscene
local introCutscene = CutsceneSystem.create({
    {
        startCFrame = CFrame.new(0, 100, 200) * CFrame.Angles(math.rad(-30), 0, 0),
        endCFrame = CFrame.new(0, 50, 100) * CFrame.Angles(math.rad(-20), 0, 0),
        duration = 3,
        holdTime = 0.5,
        style = Enum.EasingStyle.Sine
    },
    {
        startCFrame = CFrame.new(-80, 20, 0) * CFrame.Angles(math.rad(-10), math.rad(90), 0),
        endCFrame = CFrame.new(-20, 10, 0) * CFrame.Angles(math.rad(-5), math.rad(45), 0),
        duration = 2.5,
        style = Enum.EasingStyle.Quad
    },
    {
        startCFrame = CFrame.new(0, 5, 30) * CFrame.Angles(math.rad(-5), 0, 0),
        endCFrame = CFrame.new(0, 5, 10) * CFrame.Angles(math.rad(-5), 0, 0),
        duration = 2,
        holdTime = 1
    },
})

-- เล่น Cutscene เมื่อเริ่มเกม
CutsceneSystem.play(introCutscene, function()
    print("Intro Cutscene จบแล้ว!")
end)
```

---

## 28.12 Camera Offset

```lua
-- LocalScript: CameraOffset
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer

-- Camera Offset ของ Character
LocalPlayer.CharacterAdded:Connect(function(character)
    local humanoid = character:WaitForChild("Humanoid")
    
    -- เลื่อน Camera ออกจากตัวละครไปทางขวา (อย่าง Resident Evil)
    humanoid.CameraOffset = Vector3.new(2, 0, 0)
    
    -- หรือเลื่อนขึ้น (เพื่อให้เห็นพื้นที่มากขึ้น)
    -- humanoid.CameraOffset = Vector3.new(0, 3, 0)
end)
```

---

## 28.13 Camera Effects (Post-Processing)

```lua
-- LocalScript: CameraEffects
local Players = game:GetService("Players")
local Lighting = game:GetService("Lighting")

-- Blur Effect
local blur = Instance.new("BlurEffect")
blur.Size = 0
blur.Parent = workspace.CurrentCamera

-- Color Correction
local colorCorrect = Instance.new("ColorCorrectionEffect")
colorCorrect.Brightness = 0
colorCorrect.Contrast = 0
colorCorrect.Saturation = 0
colorCorrect.TintColor = Color3.new(1, 1, 1)
colorCorrect.Parent = workspace.CurrentCamera

-- Bloom
local bloom = Instance.new("BloomEffect")
bloom.Intensity = 0.5
bloom.Size = 24
bloom.Threshold = 2
bloom.Parent = workspace.CurrentCamera

-- Sun Rays
local sunRays = Instance.new("SunRaysEffect")
sunRays.Intensity = 0.3
sunRays.Spread = 0.5
sunRays.Parent = Lighting

-- ฟังก์ชันเปลี่ยน Effect เมื่อ HP ต่ำ
local function updateHealthEffect(health, maxHealth)
    local healthPercent = health / maxHealth
    
    if healthPercent < 0.3 then
        -- HP น้อย: เพิ่ม Effect
        local intensity = 1 - healthPercent / 0.3
        
        colorCorrect.Saturation = -intensity * 0.8   -- ลดสี
        colorCorrect.TintColor = Color3.new(1, 0.5, 0.5)  -- โทนแดง
        blur.Size = intensity * 10
    else
        -- HP ปกติ
        colorCorrect.Saturation = 0
        colorCorrect.TintColor = Color3.new(1, 1, 1)
        blur.Size = 0
    end
end

-- ตัวอย่างการใช้
local LocalPlayer = Players.LocalPlayer
LocalPlayer.CharacterAdded:Connect(function(character)
    local humanoid = character:WaitForChild("Humanoid")
    humanoid.HealthChanged:Connect(function(health)
        updateHealthEffect(health, humanoid.MaxHealth)
    end)
end)
```

---

## 28.14 แบบฝึกหัด

### แบบฝึกหัดที่ 1: Isometric Camera
สร้าง Isometric Camera (มุมมองแบบเกม RTS)

```lua
-- LocalScript
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local LocalPlayer = Players.LocalPlayer
local camera = workspace.CurrentCamera

camera.CameraType = Enum.CameraType.Scriptable

-- ค่า Isometric
local isoAngle = 45     -- มุมแนวนอน
local elevation = 45    -- ความสูง
local distance = 40     -- ระยะ

local panSpeed = 0.5
local panX = 0
local panZ = 0

RunService.RenderStepped:Connect(function(dt)
    local character = LocalPlayer.Character
    if not character then return end
    
    local hrp = character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    
    -- Isometric offset
    local rad = math.rad(isoAngle)
    local elevRad = math.rad(elevation)
    
    local x = hrp.Position.X + distance * math.cos(elevRad) * math.sin(rad)
    local y = hrp.Position.Y + distance * math.sin(elevRad)
    local z = hrp.Position.Z + distance * math.cos(elevRad) * math.cos(rad)
    
    camera.CFrame = CFrame.new(Vector3.new(x, y, z), hrp.Position)
end)

print("Isometric Camera พร้อมใช้งาน!")
```

---

## 28.15 สรุป

ในบทนี้เราได้เรียนรู้:

1. **Camera Basics** - CameraType ต่างๆ
2. **Camera Movement** - CFrame, Lerp, Tween
3. **Third Person Camera** - Camera ติดตามตัวละครแบบ 3rd person
4. **First Person Camera** - มุมมอง FPS
5. **Cinematic Camera** - ระบบ Cutscene
6. **Orbital Camera** - Camera หมุนรอบ Object
7. **Camera Shake** - เขย่า Camera
8. **Top-Down Camera** - มุมมองจากด้านบน
9. **FOV Effects** - เปลี่ยน FOV แบบ Dynamic
10. **Post-Processing Effects** - Blur, Color Correction, Bloom
11. **Camera Offset** - เลื่อน Camera

ในบทต่อไป เราจะเรียนรู้เกี่ยวกับ GUI Basics

---

## แหล่งอ้างอิง

- [Roblox Developer Hub - Camera](https://developer.roblox.com/en-us/api-reference/class/Camera)
- [Roblox Developer Hub - TweenService](https://developer.roblox.com/en-us/api-reference/class/TweenService)
- [Roblox Camera Types](https://developer.roblox.com/en-us/articles/camera-manipulation)
