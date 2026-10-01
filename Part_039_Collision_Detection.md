# Part 39: Collision Detection (การตรวจจับการชนกัน)

## บทนำ

Collision Detection คือการตรวจสอบว่า Objects ชนกันหรืออยู่ใกล้กันหรือไม่ Roblox มีหลายวิธีในการทำ Collision Detection ตั้งแต่การใช้ Physics Engine, Raycast, ไปจนถึง Overlap Detection ซึ่งแต่ละวิธีเหมาะกับสถานการณ์ต่างกัน

---

## 39.1 CanCollide Property

ควบคุมว่า Part จะชนกับ Parts อื่นหรือไม่

```lua
local part = workspace.MyPart

-- เปิด collision (default)
part.CanCollide = true

-- ปิด collision (ทะลุผ่านได้)
part.CanCollide = false

-- CanQuery: ตรวจจับได้ด้วย Raycast แต่ไม่ชนจริง
part.CanQuery = true   -- default true

-- CanTouch: Touched event ทำงานหรือไม่
part.CanTouch = true   -- default true
```

---

## 39.2 CollisionGroups

กำหนดกลุ่มสำหรับควบคุมว่า Groups ไหนชนกันได้

```lua
-- Script ใน ServerScriptService
local PhysicsService = game:GetService("PhysicsService")

-- สร้าง Collision Groups
PhysicsService:RegisterCollisionGroup("Players")
PhysicsService:RegisterCollisionGroup("Enemies")
PhysicsService:RegisterCollisionGroup("Projectiles")
PhysicsService:RegisterCollisionGroup("Walls")

-- กำหนดว่า Groups ไหนชนกันได้
-- (Default: ทุก group ชนกันได้หมด)

-- ผู้เล่นไม่ชนกันเอง (ไม่กัน ghost กัน)
PhysicsService:CollisionGroupSetCollidable("Players", "Players", false)

-- กระสุนไม่ชนกับผู้ยิง
PhysicsService:CollisionGroupSetCollidable("Players", "Projectiles", false)

-- ศัตรูไม่ชนกันเอง
PhysicsService:CollisionGroupSetCollidable("Enemies", "Enemies", false)

-- ตรวจสอบ
local canCollide = PhysicsService:CollisionGroupsAreCollidable("Players", "Enemies")
print(canCollide)  -- true

-- ใส่ group ให้ Part
local playerPart = workspace.PlayerCharacter.HumanoidRootPart
playerPart.CollisionGroup = "Players"

local bulletPart = workspace.Bullet
bulletPart.CollisionGroup = "Projectiles"
```

### ใส่ CollisionGroup ให้ Character อัตโนมัติ

```lua
-- Script ใน ServerScriptService
local PhysicsService = game:GetService("PhysicsService")
local Players = game:GetService("Players")

-- สร้าง group
PhysicsService:RegisterCollisionGroup("Players")
PhysicsService:CollisionGroupSetCollidable("Players", "Players", false)

local function setCharacterCollisionGroup(character)
    for _, part in pairs(character:GetDescendants()) do
        if part:IsA("BasePart") then
            part.CollisionGroup = "Players"
        end
    end
end

Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function(character)
        setCharacterCollisionGroup(character)
        
        -- ใส่ให้ parts ใหม่ที่เพิ่มเข้ามา (เช่น Tools)
        character.DescendantAdded:Connect(function(descendant)
            if descendant:IsA("BasePart") then
                descendant.CollisionGroup = "Players"
            end
        end)
    end)
end)
```

---

## 39.3 Raycast

Raycast ยิง Ray (เส้นตรง) ออกไปและหาว่าชน Part ไหน

### พื้นฐาน Raycast

```lua
-- Workspace:Raycast(origin, direction, params)
-- origin: Vector3 จุดเริ่มต้น
-- direction: Vector3 ทิศทาง (ความยาว = ระยะ)
-- params: RaycastParams (optional)

-- ตัวอย่างง่ายๆ
local origin = Vector3.new(0, 100, 0)
local direction = Vector3.new(0, -200, 0)  -- ยิงลงไป 200 studs

local result = workspace:Raycast(origin, direction)

if result then
    print("ชน:", result.Instance.Name)
    print("ตำแหน่ง:", result.Position)
    print("Normal:", result.Normal)  -- ทิศทางตั้งฉาก
    print("ระยะ:", result.Distance)
    print("Material:", result.Material)
else
    print("ไม่ชนอะไรเลย")
end
```

### RaycastParams

```lua
local params = RaycastParams.new()

-- กรองแบบ Include (เฉพาะ Parts ใน list เท่านั้น)
params.FilterDescendantsInstances = {workspace.EnemiesFolder}
params.FilterType = Enum.RaycastFilterType.Include

-- กรองแบบ Exclude (ข้ามทุกอย่างใน list)
params.FilterDescendantsInstances = {player.Character}
params.FilterType = Enum.RaycastFilterType.Exclude

-- ใช้ CollisionGroup ในการกรอง
params.CollisionGroup = "Projectiles"

-- ตรวจจับ Parts ที่ CanCollide = false ด้วย (สำหรับ Zone detection)
params.IgnoreWater = false  -- ข้ามน้ำหรือไม่
```

### Ground Detection

```lua
-- ตรวจสอบว่าตัวละครอยู่บนพื้นหรือไม่
local function isOnGround(character)
    local hrp = character:FindFirstChild("HumanoidRootPart")
    if not hrp then return false end
    
    local params = RaycastParams.new()
    params.FilterDescendantsInstances = {character}
    params.FilterType = Enum.RaycastFilterType.Exclude
    
    local origin = hrp.Position
    local direction = Vector3.new(0, -3.5, 0)  -- ยิงลงไป 3.5 studs
    
    local result = workspace:Raycast(origin, direction, params)
    
    if result then
        return true, result.Instance, result.Material
    end
    return false
end

-- ใช้งาน
local grounded, groundPart, groundMaterial = isOnGround(character)
if grounded then
    print("อยู่บน:", groundPart.Name, "Material:", groundMaterial)
end
```

### Wall Detection

```lua
-- ตรวจสอบกำแพงรอบตัวละคร
local function detectWalls(character)
    local hrp = character:FindFirstChild("HumanoidRootPart")
    if not hrp then return {} end
    
    local params = RaycastParams.new()
    params.FilterDescendantsInstances = {character}
    params.FilterType = Enum.RaycastFilterType.Exclude
    
    local directions = {
        Front = hrp.CFrame.LookVector,
        Back = -hrp.CFrame.LookVector,
        Left = -hrp.CFrame.RightVector,
        Right = hrp.CFrame.RightVector,
    }
    
    local walls = {}
    
    for name, dir in pairs(directions) do
        local result = workspace:Raycast(
            hrp.Position,
            dir * 2.5,  -- ระยะ 2.5 studs
            params
        )
        if result then
            walls[name] = result
        end
    end
    
    return walls
end

-- ใช้งาน
local walls = detectWalls(character)
if walls.Front then
    print("มีกำแพงข้างหน้า")
end
```

---

## 39.4 Overlap Detection

ตรวจจับ Parts ที่อยู่ในพื้นที่

### GetPartBoundsInBox

```lua
-- ตรวจจับทุก Part ในกล่อง
local cframe = CFrame.new(0, 5, 0)
local size = Vector3.new(10, 10, 10)  -- กล่อง 10x10x10

local overlapParams = OverlapParams.new()
overlapParams.FilterDescendantsInstances = {workspace.IgnoreFolder}
overlapParams.FilterType = Enum.RaycastFilterType.Exclude

local parts = workspace:GetPartBoundsInBox(cframe, size, overlapParams)

for _, part in pairs(parts) do
    print("พบ Part:", part.Name)
end
```

### GetPartBoundsInRadius

```lua
-- ตรวจจับทุก Part ในวงกลม
local center = Vector3.new(0, 5, 0)
local radius = 15

local overlapParams = OverlapParams.new()

local parts = workspace:GetPartBoundsInRadius(center, radius, overlapParams)

-- หา Humanoids ในรัศมี
local function findHumanoidsInRange(center, radius, exclude)
    local overlapParams = OverlapParams.new()
    if exclude then
        overlapParams.FilterDescendantsInstances = {exclude}
        overlapParams.FilterType = Enum.RaycastFilterType.Exclude
    end
    
    local parts = workspace:GetPartBoundsInRadius(center, radius, overlapParams)
    
    local humanoids = {}
    local checkedModels = {}
    
    for _, part in pairs(parts) do
        local model = part:FindFirstAncestorOfClass("Model")
        if model and not checkedModels[model] then
            checkedModels[model] = true
            local humanoid = model:FindFirstChildOfClass("Humanoid")
            if humanoid and humanoid.Health > 0 then
                table.insert(humanoids, humanoid)
            end
        end
    end
    
    return humanoids
end

-- ระเบิด AOE Damage
local function aoeExplosion(center, radius, damage)
    local humanoids = findHumanoidsInRange(center, radius)
    
    for _, humanoid in pairs(humanoids) do
        local char = humanoid.Parent
        local hrp = char:FindFirstChild("HumanoidRootPart")
        if hrp then
            -- ความเสียหายลดลงตามระยะ
            local distance = (hrp.Position - center).Magnitude
            local falloff = 1 - (distance / radius)
            local actualDamage = damage * falloff
            
            humanoid:TakeDamage(actualDamage)
        end
    end
end
```

### GetPartsInPart

```lua
-- ตรวจจับ Parts ที่ทับซ้อนกับ Part ที่กำหนด
local triggerPart = workspace.TriggerZone

local overlapParams = OverlapParams.new()
overlapParams.FilterDescendantsInstances = {triggerPart}
overlapParams.FilterType = Enum.RaycastFilterType.Exclude

local parts = workspace:GetPartsInPart(triggerPart, overlapParams)

for _, part in pairs(parts) do
    print("อยู่ใน Zone:", part.Name)
end
```

---

## 39.5 Region3 (วิธีเก่า)

Region3 เป็นวิธีเก่ากว่า แต่ยังใช้งานได้

```lua
-- สร้าง Region3 จาก Min/Max corners
local minBound = Vector3.new(-10, 0, -10)
local maxBound = Vector3.new(10, 20, 10)
local region = Region3.new(minBound, maxBound)

-- หา Parts ในพื้นที่
local parts = workspace:FindPartsInRegion3(region, nil, 100)
-- nil = ไม่ ignore Part ไหน
-- 100 = หาสูงสุด 100 Parts

for _, part in pairs(parts) do
    print("พบใน Region:", part.Name)
end

-- ใช้กับ ignore list
local ignoreList = {workspace.IgnoreModel}
local parts2 = workspace:FindPartsInRegion3WithIgnoreList(region, ignoreList, 100)

-- ใช้กับ whitelist
local whitelist = {workspace.EnemiesFolder}
local parts3 = workspace:FindPartsInRegion3WithWhiteList(region, whitelist, 100)
```

---

## 39.6 Zone System

ระบบ Zone สำหรับตรวจจับผู้เล่นในพื้นที่

```lua
-- ModuleScript: ZoneSystem
local ZoneSystem = {}
ZoneSystem.__index = ZoneSystem

local RunService = game:GetService("RunService")
local Players = game:GetService("Players")

local activeZones = {}

function ZoneSystem.new(zonePart)
    local self = setmetatable({}, ZoneSystem)
    
    self.part = zonePart
    self.playersInside = {}
    self.onEntered = Instance.new("BindableEvent")
    self.onExited = Instance.new("BindableEvent")
    self.Entered = self.onEntered.Event
    self.Exited = self.onExited.Event
    
    -- เริ่มตรวจสอบ
    self.connection = RunService.Heartbeat:Connect(function()
        self:_checkPlayers()
    end)
    
    table.insert(activeZones, self)
    return self
end

function ZoneSystem:_checkPlayers()
    if not self.part or not self.part.Parent then return end
    
    local overlapParams = OverlapParams.new()
    overlapParams.FilterDescendantsInstances = {self.part}
    overlapParams.FilterType = Enum.RaycastFilterType.Exclude
    
    local partsInZone = workspace:GetPartsInPart(self.part, overlapParams)
    
    -- หาผู้เล่นในพื้นที่
    local currentPlayers = {}
    
    for _, part in pairs(partsInZone) do
        local character = part:FindFirstAncestorOfClass("Model")
        if character then
            local player = Players:GetPlayerFromCharacter(character)
            if player and not currentPlayers[player] then
                currentPlayers[player] = true
            end
        end
    end
    
    -- ตรวจ entered
    for player in pairs(currentPlayers) do
        if not self.playersInside[player] then
            self.playersInside[player] = true
            self.onEntered:Fire(player)
        end
    end
    
    -- ตรวจ exited
    for player in pairs(self.playersInside) do
        if not currentPlayers[player] then
            self.playersInside[player] = nil
            self.onExited:Fire(player)
        end
    end
end

function ZoneSystem:getPlayersInside()
    local list = {}
    for player in pairs(self.playersInside) do
        table.insert(list, player)
    end
    return list
end

function ZoneSystem:isPlayerInside(player)
    return self.playersInside[player] == true
end

function ZoneSystem:destroy()
    self.connection:Disconnect()
    self.onEntered:Destroy()
    self.onExited:Destroy()
end

return ZoneSystem
```

### ใช้งาน ZoneSystem

```lua
-- Script ใน ServerScriptService
local ZoneSystem = require(game.ServerScriptService.ZoneSystem)

local shopZone = ZoneSystem.new(workspace.ShopZone)

shopZone.Entered:Connect(function(player)
    print(player.Name .. " เข้า Shop Zone")
    -- เปิด Shop GUI
end)

shopZone.Exited:Connect(function(player)
    print(player.Name .. " ออกจาก Shop Zone")
    -- ปิด Shop GUI
end)

-- Damage Zone
local lavaZone = ZoneSystem.new(workspace.LavaZone)
lavaZone.Entered:Connect(function(player)
    local character = player.Character
    if not character then return end
    
    -- ดาม Damage ทุก 0.5 วินาที
    local connection
    connection = game:GetService("RunService").Heartbeat:Connect(function()
        if not lavaZone:isPlayerInside(player) then
            connection:Disconnect()
            return
        end
        
        local humanoid = character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            humanoid:TakeDamage(5)
        end
    end)
end)
```

---

## 39.7 Spatial Query

### FindFirstChildWhichIsA และ GetDescendants

```lua
-- ค้นหาใน Model
local model = workspace.NPC

-- ค้นหาตัวแรก
local humanoid = model:FindFirstChildWhichIsA("Humanoid")
local basePart = model:FindFirstChildWhichIsA("BasePart")

-- ค้นหาทุก descendants ที่เป็น type นั้น
for _, part in pairs(model:GetDescendants()) do
    if part:IsA("BasePart") then
        -- ทำอะไรกับ part
    end
end
```

### Magnitude Check

```lua
-- ตรวจสอบระยะด้วย Magnitude (ง่าย แต่กลม)
local function isNear(pos1, pos2, maxDistance)
    return (pos1 - pos2).Magnitude <= maxDistance
end

-- ตัวอย่าง: NPC ไล่ผู้เล่นในระยะ
local function updateNPC(npc, player)
    local npcPos = npc.HumanoidRootPart.Position
    local playerPos = player.Character.HumanoidRootPart.Position
    
    if isNear(npcPos, playerPos, 20) then
        -- ผู้เล่นอยู่ในระยะ 20 studs
        local humanoid = npc:FindFirstChildOfClass("Humanoid")
        humanoid:MoveTo(playerPos)
    end
end
```

---

## 39.8 ตัวอย่าง: Hitbox System

```lua
-- ModuleScript: HitboxSystem
-- ระบบ Hitbox สำหรับ melee combat

local HitboxSystem = {}
HitboxSystem.__index = HitboxSystem

local RunService = game:GetService("RunService")

function HitboxSystem.new(owner, size, offset)
    local self = setmetatable({}, HitboxSystem)
    
    self.owner = owner
    self.size = size or Vector3.new(5, 5, 5)
    self.offset = offset or Vector3.new(0, 0, 3)  -- ข้างหน้า
    self.active = false
    self.hitList = {}  -- ป้องกันโดน hit ซ้ำ
    self.onHit = Instance.new("BindableEvent")
    self.Hit = self.onHit.Event
    
    return self
end

function HitboxSystem:start(duration)
    if self.active then return end
    
    self.active = true
    self.hitList = {}
    
    -- ตรวจทุก Frame
    local connection
    connection = RunService.Heartbeat:Connect(function()
        if not self.active then
            connection:Disconnect()
            return
        end
        
        self:_check()
    end)
    
    -- หยุดหลัง duration
    if duration then
        task.delay(duration, function()
            self:stop()
        end)
    end
end

function HitboxSystem:stop()
    self.active = false
end

function HitboxSystem:_check()
    local ownerPart = self.owner:FindFirstChild("HumanoidRootPart")
    if not ownerPart then return end
    
    -- คำนวณตำแหน่ง hitbox
    local hitboxCFrame = ownerPart.CFrame * CFrame.new(self.offset)
    
    local overlapParams = OverlapParams.new()
    overlapParams.FilterDescendantsInstances = {self.owner}
    overlapParams.FilterType = Enum.RaycastFilterType.Exclude
    
    local parts = workspace:GetPartBoundsInBox(hitboxCFrame, self.size, overlapParams)
    
    for _, part in pairs(parts) do
        local model = part:FindFirstAncestorOfClass("Model")
        if model and not self.hitList[model] then
            local humanoid = model:FindFirstChildOfClass("Humanoid")
            if humanoid and humanoid.Health > 0 then
                self.hitList[model] = true
                self.onHit:Fire(humanoid, model, part)
            end
        end
    end
end

function HitboxSystem:visualize(duration)
    -- สร้าง Part แสดง hitbox (debug)
    local ownerPart = self.owner:FindFirstChild("HumanoidRootPart")
    if not ownerPart then return end
    
    local viz = Instance.new("Part")
    viz.Name = "HitboxViz"
    viz.Size = self.size
    viz.Anchored = true
    viz.CanCollide = false
    viz.Transparency = 0.7
    viz.BrickColor = BrickColor.new("Bright red")
    viz.CFrame = ownerPart.CFrame * CFrame.new(self.offset)
    viz.Parent = workspace
    
    local conn = RunService.Heartbeat:Connect(function()
        if not ownerPart.Parent then
            conn:Disconnect()
            viz:Destroy()
            return
        end
        viz.CFrame = ownerPart.CFrame * CFrame.new(self.offset)
    end)
    
    task.delay(duration or 1, function()
        conn:Disconnect()
        viz:Destroy()
    end)
end

function HitboxSystem:destroy()
    self.active = false
    self.onHit:Destroy()
end

return HitboxSystem
```

### ใช้งาน HitboxSystem

```lua
-- Script ใน SwordTool
local HitboxSystem = require(game.ServerScriptService.HitboxSystem)

local tool = script.Parent
local damage = 30

tool.Activated:Connect(function()
    local character = tool.Parent
    if not character then return end
    
    -- สร้าง hitbox ขนาด 5x5x5 ข้างหน้า 3 studs
    local hitbox = HitboxSystem.new(character, Vector3.new(5, 5, 5), Vector3.new(0, 0, 3))
    
    -- เมื่อโดน
    hitbox.Hit:Connect(function(humanoid, model, hitPart)
        humanoid:TakeDamage(damage)
        print("โดน:", model.Name, "ที่ part:", hitPart.Name)
    end)
    
    -- เปิด hitbox 0.3 วินาที
    hitbox:start(0.3)
    
    -- Debug visualize
    hitbox:visualize(0.3)
    
    -- cleanup หลัง 1 วินาที
    task.delay(1, function()
        hitbox:destroy()
    end)
end)
```

---

## 39.9 ตัวอย่าง: Proximity System

```lua
-- ModuleScript: ProximitySystem
-- ตรวจจับผู้เล่นใกล้เคียงและทำ action

local ProximitySystem = {}
ProximitySystem.__index = ProximitySystem

local RunService = game:GetService("RunService")
local Players = game:GetService("Players")

-- ProximityPrompt (สำเร็จรูปของ Roblox)
function ProximitySystem.createPrompt(part, actionText, callback)
    local prompt = Instance.new("ProximityPrompt")
    prompt.ActionText = actionText
    prompt.ObjectText = part.Name
    prompt.MaxActivationDistance = 10
    prompt.HoldDuration = 0  -- กด หรือ 0 = กดทันที
    prompt.Parent = part
    
    prompt.Triggered:Connect(function(player)
        callback(player)
    end)
    
    return prompt
end

-- Custom Proximity ที่ตรวจสอบเองและแสดง GUI
function ProximitySystem.new(part, radius)
    local self = setmetatable({}, ProximitySystem)
    
    self.part = part
    self.radius = radius or 10
    self.playersNear = {}
    self.onPlayerEntered = Instance.new("BindableEvent")
    self.onPlayerExited = Instance.new("BindableEvent")
    self.PlayerEntered = self.onPlayerEntered.Event
    self.PlayerExited = self.onPlayerExited.Event
    
    self.connection = RunService.Heartbeat:Connect(function()
        self:_update()
    end)
    
    return self
end

function ProximitySystem:_update()
    if not self.part or not self.part.Parent then return end
    
    local partPos = self.part.Position
    local near = {}
    
    for _, player in pairs(Players:GetPlayers()) do
        local character = player.Character
        if character then
            local hrp = character:FindFirstChild("HumanoidRootPart")
            if hrp then
                local distance = (hrp.Position - partPos).Magnitude
                if distance <= self.radius then
                    near[player] = distance
                end
            end
        end
    end
    
    -- Entered
    for player in pairs(near) do
        if not self.playersNear[player] then
            self.playersNear[player] = near[player]
            self.onPlayerEntered:Fire(player, near[player])
        end
    end
    
    -- Exited
    for player in pairs(self.playersNear) do
        if not near[player] then
            self.playersNear[player] = nil
            self.onPlayerExited:Fire(player)
        end
    end
end

function ProximitySystem:getNearestPlayer()
    local nearest = nil
    local nearestDist = math.huge
    
    for player, dist in pairs(self.playersNear) do
        if dist < nearestDist then
            nearest = player
            nearestDist = dist
        end
    end
    
    return nearest, nearestDist
end

function ProximitySystem:destroy()
    self.connection:Disconnect()
    self.onPlayerEntered:Destroy()
    self.onPlayerExited:Destroy()
end

return ProximitySystem
```

---

## 39.10 Spatial Partitioning (Performance Optimization)

```lua
-- ModuleScript: SpatialGrid
-- แบ่งพื้นที่เป็น Grid เพื่อค้นหาได้เร็วขึ้น

local SpatialGrid = {}
SpatialGrid.__index = SpatialGrid

function SpatialGrid.new(cellSize)
    local self = setmetatable({}, SpatialGrid)
    self.cellSize = cellSize or 20
    self.grid = {}
    self.objectCells = {}  -- track ว่าแต่ละ object อยู่ cell ไหน
    return self
end

function SpatialGrid:_getKey(x, z)
    local cx = math.floor(x / self.cellSize)
    local cz = math.floor(z / self.cellSize)
    return cx .. "," .. cz
end

function SpatialGrid:insert(object, position)
    local key = self:_getKey(position.X, position.Z)
    
    if not self.grid[key] then
        self.grid[key] = {}
    end
    
    table.insert(self.grid[key], object)
    self.objectCells[object] = key
end

function SpatialGrid:update(object, newPosition)
    -- ลบจาก cell เก่า
    self:remove(object)
    -- ใส่ cell ใหม่
    self:insert(object, newPosition)
end

function SpatialGrid:remove(object)
    local key = self.objectCells[object]
    if key and self.grid[key] then
        for i, obj in ipairs(self.grid[key]) do
            if obj == object then
                table.remove(self.grid[key], i)
                break
            end
        end
        self.objectCells[object] = nil
    end
end

function SpatialGrid:getNearby(position, radius)
    local results = {}
    
    -- คำนวณ cells ที่ต้องตรวจ
    local cellRadius = math.ceil(radius / self.cellSize) + 1
    local cx = math.floor(position.X / self.cellSize)
    local cz = math.floor(position.Z / self.cellSize)
    
    for dx = -cellRadius, cellRadius do
        for dz = -cellRadius, cellRadius do
            local key = (cx + dx) .. "," .. (cz + dz)
            if self.grid[key] then
                for _, obj in ipairs(self.grid[key]) do
                    table.insert(results, obj)
                end
            end
        end
    end
    
    return results
end

return SpatialGrid
```

---

## 39.11 ตัวอย่าง: Enemy AI Collision

```lua
-- Script: EnemyAI
-- NPC ที่ใช้ Collision Detection

local PathfindingService = game:GetService("PathfindingService")
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")

local enemyModel = script.Parent
local humanoid = enemyModel:WaitForChild("Humanoid")
local hrp = enemyModel:WaitForChild("HumanoidRootPart")

local DETECT_RANGE = 30   -- ระยะตรวจจับ
local ATTACK_RANGE = 5    -- ระยะโจมตี
local DAMAGE = 10
local ATTACK_COOLDOWN = 1

local lastAttackTime = 0
local target = nil

-- หาผู้เล่นในระยะ
local function findTarget()
    local nearest = nil
    local nearestDist = DETECT_RANGE
    
    for _, player in pairs(Players:GetPlayers()) do
        local char = player.Character
        if char then
            local playerHRP = char:FindFirstChild("HumanoidRootPart")
            local playerHum = char:FindFirstChildOfClass("Humanoid")
            
            if playerHRP and playerHum and playerHum.Health > 0 then
                local dist = (playerHRP.Position - hrp.Position).Magnitude
                
                if dist < nearestDist then
                    -- ตรวจสอบว่ามีสิ่งกีดขวางหรือไม่ (Line of Sight)
                    local params = RaycastParams.new()
                    params.FilterDescendantsInstances = {enemyModel, char}
                    params.FilterType = Enum.RaycastFilterType.Exclude
                    
                    local direction = (playerHRP.Position - hrp.Position)
                    local result = workspace:Raycast(hrp.Position, direction, params)
                    
                    if not result then
                        -- ไม่มีสิ่งกีดขวาง = มองเห็นผู้เล่น
                        nearest = player
                        nearestDist = dist
                    end
                end
            end
        end
    end
    
    return nearest, nearestDist
end

-- โจมตี
local function attack(player)
    local now = tick()
    if now - lastAttackTime < ATTACK_COOLDOWN then return end
    lastAttackTime = now
    
    local char = player.Character
    if not char then return end
    
    local playerHum = char:FindFirstChildOfClass("Humanoid")
    if playerHum and playerHum.Health > 0 then
        playerHum:TakeDamage(DAMAGE)
    end
end

-- Main Loop
RunService.Heartbeat:Connect(function()
    if humanoid.Health <= 0 then return end
    
    -- หา target
    target, _ = findTarget()
    
    if target then
        local char = target.Character
        if not char then return end
        
        local playerHRP = char:FindFirstChild("HumanoidRootPart")
        if not playerHRP then return end
        
        local distance = (playerHRP.Position - hrp.Position).Magnitude
        
        if distance <= ATTACK_RANGE then
            -- โจมตี
            attack(target)
        else
            -- เดินหา
            humanoid:MoveTo(playerHRP.Position)
        end
    else
        -- Idle - เดินวนไปมา
    end
end)
```

---

## 39.12 Debug Visualization

```lua
-- ModuleScript: DebugDraw
-- ช่วย visualize collision areas

local DebugDraw = {}

local function createTempPart(cframe, size, color, transparency, duration)
    local part = Instance.new("Part")
    part.Anchored = true
    part.CanCollide = false
    part.CFrame = cframe
    part.Size = size
    part.BrickColor = BrickColor.new(color)
    part.Transparency = transparency
    part.Parent = workspace
    
    game:GetService("Debris"):AddItem(part, duration or 0.1)
    return part
end

-- วาดกล่อง
function DebugDraw.box(cframe, size, color, duration)
    return createTempPart(cframe, size, color or "Bright red", 0.7, duration)
end

-- วาดวงกลม (ใช้ Part หลายชิ้น)
function DebugDraw.sphere(center, radius, color, duration)
    return createTempPart(
        CFrame.new(center),
        Vector3.new(radius * 2, radius * 2, radius * 2),
        color or "Bright blue",
        0.7,
        duration
    )
end

-- วาด Ray
function DebugDraw.ray(origin, direction, color, duration)
    local length = direction.Magnitude
    local center = origin + direction / 2
    local cf = CFrame.new(center, origin + direction)
    
    return createTempPart(
        cf,
        Vector3.new(0.1, 0.1, length),
        color or "Bright green",
        0.3,
        duration
    )
end

-- วาดจุด
function DebugDraw.point(position, color, duration)
    return createTempPart(
        CFrame.new(position),
        Vector3.new(0.5, 0.5, 0.5),
        color or "Bright yellow",
        0,
        duration
    )
end

return DebugDraw
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Damage Zone
สร้าง Damage Zone ที่:
- กำจัด HP ผู้เล่นเมื่ออยู่ในพื้นที่
- แสดงอนุภาคเตือน
- มี grace period 1 วินาที
- Lava Zone ที่ความร้อนเพิ่มขึ้นยิ่งอยู่นาน

### แบบฝึกหัดที่ 2: Proximity Shop
สร้างระบบ Shop ที่:
- แสดง Prompt เมื่อใกล้ NPC
- เปิด GUI เมื่อคุย
- ซื้อของโดยใช้ proximity
- ปิดเมื่อออกนอกรัศมี

### แบบฝึกหัดที่ 3: AOE Spell
สร้าง AOE Spell ที่:
- เลือกพื้นที่เป้าหมาย (วงกลมบนพื้น)
- ตรวจจับ Enemies ในพื้นที่
- ทำ Damage ลดตามระยะ
- Visualize ผลกระทบด้วย particles

### แบบฝึกหัดที่ 4: Stealth System
สร้างระบบ Stealth ที่:
- ตรวจ Line of Sight ระหว่าง NPC กับผู้เล่น
- ตรวจว่าผู้เล่นอยู่หลัง NPC หรือไม่
- Noise Detection: ตรวจเสียงฝีเท้า
- Alert Level ที่เพิ่มขึ้นเมื่อโดนเห็น

---

## สรุป

ในบทนี้เราได้เรียนรู้:
- **CanCollide/CollisionGroups**: ควบคุมการชนระหว่าง Parts
- **Raycast**: ยิงเส้นตรงตรวจจับ
- **Overlap Detection**: หา Parts ในพื้นที่
- **Zone System**: ตรวจจับผู้เล่นเข้า/ออกพื้นที่
- **HitboxSystem**: Combat hitbox สำหรับ melee
- **ProximitySystem**: ตรวจจับความใกล้
- **Debug Visualization**: แสดงผล collision ระหว่าง dev

---

## อ้างอิง
- [PhysicsService](https://create.roblox.com/docs/reference/engine/classes/PhysicsService)
- [RaycastParams](https://create.roblox.com/docs/reference/engine/datatypes/RaycastParams)
- [OverlapParams](https://create.roblox.com/docs/reference/engine/datatypes/OverlapParams)
- [Workspace:Raycast](https://create.roblox.com/docs/reference/engine/classes/WorldRoot#Raycast)
- [Workspace:GetPartBoundsInBox](https://create.roblox.com/docs/reference/engine/classes/WorldRoot#GetPartBoundsInBox)
- [ProximityPrompt](https://create.roblox.com/docs/reference/engine/classes/ProximityPrompt)
