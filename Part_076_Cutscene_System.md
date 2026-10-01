# Part 76: Cutscene System (ระบบ Cutscene)

## บทนำ

Cutscene ช่วยเล่าเรื่องราวและสร้างความประทับใจในเกม ในบทนี้เราจะสร้างระบบ Cutscene ที่สมบูรณ์ รวมถึง Camera Animation, Dialogue System, Letterbox Effect และการควบคุม Player ระหว่าง Cutscene

---

## 1. โครงสร้างระบบ Cutscene

### 1.1 Cutscene Manager

```lua
-- CutsceneManager.lua (Module Script ใน ReplicatedStorage)
-- ระบบจัดการ Cutscene ทั้งหมด

local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")

local CutsceneManager = {}
CutsceneManager.__index = CutsceneManager

-- ประเภทของ Camera Actions
local CameraActionTypes = {
    MOVE = "move",          -- เคลื่อนกล้องไปยัง CFrame
    LOOK_AT = "lookAt",     -- หันกล้องไปดู Target
    FOLLOW = "follow",      -- ติดตาม Object
    SHAKE = "shake",        -- สั่นกล้อง
    ZOOM = "zoom",          -- เปลี่ยน FOV
    WAIT = "wait",          -- รอ
}

function CutsceneManager.new()
    local self = setmetatable({}, CutsceneManager)
    
    self.player = Players.LocalPlayer
    self.camera = workspace.CurrentCamera
    self.playerGui = self.player:WaitForChild("PlayerGui")
    
    self.isPlaying = false
    self.skipRequested = false
    self.currentCutscene = nil
    
    -- สร้าง UI Elements
    self:_createUI()
    
    return self
end

-- สร้าง UI สำหรับ Cutscene
function CutsceneManager:_createUI()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "CutsceneUI"
    screenGui.IgnoreGuiInset = true
    screenGui.ResetOnSpawn = false
    screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    screenGui.Enabled = false
    screenGui.Parent = self.playerGui
    
    -- Letterbox Top
    local letterboxTop = Instance.new("Frame")
    letterboxTop.Name = "LetterboxTop"
    letterboxTop.Size = UDim2.new(1, 0, 0, 0)  -- เริ่มจาก 0 height
    letterboxTop.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    letterboxTop.BorderSizePixel = 0
    letterboxTop.Parent = screenGui
    
    -- Letterbox Bottom
    local letterboxBottom = Instance.new("Frame")
    letterboxBottom.Name = "LetterboxBottom"
    letterboxBottom.Size = UDim2.new(1, 0, 0, 0)
    letterboxBottom.Position = UDim2.new(0, 0, 1, 0)
    letterboxBottom.AnchorPoint = Vector2.new(0, 1)
    letterboxBottom.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    letterboxBottom.BorderSizePixel = 0
    letterboxBottom.Parent = screenGui
    
    -- Dialogue Box
    local dialogueBox = Instance.new("Frame")
    dialogueBox.Name = "DialogueBox"
    dialogueBox.Size = UDim2.new(0.8, 0, 0, 120)
    dialogueBox.Position = UDim2.new(0.1, 0, 0.75, 0)
    dialogueBox.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    dialogueBox.BackgroundTransparency = 0.3
    dialogueBox.BorderSizePixel = 0
    dialogueBox.Visible = false
    dialogueBox.Parent = screenGui
    
    local dialogueCorner = Instance.new("UICorner")
    dialogueCorner.CornerRadius = UDim.new(0, 8)
    dialogueCorner.Parent = dialogueBox
    
    -- Speaker Name
    local speakerLabel = Instance.new("TextLabel")
    speakerLabel.Name = "SpeakerName"
    speakerLabel.Size = UDim2.new(0, 200, 0, 30)
    speakerLabel.Position = UDim2.new(0, 15, 0, -30)
    speakerLabel.BackgroundColor3 = Color3.fromRGB(30, 80, 160)
    speakerLabel.Text = ""
    speakerLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    speakerLabel.TextSize = 14
    speakerLabel.Font = Enum.Font.GothamBold
    speakerLabel.BorderSizePixel = 0
    speakerLabel.Parent = dialogueBox
    
    local speakerCorner = Instance.new("UICorner")
    speakerCorner.CornerRadius = UDim.new(0, 6)
    speakerCorner.Parent = speakerLabel
    
    -- Dialogue Text
    local dialogueText = Instance.new("TextLabel")
    dialogueText.Name = "DialogueText"
    dialogueText.Size = UDim2.new(1, -30, 1, -20)
    dialogueText.Position = UDim2.new(0, 15, 0, 10)
    dialogueText.BackgroundTransparency = 1
    dialogueText.Text = ""
    dialogueText.TextColor3 = Color3.fromRGB(255, 255, 255)
    dialogueText.TextSize = 15
    dialogueText.Font = Enum.Font.Gotham
    dialogueText.TextXAlignment = Enum.TextXAlignment.Left
    dialogueText.TextYAlignment = Enum.TextYAlignment.Top
    dialogueText.TextWrapped = true
    dialogueText.Parent = dialogueBox
    
    -- Continue Arrow
    local continueArrow = Instance.new("TextLabel")
    continueArrow.Name = "ContinueArrow"
    continueArrow.Size = UDim2.new(0, 30, 0, 20)
    continueArrow.Position = UDim2.new(1, -35, 1, -25)
    continueArrow.BackgroundTransparency = 1
    continueArrow.Text = "▼"
    continueArrow.TextColor3 = Color3.fromRGB(200, 200, 200)
    continueArrow.TextSize = 14
    continueArrow.Visible = false
    continueArrow.Parent = dialogueBox
    
    -- Skip Button
    local skipButton = Instance.new("TextButton")
    skipButton.Name = "SkipButton"
    skipButton.Size = UDim2.new(0, 100, 0, 35)
    skipButton.Position = UDim2.new(1, -110, 0, 10)
    skipButton.BackgroundColor3 = Color3.fromRGB(40, 40, 60)
    skipButton.BackgroundTransparency = 0.3
    skipButton.Text = "⏭️ ข้าม (ESC)"
    skipButton.TextColor3 = Color3.fromRGB(200, 200, 200)
    skipButton.TextSize = 12
    skipButton.Font = Enum.Font.Gotham
    skipButton.BorderSizePixel = 0
    skipButton.Visible = false
    skipButton.Parent = screenGui
    
    local skipCorner = Instance.new("UICorner")
    skipCorner.CornerRadius = UDim.new(0, 6)
    skipCorner.Parent = skipButton
    
    skipButton.Activated:Connect(function()
        self:skip()
    end)
    
    -- เก็บ References
    self.ui = {
        screenGui = screenGui,
        letterboxTop = letterboxTop,
        letterboxBottom = letterboxBottom,
        dialogueBox = dialogueBox,
        speakerLabel = speakerLabel,
        dialogueText = dialogueText,
        continueArrow = continueArrow,
        skipButton = skipButton,
    }
end

-- เริ่ม Cutscene
function CutsceneManager:play(cutscene, options)
    if self.isPlaying then
        warn("Cutscene กำลังเล่นอยู่!")
        return
    end
    
    options = options or {}
    self.isPlaying = true
    self.skipRequested = false
    self.currentCutscene = cutscene
    
    -- ล็อค Player
    self:_lockPlayer(true)
    
    -- เปิด UI
    self.ui.screenGui.Enabled = true
    if options.skippable ~= false then
        self.ui.skipButton.Visible = true
    end
    
    -- เปลี่ยน Camera Mode
    local originalCameraType = self.camera.CameraType
    self.camera.CameraType = Enum.CameraType.Scriptable
    
    -- Letterbox Effect
    return task.spawn(function()
        -- แสดง Letterbox
        if options.letterbox ~= false then
            self:_showLetterbox()
        end
        
        -- รัน Cutscene Actions
        local success, err = pcall(function()
            cutscene:run(self)
        end)
        
        if not success and not self.skipRequested then
            warn("Cutscene error:", err)
        end
        
        -- ซ่อน Letterbox
        if options.letterbox ~= false then
            self:_hideLetterbox()
        end
        
        -- คืนค่า Camera
        self.camera.CameraType = originalCameraType
        
        -- ปลดล็อค Player
        self:_lockPlayer(false)
        
        -- ปิด UI
        self.ui.screenGui.Enabled = false
        self.ui.skipButton.Visible = false
        self.ui.dialogueBox.Visible = false
        
        self.isPlaying = false
        self.currentCutscene = nil
        
        -- เรียก Callback
        if options.onComplete then
            options.onComplete()
        end
    end)
end

-- ข้าม Cutscene
function CutsceneManager:skip()
    if not self.isPlaying then return end
    self.skipRequested = true
end

-- ล็อค/ปลดล็อค Player
function CutsceneManager:_lockPlayer(lock)
    local character = self.player.Character
    if not character then return end
    
    local humanoid = character:FindFirstChild("Humanoid")
    if humanoid then
        if lock then
            humanoid.WalkSpeed = 0
            humanoid.JumpPower = 0
        else
            humanoid.WalkSpeed = 16
            humanoid.JumpPower = 50
        end
    end
end

-- แสดง Letterbox
function CutsceneManager:_showLetterbox()
    local LETTERBOX_HEIGHT = 70
    
    TweenService:Create(self.ui.letterboxTop, TweenInfo.new(0.5), {
        Size = UDim2.new(1, 0, 0, LETTERBOX_HEIGHT)
    }):Play()
    
    TweenService:Create(self.ui.letterboxBottom, TweenInfo.new(0.5), {
        Size = UDim2.new(1, 0, 0, LETTERBOX_HEIGHT)
    }):Play()
    
    task.wait(0.5)
end

-- ซ่อน Letterbox
function CutsceneManager:_hideLetterbox()
    TweenService:Create(self.ui.letterboxTop, TweenInfo.new(0.5), {
        Size = UDim2.new(1, 0, 0, 0)
    }):Play()
    
    TweenService:Create(self.ui.letterboxBottom, TweenInfo.new(0.5), {
        Size = UDim2.new(1, 0, 0, 0)
    }):Play()
    
    task.wait(0.5)
end

-- เคลื่อน Camera
function CutsceneManager:moveCameraTo(targetCFrame, duration, easingStyle)
    if self.skipRequested then return end
    
    duration = duration or 1
    easingStyle = easingStyle or Enum.EasingStyle.Quad
    
    local tween = TweenService:Create(self.camera,
        TweenInfo.new(duration, easingStyle, Enum.EasingDirection.InOut),
        {CFrame = targetCFrame}
    )
    tween:Play()
    tween.Completed:Wait()
end

-- หมุน Camera ดู Target
function CutsceneManager:lookAt(targetPosition, duration)
    if self.skipRequested then return end
    
    local newCFrame = CFrame.new(self.camera.CFrame.Position, targetPosition)
    self:moveCameraTo(newCFrame, duration)
end

-- สั่น Camera
function CutsceneManager:shake(intensity, duration)
    if self.skipRequested then return end
    
    local originalCFrame = self.camera.CFrame
    local endTime = tick() + duration
    
    while tick() < endTime and not self.skipRequested do
        local offset = Vector3.new(
            (math.random() - 0.5) * intensity,
            (math.random() - 0.5) * intensity,
            (math.random() - 0.5) * intensity
        )
        self.camera.CFrame = originalCFrame * CFrame.new(offset)
        RunService.RenderStepped:Wait()
    end
    
    self.camera.CFrame = originalCFrame
end

-- แสดง Dialogue
function CutsceneManager:showDialogue(speaker, text, options)
    if self.skipRequested then return end
    
    options = options or {}
    local typeSpeed = options.typeSpeed or 0.04  -- วินาทีต่อตัวอักษร
    local autoAdvance = options.autoAdvance or false
    local autoAdvanceDelay = options.autoAdvanceDelay or 2
    
    local dialogueBox = self.ui.dialogueBox
    local speakerLabel = self.ui.speakerLabel
    local dialogueText = self.ui.dialogueText
    local continueArrow = self.ui.continueArrow
    
    -- ตั้งค่า Speaker
    if speaker then
        speakerLabel.Text = "  " .. speaker
        speakerLabel.Visible = true
    else
        speakerLabel.Visible = false
    end
    
    -- แสดง Dialogue Box
    dialogueText.Text = ""
    dialogueBox.Visible = true
    continueArrow.Visible = false
    
    -- Type Writer Effect
    local displayedChars = 0
    local waitForAdvance = true
    
    -- ตัวอักษรปรากฏทีละตัว
    for i = 1, #text do
        if self.skipRequested then
            dialogueText.Text = text
            break
        end
        
        -- ตรวจสอบการกด Space หรือ Click เพื่อข้ามการพิมพ์
        if UserInputService:IsKeyDown(Enum.KeyCode.Space) or
           UserInputService:IsKeyDown(Enum.KeyCode.Return) then
            dialogueText.Text = text
            break
        end
        
        dialogueText.Text = string.sub(text, 1, i)
        
        -- Pause ที่ Punctuation
        local char = string.sub(text, i, i)
        if char == "." or char == "!" or char == "?" then
            task.wait(typeSpeed * 5)
        elseif char == "," then
            task.wait(typeSpeed * 2)
        else
            task.wait(typeSpeed)
        end
    end
    
    if self.skipRequested then return end
    
    if autoAdvance then
        task.wait(autoAdvanceDelay)
    else
        -- รอ Input
        continueArrow.Visible = true
        
        -- Animate arrow
        local arrowTween
        local function animateArrow()
            arrowTween = TweenService:Create(continueArrow, 
                TweenInfo.new(0.5, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, true),
                {TextTransparency = 0.5}
            )
            arrowTween:Play()
        end
        animateArrow()
        
        -- รอ Click หรือ Keypress
        local inputConn
        inputConn = UserInputService.InputBegan:Connect(function(input, processed)
            if processed then return end
            if input.UserInputType == Enum.UserInputType.MouseButton1 or
               input.KeyCode == Enum.KeyCode.Space or
               input.KeyCode == Enum.KeyCode.Return or
               input.KeyCode == Enum.KeyCode.E then
                waitForAdvance = false
            end
        end)
        
        while waitForAdvance and not self.skipRequested do
            task.wait(0.05)
        end
        
        inputConn:Disconnect()
        if arrowTween then arrowTween:Cancel() end
    end
    
    continueArrow.Visible = false
    dialogueBox.Visible = false
end

-- รอโดยสามารถ Skip ได้
function CutsceneManager:wait(duration)
    if self.skipRequested then return end
    
    local endTime = tick() + duration
    while tick() < endTime and not self.skipRequested do
        task.wait(0.05)
    end
end

return CutsceneManager
```

---

## 2. Cutscene Script Format

### 2.1 วิธีสร้าง Cutscene

```lua
-- Cutscene_Intro.lua (Module Script)
-- Cutscene แนะนำเกม

local TweenService = game:GetService("TweenService")

local IntroCutscene = {}
IntroCutscene.__index = IntroCutscene

function IntroCutscene.new()
    return setmetatable({}, IntroCutscene)
end

-- รัน Cutscene
function IntroCutscene:run(manager)
    local camera = manager.camera
    
    -- ===== Scene 1: Opening Shot =====
    -- วางกล้องที่ตำแหน่งเริ่มต้น
    camera.CFrame = CFrame.new(Vector3.new(0, 100, 200), Vector3.new(0, 0, 0))
    
    -- เคลื่อนลงมาช้าๆ
    manager:moveCameraTo(
        CFrame.new(Vector3.new(0, 30, 100), Vector3.new(0, 10, 0)),
        4,
        Enum.EasingStyle.Sine
    )
    
    if manager.skipRequested then return end
    
    -- ===== Scene 2: Dialogue =====
    manager:showDialogue(
        "ผู้เฒ่า",
        "ยินดีต้อนรับ วีรบุรุษผู้กล้าหาญ! ดินแดนของเราต้องการความช่วยเหลือของท่าน...",
        {typeSpeed = 0.05}
    )
    
    if manager.skipRequested then return end
    
    -- ===== Scene 3: Dramatic Camera Move =====
    -- Zoom เข้าไปที่ NPC
    local npcPosition = workspace:FindFirstChild("Merchant") and
        workspace.Merchant.PrimaryPart.Position or
        Vector3.new(10, 5, 10)
    
    manager:moveCameraTo(
        CFrame.new(npcPosition + Vector3.new(5, 3, 5), npcPosition),
        2
    )
    
    manager:showDialogue(
        "ผู้เฒ่า",
        "มีมังกรโบราณที่ตื่นขึ้นมาจากการหลับใหล และกำลังโจมตีหมู่บ้าน...",
        {typeSpeed = 0.04}
    )
    
    if manager.skipRequested then return end
    
    -- ===== Scene 4: Action Shot =====
    -- สั่นกล้องเพื่อความตื่นเต้น
    manager:shake(0.5, 1)
    
    manager:showDialogue(
        "ผู้เฒ่า", 
        "ท่านคือความหวังสุดท้ายของพวกเรา!",
        {typeSpeed = 0.06, autoAdvance = true, autoAdvanceDelay = 2}
    )
    
    if manager.skipRequested then return end
    
    -- ===== Scene 5: Final Shot =====
    -- กล้องกลับไปที่ Player
    local playerChar = manager.player.Character
    if playerChar and playerChar.PrimaryPart then
        local playerPos = playerChar.PrimaryPart.Position
        
        manager:moveCameraTo(
            CFrame.new(playerPos + Vector3.new(0, 5, -10), playerPos),
            2
        )
    end
    
    manager:wait(1)
    -- Cutscene จบ - CutsceneManager จะ restore camera
end

return IntroCutscene
```

---

## 3. Camera Path System

### 3.1 Smooth Camera Path

```lua
-- CameraPath.lua (Module Script)
-- ระบบ Camera Path แบบ Smooth

local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")

local CameraPath = {}
CameraPath.__index = CameraPath

function CameraPath.new(keyframes)
    local self = setmetatable({}, CameraPath)
    
    -- Keyframes: {time, position, lookAt, fov}
    self.keyframes = keyframes or {}
    self.totalDuration = 0
    
    -- คำนวณ Total Duration
    if #keyframes > 0 then
        self.totalDuration = keyframes[#keyframes].time
    end
    
    return self
end

-- เพิ่ม Keyframe
function CameraPath:addKeyframe(time, position, lookAt, fov)
    table.insert(self.keyframes, {
        time = time,
        position = position,
        lookAt = lookAt,
        fov = fov or 70
    })
    
    -- เรียงตาม time
    table.sort(self.keyframes, function(a, b) return a.time < b.time end)
    
    self.totalDuration = self.keyframes[#self.keyframes].time
end

-- Interpolate ค่าระหว่าง 2 Keyframes
local function lerpVector3(a, b, t)
    return a + (b - a) * t
end

local function easeInOut(t)
    return t < 0.5 and 2 * t * t or 1 - (-2 * t + 2)^2 / 2
end

-- หา CFrame ที่เวลาที่กำหนด
function CameraPath:getCFrameAt(time)
    if #self.keyframes == 0 then return CFrame.new() end
    if #self.keyframes == 1 then
        local kf = self.keyframes[1]
        return CFrame.new(kf.position, kf.lookAt or kf.position + Vector3.new(0, 0, -1))
    end
    
    -- หา Keyframes ที่อยู่รอบๆ เวลานั้น
    local prevKf, nextKf
    
    for i = 1, #self.keyframes - 1 do
        if self.keyframes[i].time <= time and self.keyframes[i+1].time >= time then
            prevKf = self.keyframes[i]
            nextKf = self.keyframes[i+1]
            break
        end
    end
    
    if not prevKf then
        local kf = self.keyframes[#self.keyframes]
        return CFrame.new(kf.position, kf.lookAt or kf.position + Vector3.new(0, 0, -1))
    end
    
    -- คำนวณ t (0-1 ระหว่าง 2 keyframes)
    local segmentDuration = nextKf.time - prevKf.time
    local t = (time - prevKf.time) / segmentDuration
    t = easeInOut(t)
    
    -- Interpolate Position
    local pos = lerpVector3(prevKf.position, nextKf.position, t)
    
    -- Interpolate LookAt
    local lookAt
    if prevKf.lookAt and nextKf.lookAt then
        lookAt = lerpVector3(prevKf.lookAt, nextKf.lookAt, t)
    end
    
    if lookAt then
        return CFrame.new(pos, lookAt)
    else
        return CFrame.new(pos)
    end
end

-- รัน Path
function CameraPath:play(camera, skipCallback)
    local startTime = tick()
    
    return task.spawn(function()
        while true do
            local elapsed = tick() - startTime
            
            if elapsed >= self.totalDuration then break end
            if skipCallback and skipCallback() then break end
            
            camera.CFrame = self:getCFrameAt(elapsed)
            RunService.RenderStepped:Wait()
        end
        
        -- จบที่ Keyframe สุดท้าย
        local lastKf = self.keyframes[#self.keyframes]
        if lastKf then
            local lookAt = lastKf.lookAt or lastKf.position + Vector3.new(0, 0, -1)
            camera.CFrame = CFrame.new(lastKf.position, lookAt)
        end
    end)
end

return CameraPath

--[[
ตัวอย่างการใช้งาน:

local CameraPath = require(script.CameraPath)

local path = CameraPath.new()
path:addKeyframe(0,  Vector3.new(0, 50, 100),  Vector3.new(0, 0, 0))
path:addKeyframe(3,  Vector3.new(30, 20, 50),  Vector3.new(0, 5, 0))
path:addKeyframe(5,  Vector3.new(10, 10, 20),  Vector3.new(0, 8, 0))
path:addKeyframe(8,  Vector3.new(-10, 5, 10),  Vector3.new(0, 10, 0))

-- รัน Path (ใน Cutscene)
local thread = path:play(workspace.CurrentCamera, function()
    return manager.skipRequested
end)

task.wait(path.totalDuration)  -- รอจน Path จบ
]]
```

---

## 4. Dialogue System ขั้นสูง

### 4.1 Branching Dialogue

```lua
-- DialogueTree.lua (Module Script)
-- ระบบ Dialogue แบบ Branching (มีตัวเลือก)

local DialogueTree = {}
DialogueTree.__index = DialogueTree

function DialogueTree.new(nodes)
    local self = setmetatable({}, DialogueTree)
    self.nodes = nodes or {}
    self.currentNode = nil
    return self
end

-- โหนด Dialogue
--[[
Node Format:
{
    id = "start",
    speaker = "ชื่อ NPC",
    text = "ข้อความ...",
    choices = {                      -- optional: ตัวเลือก
        {text = "ตัวเลือก 1", next = "node2"},
        {text = "ตัวเลือก 2", next = "node3", condition = function() return level >= 5 end}
    },
    next = "node2",                  -- optional: ไป Node ต่อไปอัตโนมัติ
    action = function() end,         -- optional: ทำอะไรบางอย่าง
    end = true                       -- optional: จบ Dialogue
}
]]

-- เรียกใช้ Dialogue Tree
function DialogueTree:run(manager, startNodeId)
    local currentId = startNodeId or "start"
    
    while currentId and not manager.skipRequested do
        local node = self.nodes[currentId]
        if not node then
            warn("ไม่พบ Node:", currentId)
            break
        end
        
        -- เรียก Action ถ้ามี
        if node.action then
            local success, err = pcall(node.action)
            if not success then
                warn("Dialogue action error:", err)
            end
        end
        
        -- จบ Dialogue
        if node.end then
            if node.text then
                manager:showDialogue(node.speaker, node.text, {autoAdvance = true})
            end
            break
        end
        
        -- มีตัวเลือก
        if node.choices and #node.choices > 0 then
            -- แสดง Text ก่อน
            if node.text then
                manager:showDialogue(node.speaker, node.text)
            end
            
            -- แสดง Choices
            local choice = self:_showChoices(manager, node.choices)
            if choice then
                currentId = choice.next
            else
                break
            end
        else
            -- ไม่มีตัวเลือก - แสดง Text และไป Next
            manager:showDialogue(node.speaker, node.text)
            currentId = node.next
        end
    end
end

-- แสดงตัวเลือก
function DialogueTree:_showChoices(manager, choices)
    -- กรอง Choices ตาม Condition
    local availableChoices = {}
    for _, choice in ipairs(choices) do
        if not choice.condition or choice.condition() then
            table.insert(availableChoices, choice)
        end
    end
    
    if #availableChoices == 0 then return nil end
    if #availableChoices == 1 then return availableChoices[1] end
    
    -- สร้าง Choice UI
    local TweenService = game:GetService("TweenService")
    local UserInputService = game:GetService("UserInputService")
    local Players = game:GetService("Players")
    
    local playerGui = Players.LocalPlayer:WaitForChild("PlayerGui")
    
    local choiceGui = Instance.new("ScreenGui")
    choiceGui.Name = "ChoiceUI"
    choiceGui.ResetOnSpawn = false
    choiceGui.Parent = playerGui
    
    local choiceFrame = Instance.new("Frame")
    choiceFrame.Size = UDim2.new(0, 400, 0, 50 + #availableChoices * 50)
    choiceFrame.Position = UDim2.new(0.5, -200, 0.6, 0)
    choiceFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
    choiceFrame.BackgroundTransparency = 0.2
    choiceFrame.BorderSizePixel = 0
    choiceFrame.Parent = choiceGui
    
    local frameCorner = Instance.new("UICorner")
    frameCorner.CornerRadius = UDim.new(0, 8)
    frameCorner.Parent = choiceFrame
    
    local frameLayout = Instance.new("UIListLayout")
    frameLayout.SortOrder = Enum.SortOrder.LayoutOrder
    frameLayout.Padding = UDim.new(0, 5)
    frameLayout.Parent = choiceFrame
    
    local framePadding = Instance.new("UIPadding")
    framePadding.PaddingAll = UDim.new(0, 10)
    framePadding.Parent = choiceFrame
    
    -- รอ Choice
    local selectedChoice = nil
    
    for i, choice in ipairs(availableChoices) do
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(1, 0, 0, 40)
        btn.BackgroundColor3 = Color3.fromRGB(40, 60, 100)
        btn.Text = string.format("%d. %s", i, choice.text)
        btn.TextColor3 = Color3.fromRGB(220, 220, 220)
        btn.TextSize = 14
        btn.Font = Enum.Font.Gotham
        btn.BorderSizePixel = 0
        btn.LayoutOrder = i
        btn.TextXAlignment = Enum.TextXAlignment.Left
        btn.Parent = choiceFrame
        
        local btnCorner = Instance.new("UICorner")
        btnCorner.CornerRadius = UDim.new(0, 6)
        btnCorner.Parent = btn
        
        local btnPadding = Instance.new("UIPadding")
        btnPadding.PaddingLeft = UDim.new(0, 10)
        btnPadding.Parent = btn
        
        -- Hover Effect
        btn.MouseEnter:Connect(function()
            TweenService:Create(btn, TweenInfo.new(0.1), {
                BackgroundColor3 = Color3.fromRGB(60, 90, 150)
            }):Play()
        end)
        
        btn.MouseLeave:Connect(function()
            TweenService:Create(btn, TweenInfo.new(0.1), {
                BackgroundColor3 = Color3.fromRGB(40, 60, 100)
            }):Play()
        end)
        
        btn.Activated:Connect(function()
            selectedChoice = choice
        end)
    end
    
    -- รอจน Player เลือก
    while not selectedChoice and not manager.skipRequested do
        task.wait(0.05)
    end
    
    choiceGui:Destroy()
    return selectedChoice
end

return DialogueTree
```

---

## 5. ข้อผิดพลาดที่พบบ่อย

### ❌ ข้อผิดพลาดที่ 1: ไม่คืนค่า Camera Mode

```lua
-- ❌ แบบผิด: เปลี่ยน Camera แต่ไม่คืน
local function badCutscene()
    workspace.CurrentCamera.CameraType = Enum.CameraType.Scriptable
    -- ... cutscene ...
    -- ลืม return camera type!
end

-- ✅ แบบถูก: ใช้ pcall และ always restore
local function goodCutscene()
    local originalType = workspace.CurrentCamera.CameraType
    
    local success, err = pcall(function()
        workspace.CurrentCamera.CameraType = Enum.CameraType.Scriptable
        -- ... cutscene ...
    end)
    
    -- คืนค่าเสมอ ไม่ว่าจะ error หรือไม่
    workspace.CurrentCamera.CameraType = originalType
    
    if not success then warn(err) end
end
```

### ❌ ข้อผิดพลาดที่ 2: ไม่ Handle Player Character ที่ Reset

```lua
-- ❌ แบบผิด: ไม่ตรวจสอบ Character
local function lockPlayer(lock)
    local char = player.Character
    char.Humanoid.WalkSpeed = 0  -- Error ถ้าไม่มี Character!
end

-- ✅ แบบถูก: ตรวจสอบก่อนเสมอ
local function lockPlayerSafe(lock)
    local char = player.Character
    if not char then return end
    
    local humanoid = char:FindFirstChild("Humanoid")
    if not humanoid then return end
    
    humanoid.WalkSpeed = lock and 0 or 16
    humanoid.JumpPower = lock and 0 or 50
end
```

---

## 6. แบบฝึกหัด

### แบบฝึกหัดที่ 1: Intro Cutscene
สร้าง Cutscene แนะนำเกม (ความยาว 30 วินาที):
- Camera sweep ผ่านสถานที่สำคัญ 3 แห่ง
- มี Dialogue กับ NPC หนึ่งตัว
- Letterbox Effect และ Skip Button

### แบบฝึกหัดที่ 2: Boss Introduction
สร้าง Cutscene Boss Entry:
- Camera เริ่มที่ Player
- Zoom ออกแล้ว Pan ไปหา Boss
- Camera Shake เมื่อ Boss ปรากฏ
- Boss พูด Dialogue

### แบบฝึกหัดที่ 3: Branching NPC Dialogue
สร้าง NPC ที่มี Dialogue Tree:
- Dialogue ต่างกันตาม Level
- 3+ ตัวเลือก
- บาง Choice เปิด Quest

---

## สรุป

Cutscene System ที่ดีต้องมี:
1. **Camera Control**: เคลื่อน Camera อย่าง Smooth
2. **Player Lock**: ป้องกันการ Interrupt
3. **Skip System**: ผู้เล่นต้องสามารถข้ามได้
4. **Dialogue**: Typewriter Effect และ Branching
5. **Cleanup**: คืนค่าทุกอย่างเมื่อ Cutscene จบ

ในส่วนถัดไป (Part 77) เราจะสร้าง Day/Night Cycle System พร้อมการเปลี่ยนแปลง Lighting และ Atmosphere
