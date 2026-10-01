# Part 75: Settings Menu (เมนูตั้งค่า)

## บทนำ

Settings Menu ที่ดีช่วยให้ผู้เล่นปรับแต่งประสบการณ์ให้เหมาะกับตัวเอง ในบทนี้เราจะสร้าง Settings Menu ครอบคลุม ตั้งแต่การตั้งค่า Graphics, Audio ไปจนถึง Controls พร้อมระบบ Save/Load ด้วย DataStore

---

## 1. โครงสร้าง Settings System

### 1.1 Settings Module

```lua
-- SettingsModule.lua (ModuleScript ใน ReplicatedStorage)
-- ระบบจัดการ Settings ส่วนกลาง

local SettingsModule = {}
SettingsModule.__index = SettingsModule

-- ค่า Default Settings
local DEFAULT_SETTINGS = {
    -- Graphics
    graphics = {
        qualityLevel = 5,        -- 1-10
        shadows = true,
        particles = true,
        fov = 70,               -- Field of View (60-120)
        renderDistance = 512,   -- Render Distance
    },
    
    -- Audio
    audio = {
        masterVolume = 0.8,     -- 0-1
        musicVolume = 0.6,
        sfxVolume = 1.0,
        ambientVolume = 0.7,
        voiceEnabled = true,
    },
    
    -- Controls
    controls = {
        mouseSensitivity = 0.5, -- 0.1-2.0
        invertY = false,
        toggleSprint = false,
        autoJump = true,
    },
    
    -- UI
    ui = {
        showFPS = false,
        showPing = false,
        minimapEnabled = true,
        chatVisible = true,
        uiScale = 1.0,          -- 0.5-2.0
    },
    
    -- Notifications
    notifications = {
        showAchievements = true,
        showFriendJoined = true,
        showSystemMessages = true,
    },
    
    -- Accessibility
    accessibility = {
        colorBlindMode = "None",  -- None, Protanopia, Deuteranopia, Tritanopia
        textSize = "Normal",       -- Small, Normal, Large
        reduceMotion = false,
    }
}

-- Settings Validators (ตรวจสอบค่าที่ถูกต้อง)
local VALIDATORS = {
    ["graphics.qualityLevel"] = function(v) return math.clamp(math.floor(v), 1, 10) end,
    ["graphics.fov"] = function(v) return math.clamp(v, 60, 120) end,
    ["audio.masterVolume"] = function(v) return math.clamp(v, 0, 1) end,
    ["audio.musicVolume"] = function(v) return math.clamp(v, 0, 1) end,
    ["audio.sfxVolume"] = function(v) return math.clamp(v, 0, 1) end,
    ["controls.mouseSensitivity"] = function(v) return math.clamp(v, 0.1, 2.0) end,
    ["ui.uiScale"] = function(v) return math.clamp(v, 0.5, 2.0) end,
}

function SettingsModule.new()
    local self = setmetatable({}, SettingsModule)
    
    -- Deep copy default settings
    self.settings = {}
    for category, values in pairs(DEFAULT_SETTINGS) do
        self.settings[category] = {}
        for key, value in pairs(values) do
            self.settings[category][key] = value
        end
    end
    
    self.changeCallbacks = {}  -- Callbacks เมื่อ Setting เปลี่ยน
    
    return self
end

-- ดูค่า Setting
function SettingsModule:get(path)
    local parts = string.split(path, ".")
    if #parts == 1 then
        return self.settings[path]
    elseif #parts == 2 then
        local category = self.settings[parts[1]]
        if category then
            return category[parts[2]]
        end
    end
    return nil
end

-- ตั้งค่า Setting
function SettingsModule:set(path, value)
    -- Validate ค่า
    if VALIDATORS[path] then
        value = VALIDATORS[path](value)
    end
    
    local parts = string.split(path, ".")
    local oldValue
    
    if #parts == 1 then
        oldValue = self.settings[path]
        self.settings[path] = value
    elseif #parts == 2 then
        if not self.settings[parts[1]] then return end
        oldValue = self.settings[parts[1]][parts[2]]
        self.settings[parts[1]][parts[2]] = value
    end
    
    -- เรียก Callbacks
    if oldValue ~= value then
        self:_notifyChange(path, value, oldValue)
    end
end

-- ลงทะเบียน Callback เมื่อ Setting เปลี่ยน
function SettingsModule:onChange(path, callback)
    if not self.changeCallbacks[path] then
        self.changeCallbacks[path] = {}
    end
    table.insert(self.changeCallbacks[path], callback)
    
    -- Return unsubscribe function
    return function()
        local callbacks = self.changeCallbacks[path]
        for i, cb in ipairs(callbacks) do
            if cb == callback then
                table.remove(callbacks, i)
                break
            end
        end
    end
end

-- ส่ง Notification
function SettingsModule:_notifyChange(path, newValue, oldValue)
    -- Exact path
    if self.changeCallbacks[path] then
        for _, cb in ipairs(self.changeCallbacks[path]) do
            task.spawn(cb, newValue, oldValue)
        end
    end
    
    -- Category wildcard (e.g., "graphics.*")
    local parts = string.split(path, ".")
    if #parts == 2 then
        local categoryWild = parts[1] .. ".*"
        if self.changeCallbacks[categoryWild] then
            for _, cb in ipairs(self.changeCallbacks[categoryWild]) do
                task.spawn(cb, path, newValue, oldValue)
            end
        end
    end
    
    -- Global wildcard
    if self.changeCallbacks["*"] then
        for _, cb in ipairs(self.changeCallbacks["*"]) do
            task.spawn(cb, path, newValue, oldValue)
        end
    end
end

-- Reset to defaults
function SettingsModule:reset(category)
    if category then
        if DEFAULT_SETTINGS[category] then
            for key, value in pairs(DEFAULT_SETTINGS[category]) do
                self:set(category .. "." .. key, value)
            end
        end
    else
        -- Reset ทั้งหมด
        for cat, values in pairs(DEFAULT_SETTINGS) do
            for key, value in pairs(values) do
                self:set(cat .. "." .. key, value)
            end
        end
    end
end

-- Export/Import สำหรับ DataStore
function SettingsModule:export()
    return self.settings
end

function SettingsModule:import(data)
    if type(data) ~= "table" then return end
    
    for category, values in pairs(data) do
        if type(values) == "table" then
            for key, value in pairs(values) do
                local path = category .. "." .. key
                if DEFAULT_SETTINGS[category] and DEFAULT_SETTINGS[category][key] ~= nil then
                    self:set(path, value)
                end
            end
        end
    end
end

return SettingsModule
```

---

## 2. Settings Persistence

### 2.1 Save/Load ด้วย DataStore

```lua
-- SettingsSaver.lua (Server Script)
-- บันทึกและโหลด Settings ของ Player

local DataStoreService = game:GetService("DataStoreService")
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local settingsStore = DataStoreService:GetDataStore("PlayerSettings_v2")

-- RemoteEvents สำหรับ Settings
local settingsFolder = Instance.new("Folder")
settingsFolder.Name = "SettingsEvents"
settingsFolder.Parent = ReplicatedStorage

local loadSettings = Instance.new("RemoteFunction")
loadSettings.Name = "LoadSettings"
loadSettings.Parent = settingsFolder

local saveSettings = Instance.new("RemoteEvent")
saveSettings.Name = "SaveSettings"
saveSettings.Parent = settingsFolder

-- Cache Settings ของแต่ละ Player
local playerSettings = {}

-- โหลด Settings เมื่อ Player เข้า
Players.PlayerAdded:Connect(function(player)
    local success, data = pcall(function()
        return settingsStore:GetAsync("settings_" .. player.UserId)
    end)
    
    if success and data then
        playerSettings[player.UserId] = data
        print(string.format("โหลด Settings ของ %s สำเร็จ", player.Name))
    else
        playerSettings[player.UserId] = nil  -- จะใช้ Default
        if not success then
            warn("โหลด Settings ล้มเหลว:", data)
        end
    end
end)

-- ส่ง Settings ให้ Client เมื่อร้องขอ
loadSettings.OnServerInvoke = function(player)
    return playerSettings[player.UserId]  -- nil = ใช้ Default
end

-- รับและบันทึก Settings จาก Client
saveSettings.OnServerEvent:Connect(function(player, newSettings)
    -- Validate settings data
    if type(newSettings) ~= "table" then
        warn("Invalid settings data from", player.Name)
        return
    end
    
    playerSettings[player.UserId] = newSettings
    
    -- บันทึกลง DataStore (ไม่ต้องรอ - fire and forget)
    task.spawn(function()
        local success, err = pcall(function()
            settingsStore:SetAsync("settings_" .. player.UserId, newSettings)
        end)
        
        if not success then
            warn("บันทึก Settings ล้มเหลว:", err)
        end
    end)
end)

-- ล้างข้อมูลเมื่อ Player ออก
Players.PlayerRemoving:Connect(function(player)
    playerSettings[player.UserId] = nil
end)
```

---

## 3. Settings GUI

### 3.1 Main Settings Window

```lua
-- SettingsGUI.lua (Local Script)
-- หน้าต่าง Settings แบบ Tabbed

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- โหลด Settings Module
local SettingsModule = require(ReplicatedStorage:WaitForChild("SettingsModule"))
local settings = SettingsModule.new()

-- โหลด Settings จาก Server
local settingsFolder = ReplicatedStorage:WaitForChild("SettingsEvents")
local loadSettings = settingsFolder:WaitForChild("LoadSettings")
local saveSettings = settingsFolder:WaitForChild("SaveSettings")

local savedData = loadSettings:InvokeServer()
if savedData then
    settings:import(savedData)
end

-- สี Theme
local COLORS = {
    background = Color3.fromRGB(20, 20, 30),
    panel = Color3.fromRGB(30, 30, 45),
    tab = Color3.fromRGB(40, 40, 60),
    tabActive = Color3.fromRGB(60, 100, 180),
    accent = Color3.fromRGB(100, 150, 255),
    text = Color3.fromRGB(220, 220, 220),
    textMuted = Color3.fromRGB(150, 150, 170),
    slider = Color3.fromRGB(60, 60, 80),
    sliderFill = Color3.fromRGB(80, 130, 220),
    toggle = Color3.fromRGB(60, 60, 80),
    toggleOn = Color3.fromRGB(60, 180, 100),
}

-- ===========================
-- สร้าง Main GUI
-- ===========================

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "SettingsGUI"
screenGui.ResetOnSpawn = false
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.Enabled = false
screenGui.Parent = playerGui

-- Overlay (กึ่งโปร่งใสด้านหลัง)
local overlay = Instance.new("Frame")
overlay.Name = "Overlay"
overlay.Size = UDim2.new(1, 0, 1, 0)
overlay.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
overlay.BackgroundTransparency = 0.5
overlay.BorderSizePixel = 0
overlay.Parent = screenGui

-- Main Window
local window = Instance.new("Frame")
window.Name = "Window"
window.Size = UDim2.new(0, 700, 0, 500)
window.Position = UDim2.new(0.5, -350, 0.5, -250)
window.BackgroundColor3 = COLORS.background
window.BorderSizePixel = 0
window.Parent = screenGui

local windowCorner = Instance.new("UICorner")
windowCorner.CornerRadius = UDim.new(0, 12)
windowCorner.Parent = window

-- Title Bar
local titleBar = Instance.new("Frame")
titleBar.Name = "TitleBar"
titleBar.Size = UDim2.new(1, 0, 0, 50)
titleBar.BackgroundColor3 = COLORS.panel
titleBar.BorderSizePixel = 0
titleBar.Parent = window

local titleBarCorner = Instance.new("UICorner")
titleBarCorner.CornerRadius = UDim.new(0, 12)
titleBarCorner.Parent = titleBar

-- Fix bottom corners ของ Title Bar
local titleBarFix = Instance.new("Frame")
titleBarFix.Size = UDim2.new(1, 0, 0.5, 0)
titleBarFix.Position = UDim2.new(0, 0, 0.5, 0)
titleBarFix.BackgroundColor3 = COLORS.panel
titleBarFix.BorderSizePixel = 0
titleBarFix.Parent = titleBar

local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, -60, 1, 0)
titleLabel.Position = UDim2.new(0, 20, 0, 0)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "⚙️ Settings"
titleLabel.TextColor3 = COLORS.text
titleLabel.TextSize = 20
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextXAlignment = Enum.TextXAlignment.Left
titleLabel.Parent = titleBar

-- Close Button
local closeButton = Instance.new("TextButton")
closeButton.Size = UDim2.new(0, 30, 0, 30)
closeButton.Position = UDim2.new(1, -40, 0.5, -15)
closeButton.BackgroundColor3 = Color3.fromRGB(200, 60, 60)
closeButton.Text = "✕"
closeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
closeButton.TextSize = 16
closeButton.Font = Enum.Font.GothamBold
closeButton.BorderSizePixel = 0
closeButton.Parent = titleBar

local closeCorner = Instance.new("UICorner")
closeCorner.CornerRadius = UDim.new(0, 6)
closeCorner.Parent = closeButton

-- Tab Bar
local tabBar = Instance.new("Frame")
tabBar.Name = "TabBar"
tabBar.Size = UDim2.new(0, 150, 1, -50)
tabBar.Position = UDim2.new(0, 0, 0, 50)
tabBar.BackgroundColor3 = COLORS.panel
tabBar.BorderSizePixel = 0
tabBar.Parent = window

-- Content Area
local contentArea = Instance.new("ScrollingFrame")
contentArea.Name = "ContentArea"
contentArea.Size = UDim2.new(1, -150, 1, -50)
contentArea.Position = UDim2.new(0, 150, 0, 50)
contentArea.BackgroundColor3 = COLORS.background
contentArea.BorderSizePixel = 0
contentArea.ScrollBarThickness = 4
contentArea.ScrollBarImageColor3 = COLORS.accent
contentArea.CanvasSize = UDim2.new(0, 0, 0, 0)
contentArea.AutomaticCanvasSize = Enum.AutomaticSize.Y
contentArea.Parent = window

-- Content Padding
local contentPadding = Instance.new("UIPadding")
contentPadding.PaddingLeft = UDim.new(0, 20)
contentPadding.PaddingRight = UDim.new(0, 20)
contentPadding.PaddingTop = UDim.new(0, 20)
contentPadding.PaddingBottom = UDim.new(0, 20)
contentPadding.Parent = contentArea

-- Content Layout
local contentLayout = Instance.new("UIListLayout")
contentLayout.SortOrder = Enum.SortOrder.LayoutOrder
contentLayout.Padding = UDim.new(0, 15)
contentLayout.Parent = contentArea

-- ===========================
-- Utility Functions สำหรับสร้าง Controls
-- ===========================

-- สร้าง Section Header
local function createHeader(parent, text, order)
    local header = Instance.new("TextLabel")
    header.Name = "Header_" .. text
    header.Size = UDim2.new(1, 0, 0, 30)
    header.BackgroundTransparency = 1
    header.Text = text
    header.TextColor3 = COLORS.accent
    header.TextSize = 16
    header.Font = Enum.Font.GothamBold
    header.TextXAlignment = Enum.TextXAlignment.Left
    header.LayoutOrder = order or 0
    header.Parent = parent
    
    -- Divider Line
    local divider = Instance.new("Frame")
    divider.Size = UDim2.new(1, 0, 0, 1)
    divider.BackgroundColor3 = COLORS.accent
    divider.BackgroundTransparency = 0.7
    divider.BorderSizePixel = 0
    divider.Parent = header
    divider.Position = UDim2.new(0, 0, 1, 0)
    
    return header
end

-- สร้าง Slider
local function createSlider(parent, label, settingPath, min, max, order)
    local currentValue = settings:get(settingPath) or min
    
    local container = Instance.new("Frame")
    container.Name = "Slider_" .. label
    container.Size = UDim2.new(1, 0, 0, 60)
    container.BackgroundTransparency = 1
    container.LayoutOrder = order or 0
    container.Parent = parent
    
    -- Label
    local labelText = Instance.new("TextLabel")
    labelText.Size = UDim2.new(0.6, 0, 0, 20)
    labelText.BackgroundTransparency = 1
    labelText.Text = label
    labelText.TextColor3 = COLORS.text
    labelText.TextSize = 14
    labelText.Font = Enum.Font.Gotham
    labelText.TextXAlignment = Enum.TextXAlignment.Left
    labelText.Parent = container
    
    -- Value Label
    local valueLabel = Instance.new("TextLabel")
    valueLabel.Size = UDim2.new(0.4, 0, 0, 20)
    valueLabel.Position = UDim2.new(0.6, 0, 0, 0)
    valueLabel.BackgroundTransparency = 1
    valueLabel.Text = string.format("%.2f", currentValue)
    valueLabel.TextColor3 = COLORS.textMuted
    valueLabel.TextSize = 13
    valueLabel.Font = Enum.Font.GothamBold
    valueLabel.TextXAlignment = Enum.TextXAlignment.Right
    valueLabel.Parent = container
    
    -- Slider Track
    local track = Instance.new("Frame")
    track.Size = UDim2.new(1, 0, 0, 8)
    track.Position = UDim2.new(0, 0, 0, 30)
    track.BackgroundColor3 = COLORS.slider
    track.BorderSizePixel = 0
    track.Parent = container
    
    local trackCorner = Instance.new("UICorner")
    trackCorner.CornerRadius = UDim.new(1, 0)
    trackCorner.Parent = track
    
    -- Fill
    local fill = Instance.new("Frame")
    fill.Name = "Fill"
    local fillRatio = (currentValue - min) / (max - min)
    fill.Size = UDim2.new(fillRatio, 0, 1, 0)
    fill.BackgroundColor3 = COLORS.sliderFill
    fill.BorderSizePixel = 0
    fill.Parent = track
    
    local fillCorner = Instance.new("UICorner")
    fillCorner.CornerRadius = UDim.new(1, 0)
    fillCorner.Parent = fill
    
    -- Handle (ปุ่มกลม)
    local handle = Instance.new("Frame")
    handle.Name = "Handle"
    handle.Size = UDim2.new(0, 18, 0, 18)
    handle.Position = UDim2.new(fillRatio, -9, 0.5, -9)
    handle.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    handle.BorderSizePixel = 0
    handle.ZIndex = 2
    handle.Parent = track
    
    local handleCorner = Instance.new("UICorner")
    handleCorner.CornerRadius = UDim.new(1, 0)
    handleCorner.Parent = handle
    
    -- Interaction
    local isDragging = false
    
    track.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            isDragging = true
        end
    end)
    
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            isDragging = false
        end
    end)
    
    UserInputService.InputChanged:Connect(function(input)
        if isDragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            local trackPos = track.AbsolutePosition
            local trackSize = track.AbsoluteSize
            
            local relativeX = (input.Position.X - trackPos.X) / trackSize.X
            relativeX = math.clamp(relativeX, 0, 1)
            
            local newValue = min + (max - min) * relativeX
            -- Round ถ้าเป็น Integer
            if max - min == math.floor(max - min) and (max - min) <= 100 then
                newValue = math.round(newValue)
            end
            
            -- อัพเดท Visual
            fill.Size = UDim2.new(relativeX, 0, 1, 0)
            handle.Position = UDim2.new(relativeX, -9, 0.5, -9)
            valueLabel.Text = string.format("%.2f", newValue)
            
            -- อัพเดท Setting
            settings:set(settingPath, newValue)
        end
    end)
    
    return container
end

-- สร้าง Toggle
local function createToggle(parent, label, settingPath, order)
    local currentValue = settings:get(settingPath)
    
    local container = Instance.new("Frame")
    container.Name = "Toggle_" .. label
    container.Size = UDim2.new(1, 0, 0, 40)
    container.BackgroundTransparency = 1
    container.LayoutOrder = order or 0
    container.Parent = parent
    
    -- Label
    local labelText = Instance.new("TextLabel")
    labelText.Size = UDim2.new(0.8, 0, 1, 0)
    labelText.BackgroundTransparency = 1
    labelText.Text = label
    labelText.TextColor3 = COLORS.text
    labelText.TextSize = 14
    labelText.Font = Enum.Font.Gotham
    labelText.TextXAlignment = Enum.TextXAlignment.Left
    labelText.Parent = container
    
    -- Toggle Button
    local toggleBg = Instance.new("Frame")
    toggleBg.Size = UDim2.new(0, 50, 0, 26)
    toggleBg.Position = UDim2.new(1, -50, 0.5, -13)
    toggleBg.BackgroundColor3 = currentValue and COLORS.toggleOn or COLORS.toggle
    toggleBg.BorderSizePixel = 0
    toggleBg.Parent = container
    
    local toggleCorner = Instance.new("UICorner")
    toggleCorner.CornerRadius = UDim.new(1, 0)
    toggleCorner.Parent = toggleBg
    
    local toggleKnob = Instance.new("Frame")
    toggleKnob.Size = UDim2.new(0, 20, 0, 20)
    toggleKnob.Position = currentValue and UDim2.new(1, -23, 0.5, -10) or UDim2.new(0, 3, 0.5, -10)
    toggleKnob.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    toggleKnob.BorderSizePixel = 0
    toggleKnob.Parent = toggleBg
    
    local knobCorner = Instance.new("UICorner")
    knobCorner.CornerRadius = UDim.new(1, 0)
    knobCorner.Parent = toggleKnob
    
    -- Click Handler
    local toggleButton = Instance.new("TextButton")
    toggleButton.Size = UDim2.new(1, 0, 1, 0)
    toggleButton.BackgroundTransparency = 1
    toggleButton.Text = ""
    toggleButton.Parent = toggleBg
    
    toggleButton.Activated:Connect(function()
        local newValue = not settings:get(settingPath)
        settings:set(settingPath, newValue)
        
        -- Animate Toggle
        TweenService:Create(toggleBg, TweenInfo.new(0.2), {
            BackgroundColor3 = newValue and COLORS.toggleOn or COLORS.toggle
        }):Play()
        
        TweenService:Create(toggleKnob, TweenInfo.new(0.2), {
            Position = newValue and UDim2.new(1, -23, 0.5, -10) or UDim2.new(0, 3, 0.5, -10)
        }):Play()
    end)
    
    return container
end

-- ===========================
-- สร้าง Tabs
-- ===========================

local TABS = {
    {name = "🎮 General", key = "general"},
    {name = "🎨 Graphics", key = "graphics"},
    {name = "🔊 Audio", key = "audio"},
    {name = "🎯 Controls", key = "controls"},
    {name = "📱 Interface", key = "ui"},
    {name = "♿ Accessibility", key = "accessibility"},
}

local tabPages = {}
local activeTab = nil

-- Tab List Layout
local tabListLayout = Instance.new("UIListLayout")
tabListLayout.SortOrder = Enum.SortOrder.LayoutOrder
tabListLayout.Padding = UDim.new(0, 2)
tabListLayout.Parent = tabBar

local tabPadding = Instance.new("UIPadding")
tabPadding.PaddingTop = UDim.new(0, 10)
tabPadding.PaddingLeft = UDim.new(0, 5)
tabPadding.PaddingRight = UDim.new(0, 5)
tabPadding.Parent = tabBar

local function switchTab(tabKey)
    -- ซ่อนทุก Page
    for key, page in pairs(tabPages) do
        page.Visible = false
    end
    
    -- แสดง Active Page
    if tabPages[tabKey] then
        tabPages[tabKey].Visible = true
    end
    
    -- อัพเดท Tab Buttons
    for _, button in ipairs(tabBar:GetChildren()) do
        if button:IsA("TextButton") then
            if button.Name == "Tab_" .. tabKey then
                button.BackgroundColor3 = COLORS.tabActive
                button.TextColor3 = Color3.fromRGB(255, 255, 255)
            else
                button.BackgroundColor3 = COLORS.tab
                button.TextColor3 = COLORS.textMuted
            end
        end
    end
    
    activeTab = tabKey
end

-- สร้าง Tab Buttons
for i, tab in ipairs(TABS) do
    local tabButton = Instance.new("TextButton")
    tabButton.Name = "Tab_" .. tab.key
    tabButton.Size = UDim2.new(1, 0, 0, 40)
    tabButton.BackgroundColor3 = COLORS.tab
    tabButton.Text = tab.name
    tabButton.TextColor3 = COLORS.textMuted
    tabButton.TextSize = 13
    tabButton.Font = Enum.Font.Gotham
    tabButton.BorderSizePixel = 0
    tabButton.LayoutOrder = i
    tabButton.TextXAlignment = Enum.TextXAlignment.Left
    tabButton.Parent = tabBar
    
    local tabCorner = Instance.new("UICorner")
    tabCorner.CornerRadius = UDim.new(0, 6)
    tabCorner.Parent = tabButton
    
    local tabPaddingInner = Instance.new("UIPadding")
    tabPaddingInner.PaddingLeft = UDim.new(0, 10)
    tabPaddingInner.Parent = tabButton
    
    -- สร้าง Page สำหรับ Tab นี้
    local page = Instance.new("Frame")
    page.Name = "Page_" .. tab.key
    page.Size = UDim2.new(1, 0, 0, 0)
    page.AutomaticSize = Enum.AutomaticSize.Y
    page.BackgroundTransparency = 1
    page.Visible = false
    page.LayoutOrder = i
    page.Parent = contentArea
    
    local pageLayout = Instance.new("UIListLayout")
    pageLayout.SortOrder = Enum.SortOrder.LayoutOrder
    pageLayout.Padding = UDim.new(0, 10)
    pageLayout.Parent = page
    
    tabPages[tab.key] = page
    
    tabButton.Activated:Connect(function()
        switchTab(tab.key)
    end)
end

-- ===========================
-- สร้าง Graphics Settings
-- ===========================

local graphicsPage = tabPages["graphics"]

createHeader(graphicsPage, "คุณภาพกราฟิก", 1)
createSlider(graphicsPage, "ระดับคุณภาพ (1-10)", "graphics.qualityLevel", 1, 10, 2)
createSlider(graphicsPage, "Field of View", "graphics.fov", 60, 120, 3)
createToggle(graphicsPage, "เงา (Shadows)", "graphics.shadows", 4)
createToggle(graphicsPage, "Particle Effects", "graphics.particles", 5)

-- ===========================
-- สร้าง Audio Settings
-- ===========================

local audioPage = tabPages["audio"]

createHeader(audioPage, "การตั้งค่าเสียง", 1)
createSlider(audioPage, "ระดับเสียงรวม", "audio.masterVolume", 0, 1, 2)
createSlider(audioPage, "เสียงเพลง", "audio.musicVolume", 0, 1, 3)
createSlider(audioPage, "เสียง SFX", "audio.sfxVolume", 0, 1, 4)
createToggle(audioPage, "เปิด Voice Chat", "audio.voiceEnabled", 5)

-- ===========================
-- ส่วน Save/Close
-- ===========================

-- Save Button
local saveButton = Instance.new("TextButton")
saveButton.Name = "SaveButton"
saveButton.Size = UDim2.new(0, 120, 0, 40)
saveButton.Position = UDim2.new(1, -130, 1, -50)
saveButton.BackgroundColor3 = Color3.fromRGB(60, 180, 100)
saveButton.Text = "💾 บันทึก"
saveButton.TextColor3 = Color3.fromRGB(255, 255, 255)
saveButton.TextSize = 14
saveButton.Font = Enum.Font.GothamBold
saveButton.BorderSizePixel = 0
saveButton.Parent = window

local saveCorner = Instance.new("UICorner")
saveCorner.CornerRadius = UDim.new(0, 8)
saveCorner.Parent = saveButton

saveButton.Activated:Connect(function()
    -- บันทึก Settings
    saveSettings:FireServer(settings:export())
    
    -- Visual Feedback
    saveButton.Text = "✅ บันทึกแล้ว!"
    saveButton.BackgroundColor3 = Color3.fromRGB(40, 120, 70)
    
    task.delay(2, function()
        if saveButton.Parent then
            saveButton.Text = "💾 บันทึก"
            saveButton.BackgroundColor3 = Color3.fromRGB(60, 180, 100)
        end
    end)
end)

-- Close Button Handler
closeButton.Activated:Connect(function()
    screenGui.Enabled = false
end)

-- Click Overlay to close
overlay.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        screenGui.Enabled = false
    end
end)

-- เปิด/ปิด Settings ด้วย ESC หรือ Hotkey
UserInputService.InputBegan:Connect(function(input, processed)
    if processed then return end
    
    if input.KeyCode == Enum.KeyCode.Escape then
        screenGui.Enabled = not screenGui.Enabled
    end
end)

-- เริ่มที่ Tab แรก
switchTab("graphics")

-- Apply Settings เมื่อ Load
settings:onChange("*", function(path, newValue)
    applySettingToGame(path, newValue)
end)

-- ฟังก์ชันใช้ Setting กับเกม
function applySettingToGame(path, value)
    if path == "graphics.qualityLevel" then
        settings:GetService and settings:GetService("Lighting")
    elseif path == "audio.masterVolume" then
        local soundService = game:GetService("SoundService")
        soundService.RespectFilteringEnabled = true
        -- ใช้ VolumeMultiplier
    elseif path == "graphics.fov" then
        workspace.CurrentCamera.FieldOfView = value
    end
end
```

---

## 4. ข้อผิดพลาดที่พบบ่อย

### ❌ ข้อผิดพลาดที่ 1: ไม่ Validate Settings Data

```lua
-- ❌ แบบผิด: รับข้อมูลจาก Client โดยตรง
saveSettings.OnServerEvent:Connect(function(player, data)
    playerSettings[player.UserId] = data  -- อันตราย! Client อาจส่งข้อมูลมั่ว
    settingsStore:SetAsync(key, data)
end)

-- ✅ แบบถูก: Validate ก่อนบันทึก
saveSettings.OnServerEvent:Connect(function(player, data)
    if type(data) ~= "table" then return end
    
    -- ตรวจสอบและ Sanitize
    local cleanData = {}
    for category, values in pairs(DEFAULT_SETTINGS) do
        cleanData[category] = {}
        if type(data[category]) == "table" then
            for key, defaultValue in pairs(values) do
                local value = data[category][key]
                -- ตรวจสอบ Type
                if type(value) == type(defaultValue) then
                    cleanData[category][key] = value
                else
                    cleanData[category][key] = defaultValue
                end
            end
        else
            cleanData[category] = values  -- ใช้ Default
        end
    end
    
    playerSettings[player.UserId] = cleanData
end)
```

### ❌ ข้อผิดพลาดที่ 2: Save ทุกครั้งที่ Setting เปลี่ยน

```lua
-- ❌ แบบผิด: บันทึกทุกครั้ง - เกิน DataStore Rate Limit!
settings:onChange("*", function()
    saveSettings:FireServer(settings:export())  -- เรียกบ่อยมาก
end)

-- ✅ แบบถูก: Debounce การ Save
local savePending = false
settings:onChange("*", function()
    if savePending then return end
    savePending = true
    
    task.delay(2, function()  -- รอ 2 วินาทีก่อนบันทึก
        saveSettings:FireServer(settings:export())
        savePending = false
    end)
end)
```

---

## 5. แบบฝึกหัด

### แบบฝึกหัดที่ 1: Hotkey Settings
เพิ่มระบบ Keybinding ใน Settings:
- ให้ Player เปลี่ยน Key สำหรับ Action ต่างๆ
- บันทึก Custom Keybindings
- ตรวจสอบว่าไม่มี Conflict Keys

### แบบฝึกหัดที่ 2: Graphics Presets
สร้างระบบ Preset:
- Low, Medium, High, Ultra
- แต่ละ Preset ตั้งค่า Graphics ทุกตัวพร้อมกัน
- ผู้เล่นสามารถสร้าง Custom Preset ได้

### แบบฝึกหัดที่ 3: Settings Export/Import
สร้างฟีเจอร์ Export Settings เป็น Code:
- แปลง Settings เป็น String
- ผู้เล่นสามารถ Copy และแชร์ Settings ได้
- Import Settings จาก Code ที่ได้รับ

---

## สรุป

Settings Menu ที่ดีต้องมี:
1. **Persistence**: บันทึกและโหลด Settings อัตโนมัติ
2. **Validation**: ตรวจสอบค่าก่อนบันทึก
3. **Real-time Apply**: เห็นผลทันทีเมื่อเปลี่ยน
4. **Good UX**: Controls ที่ใช้งานง่าย ชัดเจน
5. **Security**: อย่าไว้ใจ Client Data โดยตรง

ในส่วนถัดไป (Part 76) เราจะสร้าง Cutscene System ที่สวยงาม พร้อมระบบ Camera Animation และ Dialogue
