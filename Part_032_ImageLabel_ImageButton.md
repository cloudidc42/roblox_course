# ตอนที่ 32: ImageLabel และ ImageButton

## บทนำ

ImageLabel และ ImageButton เป็น GUI Elements ที่ใช้แสดงรูปภาพใน Roblox เราสามารถใช้รูปภาพจาก Roblox Asset Library, Custom Decals, หรือสร้าง Sprite Sheets เพื่อทำ Animation ในบทนี้เราจะเรียนรู้วิธีการใช้งานอย่างละเอียด

---

## 32.1 ImageLabel Properties

```lua
-- LocalScript
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

local screenGui = Instance.new("ScreenGui")
screenGui.Parent = LocalPlayer.PlayerGui

local imageLabel = Instance.new("ImageLabel")

-- ขนาดและตำแหน่ง
imageLabel.Size = UDim2.new(0, 200, 0, 200)
imageLabel.Position = UDim2.new(0.5, -100, 0.5, -100)
imageLabel.AnchorPoint = Vector2.new(0.5, 0.5)

-- รูปภาพ
imageLabel.Image = "rbxassetid://123456789"  -- Asset ID

-- พื้นหลัง
imageLabel.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
imageLabel.BackgroundTransparency = 1  -- โปร่งใส (ส่วนใหญ่ใช้แบบนี้)

-- Scale Mode
imageLabel.ScaleType = Enum.ScaleType.Fit       -- ใส่ได้พอดีโดยไม่ตัด
-- Enum.ScaleType.Crop                           -- ตัดส่วนเกิน
-- Enum.ScaleType.Stretch                        -- ยืดเต็ม
-- Enum.ScaleType.Tile                           -- ทำซ้ำ
-- Enum.ScaleType.Slice                          -- ตาราง 9-slice

-- สำหรับ Tile Mode
imageLabel.TileSize = UDim2.new(0, 50, 0, 50)  -- ขนาด Tile

-- สำหรับ Slice Mode
imageLabel.SliceCenter = Rect.new(10, 10, 190, 190)  -- ขอบที่ไม่ขยาย
imageLabel.SliceScale = 1.0

-- ความโปร่งใสของรูป
imageLabel.ImageTransparency = 0  -- 0=ทึบ, 1=โปร่งใส

-- สีของรูป (ทับสีเดิม)
imageLabel.ImageColor3 = Color3.new(1, 1, 1)  -- สีปกติ
-- ตั้งค่าสีอื่นเพื่อ Tint รูป:
-- imageLabel.ImageColor3 = Color3.fromRGB(255, 100, 100)  -- แดง

-- Rect Mode (ตัดแค่ส่วนหนึ่งของรูป)
imageLabel.ImageRectOffset = Vector2.new(0, 0)   -- ตำแหน่งเริ่มต้น
imageLabel.ImageRectSize = Vector2.new(100, 100)  -- ขนาดที่ตัด

imageLabel.Parent = screenGui
```

---

## 32.2 ImageButton Properties

```lua
local imageButton = Instance.new("ImageButton")

-- ทุก Property ของ ImageLabel บวกเพิ่ม:
imageButton.Image = "rbxassetid://123456789"

-- รูป Hover
imageButton.HoverImage = "rbxassetid://987654321"  -- รูปเมื่อ Mouse อยู่บน

-- รูป Press
imageButton.PressedImage = "rbxassetid://111222333"  -- รูปเมื่อกด

-- เปิด/ปิด Auto Color
imageButton.AutoButtonColor = true

-- Events (เหมือน TextButton)
imageButton.MouseButton1Click:Connect(function()
    print("กด Image Button!")
end)

imageButton.MouseEnter:Connect(function()
    print("Hover!")
end)

imageButton.Parent = screenGui
```

---

## 32.3 การใช้ Asset IDs

```lua
-- รูปภาพจาก Roblox Asset Library
-- Format: rbxassetid://[ID]

-- ตัวอย่าง IDs บางส่วน (ต้องเป็น ID จริงๆ จาก Roblox)
local imageIds = {
    -- ใช้ ID จาก Roblox Toolbox หรือ Creator Store
    star = "rbxassetid://45428198",       -- ดาว
    heart = "rbxassetid://1234567890",    -- หัวใจ (ตัวอย่าง)
    coin = "rbxassetid://9876543210",     -- เหรียญ (ตัวอย่าง)
}

-- URL แบบอื่น
local gameIcon = "rbxgameiconthumbnail://[GameId]?width=256&height=256"
local playerAvatar = "rbxthumb://type=Avatar&id=[UserId]&w=150&h=150"
local playerAvatarHead = "rbxthumb://type=AvatarHeadShot&id=[UserId]&w=150&h=150"
local playerBust = "rbxthumb://type=AvatarBust&id=[UserId]&w=150&h=150"
local assetThumbnail = "rbxthumb://type=Asset&id=[AssetId]&w=150&h=150"

-- ตัวอย่างการแสดง Avatar ผู้เล่น
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

local avatarLabel = Instance.new("ImageLabel")
avatarLabel.Size = UDim2.new(0, 100, 0, 100)
avatarLabel.Position = UDim2.new(0, 10, 0, 10)
avatarLabel.BackgroundTransparency = 1
avatarLabel.Image = "rbxthumb://type=AvatarHeadShot&id=" .. LocalPlayer.UserId .. "&w=150&h=150"
avatarLabel.Parent = screenGui

-- Round Avatar
local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0.5, 0)
corner.Parent = avatarLabel
```

---

## 32.4 Sprite Sheet Animation

```lua
-- LocalScript: Sprite Sheet Animation
-- Sprite Sheet คือรูปที่รวม Frames หลายรูปไว้ในรูปเดียว

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local screenGui = Instance.new("ScreenGui")
screenGui.Parent = Players.LocalPlayer.PlayerGui

local spriteLabel = Instance.new("ImageLabel")
spriteLabel.Size = UDim2.new(0, 64, 0, 64)
spriteLabel.Position = UDim2.new(0.5, -32, 0.5, -32)
spriteLabel.BackgroundTransparency = 1

-- Sprite Sheet ที่มี 8 frames เรียงแนวนอน
-- แต่ละ frame ขนาด 64x64 pixels
spriteLabel.Image = "rbxassetid://YOUR_SPRITE_SHEET_ID"
spriteLabel.ImageRectSize = Vector2.new(64, 64)  -- ขนาดต่อ frame

spriteLabel.Parent = screenGui

-- ข้อมูล Sprite Sheet
local spriteConfig = {
    columns = 8,    -- คอลัมน์
    rows = 1,       -- แถว
    frameWidth = 64,
    frameHeight = 64,
    fps = 12,       -- Frames per second
}

local currentFrame = 0
local totalFrames = spriteConfig.columns * spriteConfig.rows
local timer = 0

RunService.Heartbeat:Connect(function(dt)
    timer = timer + dt
    
    if timer >= 1 / spriteConfig.fps then
        timer = 0
        currentFrame = (currentFrame + 1) % totalFrames
        
        -- คำนวณตำแหน่งใน Sprite Sheet
        local col = currentFrame % spriteConfig.columns
        local row = math.floor(currentFrame / spriteConfig.columns)
        
        spriteLabel.ImageRectOffset = Vector2.new(
            col * spriteConfig.frameWidth,
            row * spriteConfig.frameHeight
        )
    end
end)

-- ระบบ Sprite Animation แบบ OOP
local SpriteAnimation = {}
SpriteAnimation.__index = SpriteAnimation

function SpriteAnimation.new(imageLabel, config)
    local self = setmetatable({}, SpriteAnimation)
    self.label = imageLabel
    self.config = config
    self.currentFrame = 0
    self.timer = 0
    self.isPlaying = false
    self.currentAnimation = nil
    
    return self
end

function SpriteAnimation:addAnimation(name, frames, fps, loop)
    if not self.animations then self.animations = {} end
    self.animations[name] = {
        frames = frames,  -- {row, col} ของแต่ละ frame
        fps = fps,
        loop = loop ~= false,
        frameIndex = 1
    }
end

function SpriteAnimation:play(name)
    if not self.animations or not self.animations[name] then return end
    self.currentAnimation = self.animations[name]
    self.currentAnimation.frameIndex = 1
    self.isPlaying = true
    self.timer = 0
end

function SpriteAnimation:stop()
    self.isPlaying = false
end

function SpriteAnimation:update(dt)
    if not self.isPlaying or not self.currentAnimation then return end
    
    self.timer = self.timer + dt
    
    if self.timer >= 1 / self.currentAnimation.fps then
        self.timer = 0
        
        local anim = self.currentAnimation
        local frame = anim.frames[anim.frameIndex]
        
        -- ตั้งค่า Sprite
        self.label.ImageRectOffset = Vector2.new(
            frame[2] * self.config.frameWidth,
            frame[1] * self.config.frameHeight
        )
        
        anim.frameIndex = anim.frameIndex + 1
        
        if anim.frameIndex > #anim.frames then
            if anim.loop then
                anim.frameIndex = 1
            else
                self:stop()
            end
        end
    end
end
```

---

## 32.5 Progress Bar ด้วย Image

```lua
-- LocalScript: Image Progress Bar
local TweenService = game:GetService("TweenService")
local Players = game:GetService("Players")

local LocalPlayer = Players.LocalPlayer

local screenGui = Instance.new("ScreenGui")
screenGui.Parent = LocalPlayer.PlayerGui

-- HP Bar ที่ใช้ Image
local hpContainer = Instance.new("Frame")
hpContainer.Size = UDim2.new(0, 300, 0, 30)
hpContainer.Position = UDim2.new(0, 10, 1, -50)
hpContainer.BackgroundTransparency = 1
hpContainer.Parent = screenGui

-- Background Image
local hpBg = Instance.new("ImageLabel")
hpBg.Size = UDim2.new(1, 0, 1, 0)
hpBg.BackgroundTransparency = 1
hpBg.Image = "rbxasset://textures/ui/TopBar/HeartsOverlay.png"  -- ตัวอย่าง
hpBg.ScaleType = Enum.ScaleType.Slice
hpBg.SliceCenter = Rect.new(5, 5, 25, 25)
hpBg.ImageColor3 = Color3.fromRGB(50, 20, 20)
hpBg.Parent = hpContainer

-- HP Fill
local hpFill = Instance.new("ImageLabel")
hpFill.Size = UDim2.new(1, 0, 1, 0)
hpFill.BackgroundTransparency = 1
hpFill.Image = "rbxasset://textures/ui/TopBar/HeartsOverlay.png"
hpFill.ScaleType = Enum.ScaleType.Slice
hpFill.SliceCenter = Rect.new(5, 5, 25, 25)
hpFill.ImageColor3 = Color3.fromRGB(200, 50, 50)
hpFill.ClipsDescendants = true
hpFill.Parent = hpContainer

-- HP Fill Inner (ขนาดที่ปรับตาม HP)
local hpFillInner = Instance.new("Frame")
hpFillInner.Size = UDim2.new(1, 0, 1, 0)
hpFillInner.BackgroundTransparency = 1
hpFillInner.Parent = hpFill

-- อัพเดต HP
local function setHP(current, max)
    local pct = current / max
    TweenService:Create(hpFillInner, TweenInfo.new(0.5), {
        Size = UDim2.new(pct, 0, 1, 0)
    }):Play()
end

-- HP Text
local hpText = Instance.new("TextLabel")
hpText.Size = UDim2.new(1, 0, 1, 0)
hpText.BackgroundTransparency = 1
hpText.Text = "200 / 200"
hpText.TextColor3 = Color3.new(1, 1, 1)
hpText.Font = Enum.Font.GothamBold
hpText.TextSize = 14
hpText.TextStrokeTransparency = 0
hpText.Parent = hpContainer

setHP(150, 200)  -- ตัวอย่าง
```

---

## 32.6 Icon System

```lua
-- LocalScript: Icon System
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

local screenGui = Instance.new("ScreenGui")
screenGui.Parent = LocalPlayer.PlayerGui

-- Icon Data (ใช้ Emoji หรือ Custom Image)
local Icons = {
    -- ใช้ Asset IDs จริง
    sword = {id = "rbxassetid://YOUR_SWORD_ICON"},
    shield = {id = "rbxassetid://YOUR_SHIELD_ICON"},
    potion = {id = "rbxassetid://YOUR_POTION_ICON"},
    coin = {id = "rbxassetid://YOUR_COIN_ICON"},
    star = {id = "rbxassetid://YOUR_STAR_ICON"},
    heart = {id = "rbxassetid://YOUR_HEART_ICON"},
}

-- ฟังก์ชันสร้าง Icon
local function createIcon(iconName, size, parent)
    local iconData = Icons[iconName]
    if not iconData then return nil end
    
    local container = Instance.new("Frame")
    container.Size = UDim2.new(0, size, 0, size)
    container.BackgroundTransparency = 1
    container.Parent = parent
    
    local image = Instance.new("ImageLabel")
    image.Size = UDim2.new(1, 0, 1, 0)
    image.BackgroundTransparency = 1
    image.Image = iconData.id
    image.ScaleType = Enum.ScaleType.Fit
    image.Parent = container
    
    return container, image
end

-- Inventory Grid ด้วย Icons
local invFrame = Instance.new("Frame")
invFrame.Size = UDim2.new(0, 300, 0, 200)
invFrame.Position = UDim2.new(0, 10, 0.5, -100)
invFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
invFrame.BorderSizePixel = 0
invFrame.Parent = screenGui

local invCorner = Instance.new("UICorner")
invCorner.CornerRadius = UDim.new(0, 10)
invCorner.Parent = invFrame

local invGrid = Instance.new("UIGridLayout")
invGrid.CellSize = UDim2.new(0, 55, 0, 55)
invGrid.CellPadding = UDim2.new(0, 5, 0, 5)
invGrid.Parent = invFrame

local invPadding = Instance.new("UIPadding")
invPadding.PaddingTop = UDim.new(0, 10)
invPadding.PaddingLeft = UDim.new(0, 10)
invPadding.Parent = invFrame

-- สร้าง Item Slots
local items = {"sword", "shield", "potion", "coin", "star", "heart"}

for i = 1, 12 do
    local slot = Instance.new("Frame")
    slot.BackgroundColor3 = Color3.fromRGB(50, 50, 65)
    slot.BorderSizePixel = 0
    slot.Parent = invFrame
    
    local slotCorner = Instance.new("UICorner")
    slotCorner.CornerRadius = UDim.new(0, 6)
    slotCorner.Parent = slot
    
    if items[i] then
        local _, iconImage = createIcon(items[i], 45, slot)
        if iconImage then
            iconImage.Size = UDim2.new(1, -10, 1, -10)
            iconImage.Position = UDim2.new(0, 5, 0, 5)
        end
    end
end
```

---

## 32.7 Image Effects

```lua
-- LocalScript: Image Effects
local TweenService = game:GetService("TweenService")
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local screenGui = Instance.new("ScreenGui")
screenGui.Parent = Players.LocalPlayer.PlayerGui

-- Image ที่จะใส่ Effect
local image = Instance.new("ImageLabel")
image.Size = UDim2.new(0, 150, 0, 150)
image.AnchorPoint = Vector2.new(0.5, 0.5)
image.Position = UDim2.new(0.5, 0, 0.5, 0)
image.BackgroundTransparency = 1
image.Image = "rbxassetid://YOUR_IMAGE_ID"
image.Parent = screenGui

-- 1. Pulse Effect
local function pulseEffect(imgLabel, minScale, maxScale, speed)
    minScale = minScale or 0.9
    maxScale = maxScale or 1.1
    speed = speed or 1.0
    
    local growing = true
    
    local RunService = game:GetService("RunService")
    RunService.Heartbeat:Connect(function(dt)
        -- ไม่จำเป็นต้องใช้ Heartbeat สำหรับ Pulse ง่ายๆ
    end)
    
    -- ใช้ Tween แทน
    local function pulse()
        TweenService:Create(imgLabel, TweenInfo.new(speed/2, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut), {
            Size = UDim2.new(0, 150 * maxScale, 0, 150 * maxScale)
        }):Play()
        wait(speed/2)
        TweenService:Create(imgLabel, TweenInfo.new(speed/2, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut), {
            Size = UDim2.new(0, 150 * minScale, 0, 150 * minScale)
        }):Play()
        wait(speed/2)
        pulse()
    end
    
    task.spawn(pulse)
end

-- 2. Rotation Effect
local function rotateEffect(imgLabel, speed)
    local rotation = 0
    RunService.Heartbeat:Connect(function(dt)
        rotation = rotation + speed * dt
        imgLabel.Rotation = rotation
    end)
end

-- 3. Color Cycle Effect
local function colorCycleEffect(imgLabel, speed)
    local hue = 0
    RunService.Heartbeat:Connect(function(dt)
        hue = (hue + speed * dt) % 1
        imgLabel.ImageColor3 = Color3.fromHSV(hue, 0.8, 1.0)
    end)
end

-- 4. Flicker Effect
local function flickerEffect(imgLabel, intensity, frequency)
    RunService.Heartbeat:Connect(function()
        if math.random() < frequency then
            imgLabel.ImageTransparency = math.random() * intensity
        else
            imgLabel.ImageTransparency = 0
        end
    end)
end

-- 5. Shine Effect
local function shineEffect(imgLabel)
    -- สร้าง Shine Overlay
    local shine = Instance.new("Frame")
    shine.Size = UDim2.new(0.3, 0, 1, 0)
    shine.Position = UDim2.new(-0.3, 0, 0, 0)
    shine.BackgroundTransparency = 1
    shine.ClipsDescendants = false
    shine.Parent = imgLabel
    
    local shineGrad = Instance.new("UIGradient")
    shineGrad.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.new(1, 1, 1)),
        ColorSequenceKeypoint.new(0.5, Color3.new(1, 1, 1)),
        ColorSequenceKeypoint.new(1, Color3.new(1, 1, 1))
    })
    shineGrad.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 1),
        NumberSequenceKeypoint.new(0.3, 0.5),
        NumberSequenceKeypoint.new(0.5, 0.3),
        NumberSequenceKeypoint.new(0.7, 0.5),
        NumberSequenceKeypoint.new(1, 1)
    })
    shineGrad.Rotation = 45
    shineGrad.Parent = shine
    
    local function playShine()
        while true do
            wait(3)
            TweenService:Create(shine, TweenInfo.new(0.5, Enum.EasingStyle.Linear), {
                Position = UDim2.new(1.3, 0, 0, 0)
            }):Play()
            wait(0.5)
            shine.Position = UDim2.new(-0.3, 0, 0, 0)
        end
    end
    
    task.spawn(playShine)
end

-- ใช้ Effect
-- pulseEffect(image)
-- rotateEffect(image, 90)  -- 90 degrees per second
-- colorCycleEffect(image, 0.2)
shineEffect(image)
```

---

## 32.8 Avatar Display Panel

```lua
-- LocalScript: Avatar Display
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")

local LocalPlayer = Players.LocalPlayer

local screenGui = Instance.new("ScreenGui")
screenGui.Parent = LocalPlayer.PlayerGui

-- Player Card
local card = Instance.new("Frame")
card.Size = UDim2.new(0, 220, 0, 300)
card.Position = UDim2.new(0, 10, 0.5, -150)
card.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
card.BorderSizePixel = 0
card.Parent = screenGui

local cardCorner = Instance.new("UICorner")
cardCorner.CornerRadius = UDim.new(0, 12)
cardCorner.Parent = card

local cardStroke = Instance.new("UIStroke")
cardStroke.Color = Color3.fromRGB(100, 100, 150)
cardStroke.Thickness = 1
cardStroke.Parent = card

-- Avatar Image
local avatarBg = Instance.new("Frame")
avatarBg.Size = UDim2.new(0, 120, 0, 120)
avatarBg.AnchorPoint = Vector2.new(0.5, 0)
avatarBg.Position = UDim2.new(0.5, 0, 0, 20)
avatarBg.BackgroundColor3 = Color3.fromRGB(50, 50, 70)
avatarBg.BorderSizePixel = 0
avatarBg.Parent = card

local avatarCorner = Instance.new("UICorner")
avatarCorner.CornerRadius = UDim.new(0.5, 0)
avatarCorner.Parent = avatarBg

local avatar = Instance.new("ImageLabel")
avatar.Size = UDim2.new(1, 0, 1, 0)
avatar.BackgroundTransparency = 1
avatar.Image = "rbxthumb://type=AvatarHeadShot&id=" .. LocalPlayer.UserId .. "&w=150&h=150"
avatar.ScaleType = Enum.ScaleType.Fit
avatar.Parent = avatarBg

local avatarImgCorner = Instance.new("UICorner")
avatarImgCorner.CornerRadius = UDim.new(0.5, 0)
avatarImgCorner.Parent = avatar

-- Online Indicator
local onlineDot = Instance.new("Frame")
onlineDot.Size = UDim2.new(0, 20, 0, 20)
onlineDot.Position = UDim2.new(1, -22, 1, -22)
onlineDot.BackgroundColor3 = Color3.fromRGB(0, 200, 80)
onlineDot.BorderSizePixel = 0
onlineDot.Parent = avatarBg

local dotCorner = Instance.new("UICorner")
dotCorner.CornerRadius = UDim.new(0.5, 0)
dotCorner.Parent = onlineDot

local dotStroke = Instance.new("UIStroke")
dotStroke.Color = Color3.fromRGB(25, 25, 35)
dotStroke.Thickness = 3
dotStroke.Parent = onlineDot

-- Player Name
local nameLabel = Instance.new("TextLabel")
nameLabel.Size = UDim2.new(1, -20, 0, 30)
nameLabel.Position = UDim2.new(0, 10, 0, 150)
nameLabel.BackgroundTransparency = 1
nameLabel.Text = LocalPlayer.DisplayName
nameLabel.TextColor3 = Color3.new(1, 1, 1)
nameLabel.Font = Enum.Font.GothamBold
nameLabel.TextSize = 18
nameLabel.Parent = card

-- Username
local usernameLabel = Instance.new("TextLabel")
usernameLabel.Size = UDim2.new(1, -20, 0, 20)
usernameLabel.Position = UDim2.new(0, 10, 0, 180)
usernameLabel.BackgroundTransparency = 1
usernameLabel.Text = "@" .. LocalPlayer.Name
usernameLabel.TextColor3 = Color3.fromRGB(150, 150, 180)
usernameLabel.Font = Enum.Font.Gotham
usernameLabel.TextSize = 13
usernameLabel.Parent = card

-- Stats
local statsData = {
    {"⚔️", "Level", "99"},
    {"❤️", "HP", "200"},
    {"⭐", "Score", "12,450"},
}

for i, stat in ipairs(statsData) do
    local row = Instance.new("Frame")
    row.Size = UDim2.new(1, -20, 0, 30)
    row.Position = UDim2.new(0, 10, 0, 205 + (i-1) * 30)
    row.BackgroundTransparency = 1
    row.Parent = card
    
    local icon = Instance.new("TextLabel")
    icon.Size = UDim2.new(0, 25, 1, 0)
    icon.BackgroundTransparency = 1
    icon.Text = stat[1]
    icon.TextSize = 16
    icon.Parent = row
    
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(0.5, -25, 1, 0)
    label.Position = UDim2.new(0, 25, 0, 0)
    label.BackgroundTransparency = 1
    label.Text = stat[2]
    label.TextColor3 = Color3.fromRGB(180, 180, 200)
    label.Font = Enum.Font.Gotham
    label.TextSize = 13
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = row
    
    local value = Instance.new("TextLabel")
    value.Size = UDim2.new(0.5, 0, 1, 0)
    value.Position = UDim2.new(0.5, 0, 0, 0)
    value.BackgroundTransparency = 1
    value.Text = stat[3]
    value.TextColor3 = Color3.new(1, 1, 1)
    value.Font = Enum.Font.GothamBold
    value.TextSize = 13
    value.TextXAlignment = Enum.TextXAlignment.Right
    value.Parent = row
end
```

---

## 32.9 Image Gallery

```lua
-- LocalScript: Image Gallery
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")

local LocalPlayer = Players.LocalPlayer

local screenGui = Instance.new("ScreenGui")
screenGui.Parent = LocalPlayer.PlayerGui

-- Gallery Images (ต้องใส่ Asset IDs จริง)
local galleryImages = {
    {id = "rbxassetid://001", title = "ภาพที่ 1"},
    {id = "rbxassetid://002", title = "ภาพที่ 2"},
    {id = "rbxassetid://003", title = "ภาพที่ 3"},
    {id = "rbxassetid://004", title = "ภาพที่ 4"},
    {id = "rbxassetid://005", title = "ภาพที่ 5"},
}

-- Gallery Frame
local gallery = Instance.new("Frame")
gallery.Size = UDim2.new(0, 600, 0, 400)
gallery.AnchorPoint = Vector2.new(0.5, 0.5)
gallery.Position = UDim2.new(0.5, 0, 0.5, 0)
gallery.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
gallery.BorderSizePixel = 0
gallery.Parent = screenGui

local galleryCorner = Instance.new("UICorner")
galleryCorner.CornerRadius = UDim.new(0, 12)
galleryCorner.Parent = gallery

-- Main Image Display
local mainImage = Instance.new("ImageLabel")
mainImage.Size = UDim2.new(1, -20, 1, -120)
mainImage.Position = UDim2.new(0, 10, 0, 10)
mainImage.BackgroundColor3 = Color3.fromRGB(10, 10, 20)
mainImage.ScaleType = Enum.ScaleType.Fit
mainImage.Parent = gallery

local mainImageCorner = Instance.new("UICorner")
mainImageCorner.CornerRadius = UDim.new(0, 8)
mainImageCorner.Parent = mainImage

-- Thumbnail Row
local thumbRow = Instance.new("ScrollingFrame")
thumbRow.Size = UDim2.new(1, -20, 0, 90)
thumbRow.Position = UDim2.new(0, 10, 1, -100)
thumbRow.BackgroundTransparency = 1
thumbRow.ScrollBarThickness = 0
thumbRow.ScrollingDirection = Enum.ScrollingDirection.X
thumbRow.AutomaticCanvasSize = Enum.AutomaticSize.X
thumbRow.CanvasSize = UDim2.new(0, 0, 0, 0)
thumbRow.Parent = gallery

local thumbLayout = Instance.new("UIListLayout")
thumbLayout.FillDirection = Enum.FillDirection.Horizontal
thumbLayout.Padding = UDim.new(0, 5)
thumbLayout.Parent = thumbRow

local currentIndex = 1

local function selectImage(index)
    if index < 1 or index > #galleryImages then return end
    currentIndex = index
    
    -- Fade transition
    TweenService:Create(mainImage, TweenInfo.new(0.2), {ImageTransparency = 1}):Play()
    task.wait(0.2)
    mainImage.Image = galleryImages[index].id
    TweenService:Create(mainImage, TweenInfo.new(0.2), {ImageTransparency = 0}):Play()
end

-- สร้าง Thumbnails
for i, imgData in ipairs(galleryImages) do
    local thumb = Instance.new("ImageButton")
    thumb.Size = UDim2.new(0, 80, 0, 80)
    thumb.BackgroundColor3 = Color3.fromRGB(50, 50, 65)
    thumb.Image = imgData.id
    thumb.ScaleType = Enum.ScaleType.Crop
    thumb.BorderSizePixel = 0
    thumb.Parent = thumbRow
    
    local thumbCorner = Instance.new("UICorner")
    thumbCorner.CornerRadius = UDim.new(0, 6)
    thumbCorner.Parent = thumb
    
    thumb.MouseButton1Click:Connect(function()
        selectImage(i)
    end)
end

-- เริ่มที่รูปแรก
if #galleryImages > 0 then
    mainImage.Image = galleryImages[1].id
end
```

---

## 32.10 แบบฝึกหัด

### แบบฝึกหัดที่ 1: Item Card

```lua
-- สร้าง Item Card ที่สวยงาม
local function createItemCard(parent, item)
    local rarityColors = {
        common = Color3.fromRGB(180, 180, 180),
        uncommon = Color3.fromRGB(50, 200, 50),
        rare = Color3.fromRGB(50, 100, 255),
        epic = Color3.fromRGB(160, 50, 220),
        legendary = Color3.fromRGB(255, 165, 0),
    }
    
    local card = Instance.new("Frame")
    card.Size = UDim2.new(0, 160, 0, 220)
    card.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
    card.BorderSizePixel = 0
    card.Parent = parent
    
    local cardCorner = Instance.new("UICorner")
    cardCorner.CornerRadius = UDim.new(0, 10)
    cardCorner.Parent = card
    
    local rarityColor = rarityColors[item.rarity] or rarityColors.common
    
    -- Rarity Stroke
    local stroke = Instance.new("UIStroke")
    stroke.Color = rarityColor
    stroke.Thickness = 2
    stroke.Parent = card
    
    -- Item Image
    local imgBg = Instance.new("Frame")
    imgBg.Size = UDim2.new(1, -20, 0, 130)
    imgBg.Position = UDim2.new(0, 10, 0, 10)
    imgBg.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
    imgBg.BorderSizePixel = 0
    imgBg.Parent = card
    
    local imgBgCorner = Instance.new("UICorner")
    imgBgCorner.CornerRadius = UDim.new(0, 8)
    imgBgCorner.Parent = imgBg
    
    local itemImg = Instance.new("ImageLabel")
    itemImg.Size = UDim2.new(0.8, 0, 0.8, 0)
    itemImg.AnchorPoint = Vector2.new(0.5, 0.5)
    itemImg.Position = UDim2.new(0.5, 0, 0.5, 0)
    itemImg.BackgroundTransparency = 1
    itemImg.Image = item.imageId or ""
    itemImg.ScaleType = Enum.ScaleType.Fit
    itemImg.Parent = imgBg
    
    -- Rarity Badge
    local rarity = Instance.new("TextLabel")
    rarity.Size = UDim2.new(0, 90, 0, 20)
    rarity.AnchorPoint = Vector2.new(0.5, 0)
    rarity.Position = UDim2.new(0.5, 0, 1, -10)
    rarity.BackgroundColor3 = rarityColor
    rarity.BorderSizePixel = 0
    rarity.Text = string.upper(item.rarity or "COMMON")
    rarity.TextColor3 = Color3.new(1, 1, 1)
    rarity.Font = Enum.Font.GothamBold
    rarity.TextSize = 10
    rarity.Parent = imgBg
    
    local rarityCorner = Instance.new("UICorner")
    rarityCorner.CornerRadius = UDim.new(0, 5)
    rarityCorner.Parent = rarity
    
    -- Item Name
    local name = Instance.new("TextLabel")
    name.Size = UDim2.new(1, -20, 0, 25)
    name.Position = UDim2.new(0, 10, 0, 150)
    name.BackgroundTransparency = 1
    name.Text = item.name or "Unknown"
    name.TextColor3 = Color3.new(1, 1, 1)
    name.Font = Enum.Font.GothamBold
    name.TextSize = 14
    name.TextXAlignment = Enum.TextXAlignment.Left
    name.TextWrapped = true
    name.Parent = card
    
    -- Stats
    local stats = Instance.new("TextLabel")
    stats.Size = UDim2.new(1, -20, 0, 30)
    stats.Position = UDim2.new(0, 10, 0, 175)
    stats.BackgroundTransparency = 1
    stats.Text = item.stats or ""
    stats.TextColor3 = Color3.fromRGB(180, 180, 200)
    stats.Font = Enum.Font.Gotham
    stats.TextSize = 12
    stats.TextXAlignment = Enum.TextXAlignment.Left
    stats.TextWrapped = true
    stats.Parent = card
    
    return card
end

-- ตัวอย่าง
local Players = game:GetService("Players")
local screenGui = Instance.new("ScreenGui")
screenGui.Parent = Players.LocalPlayer.PlayerGui

local container = Instance.new("Frame")
container.Size = UDim2.new(0, 600, 0, 250)
container.AnchorPoint = Vector2.new(0.5, 0.5)
container.Position = UDim2.new(0.5, 0, 0.5, 0)
container.BackgroundTransparency = 1
container.Parent = screenGui

local layout = Instance.new("UIListLayout")
layout.FillDirection = Enum.FillDirection.Horizontal
layout.Padding = UDim.new(0, 10)
layout.Parent = container

local items = {
    {name = "Excalibur", rarity = "legendary", imageId = "rbxassetid://001", stats = "ATK: 999\nSPD: +20%"},
    {name = "Iron Sword", rarity = "common", imageId = "rbxassetid://002", stats = "ATK: 50\nSPD: +5%"},
    {name = "Mana Staff", rarity = "epic", imageId = "rbxassetid://003", stats = "MGC: 200\nMP: +50"},
}

for _, item in ipairs(items) do
    createItemCard(container, item)
end
```

---

## 32.11 สรุป

ในบทนี้เราได้เรียนรู้:

1. **ImageLabel Properties** - Image, ScaleType, ImageColor3, ImageRectOffset/Size
2. **ImageButton Properties** - HoverImage, PressedImage
3. **Asset IDs** - รูปจาก Roblox และ URL Format ต่างๆ
4. **Sprite Sheet Animation** - การทำ Animation จาก Sprite Sheet
5. **Progress Bar** - ใช้ Image สำหรับ HP/XP Bar
6. **Icon System** - จัดการ Icons ในเกม
7. **Image Effects** - Pulse, Rotation, Color Cycle, Flicker, Shine
8. **Avatar Display** - แสดงรูปโปรไฟล์ผู้เล่น
9. **Image Gallery** - Gallery ด้วย Thumbnails
10. **Item Card** - Card แสดงข้อมูล Item

ในบทต่อไป เราจะเรียนรู้เกี่ยวกับ GUI Events

---

## แหล่งอ้างอิง

- [Roblox Developer Hub - ImageLabel](https://developer.roblox.com/en-us/api-reference/class/ImageLabel)
- [Roblox Developer Hub - ImageButton](https://developer.roblox.com/en-us/api-reference/class/ImageButton)
- [Roblox Asset Library](https://www.roblox.com/develop)
