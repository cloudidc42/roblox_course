# ตอนที่ 21: Players Service ใน Roblox

## บทนำ

Players Service เป็นหนึ่งใน Services ที่สำคัญที่สุดใน Roblox ใช้สำหรับจัดการผู้เล่นทั้งหมดในเกม ตั้งแต่การรับผู้เล่นเข้า ดูข้อมูล ไปจนถึงการ kick ออกจากเกม

---

## 21.1 ข้อมูลพื้นฐาน

```lua
local Players = game:GetService("Players")

-- Properties
print(Players.MaxPlayers)          -- จำนวนผู้เล่นสูงสุดที่กำหนด
print(Players.RespawnTime)         -- เวลา respawn (default 5 วินาที)

-- จำนวนผู้เล่นปัจจุบัน
local count = #Players:GetPlayers()
print("ผู้เล่นปัจจุบัน: " .. count)

-- รายชื่อผู้เล่นทั้งหมด
local allPlayers = Players:GetPlayers()
for _, player in ipairs(allPlayers) do
    print("- " .. player.Name .. " (UserId: " .. player.UserId .. ")")
end

-- LocalPlayer (ใช้ได้เฉพาะ LocalScript)
local localPlayer = Players.LocalPlayer
if localPlayer then
    print("ชื่อของคุณ: " .. localPlayer.Name)
end
```

---

## 21.2 Player Object

```lua
local player = Players:GetPlayers()[1]  -- ดึงผู้เล่นคนแรก

-- Properties
print(player.Name)           -- ชื่อ Roblox
print(player.DisplayName)    -- Display name
print(player.UserId)         -- Unique ID (ไม่เปลี่ยน)
print(player.AccountAge)     -- อายุบัญชี (วัน)
print(player.MembershipType) -- Free, Premium
print(player.TeamColor)      -- สีทีม
print(player.Team)           -- Team object
print(player.RespawnLocation)  -- CFrame ที่ spawn

-- Character
local character = player.Character
print(player.Character)      -- Character model (nil ถ้ายังไม่โหลด)

-- GUI
local playerGui = player.PlayerGui
local backpack = player.Backpack

-- Leaderstats
local leaderstats = player:FindFirstChild("leaderstats")
```

---

## 21.3 Events สำคัญ

```lua
local Players = game:GetService("Players")

-- เมื่อผู้เล่นเข้าร่วม
Players.PlayerAdded:Connect(function(player)
    print(player.Name .. " เข้าร่วมเกม")
    print("UserId: " .. player.UserId)
    print("AccountAge: " .. player.AccountAge .. " วัน")
    print("Premium: " .. tostring(player.MembershipType == Enum.MembershipType.Premium))
    
    -- Setup player data
    setupLeaderstats(player)
    
    -- รอ character
    player.CharacterAdded:Connect(function(character)
        onCharacterAdded(player, character)
    end)
end)

-- เมื่อผู้เล่นออก
Players.PlayerRemoving:Connect(function(player)
    print(player.Name .. " ออกจากเกม")
    -- บันทึกข้อมูลก่อนออก
    savePlayerData(player)
end)

-- Functions
local function setupLeaderstats(player)
    local leaderstats = Instance.new("Folder")
    leaderstats.Name = "leaderstats"
    leaderstats.Parent = player
    
    local score = Instance.new("IntValue")
    score.Name = "Score"
    score.Value = 0
    score.Parent = leaderstats
    
    local level = Instance.new("IntValue")
    level.Name = "Level"
    level.Value = 1
    level.Parent = leaderstats
    
    local cash = Instance.new("IntValue")
    cash.Name = "Cash"
    cash.Value = 100
    cash.Parent = leaderstats
end

local function onCharacterAdded(player, character)
    print(player.Name .. "'s character loaded")
    
    local humanoid = character:WaitForChild("Humanoid")
    
    -- เมื่อตาย
    humanoid.Died:Connect(function()
        print(player.Name .. " ตาย")
        
        -- เพิ่มจำนวนตาย
        local leaderstats = player:FindFirstChild("leaderstats")
        if leaderstats then
            local deaths = leaderstats:FindFirstChild("Deaths")
            if deaths then
                deaths.Value = deaths.Value + 1
            end
        end
    end)
end
```

---

## 21.4 การค้นหา Player

```lua
local Players = game:GetService("Players")

-- หาด้วย UserId
local function getPlayerByUserId(userId)
    return Players:GetPlayerByUserId(userId)
end

-- หาด้วยชื่อ
local function getPlayerByName(name)
    return Players:FindFirstChild(name)
end

-- หาจาก Character
local function getPlayerFromCharacter(character)
    return Players:GetPlayerFromCharacter(character)
end

-- หาจาก Part ที่สัมผัส
local function getPlayerFromHit(hit)
    local character = hit.Parent
    return Players:GetPlayerFromCharacter(character)
end

-- ตัวอย่างการใช้งาน
workspace.Part.Touched:Connect(function(hit)
    local player = getPlayerFromHit(hit)
    if player then
        print(player.Name .. " สัมผัส part")
    end
end)
```

---

## 21.5 Character Management

```lua
local Players = game:GetService("Players")

-- Force respawn
local function respawnPlayer(player)
    player:LoadCharacter()
end

-- Teleport character
local function teleportPlayer(player, position)
    local character = player.Character
    if character then
        local hrp = character:FindFirstChild("HumanoidRootPart")
        if hrp then
            hrp.CFrame = CFrame.new(position)
        end
    end
end

-- Set spawn location
local function setSpawnLocation(player, cframe)
    player.RespawnLocation = workspace:FindFirstChild("SpawnLocation")
end

-- ตรวจสอบว่า player มีชีวิตหรือไม่
local function isPlayerAlive(player)
    local character = player.Character
    if not character then return false end
    
    local humanoid = character:FindFirstChild("Humanoid")
    if not humanoid then return false end
    
    return humanoid.Health > 0
end

-- ให้ tool
local function giveToolToPlayer(player, toolTemplate)
    local tool = toolTemplate:Clone()
    tool.Parent = player.Backpack
end

-- ลบ tools ทั้งหมด
local function clearPlayerTools(player)
    for _, tool in ipairs(player.Backpack:GetChildren()) do
        tool:Destroy()
    end
    
    local character = player.Character
    if character then
        for _, tool in ipairs(character:GetChildren()) do
            if tool:IsA("Tool") then
                tool:Destroy()
            end
        end
    end
end
```

---

## 21.6 Leaderstats System

```lua
-- Script: LeaderstatsManager.lua

local Players = game:GetService("Players")

local LeaderstatsManager = {}

-- สร้าง leaderstats
local function createLeaderstats(player)
    local ls = Instance.new("Folder")
    ls.Name = "leaderstats"
    ls.Parent = player
    
    local function addStat(name, type, defaultValue)
        local stat = Instance.new(type .. "Value")
        stat.Name = name
        stat.Value = defaultValue
        stat.Parent = ls
        return stat
    end
    
    addStat("Score", "Int", 0)
    addStat("Kills", "Int", 0)
    addStat("Deaths", "Int", 0)
    addStat("Level", "Int", 1)
    addStat("Cash", "Int", 500)
    addStat("KD", "Number", 0)
    
    return ls
end

-- อัปเดต stat
local function updateStat(player, statName, value)
    local ls = player:FindFirstChild("leaderstats")
    if not ls then return false end
    
    local stat = ls:FindFirstChild(statName)
    if not stat then return false end
    
    stat.Value = value
    return true
end

-- เพิ่ม stat
local function addToStat(player, statName, amount)
    local ls = player:FindFirstChild("leaderstats")
    if not ls then return false end
    
    local stat = ls:FindFirstChild(statName)
    if not stat then return false end
    
    stat.Value = stat.Value + amount
    
    -- อัปเดต KD ratio
    if statName == "Kills" or statName == "Deaths" then
        local kills = ls:FindFirstChild("Kills")
        local deaths = ls:FindFirstChild("Deaths")
        local kd = ls:FindFirstChild("KD")
        
        if kills and deaths and kd then
            if deaths.Value > 0 then
                kd.Value = math.floor((kills.Value / deaths.Value) * 100) / 100
            else
                kd.Value = kills.Value
            end
        end
    end
    
    return true
end

-- Setup สำหรับผู้เล่นใหม่
Players.PlayerAdded:Connect(function(player)
    createLeaderstats(player)
    
    player.CharacterAdded:Connect(function(character)
        local humanoid = character:WaitForChild("Humanoid")
        
        humanoid.Died:Connect(function()
            addToStat(player, "Deaths", 1)
            
            -- หา killer
            -- (อาจใช้ Tag system หรือ damage tracking)
        end)
    end)
end)

-- ทดสอบ
local function testLeaderstats()
    local allPlayers = Players:GetPlayers()
    for _, player in ipairs(allPlayers) do
        addToStat(player, "Kills", 1)
        addToStat(player, "Score", 100)
        addToStat(player, "Cash", 50)
        
        print(player.Name .. " leaderstats:")
        local ls = player:FindFirstChild("leaderstats")
        if ls then
            for _, stat in ipairs(ls:GetChildren()) do
                print("  " .. stat.Name .. ": " .. tostring(stat.Value))
            end
        end
    end
end
```

---

## 21.7 Team System

```lua
local Teams = game:GetService("Teams")
local Players = game:GetService("Players")

-- สร้างทีม
local function createTeam(name, color)
    local team = Instance.new("Team")
    team.Name = name
    team.TeamColor = BrickColor.new(color)
    team.AutoAssignable = false
    team.Parent = Teams
    return team
end

local redTeam = createTeam("ทีมแดง", "Bright red")
local blueTeam = createTeam("ทีมน้ำเงิน", "Bright blue")

-- จัดทีมผู้เล่น
local function assignTeam(player, team)
    player.Team = team
    player.TeamColor = team.TeamColor
    print(player.Name .. " อยู่ทีม " .. team.Name)
end

-- Auto balance teams
local function autoBalance()
    local allPlayers = Players:GetPlayers()
    
    -- เรียงสุ่ม
    local shuffled = {}
    for _, p in ipairs(allPlayers) do
        table.insert(shuffled, math.random(#shuffled + 1), p)
    end
    
    for i, player in ipairs(shuffled) do
        if i % 2 == 1 then
            assignTeam(player, redTeam)
        else
            assignTeam(player, blueTeam)
        end
    end
end

-- ดูผู้เล่นในทีม
local function getTeamPlayers(team)
    local teamPlayers = {}
    for _, player in ipairs(Players:GetPlayers()) do
        if player.Team == team then
            table.insert(teamPlayers, player)
        end
    end
    return teamPlayers
end

-- ตรวจสอบว่าผู้เล่นอยู่ทีมเดียวกันหรือไม่
local function areSameTeam(player1, player2)
    return player1.Team ~= nil and player1.Team == player2.Team
end
```

---

## 21.8 แบบฝึกหัด

### แบบฝึกหัดที่ 1: Welcome System

```lua
-- สร้างระบบต้อนรับผู้เล่น:
-- - แสดงข้อความต้อนรับใน chat
-- - ให้ไอเทมเริ่มต้น
-- - บอกจำนวนผู้เล่นปัจจุบัน

Players.PlayerAdded:Connect(function(player)
    -- เติมโค้ด
end)
```

### แบบฝึกหัดที่ 2: Kill System

```lua
-- สร้างระบบนับ kills:
-- - เมื่อผู้เล่นฆ่าศัตรู เพิ่ม kills
-- - แสดงชื่อผู้ฆ่าและผู้ตายใน output
-- - ให้รางวัลเมื่อ kills ถึงจำนวนที่กำหนด

-- เติมโค้ด
```

---

## สรุป

| Method/Property | การใช้งาน |
|----------------|----------|
| `Players:GetPlayers()` | รายชื่อผู้เล่นทั้งหมด |
| `Players:GetPlayerByUserId(id)` | หาด้วย UserId |
| `Players.LocalPlayer` | ผู้เล่นปัจจุบัน (LocalScript) |
| `Players.PlayerAdded` | เมื่อผู้เล่นเข้า |
| `Players.PlayerRemoving` | เมื่อผู้เล่นออก |
| `player.Name` | ชื่อผู้เล่น |
| `player.UserId` | ID ไม่ซ้ำ |
| `player.Character` | Character model |
| `player:LoadCharacter()` | Force respawn |
| `player.Team` | ทีมของผู้เล่น |

### บทถัดไป

ในบทที่ 22 เราจะเรียนเรื่อง **Workspace Service** - การใช้งาน Workspace อย่างละเอียด
