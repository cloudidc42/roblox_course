# Part 48: OOP ขั้นสูงใน Roblox

## บทนำ

หลังจากเรียนรู้ OOP พื้นฐานแล้ว บทนี้จะลงลึกถึง Inheritance, Composition, Mixins และ Pattern ขั้นสูงที่ใช้ในเกมจริง

## Inheritance (การสืบทอด)

```lua
-- ReplicatedStorage/Classes/Entity.lua
-- Base class สำหรับทุกสิ่งในเกม

local Entity = {}
Entity.__index = Entity

function Entity.new(id, position)
    local self = setmetatable({}, Entity)
    self.id = id or math.random(100000, 999999)
    self.position = position or Vector3.new(0, 0, 0)
    self.isActive = true
    self.tags = {}
    self._connections = {}
    return self
end

function Entity:addTag(tag)
    self.tags[tag] = true
end

function Entity:hasTag(tag)
    return self.tags[tag] == true
end

function Entity:destroy()
    self.isActive = false
    -- ล้าง connections
    for _, conn in ipairs(self._connections) do
        if conn.Connected then
            conn:Disconnect()
        end
    end
    self._connections = {}
    print("[Entity] Destroyed: " .. tostring(self.id))
end

function Entity:addConnection(connection)
    table.insert(self._connections, connection)
end

return Entity
```

```lua
-- ReplicatedStorage/Classes/Character.lua
-- Inherits from Entity

local Entity = require(script.Parent.Entity)

local Character = setmetatable({}, { __index = Entity })
Character.__index = Character

function Character.new(name, characterClass, position)
    -- สร้าง Entity ก่อน
    local self = Entity.new(nil, position)
    
    -- เปลี่ยน metatable เป็น Character
    setmetatable(self, Character)
    
    -- เพิ่ม Character properties
    self.name = name
    self.class = characterClass
    self.level = 1
    self.health = 100
    self.maxHealth = 100
    self.mana = 50
    self.maxMana = 50
    self.attack = 10
    self.defense = 5
    self.isAlive = true
    
    -- Tag
    self:addTag("character")
    
    return self
end

-- Override destroy ของ Entity
function Character:destroy()
    print("[Character] Destroying " .. self.name)
    -- เรียก parent destroy
    Entity.destroy(self)  -- สำคัญ! เรียก parent method
end

-- Methods เฉพาะ Character
function Character:takeDamage(amount)
    self.health = math.max(0, self.health - amount)
    if self.health <= 0 and self.isAlive then
        self:die()
    end
end

function Character:die()
    self.isAlive = false
    print("💀 " .. self.name .. " ตายแล้ว!")
end

function Character:printInfo()
    print(string.format("[%s] %s Lv.%d HP:%d/%d",
        self.class, self.name, self.level,
        self.health, self.maxHealth))
end

return Character
```

```lua
-- ReplicatedStorage/Classes/Warrior.lua
-- Inherits from Character

local Character = require(script.Parent.Character)

local Warrior = setmetatable({}, { __index = Character })
Warrior.__index = Warrior

function Warrior.new(name, position)
    local self = Character.new(name, "Warrior", position)
    setmetatable(self, Warrior)
    
    -- Warrior stats (สูงกว่า base)
    self.maxHealth = 150
    self.health = 150
    self.attack = 15
    self.defense = 12
    self.rage = 0         -- ค่า rage พิเศษของ Warrior
    self.maxRage = 100
    
    -- Warrior abilities
    self.abilities = {
        "BattleCry",
        "ShieldBash",
        "Whirlwind"
    }
    
    self:addTag("warrior")
    
    return self
end

-- Override takeDamage - Warrior สะสม rage เมื่อโดนโจมตี
function Warrior:takeDamage(amount)
    -- เรียก parent method
    Character.takeDamage(self, amount)
    
    -- เพิ่ม rage เมื่อโดนโจมตี
    self.rage = math.min(self.maxRage, self.rage + amount * 0.5)
    
    if self.rage >= 100 then
        print(self.name .. " เต็ม Rage!")
    end
end

-- Warrior-specific methods
function Warrior:battleCry()
    if self.rage < 30 then
        print("Rage ไม่พอสำหรับ Battle Cry!")
        return false
    end
    
    self.rage = self.rage - 30
    self.attack = self.attack + 10
    
    print(self.name .. " ใช้ Battle Cry! ATK +" .. 10)
    
    -- ลด buff หลัง 10 วินาที
    task.delay(10, function()
        if self.isAlive then
            self.attack = self.attack - 10
            print(self.name .. " Battle Cry หมดแล้ว")
        end
    end)
    
    return true
end

function Warrior:whirlwind(enemies)
    if self.rage < 50 then
        print("Rage ไม่พอ!")
        return 0
    end
    
    self.rage = self.rage - 50
    local totalDamage = 0
    
    for _, enemy in ipairs(enemies) do
        local damage = math.floor(self.attack * 0.75)
        enemy:takeDamage(damage)
        totalDamage = totalDamage + damage
    end
    
    print(self.name .. " ใช้ Whirlwind! โจมตี " .. #enemies .. " เป้าหมาย")
    return totalDamage
end

return Warrior
```

```lua
-- ReplicatedStorage/Classes/Mage.lua
-- Inherits from Character

local Character = require(script.Parent.Character)

local Mage = setmetatable({}, { __index = Character })
Mage.__index = Mage

function Mage.new(name, position)
    local self = Character.new(name, "Mage", position)
    setmetatable(self, Mage)
    
    -- Mage stats (อ่อนแอกว่าแต่ MP สูง)
    self.maxHealth = 70
    self.health = 70
    self.maxMana = 150
    self.mana = 150
    self.attack = 5         -- physical อ่อน
    self.defense = 3        -- defense ต่ำ
    self.magicPower = 25    -- magic แรง!
    
    self:addTag("mage")
    
    return self
end

-- Spell casting
function Mage:castFireball(target)
    local manaCost = 30
    
    if self.mana < manaCost then
        print("Mana ไม่พอ!")
        return false
    end
    
    self.mana = self.mana - manaCost
    local damage = math.floor(self.magicPower * 2.5)
    target:takeDamage(damage)
    
    print(string.format("%s ร่าย Fireball! %d magic damage!", self.name, damage))
    return true
end

function Mage:castIceBlast(target)
    local manaCost = 20
    if self.mana < manaCost then return false end
    
    self.mana = self.mana - manaCost
    local damage = math.floor(self.magicPower * 1.5)
    target:takeDamage(damage)
    
    -- Slow effect
    if target.speed then
        target.speed = math.max(1, target.speed - 5)
        task.delay(5, function()
            if target.isAlive then
                target.speed = target.speed + 5
            end
        end)
        print(target.name .. " ถูก slow!")
    end
    
    print(string.format("%s ร่าย Ice Blast! %d magic damage!", self.name, damage))
    return true
end

return Mage
```

## Mixin Pattern

```lua
-- ReplicatedStorage/Mixins/Damageable.lua
-- Mixin สำหรับสิ่งที่รับ damage ได้

local Damageable = {}

function Damageable:applyDamageable(maxHealth, defense)
    self.health = maxHealth
    self.maxHealth = maxHealth
    self.defense = defense or 0
    self.isInvincible = false
    self.invincibleTimer = 0
end

function Damageable:takeDamage(amount, damageType)
    if self.isInvincible then return 0 end
    
    local finalDamage = math.max(1, amount - self.defense)
    self.health = math.max(0, self.health - finalDamage)
    
    if self.health <= 0 then
        if self.onDeath then
            self:onDeath()
        end
    end
    
    return finalDamage
end

function Damageable:makeInvincible(duration)
    self.isInvincible = true
    task.delay(duration, function()
        self.isInvincible = false
    end)
end

function Damageable:isFullHealth()
    return self.health >= self.maxHealth
end

function Damageable:getHealthPercent()
    return self.health / self.maxHealth
end

-- Damageable/Moveable
local Moveable = {}

function Moveable:applyMoveable(speed)
    self.speed = speed or 10
    self.velocity = Vector3.new(0,0,0)
    self.isMoving = false
end

function Moveable:moveTo(targetPosition)
    -- Placeholder for movement logic
    self.position = targetPosition
    self.isMoving = true
end

function Moveable:stopMoving()
    self.isMoving = false
    self.velocity = Vector3.new(0,0,0)
end

-- ฟังก์ชัน Mixin
local function applyMixin(target, mixin)
    for key, value in pairs(mixin) do
        if type(value) == "function" and key ~= "applyMixin" then
            target[key] = value
        end
    end
end

return {
    Damageable = Damageable,
    Moveable = Moveable,
    applyMixin = applyMixin
}
```

## Design Pattern: Strategy

```lua
-- Strategy Pattern: เปลี่ยน behavior ได้ตอน runtime

-- AI Strategies
local AIStrategies = {}

-- Aggressive: โจมตีเมื่อเห็นผู้เล่น
AIStrategies.aggressive = {
    name = "Aggressive",
    
    update = function(self, npc, players, dt)
        local nearest = npc:findNearestPlayer(players)
        if nearest then
            if npc:distanceTo(nearest) < 30 then
                npc:moveToward(nearest.position)
                if npc:distanceTo(nearest) < 5 then
                    npc:attackTarget(nearest)
                end
            end
        end
    end
}

-- Defensive: หนีเมื่อ HP ต่ำ
AIStrategies.defensive = {
    name = "Defensive",
    
    update = function(self, npc, players, dt)
        if npc:getHealthPercent() < 0.3 then
            -- หนี
            local nearest = npc:findNearestPlayer(players)
            if nearest then
                local awayDir = (npc.position - nearest.position).Unit
                npc:moveToward(npc.position + awayDir * 20)
            end
        else
            -- โจมตีปกติ
            AIStrategies.aggressive.update(self, npc, players, dt)
        end
    end
}

-- Patrol: เดินลาดตระเวน
AIStrategies.patrol = {
    name = "Patrol",
    patrolPoints = {},
    currentPoint = 1,
    waitTime = 2,
    waitTimer = 0,
    
    update = function(self, npc, players, dt)
        if #self.patrolPoints == 0 then return end
        
        local target = self.patrolPoints[self.currentPoint]
        
        if npc:distanceTo(target) < 2 then
            self.waitTimer = self.waitTimer + dt
            if self.waitTimer >= self.waitTime then
                self.waitTimer = 0
                self.currentPoint = (self.currentPoint % #self.patrolPoints) + 1
            end
        else
            npc:moveToward(target)
        end
        
        -- โจมตีถ้าเห็นผู้เล่น
        for _, player in ipairs(players) do
            if npc:distanceTo(player.position) < 10 then
                npc:attackTarget(player)
                break
            end
        end
    end
}

-- NPC ที่ใช้ Strategy
local NPC = {}
NPC.__index = NPC

function NPC.new(name, position, strategy)
    local self = setmetatable({}, NPC)
    self.name = name
    self.position = position
    self.health = 100
    self.maxHealth = 100
    self.strategy = strategy or AIStrategies.aggressive
    self.attackCooldown = 0
    return self
end

function NPC:setStrategy(strategy)
    print(self.name .. " เปลี่ยน AI เป็น: " .. strategy.name)
    self.strategy = strategy
end

function NPC:update(players, dt)
    if self.attackCooldown > 0 then
        self.attackCooldown = self.attackCooldown - dt
    end
    
    self.strategy:update(self, players, dt)
end

function NPC:findNearestPlayer(players)
    local nearest = nil
    local nearestDist = math.huge
    
    for _, player in ipairs(players) do
        local dist = self:distanceTo(player.position)
        if dist < nearestDist then
            nearest = player
            nearestDist = dist
        end
    end
    
    return nearest
end

function NPC:distanceTo(position)
    return (self.position - position).Magnitude
end

function NPC:moveToward(target)
    local direction = (target - self.position).Unit
    self.position = self.position + direction * 10 * (1/60)  -- placeholder
end

function NPC:attackTarget(target)
    if self.attackCooldown > 0 then return end
    self.attackCooldown = 1.5  -- cooldown 1.5 วินาที
    
    if target.takeDamage then
        target:takeDamage(15)
    end
    
    print(self.name .. " โจมตี " .. (target.name or "ผู้เล่น"))
end

function NPC:getHealthPercent()
    return self.health / self.maxHealth
end

-- ตัวอย่าง: NPC เปลี่ยน AI เมื่อ HP ต่ำ
function NPC:takeDamage(amount)
    self.health = math.max(0, self.health - amount)
    
    -- เปลี่ยน strategy เมื่อ HP ต่ำ
    if self.health < 30 and self.strategy.name == "Aggressive" then
        self:setStrategy(AIStrategies.defensive)
    end
end

return {
    NPC = NPC,
    Strategies = AIStrategies
}
```

## Design Pattern: Observer

```lua
-- Observable object - notify เมื่อ property เปลี่ยน

local Observable = {}
Observable.__index = Observable

function Observable.new(initialValue)
    local self = setmetatable({}, Observable)
    self._value = initialValue
    self._listeners = {}
    return self
end

function Observable:get()
    return self._value
end

function Observable:set(newValue)
    local oldValue = self._value
    if oldValue ~= newValue then
        self._value = newValue
        self:_notify(newValue, oldValue)
    end
end

function Observable:onChange(callback)
    table.insert(self._listeners, callback)
    return function()
        for i, listener in ipairs(self._listeners) do
            if listener == callback then
                table.remove(self._listeners, i)
                break
            end
        end
    end
end

function Observable:_notify(newValue, oldValue)
    for _, listener in ipairs(self._listeners) do
        pcall(listener, newValue, oldValue)
    end
end

-- ใช้งาน
local playerHealth = Observable.new(100)

-- ฟัง changes
local disconnect = playerHealth:onChange(function(new, old)
    print(string.format("HP เปลี่ยน: %d -> %d", old, new))
    
    if new <= 25 then
        print("⚠️ HP ต่ำมาก!")
    end
end)

playerHealth:set(75)   -- HP เปลี่ยน: 100 -> 75
playerHealth:set(20)   -- HP เปลี่ยน: 75 -> 20 + ⚠️

disconnect()  -- หยุดฟัง
```

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Enemy Hierarchy
สร้าง hierarchy ของ enemies:
- `Enemy` (base) → `MeleeEnemy` + `RangedEnemy` + `BossEnemy`

### แบบฝึกหัดที่ 2: Spell System
สร้าง spell system ด้วย OOP:
- `Spell` (base) → `FireSpell`, `IceSpell`, `HealSpell`

### แบบฝึกหัดที่ 3: Component System
สร้าง component system ที่ flexible กว่า inheritance

## สรุป

OOP ขั้นสูงใน Lua/Roblox:
- **Inheritance** - `setmetatable({}, {__index = Parent})`
- **Mixins** - เพิ่ม behavior โดยไม่ inheritance
- **Strategy Pattern** - เปลี่ยน behavior ได้ runtime
- **Observer Pattern** - react ต่อ changes
- **Composition over Inheritance** - prefer ใช้ mixins/composition

หลักการ: **"Prefer Composition over Inheritance"**
