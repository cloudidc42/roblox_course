# ตอนที่ 19: Parent-Child Hierarchy ใน Roblox

## บทนำ

ใน Roblox ทุก Instance มีความสัมพันธ์แบบ Parent-Child (พ่อแม่-ลูก) เหมือนโครงสร้างต้นไม้ การเข้าใจ hierarchy นี้เป็นสิ่งจำเป็นสำหรับการเข้าถึงและจัดการ objects ในเกม

---

## 19.1 ความเข้าใจ Hierarchy

```
game (DataModel)
├── Workspace
│   ├── Baseplate
│   ├── Camera
│   └── MyModel (Model)
│       ├── Part1 (Part)
│       ├── Part2 (Part)
│       └── Script
├── Players
│   └── Player1 (Player)
│       ├── Character (Model)
│       │   ├── HumanoidRootPart
│       │   ├── Head
│       │   ├── Torso
│       │   └── Humanoid
│       ├── PlayerGui
│       └── Backpack
├── ReplicatedStorage
├── ServerScriptService
└── StarterGui
```

---

## 19.2 การนำทางใน Hierarchy

### ไปหา Parent

```lua
local part = game.Workspace.MyModel.Part1

-- Parent โดยตรง
local parent = part.Parent          -- MyModel
local grandParent = part.Parent.Parent  -- Workspace
local root = game                       -- root

print(part.Name)           -- Part1
print(part.Parent.Name)    -- MyModel
print(part.Parent.Parent.Name)  -- Workspace
```

### หา Children

```lua
local model = game.Workspace.MyModel

-- GetChildren(): return array ของ direct children
local children = model:GetChildren()
for _, child in ipairs(children) do
    print(child.Name .. " (" .. child.ClassName .. ")")
end

-- GetDescendants(): return ทุก descendants (recursive)
local all = model:GetDescendants()
for _, desc in ipairs(all) do
    print(desc.Name)
end

-- IsA(): ตรวจสอบ class
for _, child in ipairs(children) do
    if child:IsA("BasePart") then
        child.BrickColor = BrickColor.new("Bright blue")
    end
end
```

### FindFirstChild vs WaitForChild

```lua
local workspace = game.Workspace

-- FindFirstChild: หา child ทันที (return nil ถ้าไม่พบ)
local part = workspace:FindFirstChild("MyPart")
if part then
    print("พบ: " .. part.Name)
else
    print("ไม่พบ")
end

-- FindFirstChild recursive (third arg = true)
local deepChild = workspace:FindFirstChild("DeepPart", true)

-- WaitForChild: รอจนกว่าจะพบ (timeout option)
local waitedPart = workspace:WaitForChild("MyPart", 5)  -- รอสูงสุด 5 วินาที
if waitedPart then
    print("พบหลังรอ: " .. waitedPart.Name)
end

-- FindFirstChildOfClass
local script = workspace:FindFirstChildOfClass("Script")
local humanoid = character:FindFirstChildOfClass("Humanoid")

-- FindFirstChildWhichIsA (ตรวจสอบ inheritance)
local basePart = workspace:FindFirstChildWhichIsA("BasePart")
```

---

## 19.3 การเปลี่ยน Parent

```lua
-- ย้าย instance ไปยัง parent ใหม่
local part = game.Workspace.MyPart

-- ย้ายไป folder ใหม่
local folder = game.Workspace.MyFolder
part.Parent = folder

-- ย้าย tool ไป Backpack
local tool = game.Workspace.Sword
local player = game.Players.Player1
tool.Parent = player.Backpack

-- ย้าย tool ออกจาก Backpack ลง workspace
local equippedTool = player.Backpack.Sword
equippedTool.Parent = game.Workspace
```

---

## 19.4 การค้นหาแบบ Path

```lua
-- การเข้าถึงด้วย dot notation
local workspace = game.Workspace
local myFolder = workspace.MyFolder
local myPart = workspace.MyFolder.MyPart

-- เหมือนกับ:
local myPart2 = workspace:FindFirstChild("MyFolder"):FindFirstChild("MyPart")

-- ใช้ game:GetService() สำหรับ Services
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local ServerScriptService = game:GetService("ServerScriptService")
local StarterGui = game:GetService("StarterGui")

-- ระวัง! ถ้าไม่มี object จะ error
-- local missing = workspace.DoesNotExist  -- ERROR!

-- ปลอดภัยกว่า:
local safe = workspace:FindFirstChild("MightNotExist")
if safe then
    print("พบ: " .. safe.Name)
end
```

---

## 19.5 ตัวอย่างโปรเจกต์: ระบบ Room Manager

```lua
-- Script: RoomManager.lua
-- จัดการห้องต่างๆ ในเกม

local RoomManager = {}
RoomManager.__index = RoomManager

function RoomManager.new(workspaceFolder)
    local self = setmetatable({}, RoomManager)
    self.folder = workspaceFolder
    self.rooms = {}
    self:scanRooms()
    return self
end

function RoomManager:scanRooms()
    for _, child in ipairs(self.folder:GetChildren()) do
        if child:IsA("Model") and child.Name:match("^Room_") then
            self.rooms[child.Name] = child
        end
    end
    print("พบห้องทั้งหมด: " .. self:getRoomCount())
end

function RoomManager:getRoomCount()
    local count = 0
    for _ in pairs(self.rooms) do count = count + 1 end
    return count
end

function RoomManager:getRoom(name)
    return self.rooms[name]
end

function RoomManager:getAllParts(roomName)
    local room = self:getRoom(roomName)
    if not room then return {} end
    
    local parts = {}
    for _, desc in ipairs(room:GetDescendants()) do
        if desc:IsA("BasePart") then
            table.insert(parts, desc)
        end
    end
    return parts
end

function RoomManager:setRoomVisible(roomName, visible)
    local parts = self:getAllParts(roomName)
    for _, part in ipairs(parts) do
        part.Transparency = visible and 0 or 1
        part.CanCollide = visible
    end
end

-- ใช้งาน
-- local manager = RoomManager.new(game.Workspace.Rooms)
-- manager:setRoomVisible("Room_Boss", true)
```

---

## 19.6 Character Hierarchy

```lua
-- โครงสร้างของ Character:
-- Character (Model)
-- ├── HumanoidRootPart (Part)
-- ├── Head (Part)
-- ├── UpperTorso (Part) - R15
-- ├── LowerTorso (Part) - R15
-- ├── Humanoid
-- └── Accessories, Tools, etc.

local Players = game:GetService("Players")

Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function(character)
        -- เข้าถึง parts
        local hrp = character:WaitForChild("HumanoidRootPart")
        local head = character:WaitForChild("Head")
        local humanoid = character:WaitForChild("Humanoid")
        
        -- Position
        print("ตำแหน่งผู้เล่น: " .. tostring(hrp.Position))
        
        -- สัมพัทธ์กับ character
        local localPos = hrp.CFrame:inverse() * CFrame.new(head.Position)
        print("Head อยู่เหนือ HRP: " .. tostring(localPos.Y))
        
        -- ติด accessory
        local function attachHat(hatModel)
            hatModel.Parent = character
        end
        
        -- ตรวจสอบ children ทั้งหมดของ character
        for _, child in ipairs(character:GetChildren()) do
            print(child.Name .. " (" .. child.ClassName .. ")")
        end
    end)
end)
```

---

## สรุป

| Method | การใช้งาน |
|--------|----------|
| `inst.Parent` | ดู parent |
| `inst:GetChildren()` | ดู direct children |
| `inst:GetDescendants()` | ดูทุก descendants |
| `inst:FindFirstChild(name)` | หา child ทันที |
| `inst:FindFirstChild(name, true)` | หาแบบ recursive |
| `inst:WaitForChild(name, timeout)` | รอจนพบ |
| `inst:FindFirstChildOfClass(class)` | หาตาม class |
| `inst:IsA(class)` | ตรวจสอบ class |
| `inst.Parent = newParent` | ย้าย parent |

### บทถัดไป

ในบทที่ 20 เราจะเรียนเรื่อง **Roblox Services Overview** - ภาพรวมของ services ต่างๆ
