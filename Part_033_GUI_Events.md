# ตอนที่ 33: GUI Event Handling

## บทนำ

GUI Events คือ events ที่เกิดขึ้นเมื่อผู้เล่นโต้ตอบกับ GUI เช่น คลิกปุ่ม, Hover, กด Keyboard หรือแม้แต่ Gamepad ในบทนี้เราจะเรียนรู้ทุก GUI Events ที่มีใน Roblox และวิธีการจัดการอย่างถูกต้อง

---

## 33.1 Mouse Events

```lua
-- LocalScript
local button = Instance.new("TextButton")
button.Size = UDim2.new(0, 200, 0, 50)
button.AnchorPoint = Vector2.new(0.5, 0.5)
button.Position = UDim2.new(0.5, 0, 0.5, 0)
button.Text = "ทดสอบ Events"
button.Parent = screenGui

-- Click Events
button.MouseButton1Click:Connect(function()
    print("คลิกซ้าย!")
end)

button.MouseButton2Click:Connect(function()
    print("คลิกขวา!")
end)

-- Mouse Press/Release
button.MouseButton1Down:Connect(function()
    print("กดซ้ายลง")
    button.BackgroundColor3 = Color3.fromRGB(50, 50, 200)  -- สีเข้มขณะกด
end)

button.MouseButton1Up:Connect(function()
    print("ปล่อยซ้าย")
    button.BackgroundColor3 = Color3.fromRGB(100, 100, 255)  -- สีปกติ
end)

button.MouseButton2Down:Connect(function()
    print("กดขวาลง")
end)

button.MouseButton2Up:Connect(function()
    print("ปล่อยขวา")
end)

-- Hover Events
button.MouseEnter:Connect(function()
    print("Mouse เข้า")
    button.BackgroundColor3 = Color3.fromRGB(150, 150, 255)
end)

button.MouseLeave:Connect(function()
    print("Mouse ออก")
    button.BackgroundColor3 = Color3.fromRGB(100, 100, 255)
end)

-- Mouse Move (ขณะ Hover)
button.MouseMoved:Connect(function(x, y)
    print("Mouse อยู่ที่:", x, y)
end)
```

---

## 33.2 Touch Events (Mobile)

```lua
-- LocalScript
local button = Instance.new("TextButton")
button.Size = UDim2.new(0, 200, 0, 50)
button.Parent = screenGui

-- Touch Events
button.TouchTap:Connect(function(touchPositions, gameProcessedEvent)
    print("แตะ!", #touchPositions, "นิ้ว")
end)

button.TouchLongPress:Connect(function(touchPositions, state, gameProcessedEvent)
    -- state: Begin, Change, End
    if state == Enum.UserInputState.Begin then
        print("กดค้าง!")
    end
end)

button.TouchPan:Connect(function(touchPositions, totalTranslation, velocity, state, gameProcessedEvent)
    -- เลื่อนนิ้ว
    print("เลื่อน:", totalTranslation.X, totalTranslation.Y)
end)

button.TouchPinch:Connect(function(touchPositions, scale, velocity, state, gameProcessedEvent)
    -- หยิก (Pinch to Zoom)
    print("Pinch Scale:", scale)
end)

button.TouchRotate:Connect(function(touchPositions, rotation, velocity, state, gameProcessedEvent)
    print("หมุน:", rotation)
end)

button.TouchSwipe:Connect(function(swipeDirection, numberOfTouches, gameProcessedEvent)
    -- Swipe Direction: Up, Down, Left, Right
    print("Swipe:", swipeDirection)
end)
```

---

## 33.3 Keyboard Events

```lua
-- LocalScript: ตรวจสอบ Keyboard ผ่าน GUI
local UserInputService = game:GetService("UserInputService")

-- Global Keyboard Input
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    -- gameProcessed = true ถ้า GUI กำลัง Focus (เช่น TextBox)
    if gameProcessed then return end
    
    if input.UserInputType == Enum.UserInputType.Keyboard then
        print("กดปุ่ม:", input.KeyCode)
        
        -- ปุ่มเฉพาะ
        if input.KeyCode == Enum.KeyCode.Escape then
            print("กด Escape")
        elseif input.KeyCode == Enum.KeyCode.Return or 
               input.KeyCode == Enum.KeyCode.KeypadEnter then
            print("กด Enter")
        end
    end
end)

-- ตรวจสอบว่ากดปุ่มอยู่หรือเปล่า
UserInputService.InputChanged:Connect(function(input, gameProcessed)
    if input.UserInputType == Enum.UserInputType.Keyboard then
        -- ปุ่มที่กดอยู่
    end
end)

-- ตรวจสอบ Key Hold
local RunService = game:GetService("RunService")
RunService.Heartbeat:Connect(function()
    if UserInputService:IsKeyDown(Enum.KeyCode.W) then
        -- กำลังกด W
    end
end)
```

---

## 33.4 Focus Events (TextBox)

```lua
-- LocalScript
local textBox = Instance.new("TextBox")
textBox.Size = UDim2.new(0, 300, 0, 50)
textBox.Parent = screenGui

-- เมื่อ Focus (คลิกเข้า TextBox)
textBox.Focused:Connect(function()
    print("TextBox ถูก Focus!")
    textBox.BackgroundColor3 = Color3.fromRGB(60, 60, 100)  -- ไฮไลต์
end)

-- เมื่อ FocusLost (คลิกออกจาก TextBox)
textBox.FocusLost:Connect(function(enterPressed)
    print("TextBox เสีย Focus!")
    textBox.BackgroundColor3 = Color3.fromRGB(40, 40, 60)  -- สีปกติ
    
    if enterPressed then
        print("กด Enter, ข้อความ:", textBox.Text)
    else
        print("คลิกออก")
    end
end)

-- เมื่อ Text เปลี่ยน
textBox:GetPropertyChangedSignal("Text"):Connect(function()
    print("Text เปลี่ยนเป็น:", textBox.Text)
end)

-- บังคับ Focus/Unfocus
textBox:CaptureFocus()   -- Focus
textBox:ReleaseFocus()   -- Unfocus
```

---

## 33.5 Selection Events (Gamepad/Console)

```lua
-- LocalScript: Gamepad Support
local GuiService = game:GetService("GuiService")

-- ตรวจสอบ Gamepad
local UserInputService = game:GetService("UserInputService")
if UserInputService.GamepadEnabled then
    print("มี Gamepad!")
end

-- กำหนด Navigation ด้วย Gamepad
local button1 = Instance.new("TextButton")
button1.SelectionImageObject = nil  -- Custom selection image

-- Selected Event
button1.SelectionGained:Connect(function()
    print("ปุ่มถูกเลือก (Gamepad)")
    button1.BackgroundColor3 = Color3.fromRGB(100, 150, 255)
end)

button1.SelectionLost:Connect(function()
    print("ปุ่มเลิกถูกเลือก")
    button1.BackgroundColor3 = Color3.fromRGB(70, 70, 100)
end)

-- กำหนด Next/Previous Selection
local button2 = Instance.new("TextButton")
button1.NextSelectionRight = button2
button2.NextSelectionLeft = button1
button1.NextSelectionDown = button2
button2.NextSelectionUp = button1

-- เริ่มต้นที่ปุ่ม
GuiService.SelectedObject = button1
```

---

## 33.6 Event System ครบถ้วน

```lua
-- LocalScript: Complete Event System
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

local LocalPlayer = Players.LocalPlayer

local screenGui = Instance.new("ScreenGui")
screenGui.Parent = LocalPlayer.PlayerGui

-- สร้าง Event Log
local logFrame = Instance.new("Frame")
logFrame.Size = UDim2.new(0, 400, 0, 300)
logFrame.Position = UDim2.new(1, -410, 0, 10)
logFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
logFrame.BorderSizePixel = 0
logFrame.Parent = screenGui

local logCorner = Instance.new("UICorner")
logCorner.CornerRadius = UDim.new(0, 8)
logCorner.Parent = logFrame

local logTitle = Instance.new("TextLabel")
logTitle.Size = UDim2.new(1, 0, 0, 30)
logTitle.BackgroundColor3 = Color3.fromRGB(40, 40, 60)
logTitle.BorderSizePixel = 0
logTitle.Text = "📋 Event Log"
logTitle.TextColor3 = Color3.new(1, 1, 1)
logTitle.Font = Enum.Font.GothamBold
logTitle.TextSize = 14
logTitle.Parent = logFrame

local logScroll = Instance.new("ScrollingFrame")
logScroll.Size = UDim2.new(1, -10, 1, -40)
logScroll.Position = UDim2.new(0, 5, 0, 35)
logScroll.BackgroundTransparency = 1
logScroll.ScrollBarThickness = 3
logScroll.AutomaticCanvasSize = Enum.AutomaticSize.Y
logScroll.CanvasSize = UDim2.new(0, 0, 0, 0)
logScroll.Parent = logFrame

local logList = Instance.new("UIListLayout")
logList.Padding = UDim.new(0, 2)
logList.SortOrder = Enum.SortOrder.LayoutOrder
logList.Parent = logScroll

local logCount = 0

local function addLog(event, color)
    logCount = logCount + 1
    
    local entry = Instance.new("TextLabel")
    entry.Size = UDim2.new(1, 0, 0, 20)
    entry.BackgroundTransparency = 0.9
    entry.BackgroundColor3 = color or Color3.fromRGB(50, 50, 70)
    entry.BorderSizePixel = 0
    entry.Text = "  " .. os.date("%H:%M:%S") .. " - " .. event
    entry.TextColor3 = color or Color3.fromRGB(220, 220, 220)
    entry.Font = Enum.Font.Code
    entry.TextSize = 11
    entry.TextXAlignment = Enum.TextXAlignment.Left
    entry.LayoutOrder = logCount
    entry.Parent = logScroll
    
    -- Scroll to bottom
    task.wait()
    logScroll.CanvasPosition = Vector2.new(0, logScroll.AbsoluteCanvasSize.Y)
    
    -- ลบ Log เก่าถ้ามีมากกว่า 50
    if logCount > 50 then
        logScroll:FindFirstChild("TextLabel"):Destroy()
    end
end

-- Interactive Area
local interactArea = Instance.new("Frame")
interactArea.Size = UDim2.new(0, 300, 0, 200)
interactArea.Position = UDim2.new(0, 10, 0.5, -100)
interactArea.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
interactArea.BorderSizePixel = 0
interactArea.Active = true
interactArea.Parent = screenGui

local areaCorner = Instance.new("UICorner")
areaCorner.CornerRadius = UDim.new(0, 10)
areaCorner.Parent = interactArea

local areaTitle = Instance.new("TextLabel")
areaTitle.Size = UDim2.new(1, 0, 0, 30)
areaTitle.BackgroundTransparency = 1
areaTitle.Text = "โต้ตอบที่นี่!"
areaTitle.TextColor3 = Color3.new(1, 1, 1)
areaTitle.Font = Enum.Font.GothamBold
areaTitle.TextSize = 16
areaTitle.Parent = interactArea

-- ปุ่มทดสอบ
local testBtn = Instance.new("TextButton")
testBtn.Size = UDim2.new(0, 150, 0, 40)
testBtn.AnchorPoint = Vector2.new(0.5, 0.5)
testBtn.Position = UDim2.new(0.5, 0, 0.5, 0)
testBtn.BackgroundColor3 = Color3.fromRGB(80, 130, 220)
testBtn.BorderSizePixel = 0
testBtn.Text = "คลิกฉัน"
testBtn.TextColor3 = Color3.new(1, 1, 1)
testBtn.Font = Enum.Font.GothamBold
testBtn.TextSize = 15
testBtn.Parent = interactArea

local btnCorner = Instance.new("UICorner")
btnCorner.CornerRadius = UDim.new(0, 8)
btnCorner.Parent = testBtn

-- ผูก Events
testBtn.MouseButton1Click:Connect(function()
    addLog("MouseButton1Click", Color3.fromRGB(100, 200, 100))
end)

testBtn.MouseButton2Click:Connect(function()
    addLog("MouseButton2Click", Color3.fromRGB(200, 100, 100))
end)

testBtn.MouseButton1Down:Connect(function()
    addLog("MouseButton1Down", Color3.fromRGB(50, 150, 250))
end)

testBtn.MouseButton1Up:Connect(function()
    addLog("MouseButton1Up", Color3.fromRGB(50, 200, 250))
end)

testBtn.MouseEnter:Connect(function()
    addLog("MouseEnter", Color3.fromRGB(200, 200, 100))
end)

testBtn.MouseLeave:Connect(function()
    addLog("MouseLeave", Color3.fromRGB(150, 150, 70))
end)

testBtn.MouseMoved:Connect(function(x, y)
    -- ไม่ log เพราะมันจะเยอะมาก
end)
```

---

## 33.7 Event Manager Pattern

```lua
-- ModuleScript: EventManager
local EventManager = {}

local connections = {}

-- Subscribe to Event
function EventManager.subscribe(id, event, callback)
    if not connections[id] then
        connections[id] = {}
    end
    
    local connection = event:Connect(callback)
    table.insert(connections[id], connection)
    
    return connection
end

-- Unsubscribe All Events for ID
function EventManager.unsubscribeAll(id)
    if connections[id] then
        for _, conn in ipairs(connections[id]) do
            if conn.Connected then
                conn:Disconnect()
            end
        end
        connections[id] = nil
    end
end

-- Clear All
function EventManager.clearAll()
    for id in pairs(connections) do
        EventManager.unsubscribeAll(id)
    end
end

return EventManager
```

### การใช้งาน EventManager

```lua
-- LocalScript
local EventManager = require(game.ReplicatedStorage.EventManager)

local button = Instance.new("TextButton")
button.Parent = screenGui

-- Subscribe
EventManager.subscribe("mainButton", button.MouseButton1Click, function()
    print("Click!")
end)

EventManager.subscribe("mainButton", button.MouseEnter, function()
    print("Hover!")
end)

-- Cleanup เมื่อไม่ต้องการแล้ว
-- EventManager.unsubscribeAll("mainButton")
```

---

## 33.8 Debounce Pattern

```lua
-- LocalScript: Debounce สำหรับ GUI Events
local TweenService = game:GetService("TweenService")

-- ฟังก์ชัน Debounce พื้นฐาน
local function createDebounce(cooldown)
    local lastClick = 0
    return function()
        local now = tick()
        if now - lastClick >= cooldown then
            lastClick = now
            return true
        end
        return false
    end
end

-- ใช้งาน
local button = Instance.new("TextButton")
button.Parent = screenGui

local canClick = createDebounce(1)  -- 1 วินาที Cooldown

button.MouseButton1Click:Connect(function()
    if canClick() then
        print("คลิกสำเร็จ!")
        -- ทำอะไรบางอย่าง
    else
        print("รออีกนิด...")
    end
end)

-- Visual Cooldown Button
local function createCooldownButton(parent, text, cooldown, action)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 160, 0, 50)
    btn.BackgroundColor3 = Color3.fromRGB(80, 130, 220)
    btn.BorderSizePixel = 0
    btn.Text = text
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 15
    btn.Parent = parent
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 8)
    corner.Parent = btn
    
    -- Cooldown Overlay
    local overlay = Instance.new("Frame")
    overlay.Size = UDim2.new(1, 0, 1, 0)
    overlay.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    overlay.BackgroundTransparency = 1
    overlay.BorderSizePixel = 0
    overlay.ClipsDescendants = true
    overlay.Parent = btn
    
    local overlayCorner = Instance.new("UICorner")
    overlayCorner.CornerRadius = UDim.new(0, 8)
    overlayCorner.Parent = overlay
    
    -- Cooldown Fill (ลดลงตามเวลา)
    local fill = Instance.new("Frame")
    fill.Size = UDim2.new(1, 0, 1, 0)
    fill.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    fill.BackgroundTransparency = 0.5
    fill.BorderSizePixel = 0
    fill.Parent = overlay
    
    -- Cooldown Text
    local cdText = Instance.new("TextLabel")
    cdText.Size = UDim2.new(1, 0, 1, 0)
    cdText.BackgroundTransparency = 1
    cdText.Text = ""
    cdText.TextColor3 = Color3.new(1, 1, 1)
    cdText.Font = Enum.Font.GothamBold
    cdText.TextSize = 18
    cdText.Parent = btn
    
    local onCooldown = false
    
    btn.MouseButton1Click:Connect(function()
        if onCooldown then return end
        onCooldown = true
        
        -- Execute action
        action()
        
        -- Visual Cooldown
        overlay.BackgroundTransparency = 0.5
        cdText.Text = tostring(cooldown)
        
        local remaining = cooldown
        local startTime = tick()
        
        local RunService = game:GetService("RunService")
        local conn
        conn = RunService.Heartbeat:Connect(function()
            remaining = cooldown - (tick() - startTime)
            
            if remaining <= 0 then
                conn:Disconnect()
                onCooldown = false
                overlay.BackgroundTransparency = 1
                cdText.Text = ""
                return
            end
            
            cdText.Text = string.format("%.1f", remaining)
            fill.Size = UDim2.new(1, 0, remaining/cooldown, 0)
            fill.Position = UDim2.new(0, 0, 1 - remaining/cooldown, 0)
        end)
    end)
    
    return btn
end

-- ใช้งาน
local btn = createCooldownButton(screenGui, "โจมตี!", 3, function()
    print("โจมตี!")
end)
```

---

## 33.9 Context Menu

```lua
-- LocalScript: Context Menu (Right Click)
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

local LocalPlayer = Players.LocalPlayer

local screenGui = Instance.new("ScreenGui")
screenGui.DisplayOrder = 999
screenGui.Parent = LocalPlayer.PlayerGui

local activeMenu = nil

local function closeContextMenu()
    if activeMenu then
        TweenService:Create(activeMenu, TweenInfo.new(0.15), {
            Size = UDim2.new(0, activeMenu.Size.X.Offset, 0, 0),
            BackgroundTransparency = 1
        }):Play()
        task.delay(0.15, function()
            if activeMenu then
                activeMenu:Destroy()
                activeMenu = nil
            end
        end)
    end
end

local function showContextMenu(position, options)
    closeContextMenu()
    
    local menu = Instance.new("Frame")
    menu.Size = UDim2.new(0, 180, 0, 0)
    menu.Position = UDim2.new(0, position.X, 0, position.Y)
    menu.BackgroundColor3 = Color3.fromRGB(35, 35, 50)
    menu.BorderSizePixel = 0
    menu.ClipsDescendants = true
    menu.ZIndex = 100
    menu.Parent = screenGui
    
    local menuCorner = Instance.new("UICorner")
    menuCorner.CornerRadius = UDim.new(0, 8)
    menuCorner.Parent = menu
    
    local menuStroke = Instance.new("UIStroke")
    menuStroke.Color = Color3.fromRGB(80, 80, 110)
    menuStroke.Thickness = 1
    menuStroke.Parent = menu
    
    local menuList = Instance.new("UIListLayout")
    menuList.Padding = UDim.new(0, 2)
    menuList.Parent = menu
    
    local menuPad = Instance.new("UIPadding")
    menuPad.PaddingTop = UDim.new(0, 5)
    menuPad.PaddingBottom = UDim.new(0, 5)
    menuPad.Parent = menu
    
    local totalHeight = 10  -- padding
    
    for _, option in ipairs(options) do
        if option.separator then
            -- Divider
            local sep = Instance.new("Frame")
            sep.Size = UDim2.new(1, -20, 0, 1)
            sep.BackgroundColor3 = Color3.fromRGB(70, 70, 100)
            sep.BorderSizePixel = 0
            sep.Parent = menu
            totalHeight = totalHeight + 5
        else
            local item = Instance.new("TextButton")
            item.Size = UDim2.new(1, 0, 0, 32)
            item.BackgroundTransparency = 1
            item.Text = "  " .. (option.icon or "") .. "  " .. option.text
            item.TextColor3 = option.color or Color3.fromRGB(220, 220, 230)
            item.Font = Enum.Font.Gotham
            item.TextSize = 13
            item.TextXAlignment = Enum.TextXAlignment.Left
            item.ZIndex = 100
            item.Parent = menu
            
            item.MouseEnter:Connect(function()
                item.BackgroundTransparency = 0.7
                item.BackgroundColor3 = Color3.fromRGB(70, 70, 100)
            end)
            
            item.MouseLeave:Connect(function()
                item.BackgroundTransparency = 1
            end)
            
            if option.action then
                item.MouseButton1Click:Connect(function()
                    closeContextMenu()
                    option.action()
                end)
            end
            
            totalHeight = totalHeight + 34
        end
    end
    
    -- Animate Open
    TweenService:Create(menu, TweenInfo.new(0.2, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
        Size = UDim2.new(0, 180, 0, totalHeight)
    }):Play()
    
    activeMenu = menu
    return menu
end

-- ปิด Menu เมื่อคลิกที่อื่น
UserInputService.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        closeContextMenu()
    end
end)

-- ตัวอย่างการใช้งาน (Right Click บนพื้นที่)
local testArea = Instance.new("Frame")
testArea.Size = UDim2.new(0, 400, 0, 300)
testArea.AnchorPoint = Vector2.new(0.5, 0.5)
testArea.Position = UDim2.new(0.5, 0, 0.5, 0)
testArea.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
testArea.BorderSizePixel = 0
testArea.Active = true
testArea.Parent = screenGui

testArea.MouseButton2Click:Connect(function()
    local mousePos = UserInputService:GetMouseLocation()
    showContextMenu(mousePos, {
        {icon = "✏️", text = "แก้ไข", action = function() print("แก้ไข") end},
        {icon = "📋", text = "คัดลอก", action = function() print("คัดลอก") end},
        {icon = "📌", text = "ปักหมุด", action = function() print("ปักหมุด") end},
        {separator = true},
        {icon = "🗑️", text = "ลบ", color = Color3.fromRGB(255, 100, 100), 
         action = function() print("ลบ") end},
    })
end)
```

---

## 33.10 Drag and Drop

```lua
-- LocalScript: Drag and Drop
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")

local LocalPlayer = Players.LocalPlayer

local screenGui = Instance.new("ScreenGui")
screenGui.Parent = LocalPlayer.PlayerGui

-- ฟังก์ชันทำให้ Element ลากได้
local function makeDraggable(element, handle)
    handle = handle or element
    
    local isDragging = false
    local dragStart = nil
    local startPos = nil
    local dragClone = nil
    
    handle.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or
           input.UserInputType == Enum.UserInputType.Touch then
            isDragging = true
            dragStart = input.Position
            startPos = element.Position
        end
    end)
    
    UserInputService.InputChanged:Connect(function(input)
        if not isDragging then return end
        
        if input.UserInputType == Enum.UserInputType.MouseMovement or
           input.UserInputType == Enum.UserInputType.Touch then
            local delta = input.Position - dragStart
            element.Position = UDim2.new(
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
end

-- Inventory Drag and Drop
local function createInventorySystem()
    local inventorySlots = {}
    local draggingItem = nil
    local dragOffset = Vector2.new()
    
    -- สร้าง Grid
    local invFrame = Instance.new("Frame")
    invFrame.Size = UDim2.new(0, 300, 0, 300)
    invFrame.Position = UDim2.new(0.5, -150, 0.5, -150)
    invFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
    invFrame.BorderSizePixel = 0
    invFrame.Parent = screenGui
    
    local invCorner = Instance.new("UICorner")
    invCorner.CornerRadius = UDim.new(0, 10)
    invCorner.Parent = invFrame
    
    local grid = Instance.new("UIGridLayout")
    grid.CellSize = UDim2.new(0, 60, 0, 60)
    grid.CellPadding = UDim2.new(0, 5, 0, 5)
    grid.Parent = invFrame
    
    local gridPad = Instance.new("UIPadding")
    gridPad.PaddingTop = UDim.new(0, 10)
    gridPad.PaddingLeft = UDim.new(0, 10)
    gridPad.Parent = invFrame
    
    -- สร้าง Slots
    for i = 1, 16 do
        local slot = Instance.new("Frame")
        slot.BackgroundColor3 = Color3.fromRGB(45, 45, 60)
        slot.BorderSizePixel = 0
        slot.Parent = invFrame
        
        local slotCorner = Instance.new("UICorner")
        slotCorner.CornerRadius = UDim.new(0, 6)
        slotCorner.Parent = slot
        
        inventorySlots[i] = slot
        
        -- Drop Zone
        slot.InputBegan:Connect(function(input)
            if draggingItem and input.UserInputType == Enum.UserInputType.MouseButton1 then
                -- ย้าย Item ไปยัง Slot นี้
                draggingItem.Parent = slot
                draggingItem.Position = UDim2.new(0, 5, 0, 5)
                draggingItem.Size = UDim2.new(1, -10, 1, -10)
            end
        end)
    end
    
    -- Item Function
    local function createItem(slotIndex, itemData)
        local slot = inventorySlots[slotIndex]
        if not slot then return end
        
        local item = Instance.new("TextButton")
        item.Size = UDim2.new(1, -10, 1, -10)
        item.Position = UDim2.new(0, 5, 0, 5)
        item.BackgroundColor3 = itemData.color or Color3.fromRGB(100, 100, 150)
        item.BorderSizePixel = 0
        item.Text = itemData.icon or "?"
        item.TextSize = 24
        item.ZIndex = 2
        item.Parent = slot
        
        local itemCorner = Instance.new("UICorner")
        itemCorner.CornerRadius = UDim.new(0, 4)
        itemCorner.Parent = item
        
        -- Drag Logic
        item.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 then
                draggingItem = item
                item.ZIndex = 100
                
                local mousePos = UserInputService:GetMouseLocation()
                local itemAbsPos = item.AbsolutePosition
                dragOffset = Vector2.new(
                    mousePos.X - itemAbsPos.X,
                    mousePos.Y - itemAbsPos.Y
                )
            end
        end)
        
        return item
    end
    
    -- อัพเดตตำแหน่งขณะ Drag
    UserInputService.InputChanged:Connect(function(input)
        if draggingItem and input.UserInputType == Enum.UserInputType.MouseMovement then
            local mousePos = UserInputService:GetMouseLocation()
            draggingItem.Position = UDim2.new(
                0, mousePos.X - dragOffset.X,
                0, mousePos.Y - dragOffset.Y
            )
        end
    end)
    
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 and draggingItem then
            draggingItem.ZIndex = 2
            draggingItem = nil
        end
    end)
    
    -- ใส่ Items ตัวอย่าง
    createItem(1, {icon = "⚔️", color = Color3.fromRGB(200, 150, 50)})
    createItem(2, {icon = "🛡️", color = Color3.fromRGB(100, 150, 200)})
    createItem(5, {icon = "🧪", color = Color3.fromRGB(100, 200, 150)})
    createItem(8, {icon = "🗝️", color = Color3.fromRGB(220, 200, 100)})
end

createInventorySystem()
```

---

## 33.11 แบบฝึกหัด

### แบบฝึกหัดที่ 1: Hotkey System

```lua
-- LocalScript: Hotkey System
local UserInputService = game:GetService("UserInputService")
local Players = game:GetService("Players")

local LocalPlayer = Players.LocalPlayer

local screenGui = Instance.new("ScreenGui")
screenGui.Parent = LocalPlayer.PlayerGui

-- Hotkey Registry
local hotkeys = {}

local function registerHotkey(key, action, description)
    hotkeys[key] = {action = action, description = description}
end

local function unregisterHotkey(key)
    hotkeys[key] = nil
end

-- ตรวจสอบ Hotkeys
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    local keyCode = input.KeyCode
    if hotkeys[keyCode] then
        hotkeys[keyCode].action()
    end
end)

-- Hotkey Help Panel
local helpPanel = Instance.new("Frame")
helpPanel.Size = UDim2.new(0, 250, 0, 0)
helpPanel.Position = UDim2.new(1, -260, 1, -10)
helpPanel.AnchorPoint = Vector2.new(0, 1)
helpPanel.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
helpPanel.BackgroundTransparency = 0.2
helpPanel.BorderSizePixel = 0
helpPanel.AutomaticSize = Enum.AutomaticSize.Y
helpPanel.Parent = screenGui

local helpCorner = Instance.new("UICorner")
helpCorner.CornerRadius = UDim.new(0, 8)
helpCorner.Parent = helpPanel

local helpList = Instance.new("UIListLayout")
helpList.Padding = UDim.new(0, 2)
helpList.Parent = helpPanel

local helpPad = Instance.new("UIPadding")
helpPad.PaddingTop = UDim.new(0, 5)
helpPad.PaddingBottom = UDim.new(0, 5)
helpPad.PaddingLeft = UDim.new(0, 8)
helpPad.PaddingRight = UDim.new(0, 8)
helpPad.Parent = helpPanel

local function addHotkeyDisplay(key, description)
    local row = Instance.new("Frame")
    row.Size = UDim2.new(1, 0, 0, 22)
    row.BackgroundTransparency = 1
    row.Parent = helpPanel
    
    local keyLabel = Instance.new("TextLabel")
    keyLabel.Size = UDim2.new(0, 70, 1, 0)
    keyLabel.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
    keyLabel.BorderSizePixel = 0
    keyLabel.Text = "[" .. tostring(key):gsub("Enum.KeyCode.", "") .. "]"
    keyLabel.TextColor3 = Color3.fromRGB(200, 200, 100)
    keyLabel.Font = Enum.Font.Code
    keyLabel.TextSize = 11
    keyLabel.Parent = row
    
    local keyCorner = Instance.new("UICorner")
    keyCorner.CornerRadius = UDim.new(0, 4)
    keyCorner.Parent = keyLabel
    
    local descLabel = Instance.new("TextLabel")
    descLabel.Size = UDim2.new(1, -75, 1, 0)
    descLabel.Position = UDim2.new(0, 75, 0, 0)
    descLabel.BackgroundTransparency = 1
    descLabel.Text = description
    descLabel.TextColor3 = Color3.fromRGB(180, 180, 200)
    descLabel.Font = Enum.Font.Gotham
    descLabel.TextSize = 11
    descLabel.TextXAlignment = Enum.TextXAlignment.Left
    descLabel.Parent = row
end

-- ลงทะเบียน Hotkeys
registerHotkey(Enum.KeyCode.M, function()
    print("เปิด Map!")
end, "เปิด Map")

registerHotkey(Enum.KeyCode.I, function()
    print("เปิด Inventory!")
end, "เปิด Inventory")

registerHotkey(Enum.KeyCode.P, function()
    print("เปิด Party!")
end, "เปิด Party")

registerHotkey(Enum.KeyCode.H, function()
    print("เปิด Help!")
end, "เปิด Help")

-- แสดง Hotkeys ใน Panel
addHotkeyDisplay(Enum.KeyCode.M, "เปิด Map")
addHotkeyDisplay(Enum.KeyCode.I, "เปิด Inventory")
addHotkeyDisplay(Enum.KeyCode.P, "เปิด Party")
addHotkeyDisplay(Enum.KeyCode.H, "เปิด Help")

print("Hotkey System พร้อมแล้ว!")
```

---

## 33.12 สรุป

ในบทนี้เราได้เรียนรู้:

1. **Mouse Events** - Click, Down, Up, Enter, Leave, Moved
2. **Touch Events** - Tap, LongPress, Pan, Pinch, Rotate, Swipe
3. **Keyboard Events** - InputBegan, InputEnded, IsKeyDown
4. **Focus Events** - Focused, FocusLost (TextBox)
5. **Selection Events** - Gamepad Navigation
6. **Event Manager** - Pattern สำหรับจัดการ Events
7. **Debounce** - ป้องกัน Spam Click
8. **Context Menu** - Right Click Menu
9. **Drag and Drop** - ลากและวางได้
10. **Hotkey System** - ระบบปุ่มลัด

ในบทต่อไป เราจะเรียนรู้เกี่ยวกับ TweenService สำหรับทำ Animation

---

## แหล่งอ้างอิง

- [Roblox Developer Hub - GuiObject Events](https://developer.roblox.com/en-us/api-reference/class/GuiObject)
- [Roblox Developer Hub - UserInputService](https://developer.roblox.com/en-us/api-reference/class/UserInputService)
- [Roblox Developer Hub - TouchEnabled](https://developer.roblox.com/en-us/api-reference/property/UserInputService/TouchEnabled)
