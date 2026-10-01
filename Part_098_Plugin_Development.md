# Part 98: Roblox Studio Plugin Development

## บทนำ (Introduction)

Plugins คือเครื่องมือที่ทำงานภายใน Roblox Studio เพื่อช่วย developer ในการทำงานได้เร็วขึ้น
ในบทนี้เราจะเรียนรู้วิธีสร้าง Plugin ตั้งแต่ Widget การ toolbar ไปจนถึงการเปลี่ยนแปลง workspace

### สิ่งที่จะได้เรียนรู้:
- Plugin API พื้นฐาน
- สร้าง PluginToolbar และ Button
- สร้าง Widget (DockWidgetPluginGui)
- อ่านและแก้ไข Instance ใน workspace
- Plugin settings และ configuration
- Undo/Redo system ใน plugin
- การ publish plugin ไปยัง marketplace
- ตัวอย่าง plugin จริง: Map Generator, Mass Renamer

---

## ส่วนที่ 1: พื้นฐาน Plugin API

### 1.1 Plugin Structure

```lua
-- Script: Plugin Main (ต้องอยู่ใน Plugin container)
-- โครงสร้างพื้นฐานของ Plugin
-- Basic Plugin structure

-- Plugin object ที่ Roblox ให้มาอัตโนมัติ
-- (The plugin object is automatically provided by Roblox)
-- สามารถเรียกใช้ตัวแปร `plugin` ได้เลย

-- ==================== สร้าง Toolbar ====================
local toolbar = plugin:CreateToolbar("My Plugin Tools")

-- สร้าง button ใน toolbar
local mainButton = toolbar:CreateButton(
    "Main Tool",                    -- ชื่อ button
    "Click to open main tool",      -- Tooltip
    "rbxassetid://0"                -- Icon (0 = ไม่มี icon)
)

-- ==================== สร้าง Widget ====================
-- DockWidgetPluginGuiInfo กำหนดพฤติกรรมของ widget
local widgetInfo = DockWidgetPluginGuiInfo.new(
    Enum.InitialDockState.Right,    -- ตำแหน่งเริ่มต้น (dock ทางขวา)
    true,                           -- เปิดตอน start
    false,                          -- ไม่ override ขนาด
    300,                            -- ความกว้างเริ่มต้น
    500,                            -- ความสูงเริ่มต้น
    200,                            -- ความกว้างต่ำสุด
    300                             -- ความสูงต่ำสุด
)

local widget = plugin:CreateDockWidgetPluginGui(
    "MyPluginWidget",   -- ID unique สำหรับ widget นี้
    widgetInfo
)
widget.Title = "My Plugin"
widget.ResetOnSpawn = false

-- ==================== เปิด/ปิด Widget ====================
local isOpen = false

mainButton.Click:Connect(function()
    isOpen = not isOpen
    widget.Enabled = isOpen
    mainButton:SetActive(isOpen)
end)

-- Sync state เมื่อ widget ถูกปิดจากปุ่ม X
widget:GetPropertyChangedSignal("Enabled"):Connect(function()
    if not widget.Enabled then
        isOpen = false
        mainButton:SetActive(false)
    end
end)

print("Plugin loaded!")
```

### 1.2 Plugin Settings

```lua
-- ModuleScript: PluginSettings
-- จัดการการตั้งค่า Plugin ที่ persist ระหว่าง sessions
-- Manage Plugin settings that persist between sessions

local PluginSettings = {}
PluginSettings.__index = PluginSettings

-- สร้าง settings manager (Create settings manager)
function PluginSettings.new(pluginObject, namespace)
    local self = setmetatable({}, PluginSettings)
    
    self.Plugin = pluginObject
    self.Namespace = namespace or "MyPlugin"
    self.Cache = {}
    
    return self
end

-- บันทึกค่า (Save value)
function PluginSettings:Set(key, value)
    local fullKey = self.Namespace .. "_" .. key
    self.Plugin:SetSetting(fullKey, value)
    self.Cache[key] = value
end

-- อ่านค่า (Get value)
function PluginSettings:Get(key, default)
    -- ดูจาก cache ก่อน
    if self.Cache[key] ~= nil then
        return self.Cache[key]
    end
    
    local fullKey = self.Namespace .. "_" .. key
    local value = self.Plugin:GetSetting(fullKey)
    
    if value == nil then
        return default
    end
    
    self.Cache[key] = value
    return value
end

-- ลบค่า (Delete value)
function PluginSettings:Delete(key)
    local fullKey = self.Namespace .. "_" .. key
    self.Plugin:SetSetting(fullKey, nil)
    self.Cache[key] = nil
end

-- ==================== ตัวอย่างการใช้งาน ====================
-- local settings = PluginSettings.new(plugin, "MapGenerator")
-- settings:Set("GridSize", 10)
-- local gridSize = settings:Get("GridSize", 5)

return PluginSettings
```

---

## ส่วนที่ 2: Plugin ตัวอย่าง - Mass Instance Renamer

### 2.1 Mass Renamer Plugin

```lua
-- Plugin: MassRenamer
-- Plugin สำหรับเปลี่ยนชื่อ instances หลายตัวพร้อมกัน
-- Plugin for renaming multiple instances at once

-- ==================== Toolbar Setup ====================
local toolbar = plugin:CreateToolbar("Utilities")

local renameButton = toolbar:CreateButton(
    "Mass Rename",
    "Rename multiple instances at once",
    "rbxassetid://0"
)

-- ==================== Widget Setup ====================
local widgetInfo = DockWidgetPluginGuiInfo.new(
    Enum.InitialDockState.Float,
    false,
    false,
    350,
    500,
    300,
    400
)

local widget = plugin:CreateDockWidgetPluginGui("MassRenamer", widgetInfo)
widget.Title = "Mass Renamer"

-- ==================== UI Construction ====================
local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(1, 0, 1, 0)
mainFrame.BackgroundColor3 = Color3.fromRGB(46, 46, 46)
mainFrame.BorderSizePixel = 0
mainFrame.Parent = widget

-- Title
local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, 0, 0, 40)
titleLabel.Position = UDim2.new(0, 0, 0, 0)
titleLabel.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
titleLabel.Text = "Mass Instance Renamer"
titleLabel.TextSize = 16
titleLabel.Font = Enum.Font.GothamBold
titleLabel.BorderSizePixel = 0
titleLabel.Parent = mainFrame

-- Section: Find & Replace
local findLabel = Instance.new("TextLabel")
findLabel.Size = UDim2.new(1, -20, 0, 25)
findLabel.Position = UDim2.new(0, 10, 0, 50)
findLabel.BackgroundTransparency = 1
findLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
findLabel.Text = "Find:"
findLabel.TextXAlignment = Enum.TextXAlignment.Left
findLabel.TextSize = 14
findLabel.Font = Enum.Font.Gotham
findLabel.Parent = mainFrame

local findInput = Instance.new("TextBox")
findInput.Size = UDim2.new(1, -20, 0, 35)
findInput.Position = UDim2.new(0, 10, 0, 75)
findInput.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
findInput.TextColor3 = Color3.fromRGB(255, 255, 255)
findInput.PlaceholderText = "Text to find..."
findInput.Text = ""
findInput.TextSize = 14
findInput.Font = Enum.Font.Gotham
findInput.ClearTextOnFocus = false
findInput.BorderSizePixel = 0
findInput.Parent = mainFrame

local replaceLabel = Instance.new("TextLabel")
replaceLabel.Size = UDim2.new(1, -20, 0, 25)
replaceLabel.Position = UDim2.new(0, 10, 0, 120)
replaceLabel.BackgroundTransparency = 1
replaceLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
replaceLabel.Text = "Replace with:"
replaceLabel.TextXAlignment = Enum.TextXAlignment.Left
replaceLabel.TextSize = 14
replaceLabel.Font = Enum.Font.Gotham
replaceLabel.Parent = mainFrame

local replaceInput = Instance.new("TextBox")
replaceInput.Size = UDim2.new(1, -20, 0, 35)
replaceInput.Position = UDim2.new(0, 10, 0, 145)
replaceInput.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
replaceInput.TextColor3 = Color3.fromRGB(255, 255, 255)
replaceInput.PlaceholderText = "Replacement text..."
replaceInput.Text = ""
replaceInput.TextSize = 14
replaceInput.Font = Enum.Font.Gotham
replaceInput.ClearTextOnFocus = false
replaceInput.BorderSizePixel = 0
replaceInput.Parent = mainFrame

-- Options checkboxes
local caseSensitiveCheck = Instance.new("TextButton")
caseSensitiveCheck.Size = UDim2.new(0, 200, 0, 30)
caseSensitiveCheck.Position = UDim2.new(0, 10, 0, 195)
caseSensitiveCheck.BackgroundTransparency = 1
caseSensitiveCheck.TextColor3 = Color3.fromRGB(200, 200, 200)
caseSensitiveCheck.Text = "☐ Case Sensitive"
caseSensitiveCheck.TextXAlignment = Enum.TextXAlignment.Left
caseSensitiveCheck.TextSize = 13
caseSensitiveCheck.Font = Enum.Font.Gotham
caseSensitiveCheck.Parent = mainFrame

local isCaseSensitive = false
caseSensitiveCheck.MouseButton1Click:Connect(function()
    isCaseSensitive = not isCaseSensitive
    caseSensitiveCheck.Text = isCaseSensitive and "☑ Case Sensitive" or "☐ Case Sensitive"
end)

local useRegexCheck = Instance.new("TextButton")
useRegexCheck.Size = UDim2.new(0, 200, 0, 30)
useRegexCheck.Position = UDim2.new(0, 10, 0, 225)
useRegexCheck.BackgroundTransparency = 1
useRegexCheck.TextColor3 = Color3.fromRGB(200, 200, 200)
useRegexCheck.Text = "☐ Use Pattern Matching"
useRegexCheck.TextXAlignment = Enum.TextXAlignment.Left
useRegexCheck.TextSize = 13
useRegexCheck.Font = Enum.Font.Gotham
useRegexCheck.Parent = mainFrame

local usePattern = false
useRegexCheck.MouseButton1Click:Connect(function()
    usePattern = not usePattern
    useRegexCheck.Text = usePattern and "☑ Use Pattern Matching" or "☐ Use Pattern Matching"
end)

-- Scope selector
local scopeLabel = Instance.new("TextLabel")
scopeLabel.Size = UDim2.new(1, -20, 0, 25)
scopeLabel.Position = UDim2.new(0, 10, 0, 265)
scopeLabel.BackgroundTransparency = 1
scopeLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
scopeLabel.Text = "Scope:"
scopeLabel.TextXAlignment = Enum.TextXAlignment.Left
scopeLabel.TextSize = 14
scopeLabel.Font = Enum.Font.Gotham
scopeLabel.Parent = mainFrame

local scopeOptions = {"Selection", "Workspace", "ServerScriptService", "ReplicatedStorage"}
local currentScope = 1

local scopeButton = Instance.new("TextButton")
scopeButton.Size = UDim2.new(1, -20, 0, 35)
scopeButton.Position = UDim2.new(0, 10, 0, 290)
scopeButton.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
scopeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
scopeButton.Text = scopeOptions[currentScope]
scopeButton.TextSize = 14
scopeButton.Font = Enum.Font.Gotham
scopeButton.BorderSizePixel = 0
scopeButton.Parent = mainFrame

scopeButton.MouseButton1Click:Connect(function()
    currentScope = (currentScope % #scopeOptions) + 1
    scopeButton.Text = scopeOptions[currentScope]
end)

-- Result label
local resultLabel = Instance.new("TextLabel")
resultLabel.Size = UDim2.new(1, -20, 0, 40)
resultLabel.Position = UDim2.new(0, 10, 0, 335)
resultLabel.BackgroundTransparency = 1
resultLabel.TextColor3 = Color3.fromRGB(150, 220, 150)
resultLabel.Text = ""
resultLabel.TextXAlignment = Enum.TextXAlignment.Left
resultLabel.TextSize = 13
resultLabel.Font = Enum.Font.Gotham
resultLabel.TextWrapped = true
resultLabel.Parent = mainFrame

-- Preview button
local previewButton = Instance.new("TextButton")
previewButton.Size = UDim2.new(0.48, -10, 0, 40)
previewButton.Position = UDim2.new(0, 10, 0, 385)
previewButton.BackgroundColor3 = Color3.fromRGB(70, 130, 180)
previewButton.TextColor3 = Color3.fromRGB(255, 255, 255)
previewButton.Text = "Preview"
previewButton.TextSize = 15
previewButton.Font = Enum.Font.GothamBold
previewButton.BorderSizePixel = 0
previewButton.Parent = mainFrame

-- Replace button
local replaceButton = Instance.new("TextButton")
replaceButton.Size = UDim2.new(0.48, -5, 0, 40)
replaceButton.Position = UDim2.new(0.52, -5, 0, 385)
replaceButton.BackgroundColor3 = Color3.fromRGB(34, 139, 34)
replaceButton.TextColor3 = Color3.fromRGB(255, 255, 255)
replaceButton.Text = "Replace All"
replaceButton.TextSize = 15
replaceButton.Font = Enum.Font.GothamBold
replaceButton.BorderSizePixel = 0
replaceButton.Parent = mainFrame

-- ==================== Logic ====================

-- หา instances ตาม scope (Find instances by scope)
local function getTargetInstances()
    local scopeName = scopeOptions[currentScope]
    
    if scopeName == "Selection" then
        return game:GetService("Selection"):Get()
    elseif scopeName == "Workspace" then
        return workspace:GetDescendants()
    elseif scopeName == "ServerScriptService" then
        return game:GetService("ServerScriptService"):GetDescendants()
    elseif scopeName == "ReplicatedStorage" then
        return game:GetService("ReplicatedStorage"):GetDescendants()
    end
    
    return {}
end

-- ตรวจสอบว่าชื่อตรงกับ pattern หรือไม่ (Check if name matches pattern)
local function nameMatches(name, pattern, caseSensitive, isPattern)
    if isPattern then
        local flags = caseSensitive and "" or "i"
        return name:match(pattern) ~= nil
    else
        if caseSensitive then
            return name:find(pattern, 1, true) ~= nil
        else
            return name:lower():find(pattern:lower(), 1, true) ~= nil
        end
    end
end

-- แทนที่ชื่อ (Replace name)
local function replaceName(name, findText, replaceText, caseSensitive, isPattern)
    if isPattern then
        local flags = caseSensitive and "" or "i"
        return (name:gsub(findText, replaceText))
    else
        if caseSensitive then
            return (name:gsub(findText:gsub("([%^%$%(%)%%%.%[%]%*%+%-%?])", "%%%1"), replaceText))
        else
            -- Case insensitive replacement
            local result = name
            local lowerName = name:lower()
            local lowerFind = findText:lower()
            
            local i = 1
            local newResult = ""
            
            while i <= #result do
                local start = lowerName:find(lowerFind, i, true)
                if start then
                    newResult = newResult .. result:sub(i, start - 1) .. replaceText
                    i = start + #findText
                else
                    newResult = newResult .. result:sub(i)
                    break
                end
            end
            
            return newResult
        end
    end
end

-- Preview function
previewButton.MouseButton1Click:Connect(function()
    local findText = findInput.Text
    local replaceText = replaceInput.Text
    
    if findText == "" then
        resultLabel.TextColor3 = Color3.fromRGB(220, 100, 100)
        resultLabel.Text = "Error: Find text cannot be empty"
        return
    end
    
    local instances = getTargetInstances()
    local count = 0
    local previewList = {}
    
    for _, instance in ipairs(instances) do
        if nameMatches(instance.Name, findText, isCaseSensitive, usePattern) then
            count = count + 1
            local newName = replaceName(instance.Name, findText, replaceText, isCaseSensitive, usePattern)
            
            if count <= 5 then
                table.insert(previewList, instance.Name .. " → " .. newName)
            end
        end
    end
    
    resultLabel.TextColor3 = Color3.fromRGB(150, 220, 150)
    
    if count == 0 then
        resultLabel.Text = "No matches found"
    else
        local previewText = "Will rename " .. count .. " instance(s)\n"
        previewText = previewText .. table.concat(previewList, "\n")
        if count > 5 then
            previewText = previewText .. "\n... and " .. (count - 5) .. " more"
        end
        resultLabel.Text = previewText
    end
end)

-- Replace function
replaceButton.MouseButton1Click:Connect(function()
    local findText = findInput.Text
    local replaceText = replaceInput.Text
    
    if findText == "" then
        resultLabel.TextColor3 = Color3.fromRGB(220, 100, 100)
        resultLabel.Text = "Error: Find text cannot be empty"
        return
    end
    
    local instances = getTargetInstances()
    local renamed = {}
    
    -- เก็บ history สำหรับ undo (Track changes for undo)
    for _, instance in ipairs(instances) do
        if nameMatches(instance.Name, findText, isCaseSensitive, usePattern) then
            local oldName = instance.Name
            local newName = replaceName(instance.Name, findText, replaceText, isCaseSensitive, usePattern)
            
            if newName ~= oldName then
                table.insert(renamed, {instance = instance, oldName = oldName, newName = newName})
            end
        end
    end
    
    if #renamed == 0 then
        resultLabel.TextColor3 = Color3.fromRGB(220, 220, 100)
        resultLabel.Text = "No matches found"
        return
    end
    
    -- ใช้ plugin Undo system
    plugin:SetSetting("_lastRename", os.clock())
    
    -- ดำเนินการเปลี่ยนชื่อ (Perform renaming)
    local changeHistoryService = game:GetService("ChangeHistoryService")
    changeHistoryService:SetWaypoint("Before Mass Rename")
    
    for _, change in ipairs(renamed) do
        change.instance.Name = change.newName
    end
    
    changeHistoryService:SetWaypoint("After Mass Rename")
    
    resultLabel.TextColor3 = Color3.fromRGB(150, 220, 150)
    resultLabel.Text = "Successfully renamed " .. #renamed .. " instance(s)"
end)

-- ==================== Toggle Widget ====================
renameButton.Click:Connect(function()
    widget.Enabled = not widget.Enabled
    renameButton:SetActive(widget.Enabled)
end)
```

---

## ส่วนที่ 3: Plugin ตัวอย่าง - Terrain Painter

### 3.1 Terrain Painting Plugin

```lua
-- Plugin: TerrainPainter
-- Plugin สำหรับวาด terrain material
-- Plugin for painting terrain materials

local toolbar = plugin:CreateToolbar("Terrain Tools")

local paintButton = toolbar:CreateButton(
    "Terrain Painter",
    "Paint terrain with materials",
    "rbxassetid://0"
)

-- Widget
local widgetInfo = DockWidgetPluginGuiInfo.new(
    Enum.InitialDockState.Right,
    false, false, 280, 600, 240, 400
)

local widget = plugin:CreateDockWidgetPluginGui("TerrainPainter", widgetInfo)
widget.Title = "Terrain Painter"

-- ==================== State ====================
local isActive = false
local currentMaterial = Enum.Material.Grass
local brushSize = 10
local brushStrength = 0.5

-- Materials ที่ใช้ได้ (Available materials)
local materials = {
    {name = "Grass", enum = Enum.Material.Grass, color = Color3.fromRGB(106, 127, 63)},
    {name = "Sand", enum = Enum.Material.Sand, color = Color3.fromRGB(198, 189, 129)},
    {name = "Rock", enum = Enum.Material.Rock, color = Color3.fromRGB(102, 108, 111)},
    {name = "Snow", enum = Enum.Material.Snow, color = Color3.fromRGB(200, 220, 240)},
    {name = "Mud", enum = Enum.Material.Mud, color = Color3.fromRGB(97, 79, 57)},
    {name = "Water", enum = Enum.Material.Water, color = Color3.fromRGB(63, 127, 200)},
    {name = "Ground", enum = Enum.Material.Ground, color = Color3.fromRGB(140, 130, 100)},
    {name = "WoodPlanks", enum = Enum.Material.WoodPlanks, color = Color3.fromRGB(180, 140, 100)},
}

-- ==================== UI ====================
local scrollFrame = Instance.new("ScrollingFrame")
scrollFrame.Size = UDim2.new(1, 0, 1, 0)
scrollFrame.BackgroundColor3 = Color3.fromRGB(46, 46, 46)
scrollFrame.BorderSizePixel = 0
scrollFrame.ScrollBarThickness = 6
scrollFrame.Parent = widget

local layout = Instance.new("UIListLayout")
layout.Padding = UDim.new(0, 5)
layout.Parent = scrollFrame

local padding = Instance.new("UIPadding")
padding.PaddingTop = UDim.new(0, 10)
padding.PaddingLeft = UDim.new(0, 10)
padding.PaddingRight = UDim.new(0, 10)
padding.Parent = scrollFrame

-- ฟังก์ชั่นสร้าง label
local function createLabel(text, size)
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, 0, 0, 25)
    label.BackgroundTransparency = 1
    label.TextColor3 = Color3.fromRGB(200, 200, 200)
    label.Text = text
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.TextSize = size or 14
    label.Font = Enum.Font.Gotham
    label.Parent = scrollFrame
    return label
end

-- Brush Size
createLabel("Brush Size: " .. brushSize)

local brushSizeSlider = Instance.new("Frame")
brushSizeSlider.Size = UDim2.new(1, 0, 0, 20)
brushSizeSlider.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
brushSizeSlider.Parent = scrollFrame

-- Material buttons
createLabel("Material:")

local materialGrid = Instance.new("Frame")
materialGrid.Size = UDim2.new(1, 0, 0, 180)
materialGrid.BackgroundTransparency = 1
materialGrid.Parent = scrollFrame

local gridLayout = Instance.new("UIGridLayout")
gridLayout.CellSize = UDim2.new(0.5, -5, 0, 40)
gridLayout.CellPaddingH = UDim.new(0, 5)
gridLayout.Parent = materialGrid

for _, mat in ipairs(materials) do
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 0, 0, 0)  -- GridLayout controls size
    btn.BackgroundColor3 = mat.color
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.Text = mat.name
    btn.TextSize = 11
    btn.Font = Enum.Font.GothamBold
    btn.BorderSizePixel = 1
    btn.Parent = materialGrid
    
    -- เพิ่ม stroke ถ้าเลือก
    local stroke = Instance.new("UIStroke")
    stroke.Color = Color3.fromRGB(255, 255, 255)
    stroke.Thickness = 0
    stroke.Parent = btn
    
    btn.MouseButton1Click:Connect(function()
        currentMaterial = mat.enum
        
        -- อัพเดท highlight
        for _, child in ipairs(materialGrid:GetChildren()) do
            if child:IsA("TextButton") then
                local childStroke = child:FindFirstChildOfClass("UIStroke")
                if childStroke then
                    childStroke.Thickness = child == btn and 2 or 0
                end
            end
        end
    end)
end

-- ==================== Painting Logic ====================
local mouse = plugin:GetMouse()
local paintingConnection = nil

-- Brush indicator (sphere ที่แสดงตำแหน่ง brush)
local brushIndicator = nil

local function createBrushIndicator()
    if brushIndicator then brushIndicator:Destroy() end
    
    brushIndicator = Instance.new("Part")
    brushIndicator.Name = "TerrainPainterBrush"
    brushIndicator.Shape = Enum.PartType.Ball
    brushIndicator.Size = Vector3.new(brushSize * 2, brushSize * 2, brushSize * 2)
    brushIndicator.Anchored = true
    brushIndicator.CanCollide = false
    brushIndicator.CanTouch = false
    brushIndicator.CastShadow = false
    brushIndicator.Color = Color3.fromRGB(255, 255, 0)
    brushIndicator.Material = Enum.Material.Neon
    brushIndicator.Transparency = 0.7
    brushIndicator.Parent = workspace
end

-- วาด terrain (Paint terrain at position)
local function paintTerrain(position)
    local terrain = workspace.Terrain
    
    -- FillBall วาดเป็นทรงกลม (FillBall paints a sphere)
    terrain:FillBall(
        position,
        brushSize,
        currentMaterial
    )
end

-- เริ่ม painting (Start painting mode)
local function startPainting()
    isActive = true
    paintButton:SetActive(true)
    
    createBrushIndicator()
    
    -- อัพเดต brush position ตามเมาส์
    paintingConnection = game:GetService("RunService").RenderStepped:Connect(function()
        local target = mouse.Target
        local hit = mouse.Hit
        
        if brushIndicator then
            brushIndicator.CFrame = CFrame.new(hit.Position)
        end
        
        -- วาดถ้ากด mouse
        if mouse:IsA("PluginMouse") then
            -- การตรวจสอบ click ทำในส่วน mouse events
        end
    end)
    
    -- Paint เมื่อกด mouse
    mouse.Button1Down:Connect(function()
        if isActive then
            paintTerrain(mouse.Hit.Position)
        end
    end)
    
    -- Paint ขณะ drag
    mouse.Button1Down:Connect(function()
        local dragConnection
        dragConnection = game:GetService("RunService").RenderStepped:Connect(function()
            if isActive then
                paintTerrain(mouse.Hit.Position)
            else
                dragConnection:Disconnect()
            end
        end)
        
        mouse.Button1Up:Connect(function()
            dragConnection:Disconnect()
        end)
    end)
end

-- หยุด painting (Stop painting mode)
local function stopPainting()
    isActive = false
    paintButton:SetActive(false)
    
    if paintingConnection then
        paintingConnection:Disconnect()
        paintingConnection = nil
    end
    
    if brushIndicator then
        brushIndicator:Destroy()
        brushIndicator = nil
    end
end

paintButton.Click:Connect(function()
    if isActive then
        stopPainting()
        widget.Enabled = false
    else
        widget.Enabled = true
        startPainting()
    end
end)

-- ทำความสะอาดเมื่อ unload plugin
plugin.Unloading:Connect(function()
    stopPainting()
end)
```

---

## ส่วนที่ 4: Plugin ตัวอย่าง - Instance Properties Inspector

### 4.1 Properties Inspector

```lua
-- Plugin: PropertiesInspector
-- Plugin ที่แสดงและแก้ไข properties ของ instances
-- Plugin to view and edit instance properties

local toolbar = plugin:CreateToolbar("Inspector")
local inspectButton = toolbar:CreateButton("Inspect", "Inspect selected instances", "rbxassetid://0")

local widgetInfo = DockWidgetPluginGuiInfo.new(
    Enum.InitialDockState.Right,
    false, false, 350, 600, 280, 400
)

local widget = plugin:CreateDockWidgetPluginGui("Inspector", widgetInfo)
widget.Title = "Instance Inspector"

-- ==================== UI ====================
local mainFrame = Instance.new("ScrollingFrame")
mainFrame.Size = UDim2.new(1, 0, 1, 0)
mainFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
mainFrame.BorderSizePixel = 0
mainFrame.ScrollBarThickness = 6
mainFrame.Parent = widget

local listLayout = Instance.new("UIListLayout")
listLayout.Padding = UDim.new(0, 1)
listLayout.Parent = mainFrame

-- สร้าง property row (Create property row)
local function createPropertyRow(propertyName, value, onEdit)
    local row = Instance.new("Frame")
    row.Size = UDim2.new(1, 0, 0, 30)
    row.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    row.BorderSizePixel = 0
    row.Parent = mainFrame
    
    local nameLabel = Instance.new("TextLabel")
    nameLabel.Size = UDim2.new(0.45, -2, 1, 0)
    nameLabel.Position = UDim2.new(0, 5, 0, 0)
    nameLabel.BackgroundTransparency = 1
    nameLabel.TextColor3 = Color3.fromRGB(170, 200, 255)
    nameLabel.Text = propertyName
    nameLabel.TextXAlignment = Enum.TextXAlignment.Left
    nameLabel.TextSize = 12
    nameLabel.Font = Enum.Font.Gotham
    nameLabel.ClipsDescendants = true
    nameLabel.Parent = row
    
    local valueType = typeof(value)
    
    if valueType == "string" or valueType == "number" or valueType == "boolean" then
        local valueBox = Instance.new("TextBox")
        valueBox.Size = UDim2.new(0.55, -5, 0.8, 0)
        valueBox.Position = UDim2.new(0.45, 2, 0.1, 0)
        valueBox.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
        valueBox.TextColor3 = Color3.fromRGB(220, 220, 220)
        valueBox.Text = tostring(value)
        valueBox.TextXAlignment = Enum.TextXAlignment.Left
        valueBox.TextSize = 12
        valueBox.Font = Enum.Font.Code
        valueBox.BorderSizePixel = 0
        valueBox.ClearTextOnFocus = false
        valueBox.Parent = row
        
        if onEdit then
            valueBox.FocusLost:Connect(function(enterPressed)
                if enterPressed then
                    onEdit(valueBox.Text)
                end
            end)
        end
    elseif valueType == "Vector3" then
        local valueLabel = Instance.new("TextLabel")
        valueLabel.Size = UDim2.new(0.55, -5, 0.8, 0)
        valueLabel.Position = UDim2.new(0.45, 2, 0.1, 0)
        valueLabel.BackgroundTransparency = 1
        valueLabel.TextColor3 = Color3.fromRGB(180, 240, 180)
        valueLabel.Text = string.format("(%.1f, %.1f, %.1f)", value.X, value.Y, value.Z)
        valueLabel.TextXAlignment = Enum.TextXAlignment.Left
        valueLabel.TextSize = 11
        valueLabel.Font = Enum.Font.Code
        valueLabel.Parent = row
    elseif valueType == "Color3" then
        local colorFrame = Instance.new("Frame")
        colorFrame.Size = UDim2.new(0, 50, 0.7, 0)
        colorFrame.Position = UDim2.new(0.45, 2, 0.15, 0)
        colorFrame.BackgroundColor3 = value
        colorFrame.BorderSizePixel = 1
        colorFrame.Parent = row
    else
        local valueLabel = Instance.new("TextLabel")
        valueLabel.Size = UDim2.new(0.55, -5, 0.8, 0)
        valueLabel.Position = UDim2.new(0.45, 2, 0.1, 0)
        valueLabel.BackgroundTransparency = 1
        valueLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
        valueLabel.Text = tostring(value)
        valueLabel.TextXAlignment = Enum.TextXAlignment.Left
        valueLabel.TextSize = 11
        valueLabel.Font = Enum.Font.Code
        valueLabel.ClipsDescendants = true
        valueLabel.Parent = row
    end
    
    return row
end

-- Properties ที่จะแสดง (Properties to display)
local COMMON_PROPERTIES = {
    "Name", "ClassName", "Archivable",
    -- BasePart properties
    "Position", "Size", "Color", "Material", "Transparency",
    "Anchored", "CanCollide", "CastShadow",
    -- Humanoid properties
    "WalkSpeed", "JumpHeight", "MaxHealth", "Health",
}

-- แสดง properties ของ instance (Display instance properties)
local function inspectInstance(instance)
    -- ล้าง rows เก่า (Clear old rows)
    for _, child in ipairs(mainFrame:GetChildren()) do
        if child:IsA("Frame") then
            child:Destroy()
        end
    end
    
    -- Header
    local header = Instance.new("Frame")
    header.Size = UDim2.new(1, 0, 0, 40)
    header.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    header.BorderSizePixel = 0
    header.Parent = mainFrame
    
    local headerLabel = Instance.new("TextLabel")
    headerLabel.Size = UDim2.new(1, -10, 1, 0)
    headerLabel.Position = UDim2.new(0, 10, 0, 0)
    headerLabel.BackgroundTransparency = 1
    headerLabel.TextColor3 = Color3.fromRGB(255, 200, 100)
    headerLabel.Text = instance.ClassName .. ": " .. instance.Name
    headerLabel.TextXAlignment = Enum.TextXAlignment.Left
    headerLabel.TextSize = 14
    headerLabel.Font = Enum.Font.GothamBold
    headerLabel.Parent = header
    
    -- แสดง properties (Display properties)
    for _, propName in ipairs(COMMON_PROPERTIES) do
        local success, value = pcall(function()
            return instance[propName]
        end)
        
        if success and value ~= nil then
            createPropertyRow(propName, value, function(newValue)
                -- แก้ไข property
                local parseSuccess = pcall(function()
                    local valueType = typeof(instance[propName])
                    if valueType == "number" then
                        instance[propName] = tonumber(newValue)
                    elseif valueType == "boolean" then
                        instance[propName] = newValue:lower() == "true"
                    else
                        instance[propName] = newValue
                    end
                end)
                
                if not parseSuccess then
                    warn("Failed to set property:", propName)
                end
            end)
        end
    end
    
    -- อัพเดท canvas size
    mainFrame.CanvasSize = UDim2.new(0, 0, 0, listLayout.AbsoluteContentSize.Y + 20)
end

-- ==================== Selection Handling ====================
local selectionService = game:GetService("Selection")

selectionService.SelectionChanged:Connect(function()
    local selected = selectionService:Get()
    if #selected > 0 then
        inspectInstance(selected[1])
    end
end)

inspectButton.Click:Connect(function()
    widget.Enabled = not widget.Enabled
    inspectButton:SetActive(widget.Enabled)
    
    if widget.Enabled then
        local selected = selectionService:Get()
        if #selected > 0 then
            inspectInstance(selected[1])
        end
    end
end)
```

---

## ส่วนที่ 5: Plugin ที่ใช้ Selection และ ChangeHistoryService

### 5.1 Instance Aligner

```lua
-- Plugin: InstanceAligner
-- Plugin จัด alignment ของ instances
-- Plugin for aligning instances

local toolbar = plugin:CreateToolbar("Alignment")
local ChangeHistoryService = game:GetService("ChangeHistoryService")
local Selection = game:GetService("Selection")

-- สร้างปุ่มต่างๆ (Create alignment buttons)
local alignXButton = toolbar:CreateButton("Align X", "Align selected on X axis", "rbxassetid://0")
local alignYButton = toolbar:CreateButton("Align Y", "Align selected on Y axis", "rbxassetid://0")
local alignZButton = toolbar:CreateButton("Align Z", "Align selected on Z axis", "rbxassetid://0")
local distributeButton = toolbar:CreateButton("Distribute", "Distribute selected evenly", "rbxassetid://0")

-- ==================== Alignment Functions ====================

-- หา BaseParts จาก selection (Get BaseParts from selection)
local function getSelectedParts()
    local parts = {}
    for _, instance in ipairs(Selection:Get()) do
        if instance:IsA("BasePart") then
            table.insert(parts, instance)
        elseif instance:IsA("Model") then
            local primaryPart = instance.PrimaryPart
            if primaryPart then
                table.insert(parts, primaryPart)
            end
        end
    end
    return parts
end

-- จัด align ตามแกน (Align on axis)
local function alignOnAxis(axis)
    local parts = getSelectedParts()
    if #parts < 2 then
        warn("Select at least 2 parts to align")
        return
    end
    
    -- ใช้ตำแหน่งของ part แรกเป็น reference
    local referencePos = parts[1].Position
    
    ChangeHistoryService:SetWaypoint("Before Align " .. axis)
    
    for _, part in ipairs(parts) do
        local newPos = part.Position
        
        if axis == "X" then
            newPos = Vector3.new(referencePos.X, newPos.Y, newPos.Z)
        elseif axis == "Y" then
            newPos = Vector3.new(newPos.X, referencePos.Y, newPos.Z)
        elseif axis == "Z" then
            newPos = Vector3.new(newPos.X, newPos.Y, referencePos.Z)
        end
        
        part.Position = newPos
    end
    
    ChangeHistoryService:SetWaypoint("After Align " .. axis)
end

-- กระจาย instances อย่างสม่ำเสมอ (Distribute instances evenly)
local function distributeEvenly()
    local parts = getSelectedParts()
    if #parts < 3 then
        warn("Select at least 3 parts to distribute")
        return
    end
    
    -- เรียงตามตำแหน่ง X (Sort by X position)
    table.sort(parts, function(a, b)
        return a.Position.X < b.Position.X
    end)
    
    local firstPos = parts[1].Position.X
    local lastPos = parts[#parts].Position.X
    local spacing = (lastPos - firstPos) / (#parts - 1)
    
    ChangeHistoryService:SetWaypoint("Before Distribute")
    
    for i, part in ipairs(parts) do
        local newX = firstPos + spacing * (i - 1)
        part.Position = Vector3.new(newX, part.Position.Y, part.Position.Z)
    end
    
    ChangeHistoryService:SetWaypoint("After Distribute")
end

-- ==================== Button Connections ====================
alignXButton.Click:Connect(function() alignOnAxis("X") end)
alignYButton.Click:Connect(function() alignOnAxis("Y") end)
alignZButton.Click:Connect(function() alignOnAxis("Z") end)
distributeButton.Click:Connect(function() distributeEvenly() end)
```

---

## ส่วนที่ 6: Publishing Plugin

### 6.1 ขั้นตอนการ Publish Plugin

```
การ Publish Plugin ไปยัง Roblox Marketplace:

1. เตรียม Plugin
   - ตรวจสอบว่า code ทำงานถูกต้อง
   - เขียน description ที่ชัดเจน
   - สร้าง thumbnail ขนาด 512x512
   - กำหนด version number (เช่น 1.0.0)

2. Test Plugin
   - ทดสอบใน Studio หลายๆ เวอร์ชัน
   - ทดสอบกับ places ต่างๆ
   - ตรวจสอบ memory leaks
   - ตรวจสอบ edge cases

3. Create Plugin package
   - ใน Studio: File > Publish as Plugin
   - หรือ right-click Plugin script > Publish as Plugin

4. กำหนดข้อมูล Plugin
   Plugin Name: ชื่อที่ชัดเจน
   Description: อธิบายฟีเจอร์ทั้งหมด
   Price: Free หรือ Robux
   Genre: ประเภทของ plugin

5. กำหนด Privacy
   - Public: ทุกคนใช้ได้
   - Private: เฉพาะตัวเอง
   - Group only: สำหรับสมาชิก group

6. Upload Thumbnail
   - ขนาด 512x512 pixels
   - Format: PNG
   - แสดงให้เห็นว่า plugin ทำอะไร

7. Submit สำหรับ Review
   - Roblox จะ review plugin
   - ใช้เวลา 1-3 วัน

การอัพเดท Plugin:
   1. แก้ไข Plugin script
   2. ทดสอบอีกครั้ง
   3. Right-click > Publish as Plugin
   4. เลือก overwrite version เดิม
   5. เพิ่ม changelog
```

### 6.2 Plugin Best Practices

```lua
-- Best practices สำหรับ Plugin development
-- Best practices for Plugin development

--[[
1. Cleanup เมื่อ unload (Always cleanup on unload)
   plugin.Unloading:Connect(function()
       -- ลบ connections
       -- ลบ instances ที่สร้างไว้
       -- บันทึก settings
   end)

2. ใช้ ChangeHistoryService เสมอ (Always use ChangeHistoryService)
   ChangeHistoryService:SetWaypoint("Before operation")
   -- ทำการเปลี่ยนแปลง
   ChangeHistoryService:SetWaypoint("After operation")
   
3. ตรวจสอบ Selection ก่อนทำงาน (Validate selection before operating)
   local selected = Selection:Get()
   if #selected == 0 then
       -- แจ้งผู้ใช้
       return
   end

4. ใช้ pcall สำหรับ property access (Use pcall for property access)
   local success, value = pcall(function()
       return instance.Position
   end)
   
5. Show progress สำหรับ operations ที่ใช้เวลา (Show progress for long operations)
   -- Update label ทุกๆ 100 instances ที่ process

6. Error handling ที่ดี (Good error handling)
   local function safeOperation()
       local ok, err = pcall(function()
           -- operation
       end)
       if not ok then
           warn("Plugin error:", err)
           -- แจ้งผู้ใช้ใน widget
       end
   end

7. Localization (Support multiple languages)
   local strings = {
       en = {confirm = "Confirm", cancel = "Cancel"},
       th = {confirm = "ยืนยัน", cancel = "ยกเลิก"},
   }
]]
```

---

## แบบฝึกหัด (Practice Exercises)

### แบบฝึกหัดที่ 1: Color Theme Applier
```
สร้าง Plugin ที่:
1. แสดงรายการ color schemes (Beach, Forest, Volcano, ฯลฯ)
2. เมื่อเลือก scheme, apply colors ไปยัง selected BaseParts
3. มีตัวเลือก "Apply to children" สำหรับ Models
4. รองรับ undo ด้วย ChangeHistoryService
5. บันทึก last used scheme ใน plugin settings
```

### แบบฝึกหัดที่ 2: Script Template Generator
```
สร้าง Plugin ที่:
1. มี template categories (Game Services, UI, DataStore, ฯลฯ)
2. เมื่อเลือก template, สร้าง Script ใน Selection
3. Template มี placeholder ที่ผู้ใช้กรอกข้อมูล (เช่น game name)
4. บันทึก custom templates ที่ผู้ใช้สร้าง
```

### แบบฝึกหัดที่ 3: Asset Validator
```
สร้าง Plugin ที่:
1. Scan ทุก instances ใน workspace
2. ตรวจสอบ naming conventions (PascalCase, camelCase)
3. ตรวจสอบ missing properties (เช่น Part ที่ไม่มี name)
4. แสดงรายการ issues พร้อม fix suggestions
5. มีปุ่ม "Fix All" ที่แก้ไข issues อัตโนมัติ
```

---

## สรุป (Summary)

ในบทนี้เราได้เรียนรู้:

1. **Plugin API** - toolbar, buttons, widgets, mouse, selection
2. **DockWidgetPluginGui** - การสร้าง custom window ใน Studio
3. **Mass Renamer** - Plugin จริงที่ใช้ find & replace
4. **Terrain Painter** - Plugin สำหรับวาด terrain materials
5. **Properties Inspector** - Plugin แสดงและแก้ไข instance properties
6. **Instance Aligner** - Plugin จัด position ของ instances
7. **ChangeHistoryService** - ทำให้ plugin operations undo ได้
8. **Plugin Publishing** - ขั้นตอนการ publish ไปยัง marketplace

### คำแนะนำสำหรับ Plugin Developer:
- เริ่มจาก Plugin เล็กๆ ที่แก้ปัญหาที่ตัวเองพบในการทำงาน
- ใช้งานเองก่อน แล้วค่อย publish
- รับ feedback จาก community และอัพเดทสม่ำเสมอ
- Plugin ที่ดีมักมีผู้ใช้หลักพันหลักหมื่นได้ฟรี!

*บทถัดไป: Part 99 - Building a Portfolio and Career in Roblox Development*
