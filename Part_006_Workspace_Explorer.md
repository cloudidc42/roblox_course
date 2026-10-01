# ตอนที่ 6: Workspace และ Explorer Panel
## Part 6: Workspace and Explorer Panel

---

**ระดับ:** พื้นฐาน (Beginner)  
**เวลาเรียน:** 60-90 นาที  
**ข้อกำหนดเบื้องต้น:** ตอนที่ 1-5

---

## 🎯 วัตถุประสงค์การเรียนรู้

เมื่อเรียนจบตอนนี้ คุณจะสามารถ:
1. เข้าใจโครงสร้าง DataModel ของ Roblox
2. รู้จักและเข้าใจ Services ทั้งหมด
3. ใช้งาน Explorer Panel ได้อย่างมืออาชีพ
4. จัดการ Instance Hierarchy ด้วยโค้ด
5. เข้าใจความแตกต่างระหว่าง Services

---

## 1. DataModel คืออะไร?

### 1.1 ภาพรวม

**DataModel** (เรียกว่า `game`) คือ Root Object ของทุกอย่างใน Roblox:

```lua
-- game คือ DataModel
print(game.ClassName)  -- "DataModel"
print(game.Name)       -- "Game"

-- game เป็น Parent ของทุก Service
print(game.Workspace.ClassName)  -- "Workspace"
```

### 1.2 โครงสร้าง DataModel

```
game (DataModel)
├── Workspace                    -- สภาพแวดล้อม 3D
├── Players                     -- ข้อมูล Players
├── Lighting                    -- ระบบแสงสว่าง
├── ReplicatedFirst             -- โหลดก่อนอื่น
├── ReplicatedStorage           -- แชร์ Server-Client
├── ServerScriptService         -- Scripts บน Server
├── ServerStorage               -- Storage บน Server
├── StarterGui                  -- GUI เริ่มต้น
├── StarterPack                 -- Items เริ่มต้น
├── StarterPlayer               -- Player Settings
├── Teams                       -- ทีม
├── SoundService                -- ระบบเสียง
├── Chat                        -- ระบบแชท
├── LocalizationService         -- ภาษา
├── AnalyticsService            -- Analytics
└── ...
```

---

## 2. Services สำคัญ

### 2.1 Workspace

**Workspace** คือสภาพแวดล้อม 3D หลักของเกม:

```lua
-- เข้าถึง Workspace
local workspace = game.Workspace
-- หรือ
local workspace = workspace  -- ตัวแปร Global
-- หรือ
local workspace = game:GetService("Workspace")

-- Properties ของ Workspace
workspace.Gravity = 196.2         -- แรงโน้มถ่วง (ค่าเริ่มต้น)
workspace.FallenPartsDestroyHeight = -500  -- ความสูงที่ Parts ถูกลบ
workspace.FilteringEnabled = true  -- เปิดใช้ Filtering (ความปลอดภัย)

-- การค้นหาใน Workspace
local baseplate = workspace:FindFirstChild("Baseplate")
local allParts = workspace:GetDescendants()
```

### 2.2 Players Service

```lua
-- Players Service จัดการผู้เล่นทั้งหมด
local Players = game:GetService("Players")

-- ดูผู้เล่นทั้งหมด
local allPlayers = Players:GetPlayers()
for _, player in ipairs(allPlayers) do
    print(player.Name, "- UserID:", player.UserId)
end

-- เมื่อผู้เล่นเข้า
Players.PlayerAdded:Connect(function(player)
    print(player.Name .. " เข้าเกม!")
    
    -- เมื่อ Character โหลด
    player.CharacterAdded:Connect(function(character)
        print(player.Name .. "'s character loaded!")
    end)
end)

-- เมื่อผู้เล่นออก
Players.PlayerRemoving:Connect(function(player)
    print(player.Name .. " ออกจากเกม!")
end)

-- ดูจำนวนผู้เล่น
print("ผู้เล่นปัจจุบัน:", #Players:GetPlayers())
```

### 2.3 ReplicatedStorage

```lua
-- ReplicatedStorage: ข้อมูลที่แชร์ระหว่าง Server และ Client
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- วาง RemoteEvents ที่นี่
local remoteEvent = Instance.new("RemoteEvent")
remoteEvent.Name = "MyEvent"
remoteEvent.Parent = ReplicatedStorage

-- วาง ModuleScripts ที่ใช้ร่วมกัน
local module = Instance.new("ModuleScript")
module.Name = "SharedModule"
module.Parent = ReplicatedStorage

-- Client สามารถอ่านได้
-- Server สามารถอ่านและเขียนได้
```

### 2.4 ServerScriptService

```lua
-- ServerScriptService: Scripts ที่รันบน Server เท่านั้น
local ServerScriptService = game:GetService("ServerScriptService")

-- ใส่ Server Scripts ที่นี่
-- Client ไม่สามารถ Access Scripts ใน ServerScriptService ได้
-- ปลอดภัยจาก Exploiters
```

### 2.5 ServerStorage

```lua
-- ServerStorage: Storage ที่ Client ไม่เห็น
local ServerStorage = game:GetService("ServerStorage")

-- ใส่:
-- - Models ที่จะ Clone มาใช้
-- - Weapons, Items
-- - Server-only Data
-- - NPCs

-- ตัวอย่าง Clone Model จาก ServerStorage
local enemyTemplate = ServerStorage:FindFirstChild("EnemyModel")
if enemyTemplate then
    local enemy = enemyTemplate:Clone()
    enemy.Parent = workspace
    enemy:SetPrimaryPartCFrame(CFrame.new(0, 5, 0))
end
```

### 2.6 StarterGui

```lua
-- StarterGui: GUI ที่จะถูก Copy ไปให้ผู้เล่นทุกคน
local StarterGui = game:GetService("StarterGui")

-- GUI ที่ใส่ใน StarterGui จะ:
-- 1. ถูก Copy ไปใน PlayerGui ของผู้เล่น
-- 2. Reset เมื่อ Respawn (ถ้าตั้งค่าไว้)

-- ดู PlayerGui ของผู้เล่น (ต้องใช้ใน LocalScript)
local Players = game:GetService("Players")
local player = Players.LocalPlayer
local playerGui = player.PlayerGui

-- ดู GUI ทั้งหมดของผู้เล่น
for _, gui in pairs(playerGui:GetChildren()) do
    print("GUI:", gui.Name)
end
```

### 2.7 StarterPlayer

```lua
-- StarterPlayer: ตั้งค่าเริ่มต้นของ Player Character

-- StarterPlayerScripts - Scripts ที่รันเมื่อเข้าเกม (LocalScript)
-- StarterCharacterScripts - Scripts ที่รันเมื่อ Character Spawn

-- ตัวอย่างใน StarterCharacterScripts:
-- ชื่อไฟล์: SpeedBoost (LocalScript)
local character = script.Parent  -- Character ของผู้เล่น
local humanoid = character:WaitForChild("Humanoid")

-- เพิ่มความเร็ว
humanoid.WalkSpeed = 25
humanoid.JumpPower = 80
```

### 2.8 Lighting Service

```lua
-- Lighting: ควบคุมแสงสว่างของโลก
local Lighting = game:GetService("Lighting")

-- Properties พื้นฐาน
Lighting.Ambient = Color3.fromRGB(70, 70, 70)     -- แสง ambient
Lighting.Brightness = 2                            -- ความสว่าง
Lighting.ClockTime = 14                            -- เวลา (0-24)
Lighting.GeographicLatitude = 41.7                -- ละติจูด
Lighting.GlobalShadows = true                     -- เงาทั่วโลก

-- เปลี่ยนเวลา
Lighting.ClockTime = 6    -- เช้า
Lighting.ClockTime = 12   -- เที่ยง
Lighting.ClockTime = 18   -- เย็น
Lighting.ClockTime = 0    -- กลางคืน

-- Fog
Lighting.FogEnd = 500     -- ระยะสิ้นสุด Fog
Lighting.FogStart = 100   -- ระยะเริ่ม Fog
Lighting.FogColor = Color3.fromRGB(200, 200, 200)  -- สี Fog
```

### 2.9 SoundService

```lua
-- SoundService: ระบบเสียง Global
local SoundService = game:GetService("SoundService")

-- Settings
SoundService.AmbientReverb = Enum.ReverbType.NoReverb
SoundService.DistanceFactor = 1       -- ระยะ 3D Sound
SoundService.DopplerScale = 1         -- Doppler Effect
SoundService.RolloffScale = 1         -- ความดังตามระยะ

-- เล่นเสียง Background Music
local bgm = Instance.new("Sound")
bgm.SoundId = "rbxassetid://1234567"
bgm.Volume = 0.5
bgm.Looped = true
bgm.Parent = SoundService
bgm:Play()
```

---

## 3. Explorer Panel อย่างละเอียด

### 3.1 Icons ใน Explorer

```
🎮 DataModel (game)
├── 🌍 Workspace
├── 👥 Players
├── 💡 Lighting
├── 🔄 ReplicatedFirst
├── 🔄 ReplicatedStorage
├── 💾 ServerScriptService
├── 💽 ServerStorage
├── 🎯 StarterGui
├── 🎒 StarterPack
└── 👤 StarterPlayer
    ├── 📜 StarterPlayerScripts
    └── 📜 StarterCharacterScripts

Object Icons:
🟦 Part/BasePart
📜 Script
📄 LocalScript
📦 ModuleScript
🖼️ Model
📷 Camera
💡 Light (PointLight, etc.)
🔊 Sound
🎨 Decal/Texture
📁 Folder
🎭 SpecialMesh
💥 Particle Emitter
```

### 3.2 การ Filter ใน Explorer

```
- พิมพ์ชื่อในช่อง Search บนสุด
- Explorer จะ Filter เฉพาะที่ตรง
- กด Escape เพื่อล้าง Filter
```

### 3.3 การ Sort ใน Explorer

```
Objects ใน Explorer เรียงตาม:
1. ตำแหน่งใน Hierarchy (Parent-Child)
2. ลำดับที่เพิ่มเข้ามา
```

### 3.4 Context Menu (คลิกขวา)

```
Context Menu ของ Object:
├── Cut                 (Ctrl+X)
├── Copy                (Ctrl+C)
├── Paste               (Ctrl+V)
├── Paste Into          (Ctrl+Shift+V)
├── Duplicate           (Ctrl+D)
├── Delete              (Delete)
├── Rename              (F2)
├── ─────────────────────────────
├── Group               (Ctrl+G)
├── Ungroup             (Ctrl+Shift+G)
├── ─────────────────────────────
├── Select Children
├── Select Descendants
├── ─────────────────────────────
├── Insert Object       (Ctrl+I)
├── Open Script         (สำหรับ Script)
├── ─────────────────────────────
├── Zoom To             (Z)
└── Copy Full Name
```

---

## 4. Instance Hierarchy Management

### 4.1 Parent-Child Relationship

```lua
-- ทุก Instance มี Parent (ยกเว้น game)
local part = Instance.new("Part")
print(part.Parent)  -- nil (ยังไม่ได้กำหนด Parent)

part.Parent = workspace
print(part.Parent)  -- workspace

-- เปลี่ยน Parent ย้าย Object
part.Parent = game.ServerStorage  -- ย้ายไป ServerStorage
```

### 4.2 GetChildren() vs GetDescendants()

```lua
local folder = workspace.MyFolder  -- สมมติมี Folder ชื่อ MyFolder

-- GetChildren() - ลูกโดยตรงเท่านั้น
local children = folder:GetChildren()
print("Direct children:", #children)

for _, child in pairs(children) do
    print("  -", child.Name, "(", child.ClassName, ")")
end

-- GetDescendants() - ทุก Object ในลำดับชั้น
local descendants = folder:GetDescendants()
print("\nAll descendants:", #descendants)

for _, desc in pairs(descendants) do
    -- แสดงพร้อม Indentation
    local depth = 0
    local p = desc.Parent
    while p ~= folder do
        depth = depth + 1
        p = p.Parent
    end
    print(string.rep("  ", depth) .. "- " .. desc.Name)
end
```

### 4.3 FindFirstChild() Methods

```lua
local parent = workspace

-- FindFirstChild(name) - หาลูกโดยตรงตามชื่อ
local part = parent:FindFirstChild("Baseplate")
if part then
    print("Found:", part.Name)
end

-- FindFirstChild(name, recursive) - หาแบบ Recursive
local deepPart = parent:FindFirstChild("SomePart", true)

-- FindFirstChildOfClass(className) - หาตาม Class
local anyPart = parent:FindFirstChildOfClass("Part")
local anyModel = parent:FindFirstChildOfClass("Model")

-- FindFirstChildWhichIsA(className) - หาตาม IsA (รวม Subclasses)
local anyBasePart = parent:FindFirstChildWhichIsA("BasePart")
-- จะหา Part, MeshPart, UnionOperation, etc.

-- FindFirstAncestorOfClass - หา Ancestor (Parent ขึ้นไป)
local script = script
local model = script:FindFirstAncestorOfClass("Model")
```

### 4.4 WaitForChild()

```lua
-- WaitForChild - รอจนกว่า Child จะมีอยู่
-- สำคัญมากใน LocalScript!

local Players = game:GetService("Players")
local player = Players.LocalPlayer

-- รอ Character โหลด
local character = player.Character or player.CharacterAdded:Wait()

-- รอ HumanoidRootPart
local rootPart = character:WaitForChild("HumanoidRootPart")

-- รอด้วย Timeout (5 วินาที)
local gui = player.PlayerGui:WaitForChild("MainGui", 5)
if not gui then
    warn("MainGui ไม่โหลดใน 5 วินาที!")
end

-- รอ RemoteEvent ใน ReplicatedStorage
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local remote = ReplicatedStorage:WaitForChild("MyRemoteEvent")
```

---

## 5. Instance Creation และ Destruction

### 5.1 Instance.new()

```lua
-- สร้าง Instance ใหม่
local part = Instance.new("Part")
-- หรือ
local part = Instance.new("Part", workspace)  -- กำหนด Parent ทันที

-- สร้าง Instances ต่างๆ
local folder = Instance.new("Folder")
local model = Instance.new("Model")
local script = Instance.new("Script")
local localScript = Instance.new("LocalScript")
local module = Instance.new("ModuleScript")
local remoteEvent = Instance.new("RemoteEvent")
local remoteFn = Instance.new("RemoteFunction")
local bindable = Instance.new("BindableEvent")
local sound = Instance.new("Sound")
local gui = Instance.new("ScreenGui")
```

### 5.2 Clone()

```lua
-- Clone() สร้าง Copy ของ Instance (รวม Children ทั้งหมด)
local template = ServerStorage:FindFirstChild("EnemyModel")
if template then
    local enemy = template:Clone()
    enemy.Name = "Enemy_1"
    enemy.Parent = workspace
    
    -- Clone ได้หลายครั้ง
    for i = 1, 5 do
        local clone = template:Clone()
        clone.Name = "Enemy_" .. i
        clone:SetPrimaryPartCFrame(CFrame.new(i * 10, 0, 0))
        clone.Parent = workspace
    end
end
```

### 5.3 Destroy()

```lua
-- Destroy() ลบ Instance และ Children ทั้งหมด
local part = workspace.MyPart
part:Destroy()  -- ลบทันที

-- Destroy หลังจาก Delay
local function destroyAfter(instance, delay)
    wait(delay)
    if instance and instance.Parent then
        instance:Destroy()
    end
end

local bullet = Instance.new("Part")
bullet.Parent = workspace
destroyAfter(bullet, 5)  -- ลบใน 5 วินาที

-- ใช้ task.delay() ที่ดีกว่า wait()
task.delay(5, function()
    if bullet and bullet.Parent then
        bullet:Destroy()
    end
end)
```

---

## 6. Folder Organization

### 6.1 การใช้ Folders

```lua
-- Folders ช่วยจัดระเบียบ Objects
local function setupWorkspaceOrganization()
    local workspace = game.Workspace
    
    -- สร้าง Folders
    local mapFolder = Instance.new("Folder")
    mapFolder.Name = "Map"
    mapFolder.Parent = workspace
    
    local enemiesFolder = Instance.new("Folder")
    enemiesFolder.Name = "Enemies"
    enemiesFolder.Parent = workspace
    
    local effectsFolder = Instance.new("Folder")
    effectsFolder.Name = "Effects"
    effectsFolder.Parent = workspace
    
    local pickupsFolder = Instance.new("Folder")
    pickupsFolder.Name = "Pickups"
    pickupsFolder.Parent = workspace
    
    print("Workspace organization created!")
    return {
        map = mapFolder,
        enemies = enemiesFolder,
        effects = effectsFolder,
        pickups = pickupsFolder
    }
end

local folders = setupWorkspaceOrganization()

-- ใช้ Folders
local part = Instance.new("Part")
part.Parent = folders.map  -- ใส่ใน Map folder
```

### 6.2 โครงสร้างที่แนะนำ

```
Workspace
├── Map
│   ├── Terrain_Parts
│   ├── Buildings
│   └── Decorations
├── Enemies
│   ├── Active
│   └── Spawners
├── Effects
│   ├── Particles
│   └── Sounds
└── Pickups
    ├── Coins
    └── PowerUps

ReplicatedStorage
├── Remotes
│   ├── RemoteEvents
│   └── RemoteFunctions
├── Modules
└── Assets

ServerStorage
├── Templates
│   ├── Enemies
│   └── Items
└── PrivateData

StarterGui
├── MainHud
├── PauseMenu
└── ShopGui
```

---

## 7. การใช้ CollectionService

### 7.1 CollectionService คืออะไร?

**CollectionService** ช่วย Tag Objects และค้นหาด้วย Tag:

```lua
local CollectionService = game:GetService("CollectionService")

-- เพิ่ม Tag ให้ Instance
local part = workspace.MyPart
CollectionService:AddTag(part, "Collidable")
CollectionService:AddTag(part, "Destroyable")

-- ดู Tags ทั้งหมดของ Instance
local tags = CollectionService:GetTags(part)
for _, tag in ipairs(tags) do
    print("Tag:", tag)
end

-- ค้นหาทุก Instance ที่มี Tag นี้
local allCollidables = CollectionService:GetTagged("Collidable")
print("จำนวน Collidable objects:", #allCollidables)

-- ลบ Tag
CollectionService:RemoveTag(part, "Collidable")

-- ตรวจสอบว่ามี Tag
if CollectionService:HasTag(part, "Destroyable") then
    print("Part นี้ Destroyable")
end
```

### 7.2 ตัวอย่างการใช้ CollectionService

```lua
-- ใช้ Tag เพื่อจัดการ Kill Zones
local CollectionService = game:GetService("CollectionService")

-- ทุก Part ที่มี Tag "KillZone" จะทำลาย Character
CollectionService:GetInstanceAddedSignal("KillZone"):Connect(function(part)
    part.Touched:Connect(function(hit)
        local character = hit.Parent
        local humanoid = character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            humanoid.Health = 0  -- Kill
        end
    end)
end)

-- เพิ่ม Kill Zone ใหม่
local killPart = Instance.new("Part")
killPart.Name = "LavaFloor"
killPart.Size = Vector3.new(20, 1, 20)
killPart.BrickColor = BrickColor.new("Bright orange")
killPart.Material = Enum.Material.Neon
killPart.Anchored = true
killPart.CanCollide = false
killPart.Parent = workspace

-- เพิ่ม Tag
CollectionService:AddTag(killPart, "KillZone")
```

---

## 8. Instance Events ที่เกี่ยวกับ Hierarchy

### 8.1 ChildAdded และ ChildRemoved

```lua
-- ดักจับเมื่อมี Child เพิ่ม/ลบ
workspace.ChildAdded:Connect(function(child)
    print("เพิ่ม:", child.Name, "(", child.ClassName, ")")
end)

workspace.ChildRemoved:Connect(function(child)
    print("ลบ:", child.Name)
end)

-- ทดสอบ
local part = Instance.new("Part")
part.Parent = workspace  -- จะ trigger ChildAdded
wait(2)
part:Destroy()          -- จะ trigger ChildRemoved
```

### 8.2 DescendantAdded และ DescendantRemoving

```lua
-- ดักจับ Descendants ทั้งหมด (ไม่ใช่แค่ Direct Children)
workspace.DescendantAdded:Connect(function(desc)
    if desc:IsA("BasePart") then
        print("New Part added:", desc.Name)
    end
end)

workspace.DescendantRemoving:Connect(function(desc)
    if desc:IsA("BasePart") then
        print("Part removing:", desc.Name)
    end
end)
```

### 8.3 AncestryChanged

```lua
-- เมื่อ Instance ถูกย้าย Parent
local part = Instance.new("Part")

part.AncestryChanged:Connect(function(child, parent)
    print("Ancestry changed!")
    print("Child:", child.Name)
    print("New parent:", parent and parent.Name or "nil")
end)

part.Parent = workspace           -- trigger AncestryChanged
wait(1)
part.Parent = game.ServerStorage  -- trigger อีกครั้ง
```

---

## 9. ตัวอย่างโปรแกรม: Workspace Manager

```lua
-- Script: WorkspaceManager
-- วางใน: ServerScriptService

-- ===== WorkspaceManager Module =====
local WorkspaceManager = {}
WorkspaceManager.__index = WorkspaceManager

-- สร้าง WorkspaceManager ใหม่
function WorkspaceManager.new()
    local self = setmetatable({}, WorkspaceManager)
    self.folders = {}
    self:_setupFolders()
    return self
end

-- ตั้งค่า Folders
function WorkspaceManager:_setupFolders()
    local folderNames = {"Map", "Enemies", "Effects", "Pickups", "Decorations"}
    
    for _, name in ipairs(folderNames) do
        -- ตรวจสอบว่ามีอยู่แล้ว
        local existing = workspace:FindFirstChild(name)
        if existing then
            self.folders[name] = existing
        else
            local folder = Instance.new("Folder")
            folder.Name = name
            folder.Parent = workspace
            self.folders[name] = folder
            print("Created folder:", name)
        end
    end
end

-- เพิ่ม Object ไปยัง Folder ที่กำหนด
function WorkspaceManager:AddToFolder(object, folderName)
    local folder = self.folders[folderName]
    if folder then
        object.Parent = folder
        return true
    else
        warn("Folder not found:", folderName)
        return false
    end
end

-- ดู Object ทั้งหมดใน Folder
function WorkspaceManager:GetObjectsInFolder(folderName)
    local folder = self.folders[folderName]
    if folder then
        return folder:GetDescendants()
    end
    return {}
end

-- นับ Objects ใน Folder
function WorkspaceManager:CountObjectsInFolder(folderName)
    return #self:GetObjectsInFolder(folderName)
end

-- ลบทุก Object ใน Folder
function WorkspaceManager:ClearFolder(folderName)
    local folder = self.folders[folderName]
    if folder then
        for _, child in pairs(folder:GetChildren()) do
            child:Destroy()
        end
        print("Cleared folder:", folderName)
    end
end

-- แสดงสถิติ Workspace
function WorkspaceManager:PrintStats()
    print("\n=== Workspace Statistics ===")
    for name, folder in pairs(self.folders) do
        local count = #folder:GetDescendants()
        print("  " .. name .. ":", count, "objects")
    end
    print("===========================\n")
end

-- ===== ใช้งาน WorkspaceManager =====
local wsm = WorkspaceManager.new()

-- สร้าง Parts และเพิ่มไปยัง Folders
for i = 1, 5 do
    local part = Instance.new("Part")
    part.Name = "MapPart_" .. i
    part.Size = Vector3.new(4, 2, 4)
    part.Position = Vector3.new(i * 6, 1, 0)
    part.BrickColor = BrickColor.new("Medium stone grey")
    part.Anchored = true
    wsm:AddToFolder(part, "Map")
end

for i = 1, 3 do
    local enemy = Instance.new("Model")
    enemy.Name = "Enemy_" .. i
    local enemyBody = Instance.new("Part")
    enemyBody.Size = Vector3.new(2, 4, 2)
    enemyBody.Position = Vector3.new(i * 8, 2, 10)
    enemyBody.BrickColor = BrickColor.new("Bright red")
    enemyBody.Anchored = true
    enemyBody.Parent = enemy
    wsm:AddToFolder(enemy, "Enemies")
end

-- แสดงสถิติ
wsm:PrintStats()
```

---

## 10. Performance Tips สำหรับ Workspace

### 10.1 ลด Part Count

```lua
-- ตรวจสอบจำนวน Parts
local function countParts(parent)
    local count = 0
    for _, desc in pairs(parent:GetDescendants()) do
        if desc:IsA("BasePart") then
            count = count + 1
        end
    end
    return count
end

print("Total Parts:", countParts(workspace))
-- ควรต่ำกว่า 10,000 สำหรับ Performance ที่ดี
```

### 10.2 Union Parts ที่ไม่เคลื่อนไหว

```
ลด Part Count โดย:
1. Union Parts ที่ Fixed ด้วยกัน
2. ใช้ MeshPart สำหรับ Objects ที่ซับซ้อน
3. ใช้ Terrain แทน Parts สำหรับพื้นดิน
```

### 10.3 ระยะการ Render (RenderFidelity)

```lua
-- กำหนด RenderFidelity สำหรับ MeshParts ที่อยู่ไกล
local meshPart = Instance.new("MeshPart")
meshPart.RenderFidelity = Enum.RenderFidelity.Automatic
-- Automatic - ปรับตามระยะอัตโนมัติ
-- Precise   - คุณภาพสูงสุดเสมอ
-- Performance - ลดรายละเอียดเพื่อประสิทธิภาพ
```

---

## 📚 แบบฝึกหัดตอนที่ 6

### แบบฝึกหัดที่ 1: สำรวจ DataModel
1. เปิด Explorer และ Expand ทุก Service
2. ดู Properties ของ Workspace
3. ดู Properties ของ Lighting

### แบบฝึกหัดที่ 2: Services
1. เขียนโค้ดดู Properties ของแต่ละ Service
2. ลองเปลี่ยน Gravity ของ Workspace
3. ลองเปลี่ยนเวลาใน Lighting

### แบบฝึกหัดที่ 3: Folder Organization
1. สร้าง Folder Structure ตามที่แนะนำ
2. ย้าย Objects เข้าไปใน Folders
3. ตรวจสอบด้วย GetChildren()

### แบบฝึกหัดที่ 4: Instance Management
1. สร้าง Instance.new() ชนิดต่างๆ
2. ทดสอบ Clone()
3. ทดสอบ Destroy()

### แบบฝึกหัดที่ 5: Events
1. ตั้ง ChildAdded Event บน Workspace
2. สร้าง/ลบ Parts แล้วดู Output
3. ทดสอบ AncestryChanged

### แบบฝึกหัดที่ 6: WorkspaceManager
1. คัดลอกโค้ด WorkspaceManager
2. รันและดู Output
3. ลองเพิ่ม Folder ประเภทใหม่

---

## 💡 เคล็ดลับ

1. **จัดระเบียบตั้งแต่เริ่มต้น** - ใช้ Folders จัดกลุ่ม Objects
2. **ตั้งชื่อให้มีความหมาย** - ง่ายต่อการค้นหา
3. **ใช้ FindFirstChild ก่อน Access** - ป้องกัน Nil Error
4. **WaitForChild ใน LocalScript** - เพราะโหลดช้ากว่า Server
5. **นับ Part Count** - ป้องกัน Lag

---

## ⏭️ ตอนถัดไป

ในตอนที่ 7 เราจะเรียนรู้:
- การสร้าง Baseplate และ Environment แรก
- การทำ Floor, Walls, Ceiling
- การตกแต่ง Environment เบื้องต้น

---

*ตอนที่ 6/100 | ระดับ: พื้นฐาน | เวลา: 60-90 นาที*
