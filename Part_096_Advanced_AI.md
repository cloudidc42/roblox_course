# Part 96: Advanced NPC AI และ Behavior Trees

## บทนำ (Introduction)

ในบทนี้เราจะเรียนรู้การสร้างระบบ AI ขั้นสูงสำหรับ NPC (Non-Player Character) ใน Roblox
ครอบคลุมตั้งแต่ Behavior Trees, PathfindingService, ระบบการมองเห็นและได้ยิน,
ไปจนถึง Boss AI ที่มีหลาย Phase และพฤติกรรมกลุ่ม

### สิ่งที่จะได้เรียนรู้:
- Behavior Tree architecture สำหรับ NPC decision-making
- PathfindingService และการเคลื่อนที่อัจฉริยะ
- ระบบ Perception (การมองเห็น/การได้ยิน)
- State Machine สำหรับ AI
- Group AI และการประสานงานกลุ่ม
- Boss AI หลาย Phase
- Performance optimization สำหรับ AI หลายตัว

---

## ส่วนที่ 1: พื้นฐาน AI Architecture

### 1.1 NPC Base Module

```lua
-- ModuleScript: NPCBase
-- ไฟล์พื้นฐานสำหรับ NPC ทุกประเภท
-- Base module for all NPC types

local NPCBase = {}
NPCBase.__index = NPCBase

-- Services ที่ใช้งาน (Required services)
local PathfindingService = game:GetService("PathfindingService")
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")

-- ค่าคงที่ (Constants)
local UPDATE_INTERVAL = 0.1  -- อัพเดททุก 0.1 วินาที (Update every 0.1 seconds)
local MAX_PATH_AGE = 2.0     -- Path หมดอายุใน 2 วินาที (Path expires in 2 seconds)

-- สร้าง NPC ใหม่ (Create new NPC)
function NPCBase.new(model, config)
    local self = setmetatable({}, NPCBase)
    
    -- อ้างอิงหลัก (Main references)
    self.Model = model
    self.Humanoid = model:FindFirstChildOfClass("Humanoid")
    self.HumanoidRootPart = model:FindFirstChild("HumanoidRootPart")
    self.Animator = self.Humanoid and self.Humanoid:FindFirstChildOfClass("Animator")
    
    -- การตั้งค่า (Configuration)
    self.Config = config or {}
    self.MaxHealth = config.MaxHealth or 100
    self.MoveSpeed = config.MoveSpeed or 16
    self.DetectionRange = config.DetectionRange or 30
    self.AttackRange = config.AttackRange or 5
    self.AttackDamage = config.AttackDamage or 10
    self.AttackCooldown = config.AttackCooldown or 1.5
    
    -- สถานะ AI (AI State)
    self.State = "Idle"
    self.Target = nil
    self.LastAttackTime = 0
    self.IsAlive = true
    
    -- Path information
    self.CurrentPath = nil
    self.PathAge = 0
    self.CurrentWaypoint = 1
    
    -- Animation tracks
    self.Animations = {}
    
    -- การตั้งค่า Humanoid
    if self.Humanoid then
        self.Humanoid.MaxHealth = self.MaxHealth
        self.Humanoid.Health = self.MaxHealth
        self.Humanoid.WalkSpeed = self.MoveSpeed
        
        -- ตรวจสอบการตาย (Handle death)
        self.Humanoid.Died:Connect(function()
            self:OnDeath()
        end)
    end
    
    return self
end

-- โหลด Animation (Load animations)
function NPCBase:LoadAnimations(animationIds)
    if not self.Animator then return end
    
    for name, id in pairs(animationIds) do
        local animation = Instance.new("Animation")
        animation.AnimationId = id
        self.Animations[name] = self.Animator:LoadAnimation(animation)
    end
end

-- เล่น Animation (Play animation)
function NPCBase:PlayAnimation(name, fadeTime)
    local track = self.Animations[name]
    if track and not track.IsPlaying then
        -- หยุด animation ปัจจุบัน (Stop current animation)
        for _, t in pairs(self.Animations) do
            if t.IsPlaying and t ~= track then
                t:Stop(fadeTime or 0.2)
            end
        end
        track:Play(fadeTime or 0.2)
    end
end

-- หาเส้นทาง (Find path to target position)
function NPCBase:FindPathTo(targetPosition)
    if not self.HumanoidRootPart then return nil end
    
    -- สร้าง PathfindingAgent parameters
    local agentParams = {
        AgentHeight = 5,
        AgentRadius = 2,
        AgentCanJump = true,
        AgentJumpHeight = 7.2,
        AgentMaxSlope = 45,
        WaypointSpacing = 4,
        Costs = {
            -- กำหนดต้นทุนสำหรับ material ต่างๆ
            Water = math.huge,  -- ไม่ข้ามน้ำ (Don't cross water)
            Grass = 1,
            Concrete = 1.5,
        }
    }
    
    local path = PathfindingService:CreatePath(agentParams)
    
    local success, err = pcall(function()
        path:ComputeAsync(
            self.HumanoidRootPart.Position,
            targetPosition
        )
    end)
    
    if not success then
        warn("Path computation failed:", err)
        return nil
    end
    
    if path.Status == Enum.PathStatus.Success then
        return path
    elseif path.Status == Enum.PathStatus.NoPath then
        -- ลองหาจุดใกล้ที่สุด (Try to find nearest reachable point)
        return nil
    end
    
    return nil
end

-- เดินตาม Path (Follow a computed path)
function NPCBase:FollowPath(path)
    if not path or not self.Humanoid or not self.HumanoidRootPart then return end
    
    local waypoints = path:GetWaypoints()
    if #waypoints == 0 then return end
    
    self.CurrentPath = waypoints
    self.CurrentWaypoint = 2  -- เริ่มจาก waypoint ที่ 2 (waypoint 1 คือตำแหน่งปัจจุบัน)
    
    -- ตรวจสอบ waypoints ที่ blocked (Check for blocked waypoints)
    path.Blocked:Connect(function(blockedWaypointIndex)
        if blockedWaypointIndex >= self.CurrentWaypoint then
            -- คำนวณ path ใหม่ (Recalculate path)
            self.CurrentPath = nil
        end
    end)
end

-- อัพเดทการเดิน (Update movement along path)
function NPCBase:UpdateMovement()
    if not self.CurrentPath or not self.HumanoidRootPart then return end
    
    local waypoints = self.CurrentPath
    
    if self.CurrentWaypoint > #waypoints then
        -- ถึงปลายทางแล้ว (Reached destination)
        self.CurrentPath = nil
        self.Humanoid:Move(Vector3.zero)
        return
    end
    
    local waypoint = waypoints[self.CurrentWaypoint]
    local distance = (self.HumanoidRootPart.Position - waypoint.Position).Magnitude
    
    if distance < 3 then
        -- ถึง waypoint ถัดไป (Reached next waypoint)
        self.CurrentWaypoint = self.CurrentWaypoint + 1
        
        -- ตรวจสอบว่าต้องกระโดดหรือไม่ (Check if jump is needed)
        if waypoint.Action == Enum.PathWaypointAction.Jump then
            self.Humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
        end
    else
        -- เดินไปยัง waypoint (Move toward waypoint)
        self.Humanoid:MoveTo(waypoint.Position)
    end
end

-- ตรวจสอบว่าเห็น target หรือไม่ (Check line of sight to target)
function NPCBase:HasLineOfSight(targetPosition)
    if not self.HumanoidRootPart then return false end
    
    local origin = self.HumanoidRootPart.Position + Vector3.new(0, 2, 0)
    local direction = (targetPosition - origin)
    
    -- RaycastParams เพื่อไม่นับ NPC เอง (Ignore self in raycast)
    local raycastParams = RaycastParams.new()
    raycastParams.FilterDescendantsInstances = {self.Model}
    raycastParams.FilterType = Enum.RaycastFilterType.Exclude
    
    local result = workspace:Raycast(origin, direction, raycastParams)
    
    if not result then
        return true  -- ไม่มีสิ่งกีดขวาง (No obstruction)
    end
    
    -- ตรวจสอบว่าโดน target หรือ obstacle (Check if hit target or obstacle)
    local hitModel = result.Instance:FindFirstAncestorOfClass("Model")
    if hitModel and hitModel:FindFirstChild("Humanoid") then
        return true  -- โดน Character ของ target
    end
    
    return false
end

-- รับ damage (Take damage)
function NPCBase:TakeDamage(amount, attacker)
    if not self.IsAlive or not self.Humanoid then return end
    
    self.Humanoid:TakeDamage(amount)
    
    -- ถ้ายังไม่มี target และถูกโจมตี ให้หันมาสู้ (If no target, fight back)
    if not self.Target and attacker then
        self.Target = attacker
        self:SetState("Chase")
    end
end

-- ตายแล้ว (On death callback)
function NPCBase:OnDeath()
    self.IsAlive = false
    self.State = "Dead"
    self.Target = nil
    self.CurrentPath = nil
    
    -- Ragdoll effect
    if self.Humanoid then
        self.Humanoid.PlatformStand = true
    end
    
    -- ลบหลังจาก 5 วินาที (Remove after 5 seconds)
    task.delay(5, function()
        if self.Model and self.Model.Parent then
            self.Model:Destroy()
        end
    end)
end

-- กำหนดสถานะ (Set state)
function NPCBase:SetState(newState)
    if self.State == newState then return end
    
    local oldState = self.State
    self.State = newState
    
    -- เรียก callback เมื่อเปลี่ยนสถานะ (Call state change callbacks)
    if self.OnStateChange then
        self.OnStateChange(oldState, newState)
    end
end

-- อัพเดทหลัก (Main update function)
function NPCBase:Update(dt)
    if not self.IsAlive then return end
    
    -- ให้ subclass override (Let subclass override)
end

-- เริ่มระบบ AI (Start AI system)
function NPCBase:Start()
    self._connection = RunService.Heartbeat:Connect(function(dt)
        self:Update(dt)
        self:UpdateMovement()
    end)
end

-- หยุดระบบ AI (Stop AI system)
function NPCBase:Stop()
    if self._connection then
        self._connection:Disconnect()
        self._connection = nil
    end
end

-- ทำลาย NPC (Destroy NPC)
function NPCBase:Destroy()
    self:Stop()
    if self.Model and self.Model.Parent then
        self.Model:Destroy()
    end
end

return NPCBase
```

### 1.2 Perception System (ระบบการรับรู้)

```lua
-- ModuleScript: PerceptionSystem
-- ระบบการมองเห็นและการได้ยินสำหรับ NPC
-- Vision and hearing system for NPCs

local PerceptionSystem = {}
PerceptionSystem.__index = PerceptionSystem

local Players = game:GetService("Players")

-- สร้าง Perception System (Create perception system)
function PerceptionSystem.new(npc, config)
    local self = setmetatable({}, PerceptionSystem)
    
    self.NPC = npc
    
    -- การตั้งค่าการมองเห็น (Vision settings)
    self.VisionRange = config.VisionRange or 50
    self.VisionAngle = config.VisionAngle or 90  -- องศา (degrees) - ครึ่งวงกลมด้านหน้า
    self.NightVisionMultiplier = config.NightVisionMultiplier or 0.5
    
    -- การตั้งค่าการได้ยิน (Hearing settings)
    self.HearingRange = config.HearingRange or 20
    self.AlertHearingRange = config.AlertHearingRange or 35  -- เมื่อตื่นตัว (When alert)
    
    -- สถานะการรับรู้ (Perception state)
    self.KnownTargets = {}  -- {player = {lastSeenPosition, lastSeenTime, threat}}
    self.AlertLevel = 0     -- 0=calm, 1=suspicious, 2=alert, 3=combat
    
    -- Memory
    self.MemoryDuration = config.MemoryDuration or 10  -- จำเป้าหมายได้กี่วินาที
    
    return self
end

-- ตรวจสอบว่าเห็นผู้เล่นหรือไม่ (Check if player is visible)
function PerceptionSystem:CanSeeTarget(targetCharacter)
    local npcRoot = self.NPC.HumanoidRootPart
    if not npcRoot or not targetCharacter then return false end
    
    local targetRoot = targetCharacter:FindFirstChild("HumanoidRootPart")
    if not targetRoot then return false end
    
    local toTarget = targetRoot.Position - npcRoot.Position
    local distance = toTarget.Magnitude
    
    -- ตรวจสอบระยะ (Check distance)
    if distance > self.VisionRange then return false end
    
    -- ตรวจสอบมุม (Check angle)
    local npcLook = npcRoot.CFrame.LookVector
    local toTargetNorm = toTarget.Unit
    
    local dot = npcLook:Dot(toTargetNorm)
    local angleRad = math.acos(math.clamp(dot, -1, 1))
    local angleDeg = math.deg(angleRad)
    
    if angleDeg > self.VisionAngle then return false end
    
    -- ตรวจสอบเส้นตรง (Check line of sight)
    return self.NPC:HasLineOfSight(targetRoot.Position)
end

-- ตรวจสอบว่าได้ยินเสียงหรือไม่ (Check if sound is heard)
function PerceptionSystem:CanHearTarget(targetCharacter)
    local npcRoot = self.NPC.HumanoidRootPart
    if not npcRoot or not targetCharacter then return false end
    
    local targetRoot = targetCharacter:FindFirstChild("HumanoidRootPart")
    if not targetRoot then return false end
    
    local distance = (targetRoot.Position - npcRoot.Position).Magnitude
    
    -- ระยะการได้ยินขึ้นอยู่กับสถานะ (Hearing range depends on alert level)
    local hearingRange = self.AlertLevel >= 2 
        and self.AlertHearingRange 
        or self.HearingRange
    
    -- ตรวจสอบว่าผู้เล่นกำลังวิ่งหรือเดิน (Check if player is running)
    local targetHumanoid = targetCharacter:FindFirstChildOfClass("Humanoid")
    if targetHumanoid then
        local speed = targetHumanoid.MoveDirection.Magnitude * targetHumanoid.WalkSpeed
        if speed > 10 then
            hearingRange = hearingRange * 1.5  -- วิ่งเสียงดัง (Running is louder)
        elseif speed < 1 then
            hearingRange = hearingRange * 0.3  -- เดินช้าเสียงเบา (Sneaking is quiet)
        end
    end
    
    return distance <= hearingRange
end

-- อัพเดท Perception (Update perception)
function PerceptionSystem:Update()
    local currentTime = tick()
    local detected = false
    
    -- ตรวจสอบผู้เล่นทุกคน (Check all players)
    for _, player in ipairs(Players:GetPlayers()) do
        local character = player.Character
        if not character then continue end
        
        local canSee = self:CanSeeTarget(character)
        local canHear = self:CanHearTarget(character)
        
        if canSee or canHear then
            -- อัพเดทข้อมูล target (Update target info)
            local targetRoot = character:FindFirstChild("HumanoidRootPart")
            if targetRoot then
                self.KnownTargets[player] = {
                    character = character,
                    lastSeenPosition = targetRoot.Position,
                    lastSeenTime = currentTime,
                    isVisible = canSee,
                    threat = 1.0
                }
                detected = true
                
                -- เพิ่ม alert level (Increase alert level)
                if canSee then
                    self.AlertLevel = math.min(3, self.AlertLevel + 0.5)
                elseif canHear then
                    self.AlertLevel = math.min(2, self.AlertLevel + 0.2)
                end
            end
        end
    end
    
    -- ลด alert level เมื่อไม่เห็น target (Decrease alert when no target)
    if not detected then
        self.AlertLevel = math.max(0, self.AlertLevel - 0.05)
    end
    
    -- ลบ target ที่หมด memory (Remove expired memories)
    for player, info in pairs(self.KnownTargets) do
        if currentTime - info.lastSeenTime > self.MemoryDuration then
            self.KnownTargets[player] = nil
        end
    end
end

-- หา target ที่อันตรายที่สุด (Find most threatening target)
function PerceptionSystem:GetPriorityTarget()
    local bestTarget = nil
    local bestThreat = 0
    local npcRoot = self.NPC.HumanoidRootPart
    
    for player, info in pairs(self.KnownTargets) do
        local threat = info.threat
        
        -- ปรับ threat ตามระยะทาง (Adjust threat by distance)
        if npcRoot and info.character then
            local targetRoot = info.character:FindFirstChild("HumanoidRootPart")
            if targetRoot then
                local distance = (targetRoot.Position - npcRoot.Position).Magnitude
                threat = threat * (1 / math.max(1, distance / 10))
            end
        end
        
        -- เพิ่ม threat ถ้า visible (Increase threat if visible)
        if info.isVisible then
            threat = threat * 1.5
        end
        
        if threat > bestThreat then
            bestThreat = threat
            bestTarget = info.character
        end
    end
    
    return bestTarget
end

return PerceptionSystem
```

---

## ส่วนที่ 2: Behavior Tree System

### 2.1 Behavior Tree Core

```lua
-- ModuleScript: BehaviorTree
-- ระบบ Behavior Tree สำหรับ AI decision making
-- Behavior Tree system for AI decision making

local BehaviorTree = {}
BehaviorTree.__index = BehaviorTree

-- ผลลัพธ์ของ node (Node results)
local Status = {
    SUCCESS = "Success",   -- สำเร็จ (Succeeded)
    FAILURE = "Failure",   -- ล้มเหลว (Failed)
    RUNNING = "Running",   -- กำลังทำงาน (Still running)
}

BehaviorTree.Status = Status

-- ==================== BASE NODE ====================
local BaseNode = {}
BaseNode.__index = BaseNode

function BaseNode.new(name)
    return setmetatable({
        Name = name or "Node",
        Children = {},
    }, BaseNode)
end

function BaseNode:AddChild(child)
    table.insert(self.Children, child)
    return self
end

function BaseNode:Execute(blackboard)
    return Status.FAILURE
end

-- ==================== SEQUENCE NODE ====================
-- ทำงานทุก child ตามลำดับ ถ้า child ใด FAIL ให้ FAIL ทั้งหมด
-- Run all children in order, if any child FAILS, the whole sequence FAILS
local SequenceNode = setmetatable({}, {__index = BaseNode})
SequenceNode.__index = SequenceNode

function SequenceNode.new(name)
    local self = setmetatable(BaseNode.new(name), SequenceNode)
    self._currentChild = 1
    return self
end

function SequenceNode:Execute(blackboard)
    while self._currentChild <= #self.Children do
        local child = self.Children[self._currentChild]
        local result = child:Execute(blackboard)
        
        if result == Status.FAILURE then
            self._currentChild = 1  -- reset
            return Status.FAILURE
        elseif result == Status.RUNNING then
            return Status.RUNNING
        end
        
        -- SUCCESS: ไปต่อ child ถัดไป (Continue to next child)
        self._currentChild = self._currentChild + 1
    end
    
    self._currentChild = 1  -- reset
    return Status.SUCCESS
end

-- ==================== SELECTOR NODE ====================
-- ลอง child ทีละตัว ถ้า child ใด SUCCESS ให้ SUCCESS ทั้งหมด
-- Try each child, if any child SUCCEEDS, the selector SUCCEEDS
local SelectorNode = setmetatable({}, {__index = BaseNode})
SelectorNode.__index = SelectorNode

function SelectorNode.new(name)
    local self = setmetatable(BaseNode.new(name), SelectorNode)
    self._currentChild = 1
    return self
end

function SelectorNode:Execute(blackboard)
    while self._currentChild <= #self.Children do
        local child = self.Children[self._currentChild]
        local result = child:Execute(blackboard)
        
        if result == Status.SUCCESS then
            self._currentChild = 1  -- reset
            return Status.SUCCESS
        elseif result == Status.RUNNING then
            return Status.RUNNING
        end
        
        -- FAILURE: ลอง child ถัดไป (Try next child)
        self._currentChild = self._currentChild + 1
    end
    
    self._currentChild = 1  -- reset
    return Status.FAILURE
end

-- ==================== PARALLEL NODE ====================
-- ทำทุก child พร้อมกัน SUCCESS ถ้า >= minSuccess child สำเร็จ
-- Run all children simultaneously, SUCCESS if >= minSuccess succeed
local ParallelNode = setmetatable({}, {__index = BaseNode})
ParallelNode.__index = ParallelNode

function ParallelNode.new(name, minSuccess)
    local self = setmetatable(BaseNode.new(name), ParallelNode)
    self.MinSuccess = minSuccess or 1
    return self
end

function ParallelNode:Execute(blackboard)
    local successCount = 0
    local failureCount = 0
    
    for _, child in ipairs(self.Children) do
        local result = child:Execute(blackboard)
        
        if result == Status.SUCCESS then
            successCount = successCount + 1
        elseif result == Status.FAILURE then
            failureCount = failureCount + 1
        end
    end
    
    if successCount >= self.MinSuccess then
        return Status.SUCCESS
    elseif failureCount > #self.Children - self.MinSuccess then
        return Status.FAILURE
    end
    
    return Status.RUNNING
end

-- ==================== DECORATOR NODES ====================
-- Inverter: กลับผลลัพธ์ (Invert the result)
local InverterNode = setmetatable({}, {__index = BaseNode})
InverterNode.__index = InverterNode

function InverterNode.new(name, child)
    local self = setmetatable(BaseNode.new(name), InverterNode)
    self.Child = child
    return self
end

function InverterNode:Execute(blackboard)
    local result = self.Child:Execute(blackboard)
    
    if result == Status.SUCCESS then return Status.FAILURE
    elseif result == Status.FAILURE then return Status.SUCCESS
    else return Status.RUNNING end
end

-- Repeater: ทำซ้ำ N ครั้ง หรือตลอดไป (Repeat N times or forever)
local RepeaterNode = setmetatable({}, {__index = BaseNode})
RepeaterNode.__index = RepeaterNode

function RepeaterNode.new(name, child, times)
    local self = setmetatable(BaseNode.new(name), RepeaterNode)
    self.Child = child
    self.Times = times  -- nil = ทำตลอดไป (forever)
    self._count = 0
    return self
end

function RepeaterNode:Execute(blackboard)
    local result = self.Child:Execute(blackboard)
    
    if result ~= Status.RUNNING then
        self._count = self._count + 1
        
        if self.Times and self._count >= self.Times then
            self._count = 0
            return Status.SUCCESS
        end
    end
    
    return Status.RUNNING
end

-- ==================== ACTION NODES ====================
-- Action node: leaf node ที่ทำการกระทำจริง (Leaf node that performs actual actions)
local ActionNode = setmetatable({}, {__index = BaseNode})
ActionNode.__index = ActionNode

function ActionNode.new(name, actionFunc)
    local self = setmetatable(BaseNode.new(name), ActionNode)
    self.Action = actionFunc
    return self
end

function ActionNode:Execute(blackboard)
    if self.Action then
        return self.Action(blackboard)
    end
    return Status.FAILURE
end

-- ==================== CONDITION NODES ====================
-- Condition node: ตรวจสอบเงื่อนไข (Check conditions)
local ConditionNode = setmetatable({}, {__index = BaseNode})
ConditionNode.__index = ConditionNode

function ConditionNode.new(name, conditionFunc)
    local self = setmetatable(BaseNode.new(name), ConditionNode)
    self.Condition = conditionFunc
    return self
end

function ConditionNode:Execute(blackboard)
    if self.Condition and self.Condition(blackboard) then
        return Status.SUCCESS
    end
    return Status.FAILURE
end

-- ==================== BEHAVIOR TREE MAIN ====================
function BehaviorTree.new(rootNode)
    local self = setmetatable({}, BehaviorTree)
    self.Root = rootNode
    self.Blackboard = {}  -- ข้อมูลที่แชร์ระหว่าง nodes (Shared data between nodes)
    return self
end

function BehaviorTree:Tick()
    if self.Root then
        return self.Root:Execute(self.Blackboard)
    end
    return Status.FAILURE
end

-- Factory functions สำหรับสร้าง nodes ง่ายๆ (Factory functions for creating nodes)
BehaviorTree.Sequence = function(name, ...) 
    local node = SequenceNode.new(name)
    for _, child in ipairs({...}) do
        node:AddChild(child)
    end
    return node
end

BehaviorTree.Selector = function(name, ...)
    local node = SelectorNode.new(name)
    for _, child in ipairs({...}) do
        node:AddChild(child)
    end
    return node
end

BehaviorTree.Parallel = function(name, minSuccess, ...)
    local node = ParallelNode.new(name, minSuccess)
    for _, child in ipairs({...}) do
        node:AddChild(child)
    end
    return node
end

BehaviorTree.Action = function(name, func)
    return ActionNode.new(name, func)
end

BehaviorTree.Condition = function(name, func)
    return ConditionNode.new(name, func)
end

BehaviorTree.Invert = function(name, child)
    return InverterNode.new(name, child)
end

BehaviorTree.Repeat = function(name, child, times)
    return RepeaterNode.new(name, child, times)
end

return BehaviorTree
```

### 2.2 ตัวอย่าง Guard NPC ด้วย Behavior Tree

```lua
-- Script: GuardNPC
-- NPC ยามที่ใช้ Behavior Tree
-- Guard NPC using Behavior Tree

local NPCBase = require(game.ServerScriptService.NPCBase)
local PerceptionSystem = require(game.ServerScriptService.PerceptionSystem)
local BehaviorTree = require(game.ServerScriptService.BehaviorTree)

local Status = BehaviorTree.Status

-- สร้าง Guard NPC
local function createGuard(model, patrolPoints)
    local guard = NPCBase.new(model, {
        MaxHealth = 150,
        MoveSpeed = 14,
        DetectionRange = 40,
        AttackRange = 6,
        AttackDamage = 15,
        AttackCooldown = 1.5,
    })
    
    -- โหลด animations
    guard:LoadAnimations({
        Idle = "rbxassetid://507766666",
        Walk = "rbxassetid://507777826",
        Run = "rbxassetid://507767714",
        Attack = "rbxassetid://522635514",
        Alert = "rbxassetid://507770453",
    })
    
    -- เพิ่ม perception
    local perception = PerceptionSystem.new(guard, {
        VisionRange = 45,
        VisionAngle = 75,
        HearingRange = 25,
    })
    
    guard.Perception = perception
    guard.PatrolPoints = patrolPoints
    guard.CurrentPatrolIndex = 1
    guard.LastPatrolTime = 0
    guard.IsSearching = false
    guard.SearchPosition = nil
    guard.HomePosition = model.HumanoidRootPart and model.HumanoidRootPart.Position
    
    -- ==================== สร้าง Behavior Tree ====================
    local bt = BehaviorTree.new(
        BehaviorTree.Selector("Root",
            
            -- *** ลำดับความสำคัญที่ 1: Combat ***
            BehaviorTree.Sequence("CombatSequence",
                -- ตรวจสอบว่ามี target หรือไม่
                BehaviorTree.Condition("HasTarget", function(bb)
                    return bb.target ~= nil and bb.target.Parent ~= nil
                end),
                
                -- ตรวจสอบว่า target ยังมีชีวิต
                BehaviorTree.Condition("TargetAlive", function(bb)
                    local humanoid = bb.target:FindFirstChildOfClass("Humanoid")
                    return humanoid and humanoid.Health > 0
                end),
                
                -- Selector: โจมตีหรือไล่ตาม
                BehaviorTree.Selector("AttackOrChase",
                    -- โจมตีถ้าอยู่ในระยะ
                    BehaviorTree.Sequence("AttackSequence",
                        BehaviorTree.Condition("InAttackRange", function(bb)
                            local targetRoot = bb.target:FindFirstChild("HumanoidRootPart")
                            if not targetRoot then return false end
                            local dist = (guard.HumanoidRootPart.Position - targetRoot.Position).Magnitude
                            return dist <= guard.AttackRange
                        end),
                        BehaviorTree.Action("Attack", function(bb)
                            local now = tick()
                            if now - guard.LastAttackTime < guard.AttackCooldown then
                                return Status.RUNNING
                            end
                            
                            -- โจมตี target
                            local targetHumanoid = bb.target:FindFirstChildOfClass("Humanoid")
                            if targetHumanoid then
                                targetHumanoid:TakeDamage(guard.AttackDamage)
                                guard.LastAttackTime = now
                                guard:PlayAnimation("Attack")
                            end
                            
                            return Status.SUCCESS
                        end)
                    ),
                    
                    -- ไล่ตาม target
                    BehaviorTree.Action("Chase", function(bb)
                        local targetRoot = bb.target:FindFirstChild("HumanoidRootPart")
                        if not targetRoot then return Status.FAILURE end
                        
                        guard:PlayAnimation("Run")
                        guard.Humanoid.WalkSpeed = guard.MoveSpeed * 1.5
                        
                        -- อัพเดท path ทุก 1 วินาที
                        if not bb.lastPathTime or tick() - bb.lastPathTime > 1 then
                            local path = guard:FindPathTo(targetRoot.Position)
                            if path then
                                guard:FollowPath(path)
                                bb.lastPathTime = tick()
                            else
                                -- ถ้าหา path ไม่ได้ ให้ไปตรงๆ
                                guard.Humanoid:MoveTo(targetRoot.Position)
                            end
                        end
                        
                        return Status.RUNNING
                    end)
                )
            ),
            
            -- *** ลำดับความสำคัญที่ 2: Investigate ***
            BehaviorTree.Sequence("InvestigateSequence",
                BehaviorTree.Condition("HasSearchTarget", function(bb)
                    return guard.IsSearching and guard.SearchPosition ~= nil
                end),
                BehaviorTree.Action("Investigate", function(bb)
                    if not guard.HumanoidRootPart then return Status.FAILURE end
                    
                    local dist = (guard.HumanoidRootPart.Position - guard.SearchPosition).Magnitude
                    
                    if dist < 5 then
                        -- ถึงตำแหน่งแล้ว ให้มองรอบๆ
                        guard:PlayAnimation("Alert")
                        
                        if not bb.searchStartTime then
                            bb.searchStartTime = tick()
                        end
                        
                        -- มองรอบๆ 3 วินาที
                        if tick() - bb.searchStartTime > 3 then
                            guard.IsSearching = false
                            guard.SearchPosition = nil
                            bb.searchStartTime = nil
                        end
                        
                        return Status.RUNNING
                    else
                        -- เดินไปตรวจสอบ
                        guard:PlayAnimation("Walk")
                        guard.Humanoid.WalkSpeed = guard.MoveSpeed
                        
                        if not bb.investigatePathTime or tick() - bb.investigatePathTime > 2 then
                            local path = guard:FindPathTo(guard.SearchPosition)
                            if path then
                                guard:FollowPath(path)
                            end
                            bb.investigatePathTime = tick()
                        end
                        
                        return Status.RUNNING
                    end
                end)
            ),
            
            -- *** ลำดับความสำคัญที่ 3: Patrol ***
            BehaviorTree.Sequence("PatrolSequence",
                BehaviorTree.Condition("HasPatrolPoints", function(bb)
                    return guard.PatrolPoints and #guard.PatrolPoints > 0
                end),
                BehaviorTree.Action("Patrol", function(bb)
                    if not guard.HumanoidRootPart then return Status.FAILURE end
                    
                    local targetPoint = guard.PatrolPoints[guard.CurrentPatrolIndex]
                    local dist = (guard.HumanoidRootPart.Position - targetPoint).Magnitude
                    
                    if dist < 5 then
                        -- ถึงจุด patrol แล้ว รอสักครู่
                        guard:PlayAnimation("Idle")
                        guard.Humanoid:Move(Vector3.zero)
                        
                        if not bb.patrolWaitStart then
                            bb.patrolWaitStart = tick()
                        end
                        
                        if tick() - bb.patrolWaitStart > 2 then
                            -- ไปจุดถัดไป
                            guard.CurrentPatrolIndex = (guard.CurrentPatrolIndex % #guard.PatrolPoints) + 1
                            bb.patrolWaitStart = nil
                        end
                    else
                        -- เดินไปจุด patrol
                        guard:PlayAnimation("Walk")
                        guard.Humanoid.WalkSpeed = guard.MoveSpeed
                        
                        if not bb.patrolPathTime or tick() - bb.patrolPathTime > 2 then
                            local path = guard:FindPathTo(targetPoint)
                            if path then
                                guard:FollowPath(path)
                            end
                            bb.patrolPathTime = tick()
                        end
                    end
                    
                    return Status.RUNNING
                end)
            ),
            
            -- *** ลำดับความสำคัญที่ 4: Idle ***
            BehaviorTree.Action("Idle", function(bb)
                guard:PlayAnimation("Idle")
                guard.Humanoid:Move(Vector3.zero)
                return Status.RUNNING
            end)
        )
    )
    
    -- อัพเดท blackboard และรัน behavior tree
    guard.BehaviorTree = bt
    
    -- Override Update function
    local originalUpdate = guard.Update
    guard.Update = function(self, dt)
        -- อัพเดท perception
        perception:Update()
        
        -- หา target จาก perception
        local priorityTarget = perception:GetPriorityTarget()
        bt.Blackboard.target = priorityTarget
        
        -- ถ้าเห็นผู้เล่นครั้งแรก ให้ investigate
        if priorityTarget and not guard.IsSearching and guard.State ~= "Combat" then
            local targetRoot = priorityTarget:FindFirstChild("HumanoidRootPart")
            if targetRoot then
                guard.SearchPosition = targetRoot.Position
                guard.IsSearching = true
            end
        end
        
        -- รัน Behavior Tree
        bt:Tick()
    end
    
    guard:Start()
    
    return guard
end

-- ==================== ตัวอย่างการใช้งาน ====================
-- สร้าง patrol points
local patrolPoints = {
    Vector3.new(0, 0, 0),
    Vector3.new(20, 0, 0),
    Vector3.new(20, 0, 20),
    Vector3.new(0, 0, 20),
}

-- สร้าง guard จาก model ที่มีอยู่
-- local guardModel = workspace.GuardNPC
-- local guard = createGuard(guardModel, patrolPoints)
```

---

## ส่วนที่ 3: Boss AI System

### 3.1 Boss AI หลาย Phase

```lua
-- ModuleScript: BossAI
-- ระบบ Boss AI ที่มีหลาย Phase
-- Boss AI system with multiple phases

local BossAI = {}
BossAI.__index = BossAI

local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")

-- Boss phases configuration
local BOSS_PHASES = {
    {
        name = "Phase 1",
        healthThreshold = 1.0,   -- เริ่มจาก 100% HP
        nextThreshold = 0.75,    -- เปลี่ยน Phase เมื่อเหลือ 75%
        color = Color3.fromRGB(255, 100, 100),
        moveSpeed = 14,
        attackInterval = 2.0,
        attackDamage = 20,
        abilities = {"basicAttack", "charge"},
    },
    {
        name = "Phase 2",
        healthThreshold = 0.75,
        nextThreshold = 0.50,
        color = Color3.fromRGB(255, 50, 50),
        moveSpeed = 18,
        attackInterval = 1.5,
        attackDamage = 25,
        abilities = {"basicAttack", "charge", "groundSlam"},
    },
    {
        name = "Phase 3",
        healthThreshold = 0.50,
        nextThreshold = 0.25,
        color = Color3.fromRGB(200, 0, 0),
        moveSpeed = 22,
        attackInterval = 1.2,
        attackDamage = 30,
        abilities = {"basicAttack", "charge", "groundSlam", "summonMinions"},
    },
    {
        name = "Phase 4 (Enrage)",
        healthThreshold = 0.25,
        nextThreshold = 0,
        color = Color3.fromRGB(150, 0, 0),
        moveSpeed = 28,
        attackInterval = 0.8,
        attackDamage = 40,
        abilities = {"basicAttack", "charge", "groundSlam", "summonMinions", "laserBeam"},
    },
}

-- สร้าง Boss AI (Create Boss AI)
function BossAI.new(model)
    local self = setmetatable({}, BossAI)
    
    self.Model = model
    self.Humanoid = model:FindFirstChildOfClass("Humanoid")
    self.HumanoidRootPart = model:FindFirstChild("HumanoidRootPart")
    
    self.MaxHealth = 1000
    self.CurrentPhase = 1
    self.IsAlive = true
    self.Target = nil
    self.LastAbilityTime = {}  -- cooldown ของแต่ละ ability
    
    -- HealthBar GUI (ถ้ามี)
    self.HealthBar = model:FindFirstChild("BossHealthBar", true)
    
    -- ตั้งค่า Humanoid
    if self.Humanoid then
        self.Humanoid.MaxHealth = self.MaxHealth
        self.Humanoid.Health = self.MaxHealth
        
        self.Humanoid.HealthChanged:Connect(function(health)
            self:CheckPhaseTransition(health)
            self:UpdateHealthBar(health)
        end)
        
        self.Humanoid.Died:Connect(function()
            self:OnDeath()
        end)
    end
    
    -- Announce boss spawn
    self:AnnounceBoss()
    
    return self
end

-- ประกาศการมาของ Boss (Announce boss arrival)
function BossAI:AnnounceBoss()
    for _, player in ipairs(Players:GetPlayers()) do
        -- ส่ง GUI notification ให้ผู้เล่น
        local screenGui = Instance.new("ScreenGui")
        screenGui.Name = "BossAnnouncement"
        screenGui.ResetOnSpawn = false
        
        local frame = Instance.new("Frame", screenGui)
        frame.Size = UDim2.new(0.6, 0, 0.15, 0)
        frame.Position = UDim2.new(0.2, 0, 0.1, 0)
        frame.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
        frame.BackgroundTransparency = 0.3
        
        local label = Instance.new("TextLabel", frame)
        label.Size = UDim2.new(1, 0, 1, 0)
        label.Text = "⚠️ BOSS APPEARED: THE DARK DESTROYER ⚠️"
        label.TextColor3 = Color3.fromRGB(255, 255, 0)
        label.TextScaled = true
        label.Font = Enum.Font.GothamBold
        
        screenGui.Parent = player.PlayerGui
        
        -- ลบหลังจาก 5 วินาที
        task.delay(5, function()
            screenGui:Destroy()
        end)
    end
end

-- ตรวจสอบการเปลี่ยน Phase (Check phase transitions)
function BossAI:CheckPhaseTransition(health)
    if not self.IsAlive then return end
    
    local healthPercent = health / self.MaxHealth
    local nextPhase = self.CurrentPhase + 1
    
    if nextPhase <= #BOSS_PHASES then
        local nextPhaseData = BOSS_PHASES[nextPhase]
        
        if healthPercent <= nextPhaseData.healthThreshold then
            self:TransitionToPhase(nextPhase)
        end
    end
end

-- เปลี่ยน Phase (Transition to new phase)
function BossAI:TransitionToPhase(phaseIndex)
    local phaseData = BOSS_PHASES[phaseIndex]
    self.CurrentPhase = phaseIndex
    
    -- หยุดการเคลื่อนที่ชั่วคราว (Pause movement temporarily)
    if self.Humanoid then
        self.Humanoid.WalkSpeed = 0
    end
    
    -- เอฟเฟกต์การเปลี่ยน Phase (Phase transition effect)
    self:PlayPhaseTransitionEffect(phaseData)
    
    -- ประกาศ Phase ใหม่
    for _, player in ipairs(Players:GetPlayers()) do
        local bossHealthGui = player.PlayerGui:FindFirstChild("BossHealthGui")
        if bossHealthGui then
            local phaseLabel = bossHealthGui:FindFirstChild("PhaseLabel", true)
            if phaseLabel then
                phaseLabel.Text = phaseData.name
                phaseLabel.TextColor3 = phaseData.color
            end
        end
    end
    
    -- รอ 2 วินาที แล้ว resume
    task.delay(2, function()
        if self.IsAlive and self.Humanoid then
            self.Humanoid.WalkSpeed = phaseData.moveSpeed
        end
    end)
    
    print("Boss entered", phaseData.name)
end

-- เอฟเฟกต์การเปลี่ยน Phase (Phase transition visual effect)
function BossAI:PlayPhaseTransitionEffect(phaseData)
    -- เปลี่ยนสีตัว boss
    for _, part in ipairs(self.Model:GetDescendants()) do
        if part:IsA("BasePart") then
            local tween = TweenService:Create(
                part,
                TweenInfo.new(1, Enum.EasingStyle.Bounce),
                {Color = phaseData.color}
            )
            tween:Play()
        end
    end
    
    -- สร้าง shockwave effect
    local shockwave = Instance.new("Part")
    shockwave.Shape = Enum.PartType.Cylinder
    shockwave.Anchored = true
    shockwave.CanCollide = false
    shockwave.CastShadow = false
    shockwave.CFrame = CFrame.new(self.HumanoidRootPart.Position) * CFrame.Angles(0, 0, math.pi/2)
    shockwave.Size = Vector3.new(0.5, 5, 5)
    shockwave.Color = phaseData.color
    shockwave.Material = Enum.Material.Neon
    shockwave.Transparency = 0.3
    shockwave.Parent = workspace
    
    -- ขยาย shockwave
    local expandTween = TweenService:Create(
        shockwave,
        TweenInfo.new(1.5, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
        {
            Size = Vector3.new(0.5, 50, 50),
            Transparency = 1
        }
    )
    expandTween:Play()
    
    game:GetService("Debris"):AddItem(shockwave, 2)
end

-- ==================== Abilities ====================

-- โจมตีพื้นฐาน (Basic attack)
function BossAI:BasicAttack(target)
    if not target or not self.HumanoidRootPart then return end
    
    local targetRoot = target:FindFirstChild("HumanoidRootPart")
    if not targetRoot then return end
    
    local distance = (self.HumanoidRootPart.Position - targetRoot.Position).Magnitude
    if distance > 8 then return end
    
    local targetHumanoid = target:FindFirstChildOfClass("Humanoid")
    if targetHumanoid then
        local phaseData = BOSS_PHASES[self.CurrentPhase]
        targetHumanoid:TakeDamage(phaseData.attackDamage)
    end
end

-- พุ่ง charge (Charge attack)
function BossAI:ChargeAttack(target)
    if not target or not self.HumanoidRootPart then return end
    
    local targetRoot = target:FindFirstChild("HumanoidRootPart")
    if not targetRoot then return end
    
    -- เตรียมตัว (Prepare charge)
    if self.Humanoid then
        self.Humanoid.WalkSpeed = 0
    end
    
    task.wait(0.5)  -- telegraph animation
    
    -- พุ่งไปหา target
    local direction = (targetRoot.Position - self.HumanoidRootPart.Position).Unit
    
    if self.HumanoidRootPart then
        local bodyVelocity = Instance.new("BodyVelocity")
        bodyVelocity.Velocity = direction * 80
        bodyVelocity.MaxForce = Vector3.new(1e5, 0, 1e5)
        bodyVelocity.Parent = self.HumanoidRootPart
        
        -- ลบหลัง 0.5 วินาที
        game:GetService("Debris"):AddItem(bodyVelocity, 0.5)
        
        -- ตรวจสอบการชน (Check for collision damage)
        task.wait(0.5)
        
        for _, player in ipairs(Players:GetPlayers()) do
            local char = player.Character
            if char then
                local charRoot = char:FindFirstChild("HumanoidRootPart")
                if charRoot then
                    local dist = (self.HumanoidRootPart.Position - charRoot.Position).Magnitude
                    if dist < 6 then
                        local humanoid = char:FindFirstChildOfClass("Humanoid")
                        if humanoid then
                            humanoid:TakeDamage(50)
                            -- knockback
                            local kb = Instance.new("BodyVelocity")
                            kb.Velocity = direction * 40 + Vector3.new(0, 20, 0)
                            kb.MaxForce = Vector3.new(1e5, 1e5, 1e5)
                            kb.Parent = charRoot
                            game:GetService("Debris"):AddItem(kb, 0.3)
                        end
                    end
                end
            end
        end
    end
    
    -- กลับมาเดินปกติ
    local phaseData = BOSS_PHASES[self.CurrentPhase]
    if self.Humanoid then
        self.Humanoid.WalkSpeed = phaseData.moveSpeed
    end
end

-- กระแทกพื้น (Ground slam)
function BossAI:GroundSlam()
    if not self.HumanoidRootPart then return end
    
    local slamPosition = self.HumanoidRootPart.Position
    local slamRadius = 15
    
    -- Visual effect
    local shockwave = Instance.new("Part")
    shockwave.Shape = Enum.PartType.Cylinder
    shockwave.Anchored = true
    shockwave.CanCollide = false
    shockwave.Position = slamPosition - Vector3.new(0, 2, 0)
    shockwave.CFrame = CFrame.new(slamPosition - Vector3.new(0, 2, 0)) * CFrame.Angles(0, 0, math.pi/2)
    shockwave.Size = Vector3.new(1, slamRadius * 2, slamRadius * 2)
    shockwave.Color = Color3.fromRGB(255, 100, 0)
    shockwave.Material = Enum.Material.Neon
    shockwave.Transparency = 0.5
    shockwave.Parent = workspace
    
    -- ขยายออก
    TweenService:Create(
        shockwave,
        TweenInfo.new(0.5),
        {Transparency = 1, Size = Vector3.new(1, slamRadius * 4, slamRadius * 4)}
    ):Play()
    
    game:GetService("Debris"):AddItem(shockwave, 1)
    
    -- ทำ damage ในรัศมี
    for _, player in ipairs(Players:GetPlayers()) do
        local char = player.Character
        if char then
            local charRoot = char:FindFirstChild("HumanoidRootPart")
            if charRoot then
                local dist = (slamPosition - charRoot.Position).Magnitude
                if dist <= slamRadius then
                    local humanoid = char:FindFirstChildOfClass("Humanoid")
                    if humanoid then
                        -- damage ลดลงตามระยะ (damage falls off with distance)
                        local damageFalloff = 1 - (dist / slamRadius)
                        humanoid:TakeDamage(60 * damageFalloff)
                    end
                end
            end
        end
    end
end

-- เรียก minions (Summon minions)
function BossAI:SummonMinions()
    -- สร้าง minion รอบๆ boss
    for i = 1, 3 do
        local angle = (i / 3) * math.pi * 2
        local offset = Vector3.new(math.cos(angle) * 10, 0, math.sin(angle) * 10)
        local spawnPosition = self.HumanoidRootPart.Position + offset
        
        -- Clone minion model
        -- local minionModel = game.ServerStorage.MinionNPC:Clone()
        -- minionModel:SetPrimaryPartCFrame(CFrame.new(spawnPosition))
        -- minionModel.Parent = workspace
        
        print("Summoning minion at", spawnPosition)
    end
end

-- ยิง laser (Laser beam attack)
function BossAI:LaserBeam(target)
    if not target or not self.HumanoidRootPart then return end
    
    local targetRoot = target:FindFirstChild("HumanoidRootPart")
    if not targetRoot then return end
    
    -- สร้าง laser beam visual
    local beamPart = Instance.new("Part")
    beamPart.Anchored = true
    beamPart.CanCollide = false
    beamPart.Material = Enum.Material.Neon
    beamPart.Color = Color3.fromRGB(255, 0, 255)
    
    local startPos = self.HumanoidRootPart.Position + Vector3.new(0, 2, 0)
    local endPos = targetRoot.Position
    local beamLength = (endPos - startPos).Magnitude
    local midPoint = (startPos + endPos) / 2
    
    beamPart.Size = Vector3.new(1, 1, beamLength)
    beamPart.CFrame = CFrame.lookAt(midPoint, endPos)
    beamPart.Parent = workspace
    
    -- Chase target ด้วย laser
    local laserDuration = 3
    local startTime = tick()
    
    local laserConnection
    laserConnection = RunService.Heartbeat:Connect(function()
        if tick() - startTime > laserDuration then
            laserConnection:Disconnect()
            beamPart:Destroy()
            return
        end
        
        if not targetRoot or not targetRoot.Parent then
            laserConnection:Disconnect()
            beamPart:Destroy()
            return
        end
        
        -- อัพเดท beam position
        local newStart = self.HumanoidRootPart.Position + Vector3.new(0, 2, 0)
        local newEnd = targetRoot.Position
        local newLength = (newEnd - newStart).Magnitude
        local newMid = (newStart + newEnd) / 2
        
        beamPart.Size = Vector3.new(1, 1, newLength)
        beamPart.CFrame = CFrame.lookAt(newMid, newEnd)
        
        -- ทำ damage ต่อเนื่อง (Continuous damage)
        local dist = (newEnd - newStart).Magnitude
        if dist < beamLength + 2 then
            local humanoid = target:FindFirstChildOfClass("Humanoid")
            if humanoid then
                humanoid:TakeDamage(5)  -- 5 damage ต่อ frame
            end
        end
    end)
    
    game:GetService("Debris"):AddItem(beamPart, laserDuration + 0.5)
end

-- อัพเดท health bar (Update health bar)
function BossAI:UpdateHealthBar(health)
    for _, player in ipairs(Players:GetPlayers()) do
        local bossGui = player.PlayerGui:FindFirstChild("BossHealthGui")
        if bossGui then
            local healthFill = bossGui:FindFirstChild("HealthFill", true)
            if healthFill then
                local percent = health / self.MaxHealth
                TweenService:Create(
                    healthFill,
                    TweenInfo.new(0.3),
                    {Size = UDim2.new(percent, 0, 1, 0)}
                ):Play()
            end
        end
    end
end

-- Boss เสียชีวิต (Boss death)
function BossAI:OnDeath()
    self.IsAlive = false
    
    -- Death animation effect
    for _, part in ipairs(self.Model:GetDescendants()) do
        if part:IsA("BasePart") then
            TweenService:Create(
                part,
                TweenInfo.new(2, Enum.EasingStyle.Quad),
                {Transparency = 1, Size = part.Size * 2}
            ):Play()
        end
    end
    
    -- ให้ reward ผู้เล่นทุกคน
    for _, player in ipairs(Players:GetPlayers()) do
        -- Fire event for reward system
        game.ReplicatedStorage.Events.BossDefeated:FireClient(player, {
            bossName = "The Dark Destroyer",
            reward = 1000,
            experience = 500,
        })
    end
    
    -- ลบ model หลังจาก 3 วินาที
    task.delay(3, function()
        if self.Model and self.Model.Parent then
            self.Model:Destroy()
        end
    end)
    
    print("Boss defeated!")
end

-- เริ่ม AI (Start AI)
function BossAI:Start()
    local phaseData = BOSS_PHASES[self.CurrentPhase]
    
    if self.Humanoid then
        self.Humanoid.WalkSpeed = phaseData.moveSpeed
    end
    
    self._connection = RunService.Heartbeat:Connect(function(dt)
        if not self.IsAlive then return end
        
        -- หา target ที่ใกล้ที่สุด (Find nearest player)
        local nearestPlayer = nil
        local nearestDist = math.huge
        
        for _, player in ipairs(Players:GetPlayers()) do
            local char = player.Character
            if char and self.HumanoidRootPart then
                local charRoot = char:FindFirstChild("HumanoidRootPart")
                if charRoot then
                    local dist = (self.HumanoidRootPart.Position - charRoot.Position).Magnitude
                    if dist < nearestDist then
                        nearestDist = dist
                        nearestPlayer = char
                    end
                end
            end
        end
        
        self.Target = nearestPlayer
        
        if self.Target then
            -- เลือก ability ตาม phase (Select ability based on phase)
            local phaseData = BOSS_PHASES[self.CurrentPhase]
            local now = tick()
            
            -- ไล่ตาม target (Chase target)
            if self.Humanoid and nearestDist > 8 then
                self.Humanoid:MoveTo(self.Target.HumanoidRootPart.Position)
            end
            
            -- ใช้ ability ถ้าถึงเวลา (Use ability if ready)
            for _, ability in ipairs(phaseData.abilities) do
                local cooldown = 5  -- default cooldown
                if ability == "charge" then cooldown = 8
                elseif ability == "groundSlam" then cooldown = 6
                elseif ability == "summonMinions" then cooldown = 15
                elseif ability == "laserBeam" then cooldown = 10
                end
                
                local lastUsed = self.LastAbilityTime[ability] or 0
                
                if now - lastUsed >= cooldown then
                    -- สุ่มเลือก ability (Random ability selection)
                    if math.random() < 0.3 then
                        self.LastAbilityTime[ability] = now
                        
                        if ability == "basicAttack" then
                            self:BasicAttack(self.Target)
                        elseif ability == "charge" then
                            task.spawn(function() self:ChargeAttack(self.Target) end)
                        elseif ability == "groundSlam" then
                            task.spawn(function() self:GroundSlam() end)
                        elseif ability == "summonMinions" then
                            task.spawn(function() self:SummonMinions() end)
                        elseif ability == "laserBeam" then
                            task.spawn(function() self:LaserBeam(self.Target) end)
                        end
                        
                        break  -- ใช้แค่ 1 ability ต่อ tick
                    end
                end
            end
        end
    end)
end

-- หยุด AI (Stop AI)
function BossAI:Stop()
    if self._connection then
        self._connection:Disconnect()
    end
end

return BossAI
```

---

## ส่วนที่ 4: Group AI (AI กลุ่ม)

### 4.1 Swarm AI System

```lua
-- ModuleScript: SwarmAI
-- ระบบ AI กลุ่มสำหรับ mob ที่ต้องประสานงานกัน
-- Group AI for coordinated mob behavior

local SwarmAI = {}
SwarmAI.__index = SwarmAI

local RunService = game:GetService("RunService")
local Players = game:GetService("Players")

-- สร้าง Swarm manager (Create swarm manager)
function SwarmAI.new(config)
    local self = setmetatable({}, SwarmAI)
    
    self.Members = {}           -- สมาชิกทั้งหมดในกลุ่ม
    self.Target = nil           -- target หลักของกลุ่ม
    self.State = "Idle"         -- Idle, Moving, Combat, Retreat
    self.FormationType = "circle"  -- circle, line, wedge
    
    self.Config = {
        MaxMembers = config.MaxMembers or 10,
        SeparationRadius = config.SeparationRadius or 5,
        CohesionRadius = config.CohesionRadius or 20,
        AlignmentRadius = config.AlignmentRadius or 15,
        AttackRadius = config.AttackRadius or 30,
        RetreatHealth = config.RetreatHealth or 0.3,
    }
    
    return self
end

-- เพิ่มสมาชิก (Add member)
function SwarmAI:AddMember(npc)
    if #self.Members >= self.Config.MaxMembers then return false end
    
    table.insert(self.Members, npc)
    npc.SwarmGroup = self
    return true
end

-- ลบสมาชิก (Remove member)
function SwarmAI:RemoveMember(npc)
    for i, member in ipairs(self.Members) do
        if member == npc then
            table.remove(self.Members, i)
            return
        end
    end
end

-- คำนวณ Flocking forces (Calculate flocking forces - Boids algorithm)
function SwarmAI:CalculateFlockingForce(member)
    local separation = Vector3.zero
    local cohesion = Vector3.zero
    local alignment = Vector3.zero
    
    local separationCount = 0
    local cohesionCount = 0
    local alignmentCount = 0
    
    if not member.HumanoidRootPart then return Vector3.zero end
    
    local memberPos = member.HumanoidRootPart.Position
    
    for _, other in ipairs(self.Members) do
        if other ~= member and other.HumanoidRootPart then
            local otherPos = other.HumanoidRootPart.Position
            local toOther = otherPos - memberPos
            local dist = toOther.Magnitude
            
            -- Separation: หลีกเลี่ยงสมาชิกที่ใกล้เกินไป (Avoid nearby members)
            if dist < self.Config.SeparationRadius then
                separation = separation - toOther.Unit * (1 / dist)
                separationCount = separationCount + 1
            end
            
            -- Cohesion: เข้าหาจุดกลาง (Move toward center of group)
            if dist < self.Config.CohesionRadius then
                cohesion = cohesion + otherPos
                cohesionCount = cohesionCount + 1
            end
            
            -- Alignment: เดินทิศเดียวกัน (Move in same direction)
            if dist < self.Config.AlignmentRadius and other.Humanoid then
                local otherVel = other.Humanoid.MoveDirection
                alignment = alignment + otherVel
                alignmentCount = alignmentCount + 1
            end
        end
    end
    
    -- Normalize forces
    if separationCount > 0 then
        separation = separation / separationCount
    end
    
    if cohesionCount > 0 then
        cohesion = cohesion / cohesionCount
        cohesion = (cohesion - memberPos).Unit  -- เปลี่ยนเป็น direction
    end
    
    if alignmentCount > 0 then
        alignment = (alignment / alignmentCount).Unit
    end
    
    -- รวม forces (Combine forces)
    local combined = separation * 2 + cohesion * 1 + alignment * 0.5
    
    return combined
end

-- กำหนด target ให้กลุ่ม (Set group target)
function SwarmAI:SetTarget(target)
    self.Target = target
    self.State = "Combat"
    
    -- แจ้งสมาชิกทุกคน (Notify all members)
    for _, member in ipairs(self.Members) do
        member.Target = target
        member:SetState("Combat")
    end
end

-- ถอยทัพ (Retreat when weak)
function SwarmAI:Retreat()
    self.State = "Retreat"
    self.Target = nil
    
    for _, member in ipairs(self.Members) do
        member.Target = nil
        member:SetState("Retreat")
        
        -- วิ่งไปทิศตรงข้ามจาก target
        -- (Implementation depends on retreat destination)
    end
end

-- ตรวจสอบสุขภาพกลุ่ม (Check group health)
function SwarmAI:GetGroupHealth()
    local totalHealth = 0
    local totalMaxHealth = 0
    
    for _, member in ipairs(self.Members) do
        if member.Humanoid then
            totalHealth = totalHealth + member.Humanoid.Health
            totalMaxHealth = totalMaxHealth + member.Humanoid.MaxHealth
        end
    end
    
    if totalMaxHealth == 0 then return 0 end
    return totalHealth / totalMaxHealth
end

-- อัพเดทกลุ่ม (Update group)
function SwarmAI:Update()
    -- ลบสมาชิกที่ตายแล้ว (Remove dead members)
    for i = #self.Members, 1, -1 do
        local member = self.Members[i]
        if not member.IsAlive then
            table.remove(self.Members, i)
        end
    end
    
    if #self.Members == 0 then
        self.State = "Disbanded"
        return
    end
    
    -- ตรวจสอบสุขภาพ (Check health for retreat)
    local groupHealth = self:GetGroupHealth()
    if groupHealth < self.Config.RetreatHealth and self.State == "Combat" then
        self:Retreat()
    end
    
    -- หา target ถ้ายังไม่มี (Find target if none)
    if self.State == "Idle" or (self.State == "Combat" and not self.Target) then
        -- หาผู้เล่นในรัศมี (Find player in attack radius)
        local leader = self.Members[1]
        if leader and leader.HumanoidRootPart then
            for _, player in ipairs(Players:GetPlayers()) do
                local char = player.Character
                if char then
                    local charRoot = char:FindFirstChild("HumanoidRootPart")
                    if charRoot then
                        local dist = (leader.HumanoidRootPart.Position - charRoot.Position).Magnitude
                        if dist <= self.Config.AttackRadius then
                            self:SetTarget(char)
                            break
                        end
                    end
                end
            end
        end
    end
    
    -- อัพเดท flocking สำหรับสมาชิกทุกคน (Update flocking for all members)
    for _, member in ipairs(self.Members) do
        if member.IsAlive and member.HumanoidRootPart then
            -- flocking force
            local flockForce = self:CalculateFlockingForce(member)
            
            -- ทิศทางไปยัง target (Direction toward target)
            local targetForce = Vector3.zero
            if self.Target then
                local targetRoot = self.Target:FindFirstChild("HumanoidRootPart")
                if targetRoot then
                    targetForce = (targetRoot.Position - member.HumanoidRootPart.Position).Unit
                end
            end
            
            -- รวม forces
            local finalDirection = (flockForce + targetForce * 3).Unit
            
            -- เคลื่อนที่ (Move in combined direction)
            if member.Humanoid and finalDirection.Magnitude > 0.1 then
                member.Humanoid:Move(finalDirection)
            end
        end
    end
end

-- เริ่มระบบ (Start system)
function SwarmAI:Start()
    self._connection = RunService.Heartbeat:Connect(function()
        self:Update()
    end)
end

-- หยุดระบบ (Stop system)
function SwarmAI:Stop()
    if self._connection then
        self._connection:Disconnect()
    end
end

return SwarmAI
```

---

## ส่วนที่ 5: AI Performance Optimization

### 5.1 AI Manager สำหรับ LOD (Level of Detail)

```lua
-- ModuleScript: AIManager
-- จัดการ NPC หลายตัวพร้อมกันอย่างมีประสิทธิภาพ
-- Manage multiple NPCs efficiently with LOD

local AIManager = {}
AIManager.__index = AIManager

local RunService = game:GetService("RunService")
local Players = game:GetService("Players")

-- LOD levels
local LOD_LEVELS = {
    {
        name = "High",
        maxDistance = 50,
        updateRate = 0.1,  -- อัพเดทบ่อย (Update frequently)
        enablePerception = true,
        enablePathfinding = true,
        enableAnimations = true,
    },
    {
        name = "Medium", 
        maxDistance = 100,
        updateRate = 0.3,
        enablePerception = true,
        enablePathfinding = true,
        enableAnimations = false,
    },
    {
        name = "Low",
        maxDistance = 200,
        updateRate = 1.0,  -- อัพเดทช้า (Update slowly)
        enablePerception = false,
        enablePathfinding = false,
        enableAnimations = false,
    },
    {
        name = "Frozen",
        maxDistance = math.huge,
        updateRate = math.huge,  -- ไม่อัพเดทเลย (Never update)
        enablePerception = false,
        enablePathfinding = false,
        enableAnimations = false,
    },
}

-- สร้าง AI Manager (Create AI Manager)
function AIManager.new()
    local self = setmetatable({}, AIManager)
    
    self.NPCs = {}      -- {npc, lodLevel, lastUpdateTime}
    self.MaxNPCs = 50   -- NPC สูงสุดที่ active พร้อมกัน
    
    return self
end

-- ลงทะเบียน NPC (Register NPC)
function AIManager:Register(npc)
    table.insert(self.NPCs, {
        npc = npc,
        lodLevel = 3,   -- เริ่มต้นที่ Low
        lastUpdateTime = 0,
    })
end

-- คำนวณ LOD level ตามระยะ (Calculate LOD level based on distance)
function AIManager:GetLODLevel(npcPosition)
    local nearestPlayerDist = math.huge
    
    for _, player in ipairs(Players:GetPlayers()) do
        local char = player.Character
        if char then
            local charRoot = char:FindFirstChild("HumanoidRootPart")
            if charRoot then
                local dist = (npcPosition - charRoot.Position).Magnitude
                nearestPlayerDist = math.min(nearestPlayerDist, dist)
            end
        end
    end
    
    -- หา LOD level ที่เหมาะสม
    for i, lod in ipairs(LOD_LEVELS) do
        if nearestPlayerDist <= lod.maxDistance then
            return i
        end
    end
    
    return #LOD_LEVELS  -- Frozen
end

-- อัพเดท NPC ทั้งหมด (Update all NPCs)
function AIManager:Update()
    local now = tick()
    local activeCount = 0
    
    for _, entry in ipairs(self.NPCs) do
        local npc = entry.npc
        
        if not npc.IsAlive then
            -- ลบออก (Remove dead NPCs in next cleanup)
            continue
        end
        
        if not npc.HumanoidRootPart then continue end
        
        -- อัพเดท LOD level (Update LOD level)
        entry.lodLevel = self:GetLODLevel(npc.HumanoidRootPart.Position)
        local lod = LOD_LEVELS[entry.lodLevel]
        
        -- ตรวจสอบว่าถึงเวลาอัพเดทหรือยัง (Check if it's time to update)
        if now - entry.lastUpdateTime >= lod.updateRate then
            entry.lastUpdateTime = now
            
            -- อัพเดตตาม LOD level (Update based on LOD)
            if lod.enablePerception and npc.Perception then
                npc.Perception:Update()
            end
            
            if lod.enablePathfinding then
                npc:Update(lod.updateRate)
            end
            
            -- เปิด/ปิด animations
            if npc.Humanoid then
                if lod.enableAnimations then
                    npc.Humanoid.EvaluateStateMachine = true
                else
                    npc.Humanoid.EvaluateStateMachine = false
                end
            end
            
            activeCount = activeCount + 1
        end
    end
    
    -- ล้าง NPCs ที่ตายแล้ว (Clean up dead NPCs)
    for i = #self.NPCs, 1, -1 do
        if not self.NPCs[i].npc.IsAlive then
            table.remove(self.NPCs, i)
        end
    end
end

-- เริ่มระบบ (Start system)
function AIManager:Start()
    self._connection = RunService.Heartbeat:Connect(function()
        self:Update()
    end)
end

-- สถิติการทำงาน (Performance statistics)
function AIManager:GetStats()
    local stats = {
        total = #self.NPCs,
        byLOD = {0, 0, 0, 0},
    }
    
    for _, entry in ipairs(self.NPCs) do
        if entry.lodLevel >= 1 and entry.lodLevel <= 4 then
            stats.byLOD[entry.lodLevel] = stats.byLOD[entry.lodLevel] + 1
        end
    end
    
    return stats
end

return AIManager
```

---

## แบบฝึกหัด (Practice Exercises)

### แบบฝึกหัดที่ 1: สร้าง Archer NPC
```
สร้าง NPC ยิงธนูที่:
1. รักษาระยะห่างจาก player (ไม่เข้าใกล้เกินไป)
2. ยิงธนูเมื่ออยู่ในระยะ 20-40 studs
3. หลบซ่อนเมื่อ health ต่ำกว่า 50%
4. มี special attack "Rain of Arrows" ที่ยิงในพื้นที่วงกลม
```

### แบบฝึกหัดที่ 2: Merchant NPC
```
สร้าง NPC พ่อค้าที่:
1. เดินไปมาในพื้นที่กำหนด
2. หยุดเมื่อผู้เล่นเข้าใกล้
3. แสดง dialog "สวัสดีนักผจญภัย! ต้องการซื้ออะไรไหม?"
4. เปิด shop GUI เมื่อผู้เล่นกด interact
5. หลบออกเมื่อมีการต่อสู้ในบริเวณใกล้เคียง
```

### แบบฝึกหัดที่ 3: Companion AI
```
สร้าง NPC เพื่อนร่วมทีมที่:
1. ติดตาม player ในระยะ 10 studs
2. โจมตี enemy ที่โจมตี player
3. รักษา player เมื่อ health ต่ำกว่า 30%
4. กลับไปหา player ถ้าห่างเกิน 30 studs
5. มี personality ที่แตกต่างกัน (aggressive, defensive, healer)
```

---

## สรุป (Summary)

ในบทนี้เราได้เรียนรู้:

1. **NPCBase** - โครงสร้างพื้นฐานสำหรับ NPC ทุกประเภทด้วย PathfindingService
2. **PerceptionSystem** - ระบบการมองเห็นและได้ยินที่ใช้ raycast และ field of view
3. **BehaviorTree** - ระบบ tree ที่มี Sequence, Selector, Parallel, Action, Condition nodes
4. **Guard NPC** - ตัวอย่าง NPC ที่ใช้ Behavior Tree สำหรับ patrol/investigate/combat
5. **Boss AI** - Boss หลาย phase ที่มี abilities หลายอย่างและเอฟเฟกต์การเปลี่ยน phase
6. **Swarm AI** - Boids algorithm สำหรับ mob ที่เคลื่อนที่เป็นกลุ่ม
7. **AI Manager** - LOD system สำหรับ optimize การทำงานของ NPC หลายตัว

### เทคนิคสำคัญ:
- **Behavior Tree** ดีกว่า State Machine เพราะ reusable และ maintainable
- **PathfindingService** สำคัญมากสำหรับ NPC ที่ต้องนำทางในสภาพแวดล้อมซับซ้อน
- **LOD System** ลด CPU load ได้มากเมื่อมี NPC จำนวนมาก
- **Perception System** ทำให้ AI รู้สึก "จริง" มากขึ้น

*บทถัดไป: Part 97 - Advanced Networking และ Lag Compensation*
