# Part 74: Loading Screen (หน้าจอโหลด)

## บทนำ

Loading Screen เป็นส่วนแรกที่ผู้เล่นเห็น และมีผลต่อความประทับใจแรก ในบทนี้เราจะสร้าง Loading Screen ที่สวยงาม รองรับการแสดงความคืบหน้าจริง และมีแอนิเมชั่นที่น่าดึงดูด

---

## 1. โครงสร้าง Loading Screen

### 1.1 Loading Screen Manager

```lua
-- LoadingScreenManager.lua (Local Script ใน ReplicatedFirst)
-- Loading Screen หลัก - ทำงานก่อนสิ่งอื่นทั้งหมด

local ReplicatedFirst = game:GetService("ReplicatedFirst")
local ContentProvider = game:GetService("ContentProvider")
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")

-- ลบ Default Loading Screen ของ Roblox
ReplicatedFirst:RemoveDefaultLoadingScreen()

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- ===========================================
-- สร้าง Loading Screen GUI
-- ===========================================

local function createLoadingScreen()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "LoadingScreen"
    screenGui.IgnoreGuiInset = true
    screenGui.DisplayOrder = 1000  -- แสดงบนสุด
    screenGui.ResetOnSpawn = false
    screenGui.Parent = playerGui
    
    -- Background
    local background = Instance.new("Frame")
    background.Name = "Background"
    background.Size = UDim2.new(1, 0, 1, 0)
    background.Position = UDim2.new(0, 0, 0, 0)
    background.BackgroundColor3 = Color3.fromRGB(10, 10, 20)
    background.BorderSizePixel = 0
    background.Parent = screenGui
    
    -- Gradient Background
    local gradient = Instance.new("UIGradient")
    gradient.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(5, 5, 15)),
        ColorSequenceKeypoint.new(0.5, Color3.fromRGB(20, 20, 50)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(5, 5, 15))
    })
    gradient.Rotation = 135
    gradient.Parent = background
    
    -- Game Logo
    local logoFrame = Instance.new("Frame")
    logoFrame.Name = "LogoFrame"
    logoFrame.Size = UDim2.new(0, 300, 0, 100)
    logoFrame.Position = UDim2.new(0.5, -150, 0.3, 0)
    logoFrame.BackgroundTransparency = 1
    logoFrame.Parent = background
    
    local logoLabel = Instance.new("TextLabel")
    logoLabel.Name = "Logo"
    logoLabel.Size = UDim2.new(1, 0, 1, 0)
    logoLabel.BackgroundTransparency = 1
    logoLabel.Text = "MY AWESOME GAME"  -- เปลี่ยนเป็นชื่อเกม
    logoLabel.TextColor3 = Color3.fromRGB(255, 200, 50)
    logoLabel.TextSize = 36
    logoLabel.Font = Enum.Font.GothamBold
    logoLabel.TextStrokeTransparency = 0.5
    logoLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    logoLabel.Parent = logoFrame
    
    -- Tagline
    local tagline = Instance.new("TextLabel")
    tagline.Name = "Tagline"
    tagline.Size = UDim2.new(0, 400, 0, 30)
    tagline.Position = UDim2.new(0.5, -200, 0.42, 0)
    tagline.BackgroundTransparency = 1
    tagline.Text = "กำลังโหลด..."
    tagline.TextColor3 = Color3.fromRGB(180, 180, 180)
    tagline.TextSize = 16
    tagline.Font = Enum.Font.Gotham
    tagline.Parent = background
    
    -- Progress Bar Container
    local progressContainer = Instance.new("Frame")
    progressContainer.Name = "ProgressContainer"
    progressContainer.Size = UDim2.new(0, 500, 0, 8)
    progressContainer.Position = UDim2.new(0.5, -250, 0.55, 0)
    progressContainer.BackgroundColor3 = Color3.fromRGB(40, 40, 60)
    progressContainer.BorderSizePixel = 0
    progressContainer.Parent = background
    
    local containerCorner = Instance.new("UICorner")
    containerCorner.CornerRadius = UDim.new(1, 0)
    containerCorner.Parent = progressContainer
    
    -- Progress Bar Fill
    local progressFill = Instance.new("Frame")
    progressFill.Name = "ProgressFill"
    progressFill.Size = UDim2.new(0, 0, 1, 0)
    progressFill.Position = UDim2.new(0, 0, 0, 0)
    progressFill.BackgroundColor3 = Color3.fromRGB(100, 200, 255)
    progressFill.BorderSizePixel = 0
    progressFill.Parent = progressContainer
    
    local fillCorner = Instance.new("UICorner")
    fillCorner.CornerRadius = UDim.new(1, 0)
    fillCorner.Parent = progressFill
    
    -- Gradient on progress bar
    local fillGradient = Instance.new("UIGradient")
    fillGradient.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(50, 150, 255)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(150, 230, 255))
    })
    fillGradient.Parent = progressFill
    
    -- Progress Text
    local progressText = Instance.new("TextLabel")
    progressText.Name = "ProgressText"
    progressText.Size = UDim2.new(0, 100, 0, 25)
    progressText.Position = UDim2.new(0.5, -50, 0.58, 0)
    progressText.BackgroundTransparency = 1
    progressText.Text = "0%"
    progressText.TextColor3 = Color3.fromRGB(200, 200, 200)
    progressText.TextSize = 14
    progressText.Font = Enum.Font.GothamBold
    progressText.Parent = background
    
    -- Status Text
    local statusText = Instance.new("TextLabel")
    statusText.Name = "StatusText"
    statusText.Size = UDim2.new(0, 500, 0, 20)
    statusText.Position = UDim2.new(0.5, -250, 0.62, 0)
    statusText.BackgroundTransparency = 1
    statusText.Text = "กำลังเชื่อมต่อ..."
    statusText.TextColor3 = Color3.fromRGB(150, 150, 150)
    statusText.TextSize = 12
    statusText.Font = Enum.Font.Gotham
    statusText.Parent = background
    
    -- Loading Tips
    local tipsLabel = Instance.new("TextLabel")
    tipsLabel.Name = "Tips"
    tipsLabel.Size = UDim2.new(0, 600, 0, 40)
    tipsLabel.Position = UDim2.new(0.5, -300, 0.8, 0)
    tipsLabel.BackgroundTransparency = 1
    tipsLabel.Text = ""
    tipsLabel.TextColor3 = Color3.fromRGB(200, 200, 150)
    tipsLabel.TextSize = 14
    tipsLabel.Font = Enum.Font.GothamItalic
    tipsLabel.TextWrapped = true
    tipsLabel.Parent = background
    
    -- Spinning Loader
    local spinnerFrame = Instance.new("Frame")
    spinnerFrame.Name = "Spinner"
    spinnerFrame.Size = UDim2.new(0, 40, 0, 40)
    spinnerFrame.Position = UDim2.new(0.5, -20, 0.68, 0)
    spinnerFrame.BackgroundTransparency = 1
    spinnerFrame.Parent = background
    
    -- สร้าง dots สำหรับ spinner
    for i = 1, 8 do
        local dot = Instance.new("Frame")
        dot.Name = "Dot" .. i
        dot.Size = UDim2.new(0, 6, 0, 6)
        
        local angle = (i - 1) * (math.pi * 2 / 8)
        local x = math.cos(angle) * 15
        local y = math.sin(angle) * 15
        
        dot.Position = UDim2.new(0.5, x - 3, 0.5, y - 3)
        dot.BackgroundColor3 = Color3.fromRGB(100, 200, 255)
        dot.BackgroundTransparency = 1 - (i / 8)
        dot.BorderSizePixel = 0
        
        local dotCorner = Instance.new("UICorner")
        dotCorner.CornerRadius = UDim.new(1, 0)
        dotCorner.Parent = dot
        
        dot.Parent = spinnerFrame
    end
    
    return screenGui
end

-- สร้าง Loading Screen
local loadingGui = createLoadingScreen()
local background = loadingGui.Background
local progressFill = background.ProgressContainer.ProgressFill
local progressText = background.ProgressText
local statusText = background.StatusText
local tipsLabel = background.Tips
local spinnerFrame = background.Spinner

-- ===========================================
-- Tips ที่จะแสดงระหว่างโหลด
-- ===========================================

local TIPS = {
    "💡 กดปุ่ม F3 เพื่อดู Performance Stats",
    "💡 ใช้ Shop เพื่อซื้ออุปกรณ์ใหม่",
    "💡 ร่วมมือกับผู้เล่นคนอื่นเพื่อผ่าน Boss",
    "💡 บันทึก Progress โดยอัตโนมัติทุก 30 วินาที",
    "💡 กด E เพื่อโต้ตอบกับ NPC",
    "💡 อัพเกรด Equipment เพื่อเพิ่ม Stats",
    "💡 เข้าร่วม Guild เพื่อรับ Bonus พิเศษ",
}

-- หมุน Spinner
local spinnerAngle = 0
RunService.RenderStepped:Connect(function(dt)
    spinnerAngle = spinnerAngle + dt * 200  -- องศาต่อวินาที
    spinnerFrame.Rotation = spinnerAngle
end)

-- สลับ Tips ทุก 3 วินาที
local currentTipIndex = 1
task.spawn(function()
    while loadingGui.Parent do
        tipsLabel.TextTransparency = 0
        tipsLabel.Text = TIPS[currentTipIndex]
        
        task.wait(3)
        
        -- Fade out
        local fadeTween = TweenService:Create(tipsLabel, 
            TweenInfo.new(0.5), {TextTransparency = 1})
        fadeTween:Play()
        fadeTween.Completed:Wait()
        
        currentTipIndex = currentTipIndex % #TIPS + 1
    end
end)

-- ===========================================
-- อัพเดท Progress Bar
-- ===========================================

local function updateProgress(progress, status)
    -- progress: 0 ถึง 1
    progress = math.clamp(progress, 0, 1)
    
    -- Animate Progress Bar
    local targetWidth = progress
    local tween = TweenService:Create(progressFill, 
        TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
        {Size = UDim2.new(targetWidth, 0, 1, 0)}
    )
    tween:Play()
    
    -- อัพเดท Text
    progressText.Text = string.format("%d%%", math.floor(progress * 100))
    
    if status then
        statusText.Text = status
    end
end

-- ===========================================
-- โหลด Assets
-- ===========================================

-- รายการ Assets ที่ต้องโหลด
local ASSETS_TO_PRELOAD = {
    -- รูปภาพ UI
    "rbxassetid://123456789",   -- Logo
    "rbxassetid://987654321",   -- Background
    -- เพิ่ม Asset IDs จริงๆ ที่นี่
}

local LOADING_STAGES = {
    {name = "กำลังเชื่อมต่อ Server...", weight = 0.1},
    {name = "กำลังโหลด Game Assets...", weight = 0.5},
    {name = "กำลังตั้งค่า Character...", weight = 0.2},
    {name = "กำลังโหลด Game World...", weight = 0.2},
}

local function runLoadingSequence()
    local totalProgress = 0
    
    -- Stage 1: เชื่อมต่อ
    updateProgress(0.05, LOADING_STAGES[1].name)
    task.wait(0.5)
    totalProgress = LOADING_STAGES[1].weight
    
    -- Stage 2: โหลด Assets
    updateProgress(totalProgress, LOADING_STAGES[2].name)
    
    if #ASSETS_TO_PRELOAD > 0 then
        ContentProvider:PreloadAsync(ASSETS_TO_PRELOAD, function(assetId, status)
            -- ไม่สามารถรู้จำนวนจริงได้ง่ายๆ จาก callback นี้
            -- แต่สามารถใช้ counter ได้
        end)
    end
    totalProgress = totalProgress + LOADING_STAGES[2].weight
    updateProgress(totalProgress, LOADING_STAGES[2].name)
    
    -- Stage 3: รอ Character
    updateProgress(totalProgress, LOADING_STAGES[3].name)
    
    -- รอ Character โหลด
    if not player.Character then
        player.CharacterAdded:Wait()
    end
    
    local character = player.Character
    character:WaitForChild("HumanoidRootPart")
    character:WaitForChild("Humanoid")
    
    totalProgress = totalProgress + LOADING_STAGES[3].weight
    updateProgress(totalProgress, LOADING_STAGES[3].name)
    
    -- Stage 4: รอ Game World
    updateProgress(totalProgress, LOADING_STAGES[4].name)
    
    -- รอ Game ที่สำคัญโหลด
    game:GetService("ReplicatedStorage"):WaitForChild("GameReady", 30)
    
    totalProgress = 1.0
    updateProgress(totalProgress, "พร้อมแล้ว!")
    
    task.wait(0.5)
end

-- รัน Loading Sequence
task.spawn(function()
    local success, err = pcall(runLoadingSequence)
    if not success then
        warn("Loading error:", err)
        updateProgress(1, "โหลดเสร็จ (มีข้อผิดพลาด)")
        task.wait(1)
    end
    
    -- Animate Out
    local fadeOut = TweenService:Create(background,
        TweenInfo.new(1, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
        {BackgroundTransparency = 1}
    )
    
    -- Fade ทุก Element
    for _, desc in ipairs(background:GetDescendants()) do
        if desc:IsA("TextLabel") or desc:IsA("Frame") then
            TweenService:Create(desc, TweenInfo.new(1), 
                {BackgroundTransparency = 1, TextTransparency = 1}
            ):Play()
        end
    end
    
    fadeOut:Play()
    fadeOut.Completed:Wait()
    
    loadingGui:Destroy()
end)
```

---

## 2. Animated Background

### 2.1 Particle Background Effect

```lua
-- LoadingBackground.lua (Local Script)
-- Background Effect สวยงามสำหรับ Loading Screen

local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")

local function createParticleBackground(parent)
    local particleContainer = Instance.new("Frame")
    particleContainer.Name = "Particles"
    particleContainer.Size = UDim2.new(1, 0, 1, 0)
    particleContainer.BackgroundTransparency = 1
    particleContainer.ZIndex = 1
    particleContainer.Parent = parent
    
    local particles = {}
    
    -- สร้าง Particle
    local function createParticle()
        local particle = Instance.new("Frame")
        particle.Size = UDim2.new(0, math.random(2, 5), 0, math.random(2, 5))
        particle.Position = UDim2.new(math.random(), 0, 1.1, 0)  -- เริ่มใต้หน้าจอ
        particle.BackgroundColor3 = Color3.fromHSV(
            math.random(180, 240) / 360,  -- สีฟ้า-น้ำเงิน
            0.8, 
            1
        )
        particle.BackgroundTransparency = 0.3
        particle.BorderSizePixel = 0
        particle.ZIndex = 2
        
        local corner = Instance.new("UICorner")
        corner.CornerRadius = UDim.new(1, 0)
        corner.Parent = particle
        
        particle.Parent = particleContainer
        
        -- Animate ขึ้น
        local duration = math.random(3, 8)
        local targetX = particle.Position.X.Scale + (math.random() - 0.5) * 0.2
        
        local moveTween = TweenService:Create(particle,
            TweenInfo.new(duration, Enum.EasingStyle.Linear),
            {
                Position = UDim2.new(targetX, 0, -0.1, 0),
                BackgroundTransparency = 1
            }
        )
        moveTween:Play()
        
        moveTween.Completed:Connect(function()
            particle:Destroy()
        end)
        
        return particle
    end
    
    -- Spawn particles ต่อเนื่อง
    local function spawnLoop()
        while particleContainer.Parent do
            if #particleContainer:GetChildren() < 50 then
                createParticle()
            end
            task.wait(math.random(10, 30) / 100)  -- 0.1-0.3 วินาที
        end
    end
    
    task.spawn(spawnLoop)
    
    -- สร้าง Initial particles
    for i = 1, 30 do
        local particle = Instance.new("Frame")
        particle.Size = UDim2.new(0, math.random(2, 5), 0, math.random(2, 5))
        particle.Position = UDim2.new(math.random(), 0, math.random(), 0)  -- Random position
        particle.BackgroundColor3 = Color3.fromHSV(
            math.random(180, 240) / 360,
            0.8, 1
        )
        particle.BackgroundTransparency = math.random(30, 70) / 100
        particle.BorderSizePixel = 0
        particle.ZIndex = 2
        
        local corner = Instance.new("UICorner")
        corner.CornerRadius = UDim.new(1, 0)
        corner.Parent = particle
        
        particle.Parent = particleContainer
    end
    
    return particleContainer
end

return {createParticleBackground = createParticleBackground}
```

---

## 3. Loading Screen ขั้นสูง

### 3.1 Cinematic Loading Screen

```lua
-- CinematicLoader.lua (Local Script ใน ReplicatedFirst)
-- Loading Screen แบบ Cinematic พร้อม Story Text

local ReplicatedFirst = game:GetService("ReplicatedFirst")
local TweenService = game:GetService("TweenService")
local Players = game:GetService("Players")

ReplicatedFirst:RemoveDefaultLoadingScreen()

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- เนื้อเรื่องที่จะแสดง
local STORY_LINES = {
    {text = "ในดินแดนที่ถูกความมืดปกคลุม...", delay = 2},
    {text = "มีฮีโร่คนหนึ่งที่กล้าหาญ...", delay = 2},
    {text = "นั่นคือคุณ!", delay = 1.5},
}

local function createCinematicLoader()
    local gui = Instance.new("ScreenGui")
    gui.Name = "CinematicLoader"
    gui.IgnoreGuiInset = true
    gui.DisplayOrder = 999
    gui.Parent = playerGui
    
    -- Black Background
    local bg = Instance.new("Frame")
    bg.Size = UDim2.new(1, 0, 1, 0)
    bg.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    bg.BorderSizePixel = 0
    bg.Parent = gui
    
    -- Story Text
    local storyText = Instance.new("TextLabel")
    storyText.Size = UDim2.new(0, 800, 0, 60)
    storyText.Position = UDim2.new(0.5, -400, 0.5, -30)
    storyText.BackgroundTransparency = 1
    storyText.TextColor3 = Color3.fromRGB(255, 255, 255)
    storyText.TextTransparency = 1
    storyText.TextSize = 28
    storyText.Font = Enum.Font.GothamItalic
    storyText.TextWrapped = true
    storyText.Parent = bg
    
    -- Progress Bar (บางๆ ด้านล่าง)
    local progressBg = Instance.new("Frame")
    progressBg.Size = UDim2.new(0.6, 0, 0, 2)
    progressBg.Position = UDim2.new(0.2, 0, 0.9, 0)
    progressBg.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    progressBg.BorderSizePixel = 0
    progressBg.Parent = bg
    
    local progressFill = Instance.new("Frame")
    progressFill.Name = "Fill"
    progressFill.Size = UDim2.new(0, 0, 1, 0)
    progressFill.BackgroundColor3 = Color3.fromRGB(200, 200, 200)
    progressFill.BorderSizePixel = 0
    progressFill.Parent = progressBg
    
    return gui, storyText, progressFill, bg
end

local gui, storyText, progressFill, bg = createCinematicLoader()

-- แสดง Story Lines
local function showStory()
    for i, line in ipairs(STORY_LINES) do
        storyText.Text = line.text
        
        -- Fade In
        local fadeIn = TweenService:Create(storyText,
            TweenInfo.new(0.8, Enum.EasingStyle.Quad),
            {TextTransparency = 0}
        )
        fadeIn:Play()
        fadeIn.Completed:Wait()
        
        -- อ่านได้
        task.wait(line.delay)
        
        -- Fade Out
        local fadeOut = TweenService:Create(storyText,
            TweenInfo.new(0.8, Enum.EasingStyle.Quad),
            {TextTransparency = 1}
        )
        fadeOut:Play()
        fadeOut.Completed:Wait()
        
        task.wait(0.3)
    end
end

-- Main Loading Logic
task.spawn(function()
    -- แสดง Story
    showStory()
    
    -- โหลดจริงๆ
    if not player.Character then
        player.CharacterAdded:Wait()
    end
    
    local char = player.Character
    char:WaitForChild("HumanoidRootPart")
    
    -- Animate Progress
    local fillTween = TweenService:Create(progressFill,
        TweenInfo.new(1.5),
        {Size = UDim2.new(1, 0, 1, 0)}
    )
    fillTween:Play()
    fillTween.Completed:Wait()
    
    task.wait(0.5)
    
    -- Fade Out ทั้งหมด
    local finalFade = TweenService:Create(bg,
        TweenInfo.new(1),
        {BackgroundTransparency = 1}
    )
    finalFade:Play()
    finalFade.Completed:Wait()
    
    gui:Destroy()
end)
```

---

## 4. Progress Tracking System

### 4.1 ติดตาม Progress จริงๆ จาก Server

```lua
-- ServerLoadingCoordinator.lua (Server Script)
-- ส่งข้อมูลการโหลดไปยัง Client

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- สร้าง RemoteEvents สำหรับ Loading
local loadingFolder = Instance.new("Folder")
loadingFolder.Name = "Loading"
loadingFolder.Parent = ReplicatedStorage

local progressEvent = Instance.new("RemoteEvent")
progressEvent.Name = "LoadingProgress"
progressEvent.Parent = loadingFolder

local readyEvent = Instance.new("RemoteEvent")
readyEvent.Name = "GameReady"
readyEvent.Parent = loadingFolder

-- สร้าง Folder สำหรับบอก Client ว่า Ready
local gameReadyValue = Instance.new("BoolValue")
gameReadyValue.Name = "GameReady"
gameReadyValue.Value = false
gameReadyValue.Parent = ReplicatedStorage

-- Loading Stages บน Server
local serverLoadingStages = {
    {name = "สร้าง Game World", progress = 0.3},
    {name = "โหลด NPC Data", progress = 0.5},
    {name = "เริ่ม Systems", progress = 0.8},
    {name = "พร้อมแล้ว!", progress = 1.0},
}

local function performServerLoading()
    -- Stage 1: สร้าง World
    print("[SERVER] สร้าง Game World...")
    -- สร้าง Terrain, Spawn Points, etc.
    task.wait(0.5)  -- จำลองการโหลด
    
    -- Stage 2: โหลด Data
    print("[SERVER] โหลด NPC Data...")
    task.wait(0.3)
    
    -- Stage 3: เริ่ม Systems  
    print("[SERVER] เริ่ม Systems...")
    task.wait(0.2)
    
    -- บอกทุก Client ว่าพร้อมแล้ว
    gameReadyValue.Value = true
    
    -- ส่ง Event ไปทุก Client ที่ Online
    for _, player in ipairs(Players:GetPlayers()) do
        readyEvent:FireClient(player)
    end
    
    print("[SERVER] Game Ready!")
end

-- เริ่ม Server Loading
task.spawn(performServerLoading)

-- Player เข้ามาระหว่าง Server Loading
Players.PlayerAdded:Connect(function(player)
    if gameReadyValue.Value then
        -- Server พร้อมแล้ว แจ้ง Player ทันที
        task.delay(0.5, function()
            readyEvent:FireClient(player)
        end)
    end
    -- ถ้ายังไม่พร้อม จะได้รับ event เมื่อ Server พร้อม
end)
```

```lua
-- ClientLoadingListener.lua (Local Script)
-- รับ Events จาก Server สำหรับ Progress

local ReplicatedStorage = game:GetService("ReplicatedStorage")

local loadingFolder = ReplicatedStorage:WaitForChild("Loading")
local progressEvent = loadingFolder:WaitForChild("LoadingProgress")
local readyEvent = loadingFolder:WaitForChild("GameReady")

-- รับ Progress Updates
progressEvent.OnClientEvent:Connect(function(progress, status)
    -- ส่งไปยัง Loading Screen
    local loadingGui = game.Players.LocalPlayer.PlayerGui:FindFirstChild("LoadingScreen")
    if loadingGui then
        -- อัพเดท Progress Bar
        local bg = loadingGui.Background
        if bg then
            local fill = bg.ProgressContainer.ProgressFill
            local tween = game:GetService("TweenService"):Create(fill,
                game:GetService("TweenService").TweenInfo and 
                TweenInfo.new(0.3) or TweenInfo.new(0.3),
                {Size = UDim2.new(progress, 0, 1, 0)}
            )
            tween:Play()
            
            if bg:FindFirstChild("StatusText") then
                bg.StatusText.Text = status or ""
            end
        end
    end
end)

-- เมื่อ Server Ready
readyEvent.OnClientEvent:Connect(function()
    print("Server is ready!")
    -- ส่งสัญญาณให้ Loading Screen ปิด
end)
```

---

## 5. Minimal Loading Screen

### 5.1 Simple แต่ Elegant

```lua
-- MinimalLoader.lua (Local Script ใน ReplicatedFirst)
-- Loading Screen เรียบง่ายแต่สวย

local ReplicatedFirst = game:GetService("ReplicatedFirst")
local TweenService = game:GetService("TweenService")
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

ReplicatedFirst:RemoveDefaultLoadingScreen()

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- สร้าง Minimal GUI
local gui = Instance.new("ScreenGui")
gui.Name = "MinimalLoader"
gui.IgnoreGuiInset = true
gui.DisplayOrder = 1000
gui.Parent = playerGui

local bg = Instance.new("Frame")
bg.Size = UDim2.new(1, 0, 1, 0)
bg.BackgroundColor3 = Color3.fromRGB(15, 15, 25)
bg.BorderSizePixel = 0
bg.Parent = gui

-- Animated Logo Text (ตัวอักษรปรากฏทีละตัว)
local logoText = Instance.new("TextLabel")
logoText.Size = UDim2.new(0, 400, 0, 60)
logoText.Position = UDim2.new(0.5, -200, 0.4, 0)
logoText.BackgroundTransparency = 1
logoText.Text = ""
logoText.TextColor3 = Color3.fromRGB(255, 255, 255)
logoText.TextSize = 48
logoText.Font = Enum.Font.GothamBold
logoText.Parent = bg

-- Dots Loading Indicator
local dotsLabel = Instance.new("TextLabel")
dotsLabel.Size = UDim2.new(0, 100, 0, 30)
dotsLabel.Position = UDim2.new(0.5, -50, 0.55, 0)
dotsLabel.BackgroundTransparency = 1
dotsLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
dotsLabel.TextSize = 24
dotsLabel.Font = Enum.Font.GothamBold
dotsLabel.Parent = bg

-- แอนิเมชั่นตัวอักษร
local GAME_NAME = "EPIC GAME"
local function animateLogo()
    for i = 1, #GAME_NAME do
        logoText.Text = string.sub(GAME_NAME, 1, i)
        task.wait(0.05)
    end
end

-- แอนิเมชั่น Dots
local dotCount = 0
local dotsConnection = RunService.RenderStepped:Connect(function()
    -- อัพเดทแค่ทุก 0.3 วินาที
end)

task.spawn(function()
    while dotsLabel.Parent do
        dotCount = (dotCount % 3) + 1
        dotsLabel.Text = string.rep("•", dotCount) .. string.rep(" ", 3 - dotCount)
        task.wait(0.4)
    end
end)

-- Main Flow
task.spawn(function()
    animateLogo()
    
    -- รอ Game Load
    if not player.Character then
        player.CharacterAdded:Wait()
    end
    player.Character:WaitForChild("HumanoidRootPart")
    
    -- Fade Out
    dotsConnection:Disconnect()
    
    local fadeOut = TweenService:Create(bg,
        TweenInfo.new(0.8, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
        {BackgroundTransparency = 1}
    )
    
    for _, desc in ipairs(bg:GetDescendants()) do
        if desc:IsA("TextLabel") then
            TweenService:Create(desc, TweenInfo.new(0.8), {TextTransparency = 1}):Play()
        end
    end
    
    fadeOut:Play()
    fadeOut.Completed:Wait()
    gui:Destroy()
end)
```

---

## 6. ข้อผิดพลาดที่พบบ่อย

### ❌ ข้อผิดพลาดที่ 1: ลืมลบ Default Loading Screen

```lua
-- ❌ แบบผิด: ไม่ลบ Default Screen
-- Default Screen ของ Roblox จะแสดงพร้อมกับ Custom Screen

-- ✅ แบบถูก: ลบทันที
local ReplicatedFirst = game:GetService("ReplicatedFirst")
ReplicatedFirst:RemoveDefaultLoadingScreen()  -- เรียกก่อนสิ่งอื่น
```

### ❌ ข้อผิดพลาดที่ 2: Script ไม่อยู่ใน ReplicatedFirst

```lua
-- Loading Screen Script ต้องอยู่ใน:
-- ReplicatedFirst/LocalScript  ✅
-- StarterPlayerScripts/LocalScript  ❌ (โหลดช้าเกินไป)
-- StarterGui/LocalScript  ❌ (โหลดช้าเกินไป)
```

### ❌ ข้อผิดพลาดที่ 3: ไม่รอ Character Load

```lua
-- ❌ แบบผิด: ปิด Loading Screen ทันทีโดยไม่รอ
task.wait(3)  -- แค่รอ 3 วินาที
gui:Destroy()

-- ✅ แบบถูก: รอจนกว่า Character จะ Load จริงๆ
if not player.Character then
    player.CharacterAdded:Wait()
end
local char = player.Character
char:WaitForChild("HumanoidRootPart")  -- รอจนมี HumanoidRootPart
char:WaitForChild("Humanoid")
-- ปิด Loading Screen
```

### ❌ ข้อผิดพลาดที่ 4: PreloadAsync ไม่ถูกต้อง

```lua
-- ❌ แบบผิด: ใส่ URL ผิดรูปแบบ
ContentProvider:PreloadAsync({"123456789"})  -- ไม่มี prefix

-- ✅ แบบถูก:
ContentProvider:PreloadAsync({"rbxassetid://123456789"})
```

---

## 7. แบบฝึกหัด

### แบบฝึกหัดที่ 1: Custom Progress Bar
สร้าง Progress Bar ที่:
- มี Glow Effect เมื่อโหลดถึง 100%
- แสดงตัวเลข Percentage แบบ Animated (ไม่กระโดด)
- เปลี่ยนสีจากแดงไปเขียวเมื่อ Progress เพิ่มขึ้น

### แบบฝึกหัดที่ 2: Loading Screen Themes
สร้างระบบ Theme สำหรับ Loading Screen:
- Dark Theme (ค่า default)
- Light Theme
- Space Theme (ดาว, ดาวเคราะห์)
- เลือก Theme ตาม GamePass หรือ Setting ของ Player

### แบบฝึกหัดที่ 3: Real Progress Tracking
เชื่อมต่อกับ Server เพื่อแสดง Progress จริง:
- Server ส่ง Progress Events ระหว่างโหลด
- Client แสดง Status Message จาก Server
- แสดงชื่อของ Stage ที่กำลังโหลด

---

## สรุป

Loading Screen ที่ดีต้องมี:
1. **Performance**: ใช้ ReplicatedFirst เพื่อโหลดเร็ว
2. **Feedback**: แสดง Progress จริงๆ ไม่ใช่ Fake Progress
3. **Entertainment**: Tips, Story, Animation ระหว่างรอ
4. **Reliability**: รอ Game พร้อมจริงๆ ก่อนปิด

ในส่วนถัดไป (Part 75) เราจะสร้าง Settings Menu ที่สมบูรณ์ พร้อมระบบ Save/Load Settings ด้วย DataStore
