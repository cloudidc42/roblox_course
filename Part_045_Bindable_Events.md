# Part 45: BindableEvents และ BindableFunctions - การสื่อสารภายในฝั่งเดียว

## บทนำ

BindableEvent และ BindableFunction คือเครื่องมือสำหรับการสื่อสารระหว่าง Script ที่อยู่ฝั่งเดียวกัน (Server-to-Server หรือ Client-to-Client) ต่างจาก RemoteEvent ที่สื่อสารข้ามฝั่ง เหมาะสำหรับการสร้าง event system ภายใน game logic

## ทำความเข้าใจความแตกต่าง

```
BindableEvent:  Script A ──→ Script B  (ฝั่งเดียวกัน)
RemoteEvent:    Client  ──→ Server     (ข้ามฝั่ง)

BindableFunction:  Script A ⇄ Script B  (ฝั่งเดียวกัน, มี return)
RemoteFunction:    Client  ⇄ Server     (ข้ามฝั่ง, มี return)
```

## เมื่อไหร่ควรใช้ BindableEvent?

- **การสื่อสารระหว่าง Scripts** บนฝั่งเดียวกัน
- **Decoupling** - แยก Script ให้ไม่ต้องรู้จักกัน
- **Event-Driven Architecture** - ระบบที่ขับเคลื่อนด้วย events
- **Game State Management** - จัดการสถานะเกม

## การสร้างและใช้งาน BindableEvent

### พื้นฐาน

```lua
-- ServerScriptService/EventSystem.lua
-- สร้าง BindableEvent

-- วิธีที่ 1: สร้างด้วยโค้ด
local bindableEvent = Instance.new("BindableEvent")
bindableEvent.Name = "MyEvent"
bindableEvent.Parent = script  -- หรือ ServerStorage

-- เชื่อม listener
bindableEvent.Event:Connect(function(data)
    print("รับ event:", data)
end)

-- ยิง event
bindableEvent:Fire("Hello!")

-- วิธีที่ 2: สร้างใน ServerStorage แล้ว require
-- (ใช้ ModuleScript จัดการจะดีกว่า)
```

### ระบบ Event Bus

```lua
-- ServerScriptService/EventBus.lua (ModuleScript)
-- ระบบ event ส่วนกลางสำหรับ server scripts

local EventBus = {}
local events = {}

-- สร้าง event ใหม่
function EventBus.createEvent(name)
    if events[name] then
        return events[name]  -- ใช้อันเดิมถ้ามีแล้ว
    end
    
    local bindable = Instance.new("BindableEvent")
    bindable.Name = name
    
    events[name] = {
        bindable = bindable,
        listeners = 0
    }
    
    return events[name]
end

-- subscribe ฟัง event
function EventBus.on(name, callback)
    if not events[name] then
        EventBus.createEvent(name)
    end
    
    local eventData = events[name]
    eventData.listeners = eventData.listeners + 1
    
    return eventData.bindable.Event:Connect(callback)
end

-- emit ส่ง event
function EventBus.emit(name, ...)
    if not events[name] then
        warn("EventBus: event '" .. name .. "' ไม่มีอยู่")
        return
    end
    
    events[name].bindable:Fire(...)
end

-- once - ฟังครั้งเดียวแล้วหยุด
function EventBus.once(name, callback)
    local connection
    connection = EventBus.on(name, function(...)
        connection:Disconnect()
        callback(...)
    end)
    return connection
end

-- ล้าง event
function EventBus.clear(name)
    if events[name] then
        events[name].bindable:Destroy()
        events[name] = nil
    end
end

return EventBus
```

### ใช้งาน EventBus

```lua
-- ServerScriptService/GameManager.lua (Script)
local ServerScriptService = game:GetService("ServerScriptService")
local EventBus = require(ServerScriptService.EventBus)

-- ===== กำหนด events ที่ใช้ในเกม =====
-- Game State Events
local EVENTS = {
    GAME_STARTED = "GameStarted",
    GAME_ENDED = "GameEnded", 
    ROUND_STARTED = "RoundStarted",
    ROUND_ENDED = "RoundEnded",
    
    -- Player Events
    PLAYER_JOINED = "PlayerJoined",
    PLAYER_LEFT = "PlayerLeft",
    PLAYER_DIED = "PlayerDied",
    PLAYER_LEVELED_UP = "PlayerLeveledUp",
    
    -- Combat Events
    ENEMY_KILLED = "EnemyKilled",
    BOSS_SPAWNED = "BossSpawned",
    BOSS_DEFEATED = "BossDefeated",
    
    -- Economy Events
    ITEM_DROPPED = "ItemDropped",
    ITEM_COLLECTED = "ItemCollected",
    PURCHASE_COMPLETED = "PurchaseCompleted"
}

-- ===== Game Logic =====

local gameState = "waiting"  -- waiting, playing, ended
local currentRound = 0
local playersAlive = {}

-- เมื่อเกมเริ่ม
local function startGame()
    gameState = "playing"
    currentRound = currentRound + 1
    
    print("🎮 เกมเริ่ม! รอบที่ " .. currentRound)
    
    -- แจ้ง systems อื่นว่าเกมเริ่ม
    EventBus.emit(EVENTS.GAME_STARTED, {
        round = currentRound,
        timestamp = os.time()
    })
    
    EventBus.emit(EVENTS.ROUND_STARTED, currentRound)
end

-- เมื่อผู้เล่นตาย
local function onPlayerDied(player, killer)
    playersAlive[player.UserId] = nil
    
    EventBus.emit(EVENTS.PLAYER_DIED, {
        player = player,
        killer = killer,
        round = currentRound
    })
    
    -- ตรวจสอบว่าเหลือใครอีกไหม
    local aliveCount = 0
    for _ in pairs(playersAlive) do
        aliveCount = aliveCount + 1
    end
    
    if aliveCount <= 1 then
        -- เกมจบ
        local winner = nil
        for _, p in pairs(playersAlive) do
            winner = p
        end
        endGame(winner)
    end
end

-- เมื่อเกมจบ
local function endGame(winner)
    gameState = "ended"
    
    EventBus.emit(EVENTS.GAME_ENDED, {
        winner = winner,
        round = currentRound,
        duration = os.time()
    })
    
    print("🏆 เกมจบ! ผู้ชนะ: " .. (winner and winner.Name or "ไม่มี"))
end

-- ===== Subscribe to Events =====

-- Reward system สมัคร event
EventBus.on(EVENTS.PLAYER_DIED, function(data)
    if data.killer then
        -- ให้รางวัลแก่ผู้ฆ่า
        print("💰 ให้รางวัล " .. data.killer.Name .. " ที่ฆ่า " .. data.player.Name)
        -- giveReward(data.killer, 50)
    end
end)

-- Statistics system สมัคร event
EventBus.on(EVENTS.PLAYER_DIED, function(data)
    -- บันทึกสถิติ
    print("📊 บันทึก: " .. data.player.Name .. " ตายในรอบ " .. data.round)
end)

-- เริ่มเกมหลัง 5 วินาที
task.delay(5, startGame)
```

## BindableFunction

```lua
-- BindableFunction ใช้เมื่อต้องการ return value

-- ServerScriptService/DataService.lua (ModuleScript)
local DataService = {}

-- BindableFunction สำหรับ query ข้อมูล
local getPlayerStatsBF = Instance.new("BindableFunction")
getPlayerStatsBF.Name = "GetPlayerStats"

-- กำหนด callback
getPlayerStatsBF.OnInvoke = function(player)
    -- คืนสถิติผู้เล่น
    return {
        kills = 42,
        deaths = 15,
        wins = 7,
        kdr = 42/15
    }
end

-- ฟังก์ชัน public
function DataService.getPlayerStats(player)
    return getPlayerStatsBF:Invoke(player)
end

return DataService
```

## ระบบ Event-Driven Game สมบูรณ์

```lua
-- ServerScriptService/Systems/CombatSystem.lua (ModuleScript)
-- ระบบต่อสู้ที่ใช้ EventBus

local Players = game:GetService("Players")
local ServerScriptService = game:GetService("ServerScriptService")

-- สมมติว่า EventBus อยู่ใน ServerScriptService
local EventBus = {}  -- placeholder

local CombatSystem = {}

-- ค่าตั้งค่าการต่อสู้
local COMBAT_CONFIG = {
    baseAttackDamage = 10,
    criticalChance = 0.15,    -- 15%
    criticalMultiplier = 2.0, -- 2x damage
    attackCooldown = 0.5,
    respawnTime = 5
}

-- ติดตาม cooldown
local attackCooldowns = {}

-- คำนวณ damage
local function calculateDamage(attacker, weapon)
    local baseDamage = weapon and weapon.damage or COMBAT_CONFIG.baseAttackDamage
    
    -- Critical hit
    local isCrit = math.random() < COMBAT_CONFIG.criticalChance
    local finalDamage = baseDamage
    
    if isCrit then
        finalDamage = finalDamage * COMBAT_CONFIG.criticalMultiplier
        print("💥 Critical Hit!")
    end
    
    -- ปัดเป็นจำนวนเต็ม
    return math.floor(finalDamage), isCrit
end

-- ฟังก์ชันโจมตี
function CombatSystem.attack(attacker, target, weapon)
    -- ตรวจสอบ cooldown
    local attackerId = attacker.UserId or attacker.Name
    local lastAttack = attackCooldowns[attackerId] or 0
    
    if os.clock() - lastAttack < COMBAT_CONFIG.attackCooldown then
        return false, "cooldown"
    end
    
    attackCooldowns[attackerId] = os.clock()
    
    -- ตรวจสอบ target
    local targetHumanoid = target.Character and 
                           target.Character:FindFirstChild("Humanoid")
    
    if not targetHumanoid or targetHumanoid.Health <= 0 then
        return false, "invalid_target"
    end
    
    -- คำนวณและใช้ damage
    local damage, isCrit = calculateDamage(attacker, weapon)
    targetHumanoid:TakeDamage(damage)
    
    -- ยิง event
    EventBus.emit("CombatDamage", {
        attacker = attacker,
        target = target,
        damage = damage,
        isCrit = isCrit,
        weaponId = weapon and weapon.id
    })
    
    -- ตรวจสอบว่าตายหรือยัง
    if targetHumanoid.Health <= 0 then
        EventBus.emit("PlayerDied", {
            victim = target,
            killer = attacker,
            cause = "combat"
        })
        
        -- เริ่ม respawn
        task.delay(COMBAT_CONFIG.respawnTime, function()
            if target.Character then
                target:LoadCharacter()
            end
        end)
    end
    
    return true, damage, isCrit
end

-- ฟัง event จาก systems อื่น
EventBus.on("GameStarted", function(gameData)
    -- รีเซ็ต cooldowns เมื่อเกมใหม่เริ่ม
    attackCooldowns = {}
    print("[CombatSystem] รีเซ็ต cooldowns สำหรับรอบใหม่")
end)

EventBus.on("PlayerDied", function(data)
    -- ล้าง cooldown ของผู้ที่ตาย
    if data.victim then
        local id = data.victim.UserId or data.victim.Name
        attackCooldowns[id] = nil
    end
end)

return CombatSystem
```

## Pattern: Observer

```lua
-- Observer Pattern ด้วย BindableEvent
-- ใช้สำหรับ UI update เมื่อ state เปลี่ยน

-- ServerScriptService/StateManager.lua (ModuleScript)
local StateManager = {}

-- State ปัจจุบัน
local state = {
    gamePhase = "lobby",    -- lobby, countdown, playing, ended
    playerCount = 0,
    timeRemaining = 0,
    winner = nil
}

-- BindableEvent สำหรับ state changes
local stateChangedEvent = Instance.new("BindableEvent")
stateChangedEvent.Name = "StateChanged"

-- เปลี่ยน state
function StateManager.setState(key, value)
    local oldValue = state[key]
    state[key] = value
    
    if oldValue ~= value then
        -- แจ้ง observers
        stateChangedEvent:Fire(key, value, oldValue)
    end
end

-- ดึง state
function StateManager.getState(key)
    return state[key]
end

-- ดึง state ทั้งหมด
function StateManager.getAll()
    local copy = {}
    for k, v in pairs(state) do
        copy[k] = v
    end
    return copy
end

-- Subscribe to changes
function StateManager.onChange(callback)
    return stateChangedEvent.Event:Connect(callback)
end

-- Subscribe to specific key
function StateManager.onKeyChange(key, callback)
    return stateChangedEvent.Event:Connect(function(changedKey, newValue, oldValue)
        if changedKey == key then
            callback(newValue, oldValue)
        end
    end)
end

return StateManager
```

### ใช้งาน StateManager

```lua
-- GameController.lua
local StateManager = require(script.Parent.StateManager)

-- ฟัง state change
StateManager.onKeyChange("gamePhase", function(newPhase, oldPhase)
    print(string.format("เกมเปลี่ยนจาก '%s' เป็น '%s'", oldPhase, newPhase))
    
    if newPhase == "playing" then
        -- เริ่มระบบต่างๆ
        print("🎮 เกมเริ่มเล่น!")
    elseif newPhase == "ended" then
        -- หยุดระบบต่างๆ
        print("🏁 เกมจบ!")
    end
end)

-- เปลี่ยน state
task.delay(3, function()
    StateManager.setState("gamePhase", "countdown")
end)

task.delay(8, function()
    StateManager.setState("gamePhase", "playing")
    StateManager.setState("timeRemaining", 300)
end)
```

## BindableEvent สำหรับ UI (Client-side)

```lua
-- LocalScript ใน StarterGui
-- ระบบ UI event ที่ใช้ BindableEvent

local uiEventBus = {}
local uiEvents = {}

function uiEventBus.on(eventName, callback)
    if not uiEvents[eventName] then
        uiEvents[eventName] = Instance.new("BindableEvent")
    end
    return uiEvents[eventName].Event:Connect(callback)
end

function uiEventBus.emit(eventName, ...)
    if uiEvents[eventName] then
        uiEvents[eventName]:Fire(...)
    end
end

-- ===== UI Components =====

-- HUD Component
local function createHUD()
    -- สร้าง HUD...
    
    -- ฟัง event เพื่ออัพเดท HUD
    uiEventBus.on("PlayerHealthChanged", function(health, maxHealth)
        print("❤️ HP: " .. health .. "/" .. maxHealth)
        -- อัพเดท health bar...
    end)
    
    uiEventBus.on("PlayerCoinsChanged", function(coins)
        print("💰 Coins: " .. coins)
        -- อัพเดท coin display...
    end)
end

-- Inventory Component
local function createInventory()
    uiEventBus.on("InventoryUpdated", function(items)
        print("🎒 Inventory อัพเดท: " .. #items .. " items")
        -- อัพเดท inventory UI...
    end)
    
    uiEventBus.on("ItemEquipped", function(item)
        print("⚔️ Equipped: " .. item.name)
    end)
end

-- เริ่มต้น UI
createHUD()
createInventory()

-- จำลองการเปลี่ยนข้อมูล
task.delay(2, function()
    uiEventBus.emit("PlayerHealthChanged", 75, 100)
    uiEventBus.emit("PlayerCoinsChanged", 250)
    uiEventBus.emit("InventoryUpdated", {
        { name = "Sword", quantity = 1 },
        { name = "Potion", quantity = 5 }
    })
end)
```

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Achievement System
สร้างระบบ achievement ที่ใช้ EventBus เมื่อผู้เล่นทำสิ่งต่างๆ

```lua
-- Achievement System ที่ใช้ EventBus
local AchievementSystem = {}

local achievements = {
    first_kill = {
        name = "ฆาตกรหน้าใหม่",
        description = "ฆ่าศัตรูครั้งแรก",
        reward = { coins = 100 },
        unlocked = false
    },
    level_10 = {
        name = "นักรบระดับ 10",
        description = "เลื่อนระดับถึง Level 10",
        reward = { gems = 5 },
        unlocked = false
    },
    collector = {
        name = "นักสะสม",
        description = "เก็บของ 100 ชิ้น",
        reward = { coins = 500 },
        unlocked = false
    }
}

-- ฟัง events ต่างๆ
EventBus.on("EnemyKilled", function(data)
    if not achievements.first_kill.unlocked then
        achievements.first_kill.unlocked = true
        EventBus.emit("AchievementUnlocked", "first_kill", data.player)
        print("🏆 Achievement: " .. achievements.first_kill.name)
    end
end)

EventBus.on("PlayerLeveledUp", function(data)
    if data.newLevel >= 10 and not achievements.level_10.unlocked then
        achievements.level_10.unlocked = true
        EventBus.emit("AchievementUnlocked", "level_10", data.player)
    end
end)

return AchievementSystem
```

### แบบฝึกหัดที่ 2: Notification Queue
สร้างระบบ queue notification ที่แสดงทีละอัน

### แบบฝึกหัดที่ 3: Game State Machine
สร้าง state machine สำหรับจัดการสถานะเกม (Lobby → Countdown → Playing → Ended)

## สรุป

BindableEvent และ BindableFunction เป็นเครื่องมือสำคัญสำหรับ:
- **Decoupling** - แยก modules ให้ไม่ต้องรู้จักกัน
- **Event-Driven Programming** - code ที่ react ต่อ events
- **State Management** - จัดการสถานะเกม
- **UI Updates** - อัพเดท UI เมื่อข้อมูลเปลี่ยน

ใช้ร่วมกับ RemoteEvent เพื่อสร้างระบบสื่อสารที่สมบูรณ์ทั้ง server และ client
