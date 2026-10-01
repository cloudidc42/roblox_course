# ตอนที่ 12: โครงสร้างควบคุม (Control Structures) ใน Lua

## บทนำ

โครงสร้างควบคุม (Control Structures) คือกลไกในการควบคุมการไหลของโปรแกรม ช่วยให้เราสามารถตัดสินใจ ทำซ้ำ และจัดการกับเงื่อนไขต่างๆ ได้ ใน Lua มีโครงสร้างควบคุมหลักดังนี้

---

## 12.1 If/Else Statement

### โครงสร้างพื้นฐาน

```lua
-- if ง่ายๆ
if condition then
    -- โค้ดที่ทำงานเมื่อ condition เป็น true
end

-- if/else
if condition then
    -- ทำเมื่อ condition เป็น true
else
    -- ทำเมื่อ condition เป็น false
end

-- if/elseif/else
if condition1 then
    -- ทำเมื่อ condition1 เป็น true
elseif condition2 then
    -- ทำเมื่อ condition2 เป็น true
elseif condition3 then
    -- ทำเมื่อ condition3 เป็น true
else
    -- ทำเมื่อไม่มีเงื่อนไขไหนเป็น true
end
```

### ตัวอย่างการใช้งาน

```lua
-- ตรวจสอบเลเวล
local function getPlayerTitle(level)
    if level >= 100 then
        return "ตำนาน"
    elseif level >= 50 then
        return "ผู้เชี่ยวชาญ"
    elseif level >= 25 then
        return "นักรบ"
    elseif level >= 10 then
        return "ผู้ฝึกหัด"
    else
        return "มือใหม่"
    end
end

print(getPlayerTitle(120))  -- ตำนาน
print(getPlayerTitle(75))   -- ผู้เชี่ยวชาญ
print(getPlayerTitle(30))   -- นักรบ
print(getPlayerTitle(15))   -- ผู้ฝึกหัด
print(getPlayerTitle(5))    -- มือใหม่
```

### Nested If (if ซ้อน if)

```lua
local function checkAccess(player)
    if player then
        if player.Character then
            local humanoid = player.Character:FindFirstChild("Humanoid")
            if humanoid then
                if humanoid.Health > 0 then
                    return true, "เข้าถึงได้"
                else
                    return false, "ตัวละครตายแล้ว"
                end
            else
                return false, "ไม่พบ Humanoid"
            end
        else
            return false, "ไม่มี Character"
        end
    else
        return false, "ไม่มีผู้เล่น"
    end
end

-- แบบที่ดีกว่า (Early Return)
local function checkAccessBetter(player)
    if not player then 
        return false, "ไม่มีผู้เล่น" 
    end
    
    if not player.Character then 
        return false, "ไม่มี Character" 
    end
    
    local humanoid = player.Character:FindFirstChild("Humanoid")
    if not humanoid then 
        return false, "ไม่พบ Humanoid" 
    end
    
    if humanoid.Health <= 0 then 
        return false, "ตัวละครตายแล้ว" 
    end
    
    return true, "เข้าถึงได้"
end
```

### การใช้งานจริงใน Roblox

```lua
-- Script: DoorSystem.lua
-- ระบบประตูที่ต้องตรวจสอบเงื่อนไข

local Players = game:GetService("Players")

local door = script.Parent
local REQUIRED_LEVEL = 10
local REQUIRED_TEAM = "นักรบ"

local function canOpenDoor(player)
    -- ตรวจสอบว่ามีข้อมูล leaderstats
    local leaderstats = player:FindFirstChild("leaderstats")
    if not leaderstats then
        return false, "ไม่พบข้อมูลผู้เล่น"
    end
    
    -- ตรวจสอบเลเวล
    local levelStat = leaderstats:FindFirstChild("Level")
    if not levelStat then
        return false, "ไม่พบข้อมูลเลเวล"
    end
    
    if levelStat.Value < REQUIRED_LEVEL then
        return false, "ต้องการเลเวล " .. REQUIRED_LEVEL .. " (เลเวลปัจจุบัน: " .. levelStat.Value .. ")"
    end
    
    -- ตรวจสอบทีม
    if player.Team then
        if player.Team.Name ~= REQUIRED_TEAM then
            return false, "ต้องเป็นทีม " .. REQUIRED_TEAM
        end
    end
    
    return true, "เปิดประตูได้!"
end

door.Touched:Connect(function(hit)
    local player = Players:GetPlayerFromCharacter(hit.Parent)
    if not player then return end
    
    local canOpen, message = canOpenDoor(player)
    
    if canOpen then
        -- เปิดประตู
        door.CanCollide = false
        door.Transparency = 0.5
        print(player.Name .. ": " .. message)
        
        -- ปิดประตูหลัง 3 วินาที
        task.wait(3)
        door.CanCollide = true
        door.Transparency = 0
    else
        -- แสดงข้อความ error
        print(player.Name .. ": " .. message)
    end
end)
```

---

## 12.2 While Loop

ทำซ้ำตราบเท่าที่เงื่อนไขเป็น true

```lua
-- โครงสร้างพื้นฐาน
while condition do
    -- โค้ดที่ทำซ้ำ
end

-- ตัวอย่าง
local count = 1
while count <= 5 do
    print("นับ: " .. count)
    count = count + 1
end
-- แสดง: นับ: 1, นับ: 2, นับ: 3, นับ: 4, นับ: 5

-- ระวัง! infinite loop
-- while true do
--     print("วนไม่รู้จบ!")  -- ต้องมี break หรือ condition เปลี่ยน
-- end
```

### While Loop แบบ Controlled

```lua
-- ใช้ break เพื่อออกจาก loop
local searchList = {"แอปเปิ้ล", "กล้วย", "ส้ม", "มะม่วง", "องุ่น"}
local searchFor = "ส้ม"
local found = false
local index = 1

while index <= #searchList do
    if searchList[index] == searchFor then
        found = true
        break  -- หยุด loop เมื่อเจอ
    end
    index = index + 1
end

if found then
    print("พบ " .. searchFor .. " ที่ตำแหน่ง " .. index)
else
    print("ไม่พบ " .. searchFor)
end
```

### การใช้งานจริงใน Roblox

```lua
-- Game Timer System
local function runGameTimer(duration)
    local timeLeft = duration
    
    while timeLeft > 0 do
        -- อัปเดต timer display
        local minutes = math.floor(timeLeft / 60)
        local seconds = timeLeft % 60
        
        -- แสดงเวลา (ในเกมจริงจะอัปเดต GUI)
        print(string.format("เวลาที่เหลือ: %02d:%02d", minutes, seconds))
        
        task.wait(1)  -- รอ 1 วินาที
        timeLeft = timeLeft - 1
    end
    
    print("เวลาหมด!")
end

-- Wave Spawner
local function spawnWaves(totalWaves, enemiesPerWave)
    local currentWave = 1
    
    while currentWave <= totalWaves do
        print("เริ่ม Wave " .. currentWave)
        
        -- spawn enemies
        for i = 1, enemiesPerWave do
            -- spawnEnemy()  -- ฟังก์ชัน spawn
            print("Spawn ศัตรู " .. i)
        end
        
        -- รอให้กำจัดศัตรูทั้งหมด
        -- ในเกมจริงจะตรวจสอบ enemy count
        task.wait(5)
        
        currentWave = currentWave + 1
        print("Wave " .. (currentWave - 1) .. " สำเร็จ!")
    end
    
    print("ชนะทุก Wave!")
end
```

---

## 12.3 Repeat-Until Loop

ทำซ้ำอย่างน้อยหนึ่งครั้ง แล้วตรวจสอบเงื่อนไขหลังจากนั้น

```lua
-- โครงสร้างพื้นฐาน
repeat
    -- โค้ดที่ทำซ้ำ
until condition  -- หยุดเมื่อ condition เป็น true

-- ตัวอย่าง
local count = 1
repeat
    print("นับ: " .. count)
    count = count + 1
until count > 5

-- ข้อแตกต่างจาก while: ทำก่อน ตรวจสอบทีหลัง
local x = 10
repeat
    print("x = " .. x)
    x = x + 1
until x > 5  -- condition เป็น true ทันที แต่ยังพิมพ์ครั้งแรก
-- พิมพ์: x = 10 (พิมพ์แม้ 10 > 5)
```

### การใช้งานจริงใน Roblox

```lua
-- รอจนกว่าจะโหลดเสร็จ
local function waitForLoad()
    repeat
        task.wait()  -- รอ 1 frame
    until game:IsLoaded()
    
    print("โหลดเสร็จแล้ว!")
end

-- รอให้ผู้เล่น spawn
local function waitForCharacter(player)
    repeat
        task.wait(0.1)
    until player.Character ~= nil
    
    return player.Character
end

-- Retry mechanism
local function fetchDataWithRetry(maxAttempts)
    local attempts = 0
    local success = false
    local data = nil
    
    repeat
        attempts = attempts + 1
        print("ความพยายามที่ " .. attempts)
        
        -- จำลอง request (สุ่ม success/fail)
        local succeeded = math.random(1, 3) == 1  -- 33% โอกาสสำเร็จ
        
        if succeeded then
            success = true
            data = {status = "OK", value = math.random(1, 100)}
        else
            print("ล้มเหลว ลองใหม่...")
            task.wait(1)
        end
        
    until success or attempts >= maxAttempts
    
    if success then
        print("สำเร็จหลังจาก " .. attempts .. " ครั้ง")
        return data
    else
        print("หมดความพยายาม")
        return nil
    end
end
```

---

## 12.4 Numeric For Loop

ทำซ้ำตามจำนวนที่กำหนด

```lua
-- โครงสร้าง: for i = start, end, step do
for i = 1, 10 do
    print(i)
end

-- กำหนด step
for i = 1, 10, 2 do    -- นับทีละ 2
    print(i)           -- 1, 3, 5, 7, 9
end

-- นับถอยหลัง
for i = 10, 1, -1 do   -- step ลบ
    print(i)           -- 10, 9, 8, 7, ...1
end

-- step ทศนิยม
for i = 0, 1, 0.25 do
    print(i)           -- 0, 0.25, 0.5, 0.75, 1.0
end
```

### การใช้งานจริงใน Roblox

```lua
-- สร้างแพลตฟอร์มแบบ pattern
local function createPlatformRow(count, spacing, startPos)
    for i = 1, count do
        local part = Instance.new("Part")
        part.Size = Vector3.new(4, 1, 4)
        part.Position = Vector3.new(
            startPos.X + (i - 1) * spacing,
            startPos.Y,
            startPos.Z
        )
        part.Anchored = true
        part.BrickColor = BrickColor.new(i % 2 == 0 and "Bright blue" or "Bright red")
        part.Parent = game.Workspace
    end
end

createPlatformRow(10, 5, Vector3.new(0, 5, 0))

-- Countdown timer
local function countdown(from)
    for i = from, 1, -1 do
        print("เริ่มใน " .. i .. "...")
        task.wait(1)
    end
    print("เริ่ม!")
end

-- สร้าง fireworks
local function createFireworks(count)
    for i = 1, count do
        local position = Vector3.new(
            math.random(-20, 20),
            math.random(20, 40),
            math.random(-20, 20)
        )
        
        -- สร้าง explosion particle
        local explosion = Instance.new("Explosion")
        explosion.Position = position
        explosion.BlastPressure = 0  -- ไม่ทำให้กระเด็น
        explosion.BlastRadius = 5
        explosion.Parent = game.Workspace
        
        task.wait(0.3)  -- หน่วงเวลาระหว่าง firework
    end
end

-- Spawn enemies ตามจำนวน wave
local function spawnEnemiesForWave(wave)
    local enemyCount = wave * 5  -- wave 1 = 5 ศัตรู, wave 2 = 10, etc.
    
    print("Wave " .. wave .. ": กำลัง spawn " .. enemyCount .. " ศัตรู")
    
    for i = 1, enemyCount do
        local spawnAngle = (i / enemyCount) * (2 * math.pi)
        local spawnRadius = 50
        
        local x = math.cos(spawnAngle) * spawnRadius
        local z = math.sin(spawnAngle) * spawnRadius
        
        -- spawnEnemy(Vector3.new(x, 0, z))
        print("Spawn ศัตรู " .. i .. " ที่ (" .. math.floor(x) .. ", 0, " .. math.floor(z) .. ")")
        
        task.wait(0.1)
    end
end
```

---

## 12.5 Generic For Loop (pairs และ ipairs)

ใช้สำหรับวนซ้ำผ่าน tables

```lua
-- ipairs: วนซ้ำ array (sequential table)
local fruits = {"แอปเปิ้ล", "กล้วย", "ส้ม", "มะม่วง"}

for i, fruit in ipairs(fruits) do
    print(i .. ": " .. fruit)
end
-- 1: แอปเปิ้ล
-- 2: กล้วย
-- 3: ส้ม
-- 4: มะม่วง

-- pairs: วนซ้ำ table ทุกประเภท
local playerStats = {
    name = "สมชาย",
    level = 15,
    health = 100,
    mana = 50
}

for key, value in pairs(playerStats) do
    print(key .. " = " .. tostring(value))
end
-- (ลำดับไม่แน่นอน)
```

### ความแตกต่างระหว่าง ipairs และ pairs

```lua
local mixedTable = {10, 20, 30, key1 = "a", key2 = "b", 40}

-- ipairs: วนเฉพาะ index 1, 2, 3, ... จนเจอ nil
print("ipairs:")
for i, v in ipairs(mixedTable) do
    print(i, v)
end
-- 1  10
-- 2  20
-- 3  30
-- 4  40

-- pairs: วนทุก key-value
print("pairs:")
for k, v in pairs(mixedTable) do
    print(k, v)
end
-- 1  10
-- 2  20
-- 3  30
-- 4  40
-- key1  a
-- key2  b

-- ipairs หยุดที่ nil
local withNil = {1, 2, nil, 4, 5}
print("ipairs กับ nil:")
for i, v in ipairs(withNil) do
    print(i, v)
end
-- 1  1
-- 2  2
-- (หยุด เพราะ index 3 เป็น nil)
```

### การใช้งานจริงใน Roblox

```lua
local Players = game:GetService("Players")

-- วนผ่านผู้เล่นทั้งหมด
local function getTopPlayers(count)
    local allPlayers = Players:GetPlayers()
    local playerScores = {}
    
    for _, player in ipairs(allPlayers) do
        local leaderstats = player:FindFirstChild("leaderstats")
        if leaderstats then
            local score = leaderstats:FindFirstChild("Score")
            if score then
                table.insert(playerScores, {
                    name = player.Name,
                    score = score.Value
                })
            end
        end
    end
    
    -- เรียงลำดับ
    table.sort(playerScores, function(a, b)
        return a.score > b.score
    end)
    
    -- Return top N
    local topPlayers = {}
    for i = 1, math.min(count, #playerScores) do
        table.insert(topPlayers, playerScores[i])
    end
    
    return topPlayers
end

-- แสดง leaderboard
local function displayLeaderboard()
    local topPlayers = getTopPlayers(5)
    print("=== Leaderboard ===")
    for i, playerData in ipairs(topPlayers) do
        print(i .. ". " .. playerData.name .. " - " .. playerData.score)
    end
end

-- วน loop บน children ของ folder
local function processFolder(folder)
    for _, child in ipairs(folder:GetChildren()) do
        if child:IsA("Part") then
            child.BrickColor = BrickColor.new("Bright green")
        elseif child:IsA("Model") then
            print("พบ Model: " .. child.Name)
            -- processFolder(child)  -- recursive
        end
    end
end

-- วน loop กับ configuration
local function applyConfig(config, target)
    for propertyName, value in pairs(config) do
        local success, err = pcall(function()
            target[propertyName] = value
        end)
        
        if not success then
            print("ไม่สามารถตั้งค่า " .. propertyName .. ": " .. err)
        end
    end
end

local partConfig = {
    BrickColor = BrickColor.new("Bright red"),
    Material = Enum.Material.SmoothPlastic,
    Anchored = true,
    CanCollide = true
}

local myPart = Instance.new("Part")
applyConfig(partConfig, myPart)
myPart.Parent = game.Workspace
```

---

## 12.6 Break และ Continue

### Break

หยุด loop ทันที

```lua
-- หา prime numbers
local function findFirstNPrimes(n)
    local primes = {}
    local candidate = 2
    
    while #primes < n do
        local isPrime = true
        
        for i = 2, math.sqrt(candidate) do
            if candidate % i == 0 then
                isPrime = false
                break  -- ออกจาก inner loop
            end
        end
        
        if isPrime then
            table.insert(primes, candidate)
        end
        
        candidate = candidate + 1
    end
    
    return primes
end

local primes = findFirstNPrimes(10)
for _, p in ipairs(primes) do
    io.write(p .. " ")
end
-- 2 3 5 7 11 13 17 19 23 29

-- Break ใน while loop
local players = {"สมชาย", "สมหญิง", "สมศักดิ์", nil, "สมใจ"}
local foundIndex = nil

for i, name in ipairs(players) do
    if name == "สมศักดิ์" then
        foundIndex = i
        break
    end
end

print("พบที่ index: " .. tostring(foundIndex))  -- พบที่ index: 3
```

### Continue (Lua ไม่มี continue แต่จำลองได้)

```lua
-- Lua ไม่มี continue แต่ใช้ goto หรือ inverted condition

-- วิธีที่ 1: Inverted condition
local numbers = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
print("เลขคู่:")
for _, n in ipairs(numbers) do
    if n % 2 == 0 then  -- เฉพาะเลขคู่
        print(n)
    end
    -- ข้ามเลขคี่โดยอัตโนมัติ
end

-- วิธีที่ 2: goto (Lua 5.2+, Roblox รองรับ)
print("\nเลขคู่ด้วย goto:")
for i = 1, 10 do
    if i % 2 ~= 0 then
        goto continue  -- ข้ามเลขคี่
    end
    print(i)
    ::continue::  -- label สำหรับ goto
end
```

---

## 12.7 Nested Loops

```lua
-- ตาราง multiplication
print("ตารางสูตรคูณ:")
for i = 1, 5 do
    local row = ""
    for j = 1, 5 do
        row = row .. string.format("%4d", i * j)
    end
    print(row)
end

-- สร้าง 2D Grid
local GRID_WIDTH = 5
local GRID_HEIGHT = 5

local grid = {}
for row = 1, GRID_HEIGHT do
    grid[row] = {}
    for col = 1, GRID_WIDTH do
        -- 1 = ที่ว่าง, 0 = กำแพง
        grid[row][col] = (row == 1 or row == GRID_HEIGHT or 
                         col == 1 or col == GRID_WIDTH) and 0 or 1
    end
end

-- แสดง grid
for row = 1, GRID_HEIGHT do
    local line = ""
    for col = 1, GRID_WIDTH do
        line = line .. (grid[row][col] == 0 and "#" or ".")
    end
    print(line)
end
-- #####
-- #...#
-- #...#
-- #...#
-- #####
```

### การใช้งานจริงใน Roblox

```lua
-- สร้าง Maze
local function createMaze(width, height, cellSize)
    local mazeFolder = Instance.new("Folder")
    mazeFolder.Name = "Maze"
    mazeFolder.Parent = game.Workspace
    
    for row = 1, height do
        for col = 1, width do
            local isWall = (row == 1 or row == height or 
                           col == 1 or col == width)
            
            -- สร้าง cell
            local cell = Instance.new("Part")
            cell.Name = "Cell_" .. row .. "_" .. col
            cell.Size = Vector3.new(cellSize, cellSize, cellSize)
            cell.Position = Vector3.new(
                (col - 1) * cellSize,
                cellSize / 2,
                (row - 1) * cellSize
            )
            cell.Anchored = true
            
            if isWall then
                cell.BrickColor = BrickColor.new("Dark grey")
                cell.Material = Enum.Material.Concrete
            else
                cell.BrickColor = BrickColor.new("Bright yellow")
                cell.Material = Enum.Material.SmoothPlastic
                cell.Transparency = 0.5
            end
            
            cell.Parent = mazeFolder
        end
    end
    
    print("สร้าง Maze ขนาด " .. width .. "x" .. height .. " เสร็จแล้ว")
end

-- สร้าง explosion pattern
local function createExplosionPattern(center, rings, spacing)
    for ring = 1, rings do
        local radius = ring * spacing
        local numParts = ring * 8  -- เพิ่มจำนวนตาม ring
        
        for i = 1, numParts do
            local angle = (i / numParts) * (2 * math.pi)
            local x = center.X + math.cos(angle) * radius
            local z = center.Z + math.sin(angle) * radius
            
            local part = Instance.new("Part")
            part.Size = Vector3.new(1, 1, 1)
            part.Position = Vector3.new(x, center.Y, z)
            part.Anchored = true
            part.BrickColor = BrickColor.new(ring % 2 == 0 and "Bright orange" or "Bright red")
            part.Parent = game.Workspace
        end
        
        task.wait(0.1)  -- delay ระหว่าง ring
    end
end
```

---

## 12.8 โครงสร้างควบคุม Pattern ที่ใช้บ่อยใน Roblox

### Pattern 1: Event Handler with Guard

```lua
local Players = game:GetService("Players")

Players.PlayerAdded:Connect(function(player)
    -- Guard clause: ออกถ้าเงื่อนไขไม่ผ่าน
    if not player then return end
    
    -- รอให้ character โหลด
    local character = player.Character or player.CharacterAdded:Wait()
    if not character then return end
    
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end
    
    -- main logic
    print(player.Name .. " เข้าร่วมเกม")
    
    -- Setup character events
    humanoid.Died:Connect(function()
        print(player.Name .. " ตายแล้ว")
        
        task.wait(5)  -- รอก่อน respawn
        
        if player and player.Parent then  -- ตรวจสอบว่ายังอยู่ในเกม
            player:LoadCharacter()
        end
    end)
end)
```

### Pattern 2: State Machine

```lua
-- ระบบ State Machine สำหรับตัวละคร
local CharacterState = {
    IDLE = "idle",
    WALKING = "walking",
    RUNNING = "running",
    JUMPING = "jumping",
    ATTACKING = "attacking",
    DEAD = "dead"
}

local function updateCharacterAnimation(character, currentState, newState)
    -- ตรวจสอบ transition ที่ valid
    local validTransitions = {
        [CharacterState.IDLE] = {
            CharacterState.WALKING, 
            CharacterState.RUNNING,
            CharacterState.JUMPING,
            CharacterState.ATTACKING
        },
        [CharacterState.WALKING] = {
            CharacterState.IDLE,
            CharacterState.RUNNING,
            CharacterState.JUMPING
        },
        [CharacterState.RUNNING] = {
            CharacterState.IDLE,
            CharacterState.WALKING,
            CharacterState.JUMPING
        },
        [CharacterState.JUMPING] = {
            CharacterState.IDLE,
            CharacterState.WALKING
        },
        [CharacterState.ATTACKING] = {
            CharacterState.IDLE
        },
        [CharacterState.DEAD] = {}  -- ไม่สามารถเปลี่ยนจาก dead
    }
    
    -- ตรวจสอบว่า transition valid
    local allowed = validTransitions[currentState]
    local canTransition = false
    
    if allowed then
        for _, allowedState in ipairs(allowed) do
            if allowedState == newState then
                canTransition = true
                break
            end
        end
    end
    
    if not canTransition then
        print("ไม่สามารถเปลี่ยนจาก " .. currentState .. " ไป " .. newState)
        return currentState
    end
    
    -- เปลี่ยน animation
    if newState == CharacterState.IDLE then
        print("เล่น animation: Idle")
    elseif newState == CharacterState.WALKING then
        print("เล่น animation: Walk")
    elseif newState == CharacterState.RUNNING then
        print("เล่น animation: Run")
    elseif newState == CharacterState.JUMPING then
        print("เล่น animation: Jump")
    elseif newState == CharacterState.ATTACKING then
        print("เล่น animation: Attack")
    elseif newState == CharacterState.DEAD then
        print("เล่น animation: Death")
    end
    
    return newState
end

-- ทดสอบ
local state = CharacterState.IDLE
state = updateCharacterAnimation(nil, state, CharacterState.WALKING)
state = updateCharacterAnimation(nil, state, CharacterState.RUNNING)
state = updateCharacterAnimation(nil, state, CharacterState.ATTACKING)  -- ไม่ valid!
state = updateCharacterAnimation(nil, state, CharacterState.JUMPING)
```

### Pattern 3: Game Loop

```lua
-- Game Loop หลัก
local GameState = {
    LOBBY = "lobby",
    COUNTDOWN = "countdown",
    PLAYING = "playing",
    GAMEOVER = "gameover"
}

local function runGameLoop()
    local state = GameState.LOBBY
    local players = {}
    
    while true do
        if state == GameState.LOBBY then
            -- รอผู้เล่น
            print("รอผู้เล่น... (" .. #players .. "/4)")
            
            if #players >= 2 then  -- ต้องการอย่างน้อย 2 คน
                state = GameState.COUNTDOWN
            end
            
            task.wait(1)
            
        elseif state == GameState.COUNTDOWN then
            -- นับถอยหลัง
            for i = 5, 1, -1 do
                print("เริ่มใน " .. i)
                task.wait(1)
            end
            
            state = GameState.PLAYING
            
        elseif state == GameState.PLAYING then
            -- เล่นเกม 5 นาที
            local gameTime = 300
            local startTime = os.time()
            
            repeat
                local elapsed = os.time() - startTime
                local remaining = gameTime - elapsed
                
                if remaining % 60 == 0 then
                    print("เวลาที่เหลือ: " .. (remaining / 60) .. " นาที")
                end
                
                task.wait(1)
            until remaining <= 0
            
            state = GameState.GAMEOVER
            
        elseif state == GameState.GAMEOVER then
            -- แสดงผล
            print("เกมจบ!")
            -- displayResults()
            
            task.wait(10)  -- แสดงผล 10 วินาที
            
            -- Reset
            players = {}
            state = GameState.LOBBY
        end
    end
end
```

---

## 12.9 pcall และ Error Handling

```lua
-- pcall: protected call - จับ error ที่อาจเกิดขึ้น
local success, result = pcall(function()
    -- โค้ดที่อาจเกิด error
    local x = tonumber("hello")
    return 10 / x  -- division by nil = error
end)

if success then
    print("สำเร็จ: " .. result)
else
    print("เกิด error: " .. result)
end

-- xpcall: เหมือน pcall แต่รับ error handler
local function errorHandler(err)
    print("Error Handler: " .. err)
    return "handled: " .. err
end

local success2, result2 = xpcall(function()
    error("ทดสอบ error")
end, errorHandler)

-- การใช้งานจริงใน Roblox
local function safeGetData(player, dataKey)
    local success, data = pcall(function()
        local leaderstats = player.leaderstats
        return leaderstats[dataKey].Value
    end)
    
    if success then
        return data
    else
        print("ไม่สามารถอ่านข้อมูล " .. dataKey .. ": " .. data)
        return nil
    end
end
```

---

## 12.10 แบบฝึกหัด

### แบบฝึกหัดที่ 1: FizzBuzz

```lua
-- Classic FizzBuzz:
-- - ถ้าหารด้วย 3 ลงตัว: พิมพ์ "Fizz"
-- - ถ้าหารด้วย 5 ลงตัว: พิมพ์ "Buzz"
-- - ถ้าหารด้วยทั้ง 3 และ 5: พิมพ์ "FizzBuzz"
-- - อื่นๆ: พิมพ์ตัวเลข

for i = 1, 30 do
    -- เติมโค้ดที่นี่
end
```

### แบบฝึกหัดที่ 2: เกม Guess Number

```lua
-- สร้างเกมทายตัวเลข 1-100
-- ผู้เล่นมี 7 ครั้ง
-- บอกว่า "มากกว่า" หรือ "น้อยกว่า"

math.randomseed(os.time())
local secretNumber = math.random(1, 100)
local maxAttempts = 7

-- เติมโค้ดที่นี่
-- (ในสภาพแวดล้อมจริงจะใช้ GUI แต่ทดสอบด้วย hardcoded values ได้)

local guesses = {50, 75, 62, 56, 59, 61, 60}  -- guesses สำหรับทดสอบ
for i, guess in ipairs(guesses) do
    print("ครั้งที่ " .. i .. ": ทาย " .. guess)
    -- ตรวจสอบและให้ hint
end
```

### แบบฝึกหัดที่ 3: ระบบเมนู

```lua
-- สร้างระบบเมนูด้วย if/elseif
local function handleMenuChoice(choice)
    -- 1. เริ่มเกม
    -- 2. ดูคะแนนสูงสุด
    -- 3. การตั้งค่า
    -- 4. ออกจากเกม
end

for _, choice in ipairs({1, 3, 2, 4}) do
    handleMenuChoice(choice)
end
```

---

## สรุป

| โครงสร้าง | การใช้งาน |
|----------|----------|
| `if/elseif/else` | ตรวจสอบเงื่อนไข |
| `while` | ทำซ้ำตราบเท่าที่เงื่อนไขเป็น true |
| `repeat/until` | ทำซ้ำอย่างน้อยครั้งหนึ่ง |
| `for i = s, e, step` | ทำซ้ำตามจำนวน |
| `for k, v in pairs/ipairs` | วนซ้ำ table |
| `break` | ออกจาก loop |
| `goto` | ข้าม (แทน continue) |

### สิ่งสำคัญที่ต้องจำ

1. **Early Return Pattern** ทำให้โค้ดสะอาดกว่า nested if
2. **ipairs** สำหรับ array, **pairs** สำหรับทุก table
3. **break** ออกจาก loop ในทันที
4. **pcall** จับ error ที่อาจเกิด
5. **task.wait()** ใน Roblox แทน sleep()

### บทถัดไป

ในบทที่ 13 เราจะเรียนเรื่อง **Functions** - การสร้างและใช้งานฟังก์ชันอย่างละเอียด
