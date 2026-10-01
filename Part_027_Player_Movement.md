# ตอนที่ 27: การควบคุมการเคลื่อนที่ของผู้เล่น (Player Movement)

## บทนำ

การเคลื่อนที่ของผู้เล่นเป็นหัวใจสำคัญของเกม Roblox ในบทนี้เราจะเรียนรู้วิธีการควบคุมการเคลื่อนที่ทั้งแบบพื้นฐานและขั้นสูง ไม่ว่าจะเป็นการปรับความเร็ว การกระโดด การ Dash หรือการสร้างระบบการเคลื่อนที่แบบกำหนดเอง

---

## 27.1 พื้นฐานการเคลื่อนที่

### 27.1.1 UserInputService

```lua
-- LocalScript
local UserInputService = game:GetService("UserInputService")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")

-- ตรวจสอบปุ่มที่กด
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end  -- ถ้า UI กำลังใช้งาน Input ให้ข้าม
    
    -- ตรวจสอบประเภท Input
    if input.KeyCode == Enum.KeyCode.E then
        print("กด E!")
        -- ทำอะไรบางอย่าง
    end
    
    if input.KeyCode == Enum.KeyCode.Space then
        print("กด Space (กระโดด)")
    end
    
    -- Mouse Click
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        print("Click ซ้าย!")
    end
end)

-- เมื่อปล่อยปุ่ม
UserInputService.InputEnded:Connect(function(input, gameProcessed)
    if input.KeyCode == Enum.KeyCode.E then
        print("ปล่อย E")
    end
end)
```

### 27.1.2 ContextActionService

```lua
-- LocalScript
local ContextActionService = game:GetService("ContextActionService")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

local character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")

-- Bind action ให้กับปุ่ม
local function onSprintAction(actionName, inputState, inputObject)
    if inputState == Enum.UserInputState.Begin then
        -- เริ่ม Sprint
        humanoid.WalkSpeed = 32
        print("เริ่ม Sprint!")
    elseif inputState == Enum.UserInputState.End then
        -- หยุด Sprint
        humanoid.WalkSpeed = 16
        print("หยุด Sprint")
    end
end

-- Bind ปุ่ม Shift สำหรับ Sprint
ContextActionService:BindAction(
    "Sprint",           -- ชื่อ Action
    onSprintAction,     -- Function
    true,               -- สร้าง Mobile Button ด้วย
    Enum.KeyCode.LeftShift,  -- PC Key
    Enum.KeyCode.ButtonL3    -- Controller
)

-- ยกเลิก Binding
-- ContextActionService:UnbindAction("Sprint")
```

---

## 27.2 ระบบ Sprint

```lua
-- LocalScript: SprintSystem
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer
local character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")

-- ค่าความเร็ว
local NORMAL_SPEED = 16
local SPRINT_SPEED = 32
local WALK_SPEED = 8

-- Stamina System
local maxStamina = 100
local currentStamina = maxStamina
local staminaDrainRate = 20   -- ต่อวินาที
local staminaRegenRate = 10   -- ต่อวินาที
local isSprinting = false

-- อัพเดต Stamina
RunService.Heartbeat:Connect(function(dt)
    if isSprinting and humanoid.MoveDirection.Magnitude > 0 then
        -- ลด Stamina ขณะวิ่ง
        currentStamina = math.max(0, currentStamina - staminaDrainRate * dt)
        
        if currentStamina <= 0 then
            -- หมด Stamina
            isSprinting = false
            humanoid.WalkSpeed = NORMAL_SPEED
        end
    else
        -- ฟื้นฟู Stamina
        currentStamina = math.min(maxStamina, currentStamina + staminaRegenRate * dt)
    end
end)

-- ตรวจสอบ Input
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    if input.KeyCode == Enum.KeyCode.LeftShift then
        if currentStamina > 10 then  -- ต้องมี Stamina มากกว่า 10
            isSprinting = true
            humanoid.WalkSpeed = SPRINT_SPEED
        end
    end
    
    if input.KeyCode == Enum.KeyCode.LeftControl then
        -- Crouch
        humanoid.WalkSpeed = WALK_SPEED
        -- ลดขนาดตัวละครด้วย HipHeight
        humanoid.HipHeight = 0.5
    end
end)

UserInputService.InputEnded:Connect(function(input, gameProcessed)
    if input.KeyCode == Enum.KeyCode.LeftShift then
        isSprinting = false
        humanoid.WalkSpeed = NORMAL_SPEED
    end
    
    if input.KeyCode == Enum.KeyCode.LeftControl then
        humanoid.WalkSpeed = NORMAL_SPEED
        humanoid.HipHeight = 2  -- ค่าปกติ
    end
end)

-- Character ใหม่
LocalPlayer.CharacterAdded:Connect(function(newCharacter)
    character = newCharacter
    humanoid = newCharacter:WaitForChild("Humanoid")
    currentStamina = maxStamina
    isSprinting = false
end)
```

---

## 27.3 ระบบ Double Jump

```lua
-- LocalScript: DoubleJump
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")

local LocalPlayer = Players.LocalPlayer
local character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")

local canDoubleJump = false
local hasDoubleJumped = false
local jumpDebounce = false

-- ตรวจสอบ State
humanoid.StateChanged:Connect(function(_, newState)
    if newState == Enum.HumanoidStateType.Jumping then
        -- อยู่ระหว่างกระโดด
        if not jumpDebounce then
            jumpDebounce = true
            canDoubleJump = false
            
            wait(0.2)  -- รอนิดนึงก่อนอนุญาต double jump
            canDoubleJump = not hasDoubleJumped
            jumpDebounce = false
        end
    elseif newState == Enum.HumanoidStateType.Freefall then
        -- กำลังตกลงมา (สามารถ double jump ได้)
        wait(0.1)
        canDoubleJump = not hasDoubleJumped
    elseif newState == Enum.HumanoidStateType.Landed then
        -- ลงจอดแล้ว รีเซ็ต
        canDoubleJump = false
        hasDoubleJumped = false
    end
end)

-- ตรวจสอบการกด Space
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    if input.KeyCode == Enum.KeyCode.Space then
        if canDoubleJump and not hasDoubleJumped then
            hasDoubleJumped = true
            canDoubleJump = false
            
            -- Double Jump!
            humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
            
            -- ใช้ Velocity สำหรับ jump ที่แรงขึ้น
            local hrp = character:WaitForChild("HumanoidRootPart")
            hrp.Velocity = Vector3.new(
                hrp.Velocity.X,
                humanoid.JumpPower,
                hrp.Velocity.Z
            )
            
            print("Double Jump!")
        end
    end
end)

-- Character ใหม่
LocalPlayer.CharacterAdded:Connect(function(newCharacter)
    character = newCharacter
    humanoid = newCharacter:WaitForChild("Humanoid")
    canDoubleJump = false
    hasDoubleJumped = false
    jumpDebounce = false
end)
```

---

## 27.4 ระบบ Dash

```lua
-- LocalScript: DashSystem
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")

local LocalPlayer = Players.LocalPlayer
local character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local hrp = character:WaitForChild("HumanoidRootPart")

local dashCooldown = 1      -- วินาที
local dashDuration = 0.3    -- วินาที
local dashSpeed = 80        -- ความเร็ว dash
local isDashing = false
local lastDashTime = 0

local function dash()
    local now = tick()
    if isDashing or (now - lastDashTime) < dashCooldown then
        print("Dash ยังไม่พร้อม!")
        return
    end
    
    isDashing = true
    lastDashTime = now
    
    -- ทิศทาง dash
    local dashDirection = humanoid.MoveDirection
    if dashDirection.Magnitude == 0 then
        -- ถ้าไม่ได้เดิน dash ไปข้างหน้า
        dashDirection = hrp.CFrame.LookVector
    end
    dashDirection = dashDirection.Unit
    
    -- บันทึกความเร็วเดิม
    local originalSpeed = humanoid.WalkSpeed
    
    -- เพิ่มความเร็ว
    humanoid.WalkSpeed = dashSpeed
    
    -- สร้าง VelocityForce
    local bodyVelocity = Instance.new("LinearVelocity")
    bodyVelocity.VectorVelocity = dashDirection * dashSpeed
    bodyVelocity.MaxForce = 100000
    bodyVelocity.Attachment0 = hrp:WaitForChild("RootAttachment")
    bodyVelocity.Parent = hrp
    
    -- รอจบ Dash
    wait(dashDuration)
    
    -- ลบ Force
    bodyVelocity:Destroy()
    humanoid.WalkSpeed = originalSpeed
    isDashing = false
    
    print("Dash เสร็จ!")
end

-- Bind ปุ่ม Q สำหรับ Dash
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    if input.KeyCode == Enum.KeyCode.Q then
        dash()
    end
end)

-- Character ใหม่
LocalPlayer.CharacterAdded:Connect(function(newCharacter)
    character = newCharacter
    humanoid = newCharacter:WaitForChild("Humanoid")
    hrp = newCharacter:WaitForChild("HumanoidRootPart")
    isDashing = false
end)
```

---

## 27.5 ระบบ Wall Jump

```lua
-- LocalScript: WallJump
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer
local character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local hrp = character:WaitForChild("HumanoidRootPart")

local canWallJump = false
local wallNormal = Vector3.new(0, 0, 0)
local wallJumpCooldown = false

-- ตรวจสอบว่าติดผนังหรือไม่
RunService.Heartbeat:Connect(function()
    local state = humanoid:GetState()
    
    if state == Enum.HumanoidStateType.Freefall or 
       state == Enum.HumanoidStateType.Jumping then
        
        -- Cast Ray ไปด้านข้างทั้งสองข้าง
        local leftRay = workspace:Raycast(
            hrp.Position,
            hrp.CFrame.RightVector * -2,
            RaycastParams.new()
        )
        
        local rightRay = workspace:Raycast(
            hrp.Position,
            hrp.CFrame.RightVector * 2,
            RaycastParams.new()
        )
        
        if leftRay and leftRay.Instance and 
           not leftRay.Instance:IsDescendantOf(character) then
            canWallJump = true
            wallNormal = leftRay.Normal
        elseif rightRay and rightRay.Instance and 
               not rightRay.Instance:IsDescendantOf(character) then
            canWallJump = true
            wallNormal = rightRay.Normal
        else
            canWallJump = false
        end
    else
        canWallJump = false
    end
end)

-- Wall Jump
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    if input.KeyCode == Enum.KeyCode.Space then
        if canWallJump and not wallJumpCooldown then
            wallJumpCooldown = true
            
            -- Jump ออกจากผนัง
            local jumpForce = wallNormal * 30 + Vector3.new(0, humanoid.JumpPower, 0)
            hrp.Velocity = jumpForce
            
            print("Wall Jump!")
            
            wait(0.5)
            wallJumpCooldown = false
        end
    end
end)
```

---

## 27.6 ระบบ Slide

```lua
-- LocalScript: SlideSystem
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer
local character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local hrp = character:WaitForChild("HumanoidRootPart")

local isSliding = false
local slideDuration = 1.2
local slideSpeed = 50

local function startSlide()
    if isSliding then return end
    
    local state = humanoid:GetState()
    if state ~= Enum.HumanoidStateType.Running and
       state ~= Enum.HumanoidStateType.Idle then
        return
    end
    
    isSliding = true
    
    -- ทิศทาง slide
    local slideDir = hrp.CFrame.LookVector
    
    -- ลดความสูง
    humanoid.HipHeight = 0.5
    
    -- เพิ่มความเร็ว
    local bodyVelocity = Instance.new("BodyVelocity")
    bodyVelocity.Velocity = slideDir * slideSpeed
    bodyVelocity.MaxForce = Vector3.new(math.huge, 0, math.huge)
    bodyVelocity.Parent = hrp
    
    -- ค่อยๆ ลดความเร็ว
    local startTime = tick()
    local connection
    connection = RunService.Heartbeat:Connect(function()
        local elapsed = tick() - startTime
        local progress = elapsed / slideDuration
        
        if progress >= 1 then
            -- หยุด Slide
            bodyVelocity:Destroy()
            humanoid.HipHeight = 2
            isSliding = false
            connection:Disconnect()
        else
            -- Lerp ความเร็ว
            local currentSpeed = slideSpeed * (1 - progress)
            bodyVelocity.Velocity = slideDir * currentSpeed
        end
    end)
end

-- Bind ปุ่ม C หรือ Left Ctrl
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    if input.KeyCode == Enum.KeyCode.C or 
       input.KeyCode == Enum.KeyCode.LeftControl then
        -- ตรวจสอบว่ากำลังวิ่งหรือไม่
        if humanoid.WalkSpeed >= 20 then
            startSlide()
        end
    end
end)
```

---

## 27.7 MoveTo System (NPC Movement)

```lua
-- Script: NPCMovement
local PathfindingService = game:GetService("PathfindingService")

local function moveNPCTo(npc, targetPosition)
    local humanoid = npc:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end
    
    -- สร้าง Path
    local path = PathfindingService:CreatePath({
        AgentRadius = 2,
        AgentHeight = 5,
        AgentCanJump = true,
        AgentJumpHeight = 7.2,
        AgentMaxSlope = 45
    })
    
    -- คำนวณ Path
    local success, errorMessage = pcall(function()
        path:ComputeAsync(npc.HumanoidRootPart.Position, targetPosition)
    end)
    
    if not success then
        print("ไม่สามารถหา Path ได้:", errorMessage)
        return
    end
    
    if path.Status == Enum.PathStatus.Success then
        -- เดินตาม Waypoints
        local waypoints = path:GetWaypoints()
        
        for _, waypoint in ipairs(waypoints) do
            if waypoint.Action == Enum.PathWaypointAction.Jump then
                humanoid.Jump = true
            end
            
            humanoid:MoveTo(waypoint.Position)
            
            -- รอถึง Waypoint
            local reached = humanoid.MoveToFinished:Wait(5)
            if not reached then
                -- ไม่ถึง Waypoint ในเวลา
                print("ไม่ถึง Waypoint")
                break
            end
        end
        
        print("ถึงจุดหมายแล้ว!")
    else
        print("ไม่มีเส้นทาง, Status:", path.Status)
    end
end

-- ตัวอย่าง: NPC เดินไปหา Player
local Players = game:GetService("Players")
local npc = workspace.NPC  -- Model ชื่อ NPC

while true do
    wait(0.5)
    
    -- หาผู้เล่นที่ใกล้ที่สุด
    local closestPlayer = nil
    local closestDistance = math.huge
    
    for _, player in ipairs(Players:GetPlayers()) do
        local character = player.Character
        if character then
            local hrp = character:FindFirstChild("HumanoidRootPart")
            local npcHrp = npc:FindFirstChild("HumanoidRootPart")
            
            if hrp and npcHrp then
                local distance = (hrp.Position - npcHrp.Position).Magnitude
                if distance < closestDistance then
                    closestDistance = distance
                    closestPlayer = player
                end
            end
        end
    end
    
    if closestPlayer and closestDistance < 100 then
        moveNPCTo(npc, closestPlayer.Character.HumanoidRootPart.Position)
    end
end
```

---

## 27.8 ระบบ Teleport

```lua
-- Script: TeleportSystem
local Players = game:GetService("Players")

-- Teleport ผู้เล่นไปยังตำแหน่ง
local function teleportToPosition(player, position)
    local character = player.Character
    if not character then return end
    
    local hrp = character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    
    -- Teleport
    hrp.CFrame = CFrame.new(position + Vector3.new(0, 3, 0))
    
    print(player.Name, "teleport ไป", position)
end

-- Teleport ผู้เล่นไปหาผู้เล่นอื่น
local function teleportToPlayer(fromPlayer, toPlayer)
    local toCharacter = toPlayer.Character
    if not toCharacter then return end
    
    local toHrp = toCharacter:FindFirstChild("HumanoidRootPart")
    if not toHrp then return end
    
    teleportToPosition(fromPlayer, toHrp.Position)
end

-- Teleport ผู้เล่นทั้งหมดไปที่ Arena
local function teleportAllToArena()
    local arenaPosition = Vector3.new(0, 5, 0)
    local spread = 10
    
    local players = Players:GetPlayers()
    for i, player in ipairs(players) do
        -- กระจายตำแหน่งไม่ให้ซ้อนทับ
        local angle = (i / #players) * math.pi * 2
        local offset = Vector3.new(
            math.cos(angle) * spread,
            0,
            math.sin(angle) * spread
        )
        
        teleportToPosition(player, arenaPosition + offset)
    end
end

-- ตัวอย่างการใช้งาน
local teleportPad = workspace.TeleportPad

teleportPad.Touched:Connect(function(hit)
    local character = hit.Parent
    local player = Players:GetPlayerFromCharacter(character)
    
    if player then
        teleportToPosition(player, Vector3.new(100, 5, 100))
    end
end)
```

---

## 27.9 ระบบ Flight

```lua
-- LocalScript: FlightSystem
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer
local character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local hrp = character:WaitForChild("HumanoidRootPart")

local isFlying = false
local flySpeed = 30
local flyBodyVelocity = nil
local flyBodyGyro = nil

local function startFly()
    if isFlying then return end
    isFlying = true
    
    -- ปิด Gravity
    humanoid.PlatformStand = true
    
    -- Body Velocity
    flyBodyVelocity = Instance.new("BodyVelocity")
    flyBodyVelocity.Velocity = Vector3.zero
    flyBodyVelocity.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
    flyBodyVelocity.Parent = hrp
    
    -- Body Gyro (หมุนตามทิศทาง)
    flyBodyGyro = Instance.new("BodyGyro")
    flyBodyGyro.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
    flyBodyGyro.P = 1e5
    flyBodyGyro.D = 1e3
    flyBodyGyro.CFrame = hrp.CFrame
    flyBodyGyro.Parent = hrp
    
    print("เริ่มบิน!")
end

local function stopFly()
    if not isFlying then return end
    isFlying = false
    
    humanoid.PlatformStand = false
    
    if flyBodyVelocity then
        flyBodyVelocity:Destroy()
        flyBodyVelocity = nil
    end
    
    if flyBodyGyro then
        flyBodyGyro:Destroy()
        flyBodyGyro = nil
    end
    
    print("หยุดบิน")
end

-- อัพเดต Fly
RunService.Heartbeat:Connect(function()
    if not isFlying then return end
    
    local camera = workspace.CurrentCamera
    local moveVector = Vector3.zero
    
    -- ตรวจสอบปุ่มที่กด
    if UserInputService:IsKeyDown(Enum.KeyCode.W) then
        moveVector = moveVector + camera.CFrame.LookVector
    end
    if UserInputService:IsKeyDown(Enum.KeyCode.S) then
        moveVector = moveVector - camera.CFrame.LookVector
    end
    if UserInputService:IsKeyDown(Enum.KeyCode.A) then
        moveVector = moveVector - camera.CFrame.RightVector
    end
    if UserInputService:IsKeyDown(Enum.KeyCode.D) then
        moveVector = moveVector + camera.CFrame.RightVector
    end
    if UserInputService:IsKeyDown(Enum.KeyCode.E) then
        moveVector = moveVector + Vector3.new(0, 1, 0)
    end
    if UserInputService:IsKeyDown(Enum.KeyCode.Q) then
        moveVector = moveVector - Vector3.new(0, 1, 0)
    end
    
    -- ตั้งค่าความเร็ว
    if moveVector.Magnitude > 0 then
        flyBodyVelocity.Velocity = moveVector.Unit * flySpeed
        flyBodyGyro.CFrame = CFrame.new(hrp.Position, hrp.Position + moveVector)
    else
        flyBodyVelocity.Velocity = Vector3.zero
    end
end)

-- Toggle Fly
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    if input.KeyCode == Enum.KeyCode.F then
        if isFlying then
            stopFly()
        else
            startFly()
        end
    end
end)
```

---

## 27.10 Mobile Controls

```lua
-- LocalScript: MobileControls
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")

local LocalPlayer = Players.LocalPlayer

-- ตรวจสอบว่าเป็น Mobile หรือไม่
if not UserInputService.TouchEnabled then
    print("ไม่ใช่ Mobile")
    return
end

local character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")

-- สร้าง Touch Button สำหรับ Dash
local playerGui = LocalPlayer.PlayerGui
local screenGui = Instance.new("ScreenGui")
screenGui.Parent = playerGui

local dashButton = Instance.new("TextButton")
dashButton.Size = UDim2.new(0, 80, 0, 80)
dashButton.Position = UDim2.new(1, -100, 1, -100)
dashButton.Text = "DASH"
dashButton.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
dashButton.TextColor3 = Color3.new(1, 1, 1)
dashButton.FontSize = Enum.FontSize.Size18
dashButton.Parent = screenGui

-- เพิ่ม Corner
local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(1, 0)
corner.Parent = dashButton

-- Touch Events
dashButton.TouchTap:Connect(function()
    print("Dash button กด!")
    -- เรียกใช้ dash function
end)

-- Virtual Joystick (ใช้ Roblox default หรือสร้างเอง)
-- Roblox มี Default Mobile Controls แล้ว แต่ถ้าต้องการ Custom:
local thumbstick = Instance.new("ImageLabel")
thumbstick.Size = UDim2.new(0, 150, 0, 150)
thumbstick.Position = UDim2.new(0, 20, 1, -170)
thumbstick.BackgroundTransparency = 0.5
thumbstick.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
thumbstick.Parent = screenGui

local thumb = Instance.new("ImageLabel")
thumb.Size = UDim2.new(0, 60, 0, 60)
thumb.Position = UDim2.new(0.5, -30, 0.5, -30)
thumb.BackgroundColor3 = Color3.fromRGB(200, 200, 200)
thumb.Parent = thumbstick

-- เพิ่ม UICorner ให้ thumbstick
local tCorner = Instance.new("UICorner")
tCorner.CornerRadius = UDim.new(1, 0)
tCorner.Parent = thumbstick

local tCorner2 = Instance.new("UICorner")
tCorner2.CornerRadius = UDim.new(1, 0)
tCorner2.Parent = thumb
```

---

## 27.11 Anti-Cheat Movement Validation

```lua
-- Server Script: MovementValidation
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local MAX_SPEED = 50  -- ความเร็วสูงสุดที่อนุญาต
local MAX_JUMP_HEIGHT = 100  -- ความสูงกระโดดสูงสุดที่อนุญาต
local CHECK_INTERVAL = 0.5  -- ตรวจสอบทุก X วินาที

local playerPositions = {}

Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function(character)
        local hrp = character:WaitForChild("HumanoidRootPart")
        local humanoid = character:WaitForChild("Humanoid")
        
        playerPositions[player] = {
            lastPosition = hrp.Position,
            lastTime = tick()
        }
        
        -- ตรวจสอบการเคลื่อนที่
        while character.Parent do
            wait(CHECK_INTERVAL)
            
            if not playerPositions[player] then break end
            
            local currentPos = hrp.Position
            local lastData = playerPositions[player]
            
            local distance = (currentPos - lastData.lastPosition).Magnitude
            local timeDiff = tick() - lastData.lastTime
            local speed = distance / timeDiff
            
            if speed > MAX_SPEED then
                print("⚠️ ตรวจพบการเคลื่อนที่ผิดปกติของ", player.Name)
                print("ความเร็ว:", speed, "สูงสุดที่อนุญาต:", MAX_SPEED)
                
                -- Kick หรือ Teleport กลับ
                -- player:Kick("Speed Hack Detected")
                hrp.CFrame = CFrame.new(lastData.lastPosition)
            end
            
            playerPositions[player] = {
                lastPosition = currentPos,
                lastTime = tick()
            }
        end
    end)
end)

Players.PlayerRemoving:Connect(function(player)
    playerPositions[player] = nil
end)
```

---

## 27.12 แบบฝึกหัด

### แบบฝึกหัดที่ 1: Sprint with Stamina Bar
สร้างระบบ Sprint ที่มีแถบ Stamina แสดงใน GUI

```lua
-- LocalScript: SprintWithStaminaBar
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer
local character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")

-- สร้าง Stamina Bar
local playerGui = LocalPlayer.PlayerGui
local screenGui = Instance.new("ScreenGui")
screenGui.Parent = playerGui

local staminaFrame = Instance.new("Frame")
staminaFrame.Size = UDim2.new(0, 200, 0, 20)
staminaFrame.Position = UDim2.new(0.5, -100, 1, -50)
staminaFrame.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
staminaFrame.Parent = screenGui

local staminaBar = Instance.new("Frame")
staminaBar.Size = UDim2.new(1, 0, 1, 0)
staminaBar.BackgroundColor3 = Color3.fromRGB(0, 200, 255)
staminaBar.BorderSizePixel = 0
staminaBar.Parent = staminaFrame

local staminaLabel = Instance.new("TextLabel")
staminaLabel.Size = UDim2.new(1, 0, 1, 0)
staminaLabel.BackgroundTransparency = 1
staminaLabel.Text = "Stamina"
staminaLabel.TextColor3 = Color3.new(1, 1, 1)
staminaLabel.Font = Enum.Font.GothamBold
staminaLabel.TextSize = 14
staminaLabel.Parent = staminaFrame

-- ตรรกะ Stamina
local maxStamina = 100
local stamina = maxStamina
local drainRate = 25
local regenRate = 15
local isSprinting = false

RunService.Heartbeat:Connect(function(dt)
    if isSprinting and humanoid.MoveDirection.Magnitude > 0 then
        stamina = math.max(0, stamina - drainRate * dt)
        if stamina <= 0 then
            isSprinting = false
            humanoid.WalkSpeed = 16
        end
    else
        stamina = math.min(maxStamina, stamina + regenRate * dt)
    end
    
    -- อัพเดต UI
    local pct = stamina / maxStamina
    staminaBar.Size = UDim2.new(pct, 0, 1, 0)
    staminaLabel.Text = "Stamina: " .. math.floor(stamina) .. "/" .. maxStamina
    
    -- เปลี่ยนสี
    if pct > 0.5 then
        staminaBar.BackgroundColor3 = Color3.fromRGB(0, 200, 255)
    elseif pct > 0.25 then
        staminaBar.BackgroundColor3 = Color3.fromRGB(255, 200, 0)
    else
        staminaBar.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
    end
end)

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.LeftShift then
        if stamina > 10 then
            isSprinting = true
            humanoid.WalkSpeed = 32
        end
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.LeftShift then
        isSprinting = false
        humanoid.WalkSpeed = 16
    end
end)
```

---

## 27.13 สรุป

ในบทนี้เราได้เรียนรู้:

1. **UserInputService** - การรับ Input จากผู้เล่น
2. **ContextActionService** - การ Bind Actions กับปุ่ม
3. **ระบบ Sprint** - พร้อม Stamina System
4. **Double Jump** - การกระโดดสองครั้ง
5. **Dash System** - การ Dash ไปข้างหน้า
6. **Wall Jump** - การกระโดดจากผนัง
7. **Slide System** - การ Slide
8. **MoveTo** - การเดินไปยังจุดหมาย
9. **Pathfinding** - การหาเส้นทาง
10. **Flight System** - การบิน
11. **Mobile Controls** - การควบคุมบน Mobile
12. **Anti-Cheat** - การตรวจสอบการโกง

ในบทต่อไป เราจะเรียนรู้เกี่ยวกับระบบ Camera ของ Roblox

---

## แหล่งอ้างอิง

- [Roblox Developer Hub - UserInputService](https://developer.roblox.com/en-us/api-reference/class/UserInputService)
- [Roblox Developer Hub - Humanoid](https://developer.roblox.com/en-us/api-reference/class/Humanoid)
- [Roblox Developer Hub - PathfindingService](https://developer.roblox.com/en-us/api-reference/class/PathfindingService)
