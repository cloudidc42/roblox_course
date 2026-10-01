# ตอนที่ 30: ScreenGui และ Frame Elements

## บทนำ

ในบทนี้เราจะเจาะลึกเกี่ยวกับ ScreenGui และ Frame ซึ่งเป็นพื้นฐานสำคัญของการสร้าง UI ใน Roblox เราจะเรียนรู้ Properties ทั้งหมด วิธีการจัดการ Layout และการสร้าง UI Components ที่ใช้งานได้จริง

---

## 30.1 ScreenGui Properties

```lua
-- LocalScript
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

local screenGui = Instance.new("ScreenGui")

-- Properties สำคัญ
screenGui.Name = "MainUI"
screenGui.Enabled = true              -- เปิด/ปิด GUI ทั้งหมด
screenGui.ResetOnSpawn = false        -- ไม่ลบเมื่อ Spawn ใหม่ (สำคัญ!)
screenGui.IgnoreGuiInset = false      -- false = คำนวณ Safe Zone (top bar)
screenGui.DisplayOrder = 1            -- ลำดับการแสดง (สูงกว่า = อยู่บนสุด)
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
-- Sibling = ZIndex ใช้ภายใน ScreenGui นั้นๆ
-- Global = ZIndex ใช้ global ทั้งหมด

screenGui.Parent = LocalPlayer.PlayerGui

-- เปิด/ปิด
screenGui.Enabled = false  -- ซ่อน
screenGui.Enabled = true   -- แสดง
```

---

## 30.2 Frame Properties ทั้งหมด

```lua
local frame = Instance.new("Frame")

-- ขนาดและตำแหน่ง
frame.Size = UDim2.new(0, 300, 0, 200)
frame.Position = UDim2.new(0.5, -150, 0.5, -100)
frame.AnchorPoint = Vector2.new(0.5, 0.5)
frame.SizeConstraint = Enum.SizeConstraint.RelativeXY  -- ขนาดตาม XY ของ Parent

-- สี
frame.BackgroundColor3 = Color3.fromRGB(50, 50, 60)
frame.BackgroundTransparency = 0  -- 0=ทึบ, 1=โปร่งใส

-- ขอบ
frame.BorderSizePixel = 2
frame.BorderColor3 = Color3.fromRGB(100, 100, 120)
frame.BorderMode = Enum.BorderMode.Outline  -- Outline, Inset, Middle

-- ZIndex
frame.ZIndex = 1

-- Clipping
frame.ClipsDescendants = false  -- ตัด children ที่เกินขอบหรือเปล่า

-- Interaction
frame.Active = true   -- รับ Events หรือเปล่า
frame.Selectable = false  -- สำหรับ Gamepad

-- Rotation
frame.Rotation = 45  -- หมุน 45 องศา

-- Auto Size
frame.AutomaticSize = Enum.AutomaticSize.XY  -- ปรับขนาดตาม Content

frame.Parent = screenGui
```

---

## 30.3 UICorner - มุมโค้งมน

```lua
local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 200, 0, 100)
frame.BackgroundColor3 = Color3.fromRGB(70, 130, 200)
frame.Parent = screenGui

-- เพิ่มมุมโค้ง
local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 10)   -- โค้ง 10px
-- หรือ
corner.CornerRadius = UDim.new(0.5, 0)  -- โค้งเป็นวงกลม (50% ของขนาดน้อยที่สุด)
corner.Parent = frame

-- ตัวอย่าง: สร้างวงกลม
local circle = Instance.new("Frame")
circle.Size = UDim2.new(0, 100, 0, 100)
circle.BackgroundColor3 = Color3.fromRGB(255, 100, 100)
circle.Parent = screenGui

local circleCorner = Instance.new("UICorner")
circleCorner.CornerRadius = UDim.new(0.5, 0)  -- 50% = วงกลม
circleCorner.Parent = circle
```

---

## 30.4 UIStroke - ขอบ

```lua
local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 200, 0, 100)
frame.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
frame.BorderSizePixel = 0  -- ปิดขอบเดิม
frame.Parent = screenGui

-- UIStroke
local stroke = Instance.new("UIStroke")
stroke.Color = Color3.fromRGB(150, 200, 255)
stroke.Thickness = 3
stroke.Transparency = 0  -- ทึบ
stroke.LineJoinMode = Enum.LineJoinMode.Round  -- มุมขอบแบบโค้ง
stroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border  -- Border หรือ Contextual
stroke.Parent = frame

-- UIStroke กับ Text
local label = Instance.new("TextLabel")
label.Text = "ข้อความมีขอบ"
label.Parent = screenGui

local textStroke = Instance.new("UIStroke")
textStroke.Color = Color3.new(0, 0, 0)
textStroke.Thickness = 2
textStroke.Parent = label
```

---

## 30.5 UIGradient - ไล่สี

```lua
-- Gradient บน Frame
local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 300, 0, 150)
frame.BackgroundColor3 = Color3.new(1, 1, 1)  -- สีพื้น
frame.Parent = screenGui

local gradient = Instance.new("UIGradient")

-- สีไล่
gradient.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 100, 100)),   -- แดง
    ColorSequenceKeypoint.new(0.5, Color3.fromRGB(100, 100, 255)), -- น้ำเงิน
    ColorSequenceKeypoint.new(1, Color3.fromRGB(100, 255, 100)),   -- เขียว
})

-- Transparency Gradient
gradient.Transparency = NumberSequence.new({
    NumberSequenceKeypoint.new(0, 0),    -- ทึบ
    NumberSequenceKeypoint.new(0.5, 0.3),
    NumberSequenceKeypoint.new(1, 1),    -- โปร่งใส
})

gradient.Rotation = 90  -- หมุน Gradient (0 = แนวนอน, 90 = แนวตั้ง)
gradient.Offset = Vector2.new(0, 0)  -- เลื่อน Gradient
gradient.Parent = frame
```

---

## 30.6 UIScale - ปรับขนาด

```lua
local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 200, 0, 100)
frame.Parent = screenGui

-- UIScale ปรับขนาดทั้ง Frame และ children ของมัน
local scale = Instance.new("UIScale")
scale.Scale = 1.5  -- ขยาย 1.5 เท่า
scale.Parent = frame

-- ใช้สำหรับ Hover Effect
frame.MouseEnter:Connect(function()
    TweenService:Create(scale, TweenInfo.new(0.1), {Scale = 1.1}):Play()
end)

frame.MouseLeave:Connect(function()
    TweenService:Create(scale, TweenInfo.new(0.1), {Scale = 1.0}):Play()
end)
```

---

## 30.7 UISizeConstraint - จำกัดขนาด

```lua
local frame = Instance.new("Frame")
frame.Size = UDim2.new(0.5, 0, 0.5, 0)  -- 50% ของ Parent
frame.Parent = screenGui

-- จำกัดขนาดไม่ให้เล็กหรือใหญ่เกินไป
local sizeConstraint = Instance.new("UISizeConstraint")
sizeConstraint.MinSize = Vector2.new(200, 100)   -- ขนาดต่ำสุด
sizeConstraint.MaxSize = Vector2.new(800, 400)   -- ขนาดสูงสุด
sizeConstraint.Parent = frame
```

---

## 30.8 UIAspectRatioConstraint - คงอัตราส่วน

```lua
local frame = Instance.new("Frame")
frame.Size = UDim2.new(0.5, 0, 0, 0)  -- กว้าง 50%
frame.Parent = screenGui

-- คงอัตราส่วน 16:9
local aspectRatio = Instance.new("UIAspectRatioConstraint")
aspectRatio.AspectRatio = 16/9
aspectRatio.AspectType = Enum.AspectType.FitWithinMaxSize
aspectRatio.DominantAxis = Enum.DominantAxis.Width
aspectRatio.Parent = frame
```

---

## 30.9 การสร้าง Modal Dialog

```lua
-- LocalScript: Modal Dialog System
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")

local LocalPlayer = Players.LocalPlayer

local screenGui = Instance.new("ScreenGui")
screenGui.DisplayOrder = 50  -- อยู่บนสุด
screenGui.Parent = LocalPlayer.PlayerGui

local ModalSystem = {}

function ModalSystem.show(options)
    -- Backdrop (พื้นหลังมืด)
    local backdrop = Instance.new("Frame")
    backdrop.Size = UDim2.new(1, 0, 1, 0)
    backdrop.BackgroundColor3 = Color3.new(0, 0, 0)
    backdrop.BackgroundTransparency = 1  -- เริ่มโปร่งใส
    backdrop.BorderSizePixel = 0
    backdrop.ZIndex = 10
    backdrop.Parent = screenGui
    
    -- Fade In backdrop
    TweenService:Create(
        backdrop,
        TweenInfo.new(0.3),
        {BackgroundTransparency = 0.5}
    ):Play()
    
    -- Dialog Box
    local dialog = Instance.new("Frame")
    dialog.Size = UDim2.new(0, 400, 0, 0)  -- เริ่มสูง 0
    dialog.AnchorPoint = Vector2.new(0.5, 0.5)
    dialog.Position = UDim2.new(0.5, 0, 0.5, 0)
    dialog.BackgroundColor3 = Color3.fromRGB(35, 35, 45)
    dialog.BorderSizePixel = 0
    dialog.ZIndex = 11
    dialog.AutomaticSize = Enum.AutomaticSize.Y
    dialog.Parent = screenGui
    
    local dialogCorner = Instance.new("UICorner")
    dialogCorner.CornerRadius = UDim.new(0, 12)
    dialogCorner.Parent = dialog
    
    -- UIStroke
    local dialogStroke = Instance.new("UIStroke")
    dialogStroke.Color = Color3.fromRGB(100, 100, 130)
    dialogStroke.Thickness = 1
    dialogStroke.Parent = dialog
    
    -- Content Container
    local content = Instance.new("Frame")
    content.Size = UDim2.new(1, 0, 0, 0)
    content.BackgroundTransparency = 1
    content.AutomaticSize = Enum.AutomaticSize.Y
    content.Parent = dialog
    
    local contentList = Instance.new("UIListLayout")
    contentList.Padding = UDim.new(0, 10)
    contentList.Parent = content
    
    local contentPadding = Instance.new("UIPadding")
    contentPadding.PaddingTop = UDim.new(0, 20)
    contentPadding.PaddingBottom = UDim.new(0, 20)
    contentPadding.PaddingLeft = UDim.new(0, 20)
    contentPadding.PaddingRight = UDim.new(0, 20)
    contentPadding.Parent = content
    
    -- Title
    if options.title then
        local titleLabel = Instance.new("TextLabel")
        titleLabel.Size = UDim2.new(1, 0, 0, 30)
        titleLabel.BackgroundTransparency = 1
        titleLabel.Text = options.title
        titleLabel.TextColor3 = Color3.new(1, 1, 1)
        titleLabel.Font = Enum.Font.GothamBold
        titleLabel.TextSize = 22
        titleLabel.TextXAlignment = Enum.TextXAlignment.Left
        titleLabel.Parent = content
    end
    
    -- Divider
    local divider = Instance.new("Frame")
    divider.Size = UDim2.new(1, 0, 0, 1)
    divider.BackgroundColor3 = Color3.fromRGB(80, 80, 100)
    divider.BorderSizePixel = 0
    divider.Parent = content
    
    -- Message
    if options.message then
        local msgLabel = Instance.new("TextLabel")
        msgLabel.Size = UDim2.new(1, 0, 0, 0)
        msgLabel.BackgroundTransparency = 1
        msgLabel.Text = options.message
        msgLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
        msgLabel.Font = Enum.Font.Gotham
        msgLabel.TextSize = 16
        msgLabel.TextWrapped = true
        msgLabel.AutomaticSize = Enum.AutomaticSize.Y
        msgLabel.TextXAlignment = Enum.TextXAlignment.Left
        msgLabel.Parent = content
    end
    
    -- Buttons
    local buttonRow = Instance.new("Frame")
    buttonRow.Size = UDim2.new(1, 0, 0, 40)
    buttonRow.BackgroundTransparency = 1
    buttonRow.Parent = content
    
    local buttonLayout = Instance.new("UIListLayout")
    buttonLayout.FillDirection = Enum.FillDirection.Horizontal
    buttonLayout.HorizontalAlignment = Enum.HorizontalAlignment.Right
    buttonLayout.Padding = UDim.new(0, 10)
    buttonLayout.Parent = buttonRow
    
    -- Callback เมื่อปิด
    local function closeModal(result)
        TweenService:Create(backdrop, TweenInfo.new(0.2), {BackgroundTransparency = 1}):Play()
        TweenService:Create(dialog, TweenInfo.new(0.2), {BackgroundTransparency = 1}):Play()
        wait(0.2)
        backdrop:Destroy()
        dialog:Destroy()
        if options.onClose then
            options.onClose(result)
        end
    end
    
    -- สร้างปุ่ม
    for _, btnConfig in ipairs(options.buttons or {{text = "ตกลง", style = "primary"}}) do
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(0, 100, 1, 0)
        btn.Text = btnConfig.text
        btn.Font = Enum.Font.GothamBold
        btn.TextSize = 14
        btn.BorderSizePixel = 0
        
        if btnConfig.style == "primary" then
            btn.BackgroundColor3 = Color3.fromRGB(80, 130, 220)
            btn.TextColor3 = Color3.new(1, 1, 1)
        elseif btnConfig.style == "danger" then
            btn.BackgroundColor3 = Color3.fromRGB(200, 60, 60)
            btn.TextColor3 = Color3.new(1, 1, 1)
        else
            btn.BackgroundColor3 = Color3.fromRGB(70, 70, 90)
            btn.TextColor3 = Color3.fromRGB(200, 200, 200)
        end
        
        btn.Parent = buttonRow
        
        local btnCorner = Instance.new("UICorner")
        btnCorner.CornerRadius = UDim.new(0, 6)
        btnCorner.Parent = btn
        
        btn.MouseButton1Click:Connect(function()
            closeModal(btnConfig.result or btnConfig.text)
        end)
    end
    
    -- Scale in animation
    dialog.Size = UDim2.new(0, 400, 0, 0)
    TweenService:Create(
        dialog,
        TweenInfo.new(0.3, Enum.EasingStyle.Back, Enum.EasingDirection.Out),
        {Size = UDim2.new(0, 400, 0, 200)}
    ):Play()
    
    return dialog
end

-- ตัวอย่างการใช้
ModalSystem.show({
    title = "ยืนยันการลบ",
    message = "คุณต้องการลบ Item นี้จริงหรือไม่? การกระทำนี้ไม่สามารถย้อนกลับได้",
    buttons = {
        {text = "ยกเลิก", style = "secondary", result = false},
        {text = "ลบ", style = "danger", result = true},
    },
    onClose = function(result)
        if result then
            print("ลบ Item แล้ว!")
        else
            print("ยกเลิก")
        end
    end
})
```

---

## 30.10 Notification System

```lua
-- LocalScript: Notification System
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")

local LocalPlayer = Players.LocalPlayer

local notifGui = Instance.new("ScreenGui")
notifGui.DisplayOrder = 100
notifGui.ResetOnSpawn = false
notifGui.Parent = LocalPlayer.PlayerGui

local notifContainer = Instance.new("Frame")
notifContainer.Size = UDim2.new(0, 300, 1, 0)
notifContainer.Position = UDim2.new(1, -310, 0, 0)
notifContainer.BackgroundTransparency = 1
notifContainer.Parent = notifGui

local notifList = Instance.new("UIListLayout")
notifList.VerticalAlignment = Enum.VerticalAlignment.Bottom
notifList.Padding = UDim.new(0, 8)
notifList.Parent = notifContainer

local notifPadding = Instance.new("UIPadding")
notifPadding.PaddingBottom = UDim.new(0, 10)
notifPadding.PaddingRight = UDim.new(0, 10)
notifPadding.Parent = notifContainer

-- สีตามประเภท
local notifColors = {
    success = Color3.fromRGB(40, 160, 80),
    warning = Color3.fromRGB(200, 150, 0),
    error = Color3.fromRGB(180, 50, 50),
    info = Color3.fromRGB(60, 120, 200),
}

local notifIcons = {
    success = "✅",
    warning = "⚠️",
    error = "❌",
    info = "ℹ️",
}

local function showNotification(message, type, duration)
    type = type or "info"
    duration = duration or 3
    
    -- สร้าง Notification
    local notif = Instance.new("Frame")
    notif.Size = UDim2.new(1, 0, 0, 60)
    notif.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
    notif.BorderSizePixel = 0
    notif.Position = UDim2.new(1, 20, 0, 0)  -- เริ่มนอกจอ
    notif.Parent = notifContainer
    
    -- Corner
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 8)
    corner.Parent = notif
    
    -- Left Accent Bar
    local accent = Instance.new("Frame")
    accent.Size = UDim2.new(0, 4, 1, 0)
    accent.BackgroundColor3 = notifColors[type]
    accent.BorderSizePixel = 0
    accent.Parent = notif
    
    local accentCorner = Instance.new("UICorner")
    accentCorner.CornerRadius = UDim.new(0, 2)
    accentCorner.Parent = accent
    
    -- Icon
    local icon = Instance.new("TextLabel")
    icon.Size = UDim2.new(0, 40, 1, 0)
    icon.Position = UDim2.new(0, 10, 0, 0)
    icon.BackgroundTransparency = 1
    icon.Text = notifIcons[type]
    icon.TextSize = 20
    icon.Parent = notif
    
    -- Message
    local msgLabel = Instance.new("TextLabel")
    msgLabel.Size = UDim2.new(1, -65, 1, 0)
    msgLabel.Position = UDim2.new(0, 55, 0, 0)
    msgLabel.BackgroundTransparency = 1
    msgLabel.Text = message
    msgLabel.TextColor3 = Color3.new(1, 1, 1)
    msgLabel.Font = Enum.Font.Gotham
    msgLabel.TextSize = 14
    msgLabel.TextWrapped = true
    msgLabel.TextXAlignment = Enum.TextXAlignment.Left
    msgLabel.Parent = notif
    
    -- Progress Bar
    local progressBg = Instance.new("Frame")
    progressBg.Size = UDim2.new(1, 0, 0, 3)
    progressBg.Position = UDim2.new(0, 0, 1, -3)
    progressBg.BackgroundColor3 = Color3.fromRGB(50, 50, 60)
    progressBg.BorderSizePixel = 0
    progressBg.Parent = notif
    
    local progress = Instance.new("Frame")
    progress.Size = UDim2.new(1, 0, 1, 0)
    progress.BackgroundColor3 = notifColors[type]
    progress.BorderSizePixel = 0
    progress.Parent = progressBg
    
    -- Slide In
    TweenService:Create(
        notif,
        TweenInfo.new(0.3, Enum.EasingStyle.Back, Enum.EasingDirection.Out),
        {Position = UDim2.new(0, 0, 0, 0)}
    ):Play()
    
    -- Progress Bar Tween
    TweenService:Create(
        progress,
        TweenInfo.new(duration, Enum.EasingStyle.Linear),
        {Size = UDim2.new(0, 0, 1, 0)}
    ):Play()
    
    -- Slide Out หลัง duration
    task.delay(duration, function()
        local slideOut = TweenService:Create(
            notif,
            TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.In),
            {Position = UDim2.new(1, 20, 0, 0)}
        )
        slideOut:Play()
        slideOut.Completed:Connect(function()
            notif:Destroy()
        end)
    end)
    
    return notif
end

-- ตัวอย่างการใช้
showNotification("บันทึกข้อมูลสำเร็จ!", "success")
wait(1)
showNotification("ชีวิตน้อย! ระวัง!", "warning")
wait(1)
showNotification("ไม่มี Connection", "error")
wait(1)
showNotification("ผู้เล่นใหม่เข้ามาร่วม!", "info")
```

---

## 30.11 Tab System

```lua
-- LocalScript: Tab System
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")

local LocalPlayer = Players.LocalPlayer

local screenGui = Instance.new("ScreenGui")
screenGui.Parent = LocalPlayer.PlayerGui

-- Main Panel
local panel = Instance.new("Frame")
panel.Size = UDim2.new(0, 500, 0, 400)
panel.AnchorPoint = Vector2.new(0.5, 0.5)
panel.Position = UDim2.new(0.5, 0, 0.5, 0)
panel.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
panel.BorderSizePixel = 0
panel.Parent = screenGui

local panelCorner = Instance.new("UICorner")
panelCorner.CornerRadius = UDim.new(0, 12)
panelCorner.Parent = panel

-- Tab Bar
local tabBar = Instance.new("Frame")
tabBar.Size = UDim2.new(1, 0, 0, 45)
tabBar.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
tabBar.BorderSizePixel = 0
tabBar.Parent = panel

local tabBarCorner = Instance.new("UICorner")
tabBarCorner.CornerRadius = UDim.new(0, 12)
tabBarCorner.Parent = tabBar

local tabLayout = Instance.new("UIListLayout")
tabLayout.FillDirection = Enum.FillDirection.Horizontal
tabLayout.Padding = UDim.new(0, 2)
tabLayout.Parent = tabBar

local tabPadding = Instance.new("UIPadding")
tabPadding.PaddingLeft = UDim.new(0, 5)
tabPadding.PaddingRight = UDim.new(0, 5)
tabPadding.PaddingTop = UDim.new(0, 5)
tabPadding.PaddingBottom = UDim.new(0, 5)
tabPadding.Parent = tabBar

-- Content Area
local contentArea = Instance.new("Frame")
contentArea.Size = UDim2.new(1, -20, 1, -65)
contentArea.Position = UDim2.new(0, 10, 0, 55)
contentArea.BackgroundTransparency = 1
contentArea.Parent = panel

-- Tab Data
local tabs = {
    {name = "หน้าหลัก", icon = "🏠", content = "นี่คือหน้าหลัก"},
    {name = "โปรไฟล์", icon = "👤", content = "ข้อมูลโปรไฟล์ของคุณ"},
    {name = "ร้านค้า", icon = "🛒", content = "ร้านค้าในเกม"},
    {name = "ตั้งค่า", icon = "⚙️", content = "การตั้งค่าเกม"},
}

local tabButtons = {}
local tabContents = {}
local activeTab = nil

-- สร้างแต่ละ Tab
for i, tabData in ipairs(tabs) do
    -- Tab Button
    local tabBtn = Instance.new("TextButton")
    tabBtn.Size = UDim2.new(0, 110, 1, 0)
    tabBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
    tabBtn.BorderSizePixel = 0
    tabBtn.Text = tabData.icon .. " " .. tabData.name
    tabBtn.TextColor3 = Color3.fromRGB(150, 150, 170)
    tabBtn.Font = Enum.Font.Gotham
    tabBtn.TextSize = 13
    tabBtn.LayoutOrder = i
    tabBtn.Parent = tabBar
    
    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 8)
    btnCorner.Parent = tabBtn
    
    -- Tab Content
    local tabContent = Instance.new("Frame")
    tabContent.Size = UDim2.new(1, 0, 1, 0)
    tabContent.BackgroundTransparency = 1
    tabContent.Visible = false
    tabContent.Parent = contentArea
    
    -- Content Label
    local contentLabel = Instance.new("TextLabel")
    contentLabel.Size = UDim2.new(1, 0, 0, 50)
    contentLabel.Position = UDim2.new(0, 0, 0.3, 0)
    contentLabel.BackgroundTransparency = 1
    contentLabel.Text = tabData.content
    contentLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
    contentLabel.Font = Enum.Font.Gotham
    contentLabel.TextSize = 18
    contentLabel.Parent = tabContent
    
    tabButtons[i] = tabBtn
    tabContents[i] = tabContent
    
    -- เลือก Tab
    tabBtn.MouseButton1Click:Connect(function()
        selectTab(i)
    end)
end

-- ฟังก์ชันเลือก Tab
function selectTab(index)
    -- ซ่อน Tab เก่า
    if activeTab then
        tabContents[activeTab].Visible = false
        TweenService:Create(
            tabButtons[activeTab],
            TweenInfo.new(0.2),
            {
                BackgroundColor3 = Color3.fromRGB(30, 30, 40),
                TextColor3 = Color3.fromRGB(150, 150, 170)
            }
        ):Play()
    end
    
    -- แสดง Tab ใหม่
    activeTab = index
    tabContents[index].Visible = true
    TweenService:Create(
        tabButtons[index],
        TweenInfo.new(0.2),
        {
            BackgroundColor3 = Color3.fromRGB(80, 130, 220),
            TextColor3 = Color3.new(1, 1, 1)
        }
    ):Play()
end

-- เริ่มที่ Tab แรก
selectTab(1)
```

---

## 30.12 Draggable Window

```lua
-- LocalScript: Draggable Window
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")

local LocalPlayer = Players.LocalPlayer

local screenGui = Instance.new("ScreenGui")
screenGui.Parent = LocalPlayer.PlayerGui

-- ฟังก์ชันสร้าง Draggable Frame
local function createDraggableWindow(options)
    options = options or {}
    
    -- Window Frame
    local window = Instance.new("Frame")
    window.Size = options.size or UDim2.new(0, 350, 0, 250)
    window.Position = options.position or UDim2.new(0.5, -175, 0.5, -125)
    window.BackgroundColor3 = options.bgColor or Color3.fromRGB(35, 35, 45)
    window.BorderSizePixel = 0
    window.Active = true
    window.Parent = screenGui
    
    local windowCorner = Instance.new("UICorner")
    windowCorner.CornerRadius = UDim.new(0, 10)
    windowCorner.Parent = window
    
    local windowStroke = Instance.new("UIStroke")
    windowStroke.Color = Color3.fromRGB(80, 80, 100)
    windowStroke.Thickness = 1
    windowStroke.Parent = window
    
    -- Title Bar (พื้นที่ลาก)
    local titleBar = Instance.new("Frame")
    titleBar.Size = UDim2.new(1, 0, 0, 35)
    titleBar.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
    titleBar.BorderSizePixel = 0
    titleBar.Parent = window
    
    local titleCorner = Instance.new("UICorner")
    titleCorner.CornerRadius = UDim.new(0, 10)
    titleCorner.Parent = titleBar
    
    -- Title Text
    local titleLabel = Instance.new("TextLabel")
    titleLabel.Size = UDim2.new(1, -40, 1, 0)
    titleLabel.Position = UDim2.new(0, 10, 0, 0)
    titleLabel.BackgroundTransparency = 1
    titleLabel.Text = options.title or "Window"
    titleLabel.TextColor3 = Color3.new(1, 1, 1)
    titleLabel.Font = Enum.Font.GothamBold
    titleLabel.TextSize = 14
    titleLabel.TextXAlignment = Enum.TextXAlignment.Left
    titleLabel.Parent = titleBar
    
    -- Close Button
    local closeBtn = Instance.new("TextButton")
    closeBtn.Size = UDim2.new(0, 25, 0, 25)
    closeBtn.Position = UDim2.new(1, -30, 0.5, -12)
    closeBtn.BackgroundColor3 = Color3.fromRGB(200, 60, 60)
    closeBtn.Text = "✕"
    closeBtn.TextColor3 = Color3.new(1, 1, 1)
    closeBtn.Font = Enum.Font.GothamBold
    closeBtn.TextSize = 12
    closeBtn.BorderSizePixel = 0
    closeBtn.Parent = titleBar
    
    local closeBtnCorner = Instance.new("UICorner")
    closeBtnCorner.CornerRadius = UDim.new(0, 5)
    closeBtnCorner.Parent = closeBtn
    
    closeBtn.MouseButton1Click:Connect(function()
        window:Destroy()
    end)
    
    -- Content Area
    local content = Instance.new("Frame")
    content.Size = UDim2.new(1, -20, 1, -45)
    content.Position = UDim2.new(0, 10, 0, 40)
    content.BackgroundTransparency = 1
    content.Parent = window
    
    -- Drag Logic
    local isDragging = false
    local dragStart = nil
    local startPos = nil
    
    titleBar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or
           input.UserInputType == Enum.UserInputType.Touch then
            isDragging = true
            dragStart = input.Position
            startPos = window.Position
        end
    end)
    
    UserInputService.InputChanged:Connect(function(input)
        if isDragging and (
            input.UserInputType == Enum.UserInputType.MouseMovement or
            input.UserInputType == Enum.UserInputType.Touch
        ) then
            local delta = input.Position - dragStart
            window.Position = UDim2.new(
                startPos.X.Scale,
                startPos.X.Offset + delta.X,
                startPos.Y.Scale,
                startPos.Y.Offset + delta.Y
            )
        end
    end)
    
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or
           input.UserInputType == Enum.UserInputType.Touch then
            isDragging = false
        end
    end)
    
    return window, content
end

-- ตัวอย่างการใช้
local window, content = createDraggableWindow({
    title = "🎮 Player Stats",
    size = UDim2.new(0, 300, 0, 250),
    position = UDim2.new(0.1, 0, 0.2, 0)
})

-- เพิ่มเนื้อหาใน Window
local statsLabel = Instance.new("TextLabel")
statsLabel.Size = UDim2.new(1, 0, 1, 0)
statsLabel.BackgroundTransparency = 1
statsLabel.Text = "Level: 10\nHP: 200/200\nATK: 50\nDEF: 30"
statsLabel.TextColor3 = Color3.new(1, 1, 1)
statsLabel.Font = Enum.Font.Gotham
statsLabel.TextSize = 16
statsLabel.Parent = content
```

---

## 30.13 แบบฝึกหัด

### แบบฝึกหัดที่ 1: สร้าง Settings Panel

```lua
-- LocalScript: Settings Panel
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")

local LocalPlayer = Players.LocalPlayer

local screenGui = Instance.new("ScreenGui")
screenGui.Parent = LocalPlayer.PlayerGui

-- Settings Panel
local panel = Instance.new("Frame")
panel.Size = UDim2.new(0, 400, 0, 500)
panel.AnchorPoint = Vector2.new(0.5, 0.5)
panel.Position = UDim2.new(0.5, 0, 0.5, 0)
panel.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
panel.BorderSizePixel = 0
panel.Visible = false  -- ซ่อนไว้ก่อน
panel.Parent = screenGui

local panelCorner = Instance.new("UICorner")
panelCorner.CornerRadius = UDim.new(0, 12)
panelCorner.Parent = panel

-- Title
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 50)
title.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
title.BorderSizePixel = 0
title.Text = "⚙️ ตั้งค่า"
title.TextColor3 = Color3.new(1, 1, 1)
title.Font = Enum.Font.GothamBold
title.TextSize = 20
title.Parent = panel

-- Settings Items
local settingsContainer = Instance.new("ScrollingFrame")
settingsContainer.Size = UDim2.new(1, -20, 1, -70)
settingsContainer.Position = UDim2.new(0, 10, 0, 60)
settingsContainer.BackgroundTransparency = 1
settingsContainer.ScrollBarThickness = 4
settingsContainer.AutomaticCanvasSize = Enum.AutomaticSize.Y
settingsContainer.CanvasSize = UDim2.new(0, 0, 0, 0)
settingsContainer.Parent = panel

local settingsList = Instance.new("UIListLayout")
settingsList.Padding = UDim.new(0, 8)
settingsList.Parent = settingsContainer

-- ฟังก์ชันสร้าง Toggle Setting
local function createToggle(name, defaultValue, onChange)
    local row = Instance.new("Frame")
    row.Size = UDim2.new(1, 0, 0, 50)
    row.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
    row.BorderSizePixel = 0
    row.Parent = settingsContainer
    
    local rowCorner = Instance.new("UICorner")
    rowCorner.CornerRadius = UDim.new(0, 8)
    rowCorner.Parent = row
    
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, -80, 1, 0)
    label.Position = UDim2.new(0, 15, 0, 0)
    label.BackgroundTransparency = 1
    label.Text = name
    label.TextColor3 = Color3.new(1, 1, 1)
    label.Font = Enum.Font.Gotham
    label.TextSize = 15
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = row
    
    -- Toggle Button
    local toggleBg = Instance.new("Frame")
    toggleBg.Size = UDim2.new(0, 50, 0, 26)
    toggleBg.Position = UDim2.new(1, -65, 0.5, -13)
    toggleBg.BackgroundColor3 = defaultValue and 
        Color3.fromRGB(60, 160, 80) or Color3.fromRGB(70, 70, 90)
    toggleBg.BorderSizePixel = 0
    toggleBg.Parent = row
    
    local toggleCorner = Instance.new("UICorner")
    toggleCorner.CornerRadius = UDim.new(0, 13)
    toggleCorner.Parent = toggleBg
    
    local toggleDot = Instance.new("Frame")
    toggleDot.Size = UDim2.new(0, 20, 0, 20)
    toggleDot.Position = defaultValue and 
        UDim2.new(1, -23, 0.5, -10) or UDim2.new(0, 3, 0.5, -10)
    toggleDot.BackgroundColor3 = Color3.new(1, 1, 1)
    toggleDot.BorderSizePixel = 0
    toggleDot.Parent = toggleBg
    
    local dotCorner = Instance.new("UICorner")
    dotCorner.CornerRadius = UDim.new(0, 10)
    dotCorner.Parent = toggleDot
    
    local value = defaultValue
    
    toggleBg.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            value = not value
            
            TweenService:Create(toggleBg, TweenInfo.new(0.2), {
                BackgroundColor3 = value and 
                    Color3.fromRGB(60, 160, 80) or Color3.fromRGB(70, 70, 90)
            }):Play()
            
            TweenService:Create(toggleDot, TweenInfo.new(0.2), {
                Position = value and 
                    UDim2.new(1, -23, 0.5, -10) or UDim2.new(0, 3, 0.5, -10)
            }):Play()
            
            if onChange then onChange(value) end
        end
    end)
    
    return row
end

-- สร้าง Settings
createToggle("เสียงดนตรี", true, function(v) print("Music:", v) end)
createToggle("เสียงเอฟเฟกต์", true, function(v) print("SFX:", v) end)
createToggle("แสดง FPS", false, function(v) print("FPS:", v) end)
createToggle("คุณภาพกราฟิก", true, function(v) print("Graphics:", v) end)
createToggle("แสดงชื่อผู้เล่น", true, function(v) print("Names:", v) end)

-- Toggle Panel Button
local toggleBtn = Instance.new("TextButton")
toggleBtn.Size = UDim2.new(0, 120, 0, 40)
toggleBtn.Position = UDim2.new(0, 10, 0, 10)
toggleBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
toggleBtn.Text = "⚙️ ตั้งค่า"
toggleBtn.TextColor3 = Color3.new(1, 1, 1)
toggleBtn.Font = Enum.Font.GothamBold
toggleBtn.TextSize = 14
toggleBtn.BorderSizePixel = 0
toggleBtn.Parent = screenGui

local btnCorner = Instance.new("UICorner")
btnCorner.CornerRadius = UDim.new(0, 8)
btnCorner.Parent = toggleBtn

toggleBtn.MouseButton1Click:Connect(function()
    panel.Visible = not panel.Visible
    if panel.Visible then
        panel.Size = UDim2.new(0, 0, 0, 0)
        TweenService:Create(
            panel,
            TweenInfo.new(0.3, Enum.EasingStyle.Back, Enum.EasingDirection.Out),
            {Size = UDim2.new(0, 400, 0, 500)}
        ):Play()
    end
end)
```

---

## 30.14 สรุป

ในบทนี้เราได้เรียนรู้:

1. **ScreenGui Properties** - Enabled, ResetOnSpawn, DisplayOrder
2. **Frame Properties** - ทุก Property ของ Frame
3. **UICorner** - มุมโค้งมน
4. **UIStroke** - ขอบ Element
5. **UIGradient** - ไล่สี
6. **UIScale** - ปรับขนาด
7. **UISizeConstraint** - จำกัดขนาด
8. **UIAspectRatioConstraint** - คงอัตราส่วน
9. **Modal Dialog** - กล่องยืนยัน
10. **Notification System** - ระบบแจ้งเตือน
11. **Tab System** - ระบบ Tab
12. **Draggable Window** - หน้าต่างที่ลากได้
13. **Settings Panel** - หน้า Settings

ในบทต่อไป เราจะเรียนรู้เกี่ยวกับ TextLabel และ TextButton

---

## แหล่งอ้างอิง

- [Roblox Developer Hub - Frame](https://developer.roblox.com/en-us/api-reference/class/Frame)
- [Roblox Developer Hub - ScreenGui](https://developer.roblox.com/en-us/api-reference/class/ScreenGui)
- [Roblox Developer Hub - UICorner](https://developer.roblox.com/en-us/api-reference/class/UICorner)
