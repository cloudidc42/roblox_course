# Part 54: NPC Pathfinding ด้วย PathfindingService

## บทนำ

PathfindingService ของ Roblox ช่วยให้ NPC สามารถเดินหลบสิ่งกีดขวาง หาทางไปยังเป้าหมาย และเดินในพื้นที่ซับซ้อนได้ อัลกอริทึมที่ใช้คือ A* (A-Star) ซึ่งมีประสิทธิภาพสูง

## ทำความเข้าใจ PathfindingService

```lua
-- PathfindingService ทำงานอย่างไร?
-- 1. รับ start และ end point
-- 2. คำนวณเส้นทางหลีกเลี่ยง obstacles
-- 3. คืน waypoints (จุดผ่าน) ที่ NPC ต้องเดินผ่าน

local PathfindingService = game:GetService("PathfindingService")

local path = PathfindingService:CreatePath({
    AgentRadius = 2,    -- ความกว้างของ agent
    AgentHeight = 5,    -- ความสูงของ agent
    AgentCanJump = true, -- กระโดดได้ไหม
    AgentCanClimb = false, -- ปีนได้ไหม
    WaypointSpacing = 4  -- ระยะห่างระหว่าง waypoints
})
```

## NPC Pathfinding พื้นฐาน

```lua
-- Script ใน NPC Model

local PathfindingService = game:GetService("PathfindingService")
local Players = game:GetService("Players")

local NPC = script.Parent
local humanoid = NPC:WaitForChild("Humanoid")
local rootPart = NPC:WaitForChild("HumanoidRootPart")

-- ===== Configuration =====
local CONFIG = {
    walkSpeed = 14,
    runSpeed = 20,
    attackRange = 5,
    detectionRange = 40,
    attackDamage = 15,
    attackCooldown = 1.5,
    pathUpdateInterval = 1,  -- อัพเดทเส้นทางทุกกี่วินาที
    maxJumpCount = 3
}

-- Path parameters
local pathParams = {
    AgentRadius = 2,
    AgentHeight = 5,
    AgentCanJump = true,
    AgentCanClimb = true,
    WaypointSpacing = 3,
    Costs = {
        -- กำหนดต้นทุนของ material ต่างๆ
        Water = 20,      -- หลีกเลี่ยงน้ำ
        Ice = 5          -- หลีกเลี่ยงน้ำแข็งบ้าง
    }
}

-- ===== Path State =====
local currentPath = nil
local waypoints = {}
local currentWaypointIndex = 1
local isFollowingPath = false
local lastPathUpdate = 0
local target = nil
local lastAttackTime = 0

-- ===== Pathfinding Functions =====

local function computePath(destination)
    local path = PathfindingService:CreatePath(pathParams)
    
    local success, err = pcall(function()
        path:ComputeAsync(rootPart.Position, destination)
    end)
    
    if not success then
        warn("Path computation failed: " .. err)
        return nil
    end
    
    if path.Status == Enum.PathStatus.Success then
        return path
    elseif path.Status == Enum.PathStatus.NoPath then
        -- ไม่พบเส้นทาง
        return nil
    end
    
    return nil
end

local function moveToWaypoint(waypoint)
    -- ตรวจสอบว่าต้องกระโดดหรือเปล่า
    if waypoint.Action == Enum.PathWaypointAction.Jump then
        humanoid.Jump = true
    end
    
    humanoid:MoveTo(waypoint.Position)
end

local function followPath(path)
    if not path then return end
    
    waypoints = path:GetWaypoints()
    currentWaypointIndex = 1
    isFollowingPath = true
    
    for i = 1, #waypoints do
        local waypoint = waypoints[i]
        
        if not isFollowingPath then break end
        
        -- ข้าม waypoint แรก (ตำแหน่งปัจจุบัน)
        if i == 1 then continue end
        
        moveToWaypoint(waypoint)
        
        -- รอให้ถึง waypoint
        local reached = humanoid.MoveToFinished:Wait(5)
        
        if not reached then
            -- ไม่ถึงใน timeout - คำนวณ path ใหม่
            isFollowingPath = false
            return false
        end
        
        currentWaypointIndex = i
    end
    
    isFollowingPath = false
    return true
end

-- ===== Smart Path Following =====

local function moveToTarget(targetPosition, shouldRun)
    humanoid.WalkSpeed = shouldRun and CONFIG.runSpeed or CONFIG.walkSpeed
    
    local path = computePath(targetPosition)
    
    if path then
        followPath(path)
    else
        -- ถ้าไม่มี path ลองเดินตรงๆ
        humanoid:MoveTo(targetPosition)
        humanoid.MoveToFinished:Wait(3)
    end
end

-- ===== Combat =====

local function attackTarget(targetCharacter)
    local targetHumanoid = targetCharacter:FindFirstChild("Humanoid")
    if not targetHumanoid or targetHumanoid.Health <= 0 then
        return false
    end
    
    local now = os.clock()
    if now - lastAttackTime < CONFIG.attackCooldown then
        return false
    end
    
    lastAttackTime = now
    
    -- Check range
    local targetRoot = targetCharacter:FindFirstChild("HumanoidRootPart")
    if not targetRoot then return false end
    
    local dist = (rootPart.Position - targetRoot.Position).Magnitude
    if dist > CONFIG.attackRange then return false end
    
    -- หันหน้าหา target
    local lookPos = Vector3.new(targetRoot.Position.X, rootPart.Position.Y, targetRoot.Position.Z)
    rootPart.CFrame = CFrame.lookAt(rootPart.Position, lookPos)
    
    -- โจมตี
    targetHumanoid:TakeDamage(CONFIG.attackDamage)
    print(NPC.Name .. " โจมตี! " .. CONFIG.attackDamage .. " damage")
    
    return true
end

-- ===== Main AI Loop =====

local function findTarget()
    local nearest = nil
    local nearestDist = CONFIG.detectionRange
    
    for _, player in ipairs(Players:GetPlayers()) do
        local char = player.Character
        if not char then continue end
        
        local root = char:FindFirstChild("HumanoidRootPart")
        local hum = char:FindFirstChild("Humanoid")
        
        if not root or not hum or hum.Health <= 0 then continue end
        
        local dist = (rootPart.Position - root.Position).Magnitude
        if dist < nearestDist then
            nearest = player
            nearestDist = dist
        end
    end
    
    return nearest
end

task.spawn(function()
    while NPC.Parent and humanoid.Health > 0 do
        target = findTarget()
        
        if target and target.Character then
            local targetChar = target.Character
            local targetRoot = targetChar:FindFirstChild("HumanoidRootPart")
            local targetHum = targetChar:FindFirstChild("Humanoid")
            
            if targetRoot and targetHum and targetHum.Health > 0 then
                local dist = (rootPart.Position - targetRoot.Position).Magnitude
                
                if dist <= CONFIG.attackRange then
                    -- โจมตี
                    humanoid:MoveTo(rootPart.Position)  -- หยุดเดิน
                    attackTarget(targetChar)
                    task.wait(0.1)
                else
                    -- เดินหาผ่าน pathfinding
                    local now = os.time()
                    if now - lastPathUpdate >= CONFIG.pathUpdateInterval then
                        lastPathUpdate = now
                        
                        -- คำนวณ path ใหม่
                        isFollowingPath = false
                        task.wait(0.1)
                        
                        task.spawn(function()
                            moveToTarget(targetRoot.Position, dist > 20)
                        end)
                    end
                    
                    task.wait(0.2)
                end
            end
        else
            -- ไม่มี target - กลับ spawn point
            local distToSpawn = (rootPart.Position - NPC:GetAttribute("SpawnPosition") or Vector3.new(0,0,0)).Magnitude
            
            if distToSpawn > 5 then
                task.spawn(function()
                    moveToTarget(NPC:GetAttribute("SpawnPosition") or Vector3.new(0,0,0), false)
                end)
            else
                humanoid:MoveTo(rootPart.Position)  -- หยุดนิ่ง
            end
            
            task.wait(1)
        end
    end
end)
```

## Advanced Pathfinding: Dynamic Obstacles

```lua
-- สำหรับ NPC ที่ต้องหลีกเลี่ยงสิ่งกีดขวางที่เคลื่อนที่

local PathfindingService = game:GetService("PathfindingService")

local AdvancedNPC = {}
AdvancedNPC.__index = AdvancedNPC

function AdvancedNPC.new(model)
    local self = setmetatable({}, AdvancedNPC)
    
    self.model = model
    self.humanoid = model:WaitForChild("Humanoid")
    self.root = model:WaitForChild("HumanoidRootPart")
    
    self.path = PathfindingService:CreatePath({
        AgentRadius = 2,
        AgentHeight = 5,
        AgentCanJump = true,
        WaypointSpacing = 3
    })
    
    self.waypoints = {}
    self.waypointIndex = 1
    self.isMoving = false
    self.currentTarget = nil
    self.pathBlocked = false
    
    -- ฟัง event ถ้าเส้นทางถูกกั้น
    self.path.Blocked:Connect(function(blockedIndex)
        self:onPathBlocked(blockedIndex)
    end)
    
    return self
end

function AdvancedNPC:onPathBlocked(blockedIndex)
    if blockedIndex >= self.waypointIndex then
        self.pathBlocked = true
        print(self.model.Name .. " เส้นทางถูกกั้น! คำนวณใหม่...")
        
        -- คำนวณเส้นทางใหม่
        if self.currentTarget then
            self:moveTo(self.currentTarget)
        end
    end
end

function AdvancedNPC:moveTo(destination)
    self.currentTarget = destination
    self.pathBlocked = false
    
    -- คำนวณ path
    local success = pcall(function()
        self.path:ComputeAsync(self.root.Position, destination)
    end)
    
    if not success or self.path.Status ~= Enum.PathStatus.Success then
        -- ถ้าไม่มี path ลองเดินตรง
        self.humanoid:MoveTo(destination)
        return
    end
    
    self.waypoints = self.path:GetWaypoints()
    self.waypointIndex = 1
    self.isMoving = true
    
    self:followWaypoints()
end

function AdvancedNPC:followWaypoints()
    task.spawn(function()
        for i = 2, #self.waypoints do
            if not self.isMoving then break end
            if self.pathBlocked then break end
            
            local waypoint = self.waypoints[i]
            self.waypointIndex = i
            
            if waypoint.Action == Enum.PathWaypointAction.Jump then
                self.humanoid.Jump = true
            end
            
            self.humanoid:MoveTo(waypoint.Position)
            
            local reached = self.humanoid.MoveToFinished:Wait(4)
            
            if not reached and not self.pathBlocked then
                -- ไม่ถึง - อาจมีสิ่งกีดขวาง
                self.pathBlocked = true
                self:moveTo(self.currentTarget)
                break
            end
        end
        
        self.isMoving = false
    end)
end

function AdvancedNPC:stop()
    self.isMoving = false
    self.humanoid:MoveTo(self.root.Position)
end

-- ใช้งาน
local npc = AdvancedNPC.new(workspace.EnemyNPC)
npc:moveTo(Vector3.new(100, 0, 100))
```

## NPC กับ Custom PathfindingModifiers

```lua
-- PathfindingModifier ช่วยให้กำหนดพื้นที่พิเศษสำหรับ pathfinding

-- ตัวอย่าง: สร้าง zone ที่ NPC ควรหลีกเลี่ยง (เช่น บริเวณอันตราย)

local function createDangerZone(position, radius)
    local modifier = Instance.new("PathfindingModifier")
    modifier.Label = "DangerZone"
    
    local part = Instance.new("Part")
    part.Size = Vector3.new(radius*2, 1, radius*2)
    part.Position = position
    part.Anchored = true
    part.CanCollide = false
    part.Transparency = 0.9
    part.BrickColor = BrickColor.new("Bright red")
    part.Parent = workspace
    
    modifier.Parent = part
    
    return part
end

-- ใน Path params กำหนดต้นทุน DangerZone
local dangerAwarePath = PathfindingService:CreatePath({
    AgentRadius = 2,
    AgentHeight = 5,
    AgentCanJump = true,
    Costs = {
        DangerZone = 1000,  -- ต้นทุนสูงมาก = หลีกเลี่ยง
        Water = 15
    }
})
```

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Patrol Path
สร้าง NPC ที่เดินตาม waypoints ที่กำหนดไว้ล่วงหน้า

### แบบฝึกหัดที่ 2: Group Pathfinding
สร้างกลุ่ม NPC ที่เดินพร้อมกันโดยไม่ชนกัน

### แบบฝึกหัดที่ 3: Flee Behavior
NPC ที่วิ่งหนีผู้เล่นเมื่อ HP ต่ำ

## สรุป

PathfindingService ช่วยให้สร้าง NPC ที่ฉลาดขึ้น:
- **ComputeAsync** คำนวณเส้นทาง
- **GetWaypoints** ดึงจุดที่ต้องผ่าน
- **Blocked event** ตรวจจับเส้นทางถูกกั้น
- **PathfindingModifier** กำหนดต้นทุนพื้นที่
- **Dynamic recompute** คำนวณใหม่เมื่อจำเป็น
