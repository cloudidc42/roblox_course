# ตอนที่ 34: TweenService - การทำ Animation

## บทนำ

TweenService เป็น Service ที่ทรงพลังมากใน Roblox ใช้สำหรับทำ Animation แบบ Smooth ไม่ว่าจะเป็น GUI, Parts, หรือ Properties อื่นๆ ใน Tween เราสามารถกำหนดเวลา, รูปแบบการ Ease, และค่าเริ่มต้น-สิ้นสุดได้

---

## 34.1 พื้นฐาน TweenService

```lua
-- LocalScript / Script
local TweenService = game:GetService("TweenService")

-- Tween ต้องการ:
-- 1. Instance ที่จะ Animate
-- 2. TweenInfo (ข้อมูลการ Animate)
-- 3. Goals (ค่าที่ต้องการ)

local part = workspace.TestPart

-- TweenInfo.new(
--   duration,     -- ระยะเวลา (วินาที)
--   easingStyle,  -- สไตล์การเคลื่อนที่
--   easingDirection, -- ทิศทาง Ease
--   repeatCount,  -- จำนวนการทำซ้ำ (-1 = ไม่หยุด)
--   reverses,     -- กลับด้านหรือเปล่า
--   delayTime     -- รอก่อนเริ่ม (วินาที)
-- )

local tweenInfo = TweenInfo.new(
    2,                              -- 2 วินาที
    Enum.EasingStyle.Quad,          -- Quad Easing
    Enum.EasingDirection.Out,        -- Ease Out
    0,                              -- ไม่ทำซ้ำ
    false,                          -- ไม่กลับด้าน
    0                               -- ไม่รอ
)

-- Goals
local goals = {
    Position = Vector3.new(10, 5, 0),
    BrickColor = BrickColor.new("Bright red"),
    Size = Vector3.new(5, 5, 5),
}

-- สร้าง Tween
local tween = TweenService:Create(part, tweenInfo, goals)

-- เล่น Tween
tween:Play()

-- รอให้เสร็จ
tween.Completed:Connect(function(playbackState)
    print("Tween เสร็จ! State:", playbackState)
    -- Enum.PlaybackState.Completed - เสร็จปกติ
    -- Enum.PlaybackState.Cancelled  - ถูกยกเลิก
end)

-- หยุด Tween
-- tween:Cancel()

-- Pause Tween
-- tween:Pause()

-- Resume หลัง Pause
-- tween:Play()
```

---

## 34.2 EasingStyle ต่างๆ

```lua
-- EasingStyle กำหนดรูปแบบการเคลื่อนที่
-- EasingDirection กำหนดว่า Ease จะอยู่ที่จุดไหน

--[[
EasingStyle:
  Linear   - ความเร็วคงที่
  Sine     - โค้งแบบ sin
  Back     - ดีดกลับนิดนึง
  Bounce   - กระเด้ง
  Elastic  - ยืดหยุ่น
  Quad     - กำลัง 2
  Quart    - กำลัง 4
  Quint    - กำลัง 5
  Cubic    - กำลัง 3
  Exponential - ชี้กำลัง
  Circular - วงกลม

EasingDirection:
  In    - Ease ที่จุดเริ่มต้น (เริ่มช้า เร็วขึ้น)
  Out   - Ease ที่จุดสิ้นสุด (เริ่มเร็ว ช้าลง)
  InOut - Ease ทั้งสองจุด
]]

local TweenService = game:GetService("TweenService")

-- ตัวอย่างทุก EasingStyle
local styles = {
    Enum.EasingStyle.Linear,
    Enum.EasingStyle.Sine,
    Enum.EasingStyle.Back,
    Enum.EasingStyle.Bounce,
    Enum.EasingStyle.Elastic,
    Enum.EasingStyle.Quad,
}

local function demonstrateStyle(style, position)
    local part = Instance.new("Part")
    part.Size = Vector3.new(1, 1, 1)
    part.Position = position
    part.Anchored = true
    part.Parent = workspace
    
    local tween = TweenService:Create(
        part,
        TweenInfo.new(2, style, Enum.EasingDirection.Out),
        {Position = position + Vector3.new(0, 10, 0)}
    )
    tween:Play()
end
```

---

## 34.3 GUI Animation

```lua
-- LocalScript: GUI Animations
local TweenService = game:GetService("TweenService")
local Players = game:GetService("Players")

local screenGui = Instance.new("ScreenGui")
screenGui.Parent = Players.LocalPlayer.PlayerGui

local panel = Instance.new("Frame")
panel.Size = UDim2.new(0, 300, 0, 200)
panel.AnchorPoint = Vector2.new(0.5, 0.5)
panel.Position = UDim2.new(0.5, 0, -0.5, 0)  -- เริ่มนอกจอบน
panel.BackgroundColor3 = Color3.fromRGB(50, 50, 70)
panel.BorderSizePixel = 0
panel.Parent = screenGui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 12)
corner.Parent = panel

-- Slide Down
local function slideDown()
    return TweenService:Create(
        panel,
        TweenInfo.new(0.5, Enum.EasingStyle.Back, Enum.EasingDirection.Out),
        {Position = UDim2.new(0.5, 0, 0.5, 0)}
    )
end

-- Slide Up (ออกจอ)
local function slideUp()
    return TweenService:Create(
        panel,
        TweenInfo.new(0.5, Enum.EasingStyle.Back, Enum.EasingDirection.In),
        {Position = UDim2.new(0.5, 0, -0.5, 0)}
    )
end

-- Scale In
local function scaleIn()
    panel.Size = UDim2.new(0, 0, 0, 0)
    panel.Position = UDim2.new(0.5, 0, 0.5, 0)
    return TweenService:Create(
        panel,
        TweenInfo.new(0.4, Enum.EasingStyle.Back, Enum.EasingDirection.Out),
        {Size = UDim2.new(0, 300, 0, 200)}
    )
end

-- Fade In
local function fadeIn()
    panel.BackgroundTransparency = 1
    return TweenService:Create(
        panel,
        TweenInfo.new(0.3),
        {BackgroundTransparency = 0}
    )
end

-- ลอง Animation
task.spawn(function()
    wait(0.5)
    scaleIn():Play()
end)
```

---

## 34.4 Animation ของ Parts

```lua
-- Script: Part Animations
local TweenService = game:GetService("TweenService")

-- Floating Animation
local function makeFloat(part, height, speed)
    height = height or 2
    speed = speed or 2
    
    local startY = part.Position.Y
    
    -- ขึ้น
    local upTween = TweenService:Create(
        part,
        TweenInfo.new(speed, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut),
        {Position = part.Position + Vector3.new(0, height, 0)}
    )
    
    -- ลง
    local downTween = TweenService:Create(
        part,
        TweenInfo.new(speed, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut),
        {Position = part.Position - Vector3.new(0, height/2, 0)}
    )
    
    -- วนซ้ำ
    local function cycle()
        upTween:Play()
        upTween.Completed:Connect(function()
            downTween:Play()
        end)
        downTween.Completed:Connect(function()
            upTween:Play()
        end)
    end
    
    cycle()
end

-- Rotation Animation
local function makeRotate(part, speed, axis)
    axis = axis or "Y"
    speed = speed or 90  -- degrees per second
    
    local RunService = game:GetService("RunService")
    local conn
    conn = RunService.Heartbeat:Connect(function(dt)
        if not part.Parent then
            conn:Disconnect()
            return
        end
        
        if axis == "X" then
            part.CFrame = part.CFrame * CFrame.Angles(math.rad(speed * dt), 0, 0)
        elseif axis == "Y" then
            part.CFrame = part.CFrame * CFrame.Angles(0, math.rad(speed * dt), 0)
        elseif axis == "Z" then
            part.CFrame = part.CFrame * CFrame.Angles(0, 0, math.rad(speed * dt))
        end
    end)
    
    return conn
end

-- Bounce Animation
local function makeBounce(part, bounceHeight, speed)
    bounceHeight = bounceHeight or 5
    speed = speed or 0.5
    
    local groundY = part.Position.Y
    
    local function doBounce()
        -- Naik
        local upTween = TweenService:Create(
            part,
            TweenInfo.new(speed, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
            {Position = Vector3.new(part.Position.X, groundY + bounceHeight, part.Position.Z)}
        )
        
        -- ลง (แบบ Bounce)
        local downTween = TweenService:Create(
            part,
            TweenInfo.new(speed, Enum.EasingStyle.Bounce, Enum.EasingDirection.Out),
            {Position = Vector3.new(part.Position.X, groundY, part.Position.Z)}
        )
        
        upTween:Play()
        upTween.Completed:Connect(function()
            downTween:Play()
            downTween.Completed:Connect(function()
                wait(0.5)
                doBounce()
            end)
        end)
    end
    
    doBounce()
end

-- ตัวอย่าง
local testPart = workspace.TestPart
makeFloat(testPart)

local spinPart = workspace.SpinPart
makeRotate(spinPart, 180, "Y")
```

---

## 34.5 Color Animations

```lua
-- LocalScript: Color Animations
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")

-- Tween สี (รองรับ Color3)
local part = workspace.TestPart

-- เปลี่ยนสีแบบ Tween
TweenService:Create(part, TweenInfo.new(2), {
    Color = Color3.fromRGB(255, 100, 100)
}):Play()

-- Color Cycle
local function colorCycle(instance, property, speed, saturation, value)
    saturation = saturation or 0.8
    value = value or 1.0
    speed = speed or 0.5
    
    local hue = 0
    RunService.Heartbeat:Connect(function(dt)
        hue = (hue + speed * dt) % 1
        local color = Color3.fromHSV(hue, saturation, value)
        instance[property] = color
    end)
end

-- ตัวอย่าง
local label = Instance.new("TextLabel")
label.Text = "สีสันสดใส!"
label.Parent = screenGui

colorCycle(label, "TextColor3", 0.3)

-- Gradient Tween (ใช้ UIGradient)
local frame = Instance.new("Frame")
frame.BackgroundColor3 = Color3.new(1, 1, 1)
frame.Parent = screenGui

local gradient = Instance.new("UIGradient")
gradient.Parent = frame

-- Animate Gradient Offset
TweenService:Create(gradient, TweenInfo.new(3, Enum.EasingStyle.Linear, 
    Enum.EasingDirection.InOut, -1, true), {
    Offset = Vector2.new(1, 0)
}):Play()
```

---

## 34.6 Chained Animations (Sequence)

```lua
-- LocalScript: Animation Sequence
local TweenService = game:GetService("TweenService")
local Players = game:GetService("Players")

local screenGui = Instance.new("ScreenGui")
screenGui.Parent = Players.LocalPlayer.PlayerGui

-- ระบบ Animation Sequence
local AnimSequence = {}
AnimSequence.__index = AnimSequence

function AnimSequence.new()
    return setmetatable({
        steps = {},
        isPlaying = false
    }, AnimSequence)
end

function AnimSequence:addTween(instance, tweenInfo, goals)
    table.insert(self.steps, {
        type = "tween",
        instance = instance,
        tweenInfo = tweenInfo,
        goals = goals
    })
    return self
end

function AnimSequence:addWait(duration)
    table.insert(self.steps, {
        type = "wait",
        duration = duration
    })
    return self
end

function AnimSequence:addCallback(func)
    table.insert(self.steps, {
        type = "callback",
        func = func
    })
    return self
end

function AnimSequence:play(onComplete)
    if self.isPlaying then return end
    self.isPlaying = true
    
    task.spawn(function()
        for _, step in ipairs(self.steps) do
            if step.type == "tween" then
                local tween = TweenService:Create(step.instance, step.tweenInfo, step.goals)
                tween:Play()
                tween.Completed:Wait()
            elseif step.type == "wait" then
                task.wait(step.duration)
            elseif step.type == "callback" then
                step.func()
            end
        end
        
        self.isPlaying = false
        if onComplete then onComplete() end
    end)
end

-- ตัวอย่าง
local box = Instance.new("Frame")
box.Size = UDim2.new(0, 100, 0, 100)
box.AnchorPoint = Vector2.new(0.5, 0.5)
box.Position = UDim2.new(0.1, 0, 0.5, 0)
box.BackgroundColor3 = Color3.fromRGB(100, 100, 200)
box.BorderSizePixel = 0
box.Parent = screenGui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 10)
corner.Parent = box

local seq = AnimSequence.new()
    :addTween(box, TweenInfo.new(0.5, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
        Size = UDim2.new(0, 150, 0, 150)
    })
    :addWait(0.3)
    :addTween(box, TweenInfo.new(1, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut), {
        Position = UDim2.new(0.9, 0, 0.5, 0),
        BackgroundColor3 = Color3.fromRGB(200, 100, 100)
    })
    :addCallback(function()
        print("กลางทาง!")
    end)
    :addTween(box, TweenInfo.new(0.5, Enum.EasingStyle.Back, Enum.EasingDirection.In), {
        Size = UDim2.new(0, 100, 0, 100)
    })
    :addTween(box, TweenInfo.new(1, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut), {
        Position = UDim2.new(0.5, 0, 0.5, 0),
        BackgroundColor3 = Color3.fromRGB(100, 100, 200)
    })

seq:play(function()
    print("Animation Sequence เสร็จ!")
end)
```

---

## 34.7 Parallel Animations

```lua
-- LocalScript: เล่น Animation พร้อมกัน
local TweenService = game:GetService("TweenService")

-- เล่นหลาย Tween พร้อมกัน
local function playParallel(tweens, onComplete)
    local count = #tweens
    local completed = 0
    
    for _, tween in ipairs(tweens) do
        tween.Completed:Connect(function()
            completed = completed + 1
            if completed >= count and onComplete then
                onComplete()
            end
        end)
        tween:Play()
    end
end

-- ตัวอย่าง: Animate หลาย Elements พร้อมกัน
local frame1 = Instance.new("Frame")
frame1.Size = UDim2.new(0, 100, 0, 100)
frame1.Position = UDim2.new(0.2, 0, 0.5, 0)
frame1.BackgroundColor3 = Color3.fromRGB(255, 100, 100)
frame1.Parent = screenGui

local frame2 = Instance.new("Frame")
frame2.Size = UDim2.new(0, 100, 0, 100)
frame2.Position = UDim2.new(0.5, 0, 0.5, 0)
frame2.BackgroundColor3 = Color3.fromRGB(100, 255, 100)
frame2.Parent = screenGui

local frame3 = Instance.new("Frame")
frame3.Size = UDim2.new(0, 100, 0, 100)
frame3.Position = UDim2.new(0.8, 0, 0.5, 0)
frame3.BackgroundColor3 = Color3.fromRGB(100, 100, 255)
frame3.Parent = screenGui

local t1 = TweenService:Create(frame1, TweenInfo.new(1, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
    Position = UDim2.new(0.2, 0, 0.2, 0),
    Rotation = 45
})

local t2 = TweenService:Create(frame2, TweenInfo.new(1.5, Enum.EasingStyle.Elastic, Enum.EasingDirection.Out), {
    Size = UDim2.new(0, 150, 0, 150),
    BackgroundColor3 = Color3.fromRGB(255, 200, 0)
})

local t3 = TweenService:Create(frame3, TweenInfo.new(0.8, Enum.EasingStyle.Bounce, Enum.EasingDirection.Out), {
    Position = UDim2.new(0.8, 0, 0.8, 0)
})

playParallel({t1, t2, t3}, function()
    print("ทุก Animation เสร็จแล้ว!")
end)
```

---

## 34.8 Spring Animation (Custom)

```lua
-- LocalScript: Spring Physics Animation
local RunService = game:GetService("RunService")

-- Spring Class
local Spring = {}
Spring.__index = Spring

function Spring.new(initial, damping, frequency)
    return setmetatable({
        target = initial,
        position = initial,
        velocity = 0,
        damping = damping or 1,
        frequency = frequency or 10,
    }, Spring)
end

function Spring:update(dt)
    local spring_constant = self.frequency * self.frequency
    local acceleration = spring_constant * (self.target - self.position) - 
                        2 * self.damping * self.frequency * self.velocity
    
    self.velocity = self.velocity + acceleration * dt
    self.position = self.position + self.velocity * dt
    
    return self.position
end

-- Vector3 Spring
local function createVector3Spring(initial, damping, frequency)
    return {
        x = Spring.new(initial.X, damping, frequency),
        y = Spring.new(initial.Y, damping, frequency),
        z = Spring.new(initial.Z, damping, frequency),
        target = initial,
    }
end

local function updateVector3Spring(spring, dt)
    spring.x.target = spring.target.X
    spring.y.target = spring.target.Y
    spring.z.target = spring.target.Z
    
    return Vector3.new(
        spring.x:update(dt),
        spring.y:update(dt),
        spring.z:update(dt)
    )
end

-- ตัวอย่าง: Spring Camera
local cameraSpring = createVector3Spring(Vector3.new(0, 10, 20), 0.8, 8)
local camera = workspace.CurrentCamera

RunService.RenderStepped:Connect(function(dt)
    -- ตั้ง Target
    local character = game.Players.LocalPlayer.Character
    if character then
        local hrp = character:FindFirstChild("HumanoidRootPart")
        if hrp then
            cameraSpring.target = hrp.Position + Vector3.new(0, 5, 15)
        end
    end
    
    -- อัพเดต Spring
    local position = updateVector3Spring(cameraSpring, dt)
    -- camera.CFrame = CFrame.new(position, cameraSpring.target)
end)
```

---

## 34.9 Loading Animation

```lua
-- LocalScript: Loading Animations
local TweenService = game:GetService("TweenService")
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local screenGui = Instance.new("ScreenGui")
screenGui.Parent = Players.LocalPlayer.PlayerGui

-- 1. Spinning Loader
local function createSpinner(parent, size, color)
    local container = Instance.new("Frame")
    container.Size = UDim2.new(0, size, 0, size)
    container.AnchorPoint = Vector2.new(0.5, 0.5)
    container.BackgroundTransparency = 1
    container.Parent = parent
    
    -- วงกลมหลัก
    local ring = Instance.new("ImageLabel")
    ring.Size = UDim2.new(1, 0, 1, 0)
    ring.BackgroundTransparency = 1
    ring.Image = "rbxasset://textures/loading/robloxTilt.png"
    ring.ImageColor3 = color or Color3.fromRGB(100, 150, 255)
    ring.Parent = container
    
    -- หมุน
    local rotation = 0
    RunService.Heartbeat:Connect(function(dt)
        rotation = rotation + 360 * dt
        ring.Rotation = rotation
    end)
    
    return container
end

-- 2. Dot Loader
local function createDotLoader(parent, dotCount, color)
    dotCount = dotCount or 3
    color = color or Color3.fromRGB(100, 150, 255)
    
    local container = Instance.new("Frame")
    container.Size = UDim2.new(0, dotCount * 25, 0, 20)
    container.BackgroundTransparency = 1
    container.Parent = parent
    
    local dots = {}
    for i = 1, dotCount do
        local dot = Instance.new("Frame")
        dot.Size = UDim2.new(0, 15, 0, 15)
        dot.Position = UDim2.new(0, (i-1) * 25, 0.5, -7)
        dot.AnchorPoint = Vector2.new(0, 0.5)
        dot.BackgroundColor3 = color
        dot.BorderSizePixel = 0
        dot.Parent = container
        
        local dotCorner = Instance.new("UICorner")
        dotCorner.CornerRadius = UDim.new(0.5, 0)
        dotCorner.Parent = dot
        
        dots[i] = dot
    end
    
    -- Animate dots
    local function animateDot(dot, delay)
        task.delay(delay, function()
            while dot.Parent do
                TweenService:Create(dot, TweenInfo.new(0.3, Enum.EasingStyle.Sine), {
                    Position = UDim2.new(0, dot.Position.X.Offset, 0.5, -14),
                    BackgroundTransparency = 0.3
                }):Play()
                task.wait(0.3)
                TweenService:Create(dot, TweenInfo.new(0.3, Enum.EasingStyle.Sine), {
                    Position = UDim2.new(0, dot.Position.X.Offset, 0.5, -7),
                    BackgroundTransparency = 0
                }):Play()
                task.wait(0.3 + 0.2 * dotCount)
            end
        end)
    end
    
    for i, dot in ipairs(dots) do
        animateDot(dot, (i-1) * 0.2)
    end
    
    return container
end

-- 3. Progress Bar Loader
local function createProgressBar(parent, width, color)
    local container = Instance.new("Frame")
    container.Size = UDim2.new(0, width, 0, 8)
    container.BackgroundColor3 = Color3.fromRGB(40, 40, 60)
    container.BorderSizePixel = 0
    container.Parent = parent
    
    local barCorner = Instance.new("UICorner")
    barCorner.CornerRadius = UDim.new(0, 4)
    barCorner.Parent = container
    
    local fill = Instance.new("Frame")
    fill.Size = UDim2.new(0, 0, 1, 0)
    fill.BackgroundColor3 = color or Color3.fromRGB(100, 150, 255)
    fill.BorderSizePixel = 0
    fill.Parent = container
    
    local fillCorner = Instance.new("UICorner")
    fillCorner.CornerRadius = UDim.new(0, 4)
    fillCorner.Parent = fill
    
    -- Shimmer Effect
    local shimmer = Instance.new("Frame")
    shimmer.Size = UDim2.new(0.3, 0, 1, 0)
    shimmer.BackgroundColor3 = Color3.new(1, 1, 1)
    shimmer.BackgroundTransparency = 0.7
    shimmer.BorderSizePixel = 0
    shimmer.ClipsDescendants = false
    shimmer.Parent = fill
    
    -- หน้าที่: ตั้งค่า Progress
    local function setProgress(pct)
        TweenService:Create(fill, TweenInfo.new(0.3), {
            Size = UDim2.new(pct, 0, 1, 0)
        }):Play()
    end
    
    -- Shimmer Animation
    task.spawn(function()
        while container.Parent do
            shimmer.Position = UDim2.new(-0.3, 0, 0, 0)
            TweenService:Create(shimmer, TweenInfo.new(1, Enum.EasingStyle.Linear), {
                Position = UDim2.new(1.3, 0, 0, 0)
            }):Play()
            task.wait(1.5)
        end
    end)
    
    return container, setProgress
end

-- ตัวอย่าง
local bar, setProgress = createProgressBar(screenGui, 300, Color3.fromRGB(100, 200, 100))
bar.Position = UDim2.new(0.5, -150, 0.8, 0)

-- แสดงความก้าวหน้า
task.spawn(function()
    for i = 0, 100, 5 do
        setProgress(i/100)
        task.wait(0.1)
    end
end)
```

---

## 34.10 แบบฝึกหัด

### แบบฝึกหัดที่ 1: Animated Menu

```lua
-- LocalScript: Animated Menu
local TweenService = game:GetService("TweenService")
local Players = game:GetService("Players")

local LocalPlayer = Players.LocalPlayer

local screenGui = Instance.new("ScreenGui")
screenGui.Parent = LocalPlayer.PlayerGui

-- Menu Items
local menuItems = {
    {text = "▶ เล่นเกม", color = Color3.fromRGB(80, 200, 80)},
    {text = "👥 เลือกตัวละคร", color = Color3.fromRGB(80, 130, 220)},
    {text = "⚙️ ตั้งค่า", color = Color3.fromRGB(150, 150, 220)},
    {text = "ℹ️ เกี่ยวกับ", color = Color3.fromRGB(200, 150, 80)},
    {text = "✖ ออก", color = Color3.fromRGB(220, 80, 80)},
}

-- Title
local title = Instance.new("TextLabel")
title.Size = UDim2.new(0, 400, 0, 80)
title.AnchorPoint = Vector2.new(0.5, 0)
title.Position = UDim2.new(0.5, 0, 0.1, 0)
title.BackgroundTransparency = 1
title.Text = "🎮 MY GAME"
title.TextColor3 = Color3.new(1, 1, 1)
title.Font = Enum.Font.GothamBlack
title.TextSize = 48
title.TextTransparency = 1  -- เริ่มโปร่งใส
title.Parent = screenGui

-- Buttons Container
local buttonsContainer = Instance.new("Frame")
buttonsContainer.Size = UDim2.new(0, 280, 0, 0)
buttonsContainer.AnchorPoint = Vector2.new(0.5, 0)
buttonsContainer.Position = UDim2.new(0.5, 0, 0.3, 0)
buttonsContainer.BackgroundTransparency = 1
buttonsContainer.AutomaticSize = Enum.AutomaticSize.Y
buttonsContainer.Parent = screenGui

local btnList = Instance.new("UIListLayout")
btnList.Padding = UDim.new(0, 10)
btnList.HorizontalAlignment = Enum.HorizontalAlignment.Center
btnList.Parent = buttonsContainer

-- สร้างปุ่ม
local buttons = {}
for i, item in ipairs(menuItems) do
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, 0, 0, 50)
    btn.BackgroundColor3 = item.color
    btn.BorderSizePixel = 0
    btn.Text = item.text
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 18
    btn.Position = UDim2.new(-1.5, 0, 0, 0)  -- เริ่มนอกจอซ้าย
    btn.LayoutOrder = i
    btn.Parent = buttonsContainer
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 10)
    corner.Parent = btn
    
    -- Hover Effects
    local originalColor = item.color
    btn.MouseEnter:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.1), {
            BackgroundColor3 = Color3.new(
                math.min(originalColor.R + 0.15, 1),
                math.min(originalColor.G + 0.15, 1),
                math.min(originalColor.B + 0.15, 1)
            ),
            Size = UDim2.new(1, 10, 0, 55)
        }):Play()
    end)
    
    btn.MouseLeave:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.1), {
            BackgroundColor3 = originalColor,
            Size = UDim2.new(1, 0, 0, 50)
        }):Play()
    end)
    
    buttons[i] = btn
end

-- Intro Animation
task.spawn(function()
    -- Fade in Title
    TweenService:Create(title, TweenInfo.new(0.8, Enum.EasingStyle.Sine), {
        TextTransparency = 0
    }):Play()
    
    task.wait(0.3)
    
    -- Slide in Buttons (staggered)
    for i, btn in ipairs(buttons) do
        task.delay((i-1) * 0.08, function()
            TweenService:Create(btn, TweenInfo.new(0.4, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
                Position = UDim2.new(0, 0, 0, 0)
            }):Play()
        end)
    end
end)
```

---

## 34.11 สรุป

ในบทนี้เราได้เรียนรู้:

1. **TweenService Basics** - Create, Play, Pause, Cancel
2. **TweenInfo** - Duration, EasingStyle, EasingDirection, Repeat, Reverse, Delay
3. **EasingStyles** - Linear, Sine, Back, Bounce, Elastic, Quad และอื่นๆ
4. **GUI Animation** - Slide, Scale, Fade
5. **Part Animation** - Float, Rotate, Bounce
6. **Color Animation** - Color Tween, Color Cycle
7. **Animation Sequence** - เล่นตามลำดับ
8. **Parallel Animation** - เล่นพร้อมกัน
9. **Spring Animation** - ฟิสิกส์แบบสปริง
10. **Loading Animation** - Spinner, Dot Loader, Progress Bar

ในบทต่อไป เราจะเรียนรู้เกี่ยวกับ SoundService

---

## แหล่งอ้างอิง

- [Roblox Developer Hub - TweenService](https://developer.roblox.com/en-us/api-reference/class/TweenService)
- [Roblox Developer Hub - TweenInfo](https://developer.roblox.com/en-us/api-reference/datatype/TweenInfo)
- [Roblox Easing Styles](https://developer.roblox.com/en-us/articles/Gui-tweening)
