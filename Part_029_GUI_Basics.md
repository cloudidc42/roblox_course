# ตอนที่ 29: GUI Basics - พื้นฐาน GUI ใน Roblox

## บทนำ

GUI (Graphical User Interface) คือส่วนประกอบที่แสดงบนหน้าจอผู้เล่น เช่น เมนู, ปุ่ม, แถบชีวิต, คะแนน และอื่นๆ ในบทนี้เราจะเรียนรู้พื้นฐาน GUI ใน Roblox ตั้งแต่การสร้าง GUI ไปจนถึงการใช้งานขั้นสูง

---

## 29.1 ประเภทของ GUI ใน Roblox

### 29.1.1 GUI Types

```
ScreenGui      - GUI ที่แสดงบนหน้าจอ (2D overlay)
BillboardGui   - GUI ที่ติดกับ Part ในโลก 3D
SurfaceGui     - GUI ที่แสดงบนผิวของ Part
```

### 29.1.2 GUI Elements

```
Frame          - กรอบสี่เหลี่ยม (Container)
TextLabel      - ข้อความ
TextButton     - ปุ่มที่กดได้
TextBox        - ช่องกรอกข้อมูล
ImageLabel     - รูปภาพ
ImageButton    - ปุ่มที่เป็นรูปภาพ
ScrollingFrame - Frame ที่ Scroll ได้
ViewportFrame  - แสดง 3D Object ใน GUI
```

---

## 29.2 ScreenGui

```lua
-- LocalScript
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

-- สร้าง ScreenGui
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "MyGui"
screenGui.ResetOnSpawn = false  -- ไม่รีเซ็ตเมื่อ Spawn ใหม่
screenGui.IgnoreGuiInset = false  -- ว่าจะ ignore top bar หรือเปล่า
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling  -- Z-Index behavior
screenGui.Parent = LocalPlayer.PlayerGui

print("สร้าง ScreenGui สำเร็จ!")
```

---

## 29.3 UDim2 - ระบบขนาดและตำแหน่ง

UDim2 เป็นระบบที่ Roblox ใช้กำหนดขนาดและตำแหน่ง GUI มี 2 แบบ:
- **Scale**: สัดส่วนตาม Parent (0.0 - 1.0)
- **Offset**: จำนวน Pixel

```lua
-- UDim2.new(scaleX, offsetX, scaleY, offsetY)

-- ขนาด 50% ของ Parent
local halfSize = UDim2.new(0.5, 0, 0.5, 0)

-- ขนาด 200x100 Pixels
local pixelSize = UDim2.new(0, 200, 0, 100)

-- ผสม Scale และ Offset
local mixedSize = UDim2.new(0.5, -50, 0, 100)  -- 50% ลบ 50px กว้าง, 100px สูง

-- ตำแหน่งกึ่งกลาง
local centerPos = UDim2.new(0.5, -100, 0.5, -50)
-- X: 50% ของ Parent ลบ 100px (ครึ่งความกว้างของตัวเอง)
-- Y: 50% ของ Parent ลบ 50px (ครึ่งความสูงของตัวเอง)

-- ตัวอย่างการใช้
local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 200, 0, 100)      -- 200x100 pixels
frame.Position = UDim2.new(0.5, -100, 0.5, -50)  -- กึ่งกลาง
```

---

## 29.4 AnchorPoint

AnchorPoint กำหนด "จุดอ้างอิง" ของ Element:

```lua
local frame = Instance.new("Frame")

-- AnchorPoint.new(X, Y) - ค่าระหว่าง 0-1
-- (0,0) = มุมบนซ้าย (default)
-- (0.5,0.5) = ตรงกลาง
-- (1,1) = มุมล่างขวา

frame.AnchorPoint = Vector2.new(0.5, 0.5)  -- กึ่งกลาง
frame.Position = UDim2.new(0.5, 0, 0.5, 0) -- ตรงกลางหน้าจอ

-- ตัวอย่างต่างๆ
local topLeft = Instance.new("Frame")
topLeft.AnchorPoint = Vector2.new(0, 0)
topLeft.Position = UDim2.new(0, 0, 0, 0)

local topCenter = Instance.new("Frame")
topCenter.AnchorPoint = Vector2.new(0.5, 0)
topCenter.Position = UDim2.new(0.5, 0, 0, 0)

local topRight = Instance.new("Frame")
topRight.AnchorPoint = Vector2.new(1, 0)
topRight.Position = UDim2.new(1, 0, 0, 0)

local bottomRight = Instance.new("Frame")
bottomRight.AnchorPoint = Vector2.new(1, 1)
bottomRight.Position = UDim2.new(1, 0, 1, 0)
```

---

## 29.5 ZIndex

ZIndex กำหนดลำดับการแสดงผล (ค่าสูงกว่าอยู่ด้านหน้า):

```lua
local background = Instance.new("Frame")
background.ZIndex = 1  -- อยู่ด้านหลัง

local middleFrame = Instance.new("Frame")
middleFrame.ZIndex = 2  -- อยู่กลาง

local frontButton = Instance.new("TextButton")
frontButton.ZIndex = 3  -- อยู่ด้านหน้า
```

---

## 29.6 Frame

Frame คือ Container พื้นฐานสำหรับ GUI Elements อื่นๆ

```lua
-- LocalScript
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

local screenGui = Instance.new("ScreenGui")
screenGui.Parent = LocalPlayer.PlayerGui

-- สร้าง Frame พื้นฐาน
local frame = Instance.new("Frame")
frame.Name = "MainFrame"
frame.Size = UDim2.new(0, 300, 0, 200)
frame.Position = UDim2.new(0.5, -150, 0.5, -100)
frame.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
frame.BorderSizePixel = 2
frame.BorderColor3 = Color3.fromRGB(100, 100, 100)
frame.BackgroundTransparency = 0.2
frame.Parent = screenGui

-- เพิ่ม UICorner (มุมโค้งมน)
local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 10)  -- โค้ง 10px
corner.Parent = frame

-- เพิ่ม UIStroke (ขอบ)
local stroke = Instance.new("UIStroke")
stroke.Color = Color3.fromRGB(255, 255, 255)
stroke.Thickness = 2
stroke.Transparency = 0.5
stroke.Parent = frame

-- เพิ่ม UIShadow (เงา) - ใน new Roblox
local shadow = Instance.new("UIGradient")
shadow.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(0, 0, 0)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(0, 0, 0))
})
shadow.Transparency = NumberSequence.new({
    NumberSequenceKeypoint.new(0, 0.8),
    NumberSequenceKeypoint.new(1, 1)
})
shadow.Rotation = 90
shadow.Parent = frame
```

---

## 29.7 UILayout Components

### 29.7.1 UIListLayout

จัดเรียง Elements แบบ List:

```lua
local screenGui = Instance.new("ScreenGui")
screenGui.Parent = game.Players.LocalPlayer.PlayerGui

local container = Instance.new("Frame")
container.Size = UDim2.new(0, 200, 0, 300)
container.Position = UDim2.new(0.1, 0, 0.1, 0)
container.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
container.Parent = screenGui

-- UIListLayout
local listLayout = Instance.new("UIListLayout")
listLayout.FillDirection = Enum.FillDirection.Vertical  -- จัดแนวตั้ง
listLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
listLayout.VerticalAlignment = Enum.VerticalAlignment.Top
listLayout.Padding = UDim.new(0, 5)  -- ระยะห่างระหว่าง Elements
listLayout.SortOrder = Enum.SortOrder.LayoutOrder
listLayout.Parent = container

-- เพิ่ม Elements ใน List
for i = 1, 5 do
    local item = Instance.new("TextButton")
    item.Size = UDim2.new(1, -10, 0, 40)
    item.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
    item.Text = "Item " .. i
    item.TextColor3 = Color3.new(1, 1, 1)
    item.LayoutOrder = i
    item.Parent = container
end

-- อัพเดตขนาด Container ตาม Content
listLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
    container.Size = UDim2.new(0, 200, 0, listLayout.AbsoluteContentSize.Y + 10)
end)
```

### 29.7.2 UIGridLayout

จัดเรียงแบบ Grid:

```lua
local container = Instance.new("Frame")
container.Size = UDim2.new(0, 400, 0, 300)
container.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
container.Parent = screenGui

local gridLayout = Instance.new("UIGridLayout")
gridLayout.CellSize = UDim2.new(0, 80, 0, 80)
gridLayout.CellPadding = UDim2.new(0, 5, 0, 5)
gridLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
gridLayout.VerticalAlignment = Enum.VerticalAlignment.Center
gridLayout.FillDirection = Enum.FillDirection.Horizontal
gridLayout.Parent = container

-- เพิ่ม Items
for i = 1, 12 do
    local cell = Instance.new("Frame")
    cell.BackgroundColor3 = Color3.fromHSV(i/12, 0.8, 0.9)
    cell.Parent = container
    
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, 0, 1, 0)
    label.BackgroundTransparency = 1
    label.Text = tostring(i)
    label.TextColor3 = Color3.new(1, 1, 1)
    label.Parent = cell
end
```

### 29.7.3 UITableLayout

```lua
local gridLayout = Instance.new("UITableLayout")
gridLayout.FillDirection = Enum.FillDirection.Vertical
gridLayout.Padding = UDim2.new(0, 2, 0, 2)
gridLayout.FillEmptySpaceColumns = true
gridLayout.FillEmptySpaceRows = false
gridLayout.Parent = container
```

### 29.7.4 UIPadding

เพิ่มระยะห่างภายใน Frame:

```lua
local padding = Instance.new("UIPadding")
padding.PaddingTop = UDim.new(0, 10)
padding.PaddingBottom = UDim.new(0, 10)
padding.PaddingLeft = UDim.new(0, 10)
padding.PaddingRight = UDim.new(0, 10)
padding.Parent = frame
```

---

## 29.8 ScrollingFrame

```lua
-- LocalScript
local screenGui = Instance.new("ScreenGui")
screenGui.Parent = game.Players.LocalPlayer.PlayerGui

-- ScrollingFrame
local scrollFrame = Instance.new("ScrollingFrame")
scrollFrame.Size = UDim2.new(0, 300, 0, 400)
scrollFrame.Position = UDim2.new(0.5, -150, 0.5, -200)
scrollFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
scrollFrame.ScrollBarThickness = 8
scrollFrame.ScrollBarImageColor3 = Color3.fromRGB(100, 100, 100)
scrollFrame.CanvasSize = UDim2.new(0, 0, 0, 0)  -- จะคำนวณอัตโนมัติ
scrollFrame.AutomaticCanvasSize = Enum.AutomaticSize.Y  -- Auto resize Y
scrollFrame.Parent = screenGui

-- UIListLayout ภายใน
local listLayout = Instance.new("UIListLayout")
listLayout.Padding = UDim.new(0, 5)
listLayout.Parent = scrollFrame

-- เพิ่มเนื้อหา
for i = 1, 20 do
    local item = Instance.new("TextLabel")
    item.Size = UDim2.new(1, -16, 0, 50)
    item.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    item.Text = "รายการที่ " .. i
    item.TextColor3 = Color3.new(1, 1, 1)
    item.Font = Enum.Font.Gotham
    item.TextSize = 16
    item.Parent = scrollFrame
end
```

---

## 29.9 BillboardGui

GUI ที่ติดกับ Part ในโลก 3D:

```lua
-- Script (Server)
local part = workspace.SomePart

local billboard = Instance.new("BillboardGui")
billboard.Size = UDim2.new(0, 200, 0, 50)
billboard.StudsOffset = Vector3.new(0, 5, 0)  -- เหนือ Part 5 studs
billboard.AlwaysOnTop = false  -- ถูกกีดขวางได้
billboard.MaxDistance = 100    -- มองเห็นได้ถึง 100 studs
billboard.Parent = part

-- เพิ่ม TextLabel
local label = Instance.new("TextLabel")
label.Size = UDim2.new(1, 0, 1, 0)
label.BackgroundTransparency = 1
label.Text = "ชื่อ Part"
label.TextColor3 = Color3.new(1, 1, 1)
label.Font = Enum.Font.GothamBold
label.TextSize = 20
label.TextStrokeTransparency = 0
label.TextStrokeColor3 = Color3.new(0, 0, 0)
label.Parent = billboard
```

---

## 29.10 SurfaceGui

GUI ที่แสดงบนผิวของ Part:

```lua
local part = workspace.Screen

-- สร้าง SurfaceGui
local surfaceGui = Instance.new("SurfaceGui")
surfaceGui.Face = Enum.NormalId.Front  -- ด้านหน้า
surfaceGui.CanvasSize = Vector2.new(500, 300)
surfaceGui.Adornee = part
surfaceGui.Parent = part

-- เพิ่มเนื้อหา
local background = Instance.new("Frame")
background.Size = UDim2.new(1, 0, 1, 0)
background.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
background.Parent = surfaceGui

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0.3, 0)
title.Position = UDim2.new(0, 0, 0.1, 0)
title.BackgroundTransparency = 1
title.Text = "📺 หน้าจอในเกม"
title.TextColor3 = Color3.new(1, 1, 0)
title.Font = Enum.Font.GothamBold
title.TextSize = 40
title.Parent = background
```

---

## 29.11 ViewportFrame

แสดง 3D Object ใน GUI:

```lua
-- LocalScript
local screenGui = Instance.new("ScreenGui")
screenGui.Parent = game.Players.LocalPlayer.PlayerGui

local viewport = Instance.new("ViewportFrame")
viewport.Size = UDim2.new(0, 200, 0, 200)
viewport.Position = UDim2.new(0, 10, 0.5, -100)
viewport.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
viewport.Parent = screenGui

-- สร้าง Camera สำหรับ Viewport
local viewCamera = Instance.new("Camera")
viewCamera.Parent = viewport
viewport.CurrentCamera = viewCamera

-- สร้าง Model ใน Viewport
local model = Instance.new("Model")
model.Parent = viewport

local part = Instance.new("Part")
part.Size = Vector3.new(3, 3, 3)
part.Position = Vector3.new(0, 0, 0)
part.BrickColor = BrickColor.new("Bright red")
part.Anchored = true
part.Parent = model

-- ตั้งค่า Camera
viewCamera.CFrame = CFrame.new(Vector3.new(8, 5, 8), Vector3.new(0, 0, 0))

-- หมุน Object
local RunService = game:GetService("RunService")
RunService.RenderStepped:Connect(function(dt)
    part.CFrame = part.CFrame * CFrame.Angles(0, dt, 0)
end)
```

---

## 29.12 ระบบ GUI Animation

```lua
-- LocalScript: GUI Animations
local TweenService = game:GetService("TweenService")
local Players = game:GetService("Players")

local screenGui = Instance.new("ScreenGui")
screenGui.Parent = Players.LocalPlayer.PlayerGui

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 200, 0, 100)
frame.Position = UDim2.new(0.5, -100, 1, 100)  -- อยู่ใต้หน้าจอ
frame.BackgroundColor3 = Color3.fromRGB(50, 150, 255)
frame.AnchorPoint = Vector2.new(0.5, 0.5)
frame.Parent = screenGui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 10)
corner.Parent = frame

-- Slide In from Bottom
local function slideIn()
    local tween = TweenService:Create(
        frame,
        TweenInfo.new(0.5, Enum.EasingStyle.Back, Enum.EasingDirection.Out),
        {Position = UDim2.new(0.5, 0, 0.5, 0)}
    )
    tween:Play()
    return tween
end

-- Slide Out to Bottom
local function slideOut()
    local tween = TweenService:Create(
        frame,
        TweenInfo.new(0.5, Enum.EasingStyle.Back, Enum.EasingDirection.In),
        {Position = UDim2.new(0.5, 0, 1, 100)}
    )
    tween:Play()
    return tween
end

-- Scale In
local function scaleIn()
    frame.Size = UDim2.new(0, 0, 0, 0)
    local tween = TweenService:Create(
        frame,
        TweenInfo.new(0.3, Enum.EasingStyle.Back, Enum.EasingDirection.Out),
        {Size = UDim2.new(0, 200, 0, 100)}
    )
    tween:Play()
end

-- Fade In
local function fadeIn()
    frame.BackgroundTransparency = 1
    local tween = TweenService:Create(
        frame,
        TweenInfo.new(0.5, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
        {BackgroundTransparency = 0}
    )
    tween:Play()
end

-- เรียกใช้
wait(1)
slideIn()
wait(3)
slideOut()
```

---

## 29.13 Responsive GUI Design

```lua
-- LocalScript: Responsive GUI
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

local screenGui = Instance.new("ScreenGui")
screenGui.Parent = LocalPlayer.PlayerGui

-- ตรวจสอบขนาดหน้าจอ
local camera = workspace.CurrentCamera
local screenSize = camera.ViewportSize

print("ขนาดหน้าจอ:", screenSize.X, "x", screenSize.Y)

-- สร้าง GUI ที่ปรับตามหน้าจอ
local function createResponsiveFrame()
    local frame = Instance.new("Frame")
    frame.AnchorPoint = Vector2.new(0.5, 0.5)
    frame.Position = UDim2.new(0.5, 0, 0.5, 0)
    frame.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    
    if screenSize.X < 768 then
        -- Mobile: เต็มหน้าจอ
        frame.Size = UDim2.new(0.9, 0, 0.8, 0)
    elseif screenSize.X < 1200 then
        -- Tablet: ขนาดกลาง
        frame.Size = UDim2.new(0.6, 0, 0.6, 0)
    else
        -- Desktop: ขนาดใหญ่
        frame.Size = UDim2.new(0.4, 0, 0.5, 0)
    end
    
    frame.Parent = screenGui
    return frame
end

local mainFrame = createResponsiveFrame()

-- ตรวจสอบการเปลี่ยนขนาดหน้าจอ
camera:GetPropertyChangedSignal("ViewportSize"):Connect(function()
    screenSize = camera.ViewportSize
    mainFrame:Destroy()
    mainFrame = createResponsiveFrame()
end)
```

---

## 29.14 GUI Themes

```lua
-- ModuleScript: GUITheme
local GUITheme = {}

-- Dark Theme
GUITheme.Dark = {
    Background = Color3.fromRGB(30, 30, 30),
    Surface = Color3.fromRGB(50, 50, 50),
    Primary = Color3.fromRGB(100, 149, 237),
    Secondary = Color3.fromRGB(80, 80, 80),
    Text = Color3.fromRGB(255, 255, 255),
    TextSecondary = Color3.fromRGB(180, 180, 180),
    Success = Color3.fromRGB(0, 200, 100),
    Warning = Color3.fromRGB(255, 200, 0),
    Danger = Color3.fromRGB(255, 80, 80),
    CornerRadius = UDim.new(0, 8),
    Padding = UDim.new(0, 10),
}

-- Light Theme
GUITheme.Light = {
    Background = Color3.fromRGB(240, 240, 240),
    Surface = Color3.fromRGB(255, 255, 255),
    Primary = Color3.fromRGB(70, 130, 200),
    Secondary = Color3.fromRGB(200, 200, 200),
    Text = Color3.fromRGB(20, 20, 20),
    TextSecondary = Color3.fromRGB(100, 100, 100),
    Success = Color3.fromRGB(0, 160, 80),
    Warning = Color3.fromRGB(200, 150, 0),
    Danger = Color3.fromRGB(200, 50, 50),
    CornerRadius = UDim.new(0, 8),
    Padding = UDim.new(0, 10),
}

local currentTheme = GUITheme.Dark

-- ฟังก์ชันสร้าง Styled Frame
function GUITheme.createFrame(parent, theme)
    theme = theme or currentTheme
    
    local frame = Instance.new("Frame")
    frame.BackgroundColor3 = theme.Surface
    frame.BorderSizePixel = 0
    frame.Parent = parent
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = theme.CornerRadius
    corner.Parent = frame
    
    return frame
end

-- ฟังก์ชันสร้าง Styled Button
function GUITheme.createButton(parent, text, theme)
    theme = theme or currentTheme
    
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(1, 0, 0, 40)
    button.BackgroundColor3 = theme.Primary
    button.Text = text
    button.TextColor3 = Color3.new(1, 1, 1)
    button.Font = Enum.Font.GothamBold
    button.TextSize = 14
    button.BorderSizePixel = 0
    button.Parent = parent
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = theme.CornerRadius
    corner.Parent = button
    
    -- Hover Effect
    button.MouseEnter:Connect(function()
        button.BackgroundColor3 = Color3.new(
            theme.Primary.R * 1.2,
            theme.Primary.G * 1.2,
            theme.Primary.B * 1.2
        )
    end)
    
    button.MouseLeave:Connect(function()
        button.BackgroundColor3 = theme.Primary
    end)
    
    return button
end

return GUITheme
```

---

## 29.15 Loading Screen

```lua
-- LocalScript: LoadingScreen
local Players = game:GetService("Players")
local ContentProvider = game:GetService("ContentProvider")
local TweenService = game:GetService("TweenService")

local LocalPlayer = Players.LocalPlayer

-- สร้าง Loading Screen
local screenGui = Instance.new("ScreenGui")
screenGui.IgnoreGuiInset = true
screenGui.DisplayOrder = 100
screenGui.Parent = LocalPlayer.PlayerGui

-- Background
local background = Instance.new("Frame")
background.Size = UDim2.new(1, 0, 1, 0)
background.BackgroundColor3 = Color3.fromRGB(10, 10, 20)
background.BorderSizePixel = 0
background.Parent = screenGui

-- Logo
local logo = Instance.new("TextLabel")
logo.Size = UDim2.new(0, 300, 0, 80)
logo.Position = UDim2.new(0.5, -150, 0.35, 0)
logo.BackgroundTransparency = 1
logo.Text = "🎮 MY AWESOME GAME"
logo.TextColor3 = Color3.new(1, 1, 1)
logo.Font = Enum.Font.GothamBlack
logo.TextSize = 36
logo.Parent = background

-- Loading Bar Background
local barBg = Instance.new("Frame")
barBg.Size = UDim2.new(0, 400, 0, 10)
barBg.Position = UDim2.new(0.5, -200, 0.7, 0)
barBg.BackgroundColor3 = Color3.fromRGB(50, 50, 70)
barBg.BorderSizePixel = 0
barBg.Parent = background

local barCorner = Instance.new("UICorner")
barCorner.CornerRadius = UDim.new(0, 5)
barCorner.Parent = barBg

-- Loading Bar
local bar = Instance.new("Frame")
bar.Size = UDim2.new(0, 0, 1, 0)
bar.BackgroundColor3 = Color3.fromRGB(100, 149, 237)
bar.BorderSizePixel = 0
bar.Parent = barBg

local barFillCorner = Instance.new("UICorner")
barFillCorner.CornerRadius = UDim.new(0, 5)
barFillCorner.Parent = bar

-- Loading Text
local loadText = Instance.new("TextLabel")
loadText.Size = UDim2.new(0, 400, 0, 30)
loadText.Position = UDim2.new(0.5, -200, 0.7, 20)
loadText.BackgroundTransparency = 1
loadText.Text = "กำลังโหลด..."
loadText.TextColor3 = Color3.fromRGB(180, 180, 180)
loadText.Font = Enum.Font.Gotham
loadText.TextSize = 14
loadText.Parent = background

-- โหลด Assets
local assetsToLoad = {
    "rbxassetid://123456",  -- ใส่ Asset IDs ของคุณ
    "rbxassetid://789012",
}

local loaded = 0
for i, asset in ipairs(assetsToLoad) do
    ContentProvider:PreloadAsync({asset})
    loaded = loaded + 1
    
    local progress = loaded / #assetsToLoad
    bar.Size = UDim2.new(progress, 0, 1, 0)
    loadText.Text = "กำลังโหลด... " .. math.floor(progress * 100) .. "%"
end

-- Fade Out
wait(0.5)
loadText.Text = "โหลดเสร็จแล้ว!"

local fadeTween = TweenService:Create(
    background,
    TweenInfo.new(1, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
    {BackgroundTransparency = 1}
)

fadeTween:Play()
fadeTween.Completed:Connect(function()
    screenGui:Destroy()
    print("Loading Screen ปิดแล้ว!")
end)
```

---

## 29.16 แบบฝึกหัด

### แบบฝึกหัดที่ 1: สร้าง Inventory UI

```lua
-- LocalScript: Simple Inventory
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

local screenGui = Instance.new("ScreenGui")
screenGui.Parent = LocalPlayer.PlayerGui

-- Inventory Data
local inventory = {
    {name = "ดาบ", icon = "⚔️", count = 1},
    {name = "โล่", icon = "🛡️", count = 1},
    {name = "ยา", icon = "🧪", count = 5},
    {name = "กุญแจ", icon = "🗝️", count = 2},
    {name = "เหรียญ", icon = "🪙", count = 100},
    {name = "ธนู", icon = "🏹", count = 30},
}

-- Main Panel
local panel = Instance.new("Frame")
panel.Size = UDim2.new(0, 350, 0, 400)
panel.Position = UDim2.new(0.5, -175, 0.5, -200)
panel.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
panel.BorderSizePixel = 0
panel.Parent = screenGui

local panelCorner = Instance.new("UICorner")
panelCorner.CornerRadius = UDim.new(0, 12)
panelCorner.Parent = panel

-- Title
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 50)
title.BackgroundColor3 = Color3.fromRGB(50, 50, 70)
title.BorderSizePixel = 0
title.Text = "🎒 กระเป๋า"
title.TextColor3 = Color3.new(1, 1, 1)
title.Font = Enum.Font.GothamBold
title.TextSize = 20
title.Parent = panel

local titleCorner = Instance.new("UICorner")
titleCorner.CornerRadius = UDim.new(0, 12)
titleCorner.Parent = title

-- Items Container
local itemsFrame = Instance.new("ScrollingFrame")
itemsFrame.Size = UDim2.new(1, -20, 1, -70)
itemsFrame.Position = UDim2.new(0, 10, 0, 60)
itemsFrame.BackgroundTransparency = 1
itemsFrame.ScrollBarThickness = 4
itemsFrame.AutomaticCanvasSize = Enum.AutomaticSize.Y
itemsFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
itemsFrame.Parent = panel

local itemList = Instance.new("UIListLayout")
itemList.Padding = UDim.new(0, 5)
itemList.Parent = itemsFrame

-- สร้าง Item Rows
for _, item in ipairs(inventory) do
    local row = Instance.new("Frame")
    row.Size = UDim2.new(1, 0, 0, 50)
    row.BackgroundColor3 = Color3.fromRGB(50, 50, 65)
    row.BorderSizePixel = 0
    row.Parent = itemsFrame
    
    local rowCorner = Instance.new("UICorner")
    rowCorner.CornerRadius = UDim.new(0, 8)
    rowCorner.Parent = row
    
    -- Icon
    local icon = Instance.new("TextLabel")
    icon.Size = UDim2.new(0, 40, 1, 0)
    icon.BackgroundTransparency = 1
    icon.Text = item.icon
    icon.TextSize = 24
    icon.Parent = row
    
    -- Name
    local nameLabel = Instance.new("TextLabel")
    nameLabel.Size = UDim2.new(1, -90, 1, 0)
    nameLabel.Position = UDim2.new(0, 45, 0, 0)
    nameLabel.BackgroundTransparency = 1
    nameLabel.Text = item.name
    nameLabel.TextColor3 = Color3.new(1, 1, 1)
    nameLabel.Font = Enum.Font.Gotham
    nameLabel.TextSize = 16
    nameLabel.TextXAlignment = Enum.TextXAlignment.Left
    nameLabel.Parent = row
    
    -- Count
    local countLabel = Instance.new("TextLabel")
    countLabel.Size = UDim2.new(0, 40, 1, 0)
    countLabel.Position = UDim2.new(1, -45, 0, 0)
    countLabel.BackgroundTransparency = 1
    countLabel.Text = "x" .. item.count
    countLabel.TextColor3 = Color3.fromRGB(200, 200, 100)
    countLabel.Font = Enum.Font.GothamBold
    countLabel.TextSize = 14
    countLabel.Parent = row
end

print("Inventory UI พร้อมแล้ว!")
```

---

## 29.17 สรุป

ในบทนี้เราได้เรียนรู้:

1. **GUI Types** - ScreenGui, BillboardGui, SurfaceGui
2. **UDim2** - ระบบ Scale และ Offset
3. **AnchorPoint** - จุดอ้างอิง
4. **ZIndex** - ลำดับการแสดงผล
5. **Frame** - Container พื้นฐาน
6. **UILayout** - UIListLayout, UIGridLayout, UIPadding
7. **ScrollingFrame** - Frame ที่เลื่อนได้
8. **BillboardGui** - GUI ใน 3D World
9. **SurfaceGui** - GUI บนผิว Part
10. **ViewportFrame** - แสดง 3D ใน GUI
11. **Animation** - Tween, Slide, Fade
12. **Themes** - ระบบ Theme สำหรับ GUI
13. **Loading Screen** - หน้าโหลดเกม

ในบทต่อไป เราจะเรียนรู้เกี่ยวกับ ScreenGui และ Frame อย่างละเอียด

---

## แหล่งอ้างอิง

- [Roblox Developer Hub - ScreenGui](https://developer.roblox.com/en-us/api-reference/class/ScreenGui)
- [Roblox Developer Hub - GUI Objects](https://developer.roblox.com/en-us/api-reference/class/GuiObject)
- [Roblox Developer Hub - UDim2](https://developer.roblox.com/en-us/api-reference/datatype/UDim2)
