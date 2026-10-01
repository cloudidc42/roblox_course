# ตอนที่ 17: Events และ Signals ใน Roblox

## บทนำ

Events (หรือ Signals) เป็นระบบที่สำคัญมากใน Roblox ช่วยให้โปรแกรมตอบสนองต่อสิ่งที่เกิดขึ้นในเกม เช่น เมื่อผู้เล่นเข้าร่วม เมื่อมีการกดปุ่ม หรือเมื่อตัวละครถูกสัมผัส

---

## 17.1 ความเข้าใจ Events

Events ทำงานตามหลักการ Publisher-Subscriber:
- **Publisher**: ส่วนที่ "ยิง" event (เช่น Roblox engine)
- **Subscriber**: โค้ดที่ "รอฟัง" และ "ตอบสนอง" ต่อ event

```lua
-- โครงสร้างพื้นฐาน
event:Connect(function(...)
    -- ทำอะไรบางอย่างเมื่อ event เกิดขึ้น
end)

-- ตัวอย่างจริง
local Players = game:GetService("Players")

Players.PlayerAdded:Connect(function(player)
    print(player.Name .. " เข้ามาในเกม!")
end)
```

---

## 17.2 RBXScriptSignal (Built-in Events)

Roblox มี events ที่ built-in อยู่แล้วมากมาย:

### Player Events

```lua
local Players = game:GetService("Players")

-- เมื่อผู้เล่นเข้าร่วม
Players.PlayerAdded:Connect(function(player)
    print("ยินดีต้อนรับ: " .. player.Name)
    
    -- รอให้ character โหลด
    player.CharacterAdded:Connect(function(character)
        print(player.Name .. " character โหลดแล้ว")
        
        -- ดู humanoid events
        local humanoid = character:WaitForChild("Humanoid")
        
        humanoid.Died:Connect(function()
            print(player.Name .. " ตายแล้ว!")
        end)
        
        humanoid.HealthChanged:Connect(function(health)
            print(player.Name .. " HP เปลี่ยนเป็น: " .. health)
        end)
    end)
    
    -- เมื่อ player chat
    player.Chatted:Connect(function(message)
        print(player.Name .. ": " .. message)
    end)
end)

-- เมื่อผู้เล่นออก
Players.PlayerRemoving:Connect(function(player)
    print("ลาก่อน: " .. player.Name)
    -- บันทึกข้อมูลก่อน player ออก
end)
```

### Part/Instance Events

```lua
local part = script.Parent

-- Touched: เมื่อมีอะไรสัมผัส
part.Touched:Connect(function(hit)
    print("สัมผัสโดย: " .. hit.Name)
    
    -- ตรวจสอบว่าเป็น character
    local character = hit.Parent
    local player = game.Players:GetPlayerFromCharacter(character)
    if player then
        print("ผู้เล่น " .. player.Name .. " เหยียบ!")
    end
end)

-- TouchEnded: เมื่อหยุดสัมผัส
part.TouchEnded:Connect(function(hit)
    print("หยุดสัมผัส: " .. hit.Name)
end)
```

### GUI Events

```lua
-- Button click
local button = script.Parent  -- TextButton หรือ ImageButton

button.MouseButton1Click:Connect(function()
    print("กดปุ่มแล้ว!")
end)

button.MouseButton1Down:Connect(function()
    print("กำลังกดปุ่ม")
end)

button.MouseButton1Up:Connect(function()
    print("ปล่อยปุ่ม")
end)

-- Mouse hover
button.MouseEnter:Connect(function()
    button.BackgroundColor3 = Color3.fromRGB(100, 200, 100)  -- สีเขียวเมื่อ hover
end)

button.MouseLeave:Connect(function()
    button.BackgroundColor3 = Color3.fromRGB(100, 100, 200)  -- กลับสีปกติ
end)

-- TextBox events
local textBox = script.Parent

textBox.FocusLost:Connect(function(enterPressed)
    if enterPressed then
        print("กด Enter: " .. textBox.Text)
    else
        print("คลิกออกไป: " .. textBox.Text)
    end
end)

textBox:GetPropertyChangedSignal("Text"):Connect(function()
    print("Text เปลี่ยนเป็น: " .. textBox.Text)
end)
```

---

## 17.3 การจัดการ Connection

```lua
-- เก็บ connection เพื่อ disconnect ภายหลัง
local connection = part.Touched:Connect(function(hit)
    print("Touched: " .. hit.Name)
end)

-- Disconnect เมื่อต้องการหยุดฟัง
connection:Disconnect()

-- Pattern: disconnect หลังจาก trigger ครั้งแรก
local firstTouch
firstTouch = part.Touched:Connect(function(hit)
    print("First touch: " .. hit.Name)
    firstTouch:Disconnect()  -- ฟังแค่ครั้งเดียว
end)

-- หรือใช้ Once (Roblox รองรับ)
part.Touched:Once(function(hit)
    print("One-time touch: " .. hit.Name)
    -- disconnect อัตโนมัติ
end)
```

### การจัดการ Multiple Connections

```lua
-- เก็บ connections ทั้งหมดเพื่อ cleanup
local ConnectionManager = {}

local connections = {}

local function addConnection(conn, tag)
    tag = tag or "default"
    if not connections[tag] then
        connections[tag] = {}
    end
    table.insert(connections[tag], conn)
    return conn
end

local function disconnectAll(tag)
    if tag then
        if connections[tag] then
            for _, conn in ipairs(connections[tag]) do
                conn:Disconnect()
            end
            connections[tag] = nil
        end
    else
        for _, group in pairs(connections) do
            for _, conn in ipairs(group) do
                conn:Disconnect()
            end
        end
        connections = {}
    end
end

-- ใช้งาน
addConnection(
    Players.PlayerAdded:Connect(function(player)
        print("Player joined: " .. player.Name)
    end),
    "playerEvents"
)

addConnection(
    Players.PlayerRemoving:Connect(function(player)
        print("Player left: " .. player.Name)
    end),
    "playerEvents"
)

-- Cleanup เมื่อต้องการ
-- disconnectAll("playerEvents")
```

---

## 17.4 BindableEvent (Custom Events)

ใช้สำหรับสื่อสารระหว่าง Scripts ฝั่ง Server ด้วยกัน:

```lua
-- ใน ServerScriptService
-- Script ที่ 1: สร้างและ Fire event
local bindableEvent = Instance.new("BindableEvent")
bindableEvent.Name = "PlayerKilled"
bindableEvent.Parent = game.ReplicatedStorage

-- ยิง event พร้อมข้อมูล
bindableEvent:Fire("สมชาย", "ศัตรู", 100)

-- Script ที่ 2: รับ event
local event = game.ReplicatedStorage:WaitForChild("PlayerKilled")

event.Event:Connect(function(killer, victim, damage)
    print(killer .. " สังหาร " .. victim .. " ด้วยความเสียหาย " .. damage)
end)
```

### BindableFunction

```lua
-- สำหรับ request-response pattern
local bindableFunc = Instance.new("BindableFunction")
bindableFunc.Name = "GetPlayerData"
bindableFunc.Parent = game.ReplicatedStorage

-- กำหนด handler
bindableFunc.OnInvoke = function(playerId)
    -- ดึงข้อมูลจากที่เก็บ
    return {
        name = "สมชาย",
        level = 15,
        score = 1000
    }
end

-- เรียกใช้
local data = bindableFunc:Invoke(12345)
print("ชื่อ: " .. data.name)
```

---

## 17.5 RemoteEvent (Client-Server Communication)

```lua
-- Server Script: สร้าง RemoteEvent
local remoteEvent = Instance.new("RemoteEvent")
remoteEvent.Name = "DamageEvent"
remoteEvent.Parent = game.ReplicatedStorage

-- Server: รับข้อมูลจาก Client
remoteEvent.OnServerEvent:Connect(function(player, targetName, damage)
    print(player.Name .. " โจมตี " .. targetName .. " ด้วย " .. damage)
    -- validation ก่อนเสมอ!
end)

-- Server: ส่งข้อมูลไปยัง Client เฉพาะคน
remoteEvent:FireClient(player, "คุณได้รับ 50 คะแนน!")

-- Server: ส่งไปยัง Client ทุกคน
remoteEvent:FireAllClients("Round เริ่มแล้ว!")
```

```lua
-- LocalScript (Client): ส่งข้อมูลไป Server
local remoteEvent = game.ReplicatedStorage:WaitForChild("DamageEvent")

-- ส่งข้อมูลไป Server
remoteEvent:FireServer("EnemyName", 50)

-- รับข้อมูลจาก Server
remoteEvent.OnClientEvent:Connect(function(message)
    print("ได้รับจาก Server: " .. message)
end)
```

### RemoteFunction

```lua
-- Server: กำหนด handler
local remoteFunc = Instance.new("RemoteFunction")
remoteFunc.Name = "GetShopItems"
remoteFunc.Parent = game.ReplicatedStorage

remoteFunc.OnServerInvoke = function(player)
    -- ตรวจสอบว่า player มีสิทธิ์
    return {
        {name = "ดาบ", price = 100},
        {name = "โล่", price = 80},
        {name = "ยาแดง", price = 30}
    }
end

-- Client: เรียกใช้
local remoteFunc = game.ReplicatedStorage:WaitForChild("GetShopItems")
local items = remoteFunc:InvokeServer()

for _, item in ipairs(items) do
    print(item.name .. " - " .. item.price .. " บาท")
end
```

---

## 17.6 GetPropertyChangedSignal

รับการแจ้งเตือนเมื่อ property เปลี่ยน:

```lua
local humanoid = character:FindFirstChild("Humanoid")

-- ตรวจสอบเมื่อ Health เปลี่ยน
humanoid:GetPropertyChangedSignal("Health"):Connect(function()
    local health = humanoid.Health
    local maxHealth = humanoid.MaxHealth
    local percentage = (health / maxHealth) * 100
    
    print(string.format("เลือด: %.0f/%.0f (%.1f%%)", health, maxHealth, percentage))
    
    -- เตือนเมื่อเลือดน้อย
    if percentage <= 25 then
        print("เตือน: เลือดน้อยมาก!")
    end
end)

-- ตรวจสอบ WalkSpeed
humanoid:GetPropertyChangedSignal("WalkSpeed"):Connect(function()
    print("ความเร็วเปลี่ยนเป็น: " .. humanoid.WalkSpeed)
end)

-- ตรวจสอบ Part Color
local part = workspace.MyPart
part:GetPropertyChangedSignal("BrickColor"):Connect(function()
    print("สีเปลี่ยนเป็น: " .. tostring(part.BrickColor))
end)
```

---

## 17.7 DescendantAdded/DescendantRemoving

```lua
-- ตรวจสอบเมื่อมีสิ่งของเพิ่ม/ลบใน workspace
game.Workspace.DescendantAdded:Connect(function(descendant)
    if descendant:IsA("Part") then
        print("Part ใหม่: " .. descendant.Name)
    end
end)

game.Workspace.DescendantRemoving:Connect(function(descendant)
    if descendant:IsA("Part") then
        print("Part ถูกลบ: " .. descendant.Name)
    end
end)

-- ตรวจสอบเมื่อมีผู้เล่น equip อาวุธ
local Players = game:GetService("Players")

Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function(character)
        character.ChildAdded:Connect(function(child)
            if child:IsA("Tool") then
                print(player.Name .. " equipped: " .. child.Name)
            end
        end)
        
        character.ChildRemoved:Connect(function(child)
            if child:IsA("Tool") then
                print(player.Name .. " unequipped: " .. child.Name)
            end
        end)
    end)
end)
```

---

## 17.8 RunService Events

```lua
local RunService = game:GetService("RunService")

-- Heartbeat: ทุก frame (server และ client)
local heartbeatConnection = RunService.Heartbeat:Connect(function(deltaTime)
    -- deltaTime = เวลาที่ผ่านไปตั้งแต่ frame ก่อน (วินาที)
    -- เหมาะสำหรับ: physics, AI update, timer
    
    -- ตัวอย่าง: อัปเดต timer
    -- gameTime = gameTime - deltaTime
end)

-- RenderStepped: ก่อน render (client only)
local renderConnection = RunService.RenderStepped:Connect(function(deltaTime)
    -- เหมาะสำหรับ: camera, visual effects
    -- ทำงานก่อน frame render
end)

-- Stepped: ก่อน physics step
local steppedConnection = RunService.Stepped:Connect(function(time, deltaTime)
    -- เหมาะสำหรับ: physics calculation
end)

-- หยุด loop เมื่อต้องการ
heartbeatConnection:Disconnect()
renderConnection:Disconnect()
steppedConnection:Disconnect()

-- Pattern: เกม loop ที่ดี
local lastTime = os.clock()
RunService.Heartbeat:Connect(function()
    local now = os.clock()
    local dt = now - lastTime
    lastTime = now
    
    -- อัปเดต game systems
    -- updateAI(dt)
    -- updateParticles(dt)
    -- updateTimers(dt)
end)
```

---

## 17.9 Custom Signal System

```lua
-- สร้าง Signal class เอง (เพื่อใช้ใน ModuleScripts)
local Signal = {}
Signal.__index = Signal

function Signal.new()
    return setmetatable({
        _connections = {},
        _connectionsToRemove = {},
        _firing = false
    }, Signal)
end

function Signal:Connect(callback)
    local connection = {
        callback = callback,
        connected = true
    }
    
    table.insert(self._connections, connection)
    
    -- Return connection object with Disconnect method
    return {
        Disconnect = function()
            connection.connected = false
            if not self._firing then
                self:_cleanup()
            end
        end
    }
end

function Signal:Once(callback)
    local connection
    connection = self:Connect(function(...)
        connection:Disconnect()
        callback(...)
    end)
    return connection
end

function Signal:Fire(...)
    self._firing = true
    
    for _, connection in ipairs(self._connections) do
        if connection.connected then
            -- pcall เพื่อป้องกัน error จาก callback
            local success, err = pcall(connection.callback, ...)
            if not success then
                warn("Signal callback error: " .. tostring(err))
            end
        end
    end
    
    self._firing = false
    self:_cleanup()
end

function Signal:_cleanup()
    local i = 1
    while i <= #self._connections do
        if not self._connections[i].connected then
            table.remove(self._connections, i)
        else
            i = i + 1
        end
    end
end

function Signal:Wait()
    local thread = coroutine.running()
    local conn
    conn = self:Once(function(...)
        task.spawn(thread, ...)
    end)
    return coroutine.yield()
end

-- ทดสอบ
local onDamage = Signal.new()
local onDeath = Signal.new()

onDamage:Connect(function(damage, source)
    print("รับความเสียหาย " .. damage .. " จาก " .. source)
end)

local deathConn
deathConn = onDeath:Connect(function(killer)
    print("ถูกสังหารโดย: " .. killer)
    deathConn:Disconnect()
end)

-- ยิง signals
onDamage:Fire(30, "Goblin")
onDamage:Fire(50, "Archer")
onDeath:Fire("Boss Monster")
onDeath:Fire("Skeleton")  -- ไม่ trigger เพราะ disconnect แล้ว
```

---

## 17.10 Event Best Practices

### 1. ป้องกัน Memory Leaks

```lua
-- ปัญหา: connections ไม่ถูก disconnect
local Players = game:GetService("Players")

Players.PlayerAdded:Connect(function(player)
    -- ปัญหา: ถ้า player ออก connections เหล่านี้ยังอยู่
    local conn1 = workspace.Part.Touched:Connect(function()
        -- บางอย่าง
    end)
    
    -- วิธีแก้: disconnect เมื่อ player ออก
    local cleanupConn
    cleanupConn = Players.PlayerRemoving:Connect(function(removingPlayer)
        if removingPlayer == player then
            conn1:Disconnect()
            cleanupConn:Disconnect()
        end
    end)
end)
```

### 2. Debounce Pattern

```lua
-- ป้องกัน event fire บ่อยเกินไป
local function createDebouncer(cooldown)
    local lastFired = 0
    
    return function(callback)
        return function(...)
            local now = tick()
            if now - lastFired >= cooldown then
                lastFired = now
                callback(...)
            end
        end
    end
end

local debounce = createDebouncer(1.0)  -- 1 วินาที cooldown

part.Touched:Connect(debounce(function(hit)
    print("Touched (debounced): " .. hit.Name)
end))
```

### 3. Event Queue

```lua
-- คิว events เพื่อประมวลผลทีละอัน
local EventQueue = {}
EventQueue.__index = EventQueue

function EventQueue.new()
    return setmetatable({
        queue = {},
        processing = false
    }, EventQueue)
end

function EventQueue:push(event)
    table.insert(self.queue, event)
    if not self.processing then
        self:process()
    end
end

function EventQueue:process()
    self.processing = true
    
    task.spawn(function()
        while #self.queue > 0 do
            local event = table.remove(self.queue, 1)
            -- ประมวลผล event
            if event.type == "damage" then
                print("ประมวลผล damage: " .. event.amount)
            elseif event.type == "pickup" then
                print("ประมวลผล pickup: " .. event.item)
            end
        end
        self.processing = false
    end)
end

-- ทดสอบ
local queue = EventQueue.new()
queue:push({type = "damage", amount = 50})
queue:push({type = "pickup", item = "ดาบ"})
queue:push({type = "damage", amount = 30})
```

---

## 17.11 แบบฝึกหัด

### แบบฝึกหัดที่ 1: Checkpoint System

```lua
-- สร้างระบบ Checkpoint:
-- - มี 5 checkpoints
-- - เมื่อผู้เล่นเหยียบ บันทึกตำแหน่ง
-- - เมื่อตาย spawn ที่ checkpoint ล่าสุด
-- - แสดงข้อความเมื่อถึง checkpoint ใหม่

local checkpoints = {}
local playerCheckpoints = {}  -- player -> checkpoint index

for i = 1, 5 do
    -- สมมติมี Part ชื่อ "Checkpoint1" ถึง "Checkpoint5"
    -- ใส่โค้ดที่นี่
end
```

### แบบฝึกหัดที่ 2: Chat Command System

```lua
-- สร้างระบบ command จาก chat:
-- /heal - ฟื้นฟูเลือด
-- /speed [number] - เปลี่ยนความเร็ว
-- /jump [number] - เปลี่ยน jump power
-- /reset - reset ตัวละคร

local Players = game:GetService("Players")

local commands = {}

commands["heal"] = function(player, args)
    -- เติมโค้ด
end

commands["speed"] = function(player, args)
    -- เติมโค้ด
end

Players.PlayerAdded:Connect(function(player)
    player.Chatted:Connect(function(message)
        -- ตรวจสอบและ execute command
        -- เติมโค้ด
    end)
end)
```

---

## สรุป

| Event Type | การใช้งาน |
|-----------|----------|
| `Players.PlayerAdded` | ผู้เล่นเข้าร่วม |
| `Players.PlayerRemoving` | ผู้เล่นออก |
| `player.CharacterAdded` | Character โหลด |
| `humanoid.Died` | ตัวละครตาย |
| `part.Touched` | มีอะไรสัมผัส |
| `button.MouseButton1Click` | กดปุ่ม |
| `RemoteEvent.OnServerEvent` | Client ส่งมาหา Server |
| `RemoteEvent.OnClientEvent` | Server ส่งมาหา Client |
| `RunService.Heartbeat` | ทุก frame |
| `:GetPropertyChangedSignal` | Property เปลี่ยน |

### บทถัดไป

ในบทที่ 18 เราจะเรียนเรื่อง **Instance Creation** - การสร้าง objects ใน scripts
