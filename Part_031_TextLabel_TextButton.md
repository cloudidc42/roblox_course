# ตอนที่ 31: TextLabel และ TextButton

## บทนำ

TextLabel และ TextButton เป็น GUI Elements ที่ใช้บ่อยที่สุดใน Roblox TextLabel ใช้แสดงข้อความ ส่วน TextButton ใช้สร้างปุ่มที่กดได้ ในบทนี้เราจะเรียนรู้ Properties, Styling, และการสร้าง UI Components ที่สวยงาม

---

## 31.1 TextLabel Properties

```lua
-- LocalScript
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

local screenGui = Instance.new("ScreenGui")
screenGui.Parent = LocalPlayer.PlayerGui

local label = Instance.new("TextLabel")

-- ขนาดและตำแหน่ง
label.Size = UDim2.new(0, 300, 0, 50)
label.Position = UDim2.new(0.5, -150, 0.5, -25)
label.AnchorPoint = Vector2.new(0.5, 0.5)

-- ข้อความ
label.Text = "สวัสดี Roblox!"
label.Font = Enum.Font.GothamBold   -- Font
label.TextSize = 24                  -- ขนาด Text

-- สี
label.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
label.BackgroundTransparency = 0
label.TextColor3 = Color3.new(1, 1, 1)     -- สีข้อความ

-- ขอบข้อความ (Stroke)
label.TextStrokeColor3 = Color3.new(0, 0, 0)
label.TextStrokeTransparency = 0.5  -- 0=ทึบ, 1=ซ่อน

-- การจัดตำแหน่ง
label.TextXAlignment = Enum.TextXAlignment.Center
label.TextYAlignment = Enum.TextYAlignment.Center

-- Word Wrap
label.TextWrapped = true    -- ตัดข้อความลงบรรทัดใหม่
label.TextScaled = false    -- ปรับขนาด Text ตาม Frame

-- ตัดข้อความที่ยาวเกิน
label.TextTruncate = Enum.TextTruncate.AtEnd  -- ตัดท้าย "..."

-- Auto Size
label.AutomaticSize = Enum.AutomaticSize.Y  -- ขยาย Y ตามข้อความ

-- Rich Text (HTML-like formatting)
label.RichText = true
label.Text = "<b>ตัวหนา</b> <i>เอียง</i> <font color='rgb(255,100,100)'>สีแดง</font>"

label.Parent = screenGui
```

---

## 31.2 Fonts ที่ใช้ได้

```lua
-- Fonts หลักๆ ใน Roblox
local fonts = {
    Enum.Font.Legacy,           -- Font เก่า
    Enum.Font.Arial,            -- Arial
    Enum.Font.ArialBold,        -- Arial Bold
    Enum.Font.SourceSans,       -- Source Sans
    Enum.Font.SourceSansBold,   -- Source Sans Bold
    Enum.Font.SourceSansLight,  -- Source Sans Light
    Enum.Font.Gotham,           -- Gotham (แนะนำ)
    Enum.Font.GothamBold,       -- Gotham Bold
    Enum.Font.GothamBlack,      -- Gotham Black
    Enum.Font.GothamMedium,     -- Gotham Medium
    Enum.Font.FredokaOne,       -- Fredoka One (กลม)
    Enum.Font.Roboto,           -- Roboto
    Enum.Font.RobotoMono,       -- Roboto Mono (monospace)
    Enum.Font.Code,             -- Code Font
    Enum.Font.Cartoon,          -- Cartoon
    Enum.Font.Fantasy,          -- Fantasy
    Enum.Font.Bodoni,           -- Bodoni
    Enum.Font.Antique,          -- Antique
    Enum.Font.Bangers,          -- Bangers (ตัวหนาๆ)
    Enum.Font.Oswald,           -- Oswald
    Enum.Font.PermanentMarker,  -- เหมือนเขียนด้วยปากกา
    Enum.Font.TitilliumWeb,     -- Titillium Web
    Enum.Font.Jura,             -- Jura
    Enum.Font.SpecialElite,     -- Special Elite
    Enum.Font.Ubuntu,           -- Ubuntu
    Enum.Font.Highway,          -- Highway
    Enum.Font.SciFi,            -- Sci-Fi
}

-- Custom Font (Font Face)
local label = Instance.new("TextLabel")
label.FontFace = Font.new("rbxasset://fonts/families/GothamSSm.json")
label.FontFace = Font.new(
    "rbxasset://fonts/families/GothamSSm.json",
    Enum.FontWeight.Bold,
    Enum.FontStyle.Normal
)
```

---

## 31.3 Rich Text Formatting

```lua
-- Rich Text ทำให้จัดรูปแบบ Text ได้ซับซ้อนขึ้น
local label = Instance.new("TextLabel")
label.RichText = true

-- Tags ที่ใช้ได้:
-- <b> ตัวหนา </b>
-- <i> เอียง </i>
-- <u> ขีดเส้นใต้ </u>
-- <s> ขีดฆ่า </s>
-- <br /> ขึ้นบรรทัดใหม่
-- <font color="rgb(R,G,B)"> สีข้อความ </font>
-- <font size="16"> ขนาด </font>
-- <font face="GothamBold"> Font </font>
-- <stroke color="rgb(0,0,0)" joins="round" thickness="2" transparency="0"> ขอบ </stroke>

label.Text = [[
<b>ชื่อ:</b> ผู้เล่น A <br />
<font color="rgb(255, 200, 50)">Level 99</font> <br />
HP: <font color="rgb(100, 255, 100)">200/200</font> <br />
<i>ผู้เล่นตัวอย่าง</i>
]]

-- ตัวอย่างสวยงาม
label.Text = [[<stroke color="rgb(0,0,0)" thickness="2">
<font color="rgb(255, 215, 0)"><b>⭐ LEGENDARY</b></font>
</stroke>]]
```

---

## 31.4 TextButton Properties

```lua
local button = Instance.new("TextButton")

-- Properties เหมือน TextLabel บวกเพิ่ม:
button.Text = "คลิกฉัน!"
button.Font = Enum.Font.GothamBold
button.TextSize = 16
button.BackgroundColor3 = Color3.fromRGB(70, 130, 200)
button.TextColor3 = Color3.new(1, 1, 1)
button.BorderSizePixel = 0

-- Auto Button Color (เปลี่ยนสีอัตโนมัติเมื่อ Hover/Click)
button.AutoButtonColor = true  -- Roblox จัดการให้อัตโนมัติ
-- ถ้าต้องการ Custom ให้ตั้งเป็น false

-- เมื่อกดค้าง
button.Active = true

-- Events
button.MouseButton1Click:Connect(function()
    print("คลิก!")
end)

button.MouseButton1Down:Connect(function()
    print("กดลง")
end)

button.MouseButton1Up:Connect(function()
    print("ปล่อยออก")
end)

button.MouseButton2Click:Connect(function()
    print("คลิกขวา!")
end)

button.MouseEnter:Connect(function()
    print("Mouse เข้ามา")
end)

button.MouseLeave:Connect(function()
    print("Mouse ออกไป")
end)

button.Parent = screenGui
```

---

## 31.5 สร้างปุ่มที่สวยงาม

```lua
-- LocalScript: Beautiful Buttons
local TweenService = game:GetService("TweenService")
local Players = game:GetService("Players")

local screenGui = Instance.new("ScreenGui")
screenGui.Parent = Players.LocalPlayer.PlayerGui

-- ฟังก์ชันสร้างปุ่มสไตล์ต่างๆ

-- 1. Primary Button
local function createPrimaryButton(text, parent)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 160, 0, 45)
    btn.BackgroundColor3 = Color3.fromRGB(80, 130, 220)
    btn.Text = text
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 15
    btn.BorderSizePixel = 0
    btn.AutoButtonColor = false
    btn.Parent = parent
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 8)
    corner.Parent = btn
    
    -- Hover Effect
    local originalColor = btn.BackgroundColor3
    local hoverColor = Color3.fromRGB(100, 150, 240)
    local pressColor = Color3.fromRGB(60, 110, 200)
    
    btn.MouseEnter:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.15), {BackgroundColor3 = hoverColor}):Play()
    end)
    
    btn.MouseLeave:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.15), {BackgroundColor3 = originalColor}):Play()
    end)
    
    btn.MouseButton1Down:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.1), {BackgroundColor3 = pressColor}):Play()
        TweenService:Create(btn, TweenInfo.new(0.1), {Size = UDim2.new(0, 155, 0, 42)}):Play()
    end)
    
    btn.MouseButton1Up:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.1), {BackgroundColor3 = hoverColor}):Play()
        TweenService:Create(btn, TweenInfo.new(0.1), {Size = UDim2.new(0, 160, 0, 45)}):Play()
    end)
    
    return btn
end

-- 2. Outlined Button
local function createOutlinedButton(text, color, parent)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 160, 0, 45)
    btn.BackgroundTransparency = 1
    btn.Text = text
    btn.TextColor3 = color
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 15
    btn.BorderSizePixel = 0
    btn.AutoButtonColor = false
    btn.Parent = parent
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 8)
    corner.Parent = btn
    
    local stroke = Instance.new("UIStroke")
    stroke.Color = color
    stroke.Thickness = 2
    stroke.Parent = btn
    
    btn.MouseEnter:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.15), {BackgroundTransparency = 0.8}):Play()
        btn.BackgroundColor3 = color
    end)
    
    btn.MouseLeave:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.15), {BackgroundTransparency = 1}):Play()
    end)
    
    return btn
end

-- 3. Danger Button (แดง)
local function createDangerButton(text, parent)
    local btn = createPrimaryButton(text, parent)
    btn.BackgroundColor3 = Color3.fromRGB(200, 60, 60)
    return btn
end

-- 4. Success Button (เขียว)
local function createSuccessButton(text, parent)
    local btn = createPrimaryButton(text, parent)
    btn.BackgroundColor3 = Color3.fromRGB(50, 160, 80)
    return btn
end

-- 5. Icon Button
local function createIconButton(icon, parent)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 45, 0, 45)
    btn.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
    btn.Text = icon
    btn.TextSize = 22
    btn.BorderSizePixel = 0
    btn.AutoButtonColor = false
    btn.Parent = parent
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 8)
    corner.Parent = btn
    
    btn.MouseEnter:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.15), {
            BackgroundColor3 = Color3.fromRGB(80, 80, 100)
        }):Play()
    end)
    
    btn.MouseLeave:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.15), {
            BackgroundColor3 = Color3.fromRGB(60, 60, 80)
        }):Play()
    end)
    
    return btn
end

-- สร้างตัวอย่าง
local container = Instance.new("Frame")
container.Size = UDim2.new(0, 400, 0, 300)
container.AnchorPoint = Vector2.new(0.5, 0.5)
container.Position = UDim2.new(0.5, 0, 0.5, 0)
container.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
container.BorderSizePixel = 0
container.Parent = screenGui

local layout = Instance.new("UIListLayout")
layout.Padding = UDim.new(0, 10)
layout.HorizontalAlignment = Enum.HorizontalAlignment.Center
layout.VerticalAlignment = Enum.VerticalAlignment.Center
layout.Parent = container

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 12)
corner.Parent = container

local btn1 = createPrimaryButton("✔ ยืนยัน", container)
local btn2 = createDangerButton("✖ ลบ", container)
local btn3 = createSuccessButton("💾 บันทึก", container)
local btn4 = createOutlinedButton("ข้ามไป", Color3.fromRGB(150, 150, 200), container)

btn1.MouseButton1Click:Connect(function() print("ยืนยัน!") end)
btn2.MouseButton1Click:Connect(function() print("ลบ!") end)
btn3.MouseButton1Click:Connect(function() print("บันทึก!") end)
btn4.MouseButton1Click:Connect(function() print("ข้าม") end)
```

---

## 31.6 Animated Text

```lua
-- LocalScript: Typewriter Effect
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local screenGui = Instance.new("ScreenGui")
screenGui.Parent = Players.LocalPlayer.PlayerGui

local dialogBox = Instance.new("Frame")
dialogBox.Size = UDim2.new(0.8, 0, 0, 120)
dialogBox.AnchorPoint = Vector2.new(0.5, 1)
dialogBox.Position = UDim2.new(0.5, 0, 0.95, 0)
dialogBox.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
dialogBox.BorderSizePixel = 0
dialogBox.Parent = screenGui

local dbCorner = Instance.new("UICorner")
dbCorner.CornerRadius = UDim.new(0, 10)
dbCorner.Parent = dialogBox

local dbStroke = Instance.new("UIStroke")
dbStroke.Color = Color3.fromRGB(100, 100, 150)
dbStroke.Thickness = 2
dbStroke.Parent = dialogBox

-- Speaker Name
local speakerLabel = Instance.new("TextLabel")
speakerLabel.Size = UDim2.new(0, 150, 0, 30)
speakerLabel.Position = UDim2.new(0, 15, 0, 10)
speakerLabel.BackgroundColor3 = Color3.fromRGB(80, 130, 220)
speakerLabel.BorderSizePixel = 0
speakerLabel.Text = "NPC"
speakerLabel.TextColor3 = Color3.new(1, 1, 1)
speakerLabel.Font = Enum.Font.GothamBold
speakerLabel.TextSize = 14
speakerLabel.Parent = dialogBox

local speakerCorner = Instance.new("UICorner")
speakerCorner.CornerRadius = UDim.new(0, 5)
speakerCorner.Parent = speakerLabel

-- Dialog Text
local dialogText = Instance.new("TextLabel")
dialogText.Size = UDim2.new(1, -30, 1, -55)
dialogText.Position = UDim2.new(0, 15, 0, 50)
dialogText.BackgroundTransparency = 1
dialogText.Text = ""
dialogText.TextColor3 = Color3.new(1, 1, 1)
dialogText.Font = Enum.Font.Gotham
dialogText.TextSize = 16
dialogText.TextWrapped = true
dialogText.TextXAlignment = Enum.TextXAlignment.Left
dialogText.TextYAlignment = Enum.TextYAlignment.Top
dialogText.Parent = dialogBox

-- Typewriter Effect
local function typewrite(text, speed)
    speed = speed or 0.04  -- วินาทีต่ออักษร
    dialogText.Text = ""
    
    for i = 1, #text do
        dialogText.Text = string.sub(text, 1, i)
        task.wait(speed)
    end
end

-- ตัวอย่างการใช้
task.spawn(function()
    typewrite("สวัสดีผู้กล้า! เราต้องการความช่วยเหลือของคุณ...", 0.05)
    wait(1)
    typewrite("มังกรได้มาโจมตีหมู่บ้านของเรา และเราต้องการนักรบผู้กล้า!", 0.04)
    wait(1)
    typewrite("คุณจะช่วยเราไหม?", 0.06)
end)
```

---

## 31.7 Countdown Timer

```lua
-- LocalScript: Countdown Timer
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")

local screenGui = Instance.new("ScreenGui")
screenGui.Parent = Players.LocalPlayer.PlayerGui

-- Timer Display
local timerFrame = Instance.new("Frame")
timerFrame.Size = UDim2.new(0, 120, 0, 60)
timerFrame.AnchorPoint = Vector2.new(0.5, 0)
timerFrame.Position = UDim2.new(0.5, 0, 0, 10)
timerFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
timerFrame.BorderSizePixel = 0
timerFrame.Parent = screenGui

local timerCorner = Instance.new("UICorner")
timerCorner.CornerRadius = UDim.new(0, 10)
timerCorner.Parent = timerFrame

local timerLabel = Instance.new("TextLabel")
timerLabel.Size = UDim2.new(1, 0, 1, 0)
timerLabel.BackgroundTransparency = 1
timerLabel.Text = "5:00"
timerLabel.TextColor3 = Color3.new(1, 1, 1)
timerLabel.Font = Enum.Font.GothamBlack
timerLabel.TextSize = 28
timerLabel.Parent = timerFrame

-- Countdown Logic
local function startCountdown(seconds, onComplete)
    local remaining = seconds
    
    local connection
    connection = RunService.Heartbeat:Connect(function(dt)
        remaining = remaining - dt
        
        if remaining <= 0 then
            remaining = 0
            connection:Disconnect()
            timerLabel.Text = "0:00"
            
            -- Flash สีแดง
            for i = 1, 6 do
                TweenService:Create(timerFrame, TweenInfo.new(0.1), {
                    BackgroundColor3 = i % 2 == 0 and 
                        Color3.fromRGB(20, 20, 30) or Color3.fromRGB(200, 50, 50)
                }):Play()
                task.wait(0.1)
            end
            
            if onComplete then onComplete() end
            return
        end
        
        -- Format time
        local mins = math.floor(remaining / 60)
        local secs = math.floor(remaining % 60)
        timerLabel.Text = string.format("%d:%02d", mins, secs)
        
        -- เปลี่ยนสีตามเวลาที่เหลือ
        if remaining <= 10 then
            timerLabel.TextColor3 = Color3.fromRGB(255, 80, 80)
            timerFrame.BackgroundColor3 = Color3.fromRGB(60, 20, 20)
        elseif remaining <= 30 then
            timerLabel.TextColor3 = Color3.fromRGB(255, 200, 0)
            timerFrame.BackgroundColor3 = Color3.fromRGB(50, 40, 10)
        else
            timerLabel.TextColor3 = Color3.new(1, 1, 1)
            timerFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
        end
    end)
    
    return connection
end

-- เริ่มนับถอยหลัง 5 นาที
startCountdown(300, function()
    print("หมดเวลาแล้ว!")
end)
```

---

## 31.8 Score Display

```lua
-- LocalScript: Score Display
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")

local LocalPlayer = Players.LocalPlayer

local screenGui = Instance.new("ScreenGui")
screenGui.Parent = LocalPlayer.PlayerGui

-- Score Container
local scoreFrame = Instance.new("Frame")
scoreFrame.Size = UDim2.new(0, 180, 0, 70)
scoreFrame.Position = UDim2.new(1, -190, 0, 10)
scoreFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
scoreFrame.BorderSizePixel = 0
scoreFrame.Parent = screenGui

local scoreCorner = Instance.new("UICorner")
scoreCorner.CornerRadius = UDim.new(0, 10)
scoreCorner.Parent = scoreFrame

-- "คะแนน" Label
local scoreTitleLabel = Instance.new("TextLabel")
scoreTitleLabel.Size = UDim2.new(1, 0, 0, 25)
scoreTitleLabel.BackgroundTransparency = 1
scoreTitleLabel.Text = "⭐ คะแนน"
scoreTitleLabel.TextColor3 = Color3.fromRGB(200, 200, 100)
scoreTitleLabel.Font = Enum.Font.GothamBold
scoreTitleLabel.TextSize = 13
scoreTitleLabel.Parent = scoreFrame

-- Score Number
local scoreLabel = Instance.new("TextLabel")
scoreLabel.Size = UDim2.new(1, 0, 0, 45)
scoreLabel.Position = UDim2.new(0, 0, 0, 22)
scoreLabel.BackgroundTransparency = 1
scoreLabel.Text = "0"
scoreLabel.TextColor3 = Color3.new(1, 1, 1)
scoreLabel.Font = Enum.Font.GothamBlack
scoreLabel.TextSize = 32
scoreLabel.Parent = scoreFrame

-- เพิ่ม Score พร้อม Animation
local currentScore = 0
local displayScore = 0

local function addScore(amount)
    currentScore = currentScore + amount
    
    -- แสดง +Score popup
    local popup = Instance.new("TextLabel")
    popup.Size = UDim2.new(0, 100, 0, 30)
    popup.Position = UDim2.new(0.5, -50, 0.4, 0)
    popup.BackgroundTransparency = 1
    popup.Text = "+" .. amount
    popup.TextColor3 = Color3.fromRGB(255, 220, 50)
    popup.Font = Enum.Font.GothamBold
    popup.TextSize = 22
    popup.AnchorPoint = Vector2.new(0.5, 0.5)
    popup.Parent = screenGui
    
    -- Animate popup
    TweenService:Create(popup, TweenInfo.new(1, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
        Position = UDim2.new(0.5, -50, 0.3, 0),
        TextTransparency = 1
    }):Play()
    
    task.delay(1, function() popup:Destroy() end)
    
    -- Update score display (count up animation)
    local startScore = displayScore
    local startTime = tick()
    local duration = 0.5
    
    local connection
    local RunService = game:GetService("RunService")
    connection = RunService.Heartbeat:Connect(function()
        local elapsed = tick() - startTime
        local t = math.min(elapsed / duration, 1)
        
        -- Ease out
        t = 1 - (1 - t)^3
        
        displayScore = math.floor(startScore + (currentScore - startScore) * t)
        scoreLabel.Text = tostring(displayScore)
        
        if t >= 1 then
            displayScore = currentScore
            scoreLabel.Text = tostring(currentScore)
            connection:Disconnect()
        end
    end)
end

-- ตัวอย่าง
task.spawn(function()
    wait(1)
    addScore(100)
    wait(0.5)
    addScore(250)
    wait(0.5)
    addScore(75)
    wait(0.5)
    addScore(500)
end)
```

---

## 31.9 TextBox Input

```lua
-- LocalScript: TextBox
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")

local LocalPlayer = Players.LocalPlayer

local screenGui = Instance.new("ScreenGui")
screenGui.Parent = LocalPlayer.PlayerGui

-- Input Field
local inputFrame = Instance.new("Frame")
inputFrame.Size = UDim2.new(0, 300, 0, 50)
inputFrame.AnchorPoint = Vector2.new(0.5, 0.5)
inputFrame.Position = UDim2.new(0.5, 0, 0.5, 0)
inputFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
inputFrame.BorderSizePixel = 0
inputFrame.Parent = screenGui

local inputCorner = Instance.new("UICorner")
inputCorner.CornerRadius = UDim.new(0, 8)
inputCorner.Parent = inputFrame

local inputStroke = Instance.new("UIStroke")
inputStroke.Color = Color3.fromRGB(80, 80, 120)
inputStroke.Thickness = 2
inputStroke.Parent = inputFrame

-- TextBox
local textBox = Instance.new("TextBox")
textBox.Size = UDim2.new(1, -20, 1, -10)
textBox.Position = UDim2.new(0, 10, 0, 5)
textBox.BackgroundTransparency = 1
textBox.Text = ""
textBox.PlaceholderText = "พิมพ์ชื่อตัวละคร..."
textBox.PlaceholderColor3 = Color3.fromRGB(120, 120, 140)
textBox.TextColor3 = Color3.new(1, 1, 1)
textBox.Font = Enum.Font.Gotham
textBox.TextSize = 16
textBox.ClearTextOnFocus = true  -- ลบข้อความเมื่อ Focus
textBox.TextXAlignment = Enum.TextXAlignment.Left
textBox.Parent = inputFrame

-- Events
textBox.Focused:Connect(function()
    -- ไฮไลต์เมื่อ Focus
    TweenService:Create(inputStroke, TweenInfo.new(0.2), {
        Color = Color3.fromRGB(100, 150, 250)
    }):Play()
end)

textBox.FocusLost:Connect(function(enterPressed)
    -- กลับสีปกติ
    TweenService:Create(inputStroke, TweenInfo.new(0.2), {
        Color = Color3.fromRGB(80, 80, 120)
    }):Play()
    
    if enterPressed then
        local input = textBox.Text
        if input ~= "" then
            print("ป้อนข้อมูล:", input)
        end
    end
end)

-- จำกัดความยาว
textBox:GetPropertyChangedSignal("Text"):Connect(function()
    if #textBox.Text > 20 then
        textBox.Text = string.sub(textBox.Text, 1, 20)
    end
end)
```

---

## 31.10 แบบฝึกหัด

### แบบฝึกหัดที่ 1: Chat System

```lua
-- LocalScript: Chat System UI
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local TweenService = game:GetService("TweenService")

local screenGui = Instance.new("ScreenGui")
screenGui.Parent = LocalPlayer.PlayerGui

-- Chat Container
local chatFrame = Instance.new("Frame")
chatFrame.Size = UDim2.new(0, 350, 0, 400)
chatFrame.Position = UDim2.new(0, 10, 1, -420)
chatFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
chatFrame.BackgroundTransparency = 0.2
chatFrame.BorderSizePixel = 0
chatFrame.Parent = screenGui

local chatCorner = Instance.new("UICorner")
chatCorner.CornerRadius = UDim.new(0, 10)
chatCorner.Parent = chatFrame

-- Message List
local messageScroll = Instance.new("ScrollingFrame")
messageScroll.Size = UDim2.new(1, -10, 1, -55)
messageScroll.Position = UDim2.new(0, 5, 0, 5)
messageScroll.BackgroundTransparency = 1
messageScroll.ScrollBarThickness = 3
messageScroll.AutomaticCanvasSize = Enum.AutomaticSize.Y
messageScroll.CanvasSize = UDim2.new(0, 0, 0, 0)
messageScroll.Parent = chatFrame

local msgList = Instance.new("UIListLayout")
msgList.Padding = UDim.new(0, 3)
msgList.SortOrder = Enum.SortOrder.LayoutOrder
msgList.Parent = messageScroll

-- Input Area
local inputArea = Instance.new("Frame")
inputArea.Size = UDim2.new(1, 0, 0, 45)
inputArea.Position = UDim2.new(0, 0, 1, -45)
inputArea.BackgroundColor3 = Color3.fromRGB(30, 30, 45)
inputArea.BorderSizePixel = 0
inputArea.Parent = chatFrame

local inputBox = Instance.new("TextBox")
inputBox.Size = UDim2.new(1, -60, 1, -10)
inputBox.Position = UDim2.new(0, 5, 0, 5)
inputBox.BackgroundColor3 = Color3.fromRGB(45, 45, 60)
inputBox.BorderSizePixel = 0
inputBox.Text = ""
inputBox.PlaceholderText = "พิมพ์ข้อความ..."
inputBox.PlaceholderColor3 = Color3.fromRGB(120, 120, 140)
inputBox.TextColor3 = Color3.new(1, 1, 1)
inputBox.Font = Enum.Font.Gotham
inputBox.TextSize = 14
inputBox.TextXAlignment = Enum.TextXAlignment.Left
inputBox.ClearTextOnFocus = false
inputBox.Parent = inputArea

local inputCorner = Instance.new("UICorner")
inputCorner.CornerRadius = UDim.new(0, 6)
inputCorner.Parent = inputBox

local sendBtn = Instance.new("TextButton")
sendBtn.Size = UDim2.new(0, 50, 1, -10)
sendBtn.Position = UDim2.new(1, -55, 0, 5)
sendBtn.BackgroundColor3 = Color3.fromRGB(80, 130, 220)
sendBtn.Text = "ส่ง"
sendBtn.TextColor3 = Color3.new(1, 1, 1)
sendBtn.Font = Enum.Font.GothamBold
sendBtn.TextSize = 13
sendBtn.BorderSizePixel = 0
sendBtn.Parent = inputArea

local sendBtnCorner = Instance.new("UICorner")
sendBtnCorner.CornerRadius = UDim.new(0, 6)
sendBtnCorner.Parent = sendBtn

-- Message Colors
local playerColors = {}
local colorPalette = {
    Color3.fromRGB(255, 150, 150),
    Color3.fromRGB(150, 255, 150),
    Color3.fromRGB(150, 150, 255),
    Color3.fromRGB(255, 255, 150),
    Color3.fromRGB(255, 150, 255),
    Color3.fromRGB(150, 255, 255),
}

local function getPlayerColor(playerName)
    if not playerColors[playerName] then
        local index = (#playerColors % #colorPalette) + 1
        playerColors[playerName] = colorPalette[index]
    end
    return playerColors[playerName]
end

local msgCount = 0

local function addMessage(playerName, message)
    msgCount = msgCount + 1
    
    local msgFrame = Instance.new("Frame")
    msgFrame.Size = UDim2.new(1, -5, 0, 0)
    msgFrame.AutomaticSize = Enum.AutomaticSize.Y
    msgFrame.BackgroundTransparency = 1
    msgFrame.LayoutOrder = msgCount
    msgFrame.Parent = messageScroll
    
    local msgLabel = Instance.new("TextLabel")
    msgLabel.Size = UDim2.new(1, -10, 0, 0)
    msgLabel.Position = UDim2.new(0, 5, 0, 2)
    msgLabel.BackgroundTransparency = 1
    msgLabel.AutomaticSize = Enum.AutomaticSize.Y
    msgLabel.RichText = true
    msgLabel.Text = string.format(
        "<font color='rgb(%d,%d,%d)'><b>%s:</b></font> %s",
        getPlayerColor(playerName).R * 255,
        getPlayerColor(playerName).G * 255,
        getPlayerColor(playerName).B * 255,
        playerName,
        message
    )
    msgLabel.TextColor3 = Color3.fromRGB(220, 220, 220)
    msgLabel.Font = Enum.Font.Gotham
    msgLabel.TextSize = 13
    msgLabel.TextWrapped = true
    msgLabel.TextXAlignment = Enum.TextXAlignment.Left
    msgLabel.TextYAlignment = Enum.TextYAlignment.Top
    msgLabel.Parent = msgFrame
    
    -- Scroll to bottom
    task.wait()
    messageScroll.CanvasPosition = Vector2.new(0, messageScroll.AbsoluteCanvasSize.Y)
end

-- Send Message
local function sendMessage()
    local msg = inputBox.Text
    if msg == "" or msg:gsub("%s", "") == "" then return end
    
    addMessage(LocalPlayer.Name, msg)
    inputBox.Text = ""
end

sendBtn.MouseButton1Click:Connect(sendMessage)

inputBox.FocusLost:Connect(function(enterPressed)
    if enterPressed then
        sendMessage()
        inputBox:CaptureFocus()
    end
end)

-- ตัวอย่างข้อความ
addMessage("System", "ยินดีต้อนรับสู่เกม!")
addMessage("PlayerA", "สวัสดีทุกคน!")
addMessage("PlayerB", "เฮ้! เราจะไปทำ Quest ไหม?")
```

---

## 31.11 สรุป

ในบทนี้เราได้เรียนรู้:

1. **TextLabel Properties** - Text, Font, TextSize, Color, Alignment
2. **Rich Text Formatting** - b, i, u, font color, stroke
3. **Fonts** - ทุก Font ที่ใช้ได้ใน Roblox
4. **TextButton Properties** - AutoButtonColor, Events
5. **สร้างปุ่มสวยงาม** - Primary, Outlined, Danger, Success, Icon
6. **Animated Text** - Typewriter Effect
7. **Countdown Timer** - นับถอยหลังพร้อม Animation
8. **Score Display** - แสดงคะแนนพร้อม Count Up Animation
9. **TextBox** - รับ Input จากผู้เล่น
10. **Chat System** - ระบบ Chat อย่างง่าย

ในบทต่อไป เราจะเรียนรู้เกี่ยวกับ ImageLabel และ ImageButton

---

## แหล่งอ้างอิง

- [Roblox Developer Hub - TextLabel](https://developer.roblox.com/en-us/api-reference/class/TextLabel)
- [Roblox Developer Hub - TextButton](https://developer.roblox.com/en-us/api-reference/class/TextButton)
- [Roblox Developer Hub - Rich Text](https://developer.roblox.com/en-us/articles/gui-rich-text)
