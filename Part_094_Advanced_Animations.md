# Part 94: Advanced Animations - ระบบ Animation ขั้นสูง

## บทนำ

ระบบ Animation ที่ดีทำให้เกมรู้สึกมีชีวิตชีวาและน่าเล่นมากขึ้น ในบทนี้เราจะเรียนรู้การสร้างระบบ animation ขั้นสูงด้วย AnimationController, Animator API, และการ blend animations

---

## ส่วนที่ 1: พื้นฐาน Animator API

### 1.1 โครงสร้าง Animation System

```lua
-- ModuleScript: AnimationSystem (ReplicatedStorage)
-- ระบบจัดการ animations

local AnimationSystem = {}

-- Animation IDs
local ANIMATIONS = {
    -- Movement
    idle = "rbxassetid://507766388",
    walk = "rbxassetid://507777826",
    run = "rbxassetid://507767714",
    jump = "rbxassetid://507765000",
    fall = "rbxassetid://507767968",
    land = "rbxassetid://507767312",
    
    -- Combat
    attack_light = "rbxassetid://507771019",
    attack_heavy = "rbxassetid://507771977",
    attack_combo = "rbxassetid://507773965",
    block = "rbxassetid://507775166",
    dodge = "rbxassetid://507768073",
    
    -- Hit reactions
    hit_front = "rbxassetid://507769692",
    hit_back = "rbxassetid://507770453",
    knockback = "rbxassetid://507769357",
    
    -- Special
    cast_spell = "rbxassetid://507771818",
    drink_potion = "rbxassetid://507772104",
    pickup = "rbxassetid://507771777",
    
    -- Death/Respawn
    death = "rbxassetid://507773311",
    respawn = "rbxassetid://507773311",
    
    -- Social
    wave = "rbxassetid://507770548",
    dance = "rbxassetid://507771019",
    sit = "rbxassetid://507766388",
    
    -- Victory/Defeat
    victory = "rbxassetid://507770453",
    defeat = "rbxassetid://507771357",
}

-- Properties ของแต่ละ animation
local ANIMATION_PROPS = {
    idle = {
        priority = Enum.AnimationPriority.Idle,
        looped = true,
        weight = 1,
    },
    walk = {
        priority = Enum.AnimationPriority.Movement,
        looped = true,
        weight = 1,
    },
    run = {
        priority = Enum.AnimationPriority.Movement,
        looped = true,
        weight = 1,
    },
    attack_light = {
        priority = Enum.AnimationPriority.Action,
        looped = false,
        weight = 1,
        fadeTime = 0.1,
    },
    attack_heavy = {
        priority = Enum.AnimationPriority.Action,
        looped = false,
        weight = 1,
        fadeTime = 0.15,
    },
    death = {
        priority = Enum.AnimationPriority.Action4,
        looped = false,
        weight = 1,
    },
    cast_spell = {
        priority = Enum.AnimationPriority.Action2,
        looped = false,
        weight = 1,
    },
}

return {
    IDs = ANIMATIONS,
    Props = ANIMATION_PROPS,
}
```

### 1.2 Animation Player

```lua
-- ModuleScript: AnimationPlayer (ReplicatedStorage)
-- ตัวจัดการ play animations

local AnimationConfig = require(game.ReplicatedStorage.AnimationSystem)

local AnimationPlayer = {}
AnimationPlayer.__index = AnimationPlayer

-- สร้าง AnimationPlayer ใหม่สำหรับ character
function AnimationPlayer.new(character)
    local self = setmetatable({}, AnimationPlayer)
    
    self.character = character
    self.humanoid = character:WaitForChild("Humanoid")
    self.animator = self.humanoid:WaitForChild("Animator")
    
    -- เก็บ animation tracks ที่โหลดแล้ว
    self.tracks = {}
    
    -- เก็บ track ที่กำลัง play อยู่
    self.activeTracks = {}
    
    -- โหลด animations ทั้งหมด
    self:_preloadAnimations()
    
    return self
end

-- โหลด animations ล่วงหน้า
function AnimationPlayer:_preloadAnimations()
    for name, id in pairs(AnimationConfig.IDs) do
        local animation = Instance.new("Animation")
        animation.AnimationId = id
        
        local track = self.animator:LoadAnimation(animation)
        
        -- ตั้งค่า properties
        local props = AnimationConfig.Props[name]
        if props then
            if props.priority then
                track.Priority = props.priority
            end
            if props.looped ~= nil then
                track.Looped = props.looped
            end
        end
        
        self.tracks[name] = track
    end
end

-- Play animation
function AnimationPlayer:Play(animName, fadeTime, weight, speed)
    local track = self.tracks[animName]
    if not track then
        warn("[AnimationPlayer] ไม่พบ animation: " .. animName)
        return nil
    end
    
    local props = AnimationConfig.Props[animName] or {}
    fadeTime = fadeTime or props.fadeTime or 0.1
    weight = weight or props.weight or 1
    speed = speed or 1
    
    track:Play(fadeTime, weight, speed)
    self.activeTracks[animName] = track
    
    return track
end

-- Stop animation
function AnimationPlayer:Stop(animName, fadeTime)
    local track = self.tracks[animName]
    if not track then return end
    
    track:Stop(fadeTime or 0.1)
    self.activeTracks[animName] = nil
end

-- Stop ทุก animation
function AnimationPlayer:StopAll(fadeTime)
    for name, track in pairs(self.activeTracks) do
        track:Stop(fadeTime or 0.1)
    end
    self.activeTracks = {}
end

-- ตรวจสอบว่า animation กำลัง play อยู่หรือไม่
function AnimationPlayer:IsPlaying(animName)
    local track = self.tracks[animName]
    return track and track.IsPlaying
end

-- Play และรอจนจบ
function AnimationPlayer:PlayAndWait(animName, fadeTime)
    local track = self:Play(animName, fadeTime)
    if not track then return end
    
    if not track.Looped then
        track.Stopped:Wait()
    end
end

-- Destroy
function AnimationPlayer:Destroy()
    for _, track in pairs(self.tracks) do
        if track then
            track:Stop()
            track:Destroy()
        end
    end
    self.tracks = {}
    self.activeTracks = {}
end

return AnimationPlayer
```

---

## ส่วนที่ 2: Animation State Machine

```lua
-- ModuleScript: AnimationStateMachine (ReplicatedStorage)
-- State machine สำหรับ animations

local AnimationPlayer = require(game.ReplicatedStorage.AnimationPlayer)
local RunService = game:GetService("RunService")

local AnimationStateMachine = {}
AnimationStateMachine.__index = AnimationStateMachine

-- States
local STATES = {
    IDLE = "idle",
    WALKING = "walking",
    RUNNING = "running",
    JUMPING = "jumping",
    FALLING = "falling",
    ATTACKING = "attacking",
    BLOCKING = "blocking",
    CASTING = "casting",
    DYING = "dying",
    STUNNED = "stunned",
}

-- Transitions
local TRANSITIONS = {
    [STATES.IDLE] = {
        STATES.WALKING,
        STATES.RUNNING,
        STATES.JUMPING,
        STATES.ATTACKING,
        STATES.CASTING,
        STATES.DYING,
    },
    [STATES.WALKING] = {
        STATES.IDLE,
        STATES.RUNNING,
        STATES.JUMPING,
        STATES.ATTACKING,
        STATES.DYING,
    },
    [STATES.RUNNING] = {
        STATES.IDLE,
        STATES.WALKING,
        STATES.JUMPING,
        STATES.ATTACKING,
        STATES.DYING,
    },
    [STATES.JUMPING] = {
        STATES.FALLING,
        STATES.IDLE,
    },
    [STATES.FALLING] = {
        STATES.IDLE,
        STATES.WALKING,
        STATES.RUNNING,
    },
    [STATES.ATTACKING] = {
        STATES.IDLE,
        STATES.WALKING,
        STATES.DYING,
        STATES.STUNNED,
    },
    [STATES.BLOCKING] = {
        STATES.IDLE,
        STATES.WALKING,
        STATES.DYING,
        STATES.STUNNED,
    },
    [STATES.DYING] = {}, -- Terminal state
    [STATES.STUNNED] = {
        STATES.IDLE,
        STATES.DYING,
    },
}

-- Animation ที่ใช้ใน state
local STATE_ANIMATIONS = {
    [STATES.IDLE] = "idle",
    [STATES.WALKING] = "walk",
    [STATES.RUNNING] = "run",
    [STATES.JUMPING] = "jump",
    [STATES.FALLING] = "fall",
    [STATES.ATTACKING] = nil, -- handle manually
    [STATES.BLOCKING] = "block",
    [STATES.CASTING] = "cast_spell",
    [STATES.DYING] = "death",
    [STATES.STUNNED] = "hit_front",
}

function AnimationStateMachine.new(character)
    local self = setmetatable({}, AnimationStateMachine)
    
    self.character = character
    self.humanoid = character:WaitForChild("Humanoid")
    self.animPlayer = AnimationPlayer.new(character)
    
    self.currentState = STATES.IDLE
    self.previousState = nil
    self.stateData = {}
    
    -- Callbacks
    self.onStateEnter = {}
    self.onStateExit = {}
    
    -- เริ่มต้น
    self:_startStateLoop()
    
    return self
end

-- ตรวจสอบว่า transition valid หรือไม่
function AnimationStateMachine:_canTransition(toState)
    local allowed = TRANSITIONS[self.currentState]
    if not allowed then return false end
    return table.find(allowed, toState) ~= nil
end

-- เปลี่ยน state
function AnimationStateMachine:TransitionTo(newState, data)
    if newState == self.currentState then return false end
    if not self:_canTransition(newState) then
        warn(string.format("[AnimSM] ไม่สามารถ transition จาก %s ไป %s", 
            self.currentState, newState
        ))
        return false
    end
    
    -- Call exit callback
    if self.onStateExit[self.currentState] then
        self.onStateExit[self.currentState](self, data)
    end
    
    -- Stop current animation
    local currentAnim = STATE_ANIMATIONS[self.currentState]
    if currentAnim then
        self.animPlayer:Stop(currentAnim)
    end
    
    self.previousState = self.currentState
    self.currentState = newState
    self.stateData = data or {}
    
    -- Start new animation
    local newAnim = STATE_ANIMATIONS[newState]
    if newAnim then
        self.animPlayer:Play(newAnim)
    end
    
    -- Call enter callback
    if self.onStateEnter[newState] then
        self.onStateEnter[newState](self, data)
    end
    
    return true
end

-- Auto update state จาก humanoid
function AnimationStateMachine:_startStateLoop()
    local humanoid = self.humanoid
    
    -- ตรวจสอบ movement
    humanoid.Running:Connect(function(speed)
        if self.currentState == STATES.DYING then return end
        if self.currentState == STATES.ATTACKING then return end
        
        if speed < 0.1 then
            self:TransitionTo(STATES.IDLE)
        elseif speed < 14 then
            self:TransitionTo(STATES.WALKING)
        else
            self:TransitionTo(STATES.RUNNING)
        end
    end)
    
    -- ตรวจสอบ jump
    humanoid.Jumping:Connect(function(isJumping)
        if isJumping then
            self:TransitionTo(STATES.JUMPING)
        end
    end)
    
    -- ตรวจสอบ fall
    humanoid:GetPropertyChangedSignal("FloorMaterial"):Connect(function()
        if humanoid.FloorMaterial == Enum.Material.Air then
            if self.currentState == STATES.JUMPING then
                task.delay(0.3, function()
                    if self.currentState == STATES.JUMPING then
                        self:TransitionTo(STATES.FALLING)
                    end
                end)
            end
        else
            if self.currentState == STATES.FALLING or self.currentState == STATES.JUMPING then
                self:TransitionTo(STATES.IDLE)
                -- Play land animation
                self.animPlayer:Play("land")
                task.delay(0.3, function()
                    self.animPlayer:Stop("land")
                end)
            end
        end
    end)
    
    -- ตรวจสอบ death
    humanoid.Died:Connect(function()
        self:TransitionTo(STATES.DYING)
    end)
end

-- Attack
function AnimationStateMachine:StartAttack(attackType)
    if not self:TransitionTo(STATES.ATTACKING) then return end
    
    local animName = attackType or "attack_light"
    local track = self.animPlayer:Play(animName)
    
    if track then
        track.Stopped:Connect(function()
            if self.currentState == STATES.ATTACKING then
                self:TransitionTo(STATES.IDLE)
            end
        end)
    end
end

-- Callback registration
function AnimationStateMachine:OnEnter(state, callback)
    self.onStateEnter[state] = callback
end

function AnimationStateMachine:OnExit(state, callback)
    self.onStateExit[state] = callback
end

AnimationStateMachine.States = STATES

return AnimationStateMachine
```

---

## ส่วนที่ 3: Procedural Animation

```lua
-- LocalScript: ProceduralHead
-- Animation หัวตามกล้อง

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()

local humanoid = character:WaitForChild("Humanoid")
local rootPart = character:WaitForChild("HumanoidRootPart")
local head = character:WaitForChild("Head")
local neck = character:WaitForChild("Torso") and 
             character.Torso:WaitForChild("Neck")

if not neck then
    -- R15 rig
    neck = character:WaitForChild("Head"):FindFirstChild("Neck") or
           character:WaitForChild("UpperTorso"):FindFirstChild("Neck")
end

local originalC0 = neck and neck.C0

-- หมุนหัวตาม camera
RunService.Heartbeat:Connect(function()
    if not neck or not originalC0 then return end
    if humanoid:GetState() == Enum.HumanoidStateType.Dead then return end
    
    local camera = workspace.CurrentCamera
    local cameraLookVector = camera.CFrame.LookVector
    
    -- คำนวณมุมที่หัวควรหัน
    local characterLookVector = rootPart.CFrame.LookVector
    local dot = characterLookVector:Dot(cameraLookVector)
    
    -- Pitch (ก้มเงย)
    local pitch = math.asin(math.clamp(cameraLookVector.Y, -1, 1))
    pitch = math.clamp(pitch, math.rad(-70), math.rad(70))
    
    -- Yaw (ซ้ายขวา)
    local cross = characterLookVector:Cross(cameraLookVector)
    local yaw = math.asin(math.clamp(cross.Y, -1, 1))
    yaw = math.clamp(yaw, math.rad(-70), math.rad(70))
    
    -- ใช้ CFrame rotation
    neck.C0 = originalC0 * CFrame.Angles(pitch, yaw, 0)
end)
```

### 3.2 IK (Inverse Kinematics) System

```lua
-- ModuleScript: SimpleIK (ReplicatedStorage)
-- ระบบ IK อย่างง่าย

local SimpleIK = {}

-- คำนวณ IK สำหรับ 2 joints (เช่น แขน/ขา)
function SimpleIK:TwoJoint(root, mid, tip, target, pole)
    local rootPos = root.WorldPosition
    local tipPos = target
    
    -- ความยาวของ segments
    local upperLength = (mid.WorldPosition - rootPos).Magnitude
    local lowerLength = (tip.WorldPosition - mid.WorldPosition).Magnitude
    
    local targetDistance = math.min(
        (tipPos - rootPos).Magnitude,
        upperLength + lowerLength - 0.001
    )
    
    -- คำนวณ angle
    local cosAngle = (upperLength^2 + targetDistance^2 - lowerLength^2) / 
                     (2 * upperLength * targetDistance)
    cosAngle = math.clamp(cosAngle, -1, 1)
    local angle = math.acos(cosAngle)
    
    -- Direction ไปยัง target
    local dirToTarget = (tipPos - rootPos).Unit
    
    -- Pole vector (ทิศทาง bend)
    local poleDir = Vector3.new(0, 1, 0) -- Default up
    if pole then
        poleDir = (pole - rootPos).Unit
    end
    
    -- คำนวณ bend plane
    local bendDir = (dirToTarget:Cross(poleDir)).Unit
    local bendNormal = (dirToTarget:Cross(bendDir)).Unit
    
    -- Position ของ mid joint
    local midDir = dirToTarget * math.cos(angle) + bendNormal * math.sin(angle)
    local midPos = rootPos + midDir * upperLength
    
    return midPos, tipPos
end

-- Apply IK ไปยัง Motor6D
function SimpleIK:ApplyToMotor(motor6D, worldPos)
    -- แปลง world position เป็น local
    local parent = motor6D.Parent
    if not parent then return end
    
    local localPos = parent.CFrame:PointToObjectSpace(worldPos)
    
    -- อัพเดท C0
    motor6D.C0 = CFrame.new(localPos) * motor6D.C0.Rotation
end

return SimpleIK
```

---

## ส่วนที่ 4: Animation Events

```lua
-- LocalScript: AnimationEvents (StarterCharacterScripts)
-- ตรวจสอบ events ใน animation

local Players = game:GetService("Players")
local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local animator = humanoid:WaitForChild("Animator")

-- โหลด attack animation พร้อม events
local attackAnim = Instance.new("Animation")
attackAnim.AnimationId = "rbxassetid://507771019"

local attackTrack = animator:LoadAnimation(attackAnim)

-- เชื่อม events ใน animation
-- (Events ต้องตั้งค่าใน Animation Editor)
attackTrack:GetMarkerReachedSignal("Hit"):Connect(function()
    -- เมื่อ animation ถึงจุด "Hit"
    -- ตรวจสอบ hitbox และ deal damage
    print("Animation hit point!")
    
    -- ส่ง event ไปยัง server
    local remotes = game.ReplicatedStorage.Remotes
    local attackEvent = remotes:FindFirstChild("Attack")
    if attackEvent then
        attackEvent:FireServer("light")
    end
end)

attackTrack:GetMarkerReachedSignal("FootStep"):Connect(function()
    -- เล่นเสียงเดิน
    local soundPart = character:FindFirstChild("HumanoidRootPart")
    if soundPart then
        local footStepSound = soundPart:FindFirstChild("FootStep")
        if footStepSound then
            footStepSound:Play()
        end
    end
end)

-- Play animation
attackTrack:Play()
```

---

## ส่วนที่ 5: Emote System

```lua
-- Script: EmoteSystem (Script ใน ServerScriptService)
-- ระบบ emote

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- Emotes ที่มี
local EMOTES = {
    wave = {
        id = "rbxassetid://507770548",
        name = "Wave",
        looped = false,
        unlockRequirement = nil, -- free
    },
    dance = {
        id = "rbxassetid://507771019",
        name = "Dance",
        looped = true,
        unlockRequirement = nil,
    },
    bow = {
        id = "rbxassetid://507768573",
        name = "Bow",
        looped = false,
        unlockRequirement = nil,
    },
    victory = {
        id = "rbxassetid://507770453",
        name = "Victory",
        looped = false,
        unlockRequirement = "VICTORY_DANCE_PASS",
    },
}

-- Play emote บน server (สำหรับ replicate ให้คนอื่นเห็น)
local remotes = ReplicatedStorage:WaitForChild("Remotes")
local emoteRemote = Instance.new("RemoteEvent")
emoteRemote.Name = "PlayEmote"
emoteRemote.Parent = remotes

-- Client request emote
emoteRemote.OnServerEvent:Connect(function(player, emoteName)
    local emote = EMOTES[emoteName]
    if not emote then return end
    
    -- ตรวจสอบสิทธิ์
    if emote.unlockRequirement then
        local ProfileManager = require(game.ServerStorage.ProfileManager)
        local data = ProfileManager:GetData(player)
        if not data then return end
        
        if not data.Inventory or not data.Inventory[emote.unlockRequirement] then
            return -- ไม่มีสิทธิ์
        end
    end
    
    -- Replicate ไปยังผู้เล่นอื่น
    for _, targetPlayer in ipairs(Players:GetPlayers()) do
        if targetPlayer ~= player then
            emoteRemote:FireClient(targetPlayer, player, emoteName, emote)
        end
    end
end)
```

---

## ส่วนที่ 6: Cutscene System

```lua
-- ModuleScript: CutsceneSystem (ReplicatedStorage)
-- ระบบ cutscene

local TweenService = game:GetService("TweenService")
local Players = game:GetService("Players")

local CutsceneSystem = {}

-- สร้าง cutscene
function CutsceneSystem:Play(scenes, onComplete)
    local player = Players.LocalPlayer
    local camera = workspace.CurrentCamera
    
    -- บันทึก camera mode เดิม
    local originalCameraType = camera.CameraType
    local originalCFrame = camera.CFrame
    
    -- เปลี่ยนเป็น Scriptable camera
    camera.CameraType = Enum.CameraType.Scriptable
    
    -- ปิด input ชั่วคราว
    local uis = game:GetService("UserInputService")
    -- (ปิด input ตามต้องการ)
    
    local function playNextScene(sceneIndex)
        if sceneIndex > #scenes then
            -- Cutscene จบ
            camera.CameraType = originalCameraType
            if onComplete then
                onComplete()
            end
            return
        end
        
        local scene = scenes[sceneIndex]
        
        -- ย้าย camera ไปยัง target CFrame
        local tweenInfo = TweenInfo.new(
            scene.duration or 2,
            scene.easing or Enum.EasingStyle.Sine,
            scene.easingDir or Enum.EasingDirection.InOut
        )
        
        local targetCFrame = scene.cframe
        
        -- ถ้า scene มี target object ให้ดู
        if scene.target then
            local target = workspace:FindFirstChild(scene.target)
            if target then
                targetCFrame = CFrame.new(scene.position or camera.CFrame.Position, target.Position)
            end
        end
        
        local tween = TweenService:Create(camera, tweenInfo, {CFrame = targetCFrame})
        tween:Play()
        
        -- ถ้ามี animation ให้ play
        if scene.animation then
            -- play character animation
        end
        
        -- รอจบ scene
        tween.Completed:Connect(function()
            -- หน่วงเวลาถ้ามี
            if scene.holdTime then
                task.wait(scene.holdTime)
            end
            playNextScene(sceneIndex + 1)
        end)
    end
    
    playNextScene(1)
end

-- ตัวอย่างการใช้งาน
local function playIntroCutscene()
    CutsceneSystem:Play({
        -- Scene 1: ภาพรวมแผนที่
        {
            cframe = CFrame.new(0, 100, 0) * CFrame.Angles(math.rad(-90), 0, 0),
            duration = 3,
            holdTime = 1,
        },
        -- Scene 2: เข้าใกล้ castle
        {
            cframe = CFrame.new(50, 30, -100),
            duration = 4,
        },
        -- Scene 3: ซูมเข้า gate
        {
            cframe = CFrame.new(10, 5, -50),
            duration = 2,
        },
    }, function()
        print("Cutscene จบ!")
    end)
end

return CutsceneSystem
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Combat Animation System
สร้างระบบที่:
1. Combo attack ต่อเนื่อง
2. Cancel animation ด้วย dodge
3. Hit reaction ตามทิศทาง

### แบบฝึกหัดที่ 2: Social Emotes
สร้าง emote wheel ที่:
1. แสดง 8 emotes
2. ล็อค emotes พิเศษ
3. Sync กับผู้เล่นอื่น

### แบบฝึกหัดที่ 3: Intro Cutscene
สร้าง cutscene ความยาว 30 วินาทีที่:
1. แสดงแผนที่
2. มี NPC animation
3. Fade in/out

---

## สรุปบทที่ 94

ระบบ Animation ขั้นสูงต้องการ:

1. **State Machine** - จัดการ transitions อย่างเป็นระบบ
2. **Preloading** - โหลด animations ล่วงหน้า
3. **Events** - ใช้ animation markers สำหรับ gameplay
4. **Procedural** - head tracking, IK
5. **Cutscenes** - สร้าง cinematic moments

*บทถัดไป: Part 95 - Procedural Generation*
